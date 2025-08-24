[ROS2 Autoware](./index.md)
# ROS2 Autoware


자율주행 기술은 현대 공학의 정점에 있는 가장 복잡하고 도전적인 분야 중 하나다. 수많은 센서로부터 방대한 데이터를 실시간으로 처리하고, 예측 불가능한 환경 속에서 안전을 보장하며, 정밀한 차량 제어를 수행해야 하는 이 과업은 단일 기업이나 연구 기관의 힘만으로는 해결하기 어렵다. 이러한 배경 속에서 오픈소스 협업 모델은 자율주행 기술의 민주화와 발전을 가속하는 핵심 동력으로 부상했다. 전 세계의 개발자와 연구자들이 지식을 공유하고 코드를 공동으로 개발함으로써, 개발 비용을 절감하고 기술 표준을 형성하며 강력한 생태계를 구축할 수 있기 때문이다.1


이러한 오픈소스 정신의 중심에 Autoware가 있다. 2015년 처음 등장한 Autoware는 인지, 판단, 제어에 필요한 모든 기능을 포함하는 세계 최초의 "all-in-one" 자율주행 오픈소스 소프트웨어로서, 학계와 산업계에 큰 반향을 일으켰다.2 Autoware는 특정 기업의 독점 기술에 의존하지 않고 누구나 자율주행 기술을 연구하고 개발할 수 있는 길을 열어주었으며, 이는 곧 다양한 차량과 응용 분야로의 빠른 확산으로 이어졌다.4


초기 Autoware(Autoware.AI)는 로봇 개발의 표준 프레임워크였던 ROS1(Robot Operating System 1)을 기반으로 구축되었다. 하지만 ROS1은 학술 연구 및 프로토타이핑에는 적합했지만, 상용 수준의 자율주행 시스템이 요구하는 엄격한 실시간성과 안정성, 안전성 요구사항을 충족하기에는 근본적인 한계를 가지고 있었다. 자율주행 기술이 연구실을 넘어 실제 도로로 나아가기 위해서는 프레임워크의 근본적인 체질 개선이 필요했고, 그 해답은 ROS2였다. ROS2로의 전환은 단순한 버전 업그레이드가 아니라, 자율주행 소프트웨어가 '제품'으로 거듭나기 위한 필연적인 패러다임의 전환이었다.


ROS1은 몇 가지 핵심적인 구조적 한계를 내포하고 있었다. 첫째, **중앙 집중형 통신 구조**다. 모든 노드(ROS의 실행 단위)의 이름 등록과 탐색을 담당하는 `roscore`라는 마스터(Master) 프로세스에 시스템 전체가 의존했다.5 만약 `roscore`가 중단되면 시스템 전체의 통신이 마비되는 단일 장애점(Single Point of Failure, SPOF) 문제를 안고 있었으며, 이는 수십, 수백 개의 노드가 동작하는 복잡한 자율주행 시스템의 확장성과 안정성에 치명적이었다.

둘째, **비실시간성 통신** 문제다. ROS1의 기본 통신 방식인 `TCPROS`는 표준 TCP/IP 프로토콜 위에서 동작하여 데이터 전송의 성공 여부는 보장하지만, 전송 시간을 보장하지는 않는다. 자율주행 시스템에서는 전방 장애물 정보나 제어 명령과 같은 안전 필수(safety-critical) 데이터가 정해진 시간 안에 반드시 전달되어야 한다. 이러한 결정론적(deterministic) 데이터 전송의 부재는 ROS1을 실제 차량에 적용하는 데 가장 큰 걸림돌 중 하나였다.6

셋째, **단일 스레드 기반의 콜백 처리**다. ROS1은 기본적으로 단일 스레드에서 모든 구독 메시지의 콜백 함수를 순차적으로 처리하는 '스핀(spin)' 모델을 사용했다.5 이는 여러 개의 CPU 코어를 가진 최신 프로세서의 성능을 제대로 활용하지 못하게 만들어, 인지, 판단, 제어 등 여러 고부하 연산을 동시에 수행해야 하는 자율주행 시스템에서 병목 현상을 유발하기 쉬웠다.


ROS2는 ROS1의 이러한 한계들을 극복하기 위해 아키텍처 수준에서부터 완전히 새롭게 설계되었다.

첫째, **미들웨어로 DDS(Data Distribution Service)를 채택**했다. DDS는 군사, 항공, 금융 등 미션 크리티컬(mission-critical) 산업 분야에서 오랫동안 사용되어 온 표준 통신 미들웨어다.5 DDS는 `roscore`와 같은 중앙 마스터 없이 노드들이 P2P(Peer-to-Peer) 방식으로 서로를 발견하고 통신하는 탈중앙화 구조를 갖는다. 이로써 단일 장애점이 원천적으로 제거되었다.

DDS의 가장 강력한 기능은 **QoS(Quality of Service) 정책**이다.5 개발자는 각 데이터(토픽)의 특성에 맞게 통신 방식을 세밀하게 설정할 수 있다. 예를 들어, 제어 명령(`control_cmd`)처럼 반드시 전달되어야 하는 데이터는 `신뢰성(Reliability)`을 `RELIABLE`로, LiDAR 포인트 클라우드처럼 최신 데이터가 더 중요하고 일부 유실되어도 괜찮은 대용량 데이터는 `BEST_EFFORT`로 설정할 수 있다. 또한, 데이터의 보존 기간을 정하는 `내구성(Durability)`, 데이터 수신 데드라인을 명시하는 `데드라인(Deadline)` 등 다양한 QoS 프로파일을 통해 '중요한 데이터는 반드시 시간 내에 전달한다'는 실시간 시스템의 핵심 요구사항을 충족시킬 수 있다. 이는 자율주행 시스템의 안전성과 신뢰성을 보장하는 핵심적인 기술적 기반이 된다.

둘째, **실행 모델을 혁신**했다. ROS2는 **멀티스레드 실행기(Multi-Threaded Executor)**를 도입하여 여러 노드 또는 단일 노드 내의 여러 콜백 그룹을 다수의 스레드에 분산시켜 병렬로 처리할 수 있게 했다.5 이를 통해 최신 멀티코어 CPU의 자원을 최대한 활용하여 시스템 전체의 처리량을 극대화할 수 있으며, 이는 Autoware와 같이 복잡하고 무거운 연산을 동시에 수행해야 하는 시스템에 필수적이다.

셋째, **API 및 빌드 시스템을 현대화**했다. ROS2는 C++11/14/17 표준을 적극적으로 활용하고 Python 3를 기본으로 지원한다.8 빌드 시스템은 `catkin`에서 `ament`로 전환되었는데, 이는 패키지별 격리 빌드를 통해 의존성 관리를 명확히 하고, 심볼릭 링크 설치 옵션을 제공하여 개발 중인 소스 코드를 재빌드 없이 즉시 시스템에 반영하는 등 개발 편의성을 크게 향상시켰다.8 또한, `rcl`(ROS Client Library)이라는 C 언어 기반의 공통 라이브러리 위에 C++, Python 등의 클라이언트 라이브러리(`rclcpp`, `rclpy`)를 구축함으로써, 여러 프로그래밍 언어에서 일관된 기능과 성능을 제공하기 용이해졌다.5

이러한 ROS2의 근본적인 개선은 Autoware가 연구용 프로토타입을 넘어, ISO 26262와 같은 기능 안전 표준을 만족하고 상용 배포가 가능한 수준의 신뢰성을 갖춘 소프트웨어로 발전하는 데 결정적인 발판을 마련해주었다. 즉, Autoware가 ROS2를 채택한 것은 단순히 최신 기술을 따르는 유행이 아니라, **안전 인증과 상용화를 전제로 한 필연적인 전략적 선택**이었다. 더 나아가, ROS2가 Ubuntu Linux뿐만 아니라 Windows, macOS 등 다양한 운영체제를 공식적으로 지원하게 된 점 5은 자동차 산업에서 널리 사용되는 Windows 기반의 개발 및 시뮬레이션 도구와의 통합을 용이하게 하고, 더 넓은 개발자 커뮤니티의 참여를 유도하여 Autoware 생태계의 확장을 가속하는 중요한 잠재력이 되고 있다.


Autoware의 발전사는 단순히 기능이 추가되는 선형적인 과정이 아니었다. 그것은 오픈소스 프로젝트가 겪는 '성장통'을 극복하고, 안정성과 혁신성 사이의 균형을 맞추며 지속 가능한 거버넌스 모델을 찾아가는 역동적인 여정이었다. 이 과정은 Autoware.AI, Autoware.Auto, 그리고 현재의 Autoware Core/Universe라는 세 개의 뚜렷한 단계로 구분할 수 있다.


2015년에 등장한 Autoware.AI는 ROS1을 기반으로 자율주행에 필요한 거의 모든 기능을 제공하는 최초의 통합 오픈소스 소프트웨어였다. 이는 전 세계 100개 이상의 기업과 30종 이상의 차량에 적용되며 자율주행 기술의 대중화 가능성을 증명하는 위대한 시작이었다.3 많은 기업들이 Autoware.AI를 활용해 자사의 자율주행 컨셉을 검증하고 프로토타입을 제작했다.

하지만 빠른 성장과 기능 추가의 이면에는 심각한 **기술적 부채(technical debt)**가 쌓이고 있었다. 가장 큰 문제는 **명확한 아키텍처의 부재**였다.3 초기 설계 없이 기능들이 유기적으로 추가되면서, 각 모듈 간의 의존성이 복잡하게 얽히는 '스파게티 코드' 현상이 발생했다. 이는 코드의 유지보수를 어렵게 하고, 특정 모듈의 수정이 예상치 못한 다른 부분에 영향을 미치는 부작용을 낳았다.

또한, **품질 관리의 부재**도 심각했다. 통일된 코딩 표준이 없어 패키지마다 코드 스타일이 제각각이었고, 테스트 커버리지는 매우 낮았다.3 이는 소프트웨어의 안정성과 신뢰성을 담보하기 어렵게 만들었다. 결정적으로, ROS1 자체의 한계와 맞물려 결정론적 실행이나 메모리 안전성 확보가 불가능했기 때문에, Autoware.AI는 실제 상용차에 요구되는 **기능 안전(Functional Safety) 표준을 만족시키고 인증을 받는 것이 거의 불가능**했다.10


이러한 Autoware.AI의 한계를 극복하기 위해 Autoware Foundation(AWF)은 새로운 프로젝트, Autoware.Auto를 시작했다. Autoware.Auto의 목표는 단순히 Autoware.AI를 ROS2로 포팅하는 것이 아니었다. 그것은 **처음부터 다시 설계하여 고품질, 고신뢰성, 그리고 안전 인증이 가능한 소프트웨어 스택을 만드는 것**이었다.3

Autoware.Auto는 ROS2를 기반으로, 현대적인 소프트웨어 공학 기법을 전면적으로 도입했다. 100%에 가까운 테스트 커버리지, 포괄적인 문서화, 엄격한 코드 리뷰, 그리고 CI/CD(지속적 통합/지속적 배포) 파이프라인을 통해 코드의 품질을 최고 수준으로 유지하고자 했다.10 또한, 결정론적 실행(deterministic execution)을 목표로 아키텍처를 설계하여 안전성을 확보하려 했다. 프로젝트의 범위를 명확히 하기 위해, 초기 목표 운영 설계 도메인(Operational Design Domain, ODD)을 자율 발렛 파킹(AVP)이나 화물 운송(Cargo Delivery)과 같이 상대적으로 통제된 환경으로 한정했다.11

그러나 이러한 엄격한 품질 우선주의는 새로운 문제를 낳았다. **너무 높은 진입 장벽**이 생긴 것이다. 새로운 기능을 하나 추가하기 위해서는 수많은 테스트 코드와 문서를 작성하고 엄격한 리뷰 과정을 통과해야 했다. 이로 인해 대학의 연구자들이나 학생들이 최신 연구 결과를 실험하고 기여하기가 매우 어려워졌고, 결과적으로 프로젝트의 혁신 속도가 저하되는 부작용이 발생했다.3 또한, 안정성을 유지하기 위한 관리 오버헤드가 커지면서, 대규모 아키텍처를 변경하거나 실험하는 것 역시 매우 경직되고 어려워졌다.


Autoware.Auto가 직면한 '안정성'과 '혁신성' 사이의 딜레마를 해결하기 위해, AWF는 Autoware.Auto의 엄격함과 Autoware.AI의 개방성 사이의 균형을 맞추는 새로운 패러다임인 **Autoware Core/Universe**를 제시했다.3 이는 기술적 아키텍처를 넘어, 프로젝트의 사회적, 경제적 구조를 재설계한 것에 가깝다.

- **Autoware Core:** Autoware.Auto의 철학을 계승하여, **안정적이고 잘 테스트된 핵심 기능의 집합**으로 정의된다.3 AWF가 주도적으로 관리하며, 매우 엄격한 코드 품질 기준과 리뷰 프로세스를 통과한 코드만이 포함될 수 있다. Core는 AVP, 화물 운송, 로보버스 등 Autoware가 공식적으로 지원하는 ODD를 구현하는 데 필요한 필수 기능들을 담고 있어, 상용 수준의 제품을 개발하려는 기업들에게 신뢰할 수 있는 기반을 제공한다.4
- **Autoware Universe:** Core 위에 구축되는 **확장 패키지들의 집합**이다.13 이곳은 커뮤니티, 연구자, 개발자들이 최신 알고리즘이나 실험적인 기능을 자유롭게 기여하고 테스트할 수 있는 '놀이터'와 같은 공간이다. Core보다 완화된 코드 품질 요구사항을 적용하여 진입 장벽을 낮춤으로써, 혁신을 촉진하고 생태계의 활력을 유지한다.3 Universe에서 충분히 검증되고 많은 사용자에게 유용하다고 인정받은 기능은, 엄격한 리뷰를 거쳐 향후 Core로 편입될 수 있는 경로를 제공한다.3

이러한 전환 과정에서 기존 Autoware.AI 사용자들의 불편을 최소화하기 위해 **`ros1_bridge`**라는 현명한 전략이 사용되었다. `ros1_bridge`는 ROS1 노드와 ROS2 노드 간의 토픽과 서비스를 양방향으로 변환해주는 패키지다.16 이를 통해, 시스템의 일부는 새로운 ROS2 기반의 Autoware.Auto/.Universe 코드로 교체하면서도, 나머지 부분은 기존의 ROS1 기반 Autoware.AI 코드를 그대로 사용하는 하이브리드 시스템을 구성할 수 있었다. 마치 항해 중인 '테세우스의 배'의 널빤지를 하나씩 교체하듯이, 시스템의 중단 없이 점진적으로 전체 시스템을 ROS2로 마이그레이션하는 전략을 취한 것이다.10

결론적으로, Autoware의 진화 과정은 오픈소스 프로젝트가 어떻게 성장통을 겪고 지속 가능한 거버넌스 모델을 찾아가는지를 보여주는 전형적인 사례다. 초기 '일단 되게 만들자'는 단계(Autoware.AI)에서 기술 부채 문제에 직면했고, 이에 대한 반작용으로 '품질과 안정을 최우선으로' 하는 단계(Autoware.Auto)를 거쳤으나 혁신이 저해되는 부작용을 겪었다. 최종적으로 '안정성과 혁신의 분리 및 공존'이라는 성숙한 단계(Core/Universe)에 도달함으로써, 상용 기업에게는 신뢰를, 연구 커뮤니티에게는 활력을 제공하는 균형 잡힌 생태계 구조를 확립하게 되었다.

------

**표 1: Autoware 버전별 특성 비교**

| 항목          | Autoware.AI                            | Autoware.Auto                               | Autoware Core/Universe                                |
| ------------- | -------------------------------------- | ------------------------------------------- | ----------------------------------------------------- |
| **기반 ROS**  | ROS 1                                  | ROS 2                                       | ROS 2                                                 |
| **주요 목표** | 기능의 빠른 구현, 프로토타이핑         | 고품질, 고신뢰성, 안전 인증                 | 안정성과 혁신의 공존                                  |
| **아키텍처**  | 명확한 구조 부재, 높은 모듈 결합도     | 모듈화, 명확한 인터페이스, 결정론적 설계    | Core(안정)와 Universe(확장)의 분리, 마이크로 아키텍처 |
| **품질 정책** | 낮음 (테스트 커버리지 부족, 표준 부재) | 매우 높음 (높은 테스트 커버리지, 엄격한 CI) | Core: 매우 높음 / Universe: 완화됨                    |
| **개발 주체** | 초기 커뮤니티 주도                     | AWF 주도, 소수 전문가 중심                  | Core: AWF 주도 / Universe: 커뮤니티 중심              |
| **장점**      | 풍부한 기능, 낮은 초기 진입 장벽       | 높은 안정성 및 신뢰성, 상용화 가능성        | 안정적 기반 위에서 빠른 혁신 가능, 넓은 참여 유도     |
| **단점**      | 기술 부채, 낮은 안정성, 안전 인증 불가 | 높은 진입 장벽, 개발 속도 저하, 경직성      | Core와 Universe 간의 동기화 문제, 파편화 가능성       |

------


Autoware.Universe의 아키텍처는 자율주행이라는 복잡한 과업을 논리적인 기능 단위로 분할한 6개의 핵심 스택(Stack)으로 구성된다. 이 스택들은 센서 데이터 입력부터 최종적인 차량 제어 명령 출력까지, 데이터가 처리되는 파이프라인을 형성한다. 각 스택은 독립적인 역할을 수행하면서도, ROS2의 토픽과 서비스를 통해 유기적으로 연결되어 전체 시스템을 이룬다. 이 구조는 특정 기능을 사용자의 필요에 맞게 교체하거나 확장할 수 있는 '마이크로 아키텍처'의 근간이 된다.17


Autoware의 데이터 흐름은 일반적으로 다음과 같은 계층적 구조를 따른다 4:

1. **Sensing:** LiDAR, 카메라, IMU, GNSS 등 각종 센서로부터 원시 데이터를 수집한다.
2. **Localization & Perception:** 수집된 센서 데이터와 고정밀 지도를 이용해 차량의 현재 위치를 추정(Localization)하고, 주변 환경의 객체를 탐지(Perception)한다.
3. **Planning:** 추정된 차량 위치와 인지된 객체 정보를 바탕으로 목적지까지의 안전하고 효율적인 주행 경로를 계획한다.
4. **Control:** 계획된 경로를 정확히 추종하도록 차량의 조향 및 가감속을 제어한다.
5. **Vehicle Interface:** 생성된 제어 명령을 실제 차량이 이해할 수 있는 신호(예: CAN)로 변환하여 전달한다.

이 과정에서 각 모듈은 ROS2 토픽을 통해 데이터를 주고받는다. 예를 들어, LiDAR 센서 드라이버는 `sensor_msgs/PointCloud2` 타입의 토픽을 발행하고, 인지 모듈은 이를 구독하여 처리한 뒤 `autoware_auto_perception_msgs/DetectedObjects` 타입의 객체 목록을 발행한다. 경로 계획 모듈은 이 객체 목록과 위치 정보를 입력받아 `autoware_auto_planning_msgs/Trajectory`를 생성하고, 최종적으로 제어 모듈은 이를 바탕으로 `autoware_auto_control_msgs/AckermannControlCommand`를 차량 인터페이스로 보낸다.19 이러한 전체 노드와 토픽의 연결 관계는 `rqt_graph`와 같은 도구를 통해 시각적으로 확인할 수 있다.20


Localization은 자율주행의 가장 기본이 되는 기술로, 차량이 지도 상에서 자신의 정확한 위치와 자세(6-DOF: x, y, z, roll, pitch, yaw)를 아는 것이다. Autoware는 주로 고정밀 3D 포인트 클라우드 지도(HD Map)와 실시간 LiDAR 스캔 데이터를 정합(registration)하는 방식을 사용한다.


NDT는 포인트 클라우드 정합을 위한 강력하고 효율적인 알고리즘이다. 포인트 대 포인트로 비교하는 ICP(Iterative Closest Point)와 달리, NDT는 기준이 되는 맵 포인트 클라우드를 작은 3D 공간인 복셀(Voxel)로 나누고, 각 복셀에 포함된 점들의 통계적 분포를 **정규분포(Normal Distribution)**로 모델링한다.21 즉, 맵을 이산적인 점의 집합이 아닌, 연속적인 확률 분포의 조합으로 표현하는 것이다.

실시간으로 스캔된 포인트 클라우드를 이 확률 분포 모델에 대입했을 때, 그 점들이 나타날 확률(likelihood)이 최대가 되도록 현재 차량의 위치와 자세(Transform)를 최적화 문제로 풀어낸다. 이 방식은 ICP에 비해 초기 위치 오차에 더 강건하고, 계산 속도가 빠르다는 장점이 있어 널리 사용된다.21

수학적 모델링:

어떤 포인트 $\mathbf{x}$가 특정 복셀에 속할 확률 밀도 함수(PDF)는 해당 복셀 내 점들의 평균 벡터 $\mathbf{q}$와 공분산 행렬 $\mathbf{\Sigma}$를 갖는 다변량 정규분포로 근사된다.23
$$
p(\mathbf{x}) \approx \exp\left(-\frac{(\mathbf{x} - \mathbf{q})^T \mathbf{\Sigma}^{-1} (\mathbf{x} - \mathbf{q})}{2}\right)
$$
현재 스캔된 포인트들의 집합 $\{\mathbf{x}_1,..., \mathbf{x}_n\}$과, 이 점들을 변환(회전 $\mathbf{R}$과 이동 $\mathbf{t}$로 구성된 변환 파라미터 $\mathbf{p}$)시키는 함수 $T(\mathbf{p}, \mathbf{x}_k) = \mathbf{R}\mathbf{x}_k + \mathbf{t}$가 주어졌을 때, NDT의 목표는 변환된 점들이 맵의 확률 분포 상에서 가질 우도(likelihood)의 합, 즉 score 함수를 최대화하는 파라미터 $\mathbf{p}$를 찾는 것이다.22
$$
\text{maximize}_{\mathbf{p}} \sum_{k=1}^{n} \exp\left(-\frac{(T(\mathbf{p}, \mathbf{x}_k) - \mathbf{q}_k)^T \mathbf{\Sigma}_k^{-1} (T(\mathbf{p}, \mathbf{x}_k) - \mathbf{q}_k)}{2}\right)
$$
여기서 $\mathbf{q}_k$와 $\mathbf{\Sigma}_k$는 변환된 포인트 $T(\mathbf{p}, \mathbf{x}_k)$가 위치한 복셀의 평균과 공분산이다. 이 최적화 문제는 비선형이므로, 뉴턴 방법(Newton's method)과 같은 경사하강법 기반의 수치 최적화 기법을 사용하여 해를 구한다.23


NDT만으로는 터널이나 특징이 없는 긴 도로에서 위치 추정 오차가 누적될 수 있다. 이를 보완하기 위해 IMU(관성 측정 장치), GNSS(위성 항법 시스템), 차량 바퀴의 회전 수로 속도를 측정하는 휠 오도메트리(Wheel Odometry) 등 다양한 센서의 데이터를 융합한다. 차량의 움직임은 비선형(nonlinear) 동역학 모델을 따르기 때문에, 이를 선형화하여 칼만 필터를 적용하는 **확장 칼만 필터(EKF)**가 널리 사용된다.24

수학적 모델링:

EKF는 예측(Prediction)과 업데이트(Update)의 두 단계를 반복한다. 차량의 상태 벡터를 $\mathbf{x} = [x, y, \psi, v]^T$ (x, y 위치, 헤딩 각 $\psi$, 속도 $v$)로 정의하자.24

1. **예측 (Prediction) 단계:** 이전 시점($k-1$)의 상태와 제어 입력($\mathbf{u}_k$, 예: 가속도, 조향각)을 이용해 현재 시점($k$)의 상태를 예측한다.

   - 상태 예측: $\hat{\mathbf{x}}_{k|k-1} = f(\hat{\mathbf{x}}_{k-1|k-1}, \mathbf{u}_k)$

   - 오차 공분산 예측: $\mathbf{P}_{k|k-1} = \mathbf{F}_k \mathbf{P}_{k-1|k-1} \mathbf{F}_k^T + \mathbf{Q}_k$

     여기서 $f$는 비선형 상태 전이 함수(차량 운동 모델), $\mathbf{F}_k$는 $f$를 현재 상태에 대해 편미분한 자코비안 행렬, $\mathbf{P}$는 오차 공분산 행렬, $\mathbf{Q}_k$는 프로세스 노이즈(모델의 불확실성)의 공분산이다.27

2. **업데이트 (Update) 단계:** 예측된 상태와 실제 센서 측정값($\mathbf{z}_k$, 예: GNSS 위치, NDT 위치)의 차이를 이용해 예측값을 보정한다.

   - 칼만 이득 계산: $\mathbf{K}_k = \mathbf{P}_{k|k-1} \mathbf{H}_k^T (\mathbf{H}_k \mathbf{P}_{k|k-1} \mathbf{H}_k^T + \mathbf{R}_k)^{-1}$

   - 상태 업데이트: $\hat{\mathbf{x}}_{k|k} = \hat{\mathbf{x}}_{k|k-1} + \mathbf{K}_k (\mathbf{z}_k - h(\hat{\mathbf{x}}_{k|k-1}))$

   - 오차 공분산 업데이트: $\mathbf{P}_{k|k} = (\mathbf{I} - \mathbf{K}_k \mathbf{H}_k) \mathbf{P}_{k|k-1}$

     여기서 $h$는 상태로부터 측정값을 예측하는 비선형 관측 함수, $\mathbf{H}_k$는 $h$의 자코비안 행렬, $\mathbf{R}_k$는 측정 노이즈(센서의 불확실성)의 공분산이다.27 이 과정을 반복하며 EKF는 각 센서의 장점을 취하고 단점을 보완하여 최적의 상태 추정치를 제공한다.


Perception은 센서 데이터를 해석하여 주변 환경에 존재하는 객체(차량, 보행자, 자전거 등)를 탐지하고, 그들의 위치, 크기, 속도, 종류 등을 파악하는 과정이다. Autoware는 주로 LiDAR 센서를 이용한 3D 인지에 중점을 둔다.


Autoware.Auto에서 제시된 기본적인 LiDAR 인식 파이프라인은 다음과 같은 순차적인 노드들로 구성된다 29:

1. **`point_cloud_filter_transform_node`:** 이 노드는 파이프라인의 가장 첫 단계다. 차량에 장착된 여러 LiDAR 센서로부터 들어오는 포인트 클라우드 데이터를 각각의 센서 좌표계에서 차량의 공통 기준 좌표계(예: `base_link`)로 변환(transform)하고 통합하는 역할을 한다. 또한, 설정된 거리나 강도(intensity)를 기준으로 불필요한 노이즈 포인트를 제거(filter)하여 후속 처리의 부담을 줄여준다.
2. **`ray_ground_classifier_node`:** 변환되고 필터링된 포인트 클라우드에서 **지면을 분리**하는 중요한 역할을 한다. 자율주행에서는 도로면 자체가 아닌 도로 위의 객체들이 주된 관심사이기 때문이다. 이 노드는 LiDAR 원점으로부터 각 포인트까지의 가상의 선(ray)을 긋고, 이 선의 기울기나 높이 변화 등을 분석하여 지면에 해당하는 포인트들을 분류하고 제거한다. 지면이 제거되면 남은 포인트들은 잠재적인 장애물일 가능성이 높아진다.
3. **`euclidean_cluster_node`:** 지면이 제거된 비-지면(non-ground) 포인트들을 입력으로 받아, 이들을 개별적인 객체 단위로 **그룹화(클러스터링)**한다. 가장 널리 쓰이는 방법은 유클리드 거리 기반 클러스터링으로, 물리적으로 가까이 있는 포인트들을 하나의 덩어리(클러스터)로 묶는 방식이다. 이렇게 생성된 각 클러스터에 대해 최소 경계 상자(Minimum Bounding Box)를 계산하여, 최종적으로 각 객체의 위치, 크기, 방향을 나타내는 `DetectedObjects` 메시지를 발행한다.

최근 Autoware.Universe에서는 이러한 전통적인 방식에 더해, YOLOX나 RTMDet과 같은 딥러닝 기반의 2D/3D 객체 탐지 모델을 통합하여 인식 성능을 높이려는 시도가 활발히 이루어지고 있다.30 또한 카메라와 LiDAR 데이터를 융합하는 센서 퓨전 기술을 통해 악천후 등 특정 센서가 취약한 환경에서도 강건한 인식 성능을 확보하려 노력하고 있다.


Planning은 현재 위치에서 최종 목적지까지, 주변의 정적/동적 장애물과 교통 법규를 모두 고려하여 안전하고, 효율적이며, 승차감이 좋은 주행 궤적(Trajectory)을 생성하는 과정이다. 이는 크게 거시적인 경로를 결정하는 **Mission/Behavior Planning**과, 실시간으로 장애물을 회피하며 미시적인 움직임을 만드는 **Motion Planning**으로 나뉜다.


이 단계에서는 고정밀 벡터 맵(Vector Map)에 담긴 도로 네트워크 정보(차선 연결 관계, 차선 종류 등)와 교통 규칙(신호등 위치, 제한 속도 등)을 기반으로, 현재 위치에서 사용자가 지정한 목적지까지의 전체적인 경로(Route)를 계산한다.18 이는 일반적인 내비게이션의 경로 탐색과 유사하다. 또한, 이 경로를 따라가면서 차선 변경이 필요한 시점, 교차로에 진입하는 방법, 정지선에 멈춰야 하는 상황 등 거시적인 행동 계획을 수립한다.


행동 계획에 따라 주어진 짧은 구간 내에서, 인지된 동적 장애물들을 실시간으로 회피하며 실제 차량이 따라갈 구체적인 경로와 속도 프로파일을 생성한다.

- 핵심 알고리즘 1: A* (A-Star) Search

  A*는 그래프 탐색의 일종으로, 시작점에서 목표점까지의 최단 경로를 찾는 데 널리 사용되는 휴리스틱 탐색 알고리즘이다.31 주행 가능한 공간을 격자(Grid)로 표현하고, 각 격자(노드)를 지나는 비용을 평가 함수 $f(n)$으로 계산하여 가장 비용이 낮은 경로를 우선적으로 탐색한다.

  수학적 모델링:

  A*의 핵심은 평가 함수 $f(n)$에 있다 31:
  $$
  f(n) = g(n) + h(n)
  $$

  - $g(n)$: 시작 노드에서 현재 노드 $n$까지 이동하는 데 소요된 실제 비용(cost-so-far). 보통 이동 거리를 의미한다.
  - $h(n)$: 현재 노드 $n$에서 목표 노드까지 도달하는 데 필요할 것으로 예상되는 추정 비용(heuristic). 이 추정치가 실제 비용보다 항상 작거나 같아야(admissible) 최적의 경로를 보장할 수 있다. 보통 목표점까지의 직선 거리(유클리드 거리)가 사용된다.

  A*는 주차장 내 경로 탐색이나, 복잡한 장애물 지형에서 초기 경로를 생성하는 데 유용하게 사용될 수 있다.

- 핵심 알고리즘 2: Lattice Planner

  Lattice Planner는 차량의 동역학적 제약(kinodynamic constraints), 즉 최소 회전 반경이나 최대 가속도 등을 고려하여 현실적으로 주행 가능한 경로들을 생성하는 데 특화된 기법이다.33 차량의 현재 상태(위치, 속도, 방향)에서 출발하여, 미리 정의된 여러 제어 입력(예: 직진, 좌회전, 우회전)을 적용했을 때의 미래 경로들을 샘플링하여 상태 격자(State Lattice)라는 그래프를 만든다. 이 경로들은 클로소이드(clothoid) 곡선과 같이 곡률이 부드럽게 변하는 형태로 생성되어 승차감을 확보한다. 생성된 격자 위에서 장애물과의 충돌 비용, 경로 길이 등을 고려하여 A*와 같은 탐색 알고리즘으로 최적의 지역 경로를 선택한다.


Control 스택은 Planning 스택이 생성한 목표 궤적(시간에 따른 위치, 속도, 가속도 시퀀스)을 차량이 최대한 정확하게 추종하도록 조향각과 가감속 명령을 실시간으로 생성하는 역할을 한다.


PID 제어기는 제어 공학에서 가장 널리 쓰이는 고전적인 피드백 제어기다. 그 원리는 목표값(setpoint)과 현재 측정값(process variable) 사이의 오차(error)를 기반으로 제어량을 계산하는 것이다. 횡방향 제어(Lateral Control)의 경우, 목표 경로나 차선 중심선과 차량 중심 사이의 거리 오차(cross-track error)를 최소화하도록 조향각을 제어한다.

수학적 모델링:

PID 제어기의 출력 $u(t)$(예: 조향각)는 오차 $e(t)$에 대한 세 가지 항의 합으로 결정된다 35:
$$
u(t) = K_p e(t) + K_i \int_{0}^{t} e(\tau) d\tau + K_d \frac{de(t)}{dt}
$$

- **P (Proportional, 비례) 항:** 현재 오차 $e(t)$에 비례하여 제어한다. 오차가 클수록 더 강하게 조향하여 빠르게 오차를 줄이려 한다. 하지만 $K_p$가 너무 크면 목표점을 지나치는 오버슈트(overshoot)와 진동이 발생할 수 있다.
- **I (Integral, 적분) 항:** 과거부터 현재까지의 오차를 누적($\int e(\tau)d\tau$)하여, P 제어만으로는 잡기 힘든 정상 상태 오차(steady-state error)를 없애는 역할을 한다. 예를 들어, 지속적인 외란(바람 등)이 있을 때 이를 상쇄하는 힘을 만들어준다.
- **D (Derivative, 미분) 항:** 오차의 변화율($de(t)/dt$), 즉 미래의 오차를 예측하여 제어한다. 오차가 급격히 줄어들고 있다면(목표점에 빠르게 접근 중이라면) 제어량을 줄여 오버슈트를 방지하는 '제동' 역할을 한다. 이를 통해 시스템의 안정성을 높인다.

PID 제어는 구현이 간단하고 직관적이지만, 차량의 복잡한 동역학이나 제약 조건을 명시적으로 고려하지 못하는 한계가 있다.


MPC는 PID의 한계를 극복하는 현대적인 제어 기법이다. MPC의 핵심 아이디어는 **차량의 동역학 모델을 이용해 미래를 예측하고, 최적의 제어 계획을 수립**하는 것이다.38 MPC는 매 제어 주기마다 다음의 과정을 반복한다:

1. 현재 차량의 상태(위치, 속도 등)를 측정한다.
2. 차량 모델을 기반으로, 앞으로의 짧은 미래 시간(Prediction Horizon, $N_p$) 동안의 차량 움직임을 예측한다.
3. 이 예측 기간 동안, '목표 궤적을 잘 따라가면서', '제어 입력은 너무 과격하지 않고', '차량의 물리적 제약 조건을 만족하는' 최적의 제어 입력 시퀀스(Control Sequence, 예: 앞으로 5초간의 조향각 및 가속도 계획)를 **최적화 문제**를 풀어 찾는다.
4. 계산된 제어 입력 시퀀스 중 가장 첫 번째 값만 실제 차량에 적용한다.
5. 다음 제어 주기가 되면, 1번부터 다시 반복한다.

이러한 '움직이는 예측(Receding Horizon)' 방식 덕분에 MPC는 다가올 커브를 미리 대비해 감속하거나, 조향각 한계나 타이어 마찰력 한계 같은 물리적 제약조건을 명시적으로 고려하여 안정적인 제어를 수행할 수 있다.41

수학적 모델링:

MPC는 다음의 최적화 문제를 푸는 것과 같다 40:
$$
\min_{U_t} J(x_t, U_t) = \sum_{k=0}^{N_p-1} \left( \|x_{t+k|t} - x_{ref,k}\|_{Q}^2 + \|u_{t+k|t}\|_{R}^2 \right) + \|x_{t+N_p|t} - x_{ref,N_p}\|_{P}^2
$$
**Subject to (제약 조건):**

1. $x_{t+k+1|t} = f(x_{t+k|t}, u_{t+k|t})$ (차량 동역학 모델)
2. $u_{min} \le u_{t+k|t} \le u_{max}$ (제어 입력 제약, 예: 최대 조향각, 최대/최소 가속도)
3. $x_{min} \le x_{t+k|t} \le x_{max}$ (상태 변수 제약, 예: 횡가속도 한계로 인한 전복 방지)

- **비용 함수 $J$:** 최소화하려는 목표를 나타낸다.
  - 첫 번째 항($\|x_{t+k|t} - x_{ref,k}\|_{Q}^2$): 예측된 차량 상태 $x_{t+k|t}$가 참조 궤적 $x_{ref,k}$에서 벗어나지 않도록 하는 **궤적 추종 오차** 항이다.
  - 두 번째 항($\|u_{t+k|t}\|_{R}^2$): 과도한 제어 입력 $u_{t+k|t}$ 사용을 억제하여 부드러운 주행과 에너지 효율을 도모하는 **제어 노력** 항이다.
  - $Q, R, P$는 각 항의 중요도를 조절하는 가중치 행렬로, 이 값들을 튜닝하여 차량의 주행 성격(예: 공격적 vs. 보수적)을 결정할 수 있다.

MPC의 채택은 Autoware가 단순한 경로 추종을 넘어, 차량의 동역학적 특성을 깊이 이해하고 안정성과 승차감을 모두 고려하는 고수준의 제어를 지향함을 보여준다.


Vehicle Interface는 소프트웨어의 세계와 하드웨어의 세계를 잇는 다리다. Autoware의 Control 스택이 생성한 추상적인 제어 명령(예: 조향각 5.2도, 가속도 1.0 m/s2)을 실제 차량의 구동계가 알아들을 수 있는 저수준의 CAN(Controller Area Network) 신호로 변환하여 전달하는 역할을 한다. 반대로, 차량의 CAN 버스에 흐르는 현재 속도, RPM, 실제 조향각 등의 상태 정보를 읽어와 Autoware의 다른 모듈들이 사용할 수 있도록 ROS2 토픽으로 발행한다. DataSpeed사의 DBW(Drive-by-Wire) 키트와 같이 상용화된 차량 인터페이스 시스템과의 연동을 위한 드라이버들이 이 스택에 포함된다.44

이처럼 Autoware의 6대 스택은 각자의 전문 분야에서 핵심적인 알고리즘을 수행하며 자율주행의 복잡한 문제를 해결해 나간다. 하지만 이 스택들은 독립적으로 존재하지 않는다. 한 스택의 출력 품질은 곧바로 다음 스택의 입력 품질이 되어 시스템 전체 성능에 연쇄적인 영향을 미친다. 예를 들어, Localization의 정확도가 10cm만 틀어져도 Perception이 탐지한 장애물의 위치가 10cm 어긋나고, 이는 Planning이 충돌을 아슬아슬하게 피하는 경로를 생성하게 만들 수 있다. 따라서 Autoware 시스템 전체의 성능은 이 6개 스택 중 가장 약한 고리(weakest link)에 의해 결정된다고 할 수 있으며, 이것이 바로 고정밀 지도, 정확한 위치추정, 강건한 인지가 자율주행의 선결 과제로 꼽히는 이유다.

------

**표 2: Autoware 핵심 알고리즘 요약**

| 스택             | 알고리즘             | 핵심 원리                                                    | 장점                                         | 단점                                       | Autoware 내 주요 역할                           |
| ---------------- | -------------------- | ------------------------------------------------------------ | -------------------------------------------- | ------------------------------------------ | ----------------------------------------------- |
| **Localization** | NDT                  | 포인트 클라우드를 정규분포의 집합으로 모델링하고 확률적 정합 수행 | 초기 오차에 강건, ICP 대비 빠른 속도         | 복셀 크기에 민감, 비구조적 환경에 취약     | LiDAR 기반의 주 위치 추정 알고리즘              |
|                  | EKF                  | 비선형 시스템을 선형화하여 칼만 필터로 다중 센서 데이터를 융합 | 계산량이 적고 실시간성 우수                  | 강한 비선형 시스템에서 오차 발생 가능      | NDT, IMU, GNSS 등 융합을 통한 최종 위치 보정    |
| **Perception**   | Euclidean Clustering | 지면 제거 후 남은 포인트들을 유클리드 거리 기준으로 그룹화   | 구현이 간단하고 빠름                         | 밀도가 다르거나 겹친 객체 분리 어려움      | LiDAR 포인트 클라우드로부터 객체 분할           |
| **Planning**     | A* Search            | 휴리스틱 함수를 이용해 목표까지의 최단 경로를 그래프 상에서 탐색 | 최적 경로 보장(휴리스틱이 admissible할 경우) | 고차원 공간에서 계산량 폭증                | 전역 경로 계획, 주차장 등 그리드 기반 경로 탐색 |
|                  | Lattice Planner      | 차량 동역학을 만족하는 경로 후보군(격자)을 생성 후 최적 경로 선택 | 동역학적 제약 고려, 부드러운 경로 생성       | 격자 해상도에 따라 경로 품질과 계산량 좌우 | 장애물 회피를 위한 지역 경로(Local Path) 생성   |
| **Control**      | PID                  | 목표와 현재 값의 오차에 비례, 적분, 미분하여 제어 입력 계산  | 구현 간단, 직관적, 튜닝 용이                 | 복잡한 동역학, 제약 조건 고려 못함         | 기본적인 궤적 추종 제어                         |
|                  | MPC                  | 모델 기반으로 미래를 예측하고 비용 함수를 최소화하는 최적 제어 | 제약 조건 명시적 처리, 예측 기반 선제적 제어 | 높은 계산 복잡도, 정확한 모델 필요         | 고성능 궤적 추종 및 차량 안정화 제어            |

------


Autoware의 기술적 아키텍처와 알고리즘을 이해하는 것을 넘어, 이것이 실제 세계에 적용될 때 마주하는 현실적인 문제들, 즉 안전, 생태계, 그리고 상용화 전략을 고찰하는 것은 매우 중요하다.


자율주행 시스템의 가치는 '안전'을 담보할 때 비로소 완성된다. Autoware와 같은 오픈소스 프로젝트 역시 이 엄중한 책임에서 자유로울 수 없다. 자동차 산업의 안전은 크게 두 가지 표준으로 논의된다.

- **기능 안전 (Functional Safety - ISO 26262):** 이 표준은 전자/전기 시스템의 오작동(malfunction)으로 인해 발생할 수 있는 위험을 다루는 데 중점을 둔다. 예를 들어, 소프트웨어 버그나 하드웨어 고장으로 인해 조향 시스템이 오작동하는 경우다. Autoware.Auto와 현재의 Autoware Core가 엄격한 코드 리뷰, 높은 테스트 커버리지, 포괄적인 문서화 등 체계적인 소프트웨어 개발 프로세스를 도입한 주된 이유가 바로 ISO 26262의 요구사항을 만족시키기 위함이다.10
- **의도된 기능의 안전성 (SOTIF - ISO 21448):** SOTIF는 시스템 자체는 설계된 대로 정상 작동하지만, 그 기능의 한계나 예측하지 못한 시나리오로 인해 위험이 발생하는 경우를 다룬다.45 예를 들어, 짙은 안개 속에서 LiDAR 센서의 성능이 저하되어 장애물을 인지하지 못하거나, AI 인지 모델이 이전에 학습한 적 없는 형태의 물체를 잘못 인식하는 경우가 여기에 해당한다. SOTIF는 '알려지지 않은 미지의 영역(unknown unknowns)'을 다루기 때문에 기능 안전보다 더 근본적이고 어려운 도전이다.

SOTIF의 위험을 줄이는 유일한 방법은 방대한 양의 실제 주행 데이터와 시뮬레이션 데이터를 축적하고 분석하여 시스템의 성능 한계를 끊임없이 검증(Verification & Validation, V&V)하는 것이다. 이는 단일 오픈소스 프로젝트가 감당하기 어려운 규모의 과제이며, 대규모 데이터 파이프라인과 MLOps 인프라를 갖춘 상용 플레이어들의 역할이 필수적이다. TIER IV의 `Web.Auto`와 같은 클라우드 기반 DevOps 플랫폼은 바로 이러한 SOTIF 문제를 해결하기 위한 데이터 중심의 검증 환경을 제공하는 것을 목표로 한다.46 Autoware의 미래는 SOTIF 문제를 해결하기 위한 데이터 중심의 생태계 구축에 달려있다고 해도 과언이 아니다.


Autoware의 가장 큰 힘은 기술 그 자체가 아니라, 이를 중심으로 형성된 강력한 생태계에서 나온다. Autoware의 성공은 기술 플랫폼을 기반으로 다양한 참여자들이 각자의 비즈니스 모델을 만들어가는 '플랫폼 비즈니스' 전략에 기반한다.

- **Autoware Foundation (AWF):** 이 생태계의 구심점 역할을 하는 비영리 단체다. AWF는 프로젝트의 장기적인 로드맵을 제시하고, 멤버사 간의 기술 협력을 촉진하며, 인터페이스와 같은 핵심 기술의 표준을 관리한다.4

- **주요 기여 기업 및 역할:**

  - **TIER IV:** Autoware를 탄생시킨 핵심 기업이자 생태계를 이끄는 주도적인 플레이어다.47 TIER IV는 Autoware라는 오픈소스 '운영체제(OS)' 위에, 

    `Pilot.Auto`라는 상용 수준의 소프트웨어 플랫폼과 `Web.Auto`라는 클라우드 기반 개발/운영(DevOps) 플랫폼을 제공한다.46 이를 통해 다른 기업들이 자사의 자율주행 서비스를 더 빠르고 안정적으로 개발할 수 있도록 지원하며, 일본 내 50개 지역에서 대규모 L4 실증 사업을 추진하는 등 기술을 상용화하는 데 앞장서고 있다.49

  - **Leo Drive:** 터키에 본사를 둔 풀스택 자율주행 솔루션 기업으로, Autoware 생태계의 중요한 시스템 통합(System Integrator, SI) 파트너다.51 Leo Drive는 Autoware의 오픈소스 코드를 기반으로 특정 고객(예: 버스 제조사, 물류 회사)의 요구사항에 맞는 맞춤형 하드웨어 및 소프트웨어 통합 솔루션을 제공한다.53 Autoware 공식 문서에 

    `leo-drive`의 저장소 포크(fork) 예시가 등장할 정도로 커뮤니티에 깊이 관여하고 있다.54

  - **기타 파트너:** 이 외에도 ARM(컴퓨팅 플랫폼), ADLINK(엣지 컴퓨팅), Kudan(SLAM 알고리즘), Adastec(상용차 솔루션) 등 하드웨어, 소프트웨어, 서비스 분야의 수많은 기업들이 AWF의 멤버로서 생태계를 구성하며 각자의 전문성을 기여하고 있다.52

이러한 생태계의 힘을 바탕으로 Autoware는 다양한 운영 설계 도메인(ODD)에 성공적으로 적용되고 있다. 초기 목표였던 자율 발렛 파킹(AVP)과 화물 운송을 넘어, Indy Autonomous Challenge나 F1TENTH와 같은 자율주행 레이싱, 그리고 점차 로보버스/셔틀, 로보택시와 같은 공공 도로 서비스로 그 영역을 확장하고 있다.4


Autoware는 강력한 장점을 가지고 있지만, 동시에 명확한 한계와 도전 과제를 안고 있다.

- **장점:**
  - **유연성과 확장성:** 오픈소스이므로 특정 하드웨어나 기술 스택에 종속되지 않는다. 사용자는 필요에 따라 센서 구성을 바꾸거나, 특정 인지 알고리즘을 자체 개발한 모듈로 교체하는 등 자유로운 커스터마이징이 가능하다.6
  - **빠른 개발과 혁신:** 전 세계 수많은 개발자와 연구자들의 집단 지성을 통해 최신 기술이 빠르게 테스트되고 도입된다. 이는 단일 기업이 따라가기 힘든 혁신의 속도를 제공한다.9
  - **비용 효율성:** 자율주행에 필요한 모든 소프트웨어 스택을 처음부터 개발하는 것에 비해, 이미 검증된 기반 위에서 시작함으로써 개발 비용과 시간을 획기적으로 절감할 수 있다.1
- **단점 및 한계:**
  - **높은 학습 곡선:** Autoware는 수백 개의 패키지로 구성된 방대하고 복잡한 시스템이다. ROS2, C++, DDS, 그리고 자율주행의 각 도메인 지식까지 요구하므로 신규 개발자가 전체 시스템을 이해하고 기여하기까지 상당한 시간과 노력이 필요하다.7
  - **실시간성 보장의 어려움:** ROS2와 DDS가 실시간 통신을 위한 기반을 제공하지만, 애플리케이션 레벨에서 모든 컴포넌트의 최악 실행 시간(WCET)을 분석하고 엔드-투-엔드 지연 시간을 보장하는 것은 여전히 매우 어려운 공학적 과제다.12
  - **파편화 위험:** Core/Universe 구조가 표준을 유지하려는 노력이지만, 오픈소스의 특성상 수많은 포크(fork)와 커스텀 버전이 난립할 수 있다. 이는 장기적으로 생태계의 파편화를 유발하고 호환성 문제를 야기할 수 있다.
  - **책임 소재의 불분명성:** 만약 Autoware 기반 차량에서 사고가 발생했을 경우, 그 원인이 Autoware 오픈소스 코드의 근본적인 결함인지, 아니면 이를 가져다 쓴 사용자가 시스템을 잘못 통합하거나 튜닝한 문제인지에 대한 책임 소재를 가리기가 매우 복잡하고 어려울 수 있다.



Autoware는 지난 몇 년간 괄목할 만한 기술적 진화를 이루었다. ROS1의 한계를 극복하고 ROS2의 DDS 기반 아키텍처를 전면적으로 채택함으로써, 상용화에 필수적인 실시간성, 안정성, 확장성의 기틀을 마련했다. NDT, EKF, MPC와 같은 최신 알고리즘들을 통합하여 인지, 판단, 제어 전반에 걸쳐 포괄적인 기능 스택을 제공한다.

무엇보다 중요한 성취는 **Core/Universe라는 지속 가능한 거버넌스 모델을 확립**한 것이다. 이는 상용 수준의 안정성을 요구하는 산업계와 최첨단 기술을 탐구하는 학술 연구 커뮤니티의 상이한 요구를 조화시키는 현명한 해법이다. 안정적인 Core는 기업들에게 신뢰를 주고, 개방적인 Universe는 혁신의 동력을 유지하며 생태계 전체의 선순환 구조를 만들어냈다.


Autoware의 로드맵은 현재의 성과에 안주하지 않고 더욱 복잡하고 도전적인 ODD로 나아가고 있음을 보여준다. 통제된 환경의 자율 발렛 파킹이나 화물 운송을 넘어, 로보택시나 고속도로 주행과 같이 예측 불가능성이 높은 공공 도로 환경을 목표로 기능 고도화가 진행 중이다.4

기술적으로는 차세대 AI 기술을 적극적으로 통합하려는 움직임이 주목된다. TIER IV는 확산 모델(Diffusion Model)을 이용해 인간 운전자와 유사한 자연스러운 주행 궤적을 생성하는 연구를 진행하고 있으며 49, 일부 연구에서는 대규모 언어 모델(LLM)을 활용하여 자연어 명령으로 차량의 주행 행동을 제어하는 새로운 인간-ADS 상호작용 방식을 제안하고 있다.55

이러한 노력들을 통해 Autoware는 자율주행 기술의 개발을 민주화하고, 특정 기업의 독점을 막으며, 사실상의 산업 표준(de facto standard) 플랫폼으로 자리매김할 강력한 잠재력을 가지고 있다.


Autoware에 처음 입문하는 개발자나 연구자라면, 다음과 같은 단계적 접근을 추천한다.

1. **시뮬레이션 환경 구축:** 먼저 공식 문서를 따라 AWSIM과 같은 시뮬레이터를 설치하고, 기본 데모를 실행해보며 전체 시스템의 데이터 흐름을 파악한다.
2. **Universe 패키지 분석 및 수정:** 관심 있는 분야(예: 인지, 계획)의 Universe 패키지 하나를 선택하여 코드를 분석하고, 파라미터를 수정하거나 간단한 기능을 추가해보며 시스템 구조에 익숙해진다. Universe는 상대적으로 기여의 문턱이 낮으므로 좋은 출발점이 될 수 있다.
3. **Core 기여 도전:** 시스템에 대한 이해가 깊어졌다면, Core 모듈의 버그를 수정하거나 성능을 개선하는 작은 기여부터 시작하여 프로젝트에 본격적으로 참여할 수 있다.

앞으로 Autoware 커뮤니티가 함께 풀어가야 할 연구 주제 또한 무궁무진하다. 악천후나 센서 고장 상황에서도 안전을 보장하기 위한 **강건한 SOTIF 검증 방법론** 연구, 이종 센서(LiDAR, 카메라, 레이더)의 장점을 극대화하는 **고도화된 퓨전 알고리즘**, 그리고 V2X 통신을 기반으로 여러 차량이 협력하여 주행하는 **분산 MPC(Distributed MPC)** 와 같은 분야는 Autoware를 한 단계 더 높은 수준의 자율주행으로 이끌 핵심적인 연구 영역이 될 것이다. Autoware는 완성된 제품이 아니라, 전 세계의 지성이 모여 함께 만들어가는 살아있는 프로젝트이며, 그 미래는 지금 이 글을 읽는 당신과 같은 개발자들의 손에 달려 있다.


1. What Are The Advantages and Disadvantages Of Automated Solutions? | SSI SCHAEFER, accessed July 24, 2025, https://www.ssi-schaefer.com/en-ca/what-are-the-advantages-and-disadvantages-of-automated-solutions-1525522
2. autoware.org, accessed July 24, 2025, [https://autoware.org/#:~:text=Autoware%20is%20the%20world's%20leading,range%20of%20vehicles%20and%20applications.](https://autoware.org/#:~:text=Autoware is the world's leading,range of vehicles and applications.)
3. Past, Present and the Future of Autoware, accessed July 24, 2025, https://autoware.org/past-present-and-the-future-of-autoware/
4. Autoware Overview - Autoware, accessed July 24, 2025, https://autoware.org/autoware-overview/
5. ROS 1 vs ROS 2 What are the Biggest Differences? - Model Prime, accessed July 24, 2025, https://www.model-prime.com/blog/ros-1-vs-ros-2-what-are-the-biggest-differences
6. Autonomous Driving System Architecture with Integrated ROS2 and Adaptive AUTOSAR, accessed July 24, 2025, https://www.mdpi.com/2079-9292/13/7/1303
7. Teaching Aspects of ROS 2 and Autonomous Vehicles - MDPI, accessed July 24, 2025, https://www.mdpi.com/2673-4591/79/1/49
8. Changes between ROS 1 and ROS 2 - ROS2 Design, accessed July 24, 2025, http://design.ros2.org/articles/changes.html
9. How is Autoware Core/Universe different from Autoware.AI and Autoware.Auto? - Autoware Documentation, accessed July 24, 2025, https://autowarefoundation.github.io/autoware-documentation/main/design/autoware-concepts/difference-from-ai-and-auto/
10. Migrating a large ROS 1 codebase to ROS 2 - ROSCon 2025, accessed July 24, 2025, https://roscon.ros.org/2019/talks/roscon2019_migrating_a_large_ros_1_codebase_to_ros_2.pdf
11. Autoware.Auto, accessed July 24, 2025, https://autowarefoundation.gitlab.io/autoware.auto/AutowareAuto/
12. Autoware concepts, accessed July 24, 2025, https://autowarefoundation.github.io/autoware-documentation/main/design/autoware-concepts/
13. Autoware - the world's leading open-source software project for autonomous driving - GitHub, accessed July 24, 2025, https://github.com/autowarefoundation/autoware
14. A brief recap on Autoware and the planning component | by Huawei Zhu | Medium, accessed July 24, 2025, https://medium.com/@huawei.zhu/a-brief-recap-on-autoware-and-the-planning-component-f1d8211f40f0
15. autowarefoundation/autoware_universe - GitHub, accessed July 24, 2025, https://github.com/autowarefoundation/autoware_universe
16. How to migrate a ROS project from ROS1 to ROS2 - The Robotics Back-End, accessed July 24, 2025, https://roboticsbackend.com/migrate-ros-project-from-ros1-to-ros2/
17. Architecture overview - Autoware Documentation, accessed July 24, 2025, https://autowarefoundation.github.io/autoware-documentation/release-v1.0_beta/design/autoware-architecture/
18. autoware-documentation/docs/design/autoware-architecture ..., accessed July 24, 2025, https://github.com/autowarefoundation/autoware-documentation/blob/main/docs/design/autoware-architecture/planning/index.md
19. ROS2 topic list - AWSIM Labs Documentation, accessed July 24, 2025, https://autowarefoundation.github.io/AWSIM-Labs/pr-98/Components/ROS2/ROS2TopicList/
20. Node diagram - Autoware Universe Documentation, accessed July 24, 2025, https://autowarefoundation.github.io/autoware-documentation/main/design/autoware-architecture/node-diagram/
21. (PDF) IMPROVEMENT OF LiDAR-SLAM-BASED 3D NDT LOCALIZATION USING FAULT DETECTION AND EXCLUSION ALGORITHM - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/360960460_IMPROVEMENT_OF_LiDAR-SLAM-BASED_3D_NDT_LOCALIZATION_USING_FAULT_DETECTION_AND_EXCLUSION_ALGORITHM
22. Normal distributions transform - Wikipedia, accessed July 24, 2025, https://en.wikipedia.org/wiki/Normal_distributions_transform
23. (PDF) The Normal Distributions Transform: A New Approach to Laser Scan Matching, accessed July 24, 2025, https://www.researchgate.net/publication/4045903_The_Normal_Distributions_Transform_A_New_Approach_to_Laser_Scan_Matching
24. Python sample program of Self-Localization simulation by Extended Kalman Filter, accessed July 24, 2025, https://www.eureka-moments-blog.com/entry/2025/01/14/175655
25. The Extended Kalman Filter: An Interactive Tutorial for Non-Experts - GitHub Pages, accessed July 24, 2025, https://simondlevy.github.io/ekf-tutorial/
26. (PDF) Vehicle state estimation based on Kalman filters - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/335363505_Vehicle_state_estimation_based_on_Kalman_filters
27. Dual extended Kalman filter for vehicle state and parameter estimation - SciSpace, accessed July 24, 2025, https://scispace.com/pdf/dual-extended-kalman-filter-for-vehicle-state-and-parameter-3fqhya7p26.pdf
28. Nonlinear State Estimation (Extended Kalman ... - Robert F. Stengel, accessed July 24, 2025, https://stengel.mycpanel.princeton.edu/MAE546Seminar20.pdf
29. 3D perception stack, accessed July 24, 2025, https://autowarefoundation.gitlab.io/autoware.auto/AutowareAuto/perception-stack-howto.html
30. Implement RTMDet to Perception Pipeline / Issue #7235 / autowarefoundation/autoware_universe - GitHub, accessed July 24, 2025, https://github.com/autowarefoundation/autoware.universe/issues/7235
31. A* Algorithm for Robot Path Planning - Number Analytics, accessed July 24, 2025, https://www.numberanalytics.com/blog/a-star-algorithm-for-robot-path-planning
32. The EBS-A* algorithm: An improved A* algorithm for path planning - PMC, accessed July 24, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC8853577/
33. Mastering Lattice-Based Motion Planning - Number Analytics, accessed July 24, 2025, https://www.numberanalytics.com/blog/lattice-based-motion-planning-ultimate-guide
34. High Performance State Lattice Planning Using Heuristic Look-Up Tables, accessed July 24, 2025, https://rpal.cs.cornell.edu/docs/KneKel_IROS_2006.pdf
35. Mastering PID Controllers in Automotive Systems - Number Analytics, accessed July 24, 2025, https://www.numberanalytics.com/blog/ultimate-guide-pid-controllers-automotive-control-systems
36. PID Control, accessed July 24, 2025, https://www.cds.caltech.edu/~murray/books/AM08/pdf/am06-pid_16Sep06.pdf
37. Racing Vehicle Control Systems using PID Controllers - Game AI Pro, accessed July 24, 2025, http://www.gameaipro.com/GameAIPro/GameAIPro_Chapter40_Racing_Vehicle_Control_Systems_using_PID_Controllers.pdf
38. Model Predictive Control based Trajectory Tracking for Autonomous Vehicles, accessed July 24, 2025, https://www.researchgate.net/publication/389656974_Model_Predictive_Control_based_Trajectory_Tracking_for_Autonomous_Vehicles
39. Model Predictive Control With Learned Vehicle Dynamics for Autonomous Vehicle Path Tracking - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/354588244_Model_Predictive_Control_With_Learned_Vehicle_Dynamics_for_Autonomous_Vehicle_Path_Tracking
40. (PDF) Model predictive control for autonomous ground vehicles: a review - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/353953200_Model_predictive_control_for_autonomous_ground_vehicles_a_review
41. Model Predictive Control for Autonomous Driving Vehicles - MDPI, accessed July 24, 2025, https://www.mdpi.com/2079-9292/10/21/2593
42. Model predictive control | Autonomous Vehicle Systems Class Notes - Fiveable, accessed July 24, 2025, https://library.fiveable.me/autonomous-vehicle-systems/unit-6/model-predictive-control/study-guide/Max59zsNZ9RH6vDI
43. Model-based Predictive Control (MPC), accessed July 24, 2025, https://engineering.purdue.edu/~zak/Second_ed/MPC_handout.pdf
44. Developing an Automated Vehicle Research Platform by Integrating Autoware with the DataSpeed Drive-By-Wire System - SAE International, accessed July 24, 2025, https://www.sae.org/publications/technical-papers/content/2024-01-1981/
45. Safety in ADAS/AD – SOTIF, a Risk-Based Approach | TÜV SÜD, accessed July 24, 2025, https://www.tuvsud.com/en-us/resource-centre/white-papers/safety-in-adas-ad-sotif-a-risk-based-approach
46. TIER IV Autoware Partner Program: Fueling collaboration and innovation in autonomous driving - Medium, accessed July 24, 2025, https://medium.com/tier-iv-tech-blog/tier-iv-autoware-partner-program-fueling-collaboration-and-innovation-in-autonomous-driving-e804f6322a35
47. TIER IV - Autoware, accessed July 24, 2025, https://autoware.org/autowareio/tier-iv/
48. TIER IV Autoware Partner Program: Fueling collaboration and innovation in autonomous driving, accessed July 24, 2025, https://tier4.jp/en/media/detail/?sys_id=1gPJ7yu7hl7KxcbYFxws4H&category=BLOG
49. TIER IV unveils end-to-end architecture for Level 4+ autonomy: To be demonstrated across 50 locations nationwide - PR Newswire, accessed July 24, 2025, https://www.prnewswire.com/news-releases/tier-iv-unveils-end-to-end-architecture-for-level-4-autonomy-to-be-demonstrated-across-50-locations-nationwide-302506092.html
50. Case Studies - Autoware, accessed July 24, 2025, https://autoware.org/case-studies/
51. Leo Drive - Autoware, accessed July 24, 2025, https://autoware.org/autowareio/leo-drive/
52. Ecosystem - Autoware, accessed July 24, 2025, https://autoware.org/ecosystem/
53. Leo Drive | Autonomous Technology, accessed July 24, 2025, https://www.leodrive.ai/
54. Creating Autoware repositories, accessed July 24, 2025, https://autowarefoundation.github.io/autoware-documentation/main/how-to-guides/integrating-autoware/creating-your-autoware-repositories/creating-autoware-repositories/
55. Autoware.Flex: Human-Instructed Dynamically Reconfigurable Autonomous Driving Systems - arXiv, accessed July 24, 2025, https://arxiv.org/html/2412.16265v1

