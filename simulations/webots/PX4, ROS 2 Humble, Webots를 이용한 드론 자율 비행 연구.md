# PX4, ROS 2 Humble, Webots를 이용한 드론 자율 비행 연구

### 0.1 전문가 소개

본 보고서는 항공우주 시스템, 실시간 임베디드 시스템, 로보틱스 미들웨어 분야에서 15년 이상의 경력을 보유한 수석 시스템 통합 엔지니어가 작성하였습니다. 주요 전문 분야는 PX4 비행 제어 스택의 아키텍처 분석, ROS(Robot Operating System) 기반 자율 시스템 설계, 그리고 복잡한 하드웨어-소프트웨어 시스템의 시뮬레이션 및 검증입니다. 다수의 상용 드론 플랫폼과 학술 연구 프로젝트에서 기술 자문 및 개발을 주도한 경험을 바탕으로, 본 보고서는 이론적 분석과 실질적인 구현 전략을 결합하여 심도 있는 통찰을 제공하고자 합니다.

------

## Executive Summary

본 보고서는 PX4 펌웨어 버전 1.15.4, ROS 2 Humble Hawksbill, 그리고 Webots 시뮬레이터를 활용한 드론 자율 비행 연구의 기술적 타당성과 구현 전략을 종합적으로 분석합니다. 결론부터 말하자면, **PX4 v1.15와 ROS 2 Humble을 조합한 핵심 스택은 드론 자율 비행 연구에 있어 기술적으로 매우 타당하며, 현재 PX4 개발팀이 공식적으로 권장하는 최신 표준 개발 환경**입니다.1 이 조합은 고성능, 저지연 통신을 가능하게 하는 uXRCE-DDS 브릿지를 기반으로 하여, 이전 세대인 ROS 1/MAVROS 스택을 뛰어넘는 긴밀한 통합과 실시간 제어 능력을 제공합니다.1

그러나 사용자가 고려하는 **Webots 시뮬레이터의 선택은 중대한 전략적 검토가 필요합니다.** Webots 자체는 ROS 2를 훌륭하게 지원하지만 5, PX4 비행 스택과의 직접적이고 공식적인 통합 경로는 현재 부재합니다.7 이는 업계 표준으로 자리 잡은 Gazebo 시뮬레이터와의 통합 경로와 극명한 대조를 이룹니다. Gazebo는 PX4와 매우 긴밀하게 통합되어 있어 단일 명령어로 시뮬레이션 환경 전체를 구동할 수 있으며, 방대한 양의 튜토리얼, 예제 코드, 커뮤니티 지원을 제공합니다.8

Webots를 사용하기 위해서는 PX4 SITL(Software-in-the-Loop)과 Webots 시뮬레이터 간의 센서 데이터 및 액추에이터 명령을 중계하는 '커스텀 브릿지 노드'를 연구자가 직접 개발해야 합니다. 이는 상당한 시스템 통합 노력과 시간을 요구하며, 연구 프로젝트의 본질적인 목표(자율 비행 알고리즘 개발)에서 벗어나 초기 단계에 예상치 못한 기술적 장벽과 지연을 초래할 높은 위험을 내포하고 있습니다.

따라서 본 보고서는 다음과 같은 핵심 전략을 제안합니다. **모든 드론 자율 비행 연구 프로젝트는 검증되고 방대한 자료가 축적된 PX4/ROS 2/Gazebo 스택으로 시작할 것을 강력히 권고합니다.** 이 표준 경로를 통해 연구자는 환경 구축의 어려움을 최소화하고 핵심 연구에 즉시 집중할 수 있습니다. Webots의 사용은, 해당 시뮬레이터가 제공하는 특정 기능(예: 고유한 물리 엔진 모델, 특수 센서)이 연구 목표 달성에 반드시 필요한 비대체 요소임이 명확해졌을 때, 별도의 통합 프로젝트로 계획하여 신중하게 접근해야 합니다.

본 보고서는 이러한 결론에 도달하기까지의 과정을 상세히 기술합니다. 먼저 PX4-ROS 2의 핵심 아키텍처인 uXRCE-DDS 브릿지를 심층 분석하고, 시뮬레이션 환경 선택의 중요성을 Gazebo와 Webots의 비교를 통해 설명합니다. 이후, 표준 Gazebo 환경 구축을 위한 실용적인 가이드와 Webots 통합을 위한 이론적 워크플로우를 제시합니다. 마지막으로, 이 기술 스택을 통해 개척할 수 있는 최신 연구 분야를 조망하고, 개발 과정에서 마주할 수 있는 주요 문제점과 해결 방안을 제시하여 연구자가 성공적인 프로젝트를 수행할 수 있도록 지원할 것입니다.

------

## 1.  실행 가능성 및 아키텍처 프레임워크

이 섹션에서는 PX4와 ROS 2를 중심으로 한 개발 스택의 기술적 타당성을 평가하고, 두 시스템이 어떻게 상호작용하는지에 대한 근본적인 아키텍처를 심층적으로 분석합니다. 이는 성공적인 자율 비행 연구를 위한 기술적 기반을 이해하는 데 필수적입니다.

### 1.1  핵심 스택(PX4 & ROS 2)의 타당성 평가

PX4 버전 1.15와 ROS 2 Humble의 조합은 단순한 가능성을 넘어, 현재 드론 개발 생태계에서 가장 안정적이고 미래 지향적인 표준으로 자리 잡고 있습니다. PX4 개발팀은 공식 문서를 통해 ROS 2 Humble LTS(Long-Term Support)를 Ubuntu 22.04 환경에서 사용하는 것을 명시적으로 권장하고 있으며, 이는 장기적인 기술 지원과 안정성을 보장받을 수 있음을 의미합니다.1

이러한 호환성의 기술적 핵심은 `px4_msgs` 패키지에 있습니다.11 이 ROS 2 패키지는 PX4 내부에서 사용하는 uORB(micro Object Request Broker) 메시지들을 ROS 2 환경에서 사용할 수 있는 `.msg` 파일 형태로 제공합니다. `px4_msgs` 저장소의 브랜치 관리 정책을 살펴보면, `release/1.15` 브랜치가 ROS 2 Humble과의 호환성을 명시적으로 지원하고 있음을 확인할 수 있습니다.11 이는 PX4 펌웨어와 ROS 2 노드 간의 데이터 구조가 일치하여 원활한 통신이 가능함을 보증하는 결정적인 증거입니다.

또한, PX4 v1.14부터 도입된 **uXRCE-DDS(Micro XRCE-DDS) 미들웨어**는 이 스택의 성능과 안정성을 한 차원 높였습니다.1 이전 버전(v1.13)에서 사용되던 Fast-RTPS 브릿지 방식에서 벗어나, OMG(Object Management Group)의 표준 프로토콜인 DDS-XRCE를 채택함으로써 더욱 강력하고 표준화된 통신 아키텍처를 구축하게 되었습니다.12 이 아키텍처는 고성능 컴퓨팅이 요구되는 자율 비행 연구에 필수적인 저지연, 고대역폭 통신을 실현하는 기반이 됩니다.

### 1.2  최신 PX4-ROS 2 아키텍처: uXRCE-DDS 브릿지 심층 분석

현대적인 PX4-ROS 2 연동 방식의 핵심은 uXRCE-DDS 브릿지이며, 이는 클라이언트-에이전트(Client-Agent) 모델을 기반으로 합니다. 이 구조를 이해하는 것은 시스템의 동작 원리를 파악하고 문제를 진단하는 데 매우 중요합니다.

- **클라이언트-에이전트 모델**: 이 아키텍처는 두 개의 독립적인 구성요소로 이루어집니다. 하나는 비행 컨트롤러(FCU) 또는 시뮬레이션 환경 내에서 실행되는 경량의 `uxrce_dds_client`이고, 다른 하나는 컴패니언 컴퓨터(연구자의 워크스테이션 등)에서 실행되는 `MicroXRCEAgent`입니다.1
- **에이전트의 프록시(Proxy) 역할**: `MicroXRCEAgent`는 클라이언트의 대리인 역할을 수행합니다. 리소스가 제한된 FCU의 클라이언트는 전체 DDS 네트워크의 복잡한 피어-투-피어(peer-to-peer) 통신이나 서비스 디스커버리에 직접 참여하는 대신, 오직 에이전트와 단일 연결만 유지합니다. 에이전트는 클라이언트로부터 받은 데이터를 전체 DDS/ROS 2 네트워크(이를 "글로벌 DDS 데이터 공간"이라 칭함)에 전파하고, 반대로 네트워크의 데이터를 수신하여 클라이언트에게 전달하는 중개자 역할을 합니다.13 이 구조는 FCU의 부하를 크게 줄여 비행 안정성에 기여합니다.
- **통신 채널**: 이 브릿지는 다양한 물리적, 논리적 통신 채널을 통해 동작할 수 있습니다. SITL(Software-in-the-Loop) 시뮬레이션 환경에서는 주로 로컬호스트(localhost)를 통한 UDP 통신이 사용되며, 실제 드론 하드웨어에서는 UART 시리얼 통신이나 Wi-Fi를 통한 UDP 통신이 일반적으로 사용됩니다.12

이 새로운 아키텍처는 ROS 1 시대의 MAVLink 프로토콜과 MAVROS 패키지를 사용하던 방식과 근본적인 차이를 보입니다.17 MAVROS는 MAVLink라는 텔레메트리 중심의 프로토콜을 ROS 토픽/서비스로 변환해주는 '번역기'에 가까웠습니다. 반면, uXRCE-DDS 브릿지는 PX4의 내부 메시지 버스인 uORB를 ROS 2의 네이티브 통신 시스템인 DDS와 직접 연결하는 '직통로'를 제공합니다. 이 변화는 단순한 기술 업데이트를 넘어, PX4가 더 큰 로봇 시스템의 일부로 긴밀하게 통합될 수 있도록 하는 전략적 전환을 의미합니다. 이 "깊은 통합(deep integration)" 덕분에 개발자는 이전에는 펌웨어 수정으로만 가능했던 수준의 고주파수 센서 데이터 접근과 실시간 제어 루프 구현을 컴패니언 컴퓨터의 ROS 2 노드에서 직접 수행할 수 있게 되었습니다.1 이는 복잡한 자율 비행 연구의 진입 장벽을 극적으로 낮추는 효과를 가져왔습니다.

### 1.3  핵심 소프트웨어 구성요소와 역할

성공적인 개발 환경을 구축하기 위해서는 각 소프트웨어 구성요소의 정확한 역할을 이해해야 합니다.

- **PX4-Autopilot**: 비행 제어 로직, 상태 추정, 제어기 등을 포함하는 핵심 비행 스택입니다. SITL 환경에서는 이 전체 스택이 데스크톱의 프로세스로 실행됩니다. ROS 2 연동 관점에서 중요한 점은, ROS 2가 PX4에 갖는 유일한 '하드' 의존성은 메시지 정의(`px4_msgs`)이지만, 자율 비행을 시뮬레이션하기 위해서는 완전한 SITL 실행 환경이 필요하다는 것입니다.1
- **`px4_msgs`**: PX4의 uORB 토픽 정의를 그대로 복사해 온 ROS 2 메시지 패키지입니다. 이 패키지는 PX4와 ROS 2 간의 '공통 언어' 역할을 합니다. 따라서 컴패니언 컴퓨터의 ROS 2 워크스페이스를 빌드할 때 사용된 `px4_msgs` 버전과, PX4 펌웨어를 빌드할 때 사용된 uORB 메시지 정의 버전이 반드시 일치해야 합니다. 이 버전 불일치는 통신 실패의 가장 흔한 원인 중 하나입니다.1
- **`px4_ros_com`**: PX4 v1.14 이후, 이 패키지의 역할은 크게 축소되었습니다. 과거에는 통신 브릿지의 일부 기능을 담당했지만, 현재는 주로 오프보드 제어 예제(offboard control listener), 좌표계 변환 유틸리티 함수 등 유용한 **예제 코드 모음**으로 활용됩니다.2 초심자는 이 패키지의 예제 코드를 분석하며 PX4-ROS 2 통신 방법을 학습할 수 있습니다.
- **Micro-XRCE-DDS Agent**: 컴패니언 컴퓨터에서 실행되는 독립적인 애플리케이션으로, PX4 클라이언트와 DDS 네트워크를 연결하는 브릿지 그 자체입니다. 소스 코드를 직접 빌드하거나, 바이너리 파일을 다운로드하거나, 우분투의 Snap 패키지로 간편하게 설치할 수 있습니다.12

### 1.4  버전 호환성: 수많은 실패의 근원

PX4-ROS 2 생태계에서 발생하는 문제의 상당수는 코드의 논리적 오류가 아닌, 구성요소 간의 버전 불일치에서 비롯됩니다. 특히 `px4_msgs` 패키지의 버전은 매우 중요합니다.

`px4_msgs` GitHub 저장소에서 제공하는 호환성 표에 따르면, PX4 v1.15 버전을 사용하는 경우, ROS 2 워크스페이스에서는 반드시 `release/1.15` 브랜치를 클론하여 사용해야 합니다.11 만약 다른 버전의 `px4_msgs`를 사용하면, ROS 2 노드가 메시지를 수신할 때 데이터를 올바르게 해석하지 못해 "Fast CDR exception deserializing message"와 같은 치명적인 오류가 발생합니다.22 이 오류는 uXRCE-DDS 에이전트가 PX4 클라이언트로부터 받은 데이터 스트림을 ROS 2 메시지 형식으로 변환(deserialization)하려 했으나, 데이터 구조의 정의가 일치하지 않아 실패했음을 의미합니다.

PX4 v1.16부터는 메시지 버전 관리 기능과 이를 중재하는 '메시지 변환 노드(Message Translation Node)'가 도입되어 버전 불일치 문제를 일부 완화했지만 1, 사용자가 질의한 v1.15 환경에서는 이러한 기능이 없으므로, 모든 구성요소의 버전을 정확하게 맞추는 것이 성공의 전제 조건입니다.

다음 표는 개발자가 ROS 1/MAVROS 스택에서 ROS 2/uXRCE-DDS 스택으로 전환할 때 겪는 혼란을 줄이고, 명확한 개념 모델을 제공하기 위해 두 아키텍처를 비교한 것입니다.

**표 1: PX4-ROS 통신 브릿지 아키텍처 진화 과정 비교**

| 특성 (Characteristic) | ROS 1 스택 (MAVROS)                       | ROS 2 스택 (uXRCE-DDS Bridge)                            |
| --------------------- | ----------------------------------------- | -------------------------------------------------------- |
| **핵심 프로토콜**     | MAVLink 17                                | DDS/RTPS 1                                               |
| **브릿지 노드**       | `mavros` 노드 19                          | `MicroXRCEAgent` 애플리케이션 1                          |
| **핵심 추상화**       | ROS 서비스 및 토픽 (`/mavros/...`) 17     | uORB 토픽 직접 미러링 (`/fmu/in/...`, `/fmu/out/...`) 17 |
| **Arming 명령 예시**  | `rosservice call /mavros/cmd/arming True` | `ros2 topic pub /fmu/in/vehicle_command...` 17           |
| **성능 특성**         | 텔레메트리 중심, 상대적 저주파            | 고주파, 저지연, 제어 루프에 적합 23                      |
| **실시간성**          | 제한적                                    | DDS QoS 프로파일을 통한 네이티브 지원 24                 |
| **주요 사용 사례**    | GCS 통신, 고수준 명령 전달 18             | 컴패니언 컴퓨터와의 긴밀한 통합, 고급 자율성 구현 4      |

이러한 아키텍처의 변화는 PX4가 단순한 취미용 또는 학술용 비행 컨트롤러를 넘어, 신뢰성, 보안, 실시간성이 중요한 상업 및 산업용 로보틱스 시스템의 핵심 구성요소로 자리매김하려는 전략적 방향성을 명확히 보여줍니다. DDS라는 산업 표준 미들웨어를 채택함으로써, PX4는 더 넓은 로보틱스 생태계와의 통합을 용이하게 하고, 플랫폼 자체의 성숙도를 한 단계 끌어올렸습니다.

------

## 2.  시뮬레이션 환경: Gazebo와 Webots의 비판적 비교 분석

자율 비행 연구에서 시뮬레이션 환경의 선택은 프로젝트의 성패를 좌우할 수 있는 중대한 결정입니다. 이 섹션에서는 사용자가 질의한 Webots과 사실상의 표준(de facto standard)인 Gazebo를 비교 분석하여, 각 선택이 가져올 기회와 위험을 명확히 제시합니다.

### 2.1  Gazebo: PX4 SITL의 표준 시뮬레이터

Gazebo(최신 버전은 Ignition Gazebo 또는 줄여서 Gz로 불림)는 PX4 SITL 환경과 가장 긴밀하게 통합된 시뮬레이터입니다. 이는 단순한 호환성을 넘어, 개발 워크플로우 자체가 Gazebo를 중심으로 설계되었음을 의미합니다.

- **긴밀한 통합(Tight Integration)**: PX4와 Gazebo의 통합은 매우 매끄럽습니다. PX4 소스 코드의 루트 디렉토리에서 `make px4_sitl gz_x500`과 같은 단일 명령어를 실행하는 것만으로 PX4 SITL 프로세스와 Gazebo 시뮬레이터가 동시에 실행되고, 두 시스템은 자동으로 연결됩니다.1 이 과정에서 PX4는 Gazebo로부터 가상의 센서 데이터를 받고, Gazebo 내의 드론 모델에 모터 제어 신호를 보내는 물리 브릿지가 자동으로 활성화됩니다.28
- **성숙한 생태계(Ecosystem Maturity)**: PX4/ROS 2 시뮬레이션과 관련된 거의 모든 공식 문서, 튜토리얼, 커뮤니티 프로젝트, 포럼 질문과 답변이 Gazebo를 기반으로 작성되었습니다.8 이는 개발자가 문제에 직면했을 때 참고할 수 있는 방대한 자료가 존재하며, 이미 검증된 솔루션을 쉽게 찾을 수 있음을 의미합니다.
- **성능 및 기능**: Gazebo는 강력한 3D 물리 엔진과 고품질 렌더링, 다양한 센서 모델(카메라, LiDAR, IMU 등)을 제공하여 컴퓨터 비전이나 장애물 회피와 같은 복잡한 알고리즘을 테스트하기에 매우 적합합니다.9 또한, 그래픽 인터페이스(UI) 없이 실행되는 '헤드리스 모드(Headless Mode)'를 지원하여, CI/CD(지속적 통합/지속적 배포) 파이프라인과 같이 리소스가 제한된 환경에서도 빠르고 효율적인 시뮬레이션이 가능합니다.9
- **커뮤니티의 평가**: 반면, Gazebo는 상대적으로 학습 곡선이 가파르다는 평가를 받습니다. 월드나 로봇 모델을 정의하는 데 사용되는 SDF(Simulation Description Format)는 XML 기반으로, 직관적이지 않고 장황하게 느껴질 수 있습니다.31 또한 오랜 역사와 최근의 'Ignition'에서 다시 'Gazebo'로의 명칭 변경 과정에서 최신 정보를 찾기 어렵게 만드는 정보 파편화 문제가 존재하기도 합니다.32

### 2.2  Webots: 실행 가능하지만 도전적인 대안

Webots는 그 자체로 매우 훌륭하고 ROS 2를 완벽하게 지원하는 시뮬레이터이지만, PX4와의 연동 측면에서는 상당한 기술적 과제를 안고 있습니다.

- **우수한 ROS 2 지원**: Webots는 `webots_ros2`라는 공식 패키지를 통해 ROS 2와 완벽하게 통합됩니다.5 이 패키지는 Webots에서 정의된 로봇 모델의 센서와 액추에이터를 자동으로 ROS 2 토픽, 서비스, 액션으로 노출시켜주는 강력한 기능을 제공합니다.
- **통합의 공백(The Integration Gap)**: **결정적으로, PX4 SITL과 Webots를 직접 연동하는 공식적인 가이드나 검증된 예제는 현재 존재하지 않습니다**.7 이것이 Webots 선택의 가장 큰 위험 요소입니다. PX4의 대안으로 볼 수 있는 ArduPilot 프로젝트는 Webots와의 통합 가이드를 제공하고 있지만 35, 이는 개념적 참고 자료일 뿐 PX4에 직접 적용할 수는 없습니다.
- **사용자 친화성**: Webots는 일반적으로 Gazebo보다 훨씬 사용자 친화적이라고 평가받습니다. 세련된 GUI, 직관적인 월드 및 로봇 생성 방식은 초심자가 시뮬레이션 환경을 구축하는 데 드는 시간을 크게 단축시켜 줍니다.31
- **물리 엔진**: Webots는 ODE(Open Dynamics Engine) 물리 엔진을 커스터마이징하여 사용합니다. 일부 사용자들은 특히 물체 간의 접촉이나 마찰과 같은 동역학 시뮬레이션에서 Gazebo의 기본 ODE 구현보다 더 안정적이고 정확한 결과를 보여준다고 보고합니다.38

### 2.3  PX4 SITL과 Webots 통합을 위한 제안 전략

공식적인 가이드가 없는 상황에서, Webots를 사용하고자 하는 연구자를 위해 이론적인 통합 워크플로우를 제시합니다. 이는 상당한 커스텀 개발이 필요함을 전제로 합니다.

1. **시스템 분리 (Decouple the System)**: 전체 시스템을 독립적으로 실행되고 ROS 2 네트워크를 통해 통신하는 세 개의 컴포넌트로 간주해야 합니다.
   - `PX4 SITL` 프로세스
   - `MicroXRCEAgent` 브릿지 프로세스
   - `Webots` 시뮬레이터와 `webots_ros2_driver` 노드
2. **PX4 헤드리스 실행 (Run PX4 Headless)**: Gazebo GUI 없이 PX4 SITL 프로세스만 단독으로 실행합니다 (`make px4_sitl_default none`). 이 상태에서 PX4는 시뮬레이터로부터 센서 데이터를 받기 위해 대기하고, 모터 명령을 출력할 준비를 합니다.
3. **DDS 에이전트 실행 (Run the DDS Agent)**: `MicroXRCEAgent`를 실행하여 PX4의 uORB 토픽을 ROS 2 네트워크(`/fmu/...` 토픽)에 브릿징합니다.25
4. **Webots 설정 (Configure Webots)**: Webots에서 드론 모델을 생성합니다. 이때 로봇의 컨트롤러를 단순 스크립트가 아닌 `<extern>` 컨트롤러로 지정해야 합니다. 이는 Webots에게 외부의 ROS 2 노드가 이 로봇을 제어할 것임을 알리는 설정입니다.6
5. **'접착제' 노드 개발 (Create the "Glue" Node)**: 이것이 가장 핵심적인 커스텀 개발 부분입니다. 연구자는 다음 두 가지 역할을 동시에 수행하는 별도의 ROS 2 노드를 작성해야 합니다.
   - **PX4 -->> Webots**: PX4가 출력하는 액추에이터 명령 토픽(예: `/fmu/out/actuator_outputs`)을 구독(subscribe)하고, 이 값을 Webots 드론 모델의 모터(프로펠러)를 구동하는 명령으로 변환하여 퍼블리시(publish)합니다.
   - **Webots -->> PX4**: `webots_ros2_driver`가 퍼블리시하는 Webots 내 가상 센서 데이터(예: IMU, GPS, 기압계 등)를 구독하고, 이 데이터를 PX4가 입력으로 받는 uORB 토픽(예: `/fmu/in/sensor_combined`, `/fmu/in/vehicle_gps_position` 등)의 형식에 맞게 변환하여 퍼블리시합니다.

이 '접착제' 노드는 사실상 Gazebo-PX4 간의 네이티브 물리 브릿지를 ROS 2를 통해 수동으로 구현하는 것과 같습니다. 이 작업은 좌표계 변환, 타임스탬프 동기화, 데이터 형식 변환 등 복잡하고 오류가 발생하기 쉬운 저수준(low-level) 처리를 포함합니다.

이러한 분석은 시뮬레이터 선택이 단순한 도구의 선호도 문제가 아니라, 프로젝트의 역할 정의와 직결되는 전략적 결정임을 시사합니다. Gazebo를 선택하면 연구자는 **'응용 프로그램 개발자'**가 되어 자율 비행 알고리즘 개발에 즉시 착수할 수 있습니다. 반면, Webots를 선택하면 연구자는 먼저 **'시스템 통합 엔지니어'**가 되어야 합니다. 시뮬레이터와 비행 컨트롤러를 연결하는 저수준 미들웨어 문제를 해결한 후에야 비로소 본래의 연구를 시작할 수 있습니다. 이는 프로젝트의 범위, 일정, 요구 기술 스택을 근본적으로 바꾸며, 특히 정해진 기간 내에 성과를 내야 하는 학위 과정의 연구자에게는 치명적인 위험 요소가 될 수 있습니다.

**표 2: PX4/ROS 2 자율성 연구를 위한 시뮬레이터 비교**

| 기준 (Criterion)             | Gazebo                                                  | Webots                                                       |
| ---------------------------- | ------------------------------------------------------- | ------------------------------------------------------------ |
| **PX4 SITL 통합**            | **"Out-of-the-box"**: 단일 명령어로 실행 및 자동 연결 1 | **"수동/커스텀"**: 센서/액추에이터를 위한 커스텀 브릿지 노드 개발 필수 7 |
| **ROS 2 지원**               | `ros_gz_bridge`를 통한 네이티브 지원 26                 | `webots_ros2` 패키지를 통한 네이티브 지원 5                  |
| **사용 편의성**              | 가파른 학습 곡선, XML 기반 월드 정의 31                 | 상대적으로 사용자 친화적, 세련된 GUI 32                      |
| **커뮤니티 자료 (PX4 연동)** | **풍부함**: 수많은 튜토리얼, 예제, 포럼 게시물 존재 8   | **거의 없음**: 일반적인 Webots ROS 2 문서에 의존해야 함 7    |
| **물리 엔진 정확도**         | 다중 엔진 옵션 (ODE, Bullet, DART) 38                   | 고도로 튜닝된 ODE 구현, 접촉(contact) 모델에서 더 안정적이라는 평가 38 |
| **다중 기체 시뮬레이션**     | 잘 문서화된 스크립트 제공 40                            | ROS 2를 통해 가능하나, 커스텀 런치 파일과 네임스페이스 설정 필요 |
| **프로젝트 위험도**          | **낮음**: 표준 경로를 따름                              | **높음**: 상당한 커스텀 통합 및 디버깅 작업 요구             |

------

## 3.  개발 환경 구축을 위한 실용 가이드

이 섹션에서는 이론적 분석을 바탕으로 실제 개발 환경을 구축하기 위한 구체적이고 실행 가능한 명령어 기반의 가이드를 제공합니다. '권장 경로'와 '고급 경로'로 나누어 연구자의 상황에 맞는 선택을 돕습니다.

### 3.1  권장 경로: Gazebo를 이용한 단계별 환경 설정

이 가이드는 여러 공식 문서와 튜토리얼에서 제시된 절차를 종합하여 가장 안정적이고 검증된 방법론을 제시합니다.1

**1단계: 시스템 필수 구성요소 설치 (Prerequisites)**

- **운영체제**: Ubuntu 22.04 LTS를 설치합니다.

- **ROS 2 Humble 설치**: ROS 2 Humble Desktop 버전과 개발 도구를 설치합니다.

  ```Bash
  # 로케일 설정
  sudo apt update && sudo apt install locales
  sudo locale-gen en_US en_US.UTF-8
  sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
  export LANG=en_US.UTF-8
  
  # ROS 2 저장소 추가 및 설치
  sudo apt install software-properties-common
  sudo add-apt-repository universe
  sudo apt update && sudo apt install curl -y
  sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
  sudo apt update
  sudo apt upgrade -y
  sudo apt install ros-humble-desktop ros-dev-tools
  ```

- **ROS 2 환경 설정**: `~/.bashrc` 파일에 ROS 2 환경 설정 스크립트를 추가하여 터미널을 열 때마다 자동으로 로드되도록 합니다.

  ```Bash
  echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
  source ~/.bashrc
  ```

- **파이썬 의존성 설치**: `colcon` 빌드 시스템에 필요한 파이썬 패키지를 설치합니다. 버전 충돌을 피하기 위해 명시된 버전을 사용하는 것이 중요할 수 있습니다.26

  ```Bash
  pip install --user -U empy pyros-genmsg setuptools
  ```

**2단계: PX4 툴체인 및 시뮬레이터 설치**

- `PX4-Autopilot` 저장소를 재귀적으로(submodule 포함) 클론합니다.

  ```Bash
  cd ~
  git clone https://github.com/PX4/PX4-Autopilot.git --recursive
  ```

- PX4의 `ubuntu.sh` 스크립트를 실행하여 모든 의존성(Gazebo 포함)을 설치합니다.

  ```Bash
  bash./PX4-Autopilot/Tools/setup/ubuntu.sh
  ```

  이후 시스템을 재부팅하거나 재로그인하여 변경사항을 적용하는 것이 좋습니다.

**3단계: Micro-XRCE-DDS Agent 빌드**

- 에이전트 소스 코드를 클론하고 빌드합니다. 특정 버전(예: v2.4.2)을 사용하는 것이 안정성을 높일 수 있습니다.3

  ```Bash
  cd ~
  git clone -b v2.4.2 https://github.com/eProsima/Micro-XRCE-DDS-Agent.git
  cd Micro-XRCE-DDS-Agent
  mkdir build && cd build
  cmake..
  make
  sudo make install
  sudo ldconfig /usr/local/lib/
  ```

**4단계: ROS 2 워크스페이스 생성 및 빌드**

- ROS 2 워크스페이스 디렉토리를 생성합니다.

  ```Bash
  mkdir -p ~/px4_ros_ws/src
  cd ~/px4_ros_ws/src
  ```

- 필수 패키지인 `px4_msgs`를 클론합니다. 이때, 사용 중인 PX4 버전과 호환되는 브랜치(`release/1.15`)를 명시해야 합니다.11

  ```Bash
  git clone -b release/1.15 https://github.com/PX4/px4_msgs.git
  ```

- (선택사항) 예제 코드를 위해 `px4_ros_com`을 클론합니다.

  ```Bash
  git clone https://github.com/PX4/px4_ros_com.git
  ```

- `colcon`을 사용하여 워크스페이스를 빌드합니다.

  ```bash
  cd ~/px4_ros_ws
  colcon build --symlink-install
  ```

**5단계: 첫 비행 시뮬레이션**

이제 세 개의 터미널을 열고 각각 다음 명령어를 실행하여 첫 시뮬레이션을 시작합니다. 이 세 가지 명령어의 조합은 PX4-ROS 2 시뮬레이션의 기본 패턴입니다.

- **터미널 1: PX4 SITL 및 Gazebo 실행**

  ```Bash
  cd ~/PX4-Autopilot
  make px4_sitl gz_x500
  ```

  이 명령은 PX4 SITL 프로세스와 Gazebo 시뮬레이터를 실행하고, x500 쿼드콥터 모델을 로드합니다.

- **터미널 2: Micro-XRCE-DDS Agent 실행**

  ```Bash
  MicroXRCEAgent udp4 -p 8888
  ```

  이 명령은 UDP 포트 8888에서 PX4 클라이언트의 연결을 기다리는 에이전트를 시작합니다.

- **터미널 3: ROS 2 노드 실행 (예: 센서 데이터 확인)**

  ```Bash
  # 워크스페이스 환경 설정
  cd ~/px4_ros_ws
  source install/setup.bash
  
  # 차량 상태 토픽을 echo하여 데이터 수신 확인
  ros2 topic echo /fmu/out/vehicle_status
  ```

이 과정의 복잡성은 각 구성요소가 서로 다른 소스(apt, git, pip)에서 오며, 각각의 버전과 설정이 정확히 맞아야만 전체 시스템이 동작한다는 점에서 비롯됩니다. 하나의 오래된 `pip` 패키지나 잘못된 `git` 브랜치 선택이 전체 빌드 실패로 이어질 수 있습니다. 이러한 환경의 취약성은 개발 환경을 불변의 상태로 고정하고 재현성을 보장하는 컨테이너화(Docker) 기술의 가치를 부각시킵니다. 실제로 다수의 고급 사용자 및 프로젝트는 이 복잡성을 관리하기 위해 Docker 기반 개발 환경을 구축하여 사용하고 있습니다.42 따라서 심각한 연구 개발 프로젝트에서는 Docker를 도입하는 것을 '모범 사례(best practice)'로 고려해야 합니다.

### 3.2  고급 경로: Webots 통합을 위한 이론적 워크플로우

이 경로는 상당한 시스템 통합 능력을 요구하며, 다음과 같은 이론적 절차를 따릅니다.

1. **Webots에서 로봇 컨트롤러 설정**: Webots 월드 파일(`.wbt`)에서 드론 로봇의 `controller` 필드를 `<extern>`으로 설정합니다. 이는 Webots에게 이 로봇이 외부 프로세스에 의해 제어될 것임을 알립니다.6

2. **`webots_ros2` 런치 파일 작성**: Webots 시뮬레이션과 `webots_ros2_driver`를 실행하는 ROS 2 런치 파일을 작성합니다. 이 런치 파일은 Webots를 시작하고, 지정된 로봇에 대한 ROS 2 인터페이스를 활성화하여 센서 데이터를 토픽으로 발행하고 액추에이터 제어를 위한 서비스를 노출시킵니다.6

3. **'접착제' 노드(Glue Node) 구현**: 이전에 제안된 '접착제' 노드를 실제로 구현합니다. 아래는 `rclpy`를 사용한 파이썬 코드의 개념적 골격입니다.

   ```Python
   import rclpy
   from rclpy.node import Node
   from px4_msgs.msg import ActuatorOutputs, SensorCombined
   from sensor_msgs.msg import Imu # Webots에서 발행하는 센서 타입
   #... 기타 필요한 메시지 타입 import
   
   class Px4WebotsBridge(Node):
       def __init__(self):
           super().__init__('px4_webots_bridge')
   
           # PX4 -> Webots (액추에이터 명령)
           self.actuator_outputs_sub = self.create_subscription(
               ActuatorOutputs, '/fmu/out/actuator_outputs', self.actuator_callback, 10)
           # Webots 모터 제어를 위한 퍼블리셔 또는 서비스 클라이언트 생성
           # self.webots_motor_pub = self.create_publisher(...)
   
           # Webots -> PX4 (센서 데이터)
           self.webots_imu_sub = self.create_subscription(
               Imu, '/webots/drone/imu', self.webots_imu_callback, 10)
           self.px4_sensor_combined_pub = self.create_publisher(
               SensorCombined, '/fmu/in/sensor_combined', 10)
   
       def actuator_callback(self, msg):
           # ActuatorOutputs 메시지를 Webots 모터 제어 명령으로 변환
           # 예: PWM 값을 각 모터의 속도(rad/s)로 변환
           # self.webots_motor_pub.publish(...)
           pass
   
       def webots_imu_callback(self, msg):
           # Webots의 Imu 메시지를 PX4의 SensorCombined 메시지로 변환
           # 좌표계 변환, 타임스탬프 처리 등이 필요
           px4_msg = SensorCombined()
           #... 데이터 채우기...
           self.px4_sensor_combined_pub.publish(px4_msg)
   
   def main(args=None):
       rclpy.init(args=args)
       bridge_node = Px4WebotsBridge()
       rclpy.spin(bridge_node)
       bridge_node.destroy_node()
       rclpy.shutdown()
   
   if __name__ == '__main__':
       main()
   ```

### 3.3  브릿지 연결 검증: 필수 시스템 진단 도구

환경 구축 후 시스템이 정상적으로 동작하는지 확인하는 것은 매우 중요합니다. 다음 진단 도구들을 순차적으로 활용하여 문제를 체계적으로 파악할 수 있습니다.

- **`ros2 topic list`**: 가장 먼저 사용해야 할 명령어입니다. PX4 SITL과 DDS 에이전트가 정상적으로 실행되고 연결되었다면, 터미널에 `/fmu/in/...` 및 `/fmu/out/...`으로 시작하는 수많은 토픽 목록이 나타나야 합니다.17 이 토픽들이 보이지 않는다면, PX4와 ROS 2 간의 브릿지가 전혀 동작하지 않고 있다는 명백한 증거입니다.
- **`ros2 topic echo <topic_name>`**: 특정 토픽의 데이터를 실시간으로 확인합니다. 예를 들어 `ros2 topic echo /fmu/out/vehicle_odometry`를 실행하면, 시뮬레이션 중인 드론의 위치와 자세 데이터가 지속적으로 출력되는 것을 볼 수 있습니다.22 데이터가 출력되지 않거나 간헐적으로만 나타난다면, 연결 상태가 불안정하거나 데이터 생성에 문제가 있음을 의미합니다.
- **에이전트 및 클라이언트 터미널 로그 확인**: PX4 SITL(클라이언트)과 `MicroXRCEAgent`(에이전트)가 실행 중인 각 터미널의 출력 로그는 문제 해결의 첫 단서입니다. 성공적으로 연결되면 "create entities", "session established", "synchronized with time offset"과 같은 로그가 양쪽 터미널에 나타납니다.3 연결 실패, 타임아웃, 데이터 형식 오류 등의 에러 메시지가 출력된다면 해당 메시지를 중심으로 문제를 추적해야 합니다.
- **`ros2 doctor`**: ROS 2 환경 전반의 설정 오류, 네트워크 문제 등을 점검하는 범용 진단 도구입니다.47 특히 DDS 통신에 필수적인 네트워크 멀티캐스트 설정 문제 등을 발견하는 데 유용합니다.

------

## 4.  자율 비행 연구의 최전선

성공적으로 구축된 PX4/ROS 2 개발 환경은 다양한 첨단 자율 비행 연구를 수행할 수 있는 강력한 플랫폼을 제공합니다. 이 섹션에서는 해당 스택을 활용하여 도전할 수 있는 주요 연구 분야와 그 가능성을 조망합니다.

### 4.1  기본 역량: 오프보드 제어 및 커스텀 비행 모드

모든 자율 비행의 시작은 외부 컴퓨터(컴패니언 컴퓨터)에서 드론을 제어하는 기술에서 비롯됩니다.

- **오프보드 제어(Offboard Control)**: 이는 컴패니언 컴퓨터에서 계산된 위치, 속도, 자세 등의 목표값(setpoint)을 PX4로 전송하여 드론을 제어하는 가장 기본적인 자율 비행 모드입니다.8 ROS 2 노드는 

  `/fmu/in/trajectory_setpoint`과 같은 토픽에 `TrajectorySetpoint` 메시지를 발행함으로써 드론의 움직임을 정밀하게 제어할 수 있습니다. ARK Electronics에서 제공하는 예제 프로젝트들은 이러한 오프보드 제어를 시작하는 데 훌륭한 출발점을 제공합니다.8

- **커스텀 비행 모드(Custom Flight Modes)**: 더 나아가, Auterion에서 개발한 `px4-ros2-interface-lib`와 같은 라이브러리를 사용하면, ROS 2 노드에서 완전히 새로운 비행 로직을 구현하고 이를 PX4의 정식 비행 모드처럼 등록하여 사용할 수 있습니다.4 예를 들어, 복잡한 궤적(예: 'R'자 모양 비행)을 그리는 비행 모드를 ROS 2의 C++ 또는 Python 코드로 작성하고, 이를 QGroundControl이나 조종기에서 다른 내장 비행 모드(Loiter, Mission 등)와 동일하게 선택하여 실행할 수 있습니다. 이는 PX4 펌웨어 자체를 수정하지 않고도 드론의 핵심 동작을 확장할 수 있는 매우 강력한 기능입니다.

### 4.2  인지 기반 자율성: SLAM, 장애물 회피, 비전 기반 항법

드론이 주변 환경을 인지하고 스스로 판단하여 비행하는 것은 자율 비행 연구의 핵심입니다. PX4/ROS 2 스택은 이러한 인지 기반 자율성 연구를 위한 이상적인 환경을 제공합니다.

- **장애물 회피(Obstacle Avoidance)**: PX4는 ROS 1 환경을 위한 `PX4/PX4-Avoidance`라는 공식 장애물 회피 패키지를 제공한 바 있습니다.50 이 패키지는 깊이 카메라(depth camera)와 같은 센서 데이터를 사용하여 지역 경로 계획(local planning)과 전역 경로 계획(global planning)을 수행하는 개념적 모델을 제시합니다. 하지만 

  **현재 PX4 팀이 공식적으로 유지보수하는 ROS 2 버전의 장애물 회피 솔루션은 부재한 상태**입니다.52 이는 역설적으로 연구자들에게 큰 기회를 제공합니다. 안정적인 플랫폼 위에서 자신만의 독자적인 장애물 회피 알고리즘(예: VFH+, DWA, RRT*, APF 등)을 ROS 2 노드로 구현하고 그 성능을 검증하는 것은 매우 가치 있는 연구 주제가 될 수 있습니다.

- **Visual SLAM 및 비전 기반 항법**: 최근 다수의 학술 연구에서 PX4/ROS 2/Gazebo 스택을 활용하여 VSLAM(Visual Simultaneous Localization and Mapping) 및 GPS 음영 지역에서의 항법 기술을 개발하고 있습니다.53 특히, Realsense T265와 같은 VIO(Visual-Inertial Odometry) 센서의 위치 및 자세 추정 데이터를 `/fmu/in/vehicle_visual_odometry` 토픽으로 발행하여 GPS를 대체하는 연구는 매우 활발하게 이루어지고 있습니다.57 이는 실내 환경이나 도심 협곡과 같이 GPS 신호가 불안정한 곳에서 드론을 운용하기 위한 핵심 기술입니다.58

### 4.3  고급 임무: 다중 드론 시스템 및 정밀 기동

단일 드론의 자율성을 넘어, 여러 대의 드론이 협력하거나 고도의 정밀도를 요구하는 임무는 자율 비행 기술의 정점이라 할 수 있습니다.

- **다중 기체 시뮬레이션(Multi-Vehicle Simulation)**: PX4와 Gazebo는 여러 대의 드론을 동시에 시뮬레이션하는 기능을 잘 지원합니다. 제공되는 스크립트를 사용하면 각 드론에 고유한 ID와 네임스페이스(namespace)가 할당되고, 각각의 드론은 별도의 DDS 브릿지 포트를 통해 독립적으로 제어될 수 있습니다.40 이를 통해 드론 군집 비행, 협력 탐사, 자원 할당과 같은 복잡한 다중 에이전트 시스템 연구를 수행할 수 있습니다.
- **정밀 착륙(Precision Landing)**: '라스트 마일 배송(last-mile delivery)'과 같은 시나리오에서 최종 목적지에 정확하게 착륙하는 기술은 매우 중요합니다. 여러 공개 GitHub 프로젝트에서는 카메라와 ArUco 마커 같은 컴퓨터 비전 기술을 활용하여 오차 범위 수 센티미터 이내의 정밀 착륙을 구현하는 ROS 2 노드를 개발하고 있습니다.61
- **인공지능 및 머신러닝 통합**: 이 스택은 최신 AI 기술과의 통합에도 열려 있습니다. YOLOv8과 같은 객체 탐지 모델을 ROS 2 노드에 통합하여 특정 목표물을 추적하거나 감시하는 임무를 수행할 수 있으며 44, 더 나아가 LLM(거대 언어 모델)을 활용하여 자연어 명령으로 드론을 제어하는 혁신적인 연구 프레임워크도 등장하고 있습니다.58

현재 PX4/ROS 2 생태계는 uXRCE-DDS 브릿지와 같은 핵심 기반 기술은 매우 안정화되었지만, 장애물 회피 시스템과 같은 고급 응용 프로그램은 공식 지원보다는 학술적 프로토타입이나 커뮤니티 주도 프로젝트 형태로 존재하는 과도기적 특징을 보입니다. 이는 연구자에게 중요한 시사점을 줍니다. 플랫폼은 신뢰할 수 있을 만큼 성숙했지만, 가장 흥미로운 응용 레벨의 문제들은 아직 '해결되지 않은' 상태로 남아있습니다. 따라서 연구자는 이 안정적인 기반 위에서 경로 계획, 장애물 회피, 군집 제어 등과 같은 고수준 자율성 패키지를 개발하고 공개함으로써 생태계에 의미 있는 기여를 할 수 있는 절호의 기회를 맞이하고 있습니다.

------

## 5.  주요 문제점 및 고급 문제 해결(Troubleshooting)

PX4-ROS 2 생태계는 강력하지만, 여러 독립적인 구성요소의 조합으로 이루어져 있어 다양한 문제에 직면할 수 있습니다. 이 섹션에서는 개발자가 흔히 겪는 문제들의 원인을 분석하고 실질적인 해결책을 제시합니다. 대부분의 문제는 코드의 논리적 결함보다는 환경 설정의 오류에서 비롯된다는 점을 인지하는 것이 중요합니다.

### 5.1  빌드 시스템의 복잡성 탐색

- **`colcon` 빌드 실패**: ROS 2 워크스페이스를 빌드하는 `colcon build` 명령어는 파이썬 의존성 문제로 인해 실패하는 경우가 많습니다. 특히 `setuptools`나 `empy`와 같은 패키지의 버전이 시스템에 설치된 다른 파이썬 패키지와 충돌할 때 발생합니다. 터미널에 나타나는 오류 메시지를 자세히 살펴보면 특정 패키지의 버전 문제를 지적하는 경우가 많으며, 이 경우 `pip install "setuptools<59.0.0" "empy<4"`와 같이 명시적으로 버전을 지정하여 재설치하면 문제가 해결될 수 있습니다.26
- **워크스페이스 소싱(Sourcing) 오류**: ROS 2는 여러 계층의 환경 설정(overlay)을 기반으로 동작합니다. 따라서 터미널에서 ROS 2 명령어나 노드를 실행하기 전에 올바른 순서로 `setup.bash` 파일을 소싱해야 합니다. 가장 먼저 ROS 2의 기본 환경(`underlay`)인 `/opt/ros/humble/setup.bash`를 소싱하고, 그 다음으로 개발 중인 워크스페이스의 환경(`overlay`)인 `~/your_ws/install/setup.bash`를 소싱해야 합니다.1 이 순서가 바뀌거나 워크스페이스 소싱을 누락하면, `ros2 run` 실행 시 "package not found" 오류가 발생합니다.

### 5.2  uXRCE-DDS 연결 및 성능 문제 진단

- **역직렬화 오류(Deserialization Errors)**: 터미널 로그에 `"Fast CDR exception deserializing message"`라는 오류가 나타난다면 22, 이는 거의 100%의 확률로 PX4 펌웨어에 컴파일된 메시지 정의와 ROS 2 워크스페이스의 

  `px4_msgs` 버전이 일치하지 않기 때문입니다. PX4-Autopilot 저장소의 커밋(commit)과 `px4_msgs` 저장소의 브랜치/커밋이 정확히 호환되는지 다시 한번 확인하고, 워크스페이스를 `colcon clean` 후 재빌드해야 합니다.

- **에이전트/클라이언트 연결 실패**: 브릿지가 연결되지 않는 데에는 여러 원인이 있을 수 있습니다.

  - **방화벽 문제**: 특히 WSL(Windows Subsystem for Linux) 환경에서 Windows Defender 방화벽이 UDP 통신을 차단하는 경우가 흔합니다. Webots와 ROS 2 컨트롤러 간의 연결 실패 또한 이 문제일 가능성이 높습니다.64 방화벽 설정에서 관련 포트(기본값: UDP 8888)를 허용해야 합니다.
  - **포트 충돌**: 이미 다른 프로세스가 사용 중인 UDP 포트로 에이전트를 실행하려 하거나, 여러 에이전트 인스턴스를 동일한 포트로 실행하려 하면 연결에 실패합니다.
  - **네트워크 인터페이스**: 가상 머신이나 Docker 환경에서는 네트워크 인터페이스 설정이 올바르지 않아 통신이 실패할 수 있습니다.

- **성능 병목 현상**: 실제 드론 하드웨어와 UART 시리얼 통신을 사용할 때 특히 자주 발생하는 문제입니다. PX4의 `dds_topics.yaml` 파일은 기본적으로 수많은 uORB 토픽을 브릿징하도록 설정되어 있습니다. 이 모든 데이터를 낮은 대역폭의 시리얼 링크로 전송하려고 하면 링크가 포화 상태에 빠져 메시지 손실, 높은 지연, 주기적인 연결 끊김 현상이 발생합니다. 이 문제에 대한 해결책은 PX4 소스 코드 내의 `dds_topics.yaml` 파일을 수정하여, 현재 연구에 **반드시 필요한 토픽들만** 브릿징하도록 목록을 최소화하는 것입니다. 이 간단한 최적화만으로도 통신 신뢰도와 데이터 업데이트 주기가 극적으로 향상될 수 있습니다.23

### 5.3  시뮬레이터 특정 버그 해결

- **Gazebo "No rule to make target" 오류**: `make px4_sitl gz_x500` 실행 시 이 오류가 발생한다면, Gazebo의 개발 라이브러리가 제대로 설치되지 않았음을 의미합니다.65 PX4의 

  `ubuntu.sh` 설치 스크립트가 중간에 실패했거나 중단된 경우 발생할 수 있으므로, 스크립트를 다시 실행하거나 `sudo apt install libgz-sim7-dev`와 같이 관련 패키지를 수동으로 설치해야 합니다.

- **Webots "Waiting for extern controller" 멈춤 현상**: ROS 2 런치 파일을 통해 Webots 시뮬레이션을 시작했을 때, 시뮬레이션이 시작되지 않고 "Waiting for extern controller..." 메시지에서 멈추는 경우가 있습니다. 이는 `webots_ros2_driver` 노드가 Webots 인스턴스와 통신하지 못하기 때문이며, 주된 원인은 앞서 언급한 방화벽 문제나 `WEBOTS_CONTROLLER_URL` 환경 변수의 설정 오류입니다. 특히 WSL과 Windows 호스트 간의 통신에서 자주 발생합니다.64

- **시뮬레이터와 `ros2_control` 연동 문제**: `ros2_control` 프레임워크를 사용하여 시뮬레이션 내 로봇을 제어하려는 고급 사용자들은 종종 `controller_manager` 서비스를 찾을 수 없다는 문제에 직면합니다. 이로 인해 `joint_state_broadcaster`나 `joint_trajectory_controller`와 같은 컨트롤러들을 스폰(spawn)할 수 없게 됩니다.66 이는 시뮬레이터의 ROS 2 플러그인과 

  `ros2_control` 간의 초기화 순서나 네임스페이스 문제일 수 있으며, 해결하기 까다로운 고급 주제에 속합니다.

이러한 문제 해결 과정은 개발자에게 단순한 코딩 능력을 넘어, 분산 시스템 전체를 조망하는 **'풀스택(full-stack)' 시스템 사고방식**을 요구합니다. 문제가 발생했을 때, 자신의 ROS 2 노드 코드만 디버깅하는 것이 아니라, 펌웨어 설정, 메시지 버전, DDS 에이전트 상태, 네트워크 방화벽, 워크스페이스 소싱 상태 등 통신 체인의 모든 단계를 체계적으로 점검할 수 있어야 합니다. 이는 견고한 진단 도구의 중요성과 명확하고 모호하지 않은 터미널 로그의 가치를 더욱 부각시킵니다. 또한, 이러한 환경의 복잡성과 취약성은 올바른 구성을 '구워서' 이식 가능한 결과물로 만드는 컨테이너화(Docker)가 왜 단순한 편의 기능을 넘어, 심각한 연구 개발을 위한 필수 전략이 되는지를 명확히 보여줍니다.

------

## 6.  전략적 권고 및 결론

본 보고서의 심층 분석을 종합하여, PX4, ROS 2 Humble, Webots 스택을 사용한 자율 비행 연구를 고려하는 연구자를 위한 명확하고 실행 가능한 전략적 권고를 제시합니다.

### 6.1  PX4/ROS 2/Webots 스택에 대한 최종 평가

결론적으로, PX4 v1.15.4, ROS 2 Humble, Webots 조합으로 드론 자율 비행을 연구하는 것은 **기술적으로는 가능하지만, 대부분의 연구 프로젝트에 있어 전략적으로 권장되지 않습니다.** 가장 큰 이유는 PX4 SITL과 Webots 간의 직접적이고 공식적으로 지원되는 통합 경로가 부재하다는 점입니다.7

이 기술적 공백을 메우기 위해 연구자는 센서 데이터와 액추에이터 명령을 양방향으로 중계하는 커스텀 브릿지 노드를 직접 개발해야 합니다. 이 작업은 단순한 스크립트 작성을 넘어, 좌표계 변환, 타임스탬프 동기화, 데이터 형식 변환 등 복잡한 시스템 통합 지식을 요구합니다. 이는 연구 프로젝트의 초기 단계에 상당한 시간과 노력을 투입하게 만들어, 정작 핵심 연구인 자율 비행 알고리즘 개발을 지연시키는 심각한 위험 요소로 작용합니다. 따라서 Webots가 제공하는 특정 기능이 연구 목표 달성에 대체 불가능한 필수 요소가 아닌 한, 이 경로를 선택하는 것은 높은 기회비용을 수반합니다.

### 6.2  신규 드론 자율성 프로젝트를 위한 권장 경로

새로운 드론 자율 비행 연구를 시작하는 개발자 및 연구자에게는 다음과 같은 단계적 접근법을 강력히 권고합니다.

1. **Gazebo로 시작하라 (Start with Gazebo)**: 모든 프로젝트는 예외 없이 **PX4/ROS 2/Gazebo** 스택으로 시작해야 합니다. 이 경로는 PX4 커뮤니티의 표준이며, 방대한 문서, 튜토리얼, 예제 코드, 그리고 커뮤니티의 지원을 통해 개발 환경 구축과 관련된 문제를 최소화할 수 있습니다.8 이를 통해 연구자는 시스템 통합이라는 부차적인 문제에 발목 잡히지 않고, 프로젝트 초기부터 핵심 자율성 알고리즘 개발에 집중할 수 있습니다.
2. **기본기를 마스터하라 (Master the Fundamentals)**: 안정적인 Gazebo 환경 내에서 uXRCE-DDS 브릿지의 동작 원리, 오프보드 제어 메커니즘, 그리고 기본적인 ROS 2 노드(Publisher/Subscriber) 개발 방법을 완벽하게 숙달해야 합니다. `ros2 topic`, `ros2 node`, `ros2 launch`와 같은 기본 CLI 도구를 능숙하게 사용하여 시스템을 진단하고 제어하는 능력을 갖추는 것이 필수적입니다.
3. **Webots의 필요성을 명확히 하라 (Isolate the Need for Webots)**: 표준 스택에 대한 숙련도를 갖춘 후, 자신의 연구 목표를 다시 한번 검토해야 합니다. 만약 Gazebo가 제공하지 못하는, 그러나 Webots는 제공하는 **결정적이고 대체 불가능한 기능**(예: 특정 접촉 물리 모델의 높은 정확도, 고유한 센서 시뮬레이션, 특정 시나리오에 대한 우월한 렌더링 품질 등)이 연구의 성패를 좌우하는 핵심 요소임이 명확해졌을 때만 Webots로의 전환을 고려해야 합니다.
4. **Webots 통합을 별도의 프로젝트로 취급하라 (Treat Webots Integration as a Separate Project)**: 만약 Webots로의 전환이 불가피하다면, 이를 본 연구와는 별개의 **시스템 통합 서브 프로젝트**로 계획하고 관리해야 합니다. 필요한 '접착제' 노드의 요구사항을 명확히 정의하고, 개발 및 테스트를 위한 별도의 일정과 리소스를 할당해야 합니다. 이는 통합 작업의 복잡성과 위험을 명확히 인지하고 관리하기 위함입니다.

### 6.3  미래 전망: 진화하는 드론 개발 생태계

드론 개발 생태계는 매우 역동적으로 진화하고 있습니다. uXRCE-DDS의 채택은 PX4가 더 넓은 산업용 로보틱스 시장으로 나아가기 위한 중요한 발판이 되었습니다. 복잡성이 증가함에 따라, 이를 관리하기 위한 컨테이너화(Docker) 기술의 중요성은 더욱 커질 것입니다.

또한, 시뮬레이션 분야에서는 NVIDIA의 Isaac Sim과 같이 AI 및 강화학습 연구에 특화된 강력한 시뮬레이터들이 부상하고 있습니다.31 이는 미래의 드론 연구가 더욱 데이터 중심적이고 AI 기반으로 발전할 것임을 시사합니다.

현재 생태계는 안정적인 저수준 제어 플랫폼(PX4) 위에, 커뮤니티가 주도하여 고수준 자율성 애플리케이션(장애물 회피, 경로 계획 등)을 구축해 나가는 흥미로운 단계에 있습니다. 따라서 이 분야의 연구자는 단순히 기존 도구를 사용하는 소비자를 넘어, 생태계의 발전에 기여하는 중요한 생산자가 될 수 있는 무한한 기회를 가지고 있습니다. 올바른 전략적 선택과 체계적인 접근을 통해, 연구자는 이 역동적인 분야에서 의미 있는 성과를 창출할 수 있을 것입니다.

#### **참고 자료**

1. ROS 2 User Guide | PX4 Guide (main) - PX4 docs, accessed July 25, 2025, https://docs.px4.io/main/en/ros2/user_guide
2. ROS 2 User Guide | PX4 Guide (v1.15), accessed July 25, 2025, https://docs.px4.io/v1.15/en/ros2/user_guide.html
3. ROS 2 User Guide | PX4 Guide (main) - PX4 docs - PX4 Autopilot, accessed July 25, 2025, https://docs.px4.io/main/en/ros2/user_guide.html
4. Custom flight modes using PX4 and ROS 2 - RIIS LLC, accessed July 25, 2025, https://www.riis.com/blog/custom-flight-modes-using-px4-and-ros2
5. webots_ros2_driver: Humble 2025.0.0 documentation, accessed July 25, 2025, https://docs.ros.org/en/humble/p/webots_ros2_driver/
6. Setting up a robot simulation (Basic) - ROS 2 Documentation: Humble documentation, accessed July 25, 2025, https://docs.ros.org/en/humble/Tutorials/Advanced/Simulators/Webots/Setting-Up-Simulation-Webots-Basic.html
7. cyberbotics/webots_ros2: Webots ROS 2 packages - GitHub, accessed July 25, 2025, https://github.com/cyberbotics/webots_ros2
8. ROS2 and PX4 Tutorial | Simulated Offboard Mode - YouTube, accessed July 25, 2025, https://www.youtube.com/watch?v=8gKIP0OqHdQ&pp=0gcJCfwAo7VqN5tD
9. Gazebo Simulation | PX4 User Guide (v1.12), accessed July 25, 2025, https://docs.px4.io/v1.12/en/simulation/gazebo.html
10. Ignition Gazebo for PX4 Software-In-The-Loop Simulations - Jaeyoung Lim, ETH Zurich, accessed July 25, 2025, https://www.youtube.com/watch?v=KIcFbCiK0QU
11. PX4/px4_msgs: ROS/ROS2 messages that match the ... - GitHub, accessed July 25, 2025, https://github.com/PX4/px4_msgs
12. ROS 2 user guide - GitHub, accessed July 25, 2025, https://github.com/PX4/PX4-user_guide/blob/main/tr/ros2/user_guide.md
13. eProsima/Micro-XRCE-DDS-Agent - GitHub, accessed July 25, 2025, https://github.com/eProsima/Micro-XRCE-DDS-Agent
14. uXRCE-DDS (PX4-ROS 2/DDS Bridge) | PX4 Guide (main), accessed July 25, 2025, https://docs.px4.io/main/en/middleware/uxrce_dds.html
15. uXRCE-DDS (PX4-ROS 2/DDS Bridge) | PX4 오토파일럿 사용자 설명서 (v1.14), accessed July 25, 2025, https://docs.px4.io/v1.14/ko/middleware/uxrce_dds.html
16. Announcing the DDS-XRCE Specification: A Protocol for Sensor Networks - RTI, accessed July 25, 2025, https://www.rti.com/blog/announcing-the-dds-xrce-specification-a-protocol-for-sensor-networks
17. Transitioning from ROS 1 (ArduPilot) to ROS 2 (PX4): A Complete Mental Model | by Sidharth Mohan Nair | Medium, accessed July 25, 2025, https://medium.com/@sidharthmohannair/transitioning-from-ros-1-ardupilot-to-ros-2-px4-a-complete-mental-model-98c54758f569
18. ROS with MAVROS Installation Guide | PX4 User Guide (v1.14) - PX4 docs, accessed July 25, 2025, https://docs.px4.io/v1.14/en/ros/mavros_installation.html
19. mavros - ROS Wiki, accessed July 25, 2025, http://wiki.ros.org/mavros
20. ROS 2 User Guide - PX4 docs, accessed July 25, 2025, https://docs.px4.io/v1.14/en/ros/ros2_comm.html
21. PX4/px4_ros_com: ROS2/ROS interface with PX4 through a Fast-RTPS bridge - GitHub, accessed July 25, 2025, https://github.com/PX4/px4_ros_com
22. Problems with some px4 ros topics - ROS 1 / ROS 2 - PX4 Discussion Forum, accessed July 25, 2025, https://discuss.px4.io/t/problems-with-some-px4-ros-topics/35854
23. [Bug] Issue Transporting DDS Topics Across UART / Issue #22989 / PX4/PX4-Autopilot, accessed July 25, 2025, https://github.com/PX4/PX4-Autopilot/issues/22989
24. Robot Operating System 2: Design, Architecture, and Uses In The Wild - ResearchGate, accessed July 25, 2025, https://www.researchgate.net/publication/365414963_Robot_Operating_System_2_Design_Architecture_and_Uses_In_The_Wild
25. ARK-Electronics/px4_ros2_examples_ws: ROS2 Humble desktop workspace for integration with PX4 simulation - GitHub, accessed July 25, 2025, https://github.com/ARK-Electronics/px4_ros2_examples_ws
26. ROS2 Humble, Gazebo Harmonic, PX4, QGroundControl, Micro XRCE-DDS Agent & Client Installation | by Erdem | Medium, accessed July 25, 2025, https://medium.com/@erdem.ku.3.14/ros2-humble-gazebo-harmonic-px4-ve-micro-xrce-dds-agent-client-installation-aad32d8f5669
27. How to setup PX4 SITL with ROS2 and XRCE-DDS Gazebo simulation on Ubuntu 22, accessed July 25, 2025, https://kuat-telegenov.notion.site/How-to-setup-PX4-SITL-with-ROS2-and-XRCE-DDS-Gazebo-simulation-on-Ubuntu-22-e963004b701a4fb2a133245d96c4a247
28. Understanding PX4/ROS Gazebo sim - Robotics Stack Exchange, accessed July 25, 2025, https://robotics.stackexchange.com/questions/91257/understanding-px4-ros-gazebo-sim
29. How to setup PX4 SITL with ROS2 and XRCE-DDS Gazebo simulation on Ubuntu 22, accessed July 25, 2025, https://www.youtube.com/watch?v=28oXzQYfiAQ
30. Px4 simulation with ROS2 Humble and Gazebo, accessed July 25, 2025, https://discuss.px4.io/t/px4-simulation-with-ros2-humble-and-gazebo/32661
31. Why is Gazebo very famous in the ROS community? what about Webots?, accessed July 25, 2025, https://discourse.ros.org/t/why-is-gazebo-very-famous-in-the-ros-community-what-about-webots/42459
32. Why is Gazebo very famous in the ROS community? what about Webots? - #6 by rxduty, accessed July 25, 2025, https://discourse.ros.org/t/why-is-gazebo-very-famous-in-the-ros-community-what-about-webots/42459/6
33. Setting up a robot simulation (Webots) - ROS 2 Documentation, accessed July 25, 2025, https://docs.ros.org/en/galactic/Tutorials/Advanced/Simulators/Webots.html
34. Webots - ROS 2 Documentation: Humble documentation, accessed July 25, 2025, https://docs.ros.org/en/humble/Tutorials/Advanced/Simulators/Webots/Simulation-Webots.html
35. SITL with Webots - Dev documentation - ArduPilot, accessed July 25, 2025, https://ardupilot.org/dev/docs/sitl-with-webots.html
36. Using SITL with Webots Python - Dev documentation - ArduPilot, accessed July 25, 2025, https://ardupilot.org/dev/docs/sitl-with-webots-python.html
37. gazebo vs webots : r/robotics - Reddit, accessed July 25, 2025, https://www.reddit.com/r/robotics/comments/zqzdyt/gazebo_vs_webots/
38. How does the accuracy of the dynamics of Webots compare to Gazebo? : r/robotics - Reddit, accessed July 25, 2025, https://www.reddit.com/r/robotics/comments/hhz5nn/how_does_the_accuracy_of_the_dynamics_of_webots/
39. drone simulation environment setup (PX4, ROS2, gazebo) | by Kazuya Hirotsu | Medium, accessed July 25, 2025, https://medium.com/@kazuyahirotsu/drone-simulation-environment-setup-px4-ros-89314a3fba42
40. Multi-Vehicle Simulation with Gazebo | PX4 User Guide (v1.12), accessed July 25, 2025, https://docs.px4.io/v1.12/en/simulation/multi_vehicle_simulation_gazebo.html
41. Loading multiple drones (px4 iris models) in a gazebo-classic simulation and controlling them using XRCE-DDS with ROS2 Foxy, accessed July 25, 2025, https://discuss.px4.io/t/loading-multiple-drones-px4-iris-models-in-a-gazebo-classic-simulation-and-controlling-them-using-xrce-dds-with-ros2-foxy/31870
42. mzahana/px4_ros2_humble: A Docker development environment for PX4 + ROS 2 Humble - GitHub, accessed July 25, 2025, https://github.com/mzahana/px4_ros2_humble
43. edouardrolland/vscode_ros2_px4_workspace: A template for generating a docker development environment suited for PX4, Gazebo and ROS2. - GitHub, accessed July 25, 2025, https://github.com/edouardrolland/vscode_ros2_px4_workspace
44. monemati/PX4-ROS2-Gazebo-YOLOv8 - GitHub, accessed July 25, 2025, https://github.com/monemati/PX4-ROS2-Gazebo-YOLOv8
45. PX4 to ROS2 connection does not work - ROS 1 / ROS 2 - PX4 Discussion Forum, accessed July 25, 2025, https://discuss.px4.io/t/px4-to-ros2-connection-does-not-work/31275
46. Problems with uXRCE-DDS middleware - PX4 Discussion Forum, accessed July 25, 2025, https://discuss.px4.io/t/problems-with-uxrce-dds-middleware/43670
47. Using ros2doctor to identify issues - ROS 2 Documentation: Humble documentation, accessed July 25, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Getting-Started-With-Ros2doctor.html
48. Marnonel6/ROS2_offboard_drone_control: ROS2 Off-board drone control with a companion computer and PX4 over DDS (serial). Autonomous drone with path planning. - GitHub, accessed July 25, 2025, https://github.com/Marnonel6/ROS2_offboard_drone_control
49. Custom flight modes using PX4 and ROS2 - YouTube, accessed July 25, 2025, https://m.youtube.com/watch?v=boLXuAxdF0U&pp=0gcJCU8JAYcqIYzv
50. PX4 avoidance ROS node for obstacle detection and avoidance. - GitHub, accessed July 25, 2025, https://github.com/PX4/PX4-Avoidance
51. ldg810/PX4-global-planner-ros2: PX4 avoidance ROS node for obstacle detection and avoidance. - GitHub, accessed July 25, 2025, https://github.com/ldg810/PX4-global-planner-ros2
52. Obstacle Avoidance on ROS2 - Page 2 - ROS 1 / ROS 2 - Discussion Forum for PX4, Pixhawk, QGroundControl, MAVSDK, MAVLink, accessed July 25, 2025, https://discuss.px4.io/t/obstacle-avoidance-on-ros2/32391?page=2
53. VAULT: A Mobile Mapping System for ROS 2-based Autonomous Robots - arXiv, accessed July 25, 2025, https://arxiv.org/html/2506.09583v1
54. XTDrone: A Customizable Multi-Rotor UAVs Simulation Platform - arXiv, accessed July 25, 2025, https://arxiv.org/pdf/2003.09700
55. [2003.09700] XTDrone: A Customizable Multi-Rotor UAVs Simulation Platform - arXiv, accessed July 25, 2025, https://arxiv.org/abs/2003.09700
56. [2012.00298] End-to-End UAV Simulation for Visual SLAM and Navigation - arXiv, accessed July 25, 2025, https://arxiv.org/abs/2012.00298
57. PX4-ROS2 VIO navigation in GPS denied condition, accessed July 25, 2025, https://discuss.px4.io/t/px4-ros2-vio-navigation-in-gps-denied-condition/24861
58. Taking Flight with Dialogue: Enabling Natural Language Control for PX4-based Drone Agent - arXiv, accessed July 25, 2025, https://arxiv.org/html/2506.07509v1
59. GPS-Denied Navigation Using Low-Cost Inertial Sensors and Recurrent Neural Networks - arXiv, accessed July 25, 2025, http://arxiv.org/pdf/2109.04861
60. Impossible to publish for multi-drone simulation with ROS2 & Gazebo - PX4 Autopilot, accessed July 25, 2025, https://discuss.px4.io/t/impossible-to-publish-for-multi-drone-simulation-with-ros2-gazebo/35682
61. REGATTE/Aruco_Tracker_ROS2_Drones: Precision Landing for Pixhawk drones using ROS2 and Computer Vision - GitHub, accessed July 25, 2025, https://github.com/REGATTE/Aruco_Tracker_ROS2_Drones
62. Kenil16/master_project: This project is about vision based navigation and precision landing of a drone using ROS, PX4 and OpenCV - GitHub, accessed July 25, 2025, https://github.com/Kenil16/master_project
63. Writing a simple publisher and subscriber (Python) - ROS 2 Documentation, accessed July 25, 2025, https://docs.ros.org/en/foxy/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Py-Publisher-And-Subscriber.html
64. No external controller connection(ROS2 on WLS to Webots on Windows) / Issue #759 / cyberbotics/webots_ros2 - GitHub, accessed July 25, 2025, https://github.com/cyberbotics/webots/issues/6154
65. PX4 autopilot ROS2 user guide problem, accessed July 25, 2025, https://discuss.px4.io/t/px4-autopilot-ros2-user-guide-problem/43825
66. Webots ros2_control and controller_manager issue - Robotics Stack Exchange, accessed July 25, 2025, https://robotics.stackexchange.com/questions/108070/webots-ros2-control-and-controller-manager-issue