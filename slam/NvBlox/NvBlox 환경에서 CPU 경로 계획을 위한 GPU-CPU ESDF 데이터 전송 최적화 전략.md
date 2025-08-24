# NvBlox 환경에서 CPU 경로 계획을 위한 GPU-CPU ESDF 데이터 전송 최적화 전략
[NvBLox](./index.md)



NVIDIA Isaac ROS 스위트의 핵심 구성 요소인 NvBlox는 GPU 가속을 통해 실시간 3D 환경 재구성을 수행하며, 경로 계획에 필수적인 Euclidean Signed Distance Field(ESDF)와 같은 중요 데이터 구조를 생성한다.1 그러나 ROS 2 생태계에서 보편적인 패러다임인 CPU 기반 경로 계획기를 채택한 로봇 시스템의 경우, 이 방대한 ESDF 데이터를 GPU 디바이스 메모리에서 CPU 호스트 메모리로 전송하는 과정에서 상당한 지연 시간(latency)이 발생한다. 이 지연 시간은 실시간 반응성을 저해할 수 있는 심각한 성능 병목 현상을 유발한다. 로봇이 동적인 환경에서 신속하고 안전하게 기동하기 위해서는 센서 데이터를 바탕으로 한 환경 인식과 그에 따른 경로 계획 사이의 지연을 최소화해야 하며, GPU와 CPU 간의 데이터 전송 비용은 이러한 요구사항을 충족시키는 데 있어 주요한 기술적 도전 과제이다.


본 보고서는 이 데이터 전송 병목 현상에 대한 철저하고 전문가 수준의 분석을 제공하는 것을 목표로 한다. 먼저 표준 데이터 접근 방식들을 해부하고 전송 비용의 근본적인 원인을 진단한 후, 단계별 해결책을 제시한다. 이러한 해결책은 기존 데이터 흐름에 대한 점진적인 최적화 방안부터, CPU-GPU 하이브리드 경로 계획 아키텍처로의 완전한 패러다임 전환에 이르기까지 광범위한 스펙트럼을 포괄한다. 본 보고서는 단순히 문제 해결 방법을 나열하는 것을 넘어, 각 전략이 왜 효과적인지를 CUDA 아키텍처, PCIe 버스 특성, 메모리 관리 모델의 관점에서 심도 있게 설명하여 독자가 자신의 시스템에 가장 적합한 솔루션을 설계하고 구현할 수 있도록 지원하고자 한다.


본 보고서에서 제안되는 각 전략은 CUDA 아키텍처 및 PCIe 버스 특성에 기반한 기술적 설명으로 뒷받침되며, 실제 적용 가능한 C++ 및 CUDA 코드 예시를 통해 구체화된다. 분석은 기본 구현의 한계를 이해하는 것에서 시작하여, 최첨단 성능 지향 내비게이션 스택을 설계하는 방향으로 독자를 안내할 것이다. 각 단계별 최적화 기법은 이론적 배경과 실제 구현 사이의 간극을 메우고, 성능 측정 및 분석을 통해 각 기법의 효용성을 정량적으로 입증하는 접근법을 취한다.


본 문서는 고급 ROS 2 개발자 및 로보틱스 연구자를 대상으로 한다. 독자는 NvBlox의 기본 설정 및 사용법에 익숙하다고 가정하며, 보고서는 오직 '인식-계획' 데이터 파이프라인의 성능 최적화라는 고급 주제에만 집중한다. 따라서 NvBlox 설치, 기본 지도 구성, ROS 2 기본 개념 등에 대한 설명은 생략하고, GPU-CPU 간 데이터 전송 효율을 극대화하기 위한 심층적인 기술적 논의에 초점을 맞춘다.


NvBlox는 GPU에서 생성된 고밀도 3D ESDF 맵을 외부, 특히 CPU 기반 애플리케이션에서 활용할 수 있도록 여러 인터페이스를 제공한다. 그러나 `isaac_ros_nvblox` 패키지가 제공하는 표준 ROS 2 인터페이스는 기존 생태계(예: Nav2)와의 호환성 및 접근성에 중점을 두고 설계되었기 때문에, 고빈도, 저지연의 전체 3D ESDF 데이터 접근에는 최적화되어 있지 않다. 이러한 표준 인터페이스들은 대부분 2D 슬라이스 데이터만을 제공하거나, 본질적으로 실시간 3D 경로 계획에 부적합한 동기식(synchronous) 요청 모델을 사용한다. 따라서 고성능 3D 경로 계획을 목표로 하는 개발자에게는 이러한 표준 인터페이스의 한계를 명확히 인지하고, 더 직접적인 접근법을 모색하는 것이 필수적이다.


NvBlox가 제공하는 가장 직접적인 3D ESDF 데이터 획득 방법 중 하나는 ROS 2 서비스를 이용하는 것이다. `nvblox_msgs/srv/EsdfAndGradients` 서비스는 클라이언트가 특정 영역의 3D ESDF 데이터를 요청할 수 있는 공식적인 통로를 제공한다.4


`EsdfAndGradients.srv` 서비스 정의 파일을 살펴보면, 클라이언트는 요청(request) 부분에 질의할 영역을 정의하는 Axis-Aligned Bounding Box(AABB) 정보(`aabb_min_m`, `aabb_size_m`)와 좌표계(`frame_id`)를 명시할 수 있다.4 또한, `update_esdf` 플래그를 통해 서비스 호출 시점에 ESDF 맵을 강제로 업데이트하도록 지시할 수 있으며, `visualize_esdf` 플래그로 시각화 토픽 발행을 유도할 수도 있다. 이에 대한 응답(response)으로 서버는 요청된 영역 내의 ESDF 값과 그래디언트(gradient)를 `std_msgs/Float32MultiArray` 형태의 평탄화된 배열로 반환한다. 이 배열과 함께 그리드의 원점(`origin_m`)과 복셀 크기(`voxel_size_m`)가 제공되어 클라이언트가 3D 구조를 재구성할 수 있게 한다.


`nvblox_ros` 패키지의 `nvblox_node`는 이 서비스의 서버 역할을 수행한다. `nvblox_node.cpp` 소스 코드를 분석하면, 서비스 요청이 들어왔을 때의 처리 흐름을 파악할 수 있다.5 노드는 먼저 요청된 `frame_id`가 노드의 `global_frame`과 일치하는지 확인한다. `update_esdf` 플래그가 `true`이면, `processEsdf()` 함수를 호출하여 GPU 상에서 ESDF 레이어를 최신 상태로 업데이트한다. 그 후, `esdf_and_gradients_converter_` 객체를 사용하여 `static_mapper_->esdf_layer()`로부터 요청된 AABB 내의 복셀 데이터를 추출하고, 이를 `Float32MultiArray`로 직렬화(serialize)하여 응답 메시지에 채워 넣는다.


이 서비스 기반 접근 방식은 본질적인 지연 문제를 내포하고 있다. 첫째, ROS 2의 서비스 모델은 근본적으로 동기식 요청-응답(request-response) 구조다. 경로 계획기가 이 서비스를 호출하면, 응답이 도착할 때까지 해당 스레드는 블로킹(blocking)된다. 이 시간 동안 경로 계획기는 어떠한 작업도 수행할 수 없으며, 이는 실시간 시스템에서 치명적인 '멈춤' 현상을 유발한다. 둘째, 데이터 전송 과정에 상당한 오버헤드가 발생한다. GPU 메모리에 있는 방대한 3D 복셀 데이터는 CPU로 복사된 후, ROS 2 메시지 형식으로 직렬화되고, 네트워크 스택을 거쳐 클라이언트에 전달된 후 다시 역직렬화(deserialize)되어야 한다. 이 모든 과정이 동기식 호출 내에서 순차적으로 발생하므로 지연 시간이 누적된다. 개발자 포럼의 논의에서도 이 서비스는 주기적인 업데이트보다는 필요할 때 한 번씩 호출하는(on-demand) 용도에 더 적합함이 언급된다.6


결론적으로 `EsdfAndGradients.srv`는 '풀(pull)' 모델에 해당한다. 즉, 경로 계획기가 명시적으로 데이터를 요청하고 기다려야만 정보를 얻을 수 있다. 하지만 실시간 반응형 경로 계획에는 새로운 데이터가 생성될 때마다 시스템에 의해 능동적으로 전달되는 '푸시(push)' 모델이나 연속적인 스트림 모델이 훨씬 효과적이다. 이 서비스는 비용이 많이 드는 GPU에서 CPU로의 데이터 복사 전체 과정을 동기식 트랜잭션 안에 캡슐화하기 때문에, 고성능을 요구하는 3D 경로 계획 용도로는 근본적인 한계를 가진다. 사용의 편의성과 표준화된 인터페이스라는 장점에도 불구하고, 성능이 중요한 애플리케이션에서는 다른 접근법을 고려해야 하는 이유가 여기에 있다.


ROS 2 서비스 인터페이스의 오버헤드와 블로킹 특성을 피하기 위한 다음 단계는 `nvblox` C++ 라이브러리에 직접 접근하는 것이다. 이를 위해 기존 `nvblox_ros` 노드를 수정하거나, `nvblox::Mapper` 객체를 직접 인스턴스화하고 관리하는 새로운 커스텀 노드를 작성할 수 있다. 이 접근법은 ROS 미들웨어를 우회하여 데이터에 더 가깝게 다가갈 수 있게 해준다.


NvBlox의 핵심 데이터 구조는 `nvblox::Mapper` 클래스에 의해 관리된다.7 이 `Mapper` 객체는 `LayerCake`라는 이름의 컨테이너를 소유하며, `LayerCake`는 TSDF, ESDF, Color 등 여러 유형의 레이어를 함께 묶어 관리한다.8 우리가 필요로 하는 ESDF 데이터는 `EsdfLayer`에 저장되어 있으며, 이는 `VoxelBlockLayer<EsdfVoxel>`의 `typedef`이다. 따라서 `Mapper` 객체에 접근할 수 있다면, `mapper.esdf_layer()`를 통해 `EsdfLayer` 객체에 대한 포인터나 참조를 얻을 수 있다.


`VoxelBlockLayer` 클래스는 CPU에서 복셀 데이터에 접근하기 위한 기본 메소드로 `getVoxels()`를 제공한다.9 이 함수는 특정 3D 좌표 목록을 입력받아 해당 위치의 복셀 데이터를 CPU 메모리로 가져와 벡터 형태로 반환한다. NvBlox 공식 문서는 `getVoxels()` 함수의 내부 동작 과정을 명확히 설명하고 있다.9 이 과정은 여러 단계로 이루어진다:

1. **커널 실행 (Kernel Launch):** GPU에서 CUDA 커널을 실행하여 입력된 쿼리 좌표(3D 위치)를 실제 복셀 데이터가 저장된 메모리 주소로 변환한다. NvBlox는 희소 복셀 그리드(sparse voxel grid) 구조를 사용하므로, 이 과정에는 해시 테이블 조회와 같은 연산이 포함된다.
2. **GPU 내 복사 (Intra-GPU Copy):** 변환된 메모리 주소에서 복셀 데이터를 읽어 GPU 내의 임시 출력 벡터에 복사한다.
3. **GPU-CPU 전송 (Device-to-Host Transfer):** 최종적으로 GPU의 임시 벡터에 담긴 데이터를 PCIe 버스를 통해 CPU의 호스트 메모리로 복사한다. 이 함수를 호출한 CPU 스레드는 이 모든 과정이 완료될 때까지 대기한다.


`getVoxels()` 함수는 ROS 서비스보다는 더 직접적이고 유연하지만, 근본적인 성능 병목 문제는 해결하지 못한다. 이 함수 역시 동기식(synchronous)으로 동작하는 블로킹(blocking) 호출이다. 즉, CPU 스레드는 `getVoxels()` 함수를 호출하는 순간부터 방대한 양의 데이터가 PCIe 버스를 통해 완전히 전송될 때까지 멈춰 서게 된다. 예를 들어, 경로 계획기가 10만 개의 후보 지점에 대한 ESDF 값을 질의한다면, 10만 개의 복셀 데이터가 GPU에서 CPU로 복사되는 동안 계획기는 다음 작업을 진행할 수 없다. 이는 ROS 서비스와 마찬가지로 대규모의 동기식 벌크 데이터 전송이라는 핵심적인 병목 현상을 그대로 내포하고 있다. 따라서 C++ 라이브러리에 직접 접근하는 것은 오버헤드를 줄이는 한 걸음일 뿐, 진정한 실시간 성능을 달성하기 위해서는 데이터 전송 자체의 패러다임을 바꿀 필요가 있다.


| Method (방식)     | Interface (인터페이스)                 | Data Granularity (데이터 단위) | Execution Model (실행 모델) | Pros (장점)                    | Cons (단점)                                        |
| ----------------- | -------------------------------------- | ------------------------------ | --------------------------- | ------------------------------ | -------------------------------------------------- |
| ROS 2 Service     | `nvblox_msgs/srv/EsdfAndGradients`     | AABB-bounded 3D Grid           | 동기식 요청-응답            | 언어에 비종속적, 쉬운 통합     | 높은 지연 시간, 블로킹, 연속적 계획에 부적합       |
| Direct C++ Access | `nvblox::VoxelBlockLayer::getVoxels()` | 임의의 복셀 좌표               | 동기식 함수 호출            | ROS 오버헤드 우회, 높은 유연성 | 여전히 블로킹 방식의 벌크 복사, C++ 노드 수정 필요 |


GPU에서 생성된 ESDF 데이터를 CPU로 가져올 때 발생하는 성능 저하는 단순히 데이터 크기와 하드웨어의 전송 속도 문제로 치부할 수 없다. 그 이면에는 하드웨어의 물리적 한계, 운영체제와 CUDA 런타임이 상호작용하는 메모리 아키텍처, 그리고 데이터 전송을 지시하는 실행 모델이 복잡하게 얽혀 있다. 개발자가 "전송이 느리다"고 느낄 때, 그 원인은 단순한 PCIe 대역폭 부족을 넘어, 눈에 보이지 않는 추가적인 메모리 복사, 각 전송에 따르는 고정된 오버헤드, 그리고 비효율적인 동기화 방식에 있는 경우가 많다. 이 본질적인 원인들을 정확히 진단하는 것이 효과적인 최적화 전략을 수립하는 첫걸음이다.


CPU와 GPU는 일반적으로 마더보드의 PCIe(Peripheral Component Interconnect Express) 슬롯을 통해 연결된다. 이 PCIe 버스는 두 프로세서 간 데이터가 오가는 물리적인 고속도로 역할을 한다.11


PCIe의 성능을 이야기할 때 흔히 '대역폭(bandwidth)'을 주요 지표로 삼는다. PCIe 4.0 x16 인터페이스는 이론적으로 최대 32 GB/s, PCIe 5.0 x16은 64 GB/s에 달하는 높은 대역폭을 제공한다.12 이는 초당 전송할 수 있는 데이터의 총량을 의미한다. 그러나 이 최대 대역폭은 크기가 매우 크고 연속적인 데이터를 한 번에 전송할 때 달성 가능한 이론적 수치에 가깝다. 실제 애플리케이션 성능에 더 큰 영향을 미치는 요소는 '지연 시간(latency)'과 '오버헤드(overhead)'이다. 지연 시간은 데이터 전송을 시작하라는 명령을 내린 후 첫 번째 비트가 목적지에 도달하기까지 걸리는 고정된 시간이다. 아무리 작은 데이터를 보내더라도 이 초기 지연 시간은 반드시 발생한다.


PCIe는 패킷 기반 전송 프로토콜을 사용한다.13 이는 데이터를 정해진 크기의 패킷으로 나누어 보내는 것을 의미하며, 각 패킷에는 데이터 자체 외에 주소, 제어 정보 등을 담은 헤더가 포함된다. 이 헤더가 바로 오버헤드다. 예를 들어, 128바이트 크기의 패킷에 20바이트의 헤더가 붙는다면, 실제 데이터가 차지하는 비율은 약 84%에 불과하다. 이러한 구조 때문에, 많은 수의 작은 데이터 덩어리를 개별적으로 전송하는 것은 하나의 큰 데이터 덩어리를 전송하는 것보다 훨씬 비효율적이다. 각 전송마다 초기 지연 시간과 패킷 헤더 오버헤드가 반복적으로 발생하기 때문이다.13 경로 계획기가 ESDF 맵의 여러 분산된 작은 영역을 개별적으로 요청하는 시나리오를 가정한다면, 이는 최악의 성능을 초래할 수 있다. 따라서 데이터 전송 전략을 수립할 때는 단순히 총 데이터 양뿐만 아니라, 전송 횟수와 각 전송의 크기를 함께 고려하여 오버헤드를 최소화하는 방향으로 설계해야 한다.


CUDA 프로그래밍에서 GPU-CPU 간 데이터 전송 성능을 결정짓는 가장 중요하면서도 종종 간과되는 요소는 호스트(CPU) 메모리의 할당 방식이다. C++에서 `new`나 C에서 `malloc`으로 할당된 일반적인 메모리는 '페이지 가능(pageable)' 메모리다. 반면, CUDA 런타임이 제공하는 특정 함수로 할당된 메모리는 '고정(pinned)' 또는 '페이지 고정(page-locked)' 메모리라고 불린다. 이 둘의 차이는 운영체제의 가상 메모리 관리 방식과 GPU의 DMA(Direct Memory Access) 엔진과의 상호작용에서 비롯되며, 이는 데이터 전송 경로와 속도에 결정적인 영향을 미친다.


페이지 가능 메모리는 운영체제의 가상 메모리 관리자가 필요에 따라 물리적 RAM 내에서 위치를 옮기거나, 심지어 디스크의 스왑 공간으로 내릴 수(paging out) 있는 메모리다.14 이러한 유연성은 시스템 자원을 효율적으로 관리하는 데 필수적이지만, GPU에게는 큰 문제를 야기한다. GPU의 DMA 엔진은 데이터 전송 중에 데이터의 물리적 주소가 변경되지 않는다고 가정하고 동작한다. 만약 DMA 전송이 진행되는 도중에 운영체제가 해당 메모리 페이지를 다른 위치로 옮겨버리면 데이터 무결성이 깨지게 된다.


이러한 위험을 방지하기 위해, CUDA 드라이버는 페이지 가능 메모리와의 데이터 전송 시 안전장치를 마련해 두었다. `cudaMemcpy` 함수가 페이지 가능 호스트 메모리를 소스 또는 목적지로 하여 호출되면, 드라이버는 눈에 보이지 않는 추가 단계를 수행한다.14 먼저, 운영체제가 이동시킬 수 없는 특별한 영역인 '고정된(pinned)' 호스트 메모리 공간에 임시 버퍼, 즉 '스테이징 버퍼(staging buffer)'를 할당한다. 그 다음, CPU를 사용하여 페이지 가능 메모리의 데이터를 이 스테이징 버퍼로 복사한다 (`memcpy`). 마지막으로, GPU의 DMA 엔진이 이 안전한 스테이징 버퍼로부터 데이터를 읽거나 써서 실제 GPU 디바이스 메모리와의 전송을 수행한다. 이 '숨겨진 스테이징 복사' 과정은 전송의 안정성을 보장하지만, 두 번의 복사(Pageable -> Pinned, Pinned -> Device)가 발생하므로 상당한 성능 저하를 유발한다. 개발자는 단 한 줄의 `cudaMemcpy`를 호출했지만, 내부적으로는 CPU 자원을 소모하는 추가 복사가 일어나고 있는 것이다.


고정 메모리는 `cudaHostAlloc` 또는 `cudaMallocHost`와 같은 CUDA API 함수를 통해 할당된다.16 이 함수로 할당된 메모리는 운영체제에게 페이지 아웃 대상이 아님을 명시적으로 알린다. 따라서 해당 메모리의 물리적 주소는 할당 해제될 때까지 고정된다.14 이렇게 주소가 고정된 메모리는 GPU의 DMA 엔진이 어떠한 중간 단계도 없이 직접 접근할 수 있다. 따라서 고정 메모리를 사용하면 앞서 언급된 비효율적인 스테이징 복사 단계를 완전히 제거하고, 호스트와 디바이스 간에 단일 DMA 전송만으로 데이터를 주고받을 수 있다. 이는 PCIe 버스의 최대 대역폭을 활용하기 위한 필수 전제 조건이며, 다음에 설명할 진정한 비동기 전송을 가능하게 하는 기반이 된다.


데이터 전송의 성능은 메모리 할당 방식뿐만 아니라, 전송을 명령하는 함수의 실행 모델에 의해서도 크게 좌우된다. CUDA는 동기식(synchronous)과 비동기식(asynchronous) 두 가지 방식의 메모리 전송 함수를 제공한다.


`cudaMemcpy`는 가장 기본적인 메모리 복사 함수이며, 동기식으로 동작한다.18 '동기식'이라는 것은 이 함수가 호출되면, 함수가 내부적으로 수행하는 모든 작업(앞서 설명한 스테이징 복사를 포함한 전체 데이터 전송)이 완전히 끝날 때까지 CPU 제어 흐름을 반환하지 않음을 의미한다. 즉, `cudaMemcpy` 함수 호출 다음 라인의 코드는 데이터 전송이 100% 완료된 후에야 실행될 수 있다. 이는 실행 파이프라인을 직렬화시킨다. 예를 들어, `(데이터 복사 -> 복사 완료까지 대기) -> (경로 계획 연산 -> 연산 완료까지 대기)`와 같은 순차적인 흐름이 강제되어, CPU나 GPU가 서로를 기다리며 유휴 상태에 빠지는 시간이 길어진다.


`cudaMemcpyAsync`는 이름에서 알 수 있듯이 비동기식으로 동작하는 함수다.18 이 함수는 데이터 전송 작업을 CUDA 드라이버에 '요청'한 후, 전송이 완료되기를 기다리지 않고 즉시 CPU 스레드에 제어권을 반환한다.18 이는 CPU가 데이터 전송이 백그라운드에서 일어나는 동안 다른 계산을 수행하거나 다음 작업을 준비할 수 있게 하여, 연산과 통신의 중첩(overlap)을 가능하게 한다. 이를 통해 시스템의 전체 처리량(throughput)을 극대화할 수 있다.


여기서 가장 중요한 점은 `cudaMemcpyAsync`의 '비동기' 특성이 호스트 메모리가 '고정(pinned)'되었을 때만 완벽하게 보장된다는 것이다.20 만약 `cudaMemcpyAsync`가 페이지 가능 메모리와 함께 사용되면, CUDA 드라이버는 안전한 DMA 전송을 위해 여전히 스테이징 버퍼로의 복사를 수행해야 한다. 이 스테이징 복사 과정은 동기적으로 일어날 수 있으므로, `cudaMemcpyAsync` 호출이 즉시 반환되지 않고 블로킹될 수 있다. 결국, 비동기 함수의 이점을 완전히 상실하게 되는 것이다. 따라서 고성능 데이터 파이프라인을 구축하려는 개발자는 반드시 `cudaHostAlloc`으로 고정 메모리를 할당하고, `cudaMemcpyAsync`를 사용하여 데이터 전송을 스케줄링하는 조합을 사용해야 한다. 이 조합만이 CPU, GPU, DMA 엔진이 동시에 각자의 작업을 수행하는 진정한 병렬 처리를 향한 문을 열어준다.


GPU-CPU 간 데이터 전송의 근본적인 비용 구조를 이해했다면, 이제 이를 최소화하기 위한 구체적인 최적화 전략을 적용할 수 있다. 핵심은 단순히 개별 전송 속도를 높이는 것을 넘어, 데이터 전송이라는 행위 자체를 다른 유용한 작업 뒤에 '숨기는' 파이프라인 아키텍처를 구축하는 것이다. 이는 CUDA 스트림, 고정 메모리, 이중 버퍼링과 같은 고급 기법들을 조합하여 직렬적인 처리 흐름을 병렬적으로 전환함으로써 달성할 수 있다. 이 섹션에서는 이러한 고급 최적화 기법들을 단계별로 심도 있게 다룬다.


가장 강력하고 보편적인 전송 최적화 기법은 생산자-소비자(Producer-Consumer) 모델에 기반한 비동기 파이프라인을 구축하는 것이다. 이 모델에서 NvBlox(GPU)는 ESDF 데이터를 생산하는 '생산자' 역할을, CPU 경로 계획기는 이 데이터를 소모하는 '소비자' 역할을 한다. 목표는 소비자가 이전 프레임의 데이터를 처리하는 동안, 생산자가 다음 프레임의 데이터를 생성하고 이를 백그라운드에서 비동기적으로 전송하여, 소비자의 대기 시간을 거의 0으로 만드는 것이다.


- **CUDA Streams (`cudaStream_t`):** 스트림은 GPU에서 순차적으로 실행되는 독립적인 명령어 큐(queue)다.22 기본(default) 스트림이 아닌, 사용자가 생성한 여러 개의 비-기본(non-default) 스트림에 할당된 작업들은 하드웨어 리소스가 허용하는 한 서로 다른 스트림의 작업과 동시에 실행될 수 있다.23 우리의 파이프라인에서는 최소 두 개의 스트림을 설계한다: 하나는 NvBlox의 ESDF 계산과 같은 GPU 내부 연산을 위한 

  `stream_nvblox`, 다른 하나는 GPU에서 CPU로의 데이터 복사를 전담하는 `stream_copy`다. 이렇게 작업을 분리함으로써 연산과 복사가 서로 간섭 없이 병렬로 진행될 수 있는 기반을 마련한다.

- **Pinned Memory (`cudaHostAlloc`):** 2장에서 강조했듯이, 진정한 비동기 전송을 위해서는 CPU 측 수신 버퍼가 반드시 고정 메모리로 할당되어야 한다.16

  `cudaHostAlloc`을 사용하여 할당된 이 메모리는 DMA 엔진이 직접 접근할 수 있어, `cudaMemcpyAsync`가 즉시 반환되고 백그라운드에서 전송을 수행할 수 있게 한다.14


파이프라인의 끊김 없는 작동을 위해 이중 버퍼링(double buffering) 기법을 도입한다. 이는 CPU 측에 두 개의 동일한 크기의 고정 메모리 버퍼(`cpu_buffer_A`, `cpu_buffer_B`)를 준비하는 것을 의미한다. 파이프라인은 다음과 같이 동작한다:

1. **초기 상태:** NvBlox가 첫 번째 ESDF 맵(Frame 0)을 생성하여 GPU에 가지고 있다. `cudaMemcpyAsync`를 호출하여 이 맵을 `cpu_buffer_A`로 복사하기 시작한다.
2. **사이클 1:**
   - **경로 계획기 (CPU):** `cpu_buffer_A`로의 복사가 완료되기를 기다린다. 완료되면, 이 데이터를 사용하여 경로 계획을 시작한다(소비).
   - **NvBlox (GPU):** CPU가 Frame 0 데이터로 계획을 수행하는 동안, GPU는 동시에 다음 ESDF 맵(Frame 1)을 생성한다(생산).
   - **전송 관리자:** Frame 1 생성이 완료되면, `cudaMemcpyAsync`를 `stream_copy`에 호출하여 Frame 1 데이터를 `cpu_buffer_B`로 복사하기 시작한다.
3. **사이클 2:**
   - **경로 계획기 (CPU):** Frame 0에 대한 계획을 마쳤다. 이제 Frame 1 데이터가 필요하므로, `cpu_buffer_B`로의 복사가 완료될 때까지 기다린다. 완료되면, `cpu_buffer_B`의 데이터를 사용하여 다음 경로 계획을 시작한다.
   - **NvBlox (GPU):** CPU가 Frame 1 데이터로 작업하는 동안, GPU는 Frame 2 데이터를 생성한다.
   - **전송 관리자:** Frame 2 생성이 완료되면, `cudaMemcpyAsync`를 호출하여 Frame 2 데이터를 다시 `cpu_buffer_A`로 복사하기 시작한다.

이러한 방식으로 CPU는 항상 한 버퍼의 데이터를 처리하고 있고, 다른 버퍼에는 다음 데이터가 비동기적으로 채워지고 있다. CPU가 데이터 전송을 기다리는 유일한 시간은 자신의 계산을 마쳤으나 아직 다음 데이터의 전송이 완료되지 않았을 때뿐이며, 계산 시간과 전송 시간을 비슷하게 맞춘다면 이 대기 시간을 거의 없앨 수 있다.


파이프라인의 각 단계를 정밀하게 제어하기 위해 `cudaEvent_t`를 사용한 동기화가 필수적이다.23

`cudaDeviceSynchronize()`는 장치 전체의 모든 작업을 기다리므로 파이프라인의 병렬성을 해칠 수 있다. 대신, `cudaEvent_t`를 사용하면 특정 스트림의 특정 지점까지만 기다릴 수 있다.

- `cudaEventRecord(event, stream_copy)`: `cudaMemcpyAsync` 호출 직후, 복사 스트림(`stream_copy`)에 이벤트 마커를 삽입한다.
- `cudaEventSynchronize(event)`: CPU 스레드(경로 계획기)가 이 이벤트를 기다리도록 한다. 이 함수는 `stream_copy`에서 해당 이벤트가 기록될 때까지, 즉 데이터 복사가 완료될 때까지 CPU 스레드를 블로킹한다. 다른 스트림의 작업에는 영향을 주지 않는다.


이러한 로직을 캡슐화하는 C++ 클래스, 예를 들어 `NvBloxEsdfConsumer`를 설계할 수 있다. 이 클래스는 생성자에서 스트림과 이벤트, 이중 버퍼를 초기화하고, 소멸자에서 이를 해제한다. 내부적으로는 별도의 스레드를 두어 NvBlox로부터 데이터를 받아 비동기 복사를 관리하고, 경로 계획기는 `getLatestEsdfData()`와 같은 간단한 메소드를 호출하여 동기화가 완료된 최신 데이터 버퍼에 대한 포인터를 안전하게 얻을 수 있다. 이 구조는 복잡한 비동기 로직을 추상화하여 경로 계획 코드의 간결성을 유지해준다.17


CPU와 GPU가 물리적으로 동일한 메모리를 공유하는 통합 메모리 아키텍처(Unified Memory Architecture, UMA)를 가진 시스템, 대표적으로 NVIDIA Jetson 플랫폼에서는 데이터 '전송'의 개념이 달라진다. 이러한 시스템에서는 데이터 복사 자체를 없애거나 시스템에 위임하는 고급 메모리 관리 기법을 활용하여 성능을 더욱 향상시킬 수 있다.


'제로카피(Zero-Copy)'는 `cudaHostAlloc`에 `cudaHostAllocMapped` 플래그를 추가하여 할당된 고정 메모리를 통해 구현된다.17 이렇게 할당된 호스트 메모리는 GPU의 가상 주소 공간에도 매핑된다. 

`cudaHostGetDevicePointer()` 함수를 사용하면 이 매핑된 메모리에 대한 GPU 주소를 얻을 수 있으며, 이 주소를 CUDA 커널에 직접 전달하여 GPU가 `cudaMemcpy` 호출 없이 호스트 메모리의 데이터를 직접 읽고 쓸 수 있게 된다.

- **장점:** `cudaMemcpy` API 호출을 제거하여 코드 복잡성을 줄이고, 커널이 필요한 데이터만 직접 접근하므로 전체 맵을 복사할 필요가 없는 경우(예: 희소한 접근 패턴)에 효율적이다.
- **단점:** '제로카피'라는 이름에도 불구하고, 데이터는 여전히 물리적인 버스(NVIDIA 시스템에서는 NVLink 또는 PCIe)를 통해 CPU와 GPU 사이를 오간다. 따라서 메모리 접근은 여전히 버스의 지연 시간에 의해 제한된다. 대량의 데이터를 순차적으로 접근하는 경우, 명시적인 비동기 벌크 복사(`cudaMemcpyAsync`)가 더 높은 처리량을 보일 수 있다.


통합 메모리는 `cudaMallocManaged`를 통해 할당되며, CPU와 GPU 모두에서 접근 가능한 단일 포인터를 제공한다.16 개발자는 더 이상 `cudaMemcpy`를 사용하여 데이터를 명시적으로 전송할 필요가 없다. CUDA 시스템이 데이터에 접근하는 프로세서(CPU 또는 GPU)를 추적하여, 필요할 때 자동으로 해당 데이터를 적절한 위치로 마이그레이션(migration)한다.

- **장점:** 프로그래밍 모델을 극적으로 단순화시킨다. 복잡한 메모리 관리 코드가 사라져 생산성이 향상된다.
- **단점:** 데이터 마이그레이션이 암시적으로, 페이지 폴트(page fault) 시점에 발생하므로 성능이 예측 불가능할 수 있다. 특히 실시간 시스템에서는 예기치 않은 지연을 유발할 수 있다.


통합 메모리의 예측 불가능성을 완화하기 위해 `cudaMemPrefetchAsync` 함수를 사용할 수 있다.26 이 함수는 특정 메모리 영역이 곧 특정 프로세서에서 사용될 것임을 CUDA 드라이버에 미리 알려주는 '힌트' 역할을 한다. 드라이버는 이 힌트를 받아 데이터를 비동기적으로 미리 이동시켜, 실제 데이터 접근 시에 발생하는 페이지 폴트와 마이그레이션 지연을 숨길 수 있다. 예를 들어, 경로 계획기가 곧 ESDF 데이터에 접근할 것임을 알고 있다면, `cudaMemPrefetchAsync(esdf_ptr, size, cudaCpuDeviceId, stream)`를 호출하여 데이터를 CPU로 미리 가져올 수 있다.


이러한 기법들은 CPU와 GPU가 메모리를 공유하는 Jetson과 같은 임베디드 SoC(System on Chip)에서 가장 큰 효과를 발휘한다. 별도의 외장 GPU를 사용하는 데스크톱 시스템에서는 CPU와 GPU 메모리가 물리적으로 분리되어 있으므로, 데이터는 반드시 PCIe 버스를 통해 전송되어야 한다. 이 경우, 제로카피는 여전히 PCIe를 통한 접근이며, 통합 메모리는 PCIe를 통한 페이지 마이그레이션을 의미하므로, 3.1절에서 설명한 비동기 파이프라인 방식이 일반적으로 더 예측 가능하고 높은 성능을 제공한다.


| Strategy (전략)                | Key Primitives (주요 함수)                         | Performance Gain (성능 향상)      | Implementation Complexity (구현 복잡도) |
| ------------------------------ | -------------------------------------------------- | --------------------------------- | --------------------------------------- |
| Synchronous Copy (Baseline)    | `cudaMemcpy`                                       | 낮음                              | 낮음                                    |
| Pinned Memory Copy             | `cudaHostAlloc` + `cudaMemcpy`                     | 중간                              | 낮음                                    |
| Async Pipeline (Double Buffer) | `cudaMemcpyAsync` + `cudaStream_t` + `cudaEvent_t` | 높음                              | 높음                                    |
| Zero-Copy (Mapped Memory)      | `cudaHostAllocMapped` + Kernel Access              | 중간-높음 (사용 사례에 따라 다름) | 중간                                    |


지금까지의 최적화 전략들은 GPU와 CPU 간의 데이터 전송 '비용'을 줄이거나 숨기는 데 초점을 맞추었다. 그러나 가장 근본적이고 강력한 해결책은 데이터 전송이라는 '문제 자체'를 제거하는 것이다. 이는 경로 계획 알고리즘의 가장 데이터 집약적인 부분을 CPU에서 GPU로 옮겨, 방대한 ESDF 맵이 애초에 PCIe 버스를 건너지 않도록 만드는 패러다임의 전환을 의미한다. CPU와 GPU 간의 통신은 거대한 맵 데이터가 아닌, 소량의 경로 후보군과 그에 대한 충돌 검사 결과로 축소되어 병목의 본질을 근본적으로 바꾼다.


전통적인 로보틱스 파이프라인은 "인식(Sense) -> 계획(Plan) -> 행동(Act)"의 순차적인 모듈로 구성되는 경우가 많다. 각 모듈은 독립적인 블랙박스로 작동하며, 그 사이를 거대한 중간 데이터 구조(예: 포인트 클라우드, 점유 격자, ESDF 맵)가 오가며 통신한다. 이러한 아키텍처는 모듈화와 재사용성 측면에서 장점이 있지만, 데이터 이동 자체가 성능 병목이 되는 현대의 고성능 시스템에서는 구조적인 결함이 될 수 있다.

가장 최적화된 비동기 파이프라인(3.1절)을 사용하더라도, 여전히 상당한 양의 데이터가 전송된다. 예를 들어, 20m x 20m x 2m 크기의 공간을 5cm 복셀 해상도로 표현하는 ESDF 맵은 약 640만 개(400×400×40)의 복셀을 가진다. 각 복셀이 4바이트(float) 값을 가진다고 가정하면, 프레임당 약 25.6 MB의 데이터가 전송되어야 한다. 만약 이를 30Hz로 처리한다면 초당 768 MB/s의 대역폭이 필요한데, 이는 최신 PCIe 버스라 할지라도 상당한 부담이며 CPU 메모리 대역폭에도 영향을 미친다.

여기서 발상의 전환이 필요하다. CPU 기반 경로 계획기는 ESDF 맵 전체의 구조를 '이해'할 필요가 없다. 계획기가 실제로 필요로 하는 정보는 "이 로봇 자세 q는 충돌 없이 안전한가?" 또는 "이 경로 세그먼트는 장애물과 충돌하는가?"와 같은 질의(query)에 대한 답변이다.27 이 질의에 대한 응답은 ESDF 맵이 존재하는 GPU 상에서 훨씬 효율적으로 계산될 수 있다. 특히 RRT*나 최적화 기반 플래너와 같은 샘플링 기반 알고리즘들은 한 번의 계획 사이클 동안 수백에서 수천 개의 이러한 질의를 생성하는데, 이는 본질적으로 데이터 병렬적인(data-parallel) 작업이므로 GPU 아키텍처에 매우 적합하다.28

이러한 통찰은 다음과 같은 하이브리드 아키텍처로 이어진다 29:

- **CPU (전략가, The Strategist):** RRT* 트리의 확장, 그래프 탐색, 최적화 문제의 상위 레벨 로직과 같은 복잡하고 순차적인 작업을 담당한다. 그리고 한 번에 평가할 대량의 후보 상태(configurations) 또는 경로들의 '배치(batch)'를 생성한다.
- **GPU (검증자, The Validator):** CPU로부터 작은 크기의 후보 배치 데이터를 받는다. 그리고 GPU 메모리에 상주하는 전체 ESDF 레이어에 대해 모든 후보들을 병렬적으로 충돌 검사하는 커스텀 CUDA 커널을 실행한다.

이 구조에서 데이터 전송은 근본적으로 변환된다. 수십 메가바이트의 맵을 GPU에서 CPU로 전송하는 대신, 수 킬로바이트의 쿼리 데이터를 CPU에서 GPU로, 그리고 수 킬로바이트의 결과 데이터를 GPU에서 CPU로 전송하게 된다. 이는 전송 데이터 양을 수백에서 수천 배 줄일 수 있는 혁신적인 접근법이다. 이 방식은 커스텀 CUDA 커널 작성이라는 추가적인 개발 노력을 요구하지만, 문제의 근본 원인을 직접 해결함으로써 최고의 성능을 제공한다. 이는 cuRobo와 같은 최신 로보틱스 라이브러리들이 채택하고 있는 방향과도 일치한다.31


하이브리드 아키텍처의 핵심은 GPU에서 실행되는 효율적인 배치(batch) 충돌 검사 커널이다. 이 커널은 다수의 경로 후보를 한 번의 호출로 동시에 검증하여 GPU의 병렬 처리 능력을 극대화한다.


GPU 커널을 한 번 실행하는 데는 고정된 오버헤드(launch overhead)가 발생한다. 따라서 후보 지점을 하나씩 검사하는 것은 매우 비효율적이다. 수백 또는 수천 개의 후보 지점이나 경로 세그먼트를 하나의 배치로 묶어 커널에 전달함으로써, 실행 오버헤드를 분산시키고 전체 처리량을 극적으로 높일 수 있다.


GPU 커널 내에서 NvBlox의 ESDF 데이터에 접근하기 위해서는 `nvblox::VoxelBlockLayer` 클래스가 제공하는 `getGpuLayerView()` 메소드를 사용해야 한다.9 이 메소드는 `nvblox::GPULayerView`라는 특별한 객체를 반환한다. 이 뷰 객체는 어떠한 메모리 복사도 없이, CUDA 커널 내에서 복셀 데이터에 직접 접근하는 데 필요한 모든 정보(예: 복셀 블록을 찾기 위한 해시 테이블 포인터, 블록 데이터 포인터 등)를 담고 있다. 이는 커널이 GPU 메모리에 상주하는 ESDF 맵을 직접, 그리고 매우 빠르게 조회할 수 있게 해준다.


배치 충돌 검사를 위한 커스텀 CUDA 커널은 다음과 같이 설계될 수 있다 33:

- **입력 (Input):**
  - `GPULayerView` 객체로부터 얻은 ESDF 레이어 정보.
  - CPU로부터 복사된, 검사할 3D 좌표들의 배열 (`const float* query_positions`).
  - 로봇의 반경 또는 안전 마진 (`float safety_margin`).
  - 배치의 크기 (`int num_queries`).
- **출력 (Output):**
  - 각 쿼리에 대한 충돌 여부를 저장할 불리언(boolean) 또는 정수 배열 (`bool* results`).
- **로직 (Logic):**
  - 커널은 `num_queries` 만큼의 스레드로 실행된다. CUDA의 실행 모델에 따라 각 스레드는 고유한 인덱스(`threadIdx.x + blockIdx.x * blockDim.x`)를 부여받는다.
  - 각 스레드는 자신의 인덱스를 사용하여 `query_positions` 배열에서 검사할 3D 좌표 하나를 담당한다.
  - 스레드는 `GPULayerView`와 `getVoxelAtPosition()`과 같은 NvBlox의 내부 헬퍼 함수를 사용하여, 담당하는 3D 좌표에 해당하는 ESDF 복셀 값을 GPU 메모리에서 직접 조회한다.
  - 조회한 ESDF 값(장애물까지의 거리)이 `safety_margin`보다 작거나 같으면 충돌로 간주하고, `results` 배열의 해당 인덱스에 `true`를 기록한다. 그렇지 않으면 `false`를 기록한다.

이러한 커널을 C++ 호스트 코드에서 호출하는 과정은 다음과 같다:

1. GPU에 쿼리 좌표와 결과 배열을 위한 메모리(`d_query_positions`, `d_results`)를 `cudaMalloc`으로 할당한다.
2. CPU에서 생성된 쿼리 배치(`h_query_positions`)를 `cudaMemcpy` (또는 `cudaMemcpyAsync`)를 사용하여 `d_query_positions`로 복사한다.
3. `nvblox_node`로부터 `getGpuLayerView()`를 호출하여 ESDF 뷰를 얻는다.
4. 설계된 커스텀 커널을 실행 구성(execution configuration)과 함께 호출한다: `batchCollisionCheck<<<grid_size, block_size>>>(...)`.
5. 커널 실행이 완료된 후, `cudaMemcpy`를 사용하여 `d_results`를 CPU의 `h_results` 배열로 복사해온다.
6. `cudaFree`로 GPU 메모리를 해제한다.


앞서 설명한 개념들을 종합하여 구체적인 하이브리드 경로 계획 아키텍처를 제안할 수 있다. 이 아키텍처는 CPU와 GPU의 역할을 명확히 분리하여 각자의 강점을 최대한 활용한다.

- **CPU Role (The Strategist):**

  - 전역 탐색 알고리즘(A*, RRT* 등)의 주 로직을 실행한다. 예를 들어, RRT*의 경우 트리를 관리하고, 새로운 랜덤 샘플을 생성하며, 어떤 노드를 확장할지 결정한다.
  - 확장할 노드 주변에 다수의 후보 노드(자식 노드)를 생성하거나, 최적화 기반 플래너의 경우 여러 후보 궤적을 생성하여 '평가 배치'를 구성한다.
  - 이 평가 배치를 GPU로 전송하고, GPU로부터 충돌 검사 결과를 수신한다.
  - 충돌하지 않는(collision-free) 후보들만을 사용하여 트리를 확장하거나 궤적을 갱신하는 등, 탐색 구조를 업데이트한다.

- **GPU Role (The Validator):**

  - NvBlox를 통해 지속적으로 센서 데이터를 통합하고, 최신의 고해상도 3D ESDF 맵을 GPU 메모리 내에서 유지 및 업데이트한다.
  - CPU로부터 평가 배치를 수신하면, `batchCollisionCheck` 커널을 실행하여 모든 후보를 병렬로 검사한다.
  - 결과(충돌 여부 배열)를 CPU로 다시 전송한다.

- Data Flow:

  이 아키텍처에서 PCIe 버스를 통해 흐르는 데이터는 매우 작고 간결하다.

  - **CPU -> GPU:** 수백~수천 개의 3D 좌표 또는 경로 파라미터 (수 KB ~ 수십 KB).

  - GPU -> CPU: 수백~수천 개의 불리언 값 (수 KB).

    수십 메가바이트에 달하는 ESDF 맵 전체는 경로 계획 과정에서 절대 PCIe 버스를 건너지 않는다. 이로써 데이터 전송 병목은 사실상 제거되고, 시스템의 성능 한계는 커널 실행 오버헤드나 GPU의 계산 능력(occupancy)에 의해 결정된다.


| Architecture (아키텍처)   | Planning Locus (계획 주체) | Data Transferred (전송 데이터) | Primary Bottleneck (주요 병목점) | Suitability (적합성)                   |
| ------------------------- | -------------------------- | ------------------------------ | -------------------------------- | -------------------------------------- |
| CPU-Centric (데이터 전송) | CPU                        | 전체 ESDF 맵                   | PCIe 대역폭/지연 시간            | 간단한 플래너, 레거시 시스템           |
| Hybrid (연산 이동)        | CPU (전략) + GPU (검증)    | 쿼리/결과 배치                 | 커널 실행 오버헤드               | 고성능 샘플링/최적화 기반 플래너       |
| Full GPU (e.g., cuRobo)   | GPU                        | 목표 자세 / 최종 경로          | GPU 점유율(Occupancy)            | 종단간(End-to-end) GPU 네이티브 시스템 |



본 보고서는 ROS 2 Humble 환경에서 NvBlox가 생성한 GPU 기반 ESDF 맵을 CPU 경로 계획에 활용할 때 발생하는 데이터 전송 병목 현상을 해결하기 위한 심층적인 분석과 단계별 최적화 전략을 제시했다. 분석을 통해 다음과 같은 핵심 결론에 도달했다.

첫째, `isaac_ros_nvblox`가 제공하는 표준 ROS 2 서비스(`EsdfAndGradients.srv`)나 C++ 라이브러리의 기본 접근 함수(`getVoxels()`)는 동기식 벌크 데이터 전송에 의존하므로, 실시간 3D 경로 계획에 필요한 저지연, 고빈도 데이터 접근 요구사항을 충족시키지 못한다. 이는 구조적인 한계로, 단순 사용만으로는 성능 문제를 피할 수 없다.

둘째, 데이터 전송 비용의 근본 원인은 PCIe 대역폭 문제뿐만 아니라, 페이지 가능 메모리 사용 시 발생하는 숨겨진 스테이징 복사, 전송 자체의 고정된 지연 시간 오버헤드, 그리고 동기식 실행 모델에 있다. 따라서 고정(pinned) 메모리 할당과 비동기 전송 함수(`cudaMemcpyAsync`)의 사용은 성능 개선의 필수적인 첫걸음이다.

셋째, 이중 버퍼링과 CUDA 스트림, 이벤트를 활용한 비동기 전송 파이프라인을 구축하는 것은 데이터 전송 지연 시간을 다른 유용한 연산 뒤에 숨길 수 있는 강력한 최적화 기법이다. 이 방식은 상당한 성능 향상을 가져올 수 있지만, 여전히 대량의 데이터를 PCIe 버스를 통해 전송해야 한다는 근본적인 제약은 남아있다.

넷째, 가장 효과적이고 궁극적인 해결책은 패러다임을 전환하여 데이터가 아닌 연산을 이동시키는 것이다. 경로 계획의 가장 데이터 집약적인 부분인 충돌 검사를 GPU에서 수행하는 하이브리드 CPU-GPU 아키텍처는 데이터 전송량을 수백 배 이상 줄여 병목 현상을 원천적으로 제거한다. 이는 추가적인 CUDA 커널 개발을 요구하지만, 최고의 성능을 달성할 수 있는 가장 확실한 방법이다.


개발자는 자신의 시스템 요구사항과 개발 리소스에 맞춰 적절한 전략을 선택해야 한다. 다음의 의사결정 가이드는 그 선택을 돕기 위해 제시된다.

1. **현재 시스템의 성능 병목이 정말 GPU-CPU 데이터 전송인가?**
   - NVIDIA Nsight Systems와 같은 프로파일링 도구를 사용하여 `cudaMemcpy` 관련 함수의 실행 시간을 측정하고 전체 파이프라인에서 차지하는 비중을 확인하라. 만약 이 비중이 크다면 다음 단계로 진행한다.
2. **요구되는 경로 계획의 지연 시간은 어느 정도인가?**
   - **수십~수백 밀리초(ms) 허용:** 정적인 환경이나 비-실시간 계획의 경우, `EsdfAndGradients` 서비스를 간헐적으로 호출하거나, `getVoxels()`를 사용한 동기식 접근으로도 충분할 수 있다.
   - **10~30ms 수준의 반응성 필요:** 동적인 환경에서의 기본적인 회피 기동을 위해서는 **비동기 전송 파이프라인(3.1절)** 도입을 강력히 권장한다. 고정 메모리 할당과 이중 버퍼링 구현은 필수적이다.
   - **수 밀리초(ms) 이하의 초저지연 필요:** 고속 기동, 복잡한 조작(manipulation) 등 극한의 성능이 요구된다면, **하이브리드 경로 계획 아키텍처(4장)** 외에는 대안이 없다.
3. **대상 하드웨어 플랫폼은 무엇인가?**
   - **NVIDIA Jetson (통합 메모리):** **제로카피 및 통합 메모리(3.2절)** 기법을 우선적으로 고려해볼 수 있다. 프로그래밍이 간편하고 특정 시나리오에서 효과적일 수 있다. 그러나 예측 불가능한 성능이 우려된다면, 비동기 파이프라인이 더 안정적인 선택이다.
   - **데스크톱/서버 (외장 GPU):** 물리적으로 메모리가 분리되어 있으므로, **비동기 전송 파이프라인(3.1절)** 이나 **하이브리드 아키텍처(4장)** 가 주된 선택지다.
4. **개발팀의 CUDA 프로그래밍 역량은 어떠한가?**
   - **CUDA 경험 부족:** 먼저 고정 메모리(`cudaHostAlloc`)를 적용하여 스테이징 복사를 제거하는 것부터 시작하라. 이것만으로도 상당한 성능 개선을 볼 수 있다.
   - **CUDA 중급:** 비동기 파이프라인 구축에 도전하라. 스트림과 이벤트 관리에 대한 이해가 필요하지만, 잘 구현하면 큰 성능 향상을 얻을 수 있다.
   - **CUDA 고급:** 하이브리드 아키텍처를 위한 커스텀 충돌 검사 커널을 직접 설계하고 구현하라. 이는 가장 높은 수준의 전문성을 요구하지만, 시스템의 잠재력을 최대한으로 이끌어낼 수 있다.


본 보고서에서 제안한 하이브리드 아키텍처는 로보틱스 분야의 거대한 흐름과 맥을 같이 한다. 업계는 점차적으로 인식, 계획, 제어의 더 많은 부분을 GPU 네이티브 환경으로 옮겨가는 추세다.38 NVIDIA의 Isaac Sim, cuRobo, Isaac Perceptor와 같은 플랫폼들은 GPU의 병렬 처리 능력을 최대한 활용하여 종단간(end-to-end) 파이프라인을 가속화하는 것을 목표로 한다.32 이러한 완전한 GPU 네이티브 스택은 CPU와 GPU 간의 데이터 이동을 최소화하여 지연 시간을 줄이고 전체 시스템의 처리량을 극대화한다. 본 보고서에서 제안한 하이브리드 경로 계획 아키텍처는 이러한 미래 지향적인 완전 GPU 네이티브 시스템으로 나아가는 실용적이고 강력한 중간 단계라 할 수 있다. 개발자들은 이 아키텍처를 구현함으로써 현재의 성능 병목을 해결할 뿐만 아니라, 미래의 로보틱스 소프트웨어 패러다임에 대한 귀중한 경험과 통찰을 얻게 될 것이다.


1. Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox - GitHub, accessed August 7, 2025, https://github.com/Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox
2. Nvblox - isaac_ros_docs documentation - NVIDIA Isaac ROS, accessed August 7, 2025, https://nvidia-isaac-ros.github.io/concepts/scene_reconstruction/nvblox/index.html
3. Technical Details - isaac_ros_docs documentation - NVIDIA Isaac ROS, accessed August 7, 2025, https://nvidia-isaac-ros.github.io/concepts/scene_reconstruction/nvblox/technical_details.html
4. isaac_ros_nvblox/nvblox_msgs/srv/EsdfAndGradients.srv at main - GitHub, accessed August 7, 2025, https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_nvblox/blob/main/nvblox_msgs/srv/EsdfAndGradients.srv
5. isaac_ros_nvblox/nvblox_ros/src/lib/nvblox_node.cpp at main / NVIDIA-ISAAC-ROS ... - GitHub, accessed August 7, 2025, https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_nvblox/blob/main/nvblox_ros/src/lib/nvblox_node.cpp
6. How to enable periodic ESDF updates in nvblox ？ - Isaac ROS - NVIDIA Developer Forums, accessed August 7, 2025, https://forums.developer.nvidia.com/t/how-to-enable-periodic-esdf-updates-in-nvblox/331162
7. nvblox::Mapper Class Reference, accessed August 7, 2025, https://nvblox.readthedocs.io/en/public/classnvblox_1_1Mapper.html
8. map.md - nvidia-isaac/nvblox - GitHub, accessed August 7, 2025, https://github.com/nvidia-isaac/nvblox/blob/public/docs/pages/map.md
9. nvblox/docs/pages/tutorial_library_interface.md at public - GitHub, accessed August 7, 2025, https://github.com/nvidia-isaac/nvblox/blob/public/docs/pages/tutorial_library_interface.md
10. Class Members - Functions - nvblox Documentation, accessed August 7, 2025, https://nvblox.readthedocs.io/en/public/functions_func_g.html
11. PCI Express - Wikipedia, accessed August 7, 2025, https://en.wikipedia.org/wiki/PCI_Express
12. Impact of PCIe 5.0 Bandwidth on GPU Content Creation Performance | Puget Systems, accessed August 7, 2025, https://www.pugetsystems.com/labs/articles/impact-of-pcie-5-0-bandwidth-on-gpu-content-creation-performance/
13. Why is the transfer throughput low when transferring small size data from Host to Device (or Device to Host)? - CUDA Programming and Performance - NVIDIA Developer Forums, accessed August 7, 2025, https://forums.developer.nvidia.com/t/why-is-the-transfer-throughput-low-when-transferring-small-size-data-from-host-to-device-or-device-to-host/153962
14. Module 14 – Efficient Host-Device Data Transfer - Purdue Engineering, accessed August 7, 2025, https://engineering.purdue.edu/~smidkiff/ece563/NVidiaGPUTeachingToolkit/Mod14DataXfer/Mod14DataXfer.pdf
15. AMS 148 Chapter 8: Optimization in CUDA, and Advanced Topics, accessed August 7, 2025, https://ams148-spring18-01.courses.soe.ucsc.edu/system/files/attachments/note8_0.pdf
16. CUDA Runtime API - 6.10. Memory Management - NVIDIA Docs Hub, accessed August 7, 2025, https://docs.nvidia.com/cuda/cuda-runtime-api/group__CUDART__MEMORY.html
17. Memories from CUDA - Pinned memory (III) - Code of Honour, accessed August 7, 2025, http://codeofhonour.blogspot.com/2014/10/memories-from-cuda-pinned-memory-iii.html
18. What is the difference between cudaMemcpy and cudaMemcpyAsync in CUDA?, accessed August 7, 2025, [https://massedcompute.com/faq-answers/?question=What%20is%20the%20difference%20between%20cudaMemcpy%20and%20cudaMemcpyAsync%20in%20CUDA?](https://massedcompute.com/faq-answers/?question=What+is+the+difference+between+cudaMemcpy+and+cudaMemcpyAsync+in+CUDA?)
19. What is the difference between CUDA's cudaMemcpyAsync and cudaMemcpy?, accessed August 7, 2025, [https://massedcompute.com/faq-answers/?question=What%20is%20the%20difference%20between%20CUDA%27s%20cudaMemcpyAsync%20and%20cudaMemcpy?](https://massedcompute.com/faq-answers/?question=What+is+the+difference+between+CUDA's+cudaMemcpyAsync+and+cudaMemcpy?)
20. When is safe to reuse CPU buffer when calling cudaMemcpyAsync? - Stack Overflow, accessed August 7, 2025, https://stackoverflow.com/questions/16060132/when-is-safe-to-reuse-cpu-buffer-when-calling-cudamemcpyasync
21. Multi-GPU programming with CUDA. A complete guide to NVLink. | GPGPU - Medium, accessed August 7, 2025, https://medium.com/gpgpu/multi-gpu-programming-with-cuda-6768eeb42e2c
22. How to Overlap Data Transfers in CUDA C/C++ | NVIDIA Technical Blog, accessed August 7, 2025, https://developer.nvidia.com/blog/how-overlap-data-transfers-cuda-cc/
23. CUDA C/C++ Streams and Concurrency - CSE, IIT Delhi, accessed August 7, 2025, https://www.cse.iitd.ac.in/~rijurekha/col730_2022/cudastreams_aug25_aug29.pdf
24. code-samples/series/cuda-cpp/overlap-data-transfers/async.cu at master - GitHub, accessed August 7, 2025, https://github.com/NVIDIA-developer-blog/code-samples/blob/master/series/cuda-cpp/overlap-data-transfers/async.cu
25. cudaMemcpyAsync - CUDA Programming and Performance - NVIDIA Developer Forums, accessed August 7, 2025, https://forums.developer.nvidia.com/t/cudamemcpyasync/38061
26. CUDA Series: Memory and Allocation | by Dmitrij Tichonov - Medium, accessed August 7, 2025, https://medium.com/@dmitrijtichonov/cuda-series-memory-and-allocation-fce29c965d37
27. CollisionGP: Gaussian Process-Based Collision Checking for Robot Motion Planning, accessed August 7, 2025, [https://elib.dlr.de/196266/1/RALpaperCollisionGP%20VF_copyright.pdf](https://elib.dlr.de/196266/1/RALpaperCollisionGP VF_copyright.pdf)
28. GPU-based Parallel Collision Detection for Real-Time Motion Planning - GAMMA, accessed August 7, 2025, http://gamma.cs.unc.edu/gplanner/WAFR.pdf
29. A Hybrid Architecture for Safe Human–Robot Industrial Tasks - MDPI, accessed August 7, 2025, https://www.mdpi.com/2076-3417/15/3/1158
30. HyP-DESPOT: A Hybrid Parallel Algorithm for Online Planning under Uncertainty - Robotics, accessed August 7, 2025, https://www.roboticsproceedings.org/rss14/p04.pdf
31. (PDF) GPU-Accelerated Collision-Free Path Planning for Multi- Axis Robots in Construction Automation - ResearchGate, accessed August 7, 2025, https://www.researchgate.net/publication/394014572_GPU-Accelerated_Collision-Free_Path_Planning_for_Multi-_Axis_Robots_in_Construction_Automation
32. Motion Planning with Nvidia cuRobo and ROS2 | by Gaurav Gupta | Black Coffee Robotics, accessed August 7, 2025, https://medium.com/black-coffee-robotics/motion-planning-with-nvidia-curobo-and-ros2-4b16fa6b27a9
33. Custom Cuda Kernel Samples - NVIDIA Docs Hub, accessed August 7, 2025, https://docs.nvidia.com/holoscan/sdk-user-guide/examples/custom_cuda_kernel_sample.html
34. DeepLearnPhysics Blog – Writing your own CUDA kernel (Part 1), accessed August 7, 2025, http://stanford.edu/~ldomine/2018-10-02-Writing-your-own-CUDA-kernel-1.html
35. Chapter 33. LCP Algorithms for Collision Detection Using CUDA - NVIDIA Developer, accessed August 7, 2025, https://developer.nvidia.com/gpugems/gpugems3/part-v-physics-simulation/chapter-33-lcp-algorithms-collision-detection-using-cuda
36. Writing CUDA Kernels for PyTorch - Tinkerd, accessed August 7, 2025, https://tinkerd.net/blog/machine-learning/cuda-basics/
37. Custom C++ and CUDA Operators - PyTorch Tutorials 2.7.0+cu126 documentation, accessed August 7, 2025, https://docs.pytorch.org/tutorials/advanced/cpp_custom_ops.html
38. Isaac ROS (Robot Operating System) - NVIDIA Developer, accessed August 7, 2025, https://developer.nvidia.com/isaac/ros
39. NVIDIA Isaac ROS in under 5 minutes - Intermodalics, accessed August 7, 2025, https://www.intermodalics.ai/blog/nvidia-isaac-ros-in-under-5-minutes
40. 09 Hardware Acceleration in ROS 2 Faster ROS withFPGAs GPUs and other accelerators, accessed August 7, 2025, https://www.youtube.com/watch?v=celGdC48NzU