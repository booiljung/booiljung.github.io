---
layout: page
title: Ubuntu 24.04 기반 Docker, ROS 2, NVIDIA를 활용한 GPU 가속 로보틱스 개발 환경 구축
permalink: /ros2/humble/Ubuntu 24.04 기반 Docker ROS2 NVIDIA를 활용한 GPU 가속 로보틱스 개발 환경 구축
---


이 파트에서는 컨테이너의 하드웨어 가속 및 GUI 기능을 활성화하는 데 필수적인 호스트 Ubuntu 24.04 머신의 사전 준비 단계를 상세히 설명합니다. 올바르게 구성된 호스트 시스템은 성공적인 컨테이너 환경 구축의 전제 조건입니다.


안정적이고 효율적인 컨테이너 환경을 구축하기 위한 첫 단계는 호스트 시스템에 Docker 엔진과 NVIDIA 그래픽 드라이버를 올바르게 설치하는 것입니다. 이 두 구성 요소는 물리적 하드웨어와 컨테이너화된 애플리케이션 간의 핵심적인 연결고리 역할을 합니다.


Docker 엔진은 공식 APT 저장소를 통해 설치하는 것이 가장 권장되는 방법입니다. 이 방식은 편의 스크립트나 수동 패키지 다운로드와 달리, 검증된 패키지를 제공하며 시스템의 패키지 관리자와 통합되어 향후 업데이트를 원활하게 진행할 수 있도록 보장합니다.1

설치 절차는 다음과 같습니다.

1. **Docker 공식 GPG 키 추가:** 시스템이 Docker 저장소의 패키지를 신뢰할 수 있도록 GPG 키를 추가합니다. 이 과정은 패키지의 무결성과 신뢰성을 보장합니다.2

   ```Bash
   sudo apt-get update
   sudo apt-get install -y ca-certificates curl
   sudo install -m 0755 -d /etc/apt/keyrings
   sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
   sudo chmod a+r /etc/apt/keyrings/docker.asc
   ```

2. **Docker APT 저장소 설정:** 시스템의 APT 소스 목록에 Docker 저장소를 추가합니다. 이 명령은 시스템 아키텍처와 OS 코드네임을 자동으로 감지하여 적절한 저장소 주소를 생성합니다.2

   ```bash
   echo \
     "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
     $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
     sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

3. **Docker 패키지 설치:** 저장소 설정 후 패키지 목록을 업데이트하고 Docker 관련 패키지들을 설치합니다. `docker-ce` (커뮤니티 에디션), `docker-ce-cli` (명령줄 인터페이스), `containerd.io` (컨테이너 런타임), 그리고 최신 개발 및 배포 워크플로우에 필수적인 `docker-buildx-plugin`과 `docker-compose-plugin`을 함께 설치합니다.1

   ```Bash
   sudo apt-get update
   sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```

4. **설치 후 작업:** Docker 명령을 `sudo` 없이 사용하기 위해 현재 사용자를 `docker` 그룹에 추가합니다. 이 변경 사항을 적용하려면 새 셸 세션을 시작하거나 시스템을 재부팅해야 합니다.2

   ```bash
   sudo usermod -aG docker $USER
   newgrp docker
   ```


컨테이너가 GPU를 활용하기 위해서는 호스트 시스템에 NVIDIA의 독점(proprietary) 드라이버가 반드시 설치되어 있어야 합니다. 이 드라이버는 운영체제 커널과 물리적 GPU 하드웨어 간의 통신을 담당하는 핵심 소프트웨어입니다.3

Ubuntu에서는 "추가 드라이버(Additional Drivers)" 유틸리티를 사용하거나, 다음 명령어를 통해 시스템에 맞는 가장 안정적인 드라이버를 자동으로 설치할 수 있습니다.

```bash
sudo ubuntu-drivers autoinstall
sudo reboot
```

설치 후 시스템을 재부팅하여 드라이버가 커널에 완전히 로드되도록 하는 것이 중요합니다. `nvidia-smi` 명령어를 실행하여 드라이버가 성공적으로 설치되었는지 확인할 수 있습니다.


NVIDIA Container Toolkit은 호스트의 GPU와 드라이버를 Docker 데몬에 표준화된 방식으로 노출시키는 필수적인 미들웨어입니다. 이 툴킷이 없다면 컨테이너는 호스트의 GPU를 인식하거나 사용할 수 없습니다.


툴킷 설치는 전용 APT 저장소를 통해 진행되며, 이는 안정적인 설치와 관리를 보장합니다.3

1. **GPG 키 및 저장소 설정:** NVIDIA Container Toolkit의 GPG 키와 APT 저장소를 시스템에 추가합니다.3

   ```Bash
   curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
   && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
   sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
   sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
   ```

2. **툴킷 패키지 설치:** 패키지 목록을 업데이트하고 `nvidia-container-toolkit`을 설치합니다.3

   ```Bash
   sudo apt-get update
   sudo apt-get install -y nvidia-container-toolkit
   ```


툴킷 설치 과정에서 Docker 데몬 설정 파일(`/etc/docker/daemon.json`)이 자동으로 수정되어 `nvidia` 런타임을 기본 런타임 중 하나로 등록합니다. 이 설정은 Docker가 `--gpus`와 같은 GPU 관련 플래그를 인식하고 처리하는 데 필수적입니다.3

설정이 적용되려면 반드시 Docker 서비스를 재시작해야 합니다.3

```Bash
sudo systemctl restart docker
```

이 아키텍처의 핵심은 드라이버와 툴킷의 분리입니다. 과거에는 컨테이너 내부에 호스트와 동일한 버전의 NVIDIA 드라이버를 설치해야 했습니다. 이는 이미지의 이식성을 심각하게 저해하는 문제였습니다. NVIDIA Container Toolkit은 런타임 시점에 호스트의 사용자 공간 드라이버 라이브러리와 장치 파일만을 컨테이너에 마운트하는 방식으로 이 문제를 해결합니다. 결과적으로, 컨테이너 자체는 개발에 필요한 CUDA 툴킷(라이브러리 및 컴파일러)만 포함하면 되며, 전체 드라이버를 내장할 필요가 없습니다. 이는 단일 Docker 이미지가 특정 드라이버 버전에 종속되지 않고, 최소 요구 사항을 충족하는 호환 가능한 NVIDIA 드라이버가 설치된 모든 호스트에서 실행될 수 있음을 의미합니다. 이러한 아키텍처적 분리는 현대 GPU 컨테이너화 기술의 근간을 이룹니다.


컨테이너 내부에서 실행되는 Gazebo, RViz, Webots와 같은 GUI 애플리케이션을 호스트 화면에 표시하기 위해서는 X11 포워딩 설정이 필요합니다. 로컬 개발 환경에서 가장 직접적인 방법은 호스트의 X 서버에 대한 접근 권한을 조정하는 것입니다.


`xhost`는 X 서버에 대한 접근 제어를 관리하는 유틸리티입니다. 다음 명령어를 실행하여 로컬 시스템의 모든 사용자(Docker 컨테이너에서 실행되는 프로세스 포함)가 X 서버에 접근할 수 있도록 허용할 수 있습니다.8

```bash
xhost +local:
```

이 명령어는 호스트의 네트워크 네임스페이스나 X11 소켓을 공유하는 Docker 컨테이너가 GUI를 호스트 화면에 렌더링할 수 있도록 길을 열어줍니다.

GUI 포워딩 방법의 선택은 단순히 기술적인 문제를 넘어, 보안과 편의성 사이의 균형을 맞추는 위험 관리 결정입니다. `xhost +local:` 명령어는 로컬 개발 환경에서는 매우 편리하지만, X11 프로토콜 자체의 오래된 설계로 인해 보안에 취약할 수 있습니다.10 접근이 허용된 클라이언트는 키 입력 감지, 화면 캡처, 이벤트 주입 등의 잠재적 위협이 될 수 있습니다.10 특히 다중 사용자가 접근하는 시스템에서는 이러한 방식이 심각한 보안 위험을 초래할 수 있습니다.12

더 안전한 대안으로는 `.Xauthority` 파일을 이용한 토큰 기반 인증 방식이 있으며 11, 원격 접속이나 고도의 보안이 요구되는 환경에서는 컨테이너 내부에 VNC 서버를 구축하는 방법도 고려할 수 있습니다.14 이 보고서에서는 단일 사용자 로컬 개발 환경의 편의성을 고려하여 

`xhost` 방식을 채택하지만, 이러한 보안상의 고려사항을 명확히 인지하고 사용해야 합니다.


이 파트는 보고서의 핵심으로, 요청된 모든 환경을 구축하는 최적화된 다단계 `Dockerfile`을 한 줄씩 상세히 분석합니다.


이 복잡한 환경을 구축하는 데 있어 다단계 빌드(Multi-stage build)는 업계 표준 방식입니다. 단일 단계로 빌드할 경우, 컴파일 도구, 개발용 헤더 파일, 중간 빌드 산출물 등 최종 실행 환경에 불필요한 요소들이 이미지에 그대로 남아 용량이 비대해지고 잠재적인 보안 공격 표면이 넓어집니다. 다단계 빌드는 빌드 환경과 최종 런타임 환경을 명확히 분리하여, 최종 이미지를 훨씬 작고, 안전하며, 효율적으로 만듭니다.17

본 보고서에서 사용할 `Dockerfile`은 두 단계로 구성됩니다:

1. **`builder` 단계:** NVIDIA의 `devel` 이미지를 기반으로, 모든 소프트웨어를 소스에서 컴파일하거나 패키지로 설치하고 필요한 모든 의존성을 해결하는 포괄적인 단계입니다.
2. **`final` 단계:** NVIDIA의 `runtime` 이미지를 기반으로, `builder` 단계에서 생성된 필수 실행 파일, 런타임 라이브러리, 설정 파일만을 선별적으로 복사하여 가볍고 최적화된 최종 이미지를 생성하는 단계입니다.


이 단계의 목표는 최종 이미지에 포함될 모든 구성 요소를 준비하고 컴파일하는 것입니다.


빌드 단계의 시작은 적절한 베이스 이미지를 선택하는 것입니다. `FROM` 지시어는 `nvidia/cuda:12.5.1-devel-ubuntu22.04`를 사용합니다. 이 태그는 Ubuntu 22.04를 기반으로 하며, 개발에 필요한 모든 도구를 포함하고 있습니다.20

NVIDIA는 `base`, `runtime`, `devel` 세 가지 주요 이미지 변형을 제공합니다.20

`base` 이미지는 CUDA 런타임(cudart)만 포함하는 가장 기본적인 형태입니다. `runtime` 이미지는 `base` 위에 CUDA 수학 라이브러리(cuBLAS 등)와 NCCL을 추가합니다. 마지막으로 `devel` 이미지는 `runtime` 위에 NVCC 컴파일러, 헤더 파일, 디버깅 도구 등 CUDA 애플리케이션을 *빌드*하는 데 필요한 모든 것을 포함합니다. `builder` 단계의 목표는 일부 CUDA 커널을 포함할 수 있는 ROS 2 패키지를 컴파일하는 것이므로, `devel` 이미지를 사용하는 것이 필수적입니다. 반면, 컴파일된 코드를 *실행*하기만 하면 되는 `final` 단계에서는 더 작은 `runtime` 이미지를 기반으로 하여 최종 이미지의 크기를 최적화할 수 있습니다. 이 선택은 다단계 빌드 전략의 효율성을 극대화하는 핵심 요소입니다.


베이스 이미지 위에 필요한 소프트웨어를 설치합니다. `DEBIAN_FRONTEND=noninteractive` 환경 변수를 설정하여 패키지 설치 과정에서 사용자 입력 대기로 인해 빌드가 중단되는 것을 방지합니다. 또한, `RUN apt-get update && apt-get install -y... && rm -rf /var/lib/apt/lists/*`와 같이 여러 명령을 단일 `RUN` 지시어로 연결하여 Docker 이미지 레이어 수를 최소화하고 이미지 크기를 최적화합니다.22

- **ROS 2 Humble 설치:** 공식 문서에 명시된 비대화형 설치 절차를 정확히 따릅니다. 여기에는 로케일 설정, ROS 2 APT 저장소 추가, 그리고 `ros-humble-desktop` 및 `ros-dev-tools` 패키지 설치가 포함됩니다.23
- **OpenGL 3.3+ 라이브러리:** `nvidia/cuda` 이미지는 핵심 NVIDIA OpenGL 라이브러리를 포함하지만, Gazebo와 같은 애플리케이션을 빌드하고 디버깅하기 위해서는 추가적인 개발 헤더와 유틸리티가 필요합니다. `mesa-utils`(`glxinfo` 제공), `libgl1-mesa-dev`, `libglfw3-dev` 등을 설치하여 완전한 호환성을 확보하고 검증 도구를 제공합니다.24
- **기타 유틸리티:** `terminator` 터미널 에뮬레이터와 개발에 유용한 기타 도구들(`git`, `vim` 등)을 설치합니다.

다음은 `builder` 단계의 핵심적인 `RUN` 명령의 일부입니다.

```Dockerfile
#...
RUN apt-get update && apt-get install -y --no-install-recommends \
    # ROS 2 Humble 설치에 필요한 패키지
    software-properties-common curl gnupg2 lsb-release \
    # GUI 및 OpenGL 관련 패키지
    xterm terminator mesa-utils libgl1-mesa-dev libglfw3-dev libglew-dev \
    # 기타 유틸리티
    git vim build-essential python3-pip \
    && rm -rf /var/lib/apt/lists/*
#... ROS 2 저장소 설정 및 설치...
```


두 시뮬레이터는 각각의 공식 패키지 저장소를 통해 설치합니다.

- **Gazebo Fortress:** `packages.osrfoundation.org` 저장소를 사용합니다. 최신 GPG 키링 기반의 추가 방식을 적용하여 `ignition-fortress` 패키지를 설치합니다.27

  ```Dockerfile
  RUN curl -sSL https://packages.osrfoundation.org/gazebo.gpg -o /usr/share/keyrings/pkgs-osrf-archive-keyring.gpg \
      && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/pkgs-osrf-archive-keyring.gpg] http://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/gazebo-stable.list > /dev/null \
      && apt-get update \
      && apt-get install -y ignition-fortress \
      && rm -rf /var/lib/apt/lists/*
  ```

- **Webots:** Cyberbotics의 공식 APT 저장소를 추가하여 `webots` 패키지를 설치합니다. 이 방식은 의존성을 자동으로 관리해주어 가장 안정적입니다.30

  ```Dockerfile
  RUN wget -qO- https://cyberbotics.com/Cyberbotics.asc | apt-key add - \
      && apt-add-repository 'deb https://cyberbotics.com/debian/ binary-amd64/' \
      && apt-get update \
      && apt-get install -y webots \
      && rm -rf /var/lib/apt/lists/*
  ```


이 단계에서는 `builder` 단계의 결과물을 바탕으로, 실제 개발 및 실행에 사용될 최종 이미지를 만듭니다.


새로운 빌드 단계를 `FROM nvidia/cuda:12.5.1-runtime-ubuntu22.04 AS final`로 시작합니다. `devel` 이미지보다 훨씬 작은 `runtime` 이미지를 사용하여 최종 이미지의 크기를 줄입니다.

`COPY --from=builder` 지시어를 사용하여 `builder` 단계에서 설치된 필수적인 파일과 디렉토리만을 선별적으로 복사합니다. 이는 다단계 빌드의 핵심적인 장점을 활용하는 과정입니다.17

```Dockerfile
# ROS 2 Humble 복사
COPY --from=builder /opt/ros/humble /opt/ros/humble
# Gazebo Fortress 관련 파일 복사
COPY --from=builder /usr/lib/x86_64-linux-gnu/libignition-* /usr/lib/x86_64-linux-gnu/
COPY --from=builder /usr/lib/x86_64-linux-gnu/libsdformat* /usr/lib/x86_64-linux-gnu/
COPY --from=builder /usr/bin/ign /usr/bin/
#... 기타 Gazebo 및 Webots 관련 파일 복사...
# Terminator 및 의존성 복사
COPY --from=builder /usr/bin/terminator /usr/bin/
COPY --from=builder /usr/share/terminator /usr/share/terminator
#...
```


보안상의 이유로 컨테이너를 루트(root) 사용자로 실행하는 것은 매우 위험한 관행입니다.33 따라서 최소한의 권한을 가진 전용 사용자를 생성하여 컨테이너의 기본 사용자로 설정합니다.

`groupadd`와 `useradd` 명령어를 사용하여 `rosdev`라는 이름의 사용자를 생성합니다. 이때, 호스트 사용자와의 파일 권한 충돌을 방지하기 위해 일반적으로 사용되는 UID/GID 값인 1000으로 지정합니다. 이 UID/GID 매칭은 이후 바인드 마운트(bind mount)를 사용한 개발 워크플로우에서 매우 중요한 역할을 합니다.33

```Dockerfile
ARG USER_ID=1000
ARG GROUP_ID=1000
RUN groupadd -g $GROUP_ID rosdev && \
    useradd -u $USER_ID -g $GROUP_ID -ms /bin/bash rosdev
USER rosdev
WORKDIR /home/rosdev
```

`USER rosdev` 지시어를 통해 이후의 모든 명령어와 컨테이너의 기본 실행 사용자를 `rosdev`로 전환합니다.


최종 사용자 환경을 설정하고 컨테이너 시작 시 실행될 스크립트를 구성합니다.

- **환경 변수 설정:** `ENV` 지시어를 사용하여 필요한 환경 변수를 설정합니다. 특히 `NVIDIA_DRIVER_CAPABILITIES`는 GPU의 모든 기능을 활성화하는 데 결정적인 역할을 합니다.

  ```Dockerfile
  ENV NVIDIA_VISIBLE_DEVICES=all
  ENV NVIDIA_DRIVER_CAPABILITIES=graphics,utility,compute
  ENV QT_X11_NO_MITSHM=1
  ```

  `NVIDIA_DRIVER_CAPABILITIES` 환경 변수는 NVIDIA Container Toolkit이 컨테이너에 어떤 드라이버 기능을 노출할지 제어하는 핵심 설정입니다.36 이 변수는 쉼표로 구분된 기능 목록을 값으로 가집니다: 

  `compute`는 CUDA 및 OpenCL, `graphics`는 OpenGL 및 Vulkan, `utility`는 `nvidia-smi`와 같은 관리 도구를 의미합니다.13 이 프로젝트는 CUDA 연산, Gazebo/RViz의 OpenGL 렌더링, 

  `nvidia-smi`를 통한 모니터링을 모두 요구하므로, 이 세 가지 기능을 모두 포함하는 `graphics,utility,compute` (또는 간편하게 `all`)로 설정해야 합니다. 만약 이 변수가 잘못 설정되거나 누락되면(예: 기본값인 `compute,utility` 사용), Gazebo와 같은 OpenGL 애플리케이션은 하드웨어 가속을 사용하지 못하고 `llvmpipe`를 이용한 느린 소프트웨어 렌더링으로 대체되거나 아예 실행에 실패할 수 있습니다.

- **Entrypoint 스크립트:** 컨테이너가 시작될 때마다 ROS 2 환경을 자동으로 활성화하기 위해 `entrypoint.sh` 스크립트를 생성하고 사용합니다. 이 스크립트는 ROS 2의 `setup.bash`를 소싱한 후, `docker run`이나 `docker-compose`에서 전달된 주 명령어를 실행(`exec "$@"`)합니다. 이를 통해 사용자는 컨테이너에 접속하거나 명령을 실행할 때마다 수동으로 환경을 설정할 필요가 없어집니다.37

  ```Bash
  # entrypoint.sh
  #!/bin/bash
  set -e
  source /opt/ros/humble/setup.bash
  exec "$@"
  ```

  `Dockerfile`에서는 이 스크립트를 복사하고 실행 권한을 부여한 뒤 `ENTRYPOINT`로 지정합니다.

  Dockerfile

  ```
  COPY entrypoint.sh /usr/local/bin/
  RUN chmod +x /usr/local/bin/entrypoint.sh
  ENTRYPOINT ["entrypoint.sh"]
  CMD ["bash"]
  ```

  `CMD ["bash"]`는 컨테이너 실행 시 별도의 명령이 주어지지 않으면 기본적으로 대화형 셸을 시작하도록 합니다.


이 파트에서는 빌드된 이미지를 실제 개발 주기에 효과적으로 통합하고 사용하는 방법을 다룹니다.


`Dockerfile` 작성이 완료되면, 다음 명령어를 사용하여 이미지를 빌드합니다. `DOCKER_BUILDKIT=1`은 더 빠르고 효율적인 최신 빌드 엔진을 사용하도록 지정합니다.

```Bash
DOCKER_BUILDKIT=1 docker build -t ros-dev-env:humble.
```

이미지 태그는 프로젝트의 내용을 명확하게 나타내도록 지정하는 것이 중요합니다. `[이름]:` 형식(예: `ros-dev-env:humble`)은 직관적이고 관리가 용이합니다. 지속적 통합/지속적 배포(CI/CD) 파이프라인에서는 버전 추적을 위해 Git 커밋 해시를 추가 태그로 사용하는 것이 모범 사례입니다.39


`docker run` 명령어는 GPU, GUI 포워딩, 볼륨 마운트 등 수많은 옵션으로 인해 길고 복잡해지기 쉽습니다. `docker-compose.yml` 파일을 사용하면 이러한 복잡한 실행 구성을 코드 형태로 관리하여 일관성과 재현성을 보장할 수 있습니다.42

다음은 이 개발 환경을 위한 완전한 `docker-compose.yml` 파일 예시입니다.

```YAML
services:
  ros_dev:
    image: ros-dev-env:humble
    container_name: ros_dev_humble
    # build:
    #   context:.
    #   args:
    #     - USER_ID=${UID:-1000}
    #     - GROUP_ID=${GID:-1000}
    runtime: nvidia
    environment:
      - DISPLAY=${DISPLAY}
      - QT_X11_NO_MITSHM=1
      - NVIDIA_DRIVER_CAPABILITIES=all
    volumes:
      - /tmp/.X11-unix:/tmp/.X11-unix:rw
      # 아래 줄의 주석을 해제하고 호스트의 워크스페이스 경로로 수정하세요.
      # - ~/ros2_ws/src:/home/rosdev/ros2_ws/src
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
    network_mode: "host"
    command: terminator
```

이 파일은 GPU 자원 요청(`deploy`), 환경 변수 설정, X11 소켓 볼륨 마운트, 네트워크 모드 설정 등 모든 실행 옵션을 체계적으로 정의합니다.45

`docker compose up` 명령어 하나로 이 모든 구성이 적용된 컨테이너를 실행할 수 있습니다.


이 환경의 가장 큰 장점 중 하나는 호스트 시스템의 소스 코드 디렉토리를 컨테이너 내부의 작업 공간에 직접 연결(바인드 마운트)하여 실시간으로 코드를 편집하고 빌드할 수 있다는 점입니다.48


1. **코드 편집:** 호스트 머신에서 선호하는 IDE(예: VS Code)를 사용하여 `~/ros2_ws/src` 디렉토리의 소스 코드를 수정합니다.
2. **실시간 동기화:** 바인드 마운트를 통해 변경 사항이 즉시 컨테이너 내부의 `/home/rosdev/ros2_ws/src`에 반영됩니다.
3. **컨테이너 내 빌드:** 컨테이너의 터미널(Terminator)에서 `cd ~/ros2_ws`로 이동한 후 `colcon build` 명령을 실행하여 코드를 컴파일합니다.
4. **테스트 및 실행:** 빌드가 완료되면 컨테이너 내에서 `ros2 launch` 또는 `ros2 run` 명령으로 노드를 실행하여 변경 사항을 즉시 테스트합니다.


바인드 마운트를 사용할 때 발생하는 가장 흔한 문제는 파일 권한 충돌입니다. 컨테이너 내부에서 `colcon build`를 실행하면 `build/`와 `install/` 디렉토리가 생성되는데, 만약 컨테이너 내부 사용자의 UID/GID와 호스트 사용자의 UID/GID가 다를 경우, 호스트에서 이 파일들을 수정하거나 삭제하려고 할 때 "권한 거부(Permission denied)" 오류가 발생합니다.51

이 보고서에서 제시한 `Dockerfile`과 `docker-compose.yml`은 이 문제를 근본적으로 해결합니다. `Dockerfile`에서 `rosdev` 사용자를 생성할 때 `ARG`를 통해 외부에서 UID/GID를 전달받도록 설계했습니다(섹션 2.3.2). 그리고 `docker-compose.yml`의 `build.args` 섹션에서 호스트의 현재 사용자 UID(`$UID`)와 GID(`$GID`)를 빌드 시점에 동적으로 전달합니다. 이 과정을 통해 컨테이너 내부 사용자는 호스트 사용자와 완벽하게 동일한 소유권을 갖게 되어, 바인드 마운트된 디렉토리에서 생성된 모든 파일에 대해 양쪽에서 자유롭게 읽고 쓸 수 있습니다. 이는 지저분한 `chown` 명령어나 권한 변경 스크립트 없이도 이식성 있고 원활한 개발 경험을 제공하는 가장 견고한 해결책입니다.35


복잡한 환경을 구축한 후에는 각 구성 요소가 의도대로 작동하는지 체계적으로 확인하는 과정이 필수적입니다. 다음 검증 매트릭스는 초기 설정 확인 및 향후 문제 해결을 위한 가이드라인을 제공합니다. 모든 검증 명령어는 `docker compose up`으로 실행된 컨테이너 내부의 터미널에서 수행되어야 합니다.

이러한 구조화된 접근 방식은 시스템의 여러 잠재적 실패 지점(GPU 드라이버, 툴킷, X11, ROS, 시뮬레이터 등)을 개별적으로 검증할 수 있게 해줍니다. 단순한 "실행된다"는 확인을 넘어, 각 구성 요소의 기능성을 명확히 확인함으로써 사용자는 문제 발생 시 원인을 신속하게 파악하고 격리할 수 있습니다.


| 구성 요소                  | 검증 명령어 (컨테이너 내부)                                  | 예상 출력/동작                                               | 중요성 및 관련 정보                                          |
| -------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **NVIDIA 드라이버 & CUDA** | `nvidia-smi`                                                 | 호스트의 드라이버 버전, CUDA 버전, GPU 상세 정보가 포함된 표가 오류 없이 출력됩니다. | NVIDIA Container Toolkit이 드라이버 접근을 올바르게 전달하는지 확인합니다. "Driver/library version mismatch" 오류는 호스트 드라이버 문제를 의미하며, 보통 호스트 재부팅으로 해결됩니다. 54 |
| **OpenGL 가속**            | `glxinfo -B`                                                 | `direct rendering: Yes`가 출력되고, `OpenGL renderer string:` 항목에 `llvmpipe`나 `Mesa`가 아닌 **NVIDIA GPU 이름**이 명시되어야 합니다. | Gazebo, RViz의 하드웨어 그래픽 가속 여부를 확인하는 가장 중요한 단계입니다. `llvmpipe`가 출력되면 CPU를 사용한 느린 소프트웨어 렌더링으로 대체되었음을 의미하며, 이는 구성 오류의 전형적인 증상입니다. 26 |
| **ROS 2 Humble**           | 터미널 1: `ros2 run demo_nodes_cpp talker`  터미널 2: `ros2 run demo_nodes_py listener` | `talker`가 메시지를 발행하고, `listener`가 해당 메시지를 수신하여 출력합니다. | ROS 2의 핵심 통신 메커니즘(DDS)이 정상적으로 작동하는지 검증합니다. 23 |
| **Gazebo Fortress**        | `ign gazebo shapes.sdf`                                      | 호스트 화면에 Gazebo GUI 창이 열리고, 세 개의 간단한 도형이 렌더링됩니다. | Gazebo의 실행, OpenGL을 통한 그래픽 렌더링, X11 포워딩 기능이 모두 정상 작동함을 확인합니다. 61 |
| **Webots**                 | `webots`                                                     | 호스트 화면에 Webots GUI 창이 열립니다.                      | Webots의 설치 및 GUI 포워딩 기능이 정상 작동함을 확인합니다. 30 |
| **GUI 터미널**             | `terminator`                                                 | 호스트 화면에 Terminator 터미널 애플리케이션 창이 열립니다.  | 일반적인 GUI 포워딩 메커니즘이 올바르게 설정되었는지 최종적으로 확인합니다. |


이 마지막 파트에서는 보다 심층적인 주제와 일반적인 문제 해결 방안을 다루어, 이 보고서를 단순한 가이드를 넘어 포괄적인 참조 문서로 완성합니다.


"Shift-left" 보안 개념에 따라, 컨테이너 이미지를 배포하기 전에 알려진 보안 취약점(CVE)을 스캔하는 것이 중요합니다. Trivy는 이를 위한 강력하고 사용하기 쉬운 오픈소스 도구입니다.

새롭게 빌드한 `ros-dev-env:humble` 이미지를 스캔하는 방법은 다음과 같습니다.

1. **Trivy 설치 (호스트):**

   ```Bash
   sudo apt-get install wget apt-transport-https gnupg lsb-release
   wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
   echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
   sudo apt-get update
   sudo apt-get install -y trivy
   ```

2. **이미지 스캔:**

   ```Bash
   trivy image ros-dev-env:humble
   ```

Trivy는 이미지의 모든 레이어를 분석하여 OS 패키지와 애플리케이션 의존성에서 발견된 취약점을 심각도(CRITICAL, HIGH, MEDIUM, LOW)에 따라 보고합니다.64 이 결과를 바탕으로 패키지를 업데이트하거나 보안 패치가 적용된 베이스 이미지를 사용하여 공격 표면을 최소화하는 조치를 취할 수 있습니다.


로보틱스 개발의 실제 환경에서는 여러 로봇이나 여러 시뮬레이션을 동시에 다루는 경우가 빈번합니다. ROS 2의 기본 네트워킹(DDS)은 동일 네트워크상의 모든 노드가 서로를 발견하고 통신하도록 설계되었습니다.67 이로 인해 두 개의 독립적인 시뮬레이션을 실행하면, 각 시뮬레이션의 `/cmd_vel`과 같은 공통 토픽이 서로 간섭하여 예측 불가능하고 혼란스러운 동작을 유발할 수 있습니다.

이러한 문제를 방지하고 각 ROS 2 시스템을 논리적으로 격리하는 표준적인 방법은 `ROS_DOMAIN_ID` 환경 변수를 사용하는 것입니다.69 각 시스템 그룹에 고유한 ID를 할당하면, 동일한 ID를 가진 노드들만 서로 통신할 수 있습니다.

`docker-compose.yml` 파일의 `environment` 섹션에 `ROS_DOMAIN_ID`를 추가하여 컨테이너의 ROS 2 네트워크를 호스트나 다른 컨테이너의 ROS 2 시스템으로부터 쉽게 격리할 수 있습니다.

```YAML
services:
  ros_dev:
    #...
    environment:
      - DISPLAY=${DISPLAY}
      #...
      - ROS_DOMAIN_ID=10 # 0-101 사이의 고유한 값으로 설정
```

이러한 사전 예방적 조치는 단일 인스턴스 설정을 넘어, 로보틱스 개발의 운영 현실에 대한 깊은 이해를 반영하는 전문가 수준의 가이드에서 필수적인 부분입니다.


- **`cannot open display` 오류:** GUI 포워딩 시 가장 흔하게 발생하는 문제입니다. 원인은 주로 잘못된 `DISPLAY` 환경 변수 설정, `/tmp/.X11-unix` 볼륨 마운트 누락, 또는 `xhost` 권한 미설정입니다. 파트 I, III의 설정을 다시 확인하여 해결할 수 있습니다.72
- **`Failed to initialize NVML: Driver/library version mismatch` 오류:** 이 오류는 컨테이너가 아닌 호스트 시스템의 문제입니다. 주로 호스트의 NVIDIA 드라이버가 업데이트된 후 시스템을 재부팅하지 않았을 때 발생합니다. 가장 확실한 해결책은 호스트 머신을 재부팅하는 것입니다. 드물게는 커널 모듈을 수동으로 재로드해야 할 수도 있습니다.56
- **Gazebo/RViz 실행 속도 저하:** 이는 하드웨어 가속 실패의 전형적인 증상입니다. 컨테이너 내에서 `glxinfo -B` 명령을 실행하여 `OpenGL renderer string`이 `llvmpipe`가 아닌 NVIDIA GPU로 표시되는지 확인해야 합니다(파트 IV 참조). `llvmpipe`로 표시된다면, `NVIDIA_DRIVER_CAPABILITIES` 환경 변수 설정이나 NVIDIA Container Toolkit 설치 과정에 문제가 있을 가능성이 높습니다.60



```Dockerfile
# syntax=docker/dockerfile:1.4

# ==================================================================
# Builder Stage: 모든 의존성을 설치하고 소스를 컴파일하는 단계
# ==================================================================
FROM nvidia/cuda:12.5.1-devel-ubuntu22.04 AS builder

# 빌드 중 대화형 프롬프트 방지
ENV DEBIAN_FRONTEND=noninteractive

# 기본 패키지 및 유틸리티 설치
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    git \
    vim \
    curl \
    gnupg2 \
    lsb-release \
    software-properties-common \
    python3-pip \
    python3-colcon-common-extensions \
    python3-rosdep \
    # GUI 및 OpenGL 관련 패키지
    xterm \
    terminator \
    mesa-utils \
    libgl1-mesa-dev \
    libglfw3-dev \
    libglew-dev \
    && rm -rf /var/lib/apt/lists/*

# ROS 2 Humble 설치
# 1. 로케일 설정
RUN apt-get update && apt-get install -y locales \
    && locale-gen en_US en_US.UTF-8 \
    && update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8 \
    && rm -rf /var/lib/apt/lists/*
ENV LANG=en_US.UTF-8

# 2. ROS 2 소스 설정
RUN curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null

# 3. ROS 2 패키지 설치
RUN apt-get update && apt-get upgrade -y \
    && apt-get install -y \
    ros-humble-desktop \
    ros-dev-tools \
    && rm -rf /var/lib/apt/lists/*

# Gazebo Fortress 설치
RUN curl -sSL https://packages.osrfoundation.org/gazebo.gpg -o /usr/share/keyrings/pkgs-osrf-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/pkgs-osrf-archive-keyring.gpg] http://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/gazebo-stable.list > /dev/null \
    && apt-get update \
    && apt-get install -y ignition-fortress \
    && rm -rf /var/lib/apt/lists/*

# Webots 설치
RUN wget -qO- https://cyberbotics.com/Cyberbotics.asc | apt-key add - \
    && apt-add-repository 'deb https://cyberbotics.com/debian/ binary-amd64/' \
    && apt-get update \
    && apt-get install -y webots \
    && rm -rf /var/lib/apt/lists/*

# ==================================================================
# Final Stage: 최종 런타임 이미지를 생성하는 단계
# ==================================================================
FROM nvidia/cuda:12.5.1-runtime-ubuntu22.04 AS final

# 빌드 중 대화형 프롬프트 방지
ENV DEBIAN_FRONTEND=noninteractive

# Builder 단계에서 설치한 런타임 의존성 패키지들을 설치
# (COPY만으로는 공유 라이브러리 의존성이 깨질 수 있으므로, 핵심 런타임 라이브러리는 다시 설치)
RUN apt-get update && apt-get install -y --no-install-recommends \
    libgl1-mesa-glx \
    libglib2.0-0 \
    libx11-6 \
    libxcb1 \
    libfontconfig1 \
    terminator \
    && rm -rf /var/lib/apt/lists/*

# Builder 단계에서 빌드/설치된 주요 애플리케이션들을 복사
COPY --from=builder /opt/ros/humble /opt/ros/humble
COPY --from=builder /usr/local/webots /usr/local/webots
COPY --from=builder /usr/bin/ign /usr/bin/ign
COPY --from=builder /usr/lib/x86_64-linux-gnu/libignition-* /usr/lib/x86_64-linux-gnu/
COPY --from=builder /usr/lib/x86_64-linux-gnu/libsdformat* /usr/lib/x86_64-linux-gnu/
COPY --from=builder /usr/share/ignition /usr/share/ignition
COPY --from=builder /usr/share/sdformat /usr/share/sdformat

# 최소 권한 원칙을 위한 비-루트 사용자 생성
# 호스트 사용자와의 UID/GID 매칭을 위해 ARG 사용
ARG USER_ID=1000
ARG GROUP_ID=1000
RUN groupadd -g $GROUP_ID rosdev && \
    useradd -u $USER_ID -g $GROUP_ID -ms /bin/bash rosdev && \
    mkdir -p /home/rosdev/ros2_ws/src && \
    chown -R rosdev:rosdev /home/rosdev

# 사용자 전환
USER rosdev
WORKDIR /home/rosdev

# 환경 변수 설정
ENV NVIDIA_VISIBLE_DEVICES=all
ENV NVIDIA_DRIVER_CAPABILITIES=graphics,utility,compute
ENV QT_X11_NO_MITSHM=1
ENV LD_LIBRARY_PATH=/usr/local/webots/lib:${LD_LIBRARY_PATH}

# Entrypoint 설정
COPY entrypoint.sh /usr/local/bin/
RUN sudo chmod +x /usr/local/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]
CMD ["bash"]
```


```YAML
# docker-compose.yml
# 이 서비스를 실행하려면: docker compose up -d
# 컨테이너에 접속하려면: docker exec -it ros_dev_humble bash
services:
  ros_dev:
    # 빌드된 이미지 이름을 사용합니다.
    image: ros-dev-env:humble
    container_name: ros_dev_humble

    # 로컬에서 Dockerfile을 직접 빌드하려면 아래 섹션의 주석을 해제합니다.
    # build:
    #   context:.
    #   dockerfile: Dockerfile
    #   # 호스트의 UID/GID를 빌드 인자로 전달하여 파일 권한 문제를 해결합니다.
    #   args:
    #     - USER_ID=${UID:-1000}
    #     - GROUP_ID=${GID:-1000}

    # NVIDIA 런타임을 사용하여 GPU에 접근합니다.
    runtime: nvidia

    # 컨테이너 환경 변수를 설정합니다.
    environment:
      - DISPLAY=${DISPLAY}
      - QT_X11_NO_MITSHM=1
      - NVIDIA_DRIVER_CAPABILITIES=all
      # 다중 로봇/시뮬레이션 환경을 위해 고유 ID를 설정합니다.
      # - ROS_DOMAIN_ID=10

    # 볼륨을 마운트합니다.
    volumes:
      # GUI 애플리케이션을 위한 X11 소켓 마운트
      - /tmp/.X11-unix:/tmp/.X11-unix:rw
      # 호스트의 소스 코드 디렉토리를 컨테이너의 워크스페이스에 마운트합니다.
      # 아래 경로를 실제 호스트 워크스페이스 경로로 수정하세요.
      - ~/ros2_ws/src:/home/rosdev/ros2_ws/src

    # GPU 자원을 컨테이너에 할당합니다.
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]

    # 호스트의 네트워크를 공유하여 GUI 포워딩 및 DDS 통신을 용이하게 합니다.
    network_mode: "host"
    
    # 컨테이너가 종료되지 않도록 유지하고, 기본적으로 terminator를 실행합니다.
    stdin_open: true
    tty: true
    command: terminator
```


| 목적                           | 명령어                                           | 실행 위치       |
| ------------------------------ | ------------------------------------------------ | --------------- |
| **호스트 설정**                |                                                  |                 |
| Docker 설치                    | (파트 I, 섹션 1.1 참조)                          | 호스트 터미널   |
| NVIDIA 드라이버 설치           | `sudo ubuntu-drivers autoinstall && sudo reboot` | 호스트 터미널   |
| NVIDIA Container Toolkit 설치  | (파트 I, 섹션 1.2 참조)                          | 호스트 터미널   |
| X11 접근 허용                  | `xhost +local:`                                  | 호스트 터미널   |
| **이미지 및 컨테이너 관리**    |                                                  |                 |
| Docker 이미지 빌드             | `docker build -t ros-dev-env:humble.`            | 호스트 터미널   |
| Docker Compose로 컨테이너 시작 | `docker compose up -d`                           | 호스트 터미널   |
| 실행 중인 컨테이너에 접속      | `docker exec -it ros_dev_humble bash`            | 호스트 터미널   |
| 컨테이너 중지 및 제거          | `docker compose down`                            | 호스트 터미널   |
| **컨테이너 내부 작업**         |                                                  |                 |
| ROS 2 워크스페이스 빌드        | `cd ~/ros2_ws && colcon build`                   | 컨테이너 터미널 |
| ROS 2 환경 소싱                | (entrypoint.sh에 의해 자동 실행됨)               | 컨테이너 터미널 |
| **시스템 검증**                |                                                  |                 |
| GPU 상태 확인                  | `nvidia-smi`                                     | 컨테이너 터미널 |
| OpenGL 가속 확인               | `glxinfo -B`                                     | 컨테이너 터미널 |
| Gazebo 실행                    | `ign gazebo shapes.sdf`                          | 컨테이너 터미널 |
| ROS 2 데모 실행                | `ros2 run demo_nodes_cpp talker`                 | 컨테이너 터미널 |


1. Install Docker Engine on Ubuntu - Docker Docs, accessed July 24, 2025, https://docs.docker.com/engine/install/ubuntu/
2. How to Install Docker on Ubuntu 24.04 Step-by-Step - LinuxTechi, accessed July 24, 2025, https://www.linuxtechi.com/install-docker-on-ubuntu-24-04/
3. Installing the NVIDIA Container Toolkit - NVIDIA Docs Hub, accessed July 24, 2025, https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html
4. Ubuntu 24.04 : NVIDIA Container Toolkit : Install - Server World, accessed July 24, 2025, https://www.server-world.info/en/note?os=Ubuntu_24.04&p=nvidia&f=3
5. Using GPU With Docker: A How-to Guide - DevZero, accessed July 24, 2025, https://www.devzero.io/blog/docker-gpu
6. Install NVIDIA Container Toolkit on Ubuntu 24.04 - Lindevs, accessed July 24, 2025, https://lindevs.com/install-nvidia-container-toolkit-on-ubuntu
7. Installing NVIDIA Container Toolkit and Docker - IBM, accessed July 24, 2025, https://www.ibm.com/docs/en/masv-and-l/maximo-vi/cd?topic=planning-installing-nvidia-container-toolkit-docker
8. Running GUI Applications in a Docker Container / GitHub, accessed July 24, 2025, https://gist.github.com/janert/e1d8e6ae74a8c94173ef35fa356ce2da
9. Making X work inside docker - GitHub Gist, accessed July 24, 2025, https://gist.github.com/MarcinZukowski/ecea29823907df3386c2cac7b56e0385
10. Docker X11 Forwarding Security, accessed July 24, 2025, https://security.stackexchange.com/questions/104222/docker-x11-forwarding-security
11. Running a GUI Application in a Docker Container: Users, Permissions, Dockerfile, accessed July 24, 2025, https://janert.me/blog/2022/running-a-gui-application-in-a-docker-container2/
12. Docker GUI Applications (use of xhost+) - General, accessed July 24, 2025, https://forums.docker.com/t/docker-gui-applications-use-of-xhost/133342
13. NVIDIA Docker2 with OpenGL and X Display Output | Puget Systems, accessed July 24, 2025, https://www.pugetsystems.com/labs/hpc/nvidia-docker2-with-opengl-and-x-display-output-1527/
14. Using VNCserver + GUI application + Virtual Display in Docker container - Stack Overflow, accessed July 24, 2025, https://stackoverflow.com/questions/36221215/using-vncserver-gui-application-virtual-display-in-docker-container
15. bandi13/gui-docker: Image with a VNC server on port 5900 and a webclient on port 5901 for containerized GUI applications - GitHub, accessed July 24, 2025, https://github.com/bandi13/gui-docker
16. How to add VNC to the NVIDIA docker container? - Reddit, accessed July 24, 2025, https://www.reddit.com/r/docker/comments/1ehgkfz/how_to_add_vnc_to_the_nvidia_docker_container/
17. Multi-stage builds - Docker Docs, accessed July 24, 2025, https://docs.docker.com/build/building/multi-stage/
18. About Docker Multi-Stage Builds and COPY -from | Level Up Coding, accessed July 24, 2025, https://levelup.gitconnected.com/docker-multi-stage-builds-and-copy-from-other-images-3bf1a2b095e0
19. Building best practices - Docker Docs, accessed July 24, 2025, https://docs.docker.com/build/building/best-practices/
20. nvidia/cuda - Docker Image - Docker Hub, accessed July 24, 2025, https://hub.docker.com/r/nvidia/cuda
21. CUDA - NVIDIA NGC, accessed July 24, 2025, https://catalog.ngc.nvidia.com/orgs/nvidia/containers/cuda
22. How to lint Dockerfiles with Hadolint - BellSoft, accessed July 24, 2025, https://bell-sw.com/blog/how-to-use-a-dockerfile-linter/
23. Ubuntu (deb packages) - ROS 2 Documentation: Humble ..., accessed July 24, 2025, https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debs.html
24. libglfw3-dev : Jammy (22.04) : Ubuntu - Launchpad, accessed July 24, 2025, https://launchpad.net/ubuntu/jammy/+package/libglfw3-dev
25. How to install OpenGL on Ubuntu 22.04 - YouTube, accessed July 24, 2025, https://www.youtube.com/watch?v=JxDLGHil-Cw
26. Unable to use Nvidia opengl driver inside a docker container - Linux ..., accessed July 24, 2025, https://forums.developer.nvidia.com/t/unable-to-use-nvidia-opengl-driver-inside-a-docker-container/77418
27. Install Gazebo Fortress and confirm that it is correct., accessed July 24, 2025, https://reifxsolutions.com/tut-for-hum-fort-setup/
28. Installing Gazebo and ROS2 on Ubuntu 22 - Jeremy Pedersen, accessed July 24, 2025, https://jeremypedersen.com/posts/2024-07-17-gazebo-ros-install/
29. Binary Installation on Ubuntu - Gazebo ionic documentation, accessed July 24, 2025, https://gazebosim.org/docs/fortress/install_ubuntu
30. Webots documentation: Installation Procedure, accessed July 24, 2025, https://cyberbotics.com/doc/guide/installation-procedure
31. Webots documentation: Installation Procedure, accessed July 24, 2025, https://www.cyberbotics.com/doc/guide/installation-procedure?version=master
32. Copying things out of a Docker based build / community / Discussion #27092 - GitHub, accessed July 24, 2025, https://github.com/orgs/community/discussions/27092
33. 10 Docker Security Best Practices - Snyk, accessed July 24, 2025, https://snyk.io/blog/10-docker-image-security-best-practices/
34. Top 20 Dockerfile best practices - Sysdig, accessed July 24, 2025, https://sysdig.com/learn-cloud-native/dockerfile-best-practices/
35. Allow Docker Container & Host User To Write on Bind Mounted Host Directory, accessed July 24, 2025, https://stackoverflow.com/questions/71918710/allow-docker-container-host-user-to-write-on-bind-mounted-host-directory
36. Specialized Configurations with Docker - NVIDIA Container Toolkit, accessed July 24, 2025, https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/docker-specialized.html
37. Docker for Development: Zero to Hero - Nav2 1.0.0 documentation, accessed July 24, 2025, https://docs.nav2.org/tutorials/docs/docker_dev.html
38. An Updated Guide to Docker and ROS 2 - Robotic Sea Bass, accessed July 24, 2025, https://roboticseabass.com/2023/07/09/updated-guide-docker-and-ros2/
39. Using Semver for Docker Image Tags | by Marc Campbell - Medium, accessed July 24, 2025, https://medium.com/@mccode/using-semantic-versioning-for-docker-image-tags-dfde8be06699
40. Docker Tagging: Best practices for tagging and versioning docker images - Steve Lasker, accessed July 24, 2025, https://stevelasker.blog/2018/03/01/docker-tagging-best-practices-for-tagging-and-versioning-docker-images/
41. Properly Versioning Docker Images - Stack Overflow, accessed July 24, 2025, https://stackoverflow.com/questions/56212495/properly-versioning-docker-images
42. Running ROS 2 nodes in Docker [community-contributed], accessed July 24, 2025, https://docs.ros.org/en/humble/How-To-Guides/Run-2-nodes-in-single-or-separate-docker-containers.html
43. Trying out ROS2 Jazzy Jalisco in 10 mins | by RoboFoundry - Medium, accessed July 24, 2025, https://robofoundry.medium.com/trying-out-ros2-jazzy-jalisco-in-10-mins-177ad6bcbed2
44. ROS Development in Docker - Kevin DeMarco, accessed July 24, 2025, https://www.kevindemarco.com/ros/docker/docker-compose/robotics/programming/development/2022/12/28/ros-docker.html
45. Enable GPU support - Docker Docs, accessed July 24, 2025, https://docs.docker.com/compose/how-tos/gpu-support/
46. dockerfiles/ros2/humble-cuda.Dockerfile at main / athackst/dockerfiles - GitHub, accessed July 24, 2025, https://github.com/athackst/dockerfiles/blob/main/ros2/humble-cuda.Dockerfile
47. niladut/ros2-docker-workspace: Streamline ROS 2 development with Docker & VSCode. Isolated environments, graphical app support, collaboration-friendly. Get started quickly. - GitHub, accessed July 24, 2025, https://github.com/niladut/ros2-docker-workspace
48. ROS2 Humble GUI-Docker Container: A Step-by-Step Guide | by Shashank Goyal | FAUN, accessed July 24, 2025, https://faun.pub/ros2-humble-gui-docker-container-a-step-by-step-guide-c541b73fe141
49. My Docker Development Workflow: Code, Build, Push, Run - Tutorial Works, accessed July 24, 2025, https://www.tutorialworks.com/container-development-workflow/
50. Software Development workflow with Docker | by Techbriel | Medium, accessed July 24, 2025, https://medium.com/@media_9518/software-development-workflow-with-docker-dd7fe49ef096
51. Is there an easy solution to the permission problem on bind mount folders? : r/docker, accessed July 24, 2025, https://www.reddit.com/r/docker/comments/1b8dnm6/is_there_an_easy_solution_to_the_permission/
52. How to keep clean your Docker bind-mounts files permissions? - Willow's site, accessed July 24, 2025, https://www.willowbarraco.fr/perfect-docker-bind-mount-permissions/
53. Docker and the host filesystem owner matching problem - Joyful Bikeshedding, accessed July 24, 2025, https://www.joyfulbikeshedding.com/blog/2021-03-15-docker-and-the-host-filesystem-owner-matching-problem.html
54. Running a Sample Workload - NVIDIA Container Toolkit, accessed July 24, 2025, https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/sample-workload.html
55. A Beginner's Guide to NVIDIA Container Toolkit on Docker | by Umberto Junior Mele, accessed July 24, 2025, https://medium.com/@u.mele.coding/a-beginners-guide-to-nvidia-container-toolkit-on-docker-92b645f92006
56. Failed to initialize NVML: Driver/library version mismatch - GPU Mart, accessed July 24, 2025, https://www.gpu-mart.com/blog/failed-to-initialize-nvml-driver-library-version-mismatch
57. Nvidia NVML Driver/library version mismatch [closed] - Stack Overflow, accessed July 24, 2025, https://stackoverflow.com/questions/43022843/nvidia-nvml-driver-library-version-mismatch
58. docker/Tutorials/Hardware Acceleration - ROS Wiki, accessed July 24, 2025, [http://wiki.ros.org/docker/Tutorials/Hardware%20Acceleration](http://wiki.ros.org/docker/Tutorials/Hardware Acceleration)
59. Gazebo in Windows Docker cannot use Nvidia GPU, falls back to using CPU. / Issue #2595 / gazebosim/gz-sim - GitHub, accessed July 24, 2025, https://github.com/gazebosim/gz-sim/issues/2595
60. Llvmpipe is used as OpenGL render instead of NVIDIA. Cannot switch to NVIDIA GPU, accessed July 24, 2025, https://forums.developer.nvidia.com/t/llvmpipe-is-used-as-opengl-render-instead-of-nvidia-cannot-switch-to-nvidia-gpu/286331
61. Getting Started with Gazebo? - Gazebo ionic documentation, accessed July 24, 2025, https://gazebosim.org/docs/latest/getstarted/
62. Gazebo Fortress installation issue on Ubuntu 22.04 (Humble) – gz sim not running - Help, accessed July 24, 2025, https://community.gazebosim.org/t/gazebo-fortress-installation-issue-on-ubuntu-22-04-humble-gz-sim-not-running/3422
63. Unable to install gazebo-ros and its plugins on ros 2 humble ubuntu 22.04 on mac m2 via UTM hypervisor - Reddit, accessed July 24, 2025, https://www.reddit.com/r/ROS/comments/16wc472/unable_to_install_gazeboros_and_its_plugins_on/
64. Tutorial: Container image scans with Aqua Trivy | Harness Developer Hub, accessed July 24, 2025, https://developer.harness.io/docs/security-testing-orchestration/sto-techref-category/trivy/container-scan-aqua-trivy/
65. Using Trivy Container Scanner to Detect Vulnerabilities (Comprehensive Guide), accessed July 24, 2025, https://devopscube.com/scan-docker-images-using-trivy/
66. Scanning Docker Images for Vulnerabilities: Using Trivy for Effective Security Analysis | by Ramkrushna Maheshwar | Medium, accessed July 24, 2025, https://medium.com/@maheshwar.ramkrushna/scanning-docker-images-for-vulnerabilities-using-trivy-for-effective-security-analysis-fa3e2844db22
67. Running ROS 2 on Multiple Machines - Husarion, accessed July 24, 2025, https://husarion.com/tutorials/ros2-tutorials/6-robot-network/
68. Using multiple Create® 3 robots - GitHub Pages, accessed July 24, 2025, https://iroboteducation.github.io/create3_docs/setup/multi-robot/
69. Multiple robots / User Manual - GitHub Pages, accessed July 24, 2025, https://turtlebot.github.io/turtlebot4-user-manual/tutorials/multiple_robots.html
70. ROS2 Multiple Machines Tutorial (including Raspberry Pi) - The Robotics Back-End, accessed July 24, 2025, https://roboticsbackend.com/ros2-multiple-machines-including-raspberry-pi/
71. The ROS_DOMAIN_ID - ROS 2 Documentation, accessed July 24, 2025, https://docs.ros.org/en/foxy/Concepts/About-Domain-ID.html
72. error at docker visual compose / Issue #4 / lgsvl/lanefollowing - GitHub, accessed July 24, 2025, https://github.com/lgsvl/lanefollowing/issues/4
73. turlucode/ros-docker-gui: ROS Docker Containers with X11 (GUI) support [Linux] - GitHub, accessed July 24, 2025, https://github.com/turlucode/ros-docker-gui
74. Docker, X11 fails to open display - Ask Ubuntu, accessed July 24, 2025, https://askubuntu.com/questions/1201103/docker-x11-fails-to-open-display
75. nvidia-smi is returning "NVML: Driver/library version mismatch". - Hyperstack Support Hub, accessed July 24, 2025, https://portal.hyperstack.cloud/knowledge/nvidia-smi-is-returning-nvml-driver/library-version-mismatch
76. After upgrade to properietary nvidia 535 driver, "glxinfo" and Firefox show llvmpipe only, but "nvidia-smi" works correctly : r/Ubuntu - Reddit, accessed July 24, 2025, https://www.reddit.com/r/Ubuntu/comments/14pgni5/after_upgrade_to_properietary_nvidia_535_driver/

