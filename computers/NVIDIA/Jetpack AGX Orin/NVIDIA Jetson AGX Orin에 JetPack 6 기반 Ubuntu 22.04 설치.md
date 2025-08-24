[NVIDIA Jetpack for AGX Orin](./index.md)
# NVIDIA Jetson AGX Orin에 JetPack 6 기반 Ubuntu 22.04 설치



NVIDIA Jetson AGX Orin은 엣지 AI 및 로보틱스 애플리케이션을 위한 고성능 컴퓨팅 플랫폼이다. 이 플랫폼의 핵심은 NVIDIA Ampere 아키텍처 GPU, Arm Cortex-A78AE CPU 클러스터, 그리고 차세대 딥러닝 및 비전 가속기를 통합한 시스템 온 모듈(System-on-Module, SoM)이다.1 이러한 이기종 컴퓨팅 자원의 조합은 자연어 이해, 3D 인식, 다중 센서 융합과 같은 복잡하고 까다로운 AI 워크로드를 실시간으로 처리할 수 있는 강력한 기반을 제공한다.

Jetson AGX Orin 개발자 키트는 이러한 모듈의 성능을 최대한 활용할 수 있도록 설계된 레퍼런스 하드웨어다. 개발자 키트는 고속 I/O 인터페이스와 다양한 스토리지 옵션을 제공한다. 기본적으로는 모듈에 내장된 eMMC 플래시 메모리를 사용하지만, M.2 M-Key 슬롯을 통해 고속 NVMe SSD를 추가 장착할 수 있으며, USB 포트를 통한 외부 저장 장치 연결도 지원한다.2 운영체제(OS)를 어떤 저장 매체에 설치할 것인지는 전체 시스템의 성능과 부팅 속도에 직접적인 영향을 미치므로, 설치 과정에서 저장 매체를 신중하게 선택하는 것이 중요하다.


Jetson 플랫폼의 소프트웨어 환경을 이해하기 위해서는 JetPack SDK, Jetson Linux, 그리고 Ubuntu의 관계를 명확히 파악해야 한다.

**NVIDIA JetPack SDK**는 Jetson 모듈을 위한 가장 포괄적인 소프트웨어 개발 솔루션이다.3 이는 단순히 운영체제를 의미하는 것이 아니라, AI 애플리케이션 개발에 필요한 모든 요소를 포함하는 통합 개발 환경이다. JetPack은 부트로더, Linux 커널, Ubuntu 데스크톱 환경을 포함하는 **Jetson Linux**와 GPU 컴퓨팅, 멀티미디어, 그래픽스, 컴퓨터 비전 가속을 위한 완벽한 라이브러리 세트(CUDA, cuDNN, TensorRT, VPI 등)를 모두 포함한다.3

**Jetson Linux**는 과거 'Linux for Tegra (L4T)'로 알려졌던 보드 지원 패키지(Board Support Package, BSP)의 공식 명칭이다. 이것이 바로 Jetson 하드웨어에서 직접 실행되는 운영체제이며, UEFI 기반 부트로더, Linux 커널, NVIDIA 드라이버, 그리고 특정 버전의 Ubuntu를 기반으로 하는 루트 파일 시스템(rootfs)으로 구성된다.4

사용자가 요구하는 "Tegra Ubuntu 22.04"는 바로 이 Jetson Linux의 구성 요소 중 하나인 루트 파일 시스템이 Ubuntu 22.04 (Jammy Jellyfish) 버전을 기반으로 하는 경우를 지칭한다. 이는 NVIDIA Jetson 소프트웨어 스택에서 **JetPack 6.x** 버전을 통해서만 제공된다.4 이전 버전인 JetPack 5.x는 Ubuntu 20.04 (Focal Fossa)를 기반으로 하므로, Ubuntu 22.04 환경을 구축하기 위해서는 반드시 JetPack 6.x 버전을 선택해야 한다.3 이 종속성은 설치 과정 전체를 관통하는 핵심적인 전제 조건이다.

이러한 관계를 명확히 하기 위해 아래의 버전 매핑 테이블을 참조해야 한다.

| JetPack SDK 버전                                 | Jetson Linux (L4T) 버전 | 기반 Ubuntu 버전            |
| ------------------------------------------------ | ----------------------- | --------------------------- |
| JetPack 6.x                                      | R36.x                   | 22.04 LTS (Jammy Jellyfish) |
| JetPack 5.x                                      | R35.x                   | 20.04 LTS (Focal Fossa)     |
| JetPack 4.x                                      | R32.x                   | 18.04 LTS (Bionic Beaver)   |
| *표 1: JetPack 버전별 L4T 및 Ubuntu 버전 매핑* 4 |                         |                             |


Jetson AGX Orin에 Jetson Linux를 설치하는 방법은 크게 두 가지로 나뉜다.

1. **NVIDIA SDK Manager (GUI):** 대부분의 개발자에게 권장되는 표준 설치 도구다. 그래픽 사용자 인터페이스(GUI)를 통해 설치 과정을 단계별로 안내하며, 호스트 PC와 대상 Jetson 장치에 필요한 모든 소프트웨어 구성 요소를 관리하고 설치하는 엔드투엔드 솔루션을 제공한다.7 이 방법은 복잡성을 줄이고 사용자 실수를 최소화하는 데 중점을 둔다.
2. **명령줄 인터페이스 (CLI):** `flash.sh` 스크립트를 사용하는 고급 설치 방법이다. 이 방법은 설치 프로세스를 자동화하거나, 지속적 통합/지속적 배포(CI/CD) 파이프라인에 통합하거나, 커널이나 디바이스 트리와 같은 저수준 시스템 구성 요소를 직접 수정해야 하는 고급 사용자에게 적합하다.10 CLI 방식은 더 높은 유연성을 제공하지만, 정확한 매개변수와 절차에 대한 깊은 이해를 요구한다.

본 문서는 두 가지 방법론을 모두 상세히 다루어, 사용자의 기술 수준과 요구사항에 맞는 최적의 설치 경로를 제공하는 것을 목표로 한다.


Jetson AGX Orin에 운영체제를 성공적으로 설치하기 위해서는 호스트 PC와 대상 장치 모두에 대한 철저한 사전 준비가 필수적이다. 이 단계에서의 준비 상태가 전체 설치 과정의 성패를 좌우할 수 있다.


Jetson 장치에 OS를 플래싱하는 작업은 Jetson 자체에서 수행되는 것이 아니라, 별도의 호스트 PC에서 NVIDIA SDK Manager를 통해 이루어진다. 따라서 호스트 PC는 특정 요구사항을 충족해야 한다.

- **운영체제:** JetPack 6.x 버전을 설치하기 위한 SDK Manager는 **Ubuntu Desktop 20.04 또는 22.04 (x86_64)**가 설치된 호스트 PC를 공식적으로 지원한다.7 또한, Windows 10 및 11 (Windows Subsystem for Linux, WSL 사용)과 Docker 컨테이너 환경도 지원되어 개발 환경의 유연성을 높였다.7
- **권장 사항:** 가장 안정적이고 원활한 설치 경험을 위해서는 대상 장치(Jetson)에 설치될 OS와 동일한 **네이티브 Ubuntu 22.04 환경의 호스트 PC를 사용하는 것이 강력히 권장된다.** 커뮤니티에서는 새로 설치된(fresh install) Ubuntu 20.04 호스트에서 USB 타임아웃과 같은 플래싱 실패 사례가 빈번하게 보고되고 있다.14 이는 Ubuntu 20.04의 기본 USB 자동 절전(autosuspend) 설정과 같은 미묘한 구성 차이에서 비롯될 수 있다.16 따라서 잠재적인 호환성 문제를 최소화하기 위해 Ubuntu 22.04를 주 호스트 OS로 사용하는 것이 바람직하다.
- **하드웨어:** 최소 **8GB의 시스템 메모리(RAM)**와 각 JetPack 버전을 설치하기 위한 **최소 30GB 이상의 여유 디스크 공간**이 필요하다.12 Jetson OS 이미지와 SDK 컴포넌트들은 상당한 저장 공간을 차지하므로 충분한 공간을 확보하는 것이 중요하다.
- **네트워크:** SDK Manager가 NVIDIA 서버로부터 필요한 소프트웨어 패키지를 다운로드해야 하므로, 안정적인 인터넷 연결이 필수적이다.12 방화벽이나 프록시 환경에서는 연결 문제가 발생할 수 있으므로, 사전에 네트워크 설정을 확인해야 한다.


- **필수 하드웨어:**
  - NVIDIA Jetson AGX Orin 개발자 키트
  - 키트에 포함된 정격 전원 공급 장치 (DC 잭 또는 USB-C PD) 17
  - 데이터 전송이 가능한 고품질 USB Type-A to Type-C 케이블. 이 케이블은 호스트 PC와 Jetson을 연결하는 데 사용된다.
- **핵심 주의사항:** 플래싱 작업 시에는 반드시 Jetson AGX Orin 개발자 키트의 **40핀 헤더 옆에 위치한 USB-C 포트**를 사용해야 한다. 이 포트는 UFP(Upstream Facing Port)로 기능하여 Jetson이 호스트 PC의 장치로 인식되도록 한다.2 다른 USB-C 포트를 사용하면 플래싱이 진행되지 않는다.
- **주변 장치:**
  - OS 설치 후 초기 설정을 위해 DisplayPort를 지원하는 모니터, USB 키보드, USB 마우스가 필요하다.19 Jetson AGX Orin 개발자 키트는 DisplayPort 출력만 지원하므로, HDMI 입력만 있는 모터를 사용하는 경우 DisplayPort to HDMI 변환 어댑터가 필요할 수 있다.2
  - 안정적인 네트워크 연결을 위한 이더넷 케이블.
- **저장 매체:**
  - 기본 저장소는 모듈에 내장된 eMMC 플래시 메모리다.
  - 더 빠른 부팅 속도와 대용량 저장 공간을 원한다면, M.2 M-Key 슬롯(J1)에 **NVMe SSD**를 장착하는 것이 권장된다.2 SDK Manager는 플래싱 과정에서 OS를 설치할 대상을 eMMC와 NVMe 중에서 선택할 수 있는 옵션을 제공한다.


Jetson Orin 개발자 키트는 공장 출하 시 JetPack 5.x 버전과 호환되는 펌웨어(UEFI 부트로더 등)가 설치되어 있다. 따라서 Ubuntu 22.04 기반의 JetPack 6.x를 설치하기 위해서는 이 펌웨어를 먼저 업데이트해야 한다.4

일반적으로 NVIDIA SDK Manager를 사용하여 플래싱을 진행하면, 이 과정에서 필요한 펌웨어 업데이트가 자동으로 감지되고 처리된다. 따라서 대부분의 사용자는 이 과정을 별도로 신경 쓸 필요가 없다. 하지만, 수동으로 설치를 진행하거나 플래싱 후 부팅 과정에서 예기치 않은 문제가 발생하는 경우, 펌웨어와 OS 버전 간의 비호환성이 근본적인 원인일 수 있다는 점을 인지하는 것이 중요하다. 부팅 실패 문제를 해결할 때, 펌웨어 호환성은 가장 먼저 고려해야 할 진단 항목 중 하나다.


NVIDIA SDK Manager는 Jetson 개발 환경을 설정하는 가장 안정적이고 권장되는 방법이다. 이 섹션에서는 GUI를 통해 Jetson AGX Orin에 Jetson Linux(Ubuntu 22.04 기반)를 설치하는 전체 과정을 단계별로 상세히 안내한다.


1. **다운로드:** 먼저 NVIDIA 개발자 웹사이트의 SDK Manager 페이지에 접속하여 설치 파일을 다운로드한다.7 호스트 PC의 운영체제에 맞는 버전을 선택해야 한다.

2. **설치:**

   - **Ubuntu 호스트:** 다운로드한 `.deb` 패키지가 있는 디렉토리에서 터미널을 열고 다음 명령어를 실행하여 설치한다.7

     ```Bash
  sudo apt install./sdkmanager_<version>-<build>_amd64.deb
     ```
     
   - **Windows 호스트:** 다운로드한 `.exe` 설치 프로그램을 실행하고 화면의 지시에 따라 설치를 완료한다. 설치 후 시작 메뉴 또는 설치 디렉토리에서 GUI나 CLI 버전을 실행할 수 있다.7

   - **Docker 환경:** 제공되는 Docker 이미지를 다운로드하고 `docker load` 명령어를 사용하여 이미지를 로드한다. 이후 `docker run` 명령어를 통해 SDK Manager CLI를 실행할 수 있다.7

3. **로그인:** SDK Manager를 처음 실행하면 NVIDIA 개발자 계정으로 로그인하라는 창이 나타난다. 웹 브라우저를 통해 인증 과정을 완료해야만 다음 단계로 진행할 수 있다.12


플래싱은 Jetson 장치가 특수한 상태인 '강제 복구 모드(Force Recovery Mode, RCM)'에 있을 때만 가능하다. 이 모드는 장치가 호스트 PC로부터 새로운 펌웨어와 OS 이미지를 수신할 수 있도록 한다.

**절차:**

1. **물리적 연결:** 호스트 PC와 Jetson AGX Orin 개발자 키트를 USB Type-A to Type-C 케이블로 연결한다. 이때, Jetson 보드의 **40핀 헤더 옆에 있는 USB-C 포트(UFP, 포트 번호 10)**에 케이블을 연결해야 한다.17

2. **전원 차단:** Jetson 보드의 전원이 완전히 꺼져 있는지 확인한다. 전원 케이블이 연결되어 있다면 분리한다.

3. 버튼 조작 (전원 꺼진 상태 기준):

   a.  가운데에 위치한 강제 복구(Force Recovery) 버튼(버튼 번호 2)을 누른 상태를 유지한다.17

   b.  강제 복구 버튼을 누른 상태에서 전원 공급 장치(DC 잭 또는 USB-C PD)를 연결하여 보드에 전원을 공급한다.17

   c.  전원이 인가된 후 몇 초 뒤에, 눌렀던 강제 복구 버튼에서 손을 뗀다.

4. **검증:** 호스트 PC에서 터미널을 열고 `lsusb` 명령어를 실행한다. 출력된 USB 장치 목록 중에 `ID 0955:7023 NVidia Corp.` 또는 유사한 항목이 보이면 장치가 성공적으로 복구 모드에 진입한 것이다.20 이 항목이 보이지 않는다면, 위의 절차를 다시 시도해야 한다. 이 검증 단계는 다음 단계로 넘어가기 전 필수적으로 거쳐야 하는 체크포인트다.



1. 호스트 PC에서 `sdkmanager` 명령어를 실행하거나 애플리케이션 아이콘을 클릭하여 SDK Manager를 시작한다.
2. **Product Category** 패널에서 **Jetson**을 선택한다.12
3. **System Configuration** 패널에서, 먼저 **Host Machine**의 체크박스를 해제한다. 이는 호스트 PC에 SDK를 설치하지 않고 오직 대상 장치(Target)에만 OS를 플래싱하기 위함이다.18
4. **Target Hardware** 드롭다운 메뉴에서 **Jetson AGX Orin module** 또는 이와 유사한 항목을 선택한다. 앞선 3.2 단계에서 장치가 복구 모드로 올바르게 연결되었다면, SDK Manager가 자동으로 장치를 감지하고 해당 항목을 추천해준다.12
5. **SDK Version** 패널에서 Ubuntu 22.04를 지원하는 최신 **JetPack 6.x** 버전을 선택한다.12
6. **CONTINUE** 버튼을 클릭하여 다음 단계로 진행한다.


1. 이 단계에서는 설치될 소프트웨어 컴포넌트 목록이 표시된다. 설치 과정을 명확히 분리하기 위해, 먼저 OS만 설치한다. **Jetson OS** 항목만 선택된 상태로 두고, **Jetson SDK Components** 항목의 체크박스는 해제한다. SDK 컴포넌트들은 OS 설치가 완료된 후 Jetson 장치에서 직접 설치하는 것이 더 효율적이다.12
2. 화면 하단에서 다운로드될 파일이 저장될 **Download folder**와 플래싱 이미지가 생성될 **Target HW image folder**의 경로를 확인한다. 필요하다면 이 경로를 변경할 수 있다.12
3. **I accept the terms and conditions of the license agreements** 확인란을 체크하여 라이선스에 동의한다.12
4. **CONTINUE** 버튼을 클릭한다. 관리자 권한이 필요하므로 호스트 PC의 `sudo` 비밀번호를 입력하라는 창이 나타날 수 있다.


1. SDK Manager가 NVIDIA 서버로부터 필요한 파일(BSP, RootFS 등)을 다운로드하고 압축을 해제한 뒤, 플래싱에 사용될 OS 이미지를 생성한다. 이 과정은 인터넷 속도와 호스트 PC 성능에 따라 시간이 소요될 수 있다.12
2. 이미지 생성이 완료되면, 플래싱 설정을 위한 팝업창이 나타난다. 여기서 두 가지 중요한 옵션을 선택해야 한다.12
   - **OEM Configuration:**
     - **Runtime:** 이 옵션을 선택하면, 플래싱 후 Jetson이 처음 부팅될 때 사용자 이름, 비밀번호, 언어, 네트워크 등을 설정하는 초기 설정 화면이 나타난다. 대부분의 사용자에게 권장되는 방식이다.
     - **Pre-Config:** 플래싱을 시작하기 전에 호스트 PC의 SDK Manager 창에서 미리 Jetson의 사용자 이름과 비밀번호를 설정한다. 헤드리스(headless) 환경이나 자동화된 설정에 유용하다.
   - **Storage Device:** OS를 설치할 저장 매체를 선택한다. 내장 eMMC에 설치하려면 **EMMC**를, 별도로 장착한 NVMe SSD에 설치하려면 **NVMe**를 선택한다.2
3. 설정을 마친 후 **Flash** 버튼을 클릭하면 플래싱 프로세스가 시작된다. 진행 상황은 터미널 로그를 통해 상세히 확인할 수 있다.
4. 플래싱이 성공적으로 완료되면, SDK Manager에 완료 메시지가 표시되고 Jetson AGX Orin은 자동으로 재부팅되어 새로 설치된 OS로 부팅을 시작한다.18


명령줄 인터페이스(CLI)를 사용한 플래싱은 GUI 환경 없이 설치를 진행하거나, 설치 과정을 스크립트로 자동화해야 하는 고급 사용자를 위한 강력한 방법이다. 이 절차는 SDK Manager를 통해 다운로드된 `Linux_for_Tegra` 패키지 내의 `flash.sh` 스크립트를 사용한다.


1. **플래싱 스크립트 위치 확인:** CLI 플래싱을 위해서는 먼저 SDK Manager가 다운로드하고 압축을 해제한 `Linux_for_Tegra` 디렉토리로 이동해야 한다. 이 디렉토리의 기본 경로는 일반적으로 `/home/<user>/nvidia/nvidia_sdk/JetPack_.../Linux_for_Tegra` 이다.21

   ```Bash
   cd ~/nvidia/nvidia_sdk/JetPack_<version>_Linux_JETSON_AGX_ORIN_TARGETS/Linux_for_Tegra/
   ```
   
2. **강제 복구 모드 진입 및 확인:** GUI 방식과 마찬가지로, Jetson AGX Orin을 강제 복구 모드로 설정해야 한다. 본 문서의 **III-3.2** 섹션에 기술된 절차를 정확히 따른다. 이후, 호스트 PC 터미널에서 `lsusb` 명령어를 실행하여 `NVidia Corp.` 장치가 정상적으로 인식되었는지 반드시 확인한다.21


`flash.sh` 스크립트의 기본 구문은 다음과 같다.24

```Bash
sudo./flash.sh [options] <board> <rootdev>
```

- **`[options]`:** 추가적인 동작을 지정하는 옵션. 예를 들어, `-r`은 기존 시스템 이미지를 재사용하여 플래싱 시간을 단축한다.
- **`<board>`:** 플래싱할 대상 보드의 구성(configuration)을 지정하는 문자열이다. Jetson AGX Orin 개발자 키트의 경우, 이 값은 `jetson-agx-orin-devkit`이다.23
- **`<rootdev>`:** 루트 파일 시스템(rootfs)이 설치될 대상 저장 장치를 지정하는 매우 중요한 파라미터다. 이 값을 잘못 지정하면 플래싱이 실패하거나 의도하지 않은 장치에 OS가 설치될 수 있다.24

CLI 방식의 성공은 정확한 `<board>`와 `<rootdev>` 파라미터를 아는 것에 달려있다. 아래 표는 가장 일반적으로 사용되는 저장 매체에 대한 정확한 파라미터 값을 제공하여 사용자 실수를 방지한다.


| 대상 저장 매체 | `<rootdev>` 파라미터 | 전체 실행 명령어 예시                             |
| -------------- | -------------------- | ------------------------------------------------- |
| 내장 eMMC      | `mmcblk0p1`          | `sudo./flash.sh jetson-agx-orin-devkit mmcblk0p1` |
| NVMe SSD       | `nvme0n1p1`          | `sudo./flash.sh jetson-agx-orin-devkit nvme0n1p1` |

*표 2: `flash.sh` 주요 저장 매체별 명령어* 21

위의 명령어를 실행하면, 스크립트가 시스템 이미지를 생성하고 지정된 저장 장치로 전송하는 과정을 시작한다. 이 과정은 수십 분이 소요될 수 있으며, 완료되면 Jetson 보드가 자동으로 재부팅된다.


헤드리스(headless) 환경에서 Jetson을 설정하거나, 첫 부팅 시 OEM 구성 단계를 생략하고 싶을 경우, 플래싱 전에 기본 사용자 계정을 미리 생성할 수 있다. `Linux_for_Tegra` 디렉토리 내의 `tools` 하위 디렉토리에 있는 `l4t_create_default_user.sh` 스크립트를 사용한다.21

```Bash
cd tools/
sudo./l4t_create_default_user.sh -u <username> -p <password> -n <hostname>
cd..
```

- `<username>`: 생성할 사용자 이름
- `<password>`: 해당 사용자의 비밀번호
- `<hostname>`: 장치의 호스트 이름

이 스크립트를 실행한 후 `flash.sh`를 실행하면, 새로 플래싱된 시스템은 지정된 사용자 계정과 함께 부팅되어 별도의 초기 설정 없이 바로 로그인이 가능하다.


호스트 PC를 통해 Jetson Linux 기본 OS의 플래싱을 완료하는 것은 전체 설치 과정의 첫 번째 단계일 뿐이다. Jetson 플랫폼의 진정한 성능을 활용하기 위해서는 CUDA, cuDNN, TensorRT와 같은 핵심 AI 및 컴퓨팅 라이브러리를 포함하는 JetPack 컴포넌트들을 설치해야 한다. 이 두 번째 단계는 새로 설치된 OS가 부팅된 Jetson 장치 자체에서 수행된다.


플래싱 후 Jetson AGX Orin을 처음 부팅하면, OEM(Original Equipment Manufacturer) 구성 프로세스가 시작된다. 만약 플래싱 시 'Runtime' OEM 구성을 선택했다면, 이 단계에서 모니터와 키보드를 통해 다음 정보를 설정해야 한다.6

- 사용자 계정 이름 및 비밀번호
- 시스템 언어 및 키보드 레이아웃
- 시간대
- 네트워크 연결 (Wi-Fi 또는 이더넷)

이 설정이 완료되면 Ubuntu 데스크톱 환경으로 진입하게 된다.


기본 OS에는 NVIDIA의 가속 라이브러리들이 포함되어 있지 않다. 이들은 Jetson 장치가 인터넷에 연결된 상태에서 Debian 패키지 관리 도구인 `apt`를 통해 설치해야 한다.

1. **터미널 실행:** Jetson의 Ubuntu 데스크톱에서 터미널을 연다 (`Ctrl` + `Alt` + `T`).

2. **패키지 목록 업데이트:** 시스템의 패키지 목록을 최신 상태로 업데이트한다.

   ```Bash
   sudo apt update
   ```
   
3. **JetPack 메타 패키지 설치:** 다음 명령어를 실행하여 모든 JetPack 컴포넌트를 한 번에 설치한다. 이 명령어는 현재 설치된 Jetson Linux 버전에 맞는 모든 관련 라이브러리(CUDA, cuDNN, TensorRT, VPI, OpenCV 등)를 자동으로 다운로드하고 설치한다.5

   ```Bash
sudo apt install nvidia-jetpack
   ```
   
   `nvidia-jetpack` 메타 패키지는 두 개의 하위 메타 패키지로 구성된다: `nvidia-jetpack-runtime`은 애플리케이션 실행에 필요한 런타임 라이브러리만을 포함하며, `nvidia-jetpack-dev`는 샘플 코드, 문서, 개발 헤더 파일 등을 포함한다. `nvidia-jetpack`을 설치하면 이 두 가지가 모두 설치된다.25 저장 공간이 제한적인 경우, 필요에 따라 개별 컴포넌트 패키지를 선택하여 설치할 수도 있다.

4. **설치 완료 및 재부팅:** 설치가 완료되면, 변경 사항을 시스템에 완전히 적용하기 위해 재부팅하는 것이 좋다.

   ```Bash
   sudo reboot
   ```


모든 설치 과정이 올바르게 완료되었는지 확인하기 위해, 터미널에서 몇 가지 명령어를 실행하여 주요 소프트웨어 구성 요소의 버전을 검증해야 한다. 이는 시스템의 현재 상태를 정확히 파악하고 향후 개발 및 디버깅의 기준점으로 삼기 위해 필수적인 과정이다.

| 검증 대상                 | 실행 명령어                          | 예상 출력 예시 (버전에 따라 다름)                 |
| ------------------------- | ------------------------------------ | ------------------------------------------------- |
| JetPack 버전              | `sudo apt-cache show nvidia-jetpack` | `Package: nvidia-jetpack` `Version: 6.0-b52`      |
| Jetson Linux (L4T) 릴리스 | `cat /etc/nv_tegra_release`          | `# R36 (release), REVISION: 3.0,...`              |
| Ubuntu 버전               | `lsb_release -a`                     | `Description: Ubuntu 22.04.3 LTS`                 |
| CUDA 버전                 | `nvcc --version`                     | `Cuda compilation tools, release 12.2, V12.2.140` |

*표 3: 시스템 검증을 위한 핵심 명령어* 26

이 명령어들의 출력을 통해 JetPack 6.x, L4T R36.x, Ubuntu 22.04, 그리고 해당 JetPack 버전에 맞는 CUDA Toolkit이 모두 정상적으로 설치되었음을 확인할 수 있다.


Jetson AGX Orin은 다양한 전력 모드를 제공하여 성능과 전력 소비 간의 균형을 조절할 수 있다. AI 추론이나 고성능 컴퓨팅 작업을 수행하기 전, 시스템 성능을 최대로 끌어올리는 것이 좋다.

- **전력 모드 변경 (`nvpmodel`):** `nvpmodel` 유틸리티를 사용하여 사전 정의된 전력 프로파일을 선택할 수 있다. 최대 성능을 위해서는 **MAXN** 모드를 사용한다.6

  ```Bash
  # 현재 전력 모드 확인
  sudo nvpmodel -q
  
  # MAXN 모드로 설정 (최대 성능)
  sudo nvpmodel -m 0
  ```
  
- **클럭 속도 고정 (`jetson_clocks`):** `jetson_clocks` 스크립트는 CPU, GPU, 메모리 컨트롤러 등의 클럭 속도를 최대로 고정하여 동적 클럭 조절로 인한 성능 저하를 방지한다. 이는 일관된 최대 성능이 요구되는 벤치마킹이나 실시간 애플리케이션에 유용하다.6

  ```Bash
  sudo jetson_clocks
  ```

이 두 가지 최적화 단계를 거치면 Jetson AGX Orin의 하드웨어 잠재력을 최대한 활용할 준비가 완료된다.


Jetson 플랫폼에 OS를 설치하는 과정은 여러 단계로 구성되어 있어 다양한 문제가 발생할 수 있다. 이 섹션에서는 사용자들이 가장 흔하게 겪는 문제들의 원인을 분석하고, 검증된 해결 방안을 제시한다.


- **문제점:** SDK Manager를 사용한 플래싱이 특정 단계(예: 99%)에서 멈추거나, 알 수 없는 오류 메시지와 함께 실패한다.
- **원인 분석:** 가장 빈번하게 보고되는 원인은 호스트 PC의 운영체제 환경과의 비호환성이다. 특히, 공식적으로 지원됨에도 불구하고 새로 설치된(fresh install) Ubuntu 20.04 환경에서 문제가 자주 발생한다는 다수의 커뮤니티 보고가 있다.14 이는 OS 자체의 문제라기보다는 기본 설정값의 차이에서 비롯될 가능성이 높다. 또한, 가상 머신(VM) 환경은 USB 장치를 호스트와 게스트 OS 간에 전달하는(pass-through) 과정이 불안정하여 플래싱 실패의 주요 원인이 되기도 한다.32
- **해결 방안:**
  1. **네이티브 Ubuntu 22.04 호스트 사용:** 가장 확실하고 안정적인 해결책은 Jetson에 설치할 OS와 동일한 버전인 **네이티브 Ubuntu 22.04가 설치된 호스트 PC를 사용하는 것**이다. 이는 잠재적인 라이브러리 및 커널 버전 충돌을 최소화한다.7
  2. **가상 머신 사용 지양:** 가능한 한 가상 머신 대신 물리적인 PC에 설치된 Ubuntu 환경에서 플래싱을 시도한다.
  3. **USB 케이블 및 포트 교체:** 데이터 전송 기능이 불확실하거나 품질이 낮은 USB 케이블은 플래싱 실패의 숨은 원인이 될 수 있다. 데이터 전송이 검증된 고품질의 다른 USB 케이블로 교체하고, 호스트 PC의 다른 USB 포트(가급적이면 메인보드에 직접 연결된 포트)에 연결하여 테스트한다.15


- **문제점:** 플래싱 도중 "might be timeout in USB write"와 유사한 오류 메시지가 나타나며 프로세스가 중단된다.

- **원인 분석:** 이 문제는 특히 Ubuntu 20.04 호스트 PC에서 자주 발생하며, OS의 기본 전원 관리 정책 중 하나인 **USB 자동 절전(autosuspend) 기능**이 원인인 경우가 많다.16 플래싱 과정은 수십 분이 소요될 수 있는데, 이 시간 동안 호스트 PC가 USB 포트를 절전 모드로 전환하면서 Jetson과의 연결이 끊어지는 것이다.

- **해결 방안:** 호스트 PC에서 USB 자동 절전 기능을 영구적으로 또는 일시적으로 비활성화한다. 다음 명령어를 호스트 PC의 터미널에서 실행하면 현재 세션 동안 이 기능이 비활성화된다.16

  ```Bash
  sudo -s
  echo -1 > /sys/module/usbcore/parameters/autosuspend
  exit
  ```
  
  이 명령어를 실행한 후, Jetson에서 USB 케이블을 분리했다가 다시 연결하고, 장치를 다시 강제 복구 모드로 진입시킨 뒤 플래싱을 재시도한다.


- **문제점:** `lsusb` 명령어를 실행해도 Jetson 장치가 목록에 나타나지 않으며, SDK Manager가 장치를 감지하지 못한다.

- **원인 분석:** 이 문제는 대부분 물리적인 조작 실수에서 비롯된다.

  1. 잘못된 버튼 조합 또는 타이밍으로 복구 모드 진입에 실패한 경우.
  2. 플래싱용이 아닌 다른 USB-C 포트에 케이블을 연결한 경우.34
  3. 결함이 있거나 데이터 전송을 지원하지 않는 USB 케이블을 사용한 경우.33
  4. 드물게는 Jetson 보드 자체의 하드웨어 결함일 수 있다.35

- **해결 방안:**

  1. **절차 재확인:** 본 문서의 **III-3.2** 섹션에 명시된 강제 복구 모드 진입 절차를 천천히, 그리고 정확하게 다시 따른다. 특히 **40핀 헤더 옆의 올바른 USB-C 포트**를 사용하고 있는지 반드시 재확인한다.

  2. **케이블 교체:** 다른 USB 케이블로 교체하여 문제가 해결되는지 확인한다.33

  3. **다른 호스트 PC에서 테스트:** 가능하다면 다른 호스트 PC에 연결하여 문제가 호스트 측의 USB 드라이버나 포트 문제인지, 아니면 Jetson 장치나 케이블의 문제인지 분리하여 진단한다.36

  4. **명령줄로 재부팅 시도:** 만약 Jetson이 정상 부팅된 상태라면, 터미널에서 다음 명령어를 통해 강제 복구 모드로 재부팅을 시도해 볼 수 있다.37

     ```Bash
     sudo reboot --force forced-recovery
     ```


- **문제점:** SDK Manager가 "Internet connection check failure" 오류를 표시하거나, NVIDIA 서버에 로그인하지 못하거나, 특정 패키지 다운로드에 실패한다.
- **원인 분석:** 이 문제는 대부분 호스트 PC의 네트워크 환경 설정과 관련이 있다. 기업이나 기관의 네트워크에서 사용하는 방화벽이나 프록시 서버가 SDK Manager의 연결을 차단하는 경우가 많다.38
- **해결 방안:**
  1. **방화벽 및 프록시 설정 확인:**
     - 호스트 PC의 방화벽(예: `ufw`)이 SDK Manager의 외부 연결을 막고 있는지 확인하고, 필요한 경우 일시적으로 비활성화한 후 재시도한다.36
     - 프록시 서버 환경이라면, SDK Manager의 설정 메뉴에서 프록시 서버 주소와 포트를 정확하게 입력해야 한다.40
  2. **기본 네트워크 연결 점검:** 터미널에서 `ping nvidia.com`과 같은 명령어를 실행하여 호스트 PC가 외부 인터넷에 정상적으로 연결되어 있는지 기본적인 상태를 점검한다.40
  3. **오프라인 설치 고려:** 네트워크 제약이 심한 환경에서는 SDK Manager의 오프라인 설치 기능을 활용할 수 있다. 인터넷이 가능한 다른 PC에서 모든 패키지를 미리 다운로드한 후, 해당 파일을 대상 호스트 PC로 옮겨 설치를 진행하는 방식이다.42


1. Powering AGV & AMR Robotics with Rugged Industrial GPU Computing - Premio Inc, 8월 22, 2025에 액세스, https://premioinc.com/blogs/blog/powering-agv-amr-robotics-with-rugged-industrial-gpu-computing
2. Jetson AGX Orin Developer Kit User Guide - Hardware Layout, 8월 22, 2025에 액세스, https://developer.nvidia.com/embedded/learn/jetson-agx-orin-devkit-user-guide/developer_kit_layout.html
3. JetPack SDK - NVIDIA Developer, 8월 22, 2025에 액세스, https://developer.nvidia.com/embedded/jetpack-sdk-513
4. JetPack SDK | NVIDIA Developer, 8월 22, 2025에 액세스, https://developer.nvidia.com/embedded/jetpack
5. JetPack SDK 6.0 - NVIDIA Developer, 8월 22, 2025에 액세스, https://developer.nvidia.com/embedded/jetpack-sdk-60
6. How to upgrade Jetpack 5.X to 6.X on NVIDIA Jetson Orin Nano Super - Collabnix, 8월 22, 2025에 액세스, https://collabnix.com/how-to-upgrade-jetpack-5-x-to-6-x-on-nvidia-jetson-orin-nano-super/
7. SDK Manager | NVIDIA Developer, 8월 22, 2025에 액세스, https://developer.nvidia.com/sdk-manager
8. NVIDIA Jetson Setup Using SDK Manager - RS Online, 8월 22, 2025에 액세스, https://www.rs-online.com/designspark/nvidia-jetson-setup-using-sdk-manager
9. NVIDIA SDK Manager Tutorial: Installing Jetson Software Explained - YouTube, 8월 22, 2025에 액세스, https://www.youtube.com/watch?v=Ucg5Zqm9ZMk
10. Jetpack 6.2: Command Line Install for Orin Nano and AGX - YouTube, 8월 22, 2025에 액세스, https://www.youtube.com/watch?v=WQg3PEUBiD8
11. Jetson Orin Nano Tutorial: SSD Install, Boot, and JetPack Setup - JetsonHacks, 8월 22, 2025에 액세스, https://jetsonhacks.com/2023/05/30/jetson-orin-nano-tutorial-ssd-install-boot-and-jetpack-setup/
12. JetPack 6 Developer Guide - Flashing With SDK Manager, 8월 22, 2025에 액세스, https://developer.ridgerun.com/wiki/index.php/JetPack_6_Migration_and_Developer_Guide/Installing_JetPack_6/Flashing_with_SDK_Manager
13. System Requirements — SDK Manager, 8월 22, 2025에 액세스, https://docs.nvidia.com/sdk-manager/system-requirements/index.html
14. Use of SDK manager to setup Jetson AGX Orin - NVIDIA Developer Forums, 8월 22, 2025에 액세스, https://forums.developer.nvidia.com/t/use-of-sdk-manager-to-setup-jetson-agx-orin/240879
15. Jetson AGX Orin Flashing Issues - NVIDIA Developer Forums, 8월 22, 2025에 액세스, https://forums.developer.nvidia.com/t/jetson-agx-orin-flashing-issues/243627
16. Error trying to flash Jetson AGX Orin with SDK Manager - Jetson ..., 8월 22, 2025에 액세스, https://forums.developer.nvidia.com/t/error-trying-to-flash-jetson-agx-orin-with-sdk-manager/312289
17. Jetson AGX Orin Developer Kit User Guide - How-to | NVIDIA ..., 8월 22, 2025에 액세스, https://developer.nvidia.com/embedded/learn/jetson-agx-orin-devkit-user-guide/howto.html
18. Jetson AGX Orin Developer Kit User Guide - Two Ways to Set Up Software, 8월 22, 2025에 액세스, https://developer.nvidia.com/embedded/learn/jetson-agx-orin-devkit-user-guide/two_ways_to_set_up_software.html
19. Initial Setup (SDK Manager method) - NVIDIA Jetson AI Lab, 8월 22, 2025에 액세스, https://www.jetson-ai-lab.com/initial_setup_jon_sdkm.html
20. NVIDIA Jetson Orin AGX - JetPack 5.0.2 - Getting Started - Wizard ..., 8월 22, 2025에 액세스, https://developer.ridgerun.com/wiki/index.php/NVIDIA_Jetson_Orin/JetPack_5.0.2/Getting_Started/Wizard_Flashing
21. NVIDIA Jetson Orin AGX - JetPack 5.0.2 - Flashing Board, 8월 22, 2025에 액세스, https://developer.ridgerun.com/wiki/index.php/NVIDIA_Jetson_Orin/JetPack_5.0.2/Flashing_Board
22. Install Jetson Software with SDK Manager, 8월 22, 2025에 액세스, https://docs.nvidia.com/sdk-manager/install-with-sdkm-jetson/index.html
23. NVIDIA Jetson Orin AGX - Flashing commands for emulation - RidgeRun Developer Wiki, 8월 22, 2025에 액세스, https://developer.ridgerun.com/wiki/index.php/NVIDIA_Jetson_Orin/Flashing_commands_for_emulation
24. Flashing Support — Jetson Linux Developer Guide documentation, 8월 22, 2025에 액세스, https://docs.nvidia.com/jetson/archives/r35.4.1/DeveloperGuide/text/SD/FlashingSupport.html
25. How to Install and Configure JetPack SDK — JetPack, 8월 22, 2025에 액세스, https://docs.nvidia.com/jetson/jetpack/install-setup/index.html
26. How to verify the installed Jetpack version? - Aetina Corporation, 8월 22, 2025에 액세스, https://www.aetina.com/support-faq-detail.php?i=63
27. How to check the JetPack Version - Jetson TX2 - NVIDIA Developer Forums, 8월 22, 2025에 액세스, https://forums.developer.nvidia.com/t/how-to-check-the-jetpack-version/69549
28. different ways to check for nvidia jetpack versions - Github-Gist, 8월 22, 2025에 액세스, https://gist.github.com/jadwigo/86b905ca2573dc7b9a685652b82ef590
29. How to find Jetpack version of NVIDIA Jetson Nano - Collabnix, 8월 22, 2025에 액세스, https://collabnix.com/how-to-find-jetpack-version-of-nvidia-jetson-nano/
30. How do I check the version of Ubuntu I am running? [duplicate], 8월 22, 2025에 액세스, https://askubuntu.com/questions/686239/how-do-i-check-the-version-of-ubuntu-i-am-running
31. Updating Jetson AGX Orin with Jepack 5.1.2 to CUDA 11.8 and PyTorch installation from source, 8월 22, 2025에 액세스, https://discuss.pytorch.org/t/updating-jetson-agx-orin-with-jepack-5-1-2-to-cuda-11-8-and-pytorch-installation-from-source/217761
32. Cannot install sdk manager or jetpack. : r/JetsonNano - Reddit, 8월 22, 2025에 액세스, https://www.reddit.com/r/JetsonNano/comments/10qohrl/cannot_install_sdk_manager_or_jetpack/
33. Jetson AGX Orin won't boot, host doesn't recognize it : r/JetsonNano - Reddit, 8월 22, 2025에 액세스, https://www.reddit.com/r/JetsonNano/comments/1idimok/jetson_agx_orin_wont_boot_host_doesnt_recognize_it/
34. Unable to enter Recovery Mode on Jetson Orin NX - NVIDIA Developer Forums, 8월 22, 2025에 액세스, https://forums.developer.nvidia.com/t/unable-to-enter-recovery-mode-on-jetson-orin-nx/322718
35. Jetson Nano Orin 2024 Dead? : r/JetsonNano - Reddit, 8월 22, 2025에 액세스, https://www.reddit.com/r/JetsonNano/comments/1hzmi5m/jetson_nano_orin_2024_dead/
36. Jetson Orin flashing error : r/JetsonNano - Reddit, 8월 22, 2025에 액세스, https://www.reddit.com/r/JetsonNano/comments/197b9sy/jetson_orin_flashing_error/
37. Jetson Orin Nano Dev. Kit does not boot into recovery mode - NVIDIA Developer Forums, 8월 22, 2025에 액세스, https://forums.developer.nvidia.com/t/jetson-orin-nano-dev-kit-does-not-boot-into-recovery-mode/305948
38. Unable to install new sdk's using android sdkmanager in visual studio latest version 2022, 8월 22, 2025에 액세스, https://learn.microsoft.com/en-us/answers/questions/2046320/unable-to-install-new-sdks-using-android-sdkmanage
39. Android SDK installation failed - Connection reset. Nothing is installed - Stack Overflow, 8월 22, 2025에 액세스, https://stackoverflow.com/questions/10320793/android-sdk-installation-failed-connection-reset-nothing-is-installed
40. SDK Manager "Internet connection check failure" - NVIDIA Developer Forums, 8월 22, 2025에 액세스, https://forums.developer.nvidia.com/t/sdk-manager-internet-connection-check-failure/191718
41. sdkmanager | Android Studio, 8월 22, 2025에 액세스, https://developer.android.com/tools/sdkmanager
42. NVIDIA SDK Manager Documentation, 8월 22, 2025에 액세스, https://docs.nvidia.com/sdk-manager/

