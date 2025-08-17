# 상업용 드론 관제 시스템의 안드로이드 클라이언트를 위한 QUIC 프로토콜 도입 기술 검토

## 1. Executive Summary & Key Recommendations

### 0.1  최종 결론

본 보고서는 상업용 드론 관제 시스템의 안드로이드 클라이언트와 서버 간 보안 데이터그램 전송을 위한 통신 프로토콜로 QUIC을 채택할 것을 강력히 권고한다. 기술적 분석 결과, QUIC은 기존의 TCP+TLS 스택 대비 상업용 드론, 특히 비가시권(BVLOS) 운용 환경에서 요구되는 통신 안정성, 저지연성, 보안성을 월등히 높은 수준으로 만족시킨다. QUIC의 도입은 단순한 성능 개선을 넘어, 시스템의 근본적인 신뢰성과 운용 가능성을 제고하는 전략적 결정이 될 것이다.

### 0.2  핵심 근거

QUIC 채택 권고의 핵심 근거는 다음과 같다.

1. **Head-of-Line (HOL) 블로킹 제거:** QUIC은 단일 연결 내에서 독립적인 다중 스트림을 지원하여, 하나의 스트림에서 발생한 패킷 손실이 다른 스트림의 데이터 전송을 지연시키는 문제를 원천적으로 해결한다.1 이는 고대역폭의 영상 스트림 패킷 손실이 치명적인 저지연 제어(C2) 명령 패킷의 전달을 방해하는 최악의 시나리오를 방지하여, 드론 제어의 안정성을 극대화한다.
2. **끊김 없는 연결 마이그레이션(Connection Migration):** QUIC 연결은 IP 주소와 포트로 식별되는 TCP와 달리, 고유한 연결 ID(Connection ID)로 식별된다.3 이로 인해 드론 관제 클라이언트(안드로이드)가 Wi-Fi와 셀룰러(5G/LTE) 네트워크 간을 전환할 때 IP 주소가 변경되어도 연결이 끊기지 않고 원활하게 유지된다. 이는 BVLOS 운용의 필수 전제조건인 지속적인 연결성을 보장하는 핵심 기능이다.
3. **저지연 연결 설정 (0-RTT/1-RTT):** QUIC은 전송 계층과 암호화 계층의 핸드셰이크를 통합하여, 새로운 연결을 단 한 번의 왕복 시간(1-RTT) 만에 설정할 수 있다.5 이전에 연결했던 서버와는 0-RTT로 즉시 데이터 전송이 가능하여, 일시적인 통신 두절 후 제어 링크를 복구하는 시간을 극적으로 단축시킨다.

### 0.3  권장 구현 경로

최적의 구현을 위해 다음 경로를 제안한다.

- **안드로이드 라이브러리:** **Google Cronet (Google Play Services 경유)** 채택을 권장한다. Cronet은 Google의 공식 지원을 받으며, 수십억 사용자를 통해 검증된 Chromium 네트워크 스택을 기반으로 하여 안정성과 보안성이 매우 높다.7 Play Services를 통한 배포는 앱의 용량을 줄이고, 최신 보안 패치 및 성능 개선을 자동으로 적용받는 이점이 있다.9
- **필수 안전장치:** 클라이언트 측에 **TCP로의 자동 폴백(Fallback) 메커니즘**을 반드시 구현해야 한다. 일부 제한적인 네트워크 환경(기업 방화벽, 통신사 정책 등)에서는 UDP 기반의 QUIC 트래픽이 차단될 수 있으므로 10, 연결 실패 시 투명하게 TCP+TLS 기반 통신으로 전환하여 드론의 제어 가능성을 상시 보장해야 한다.

### 1.4. 주요 리스크 및 완화 방안 요약

QUIC 도입 시 예상되는 주요 리스크와 그에 대한 완화 방안은 다음과 같다.

- **리스크 1: 안드로이드 클라이언트의 CPU 및 배터리 소모 증가:** QUIC은 커널이 아닌 사용자 공간(User-space)에서 처리되므로, 최적화된 TCP 스택에 비해 CPU 점유율과 배터리 소모가 높을 수 있다.12
  - **완화 방안:** 고도로 최적화된 Google Cronet 라이브러리를 사용하고, 개발 단계에서 철저한 리소스 프로파일링을 통해 애플리케이션 레벨의 최적화(예: 텔레메트리 전송 주기 조절)를 병행한다.
- **리스크 2: 중간 장비(Middlebox)에 의한 UDP 트래픽 차단:** 일부 네트워크 방화벽이나 라우터가 QUIC이 사용하는 UDP 트래픽을 차단하거나 속도를 제한할 수 있다.5
  - **완화 방안:** 앞서 언급한 TCP 폴백 메커니즘을 구현하여, QUIC 연결이 불가능한 환경에서도 서비스 연속성을 확보한다.

## 1.  상업용 드론 관제 시스템의 통신 요구사항 분석

상업용 드론 관제 시스템을 위한 최적의 통신 프로토콜을 선정하기 위해서는, 먼저 시스템이 처리해야 하는 데이터의 종류와 각 데이터 흐름의 고유한 요구사항을 명확히 정의해야 한다. 특히 장거리 비가시권(BVLOS) 운용을 전제로 할 때, 통신 프로토콜은 극도의 신뢰성과 저지연성을 보장해야 한다.

### 1.1  핵심 데이터 흐름 정의

상업용 드론 관제 시스템의 통신은 크게 세 가지의 이질적인 데이터 흐름으로 구성되며, 이들은 단일 통신 링크를 통해 동시에 처리되어야 한다.

- **명령 및 제어 (Command & Control, C2):** C2 데이터는 조종사의 입력, 자동 항법 명령, 임무 수행 지시 등 드론의 기동을 직접 제어하는 핵심 정보다.15 이 데이터 흐름은 대역폭 요구량은 매우 낮지만, 실시간 반응성을 위해 극도로 낮은 지연 시간(ultra-low latency)과 무손실(lossless) 전송이 절대적으로 요구된다. 단 하나의 C2 패킷 손실이나 과도한 지연은 드론의 통제 불능 상태를 초래할 수 있는 치명적인 결과를 낳는다.
- **텔레메트리 (Telemetry):** 텔레메트리는 드론의 현재 상태를 나타내는 모든 데이터를 포함한다. GPS 좌표, 고도, 속도, 배터리 잔량, 모터 상태 등의 정보가 지속적으로 지상 관제소(GCS)로 전송된다.17 이 데이터는 MAVLink와 같은 표준 프로토콜을 통해 구조화되는 경우가 많다.17 C2만큼 치명적이지는 않지만, 안정적인 상황 인식을 위해 낮은 지연과 높은 신뢰성이 요구된다. 다만, 데이터가 지속적으로 갱신되므로 아주 드문 패킷 손실은 감내가 가능하다.
- **고화질 영상 스트림 (High-Definition Video Stream):** 드론에 장착된 카메라로부터 실시간 영상을 전송하는 데이터 흐름이다. 원격 조종, 감시, 정찰 등 다양한 임무 수행에 필수적이다. 이 데이터는 높은 대역폭을 요구하며 지연 시간에 민감하다.18 그러나 영상 스트림의 특성상, 오래된 프레임을 재전송하는 것은 실시간성에 오히려 해가 되므로 약간의 프레임 손실(frame drop)은 허용된다. 중요한 것은 끊김 없는 실시간 화면을 제공하는 것이다. 종단 간 지연(end-to-end latency)은 영상 캡처, 인코딩, 네트워크 전송, 디코딩, 디스플레이 단계를 모두 포함한 시간으로 측정된다.18

### 1.2  BVLOS 운용의 필수 조건: 저지연 및 연결 복원력

BVLOS 운용은 드론이 조종사의 시야를 벗어나 수십 킬로미터 이상 원거리에서 임무를 수행하는 것을 의미하며, 이는 통신 시스템에 극한의 요구사항을 부과한다.

- **지연 시간 예산 (Latency Budgets):** 각 데이터 흐름은 임무 성공과 안전을 위해 엄격한 지연 시간 목표를 충족해야 한다. 업계 표준에 따르면, C2 데이터의 종단 간 지연은 조종사의 즉각적인 제어를 위해 100ms 미만으로 유지되어야 한다.20 실시간 영상 스트림의 경우, 원활한 원격 조종을 위해 200-300ms 이내의 지연 시간이 목표가 된다.18
- **모바일 네트워크의 도전 과제:** 상업용 드론은 안정적인 통신 커버리지를 확보하기 위해 Wi-Fi, 4G LTE, 5G 등 가용한 모든 무선 네트워크를 활용한다.24 이러한 모바일 네트워크 환경은 지연 시간, 지터(jitter), 패킷 손실률이 시시각각 변하는 예측 불가능한 특성을 가진다. 가장 결정적인 문제는, 드론이나 조종사가 이동하며 네트워크 기지국이나 접속점(AP)이 변경될 때 클라이언트의 IP 주소가 바뀐다는 점이다. 기존의 통신 프로토콜은 이러한 IP 주소 변경 시 연결이 완전히 끊어지므로, BVLOS 운용의 연속성을 심각하게 저해한다.

### 1.3  상업용 운용을 위한 보안 요건

상업용 드론 시스템에서 보안은 선택이 아닌 필수 사항이다. 드론 탈취(hijacking), 데이터 도청, 조작 등을 방지하기 위해 드론과 GCS 간의 모든 데이터 흐름은 강력한 종단 간 암호화(end-to-end encryption) 및 인증(authentication)을 통해 보호되어야 한다.15 통신 프로토콜은 설계 단계부터 보안을 핵심 기능으로 내장하고 있어야 한다.

이상의 요구사항을 종합하여, 향후 프로토콜 평가의 기준으로 삼을 정량적 목표를 아래 표와 같이 정리할 수 있다. 이 표는 추상적인 요구사항을 구체적이고 측정 가능한 지표로 변환하여, 객관적인 기술 평가의 기반을 제공한다. 드론 관제 시스템의 통신 문제는 단일한 문제가 아니라, 서로 다른 특성을 가진 데이터 흐름을 동시에, 안정적으로 처리해야 하는 다중화(multiplexing)의 문제임을 명확히 보여준다. 기존 TCP 기반 접근법의 가장 큰 맹점은 단일 연결 상에서 이러한 데이터 흐름들의 각기 다른 신뢰도 요구사항을 구분하지 못한다는 점이다. 예를 들어, 손실이 허용되는 고대역폭 영상 스트림의 패킷 하나가 유실될 경우, TCP는 이 패킷의 재전송이 완료될 때까지 후속하는 모든 패킷의 전달을 막는다 (HOL 블로킹).1 이는 이미 수신단에 정상적으로 도착한 치명적인 C2 명령 패킷의 처리를 지연시켜 심각한 운용상의 위험을 초래할 수 있다. 그렇다고 해서 각 데이터 흐름마다 별도의 TCP 연결을 생성하는 것은 핸드셰이크, 리소스 소모, 혼잡 제어 관리 측면에서 엄청난 오버헤드를 유발하며, 특히 모바일 환경에서는 더욱 비효율적이다.26 따라서 이상적인 프로토콜은 독립적인 스트림 신뢰도를 보장하는 다중화를 네이티브로 지원해야 하며, 이는 TCP가 해결하도록 설계되지 않았지만 QUIC의 핵심 설계 목표 중 하나이다.27

| 데이터 흐름          | 지연 시간 목표 | 신뢰도 요구사항      | 일반적인 대역폭    | 보안 수준             |
| -------------------- | -------------- | -------------------- | ------------------ | --------------------- |
| 명령 및 제어 (C2)    | < 100ms        | 무손실 (신뢰성 보장) | 낮음 (<50 kbps)    | 높음 (인증 및 암호화) |
| 텔레메트리 (MAVLink) | < 200ms        | 경미한 손실 허용     | 중간 (50-500 kbps) | 높음 (인증 및 암호화) |
| HD 영상 스트림       | < 300ms        | 프레임 손실 허용     | 높음 (2-10 Mbps)   | 높음 (인증 및 암호화) |

## 2.  실시간 모바일 애플리케이션을 위한 QUIC 기술 심층 분석

QUIC 프로토콜은 IETF RFC 9000, 9001, 9002 등의 표준 문서를 통해 정의된 차세대 전송 프로토콜이다.14 QUIC의 핵심 설계 철학은 앞서 2장에서 정의한 상업용 드론 관제 시스템의 까다로운 요구사항들을 직접적으로 해결하는 데 초점이 맞춰져 있다.

### 2.1  핵심 프로토콜의 장점: TCP+TLS로부터의 패러다임 전환

QUIC은 기존 TCP+TLS 스택의 근본적인 한계를 극복하기 위해 설계되었다.

- **UDP 기반 설계:** QUIC이 TCP가 아닌 UDP를 기반으로 하는 것은 매우 전략적인 선택이다.14 이는 프로토콜 스택을 운영체제 커널이 아닌 사용자 공간에 구현할 수 있게 하여, TCP가 수십 년간 겪어온 프로토콜 경직화(protocol ossification) 문제와 커널 업데이트의 제약에서 벗어나게 해준다.27 덕분에 새로운 기능을 신속하게 개발하고 배포하며, 프로토콜 자체의 빠른 진화가 가능하다.
- **통합된 보안:** QUIC은 TLS 1.3 핸드셰이크를 전송 연결 설정 과정에 완벽하게 통합한다.5 이는 단순히 TLS를 TCP 위에서 실행하는 것과는 차원이 다르다. 전송 계층과 보안 계층이 하나의 통합된 핸드셰이크를 수행함으로써, 첫 패킷 교환부터 연결 인증과 암호화 키 협상이 동시에 이루어진다. 이로 인해 보안이 기본적으로 보장될 뿐만 아니라, 연결 설정에 필요한 지연 시간이 획기적으로 줄어든다.

### 2.2  저지연 핸드셰이크 (0-RTT/1-RTT)

QUIC의 빠른 연결 설정 능력은 드론 운용 환경에서 실질적인 이점을 제공한다.

- **1-RTT 핸드셰이크:** 새로운 서버에 처음 연결할 때, QUIC은 전송 핸드셰이크와 암호화 핸드셰이크를 결합하여 단 한 번의 RTT(Round-Trip Time)만으로 보안 연결 설정을 완료한다. 이는 최소 2-3 RTT가 소요되는 TCP + TLS 방식에 비해 연결 지연을 절반 이하로 줄이는 효과를 가져온다.5 드론이 일시적으로 통신이 끊겼을 때, 이 차이는 제어권을 다시 확보하기까지의 시간을 초 단위로 단축시킬 수 있다.
- **0-RTT 재연결:** 클라이언트가 이전에 접속했던 서버에 다시 연결할 경우, QUIC은 이전에 캐시된 정보를 활용하여 핸드셰이크 과정 없이 첫 번째 패킷에 암호화된 애플리케이션 데이터를 실어 보낼 수 있다.30 이는 RTT가 전혀 발생하지 않는 '0-RTT'를 의미하며, 네트워크 환경이 불안정하여 짧은 끊김이 빈번하게 발생하는 상황에서 C2 링크를 거의 중단 없이 유지하는 데 결정적인 역할을 한다.



### 2.3  전송 계층 다중화: Head-of-Line 블로킹 제거



이 기능은 드론 관제 시나리오에서 QUIC이 제공하는 가장 중요한 장점이라고 할 수 있다.

- **독립적인 스트림:** QUIC은 단일 물리적 연결 내에서 논리적으로 완전히 독립된 다수의 스트림을 생성하고 관리할 수 있다.14 각 스트림은 고유의 흐름 제어(flow control)를 가지며, 데이터는 스트림별로 독립적으로 재조립된다.
- **C2/영상 데이터 충돌 문제 해결:** 이 아키텍처를 드론 관제 시스템에 적용하면, C2, 텔레메트리, 영상 데이터를 각각 별개의 스트림에 매핑할 수 있다. 그 결과, 대역폭을 많이 차지하는 영상 스트림에서 패킷 손실이 발생하더라도, 해당 손실은 오직 영상 스트림에만 영향을 미친다. 다른 스트림으로 전송되는 C2 명령과 텔레메트리 데이터는 아무런 지연 없이 계속해서 처리될 수 있다. 이는 TCP에서 발생하는 전송 계층 HOL 블로킹 문제를 근본적으로 해결하며 1, 시스템의 안정성과 안전성을 비약적으로 향상시킨다.



### 2.4  연결 마이그레이션: 모바일 환경에서의 강인함



연결 마이그레이션 기능은 BVLOS 운용 시 네트워크 변화에 대응하기 위한 QUIC의 핵심 무기이다.

- **연결 ID (Connection ID):** TCP 연결은 '소스 IP/포트, 목적지 IP/포트'의 4-튜플(4-tuple)로 정의된다. 반면, QUIC 연결은 이와 무관한 고유의 연결 ID(CID)에 의해 식별된다.3
- **원활한 네트워크 핸드오버:** 안드로이드 관제 클라이언트가 Wi-Fi에서 셀룰러 네트워크로 전환되면 클라이언트의 IP 주소가 변경된다. TCP 환경에서는 이 순간 기존 연결이 강제로 종료되고, 모든 것을 처음부터 다시 시작하는 새로운 연결을 맺어야 한다.5 하지만 QUIC에서는 클라이언트가 단순히 새로운 IP 주소에서 동일한 CID를 사용하여 서버로 패킷을 보내기만 하면 된다. 서버는 CID를 보고 기존 세션임을 인지하고, 아무런 중단 없이 통신을 이어나간다.3 이는 네트워크 환경이 급변하는 상황에서도 드론에 대한 제어권을 놓치지 않게 해주는, BVLOS 운용의 필수 불가결한 기능이다.



### 2.5  비신뢰성 데이터그램 확장 (RFC 9221): 맞춤형 솔루션



QUIC의 표준 확장 기능인 비신뢰성 데이터그램은 실시간 드론 데이터 전송에 완벽하게 부합하는 솔루션을 제공한다.

- **'Fire-and-Forget' 방식:** RFC 9221은 보안이 설정된 QUIC 연결 내에서 비신뢰성 데이터그램을 전송하는 메커니즘을 정의한다.34 이 데이터그램들은 암호화되고 혼잡 제어의 대상이 되지만, 패킷 손실이 발생해도 재전송되지 않는다.
- **텔레메트리 및 영상 전송에 최적화:** 이 방식은 오래된 데이터가 무의미한 실시간 영상 프레임이나 주기적으로 갱신되는 텔레메트리 데이터 전송에 이상적이다. 신뢰성 보장을 위한 재전송 오버헤드를 제거하여 지연 시간을 최소화하면서도, QUIC 연결의 통합된 보안 및 혼잡 제어 컨텍스트의 이점은 그대로 누릴 수 있다.34 이는 기존에 TCP 연결과 별도로 UDP/DTLS 채널을 복잡하게 관리해야 했던 방식에 비해 훨씬 효율적이고 간단한 아키텍처를 가능하게 한다.

QUIC은 이처럼 드론 통신 링크가 TCP 기반 시스템에서 겪는 주요 실패 지점들(느린 연결 재설정, HOL 블로킹으로 인한 제어 지연, 네트워크 핸드오버 시 연결 단절)을 직접적으로 해결하는 기능들을 프로토콜 설계 단에 내장하고 있다. 따라서 QUIC의 도입은 단순한 성능 향상을 넘어, 시스템의 근본적인 안정성과 안전성을 확보하는 전략적 선택이라 할 수 있다. 아래 표는 드론 운용 관점에서 QUIC과 TCP+TLS의 핵심 기능 차이를 명확하게 비교한다.

| 기능                 | TCP+TLS                           | QUIC                                | 드론 제어 관련성                                   |
| -------------------- | --------------------------------- | ----------------------------------- | -------------------------------------------------- |
| 연결 설정            | 2-3 RTTs 14                       | 0-1 RTT 32                          | 통신 두절 후 제어권 회복 속도에 결정적             |
| Head-of-Line 블로킹  | 있음 (전송 계층) 1                | 없음 2                              | 영상 패킷 손실이 C2 명령을 지연시키는 것을 방지    |
| 연결 마이그레이션    | 없음 (IP 변경 시 연결 단절) 5     | 있음 (끊김 없음) 3                  | Wi-Fi/셀룰러 전환 시 비행 안정성 확보에 필수       |
| 비신뢰성 데이터 전송 | 없음 (별도 UDP/DTLS 채널 필요) 37 | 있음 (데이터그램 확장, RFC 9221) 34 | 효율적인 저지연 텔레메트리 및 영상 스트리밍에 최적 |



## 3.  안드로이드 플랫폼에서의 성능 및 리소스 영향 분석



QUIC이 드론 관제 시스템의 요구사항에 기능적으로 부합하더라도, 클라이언트가 구동되는 안드로이드 플랫폼의 제한된 리소스 환경에서 어떤 성능적 특성을 보이는지 분석하는 것은 매우 중요하다. QUIC의 가장 큰 구조적 특징인 사용자 공간(user-space) 구현은 성능과 리소스 사용량 측면에서 명확한 장단점을 가진다.



### 3.1  사용자 공간의 트레이드오프



- **커널 vs. 사용자 공간:** TCP는 지난 수십 년간 운영체제 커널 깊숙이 통합되어 하드웨어 오프로딩을 포함한 수많은 최적화의 혜택을 받아왔다.13 반면, QUIC은 애플리케이션의 일부로 사용자 공간에서 실행됨으로써 이러한 커널 최적화를 우회한다.
- **CPU 부하 증가:** 사용자 공간에서의 패킷 처리, 암호화, 혼잡 제어 등의 작업은 TCP에 비해 더 많은 CPU 사이클을 소모한다.39 데이터는 커널과 사용자 공간 사이를 여러 번 복사해야 하며, 과거에는 네트워크 카드(NIC)나 커널이 처리하던 작업을 애플리케이션이 직접 수행해야 하기 때문이다.13
- **배터리 소모:** 모바일 드론 컨트롤러에서 CPU 사용량 증가는 곧 배터리 수명 단축으로 이어진다.41 이는 장시간의 야외 임무 수행 시 중요한 제약 조건이 될 수 있다. 일부 자료에서는 QUIC과 같은 효율적인 프로토콜이 오히려 에너지 소모를 줄일 수 있다고 언급하지만 41, 이는 웹 페이지 로딩과 같이 짧고 단발적인 통신에 해당할 가능성이 높다. 드론 관제와 같이 지속적인 데이터 스트림이 발생하는 환경에서는 CPU 부하로 인한 전력 소모 증가가 더 두드러질 수 있다.



### 3.2  성능 벤치마크: 상황에 따른 성능 변화



다양한 학술 및 실제 환경 벤치마크 결과들은 QUIC의 성능이 네트워크 환경에 따라 크게 달라진다는 것을 일관되게 보여준다.

- **이상적인 네트워크 (고대역폭, 저지연):** 안정적인 고속 Wi-Fi나 500 Mbps 이상의 5G 네트워크와 같이 대역폭이 풍부하고 지연 및 손실이 거의 없는 환경에서는, QUIC의 사용자 공간 처리 오버헤드가 성능의 병목이 될 수 있다. 다수의 연구에서 이러한 이상적인 조건 하에서는 TCP가 QUIC보다 더 높은 순수 전송률(throughput)을 기록하는 것으로 나타났다.39
- **열악한 네트워크 (고지연, 패킷 손실):** QUIC이 진정한 강점을 발휘하는 구간이다. RTT가 높고 패킷 손실이 빈번한 일반적인 모바일 네트워크 환경에서는, QUIC의 우수한 손실 복구 메커니즘과 HOL 블로킹 회피 능력이 빛을 발한다. 이러한 조건에서는 TCP의 성능이 급격히 저하되는 반면, QUIC은 훨씬 더 안정적이고 높은 유효 전송률(goodput)과 낮은 지연 시간을 유지한다.43 특히 영상 스트리밍의 경우, 이는 끊김(stall) 현상의 횟수와 시간을 현저히 줄이는 결과로 이어진다.39



### 3.3  안드로이드 특화 분석



모바일 기기에서의 성능은 데스크톱 환경과 다른 양상을 보인다.

- **성능 이점 감소:** 모바일 환경을 특정한 연구들은 QUIC이 여전히 TCP보다 우수한 경우가 많지만, 그 성능 향상의 폭이 데스크톱 환경에 비해 줄어든다고 지적한다.12 이는 모바일 CPU의 제한된 처리 능력과 사용자 공간 스택의 오버헤드 때문이다. 특히 높은 데이터 전송률에서는 애플리케이션이 수신된 패킷을 처리하는 속도가 네트워크 속도를 따라가지 못하는 '애플리케이션 제한(Application Limited)' 상태에 더 자주 빠지게 된다.44

이러한 분석을 통해 도출되는 결론은, 안드로이드 환경에서 QUIC을 선택하는 것은 '어느 프로토콜이 절대적으로 더 빠른가'의 문제가 아니라 '어느 프로토콜이 더 강인한가'의 문제라는 점이다. 상업용 드론 관제 시스템은 완벽한 Wi-Fi 환경이 아닌, 신호가 약한 셀룰러 환경과 같은 최악의 시나리오에서도 안정적으로 동작해야 한다. 약한 신호 환경에서의 심각한 성능 저하나 연결 실패는 치명적인 안전 문제로 직결된다. 반면, 최상의 네트워크 환경에서 최대 전송률이 다소 낮아지는 것은 충분히 감수할 수 있는 트레이드오프다. 따라서, 스트레스 상황에서 더 나은 성능을 보이는 프로토콜을 우선적으로 고려해야 한다. QUIC의 높은 CPU 및 배터리 소모는 이러한 '강인함'을 얻기 위해 지불하는 비용으로 해석할 수 있으며, 엔지니어링의 초점은 효율적인 라이브러리 선택과 구현 최적화를 통해 이 비용을 최소화하는 데 맞춰져야 한다.



## 4.  안드로이드 구현을 위한 QUIC 라이브러리 평가



안드로이드 클라이언트에서 QUIC을 구현하기 위해 사용할 수 있는 라이브러리는 여러 가지가 있으며, 각각의 특성과 장단점이 명확하다. 상업용 제품에 적용하기 위해서는 성능뿐만 아니라 안정성, 기술 지원, 통합 용이성 등을 종합적으로 고려해야 한다.



### 4.1  Google Cronet (Google Play Services 경유): 플랫폼 표준



- **개요:** Cronet은 Google Chrome 브라우저의 네트워킹 스택을 안드로이드 앱에서 사용할 수 있도록 라이브러리 형태로 패키징한 것이다.7 YouTube, Google Maps 등 수억 명이 사용하는 Google의 핵심 앱에서 이미 사용되며 그 안정성과 성능이 검증되었다.8 내부적으로는 Google의 QUIC 구현체인 QUICHE를 사용한다.48

- **통합:** 가장 권장되는 통합 방식은 Google Play Services를 통하는 것이다. 이 방식은 앱의 최종 바이너리 크기를 약 5MB 가량 줄여주고, Google이 직접 최신 보안 패치와 성능 개선 사항을 사용자 기기에 배포해주는 큰 장점이 있다.9 통합 과정은 Gradle 의존성을 추가하고 

  `CronetEngine.Builder`를 통해 엔진을 설정하는 비교적 간단한 절차로 이루어진다.9

- **기능 및 설정:** 요청 및 콜백 처리를 위한 고수준 API를 제공하며, QUIC 활성화, 캐싱 정책, 세부 QUIC 파라미터 조정 등 광범위한 설정 옵션을 지원한다.47 또한 ExoPlayer(영상), gRPC 등 다른 주요 안드로이드 라이브러리와의 공식적인 통합을 지원하여 생태계 호환성이 뛰어나다.49

- **장점:** Google의 공식 지원, 대규모 환경에서 검증된 안정성, 손쉬운 통합, 자동 업데이트.

- **단점:** Quiche와 같은 저수준 라이브러리를 직접 사용하는 것에 비해 프로토콜 동작에 대한 세밀한 제어는 제한될 수 있다.



### 4.2  Cloudflare Quiche (직접 통합): 저수준의 강력함



- **개요:** Cloudflare가 개발한 Rust 기반의 고성능 오픈소스 QUIC 및 HTTP/3 구현체이다.50 Cloudflare의 엣지 네트워크와 안드로이드 OS의 DNS-over-HTTP/3 리졸버 등에서 실제로 사용되고 있다.50
- **통합:** 통합 과정이 상당히 복잡하다. Rust로 작성된 라이브러리를 안드로이드 NDK와 `cargo-ndk` 툴체인을 사용하여 각 CPU 아키텍처에 맞게 크로스 컴파일한 후, FFI(Foreign Function Interface)를 통해 Java/Kotlin 코드와 연동해야 한다.50 또한, 소켓 핸들링과 같은 모든 I/O 처리와 이벤트 루프 관리를 애플리케이션이 직접 책임져야 한다.53
- **기능 및 설정:** 혼잡 제어 알고리즘 선택, 패킷 페이싱(pacing), 타이머 관리 등 프로토콜의 거의 모든 동작을 직접 제어할 수 있는 최대의 유연성을 제공한다.53 이는 강력한 장점인 동시에 높은 복잡도를 수반한다.
- **장점:** Rust의 메모리 안전성 덕분에 잠재적인 보안 취약점이 적고, 성능이 매우 뛰어나며, 최대의 유연성과 제어권을 제공한다.
- **단점:** Rust와 저수준 네트워킹에 대한 깊은 전문 지식이 없는 팀에게는 구현 및 유지보수 부담이 매우 크다.



### 4.3  Microsoft MsQuic: 고성능 경쟁자



- **개요:** Microsoft가 C언어로 개발한 고성능, 크로스플랫폼 QUIC 라이브러리다. 높은 처리량과 낮은 지연 시간을 모두 목표로 최적화되었으며, Windows의 HTTP/3와 SMB 프로토콜 스택의 기반 기술로 사용된다.54
- **통합:** 안드로이드용 빌드를 제공하기는 하지만, 이는 '비공식 지원(unsupported)' 상태이다.54 즉, Microsoft가 안드로이드 플랫폼에서의 정상 동작, 향후 업데이트, 보안 취약점 대응을 보장하지 않는다는 의미다. 통합을 위해서는 C/C++ NDK 전문 지식이 필요하다.
- **기능 및 설정:** 다른 구현체들을 능가하는 최고 수준의 성능을 목표로 설계되었다.55 라이선스는 상업적 사용에 제약이 없는 MIT 라이선스를 따른다.58
- **장점:** 잠재적으로 가장 높은 성능을 기대할 수 있다. MIT 라이선스로 자유로운 사용이 가능하다.
- **단점:** **'비공식 안드로이드 지원'은 상업용 제품에 채택하기에는 치명적인 단점이다.** 향후 OS 업데이트와의 호환성 문제, 보안 취약점 발견 시 신속한 패치 부재 등 예측 불가능한 리스크를 감수해야 한다.

결론적으로, 라이브러리 선택은 제어권과 복잡성 사이의 전형적인 엔지니어링 트레이드오프 문제이다. 신뢰성과 유지보수성이 최우선인 상업용 제품의 관점에서 볼 때, 공식적으로 지원되며 고수준 API를 제공하는 '충분히 좋은' 솔루션(Cronet)이, 잠재적 성능은 더 높을 수 있으나 복잡하고 지원이 불확실한 대안(Quiche, MsQuic)보다 전략적으로 우월하다. Quiche가 제공하는 극한의 제어권은 애플리케이션 개발팀에게 사실상 전송 계층을 직접 관리하라는 부담을 주며 50, MsQuic의 비공식 지원 상태는 장기적인 안정성을 담보할 수 없다.54 반면 Cronet은 이러한 복잡성을 추상화하고 Google에 의해 유지보수 및 보안이 관리되며, 이미 이 프로젝트의 요구사항을 훨씬 뛰어넘는 규모에서 그 성능과 안정성을 입증했다.8 따라서 Cronet이 제공하는 안정성, 보안, 개발 오버헤드 감소라는 막대한 이점은 Quiche나 MsQuic이 제공할 수 있는 미미한 성능 향상의 가능성을 충분히 압도한다.

| 평가 기준           | Google Cronet                   | Cloudflare Quiche (직접 통합) | Microsoft MsQuic    |
| ------------------- | ------------------------------- | ----------------------------- | ------------------- |
| **안드로이드 지원** | **공식 (Play Services 경유)** 9 | 공식 (빌드 필요) 50           | 비공식 54           |
| **통합 용이성**     | **높음** 9                      | 낮음 52                       | 중간 (C-interop)    |
| **성능**            | 좋음 (검증됨) 8                 | 높음 (튜닝 가능) 50           | 매우 높음 (주장) 55 |
| **리소스 사용량**   | 중간 12                         | 낮음 (최적화 가능)            | 낮음 (최적화됨)     |
| **설정 유연성**     | 높음 48                         | **매우 높음** 53              | 높음                |
| **라이선스**        | Apache 2.0                      | BSD-2-Clause 50               | MIT 58              |



## 5.  시스템 아키텍처 및 구현 전략



성공적인 QUIC 도입은 단순히 라이브러리를 선택하는 것에서 그치지 않는다. 프로토콜의 특성을 최대한 활용하고 잠재적인 약점을 보완하는 견고한 시스템 아키텍처를 설계하는 것이 무엇보다 중요하다.



### 5.1  제안 아키텍처



- **클라이언트 (안드로이드):** 5장에서의 평가 결과를 바탕으로 **Google Cronet**을 채택한다. 애플리케이션 전역에서 단일 `CronetEngine` 인스턴스를 생성하여 관리하며, 이 인스턴스는 QUIC 프로토콜을 사용하도록 명시적으로 설정한다.9 C2, 텔레메트리, 영상 스트림 등 모든 데이터 흐름은 이 단일 엔진을 통해 생성된 하나의 QUIC 연결 위에서 다중화되어 서버와 통신한다.

- **서버 (지상 관제소):** 서버 측은 안드로이드 환경의 제약에서 자유로우므로, 서버 환경에 최적화된 고성능 QUIC 구현체를 선택해야 한다. 후보로는 LiteSpeed의 `lsquic` 59, 서버 환경에서 공식 지원되는 

  `msquic` 54, 또는 Cloudflare의 패치를 적용한 NGINX 60 등이 있다. 최종 선택은 기존 서버 백엔드와의 통합 용이성 및 성능 벤치마크 결과에 따라 결정되어야 한다.



### 5.2  네트워크 적대 환경 대응: 폴백 메커니즘



이는 시스템의 신뢰성을 보장하기 위한 가장 중요한 안전장치이다.

- **문제점:** QUIC은 UDP를 기반으로 하므로, 일부 기업이나 공공기관의 엄격한 방화벽 정책, 혹은 특정 이동통신사의 정책에 의해 UDP 트래픽 자체가 차단되거나 속도가 심하게 제한될 수 있다.10 QUIC에만 의존하는 시스템은 이러한 환경에서 완전히 무력화될 위험이 있다.
- **해결책:** 안드로이드 클라이언트는 'Happy Eyeballs' (RFC 8305) 또는 '연결 경주(Connection Racing)' 메커니즘을 반드시 구현해야 한다. 연결을 시작할 때, QUIC 연결 시도와 전통적인 TCP+TLS 연결 시도를 동시에 병렬적으로 시작한다.62 둘 중 먼저 핸드셰이크를 성공적으로 마치는 연결을 사용하고, 다른 하나는 즉시 폐기한다. 만약 QUIC 연결이 짧은 시간(예: 1-2초) 내에 설정되지 않으면, 클라이언트는 자동으로 TCP 연결로 폴백하여 통신을 계속해야 한다.5 이 메커니즘은 적대적인 네트워크 환경에서도 드론의 제어 가능성을 잃지 않도록 보장하는 핵심적인 역할을 한다. Cronet의 빌더는 SPDY/HTTP2와 같은 보조 경로가 사용 가능할 때의 동작을 제어하는 옵션을 제공하며, 이를 폴백 로직 구현에 활용할 수 있다.48



### 5.3  데이터 스트림 매핑 전략



QUIC의 다중화 기능을 효과적으로 활용하기 위한 구체적인 매핑 전략은 다음과 같다.

- **C2 명령:** **신뢰성 있는 양방향 스트림(reliable bidirectional stream)**에 매핑한다. 이는 TCP와 같이 순서가 보장되고 손실 없는 데이터 전송을 보장하므로, 조종 입력과 같은 핵심 명령에 가장 적합하다.
- **텔레메트리 (MAVLink):** **비신뢰성 데이터그램(unreliable datagram)** 스트림에 매핑한다. RFC 9221 확장을 활용하여, 주기적으로 갱신되는 데이터의 재전송으로 인한 지연을 최소화한다. 만약 애플리케이션 레벨에서 순서 보장이 필요하다면, 페이로드 내에 경량의 시퀀스 번호를 포함하여 과도한 데이터 손실 여부를 감지할 수 있다.
- **영상 프레임:** 각 영상 프레임(또는 H.264의 슬라이스 단위)을 **개별 비신뢰성 데이터그램**으로 매핑한다. 이는 시간 민감도가 매우 높은 영상 데이터에 가장 효율적인 방식이다. 패킷 손실이 발생해도 재전송을 기다리지 않으므로 HOL 블로킹이 발생하지 않으며, 실시간성을 극대화할 수 있다.34



### 5.4  설정 및 튜닝



- **0-RTT 활성화:** Cronet 엔진 설정 시 0-RTT 기능을 명시적으로 활성화하여, 재연결 시의 지연을 최소화하도록 구성해야 한다.48
- **유휴 시간 초과(Idle Timeout) 조정:** 네트워크 상태에 따라 유휴 시간 초과 값을 공격적으로 설정할 필요가 있다. 수 초간 아무런 데이터 교환이 없을 경우, 연결이 비정상 상태라고 간주하고 선제적으로 새로운 연결을 시도하여 항상 반응성 높은 링크를 유지하도록 설계한다.
- **리소스 관리 및 모니터링:** 개발 및 테스트 단계에서 목표 안드로이드 기기들을 대상으로 CPU 및 배터리 사용량을 지속적으로 프로파일링해야 한다. 만약 소모량이 과도할 경우, QUIC을 비활성화하기보다는 텔레메트리 전송 빈도를 동적으로 조절하거나 영상 비트레이트를 조정하는 등 애플리케이션 레벨에서의 최적화를 우선적으로 고려해야 한다.

성공적인 QUIC 구현은 단순히 라이브러리를 사용하는 것을 넘어, 프로토콜을 중심으로 한 강인한 시스템을 설계하는 과정이다. 특히 폴백 메커니즘과 데이터 스트림 매핑 전략은 프로토콜 자체만큼이나 시스템의 전체 성능과 안정성을 좌우하는 핵심 요소임을 명심해야 한다.



## 6.  결론 및 전략적 권고





### 6.1  최종 결론



본 보고서의 심층 분석 결과에 기반하여, 상업용 드론 관제 시스템의 주 통신 프로토콜로 QUIC을 채택할 것을 다시 한번 강력히 권고한다. 이는 단순히 기술적 우위를 점하는 것을 넘어, 드론 시스템의 신뢰성, 성능, 그리고 미래 확장성을 보장하는 전략적 투자이다. QUIC은 모바일, 실시간, 다중 데이터 스트림이라는 드론 통신의 핵심 과제를 해결하기 위해 탄생한 기술이며, 기존 TCP 기반 시스템이 가진 근본적인 한계에 대한 명확한 해답을 제시한다.



### 6.2  권장 라이브러리 및 근거



안드로이드 클라이언트 구현을 위한 라이브러리로는 **Google Cronet (Google Play Services 경유 방식)**을 공식적으로 권장한다. 이 결정은 5장의 평가 매트릭스에 근거하며, 다음과 같은 이유로 뒷받침된다.

- **신뢰성과 안정성:** Google의 공식 지원과 수십억 사용자를 통한 검증은 상업용 제품에서 요구되는 최고 수준의 안정성을 보장한다.
- **보안 및 유지보수:** Play Services를 통해 배포됨으로써, 최신 보안 취약점 패치와 성능 개선이 Google에 의해 자동으로 관리되어 개발팀의 유지보수 부담을 크게 경감시킨다.
- **개발 효율성:** 고수준 API와 잘 갖춰진 문서는 복잡한 저수준 네트워크 프로그래밍 없이도 신속한 개발을 가능하게 하여, 프로젝트 리스크와 시장 출시 기간(Time-to-Market)을 단축시킨다.

Cronet은 성능, 공식 지원, 보안, 구현 용이성 측면에서 가장 최적의 균형을 제공하는, 가장 실용적이고 전문적인 선택이다.



### 6.3  구현 및 테스트 로드맵



성공적인 도입을 위해 다음의 단계적 로드맵을 제안한다.

- **1단계: 프로토타입 개발:** Google Cronet을 사용하여 프로토타입을 개발한다. 이 단계의 핵심 목표는 6장에서 제안한 듀얼 스택(QUIC 우선, TCP 폴백) 연결 로직과 데이터 스트림(C2, 텔레메트리, 영상) 매핑 전략을 구현하고 검증하는 것이다.
- **2단계: 엄격한 테스트:** 실험실 환경의 네트워크 에뮬레이션과 실제 필드 테스트를 병행하여 광범위한 테스트를 수행한다. 테스트 시나리오는 다음을 반드시 포함해야 한다.
  - **네트워크 핸드오버 테스트:** Wi-Fi와 5G/LTE 네트워크 간의 반복적인 전환 상황에서 연결 마이그레이션 기능의 안정성을 검증한다.
  - **네트워크 품질 저하 테스트:** 높은 지연, 지터, 다양한 수준의 패킷 손실률을 인위적으로 발생시켜 QUIC의 강인성과 성능을 TCP와 비교 측정한다.
  - **방화벽 통과 테스트:** UDP 트래픽을 차단하는 것으로 알려진 네트워크 환경에서 TCP 폴백 메커니즘이 정상적으로, 그리고 신속하게 동작하는지 검증한다.
  - **리소스 프로파일링:** 다양한 사양의 목표 안드로이드 기기에서 장시간 운용 시 CPU 점유율과 배터리 소모 패턴을 측정하고, 최적화 방안을 모색한다.
- **3단계: 단계적 배포:** QUIC이 활성화된 클라이언트를 제한된 사용자 그룹(베타 테스터)에게 우선 배포하여 실제 환경에서의 성능 및 안정성 데이터를 수집한다. 원격으로 QUIC 기능을 비활성화하고 강제로 TCP 폴백을 유도할 수 있는 기능 플래그(feature flag)를 구현하여, 예상치 못한 문제 발생 시 신속하게 대응할 수 있도록 한다. QUIC과 TCP의 실제 연결 성공률을 지속적으로 모니터링하여, 현실 세계의 네트워크 적대성 수준을 정량적으로 파악하고 향후 개선 방향을 설정한다.

#### **참고 자료**

1. The Full Picture on HTTP/2 and HOL Blocking - Salesforce Engineering Blog, accessed July 29, 2025, https://engineering.salesforce.com/the-full-picture-on-http-2-and-hol-blocking-7f964b34d205/
2. Why HTTP/3 Was Invented: Solving Head-of-Line Blocking in HTTP/2 | by Ahmad Bilal, accessed July 29, 2025, https://medium.com/@ahmadbilalch891/why-http-3-was-invented-solving-head-of-line-blocking-in-http-2-daffac76ba01
3. Connection Migration – quic-go docs, accessed July 29, 2025, https://quic-go.net/docs/quic/connection-migration/
4. An Analysis of QUIC Connection Migration in the Wild - arXiv, accessed July 29, 2025, https://arxiv.org/html/2410.06066v1
5. QUIC: The Secure Communication Protocol Shaping the Internet's Future - Zscaler, accessed July 29, 2025, https://www.zscaler.com/blogs/product-insights/quic-secure-communication-protocol-shaping-future-of-internet
6. QUIC vs TCP+TLS - and why QUIC is not the next big thing | by Codavel - Medium, accessed July 29, 2025, https://medium.com/codavel-blog/quic-vs-tcp-tls-and-why-quic-is-not-the-next-big-thing-d4ef59143efd
7. What is Cronet, accessed July 29, 2025, https://chromium.googlesource.com/chromium/src/+/lkgr/components/cronet/README.md
8. Perform network operations using Cronet | Connectivity - Android Developers, accessed July 29, 2025, https://developer.android.com/develop/connectivity/cronet
9. Cronet Basics - Android Developers, accessed July 29, 2025, https://developer.android.com/codelabs/cronet
10. Recommended method to block QUIC and HTTP/3 - Zscaler Community, accessed July 29, 2025, https://community.zscaler.com/s/question/0D54u00009evmkVCAQ/recommended-method-to-block-quic-and-http3
11. Managing the QUIC Protocol - Zscaler Help Portal, accessed July 29, 2025, https://help.zscaler.com/zia/managing-quic-protocol
12. Taking a Long Look at QUIC: An Approach for Rigorous Evaluation of Rapidly Evolving Transport Protocols - Communications of the ACM, accessed July 29, 2025, https://cacm.acm.org/research/taking-a-long-look-at-quic/
13. What makes QUIC less efficient in CPU and memory usage? - Hacker News, accessed July 29, 2025, https://news.ycombinator.com/item?id=39169357
14. How IETF QUIC (RFC 9000) Reinvents Transport Protocols - Patsnap Eureka, accessed July 29, 2025, https://eureka.patsnap.com/article/how-ietf-quic-rfc-9000-reinvents-transport-protocols
15. Communication architecture of UAV using strategic and tactical messages. - ResearchGate, accessed July 29, 2025, https://www.researchgate.net/figure/Communication-architecture-of-UAV-using-strategic-and-tactical-messages_fig3_377461737
16. [2501.00856] Advances in UAV Avionics Systems Architecture, Classification and Integration: A Comprehensive Review and Future Perspectives - arXiv, accessed July 29, 2025, https://arxiv.org/abs/2501.00856
17. MAVLink Communication Protocol: the open standard – Auterion, accessed July 29, 2025, https://auterion.com/mavlink-communication-protocol-the-open-standard/
18. Low-latency design considerations for video-enabled aerial drones - Texas Instruments, accessed July 29, 2025, https://www.ti.com/lit/wp/spry301/spry301.pdf
19. Drone Communication Systems - Fly Eye, accessed July 29, 2025, https://www.flyeye.io/drone-technology-communication/
20. FPV Drone Latency Explained: What It Is and How to Reduce It - Oscar Liang, accessed July 29, 2025, https://oscarliang.com/fpv-drone-latency/
21. Achieving Ultra-Low Latency Streaming Techniques - FastPix, accessed July 29, 2025, https://www.fastpix.io/blog/benefits-of-ultra-low-latency-video-streaming
22. Remote Operation - Low Latency Video | Soliton Systems, accessed July 29, 2025, https://www.solitonsystems.com/low-latency-video/remote-operation
23. Standard GPS Drones vs. FPV Drones Video Latency | FPVCraft, accessed July 29, 2025, https://fpvcraft.com/standard-gps-drones-vs-fpv-drones-video-latency/
24. Reliable Communication Systems for Long-Range Drone Operations - Xray, accessed July 29, 2025, https://xray.greyb.com/drones/communication-protocols-long-range-drone-networks
25. Elevating industries with the Internet of Drones - Berg Insight, accessed July 29, 2025, https://www.berginsight.com/elevating-industries-with-the-internet-of-drones
26. How does HTTP2 solve Head of Line blocking (HOL) issue - Stack Overflow, accessed July 29, 2025, https://stackoverflow.com/questions/45583861/how-does-http2-solve-head-of-line-blocking-hol-issue
27. QUIC - Wikipedia, accessed July 29, 2025, https://en.wikipedia.org/wiki/QUIC
28. What Is the QUIC Protocol? | EMQ - EMQX, accessed July 29, 2025, https://www.emqx.com/en/blog/quic-protocol-the-features-use-cases-and-impact-for-iot-iov
29. QUIC Working Group, accessed July 29, 2025, https://quicwg.org/
30. What is QUIC (Quick UDP Internet Connections)? - JumpCloud, accessed July 29, 2025, https://jumpcloud.com/it-index/what-is-quic-quick-udp-internet-connections
31. RFC 9000 - QUIC: A UDP-Based Multiplexed and Secure Transport - Datatracker - IETF, accessed July 29, 2025, https://datatracker.ietf.org/doc/rfc9000/
32. What is QUIC Protocol? - PubNub, accessed July 29, 2025, https://www.pubnub.com/learn/glossary/quic-protocol/
33. Design and evaluation of an inter- core QUIC connection migration approach for intra-server load balancing - DiVA, accessed July 29, 2025, https://kth.diva-portal.org/smash/get/diva2:1615017/FULLTEXT01.pdf
34. RFC 9221 - An Unreliable Datagram Extension to QUIC - IETF Datatracker, accessed July 29, 2025, https://datatracker.ietf.org/doc/html/rfc9221
35. Information on RFC 9221 - » RFC Editor, accessed July 29, 2025, https://www.rfc-editor.org/info/rfc9221
36. QUIC: Unreliable Datagram Extension (RFC 9221) implementation / Issue #24003 - GitHub, accessed July 29, 2025, https://github.com/openssl/openssl/issues/24003
37. TCP, UDP or QUIC? Read this before you choose. - DEV Community, accessed July 29, 2025, https://dev.to/pieter/tcp-udp-or-quic-read-this-before-you-choose-432e
38. QUIC is not Quick Enough over Fast Internet : r/programming - Reddit, accessed July 29, 2025, https://www.reddit.com/r/programming/comments/1g7vv66/quic_is_not_quick_enough_over_fast_internet/
39. Comparing TCP and QUIC - Hacker News, accessed July 29, 2025, https://news.ycombinator.com/item?id=33431669
40. QUIC is not Quick Enough over Fast Internet - arXiv, accessed July 29, 2025, https://arxiv.org/html/2310.09423v2
41. How App Architecture Choices Impact Battery Life on Android Devices - MoldStud, accessed July 29, 2025, https://moldstud.com/articles/p-how-app-architecture-choices-impact-battery-life-on-android-devices
42. Relationship between CPU Usage and Battery Usage - Android Enthusiasts Stack Exchange, accessed July 29, 2025, https://android.stackexchange.com/questions/167045/relationship-between-cpu-usage-and-battery-usage
43. QUIC vs TCP: A Performance Evaluation over LTE with NS-3 - Scirp.org., accessed July 29, 2025, https://www.scirp.org/journal/paperinformation?paperid=113892
44. Measuring QUIC vs TCP on mobile and desktop - APNIC Blog, accessed July 29, 2025, https://blog.apnic.net/2018/01/29/measuring-quic-vs-tcp-mobile-desktop/
45. QUIC vs TCP: "The Ultimate Benchmark for Modern Network Performance" | by Abhi Patel, accessed July 29, 2025, https://medium.com/@abhi.patel.7172/quic-vs-tcp-the-ultimate-benchmark-for-modern-network-performance-22be6e5af66d
46. [2504.10054] Implementation and Performance Evaluation of TCP over QUIC Tunnels, accessed July 29, 2025, https://arxiv.org/abs/2504.10054
47. Quick Start Guide to Using Cronet, accessed July 29, 2025, https://chromium.googlesource.com/experimental/chromium/src/+/refs/wip/bajones/webvr_1/components/cronet/README.md
48. QuicOptions.Builder | Connectivity - Android Developers, accessed July 29, 2025, https://developer.android.com/develop/connectivity/cronet/reference/org/chromium/net/QuicOptions.Builder
49. Use Cronet with other libraries | Connectivity - Android Developers, accessed July 29, 2025, https://developer.android.com/develop/connectivity/cronet/integration
50. cloudflare/quiche: Savoury implementation of the QUIC ... - GitHub, accessed July 29, 2025, https://github.com/cloudflare/quiche
51. quiche 0.24.4 - Docs.rs, accessed July 29, 2025, https://docs.rs/crate/quiche/latest/source/README.md
52. Kashyap Thimmaraju / quiche - GitLab, accessed July 29, 2025, https://gitlab.informatik.hu-berlin.de/thimmaka/quiche
53. quiche - Rust - quic.tech, accessed July 29, 2025, https://docs.quic.tech/quiche/
54. MsQuic - Wikipedia, accessed July 29, 2025, https://en.wikipedia.org/wiki/MsQuic
55. MsQuic -- A High-speed QUIC Implementation - Chair of Network Architectures and Services, accessed July 29, 2025, https://www.net.in.tum.de/fileadmin/TUM/NET/NET-2023-11-1/NET-2023-11-1_01.pdf
56. About: MsQuic - DBpedia, accessed July 29, 2025, https://dbpedia.org/page/MsQuic
57. msquic is one of the most performant QUIC protocol library out there. go-msquic - Hacker News, accessed July 29, 2025, https://news.ycombinator.com/item?id=43098691
58. microsoft/msquic: Cross-platform, C implementation of the ... - GitHub, accessed July 29, 2025, https://github.com/microsoft/msquic
59. litespeedtech/lsquic: LiteSpeed QUIC and HTTP/3 Library - GitHub, accessed July 29, 2025, https://github.com/litespeedtech/lsquic
60. Experiment with HTTP/3 using NGINX and quiche - The Cloudflare Blog, accessed July 29, 2025, https://blog.cloudflare.com/experiment-with-http-3-using-nginx-and-quiche/
61. Technical Tip: How to block/disable QUIC - the Fortinet Community!, accessed July 29, 2025, https://community.fortinet.com/t5/FortiGate/Technical-Tip-How-to-block-disable-QUIC/ta-p/191273
62. QUIC fallback to TCP scenario - Google Groups, accessed July 29, 2025, https://groups.google.com/a/chromium.org/g/proto-quic/c/zo7--OQLQBo
63. QUIC vs TCP+TLS and why QUIC is not the next big thing - Blog, accessed July 29, 2025, https://blog.codavel.com/quic-vs-tcptls-and-why-quic-is-not-the-next-big-thing

