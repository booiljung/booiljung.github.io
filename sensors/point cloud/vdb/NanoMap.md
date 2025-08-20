# NanoMap GPU 가속 3D 매핑

## 1.  실시간 3D 환경 인식의 도전과제

자율 로봇이 복잡하고 예측 불가능한 환경에서 지능적으로 임무를 수행하기 위한 가장 근본적인 전제 조건은 주변 세계를 정확하게 인식하고 이해하는 능력이다. 이러한 능력의 핵심에는 3차원(3D) 매핑 기술이 자리 잡고 있다. 3D 맵은 단순히 환경의 기하학적 모델을 구축하는 것을 넘어, 로봇이 자신의 위치를 추정(Localization)하고, 안전한 경로를 계획(Path Planning)하며, 주변 객체와 상호작용(Interaction)하기 위한 필수적인 정보의 원천으로 기능한다.1 신뢰할 수 있고 최신 정보가 반영된 맵은 로봇의 상황 인식(Situational Awareness)의 초석이며, 이 맵의 품질과 실시간성은 로봇의 자율성과 안전성에 직접적인 영향을 미친다.2

최근 수십 년간 센서 기술은 비약적인 발전을 거듭했다. 고해상도 RGB-D 카메라, 다채널 LiDAR(Light Detection and Ranging)와 같은 센서들은 초당 수십만에서 수백만 개의 데이터 포인트를 생성하며, 로봇에게 전례 없이 풍부하고 정밀한 환경 정보를 제공한다. 그러나 이러한 데이터의 홍수(Data Deluge)는 역설적으로 새로운 도전과제를 낳았다. 방대한 양의 센서 데이터를 실시간으로 처리하여 3D 맵을 생성하고 지속적으로 갱신하는 것은 엄청난 계산 부하를 유발하며, 이는 로봇 시스템의 심각한 병목 현상으로 작용한다.4 특히, 드론이나 소형 지상 로봇과 같이 계산 자원과 전력 소모에 제약이 큰 플랫폼에서는 이 문제가 더욱 심화된다.4

3D 매핑의 역사에서 점유 격자 지도(Occupancy Grid Map)는 중요한 위치를 차지한다. 모라벡(Moravec)과 엘페스(Elfes)에 의해 개척된 이 방법론은 환경을 작은 부피 단위(복셀, Voxel)로 나누고 각 복셀이 장애물에 의해 점유되었을 확률을 추정하는 방식으로, 불확실한 센서 정보를 통합하는 강력한 프레임워크를 제공했다.6 OctoMap과 같은 옥트리(Octree) 기반의 확률적 점유 격자 기법은 3D 공간을 효율적으로 표현하며 로봇 공학계에서 널리 사용되어 왔다.7 하지만 전통적인 CPU 기반의 점유 격자 방식은 근본적인 한계를 지닌다. 고해상도 맵을 생성할 경우 메모리 소모가 기하급수적으로 증가하며, 방대한 데이터 구조를 갱신하는 데 필요한 계산 비용은 실시간성을 저해하는 주요 원인이 된다. 이는 동적인 환경에서 맵 갱신 지연을 초래하여 로봇의 안전한 운용을 위협할 수 있다.4

초기 로봇 매핑 연구가 센서 노이즈 처리와 불확실한 공간의 근본적인 표현 방법에 초점을 맞췄다면 1, 현대의 매핑 문제는 센서 기술의 발전으로 인해 데이터 처리량(Throughput)과 계산 자원 관리의 문제로 그 성격이 전환되었다. 바로 이 지점에서 2022년 Violet Walker 등이 발표한 NanoMap은 중요한 의미를 가진다. NanoMap은 새로운 확률 이론을 제시하기보다는, 현대적 병목 현상을 해결하기 위한 새로운 계산 아키텍처(Computational Architecture)를 제안한다. 이는 시각 효과(VFX) 산업에서 검증된 고효율 희소 데이터 구조인 OpenVDB와, 고성능 컴퓨팅(HPC) 분야의 핵심 기술인 GPU 병렬 처리 능력을 전략적으로 융합한 것이다. 본 보고서는 NanoMap이 제시한 이 아키텍처를 심층적으로 해부하고, 핵심 알고리즘과 성능을 비판적으로 분석하며, 기존 및 후속 기술과의 비교를 통해 3D 매핑 기술의 지형도 속에서 NanoMap의 현재적 가치와 미래 전망을 고찰하고자 한다. 결론적으로, NanoMap은 고성능 시스템뿐만 아니라 임베디드 시스템에서도 실시간 고해상도 점유 격자 매핑을 향한 실용적이고 강력한 길을 제시한 중요한 기술적 이정표로 평가될 수 있다.

## 2.  NanoMap의 아키텍처 해부: GPU와 희소 볼륨 데이터의 융합

NanoMap의 혁신성은 완전히 새로운 개념의 발명이 아니라, 각기 다른 분야에서 고도로 발전된 강력한 기술들을 로봇 매핑이라는 특정 문제 해결을 위해 독창적으로 융합한 데 있다. 이는 시각 효과 산업의 OpenVDB, 고성능 컴퓨팅의 CUDA, 그리고 이 둘을 연결하는 NanoVDB라는 세 가지 핵심 요소의 전략적 조합으로 구현된다. NanoMap의 아키텍처를 이해하는 것은 이 세 가지 구성 요소의 역할과 상호작용, 그리고 그로 인해 발생하는 제약사항이 어떻게 전체 시스템 설계를 결정했는지를 파악하는 과정이다.

### 2.1 A. OpenVDB: The Cornerstone of Sparse Data Representation

NanoMap 아키텍처의 근간을 이루는 것은 OpenVDB라는 데이터 구조이다.4 OpenVDB는 원래 대규모의 희소 볼륨 데이터(Sparse Volumetric Data)를 효율적으로 저장하고 조작하기 위해 영화 및 시각 효과(VFX) 산업에서 개발된 C++ 라이브러리다.9 VFX 환경은 방대한 데이터셋을 다루면서도 높은 성능과 메모리 효율성을 동시에 요구하는데, 이러한 특성이 로봇 매핑 문제에도 매우 적합하다.

OpenVDB의 핵심적인 구조적 특징은 B+ 트리와 유사한, 얕고 넓은(Shallow and Wide) 계층 구조에 있다.11 전통적인 옥트리(Octree)가 하나의 노드를 8개의 자식 노드로 재귀적으로 분할하여 깊고 좁은(Deep and Narrow) 트리를 형성하는 것과 대조적이다. B+ 트리와 유사한 구조는 루트 노드에서 특정 복셀 데이터가 저장된 리프 노드(Leaf Node)까지 도달하는 데 필요한 포인터 역참조(Indirection) 횟수를 최소화한다. 이는 데이터 접근 속도에 직접적인 영향을 미치며, 잦은 데이터 접근이 필요한 매핑 작업에서 옥트리 구조 대비 상당한 성능 이점을 제공한다.4

또한, OpenVDB는 '활성화된(active)' 공간에 대해서만 메모리를 할당하는 동적 희소 표현 방식을 사용한다.9 로봇이 탐사하는 대부분의 환경은 장애물보다 빈 공간이 훨씬 넓기 때문에, 맵 데이터는 본질적으로 희소(sparse)하다. OpenVDB는 이러한 희소성을 활용하여 비어있는 광대한 영역에 대한 메모리 낭비를 원천적으로 방지한다. 이는 고정된 크기의 배열을 사용하는 밀집 표현(dense representation) 방식에 비해 압도적인 메모리 효율성을 가지며, 옥트리보다도 더욱 압축적인 표현이 가능하다. 더불어, 맵의 경계를 미리 지정할 필요 없이 탐사 영역이 넓어짐에 따라 동적으로 확장될 수 있어 미지의 환경을 탐사하는 로봇에게 필수적인 유연성을 제공한다.7

### 2.2 B. CUDA: Unleashing Parallel Processing for Ray Casting

OpenVDB가 데이터 '저장'의 효율성을 담당한다면, 데이터 '처리'의 속도는 NVIDIA의 CUDA(Compute Unified Device Architecture)가 책임진다. CUDA는 GPU의 방대한 병렬 처리 능력을 일반 목적의 컴퓨팅에 활용할 수 있도록 하는 병렬 컴퓨팅 플랫폼 및 프로그래밍 모델이다.13

CUDA의 핵심은 SIMT(Single Instruction, Multiple Thread) 실행 모델에 있다.14 SIMT 모델에서 GPU는 수십 개의 스레드를 '워프(Warp)'라는 단위(일반적으로 32개 스레드)로 묶어 관리한다. 한 워프 내의 모든 스레드는 동일한 명령어를 동시에 실행하되, 각자 다른 데이터를 처리한다.13 이러한 구조는 데이터 병렬성(Data Parallelism)이 높은 문제에 매우 적합하다.

로봇 매핑에서 가장 계산 비용이 많이 드는 작업 중 하나는 레이캐스팅(Ray Casting)이다. 이는 센서 원점으로부터 관측된 포인트 클라우드의 각 끝점(endpoint)까지 가상의 광선(ray)을 쏘아, 광선이 통과하는 공간은 '비어있음(free)'으로, 끝점이 위치한 공간은 '점유됨(occupied)'으로 갱신하는 과정이다. 이 레이캐스팅 과정은 본질적으로 '당혹스러울 정도로 병렬적인(Embarrassingly Parallel)' 문제다. 즉, 각각의 광선에 대한 계산은 다른 광선의 계산과 완전히 독립적으로 수행될 수 있다.16 NanoMap은 바로 이 지점을 공략한다. 수만, 수십만 개의 광선 처리 작업을 수천 개의 GPU 코어에 분산 할당함으로써, CPU에서 순차적으로 처리할 때와는 비교할 수 없는 엄청난 속도 향상을 달성한다.4

그러나 GPU를 활용하는 데에는 한 가지 중요한 난관이 존재한다. 바로 호스트(CPU) 메모리와 디바이스(GPU) 메모리 간의 데이터 전송 병목이다.4 GPU가 아무리 빨리 계산을 마쳐도, 계산에 필요한 데이터를 GPU로 보내고 결과를 다시 CPU로 가져오는 과정이 느리다면 전체 시스템의 성능은 저하된다. NanoMap의 개발자들은 이 문제를 핵심적인 구현 과제로 인식했으며, 뒤이어 설명할 NanoMap의 전체 데이터 흐름과 알고리즘 설계는 이 CPU-GPU 간의 데이터 전송 오버헤드를 최소화하려는 노력의 산물이라 할 수 있다.

### 2.3 C. The Role and Limitations of NanoVDB

OpenVDB와 CUDA를 연결하는 다리 역할을 하는 것이 NanoVDB이다. NanoVDB는 OpenVDB 데이터 구조를 GPU에서 효율적으로 렌더링하고 접근할 수 있도록 설계된 모듈이다.4 이는 OpenVDB 트리를 GPU가 이해할 수 있는 선형적이고 압축된, 읽기 전용(read-only)의 형태로 변환하여 GPU 메모리에 적재하는 역할을 한다.17 NanoMap은 GPU에서 레이캐스팅을 수행할 때, 이 NanoVDB가 제공하는 도구와 객체를 사용하여 VDB 트리의 계층 구조를 순회한다.4

여기서 가장 결정적인 제약사항은 NanoVDB를 통해 GPU에 올라간 데이터 구조가 '읽기 전용'이라는 점이다.4 GPU 아키텍처의 특성상, 복잡한 트리 구조의 메모리를 동적으로 할당하고 노드를 삽입하거나 삭제하는 작업은 매우 어렵고 비효율적이다. 이 제약사항은 NanoMap의 전체 아키텍처를 결정짓는 가장 중요한 요인이다. 만약 GPU에서 맵을 직접 수정할 수 있었다면, 모든 작업이 GPU 내에서 완결되었을 것이다.

하지만 이 '읽기 전용' 제약 때문에 NanoMap은 하이브리드(Hybrid) CPU-GPU 아키텍처를 채택할 수밖에 없었다. 즉, GPU는 맵을 직접 '수정'하는 것이 아니라, 맵을 '수정하기 위해 필요한 정보'를 계산하는 역할만 담당한다. GPU는 레이캐스팅과 같은 병렬 계산에 특화된 작업을 수행하여 어떤 복셀들을 '비어있음'으로, 어떤 복셀들을 '점유됨'으로 갱신해야 하는지에 대한 '업데이트 목록'을 생성한다. 이 목록은 압축되어 CPU로 다시 전송되며, 최종적으로 CPU가 이 정보를 바탕으로 메인 OpenVDB 맵 구조에 대한 실제 쓰기(write) 작업을 수행한다.4 따라서 NanoMap은 작업의 성격에 따라 GPU와 CPU의 역할을 명확히 분담한 시스템이며, 이러한 분담은 NanoVDB의 기술적 제약에서 비롯된 필연적인 설계 선택이었다. 또한, NanoMap 라이브러리의 시뮬레이션 기능에서는 명시적인 NanoVDB 그리드 구조가 사용된다.4

## 3.  NanoMap의 핵심 매핑 프로세스 분석

NanoMap의 핵심 매핑 프로세스는 GPU의 병렬 처리 능력을 극대화하고 CPU-GPU 간 데이터 전송량을 최소화하기 위해 정교하게 설계된 다단계 파이프라인이다. 이 프로세스는 하드웨어의 제약을 극복하기 위한 알고리즘적 최적화의 집약체라 할 수 있으며, 크게 GPU 가속 레이캐스팅, 확률적 점유 갱신, 그리고 데이터 압축 및 전송의 세 단계로 나눌 수 있다.

### 3.1 A. The Two-Stage GPU-Accelerated Ray Casting Mechanism

NanoMap은 무작정 모든 복셀에 대해 레이캐스팅을 수행하는 대신, OpenVDB의 계층 구조를 활용한 효율적인 2단계 접근법을 사용한다. 이는 전형적인 'Coarse-to-Fine' (개략적 단계에서 세부적 단계로) 최적화 전략이다.

- Stage 1: Coarse-Grained Leaf Node Ray Casting (개략적 리프 노드 레이캐스팅)

  프로세스는 개별 복셀 수준이 아닌, OpenVDB 트리의 리프 노드(Leaf Node) 수준에서 시작된다. 리프 노드는 복셀들의 집합(예: 기본 설정에서 8x8x8=512개의 복셀)으로 구성된 큐브 형태의 공간이다.4 GPU는 먼저 센서의 광선들이 어떤 리프 노드들을 통과하는지를 계산한다. 이 단계는 일종의 광역 컬링(Broad-phase Culling) 역할을 한다. 광선이 전혀 지나가지 않는 맵의 광대한 빈 공간들은 이 단계에서 즉시 계산 대상에서 제외된다. 이를 통해 GPU는 이후의 연산을 '활성화된(active)' 리프 노드, 즉 실제로 정보가 갱신될 가능성이 있는 영역에만 집중할 수 있게 된다. 이는 계산량을 대폭 줄이고 GPU 내에서 필요한 임시 데이터 구조의 크기를 최소화하는 효과를 가져온다.

- Stage 2: Fine-Grained Voxel-Level Ray Casting (세부적 복셀 수준 레이캐스팅)

  1단계에서 '활성화'된 것으로 식별된 리프 노드들에 대해서만, GPU는 더 계산 비용이 높은 복셀 수준의 레이캐스팅을 수행한다.4 GPU 스레드들은 각 광선을 따라 이 활성 노드 내부를 단계적으로 진행(step along)하며, 광선이 통과하는 모든 개별 복셀의 좌표를 식별하여 임시 버퍼에 기록한다. 이 2단계 접근법은 OpenVDB의 계층적 데이터 구조와 GPU의 병렬 처리 능력을 결합하여 불필요한 연산을 회피하는 NanoMap의 핵심적인 성능 최적화 기법이다.

- GPU-Accelerated Voxelization Filter (GPU 가속 복셀화 필터)

  NanoMap은 여기서 한 걸음 더 나아가, 입력 데이터 자체의 중복성을 해결하기 위한 최적화 기법을 제공한다. 고해상도 깊이 카메라와 같은 센서는 매우 조밀한 포인트 클라우드를 생성하는데, 이 경우 다수의 포인트가 맵의 동일한 단일 복셀에 해당될 수 있다.4 이 모든 포인트에 대해 개별적으로 레이캐스팅을 수행하는 것은 극심한 낭비다.

  이를 해결하기 위해 NanoMap은 GPU 가속 복셀화 필터를 제공한다. 이 필터는 레이캐스팅을 수행하기 전에 먼저 입력 포인트 클라우드의 모든 점을 해당하는 최종 복셀에 매핑한다. 그런 다음, 중복된 점들을 병합하여 각 점유 복셀에 대해 단 한 번의 레이캐스팅만 수행한다.4 이 최적화는 맵의 해상도가 센서의 포인트 밀도보다 낮을 때 처리 속도를 극적으로 향상시키는 효과를 가져온다.

### 3.2 B. Probabilistic Occupancy Update

GPU가 레이캐스팅을 통해 어떤 복셀이 비어있고 어떤 복셀이 점유되었는지에 대한 정보를 생성하면, 이 정보를 기존 맵에 통합해야 한다. NanoMap은 이 과정에서 OctoMap과 유사한 확률적 갱신 방식을 사용한다.4

- Bayesian Inference and Log-Odds (베이즈 추론과 로그-오즈)

  이 방식은 베이즈 추론에 기반하며, 각 복셀의 점유 상태를 이진 확률 변수로 모델링한다. 그러나 확률 값을 직접 저장하고 갱신하는 것은 곱셈 연산이 필요하여 수치적으로 불안정하고 계산 비용이 높을 수 있다. 대신, 대부분의 점유 격자 기법은 확률을 로그-오즈(Log-Odds) 형태로 변환하여 사용한다.8 로그-오즈는 확률 $P(n)$을 다음과 같이 변환한 값 $L(n)$이다.
  $$
  L(n) = \log\left(\frac{P(n)}{1 - P(n)}\right)
  $$
  이 변환의 가장 큰 장점은 베이즈 갱신 규칙이 복잡한 곱셈에서 간단한 덧셈 연산으로 바뀐다는 점이다. 새로운 관측 zt가 주어졌을 때, 복셀 n의 로그-오즈 값은 다음과 같이 갱신된다.19
  $$
  L(n | z_{1:t}) = L(n | z_{1:t-1}) + L(n | z_t)
  $$
  여기서 $L(n | z_{1:t-1})$은 이전까지의 누적된 로그-오즈 값이며, $L(n | z_t)$는 현재 관측에 대한 역센서 모델(Inverse Sensor Model)이다. 역센서 모델은 센서 '히트(hit)', 즉 광선의 끝점이 해당 복셀에 위치할 경우 양수 값을, 센서 '미스(miss)', 즉 광선이 해당 복셀을 통과했으나 장애물이 없었을 경우 음수 값을 반환한다.18 이 덧셈 기반 갱신 방식은 여러 번의 관측을 자연스럽게 융합하고 센서 노이즈에 강건한 맵을 구축할 수 있게 한다. NanoMap 논문 자체는 이 수식을 명시하지 않았지만 4, OctoMap과 유사한 방식을 채택했다고 언급한 점으로 보아 표준적인 로그-오즈 업데이트 규칙을 따르는 것으로 추정된다.

### 3.3 C. Data Flow and Compression

GPU에서의 계산이 완료된 후, 결과물을 효율적으로 CPU로 가져와 최종 맵에 통합하는 과정은 NanoMap 파이프라인의 마지막 핵심 단계이다.

- From GPU Buffers to CPU Map (GPU 버퍼에서 CPU 맵으로)

  2단계 레이캐스팅이 끝나면, GPU 메모리에는 광선이 통과한 모든 복셀의 좌표 정보가 담긴 거대한 부동소수점(float) 배열이 생성된다.4 이 배열은 갱신이 필요한 모든 '비어있음' 및 '점유됨' 복셀의 목록이다.

- On-GPU Compression (GPU 상에서의 압축)

  이 부동소수점 배열을 그대로 CPU로 전송하는 것은 PCI-Express 버스의 대역폭을 과도하게 사용하여 병목 현상을 유발할 수 있다. NanoMap은 이 문제를 해결하기 위해 데이터를 전송하기 전에 GPU 상에서 압축을 수행한다. 구체적으로, 각 좌표 값을 나타내는 부동소수점 배열을 그에 상응하는 단일 바이트 정수(single-byte integer) 배열로 변환한다.4 이 간단한 변환만으로도 전송해야 할 데이터의 크기가 1/4로 줄어들며(float가 4바이트, 정수가 1바이트일 경우), 이는 CPU-GPU 간 데이터 전송 속도를 크게 향상시키는 결정적인 최적화 단계이다.

- CPU-Side Integration (CPU 측 통합)

  압축된 복셀 좌표 데이터가 CPU 메모리로 복사되면, 해당 프레임에 대한 GPU의 역할은 종료된다. 이제 CPU는 이 데이터를 순회하며 실제 맵 갱신 작업을 수행한다. 즉, CPU는 전달받은 좌표 목록을 바탕으로 메인 OpenVDB 맵 데이터 구조에 접근하여 각 복셀의 로그-오즈 값을 덧셈 연산을 통해 갱신한다.4 다음 포인트 클라우드 데이터가 수신될 때까지 이 CPU 측 통합 작업이 진행된다. 이처럼 NanoMap은 각 하드웨어의 장점을 극대화하는 방식으로 작업을 분할하여 전체 파이프라인의 처리량을 극대화한다.

## 4.  성능 평가 및 기존 기술과의 비교 분석

NanoMap의 진정한 가치는 제안된 아키텍처가 실제로 얼마나 뛰어난 성능을 보이는지에 대한 정량적 평가를 통해 입증된다. NanoMap 논문은 다양한 플랫폼과 시나리오에서 기존의 대표적인 매핑 기술인 OctoMap과 직접적인 성능 비교를 수행했으며, 그 결과는 GPU 가속화의 잠재력을 명확하게 보여준다.

### 4.1 A. Experimental Setup and Benchmark Analysis

NanoMap의 실험 설계에서 가장 주목할 만한 점은 두 가지 매우 다른 특성의 하드웨어 플랫폼에서 성능을 측정했다는 것이다. 하나는 고성용 노트북(Ryzen 4900HS CPU, Nvidia RTX 2060 MaxQ GPU)이고, 다른 하나는 저전력 임베디드 시스템(Jetson Nano)이다.4 이러한 이원적 평가는 NanoMap의 접근 방식이 고성능 컴퓨팅 환경뿐만 아니라, 실제 로봇(특히 드론과 같은 소형 로봇)에 탑재되는 자원 제약적인 환경에서도 효과적인지를 검증하는 중요한 척도가 된다. 이는 제안된 기술의 확장성과 실용성을 동시에 보여주는 강력한 증거이다.

데이터셋으로는 두 가지 유형이 사용되었다. 프러스텀(frustum, 절두체) 기반의 밀집 데이터 테스트를 위해, 로봇 비전 분야에서 널리 사용되는 표준 벤치마크 데이터셋인 TUM Freiburg Long Office Household Dataset이 사용되었다.4 이는 RGB-D 카메라와 같은 센서를 시뮬레이션하며, 다른 연구들과의 공정한 비교를 가능하게 한다. 반면, 희소 데이터 테스트를 위해서는 VLP-16 LiDAR의 센서 특성을 모방한 시뮬레이션 데이터셋을 NanoMap 자체의 시뮬레이션 기능을 사용하여 생성했다.4 이는 특정 센서(LiDAR)에 대한 성능을 통제된 환경에서 정밀하게 평가하기 위함이다.

아래 표 1은 NanoMap을 동시대의 주요 3D 매핑 프레임워크와 비교하여 그 구조적, 기능적 차이점을 요약한 것이다.

**Table 1: Key 3D Mapping Frameworks Comparison**

| Feature                    | OctoMap                                 | VDB-Mapping                          | Voxblox                                       | NanoMap                      |      |
| -------------------------- | --------------------------------------- | ------------------------------------ | --------------------------------------------- | ---------------------------- | ---- |
| **Core Data Structure**    | Octree                                  | OpenVDB (B+ Tree-like)               | Hashed Voxel Grid                             | OpenVDB (B+ Tree-like)       |      |
| **Primary Representation** | Probabilistic Occupancy                 | Probabilistic Occupancy              | Truncated Signed Distance Field (TSDF)        | Probabilistic Occupancy      |      |
| **Primary Compute**        | CPU                                     | CPU                                  | CPU                                           | CPU-GPU Hybrid               |      |
| **Key Advantage**          | Widespread adoption, good for basic use | Good CPU performance for sparse data | Excellent for surface reconstruction/planning | Extreme speed for dense data |      |
| **Key Disadvantage**       | CPU bottleneck with dense data          | CPU-only, limited adoption           | Not occupancy-based, focused on local maps    | LiDAR pipeline is CPU-only   |      |

Data sourced from.4

### 4.2 B. Quantitative Performance: NanoMap vs. OctoMap

성능 비교 결과는 NanoMap의 압도적인 우위를 보여준다. 특히 밀집 포인트 클라우드를 처리하는 프러스텀 테스트에서 그 차이가 두드러진다.

- **Frustum Test Results:** 저전력 플랫폼인 Jetson Nano에서조차 NanoMap은 OctoMap 대비 엄청난 속도 향상을 보였다. 0.1m 해상도에서는 5.5배, 그리고 해상도가 높아져 계산량이 더 많아지는 0.025m 해상도에서는 7.6배 더 빠른 성능을 기록했다.4 고성능 노트북에서는 그 격차가 더욱 극적으로 벌어진다. 동일한 조건에서 각각 7.5배, 그리고 무려 50.8배의 성능 차이를 보였다.4

이러한 비선형적인 성능 향상은 CPU와 GPU의 근본적인 작동 방식의 차이에서 기인한다. OctoMap과 같은 CPU 기반 방식은 해상도가 높아질수록 트리 순회 깊이가 깊어지고 캐시 미스가 증가하는 등, 계산 비용이 급격하게 증가하는 구조적 한계를 가진다. 반면, GPU는 방대한 병렬 연산에 최적화되어 있어, 처리해야 할 복셀의 수가 늘어나는 것을 '더 많은 병렬 작업'으로 소화할 수 있다. GPU 접근법의 커널 실행 및 메모리 복사와 같은 고정 오버헤드는 작업량이 증가함에 따라 상각(amortize)되는 반면, CPU 접근법의 확장성 비용은 지배적이게 된다. 결과적으로, 고해상도의 밀집 매핑 작업일수록 GPU 기반인 NanoMap의 성능 우위가 비선형적으로 증폭되는 것이다.

- **Achieving Real-Time on Embedded Systems:** 아마도 이 논문에서 가장 중요한 성과는 저전력 임베디드 시스템에서의 실시간 성능 달성일 것이다. Jetson Nano에서 0.05m의 비교적 높은 해상도로 매핑을 수행하면서도 평균 프레임 처리 시간을 30ms 이하로 유지했다는 것은, 초당 30프레임(30 Hz) 이상의 센서 입력을 실시간으로 처리할 수 있음을 의미한다.4 이는 기존의 CPU-only 방식으로는 달성하기 매우 어려웠던 목표로, 페이로드와 전력에 제약이 큰 드론과 같은 로봇 플랫폼에 고해상도 3D 매핑 기능을 탑재할 수 있는 실질적인 가능성을 열었다는 점에서 큰 의의를 가진다.

### 4.3 C. Sparse Data (LiDAR) Processing: A Noted Limitation

논문은 LiDAR와 같이 희소한(sparse) 포인트 클라우드를 처리하는 파이프라인은 현재 CPU-only로 구현되어 있음을 명시적으로 밝히고 있다.4

- **CPU-Only Implementation and Performance:** 흥미로운 점은, GPU 가속 없이 CPU만으로 작동할 때조차 NanoMap의 OpenVDB 기반 접근 방식이 OctoMap보다 뛰어난 성능을 보인다는 것이다.4 이는 OpenVDB 데이터 구조 자체가 옥트리에 비해 데이터 접근 및 삽입 연산에서 근본적으로 더 효율적임을 시사한다.
- **Reasoning for Lack of GPU Acceleration:** 저자들은 LiDAR 데이터의 희소한 특성과 상대적으로 낮은 해상도로 인해, 밀집 데이터 처리에 최적화된 현재의 GPU 가속 방식으로는 큰 이점을 얻지 못한다고 설명한다.4 이는 NanoMap의 GPU 파이프라인이 밀집된 광선 다발(frustum of rays)을 처리하도록 설계되었기 때문이다. LiDAR의 불규칙하고 희소한 광선 패턴에 이 알고리즘을 그대로 적용하는 것은 비효율적일 수 있다.

그러나 이 '한계'는 GPU가 LiDAR 데이터 처리에 부적합하다는 근본적인 결론이 아니라, 2022년 NanoMap 논문의 범위가 밀집 데이터에 초점을 맞추고 있다는 사실을 반영하는 것이다. 이는 GPU 가속이 불가능하다는 의미가 아니라, 희소 광선 패턴에 최적화된 별도의 GPU 커널 설계가 필요함을 시사한다. 실제로 이후의 많은 GPU 기반 SLAM 시스템들은 LiDAR 데이터 처리를 효과적으로 가속화하고 있다.23 따라서 이 한계점은 NanoMap의 약점이라기보다는, 향후 연구를 위한 명확한 방향을 제시하는 지표로 해석될 수 있다.

## 5.  로컬 프레임 중심 매핑: Voxblox와의 개념적 비교

로봇 매핑 시스템을 설계할 때 고려해야 할 근본적인 철학 중 하나는 맵을 '어떻게' 관리할 것인가이다. 로봇이 탐사하는 공간이 무한히 넓거나, 로봇의 임무가 특정 영역의 완전한 재구성이 아닌 지속적인 항해일 경우, 맵 전체를 단일 전역 좌표계(Global Frame)에 저장하고 계속 확장하는 것은 메모리 측면에서 비현실적이다.8 이러한 문제를 해결하기 위해 '로컬 프레임 중심(Local-Frame-Centric)' 매핑이라는 개념이 등장했으며, 이 분야의 대표적인 시스템인 Voxblox와의 비교는 NanoMap의 설계 철학을 더욱 명확하게 이해하는 데 도움을 준다.

### 5.1 A. The Challenge of Dynamic Environments and Large-Scale Exploration

로봇이 광활한 지역을 탐사하거나 장시간 임무를 수행할 때, 전역 맵(Global Map)은 계속해서 커진다. 이는 결국 사용 가능한 메모리를 모두 소진하게 만들며, 맵의 크기가 커질수록 데이터 구조를 탐색하고 갱신하는 데 걸리는 시간도 증가하여 시스템 전체의 성능을 저하시킬 수 있다. 더욱이, 대부분의 로봇 작업, 예를 들어 즉각적인 장애물 회피나 지역 경로 계획 등은 로봇 주변의 즉각적인 환경 정보만을 필요로 한다. 로봇으로부터 수백 미터 떨어진 과거의 맵 정보는 현재의 의사결정에 거의 영향을 미치지 않는다. 이러한 배경에서 로봇의 현재 위치를 중심으로 한 '슬라이딩 윈도우(Sliding Window)' 형태의 맵을 유지하는 아이디어가 탄생했다.

### 5.2 B. Voxblox's Approach: The Circular Buffer for a "Sliding Window" Map

Voxblox는 주로 마이크로 비행체(MAV)의 실시간 경로 계획을 위해 개발된 매핑 시스템으로, 로컬 맵 관리의 대표적인 사례를 보여준다.21 Voxblox는 점유 확률 대신 장애물까지의 부호화된 유클리드 거리(Signed Euclidean Distance)를 저장하는 TSDF(Truncated Signed Distance Field) 맵을 생성하는데, 이는 특히 최적화 기반의 경로 계획기(Trajectory Optimization Planner)에 매우 유용한 정보를 제공한다.24

이러한 로컬 맵을 효율적으로 관리하기 위해 Voxblox는 순환 버퍼(Circular Buffer), 또는 링 버퍼(Ring Buffer)라 불리는 데이터 구조를 활용한다.25 순환 버퍼는 고정된 크기의 메모리 공간을 마치 끝과 시작이 연결된 고리처럼 사용하는 방식이다.25 로봇이 앞으로 움직이면, 진행 방향에 있는 새로운 맵 영역(블록 또는 청크)이 버퍼에 할당된다. 동시에, 로봇의 뒤쪽으로 멀어져 더 이상 유용하지 않게 된 오래된 맵 영역은 버퍼에서 해제된다. 해제된 메모리 공간은 다시 새로운 맵 영역을 위해 재사용된다. 이 메커니즘을 통해 맵은 로봇을 따라다니는 '슬라이딩 윈도우'처럼 작동하며, 전체 메모리 사용량을 일정하게 유지하면서도 로봇 주변의 최신 정보를 항상 담보할 수 있다.

### 5.3 C. NanoMap's Global Frame Approach and Its Implications

반면, NanoMap 논문은 전역 좌표계(Global Coordinate Frame)에 맵을 구축하는 시스템을 설명한다.4 논문에서는 오래된 데이터를 제거하거나 맵의 원점을 이동시키는 메커니즘에 대한 언급이 없다.4 이는 NanoMap이 로컬 프레임 중심의 매핑 시스템과는 다른 설계 철학을 가지고 있음을 시사한다.

이 차이는 두 시스템의 근본적인 목적의 차이에서 비롯된다. Voxblox는 잠재적으로 매우 넓은 환경에서 '지속적인 항해(Navigation)'와 '지역 경로 계획(Local Planning)'을 가능하게 하는 데 초점을 맞춘다.21 맵은 항해를 위한 '수단'이며, 따라서 로봇 주변의 정보만 효율적으로 유지하는 것이 중요하다.

반면, NanoMap의 설계 목표와 성능 지표는 한정된 영역의 '완전한 재구성(Reconstruction)'을 최대한 '빠르게' 수행하는 데 맞춰져 있다.4 NanoMap이 답하고자 하는 질문은 "지금 내 주변에 장애물이 어디 있는가?"라기보다는 "이 공간 전체가 어떻게 생겼는가?"에 가깝다. 알려진 크기의 특정 물체나 지역을 신속하게 스캔하고 검사하는 작업이 이에 해당한다. 이러한 재구성 임무에서는 맵 전체를 일관된 전역 좌표계에 저장하는 것이 자연스럽고 필수적이다.

결론적으로, NanoMap과 Voxblox는 모든 작업에서 서로를 대체하는 직접적인 경쟁 관계가 아니다. 이들은 로봇 공학이라는 더 넓은 문제 영역의 서로 다른 측면을 해결하기 위해 특화된 도구들이다. NanoMap은 '신속한 장면 조사'에, Voxblox는 '지속적인 지역 항해'에 더 적합하다. 실제로 완전한 자율 로봇 시스템은 이 두 가지 철학을 모두 활용할 수 있다. 예를 들어, 초기 환경 탐사 단계에서는 NanoMap과 같은 빠른 재구성 시스템을 사용하여 전역 맵의 개요를 신속하게 파악하고, 이후 지속적인 임무 수행 단계에서는 Voxblox와 같은 로컬 매핑 시스템으로 전환하여 효율적인 항해를 수행하는 하이브리드 접근법을 상상해 볼 수 있다.

## 6.  2022년 이후의 지형 변화: NanoMap의 현재적 가치 재평가

기술 발전의 속도가 매우 빠른 로봇 공학과 컴퓨터 비전 분야에서 2022년에 발표된 NanoMap의 가치를 현재 시점에서 재평가하기 위해서는 그 이후에 등장한 새로운 기술들과의 비교가 필수적이다. NanoMap이 GPU 가속 점유 격자 매핑의 새로운 기준을 제시한 이후, 이 분야는 더욱 정교하고 다양한 방향으로 분화하며 발전했다. 특히 NVIDIA의 nvblox, 그리고 렌더링 품질과 표현력에서 혁신을 가져온 NeRF와 3D Gaussian Splatting의 등장은 3D 매핑의 지형도를 크게 바꾸어 놓았다.

### 6.1 A. Nvblox: The GPU-Native Successor for Robotic Planning

NanoMap 발표 이후 등장한 nvblox는 GPU 가속 로봇 매핑의 자연스러운 진화 방향을 보여주는 대표적인 사례이다.28 NVIDIA에서 개발한 오픈소스 라이브러리인 nvblox는 NanoMap과 마찬가지로 GPU를 활용하여 체적 맵(Volumetric Map)을 실시간으로 구축하지만, 그 목적과 출력물에서 중요한 차이를 보인다.

nvblox의 핵심 목표는 로봇 경로 계획에 직접적으로 사용될 수 있는 맵을 생성하는 것이다. 이를 위해 nvblox는 TSDF(Truncated Signed Distance Field)를 기반으로 ESDF(Euclidean Signed Distance Field)를 계산하는 데 중점을 둔다.28 ESDF는 맵 상의 모든 지점에서 가장 가까운 장애물까지의 실제 유클리드 거리를 제공하기 때문에, 충돌 없는 안전한 경로를 생성하는 최적화 기반 플래너에게 매우 중요한 정보를 제공한다.

성능 면에서 nvblox는 처음부터 GPU 네이티브로 설계되어, CPU 기반의 Voxblox와 같은 기존 방법 대비 표면 재구성에서 최대 177배, 거리장 계산에서 최대 31배의 엄청난 속도 향상을 달성했다고 주장한다.28 이는 빠른 GPU 재구성 기술과 로봇 경로 계획에 필수적인 맵 표현 사이의 간극을 메우는 중요한 발전이다.

NanoMap과의 비교 관점에서, 두 시스템은 서로 다른 목적을 위한 도구로 볼 수 있다. NanoMap은 "이 공간이 점유되었는가, 비어있는가?"라는 질문에 답하는 '점유(Occupancy)' 정보를 초고속으로 제공한다. 반면, nvblox는 "가장 가까운 장애물까지의 거리가 얼마인가?"라는 질문에 답하는 '거리(Distance)' 정보를 제공한다. 따라서 순수한 점유 격자 맵의 신속한 구축이 목표라면 NanoMap이 여전히 강력한 솔루션이지만, 구축된 맵을 기반으로 한 정교한 동적 경로 계획이 주된 목적이라면 nvblox가 더 적합한 후속 기술이라 할 수 있다.

### 6.2 B. The Rise of Neural and Gaussian Representations

2022년 이후 3D 장면 표현 분야에서 가장 큰 파장을 일으킨 것은 단연 신경망 기반의 표현 방식들이다.

- **Neural Radiance Fields (NeRF):** NeRF는 여러 장의 2D 이미지로부터 3D 장면의 형상과 외관을 신경망(MLP)의 가중치 안에 암시적(Implicit)으로 학습하는 혁신적인 기술이다.29 NeRF는 특정 시점에서 바라본 장면을 놀랍도록 사실적인 품질의 새로운 이미지로 렌더링할 수 있다. 초기에는 학습에 수 시간에서 수일이 걸리는 오프라인 기술이었지만, 이후 연구를 통해 실시간 학습 및 렌더링이 가능한 변형들이 등장하면서 로봇 공학 분야에도 빠르게 통합되고 있다. NeRF-VINS, RO-MAP과 같은 시스템들은 NeRF를 SLAM(Simultaneous Localization and Mapping)에 접목하여, 시각적 정보에 기반한 더 정확한 위치 추정과 고품질의 맵 재구성을 동시에 수행한다.31
- **3D Gaussian Splatting (3DGS):** 3DGS는 NeRF보다 더 최근에 등장하여 실시간 렌더링 분야에 큰 충격을 준 기술이다. 3DGS는 장면을 수백만 개의 3D 가우시안(위치, 모양, 색상, 불투명도를 가진 타원체)들의 집합이라는 명시적(Explicit)인 형태로 표현한다.34 이 방식은 NeRF와 동등하거나 더 나은 품질의 렌더링을 훨씬 빠른 속도로 수행할 수 있어, 로보틱스, 가상현실(VR), 증강현실(AR) 등 다양한 분야에서 폭발적인 관심을 받고 있다. 현재 3DGS를 자원이 제한된 엣지 디바이스에서 구동하고 대규모 장면을 스트리밍하기 위한 가속화 연구가 활발히 진행 중이다.35

이러한 신경망 기반 표현 방식들은 NanoMap과 같은 전통적인 기하학적 맵과는 근본적으로 다른 패러다임을 제시한다. NanoMap이 '미터(meter)' 단위의 정확한 기하학적 구조에 초점을 맞춘다면, NeRF와 3DGS는 '픽셀(pixel)' 단위의 시각적 사실성에 중점을 둔다.

아래 표 2는 2022년 이후 등장한 주요 GPU 기반 3D 매핑 기술들의 특징을 비교하여 현재 기술 지형도를 요약한 것이다.

**Table 2: Post-2022 GPU-Based 3D Mapping Technology Landscape**

| Technology                  | Core Representation   | Primary Output               | Key Advantage                             | Primary Application                        |      |
| --------------------------- | --------------------- | ---------------------------- | ----------------------------------------- | ------------------------------------------ | ---- |
| **NanoMap (2022)**          | OpenVDB               | Probabilistic Occupancy Grid | Extreme speed for dense occupancy mapping | Rapid inspection/reconstruction            |      |
| **nvblox (2023)**           | Hashed Voxel Grid     | TSDF/ESDF                    | Real-time distance field for planning     | On-board robot navigation                  |      |
| **NeRF-based SLAM (2023+)** | Neural Network (MLP)  | Color + Density              | High-fidelity rendering from sparse views | Visually-aided localization, digital twins |      |
| **3DGS (2023+)**            | Explicit 3D Gaussians | Color + Covariance           | Unprecedented real-time rendering quality | Novel view synthesis, VR/AR                |      |

Data sourced from.4

### 6.3 C. NanoMap's Enduring Legacy and Future Role

이처럼 빠르게 변화하는 기술 환경 속에서 NanoMap의 가치는 무엇인가? NanoMap은 더 이상 모든 면에서 최첨단 기술은 아닐지라도, 여전히 중요하고 지속적인 유산을 남겼다.

- **Pioneering Contribution:** NanoMap의 가장 큰 공헌은 전통적인 확률적 점유 격자 매핑에 GPU 가속을 적용했을 때 얻을 수 있는 엄청난 성능 향상을 정량적으로 입증하고, 그 구체적인 구현 방법을 제시했다는 점이다. 이는 후속 GPU 기반 매핑 시스템들의 개발을 촉진하는 기폭제가 되었으며, 새로운 성능 기준선을 설정했다.
- **A Niche of Simplicity and Speed:** 점점 더 복잡해지는 신경망 모델의 시대에, NanoMap의 강점은 오히려 그 상대적인 '단순함'과 명확하게 정의된 작업에 대한 '압도적인 속도'에 있다. 사실적인 렌더링이나 거리장 정보 없이, 오직 빠르고 정확한 고해상도 점유 격자 맵만이 필요한 응용 분야(예: 단순 장애물 유무 확인, 부피 측정, 변화 탐지 등)에서는 복잡한 신경망 모델보다 NanoMap이 훨씬 더 효율적이고 실용적인 솔루션이 될 수 있다.
- **Future Integration:** NanoMap이 보여준 원리들, 즉 GPU 가속 레이캐스팅과 효율적인 메모리 관리 기법은 여전히 근본적이고 중요하다. 미래의 진보된 매핑 시스템은 NanoMap과 같은 빠른 기하학적 처리 기술과 NeRF와 같은 신경망 표현 기술을 융합하는 하이브리드 형태가 될 가능성이 높다. 예를 들어, 1차적으로 NanoMap과 유사한 방식으로 대략적인 장면의 기하 구조를 신속하게 파악한 뒤, 이 정보를 기반으로 신경망 모델이 세부적인 텍스처와 의미 정보를 채워 넣는 방식의 협업을 상상할 수 있다. 이처럼 NanoMap은 그 자체로 완결된 솔루션일 뿐만 아니라, 미래의 더 복잡한 시스템을 구성하는 강력한 기초 모듈로서의 역할을 수행할 수 있다.

## 7.  결론: 종합적 평가 및 미래 연구 제언

2022년 발표된 NanoMap은 로봇 3D 매핑 분야에서 중요한 기술적 전환점을 제시했다. 이 연구는 고도로 최적화된 희소 볼륨 데이터 구조인 OpenVDB와 GPU의 대규모 병렬 처리 능력을 결합함으로써, 기존의 CPU 기반 방식으로는 도달하기 어려웠던 실시간 고해상도 점유 격자 매핑의 가능성을 현실로 만들었다. 본 보고서는 NanoMap의 아키텍처, 핵심 프로세스, 성능을 심층적으로 분석하고, 동시대 및 후속 기술들과의 비교를 통해 그 역사적 의의와 현재적 가치를 평가했다.

### Summary of NanoMap's Contributions

NanoMap의 핵심 기여는 다음과 같이 요약할 수 있다. 첫째, GPU를 활용하여 전통적인 확률적 점유 격자 매핑의 가장 큰 병목이었던 레이캐스팅 연산을 극적으로 가속할 수 있음을 정량적으로 입증했다. 특히 고해상도, 밀집 데이터 환경에서 CPU 기반의 OctoMap 대비 최대 50배 이상의 성능 향상을 보인 것은 GPU 가속의 잠재력을 명확히 보여준 결과다.4 둘째, 저전력 임베디드 플랫폼인 Jetson Nano에서도 30Hz 이상의 실시간 성능을 달성함으로써, 자원 제약이 심한 소형 로봇 및 드론에서도 고품질 3D 매핑이 가능함을 실증했다.4 이는 이론적 가능성을 넘어 실용적 적용의 문을 연 중요한 성과다. 마지막으로, OpenVDB라는 VFX 산업의 성숙한 기술을 로봇 공학 문제에 성공적으로 도입하여, 데이터 구조 자체가 성능에 미치는 지대한 영향을 환기시켰다.

### 7.1 Addressing the Limitations

물론 NanoMap에도 명확한 한계와 개선의 여지가 존재한다. 가장 두드러지는 한계는 LiDAR와 같은 희소 데이터 처리를 위한 파이프라인이 CPU 기반으로만 구현되어 있다는 점이다.4 이는 논문의 주된 초점이 밀집 데이터 처리에 있었기 때문이지만, 향후 연구를 위한 명확한 방향을 제시한다. 희소한 광선 패턴에 최적화된 새로운 GPU 커널을 설계하는 것은 중요한 후속 연구가 될 수 있다. 이는 현재의 프러스텀 기반 스레드 그룹핑 방식과는 다른, 예를 들어 광선별 또는 지역별 동적 작업 할당과 같은 새로운 병렬화 전략을 요구할 것이다. 또한, 논문에서 스스로 언급했듯이, GPU에서 계산된 결과를 최종 맵에 통합하는 CPU 측 프로세스에 멀티스레딩을 적용하여 추가적인 성능 개선을 탐색하는 것도 유망한 방향이다.4

### 7.2 The Future of GPU-Accelerated Mapping

NanoMap 이후의 기술 지형 변화를 고려할 때, 미래의 GPU 가속 매핑 연구는 다음과 같은 방향으로 나아갈 것으로 전망된다.

- **Algorithm-Hardware Co-design (알고리즘-하드웨어 공동 설계):** 미래의 성능 향상은 단순히 기존 알고리즘을 GPU로 포팅하는 것을 넘어, 하드웨어 아키텍처의 특성을 깊이 이해하고 이에 최적화된 알고리즘을 설계하는 '공동 설계'에서 비롯될 것이다. 3DGS 가속화 연구에서 볼 수 있듯이 35, GPU의 텐서 코어(Tensor Core)나 RTX 시리즈의 레이 트레이싱 전용 하드웨어(RT Core)를 직접 활용하는 알고리즘은 기존 방식으로는 불가능했던 수준의 효율성을 달성할 수 있다.38
- **Hybrid Representations (하이브리드 표현 방식):** 로봇 공학의 다양한 요구사항을 단일 맵 표현 방식으로 모두 충족시키기는 어렵다. 미래는 NanoMap과 같은 기하학적 방법론의 속도 및 계측 정확성과, NeRF나 3DGS와 같은 신경망 표현 방식의 풍부한 시각적, 의미론적 표현력을 융합하는 하이브리드 시스템으로 나아갈 것이다. 로봇은 빠른 충돌 회피를 위해 저수준의 기하학적 맵을 실시간으로 사용하면서, 동시에 배경에서는 고수준의 장면 이해, 객체 인식, 인간-로봇 상호작용을 위해 풍부한 신경망 기반 맵을 구축하고 갱신할 것이다.
- **Beyond Occupancy (점유 정보를 넘어서):** NanoMap이 점유 격자 생성의 가속화에 집중했다면, 미래 연구는 이 GPU 가속 프레임워크를 확장하여 로봇의 고차원적 추론에 필수적인 다른 유형의 맵들을 실시간으로 생성하는 데 초점을 맞출 것이다. 예를 들어, 각 공간의 불확실성을 표현하는 불확실성 맵(Uncertainty Map)이나, 객체의 종류를 식별하는 시맨틱 맵(Semantic Map)을 GPU 상에서 실시간으로 계산하고 갱신하는 기술은 자율 로봇의 상황 인식 능력을 한 차원 높은 수준으로 끌어올릴 것이다.3

결론적으로, NanoMap은 그 자체로 완결된 강력한 도구이자, GPU 가속 3D 매핑이라는 새로운 시대의 서막을 연 중요한 연구이다. 비록 새로운 기술들이 등장하며 그 역할이 변화하고 있지만, NanoMap이 제시한 핵심 원리와 성능에 대한 입증은 미래의 더욱 정교하고 지능적인 로봇 인식 시스템을 위한 견고한 초석으로 남아 지속적인 영향을 미칠 것이다.

#### 참고자료

1. Spatial Representation and Reasoning for Robot Mapping - A Shape-Based Approach - - Temple CIS, accessed August 5, 2025, https://cis.temple.edu/~latecki/Dissertations/Wolterdiss.pdf
2. From SLAM to Situational Awareness: Challenges and Survey - PMC, accessed August 5, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10222985/
3. Challenges and Solutions for Autonomous Ground Robot Scene Understanding and Navigation in Unstructured Outdoor Environments: A Review - MDPI, accessed August 5, 2025, https://www.mdpi.com/2076-3417/13/17/9877
4. NanoMap: A GPU-Accelerated OpenVDB-Based Mapping and ..., accessed August 5, 2025, https://eprints.qut.edu.au/236544/1/118017082.pdf
5. Present and Future of SLAM in Extreme Underground Environments - JPL Robotics, accessed August 5, 2025, https://www-robotics.jpl.nasa.gov/media/documents/2208.01787.pdf
6. OctoMap: A Probabilistic, Flexible, and Compact 3D Map Representation for Robotic Systems - ResearchGate, accessed August 5, 2025, https://www.researchgate.net/publication/235008236_OctoMap_A_Probabilistic_Flexible_and_Compact_3D_Map_Representation_for_Robotic_Systems
7. OctoMap - 3D occupancy mapping, accessed August 5, 2025, https://octomap.github.io/
8. OctoMap: An Efficient Probabilistic 3D Mapping ... - Washington, accessed August 5, 2025, https://courses.cs.washington.edu/courses/cse571/16au/slides/hornung13auro.pdf
9. 3D Gaussian Particle Approximation of VDB Datasets: A Study for Scientific Visualization, accessed August 5, 2025, https://arxiv.org/html/2504.04857v1
10. NanoMap: A GPU-Accelerated OpenVDB-Based Mapping and Simulation Package for Robotic Agents - MDPI, accessed August 5, 2025, https://www.mdpi.com/2072-4292/14/21/5463
11. GPU Volume Rendering with VDB Compression - arXiv, accessed August 5, 2025, https://arxiv.org/html/2504.04564v1
12. Hierarchical, Dense and Dynamic 3D Reconstruction Based on VDB Data Structure for Robotic Manipulation Tasks - Frontiers, accessed August 5, 2025, https://www.frontiersin.org/journals/robotics-and-ai/articles/10.3389/frobt.2020.600387/full
13. Ray Casting Deformable Models on the GPU - CiteSeerX, accessed August 5, 2025, https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=db3b48ccf0d688383cfaab01ba03d7115d35164c
14. [CUDA] Warps and SIMT - Medium, accessed August 5, 2025, https://medium.com/@minkj1992/cuda-warps-and-simt-9fb83d164a24
15. Using CUDA Warp-Level Primitives | NVIDIA Technical Blog, accessed August 5, 2025, https://developer.nvidia.com/blog/using-cuda-warp-level-primitives/
16. Algorithm of Ray Casting Volume Rendering Based on CUDA - SciSpace, accessed August 5, 2025, https://scispace.com/pdf/algorithm-of-ray-casting-volume-rendering-based-on-cuda-3gfb8atzoa.pdf
17. Optimizing Large-Scale Sparse Volumetric Data with NVIDIA NeuralVDB Early Access, accessed August 5, 2025, https://developer.nvidia.com/blog/optimizing-large-scale-sparse-volumetric-data-with-nvidia-neuralvdb-early-access/
18. Improve quality of octomap with params - Robotics Stack Exchange, accessed August 5, 2025, https://robotics.stackexchange.com/questions/43424/improve-quality-of-octomap-with-params
19. OctoMap: An efficient probabilistic 3D mapping framework based on octrees - ResearchGate, accessed August 5, 2025, https://www.researchgate.net/publication/257523133_OctoMap_An_efficient_probabilistic_3D_mapping_framework_based_on_octrees
20. Volumetric Data Fusion of External Depth and Onboard Proximity Data For Occluded Space Reduction - arXiv, accessed August 5, 2025, https://arxiv.org/pdf/2110.11512
21. Voxblox: Incremental 3D Euclidean Signed Distance Fields for On-Board MAV Planning, accessed August 5, 2025, https://huggingface.co/papers/1611.03631
22. ethz-asl/voxblox: A library for flexible voxel-based mapping, mainly focusing on truncated and Euclidean signed distance fields. - GitHub, accessed August 5, 2025, https://github.com/ethz-asl/voxblox
23. GLIM: 3D Range-Inertial Localization and Mapping with GPU-Accelerated Scan Matching Factors - arXiv, accessed August 5, 2025, https://arxiv.org/html/2407.10344v1
24. Voxblox: Incremental 3D Euclidean Signed Distance Fields for On-Board MAV Planning - Helen Oleynikova, accessed August 5, 2025, https://helenol.github.io/publications/iros_2017_voxblox.pdf
25. Circular buffer - Wikipedia, accessed August 5, 2025, https://en.wikipedia.org/wiki/Circular_buffer
26. Creating a Circular Buffer in C and C++ - Embedded Artistry, accessed August 5, 2025, https://embeddedartistry.com/blog/2017/05/17/creating-a-circular-buffer-in-c-and-c/
27. Uncertainty-aware visually-attentive navigation using ... - NTNU Open, accessed August 5, 2025, https://ntnuopen.ntnu.no/ntnu-xmlui/bitstream/handle/11250/3111674/nguyen-et-al-2023-uncertainty-aware-visually-attentive-navigation-using-deep-neural-networks.pdf?sequence=4&isAllowed=y
28. nvblox: GPU-Accelerated Incremental Signed Distance Field Mapping - arXiv, accessed August 5, 2025, https://arxiv.org/html/2311.00626v2
29. Neural Radiance Fields for the Real World: A Survey - arXiv, accessed August 5, 2025, https://arxiv.org/html/2501.13104v1
30. Benchmarking Neural Radiance Fields for Autonomous Robots: An Overview - arXiv, accessed August 5, 2025, https://arxiv.org/html/2405.05526v1
31. [2304.05735] RO-MAP: Real-Time Multi-Object Mapping with Neural Radiance Fields - arXiv, accessed August 5, 2025, https://arxiv.org/abs/2304.05735
32. [2309.09295] NeRF-VINS: A Real-time Neural Radiance Field Map-based Visual-Inertial Navigation System - arXiv, accessed August 5, 2025, https://arxiv.org/abs/2309.09295
33. NerfBridge: Bringing Real-time, Online Neural Radiance Field Training to Robotics, accessed August 5, 2025, https://www.researchgate.net/publication/370841848_NerfBridge_Bringing_Real-time_Online_Neural_Radiance_Field_Training_to_Robotics
34. Virtual Memory for 3D Gaussian Splatting - arXiv, accessed August 5, 2025, https://arxiv.org/html/2506.19415v1
35. No Redundancy, No Stall: Lightweight Streaming 3D Gaussian Splatting for Real-time Rendering - arXiv, accessed August 5, 2025, https://arxiv.org/html/2507.21572v1
36. GauRast: Enhancing GPU Triangle Rasterizers to Accelerate 3D Gaussian Splatting - arXiv, accessed August 5, 2025, https://arxiv.org/html/2503.16681v1
37. Voyager: Real-Time Splatting City-Scale 3D Gaussians on Your Phone - arXiv, accessed August 5, 2025, https://arxiv.org/html/2506.02774v2
38. Is Hardware Accelerated Min/Max Ray Casting Available with Cuda/Optix? - Stack Overflow, accessed August 5, 2025, https://stackoverflow.com/questions/66978061/is-hardware-accelerated-min-max-ray-casting-available-with-cuda-optix