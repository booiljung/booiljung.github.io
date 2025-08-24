# Zenoh 프로토콜
[Zenoh](./index.md)



Zenoh는 이동 중인 데이터(Data in Motion), 정지된 데이터(Data at Rest), 그리고 연산(Computation)을 통합하는 것을 목표로 하는 Pub/Sub/Query 프로토콜로 공식적으로 정의된다.1 이 핵심 개념은 Zenoh를 단순한 메시징 프로토콜이 아닌, 현대 분산 시스템을 위한 포괄적인 "데이터 해방자(data liberator)" 2 또는 "데이터 패브릭(data fabric)" 4으로 위치시킨다.

Zenoh의 근본적인 개발 동기는 기존 프로토콜들이 종종 엔터프라이즈 또는 임베디드 환경에 특화되어 만들어내는 단절된 "연결성의 섬(connectivity islands)"을 극복하는 데 있다.1 Zenoh의 핵심 가치 제안은 강력한 데이터 센터 서버부터 자원이 극도로 제한된 마이크로컨트롤러에 이르기까지, 전체 컴퓨팅 연속체(continuum)에 걸쳐 끊김 없는 통신을 구현하는 것이다.1

이러한 통합 패러다임은 시스템 설계의 근본적인 마찰을 줄이기 위한 전략적 시도에서 비롯되었다. Zenoh의 창시자인 안젤로 코르사로(Angelo Corsaro)는 초기 스마트 시티 시스템 구축 경험이 데이터 흐름, 연산, 저장을 위해 "여러 기술을 이어 붙이는(stitching together technologies)" 고통스러운 과정이었다고 명시적으로 언급했다.6 전통적인 아키텍처는 이동 중인 데이터를 위해 MQTT나 DDS를, 정지된 데이터를 위해 SQL/NoSQL 데이터베이스를, 그리고 연산을 위해 별도의 RPC 프레임워크를 사용하는 경향이 있었다.4 Zenoh는 이러한 세 가지 패러다임(이동, 정지, 연산)을 위치 투명성(location transparency)을 갖춘 단일 추상화 집합 아래 통합함으로써, 아키텍처의 복잡성, 통합 오버헤드, 그리고 개발자의 인지 부하를 극적으로 줄이는 것을 목표로 한다. 이 통합이야말로 Zenoh의 가장 중요한 전략적 지향점이다.


- **제로 오버헤드 네트워크 프로토콜(/zeno/):** Zenoh라는 이름 자체는 최소한의 오버헤드라는 근본적인 설계 목표를 상징한다. 이는 4-6바이트에 불과한 최소 와이어 오버헤드(wire overhead)를 통해 달성되며, 이는 에너지 효율성과 제한된 네트워크 환경에서의 성능에 매우 중요하다.2
- **위치 투명성:** Zenoh는 Pub/Sub 시스템에서 흔히 볼 수 있는 이동 중인 데이터에 대한 위치 투명성을 정지된 데이터로까지 확장한다. 이는 애플리케이션이 데이터베이스나 스토리지 백엔드의 물리적 또는 네트워크 위치를 알 필요 없이 쿼리를 발행할 수 있음을 의미한다.2 Zenoh 인프라스트럭처는 쿼리를 가장 최적의 가용 스토리지 집합으로 라우팅하는 책임을 진다.2 이는 전통적인 데이터 접근 모델로부터의 근본적인 전환이다.

이러한 정지된 데이터에 대한 위치 투명성은 분산 시스템의 패러다임 전환을 의미한다. Pub/Sub는 오랫동안 발행자(Publisher)에 대한 위치 투명성을 제공해왔다. 즉, 구독자는 토픽을 구독할 뿐 발행자의 위치에는 신경 쓸 필요가 없다.2 Zenoh는 이와 동일한 논리를 쿼리에 적용한다.2 애플리케이션은 데이터가 로봇 자체에 있는지, 엣지 게이트웨이에 있는지, 아니면 클라우드 데이터베이스에 저장되어 있는지 알 필요 없이 

`get('robot/**/battery_level')`과 같은 쿼리를 실행할 수 있다. 이는 애플리케이션을 데이터 스토리지 토폴로지로부터 근본적으로 분리시킨다. 이를 통해 성능, 비용 또는 개인 정보 보호 측면에서 가장 합리적인 위치에 데이터를 저장하고 처리하는 진정한 "포그 및 엣지 중심 패러다임(fog and edge-centric paradigms)" 8을 구현할 수 있으며, 이 과정에서 애플리케이션 로직을 변경할 필요가 없다. 이는 네트워크 토폴로지가 동적인 모바일 시스템(로봇, 차량)에 특히 중요하다.


Zenoh는 몇 가지 핵심적인 추상화를 통해 애플리케이션을 구성한다.

- **리소스(Resources):** Zenoh의 기본 단위로, 키(key)와 값(value)으로 구성된다.12
- **키 표현식(Key Expressions):** `robot/sensor/temp`와 같이 계층적이고 경로 기반의 네이밍 시스템을 사용하며, 와일드카드(`*`는 단일 레벨, `**`는 여러 레벨)를 지원하여 리소스 집합을 주소 지정할 수 있다.12
- **발행자(Publishers) & 구독자(Subscribers):** 키 표현식과 연관된 데이터를 생산하고 소비하는, 이동 중인 데이터를 위한 고전적인 엔티티다.3
- **쿼리 가능 객체(Queryables):** 쿼리에 응답할 수 있는 네트워크 엔티티를 나타내는 Zenoh 고유의 추상화다. 이 강력한 기본 요소는 RPC나 맵리듀스(map-reduce)와 같은 패턴을 구현할 수 있게 한다.3
- **스토리지(Storages):** 구독자와 쿼리 가능 객체를 결합한 복합 엔티티로, 데이터 업데이트를 수동적으로 수신하고 과거 데이터에 대한 쿼리에 능동적으로 응답할 수 있다.3



Zenoh는 세 가지 주요 동작 모드를 통해 "스케일 업/다운" 철학을 직접적으로 구현한다. 이 광범위한 적용 범위는 단일 모놀리식 아키텍처로는 불가능하다. `client` 모드는 "스케일 다운"의 핵심으로 장치의 리소스 사용을 최소화하며, `router`와 `peer` 모드는 "스케일 업/아웃"의 핵심으로 고성능의 복잡한 네트워크 구조를 가능하게 한다. 따라서 peer, client, router 모드 간의 선택은 시스템 설계자가 노드별로 지연 시간, 리소스 소비, 네트워크 복잡성 간의 명시적인 트레이드오프를 결정할 수 있는 근본적인 아키텍처 결정이다. 이러한 세분화된 제어는 클라우드-사물 연속체 전체에 걸친 이기종 시스템을 최적화하는 강력한 도구다.

- **Peer 모드:** 기본 모드로, 애플리케이션들이 P2P(peer-to-peer) 방식으로 직접 통신한다. Peer는 연결된 다른 Peer들과의 세션 상태를 자체적으로 유지한다.15 이 모드는 로컬 네트워크 내에서 낮은 지연 시간 통신에 최적이지만, 상태 유지가 집약적일 수 있다.
- **Client 모드:** 애플리케이션이 라우터와 단일 세션만을 유지하는 경량 모드다. 이는 라우팅 및 상태 관리를 라우터에 오프로드하므로 자원이 제한된 장치나 확장성이 가장 중요한 시나리오를 위해 설계되었다.15 예를 들어, Zenoh Python API는 역사적으로 순수 클라이언트로 작동했다.17
- **Router (`zenohd`):** Client, Peer, 그리고 다른 Router 간의 데이터 라우팅을 전담하는 프로세스다. Router는 대규모, 복합, 또는 인터넷을 가로지르는 토폴로지를 생성하기 위한 백본이다. 기본적으로 자동으로 상호 연결되지 않으며, 다른 Router에 연결하도록 명시적으로 구성해야 한다.15 자동차의 Zonal 아키텍처에서는 Router가 중앙 게이트웨이와 유사하게 존(zone) 간 통신을 관리하는 데 사용된다.1


Zenoh의 탐색 메커니즘은 종종 간과되지만, 특히 무선 및 대규모 환경에서 중요한 차별화 요소다. DDS의 탐색 프로토콜은 일부 요인에서 O(N2) 복잡도를 가지며 멀티캐스트 트래픽 폭풍으로 인해 무선 네트워크에서 문제가 되는 "무거운(heavyweight)" 프로토콜로 묘사된다.19 반면 Zenoh는 탐색 트래픽을 90% 이상 줄일 수 있다고 주장한다.19 Indy Autonomous Challenge 21 및 기타 로보틱스 사용 사례 19에서는 제한된 무선 링크에서의 DDS 실패가 Zenoh 채택의 주요 동기였음을 명시적으로 지적한다. 이는 Zenoh의 효율적인 탐색(멀티캐스트와 가십의 조합)이 단순한 최적화가 아니라, 대규모 모바일 및 무선 연결 시스템(예: 로봇 스웜, V2X)을 위한 핵심 기술임을 시사한다. DDS가 단일 로봇 내 유선 LAN에서는 탁월하지만, Zenoh는 네트워크 조건이 불안정한 로봇 간 및 로봇-클라우드 통신이라는 훨씬 더 어려운 문제를 해결하도록 설계되었다.

- **토폴로지 유연성:** Zenoh는 P2P, 라우팅/브로커, 메시 네트워크 등 임의의 토폴로지를 지원한다.9 이는 MQTT의 경직된 허브-앤-스포크 모델이나 DDS의 멀티캐스트 의존적인 클리크(clique) 모델과 대조된다.11

- **스카우팅(Scouting):** 탐색 프로세스다. Peer 모드에서 애플리케이션은 UDP 멀티캐스트(주소 `224.0.0.224:7446`)와 가십 프로토콜을 모두 사용하여 서로를 찾는다.15 Client는 멀티캐스트를 사용하여 Router를 찾는다.15 이 탐색 기능은 보다 통제된 환경을 위해 비활성화할 수 있다(

  `--no-multicast-scouting`).18

- **메시 네트워크를 위한 링크-상태 라우팅:** 직접 연결이 보장되지 않는 메시 네트워크와 같은 까다로운 환경을 위해, Peer들은 경로를 설정하기 위해 링크-상태 프로토콜을 실행할 수 있다.15 이는 시야각 통신이 필요한 플래투닝(platooning)과 같은 모바일 로보틱스 시나리오에서 특히 유용하다.19


- **OSI 계층 유연성:** Zenoh는 데이터 링크(계층 2), 네트워크(계층 3), 전송(계층 4) 등 다양한 계층에서 작동하도록 설계되었다.1 이를 통해 시리얼, 블루투스, TCP/IP, UDP/IP, QUIC, 심지어 CANbus와 같은 다양한 전송 매체 위에서 실행될 수 있다.1 Zenoh-pico를 위한 원시 이더넷 지원도 계획되어 있다.24
- **최소한의 풋프린트:** 효율성에 중점을 둔 설계 덕분에 Zenoh는 매우 제한된 하드웨어에서도 실행될 수 있다. 가장 작은 구현체인 `zenoh-pico`는 8비트 Atmel 마이크로컨트롤러에서 300바이트만큼 작은 풋프린트를 가진다.3
- **와이어 오버헤드:** 데이터 메시지의 최소 와이어 오버헤드는 4-5바이트이며, 이는 긴 키 표현식을 정수 ID에 매핑하고 가변 길이 인코딩(Variable Length Encoding, VLE)을 사용하여 달성된다.3



Zenoh는 다중 신뢰성 수준, 대규모 메시지를 위한 단편화(fragmentation), 효율성 향상을 위한 자동 와이어 레벨 배치(batching)를 지원하는 강력한 Pub/Sub 기본 요소를 제공한다.3

신뢰성은 단순한 on/off 스위치가 아니다. 이는 신뢰성, 진행(progress), 메모리 소비 간의 미묘한 트레이드오프다. Zenoh는 혼잡 제어를 신뢰성과 분리하여, 수신자는 재전송을 요청할 수 있고 송신자는 혼잡 상황에서 데이터 폐기/차단 전략을 제어할 수 있게 한다.26 이러한 분리는 분산 시스템의 근본적인 문제에 대한 정교한 해결책을 제시한다. "느린 소비자" 문제는 신뢰성 있는 전체 시스템을 중단시킬 수 있는데, 이는 생산자가 무한정 버퍼링하거나(무한 메모리 소비), 메시지를 버리거나(신뢰성 상실), 또는 차단(진행 손실)하도록 강요하기 때문이다.26 DDS는 이를 

`RELIABILITY` 및 `HISTORY`와 같은 QoS 정책으로 처리하지만 28, Zenoh는 우려 사항을 명확히 분리한다. 즉, 수신자는 신뢰성을 요청하지만, 송신자와 인프라는 메모리 사용 및 혼잡 동작(폐기 대 차단)을 제어한다.27 Zenoh의 접근 방식은 스트레스 상황에서 시스템 동작에 대해 보다 세분화되고 현실적인 제어를 제공한다. 이는 이기종 시스템에서 강력한 서버(생산자)가 느린 마이크로컨트롤러(소비자)에 의해 차단되어서는 안 된다는 점을 인정한다. 시스템 설계자는 다양한 데이터 스트림에 대해 서로 다른 정책을 정의할 수 있어(예: 중요한 명령어는 차단하고, 고주파 원격 측정 데이터는 폐기), 많은 전면적인 구현보다 더 강력하고 실용적인 신뢰성 접근 방식을 제공한다.


Zenoh는 샤딩(sharding)과 복제(replication)를 지원하는 지리적 분산 스토리지(geo-distributed storages)를 정의하기 위한 기본 요소를 제공한다.3 애플리케이션은 키 표현식과 일치하는 리소스에 대해 쿼리(`get`)를 발행할 수 있으며, 네트워크는 쿼리를 만족시킬 수 있는 가장 가까운 가용 스토리지 또는 쿼리 가능 객체로 쿼리를 라우팅한다.10 Zenoh는 RocksDB, InfluxDB, Filesystem 29 및 커뮤니티에서 요청한 S3 백엔드 30를 위한 기존 플러그인을 통해 플러그형 스토리지 백엔드를 지원한다.


Query/Queryable 메커니즘은 단순한 데이터 검색 이상의 강력한 패턴으로, 들어오는 쿼리에 의해 트리거되는 연산을 애플리케이션이 등록할 수 있도록 한다.3 이는 원격 프로시저 호출(RPC)을 구현하는 데 사용될 수 있다. 쿼리는 페이로드(예: 함수 인수)를 전달할 수 있으며, 쿼리 가능 객체는 연산을 수행하고 결과를 응답으로 반환할 수 있다.30 이 추상화는 데이터 검색과 연산을 단일의 위치 투명한 패러다임 아래 통합한다.13

이 Query/Queryable 메커니즘은 Zenoh의 다양한 통신 패턴을 통합하는 "핵심 기능(killer feature)"이다. Zenoh가 "Pub/Sub/Query" 프로토콜로 묘사되는 데에는 이유가 있으며 1, "Query" 부분은 단순한 부가 기능이 아니다. 

`get`은 데이터 검색에 사용되지만 12, Queryable은 연산을 트리거할 수 있고 3 쿼리는 사용자 페이로드를 전달할 수 있다.30 Queryable 추상화는 강력한 일반화다. 이를 통해 Zenoh는 전통적인 RPC 프레임워크(예: gRPC)와 상태 저장 서비스의 기능을 데이터 중심 모델 내에 포함할 수 있다. 예를 들어, 애플리케이션은 목적지 페이로드를 포함하여 `/robot/42/compute_path`를 쿼리할 수 있고, 로봇의 Queryable은 경로 계획을 수행하고 응답한다. 이는 원격 측정을 위한 Pub/Sub 시스템과 명령을 위한 별도의 RPC 시스템을 함께 묶을 필요를 없애, 핵심적인 "통합" 약속을 이행한다.



본 섹션에서는 NTU(National Taiwan University) 연구에서 설명된 네 가지 프로토콜과 그 핵심 아키텍처 모델을 소개한다 32:

- **MQTT:** 중앙 집중식 브로커, 경량, IoT에서 널리 사용됨.32
- **Kafka:** 분산 스트리밍 플랫폼, 브로커 기반, 고처리량 데이터 파이프라인.32
- **DDS:** OMG 표준, 분산형, 실시간, UDP 멀티캐스트 기반.32
- **Zenoh:** 데이터 중심, 브로커, P2P, 라우팅 모드 지원.33


NTU 연구는 Zenoh의 성능 주장에 대한 가장 많이 인용되는 정량적 증거다. 아래 표는 아키텍트와 엔지니어가 정보에 입각한 데이터 기반 결정을 내리는 데 필수적인 정보를 제공한다. 이는 "빠르다"는 정성적 주장 대신 정량적 증거를 제시한다. 이 표는 여러 자료 28의 데이터를 단일의 권위 있는 시각으로 통합한다.

- **테스트 환경:** AMD Ryzen 7 5800X CPU, 32 GiB RAM, 100 Gb 이더넷, Ubuntu 20.04.32

- **처리량 결과:** Zenoh(P2P)는 100GbE에서 16KB 페이로드에 대해 50 Gbps 이상의 처리량을 일관되게 보여주며, DDS(~14 Gbps), Kafka(~5 Gbps), MQTT(~5 Gbps)를 크게 능가한다.10 성능 격차는 Zenoh의 낮은 오버헤드가 큰 이점을 제공하는 작은 페이로드에서 가장 두드러진다.32

- **지연 시간 결과:** 단일 머신(루프백) 테스트에서 `zenoh-pico`가 가장 낮은 지연 시간(5µs)을 보였고, Cyclone DDS(8µs)와 Zenoh P2P(10µs)가 그 뒤를 이었다.28 100GbE를 통한 다중 머신 테스트에서도 

  `zenoh-pico`(13µs)와 Zenoh P2P(16µs)가 선두를 차지하며 Cyclone DDS(38µs)를 크게 앞섰다.32

**표 1: NTU 성능 벤치마크 요약**

| 페이로드 크기 | 측정 항목                    | Zenoh (P2P)   | Zenoh (Brokered) | Cyclone DDS | Kafka | MQTT |
| ------------- | ---------------------------- | ------------- | ---------------- | ----------- | ----- | ---- |
| **8 B**       | 처리량 (msg/s, 단일 머신)    | ~4M           | ~4M              | ~2M         | ~63K  | ~38K |
| **16 KB**     | 처리량 (Gbps, 다중 머신)     | **~50**       | ~34              | ~14         | ~5    | ~9   |
| **32 KB**     | 처리량 (Gbps, 다중 머신)     | **>50**       | >34              | ~14         | ~5    | ~9   |
| **64 B**      | 지연 시간 (µs, 단일 머신)    | 10            | 21               | **8**       | 73    | 27   |
| **64 B**      | 지연 시간 (µs, 다중 머신)    | **16**        | 41               | 38          | 84    | 45   |
| **64 B**      | 지연 시간 (µs, `zenoh-pico`) | **5 (단일)**  | -                | -           | -     | -    |
| **64 B**      | 지연 시간 (µs, `zenoh-pico`) | **13 (다중)** | -                | -           | -     | -    |

참고: 위 표는 여러 소스 28에서 집계된 대표적인 값이다. 정확한 값은 테스트 조건에 따라 다를 수 있다. Zenoh-pico는 별도로 명시되었다.

Zenoh의 성능은 점진적인 개선이 아니라, 근본적으로 아키텍처에 의해 가능해진 다른 성능 등급을 나타낸다. Zenoh의 처리량은 가장 가까운 경쟁자인 DDS보다 2-3배, 많은 시나리오에서 MQTT/Kafka보다 10배 이상 높다.10 이는 동일한 아이디어의 더 나은 구현 때문이 아니라, Zenoh의 P2P 모드가 브로커 중심 프로토콜과 비교하여 데이터 경로를 근본적으로 변경하기 때문이다. 이는 주요 병목 현상을 피한다. 미들웨어 성능에 병목 현상이 있는 애플리케이션(예: 고주파 센서 융합, 대규모 데이터 집계)의 경우, Zenoh는 잠재적인 옵션일 뿐만 아니라 새로운 기능을 가능하게 하는 잠재적 요인이다. 성능 차이는 아키텍처적으로 실현 가능한 것으로 간주되는 것을 바꿀 만큼 크다.

또한, `zenoh-pico`의 지연 시간 수치는 복잡한 기능을 제거했을 때 프로토콜의 순수한 잠재력을 암시한다. `zenoh-pico`는 모든 테스트에서 지속적으로 가장 낮은 지연 시간을 기록하며, 전체 기능 Zenoh 및 DDS보다도 우수하다.28 그 이유는 `zenoh-pico`가 메시 라우팅을 지원하지 않기 때문이라고 설명된다.28 이는 전체 Zenoh 구현의 라우팅 로직이 작지만 측정 가능한 지연 시간 오버헤드를 추가함을 시사한다. 이는 아키텍트에게 명확한 트레이드오프를 제공한다. 복잡한 라우팅이 필요 없는 간단한 P2P 또는 멀티캐스트 시나리오에서 절대적으로 가장 낮은 지연 시간을 원한다면 `zenoh-pico`가 최적의 선택이다. 더 복잡하고 확장 가능한 네트워크의 경우, 전체 Zenoh 스택은 몇 마이크로초의 지연 시간 비용으로 라우팅 유연성을 제공한다. 이를 통해 사용 사례 요구 사항에 따라 미세 조정된 최적화가 가능하다.


- **P2P 대 브로커:** 벤치마크는 Zenoh의 P2P 모드가 자체 브로커 모드 및 MQTT, Kafka와 같은 다른 브로커 기반 프로토콜에 비해 우수한 성능(더 높은 처리량, 더 낮은 지연 시간)을 제공함을 명확히 보여준다. 이는 중개자를 제거하기 때문이다.33
- **무선 네트워크:** NTU 연구는 유선 이더넷에 중점을 두었지만, 다른 자료에 따르면 Zenoh의 성능 이점은 DDS의 멀티캐스트 탐색이 병목이 되는 무선 환경(Wi-Fi, 4G)에서 더욱 중요하다.19


Zenoh는 공유 메모리 및 제로-카피(zero-copy) 통신을 지원하며, 이는 동일한 호스트의 프로세스 간 대용량 데이터 교환에 매우 중요하다.10 이는 `iceoryx`를 사용하는 DDS 구현과 공유하는 기능이다.10 Zenoh의 구현은 네이티브이며 Rust로 작성되었다.28



Zenoh는 다양한 내장 보안 기능을 제공한다.

- **인증(Authentication):** Zenoh는 서버 및 클라이언트 인증을 위한 상호 TLS(mTLS)를 포함한 플러그형 인증 메커니즘을 지원한다.1

- **인가/접근 제어(Authorization/Access Control):** 접근 제어 목록(ACL)을 통해 세분화된 접근 제어가 가능하며, 특정 키 표현식에 대한 읽기/쓰기 접근을 제한할 수 있다.1

  `zenoh_security_tools` 패키지는 ROS 2 환경을 위해 이러한 구성을 생성하는 데 도움을 준다.39

- **기밀성(홉-바이-홉):** Zenoh는 TLS를 사용하여 전송 중인 데이터를 보호하는 보안 채널을 제공한다. 그러나 이 암호화는 홉-바이-홉(hop-by-hop) 방식으로 적용되므로, 데이터는 각 중간 라우터에서 해독되고 다시 암호화된다.1


보안은 자동차 및 로보틱스와 같은 대상 산업에서 타협할 수 없는 측면이다. 아래 표는 Zenoh가 제공하는 것과 더 중요하게는 제공하지 않는 것(그리고 그 실제적 함의)을 명확하고 간결하게 보여주어 기술 전략가에게 매우 중요하다. 이는 Zenoh를 고도의 보안 애플리케이션에 즉시 사용 가능하게 만드는 데 필요한 작업에 대한 현실적인 평가를 강제한다.

**표 2: Zenoh 보안 기능 분석**

| 보안 기능                | 메커니즘                     | Zenoh 제공 여부 | 강점                                                 | 한계 및 시사점                                               |
| ------------------------ | ---------------------------- | --------------- | ---------------------------------------------------- | ------------------------------------------------------------ |
| **인증**                 | mTLS, 플러그형 인증          | 예              | 허가되지 않은 노드의 네트워크 참여 방지.             | -                                                            |
| **접근 제어**            | 접근 제어 목록 (ACLs)        | 예              | 특정 키(토픽)에 대한 읽기/쓰기 권한의 세분화된 제어. | -                                                            |
| **홉-바이-홉 암호화**    | 각 링크에 대한 TLS           | 예              | 두 직접 연결 노드 간의 데이터 기밀성 보장.           | 손상된 중간 라우터로부터 데이터를 보호하지 못함.             |
| **종단 간 암호화(E2EE)** | N/A - 애플리케이션 계층 필요 | **아니요**      | -                                                    | 데이터가 신뢰할 수 없는 네트워크를 통과할 때 심각한 보안 위험. 키 관리에 대한 상당한 개발 오버헤드 발생. |

- **핵심 문제:** 네이티브 E2EE의 부재는 중요한 보안 격차로 식별된다.1 E2EE 시스템에서는 데이터가 원본 발행자에 의해 암호화되고 최종 구독자만이 해독할 수 있어 모든 중개자에게 불투명하다.41 Zenoh의 홉-바이-홉 모델은 이 원칙을 위반한다.
- **손상된 라우터 시나리오 (자동차 사례 연구):** 상세한 개념 증명(Proof of Concept)이 이 취약점을 보여준다.1 Zonal 자동차 아키텍처에서 공격자가 Zonal 라우터를 손상시키면, 라우터 간 링크에 TLS가 사용되더라도 이를 통과하는 모든 데이터를 평문으로 가로채 읽을 수 있다. 이는 TLS 세션이 손상된 라우터에서 종료되어 데이터가 다음 홉으로 재암호화되기 전에 노출되기 때문이다. 이는 신뢰할 수 없는 노드(예: 공용 인프라를 통과하는 V2X 데이터 또는 다른 OEM이 공급한 부품)를 통과하는 민감한 데이터에 심각한 위험을 초래한다.1

Zenoh의 고성능 라우팅/쿼리 기능과 강력한 E2EE 요구 사항 사이에는 근본적인 긴장 관계가 존재한다. Zenoh의 강력함은 라우터가 메시지 키를 검사하여 지능적인 라우팅, 필터링, 쿼리 집계를 수행할 수 있는 능력에서 비롯된다.3 진정한 E2EE는 라우터가 기능을 수행하는 데 필요한 키나 메타데이터를 포함한 전체 페이로드를 암호화할 것이다. 키 자체가 암호화되면 라우팅이 불가능해진다. 값만 암호화되면 라우터는 라우팅은 할 수 있지만 내용 기반 필터링이나 집계는 수행할 수 없다. 따라서 E2EE의 부재는 단순한 간과가 아니라 뿌리 깊은 아키텍처적 트레이드오프일 수 있다. Zenoh에 E2EE를 구현하는 것은 암호화 라이브러리를 추가하는 것뿐만 아니라 라우팅과 쿼리가 작동하는 방식에 대한 근본적인 재설계를 요구할 수 있다. 이는 일부 메타데이터는 인프라에 보이도록 유지하면서 핵심 페이로드는 불투명하게 만드는 복잡한 다층 보안 모델을 필요로 할 수 있다. 이 아키텍처적 과제는 즉각적인 로드맵에서 E2EE가 빠진 이유를 설명할 수 있다.


- **애플리케이션 수준 암호화:** 권장되는 주요 해결책은 클라이언트가 Zenoh에 게시하기 전에 애플리케이션 계층에서 암호화를 수행하는 것이다. 이는 진정한 E2EE를 달성하지만 복잡한 키 관리, 배포 및 수명 주기 제어의 부담을 애플리케이션 개발자에게 전가한다.1 Zenoh의 단순하고 우아한 API가 개발자를 생산적으로 만든다고 칭찬받지만 2, E2EE를 위한 권장 해결책은 "강력한 키 관리, 안전한 키 배포 및 수명 주기 관리"를 요구한다.1 이는 결코 사소한 엔지니어링 과제가 아니다. 보안이 중요한 애플리케이션을 위해 Zenoh를 채택하는 모든 팀에게, Zenoh의 간단한 API가 제공하는 "시장 출시 시간" 이점은 복잡한 맞춤형 공개 키 기반 구조(PKI) 또는 키 관리 서비스(KMS)를 구축해야 할 필요성에 의해 즉시 상쇄된다. 이는 총 소유 비용과 개발팀에 요구되는 보안 전문 지식을 크게 증가시킨다.
- **사용자 지정 보안 플러그인:** Zenoh의 플러그형 아키텍처는 E2EE를 구현할 수 있는 사용자 지정 보안 플러그인 개발을 허용한다. 이는 보다 통합된 솔루션이 될 수 있지만 상당한 개발 및 유지 관리 노력이 필요하다.1


공개 로드맵 저장소는 존재하지만 42, 제공된 자료는 공식 로드맵 뷰에서 E2EE에 대한 구체적이고 확정된 일정을 보여주지 않는다.42 GitHub 및 Discord의 커뮤니티 토론이 기능 요청의 주요 창구이며 29, 관찰 가능성 및 접근 제어와 같은 보안 기능에 대한 논의는 있지만 24, E2EE는 사용 가능한 자료에서 곧 출시될 기능으로 명시적으로 자세히 설명되지 않았다. 이는 단기적으로 최우선 순위가 아니거나, 연구에서 포착되지 않은 채널에서 논의되고 있음을 시사한다.



Zenoh는 잘 알려진 오픈 소스 조직인 이클립스 재단(Eclipse Foundation) 산하의 인큐베이팅 프로젝트로, 벤더 중립적인 거버넌스 구조를 제공한다.29 ZettaScale Technology는 안젤로 코르사로가 이끄는 Zenoh 프로젝트의 창시자이자 주요 상업적 동력이다.6 이 이중 주체 리더십은 오픈 소스의 신뢰성과 집중적인 상업적 개발의 강력한 조합을 제공한다. 이클립스 재단은 대규모의 위험 회피적인 기업과 경쟁하는 자동차 OEM들의 채택에 중요한 벤더 중립적인 기반을 제공한다. 이는 프로젝트의 장기적인 지속 가능성을 보장하고 단일 벤더의 독점 솔루션이 되는 것을 방지한다. 동시에 ZettaScale은 프로젝트를 공격적으로 추진하고 시장 요구에 대응하며(S3 백엔드 요청처럼 31) 상업적 지원을 제공하는 데 필요한 전담 엔지니어링 자원과 상업적 동력을 제공한다. 이러한 하이브리드 거버넌스 모델은 순수 커뮤니티 주도 프로젝트(정체될 수 있음)와 순수 기업 오픈 소스 프로젝트(이기적일 수 있음)와 관련된 위험을 모두 완화하는 주요 전략적 자산이다.


커뮤니티는 주로 질문을 위한 Discord 서버와 이슈 및 기능 제안을 위한 GitHub를 통해 참여한다.29 프로젝트는 GitHub에 공개 로드맵을 유지하며 42 공개적으로 개발되고, 다른 언어를 위한 API별 저장소를 가지고 있다.29


ZettaScale이 DDS(Cyclone DDS)와 Zenoh 모두에 대한 전문 지식을 보유하고 있다는 점은 로보틱스 및 자동차 시장에서 독특하고 강력한 위치를 부여한다. ZettaScale의 제품 포트폴리오에는 Cyclone DDS와 Zenoh가 모두 포함되어 있다.7 ROS 2 및 많은 자동차 시스템의 주요 미들웨어는 DDS이며, Zenoh가 이러한 생태계에 진입하는 가장 일반적인 경로는 

`zenoh-bridge-dds`를 통해서다.47 ZettaScale은 단순히 DDS의 경쟁자가 아니라 DDS의 대가다. 이를 통해 두 세계 사이에 고도로 최적화되고 견고한 브리지를 구축할 수 있다. 그들은 Zenoh를 "DDS 킬러"가 아닌 "DDS 확장기"로 마케팅할 수 있는 독보적인 위치에 있으며, DDS의 약점(WAN/무선 통신, 탐색 오버헤드)을 해결하면서 강점(풍부한 QoS, 차량 내 표준)과 공존한다. 두 프로토콜에 대한 깊은 전문 지식으로 가능해진 이러한 미묘한 포지셔닝은 순전히 대립적인 전략보다 훨씬 효과적인 채택 전략이다.

- **자동차:** Zenoh는 상당한 채택을 보였다. General Motors의 uProtocol에 채택되었으며 6, Bosch, Foxconn, FARobot, Volocopter 등이 채택 기업에 포함된다.37 TTTech Auto는 ZettaScale과 파트너십을 맺어 Zenoh, DDS, TSN을 결합한 "Zetta Auto" 통신 스택을 만들었다.49
- **로보틱스:** Zenoh는 특히 로봇-사물 간 통신(R2X)을 위한 ROS 2의 선도적인 프로토콜로 인정받고 있다.20 Indy Autonomous Challenge 2와 CARMA와 같은 이니셔티브에서 사용된다.2
- **기타 산업:** 제조, 운송, 의료, 산업 자동화 등도 채택 산업으로 등재되어 있다.37



Zenoh의 이중 ROS 2 전략(`bridge`와 `native RMW`)은 생태계 채택을 주도하는 탁월한 전략이다. Zenoh는 ROS 2와 통합하는 두 가지 뚜렷한 방법을 제공한다: 브리지(`zenoh-bridge-dds`)와 네이티브 미들웨어 구현(`rmw_zenoh`)이다.50 브리지는 기존 ROS 2 사용자에게 노력 없이 이전 버전과 호환되는 솔루션을 제공한다. 그들은 애플리케이션 코드를 한 줄도 변경하지 않고도 WAN/무선 통신 문제를 즉시 해결할 수 있다. 이것이 저마찰 진입점이다. 반면, 네이티브 RMW는 새로운 프로젝트나 Zenoh 생태계에 완전히 전념하려는 사람들에게 최고의 성능과 효율성을 제공한다. 이 이중 전략은 문제를 해결하려는 신중한 기존 사용자부터 그린필드 시스템을 구축하는 미래 지향적인 아키텍트까지 잠재 사용자 전체 스펙트럼을 훌륭하게 충족시킨다. 이를 통해 Zenoh는 브리지를 통해 신속하게 사용자 기반을 구축하는 동시에 우수한 네이티브 대안으로서의 장기적인 가치를 입증할 수 있다.

- **DDS의 로보틱스 과제:** 기본 ROS 2 미들웨어인 DDS는 다중 로봇 및 R2X 시나리오에서 흔한 무선 네트워크, WAN, 대규모 탐색에 어려움을 겪는다.13
- **`zenoh-bridge-dds`:** 이 플러그인은 투명한 브리지 역할을 하여 까다로운 네트워크를 통해 전송하기 위해 DDS 트래픽을 Zenoh로 변환하고 다른 쪽에서 다시 DDS로 변환한다. 이를 통해 기존 ROS 2 애플리케이션은 코드 변경 없이 Zenoh의 이점을 얻을 수 있다.47 이는 탐색 트래픽을 극적으로 줄인다.19
- **`rmw_zenoh` 구현:** 새롭거나 성능이 중요한 애플리케이션을 위해 `rmw_zenoh`는 ROS 2를 위한 네이티브 Zenoh 통신 계층을 제공하여 최대 성능과 효율성을 위해 DDS를 완전히 우회한다.51
- **사용 사례:** 로봇 스웜, 인터넷 규모의 원격 조작, 견고한 R2X 통신 활성화.13


로보틱스와 자동차 분야 모두에서 Zenoh의 주요 가치는 단일 기계 *내부*가 아니라 기계 *간* 및 클라우드 *사이*에 있다. Autoware 48 및 일반 로보틱스 19 사용 사례 모두에서 해결되는 문제는 무선, 인터넷 또는 다른 서브넷 간의 

*네트워크를 통한* 통신이다. DDS는 차량이나 로봇 *내부*의 유선 LAN에서는 잘 작동하는 것으로 인정된다. Zenoh의 기능(효율적인 탐색, 토폴로지 유연성, 낮은 오버헤드, WAN 라우팅)은 모두 네트워크가 제한적이거나, 신뢰할 수 없거나, 대규모인 곳에서 탁월하도록 설계되었다. Zenoh의 전략적 포지셔닝은 분산된 모바일 시스템을 위한 "접착제"로서이다. 이는 로봇, 자동차, 엣지, 클라우드를 연결하는 네트워크를 위한 프로토콜이다. 프로세스 간 또는 호스트 내 통신(특히 제로-카피 사용 시)에 사용될 수 있지만, 가장 강력하고 방어 가능한 가치 제안은 레거시 프로토콜이 설계되지 않은 시스템 간 통신 과제를 해결하는 데 있다.

- **V2X 및 차량 내 통신:** Zenoh의 낮은 지연 시간(~13µs)과 높은 처리량(~50 Gbps)은 V2X(V2V, V2I, V2C) 및 차량 내 네트워크용 CAN 버스, MQTT 또는 DDS의 대안으로 강력한 후보가 된다.1 ITU에 의해 ITS 표준화 통신 프로토콜로 권장되었다.1

- **사례 연구: 다중 Autoware 차량 관리:** 프로토타입은 클라우드 기반 FMS(Fleet Management System)에서 Autoware 기반 자율 주행 차량 군집을 관리하기 위해 Zenoh를 사용하는 방법을 보여준다.55 각 차량의 

  `zenoh-bridge-dds`는 중앙 서버와 통신하기 위한 게이트웨이로 사용되어 DDS의 인터넷 비호환성을 극복한다.48

- **Zetta Auto:** ZettaScale과 TTTech Auto의 상용 제품으로, AUTOSAR 및 ROS 2를 지원하는 완전한 안전 지향 자동차 통신 스택을 위해 Zenoh, DDS, TSN을 결합한다.49



Zenoh-Flow는 Zenoh 위에 구축된 데이터-흐름 프로그래밍 프레임워크다.56 이를 통해 애플리케이션을 노드(소스, 연산자, 싱크)의 방향성 그래프로 구조화할 수 있으며, 여기서 아크는 데이터 스트림을 나타낸다.57 이는 로보틱스 및 자율 주행에서 처리 파이프라인(예: 센서 -> 인식 -> 계획 -> 작동)을 만드는 일반적인 패러다임이다.57

Zenoh-Flow는 Zenoh의 핵심 "통합" 철학의 애플리케이션 수준 표현이다. Zenoh가 통합 통신, 스토리지 및 연산을 위한 저수준 프로토콜을 제공하는 반면 1, Zenoh-Flow는 연산 노드의 그래프로 애플리케이션을 구축하기 위한 고수준 프레임워크를 제공한다.56 Zenoh가 "어떻게"라면 Zenoh-Flow는 "무엇"이다. Zenoh-Flow는 Zenoh의 위치 투명성과 통신 기본 요소를 활용하여 분산 애플리케이션을 오케스트레이션한다. 선언적 그래프는 데이터 흐름(이동), 상태 저장 연산자(정지/연산), 싱크/소스(I/O)의 통합된 표현이다. 이는 Zenoh를 단순한 프로토콜에서 완전한 개발 및 배포 플랫폼의 기반으로 끌어올리는 전략적인 다음 단계다. 이는 개발자가 분산 및 통신의 저수준 세부 정보를 직접 관리할 필요 없이 복잡하고 탄력적인 애플리케이션을 생성할 수 있도록 하여 개발자 생산성을 높이는 데 중요한 단계다. 이는 "이것으로 실제로 무엇을 만들 수 있는가?"라는 질문에 대한 답이다.


- **선언적 접근 방식:** 애플리케이션 그래프는 설명자 파일(예: YAML)에 선언적으로 정의되며, 노드, 그 구현(공유 라이브러리), 그리고 그들 사이의 링크를 지정한다.56 이는 데이터 흐름을 명시적이고 수정하기 쉽게 만든다.57
- **위치 투명성 및 자동 배포:** Zenoh-Flow는 분산 인프라 전반에 걸쳐 노드의 배포 및 상호 연결을 위치 투명한 방식으로 처리한다.57
- **성능 최적화:** 상황에 맞는 최적화를 수행한다. 예를 들어, 두 노드가 동일한 호스트에 배포되면 전체 네트워크 Pub/Sub 쌍 대신 제로-카피 인-메모리 채널을 사용하여 데이터를 교환한다.57
- **실시간 제어:** 작업에 우선순위 할당, CPU 코어 피닝, 사용자 지정 스케줄링 등 실시간 기능을 염두에 두고 개발되고 있어 자동차 및 안전이 중요한 애플리케이션에 적합하다.58


Zenoh-Flow는 성능과 메모리 안전성을 위해 Rust로 개발되었다.58 주요 사용 사례는 Pylot 자율 주행 플랫폼을 이식한 것으로, 상당한 성능 향상을 가져왔다.57 이는 소프트웨어 정의 차량 아키텍처를 위한 핵심 프레임워크로 자리매김하고 있다.58



Zenoh는 광범위한 개발자 접근성에 대한 약속을 보여주며 다양한 인기 언어에 대한 API를 제공한다.

- **Rust:** 네이티브 구현 언어로서 Rust API는 가장 기능이 풍부하며 핵심 개발에 사용된다.12 예제는 비동기 작업을 위한 `tokio` 사용을 보여준다.12
- **Python:** 성숙한 API가 `pip`를 통해 제공되며, 특히 스크립팅 및 ROS 2 생태계와의 통합을 위해 예제에서 널리 사용된다.17
- **C/C++:** C 및 C++ 바인딩이 제공되며, 임베디드 시스템 및 자동차, 로보틱스의 레거시 코드베이스와의 통합에 매우 중요하다.30
- **신흥 API:** JavaScript/TypeScript(WASM을 통한 브라우저용), Java/Kotlin(Android 및 엔터프라이즈용)이 생태계를 넓히기 위해 활발히 개발 중이다.24


API 설계 전략은 아키텍처 전략을 반영한다: 깊이 있는 선택적 복잡성을 갖춘 간단한 진입점을 제공하는 것이다. 간단한 `zenoh.open().put(...)`은 매우 쉽지만 12, 구성 및 세션 옵션은 엄청난 깊이를 드러낸다(예: `CongestionControl`, `ConsolidationMode`, `Priority`).16 이 디자인 패턴은 "점진적 공개(progressive disclosure)"로 알려져 있다. 이는 초보자나 간단한 사용 사례에 대한 프레임워크를 접근하기 쉽게 만들면서도, 전문가가 고급 최적화 및 제어를 위해 필요한 강력한 노브와 레버를 여전히 제공한다. 이는 진입 장벽을 낮추어 개발자가 몇 분 안에 "hello world"를 실행할 수 있게 한다. 요구 사항이 더 복잡해짐에 따라, 처음부터 압도당하지 않고 점진적으로 더 고급 기능을 배우고 적용할 수 있다. 이는 초기 채택과 장기적인 숙달을 모두 가속화한다.

- API는 간단하고 빠르게 채택할 수 있도록 설계되었다.3 핵심 작업에는 `open`, `close`, `put`, `get`, `declare_subscriber`가 포함된다.12
- 구성은 유연하여 환경 변수, 파일(JSON5) 또는 프로그래밍 방식으로 설정할 수 있다.16
- API는 신뢰성 설정, 혼잡 제어, 쿼리 통합 모드와 같은 고급 기능을 노출하여 세분화된 제어를 제공한다.16



- **강점:** 타의 추종을 불허하는 성능과 효율성, 아키텍처 유연성(MCU에서 클라우드까지의 연속체), 통합 데이터 패러다임(이동, 정지, 연산), 그리고 실용적인 생태계 통합 전략(브리지).
- **약점:** 네이티브 E2EE 부재로 인한 심각한 보안 격차. 이는 보안 환경의 개발자에게 상당한 부담을 준다. DDS 및 MQTT와 같은 기존 표준에 비해 프로젝트의 상대적인 미성숙함.
- **고유 가치 제안:** Zenoh는 엣지에서 클라우드에 이르는 이기종 분산 시스템을 위한 단일의 위치 투명한 데이터 패브릭을 제공하도록 처음부터 설계된 유일한 프로토콜이며, 여러 전문 프로토콜을 이어 붙이는 데 내재된 아키텍처적 마찰을 해결한다.


- **성능이 중요한 시스템의 경우:** Zenoh는 특히 기존 미들웨어가 병목 현상을 일으키는 경우 주요 평가 후보가 되어야 한다.
- **보안이 중요한 시스템의 경우:** 애플리케이션 수준의 E2EE를 구현하고 관리하기 위한 명확하고 자원이 충분한 계획이 있을 때만 Zenoh를 채택해야 한다. 기본 제공 보안은 신뢰할 수 없는 네트워크를 통과하는 데이터에 대해 불충분하다.
- **ROS 2/DDS 사용자의 경우:** `zenoh-bridge-dds`를 무선 및 WAN 통신 문제를 즉시 해결하기 위한 저위험, 고수익 솔루션으로 평가해야 한다.


1.0 릴리스로의 경로는 프로젝트의 성숙도와 장기적인 지원을 시사한다.24 Zenoh-Flow의 개발은 프로토콜에서 완전한 애플리케이션 프레임워크로 소프트웨어 스택을 상향 이동하는 전략적인 움직임을 나타낸다. 커뮤니티 주도 기능 요청(예: S3 백엔드, 개선된 접근 제어, CAN 지원)은 건강하고 반응이 빠른 개발 프로세스를 보여준다.24 미래의 핵심 질문은 E2EE 격차 해결의 우선순위와 일정에 관한 것으로 남아 있다.


1. Zenoh Protocol Security Analysis - Census Labs, accessed July 9, 2025, https://census-labs.com/news/2025/03/17/zenoh-protocol-security-analysis/
2. What is Zenoh? / Zenoh - pub/sub, geo distributed storage, query, accessed July 9, 2025, https://zenoh.io/docs/overview/what-is-zenoh/
3. Zenoh - The Zero Overhead, Pub/Sub, Store, Query, and Compute Protocol., accessed July 9, 2025, https://zenoh.io/
4. Eclipse zenoh: The Edge Data Fabric, accessed July 9, 2025, https://newsroom.eclipse.org/eclipse-newsletter/2021/july/eclipse-zenoh-edge-data-fabric
5. zenoh: The Edge Data Fabric | PPT - SlideShare, accessed July 9, 2025, https://www.slideshare.net/slideshow/zenoh-the-edge-data-fabric/248636044
6. ZettaScale's Journey and Impact in Open-Source Auto Software Development - Bisinfotech, accessed July 9, 2025, https://www.bisinfotech.com/zettascale-journey-and-impact-in-open-source-auto-software-development/
7. ZettaScale Technology, accessed July 9, 2025, https://www.zettascale.tech/
8. ADLINK Tech | Eclipse Zenoh Zero-overhead middleware, accessed July 9, 2025, https://www.adlinktech.com/en/zenoh
9. How does the Zenoh protocol enhance edge device operation? - Microcontroller Tips, accessed July 9, 2025, https://www.microcontrollertips.com/how-does-the-zenoh-protocol-enhance-edge-device-operation/
10. Zenoh Performances : r/rust - Reddit, accessed July 9, 2025, https://www.reddit.com/r/rust/comments/123ioy1/zenoh_performances/
11. Eclipse Zenoh 0.10.0 is out : r/rust - Reddit, accessed July 9, 2025, https://www.reddit.com/r/rust/comments/16yneun/eclipse_zenoh_0100_is_out/
12. zenoh - Rust - Docs.rs, accessed July 9, 2025, https://docs.rs/zenoh/latest/zenoh/
13. Edge Robotics with Eclipse zenoh and ROS 2, accessed July 9, 2025, https://www.eclipse.org/community/eclipse_newsletter/2020/september/1.php
14. Zenoh in action / Zenoh - pub/sub, geo distributed storage, query, accessed July 9, 2025, https://zenoh.io/docs/overview/zenoh-in-action/
15. Deployment / Zenoh - pub/sub, geo distributed storage, query, accessed July 9, 2025, https://zenoh.io/docs/getting-started/deployment/
16. API Reference - zenoh-c 0.5.0 documentation, accessed July 9, 2025, https://zenoh-c.readthedocs.io/en/0.5.0-b5/api.html
17. zenoh - PyPI, accessed July 9, 2025, https://pypi.org/project/zenoh/
18. eclipse-zenoh/zenoh: zenoh unifies data in motion, data in-use, data at rest and computations. It carefully blends traditional pub/sub with geo-distributed storages, queries and computations, while retaining a level of time and space efficiency that is well beyond any of the mainstream stacks. - GitHub, accessed July 9, 2025, https://github.com/eclipse-zenoh/zenoh
19. Zenoh- Mobile Robotics through Communication and Scalability - Bisinfotech, accessed July 9, 2025, https://www.bisinfotech.com/enhancing-mobile-robotics-through-efficient-communication-and-scalability/
20. Transforming the robotics industry: A sustainable communication protocol, accessed July 9, 2025, https://www.controleng.com/transforming-the-robotics-industry-a-sustainable-communication-protocol/
21. Indy Autonomous Challenge (IAC): Experiences from the Trenches - Zenoh, accessed July 9, 2025, https://zenoh.io/blog/2021-09-28-iac-experiences-from-the-trenches/
22. Zenoh Explained - YouTube, accessed July 9, 2025, https://www.youtube.com/watch?v=qeEQJDoVrg8
23. A Performance Study on the Throughput and Latency of Zenoh, MQTT, Kafka, and DDS - arXiv, accessed July 9, 2025, https://arxiv.org/pdf/2303.09419
24. Zenoh and What's Next in 2024 - ZettaScale Technology, accessed July 9, 2025, https://www.zettascale.tech/news/zenoh-and-whats-next-in-2024/
25. Zenoh - A Protocol That Should be on Your Radar | by Jkel - Medium, accessed July 9, 2025, https://medium.com/@kelj/zenoh-a-protocol-that-should-be-on-your-radar-72befa697411
26. Eclipse Zenoh: Supporting ICOS's Data Distribution from the Cloud-to-Thing continuum, accessed July 9, 2025, https://www.icos-project.eu/blog/eclipse-zenoh/
27. Zenoh Reliability, Scalability and Congestion Control, accessed July 9, 2025, https://zenoh.io/blog/2021-06-14-zenoh-reliability/
28. Zenoh Performance - ROS General - Open Robotics Discourse, accessed July 9, 2025, https://discourse.ros.org/t/zenoh-performance/30494
29. The Zenoh Community / Zenoh - pub/sub, geo distributed storage, query, accessed July 9, 2025, https://zenoh.io/community/
30. Zenoh Charmander is coming to town / Zenoh - pub/sub, geo distributed storage, query, accessed July 9, 2025, https://zenoh.io/blog/2023-01-10-zenoh-charmander/
31. Zenoh Storage Gets a Boost: Empowering Storage with S3 Integration, accessed July 9, 2025, https://zenoh.io/blog/2023-07-17-s3-backend/
32. A Performance Study on the Throughput and Latency of Zenoh, MQTT, Kafka, and DDS, accessed July 9, 2025, https://www.alphaxiv.org/overview/2303.09419v1
33. Comparing the Performance of Zenoh, MQTT, Kafka, and DDS ..., accessed July 9, 2025, https://zenoh.io/blog/2023-03-21-zenoh-vs-mqtt-kafka-dds/
34. Comparison of Middlewares in Edge-to-Edge and Edge-to-Cloud Communication for Distributed ROS2 Systems | Papers With Code, accessed July 9, 2025, https://paperswithcode.com/paper/comparison-of-dds-mqtt-and-zenoh-in-edge-to
35. [2309.07496] Comparison of Middlewares in Edge-to-Edge and Edge-to-Cloud Communication for Distributed ROS2 Systems - arXiv, accessed July 9, 2025, https://arxiv.org/abs/2309.07496
36. Comparison of DDS, MQTT, and Zenoh in Edge-to-Edge/Cloud Communication with ROS 2, accessed July 9, 2025, https://deepai.org/publication/comparison-of-dds-mqtt-and-zenoh-in-edge-to-edge-cloud-communication-with-ros-2
37. Eclipse Foundation releases new communications protocol for robotics, accessed July 9, 2025, https://roboticsandautomationnews.com/2024/11/01/eclipse-foundation-releases-new-communications-protocol-for-robotics/86691/
38. Eclipse Zenoh 1.0.0 Debuts, Redefining Connectivity for Robotics and Automotive, accessed July 9, 2025, https://newsroom.eclipse.org/news/announcements/eclipse-zenoh-100-debuts-redefining-connectivity-robotics-and-automotive
39. zenoh_security_tools: Rolling 0.8.0 documentation - ROS 2, accessed July 9, 2025, https://docs.ros.org/en/rolling/p/zenoh_security_tools/
40. Zenoh Protocol Security Analysis - Census Labs, accessed July 9, 2025, https://census-labs.com/news/tag/zenoh/
41. What is end-to-end encryption and how to do it (correctly / securely)? - Information Security Stack Exchange, accessed July 9, 2025, https://security.stackexchange.com/questions/230068/what-is-end-to-end-encryption-and-how-to-do-it-correctly-securely
42. eclipse-zenoh/roadmap - GitHub, accessed July 9, 2025, https://github.com/eclipse-zenoh/roadmap
43. Roadmap / eclipse-zenoh/zenoh Wiki - GitHub, accessed July 9, 2025, https://github.com/eclipse-zenoh/zenoh/wiki/Roadmap
44. Issues / eclipse-zenoh/zenoh / GitHub, accessed July 9, 2025, https://github.com/eclipse-zenoh/zenoh/issues
45. Observability / eclipse-zenoh roadmap / Discussion #92 - GitHub, accessed July 9, 2025, https://github.com/eclipse-zenoh/roadmap/discussions/92
46. Eclipse Zenoh | projects.eclipse.org, accessed July 9, 2025, https://projects.eclipse.org/projects/iot.zenoh/governance
47. Integrating ROS2 with Eclipse zenoh / Zenoh - pub/sub, geo ..., accessed July 9, 2025, https://zenoh.io/blog/2021-04-28-ros2-integration/
48. Driving Autoware with Zenoh, accessed July 9, 2025, https://autoware.org/driving-autoware-with-zenoh/
49. Zetta Auto | TTTech Auto, accessed July 9, 2025, https://www.tttech-auto.com/software-products/zetta-auto
50. Awesome Robot Operating System 2 (ROS 2) | Awesome ROS2 - GitHub Pages, accessed July 9, 2025, https://fkromer.github.io/awesome-ros2/
51. Zenoh - ROS 2 Documentation: Rolling documentation, accessed July 9, 2025, https://docs.ros.org/en/rolling/Installation/RMW-Implementations/Non-DDS-Implementations/Working-with-Zenoh.html
52. The Future of Robotics with Zenoh Integration - ZettaScale Technology, accessed July 9, 2025, https://www.zettascale.tech/news/the-future-of-robotics-with-zenoh-integration/
53. Zenoh: The Future of Communication for Edge Computing and IoT - YouTube, accessed July 9, 2025, https://www.youtube.com/watch?v=ahO4kT_Zg7s
54. Overcome DDS Limitations with Zenoh's DDS Plugin and DDS Bridge for Robotic Systems - YouTube, accessed July 9, 2025, https://www.youtube.com/watch?v=9h01_MSKPS0
55. Managing Multiple Autoware Vehicles with Zenoh - Autoware, accessed July 9, 2025, https://autoware.org/managing-multiple-autoware-vehicles-with-zenoh/
56. Zenoh-Flow 0.6.0-rc: Getting Started, accessed July 9, 2025, https://zenoh.io/blog/2024-01-31-zenoh-flow-getting-started/
57. Data Flow programming with Zenoh-Flow, accessed July 9, 2025, https://zenoh.io/blog/2023-02-10-zenoh-flow/
58. Optimising Eclipse Zenoh-Flow: A Comprehensive Guide - ZettaScale Technology, accessed July 9, 2025, https://www.zettascale.tech/news/optimising-eclipse-zenoh-flow-a-comprehensive-guide/
59. Zenoh-Flow drift improving an Autonomous Driving platform - YouTube, accessed July 9, 2025, https://www.youtube.com/watch?v=QajVWYshaHk
60. Eclipse Zenoh-Flow | projects.eclipse.org, accessed July 9, 2025, https://projects.eclipse.org/proposals/eclipse-zenoh-flow
61. Zenoh Tutorial – Part I - YouTube, accessed July 9, 2025, https://www.youtube.com/watch?v=xNJlDIkV2uo
62. Zenoh API Reference - zenoh-python 1.4.0 documentation, accessed July 9, 2025, https://zenoh-python.readthedocs.io/
63. zenoh-python/examples/z_sub.py at main - GitHub, accessed July 9, 2025, https://github.com/eclipse-zenoh/zenoh-python/blob/main/examples/z_sub.py
64. zenoh-python 1.4.0 documentation, accessed July 9, 2025, https://zenoh-python.readthedocs.io/en/latest/
65. Zenoh Gozuryū is Here!, accessed July 9, 2025, https://zenoh.io/blog/2025-04-14-zenoh-gozuryu/

