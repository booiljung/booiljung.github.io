[Zenoh](./index.md)
# 군수용, 로봇, 드론 및 항공 소프트웨어 시스템을 위한 Zenoh 프로토콜 적용


본 보고서는 차세대 데이터 통신 프로토콜인 Zenoh의 기술적 특성을 심층 분석하고, 이를 군용, 로봇, 드론 및 항공 소프트웨어 시스템에 적용할 경우의 타당성을 종합적으로 평가한다. Zenoh는 데이터 이동(in motion), 저장(at rest), 연산(computations)을 통합하는 혁신적인 패러다임을 제시하며, 특히 무선 및 불안정한 네트워크 환경에서 기존 미들웨어(DDS, MQTT 등)를 능가하는 압도적인 성능을 입증하였다.1 이러한 특성은 Zenoh를 현대 로보틱스 및 특정 군용 시스템을 위한 강력한 후보로 부상시킨다.

그러나 Zenoh의 현재 상태는 고도의 신뢰성과 보안이 요구되는 분야에 즉시 적용하기에는 명백한 한계를 드러낸다. 특히, 네이티브 종단간 암호화(End-to-End Encryption, E2EE)의 부재는 데이터가 중간 라우터를 통과할 때 평문으로 노출될 수 있는 심각한 보안 취약점으로 작용하며, 이는 군사 및 기밀 데이터를 다루는 시스템에 치명적이다.3 또한, 항공 소프트웨어에 필수적인 DO-178C 안전성 인증을 위한 공식적인 절차나 산출물이 전무하여, 상용 및 군용 항공기 적용은 현재로서는 불가능하다.

따라서 본 보고서는 각 도메인별로 차별화된 적용 전략을 권고한다. **로보틱스 및 드론 분야**에서는 입증된 성능 우위를 바탕으로 단기적인 평가 및 적극적인 도입을 강력히 추천한다. **군용 시스템 분야**에서는 DIL(Disconnected, Intermittent, Limited) 환경에 대한 탁월한 아키텍처 적합성을 인정하되, 민감 정보를 다루는 시스템에 적용하기 위해서는 E2EE 보안 계층의 성공적인 개발 및 통합을 전제 조건으로 하는 조건부 도입을 제안한다. 마지막으로 **항공 시스템 분야**에서는 인증까지의 막대한 장벽을 고려하여, 단기적인 솔루션이 아닌 장기적인 연구개발(R&D) 대상으로 접근할 것을 권고한다. Zenoh는 잠재적으로 혁신적인 기술이지만, 그 도입은 각 분야의 엄격한 요구사항에 대한 명확한 이해와 전략적인 위험 관리를 동반해야 한다.



Zenoh(Zero Overhead Network Protocol)는 현대 분산 시스템이 직면한 근본적인 문제, 즉 데이터 통신의 파편화를 해결하기 위해 설계된 프로토콜이다.4 기존 시스템 아키텍처는 종종 리소스가 제한된 마이크로컨트롤러(MCU), 엣지 서버, 클라우드 데이터 센터 등 각기 다른 환경에 최적화된 별개의 프로토콜을 사용하여 "연결성의 섬(connectivity islands)"을 만들어냈다.3 예를 들어, 엣지 장치 간에는 DDS를, 엣지-클라우드 간에는 MQTT를 사용하는 식이다. 이러한 이기종 프로토콜의 혼용은 복잡한 브릿지(bridge)와 게이트웨이(gateway)를 필요로 하며, 이는 시스템의 복잡성, 취약성, 그리고 관리 비용을 증가시키는 주요 원인이 된다.

Zenoh는 이러한 문제를 "데이터 해방(data liberation)"이라는 개념을 통해 해결하고자 한다.6 이는 데이터 이동(data in motion), 데이터 저장(data at rest), 그리고 연산(computations)이라는 세 가지 핵심 요소를 단일 프로토콜 내에서 통합적으로 추상화함으로써 달성된다.3 Zenoh는 게시/구독(Pub/Sub), 쿼리(Query), 분산 저장(Storage) 기능을 하나의 데이터 중심(data-centric) 패브릭으로 묶어, 개발자가 MCU에서 데이터 센터에 이르는 전체 컴퓨팅 연속체(continuum)에 걸쳐 데이터를 원활하게 흐르게 할 수 있도록 지원한다.6 이 통합된 접근 방식은 복잡한 이기종 시스템 아키텍처를 근본적으로 단순화하고, 서로 다른 시스템 세그먼트 간의 데이터 통합 오버헤드와 아키텍처의 취약성을 크게 감소시키는 핵심적인 가치를 제공한다.


Zenoh의 가장 큰 아키텍처적 장점 중 하나는 통신 토폴로지에 대한 탁월한 유연성이다. 이 프로토콜은 특정 통신 모델에 얽매이지 않고, 피어-투-피어(Peer-to-Peer, P2P), 라우팅(Routed), 브로커(Brokered) 모델을 모두 지원하며, 필요에 따라 이들을 혼합하여 사용할 수 있다.9

- **피어-투-피어(P2P):** DDS와 유사하게, 네트워크상의 노드들이 중앙 중개자 없이 직접 통신하는 모델이다. 이는 로컬 네트워크 내에서 최소한의 지연 시간과 최대의 처리량을 달성하는 데 이상적이다.
- **라우팅(Routed):** Zenoh 라우터들이 메시지를 지능적으로 전달하여 네트워크를 확장하는 모델이다. 이를 통해 인터넷과 같은 광역 네트워크(WAN)를 통해 지리적으로 분산된 시스템을 구축할 수 있으며, NAT 순회(NAT traversal)와 같은 기능을 제공한다.1
- **브로커(Brokered):** MQTT와 유사하게, 클라이언트가 중앙 라우터(브로커 역할)에 연결하여 통신하는 모델이다. 이는 리소스가 극도로 제한된 장치나 클라이언트-서버 통신 패턴에 유용하다.

이러한 유연성은 Zenoh가 전송 계층에 구애받지 않는(transport-agnostic) 특성에 의해 더욱 강화된다. Zenoh는 TCP/IP, UDP, QUIC과 같은 표준 네트워크 프로토콜뿐만 아니라, 직렬 통신(Serial Links), 블루투스(Bluetooth), 심지어 CANbus와 같은 비-IP 기반의 데이터 링크 계층에서도 동작할 수 있다.1

이러한 아키텍처적 유연성은 DDS나 MQTT와 같은 기존 미들웨어와 Zenoh를 근본적으로 차별화하는 지점이다. DDS는 주로 P2P 통신에 강점을 보이지만 WAN 환경으로의 확장에 어려움을 겪는 반면, MQTT는 브로커 모델에 엄격하게 의존하여 확장성은 좋지만 브로커가 성능 병목 및 단일 장애점(Single Point of Failure, SPOF)이 될 수 있다. Zenoh는 애플리케이션의 변경 없이 시스템의 요구사항에 따라 로컬에서는 효율적인 P2P 통신을 사용하고, 네트워크 경계를 넘을 때는 라우터를 통해 원활하게 통신을 확장하는 하이브리드 접근 방식을 취할 수 있다. 이는 복잡한 분산 시스템을 구축하는 데 있어 매우 강력한 장점으로 작용한다.


Zenoh의 성능은 여러 독립적인 벤치마크 연구를 통해 입증되었으며, 특히 처리량과 지연 시간 측면에서 기존 프로토콜들을 압도하는 결과를 보여준다. 대만 국립대학교(NTU)에서 수행된 포괄적인 성능 비교 연구와 ROS 2 환경에 초점을 맞춘 다수의 연구는 Zenoh의 성능 우위를 정량적으로 뒷받침한다.1

주요 성능 지표는 다음과 같다:

- **처리량(Throughput):** Zenoh는 P2P 구성에서 최대 50-67 Gbps의 처리량을 기록했으며, 이는 동일 조건에서 측정한 CycloneDDS(최대 26 Gbps), Kafka(최대 5 Gbps), MQTT(최대 9 Gbps)를 크게 상회하는 수치다.1 특히 대용량 페이로드(16KB 이상) 전송 시 Zenoh의 성능 우위는 더욱 두드러진다. 작은 메시지를 초당 수백만 개 전송하는 능력(최대 400만 msg/s) 또한 실시간 제어 및 이벤트 처리에서 중요한 장점이다.1
- **지연 시간(Latency):** 최적화된 P2P 환경에서 Zenoh는 13 µs 미만의 지연 시간을 달성할 수 있으며 3, 마이크로컨트롤러를 위한 경량 구현인 Zenoh-Pico는 5 µs라는 극도로 낮은 지연 시간을 기록하기도 했다.1 이는 하드웨어 수준의 실시간 제어가 요구되는 애플리케이션에 Zenoh가 적합함을 시사한다.

이러한 성능 지표는 단순히 숫자의 우위를 넘어, 실제 시스템의 운영 능력과 직결된다. 높은 처리량은 라이다(LiDAR) 포인트 클라우드나 고해상도 비디오 스트림과 같은 풍부한 센서 데이터를 손실 없이 전송할 수 있음을 의미하며, 낮은 지연 시간은 원격 제어나 로봇의 동적 제어 루프를 더욱 정밀하고 안정적으로 만들 수 있음을 의미한다.

다음 표는 주요 미들웨어 간의 성능을 요약하여 비교한 것이다.

**표 1: 주요 미들웨어 성능 비교 분석: Zenoh vs. DDS vs. MQTT**

| 메트릭                        | Zenoh (P2P/라우팅) | CycloneDDS | MQTT   | Kafka  | 네트워크 조건           | 데이터 소스 |
| ----------------------------- | ------------------ | ---------- | ------ | ------ | ----------------------- | ----------- |
| **최대 처리량 (Gbps)**        | **~50-67 / ~34**   | ~14-26     | ~9     | ~5     | 유선 이더넷 (100Gbps)   | 1           |
| **최대 메시지 속도 (Mmsg/s)** | **~4.0**           | ~2.0       | ~0.04  | ~0.06  | 유선 이더넷 (단일 머신) | 1           |
| **지연 시간 (µs)**            | **~10-16**         | **~8-38**  | ~27-45 | ~73-84 | 유선 이더넷             | 1           |
| **지연 시간 (ms, Array1k)**   | **~12-15**         | ~51        | -      | -      | Wi-Fi                   | 2           |
| **지연 시간 (ms, Array1k)**   | **~72-134**        | ~59        | -      | -      | 4G                      | 2           |

*참고: Zenoh-Pico는 5µs의 더 낮은 지연 시간을 기록했으나, 일반적인 Zenoh 구현체와 비교를 위해 해당 값을 기준으로 제시함. MQTT/Kafka는 ROS 2 테스트에서 직접 비교되지 않아 일부 데이터가 누락됨.*

이 표는 Zenoh의 전반적인 성능 우위를 명확하게 보여준다. 특히 주목할 점은 Wi-Fi 및 4G와 같은 무선 환경에서의 지연 시간이다. 유선 환경에서는 DDS가 매우 낮은 지연 시간을 보이지만, 무선 환경에서는 DDS의 UDP 멀티캐스트 기반 탐색 메커니즘이 "네트워크 플러딩(flooding effect)"을 유발하여 성능이 급격히 저하되는 반면, Zenoh는 안정적인 성능을 유지한다.2 이 특성은 모바일 로봇이나 드론과 같이 무선 통신이 필수적인 애플리케이션에 Zenoh가 왜 더 적합한지를 설명하는 핵심적인 근거가 된다.


Zenoh는 단순히 기존 프로토콜의 성능을 개선한 것이 아니라, 그들의 근본적인 한계를 극복하기 위한 새로운 아키텍처적 접근 방식을 제시한다. 이는 Zenoh를 단순한 '대안'이 아닌, DDS와 MQTT의 장점을 결합한 '하이브리드' 패러다임으로 이해해야 하는 이유다.

- **DDS와의 비교:** DDS는 22개의 복잡한 QoS(Quality of Service) 정책을 통해 강력한 데이터 제어 기능을 제공하지만, 이는 동시에 높은 학습 곡선과 복잡성을 야기한다.14 Zenoh는 훨씬 단순한 API와 개념 모델을 통해 유사한 수준의 신뢰성과 데이터 제어 기능을 제공한다. 더 중요한 차이점은 확장성이다. 표준 DDS의 탐색 메커니즘(Discovery)은 UDP 멀티캐스트에 크게 의존하는데, 이는 라우터를 거치는 WAN 환경이나 손실이 많은 무선 네트워크에서는 비효율적이거나 제대로 동작하지 않는 경우가 많다.1 이로 인해 원격 로봇 제어나 클라우드 기반 관제 시스템 구축에 한계가 발생한다. 반면, Zenoh의 라우팅 아키텍처는 처음부터 인터넷 규모의 분산 환경을 염두에 두고 설계되어 이러한 확장성 문제를 근본적으로 해결한다.1
- **MQTT와의 비교:** MQTT는 브로커를 통한 중앙 집중식 아키텍처 덕분에 WAN 환경에서 뛰어난 연결성을 제공한다. 하지만 이 브로커는 시스템 전체의 성능을 좌우하는 병목 지점이자, 브로커 장애 시 전체 시스템이 마비될 수 있는 단일 장애점(SPOF)이 된다.1 Zenoh는 분산 아키텍처를 통해 이러한 문제를 회피한다. P2P 통신과 라우팅 모델을 통해 중앙 의존성을 제거하고, 시스템의 강건성(robustness)과 확장성을 높인다. 또한 Zenoh는 자체적으로 MQTT 브로커 역할을 수행할 수 있는 플러그인을 제공하여, 기존 MQTT 기반 시스템을 Zenoh 생태계로 원활하게 통합하거나 확장할 수 있는 경로를 제공한다.12

이러한 차이점을 종합해 보면, Zenoh의 핵심적인 혁신은 DDS의 강점인 '로컬 네트워크에서의 저지연/고성능'과 MQTT의 강점인 '광역 네트워크에서의 안정적인 연결성'을 하나의 프로토콜 안에 성공적으로 융합했다는 점에 있다. 기존에는 로봇 내부 통신에는 DDS를, 로봇-클라우드 간 통신에는 MQTT를 사용하는 등 복잡한 다중 프로토콜 아키텍처가 필요했다면, Zenoh는 이를 하나의 통일된 데이터 패브릭으로 대체할 수 있는 잠재력을 가지고 있다. 이는 자율주행차나 전 세계에 분산된 로봇 플릿과 같은 복잡한 시스템의 아키텍처를 획기적으로 단순화하고 효율화할 수 있는 중요한 전략적 이점이다.

------



현대 로보틱스 시스템, 특히 자율 이동 로봇(AMR)과 드론은 점점 더 복잡해지고 데이터 집약적으로 변모하고 있다. 이러한 시스템을 안정적으로 구동하기 위한 통신 미들웨어는 다음과 같은 핵심 요구사항을 만족해야 한다.

- **실시간 성능 및 결정성(Determinism):** 모터 제어, 센서 융합, 충돌 회피와 같은 작업은 수 밀리초(ms) 단위의 정밀한 시간 제약을 요구한다. 미들웨어는 예측 가능한 시간 내에 데이터를 전달하여 제어 루프의 안정성을 보장해야 한다.15
- **높은 처리량(High Throughput):** 3D 라이다, 스테레오 카메라, 고해상도 이미지 센서 등은 초당 수십에서 수백 메가바이트(MB)에 달하는 방대한 양의 데이터를 생성한다. 미들웨어는 이러한 대용량 데이터를 손실이나 큰 지연 없이 처리하여 로봇의 '인식(perception)' 능력을 뒷받침해야 한다.17
- **낮은 지연 시간(Low Latency):** 원격 조종(teleoperation)이나 인간-로봇 상호작용(HRI)에서는 조작자의 입력이 로봇의 반응으로 즉각 이어져야 한다. 낮은 종단간(end-to-end) 지연 시간은 원활하고 안전한 원격 작업을 위한 필수 조건이다.
- **모듈성 및 확장성(Modularity & Scalability):** 로봇 소프트웨어는 다양한 기능을 수행하는 여러 개의 독립적인 모듈(노드)로 구성된다. 미들웨어는 이러한 모듈 간의 통신을 원활하게 하고, 새로운 기능을 쉽게 추가하거나 시스템을 확장할 수 있도록 지원해야 한다.18
- **상호운용성(Interoperability):** 로보틱스 분야의 사실상 표준 프레임워크인 ROS(Robot Operating System) 및 ROS 2와의 완벽한 통합은 필수적이다. 미들웨어는 ROS 2의 통신 시스템(RMW, ROS Middleware) 계층과 호환되어 기존의 방대한 ROS 생태계와 도구들을 활용할 수 있어야 한다.2

이러한 요구사항들은 이동성이 높고 무선 통신에 의존하는 최신 로봇 시스템에서 더욱 중요해지며, 기존 통신 미들웨어에 상당한 부담을 주고 있다.


Zenoh는 ROS 2의 RMW(ROS Middleware) 구현체(`rmw_zenoh_cpp`)를 공식적으로 지원하며, ROS 2 생태계 내에서 강력한 대안으로 부상하고 있다.19 Zenoh가 특히 주목받는 이유는, ROS 2의 기본 DDS 구현체들(CycloneDDS, FastDDS 등)이 어려움을 겪는 특정 시나리오에서 월등한 성능을 보이기 때문이다.

여러 실증 연구에 따르면, Zenoh는 특히 Wi-Fi나 4G/5G와 같은 무선 및 손실이 많은 네트워크 환경에서 표준 DDS 구현체보다 훨씬 낮은 지연 시간과 높은 처리량을 제공한다.2 이는 앞서 언급했듯이, DDS의 UDP 멀티캐스트 기반 탐색 프로토콜이 무선 환경에서 비효율적으로 작동하는 반면, Zenoh는 이러한 환경에 더 잘 적응하도록 설계되었기 때문이다.

이러한 성능 차이가 실제 로봇의 운영 능력에 어떤 영향을 미치는지를 가장 명확하게 보여주는 사례가 바로 '터틀봇4(TurtleBot 4) 궤적 이탈(trajectory drift) 테스트'이다.2 이 테스트에서는 동일한 Wi-Fi 환경에서 노트북이 로봇에게 사각형 경로를 따라 이동하라는 속도 명령을 전송하고, 모션 캡처 시스템으로 실제 이동 경로를 측정했다. 그 결과, Zenoh를 미들웨어로 사용한 로봇이 목표 경로를 가장 정확하게 추종했으며 시간이 지나도 궤적 이탈이 가장 적었다. 반면, DDS나 MQTT를 사용한 로봇은 시간이 지남에 따라 점점 더 경로에서 벗어나는 모습을 보였다.

이 테스트 결과는 매우 중요한 시사점을 가진다. 이는 Zenoh의 성능 우위가 단순히 '지연 시간이 몇 밀리초 더 짧다'거나 '처리량이 몇 Mbps 더 높다'는 추상적인 벤치마크 수치에 그치지 않고, '로봇이 얼마나 더 정확하게 움직이는가'라는 구체적이고 실질적인 운영 능력의 향상으로 직접 이어진다는 것을 물리적으로 입증한 것이다. 이동 로봇이나 드론에게 위치 및 경로의 정확성은 시스템의 효율성, 신뢰성, 그리고 안전을 결정하는 가장 핵심적인 성능 지표다. 제어 루프는 지속적인 명령 전송과 센서 피드백에 의존하는데, 무선 네트워크에서 흔히 발생하는 패킷 손실과 지터(jitter)는 이 루프의 타이밍을 방해하여 로봇의 움직임을 불안정하게 만든다. Zenoh는 이러한 네트워크 불완전성을 더 효과적으로 처리함으로써, 결과적으로 더 부드럽고 정확한 로봇 제어를 가능하게 한다. 따라서 자율 창고 로봇 플릿이나 정밀 검사 드론을 설계하는 시스템 아키텍트에게 미들웨어의 선택은 단순한 부품 선정을 넘어, 최종 제품의 핵심 가치와 안전성을 좌우하는 전략적 결정이 된다.


Zenoh의 아키텍처는 단일 로봇 시스템을 넘어 다중 로봇 시스템(스웜)과 원격 조종(teleoperation)과 같은 고도의 로보틱스 애플리케이션에 특히 적합한 특성을 가지고 있다.24

- **스웜 로보틱스:** 다수의 로봇이 협력하여 작업을 수행하는 스웜 시스템에서는 로봇 간의 다대다(many-to-many) 통신이 빈번하게 발생한다. Zenoh의 낮은 와이어 오버헤드(최소 4-5 바이트)는 수많은 메시지가 오가는 스웜 환경에서 네트워크 대역폭을 효율적으로 사용하는 데 결정적인 역할을 한다.3 또한, 중앙 브로커가 없는 분산 아키텍처는 스웜 전체의 통신을 담당하는 중앙 노드가 없어 병목 현상이나 단일 장애점 없이 시스템 규모를 확장할 수 있게 해준다.10

- **원격 조종:** 인터넷을 통해 로봇을 원격으로 제어하는 경우, 효율적인 WAN 라우팅이 필수적이다. Zenoh의 라우팅 인프라는 지리적으로 떨어진 조작자와 로봇 간의 통신 경로를 최적화하고, NAT 환경에서도 원활한 연결을 보장한다.1 특히 Zenoh가 제공하는 DDS 브릿지(

  `zenoh-plugin-dds`)는 기존에 로컬 네트워크(LAN)에서 DDS를 사용하던 단일 로봇 시스템을 큰 수정 없이 인터넷이나 5G망을 통해 원격으로 제어하거나, 여러 로봇을 묶어 원격 관제하는 시스템으로 손쉽게 확장할 수 있는 강력한 기능을 제공한다.25

결론적으로 Zenoh는 첨단 로보틱스가 직면한 두 가지 주요 과제, 즉 '단일 로봇에서 다중 로봇으로의 확장(scale-out)'과 '로컬 제어에서 원격 제어로의 확장(scale-up)'에 대한 효과적인 해답을 아키텍처 수준에서 제공한다. 다른 미들웨어를 사용할 경우 복잡한 맞춤형 게이트웨이나 브릿지를 개발해야 해결할 수 있었던 문제들을 Zenoh는 네이티브 기능으로 지원함으로써, 차세대 로봇 시스템 개발을 가속화할 수 있다.


결론적으로, Zenoh는 광범위한 로보틱스 및 드론 애플리케이션에 있어 단순한 대안을 넘어, 잠재적으로 더 우월한 통신 패브릭(communication fabric)이다. 특히 이동성, 무선 통신, 분산 지능을 특징으로 하는 시스템에서 그 가치가 극대화된다. Zenoh의 성능상 이점은 측정 가능한 수치를 넘어, 로봇의 정밀도와 신뢰성이라는 실질적인 운영 효율성 향상으로 이어진다. 따라서 새로운 로봇 시스템 설계 시 Zenoh를 최우선적으로 고려하고, 기존 ROS 2 기반 시스템의 확장 및 성능 개선을 위해 Zenoh로의 전환을 적극적으로 검토할 것을 강력히 권고한다.

------



군사 작전, 특히 최전선에 위치한 전술적 엣지(tactical edge) 환경에서의 통신은 상용 환경과는 비교할 수 없을 정도로 열악하고 예측 불가능하다. 이러한 환경은 종종 **DIL(Disconnected, Intermittent, and Limited bandwidth)** 네트워크로 특징지어진다.28 DIL 환경의 주요 통신 도전 과제는 다음과 같다.

- **불안정한 연결성과 제한된 대역폭:** 전술 네트워크는 본질적으로 무선 애드혹(ad-hoc) 링크에 의존하므로, 적의 전파 방해(jamming), 지형적 장애물, 노드의 이동 등으로 인해 연결이 수시로 끊어지고(unreliable connectivity), 사용 가능한 대역폭이 매우 낮고 가변적(limited and variable bandwidth)이다.30
- **높은 이동성과 동적 토폴로지:** 작전 중 부대의 이동은 네트워크 토폴로지를 끊임없이 변화시키며, 이로 인해 노드 간의 연결이 예측 불가능하게 생성되고 소멸된다. 이는 안정적인 통신 경로 유지를 극도로 어렵게 만든다.31
- **이기종 애플리케이션 및 데이터:** 전술 네트워크는 위치 정보를 공유하는 Blue Force Tracking(BFT), 로봇 원격 제어, 음성 통신(VoIP), 센서 데이터 스트리밍 등 다양한 유형의 데이터를 동시에 처리해야 한다. 각 데이터는 신뢰성, 지연 시간, 순서 보장 등 서로 다른 서비스 품질(QoS) 요구사항을 가진다.31
- **데이터 우선순위 관리의 중요성:** 제한된 대역폭 하에서 모든 데이터를 동일하게 처리하는 것은 비효율적이고 위험하다. 적 위치 정보나 교전 명령과 같은 임무 핵심(mission-critical) 데이터는 일상적인 보고나 저해상도 비디오 피드보다 높은 우선순위를 갖고 먼저 전송되어야 한다.29

이러한 극한의 환경에서 효과적인 정보 유통을 보장하기 위해, 군사 통신 연구는 오래전부터 애플리케이션의 요구사항을 이해하고 네트워크 상태에 동적으로 적응할 수 있는 특화된 "통신 미들웨어"의 필요성을 강조해왔다.30


Zenoh의 아키텍처는 DIL 환경이 제기하는 도전 과제들을 해결하는 데 매우 적합한 특성들을 다수 포함하고 있다. 다음 표는 DIL 환경의 요구사항과 Zenoh의 기능을 직접적으로 매핑하여 그 적합성을 분석한 것이다.

**표 2: 전술적 DIL 환경 요구사항에 대한 Zenoh 기능 매핑**

| DIL 도전 과제                 | 요구되는 능력                                      | 관련된 Zenoh 기능                                            | 증거 자료 |
| ----------------------------- | -------------------------------------------------- | ------------------------------------------------------------ | --------- |
| **불안정한 연결성**           | 단일 장애점(SPOF) 제거, 장애 감내(Fault Tolerance) | 분산형 P2P 및 라우팅 아키텍처, 브로커 의존성 없음            | 1         |
| **제한된 대역폭**             | 낮은 프로토콜 오버헤드, 효율적인 대역폭 사용       | 최소 4바이트의 와이어 오버헤드, 자동 메시지 배치(Batching), 투명한 압축 지원 | 3         |
| **높은 이동성/동적 토폴로지** | 최적의 통신 경로 동적 탐색 및 세션 유지            | 효율적인 탐색(Scouting) 프로토콜, 라우터 간 세션 마이그레이션(Session Migration) 지원 | 10        |
| **데이터 우선순위 관리**      | 메시지별 중요도에 따른 차등 전송                   | 메시지별 우선순위(Priority) 설정 (e.g., REAL_TIME, DATA_LOW 등) | 32        |
| **이기종 전송 환경**          | 다양한 물리적 통신 매체 지원                       | TCP, UDP, QUIC 외 직렬(Serial), CANbus 등 비-IP 전송 계층 지원 | 1         |
| **데이터 손실 및 재동기화**   | 연결 복구 시 데이터 일관성 보장                    | 신뢰성 있는(Reliable) 전송 모드, 저장소 간의 안티-엔트로피(Anti-Entropy) 동기화 프로토콜 | 35        |

이 표에서 볼 수 있듯이, Zenoh의 핵심 아키텍처 특성들은 DIL 환경의 문제점들을 직접적으로 해결하도록 설계되었다. 분산 아키텍처는 중앙 서버의 파괴나 고장에도 통신망이 유지되게 하며, 극도로 낮은 오버헤드는 귀중한 전술 대역폭을 절약해준다. 특히, 노드가 이동함에 따라 가장 가까운 라우터로 통신 세션을 원활하게 이전하는 '세션 마이그레이션' 기능은 기동성이 높은 전술 환경에서 통신 품질을 최적으로 유지하는 데 결정적인 역할을 한다.10 또한, 연결이 끊겼다가 복구되었을 때 분산된 저장소 간의 데이터 일관성을 점진적으로 맞춰주는 '안티-엔트로피' 프로토콜은 DIL 환경에서 데이터 동기화를 위한 강력한 메커니즘을 제공한다.35


DIL 환경에서는 단순히 데이터를 전송하는 것을 넘어, '어떤 데이터를', '얼마나 신뢰성 있게', '네트워크가 혼잡할 때 어떻게' 전송할지를 제어하는 능력이 매우 중요하다. Zenoh는 이를 위해 세분화된 제어 메커니즘을 제공한다.

- **신뢰성(Reliability):** Zenoh는 두 가지 수준의 신뢰성을 제공한다. `Best-Effort`는 UDP처럼 메시지를 한 번만 전송하며, 손실될 수 있다. `Reliable`은 TCP처럼 메시지 수신을 확인하고, 손실 시 재전송하여 전달을 보장한다.32 애플리케이션은 각 데이터 스트림의 중요도에 따라 이 모드를 선택할 수 있다. Zenoh는 또한 홉-바이-홉(hop-by-hop) 신뢰성과 종단간(end-to-end) 신뢰성이라는 더 세분화된 개념을 제공하여, 시스템 아키텍트가 확장성과 신뢰성 사이의 균형을 정밀하게 조절할 수 있게 한다.36

- **혼잡 제어(Congestion Control):** 느린 수신자나 네트워크 병목으로 인해 송신측에 데이터가 쌓일 때 어떻게 대처할지를 결정하는 중요한 기능이다. Zenoh는 `Block`과 `Drop` 두 가지 전략을 제공한다.32

  `Block` 모드는 큐가 가득 차면 새로운 데이터 전송을 차단하여 진행(progress)을 희생하는 대신 데이터 손실을 막는다. `Drop` 모드는 오래된 데이터를 버리고 새로운 데이터를 전송하여 진행을 유지하지만 데이터 손실을 감수한다. BFT와 같이 최신 정보가 중요한 데이터는 `Drop`이, 파일 전송과 같이 모든 데이터가 필요한 경우는 `Block`이 적합할 수 있다. 이처럼 Zenoh는 신뢰성과 혼잡 제어를 분리하여, 애플리케이션이 상황에 맞는 최적의 트레이드오프를 선택할 수 있도록 한다.

- **우선순위(Priority):** Zenoh는 각 메시지에 우선순위 태그를 부여할 수 있다. `REAL_TIME`, `INTERACTIVE_HIGH`, `DATA`, `DATA_LOW`, `BACKGROUND` 등 7단계의 우선순위를 통해, 통신 미들웨어는 네트워크 자원이 제한될 때 어떤 메시지를 먼저 처리하고 전송할지 결정할 수 있다.32 이는 긴급한 교전 명령이 대용량 정찰 사진 전송에 밀려 지연되는 상황을 방지하는 데 필수적인 기능이다.34

이러한 정교한 제어 기능들은 DIL 환경에서 "모든 것을 동일하게" 처리하는 방식의 비효율성과 위험성을 극복하고, 통신 자원을 작전 목표에 맞게 지능적으로 배분할 수 있는 능력을 제공한다.31


Zenoh의 군사적 잠재력은 이론에만 머무르지 않고 실제 시연을 통해 입증된 바 있다. 호주의 방산 컨설팅 기업인 DefenceX는 호주 육군의 주요 관계자들을 대상으로 Zenoh의 능력을 시연했다.27

시연의 구성은 다음과 같다:

1. 사무실 Wi-Fi에 연결된 터틀봇(Turtlebot) 로봇
2. 5G 모바일 네트워크에 연결된 DDS 기반의 지휘 통제 센터
3. 시드니에 위치한 사설 서버에서 실행되는 Zenoh 라우터

이 시연에서 Zenoh 라우터는 서로 다른 네트워크(Wi-Fi와 5G)와 서로 다른 미들웨어(로봇의 ROS 2와 통제 센터의 DDS) 사이를 연결하는 '브릿지' 역할을 수행했다. 이를 통해 5G 망에 있는 지휘관이 Wi-Fi 망에 있는 로봇을 원활하게 원격 제어할 수 있었다.27

이 사례 연구는 Zenoh의 군사적 가치를 명확하게 보여준다. 이는 단순히 새로운 시스템을 위한 프로토콜이 아니라, 기존의 DDS 기반 자산(레거시 시스템)을 이기종 광역 네트워크를 통해 통합하고 확장할 수 있는 핵심적인 '연결 기술'로서의 가치를 입증한 것이다. 이는 전 세계에 분산된 로봇 자산을 현지에서 원격으로 운용하거나, 서로 다른 통신 체계를 가진 동맹군과의 상호운용성을 확보하는 등 국방 분야의 중요한 과제를 해결할 수 있는 거대한 기회를 제시한다.

------



Zenoh는 설계 초기부터 보안을 고려하여 여러 강력한 내재적 보안 기능을 갖추고 있다.

- **메모리 안전성(Memory Safety):** Zenoh의 핵심 구현체는 Rust 언어로 작성되었다.9 Rust는 언어 차원에서 버퍼 오버플로우, 댕글링 포인터와 같은 메모리 관련 취약점을 컴파일 시점에 원천적으로 방지한다. 이는 C/C++로 작성된 많은 기존 미들웨어가 겪는 고질적인 보안 문제의 상당 부분을 제거하여 프로토콜의 근본적인 안정성을 높인다.
- **인증(Authentication):** Zenoh는 플러그형 인증 메커니즘을 지원하며, 특히 상호 TLS(mTLS) 인증을 통해 강력한 노드 신원 확인 기능을 제공한다.3 mTLS를 사용하면, 네트워크에 참여하는 모든 노드(클라이언트, 라우터)가 암호학적으로 검증된 인증서를 통해 서로의 신원을 확인하므로, 인가되지 않은 장치가 네트워크에 접속하는 것을 막을 수 있다.
- **인가/접근 제어(Authorization/Access Control):** 인증을 통해 신원이 확인된 노드라 할지라도, 모든 데이터에 접근할 수 있는 것은 아니다. Zenoh는 접근 제어 목록(ACLs)을 사용하여 각 노드가 특정 키 표현식(key expression)에 대해 수행할 수 있는 작업(게시, 구독, 쿼리 등)을 세밀하게 제어할 수 있다.9 이를 통해 '지휘관' 역할의 노드는 모든 데이터에 접근할 수 있지만, '일선 센서' 역할의 노드는 자신의 데이터만 게시하고 제한된 정보만 구독하도록 정책을 강제할 수 있다.

이러한 기능들은 Zenoh가 기본적인 보안 요구사항을 충족하는 견고한 기반을 갖추고 있음을 보여준다. 다음 표는 Zenoh의 보안 모델을 기능별로 요약하고 그 한계를 명시한 것이다.

**표 3: Zenoh 보안 모델: 역량 및 한계점**

| 보안 영역               | Zenoh의 구현 방식                      | 주요 한계점 / 갭                                             | 증거 자료 |
| ----------------------- | -------------------------------------- | ------------------------------------------------------------ | --------- |
| **메모리 안전성**       | Rust 언어 기반 구현                    | 해당 없음 (강점)                                             | 9         |
| **인증**                | 상호 TLS(mTLS)를 통한 강력한 노드 인증 | 없음 (플러그형으로 확장 가능)                                | 3         |
| **접근 제어**           | 키 표현식 기반의 세분화된 ACL          | 없음 (강력한 기능)                                           | 9         |
| **기밀성 (홉-바이-홉)** | TLS를 통한 노드 간 통신 채널 암호화    | 없음 (표준 기술 사용)                                        | 3         |
| **기밀성 (종단간)**     | **네이티브 지원 부재**                 | **중간 라우터에서 데이터가 평문으로 노출됨 (치명적 취약점)** | 3         |

이 표는 Zenoh의 보안 태세가 '강력한 기반'과 '치명적인 아키텍처적 갭'이 공존하는 양면성을 가지고 있음을 명확히 보여준다.


Zenoh의 보안 아키텍처에서 가장 심각하고 반복적으로 지적되는 약점은 네이티브 종단간 암호화(End-to-End Encryption, E2EE)의 부재이다.3 Zenoh는 TLS를 사용하여 노드와 노드 사이의 통신 채널을 암호화하지만, 이 암호화는 홉-바이-홉(hop-by-hop) 방식으로 적용된다.

이는 데이터가 송신자(publisher)로부터 수신자(subscriber)에게 전달되는 과정에서 중간에 위치한 모든 Zenoh 라우터를 거칠 때마다 암호화가 해제(복호화)되었다가 다시 암호화된다는 것을 의미한다. 즉, **각 라우터의 메모리 내에서는 데이터가 잠시 평문(plaintext) 상태로 존재하게 된다.**

이러한 구조는 군사적 맥락에서 심각한 취약점으로 작용한다. 전술 네트워크의 라우터와 같은 중간 인프라는 항상 신뢰할 수 있는 아군 통제 하에 있다고 가정할 수 없다. 적군이 물리적으로 라우터를 탈취하거나 원격으로 해킹하여 제어권을 획득할 경우, 해당 라우터를 통과하는 모든 통신 내용을 평문으로 가로채거나(eavesdropping), 심지어 내용을 변조한 후(tampering) 다음 노드로 전달할 수 있다. 예를 들어, 아군의 위치 정보를 적군의 위치 정보로 바꾸거나, 공격 명령을 철수 명령으로 위조하는 등의 공격이 가능해진다.

이러한 홉-바이-홉 암호화 방식의 취약점은 Zenoh가 라우팅, 필터링, 데이터 변환 등 메시지 내용에 접근해야 하는 기능을 수행하기 위한 아키텍처적 선택의 결과이다. 그러나 보안이 최우선시되는 환경에서는 데이터의 기밀성(confidentiality)과 무결성(integrity)이 전송 경로 전체에 걸쳐 보장되어야 한다. 따라서 E2EE의 부재는 Zenoh를 현재 상태 그대로 민감 정보나 기밀 정보를 다루는 시스템에 적용하는 것을 근본적으로 불가능하게 만드는 결정적인 장애물이다.


Zenoh의 E2EE 부재 문제를 해결하기 위해 보안 분석가들은 두 가지 주요 완화 경로를 제시한다.3 그러나 이 해결책들은 상당한 공학적 부담을 시스템 통합 업체에게 전가한다는 공통점을 가진다.

1. 애플리케이션 레벨 암호화 (Application-Level Encryption):

   송신 측 클라이언트 애플리케이션이 데이터를 Zenoh에 게시하기 전에 자체적으로 페이로드를 암호화하고, 수신 측 클라이언트 애플리케이션이 데이터를 받은 후에 이를 복호화하는 방식이다. 이 경우, 데이터는 Zenoh 네트워크를 통과하는 내내 암호화된 상태로 유지되므로, 중간 라우터가 탈취되어도 데이터의 내용은 안전하게 보호된다.

2. 맞춤형 보안 플러그인 (Custom Security Plugins):

   Zenoh의 플러그형 아키텍처를 활용하여 E2EE를 투명하게 처리하는 전용 보안 플러그인을 개발하는 방식이다.39 이 플러그인은 게시 시 자동으로 데이터를 암호화하고, 구독 시 자동으로 복호화하는 역할을 수행하여 애플리케이션 개발자가 암호화 로직을 직접 다루지 않도록 할 수 있다.

이러한 완화책들은 이론적으로는 타당하지만, 실제 구현은 결코 간단하지 않다. 이는 '직접 해결하라(do it yourself)'는 접근 방식으로, 암호화와 관련된 복잡하고 어려운 엔지니어링 문제를 시스템 통합 업체가 모두 책임져야 함을 의미한다. 강력한 암호화를 구현하는 것은 단순히 암호화/복호화 알고리즘을 호출하는 것에서 그치지 않는다. 이는 안전하고, 확장 가능하며, 관리 가능한 **키 관리 인프라(Key Management Infrastructure, KMI)**의 구축을 필연적으로 동반한다. KMI는 암호화 키의 생성, 배포, 교체(rotation), 폐기(revocation) 등 키의 전체 생명주기를 관리하는 복잡한 시스템이다. 이러한 시스템을 처음부터 안전하게 설계하고 구축하는 것은 그 자체로 막대한 비용과 시간이 소요되는 고위험 프로젝트이다.

결론적으로, E2EE 갭에 대한 완화책은 Zenoh 도입의 비용-편익 분석을 근본적으로 바꾸어 놓는다. 국방 계약업체가 Zenoh를 선택한다면, 단순히 미들웨어 통합 비용뿐만 아니라, E2EE 계층과 KMI를 구축하기 위한 별도의 복잡하고 위험 부담이 큰 보안 개발 프로젝트 예산을 함께 책정해야 한다. 이는 잠재적으로 전체 통합 노력을 두 배로 늘리고 프로젝트의 위험 프로파일을 크게 높일 수 있다.


Zenoh의 현재 보안 태세는 강력하고 현대적인 기반 위에 중대한 아키텍처적 격차가 존재하는 혼합적인 형태다. 비기밀 데이터나 덜 민감한 정보를 다루는 경로(예: 비작전 물류 정보)에서는 내장된 보안 기능만으로도 충분할 수 있다. 그러나 민감 정보나 기밀 정보를 처리하는 모든 군용 및 항공 시스템의 경우, Zenoh의 도입은 강력하고 검증 가능한 E2EE 오버레이의 성공적인 구현을 전제 조건으로 한다. 이는 단순한 기술 통합을 넘어, 상당한 규모의 보안 엔지니어링 프로젝트를 수반하는 전략적 결정임을 명확히 인지해야 한다.

------



항공기, 특히 상업용 및 군용 항공기에 탑재되는 모든 소프트웨어는 상상할 수 있는 가장 엄격한 안전성 요구사항을 충족해야 한다. 이를 보장하기 위한 업계 표준이 바로 **DO-178C, "항공 시스템 및 장비 인증에 관한 소프트웨어 고려사항"**이다.40 DO-178C는 미국 연방항공청(FAA), 유럽항공안전청(EASA) 등 전 세계 항공 인증 당국이 소프트웨어의 안전성을 승인하는 주요 근거 문서다.

DO-178C의 핵심 원칙은 다음과 같다:

- **객체 기반(Objective-based):** 이 표준은 특정 개발 방법론(예: Agile, Waterfall)을 강요하지 않는다. 대신, 소프트웨어 개발 생명주기 동안 반드시 만족해야 하는 '객체(objective)'들의 목록을 정의한다. 프로젝트는 어떤 개발 프로세스를 따르든, 이 객체들을 모두 만족했음을 객관적인 증거로 입증해야 한다.41
- **설계 보증 수준(Design Assurance Levels, DALs):** 소프트웨어의 고장이 항공기, 승무원, 승객에게 미치는 영향의 심각도에 따라 5개의 등급(A부터 E까지)으로 분류된다.41 DAL A는 '파국적(Catastrophic)'인 고장(항공기 손실 및 인명 사망)을 유발할 수 있는 소프트웨어에 적용되며 가장 엄격한 요구사항을 가진다. 반면 DAL E는 안전에 영향이 없는 소프트웨어에 해당한다.
- **독립성(Independence):** 높은 DAL(A, B, C)에서는 특정 검증 활동이 개발자로부터 '독립적인' 인원이나 팀에 의해 수행되어야 한다. 이는 개발자가 자신의 작업물을 스스로 검증함으로써 발생할 수 있는 오류를 방지하기 위함이다.41
- **양방향 추적성(Bidirectional Traceability):** DO-178C의 핵심 중 하나는 모든 개발 산출물 간의 완벽한 양방향 추적성을 요구한다는 점이다.40 예를 들어, 시스템의 상위 요구사항은 이를 구현하는 하위 요구사항으로, 하위 요구사항은 이를 구현하는 소스 코드로, 소스 코드는 이를 검증하는 테스트 케이스로, 테스트 케이스는 그 실행 결과로까지 끊김 없이 연결되어야 한다. 이 추적성은 모든 요구사항이 코드로 구현되고, 모든 코드가 테스트되었으며, 목적 없는 코드가 존재하지 않음을 보장한다.

이러한 원칙들은 DO-178C 인증이 단순히 최종 코드의 품질을 평가하는 것이 아니라, 그 코드를 만들어낸 '프로세스'의 엄격함, 문서화, 추적성을 평가하는 것임을 명확히 보여준다.


현재의 오픈소스 Zenoh 프로젝트를 DO-178C의 요구사항에 비추어 볼 때, 그 격차는 매우 크다. DO-178C 인증을 위해서는 코드 외에도 방대한 양의 공식적인 '인증 산출물(certification artifacts)'이 필요한데, 현재 Zenoh 프로젝트는 이러한 산출물을 거의 가지고 있지 않다.

다음 표는 DO-178C에서 요구하는 주요 산출물과 현재 Zenoh의 상태를 비교하여 예비 격차를 분석한 것이다.

**표 4: Zenoh의 예비 DO-178C 준수 격차 분석**

| DO-178C 산출물/프로세스                   | 설명                                                         | 현재 Zenoh 상태                                              | 준수 경로                                                    |
| ----------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **소프트웨어 인증 계획서 (PSAC)**         | 개발, 검증, 형상 관리, 품질 보증 등 전체 인증 프로세스를 기술하는 최상위 계획 문서 | **부재**                                                     | 처음부터 새로 작성 필요. 시스템의 DAL 수준에 맞춰 모든 계획 수립. |
| **소프트웨어 요구사항 데이터 (SRD)**      | 시스템의 기능적, 비기능적 요구사항을 공식적으로 기술한 문서  | **부재** (비공식적 로드맵, 기능 설명만 존재)                 | Zenoh의 모든 기능을 공식적인 요구사항 형식으로 재정의하고 문서화. |
| **소프트웨어 설계 문서 (SDD)**            | 요구사항을 만족시키기 위한 소프트웨어 아키텍처 및 상세 설계를 기술한 문서 | **부재** (개념적 설명은 있으나 공식 설계 문서 없음)          | 요구사항에 기반하여 아키텍처와 상세 설계를 처음부터 문서화.  |
| **양방향 추적성 매트릭스**                | 요구사항-설계-코드-테스트 간의 연결 관계를 보여주는 매트릭스 | **부재**                                                     | 모든 산출물 생성 후, 각 요소 간의 추적성을 수동 또는 도구를 통해 설정. |
| **소프트웨어 검증 케이스 및 절차 (SVCP)** | 모든 요구사항을 검증하기 위한 테스트 케이스와 절차를 기술한 문서 | **부재** (단위/통합 테스트는 있으나 인증 목적의 공식 문서 없음) | 모든 요구사항에 대해 100% 커버리지를 목표로 테스트 케이스 및 절차 개발. |
| **구조적 커버리지 분석**                  | 테스트가 코드의 모든 부분을 실행했음을 증명하는 분석 (e.g., Statement, Decision Coverage) | **부재**                                                     | DAL 수준에 맞는 커버리지 분석 도구를 사용하여 분석 수행 및 결과 문서화. |

데이터 소스: 40 (DO-178C 요구사항); 44 (Zenoh 상태)

이 표는 Zenoh가 항공 분야에 적용되기 위해 넘어야 할 '인증의 심연(certification chasm)'이 얼마나 깊은지를 명확하게 보여준다. 이는 단순히 몇 가지 기능을 추가하거나 버그를 수정하는 차원의 문제가 아니라, 프로젝트 전체의 개발 철학과 프로세스를 DO-178C의 틀에 맞춰 처음부터 재구성해야 하는 거대한 작업이다.


Zenoh가 Rust로 작성되었다는 점은 안전성 측면에서 긍정적인 요소다. Rust의 메모리 안전성 보장은 잠재적인 소프트웨어 결함의 상당 부분을 제거하여 검증 노력을 줄이는 데 도움이 될 수 있다. 최근 업계에서는 Rust를 안전 최우선 시스템에 사용하려는 움직임이 활발하며, Ferrous Systems의 Ferrocene과 같이 DO-178C와 같은 표준에 맞게 '검증된(qualified)' Rust 컴파일러 툴체인도 등장하고 있다.47

그러나 이는 Rust 사용이 인증의 '프리패스'를 의미하지는 않는다는 점을 명심해야 한다. DO-330 "소프트웨어 툴 검증 고려사항"에 따라, 인증 대상 소프트웨어를 개발하는 데 사용된 모든 도구(컴파일러, 링커, 정적 분석기 등)는 그 자체로 신뢰성을 검증받아야 한다.40 즉, 특정 버전의 Rust 컴파일러를 포함한 전체 툴체인을 프로젝트에 맞게 검증하는 과정은 그 자체로 상당한 노력과 비용이 드는 작업이다. 따라서 Rust의 사용은 인증 과정을 '용이하게' 할 수는 있지만, 그 엄격한 요구사항을 '면제'해주지는 않는다.


현재 Zenoh의 공식 로드맵이나 커뮤니티 논의에서 DO-178C 인증에 대한 구체적인 계획은 찾아볼 수 없다.46 이는 빠르게 변화하는 커뮤니티 기반 오픈소스 프로젝트의 개발 모델과, 막대한 비용과 시간이 소요되며 엄격한 형식주의를 요구하는 DO-178C 인증 프로세스가 근본적으로 양립하기 어렵기 때문이다.48

그러나 여기서 중요한 단서를 자동차 기능 안전 표준인 ISO 26262 관련 논의에서 찾을 수 있다. Zenoh 커뮤니티 포럼의 질문에 대해, ZettaScale(Zenoh의 핵심 개발사) 관계자는 ISO 26262 인증을 진행하고 있으며, 이는 **상업적 제공(commercial offering)**의 일부라고 명확히 밝혔다.44

이 사실로부터 우리는 Zenoh의 항공 분야 인증 경로에 대한 논리적인 추론을 할 수 있다. ISO 26262보다 훨씬 더 엄격하고 비용이 많이 드는 DO-178C 인증 역시, ZettaScale이나 이와 유사한 전문 기업이 별도의 상업용 라이선스 제품으로 제공할 가능성이 매우 높다. 즉, 항공 시스템 개발사가 미래에 사용하게 될 '인증된 Zenoh'는 현재의 오픈소스 Zenoh가 아니라, 인증에 필요한 모든 산출물(PSAC, SDD, 테스트 결과 등)이 포함된 고가의 '인증 데이터 패키지'와 함께 제공되는 상업용 소프트웨어일 것이다.

이 추론은 항공우주 기업의 시스템 아키텍트에게 매우 중요한 전략적 시사점을 제공한다. 이는 단순히 'Zenoh라는 오픈소스 기술을 사용한다'는 기술적 평가에서, '특정 상업용 벤더로부터 인증된 Zenoh 제품을 조달한다'는 구매 및 파트너십 전략으로 문제의 성격을 완전히 바꾸어 놓는다. 따라서 고려해야 할 사항은 Zenoh의 기술적 장점뿐만 아니라, 해당 벤더의 장기적인 비전과 지원 모델, 그리고 상당한 비용이 예상되는 인증 데이터 패키지의 조달 전략까지 포함하게 된다.

------



본 보고서의 분석을 종합하면, Zenoh 프로토콜의 강점과 약점은 다음과 같이 요약될 수 있다.

**강점:**

- **압도적인 성능:** 특히 무선 및 불안정 네트워크 환경에서 기존 미들웨어를 능가하는 높은 처리량과 낮은 지연 시간을 제공한다.
- **탁월한 아키텍처 유연성:** P2P, 라우팅, 브로커 모델을 모두 지원하여 다양한 네트워크 토폴로지에 유연하게 적응할 수 있다.
- **뛰어난 확장성:** 분산 아키텍처와 효율적인 라우팅을 통해 로컬 네트워크에서 인터넷 규모의 지리적 분산 시스템까지 원활하게 확장된다.
- **낮은 오버헤드:** 최소 4바이트의 와이어 오버헤드는 리소스가 제한된 장치나 대역폭이 협소한 네트워크에 매우 유리하다.

**약점:**

- **E2EE 부재:** 네이티브 종단간 암호화의 부재는 중간 노드에서의 데이터 노출이라는 심각한 보안 취약점을 야기한다.
- **안전성 인증 부재:** DO-178C와 같은 안전 최우선 시스템을 위한 인증 경로 및 관련 산출물이 전무하다.
- **생태계 성숙도:** DDS나 MQTT와 같은 기존 기술에 비해 커뮤니티, 서드파티 도구, 상용 지원 등의 생태계가 아직 성장 단계에 있다.


Zenoh 프로젝트의 공식 로드맵과 커뮤니티 활동을 살펴보면, 프로젝트가 매우 활발하고 빠르게 발전하고 있음을 알 수 있다.45 Zenoh 1.0 "Firesong"의 출시와 "Gozuryū"와 같은 후속 업데이트는 API 안정화, 성능 개선, 공유 메모리 서브시스템 재작업, Zenoh-Flow와 같은 새로운 데이터 플로우 프로그래밍 모델 도입 등 기능적 성숙에 초점을 맞추고 있다.34 또한, ROS 2와의 통합을 강화하고(

`rmw_zenoh_cpp`), 다양한 언어 바인딩(C++, Python, Java 등)을 지속적으로 개선하는 등 개발자 생태계 확장에도 많은 노력을 기울이고 있다.20

그러나 이러한 발전 방향은 주로 성능, 기능, 사용 편의성에 집중되어 있으며, DO-178C와 같은 공식적인 안전성 인증 프로세스에 필요한 엄격한 문서화, 추적성, 형식적 검증과는 거리가 있다. 이는 5장에서 내린 결론, 즉 인증은 오픈소스 프로젝트의 주된 관심사가 아니며 별도의 상업적 노력을 통해 이루어질 것이라는 분석을 다시 한번 뒷받침한다. Zenoh 생태계는 빠르게 성숙하고 있지만, 그 방향은 안전 최우선 항공 분야보다는 로보틱스, IoT, 자동차 산업에 더 맞춰져 있다.


이상의 분석을 바탕으로, 대한민국 국방 및 항공우주 분야의 시스템 아키텍트를 위해 다음과 같은 단계별, 분야별 도입 전략을 권고한다.

- **로보틱스 및 드론 분야:**
  - **권고:** **단기적인 평가 및 적극적인 도입을 강력히 추천한다.**
  - **근거:** 입증된 성능상의 이점, 특히 무선 환경에서의 우위는 로봇의 정밀도와 신뢰성이라는 실질적인 운영 능력 향상으로 직접 이어진다.2 ROS 2 생태계와의 통합도 원활하며, 이 분야에서 요구하는 기술적 성숙도는 이미 충분히 갖추었다고 판단된다. 새로운 모바일 로봇 및 드론 시스템 설계 시 최우선 미들웨어 후보로 고려해야 한다.
- **군용 시스템 분야:**
  - **권고:** **조건부 도입 및 단계적 접근을 권고한다.**
  - **근거:** Zenoh의 아키텍처는 DIL 환경에 매우 적합하며(표 2 참조), 이는 큰 장점이다. 따라서 비기밀 정보를 다루는 시스템(예: 물류, 기상 정보)이나 프로토타이핑에는 현재 상태로도 즉시 적용을 고려할 수 있다. 그러나 지휘통제(C2), 정찰(ISR) 등 민감 정보를 다루는 시스템에 적용하기 위해서는 **E2EE 계층의 성공적인 개발 및 통합이 반드시 선행되어야 한다.** 따라서, 단기적으로는 비민감 분야에 Zenoh를 적용하여 운영 경험을 축적하고, 이와 병행하여 Zenoh 기반의 강력한 E2EE 플러그인 및 키 관리 시스템(KMI)에 대한 연구개발(R&D) 프로젝트를 즉시 착수하는 '투 트랙(two-track)' 전략이 바람직하다.
- **항공 시스템 분야:**
  - **권고:** **단기 솔루션이 아닌 장기적인 연구개발(R&D) 대상으로 취급해야 한다.** 향후 5-7년 내에 인증이 필요한 시스템에 Zenoh를 적용하는 계획은 배제해야 한다.
  - **근거:** '인증의 심연'(섹션 5.2, 표 4 참조)은 기술적으로나 비용적으로 극복하기 매우 어려운 장벽이다. DO-178C 인증 버전의 Zenoh는 오픈소스가 아닌 고가의 상업용 제품으로 제공될 가능성이 높으며(섹션 5.4 참조), 그 시점 또한 불확실하다. 따라서 현재 단계에서의 관여는 기술 동향을 추적하고, ZettaScale의 상업적 안전성 인증 노력(ISO 26262 등)의 진행 상황을 모니터링하며, 잠재적인 아키텍처 연구나 개념 증명(PoC) 수준에 머물러야 한다.


Zenoh는 데이터 통신 분야에서 잠재적으로 패러다임을 바꿀 수 있는 강력하고 혁신적인 기술이다. 그러나 그 도입은 각 적용 분야의 고유한 요구사항과 한계에 대한 명확한 이해를 바탕으로 전략적으로 이루어져야 한다. 로보틱스 분야에게 Zenoh는 이미 현실화된 미래다. 군용 시스템에게는 신중한 보안 엔지니어링을 통해 열 수 있는 문이다. 그리고 항공 시스템에게 Zenoh는 아직은 먼 지평선 너머에 있지만, 지속적인 관심과 전략적 투자를 통해 도달해야 할 목표다.


1. A Performance Study on the Throughput and Latency of Zenoh, MQTT, Kafka, and DDS - arXiv, accessed July 9, 2025, https://arxiv.org/pdf/2303.09419
2. arXiv:2309.07496v4 [cs.RO] 16 Nov 2024, accessed July 9, 2025, https://arxiv.org/abs/2309.07496
3. Zenoh Protocol Security Analysis - Census Labs, accessed July 9, 2025, https://census-labs.com/news/2025/03/17/zenoh-protocol-security-analysis/
4. census-labs.com, accessed July 9, 2025, [https://census-labs.com/news/2025/03/17/zenoh-protocol-security-analysis/#:~:text=Zenoh%20%E2%80%94%20the%20Zero%20Overhead%20Network,centers%20to%20resource%2Dconstrained%20microcontrollers.](https://census-labs.com/news/2025/03/17/zenoh-protocol-security-analysis/#:~:text=Zenoh - the Zero Overhead Network,centers to resource-constrained microcontrollers.)
5. Zenoh - ZettaScale Technology, accessed July 9, 2025, https://www.zettascale.tech/zenoh/
6. What is Zenoh? / Zenoh - pub/sub, geo distributed storage, query, accessed July 9, 2025, https://zenoh.io/docs/overview/what-is-zenoh/
7. Zenoh - A Protocol That Should be on Your Radar | by Jkel - Medium, accessed July 9, 2025, https://medium.com/@kelj/zenoh-a-protocol-that-should-be-on-your-radar-72befa697411
8. Zenoh - The Zero Overhead, Pub/Sub, Store, Query, and Compute Protocol., accessed July 9, 2025, https://zenoh.io/
9. How does the Zenoh protocol enhance edge device operation? - Microcontroller Tips, accessed July 9, 2025, https://www.microcontrollertips.com/how-does-the-zenoh-protocol-enhance-edge-device-operation/
10. Mobility, Latency and Energy saving / Zenoh - pub/sub, geo distributed storage, query, accessed July 9, 2025, https://zenoh.io/blog/2022-03-30-zenoh-mobility/
11. Comparing the Performance of Zenoh, MQTT, Kafka, and DDS, accessed July 9, 2025, https://zenoh.io/blog/2023-03-21-zenoh-vs-mqtt-kafka-dds/
12. Zenoh's Top Features Released in 2023 - ZettaScale Technology, accessed July 9, 2025, https://www.zettascale.tech/news/zenohs-top-features-released-in-2023/
13. Zenoh Performance - ROS General - Open Robotics Discourse, accessed July 9, 2025, https://discourse.ros.org/t/zenoh-performance/30494
14. QoS: DDS vs Zenoh - Speaker Deck, accessed July 9, 2025, https://speakerdeck.com/kydos/qos-dds-vs-zenoh
15. µRT: A lightweight real-time middleware with integrated validation of timing constraints - Frontiers, accessed July 9, 2025, https://www.frontiersin.org/journals/robotics-and-ai/articles/10.3389/frobt.2023.1081875/full
16. Formal Verification of Real-Time Autonomous Robots: An Interdisciplinary Approach - PMC, accessed July 9, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9043953/
17. Real-Time Drone Communication System Using ROS 2 and GStreamer with YOLOv8-Seg for Face Segmentation - ResearchGate, accessed July 9, 2025, https://www.researchgate.net/publication/392225374_Real-Time_Drone_Communication_System_Using_ROS_2_and_GStreamer_with_YOLOv8-Seg_for_Face_Segmentation
18. Robotic Middleware: Definition & Applications | Vaia, accessed July 9, 2025, https://www.vaia.com/en-us/explanations/engineering/robotics-engineering/robotic-middleware/
19. Roadmap - Programming Multiple Robots with ROS 2 - GitHub Pages, accessed July 9, 2025, https://osrf.github.io/ros2multirobotbook/roadmap.html
20. Google Summer of Code : Zenoh Support & Benchmarking - MoveIt, accessed July 9, 2025, https://moveit.ai/moveit/gsoc/2024/08/19/GSoC-2024-Zenoh-Support-and-Benchmarking.html
21. Zenoh - Jazzy documentation - ROS 2, accessed July 9, 2025, https://docs.ros.org/en/jazzy/Installation/RMW-Implementations/Non-DDS-Implementations/Working-with-Zenoh.html
22. Comparison of DDS, MQTT, and Zenoh in Edge-to-Edge/Cloud Communication with ROS 2, accessed July 9, 2025, https://deepai.org/publication/comparison-of-dds-mqtt-and-zenoh-in-edge-to-edge-cloud-communication-with-ros-2
23. Comparison of Middlewares in Edge-to-Edge and Edge-to-Cloud Communication for Distributed ROS2 Systems | Papers With Code, accessed July 9, 2025, https://paperswithcode.com/paper/comparison-of-dds-mqtt-and-zenoh-in-edge-to
24. Zenoh: The Future of Communication for Edge Computing and IoT - YouTube, accessed July 9, 2025, https://www.youtube.com/watch?v=ahO4kT_Zg7s
25. Overcome DDS Limitations with Zenoh's DDS Plugin and DDS Bridge for Robotic Systems - YouTube, accessed July 9, 2025, https://www.youtube.com/watch?v=9h01_MSKPS0
26. Zenoh - DefenceX, accessed July 9, 2025, https://defencex.ai/wordpress/zenho/
27. Zenoh Bridges gaps in Robotics applications - DefenceX, accessed July 9, 2025, https://defencex.ai/wordpress/zenoh-bridges-gaps-in-robotics-applications/
28. Communications Challenges in DIL Environments | REDCOM, accessed July 9, 2025, https://www.redcom.com/dil-environments/
29. New research shows promising future for warfighter communication | Article - Army.mil, accessed July 9, 2025, https://www.army.mil/article/236948/new_research_shows_promising_future_for_warfighter_communication
30. Communications Middleware for Tactical Environments: Observations, Experiences, and Lessons Learned - ResearchGate, accessed July 9, 2025, https://www.researchgate.net/publication/224597591_Communications_Middleware_for_Tactical_Environments_Observations_Experiences_and_Lessons_Learned
31. Communications Middleware for Tactical Environments ... - DTIC, accessed July 9, 2025, https://apps.dtic.mil/sti/tr/pdf/ADA517067.pdf
32. Commonly used types - zenoh-cpp 1.4.0 documentation, accessed July 9, 2025, https://zenoh-cpp.readthedocs.io/en/stable/commons.html
33. zenoh-python 1.4.0 documentation, accessed July 9, 2025, https://zenoh-python.readthedocs.io/en/latest/
34. Zenoh Gozuryū is Here!, accessed July 9, 2025, https://zenoh.io/blog/2025-04-14-zenoh-gozuryu/
35. Keeping storages aligned in Zenoh, accessed July 9, 2025, https://zenoh.io/blog/2022-11-29-zenoh-alignment/
36. Zenoh Reliability, Scalability and Congestion Control, accessed July 9, 2025, https://zenoh.io/blog/2021-06-14-zenoh-reliability/
37. Category: Autonomous Vehicles - DefenceX, accessed July 9, 2025, https://defencex.ai/wordpress/category/autonomous-vehicles/
38. Access Control / Zenoh - pub/sub, geo distributed storage, query, accessed July 9, 2025, https://zenoh.io/docs/manual/access-control/
39. Writing a Zenoh Plugin to house the uProtocol uStreamer - Pete LeVasseur's Blog [plog], accessed July 9, 2025, https://petelevasseur.com/articles/004-writing-zenoh-plugin.html
40. Navigating the DO-178C Certification Process for Airborne Software - TheCloudStrap, accessed July 9, 2025, https://thecloudstrap.com/navigating-the-do-178c-certification-process/
41. DO-178C - Wikipedia, accessed July 9, 2025, https://en.wikipedia.org/wiki/DO-178C
42. What Is DO-178C? - Wind River, accessed July 9, 2025, https://www.windriver.com/solutions/learning/do-178c
43. DO-178C DAL A Safety-Critical RTOS – INTEGRITY-178 tuMP - Green Hills Software, accessed July 9, 2025, https://www.ghs.com/products/safety_critical/integrity_178_safety_critical.html
44. ISO 26262 certification for Zenoh #167 - GitHub, accessed July 9, 2025, https://github.com/eclipse-zenoh/roadmap/discussions/167
45. eclipse-zenoh roadmap / Discussions / GitHub, accessed July 9, 2025, https://github.com/eclipse-zenoh/roadmap/discussions
46. accessed January 1, 1970, https://zenoh.io/blog/
47. Rust is DO-178C Certifiable - Pictorus Blog, accessed July 9, 2025, https://blog.pictor.us/rust-is-do-178-certifiable/
48. Lynx DO-178C Off-the-Shelf Certification Solutions for Safety-Critical Systems, accessed July 9, 2025, [https://www.lynx.com/hubfs/Data%20sheets/D0-178C%20Certification%20Datasheet.pdf](https://www.lynx.com/hubfs/Data sheets/D0-178C Certification Datasheet.pdf)
49. Developing Safety Critical Software A Practical For Aviation Software And Do 178c Compliance, accessed July 9, 2025, https://stat.somervillema.gov/HomePages/uploaded-files/4020073/Developing_Safety_Critical_Software_A_Practical_For_Aviation_Software_And_Do_178c_Compliance.pdf
50. eclipse-zenoh/roadmap - GitHub, accessed July 9, 2025, https://github.com/eclipse-zenoh/roadmap
51. 2021 / eclipse-zenoh/zenoh Wiki - GitHub, accessed July 9, 2025, https://github.com/eclipse-zenoh/zenoh/wiki/2021
52. Zenoh-Pico performance improvements, accessed July 9, 2025, https://zenoh.io/blog/2025-04-09-zenoh-pico-performance/
53. Zenoh-Flow 0.6.0-rc: Getting Started, accessed July 9, 2025, https://zenoh.io/blog/2024-01-31-zenoh-flow-getting-started/
54. Zenoh 1.0.0 "Firesong" is ready to rock!, accessed July 9, 2025, https://zenoh.io/blog/2024-10-21-zenoh-firesong/



