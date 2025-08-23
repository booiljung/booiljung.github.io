# PX4-Autopilot v1.15의 ubuntu.sh 개발 환경 설정 스크립트



PX4-Autopilot 생태계에서 `ubuntu.sh` 스크립트는 단순한 설치 도구를 넘어, 일관되고 재현 가능하며 지원되는 개발 환경을 보장하는 핵심적인 기반이다. 드론 펌웨어 개발의 복잡성을 고려하면, 개발자마다 다른 환경 설정은 예측 불가능한 빌드 오류, 미묘한 런타임 버그, 그리고 디버깅의 어려움으로 이어질 수 있다. `ubuntu.sh`는 이러한 위험을 완화하기 위해 설계된 자동화된 솔루션이다. 이 스크립트는 PX4 개발팀이 공식적으로 지원하는 "골든 경로(golden path)"를 제공하며, 개발자가 수동으로 수십 개의 종속성을 설치하는 과정에서 발생할 수 있는 잠재적 오류를 근본적으로 차단한다.1 공식 문서에서 "깨끗한(clean) 우분투 LTS 설치" 환경에서 실행할 것을 반복적으로 권장하는 이유는 바로 이 스크립트가 시스템 전반에 걸쳐 특정 버전의 툴체인, 라이브러리, 시뮬레이터 등을 설치하여 예측 가능한 개발 기반을 구축하기 때문이다.3 따라서 이 스크립트의 분석은 단순히 명령어의 나열이 아니라, 거대한 오픈소스 프로젝트가 어떻게 안정성과 확장성을 유지하는지에 대한 근본적인 이해를 제공한다.


일반적인 PX4 개발자는 다음과 같은 표준화된 워크플로우를 따르게 되며, `ubuntu.sh`는 이 과정의 필수적인 초기 단계에 위치한다.

1. **저장소 복제(Clone):** 먼저, 개발자는 GitHub에서 공식 `PX4-Autopilot` 저장소를 자신의 로컬 컴퓨터로 복제한다. 이때 `--recursive` 플래그를 사용하여 NuttX, MAVLink 등과 같은 모든 하위 모듈(submodule)을 함께 다운로드해야 한다.1
2. **특정 릴리스 체크아웃(Checkout):** 안정적인 개발 및 테스트를 위해, 최신 `main` 브랜치보다는 특정 릴리스 태그(예: `v1.15.0`)로 전환하는 것이 일반적이다. 이는 특정 시점의 검증된 코드베이스를 기반으로 작업을 진행하기 위함이다.6
3. **개발 환경 구축:** 이 단계에서 `ubuntu.sh` 스크립트가 실행된다. 스크립트는 현재 운영체제 버전을 감지하여 필요한 모든 컴파일러, 라이브러리, 파이썬 패키지, 시뮬레이션 도구들을 자동으로 설치한다.1
4. **시뮬레이션 빌드 및 검증:** 환경 설정이 완료되면, 개발자는 먼저 시뮬레이션(SITL, Software-in-the-Loop) 타겟으로 펌웨어를 빌드한다. 예를 들어 `make px4_sitl_default gazebo-classic` 명령어를 실행하여 Gazebo 시뮬레이터에서 가상 드론을 구동해 볼 수 있다. 이 과정을 통해 실제 하드웨어 없이 코드 변경 사항을 안전하고 신속하게 테스트하고, 개발 환경 자체가 올바르게 구성되었는지 검증할 수 있다.4
5. **하드웨어 펌웨어 빌드:** 시뮬레이션 검증이 완료되면, 실제 비행 컨트롤러(예: Pixhawk 5X)를 위한 펌웨어를 빌드한다. `make px4_fmu-v5x_default`와 같은 명령어를 사용하면 해당 하드웨어에 맞는 바이너리 파일(`.px4`)이 생성되며, 이를 QGroundControl과 같은 GCS(Ground Control Station)를 통해 업로드할 수 있다.4


`ubuntu.sh` 스크립트의 내용을 정확히 이해하기 위해서는 PX4 v1.15 릴리스의 주요 목표와 전략적 방향을 파악하는 것이 중요하다. 스크립트에 포함된 종속성들은 임의로 선택된 것이 아니라, v1.15의 핵심 기능들을 지원하기 위해 신중하게 구성되었기 때문이다.

- **주요 변경 사항:** v1.15 릴리스는 몇 가지 중요한 기술적 진보를 이루었다. 가장 주목할 만한 것은 uXRCE-DDS 미들웨어를 통한 ROS 2(Robot Operating System 2) 와의 통합 강화이다. 이는 PX4를 ROS 2 기반의 로봇 애플리케이션과 더욱 긴밀하게 연동할 수 있게 만들어, 외부 컴퓨터에서의 제어 및 데이터 모니터링을 훨씬 용이하게 한다. 또한, EKF2(Extended Kalman Filter)의 항법 및 추정 알고리즘이 대폭 개선되었고, 새로운 비행 컨트롤러와 센서에 대한 지원이 확장되었다.11
- **스크립트에 미치는 영향:** 이러한 v1.15의 전략적 목표는 `ubuntu.sh`가 설치하는 패키지 목록에 직접적인 영향을 미친다. 예를 들어, uXRCE-DDS 클라이언트를 빌드하고 실행하는 데 필요한 라이브러리들이 기본 종속성에 포함된다. 또한, 새롭게 지원되는 비행 컨트롤러 보드(예: FMUv6X-RT)를 컴파일하기 위한 특정 버전의 ARM 툴체인이 스크립트를 통해 설치된다. 즉, `ubuntu.sh`는 v1.15 릴리스의 새로운 기능들을 개발자들이 원활하게 사용하고 기여할 수 있도록 하는 기술적인 출발점을 마련하는 역할을 수행한다.

이처럼 `ubuntu.sh`는 단순한 편의성 스크립트를 넘어, PX4 프로젝트의 개발 철학을 담고 있는 중요한 요소이다. 전 세계 수많은 개발자들이 동일한 코드베이스 위에서 협업하기 위해서는, 모두가 공유하는 안정적인 개발 환경의 기준선이 필수적이다. 이 스크립트는 바로 그 기준선을 설정하고 강제하는 정책 도구로서 기능하며, 이를 통해 프로젝트의 전반적인 복잡성을 관리하고 개발 생산성을 극대화한다.


PX4 v1.15의 `ubuntu.sh` 스크립트는 다양한 개발 환경과 요구사항에 대응할 수 있도록 방어적이고 모듈화된 아키텍처로 설계되었다. 이 스크립트는 단순히 명령어를 순차적으로 실행하는 것을 넘어, 실행 환경을 능동적으로 감지하고 사용자의 선택에 따라 설치 과정을 맞춤화하는 지능적인 로직을 포함하고 있다.


스크립트는 표준 Bash 스크립트의 관례를 따르며 시작된다.

- **Shebang 및 오류 처리:** 스크립트의 첫 줄은 `#!/usr/bin/env bash`로 시작하여, 시스템의 `PATH`에서 `bash` 인터프리터를 찾아 스크립트를 실행하도록 지정한다. 이는 다양한 시스템에서 스크립트의 이식성을 보장한다. 바로 뒤이어 `set -e` 명령어가 사용되는데, 이는 스크립트 실행 중 어느 한 명령어라도 0이 아닌 종료 코드(즉, 오류)를 반환할 경우 즉시 실행을 중단시키는 'fail-fast' 정책이다. 이 설정은 일부 종속성 설치 실패가 전체 시스템을 불안정한 상태로 만드는 것을 방지하는 중요한 안전장치이다.
- **인수 파싱 및 사용자 정의 플래그:** 스크립트는 실행 시 전달된 명령줄 인수를 처리하기 위해 `for` 루프를 사용한다. 이를 통해 개발자는 전체 설치 과정 중 특정 부분을 건너뛸 수 있다.13
  - `--no-nuttx`: 이 플래그가 전달되면, ARM 임베디드 툴체인(NuttX 타겟 빌드용)의 설치를 건너뛴다. 이는 시뮬레이션(SITL) 환경에서만 작업하거나, ROS와 같은 상위 레벨 소프트웨어 개발에만 집중하는 개발자, 또는 CI/CD(지속적 통합/지속적 배포) 파이프라인에서 하드웨어 빌드가 필요 없는 경우에 유용하다.1
  - `--no-sim-tools`: 이 플래그는 Gazebo나 jMAVSim과 같은 시뮬레이터 및 관련 종속성의 설치를 건너뛴다. 이는 실제 하드웨어 개발에만 집중하거나, 라즈베리 파이와 같은 리소스가 제한된 컴패니언 컴퓨터에 개발 환경을 설정할 때 매우 유용하다.1


스크립트는 본격적인 설치에 앞서 현재 시스템의 상태를 확인하는 여러 검사를 수행한다.

- **도커(Docker) 환경 탐지:** 스크립트는 `if [ -f /.dockerenv ]; then` 조건문을 통해 파일 시스템의 루트에 `/.dockerenv` 파일이 존재하는지 확인한다. 이 파일은 도커 컨테이너 환경의 특징적인 지표이다. 만약 컨테이너 내부에서 실행 중인 것으로 판단되면, `apt` 저장소 추가나 GPG 키 관리 등에 필요한 최소한의 기본 패키지(`ca-certificates`, `gnupg`, `sudo` 등)를 먼저 설치한다. 이는 깨끗한 도커 이미지에서 스크립트가 원활하게 실행될 수 있도록 보장하는 방어적 로직이다.13

- **`requirements.txt` 파일 존재 여부 확인:** 스크립트는 동일한 디렉토리 내에 `requirements.txt` 파일이 있는지 확인하고, 만약 없다면 오류 메시지를 출력하며 실행을 중단한다.13 이는 

  `ubuntu.sh`가 파이썬 종속성 목록을 담고 있는 `requirements.txt`와 분리될 수 없는 한 쌍임을 명확히 하며, 사용자가 실수로 스크립트만 단독으로 실행하는 것을 방지한다.

- **우분투 버전 및 아키텍처 식별:** 스크립트의 적응형 로직의 핵심은 `lsb_release -rs`와 `uname -m` 명령어에 있다. `lsb_release -rs`는 우분투의 릴리스 번호(예: `18.04`, `20.04`, `22.04`)를 반환하고, `uname -m`은 시스템의 하드웨어 아키텍처(예: `x86_64`, `aarch64`)를 반환한다. 스크립트는 이 정보들을 변수에 저장하여 이후의 설치 과정에서 각 환경에 맞는 올바른 버전의 Gazebo 시뮬레이터나 컴파일러를 설치하는 데 사용한다.13


사전 검사가 완료되면 스크립트는 다음과 같은 논리적 흐름에 따라 주요 종속성들을 설치한다.

1. **일반 종속성 설치:** 빌드 시스템, 컴파일러, 코드 관리 도구 등 모든 개발 환경에 공통적으로 필요한 기본 패키지들을 `apt`를 통해 설치한다. 이 블록은 항상 실행된다.
2. **파이썬 종속성 설치:** `requirements.txt` 파일에 명시된 파이썬 패키지들을 `pip`을 사용하여 설치한다. 이 블록 또한 항상 실행된다.
3. **NuttX 툴체인 설치:** `INSTALL_NUTTX` 플래그가 `true`일 경우에만 실행된다. ARM 크로스 컴파일러와 관련 도구들을 설치한다.
4. **시뮬레이션 도구 설치:** `INSTALL_SIM` 플래그가 `true`일 경우에만 실행된다. 우분투 버전에 맞는 Gazebo 시뮬레이터와 관련 라이브러리들을 설치한다.
5. **후속 설치 단계:** 모든 주요 설치가 끝난 후, 시리얼 포트 접근 권한을 위한 `dialout` 그룹에 사용자 추가, VMware 환경을 위한 그래픽 가속 문제 해결 등 시스템 설정을 마무리하는 작업들을 수행한다.

이러한 모듈화된 구조는 스크립트의 유지보수성을 높이고, 개발자가 자신의 특정 요구에 맞춰 개발 환경을 효율적으로 구성할 수 있도록 지원한다. 예를 들어, ARM 기반의 최신 하드웨어(Apple Silicon 등)에서 리눅스 가상 머신을 사용하는 개발자는 `g++-multilib`와 같은 x86 전용 패키지 설치 시 오류를 겪을 수 있다. 커뮤니티 포럼에서는 이러한 경우 스크립트의 해당 라인을 수동으로 주석 처리하는 해결책이 공유되기도 했다.16 이는 스크립트가 매우 강력하지만 모든 엣지 케이스를 완벽하게 처리하지는 못할 수 있으며, 이러한 경우 스크립트의 구조를 이해하는 것이 문제 해결에 중요함을 보여준다.


`ubuntu.sh` 스크립트가 `apt` 패키지 매니저를 통해 설치하는 시스템 종속성들은 PX4 펌웨어를 빌드하고, 디버깅하며, 테스트하는 데 필요한 견고한 기반을 형성한다. 이 패키지들은 기능에 따라 몇 가지 주요 그룹으로 나눌 수 있다.


이 그룹의 패키지들은 소스 코드를 실행 가능한 바이너리로 변환하는 핵심적인 역할을 담당한다.

- `build-essential`, `g++`, `gcc`: 이 패키지들은 우분투에서 C 및 C++ 코드를 컴파일하는 데 필요한 가장 기본적인 도구 모음이다. `gcc` (GNU Compiler Collection)와 `g++` 컴파일러, 그리고 관련 라이브러리 및 `make` 유틸리티가 포함되어 있다.
- `cmake`, `ninja-build`, `make`: PX4는 복잡한 다중 계층 빌드 시스템을 사용한다. `CMake`는 최상위 빌드 구성 도구로, 플랫폼 독립적인 `CMakeLists.txt` 파일을 읽어 각 시스템에 맞는 빌드 스크립트(예: `Makefile` 또는 `ninja.build` 파일)를 생성한다. `make`는 전통적인 빌드 실행 도구이며, `ninja`는 대규모 프로젝트에서 `make`보다 훨씬 빠른 병렬 빌드 성능을 제공하도록 설계된 최신 도구이다. PX4는 `ninja`를 기본 빌드 도구로 선호하여 개발자의 컴파일 시간을 단축시킨다.10
- `ccache`: 'compiler cache'의 약자로, 컴파일러 캐시 도구이다. `ccache`는 이전 컴파일의 결과를 캐시에 저장해 두었다가, 동일한 소스 코드를 다시 컴파일할 때 캐시된 결과를 즉시 반환한다. 코드의 일부만 수정한 후 다시 빌드할 때, 변경되지 않은 파일들의 컴파일 과정을 건너뛰게 해주므로 전체 빌드 시간을 극적으로 단축시킨다. 이는 개발 과정에서 빈번하게 발생하는 컴파일-테스트 주기의 효율성을 높이는 데 결정적인 역할을 한다.13


이 패키지들은 대규모 분산 개발 환경에서 코드의 일관성과 품질을 유지하기 위해 필수적이다.

- `git`: PX4의 소스 코드는 GitHub에서 관리되므로, 버전 관리 시스템인 `git`은 필수적인 도구이다.6
- `astyle`, `cppcheck`, `shellcheck`: 이들은 코드 린터(linter) 및 정적 분석 도구이다. `astyle`은 C/C++ 코드의 스타일을 자동으로 포맷팅하여 코드의 가독성과 일관성을 유지한다. `cppcheck`는 C/C++ 코드에서 잠재적인 버그나 정의되지 않은 동작과 같은 문제들을 컴파일 없이 정적으로 분석한다. `shellcheck`는 셸 스크립트의 잠재적인 오류를 찾아주는 도구이다. 이러한 도구들이 기본 개발 환경에 포함된다는 것은 PX4 프로젝트가 높은 수준의 소프트웨어 엔지니어링 표준을 지향하고 있음을 보여준다.13


이 패키지들은 다양한 시스템 수준의 기능을 제공하며 다른 여러 도구들의 기반이 된다.

- `libssl-dev`, `libxml2-dev`, `libxml2-utils`: `libssl-dev`는 보안 통신(SSL/TLS)에 필요한 암호화 라이브러리의 개발 헤더 파일이다. `libxml2`는 XML 파일을 파싱하고 처리하는 데 사용되는 강력한 라이브러리로, MAVLink 메시지 정의나 다양한 설정 파일 형식에서 널리 사용된다.13
- `rsync`, `unzip`, `zip`, `file`: 다양한 빌드 및 배포 스크립트에서 사용되는 표준 파일 동기화, 압축 해제, 압축, 파일 유형 식별 유틸리티이다.13
- `gdb`: GNU 디버거(GNU Debugger)는 소프트웨어 개발의 필수 도구이다. 개발자는 `gdb`를 사용하여 SITL 시뮬레이션 환경에서 실행 중인 PX4 프로세스에 연결하여 중단점(breakpoint)을 설정하고, 변수 값을 확인하며, 코드 실행 흐름을 단계별로 추적할 수 있다. 이는 복잡한 알고리즘이나 비행 로직의 문제를 해결하는 데 매우 중요하다.9


PX4 프로젝트에서 파이썬은 선택적인 스크립팅 언어가 아니라, 빌드 및 개발 프로세스의 핵심적인 부분을 담당하는 필수 구성 요소이다. `ubuntu.sh` 스크립트가 파이썬 및 관련 패키지 설치에 상당 부분을 할애하는 이유는 파이썬이 다음과 같은 중요한 역할을 수행하기 때문이다.


- **코드 생성(Code Generation):** PX4의 가장 중요한 특징 중 하나는 uORB와 MAVLink라는 메시징 시스템이다. uORB는 시스템 내부 모듈 간의 통신을, MAVLink는 기체와 지상국(GCS) 또는 컴패니언 컴퓨터 간의 통신을 담당한다. 이 두 시스템의 메시지 형식은 사람이 읽기 쉬운 `.msg` 파일로 정의된다. 빌드 과정에서 파이썬 스크립트(특히 `empy`와 `jinja2` 같은 템플릿 엔진을 사용)가 이 `.msg` 파일들을 파싱하여 C/C++ 헤더 파일을 자동으로 생성한다. 이 자동화된 접근 방식 덕분에 개발자는 메시지 구조를 쉽게 추가하거나 수정할 수 있으며, 통신 코드의 일관성과 정확성이 보장된다.17
- **설정 관리(Configuration Management):** 특정 비행 컨트롤러 보드에 대한 펌웨어 빌드는 `Kconfig` 시스템을 통해 관리된다. 이는 리눅스 커널에서 유래한 설정 시스템으로, 각 보드의 하드웨어 사양, 포함될 드라이버 및 모듈 등을 정의하는 `.px4board` 파일들로 구성된다. 파이썬 기반의 `kconfiglib` 라이브러리는 이 설정 파일들을 처리하여 최종 펌웨어에 어떤 기능이 포함될지를 결정하는 데 사용된다.10
- **빌드 시스템 스크립팅:** CMake 빌드 시스템은 다양한 보조 작업을 수행하기 위해 여러 파이썬 스크립트를 호출한다.
- **로그 분석 및 시뮬레이션:** PX4는 비행 데이터를 `.ulg` 형식의 바이너리 로그 파일로 기록한다. `pyulog`라는 파이썬 라이브러리는 이 로그 파일을 파싱하고 분석하는 표준 도구이다. 또한, 시뮬레이터와 상호작용하거나 비행 데이터를 시각화하는 많은 스크립트들이 파이썬으로 작성된다.19


PX4 v1.15의 빌드 프로세스에 필요한 파이썬 패키지들은 `Tools/setup/requirements.txt` 파일에 명시되어 있다. 이 파일의 내용을 분석하면 PX4 개발 생태계의 내부 동작을 더 깊이 이해할 수 있다.3

**표 4.1: PX4 v1.15의 주요 파이썬 종속성 및 기능**

이 표는 `requirements.txt`에 나열된 수많은 패키지들의 역할을 명확히 설명하여, 단순한 종속성 목록을 소프트웨어 아키텍처의 지도로 변환한다.

| 패키지       | 버전 제약 (v1.15 기준) | PX4 생태계에서의 핵심 기능                                   |
| ------------ | ---------------------- | ------------------------------------------------------------ |
| `empy`       | `>=3.3,<4`             | **템플릿 엔진:** uORB 메시지 헤더 및 MAVLink 다이얼렉트 파일과 같이 템플릿으로부터 소스 코드를 생성하는 데 주로 사용되는 핵심 도구이다. |
| `jinja2`     | `>=2.8`                | **템플릿 엔진:** `empy`와 유사하게, 빌드 과정에서 다양한 설정 파일과 스크립트를 생성하는 데 사용되는 강력한 템플릿 엔진이다. |
| `kconfiglib` | *(버전 명시 없음)*     | **Kconfig 파서:** `Kconfig` 및 `.px4board` 파일을 처리하는 데 필수적이다. 특정 비행 컨트롤러 빌드에 대한 하드웨어 구성 및 포함 모듈을 정의한다.10 |
| `pymavlink`  | *(버전 명시 없음)*     | **MAVLink 프로토콜 라이브러리:** MAVLink 프로토콜의 핵심 파이썬 구현체이다. SITL에서의 통신, 스크립팅, 컴패니언 컴퓨터와의 연동 등에 사용된다. |
| `pyulog`     | `>=0.5.0`              | **ULog 파싱 라이브러리:** PX4의 바이너리 비행 로그 파일(`.ulg`)을 읽고, 파싱하고, 분석하기 위한 표준 도구이다.19 |
| `numpy`      | `>=1.13`               | **수치 연산:** `pyulog`, `matplotlib` 등 거의 모든 과학 및 엔지니어링 파이썬 코드의 기본이 되는 라이브러리로, 데이터 분석 및 처리에 필수적이다. |
| `matplotlib` | `>=3.0`                | **플로팅 라이브러리:** 비행 로그 데이터나 시뮬레이션 결과를 시각화하는 데 사용된다. |
| `pyyaml`     | *(버전 명시 없음)*     | **YAML 파서:** 파라미터 메타데이터 및 기타 설정에 점점 더 많이 사용되는 YAML 형식의 설정 파일을 파싱하는 데 사용된다.18 |
| `sympy`      | `>=1.10.1`             | **기호 수학:** 특히 EKF(확장 칼만 필터)와 같이 기호적 미분이 필요한 제어 알고리즘의 코드 생성과 같은 고급 작업에 사용된다. |


`ubuntu.sh` 스크립트는 `pip`을 사용하여 이러한 패키지들을 설치하며, 시스템의 파이썬 버전에 따라 다른 로직을 적용할 수 있다. 특히 v1.15 시대에는 파이썬 버전과 관련된 알려진 문제점이 있었다. `sympy` 패키지의 특정 버전(`>=1.10.1`)은 파이썬 3.8 이상을 요구했지만, 당시 널리 사용되던 우분투 18.04 LTS의 기본 파이썬 버전은 3.6이었다. 이 불일치로 인해 많은 개발자들이 우분투 18.04에서 환경 설정 시 빌드 오류를 겪었다.20 해결책은 우분투 20.04(기본 파이썬 3.8)로 업그레이드하거나, `requirements.txt`에서 `sympy` 관련 라인을 수동으로 수정하는 것이었다. 이는 특정 소프트웨어 버전에 맞는 개발 환경을 구축하는 것이 얼마나 중요하고 때로는 까다로운지를 보여주는 대표적인 사례이다.


PX4 펌웨어가 다양한 비행 컨트롤러 하드웨어에서 동작할 수 있는 이유는 운영체제(OS)에 대한 추상화 계층을 잘 설계했기 때문이다. 특히 Pixhawk 시리즈와 같은 전통적인 임베디드 타겟을 위해, PX4는 NuttX라는 실시간 운영체제(RTOS)를 기반으로 작동한다. `ubuntu.sh` 스크립트의 `--no-nuttx` 플래그를 사용하지 않는 한, 이 섹션에서 설명하는 NuttX 개발용 툴체인이 설치된다.


NuttX는 리소스가 제한된 마이크로컨트롤러(MCU) 환경을 위해 설계된 경량의 실시간 운영체제이다. NuttX가 PX4의 주요 임베디드 타겟 OS로 채택된 이유는 다음과 같은 특징 때문이다 21:

- **실시간성(Real-Time):** 드론 제어와 같이 밀리초 단위의 정밀한 시간 제어가 요구되는 애플리케이션에서는 태스크의 실행 시간을 보장하는 실시간성이 필수적이다. NuttX는 이러한 요구사항을 만족시키는 RTOS이다.
- **POSIX 호환성:** NuttX는 POSIX(Portable Operating System Interface) 표준과 높은 수준의 호환성을 제공한다. 이는 파일 시스템, 스레딩, 네트워킹 등 리눅스와 유사한 프로그래밍 인터페이스를 제공하여, 개발자들이 리눅스 환경에서 작성한 코드를 비교적 쉽게 NuttX 환경으로 이식할 수 있게 해준다. 이 덕분에 PX4의 모듈식 아키텍처가 SITL(리눅스 기반)과 실제 하드웨어(NuttX 기반) 양쪽에서 거의 동일하게 동작할 수 있다.17
- **이식성:** NuttX는 다양한 MCU 아키텍처를 지원하므로, 새로운 하드웨어 보드에 PX4를 이식하는 작업을 용이하게 한다.


NuttX 툴체인의 가장 핵심적인 구성 요소는 `gcc-arm-none-eabi`라는 크로스 컴파일러이다.

- **핵심 구성 요소:** 개발자의 컴퓨터는 대부분 x86 아키텍처(Intel, AMD)를 사용하는 반면, Pixhawk와 같은 비행 컨트롤러는 ARM Cortex-M 시리즈 아키텍처의 MCU를 사용한다. 크로스 컴파일러는 한 아키텍처(x86)에서 다른 아키텍처(ARM)에서 실행될 수 있는 기계 코드를 생성하는 컴파일러이다. 즉, `gcc-arm-none-eabi`는 PX4의 C/C++ 소스 코드를 비행 컨트롤러가 이해할 수 있는 바이너리 코드로 변환하는 역할을 한다.

- **v1.15의 버전 특정성:** 임베디드 시스템 개발에서 툴체인 버전의 일관성은 매우 중요하다. PX4 v1.15 공식 문서에서는 `GNU Arm Embedded Toolchain 9-2020-q2-update`, 즉 GCC 버전 9.3.1을 사용하도록 명시하고 있다.3

  `ubuntu.sh` 스크립트는 `apt` 저장소를 통해 바로 이 특정 버전의 툴체인을 설치하도록 설계되었다.

- **버전이 중요한 이유:** 컴파일러 버전에 따라 코드 최적화 방식, 표준 라이브러리 구현, 심지어는 언어 표준 지원 범위가 미세하게 달라질 수 있다. 다른 버전의 컴파일러를 사용하면 동일한 소스 코드라도 최종 펌웨어의 크기나 실행 속도가 달라질 수 있으며, 최악의 경우 예측하기 어려운 런타임 버그를 유발할 수 있다. PX4 프로젝트가 특정 툴체인 버전을 지정하고 스크립트를 통해 이를 강제하는 것은 전 세계 개발자들이 동일한 소스 코드로부터 비트 수준까지 동일한(bit-for-bit identical) 펌웨어를 생성하여 빌드의 재현성을 보장하기 위함이다.


크로스 컴파일러 외에도 NuttX 타겟을 빌드하고 디버깅하기 위해 여러 보조 도구들이 설치된다.

- `gdb-multiarch`: 다중 아키텍처를 지원하는 GNU 디버거이다. 일반 `gdb`가 x86 바이너리를 디버깅하는 반면, `gdb-multiarch`는 ARM용으로 컴파일된 바이너리를 디버깅하는 데 사용된다.

- `kconfig-frontends`: NuttX 커널 설정을 위한 텍스트 기반 메뉴 인터페이스(`menuconfig`)를 제공하는 도구이다. 개발자는 이를 통해 커널의 세부 기능을 활성화하거나 비활성화할 수 있다.

- `genromfs`: ROM 파일 시스템 이미지를 생성하는 유틸리티이다. PX4는 이 도구를 사용하여 기동 시 실행되는 셸 스크립트(`rcS`), 믹서 파일, 기본 파라미터 등을 펌웨어 바이너리 내에 하나의 파일 시스템으로 패키징한다.

- **하드웨어 접근 권한:** 스크립트는 `sudo usermod -a -G dialout $USER` 명령을 실행하여 현재 사용자를 `dialout` 그룹에 추가한다.13 이는 리눅스 시스템에서 매우 중요한 단계로, USB를 통해 연결된 시리얼 장치(예: Pixhawk)에 대한 접근 권한을 부여한다. 이 권한이 없으면 일반 사용자 계정으로는 QGroundControl이나 

  `make upload` 명령을 통해 펌웨어를 업로드하거나 기체와 통신할 수 없으며, 매번 `sudo`를 사용해야 하는 불편함이 따른다. 스크립트는 이 과정을 자동화하여 개발 편의성을 크게 향상시킨다.


현대의 드론 개발에서 시뮬레이션은 선택이 아닌 필수이다. 소프트웨어 인 더 루프(SITL) 시뮬레이션은 실제 하드웨어 없이 개발자의 컴퓨터 상에서 PX4 펌웨어 전체를 가상으로 실행하는 기술이다. 이는 알고리즘 개발, 비행 모드 테스트, 고장 상황 시나리오 검증 등을 물리적 기체의 손상 위험이나 비용 부담 없이 안전하고 신속하게 수행할 수 있게 해준다.4

`ubuntu.sh` 스크립트는 이러한 필수적인 SITL 환경을 구축하기 위해 Gazebo 시뮬레이터와 관련 종속성들을 설치한다.


Gazebo는 강력한 3D 물리 엔진을 갖춘 오픈소스 로봇 시뮬레이터로, PX4의 주요 SITL 백엔드 중 하나이다. `ubuntu.sh` 스크립트는 개발자의 운영체제 버전에 따라 각기 다른 버전의 Gazebo를 설치하는 지능적인 로직을 포함하고 있으며, 이는 PX4 v1.15 개발 환경 설정에서 흔히 발생하는 혼란의 원인이기도 하다.

- **v1.15의 접근 방식:** PX4 v1.15용 `ubuntu.sh` 스크립트는 다음과 같이 우분투 릴리스에 따라 다른 버전의 Gazebo를 설치한다.3
  - **우분투 18.04 (Bionic Beaver):** **Gazebo Classic 9**를 설치한다.
  - **우분투 20.04 (Focal Fossa):** **Gazebo Classic 11**을 설치한다.
  - **우분투 22.04 (Jammy Jellyfish):** 최신 버전인 **Gazebo "Garden"** (이전의 "Ignition Gazebo")을 설치한다. 'Classic'이 붙지 않은 이 버전은 기존 Gazebo와 아키텍처가 다르다.

이러한 버전 차이는 매우 중요하다. 예를 들어, 우분투 20.04에 설치된 Gazebo Classic 11을 실행하기 위해서는 `make px4_sitl_default gazebo-classic` 명령어를 사용해야 하지만 24, 우분투 22.04에 설치된 Gazebo Garden을 실행하기 위해서는 

`make px4_sitl_default gz` 명령어를 사용해야 한다. 이 차이를 인지하지 못하면 시뮬레이션 실행에 실패하게 된다.

**표 6.1: PX4 v1.15 설정의 Gazebo 버전 매트릭스**

이 표는 개발자가 자신의 우분투 버전에 따라 어떤 Gazebo가 설치되고 어떤 `make` 명령어를 사용해야 하는지를 명확하게 보여준다.

| 우분투 LTS 버전 | 코드명          | `ubuntu.sh`가 설치하는 Gazebo 버전 | PX4 `make` 타겟  |
| --------------- | --------------- | ---------------------------------- | ---------------- |
| 18.04           | Bionic Beaver   | Gazebo Classic 9                   | `gazebo-classic` |
| 20.04           | Focal Fossa     | Gazebo Classic 11                  | `gazebo-classic` |
| 22.04           | Jammy Jellyfish | Gazebo "Garden"                    | `gz`             |


Gazebo를 원활하게 구동하고 PX4와 연동하기 위해 여러 추가적인 라이브러리들이 설치된다.

- `protobuf-compiler`: 프로토콜 버퍼(Protocol Buffers) 컴파일러이다. Gazebo는 내부적으로 모듈 간 통신 및 메시지 교환을 위해 구글이 개발한 프로토콜 버퍼를 사용한다. 이 컴파일러는 `.proto` 형식의 메시지 정의 파일로부터 C++ 코드를 생성하는 데 필요하다.
- `gstreamer1.0-plugins-*`: GStreamer는 멀티미디어 처리를 위한 파이프라인 기반 프레임워크이다. `plugins-base`, `plugins-good`, `plugins-bad`, `plugins-ugly` 등 다양한 플러그인 패키지들은 시뮬레이터 내의 가상 카메라로부터 비디오 스트림을 생성하고, 이를 QGroundControl이나 다른 ROS 노드로 전송하는 기능을 구현하는 데 사용된다.13
- `libeigen3-dev`: Eigen은 행렬, 벡터, 수치 해석 등을 위한 고성능 C++ 템플릿 라이브러리이다. 3D 그래픽, 물리 엔진, 로보틱스 분야에서 사실상의 표준으로 사용되며, Gazebo와 PX4의 내부 수학 연산에 필수적이다.13


`ubuntu.sh` 스크립트에는 매우 구체적인 환경 문제를 자동으로 해결하는 로직이 포함되어 있다. 대표적인 예가 VMware 가상 머신에 대한 처리이다.

```Bash
if [[ $(dmidecode -s system-manufacturer) == "VMware, Inc." ]]; then
    # Fix 3D acceleration for gazebo
    echo "export SVGA_VGPU10=0" >> ~/.profile
fi
```

이 코드 블록은 `dmidecode` 명령어를 통해 시스템 제조업체가 "VMware, Inc."인지 확인한다. 만약 VMware 환경이라면, 사용자의 `~/.profile` 파일에 `export SVGA_VGPU10=0`이라는 환경 변수를 추가한다. 이는 VMware의 특정 3D 그래픽 가속 기능(SVGA)이 Gazebo의 렌더링 엔진과 충돌하여 그래픽 깨짐이나 실행 실패를 유발하는 알려진 문제를 해결하기 위한 조치이다. 이처럼 스크립트는 단순히 패키지를 설치하는 것을 넘어, 커뮤니티를 통해 축적된 특정 환경에 대한 문제 해결 노하우를 코드의 형태로 담고 있으며, 이를 통해 개발자가 겪을 수 있는 잠재적인 함정을 미리 제거해 준다.13


이론적인 분석을 바탕으로, 이제 PX4 v1.15 개발 환경을 실제로 구축하고 모든 것이 올바르게 작동하는지 확인하는 구체적인 절차를 정리한다. 이 과정은 새로운 개발자가 프로젝트에 참여할 때 따를 수 있는 실용적인 가이드 역할을 한다.


v1.15 생태계와의 최상의 호환성을 위해, 깨끗하게 설치된 우분투 20.04 LTS를 사용하는 것을 권장한다.

1. **사전 준비:** 지원되는 우분투 LTS 버전(18.04, 20.04, 22.04 중 20.04 권장)이 설치된 시스템을 준비한다.

2. **저장소 복제:** 터미널을 열고 다음 명령어를 실행하여 `PX4-Autopilot` 저장소와 모든 하위 모듈을 복제한다.

   ```Bash
   git clone https://github.com/PX4/PX4-Autopilot.git --recursive
   ```

   1

3. **디렉토리 이동:** 복제된 디렉토리로 이동한다.

   ```Bash
   cd PX4-Autopilot
   ```

4. **v1.15 릴리스 체크아웃:** 재현 가능한 빌드를 위해 특정 릴리스 태그로 전환한다. `v1.15.0`을 기준으로 하며, 이후 하위 모듈을 해당 버전에 맞게 업데이트한다.

   ```Bash
   git checkout v1.15.0
   git submodule update --init --recursive
   ```

   6

5. **설정 스크립트 실행:** `ubuntu.sh` 스크립트를 실행하여 모든 종속성을 설치한다. 스크립트 실행 중 `sudo` 암호를 묻거나 설치 확인 프롬프트가 나타날 수 있다.

   ```Bash
   bash./Tools/setup/ubuntu.sh
   ```

   1

6. **시스템 재부팅:** 스크립트가 완료되면, 변경된 그룹 권한(예: `dialout`)과 환경 변수가 시스템에 완전히 적용되도록 재부팅할 것을 권장한다.

   ```Bash
   sudo reboot now
   ```


설치가 완료된 후, 주요 구성 요소들이 올바른 버전으로 설치되었는지 확인하는 것은 중요하다.

- **툴체인 버전 확인:** NuttX 펌웨어 빌드를 위한 ARM 크로스 컴파일러의 버전을 확인한다.

  ```Bash
  arm-none-eabi-gcc --version
  ```

  출력 결과에 `(GNU Arm Embedded Toolchain 9-2020-q2-update)` 및 `9.3.1`과 같은 내용이 포함되어야 한다.3

- **파이썬 종속성 확인:** `pip`를 사용하여 `requirements.txt`에 명시된 패키지들이 설치되었는지 확인한다.

  ```Bash
  pip3 list
  ```

  출력 목록에서 `empy`, `kconfiglib`, `pymavlink`, `pyulog` 등의 패키지를 확인할 수 있어야 한다.


환경 검증의 마지막 단계는 실제로 펌웨어를 빌드하여 전체 툴체인이 유기적으로 작동하는지 확인하는 것이다.

- **SITL 빌드 (시뮬레이션):** 먼저 소프트웨어 인 더 루프(SITL) 타겟을 빌드한다. 우분투 20.04 환경이므로 `gazebo-classic` 타겟을 사용한다.

  ```Bash
  make px4_sitl_default gazebo-classic
  ```

  빌드가 성공적으로 완료되면 Gazebo 시뮬레이터 창이 나타나고, 터미널에는 PX4 SITL 인스턴스의 콘솔 프롬프트(`pxh>`)가 표시된다. 이는 시뮬레이션 관련 종속성과 빌드 시스템이 모두 올바르게 작동함을 의미한다. Gazebo 창에서 가상의 드론이 나타나고, PX4 콘솔에서 `commander takeoff`와 같은 명령어를 입력했을 때 드론이 이륙하면 성공이다.4

- **하드웨어 빌드 (NuttX):** 다음으로 실제 비행 컨트롤러용 펌웨어를 빌드한다. 예를 들어, Pixhawk 4 (FMUv5) 보드를 타겟으로 하는 경우 다음 명령어를 실행한다.

  ```Bash
  make px4_fmu-v5_default
  ```

  이 명령어가 오류 없이 완료되고 `build/px4_fmu-v5_default/` 디렉토리 내에 `px4_fmu-v5_default.px4` 파일이 생성되었다면, NuttX 툴체인(ARM 크로스 컴파일러 등)이 성공적으로 구성되었음을 의미한다. 이 `.px4` 파일은 실제 하드웨어에 업로드할 수 있는 최종 펌웨어이다.4

이러한 검증 절차를 모두 통과했다면, PX4 v1.15 개발을 위한 모든 준비가 완료된 것이다.



PX4-Autopilot v1.15의 `ubuntu.sh` 스크립트에 대한 심층 분석 결과, 이 스크립트는 단순한 패키지 설치 도구를 훨씬 뛰어넘는 정교하고 지능적인 시스템임이 명확해졌다. 이 스크립트는 환경을 인지하고(우분투 버전, 아키텍처, 도커 환경), 모듈화된(NuttX 및 시뮬레이션 도구 선택적 설치) 아키텍처를 통해 다양한 개발자의 요구사항에 부응한다. 본질적으로 `ubuntu.sh`는 PX4 프로젝트가 전 세계의 분산된 개발자 커뮤니티를 대상으로 안정적이고 일관된 개발 경험을 제공하기 위한 **기술적 정책 강제 메커니즘**으로 기능한다.

스크립트는 특정 버전의 컴파일러, 라이브러리, 파이썬 패키지를 명시적으로 설치함으로써, "내 컴퓨터에서는 되는데"와 같은 일반적인 개발 환경 의존성 문제를 사전에 방지한다. 또한, 시리얼 포트 권한 설정이나 VMware 그래픽 문제 해결과 같이 커뮤니티를 통해 축적된 실용적인 노하우를 코드에 통합하여, 개발자가 겪을 수 있는 잠재적인 장애물들을 자동으로 제거한다. 결론적으로, `ubuntu.sh`는 PX4 v1.15 개발을 시작하기 위한 가장 신뢰할 수 있고 효율적인 경로이며, 그 구조와 내용을 이해하는 것은 PX4 생태계의 복잡성을 성공적으로 탐색하는 데 필수적이다.


이 분석을 바탕으로, PX4 v1.15 개발 환경을 구축하려는 개발자에게 다음과 같은 전문가적 권장 사항을 제시한다.

- **스크립트를 신뢰하고 따를 것:** 항상 지원되는 우분투 LTS 버전(v1.15의 경우 18.04, 20.04, 22.04)의 깨끗한 설치 환경에서 `ubuntu.sh` 스크립트를 사용하는 것을 원칙으로 삼아야 한다. 수동으로 패키지를 설치하거나 지원되지 않는 OS 버전을 사용하려는 시도는 진단하기 어려운 문제로 이어질 가능성이 높다.
- **플래그를 적극적으로 활용할 것:** `--no-nuttx`와 `--no-sim-tools` 플래그의 의미를 이해하고, 자신의 개발 목적에 맞게 활용해야 한다. CI/CD 파이프라인, 컴패니언 컴퓨터 설정, 또는 순수 하드웨어 펌웨어 개발과 같이 특정 목적을 위한 최소한의 린(lean) 환경을 구축하는 데 이 플래그들은 매우 유용하다.
- **버전 고정(Pinning)의 중요성:** 연구의 재현성이나 제품 개발의 안정성을 위해서는, 항상 `git checkout`을 통해 특정 릴리스 태그(예: `v1.15.2`)를 명시적으로 지정하여 작업해야 한다. 계속해서 변경되는 `main`이나 `release` 브랜치에서 작업하는 것은 예기치 않은 변경 사항으로 인해 빌드가 실패하거나 동작이 달라지는 원인이 될 수 있다. `ubuntu.sh`는 체크아웃한 특정 버전의 소스 코드에 맞는 정확한 개발 환경을 보장한다.
- **ARM 기반 시스템에서의 문제 해결:** Apple Silicon (M1/M2)이나 다른 ARM 기반 호스트에서 리눅스 가상 머신을 사용하는 개발자는 `ubuntu.sh` 실행 시 x86 전용 `multilib` 패키지 관련 오류를 마주할 수 있다. 커뮤니티 포럼에서 논의된 바와 같이, 이 경우 스크립트를 수동으로 편집하여 관련 라인을 주석 처리해야 할 수 있음을 인지하고 있어야 한다.16


PX4는 활발하게 개발이 진행 중인 프로젝트이다. v1.15 릴리스 이후 `main` 브랜치의 `ubuntu.sh` 스크립트는 이미 많은 변화를 겪었다. 예를 들어, 우분투 18.04와 같은 오래된 버전에 대한 지원이 중단되고, Gazebo Classic 대신 최신 Gazebo(Harmonic) 버전으로의 전환이 가속화되었다.13 이는 PX4 생태계가 지속적으로 발전하고 있음을 보여주는 동시에, 개발자가 빌드하려는 소스 코드의 버전에 맞는 버전의 `ubuntu.sh` 스크립트를 사용하는 것이 얼마나 중요한지를 강조한다. 항상 작업하려는 코드 저장소에 포함된 설정 스크립트를 사용하는 것이 불일치로 인한 문제를 피하는 가장 확실한 방법이다.


1. Ubuntu Development Environment | PX4 User Guide (v1.14), accessed July 25, 2025, https://docs.px4.io/v1.14/en/dev_setup/dev_env_linux_ubuntu.html
2. PX4-user_guide/tr/dev_setup/dev_env_linux_ubuntu.md at main - GitHub, accessed July 25, 2025, https://github.com/PX4/PX4-user_guide/blob/main/tr/dev_setup/dev_env_linux_ubuntu.md
3. Ubuntu Development Environment | PX4 Guide (v1.15), accessed July 25, 2025, https://docs.px4.io/v1.15/en/dev_setup/dev_env_linux.html
4. Building PX4 Software | PX4 Guide (v1.15), accessed July 25, 2025, https://docs.px4.io/v1.15/en/dev_setup/building_px4.html
5. Building PX4 - EchoPilot AI Documentation, accessed July 25, 2025, https://echomav.github.io/docs/Rev1B/build_px4/
6. Download PX4 Source Code in Ubuntu 20.04 and 22.04 - MATLAB & - MathWorks, accessed July 25, 2025, https://la.mathworks.com/help/uav/px4/ug/download-px4-source-code-ubuntu.html
7. GIT Examples | PX4 Guide (main), accessed July 25, 2025, https://docs.px4.io/main/en/contribute/git_examples.html
8. Setting up your PX4 development environment on Linux - Getting Started with Drone Development - YouTube, accessed July 25, 2025, https://www.youtube.com/watch?v=OtValQdAdrU
9. Building PX4 - AirSim - Microsoft Open Source, accessed July 25, 2025, https://microsoft.github.io/AirSim/px4_build/
10. PX4-user_guide/en/dev_setup/building_px4.md at main - GitHub, accessed July 25, 2025, https://github.com/PX4/PX4-user_guide/blob/main/en/dev_setup/building_px4.md
11. PX4-Autopilot v1.15 Release Notes | PX4 Guide (main), accessed July 25, 2025, https://docs.px4.io/main/en/releases/1.15
12. PX4 Autopilot Release v1.15: What You Need To Know, accessed July 25, 2025, https://px4.io/px4-autopilot-release-v1-15-what-you-need-to-know/
13. PX4-Autopilot/Tools/setup/ubuntu.sh at main - GitHub, accessed July 25, 2025, https://github.com/PX4/PX4-Autopilot/blob/main/Tools/setup/ubuntu.sh
14. Ubuntu Development Environment | PX4 User Guide (v1.12), accessed July 25, 2025, https://docs.px4.io/v1.12/en/dev_setup/dev_env_linux_ubuntu.html
15. Can a PX4 development environment be setup on an ARM-based VM?, accessed July 25, 2025, https://discuss.px4.io/t/can-a-px4-development-environment-be-setup-on-an-arm-based-vm/26227
16. Installing PX4-Autopilot for M1 Mac Users Running Parallels Ubuntu 20.04 Virtual Machine #21117 - GitHub, accessed July 25, 2025, https://github.com/PX4/PX4-Autopilot/issues/21117
17. PX4 Autopilot - GitHub, accessed July 25, 2025, https://github.com/px4
18. Parameters & Configurations | PX4 Guide (v1.15), accessed July 25, 2025, https://docs.px4.io/v1.15/en/advanced/parameters_and_configurations.html
19. PX4-Autopilot/Tools/setup/requirements.txt at main - GitHub, accessed July 25, 2025, https://github.com/PX4/PX4-Autopilot/blob/master/Tools/setup/requirements.txt
20. PX4 installation - PX4 Autopilot - Discussion Forum for PX4, Pixhawk, QGroundControl, MAVSDK, MAVLink, accessed July 25, 2025, https://discuss.px4.io/t/px4-installation/29278
21. PX4 Autopilot Software - GitHub, accessed July 25, 2025, https://github.com/PX4/PX4-Autopilot
22. About PX4 Autopilot | NXP UCANS32K1 CAN Nodes - GitBook, accessed July 25, 2025, https://nxp.gitbook.io/ucans32k1/px4-autopilot/about-px4-autopilot
23. Error building px4_fmu-v5 - PX4 Autopilot, accessed July 25, 2025, https://discuss.px4.io/t/error-building-px4-fmu-v5/35504
24. GAZEBO PX4 Setup (with ROS NOETIC UBUNTU 20.04) - YouTube, accessed July 25, 2025, https://www.youtube.com/watch?v=dX6C4U5TTug
25. PX4 install on Ubuntu 24.04 - Help - Gazebo Community, accessed July 25, 2025, https://community.gazebosim.org/t/px4-install-on-ubuntu-24-04/3704

