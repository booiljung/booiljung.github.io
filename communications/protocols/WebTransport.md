# WebTransport 고찰



현대 웹 애플리케이션은 단순한 정보 제공을 넘어 사용자에게 풍부한 실시간 상호작용 경험을 제공하는 방향으로 진화하고 있다. 실시간 멀티플레이어 게임, 클라우드 기반의 협업 편집 도구, 양방향 라이브 스트리밍 서비스 등은 이제 웹 생태계의 핵심적인 부분으로 자리 잡았다. 이러한 애플리케이션들의 공통적인 요구사항은 바로 클라이언트와 서버 간의 저지연(low-latency) 양방향 통신 채널이다.

이러한 요구에 부응하기 위해 지난 수십 년간 다양한 기술이 등장하고 발전해왔다. 초기의 롱 폴링(Long Polling) 방식은 HTTP 요청-응답 모델의 한계 내에서 실시간성을 흉내 냈으나, 본질적인 지연 시간과 서버 부하 문제에서 자유롭지 못했다.1 이후 등장한 WebSocket은 단일 TCP 연결을 통해 지속적인 양방향 통신 채널을 제공하며 실시간 웹의 표준으로 자리매김했다. 그러나 WebSocket 역시 근본적인 한계를 내포하고 있었다. 가장 큰 문제는 TCP 프로토콜에 기인하는 'Head-of-Line(HOL) 블로킹' 현상이다. 단일 TCP 연결 위에서 동작하기 때문에, 하나의 패킷이 유실되면 해당 패킷이 재전송되어 도착할 때까지 연결 전체가 대기 상태에 빠지게 된다. 이는 여러 독립적인 데이터 스트림(예: 채팅 메시지와 게임 상태 업데이트)을 동시에 처리해야 하는 복잡한 애플리케이션에서 심각한 성능 저하를 유발한다.5

또 다른 대안인 WebRTC(Web Real-Time Communication)는 P2P(Peer-to-Peer) 통신에 최적화되어 있어 매우 낮은 지연 시간을 자랑하지만, 클라이언트-서버 통신 모델에 적용하기에는 구조적으로 복잡하다. WebRTC를 서버와 통신하는 데 사용하려면 ICE, STUN, TURN, SCTP 등 여러 프로토콜로 구성된 복잡한 스택을 서버 측에 구현해야 하며, 이는 상당한 개발 및 운영 비용을 초래한다.4 이처럼 기존 기술들은 현대 웹이 요구하는 저지연, 고효율, 다중화(multiplexing) 통신을 완벽하게 만족시키지 못하는 명확한 한계를 가지고 있었다. 바로 이러한 기술적 공백을 메우기 위해 WebTransport가 등장했다.


WebTransport는 기존 기술의 한계를 극복하고 차세대 실시간 웹 통신을 선도하기 위해 설계된 새로운 웹 API이자 프로토콜 프레임워크다.10 그 핵심 설계 철학은 HTTP/3의 기반이 되는 QUIC(Quick UDP Internet Connections) 프로토콜의 강력한 성능과 유연성을 웹 개발자에게 안전하고 효율적인 방식으로 노출하는 데 있다.12

WebTransport의 가장 중요한 가치는 단일 연결 내에서 두 가지 상이한 특성의 데이터 전송 방식을 동시에 제공한다는 점이다. 하나는 스트림 API(Streams API)를 통해 제공되는 '신뢰성 있는(reliable)' 순차적 데이터 전송이고, 다른 하나는 데이터그램 API(Datagrams API)를 통해 제공되는 '비신뢰성(unreliable)' 비순차적 데이터 전송이다.8 이를 통해 개발자는 전송하려는 데이터의 특성과 애플리케이션의 요구사항에 따라 최적의 전송 모드를 유연하게 선택할 수 있다. 예를 들어, 반드시 순서대로 전달되어야 하는 채팅 메시지는 신뢰성 스트림으로, 약간의 손실이 허용되지만 지연 시간에 극도로 민감한 게임 캐릭터의 위치 정보는 비신뢰성 데이터그램으로 전송하는 것이 가능하다.

이러한 설계는 WebTransport가 단순한 WebSocket의 대체재가 아님을 시사한다. 이는 통신 아키텍처에 대한 패러다임의 전환을 의미한다. WebSocket 환경에서는 단일 TCP 연결 위에서 하나의 논리적 메시지 스트림만이 제공되었다.6 따라서 채팅, 사용자 상태, 파일 전송 등 여러 종류의 데이터를 처리해야 하는 복잡한 애플리케E이션은 이 모든 데이터를 단일 스트림에 섞어 보낸 뒤, 애플리케이션 계층에서 메시지 포맷을 정의하고 파싱하여 각기 다른 데이터 흐름을 수동으로 관리해야 했다. 이는 통신 로직의 복잡성을 프로토콜이 아닌 애플리케이션 개발자에게 전가하는 방식이었다. HOL 블로킹을 피하기 위해 여러 개의 WebSocket 연결을 여는 것은 비효율적이고 자원 낭비가 심한 임시방편에 불과했다.16

반면, WebTransport는 QUIC의 스트림 다중화(multiplexing) 기능을 API 레벨에서 직접 노출한다.4 개발자는 논리적으로 분리된 데이터 흐름(예: 채팅, 오디오, 게임 이벤트)을 각각 독립적인 스트림으로 매핑할 수 있다. 한 스트림에서 문제가 발생하더라도 다른 스트림은 전혀 영향을 받지 않는다. 즉, 프로토콜 자체가 데이터 흐름의 분리와 관리에 대한 복잡성을 상당 부분 흡수하고, 개발자에게는 더 단순하면서도 강력한 추상화 모델을 제공하는 것이다. 이는 애플리케이션 아키텍처를 근본적으로 단순화하고, 성능과 안정성을 동시에 향상시키는 중요한 패러다임의 전환이라 할 수 있다.


WebTransport의 강력한 성능과 유연성은 그 기반이 되는 QUIC 프로토콜과 HTTP/3에 깊이 뿌리내리고 있다. WebTransport를 이해하기 위해서는 먼저 이 두 기술이 기존의 TCP와 HTTP/2에 비해 어떤 혁신을 이루었는지 파악하는 것이 필수적이다.


QUIC은 본래 구글(Google)에 의해 실험적으로 개발되어 시작되었으나, 그 우수성을 인정받아 IETF(Internet Engineering Task Force)에서 표준화 작업을 거쳐 공식적인 차세대 전송 프로토콜로 자리매김했다.11 QUIC은 신뢰성 있는 전송을 보장하면서도 TCP의 고질적인 문제들을 해결하기 위해 UDP(User Datagram Protocol) 위에 구현된 현대적인 전송 프로토콜이다.18


기존의 웹 통신에서 보안 연결(HTTPS)을 설정하기 위해서는 TCP 핸드셰이크(3-way handshake)와 TLS(Transport Layer Security) 핸드셰이크가 순차적으로 이루어져야 했다. 이 과정은 네트워크 상황에 따라 최소 2번에서 3번의 왕복 시간(Round-Trip Time, RTT)을 소요하여, 실제 데이터를 전송하기 전부터 상당한 지연을 유발했다.

QUIC은 이 과정을 획기적으로 개선했다. 전송 계층 핸드셰이크와 암호화 계층 핸드셰이크를 하나의 과정으로 통합하여, 새로운 서버에 처음 연결할 때 단 1-RTT 만에 안전한 연결을 수립할 수 있다.20 이는 사용자가 웹사이트에 접속하거나 애플리케이션을 시작할 때 느끼는 초기 로딩 시간을 크게 단축시킨다.

더 나아가 QUIC은 '0-RTT 핸드셰이크'라는 강력한 기능을 제공한다. 클라이언트가 이전에 연결했던 서버에 다시 접속할 경우, 저장해 둔 세션 정보를 활용하여 핸드셰이크가 완전히 완료되기도 전에 애플리케이션 데이터를 전송하기 시작할 수 있다.22 이는 연결 재개 시의 지연 시간을 거의 '0'에 가깝게 만들어준다. 다만, 0-RTT로 전송된 데이터는 재전송 공격(replay attack)에 취약할 수 있다는 보안상의 한계가 존재한다. 공격자가 0-RTT 데이터를 가로채 동일한 서버에 다시 전송할 수 있기 때문이다. 따라서 0-RTT는 서버의 상태를 변경하지 않는 멱등성(idempotent)을 가진 요청(예: 단순 데이터 조회)에 한해 신중하게 사용되어야 한다.22


Head-of-Line(HOL) 블로킹은 TCP 기반 프로토콜의 가장 큰 성능 저하 요인 중 하나였다. TCP는 전송되는 모든 데이터를 단일한 순차적 스트림으로 간주한다. 따라서 이 스트림의 중간에 있는 패킷 하나가 유실되면, 해당 패킷이 재전송되어 도착할 때까지 그 뒤에 오는 모든 패킷들은 수신되었더라도 처리되지 못하고 대기해야 한다.19

HTTP/2는 요청과 응답을 프레임 단위로 나누어 하나의 TCP 연결 위에서 다중화함으로써 HTTP 계층의 HOL 블로킹(예: 큰 이미지 다운로드 때문에 작은 CSS 파일 요청이 지연되는 현상)은 해결했다. 하지만 HTTP/2는 여전히 TCP 위에서 동작하므로, TCP 계층의 HOL 블로킹 문제에서는 자유롭지 못했다. 즉, TCP 패킷 하나가 유실되면 HTTP/2의 모든 스트림이 동시에 멈추는 현상은 여전했다.25

QUIC은 이 문제를 근본적으로 해결한다. QUIC 연결 내의 각 스트림은 논리적으로 완전히 독립적이다. 각 스트림은 자신만의 흐름 제어(flow control)와 패킷 복구 메커니즘을 가지고 있다. 따라서 특정 스트림에서 패킷 유실이 발생하더라도, 다른 스트림들은 아무런 영향 없이 계속해서 데이터를 전송하고 처리할 수 있다.5 이 아키텍처는 WebTransport가 여러 독립적인 데이터 채널을 효율적으로 제공할 수 있는 기술적 근간이 된다.


모바일 환경이 보편화되면서 사용자의 네트워크 환경은 수시로 변한다. 예를 들어, 이동 중에 Wi-Fi에서 LTE 네트워크로 전환되는 경우가 빈번하다. 기존의 TCP 연결은 클라이언트와 서버의 IP 주소 및 포트 번호의 조합(4-tuple)으로 식별되었기 때문에, 클라이언트의 IP 주소가 바뀌면 연결은 즉시 끊어지고 처음부터 다시 연결을 수립해야 했다.

QUIC은 '연결 ID(Connection ID)'라는 새로운 개념을 도입하여 이 문제를 해결했다. QUIC 연결은 IP 주소나 포트가 아닌, 이 고유한 연결 ID를 통해 식별된다. 따라서 클라이언트의 IP 주소가 변경되더라도, 클라이언트는 동일한 연결 ID를 사용하여 서버와의 통신을 중단 없이 이어갈 수 있다.3 이 연결 마이그레이션 기능은 모바일 기기에서의 실시간 애플리케이션 사용자 경험을 극적으로 향상시키는 핵심적인 혁신이다.


HTTP/3는 본래 'HTTP-over-QUIC'이라는 이름으로 논의되다가, IETF 내에서 공식적으로 'Hypertext Transfer Protocol Version 3'로 명명되었다.17 이름에서 알 수 있듯이 HTTP/3는 전송 계층으로 TCP 대신 QUIC을 사용하는 새로운 버전의 HTTP이다.

WebTransport는 바로 이 HTTP/3 연결 위에서 동작한다. 클라이언트는 HTTP/3 `CONNECT` 메서드를 사용하여 서버에 WebTransport 세션 생성을 요청한다. 일단 세션이 수립되면, 해당 연결은 WebTransport의 스트림 및 데이터그램 트래픽과 일반적인 HTTP/3 요청-응답 트래픽(예: API 호출, 이미지 로드)을 동시에 다중화하여 처리할 수 있다.13 이는 별도의 포트나 연결을 관리할 필요 없이 기존 웹 인프라와의 통합을 용이하게 한다.

여기서 중요한 점은 WebTransport가 QUIC의 기능을 웹에 직접 노출하는 원시 UDP 소켓 API가 아니라는 것이다. 브라우저 환경에서 임의의 UDP 소켓을 열어주는 것은 심각한 보안 위협을 초래할 수 있다. WebTransport는 `https://` 스키마를 사용하고, 신뢰할 수 있는 HTTP/3 서버에 연결하며, 동일 출처 정책(Same-Origin Policy)과 같은 기존 웹 보안 모델을 준수한다.8 즉, WebTransport는 'HTTP/3'라는 검증되고 안전한 컨텍스트를 매개체로 삼아 QUIC의 강력한 성능을 웹 개발자에게 제공하는 것이다. 이는 P2P 연결을 위해 ICE, STUN, TURN 등 복잡하고 별도의 외부 메커니즘을 필요로 하는 WebRTC와 명확히 대조되는 지점이며, WebTransport의 실용성과 배포 용이성을 높이는 핵심적인 설계 결정이다.8


WebTransport는 강력한 저수준 프로토콜 기능을 개발자가 쉽게 활용할 수 있도록 잘 설계된 고수준 JavaScript API를 제공한다. 이 API는 현대 웹 플랫폼의 표준인 `Promise`, `async/await`, 그리고 `Streams API`와 깊게 통합되어 있어 일관되고 효율적인 비동기 프로그래밍을 가능하게 한다.


WebTransport API의 중심에는 '세션(Session)'과 '스트림(Stream)'이라는 두 가지 핵심 추상화 개념이 있다.

- **세션 (Session):** 클라이언트와 서버 간의 단일 논리적 연결을 나타낸다. 이 세션은 `new WebTransport(url)` 생성자를 호출함으로써 시작된다.10

  `url`은 반드시 `https://` 스키마를 사용해야 하며, 명시적인 포트 번호를 포함해야 한다. 생성된 `WebTransport` 객체는 두 개의 중요한 `Promise` 기반 속성을 가진다.

  - `ready`: 이 프로미스가 이행(resolve)되면, 클라이언트와 서버 간의 연결이 성공적으로 수립되었으며 데이터를 주고받을 준비가 되었음을 의미한다.
  - `closed`: 이 프로미스는 연결이 종료될 때 이행된다. 정상적으로 종료되면 이행되고, 오류로 인해 비정상적으로 종료되면 거부(reject)된다. 이를 통해 연결 상태를 비동기적으로 감지하고 처리할 수 있다.

- **스트림 (Stream):** 세션 내에서 데이터를 주고받는 통로 역할을 한다. 하나의 세션 안에는 여러 개의 독립적인 스트림과 데이터그램 흐름이 공존할 수 있다.29


데이터그램 API는 전송 순서나 성공 여부가 중요하지 않은, 지연 시간에 매우 민감한 데이터를 위한 통신 방식이다.

- **작동 원리:** 데이터그램은 QUIC 프로토콜의 `DATAGRAM` 프레임을 직접 활용한다. 각 데이터그램은 독립적인 패킷으로 취급되며, 전송 순서가 보장되지 않고 중간에 유실될 수도 있다. 재전송 메커니즘이 없기 때문에 매우 낮은 지연 시간을 달성할 수 있다.8 각 데이터그램의 크기는 네트워크의 최대 전송 단위(Maximum Transmission Unit, MTU)에 의해 제한된다. 흔히 UDP 메시지와 비교되지만, WebTransport의 데이터그램은 모든 데이터가 

  **암호화**되고 QUIC의 **혼잡 제어(congestion control)** 메커니즘의 적용을 받는다는 결정적인 차이점이 있다.5

- **최적 사용 사례:** 최신 데이터가 이전 데이터를 즉시 대체하는 시나리오에 이상적이다. 예를 들어, 실시간 멀티플레이어 게임에서 플레이어의 위치 좌표를 계속해서 전송하는 경우, 중간에 일부 좌표 데이터가 유실되거나 순서가 바뀌어 도착하더라도 최종적으로 도착한 최신 좌표만 사용하면 되므로 문제가 되지 않는다. 일부 데이터 손실이 허용되는 실시간 미디어 전송에도 활용될 수 있다.8

- **API 명세 및 코드 예제:** 데이터그램 통신은 `transport.datagrams` 속성을 통해 이루어진다. 이 속성은 `WebTransportDatagramDuplexStream` 객체를 반환하며, 여기에는 `readable`과 `writable`이라는 두 개의 스트림이 포함되어 있다.

  - **데이터 전송 (쓰기):** `transport.datagrams.writable`은 `WritableStream` 객체다. `getWriter()`를 통해 writer를 얻고, `write()` 메서드로 `Uint8Array` 형식의 데이터를 서버에 전송한다.10

  ```JavaScript
  // 데이터그램 전송 예제
  const writer = transport.datagrams.writable.getWriter();
  const data1 = new Uint8Array();
  const data2 = new Uint8Array();
  
  writer.write(data1);
  writer.write(data2);
  ```

  - **데이터 수신 (읽기):** `transport.datagrams.readable`은 `ReadableStream` 객체다. `getReader()`를 통해 reader를 얻고, `async/await`와 함께 `read()` 메서드를 반복 호출하여 서버로부터 들어오는 데이터그램을 수신한다.10

  ```JavaScript
  // 데이터그램 수신 예제
  const reader = transport.datagrams.readable.getReader();
  while (true) {
      const { value, done } = await reader.read();
      if (done) {
          break; // 스트림 종료
      }
      // value는 Uint8Array 형식의 수신된 데이터
      console.log(value);
  }
  ```

  이처럼 표준 `Streams API`를 활용함으로써, 데이터 흐름이 너무 빠를 경우 자동으로 속도를 조절하는 백프레셔(backpressure) 기능을 자연스럽게 이용할 수 있다.


스트림 API는 데이터의 전송 순서와 성공적인 전달이 반드시 보장되어야 하는 경우에 사용된다.

- **작동 원리:** QUIC의 `STREAM` 프레임을 기반으로 동작하며, TCP와 유사하게 데이터의 순차성과 신뢰성을 보장한다. 하지만 QUIC의 스트림은 각기 독립적으로 동작하므로, 한 스트림의 문제가 다른 스트림에 영향을 주지 않는다는 큰 장점이 있다.8


데이터가 한쪽 방향으로만 흐르는 스트림이다.

- **클라이언트 시작 (Outgoing):** 클라이언트가 서버로 데이터를 보내기만 할 때 사용된다. `transport.createUnidirectionalStream()` 메서드를 호출하면, `WebTransportSendStream` 객체에 대한 프로미스가 반환된다. 이 객체의 `writable` 속성을 통해 데이터를 전송할 수 있다. 클라이언트의 로그나 센서 데이터를 서버로 전송하는 등의 용도에 적합하다.10

  ```JavaScript
  // 단방향 송신 스트림 생성 및 데이터 전송
  const uniStream = await transport.createUnidirectionalStream();
  const writer = uniStream.getWriter(); // WebTransportSendStream은 WritableStream이다.
  await writer.write(new Uint8Array());
  await writer.close();
  ```

- **서버 시작 (Incoming):** 서버가 클라이언트로 데이터를 일방적으로 푸시할 때 사용된다. 클라이언트는 `transport.incomingUnidirectionalStreams` 속성을 통해 이를 수신한다. 이 속성은 `ReadableStream`이며, 이 스트림에서 읽어 들인 각 청크(chunk)는 `WebTransportReceiveStream` 객체다. 이 객체의 `readable` 속성을 통해 서버가 보낸 실제 데이터를 읽을 수 있다. 서버 푸시 알림이나 미디어 스트리밍에 활용될 수 있다.10


클라이언트와 서버가 동일한 스트림을 통해 데이터를 양방향으로 주고받을 수 있다.

- `transport.createBidirectionalStream()`를 호출하거나, 서버에서 시작된 스트림을 `transport.incomingBidirectionalStreams`를 통해 수신하여 `WebTransportBidirectionalStream` 객체를 얻는다. 이 객체는 `readable`과 `writable` 속성을 모두 가지고 있어, 하나의 스트림으로 읽기와 쓰기가 모두 가능하다. 특정 요청에 대한 응답을 받는 패턴이나, 실시간 채팅 채널과 같은 양방향 대화에 적합하다.10


스트림의 종료는 두 가지 방식으로 관리된다.

- **정상 종료 (Graceful Close):** `writer.close()` 메서드를 호출한다. 이 경우, 쓰기 버퍼에 남아있는 모든 데이터가 성공적으로 전송된 후에 스트림이 정상적으로 닫힌다. 상대방은 스트림의 끝(end-of-stream)을 감지할 수 있다.10

- **비정상 종료 (Abrupt Close):** `writer.abort()` 메서드를 호출한다. 이 경우, 버퍼에 남아있는 데이터는 모두 폐기되고, 상대방에게는 스트림이 즉시 중단되었음을 알리는 `RESET_STREAM` 신호가 전송된다.10 이 기능은 '부분적 신뢰성(partial reliability)'을 구현하는 데 유용하게 사용될 수 있다. 예를 들어, 실시간 비디오 스트리밍 중 네트워크 지연으로 인해 특정 비디오 프레임의 전송이 너무 늦어졌을 때, 더 이상 필요 없어진 해당 프레임이 담긴 스트림을 

  `abort()`하여 즉시 취소하고 다음 프레임 전송에 집중할 수 있다.21

WebTransport API가 이처럼 표준 `Streams API` 명세를 적극적으로 채택한 것은 매우 중요한 설계 결정이다. 이는 개발자에게 `fetch` API의 스트리밍 응답 처리 등에서 이미 익숙한 비동기 프로그래밍 패턴을 일관되게 제공한다.13 더 중요한 것은, `Streams API`에 내장된 백프레셔 메커니즘을 자연스럽게 활용할 수 있다는 점이다. 수신 측이 데이터를 처리하는 속도보다 송신 측이 더 빠르게 데이터를 보내면, 스트림의 내부 버퍼가 가득 차게 되고, 이는 자동으로 송신 측의 `writer.write()` 프로미스가 해결되는 것을 지연시켜 전송 속도를 조절한다. 이는 별도의 백프레셔 구현이 없어 메시지 폭주 시 메모리 과부하 및 CPU 사용량 급증 문제를 겪었던 WebSocket API의 고질적인 단점을 해결하는 핵심적인 개선점이다.35 결국 WebTransport는 새로운 통신 기능을 제공하는 것을 넘어, 현대 웹 플랫폼의 비동기 처리 표준과 깊게 통합됨으로써 개발자 경험을 향상시키고 애플리케이션의 전반적인 안정성을 높인다.


WebTransport의 진정한 가치를 이해하기 위해서는 기존의 주요 실시간 통신 기술들과의 다각적인 비교가 필수적이다. 각 기술은 고유한 장단점과 최적의 사용 사례를 가지고 있으며, WebTransport는 이들 사이에서 독자적인 위치를 점하고 있다.


WebSocket은 오랫동안 실시간 웹 통신의 표준이었지만, WebTransport는 여러 측면에서 이를 능가하는 성능과 유연성을 제공한다.

- **성능:** 가장 큰 차이는 초기 연결 설정 시간과 HOL 블로킹 문제에서 비롯된다. WebTransport는 QUIC의 1-RTT 또는 0-RTT 핸드셰이크 덕분에 TCP와 TLS 핸드셰이크를 순차적으로 거쳐야 하는 WebSocket보다 훨씬 빠르게 연결을 수립할 수 있다.8 또한, QUIC의 독립적인 스트림 다중화 덕분에 네트워크 혼잡이나 패킷 손실이 발생하는 환경에서도 HOL 블로킹 없이 안정적인 성능을 유지한다. 반면 WebSocket은 TCP의 HOL 블로킹 문제에 그대로 노출된다.5
- **효율성 및 유연성:** WebSocket은 단일 연결에 단일 논리적 스트림만을 제공한다. 여러 종류의 데이터를 독립적으로 처리하려면 여러 개의 WebSocket 연결을 생성해야 하는데, 이는 서버와 클라이언트 모두에게 상당한 리소스 부담을 준다.16 WebTransport는 단일 연결 내에서 다수의 독립적인 스트림을 생성할 수 있어 훨씬 효율적이다. 더불어, 애플리케이션의 요구에 따라 신뢰성 있는 스트림과 비신뢰성 데이터그램을 선택적으로 사용할 수 있는 유연성은 WebSocket에는 없는 독보적인 장점이다.4
- **생태계:** 현재 시점에서 WebSocket의 유일한 우위는 생태계의 성숙도이다. 거의 모든 브라우저와 서버 프레임워크에서 안정적으로 지원되며, 풍부한 라이브러리와 자료를 쉽게 찾을 수 있다. 반면, WebTransport는 Safari 브라우저에서의 지원이 아직 완전하지 않고, 서버 측 생태계도 성장하는 단계에 있다.1


WebTransport와 WebRTC는 저지연 통신을 목표로 한다는 점에서 유사해 보이지만, 근본적인 통신 모델과 설계 철학이 다르다.

- **통신 모델:** WebTransport는 명확하게 클라이언트-서버(Client-Server) 통신 모델을 위해 설계되었다.8 서버는 공인 IP 주소를 가지고 클라이언트의 연결을 기다리는 전통적인 웹 아키텍처에 완벽하게 부합한다. 반면, WebRTC는 본질적으로 P2P(Peer-to-Peer) 통신을 위해 설계된 기술이다. 클라이언트-서버 통신을 위해 WebRTC를 사용하려면, 서버가 P2P 통신을 위한 복잡한 시그널링 과정과 STUN/TURN/ICE/SCTP 프로토콜 스택을 구현해야 하는 부담이 있다. 이는 WebTransport에 비해 훨씬 복잡하고 비효율적이다.4
- **사용 사례:** P2P 기반의 화상 회의나 음성 통화, 브라우저 간 직접 파일 전송 등은 여전히 WebRTC가 가장 적합한 영역이다. 그러나 저지연이 필수적인 온라인 게임, 서버가 클라이언트에게 미디어를 스트리밍하는 서비스, 다수의 사용자가 서버를 중심으로 데이터를 동기화하는 협업 도구 등 클라이언트-서버 통신이 중심이 되는 대부분의 시나리오에서는 WebTransport가 훨씬 더 간단하고 효율적인 솔루션을 제공한다.21


Long Polling과 SSE(Server-Sent Events)는 WebTransport나 WebSocket과 비교할 때 레거시 기술로 분류될 수 있다.

- **Long Polling:** 반복적인 HTTP 연결 생성으로 인한 높은 지연 시간과 서버 부하 때문에 현대적인 실시간 애플리케이션에는 거의 사용되지 않는다.1
- **SSE:** 서버에서 클라이언트로의 단방향(unidirectional) 통신에 특화되어 있다.1 뉴스 피드나 주식 시세 업데이트처럼 클라이언트의 입력이 거의 필요 없는 경우 유용할 수 있지만, 양방향 통신이 필요한 대부분의 애플리케이션에는 적합하지 않다.

이 두 기술은 WebTransport나 WebSocket을 사용할 수 없는 매우 제한적인 환경을 위한 최후의 폴백(fallback) 수단 외에는 새로운 프로젝트에서 주력 기술로 고려하기 어렵다.36


| 특성               | WebTransport              | WebSocket                  | WebRTC (Data Channel)      | Server-Sent Events (SSE) | Long Polling           |
| ------------------ | ------------------------- | -------------------------- | -------------------------- | ------------------------ | ---------------------- |
| **기반 프로토콜**  | HTTP/3 over QUIC/UDP      | HTTP/1.1 over TCP          | SCTP over DTLS/UDP         | HTTP/1.1 over TCP        | HTTP/1.1 over TCP      |
| **통신 모델**      | 클라이언트-서버           | 클라이언트-서버            | P2P (클라이언트-서버 가능) | 단방향 (서버-->>클라이언트) | 요청-응답 (에뮬레이션) |
| **초기 연결 지연** | 낮음 (0-1 RTT)            | 중간 (2-3 RTT)             | 높음 (시그널링 필요)       | 중간 (1-2 RTT)           | 높음 (매번 연결)       |
| **HOL 블로킹**     | 없음                      | TCP 계층에 존재            | 없음 (SCTP 기반)           | TCP 계층에 존재          | HTTP 계층에 존재       |
| **스트림 다중화**  | 네이티브 지원             | 미지원 (애플리케이션 계층) | 네이티브 지원              | 미지원                   | 미지원                 |
| **신뢰성 제어**    | 신뢰성/비신뢰성 선택 가능 | 신뢰성만 지원              | 신뢰성/비신뢰성 선택 가능  | 신뢰성만 지원            | 신뢰성만 지원          |
| **주요 사용 사례** | 게임, 스트리밍, 협업 도구 | 채팅, 일반 실시간 앱       | 화상회의, P2P 파일 공유    | 뉴스 피드, 알림          | 레거시, 폴백           |
| **생태계 성숙도**  | 성장 중                   | 성숙                       | 성숙                       | 성숙                     | 성숙                   |

이 표는 각 기술의 핵심적인 특징을 요약하여 보여준다. WebTransport는 QUIC의 장점을 바탕으로 낮은 연결 지연, HOL 블로킹 부재, 네이티브 다중화, 유연한 신뢰성 제어 등 다방면에서 기술적 우위를 점하고 있음을 명확히 알 수 있다.


아무리 뛰어난 기술이라도 실제 현장에 적용하기 위해서는 표준화, 생태계, 네트워크 호환성 등 실용적인 측면을 반드시 고려해야 한다. WebTransport는 아직 발전 중인 기술인 만큼 이러한 현실적인 문제들을 안고 있다.


WebTransport의 표준화는 두 개의 주요 국제 표준화 기구인 IETF와 W3C(World Wide Web Consortium)의 긴밀한 협력을 통해 진행되고 있다.

- **IETF WEBTRANS WG:** 프로토콜의 저수준 명세를 담당한다. `WebTransport over HTTP/3` 26와 네트워크 제약 환경을 위한 

  `WebTransport over HTTP/2` 39 등의 프로토콜 사양을 정의한다.

- **W3C WebTransport WG:** 웹 개발자가 사용하게 될 고수준 JavaScript API 명세를 담당한다.29

이 두 그룹은 프로토콜과 API가 서로 긴밀하게 연동되며 발전할 수 있도록 협력하고 있다. 현재(2025년 기준) 관련 명세들은 대부분 'Working Draft(초안)' 상태로, 향후 세부 내용이 변경될 가능성이 있다. 이는 아직 기술이 완전히 안정화되지 않았음을 의미하며, 프로덕션 환경에 도입할 때 주의가 필요함을 시사한다.29


- **브라우저 지원:** Chrome, Firefox, Edge 등 주요 데스크톱 브라우저들은 이미 WebTransport를 지원하고 있다. 그러나 가장 큰 걸림돌은 **Safari의 미지원**이다. Apple의 Safari와 Safari on iOS가 WebTransport를 지원하지 않는 한, 이 기술을 보편적인 웹 서비스에 전면적으로 도입하기는 어렵다. 이는 WebTransport 채택의 가장 중요한 제약 사항 중 하나다.37

- **서버 지원:** 서버 측 생태계 역시 아직 성장 단계에 있다. 특히 웹 서버 개발에서 가장 널리 사용되는 **Node.js의 네이티브 지원 부재**가 큰 허들로 작용한다.1 물론 

  `aioquic`(Python), `quiche`(Rust), `webtransport-go`(Go) 등 다양한 언어로 구현된 라이브러리들이 등장하고 있지만 42, WebSocket처럼 프레임워크에 기본적으로 통합된 수준에는 이르지 못했다. 이는 서버 구축 및 운영의 복잡성을 증가시키는 요인이다.


WebTransport는 UDP 기반의 QUIC 프로토콜 위에서 동작하기 때문에, 기존 TCP 기반 인터넷 환경에서 발생할 수 있는 여러 네트워크 제약에 직면할 수 있다.

- **방화벽의 UDP 차단 문제:** 많은 기업이나 공공기관의 네트워크는 보안상의 이유로 잘 알려진 포트(예: DNS용 53번)를 제외한 대부분의 UDP 트래픽을 차단하는 정책을 사용한다.45 이러한 환경에서는 QUIC 연결 자체가 불가능하므로 WebTransport 역시 동작할 수 없다.
- **프록시 호환성:** 기존의 많은 HTTP 프록시 서버들은 TCP 트래픽 처리에 최적화되어 있어, UDP 기반의 QUIC 패킷을 제대로 이해하거나 전달하지 못할 수 있다. 이 또한 WebTransport 연결 실패의 원인이 될 수 있다.7
- **HTTP/2 폴백 메커니즘:** 이러한 현실적인 네트워크 제약들을 극복하기 위해, WebTransport 명세에는 **HTTP/2를 이용한 폴백(Fallback) 메커니즘**이 포함되어 있다.39 만약 클라이언트가 서버와 QUIC(HTTP/3) 연결을 시도했으나 방화벽이나 프록시 등의 문제로 실패할 경우, 브라우저는 자동으로 동일한 WebTransport API를 유지하면서 내부적으로는 TCP 기반의 HTTP/2 연결로 전환하여 통신을 시도한다.

이 HTTP/2 폴백 기능은 WebTransport의 실용성을 크게 높여주는 중요한 안전장치다. 하지만 이 폴백에는 명확한 트레이드오프가 존재한다. HTTP/2 폴백이 동작하는 순간, WebTransport는 TCP 위에서 동작하게 되므로 QUIC의 가장 핵심적인 이점인 **'HOL 블로킹 해결' 능력을 상실하게 된다**. TCP는 본질적으로 HOL 블로킹 문제를 가지고 있기 때문이다.25 결과적으로, 폴백 상태의 WebTransport는 API의 일관성은 유지되지만 성능 특성은 WebSocket과 매우 유사해진다. 따라서 개발자는 WebTransport를 도입할 때, 최상의 시나리오(QUIC 연결)와 차선책 시나리오(HTTP/2 폴백) 모두를 염두에 두고 애플리케이션의 성능을 설계하고 테스트해야 한다. 폴백은 기능적 연속성을 보장하기 위한 수단일 뿐, 성능 저하를 감수한 타협점임을 명확히 인지해야 한다.


WebTransport는 웹의 보안 모델 내에서 안전하게 동작하도록 설계되었다. 이는 내재된 프로토콜 수준의 보안과 개발자가 선택할 수 있는 유연한 인증 메커니즘을 통해 달성된다.


WebTransport의 모든 연결은 기반 프로토콜인 QUIC을 통해 TLS 1.3 이상으로 암호화가 강제된다. 이는 WebSocket에서 `ws://`와 같이 암호화되지 않은 연결을 허용했던 것과 달리, 모든 통신이 기본적으로 암호화됨을 보장한다. 이를 통해 중간자 공격(Man-in-the-middle attack)이나 데이터 도청(eavesdropping)의 위험을 원천적으로 차단하여 높은 수준의 보안을 제공한다.2


일반적으로 웹 브라우저는 서버에 연결할 때, 서버가 제시하는 TLS 인증서가 신뢰할 수 있는 공인 인증 기관(Certificate Authority, CA)에 의해 발급되었는지를 검증한다. 하지만 모든 서버가 공인 CA 인증서를 발급받을 수 있는 것은 아니다. 예를 들어, 외부에서 접근할 수 없는 사설 네트워크 내의 서버나, 클라우드 환경에서 동적으로 생성되고 소멸하는 수명이 짧은 가상 머신(VM) 등이 그렇다.

WebTransport는 이러한 시나리오를 지원하기 위해 `serverCertificateHashes`라는 강력하고 유연한 옵션을 제공한다.27 개발자는 `new WebTransport()` 생성자에 서버의 자체 서명된(self-signed) 인증서의 해시(SHA-256) 값을 배열 형태로 전달할 수 있다. 브라우저는 서버로부터 인증서를 받으면, CA 체인을 검증하는 대신 이 해시 값이 일치하는지만을 확인한다. 해시 값이 일치하면, 브라우저는 해당 서버를 신뢰하고 안전한 연결을 수립한다. 이 기능은 개발자가 CA에 의존하지 않고도 종단 간 암호화된 보안 연결을 구축할 수 있게 해주며, 특히 libp2p와 같은 P2P 네트워크에서 노드들이 서로를 신뢰하고 연결하는 데 매우 유용하게 활용된다.46


WebTransport는 연결을 설정하는 초기 `CONNECT` 요청 시, HTTP 쿠키나 `Authorization` 헤더와 같은 전통적인 HTTP 인증 정보를 함께 전송하는 메커니즘을 지원하지 않는다.9 이는 기존 웹 애플리케이션에서 널리 사용되는 세션 관리 방식과 충돌하여 개발자에게 새로운 과제를 제시한다.

이러한 제약 하에서 WebTransport 세션을 안전하게 인증하기 위한 권장 패턴은 다음과 같다.

1. **토큰 획득:** 사용자는 먼저 일반적인 HTTPS 요청(예: 로그인 API)을 통해 서버에 인증한다. 서버는 인증에 성공한 사용자에게 단기간 유효한 세션 토큰(예: JWT - JSON Web Token)을 발급하여 응답한다.
2. **연결 수립:** 클라이언트는 `new WebTransport(url)`를 호출하여 서버와 WebTransport 연결을 수립한다. 이 단계에서는 아직 사용자가 누구인지 인증되지 않은 상태다.
3. **토큰 전송:** 연결의 `ready` 프로미스가 이행되면, 클라이언트는 신뢰성 있는 스트림(`createUnidirectionalStream` 또는 `createBidirectionalStream`)을 하나 생성하여 1단계에서 획득한 세션 토큰을 서버로 전송한다.
4. **서버 검증:** 서버는 해당 스트림을 통해 토큰을 수신하고, 그 유효성을 검증한다. 토큰이 유효하면, 서버는 해당 WebTransport 세션을 '인증된 상태'로 승격시키고 이후의 통신을 허용한다. 만약 토큰이 유효하지 않거나 전송되지 않으면, 서버는 보안 정책에 따라 연결을 즉시 종료해야 한다.

이러한 인증 방식은 표면적으로는 번거로워 보일 수 있지만, 중요한 아키텍처적 이점을 가진다. 쿠키 기반 인증은 브라우저와 서버 간의 암묵적인 상태 공유에 의존하는 반면, WebTransport의 인증 방식은 이러한 암묵적인 컨텍스트를 제거했다.9 대신, 개발자는 인증 상태를 토큰이라는 매개체를 통해 명시적으로 관리해야 한다. 이는 각 연결 요청이 자체적으로 인증 정보를 포함하여 처리되는 RESTful API의 상태 비저장(stateless) 원칙과 일맥상통한다. 서버는 공유된 쿠키 저장소 같은 상태에 의존할 필요 없이, 수신한 토큰만으로 세션을 검증할 수 있어 더 깔끔하고 확장 가능한 서버 아키텍처 설계를 유도한다. 즉, WebTransport의 이러한 제약은 오히려 더 나은 소프트웨어 설계를 장려하는 긍정적인 측면을 가지고 있다.


WebTransport의 다채로운 기능(다중 스트림, 신뢰성/비신뢰성 전송)은 다양한 실시간 애플리케이션의 아키텍처를 혁신할 잠재력을 가지고 있다. 각 기능이 특정 사용 사례에서 어떻게 활용될 수 있는지 구체적으로 분석한다.

7.1. 실시간 멀티플레이어 게임

실시간 멀티플레이어 게임은 WebTransport의 장점을 가장 극적으로 활용할 수 있는 분야다. 단일 WebTransport 연결 내에서 전송하는 데이터의 종류에 따라 스트림과 데이터그램을 전략적으로 혼합하여 사용할 수 있다.

- **아키텍처:**
  - **데이터그램 활용:** 플레이어의 위치, 방향, 총알 발사 이벤트 등 순서 보장보다는 즉각적인 전달이 훨씬 중요한 데이터는 비신뢰성 데이터그램으로 전송한다. 약간의 패킷 손실은 다음 패킷으로 보정될 수 있으므로, 지연 시간을 최소화하는 것이 게임 경험에 더 유리하다.8
  - **신뢰성 스트림 활용:** 채팅 메시지, 아이템 획득, 게임 시작/종료와 같은 중요한 게임 이벤트는 반드시 순서대로, 누락 없이 전달되어야 하므로 신뢰성 스트림을 사용한다.48
  - **독립 스트림 활용:** 게임 상태 업데이트, 플레이어 음성 채팅, 텍스트 채팅 등 논리적으로 다른 종류의 데이터를 각각 별도의 스트림으로 분리하여 전송한다. 이를 통해, 예를 들어 용량이 큰 음성 데이터 전송이 지연되더라도 중요한 게임 상태 업데이트나 채팅 메시지 전달에 영향을 주지 않아 HOL 블로킹을 효과적으로 방지할 수 있다.48


WebTransport는 WebCodecs API와 결합될 때, 브라우저 내에서 미디어 파이프라인을 직접 제어하는 강력한 초저지연 스트리밍 솔루션을 구축할 수 있다.

- **아키텍처:**
  - 인코딩된 비디오와 오디오 프레임을 각각 별도의 스트림이나 데이터그램으로 분리하여 서버로 전송하거나 서버로부터 수신한다. 이는 미디어 데이터의 전송 지연을 최소화하고, 오디오와 비디오 간의 싱크를 유연하게 제어할 수 있게 한다.21
  - 자막, 시청자 상호작용 데이터(예: 투표, 이모티콘), 광고 정보 등 부가적인 데이터는 미디어 스트림과 완전히 독립된 별도의 신뢰성 스트림을 통해 전송한다. 이를 통해 대용량 미디어 데이터가 다른 중요한 데이터의 전송을 방해하는 문제를 원천적으로 차단할 수 있다.30


Google Docs나 Figma와 같은 실시간 협업 도구는 여러 사용자의 동시 편집 내용을 빠르고 정확하게 동기화해야 한다.

- **아키텍처:**
  - 사용자 간의 편집 내용(Op - Operational Transformation 데이터 또는 CRDT - Conflict-free Replicated Data Type 데이터)은 신뢰성 있는 양방향 스트림을 통해 안정적으로 동기화한다. 데이터의 순서와 무결성이 매우 중요하기 때문이다.6
  - 다른 사용자의 커서 위치 정보, 현재 선택 영역 표시 등 상대적으로 덜 중요하지만 실시간성이 요구되는 보조 데이터는 별도의 스트림으로 관리할 수 있다. 이를 통해 주 편집 데이터의 흐름을 방해하지 않으면서 부드러운 사용자 경험을 제공할 수 있다.


수많은 IoT(사물 인터넷) 기기가 서버로 주기적인 센서 데이터를 전송하는 시나리오에서도 WebTransport는 효율적인 솔루션을 제공한다.

- **아키텍처:**
  - QUIC의 가벼운 0-RTT 연결 재개를 활용하여, 데이터를 전송할 때만 간헐적으로 연결하는 IoT 기기들의 연결/해제 오버헤드를 최소화할 수 있다. 이는 특히 배터리로 동작하는 기기의 전력 소모를 줄이는 데 큰 도움이 된다.30
  - 온도, 습도 등 주기적으로 수집되지만 약간의 손실이 허용되는 센서 데이터는 비신뢰성 데이터그램을 사용하여 전송 효율을 극대화할 수 있다. 반면, 기기의 상태를 제어하는 중요한 명령어는 신뢰성 스트림을 통해 전달하여 안정성을 확보한다.48



WebTransport는 단순한 기술적 개선을 넘어, 실시간 웹 애플리케이션을 구축하는 방식에 근본적인 변화를 가져올 잠재력을 지닌 프로토콜 프레임워크다. 그 핵심 가치는 QUIC 프로토콜의 성능과 유연성을 웹의 안전한 보안 모델 내에서 개발자에게 제공한다는 점에 있다.

- **성능 혁신:** TCP의 고질적인 HOL 블로킹 문제를 근본적으로 해결하고, 1-RTT/0-RTT 핸드셰이크를 통해 연결 지연을 최소화함으로써 기존 기술로는 도달하기 어려웠던 수준의 저지연 통신을 실현한다.
- **아키텍처 유연성:** 단일 연결 내에서 다수의 독립적인 스트림과 비신뢰성 데이터그램을 동시에 지원함으로써, 개발자는 애플리케이션의 요구사항에 맞춰 통신 방식을 정교하게 설계할 수 있다. 이는 복잡한 통신 로직을 애플리케이션 계층에서 프로토콜 계층으로 옮겨와, 더 단순하고 견고한 아키텍처를 가능하게 한다.

이러한 특성들은 WebTransport를 WebSocket의 단순한 상위 호환이 아닌, 실시간 게임, 인터랙티브 스트리밍, 대규모 협업 도구 등 차세대 웹 애플리케이션을 위한 새로운 표준 통신 프로토콜로 자리매김하게 할 것이다.


WebTransport는 강력한 잠재력을 가지고 있지만, 아직 발전 중인 기술이므로 도입 시 신중한 전략이 필요하다.

- **점진적 도입 고려:** 현재 WebSocket은 여전히 가장 안정적이고 보편적인 실시간 통신 솔루션이다. 따라서 모든 것을 한 번에 WebTransport로 전환하기보다는, 성능이 극도로 중요하거나 다중 스트림 및 비신뢰성 전송이 필수적인 특정 사용 사례부터 점진적으로 도입하는 것이 현명하다. 예를 들어, 기존 채팅 기능은 WebSocket을 유지하되, 새로 추가되는 저지연 비디오 피드 기능에만 WebTransport를 적용하는 방식을 고려할 수 있다.35
- **생태계 성숙도 확인:** 기술 도입을 결정하기 전에, 목표로 하는 모든 플랫폼(특히 Apple의 Safari)에서의 브라우저 지원 현황을 면밀히 검토해야 한다. 또한, 주력으로 사용하는 서버 프레임워크(특히 Node.js)의 WebTransport 지원 수준과 안정성을 반드시 확인해야 한다. 생태계가 충분히 성숙하지 않았을 경우, 개발 및 운영 비용이 예상보다 커질 수 있다.42
- **폴백 전략 필수:** WebTransport의 기반인 QUIC은 기업 방화벽 등 특정 네트워크 환경에서 차단될 수 있다. 따라서 안정적인 서비스 제공을 위해서는 HTTP/2 폴백 시나리오에 대한 충분한 테스트와 성능 계획을 반드시 수립해야 한다. 폴백 시에는 HOL 블로킹이 다시 발생하여 성능이 저하될 수 있음을 인지하고, 이에 대한 대비책을 마련해야 한다.
- **새로운 패러다임 수용:** WebTransport를 단순히 '더 빠른 WebSocket'으로만 접근하는 것은 그 잠재력을 온전히 활용하지 못하는 것이다. 다중 스트림과 데이터그램이라는 새로운 도구를 활용하여 기존 애플리케이션의 통신 로직을 어떻게 더 효율적이고 단순하게 재구성할 수 있을지 고민하는 것이 중요하다. 이는 단순히 코드를 바꾸는 것을 넘어, 애플리케이션 아키텍처의 복잡성을 줄이고 성능을 극대화하는 새로운 기회가 될 것이다.

결론적으로, WebTransport는 실시간 웹의 미래를 향한 중요한 발걸음이다. 비록 생태계와 표준화가 완성되기까지 시간이 더 필요하겠지만, 그 기술적 우위는 명확하다. 개발자와 아키텍트는 이 새로운 기술의 특성을 깊이 이해하고, 현실적인 제약과 가능성을 균형 있게 평가하여 미래의 웹 서비스를 준비해 나가야 할 것이다.


1. 웹소켓 대 서버 전송 이벤트 대 롱 폴링 대 WebRTC 대 WebTransport 비교 - GeekNews, accessed August 6, 2025, https://news.hada.io/topic?id=13888
2. What is WebTransport and How Does it Work? - PubNub, accessed August 6, 2025, https://www.pubnub.com/guides/webtransport/
3. WebTransport Explained: Low-Latency Communication over HTTP/3 - GoCodeo, accessed August 6, 2025, https://www.gocodeo.com/post/webtransport-explained-low-latency-communication-over-http-3
4. 롱 폴링 vs 웹소켓 vs Server-Sent Events vs Web Transport vs WebRTC, accessed August 6, 2025, https://velog.io/@pon06188/real-time-communications
5. TPAC 2021: WebTransport Working Group Updates - W3C, accessed August 6, 2025, https://www.w3.org/2021/10/TPAC/demos/webtransport.html
6. Building Real-Time Web Apps with WebTransport (Replacing WebSockets?), accessed August 6, 2025, https://dev.to/mukhilpadmanabhan/building-real-time-web-apps-with-webtransport-replacing-websockets-3348
7. WebTransport is a Game Changer Protocol - YouTube, accessed August 6, 2025, https://www.youtube.com/watch?v=SEF8VBYlLik
8. Using WebTransport | Capabilities - Chrome for Developers, accessed August 6, 2025, https://developer.chrome.com/docs/capabilities/web-apis/webtransport
9. WebTransport. What will drive next-generation... | by Poby's Home - Medium, accessed August 6, 2025, https://poby.medium.com/webtransport-472621795cf7
10. WebTransport 사용 | Capabilities | Chrome for Developers, accessed August 6, 2025, https://developer.chrome.com/docs/capabilities/web-apis/webtransport?hl=ko
11. 롱 폴링 vs 웹소켓 vs Server-Sent Events vs Web Transport vs WebRTC - velog, accessed August 6, 2025, https://velog.io/@pon06188/real-time-communication
12. news.hada.io, accessed August 6, 2025, [https://news.hada.io/topic?id=13888#:~:text=WebTransport%20API%EB%9E%80%3F,%EA%B8%B0%EB%8A%A5%EC%9D%84%20%EA%B0%80%EB%8A%A5%ED%95%98%EA%B2%8C%20%ED%95%A8.](https://news.hada.io/topic?id=13888#:~:text=WebTransport API란%3F,기능을 가능하게 함.)
13. WebTransport API - Hacker News, accessed August 6, 2025, https://news.ycombinator.com/item?id=23666364
14. What is Replacing WebSockets? Deep Dive Into WebTransport, HTTP/3 & Real-Time Protocols (2025 Guide) - VideoSDK, accessed August 6, 2025, https://www.videosdk.live/developer-hub/websocket/what-is-replacing-websockets
15. EMPIRICAL ANALYSIS OF THE IMPACT OF PACKET LOSS ON WEBTRANSPORT USING SOCKET.IO - DiVA portal, accessed August 6, 2025, http://www.diva-portal.org/smash/get/diva2:1876796/FULLTEXT01.pdf
16. https://caniuse.com/webtransport Yeah, it seems there's an older entry in ther... | Hacker News, accessed August 6, 2025, https://news.ycombinator.com/item?id=32869523
17. HTTP 3.0 기반 웹소켓, WebTransport 프로토콜의 표준화 논의 시작, accessed August 6, 2025, http://weekly.tta.or.kr/weekly/files/20201423081406_weekly.pdf
18. [Network] HTTP/3 and QUIC - 잼있는 블로그 - 티스토리, accessed August 6, 2025, https://shinjam.tistory.com/entry/Network-HTTP3-and-QUIC
19. WebTransport Protocol - InterviewNoodle, accessed August 6, 2025, https://interviewnoodle.com/webtransport-protocol-3d0169dc3b56
20. HTTP/3를 통해 빠르고, 안정적이고, 안전한 웹 경험을 제공하세요 ..., accessed August 6, 2025, https://www.akamai.com/ko/blog/performance/deliver-fast-reliable-secure-web-experiences-http3
21. Replacing WebRTC - Media over QUIC, accessed August 6, 2025, https://quic.video/blog/replacing-webrtc/
22. Running a QUIC Client – quic-go docs, accessed August 6, 2025, https://quic-go.net/docs/quic/client/
23. draft-vvv-webtransport-quic-00 - Datatracker, accessed August 6, 2025, https://datatracker.ietf.org/doc/html/draft-vvv-webtransport-quic-00
24. Enabling HTTP/3 for Fastly services | Fastly Documentation, accessed August 6, 2025, https://www.fastly.com/documentation/guides/full-site-delivery/performance/enabling-http3-for-fastly-services/
25. How does HTTP2 solve Head of Line blocking (HOL) issue - Stack Overflow, accessed August 6, 2025, https://stackoverflow.com/questions/45583861/how-does-http2-solve-head-of-line-blocking-hol-issue
26. draft-ietf-webtrans-http3-13 - WebTransport over HTTP/3, accessed August 6, 2025, https://datatracker.ietf.org/doc/draft-ietf-webtrans-http3/
27. WebTransport: WebTransport() constructor - Web APIs - MDN Web Docs - Mozilla, accessed August 6, 2025, https://developer.mozilla.org/en-US/docs/Web/API/WebTransport/WebTransport
28. WebTransport Development Guide With Examples - The Syntax Diaries, accessed August 6, 2025, https://thesyntaxdiaries.com/webtransport-development-guide
29. WebTransport - W3C, accessed August 6, 2025, https://www.w3.org/TR/webtransport/
30. WebTransport: Bridging the Gap Beyond WebRTC & WebSockets - Mindfire Solutions, accessed August 6, 2025, https://www.mindfiresolutions.com/blog/2023/08/webtransport-the-future-of-real-time-communication/
31. What is WebTransport Protocol? - VideoSDK, accessed August 6, 2025, https://www.videosdk.live/developer-hub/webtransport/webtransport-protocol
32. WebTransport: datagrams property - Web APIs | MDN, accessed August 6, 2025, https://developer.mozilla.org/en-US/docs/Web/API/WebTransport/datagrams
33. Exploring the WebTransport API: A New Era of Web Communication, accessed August 6, 2025, https://jsdev.space/webtransport-api/
34. WebTransport API - MDN Web Docs - Mozilla, accessed August 6, 2025, https://developer.mozilla.org/en-US/docs/Web/API/WebTransport_API
35. The WebSocket API (WebSockets) - Web APIs - MDN Web Docs, accessed August 6, 2025, https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API
36. WebSockets vs Server-Sent-Events vs Long-Polling vs WebRTC vs WebTransport | RxDB - JavaScript Database, accessed August 6, 2025, https://rxdb.info/articles/websockets-sse-polling-webrtc-webtransport.html
37. WebTransport | Can I use... Support tables for HTML5, CSS3, etc, accessed August 6, 2025, https://caniuse.com/webtransport
38. WebRTC vs WebTransport: Comparison Guide - VideoSDK, accessed August 6, 2025, https://www.videosdk.live/developer-hub/webtransport/webrtc-vs-webtransport
39. draft-ietf-webtrans-http2-12 - WebTransport over HTTP/2, accessed August 6, 2025, https://datatracker.ietf.org/doc/draft-ietf-webtrans-http2/
40. WebTransport - W3C Wiki, accessed August 6, 2025, https://www.w3.org/wiki/WebTransport
41. PROPOSED WebTransport Working Group Charter - W3C on GitHub, accessed August 6, 2025, https://w3c.github.io/charter-drafts/wt-2024.html
42. Socket.IO with WebTransport, accessed August 6, 2025, https://socket.io/get-started/webtransport
43. How to Implement WebTransport API using JS? - VideoSDK, accessed August 6, 2025, https://www.videosdk.live/developer-hub/webtransport/webtransport-api
44. WebTransport – quic-go docs, accessed August 6, 2025, https://quic-go.net/docs/webtransport/
45. WebRTC UDP Firewall Traversal - IETF, accessed August 6, 2025, https://www.ietf.org/ietf-ftp/slides/slides-semiws-webrtc-udp-firewall-traversal-00.pdf
46. WebTransport - libp2p, accessed August 6, 2025, https://docs.libp2p.io/concepts/transports/webtransport/
47. Question: Best Practices WebTransport Client Authentication? : r/programminghelp - Reddit, accessed August 6, 2025, https://www.reddit.com/r/programminghelp/comments/1lhum2t/question_best_practices_webtransport_client/
48. What is WebTransport and can it replace WebSockets? - Ably, accessed August 6, 2025, https://ably.com/blog/can-webtransport-replace-websockets

