---
layout: page
title: KVM 하이퍼바이저 기반 윈도우 10 가상 머신 구축 및 최적화
permalink: /systems/hyper-visor/KVM 하이퍼바이저 기반 윈도우 10 가상 머신 구축 및 최적화
---


리눅스 환경에서 가상화를 구현하는 다양한 기술 중, KVM(Kernel-based Virtual Machine)은 성능, 유연성, 그리고 성숙도 측면에서 독보적인 위치를 차지한다. 본 안내서는 KVM 하이퍼바이저를 기반으로 윈도우 10 게스트 운영체제를 설치하고, 최적의 성능을 발휘하도록 구성하며, 발생 가능한 문제들을 해결하는 전 과정을 체계적이고 심도 있게 다룬다. 단순한 명령어 나열을 넘어, 각 단계의 기술적 배경과 아키텍처적 의미를 설명함으로써 사용자가 KVM 환경을 깊이 이해하고 능동적으로 관리할 수 있도록 하는 것을 목표로 한다.


KVM은 단일 애플리케이션이 아닌, 여러 구성 요소가 유기적으로 결합된 계층적 아키텍처를 가진다. 이 구조를 이해하는 것은 설치 및 문제 해결의 핵심이다.

- **KVM (Kernel-based Virtual Machine):** KVM의 본질은 리눅스 커널 자체를 타입 1(네이티브) 하이퍼바이저로 변환하는 커널 모듈이다.1 이는 별도의 하이퍼바이저 운영체제 계층 없이, 호스트 시스템의 하드웨어와 직접 상호작용함을 의미한다. KVM은 CPU의 가상화 확장 기능(Intel VT-x 또는 AMD-V)을 활용하여 게스트의 명령어를 하드웨어에서 직접 실행하도록 중개하며, 이를 통해 거의 네이티브에 가까운 CPU 및 메모리 성능을 보장한다. `/dev/kvm`이라는 특수 장치 파일을 통해 사용자 공간 프로그램과 통신한다.
- **QEMU (Quick EMUlator):** QEMU는 본래 전체 시스템을 소프트웨어적으로 에뮬레이션하는 타입 2 하이퍼바이저이지만, KVM 환경에서는 그 역할이 변모한다. KVM과 함께 사용될 때 QEMU는 가상 머신 모니터(VMM)로서 사용자 공간에서 실행되며, 가상 디스크, 가상 네트워크 카드, USB 컨트롤러 등과 같은 하드웨어 장치 에뮬레이션을 담당한다.3 KVM이 CPU와 메모리 가상화를 가속하는 동안, QEMU는 나머지 I/O 장치들을 처리하는 것이다. 이 협력 관계 덕분에 KVM은 완전 가상화(Full Virtualization)를 고성능으로 구현할 수 있다.4
- **libvirt:** `libvirt`는 KVM/QEMU를 포함하여 Xen, LXC 등 다양한 가상화 기술을 일관된 방식으로 관리하기 위한 오픈소스 API, 데몬(`libvirtd`), 그리고 관리 도구의 집합체다.5 `libvirt`는 가상 머신의 생성, 시작, 중지, 마이그레이션 등 모든 생명주기를 관리하는 추상화 계층을 제공한다. XML 기반의 도메인 설정 파일을 사용하여 VM의 구성을 정의하며, `virsh`라는 강력한 명령줄 인터페이스(CLI)를 통해 세밀한 제어가 가능하다.
- **virt-manager & virt-install:** 이들은 `libvirt` API를 활용하는 상위 레벨 관리 도구다. `virt-manager`는 직관적인 그래픽 사용자 인터페이스(GUI)를 제공하여 VM의 생성부터 관리, 모니터링까지의 작업을 용이하게 한다.7 반면, `virt-install`은 명령줄에서 VM을 생성하고 프로비저닝하는 도구로, 스크립트를 통한 자동화에 매우 유용하다.8

이러한 계층적 구조는 문제 발생 시 체계적인 접근을 요구한다. 예를 들어, "VM이 시작되지 않는" 문제는 BIOS의 VT-x 비활성화(하드웨어 계층), `kvm_intel` 모듈 미로드(커널 계층), `/dev/kvm` 접근 권한 부재(권한/libvirt 계층), 또는 XML 설정 오류(관리 도구 계층) 등 다양한 원인에서 비롯될 수 있다. 따라서 성공적인 KVM 운영은 이 아키텍처에 대한 이해에서 출발한다.


KVM에서 윈도우 게스트의 성능을 논할 때, VirtIO(Virtual I/O)는 가장 핵심적인 개념이다. 완전 가상화 환경에서 QEMU는 실제 하드웨어(예: Intel e1000 네트워크 카드, IDE 컨트롤러)를 에뮬레이션한다. 이는 높은 호환성을 보장하지만, 에뮬레이션 과정에서 상당한 오버헤드가 발생하여 I/O 성능이 저하된다.

VirtIO는 이러한 비효율성을 극복하기 위한 반가상화(Paravirtualization) 인터페이스 표준이다.10 게스트 운영체제가 자신이 가상 환경에서 실행 중임을 인지하고, 표준화된 VirtIO 인터페이스를 통해 하이퍼바이저와 직접 통신하는 방식이다. 이는 복잡한 하드웨어 에뮬레이션 계층을 우회하여 I/O 처리량을 극대화하고 지연 시간을 최소화한다.

리눅스 게스트는 VirtIO 드라이버를 커널에 내장하고 있어 별도의 설정 없이도 최적의 성능을 발휘한다. 하지만 윈도우는 이 드라이버를 포함하고 있지 않다. 따라서 KVM 환경에서 윈도우를 설치하고 사용하는 과정에서 사용자가 직접 VirtIO 드라이버를 제공해야 한다. 이 과정은 선택이 아닌 필수이며, 특히 디스크와 네트워크 성능에 결정적인 영향을 미친다.12 본 안내서는 이 VirtIO 드라이버의 통합 과정을 설치 전, 설치 중, 설치 후 단계에 걸쳐 상세히 다룰 것이다.


본 안내서는 다음과 같은 6개의 논리적 단계에 따라 KVM 기반 윈도우 10 가상 머신 구축의 전 과정을 안내한다.

1. **호스트 시스템 준비:** 하드웨어 가상화 기능 확인부터 KVM 관련 패키지 설치, `libvirt` 데몬 설정 및 사용자 권한 부여까지 호스트 환경을 구축한다.
2. **필수 미디어 확보:** 윈도우 10 설치를 위한 공식 ISO 파일과, 성능 최적화를 위한 필수 VirtIO 드라이버 ISO 파일을 다운로드한다.
3. **가상 머신 생성 및 구성:** `virt-manager` 또는 `virt-install`을 사용하여 VM의 기본 틀을 만들고, 설치 전 성능 극대화를 위한 고급 하드웨어 설정을 적용한다.
4. **윈도우 10 설치:** VM을 부팅하고, 설치 과정 중 VirtIO 스토리지 드라이버를 로드하여 가상 디스크를 인식시킨 후, 표준 절차에 따라 윈도우를 설치한다.
5. **게스트 최적화:** 윈도우 설치 완료 후, 나머지 VirtIO 드라이버와 SPICE 게스트 도구를 설치하여 네트워크, 그래픽, 클립보드 공유 등의 기능을 활성화하고 최적화한다.
6. **고급 설정 및 문제 해결:** CPU 피닝, HugePages 등 고급 성능 튜닝 기법을 소개하고, 사용 중 발생할 수 있는 일반적인 문제들에 대한 체계적인 해결 방안을 제시한다.

이러한 단계별 접근을 통해 사용자는 안정적이고 성능이 뛰어난 윈도우 10 가상 환경을 성공적으로 구축할 수 있을 것이다.


KVM 가상 머신을 생성하기에 앞서, 호스트 리눅스 시스템이 가상화를 지원하고 관련 소프트웨어 스택이 올바르게 설치 및 구성되었는지 확인하는 것은 필수적인 첫 단계다. 이 과정은 하드웨어 수준의 검증부터 커널 모듈, 관리 데몬, 사용자 권한 설정에 이르기까지 체계적으로 진행되어야 한다.


KVM은 CPU에 내장된 하드웨어 가상화 확장 기능을 직접 활용하므로, 이 기능의 지원 여부와 활성화 상태를 확인하는 것이 가장 먼저 수행해야 할 작업이다.1 이 검증 과정은 여러 단계로 이루어지며, 각 단계의 실패는 서로 다른 근본 원인을 시사한다.


터미널에서 다음 명령어들을 순차적으로 실행하여 CPU의 가상화 지원 상태를 진단할 수 있다.

1. CPU 플래그 확인 (하드웨어 역량 검증):

   먼저 CPU 자체가 가상화 확장 기능을 물리적으로 지원하는지 확인한다.

   ```Bash
   egrep -c '(vmx|svm)' /proc/cpuinfo
   ```

   이 명령어는 `/proc/cpuinfo` 파일에서 가상화 관련 플래그를 검색하여 그 개수를 출력한다. Intel CPU의 경우 `vmx`(Virtual Machine eXtensions), AMD CPU의 경우 `svm`(Secure Virtual Machine) 플래그를 찾는다.16 만약 출력 결과가 `0`이라면, 해당 CPU는 하드웨어 가상화를 지원하지 않으므로 KVM을 가속 모드로 사용할 수 없다. 결과가 `1` 이상의 정수(일반적으로 CPU 코어 수)라면 CPU는 해당 기능을 지원한다.16

   또는 `lscpu` 명령어를 사용할 수도 있다.

   ```Bash
   lscpu | grep "Virtualization"
   ```

   이 명령어의 출력에 `Virtualization: VT-x` 또는 `Virtualization: AMD-V`가 포함되어 있다면 CPU가 가상화를 지원하는 것이다.2

2. kvm-ok 유틸리티 (BIOS 활성화 및 커널 준비 상태 검증):

   CPU가 가상화 기능을 지원하더라도 BIOS/UEFI 설정에서 비활성화되어 있을 수 있다.19

   `kvm-ok` 유틸리티는 이 기능이 실제로 활성화되어 커널이 사용할 수 있는 상태인지를 종합적으로 점검한다.

   ```Bash
   sudo apt install cpu-checker  # Ubuntu/Debian, 필요한 경우 설치
   kvm-ok
   ```

   이 명령어를 실행했을 때 다음과 같은 메시지가 출력되면 호스트 시스템은 KVM 가속을 사용할 준비가 된 것이다.20

   ```
   INFO: /dev/kvm exists
   KVM acceleration can be used
   ```

   만약 다음과 같은 메시지가 출력된다면, CPU는 기능을 지원하지만 BIOS/UEFI에서 비활성화된 상태이므로 다음 단계인 BIOS/UEFI 설정 변경이 필요하다.20

   ```
   INFO: KVM (vmx) is disabled by your BIOS
   HINT: Enter your BIOS setup and enable Virtualization Technology (VT)
   ```


`kvm-ok` 명령어 실행 결과 가상화 기능이 비활성화되어 있다는 메시지가 나타났다면, 시스템을 재부팅하여 BIOS/UEFI 설정 메뉴로 진입해야 한다.

1. **BIOS/UEFI 진입:** 시스템 부팅 초기 화면에서 `F2`, `F10`, `Delete`, `Esc` 등 제조사가 지정한 키를 눌러 설정 메뉴로 들어간다.16
2. **가상화 옵션 찾기:** 설정 메뉴는 제조사마다 구조가 다르지만, 일반적으로 'Advanced', 'CPU Configuration', 'Chipset', 'Northbridge' 또는 'Overclocking'과 같은 항목 아래에 위치한다.16
3. **옵션 활성화:** 해당 옵션은 다음과 같은 다양한 이름으로 표시될 수 있다. 이 옵션을 찾아 'Enabled' 또는 '활성화'로 변경한다.15
   - Intel Virtualization Technology (Intel VT-x)
   - Vanderpool Technology
   - Virtualization Extensions (VT-d)
   - AMD-V
   - Secure Virtual Machine (SVM) Mode
4. **저장 및 재부팅:** 변경 사항을 저장(일반적으로 `F10` 키)하고 시스템을 재부팅한다. 재부팅 후 다시 터미널에서 `kvm-ok` 명령어를 실행하여 KVM 가속이 사용 가능한지 최종 확인한다.


호스트 시스템의 하드웨어 및 BIOS 설정이 완료되었다면, 다음은 KVM을 구동하는 데 필요한 소프트웨어 패키지를 설치할 차례다. 설치 명령어는 사용하는 리눅스 배포판에 따라 다르다.


Ubuntu 및 Debian 계열 배포판에서는 `apt` 패키지 관리자를 사용한다.

1. **패키지 목록 갱신:**

   ```Bash
   sudo apt update
   ```

   최신 버전의 패키지를 설치하기 위해 로컬 패키지 인덱스를 먼저 갱신한다.18

2. **필수 패키지 설치:**

   ```Bash
   sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager -y
   ```

   이 명령어는 KVM 가상화 환경을 구축하고 관리하는 데 필요한 핵심 패키지들을 한 번에 설치한다.3

   - `qemu-kvm`: KVM 가속을 사용하는 QEMU 에뮬레이터 및 관련 도구.
   - `libvirt-daemon-system`: `libvirtd` 데몬을 시스템 서비스로 실행하기 위한 패키지.
   - `libvirt-clients`: `virsh`와 같은 클라이언트 측 관리 도구.
   - `bridge-utils`: 가상 머신을 위한 네트워크 브리지를 생성하고 관리하는 유틸리티.
   - `virt-manager`: 가상 머신 관리를 위한 GUI 애플리케이션.


CentOS, RHEL, Rocky Linux 등 Red Hat 계열 배포판에서는 `dnf` 또는 `yum` 패키지 관리자를 사용하며, 패키지 그룹을 통해 설치하는 것이 편리하다.

1. **시스템 업데이트:**

   ```Bash
   sudo dnf update -y
   ```

   커널 업데이트를 포함한 시스템 전체를 최신 상태로 유지한다.2

2. **가상화 그룹 패키지 설치:**

   ```Bash
   sudo dnf install @virt -y
   ```

   `@virt` 그룹은 KVM 하이퍼바이저와 `libvirtd` 데몬 등 가상화 호스트에 필요한 최소한의 패키지들을 포함한다.2

3. **관리 도구 및 GUI 설치:**

   ```Bash
   sudo dnf install virt-manager virt-install virt-viewer -y
   ```

   GUI 관리 도구(`virt-manager`), CLI 생성 도구(`virt-install`), 그리고 그래픽 콘솔 뷰어(`virt-viewer`)를 추가로 설치한다.2


Fedora는 최신 가상화 기술을 빠르게 도입하며, `dnf`를 통해 관련 패키지 그룹을 쉽게 설치할 수 있다.

1. **가상화 그룹 패키지 설치:**

   ```Bash
   sudo dnf install @virtualization -y
   ```

   `@virtualization` 그룹은 `qemu-kvm`, `libvirt`, `virt-manager` 등 필요한 모든 구성 요소를 포함하고 있어 이 명령어 하나로 대부분의 설치가 완료된다.25


패키지 설치 후에는 `libvirt` 데몬(`libvirtd`)이 시스템 부팅 시 자동으로 실행되도록 설정하고, 일반 사용자가 `sudo` 없이 가상 머신을 관리할 수 있도록 권한을 부여해야 한다.

1. **`libvirtd` 서비스 활성화 및 시작:**

   ```Bash
   sudo systemctl enable --now libvirtd
   ```

   `enable`은 부팅 시 서비스가 자동으로 시작되도록 설정하고, `--now` 플래그는 즉시 서비스를 시작한다.2

2. **서비스 상태 확인:**

   ```Bash
   sudo systemctl status libvirtd
   ```

   명령어 실행 결과 `Active: active (running)`이라는 녹색 문구가 표시되면 서비스가 정상적으로 동작하고 있는 것이다.2

3. 사용자 그룹에 추가:

   보안상의 이유로, libvirt 소켓에 접근하여 VM을 관리할 수 있는 권한은 기본적으로 root와 libvirt 그룹의 사용자에게만 허용된다. sudo를 매번 입력하는 번거로움을 피하고 virt-manager를 일반 사용자 권한으로 실행하려면, 현재 사용자를 libvirt와 kvm 그룹에 추가해야 한다.

   ```Bash
   sudo adduser $USER libvirt
   sudo adduser $USER kvm
   ```

   `$USER` 환경 변수는 현재 로그인된 사용자 이름을 자동으로 대체한다.3 일부 시스템에서는 

   `sudo usermod -aG libvirt,kvm $USER` 형태의 명령어를 사용할 수도 있다.3

4. 그룹 변경 사항 적용:

   새로운 그룹 멤버십은 다음 로그인 시에 적용된다. 변경 사항을 즉시 적용하려면 시스템에서 로그아웃한 후 다시 로그인하거나, 현재 셸에만 임시로 적용하기 위해 newgrp libvirt 명령어를 실행할 수 있다.5


모든 설정이 완료된 후, KVM 스택이 올바르게 작동하는지 최종적으로 검증한다.

1. **커널 모듈 확인:**

   ```Bash
   lsmod | grep kvm
   ```

   출력 결과에 `kvm_intel` (Intel CPU) 또는 `kvm_amd` (AMD CPU) 모듈이 로드되어 있는지 확인한다. 이 모듈들이 KVM의 핵심이다.2

2. **`libvirt` 데몬 통신 확인:**

   ```Bash
   sudo virsh list --all
   ```

   이 명령어는 현재 `libvirt` 데몬에 정의된 모든 가상 머신(실행 중 및 중지 상태 포함)의 목록을 요청한다. 설치 직후에는 아무런 VM도 없으므로 비어 있는 헤더만 출력되지만, 에러 메시지 없이 정상적으로 실행된다면 `virsh` 클라이언트가 `libvirtd` 데몬과 성공적으로 통신하고 있음을 의미한다.5

이 단계까지 모두 성공적으로 마쳤다면, 호스트 시스템은 윈도우 10 가상 머신을 생성하고 실행할 준비가 완료된 것이다.


성능이 뛰어난 KVM 기반 윈도우 10 가상 머신을 구축하기 위해서는 두 가지 핵심적인 ISO 이미지 파일이 필요하다. 첫째는 윈도우 10 운영체제 자체를 설치하기 위한 공식 설치 미디어이며, 둘째는 가상화 환경에서 최적의 I/O 성능을 내기 위한 VirtIO 드라이버 미디어다. 이 두 가지를 사전에 준비하는 것은 원활한 설치 과정의 필수 조건이다.


윈도우 10 설치 ISO 파일은 반드시 Microsoft의 공식 채널을 통해 확보해야 한다. 이는 시스템의 안정성과 보안을 보장하는 가장 기본적인 조치다.

- **공식 다운로드 페이지:** Microsoft의 공식 'Windows 10 다운로드' 페이지가 ISO 파일을 얻는 유일하고 안전한 출처다.28
- **다운로드 방법:**
  1. **Windows 환경:** 현재 사용 중인 운영체제가 윈도우인 경우, 해당 페이지에 접속하면 '지금 업데이트'와 '지금 도구 다운로드' 두 가지 옵션이 나타난다. '지금 도구 다운로드'를 선택하여 '미디어 생성 도구(Media Creation Tool)'를 다운로드한다.29 이 도구를 실행한 후, '다른 PC용 설치 미디어(USB 플래시 드라이브, DVD 또는 ISO 파일) 만들기' 옵션을 선택하고, 언어, 아키텍처(64비트 권장)를 지정한 뒤 최종적으로 'ISO 파일'을 선택하여 저장한다.28
  2. **Linux/macOS 환경:** 리눅스나 macOS와 같은 비-윈도우 운영체제에서 다운로드 페이지에 접속하면, 미디어 생성 도구 대신 ISO 파일을 직접 다운로드할 수 있는 옵션이 제공된다.28 만약 윈도우 환경으로 인식되어 도구 다운로드 페이지만 보인다면, 웹 브라우저의 개발자 도구(F12)를 열어 사용자 에이전트(User-Agent) 문자열을 리눅스나 모바일 기기(예: iPad)로 변경한 후 페이지를 새로고침하면 ISO 다운로드 선택 메뉴가 나타난다.29
- **에디션 선택:** 다운로드 과정에서 에디션을 선택하라는 메시지가 나타나면, 'Windows 10 (multi-edition ISO)'를 선택한다. 이 ISO 파일에는 Windows 10 Home과 Windows 10 Pro 버전이 모두 포함되어 있어 설치 과정에서 원하는 에디션을 선택할 수 있다.28


VirtIO 드라이버는 KVM 환경에서 윈도우 게스트가 네이티브에 가까운 디스크 및 네트워크 성능을 발휘하도록 만드는 핵심적인 반가상화 드라이버 모음이다.11 이는 VirtualBox의 '게스트 확장'이나 VMware의 'VMware Tools'와 유사한 역할을 하지만, KVM의 개방적인 생태계 특성상 별도로 다운로드하여 설치 과정에 통합해야 한다.

- **공식 다운로드 소스:** 안정적인 최신 VirtIO 드라이버는 Fedora 프로젝트에서 공식적으로 배포한다. 다음 링크에서 'stable' 버전의 ISO 파일을 다운로드하는 것이 권장된다.
  - **다운로드 링크:** `https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso` 12
- **ISO 파일의 내용물:** 이 `virtio-win.iso` 파일 안에는 다음과 같은 다양한 종류의 드라이버가 포함되어 있다. 각 드라이버는 특정 가상 하드웨어의 성능을 최적화하는 역할을 한다.11
  - **`viostor` (VirtIO Block Driver):** 가상 하드 디스크의 I/O 성능을 담당하는 가장 중요한 드라이버. 윈도우 설치 과정에서 이 드라이버를 로드해야 가상 디스크를 인식할 수 있다.
  - **`NetKVM` (VirtIO Network Driver):** 가상 네트워크 어댑터의 성능을 최적화한다.
  - **`Balloon` (VirtIO Memory Balloon Driver):** 동적 메모리 관리를 가능하게 하여 호스트의 메모리 자원을 효율적으로 사용할 수 있게 한다.
  - **`viorserial` (VirtIO Serial Driver):** 호스트와 게스트 간의 효율적인 통신 채널을 제공하며, 게스트 에이전트 등이 사용한다.
  - **QXL 그래픽 드라이버:** SPICE 프로토콜 사용 시 향상된 2D 그래픽 성능을 제공한다.
- **중요성 및 필요성:** 윈도우 설치 프로그램은 VirtIO 규격의 가상 디스크를 기본적으로 인식하지 못한다. 따라서 설치 과정 중 "드라이브를 찾을 수 없습니다"라는 메시지를 만나게 되는데, 이때 이 VirtIO 드라이버 ISO를 통해 `viostor` 드라이버를 수동으로 로드해주어야만 설치를 계속 진행할 수 있다. 이 단계는 KVM에 윈도우를 설치하는 과정에서 가장 핵심적인 부분 중 하나다.12 성능과 안정성을 위해 항상 최신 안정(stable) 버전의 드라이버를 사용하는 것이 좋다.11

이 두 가지 ISO 파일을 모두 준비했다면, 이제 가상 머신을 생성하고 윈도우를 설치할 준비가 완료된 것이다.


필수 설치 미디어가 준비되었다면, 이제 가상 머신의 '그릇'을 만들 차례다. 이 단계에서는 가상 머신의 하드웨어 사양을 정의하고, 특히 윈도우 10이 최적의 성능을 낼 수 있도록 고급 설정을 적용하는 것이 매우 중요하다. `virt-manager` (GUI)와 `virt-install` (CLI) 두 가지 방법을 모두 다룬다.


`virt-manager`는 직관적인 마법사 인터페이스를 제공하여 가상 머신 생성을 용이하게 한다. 하지만 기본 설정값은 레거시 호환성에 초점을 맞추고 있으므로, 성능을 위해서는 마지막 단계에서 반드시 구성을 사용자 정의해야 한다.

1. **마법사 시작:** 터미널에서 `virt-manager`를 실행하거나 애플리케이션 메뉴에서 '가상 머신 관리자'를 찾아 실행한다. 메인 창에서 컴퓨터 모니터 모양의 '새 가상 머신 만들기' 아이콘을 클릭하여 생성 마법사를 시작한다.18
2. **1단계 (설치 미디어 선택):** '로컬 설치 미디어(ISO 이미지 또는 CDROM)' 옵션을 선택하고 '앞으로'를 클릭한다. '찾아보기...' 버튼을 눌러 로컬 저장소에서 앞서 다운로드한 Windows 10 ISO 파일을 선택한다. `virt-manager`가 OS 종류를 자동으로 탐지하려 시도할 것이다. 만약 올바르게 탐지되지 않으면, 'OS 종류'와 '버전' 목록에서 'Windows'와 'Microsoft Windows 10'을 수동으로 선택한다.34
3. **2단계 (메모리 및 CPU 설정):**
   - **메모리(RAM):** Windows 10 게스트가 원활하게 동작하기 위해 최소 4096 MB (4 GB) 이상을 할당하는 것이 권장된다. 데스크톱 환경으로 사용하려면 8192 MB (8 GB) 이상을 할당하는 것이 좋다. 호스트 시스템의 전체 물리적 RAM 용량을 고려하여 적절한 값을 입력한다.36
   - **CPU:** 가상 CPU(vCPU) 개수를 할당한다. 일반적인 데스크톱 작업에는 2-4개의 vCPU가 적당하다. 할당하는 vCPU의 수는 호스트의 물리적 CPU 코어 수를 초과하지 않는 것이 일반적이다. 과도한 할당은 오히려 컨텍스트 스위칭 비용을 증가시켜 성능을 저하시킬 수 있다.37
4. **3단계 (스토리지 설정):**
   - '이 가상 머신에 대한 디스크 이미지 활성화'를 선택한다.
   - '가상 디스크 이미지 만들기' 옵션을 선택하고, Windows 운영체제, 애플리케이션, 사용자 데이터를 저장하기에 충분한 크기를 할당한다. 최소 60 GB 이상을 권장하며, 주 용도에 따라 80-120 GB 또는 그 이상을 할당할 수 있다.14
   - `qcow2` 포맷은 씬 프로비저닝(실제 사용량만큼만 디스크 공간 차지)과 스냅샷 기능을 지원하므로 특별한 이유가 없는 한 기본값으로 유지하는 것이 좋다.39
5. **4단계 (최종 설정 및 사용자 정의):**
   - 가상 머신의 이름을 지정한다(예: `win10-vm`).
   - **가장 중요한 단계:** 네트워크 선택 아래에 있는 **'설치 전에 구성 사용자 정의'** 체크박스를 **반드시 선택**한다. 이 옵션을 선택하지 않으면 성능 최적화 설정을 적용할 수 없으며, 레거시 호환 모드로 VM이 생성되어 심각한 성능 저하를 겪게 된다. '마침'을 클릭하면 VM 생성이 완료되고, 하드웨어 세부 설정 창이 나타난다.14


'설치 전에 구성 사용자 정의'를 선택하면 나타나는 하드웨어 상세 설정 화면에서 다음 항목들을 변경하여 VM의 성능을 극대화한다. 이 과정은 KVM에서 윈도우를 제대로 사용하기 위한 핵심적인 절차다.

- **개요 (Overview):**
  - **칩셋:** `i440FX`로 되어 있는 기본값을 `Q35`로 변경한다. Q35는 더 현대적인 PCIe 기반 아키텍처를 제공하여 성능과 기능 면에서 우수하다.14
  - **펌웨어:** `BIOS`로 되어 있는 기본값을 `UEFI x86_64: /usr/share/OVMF/OVMF_CODE.fd` 와 같은 UEFI 펌웨어로 변경한다. UEFI는 최신 운영체제에 더 적합하며, 더 빠른 부팅과 Secure Boot 같은 고급 기능을 지원한다.14
- **CPU:**
  - CPU 모델 옆의 '복사 호스트 CPU 구성' 버튼을 클릭하거나, 모델 목록에서 'host-passthrough'를 직접 선택한다. 이 설정은 QEMU의 CPU 에뮬레이션을 비활성화하고, 게스트가 호스트 CPU의 모든 기능과 명령어 셋을 직접 사용하게 하여 CPU 성능을 거의 네이티브 수준으로 끌어올린다.4
  - (선택 사항) '토폴로지'를 수동으로 설정하여 호스트의 물리적 CPU 구조(소켓 수, 코어 수, 스레드 수)를 모방하면, 게스트 OS의 스케줄러가 CPU 캐시를 더 효율적으로 활용하여 성능이 향상될 수 있다.37
- **SATA 디스크 1 (부팅 디스크):**
  - 왼쪽 목록에서 'SATA 디스크 1'을 선택한다.
  - '고급 옵션'을 펼치고 '디스크 버스'를 `SATA`에서 **`VirtIO`**로 변경한다. 이는 디스크 I/O 성능을 비약적으로 향상시키는 가장 중요한 설정이다.14
  - '성능 옵션'에서 '캐시 모드'를 `writeback`으로 설정하면 디스크 쓰기 성능이 향상된다. 다만, 호스트 시스템이 갑자기 다운될 경우 캐시에 있던 데이터가 유실될 위험이 있으므로, 데이터 무결성이 매우 중요한 환경에서는 `none`으로 설정하는 것을 고려해야 한다.14
- **NIC (네트워크 인터페이스):**
  - 왼쪽 목록에서 NIC 장치를 선택한다.
  - '장치 모델'을 기본값(예: `e1000e`)에서 **`virtio`**로 변경한다. 이 또한 네트워크 처리량과 지연 시간을 개선하는 핵심 설정이다.14
- **새 하드웨어 추가 (Add Hardware):**
  - 창 왼쪽 하단의 '하드웨어 추가' 버튼을 클릭한다.
  - '스토리지'를 선택하고, '장치 유형'을 'CDROM 장치'로 설정한다.
  - '관리...' 버튼을 클릭한 뒤, '로컬 찾아보기'를 통해 앞서 다운로드한 **`virtio-win.iso`** 파일을 선택하고 '마침'을 누른다. 이로써 윈도우 설치 ISO와 VirtIO 드라이버 ISO가 각각 별도의 가상 CDROM 드라이브에 마운트된다. 이는 윈도우 설치 중 스토리지 드라이버를 로드하기 위해 반드시 필요하다.12
- **부팅 옵션 (Boot Options):**
  - 부팅 장치 목록에서 Windows 10 설치 ISO가 마운트된 'SATA CDROM 1'이 'VirtIO Disk 1'보다 위에 있도록 순서를 조정한다. 이를 통해 VM이 하드 디스크가 아닌 설치 미디어로 먼저 부팅되도록 한다.12

모든 설정이 완료되었으면, 창 상단의 '설치 시작' 버튼을 클릭하여 가상 머신을 부팅하고 윈도우 설치를 시작한다.


스크립트를 통한 자동화나 GUI 환경이 없는 서버에서 VM을 생성할 때는 `virt-install` 명령어가 매우 유용하다.9 위에서 `virt-manager`로 설정한 최적화 항목들을 모두 포함하는 명령어 예시는 다음과 같다.

```Bash
virt-install \
--name win10-pro-vm \
--description "Windows 10 Pro KVM Guest" \
--ram 8192 \
--vcpus 4,sockets=1,cores=2,threads=2 \
--cpu host-passthrough \
--disk path=/var/lib/libvirt/images/win10-pro.qcow2,size=80,bus=virtio,format=qcow2 \
--cdrom /home/user/iso/Win10_22H2_x64.iso \
--disk /home/user/iso/virtio-win.iso,device=cdrom \
--network network=default,model=virtio \
--os-variant win10 \
--graphics spice,listen=0.0.0.0 \
--video qxl \
--sound ich9 \
--boot uefi
```

**주요 옵션 해설:**

- `--name`, `--ram`, `--vcpus`: VM 이름, RAM(MB 단위), 가상 CPU 설정.
- `--cpu host-passthrough`: 호스트 CPU 구성을 그대로 사용.
- `--disk... bus=virtio`: 첫 번째 `--disk` 옵션으로 VirtIO 버스를 사용하는 80GB qcow2 이미지를 주 디스크로 지정.
- `--cdrom...`: Windows 10 설치 ISO 경로 지정.
- `--disk...,device=cdrom`: 두 번째 `--disk` 옵션으로 VirtIO 드라이버 ISO를 추가 CDROM 장치로 지정.
- `--network...,model=virtio`: 기본(`default`) NAT 네트워크에 VirtIO 모델로 연결.
- `--os-variant win10`: `libvirt`가 Windows 10에 최적화된 기본 설정을 적용하도록 힌트를 준다. 사용 가능한 목록은 `osinfo-query os` 명령으로 확인할 수 있다.2
- `--graphics spice`: SPICE 프로토콜을 사용하여 그래픽 콘솔에 연결.
- `--video qxl`: SPICE에 최적화된 QXL 가상 비디오 어댑터 사용.
- `--boot uefi`: 레거시 BIOS 대신 UEFI 펌웨어로 부팅.

이 명령어를 실행하면 VM이 생성 및 시작되고, `virt-viewer`와 같은 클라이언트를 통해 그래픽 콘솔에 자동으로 연결되어 윈도우 설치를 진행할 수 있다.


가상 머신의 하드웨어 구성이 완료되고 부팅이 시작되면, 이제 윈도우 10 운영체제를 설치할 차례다. 이 과정은 표준적인 윈도우 설치와 대부분 유사하지만, KVM 환경의 핵심인 VirtIO 스토리지 드라이버를 수동으로 로드하는 결정적인 단계가 포함된다. 이 단계는 실패가 아니라, 고성능 반가상화 장치를 게스트 OS에 인식시키는 정상적이고 필수적인 절차다.


1. `virt-manager`에서 '설치 시작' 버튼을 클릭하거나 `virt-install` 명령어를 실행하면, 가상 머신 콘솔 창(`virt-viewer`)이 나타나며 VM이 부팅된다.
2. 화면에 "Press any key to boot from CD or DVD..."라는 메시지가 나타나면, 즉시 아무 키나 눌러 Windows 설치 미디어로 부팅을 시작한다.12 이 메시지를 놓치면 VM이 부팅 가능한 장치를 찾지 못하고 멈출 수 있다. 이 경우 VM을 강제 종료하고 다시 시작해야 한다.
3. Windows 설치 프로그램이 로드되면, 언어, 시간 및 통화 형식, 키보드 또는 입력 방법을 설정하고 '다음'을 클릭한다.
4. '지금 설치' 버튼을 클릭하여 설치를 시작한다. 제품 키를 입력하라는 메시지가 나타나면 보유한 키를 입력하거나 '제품 키가 없음'을 선택하여 나중에 인증할 수 있다. 이후 설치할 Windows 10 에디션(예: Pro)을 선택한다.


이 단계는 KVM에서 윈도우를 설치할 때 가장 중요한 부분이다. 앞서 VM의 주 디스크를 성능이 우수한 `VirtIO` 버스로 설정했기 때문에, 표준 윈도우 설치 프로그램은 이 가상 디스크를 인식할 수 없다.

1. **설치 유형 선택:** '어떤 종류의 설치를 원하시나요?' 화면에서 '사용자 지정: Windows만 설치(고급)'을 선택한다.24

2. **드라이버 부재 확인:** 'Windows를 설치할 위치를 지정하세요.' 화면에 아무런 디스크도 표시되지 않고, "드라이브를 찾을 수 없습니다. 저장소 드라이버를 로드하려면 드라이버 로드를 클릭하세요."라는 메시지가 나타난다.12 이것은 오류가 아니라 예상된 상황이다.

3. **드라이버 로드 시작:** 화면 하단의 '드라이버 로드' 링크를 클릭한다.12

4. 드라이버 위치 탐색: '드라이버 로드' 팝업 창에서 '찾아보기' 버튼을 클릭한다. 파일 탐색기 창이 나타나면, '컴퓨터'를 확장하여 두 개의 CD 드라이브를 확인한다. 하나는 윈도우 설치 미디어이고, 다른 하나는 우리가 추가한 virtio-win.iso 파일이다. 후자를 선택하고 다음 경로로 이동한다 (64비트 윈도우 기준):

   E:\viostor\w10\amd64

   (드라이브 문자는 시스템에 따라 다를 수 있다).12

5. **드라이버 선택 및 설치:** 해당 폴더를 선택하고 '확인'을 클릭하면, 설치할 수 있는 드라이버 목록에 'Red Hat VirtIO SCSI pass-through controller'가 나타난다. 이 드라이버를 선택하고 '다음'을 클릭한다.12 드라이버 파일이 시스템에 복사되고 나면, 이전 단계의 디스크 선택 화면으로 돌아간다.


VirtIO 스토리지 드라이버가 성공적으로 로드되면, 이전에 보이지 않던 가상 하드 디스크가 화면에 나타난다.

1. **파티션 생성:** '드라이브 0 할당되지 않은 공간'이라고 표시된 디스크를 선택한다.48 화면 하단의 '새로 만들기'를 클릭한다. 전체 크기를 그대로 사용하려면 '적용'을 클릭한다. 이때 "Windows에서 시스템 파일에 대해 추가 파티션을 만들 수 있습니다."라는 경고 메시지가 나타나는데, 이는 정상이다. '확인'을 클릭하면 EFI 시스템 파티션, MSR(Microsoft Reserved) 파티션 등 윈도우 부팅에 필요한 작은 파티션들이 자동으로 생성된다.49
2. **설치 파티션 선택:** 여러 파티션 중 가장 크기가 큰 '주' 파티션을 선택하고 '다음'을 클릭한다.
3. **Windows 설치 진행:** 이제 Windows 파일 복사, 기능 설치, 업데이트 설치 등의 과정이 자동으로 진행된다. 이 과정에서 가상 머신은 몇 차례 자동으로 재부팅된다.
4. **초기 설정(OOBE):** 설치가 완료되면 지역, 키보드 레이아웃, 네트워크 연결, Microsoft 계정 또는 로컬 계정 설정 등 OOBE(Out-of-Box Experience) 과정을 거쳐 Windows 설정을 마무리한다.

이 모든 과정이 끝나면, 기본적인 윈도우 10 설치가 완료된다. 하지만 아직 네트워크, 그래픽 등 다른 VirtIO 장치들의 드라이버가 설치되지 않은 상태이므로, 다음 단계인 설치 후 최적화 작업이 반드시 필요하다.


윈도우 10의 기본 설치가 완료되었지만, 현재 상태의 가상 머신은 최적의 성능과 사용 편의성을 갖추지 못했다. 네트워크가 연결되지 않았을 수 있으며, 그래픽 성능이 낮고, 호스트와의 상호작용(클립보드 공유, 화면 크기 자동 조절 등)이 불가능하다. 이 단계에서는 나머지 VirtIO 드라이버와 SPICE 게스트 도구를 설치하여 VM의 잠재력을 최대한 끌어내는 것을 목표로 한다.


윈도우 설치 과정에서는 부팅에 필수적인 스토리지 드라이버(`viostor`)만 로드했다. 이제 나머지 모든 반가상화 장치들의 드라이버를 설치해야 한다.

1. **장치 관리자 확인:** 윈도우에 로그인한 후, '시작' 버튼을 마우스 오른쪽 버튼으로 클릭하고 '장치 관리자'를 연다. '기타 장치' 항목 아래에 '이더넷 컨트롤러', 'PCI 장치', 'PCI 단순 통신 컨트롤러' 등 여러 장치에 노란색 경고 아이콘(느낌표)이 표시되어 있는 것을 확인할 수 있다. 이는 해당 장치들의 드라이버가 설치되지 않았음을 의미한다.11
2. **VirtIO 게스트 도구 설치 프로그램 실행:**
   - 파일 탐색기를 열고 `virtio-win.iso`가 마운트된 CD 드라이브로 이동한다.
   - 드라이브 내에서 `virtio-win-guest-tools.exe` 파일을 찾아 실행한다.14
   - 설치 마법사가 시작되면, 기본 설정에 따라 모든 권장 드라이버와 에이전트(Guest Agent)를 설치하도록 진행한다. 이 통합 설치 프로그램은 네트워크(`NetKVM`), 메모리 벌루닝(`Balloon`), 시리얼 통신(`vioserial`) 등 필요한 모든 드라이버를 자동으로 설치해준다.
3. **설치 완료 및 재부팅:** 설치가 완료되면 시스템을 재부팅하라는 메시지가 나타난다. 재부팅 후 다시 장치 관리자를 확인하면, 이전에 경고 아이콘이 있던 모든 장치들이 '네트워크 어댑터', '시스템 장치' 등 올바른 범주 아래에 정상적으로 인식된 것을 볼 수 있다. 또한, 네트워크 연결이 활성화되어 인터넷 사용이 가능해진다.


SPICE는 단순한 원격 디스플레이 프로토콜인 VNC를 넘어, KVM 게스트와의 긴밀한 통합을 위한 고급 기능을 제공한다. 이러한 기능을 완전히 활용하려면 게스트 내에 SPICE 게스트 도구를 설치해야 한다.

- **SPICE 프로토콜의 장점:**

  - **향상된 그래픽 성능:** QXL 가상 비디오 드라이버를 통해 VNC보다 훨씬 부드러운 2D 그래픽 가속을 제공한다.52
  - **양방향 클립보드 공유:** 호스트와 게스트 간에 텍스트를 자유롭게 복사하고 붙여넣을 수 있다.54
  - **화면 해상도 자동 조절:** `virt-viewer`나 `virt-manager` 콘솔 창의 크기를 변경하면 게스트의 화면 해상도가 그에 맞게 동적으로 변경된다.52
  - **USB 장치 리디렉션:** 호스트에 연결된 USB 장치를 게스트에서 직접 사용할 수 있다.
  - **오디오 리디렉션:** 게스트의 소리를 호스트의 스피커로 들을 수 있다.53

- virt-manager 설정 확인:

  SPICE 기능이 제대로 작동하려면 VM의 가상 하드웨어 구성이 올바르게 설정되어 있어야 한다. virt-manager에서 VM의 하드웨어 세부 정보를 열고 다음을 확인한다.

  1. **디스플레이:** 'SPICE 서버'로 설정되어 있어야 한다.
  2. **비디오:** 모델이 'QXL'로 설정되어 있어야 한다.
  3. **채널:** `com.redhat.spice.0`이라는 이름의 'Spice aget (spicevmc)' 유형 채널 장치가 존재해야 한다. 이 채널은 호스트와 게스트 에이전트 간의 통신 통로 역할을 하므로, 클립보드 공유와 같은 기능에 필수적이다. 만약 없다면 '하드웨어 추가'를 통해 생성해야 한다.53

- **SPICE 게스트 도구 설치:**

  1. 윈도우 게스트 내에서 웹 브라우저를 실행하여 SPICE 공식 다운로드 페이지에 접속한다: `https://www.spice-space.org/download.html`.58
  2. 'Windows Binaries' 섹션에서 'Windows SPICE Guest Tools'의 최신 설치 파일(`spice-guest-tools-latest.exe`)을 다운로드한다.54
  3. 다운로드한 설치 파일을 실행하고, 마법사의 지시에 따라 설치를 완료한다. 이 과정에서 QXL 그래픽 드라이버와 SPICE VDAgent 서비스가 설치된다.
  4. 설치 완료 후 시스템을 재부팅한다.

- **기능 활성화 및 확인:**

  - 재부팅 후, 호스트와 게스트 간에 텍스트를 복사하여 붙여넣기가 되는지 확인한다.
  - `virt-manager`의 VM 콘솔 창에서 '보기' > '크기 조정' 메뉴로 이동하여 '창에 맞게 게스트 자동 조절' 옵션이 체크되어 있는지 확인한다. 이 상태에서 콘솔 창의 크기를 조절하면 윈도우의 해상도가 실시간으로 변경되어야 한다.56


기본적으로 `libvirt`는 VM을 위한 격리된 가상 네트워크를 생성하고 NAT(Network Address Translation)를 통해 외부 인터넷 접속을 제공한다. 대부분의 경우 이 설정으로 충분하지만, VM을 호스트와 동일한 로컬 네트워크(LAN)의 일원으로 만들어야 할 때는 브리지(Bridged) 네트워킹 설정이 필요하다.

- **기본 NAT 네트워크:**
  - `libvirt` 설치 시 `default`라는 이름의 가상 네트워크가 자동으로 생성된다. 이 네트워크는 호스트에 `virbr0`라는 가상 브리지 인터페이스를 만들고, `192.168.122.0/24`와 같은 사설 IP 대역을 VM에 할당한다.62
  - VM에서 발생하는 트래픽은 호스트의 IP 주소를 통해 NAT 처리되어 외부로 나가게 된다. 이 방식은 설정이 간편하고 보안에 유리하지만, 외부 네트워크의 다른 장치들이 VM에 직접 접근할 수는 없다.63
- **브리지(Bridged) 네트워크:**
  - 브리지 네트워크는 가상 네트워크 스위치처럼 작동하여 VM의 가상 네트워크 인터페이스를 호스트의 물리적 네트워크 인터페이스(예: `enp3s0`)에 직접 연결한다.65
  - 이 결과, VM은 호스트와 동일한 LAN에 속하게 되며, LAN의 DHCP 서버로부터 직접 IP 주소를 할당받는다. 이를 통해 LAN 내의 다른 모든 장치와 양방향으로 자유롭게 통신할 수 있다.64
  - **설정 방법 (NetworkManager 사용 시):**
    1. 호스트 터미널에서 `nmtui` (텍스트 기반 UI) 또는 `nm-connection-editor` (GUI)를 실행한다.
    2. 새로운 연결을 추가하고, 연결 유형으로 '브리지(Bridge)'를 선택한다.
    3. 브리지 인터페이스의 이름(예: `br0`)을 지정한다.
    4. 브리지된 연결(slaves)을 추가하고, 호스트의 주 물리적 네트워크 인터페이스(예: `enp3s0`)를 선택한다.65
    5. 기존 물리적 인터페이스의 IP 설정(DHCP 또는 고정 IP)을 새로 생성된 브리지 인터페이스(`br0`)로 옮긴다. 이제 호스트의 모든 네트워크 트래픽은 `br0`를 통해 처리된다.
    6. `virt-manager`에서 VM의 NIC 하드웨어 설정으로 이동한다. '네트워크 소스'를 '브리지 장치'로 변경하고, '장치 이름' 필드에 `br0`를 입력한다.66
    7. VM을 재부팅하면 LAN의 DHCP 서버로부터 새로운 IP 주소를 할당받게 된다.

이로써 윈도우 게스트는 완벽하게 기능하는 상태가 되며, 사용 목적에 맞는 네트워크 환경까지 구성되었다. 다음 장에서는 성능을 한계까지 끌어올리는 고급 튜닝 기법과 발생 가능한 문제들에 대한 해결책을 다룬다.


기본적인 설치와 최적화가 완료된 가상 머신은 대부분의 작업을 무리 없이 수행할 수 있다. 하지만 고사양 게임, 비디오 인코딩, 대규모 컴파일과 같이 극한의 성능을 요구하는 작업을 위해서는 추가적인 고급 튜닝이 필요하다. 또한, 가상화 환경의 복잡성으로 인해 예기치 않은 문제들이 발생할 수 있다. 이 장에서는 성능을 극대화하는 기법과 일반적인 문제들에 대한 체계적인 해결 방안을 제시한다.


다음 기법들은 시스템 리소스를 VM에 보다 직접적이고 효율적으로 할당하여 오버헤드를 최소화하는 데 중점을 둔다.

- **CPU 피닝(Pinning):**

  - **개념:** 기본적으로 게스트의 가상 CPU(vCPU)는 호스트의 물리적 CPU 코어들 사이를 자유롭게 이동하며 스케줄링된다. 이는 유연성을 제공하지만, vCPU가 다른 코어로 이동할 때마다 해당 코어의 L1/L2 캐시가 무효화되어 캐시 미스(cache miss)가 발생하고 성능 저하를 유발할 수 있다. CPU 피닝은 특정 vCPU를 특정 물리적 CPU 코어(또는 스레드)에 영구적으로 할당(고정)하는 기술이다.42

  - **효과:** vCPU가 항상 동일한 물리 코어에서 실행되므로 캐시 효율성이 극대화되고, 호스트의 다른 프로세스와의 경합이 줄어들어 지연 시간(latency)이 감소하고 일관된 성능을 제공한다. 특히 실시간성이 중요한 애플리케이션이나 게임에서 효과가 크다.

  - **설정 방법:** `sudo virsh edit <VM이름>` 명령으로 VM의 XML 설정을 열고, `<vcpu>` 및 `<cputune>` 섹션을 다음과 같이 수정한다. 아래 예시는 4개의 vCPU를 호스트의 물리 코어 2, 3, 4, 5에 각각 고정하는 경우다.

    ```XML
    <vcpu placement='static'>4</vcpu>
    <cputune>
      <vcpupin vcpu='0' cpuset='2'/>
      <vcpupin vcpu='1' cpuset='3'/>
      <vcpupin vcpu='2' cpuset='4'/>
      <vcpupin vcpu='3' cpuset='5'/>
    </cputune>
    ```

- **HugePages:**

  - **개념:** 현대 CPU는 가상 메모리 주소를 물리 메모리 주소로 변환하기 위해 TLB(Translation Lookaside Buffer)라는 고속 캐시를 사용한다. 리눅스의 기본 메모리 페이지 크기는 4KB로 매우 작기 때문에, 대용량 메모리를 사용하는 VM은 수많은 페이지 테이블 항목을 필요로 하고 이는 TLB 미스를 자주 유발하여 성능 저하의 원인이 된다. HugePages는 2MB 또는 1GB와 같은 훨씬 큰 메모리 페이지를 사용하여 페이지 테이블 항목의 수를 획기적으로 줄여 TLB 부하를 감소시키는 기술이다.42

  - **효과:** 메모리 집약적인 애플리케이션의 성능을 크게 향상시킨다.

  - **설정 방법:**

    1. **호스트에서 HugePages 예약:** 먼저 호스트 부팅 시에 HugePages를 예약해야 한다. GRUB 설정 파일(`/etc/default/grub`)의 커널 파라미터에 `hugepagesz=1G hugepages=8`과 같이 추가하여 1GB 페이지 8개(총 8GB)를 예약하고 시스템을 재부팅한다.44

    2. **VM XML 설정:** `sudo virsh edit <VM이름>`으로 XML 설정을 열고, `<domain>` 태그 최하단에 다음 섹션을 추가한다.

       ```XML
       <memoryBacking>
         <hugepages/>
       </memoryBacking>
       ```

       이 설정은 `libvirt`가 VM의 메모리를 예약된 HugePages에서 할당하도록 지시한다.45

- **Hyper-V Enlightenments (계몽):**

  - **개념:** Windows는 자사의 하이퍼바이저인 Hyper-V 환경에서 실행될 때, 성능 향상을 위한 특정 최적화 기능(Enlightenments)을 사용한다. KVM은 이러한 Hyper-V의 인터페이스 일부를 에뮬레이션하여 Windows 게스트가 KVM 환경을 Hyper-V로 착각하게 만들고, 이를 통해 성능을 향상시킬 수 있다.4

  - **효과:** 타이머, 스핀락(spinlock), APIC(Advanced Programmable Interrupt Controller) 등의 처리 방식이 최적화되어 CPU 사용률이 감소하고 VM의 전반적인 반응성이 향상된다.42

  - **설정 방법:** `sudo virsh edit <VM이름>`으로 XML 설정을 열고, `<features>` 섹션 아래에 다음과 같이 `<hyperv>` 블록을 추가하거나 수정한다.

    ```XML
    <features>
    ...
      <hyperv>
        <relaxed state='on'/>
        <vapic state='on'/>
        <spinlocks state='on' retries='8191'/>
        <vpindex state='on'/>
        <runtime state='on'/>
        <synic state='on'/>
        <stimer state='on'/>
        <reset state='on'/>
        <frequencies state='on'/>
      </hyperv>
    ...
    </features>
    ```


KVM 환경에서 윈도우 게스트를 운영할 때 사용자들이 빈번하게 겪는 문제들과 그 해결 방안을 체계적으로 정리하면 다음과 같다. 이 표는 증상, 예상 원인, 그리고 구체적인 해결 절차를 제공하여 신속한 문제 진단을 돕는다.

**표 1: KVM 윈도우 10 게스트 문제 해결 가이드**

| 문제 현상                        | 예상 원인                                                    | 해결 절차                                                    | 관련 자료 |
| -------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | --------- |
| **성능 저하/느림**               | 1. VirtIO 드라이버 미설치 또는 비활성화 2. CPU 설정 미흡 (host-passthrough 미사용) 3. 디스크 캐시 모드 비효율 4. 호스트 CPU 거버너가 'powersave'로 설정됨 | 1. `virt-manager`에서 디스크/네트워크 장치 모델을 `VirtIO`로 변경하고, 게스트 내에서 `virtio-win-guest-tools.exe`를 실행하여 드라이버를 설치한다. 2. VM XML 설정에서 CPU 모드를 `host-passthrough`로 변경하여 CPU 에뮬레이션 오버헤드를 제거한다. 3. 디스크 성능 옵션에서 캐시 모드를 `writeback` 또는 `none`으로 변경하여 I/O 성능을 튜닝한다. 4. 호스트 터미널에서 `cpupower frequency-set -g performance` 등의 명령어로 CPU 거버너를 `performance` 또는 `ondemand`로 변경한다. | 4         |
| **네트워크 연결 불가**           | 1. `NetKVM` VirtIO 드라이버 미설치 2. `libvirt` 기본 NAT 네트워크(`default`) 비활성화 3. 호스트 방화벽 규칙 문제 4. 브리지 네트워크 설정 오류 | 1. 게스트의 장치 관리자에서 '이더넷 컨트롤러'를 확인하고, `virtio-win.iso`를 마운트하여 드라이버를 수동으로 업데이트하거나 `virtio-win-guest-tools.exe`를 재설치한다. 2. 호스트에서 `sudo virsh net-start default` 및 `sudo virsh net-autostart default` 명령을 실행하여 `default` 네트워크를 활성화한다. 3. `firewalld` 또는 `iptables` 설정에서 `virbr0` 인터페이스를 신뢰하는 영역(trusted zone)에 추가하거나 관련 트래픽을 허용하는 규칙을 추가한다. 4. 브리지에 물리 NIC가 올바르게 연결되었는지(`brctl show`), VM이 해당 브리지를 사용하도록 설정되었는지 확인한다. | 73        |
| **클립보드 공유 안 됨**          | 1. SPICE 게스트 도구 미설치 2. `spice-vdagent` 서비스가 게스트에서 실행되지 않음 3. VM에 `spicevmc` 채널 하드웨어가 없음 4. 펌웨어가 UEFI일 때 발생하는 호환성 문제 | 1. 게스트 내에서 `spice-space.org`에 접속하여 `spice-guest-tools-latest.exe`를 다운로드 및 설치한다. 2. 게스트의 `services.msc`에서 'Spice VDAgent' 서비스가 '실행 중'이고 시작 유형이 '자동'인지 확인한다. 문제가 지속되면 서비스를 재시작한다. 3. `virt-manager`에서 VM의 하드웨어 추가 기능을 통해 '채널' 장치를 추가하고, 유형을 'Spice agent (spicevmc)'로 선택한다. 4. 드물게 UEFI 펌웨어와 SPICE 에이전트 간의 호환성 문제가 보고된다. 문제가 지속되면 VM을 BIOS 펌웨어로 재생성하여 테스트해볼 수 있다. | 53        |
| **화면 자동 크기 조절 안 됨**    | 1. SPICE 게스트 도구 미설치 2. 비디오 모델이 `QXL`이 아님 3. `virt-manager` 콘솔 설정 비활성화 4. 게스트 내 `spice-vdagent` 문제 | 1. 게스트에 `spice-guest-tools-latest.exe`를 설치한다. 2. VM 하드웨어 설정에서 '비디오' 장치의 모델을 `QXL`로 설정한다. `VirtIO` 비디오는 3D 가속에 유리하지만, 자동 리사이징은 QXL과 SPICE 에이전트의 조합에서 가장 안정적으로 동작한다. 3. `virt-manager`의 VM 콘솔 창 메뉴에서 '보기' -> '크기 조정' -> '창에 맞게 게스트 자동 조절'이 활성화되어 있는지 확인한다. 4. 게스트에서 `spice-vdagent` 서비스를 재시작한다. | 52        |
| **오디오 문제 (소리 없음/깨짐)** | 1. VM에 오디오 장치가 없음 2. `libvirt`의 오디오 백엔드 설정 문제 3. `libvirt` 데몬의 사용자 권한 문제 4. 호스트와 게스트의 샘플 레이트 불일치 | 1. VM 하드웨어 설정에서 '사운드' 장치를 추가하고, 모델로 `ich6` 또는 `ac97`을 선택한다. 2. `/etc/libvirt/qemu.conf` 파일을 편집하여 `vnc_allow_host_audio = 1` 주석을 해제하고, `user = "your_username"`을 설정하여 `libvirtd`가 일반 사용자 권한으로 오디오 장치에 접근할 수 있도록 한다. 이후 `sudo systemctl restart libvirtd`로 서비스를 재시작한다. 3. 호스트와 게스트 양쪽의 사운드 설정에서 출력 샘플 레이트(예: 16비트, 44100Hz)를 동일하게 맞추어 오디오 깨짐 현상을 방지한다. | 68        |

이러한 체계적인 접근법을 통해 사용자는 KVM 환경에서 발생하는 대부분의 일반적인 문제들을 효과적으로 진단하고 해결할 수 있다.


지금까지 KVM 하이퍼바이저 위에 윈도우 10 가상 머신을 구축하고, 최적의 성능을 이끌어내며, 발생 가능한 문제들을 해결하는 전 과정을 상세히 살펴보았다. KVM은 그 자체로 강력하고 유연한 가상화 솔루션이지만, 그 잠재력을 완전히 활용하기 위해서는 아키텍처에 대한 이해를 바탕으로 한 세심한 구성과 최적화 과정이 필수적이다. 본 안내서에서 다룬 핵심 사항들을 최종적으로 점검함으로써, 사용자는 안정적이고 고성능의 윈도우 가상 환경을 완성하고 유지할 수 있다.


성공적인 KVM 윈도우 10 가상 환경을 구축하기 위한 가장 중요한 설정들을 다시 한번 요약하면 다음과 같다. 이 항목들은 최소한으로 만족시켜야 할 필수 조건이다.

- **호스트 시스템 준비:**
  - **하드웨어 가상화 활성화:** BIOS/UEFI 설정에서 Intel VT-x 또는 AMD-V 기능이 반드시 활성화되어 있어야 한다.
  - **필수 패키지 설치:** `qemu-kvm`, `libvirt-daemon-system`, `virt-manager` 등 배포판에 맞는 가상화 패키지 그룹을 설치한다.
  - **`libvirtd` 데몬 및 권한:** `libvirtd` 서비스가 시스템 부팅 시 자동으로 실행되도록 활성화하고, `sudo` 없이 관리 도구를 사용하기 위해 현재 사용자를 `libvirt` 및 `kvm` 그룹에 추가한다.
- **가상 머신 하드웨어 구성:**
  - **칩셋 및 펌웨어:** 현대적인 `Q35` 칩셋과 `UEFI` 펌웨어를 선택하여 최신 기능과 호환성을 확보한다.
  - **CPU 모델:** CPU 에뮬레이션 오버헤드를 제거하기 위해 `host-passthrough` 모드를 사용한다.
  - **핵심은 VirtIO:** 디스크와 네트워크 인터페이스의 버스/모델 유형을 반드시 `VirtIO`로 설정하여 반가상화의 성능 이점을 극대화한다.
- **게스트 운영체제 최적화:**
  - **VirtIO 드라이버 설치:** 윈도우 설치 중 `viostor` 드라이버를 로드하고, 설치 완료 후에는 `virtio-win-guest-tools.exe`를 실행하여 모든 관련 드라이버를 설치한다.
  - **SPICE 게스트 도구 설치:** 클립보드 공유, 화면 자동 리사이징 등 향상된 사용자 경험을 위해 `spice-guest-tools-latest.exe`를 설치하고, VM에 `spicevmc` 채널이 구성되어 있는지 확인한다.


본 안내서에서는 `writeback` 디스크 캐싱, CPU 오버커밋, HugePages 등 다양한 고급 성능 튜닝 기법을 소개했다. 이러한 설정들은 특정 워크로드에서 성능을 극적으로 향상시킬 수 있지만, 시스템의 안정성에 영향을 미칠 수 있는 트레이드오프 관계를 가진다. 예를 들어, `writeback` 캐싱은 호스트의 갑작스러운 전원 차단 시 데이터 유실의 위험을 내포하고 있으며, 과도한 CPU 및 메모리 오버커밋은 리소스 경합으로 인한 시스템 전체의 불안정성을 초래할 수 있다. 따라서 이러한 고급 옵션을 적용할 때는 운영하는 서비스의 특성과 요구되는 안정성 수준을 신중하게 고려하여, 충분한 테스트를 거친 후 단계적으로 도입하는 것이 현명하다.


최적의 가상 환경을 구축하는 것만큼이나 중요한 것은 그것을 지속적으로 관리하고 최신 상태로 유지하는 것이다.

- **호스트 시스템 업데이트:** 리눅스 커널, QEMU, `libvirt` 패키지는 지속적으로 버그가 수정되고 성능이 개선되므로, 호스트 시스템의 패키지 관리자를 통해 정기적으로 업데이트를 적용하는 것이 좋다.
- **VirtIO 드라이버 업데이트:** Fedora 프로젝트에서 새로운 안정(stable) 버전의 `virtio-win.iso`가 출시되면, 이를 다운로드하여 게스트 내의 드라이버를 업데이트할 것을 권장한다. 이는 새로운 윈도우 업데이트와의 호환성을 보장하고 잠재적인 성능 및 안정성 문제를 해결하는 데 도움이 된다.11

KVM과 윈도우의 조합은 올바르게 구성되었을 때, 개발, 테스트, 일반 사무, 심지어 고사양 게이밍에 이르기까지 광범위한 요구를 만족시킬 수 있는 강력한 플랫폼이다. 본 안내서가 사용자들이 KVM의 복잡성을 극복하고, 자신의 필요에 맞는 최적의 윈도우 가상 환경을 성공적으로 구축하는 데 든든한 기반이 되기를 바란다.


1. How to Install KVM on Ubuntu 24.04: Step-By-Step | Cherry Servers, accessed August 17, 2025, https://www.cherryservers.com/blog/install-kvm-ubuntu
2. How To Install and Use KVM on CentOS Stream 9|8 ..., accessed August 17, 2025, https://computingforgeeks.com/install-and-use-kvm-on-centos-stream/
3. How to Create Virtual Machines in Ubuntu Using QEMU/KVM Tool - Tecmint, accessed August 17, 2025, https://www.tecmint.com/install-qemu-kvm-ubuntu-create-virtual-machines/
4. Improving the performance of a Windows 11 Guest on QEMU | by Leduccc - Medium, accessed August 17, 2025, https://leduccc.medium.com/improving-the-performance-of-a-windows-10-guest-on-qemu-a5b3f54d9cf5
5. Libvirt - Ubuntu Server documentation, accessed August 17, 2025, https://documentation.ubuntu.com/server/how-to/virtualisation/libvirt/
6. Installing libvirt and virt-install on Fedora Linux, accessed August 17, 2025, https://developer.fedoraproject.org/tools/virtualization/installing-libvirt-and-virt-install-on-fedora-linux.html
7. Virtual Machine Manager - Ubuntu Server documentation, accessed August 17, 2025, https://documentation.ubuntu.com/server/how-to/virtualisation/virtual-machine-manager/
8. Virtual Machine Manager, accessed August 17, 2025, https://virt-manager.org/
9. kvm - How to create a VM from scratch with virsh? - Unix & Linux ..., accessed August 17, 2025, https://unix.stackexchange.com/questions/309788/how-to-create-a-vm-from-scratch-with-virsh
10. Chapter 9. Installing a Fully-virtualized Windows Guest - Red Hat Documentation, accessed August 17, 2025, https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/6/html/virtualization_host_configuration_and_guest_installation_guide/sect-virtualization_host_configuration_and_guest_installation_guide-windows_installations-installing_windows_xp_as_a_fully_virtualized_guest
11. Installing/Updating KVM Virtio Drivers in Windows - STC Solutions KB, accessed August 17, 2025, https://kb.bluvalt.com/virtual-storage/install_update_virtio_drivers/
12. How To Create an Efficient Windows 10 Virtual Machine In Linux – QEMU/KVM - Reddit, accessed August 17, 2025, https://www.reddit.com/r/IntelligentGaming2020/comments/umok4z/how_to_create_an_efficient_windows_10_virtual/
13. Creating Windows virtual machines using virtIO drivers - Fedora Docs, accessed August 17, 2025, https://docs.fedoraproject.org/en-US/quick-docs/creating-windows-virtual-machines-using-virtio-drivers/
14. Running Windows 10 in a VM on Linux Mint with KVM, QEMU, and Virt-Manager, accessed August 17, 2025, https://nmanzi.com/posts/windows-guest-on-linux-mint/
15. eveng - neither intel vt-x or amd-v found - Proxmox Support Forum, accessed August 17, 2025, https://forum.proxmox.com/threads/eveng-neither-intel-vt-x-or-amd-v-found.135565/
16. linux - How to detect if VT-X has been turned on in the BIOS ..., accessed August 17, 2025, https://www.geeksforgeeks.org/linux-unix/linux-how-to-detect-if-vt-x-has-been-turned-on-in-the-bios/
17. HowTos/KVM - CentOS Wiki, accessed August 17, 2025, https://wiki.centos.org/HowTos/KVM
18. How to Install KVM on Ubuntu | phoenixNAP KB, accessed August 17, 2025, https://phoenixnap.com/kb/ubuntu-install-kvm
19. Enable AMD-V (virtualization) - Ask Ubuntu, accessed August 17, 2025, https://askubuntu.com/questions/739454/enable-amd-v-virtualization
20. How to determine if CPU VT extensions are enabled in bios? - Ask Ubuntu, accessed August 17, 2025, https://askubuntu.com/questions/103965/how-to-determine-if-cpu-vt-extensions-are-enabled-in-bios
21. How to detect if VT-X has been turned on in the BIOS? - Super User, accessed August 17, 2025, https://superuser.com/questions/567208/how-to-detect-if-vt-x-has-been-turned-on-in-the-bios
22. Install KVM & Virtual Machine Manager on Ubuntu (Complete Tutorial) - YouTube, accessed August 17, 2025, https://www.youtube.com/watch?v=9ea_pKCOQ9Q
23. How to Install KVM on CentOS 8: A Step by Step Tutorial - Liquid Web, accessed August 17, 2025, https://www.liquidweb.com/blog/install-kvm-centos-8/
24. Install KVM on CentOS or Rocky Linux - phoenixNAP, accessed August 17, 2025, https://phoenixnap.com/kb/install-kvm-centos
25. Virtualization – Getting Started :: Fedora Docs, accessed August 17, 2025, https://docs.fedoraproject.org/en-US/quick-docs/virtualization-getting-started/
26. How to Install KVM on Fedora 37/36 Step-by-Step - LinuxTechi, accessed August 17, 2025, https://www.linuxtechi.com/how-to-install-kvm-on-fedora-step-by-step/
27. How to enable nested virtualisation - Ubuntu Server documentation, accessed August 17, 2025, https://documentation.ubuntu.com/server/how-to/virtualisation/enable-nested-virtualisation/
28. Download Windows 10 Disc Image (ISO File) - Microsoft, accessed August 17, 2025, https://www.microsoft.com/software-download/windows10
29. Download Windows 10 ISO files, save a copy before end of support, accessed August 17, 2025, https://www.windowslatest.com/2025/08/08/download-windows-10-iso-version-22h2-before-end-of-support/
30. Windows 10 Download | MAS, accessed August 17, 2025, https://massgrave.dev/windows_10_links
31. Windows virtio drivers - KubeVirt user guide, accessed August 17, 2025, https://kubevirt.io/user-guide/user_workloads/windows_virtio_drivers/
32. 10.2. Installing the Drivers on an Installed Windows Guest Virtual Machine, accessed August 17, 2025, https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/6/html/virtualization_host_configuration_and_guest_installation_guide/form-virtualization_host_configuration_and_guest_installation_guide-para_virtualized_drivers-mounting_the_image_with_virt_manager
33. Can't install windows 10 because hard drive is not found, but I put a hard drive in the VM. What is the problem? : r/kvm - Reddit, accessed August 17, 2025, https://www.reddit.com/r/kvm/comments/mgv722/cant_install_windows_10_because_hard_drive_is_not/
34. 6.3. Creating Guests with virt-manager | Virtualization Host ..., accessed August 17, 2025, https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/6/html/virtualization_host_configuration_and_guest_installation_guide/chap-virtualization_host_configuration_and_guest_installation_guide-guest_installation_virt_manager-creating_guests_with_virt_manager
35. Creating a new virtual machine in Virtual Machine Manager - libvirt Wiki, accessed August 17, 2025, https://wiki.libvirt.org/CreatingNewVM_in_VirtualMachineManager.html
36. Allocating virtual machine resources with virt-manager - Fedora Discussion, accessed August 17, 2025, https://discussion.fedoraproject.org/t/allocating-virtual-machine-resources-with-virt-manager/76357
37. How many CPUs to allocate in virt-manager : r/VFIO - Reddit, accessed August 17, 2025, https://www.reddit.com/r/VFIO/comments/4q9mrj/how_many_cpus_to_allocate_in_virtmanager/
38. How to allocate CPU resources for a VM correctly? - Proxmox Support Forum, accessed August 17, 2025, https://forum.proxmox.com/threads/how-to-allocate-cpu-resources-for-a-vm-correctly.132322/
39. Quickly create guest VMs using virsh, cloud image files, and cloud-init | Earl C. Ruby III, accessed August 17, 2025, https://earlruby.org/2023/02/quickly-create-guest-vms-using-virsh-cloud-image-files-and-cloud-init/
40. Create Virtual Machines in KVM Virt-Manager like a Pro: All Options Explained - SysGuides, accessed August 17, 2025, https://sysguides.com/create-virtual-machines-in-kvm-virt-manager
41. How to run virtual machines with virt-manager - Fedora Magazine, accessed August 17, 2025, https://fedoramagazine.org/full-virtualization-system-on-fedora-workstation-30/
42. Comprehensive guide to performance optimizations for gaming on virtual machines with KVM/QEMU and PCI passthrough - MathiasHueber.com, accessed August 17, 2025, https://mathiashueber.com/performance-tweaks-gaming-on-virtual-machines/
43. Install Windows 10 from local .iso using Qemu - Page 2 - It's FOSS Community, accessed August 17, 2025, https://itsfoss.community/t/install-windows-10-from-local-iso-using-qemu/11593?page=2
44. ohthehugemanatee/win10vm: My performant Windows 10 VM configuration - GitHub, accessed August 17, 2025, https://github.com/ohthehugemanatee/win10vm
45. Windows 10 guest poor performance : r/kvm - Reddit, accessed August 17, 2025, https://www.reddit.com/r/kvm/comments/i3hhwp/windows_10_guest_poor_performance/
46. Setting up a Windows 10 VM with QEmu on Ubuntu 22.04 | Ecology, Stats & Bioacoustics, accessed August 17, 2025, https://rtbecard.gitlab.io/2022/07/23/QEmu_win10.html
47. How to install Windows 10 with KVM? - Mevspace Docs, accessed August 17, 2025, https://docs.mevspace.com/tutorials/how-to-install-windows10-with-kvm
48. How to Create Partition to Install Windows 10 - YouTube, accessed August 17, 2025, https://www.youtube.com/watch?v=2kKw8bq9CA8&pp=0gcJCfwAo7VqN5tD
49. Steps to Make Disk Partition to Install Windows 10/8/7 - Cocosenor, accessed August 17, 2025, https://www.cocosenor.com/articles/windows-10/how-to-make-disk-partition-to-install-windows-10-8-7.html
50. How To Partition Disk Drive During Windows Installation | Windows 7/8/10/11 - YouTube, accessed August 17, 2025, https://www.youtube.com/watch?v=ukFkMSO59m0
51. Driver installation · virtio-win/kvm-guest-drivers-windows Wiki - GitHub, accessed August 17, 2025, https://github.com/virtio-win/kvm-guest-drivers-windows/wiki/Driver-installation
52. About Windows 10 KVM guest installation - Virtualization - openSUSE Forums, accessed August 17, 2025, https://forums.opensuse.org/t/about-windows-10-kvm-guest-installation/128073
53. Spice User Manual - spice-space.org, accessed August 17, 2025, https://www.spice-space.org/spice-user-manual.html
54. Spice client agents for Windows - GitHub Gist, accessed August 17, 2025, https://gist.github.com/kurtis318/4f703c2ca50e7ab0db815954ef3ed03c
55. Getting two way clipboard between host and guest with virt-manager and Spice | skybert.net, accessed August 17, 2025, https://skybert.net/linux/getting-two-way-clipboard-between-host-and-guest-with-virt-manager-and-spice/
56. How to set up dynamic screen sizing for a qemu windows 10 guest on ubuntu 18.04?, accessed August 17, 2025, https://askubuntu.com/questions/1219817/how-to-set-up-dynamic-screen-sizing-for-a-qemu-windows-10-guest-on-ubuntu-18-04
57. QEMU/KVM Windows 10 Guest won't copy/paste text or files back to host, accessed August 17, 2025, https://unix.stackexchange.com/questions/500460/qemu-kvm-windows-10-guest-wont-copy-paste-text-or-files-back-to-host
58. Download - spice-space.org, accessed August 17, 2025, https://www.spice-space.org/download.html
59. How to Enable clipboard and folder sharing in Qemu/KVM on Windows Guest, accessed August 17, 2025, https://dausruddin.com/how-to-enable-clipboard-and-folder-sharing-in-qemu-kvm-on-windows-guest/
60. KVM: Unable to cut/paste between ubuntu host and windows 10 guest - Super User, accessed August 17, 2025, https://superuser.com/questions/1485479/kvm-unable-to-cut-paste-between-ubuntu-host-and-windows-10-guest
61. 3440x1440 resolution for Windows 10 KVM VM - GitHub Gist, accessed August 17, 2025, https://gist.github.com/PhilipSchmid/2c21146f83e197e7c39dad2b9a870485
62. Qemu, virt-manager and bridge network - Applications - EndeavourOS Forum, accessed August 17, 2025, https://forum.endeavouros.com/t/qemu-virt-manager-and-bridge-network/45005
63. virt-manager NAT virtual network not working : r/VFIO - Reddit, accessed August 17, 2025, https://www.reddit.com/r/VFIO/comments/r2rvsr/virtmanager_nat_virtual_network_not_working/
64. NAT forwarding (aka "virtual networks") - libvirt, accessed August 17, 2025, https://wiki.libvirt.org/Networking.html
65. How to set up a network bridge for virtual machine communication, accessed August 17, 2025, https://www.redhat.com/en/blog/setup-network-bridge-VM
66. Setting up bridged connection between VM (virt-manager 1.5.0) and CentOS 7.9 host, accessed August 17, 2025, https://discussion.fedoraproject.org/t/setting-up-bridged-connection-between-vm-virt-manager-1-5-0-and-centos-7-9-host/132244
67. Switch a KVM VM from NAT to Bridged on Oracle Linux - YouTube, accessed August 17, 2025, https://www.youtube.com/watch?v=hMstMTqzP_Q
68. Windows KVM: Tips and tricks for audio and improving performance - Level1Techs Forums, accessed August 17, 2025, https://forum.level1techs.com/t/windows-kvm-tips-and-tricks-for-audio-and-improving-performance/130620
69. Chapter 4. Optimizing Windows virtual machines - Red Hat Documentation, accessed August 17, 2025, https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/configuring_and_managing_windows_virtual_machines/optimizing-windows-virtual-machines
70. Windows 10 KVM guest slow performance - Virtualization - openSUSE Forums, accessed August 17, 2025, https://forums.opensuse.org/t/windows-10-kvm-guest-slow-performance/130643
71. Windows VM suddenly run very slow in KVM/QEMU - Server Fault, accessed August 17, 2025, https://serverfault.com/questions/1092404/windows-vm-suddenly-run-very-slow-in-kvm-qemu
72. Windows 10 VM poor performance, 100% CPU after 6.9 upgrade - Forums - Unraid, accessed August 17, 2025, https://forums.unraid.net/topic/103581-windows-10-vm-poor-performance-100-cpu-after-69-upgrade/
73. No Internet on Linux KVM Windows 10 guest using VirtIO drivers - Super User, accessed August 17, 2025, https://superuser.com/questions/1712343/no-internet-on-linux-kvm-windows-10-guest-using-virtio-drivers
74. No Network in Windows 10 VM even with virtio drivers installed : r/Proxmox - Reddit, accessed August 17, 2025, https://www.reddit.com/r/Proxmox/comments/rygj9n/no_network_in_windows_10_vm_even_with_virtio/
75. [SOLVED] No Network Connectivity in Virt Manager with KVM/QEMU - Arch Linux Forums, accessed August 17, 2025, https://bbs.archlinux.org/viewtopic.php?id=291898
76. Windows 10 KVM VM getting IP but no internet connection - openSUSE Forums, accessed August 17, 2025, https://forums.opensuse.org/t/windows-10-kvm-vm-getting-ip-but-no-internet-connection/175630
77. QEMU/KVM spice clipboard sharing not working on Windows guest : r/virtualization - Reddit, accessed August 17, 2025, https://www.reddit.com/r/virtualization/comments/117kqj1/qemukvm_spice_clipboard_sharing_not_working_on/
78. Fixing Windows 10 spice-guest-tools Agent crash-on-start | bazile.org, accessed August 17, 2025, https://bazile.org/writing/2023/windows_10_spice_guest_tools_agent.html
79. Virt-Manager: Auto adjust of resolution to screen size not working : r/qemu_kvm - Reddit, accessed August 17, 2025, https://www.reddit.com/r/qemu_kvm/comments/1efu3by/virtmanager_auto_adjust_of_resolution_to_screen/
80. Virt-Manager -- how to autoresize VM with window - Applications - EndeavourOS Forum, accessed August 17, 2025, https://forum.endeavouros.com/t/virt-manager-how-to-autoresize-vm-with-window/50360
81. Fixing Audio Issues with Your KVM Switch (No Sound, Mac & PC) - AvicoTech, accessed August 17, 2025, https://avicotech.com/blogs/kvm/no-audio-fix-mac-pc
82. Sound is not working in Windows virtual machine - Broadcom support portal, accessed August 17, 2025, https://knowledge.broadcom.com/external/article/339532/sound-is-not-working-in-windows-virtual.html
83. KVM Windows No Sound? – Let's fix it. - GetLabsDone, accessed August 17, 2025, https://getlabsdone.com/kvm-windows-no-sound-lets-fix-it/

