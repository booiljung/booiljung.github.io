[ROS2 RMW](./index.md)
# ROS 2 토픽 설계 원칙



Robot Operating System 2 (ROS 2)로 구축된 모든 로봇 시스템의 근간에는 데이터가 있다. 센서에서 인식한 주변 환경 정보, 로봇의 현재 상태, 그리고 목표를 달성하기 위한 제어 명령에 이르기까지, 이러한 데이터의 원활한 흐름 없이는 어떠한 지능적 동작도 불가능하다. ROS 2 아키텍처에서 이 데이터 흐름을 담당하는 가장 핵심적인 통신 메커니즘이 바로 **토픽(Topic)**이다.1 토픽은 분산된 계산 단위인 노드(Node)들이 서로 정보를 교환하는 비동기식 발행-구독(Publish-Subscribe) 방식의 통신 채널이다.3 이는 로봇 시스템을 구성하는 수많은 소프트웨어 모듈들을 연결하는 신경망과 같다. 따라서 토픽을 어떻게 설계하는가는 단순히 데이터 전달 경로를 설정하는 문제를 넘어, 전체 시스템의 모듈성, 성능, 확장성, 그리고 유지보수 용이성을 결정짓는 가장 중요한 아키텍처 설계 행위 중 하나이다.


ROS 2의 토픽 설계를 논하기에 앞서, ROS 1과의 근본적인 차이점을 이해하는 것이 필수적이다. ROS 1은 중앙 집중형 `roscore`(ROS Master)에 의존하여 노드 간의 연결 정보를 관리했다.4 통신은 TCPROS 및 UDPROS라는 자체 프로토콜을 기반으로 이루어졌다. 이러한 구조는 사용이 간편했지만, 마스터가 시스템 전체의 단일 장애점(Single Point of Failure)이 되는 본질적인 한계를 가졌다.4

ROS 2는 이러한 한계를 극복하기 위해 아키텍처를 완전히 재설계했다. 가장 큰 변화는 산업 표준 미들웨어인 DDS(Data Distribution Service)를 통신 계층의 기본으로 채택한 것이다.5 DDS의 도입으로 ROS 2는 중앙 마스터 없이 각 노드가 네트워크상에서 서로를 자동으로 탐색(discovery)하는 탈중앙화된 P2P(Peer-to-Peer) 통신 아키텍처를 갖추게 되었다.7 이 구조적 변화는 토픽 설계에 지대한 영향을 미쳤다. 개발자는 이제 DDS가 제공하는 강력한 서비스 품질(Quality of Service, QoS) 정책을 통해 데이터 통신의 신뢰성, 실시간성, 내구성 등을 토픽 단위로 세밀하게 제어할 수 있게 되었다.9 이는 ROS 1에서는 불가능했던 수준의 통신 제어 능력으로, ROS 2 토픽 설계의 핵심 고려사항이 되었다.


결론적으로, 성공적인 ROS 2 토픽 설계는 다음 세 가지 핵심 요소를 중심으로 이루어져야 한다. 첫째, **명확성(Clarity)**이다. 잘 정의된 네이밍 컨벤션과 의미론적으로 명확한 메시지 구조는 시스템의 데이터 흐름을 직관적으로 이해할 수 있게 하여 개발 및 디버깅 효율을 극대화한다. 둘째, **효율성(Efficiency)**이다. 데이터의 특성에 맞는 적절한 메시지 타입과 QoS 정책을 선택하고, 토픽의 세분화(granularity)를 최적화함으로써 불필요한 네트워크 대역폭과 CPU 자원 낭비를 막아야 한다. 셋째, **확장성(Scalability)**이다. 네임스페이스를 활용한 모듈화와 인터페이스 패키지의 분리를 통해, 새로운 기능이 추가되거나 시스템의 규모가 커지더라도 기존 아키텍처를 해치지 않고 유연하게 확장할 수 있는 구조를 만들어야 한다. 본 보고서는 이러한 핵심 요소들을 바탕으로 ROS 2 토픽 설계의 모든 측면을 심층적으로 분석하고, 실질적인 가이드라인을 제공하는 것을 목표로 한다.



ROS 2 통신의 가장 기본이 되는 패러다임은 발행-구독 모델이다. 이 모델에서 데이터의 생산자인 **발행자(Publisher)**와 소비자인 **구독자(Subscriber)**는 '토픽'이라는 이름의 매개체를 통해 통신한다.3 발행자는 특정 토픽으로 메시지를 '발행'하고, 해당 토픽에 관심 있는 모든 구독자는 그 메시지를 '구독'하여 수신한다. 이 모델의 핵심은 발행자와 구독자 간의 **분리(Decoupling)**에 있다.12 발행자는 어떤 구독자가 자신의 메시지를 듣고 있는지 알 필요가 없으며, 구독자 또한 어떤 발행자가 메시지를 보내는지 알 필요가 없다. 이러한 구조는 다수의 발행자가 하나의 토픽에 발행하고, 다수의 구독자가 동시에 구독하는 다대다(many-to-many) 통신을 자연스럽게 지원한다.12

이러한 분리 구조는 시스템에 엄청난 유연성을 부여한다. 노드들은 동적으로 시스템에 추가되거나 제거될 수 있으며, 이 과정에서 다른 노드들의 동작에 영향을 주지 않는다.3 예를 들어, 시스템의 특정 토픽에서 오가는 데이터를 기록하고 싶을 때 `ros2 bag record` 명령어를 사용하면, 이 명령어는 해당 토픽에 새로운 구독자로 참여하여 데이터를 기록한다.3 기존에 통신하던 발행자와 구독자들은 이 새로운 구독자의 존재를 전혀 인지하지 못하며, 데이터 흐름에도 아무런 방해가 없다. 이는 시스템의 상태를 실시간으로 검사(introspection)하고 디버깅하는 작업을 매우 용이하게 만든다.


ROS 2 토픽의 본질은 세 가지 핵심 특성으로 요약할 수 있다.

첫째, **익명성(Anonymous)**이다. 앞서 언급했듯이, 구독자는 메시지를 수신할 때 일반적으로 어떤 발행 노드가 그 메시지를 보냈는지 알거나 신경 쓸 필요가 없다.3 이 익명성은 시스템의 모듈성을 극대화한다. 예를 들어, 시뮬레이션 환경에서 가상의 센서 데이터를 발행하던 노드를 실제 하드웨어 드라이버 노드로 교체하더라도, 해당 토픽을 구독하던 다른 노드들은 아무런 코드 수정 없이 동일하게 동작할 수 있다. 그러나 이러한 익명성은 복잡한 시스템에서 데이터 흐름을 추적하기 어렵게 만드는 요인이 되기도 한다. 수십 개의 노드가 얽힌 시스템에서 특정 토픽에 잘못된 데이터가 발행될 경우, 그 근원지를 찾는 것은 `rqt_graph`와 같은 시각화 도구나 명확한 아키텍처 문서 없이는 매우 어려운 작업이 될 수 있다. 즉, 런타임의 분리는 설계 및 디버깅 단계에서 시스템 전체 그래프에 대한 깊은 이해를 요구하는 대가를 수반한다.

둘째, **강력한 타입(Strongly-typed)**이다. 모든 ROS 2 토픽은 반드시 특정 메시지 타입과 연결된다.3 예를 들어 `/cmd_vel` 토픽은 `geometry_msgs/msg/Twist` 타입의 메시지만을 전달할 수 있다. 발행자와 구독자는 반드시 동일한 메시지 타입을 사용해야만 통신이 가능하다.1 이 제약은 두 가지 중요한 이점을 제공한다. 하나는 컴파일 시점에 타입 불일치 오류를 잡아낼 수 있어 시스템의 안정성을 높인다는 점이다. 다른 하나는 통신에 사용되는 데이터 구조에 대한 명확한 '계약'을 정의함으로써, 서로 다른 개발자나 팀이 만든 노드 간의 통합을 용이하게 한다는 것이다.3

셋째, **비동기성(Asynchronous)**이다. 발행자는 구독자의 상태와 무관하게 메시지를 발행한다. 구독자가 메시지를 처리할 준비가 되었는지, 혹은 이전 메시지를 모두 처리했는지 기다리지 않는다. 이는 센서 데이터와 같이 지속적으로 생성되는 데이터 스트림을 처리하는 데 매우 적합하며, 시스템의 특정 부분이 느려지더라도 전체 시스템이 멈추는(blocking) 현상을 방지한다. 데이터의 손실 가능성은 뒤에서 다룰 QoS 정책을 통해 제어된다.


ROS 2 시스템은 전체적으로 **ROS 그래프(ROS Graph)**라는 네트워크로 표현될 수 있다. 이 그래프의 정점(vertex)은 계산을 수행하는 주체인 **노드(Node)**이며, 간선(edge)은 노드 간의 통신 경로인 토픽, 서비스, 액션을 나타낸다.7

ROS 2는 이 그래프를 구성하기 위해 DDS(Data Distribution Service) 미들웨어를 사용한다. 노드가 시작되면, 동일한 `ROS_DOMAIN_ID` 환경 변수로 설정된 네트워크 도메인 내의 다른 노드들에게 자신의 존재를 알린다.7 이 광고(advertisement)에는 해당 노드가 어떤 토픽을 발행하고 구독하는지, 그리고 어떤 QoS 정책을 사용하는지에 대한 정보가 포함된다. 다른 노드들은 이 광고를 수신하고, 만약 서로 호환되는 토픽과 QoS 정책을 가지고 있다면 직접적인 통신 채널을 설정한다.7 이 모든 과정은 중앙 서버 없이 P2P 방식으로 자동으로 이루어지며, 이를 **자동 탐색(Discovery)** 메커니즘이라 부른다.7

ROS 1의 중앙 마스터 방식에서 ROS 2의 탈중앙화된 자동 탐색 방식으로의 전환은 시스템의 견고성을 높였지만, 동시에 새로운 유형의 잠재적 문제점을 야기했다. ROS 1에서는 노드가 마스터에만 연결되면 다른 노드를 찾는 데 큰 문제가 없었지만, ROS 2에서는 DDS의 탐색 메커니즘이 의존하는 UDP 멀티캐스트(multicast)와 같은 저수준 네트워크 프로토콜에 대한 이해가 중요해졌다. 잘못된 네트워크 인터페이스 설정, 방화벽의 멀티캐스트 차단, Docker 컨테이너의 네트워크 구성 오류, 혹은 단순한 `ROS_DOMAIN_ID` 불일치 등은 노드들이 서로를 "조용히" 발견하지 못하는 문제로 이어질 수 있다. 이는 명시적인 에러 메시지 없이 통신이 두절되는 현상을 낳기 때문에, ROS 2 개발자는 로봇 소프트웨어뿐만 아니라 네트워크 수준의 디버깅 능력 또한 갖추어야 함을 시사한다.



토픽의 이름은 해당 토픽을 통해 전달되는 데이터의 의미를 전달하는 첫 번째 수단이다. 명확하고 일관된 네이밍 규칙은 시스템의 가독성과 유지보수성을 크게 향상시킨다.


ROS 2 토픽 이름은 엄격한 규칙을 따른다. 유효한 이름은 알파벳(`a-z`, `A-Z`), 숫자(`0-9`), 언더스코어(`_`), 그리고 네임스페이스를 구분하는 포워드 슬래시(`/`)로만 구성될 수 있다.14 이름은 알파벳, `~`, 또는 `/`로 시작해야 하며, 숫자로 시작할 수 없다. 또한, 이름이 `/`로 끝나거나, `//`처럼 연속된 슬래시, 혹은 `__`처럼 연속된 언더스코어를 포함하는 것은 허용되지 않는다.14

ROS 1과 구별되는 중요한 규칙 중 하나는 프라이빗 네임스페이스를 나타내는 물결표(`~`)의 사용법이다. ROS 2에서는 반드시 `~/` 형태로 슬래시와 함께 사용해야 한다. 예를 들어 `~/my_topic`은 유효하지만, ROS 1 스타일의 `~my_topic`은 유효하지 않다.14 이는 파일 시스템 경로와의 혼동을 피하고 일관성을 유지하기 위한 설계 결정이다.


네임스페이스는 복잡한 시스템에서 토픽 이름을 체계적으로 구성하고 충돌을 방지하는 핵심적인 도구이다.

- **절대 경로 (Absolute Path)**: 이름이 슬래시(`/`)로 시작하는 경우, 이는 절대 경로로 간주된다. 절대 경로 토픽은 노드가 어떤 네임스페이스에서 실행되든 관계없이 항상 동일한 전체 이름을 갖는다.14

  `/tf`, `/tf_static`, `/rosout`과 같이 시스템 전반에 걸쳐 유일하고 핵심적인 역할을 하는 토픽에 주로 사용된다.

- **상대 경로 (Relative Path)**: 이름이 슬래시(`/`)로 시작하지 않는 경우, 이는 상대 경로로 간주되며 노드가 실행되는 네임스페이스에 종속된다.14 예를 들어, `camera_driver` 노드가 `image_raw`라는 상대 경로 토픽을 발행하고, 이 노드가 `/robot1/front_camera`라는 네임스페이스에서 실행된다면, 최종 토픽의 전체 이름은 `/robot1/front_camera/image_raw`가 된다.15 이 방식은 로봇의 특정 부분(예: 카메라, 팔)과 관련된 노드들을 하나의 그룹으로 묶어 재사용성을 극대화하는 데 매우 유용하다. 동일한 노드들을 `/robot2/front_camera` 네임스페이스에서 실행하면 토픽 이름이 자동으로 구분되어 다중 로봇 시스템을 쉽게 구성할 수 있다.

- **프라이빗 경로 (Private Path)**: 이름이 `~/`로 시작하는 경우, 이는 해당 노드에 고유한 프라이빗 경로로 취급된다. `~/`는 런타임에 `/<node_namespace>/<node_name>/`으로 확장된다.14 이 경로는 주로 해당 노드의 내부 동작을 디버깅하기 위한 토픽이나, 외부에서 직접 접근할 필요가 없는 파라미터 관련 토픽에 사용된다.


대규모 오픈소스 프로젝트인 Autoware는 토픽 네이밍에 대한 매우 효과적인 관례를 제시한다. Autoware에서는 각 노드의 프라이빗 네임스페이스 내에서, 구독하는 토픽에는 `input/` 접두사를, 발행하는 토픽에는 `output/` 접두사를, 그리고 시각화나 디버깅을 위한 토픽에는 `debug/` 접두사를 붙인다.16 예를 들어, 포인트 클라우드 데이터를 입력받아 필터링하여 출력하는 노드가 있다면, 입력 토픽은 `~/input/points_original`, 출력 토픽은 `~/output/points_filtered`로 명명된다.

이 규칙은 런치 파일(launch file)에서 노드들을 연결하고 토픽을 리매핑(remapping)할 때 데이터의 흐름을 매우 명확하게 보여준다. `input`과 `output`이라는 이름만으로도 각 토픽의 역할을 직관적으로 파악할 수 있어, 시스템 전체의 아키텍처를 이해하고 디버깅하는 데 큰 도움이 된다.

| 규칙 (Rule)          | 설명 (Description)                                           | 유효한 예시 (Valid Example) | 유효하지 않은 예시 (Invalid Example) | 근거 (Rationale)                                             |
| -------------------- | ------------------------------------------------------------ | --------------------------- | ------------------------------------ | ------------------------------------------------------------ |
| 숫자 시작 금지       | 토픽 이름의 각 토큰(슬래시로 구분된 부분)은 숫자로 시작할 수 없다. | `camera1/image_raw`         | `/1camera/image_raw`                 | 파싱의 모호함을 피하고 변수명 규칙과 일관성을 유지하기 위함. |
| 연속 슬래시 금지     | 토픽 이름에 `//`와 같이 연속된 슬래시를 포함할 수 없다.      | `/foo/bar`                  | `/foo//bar`                          | 이름 결합 시 발생할 수 있는 우발적인 `//`를 방지하고 정규화 필요성을 제거함. |
| 슬래시로 종료 금지   | 토픽 이름은 `/`로 끝날 수 없다.                              | `/foo/bar`                  | `/foo/bar/`                          | 모든 이름이 베이스 이름을 가지도록 보장하여 일관성을 유지함. |
| `~` 사용법           | 프라이빗 네임스페이스 지정자 `~`는 반드시 `~/` 형태로 사용해야 한다. | `~/status`                  | `~status`                            | 파일 시스템 경로 해석과의 일관성을 유지하고 모호함을 제거함. |
| 연속 언더스코어 금지 | 토픽 이름에 `__`와 같이 연속된 언더스코어를 포함할 수 없다.  | `laser_scan`                | `laser__scan`                        | 이름의 가독성을 높이고 일관된 작명 관례를 장려함.            |
| 숨겨진 토픽          | 토큰이 언더스코어(`_`)로 시작하면 '숨겨진' 토픽으로 간주된다. | `/_internal/status`         | -                                    | `ros2 topic list`와 같은 도구에서 기본적으로 표시되지 않아, 내부용 또는 디버깅용 토픽을 분리하는 데 사용됨. |


토픽의 이름만큼이나 중요한 것이 그 토픽을 통해 전달되는 메시지의 구조를 설계하는 것이다. 잘 설계된 메시지는 데이터의 의미를 명확하게 전달하고 시스템의 효율성을 높인다.


새로운 커스텀 메시지를 만들기 전에, 가장 먼저 해야 할 일은 ROS 2 커뮤니티에서 이미 표준으로 제공하는 메시지가 있는지 확인하는 것이다. `common_interfaces`라는 메타 패키지에는 `sensor_msgs`(센서 데이터), `geometry_msgs`(기하학적 데이터), `nav_msgs`(내비게이션 데이터) 등 다양한 표준 메시지 패키지들이 포함되어 있다.17 예를 들어, 2D 라이다 데이터를 다룬다면 직접 메시지를 정의할 것이 아니라 `sensor_msgs/msg/LaserScan`을 사용해야 한다. 표준 메시지를 사용하면 Rviz, `ros2 bag` 등 커뮤니티의 수많은 도구 및 다른 로봇 시스템과의 호환성을 즉시 확보할 수 있다.19

반면, ROS 1에서 프로토타이핑 용도로 널리 사용되었던 `std_msgs` 패키지(ROS 2에서는 `example_interfaces`가 일부 역할을 대체함)의 `String`, `Int32`와 같은 제네릭 타입 메시지는 실제 애플리케이션에서 사용하는 것을 적극적으로 지양해야 한다.17 이 메시지들의 필드 이름은 모두 `data`로 동일하여, 그 안에 담긴 데이터가 무엇을 의미하는지에 대한 의미론적 정보(semantic meaning)를 전혀 제공하지 못한다.20 예를 들어, `std_msgs/String` 타입의 `/robot_command`라는 토픽이 있다고 가정해보자. 이 토픽에 "start"라는 문자열이 전달될 수도 있고, "stop"이라는 문자열이 전달될 수도 있다. 구독하는 노드는 수신한 문자열의 내용을 직접 파싱하고 비교해야만 그 의미를 알 수 있다. 이는 런타임 오류의 가능성을 높이고 코드의 가독성을 심각하게 저해한다. 이는 단기적인 개발 편의성을 위해 장기적인 시스템의 명확성과 안정성을 희생하는 행위이며, ROS 2의 설계 철학은 이러한 관행을 지양하고 명시적이고 자기 서술적인 인터페이스 설계를 장려하는 방향으로 성숙했다.


표준 메시지가 요구사항에 맞지 않아 커스텀 메시지를 만들어야 할 경우, 메시지, 서비스, 액션 정의만을 위한 별도의 패키지를 생성하는 것이 ROS 2의 모범 사례(best practice)이다.17 이 패키지의 이름은 관례적으로 `<project_name>_interfaces`와 같이 `_interfaces` 접미사를 붙인다.17

이러한 인터페이스 패키지 분리 원칙은 단순한 스타일 가이드를 넘어, 대규모 시스템의 아키텍처를 건전하게 유지하기 위한 핵심 전략이다. ROS 1에서는 종종 기능을 수행하는 노드와 그 노드가 사용하는 메시지 정의가 같은 패키지 안에 공존했다.21 이 경우, 패키지 A가 패키지 B의 메시지를 필요로 하고, 동시에 패키지 B가 패키지 A의 유틸리티 라이브러리를 필요로 하는 상황이 발생하면 **순환 의존성(circular dependency)**이 발생하여 빌드 시스템을 망가뜨릴 수 있다. 인터페이스를 별도의 패키지로 분리하면, 모든 기능 패키지들이 인터페이스 패키지에 의존하지만 인터페이스 패키지는 어떠한 기능 패키지에도 의존하지 않는 단방향 의존성 그래프(Directed Acyclic Graph, DAG)가 형성된다. 이는 패키지 수준에서 계층적인 아키텍처를 강제하여, 여러 팀이 협업하는 대규모 프로젝트에서 아키텍처의 부패를 방지하는 매우 중요한 역할을 한다.

인터페이스 패키지는 `msg/`, `srv/`, `action/` 디렉토리를 포함하며, `CMakeLists.txt`와 `package.xml` 파일에 `rosidl_default_generators`와 관련된 의존성을 명시하여 빌드 시스템이 이 파일들로부터 각 언어에 맞는 코드를 생성하도록 설정해야 한다.17


메시지 파일 자체를 작성할 때도 명확한 규칙을 따라야 한다.

- **파일 및 필드명 규칙**: 메시지 파일의 이름은 `PascalCase`(예: `MotorStatus.msg`)를 사용한다.17 파일 이름에 `Msg`나 `Srv`와 같은 접미사를 붙이는 것은 불필요한 중복이므로 피해야 한다.17 메시지 내부의 필드 이름은 `snake_case`(예: `motor_temperature`)를 사용한다.23
- **의미론적 명확성**: 필드의 이름은 그 데이터가 나타내는 물리적 의미를 명확하게 전달해야 한다. 예를 들어, 단순히 `velocity`라고 명명하기보다는 `linear_velocity_mps` (초당 미터) 또는 `angular_velocity_rps` (초당 라디안)와 같이 단위를 이름에 포함시키는 것이 좋다. 이것이 어렵다면, 메시지 파일 내에 주석으로 단위를 명시해야 한다. 예를 들어, `sensor_msgs/msg/Imu` 메시지는 `angular_velocity` 필드가 `rad/s` 단위임을 명확히 하고 있다.3 이는 메시지를 사용하는 모든 개발자가 데이터의 의미를 동일하게 해석하도록 보장하는 중요한 약속이다.
- **데이터 타입 선택**: 고정 크기 배열과 가변 크기 배열, 그리고 기본 데이터 타입(primitive types)을 적절히 선택해야 한다. 예를 들어, 로봇 팔의 관절 수가 6개로 고정되어 있다면 `float64 joint_positions`와 같이 고정 크기 배열을 사용하는 것이 메모리 할당 측면에서 더 효율적이다. 반면, 라이다 스캔 데이터처럼 측정값의 개수가 변할 수 있는 경우에는 `float32 ranges`와 같이 가변 크기 배열을 사용해야 한다.24


ROS 2에서 토픽 설계를 할 때 가장 중요하고 강력한 기능 중 하나는 서비스 품질(QoS) 정책이다. QoS는 DDS 미들웨어의 핵심 기능으로, 개발자가 데이터 통신의 특성을 매우 세밀하게 조정할 수 있게 해준다. 올바른 QoS 설정은 시스템의 신뢰성, 효율성, 실시간성을 보장하는 데 결정적인 역할을 한다.


가장 기본적이고 자주 사용되는 네 가지 QoS 정책은 다음과 같다.

- **신뢰성 (Reliability)**: 메시지 전달을 보장할지 여부를 결정한다.9
  - `RELIABLE`: TCP와 유사하게, 발행된 메시지가 구독자에게 반드시 전달되는 것을 보장한다. 이를 위해 내부적으로 확인 응답(acknowledgement) 및 재전송 메커니즘을 사용한다. 로봇의 이동을 제어하는 `/cmd_vel` 토픽과 같이 데이터 손실이 치명적인 경우에 반드시 사용해야 한다.
  - `BEST_EFFORT`: UDP와 유사하게, 메시지 전달을 보장하지 않는다. 네트워크 상황에 따라 메시지가 유실될 수 있다. 고주파로 발행되는 센서 데이터(예: `/scan`, `/image_raw`)와 같이, 일부 데이터가 손실되더라도 최신 데이터를 빠르게 수신하는 것이 더 중요한 경우에 적합하다. `RELIABLE`에 비해 오버헤드가 적다.9
- **내구성 (Durability)**: 발행자가 메시지를 얼마나 오랫동안 보관할지를 결정한다.9
  - `VOLATILE`: 발행자는 메시지를 보관하지 않는다. 구독자는 자신이 구독을 시작한 시점 이후에 발행된 메시지만 수신할 수 있다. 대부분의 동적인 데이터 스트림에 사용되는 기본 설정이다.
  - `TRANSIENT_LOCAL`: 발행자는 지정된 수만큼의 최신 메시지를 로컬에 저장하고 있다가, 늦게 참여하는(late-joining) 구독자에게 전달해준다. 시스템의 정적인 설정 정보나 상태를 나타내는 토픽에 매우 유용하다. 예를 들어, 한 번 발행된 지도를 `/map` 토픽에 `TRANSIENT_LOCAL`로 발행하면, 나중에 실행되는 내비게이션 관련 노드들이 실행되자마자 이 지도를 즉시 수신할 수 있다.27
- **이력 (History) 및 깊이 (Depth)**: 구독자가 메시지를 처리하는 속도보다 발행자가 메시지를 발행하는 속도가 더 빠를 때 메시지를 어떻게 처리할지 결정한다.25
  - `KEEP_LAST`: `Depth` 옵션으로 지정된 크기의 큐를 유지하며, 최신 메시지만큼만 저장한다. 큐가 가득 차면 가장 오래된 메시지부터 버린다. 대부분의 경우 이 설정이 권장된다.
  - `KEEP_ALL`: 미들웨어의 리소스 한계에 도달할 때까지 모든 메시지를 큐에 저장한다. 메모리 사용량이 예측 불가능하게 증가할 수 있어 신중하게 사용해야 한다.


ROS 2는 보다 엄격한 실시간 요구사항을 만족시키기 위한 고급 QoS 정책도 제공한다.

- **기한 (Deadline)**: 메시지가 발행되거나 수신되어야 하는 최대 시간 간격을 설정한다.25 발행자는 설정된 

  `Deadline` 주기 내에 반드시 메시지를 발행해야 하며, 구독자는 이 주기 내에 메시지를 수신할 것을 기대한다. 만약 이 기한이 지켜지지 않으면 QoS 이벤트가 발생하여, 시스템이 주기적인 데이터 스트림의 단절을 감지하고 대응할 수 있게 한다.

- **수명 (Lifespan)**: 메시지의 유효 기간을 설정한다.25 발행된 메시지는 

  `Lifespan`으로 설정된 시간이 지나면 만료되어 더 이상 구독자에게 전달되지 않는다. 이는 오래된 데이터가 오히려 시스템에 위험을 초래할 수 있는 제어 시스템에서 유용하게 사용될 수 있다.

- **활성 (Liveliness)**: 발행 노드가 여전히 "살아있는지"를 판단하는 메커니즘을 정의한다.25 발행자는 주기적으로 자신의 활성 상태를 네트워크에 알리며, 구독자는 이 신호가 특정 시간 동안 수신되지 않으면 해당 발행자에 문제가 발생했다고 판단할 수 있다. 이는 시스템의 특정 구성 요소의 장애를 감지하는 데 사용된다.


이처럼 다양한 QoS 정책을 매번 직접 조합하는 것은 복잡하고 오류가 발생하기 쉽다. 이를 돕기 위해 ROS 2는 일반적인 사용 사례에 맞춰 미리 정의된 QoS 프로파일을 제공한다.9

- **Sensor Data Profile**: 고주파 센서 데이터에 최적화되어 있다. `BEST_EFFORT` 신뢰성, `VOLATILE` 내구성, 그리고 작은 `Depth` 값을 사용하여 최신 데이터를 최소한의 오버헤드로 빠르게 전달하는 데 중점을 둔다.9
- **Services Profile**: 요청-응답 통신에 사용된다. `RELIABLE` 신뢰성과 `VOLATILE` 내구성을 사용하여 요청과 응답이 확실히 전달되도록 보장한다. 특히 `VOLATILE` 설정은 재시작된 서비스 서버가 과거의 오래된 요청을 수신하는 것을 방지하는 데 중요하다.9
- **Parameters Profile**: 서비스에 기반하지만, 요청이 유실될 가능성을 최소화하기 위해 매우 큰 `Depth` 값을 사용한다. 이는 파라미터 클라이언트와 서버 간의 통신이 일시적으로 원활하지 않더라도 요청이 보존되도록 한다.9


발행자와 구독자 간의 통신이 성립되기 위해서는 토픽 이름과 메시지 타입이 일치하는 것만으로는 충분하지 않다. 두 노드의 QoS 정책 또한 반드시 **호환(compatible)**되어야 한다. 이 호환성은 "요청 대 제공(Request vs. Offered)" 모델을 따른다.25 즉, 구독자가 '요청'하는 QoS 수준이 발행자가 '제공'할 수 있는 QoS 수준보다 더 엄격해서는 안 된다.

예를 들어, 한 구독자가 `RELIABLE` 통신을 요청하는데 발행자가 `BEST_EFFORT`만 제공한다면, 발행자는 구독자의 요구사항을 만족시킬 수 없으므로 둘 사이의 연결은 성립되지 않는다.26 반대로, 발행자가 `RELIABLE`을 제공하고 구독자가 `BEST_EFFORT`를 요청하는 경우는 호환 가능하다. `Durability` 정책의 경우, `TRANSIENT_LOCAL`을 요청하는 구독자는 `VOLATILE`만 제공하는 발행자와 연결될 수 없다.

이 QoS 호환성 문제는 ROS 2에서 발생하는 "조용한 실패(silent failure)"의 주된 원인 중 하나이다. 노드들은 정상적으로 실행되고 있고 `ros2 topic list`에도 토픽이 나타나지만, QoS 불일치로 인해 실제 데이터는 전혀 전달되지 않는 상황이 발생할 수 있다. 이는 ROS 1에서는 거의 겪어보지 못한 유형의 문제로, 통신 문제를 디버깅할 때 `ros2 topic info -v` 명령어를 통해 각 발행자와 구독자의 QoS 설정을 확인하는 것이 ROS 2에서는 필수적인 첫 단계가 되었다.29

또한, `TRANSIENT_LOCAL` 내구성은 매우 유용하지만, 신중하게 사용하지 않으면 디버깅하기 어려운 "오래된 데이터(stale data)" 문제를 일으킬 수 있다. 예를 들어, 로봇의 초기 상태를 `TRANSIENT_LOCAL` 토픽으로 발행하는 노드가 비정상적으로 종료되었다가 재시작되었다고 가정해보자. 새로 네트워크에 참여하는 구독자는 재시작된 노드가 새로운 상태를 발행하기 전에, 미들웨어에 의해 유지되고 있던 *과거의 오래된* 초기 상태 값을 먼저 수신할 수 있다. 이는 시스템이 잘못된 상태로 초기화되는 심각한 문제로 이어질 수 있다. 따라서 `TRANSIENT_LOCAL`은 지도와 같이 내용이 거의 변하지 않는 정적인 데이터에 주로 사용하고, 동적인 상태 정보에는 `VOLATILE`을 사용하는 것이 일반적으로 더 안전한 설계이다.

| 정책 (Policy)   | 설정 (Setting)    | 설명 (Description)                                  | 권장 사용 사례 (Recommended Use Case)                    |
| --------------- | ----------------- | --------------------------------------------------- | -------------------------------------------------------- |
| **Reliability** | `RELIABLE`        | 메시지 전달을 보장함 (TCP와 유사).                  | 제어 명령 (`/cmd_vel`), 목표 지점 (`/goal_pose`)         |
|                 | `BEST_EFFORT`     | 메시지 전달을 보장하지 않음 (UDP와 유사).           | 고주파 센서 데이터 (`/scan`, `/image_raw`)               |
| **Durability**  | `VOLATILE`        | 늦게 참여한 구독자에게 과거 메시지를 전달하지 않음. | 대부분의 동적 데이터 스트림                              |
|                 | `TRANSIENT_LOCAL` | 늦게 참여한 구독자에게 최신 메시지를 전달함.        | 지도 (`/map`), 로봇 설명 (`/robot_description`)          |
| **History**     | `KEEP_LAST`       | `Depth` 크기만큼의 최신 메시지만 큐에 저장함.       | 대부분의 사용 사례                                       |
|                 | `KEEP_ALL`        | 리소스 한계까지 모든 메시지를 큐에 저장함.          | 모든 메시지 기록이 반드시 필요한 경우 (신중한 사용 필요) |
| **Depth**       | `Integer`         | `KEEP_LAST` 사용 시 큐의 크기를 지정함.             | 데이터의 중요도와 발행 빈도에 따라 조절                  |



토픽을 설계할 때 가장 중요한 결정 중 하나는 '얼마나 잘게 나눌 것인가'하는 세분화(granularity) 문제이다. 이는 소프트웨어 공학의 단일 책임 원칙(Single Responsibility Principle)과 맞닿아 있다. 하나의 토픽은 논리적으로 응집된 단일 종류의 정보를 전달해야 한다. 예를 들어, 로봇의 모든 상태 정보를 담은 거대한 `RobotAllState`라는 단일 메시지를 만들어 하나의 토픽으로 발행하는 것은 좋지 않은 설계이다. 대신, `/odometry`(위치 및 속도 정보), `/joint_states`(관절 상태), `/battery_state`(배터리 정보), `/diagnostics`(진단 정보) 등 논리적 단위로 토픽을 분리하는 것이 바람직하다.

이렇게 토픽을 세분화하면 여러 장점이 있다. 첫째, 시스템의 모듈성이 향상된다. 배터리 모니터링 노드는 `/battery_state` 토픽만 구독하면 되므로, 거대한 `RobotAllState` 메시지 전체를 수신하여 불필요한 데이터를 파싱하는 낭비를 피할 수 있다. 이는 네트워크 대역폭과 CPU 사용량을 절약하는 효과를 가져온다. 둘째, 각 데이터의 발행 주기를 독립적으로 관리할 수 있다. `/joint_states`는 100Hz로 발행해야 하지만, `/battery_state`는 1Hz로도 충분할 수 있다.

하지만 과도한 세분화는 오히려 시스템의 복잡도를 높일 수 있다. 토픽의 수가 지나치게 많아지면 전체 데이터 흐름을 파악하기 어려워지고, 서로 다른 토픽으로 분리된 데이터 간의 시간 동기화 문제가 발생할 수 있다. 예를 들어, 특정 시점의 이미지 데이터와 그 이미지가 촬영될 당시의 로봇 팔 관절 각도를 함께 사용해야 하는 경우, `/camera/image_raw` 토픽과 `/joint_states` 토픽의 메시지들을 시간 기준으로 정확히 매칭해야 한다. 이를 위해 ROS 메시지의 `std_msgs/Header`에 포함된 `stamp` 필드를 활용한 시간 동기화 기법이 매우 중요해진다.


ROS 2는 **노드 컴포지션(Node Composition)** 또는 **컴포넌트(Components)**라는 강력한 기능을 제공한다. 이는 여러 개의 논리적인 노드를 하나의 운영체제 프로세스 내에서 실행할 수 있게 해주는 기능이다.8 ROS 1의 `nodelet`과 유사하지만, ROS 2에서는 핵심 기능으로 통합되어 훨씬 사용하기 편리해졌다.

컴포지션을 사용하는 주된 이유는 성능 향상이다. 각 노드를 별도의 프로세스로 실행하면, 노드 간 통신은 운영체제의 네트워킹 스택을 거쳐야 하므로 상당한 오버헤드가 발생한다. 반면, 여러 노드를 하나의 프로세스에 배치하면, 이들 간의 통신은 **Intra-Process Communication (IPC)**을 통해 최적화된다.30 IPC는 메시지를 직렬화(serialization)하고 네트워크를 통해 전송하는 대신, 메시지가 담긴 메모리의 포인터만을 주고받는 **제로카피(zero-copy)** 방식으로 동작한다. 이는 통신 지연 시간(latency)을 극적으로 줄이고 CPU 사용량을 감소시켜, 특히 고대역폭 데이터(예: 카메라 이미지, 포인트 클라우드)를 처리하는 파이프라인에서 엄청난 성능 이점을 제공한다.30

그러나 컴포지션은 장점만 있는 것이 아니다. 여러 노드가 하나의 프로세스를 공유하므로, 그중 하나의 노드에서 메모리 누수나 세그멘테이션 폴트(segmentation fault)와 같은 심각한 오류가 발생하면 프로세스 전체가 비정상적으로 종료될 수 있다. 이는 해당 프로세스에 포함된 모든 노드가 동시에 다운되는 단일 장애점(single point of failure)을 만드는 셈이다. 따라서 안정성이 충분히 검증된 노드들이나, 기능적으로 강하게 결합된 노드들을 함께 구성하는 것이 바람직하다.

토픽 세분화와 노드 컴포지션 사이에는 미묘한 상충 관계가 존재한다. 모듈성을 위해 토픽을 잘게 나누는 것이 좋지만, 만약 잘게 나뉜 토픽으로 통신하는 노드들이 서로 다른 프로세스에 배치된다면 IPC의 이점을 누릴 수 없다. 예를 들어, `/odom` 토픽을 발행하는 `odometry_publisher` 노드와 이를 구독하는 `localization` 노드를 같은 프로세스에 구성하여 IPC로 최적화했다고 가정해보자. 그런데 만약 다른 프로세스에 있는 `power_management` 노드 또한 로봇의 속도 기반 전력 소모 예측을 위해 `/odom` 토픽을 구독해야 한다면, `/odom` 토픽은 효율적인 IPC와 비효율적인 프로세스 간 통신(Inter-Process Communication) 양쪽으로 모두 전송되어야 한다. 이는 DDS 미들웨어에 추가적인 복잡성을 야기하고 IPC의 이점을 일부 상쇄시킬 수 있다. 따라서 최적의 컴포지션 전략은 단순히 '관련된' 노드를 묶는 것을 넘어, '높은 대역폭과 낮은 지연 시간 요구사항을 갖는' 노드들을 함께 묶는, 보다 정교한 아키텍처적 결정이 필요하다.



토픽이 사용하는 네트워크 대역폭은 메시지의 크기와 발행 빈도에 의해 결정된다. 여기서 '메시지 크기'란 메모리상의 데이터 구조 크기가 아닌, 네트워크로 전송되기 위해 **직렬화된(serialized)** 후의 크기를 의미한다. 직렬화 과정에서 데이터는 바이트 스트림으로 변환되며, 이 크기는 `ros2 topic bw`와 같은 명령줄 도구를 통해 측정할 수 있다.2

프로그래밍 방식으로는 `rclcpp::SerializedMessage` 클래스를 사용하여 메시지를 직접 직렬화하고 그 크기를 `size()` 메소드로 얻을 수 있다.31 하지만 이는 실제 직렬화를 수행한 후에야 알 수 있는 값이다. 메시지 타입이 컴파일 타임에 고정된 크기를 갖는지(`is_fixed_size`) 혹은 가변 크기인지(`is_bounded_size`는 최대 크기를 가짐)는 타입 특성(type traits)을 통해 확인할 수 있다.32 고해상도 비압축 이미지나 대규모 포인트 클라우드와 같이 잠재적으로 매우 큰 데이터를 토픽으로 전송할 때는, 예상 대역폭 요구사항을 신중하게 계산하고 필요하다면 압축과 같은 기법을 적용해야 한다.33

단순히 `메시지 크기 × 발행 빈도`로 대역폭을 계산하는 것은 종종 실제 사용량과 차이를 보인다. 실제 네트워크상에서 전송되는 데이터의 크기는 순수 메시지 데이터 외에 DDS/RTPS 프로토콜 헤더, 그리고 QoS 정책에 따른 추가적인 제어 패킷을 포함하기 때문이다. 예를 들어, `RELIABLE` QoS를 사용하면 구독자는 수신 확인을 위해 발행자에게 ACK/NACK 패킷을 보내고, 발행자는 활성 상태를 알리기 위해 하트비트(heartbeat) 패킷을 보낸다.34 이러한 양방향 제어 트래픽이 추가적인 대역폭을 소모한다. 따라서 실제 대역폭 B는 B=(Smsg+Shdr)×Rmsg+Bqos 와 같이 모델링할 수 있다. 여기서 $S_{msg}$는 메시지 크기, $S_{hdr}$는 헤더 크기, $R_{msg}$는 발행 빈도, 그리고 $B_{qos}$는 QoS 관련 제어 패킷이 사용하는 대역폭을 의미한다. 이것이 동일한 데이터를 전송하더라도 `BEST_EFFORT`가 `RELIABLE`보다 일반적으로 더 적은 대역폭을 사용하는 이유이다.


ROS 2는 토픽의 성능을 간단히 측정할 수 있는 명령줄 도구를 제공한다.

- `ros2 topic hz <topic_name>`: 지정된 토픽의 메시지가 수신되는 평균 빈도(Hz), 최소/최대 간격, 표준 편차 등을 출력한다.35 이는 발행자가 의도한 주기대로 메시지를 발행하고 있는지, 혹은 네트워크나 구독자의 처리 지연으로 인해 주기가 불안정해지지는 않는지 확인하는 데 매우 유용하다.
- `ros2 topic bw <topic_name>`: 지정된 토픽이 사용하고 있는 데이터 전송률, 즉 대역폭(예: KB/s)을 출력한다.2 평균 메시지 크기 정보도 함께 제공된다.

이 도구들을 사용할 때 반드시 기억해야 할 중요한 사실이 있다. 이 도구들은 내부적으로 새로운 구독자 노드를 생성하여 토픽을 구독하고, 그 구독자가 수신하는 메시지를 기반으로 통계를 계산한다.2 이는 두 가지 함의를 갖는다. 첫째, 측정 행위 자체가 네트워크 부하를 증가시킨다. 특히 고대역폭 토픽의 경우, `ros2 topic bw`를 실행하는 것만으로도 네트워크 전체 성능에 영향을 줄 수 있다.37 둘째, 측정된 값은 발행자의 실제 발행률이나 대역폭과 정확히 일치하지 않을 수 있다. 측정 도구가 실행되는 머신의 리소스 상태, 네트워크 혼잡도, 그리고 측정용 구독자의 QoS 설정 등 다양한 요인에 의해 영향을 받기 때문이다.2 따라서 이 도구들의 출력은 시스템 성능에 대한 유용한 지표이지만, 절대적인 값으로 맹신하기보다는 경향성을 파악하는 용도로 활용하는 것이 바람직하다.


ROS 2는 토픽 외에도 서비스(Services)와 액션(Actions)이라는 두 가지 주요 통신 메커니즘을 제공한다. 시스템을 설계할 때 주어진 문제에 가장 적합한 통신 패턴을 선택하는 것은 매우 중요하다.


- **토픽 (Topics)**: 연속적인 데이터 스트림을 위한 비동기, 단방향 통신이다.38 발행자는 구독자의 존재 여부나 상태에 관계없이 계속해서 데이터를 방송(broadcast)한다. 다수의 발행자와 다수의 구독자가 참여하는 다대다 통신에 적합하다.
- **서비스 (Services)**: 원격 프로시저 호출(Remote Procedure Call, RPC)을 위한 양방향 통신이다.40 클라이언트가 서버에 요청(request)을 보내면, 서버는 작업을 수행한 후 응답(response)을 반환한다. 통신은 기본적으로 일대일(하나의 서버에 여러 클라이언트가 요청 가능)이며, 요청과 응답이 명확하게 짝을 이룬다. ROS 2 서비스는 비동기 호출을 지원하지만, 개념적으로는 특정 작업에 대한 결과를 기다리는 동기적 상호작용에 가깝다.
- **액션 (Actions)**: 장시간 실행되는 작업을 위한 비동기, 양방향 통신이다.41 액션은 **목표(Goal)**, 주기적인 **피드백(Feedback)**, 그리고 최종 **결과(Result)**라는 세 부분으로 구성된다.43 클라이언트는 서버에 목표를 전송한 후 다른 작업을 계속할 수 있으며, 서버는 목표를 달성하는 동안 진행 상황을 피드백으로 알려준다. 또한, 클라이언트는 진행 중인 작업을 언제든지 **취소(preempt)**할 수 있다.


어떤 통신 패턴을 선택해야 할지는 전달하려는 데이터나 수행하려는 작업의 특성에 따라 결정된다.

- **토픽을 사용해야 할 때**:
  - 센서 데이터(라이다 스캔, 카메라 이미지, IMU 데이터 등)를 지속적으로 방송할 때.38
  - 로봇의 현재 상태(위치, 속도, 관절 각도 등)를 주기적으로 알릴 때.39
  - 데이터의 흐름이 단방향이고, 수신자가 누구인지 신경 쓸 필요가 없을 때.
- **서비스를 사용해야 할 때**:
  - 빠르게 완료될 수 있는 요청-응답 상호작용이 필요할 때.39
  - 예: "현재 로봇의 배터리 전압은 얼마인가?", "이 좌표에 대한 역기구학(IK) 해를 계산해줘.", "비용 지도를 초기화해줘(`clear_costmaps`)."
  - **주의**: 서비스 콜백 함수는 오랜 시간이 걸리는 작업을 수행해서는 안 된다. 이는 서비스 클라이언트의 응답 대기 시간을 길게 만들고, 서버 노드의 다른 작업들을 블로킹할 수 있다. 장시간 실행되는 작업에는 반드시 액션을 사용해야 한다.44
- **액션을 사용해야 할 때**:
  - 실행에 수 초에서 수 분 이상이 걸릴 수 있는 장기 실행 작업을 요청할 때.38
  - 작업 진행 중에 중간 피드백이 필요할 때.
  - 작업을 중간에 취소할 수 있는 기능이 필요할 때.42
  - 예: "좌표 (x, y)로 이동하라.", "테이블 위의 물체를 집어서 상자에 넣어라.", "지정된 영역을 360도 스캔하라."

이러한 통신 패턴의 선택은 단순히 API 호출 방식의 차이를 넘어, 노드의 내부 아키텍처에 깊은 영향을 미친다. 서비스를 제공하는 노드는 요청-응답 처리를 위한 비교적 간단한 콜백 구조를 갖지만, 액션 서버를 구현하는 노드는 목표 수락, 실행, 피드백 발행, 취소 요청 처리, 성공/실패/취소 등 복잡한 상태를 관리하는 정교한 상태 머신(state machine)을 내부에 구현해야 한다. 따라서 통신 패턴의 선택은 노드의 내부 복잡성과 실행 모델을 결정하는 근본적인 설계 결정이다.


액션은 ROS 2의 근본적인 통신 프리미티브가 아니다. 대신, 서비스와 토픽을 조합하여 더 높은 수준의 통신 패턴을 구현한 것이다.41 하나의 액션은 내부적으로 3개의 서비스와 2개의 토픽으로 구성된다.41

- **서비스 (3개)**:
  1. `/_action/send_goal`: 클라이언트가 서버에 새로운 목표를 전송하기 위해 호출하는 서비스.
  2. `/_action/cancel_goal`: 클라이언트가 진행 중인 목표의 취소를 요청하기 위해 호출하는 서비스.
  3. `/_action/get_result`: 클라이언트가 완료된 목표의 최종 결과를 요청하기 위해 호출하는 서비스.
- **토픽 (2개)**:
  1. `/_action/feedback`: 서버가 목표를 수행하는 동안 주기적인 피드백 메시지를 발행하는 토픽.
  2. `/_action/status`: 서버가 현재 처리 중인 모든 목표들의 상태(예: `ACCEPTED`, `EXECUTING`, `SUCCEEDED`) 목록을 발행하는 토픽.

이러한 내부 구조는 `ros2 topic list --include-hidden-topics`나 `ros2 service list -a`와 같은 명령어를 통해 직접 확인할 수 있다.41 개발자는 보통 

`rclcpp::action`이나 `rclpy.action`에서 제공하는 고수준 API를 사용하므로 이 내부 구조를 직접 다룰 필요는 없지만, 이를 이해하는 것은 액션의 동작 방식을 깊이 있게 파악하고 디버깅하는 데 큰 도움이 된다.45

| 특성 (Characteristic) | 토픽 (Topic)                  | 서비스 (Service)                  | 액션 (Action)                           |
| --------------------- | ----------------------------- | --------------------------------- | --------------------------------------- |
| **통신 스타일**       | 발행-구독 (Publish-Subscribe) | 요청-응답 (Request-Response, RPC) | 목표-피드백-결과 (Goal-Feedback-Result) |
| **동기성**            | 비동기 (Asynchronous)         | 동기/비동기 RPC                   | 비동기 (Asynchronous)                   |
| **핵심 사용 사례**    | 연속적인 데이터 스트림        | 빠르게 끝나는 원격 호출           | 장시간 실행되는 작업                    |
| **피드백**            | 없음                          | 없음                              | 있음 (주기적)                           |
| **취소 가능**         | 아니오                        | 아니오                            | 예                                      |
| **기반 프리미티브**   | DDS Topic                     | DDS Request-Reply                 | 3개의 서비스, 2개의 토픽                |


잘못된 토픽 설계는 시스템을 비효율적이고, 이해하기 어렵고, 확장하기 힘들게 만든다. 이러한 일반적인 실수들을 '안티패턴'이라고 부른다. 안티패턴을 인지하고 이를 피하는 것은 견고한 시스템을 구축하는 데 필수적이다.


- **만능 토픽 (The God Topic)**: 서로 관련 없는 다양한 정보를 하나의 거대한 메시지 구조에 모두 담아 단일 토픽으로 발행하는 패턴이다. 예를 들어, 로봇의 위치, 속도, 배터리 상태, 센서 값, 에러 코드 등을 모두 포함하는 `GodMessage`를 만들어 `/god_topic`으로 발행하는 경우이다. 이는 단일 책임 원칙을 명백히 위반하며, 특정 정보(예: 배터리 상태)만 필요한 노드조차 거대한 메시지 전체를 수신하고 파싱해야 하므로 심각한 대역폭 및 CPU 낭비를 초래한다. 또한, 메시지 구조의 작은 변경이 수많은 관련 없는 노드에 영향을 미치는 강한 결합(tight coupling)을 유발한다.
- **토픽을 이용한 RPC (RPC-over-Topics)**: 서비스가 제공하는 요청-응답 기능을 토픽으로 흉내 내는 패턴이다. 예를 들어, 클라이언트가 `/request` 토픽에 요청 메시지를 발행하고, 서버가 `/response` 토픽에 응답 메시지를 발행하는 방식이다. 이 방식의 가장 큰 문제는 요청과 응답 간의 연관 관계를 보장할 수 없다는 것이다. 만약 두 클라이언트가 거의 동시에 요청을 발행하면, 어떤 응답이 어떤 요청에 대한 것인지 구분할 방법이 없다. 서비스는 미들웨어 수준에서 이러한 요청-응답의 상관관계를 보장해주므로, RPC가 필요할 때는 반드시 서비스를 사용해야 한다.
- **네임스페이스 오용 (Namespace Misuse)**: 모든 토픽을 `/`로 시작하는 전역 네임스페이스에 정의하는 것은 노드의 재사용성을 심각하게 저해한다. 이는 마치 프로그래밍에서 모든 변수를 전역 변수로 선언하는 것과 같다. 반대로, 너무 깊고 복잡한 네임스페이스 구조는 토픽 이름을 불필요하게 길게 만들어 가독성을 떨어뜨릴 수 있다.14
- **의미 없는 메시지 타입 사용 (Using Non-Semantic Message Types)**: 프로토타이핑 단계를 넘어 실제 애플리케이션 코드에서 `std_msgs/String`이나 `std_msgs/Int32`와 같은 제네릭 타입을 사용하는 것은 대표적인 안티패턴이다.19 이는 데이터의 의미를 코드 내의 문자열 파싱이나 값 비교 로직 속에 숨겨버려, 시스템의 의도를 불분명하게 만들고 잠재적인 버그의 원인이 된다.
- **QoS 불일치 (QoS Mismatch)**: 통신하려는 발행자와 구독자 간에 호환되지 않는 QoS 설정을 사용하여 데이터가 "조용히" 유실되는 문제이다. 이는 ROS 2에서 가장 흔하게 발생하는 문제 중 하나이며, 명시적인 에러 없이 통신이 두절되기 때문에 초보자들이 원인을 찾기 매우 어렵다.


- **만능 토픽 → 세분화된 토픽**: 거대한 메시지를 논리적 데이터 단위(예: 위치, 배터리, 진단)로 분해하고, 각각을 별도의 메시지 타입과 토픽으로 정의하여 리팩토링한다.
- **토픽을 이용한 RPC → 서비스/액션**: 요청-응답 패턴이 필요하면 서비스로 전환한다. 만약 작업이 오래 걸리고 피드백이나 취소 기능이 필요하다면 액션으로 전환하는 것이 올바른 해결책이다.
- **전역 네임스페이스 → 상대/프라이빗 네임스페이스**: 재사용을 염두에 둔 노드는 반드시 상대 네임스페이스를 사용하도록 설계한다. 이후 런치 파일에서 `<group>` 태그와 `<push_ros_namespace>`를 사용하여 노드 그룹에 적절한 네임스페이스를 부여한다.
- **의미 없는 메시지 → 의미 있는 커스텀 메시지**: `std_msgs/String`으로 "start", "stop"을 보내는 대신, `uint8 command` 필드와 `uint8 CMD_START=0`, `uint8 CMD_STOP=1`과 같은 상수를 가진 `Command.msg` 커스텀 메시지를 정의하여 사용한다.


ROS 1에 익숙한 개발자들이 ROS 2로 전환하면서 과거의 습관을 그대로 적용하여 발생하는 안티패턴도 존재한다.

- **Nodelets vs. Components**: ROS 1에서 제로카피 통신을 위해 사용되던 `nodelet`은 강력했지만, 사용법이 다소 복잡하고 제약이 많았다. ROS 2의 컴포넌트는 이 개념을 핵심 기능으로 흡수하여 훨씬 더 유연하고 편리하게 만들었다.8 ROS 1에서의 경험 때문에 컴포넌트의 강력한 성능 이점을 활용하지 않고 모든 노드를 별도의 프로세스로 실행하려는 것은 ROS 2 시대의 안티패턴이라 할 수 있다.
- **중앙 집중적 사고**: ROS 1의 ROS Master와 파라미터 서버에 익숙한 나머지, ROS 2에서도 모든 설정 정보를 관리하는 중앙 노드를 만들려는 경향이 있을 수 있다. 하지만 ROS 2의 철학은 탈중앙화에 있으며, 파라미터는 각 노드에 귀속되어 관리되는 것이 기본이다.8 전역적인 설정이 필요하다면, `TRANSIENT_LOCAL` QoS를 가진 토픽을 통해 상태를 공유하는 방식이 더 ROS 2의 설계에 부합한다.

이러한 안티패턴들의 근본 원인은 ROS 2의 새롭고 더 명시적인 설계 철학을 온전히 받아들이지 못하는 데 있다. ROS 1은 많은 부분이 암묵적인 관례와 기본값으로 "그냥 동작"하는 것처럼 보였다. 반면 ROS 2는 개발자에게 QoS 설정, 컴포넌트 구성 등 더 많은 제어권을 부여하는 대신, 그에 대한 명시적인 설계를 요구한다. ROS 1의 사고방식에 머물러 이러한 새로운 설계 요소를 소홀히 다루는 것이야말로 가장 큰 안티패턴이라 할 수 있다.


이론적인 설계 원칙들이 실제 복잡한 로봇 시스템에서 어떻게 적용되는지 살펴보는 것은 매우 중요하다. ROS 2 생태계의 대표적인 두 프로젝트인 Nav2와 MoveIt2/ros2_control의 토픽 아키텍처는 우리가 논의한 원칙들의 훌륭한 실증 사례이다.


Nav2는 ROS 2를 위한 표준 2D 내비게이션 스택이다. Nav2의 아키텍처는 ROS 2의 통신 패턴을 교과서적으로 활용한다.

- **핵심 인터페이스**: Nav2의 주된 외부 인터페이스는 `nav2_msgs/action/NavigateToPose`라는 액션이다.46 로봇을 특정 목표 지점으로 이동시키는 것은 시간이 오래 걸리고, 중간 경로상의 현재 위치를 피드백으로 받아야 하며, 언제든지 취소될 수 있어야 하는 작업의 전형적인 예이다. 따라서 액션을 사용하는 것은 이 문제에 대한 완벽한 모델링이다.
- **주요 토픽과 QoS 설계**:
  - `/scan` (from LiDAR), `/image` (from Camera): 고주파로 발행되는 원시 센서 데이터. 일부 손실이 허용되므로 `Sensor Data` QoS 프로파일(주로 `BEST_EFFORT` 신뢰성)이 사용된다.
  - `/odom`: 로봇의 바퀴 회전 등으로 추정한 주행 거리계 정보. 경로 추적에 중요하므로 비교적 높은 신뢰성이 요구된다.
  - `/map`: SLAM 등을 통해 생성된 정적 지도. 시스템 시작 시 한 번 발행되고 여러 노드가 사용해야 하므로, `TRANSIENT_LOCAL` 내구성을 가진 `RELIABLE` QoS로 발행하는 것이 표준적인 방법이다.48
  - `/cmd_vel`: 내비게이션 스택의 최종 출력물인 로봇 구동계(mobile base)를 위한 속도 명령. 이 명령이 유실되면 로봇이 멈추거나 오작동할 수 있으므로, 반드시 `RELIABLE` QoS를 사용해야 한다.
  - `/tf`, `/tf_static`: 로봇의 각 부분(link) 간의 좌표계 변환 정보. 시스템의 거의 모든 노드가 이 정보를 필요로 하며, 정적인 변환 정보(`tf_static`)는 `TRANSIENT_LOCAL`로 발행되어 효율성을 높인다.

Nav2는 플래너, 컨트롤러, 복구 행동 등을 각각 독립적인 서버 노드(컴포넌트)로 분리하고, 이들의 실행 흐름을 비헤이비어 트리(Behavior Tree)를 통해 동적으로 조율하는 고도로 모듈화된 아키텍처를 채택했다.46 이는 각 알고리즘을 쉽게 교체하거나 새로운 행동을 추가할 수 있게 하여 시스템의 유연성과 확장성을 극대화한다.


MoveIt2는 로봇 팔(manipulator)의 모션 플래닝을 위한 핵심 프레임워크이며, `ros2_control`은 하드웨어와의 인터페이스를 표준화하는 프레임워크이다. 이 둘의 상호작용은 정교한 인터페이스 설계를 보여준다.

- **MoveIt2**:

  - MoveIt2의 핵심 인터페이스 역시 `moveit_msgs/action/MoveGroup` 액션이다.50 사용자는 목표 관절 각도, 엔드 이펙터의 목표 자세(pose) 등을 골(goal)로 전달하여 모션 플래닝 및 실행을 요청한다. 이 과정은 수 초가 걸릴 수 있고, 실행 궤적을 피드백으로 받으며, 취소가 가능해야 하므로 액션이 가장 적합한 통신 패턴이다.
  - `/joint_states`: 로봇의 모든 관절의 현재 각도, 속도, 힘(effort) 정보를 담고 있는 `sensor_msgs/msg/JointState` 메시지를 발행하는 토픽이다. MoveIt2는 이 토픽을 구독하여 로봇의 현재 상태를 인지한다.
  - `/display_planned_path`: MoveIt2가 계산한 경로를 Rviz와 같은 시각화 도구에서 보여주기 위해 `moveit_msgs/msg/DisplayTrajectory` 메시지를 발행하는 토픽이다.

- **ros2_control**:

  - `ros2_control`은 실시간 제어 루프의 성능을 보장하기 위해 독특한 인터페이스 설계를 사용한다. 외부 ROS 시스템과의 통신에는 토픽이나 액션을 사용하지만, 실시간성이 중요한 내부 제어 루프에서는 토픽을 사용하지 않는다.

  - **외부 인터페이스**: `joint_trajectory_controller`와 같은 `ros2_control`의 컨트롤러는 MoveIt2와 같은 상위 시스템으로부터 명령을 받기 위해 `control_msgs/action/FollowJointTrajectory` 액션 서버를 제공한다. 또한, 로봇의 현재 상태를 외부로 알리기 위해 `joint_state_broadcaster`가 `/joint_states` 토픽을 발행한다.

  - **내부 인터페이스**: 실제 제어 루프 내에서는 **`command_interfaces`**와 **`state_interfaces`**라는 특수한 메모리 공유 데이터 핸들을 사용한다.52 컨트롤러는 하드웨어로부터 읽어온 센서 값(예: 엔코더 값)을 

    `state_interfaces`를 통해 직접 읽고, 계산된 제어 명령(예: 목표 속도, 토크)을 `command_interfaces`에 직접 쓴다. 하드웨어 인터페이스 노드는 이 `command_interfaces`의 값을 읽어 실제 액추에이터에 전달한다.52 이 내부 루프에서는 메시지 직렬화, DDS 전송, 스케줄링 등의 오버헤드가 있는 토픽 통신을 완전히 배제함으로써, 예측 가능하고 낮은 지연 시간을 갖는 하드웨어 제어를 달성한다.

`ros2_control`의 사례는 매우 중요한 설계 원칙을 보여준다: **시스템 수준의 통합에는 토픽과 액션을 사용하고, 실시간성이 극도로 중요한 제어 루프에는 더 직접적인 데이터 공유 메커니즘을 사용하라.** 이는 ROS 2가 단순한 로봇 미들웨어를 넘어, 하드웨어에 가까운 실시간 제어 시스템까지 포괄할 수 있는 프레임워크로 발전했음을 보여주는 증거이다.



ROS 2에서의 토픽 설계는 분산 로봇 시스템의 성능과 안정성을 좌우하는 핵심적인 아키텍처 활동이다. 본 보고서에서 심층적으로 분석한 내용을 바탕으로, 성공적인 토픽 설계를 위한 핵심 원칙들을 다음과 같이 요약할 수 있다.

1. **명확하고 일관된 네이밍**: 상대 경로 네임스페이스를 적극적으로 활용하여 노드의 재사용성을 높이고, Autoware의 `input/output` 접두사와 같은 명확한 관례를 도입하여 데이터 흐름의 가독성을 확보해야 한다.
2. **의미론적으로 명확한 인터페이스**: `std_msgs`와 같은 제네릭 타입의 사용을 지양하고, `common_interfaces`를 최대한 활용해야 한다. 커스텀 메시지가 필요할 경우, 데이터의 물리적 의미와 단위를 명확히 하는 자기 서술적인 메시지를 설계하고, 이를 별도의 `_interfaces` 패키지에서 체계적으로 관리하여 의존성 문제를 원천적으로 방지해야 한다.
3. **데이터 특성에 맞는 QoS 선택**: 데이터의 중요도, 실시간성, 발행 빈도를 고려하여 신뢰성, 내구성, 이력 등의 QoS 정책을 신중하게 선택해야 한다. QoS 불일치로 인한 '조용한 실패'를 방지하기 위해, 통신하는 노드 간의 QoS 호환성을 반드시 검증하는 습관이 필요하다.
4. **올바른 통신 패턴의 사용**: 연속적인 데이터 스트림에는 토픽, 빠르고 간단한 요청-응답에는 서비스, 장시간 실행되며 피드백과 취소가 필요한 작업에는 액션을 사용하는 명확한 기준을 세워야 한다.
5. **성능과 모듈성의 균형**: 단일 책임 원칙에 따라 토픽을 논리적으로 세분화하되, 노드 컴포지션과 IPC의 이점을 극대화할 수 있도록 고대역폭 통신이 필요한 노드들을 전략적으로 그룹화하여 성능과 모듈성 사이의 최적점을 찾아야 한다.


ROS 2와 로봇 미들웨어 생태계는 계속해서 빠르게 발전하고 있다. 토픽 설계와 관련하여 주목해야 할 미래 동향은 다음과 같다.

- **새로운 미들웨어의 등장**: 현재 ROS 2는 DDS를 주요 미들웨어로 사용하지만, Zenoh와 같이 다른 특성을 가진 미들웨어를 통합하려는 노력이 진행 중이다. 새로운 미들웨어의 도입은 QoS 정책의 형태를 변화시키거나, 대규모 분산 시스템에서의 토픽 탐색 및 라우팅 방식에 새로운 패러다임을 제시할 수 있다.
- **실시간성 및 보안 강화**: 실시간 운영체제(RTOS)와의 결합이 더욱 긴밀해지고, 시간 민감 네트워킹(TSN) 기술이 ROS 2에 통합되면서, 마이크로초 단위의 결정론적 통신을 보장하기 위한 새로운 QoS 정책과 도구들이 등장할 것이다. 또한, SROS2의 발전을 통해 토픽 단위의 암호화 및 접근 제어가 더욱 정교해질 것이다.
- **고수준 아키텍처 패턴 및 도구**: 시스템의 복잡도가 증가함에 따라, 개별 토픽 설계를 넘어 전체 시스템의 데이터 흐름을 시각화하고, 아키텍처 안티패턴을 자동으로 탐지하며, 최적의 노드 컴포지션 구성을 추천해주는 등, 고수준의 시스템 설계를 지원하는 지능적인 도구들이 등장할 것으로 예측된다.

결론적으로, ROS 2의 토픽은 단순한 통신 채널이 아니라, 시스템의 근본적인 특성을 규정하는 설계의 핵심 요소이다. 본 보고서에서 제시된 원칙들을 깊이 이해하고 실천함으로써, 로봇 공학자들은 더욱 견고하고, 효율적이며, 미래의 요구사항에 유연하게 대응할 수 있는 확장 가능한 로봇 시스템을 구축할 수 있을 것이다.


1. Understanding topics — ROS 2 Documentation: Foxy documentation, accessed August 21, 2025, https://docs.ros.org/en/foxy/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Topics/Understanding-ROS2-Topics.html
2. Understanding topics — ROS 2 Documentation: Humble documentation, accessed August 21, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Topics/Understanding-ROS2-Topics.html
3. Topics — ROS 2 Documentation: Humble documentation, accessed August 21, 2025, https://docs.ros.org/en/humble/Concepts/Basic/About-Topics.html
4. ROS2 vs. ROS1— key differences and which one is better? | by Omar Elmofty | Medium, accessed August 21, 2025, https://medium.com/@oelmofty/ros2-how-is-it-better-than-ros1-881632e1979a
5. ROS 2 middleware interface, accessed August 21, 2025, https://design.ros2.org/articles/ros_middleware_interface.html
6. ROS on DDS, accessed August 21, 2025, https://design.ros2.org/articles/ros_on_dds.html
7. Concepts — ROS 2 Documentation: Foxy documentation, accessed August 21, 2025, https://docs.ros.org/en/foxy/Concepts.html
8. ROS1 vs ROS2, Practical Overview For ROS Developers - The Robotics Back-End, accessed August 21, 2025, https://roboticsbackend.com/ros1-vs-ros2-practical-overview/
9. ROS 2 Quality of Service policies - ROS2 Design, accessed August 21, 2025, https://design.ros2.org/articles/qos.html
10. Design guide: common patterns in ROS 2 - ROS documentation, accessed August 21, 2025, https://docs.ros.org/en/eloquent/Contributing/Design-Guide.html
11. ROS 2 Communication Basics: Publishers, Subscribers, Topics - Automatic Addison, accessed August 21, 2025, https://automaticaddison.com/ros-2-communication-basics-publishers-subscribers-topics/
12. Exchange Data with ROS 2 Publishers and Subscribers - MATLAB & - MathWorks, accessed August 21, 2025, https://www.mathworks.com/help/ros/ug/exchange-data-with-ros-2-publishers-and-subscribers.html
13. ROS2 Learning Series - Blog 2 - Basic Concepts - element14 Community, accessed August 21, 2025, https://community.element14.com/technologies/robotics/b/blog/posts/ros2_2d00_learning_2d00_series_2d00_blog2
14. Topic and Service name mapping to DDS - ROS 2 Design, accessed August 21, 2025, https://design.ros2.org/articles/topic_and_service_names.html
15. Learn ROS 2 - Intro: Topics - Hadabot, accessed August 21, 2025, https://www.hadabot.com/learn-ros2-intro-topics.html?step=experiment-ros2-cli-topic
16. Naming convention - Autoware Documentation, accessed August 21, 2025, https://tier4.github.io/pilot-auto-ros2-iv-archive/tree/main/design/software_architecture/NamingConvention/
17. ROS2 Create Custom Message (Msg/Srv) - The Robotics Back-End, accessed August 21, 2025, https://roboticsbackend.com/ros2-create-custom-message/
18. ros2/common_interfaces: A set of packages which contain common interface files (.msg and .srv). - GitHub, accessed August 21, 2025, https://github.com/ros2/common_interfaces
19. Messages and Services (ros2 interface) - (Murilo's) ROS2 Tutorial - Read the Docs, accessed August 21, 2025, https://ros2-tutorial.readthedocs.io/en/latest/messages.html
20. ROS Package: std_msgs, accessed August 21, 2025, https://index.ros.org/p/std_msgs/
21. Custom message with Python ROS 2 Jazzy Jalisco - Reddit, accessed August 21, 2025, https://www.reddit.com/r/ROS/comments/1g4youj/custom_message_with_python_ros_2_jazzy_jalisco/
22. Creating custom ROS 2 msg and srv files, accessed August 21, 2025, https://docs.ros.org/en/crystal/Tutorials/Custom-ROS2-Interfaces.html
23. ROS/Patterns/Conventions - ROS Wiki, accessed August 21, 2025, http://wiki.ros.org/ROS/Patterns/Conventions
24. About ROS 2 interfaces — ROS 2 Documentation: Foxy documentation, accessed August 21, 2025, https://docs.ros.org/en/foxy/Concepts/About-ROS-Interfaces.html
25. Quality of Service settings — ROS 2 Documentation: Rolling documentation, accessed August 21, 2025, https://docs.ros.org/en/rolling/Concepts/Intermediate/About-Quality-of-Service-Settings.html
26. Manage Quality of Service Policies in ROS 2 - MATLAB & Simulink - MathWorks, accessed August 21, 2025, https://www.mathworks.com/help/ros/ug/manage-quality-of-service-policies-in-ros2.html
27. Manage Quality of Service Policies in ROS 2 Application with TurtleBot - MATLAB &, accessed August 21, 2025, https://www.mathworks.com/help/ros/ug/manage-quality-of-service-policies-using-ros2-in-turtlebot.html
28. ROS QoS - Deadline, Liveliness, and Lifespan - ROS 2 Design, accessed August 21, 2025, https://design.ros2.org/articles/qos_deadline_liveliness_lifespan.html
29. ROS 2 Quality of Service (QoS) - Isaac Sim Documentation, accessed August 21, 2025, https://docs.isaacsim.omniverse.nvidia.com/4.2.0/ros2_tutorials/tutorial_ros2_qos.html
30. Impact of ROS 2 Node Composition in Robotic Systems - arXiv, accessed August 21, 2025, https://arxiv.org/pdf/2305.09933
31. rclcpp::SerializedMessage Class Reference - ROS Documentation, accessed August 21, 2025, https://docs.ros2.org/foxy/api/rclcpp/classrclcpp_1_1SerializedMessage.html
32. How to get the serialized message size/length in ROS2 - ROS Answers archive, accessed August 21, 2025, https://answers.ros.org/question/303992/
33. Unable to receive large messages - ros2 - Robotics Stack Exchange, accessed August 21, 2025, https://robotics.stackexchange.com/questions/102371/unable-to-receive-large-messages
34. Probabilistic Latency Analysis of the Data Distribution Service in ROS 2 - arXiv, accessed August 21, 2025, https://arxiv.org/html/2508.10413v1
35. Inspecting topics (ros2 topic) - (Murilo's) ROS2 Tutorial - Read the Docs, accessed August 21, 2025, https://ros2-tutorial.readthedocs.io/en/latest/inspecting_topics.html
36. What exactly does ros2 topic hz display - Robotics Stack Exchange, accessed August 21, 2025, https://robotics.stackexchange.com/questions/102519/what-exactly-does-ros2-topic-hz-display
37. Tool for analyzing ROS network traffic? - Robotics Stack Exchange, accessed August 21, 2025, https://robotics.stackexchange.com/questions/40981/tool-for-analyzing-ros-network-traffic
38. Topics vs. Services vs. Actions in ROS2-Based Projects, accessed August 21, 2025, https://automaticaddison.com/topics-vs-services-vs-actions-in-ros2-based-projects/
39. Topics vs Services vs Actions — ROS 2 Documentation: Foxy documentation, accessed August 21, 2025, https://docs.ros.org/en/foxy/How-To-Guides/Topics-Services-Actions.html
40. Understanding services — ROS 2 Documentation: Foxy documentation, accessed August 21, 2025, https://docs.ros.org/en/foxy/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Services/Understanding-ROS2-Services.html
41. Actions - ROS 2 Design, accessed August 21, 2025, https://design.ros2.org/articles/actions.html
42. Understanding actions — ROS 2 Documentation: Foxy documentation, accessed August 21, 2025, https://docs.ros.org/en/foxy/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Actions/Understanding-ROS2-Actions.html
43. Creating ROS 2 Actions - Foxglove, accessed August 21, 2025, https://foxglove.dev/blog/creating-ros2-actions
44. Services — ROS 2 Documentation: Humble documentation, accessed August 21, 2025, https://docs.ros.org/en/humble/Concepts/Basic/About-Services.html
45. ROS2 Part 10 - ROS2 Actions in Python and C++ - RoboticsUnveiled, accessed August 21, 2025, https://www.roboticsunveiled.com/ros2-actions-in-python-and-cpp/
46. Navigation Concepts — Nav2 1.0.0 documentation, accessed August 21, 2025, https://docs.nav2.org/concepts/index.html
47. ROS 2 Navigation Tuning Guide – Nav2 - Automatic Addison, accessed August 21, 2025, https://automaticaddison.com/ros-2-navigation-tuning-guide-nav2/
48. ROS 2 Navigation — ROS 2 workshop documentation - Read the Docs, accessed August 21, 2025, https://ros2-industrial-workshop.readthedocs.io/en/latest/_source/navigation/ROS2-Navigation.html
49. Navigation2 Overview, accessed August 21, 2025, https://roscon.ros.org/2019/talks/roscon2019_navigation2_overview_final.pdf
50. ROS 2 Manipulation Basics - The Construct, accessed August 21, 2025, https://www.theconstruct.ai/robotigniteacademy_learnros/ros-courses-library/ros2-manipulation-basics/
51. o3de-extras/Templates/Ros2RoboticManipulationTemplate/README.md at development, accessed August 21, 2025, https://github.com/o3de/o3de-extras/blob/development/Templates/Ros2RoboticManipulationTemplate/README.md
52. Example 7: Full tutorial with a 6DOF robot — ROS2_Control: Rolling ..., accessed August 21, 2025, https://control.ros.org/rolling/doc/ros2_control_demos/example_7/doc/userdoc.html