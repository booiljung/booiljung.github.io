[VDB](./index.md)
# NanoVDB


현대 컴퓨터 그래픽스, 특히 시각 효과(VFX)와 실시간 게임 분야에서 연기, 구름, 불, 물과 같은 볼류메트릭(volumetric) 효과의 사실적인 표현은 몰입감을 결정하는 핵심 요소로 자리 잡았다.1 이러한 복잡한 효과를 효율적으로 저장하고 처리하기 위해, 아카데미상을 수상한 오픈소스 라이브러리인 OpenVDB는 업계의 표준으로 확고히 자리매김했다.1 그러나 OpenVDB는 본질적으로 CPU의 다중 스레딩 환경에 최적화된 아키텍처를 가지고 있어, 대규모 병렬 처리 능력을 갖춘 GPU를 활용한 실시간 렌더링 및 상호작용에는 근본적인 한계를 보여왔다.5 OpenVDB의 동적인 트리 구조와 포인터 기반의 메모리 접근 방식, 그리고 여러 외부 라이브러리에 대한 의존성은 GPU로의 직접적인 이식과 효율적인 실행을 저해하는 주요 장벽이었다.5

이러한 기술적 공백을 메우고 GPU의 잠재력을 최대한 활용하기 위해, NVIDIA의 Ken Museth(OpenVDB의 원개발자) 주도로 새로운 데이터 구조인 NanoVDB가 개발되었다.1 NanoVDB는 OpenVDB의 핵심 철학인 희소 볼륨(sparse volume) 표현을 계승하면서도, GPU 아키텍처에 완벽하게 부합하도록 재설계된 경량의 읽기 전용(read-only) 데이터 구조이다.8 이는 OpenVDB 구조의 특정 시점 상태를 GPU 친화적인 형태로 직렬화한 '선형화된 스냅샷(linearized snapshot)'으로 정의할 수 있다.7

본 보고서는 NanoVDB의 핵심 아키텍처 원리를 심층적으로 분석하고, OpenVDB와의 구조적 및 성능적 차이를 정량적으로 비교한다. 또한, 주요 소프트웨어에서의 구현 사례와 다양한 산업 분야에서의 응용을 고찰하며, VDB 생태계가 NeuralVDB로 확장되는 기술 발전의 맥락 속에서 NanoVDB가 가지는 기술사적 의의와 미래 가치를 평가하는 것을 목적으로 한다.


NanoVDB의 혁신적인 성능은 GPU의 병렬 아키텍처에 대한 깊은 이해를 바탕으로 한 설계 철학에서 비롯된다. '선형화된 포인터-리스 구조'와 '정적 토폴로지'라는 두 가지 핵심 원칙은 동적 유연성을 희생하는 대신 GPU에서의 압도적인 읽기 성능을 확보하기 위한 전략적 선택이다.


NanoVDB 아키텍처의 가장 근본적인 특징은 포인터를 완전히 배제하고 전체 트리 구조를 단일의 연속적인(contiguous) 메모리 블록에 직렬화(serialize)하여 저장하는 것이다.5 이는 노드 간의 연결을 위해 포인터를 사용함으로써 메모리상에 데이터가 단편화(fragmented)될 수 있는 OpenVDB와 근본적으로 다른 접근 방식이다.7 NanoVDB가 '선형화된 스냅샷'으로 불리는 이유가 바로 여기에 있다.10

이 연속적인 메모리 블록은 GPU 메모리 아키텍처에 최적화되도록 세심하게 설계되었다. 모든 노드와 데이터는 32바이트 경계에 정렬(32-byte aligned)되어 있는데 7, 이는 많은 GPU 아키텍처의 메모리 트랜잭션 단위와 일치하여 접근 효율을 극대화한다. 메모리 버퍼는 일반적으로 `GridData`, `TreeData`, `RootData`, `InternalNode` 배열, 그리고 `LeafNode` 배열 순으로 배치되어 예측 가능한 구조를 가진다.11

이러한 포인터-리스 구조는 GPU 활용에 결정적인 이점을 제공한다. CPU 메모리에 있는 NanoVDB 데이터를 GPU로 전송할 때, 복잡한 포인터 재연결(pointer patching) 과정 없이 `cudaMemcpy`와 같은 단순한 메모리 복사 연산만으로 전체 데이터 구조를 한 번에 이전할 수 있다.10 이는 데이터 전송에 따르는 오버헤드를 획기적으로 줄여, 실시간 렌더링 및 상호작용을 가능하게 하는 핵심적인 기술적 기반이 된다.


NanoVDB의 또 다른 핵심 설계 원칙은 '정적 토폴로지'이다.7 이는 한 번 생성된 NanoVDB 그리드의 트리 구조, 즉 활성 복셀의 공간적 분포(sparsity)를 동적으로 변경할 수 없음을 의미한다. 노드를 추가하거나 삭제하는 등의 위상 변경 작업은 허용되지 않는다.

그러나 토폴로지가 고정되어 있더라도, 이미 활성화된 복셀(voxel)이 가진 '값'(예: 밀도, 온도, 색상, 속도 벡터 등)은 수정이 가능하다.9 이 특성은 렌더링 과정에서 셰이더를 통해 절차적으로 값을 계산하거나, 물리 시뮬레이션에서 정적인 경계 조건(static boundary conditions)으로 활용하는 등 다양한 응용을 가능하게 한다.9

값 수정 기능은 공식 API로 명시적으로 제공되지는 않지만, `ReadAccessor`를 통해 얻은 리프 노드에 직접 접근하여 메모리상의 값을 변경하는 방식으로 구현될 수 있다.13 하지만 이 방법은 호스트 애플리케이션(예: Houdini)이 데이터 변경을 인지하지 못해 GPU와 CPU 간의 데이터 동기화 문제를 야기할 수 있는 잠재적 위험을 내포하고 있으므로 신중한 접근이 필요하다.13 이처럼 정적 토폴로지는 단순한 제약이 아니라, NanoVDB가 목표로 하는 '읽기 중심(read-heavy)' 작업에 최적화된 의도적인 설계 결정이다. 렌더링과 같은 작업에서는 데이터가 프레임 단위로 불변(immutable)한 경우가 많으므로, 이러한 제약은 성능 저하 없이 예측 가능성을 높이는 장점으로 작용한다.


NanoVDB는 빠른 데이터 조회를 위해 여러 최적화 기법을 사용한다.

첫째, OpenVDB와 마찬가지로 좌표를 이용해 가장 낮은 레벨의 리프 노드에서부터 상위 노드로 올라가는 **상향식 순회(Bottom-up Traversal)** 방식을 사용한다. 이를 통해 특정 좌표의 값을 조회할 때 평균적으로 상수 시간($O(1)$)의 매우 빠른 임의 접근(random access) 속도를 보장한다.5

둘째, 반복적인 접근 성능을 가속화하기 위해 **`ReadAccessor`** 클래스를 제공한다.11 `ReadAccessor`는 이전에 접근했던 노드의 정보를 내부적으로 캐시하여, 공간적으로 인접한 복셀에 연속적으로 접근할 때 매번 트리 전체를 순회하는 비용을 크게 줄여준다.5 다만, 이 캐싱 메커니즘은 스레드로부터 안전하지 않으므로(not thread-safe), GPU 커널과 같은 병렬 환경에서는 각 스레드가 자신만의 독립적인 `ReadAccessor` 인스턴스를 사용해야 한다.12

셋째, 레이 트레이싱(ray tracing)과 같은 순차적 접근 패턴을 위해 **계층적 DDA(Hierarchical Digital Differential Analyzer, HDDA)** 알고리즘을 구현했다.5 HDDA는 광선(ray)이 데이터가 없는 비어있는 공간(empty space)을 효율적으로 건너뛸 수 있도록 하여, 불필요한 연산을 제거하고 렌더링 성능을 극적으로 향상시킨다.14

이러한 설계는 GPU 아키텍처에 대한 깊은 이해에서 비롯된 필연적 귀결이다. 포인터 기반의 동적 메모리 할당은 GPU의 SIMT(Single Instruction, Multiple Thread) 실행 모델과 메모리 병합(coalesced memory access) 접근 방식에 근본적으로 비효율적이다. 따라서, **(1) 포인터 제거 -->> (2) 선형화된 단일 메모리 블록 -->> (3) 정적 토폴로지 제약** 이라는 인과적 설계 결정이 내려졌다. 이는 동적 유연성을 희생하여 GPU에서의 압도적인 읽기 성능을 얻는 명확한 트레이드오프(trade-off) 전략이다.


NanoVDB의 가치를 정확히 이해하기 위해서는 산업 표준인 OpenVDB와의 직접적인 비교가 필수적이다. 이 장에서는 두 기술의 구조적, 기능적 차이점을 분석하고, 정량적인 벤치마크 데이터를 통해 NanoVDB의 성능적 우위를 입증한다.


OpenVDB와 NanoVDB는 동일한 VDB 개념에서 출발했지만, 목표하는 사용 환경과 최적화 방향이 달라 여러 측면에서 뚜렷한 차이를 보인다.

- **데이터 구조 및 메모리 레이아웃:** OpenVDB는 동적 토폴로지를 지원하는 포인터 기반 트리 구조로, 데이터의 생성 및 편집에 강점을 가진다. 반면, NanoVDB는 GPU 전송 및 접근에 최적화된 선형화된 정적 토폴로지 구조를 채택한다.3 이로 인해 OpenVDB는 동적 메모리 할당으로 인해 메모리 단편화가 발생할 수 있지만, NanoVDB는 항상 연속적인 메모리 블록을 사용하여 GPU의 메모리 접근 패턴에 유리하고 CPU 캐시 효율성 또한 극대화한다.7
- **의존성 및 이식성:** OpenVDB는 Boost, Intel TBB 등 여러 외부 라이브러리에 의존하여 특정 개발 환경을 구축하는 데 복잡성이 따를 수 있다.5 반면, NanoVDB는 C++11 또는 C99 표준 외에 거의 의존성이 없는 소수의 헤더 파일로 구현되어 이식성이 매우 높고 기존 프로젝트에 통합하기가 용이하다.7
- **기능 범위:** OpenVDB는 필터링, CSG(Constructive Solid Geometry) 연산, 동적 시뮬레이션 지원 툴 등 방대하고 포괄적인 기능 셋을 제공한다.3 NanoVDB는 빠른 데이터 조회(retrieval)에 초점을 맞춰, 값 접근, 보간, 레이 트레이싱 지원 등 렌더링 및 정적 충돌 감지에 필수적인 핵심 기능에 집중한다.7

이러한 차이점은 아래 표로 요약할 수 있다. 이 표는 두 기술의 핵심적인 트레이드오프를 한눈에 보여주며, 사용자가 자신의 애플리케이션 요구사항에 따라 어떤 기술이 더 적합한지 신속하게 판단하는 데 도움을 준다.

| 특징 (Feature)      | OpenVDB                                              | NanoVDB                                                      |
| ------------------- | ---------------------------------------------------- | ------------------------------------------------------------ |
| **주 사용 환경**    | CPU (다중 스레딩 최적화)                             | GPU (대규모 병렬 처리 최적화), CPU                           |
| **데이터 구조**     | 포인터 기반 동적 트리 (Pointer-based Dynamic Tree)   | 선형화된 포인터-리스 정적 트리 (Linearized Pointer-less Static Tree) |
| **메모리 레이아웃** | 단편화(Fragmented) 가능                              | 연속적(Contiguous), 캐시 친화적(Cache-friendly)              |
| **토폴로지**        | **동적 (Dynamic)**: 노드 추가/삭제 등 구조 변경 가능 | **정적 (Static)**: 노드 구조 변경 불가, 값만 수정 가능       |
| **의존성**          | Boost, TBB, OpenEXR 등 다수                          | C++11/C99 표준 외 최소화 (거의 없음)                         |
| **핵심 기능**       | 데이터 생성, 동적 수정, 필터링 등 포괄적 툴셋        | 빠른 읽기/접근, 렌더링, 정적 충돌 감지에 특화                |
| **데이터 전송**     | GPU 전송 시 복잡한 처리 필요                         | `memcpy`로 단순하고 빠른 GPU 전송 가능                       |
| **이식성**          | 상대적으로 낮음                                      | 매우 높음 (다양한 그래픽 API 지원)                           |


NanoVDB의 설계 목표는 성능 향상이며, 이는 다양한 벤치마크를 통해 입증된다.

- **CPU 성능:** 흥미롭게도 NanoVDB는 GPU뿐만 아니라 CPU 환경에서도 성능 이점을 보인다. 이는 단편화되지 않은 연속적인 메모리 레이아웃이 CPU 캐시의 지역성(locality)을 높여주기 때문이다.7 NVIDIA가 공개한 벤치마크에 따르면, TBB(Threading Building Blocks)를 사용한 CPU 환경에서 LevelSet 데이터에 임의 접근하는 테스트에서 OpenVDB는 148.182ms가 소요된 반면, NanoVDB는 11.554ms를 기록하여 12배 이상 빠른 성능을 보였다.5
- **GPU 성능:** GPU 가속은 NanoVDB의 존재 이유이자 가장 큰 장점이다. SideFX Houdini 18.5에서는 Vellum 솔버의 정적 충돌 계산과 Pyro 솔버의 소싱(sourcing) 작업을 CPU에서 GPU로 이전하면서 NanoVDB를 채택했고, 이를 통해 아티스트들은 훨씬 더 빠르고 유연한 상호작용 경험을 얻게 되었다.1 로보틱스 분야의 한 연구에서는 고밀도 포인트 클라우드 데이터를 NanoVDB 구조로 GPU 상에서 인코딩함으로써 맵 생성 및 업데이트 속도를 획기적으로 향상시킨 사례를 보고했다.16
- **메모리 사용량:** NanoVDB는 구조적으로 OpenVDB보다 약간 더 적은 메모리를 사용하는 경향이 있다.9 더 나아가, NanoVDB는 내장된 런타임(in-core) 압축 기능을 제공한다. 이 기능을 사용하면 메모리 점유율을 4배에서 6배까지 줄일 수 있으며, 압축 해제에 드는 약간의 연산 비용보다 메모리 대역폭 절감으로 인한 이득이 더 커져, 오히려 렌더링 성능이 10%에서 30%까지 향상되는 현상이 관찰되기도 했다.7

결론적으로, NanoVDB는 OpenVDB를 대체하는 기술이 아니라 상호 보완하고 확장하는 관계에 있다. 일반적인 프로덕션 워크플로우는 **"생성/시뮬레이션(OpenVDB, CPU) -->> 변환(Converter) -->> 렌더링/상호작용(NanoVDB, GPU)"** 의 파이프라인을 형성한다.1 NanoVDB의 등장은 OpenVDB 생태계를 실시간 처리 영역으로 확장시키는 결정적인 교두보 역할을 수행했다.


이 장에서는 개발자 관점에서 NanoVDB를 실제 프로젝트에 어떻게 통합하고 프로그래밍하는지에 대해 구체적인 코드 예제와 함께 설명한다. NanoVDB의 API는 복잡한 GPU 프로그래밍 세부 사항을 효과적으로 추상화하여 개발자가 고수준에서 GPU 가속을 활용할 수 있도록 설계되었다.


NanoVDB의 가장 큰 장점 중 하나는 뛰어난 이식성과 간편한 통합 방식이다. 핵심 기능은 `NanoVDB.h`(C++11), `CNanoVDB.h`(C99), `PNanoVDB.h`(이식성을 극대화한 C99)와 같은 소수의 헤더 파일에 독립적으로 구현되어 있어, 프로젝트에 추가하는 과정이 매우 간단하다.9

또한, NanoVDB는 특정 벤더나 플랫폼에 종속되지 않는 범용 솔루션을 지향한다. CUDA, OpenCL, OpenGL, DirectX, OptiX, Vulkan, HLSL, GLSL 등 현존하는 거의 모든 주요 그래픽 및 컴퓨팅 API를 지원하도록 설계되었다.8 이러한 플랫폼 불가지론(agnosticism)은 Academy Software Foundation(ASWF) 산하의 오픈소스 프로젝트로서의 철학을 반영하며, 광범위한 채택을 가능하게 하는 중요한 요소이다.6


기존 OpenVDB 자산을 활용하기 위해, NanoVDB는 간편한 변환 유틸리티를 제공한다. 변환 프로세스는 `nanovdb/util/CreateNanoGrid.h` 또는 `nanovdb/util/OpenToNanoVDB.h` 헤더에 포함된 함수를 통해 이루어진다.21

다음은 OpenVDB 그리드를 생성하여 NanoVDB 파일(`.nvdb`)로 저장하는 기본적인 C++ 코드 예제이다.

```C++
// 1. OpenVDB 및 NanoVDB 유틸리티 헤더 포함
#include <openvdb/tools/LevelSetSphere.h>
#include <nanovdb/util/CreateNanoGrid.h>
#include <nanovdb/util/IO.h>

int main() {
    try {
        // 2. OpenVDB 그리드 생성 (예: 구 형태의 레벨셋) [21]
        auto srcGrid = openvdb::tools::createLevelSetSphere<openvdb::FloatGrid>(
            100.0f, openvdb::Vec3f(0.0f), 1.0f);

        // 3. OpenVDB 그리드를 NanoVDB 그리드로 변환 [21, 23]
        // 이 함수가 OpenVDB의 동적 트리를 순회하여 선형화된 메모리 버퍼를 생성한다.
        auto handle = nanovdb::createNanoGrid(*srcGrid);

        // 4. 변환된 NanoVDB 그리드를 파일로 저장 [21]
        nanovdb::io::writeGrid("data/sphere.nvdb", handle);

    } catch (const std::exception& e) {
        std::cerr << "An exception occurred: \"" << e.what() << "\"" << std::endl;
    }
    return 0;
}
```

위 코드에서 `nanovdb::createNanoGrid()` 함수는 복잡한 변환 과정을 캡슐화한다. 반환된 `handle` 객체(`nanovdb::GridHandle`)는 생성된 메모리 버퍼의 소유권을 안전하게 관리하는 RAII(Resource Acquisition Is Initialization) 래퍼(wrapper) 역할을 한다.5


NanoVDB의 진정한 힘은 CUDA 커널 내에서 발휘된다. `nanovdb::CudaDeviceBuffer`를 사용하면 GPU 메모리에 직접 NanoVDB 그리드를 생성하거나, 호스트에서 생성된 그리드를 GPU로 효율적으로 업로드할 수 있다.5

다음은 OpenVDB 그리드를 GPU 메모리에 상주하는 NanoVDB 그리드로 변환하고, 이를 CUDA 커널에서 접근하는 예제이다.

**호스트 코드 (C++/CUDA):**

```C++
// (srcGrid는 openvdb::FloatGrid::Ptr 타입으로 미리 생성되었다고 가정)

// 1. CudaDeviceBuffer를 템플릿 인자로 지정하여 GPU 메모리에 NanoVDB 그리드 생성 [5, 24]
auto handle = nanovdb::openToNanoVDB<nanovdb::CudaDeviceBuffer>(*srcGrid);

// 2. (필요 시) 호스트의 데이터를 디바이스로 업로드
handle.deviceUpload();

// 3. CUDA 커널에 전달할 디바이스 그리드의 포인터 획득
const nanovdb::FloatGrid* deviceGrid = handle.deviceGrid<float>();

// 4. CUDA 커널 실행
my_nanovdb_kernel<<<gridDim, blockDim>>>(deviceGrid, /*... 다른 파라미터... */);
```

**디바이스 코드 (CUDA 커널):**

```C++
__global__ void my_nanovdb_kernel(const nanovdb::FloatGrid* grid, float* output)
{
    // 스레드 ID 계산
    int x = blockIdx.x * blockDim.x + threadIdx.x;
    int y = blockIdx.y * blockDim.y + threadIdx.y;
    
    // 1. 각 스레드는 독립적인 ReadAccessor를 생성 [12]
    // 이는 스레드 안전성을 보장하고 접근 캐싱을 활용하기 위함이다.
    auto accessor = grid->getAccessor();

    // 2. 월드 좌표(worldPos)로부터 복셀 값(value)을 조회
    nanovdb::Vec3f worldPos(x, y, 0.0f); // 스레드별로 계산된 위치
    float value = accessor.getValue(worldPos);

    // 3. 조회된 값을 사용하여 연산 수행
    output[y * grid_width + x] = value * 2.0f;
}
```

NanoVDB를 사용하는 CUDA 커널을 최적화할 때는 GPU 아키텍처의 특성을 고려해야 한다. `ReadAccessor`의 적극적인 사용은 필수적이며, 공간적으로 인접한 픽셀을 처리하는 스레드들이 인접한 복셀에 접근하도록 스레드 작업을 매핑하면 `ReadAccessor`의 캐시 효율과 GPU의 메모리 병합(coalescing) 효과를 동시에 극대화할 수 있다.5

`ReadAccessor` 자체가 매우 가볍게 설계되었기 때문에 12, 스레드당 레지스터 사용량을 적게 유지하여 높은 점유율(occupancy)을 달성하고 메모리 접근 지연 시간(latency)을 효과적으로 숨기는 데 유리하다.


NanoVDB는 뛰어난 성능과 이식성을 바탕으로 발표 이후 빠르게 산업계 전반에 확산되었다. 처음에는 VFX 분야의 오프라인 렌더링 가속에 집중되었으나 6, 점차 실시간 게임 엔진, 과학 시각화, 로보틱스 등 다양한 분야로 그 영향력을 넓혀가고 있다.


- **SideFX Houdini:** Houdini는 NanoVDB를 가장 적극적으로 도입한 DCC(Digital Content Creation) 툴 중 하나이다. Houdini 18.5 버전부터 Pyro FX 솔버의 소싱(sourcing), Vellum 솔버의 정적 충돌 감지 등에 NanoVDB를 활용하여 GPU 가속을 구현했다.1 최신 버전인 Houdini 20.5에서는 MPM(Material Point Method) 솔버의 배경 그리드에 NanoVDB를 사용하여 대규모 고체 역학 시뮬레이션의 효율을 극대화했다.26 이를 통해 아티스트는 더 복잡한 시뮬레이션을 훨씬 빠른 속도로 반복하며 창의적인 결과물을 만들어낼 수 있게 되었다.
- **Blender:** Blender는 렌더링 엔진인 Cycles에 NanoVDB를 통합하여 GPU 렌더링 시 메모리 사용량을 획기적으로 줄였다.1 이 통합은 Blender 2.92 버전부터 기본으로 활성화되었으며 28, 덕분에 기존에는 메모리 부족으로 렌더링이 불가능했던 복잡하고 거대한 볼륨 씬을 GPU에서 직접 렌더링할 수 있게 되었다. 또한, 뷰포트에서의 인터랙티브한 피드백 속도 또한 크게 향상되어 아티스트의 작업 효율을 높였다.27
- **Unreal Engine:** 공식적인 지원은 아직 없지만, Eidos-Montréal에서 개발한 오픈소스 플러그인을 통해 OpenVDB 및 NanoVDB 파일을 Unreal Engine 5로 가져와 렌더링하는 것이 가능하다.30 이 플러그인은 임포트 과정에서 OpenVDB를 GPU 친화적인 NanoVDB 포맷으로 자동 변환하며, 실시간 렌더링뿐만 아니라 Unreal의 고품질 패스 트레이서(Path Tracer)를 이용한 오프라인 렌더링까지 지원한다.30 이는 게임 엔진을 활용한 고품질 시네마틱 제작 및 가상 프로덕션에서 VDB 데이터 활용의 새로운 가능성을 열어주고 있다.
- **기타 주요 렌더러:** Autodesk의 Arnold, Pixar의 RenderMan과 같은 업계 최고의 프로덕션 렌더러들 또한 GPU 기반 렌더링 파이프라인을 강화하기 위해 NanoVDB를 적극적으로 채택하고 있다.8


- **대규모 데이터 시각화:** 기후 모델링, 천체물리 시뮬레이션 등에서 생성되는 방대한 양의 볼류메트릭 데이터셋을 실시간으로 탐색하고 분석하는 데 NanoVDB가 중요한 역할을 할 수 있다.34 예를 들어, ParaView와 같은 과학 시각화 도구가 NVIDIA Omniverse Connector를 통해 OpenVDB/NanoVDB를 지원하게 되면서, 연구자들은 복잡한 시뮬레이션 결과를 사진처럼 사실적인 비주얼로 상호작용하며 분석할 수 있게 되었다.34
- **의료 영상:** 고해상도 CT 및 MRI 스캔 데이터는 본질적으로 희소 볼류메트릭 데이터이다. NanoVDB와 그 후속 기술인 NeuralVDB는 이러한 대용량 의료 데이터를 효율적으로 저장하고 실시간으로 3D 시각화하여 의사의 진단 및 수술 계획 수립을 돕는 데 사용될 잠재력이 매우 크다.36
- **로보틱스 및 자율주행:** 로봇이 주변 환경을 3D로 인식하기 위해 생성하는 포인트 클라우드 기반의 점유 격자 맵(occupancy map)을 NanoVDB를 이용해 GPU에서 빠르게 처리하고 업데이트하는 연구가 활발히 진행되고 있다.16 이는 제한된 컴퓨팅 자원을 가진 로봇이 실시간으로 장애물을 회피하고 경로를 계획하는 데 필수적인 기술이다.

NanoVDB의 빠른 산업계 채택은 단순히 '성능' 때문만이 아니다. **(1) 오픈소스(ASWF 관리), (2) 높은 이식성(최소 의존성), (3) OpenVDB와의 완벽한 호환성**이라는 세 가지 요소가 결합하여 기존 파이프라인에 대한 '도입 저항'을 최소화했기 때문이다. 기업들은 큰 비용과 노력 없이 기존의 방대한 OpenVDB 자산을 그대로 활용하면서 GPU 가속의 이점을 누릴 수 있었다.

| 소프트웨어/렌더러    | 버전  | 주요 통합 내용                                    | 관련 자료 |
| -------------------- | ----- | ------------------------------------------------- | --------- |
| **SideFX Houdini**   | 18.5+ | Pyro 소싱, Vellum 충돌 감지 GPU 가속              | 1         |
|                      | 20.5+ | MPM 솔버 배경 그리드                              | 26        |
| **Blender**          | 2.92+ | Cycles 렌더러 GPU 메모리 최적화, 뷰포트 성능 향상 | 1         |
| **Unreal Engine**    | 5.x   | 서드파티 플러그인을 통한 임포트 및 렌더링 지원    | 30        |
| **Autodesk Arnold**  | 7+    | GPU 렌더링 지원                                   | 8         |
| **NVIDIA Omniverse** | -     | ParaView Connector를 통한 OpenVDB/NanoVDB 시각화  | 8         |


NanoVDB는 실시간 볼류메트릭 렌더링의 새로운 장을 열었지만, 기술적 한계 또한 명확했다. 이 장에서는 NanoVDB의 한계를 조명하고, 이를 극복하기 위해 등장한 후속 기술인 NeuralVDB를 분석함으로써 VDB 기술의 발전 방향과 실시간 그래픽스의 미래를 전망한다.


NanoVDB의 가장 큰 한계는 메모리 압축률에 있었다. 내장된 런타임 압축 기능은 시각적 품질 저하가 거의 없는 수준에서 4배에서 6배 정도의 압축률을 제공하는 데 그쳤다.7 초고해상도 볼륨 데이터나 대규모 시뮬레이션 데이터셋의 경우, 여전히 GPU 메모리 점유율이 큰 부담으로 작용했다.

이러한 메모리 한계를 근본적으로 해결하기 위해, NVIDIA는 2022년 SIGGRAPH에서 AI와 머신러닝 기술을 VDB에 접목한 **NeuralVDB**를 발표했다.36 NeuralVDB는 VDB 데이터 구조를 신경망을 이용해 표현함으로써, 약간의 손실 압축을 감수하는 대신 메모리 점유율을 획기적으로 줄이는 것을 목표로 한다.36 이는 기술의 목표점에서 중요한 차이를 보인다. NanoVDB의 주 목표가 '렌더링 속도 향상'이었다면, NeuralVDB의 주 목표는 '메모리 점유율의 혁신적 감소'이다. NeuralVDB는 이를 위해 렌더링 속도를 일부 희생할 수 있음을 명시적으로 밝히고 있다.41


NeuralVDB는 순수한 신경망 표현이 아니라, 기존 VDB 트리와 신경망의 장점을 결합한 하이브리드(hybrid) 구조를 채택한다.41 VDB 트리의 상위 레벨은 그대로 유지하여 데이터의 희소한 영역을 효율적으로 건너뛰는 공간적 적응성(spatial adaptivity)을 확보한다. 그리고 데이터가 밀집된 하위 노드들(주로 리프 노드)을 작고 효율적인 다수의 신경망으로 대체한다.41

이 접근법의 핵심은 **토폴로지(topology)와 값(value)의 분리 인코딩**이다.41 각 하위 영역을 담당하는 신경망은 두 부분으로 나뉜다.

1. **무손실 분류기 (Lossless Classifier):** 특정 좌표에 복셀이 존재하는지 여부, 즉 활성 상태(토폴로지)를 예측한다. 이 과정은 무손실로 이루어져 공간 구조 정보가 정확하게 보존된다.
2. **손실 회귀기 (Lossy Regressor):** 활성 상태로 분류된 복셀의 실제 값(밀도, 색상 등)을 예측한다. 이 과정은 손실 압축으로, 사용자가 허용 오차를 제어하여 압축률과 품질 사이의 균형을 맞출 수 있다.

이러한 하이브리드 방식은 전통적인 데이터 압축을 넘어, 데이터의 내재적 특성과 패턴을 학습하여 근사(approximating)하는 AI 기반 접근법으로의 패러다임 전환을 의미한다.41


NeuralVDB는 메모리 효율성 측면에서 놀라운 결과를 보여준다. 이미 압축된 OpenVDB 파일 대비 10배에서 최대 100배에 달하는 추가적인 메모리 점유율 감소 효과를 보인다.36 대표적인 예로, 1.5GB에 달하는 Disney Cloud 데이터셋은 NeuralVDB로 변환 시 단 25MB로 압축되어 60배의 압축률을 기록했다.41

애니메이션 시퀀스의 경우, 이전 프레임의 학습된 신경망 가중치를 다음 프레임 학습의 초기값(warm-starting)으로 사용하여 압축(학습) 속도를 최대 2배까지 가속할 수 있다. 또한, 이 과정은 프레임 간의 부드러운 전환, 즉 시간적 일관성(temporal coherency)을 보장하여 모션 블러와 같은 후처리 효과의 필요성을 줄여준다.37

NanoVDB가 실시간 렌더링의 문을 열었다면, NeuralVDB는 대규모 디지털 트윈, 현실 규모의 과학 시뮬레이션 시각화, AI 기반 의료 영상 분석 등 지금까지 하드웨어 메모리 한계로 인해 시도하기 어려웠던 영역으로 VDB의 활용 범위를 확장할 것이다.36 VDB 기술은 

**OpenVDB (CPU, 동적) -->> NanoVDB (GPU, 정적, 고속 읽기) -->> NeuralVDB (GPU+AI, 초고압축)** 의 단계로 명확하게 진화하고 있다. 각 단계는 이전 기술을 대체하는 것이 아니라 상호 보완하며 새로운 응용 분야를 개척하고 있다. 이러한 VDB 생태계의 발전은 미래 그래픽스가 '더 빠른 렌더링'뿐만 아니라 '더 거대한 데이터를 어떻게 효율적으로 다룰 것인가'라는 문제에 집중하고 있음을 시사한다.


NanoVDB는 산업 표준인 OpenVDB를 GPU 시대의 요구에 맞춰 재해석한, 고도로 최적화된 읽기 전용 데이터 구조이다. 이는 기존 기술의 단순한 이식(porting)을 넘어, GPU 하드웨어 아키텍처의 본질에 부합하도록 데이터 표현 방식을 근본적으로 재설계한 기술적 성취라 할 수 있다.

NanoVDB의 성공은 여러 요인이 복합적으로 작용한 결과이다. 선형화된 포인터-리스 아키텍처는 GPU와의 데이터 전송 및 접근 효율을 극대화하여 실시간 렌더링과 시뮬레이션의 새로운 가능성을 열었다. '정적 토폴로지'라는 명확한 제약은 동적 시뮬레이션에는 한계로 작용하지만, 렌더링과 같이 '읽기 중심'의 작업에서는 오히려 예측 가능성과 성능을 높이는 장점으로 작용했다. 이러한 명확한 목적성과 트레이드오프 전략, 그리고 OpenVDB와의 완벽한 호환성 및 오픈소스 정책은 산업계의 빠른 채택을 이끌어낸 핵심 동력이었다.

기술사적 관점에서 NanoVDB는 하드웨어 아키텍처에 맞는 데이터 구조 설계의 중요성을 입증한 모범 사례이자, VDB 기술 진화의 중요한 변곡점이다. 이는 AI 기술을 접목하여 메모리 효율을 극한으로 끌어올린 NeuralVDB로 발전하는 필수적인 교두보 역할을 수행했다. OpenVDB-NanoVDB-NeuralVDB로 이어지는 상호 보완적인 생태계는 앞으로 볼류메트릭 데이터가 더욱 거대해지고 복잡해지는 미래에, 데이터의 생성, 처리, 저장, 시각화 전반에 걸쳐 유연하고 강력한 솔루션을 제공할 것이다. 이는 시각 효과와 게임 산업을 넘어 과학, 의료, 로보틱스 등 산업 전반에 걸쳐 실시간 3D 데이터 활용의 새로운 지평을 열 것으로 기대된다.


1. NanoVDB - NVIDIA Developer, accessed August 4, 2025, https://developer.nvidia.com/nanovdb
2. Open VDB and Nano VDB Rendering - Feedback & Requests - Unreal Engine Forums, accessed August 4, 2025, https://forums.unrealengine.com/t/open-vdb-and-nano-vdb-rendering/150807
3. What is OpenVDB and why should you care? [thinkingParticles Documentation], accessed August 4, 2025, https://cebas.com/manual/LifeLicenser/doku.php?id=reference_guide:thinkingparticles_nodes:operator_nodes:openvdb:what_is_openvdb
4. OpenVDB - Wikipedia, accessed August 4, 2025, https://en.wikipedia.org/wiki/OpenVDB
5. Accelerating OpenVDB on GPUs with NanoVDB | NVIDIA Technical Blog, accessed August 4, 2025, https://developer.nvidia.com/blog/accelerating-openvdb-on-gpus-with-nanovdb/
6. ASWF - OpenVDB Adds GPU Support with NanoVDB; OpenVDB Version 7.1 Now Available, accessed August 4, 2025, https://www.aswf.io/news/nanovdb/
7. NanoVDB: A GPU-Friendly and Portable VDB Data Structure For Real-Time Rendering And Simulation - Research at NVIDIA, accessed August 4, 2025, https://research.nvidia.com/labs/prl/nanovdb/nanovdb2021.pdf
8. NanoVDB: A GPU-Friendly and Portable VDB Data Structure For Real-Time Rendering And Simulation - Research at NVIDIA, accessed August 4, 2025, https://research.nvidia.com/labs/prl/publication/nanovdb/
9. OpenVDB: FAQ - GitHub Pages, accessed August 4, 2025, https://academysoftwarefoundation.github.io/openvdb/NanoVDB_FAQ.html
10. NanoVDB: A GPU-Friendly and Portable VDB Data Structure For Real-Time Rendering And Simulation | Request PDF - ResearchGate, accessed August 4, 2025, https://www.researchgate.net/publication/353747008_NanoVDB_A_GPU-Friendly_and_Portable_VDB_Data_Structure_For_Real-Time_Rendering_And_Simulation
11. nanovdb/NanoVDB.h Source File - OpenVDB, accessed August 4, 2025, https://www.openvdb.org/documentation/doxygen/NanoVDB_8h_source.html
12. NanoVDB.h - OpenVDB - GitHub Pages, accessed August 4, 2025, https://academysoftwarefoundation.github.io/openvdb/NanoVDB_8h_source.html
13. [NanoVDB] Is it really possible to modify the voxel values, and how ..., accessed August 4, 2025, https://github.com/AcademySoftwareFoundation/openvdb/discussions/1208
14. NanoVDB: A GPU-Friendly and Portable VDB Data Structure For Real-Time Rendering And Simulation - Semantic Scholar, accessed August 4, 2025, https://www.semanticscholar.org/paper/NanoVDB%3A-A-GPU-Friendly-and-Portable-VDB-Data-For-Museth/9884344a849fee4cdeee6ee05d952b245b8b6749
15. About OpenVDB, accessed August 4, 2025, https://www.openvdb.org/about/
16. NanoMap: A GPU-Accelerated OpenVDB-Based Mapping and Simulation Package for Robotic Agents | QUT ePrints, accessed August 4, 2025, https://eprints.qut.edu.au/236544/
17. NanoMap: A GPU-Accelerated OpenVDB-Based Mapping and Simulation Package for Robotic Agents - MDPI, accessed August 4, 2025, https://www.mdpi.com/2072-4292/14/21/5463
18. OpenVDB NanoVDB - Source tree structure, accessed August 4, 2025, https://www.openvdb.org/documentation/doxygen/NanoVDB_SourceTree.html
19. SIGGRAPH '21: ACM SIGGRAPH 2021 Talks, accessed August 4, 2025, https://www.siggraph.org/wp-content/uploads/2021/08/ACM-SIGGRAPH-2021-Talks.html
20. AcademySoftwareFoundation/openvdb: OpenVDB - Sparse volume data structure and tools, accessed August 4, 2025, https://github.com/AcademySoftwareFoundation/openvdb
21. NanoVDB Hello World Examples - OpenVDB, accessed August 4, 2025, https://www.openvdb.org/documentation/doxygen/NanoVDB_HelloWorld.html
22. File List - OpenVDB, accessed August 4, 2025, https://www.openvdb.org/documentation/doxygen/files.html
23. Tutorial: Converting VDB files to NanoVDB via OpenVDB in C++ | Ben Ahlbrand CV, accessed August 4, 2025, https://benjamin.ahlbrand.me/post/2022-06-27-tutorial-converting-vdb-files-to-nanovdb-via-openvdb-in-c/
24. [HDK] Create a NanoVDB grid using a Cuda Device Buffer ? | Forums - SideFX, accessed August 4, 2025, https://www.sidefx.com/forum/topic/97386/
25. GPU Architecture and Optimization Blueprint | by saadtariq | Jul, 2025 - Medium, accessed August 4, 2025, https://medium.com/@saadtariq0812/gpu-architecture-and-optimization-blueprint-4ba3c34d8019
26. What's new MPM - SideFX, accessed August 4, 2025, https://www.sidefx.com/docs/houdini/news/20_5/mpm.html
27. Interactive Volumes with NanoVDB in Blender Cycles - YouTube, accessed August 4, 2025, https://www.youtube.com/watch?v=F8xa_CA53jA
28. Cycles: enable NanoVDB by default #81454 - Blender Developer, accessed August 4, 2025, https://developer.blender.org/T81454
29. Interactive volumes with NanoVDB in Blender Cycles - NVIDIA, accessed August 4, 2025, https://resources.nvidia.com/en-us-new-media-entertainment/f8xa_ca53ja
30. OpenVDB and NanoVDB in Unreal 5 #UE5, plugin from Eidos-Montreal and Eidos-Sherbrooke - YouTube, accessed August 4, 2025, https://www.youtube.com/watch?v=jgi5htRqsAE
31. Adding OpenVDB Support to Unreal - Eidos-Montréal, accessed August 4, 2025, https://www.eidosmontreal.com/news/adding-openvdb-support-to-unreal/
32. eidosmontreal/unreal-vdb: This repo is a non-official Unreal plugin that can read OpenVDB and NanoVDB files in Unreal. - GitHub, accessed August 4, 2025, https://github.com/eidosmontreal/unreal-vdb
33. 3D Gaussian Particle Approximation of VDB Datasets: A Study for Scientific Visualization, accessed August 4, 2025, https://arxiv.org/html/2504.04857v1
34. Accelerate Scientific Visualizations with NVIDIA Omniverse ParaView Connector, Now Available - Kitware, Inc., accessed August 4, 2025, https://www.kitware.com/nvidia-omniverse-paraview-connector/
35. GPU-Powered Scientific Visualization - NVIDIA, accessed August 4, 2025, https://www.nvidia.com/en-us/high-performance-computing/scientific-visualization/
36. NVIDIA NeuralVDB - NVIDIA Developer, accessed August 4, 2025, https://developer.nvidia.com/rendering-technologies/neuralvdb
37. Upping the Standard: NVIDIA Introduces NeuralVDB, Bringing AI and GPU Optimization to Award-Winning OpenVDB, accessed August 4, 2025, https://blogs.nvidia.com/blog/neuralvdb-ai/
38. NVIDIA NeuralVDB Brings AI and GPU Optimization to OpenVDB - YouTube, accessed August 4, 2025, https://www.youtube.com/watch?v=uAs8X5es1DE
39. Search | Forums | SideFX, accessed August 4, 2025, [https://www.sidefx.com/forum/search/?keywords=&sort_by=0&sort_dir=DESC&search_in=all&forum=0&action=search&author=drew&show_as=posts](https://www.sidefx.com/forum/search/?keywords&sort_by=0&sort_dir=DESC&search_in=all&forum=0&action=search&author=drew&show_as=posts)
40. How do I render VDBs from Bifrost Graph to Arnold (GPU)? "[gpu] could not create bifrost_multires_v... - Autodesk Forums, accessed August 4, 2025, https://forums.autodesk.com/t5/bifrost-forum/how-do-i-render-vdbs-from-bifrost-graph-to-arnold-gpu-quot-gpu/td-p/11752005
41. NeuralVDB: High-resolution Sparse Volume Representation using ..., accessed August 4, 2025, https://research.nvidia.com/labs/prl/neuralvdb/neuralvdb2024.pdf
42. NVIDIA Introduces NeuralVDB at SIGGRAPH - HPCwire, accessed August 4, 2025, https://www.hpcwire.com/off-the-wire/nvidia-introduces-neuralvdb-at-siggraph/
43. Optimizing Large-Scale Sparse Volumetric Data with NVIDIA NeuralVDB Early Access, accessed August 4, 2025, https://developer.nvidia.com/blog/optimizing-large-scale-sparse-volumetric-data-with-nvidia-neuralvdb-early-access/
44. NeuralVDB: High-resolution Sparse Volume Representation using Hierarchical Neural Networks | Request PDF - ResearchGate, accessed August 4, 2025, https://www.researchgate.net/publication/377642706_NeuralVDB_High-resolution_Sparse_Volume_Representation_using_Hierarchical_Neural_Networks