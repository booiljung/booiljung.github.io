# ROS2 Humble 완전 정복

## 1.  ROS2, 로봇 개발의 새로운 표준: 왜 ROS2인가?

로봇 개발의 세계에 첫발을 내딛는 것을 환영한다. 수많은 기술과 프레임워크 속에서 길을 잃기 쉽지만, 현대 로봇 공학의 중심에는 ROS2(Robot Operating System 2)가 굳건히 자리 잡고 있다. 이 섹션에서는 ROS가 무엇인지, 그리고 왜 ROS1에서 ROS2로의 전환이 단순한 업데이트가 아닌 혁명적인 변화인지, 마지막으로 왜 우리가 'Humble Hawksbill' 배포판으로 시작해야 하는지에 대해 깊이 파고들 것이다.

### 1.1  ROS(로봇 운영체제)란 무엇인가?

먼저 이름에 대한 오해부터 바로잡자. ROS는 '운영체제(OS)'라는 이름을 가지고 있지만, 윈도우나 리눅스 같은 전통적인 의미의 OS는 아니다.1 ROS는 로봇 애플리케이션 개발을 위한 오픈 소스 **소프트웨어 프레임워크**이자 **미들웨어**다.1 복잡한 로봇 시스템을 개발하는 과정을 더 쉽고 효율적으로 만들기 위해 탄생했다.3

로봇은 카메라, 라이다(LiDAR), 모터, 관성측정장치(IMU) 등 수많은 하드웨어 센서와 액추에이터, 그리고 이들을 제어하고 데이터를 처리하는 수많은 소프트웨어 알고리즘의 집합체다. ROS는 이처럼 각기 다른 구성 요소들을 표준화된 방식으로 연결하고 서로 통신하게 해주는 '접착제' 역할을 한다. 개발자는 하드웨어 드라이버의 저수준(low-level) 구현에 신경 쓸 필요 없이, ROS가 제공하는 풍부한 라이브러리와 도구를 활용하여 로봇의 '지능'을 구현하는 데 집중할 수 있다.3

### 1.2  ROS1에서 ROS2로의 혁신적 전환

ROS2는 ROS1의 단순한 버전 업그레이드가 아니다.5 이는 로봇 기술의 적용 분야가 학술 연구를 넘어 실제 산업 현장으로 확장되면서 마주한 근본적인 한계들을 극복하기 위한 **아키텍처의 전면적인 재설계**다. ROS1이 주로 대학과 연구소 환경에서 단일 로봇 연구를 위해 탄생했다면 6, ROS2는 상업용 제품, 다중 로봇 시스템, 실시간 제어, 강화된 보안 등 산업계의 엄격한 요구사항을 충족시키기 위해 태어났다.6

이러한 변화의 배경에는 로봇 기술의 패러다임 전환이 있다. 더 이상 실험실의 연구 과제가 아닌, 공장, 물류창고, 도로, 심지어 우주에서까지 신뢰성과 안정성을 보장해야 하는 '제품'으로서의 로봇을 만들기 위한 필연적인 선택이었던 것이다. ROS2가 산업용 통신 미들웨어의 표준인 DDS를 핵심 아키텍처로 채택한 것은 이러한 변화를 상징적으로 보여준다. ROS2를 배운다는 것은 단순히 새로운 도구를 익히는 것을 넘어, 현대 로봇 산업의 표준을 이해하는 과정이다.

#### 1.2.1 핵심 아키텍처 변화: ROS Master의 제거와 DDS의 도입

ROS1과 ROS2의 가장 근본적인 차이는 통신 아키텍처에 있다. ROS1은 `roscore`라는 중앙 집중식 **마스터 노드**에 모든 노드가 의존하는 'Master-Slave' 구조였다.9 모든 노드는 시작할 때 마스터에 자신의 정보를 등록하고, 다른 노드의 정보를 마스터로부터 받아와야 통신이 가능했다. 이는 명백한 단점을 가졌는데, 바로 마스터가 다운되면 전체 로봇 시스템이 마비되는 **단일 장애점(Single Point of Failure)** 문제가 존재했다는 것이다.8

ROS2는 이 `roscore`를 과감히 제거했다.8 대신 산업 표준 통신 미들웨어인 **DDS(Data Distribution Service)**를 채택하여, 모든 노드가 서로를 자동으로 발견하고 직접 통신하는 완전한 **분산 시스템(P2P, Peer-to-Peer)**을 구현했다.5 이로 인해 특정 노드 하나가 중단되더라도 전체 시스템의 통신에는 영향을 미치지 않아 시스템의 강건성(robustness)이 획기적으로 향상되었다. 특히, 불안정한 무선 네트워크 환경에서 동작하는 다중 로봇 시스템이나 자율주행차 같은 애플리케이션에서 이점은 극대화된다.5

#### 1.2.2 주요 개선 사항 상세 분석

DDS 도입을 필두로, ROS2는 여러 방면에서 ROS1을 뛰어넘는 혁신을 이루었다.

- **플랫폼 확장성**: ROS1은 사실상 우분투 리눅스에서만 원활하게 동작했다. 반면, ROS2는 처음부터 멀티 플랫폼을 목표로 설계되어 리눅스는 물론 **윈도우, macOS**를 공식적으로 지원한다. 이를 통해 더 넓은 개발자 커뮤니티와 다양한 산업 분야로의 확장이 가능해졌다.6
- **실시간 제어(Real-Time Control)**: 정밀한 모터 제어나 고속 주행 중인 자율주행차의 반응 속도는 밀리초(ms) 단위의 시간 약속을 반드시 지켜야 한다. ROS1은 이러한 실시간성을 보장하지 못했다.6 ROS2는 DDS의 실시간 발행-구독 프로토콜(RTPS)과 실시간 운영체제(RTOS)와의 결합을 통해 **실시간 제어를 지원**하도록 설계되었다. 이는 로봇 기술이 안전이 최우선인 미션 크리티컬(mission-critical) 시스템에 적용될 수 있는 길을 열었다.7
- **보안 강화**: ROS1은 보안에 매우 취약했다. 네트워크에 연결된 누구나 IP 주소와 포트 번호만 알면 로봇의 모든 데이터에 접근하고 제어권을 탈취할 수 있었다.8 상업용 제품에서는 절대 용납될 수 없는 문제다. ROS2는 DDS-Security 표준을 통해 **인증(Authentication), 암호화(Encryption), 접근 제어(Access Control)** 등 강력하고 세분화된 보안 기능을 제공한다. 이를 통해 해킹이나 데이터 탈취로부터 로봇 시스템을 안전하게 보호할 수 있다.8
- **클라이언트 라이브러리(RCL) 구조 개선**: ROS1에서는 C++ 라이브러리인 `roscpp`와 Python 라이브러리인 `rospy`가 독립적으로 개발되어 기능이나 성능 면에서 차이가 있었다.9 ROS2는 C언어로 작성된 핵심 라이브러리 **`rcl(ROS Client Library)`** 위에 각 언어별 라이브러리(`rclcpp`, `rclpy` 등)를 구축하는 계층적 구조를 채택했다.10 이로써 언어 간 기능적 동등성을 보장하고, Java, C#, Rust 등 새로운 프로그래밍 언어를 지원하기가 훨씬 수월해졌다.9

| 항목                      | ROS1                        | ROS2                                | 핵심 변화의 의미                                  |
| ------------------------- | --------------------------- | ----------------------------------- | ------------------------------------------------- |
| **아키텍처**              | 중앙 집중형 (Master-Slave)  | 분산형 (Peer-to-Peer)               | 단일 장애점 제거, 시스템 안정성 및 확장성 증대    |
| **통신 미들웨어**         | TCPROS/UDPROS (자체 개발)   | DDS (산업 표준)                     | 실시간성, 보안, QoS 등 검증된 산업용 기능 활용    |
| **실시간성**              | 미지원                      | 지원 (RTOS 및 RTPS 결합 시)         | 정밀 제어 및 안전 필수 시스템에 적용 가능         |
| **보안**                  | 취약 (인증/암호화 부재)     | 강화 (DDS-Security 기반)            | 상업용 제품 수준의 보안 요구사항 충족             |
| **플랫폼 지원**           | 리눅스 중심                 | 멀티 플랫폼 (리눅스, 윈도우, macOS) | 개발 환경 및 적용 분야의 폭넓은 확장              |
| **클라이언트 라이브러리** | `roscpp`, `rospy` 독립 개발 | `rcl` 기반 계층 구조                | 언어 간 기능적 일관성 확보 및 신규 언어 지원 용이 |
| **빌드 시스템**           | Catkin                      | Ament + Colcon                      | 유연성 및 확장성 개선, Python 패키지 지원 강화    |
| **다중 로봇 지원**        | 제한적                      | 기본적으로 지원                     | 다개체 로봇 시스템 개발의 용이성 증대             |

### 1.3  Humble Hawksbill: 안정성을 선택해야 하는 이유

ROS2는 주기적으로 새로운 배포판(Distribution)을 릴리스한다. 배포판은 크게 두 종류로 나뉜다. 하나는 2년마다 출시되어 5년간 장기 지원되는 **LTS(Long Term Support)** 버전이고, 다른 하나는 6개월마다 새로운 기능이 추가되는 개발 버전(STS, Short Term Support)이다.18

**Humble Hawksbill (험블 혹스빌)**은 2022년 5월에 출시된 **LTS 버전**으로, **2027년 5월까지 공식 지원**된다.9 입문자에게 LTS 버전을 강력히 추천하는 이유는 다음과 같다.

- **안정성**: 수많은 테스트를 거쳐 안정성이 검증되었으며, 장기간 버그 수정과 보안 업데이트가 제공된다.
- **생태계 성숙도**: 대부분의 주요 라이브러리와 패키지들이 LTS 버전을 기준으로 개발되고 문서화된다. 학습 자료를 찾거나 커뮤니티의 도움을 받기 훨씬 수월하다.
- **호환성**: Humble은 **Ubuntu 22.04 (Jammy Jellyfish)를 Tier 1 플랫폼으로 공식 지원**한다.18 이는 운영체제와의 호환성이 가장 뛰어나고 예측하지 못한 문제를 겪을 확률이 적다는 것을 의미한다.

따라서, 지금 ROS2를 시작한다면 가장 안정적이고 성숙한 생태계를 갖춘 Humble과 Ubuntu 22.04 조합이 최상의 선택이다.

## 2.  ROS2 시스템의 해부: 핵심 개념 정복하기

ROS2 시스템은 마치 레고 블록처럼 여러 개의 작은 부품들이 모여 거대한 구조물을 이루는 것과 같다. 이 '부품'에 해당하는 핵심 개념들을 이해하는 것은 ROS2의 작동 방식을 파악하는 첫걸음이다. 이 섹션에서는 노드, 토픽, 서비스, 액션, 파라미터라는 5가지 핵심 개념을 쉬운 비유를 통해 완벽하게 정복해 본다.

### 2.1  모든 것은 노드(Node)에서 시작된다

**노드(Node)**는 ROS2 시스템을 구성하는 가장 기본적인 실행 단위다.21 하나의 노드는 특정 임무를 수행하는 독립적인 프로그램(프로세스)이라고 생각하면 된다. 예를 들어, 로봇 청소기는 '카메라 센서에서 장애물 인식', '바퀴 모터 제어', '배터리 상태 관리', '청소 경로 계획' 등 여러 가지 기능을 동시에 수행해야 한다. ROS2에서는 이 각각의 기능을 별개의 노드로 구현한다.

- 비유: 스마트폰의 '앱'

  스마트폰에 카메라 앱, 갤러리 앱, 지도 앱이 각자 독립적으로 실행되면서 고유한 기능을 수행하는 것처럼, ROS2 로봇 시스템도 '카메라 노드', '모터 제어 노드', '경로 계획 노드' 등이 각자의 역할을 담당한다.23 이렇게 기능을 잘게 쪼개어 모듈화하면 개발, 테스트, 재사용이 매우 용이해진다. 하나의 앱이 문제를 일으켜도 다른 앱은 정상적으로 동작하는 것처럼, 하나의 노드에 문제가 생겨도 다른 노드에 미치는 영향을 최소화할 수 있다.

하나의 C++ 또는 Python 실행 파일은 여러 개의 노드를 포함할 수도 있는데, 이를 **컴포넌트(Component)**라고 부른다.16 컴포넌트들은 동일한 프로세스 내에서 실행되므로 노드 간 통신 시 데이터 복사가 발생하지 않아 매우 효율적이다.

### 2.2  노드 간의 대화법: Topic, Service, Action

독립적인 노드들은 **ROS 그래프(Graph)**라는 가상의 네트워크를 통해 서로 정보를 주고받으며 협력한다.21 이 정보 교환 방식, 즉 노드 간의 '대화법'에는 크게 세 가지 종류가 있다. 어떤 대화법을 선택하느냐는 전달하려는 정보의 성격과 목적에 따라 달라진다.

이 세 가지 통신 방식의 구분은 단순히 기술적인 차이를 넘어, 로봇이 수행하는 작업의 '시간적 특성'과 '상호작용의 복잡성'에 따라 통신을 최적화하려는 ROS2의 깊은 설계 철학을 담고 있다. 로봇 시스템을 설계할 때 각 기능의 성격을 명확히 정의하고 그에 맞는 통신 방식을 선택하는 능력은 효율적이고 안정적인 시스템을 만드는 핵심 역량이다.

#### 2.2.1  토픽(Topic): 지속적인 정보의 흐름

- **개념**: **비동기식 단방향 통신** 채널이다. 정보를 보내는 **퍼블리셔(Publisher)**와 정보를 받는 **서브스크라이버(Subscriber)**가 존재한다.21 퍼블리셔는 특정 이름의 토픽에 계속해서 메시지를 발행하고, 해당 토픽을 구독하는 모든 서브스크라이버는 그 메시지를 받게 된다.

- 비유: '유튜브 채널' 또는 '그룹 채팅방'

  유튜버(퍼블리셔)는 자신의 채널(토픽)에 불특정 다수를 향해 계속해서 영상을 올린다. 구독자(서브스크라이버)들은 관심 있는 채널을 구독하고, 새로운 영상이 올라올 때마다 알림을 받는다.23 중요한 점은, 유튜버는 구독자가 누구인지, 몇 명인지, 지금 영상을 보고 있는지 전혀 신경 쓰지 않는다는 것이다. 그저 자신의 채널에 콘텐츠를 발행할 뿐이다. 토픽 통신도 이와 같다.

- **사용 시점**: 카메라 이미지, 라이다 스캔 데이터, 로봇의 현재 위치나 속도처럼 **지속적으로, 그리고 여러 노드가 동시에 알아야 할 필요가 있는(일대다, one-to-many) 데이터**를 전달하는 데 적합하다.25

#### 2.2.2  서비스(Service): 빠르고 확실한 요청과 응답

- **개념**: **동기식 양방향 통신**이다. 요청을 보내는 **클라이언트(Client)**와 요청을 처리하고 반드시 하나의 응답을 보내는 **서버(Server)**로 구성된다.21 클라이언트는 서버에 요청을 보낸 후, 응답이 올 때까지 기다린다(blocking).

- 비유: '자판기' 또는 '함수 호출'

  내가(클라이언트) 자판기(서버)에 돈을 넣고 버튼을 누르면(요청), 자판기는 반드시 음료수(응답)를 내어준다. 나는 음료수가 나올 때까지 그 앞에서 기다려야 한다.23 이처럼 서비스는 명확한 요청에 대한 확실한 단일 응답이 보장되는 일회성 작업에 사용된다.

- **사용 시점**: "현재 배터리 잔량을 알려줘", "두 숫자를 더한 결과를 반환해줘", "로봇 팔의 현재 관절 각도를 알려줘"처럼, **즉각적인 응답이 필요하고 비교적 짧은 시간 안에 끝나는 작업**에 적합하다.25

#### 2.2.3  액션(Action): 피드백이 있는 장기 목표 수행

- **개념**: **비동기식 양방향 통신**으로, 서비스의 확장판이라고 볼 수 있다. 클라이언트는 서버에 **목표(Goal)**를 전달하고, 서버는 목표를 수행하는 동안 주기적으로 **피드백(Feedback)**을 클라이언트에게 보내준다. 목표 수행이 완료되면 최종 **결과(Result)**를 반환하며, 클라이언트는 목표 수행 도중에 **취소(Cancel)**를 요청할 수도 있다.28

- 비유: '음식 배달 앱' 또는 '우버 호출'

  내가(클라이언트) 배달 앱(서버)으로 음식을 주문하면(목표 전달), 앱은 '주문 접수', '조리 중', '배달 시작'과 같은 진행 상황(피드백)을 실시간으로 알려준다. 최종적으로 음식이 도착하면 '배달 완료'(결과) 알림이 온다. 중간에 마음이 바뀌면 주문을 취소할 수도 있다.23 서비스와 가장 큰 차이점은, 주문을 해놓고 배달이 완료될 때까지 앱 화면만 쳐다보며 기다릴 필요 없이 다른 일을 할 수 있다는 점(비동기)과, 중간 과정을 계속 확인할 수 있다는 점(피드백)이다.

- **사용 시점**: "좌표 (10, 20)으로 이동해라", "로봇 팔로 저기 있는 파란색 물체를 집어서 상자에 넣어라"처럼, **완료하는 데 시간이 걸리고, 중간 과정의 모니터링이 필요하며, 취소될 가능성이 있는 장기적인 작업**에 적합하다.25

### 2.3  파라미터(Parameter): 노드의 설정을 바꾸다

- **개념**: **파라미터(Parameter)**는 노드가 실행되는 동안 외부에서 읽거나 수정할 수 있는 **설정값**이다.23 ROS1에서는 모든 파라미터를 중앙의 '파라미터 서버'가 관리했지만, ROS2에서는 각 노드가 자신의 파라미터를 독립적으로 관리하는 방식으로 변경되어 모듈성이 향상되었다.16

- 비유: '앱의 설정 메뉴'

  앱을 재시작하지 않고도 설정 메뉴에 들어가 알림 수신 여부나 화면 테마를 바꿀 수 있는 것처럼, 파라미터를 사용하면 노드를 재시작하지 않고도 로봇의 최대 이동 속도, 카메라의 해상도, 센서의 민감도 같은 값들을 동적으로 변경할 수 있다.23 이는 로봇을 튜닝하거나 여러 환경에 맞게 동작을 조정할 때 매우 유용하다.

## 3.  개발 환경 구축: 우분투 22.04에 Humble 설치하기

이론은 충분하다. 이제 직접 ROS2를 설치하고 개발 환경을 구축해 보자. 이 가이드는 ROS2 Humble과 가장 좋은 궁합을 자랑하는 Ubuntu 22.04 LTS(Jammy Jellyfish)를 기준으로 한다. 단계별 명령어를 그대로 따라 하면 누구나 쉽게 설치를 마칠 수 있다.

### 3.1  사전 준비: 시스템 환경 설정

ROS2를 설치하기 전에, 우리 시스템이 ROS2를 맞이할 준비가 되었는지 확인해야 한다.

- **운영체제**: **Ubuntu 22.04 (Jammy Jellyfish) 64-bit** 버전이 설치되어 있어야 한다. PC에 직접 설치(Native), 가상 머신(Virtual Machine), 또는 윈도우의 WSL2(Windows Subsystem for Linux 2) 환경 모두 가능하지만, 네트워크 설정 등 추가적인 복잡성을 피하기 위해 입문자에게는 PC에 직접 설치하는 것을 가장 권장한다.30

- **로케일(Locale) 설정**: ROS2는 문자 인코딩으로 UTF-8을 사용한다. 터미널을 열고 다음 명령어를 순서대로 입력하여 시스템 로케일을 `en_US.UTF-8`로 설정한다.

  ```Bash
  # 현재 로케일 확인 (출력에 UTF-8이 포함되어야 함)
  locale
  # 로케일 관련 패키지 설치 및 업데이트
  sudo apt update && sudo apt install locales
  # en_US.UTF-8 로케일 생성
  sudo locale-gen en_US en_US.UTF-8
  # 시스템 기본 로케일로 설정
  sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
  # 현재 터미널 세션에 환경 변수 적용
  export LANG=en_US.UTF-8
  ```

  이 과정은 시스템이 다양한 언어와 문자를 올바르게 처리하도록 보장한다.30

- **Ubuntu Universe 저장소 활성화**: ROS2 패키지가 의존하는 많은 소프트웨어들이 Ubuntu의 'Universe' 저장소에 포함되어 있다. 다음 명령어로 Universe 저장소를 활성화한다.

  ```Bash
  sudo apt install software-properties-common
  sudo add-apt-repository universe
  ```

  20

### 3.2  ROS2 Humble 설치: 단계별 명령어 가이드

이제 본격적으로 ROS2 Humble을 설치할 차례다.

- **ROS2 APT 저장소 추가**: ROS2 패키지를 다운로드할 공식 저장소 정보를 우리 시스템에 추가해야 한다. 이를 위해 먼저 GPG 키를 등록하여 패키지의 무결성을 보장하고, `sources.list`에 저장소 주소를 추가한다.

  ```Bash
  # curl 설치
  sudo apt update && sudo apt install curl -y
  # ROS 2 GPG 키 추가
  sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
  # sources.list에 저장소 추가
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
  ```

  30

- **시스템 업데이트 및 업그레이드**: 새로운 저장소 정보를 시스템에 반영하고, 설치된 모든 패키지를 최신 상태로 업데이트한다.

  ```Bash
  sudo apt update
  sudo apt upgrade
  ```

  **경고**: 이 과정은 매우 중요하다. 특히 새로 설치한 Ubuntu 22.04 시스템에서 `sudo apt upgrade`를 실행하지 않고 ROS2를 설치하면, 의존성 문제로 인해 `systemd`나 `udev` 같은 핵심 시스템 패키지가 제거되어 시스템이 부팅 불능 상태에 빠질 수 있다.33 반드시 `upgrade`를 먼저 실행해야 한다.

- **ROS2 패키지 설치**: 이제 `apt`를 통해 ROS2를 설치한다. 몇 가지 선택지가 있지만, 입문자에게는 `desktop` 버전을 강력히 권장한다.

  - **Desktop 버전 (권장)**: ROS2 핵심 기능뿐만 아니라 RViz2(3D 시각화 도구), Gazebo(시뮬레이터), 데모, 튜토리얼 등 개발에 유용한 모든 것이 포함되어 있다.30

    ```Bash
    sudo apt install ros-humble-desktop
    ```

  - **Base 버전**: GUI 도구 없이 최소한의 통신 라이브러리와 명령줄 도구만 설치한다. 서버 환경이나 임베디드 시스템에 적합하다.30

    ```Bash
    sudo apt install ros-humble-ros-base
    ```

- **핵심 개발 도구 설치**: ROS2 패키지를 직접 만들고 빌드하려면 컴파일러와 빌드 도구가 필요하다. 다음 명령어로 필수 개발 도구를 설치한다.

  ```Bash
  sudo apt install ros-dev-tools python3-colcon-common-extensions
  ```

  `colcon`은 ROS2의 표준 빌드 도구로, 이후 섹션에서 자세히 다룰 것이다.30

### 3.3  환경 설정 스크립트 적용

ROS2의 명령어(예: `ros2 run`)를 터미널에서 사용하려면, ROS2 환경이 설정되어 있어야 한다. 다음 명령어를 실행하여 환경을 설정할 수 있다.

```Bash
source /opt/ros/humble/setup.bash
```

하지만 터미널을 새로 열 때마다 이 명령어를 입력하는 것은 매우 번거롭다. 이 과정을 자동화하기 위해, 터미널이 시작될 때마다 자동으로 실행되는 `.bashrc` 파일에 이 명령어를 추가하자.

```Bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
```

이제 터미널을 새로 열면 자동으로 ROS2 환경이 설정된다.30

### 3.4  설치 확인: Talker-Listener 예제 실행

모든 설치가 성공적으로 끝났는지 확인해 보자. ROS2는 C++과 Python API가 모두 잘 작동하는지 간단히 테스트해볼 수 있는 예제 노드를 제공한다.

1. **첫 번째 터미널**을 열고 C++로 작성된 `talker` 노드를 실행한다. 이 노드는 "Hello World" 메시지를 주기적으로 발행(publish)한다.

   ```Bash
   ros2 run demo_nodes_cpp talker
   ```

2. **두 번째 터미널**을 열고 Python으로 작성된 `listener` 노드를 실행한다. 이 노드는 `talker`가 발행하는 메시지를 구독(subscribe)하여 화면에 출력한다.

   ```Bash
   ros2 run demo_nodes_py listener
   ```

첫 번째 터미널에 `Publishing: [...]` 메시지가 계속 출력되고, 두 번째 터미널에 `I heard: [...]` 메시지가 나타난다면, ROS2 Humble이 당신의 시스템에 성공적으로 설치된 것이다.20

`ros-humble-desktop` 버전을 설치하는 것은 단순히 편의를 위한 선택이 아니다. 이는 ROS2의 핵심 철학 중 하나인 '풍부한 도구 생태계(Tooling Ecosystem)'를 처음부터 경험하는 중요한 과정이다. `ros-base`만 설치하면 ROS2의 핵심 통신 기능만 사용할 수 있지만, `ros-desktop`에 포함된 RViz2, Gazebo, `rqt_graph`와 같은 도구들은 복잡한 로봇 시스템의 내부 동작을 시각적으로 분석하고 디버깅하는 데 필수적인 '인트로스펙션(Introspection)' 도구들이다.35 ROS의 진정한 강력함은 개별 노드의 성능뿐만 아니라, 이처럼 시스템 전체를 조망하고 문제를 해결할 수 있는 도구 생태계에서 나온다. 따라서 입문자가 처음부터 이 도구들에 익숙해지는 것은, 단순히 예제를 따라 하는 것을 넘어 ROS 개발자처럼 생각하고 문제를 해결하는 방식을 배우는 첫걸음이 될 것이다.

## 4.  워크스페이스와 빌드 시스템: 당신의 첫 작업 공간

ROS2 개발은 **워크스페이스(Workspace)**라는 특별한 디렉토리 안에서 이루어진다. 이곳은 당신의 모든 ROS2 프로젝트 소스 코드를 체계적으로 관리하고, **`colcon`**이라는 빌드 도구를 통해 컴파일하고 실행 가능한 형태로 만드는 공간이다. 이 섹션에서는 워크스페이스의 구조를 이해하고 빌드 과정을 마스터하여, 본격적인 개발을 위한 탄탄한 기초를 다진다.

### 4.1  워크스페이스(Workspace)란 무엇인가?

워크스페이스는 ROS2 프로젝트를 위한 최상위 디렉토리다. 여러 개의 패키지(Package, ROS2 코드의 기본 단위)를 한 곳에서 관리하고 빌드하는 작업 공간이라고 생각하면 된다.38

`colcon`으로 빌드를 수행하면 워크스페이스 내부에 다음과 같은 표준 디렉토리들이 생성된다.

- `src/`: **소스(Source) 디렉토리**. 당신이 작성하거나 다운로드한 모든 ROS2 패키지의 소스 코드는 이 디렉토리 안에 위치해야 한다. `colcon`은 빌드를 시작할 때 이 디렉토리 안을 탐색하여 빌드할 패키지들을 찾는다.38
- `build/`: **빌드(Build) 디렉토리**. `colcon`이 소스 코드를 컴파일하는 과정에서 생성되는 중간 파일들(예: CMake 캐시 파일, C++ 객체 파일)이 저장되는 곳이다. 디버깅 목적이 아니라면 직접 들여다볼 일은 거의 없다.41
- `install/`: **설치(Install) 디렉토리**. 빌드가 성공적으로 완료된 최종 결과물들이 설치되는 곳이다. 실행 파일, 공유 라이브러리, Python 모듈, 런치 파일, 메시지 정의 등 ROS2 시스템이 실제로 사용하는 모든 파일이 이곳에 모인다. 이 디렉토리 안에 있는 `setup.bash` 파일을 `source`해야만 우리가 만든 패키지를 ROS2 시스템이 인식하고 사용할 수 있다.41
- `log/`: **로그(Log) 디렉토리**. `colcon` 빌드 과정의 상세한 로그가 저장된다. 빌드가 실패했을 때, 이 디렉토리 안의 로그 파일을 열어보면 어떤 단계에서 무슨 오류가 발생했는지 정확히 파악할 수 있다.41

### 4.2  `colcon`: ROS2의 통합 빌드 도구

`colcon`은 ROS1의 `catkin_make`를 대체하는 ROS2의 표준 빌드 도구다.42

`colcon`의 핵심 역할은 워크스페이스 내의 여러 패키지들의 의존 관계를 자동으로 파악하여 올바른 순서대로 빌드해주는 것이다.44

**빌드 과정은 간단하다:**

1. 워크스페이스의 최상위 디렉토리(예: `~/ros2_ws/`)로 이동한다.
2. 터미널에 `colcon build` 명령어를 입력한다.
3. `colcon`은 `src/` 디렉토리 안의 모든 패키지를 탐색한다.
4. 각 패키지의 `package.xml` 파일을 읽어 패키지 간의 의존성을 분석하고, 빌드 순서를 결정한다. 예를 들어, 패키지 B가 패키지 A에 의존한다면, 반드시 패키지 A를 먼저 빌드한다.
5. 결정된 순서에 따라 각 패키지를 빌드하고, 그 결과물을 `build/`와 `install/` 디렉토리에 생성한다.

### 4.3  효율적인 개발을 위한 `colcon` 핵심 옵션

대규모 프로젝트를 다루거나 개발 속도를 높이기 위해 반드시 알아야 할 `colcon`의 핵심 옵션 두 가지가 있다.

- `--packages-select <package_name>`: 워크스페이스에 수십 개의 패키지가 있을 때, 매번 모든 패키지를 다시 빌드하는 것은 엄청난 시간 낭비다. 이 옵션을 사용하면 **지정한 패키지만 선택적으로 빌드**할 수 있다.46 예를 들어, 

  `my_robot_driver`라는 패키지만 수정했다면 `colcon build --packages-select my_robot_driver` 명령으로 해당 패키지만 빠르게 다시 빌드할 수 있다.

- `--symlink-install`: 이 옵션은 **개발 효율을 극적으로 향상시키는 마법**과 같다. Python 스크립트, 런치 파일, 설정 파일처럼 별도의 컴파일 과정이 필요 없는 파일을 수정했다고 가정해 보자. 일반적으로는 이 변경 사항을 `install/` 디렉토리에 반영하기 위해 매번 `colcon build`를 다시 실행해야 한다. 하지만 `--symlink-install` 옵션을 사용하면, `colcon`은 파일을 `install/` 디렉토리에 복사하는 대신, `src/` 디렉토리에 있는 원본 파일을 가리키는 **심볼릭 링크(Symbolic Link)**를 생성한다.42 그 결과, 

  `src/` 디렉토리에서 파일을 수정하고 저장하는 즉시, `install/` 디렉토리에도 변경 사항이 실시간으로 반영된다. 더 이상 사소한 수정을 위해 빌드를 기다릴 필요가 없는 것이다.

### 4.4  오버레이(Overlay)와 언더레이(Underlay)

ROS2 개발 환경은 계층 구조를 가진다. 우리가 `apt`로 설치한 기본 ROS2 시스템(예: `/opt/ros/humble`)을 **언더레이(Underlay)**라고 부른다.41 그리고 우리가 직접 만든 워크스페이스(예: `~/ros2_ws`)는 이 언더레이 위에 쌓는 **오버레이(Overlay)**가 된다.9

터미널에서 `source ~/ros2_ws/install/setup.bash` 명령을 실행하면, 우리 시스템은 `~/ros2_ws` 워크스페이스의 패키지를 `/opt/ros/humble`의 기본 패키지보다 우선적으로 찾게 된다. 이를 '오버레이가 언더레이를 가린다(shadows)'고 표현한다. 이 메커니즘을 통해 우리는 기존 ROS2 패키지의 기능을 수정하거나 확장하는 자신만의 버전을 만들어 테스트할 수 있다.

ROS2의 빌드 시스템은 '소스 코드'와 '설치된 실행 환경'을 명확히 분리하는 현대적인 소프트웨어 공학 원칙을 따른다. 이는 ROS1의 `devel` 공간이 야기했던 모호성을 해결한다. ROS1의 `catkin`은 `devel`이라는 공간을 만들어, 빌드 후 별도의 설치 과정 없이도 패키지를 사용할 수 있게 했다.10 이는 편리했지만, 소스 공간과 실제 실행 환경이 뒤섞여 패키지 의존성 관리를 복잡하게 만드는 원인이 되기도 했다. 반면, ROS2의 `colcon`은 이 `devel` 공간을 없애고, 모든 결과물을 `install` 공간에 명시적으로 설치하도록 강제한다.10 이는 '빌드'와 '설치' 단계를 명확히 분리하여, 배포 및 의존성 관리를 훨씬 깔끔하고 예측 가능하게 만든다. 하지만 Python 코드나 설정 파일을 수정할 때마다 매번 빌드-설치 과정을 거치는 것은 번거로울 수 있다. 바로 이 지점에서 `--symlink-install` 옵션이 빛을 발한다. 이 옵션은 ROS2의 깔끔한 환경 분리 철학의 이점을 누리면서도, ROS1 `devel` 공간의 편리함(빠른 코드 수정 및 테스트)을 동시에 얻는 절묘한 타협점이다. 따라서 초보자는 `colcon build --symlink-install`을 항상 사용하는 습관을 들이는 것이 좋다.

## 5.  ROS2 패키지 제작 실습

이제 워크스페이스와 빌드 시스템에 대한 이해를 바탕으로, 우리만의 ROS2 패키지를 직접 만들어 볼 차례다. 패키지는 ROS2 코드와 관련 파일들을 담는 컨테이너로, ROS2 개발의 가장 기본적인 구성 단위다.

### 5.1  패키지(Package)의 역할과 기본 구조

**패키지**는 ROS2 코드를 구성하는 기본 단위다. C++이나 Python 소스 코드, 런치 파일, 설정 파일, 메시지/서비스 정의 파일 등 프로젝트에 필요한 모든 자원을 하나의 논리적인 묶음으로 관리한다.45 잘 만들어진 패키지는 다른 사람들과 쉽게 공유하고 재사용할 수 있다.

모든 ROS2 패키지는 반드시 **`package.xml`** 파일을 포함해야 한다. 이 파일은 패키지의 '신분증' 또는 '명세서'와 같다. 여기에는 다음과 같은 핵심 메타 정보가 담겨 있다 45:

- **패키지 이름, 버전, 설명, 라이선스, 관리자 정보**
- **빌드 타입**: C++ 패키지는 `ament_cmake`, Python 패키지는 `ament_python`으로 지정한다.50
- **의존성(Dependencies)**: 이 패키지가 빌드되거나 실행되기 위해 필요한 다른 패키지들의 목록. `colcon`은 빌드 시 이 정보를 참조하여 빌드 순서를 결정하고, `rosdep`이라는 도구는 시스템에 필요한 라이브러리를 자동으로 설치해준다.

`package.xml`과 빌드 파일(`CMakeLists.txt` 또는 `setup.py`)은 단순히 코드를 컴파일하기 위한 설정 파일이 아니다. 이것들은 당신의 코드가 `colcon`, `rosdep`, `ros2 run` 등 ROS2 생태계 전체와 '소통'하기 위한 표준화된 인터페이스다. 이 파일들을 정확하게 작성하는 것은 당신의 코드가 ROS2라는 거대한 생태계의 일부로서 제대로 기능하고, 다른 사람들과 협업하며, 재사용될 수 있도록 만드는 핵심적인 약속이다. 따라서 이 파일들의 중요성을 인지하고 꼼꼼하게 관리하는 습관을 들여야 한다.

### 5.2  `ros2 pkg create` 명령어로 패키지 생성하기

매번 패키지 구조와 설정 파일을 수동으로 만드는 것은 번거롭다. ROS2는 `ros2 pkg create`라는 편리한 명령어를 제공하여 패키지의 기본 골격을 자동으로 생성해준다.46

먼저, 지난 섹션에서 만든 워크스페이스의 `src` 디렉토리로 이동한다.

```Bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
```

#### 5.2.1 C++ 패키지 생성 예시

C++로 퍼블리셔와 서브스크라이버 노드를 만들 `cpp_pubsub`이라는 이름의 패키지를 생성해 보자.

```Bash
ros2 pkg create --build-type ament_cmake --license Apache-2.0 cpp_pubsub --dependencies rclcpp std_msgs
```

- `--build-type ament_cmake`: 이 패키지가 CMake를 사용하는 C++ 패키지임을 명시한다.
- `--license Apache-2.0`: 패키지의 라이선스를 지정한다.
- `--dependencies rclcpp std_msgs`: 이 패키지가 `rclcpp`(ROS2 C++ 클라이언트 라이브러리)와 `std_msgs`(표준 메시지 타입) 패키지에 의존함을 명시한다. 이 옵션 덕분에 `package.xml`과 `CMakeLists.txt`에 필요한 의존성 설정이 자동으로 추가된다.54

#### 5.2.2 Python 패키지 생성 예시

이번에는 Python으로 동일한 기능을 하는 `py_pubsub` 패키지를 생성해 보자.

```Bash
ros2 pkg create --build-type ament_python --license Apache-2.0 py_pubsub --dependencies rclpy std_msgs
```

- `--build-type ament_python`: 이 패키지가 Python 패키지임을 명시한다.
- `--dependencies rclpy std_msgs`: Python 클라이언트 라이브러리인 `rclpy`와 `std_msgs`에 대한 의존성을 `package.xml`에 자동으로 추가한다.55

명령을 실행하면 `src` 디렉토리 안에 각 패키지 이름의 폴더가 생성되고, 그 안에 필요한 기본 파일들이 만들어진 것을 확인할 수 있다.

### 5.3  C++ 패키지 설정 (`CMakeLists.txt`)

C++ 패키지의 빌드 과정은 `CMakeLists.txt` 파일에 의해 제어된다. `ros2 pkg create`로 생성된 기본 파일에 몇 가지를 추가해야 한다.

- `find_package()`: `rclcpp`나 `std_msgs`처럼 우리 코드가 사용하는 외부 라이브러리(패키지)를 찾도록 CMake에 지시한다.56
- `add_executable()`: C++ 소스 코드 파일(`.cpp`)을 컴파일하여 실행 파일을 생성한다. 예를 들어, `add_executable(talker src/talker.cpp)`는 `talker.cpp` 파일을 컴파일하여 `talker`라는 실행 파일을 만든다.56
- `ament_target_dependencies()`: 위에서 생성한 실행 파일이 어떤 라이브러리(예: `rclcpp`)에 링크되어야 하는지 지정한다. 링크 과정을 통해 실행 파일은 라이브러리의 함수들을 사용할 수 있게 된다.56
- `install(TARGETS...)`: `colcon build`가 완료되었을 때, 생성된 실행 파일을 `install/lib/<package_name>/` 디렉토리에 복사하도록 지시한다. 이 과정이 있어야 `ros2 run` 명령어로 우리 노드를 실행할 수 있다.53

### 5.4  Python 패키지 설정 (`setup.py`)

Python 패키지는 `setuptools`라는 표준 Python 도구를 사용하여 빌드 및 설치된다.52

`setup.py` 파일이 C++의 `CMakeLists.txt`와 유사한 역할을 한다.

`setup.py` 파일에서 가장 중요한 부분은 `entry_points` 섹션이다.53 이 섹션은 우리의 Python 스크립트를 

`ros2 run` 명령어로 실행할 수 있는 '콘솔 스크립트'로 등록하는 역할을 한다.

```Python
entry_points={
    'console_scripts': [
        'talker = py_pubsub.talker_node:main',
        'listener = py_pubsub.listener_node:main',
    ],
},
```

위 예시는 `ros2 run py_pubsub talker`를 실행하면 `py_pubsub` 패키지 안의 `talker_node.py` 파일에 있는 `main` 함수를 실행하라고 알려주는 것이다. 이 설정을 통해 Python 스크립트는 ROS2 시스템의 정식 실행 파일로 취급된다.

## 6.  코드로 만나는 ROS2: Publisher와 Subscriber 작성

이론과 설정은 충분하다. 이제 직접 코드를 작성하여 ROS2의 핵심 통신 방식인 퍼블리셔(Publisher)와 서브스크라이버(Subscriber)를 구현해 볼 시간이다. C++과 Python 두 가지 언어로 모두 작성해 보면서, 언어는 다르지만 그 안에 담긴 ROS2의 핵심 개념과 구조는 동일하다는 것을 느껴보자.

C++와 Python 코드는 표면적으로 문법은 다르지만, 내부적으로는 `rclcpp::Node`와 `rclpy.node.Node`, `create_publisher`, `create_timer`, `spin` 등 동일한 ROS2 핵심 개념과 디자인 패턴을 공유한다. 이는 ROS2의 계층적 아키텍처 덕분으로, 두 라이브러리 모두 내부적으로는 공통 C 라이브러리인 `rcl`을 호출하기 때문이다.10 따라서 특정 언어의 문법에 매몰되기보다, 두 언어에 공통적으로 나타나는 이 핵심 패턴을 파악하는 데 집중하는 것이 중요하다. 이 패턴을 한번 익히면, 한 언어를 배운 후 다른 언어로 쉽게 전환할 수 있다.

### 6.1  C++로 Pub/Sub 노드 작성하기 (`rclcpp`)

앞서 생성한 `cpp_pubsub` 패키지를 사용한다. `ros2_ws/src/cpp_pubsub/src/` 디렉토리 안에 두 개의 C++ 파일을 생성하자.

#### 6.1.1 `publisher_member_function.cpp` (퍼블리셔 노드)

이 노드는 0.5초마다 "Hello, world! [숫자]" 형태의 메시지를 `topic`이라는 이름의 토픽으로 발행한다.

```C++
#include <chrono>
#include <functional>
#include <memory>
#include <string>

#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/string.hpp"

using namespace std::chrono_literals;

class MinimalPublisher : public rclcpp::Node
{
public:
  MinimalPublisher()
  : Node("minimal_publisher"), count_(0)
  {
    publisher_ = this->create_publisher<std_msgs::msg::String>("topic", 10);
    timer_ = this->create_wall_timer(
      500ms, std::bind(&MinimalPublisher::timer_callback, this));
  }

private:
  void timer_callback()
  {
    auto message = std_msgs::msg::String();
    message.data = "Hello, world! " + std::to_string(count_++);
    RCLCPP_INFO(this->get_logger(), "Publishing: '%s'", message.data.c_str());
    publisher_->publish(message);
  }
  rclcpp::TimerBase::SharedPtr timer_;
  rclcpp::Publisher<std_msgs::msg::String>::SharedPtr publisher_;
  size_t count_;
};

int main(int argc, char * argv)
{
  rclcpp::init(argc, argv);
  rclcpp::spin(std::make_shared<MinimalPublisher>());
  rclcpp::shutdown();
  return 0;
}
```

56

- **핵심 분석**:
  - `class MinimalPublisher : public rclcpp::Node`: `rclcpp::Node` 클래스를 상속받아 우리만의 노드 클래스를 정의한다.
  - `create_publisher<std_msgs::msg::String>("topic", 10)`: `std_msgs::msg::String` 타입의 메시지를 `topic`이라는 이름의 토픽으로 발행하는 퍼블리셔를 생성한다. `10`은 큐(Queue) 크기다.
  - `create_wall_timer(500ms,...)`: 500밀리초(0.5초)마다 `timer_callback` 함수를 호출하는 타이머를 생성한다.
  - `timer_callback()`: 메시지를 생성하고, `RCLCPP_INFO`로 로그를 출력한 뒤, `publisher_->publish(message)`를 통해 실제로 메시지를 발행한다.
  - `rclcpp::spin()`: 노드를 실행 상태로 만들고, 타이머 콜백과 같은 이벤트가 처리되도록 대기한다.

#### 6.1.2 `subscriber_member_function.cpp` (서브스크라이버 노드)

이 노드는 `topic` 토픽을 구독하고 있다가, 메시지가 들어오면 그 내용을 터미널에 출력한다.

```C++
#include <memory>

#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/string.hpp"
using std::placeholders::_1;

class MinimalSubscriber : public rclcpp::Node
{
public:
  MinimalSubscriber()
  : Node("minimal_subscriber")
  {
    subscription_ = this->create_subscription<std_msgs::msg::String>(
      "topic", 10, std::bind(&MinimalSubscriber::topic_callback, this, _1));
  }

private:
  void topic_callback(const std_msgs::msg::String & msg) const
  {
    RCLCPP_INFO(this->get_logger(), "I heard: '%s'", msg.data.c_str());
  }
  rclcpp::Subscription<std_msgs::msg::String>::SharedPtr subscription_;
};

int main(int argc, char * argv)
{
  rclcpp::init(argc, argv);
  rclcpp::spin(std::make_shared<MinimalSubscriber>());
  rclcpp::shutdown();
  return 0;
}
```

57

- **핵심 분석**:
  - `create_subscription<...>("topic", 10,...)`: `topic` 토픽을 구독하는 서브스크라이버를 생성한다.
  - `std::bind(&MinimalSubscriber::topic_callback, this, _1)`: 메시지가 수신되었을 때 호출될 콜백 함수로 `topic_callback`을 지정한다. `_1`은 수신된 메시지 객체를 콜백 함수의 인자로 전달하라는 의미다.
  - `topic_callback(const std_msgs::msg::String & msg)`: 메시지를 수신했을 때 실행되는 함수. `msg.data`를 통해 메시지 내용에 접근하고, 로그로 출력한다.

#### 6.1.3 빌드 설정 (`CMakeLists.txt` 및 `package.xml`)

`package.xml`에 `<depend>rclcpp</depend>`와 `<depend>std_msgs</depend>`가 있는지 확인한다. `CMakeLists.txt`에는 다음과 같이 두 개의 실행 파일을 만들고 설치하도록 설정해야 한다.

```CMake
#... (find_package for rclcpp and std_msgs)

add_executable(talker src/publisher_member_function.cpp)
ament_target_dependencies(talker rclcpp std_msgs)

add_executable(listener src/subscriber_member_function.cpp)
ament_target_dependencies(listener rclcpp std_msgs)

install(TARGETS
  talker
  listener
  DESTINATION lib/${PROJECT_NAME}
)

#... (ament_package())
```

57

### 6.2  Python으로 Pub/Sub 노드 작성하기 (`rclpy`)

이번에는 `py_pubsub` 패키지를 사용한다. `ros2_ws/src/py_pubsub/py_pubsub/` 디렉토리 안에 두 개의 Python 파일을 생성하자.

#### 6.2.1 `publisher_member_function.py` (퍼블리셔 노드)

```Python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class MinimalPublisher(Node):
    def __init__(self):
        super().__init__('minimal_publisher')
        self.publisher_ = self.create_publisher(String, 'topic', 10)
        timer_period = 0.5  # seconds
        self.timer = self.create_timer(timer_period, self.timer_callback)
        self.i = 0

    def timer_callback(self):
        msg = String()
        msg.data = 'Hello World: %d' % self.i
        self.publisher_.publish(msg)
        self.get_logger().info('Publishing: "%s"' % msg.data)
        self.i += 1

def main(args=None):
    rclpy.init(args=args)
    minimal_publisher = MinimalPublisher()
    rclpy.spin(minimal_publisher)
    minimal_publisher.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

58

- **핵심 분석**: C++ 버전과 구조가 거의 동일하다. `self.create_publisher(...)`로 퍼블리셔를, `self.create_timer(...)`로 타이머를 생성하고, 콜백 함수 `timer_callback`에서 `self.publisher_.publish(msg)`로 메시지를 발행한다.

#### 6.2.2 `subscriber_member_function.py` (서브스크라이버 노드)

```Python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class MinimalSubscriber(Node):
    def __init__(self):
        super().__init__('minimal_subscriber')
        self.subscription = self.create_subscription(
            String,
            'topic',
            self.listener_callback,
            10)
        self.subscription  # prevent unused variable warning

    def listener_callback(self, msg):
        self.get_logger().info('I heard: "%s"' % msg.data)

def main(args=None):
    rclpy.init(args=args)
    minimal_subscriber = MinimalSubscriber()
    rclpy.spin(minimal_subscriber)
    minimal_subscriber.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

58

- **핵심 분석**: `self.create_subscription(...)`으로 서브스크라이버를 생성하고, 콜백 함수로 `self.listener_callback`을 지정한다. 메시지가 수신되면 이 함수가 자동으로 호출된다.

#### 6.2.3 빌드 설정 (`setup.py` 및 `package.xml`)

`package.xml`에 `<exec_depend>rclpy</exec_depend>`와 `<exec_depend>std_msgs</exec_depend>`가 있는지 확인한다. `setup.py`의 `entry_points`를 다음과 같이 설정하여 두 스크립트를 실행 파일로 등록한다.

```Python
entry_points={
    'console_scripts': [
        'talker = py_pubsub.publisher_member_function:main',
        'listener = py_pubsub.subscriber_member_function:main',
    ],
},
```

58

### 6.3  빌드 및 실행, 통신 확인

1. 워크스페이스의 루트(`~/ros2_ws`)로 이동하여 두 패키지를 빌드한다.

   ```Bash
   colcon build --packages-select cpp_pubsub py_pubsub
   ```

   62

2. **첫 번째 터미널**을 열고, 워크스페이스 환경을 설정한 뒤 C++ 퍼블리셔를 실행한다.

   ```Bash
   source ~/ros2_ws/install/setup.bash
   ros2 run cpp_pubsub talker
   ```

3. **두 번째 터미널**을 열고, 환경 설정 후 Python 서브스크라이버를 실행한다.

   ```Bash
   source ~/ros2_ws/install/setup.bash
   ros2 run py_pubsub listener
   ```

   두 번째 터미널에서 첫 번째 터미널이 보내는 메시지를 수신하는 것을 확인할 수 있다. 반대로 Python 퍼블리셔와 C++ 서브스크라이버를 실행해도 동일하게 통신이 이루어진다. 이는 ROS2가 언어에 구애받지 않고 통신할 수 있음을 보여준다.62

## 7.  필수 커맨드 라인 도구(CLI) 마스터하기

ROS2 개발은 코드 작성만큼이나 시스템의 상태를 확인하고 디버깅하는 과정이 중요하다. ROS2는 이를 위해 강력하고 다양한 명령줄 인터페이스(CLI) 도구를 제공한다. 이 도구들을 능숙하게 사용하는 것은 개발 효율을 비약적으로 향상시킨다. 이 섹션에서는 가장 필수적인 CLI 명령어들을 마스터하여, 복잡한 로봇 시스템의 내부를 손바닥 보듯 들여다보는 능력을 기른다.

### 7.1  CLI의 기본 구조

모든 ROS2 명령어는 `ros2`로 시작하며, 다음과 같은 일관된 구조를 가진다.65

ros2 <command> <verb> [arguments]

- **command**: 작업의 대상을 지정한다 (예: `node`, `topic`, `service`).
- **verb**: 수행할 동작을 지정한다 (예: `list`, `info`, `echo`).
- **arguments**: 동사에 필요한 추가 정보를 제공한다 (예: 노드 이름, 토픽 이름).

어떤 명령어나 동사를 사용해야 할지 헷갈릴 때는, 명령어 뒤에 `-h` 또는 `--help`를 붙이면 상세한 도움말을 볼 수 있다.66

ROS2 CLI 도구들은 단순히 개별 기능을 제공하는 것을 넘어, 서로 유기적으로 연결되어 '탐색 -->> 분석 -->> 테스트'라는 체계적인 디버깅 워크플로우를 형성한다. 예를 들어, 로봇이 이상하게 움직일 때, 먼저 `ros2 node list`와 `ros2 topic list`로 시스템의 전체적인 구조를 파악하고('탐색'), 의심스러운 노드나 토픽을 `ros2 node info`와 `ros2 topic info`로 상세히 들여다본다('분석'). 데이터 자체에 문제가 있다고 판단되면 `ros2 topic echo`로 실제 값을 확인하고, `ros2 topic pub`으로 정상적인 값을 직접 주입해보며 문제를 국소화한다('테스트'). 이 워크플로우를 익히는 것이 단순히 명령어 목록을 암기하는 것보다 훨씬 중요하다.

### 7.2  시스템 상태 확인 및 분석 (Introspection)

- `ros2 node list`: 현재 실행 중인 모든 노드의 목록을 보여준다. 시스템이 어떤 모듈들로 구성되어 있는지 한눈에 파악할 수 있다.36
- `ros2 node info <node_name>`: 특정 노드가 어떤 토픽을 발행/구독하는지, 어떤 서비스를 제공/사용하는지 등 상세한 정보를 보여준다. 노드 간의 연결 관계를 파악하는 데 필수적이다.68
- `ros2 topic list`: 현재 활성화된 모든 토픽의 목록을 보여준다. `-t` 옵션을 추가하면 각 토픽의 메시지 타입도 함께 볼 수 있어 매우 유용하다.36
- `ros2 topic info <topic_name>`: 특정 토픽의 메시지 타입과, 현재 해당 토픽에 연결된 퍼블리셔와 서브스크라이버의 수를 보여준다. 통신이 제대로 연결되었는지 확인할 때 사용한다.71
- `ros2 service list`, `ros2 action list`: 현재 사용 가능한 서비스와 액션의 목록을 보여준다. `-t` 옵션으로 타입 정보도 함께 볼 수 있다.36

### 7.3  데이터 실시간 확인 및 전송 (Interaction)

- `ros2 topic echo <topic_name>`: **디버깅의 핵심**. 특정 토픽을 통해 실시간으로 주고받는 메시지의 내용을 터미널에 그대로 출력해준다. 센서 값이 제대로 들어오는지, 제어 명령이 올바르게 전달되는지 확인할 때 가장 빈번하게 사용된다.71
- `ros2 topic pub <topic_name> <msg_type> '<args>'`: 명령줄에서 직접 토픽에 메시지를 발행한다. 로봇을 수동으로 움직여보거나, 특정 노드의 입력값을 임의로 만들어 테스트할 때 매우 유용하다. 인자는 반드시 YAML 형식으로 입력해야 한다.71
- `ros2 service call <service_name> <srv_type> '<args>'`: 명령줄에서 직접 서비스를 호출하고 응답을 확인한다.73
- `ros2 topic hz <topic_name>`: 특정 토픽의 메시지 발행 주기(주파수, Hz)를 측정한다. 센서 데이터가 정해진 속도로 들어오고 있는지 확인할 때 유용하다.71

### 7.4  노드 실행 및 파라미터 관리

- `ros2 run <package_name> <executable_name>`: 패키지에 포함된 실행 파일을 실행하여 노드를 시작한다. 가장 기본적인 노드 실행 명령어다.68
- `ros2 param list <node_name>`: 특정 노드가 가지고 있는 모든 파라미터의 목록을 보여준다.78
- `ros2 param get <node_name> <param_name>`: 특정 파라미터의 현재 값을 읽어온다.36
- `ros2 param set <node_name> <param_name> <value>`: 노드를 재시작하지 않고도 파라미터 값을 실시간으로 변경한다. 로봇의 동작을 튜닝할 때 매우 강력한 기능이다.36
- `ros2 param dump <node_name> > params.yaml`: 노드의 현재 파라미터 값들을 YAML 파일 형태로 저장한다. 최적의 튜닝값을 찾았을 때 저장해두고 나중에 다시 불러올 수 있다.78

| 구분         | 명령어              | 설명                                                   | 사용 예시                                                    |
| ------------ | ------------------- | ------------------------------------------------------ | ------------------------------------------------------------ |
| **노드**     | `ros2 run`          | 패키지의 실행 파일을 실행하여 노드를 시작한다.         | `ros2 run turtlesim turtlesim_node`                          |
|              | `ros2 node list`    | 현재 실행 중인 모든 노드의 목록을 보여준다.            | `ros2 node list`                                             |
|              | `ros2 node info`    | 특정 노드의 상세 정보(Pub/Sub, Service 등)를 보여준다. | `ros2 node info /turtlesim`                                  |
| **토픽**     | `ros2 topic list`   | 활성화된 모든 토픽 목록을 보여준다. (`-t`로 타입 표시) | `ros2 topic list -t`                                         |
|              | `ros2 topic echo`   | 특정 토픽의 메시지 내용을 실시간으로 출력한다.         | `ros2 topic echo /turtle1/cmd_vel`                           |
|              | `ros2 topic pub`    | 명령줄에서 토픽으로 메시지를 직접 발행한다.            | `ros2 topic pub /turtle1/cmd_vel geometry_msgs/Twist "{linear: {x: 1.0}}"` |
|              | `ros2 topic info`   | 특정 토픽의 정보(타입, Pub/Sub 수)를 보여준다.         | `ros2 topic info /turtle1/cmd_vel`                           |
| **서비스**   | `ros2 service list` | 사용 가능한 모든 서비스 목록을 보여준다.               | `ros2 service list`                                          |
|              | `ros2 service call` | 명령줄에서 서비스를 호출하고 응답을 받는다.            | `ros2 service call /clear std_srvs/srv/Empty`                |
| **액션**     | `ros2 action list`  | 사용 가능한 모든 액션 목록을 보여준다.                 | `ros2 action list`                                           |
| **파라미터** | `ros2 param list`   | 특정 노드의 모든 파라미터 목록을 보여준다.             | `ros2 param list /turtlesim`                                 |
|              | `ros2 param get`    | 특정 파라미터의 값을 읽는다.                           | `ros2 param get /turtlesim background_g`                     |
|              | `ros2 param set`    | 특정 파라미터의 값을 실시간으로 변경한다.              | `ros2 param set /turtlesim background_r 150`                 |

## 8.  시각화와 시뮬레이션 입문

백문이 불여일견(百聞不如一見). 복잡한 로봇 시스템을 개발할 때, 수많은 숫자와 텍스트 로그만으로는 시스템이 어떻게 동작하는지 직관적으로 파악하기 어렵다. 이때 **RViz2**와 **Gazebo**는 우리의 눈이 되어준다. RViz2는 보이지 않는 데이터를 시각화하고, Gazebo는 실제 로봇 없이도 가상 환경에서 로봇을 테스트할 수 있게 해준다. 이 둘의 조합은 현대 로봇 개발의 핵심 워크플로우를 완성한다.

### 8.1  RViz2: 로봇의 눈

**RViz2**는 ROS2 데이터를 3D 공간에 시각화하는 매우 강력한 도구다.37 RViz2를 통해 우리는 다음과 같은 것들을 눈으로 직접 확인할 수 있다.

- 로봇의 3D 모델과 각 관절의 움직임
- 라이다(LiDAR)나 뎁스 카메라가 측정한 포인트 클라우드 데이터
- 카메라 이미지
- 로봇의 각 부분(바퀴, 몸체, 팔 등)의 위치 관계를 나타내는 좌표계(TF)
- 로봇이 이동하려는 경로

#### 8.1.1 기본 인터페이스

RViz2를 처음 실행하면 다소 복잡해 보일 수 있지만, 핵심적인 몇 가지 요소만 이해하면 금방 익숙해질 수 있다.37

- **3D View (중앙)**: 모든 시각화 결과물이 표시되는 메인 화면이다.37
- **Displays Panel (좌측)**: 시각화할 데이터의 종류를 추가하고 설정하는 가장 중요한 패널이다. 좌측 하단의 `Add` 버튼을 눌러 새로운 **디스플레이(Display)**를 추가할 수 있다. 예를 들어, 로봇 모델을 보려면 `RobotModel` 디스플레이를, 라이다 데이터를 보려면 `LaserScan` 또는 `PointCloud2` 디스플레이를 추가하고, 해당 데이터가 발행되는 토픽 이름을 지정해주어야 한다.37
- **Global Options (Displays 패널 상단)**: 시각화의 기준이 되는 **`Fixed Frame`**을 설정한다. RViz2의 모든 데이터는 이 기준 좌표계에 맞춰서 표시된다. 로봇의 움직임을 보려면 보통 `odom`이나 `map` 프레임을 기준으로 설정한다.37
- **Views Panel (우측 또는 하단)**: 3D View의 카메라 시점을 제어한다. 마우스로 자유롭게 회전하고 확대/축소하는 `Orbital` 뷰, 위에서 내려다보는 `Top-down` 뷰 등이 있다.37
- **Toolbar (상단)**: `2D Nav Goal` (내비게이션 목표 지점 설정), `2D Pose Estimate` (로봇의 초기 위치 추정) 등 사용자가 시각화 환경과 상호작용할 수 있는 여러 도구를 제공한다.37

### 8.2  Gazebo: 가상 세계의 로봇

**Gazebo**는 물리 엔진을 갖춘 고성능 3D 로봇 시뮬레이터다.83 비싸고 파손 위험이 있는 실제 로봇 없이도, 가상 환경에서 로봇 모델을 움직여보고 센서 데이터를 생성하며 알고리즘을 안전하고 빠르게 테스트할 수 있다.

#### 8.2.1 ROS2와의 연동: `ros_gz_bridge`

Gazebo는 그 자체로 독립적인 시뮬레이션 프로그램이지만, ROS2와 데이터를 주고받으며 긴밀하게 연동될 수 있다. 이 둘을 연결하는 핵심적인 다리 역할을 하는 것이 바로 **`ros_gz_bridge`** 패키지다.84

Gazebo는 자체적인 통신 시스템인 'Gazebo Transport'를 사용하고, ROS2는 DDS를 사용한다. `ros_gz_bridge`는 이 두 개의 서로 다른 통신 시스템 사이에서 메시지를 번역하고 전달해주는 역할을 한다. 예를 들어, Gazebo 시뮬레이터의 가상 라이다 센서가 생성한 데이터를 ROS2의 `sensor_msgs/msg/LaserScan` 타입의 토픽으로 변환하여 발행해주거나, ROS2 노드가 발행한 `geometry_msgs/msg/Twist` 타입의 속도 제어 명령을 Gazebo의 로봇 모델에 전달하여 움직이게 할 수 있다.84

RViz2와 Gazebo의 연동은 '시뮬레이션 기반 개발(Simulation-driven Development)'이라는 현대 로봇 개발의 핵심 워크플로우를 가능하게 한다. 과거에는 로봇 알고리즘을 개발하고 테스트하려면 항상 실제 로봇 하드웨어가 필요했다. 이는 비용과 시간이 많이 들고, 테스트 중 하드웨어 파손의 위험도 컸다. 하지만 이제는 Gazebo를 통해 물리적으로 정확한 가상 로봇으로 알고리즘을 충분히 테스트하고, RViz2를 통해 시뮬레이션 내부에서 보이지 않는 데이터(센서 값, 좌표계, 계획된 경로 등)를 시각화하며 알고리즘이 의도대로 작동하는지 직관적으로 디버깅할 수 있다. 이렇게 시뮬레이션 환경에서 충분히 검증된 소프트웨어를 실제 로봇에 적용함으로써, 개발 비용과 시간을 획기적으로 줄이고 안전성을 높일 수 있다. 이 워크플로우를 이해하고 활용하는 것은 로봇 개발자로서 반드시 갖춰야 할 역량이다.

## 9.  더 깊은 이해를 향하여: DDS, QoS, 그리고 실시간성

지금까지 ROS2의 기본적인 사용법을 익혔다. 하지만 진정한 전문가로 성장하기 위해서는 그 내부 동작 원리에 대한 깊은 이해가 필요하다. 이 마지막 섹션에서는 ROS2 통신의 심장인 **DDS**, 통신의 품질을 제어하는 **QoS**, 그리고 고성능 로봇 시스템의 필수 조건인 **실시간성**이라는 세 가지 고급 주제를 소개한다. 입문 단계에서 이 모든 것을 완벽히 마스터할 필요는 없지만, 이러한 개념이 왜 존재하고 어떤 가능성을 열어주는지 이해하는 것은 앞으로의 학습 방향을 설정하는 데 큰 도움이 될 것이다.

### 9.1  ROS2 통신의 심장, DDS(Data Distribution Service)

앞서 언급했듯이, ROS2는 통신 미들웨어로 **DDS**를 채택했다. DDS는 단순히 메시지를 주고받는 프로토콜이 아니라, 데이터 자체를 중심으로(Data-Centric) 통신하는 발행-구독(Publish-Subscribe) 모델을 기반으로 하는 강력한 **산업 표준**이다.11 항공, 국방, 금융 등 극도의 신뢰성이 요구되는 시스템에서 오랫동안 사용되어 온 기술이다.

ROS2는 **RMW(ROS Middleware Interface)**라는 추상화 계층을 두고 있어, 특정 DDS 구현체에 종속되지 않는다.88 덕분에 개발자는 필요에 따라 eProsima의 **Fast DDS(기본값)**, RTI의 **Connext DDS**, Eclipse의 **Cyclone DDS** 등 다양한 DDS 벤더의 구현체를 선택하거나 심지어 Zenoh와 같은 비-DDS 미들웨어로 교체할 수도 있다.88 이러한 유연성은 ROS2의 큰 장점 중 하나다.

### 9.2  통신의 품질을 제어하는 QoS(Quality of Service)

DDS의 가장 강력하고 독특한 기능은 바로 **QoS(서비스 품질)** 설정이다. QoS는 개발자가 각 퍼블리셔와 서브스크라이버의 통신 방식을 매우 세밀하게 제어할 수 있게 해주는 정책들의 집합이다.15 ROS1의 TCP 기반 통신이 '모든 메시지를 안정적으로 전달'하는 한 가지 방식만 제공했다면, ROS2의 QoS는 '어떤 종류의 데이터를, 얼마나 중요하게, 어떻게 전달할 것인가'를 직접 설계할 수 있는 권한을 개발자에게 부여한다.

주요 QoS 정책 몇 가지를 살펴보자.

- **Reliability (신뢰성)**: 메시지 전달을 보장할지 여부를 결정한다.
  - `RELIABLE`: TCP처럼, 메시지가 수신자에게 전달되었는지 확인하고 유실 시 재전송을 시도한다. 로봇의 이동 명령이나 비상 정지 신호처럼 **절대 유실되어서는 안 되는 중요한 데이터**에 사용된다.90
  - `BEST_EFFORT`: UDP처럼, 메시지를 한번 보내고 전달 여부를 확인하지 않는다. 네트워크 상태가 나쁘면 유실될 수 있지만, 속도가 빠르고 오버헤드가 적다. 초당 수십 번씩 들어오는 카메라 이미지나 라이다 데이터처럼, **최신 데이터가 중요하고 일부 유실은 감수할 수 있는 센서 데이터**에 적합하다.90
- **Durability (지속성)**: 퍼블리셔가 발행한 메시지를 얼마나 오래 보관할지 결정한다.
  - `TRANSIENT_LOCAL`: 퍼블리셔가 최근에 발행한 메시지들을 보관하고 있다가, **나중에 새로 참여하는 서브스크라이버에게 전달**해준다. 로봇의 현재 상태나 완성된 지도 데이터처럼, 구독을 시작하는 시점의 최신 정보가 중요한 경우에 매우 유용하다.90
  - `VOLATILE`: 메시지를 보관하지 않는다. 서브스크라이버는 자신이 구독을 시작한 이후에 발행된 메시지만 받을 수 있다.90
- **History & Depth**: 메시지 큐에 메시지를 얼마나 보관할지 설정한다. `keep_last`는 `depth`로 지정된 개수만큼 최신 메시지만 보관하고, `keep_all`은 리소스 한도 내에서 모든 메시지를 보관하려고 시도한다.90

이 외에도 Deadline(메시지 도착 시간 제한), Lifespan(메시지 유효 기간), Liveliness(노드 생존 확인) 등 다양한 QoS 정책이 있다.91 ROS2는 '센서 데이터용', '서비스 기본값' 등 일반적인 사용 사례에 맞는 QoS 프로파일을 미리 정의해두어, 초보자도 쉽게 적절한 설정을 선택할 수 있도록 돕는다.92

### 9.3  실시간(Real-Time) 시스템을 향한 길

많은 사람들이 '실시간'을 '매우 빠름'과 혼동하지만, 공학에서 **실시간 시스템**이란 '빠른' 시스템이 아니라, 정해진 시간(**Deadline**) 안에 작업을 완료하는 것을 **'보장'**하는 **결정론적(Deterministic)** 시스템을 의미한다.13 예를 들어, 자동차의 에어백은 충돌 후 수십 밀리초 안에 반드시 터져야만 의미가 있다. 0.1초만 늦어도 치명적인 결과를 낳는다.

ROS2는 그 자체로 실시간 시스템은 아니지만, **실시간 시스템을 구축할 수 있도록 설계**되었다. 진정한 실시간 성능을 달성하기 위해서는 ROS2뿐만 아니라 시스템의 모든 계층에서 결정론적인 동작이 보장되어야 한다.96

- **실시간 운영체제(RTOS)**: 리눅스 커널에 `RT_PREEMPT` 패치를 적용하여 선점형 스케줄링이 가능하도록 만들어야 한다.13
- **결정론적 메모리 관리**: 실시간으로 동작해야 하는 코드 경로에서는 예측 불가능한 지연을 유발하는 동적 메모리 할당(`new`, `malloc`)이나 페이지 폴트(page fault)를 피해야 한다.13
- **실시간 통신**: DDS의 RTPS(Real-Time Publish-Subscribe) 프로토콜과 적절한 QoS 설정을 활용해야 한다.10
- **실시간 안전(Real-time Safe) 코드 작성**: 코드 자체가 예측 가능한 시간 안에 실행되도록 작성되어야 한다.

입문자 단계에서는 이러한 복잡한 요소들을 모두 다룰 필요는 없다. 다만, ROS2가 단순한 로봇 제어를 넘어, 안전과 신뢰성이 극도로 중요한 고성능 실시간 시스템을 구축할 수 있는 잠재력을 가진 강력한 프레임워크라는 점을 이해하는 것이 중요하다.

QoS와 실시간성 지원은 ROS2가 '최선을 다하는(best-effort)' 연구용 프레임워크를 넘어, '성능을 보장해야 하는(guaranteed-performance)' 산업용 프레임워크로 도약했음을 보여주는 핵심 증거다. 이는 개발자에게 더 많은 제어 권한과 동시에 더 큰 책임을 부여한다. ROS2 개발자는 이제 단순한 '사용자'를 넘어, 시스템의 통신 동작까지 정밀하게 설계하는 '시스템 아키텍트'의 역할을 수행해야 한다. 이것이 바로 당신이 앞으로 나아가야 할 방향이다. 이 안내서가 그 여정의 훌륭한 첫걸음이 되기를 바란다.

#### **참고 자료**

1. ROS2 이론 정리 - 개발새발 - 티스토리, accessed July 26, 2025, [https://kfdd6630.tistory.com/entry/ROS2-%EC%9D%B4%EB%A1%A0-%EC%A0%95%EB%A6%AC](https://kfdd6630.tistory.com/entry/ROS2-이론-정리)
2. ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/index.html
3. 로봇공학자, accessed July 26, 2025, https://rosmaster.tistory.com/
4. ROS 2 Key Challenges and Advances: A Survey of ROS 2 Research, Libraries, and Applications - ResearchGate, accessed July 26, 2025, https://www.researchgate.net/publication/384942758_ROS_2_Key_Challenges_and_Advances_A_Survey_of_ROS_2_Research_Libraries_and_Applications
5. ROS1이랑 ROS2, 뭐가 다른지 1분 안에 정리해드립니다! - YouTube, accessed July 26, 2025, https://www.youtube.com/shorts/_-P3tGw4b_M
6. ROS1 / ROS2 | Youngman, accessed July 26, 2025, https://eeoon.github.io/devops/ros-content/
7. eeoon.github.io, accessed July 26, 2025, [https://eeoon.github.io/devops/ros-content/#:~:text=ROS1%EC%9D%80%202020%EB%85%84%20%EC%9D%B4%ED%9B%84,%EA%B0%80%EB%8A%A5%ED%95%98%EB%8B%A4%EB%8A%94%20%EC%9E%A5%EC%A0%90%EC%9D%B4%20%EC%9E%88%EB%8B%A4.](https://eeoon.github.io/devops/ros-content/#:~:text=ROS1은 2020년 이후,가능하다는 장점이 있다.)
8. ROS2 시작하기(1) - 왜 ROS2를 사용해야할까?, install - 퇴근할게요 교수님, accessed July 26, 2025, https://run2run.tistory.com/10
9. ROS 1 vs ROS 2 What are the Biggest Differences? - Model Prime, accessed July 26, 2025, https://www.model-prime.com/blog/ros-1-vs-ros-2-what-are-the-biggest-differences
10. 로봇 소프트웨어 - ROS에 대해 좀 더 자세히(ROS1 & ROS2) - 지식 맛집 - 티스토리, accessed July 26, 2025, https://tristanchoi.tistory.com/584
11. ROS on DDS - ROS2 Design, accessed July 26, 2025, https://design.ros2.org/articles/ros_on_dds.html
12. Changes between ROS 1 and ROS 2 - ROS2 Design, accessed July 26, 2025, http://design.ros2.org/articles/changes.html
13. Introduction to Real-time Systems - ROS2 Design, accessed July 26, 2025, https://design.ros2.org/articles/realtime_background.html
14. ROS 2 DDS-Security integration - ROS2 Design, accessed July 26, 2025, https://design.ros2.org/articles/ros2_dds_security.html
15. ROS 2: What is DDS | Data Distribution Service (DDS) Community RTI Connext Users, accessed July 26, 2025, https://community.rti.com/page/ros-2-what-dds
16. ROS1 vs ROS2, Practical Overview For ROS Developers - The Robotics Back-End, accessed July 26, 2025, https://roboticsbackend.com/ros1-vs-ros2-practical-overview/
17. A first look at ROS 2 applications written in asynchronous Rust - arXiv, accessed July 26, 2025, https://arxiv.org/html/2505.21323v2
18. '로봇 이야기/ros2' 카테고리의 글 목록, accessed July 26, 2025, [https://duvallee.tistory.com/category/%EB%A1%9C%EB%B4%87%20%EC%9D%B4%EC%95%BC%EA%B8%B0/ros2](https://duvallee.tistory.com/category/로봇 이야기/ros2)
19. ROS documentation, accessed July 26, 2025, https://docs.ros.org/
20. Ubuntu (source) - ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/Installation/Alternatives/Ubuntu-Development-Setup.html
21. 개발자와 함께하는 ROS2 Humble Node 이해하기, ROS2 명령어-2 - velog, accessed July 26, 2025, [https://velog.io/@i_robo_u/ROS2-Humble-Node-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-ROS2-%EB%AA%85%EB%A0%B9%EC%96%B4-2](https://velog.io/@i_robo_u/ROS2-Humble-Node-이해하기-ROS2-명령어-2)
22. [ROS2] 개념 및 용어 정리 - velog, accessed July 26, 2025, [https://velog.io/@running_learning/ROS2-%EA%B0%9C%EB%85%90-%EB%B0%8F-%EC%9A%A9%EC%96%B4-%EC%A0%95%EB%A6%AC](https://velog.io/@running_learning/ROS2-개념-및-용어-정리)
23. From Zero to Robotics Hero: A Beginner's Guide to ROS 2 | by Riya ..., accessed July 26, 2025, https://riyagoja.medium.com/from-zero-to-robotics-hero-a-beginners-guide-to-ros-2-90ac9c3b87ba
24. Understanding ROS 2 topics - ROS documentation, accessed July 26, 2025, https://docs.ros.org/en/dashing/Tutorials/Topics/Understanding-ROS2-Topics.html
25. Topics vs Services vs Actions - ROS 2 Documentation: Foxy documentation, accessed July 26, 2025, https://docs.ros.org/en/foxy/How-To-Guides/Topics-Services-Actions.html
26. [ROS2 How-to] ROS2 Topics vs Services vs Actions #5 - The Construct, accessed July 26, 2025, https://www.theconstruct.ai/ros2-how-to-ros2-topics-vs-services-vs-actions-5/
27. Getting Started with ROS2: What is a ROS2 Service? Implementing ROS2 Service-Client Nodes in Python | by Sagar Kumar | Spinor | Medium, accessed July 26, 2025, https://medium.com/spinor/getting-started-with-ros2-what-is-a-ros2-service-implementing-ros2-service-client-nodes-in-python-e59456a1bd92
28. Actions - ROS2 Design, accessed July 26, 2025, https://design.ros2.org/articles/actions.html
29. Creating ROS 2 Actions - Foxglove, accessed July 26, 2025, https://foxglove.dev/blog/creating-ros2-actions
30. Install ROS 2 Humble with Create 3 Messages on an Ubuntu 22.04 Machine - GitHub Pages, accessed July 26, 2025, https://iroboteducation.github.io/create3_docs/setup/ubuntu2204/
31. RoArm-M2-S 10. Ubuntu 22.04 Install ROS2 Humble Tutorial - Waveshare Wiki, accessed July 26, 2025, https://www.waveshare.com/wiki/RoArm-M2-S_10._Ubuntu_22.04_Install_ROS2_Humble_Tutorial
32. Ubuntu 22.04에 ROS2 humble과 Gazebo 설치 - For a better world - 티스토리, accessed July 26, 2025, https://roytravel.tistory.com/378
33. Ubuntu (deb packages) - ROS 2 Documentation: Humble ..., accessed July 26, 2025, https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debs.html
34. ubuntu 22.04에 ROS 2 humble 설치 + colcon + Gazebo - Develop me - 티스토리, accessed July 26, 2025, https://cobang.tistory.com/104
35. 개발자와 함께하는 ROS2 Humble Topic 이해하기, ROS2 명령어-3 - velog, accessed July 26, 2025, [https://velog.io/@i_robo_u/%EA%B0%9C%EB%B0%9C%EC%9E%90%EC%99%80-%ED%95%A8%EA%BB%98%ED%95%98%EB%8A%94-ROS2-Humble-Topic-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-ROS2-%EB%AA%85%EB%A0%B9%EC%96%B4-3](https://velog.io/@i_robo_u/개발자와-함께하는-ROS2-Humble-Topic-이해하기-ROS2-명령어-3)
36. [ROS2] 명령어(패키지, 노드, 토픽, 서비스, 액션, 파라미터) - 기계공학도 LUKE, accessed July 27, 2025, [https://informluke.tistory.com/entry/ROS2-%EB%AA%85%EB%A0%B9%EC%96%B4%ED%8C%A8%ED%82%A4%EC%A7%80-%EB%85%B8%EB%93%9C-%ED%86%A0%ED%94%BD-%EC%84%9C%EB%B9%84%EC%8A%A4-%EC%95%A1%EC%85%98](https://informluke.tistory.com/entry/ROS2-명령어패키지-노드-토픽-서비스-액션)
37. RViz User Guide - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/RViz/RViz-User-Guide/RViz-User-Guide.html
38. How to create a ROS2 Workspace - The Construct, accessed July 26, 2025, https://www.theconstruct.ai/ros2-in-5-mins-007-how-to-create-a-ros2-overlay-workspace/
39. Create ROS Workspace - Industrial Training documentation, accessed July 26, 2025, https://industrial-training-master.readthedocs.io/en/foxy/_source/session1/ros2/1-Create-ROS-Workspace.html
40. How to Create a Workspace in ROS 2 Jazzy - Automatic Addison, accessed July 26, 2025, https://automaticaddison.com/how-to-create-a-workspace-in-ros2-jazzy/
41. 개발자와 함께하는 ROS2 Humble에서 작업공간 생성하기 - velog, accessed July 26, 2025, [https://velog.io/@i_robo_u/%EA%B0%9C%EB%B0%9C%EC%9E%90%EC%99%80-%ED%95%A8%EA%BB%98%ED%95%98%EB%8A%94-ROS2-Humble%EC%97%90%EC%84%9C-%EC%9E%91%EC%97%85%EA%B3%B5%EA%B0%84-%EC%83%9D%EC%84%B1%ED%95%98%EA%B8%B0](https://velog.io/@i_robo_u/개발자와-함께하는-ROS2-Humble에서-작업공간-생성하기)
42. Using colcon to build packages - ROS 2 Documentation, accessed July 26, 2025, https://docs.ros.org/en/crystal/Tutorials/Colcon-Tutorial.html
43. 패키지 빌드 - velog, accessed July 26, 2025, [https://velog.io/@mcleroysane19/%ED%8C%A8%ED%82%A4%EC%A7%80-%EB%B9%8C%EB%93%9C](https://velog.io/@mcleroysane19/패키지-빌드)
44. A universal build tool - ROS2 Design, accessed July 26, 2025, https://design.ros2.org/articles/build_tool.html
45. 개발자와 함께하는 ROS2 Humble에서 패키지 만들기 - velog, accessed July 26, 2025, [https://velog.io/@i_robo_u/%EA%B0%9C%EB%B0%9C%EC%9E%90%EC%99%80-%ED%95%A8%EA%BB%98%ED%95%98%EB%8A%94-ROS2-Humble%EC%97%90%EC%84%9C-%ED%8C%A8%ED%82%A4%EC%A7%80-%EB%A7%8C%EB%93%A4%EA%B8%B0](https://velog.io/@i_robo_u/개발자와-함께하는-ROS2-Humble에서-패키지-만들기)
46. Create a Package in ROS 2 - ENPM 662 - Read the Docs, accessed July 26, 2025, https://enpm-662introduction-to-robot-modelling.readthedocs.io/en/latest/package_creation.html
47. ROS2 Creating a package 2 [ROS2 패키지 만들기 2] - Intuitive-Robotics - 티스토리, accessed July 26, 2025, https://intuitive-robotics.tistory.com/175
48. 04장 Build system | ROS2 하루에 입문하기, accessed July 26, 2025, https://robertchoi.gitbook.io/ros2/04-build-system
49. Using colcon to build packages - ROS 2 Documentation: Foxy ..., accessed July 27, 2025, https://docs.ros.org/en/foxy/Tutorials/Beginner-Client-Libraries/Colcon-Tutorial.html
50. Creating a package - ROS 2 Documentation: Foxy documentation, accessed July 27, 2025, https://docs.ros.org/en/foxy/Tutorials/Beginner-Client-Libraries/Creating-Your-First-ROS2-Package.html
51. Creating your first ROS 2 package - ROS documentation, accessed July 26, 2025, https://docs.ros.org/en/eloquent/Tutorials/Creating-Your-First-ROS2-Package.html
52. Create a ROS2 Python package - The Robotics Back-End, accessed July 26, 2025, https://roboticsbackend.com/create-a-ros2-python-package/
53. Developing a ROS 2 package - ROS 2 Documentation: Humble ..., accessed July 27, 2025, https://docs.ros.org/en/humble/How-To-Guides/Developing-a-ROS-2-Package.html
54. Writing a simple service and client (C++) - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Cpp-Service-And-Client.html
55. Create a ROS2 Python Package - Kev's Robots, accessed July 26, 2025, https://www.kevsrobots.com/learn/learn_ros/11_create_py_package.html
56. Writing a simple publisher and subscriber (C++) - ROS 2 Documentation, accessed July 27, 2025, https://docs.ros.org/en/foxy/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Cpp-Publisher-And-Subscriber.html
57. Writing a simple publisher and subscriber (C++) - ROS 2 ..., accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Cpp-Publisher-And-Subscriber.html
58. Writing a simple publisher and subscriber (Python) - ROS 2 ..., accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Py-Publisher-And-Subscriber.html
59. Robotics crash course (Part 8 - Writing a ROS 2 publisher and subscriber in C++), accessed July 27, 2025, https://jeremypedersen.com/posts/2024-08-15-pt8-ros2-cpp-pubsub/
60. Writing a simple publisher and subscriber (Python) - ROS 2 Documentation, accessed July 27, 2025, https://docs.ros.org/en/foxy/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Py-Publisher-And-Subscriber.html
61. Getting Started with ROS2 - Tutorial 1: Writing a simple Publisher and Subscriber in python | by Rupesh Kumar Yadav Mediboyina | Medium, accessed July 27, 2025, https://medium.com/@rupesh32003/getting-started-with-ros2-tutorial-1-writing-a-simple-publisher-and-subscriber-in-python-7a7133e0e3dd
62. How to Write C++ Subscriber and Publisher Nodes in ROS2 Humble from Scratch, accessed July 27, 2025, https://www.youtube.com/watch?v=SwBb-pIMU60
63. Create a Basic Publisher and Subscriber (Python) | ROS2 Foxy - Automatic Addison, accessed July 27, 2025, https://automaticaddison.com/create-a-basic-publisher-and-subscriber-python-ros2-foxy/
64. Create and Run Subscriber and Publisher Nodes in Python and ROS2 Humble - YouTube, accessed July 27, 2025, https://www.youtube.com/watch?v=u3uGcT8yuG4
65. ROS 2 Commands Cheat Sheet - The Robotics Space, accessed July 27, 2025, https://www.theroboticsspace.com/assets/article3/ros2_humble_cheat_sheet2.pdf
66. ros2cli repository - ROS Index, accessed July 27, 2025, https://index.ros.org/r/ros2cli/
67. ROS2 간단한 명령어 모음 - snowdeer's Code Holic, accessed July 27, 2025, https://snowdeer.github.io/ros2/2017/12/20/simple-command-of-ros2/
68. ros2 run and ros2 node - Start and Introspect your ROS2 Nodes - The Robotics Back-End, accessed July 27, 2025, https://roboticsbackend.com/ros2-run-and-ros2-node-start-debug-ros2-nodes/
69. Exploring ROS2 Node Interactions: A Bash Script Approach to Trace Topic, Service, and Action Connections" | by Arshad Mehmood | Medium, accessed July 27, 2025, https://medium.com/@arshad.mehmood/exploring-ros2-node-interactions-a-bash-script-approach-to-trace-topic-service-and-action-f380965828e
70. ROS Cheat Sheet | Clearpath Robotics Documentation, accessed July 27, 2025, https://docs.clearpathrobotics.com/docs/ros/tutorials/cheat_sheet
71. Understanding topics - ROS 2 Documentation: Humble ..., accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Topics/Understanding-ROS2-Topics.html
72. ROS2 topic 2 [ROS2 토픽 2] - Intuitive-Robotics - 티스토리, accessed July 27, 2025, https://intuitive-robotics.tistory.com/180
73. Understanding ROS 2 services - ROS documentation, accessed July 27, 2025, https://docs.ros.org/en/eloquent/Tutorials/Services/Understanding-ROS2-Services.html
74. Inspecting topics (ros2 topic) - (Murilo's) ROS2 Tutorial - Read the Docs, accessed July 27, 2025, https://ros2-tutorial.readthedocs.io/en/latest/inspecting_topics.html
75. ros2 topic Command Line Tool - Debug ROS2 Topics From the Terminal, accessed July 27, 2025, https://roboticsbackend.com/ros2-topic-cmd-line-tool-debug-ros2-topics-from-the-terminal/
76. Call Services in ROS 2 - Tutorials - Ubuntu Discourse, accessed July 27, 2025, https://discourse.ubuntu.com/t/call-services-in-ros-2/15261
77. 02장 ROS2 사용방법(Topic, Service, Action 등), accessed July 27, 2025, https://robertchoi.gitbook.io/ros2/02-ros2
78. Using the ros2 param command-line tool - ROS documentation, accessed July 27, 2025, https://docs.ros.org/en/foxy/How-To-Guides/Using-ros2-param.html
79. Using the ros2 param command-line tool - ROS 2 Documentation: Jazzy documentation, accessed July 27, 2025, https://docs.ros.org/en/jazzy/How-To-Guides/Using-ros2-param.html
80. rclpy Params Tutorial - Get and Set ROS2 Params with Python - The Robotics Back-End, accessed July 27, 2025, https://roboticsbackend.com/rclpy-params-tutorial-get-set-ros2-params-with-python/
81. Rviz2 / User Manual - GitHub Pages, accessed July 27, 2025, https://turtlebot.github.io/turtlebot4-user-manual/software/rviz.html
82. [ros2] rviz - velog, accessed July 27, 2025, https://velog.io/@jk01019/ros2-rviz
83. Integrating ROS 2 with Gazebo - Tutorial, accessed July 27, 2025, https://discourse.ros.org/t/integrating-ros-2-with-gazebo-tutorial/23118
84. Use ROS 2 to interact with Gazebo, accessed July 27, 2025, https://gazebosim.org/docs/latest/ros2_integration/
85. docs/garden/ros2_integration.md at master / gazebosim/docs - GitHub, accessed July 27, 2025, https://github.com/gazebosim/docs/blob/master/garden/ros2_integration.md
86. Use ROS 2 to interact with Gazebo - Gazebo ionic documentation, accessed July 27, 2025, https://gazebosim.org/docs/latest/ros2_integration
87. ROS 2 Integration - Gazebo fortress documentation, accessed July 27, 2025, https://gazebosim.org/docs/fortress/ros2_integration/
88. RMW implementations - ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/Installation/RMW-Implementations.html
89. DDS implementations - ROS 2 Documentation: Iron documentation, accessed July 26, 2025, https://docs.ros.org/en/iron/Installation/DDS-Implementations.html
90. Manage Quality of Service Policies in ROS 2 - MATLAB & Simulink - MathWorks, accessed July 26, 2025, https://www.mathworks.com/help/ros/ug/manage-quality-of-service-policies-in-ros2.html
91. Quality of Service settings - ROS 2 Documentation: Rolling documentation, accessed July 26, 2025, https://docs.ros.org/en/rolling/Concepts/Intermediate/About-Quality-of-Service-Settings.html
92. ROS 2 Quality of Service policies - ROS2 Design, accessed July 26, 2025, https://design.ros2.org/articles/qos.html
93. ROS QoS - Deadline, Liveliness, and Lifespan - ROS2 Design, accessed July 26, 2025, https://design.ros2.org/articles/qos_deadline_liveliness_lifespan.html
94. ros2_cookbook/pages/qos.md at main - GitHub, accessed July 26, 2025, https://github.com/mikeferguson/ros2_cookbook/blob/main/pages/qos.md
95. Understanding real-time programming - ROS 2 Documentation: Foxy documentation, accessed July 26, 2025, https://docs.ros.org/en/foxy/Tutorials/Demos/Real-Time-Programming.html
96. Real-time ROS 2 - KRS 1.0 documentation - GitHub Pages, accessed July 26, 2025, https://xilinx.github.io/KRS/sphinx/build/html/docs/features/realtime_ros2.html

