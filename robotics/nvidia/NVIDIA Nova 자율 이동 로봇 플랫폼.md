NVIDIA Nova 자율 이동 로봇(AMR) 플랫폼

## I. 서론: 차세대 AMR 개발 패러다임, NVIDIA Isaac AMR 및 Nova 플랫폼

### 0.1 A. 자율 이동 로봇(AMR) 시장의 기술적 변곡점과 도전 과제

자율 이동 로봇(AMR) 시장은 물류 자동화와 스마트 팩토리의 부상에 힘입어 전례 없는 성장기에 진입했다. 시장 분석에 따르면, 전 세계 물류 AMR 시장 규모는 2021년 80억 달러에서 2030년 460억 달러 규모로 약 6배 가까이 성장할 것으로 예측된다.1 이러한 폭발적인 수요는 단순히 정해진 경로를 따라 반복 작업을 수행하는 자동 운반 차량(AGV)의 한계를 넘어, 복잡하고 끊임없이 변화하는 비정형 환경(unstructured environments)에서 인간과 안전하게 협업하며 지능적으로 작업을 수행할 수 있는 고도화된 자율성에 대한 산업계의 절실한 요구를 반영한다.2

그러나 이러한 고도화된 AMR을 개발하고 배포하는 과정은 수많은 기술적 난제에 직면해 있다. AMR 제조사들은 최적의 센서를 선정하고, 이를 물리적으로 통합하며, 각 센서의 데이터를 정밀하게 보정하고, 안정적인 디바이스 드라이버를 개발하는 데에만 수년에 달하는 시간과 막대한 엔지니어링 자원을 투입해야 할 수 있다.4 또한, 각각의 로봇 플랫폼마다 자율 주행 소프트웨어 스택을 처음부터 구축하는 것은 극도의 복잡성을 야기하며, 수십만 제곱미터에 달하는 대규모 물류창고의 3차원 지도를 정밀하게 제작하고 변화하는 환경에 맞춰 지속적으로 갱신하는 것 역시 막대한 비용과 전문 인력을 요구하는 작업이다.6 기존의 창고 관리 시스템(WMS) 및 차량 관제 시스템(FMS)과의 원활한 통합 문제 역시 AMR의 광범위한 확산을 가로막는 주요 장벽으로 작용하고 있다.

### 0.2 B. NVIDIA의 해법: 통합 레퍼런스 플랫폼 'Nova Orin'

NVIDIA는 이러한 AMR 개발의 근본적인 문제들을 해결하기 위한 청사진으로 Isaac AMR 플랫폼과 그 핵심인 'Nova Orin' 레퍼런스 아키텍처를 제시한다.1 이 접근법은 개별 하드웨어 부품, 즉 컴퓨팅 모듈이나 센서를 공급하는 전통적인 방식을 완전히 탈피한다. 대신, 사전 검증을 거쳐 긴밀하게 통합된 하드웨어(컴퓨팅, 센서, 차체), 소프트웨어 스택(OS, 드라이버, 미들웨어, AI 라이브러리), 그리고 시뮬레이션 도구를 포괄하는 완전한 엔드-투-엔드(end-to-end) 솔루션을 제공하는 것을 목표로 한다.2

Nova Orin 플랫폼의 궁극적인 목표는 AMR 제조업체(OEM)와 소프트웨어 개발사(ISV)가 센서 통합이나 저수준(low-level) 소프트웨어 개발과 같은 비차별적인 엔지니어링 문제에서 벗어나, 자사의 고유한 애플리케이션과 서비스 개발이라는 핵심 역량에 집중할 수 있는 환경을 제공하는 것이다. 이를 통해 전체 개발 수명주기를 수년 단위로 단축하고, 수백만 달러에 달하는 개발 비용을 절감함으로써 AMR 기술의 혁신과 시장 도입을 극적으로 가속화하고자 한다.1 이러한 접근 방식은 NVIDIA가 단순히 반도체 공급자를 넘어, AMR 개발의 표준을 제시하는 플랫폼 제공자로서의 역할을 수행하려는 전략적 의도를 명확히 보여준다. PC 시장에서 운영체제(OS)와 하드웨어 레퍼런스 디자인이 생태계를 지배했던 것처럼, NVIDIA는 Isaac과 Nova를 통해 '로보틱스의 운영체제'가 되려는 거대한 비전을 그리고 있다. 이는 개별 AMR 제조사들을 하드웨어 스펙 경쟁에서 소프트웨어 및 애플리케이션 기반의 부가가치 경쟁으로 전환시키며, 전체 산업의 혁신 속도를 높이는 동시에 NVIDIA 플랫폼에 대한 기술적 의존도를 심화시키는 결과를 가져올 것이다.

### 0.3 C. 플랫폼의 핵심 철학: Sim-to-Real, Vision-First, Open Ecosystem

Nova 플랫폼의 혁신성은 세 가지 핵심 철학의 유기적인 결합에서 비롯된다.

첫째, **Sim-to-Real 워크플로우**이다. NVIDIA Omniverse 플랫폼 위에서 구동되는 Isaac Sim은 USD(Universal Scene Description)와 PhysX 물리 엔진을 기반으로 현실 세계와 거의 동일한 수준의 물리적 정확성을 갖춘 디지털 트윈(Digital Twin)을 생성한다.2 개발자는 실제 로봇을 제작하거나 물리적 테스트 환경을 구축하기 전에, 이 가상 환경 속에서 로봇의 자율 주행 알고리즘, 인식 스택, 작업 시나리오를 수천, 수만 번 반복하여 테스트하고 검증할 수 있다.4 이는 전통적인 로봇 개발의 '설계-제작-테스트-수정'이라는 시간과 비용이 많이 드는 물리적 반복 과정을 '가상 검증(virtual commissioning)' 단계로 대체하는 패러다임의 전환이다. 이를 통해 개발 비용과 위험을 획기적으로 절감하고, 제품의 신뢰성과 안전성을 비약적으로 향상시킬 수 있다.

둘째, **Vision-First Perception** 전략이다. 전통적인 AMR이 고가의 3D LiDAR 센서에 크게 의존했던 것과 달리, Nova 플랫폼은 상대적으로 저렴한 다수의 카메라를 통해 360도 3D 서라운드 비전(surround vision)을 구현하는 Isaac Perceptor 스택을 전면에 내세운다.3 이는 강력한 GPU 가속 컴퓨팅 파워를 바탕으로, 풍부한 색상과 텍스처 정보를 담고 있는 시각 데이터를 실시간으로 처리하여 환경을 3차원으로 이해하는 접근법이다. 이 전략은 단순히 비용 효율성을 높이는 것을 넘어, 기존 LiDAR 센서가 감지하기 어려운 얇은 장애물이나 낮은 높이의 팔레트 포크와 같은 물체를 정확하게 인식하는 데 기술적 우위를 제공한다.3

셋째, **Open Ecosystem**의 구축이다. Nova 플랫폼의 소프트웨어 기반은 산업 표준으로 자리 잡은 오픈소스 로봇 운영체제인 ROS 2(Robot Operating System)이다.9 이는 전 세계 수백만 명에 달하는 방대한 ROS 개발자 커뮤니티가 기존의 지식과 자산을 활용하여 NVIDIA의 강력한 GPU 가속 라이브러리와 최첨단 AI 모델을 손쉽게 자신들의 애플리케이션에 통합할 수 있도록 생태계의 문을 활짝 열어둔 것이다.9 이를 통해 NVIDIA는 자사 플랫폼의 빠른 확산을 유도하고, 다양한 파트너사들이 참여하는 풍부하고 역동적인 로보틱스 생태계를 조성하고자 한다.

## 1. II. Nova Orin 레퍼런스 아키텍처: 하드웨어 구성 요소 분석

Nova Orin 아키텍처는 AMR의 '두뇌'와 '눈', 그리고 '몸체'를 구성하는 핵심 하드웨어 요소들을 사전 통합하여 제공하는 청사진이다. 이는 강력한 컴퓨팅 엔진, 즉시 배포 가능한 레퍼런스 로봇, 그리고 360도 인식을 위한 포괄적인 센서 스위트로 구성된다.

### 1.1 A. 컴퓨팅 엔진: NVIDIA Jetson AGX Orin

플랫폼의 심장부에는 NVIDIA Jetson AGX Orin 시스템-온-모듈(SoM)이 자리 잡고 있다.12 이 모듈은 최대 275 TOPS(초당 테라 연산)에 달하는 막대한 AI 연산 성능을 제공하여, 다수의 고해상도 카메라와 LiDAR로부터 들어오는 방대한 센서 데이터 스트림을 실시간으로 처리하고, 복잡한 심층 신경망(DNN) 기반의 인식 및 탐색 알고리즘을 엣지(edge) 단에서 직접 실행하는 핵심적인 역할을 수행한다.6 64GB의 대용량 통합 메모리와 2TB의 고속 NVMe SSD는 대규모 3D 맵 데이터와 센서 로그를 원활하게 처리하기 위한 필수 요소이며, 통합된 10GbE PCIe 네트워크 카드는 클라우드 기반 맵핑 서비스나 중앙 관제 시스템과의 고속 데이터 통신을 보장하여 로봇의 운영 효율성을 극대화한다.12

### 1.2 B. 레퍼런스 로봇 플랫폼: Nova Carter

Nova Carter는 Nova Orin 아키텍처를 기반으로 제작된 완전한 기능의 AMR 레퍼런스 플랫폼이다.14 이는 개발자가 하드웨어 조립이나 통합 과정 없이 즉시 소프트웨어 개발과 애플리케이션 테스트에 착수할 수 있도록 'out-of-the-box' 형태로 제공된다.4 차체는 Segway Robotics의 검증된 RMP Lite 220 차륜 구동 플랫폼을 기반으로 하여, 산업 현장의 다양한 바닥 조건에서도 안정적인 주행 성능과 내구성을 보장한다.4

Nova Carter는 최대 50kg의 페이로드를 탑재할 수 있어, 컨베이어, 로봇 팔, 측정 장비 등 다양한 상부 모듈을 장착하여 특정 임무에 맞게 커스터마이징이 가능하다.12 1033Wh 용량의 대용량 배터리는 한 번의 충전으로 최소 8시간 이상 연속 작동이 가능하며, 현장에서 신속하게 교체할 수 있도록 설계되어 로봇의 가동 중단 시간(downtime)을 최소화한다.12 또한, IPX7 등급의 배터리 팩과 산업 환경에 적합한 내구성을 갖춘 차체는 공장이나 창고에서 발생할 수 있는 먼지나 습기로부터 핵심 부품을 안전하게 보호한다.

#### 1.2.1 핵심 테이블 1: Nova Carter P1 상세 하드웨어 제원표

Nova 플랫폼의 물리적, 전기적, 연산적 성능 한계를 명확히 파악하고 타 AMR과의 정량적 비교를 용이하게 하기 위해, 여러 자료에 분산된 Nova Carter P1의 핵심 제원을 아래와 같이 통합하여 정리하였다.4

| 분류            | 항목             | 제원                                           | 출처 |
| --------------- | ---------------- | ---------------------------------------------- | ---- |
| **컴퓨팅**      | 프로세서         | NVIDIA Jetson AGX Orin™ 64GB Module            | 12   |
|                 | AI 성능          | 275 TOPS                                       | 6    |
|                 | 스토리지         | 2TB NVMe SSD (Samsung 980 PRO)                 | 12   |
|                 | 네트워크         | 10GbE PCIe Card (Intel X540-10G-2T-X8)         | 12   |
| **차체**        | 모델             | Segway RMPLite 220 기반                        | 4    |
|                 | 크기 (L×W×H)     | 722 × 500 × 556 mm                             | 12   |
|                 | 중량             | 49.6 kg                                        | 12   |
|                 | 최대 페이로드    | 50 kg                                          | 12   |
|                 | 최고 속도        | ≥12 km/h (무부하)                              | 12   |
|                 | 장애물 극복 높이 | 25 mm                                          | 12   |
|                 | 작동 온도        | 0 ~ 35 °C                                      | 12   |
| **전력 시스템** | 배터리 용량      | 1033 Wh                                        | 12   |
|                 | 작동 시간        | ≥8 시간                                        | 12   |
|                 | 충전 시간        | 5 시간                                         | 12   |
|                 | 배터리 IP 등급   | IPX7                                           | 12   |
| **센서 스위트** | 스테레오 카메라  | 4 × Leopard Imaging Hawk (2.3 MP, HFOV 121.5°) | 12   |
|                 | 어안 카메라      | 4 × Leopard Imaging Owl (2.3 MP, HFOV 202°)    | 12   |
|                 | 2D LiDAR         | 2 × Slamtec RPLidar S2E (360°, 30m range)      | 12   |
|                 | 3D LiDAR         | 1 × Hesai XT-32 (360°, 120m range)             | 12   |
|                 | IMU              | 2 × Bosch BMI088 (전면 스테레오, 차체)         | 12   |

### 1.3 C. 센서 스위트: 360도 서라운드 인식의 구현

Nova Carter의 센서 구성은 단순히 여러 종류의 센서를 나열하는 것을 넘어, 각 센서의 장점을 극대화하고 단점을 상호 보완하는 정교한 설계 철학을 담고 있다. 이는 특정 센서 하나에 의존할 때 발생할 수 있는 치명적인 실패 지점(single point of failure)을 원천적으로 제거하여, 예측 불가능한 산업 현장에서 로봇의 안전성과 신뢰성을 극대화하려는 의도를 보여준다.

로봇의 전후좌우에는 총 4개의 Leopard Imaging Hawk 스테레오 카메라와 4개의 Owl 어안 카메라가 장착되어, 사각지대 없는 360도 서라운드 비전을 완벽하게 구현한다.4 이 다중 카메라 시스템은 초당 800메가픽셀 이상의 방대한 시각 데이터를 생성하며 4, 이는 Visual SLAM을 통한 위치 추정, DNN 기반의 깊이 추정, 그리고 의미론적 분할(semantic segmentation)을 통한 장애물 감지 등 시각 기반 인식 알고리즘의 핵심적인 입력 데이터로 활용된다.

여기에 2개의 2D LiDAR와 1개의 3D LiDAR가 추가로 통합되어 있다.6 2개의 2D RPLidar는 로봇 주변의 평면적인 장애물을 안정적으로 감지하고 전통적인 2D SLAM 알고리즘을 위한 데이터를 제공하는 역할을 한다. 상단에 장착된 Hesai XT-32 3D LiDAR는 고정밀 3D 맵을 제작하거나, 비전 시스템의 성능을 검증하기 위한 지상 실측 정보(Ground Truth)를 확보하는 데 사용된다.1 이러한 구성은 카메라가 조명 변화나 텍스처가 없는 벽면에 취약한 단점을 LiDAR가 보완하고, 반대로 LiDAR가 유리나 검은색 물체를 인식하지 못하는 한계를 카메라가 극복하는 강력한 상호 보완 및 이중화(redundancy) 체계를 구축한다.

### 1.4 D. 개발자 키트: Nova Orin Developer Kit

NVIDIA는 완제품 형태의 Nova Carter와 함께, 핵심 컴퓨팅 및 센서 모듈만을 패키징한 Nova Orin Developer Kit(DevKit)도 제공한다.13 이는 시장에 대한 이원적 접근 전략을 명확히 보여준다. Nova Carter가 AMR 개발 역량이 부족하거나 빠른 프로토타이핑을 원하는 기업을 대상으로 하는 '즉시 사용 가능한 솔루션'이라면, DevKit은 이미 자체적인 로봇 차체나 플랫폼을 보유한 대형 제조사 및 특정 용도에 맞는 커스텀 로봇을 제작하려는 전문 개발자들을 대상으로 한다.

DevKit은 Jetson AGX Orin 모듈과 3개의 Hawk 스테레오 카메라, 3개의 Owl 어안 카메라를 포함한 핵심 센서부를 하나의 통합된 형태로 제공한다.13 개발자들은 이를 자신들의 로봇에 '이식'함으로써, NVIDIA Isaac이라는 강력한 두뇌와 신경계를 손쉽게 탑재할 수 있다.17 이 키트는 개발자 커뮤니티의 피드백을 반영하여 Nova Carter보다 높은 수준의 커스터마이징 유연성을 제공하며, Isaac Perceptor 스택을 즉시 경험할 수 있도록 관련 소프트웨어가 사전 설치된 상태로 공급되어 개발 주기를 더욱 단축시킨다.13 이 전략을 통해 NVIDIA는 신규 고객을 유치하는 동시에, 기존 로보틱스 플레이어들을 자사 플랫폼 생태계 안으로 끌어들여 기술적 영향력을 기하급수적으로 확장하고 있다.

## 2. III. 소프트웨어 스택의 구조와 핵심 기술

NVIDIA Nova 플랫폼의 진정한 가치는 하드웨어와 유기적으로 결합하여 그 성능을 극대화하는 정교한 소프트웨어 스택에 있다. 이 스택은 개방형 표준인 Isaac ROS를 기반으로, 고성능 3D 인식을 위한 Isaac Perceptor 워크플로우와, 그 핵심을 이루는 cuVSLAM 및 Nvblox와 같은 최첨단 알고리즘으로 구성된다.

### 2.1 A. 기반 프레임워크: Isaac ROS (Robot Operating System)

Isaac ROS는 로보틱스 분야의 사실상 표준인 ROS 2를 기반으로 구축되어, 기존 ROS 개발자들이 익숙한 환경에서 개발을 시작할 수 있도록 진입 장벽을 낮춘다.9 그러나 Isaac ROS의 핵심적인 차별점은 ROS의 유연성과 개방성에 NVIDIA의 CUDA GPU 가속 기술을 완벽하게 결합했다는 데 있다.

이 결합의 중심에는 **NITROS(NVIDIA Isaac Transport for ROS)** 라는 독자적인 기술이 있다.9 전통적인 ROS 2 시스템에서는 각 노드(프로세스)가 데이터를 주고받을 때마다 메모리 복사가 발생하여, 특히 고해상도 이미지나 포인트 클라우드와 같은 대용량 데이터를 처리할 때 심각한 성능 병목 현상을 유발했다. NITROS는 이러한 데이터 파이프라인에서 불필요한 메모리 복사를 원천적으로 제거하는 제로카피(Zero-Copy) 전송 방식을 구현한다. 이를 통해 GPU 메모리에 있는 데이터를 CPU를 거치지 않고 다른 GPU 가속 노드로 직접 전달할 수 있게 되어, 전체 시스템의 처리량(throughput)과 지연 시간(latency)을 극적으로 개선한다. 이는 단순히 개별 패키지의 성능을 높이는 것을 넘어, ROS 2 생태계 전체를 위한 '데이터 고속도로'를 구축하는 것과 같은 전략적 가치를 지닌다. 개발자들은 NITROS가 제공하는 압도적인 성능 향상 때문에 자연스럽게 NVIDIA 플랫폼을 선택하게 되며, 이는 Isaac ROS 생태계의 선순환적 확장과 시장 지배력 강화로 이어진다.

Isaac ROS는 특정 기능을 수행하는 GPU 가속화된 개별 ROS 2 패키지, 즉 **GEMs(Hardware-Accelerated Modules)** 의 집합체로 구성된다. 개발자는 이 GEM들을 레고 블록처럼 조합하여 NITROS라는 고속 파이프라인 위에서 자신만의 고성능 로봇 애플리케이션을 구축할 수 있다.9

#### 2.1.1 핵심 테이블 2: Isaac ROS 주요 패키지(GEMs) 및 기능

Isaac ROS는 수많은 패키지로 구성된 복잡한 소프트웨어 스택이다. 아래 표는 Nova 플랫폼의 자율 주행 파이프라인에서 핵심적인 역할을 담당하는 주요 패키지들을 기능별로 분류하여, 각 기술 요소의 역할과 상호 연관성을 명확히 보여준다.3

| 기능 분류               | 패키지명 (핵심 기술)                       | 설명                                                         | 출처 |
| ----------------------- | ------------------------------------------ | ------------------------------------------------------------ | ---- |
| **측위 및 맵핑**        | `isaac_ros_visual_slam` (cuVSLAM)          | 다중 스테레오 카메라와 IMU 데이터를 활용한 고성능, GPU 가속 시각 SLAM. 실시간 주행 거리 측정(Odometry) 및 루프 폐쇄 기능 제공. | 9    |
|                         | `isaac_ros_nvblox` (Nvblox)                | 깊이 센서(카메라/LiDAR) 데이터로부터 실시간 3D 장면 재구성. 동적 객체 필터링 및 Nav2용 2D Costmap 생성. | 3    |
| **인식 (Perception)**   | `isaac_ros_ess` / `isaac_ros_bi3d`         | DNN 기반 스테레오 깊이 추정. 고밀도의 정확한 깊이 이미지 생성. | 9    |
|                         | `isaac_ros_unet` / `isaac_ros_segformer`   | DNN 기반 이미지 분할(Segmentation). 사람, 자유 공간 등 의미론적 정보 추출. | 9    |
|                         | `isaac_ros_detectnet` / `isaac_ros_yolov8` | DNN 기반 2D 객체 탐지.                                       | 9    |
| **조작 (Manipulation)** | `isaac_ros_foundationpose`                 | 파운데이션 모델 기반 6D 객체 자세 추정. 처음 보는 물체에도 적용 가능. | 9    |
|                         | `isaac_ros_cumotion`                       | CUDA 가속 로봇 팔 모션 플래닝. 충돌 회피 및 최적 경로 생성.  | 9    |
| **하드웨어 드라이버**   | `isaac_ros_hawk` / `isaac_ros_owl`         | Nova 플랫폼의 Hawk, Owl 카메라를 위한 최적화된 ROS 2 드라이버. | 9    |

### 2.2 B. 3D 인식 스택: Isaac Perceptor

Isaac Perceptor는 개별 Isaac ROS GEM들을 유기적으로 조합하여 만든 **레퍼런스 워크플로우(Reference Workflow)** 로, 특히 Nova Orin의 다중 카메라 하드웨어를 활용한 고성능 3D 서라운드 비전 기능 구현에 초점을 맞춘다.10 이는 개발자가 cuVSLAM, Nvblox, DNN 모델 등 복잡한 패키지들을 일일이 조합하고 파라미터를 튜닝하는 수고를 덜어주며, Nova Orin 플랫폼에서 즉시 최적화된 3D 인식 기능을 사용할 수 있도록 지원하는 일종의 '솔루션 템플릿'이다.16 Isaac Perceptor의 핵심 기능은 cuVSLAM을 이용한 강인한 실시간 위치 추정, Nvblox를 통한 동적 환경에서의 3D 맵핑 및 장애물 감지, 그리고 DNN을 활용한 의미론적 이해(semantic understanding)를 통합하여 제공하는 것이다.19

### 2.3 C. 핵심 알고리즘 1: Visual SLAM (cuVSLAM)과 Bundle Adjustment

cuVSLAM은 시각 동시적 위치추정 및 지도작성(Visual SLAM) 기술의 핵심으로, 전통적인 SLAM 알고리즘에서 가장 계산 비용이 높은 최적화 과정인 **번들 조정(Bundle Adjustment, BA)** 을 대규모 병렬 처리가 가능한 GPU 아키텍처에 맞게 재설계하여 성능을 극대화한 것이다.9

BA의 근본적인 목표는 로봇이 이동하면서 촬영한 여러 이미지에 걸쳐 관측된 특징점(feature)들의 위치를 가장 잘 설명할 수 있는 일련의 카메라 포즈(자세, $\mathbf{T}_i$)와 3차원 랜드마크(특징점의 3D 위치, $\mathbf{p}_j$)들의 집합을 동시에 추정하는 것이다. 이는 추정된 3D 랜드마크를 각 카메라의 추정된 포즈를 기준으로 2D 이미지 평면에 다시 투영(reproject)했을 때, 실제 이미지에서 관측된 특징점의 위치와의 차이, 즉 **재투영 오차(reprojection error)** 를 최소화하는 거대한 비선형 최소제곱(nonlinear least-squares) 최적화 문제로 귀결된다.21 이 최적화 문제는 수학적으로 다음과 같이 정의된다:
$$
\{\mathbf{T}^*_i, \mathbf{p}^*_j\} = \arg\min_{\mathbf{T}_i, \mathbf{p}_j} \sum_{i} \sum_{j} v_{ij} \rho \left( \left\| \pi(\mathbf{T}_i, \mathbf{p}_j) - \mathbf{z}_{ij} \right\|^2_{\Sigma_{ij}} \right)
$$
위 식에서 각 기호는 다음을 의미한다:

- $\mathbf{T}_i \in SE(3)$: i번째 카메라 프레임의 6자유도(6-DoF) 포즈 (위치 및 방향).
- $\mathbf{p}_j \in \mathbb{R}^3$: j번째 랜드마크의 3차원 월드 좌표.
- $\mathbf{z}_{ij} \in \mathbb{R}^2$: 이미지 i에서 관측된 랜드마크 j의 2차원 픽셀 좌표.
- $\pi(\cdot)$: 3D 포인트 $\mathbf{p}_j$를 카메라 포즈 $\mathbf{T}_i$를 이용해 이미지 평면으로 투영하는 함수.
- $v_{ij}$: 랜드마크 j가 이미지 i에서 관측되었으면 1, 아니면 0인 가시성 변수.
- $\Sigma_{ij}$: 측정 노이즈의 불확실성을 나타내는 공분산 행렬.
- $\rho(\cdot)$: 잘못된 특징점 매칭(outlier)으로 인한 오차의 영향을 줄이기 위한 강인한 비용 함수(robust cost function, 예: Huber loss).

이 문제는 수백 개의 카메라 포즈와 수만 개의 랜드마크를 동시에 최적화해야 하므로 변수의 수가 매우 많고, 투영 함수 $\pi(\cdot)$의 비선형성 때문에 계산이 매우 복잡하다. cuVSLAM은 이 거대한 최적화 문제를 CUDA를 통해 GPU에서 병렬로 처리함으로써, CPU 기반의 전통적인 VSLAM 알고리즘으로는 불가능했던 실시간 처리 성능(250 fps 이상)을 달성한다.9

### 2.4 D. 핵심 알고리즘 2: 3D 장면 재구성 (Nvblox)과 Graph-Based SLAM

Nvblox는 로봇 주변 환경을 실시간으로 3차원으로 재구성하고, 이를 기반으로 자율 주행을 위한 네비게이션 맵을 생성하는 핵심적인 역할을 담당한다. 이 기술의 근간에는 **그래프 기반 SLAM(Graph-Based SLAM)** 의 원리가 자리 잡고 있다.25 그래프 기반 SLAM은 로봇의 시간에 따른 연속적인 포즈들을 그래프의 노드(node)로 표현하고, 한 포즈에서 다음 포즈로의 움직임(odometry)이나 특정 포즈에서 관측된 랜드마크와의 관계를 엣지(edge)로 표현하여 전체적인 SLAM 문제를 하나의 거대한 최적화 문제로 푸는 방식이다.

Nvblox는 cuVSLAM과 같은 외부 소스로부터 실시간으로 제공되는 로봇의 정확한 포즈($\mathbf{x}_t$)와, 해당 포즈에서 스테레오 카메라나 3D LiDAR를 통해 얻은 깊이 정보($D_t$)를 입력받는다. 그리고 이 정보를 TSDF(Truncated Signed Distance Field) 또는 ESDF(Euclidean Signed Distance Field)와 같은 효율적인 자료 구조를 사용하여 3차원 복셀 맵(voxel map)에 점진적으로 통합하고 갱신한다.

그래프 기반 SLAM의 최적화 목표는 모든 제약 조건(엣지)에 의해 발생하는 누적 오차를 최소화하는 노드(포즈)들의 전역적으로 일관된(globally consistent) 배치를 찾는 것이다. 이는 수학적으로 다음과 같은 최소제곱 문제로 공식화될 수 있다 27:
$$
\mathbf{X}^* = \arg\min_{\mathbf{X}} \sum_{\langle i,j \rangle \in \mathcal{C}} \mathbf{e}_{ij}(\mathbf{x}_i, \mathbf{x}_j)^T \Omega_{ij} \mathbf{e}_{ij}(\mathbf{x}_i, \mathbf{x}_j)
$$
위 식에서 각 기호는 다음을 의미한다:

- $\mathbf{X} = \{\mathbf{x}_i\}$: 최적화하려는 모든 로봇 포즈의 집합.
- $\mathcal{C}$: 그래프의 모든 제약 조건(엣지)의 집합.
- $\mathbf{e}_{ij}(\mathbf{x}_i, \mathbf{x}_j) = h(\mathbf{x}_i, \mathbf{x}_j) - \mathbf{z}_{ij}$: 포즈 $\mathbf{x}_i$와 $\mathbf{x}_j$ 사이의 예측된 관계($h(\cdot)$)와 실제 측정값($\mathbf{z}_{ij}$, 예: odometry) 사이의 오차 벡터.
- $\Omega_{ij}$: 측정의 신뢰도를 나타내는 정보 행렬(information matrix), 공분산 행렬의 역행렬.

Nvblox의 중요한 특징 중 하나는 동적 환경에 대한 강인성이다. UNet과 같은 의미론적 분할(semantic segmentation) DNN 모델을 활용하여, 입력되는 컬러 이미지에서 사람이나 다른 이동체와 같은 동적 객체를 사전에 식별하고, 해당 영역을 3D 맵 업데이트 과정에서 제외할 수 있다.3 이를 통해 로봇은 사람들이 분주하게 움직이는 복잡한 창고 환경 속에서도 움직이지 않는 정적인 구조물만의 일관된 지도를 구축할 수 있다. 마지막으로, Nvblox는 이렇게 구축된 3D 복셀 맵에서 로봇이 주행하는 평면의 높이를 기준으로 단면을 잘라내어(slice), ROS 2 Nav2와 같은 표준 네비게이션 스택이 장애물 회피에 직접 사용할 수 있는 2D 비용 지도(costmap)를 실시간으로 생성하여 제공한다.3

이처럼 cuVSLAM과 Nvblox는 각각 '자신의 위치를 아는 것(측위)'과 '주변 환경을 이해하는 것(맵핑)'을 담당하지만, 이 둘은 단순한 조합을 넘어선 깊은 공생 관계에 있다. Nvblox가 정확한 3D 맵을 만들려면 cuVSLAM이 제공하는 정확한 포즈 정보가 필수적이며 3, 반대로 cuVSLAM이 과거에 방문했던 장소를 다시 인식하고 경로 오차를 보정(loop closure)할 때 Nvblox가 구축한 3D 맵은 중요한 참조 정보가 될 수 있다. 이 두 핵심 알고리즘의 GPU 가속 기반 실시간 연동은 Isaac Perceptor 성능의 근간을 이루는 강력한 시너지 효과를 창출한다.

## 3. IV. 개발 생태계와 시뮬레이션 기반 워크플로우

NVIDIA Nova 플랫폼의 경쟁력은 단순히 하드웨어와 소프트웨어의 성능을 넘어, 개발, 테스트, 배포의 전 과정을 아우르는 강력한 개발 생태계에서 나온다. 그 중심에는 물리적으로 정확한 시뮬레이션 환경을 제공하는 NVIDIA Isaac Sim이 있다.

### 3.1 A. 디지털 트윈의 역할: NVIDIA Isaac Sim on Omniverse

Isaac Sim은 NVIDIA의 실시간 3D 협업 및 시뮬레이션 플랫폼인 Omniverse 위에서 구축된 로보틱스 특화 애플리케이션이다.7 이는 픽사(Pixar)에서 개발한 오픈소스 3D 장면 기술 표준인 USD(Universal Scene Description)와 고성능 물리 엔진인 PhysX를 기반으로, 실제 세계와 거의 동일한 물리 법칙(중력, 마찰, 충돌 등)이 적용되는 극사실적인 가상 환경을 제공한다.7

개발자들은 이 가상 환경 속에 Nova Carter의 정밀한 디지털 트윈 모델을 배치할 수 있다.4 이 디지털 트윈은 로봇의 외형뿐만 아니라 센서의 특성(카메라 왜곡, LiDAR 스캔 패턴 등), 구동계의 동역학적 특성까지 정밀하게 시뮬레이션한다. 이를 통해 개발자는 실제 로봇 하드웨어 없이도 소프트웨어 스택 전체의 로직과 성능을 테스트하는 

**소프트웨어 인 더 루프(Software-in-the-Loop, SIL)** 시뮬레이션을 수행할 수 있다. 더 나아가, 시뮬레이션 환경을 실제 로봇의 컴퓨팅 하드웨어(Jetson AGX Orin)와 연동하여 실시간으로 상호작용하게 하는 **하드웨어 인 더 루프(Hardware-in-the-Loop, HIL)** 테스트도 가능하다. 이러한 시뮬레이션 기반 워크플로우는 위험하고 비용이 많이 들며 시간이 오래 걸리는 물리적 테스트를 최소화하고, 개발 초기 단계에서 알고리즘의 잠재적인 문제를 사전에 발견하여 수정할 기회를 제공함으로써 전체 개발 수명주기를 획기적으로 단축시킨다.2

### 3.2 B. 합성 데이터 생성(Synthetic Data Generation)과 AI 모델 훈련

과거의 시뮬레이터가 주로 알고리즘의 논리적 오류를 검증하는 '디버깅 도구'에 머물렀다면, Isaac Sim은 여기서 한 단계 더 나아가 AI 모델을 '생산'하는 공장의 역할을 수행한다. Isaac Sim은 절차적 생성(procedural generation)과 영역 무작위화(domain randomization)와 같은 기술을 통해, 다양한 조명 조건, 물체 배치, 재질 등을 가진 수많은 가상 환경을 자동으로 생성할 수 있다. 이 환경에서 로봇의 센서 데이터를 시뮬레이션하면, 픽셀 단위의 정확한 의미론적 분할 마스크, 깊이 정보, 경계 상자(bounding box) 등 완벽한 주석(annotation)이 달린 방대한 양의 **합성 데이터(Synthetic Data)** 를 자동으로 생성할 수 있다.7

이는 데이터의 양과 질이 AI 모델의 성능을 좌우하는 딥러닝 시대에 결정적인 경쟁 우위를 제공한다. 실제 세계에서는 수집이 거의 불가능하거나 매우 위험한 엣지 케이스(edge case), 예를 들어 어두운 창고에서 갑자기 사람이 나타나는 시나리오나 특정 형태의 장애물이 경로를 막는 경우 등에 대한 데이터를 무한정 생성하여 AI 인식 모델의 강인성(robustness)을 획기적으로 높일 수 있다.30 24시간 가동되는 이 가상 데이터 공장은 실제 데이터 수집에 따르는 물리적, 법적, 비용적 제약에서 개발자를 해방시킨다.

더 나아가, NVIDIA Cosmos와 같은 최신 월드 파운데이션 모델은 실제 센서 데이터를 기반으로 상호작용이 가능한 사실적인 시뮬레이션 환경을 생성하는 기술로, Isaac Sim의 합성 데이터 생성 능력을 더욱 고도화하여 현실과 가상의 간극(sim-to-real gap)을 줄일 잠재력을 가지고 있다.7 이처럼 Isaac Sim과 Nova 플랫폼은 단순히 AMR 개발을 가속화하는 것을 넘어, NVIDIA가 주도하는 '산업 메타버스' 생태계로 기업들을 끌어들이는 핵심적인 관문 역할을 수행한다. Omniverse의 장기적인 비전은 로봇뿐만 아니라 공장 설비, 작업자 등 산업 현장의 모든 요소를 디지털 트윈으로 연결하여, 가상 세계에서 전체 공장 운영을 최적화하고 그 결과를 실제 세계에 즉시 반영하는 것이다.31

## 4. V. 산업 적용 사례 및 시장 분석

NVIDIA Nova 플랫폼은 이론적 가능성을 넘어 실제 산업 현장에서 가시적인 성과를 창출하며 그 가치를 입증하고 있다. 특히 변화가 빠르고 효율성 요구가 높은 물류 및 제조 자동화 분야에서 선도적인 기업들과의 협력을 통해 기술의 상용화를 이끌고 있다.

### 4.1 A. 물류 및 제조 자동화

창고, 유통 센터, 공장 내에서 자재를 입고, 보관, 피킹, 출고하는 일련의 프로세스는 AMR의 가장 크고 중요한 적용 분야이다. Nova 플랫폼은 이러한 역동적이고 복잡한 환경에서 인간 작업자와의 충돌을 피하고 안전하게 공존하며, 전체 물류 흐름의 효율성을 극대화하도록 설계되었다.1

**ArcBest 사례 분석**: 미국의 물류 대기업 ArcBest는 자사의 차세대 자율 지게차 및 리치 트럭 솔루션인 'Vaux Smart Autonomy'에 NVIDIA Isaac Perceptor를 핵심 인식 기술로 채택했다.8 주목할 점은 ArcBest가 기존에 사용하던 고가의 3D LiDAR 기반 인식 시스템에서 NVIDIA의 시각 AI(Visual AI) 기술로 전환했다는 사실이다. 이러한 결정의 배경에는 단순한 비용 절감을 넘어선 명확한 기술적 우위가 존재한다. ArcBest에 따르면, Isaac Perceptor는 다중 카메라를 통해 초당 1,650만 개 이상의 3D 포인트 클라우드를 생성하여, 기존 3D LiDAR가 종종 놓치기 쉬웠던 낮은 높이의 장애물(예: 지게차 포크)까지 정밀하게 감지하고 3D 점유 격자 지도(occupancy map)를 생성할 수 있다.8 이는 창고 내에서의 안전 사고를 예방하고, 더 효율적이고 유연한 자율 주행 경로 계획을 가능하게 한다. 이 사례는 비전 중심 인식 기술이 더 이상 LiDAR의 저가형 대안이 아니라, 특정 산업 환경에서는 오히려 성능적으로 더 우월할 수 있음을 입증하는 강력한 증거이다.

**주요 파트너십**: Nova 플랫폼의 영향력은 ArcBest를 넘어 산업 전반으로 빠르게 확산되고 있다. 세계 최대의 전기차 및 지게차 제조업체인 BYD, 유럽 창고 자동화 솔루션의 강자인 KION Group, 산업 자동화 분야의 글로벌 리더인 Rockwell Automation(OTTO Motors 인수) 등 각 분야의 핵심 기업들이 Isaac Perceptor 및 Nova 플랫폼을 자사의 AMR 솔루션에 도입하고 있다.20 이는 NVIDIA의 기술이 특정 틈새시장을 넘어 광범위한 산업 자동화 분야의 표준 기술로 자리매김하고 있음을 시사한다.

### 4.2 B. 경쟁 환경 및 플랫폼의 한계

NVIDIA Nova가 AMR 개발의 새로운 패러다임을 제시하고 있지만, 시장에는 각기 다른 접근법과 강점을 가진 경쟁 플랫폼들이 존재한다.

**경쟁 플랫폼 비교**:

- **Boston Dynamics Spot**: 4족 보행 로봇인 Spot은 계단이나 비포장도로와 같은 험지를 극복하는 탁월한 기동성을 바탕으로, 건설 현장 감리, 위험 지역 검사, 보안 순찰 등 특수 목적의 임무 수행에 독보적인 강점을 보인다.36 반면, Nova Carter는 평탄한 실내 환경에서의 효율적인 물류 운송에 최적화된 바퀴 기반 플랫폼이다. 페이로드 용량에서도 Nova Carter(50kg)가 Spot(14kg)을 크게 앞선다.12 기술적 접근법에서 Spot은 로봇 자체의 기계적 완성도와 동역학 제어 기술에, Nova는 개방형 개발 생태계와 AI 기반의 소프트웨어 스택에 더 큰 중점을 둔다.
- **OTTO Motors OTTO 1500**: OTTO 1500은 Nova Carter와 마찬가지로 실내 물류 자동화를 목표로 하지만, 최대 1900kg에 달하는 초고중량 화물을 운송하는 데 특화되어 있어 목표 시장이 명확히 구분된다.38 OTTO Motors는 수년간 산업 현장에서 검증된 신뢰성과 내구성을 핵심 경쟁력으로 내세우는 반면, NVIDIA Nova는 최첨단 AI 인식 기술과 개발자에게 제공되는 높은 수준의 유연성을 차별점으로 강조한다.

이러한 비교를 통해 각 플랫폼의 전략적 차이가 드러난다. Boston Dynamics와 OTTO Motors는 고도로 완성된 로봇 '제품'을 판매하는 데 주력하는 반면, NVIDIA의 Nova는 그 자체가 최종 제품이라기보다는 다른 회사들이 자신들의 AMR 제품을 만들 수 있도록 지원하는 '개발 플랫폼'이자 '생태계'이다. 이는 NVIDIA가 직접 모든 AMR 시장의 경쟁자와 싸우는 대신, 모든 AMR 제조사를 잠재적인 고객이자 파트너로 만드는 훨씬 확장성 높은 '청바지와 곡괭이를 파는' 전략을 구사하고 있음을 보여준다.

**플랫폼의 잠재적 한계 고찰**:

- **높은 컴퓨팅 요구사항**: Isaac Perceptor와 같은 복잡한 AI 스택은 Jetson AGX Orin의 강력한 GPU 성능을 전제로 설계되었기 때문에, 저사양의 임베디드 컴퓨팅 플랫폼으로 이식하기 어렵다. 이는 플랫폼 도입의 초기 비용을 높이는 요인이 될 수 있으며, 추가적인 센서를 활성화할 경우 계산 부하가 더욱 증가한다.41
- **하드웨어 종속성**: 소프트웨어 스택이 Nova Orin의 특정 센서 구성(예: GMSL 인터페이스 카메라)에 맞춰 고도로 최적화되어 있어, 다른 종류의 카메라나 LiDAR를 사용하려는 개발자는 상당한 수준의 커스터마이징 및 드라이버 개발 노력을 기울여야 할 수 있다.42
- **비전 기반 인식의 내재적 취약점**: 아무리 발전된 AI 모델이라 할지라도, 극단적인 조명 변화(강한 역광, 갑작스러운 암전), 짙은 안개나 먼지, 또는 렌즈 오염과 같이 시각 정보를 근본적으로 왜곡하는 환경에서는 인식 성능이 저하될 수 있다. Nova 플랫폼이 LiDAR와의 상호보완 설계를 통해 이러한 취약점을 완화하고 있지만, 비전 기술을 주력으로 사용하는 만큼 이러한 내재적 한계는 명확히 존재한다.

## 5. VI. 결론: 물리적 AI 시대를 여는 Nova 플랫폼의 미래 전망

### 5.1 A. AMR 개발 생태계에 미치는 영향 종합

NVIDIA Nova 플랫폼은 자율 이동 로봇 개발의 패러다임을 근본적으로 바꾸고 있다. 과거 AMR 개발이 고도의 전문성을 요구하는 소수 기업의 영역이었다면, Nova는 표준화되고 사전 통합된 고성능 하드웨어와 소프트웨어 스택을 제공함으로써 개발의 진입 장벽을 획기적으로 낮추고 있다. 개발자들은 더 이상 저수준의 하드웨어 통합, 센서 보정, 디바이스 드라이버 개발과 같은 반복적이고 비차별적인 작업에 시간을 낭비하는 대신, 자사의 도메인 지식을 활용하여 부가가치가 높은 애플리케이션과 서비스를 창출하는 데 역량을 집중할 수 있게 되었다. 이는 전체 AMR 산업의 혁신 속도를 가속화하고, 더욱 다양하고 지능적인 로봇 솔루션의 등장을 촉진하는 기폭제가 되고 있다.

### 5.2 B. 차세대 기술과의 융합: 생성형 AI와 파운데이션 모델

Nova 플랫폼의 미래는 현재의 인식 및 자율 주행 기술을 넘어, 생성형 AI(Generative AI) 및 파운데이션 모델(Foundation Models)과의 융합을 통해 더욱 확장될 것이다. NVIDIA는 최근 인간형 로봇을 위한 범용 AI 에이전트 모델인 'Project GR00T'를 발표하며, 로보틱스의 다음 단계를 향한 비전을 제시했다.10 이는 자연어 명령을 이해하고, 비디오 시연을 통해 새로운 기술을 학습하며, 복잡하고 예측 불가능한 환경과 상호작용할 수 있는 범용 로봇 인텔리전스를 목표로 한다.

가까운 미래에, Isaac Perceptor를 통해 수집된 방대한 실제 세계의 시각 데이터를 기반으로 훈련된 거대 언어-비전 모델(Large Vision-Language Models, LVLM)이 Nova 플랫폼에 탑재될 것이다. 이를 통해 AMR은 "저 선반 위에 있는 파란색 상자를 A-3 구역으로 옮겨줘"와 같이 추상적이고 모호한 인간의 언어 명령을 이해하고, 스스로 주변 환경을 인식하여 작업 계획을 수립하며, 돌발 상황에 대처하여 임무를 완수하는 진정한 의미의 **'물리적 AI(Physical AI)'** 에이전트로 진화하게 될 것이다.43

### 5.3 C. 전략적 제언 및 미래 전망

AMR의 도입을 고려하는 기업들은 더 이상 특정 기능이나 스펙을 가진 개별 하드웨어 '제품'을 구매하는 단기적인 관점에서 벗어나, Nova와 같이 지속적으로 발전하고 확장 가능한 '플랫폼'을 도입하는 장기적인 전략적 관점을 가져야 한다. 이는 단순히 현재의 문제를 해결하는 것을 넘어, 향후 소프트웨어 업데이트, AI 모델 개선, 그리고 더 넓은 Omniverse 디지털 트윈 생태계와의 연동을 통해 지속적인 운영 효율성 향상과 새로운 가치 창출을 가능하게 할 것이다.

결론적으로, NVIDIA Nova 플랫폼은 단순한 AMR 개발 도구의 집합이 아니다. 이는 NVIDIA가 지난 수십 년간 데이터 센터와 AI 연구를 통해 축적한 압도적인 기술력을 현실 세계의 물리적 공간으로 확장하고, 모든 사물이 지능을 갖게 되는 미래를 구현하려는 거대한 비전이 응축된 산물이다. 이 플랫폼의 성공적인 확산은 향후 10년간 물류와 제조 산업을 필두로 서비스, 헬스케어, 농업 등 인간 사회의 거의 모든 영역에서 자동화의 지형을 근본적으로 재편하는 결정적인 변곡점이 될 것이다.10

#### **참고 자료**

1. NVIDIA Isaac Nova Orin Opens New Era of Innovation for Autonomous Mobile Robots, 8월 15, 2025에 액세스, https://blogs.nvidia.com/blog/nvidia-isaac-nova-orin/
2. NVIDIA Isaac AMR | End-to-End Autonomy Platform for Next Generation AMRs - YouTube, 8월 15, 2025에 액세스, https://www.youtube.com/watch?v=SMZCio24Zx4
3. Towards Next-Gen Autonomous Mobile Robotics: A Technical Deep ..., 8월 15, 2025에 액세스, https://www.kudan.io/blog/a-technical-deep-dive-into-visual-data-driven-amrs-powered-by-kdvisual-and-nvidia-isaac-perceptor/
4. NVIDIA gives mobile robots all-around sight with Nova Carter - The Robot Report, 8월 15, 2025에 액세스, https://www.therobotreport.com/rbr50-company-2024/nvidia-gives-mobile-robots-all-around-sight-with-nova-carter/
5. RBR50 Spotlight: NVIDIA gives mobile robots all-around sight with Nova Carter, 8월 15, 2025에 액세스, https://www.therobotreport.com/rbr50-spotlight-nvidia-gives-mobile-robots-all-around-sight-with-nova-carter/
6. NVIDIA Brings Advanced Autonomy to Mobile Robots With Isaac ..., 8월 15, 2025에 액세스, https://blogs.nvidia.com/blog/isaac-amr-nova-orin-autonomous-mobile-robots/
7. Isaac Sim - Robotics Simulation and Synthetic Data Generation - NVIDIA Developer, 8월 15, 2025에 액세스, https://developer.nvidia.com/isaac/sim
8. ArcBest Helps Bridge the Gap Between Robotics and Logistics Using NVIDIA Technology, 8월 15, 2025에 액세스, https://investors.arcb.com/news-events/news/News-Details/2024/ArcBest-Helps-Bridge-the-Gap-Between-Robotics-and-Logistics-Using-NVIDIA-Technology/
9. Isaac ROS (Robot Operating System) | NVIDIA Developer, 8월 15, 2025에 액세스, https://developer.nvidia.com/isaac/ros
10. NVIDIA Isaac - AI Robot Development Platform, 8월 15, 2025에 액세스, https://developer.nvidia.com/isaac
11. Accelerate AI-Enabled Robotics with Advanced Simulation and Perception Tools on NVIDIA Isaac Platform, 8월 15, 2025에 액세스, https://developer.nvidia.com/blog/accelerate-ai-enabled-robotics-with-advanced-simulation-and-perception-tools-in-nvidia-isaac-platform/
12. Nova Carter Product Manual 20231031 - Segway Robotics, 8월 15, 2025에 액세스, https://robotics.segway.com/wp-content/uploads/2023/11/Nova-Carter-Product-Manual-v1.0.pdf
13. Nova Orin Developer Kit - Segway Robotics, 8월 15, 2025에 액세스, https://robotics.segway.com/nova-dev-kit/
14. Nova Carter - Segway Robotics, 8월 15, 2025에 액세스, https://robotics.segway.com/nova-carter/
15. Nova Carter - Isaac Sim Documentation, 8월 15, 2025에 액세스, https://docs.isaacsim.omniverse.nvidia.com/4.2.0/landing_pages/nova_carter_landing_page.html
16. NVIDIA Isaac Perceptor - NVIDIA Developer, 8월 15, 2025에 액세스, https://developer.nvidia.com/isaac/perceptor
17. Segway Collaborates with NVIDIA to Introduce NVIDIA Isaac-powered Nova Orin Developer Kit for Autonomous Mobile Robots, 8월 15, 2025에 액세스, https://robotics.segway.com/news/segway-collaborates-with-nvidia-to-introduce-nvidia-isaac-powered-nova-orin-developer-kit/
18. NVIDIA is Committed to reshape the AMR Industry - AGV Network, 8월 15, 2025에 액세스, https://www.agvnetwork.com/news/nvidia-is-committed-to-reshape-the-amr-industry
19. Advancing Robot Learning, Perception, and Manipulation with Latest NVIDIA Isaac Release, 8월 15, 2025에 액세스, https://developer.nvidia.com/blog/advancing-robot-learning-perception-and-manipulation-with-latest-nvidia-isaac-release/
20. NVIDIA Isaac Taps Generative AI for Manufacturing and Logistics Applications, 8월 15, 2025에 액세스, https://blogs.nvidia.com/blog/isaac-generative-ai-manufacturing-logistics/
21. Bundle Adjustment (BA) - Steven Gong, 8월 15, 2025에 액세스, https://stevengong.co/notes/Bundle-Adjustment
22. Visual SLAM Tutorial: Bundle Adjustment, 8월 15, 2025에 액세스, https://www.cs.cmu.edu/~kaess/vslam_cvpr14/media/VSLAM-Tutorial-CVPR14-A13-BundleAdjustment-handout.pdf
23. Bundle adjustment - Wikipedia, 8월 15, 2025에 액세스, https://en.wikipedia.org/wiki/Bundle_adjustment
24. Photometric Bundle Adjustment for Vision-Based SLAM - arXiv, 8월 15, 2025에 액세스, http://arxiv.org/pdf/1608.02026
25. www2.informatik.uni-freiburg.de, 8월 15, 2025에 액세스, http://www2.informatik.uni-freiburg.de/~stachnis/pdf/grisetti10titsmag.pdf
26. Graph SLAM: From Theory to Implementation - Federico Sarrocco, 8월 15, 2025에 액세스, https://federicosarrocco.com/blog/graph-slam-tutorial
27. Mastering Graph-Based SLAM in Topological Robotics - Number Analytics, 8월 15, 2025에 액세스, https://www.numberanalytics.com/blog/graph-based-slam-topological-robotics
28. The GraphSLAM Algorithm with Applications to Large-Scale Mapping of Urban Structures - Sebastian Thrun, 8월 15, 2025에 액세스, http://robots.stanford.edu/papers/thrun.graphslam.pdf
29. Nova Carter - Isaac Sim Documentation, 8월 15, 2025에 액세스, https://docs.isaacsim.omniverse.nvidia.com/4.5.0/assets/nova_carter_landing_page.html
30. AI for Robotics - NVIDIA, 8월 15, 2025에 액세스, https://www.nvidia.com/en-us/industries/robotics/
31. The New Isaac AMR Platform (Full Version) - YouTube, 8월 15, 2025에 액세스, https://www.youtube.com/watch?v=79QZrREqD6g
32. Robotic Revolution: Rockwell and NVIDIA Forge Ahead in Automation - AGV Network, 8월 15, 2025에 액세스, https://www.agvnetwork.com/news/robotic-revolution-rockwell-and-nvidia-forge-ahead-in-automation
33. ArcBest partners with NVIDIA for AI-driven logistics tech - Investing.com, 8월 15, 2025에 액세스, https://www.investing.com/news/stock-market-news/arcbest-partners-with-nvidia-for-aidriven-logistics-tech-93CH-3342992
34. Gideon and NVIDIA Power Up AI-Driven Supply Chain - AGV Network, 8월 15, 2025에 액세스, https://www.agvnetwork.com/news/gideon-and-nvidia-power-up-ai-driven-supply-chain
35. NVIDIA Robotics Adopted by Industry Leaders for Development of Tens of Millions of AI-Powered Autonomous Machines, 8월 15, 2025에 액세스, https://nvidianews.nvidia.com/news/robotics-industry-development-ai-autonomous-machines
36. Spot robot - Boston Dynamics, 8월 15, 2025에 액세스, https://www.generationrobots.com/en/527-spot-from-boston-dynamics-an-autonomous-four-legged-robot
37. Spot | Boston Dynamics, 8월 15, 2025에 액세스, https://bostondynamics.com/products/spot/
38. OTTO 1500 Autonomous Mobile Robot | OTTO by Rockwell Automation, 8월 15, 2025에 액세스, https://ottomotors.com/1500/
39. OTTO 1500 v2, 8월 15, 2025에 액세스, https://www.amrobotics.eu/wp-content/uploads/2022/12/OTTO-1500-v2-Data-Sheet.pdf
40. OTTO 1500 - Toyota Material Handling Systems, 8월 15, 2025에 액세스, https://www.atlantaforklifts.com/wp-content/uploads/2022/09/2024-OTTO1500-DS001C-EN-FEB2024.pdf
41. Nova Carter - Isaac Sim Documentation, 8월 15, 2025에 액세스, https://docs.isaacsim.omniverse.nvidia.com/latest/assets/nova_carter_landing_page.html
42. NVIDIA Nova — nova_docs documentation - NVIDIA Isaac ROS, 8월 15, 2025에 액세스, https://nvidia-isaac-ros.github.io/nova/index.html
43. GTC March 2025 Keynote with NVIDIA CEO Jensen Huang - YouTube, 8월 15, 2025에 액세스, https://www.youtube.com/watch?v=_waPvOwL9Z8
44. Keynote Wrap-Up: Turning Data Centers into 'AI Factories,' NVIDIA CEO Intros Hopper Architecture, H100 GPU, New Supercomputers, Software, 8월 15, 2025에 액세스, https://blogs.nvidia.com/blog/ai-factories-hopper-h100-nvidia-ceo-jensen-huang/