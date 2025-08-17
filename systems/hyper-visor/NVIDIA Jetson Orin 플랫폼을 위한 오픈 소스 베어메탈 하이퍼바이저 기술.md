# NVIDIA Jetson Orin 플랫폼을 위한 오픈 소스 베어메탈 하이퍼바이저 기술 분석





### **Executive Summary**



본 보고서는 NVIDIA Jetson Orin 플랫폼에서 사용 가능한 오픈 소스 베어메탈(Type-1) 하이퍼바이저 솔루션에 대한 심층 기술 분석을 제공합니다. Jetson Orin 시스템 온 칩(SoC)은 Arm Cortex-A78AE CPU와 Ampere 아키텍처 기반의 통합 GPU를 탑재하여 하드웨어 수준에서 강력한 가상화 기능을 내재하고 있습니다. 그러나 이러한 하드웨어 역량과 NVIDIA의 공식 지원 정책 사이에는 명백한 괴리가 존재합니다. 공식적으로 Jetson 플랫폼에서의 하이퍼바이저 사용은 지원되지 않으며 1, 이는 모든 가상화 구현이 커뮤니티 주도의 노력에 의존해야 함을 의미합니다.

본 분석은 KVM, Xen, Jailhouse, seL4 네 가지 주요 오픈 소스 하이퍼바이저에 초점을 맞춥니다. 각각의 아키텍처, 기능, 구현 가능성, 그리고 Jetson Orin 플랫폼에서의 적합성을 평가합니다.

분석 결과, 다음과 같은 결론을 도출했습니다.

- **Xen 하이퍼바이저**는 `meta-xen` Yocto 프로젝트를 통해 Jetson Orin에서 가장 성숙하고 접근성 높은 범용 가상화 경로를 제공합니다.2 특히 복잡한 통합 GPU(iGPU) 패스스루(passthrough) 문제에 대해 가장 유망한 접근 방식을 보여줍니다.
- **Jailhouse**는 정적 파티셔닝 하이퍼바이저로서, 실시간 운영체제(RTOS)와 고성능 리눅스 환경을 병존시켜야 하는 혼합 현실(mixed-criticiality) 시스템에 가장 적합한 솔루션입니다.3
- **seL4** 마이크로커널은 형식 검증을 통해 최고의 보안 보증 수준을 제공하므로, 국방, 항공우주 등 고신뢰성(high-assurance) 보안이 요구되는 특수 목적 시스템에 필수적인 선택지입니다.5
- **KVM**은 리눅스 커널에 통합된 하이퍼바이저로서 개념적 진입 장벽이 낮아 가상화 실험 및 학습에 적합하지만, 현재로서는 Orin의 iGPU 가상화에 대한 명확하고 안정적인 해결책이 부재합니다.

따라서 프로젝트의 목표가 범용 OS 통합인지, 실시간 성능 보장인지, 혹은 검증된 보안인지에 따라 최적의 하이퍼바이저 선택이 달라져야 합니다. 본 보고서는 이러한 기술적 결정을 내리는 데 필요한 심층적인 데이터와 분석을 제공하는 것을 목표로 합니다.

------



## 1.  NVIDIA Jetson Orin의 가상화 아키텍처 기반



이 장에서는 Jetson Orin SoC의 가상화 관련 하드웨어 구성 요소를 상세히 분석하고, 이를 가능하게 하는 기반 기술인 ARMv8-A 아키텍처의 특징을 설명합니다. 이를 통해 "Jetson Orin은 어떤 하드웨어 및 아키텍처 기능을 제공하며, 이것이 어떻게 가상화를 지원하는가?"라는 근본적인 질문에 답하고자 합니다.



### 1.1  Jetson Orin 시스템 온 칩(SoC) 환경



Jetson Orin 플랫폼의 가상화 잠재력을 이해하기 위해서는 먼저 그 핵심인 SoC의 구조를 파악해야 합니다. Orin 제품군은 개발자 킷부터 산업용 모듈에 이르기까지 다양한 폼팩터로 제공되지만, 모두 동일한 SoC 아키텍처를 공유합니다.7 가상화와 직접적으로 관련된 핵심 구성 요소는 다음과 같습니다.

- **CPU:** Orin은 멀티코어 Arm Cortex-A78AE v8.2 64비트 CPU 클러스터를 탑재합니다.7 여기서 "AE(Automotive Enhanced)" 접미사는 이 CPU가 단순한 고성능 코어가 아니라, 혼합 현실 및 기능 안전 애플리케이션을 위해 설계된 스플릿-락(split-lock)과 같은 기능을 포함하고 있음을 시사합니다.11 이는 Xen이나 Jailhouse와 같은 하이퍼바이저가 요구하는 엄격한 파티셔닝에 매우 유리한 하드웨어적 특성입니다.
- **통합 GPU (iGPU):** NVIDIA Ampere 아키텍처 기반의 GPU는 CUDA 코어와 텐서 코어를 통해 Orin의 AI 연산 능력을 책임지는 핵심 장치입니다.7 이 iGPU는 CPU, 기타 가속기와 고대역폭 LPDDR5 메모리를 공유하는 통합된 구조를 가지며 7, 이는 가상화에서 가장 어려운 과제 중 하나인 GPU 자원 분할 및 할당 문제를 야기합니다.
- **전용 가속기:** 딥러닝 가속기(NVDLA)와 프로그래머블 비전 가속기(PVA)는 특정 AI 워크로드를 효율적으로 처리하지만, iGPU와 마찬가지로 가상 게스트에게 할당하기 위한 복잡한 문제를 제시합니다.7
- **입출력(I/O):** PCIe, USB, 이더넷, MIPI CSI 등 풍부한 I/O 인터페이스는 가상 머신(VM)에 직접 할당(패스스루)되거나 가상화 계층을 통해 공유되어야 하는 자원들입니다.9

이러한 하드웨어 자원의 규모와 구성은 가상화 프로젝트의 범위와 성능을 직접적으로 결정합니다. 예를 들어, 여러 개의 리눅스 VM을 동시에 실행하려면 더 많은 CPU 코어와 메모리를 갖춘 Jetson AGX Orin 모듈이 필수적입니다.



#### 1.1.1 표 1: NVIDIA Jetson Orin 제품군 주요 사양 비교



| 모듈                     | CPU 코어                | GPU (아키텍처, CUDA 코어, 텐서 코어) | AI 성능 (TOPS) | 메모리 (크기, 타입, 대역폭) | 스토리지       | 주요 가속기            |
| ------------------------ | ----------------------- | ------------------------------------ | -------------- | --------------------------- | -------------- | ---------------------- |
| **Jetson AGX Orin 64GB** | 12코어 Arm Cortex-A78AE | Ampere, 2048, 64                     | 275            | 64GB LPDDR5, 204.8 GB/s     | 64GB eMMC      | 2x NVDLA v2, 1x PVA v2 |
| **Jetson AGX Orin 32GB** | 12코어 Arm Cortex-A78AE | Ampere, 1792, 56                     | 200            | 32GB LPDDR5, 204.8 GB/s     | 64GB eMMC      | 2x NVDLA v2, 1x PVA v2 |
| **Jetson Orin NX 16GB**  | 8코어 Arm Cortex-A78AE  | Ampere, 1024, 32                     | 100            | 16GB LPDDR5, 102.4 GB/s     | 외부 NVMe 지원 | 2x NVDLA v2            |
| **Jetson Orin NX 8GB**   | 6코어 Arm Cortex-A78AE  | Ampere, 1024, 32                     | 70             | 8GB LPDDR5, 102.4 GB/s      | 외부 NVMe 지원 | 1x NVDLA v2            |
| **Jetson Orin Nano 8GB** | 6코어 Arm Cortex-A78AE  | Ampere, 1024, 32                     | 40             | 8GB LPDDR5, 68 GB/s         | 외부 NVMe 지원 | -                      |
| **Jetson Orin Nano 4GB** | 6코어 Arm Cortex-A78AE  | Ampere, 512, 16                      | 20             | 4GB LPDDR5, 34 GB/s         | 외부 NVMe 지원 | -                      |

데이터 출처: 7



### 1.2  ARMv8-A 가상화 확장: 하이퍼바이저의 도구 상자



Jetson Orin의 하드웨어가 가상화를 지원할 수 있는 근본적인 이유는 그 기반이 되는 ARMv8-A 아키텍처에 내장된 하드웨어 가상화 확장(Virtualization Extensions) 기능 때문입니다. 이 기능들은 하이퍼바이저가 하드웨어를 직접 제어하며 여러 게스트 운영체제를 효율적이고 안전하게 관리할 수 있도록 지원합니다.

- **예외 레벨(Exception Levels, ELs):** ARMv8 아키텍처는 EL0(사용자 모드), EL1(OS 커널 모드), EL2(하이퍼바이저 모드), EL3(보안 모니터 모드)의 4단계 예외 레벨을 정의합니다.19 이것이 하드웨어 지원 가상화의 핵심입니다. 하이퍼바이저는 가장 높은 권한 수준인 EL2에서 실행되면서, 자신보다 낮은 권한 수준인 EL1/EL0에서 실행되는 게스트 OS들을 관리하고 격리합니다. 이 구조 덕분에 OS 코드를 수정하는 파라가상화(paravirtualization) 없이도 완전 가상화(full virtualization)가 가능합니다.
- **시스템 메모리 관리 장치(SMMU/IOMMU):** SMMU는 CPU가 아닌 다른 장치들(예: GPU, 네트워크 카드)이 메모리에 접근할 때 발생하는 주소(DMA, Direct Memory Access)를 변환하고 통제하는 하드웨어입니다.19 가상화 환경에서 SMMU의 역할은 절대적입니다. 하이퍼바이저는 SMMU를 이용해 특정 하드웨어 장치를 하나의 게스트 VM에 완전히 할당(패스스루)하고, 해당 VM이 허가된 메모리 영역에만 접근하도록 강제할 수 있습니다. 이는 VFIO(Virtual Function I/O)를 통한 PCIe 장치 패스스루의 기반 기술이며 20, iGPU와 같은 복잡한 통합 장치를 가상화하는 데에도 필수적입니다.
- **일반 인터럽트 컨트롤러(GICv3/v4):** GIC는 하드웨어 인터럽트를 CPU 코어로 라우팅하는 역할을 합니다. GICv3 이상에 포함된 가상화 확장은 하이퍼바이저가 하드웨어 인터럽트를 먼저 수신(trap)한 뒤, 이를 가상 인터럽트로 변환하여 목표 게스트 VM에 주입(inject)할 수 있게 해줍니다.19 이 기능이 없다면 게스트 OS는 가상 장치는 물론 패스스루된 물리 장치와도 상호작용할 수 없습니다. 실제로 커뮤니티 포럼에서 보고된 "GICv3: no GICV resource entry" 오류는 22 정확한 GIC 설정이 KVM 활성화에 얼마나 중요한지를 보여줍니다.
- **가상 호스트 확장(VHE):** VHE는 호스트 커널이 EL1이 아닌 EL2에서 직접 실행될 수 있도록 하는 최적화 기능입니다.19 이를 통해 호스트와 하이퍼바이저 간의 불필요한 컨텍스트 스위칭을 줄여 KVM과 같은 하이퍼바이저의 성능을 향상시킵니다.

이러한 하드웨어 기능들은 Jetson Orin이 가상화를 실행할 수 있는 강력한 잠재력을 가지고 있음을 명확히 보여줍니다. Cortex-A78**AE** CPU의 선택은 혼합 현실 시스템을 위한 강력한 하드웨어 격리 기능이 실리콘 수준에서부터 고려되었음을 암시합니다. 이는 사용자가 달성하고자 하는 바로 그 목표, 즉 하이퍼바이저를 통한 시스템 파티셔닝에 대한 하드웨어적 타당성을 뒷받침합니다.

하지만 이 강력한 하드웨어 잠재력과 NVIDIA의 공식 지원 정책 사이에는 명백한 모순이 존재합니다. NVIDIA 개발자 포럼에서 한 직원은 "우리는 Jetson 플랫폼에서 하이퍼바이저 모드를 지원하지 않습니다"라고 명시적으로 밝혔으며, L4T 소스 코드에 존재하는 가상화 관련 장치 트리 파일(`tegra234-soc-virt.dsti`)은 Jetson이 아닌 DRIVE 플랫폼용 코드베이스에서 파생된 것이라고 설명했습니다.1 이는 기술적 한계가 아닌, 고가의 자동차(DRIVE) 시장과 상대적으로 저렴한 엣지 컴퓨팅(Jetson) 시장에 대한 사업적/지원적 결정의 차이로 해석됩니다. 개발자에게 이것이 의미하는 바는 명확합니다. Jetson Orin에서 하이퍼바이저를 구현하는 것은 전적으로 커뮤니티의 노력과 비공식적인 방법에 의존해야 하며, 공식 문서나 기술 지원을 기대할 수 없다는 것입니다.

------



## 2.  범용 Type-1 하이퍼바이저 분석



이 장에서는 여러 개의 완전한 기능을 갖춘 운영체제를 통합하기 위해 설계된 범용 하이퍼바이저인 KVM과 Xen을 심층적으로 분석합니다. Orin 플랫폼에 대한 아키텍처 적합성과 구현의 현실적인 과제, 특히 통합 GPU(iGPU) 가상화 문제에 초점을 맞춥니다.



### 2.1  KVM (Kernel-based Virtual Machine): 리눅스 네이티브 접근 방식



- **아키텍처:** KVM은 독립적인 하이퍼바이저가 아니라, 리눅스 커널에 통합된 커널 모듈(`kvm.ko`)입니다.23 이 모듈이 로드되면 리눅스 커널 자체가 Type-1 하이퍼바이저로 동작하게 됩니다. 호스트 리눅스 커널은 EL2에서 실행되며(VHE 활용), 게스트 VM은 QEMU와 같은 사용자 공간 도구에 의해 일반적인 리눅스 프로세스처럼 관리됩니다.26 이러한 리눅스와의 긴밀한 통합은 성숙한 리눅스 스케줄러, 드라이버 모델, 메모리 관리를 활용할 수 있다는 큰 장점을 가지지만, 동시에 Jetson과 같은 특수한 환경에서는 제약 조건이 되기도 합니다.
- **Orin에서 KVM 활성화:** KVM을 Orin에서 활성화하는 과정은 L4T 커널을 특정 `CONFIG` 플래그(`CONFIG_KVM=y`, `CONFIG_VHOST_NET=m` 등)를 켜고 재컴파일하는 것을 포함합니다.28 더 중요한 것은, KVM 서브시스템이 GIC와 타이머 자원을 올바르게 인식하도록 장치 트리 블롭(DTB)을 수정해야 한다는 점입니다.28 이는 상당한 난관으로, 포럼에서는 부정확하거나 불완전한 DTB 설정으로 인해 "GICv3: no GICV resource entry"와 같은 오류를 겪는 사례가 다수 보고되었습니다.22
- **iGPU 패스스루의 난제:** 이는 Jetson에서 KVM을 사용하는 데 있어 가장 큰 장애물입니다.
  - **dGPU 대 iGPU:** 개별적인 PCIe 슬롯에 장착되는 외장 GPU(dGPU)에 잘 동작하는 표준 VFIO 기반 패스스루 방식 21이 Orin의 통합 GPU에는 직접 적용되지 않는 이유를 명확히 이해해야 합니다.32 iGPU는 분리하여 전달할 수 있는 단순한 PCIe 장치가 아니기 때문입니다. 메모리를 공유하고 시스템 버스에 복잡하게 얽혀 있어, 이를 호스트 커널로부터 완전히 분리하여 게스트에 할당하는 것은 매우 어렵습니다.
  - **AArch64에서의 `host-model` 부재:** AArch64 아키텍처의 Libvirt/QEMU는 표준 `host-model` CPU 설정을 사용할 수 없어 `host-passthrough`로 대체해야 합니다.33 이는 단일 노드 임베디드 시스템에서는 큰 문제가 아닐 수 있으나, CPU 기능의 정확한 모델링이 제한됨을 의미합니다.
  - **현재 상태:** 개발자 포럼과 커뮤니티의 논의를 종합해 볼 때, KVM을 사용하여 Jetson 플랫폼(Orin 포함)에서 하드웨어 가속이 완벽하게 지원되는 iGPU 패스스루를 안정적이고 재현 가능하게 성공시킨 공개적인 사례는 아직 없습니다. 이전 세대인 Xavier에서의 시도 29가 출발점을 제공하지만, Orin에서의 성공 경로는 여전히 불분명하며 공식적으로 지원되지 않습니다.1
- **지원 게스트 OS:** AArch64용 KVM은 다양한 리눅스 배포판, FreeBSD 등 AArch64와 호환되는 모든 운영체제를 게스트로 실행할 수 있습니다.27



### 2.2  Xen 프로젝트 하이퍼바이저: 성숙한 마이크로커널



- **아키텍처:** Xen은 마이크로커널 설계를 채택하고 있습니다.37 Xen 하이퍼바이저 자체는 하드웨어 위 EL2에서 직접 실행되며, CPU, 메모리, 인터럽트 관리와 같은 최소한의 기능만 수행합니다. 시스템을 제어하는 첫 번째 VM인 Dom0(제어 도메인)은 특권이 부여되어 있으며, 

  `xl`과 같은 관리 도구와 실제 하드웨어 드라이버를 포함합니다. 다른 비특권 게스트 VM(DomU)들은 Dom0과 나란히 실행됩니다. 이러한 '드라이버 도메인' 모델은 장치 패스스루와 I/O 가상화에 구조적으로 매우 적합합니다.

- **Orin에서 Xen 활성화 - `meta-xen` 프로젝트:** 이것이 Orin에서 Xen의 실현 가능성을 높이는 결정적인 요소입니다. `meta-xen`이라는 Yocto 레이어 2는 Jetson Orin Nano에서 Xen을 실행하기 위한 완전하고 빌드 가능한 솔루션을 제공합니다.

  - **`meta-xen` 분석:** 이 저장소는 `local.conf` 설정, 커널 구성, 부트로더(`extlinux.conf` 및 UEFI) 설정 등 Xen이 활성화된 이미지를 빌드하기 위한 명시적인 지침을 제공합니다.2 이는 KVM을 위해 수동으로 패치하고 컴파일하는 과정에 비해 엄청난 이점입니다. 

    `MACHINE_FEATURES:append = " xen"`과 같은 설정과 완전한 부트 엔트리 예제는 이미 성공적인 통합이 이루어졌음을 보여줍니다.

- **GPU 패스스루 잠재력:** Xen의 아키텍처는 iGPU 문제에 더 유연하게 대처할 수 있습니다.

  - **PCI 패스스루:** Xen은 장치 할당의 기초가 되는 PCI 패스스루(`pciback`)에 대한 성숙한 지원을 갖추고 있습니다.20
  - **`meta-xen`의 주장:** 이 레이어가 "NVIDIA GPU를 위한 장치 패스스루 설정"을 특징으로 내세운다는 점이 가장 중요한 증거입니다.2 구체적인 메커니즘이 상세히 설명되어 있지는 않지만, 이는 프로젝트가 Dom0에서 iGPU를 분리하여 DomU에 할당하는 복잡한 문제를 해결했음을 강력하게 시사합니다. 여기에는 복잡한 DTB 오버레이, 커스텀 커널 패치, 특정 부트 인자 사용 등이 포함될 가능성이 높습니다.
  - **VirtIO-GPU:** 패스스루가 아닌 시나리오를 위해, Xen은 VirtIO-GPU를 통한 파라가상화 그래픽도 지원합니다. 이는 완전한 CUDA/텐서 코어 접근은 아니지만, 2D/기본 3D 가속을 제공할 수 있습니다.41

- **지원 게스트 OS:** ARM에서의 Xen은 리눅스, Zephyr, 다양한 BSD 계열 등 광범위한 게스트 OS를 지원하여 42 시스템 통합에 높은 유연성을 제공합니다.

Xen의 드라이버 도메인(Dom0) 모델은 KVM의 모놀리식 커널 접근 방식보다 장치 가상화를 위한 더 명확한 아키텍처적 분리를 제공합니다. KVM에서는 호스트 커널이 기본적으로 모든 장치를 '소유'하므로, Orin iGPU와 같이 복잡하게 통합된 장치를 분리해내기가 어렵습니다. 반면 Xen에서 Dom0은 (특권을 가졌을 뿐) 또 하나의 VM에 불과합니다. 따라서 "Dom0에서 장치를 빼앗아 DomU에 준다"는 개념적 모델이 훨씬 자연스럽습니다. 이러한 아키텍처의 차이점은 커뮤니티가 Orin에서 KVM보다 Xen(`meta-xen`)으로 더 가시적인 성공을 거둔 이유를 설명해 줍니다.

또한, `meta-xen`이라는 Yocto 레이어의 존재는 그 자체로 강력한 이점입니다. Yocto는 커널, 부트로더, 루트 파일 시스템, 모든 구성을 포함하는 임베디드 시스템용 맞춤형 리눅스 배포판을 빌드하는 복잡성을 처리하도록 설계되었습니다. 커뮤니티가 `meta-xen` 레이어를 제공함으로써, Orin을 위한 완전하고 작동하는 Xen 이미지를 재현 가능하고 자동화된 방식으로 빌드할 수 있게 되었습니다. 이는 KVM 가이드에서 볼 수 있는 수동 패치 및 컴파일 단계의 집합보다 훨씬 견고하고 유지보수가 용이한 솔루션입니다.28 이는 Xen 커뮤니티의 노력이 더 높은 성숙도에 도달했음을 시사하며, 실제 제품 개발에 있어 중요한 장점으로 작용합니다.

------



## 3.  특수 목적 및 실시간 하이퍼바이저 분석



이 장에서는 범용 통합이 아닌, 실시간 결정성 및 고신뢰성 보안과 같은 특정 사용 사례를 위해 설계된 하이퍼바이저에 초점을 맞춥니다. "언제 KVM과 Xen이 부적합한 도구이며, 그 대안으로 무엇을 사용해야 하는가?"라는 질문에 답하는 것을 목표로 합니다.



### 3.1  Jailhouse: 혼합 현실을 위한 정적 파티셔닝 하이퍼바이저



- **아키텍처:** Jailhouse는 "정적 파티셔닝" 하이퍼바이저입니다.3 범용 가상화 도구와는 근본적으로 다릅니다. 이 하이퍼바이저는 리눅스가 부팅된 *후에* 로드되어, 실행 중인 시스템에서 하드웨어 자원(CPU 코어, 메모리 영역, 장치 MMIO 범위)을 그대로 '분할'하여 '셀(cell)'이라는 격리된 공간을 만듭니다. Jailhouse는 스케줄링, 자원 초과 할당(overcommitment), 복잡한 장치 에뮬레이션을 수행하지 않습니다. 그 목표는 단순성과 최소한의 간섭입니다.3
- **실시간 성능:** Jailhouse의 주된 사용 사례는 하드 실시간 운영체제(RTOS)나 베어메탈 애플리케이션을 하나의 셀에서 실행시켜, 비실시간 운영체제인 리눅스 '루트 셀'로부터 완벽하게 격리하는 것입니다.4 CPU 코어와 주변 장치를 '인메이트(inmate)'에게 독점적으로 할당함으로써, Jailhouse는 범용 OS가 보장하기 어려운 극도로 낮고 예측 가능한 인터럽트 지연 시간을 달성할 수 있습니다.4
- **Jetson에서의 구현:** 메인 Jailhouse 저장소는 이전 세대인 Jetson TX1/TX2 보드를 지원 목록에 포함하고 있으며 3, `linux-jailhouse-jetson`이라는 별도의 포크는 특정 커널 버전과 구성으로 이들 플랫폼을 지원합니다.50 제공된 자료에서는 Orin에 대한 직접적인 포팅 성공 사례는 없지만, 그 원리는 이전 가능합니다. Orin 포팅 작업에는 다음이 포함될 것입니다:
  - Orin SoC를 위한 새로운 보드 구성 파일 생성.
  - 부트로더 인자를 통해 연속적인 메모리 영역 예약.
  - Jailhouse 지원을 활성화하여 L4T 커널 재컴파일.
  - 특정 UART나 GPIO 컨트롤러를 인메이트 셀에 할당하는 것과 같은 장치 할당 작업.50
- **지원 "인메이트":** Jailhouse는 베어메탈 애플리케이션, RTOS, 심지어는 또 다른 리눅스 인스턴스를 셀 안에서 실행하도록 설계되었습니다.52



### 3.2  seL4: 고신뢰성 시스템을 위한 형식 검증 마이크로커널



- **아키텍처와 보안 보증:** seL4의 가장 큰 특징은 '형식 검증'입니다. 이는 커널의 C 코드가 사양을 정확하게 구현했음을 수학적으로 증명한 것을 의미합니다.6 이는 커널 자체의 버그에 대해 타의 추종을 불허하는 수준의 신뢰성을 제공합니다. seL4는 모든 자원 접근에 대해 '기본 거부(default-deny)' 정책을 강제하는 역량 기반(capability-based) 보안 모델을 사용합니다.6
- **실시간 및 혼합 현실:** seL4는 혼합 현실 시스템(MCS) 확장을 통해 보장된 스케줄링 예산과 주기를 제공하여, 중요한 작업과 비중요 작업을 검증된 시간적 격리 하에 공존시킬 수 있습니다.6 이는 안전이 중요한 애플리케이션에서 Jailhouse의 경쟁자가 될 수 있음을 의미합니다.
- **Orin 포팅 및 실현 가능성:** 공식 seL4 지원 목록에는 Jetson TX1과 TX2가 포함되어 있어 5, seL4 커뮤니티가 Tegra SoC 제품군에 대한 경험을 가지고 있음을 알 수 있습니다. 그러나 Orin은 아직 목록에 없습니다. seL4를 새로운 플랫폼에 포팅하는 것은 상당한 노력이 필요한 작업으로, elfloader 수정, 타이머 및 인터럽트 컨트롤러 드라이버 추가, 빌드 시스템 수정 등이 요구됩니다.55 새로운 Orin 포트에 대해 기존의 증명이 즉시 적용되지는 않겠지만, 그 기반이 되는 고신뢰성 커널 코드는 일반 커널에 비해 여전히 높은 수준의 보증을 제공합니다.55
- **가상화:** seL4는 ARM HYP 모드를 지원하여 게스트 VM을 호스팅하는 하이퍼바이저 역할을 할 수 있습니다.5 이를 통해 검증된 seL4 커널 위에서 리눅스와 같은 완전한 OS를 VM으로 실행하는 시스템을 구축할 수 있습니다.

이 네 가지 하이퍼바이저는 격리의 보증 수준과 복잡성이라는 측면에서 하나의 스펙트럼을 형성합니다. KVM과 Xen은 범용 OS를 위한 **기능적 격리**를 제공하여 VM들이 서로를 다운시키지 않도록 하는 데 중점을 둡니다. 반면 Jailhouse는 실시간 작업을 위한 **시간적 격리**를 제공하여, 리눅스가 RTOS의 데드라인을 놓치게 만드는 것을 방지하는 것이 목표입니다.4 마지막으로 seL4는 

**증명 가능한 보안 격리**를 제공하여, 한 컴포넌트의 악의적인 코드가 다른 컴포넌트를 침해하는 것을 수학적 확실성으로 방지하고자 합니다.6 이는 사용자의 시스템 요구사항에 따라 명확한 의사결정 프레임워크를 제공합니다.

또한, 이들 하이퍼바이저는 활성화 방식에서도 중요한 차이를 보입니다. Xen과 seL4는 부트로더가 커널 대신 자신을 먼저 로드하는 고전적인 Type-1 하이퍼바이저입니다. 반면 Jailhouse는 완전한 리눅스 시스템이 부팅된 후, 런타임에 커널 모듈(`jailhouse.ko`)로 로드되어 시스템을 파티셔닝하는 독특한 모델을 가집니다.3 이러한 Jailhouse의 접근 방식은 개발과 디버깅에 있어 실용적인 이점을 제공합니다. 언제든지 안정적인 리눅스 시스템으로 돌아갈 수 있으며, 부트로더를 건드리는 복잡한 과정 없이 파티셔닝을 활성화/비활성화할 수 있습니다. 이는 기존 리눅스 기반 Jetson 프로젝트에 점진적으로 실시간 기능을 추가하고자 할 때, Xen이나 seL4에 비해 훨씬 낮은 진입 장벽을 제공합니다.

------



## 4.  비교 분석 및 전략적 권고



이 마지막 장에서는 모든 분석 결과를 종합하여 직접적인 비교를 제공하고, 명확하며 사용 사례에 기반한 권고안을 제시합니다. "이 모든 정보를 고려할 때, 내 프로젝트에 어떤 하이퍼바이저를 선택해야 하는가?"라는 최종 질문에 답하는 것을 목표로 합니다.



### 4.1  기능 및 구현 비교 매트릭스



다음 표는 본 보고서의 분석 결과를 요약하여 네 가지 하이퍼바이저 옵션을 다각적으로 비교할 수 있는 의사결정 도구를 제공합니다.



#### 4.1.1 표 2: 하이퍼바이저 종합 비교 매트릭스



| 비교 기준              | KVM                           | Xen                                 | Jailhouse                        | seL4                                   |
| ---------------------- | ----------------------------- | ----------------------------------- | -------------------------------- | -------------------------------------- |
| **하이퍼바이저 타입**  | 리눅스 커널 모듈 (Type-1)     | 마이크로커널 (Type-1)               | 정적 파티셔닝 하이퍼바이저       | 형식 검증 마이크로커널 (Type-1)        |
| **주요 사용 사례**     | 범용 서버/데스크톱 가상화     | 서버 통합, 클라우드, 임베디드       | 혼합 현실, 실시간 시스템         | 고신뢰성 보안/안전 시스템              |
| **NVIDIA 공식 지원**   | 없음                          | 없음                                | 없음                             | 없음                                   |
| **Orin 커뮤니티 지원** | 낮음 (커널/DTB 패치 필요)     | 높음 (`meta-xen` Yocto 레이어 존재) | 없음 (TX1/TX2 기반 포팅 필요)    | 없음 (TX1/TX2 기반 포팅 필요)          |
| **구현 용이성**        | 중간 (커널 컴파일)            | 높음 (Yocto 빌드 자동화)            | 낮음 (포팅 및 수동 설정)         | 매우 낮음 (포팅 및 복잡한 시스템 설계) |
| **CPU 가상화**         | 우수 (리눅스 네이티브)        | 우수 (성숙한 구현)                  | 정적 파티셔닝 (가상화 아님)      | 우수 (증명된 격리)                     |
| **iGPU 가상화 상태**   | 불안정 (커뮤니티 해결책 부재) | 유망 (`meta-xen`에서 지원 주장)     | 미지원 (장치 직접 할당 모델)     | 미지원 (포팅 필요)                     |
| **실시간 성능**        | 낮음 (범용 스케줄러)          | 중간 (실시간 스케줄러 사용 가능)    | 매우 우수 (결정론적, 최소 간섭)  | 우수 (MCS 스케줄링 보장)               |
| **보안 보증**          | 표준 리눅스 수준              | 높음 (마이크로커널 설계)            | 높음 (엄격한 정적 격리)          | 최상 (형식 검증 완료)                  |
| **게스트 OS 지원**     | 광범위 (AArch64 호환 OS)      | 광범위 (Linux, BSD, Zephyr 등)      | 제한적 (Linux, RTOS, Bare-metal) | 제한적 (Linux, CAmkES 컴포넌트)        |
| **라이선스**           | GPLv2 (커널), LGPL (QEMU)     | GPLv2                               | GPLv2                            | GPLv2                                  |



### 4.2  사용 사례별 적합성: 임무에 맞는 하이퍼바이저 선택



- **시나리오 1: AI 개발 및 다중 사용자 환경**
  - **설명:** 단일 AGX Orin 장치에서 여러 개발자나 프로젝트를 위해 다수의 리눅스 환경을 통합해야 하는 경우.
  - **권고:** **Xen**
  - **근거:** `meta-xen`을 통한 Orin 지원이 가장 강력하며 2, 아키텍처적으로 iGPU 패스스루 문제 해결에 가장 적합합니다. 범용 게스트 OS에 대한 지원이 성숙하여 42 다중 개발 환경 구축에 유리합니다.
- **시나리오 2: 혼합 현실 로보틱스**
  - **설명:** 자율 로봇이 리눅스에서 ROS 2 인식 스택을 실행하는 동시에, RTOS(예: Zephyr)나 베어메탈 환경에서 하드 실시간 모터 제어/안전 루프를 실행해야 하는 경우.
  - **권고:** **Jailhouse**
  - **근거:** 정적 파티셔닝과 시간적 격리를 위해 특별히 설계되었습니다.3 유사한 ARM SoC에서 뛰어난 실시간 성능을 제공함이 입증되었으며 48, 이 특정 작업을 위해 Xen과 같은 완전한 기능의 하이퍼바이저보다 구현 및 관리가 단순합니다.
- **시나리오 3: 고신뢰성 항공우주/국방 시스템**
  - **설명:** 보안이 침해된 컴포넌트가 비행 제어와 같은 핵심 시스템에 영향을 미쳐서는 안 되는 보안 드론이나 지상 차량.
  - **권고:** **seL4**
  - **근거:** 형식 검증과 증명 가능한 격리 보증을 제공하는 유일한 옵션입니다.6 역량 기반 보안 모델은 설계 단계부터 안전한 시스템을 구축하는 데 이상적입니다. 극도의 보안 요구사항은 seL4의 성능 및 개발 오버헤드를 정당화합니다.
- **시나리오 4: 최소한의 노력으로 가상화 실험**
  - **설명:** 연구원이나 취미 개발자가 Orin에서 가상화를 실험해보고자 하는 경우.
  - **권고:** **KVM** 커널 컴파일로 시작.
  - **근거:** iGPU 패스스루는 큰 장애물이지만, 단순히 KVM을 실행하고 기본적인 커맨드 라인 리눅스 게스트를 부팅하는 것은 "단지" 커널 재컴파일과 DTB 수정만으로 가능하여 가장 직접적인 경로를 제공합니다.28 이는 표준 리눅스 도구를 활용하며, 완전한 Xen/Yocto 빌드 환경을 설정하는 것보다 개념적 진입 장벽이 낮습니다. 이를 통해 더 복잡한 솔루션에 전념하기 전에 좋은 학습 경험을 얻을 수 있습니다.



### 4.3  최종 권고 및 향후 전망



본 분석을 통해 Jetson Orin 플랫폼에서의 베어메탈 하이퍼바이저 선택은 "최고"의 솔루션을 찾는 것이 아니라, "최적"의 솔루션을 찾는 과정임이 명확해졌습니다. 각 하이퍼바이저는 뚜렷한 강점과 약점을 가지며, 프로젝트의 핵심 요구사항(성능, 실시간성, 보안)에 따라 선택이 달라져야 합니다.

VM 내부에서 GPU 가속이 필요한 모든 프로젝트의 경우, 가장 중요한 과제는 iGPU 패스스루 문제를 해결하는 것입니다. 현재로서는 `meta-xen` 프로젝트 2가 커뮤니티 전체에서 가장 유망한 출발점을 제공하고 있습니다.

NVIDIA의 공식 지원이 부재한 상황에서 1, 모든 진전은 전적으로 오픈 소스 커뮤니티의 협력에 달려 있습니다. 

`meta-xen`과 같은 프로젝트를 지원하고, KVM을 위한 DTB 구성을 공유하며, Jailhouse/seL4를 Orin에 포팅하는 노력은 Jetson Orin 플랫폼의 가상화 잠재력을 최대한 발휘하는 데 매우 중요합니다.

엣지에서의 가상화와 워크로드 통합의 중요성이 계속해서 커지고 있으므로, 커뮤니티의 수요가 증가하면 NVIDIA가 DRIVE 플랫폼과 유사하게 Jetson에 대해서도 더 많은 공식 지원을 제공할 가능성을 배제할 수 없습니다. 또한 Ampere 서버에서의 Xen 지원과 같은 42 더 넓은 ARM 가상화 생태계의 발전 역시 Jetson 사용자들에게 긍정적인 영향을 미칠 것입니다.



#### **참고 자료**

1. Nvidia Jetson AGX Orin VM in hypervisor mode, accessed July 15, 2025, https://forums.developer.nvidia.com/t/nvidia-jetson-agx-orin-vm-in-hypervisor-mode/261056
2. Meta-xen layer - GitHub, accessed July 15, 2025, https://github.com/KPGURAV10/meta-xen
3. siemens/jailhouse: Linux-based partitioning hypervisor - GitHub, accessed July 15, 2025, https://github.com/siemens/jailhouse
4. Virtualization: Jailhouse Hypervisor on AM572x Reference Design - Texas Instruments, accessed July 15, 2025, https://www.ti.com/lit/pdf/tidudf8
5. Supported platforms - seL4 Docs, accessed July 15, 2025, https://docs.sel4.systems/Hardware/
6. seL4 - Open Source Real-Time Operating Systems (RTOS) - OSRTOS, accessed July 15, 2025, https://www.osrtos.com/rtos/sel4/
7. Jetson Modules, Support, Ecosystem, and Lineup | NVIDIA Developer, accessed July 15, 2025, https://developer.nvidia.com/embedded/jetson-modules
8. jetson-orin-nano-datasheet-r4-web.pdf - Open Zeka, accessed July 15, 2025, https://openzeka.com/wp-content/uploads/2023/03/jetson-orin-nano-datasheet-r4-web.pdf
9. NVIDIA Jetson AGX Orin Developer Kit | Datasheet, accessed July 15, 2025, https://assets.alliedelec.com/v1670929267/Datasheets/6025cdcab30cf3284777cae5276ec75b.pdf
10. Welcome - NVIDIA Jetson Linux Developer Guide 1 documentation, accessed July 15, 2025, https://docs.nvidia.com/jetson/archives/r35.5.0/DeveloperGuide/index.html
11. NVIDIA Jetson AGX Orin - OpenZeka, accessed July 15, 2025, https://openzeka.com/wp-content/uploads/2022/02/Jetson_AGX_Orin_DS-10662-001_v0.2.pdf
12. NVIDIA Jetson Orin NX Series, accessed July 15, 2025, https://developer.nvidia.com/downloads/jetson-orin-nx-series-data-sheet
13. NVIDIA Jetson AGX Orin GPU: Technical Specs, Features, and Use Cases - server-parts.eu, accessed July 15, 2025, https://www.server-parts.eu/post/nvidia-jetson-agx-orin-gpu-specs
14. NVIDIA Jetson Orin Developer Kit - Firefly, accessed July 15, 2025, https://en.t-firefly.com/nv/developer
15. NVIDIA Jetson AGX Orin 64 GB Specs | TechPowerUp GPU Database, accessed July 15, 2025, https://www.techpowerup.com/gpu-specs/jetson-agx-orin-64-gb.c4085
16. NVIDIA Jetson AGX Orin Series, accessed July 15, 2025, https://www.nvidia.com/content/dam/en-zz/Solutions/gtcf21/jetson-orin/nvidia-jetson-agx-orin-technical-brief.pdf
17. NVIDIA Jetson Orin Nano Developer Kit - Seeed Studio, accessed July 15, 2025, https://files.seeedstudio.com/wiki/Jetson-Orin-Nano-DevKit/jetson-orin-nano-developer-kit-datasheet.pdf
18. NVIDIA® Jetson Orin™ NX Edge AI Computing - Specifications - NEXCOM, accessed July 15, 2025, https://www.nexcom.com/Products/multi-media-solutions/ai-edge-computer/nvidia-solutions/aiedge-x-80/Specifications
19. Introduction to the ARMv8 Virtualization System | openEuler, accessed July 15, 2025, https://www.openeuler.org/en/blog/yorifang/2020-10-24-arm-virtualization-overview.html
20. Xen VGA Passthrough - Xen Project Wiki, accessed July 15, 2025, https://wiki.xenproject.org/wiki/Xen_VGA_Passthrough
21. bryansteiner/gpu-passthrough-tutorial - GitHub, accessed July 15, 2025, https://github.com/bryansteiner/gpu-passthrough-tutorial
22. Boot a VM on an NVIDIA Jetson AGX Orin - Cloudkernels, accessed July 15, 2025, https://blog.cloudkernels.net/posts/orin-vm/
23. How is KVM a bare metal Hypervisor? : r/kernel - Reddit, accessed July 15, 2025, https://www.reddit.com/r/kernel/comments/1854y3r/how_is_kvm_a_bare_metal_hypervisor/
24. Is it possible to install KVM on bare metal? - Unix & Linux Stack Exchange, accessed July 15, 2025, https://unix.stackexchange.com/questions/269049/is-it-possible-to-install-kvm-on-bare-metal
25. KVM: Bare-Metal Hypervisor? - Virtualization Review, accessed July 15, 2025, https://virtualizationreview.com/blogs/mental-ward/2009/02/kvm-baremetal-hypervisor.aspx
26. guide to set up a KVM development environment on 64-bit ARMv8 processors, accessed July 15, 2025, http://www.virtualopensystems.com/en/solutions/guides/kvm-on-armv8/
27. Arm System emulator - QEMU documentation, accessed July 15, 2025, https://www.qemu.org/docs/master/system/target-arm.html
28. lattice0/jetson_nano_kvm: How to rebuild the kernel to activate KVM on the Nvidia Jetson Nano - GitHub, accessed July 15, 2025, https://github.com/lattice0/jetson_nano_kvm
29. Guide to enable KVM on the Xavier - Jetson AGX Xavier - NVIDIA Developer Forums, accessed July 15, 2025, https://forums.developer.nvidia.com/t/guide-to-enable-kvm-on-the-xavier/119777
30. How to enable GPU passthrough with KVM/QEMU ? : r/pop_os - Reddit, accessed July 15, 2025, https://www.reddit.com/r/pop_os/comments/188f1ym/how_to_enable_gpu_passthrough_with_kvmqemu/
31. Qemu KVM with GPU passthrough on two nvidia GPUs in a wayland session. - Reddit, accessed July 15, 2025, https://www.reddit.com/r/linuxquestions/comments/gk7c82/qemu_kvm_with_gpu_passthrough_on_two_nvidia_gpus/
32. Virtualization for PCIe? - Jetson Orin Nano - NVIDIA Developer Forums, accessed July 15, 2025, https://forums.developer.nvidia.com/t/virtualization-for-pcie/260536
33. libvirt: use 'host-passthrough' as default on AArch64 / 8bc7b950b7 - nova - OpenDev, accessed July 15, 2025, https://opendev.org/openstack/nova/commit/8bc7b950b7c0a3c80cdd120fe4df97c14848c344
34. GPU passthrough virtualisation on AGX orin with kvm - NVIDIA Developer Forums, accessed July 15, 2025, https://forums.developer.nvidia.com/t/gpu-passthrough-virtualisation-on-agx-orin-with-kvm/322467
35. KVM/arm64-specific hypercalls exposed to guests - The Linux Kernel documentation, accessed July 15, 2025, https://docs.kernel.org/virt/kvm/arm/hypercalls.html
36. QEMU Emulation for ARM AArch64 Virt KVM - Zephyr Project Documentation, accessed July 15, 2025, https://docs.zephyrproject.org/latest/boards/qemu/kvm_arm64/doc/index.html
37. Virtualization on ARM with Xen, accessed July 15, 2025, https://xenproject.org/blog/virtualization-on-arm-with-xen/
38. Hypervisor | Xen Project, accessed July 15, 2025, https://xenproject.org/projects/hypervisor/
39. The Bare-Metal Hypervisor as a Platform for Innovation - Xen Project, accessed July 15, 2025, https://xenproject.org/blog/the-bare-metal-hypervisor-as-a-platform-for-innovation/
40. Looking for tips: building a workstation around Xen and GPU pass-through - Super User, accessed July 15, 2025, https://superuser.com/questions/284779/looking-for-tips-building-a-workstation-around-xen-and-gpu-pass-through
41. VirtIO GPU and Passthrough GPU Support for Xen - Ray Huang, AMD - YouTube, accessed July 15, 2025, https://m.youtube.com/watch?v=o2cltCpRDtU
42. Xen on Ampere: A New Era for ARM in the Data Center - XCP-ng, accessed July 15, 2025, https://xcp-ng.org/blog/2025/03/18/xen-on-ampere-new-era/
43. Xen Hypervisor - Arm Developer, accessed July 15, 2025, https://developer.arm.com/documentation/110457/0100/Xen-Hypervisor
44. How to Support Your OS as Xen on ARM Guest - Julien Grall, Linaro - YouTube, accessed July 15, 2025, https://www.youtube.com/watch?v=lNJFTgv5YPU
45. 3.8.1. Jailhouse Hypervisor - Processor SDK Linux Automotive Documentation - http, accessed July 15, 2025, https://software-dl.ti.com/jacinto7/esd/processor-sdk-linux-jacinto7/06_01_01_02/exports/docs/linux/Foundational_Components/Virtualization/Jailhouse.html
46. 3.14.1. Jailhouse Hypervisor - Processor SDK Linux Documentation - http, accessed July 15, 2025, https://software-dl.ti.com/processor-sdk-linux/esd/docs/06_02_00_81/linux/Foundational_Components/Virtualization/Jailhouse.html
47. BU-maintained version of the Jailhouse partitioning hypervisor with real-time features - GitHub, accessed July 15, 2025, https://github.com/rntmancuso/jailhouse-rt
48. Building Mixed Criticality Linux Systems with the Jailhouse Hypervisor - Ralf Ramsauer, accessed July 15, 2025, https://www.youtube.com/watch?v=pvs0fv-gnvw
49. Real Time Applications On Jetson TX1 With Jailhouse Hypervisor - YouTube, accessed July 15, 2025, https://www.youtube.com/watch?v=RXV8L2YHHPw
50. Jailhouse hypervisor for Nvidia Jetson TX1 and TX2 - GitHub, accessed July 15, 2025, https://github.com/evidence/linux-jailhouse-jetson
51. Real Time Applications on Jetson with Jailhouse Hypervisor - Embien Technologies, accessed July 15, 2025, https://www.embien.com/blog/real-time-applications-on-jetson-with-jailhouse-hypervisor
52. 3.7.1. Jailhouse Hypervisor - Processor SDK Linux for J721e Documentation - http, accessed July 15, 2025, https://software-dl.ti.com/jacinto7/esd/processor-sdk-linux-jacinto7/07_03_00_05/exports/docs/linux/Foundational_Components/Virtualization/Jailhouse.html
53. jailhouse/README.md at master - GitHub, accessed July 15, 2025, https://github.com/siemens/jailhouse/blob/master/README.md
54. seL4 Reference Manual Version 13.0.0, accessed July 15, 2025, https://sel4.systems/Info/Docs/seL4-manual-latest.pdf
55. How can I port seL4 to a new ARM hardware platform?, accessed July 15, 2025, https://sel4.discourse.group/t/how-can-i-port-sel4-to-a-new-arm-hardware-platform/15

