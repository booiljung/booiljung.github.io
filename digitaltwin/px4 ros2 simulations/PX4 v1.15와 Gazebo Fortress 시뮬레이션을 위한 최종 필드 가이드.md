[PX4 ROS2 Simulation](./index.md)
# PX4 v1.15와 Gazebo Fortress 시뮬레이션을 위한 최종 필드 가이드


PX4 개발 환경, 특히 시뮬레이션 설정은 혼란의 도가니다. 이 혼란의 근원을 이해하지 못하면, 개발자는 끝없는 의존성 문제와 비호환성 오류의 늪에 빠지게 된다. 이 섹션에서는 현재의 혼란을 야기한 시뮬레이터의 역사, 명칭 변경, 그리고 버전 관리의 복잡성을 해부하여, 개발자가 겪는 문제의 근본 원인을 명확히 하고 안정적인 개발 환경을 구축하기 위한 필수적인 기초 지식을 제공한다.


현재 시뮬레이터 생태계는 두 개의 주요 갈래로 나뉘어 있다. 하나는 "Gazebo Classic"이라 불리는 레거시 시뮬레이터이고, 다른 하나는 이전에 "Ignition Gazebo"로 알려졌다가 현재는 단순히 "Gazebo"라고 불리는 현대적인 시뮬레이터다. 이 명칭 변경은 현재 개발자들이 겪는 혼란의 '원죄'와도 같다. 개발자가 도움을 얻기 위해 "Gazebo"를 검색하면, 지난 10년간 축적된 방대한 양의 튜토리얼과 문서가 나오지만, 이들 대부분은 이제 레거시 시스템인 "Gazebo Classic"에 관한 것이라 현대적인 툴체인에는 적용되지 않는다.

이 보고서에서는 명확한 구분을 위해 다음과 같은 규칙을 따른다. 만약 `make` 명령어가 `gazebo-classic`으로 시작한다면, 이는 구형 시스템을 사용하는 것이다. 반면, 명령어가 `gz_`로 시작한다면, 이는 신형 Gazebo 시스템을 사용하는 것을 의미한다. 가장 중요한 점은, 이 두 시스템 간에 차량 모델, 월드, 플러그인이 전혀 호환되지 않는다는 것이다. 이는 단순히 이름만 바뀐 것이 아니라, API와 플러그인 시스템을 포함한 내부 아키텍처가 완전히 다르기 때문이다. 따라서 Gazebo Classic에서 축적된 기술과 자산은 새로운 Gazebo로 직접 이전되지 않으며, 이는 커뮤니티에 분절된 지식과 쓸모없어진 자산이라는 상당한 비용을 초래했다.


새로운 Gazebo는 그 자체로 Fortress, Garden, Harmonic 등 여러 버전으로 나뉜다. 문제는 여기서부터 복잡해진다. PX4의 공식 설치 스크립트인 `ubuntu.sh`는 Ubuntu 22.04 환경에서 Gazebo "Harmonic" 버전을 설치한다. 하지만 Ubuntu 22.04의 표준 ROS 2 버전인 Humble Hawksbill과 함께 사용하도록 권장되는 Gazebo 버전은 "Fortress"이다.

이것은 매우 중요한 모순이며, 개발자들을 함정에 빠뜨리는 주된 요인이다. 개발자를 돕기 위해 만들어진 `ubuntu.sh` 스크립트가 오히려 비표준적인 환경을 조성하여, 특히 ROS 2를 함께 사용할 때 미묘한 버그와 의존성 충돌을 야기한다. 스크립트가 이렇게 동작하는 이유는 Ubuntu의 LTS(Long-Term Support) 릴리스 주기와 밀접한 관련이 있다. Canonical이 Ubuntu 22.04를 출시하면서 Gazebo Classic(버전 11)을 더 이상 쉽게 지원하거나 패키징하기 어려워졌고, 이로 인해 PX4 프로젝트는 어쩔 수 없이 새로운 Gazebo로의 전환을 강요받았다. 스크립트가 Fortress 대신 Harmonic을 선택한 것은 당시의 안정성이나 가용성에 기반한 결정이었을 수 있으나, 현재는 더 넓은 생태계의 권장 사항과 충돌하게 된 것이다.

이러한 상황은 여러 대규모 오픈소스 프로젝트의 비조정된 릴리스 일정으로 인해 발생하는 시스템적인 문제임을 보여준다. 이 문제의 연쇄 고리는 다음과 같다:

1. **원인:** Canonical이 Ubuntu 22.04를 릴리스한다.
2. **결과:** Gazebo Classic (v11)의 지원 및 패키징이 어려워진다.
3. **결과:** PX4 프로젝트는 v1.15에서 새로운 Gazebo를 기본 시뮬레이터로 채택하고 `ubuntu.sh` 스크립트를 업데이트한다.
4. **결과:** 이 스크립트는 특정 Gazebo 버전(예: Harmonic)을 설치한다.
5. **결과:** ROS 2 커뮤니티는 해당 ROS 2 LTS(Humble)에 대해 다른 버전(Fortress)을 표준으로 채택한다.
6. **최종 결과:** 개발자는 두 프로젝트의 공식 가이드를 따랐음에도 불구하고, 서로 충돌하고 디버깅하기 어려운 환경에 놓이게 된다.

이 문제를 해결하기 위한 유일한 방법은 개발자가 수동으로 자신의 환경을 통제하는 것이다.


PX4 v1.15 릴리스는 공식적으로 Gazebo를 기본 시뮬레이터로 지정하고, 기존의 jMAVSim을 대체했다. 또한 이 릴리스는 새로운 C++ 인터페이스 라이브러리를 포함하여 ROS 2 통합을 위한 주요 개선 사항을 도입했다. 이는 더 이상 새로운 Gazebo를 이해하는 것이 선택이 아닌 필수가 되었음을 의미한다. 이전 버전의 PX4 사용자는 이 변화를 무시할 수 있었을지 모르지만, v1.15부터는 이것이 앞으로 나아갈 주요 경로다. 특히 ROS 2 통합에 대한 집중적인 개선은 Gazebo 버전 충돌 문제를 해결하는 것이 그 어느 때보다 중요해졌음을 시사한다.


개발자의 가장 소중한 자원은 시간이다. 잘못된 초기 설정으로 인해 깨진 환경과 싸우며 며칠을 낭비하는 것은 흔하고 사기를 꺾는 경험이다. 아래의 표는 십여 개가 넘는 소스에서 흩어져 있는 호환성 정보를 단 하나의 명확한 참조 자료로 종합한 것이다. 이 표는 가장 흔한 설정 오류를 직접적으로 방지하고, 이 보고서의 나머지 부분을 위한 기초적인 "치트 시트" 역할을 한다.

**표 1: PX4 v1.15 시뮬레이션 호환성 매트릭스**

| 운영체제             | PX4 버전     | 권장 Gazebo                      | 권장 ROS 2                 | 주요 `make` 접두사 | 핵심 노트 및 일반적인 문제                                   |
| -------------------- | ------------ | -------------------------------- | -------------------------- | ------------------ | ------------------------------------------------------------ |
| Ubuntu 22.04 LTS     | v1.15 / main | **Fortress**                     | Humble                     | `gz_`              | **주요 권장 설정.** `ubuntu.sh`는 Harmonic을 설치하므로 Fortress의 수동 설치가 필요함. 다른 버전이 존재할 경우 protobuf 충돌 가능성 있음. |
| Ubuntu 20.04 LTS     | v1.15 / main | Gazebo Classic (v11)             | Foxy                       | `gazebo-classic_`  | 레거시지만 안정적인 경로. 새로운 Gazebo(Garden)를 수동으로 설치할 수 있지만 Classic을 제거함. 향후 PX4 개발의 초점은 아님. |
| Windows 10/11 (WSL2) | v1.15 / main | **Fortress** (Ubuntu 22.04 통해) | Humble (Ubuntu 22.04 통해) | `gz_`              | 네이티브 Ubuntu 22.04와 동일하나, QGroundControl을 위한 추가적인 네트워킹 복잡성이 있음 (섹션 5 참조). |
| macOS                | v1.15 / main | Gazebo Classic (v11)             | -                          | `gazebo-classic_`  | 새로운 Gazebo는 macOS에서 잘 지원되지 않음. 개발은 Classic 시뮬레이터로 제한됨. |


이 섹션에서는 *올바른* 환경을 구축하기 위한 규범적이고 단계적인 가이드를 제공한다. 공식 문서가 때때로 간과하는 필수적인 수동 개입 단계를 강조하여, 개발자가 안정적인 기반 위에서 시작할 수 있도록 한다.


이 가이드는 네이티브 또는 Windows의 WSL2를 통한 Ubuntu 22.04 LTS에 초점을 맞춘다. 이것이 PX4 개발을 위해 가장 권장되는 플랫폼이기 때문이다. 시작하기 전에, 기본적인 시스템 업데이트 (`sudo apt update && sudo apt upgrade`), `git` 설치를 완료해야 한다. WSL2 사용자의 경우, `wsl --install` 명령을 통한 초기 설치와 BIOS에서의 가상화 기능 활성화 확인이 선행되어야 한다.


표준 절차는 `PX4-Autopilot` 저장소를 복제하고 `Tools/setup/ubuntu.sh` 스크립트를 실행하는 것이다. 이 스크립트는 ARM 크로스 컴파일러, Python 의존성 등 PX4 빌드에 필수적인 전체 툴체인을 설치하므로 반드시 거쳐야 할 단계다. 터미널에서 다음 명령을 실행한다:

```Bash
git clone https://github.com/PX4/PX4-Autopilot.git --recursive
cd PX4-Autopilot
bash./Tools/setup/ubuntu.sh
```


이것이 안정적인 환경을 구축하는 가장 중요한 단계다. 앞서 설명했듯이, `ubuntu.sh` 스크립트는 Fortress가 아닌 Gazebo Harmonic 또는 Garden을 설치하며, 여러 버전의 Gazebo를 함께 설치하면 심각한 충돌을 일으킨다.

이 문제를 해결하기 위한 "비밀 악수"는 바로 수동 개입이다. 다음의 정확한 명령 시퀀스를 따라 스크립트가 설치한 Gazebo 버전을 완전히 제거하고, Fortress를 명시적으로 설치해야 한다.

1. 기존 Gazebo 및 관련 패키지 완전 제거:

   스크립트가 설치했을 수 있는 모든 Gazebo, Ignition, gz 관련 패키지를 제거하여 시스템을 깨끗한 상태로 만든다.

   ```Bash
   sudo apt-get remove --purge '.*gazebo.*' '.*gz-.*' '.*ignition-.*' -y
   sudo apt-autoremove -y
   ```

2. OSRF(Open Source Robotics Foundation) 저장소 추가:

   Gazebo 패키지를 제공하는 공식 저장소를 시스템에 추가한다.

   ```Bash
   sudo wget https://packages.osrfoundation.org/gazebo.gpg -O /usr/share/keyrings/pkgs-osrf-archive-keyring.gpg
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/pkgs-osrf-archive-keyring.gpg] http://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/gazebo-stable.list > /dev/null
   sudo apt-get update
   ```

3. Gazebo Fortress 명시적 설치:

   이제 ignition-fortress 패키지를 설치한다. ign 접두사는 레거시 이름이지만, 패키지 이름은 여전히 이를 사용한다.

   ```Bash
   sudo apt-get install ignition-fortress -y
   ```

이 수동 개입 과정을 명시적으로 문서화함으로써, 이 가이드는 PX4와 ROS 2 문서 간의 핵심적인 모순을 해결하고, 수많은 포럼 게시물에서 언급된 의존성 지옥을 예방한다.


모든 설정이 완료되면, `PX4-Autopilot` 디렉터리에서 다음 명령을 실행하여 툴체인이 올바르게 설치되었는지 확인한다:

```Bash
make px4_sitl
```

이 명령은 시뮬레이터 없이 PX4의 SITL(Software-in-the-Loop) 바이너리만 빌드한다. 빌드가 성공적으로 완료되면 ARM 크로스 컴파일러와 기타 PX4 관련 도구들이 올바르게 설치되었음을 의미한다. 이를 통해 툴체인 문제와 시뮬레이터 문제를 분리하여 진단할 수 있다.


환경 구축이 완료되었으니, 이제 시뮬레이션을 실행하고 맞춤화하는 다양한 방법을 알아본다. 이 섹션에서는 기본적인 실행부터 고급 제어까지 다룬다.


시뮬레이션을 시작하는 가장 간단한 방법은 `make` 명령을 사용하는 것이다. 예를 들어, x500 쿼드로터 모델로 시뮬레이션을 시작하려면 다음 명령을 사용한다:

```Bash
make px4_sitl gz_x500
```

모든 현대적인 Gazebo 타겟은 `gz_` 접두사를 사용한다는 점을 기억해야 한다. GUI 없이 시뮬레이션을 실행하여 시스템 자원을 절약하려면 `HEADLESS=1` 플래그를 추가할 수 있다:

```Bash
HEADLESS=1 make px4_sitl gz_x500
```


`make` 명령어는 편리하지만, 내부적으로는 PX4 SITL 바이너리를 특정 환경 변수와 함께 실행하는 단축키에 불과하다. 이 환경 변수들을 직접 제어하면 시뮬레이션 실행을 훨씬 더 세밀하게 맞춤화할 수 있다.

이 변수들을 이해하는 것은 개발자에게 커스텀 실행 스크립트를 작성하고 PX4 빌드 시스템을 수정하지 않고도 유연하게 시뮬레이션을 제어할 수 있는 힘을 준다.

- `PX4_SIMULATOR=gz`: PX4에 새로운 Gazebo 백엔드를 사용하도록 명시적으로 지시한다.
- `PX4_SIM_MODEL`: 시뮬레이터에 스폰할 차량 모델의 이름을 지정한다 (예: `x500`). 이 변수는 이제 사용되지 않는 `PX4_GZ_MODEL`을 대체하므로, 오래된 튜토리얼을 볼 때 주의해야 한다.
- `PX4_GZ_WORLD`: 로드할 월드 파일의 이름을 지정한다 (예: `baylands`).
- `PX4_GZ_MODEL_POSE="x,y,z,R,P,Y"`: 차량을 지정된 위치(미터)와 방향(라디안)으로 스폰한다.
- `PX4_GZ_MODEL_NAME`: 이미 실행 중인 Gazebo 인스턴스에 있는 특정 이름의 모델에 PX4를 연결한다.

예를 들어, `make px4_sitl gz_x500` 명령은 내부적으로 `PX4_SYS_AUTOSTART=4001`과 같은 변수를 설정하여 실행된다. 이 `AUTOSTART` ID는 특정 기체 설정 파일을 가리키며, 이 파일 안에서 `PX4_SIM_MODEL=x500`과 같은 변수들이 정의되어 있다.


개발 과정에서 코드를 약간 수정하고 다시 컴파일하여 테스트하는 작업은 매우 빈번하게 일어난다. 이때마다 무거운 Gazebo 시뮬레이터를 재시작하는 것은 상당한 시간 낭비다. 더 효율적인 방법은 PX4와 Gazebo를 별도로 시작하는 것이다.

이 워크플로우는 두 개의 터미널을 사용한다:

1. 터미널 1 (Gazebo 실행):

   먼저 Gazebo 서버를 원하는 월드와 함께 실행한다. -r 플래그는 시뮬레이션을 즉시 '재생' 상태로 시작하게 한다.

   ```Bash
   gz sim -r windy.sdf
   ```

2. 터미널 2 (PX4 SITL 실행):

   이제 PX4 SITL 인스턴스를 시작하면서, 이미 실행 중인 Gazebo에 연결하도록 지시한다. 이때 PX4_GZ_MODEL을 사용하여 새로운 모델을 스폰할 수 있다.

   ```Bash
   PX4_SIMULATOR=gz PX4_GZ_MODEL=x500./build/px4_sitl_default/bin/px4
   ```

이 방식을 사용하면, PX4 코드만 수정하고 다시 컴파일한 후 터미널 2에서 PX4 SITL만 재시작하면 된다. Gazebo는 계속 실행 중이므로 로딩 시간을 기다릴 필요가 없다.


이 섹션은 어떤 기체를 시뮬레이션할 수 있는지, 그리고 특정 모델을 선택하는 이유가 무엇인지에 대한 실용적인 참조 자료 역할을 한다.


`x500`은 다양한 센서가 장착된 여러 변형 모델로 제공되어, 특정 기능 테스트에 매우 유용하다.

- `gz_x500`: 표준 비행 제어 알고리즘 테스트용 기본 모델.
- `gz_x500_vision`: VIO(Visual-Inertial Odometry) 알고리즘 개발 및 테스트용.
- `gz_x500_depth`: OAK-D와 같은 깊이 카메라를 사용한 충돌 회피 및 장애물 감지 알고리즘 테스트용.
- `gz_x500_lidar_2d`: 2D 라이다를 이용한 SLAM, 매핑, 내비게이션 알고리즘 테스트용.
- `gz_x500_gimbal`: 카메라 짐벌 제어 및 페이로드 통합 테스트용.


고정익 항공기와 수직이착륙기(VTOL) 모델은 특히 복잡한 비행 동역학 및 전환 로직 테스트에 사용된다.

- `gz_rc_cessna`: 기본적인 고정익 비행 제어 테스트.
- `gz_advanced_plane`: 더 정교한 공기역학 플러그인을 사용하여 고충실도 시뮬레이션을 제공한다. 특정 기체의 공기역학적 특성을 더 정확하게 모델링할 때 유용하다.
- `gz_standard_vtol`, `gz_quadtailsitter`, `gz_tiltrotor`: 각기 다른 방식의 VTOL 기체 모델로, 멀티콥터 모드와 고정익 모드 간의 복잡하고 중요한 전환 로직을 테스트하는 데 필수적이다.


지상 차량 개발을 위한 모델도 지원된다.

- `gz_r1_rover`: 차동 구동 방식 로버.
- `gz_rover_ackermann`: 애커먼 조향 방식 로버.


PX4는 특정 테스트 시나리오를 위해 다양한 월드를 제공한다. `make` 명령어의 마지막에 월드 이름을 추가하거나(`make px4_sitl gz_x500_windy`), `PX4_GZ_WORLD` 환경 변수를 사용하여 월드를 지정할 수 있다.

- `windy`: 바람이 부는 환경을 시뮬레이션하여, 외란에 대한 비행 컨트롤러의 강인성을 테스트한다.
- `baylands`: 넓은 개활지와 수역이 있어 GPS 기반 장거리 임무 비행 테스트에 적합하다.
- `walls`: 벽과 장애물이 있는 환경으로, 충돌 방지 알고리즘을 테스트하는 데 사용된다.
- `aruco`: Aruco 마커가 배치되어 있어, 하방 카메라를 이용한 정밀 착륙 알고리즘 테스트에 최적화되어 있다.


QGroundControl(QGC)은 PX4 개발에 필수적인 지상 통제 소프트웨어(GCS)다. 이 섹션에서는 QGC를 시뮬레이션 환경에 연결하는 방법을 상세히 다루며, 특히 WSL2 환경에서 발생하는 일반적인 함정을 해결하는 데 중점을 둔다.


네이티브 리눅스 시스템에서는 QGC 연결이 매우 간단하다. PX4 SITL은 기본적으로 UDP 포트 14550으로 MAVLink 메시지를 브로드캐스트하며, QGC는 이 포트를 자동으로 감지하여 연결한다. QGC 공식 웹사이트에서 최신 AppImage를 다운로드하고 실행 권한을 부여한 뒤 실행하면 된다.

```Bash
chmod +x./QGroundControl.AppImage
./QGroundControl.AppImage
```


Windows에서 실행되는 QGC는 WSL2 내부에서 실행되는 SITL 인스턴스에 자동으로 연결되지 않는다. 이는 WSL2가 Windows 호스트와는 별개의 IP 주소를 가진 가상 네트워크 위에서 동작하기 때문이다. 따라서 사용자는 QGC에 수동으로 UDP 통신 링크를 생성해야 한다.

이 문제는 개발자들이 흔히 겪는 큰 장애물이다. 다음은 이 문제를 해결하기 위한 명확하고 그림 같은 단계별 안내다.

1. WSL2의 IP 주소 확인:

   WSL2 터미널을 열고 다음 명령을 실행하여 가상 이더넷 어댑터(eth0)의 IP 주소를 확인한다.

   ```Bash
   ip addr | grep eth0
   ```

   출력에서 `inet` 다음에 나오는 IP 주소(예: `172.25.98.123`)를 복사한다.

2. Windows QGC에서 UDP 링크 생성:

   Windows에서 QGC를 실행하고, 좌측 상단의 'Q' 아이콘을 클릭하여 Application Settings > Comm Links로 이동한다.

3. 링크 설정 및 연결:

   Add 버튼을 클릭하여 새로운 통신 링크를 생성한다.

   - **Name:** `WSL` 또는 식별하기 쉬운 이름으로 설정한다.
   - **Type:** `UDP`로 선택한다.
   - **Server Addresses:** `Add`를 클릭하고 1단계에서 복사한 IP 주소를 입력한다.
   - **Listening Port:** `14550`으로 설정한다.
   - `OK`를 눌러 링크를 저장하고, 새로 생성된 `WSL` 링크를 선택한 후 `Connect` 버튼을 누른다.

이 과정을 거치면 Windows QGC가 WSL2의 PX4 SITL에 성공적으로 연결된다.

가장 중요한 점은 이 WSL2의 IP 주소가 영구적이지 않다는 것이다. 호스트 컴퓨터를 재부팅하거나 WSL을 재시작할 때마다 IP 주소가 변경될 수 있다. 연결이 되지 않을 때는 항상 1단계부터 다시 시작하여 새로운 IP 주소로 QGC의 통신 링크를 업데이트해야 한다. 이 사실을 미리 인지하면, 반복적으로 발생하는 "버그"를 개발 환경의 예측 가능하고 관리 가능한 특성으로 이해할 수 있다.


이 섹션은 PX4를 더 넓은 로봇 공학 생태계와 연동해야 하는 개발자들을 위한 것이다.


PX4와 ROS 2 간의 통신은 Micro XRCE-DDS 에이전트와 클라이언트를 통해 이루어진다. 먼저, 시스템에 DDS 에이전트를 설치해야 한다.

```Bash
sudo apt update
sudo apt install micrortps-agent
```

그런 다음, PX4 SITL이 통신할 수 있도록 올바른 UDP 포트(기본값 8888)로 에이전트를 실행한다:

```Bash
MicroXRCEAgent udp4 -p 8888
```

이 에이전트가 실행 중인 상태에서 PX4 SITL을 시작하면, PX4 내부의 uXRCE-DDS 클라이언트가 이 에이전트에 연결하여 uORB 메시지를 ROS 2 네트워크의 DDS 토픽으로 변환하고, 그 반대의 작업도 수행한다.


DDS 에이전트를 활성화하면 시뮬레이션이 불안정해지면서 드론이 비정상적으로 움직이거나 가라앉고, 콘솔에 `[timesync] time jump detected` 오류가 출력되는 현상이 발생할 수 있다.

이것은 매우 중요한 아키텍처적 문제이며, 그 원인을 이해하는 것이 핵심이다. Gazebo와 PX4는 시뮬레이션된 시간을 사용한다. 이 시간은 일시 중지될 수도 있고, 실제 시간보다 빠르거나 느리게 실행될 수도 있다. 반면, ROS 2 노드와 DDS 에이전트는 기본적으로 시스템의 실제 "벽시계 시간(wall clock)"을 사용한다. 만약 시뮬레이션을 잠시 멈췄다가 다시 시작하면, 벽시계 시간은 계속 흘러갔기 때문에 PX4는 ROS 2로부터 들어오는 메시지에서 엄청난 시간 점프를 감지하게 된다. 이는 EKF(확장 칼만 필터)와 제어기들이 오작동하는 원인이 된다.

해결책은 단순히 명령어를 입력하는 것이 아니라, 로봇 공학 시뮬레이션의 근본적인 개념을 이해하는 데 있다. ROS 2는 `use_sim_time`이라는 파라미터를 제공한다. ROS 2 launch 파일에서 이 파라미터를 `true`로 설정하면, 모든 ROS 2 노드들은 시스템의 벽시계 시간을 무시하고, 대신 Gazebo-ROS 브리지가 발행하는 `/clock` 토픽을 구독하여 시간을 동기화한다. 이렇게 하면 시스템의 모든 구성 요소가 단일화된 시뮬레이션 시간을 공유하게 되어 문제를 근본적으로 해결할 수 있다. 이와 함께, 올바른 ROS-Gazebo 브리지 패키지(예: `ros-humble-ros-gzfortress`)가 설치되어 있어야 한다.


다음은 시뮬레이션에서 실시간 데이터를 수신하는 최소한의 완전한 ROS 2 예제다.

1. **워크스페이스 생성 및 `px4_msgs` 추가:**

   ```Bash
   mkdir -p ~/px4_ros_com_ws/src
   cd ~/px4_ros_com_ws/src
   git clone https://github.com/PX4/px4_msgs.git
   ```

2. Python 구독자 노드 작성:

   ~/px4_ros_com_ws/src에 px4_ros_examples 같은 새 패키지를 만들고, 그 안에 vehicle_odometry_listener.py라는 파일을 다음과 같이 작성한다.

   ```Python
   import rclpy
   from rclpy.node import Node
   from px4_msgs.msg import VehicleOdometry
   
   class OdometryListener(Node):
       def __init__(self):
           super().__init__('odometry_listener')
           self.subscription = self.create_subscription(
               VehicleOdometry,
               '/fmu/out/vehicle_odometry',
               self.listener_callback,
               10)
           self.get_logger().info('Odometry listener started.')
   
       def listener_callback(self, msg):
           self.get_logger().info(f"Position: [x: {msg.position:.2f}, y: {msg.position:.2f}, z: {msg.position:.2f}]")
   
   def main(args=None):
       rclpy.init(args=args)
       odometry_listener = OdometryListener()
       rclpy.spin(odometry_listener)
       odometry_listener.destroy_node()
       rclpy.shutdown()
   
   if __name__ == '__main__':
       main()
   ```

3. 빌드 및 실행:

   워크스페이스 루트(~/px4_ros_com_ws)로 이동하여 빌드하고, 환경을 소싱한 후 노드를 실행한다.

   ```Bash
   cd ~/px4_ros_com_ws
   colcon build
   source install/setup.bash
   ros2 run px4_ros_examples vehicle_odometry_listener.py
   ```

이제 PX4 SITL 시뮬레이션을 실행하면, 이 ROS 2 노드가 드론의 위치 데이터를 실시간으로 수신하여 터미널에 출력하는 것을 볼 수 있다.


이 섹션은 개발자가 스스로 문제를 진단하고 해결할 수 있도록 돕는 종합적인 문제 해결 매뉴얼이다.


- **문제:** `make` 실행 중 Protobuf 버전 충돌 발생.
- **진단:** 오류 메시지에 `File already exists in database` 또는 protobuf 버전 불일치와 관련된 내용이 포함된다.
- **해결책:** 이 문제의 거의 모든 원인은 여러 개의 충돌하는 Gazebo/Ignition 버전이 설치되어 있기 때문이다. 해결책은 섹션 2.3의 절차에 따라 모든 관련 버전을 완전히 제거하고 오직 Fortress만 설치하는 것이다.


- **문제:** PX4 SITL이 `Waiting for simulator to accept connection on TCP port 4560`에서 멈춤.
- **진단:** PX4 프로세스는 시작되었지만 Gazebo 서버와의 TCP 연결을 설정하지 못하고 있다.
- **해결책:**
  1. Gazebo가 실제로 실행 중인지 확인한다. `make` 명령은 둘 다 실행해야 하지만, 독립 실행 모드에서는 Gazebo를 먼저 시작해야 한다.
  2. 좀비/유령 Gazebo 프로세스가 있는지 확인하고(`ps aux | grep gzserver`), 있다면 강제 종료한다(`pkill -9 gzserver`).
  3. 설치가 손상되었을 수 있다. 섹션 2의 절차에 따라 완전히 재설치하는 것이 확실한 해결책이다.
- **문제:** `ERROR [gz_bridge] timed out waiting for clock message`.
- **진단:** PX4의 `gz_bridge` 모듈이 Gazebo에 연결되었지만, 시뮬레이션 시간 업데이트를 받지 못했다.
- **해결책:** 이는 종종 Gazebo 인스턴스가 일시 중지되었거나 충돌했음을 나타낸다. GUI에서 시뮬레이션이 '재생' 상태인지 확인한다. 또한 `GZ_IP`와 같은 환경 변수가 잘못 설정되지 않았는지 확인한다.


- **문제:** 기체가 시동(arm)은 걸리지만, 이륙 명령에 반응하지 않음.
- **진단 순서도:**
  1. **PX4 콘솔 확인:** `Preflight Fail` 메시지를 찾는다. 일반적인 실패 원인으로는 `Accel Sensor 0 missing`, `barometer 0 missing`과 같은 센서 누락이 있다. 이는 SDF 모델이나 센서 플러그인에 문제가 있음을 의미한다.
  2. **QGroundControl 확인:** 데이터링크가 연결되어 있는가? "no datalink" 안전장치가 발동하면 이륙을 막는다.
  3. **차량 물리(SDF) 확인:** 모델의 질량(`mass`)이 올바른가? 모터의 추력 상수(`motorConstant`)가 기체 무게에 비해 너무 낮은가?. 문제가 발생하면, 작동하는 기본 `x500` 모델에서 시작하여 점진적으로 수정하는 것이 좋다.
  4. **모터 매핑 확인:** SDF 파일에서 `gazebo_mavlink_interface` 플러그인이 제어 채널을 모터 조인트에 올바르게 매핑하고 있는지 확인한다. QGC의 MAVLink Inspector를 사용하여 액추에이터 출력이 전송되고 있는지 확인한다.
  5. **지면 글리치 확인:** 기체가 지면에 끼어있지 않은지 확인하기 위해 약간 위에서 스폰해본다 (`PX4_GZ_MODEL_POSE="0,0,0.5,0,0,0"`).
- **문제:** 기체가 이륙하지만 제어 불능 상태로 상승하거나 비정상적으로 동작함.
- **진단:** 이는 종종 잘못된 물리 스케일링으로 인해 악화된 컨트롤러 튜닝 문제다.
- **해결책:** 모터 플러그인의 `motorConstant` 및 `maxRotVelocity`, 그리고 mavlink 인터페이스 플러그인의 `input_scaling`이 매우 중요하다. PX4의 기본 컨트롤러 게인은 특정 추력 반응을 가정한다. 만약 시뮬레이션된 모터가 비정상적으로 강력하면 기체는 불안정해진다. 해결책은 호버링이 약 50% 스로틀에서 이루어질 때까지 이 SDF 파라미터들을 조정하는 것이다. VIO를 사용하는 경우, 비전 시스템과 PX4의 NED 좌표계 간의 좌표 변환 오류로 인해 발생할 수도 있다.


- **문제:** `ros2 topic list`에는 토픽이 보이지만 `ros2 topic echo`에 데이터가 출력되지 않음.
- **진단:** `ros_gz_bridge`가 메시지를 올바르게 중계하지 못하고 있다.
- **해결책:** 이는 알려진 문제다. 권장되는 해결책은 종종 사전 빌드된 브리지를 제거하고(`sudo apt-get remove ros-humble-ros-gz-bridge`), 자신의 워크스페이스 내에서 소스로부터 직접 빌드하는 것이다.
- **문제:** PX4 콘솔에 `multicast failed` 오류가 계속 출력됨.
- **진단:** 이는 DDS의 검색 메커니즘과 관련이 있다.
- **해결책:** 정확한 원인은 네트워크 관련(예: 불안정한 WiFi)일 수 있지만, 이는 종종 근본적인 시간 동기화 문제의 증상이다. 섹션 6.2에서 설명한 `use_sim_time` 문제를 해결하면 이러한 부차적인 오류들이 종종 함께 해결된다.