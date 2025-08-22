---
layout: page
title: Nova Carter AMR 개발 플랫폼
permalink: /robotics/nvidia/Nova Carter AMR 개발 플랫폼
---



자율 이동 로봇(AMR) 기술은 물류, 제조, 헬스케어 등 다양한 산업 분야로 빠르게 확산되고 있다.1 특히 창고나 공장과 같이 비정형적이고 동적인 환경에서 인간과 안전하게 협업할 수 있는 AMR에 대한 수요는 폭발적으로 증가하고 있다.2 그러나 이러한 고도화된 AMR을 개발하는 과정은 상당한 기술적 장벽에 직면해 있다. 상용 등급의 모바일 로봇을 개발하는 것은 매우 시간 소모적인 프로세스이며, 개발의 초기 단계부터 난관이 시작된다.3

전통적인 AMR 개발 방식은 파편화된 접근법에 의존한다. 개발팀은 먼저 로봇의 기동성을 책임질 섀시, 방대한 센서 데이터를 처리할 컴퓨팅 모듈, 그리고 주변 환경을 인식하기 위한 다양한 센서(카메라, LiDAR 등)를 개별적으로 소싱해야 한다. 이 과정 이후, 개발의 핵심이라 할 수 있는 고부가가치의 자율 주행 알고리즘이나 애플리케이션별 인공지능(AI)을 개발하는 대신, 대부분의 시간과 자원을 이질적인 하드웨어 구성 요소들을 통합하고, 각 센서에 맞는 디바이스 드라이버를 개발하며, 복잡한 센서 데이터를 보정(calibration)하는 저수준(low-level) 작업에 할애하게 된다. 이 통합 및 보정 과정에만 수년이 소요될 수 있다는 사실은 AMR 기술의 혁신 속도를 저해하는 가장 큰 요인 중 하나로 지목된다.3 이러한 기술적 허들은 자본과 인력이 부족한 스타트업이나 연구 기관에게는 진입 장벽으로 작용하며, 대기업에게는 신제품 출시 지연과 개발 비용 증가의 원인이 된다.


이러한 AMR 개발의 근본적인 문제를 해결하기 위해, 인공지능 컴퓨팅 분야의 선두주자인 NVIDIA와 로보틱 모빌리티 플랫폼(RMP) 분야에서 십수 년간의 경험과 깊이 있는 공급망 노하우를 축적한 Segway Robotics가 전략적 협력 관계를 구축했다.2 이들의 협력은 단순한 부품 공급 계약을 넘어, 각자의 핵심 역량을 결합한 심도 있는 공동 개발의 결과물로 나타났으며, 그 결정체가 바로 Nova Carter AMR 개발 플랫폼이다.2 이들의 파트너십은 'Carter 1.0' 개발부터 이어져 온 것으로, Nova Carter는 오랜 협력의 진화된 형태라 할 수 있다.2

이 협력 관계는 본질적으로 상호보완적이다. Segway Robotics는 자사의 주력 제품인 RMP Lite 220 섀시를 통해 강력하고 신뢰성 높은 기동성을 제공하며, 로봇의 '몸체'와 '골격'을 책임진다.2 반면, NVIDIA는 Nova Orin™ 참조 아키텍처의 핵심인 Jetson AGX Orin 컴퓨팅 모듈과 포괄적인 센서 스위트, 그리고 이를 구동하는 알고리즘과 소프트웨어 시스템 전반을 제공하며 로봇의 '두뇌'와 '신경계' 역할을 담당한다.2

이러한 전문성의 수직적 통합은 시장에 대한 직접적이고 전략적인 대응이다. 파편화되고, 느리며, 비용이 많이 드는 전통적인 AMR R&D 방식의 문제를 정면으로 겨냥한다. NVIDIA의 AI 및 소프트웨어 전문성과 Segway의 하드웨어 및 제조 전문성을 결합함으로써, 개발자들이 겪어야 했던 고통스러운 통합 과정을 원천적으로 제거한 새로운 형태의 제품, 즉 'R&D 가속기'를 탄생시켰다. 이는 다른 기업들의 시장 진입 장벽을 낮추고, 결과적으로 전체 AMR 시장의 성장을 촉진하며 NVIDIA의 Isaac 생태계 채택을 유도하는 촉매제 역할을 한다.


Nova Carter의 핵심 가치 제안은 '개발 가속화'라는 한마디로 요약할 수 있다.2 이 플랫폼은 사전 구성된 소프트웨어를 탑재하여 박스를 개봉하는 즉시 사용할 수 있는 완전한 기능의 로봇 개발 플랫폼이다.2 수동 지게차나 유도 차량(AGV)에서 완전한 자율성을 갖춘 차세대 AMR로의 전환을 획기적으로 단축시키는 것을 목표로 한다.2

Nova Carter는 개발자들이 저수준의 하드웨어 통합 및 보정 작업에서 해방되어, 로봇의 지능을 결정하는 고수준의 AI 애플리케이션 개발에 집중할 수 있도록 설계되었다.3 또한, NVIDIA의 Isaac AMR 및 Isaac ROS 소프트웨어 스택을 위한 공식 '참조 플랫폼(reference platform)'으로서의 역할을 수행한다.10 이는 단순한 기술적 명칭을 넘어선 핵심적인 사업 전략을 내포한다. 참조 플랫폼은 특정 소프트웨어 생태계를 위한 표준화되고 최적화된 하드웨어 타겟을 제공한다. Nova Carter가 Isaac 생태계의 참조 하드웨어 역할을 함으로써, NVIDIA는 개발자들에게 원활한 '즉시 사용 가능한' 경험을 제공하고, 자연스럽게 Isaac 생태계 내에서 애플리케이션을 구축하도록 유도한다. 개발자들이 이 생태계에 시간과 노력을 투자하게 되면, 최종 상용 제품 역시 NVIDIA 하드웨어 기반으로 배포할 가능성이 높아진다. 결과적으로, Nova Carter는 전체 NVIDIA Isaac 소프트웨어 스택의 산업 표준화를 주도하고 채택을 가속화하는 물리적 관문 역할을 수행하게 된다.


Nova Carter의 하드웨어는 고성능 컴퓨팅, 견고한 기동성, 그리고 포괄적인 센서 데이터를 실시간으로 융합하기 위한 정밀한 설계가 결합된 결과물이다. 이는 단순히 고사양 부품의 집합이 아닌, 지능형 자율성을 물리적으로 구현하기 위한 유기적인 시스템이다.


플랫폼의 심장부에는 NVIDIA Jetson AGX Orin 64GB 모듈이 탑재되어 있다.2 이 모듈은 최대 275 TOPS(초당 테라 연산)의 AI 연산 성능을 제공하여, 여러 센서로부터 들어오는 방대한 데이터를 실시간으로 처리하고 복잡한 AI 모델을 구동하기에 충분한 성능을 보장한다.2 데이터 저장 및 전송을 위해 최대 4 GB/sec의 데이터 캡처 속도를 지원하는 2TB 용량의 Samsung 980 PRO M.2 SSD와 고성능 데이터 업로드를 위한 10GbE PCIe 카드가 장착되어 있다.2

이 강력한 컴퓨팅 성능을 물리적 움직임으로 변환하는 것은 Segway의 RMP Lite 220 섀시다.2 이 차동 구동 방식의 섀시는 뛰어난 충격 흡수 성능과 어느 정도의 비포장도로 주행 능력(off-road capabilities)을 갖추고 있어, 매끄러운 바닥의 창고뿐만 아니라 다양한 산업 현장에서의 안정적인 기동성을 보장한다.2 Jetson AGX Orin의 연산 능력과 RMP Lite 220의 견고한 기동성의 결합은 Nova Carter가 복잡하고 동적인 환경에서도 신뢰성 있게 임무를 수행할 수 있는 기반이 된다.


Nova Carter의 가장 큰 특징 중 하나는 로봇 주변 360도 전방위를 끊김 없이 인식할 수 있는 포괄적이고 다양한 센서 스위트다.2 이 센서들은 초당 800메가픽셀 이상의 방대한 데이터를 생성하며, 이는 로봇이 주변 환경에 대한 매우 상세하고 풍부한 정보를 얻을 수 있음을 의미한다.3

- **시각 센서 (Visual Sensors):**
  - **Leopard Imaging Hawk 스테레오 카메라:** 4개의 2.3메가픽셀 컬러 글로벌 셔터(OnSemi AR0234) 스테레오 카메라가 장착되어 있다.12 글로벌 셔터 방식은 빠른 움직임에도 이미지 왜곡이 없어 정확한 깊이 인식, 시각 주행 거리 측정(Visual Odometry), 그리고 vSLAM(Visual Simultaneous Localization and Mapping)에 필수적이다. 각 Hawk 카메라는 120도의 넓은 화각을 가지며, 6축 관성 측정 장치(IMU)가 내장되어 있어 영상 프레임에 정밀한 타임스탬프를 기록하는 데 사용된다.15
  - **Leopard Imaging Owl 어안 카메라:** 4개의 2.3메가픽셀 컬러 글로벌 셔터(OnSemi AR0234) 어안 카메라가 로봇 주변의 완전한 서라운드 뷰를 제공한다.12 이는 원격 조작 시 운영자에게 넓은 시야를 제공하고, 전반적인 상황 인지 능력을 향상시키는 데 활용된다.
- **LiDAR 시스템:**
  - **Hesai XT-32 3D LiDAR:** 1개의 360도 3D LiDAR가 장착되어 상세한 3차원 환경 지도를 생성하고 원거리 객체를 탐지하는 데 사용된다.12 XT-32 LiDAR는 센티미터 수준의 정밀도와 360°(수평) x 40.3°(수직)의 넓은 시야각을 제공하여, 창고의 선반이나 움직이는 작업자와 같은 복잡한 구조물을 정확하게 인식한다.17
  - **Slamtec RPLidar S2E 2D LiDAR:** 로봇의 전면과 후면에 각각 1개씩, 총 2개의 2D LiDAR가 장착되어 있다.12 이는 주로 평면상의 장애물을 신속하게 감지하고, 2D SLAM 및 내비게이션 작업을 보조하는 데 사용된다.
- **기타 센서:** 섀시에는 Bosch사의 IMU, 지자기 센서(Magnetometer), 기압계(Barometer)가 장착되어 로봇의 자세, 방향, 고도와 같은 추가적인 상태 추정 데이터를 제공한다.13

이처럼 다양한 센서들의 조합은 특정 목적을 위해 신중하게 선택되고 배치되었다. 예를 들어, 스테레오 카메라의 시야각은 서로 겹치도록 설계되어 사각지대 없는 깊이 정보를 제공하며, 2D LiDAR와 3D LiDAR는 각각 신속한 평면 장애물 감지와 정밀한 3차원 구조 파악이라는 상호 보완적인 역할을 수행한다. 이는 Nova Carter의 하드웨어가 단순히 데이터를 수집하는 장치들의 집합이 아니라, 실시간 다중 모드 센서 융합이라는 어려운 문제를 해결하기 위해 처음부터 통합된 '인식 장비(perception instrument)'로 설계되었음을 보여준다.


Nova Carter 하드웨어 아키텍처의 핵심 기술 중 하나는 모든 센서 간의 고정밀 시간 동기화(time synchronization)다.13 이는 Jetson Orin 모듈 내부의 하드웨어 타임스탬핑 기능과 이더넷을 통한 정밀 시간 프로토콜(PTP, Precision Time Protocol)을 통해 구현된다. 그 결과, 각 센서의 데이터 획득 시간 간의 오차(jitter)가 10 마이크로초($10 \mu s$) 미만으로 유지된다.13

특히, 8개의 모든 카메라는 단일 트리거 신호에 의해 100 마이크로초($100 \mu s$) 이내에 동시에 촬영이 이루어진다.13 NVIDIA는 이 기술을 통해 고해상도 인식 능력이 12배 향상되었다고 주장한다.3

이러한 수준의 정밀한 동기화는 결코 과장된 사양이 아니다. 만약 동기화가 제대로 이루어지지 않는다면, 최대 3.3 m/s로 움직이는 로봇이 13 각기 다른 시간에 획득한 센서 데이터를 융합할 때 심각한 불일치가 발생한다. 이는 인식된 세계에 '번짐(smearing)'이나 '유령(ghosting)' 현상을 일으켜, vSLAM이나 3D 재구성과 같은 핵심적인 자율 주행 알고리즘의 신뢰도를 급격히 떨어뜨린다. 따라서 고정밀 시간 동기화는 Nova Carter가 고속으로 움직이면서도 주변 환경을 일관되고 정확하게 인식하기 위한 필수 불가결한 기술이다.


Nova Carter의 하드웨어는 연구 개발 환경에서의 유연성과 실제 산업 현장에서의 신뢰성을 동시에 만족시키도록 설계되었다. 이는 개발 친화적인 특징과 대규모 배포에 적합한 견고한 사양의 조화를 통해 드러난다. 개발 단계에서는 Jetson 모듈에 직접 연결된 USB-C, DisplayPort, 이더넷과 같은 I/O 패널을 통해 손쉽게 개발 및 시스템 재설치(re-flashing)가 가능하다.18 반면, 상용 배포를 고려할 때는 견고한 섀시, 보증 및 기술 지원 서비스, IPX7 등급의 방수 배터리, 그리고 상용 로봇 플릿으로 확장 가능한 솔루션이라는 점이 부각된다.2

이러한 이중적 설계 철학은 매우 중요한 전략적 가치를 지닌다. 기업은 초기 알고리즘 개발부터 최종 제품의 대규모 배포에 이르기까지 전 과정에 걸쳐 동일한 플랫폼을 사용할 수 있다. 이는 로보틱스 프로젝트에서 흔히 발생하는 '시뮬레이션-현실(sim-to-real)' 및 '프로토타입-양산(prototype-to-production)' 간의 격차를 해소하여, 개발 수명 주기 전반에 걸쳐 일관성과 효율성을 보장한다. 플랫폼의 주요 물리적 및 운용 사양은 아래 표와 같다.

| 파라미터 (Parameter)               | 사양 (Specification) | 출처 (Source) |
| ---------------------------------- | -------------------- | ------------- |
| 모델 (Model)                       | Nova Carter P1       | 2             |
| 크기 (Vehicle Size)                | 722 * 500 * 556 mm   | 2             |
| 순 중량 (Net Weight)               | 51.2 kg              | 2             |
| 최대 적재량 (Maximum Load)         | 50 kg                | 11            |
| 최고 속도 (Max Speed)              | 3.3 m/s              | 13            |
| 장애물 극복 높이 (Overrun Height)  | 25 mm                | 2             |
| 배터리 용량 (Battery Capacity)     | 1033 Wh              | 2             |
| 운용 시간 (Operation Hours)        | $\ge 8$ h            | 2             |
| 충전 시간 (Charging Time)          | 5 h (Sleep mode)     | 2             |
| 컴퓨팅 플랫폼 (Computing)          | Jetson AGX Orin 64GB | 2             |
| 저장 장치 (Storage)                | 2TB M.2 SSD          | 2             |
| 작동 온도 (Working Temp.)          | 0 ~ 35 °C            | 2             |
| 배터리 IP 등급 (Battery IP Rating) | IPX7                 | 2             |


Nova Carter의 진정한 가치는 강력한 하드웨어와 긴밀하게 통합된 NVIDIA의 소프트웨어 스택 및 AI 생태계에서 발현된다. 이 플랫폼은 단순히 하드웨어를 제공하는 것을 넘어, 개발자가 즉시 지능형 자율성을 구현할 수 있도록 완벽하게 준비된 소프트웨어 환경을 제공한다.


Nova Carter는 NVIDIA Isaac 플랫폼과 완벽하게 통합되도록 설계되었다.2 이는 크게 두 가지 핵심 요소로 구성된다.

- **Isaac ROS:** Isaac ROS는 GPU 가속을 통해 고성능을 발휘하는 ROS 2 패키지들의 집합이다.19 Nova Carter는 인식, 위치 추정, 내비게이션 등 핵심 자율 주행 기능에 이 패키지들을 적극적으로 활용한다. 특히, 노드(node) 간의 데이터 전송 시 메모리 복사를 제거하여 오버헤드를 최소화하는 NITROS(NVIDIA Isaac Transport for ROS) 기술을 통해, 초당 800메가픽셀이 넘는 방대한 센서 데이터를 효율적으로 처리한다.19
- **Isaac AMR:** Isaac AMR은 개별 ROS 패키지를 넘어선 더 높은 수준의 자율성 플랫폼이다.2 여기에는 클라우드 기반의 지도 생성 서비스, 여러 로봇의 임무를 조율하고 최적의 경로를 계획하는 미션 컨트롤(Mission Control) 기능 등이 포함된다. 경로 계획에는 NVIDIA의 경로 최적화 엔진인 cuOpt™가 사용되어 복잡한 물류 환경에서도 효율적인 로봇 운영을 가능하게 한다.2

Nova Carter는 이러한 Isaac AMR 및 Isaac ROS 애플리케이션을 개발하고 테스트하기 위한 공식 참조 로봇으로서, 개발자들에게 NVIDIA가 제공하는 최첨단 로보틱스 AI 기술을 가장 빠르고 안정적으로 접할 수 있는 통로를 제공한다.22


Nova Carter는 복잡한 설정 없이 즉시 사용할 수 있는(out-of-the-box) 다양한 첨단 자율 주행 기능을 지원한다.2

- **지도 작성 및 위치 추정 (Mapping and Localization):**
  - **3D 지도 작성 및 재구성:** Isaac ROS Nvblox 패키지를 활용하여 주변 환경의 3차원 장면을 실시간으로 재구성한다.23 이는 로봇이 장애물을 입체적으로 인식하고 회피하는 데 필수적이다.
  - **시각적 SLAM (vSLAM):** Isaac ROS cuVSLAM 패키지를 통해 스테레오 카메라와 IMU 데이터를 융합하는 vSLAM을 수행한다.24 이 기술은 시각적 특징이 부족하거나 반복적인 패턴이 많은 까다로운 환경에서도 강인한 위치 추정 성능을 보인다. Nova Carter의 공식 소프트웨어 저장소에는 Isaac ROS Visual SLAM을 위한 패키지가 포함되어 있다.23
  - **전역 위치 추정 (Global Localization):** 사전에 구축된 지도 내에서 로봇이 자신의 초기 위치를 스스로 파악하는 전역 위치 추정 기능을 지원한다.3
- **내비게이션 (Navigation):**
  - **Nav2 통합:** Nova Carter는 ROS 2의 표준 내비게이션 스택인 Nav2와 함께 사용되도록 Open Navigation에 의해 공식적으로 검증되었다.3 기본 제공되는 ROS 2 패키지에는 Nav2를 활용한 자율 주행 애플리케이션이 포함되어 있어, 개발자들은 복잡한 경로 계획, 장애물 회피, 목표 지점 도달 등의 기능을 쉽게 구현할 수 있다.13
- **인식 (Perception):**
  - **Isaac Perceptor:** Nova Carter는 Isaac Perceptor 스택을 구동하기 위한 핵심 플랫폼이다. Isaac Perceptor는 3D 서라운드 비전 기술을 통해 장애물을 감지하고 점유 격자 지도(occupancy map)를 생성하는 데 특화된 워크플로우다.22
  - **AI 기반 객체 탐지 및 분할:** Jetson AGX Orin의 강력한 AI 연산 성능을 바탕으로, 이미지 내에서 특정 객체(사람, 지게차 등)를 탐지하고 그 영역을 정밀하게 분할하는 기능을 지원한다.3

이러한 기능들은 Nova Carter가 단순한 이동 플랫폼이 아니라, 고도의 지능을 갖춘 자율 시스템을 구축하기 위한 강력한 기반임을 보여준다.


Nova Carter는 개발자 친화적인 환경을 제공하여 신속한 프로토타이핑과 개발을 지원한다. 시스템은 ROS 2를 기반으로 하며, 사전 구성된 운영체제(OS)와 소프트웨어가 함께 제공된다.2

- **설정 및 운영:** 개발자는 주로 SSH(Secure Shell)를 통해 로봇에 원격으로 접속하여 작업을 수행한다.18 애플리케이션은 일반적으로 Docker 컨테이너 환경에서 실행되며, 플랫폼의 고속 SSD 성능을 최대한 활용하기 위한 Docker 설정 가이드가 제공된다.18
- **제어 및 시각화:** 로봇과 함께 제공되는 PS5 조이스틱 컨트롤러를 사용하여 직관적인 원격 조작(teleoperation)이 가능하다.18 LiDAR의 포인트 클라우드, 카메라 이미지 등 방대한 센서 데이터와 로봇의 상태는 RViz나 Foxglove와 같은 표준 ROS 시각화 도구를 통해 실시간으로 확인할 수 있다.7
- **안전 기능:** 로봇 후면에는 눈에 잘 띄는 붉은색 비상 정지(ESTOP) 버튼이 있어, 위급 상황 시 즉시 모터로 가는 전원을 차단할 수 있다.18 또한, 로봇 전면의 상태 표시 LED는 부팅(노란색), 정상 작동(녹색), 오류 또는 비상 정지(붉은색) 등 시스템의 현재 상태를 시각적으로 알려준다.18

Nova Carter는 '개발 플랫폼'과 '상용 로봇'이라는 두 가지 정체성을 동시에 가지고 있다. 이는 개발자에게는 ROS 2, Docker, 오픈소스 패키지를 제공하는 유연한 개발 키트이며 13, 상용 배포를 고려하는 기업에게는 보증과 기술 지원이 포함된 확장 가능한 솔루션이다.2 이 이중적 포지셔닝 전략은 NVIDIA와 Segway가 단일 하드웨어 플랫폼으로 두 개의 다른 시장을 공략할 수 있게 하며, 더 중요하게는 고객에게 원활한 확장 경로를 제공한다. 즉, 한 대의 로봇으로 연구개발을 시작한 기업이 근본적인 하드웨어와 소프트웨어 스택을 변경하지 않고도 수십, 수백 대의 플릿으로 확장할 수 있게 되는 것이다. 이는 프로토타입에서 양산으로 넘어가는 과정의 마찰과 위험을 크게 줄여준다.

또한, 이 플랫폼은 ROS 2와 Nav2 같은 오픈소스 표준을 적극적으로 활용하지만 7, 광고하는 고성능을 달성하기 위해서는 결국 NVIDIA의 독점적인 GPU 가속 Isaac ROS 패키지(cuVSLAM, Nvblox 등)를 사용하도록 유도된다. 이는 NVIDIA가 개방형 표준인 ROS 2를 일종의 'API 계층'으로 사용하여 방대한 ROS 개발자 커뮤니티를 유인한 뒤, 이들을 자사의 하드웨어 가속 기반 독점 컴퓨팅 생태계로 끌어들이는 정교한 전략이다.


Nova Carter 개발 패러다임의 핵심은 물리적 로봇과 완벽하게 동기화된 가상 로봇, 즉 디지털 트윈(Digital Twin)을 활용하는 '시뮬레이션 우선(simulation-first)' 접근법에 있다. 이는 개발의 효율성과 안전성을 극대화하는 현대 로보틱스 개발의 표준 워크플로우를 제시한다.


Nova Carter의 디지털 트윈은 NVIDIA의 로보틱스 시뮬레이션 애플리케이션인 Isaac Sim 내에서 제공된다.3 이 디지털 트윈은 단순히 로봇의 3D 모델을 보여주는 것을 넘어선다. 로봇의 섀시는 물론, 모든 센서(카메라, LiDAR, IMU)가 실제와 동일한 위치에 동일한 특성을 가지고 구현되어 있다.12

이 시뮬레이션은 물리적으로 매우 정확하다. NVIDIA의 실시간 레이 트레이싱 기술인 RTX를 기반으로 한 센서 시뮬레이션은 실제 센서가 수집하는 것과 거의 동일한, 사실적인 데이터를 생성한다.7 예를 들어, 시뮬레이션 속 LiDAR는 물체의 재질에 따라 레이저가 반사되는 특성까지 모사하며, 카메라는 조명과 그림자의 변화를 사실적으로 렌더링한다. 흥미로운 점은 이 시뮬레이션 모델이 실제 하드웨어가 완성되기도 전에 먼저 개발되었다는 사실이며, 이는 개발 초기부터 시뮬레이션을 핵심 도구로 삼는 '시뮬레이션 우선' 철학을 명확히 보여준다.27

이러한 고충실도 시뮬레이션은 AI 모델 훈련을 위한 방대한 양의 합성 데이터(synthetic data)를 생성하는 데 매우 유용하다. 실제 세계에서는 수집하기 어렵거나 위험한 다양한 시나리오(예: 조명이 어두운 환경, 갑자기 나타나는 장애물)를 가상 환경에서 무한히 생성하여 AI 모델의 강인성을 높일 수 있다.


Nova Carter의 디지털 트윈은 소프트웨어 인 더 루프(SIL, Software-in-the-Loop) 및 하드웨어 인 더 루프(HIL, Hardware-in-the-Loop) 테스트를 가능하게 한다.3 개발자는 실제 로봇 없이도 자신의 전체 소프트웨어 스택을 시뮬레이션 환경에서 먼저 검증할 수 있다.27

Isaac Sim 내의 가상 로봇은 ROS 2 브리지를 통해 ROS 2 생태계에 연결된다.12 이는 개발자가 시뮬레이션 환경과 실제 환경에서 동일한 제어 및 내비게이션 코드를 사용할 수 있음을 의미한다. Isaac Sim은 작은 창고와 같은 샘플 환경을 기본으로 제공하여, 개발자가 즉시 내비게이션 및 제어 알고리즘을 테스트해 볼 수 있도록 지원한다.12

이러한 Sim-to-Real 워크플로우는 개발 시간과 비용을 획기적으로 줄여준다. 특히 개발 초기 단계에서 값비싼 실제 로봇을 계속해서 구동할 필요가 없어진다.27 또한, 로봇이 고장 나거나 위험한 상황에 처할 수 있는 다양한 실패 시나리오와 엣지 케이스(edge case)를 가상 환경에서 안전하고 철저하게 테스트할 수 있어 안전성도 크게 향상된다. NVIDIA와 Segway가 직접 제공하는 고품질 모델은 'Sim-to-Real 격차'를 최소화하여, 시뮬레이션에서 성공적으로 작동하는 코드가 실제 로봇에서도 최소한의 수정만으로 잘 작동할 것을 보장한다.27

이처럼 공식적이고 고충실도의 디지털 트윈이 제품의 핵심 요소로 제공된다는 점은 '로봇'이라는 제품의 정의를 근본적으로 바꾸고 있다. 고객은 더 이상 하드웨어만을 구매하는 것이 아니다. 가상과 현실의 인스턴스가 동전의 양면처럼 완벽하게 연동되는 하드웨어-소프트웨어 통합 패키지를 구매하는 것이다. 이는 로보틱스 플랫폼이 제품화되는 방식에 있어 중요한 진화를 의미한다.

나아가, 이 디지털 트윈은 초기 개발 단계를 넘어 전체 로봇 플릿(fleet)을 확장하고 관리하는 데에도 핵심적인 역할을 한다. Isaac AMR 플랫폼은 클라우드 기반의 미션 컨트롤 기능을 제공하는데 2, 실제 창고에서 운영 중인 수십 대의 로봇 플릿에 새로운 소프트웨어 업데이트나 미션 계획을 배포하기 전에, Isaac Sim 내에 구축된 해당 시설의 디지털 트윈에서 가상 플릿을 대상으로 먼저 검증을 수행할 수 있다.21 이를 통해 다중 로봇 간의 협업, 교통 관리, 새로운 소프트웨어의 안정성 등을 실제 운영에 전혀 지장을 주지 않으면서 대규모로 검증할 수 있다. 따라서 시뮬레이션 워크플로우는 연구개발 단계를 넘어 운영 및 배포 단계까지 확장되어, 대규모 상용 배포 환경에서의 위험 관리와 운영 효율성을 위한 필수 도구가 된다.


Nova Carter는 특정 응용 분야를 목표로 설계되었으며, 그 유연성 덕분에 다양한 영역에서 활용될 수 있는 잠재력을 지니고 있다. 주된 목표 시장은 물류 및 창고 자동화이며, 동시에 로보틱스 연구개발을 위한 표준 플랫폼으로서의 역할도 매우 중요하다.


Nova Carter의 가장 핵심적인 목표 응용 분야는 창고 관리 및 물류 핸들링이다.2 이 플랫폼은 작업자, 지게차, 다른 로봇들이 끊임없이 움직이는 매우 동적이고 비정형적인 창고 환경에서 인간과 함께 안전하게 작동하도록 설계되었다.2

구체적인 활용 사례로, Nova Carter를 사용하여 창고나 공장과 같은 테스트 영역의 정밀한 3D 지도를 수집할 수 있다. 이렇게 생성된 지도는 이후 로봇이 완전 자율 주행 임무를 수행하는 데 기반 데이터로 사용된다.5 예를 들어, 'A 지점에서 팔레트를 집어 B 지점으로 옮기라'는 명령을 수행하기 위해, 로봇은 이 지도를 바탕으로 최적의 경로를 계획하고, 실시간으로 센서 데이터를 처리하여 예기치 않은 장애물(예: 바닥에 떨어진 상자, 갑자기 나타난 작업자)을 회피하며 이동한다. 특히 Hesai XT-32 3D LiDAR는 복잡하게 배치된 선반과 끊임없이 움직이는 객체들을 탐지하는 데 매우 효과적인 성능을 보인다.17

궁극적으로 Nova Carter는 이러한 환경에서 생산성을 크게 향상시키고, 운영 비용을 절감하며, 대규모 로봇 플릿의 배포를 가속화하는 동시에 작업장 안전을 보장하는 '혁신적인 로봇'으로 자리매김하는 것을 목표로 한다.2


Nova Carter는 상용 애플리케이션뿐만 아니라, 로보틱스 개발 및 연구를 위한 참조 AMR(Reference AMR)로서 명확하게 포지셔닝되어 있다.7 이 플랫폼은 연구자들에게 자율주행 자동차 분야의 최첨단 기술을 로보틱스에 맞게 변형하고 적용한 최신 플랫폼을 제공한다.13

연구자들은 NVIDIA가 자사의 Isaac ROS를 개발하고 테스트하는 데 사용하는 것과 동일한 시스템 소프트웨어, 드라이버, 하드웨어, 그리고 성능 튜닝 환경에 접근할 수 있다.13 이는 연구자들이 매번 새로운 로봇 플랫폼을 처음부터 구축하는 데 드는 시간과 노력을 절약하고, 대신 새로운 인식, 계획, 제어 알고리즘을 개발하는 본연의 연구에 집중할 수 있게 해준다.

이 플랫폼은 두 가지 방식으로 접근 가능하다. 물리적인 실험이 필요한 연구자들은 Segway Robotics를 통해 실제 로봇을 구매할 수 있으며, 시뮬레이션 기반의 연구를 선호하는 연구자들은 Isaac Sim 내에서 무료로 제공되는 디지털 트윈을 활용할 수 있다.7 이러한 접근성은 첨단 로보틱스 연구의 진입 장벽을 낮추는 데 크게 기여한다.

이러한 응용 분야를 통해 Nova Carter는 단순히 제품을 판매하는 것을 넘어, 시장 생태계 자체를 형성하는 역할을 수행한다. Hesai XT-32 LiDAR나 Leopard Imaging 카메라와 같은 최첨단 센서들을 Jetson AGX Orin과 함께 패키징함으로써, 이 플랫폼은 더 넓은 개발자 기반에 이러한 고성능 부품들을 소개하고 표준화하는 매개체 역할을 한다. 개별적으로 구매하고 통합하기에는 부담스러웠을 고가의 복잡한 센서들을 사전 통합되고 검증된 패키지로 제공함으로써, 개발자들은 통합의 위험과 복잡성 없이 이들의 성능에 익숙해진다. 그리고 이들이 나중에 자신만의 맞춤형 로봇을 대량 생산하게 될 때, 이미 익숙하고 신뢰하는 바로 그 부품들을 채택할 가능성이 매우 높아진다. 따라서 Nova Carter는 NVIDIA의 하드웨어 파트너사들의 미래 판매를 촉진하는 '시장 파종(market seeding)' 메커니즘으로 기능한다.

결론적으로, Nova Carter 프로젝트의 궁극적인 성공 지표는 로봇 자체의 판매 대수가 아니라, 이를 통해 얼마나 많은 개발자와 기업을 Isaac/Jetson 생태계로 끌어들였는가에 있다. 판매된 모든 Nova Carter는 NVIDIA 로보틱스 생태계에 새로 합류한 팀 또는 회사를 의미한다. 이들이 나중에 어떤 섀시를 사용하든, 그들의 상용 플릿이 NVIDIA의 반도체와 소프트웨어 위에서 구동된다면 NVIDIA의 목표는 달성된 것이다. Nova Carter의 성공은 전체 Isaac 플랫폼의 미래 시장 점유율을 가늠하는 선행 지표인 셈이다.


Nova Carter는 단순한 로봇 제품을 넘어, 차세대 AMR 개발의 방향성과 표준을 제시하는 중요한 플랫폼이다. 이 플랫폼이 로봇 공학 생태계에 미치는 영향은 세 가지 핵심적인 측면에서 종합적으로 평가할 수 있다.


Nova Carter는 로봇 개발 패러다임이 파편화된 부품 기반 조립 방식에서, 긴밀하게 통합되고 공동 설계된 플랫폼으로 전환되고 있음을 명확하게 보여주는 사례다. NVIDIA의 Jetson AGX Orin 컴퓨팅 하드웨어, Isaac 소프트웨어 스택, 그리고 Segway의 RMP Lite 220 섀시 간의 완벽한 연동은 개발 효율성에 대한 새로운 기준을 제시한다. 과거 개발자들이 수많은 시행착오를 겪으며 해결해야 했던 하드웨어와 소프트웨어 간의 비호환성, 데이터 동기화, 드라이버 개발 등의 문제가 플랫폼 수준에서 해결됨으로써, 개발자들은 본질적인 가치 창출에 집중할 수 있게 되었다. 이러한 수직 통합 모델은 향후 로보틱스 플랫폼 개발의 주류가 될 것이며, 시장의 혁신 속도를 더욱 가속화할 것이다.


Nova Carter와 Isaac Sim의 깊은 통합은 고충실도 디지털 트윈이 더 이상 일부 고급 프로젝트에서만 사용되는 틈새 기능이 아니라, 현대 로보틱스 플랫폼의 핵심 구성 요소임을 입증했다. 개발 초기 단계부터 물리적 로봇과 동일하게 작동하는 가상 환경을 제공함으로써, 개발 과정의 위험을 줄이고, 대규모 테스트에 대한 접근성을 민주화하며, 전반적인 개발 비용과 시간을 획기적으로 단축시킨다. '시뮬레이션 우선' 접근법은 이제 복잡한 자율 시스템을 개발하기 위한 필수적인 워크플로우로 자리 잡았으며, 향후 출시될 모든 개발 플랫폼이 갖추어야 할 표준적인 기능이 될 것으로 전망된다.


가장 중요한 영향은 Nova Carter가 첨단 AMR 기술에 대한 접근성을 대중화했다는 점이다. 막대한 기술적, 재정적 진입 장벽을 낮춤으로써, 이 플랫폼은 학술 연구실부터 자금이 부족한 스타트업에 이르기까지 훨씬 더 넓은 범위의 혁신가들이 AMR 분야에서 경쟁할 수 있는 기회를 제공한다. 이는 필연적으로 새로운 애플리케이션의 폭발적인 증가로 이어질 것이며, 자율 로보틱스 분야의 전반적인 발전 속도를 끌어올릴 것이다. 미래에는 Nova Carter와 같은 플랫폼들이 더욱 발전된 AI 파운데이션 모델을 통합하고, 훨씬 정교한 센서 기술을 탑재하며, 인간과 로봇의 상호작용을 새로운 차원으로 끌어올리는 방향으로 진화할 것이다. Nova Carter는 그 미래를 향한 중요한 첫걸음이다.


1. Create, Design, and Deploy Robotics Applications Using New NVIDIA Isaac Foundation Models and Workflows, 8월 15, 2025에 액세스, https://developer.nvidia.com/blog/create-design-and-deploy-robotics-applications-using-new-nvidia-isaac-foundation-models-and-workflows/
2. Nova Carter - Segway Robotics, 8월 15, 2025에 액세스, https://robotics.segway.com/nova-carter/
3. RBR50 Spotlight: NVIDIA gives mobile robots all-around sight with ..., 8월 15, 2025에 액세스, https://www.therobotreport.com/rbr50-spotlight-nvidia-gives-mobile-robots-all-around-sight-with-nova-carter/
4. NVIDIA gives mobile robots all-around sight with Nova Carter - The Robot Report, 8월 15, 2025에 액세스, https://www.therobotreport.com/rbr50-company-2024/nvidia-gives-mobile-robots-all-around-sight-with-nova-carter/
5. Segway Robotics and NVIDIA Launch Nova Carter AMR, a Complete Robotics Development Platform - PR Newswire, 8월 15, 2025에 액세스, https://www.prnewswire.com/news-releases/segway-robotics-and-nvidia-launch-nova-carter-amr-a-complete-robotics-development-platform-301963050.html
6. Nova Carter - isaac_ros_docs documentation, 8월 15, 2025에 액세스, https://nvidia-isaac-ros.github.io/v/release-3.1/robots/nova_carter/index.html
7. Nova Carter AMR for ROS 2 w/ 800 megapixel/sec sensor processing, 8월 15, 2025에 액세스, https://discourse.openrobotics.org/t/nova-carter-amr-for-ros-2-w-800-megapixel-sec-sensor-processing/34215
8. Segway Robotics, NVIDIA Develop Full AMR Development Platform | AEI, 8월 15, 2025에 액세스, https://aei.dempa.net/archives/20412
9. Segway Robotics and NVIDIA launch Nova Carter AMR - AI-Tech Park, 8월 15, 2025에 액세스, https://ai-techpark.com/segway-robotics-and-nvidia-launch-nova-carter-amr/
10. Featured Assets - Isaac Sim Documentation - NVIDIA, 8월 15, 2025에 액세스, https://docs.isaacsim.omniverse.nvidia.com/4.5.0/assets/usd_assets_featured.html
11. Introducing the Segway Nova Carter mobile robot development platform, 8월 15, 2025에 액세스, https://www.automatedwarehouseonline.com/introducing-the-segway-nova-carter-mobile-robot-development-platform/
12. Nova Carter - Isaac Sim Documentation, 8월 15, 2025에 액세스, https://docs.isaacsim.omniverse.nvidia.com/latest/assets/nova_carter_landing_page.html
13. Robots/NovaCarter - ROS Wiki, 8월 15, 2025에 액세스, http://wiki.ros.org/Robots/NovaCarter
14. Nova Carter - robots.ros.org, 8월 15, 2025에 액세스, https://robots.ros.org/nova-carter/
15. Leopard Imaging Hawk and Owl Cameras Powers Nova Carter AMR for ROS 2 w/ 800 megapixel/sec sensor processing, 8월 15, 2025에 액세스, https://leopardimaging.com/leopard-imaging-hawk-and-owl-cameras-powers-nova-carter-amr-for-ros-2-w-800-megapixel-sec-sensor-processing/
16. Leopard Imaging to Showcase NVIDIA Isaac Nova Orin-Based Reference Design Cameras Hawk and Owl from Segway Robotics' Nova Carter at CES 2024 - PR Newswire, 8월 15, 2025에 액세스, https://www.prnewswire.com/news-releases/leopard-imaging-to-showcase-nvidia-isaac-nova-orin-based-reference-design-cameras-hawk-and-owl-from-segway-robotics-nova-carter-at-ces-2024-302028265.html
17. Warehouse Automation Lidar Solutions | Hesai Technology, 8월 15, 2025에 액세스, https://www.hesaitech.com/industry/warehousing-and-logistics/
18. Nova Carter - nova_docs documentation - NVIDIA Isaac ROS, 8월 15, 2025에 액세스, https://nvidia-isaac-ros.github.io/nova/getting_started/platforms/nova_carter.html
19. NVIDIA Isaac ROS - GitHub, 8월 15, 2025에 액세스, https://github.com/NVIDIA-ISAAC-ROS
20. Build High Performance Robotic Applications with NVIDIA Isaac ROS Developer Preview 3, 8월 15, 2025에 액세스, https://developer.nvidia.com/blog/build-high-performance-robotic-applications-with-nvidia-isaac-ros-developer-preview-3/
21. NVIDIA Isaac AMR | End-to-End Autonomy Platform for Next Generation AMRs - YouTube, 8월 15, 2025에 액세스, https://www.youtube.com/watch?v=SMZCio24Zx4
22. Nova Carter | NVIDIA NGC, 8월 15, 2025에 액세스, https://catalog.ngc.nvidia.com/orgs/nvidia/teams/isaac/containers/nova_carter_bringup
23. README.md - NVIDIA-ISAAC-ROS/nova_carter - GitHub, 8월 15, 2025에 액세스, https://github.com/NVIDIA-ISAAC-ROS/nova_carter/blob/main/README.md
24. Isaac Perceptor | NVIDIA Developer, 8월 15, 2025에 액세스, https://developer.nvidia.com/isaac/perceptor
25. Leopard Imaging's Nova Orin Development Kit, 8월 15, 2025에 액세스, https://leopardimaging.com/nvidia-nova-devkit/
26. Nova Carter - Isaac Sim Documentation, 8월 15, 2025에 액세스, https://docs.isaacsim.omniverse.nvidia.com/4.5.0/assets/nova_carter_landing_page.html
27. Introduction to Segway Nova Carter AMR platform - YouTube, 8월 15, 2025에 액세스, https://www.youtube.com/watch?v=KrMsumU4voQ

