[ROS2 Humble](./index.md)
# ROS2 Humble 애플리케이션의 Balena 플랫폼 배포


**컨텍스트:** 사물 인터넷(IoT), 엣지 컴퓨팅, 로보틱스의 융합은 로봇 시스템의 개발 및 관리 방식에 근본적인 변화를 주도하고 있다. 단일체(monolithic) 구조의 수동으로 업데이트되던 전통적인 소프트웨어 모델은 컨테이너화되고 원격으로 관리되는 마이크로서비스로 대체되고 있으며, 이는 전례 없는 확장성과 민첩성을 가능하게 한다.1

**솔루션:** 본 보고서는 로보틱스 애플리케이션 개발의 사실상 표준인 Robot Operating System 2 (ROS2) Humble Hawksbill 3과, 커넥티드 디바이스 플릿(fleet)에 컨테이너화된 애플리케이션을 배포하고 관리하기 위해 설계된 포괄적인 도구 모음인 Balena 플랫폼 1의 강력한 조합을 심도 있게 분석한다.

**목표 및 범위:** 이 문서는 아키텍트와 엔지니어를 위한 철저하고 전문가 수준의 가이드를 제공한다. 단순한 튜토리얼을 넘어, 전체 배포 수명 주기에 대한 깊이 있는 "고찰"을 전달하는 것을 목표로 한다. 두 시스템의 기본 아키텍처, 컨테이너화 모범 사례, 복잡한 네트워킹 문제 해결 전략, 그리고 무선(Over-the-Air, OTA) 배포 및 플릿 관리를 위한 완전한 워크플로우를 다룰 것이다.



Balena 플랫폼은 원격 임베디드 환경의 핵심 과제인 신뢰성과 효율성을 해결하기 위해 상호 연결된 시스템으로 설계되었다. 각 구성 요소의 아키텍처 선택은 독립적인 기능이 아니라, 전체 플랫폼의 목표를 달성하기 위한 유기적인 결정의 결과이다.

- **BalenaOS:** 임베디드 장치에서 컨테이너를 안정적으로 실행한다는 단 하나의 목적을 위해 설계된 최소한의 Yocto 기반 호스트 운영 체제이다.6 `systemd`, NetworkManager, chrony와 같은 필수 서비스만 포함하며 7, 읽기 전용 루트 파일 시스템(`resin-rootA`/`resin-rootB`)을 사용하여 상태 비저장(stateless) 및 예측 가능한 작동을 보장한다.11 이러한 의도적인 호스트 OS의 미니멀리즘은 개발자의 전체 환경(라이브러리, 종속성, 코드)이 컨테이너 내에 패키징되는 강력한 컨테이너화 전략을 필수로 만든다. 이것이 바로 Balena의 핵심 철학이다.7
- **BalenaEngine:** Docker의 Moby 프로젝트를 기반으로 하는 특수 목적의 컨테이너 엔진이다.14 Swarm 및 오버레이 네트워킹과 같은 클라우드 중심 기능을 의도적으로 제거하여 Docker CE보다 3.5배 작은 바이너리 공간을 달성했다.14 더 중요한 것은 바이너리 델타 업데이트 및 최소 마모 스토리지 전략과 같은 IoT 특정 기능을 추가하여 Balena의 핵심 가치를 제공한다는 점이다.11 표준 Docker CE는 데이터 센터용으로 설계되었으므로 14, IoT에 컨테이너화를 실질적으로 적용하기 위해 Balena는 Moby를 포크하여 BalenaEngine을 만들어야 했다.16 이를 통해 불필요한 무게를 줄이고, 대역폭 효율성(델타 업데이트) 및 스토리지 수명(최소 마모 쓰기)과 같은 특정 IoT 문제점을 해결하는 기능을 추가할 수 있었다.14
- **BalenaCloud:** 중앙 집중식 클라우드 기반 관리 플랫폼이다.1 장치 프로비저닝, 코드 배포 및 플릿 모니터링을 조율하는 대시보드, API 및 빌드 서버를 제공한다.2 자체 호스팅 대안으로 OpenBalena도 존재한다.6


Supervisor는 모든 장치에서 실행되는 경량 컨테이너로, 장치와 BalenaCloud 간의 다리 역할을 한다.11 그 주요 역할은 다음과 같다:

1. **애플리케이션 상태 관리:** `docker-compose.yml`에 정의된 서비스가 올바르게 실행되도록 보장한다.11
2. **업데이트 오케스트레이션:** BalenaCloud API와 통신하여 새 릴리스를 감지하고, BalenaEngine을 사용하여 다운로드하며, 업데이트 프로세스를 관리한다.5
3. **API 및 로깅:** 장치 상태 쿼리를 위한 로컬 API를 제공하고 컨테이너 로그를 BalenaCloud 대시보드로 전달한다.5


BalenaOS는 무인 원격 장치에 매우 중요한 기능인 호스트 OS 업데이트를 위해 강력한 A/B 이중화 파티션 체계를 사용한다.11 이 프로세스는 `resin-rootA`(활성)와 `resin-rootB`(휴면/비활성)라는 두 개의 루트 파티션을 포함한다.11

- **업데이트 프로세스:** 호스트 OS 업데이트가 트리거되면 Supervisor는 새 OS 이미지를 비활성 파티션(`resin-rootB`)에 다운로드한다. 그런 다음 부트로더 설정(예: U-Boot 환경 변수 또는 GRUB 구성)을 수정하여 다음 재부팅 시 새 파티션에서 부팅하도록 한다.5
- **자동 롤백:** 이 시스템은 원자성(atomicity)을 위해 설계되었다. 새 OS 버전이 부팅에 실패하거나 상태 확인을 통과하지 못하면, 부트로더의 로직(또는 Balena의 커스텀 부트로더)은 다음 부팅 시도에서 이전에 정상 작동하던 파티션(`resin-rootA`)으로 자동 롤백한다.5 이를 통해 업데이트 실패 후에도 장치가 온라인 상태를 유지하고 작동할 수 있다.


애플리케이션 업데이트는 호스트 OS 업데이트와 구별되며 Supervisor와 BalenaEngine에 의해 관리된다.5 핵심 기술은 **바이너리 델타 업데이트**이다.14 BalenaEngine은 새로운 컨테이너 레이어 전체를 다운로드하는 대신, 현재 실행 중인 이미지와 새 이미지 간의 바이너리 차이점을 계산한다.

Supervisor는 이 작은 바이너리 패치(델타)만 다운로드하며, 이는 전체 레이어 풀보다 10-70배 작을 수 있다.14 이는 계량(예: 셀룰러) 또는 신뢰할 수 없는 네트워크에 있는 로봇의 대역폭 소비와 업데이트 시간을 극적으로 줄여준다. 

`docker-compose.yml`의 레이블을 통해 업데이트 전략(예: `download-then-kill`)을 구성하여 이전 서비스를 중지하고 새 서비스를 시작하는 정확한 순서를 제어함으로써 애플리케이션 다운타임을 최소화할 수 있다.5

이러한 델타 업데이트 메커니즘은 단순한 부가 기능이 아니라, Balena가 대상 사용 사례에 대한 표준 Docker의 단점을 파악하고 해결한 직접적인 결과이다. 이는 결과적으로 애플리케이션 내의 아키텍처 선택(예: 섹션 2에서 논의될 다단계 빌드 사용)을 단순한 "모범 사례"가 아닌, Balena 핵심 OTA 기술의 효율성을 극대화하는 중요한 단계로 만든다. 최종 이미지가 작을수록 잠재적 델타가 작아지고 전체 플랫폼이 더욱 효과적이 된다. 따라서 Balena에 배포하는 엔지니어는 이를 단순한 "Docker가 설치된 Linux"가 아닌 전체적인 플랫폼으로 생각해야 한다.

| 기능                  | 표준 Docker CE         | BalenaEngine                             | ROS2 배포에 대한 시사점                                      |
| --------------------- | ---------------------- | ---------------------------------------- | ------------------------------------------------------------ |
| **바이너리 크기**     | 상대적으로 큼          | 3.5배 더 작음 14                         | 리소스가 제한된 임베디드 장치에서 더 적은 저장 공간을 차지하고 시스템 오버헤드를 줄임. |
| **업데이트 메커니즘** | 전체 레이어 풀(Pull)   | 바이너리 델타(10-70배 효율적) 14         | 셀룰러/Wi-Fi 등 대역폭이 제한된 로봇의 OTA 업데이트 비용과 시간을 극적으로 절감. |
| **스토리지 I/O**      | 표준 쓰기 방식         | 최소 마모(Minimal wear-and-tear) 쓰기 14 | SD 카드와 같은 플래시 스토리지의 수명을 연장하여 장기적인 신뢰성을 향상시킴. |
| **네트워킹**          | 오버레이 네트워킹 포함 | 오버레이 드라이버 제외 14                | IoT에 불필요한 복잡성을 제거하고, ROS2에 필요한 `host` 모드 네트워킹에 집중. |
| **오케스트레이션**    | Docker Swarm 포함      | Docker Swarm 제외 14                     | 경량화를 위해 불필요한 기능을 제거. Balena는 Supervisor를 통해 자체 오케스트레이션을 제공. |
| **초점**              | 데이터 센터, 서버      | 임베디드, IoT, 엣지 14                   | 로보틱스 사용 사례의 고유한 요구 사항(신뢰성, 효율성, 원격 관리)에 최적화됨. |
|                       |                        |                                          |                                                              |

**표 1: 로보틱스를 위한 BalenaEngine 대 표준 Docker 엔진 비교**



- **기본 이미지 선택:** 컨테이너의 기초이다. 공식 `osrf/ros:humble-desktop` 또는 `osrf/ros:humble-ros-base` 이미지를 사용할 수 있지만 20, Balena에서의 프로덕션을 위해서는 

  `balenalib` 기반 이미지(예: `balenalib/%%BALENA_MACHINE_NAME%%-ubuntu:jammy`)로 시작하는 것이 좋다.22 이는 Balena 환경 및 빌드 시스템과의 호환성을 보장한다.

- **ROS2 설치:** Dockerfile에는 다음 단계가 포함되어야 한다:

  1. 로케일 및 `DEBIAN_FRONTEND=noninteractive` 설정.22
  2. ROS2 apt 저장소 및 키 추가.22
  3. 필요한 ROS2 패키지(예: `ros-humble-ros-base`, `ros-dev-tools` 및 애플리케이션 특정 패키지) `apt-get install`.22
  4. `colcon` 및 기타 Python 빌드 종속성 설치.23


Dockerfile은 사용자의 커스텀 ROS2 노드를 빌드하기 위한 워크플로우를 정의해야 한다.

- **소스 코드:** `WORKDIR /ros2_ws`를 사용하여 작업 디렉토리를 설정하고 `COPY./src./src`를 사용하여 애플리케이션의 소스 코드를 이미지에 복사한다.24

- **종속성:** `rosdep install`을 실행하여 `package.xml` 파일에 나열된 모든 패키지 종속성을 자동으로 설치한다.23

  `rosdep install -i --from-path src --rosdistro humble -y`가 표준 명령어이다.

- **빌드:** `colcon build`를 실행하여 워크스페이스를 컴파일한다.23

- **소싱(Sourcing):** 컨테이너의 엔트리포인트 또는 명령어는 노드를 시작하기 전에 ROS2 환경(`source /opt/ros/humble/setup.bash`)과 로컬 워크스페이스(`source /ros2_ws/install/setup.bash`)를 소싱해야 한다.23 이는 종종 커스텀 `entrypoint.sh` 스크립트에서 처리된다.


이는 Balena에서의 프로덕션 배포를 위한 중요한 최적화 기법이다.27 빌드 환경과 최종 런타임 환경을 분리하여 훨씬 작고 안전한 이미지를 만든다. 이 패턴은 단순한 Docker "모범 사례"가 아니라, Balena의 핵심 OTA 가치 제안을 가능하게 하는 근본적인 요소이다.

Balena의 주요 장점은 효율적인 바이너리 델타 OTA 업데이트이다.14 이 델타의 효율성은 이전 이미지와 새 이미지 간의 유사성에 비례하고 전체 이미지 크기에 반비례한다. 단일 단계 빌드 프로세스는 이미지에 많은 "노이즈"를 유발한다. 작은 코드 변경이 

`rosdep install`이나 다른 패키지 관리자 명령을 트리거하여 실제 애플리케이션 로직과 관련 없는 많은 레이어와 파일을 변경할 수 있다. 이는 크고 비효율적인 델타를 초래한다.

다단계 빌드는 애플리케이션을 빌드 환경에서 격리한다. 최종 `production` 단계에는 컴파일된 애플리케이션(`/ros2_ws/install`)과 최소한의 런타임 종속성만 포함된다. 개발자가 코드 변경을 푸시하면 `builder` 단계만 다시 컴파일된다. `COPY --from=builder` 단계는 새롭고 작은 컴파일된 바이너리 세트를 `production` 단계로 전송한다. 프로덕션 이미지의 기본 OS 레이어는 변경되지 않은 상태로 유지된다. 이는 OTA로 전송되는 바이너리 델타가 빌드 프로세스의 노이즈가 아닌 거의 순수하게 애플리케이션의 컴파일된 코드 변경 사항임을 의미한다. 이는 BalenaEngine의 핵심 기능 효율성을 극대화한다. 따라서 다단계 빌드를 사용하지 않는 것은 Balena 플랫폼의 모든 기능을 활용하지 못하는 것과 같으며, 이는 더 크고, 느리고, 더 비싼 OTA 업데이트로 이어져 애초에 Balena를 선택한 핵심 이유를 약화시킨다.

- **`builder` 단계:**
  - `FROM balenalib/%%BALENA_MACHINE_NAME%%-ubuntu:jammy AS builder`
  - 이 단계에서는 `build-essential`, `ros-dev-tools`, `colcon` 등 모든 빌드 시간 종속성을 설치한다.22
  - 소스 코드를 복사하고, `rosdep`을 실행하고, `colcon build`를 실행한다.
- **`production` 단계:**
  - `FROM balenalib/%%BALENA_MACHINE_NAME%%-ubuntu:jammy` (또는 더 작은 베이스 이미지).
  - 이 단계에서는 ROS2 애플리케이션의 *런타임* 종속성만 설치한다.
  - 결정적으로, `COPY --from=builder /ros2_ws/install /ros2_ws/install`을 사용하여 `builder` 단계에서 컴파일된 아티팩트*만* 복사한다. 필요한 런타임 자산(런치 파일, 구성 파일)도 복사한다.
  - 이 단계에는 컴파일러, 빌드 도구 또는 소스 코드가 포함되지 않아 가볍고 안전하다.
- **Balena 통합:** `docker-compose.yml`은 `target` 키워드를 사용하여 빌드할 단계를 지정할 수 있어, 로컬 개발을 위한 `dev` 단계와 클라우드 빌드를 위한 `prod` 단계를 설정할 수 있다.28


Balena의 `docker-compose.yml` 파일은 단순한 구성 파일을 넘어, 애플리케이션과 그 기반이 되는 BalenaOS 및 Supervisor와의 관계를 정의하는 계약서와 같다. 표준 Docker Compose 지식은 필요하지만 충분하지 않다. 개발자는 "Docker Compose 사용"에서 "Compose와 유사한 매니페스트를 통해 Balena Supervisor 구성"으로 사고방식을 전환해야 한다. 이는 Balena 특정 레이블(예: `io.balena.features.balena-socket`)을 수용하고 29, Balena의 변수 관리 시스템을 사용하며, 파일의 주요 소비자가 표준 Docker 데몬이 아닌 Balena Supervisor임을 이해하는 것을 의미한다.


Balena의 다중 컨테이너 기능은 Docker Compose v2.1/v2.4 파일 형식을 기반으로 한다.19 각 ROS2 노드 또는 논리적 노드 그룹은 `docker-compose.yml` 파일에서 별도의 `service`로 정의되어야 한다.19 이는 모듈성, 격리성을 촉진하고 서비스의 독립적인 업데이트를 가능하게 한다. 

`depends_on`을 사용하여 서비스 시작 순서를 제어할 수 있지만, 런타임에 종속성이 재시작되는 것을 처리하지는 않는다는 점에 유의해야 한다.19

```YAML
version: '2.1'
services:
  camera_driver:
    build:./camera_driver_pkg
    #... hardware access...
  perception_node:
    build:./perception_pkg
    depends_on:
      - camera_driver
  control_node:
    build:./control_pkg
    #...
```


- **근본 원인:** ROS2는 통신을 위해 데이터 분산 서비스(DDS)에 의존한다.3 대부분의 DDS 구현(Fast DDS, CycloneDDS 등)의 기본 검색 메커니즘은 다른 노드를 찾기 위해 UDP 멀티캐스트를 사용한다.31 Docker의 기본 

  `bridge` 네트워크는 컨테이너를 격리하고 일반적으로 컨테이너와 호스트 간의 멀티캐스트 트래픽을 차단하거나 방해한다.

- **표준 솔루션: `network_mode: "host"`**

  - 이는 Docker/Balena에서 ROS2를 위한 가장 일반적이고 권장되는 접근 방식이다.24
  - `docker-compose.yml`의 모든 ROS2 서비스에 `network_mode: "host"`를 설정하면 네트워크 격리가 제거된다. 모든 컨테이너는 호스트의 네트워크 스택을 공유하여, 마치 네이티브 프로세스처럼 멀티캐스트를 사용하고 서로를 원활하게 검색할 수 있다.24
  - **시사점:** 이는 기능적 통신을 위한 가장 간단한 경로이지만, 네트워크 격리의 보안 이점을 희생한다. 모든 컨테이너 포트는 호스트의 네트워크 인터페이스에 노출된다.

- **고급 대안: 유니캐스트 DDS 구성**

  - `host` 네트워킹이 바람직하지 않은 환경(예: 보안상의 이유)에서는 유니캐스트 검색이 대안이 될 수 있다. 이는 훨씬 더 복잡하다.

  - 멀티캐스트를 비활성화하고 각 DDS 참가자에게 피어의 IP 주소를 명시적으로 알려주는 작업이 포함된다.

  - **CycloneDDS 예시:** `cyclonedds.xml` 구성 파일을 생성해야 한다.31 이 파일에는 

    `<AllowMulticast>false</AllowMulticast>`와 다른 노드의 주소가 포함된 `<Peers>` 목록이 포함된다.31 이 XML 파일은 컨테이너에 마운트되어야 하며, 

    `CYCLONEDDS_URI` 환경 변수가 이를 가리켜야 한다.36

  - **시사점:** 이 접근 방식은 IP 주소 또는 호스트 이름 및 구성 파일의 수동 관리가 필요하여, 특히 동적 플릿에서 상당한 운영 오버헤드를 추가한다. 보안 요구 사항이 호스트 네트워킹을 엄격하게 금지하지 않는 한 일반적으로 피한다.

- **환경 변수:** 통신해야 하는 모든 컨테이너에서 `ROS_DOMAIN_ID` 환경 변수가 일관되어야 한다.32 일부 구성에서는 

  `ROS_LOCALHOST_ONLY=0`이 필요할 수도 있다.39

| 전략                     | `docker-compose.yml` 구성             | 작동 방식                                                    | 장점                                                         | 단점                                                         | 최적 사용 사례                                               |
| ------------------------ | ------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **호스트 모드**          | `network_mode: "host"`                | 컨테이너가 호스트의 네트워크 스택을 직접 공유하여 네트워크 격리를 제거함. | 구성이 매우 간단하고, DDS 멀티캐스트 검색이 즉시 작동함. 최고의 성능을 제공함. | 컨테이너 네트워크 격리가 없어 보안에 취약할 수 있음. 포트 충돌 가능성이 있음. | 대부분의 Balena 기반 ROS2 배포. 특히 단일 목적의 임베디드 장치에 적합함. |
| **브리지 모드 (기본값)** | (명시적 구성 없음)                    | Docker가 가상 브리지 네트워크를 생성하고 각 컨테이너에 사설 IP를 할당함. | 컨테이너 간 네트워크 격리로 보안이 강화됨.                   | DDS 멀티캐스트가 기본적으로 작동하지 않아 노드 검색이 실패함. 복잡한 포트 포워딩이 필요함. | ROS2에는 거의 사용되지 않으며, DDS가 아닌 다른 프로토콜을 사용하는 서비스에 적합함. |
| **유니캐스트 구성**      | `environment:` `- CYCLONEDDS_URI=...` | 멀티캐스트를 비활성화하고, 각 노드가 연결할 다른 노드의 IP 주소를 정적으로 지정함. | 네트워크 격리를 유지하면서 통신이 가능함.                    | 구성이 매우 복잡하고, IP 주소를 수동으로 관리해야 함. 확장성이 떨어짐. | 엄격한 보안 요구사항으로 호스트 네트워킹이 금지된 환경.      |

**표 2: Docker/Balena에서의 ROS2 네트워킹 전략 비교**


Balena의 다중 컨테이너 서비스는 기본적으로 권한 없는(non-privileged) 모드에서 실행된다.40 카메라, GPIO, I2C, 직렬 포트와 같은 하드웨어에 대한 접근은 `docker-compose.yml`에서 명시적으로 부여해야 한다.

- **방법:**
  1. **`privileged: true`**: 가장 간단한 방법. 컨테이너에 호스트의 모든 장치(`/dev`)에 대한 전체 접근 권한을 부여한다. 복잡한 하드웨어 상호 작용이 필요할 때 사용하지만 보안 영향을 인지해야 한다.40
  2. **`devices`**: 특정 하드웨어에 대한 선호되는 방법. 호스트 장치 파일을 컨테이너에 직접 매핑한다(예: 카메라의 경우 `/dev/video0`, I2C 버스의 경우 `/dev/i2c-1`).40
  3. **`cap_add`**: 특정 Linux 기능을 추가한다. `SYS_RAWIO`는 일부 GPIO 라이브러리에 필요한 직접 메모리 접근에 종종 필요하다.40

| 지시어             | 예시                                     | 사용 사례                                                    | 접근 수준                                                    |
| ------------------ | ---------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `privileged: true` | `privileged: true`                       | 여러 복합적인 하드웨어(카메라, GPIO, SPI 등)에 동시에 접근해야 할 때. | **최상:** 컨테이너에 호스트의 모든 장치에 대한 루트 수준의 접근 권한을 부여함. |
| `devices`          | `devices:` `- "/dev/video0:/dev/video0"` | 특정 장치 파일에 대한 접근이 필요할 때 (예: USB 카메라, I2C 버스, 시리얼 포트). | **중간:** 명시된 장치 파일에 대해서만 접근을 허용하여 권한을 최소화함. |
| `cap_add`          | `cap_add:` `- SYS_RAWIO`                 | `mmap`이나 I/O 포트 접근과 같이 특정 커널 기능이 필요할 때 (예: 일부 GPIO 라이브러리). | **최소:** 장치 파일 접근 대신 특정 시스템 호출 권한만 부여함. `devices`와 함께 사용되는 경우가 많음. |

**표 3: `docker-compose.yml`의 하드웨어 접근 구성**


맵, 보정 파일, 로그, 데이터베이스와 같은 데이터가 컨테이너 업데이트나 장치 재부팅 후에도 유지되도록 하려면 Balena는 **명명된 볼륨(named volumes)**을 사용한다.19

- **구현:**

  1. `docker-compose.yml` 파일의 최상위 수준에서 명명된 볼륨을 선언한다. 일반적인 관례는 `resin-data`이다.12
  2. 서비스 정의에서 `volumes` 키를 사용하여 명명된 볼륨을 컨테이너 내부의 경로에 마운트한다.

- **예시 (`docker-compose.yml`):**

  ```YAML
  version: '2.1'
  volumes:
    ros-data: # 명명된 볼륨 선언
  services:
    mapping_node:
      build:./mapping_pkg
      volumes:
        - 'ros-data:/data/maps' # 'ros-data'를 컨테이너의 '/data/maps'에 마운트
    localization_node:
      build:./localization_pkg
      volumes:
        - 'ros-data:/data/maps' # 동일한 볼륨을 공유할 수 있음
  ```

- 이제 `ros-data` 볼륨의 데이터는 호스트 OS의 `resin-data` 파티션(`/mnt/data/docker/volumes/...`)에 영구적으로 저장되며 애플리케이션 업데이트 후에도 유지된다.12


Balena CLI는 두 가지 주요 엔지니어링 단계, 즉 신속하고 반복적인 개발과 확장 가능하고 강력한 배포에 직접적으로 매핑되는 두 가지 별개의 목적별 워크플로우(`local push` 대 `cloud push`)를 제공한다. 언제 어떤 도구를 사용해야 하는지 이해하는 것이 개발자 생산성의 핵심이다.


신속한 개발을 위해 Balena는 클라우드 빌더를 우회하는 "로컬 모드"를 제공한다.43

- **워크플로우:**

  1. BalenaCloud 대시보드를 통해 장치에서 "로컬 모드"를 활성화한다.

  2. `balena scan` 또는 `balena device detect`를 사용하여 장치의 로컬 IP 주소를 찾는다.8

  3. `balena push <device-ip>`(예: `balena push 192.168.1.5`)를 사용하여 프로젝트 소스를 장치에 직접 푸시한다.43 빌드는 

     *장치에서* 이루어진다.

  4. 이 명령어는 로그를 터미널로 스트리밍한다. `Ctrl+C`는 로그 스트림을 중지하지만 서비스는 장치에서 계속 실행된다.43

- **Livepush:** 로컬 모드의 핵심 기능이다. 소스 코드 변경(Dockerfile 변경 아님) 시 Balena는 수정된 파일을 실행 중인 컨테이너에 직접 주입하고 프로세스를 다시 시작하여 전체 재빌드를 피할 수 있다. 이는 거의 즉각적인 피드백 루프를 제공한다.43


이는 장치 플릿을 업데이트하기 위한 표준 워크플로우이다.

- `balena push <fleet-name>`(예: `balena push my-ros-fleet`) 명령어는 프로젝트 소스 코드를 Balena의 클라우드 빌드 서버로 보낸다.45
- 빌더는 플릿의 아키텍처에 맞는 컨테이너 이미지를 생성한다. 완료되면 새 "릴리스"가 생성된다.
- 플릿의 장치들은 새 릴리스를 감지하고 섹션 1.4에서 설명한 대로 바이너리 델타를 다운로드하여 OTA 업데이트 프로세스를 시작한다.


Balena CLI는 원격 장치 관리를 위한 필수 도구이다.43

- **로그 보기:** `balena logs <device-uuid>`를 사용하여 원격 장치에서 로그를 스트리밍한다.

  - `--service <service-name>`는 특정 서비스에 대한 필터링을 한다.43
  - `--system`은 호스트 OS 로그를 표시한다.43

- **원격 셸 접근 (SSH):**

  - SSH 공개 키가 BalenaCloud 계정에 추가되었는지 확인한다.43

  - `balena ssh <device-uuid>`는 호스트 BalenaOS에 대한 셸을 제공한다.43 이는 저수준 디버깅에 매우 유용하다.

  - `balena ssh <device-uuid> <service-name>`는 실행 중인 서비스 컨테이너 내부에 직접 셸을 제공한다(예: `balena ssh <uuid> camera_node`).43 이를 통해 라이브 장치에서 

    `ros2 topic list`, `ros2 node info` 등을 실행할 수 있다.

| 명령어                        | 예시                                                         | 사용 사례                                                    | 단계 |
| ----------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| `balena login`                | `balena login --credentials`                                 | BalenaCloud에 CLI를 인증함.                                  | 설정 |
| `balena scan`                 | `sudo balena scan`                                           | 로컬 네트워크에서 개발 모드 장치를 찾음.                     | 개발 |
| `balena push <ip>`            | `balena push 192.168.1.10.local`                             | 코드를 로컬 장치에 직접 푸시하여 빠른 개발 및 테스트를 수행함. | 개발 |
| `balena push <fleet>`         | `balena push my-ros-fleet`                                   | 코드를 클라우드 빌더에 푸시하여 플릿 전체에 릴리스를 배포함. | 배포 |
| `balena logs <uuid>`          | `balena logs 1a2b3c4 --service perception`                   | 원격 장치의 특정 서비스에서 로그를 스트리밍하여 디버깅함.    | 관리 |
| `balena ssh <uuid> [service]` | `balena ssh 1a2b3c4 camera_node`                             | 원격 장치의 호스트 OS나 특정 컨테이너에 접속하여 실시간으로 문제를 해결함. | 관리 |
| `balena fleet create`         | `balena fleet create new-fleet --type raspberrypi4-64`       | 새 장치 그룹(플릿)을 생성함.                                 | 설정 |
| `balena env add <var>`        | `balena env add ROS_DOMAIN_ID --fleet my-ros-fleet --value 5` | 플릿 또는 장치에 환경 변수를 추가하여 원격으로 애플리케이션을 구성함. | 관리 |
|                               |                                                              |                                                              |      |

**표 4: ROS2 배포를 위한 필수 Balena CLI 명령어**


작동하는 프로토타입에서 프로덕션 준비 시스템으로 전환하려면 "작동하게 만들기"에서 "우아하게 실패하고 대규모로 관리 가능하게 만들기"로 초점을 전환해야 한다.


리소스가 제한된 임베디드 장치(예: Raspberry Pi, Jetson Nano)에서는 시스템 불안정을 방지하기 위해 CPU 및 메모리 관리가 중요하다.51 Balena는 

`docker-compose.yml`에서 Docker의 리소스 제약 옵션을 지원하지만, 구문은 Compose v2.1에 특정될 수 있다.53

- **메모리:** `mem_limit` 및 `mem_reservation`을 사용하여 각 서비스에 대한 하드 및 소프트 메모리 제한을 설정한다.

- **CPU:** `cpu_quota`, `cpu_period` 또는 `cpus`를 사용하여 CPU 사용량을 제한한다.53 다중 코어 장치의 경우, 

  `cpuset`을 사용하여 고강도 서비스를 특정 코어에 고정하여 다른 중요한 프로세스를 방해하지 않도록 할 수 있다.51

- **예시 (`docker-compose.yml`):**

  ```YAML
  services:
    intensive_node:
      build:./intensive_pkg
      mem_limit: 512m
      cpuset: "0" # CPU 코어 0에 고정
  ```


구성(IP 주소, 보정 데이터, API 키)을 Dockerfile에 하드코딩하는 것은 나쁜 관행이다. Balena는 구성을 위한 강력한 변수 시스템을 제공한다.46 변수는 플릿 수준(모든 장치에 적용) 또는 장치 수준(특정 장치에 대한 플릿 변수 재정의)에서 설정할 수 있다. 이러한 변수는 런타임에 모든 서비스 컨테이너에 환경 변수로 주입된다.

- **ROS2 통합:** 이는 ROS2 매개변수를 전달하는 이상적인 방법이다. ROS2 런치 파일은 이러한 환경 변수를 읽어 노드를 동적으로 구성할 수 있다.
- **관리:** 변수는 BalenaCloud 대시보드 또는 `balena env` 및 `balena device env` CLI 명령을 통해 관리할 수 있다.49


- **권한 최소화:** 가능한 한 `privileged: true`에서 벗어나라. 더 세분화된 `devices` 및 `cap_add`를 사용하여 필요한 권한만 부여하라.40
- **경량 이미지:** 다단계 빌드를 사용하여 최종 프로덕션 이미지에 빌드 도구, 컴파일러 또는 불필요한 패키지가 포함되지 않도록 하여 공격 표면을 줄여라.27
- **ROS2 보안:** 민감한 애플리케이션의 경우 ROS2의 내장 보안 기능(SROS2)을 활용하라. 이는 DDS 트래픽에 대한 인증, 암호화 및 접근 제어를 활성화하기 위해 키 저장소와 인클레이브를 생성하는 것을 포함한다. 이는 필요한 보안 아티팩트(키 및 인증서)를 안전하게 관리하고 배포함으로써 Balena 배포와 통합될 수 있다.57


- **네트워킹 문제:**

  - **증상:** 다른 컨테이너의 노드가 서로를 볼 수 없음 (`ros2 node list`가 로컬 노드만 표시).

  - **원인:** `network_mode: "host"`가 없거나 잘못되었거나, 방화벽이 멀티캐스트를 차단하는 경우가 대부분이다.

  - **해결책:** `docker-compose.yml`에서 `network_mode: "host"`를 확인하라. 호스트 OS에서 `ros2 multicast receive/send`를 사용하여 멀티캐스트 기능을 확인하라.33 일관된 

    `ROS_DOMAIN_ID`를 확인하라.38

- **컨테이너 시작 실패 / 재시작 루프:**

  - **증상:** Balena 대시보드에서 서비스가 "restarting" 상태에 멈춰 있음.
  - **원인:** 컨테이너의 주 프로세스가 충돌하고 있다. 이는 ROS2 노드의 버그, 누락된 종속성 또는 잘못된 구성일 수 있다.
  - **해결책:** `balena logs <uuid> --service <name>`을 사용하여 오류 메시지를 확인하라. `balena ssh <uuid> <name>`을 사용하여 실패하는 컨테이너 내부에서 셸을 얻고 애플리케이션을 수동으로 실행하여 오류를 확인하라.

- **하드웨어 접근 불가:**

  - **증상:** `/dev/i2c-1` 또는 `/dev/video0`과 같은 장치 파일을 열려고 할 때 코드가 "permission denied"로 실패한다.
  - **원인:** 서비스가 `docker-compose.yml`에 필요한 권한이 없다.
  - **해결책:** 해당 서비스에 적절한 `devices`, `cap_add` 또는 `privileged: true` 설정을 추가하라.

- **빌드 실패:**

  - **증상:** `balena push`가 빌드 과정에서 실패한다.
  - **원인:** 누락된 종속성(`rosdep` 실패), 컴파일 오류(`colcon build` 실패) 또는 Dockerfile의 구문 오류.
  - **해결책:** `balena push`에서 제공하는 빌드 로그를 주의 깊게 읽어라. 오류 메시지는 일반적으로 실패한 정확한 명령을 가리킨다.


- **요약:** 아키텍처 원칙에서부터 완전히 구현된 배포 전략까지의 여정을 요약한다. Balena의 IoT 중심 플랫폼과 ROS2의 로보틱스 프레임워크의 조합이 올바르게 사용될 때, 진정으로 현대적이고 확장 가능하며 원격으로 관리 가능한 로보틱스 제품의 생성을 가능하게 한다는 점을 강조한다.
- **핵심 전략 재검토:**
  1. **플랫폼을 위한 아키텍처 설계:** Balena의 델타 업데이트 효율성을 극대화하기 위해 경량의 다단계 컨테이너를 설계하라.
  2. **네트워킹 우선 해결:** ROS2 DDS 통신을 위한 실용적인 기본값으로 `network_mode: "host"`를 사용하라.
  3. **매니페스트를 계약으로:** `docker-compose.yml`을 서비스, 하드웨어 접근, 데이터 지속성 및 리소스 제한을 정의하는 중앙 계약으로 취급하라.
  4. **CLI 워크플로우 수용:** 최대의 개발 속도와 배포 견고성을 위해 별개의 로컬 및 클라우드 푸시 워크플로우를 사용하라.
  5. **코딩 대신 구성:** 플릿 확장성을 가능하게 하기 위해 모든 구성에 Balena의 환경 변수를 활용하라.
- **최종 권장 사항:** Balena에서 ROS2 Humble 애플리케이션의 성공적인 배포는 로보틱스 엔지니어링과 현대 DevOps의 실천이다. 컨테이너화, 코드로서의 인프라(`docker-compose.yml`을 통해), 그리고 규율 있는 릴리스 관리 원칙을 수용함으로써, 팀은 단일 장치 프로토타입을 넘어 현장에서 견고한 로봇 플릿을 구축하고 유지 관리할 수 있다.


1. balena | InfluxData, accessed July 24, 2025, https://www.influxdata.com/partners/balena/
2. Balena balenaCloud Solution Brief - Intel® Network Builders, accessed July 24, 2025, https://builders.intel.com/docs/networkbuilders/mastering-iot-deployment-and-device-management-at-the-edge-with-balenacloud-balenacloud-1717495952.pdf
3. Basic Concepts - ROS 2 Documentation: Humble documentation, accessed July 24, 2025, https://docs.ros.org/en/humble/Concepts/Basic.html
4. Tutorials - ROS 2 Documentation: Humble documentation, accessed July 24, 2025, https://docs.ros.org/en/humble/Tutorials.html
5. Core Concepts | balena, accessed July 24, 2025, https://docs.balena.io/learn/welcome/primer/
6. Run containers on embedded edge devices - balenaOS, accessed July 24, 2025, https://www.balena.io/os
7. balenaOS: Introduction, accessed July 24, 2025, https://balena-os.pages.dev/
8. BalenaOS Installation | Seeed Studio Wiki, accessed July 24, 2025, https://wiki.seeedstudio.com/BalenaOS-X86-Getting-Started/
9. DIDO Wiki - balenaOS, accessed July 24, 2025, https://www.omgwiki.org/dido/doku.php?id=dido:public:ra:xapend:xapend.d_opsys:balenaos:start
10. balena-os/meta-balena: A collection of Yocto layers used to build balenaOS images - GitHub, accessed July 24, 2025, https://github.com/balena-os/meta-balena
11. Architecture Overview | balenaOS, accessed July 24, 2025, https://balena-os.pages.dev/architecture/
12. What is balenaOS?, accessed July 24, 2025, https://docs.balena.io/reference/OS/overview/
13. Frequently Asked Questions (FAQ's) - balenaOS, accessed July 24, 2025, https://balena-os.pages.dev/faqs/
14. balenaEngine - A container engine purpose-built for IoT devices, accessed July 24, 2025, https://www.balena.io/engine
15. matrix-io/balena: Moby-based Container Engine for Embedded, IoT, and Edge uses - GitHub, accessed July 24, 2025, https://github.com/matrix-io/balena
16. Device fleets are incredibly hard to get right, and if you have no updateability... | Hacker News, accessed July 24, 2025, https://news.ycombinator.com/item?id=18753734
17. Balena's Efficient OTA Updates: A Technical Deep Dive - YouTube, accessed July 24, 2025, https://m.youtube.com/shorts/0JmZLInmacs
18. Update process details - balena docs, accessed July 24, 2025, https://docs.balena.io/reference/OS/updates/update-process/
19. Multiple containers - balena docs, accessed July 24, 2025, https://docs.balena.io/learn/develop/multicontainer/
20. rticommunity/rticonnextdds-ros2-docker: Docker images for ROS 2 development with RTI Connext DDS - GitHub, accessed July 24, 2025, https://github.com/rticommunity/rticonnextdds-ros2-docker
21. Running ROS 2 nodes in Docker [community-contributed] - ROS 2 ..., accessed July 24, 2025, https://docs.ros.org/en/foxy/How-To-Guides/Run-2-nodes-in-single-or-separate-docker-containers.html
22. balena-ros2-foxy-base/Dockerfile at main - GitHub, accessed July 24, 2025, https://github.com/balena-io-examples/balena-ros2-foxy-base/blob/main/Dockerfile
23. ysimonov/ros2_humble_examples: ros2 humble simple examples and environment configuration C++ - GitHub, accessed July 24, 2025, https://github.com/ysimonov/ros2_humble_examples
24. The Complete Beginner's Guide to Docker for ROS 2 Deployment ..., accessed July 24, 2025, https://blog.robotair.io/the-complete-beginners-guide-to-using-docker-for-ros-2-deployment-2025-edition-0f259ca8b378
25. Docker Setup for ROS 2 - Robotics SDK Documentation - Texas Instruments, accessed July 24, 2025, https://software-dl.ti.com/jacinto7/esd/robotics-sdk/08_01_00/docs/source/docker/setting_docker_ros2.html
26. The Complete Guide to Docker for ROS 2 Jazzy Projects - Automatic Addison, accessed July 24, 2025, https://automaticaddison.com/the-complete-guide-to-docker-for-ros-2-jazzy-projects/
27. Best Practices for Deploying a Production-Ready ROS2 Robot : r/ROS - Reddit, accessed July 24, 2025, https://www.reddit.com/r/ROS/comments/1idvrr4/best_practices_for_deploying_a_productionready/
28. balena-io-examples/balena-multistage-dockerfile-example ... - GitHub, accessed July 24, 2025, https://github.com/balena-io-examples/balena-multistage-dockerfile-example
29. docker-compose.yml fields - balena docs, accessed July 24, 2025, https://docs.balena.io/reference/supervisor/docker-compose/
30. Automatic Deployment of ROS2 Based System to remote devices: Dual Copy or Containers?, accessed July 24, 2025, https://discourse.ros.org/t/automatic-deployment-of-ros2-based-system-to-remote-devices-dual-copy-or-containers/33884
31. ROS2 foxy Eclipse Cyclone DDS communication between Docker ubuntu container and Windows host / Issue #677 - GitHub, accessed July 24, 2025, https://github.com/eclipse-cyclonedds/cyclonedds/issues/677
32. Installation troubleshooting - ROS 2 Documentation: Foxy documentation, accessed July 24, 2025, https://docs.ros.org/en/foxy/How-To-Guides/Installation-Troubleshooting.html
33. Installation troubleshooting - ROS 2 Documentation: Rolling documentation, accessed July 24, 2025, https://docs.ros.org/en/rolling/How-To-Guides/Installation-Troubleshooting.html
34. Devices in Docker - Articulated Robotics, accessed July 24, 2025, https://articulatedrobotics.xyz/tutorials/docker/devices-docker/
35. ROS 1 Bridge running in docker container not receiving published topics from ROS 2 humble - ROS Answers, accessed July 24, 2025, https://answers.ros.org/question/415606/
36. Connecting ROS 2 Nodes with the Custom CycloneDDS XML Config File | Husarnet, accessed July 24, 2025, https://husarnet.com/docs/ros2/custom-cyclonedds-xml/
37. Connecting Remote Robots Using ROS2, Docker & VPN - Husarnet, accessed July 24, 2025, https://husarnet.com/blog/ros2-docker/
38. Problems with BlueOS and ROS2 for BlueBoat - Blue Robotics Community Forums, accessed July 24, 2025, https://discuss.bluerobotics.com/t/problems-with-blueos-and-ros2-for-blueboat/20258
39. Setting up Rosbridge in a docker container - Dan Aukes, accessed July 24, 2025, https://www.danaukes.com/notebook/ros2/41-setting-up-rosbridge-on-docker/
40. Interact with hardware | balena, accessed July 24, 2025, https://docs.balena.io/learn/develop/hardware/
41. Activating GPIO remotely - Project help - balena Forums, accessed July 24, 2025, https://forums.balena.io/t/activating-gpio-remotely/369494
42. docs/shared/general/persistent-storage.md at master / balena-io/docs - GitHub, accessed July 24, 2025, https://github.com/balena-io/docs/blob/master/shared/general/persistent-storage.md
43. balena-io/balena-cli-masterclass: A guide to getting started ... - GitHub, accessed July 24, 2025, https://github.com/balena-io/balena-cli-masterclass
44. raspberrypi3 - balenaOS, accessed July 24, 2025, [https://balena-os.pages.dev/Getting%20Started/raspberrypi3](https://balena-os.pages.dev/Getting Started/raspberrypi3)
45. balena ROS2 Foxy Base container - GitHub, accessed July 24, 2025, https://github.com/balena-io-examples/balena-ros2-foxy-base
46. Balena Deployment | OpenRemote Documentation, accessed July 24, 2025, https://docs.openremote.io/docs/user-guide/deploying/balena-deployment/
47. Installing Balena-CLI on Windows - YouTube, accessed July 24, 2025, https://www.youtube.com/watch?v=2LApclXFqsg
48. balena CLI and Docker - GitHub, accessed July 24, 2025, https://github.com/balena-io-experimental/balena-cli-docker
49. balena CLI Documentation, accessed July 24, 2025, https://docs.balena.io/reference/balena-cli/latest/
50. Upload a file to Balena container through the scp command | by Ats - Medium, accessed July 24, 2025, https://atsss.medium.com/upload-a-file-to-balena-container-through-the-scp-command-53556dfa158b
51. Is there a Maximum Number Allowable Containers? - balena Forums, accessed July 24, 2025, https://forums.balena.io/t/is-there-a-maximum-number-allowable-containers/146698
52. How do you guys manage CPU/memory usage in Docker (Compose) containers vs. essential services? - Reddit, accessed July 24, 2025, https://www.reddit.com/r/selfhosted/comments/1dg0qd4/how_do_you_guys_manage_cpumemory_usage_in_docker/
53. Limiting CPU usage in multi-container environment via docker compose - balena Forums, accessed July 24, 2025, https://forums.balena.io/t/limiting-cpu-usage-in-multi-container-environment-via-docker-compose/3469
54. How to pass environment variable to docker-compose up - Stack Overflow, accessed July 24, 2025, https://stackoverflow.com/questions/49293967/how-to-pass-environment-variable-to-docker-compose-up
55. Balena CLI Advanced Masterclass - GitHub, accessed July 24, 2025, https://github.com/balena-io/balena-cli-advanced-masterclass
56. Support environment variables with balena push
57. Deployment Guidelines - ROS 2 Documentation: Humble documentation, accessed July 24, 2025, https://docs.ros.org/en/humble/Tutorials/Advanced/Security/Deployment-Guidelines.html

