# Rust QUIC 서버와 Android 클라이언트 통신

## 서론: 왜 QUIC인가? TCP를 넘어선 차세대 프로토콜

현대 인터넷 애플리케이션의 성능 요구사항은 기존의 전송 프로토콜인 TCP(Transmission Control Protocol)의 한계를 넘어서고 있다. 1974년에 설계된 TCP는 신뢰성을 보장하는 훌륭한 프로토콜이지만, 현대의 다중화되고 지연 시간에 민감한 환경에서는 몇 가지 근본적인 문제를 드러낸다. QUIC(Quick UDP Internet Connections)은 이러한 문제들을 해결하기 위해 UDP(User Datagram Protocol) 위에 구축된 차세대 전송 프로토콜이다.

### 0.1 QUIC의 기술적 우위

QUIC이 제공하는 가장 핵심적인 개선점은 **Head-of-Line (HOL) 블로킹 문제의 해결**이다. HTTP/2는 단일 TCP 연결 내에서 여러 개의 논리적 스트림을 다중화하여 성능을 개선했지만, TCP 계층 자체는 스트림의 존재를 인지하지 못한다. 따라서 TCP 패킷 하나가 유실되면, 해당 패킷이 재전송되어 순서가 맞춰질 때까지 그 뒤에 오는 모든 패킷(다른 스트림에 속한 패킷 포함)이 전송 계층에서 대기해야 한다. 이는 관련 없는 스트림까지 불필요하게 지연시키는 결과를 낳는다.

반면 QUIC은 UDP를 기반으로 동작하며, 각 스트림을 전송 계층 수준에서 독립적인 단위로 취급한다. 한 스트림에서 패킷 손실이 발생하더라도 다른 스트림의 데이터 전송에는 영향을 주지 않는다. 이는 특히 이미지, CSS, JavaScript 등 수많은 리소스를 동시에 요청하는 현대 웹 환경이나, 여러 데이터 채널이 필요한 실시간 애플리케이션에서 지연 시간을 극적으로 줄여준다.

### 0.2 내장된 보안 및 성능

QUIC의 또 다른 강력한 특징은 **보안과 성능을 프로토콜 설계의 핵심에 통합**했다는 점이다. QUIC은 TLS 1.3을 전송 핸드셰이크 과정에 내장했다. 기존의 TCP + TLS 스택에서는 TCP 연결을 위한 3-way 핸드셰이크가 완료된 후에 별도로 TLS 핸드셰이크가 진행되어 여러 번의 왕복 시간(Round-Trip Time, RTT)이 소요되었다. QUIC은 이 두 과정을 하나로 통합하여 연결 설정에 필요한 RTT를 최소 1회로 줄였다. 특히, 한 번 연결했던 서버에 다시 접속할 때는 0-RTT 핸드셰이크를 통해 데이터를 즉시 전송할 수 있어, 모바일 환경처럼 네트워크 전환이 잦은 경우에도 매우 빠른 연결 재개를 보장한다.

### 0.3 가이드의 목표

이 보고서는 Rust로 구현된 고성능 QUIC 서버와 Android 클라이언트 간의 완전한 통신 파이프라인을 구축하는 과정을 상세히 안내하는 것을 목표로 한다. 단순한 이론 설명에 그치지 않고, 실제 개발 환경에서 마주하게 될 가장 현실적인 시나리오, 즉 **자체 서명 인증서(self-signed certificate)를 사용한 TLS 보안 통신** 예제를 제공한다. 이 과정에서 발생하는 주요 기술적 난관, 특히 Android 클라이언트 구현 시의 함정을 분석하고, 이를 해결하기 위한 가장 안정적이고 유연한 아키텍처를 제시할 것이다.

## 1. 부: Rust QUIC 서버 구축

Rust는 메모리 안전성과 높은 성능을 보장하므로, 네트워크 서버와 같은 시스템 프로그래밍에 이상적인 언어다. Rust 생태계에는 강력한 QUIC 구현체들이 존재하며, 이를 활용하여 안정적인 QUIC 서버를 손쉽게 구축할 수 있다.

### 1.1  라이브러리 선택: `quinn` vs. `s2n-quic`

Rust 생태계에서 QUIC 서버를 구현할 때 주로 고려되는 라이브러리는 `quinn`과 AWS에서 개발한 `s2n-quic`이다. 두 라이브러리 모두 IETF 표준을 준수하는 고품질 구현체이지만, 설계 철학과 사용성에 차이가 있다.

- **`quinn`**: 가장 널리 알려진 순수 Rust 구현체 중 하나로, Rust의 대표적인 비동기 런타임인 `tokio`와 긴밀하게 통합되어 있다. `quinn`은 사용하기 쉬운 고수준의 비동기 API를 제공하여 개발자가 QUIC의 복잡한 내부 동작을 신경 쓰지 않고도 빠르게 애플리케이션을 개발할 수 있도록 돕는다. 풍부한 공식 예제는 서버 및 클라이언트 구현, 특히 자체 서명 인증서를 사용한 테스트 시나리오까지 상세히 다루고 있어, 대부분의 일반적인 사용 사례에 가장 적합한 선택이다.
- **`s2n-quic`**: AWS에서 개발한 이 라이브러리는 'provider'라는 독특한 아키텍처를 통해 높은 수준의 구성 가능성(configurability)과 모듈성을 제공한다. 개발자는 TLS 구현체(`s2n-tls` 또는 `rustls`), 이벤트 로깅, 주소 토큰 관리 등 QUIC 스택의 핵심 구성 요소를 직접 선택하고 교체할 수 있다. 이는 FIPS 규격 암호화가 필요하거나, 특정 클라우드 환경에 최적화된 로직을 구현해야 하는 등 세밀한 제어가 요구되는 특수한 환경에 매우 강력한 장점을 제공한다.

이 가이드의 목표는 실용적이고 이해하기 쉬운 예제를 제공하는 것이므로, 범용성과 개발 편의성이 뛰어난 **`quinn`**을 선택한다. `quinn`은 자체 서명 인증서를 생성하고 서버에 적용하는 과정을 직관적으로 보여주어, 사용자의 요구사항에 가장 직접적으로 부합한다.

| 기능              | `quinn`                        | `s2n-quic`                          | 선택 근거                                                    |
| ----------------- | ------------------------------ | ----------------------------------- | ------------------------------------------------------------ |
| **API 스타일**    | 고수준 비동기 API              | Provider 기반의 모듈형 API          | `quinn`의 API는 `tokio` 사용자에게 친숙하며, 적은 코드로 빠르게 구현 가능하다. |
| **비동기 런타임** | `tokio`와 긴밀하게 통합        | `tokio` 기반으로 동작               | 두 라이브러리 모두 `tokio`를 사용하지만, `quinn`은 더 깊이 통합된 경험을 제공한다. |
| **TLS 백엔드**    | `rustls`                       | `s2n-tls` 또는 `rustls` (선택 가능) | `s2n-quic`은 TLS 백엔드 선택권을 주지만, 일반적인 경우 `rustls`로 충분하다. |
| **인증서 처리**   | 예제에서 `rcgen`으로 자동 생성 | PEM 파일 경로를 직접 지정           | `quinn`의 예제는 개발 환경 설정의 번거로움을 줄여준다.       |
| **사용 편의성**   | 높음 (빠른 시작에 유리)        | 중간 (세밀한 제어에 유리)           | 이 가이드의 목표는 '실용적인 예제'이므로, 사용 편의성이 높은 `quinn`이 더 적합하다. |
| **구성 가능성**   | 중간                           | 높음                                | `s2n-quic`은 매우 유연하지만, 이 예제에서는 그 정도의 구성 가능성이 필요하지 않다. |

### 1.2  개발 환경을 위한 자체 서명 인증서 생성

QUIC은 TLS 1.3을 기반으로 하므로 모든 통신은 기본적으로 암호화된다. 이를 위해 서버는 자신의 신원을 증명하는 TLS 인증서를 클라이언트에게 제공해야 한다. 프로덕션 환경에서는 Let's Encrypt와 같은 신뢰할 수 있는 공인 인증 기관(Certificate Authority, CA)에서 발급한 인증서를 사용하는 것이 필수적이다.

하지만 개발 및 테스트 환경에서는 매번 공인 인증서를 발급받는 것이 번거롭다. 이 경우, **자체 서명 인증서**를 사용하는 것이 효율적이다. `quinn`은 `rcgen`이라는 Rust 크레이트를 사용하여 프로그래밍 방식으로 자체 서명 인증서와 개인 키를 동적으로 생성하는 방법을 공식 예제에서 보여준다. 이 방식은 외부 도구(예: `openssl`)에 대한 의존성 없이 순수 Rust 코드로 인증서 문제를 해결할 수 있어 매우 편리하다.

다음은 `rcgen`을 사용하여 `localhost`를 위한 인증서와 개인 키를 생성하는 함수다.

```Rust
// src/main.rs (일부)
use rustls::pki_types::{CertificateDer, PrivateKeyDer, PrivatePkcs8KeyDer};

/// rcgen을 사용하여 자체 서명 인증서를 생성한다.
fn generate_self_signed_cert() -> Result<(CertificateDer<'static>, PrivateKeyDer<'static>), Box<dyn std::error::Error>> {
    let cert = rcgen::generate_simple_self_signed(vec!["localhost".into()])?;
    let cert_der = CertificateDer::from(cert.cert);
    let key_der = PrivateKeyDer::Pkcs8(PrivatePkcs8KeyDer::from(cert.key_pair.serialize_der()));
    Ok((cert_der, key_der))
}
```

이 함수는 `rcgen::generate_simple_self_signed`를 호출하여 인증서를 생성하고, `quinn`이 요구하는 `rustls::pki_types`의 `CertificateDer`와 `PrivateKeyDer` 형식으로 변환하여 반환한다. 생성된 인증서와 키는 서버 시작 시 메모리에 직접 로드하여 사용하거나, `cert.der`, `key.der`와 같은 파일로 저장하여 재사용할 수 있다. 인증서를 파일로 저장하고 재사용하는 방식은 클라이언트가 특정 인증서를 기억하고 신뢰하는 Trust-on-First-Use (TOFU) 시나리오를 구현할 때 유용하다.

### 1.3  비동기 Echo 서버 구현

이제 `quinn`과 위에서 만든 자체 서명 인증서를 사용하여 간단한 Echo 서버를 구현한다. 이 서버는 클라이언트로부터 메시지를 받아 그대로 다시 돌려주는 역할을 한다.

먼저 `Cargo.toml`에 필요한 의존성을 추가한다.

```Ini, TOML
# Cargo.toml
[dependencies]
anyhow = "1.0"
quinn = "0.11"
rustls = "0.23"
rcgen = "0.13"
tokio = { version = "1", features = ["full"] }
tracing = "0.1"
tracing-subscriber = "0.3"
```

다음은 전체 서버 코드다.

```Rust
// src/main.rs

use anyhow::Result;
use quinn::{Endpoint, ServerConfig};
use std::sync::Arc;
use rustls::pki_types::{CertificateDer, PrivateKeyDer, PrivatePkcs8KeyDer};

#[tokio::main]
async fn main() -> Result<()> {
    // 1. 자체 서명 인증서 생성
    let (cert, key) = generate_self_signed_cert()?;
    let mut server_crypto = rustls::ServerConfig::builder()
       .with_no_client_auth()
       .with_single_cert(vec![cert], key)?;
    // ALPN 프로토콜 설정 (예: HTTP/0.9)
    server_crypto.alpn_protocols = vec![b"hq-29".to_vec()]; // 예시 프로토콜
    let server_config = ServerConfig::with_crypto(Arc::new(server_crypto));

    // 2. QUIC 엔드포인트 생성 및 바인딩
    let listen_addr = "127.0.0.1:4433".parse()?;
    let endpoint = Endpoint::server(server_config, listen_addr)?;
    println!("Listening on {}", endpoint.local_addr()?);

    // 3. 들어오는 연결 수락
    while let Some(connecting) = endpoint.accept().await {
        println!("Connection incoming");
        tokio::spawn(async move {
            match connecting.await {
                Ok(connection) => {
                    println!("Connection established");
                    // 4. 각 연결에 대한 스트림 처리
                    while let Ok((mut send, mut recv)) = connection.accept_bi().await {
                        let mut buf = vec![0; 1024];
                        if let Ok(Some(len)) = recv.read(&mut buf).await {
                            let received = &buf[..len];
                            println!("Received: {}", String::from_utf8_lossy(received));
                            // Echo back
                            send.write_all(received).await.unwrap();
                            send.finish().await.unwrap();
                        }
                    }
                }
                Err(e) => {
                    eprintln!("Connection failed: {}", e);
                }
            }
        });
    }

    Ok(())
}

/// rcgen을 사용하여 자체 서명 인증서를 생성한다.
fn generate_self_signed_cert() -> Result<(CertificateDer<'static>, PrivateKeyDer<'static>), Box<dyn std::error::Error>> {
    let cert = rcgen::generate_simple_self_signed(vec!["localhost".into()])?;
    let cert_der = CertificateDer::from(cert.cert);
    let key_der = PrivateKeyDer::Pkcs8(PrivatePkcs8KeyDer::from(cert.key_pair.serialize_der()));
    Ok((cert_der, key_der))
}
```

이 코드의 핵심 구성 요소는 다음과 같다.

1. **`ServerConfig` 구성**: `rustls::ServerConfig::builder()`를 사용하여 TLS 설정을 시작하고, `with_single_cert` 메소드를 통해 1.2절에서 생성한 인증서와 키를 제공한다. `alpn_protocols` 필드는 클라이언트와 서버가 QUIC 위에서 동작할 상위 애플리케이션 프로토콜(예: HTTP/3)을 협상하는 데 사용된다.
2. **`Endpoint` 생성**: `Endpoint::server` 함수는 구성된 `ServerConfig`와 수신 대기할 소켓 주소를 인자로 받아 QUIC 엔드포인트를 생성한다. 엔드포인트는 단일 UDP 소켓에 해당하며, 이 소켓을 통해 여러 클라이언트와의 다중 연결을 관리한다.
3. **연결 수락 루프**: `endpoint.accept().await`는 비동기적으로 클라이언트의 연결 시도를 기다린다. 새로운 연결 시도가 감지되면 `Connecting` 객체가 반환되며, 이를 `.await`하면 실제 `Connection` 객체를 얻을 수 있다.
4. **스트림 처리 및 동시성**: 각 `Connection`은 여러 개의 스트림을 가질 수 있다. `connection.accept_bi().await`를 통해 양방향(bidirectional) 스트림을 비동기적으로 수락한다. 새로운 연결이나 스트림이 생성될 때마다 `tokio::spawn`을 사용하여 별도의 비동기 태스크로 분리한다. 이 덕분에 서버는 하나의 연결을 처리하는 동안 다른 연결 요청을 블로킹하지 않고 동시에 여러 클라이언트를 효율적으로 처리할 수 있다.

## 2. 부: Android QUIC 클라이언트: 두 가지 경로와 올바른 선택

Rust 서버가 준비되었으니, 이제 Android 클라이언트를 구현할 차례다. Android에서 QUIC을 사용하는 방법은 크게 두 가지 경로로 나뉜다. 하지만 이 예제의 핵심 요구사항인 '자체 서명 인증서 사용'이라는 제약 조건 때문에, 한 가지 경로는 사실상 막다른 길이다.

### 2.1  경로 A: 표준 접근법, Cronet (그리고 그 한계)

Android에서 QUIC/HTTP3를 사용하는 가장 표준적인 방법은 **Cronet** 라이브러리를 이용하는 것이다. Cronet은 Google이 Chrome 브라우저에서 사용하는 강력한 네트워킹 스택을 안드로이드 앱에서 사용할 수 있도록 라이브러리 형태로 제공하는 것이다. 수많은 Google 앱(YouTube, Google 포토 등)에서 사용될 만큼 안정성과 성능이 검증되었다.

`CronetEngine.Builder`를 사용하여 `enableQuic(true)`를 호출하는 것만으로 QUIC 기능을 간단히 활성화할 수 있다.

```Kotlin
// Cronet 활성화 예시
val engineBuilder = CronetEngine.Builder(context)
   .enableQuic(true)
   .enableHttp2(true)
//... 기타 설정
val cronetEngine = engineBuilder.build()
```

일반적으로 Android 앱에서 자체 서명 인증서를 신뢰하도록 만드는 방법은 `res/xml/network_security_config.xml` 파일을 사용하는 것이다. 개발자는 이 파일에 커스텀 CA나 특정 도메인의 인증서를 신뢰하도록 명시하여, 로컬 개발 서버나 프록시를 통한 HTTPS 트래픽을 디버깅할 수 있다.

```XML
<network-security-config>
    <base-config>
        <trust-anchors>
            <certificates src="@raw/my_ca" />
            <certificates src="system" />
        </trust-anchors>
    </base-config>
</network-security-config>
```

논리적으로는 Cronet의 QUIC 기능과 네트워크 보안 구성을 조합하면 자체 서명 인증서를 사용하는 QUIC 서버와 통신할 수 있을 것처럼 보인다. **하지만 이는 불가능하다.**

Chromium 개발팀은 QUIC 프로토콜의 장기적인 발전을 저해할 수 있는 중간 장비(middlebox)의 QUIC 트래픽 가로채기 및 검사를 원천적으로 방지하기 위해, **Cronet의 QUIC 연결에 대해서는 공인되지 않은 신뢰 앵커(non-publicly-trusted certificates)를 의도적으로 허용하지 않는 정책**을 채택했다. 즉, `network_security_config.xml`에 자체 서명 인증서를 추가하더라도 Cronet은 QUIC 연결 시 이를 무시한다. 이 정책은 버그가 아니며, QUIC 프로토콜의 '동맥 경화(ossification)'를 막기 위한 확고한 설계 결정이다.

결과적으로, Cronet을 사용하여 우리가 1부에서 만든 자체 서명 인증서 기반의 QUIC 서버에 연결을 시도하면, 인증서 검증 단계에서 `ERR_QUIC_PROTOCOL_ERROR`와 유사한 오류가 발생하며 실패하게 된다. 따라서 경로 A는 이 가이드의 목표를 달성할 수 없는 막다른 길이다.

### 2.2  경로 B: 유연하고 강력한 접근법, Rust 네이티브 라이브러리 (JNI)

Cronet의 정책적 한계를 우회하고 자체 서명 인증서를 완벽하게 제어하는 가장 확실한 방법은 **QUIC 클라이언트 로직 전체를 Rust 네이티브 코드로 직접 구현하고, 이를 JNI(Java Native Interface)를 통해 Android 앱과 연동**하는 것이다.

이 접근법의 장점은 명확하다.

1. **완벽한 제어권**: Rust 코드 내에서 `quinn` 클라이언트의 TLS 설정을 직접 제어할 수 있다. 서버의 자체 서명 인증서(또는 해당 인증서를 서명한 CA)를 클라이언트가 명시적으로 신뢰하도록 구성하는 것이 가능하다.
2. **일관된 코드베이스**: 서버와 클라이언트 간에 공유되는 데이터 모델이나 커스텀 프로토콜 로직이 있다면, 이를 Rust로 작성하여 양쪽에서 모두 재사용할 수 있다.
3. **성능**: JNI 호출에 따른 약간의 오버헤드는 존재하지만, 핵심 네트워킹 로직은 C/C++과 동등한 네이티브 속도로 실행되므로 높은 성능을 기대할 수 있다.

`quinn` 클라이언트에서 자체 서명 인증서를 신뢰하도록 설정하는 방법은 다음과 같다. 먼저 `rustls`의 `dangerous_configuration` 기능을 `Cargo.toml`에서 활성화해야 한다. 이 기능은 이름에서 알 수 있듯이 프로덕션 환경에서는 신중하게 사용해야 하지만, 개발 및 테스트 단계에서 인증서 검증 로직을 커스터마이징할 수 있는 유연성을 제공한다.

그 후, 서버의 인증서 파일(`cert.der`)을 Android 앱의 자산(asset)으로 포함시키고, JNI를 통해 이 파일을 읽어 `rustls::RootCertStore`에 추가하면 된다. `quinn` 클라이언트 예제는 로컬 디렉토리에서 인증서 파일을 읽어 신뢰 목록에 추가하는 방식을 보여준다.

```Rust
// Rust 클라이언트에서 커스텀 CA를 신뢰하도록 설정하는 예시
use std::fs;
use quinn::ClientConfig;
use rustls::pki_types::CertificateDer;

//...
let mut roots = rustls::RootCertStore::empty();
// 안드로이드 앱의 asset에서 읽어온 인증서 데이터를 추가
let cert_der = CertificateDer::from(ca_bytes_from_android);
roots.add(cert_der)?;

let client_crypto = rustls::ClientConfig::builder()
   .with_root_certificates(roots)
   .with_no_client_auth();

let client_config = ClientConfig::new(Arc::new(client_crypto));
//... 이 config를 사용하여 endpoint.connect_with 호출
```

이처럼 JNI를 통한 네이티브 구현은 Cronet의 한계를 완벽하게 극복하고, 우리의 요구사항을 만족시키는 유일하고 강력한 해결책이다.

| 측면                      | 경로 A: Cronet                | 경로 B: JNI + Native Rust | 선택 근거                                                    |
| ------------------------- | ----------------------------- | ------------------------- | ------------------------------------------------------------ |
| **자체 서명 인증서 지원** | **불가능 (정책적 제한)**      | **가능 (완벽 제어)**      | JNI 접근법은 이 예제의 핵심 요구사항을 만족시키는 유일한 방법이다. |
| **TLS 설정 제어**         | 제한적                        | 완전한 제어               | Rust 코드 내에서 `rustls`를 통해 모든 TLS 파라미터를 직접 제어할 수 있다. |
| **코드 재사용성**         | 불가능                        | 높음 (서버와 공유 가능)   | 데이터 모델, 프로토콜 로직 등을 Rust로 작성하여 서버와 클라이언트에서 공유할 수 있다. |
| **구현 복잡도**           | 낮음                          | 높음                      | JNI 브릿지, 비동기 런타임 관리 등 추가적인 복잡성이 수반된다. |
| **성능**                  | 높음 (최적화된 네이티브 스택) | 높음 (네이티브 실행)      | JNI 오버헤드가 있지만, 핵심 로직은 네이티브 속도로 실행되어 성능 저하가 미미하다. |

## 3. 부: JNI를 통한 Rust와 Android 연결

경로 B, 즉 JNI를 통해 Rust 네이티브 라이브러리를 Android 앱에 통합하는 방법을 구체적으로 살펴본다. 이 과정은 환경 설정, 네이티브 라이브러리 빌드, JNI 인터페이스 설계, 그리고 가장 중요한 비동기 런타임 관리의 네 단계로 구성된다.

### 3.1  통합 프로젝트 환경 설정

먼저 Android Studio 프로젝트 내에 Rust 라이브러리 크레이트를 포함하는 통합 개발 환경을 구성해야 한다.

#### 3.1.1 디렉토리 구조

일반적으로 Android Studio 프로젝트의 `app/src/main/` 디렉토리 아래에 `rust/` 또는 `rust-client/`와 같은 이름으로 Rust 프로젝트를 위치시킨다.

```
MyAndroidApp/
├── app/
│   ├── build.gradle
│   └── src/
│       └── main/
│           ├── java/
│           │   └── com/example/myapp/
│           │       ├── MainActivity.kt
│           │       └── QuicClient.kt
│           ├── jniLibs/  <-- 빌드된.so 파일이 위치할 곳
│           │   ├── arm64-v8a/
│           │   │   └── librust_client.so
│           │   └──... (다른 ABI)
│           └── rust-client/  <-- Rust 라이브러리 프로젝트
│               ├── Cargo.toml
│               └── src/
│                   └── lib.rs
└──...
```

#### 3.1.2 `Cargo.toml` 설정

Rust 라이브러리가 Android에서 동적 공유 라이브러리(`.so` 파일)로 로드될 수 있도록 `Cargo.toml`을 설정한다.

```Ini, TOML
# MyAndroidApp/app/src/main/rust-client/Cargo.toml

[package]
name = "rust-client"
version = "0.1.0"
edition = "2021"

[lib]
name = "rust_client" #.so 파일의 이름이 librust_client.so가 됨
crate-type = ["cdylib"] # 동적 공유 라이브러리 생성

[dependencies]
# QUIC 및 비동기 런타임
quinn = "0.11"
tokio = { version = "1", features = ["full"] }
rustls = { version = "0.23", features = ["dangerous_configuration"] }

# JNI 연동
jni = "0.21"

# 기타 유틸리티
anyhow = "1.0"
log = "0.4"
android_logger = "0.13" # Android logcat으로 로깅
once_cell = "1.19" # 전역 런타임 관리를 위해
```

- `crate-type = ["cdylib"]`: C 호환 동적 라이브러리를 생성하도록 Rust 컴파일러에 지시한다.
- `jni`: Rust와 Java 간의 상호작용을 위한 핵심 바인딩을 제공한다.
- `once_cell`: 스레드로부터 안전한 방식으로 전역 변수(예: Tokio 런타임)를 한 번만 초기화하는 데 사용된다.

#### 3.1.3 `build.gradle` (모듈 수준) 설정

Gradle이 빌드 시 생성된 `.so` 파일을 APK에 올바르게 포함하도록 설정해야 한다. `sourceSets` 블록을 사용하여 `jniLibs` 디렉토리의 위치를 명시하는 것이 가장 간단한 방법이다.

```Groovy
// MyAndroidApp/app/build.gradle

android {
    //...
    sourceSets {
        main {
            jniLibs.srcDirs = ['src/main/jniLibs']
        }
    }
}
```

빌드 스크립트를 통해 Rust 코드를 자동으로 빌드하고 `.so` 파일을 `jniLibs`로 복사하는 작업을 추가할 수도 있지만, 이 가이드에서는 수동 빌드 및 복사를 전제로 한다.

### 3.2  네이티브 라이브러리 빌드

Rust 코드를 Android의 다양한 하드웨어 아키텍처(ABI)에 맞게 크로스 컴파일해야 한다.

1. **Android NDK 설치**: Android Studio의 `SDK Manager` > `SDK Tools` 탭에서 `NDK (Side by side)`를 설치한다. 그리고 `ANDROID_NDK_HOME` 환경 변수를 설치된 NDK 경로로 설정한다.

2. **Rust 타겟 추가**: `rustup`을 사용하여 Android ABI에 대한 컴파일 타겟을 설치한다.

   ```Bash
   rustup target add aarch64-linux-android armv7-linux-androideabi i686-linux-android x86_64-linux-android
   ```
   
3. **`cargo-ndk` 사용**: `cargo-ndk`는 Android NDK와의 연동을 자동화하여 크로스 컴파일 과정을 매우 간단하게 만들어주는 필수 도구다.

   ```Bash
   cargo install cargo-ndk
   ```
   
4. **빌드 실행**: Rust 프로젝트 디렉토리에서 `cargo ndk build` 명령을 실행하여 각 ABI에 맞는 `.so` 파일을 생성한다.

   ```Bash
   # 예: arm64-v8a 아키텍처, API 레벨 21 타겟으로 릴리즈 빌드
   cargo ndk -t arm64-v8a -p 21 build --release
   ```
   
   빌드가 완료되면 `target/<ABI>/release/` 디렉토리에 `librust_client.so` 파일이 생성된다. 이 파일을 `app/src/main/jniLibs/<ABI>/` 디렉토리로 복사한다.

### 3.3  JNI 인터페이스 설계 및 구현

JNI는 Java(또는 Kotlin) 코드와 네이티브 코드 간의 다리 역할을 한다. Rust 함수를 Kotlin에서 호출할 수 있도록 JNI 인터페이스를 설계하고 구현해야 한다.

#### 3.3.1 Rust 함수 노출

JNI 규칙에 따라 Rust 함수를 작성해야 한다.

- `#[no_mangle]` 어트리뷰트로 함수 이름이 변경되지 않도록 한다.
- `extern "system" fn` 또는 `extern "C" fn`으로 C 호출 규약을 따르도록 한다.
- 함수 이름은 `Java_{패키지}_{클래스}_{메소드}` 형식이어야 한다. 패키지의 `.`은 `_`로, 클래스 이름의 `_`는 `_1`로 변환된다.

```Rust
// rust-client/src/lib.rs (일부)
use jni::JNIEnv;
use jni::objects::{JClass, JString};
use jni::sys::jstring;

#[no_mangle]
pub extern "system" fn Java_com_example_myapp_QuicClient_nativeInit(
    env: JNIEnv,
    _class: JClass,
) {
    //... 초기화 로직
}

#[no_mangle]
pub extern "system" fn Java_com_example_myapp_QuicClient_sayHello<'local>(
    mut env: JNIEnv<'local>,
    _class: JClass<'local>,
    input: JString<'local>,
) -> jstring {
    // Java 문자열을 Rust 문자열로 변환
    let input_str: String = env.get_string(&input).expect("Couldn't get java string!").into();

    // 새로운 Rust 문자열 생성
    let output_str = format!("Hello from Rust, {}!", input_str);

    // Rust 문자열을 Java 문자열로 변환하여 반환
    let output = env.new_string(output_str).expect("Couldn't create java string!");

    // 소유권을 Java 측으로 이전
    output.into_raw()
}
```

### 3.4  비동기 런타임 관리: 가장 중요한 난관

Android 앱의 UI 스레드나 일반적인 콜백 내에서 JNI 함수를 호출하고, 그 함수 안에서 `tokio::runtime::...block_on()`을 호출하여 비동기 작업을 동기적으로 기다리는 것은 심각한 문제를 야기한다. 만약 외부 호출 환경이 이미 다른 Tokio 런타임에 의해 관리되는 스레드라면, "Cannot start a runtime from within a runtime"이라는 메시지와 함께 앱이 패닉으로 종료된다. Android 프레임워크나 다른 서드파티 라이브러리가 내부적으로 비동기 런타임을 사용할 수 있으므로 이 문제는 언제든 발생할 수 있다.

이 문제를 해결하는 가장 안정적이고 견고한 패턴은 **애플리케이션의 생명주기와 독립적으로 동작하는 전용 백그라운드 스레드에서 단일 Tokio 런타임을 생성하고, 통신 채널을 통해 작업을 주고받는 것**이다. 이 패턴은 `rousan/AndroidWithRust`나 `jni-utils`와 같은 프로젝트에서 암시하는 구조와 일치한다.

#### 3.4.1 올바른 구현 패턴

1. **전역 런타임 생성 및 관리**: `once_cell`을 사용하여 앱 전체에서 단 한 번만 초기화되는 전역 Tokio 런타임 핸들을 관리한다. `nativeInit` JNI 함수가 호출될 때, `std::thread::spawn`을 사용해 새로운 OS 스레드를 생성하고, 그 안에서 `tokio::runtime::Runtime::new()`로 런타임을 구축한다.
2. **MPSC 채널을 통한 비동기 통신**: `tokio::sync::mpsc` (Multi-Producer, Single-Consumer) 채널을 사용하여 Kotlin 스레드(Producer)와 Rust의 Tokio 런타임 스레드(Consumer) 간에 명령(Command)과 데이터를 안전하게 주고받는다.
3. **JNI 함수의 역할 분리**: `connect`, `sendMessage`와 같은 JNI 함수들은 실제 작업을 수행하는 대신, 작업에 필요한 정보를 담은 `Command` 열거형을 만들어 MPSC 채널의 `Sender`를 통해 전송하는 역할만 한다. 이 함수들은 즉시 반환되므로 UI 스레드를 전혀 블로킹하지 않는다.
4. **백그라운드 작업 루프**: Tokio 런타임 스레드에서는 MPSC 채널의 `Receiver`를 `await`하며 명령을 기다린다. 명령을 받으면 해당 비동기 작업(예: `quinn` 연결, 데이터 전송)을 `tokio::spawn`으로 실행한다.
5. **결과 반환 (콜백)**: Rust에서 작업이 완료되면 그 결과를 Kotlin으로 알려줘야 한다. 이를 위해 Kotlin에서 콜백 인터페이스를 정의하고, 해당 객체를 Rust로 전달한다. Rust에서는 `JNIEnv::new_global_ref`를 사용하여 콜백 객체에 대한 전역 참조를 만들고, `JavaVM` 포인터를 저장해 둔다. Tokio 런타임 스레드에서 콜백을 호출해야 할 때, 저장된 `JavaVM`을 통해 현재 스레드를 JVM에 연결(`attach_current_thread`)하여 `JNIEnv`를 얻고, 전역 참조를 통해 Kotlin 객체의 메소드를 호출한다.

이 아키텍처는 Android의 생명주기와 Rust의 비동기 런타임을 안전하게 분리하여, 예측 가능하고 안정적인 동작을 보장하는 핵심적인 설계다.

## 4. 부: 전체 통합 예제 코드

앞서 설명한 아키텍처를 바탕으로 한 완전한 통합 예제 코드를 제시한다. 각 파일에는 상세한 주석을 달아 코드의 흐름을 쉽게 이해할 수 있도록 했다.

### 4.1  Rust QUIC 서버 (`server/src/main.rs`)

1부에서 설명한 Echo 서버의 최종 코드다. `cargo run`으로 실행하여 클라이언트의 연결을 기다린다.

```Rust
// server/src/main.rs

use anyhow::Result;
use quinn::{Endpoint, ServerConfig};
use std::sync::Arc;
use rustls::pki_types::{CertificateDer, PrivateKeyDer, PrivatePkcs8KeyDer};
use tracing_subscriber::EnvFilter;

#[tokio::main]
async fn main() -> Result<()> {
    tracing_subscriber::fmt()
       .with_env_filter(EnvFilter::from_default_env())
       .init();

    // 1. 자체 서명 인증서 생성
    let (cert, key) = generate_self_signed_cert()?;
    let mut server_crypto = rustls::ServerConfig::builder()
       .with_no_client_auth()
       .with_single_cert(vec![cert], key)?;
    // ALPN 프로토콜은 클라이언트와 서버가 동일하게 맞춰야 한다.
    server_crypto.alpn_protocols = vec![b"echo-proto".to_vec()];
    let server_config = ServerConfig::with_crypto(Arc::new(server_crypto));

    // 2. QUIC 엔드포인트 생성 및 바인딩
    let listen_addr = "0.0.0.0:4433".parse()?; // 모든 인터페이스에서 수신
    let endpoint = Endpoint::server(server_config, listen_addr)?;
    println!(" Listening on {}", endpoint.local_addr()?);

    // 3. 들어오는 연결 수락
    while let Some(connecting) = endpoint.accept().await {
        println!(" Connection incoming");
        tokio::spawn(async move {
            match connecting.await {
                Ok(connection) => {
                    println!(" Connection established from: {}", connection.remote_address());
                    // 4. 각 연결에 대한 스트림 처리
                    loop {
                        match connection.accept_bi().await {
                            Ok((mut send, mut recv)) => {
                                tokio::spawn(async move {
                                    let mut buf = vec![0; 2048];
                                    while let Ok(Some(len)) = recv.read(&mut buf).await {
                                        let received = &buf[..len];
                                        println!(" Received: {}", String::from_utf8_lossy(received));
                                        // Echo back
                                        if let Err(e) = send.write_all(received).await {
                                            eprintln!(" Write error: {}", e);
                                            break;
                                        }
                                    }
                                });
                            }
                            Err(quinn::ConnectionError::ApplicationClosed {.. }) => {
                                println!(" Connection closed by application");
                                break;
                            }
                            Err(e) => {
                                eprintln!(" Stream accept error: {}", e);
                                break;
                            }
                        }
                    }
                }
                Err(e) => {
                    eprintln!(" Connection failed: {}", e);
                }
            }
        });
    }

    Ok(())
}

fn generate_self_signed_cert() -> Result<(CertificateDer<'static>, PrivateKeyDer<'static>), Box<dyn std::error::Error>> {
    let cert = rcgen::generate_simple_self_signed(vec!["localhost".into()])?;
    let cert_der = CertificateDer::from(cert.cert);
    let key_der = PrivateKeyDer::Pkcs8(PrivatePkcs8KeyDer::from(cert.key_pair.serialize_der()));
    Ok((cert_der, key_der))
}
```

### 4.2  Rust JNI 클라이언트 라이브러리 (`android/rust-client/src/lib.rs`)

3.4절에서 설명한 전역 Tokio 런타임과 MPSC 채널 패턴을 구현한 JNI 라이브러리다.

```Rust
// android/rust-client/src/lib.rs

use jni::objects::{JClass, JObject, JString, GlobalRef};
use jni::sys::{jint, jstring};
use jni::{JNIEnv, JavaVM};
use once_cell::sync::OnceCell;
use std::sync::{Arc, Mutex};
use tokio::runtime::Runtime;
use tokio::sync::mpsc;

use quinn::{ClientConfig, Endpoint};
use rustls::pki_types::CertificateDer;

// 전역 Tokio 런타임과 통신 채널을 관리하기 위한 구조체
static TOKIO_RUNTIME: OnceCell<Runtime> = OnceCell::new();
static COMMAND_SENDER: OnceCell<mpsc::Sender<Command>> = OnceCell::new();
static JVM: OnceCell<JavaVM> = OnceCell::new();
static CALLBACK_HANDLER: OnceCell<Mutex<GlobalRef>> = OnceCell::new();

// Kotlin과 Rust 간의 통신을 위한 명령
enum Command {
    Connect {
        addr: String,
        port: u16,
        ca_cert_der: Vec<u8>,
    },
    SendMessage(String),
    Shutdown,
}

// JNI 초기화 함수. 앱 시작 시 한 번만 호출되어야 한다.
#[no_mangle]
pub extern "system" fn Java_com_example_myapp_QuicClient_nativeInit<'local>(
    mut env: JNIEnv<'local>,
    _class: JClass<'local>,
    callback: JObject<'local>,
) {
    android_logger::init_once(
        android_logger::Config::default().with_min_level(log::Level::Info),
    );

    let jvm = env.get_java_vm().unwrap();
    JVM.set(jvm).unwrap();

    let callback_ref = env.new_global_ref(callback).unwrap();
    CALLBACK_HANDLER.set(Mutex::new(callback_ref)).unwrap();

    let (tx, mut rx) = mpsc::channel(10);
    COMMAND_SENDER.set(tx).unwrap();

    let runtime = tokio::runtime::Builder::new_multi_thread()
       .enable_all()
       .build()
       .unwrap();

    std::thread::spawn(move |

| {
        runtime.block_on(async move {
            log::info!(" Tokio runtime started");
            let mut connection_sender: Option<mpsc::Sender<String>> = None;

            while let Some(command) = rx.recv().await {
                match command {
                    Command::Connect { addr, port, ca_cert_der } => {
                        let (conn_tx, conn_rx) = mpsc::channel(10);
                        connection_sender = Some(conn_tx);
                        tokio::spawn(run_quic_client(addr, port, ca_cert_der, conn_rx));
                    }
                    Command::SendMessage(msg) => {
                        if let Some(sender) = &connection_sender {
                            if sender.send(msg).await.is_err() {
                                log::error!(" Failed to send message to connection task");
                            }
                        }
                    }
                    Command::Shutdown => {
                        log::info!(" Shutdown command received");
                        break;
                    }
                }
            }
        });
    });
}

// QUIC 클라이언트 로직을 실행하는 비동기 함수
async fn run_quic_client(addr: String, port: u16, ca_cert_der: Vec<u8>, mut msg_rx: mpsc::Receiver<String>) {
    let server_addr = format!("{}:{}", addr, port).parse().unwrap();
    
    // 자체 서명 CA 신뢰 설정
    let mut roots = rustls::RootCertStore::empty();
    roots.add(CertificateDer::from(ca_cert_der)).unwrap();
    let client_crypto = rustls::ClientConfig::builder()
       .with_root_certificates(roots)
       .with_no_client_auth();
    let client_config = ClientConfig::new(Arc::new(client_crypto));

    let mut endpoint = Endpoint::client("0.0.0.0:0".parse().unwrap()).unwrap();
    endpoint.set_default_client_config(client_config);

    log::info!(" Connecting to {}:{}", addr, port);
    let connection = match endpoint.connect(server_addr, "localhost").unwrap().await {
        Ok(conn) => {
            on_event("ConnectionEstablished".to_string());
            conn
        },
        Err(e) => {
            log::error!(" Connection error: {}", e);
            on_event(format!("Error: Connection failed: {}", e));
            return;
        }
    };

    let (mut send, mut recv) = connection.open_bi().await.unwrap();
    on_event("StreamOpened".to_string());

    // 메시지 수신 루프
    tokio::spawn(async move {
        let mut buf = vec![0; 2048];
        while let Ok(Some(len)) = recv.read(&mut buf).await {
            let msg = String::from_utf8_lossy(&buf[..len]).to_string();
            log::info!(" Received from server: {}", msg);
            on_event(format!("MessageReceived:{}", msg));
        }
        on_event("ReceiveStreamClosed".to_string());
    });

    // 메시지 전송 루프
    while let Some(msg) = msg_rx.recv().await {
        log::info!(" Sending to server: {}", msg);
        if let Err(e) = send.write_all(msg.as_bytes()).await {
            log::error!(" Send error: {}", e);
            on_event(format!("Error: Send failed: {}", e));
            break;
        }
    }
}

// Kotlin 콜백을 호출하는 헬퍼 함수
fn on_event(event_str: String) {
    if let (Some(jvm), Some(callback_handler)) = (JVM.get(), CALLBACK_HANDLER.get()) {
        let mut env = jvm.attach_current_thread().unwrap();
        let callback = callback_handler.lock().unwrap();
        let event_jstring = env.new_string(&event_str).unwrap();
        
        env.call_method(
            callback.as_obj(),
            "onRustEvent",
            "(Ljava/lang/String;)V",
            &[(&event_jstring).into()],
        ).unwrap();
    }
}

#[no_mangle]
pub extern "system" fn Java_com_example_myapp_QuicClient_nativeConnect<'local>(
    mut env: JNIEnv<'local>,
    _class: JClass<'local>,
    addr: JString<'local>,
    port: jint,
    ca_cert: jni::sys::jbyteArray,
) {
    let addr_str: String = env.get_string(&addr).unwrap().into();
    let ca_cert_der = env.convert_byte_array(ca_cert).unwrap();
    if let Some(sender) = COMMAND_SENDER.get() {
        let cmd = Command::Connect { addr: addr_str, port: port as u16, ca_cert_der };
        sender.try_send(cmd).unwrap();
    }
}

#[no_mangle]
pub extern "system" fn Java_com_example_myapp_QuicClient_nativeSendMessage<'local>(
    mut env: JNIEnv<'local>,
    _class: JClass<'local>,
    message: JString<'local>,
) {
    let msg_str: String = env.get_string(&message).unwrap().into();
    if let Some(sender) = COMMAND_SENDER.get() {
        sender.try_send(Command::SendMessage(msg_str)).unwrap();
    }
}

#[no_mangle]
pub extern "system" fn Java_com_example_myapp_QuicClient_nativeShutdown(_env: JNIEnv, _class: JClass) {
    if let Some(sender) = COMMAND_SENDER.get() {
        sender.try_send(Command::Shutdown).unwrap();
    }
}
```

### 4.3  Kotlin JNI 래퍼 클래스 (`QuicClient.kt`)

Rust JNI 함수들을 감싸는 Kotlin 싱글톤 객체다. 네이티브 메소드를 선언하고, 서버의 인증서를 로드하는 로직을 포함한다.

```Kotlin
// android/app/src/main/java/com/example/myapp/QuicClient.kt

package com.example.myapp

import android.content.Context
import java.io.InputStream

// Rust 라이브러리와 상호작용하는 콜백 인터페이스
interface RustEventListener {
    fun onRustEvent(event: String)
}

object QuicClient {
    private var listener: RustEventListener? = null
    private lateinit var caCertBytes: ByteArray

    init {
        System.loadLibrary("rust_client")
    }

    fun initialize(context: Context, listener: RustEventListener) {
        this.listener = listener
        // 서버의 자체 서명 인증서를 raw 리소스로부터 로드
        // 이 예제에서는 서버가 생성한 cert.der 파일을 cert.crt로 이름을 바꿔 raw에 추가했다고 가정
        val inputStream: InputStream = context.resources.openRawResource(R.raw.cert)
        caCertBytes = inputStream.readBytes()
        inputStream.close()
        
        nativeInit(object : RustEventCallback {
            override fun onRustEvent(event: String) {
                this@QuicClient.listener?.onRustEvent(event)
            }
        })
    }

    fun connect(addr: String, port: Int) {
        nativeConnect(addr, port, caCertBytes)
    }

    fun sendMessage(message: String) {
        nativeSendMessage(message)
    }

    fun shutdown() {
        nativeShutdown()
    }

    // Rust에서 호출할 콜백 인터페이스
    private interface RustEventCallback {
        fun onRustEvent(event: String)
    }

    // JNI 함수 선언
    private external fun nativeInit(callback: RustEventCallback)
    private external fun nativeConnect(addr: String, port: Int, caCert: ByteArray)
    private external fun nativeSendMessage(message: String)
    private external fun nativeShutdown()
}
```

### 4.4  Android UI 및 클라이언트 로직 (`MainActivity.kt`)

앱의 메인 화면으로, `QuicClient`를 초기화하고 사용자 입력을 받아 Rust로 전달하며, Rust로부터 받은 이벤트를 화면에 표시한다.

```Kotlin
// android/app/src/main/java/com/example/myapp/MainActivity.kt

package com.example.myapp

import android.os.Bundle
import android.widget.Button
import android.widget.EditText
import android.widget.TextView
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity(), RustEventListener {

    private lateinit var statusTextView: TextView
    private lateinit var messageLogTextView: TextView
    private lateinit var serverIpEditText: EditText
    private lateinit var messageEditText: EditText

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        statusTextView = findViewById(R.id.statusTextView)
        messageLogTextView = findViewById(R.id.messageLogTextView)
        serverIpEditText = findViewById(R.id.serverIpEditText)
        messageEditText = findViewById(R.id.messageEditText)
        val connectButton: Button = findViewById(R.id.connectButton)
        val sendButton: Button = findViewById(R.id.sendButton)

        // QuicClient 초기화
        QuicClient.initialize(this, this)

        connectButton.setOnClickListener {
            val ip = serverIpEditText.text.toString()
            // 로컬 개발 환경에서는 PC의 IP 주소를 사용
            QuicClient.connect(ip, 4433)
            statusTextView.text = "Connecting..."
        }

        sendButton.setOnClickListener {
            val message = messageEditText.text.toString()
            if (message.isNotEmpty()) {
                QuicClient.sendMessage(message)
                messageEditText.text.clear()
            }
        }
    }

    override fun onRustEvent(event: String) {
        runOnUiThread {
            when {
                event.startsWith("MessageReceived:") -> {
                    val msg = event.substringAfter("MessageReceived:")
                    messageLogTextView.append("Server: $msg\n")
                }
                event.startsWith("Error:") -> {
                    val error = event.substringAfter("Error:")
                    statusTextView.text = "Error: $error"
                }
                else -> {
                    statusTextView.text = "Status: $event"
                }
            }
        }
    }

    override fun onDestroy() {
        super.onDestroy()
        QuicClient.shutdown()
    }
}
```

### 4.5  빌드 스크립트

위 코드들이 올바르게 빌드되고 패키징되기 위한 `Cargo.toml`과 `build.gradle` 파일의 전체 내용이다.

**`Cargo.toml`**:

```Ini, TOML
# android/rust-client/Cargo.toml
[package]
name = "rust-client"
version = "0.1.0"
edition = "2021"

[lib]
name = "rust_client"
crate-type = ["cdylib"]

[dependencies]
quinn = "0.11"
tokio = { version = "1", features = ["full"] }
rustls = { version = "0.23", features = ["dangerous_configuration"] }
jni = "0.21"
anyhow = "1.0"
log = "0.4"
android_logger = "0.13"
once_cell = "1.19"
```

**`app/build.gradle`**:

```Groovy
// android/app/build.gradle
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
}

android {
    namespace 'com.example.myapp'
    compileSdk 34

    defaultConfig {
        applicationId "com.example.myapp"
        minSdk 24
        targetSdk 34
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    kotlinOptions {
        jvmTarget = '1.8'
    }
    sourceSets {
        main {
            // 빌드된.so 파일이 위치할 디렉토리를 Gradle에 알려줌
            jniLibs.srcDirs = ['src/main/jniLibs']
        }
    }
    // 레이아웃 파일은 생략
}

dependencies {
    implementation 'androidx.core:core-ktx:1.12.0'
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'com.google.android.material:material:1.11.0'
    //...
}
```

## 5. 결론 및 권장 사항

이 보고서는 Rust로 작성된 QUIC 서버와 Android 클라이언트 간에 자체 서명 인증서를 사용하여 TLS 보안 통신을 구현하는 전체 과정을 상세히 다루었다.

### 핵심 요약

분석 결과, **Cronet을 사용하여 자체 서명 인증서 기반의 QUIC 서버와 통신하는 것은 Chromium의 정책적 한계로 인해 사실상 불가능하다**는 결론에 도달했다. 따라서 이 문제에 대한 가장 현실적이고 강력한 해결책은 **네이티브 Rust 라이브러리를 JNI를 통해 Android 앱에 연동**하는 것이다. 이 접근법은 인증서 관리와 TLS 설정에 대한 완전한 제어권을 제공하지만, JNI 브릿지 설계와 특히 Android 생명주기 내에서 Rust의 비동기 런타임을 안전하게 관리하는 것에 대한 깊은 이해를 요구한다. 본문에서 제시한 전역 Tokio 런타임과 MPSC 채널을 이용한 아키텍처는 이러한 복잡성을 해결하는 안정적인 패턴이다.

### 5.1 프로덕션 환경을 위한 권장 사항

- **인증서**: 개발 단계에서는 자체 서명 인증서가 유용하지만, 프로덕션 환경에서는 반드시 **Let's Encrypt와 같은 공개적으로 신뢰받는 CA에서 발급한 정식 인증서**를 사용해야 한다. 이 경우, 클라이언트 측에서는 별도의 커스텀 CA 신뢰 로직 없이 표준 `rustls` 설정을 사용하면 된다.
- **아키텍처 트레이드오프**: JNI를 통한 네이티브 라이브러리 통합은 앱의 빌드 과정과 코드베이스의 복잡성을 증가시킨다. QUIC이 제공하는 지연 시간 감소, HOL 블로킹 해결 등의 이점이 이러한 추가적인 복잡성을 상쇄할 만큼 충분히 큰지 신중하게 평가해야 한다. 실시간 비디오 스트리밍, 멀티플레이어 게임, 지연 시간에 매우 민감한 금융 거래 앱 등 특정 도메인에서는 그 가치가 충분하지만, 일반적인 정보 조회 앱에서는 과도한 설계일 수 있다.

### 5.2 향후 개선 방향

- **`uniffi-rs` 활용**: Mozilla에서 개발한 `uniffi-rs`는 UDL(UniFFI Definition Language)이라는 인터페이스 정의 언어를 사용하여 JNI(Kotlin), Swift, Python 등 여러 언어의 바인딩 코드를 자동으로 생성해주는 강력한 도구다. 이를 도입하면 수동으로 작성해야 했던 JNI 보일러플레이트 코드를 크게 줄여 개발 생산성과 유지보수성을 획기적으로 향상시킬 수 있다.
- **`quiche` FFI 고려**: `cloudflare/quiche` 라이브러리는 `quinn`보다 저수준의 C FFI를 제공한다. 이는 UDP 소켓 관리, 타이머를 포함한 이벤트 루프를 개발자가 직접 구현해야 하는 부담이 있지만, 기존 C/C++ 코드베이스와의 통합이 필요하거나 극도의 성능 최적화가 요구되는 시나리오에서는 유용한 대안이 될 수 있다.