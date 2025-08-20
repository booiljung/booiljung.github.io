# ROS 2 Jazzy Jalisco

## 1.  ROS 2 Jazzy Jalisco의 등장과 그 의미

### 1.1  10번째 ROS 2 릴리스와 새로운 LTS의 탄생

2024년 5월 23일, 세계 거북이의 날(World Turtle Day)에 맞춰 Robot Operating System (ROS) 2의 10번째 공식 릴리스인 Jazzy Jalisco가 발표되었다.1 이번 릴리스는 단순한 연례 업데이트를 넘어, 2029년 5월까지 5년간의 장기 지원(Long-Term Support, LTS)을 약속한다는 점에서 로보틱스 커뮤니티와 산업계에 중요한 의미를 가진다.2 이는 프로덕션 레벨의 로봇 시스템이나 장기적인 연구 개발 프로젝트를 진행하는 개발자들에게 안정적이고 예측 가능한 소프트웨어 기반을 제공하는 것을 목표로 한다. 이전 LTS 버전인 Humble Hawksbill의 지원이 2027년에 종료되는 것과 비교하면, Jazzy는 2년 더 긴 지원 기간을 보장하여 기술 채택의 안정성을 크게 높였다.4

릴리스의 이름은 멕시코 할리스코(Jalisco) 주에 주로 서식하는 'Jalisco River Turtle'이라는 거북이 종에서 유래했다. 코드네임에 포함된 'Jazzy'라는 단어는 2023년 ROS 개발자 컨퍼런스(ROSCon)가 열렸던 미국 뉴올리언스의 활기차고 음악적인 분위기에서 영감을 받은 것으로 알려져 있다.2 이러한 작명은 ROS 커뮤니티의 창의적이고 협력적인 문화를 상징적으로 보여준다.

### 1.2  핵심 플랫폼 및 지원 환경

ROS 2 Jazzy Jalisco의 가장 큰 구조적 변화 중 하나는 공식 지원 운영체제가 Ubuntu 24.04 (Noble Numbat)로 상향 조정된 점이다.3 이는 개발자들이 최신 리눅스 커널의 기능, 향상된 보안 패치, 그리고 업데이트된 시스템 라이브러리를 기반으로 로봇 애플리케이션을 구축할 수 있음을 의미한다. 지원 플랫폼은 ROS Enhancement Proposal (REP) 2000 문서에 정의된 계층(Tier) 시스템에 따라 체계적으로 관리되고, 각 계층은 지원 수준과 보장 범위를 명확히 한다.6

Jazzy Jalisco의 지원 플랫폼은 다음과 같이 분류된다.

**Table 1: ROS 2 Jazzy Jalisco 지원 플랫폼 (Supported Platforms for ROS 2 Jazzy Jalisco)**

| 계층 (Tier) | 운영체제 (Operating System)         | 아키텍처 (Architecture) |
| ----------- | ----------------------------------- | ----------------------- |
| **Tier 1**  | Ubuntu 24.04 (Noble Numbat)         | `amd64`, `arm64`        |
|             | Windows 10 (Visual Studio 2019)     | `amd64`                 |
| **Tier 2**  | Red Hat Enterprise Linux 9 (RHEL 9) | `amd64`                 |
| **Tier 3**  | macOS                               | `amd64`                 |
|             | Debian Bookworm                     | `amd64`                 |

출처: 3

이러한 플랫폼 지원 전략을 통해 Jazzy는 데스크톱 개발 환경부터 산업용 서버, 임베디드 시스템에 이르기까지 광범위한 하드웨어에서 일관된 개발 경험을 제공하고자 한다.

Jazzy 릴리스의 전략적 방향성은 이중적인 목표를 동시에 추구한다는 점에서 주목할 만하다. 한편으로는 시뮬레이션 환경인 Gazebo와의 통합을 대폭 간소화하여 신규 사용자와 학생들의 진입 장벽을 낮추는 '접근성 확대'를 꾀하고 있다.2 많은 초심자들이 ROS와 시뮬레이터를 연동하는 과정에서 겪는 어려움은 잘 알려진 문제였다. OSRF(Open Source Robotics Foundation)가 직접 Gazebo Harmonic 버전을 공식 권장하고, 

`apt` 명령어로 쉽게 설치할 수 있는 벤더 패키지를 제공하기로 한 결정은 이 문제를 정면으로 해결하려는 의지를 보여준다. 이는 ROS 생태계의 저변을 넓히고 교육적 활용을 촉진하기 위한 명백한 전략이다.

다른 한편으로는 `rmw_zenoh`라는 고성능의 새로운 미들웨어(middleware)를 실험적으로 도입하여 '전문성 강화'를 추구한다.1 로보틱스 기술이 상용화되고 시스템이 복잡해지면서, 기존의 DDS(Data Distribution Service) 기반 미들웨어가 특정 네트워크 환경에서 보이는 성능적 한계나 라이선스 문제를 넘어서려는 요구가 꾸준히 제기되어 왔다. 

`rmw_zenoh`의 등장은 이러한 고성능, 저지연, 대규모 분산 시스템에 대한 산업계의 요구에 부응하기 위한 움직임이다. 특히 Alphabet의 로보틱스 자회사인 Intrinsic 플랫폼과의 연결성이 공식적으로 언급된 점은 3, 이것이 단순한 기술적 실험을 넘어 산업 생태계 확장을 위한 전략적 포석임을 강력하게 시사한다.

결론적으로 Jazzy Jalisco는 '교육 및 입문'과 '산업 및 고성능'이라는, 표면적으로는 상이해 보이는 두 가지 목표를 동시에 끌어안고 있다. 이는 ROS 2가 학술 연구의 단계를 넘어 산업 표준 플랫폼으로 자리매김하려는 성숙 단계에 진입했음을 보여주는 중요한 지표이고, 앞으로의 로보틱스 생태계 발전에 대한 기대를 높이는 부분이다.

------

## 2.  핵심 기능 심층 분석: 개발자 경험의 혁신

ROS 2 Jazzy Jalisco는 단순히 내부 아키텍처를 개선하는 데 그치지 않고, 개발자가 일상적으로 사용하는 도구와 워크플로우를 혁신하여 생산성을 극대화하는 데 중점을 두었다. 시뮬레이션 설정부터 디버깅, 시각화에 이르기까지 개발 과정 전반의 마찰을 줄이려는 체계적인 노력이 엿보인다.

### 2.1  시뮬레이션의 재정의: Gazebo Harmonic과의 통합

과거 ROS 개발자들은 시뮬레이터로 Gazebo를 사용하기 위해 복잡한 설치 과정과 버전 호환성 문제에 직면하는 경우가 많았다. Jazzy는 이러한 불편을 해소하기 위해 Gazebo와의 통합 방식을 근본적으로 개선했다.2

Jazzy부터는 각 ROS 2 릴리스에 공식적으로 권장되고 지원되는 특정 Gazebo 릴리스가 지정된다. Jazzy Jalisco의 경우, 그 파트너는 **Gazebo Harmonic**이다.2 가장 큰 변화는 사용자가 Gazebo를 더 이상 소스 코드로부터 직접 빌드할 필요 없이, 간단한 `apt` 명령어를 통해 ROS 패키지처럼 설치할 수 있는 'Gazebo 벤더 패키지'를 배포한다는 점이다.2 이는 ROS와 Gazebo를 즉시 함께 사용할 수 있게 하여, 특히 로보틱스에 처음 입문하는 사용자들이 겪는 초기 설정의 장벽을 크게 낮춘다.

이러한 변화는 ROS를 특정 시뮬레이터에 종속시키지 않으려는 기존의 철학을 훼손하지 않으면서도, "batteries included(필요한 모든 것이 포함된)" 접근법을 통해 사용자 편의성을 극대화하려는 실용적인 선택이다. 개발자는 여전히 다른 시뮬레이터를 선택할 자유를 가지지만, 권장 조합을 통해 검증된 안정성과 손쉬운 시작을 보장받을 수 있다.2

### 2.2  디버깅의 새로운 지평: 서비스 로깅을 지원하는 ROSBag

ROSBag는 토픽 데이터를 기록하고 재생하여 로봇의 동작을 분석하는 데 필수적인 도구이다. 그러나 이전 버전의 ROS 2에서는 시스템의 핵심적인 상호작용 방식인 서비스(Service) 호출을 기록할 수 없다는 치명적인 한계가 있었다.2 이로 인해 서비스 요청의 실패가 로봇 오작동의 원인일 경우, 그 원인을 추적하는 것이 거의 불가능했다.

Jazzy Jalisco는 이 문제를 정면으로 해결했다. 이제 `rosbag record` 명령어를 통해 서비스 요청과 응답을 토픽과 마찬가지로 bag 파일에 기록할 수 있다.2 기록된 서비스는 재생(replay) 또한 가능하여, 특정 실패 상황을 재현하고 분석하는 데 결정적인 정보를 제공한다. 이는 마치 비행기의 "블랙박스"처럼, 복잡한 로봇 시스템의 동작 이력을 온전히 보존하여 디버깅 능력을 획기적으로 향상시킨다.2

또한, bag 파일의 효율적인 관리를 위한 기능도 추가되었다. 이제 메시지 타입(예: `$sensor_msgs/msg/Image$`)을 기준으로 특정 타입의 모든 토픽을 한 번에 기록하거나, 반대로 특정 타입의 토픽을 기록에서 제외할 수 있다.2 이 기능은 대용량 센서 데이터(카메라, 라이다 등)가 포함된 bag 파일에서 필요한 정보만 선택적으로 저장하여 파일 크기를 줄이고, 동료와의 공유 및 분석을 용이하게 한다.

### 2.3  직관적인 시각화: RViz2의 진화

RViz2는 로봇의 상태와 센서 데이터를 3D로 시각화하는 핵심 도구이다. Jazzy에서는 RViz2의 사용성과 정보 필터링 능력을 개선하는 다양한 기능이 추가되어, 복잡한 시스템을 더 직관적으로 이해할 수 있게 되었다.2

주요 개선 사항은 다음과 같다:

- **정규식(Regex) 기반 TF 필터링:** 복잡한 로봇의 TF(Transform) 트리는 수백 개의 프레임으로 구성될 수 있어, 원하는 정보를 찾는 것이 어려웠다. 이제 프레임 이름을 기반으로 하는 정규식을 사용하여 보고 싶은 프레임 그룹만 선택적으로 표시하거나 숨길 수 있다.2 예를 들어, 로봇의 '왼쪽 팔'과 관련된 모든 프레임(

  `left_arm_*`)만 시각화하는 것이 가능해진다.

- **상태 표시줄의 토픽 주파수:** 모든 구독 토픽의 수신 주파수(Hz)가 RViz2 하단의 상태 표시줄에 자동으로 표시된다.2 이를 통해 각 센서 데이터가 정상적인 주기로 들어오고 있는지 실시간으로 확인할 수 있다.

- **`point_cloud_transport` 시각화:** 압축된 포인트 클라우드 데이터를 전송하는 `point_cloud_transport` 토픽을 RViz2에서 직접 구독하고 시각화할 수 있게 되어, 대용량 3D 데이터를 효율적으로 처리하고 확인할 수 있다.2

- **ROS 1 기능 패리티(Parity) 확보:** `DepthCloud`, `AccelStamped`, `TwistStamped`, `WrenchStamped`, `Effort` 등 ROS 1에서는 지원되었으나 ROS 2에서는 누락되었던 주요 메시지 타입들의 시각화 기능이 추가되어, ROS 1에서 마이그레이션하는 사용자들이 겪는 불편을 해소했다.2

이러한 Gazebo 통합, ROSBag 서비스 로깅, RViz2 필터링, 그리고 뒤이어 설명할 CLI 개선 사항들은 개별적인 기능 업데이트처럼 보이지만, 사실은 '개발 및 디버깅 과정의 마찰 감소'라는 하나의 일관된 목표를 향하고 있다. 로봇 시스템은 수십, 수백 개의 노드가 비동기적으로 상호작용하는 극도로 복잡한 분산 시스템이고, 문제의 원인을 찾는 것은 종종 매우 어려운 과제이다.

Jazzy 이전에는 서비스 호출이 문제의 원인이더라도 ROSBag에 기록되지 않아 추적의 '블랙홀'로 남아있었다.2 서비스 로깅 기능은 이 블랙홀을 메워주는 결정적인 한 수이다. 또한, 복잡한 로봇의 TF 트리를 RViz2에서 볼 때 수많은 프레임으로 인한 정보 과부하 문제는 디버깅의 효율을 떨어뜨리는 주범이었다. 정규식 필터링은 이 문제를 효과적으로 해결한다.3 CLI에서 서비스 클라이언트 수나 액션 타입을 즉시 확인할 수 있는 기능들은 시스템의 현재 상태를 '외부에서' 신속하게 진단할 수 있는 강력한 도구를 제공한다.3

결론적으로, Jazzy의 핵심 기능들은 복잡한 로봇 시스템의 상태를 더욱 투명하게 만들고, 문제 발생 시 원인 규명까지 걸리는 시간을 획기적으로 단축시키려는 체계적인 노력의 산물이다. 이는 ROS 2가 학술 연구 단계를 넘어, 상용 제품 개발의 복잡성과 신뢰성 요구사항을 해결할 수 있는 강력한 프레임워크로 성숙하고 있음을 명확히 보여주는 증거이다.

### 2.4  강력해진 개발 도구: ROS 2 CLI의 새로운 기능들

ROS 2의 커맨드 라인 인터페이스(CLI)는 개발자가 시스템과 상호작용하는 가장 기본적인 수단이다. Jazzy에서는 개발자의 편의성을 높이고 더 많은 제어권을 부여하는 여러 유용한 기능이 추가되었다.3

- **사용자 지정 로그 파일 이름:** `ros2 run` 명령어를 사용할 때 `--log-file-name` 인자를 추가하여 생성되는 로그 파일의 접두사를 직접 지정할 수 있다.3 이를 통해 여러 노드를 동시에 실행하고 테스트할 때 로그 파일을 체계적으로 관리하기가 용이해진다.
- **서비스 클라이언트 수 확인:** `ros2 service info <service_name>` 명령어를 실행하면, 해당 서비스에 현재 연결된 클라이언트의 수를 표시해 준다.3 이는 시스템의 노드들이 정상적으로 연결되어 있는지 빠르게 진단하는 데 유용하다.
- **액션 타입 확인:** `ros2 action type <action_name>` 명령어를 통해 특정 액션의 인터페이스 타입을 즉시 확인할 수 있다.3 이는 복잡한 시스템에서 다양한 액션을 다룰 때 필요한 정보를 빠르게 찾는 데 도움을 준다.
- **토픽 통계 QoS 설정:** 토픽의 통계(메시지 지연, 주파수 등)를 계산할 때, 원래 토픽의 QoS(Quality of Service) 설정과 다른 별도의 QoS 설정을 지정할 수 있게 되었다.3 이를 통해 통계 수집 과정이 실제 데이터 흐름에 미치는 영향을 최소화하면서 정확한 모니터링을 수행할 수 있다.

------

## 3.  아키텍처 변화와 API 업데이트: 시스템의 심장부를 들여다보다

ROS 2 Jazzy는 사용자 경험을 개선하는 상위 레벨 기능뿐만 아니라, 시스템의 근간을 이루는 저수준 아키텍처와 핵심 API에서도 의미 있는 발전을 이루었다. 이러한 변화들은 시스템의 성능, 실시간성, 그리고 내부 상태를 관찰하고 분석하는 '관찰 가능성(Observability)'을 크게 향상시켰다.

### 3.1  클라이언트 라이브러리(RCL/RCLCPP/RCLPY)의 발전

ROS 클라이언트 라이브러리(RCL)는 C, C++, Python 등 다양한 언어로 ROS 노드를 작성하기 위한 기반을 제공한다. Jazzy에서는 이 라이브러리들에 중요한 기능들이 추가되었다.

- **RCL (ROS Client Library):** C 언어 기반의 가장 핵심적인 라이브러리인 `rcl`에는 실시간 시스템 분석을 위한 강력한 도구가 추가되었다.
  - **`rcl_timer_call_with_info` API:** 새로운 타이머 API인 이 함수는 타이머 콜백이 호출될 때, 타이머가 호출되기로 '예상했던 시간'과 '실제로 호출된 시간' 정보를 함께 제공한다.10 이 정보는 실시간 운영체제(RTOS) 환경에서 태스크의 지연(latency)이나 지터(jitter)를 정밀하게 측정하고 분석하는 데 매우 유용하며, 시스템의 실시간 성능을 검증하는 데 필수적이다.
  - **`rcl_wait` 개선:** 노드가 이벤트(메시지 수신, 타이머 만료 등)를 기다리는 핵심 함수인 `rcl_wait`의 타임아웃 계산 로직이 개선되었다. 특히, 시뮬레이션 시간처럼 외부에서 시간이 제어되는 '시간 재정의(time override)'가 활성화된 클럭을 사용하는 타이머의 경우, 불필요한 대기 시간을 계산하지 않도록 하여 효율성을 높였다.10
  - **`~/get_type_description` 서비스 (REP 2016):** 각 노드는 이제 자신이 사용하는 모든 메시지, 서비스, 액션 타입의 정의를 동적으로 제공하는 `~/get_type_description` 서비스를 구현한다.8 이를 통해 introspection 도구나 브릿지 같은 외부 프로그램이 실행 중인 ROS 시스템의 인터페이스 구조를 컴파일 타임 정보 없이도 파악할 수 있게 되어, 시스템의 유연성과 확장성이 크게 향상된다.
- **RCLCPP & RCLPY (C++ 및 Python 클라이언트 라이브러리):**
  - **`ParameterEventHandler`:** C++(`rclcpp`)과 Python(`rclpy`) 양쪽 모두에 `ParameterEventHandler` 클래스가 도입되었다.8 이 클래스는 특정 노드의 파라미터 변경 사항을 이벤트 기반으로 구독하고 처리할 수 있게 해준다. 이전에는 파라미터 변경을 감지하기 위해 주기적으로 폴링(polling)해야 했지만, 이제는 변경이 발생했을 때만 콜백 함수를 실행하는 효율적이고 반응적인 로직을 쉽게 구현할 수 있다.
  - **`message_filters`와 `TypeAdapters`의 결합:** `TypeAdaptation`은 ROS 메시지 타입과 사용자 정의 데이터 구조(예: OpenCV의 `cv::Mat`과 `sensor_msgs/msg/Image`) 간의 변환을 자동화하는 강력한 기능이다. Jazzy에서는 이 기능이 여러 토픽을 동기화하는 `message_filters` 라이브러리와 함께 사용될 수 있도록 지원이 추가되었다.8 이를 통해 개발자는 복잡한 센서 퓨전 로직을 작성할 때, 데이터 변환 코드를 직접 작성하는 수고를 덜고 핵심 알고리즘에 집중할 수 있다.

### 3.2  핵심 인터페이스 및 라이브러리 업데이트

시스템 전반에서 사용되는 공통 인터페이스와 핵심 라이브러리들도 현대적인 요구사항에 맞춰 개선되었다.

- **`common_interfaces`:**
  - **`VelocityStamped` 메시지:** 선속도(linear velocity)와 각속도(angular velocity)를 하나의 헤더와 함께 포함하는 새로운 표준 메시지 타입인 `geometry_msgs/msg/VelocityStamped`가 추가되었다.8 이는 속도 데이터를 좌표계 변환(transform)이 용이한 형태로 표준화하여 코드의 일관성과 재사용성을 높인다.
  - **`ARROW_STRIP` 마커:** RViz 시각화를 위한 `visualization_msgs/msg/Marker`에 `ARROW_STRIP` 타입이 추가되어, 경로(path)나 벡터 필드(vector field)를 연속된 화살표로 표현하는 등 더 풍부한 시각화가 가능해졌다.8
- **`geometry2`:** TF2(Transform Library)를 포함하는 `geometry2` 저장소에서는 API 현대화 작업이 진행되었다.
  - Humble 버전에서 사용 중단(deprecated) 경고가 발생했던 구식 `.h` 확장자 헤더 파일들이 Jazzy에서는 완전히 제거되고, C++ 표준에 맞는 `.hpp` 확장자 헤더로 통일되었다.10 이는 코드의 일관성을 높이고 잠재적인 컴파일 오류를 줄인다.
  - 비동기적으로 TF 변환을 기다리는 `wait_for_transform_async` 함수의 반환 타입이 단순한 `bool` 값의 future에서, 변환 정보 자체를 담고 있는 future로 변경되었다.10 이로 인해 비동기 작업이 완료된 후, 변환 정보를 얻기 위해 별도의 API를 호출할 필요 없이 즉시 결과를 사용할 수 있어 코드가 더 간결하고 효율적으로 변한다.
- **`image_transport`:** 고해상도 이미지 데이터를 효율적으로 처리하기 위한 `image_transport` 라이브러리에 여러 고급 기능이 추가되었다. 구독자가 실제로 필요할 때까지 이미지 압축 해제를 지연시키는 'Lazy subscriber' 지원, 이미지 처리 콜백을 특정 스레드에 할당할 수 있는 '콜백 그룹 설정' 옵션, 그리고 특정 압축 플러그인의 사용을 막는 '허용 목록(allow list)' 기능 등이 포함되어, 리소스가 제한된 시스템에서 이미지 처리 파이프라인을 더욱 세밀하게 최적화할 수 있게 되었다.8

### 3.3  추적과 분석의 정교화: `ros2_tracing`의 개선

`ros2_tracing`은 시스템의 성능 병목을 찾고 메시지 흐름을 분석하기 위한 강력한 도구이다. Jazzy에서는 이 도구의 사용성을 획기적으로 개선하는 중요한 변경이 있었다.

가장 큰 개선점은 메시지가 발행(publication)되어 구독(subscription)되기까지의 종단간(end-to-end) 지연 시간을 추적하기 위해 더 이상 DDS 미들웨어를 소스 코드에서부터 직접 빌드할 필요가 없게 되었다는 점이다.1 이전 버전에서는 메시지에 포함된 `source_timestamp` 값을 얻기 위해 Fast DDS나 Cyclone DDS의 특수한 트레이싱 계측(instrumentation) 기능이 필요했다. 이 기능은 기본 바이너리 배포판에 포함되지 않았기 때문에, 정교한 성능 분석을 원하는 개발자는 미들웨어를 소스에서부터 다시 컴파일해야 하는 큰 부담을 안고 있었다.

Jazzy에서는 이 `source_timestamp` 값이 RMW 구현체 레벨에서 직접 제공되도록 아키텍처가 변경되었다.1 그 결과, 이제 개발자들은 ROS 2 Jazzy를 기본 설치하는 것만으로도 Linux 환경에서 즉시 정교한 종단간 성능 추적 및 분석 기능을 활용할 수 있게 되었다.

이러한 `rcl_timer_call_with_info`, `get_type_description` 서비스, `ros2_tracing`의 개선 등 아키텍처 레벨의 변경점들은 시스템의 '내부 상태를 외부에서 얼마나 잘 관찰하고 분석할 수 있는가'라는 '관찰 가능성(Observability)'을 체계적으로 강화하고 있다. 초기 소프트웨어 프레임워크는 단순히 '동작'하는 것에 집중하지만, 프레임워크가 성숙해짐에 따라 '성능 분석', '디버깅의 용이성', '안정성 검증'이 더욱 중요해진다.

`rcl_timer_call_with_info`는 실시간 시스템에서 "타이머가 과연 제 시간에 실행되었는가?"라는 핵심적인 질문에 대한 정량적인 답을 제공한다.10 이는 단순한 기능 추가가 아니라, 시스템의 시간적 동작을 정밀하게 측정하고 검증할 수 있는 도구를 제공한다는 점에서 의미가 크다. 

`get_type_description` 서비스는 실행 중인 시스템의 구조를 동적으로 파악할 수 있게 하여, 정적인 코드 분석 없이도 시스템을 이해하고 연동하는 지능적인 도구를 만들 수 있는 기반을 마련한다.8

`ros2_tracing`의 개선은 이러한 흐름의 가장 극적인 예시이다. 소스 빌드라는 거대한 기술적 장벽을 제거함으로써, 사실상 모든 사용자가 자신의 복잡한 분산 시스템에서 메시지가 어떻게 흐르고 어디서 지연이 발생하는지를 손쉽게 분석할 수 있는 길을 열어주었다.1 이러한 변화들은 ROS 2가 단순한 '메시지 전달 시스템'을 넘어, 자신의 동작을 정밀하게 측정하고 분석할 수 있는 '관찰 가능한 플랫폼'으로 진화하고 있음을 명확히 보여준다. 이는 산업 현장에서 요구하는 높은 수준의 신뢰성과 성능 검증 요구사항을 만족시키기 위한 필수적인 발전 단계라고 할 수 있다.

------

## 4.  차세대 미들웨어의 부상: RMW와 Zenoh 프로토콜

ROS 2의 핵심 아키텍처 특징 중 하나는 통신을 담당하는 미들웨어 계층을 교체할 수 있다는 점이다. Jazzy Jalisco는 이 RMW(ROS Middleware Interface) 아키텍처 위에서 기존의 DDS 기반 솔루션과 더불어, Zenoh라는 새로운 프로토콜 기반의 구현체를 선보이며 미래 통신 기술의 방향성을 제시하고 있다.

### 4.1  RMW 아키텍처와 기본 미들웨어

ROS 2는 RMW라는 추상화 인터페이스를 통해 특정 통신 기술에 대한 종속성을 제거한다. 이를 통해 개발자는 애플리케이션의 요구사항(예: 실시간성, 네트워크 환경, 라이선스)에 가장 적합한 미들웨어를 선택할 수 있다. ROS 2의 기본 미들웨어는 eProsima사의 Fast DDS이고, 이 외에도 Eclipse Cyclone DDS, RTI Connext DDS 등 여러 DDS 벤더의 구현체를 지원한다.7 사용자는 `RMW_IMPLEMENTATION`이라는 환경 변수를 설정하여 원하는 RMW 구현체를 손쉽게 전환할 수 있다.11

### 4.2  주목받는 새로운 대안: `rmw_zenoh` 심층 탐구

Jazzy에서 가장 주목받는 아키텍처 변화 중 하나는 `rmw_zenoh`의 공식적인 도입이다. `rmw_zenoh`는 Zenoh라는 차세대 프로토콜을 기반으로 하는 새로운 RMW 구현체이다.1 Zenoh는 고성능, 저지연, 높은 확장성을 목표로 설계된 Pub/Sub 및 분산 쿼리 프로토콜로, 특히 대규모 로봇 플릿이나 무선 통신, 리소스가 제한된 임베디드 장치와 같이 기존 DDS가 어려움을 겪는 시나리오에서 강력한 성능을 발휘하도록 설계되었다.13

Jazzy에서 `rmw_zenoh`는 아직 실험적인 단계에 있지만, 차기 LTS 릴리스인 Kilted Kaiju에서는 최고 지원 등급인 Tier 1으로 격상되는 것을 목표로 하고 있다.13 이는 Zenoh가 ROS 2의 미래 통신 스택에서 핵심적인 역할을 맡게 될 것임을 시사한다.

특히 `rmw_zenoh`는 Alphabet의 로보틱스 소프트웨어 자회사인 Intrinsic의 개발자 플랫폼 'Flowstate'와 기존 ROS 코드를 연결하는 핵심 기술로 공식 언급되었다.3 이는 `rmw_zenoh`가 단순한 기술적 대안을 넘어, 거대 IT 기업의 클라우드 및 AI 기반 로보틱스 플랫폼 전략과 깊이 맞물려 있음을 보여주는 중요한 대목이다. Jazzy에서는 사용자들이 이 새로운 기술을 쉽게 테스트하고 피드백을 제공할 수 있도록, `apt`를 통해 설치 가능한 바이너리 패키지를 제공하고 있다.14

### 4.3  `rmw_zenoh` 설정 및 활용 가이드

`rmw_zenoh`를 사용하기 위해서는 기존 DDS 기반 RMW와는 다른 설정 및 실행 방식에 대한 이해가 필요하다.

- **기본 구조:** `rmw_zenoh`는 기본적으로 `rmw_zenohd`라는 중앙 집중식 라우터(router)를 통해 노드 간의 검색(discovery)과 통신을 중개하는 방식을 채택한다.14 이는 네트워크상의 모든 노드가 서로를 직접 찾는 DDS의 멀티캐스트 방식과 대조된다.
- **설치 및 전환:** `sudo apt install ros-jazzy-rmw-zenoh-cpp` 명령어로 패키지를 설치한 후, 셸에서 `export RMW_IMPLEMENTATION=rmw_zenoh_cpp` 명령어를 실행하여 RMW 구현체를 Zenoh로 전환한다.14
- **실행:** Zenoh를 사용하는 ROS 노드를 실행하기 전에, 반드시 별도의 터미널에서 `ros2 run rmw_zenoh_cpp rmw_zenohd` 명령어를 통해 Zenoh 라우터를 먼저 실행해야 한다. 이 라우터가 노드 간의 연결을 중개하는 역할을 한다.14
- **주요 설정 변수:** `rmw_zenoh`는 다양한 환경 변수를 통해 동작을 세밀하게 제어할 수 있다.

**Table 2: `rmw_zenoh` 주요 설정 환경 변수 (Key Configuration Environment Variables for `rmw_zenoh`)**

| 환경 변수 (Environment Variable) | 설명 (Description)                                           |
| -------------------------------- | ------------------------------------------------------------ |
| `ZENOH_ROUTER_CONFIG_URI`        | Zenoh 라우터의 동작을 정의하는 JSON5 설정 파일의 경로를 지정한다. |
| `ZENOH_SESSION_CONFIG_URI`       | 각 ROS 노드(세션)의 동작을 정의하는 JSON5 설정 파일의 경로를 지정한다. |
| `ZENOH_CONFIG_OVERRIDE`          | 설정 파일의 특정 필드 값을 커맨드라인에서 직접 재정의할 때 사용한다. (예: `mode="client"`) |
| `ZENOH_ROUTER_CHECK_ATTEMPTS`    | 노드가 시작될 때 라우터와의 연결을 시도하는 횟수와 대기 방식을 제어한다. (예: `-1`은 확인 생략) |

출처: 18

`rmw_zenoh`가 채택한 중앙 라우터 기반의 하이브리드 접근 방식은 중요한 전략적 의미를 가진다. ROS 2의 기본 DDS 미들웨어는 멀티캐스트(multicast)를 사용하여 네트워크상의 모든 노드가 서로를 직접 발견하는 완전 분산형(P2P) 모델을 사용한다. 이 방식은 별도의 설정 없이 노드들이 서로를 찾을 수 있다는 장점이 있지만, 멀티캐스트 트래픽이 차단되거나 제한되는 기업, 대학, 또는 공용 Wi-Fi 네트워크 환경에서는 노드 간 통신이 실패하는 주된 원인이 되어왔다.12

`rmw_zenoh`는 모든 노드가 중앙 라우터에만 연결하면 되도록 설계하여 이 문제를 근본적으로 회피한다. 노드들은 라우터를 통해 다른 노드들의 존재와 주소를 알게 되므로, 복잡하거나 제한적인 네트워크 환경에서도 안정적인 검색과 연결이 가능해진다. 이는 기능적으로 ROS 1의 ROS Master와 유사한 역할을 수행하지만, 실제 데이터 통신은 노드 간에 직접(P2P) 이루어질 수 있는 효율적인 하이브리드 구조를 가진다. 또한, 여러 대의 라우터를 서로 연결(bridging)하여 물리적으로 분리된 네트워크 세그먼트(예: 로봇 내부 네트워크와 원격 관제 PC 네트워크)를 손쉽게 통합할 수 있다.15 이는 DDS의 Discovery Server와 유사한 기능을 더 간결하고 유연한 방식으로 제공한다.

결론적으로, `rmw_zenoh`는 DDS의 완전 분산 모델이 현실 세계의 네트워크 환경에서 야기하는 문제들을 해결하기 위해 '제어된 중앙 집중화'라는 매우 실용적인 접근법을 채택했다. 이는 ROS 2의 통신 아키텍처에 대한 근본적인 대안을 제시하고, 특히 대규모 시스템이나 복잡한 네트워크 토폴로지를 가진 상용 로봇 시스템에 더 적합한 솔루션이 될 잠재력을 가지고 있다.

### 4.4  Jazzy에서의 `rmw_zenoh` 와이어 포맷 변경 이슈 분석

Jazzy가 LTS 릴리스임에도 불구하고, 출시 이후 `rmw_zenoh` 패키지에서 통신 호환성을 깨는(wire-breaking) 업데이트가 발생하여 커뮤니티의 주목을 받았다. `rmw_zenoh` 0.2.3 버전에서 0.2.5 버전으로 업데이트되면서, 네트워크를 통해 전송되는 데이터의 직렬화 포맷이 변경되어, 이전 버전과 통신할 수 없게 되었다.13

LTS 릴리스의 핵심 가치인 '안정성'을 해칠 수 있는 이러한 결정은 이례적이었다. 그러나 이 변경은 실수가 아닌, 명확한 의도를 가진 '전략적 백포트(backport)'였다. 주된 이유는 `zenoh-pico`를 사용하는 저사양 임베디드 시스템과의 통신 성능을 획기적으로 개선하고, 전반적인 데이터 처리 효율을 높이기 위함이었다.13

이 결정이 가능했던 배경에는 `rmw_zenoh`가 Jazzy에서 아직 Tier 1 등급이 아닌, 커뮤니티의 테스트와 피드백을 받기 위한 개발 단계의 패키지였다는 점이 있다. 개발팀은 장기적인 안정성 유지보다, 이 중요한 테스트 기간 동안 커뮤니티의 피드백을 적극적으로 반영하여 제품을 개선하는 것이 더 큰 이익이라고 판단한 것이다.13

이 사건은 LTS 릴리스의 운영 정책과 ROS 2 생태계의 진화 방식을 보여주는 중요한 사례이다. 이는 LTS의 제1원칙이 '안정성'과 '예측 가능성'임을 재확인하면서도, 미래를 위한 핵심 기술에 대해서는 유연한 전략을 구사할 수 있음을 보여준다. 즉, ROS 2 개발팀은 엄격한 안정성 정책을 유지하는 동시에, 차세대 핵심 기술에 대해서는 커뮤니티와 긴밀하게 협력하며 빠르게 개선해 나가는 실용적인 접근법을 취하고 있다. 이 사건은 사용자들에게 LTS 배포판을 사용하더라도, 개별 패키지의 지원 등급(Tier)과 개발 상태를 반드시 확인하고 시스템에 적용해야 한다는 중요한 교훈을 준다.

------

## 5.  마이그레이션 전략 및 고려사항

새로운 ROS 2 릴리스의 등장은 기존 사용자들에게 마이그레이션이라는 과제를 안겨준다. Jazzy로의 전환은 ROS 1에서의 전환과 이전 LTS인 Humble에서의 업그레이드, 두 가지 주요 시나리오로 나눌 수 있고, 각각 다른 수준의 복잡성과 고려사항을 가진다.

### 5.1  ROS 1에서 ROS 2 Jazzy로의 전환

ROS 1의 마지막 LTS 버전인 Noetic Ninjemys의 공식 지원이 2025년 5월로 종료됨에 따라, ROS 2로의 마이그레이션은 더 이상 선택이 아닌 필수가 되고 있다.19 이 전환은 단순한 API 변경 대응을 넘어, 프레임워크의 근본적인 패러다임 변화를 이해하고 적응하는 과정이다.

**Table 3: ROS 1 vs. ROS 2 핵심 아키텍처 비교 (Core Architectural Comparison: ROS 1 vs. ROS 2)**

| 항목 (Item)     | ROS 1                                   | ROS 2 Jazzy                                  |
| --------------- | --------------------------------------- | -------------------------------------------- |
| **통신 모델**   | 중앙 집중식 Master-Slave (XML-RPC 기반) | 분산형 P2P (DDS/RMW 기반)                    |
| **빌드 시스템** | `catkin`                                | `colcon` (주로 `ament` 사용)                 |
| **런치 파일**   | XML 기반                                | Python 스크립트 기반 (권장)                  |
| **노드 모델**   | 단일 실행 파일에 하나의 노드            | 컴포넌트(Components), 라이프사이클 노드 지원 |
| **파라미터**    | 중앙 집중식 파라미터 서버               | 노드별 독립적인 파라미터 관리                |
| **실시간성**    | 제한적 지원                             | 실시간 제어를 고려한 설계 (QoS, 스케줄링)    |

출처: 20

이러한 근본적인 차이로 인해 마이그레이션은 단순히 코드를 '포팅'하는 것이 아니라, ROS 2의 분산형, 컴포넌트 기반 설계 사상에 맞게 시스템 아키텍처를 '재설계'하는 과정에 가깝다. 점진적인 마이그레이션을 위해 ROS 1 노드와 ROS 2 노드를 연결해주는 `ros1_bridge`가 제공되지만, 이는 어디까지나 과도기적 해결책이다. 대규모 시스템에서는 브릿지로 인한 성능 병목이나 관리의 복잡성이 증가할 수 있으므로, 최종적으로는 모든 노드를 ROS 2 네이티브로 전환하는 것을 목표로 해야 한다.19

### 5.2  Humble Hawksbill에서 Jazzy Jalisco로 업그레이드

이전 LTS 버전인 Humble에서 Jazzy로 업그레이드하는 것은 ROS 1에서의 전환보다는 간단하지만, 여전히 신중한 계획이 필요한 주요 작업이다. 가장 큰 허들은 기반 운영체제가 Ubuntu 22.04에서 24.04로 변경된다는 점이다.22 이는 단순한 패키지 업그레이드가 아니라 OS 자체를 재설치하거나 업그레이드해야 함을 의미한다. 특히 NVIDIA Jetson과 같이 하드웨어 벤더가 특정 OS 버전에 대한 지원을 제공하는 임베디드 시스템의 경우, 이 OS 업그레이드가 기술적으로 불가능하거나 매우 어려운 도전 과제가 될 수 있다.23

따라서 업그레이드를 진행하기 전에는 `/etc/network` (네트워크 설정), `/etc/ros` (ROS 환경 설정), `udev` 규칙, 그리고 사용자의 `colcon_ws` (작업 공간) 등 시스템의 중요 설정과 코드를 철저히 백업하는 것이 필수적이다.22 Clearpath Robotics와 같은 회사들은 이러한 과정을 위해 하드 드라이브 전체 이미지를 백업할 것을 강력히 권장하고 있다.22

코드 레벨에서는 특히 `ros2_control` 프레임워크에서 주목할 만한 API 변경이 있었다. 이는 로봇의 하드웨어 추상화와 제어기 관리를 담당하는 핵심 패키지이므로, 관련 코드를 사용하는 모든 사용자는 마이그레이션 가이드를 숙지하고 코드를 수정해야 한다.24 주요 변경점은 다음과 같다.

- **`diff_drive_controller`:** 속도 명령(`cmd_vel`) 토픽이 이제 시간 정보가 포함된 `TwistStamped` 타입을 요구하며, 가속도/저크 제한 관련 파라미터가 제거되고 `NAN` 값을 사용하도록 변경되었다.
- **`joint_trajectory_controller`:** 궤적 실행과 관련된 여러 파라미터의 기본값이 변경되거나 제거되어, 제어기의 기본 동작이 달라졌다.
- **`pid_controller`:** 안티 와인드업(anti-windup) 로직을 설정하는 방식이 새로운 `antiwindup_strategy` 파라미터로 통합되었다.

이러한 변화들은 마이그레이션이 단순한 코드 변환이 아니라, 시스템의 동작 방식과 배포 전략에 대한 근본적인 재검토를 요구하는 '시스템 재설계' 프로젝트임을 명확히 보여준다. 특히 OS 버전이 묶여 있는 경우, 업그레이드 결정은 기술적 문제를 넘어 비즈니스 및 제품 로드맵과 연계된 중요한 전략적 결정이 된다.

### 5.3  안정성 vs. 신기능: Humble과 Jazzy 사이의 선택 가이드

새로운 프로젝트를 시작하거나 기존 시스템을 업그레이드할 때, 개발자들은 검증된 안정성의 Humble과 최신 기능의 Jazzy 사이에서 선택의 기로에 놓이게 된다.

**Table 4: ROS 2 Humble vs. Jazzy 주요 차이점 비교 (Key Differences Between Humble and Jazzy)**

| 항목 (Item)          | ROS 2 Humble Hawksbill                                       | ROS 2 Jazzy Jalisco                                          |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **지원 종료 (EOL)**  | 2027년 5월                                                   | 2029년 5월                                                   |
| **기반 OS**          | Ubuntu 22.04 (Jammy Jellyfish)                               | Ubuntu 24.04 (Noble Numbat)                                  |
| **주요 기능**        | Type Adaptation, Content Filtered Topics, 초기 ROS 2 성숙기 기능 | 서비스 로깅, Gazebo 통합, `rmw_zenoh`, 향상된 CLI, RViz2 개선 |
| **안정성 및 생태계** | 수년간 검증되어 매우 안정적이며, 튜토리얼과 커뮤니티 자료가 풍부함 | 최신 LTS로 초기 버그 가능성이 있으나, 커뮤니티가 빠르게 이동 중 |

출처: 2

두 배포판 사이의 선택은 프로젝트의 특성과 요구사항에 따라 달라진다.

- **Humble을 추천하는 경우:**
  - 이미 Humble 기반으로 운영 중인 프로젝트의 안정적인 유지보수가 필요한 경우.
  - 사용하는 하드웨어나 임베디드 보드가 Ubuntu 22.04만 지원하는 경우.
  - 극도의 안정성이 최우선이며, 새로운 기능보다는 검증된 생태계가 더 중요한 상용 제품.
- **Jazzy를 추천하는 경우:**
  - 새로운 로봇 프로젝트를 처음부터 시작하는 경우.
  - 서비스 로깅, `rmw_zenoh`, 개선된 시뮬레이션 통합 등 Jazzy의 최신 기능이 필수적인 경우.
  - 5년 이상의 장기적인 기술 지원과 유지보수가 필요한 프로젝트.
  - 최신 하드웨어와 Ubuntu 24.04 환경에서 개발을 진행하는 경우.

커뮤니티의 반응을 보면, 많은 숙련된 개발자들은 당장의 안정성과 풍부한 자료를 이유로 중요한 프로젝트에는 Humble을 유지할 것을 권장하는 반면, 27 새로운 학습자나 장기적인 관점을 가진 프로젝트에는 Jazzy로 시작하는 것이 논리적이라는 의견이 지배적이다.28 결국, 이 선택은 단기적인 안정성과 장기적인 기술 발전 사이의 균형을 맞추는 전략적인 결정이라 할 수 있다.

------

## 6.  결론 및 전망: Jazzy가 열어가는 로보틱스의 미래

ROS 2 Jazzy Jalisco는 단순한 버전 업데이트를 넘어, ROS 2 프로젝트가 중요한 변곡점을 지나고 있음을 알리는 신호탄이다. 이 릴리스는 지난 몇 년간 축적된 기술적 성숙을 바탕으로 개발자의 생산성을 극대화하고, 동시에 미래 로보틱스 기술의 패러다임을 선도하려는 야심 찬 비전을 담고 있다.

### 6.1  Jazzy Jalisco 종합 평가: 성숙과 혁신의 교차점

Jazzy Jalisco는 ROS 2가 학술적 연구 도구를 넘어, 산업 현장에서 요구하는 '사용성', '디버깅 편의성', '고성능'이라는 세 가지 핵심 요소를 모두 갖춘 성숙한 플랫폼으로 진화했음을 명확히 보여주는 이정표이다.

Gazebo와의 손쉬운 통합, 서비스 호출까지 기록하는 ROSBag, 그리고 직관적인 RViz2 개선 사항들은 복잡한 로봇 시스템을 개발하고 디버깅하는 과정의 고통을 크게 줄여주는 실용적인 개선이다. 이러한 변화들은 개발자가 본질적인 로직 개발에 더 집중할 수 있게 하여, 전체적인 개발 사이클을 단축시키고 결과물의 품질을 높이는 데 직접적으로 기여한다.

동시에, `rmw_zenoh`의 실험적 도입은 ROS 2의 미래 통신 아키텍처에 대한 새로운 가능성을 제시한다. 이는 기존 DDS의 한계를 극복하고, 특히 산업용 IoT(IIoT), 대규모 로봇 플릿 운영, 그리고 클라우드 연동과 같은 차세대 로보틱스 애플리케이션에 대한 강력한 비전을 보여준다. Jazzy는 이처럼 현재의 안정성을 공고히 하는 동시에 미래를 향한 혁신의 씨앗을 심는, 성숙과 혁신이 교차하는 지점에 서 있다.

### 6.2  산업계 도입 가속화 전망

Jazzy Jalisco의 출시는 산업계의 ROS 2 도입을 더욱 가속화할 중요한 촉매제가 될 것으로 전망된다. Alphabet의 Intrinsic 플랫폼과의 기술적 연계 3는 ROS 2가 단순한 오픈 소스 프레임워크를 넘어, 글로벌 IT 기업들이 주도하는 상용 로보틱스 솔루션의 핵심 구성 요소로 자리 잡을 가능성을 강력하게 시사한다.

또한, 병원, 공항, 물류 창고 등에서 여러 종류의 로봇 플릿을 통합 관리하는 Open-RMF(Open Robotics Middleware Framework)와의 시너지는 3 ROS 2가 복잡한 환경에서의 이기종 로봇 상호 운용성 표준으로서의 입지를 더욱 강화할 것임을 예고한다. 로봇 팔 제어의 표준으로 자리 잡은 MoveIt 2와 같은 핵심 생태계 프로젝트들이 Jazzy를 새로운 LTS 권장 버전으로 공식 채택함에 따라, 29 이를 기반으로 하는 수많은 산업용 애플리케이션의 Jazzy 도입 역시 자연스럽게 가속화될 것이다.

### 6.3  다음 릴리스 'Kilted Kaiju'를 향하여

ROS 2 프로젝트는 이미 다음 단계를 향해 나아가고 있다. 2025년 5월 출시 예정인 차기 릴리스의 이름은 'Kilted Kaiju'로 발표되었다.2 Jazzy에서 실험적으로 도입되어 가능성을 검증받은 

`rmw_zenoh`는 Kilted에서 최고 지원 등급인 Tier-1으로 격상될 예정이다.13 이는 ROS 2의 기본 통신 스택에 중대한 변화가 일어날 수 있음을 예고하고, 개발자들은 이 새로운 미들웨어의 특성과 장점을 적극적으로 학습하고 대비해야 할 것이다.

Jazzy Jalisco의 성공적인 출시는 ROS 2가 매년 안정적인 릴리스 주기를 유지하고, 30 커뮤니티의 피드백과 산업계의 요구사항에 기민하게 부응하는 신뢰할 수 있는 플랫폼으로 확고히 발전하고 있음을 증명한다. Jazzy가 다져놓은 견고한 기반 위에서, 로보틱스 생태계는 Kilted Kaiju와 함께 또 한 번의 의미 있는 도약을 준비하고 있다.

#### **참고 자료**

1. ROS 2 Jazzy Jalisco Released! - Open Robotics Discourse, accessed July 22, 2025, https://discourse.ros.org/t/ros-2-jazzy-jalisco-released/37862
2. ROS 2 Jazzy Jalisco Released - Open Robotics, accessed July 22, 2025, https://www.openrobotics.org/blog/2024/5/ros-jazzy-jalisco-released
3. OSRF releases ROS 2 Jazzy Jalisco - The Robot Report, accessed July 22, 2025, https://www.therobotreport.com/ros-2-jazzy-jalisco-released-osrf/
4. How to choose between ROS 2 Humble and Jazzy? - Robotics Stack Exchange, accessed July 22, 2025, https://robotics.stackexchange.com/questions/113114/how-to-choose-between-ros-2-humble-and-jazzy
5. How to Install ROS 2 Jazzy - Automatic Addison, accessed July 22, 2025, https://automaticaddison.com/how-to-install-ros-2-jazzy/
6. Installation - ROS 2 Documentation: Jazzy documentation, accessed July 22, 2025, https://docs.ros.org/en/jazzy/Installation.html
7. RMW implementations - ROS 2 Documentation: Jazzy ..., accessed July 22, 2025, https://docs.ros.org/en/jazzy/Installation/RMW-Implementations.html
8. Jazzy Jalisco (jazzy) - ROS 2 Documentation: Rolling documentation, accessed July 22, 2025, https://docs.ros.org/en/rolling/Releases/Release-Jazzy-Jalisco.html
9. Jazzy Jalisco (jazzy) - ROS 2 Documentation: Jazzy documentation, accessed July 22, 2025, https://docs.ros.org/en/jazzy/Releases/Release-Jazzy-Jalisco.html
10. Jazzy Jalisco (jazzy) - ROS 2 Documentation: Humble ..., accessed July 22, 2025, https://docs.ros.org/en/humble/Releases/Release-Jazzy-Jalisco.html
11. Working with multiple ROS 2 middleware implementations - ROS 2 Documentation: Jazzy documentation - the official ROS docs, accessed July 22, 2025, https://docs.ros.org/en/jazzy/How-To-Guides/Working-with-multiple-RMW-implementations.html
12. ROS 2 Middleware (RMW) Configuration - GitHub Pages, accessed July 22, 2025, https://iroboteducation.github.io/create3_docs/setup/xml-config/
13. Wire Breaking Change in rmw_zenoh for ROS 2 Jazzy - Open Robotics Discourse, accessed July 22, 2025, https://discourse.openrobotics.org/t/wire-breaking-change-in-rmw-zenoh-for-ros-2-jazzy/44367
14. Zenoh - ROS 2 Documentation: Jazzy documentation, accessed July 22, 2025, https://docs.ros.org/en/jazzy/Installation/RMW-Implementations/Non-DDS-Implementations/Working-with-Zenoh.html
15. rmw_zenoh binaries for Rolling, Jazzy and Humble - ROS Discourse, accessed July 22, 2025, https://discourse.ros.org/t/rmw-zenoh-binaries-for-rolling-jazzy-and-humble/41395
16. Items to finish before Jazzy release / Issue #110 / ros2/rmw_zenoh - GitHub, accessed July 22, 2025, https://github.com/ros2/rmw_zenoh/issues/110
17. rmw_zenoh binaries for Rolling, Jazzy and Humble - #24 by JEnoch - General - ROS, accessed July 22, 2025, https://discourse.ros.org/t/rmw-zenoh-binaries-for-rolling-jazzy-and-humble/41395/24
18. ros2/rmw_zenoh: RMW for ROS 2 using Zenoh as the ... - GitHub, accessed July 22, 2025, https://github.com/ros2/rmw_zenoh
19. Migrating from ROS 1 to ROS 2: what you need to know (before the robot apocalypse) - Ekumen, accessed July 22, 2025, https://ekumenlabs.com/blog/posts/migrating-from-ros1-to-ros2/
20. ROS 1 vs ROS 2 What are the Biggest Differences? - Model Prime, accessed July 22, 2025, https://www.model-prime.com/blog/ros-1-vs-ros-2-what-are-the-biggest-differences
21. ROS1 vs ROS2, Practical Overview For ROS Developers - The ..., accessed July 22, 2025, https://roboticsbackend.com/ros1-vs-ros2-practical-overview/
22. Upgrading to Jazzy | Clearpath Robotics Documentation, accessed July 22, 2025, https://docs.clearpathrobotics.com/docs/ros/installation/upgrading/
23. ROS 2 Jazzy Debian Packages On Ubuntu 22.04 - Open Robotics Discourse, accessed July 22, 2025, https://discourse.openrobotics.org/t/ros-2-jazzy-debian-packages-on-ubuntu-22-04/41116
24. Migration Guides: Humble to Jazzy - ROS2_Control: Jazzy Jun ..., accessed July 22, 2025, https://control.ros.org/jazzy/doc/ros2_controllers/doc/migration.html
25. Migration Guides - ROS2_Control: Jazzy Jul 2025 documentation, accessed July 22, 2025, https://control.ros.org/jazzy/doc/migration/migration.html
26. Humble Hawksbill (humble) - ROS 2 Documentation: Jazzy documentation, accessed July 22, 2025, https://docs.ros.org/en/jazzy/Releases/Release-Humble-Hawksbill.html
27. ROS2 , Ubuntu version : r/ROS - Reddit, accessed July 22, 2025, https://www.reddit.com/r/ROS/comments/1eg4wu2/ros2_ubuntu_version/
28. Install ROS2 Jazzy Jalisco - New 2025 Tutorial - YouTube, accessed July 22, 2025, https://www.youtube.com/watch?v=aQeirEM59zg
29. New MoveIt LTS release for ROS 2 Jazzy! - PickNik Robotics, accessed July 22, 2025, https://picknik.ai/release/jazzy/rolling/2024/06/30/New-MoveIt-LTS-release-for-ROS-2-Jazzy.html
30. Distributions - ROS 2 Documentation: Jazzy documentation, accessed July 22, 2025, https://docs.ros.org/en/jazzy/Releases.html

