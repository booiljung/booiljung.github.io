# Isaac Sim에서의 비전 기반 강화학습 아키텍처
[NVidia Omniverse](./index.md)


본 보고서는 NVIDIA Isaac Sim 환경에서 동역학 시뮬레이션을 활용한 비전 기반 강화학습(Vision-based Reinforcement Learning)을 수행할 때, 로봇 운영체제(Robot Operating System, ROS) 2의 필요성에 대한 심층적인 기술적 분석을 제공합니다. 사용자의 질문인 "Isaac Sim에서 동역학적 시뮬레이션을 수행하여 비전 기반 강화학습을 할 때 ROS 2가 필수적인가?"에 대한 명확한 답변부터 제시하자면, **결론적으로 ROS 2는 강화학습 \*훈련(training)\* 단계에서 필수적이지 않으며, 오히려 최고 성능을 위해서는 권장되지 않습니다.**

복잡하고 데이터 집약적인 비전 기반 정책을 훈련시키는 데 요구되는 극한의 성능을 달성하기 위한 가장 효율적이고 권장되는 아키텍처는 **Isaac Lab이 제공하는 Direct Python API 파이프라인**입니다.1 이 접근 방식은 미들웨어로 인한 오버헤드를 원천적으로 배제하고, 시뮬레이션과 학습 알고리즘 간의 데이터 교환을 GPU 내에서 직접 처리하여 훈련 속도를 극대화합니다.

본 보고서의 핵심 주제는 로보틱스 개발에서 **훈련 환경**과 **배포 환경**을 명확히 구분하는 것의 중요성입니다. Direct API가 훈련 단계에서 압도적인 성능 우위를 보이는 반면, ROS 2는 실제 로봇의 소프트웨어 통합, 통신, 그리고 최종적인 배포(deployment) 단계에서 사실상의 산업 표준(de facto industry standard)으로 자리 잡고 있습니다.2

따라서, 연구 개발부터 실제 적용까지 아우르는 가장 효과적인 전문 워크플로우는 이 두 가지 아키텍처의 장점을 모두 활용하는 **하이브리드 접근 방식(Hybrid Approach)**입니다. 이 워크플로우는 다음과 같은 단계로 구성됩니다:

1. **훈련 (Training):** Isaac Lab의 Direct API, 즉 GPU 네이티브 파이프라인을 사용하여 최대 속도로 강화학습 정책을 훈련합니다.
2. **내보내기 (Export):** 훈련된 정책을 ONNX(Open Neural Network Exchange)와 같은 표준화된 형식으로 내보냅니다.4
3. **배포 (Deployment):** 내보낸 정책을 물리적 로봇의 경량 ROS 2 노드 내에서 로드하여 실제 환경에서의 추론(inference)을 수행합니다.

이러한 하이브리드 워크플로우는 훈련의 효율성과 배포의 호환성 및 표준성을 모두 만족시키는 최적의 설계 패턴입니다. 이어지는 섹션들에서는 각 아키텍처의 기술적 구조, 데이터 흐름, 성능 벤치마크, 그리고 실제 워크플로우를 심층적으로 분석하여, NVIDIA의 공식 문서, 관련 연구 논문, 그리고 커뮤니티의 실증적 데이터를 기반으로 위 핵심 결론을 상세히 뒷받침할 것입니다.


강화학습, 특히 비전 기반 강화학습의 정책을 효율적으로 훈련시키기 위해서는 막대한 양의 데이터를 신속하게 처리할 수 있는 능력이 필수적입니다. NVIDIA는 이러한 요구사항을 충족시키기 위해 미들웨어의 오버헤드를 제거하고 GPU의 병렬 처리 능력을 극대화하는 직접 훈련 파이프라인을 구축했습니다. 이 섹션에서는 해당 아키텍처가 어떻게 최고 성능을 달성하는지 그 진화 과정과 기술적 핵심을 분석합니다.


NVIDIA의 로보틱스 강화학습 도구는 지속적인 발전을 거쳐왔으며, 현재의 Isaac Lab은 이러한 진화의 정점에 있습니다. 초기에는 독립형 `Isaac Gym` 프리뷰 버전이 공개되어, 렌더링 기능 없이 순수 물리 시뮬레이션에 집중하여 GPU 가속 기반의 강화학습 개념을 증명했습니다.6 이는 수만 개의 환경을 병렬로 시뮬레이션하며 기존 CPU 기반 시뮬레이터 대비 수백 배 빠른 훈련 속도를 보여주었지만, 고품질 렌더링이나 ROS와 같은 외부 시스템과의 연동은 지원하지 않는 한계가 있었습니다.8

이후 Isaac Sim 내에서 강화학습을 지원하기 위해 `OmniIsaacGymEnvs` (OIGE)와 같은 저장소가 등장했지만, 이는 Isaac Gym의 순수 성능과 Isaac Sim의 풍부한 기능 사이에서 생태계가 다소 파편화되는 결과를 낳았습니다.9 개발자들은 어떤 프레임워크를 선택해야 할지 혼란을 겪었으며, 이는 커뮤니티 포럼에서도 자주 언급되는 문제였습니다.10

이러한 문제점을 해결하기 위해 NVIDIA는 **Isaac Lab**을 공식적이고 통합된 오픈소스 프레임워크로 발표하며, 기존의 Isaac Gym, OIGE, Orbit 등을 모두 대체하였습니다.1 Isaac Lab은 Isaac Gym의 초고속 물리 시뮬레이션 능력과 Isaac Sim의 사실적인 렌더링 및 다양한 로봇 지원 기능을 하나로 통합한 플랫폼입니다.1 이를 통해 개발자들은 UR10, Franka, Unitree A1/H1 등 다양한 로봇과 사전 구성된 환경을 즉시 활용할 수 있는 "배터리 포함(batteries-included)" 경험을 얻게 되었습니다.1

이러한 전략적 통합은 단순한 브랜드 변경을 넘어선 아키텍처적 진화로 평가할 수 있습니다. 이는 순수 물리 시뮬레이터의 속도와 완전한 기능을 갖춘 로보틱스 애플리케이션의 현실성 사이의 기술적 긴장을 해소하고, 파편화되었던 개발 경로를 하나로 통일하여 연구 및 개발의 효율성을 극대화하려는 명확한 의도를 보여줍니다. 개발자는 더 이상 여러 도구 사이에서 고민할 필요 없이, Isaac Lab이라는 단일 진입점을 통해 고성능 훈련과 고품질 시뮬레이션을 동시에 달성할 수 있게 되었습니다.


Isaac Lab의 경이로운 성능의 핵심은 **엔드-투-엔드 GPU 파이프라인(end-to-end GPU pipeline)**에 있습니다. 이는 훈련 루프의 모든 주요 구성 요소, 즉 물리 시뮬레이션(NVIDIA PhysX)과 정책 학습(PyTorch 기반의 RSL-RL, rl_games 등)이 모두 GPU 상에서 실행됨을 의미합니다.6

이 아키텍처의 근간에는 **벡터화된 환경(vectorized environments)**이라는 개념이 있습니다. 시스템은 `num_envs` 파라미터를 통해 수천, 수만 개의 독립적인 시뮬레이션 환경을 병렬로 생성합니다.9 각 환경에서 발생하는 모든 데이터-관측(observations), 행동(actions), 보상(rewards), 상태(states)-는 GPU 메모리 상의 거대하고 연속적인 텐서(tensor)에 저장됩니다.

Isaac Lab은 **텐서 API(Tensor API)**를 통해 이 구조를 지원합니다. 이 API는 GPU 메모리에 있는 원시 데이터 버퍼를 PyTorch 텐서 객체로 "감싸서(wrap)" 파이썬 코드에서 직접 접근할 수 있게 해줍니다. 이 과정에서 메모리 복사(copy) 오버헤드가 전혀 발생하지 않습니다.7 훈련 스크립트 실행 시 

`pipeline=gpu` 인자를 설정하면, 관측 데이터가 생성되고 행동이 적용되는 전체 훈련 루프 동안 데이터가 단 한 번도 GPU 메모리를 벗어나지 않습니다.9 이는 매 스텝마다 데이터를 CPU 메모리로 복사하는 `pipeline=cpu` 옵션과 극명한 대조를 이룹니다.9

이 데이터 흐름은 다음과 같이 개념적으로 표현할 수 있습니다:

Isaac Sim (PhysX) -->> GPU Memory (Observation Tensor) -->> RL Algorithm (PyTorch) -->> GPU Memory (Action Tensor) -->> Isaac Sim (PhysX)

Isaac Lab은 환경 설계를 위해 두 가지 워크플로우를 제공합니다: `ManagerBasedEnv`와 `DirectRLEnv`입니다. `ManagerBasedEnv`는 씬, 관측, 행동, 이벤트 등을 각각의 '매니저'로 분리하여 모듈식으로 구성하는 설정 기반 방식입니다.19 반면 `DirectRLEnv`는 보상 및 관측 함수를 스크립트 내에 직접 구현하여 복잡한 로직에 대한 더 세밀한 제어를 제공합니다.21 중요한 점은 두 워크플로우 모두 벡터화된 텐서 데이터를 기반으로 동일한 GPU 네이티브 원칙 위에서 동작한다는 것입니다.18

이러한 직접 파이프라인의 성능은 단순한 최적화를 넘어선 근본적인 패러다임의 전환입니다. 이는 '환경'의 개념을 순차적으로 실행되는 단일 프로세스에서, 텐서에 대해 동작하는 대규모 병렬 커널로 재정의합니다. 전통적인 RL 훈련의 주된 병목이었던 CPU(시뮬레이션 로직)와 GPU(신경망 정책) 간의 데이터 전송을 완전히 제거함으로써, 기존 CPU 기반 시뮬레이터 대비 2~3 자릿수(orders of magnitude)의 성능 향상을 달성하는 것입니다.


비전 기반 강화학습은 이미지라는 고차원 데이터를 입력으로 사용하기 때문에 본질적으로 데이터 집약적입니다. 수천 개의 병렬 환경에서 각각 카메라 뷰를 렌더링하는 작업은, 아무리 빠른 시뮬레이터라 할지라도 새로운 병목이 될 수 있습니다.

이 문제를 해결하기 위해 Isaac Lab은 **타일드 렌더링(Tiled Rendering)**이라는 핵심 기술을 도입했습니다.1 이 기능은 여러 환경에 분산된 다수의 카메라로부터의 출력을 GPU 상의 단일하고 거대한 렌더 타겟(render target)으로 통합합니다. 즉, 4096개의 환경에서 각각 카메라 이미지를 개별적으로 렌더링하는 대신, 이 이미지들을 하나의 큰 타일 맵처럼 취급하여 한 번의 효율적인 렌더링 패스로 처리합니다.

가장 중요한 점은 이렇게 렌더링된 결과물(RGB, 깊이, 세분화 정보 등)이 중간 저장 과정 없이 **직접적으로 강화학습 정책이 소비하는 관측 텐서에 기록된다**는 것입니다.22 이는 렌더링된 이미지를 메모리에 저장하고 다시 불러오는 전통적인 방식을 완전히 생략하여, 막대한 I/O 오버헤드를 제거합니다. Isaac Lab은 RGB, 깊이(Depth), 법선(Normals), 의미론적 분할(Semantic Segmentation) 등 다양한 데이터 유형을 위한 여러 어노테이터(annotators)를 지원하여 이 과정을 더욱 유연하게 만듭니다.1

타일드 렌더링은 대규모 비전 기반 강화학습을 현실적으로 가능하게 만드는 결정적인 기술입니다. 만약 이 기술이 없다면, 렌더링 과정 자체가 GPU 기반 물리 시뮬레이션의 성능 이점을 모두 상쇄시켜 버릴 것입니다. 이는 물리 시뮬레이션 데이터를 위한 텐서 API와 동일한 아키텍처적 원리를 시각 데이터 영역에 적용한 것으로, 렌더링을 단순히 관측 텐서의 일부를 채우는 또 다른 병렬 연산으로 취급합니다. 이로써 수천 프레임의 시각 데이터를 초당 처리하는 고처리량 비전 기반 강화학습이 가능해집니다.


Direct API 파이프라인의 성능은 정량적 데이터로 명확히 입증됩니다. 예를 들어, RSL-RL 라이브러리를 사용할 경우 `Isaac-Humanoid-v0` 태스크는 단일 RTX 6000 GPU에서 4096개의 병렬 환경을 사용하여 단 199초 만에 훈련을 완료할 수 있습니다.23 Ant나 Humanoid와 같은 고전적인 환경들은 단 몇 분, 심지어 몇 초 안에 걷는 법을 학습할 수 있습니다.7

강화학습에서 가장 중요한 성능 지표는 초당 프레임 수(Frames Per Second, FPS)이며, 이는 정책 수렴 속도와 직결됩니다. Isaac Lab의 전신인 Orbit은 RSL-RL이나 RL-Games와 같은 최적화된 라이브러리와 함께 사용할 때 최대 100,000 FPS에 달하는 처리량을 달성할 수 있었습니다.24

이러한 최고 성능을 달성하기 위해서는 훈련 스크립트를 실행할 때 `--headless` 플래그를 사용하는 것이 매우 중요합니다.9 이 플래그는 시뮬레이션 화면을 실시간으로 보여주는 GUI 뷰포트를 비활성화하여 렌더링 오버헤드를 제거합니다. 뷰포트가 활성화된 상태에서는 훈련 속도가 현저히 저하되므로, 순수 훈련 목적일 경우 반드시 headless 모드를 사용해야 합니다.


Direct API가 강화학습 훈련의 최고 성능을 위한 길이라면, ROS 2 통합 파이프라인은 로봇을 더 넓은 소프트웨어 생태계에 연결하고 실제 환경에 배포하기 위한 길입니다. 이 섹션에서는 ROS 2 연동 아키텍처를 분석하고, 이것이 왜 순수 훈련보다는 통합 및 테스트에 더 적합한지를 설명합니다.


`isaacsim.ros2.bridge`는 Isaac Sim과 ROS 2 네트워크 간의 양방향 통신을 가능하게 하는 공식 익스텐션입니다.2 이 브리지의 주된 목적은 Isaac Sim이 표준 ROS 2 시스템 내에서 물리적 로봇의 고충실도 "디지털 트윈(digital twin)" 역할을 수행하도록 하는 것입니다.

이 브리지는 **OmniGraph (OG) 노드**를 통해 작동합니다. OG 노드는 Isaac Sim 내부의 시뮬레이션 기능들을 ROS 2의 개념(토픽, 서비스 등)에 매핑하는 시각적 스크립팅 요소입니다.27 예를 들어, 

`ROS2 Publish Clock` OG 노드는 시뮬레이션 시간을 ROS 2의 `/clock` 토픽으로 발행합니다. 이는 실제 시간과 다르게 흘러갈 수 있는 시뮬레이션 환경과 외부 ROS 2 노드들을 동기화하는 데 필수적인 기능입니다.30

ROS 2 브리지는 다양한 표준 ROS 2 메시지 타입을 지원하여 폭넓은 호환성을 제공합니다. 지원되는 주요 메시지는 다음과 같습니다:

- `sensor_msgs/Image`, `sensor_msgs/CameraInfo`: 카메라 데이터 전송 2
- `sensor_msgs/PointCloud2`: LiDAR 데이터 전송 30
- `sensor_msgs/Imu`: 관성 측정 장치(IMU) 데이터 전송 29
- `sensor_msgs/JointState`: 로봇 관절 상태 발행 및 구독 29
- `geometry_msgs/Twist`: 로봇 제어를 위한 속도 명령어 수신 30
- `tf2_msgs/TFMessage`: 좌표계 변환 정보 발행 30

또한, Isaac Sim을 실행하기 전에 ROS 2 워크스페이스를 소싱(sourcing)하면 사용자가 정의한 커스텀 메시지 타입도 지원하여 확장성을 보장합니다.27


ROS 2 브리지를 사용하여 비전 기반 강화학습 루프를 구성할 경우, 데이터는 복잡하고 여러 단계를 거치는 경로를 따르게 됩니다. 이 과정은 다음과 같이 추적할 수 있습니다.

1. Isaac Sim의 RTX 렌더러가 이미지를 생성합니다.
2. OG 노드가 이 렌더 버퍼를 읽습니다.
3. `ROS2 Publish Image` OG 노드가 이미지 데이터를 `sensor_msgs/Image` 메시지 형식으로 직렬화(serialization)합니다. 이 과정에서 **GPU에서 CPU로의 메모리 복사 및 데이터 변환**이 발생합니다.
4. ROS 2 미들웨어(DDS)가 직렬화된 메시지를 네트워크(로컬호스트 포함)를 통해 전송합니다.
5. 외부 ROS 2 노드(강화학습 에이전트)가 해당 토픽을 구독하고, 메시지를 수신하여 역직렬화(deserialization)한 후 자신의 메모리로 복사합니다.
6. 에이전트가 신경망 추론을 수행합니다. 이 때 정책이 GPU에서 실행된다면, 수신한 데이터를 다시 **CPU에서 GPU로 복사**해야 합니다.
7. 에이전트가 행동(예: `sensor_msgs/JointState`)을 결정하고 토픽에 발행합니다.
8. 데이터는 다시 미들웨어를 통해 Isaac Sim의 `ROS2 Subscribe Joint States` 노드로 전달되고, 역직렬화 과정을 거쳐 시뮬레이션에 적용됩니다.

이러한 다단계 프로세스는 여러 번의 메모리 복사, 직렬화/역직렬화 단계를 포함하므로 상당한 지연 시간(latency)을 유발합니다. 한 커뮤니티 포럼 사용자의 측정 결과에 따르면, Isaac Sim과 ROS 2 간의 메시지 왕복 지연 시간은 약 **16.7ms**로 일관되게 나타났습니다.32 이 지연 시간은 시뮬레이션의 물리/렌더링 속도(예: 60Hz = 16.67ms/프레임)와 직접적으로 연관되어 있으며, 브리지는 각 시뮬레이션 스텝(

`OnPlaybackTick` 또는 `OnPhysicsStep`)마다 호출되기 때문입니다.29

처리량(throughput) 측면에서도 브리지는 병목이 될 수 있습니다. 특히 대용량 데이터를 전송할 때 그 영향은 더욱 커집니다. 한 사용자는 대용량 포인트 클라우드 데이터를 ROS 2 브리지를 통해 발행하자 시뮬레이션 FPS가 약 20Hz에서 0.5Hz로 급격히 떨어졌다고 보고했습니다.33 이는 통신 자체가 연산 시간을 압도할 수 있음을 보여주는 명백한 증거입니다.

결론적으로, ROS 2 브리지는 데이터 교환 방식을 근본적으로 변화시킵니다. Direct API의 제로 카피(zero-copy) 프로세스 내부 메모리 접근 방식에서, 프로세스 간 통신(Inter-Process Communication, IPC) 문제로 전환시키는 것입니다. 이러한 구조적 변화는 데이터 직렬화, 메모리 복사, 미들웨어 전송으로 인한 오버헤드를 필연적으로 수반하며, 이는 Direct API 대비 수십, 수백 배 느린 성능을 초래합니다. 따라서 이 아키텍처는 고처리량이 요구되는 강화학습 훈련에는 부적합합니다.


원시(raw) 이미지 데이터는 크기가 매우 큽니다. 예를 들어, 1080p 해상도의 30fps 비디오 스트림은 초당 약 177MB의 데이터를 생성합니다.34 이 데이터를 표준 ROS 2 브리지를 통해 그대로 전송하는 것은 매우 비효율적입니다. NVIDIA는 이 문제를 해결하기 위한 두 가지 핵심 솔루션을 제공합니다.

솔루션 1: 하드웨어 가속 이미지 압축

isaac_ros_compression 메타패키지는 NVIDIA GPU 및 Jetson 플랫폼의 전용 하드웨어 엔진인 NVENC를 활용하는 ROS 2 노드(isaac_ros_h264_encoder)를 제공합니다.34 이 노드는 이미지를 H.264 비디오 스트림으로 압축하여 데이터 크기를 약 10배까지 줄일 수 있습니다. 이는 JPEG 표준을 사용하는 CPU 기반의 표준 

`image_transport_plugins`보다 훨씬 효율적이며, CPU의 부하를 줄여줍니다.34

솔루션 2: NVIDIA Isaac ROS (NITROS)

NITROS는 고성능 ROS 2 인식을 위한 가장 중요한 기술입니다. 이는 ROS 2 Humble 버전부터 도입된 타입 적응(Type Adaptation) 및 협상(Negotiation) 기능의 NVIDIA 구현체입니다.37

- **작동 방식:** NITROS를 지원하는 두 노드가 연결되면, 이들은 통신을 위한 최적의 "NITROS 타입"을 서로 협상할 수 있습니다. 이를 통해 전체 데이터를 CPU로 복사하여 전송하는 대신, GPU 메모리 버퍼를 가리키는 포인터를 전달할 수 있습니다. 이는 동일 프로세스 내에서 실행되는 ROS 2 노드 간에 **제로 카피(zero-copy)** 데이터 전송을 가능하게 합니다.38
- **영향:** NITROS는 인식 파이프라인의 성능을 극적으로 향상시킵니다. 예를 들어, `isaac_ros_image_pipeline`의 전체 스테레오 깊이 추정 그래프는 RTX 4090에서 초당 692 프레임(FPS)으로 실행될 수 있으며, 이는 표준 ROS 방식으로는 달성하기 어려운 속도입니다.41

표준 ROS 2 브리지는 비전 데이터 전송에 있어 느리지만, Isaac Sim, ROS 2 브리지, 그리고 Isaac ROS NITROS 패키지를 조합하면 **추론(inference)** 및 **소프트웨어 인 더 루프(Software-in-the-Loop, SIL) 테스트**를 위한 매우 강력하고 실행 가능한 고성능 파이프라인을 구축할 수 있습니다. 이 아키텍처는 실제 로봇의 전체 소프트웨어 스택을 시뮬레이션 내에서 테스트하는 데 이상적입니다.27 하지만, 수천 개의 병렬 환경과 에이전트-환경 루프의 최소 지연 시간이 요구되는 강화학습 

**훈련**의 고유한 요구사항을 고려할 때, 이 최적화된 브리지조차도 Direct API의 제로 오버헤드, 프로세스 내 통신 방식보다 느릴 수밖에 없습니다. 즉, Direct API는 **훈련**을 위해, 브리지+NITROS는 ROS 2 시스템의 **통합 및 테스트**를 위해 설계된 각기 다른 목적의 아키텍처입니다.


이전 섹션들의 분석을 종합하여, Isaac Sim에서 비전 기반 강화학습을 위한 두 가지 주요 아키텍처, 즉 Direct API와 ROS 2 통합 파이프라인을 직접적으로 비교하고, 각 선택에 따르는 트레이드오프를 명확히 제시합니다.


**강화학습 훈련 속도 (FPS):** 이 지표에서 Direct API 파이프라인은 압도적인 우위를 점합니다. GPU 네이티브 데이터 흐름과 벡터화된 환경 덕분에 10,000에서 100,000 FPS를 초과하는 처리량을 달성할 수 있으며, 이는 오직 GPU의 연산 능력에 의해서만 제한됩니다.7 반면, 표준 ROS 2 브리지는 IPC, 데이터 직렬화, 그리고 반복적인 CPU-GPU 메모리 복사로 인해 FPS가 수십에서 수백 수준으로 급격히 떨어져, 대규모 강화학습 훈련에는 사실상 부적합합니다.33 NITROS를 사용하면 이 병목이 일부 완화되지만, 여전히 브리지 자체의 오버헤드가 남아있어 Direct API의 성능에는 미치지 못합니다.

**통신 지연 시간:** Direct API는 GPU 내에서 제로 카피 텐서 교환을 수행하므로 지연 시간은 1ms 미만으로 거의 무시할 수 있는 수준입니다.7 반면 ROS 2 브리지는 시뮬레이션 스텝 시간(예: 60Hz에서 16.7ms)과 IPC 오버헤드가 더해져 수십 ms의 왕복 지연 시간을 보입니다.32 이 지연 시간은 고주파 제어가 필요한 로봇의 안정성에 치명적일 수 있습니다.

**Sim-to-Real 용이성:** 이 측면에서는 ROS 2 통합 파이프라인이 명백한 장점을 가집니다. 시뮬레이션에서 사용된 것과 동일한 ROS 2 노드(코드)를 거의 수정 없이 실제 로봇에 배포할 수 있어 코드 재사용성이 매우 높습니다.5 반면 Direct API로 훈련된 정책은 별도의 '내보내기' 단계를 거쳐야 하며, 훈련 코드와 배포 코드가 분리됩니다.4

**기존 ROS 스택과의 통합:** ROS 2 브리지의 가장 큰 강점은 Nav2, MoveIt, RViz 등 기존의 방대한 ROS 2 생태계와 원활하게 통합된다는 점입니다.27 이는 로봇의 전체 시스템을 테스트하는 데 필수적입니다. Direct API는 설계상 이러한 통합을 고려하지 않으며, ROS와의 연동을 위해서는 하이브리드 워크플로우가 필요합니다.

**개발 복잡성:** Direct API는 Isaac Lab의 API와 환경 설계 개념(매니저 등)을 학습해야 하는 부담이 있습니다.19 그러나 ROS 2 통합 파이프라인은 Isaac Sim과 ROS 2라는 두 개의 복잡한 비동기 시스템 간의 상호작용을 디버깅해야 하므로 훨씬 더 높은 복잡성을 가질 수 있습니다.29 특히 NITROS를 사용할 경우, 타입 협상 및 그래프 구성에 대한 추가적인 이해가 필요합니다.38


아래 표는 본 보고서의 상세 분석을 요약하여, 사용자가 자신의 프로젝트 우선순위에 따라 최적의 아키텍처를 선택할 수 있도록 돕는 의사결정 도구입니다. 각 항목의 평가는 보고서 본문에 제시된 증거에 기반합니다.

| 기준                         | Direct API (Isaac Lab)                                       | ROS 2 브리지 (표준)                                          | ROS 2 브리지 (NITROS 포함)                                   |
| ---------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **강화학습 훈련 속도 (FPS)** | **매우 높음** (10k-100k+ FPS). GPU 연산에 의해서만 제한됨. 훈련을 위한 결정적 선택. 7 | **매우 낮음**. IPC, 직렬화, CPU-GPU 복사로 인해 제한됨. 본격적인 RL 훈련에 부적합. 33 | **중간**. 표준 브리지보다 빠르지만 브리지 오버헤드가 여전히 존재. 고처리량 훈련보다는 테스트에 적합. |
| **통신 지연 시간**           | **극도로 낮음** (<1ms). GPU 내 제로 카피 텐서 교환. 7        | **중간-높음** (~16.7ms+ 왕복). 시뮬레이션 스텝 시간과 IPC 오버헤드가 지배적. 32 | **낮음-중간**. ROS 그래프 내 지연은 감소하지만, 초기 브리지 통과 오버헤드는 남음. |
| **Sim-to-Real 용이성**       | **간접적**. 별도의 "정책 내보내기" 단계 필요. 훈련 코드와 배포 코드가 분리됨. 4 | **직접적**. 시뮬레이션에 사용된 동일한 ROS 2 노드를 실제 로봇에서 재사용 가능. 5 | **직접적**. 표준 브리지와 동일한 장점을 가지며 성능은 더 높음. 37 |
| **ROS 스택과의 통합**        | **없음 (설계상)**. 독립적인 파이썬 환경에서 작동. ROS 연결을 위해 하이브리드 워크플로우 필요. 25 | **네이티브**. Nav2, MoveIt, RViz 등 모든 ROS 2 패키지와 원활하게 통합. 이것이 주된 강점. 27 | **네이티브**. GPU 가속 인식을 ROS 2 생태계와 통합하는 최고의 방법. 37 |
| **개발 복잡성**              | **중간**. Isaac Lab API 및 환경 설계 개념(매니저 등) 학습 필요. 19 | **높음**. 두 개의 복잡한 비동기 시스템(Isaac Sim, ROS 2 그래프) 관리 및 상호작용 디버깅 필요. 29 | **높음**. 표준 브리지와 동일하며, NITROS 타입 및 그래프 구성에 대한 추가적인 복잡성 존재. 38 |
| **자원 활용도**              | **GPU에 최적화**. 전체 루프가 GPU 활용도를 극대화하고 CPU-GPU 트래픽을 최소화하도록 설계됨. 7 | **높은 CPU/메모리**. 브리지 자체, 직렬화, 데이터 복사가 CPU 집약적. 33 | **GPU/HW 엔진에 최적화**. 인식 및 압축을 전용 하드웨어로 오프로드하여 CPU/GPU 코어 부담 감소. 34 |


앞선 분석에서 확인했듯이, Direct API는 훈련 속도에서, ROS 2는 통합 및 배포에서 각각 명확한 장점을 가집니다. 따라서 두 아키텍처의 장점만을 취하는 하이브리드 접근법이 전문가들 사이에서 표준 워크플로우로 자리 잡고 있습니다. 이 섹션에서는 그 구체적인 단계를 상세히 설명합니다.


이 단계의 목표는 가능한 가장 빠르고 효율적으로 고품질의 수렴된 정책을 얻는 것입니다. 이를 위해 섹션 2에서 설명한 **Direct API 파이프라인**을 사용합니다. 수천 개의 병렬 환경에서 GPU 네이티브 데이터 흐름을 통해 훈련을 가속화합니다.

이 과정에서 **도메인 무작위화(Domain Randomization)**는 매우 중요한 역할을 합니다.1 시뮬레이션 환경의 물리 파라미터(질량, 마찰, 감쇠), 센서 속성, 조명, 텍스처 등을 훈련 중에 무작위로 변경함으로써, 정책이 다양한 변화에 강건해지도록 만듭니다. 이는 시뮬레이션과 실제 세계 간의 차이(Sim-to-Real Gap)를 극복하고, 시뮬레이션에서 훈련된 정책이 실제 로봇에서도 성공적으로 작동할 확률을 크게 높여줍니다.


훈련이 완료되면, 학습된 PyTorch 정책 모델은 배포에 용이한 표준 형식으로 변환되어야 합니다. Isaac Lab의 `play.py` 스크립트는 이 과정을 지원하며, 훈련된 체크포인트 파일을 입력으로 받아 추론 가능한 모델을 생성합니다.42

주요 내보내기 형식은 다음과 같습니다:

- **TorchScript (`.pt`, `.jit`):** PyTorch 모델을 직렬화하고 최적화하는 방법입니다. Isaac Lab 튜토리얼에서는 이 형식의 파일을 직접 로드하여 추론을 수행하는 예제를 보여줍니다.42 이는 PyTorch 생태계 내에서 간편하게 사용할 수 있는 장점이 있습니다.
- **ONNX (`.onnx`):** 머신러닝 모델을 위한 개방형 표준 형식입니다. ONNX는 가장 범용적인 선택지로, ONNX Runtime이나 NVIDIA TensorRT와 같은 다양한 추론 엔진을 통해 여러 하드웨어 플랫폼에서 실행될 수 있습니다. 다수의 Sim-to-Real 파이프라인 관련 자료에서 ONNX로의 내보내기를 핵심 단계로 강조하고 있습니다.4


마지막 단계는 물리적 로봇에서 실행될 간단한 ROS 2 노드를 작성하는 것입니다. 이 추론 노드의 역할은 명확하고 단순합니다:

1. **모델 로드:** 2단계에서 내보낸 ONNX 또는 TorchScript 모델 파일을 로드합니다.
2. **센서 데이터 구독:** 실제 로봇의 센서 토픽(예: `/camera/image_raw`, `/joint_states`)을 구독합니다.
3. **데이터 전처리:** 수신한 센서 데이터를 훈련 시 사용했던 관측(observation) 형식과 일치하도록 전처리합니다.
4. **추론 실행:** 전처리된 데이터를 모델에 입력하여 행동(action)을 출력받습니다.
5. **명령 발행:** 출력된 행동을 로봇의 제어 토픽(예: `/joint_trajectory_controller/joint_trajectory`)에 발행합니다.

이 접근 방식은 매우 효율적입니다. 추론 노드는 가볍고, 배포 시점에는 무거운 시뮬레이션 환경(Isaac Sim)이 전혀 필요하지 않습니다.45 이는 훈련과 배포라는 두 가지 다른 관심사를 깔끔하게 분리합니다. 실제로 모바일 로봇 내비게이션부터 매니퓰레이터 제어에 이르기까지, 이 하이브리드 워크플로우가 성공적으로 적용된 다수의 연구 및 커뮤니티 사례가 존재합니다.5

결론적으로, 하이브리드 워크플로우는 타협의 산물이 아닌 **최적의 설계 패턴**입니다. 이는 계산 집약적이고 오프라인 작업인 정책 **훈련**과, 자원이 제한적이고 실시간성이 요구되는 정책 **추론**을 전략적으로 분리합니다. 이를 통해 각 단계에 가장 적합한 아키텍처 환경을 사용하여, Isaac Sim(훈련용)과 ROS 2(배포용)의 강점을 모두 극대화할 수 있습니다.


Direct API를 이용한 훈련과 ROS 2를 통한 배포라는 핵심 하이브리드 워크플로우 외에도, 특정 요구사항에 맞는 고급 워크플로우와 성공적인 Sim-to-Real 전환을 위해 반드시 고려해야 할 시스템 수준의 요소들이 존재합니다.


강화학습 정책의 순수 훈련에는 ROS 2 브리지가 비효율적이지만, 시뮬레이션 중에 ROS 2를 사용하는 것이 필수적인 시나리오도 있습니다. 가장 대표적인 사례는 **전체 소프트웨어 스택의 통합 검증(Software-in-the-Loop, SIL)**입니다.27

예를 들어, 이미 훈련된 강화학습 정책이 Nav2와 같은 복잡한 기존 ROS 2 기반 내비게이션 스택과 어떻게 상호작용하는지 테스트하고 싶다고 가정해 봅시다.5 이 경우, Nav2 스택 전체를 외부 ROS 2 노드로 실행하고, 이 노드들이 ROS 2 브리지를 통해 Isaac Sim 내의 시뮬레이션 로봇과 통신하도록 구성해야 합니다. 여기서 목표는 정책을 훈련하는 것이 아니라, 

**이미 훈련된 정책**이 더 큰 시스템 내에서 올바르게 통합되고 작동하는지를 검증하는 것입니다. Isaac Sim 공식 문서에는 이러한 하이브리드 형태, 즉 강화학습 정책을 ROS 제어와 함께 실행하는 튜토리얼이 명시적으로 제공됩니다.28


Sim-to-Real의 가장 큰 난관은 시뮬레이션의 물리 모델과 실제 세계의 동역학 간의 불일치, 즉 '현실 격차(reality gap)'입니다.45 이 격차를 줄이기 위한 두 가지 핵심 기술은 시스템 식별과 도메인 무작위화입니다.

- **도메인 무작위화 (Domain Randomization, DR):** 앞서 언급했듯이, Isaac Lab은 DR을 위한 광범위한 기능을 내장하고 있습니다.1

  `isaacsim.replicator.domain_randomization` 익스텐션은 물리 파라미터(질량, 마찰, 감쇠), 액추에이터 속성(강성, 댐핑), 센서 노이즈, 시각적 요소(조명, 색상, 텍스처) 등을 훈련 중에 무작위로 변경할 수 있는 세밀한 API를 제공합니다.43 이는 정책이 특정 시뮬레이션 환경에 과적합되는 것을 방지하고, 예측하지 못한 실제 환경의 변화에 대해 강건하게 대처할 수 있도록 만듭니다.

- **시스템 식별 (System Identification):** 이는 실제 로봇의 물리적 특성을 측정하여 시뮬레이션 파라미터를 실제와 최대한 가깝게 튜닝하는 과정입니다. 예를 들어, 실제 로봇의 액추에이터 응답을 측정하고, 그 데이터에 가장 잘 맞는 시뮬레이션의 강성(stiffness) 및 감쇠(damping) 값을 찾는 것입니다. 이 과정은 수동으로 진행될 수도 있지만, 최근 연구에서는 이를 자동화하려는 시도가 이루어지고 있습니다. 한 연구 논문은 "자동화된 real-to-sim 튜닝 모듈"을 제안했으며 50, 다른 커뮤니티 게시물에서는 액추에이터 파라미터를 맞추기 위해 시스템 식별을 사용한 사례를 논의합니다.52 정확한 시스템 식별은 도메인 무작위화를 적용하기 전, 시뮬레이션의 '중심점'을 현실에 맞추는 매우 중요한 선행 단계입니다.


Isaac Sim은 ROS 2 브리지 외에도 **ZeroMQ (ZMQ) 브리지**를 고성능 커스텀 미들웨어 연결을 위한 참조 구현으로 제공합니다.14

ZMQ는 경량의 고성능 메시징 라이브러리로, ROS 2의 포괄적인 기능(노드 발견, 파라미터 서버 등) 없이 순수한 데이터 전송 속도가 필요할 때 유용합니다. Isaac Sim의 ZMQ 브리지는 메시징을 위해 ZMQ를, 데이터 직렬화를 위해 Protobuf를 사용하여 외부 애플리케이션(예: 커스텀 C++ 또는 파이썬 제어 프로그램)과 직접적이고 빠른 데이터 링크를 구축합니다.54 이는 ROS 2 프레임워크 전체의 오버헤드나 복잡성 없이 특정 SIL 또는 하드웨어 인 더 루프(Hardware-in-the-Loop, HIL) 테스트를 수행해야 하는 고급 사용자를 위한 선택지입니다. 이는 ROS 2를 대체하는 범용 솔루션이 아니라, 특정 목적을 위한 경량화된 대안으로 이해해야 합니다.


본 보고서는 NVIDIA Isaac Sim 환경에서 비전 기반 강화학습을 수행할 때 요구되는 아키텍처에 대한 심층 분석을 제공했습니다. 모든 증거와 분석을 종합하여, 사용자의 초기 질문에 대한 최종 결론과 구체적인 상황에 맞는 권장사항을 제시합니다.


**비전 기반 강화학습 정책의 \*훈련\* 단계에서 ROS 2는 필수적이지 않으며, 성능상의 이유로 권장되지 않습니다.** 상당한 통신 오버헤드를 유발하는 ROS 2 브리지와 달리, **Isaac Lab의 Direct API 파이프라인**은 GPU 네이티브 데이터 흐름을 통해 훈련 속도를 극대화하므로, 이 단계에서는 아키텍처적으로 명백히 우월한 선택입니다.

로보틱스 개발의 전체 라이프사이클을 고려할 때, 단일 아키텍처가 모든 요구사항을 만족시킬 수는 없습니다. 훈련의 효율성과 배포의 표준화라는 상충될 수 있는 목표를 모두 달성하기 위해서는, 각 단계의 목적에 맞는 최적의 도구를 사용하는 **하이브리드 워크플로우**가 가장 합리적이고 전문적인 접근 방식입니다.


사용자의 프로젝트 목표에 따라 다음과 같은 의사결정 프레임워크를 따를 것을 권장합니다.

- **주요 목표가 강화학습 연구 및 최첨단 정책을 가능한 한 빨리 개발하는 것이라면:**
  - **권장 아키텍처:** **Direct API (Isaac Lab) 파이프라인**을 독점적으로 사용하십시오.
  - **근거:** 이는 훈련 처리량(FPS)을 최대화하고 정책 수렴 시간을 최소화하는 유일한 방법입니다. ROS 2는 이 단계에서 불필요한 복잡성과 성능 저하만 초래합니다.
- **주요 목표가 훈련된 정책을 물리적 로봇에 배포하는 것이라면:**
  - **권장 아키텍처:** **하이브리드 워크플로우**를 채택하십시오.
  - **실행 순서:** (1) Direct API로 훈련하고, (2) 정책을 ONNX로 내보낸 후, (3) 경량 ROS 2 노드에서 배포하십시오.
  - **근거:** 이 방법은 훈련의 속도와 배포의 실용성 및 ROS 생태계와의 호환성을 모두 확보하는 최적의 경로입니다.
- **주요 목표가 \*이미 훈련된\* 정책을 Nav2, MoveIt과 같은 다른 ROS 2 구성 요소와 통합하여 테스트하는 것이라면:**
  - **권장 아키텍처:** **ROS 2 브리지와 NITROS 가속 패키지**를 사용하십시오.
  - **근거:** 이는 전체 소프트웨어 스택을 시뮬레이션 내에서 검증(SIL)하는 가장 정확하고 효율적인 방법입니다. NITROS는 비전 데이터 파이프라인의 병목을 완화하여 고충실도 테스트를 가능하게 합니다.
- **주요 목표가 ROS가 아닌 커스텀 애플리케이션과 고성능으로 연동하는 것이라면:**
  - **권장 아키텍처:** **ZMQ 브리지**를 고급 대안으로 고려하십시오.
  - **근거:** 이는 ROS 2의 복잡성 없이 두 시스템 간의 직접적이고 빠른 데이터 링크가 필요할 때 유용한 경량 솔루션입니다.


NVIDIA의 개발 방향은 명확합니다. Isaac Lab은 로봇 학습을 위한 핵심 프레임워크로 지속적으로 발전할 것이며, ROS 2와의 통합은 배포 및 시스템 검증을 위해 더욱 강화될 것입니다. 특히 최근 발표된 표준화된 ROS 2 시뮬레이션 인터페이스와 같은 노력은 55 Isaac Sim과 다른 시뮬레이터들이 ROS 2 생태계 내에서 더욱 일관된 방식으로 작동하도록 만들 것입니다.

앞으로도 훈련 환경과 배포 환경을 분리하여 각 단계의 성능과 호환성을 극대화하는 하이브리드 접근법은 로보틱스 분야에서 AI 기반 제어기를 개발하는 데 있어 가장 효과적인 모범 사례로 남을 것입니다. 기술이 발전함에 따라 두 환경 간의 경계는 더욱 흐려지고 전환은 더 매끄러워지겠지만, 각 작업에 최적화된 도구를 전략적으로 선택하는 근본적인 아키텍처 원칙은 변치 않을 것입니다.


1. Welcome to Isaac Lab!, accessed July 2, 2025, https://isaac-sim.github.io/IsaacLab/
2. Isaac Sim integration with ROS 2 - Blog - Marvik, accessed July 2, 2025, https://blog.marvik.ai/2024/12/17/isaac-sim-integration-with-ros-2/
3. Building the future of robotics with open-source and ROS 2, accessed July 2, 2025, https://pal-robotics.com/blog/building-the-future-of-robotics-with-open-source-and-ros-2/
4. Isaac Gym to Onnx - Tyler Barkin, accessed July 2, 2025, https://www.tylerbarkin.com/isaac-gym-to-onnx
5. (PDF) Sim-to-Real Transfer for Mobile Robots with Reinforcement Learning: from NVIDIA Isaac Sim to Gazebo and Real ROS 2 Robots - ResearchGate, accessed July 2, 2025, https://www.researchgate.net/publication/387767755_Sim-to-Real_Transfer_for_Mobile_Robots_with_Reinforcement_Learning_from_NVIDIA_Isaac_Sim_to_Gazebo_and_Real_ROS_2_Robots
6. Isaac Gym - Preview Release - NVIDIA Developer, accessed July 2, 2025, https://developer.nvidia.com/isaac-gym
7. Isaac Gym: High Performance GPU-Based Physics Simulation For Robot Learning - ar5iv, accessed July 2, 2025, https://ar5iv.labs.arxiv.org/html/2108.10470
8. Isaac Lab Ecosystem, accessed July 2, 2025, https://isaac-sim.github.io/IsaacLab/main/source/setup/ecosystem.html
9. isaac-sim/OmniIsaacGymEnvs: Reinforcement Learning Environments for Omniverse Isaac Gym - GitHub, accessed July 2, 2025, https://github.com/isaac-sim/OmniIsaacGymEnvs
10. How to code reinforcement learning for mobile robots - NVIDIA Developer Forums, accessed July 2, 2025, https://forums.developer.nvidia.com/t/how-to-code-reinforcement-learning-for-mobile-robots/272917
11. Reinforcement Learning in Isaac Sim: Which Framework? - NVIDIA Developer Forums, accessed July 2, 2025, https://forums.developer.nvidia.com/t/reinforcement-learning-in-isaac-sim-which-framework/289204
12. Isaac Lab - Isaac Sim Documentation, accessed July 2, 2025, https://docs.isaacsim.omniverse.nvidia.com/4.5.0/isaac_lab_tutorials/index.html
13. Deep-Dive Into Isaac Lab Workflows | Isaac Lab Office Hours - YouTube, accessed July 2, 2025, https://www.youtube.com/watch?v=_UHxP0FbOws
14. NVIDIA Isaac Sim - GitHub, accessed July 2, 2025, https://github.com/isaac-sim
15. NVIDIA Isaac Lab, accessed July 2, 2025, https://developer.nvidia.com/isaac/lab
16. Unified framework for robot learning built on NVIDIA Isaac Sim - GitHub, accessed July 2, 2025, https://github.com/isaac-sim/IsaacLab
17. isaac-sim/IsaacGymEnvs: Isaac Gym Reinforcement Learning Environments - GitHub, accessed July 2, 2025, https://github.com/isaac-sim/IsaacGymEnvs
18. Environment Design - Isaac Lab Documentation, accessed July 2, 2025, https://isaac-sim.github.io/IsaacLab/main/source/setup/walkthrough/technical_env_design.html
19. isaaclab.envs - Isaac Lab Documentation, accessed July 2, 2025, https://isaac-sim.github.io/IsaacLab/main/source/api/lab/isaaclab.envs.html
20. docs/source/refs/reference_architecture / main / Yichen Cai / IsaacLab - GitLab, accessed July 2, 2025, https://git.ias.informatik.tu-darmstadt.de/cai/IsaacLab/-/tree/main/docs/source/refs/reference_architecture?ref_type=heads
21. Creating a Direct Workflow RL Environment - Isaac Lab Documentation, accessed July 2, 2025, https://isaac-sim.github.io/IsaacLab/main/source/tutorials/03_envs/create_direct_rl_env.html
22. Fast-Track Robot Learning in Simulation Using NVIDIA Isaac Lab | NVIDIA Technical Blog, accessed July 2, 2025, https://developer.nvidia.com/blog/fast-track-robot-learning-in-simulation-using-nvidia-isaac-lab/
23. Reinforcement Learning Library Comparison - Isaac Lab Documentation, accessed July 2, 2025, https://isaac-sim.github.io/IsaacLab/main/source/overview/reinforcement-learning/rl_frameworks.html
24. ORBIT: A Unified Simulation Framework for Interactive Robot Learning Environments, accessed July 2, 2025, https://isaac-orbit.github.io/
25. Reinforcement Learning Wrappers - Isaac Lab Documentation, accessed July 2, 2025, https://docs.robotsfan.com/isaaclab_official/v1.3.0/source/overview/reinforcement-learning/rl_existing_scripts.html
26. Installation using Isaac Sim pip - Isaac Lab Documentation, accessed July 2, 2025, https://isaac-sim.github.io/IsaacLab/main/source/setup/installation/pip_installation.html
27. A Beginner's Guide to Simulating and Testing Robots with ROS 2 and NVIDIA Isaac Sim, accessed July 2, 2025, https://developer.nvidia.com/blog/a-beginners-guide-to-simulating-and-testing-robots-with-ros-2-and-nvidia-isaac-sim/
28. What Is Isaac Sim? - NVIDIA Omniverse, accessed July 2, 2025, https://docs.omniverse.nvidia.com/isaacsim
29. Running a Reinforcement Learning Policy through ROS2 and Isaac Sim, accessed July 2, 2025, https://docs.isaacsim.omniverse.nvidia.com/latest/ros2_tutorials/tutorial_ros2_rl_controller.html
30. ROS 2 Bridge in Standalone Workflow - Isaac Sim Documentation, accessed July 2, 2025, https://docs.isaacsim.omniverse.nvidia.com/4.5.0/ros2_tutorials/tutorial_ros2_python.html
31. Using ZED with ROS 2 and Isaac Sim - Stereolabs, accessed July 2, 2025, https://www.stereolabs.com/docs/isaac-sim/ros2_integration
32. Measuing lag between Isaac sim and ros2 - NVIDIA Developer Forums, accessed July 2, 2025, https://forums.developer.nvidia.com/t/measuing-lag-between-isaac-sim-and-ros2/294562
33. [Question] Will ROS affect the overall program performance? Especially when subscribing a topic / Issue #675 / isaac-sim/IsaacLab - GitHub, accessed July 2, 2025, https://github.com/isaac-sim/IsaacLab/issues/675
34. Isaac ROS Compression - isaac_ros_docs documentation, accessed July 2, 2025, https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_compression/index.html
35. NVIDIA-ISAAC-ROS/isaac_ros_compression: NVIDIA-accelerated data compression, accessed July 2, 2025, https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_compression
36. Image_transport_plugins vs hardware accelerated H.264 de|compression - ROS Discourse, accessed July 2, 2025, https://discourse.ros.org/t/image-transport-plugins-vs-hardware-accelerated-h-264-de-compression/28052
37. Isaac ROS (Robot Operating System) - NVIDIA Developer, accessed July 2, 2025, https://developer.nvidia.com/isaac/ros
38. NITROS - isaac_ros_docs documentation - NVIDIA Isaac ROS, accessed July 2, 2025, https://nvidia-isaac-ros.github.io/concepts/nitros/index.html
39. Boosting Custom ROS Graphs Using NVIDIA Isaac Transport for ROS, accessed July 2, 2025, https://developer.nvidia.com/blog/boosting-custom-ros-graphs-using-nvidia-isaac-transport-for-ros/
40. Boosting Custom ROS Graphs Using NVIDIA Isaac Transport for ROS - daily.dev, accessed July 2, 2025, https://app.daily.dev/posts/4wfalAKtV
41. NVIDIA-ISAAC-ROS/isaac_ros_image_pipeline: NVIDIA-accelerated ROS 2 packages for camera image processing. - GitHub, accessed July 2, 2025, https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_image_pipeline
42. Policy Inference in USD Environment - Isaac Lab Documentation, accessed July 2, 2025, https://isaac-sim.github.io/IsaacLab/main/source/tutorials/03_envs/policy_inference_in_usd.html
43. Beginner - Isaac Sim - NVIDIA Developer Forums, accessed July 2, 2025, https://forums.developer.nvidia.com/t/beginner/268835
44. Deploying Policies in Isaac Sim, accessed July 2, 2025, https://docs.isaacsim.omniverse.nvidia.com/4.5.0/isaac_lab_tutorials/tutorial_policy_deployment.html
45. Sim2Real RL Pipeline for Kinova Gen3 – Isaac Lab + ROS 2 Deployment - Reddit, accessed July 2, 2025, https://www.reddit.com/r/robotics/comments/1kgoby8/sim2real_rl_pipeline_for_kinova_gen3_isaac_lab/
46. Toolbox for Real World Deployment using ROS2 (sim2real) / isaac-sim IsaacLab / Discussion #2412 - GitHub, accessed July 2, 2025, https://github.com/isaac-sim/IsaacLab/discussions/2412
47. Sim-to-Real Transfer for Mobile Robots with Reinforcement Learning: from NVIDIA Isaac Sim to Gazebo and Real ROS 2 Robots - arXiv, accessed July 2, 2025, [https://arxiv.org/pdf/2501.02902?](https://arxiv.org/pdf/2501.02902)
48. [2501.02902] Sim-to-Real Transfer for Mobile Robots with Reinforcement Learning: from NVIDIA Isaac Sim to Gazebo and Real ROS 2 Robots - arXiv, accessed July 2, 2025, https://arxiv.org/abs/2501.02902
49. Isaac Sim Setup - isaac_ros_docs documentation, accessed July 2, 2025, https://nvidia-isaac-ros.github.io/getting_started/isaac_sim/index.html
50. arxiv.org, accessed July 2, 2025, https://arxiv.org/html/2502.20396v1
51. [isaacsim.replicator.domain_randomization] Isaac Sim Replicator ..., accessed July 2, 2025, https://docs.isaacsim.omniverse.nvidia.com/4.5.0/py/source/extensions/isaacsim.replicator.domain_randomization/docs/index.html
52. Building a data acquisition pipeline via teleoperation with NVIDIA Isaac Sim, accessed July 2, 2025, https://x-humanoid.com/news-view-159.html
53. Isaac Sim and Isaac Lab Are Now Available for Early Developer Preview | NVIDIA Technical Blog, accessed July 2, 2025, https://developer.nvidia.com/blog/isaac-sim-and-isaac-lab-are-now-available-for-early-developer-preview/
54. isaac-sim/IsaacSimZMQ: Isaac SIM extension for ... - GitHub, accessed July 2, 2025, https://github.com/isaac-sim/IsaacSimZMQ
55. Advanced Sensor Physics, Customization, and Model Benchmarking Coming to NVIDIA Isaac Sim and NVIDIA Isaac Lab | NVIDIA Technical Blog - NVIDIA Developer, accessed July 2, 2025, https://developer.nvidia.com/blog/advanced-sensor-physics-customization-and-model-benchmarking-coming-to-nvidia-isaac-sim-and-nvidia-isaac-lab/