---
layout: page
title: KVM(Kernel-based Virtual Machine)에 대한 아키텍처 및 기술
permalink: /systems/hyper-visor/KVM에 대한 아키텍처 및 기술
---



KVM(Kernel-based Virtual Machine)은 Linux 커널을 Type-1(Bare-metal) 하이퍼바이저로 변환하는 오픈 소스 가상화 기술이다.1 이는 KVM이 독립적인 소프트웨어 제품이 아니라, Linux 커널 자체에 내재된 하나의 기능 모듈임을 의미한다.4 따라서 현대의 모든 Linux 배포판은 별도의 하이퍼바이저 설치 없이도 잠재적으로 강력한 가상화 호스트가 될 수 있는 능력을 갖추고 있다.2

KVM은 가상 머신(VM)을 생성하고 관리하는 핵심 기반을 제공하며, 각 VM은 호스트 시스템에서 하나의 독립된 Linux 프로세스로 실행된다.1 이 독특한 아키텍처 덕분에 KVM은 수십 년간 검증되고 최적화된 Linux 커널의 핵심 기능들, 즉 프로세스 스케줄러, 메모리 관리자, 네트워크 스택, 보안 모듈(SELinux 등)을 그대로 상속받아 활용한다.1 이러한 특성은 KVM에 높은 수준의 안정성, 성능, 그리고 보안을 부여하는 근간이 된다.

오늘날 KVM은 Red Hat Virtualization, OpenStack, Proxmox VE와 같은 주요 오픈 소스 가상화 및 클라우드 플랫폼의 핵심 엔진으로 작동하며, 사실상 오픈 소스 클라우드 인프라의 표준 하이퍼바이저로 자리매김하였다.1


가상화 기술의 역사는 1960년대 IBM 메인프레임에서 시작되었으나, x86 아키텍처에서의 본격적인 가상화는 2000년대 초반 VMware의 등장과 함께 시작되었다.12 당시 오픈 소스 진영에서는 Xen이 사실상의 표준 하이퍼바이저로 군림하고 있었다.13 Xen은 하드웨어 가상화 지원이 보편화되기 이전에 개발되어, 게스트 커널을 수정해야 하는 반가상화(Paravirtualization) 방식에 크게 의존했다. 또한, 시스템 전체를 제어하는 특권 가상 머신인 'Dom0'를 두는 독특한 마이크로커널 아키텍처를 채택했는데, 이는 구조적 복잡성을 증가시키는 요인이었다.13

이러한 기술적 배경 속에서 2005년과 2006년, Intel(VT-x)과 AMD(AMD-V)가 CPU에 하드웨어 가상화 확장 기능을 본격적으로 도입하기 시작했다.15 이 기술적 변곡점은 가상화의 패러다임을 바꿀 잠재력을 지니고 있었다. 하드웨어가 직접 민감한 명령어를 처리하고 가상 머신 상태 전환(VM-exit/entry)을 관리하게 되면서, 더 이상 복잡한 바이너리 변환이나 게스트 커널 수정 없이도 높은 성능을 달성할 수 있는 길이 열린 것이다.13

2006년, 이스라엘의 스타트업 Qumranet에 소속된 개발자 Avi Kivity는 Xen의 복잡성에 대한 대안으로 이 새로운 하드웨어 기능을 최대한 활용하는 방안을 모색했다.13 그의 접근법은 혁신적이면서도 지극히 실용적이었다. 새로운 하이퍼바이저를 처음부터 만드는 대신, 이미 세계에서 가장 성숙하고 안정적인 운영체제 커널인 Linux 커널 자체를 하이퍼바이저로 만들자는 것이었다. 이 아이디어가 바로 KVM의 시작이었다.

KVM은 2006년 10월 처음 공개되었고, 불과 몇 달 만인 2007년 2월 5일, Linux 커널 2.6.20 버전에 공식적으로 병합되었다.4 이는 가상화 기능이 더 이상 외부의 특수한 소프트웨어가 아닌, 운영체제의 핵심 기능으로 통합되는 중요한 전환을 의미했다. KVM의 등장은 기술적 필연성의 산물이었다. 이는 CPU 하드웨어의 발전, 기존 기술의 한계에 대한 반작용, 그리고 Linux 커널의 성숙도를 최대한 재사용하려는 실용주의적 철학이 완벽하게 결합된 결과물이다. 이로써 KVM은 Linux의 활용 범위를 서버 운영체제에서 클라우드 인프라의 근간으로 확장시키는 결정적인 계기를 마련했다.


KVM의 아키텍처는 "역할 분담을 통한 최적화"라는 철학을 명확히 보여준다. 성능이 결정적인 CPU 및 메모리 가상화는 커널과 하드웨어에 위임하고, 유연성과 호환성이 중요한 I/O 장치 에뮬레이션은 사용자 공간의 QEMU에 맡기는 분리 구조는 KVM의 기술적 특징과 성공의 근원이다.


KVM의 가장 근본적인 아키텍처 특징은 Linux 커널과의 완전한 통합이다. KVM은 독립적인 하이퍼바이저 프로그램이 아니라, 커널에 동적으로 적재 가능한 모듈 세트로 구현된다. 핵심 모듈인 `kvm.ko`는 아키텍처에 독립적인 가상화 인프라를 제공하며, `kvm-intel.ko` 또는 `kvm-amd.ko`와 같은 프로세서별 모듈이 실제 하드웨어 제어를 담당한다.4

이 모듈들이 로드되면, Linux 커널은 하드웨어 가상화 확장 기능을 직접 제어할 수 있는 완전한 Type-1(Bare-metal) 하이퍼바이저로 변모한다.2 이 상태에서 생성된 각 가상 머신은 놀랍게도 일반적인 Linux 프로세스로 취급된다. 각 가상 CPU(vCPU)는 해당 프로세스 내의 스레드(thread)에 일대일로 매핑되어, Linux의 표준 스케줄러인 CFS(Completely Fair Scheduler)에 의해 다른 일반 프로세스들과 동등하게 스케줄링된다.1

이러한 설계는 KVM에 막대한 이점을 제공한다. KVM은 별도의 메모리 관리자, 프로세스 스케줄러, I/O 스택, 디바이스 드라이버, 보안 프레임워크를 개발할 필요 없이, 수십 년간 전 세계 개발자들에 의해 검증되고 최적화된 Linux 커널의 자산을 그대로 활용할 수 있다.1 결과적으로 KVM은 매우 작고 효율적인 코드베이스를 유지하면서도, Linux 커널의 모든 발전(성능 개선, 보안 패치, 새로운 하드웨어 지원)의 혜택을 즉시 누릴 수 있는 독보적인 위치를 차지하게 된다.


KVM은 소프트웨어 에뮬레이션에 의존하지 않으며, 반드시 CPU에서 제공하는 하드웨어 가상화 확장 기능을 요구한다.4 Intel의 VT-x(Virtualization Technology)와 AMD의 AMD-V(AMD Virtualization)가 대표적인 예이다.

이 확장 기능들은 CPU에 새로운 실행 모드를 도입한다. Intel의 경우, 하이퍼바이저가 실행되는 'VMX root operation' 모드와 게스트 VM이 실행되는 'VMX non-root operation' 모드로 나뉜다. 게스트가 non-root 모드에서 실행되다가 `IN/OUT` 명령어와 같은 입출력 명령이나 `HLT`와 같은 시스템 상태 변경 명령어 등 민감한 명령어를 실행하면, CPU는 자동으로 실행을 중단하고 'VM-exit'이라는 이벤트를 발생시킨다.13 이 이벤트는 제어권을 하이퍼바이저(KVM)에게 안전하게 넘겨주는 하드웨어 메커니즘이다. KVM은 VM-exit의 원인을 파악하여 필요한 작업을 처리한 후, 'VM-entry'를 통해 다시 게스트의 실행을 재개한다.

또한, EPT(Extended Page Tables, Intel)나 RVI(Rapid Virtualization Indexing, AMD)와 같은 2단계 주소 변환(SLAT, Second Level Address Translation) 기능은 메모리 가상화의 성능을 극적으로 향상시킨다.15 하이퍼바이저가 소프트웨어적으로 관리하던 섀도우 페이지 테이블(shadow page table) 대신, 하드웨어 MMU가 직접 게스트 가상 주소(GVA)에서 호스트 물리 주소(HPA)로의 변환을 처리함으로써 메모리 접근 오버헤드를 크게 줄인다. KVM은 이러한 하드웨어 기능을 적극적으로 활용하여 거의 네이티브에 가까운 성능을 달성한다.


KVM의 아키텍처를 이해하는 데 있어 가장 중요한 부분 중 하나는 QEMU(Quick Emulator)와의 관계이다. KVM 커널 모듈 자체는 오직 CPU와 메모리의 가상화, 즉 게스트 코드의 안전하고 빠른 실행에만 집중한다.16 디스크 컨트롤러, 네트워크 카드, USB 포트, 그래픽 어댑터와 같은 수많은 종류의 하드웨어 장치에 대한 에뮬레이션 기능은 KVM에 포함되어 있지 않다.

이 중요한 역할은 사용자 공간(userspace)에서 실행되는 QEMU 프로세스가 담당한다.16 QEMU는 원래 하드웨어 가상화 지원 없이 순수 에뮬레이션만으로 동작하는 독립적인 프로젝트였으나, KVM의 등장 이후 KVM의 가속기(accelerator) 역할을 하는 파트너가 되었다. 오늘날 KVM 기반 가상화는 사실상 "KVM/QEMU" 스택을 의미한다.

이러한 역할 분담은 매우 효율적이다. KVM은 커널 공간에서 하드웨어와 직접 상호작용하며 성능이 중요한 CPU 및 메모리 가상화를 처리한다. 반면, QEMU는 사용자 공간에서 다양한 종류의 가상 하드웨어를 에뮬레이션하며 높은 호환성과 유연성을 제공한다.23 VM은 QEMU 프로세스 내부에서 실행되며, 각 vCPU는 QEMU의 스레드로 존재한다. 이 스레드가 게스트의 일반적인 코드를 실행할 때는 QEMU가 KVM에 제어권을 넘겨 하드웨어 가상화 모드에서 직접 실행시키고, 게스트가 가상 I/O 장치에 접근할 때는 VM-exit을 통해 제어권이 QEMU로 돌아와 해당 장치 동작을 소프트웨어적으로 에뮬레이션한다.26


KVM 커널 모듈과 사용자 공간의 QEMU를 연결하는 다리는 `/dev/kvm`이라는 문자 디바이스(character device) 파일이다.16 QEMU와 같은 가상 머신 모니터(VMM)는 이 파일을 `open()` 시스템 콜로 열어 KVM 서브시스템을 제어할 수 있는 파일 디스크립터(file descriptor, fd)를 획득한다.29

이 fd를 통해 이루어지는 모든 통신은 `ioctl` (Input/Output Control) 시스템 콜을 통해 수행된다. 이 API는 계층적 구조를 가진다.26

1. **시스템 `ioctl`:** `/dev/kvm` fd에 직접 `ioctl`을 호출한다. 가장 중요한 것은 `KVM_CREATE_VM`으로, 새로운 가상 머신을 생성하고 해당 VM을 제어하기 위한 새로운 fd(VM fd)를 반환한다.
2. **VM `ioctl`:** VM fd에 `ioctl`을 호출한다. `KVM_CREATE_VCPU`는 VM에 가상 CPU를 추가하고 vCPU를 제어하기 위한 fd(vCPU fd)를 반환한다. `KVM_SET_USER_MEMORY_REGION`은 게스트의 물리 주소 공간(GPA)을 QEMU 프로세스의 가상 주소 공간(HVA)에 매핑하여 메모리를 할당한다.
3. **vCPU `ioctl`:** vCPU fd에 `ioctl`을 호출한다. 가장 핵심적인 `ioctl`은 `KVM_RUN`이다. QEMU의 vCPU 스레드가 이 `ioctl`을 호출하면, 커널은 CPU를 게스트 모드로 전환(VM-entry)하여 게스트 코드를 실행시킨다.

게스트 코드 실행 중 VM-exit이 발생하면, CPU는 다시 호스트 모드로 전환되고 `KVM_RUN` 호출이 반환된다. 이때 `kvm_run`이라는 공유 메모리 구조체를 통해 QEMU에게 VM-exit이 발생한 이유(exit reason, 예: `KVM_EXIT_IO`)와 관련 정보(예: I/O 포트 번호, 데이터)가 전달된다.26 QEMU는 이 정보를 바탕으로 필요한 에뮬레이션(예: 가상 NIC로 데이터 전송)을 수행한 뒤, 다시 `KVM_RUN`을 호출하여 게스트 실행을 재개한다. 이 `KVM_RUN` 루프가 KVM 가상화의 핵심적인 제어 흐름이다.


KVM의 I/O 가상화 전략은 "성능과 기능 사이의 점진적 최적화" 경로를 보여준다. 완전 에뮬레이션에서 시작하여 VirtIO를 통한 반가상화, 그리고 vhost-net을 통한 커널 가속화로 이어지는 발전 과정은, 시스템의 병목 지점을 정확히 파악하고 이를 해결하기 위해 데이터 플레인(data plane)을 사용자 공간에서 커널 공간으로 점진적으로 이동시킨 공학적 최적화의 결과물이다.


초기 가상화 환경에서는 호환성을 위해 실제 하드웨어(예: Intel e1000 네트워크 카드, IDE 디스크 컨트롤러)를 소프트웨어로 그대로 모방하는 완전 에뮬레이션(Full Emulation) 방식을 사용했다.32 이 방식은 게스트 OS에 별도의 드라이버 설치가 필요 없다는 장점이 있지만, 모든 I/O 작업이 하이퍼바이저의 개입(VM-exit)을 유발하여 심각한 성능 저하를 초래하는 근본적인 한계를 지녔다.32

이 문제를 해결하기 위해 등장한 것이 바로 VirtIO이다. VirtIO는 가상화 환경을 위해 특별히 설계된 I/O 반가상화(Paravirtualization) 표준이다.16 VirtIO의 핵심 철학은 게스트 OS가 자신이 가상 환경에서 실행되고 있음을 인지하고, 하이퍼바이저와 가장 효율적인 방식으로 협력하는 것이다.

VirtIO의 아키텍처는 다음과 같은 세 가지 핵심 요소로 구성된다.

- **프론트엔드 드라이버 (Frontend Driver):** 게스트 OS의 커널에 설치되는 가상 디바이스 드라이버다. 게스트 OS의 애플리케이션에게는 일반적인 네트워크 카드나 디스크 드라이버처럼 보이지만, 내부적으로는 백엔드 드라이버와 통신하도록 설계되었다.34
- **백엔드 드라이버 (Backend Driver):** 호스트의 VMM(QEMU)에 구현된다. 프론트엔드 드라이버로부터 요청을 받아 이를 호스트의 실제 물리 장치나 네트워크 스택으로 전달하는 역할을 한다.36
- **Virtqueue:** 프론트엔드와 백엔드 간의 통신 채널이다. 이는 양측이 공유하는 메모리 영역에 구현된 고성능 링 버퍼(ring buffer) 구조로, 데이터 전송을 위한 컨텍스트 스위칭과 데이터 복사를 최소화한다. 각 virtqueue는 게스트가 요청을 담아두는 '사용 가능 링(Available Ring)', 호스트가 처리 완료된 요청을 알려주는 '사용된 링(Used Ring)', 그리고 실제 데이터 버퍼의 위치와 크기를 가리키는 '디스크립터 테이블(Descriptor Table)'로 구성된다.34

이 구조를 통해 게스트는 I/O 요청을 virtqueue에 넣고 호스트에 간단한 알림(kick)만 보내면 되며, 호스트는 요청 처리가 끝난 후 인터럽트를 통해 게스트에게 완료를 알린다. 이 방식은 기존 에뮬레이션 방식에 비해 VM-exit 발생 횟수를 획기적으로 줄여 I/O 성능을 크게 향상시킨다. VirtIO는 네트워크(`virtio-net`), 블록 스토리지(`virtio-blk`, `virtio-scsi`), 메모리 벌루닝(`virtio-balloon`) 등 다양한 가상 장치에 대한 표준화된 인터페이스를 제공한다.16


표준 VirtIO 아키텍처는 성능을 크게 향상시켰지만, 여전히 백엔드 드라이버가 사용자 공간의 QEMU 프로세스 내에서 실행된다는 한계가 있었다. 이로 인해 네트워크 패킷이 게스트 애플리케이션에서 물리 NIC로 전달되기까지 `게스트 커널 -> (VM-exit) -> KVM -> QEMU(사용자 공간) -> 호스트 커널(네트워크 스택) -> 물리 NIC` 와 같은 복잡한 경로를 거쳐야 했다. 이 과정에서 발생하는 여러 번의 컨텍스트 스위칭과 사용자 공간-커널 공간 간의 데이터 복사는 여전히 상당한 오버헤드를 유발했다.38

vhost-net은 이러한 병목을 해결하기 위해 `virtio-net`의 백엔드 드라이버를 QEMU 사용자 공간에서 호스트의 커널 공간으로 직접 이동시킨 기술이다.26 이는 컨트롤 플레인(Control Plane)과 데이터 플레인(Data Plane)을 분리하는 현대적인 네트워킹 설계 사상을 반영한 것이다.

- **컨트롤 플레인:** VM 생성, virtqueue 설정, 기능 협상 등과 같은 초기 설정 및 관리 작업은 여전히 QEMU가 담당한다. QEMU는 vhost 프로토콜을 사용하여 `vhost-net` 커널 모듈에 virtqueue의 메모리 주소와 같은 필수 정보를 전달한다.39
- **데이터 플레인:** 일단 설정이 완료되면, 실제 네트워크 패킷의 송수신은 QEMU를 완전히 우회하여 게스트 커널의 `virtio-net` 프론트엔드와 호스트 커널의 `vhost-net` 백엔드 간에 직접 이루어진다.38 데이터는 공유 메모리를 통해 전달되며, 알림(notification)은 `eventfd`와 `irqfd`를 통해 커널 내부에서 효율적으로 처리된다.

이러한 구조는 불필요한 컨텍스트 스위칭과 데이터 복사를 제거하여 네트워크 지연 시간을 줄이고 처리량을 극대화하며, KVM 네트워킹을 네이티브 성능에 가깝게 만드는 핵심적인 역할을 한다.


KVM은 다양한 가상 디스크 이미지 포맷을 지원하지만, 가장 널리 사용되는 것은 `raw`와 `qcow2`이다. 두 포맷은 성능과 기능 면에서 명확한 트레이드오프 관계를 가진다.

- raw 포맷:

  어떠한 메타데이터나 추가적인 구조 없이 디스크의 원시 비트(raw bit)를 그대로 파일에 저장하는 가장 단순한 형식이다.41 I/O 요청이 가상 디스크의 특정 오프셋에 발생하면, 이는 파일 시스템을 통해 파일의 동일한 오프셋에 대한 I/O로 직접 변환된다. 이러한 단순함 덕분에 

  `raw` 포맷은 추가적인 처리 오버헤드가 없어 가장 높은 I/O 성능을 제공한다.42 그러나 이미지 생성 시 지정된 전체 공간을 미리 할당해야 하므로 스토리지 공간 효율성이 떨어지며, 포맷 자체적으로 스냅샷과 같은 고급 기능을 지원하지 않는다.41

- qcow2 (QEMU Copy-On-Write 2) 포맷:

  KVM의 기본 이미지 포맷으로, 다양한 고급 기능을 제공하도록 설계되었다.41 핵심은 Copy-on-Write(CoW) 메커니즘으로, 데이터 변경 시 원본 블록을 덮어쓰는 대신 새로운 위치에 변경된 데이터를 기록하고 메타데이터(매핑 테이블)를 업데이트한다.41 이 메커니즘을 기반으로 다음과 같은 강력한 기능들을 지원한다.

  - **씬 프로비저닝 (Thin Provisioning):** 실제 데이터가 기록된 만큼만 물리적 공간을 차지하여 스토리지 공간을 효율적으로 사용할 수 있다.42
  - **스냅샷 (Snapshots):** 특정 시점의 VM 상태를 이미지 파일 내에 저장할 수 있어 백업 및 복구가 용이하다.42
  - **백킹 파일 (Backing Files):** 읽기 전용의 기본 이미지(템플릿)를 두고, 변경 사항만 별도의 `qcow2` 파일에 저장하여 여러 VM을 빠르고 효율적으로 배포할 수 있다.44
  - **암호화 및 압축:** AES 암호화와 zlib 압축을 지원하여 보안성과 공간 효율성을 높일 수 있다.42 이러한 기능들은 메타데이터를 관리하기 위한 추가적인 I/O를 유발하므로 raw 포맷에 비해 약간의 성능 오버헤드가 발생할 수 있다.42

아래 표는 `raw`와 `qcow2` 포맷의 주요 특징을 비교한 것이다.

| 특성 (Feature)          | `raw`                                                        | `qcow2` (QEMU Copy-On-Write 2)                               |
| ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **I/O 성능**            | **최상 (Excellent).** 메타데이터나 추가 계층 없이 블록 디바이스에 직접 I/O를 수행하여 오버헤드가 거의 없음.42 | **우수 (Good).** 메타데이터 관리 및 CoW 계층으로 인해 미세한 성능 오버헤드 발생 가능. 그러나 최신 KVM/QEMU에서는 성능 격차가 많이 줄어듦.42 |
| **이미지 크기**         | **고정 크기 (Pre-allocated).** 이미지 생성 시 지정한 전체 크기를 차지함. 씬 프로비저닝 미지원 (파일 시스템이 hole punching을 지원하는 경우는 예외).41 | **가변 크기 (Thin-provisioned).** 실제 데이터가 기록된 만큼만 물리적 공간을 차지하여 스토리지 효율성이 높음.41 |
| **스냅샷 (Snapshots)**  | **자체 지원 안 함.** LVM 스냅샷 등 호스트의 외부 스토리지 기술을 이용해야 함.42 | **자체 지원.** 이미지 파일 내에 여러 시점의 스냅샷을 저장할 수 있는 강력한 기능 제공 (내부/외부 스냅샷).42 |
| **Copy-on-Write (CoW)** | **미지원.**                                                  | **핵심 기능.** 템플릿 기반 VM 배포(backing file) 및 스냅샷의 기반이 됨.41 |
| **기타 기능**           | 없음.                                                        | AES 암호화, zlib 압축, 라이브 스냅샷 등 다양한 고급 기능 지원.42 |
| **주요 사용 사례**      | - 절대적인 디스크 I/O 성능이 요구되는 프로덕션 데이터베이스 - 스냅샷 기능이 불필요한 단일 목적 VM | - 개발 및 테스트 환경 - 스냅샷 기반 백업 및 복구 전략이 중요한 환경 - 스토리지 공간 효율성이 중요한 클라우드 환경 |


KVM 게스트 VM에 네트워크 연결을 제공하는 방식은 크게 NAT와 브릿지로 나뉜다.

- NAT (Network Address Translation) 방식:

  KVM 설치 시 기본으로 생성되는 virbr0 인터페이스를 통해 제공되는 방식이다.46 게스트 VM들은 호스트 내부에 생성된 가상의 사설 네트워크(

  `192.168.122.0/24` 등)에 속하게 된다. 게스트가 외부로 통신할 때는 호스트가 NAT를 통해 자신의 IP 주소로 변환하여 요청을 전달한다.46

  - **장점:** 별도의 설정 없이 즉시 사용 가능하며, 호스트의 네트워크 환경(유선, 무선 등)에 구애받지 않고 외부 인터넷에 연결할 수 있어 편리하다.46
  - **단점:** 외부 네트워크에서 게스트 VM으로 직접 접근하는 것이 불가능하다. 모든 트래픽이 호스트의 NAT를 거치므로 브릿지 방식에 비해 성능 오버헤드가 발생한다.46
  - **주요 사용 사례:** 개발자의 로컬 머신에서 인터넷 접근이 필요한 VM을 실행하거나, 외부 접근이 필요 없는 간단한 테스트 환경에 적합하다.

- 브릿지 (Bridge) 방식:

  호스트의 물리적 네트워크 인터페이스(예: eth0)를 Linux 브릿지(가상 스위치)에 연결하고, 게스트 VM들의 가상 NIC 역시 이 브릿지에 연결하는 방식이다.47 이 구성을 통해 게스트 VM은 마치 호스트와 동일한 물리적 네트워크에 직접 연결된 것처럼 동작한다.

  - **장점:** 게스트 VM이 물리 네트워크의 다른 서버들과 동등한 IP 주소를 할당받아 양방향 통신이 가능하다. NAT 계층이 없으므로 성능이 우수하다.48
  - **단점:** 호스트의 물리 네트워크 환경에 대한 이해와 추가적인 설정(브릿지 생성 및 인터페이스 연결)이 필요하다.48
  - **주요 사용 사례:** 외부에서 직접 접속해야 하는 웹 서버나 데이터베이스 서버를 VM에서 운영하거나, VM이 물리 네트워크의 다른 장치들과 자유롭게 통신해야 하는 프로덕션 환경에 필수적이다.


KVM은 단순한 서버 통합 도구를 넘어, 유연하고 탄력적인 클라우드 인프라를 구축하기 위한 핵심 구성 요소임을 증명하는 고급 기능들을 제공한다. 라이브 마이그레이션과 중첩 가상화는 KVM 아키텍처의 유연성과 하드웨어 가상화 기능의 깊은 활용을 보여주는 대표적인 예이다.


라이브 마이그레이션(Live Migration)은 실행 중인 가상 머신을 최소한의 서비스 중단 시간(downtime)으로 한 물리적 호스트에서 다른 물리적 호스트로 이전하는 강력한 기능이다.1 이 기능은 하드웨어 유지보수, 부하 분산, 장애 복구 등 현대 데이터센터 운영에 필수적이다.

성공적인 라이브 마이그레이션을 위한 핵심 전제 조건은 소스 호스트와 대상 호스트가 VM의 디스크 이미지가 저장된 공유 스토리지(예: NFS, iSCSI, GlusterFS)에 동일한 경로로 접근할 수 있어야 한다는 것이다.49

KVM의 라이브 마이그레이션은 주로 **Pre-copy** 방식으로 동작하며, 그 과정은 다음과 같다.49

1. **초기화 단계 (Setup):** 먼저 대상 호스트에 마이그레이션될 VM과 동일한 구성의 프로세스를 생성하고 소스 호스트와 통신 채널을 설정한다. 이후 소스 호스트는 VM 메모리의 변경 사항을 추적하기 위해 더티 페이지 로깅(dirty page logging)을 활성화한다.
2. **반복적 메모리 복사 (Iterative Memory Copy):** 이 단계가 라이브 마이그레이션의 핵심이다. 소스 VM이 계속 서비스를 제공하는 상태에서, VM의 전체 메모리 내용을 네트워크를 통해 대상 호스트로 복사한다. 이 복사가 진행되는 동안 소스 VM의 애플리케이션이 메모리 페이지를 수정하면, 해당 페이지는 '더티(dirty)'로 표시된다. KVM은 이 더티 페이지 목록을 추적하여, 1차 전체 복사가 끝난 후 변경된 더티 페이지만을 다시 대상 호스트로 전송한다. 이 과정은 더 이상 전송할 더티 페이지가 거의 남지 않을 때까지 여러 차례 반복된다.
3. **중단 및 최종 동기화 (Stop-and-Copy):** 반복적 복사를 통해 전송해야 할 더티 페이지의 양이 매우 적어지면(일반적으로 수십 밀리초 내에 전송 가능한 수준), KVM은 소스 VM의 실행을 잠시 중단(pause)시킨다. 이 극히 짧은 중단 시간 동안, 마지막으로 변경된 더티 페이지와 CPU 레지스터 상태, 가상 장치의 내부 상태 등 VM의 최종 상태 정보를 모두 대상 호스트로 전송한다. 또한 공유 스토리지의 디스크 이미지 상태를 동기화한다.
4. **서비스 재개 (Resumption):** 모든 상태 정보가 전송되면, 대상 호스트에서 VM의 실행을 재개(resume)한다. 이와 동시에 대상 호스트는 Gratuitous ARP 패킷을 브로드캐스트하여 네트워크 스위치의 MAC 주소 테이블을 업데이트하고, VM의 네트워크 연결이 새로운 물리적 위치로 즉시 전환되도록 한다.

이러한 정교한 과정을 통해 사용자는 거의 인지할 수 없는 수준의 서비스 중단만으로 VM을 이전할 수 있다. 메모리 쓰기 작업이 매우 많은 워크로드의 경우, **Post-copy** 마이그레이션 방식도 대안으로 사용될 수 있다. 이 방식은 최소한의 메모리만 전송한 후 대상 호스트에서 VM을 즉시 활성화하고, 게스트가 아직 전송되지 않은 메모리 페이지에 접근할 때마다 네트워크를 통해 해당 페이지를 실시간으로 가져오는 방식이다.51


중첩 가상화(Nested Virtualization)는 KVM을 통해 생성된 가상 머신(L1 게스트) 내부에서 다시 KVM 하이퍼바이저를 활성화하여, 그 안에 또 다른 가상 머신(L2 게스트)을 실행할 수 있게 하는 기능이다.52 이는 하이퍼바이저 자체를 가상화하는 것으로, 클라우드 환경 테스트, 가상화 기술 교육, 보안 연구를 위한 다층 샌드박스 구축 등 다양한 고급 시나리오를 가능하게 한다.

중첩 가상화를 활성화하는 과정은 호스트(L0)와 게스트(L1) 양쪽에서의 설정이 필요하다.

1. **호스트(L0) 설정:** 물리적 서버의 KVM 커널 모듈에 중첩 기능을 활성화하는 파라미터를 전달해야 한다. 이는 일반적으로 `/etc/modprobe.d/` 디렉토리에 설정 파일을 생성하여 `options kvm-intel nested=1` (Intel CPU) 또는 `options kvm-amd nested=1` (AMD CPU)을 추가하는 방식으로 이루어진다. 설정 후 모듈을 다시 로드하면 `/sys/module/kvm_intel/parameters/nested` 파일의 값이 'Y' 또는 '1'로 변경되어 활성화 여부를 확인할 수 있다.53
2. **게스트(L1) 설정:** L1 VM이 호스트 CPU의 가상화 확장 기능(VMX 또는 SVM 플래그)을 인식하고 사용할 수 있도록 CPU 설정을 변경해야 한다. 이는 L1 VM의 XML 설정에서 `<cpu>` 모드를 `host-passthrough` 또는 `host-model`로 지정함으로써 가능하다.53
   - `host-passthrough`: 호스트 CPU의 모델과 모든 기능을 그대로 L1 게스트에 노출시킨다. 최고의 성능을 제공하지만, 다른 종류의 CPU를 가진 호스트로의 마이그레이션이 불가능해진다.
   - `host-model`: 호스트 CPU와 가장 유사한 기본 CPU 모델을 선택하고, 호스트가 지원하는 추가 기능들을 활성화하여 L1 게스트에 제공한다. 성능과 마이그레이션 호환성 사이의 균형을 제공한다.

이 설정이 완료되면 L1 게스트 VM 내에서 KVM 관련 패키지를 설치하고, 마치 물리적 머신에서처럼 L2 VM을 생성하고 실행할 수 있다. 중첩 가상화는 강력한 기능이지만, L0에서 L1을 거쳐 L2로 이어지는 다층의 가상화로 인해 일정 수준의 성능 저하가 발생할 수 있으며, 일부 KVM 기능(예: L1 VM의 라이브 마이그레이션)에 제약이 생길 수 있다.53


KVM 성능 튜닝의 핵심은 "가상화 오버헤드 최소화"와 "호스트 리소스의 효율적 활용"이라는 두 가지 원칙으로 요약된다. 이는 게스트가 물리적 하드웨어에 최대한 가깝게(near-native) 접근하도록 하고, Linux 커널의 고급 기능들을 가상 환경에 맞게 재구성하는 정교한 시스템 엔지니어링 과정이다. 최적의 성능은 각 워크로드의 특성에 맞는 맞춤형 튜닝을 통해서만 달성될 수 있다.


- CPU 모델 및 토폴로지:

  게스트 VM에 노출되는 CPU 모델은 성능에 직접적인 영향을 미친다. 최고의 성능을 위해서는 QEMU 실행 시 -cpu host 옵션을 사용하는 것이 권장된다. 이 옵션은 호스트 CPU의 모든 기능(예: AVX, TSX 등)을 게스트에 그대로 전달하여, 게스트 OS와 애플리케이션이 최신 명령어 세트를 활용할 수 있게 한다.54 그러나 이 방식은 다른 CPU 모델을 가진 호스트로의 라이브 마이그레이션 호환성을 저해할 수 있다. 마이그레이션 호환성이 필요하다면, 클러스터 내 모든 호스트가 공통으로 지원하는 가장 높은 수준의 명명된 CPU 모델(예: `Cascadelake-Server`)을 지정하는 것이 좋다.55

- vCPU 할당 및 고정 (Pinning):

  게스트에 과도한 수의 vCPU를 할당하는 것은 컨텍스트 스위칭 오버헤드를 증가시켜 오히려 성능을 저하시킬 수 있다. 워크로드의 실제 요구사항을 분석하여 필요한 만큼의 vCPU만 할당하는 것이 중요하다.56 NUMA(Non-Uniform Memory Access) 아키텍처를 사용하는 호스트에서는 vCPU 스레드를 특정 물리 CPU 코어에 고정(pinning)하고, 해당 vCPU가 사용할 메모리 또한 동일한 NUMA 노드 내에 할당하는 것이 필수적이다. 이는 CPU와 메모리 간의 접근 지연 시간을 최소화하고 캐시 효율성을 극대화하여 성능을 크게 향상시킨다.57

- Huge Pages:

  Linux의 표준 메모리 페이지 크기는 4KB이다. 메모리 집약적인 애플리케이션을 실행하는 VM은 수백만 개의 페이지 테이블 엔트리(PTE)를 필요로 하며, 이는 TLB(Translation Lookaside Buffer)의 부하를 가중시켜 TLB miss를 유발하고 성능 저하의 원인이 된다. Huge Pages(일반적으로 2MB 또는 1GB)를 사용하면 동일한 양의 메모리를 훨씬 적은 수의 PTE로 관리할 수 있다. KVM 게스트의 메모리를 호스트의 Huge Pages로 백업하도록 설정하면 TLB 효율성이 극적으로 향상되어 데이터베이스, 인메모리 캐시 등 대용량 메모리를 사용하는 워크로드의 성능을 크게 개선할 수 있다.56

- KSM (Kernel Same-page Merging):

  KSM은 여러 실행 중인 VM들이 동일한 내용의 메모리 페이지를 공유하도록 하여 전체 메모리 사용량을 줄이는 기능이다. 이는 VM 밀도를 높이는 데 유용하지만, 지속적으로 메모리 페이지를 스캔하고 비교하는 과정에서 CPU 오버헤드를 유발할 수 있다. 따라서 I/O 지연 시간에 민감하거나 CPU 성능이 중요한 워크로드에서는 KSM을 비활성화하는 것이 성능에 유리할 수 있다.57


- 네트워크 최적화:

  최상의 네트워크 성능을 위해서는 반드시 virtio-net 가상 NIC를 사용해야 한다. 여기에 더해, vhost-net 기능을 활성화하여 네트워크 데이터 플레인을 호스트 커널에서 직접 처리하도록 하는 것이 표준적인 최적화 방식이다.54 vCPU가 여러 개인 VM의 경우, `multi-queue virtio-net`을 활성화하면 여러 vCPU가 네트워크 패킷을 병렬로 처리할 수 있어 네트워크 처리량을 크게 높일 수 있다. 큐의 수는 일반적으로 vCPU의 수와 동일하게 설정하는 것이 권장된다.56

- 스토리지 최적화:

  가상 디스크 인터페이스로는 레거시 IDE 대신 virtio-blk 또는 virtio-scsi를 사용해야 한다. virtio-scsi는 더 많은 장치를 지원하고 고급 SCSI 명령을 처리할 수 있어 유연성이 더 높다.33 디스크 캐시 설정은 워크로드에 따라 신중하게 선택해야 한다. 일반적으로 `cache=none` 옵션은 게스트 OS가 캐싱을 직접 제어하도록 하고, 호스트에서의 이중 캐싱(double caching)을 방지하여 예측 가능하고 안정적인 성능을 제공한다.54 I/O 집약적인 워크로드의 경우, `iothread`를 구성하여 스토리지 I/O 작업을 vCPU 스레드가 아닌 별도의 전용 스레드에서 처리하도록 하면, vCPU가 애플리케이션 로직 처리에 집중할 수 있어 전체적인 응답성을 향상시킬 수 있다.33

- 성능 분석 도구:

  KVM 환경의 성능 병목을 진단하기 위해 perf 도구는 매우 강력한 기능을 제공한다. 특히 perf kvm 하위 명령은 VM-exit 이벤트, KVM 내부의 트레이스포인트 등을 분석하여 가상화 오버헤드가 발생하는 원인을 구체적으로 파악하는 데 도움을 준다. 이를 통해 호스트와 게스트 양쪽의 성능 데이터를 종합적으로 분석하여 최적화 지점을 찾아낼 수 있다.18


KVM은 단독으로 사용되기보다는 거대한 오픈 소스 생태계의 '엔진' 또는 '구성 요소'로서 기능할 때 진정한 가치를 발휘한다. KVM의 성공은 기술적 우수성뿐만 아니라, libvirt, OpenStack, Kubernetes 등 다른 핵심 프로젝트와 유기적으로 결합하여 강력한 시너지를 창출하는 개방형 아키텍처에 기인한다.


KVM 자체는 로우레벨의 가상화 기능을 제공하는 커널 모듈이므로, 사용자가 이를 편리하게 관리하고 자동화하기 위한 상위 계층의 도구와 플랫폼이 필수적이다.

- **libvirt:** KVM 관리 생태계의 가장 핵심적인 구성 요소로, 다양한 하이퍼바이저(KVM, Xen, LXC 등)를 위한 공통되고 안정적인 API를 제공하는 오픈 소스 라이브러리다.1 `libvirt`는 VM의 라이프사이클 관리, 스토리지 풀 및 네트워크 설정, 마이그레이션 등 거의 모든 관리 작업을 추상화된 API로 제공한다. 이를 기반으로 하는 `virsh`(명령줄 인터페이스)와 `virt-manager`(그래픽 사용자 인터페이스)는 단일 호스트 환경에서 KVM을 관리하는 사실상의 표준 도구로 사용된다.1 `libvirt`의 존재 덕분에 상위 관리 플랫폼들은 KVM의 내부 구현에 직접 의존하지 않고도 KVM을 일관된 방식으로 제어할 수 있다.
- **OpenStack:** 수천 개의 물리적 서버와 스토리지, 네트워크 자원을 통합하여 대규모 IaaS(Infrastructure as a Service) 클라우드를 구축하기 위한 가장 대표적인 오픈 소스 플랫폼이다.11 OpenStack의 핵심 컴퓨팅 서비스인 `Nova`는 `libvirt`를 통해 다양한 하이퍼바이저를 지원하며, 그 중 KVM이 가장 널리 사용되는 기본 하이퍼바이저다.11 OpenStack 환경에서 KVM은 개별 VM을 실행하는 엔진 역할을 하며, OpenStack은 그 위에 셀프서비스 프로비저닝, 멀티테넌시, 대규모 자동화와 같은 클라우드 서비스를 구현한다.
- **Proxmox VE (Virtual Environment):** Debian Linux를 기반으로 KVM(가상 머신용)과 LXC(컨테이너용)를 긴밀하게 통합한 올인원(all-in-one) 가상화 관리 플랫폼이다.10 Proxmox VE는 사용하기 쉬운 웹 기반 관리 인터페이스, 클러스터링, 고가용성(HA), 스토리지 통합(Ceph, ZFS), 백업/복구 기능 등을 기본적으로 내장하고 있어, 별도의 복잡한 설정 없이도 엔터프라이즈급 가상화 환경을 빠르게 구축할 수 있다.20 중소규모 기업(SMB)이나 연구소, 홈랩 등에서 특히 높은 인기를 얻고 있다.
- **oVirt:** Red Hat Enterprise Virtualization(RHV)의 업스트림 커뮤니티 프로젝트로, 대규모 엔터프라이즈 데이터센터 환경을 위한 중앙 집중식 KVM 관리 플랫폼이다.19 oVirt는 고급 스토리지 및 네트워크 관리, 정책 기반의 고가용성 및 부하 분산, 포괄적인 모니터링 및 리포팅 기능을 제공하며, VMware vCenter와 유사한 역할을 목표로 한다.64


KVM의 기술적, 전략적 위치를 명확히 이해하기 위해서는 다른 주요 가상화 및 컨테이너 기술과의 비교가 필수적이다. 아래 표는 KVM을 Xen, VMware ESXi, 그리고 Docker와 핵심적인 측면에서 비교 분석한 것이다.

| 구분               | **KVM**                                                      | **Xen**                                                      | **VMware ESXi**                                              | **Docker (컨테이너)**                                        |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **기술 유형**      | Type-1 (Bare-metal) 하이퍼바이저                             | Type-1 (Bare-metal) 하이퍼바이저                             | Type-1 (Bare-metal) 하이퍼바이저                             | OS 수준 가상화 (컨테이너)                                    |
| **아키텍처**       | **Linux 커널에 통합**. 각 VM은 호스트에서 일반 프로세스로 실행.1 | **마이크로커널 아키텍처**. Dom0(제어 VM)이 하드웨어 접근 및 다른 VM(DomU)을 관리.13 | **독자적인 마이크로커널** 기반의 하이퍼바이저. vCenter를 통한 중앙 관리.9 | **호스트 OS 커널 공유**. 애플리케이션과 종속성을 격리된 사용자 공간(컨테이너)에 패키징.66 |
| **격리 수준**      | **강력함 (Strong).** 하드웨어 수준에서 VM 간 완전한 격리. 각 VM은 독립적인 커널을 가짐.68 | **강력함 (Strong).** VM 간 하드웨어 수준 격리. 독립적인 커널. | **강력함 (Strong).** VM 간 하드웨어 수준 격리. 독립적인 커널. | **프로세스 수준 (Process-level).** 호스트 커널을 공유하므로 VM보다 격리 수준이 낮음.66 |
| **성능/오버헤드**  | **낮음 (Low).** 하드웨어 가상화 확장 기능과 커널 통합으로 거의 네이티브에 가까운 성능.2 | **낮음 (Low).** 초기에는 반가상화(PV)에서 강점을 보였으나, 현재는 HVM에서도 KVM과 유사한 성능.14 | **낮음 (Low).** 고도로 최적화된 독점 하이퍼바이저로 우수한 성능 제공.9 | **매우 낮음 (Very Low).** OS 부팅 과정이 없어 시작이 매우 빠르고 리소스 사용량이 적음.66 |
| **게스트 OS**      | Linux, Windows, BSD 등 수정되지 않은 다양한 OS 지원.16       | 초기에는 커널 수정이 필요한 반가상화(PV) 위주였으나, 현재는 HVM을 통해 수정 없이도 대부분 OS 지원.14 | 광범위한 게스트 OS 지원.                                     | **호스트와 동일한 커널**을 사용하는 OS만 가능 (주로 Linux 기반).67 |
| **비용/라이선스**  | **무료 (오픈 소스).** 엔터프라이즈 지원은 Red Hat 등 벤더를 통해 유료 구매 가능.2 | **무료 (오픈 소스).** Citrix Hypervisor 등 상용 버전 존재.   | **상용 (Proprietary).** 무료 버전도 있으나 기능이 제한적. 엔터프라이즈 기능은 라이선스 구매 필요.7 | **오픈 소스 (Docker CE).** Docker EE(Enterprise Edition)는 유료 지원 포함. |
| **주요 사용 사례** | - 오픈 소스 기반 클라우드(OpenStack) - 엔터프라이즈 데이터센터 - 개발/테스트 환경 | - 대규모 퍼블릭 클라우드(초기 AWS) - 보안 및 격리가 중요한 환경 | - 엔터프라이즈 프라이빗 클라우드 - VDI(가상 데스크톱 인프라) - 미션 크리티컬 애플리케이션 | - 마이크로서비스 아키텍처 - CI/CD 파이프라인 - 애플리케이션 배포 및 확장 |

이 비교를 통해 KVM의 핵심적인 정체성이 드러난다. KVM은 VMware처럼 모든 기능이 통합된 완제품(turnkey solution)이 아니다. 오히려 강력하고 표준화된 '엔진'에 가깝다. 이 엔진은 `libvirt`라는 표준 변속기를 통해 다양한 차체(관리 플랫폼)와 결합하여, 단일 서버 가상화부터 대규모 클라우드, 그리고 컨테이너-VM 융합 플랫폼에 이르기까지 무한한 확장성을 보여준다. 이것이 바로 KVM 생태계의 가장 큰 강점이자 다른 독점 기술과의 근본적인 차별점이다.



본 고찰을 통해 분석한 KVM의 핵심 강점과 기술적 함의는 다음과 같이 요약할 수 있다.

1. **Linux 커널과의 완벽한 통합:** KVM은 별도의 하이퍼바이저가 아닌 Linux 커널의 내재된 기능으로, 수십 년간 축적된 커널의 안정성, 성능, 보안 기능을 그대로 활용한다. 각 VM이 일반 Linux 프로세스로 관리되는 아키텍처는 시스템 관리의 일관성과 효율성을 극대화한다.
2. **하드웨어 가상화의 적극적 활용:** KVM은 Intel VT-x, AMD-V와 같은 하드웨어 가상화 확장 기능에 전적으로 의존하여, CPU와 메모리 가상화 오버헤드를 최소화하고 거의 네이티브에 가까운 성능을 달성한다.
3. **역할 분담을 통한 유연성:** 성능이 중요한 CPU/메모리 가상화는 커널(KVM)이, 호환성과 기능 다양성이 중요한 I/O 에뮬레이션은 사용자 공간(QEMU)이 담당하는 명확한 역할 분담은 KVM 아키텍처의 핵심이다. 이는 안정성과 확장성이라는 두 마리 토끼를 모두 잡는 실용적인 설계이다.
4. **고성능 I/O 표준, VirtIO:** 반가상화 I/O 표준인 VirtIO와 이를 커널 수준에서 가속하는 vhost-net을 통해 가상화의 고질적인 병목이었던 I/O 성능 문제를 효과적으로 해결하였다.
5. **개방성과 생태계:** KVM은 무료 오픈 소스 소프트웨어로서 초기 도입 비용이 없으며, `libvirt`라는 강력한 추상화 계층을 통해 OpenStack, Proxmox, oVirt 등 거대한 오픈 소스 생태계의 핵심 엔진으로 기능한다. 이러한 개방성은 벤더 종속성을 피하고, 특정 요구사항에 맞는 맞춤형 인프라 구축을 가능하게 한다.


컨테이너 기술이 클라우드 네이티브 애플리케이션의 표준으로 자리 잡았음에도 불구하고, KVM 기반 가상 머신의 역할은 축소되기는커녕 오히려 더욱 중요해지고 있다. 여러 기술이 공존하며 상호 보완하는 하이브리드 클라우드 환경에서 KVM은 새로운 역할과 가능성을 보여주고 있다.

- **컨테이너와의 공존 및 통합:** 강력한 보안 격리가 요구되는 멀티테넌트 환경, 레거시 애플리케이션의 컨테이너화 이전 단계, 또는 Windows와 같은 비-Linux 워크로드를 컨테이너와 함께 운영해야 할 필요성으로 인해 VM의 수요는 여전히 견고하다. 특히 **KubeVirt**와 같은 프로젝트는 Kubernetes가 컨테이너뿐만 아니라 KVM 가상 머신까지 동일한 방식으로 오케스트레이션할 수 있는 길을 열었다. 이는 VM과 컨테이너의 경계를 허물고, 단일 플랫폼에서 두 워크로드를 통합 관리하는 미래 인프라의 핵심 기술로 KVM이 자리매김할 것임을 시사한다.
- **경량 가상화와 새로운 워크로드:** 서버 전체를 가상화하는 전통적인 VM을 넘어, 단일 애플리케이션을 위한 초경량 VM 기술이 부상하고 있다. Amazon의 **Firecracker**나 Google의 **crosvm**과 같은 경량 VMM(Virtual Machine Monitor)은 KVM을 기반으로 수백 밀리초 내에 부팅되는 마이크로VM(microVM)을 구현한다.16 이는 컨테이너의 빠른 시작 속도와 VM의 강력한 보안 격리라는 장점을 결합한 것으로, 서버리스 컴퓨팅, 엣지 컴퓨팅, CI/CD 파이프라인의 보안 강화 등 새로운 영역에서 KVM의 활용 범위를 폭발적으로 확장시키고 있다.
- **보안의 진화, 기밀 컴퓨팅(Confidential Computing):** 데이터가 사용 중(in-use)일 때에도 암호화를 통해 보호하는 기밀 컴퓨팅 패러다임에서 KVM은 핵심적인 역할을 수행할 것이다. Intel TDX(Trust Domain Extensions)나 AMD SEV(Secure Encrypted Virtualization)와 같은 차세대 하드웨어 보안 기술은 KVM과 긴밀하게 통합되어, 하이퍼바이저조차 게스트 VM의 메모리 내용을 들여다볼 수 없는 매우 높은 수준의 보안 환경을 제공한다.71 이는 금융, 의료, 공공 등 민감 데이터를 다루는 분야에서 KVM의 전략적 가치를 더욱 높일 것이다.

결론적으로, KVM은 지난 10여 년간 오픈 소스 가상화 기술의 표준을 정립해왔으며, 이제는 클라우드 네이티브라는 새로운 패러다임 속에서 컨테이너 기술과 융합하고, 서버리스와 엣지 컴퓨팅으로 영역을 확장하며, 기밀 컴퓨팅의 기반을 제공하는 등 끊임없이 진화하고 있다. Linux 커널과 함께 발전하는 KVM의 여정은 앞으로도 현대 IT 인프라의 근간을 이루는 핵심 동력으로 계속될 것이다.


1. What is KVM? - Red Hat, 8월 17, 2025에 액세스, https://www.redhat.com/en/topics/virtualization/what-is-KVM
2. What is KVM (Kernel-Based Virtual Machine)? - AWS, 8월 17, 2025에 액세스, https://aws.amazon.com/what-is/kvm/
3. What is KVM? - OVHcloud, 8월 17, 2025에 액세스, https://us.ovhcloud.com/learn/what-is-kvm/
4. Linux KVM, 8월 17, 2025에 액세스, https://linux-kvm.org/page/Main_Page
5. What is KVM (Kernel-based Virtual Machine) and How Does It Work? | FibaCloud.com Blog, 8월 17, 2025에 액세스, https://www.fibacloud.com/content/compute/what-is-kvm-kernel-based-virtual-machine-and-how-does-it-work/
6. KVM hypervisor: a beginners' guide - Ubuntu, 8월 17, 2025에 액세스, https://ubuntu.com/blog/kvm-hyphervisor
7. KVM vs. VMware: Choosing the Right Hypervisor for Your Needs - Lightbits Labs, 8월 17, 2025에 액세스, https://www.lightbitslabs.com/blog/kvmvsvmware/
8. Kernel-based Virtual Machine - IBM, 8월 17, 2025에 액세스, https://www.ibm.com/docs/en/license-metric-tool/9.2.0?topic=connections-kernel-based-virtual-machine
9. KVM vs. VMware - Red Hat, 8월 17, 2025에 액세스, https://www.redhat.com/en/topics/virtualization/kvm-vs-vmware-comparison
10. Proxmox Virtual Environment - Open-Source Server Virtualization Platform, 8월 17, 2025에 액세스, https://www.proxmox.com/en/products/proxmox-virtual-environment/overview
11. OpenStack vs. KVM - Pure Storage Blog, 8월 17, 2025에 액세스, https://blog.purestorage.com/purely-educational/openstack-vs-kvm/
12. Timeline of virtualization technologies - Wikipedia, 8월 17, 2025에 액세스, https://en.wikipedia.org/wiki/Timeline_of_virtualization_technologies
13. Ten years of KVM - LWN.net, 8월 17, 2025에 액세스, https://lwn.net/Articles/705160/
14. KVM Vs Xen Performance | Comparison, Pros and Cons - Hostsailor, 8월 17, 2025에 액세스, https://web.hostsailor.com/blog/kvm_vs_xen_performance/
15. x86 virtualization - Wikipedia, 8월 17, 2025에 액세스, https://en.wikipedia.org/wiki/X86_virtualization
16. Kernel-based Virtual Machine - Wikipedia, 8월 17, 2025에 액세스, https://en.wikipedia.org/wiki/Kernel-based_Virtual_Machine
17. SLES 12 SP5 | Virtualization Guide | Introduction to KVM Virtualization - SUSE Documentation, 8월 17, 2025에 액세스, https://documentation.suse.com/en-us/sles/12-SP5/html/SLES-all/cha-kvm-intro.html
18. Virtualization Tuning and Optimization Guide | Red Hat Enterprise Linux | 6, 8월 17, 2025에 액세스, https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/6/html-single/virtualization_tuning_and_optimization_guide/index
19. What Is KVM (Kernel-based Virtual Machine) - ChemiCloud, 8월 17, 2025에 액세스, https://chemicloud.com/glossary/term/kvm/
20. Features - Proxmox Virtual Environment, 8월 17, 2025에 액세스, https://www.proxmox.com/en/products/proxmox-virtual-environment/features
21. Understanding Hardware-Assisted Virtualization - ADMIN Magazine, 8월 17, 2025에 액세스, https://www.admin-magazine.com/Articles/Hardware-assisted-Virtualization
22. QEMU - Wikipedia, 8월 17, 2025에 액세스, https://en.wikipedia.org/wiki/QEMU
23. KVM Internal: How a VM is Created? · Better Tomorrow with ..., 8월 17, 2025에 액세스, https://insujang.github.io/2017-05-02/kvm-internal-how-a-vm-is-created/
24. Host Architecture - Virtualization - Oracle Help Center, 8월 17, 2025에 액세스, https://docs.oracle.com/en/virtualization/oracle-linux-virtualization-manager/arch/architecture-arch-host-arch.html
25. Introduction | QEMU internals - GitHub Pages, 8월 17, 2025에 액세스, https://airbus-seclab.github.io/qemu_blog/
26. KVM Architecture Overview - Stefan Hajnoczi, 8월 17, 2025에 액세스, https://vmsplice.net/~stefan/qemu-kvm-architecture-2015.pdf
27. Adaptive CPU Resource Allocation for Emulator in Kernel ... - arXiv, 8월 17, 2025에 액세스, https://arxiv.org/pdf/2310.14741
28. kvm: the Linux Virtual Machine Monitor, 8월 17, 2025에 액세스, https://www.kernel.org/doc/ols/2007/ols2007v1-pages-225-230.pdf
29. The Definitive KVM (Kernel-based Virtual Machine) API ..., 8월 17, 2025에 액세스, https://docs.kernel.org/virt/kvm/api.html
30. Linux-KVM (Kernel-based Virtual Machine) Explained | by TechExplorer - Medium, 8월 17, 2025에 액세스, https://techexplorer42.medium.com/going-through-linux-kvm-kernel-base-virtual-machine-d15b5dbf10f4
31. Documentation/virtual/kvm/api.txt - kernel/msm - Git at Google - Android GoogleSource, 8월 17, 2025에 액세스, https://android.googlesource.com/kernel/msm/+/android-6.0.1_r0.125/Documentation/virtual/kvm/api.txt
32. Virtio - libvirt Wiki, 8월 17, 2025에 액세스, https://wiki.libvirt.org/Virtio.html
33. QEMU/KVM Virtual Machines - Proxmox VE, 8월 17, 2025에 액세스, https://pve.proxmox.com/pve-docs/chapter-qm.html
34. Virtio: An I/O virtualization framework for Linux - IBM Developer, 8월 17, 2025에 액세스, https://developer.ibm.com/articles/l-virtio/
35. Virtio 1.2 is Coming! - Alibaba Cloud Community, 8월 17, 2025에 액세스, https://www.alibabacloud.com/blog/virtio-1-2-is-coming_599615
36. Introduction to VirtIO - Oracle Blogs, 8월 17, 2025에 액세스, https://blogs.oracle.com/linux/post/introduction-to-VirtIO
37. Virtio on Linux - The Linux Kernel documentation, 8월 17, 2025에 액세스, https://docs.kernel.org/driver-api/virtio/virtio.html
38. Virtio and Vhost Architecture - Part 2 · Better Tomorrow with Computer Science - Insu Jang, 8월 17, 2025에 액세스, https://insujang.github.io/2021-03-15/virtio-and-vhost-architecture-part-2/
39. Introduction to virtio-networking and vhost-net - Red Hat, 8월 17, 2025에 액세스, https://www.redhat.com/en/blog/introduction-virtio-networking-and-vhost-net
40. Deep dive into Virtio-networking and vhost-net - Red Hat, 8월 17, 2025에 액세스, https://www.redhat.com/en/blog/deep-dive-virtio-networking-and-vhost-net
41. 2.4. Storage Formats for Virtual Disks | Technical Reference - Red Hat Documentation, 8월 17, 2025에 액세스, https://docs.redhat.com/en/documentation/red_hat_virtualization/4.3/html/technical_reference/qcow2
42. Raw vs Qcow2 Image | Storware BLOG, 8월 17, 2025에 액세스, https://storware.eu/blog/raw-vs-qcow2-image/
43. Which is better image format, raw or qcow2, to use as a baseimage for other VMs?, 8월 17, 2025에 액세스, https://serverfault.com/questions/677639/which-is-better-image-format-raw-or-qcow2-to-use-as-a-baseimage-for-other-vms
44. QCOW2 backing files & overlays, 8월 17, 2025에 액세스, https://kashyapc.fedorapeople.org/virt/lc-2012/snapshots-handout.html
45. How to Create and Manage Internal Snapshots in KVM - SysGuides, 8월 17, 2025에 액세스, https://sysguides.com/create-and-manage-internal-snapshots-in-kvm
46. KVM default NAT-based networking - IBM, 8월 17, 2025에 액세스, https://www.ibm.com/docs/en/linux-on-systems?topic=choices-kvm-default-nat-based-networking
47. KVM Host Networking Configuration Choices - IBM, 8월 17, 2025에 액세스, https://www.ibm.com/docs/en/linux-on-systems?topic=recommendations-kvm-host-networking-configuration-choices
48. How to set up a network bridge for virtual machine communication - Red Hat, 8월 17, 2025에 액세스, https://www.redhat.com/en/blog/setup-network-bridge-VM
49. Migration - KVM, 8월 17, 2025에 액세스, https://linux-kvm.org/page/Migration
50. Kernel-based Virtual Machine KVM Architecture & Installation | Asher's Blogs, 8월 17, 2025에 액세스, https://asheritto.wordpress.com/devops/virtualization/kernel-based-virtual-machine-kvm-2/kernel-based-virtual-machine-kvm/
51. Configure live migrations - OpenStack Docs, 8월 17, 2025에 액세스, https://docs.openstack.org/nova/queens/admin/configuring-migrations.html
52. Create nested VMs | Compute Engine Documentation - Google Cloud, 8월 17, 2025에 액세스, https://cloud.google.com/compute/docs/instances/nested-virtualization/creating-nested-vms
53. How to enable nested virtualisation - Ubuntu Server documentation, 8월 17, 2025에 액세스, https://documentation.ubuntu.com/server/how-to/virtualisation/enable-nested-virtualisation/
54. Tuning KVM, 8월 17, 2025에 액세스, https://www.linux-kvm.org/page/Tuning_KVM
55. QEMU / KVM CPU model configuration, 8월 17, 2025에 액세스, https://qemu-project.gitlab.io/qemu/system/qemu-cpu-models.html
56. Performance hints and tips summary - IBM, 8월 17, 2025에 액세스, https://www.ibm.com/docs/en/linux-on-systems?topic=performance-summary
57. Virtualization Tuning and Optimization Guide | Red Hat Enterprise ..., 8월 17, 2025에 액세스, https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html-single/virtualization_tuning_and_optimization_guide/index
58. KVM/Qemu Virtualization Tuning Guide on 3rd Generation Intel® Xeon® Scalable Processors Based Platform, 8월 17, 2025에 액세스, https://cdrdv2-public.intel.com/686407/kvm-tuning-guide-icx.pdf
59. KVM/Qemu Virtualization Tuning Guide on Intel® Xeon® Based Systems, 8월 17, 2025에 액세스, https://www.intel.com/content/www/us/en/developer/articles/guide/kvm-tuning-guide-on-xeon-based-systems.html
60. System architecture - OpenStack Docs, 8월 17, 2025에 액세스, https://docs.openstack.org/nova/train/admin/arch.html
61. Nova System Architecture — nova 31.1.0.dev283 documentation, 8월 17, 2025에 액세스, https://docs.openstack.org/nova/latest/admin/architecture.html
62. Proxmox Virtual Environment - Wikipedia, 8월 17, 2025에 액세스, https://en.wikipedia.org/wiki/Proxmox_Virtual_Environment
63. oVirt vs KVM: Key Differences, Performance, and Which Virtualization to Choose, 8월 17, 2025에 액세스, https://www.diskinternals.com/vmfs-recovery/ovirt-vs-kvm/
64. oVirt | oVirt is a free open-source virtualization solution for your entire enterprise, 8월 17, 2025에 액세스, https://www.ovirt.org/
65. Architecture | oVirt, 8월 17, 2025에 액세스, https://www.ovirt.org/develop/architecture/architecture.html
66. Docker Containers vs. Virtual Machines: Which Should You Use? - HAKIA.com, 8월 17, 2025에 액세스, https://www.hakia.com/posts/docker-containers-vs-virtual-machines-which-should-you-use
67. ELI5: What is the difference between a container and a virtual machine? : r/compsci - Reddit, 8월 17, 2025에 액세스, https://www.reddit.com/r/compsci/comments/f2d3a6/eli5_what_is_the_difference_between_a_container/
68. Containers vs Virtual Machines | Atlassian, 8월 17, 2025에 액세스, https://www.atlassian.com/microservices/cloud-computing/containers-vs-vms
69. Containers vs. virtual machines (VMs) | Google Cloud, 8월 17, 2025에 액세스, https://cloud.google.com/discover/containers-vs-vms
70. Xen vs KVM: What Is The Difference? - ServerMania, 8월 17, 2025에 액세스, https://www.servermania.com/kb/articles/xen-vs-kvm
71. KVM - The Linux Kernel documentation, 8월 17, 2025에 액세스, https://docs.kernel.org/virt/kvm/index.html