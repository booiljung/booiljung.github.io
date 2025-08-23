# PX4-Humble-Harmonic 개발 환경 구축을 위한 종합 가이드


이 보고서는 Ubuntu 22.04 호스트 시스템에서 Visual Studio Code (VS Code)와 DevContainer를 활용하여 PX4-Autopilot, ROS 2 Humble, Gazebo Harmonic을 포함하는 복잡한 로보틱스 개발 환경을 구축하는 절차를 상세히 기술한다. 이 가이드는 단순한 명령어 나열을 넘어, 각 구성 요소의 역할과 상호 작용을 설명하고, 개발 생산성과 보안을 극대화하기 위한 모범 사례를 제시한다. 특히, 멀티스테이지 Docker 빌드를 통해 재현 가능하고 이식성 높은 개발 환경과 최소한의 보안이 강화된 프로덕션 환경을 단일 Dockerfile에서 생성하는 방법을 중점적으로 다룬다. 최종 목표는 VS Code의 통합된 GUI 환경 내에서 코드 빌드, 테스트, 그래픽 시뮬레이션 및 C++ 노드 디버깅까지 원활하게 수행할 수 있는 전문가 수준의 개발 워크플로우를 확립하는 것이다.


컨테이너화된 개발 환경을 성공적으로 운영하기 위해서는 호스트 시스템이 컨테이너에 필요한 자원(GPU, 디스플레이 서버 등)을 안정적으로 제공할 수 있도록 사전에 정확하게 구성하는 것이 무엇보다 중요하다. 이 섹션에서는 Ubuntu 22.04 호스트에 필요한 드라이버, 런타임 및 도구를 설치하고 설정하는 과정을 단계별로 안내한다.


컨테이너 내부에서 GPU 가속을 활용하려면 호스트 시스템에 NVIDIA의 독점(proprietary) 드라이버가 반드시 설치되어 있어야 한다. 컨테이너는 드라이버를 포함하지 않으며, 호스트의 드라이버와 통신하여 GPU 하드웨어를 사용한다.

- 그래픽 드라이버 PPA 추가 및 드라이버 설치:

  안정적인 최신 드라이버를 설치하기 위해 PPA(Personal Package Archive)를 추가하고 권장 버전(예: 550)을 설치한다.

  ```Bash
  sudo add-apt-repository ppa:graphics-drivers/ppa -y
  sudo apt update
  sudo apt install nvidia-driver-550 -y
  ```

- 시스템 재부팅:

  드라이버 설치를 완료하고 시스템에 적용하기 위해 재부팅이 필요하다.

  ```Bash
  sudo reboot
  ```

- 설치 확인:

  재부팅 후, 터미널에서 nvidia-smi 명령어를 실행하여 드라이버가 성공적으로 로드되었고 GPU가 올바르게 인식되는지 확인한다. 이 명령어는 GPU 모델, 드라이버 버전, CUDA 버전 등의 정보를 출력해야 한다.


Docker는 컨테이너화 기술의 핵심 엔진이다. 최신 안정 버전을 사용하기 위해 Docker의 공식 리포지토리에서 직접 설치한다.

- Docker 설치:

  Docker 공식 편의 스크립트(convenience script)를 사용하여 간단하게 설치를 진행한다.

  ```Bash
  curl https://get.docker.com | sh
  ```

- 설치 후 보안 설정 (Post-installation):

  보안 모범 사례에 따라, sudo 없이 Docker 명령어를 실행할 수 있도록 현재 사용자를 docker 그룹에 추가한다. 이는 모든 Docker 관련 작업을 루트 권한으로 실행하는 것을 방지한다.

  ```Bash
  sudo usermod -aG docker $USER
  ```

  이 그룹 설정은 새로운 로그인 세션부터 적용되므로, 시스템을 재부팅하거나 로그아웃 후 다시 로그인해야 한다.


이 툴킷은 Docker 엔진과 호스트의 NVIDIA 드라이버를 연결하는 필수적인 다리 역할을 한다. 컨테이너가 GPU에 접근할 수 있도록 런타임에 필요한 장치 파일과 라이브러리를 주입한다.

- NVIDIA 리포지토리 설정 및 툴킷 설치:

  GPG 키와 소스 리스트를 설정한 후 nvidia-container-toolkit 패키지를 설치한다.

  ```Bash
  curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
  sudo apt-get update
  sudo apt-get install -y nvidia-container-toolkit
  ```

- Docker와 통합:

  Docker가 NVIDIA 런타임을 사용하도록 설정 파일을 수정하고 Docker 데몬을 재시작한다. 이 단계는 Docker에게 --gpus와 같은 플래그를 어떻게 처리해야 하는지 알려준다.

  ```Bash
  sudo nvidia-ctk runtime configure --runtime=docker
  sudo systemctl restart docker
  ```

- 설치 확인:

  테스트용 CUDA 컨테이너를 실행하여 컨테이너가 호스트 GPU를 성공적으로 인식하고 통신할 수 있는지 최종 확인한다. nvidia-smi의 출력이 호스트에서 실행했을 때와 유사하게 나타나야 한다.

  ```Bash
  sudo docker run --rm --gpus all nvidia/cuda:12.2.0-base-ubuntu22.04 nvidia-smi
  ```


VS Code는 주력 IDE이며, `ms-vscode-remote.remote-containers` 확장 프로그램은 컨테이너 기반 개발 경험을 조율하는 핵심 도구다. VS Code를 설치한 후, 확장 프로그램 마켓플레이스에서 "Dev Containers"를 검색하여 설치한다.


컨테이너에서 실행되는 Gazebo나 RViz2와 같은 GUI 애플리케이션을 호스트 화면에 표시하려면, 컨테이너의 X11 클라이언트가 호스트의 X 서버에 연결할 수 있도록 허용해야 한다.

`xhost +local:` 명령어를 호스트 터미널에서 실행한다. 이 명령어는 `xhost +`보다 안전한 대안으로, 네트워크 전체에 X 서버를 개방하는 대신 로컬 연결(Docker 컨테이너 포함)만을 허용한다. 이 설정은 호스트 세션마다 한 번씩 실행해야 할 수 있다.

```Bash
xhost +local:
```

다음 표는 호스트 시스템 준비 과정을 요약한 체크리스트다. 이 단계를 모두 완료하고 확인해야 다음 단계에서 발생할 수 있는 문제를 예방할 수 있다.

**표 1: 호스트 시스템 사전 요구사항 및 확인**

| 구성 요소                | 설치/설정 명령어 (요약)                     | 확인 명령어                                | 목적 및 참고사항                                             |
| ------------------------ | ------------------------------------------- | ------------------------------------------ | ------------------------------------------------------------ |
| NVIDIA 드라이버          | `sudo apt install nvidia-driver-550`        | `nvidia-smi`                               | 컨테이너에 GPU 하드웨어를 노출시키기 위한 필수 드라이버.     |
| Docker 엔진              | `curl https://get.docker.com                | sh`                                        | `docker --version`                                           |
| Docker 그룹 설정         | `sudo usermod -aG docker $USER`             | `groups`                                   | `sudo` 없이 Docker 명령을 실행하기 위한 보안 설정 (재로그인 필요). |
| NVIDIA Container Toolkit | `sudo apt install nvidia-container-toolkit` | `docker run --rm --gpus all... nvidia-smi` | Docker가 GPU를 인식하고 사용하도록 하는 브릿지.              |
| VS Code + 확장           | VS Code 설치 후 마켓플레이스에서 설치       | -                                          | `ms-vscode-remote.remote-containers` 확장 설치.              |
| X11 접근 허용            | `xhost +local:`                             | -                                          | 컨테이너의 GUI 앱을 호스트에 표시하기 위한 권한 설정.        |



`docker build` 명령이 실행될 때, 현재 디렉토리의 모든 내용(빌드 컨텍스트)이 Docker 데몬으로 전송됩니다. 빌드 속도를 높이고 민감한 정보가 이미지에 포함되는 것을 방지하기 위해, 프로젝트 루트에 `.dockerignore` 파일을 생성하는 것이 필수적이다.

```
# Git files
.git
.gitignore

# VSCode and DevContainer settings (handled by devcontainer.json)
#.vscode
#.devcontainer

# Build artifacts
build/
install/
log/

# Python cache
*.pyc
__pycache__/

# Other
*.swp
README.md
```


멀티스테이지 빌드는 Dockerfile 내에 여러 개의 `FROM` 구문을 사용하여 빌드 과정을 여러 단계로 분리하는 기법이다. 초기 단계(예: `builder`)에서는 소스 코드를 컴파일하고 의존성을 설치하며, 후속 단계에서는 `COPY --from=builder` 구문을 사용하여 이전 단계에서 생성된 결과물(실행 파일, 라이브러리 등)만 선택적으로 복사한다. 이를 통해 최종 이미지에는 컴파일러, 소스 코드, 중간 파일 등이 포함되지 않아 이미지 크기가 획기적으로 줄어들고 보안이 강화된다.


아래는 개발 및 프로덕션 환경 구축을 위한 전체 멀티스테이지 Dockerfile이다. 각 단계에 대한 상세 설명은 이어지는 부제에서 다룬다.

```Dockerfile
# syntax=docker/dockerfile:1.4

# ==============================================================================
# Stage 1: base - 모든 스테이지를 위한 공통 기반 설정
#
# 이 스테이지는 모든 후속 스테이지에서 공유되는 공통 구성을 정의한다.
# (환경 변수, 로케일, non-root 사용자 생성 등)
# 이를 통해 코드 중복을 방지하고 Docker 빌드 캐시 효율성을 극대화한다.
# ==============================================================================
FROM ubuntu:22.04 AS base

# --- 공통 환경 설정 ---
# 패키지 설치 시 대화형 프롬프트를 방지하고, 시간대 및 로케일을 일관되게 설정
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Asia/Seoul
RUN apt-get update && \
    apt-get install -y --no-install-recommends locales && \
    locale-gen en_US.UTF-8 && \
    update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8 && \
    rm -rf /var/lib/apt/lists/*
ENV LANG=en_US.UTF-8

# --- Non-Root 사용자 생성 스크립트 ---
# 이 스크립트는 후속 스테이지에서 non-root 사용자를 생성하는 데 사용된다.
# ARG를 사용하여 빌드 시 호스트의 UID/GID와 일치시킬 수 있다.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID
RUN groupadd --gid $USER_GID $USERNAME && \
    useradd --uid $USER_UID --gid $USER_GID -m $USERNAME --shell /bin/bash

# ==============================================================================
# Stage 2: builder - 모든 빌드 활동을 위한 통합 스테이지
#
# 이 스테이지는 CUDA 개발 이미지를 기반으로 하며, PX4, ROS 2, Gazebo 등
# 모든 의존성을 설치하고 소스 코드를 컴파일한다.
# 여기서 생성된 아티팩트(바이너리, 라이브러리)는 다른 스테이지로 복사된다.
# ==============================================================================
FROM nvidia/cuda:12.2.2-devel-ubuntu22.04 AS builder

# --- base 스테이지에서 공통 환경 설정 복사 ---
# ENV, LANG 등 base에서 설정한 환경을 그대로 가져온다.
COPY --from=base /etc/environment /etc/environment
COPY --from=base /etc/locale.gen /etc/locale.gen
COPY --from=base /etc/default/locale /etc/default/locale
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Asia/Seoul
ENV LANG=en_US.UTF-8

# --- base 스테이지에서 생성된 사용자 상속 ---
# base 스테이지에서 생성한 사용자의 홈 디렉토리와 passwd/group 정보를 복사한다.
COPY --from=base /etc/passwd /etc/passwd
COPY --from=base /etc/group /etc/group
COPY --from=base /etc/shadow /etc/shadow
COPY --from=base --chown=vscode:vscode /home/vscode /home/vscode

# --- 모든 빌드 타임 의존성 설치 ---
# 단일 RUN 레이어로 결합하여 이미지 크기를 최적화하고 빌드 속도를 향상시킨다.
# ccache: C/C++ 컴파일 캐시 도구로, 반복적인 빌드 시간을 획기적으로 단축시킨다.
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential cmake ccache ninja-build git curl wget \
    python3-pip python3-venv python3-dev python3-setuptools python3-wheel \
    x11-apps libgl1-mesa-glx libglu1-mesa terminator \
    ca-certificates gnupg gosu lsb-release software-properties-common \
    astyle bc cppcheck file gdb lcov libfuse2 libssl-dev libxml2-dev libxml2-utils \
    make rsync shellcheck unzip zip sudo \
    && echo "vscode ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/vscode \
    && chmod 0440 /etc/sudoers.d/vscode \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# --- ROS 2 Humble 및 Gazebo Harmonic 설치 ---
RUN curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null && \
    wget https://packages.osrfoundation.org/gazebo.gpg -O /usr/share/keyrings/pkgs-osrf-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/pkgs-osrf-archive-keyring.gpg] http://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/gazebo-stable.list > /dev/null && \
    apt-get update && \
    # 개발에는 GUI 도구와 시뮬레이션 전체가 포함된 desktop 버전을 설치한다.
    apt-get install -y --no-install-recommends \
        ros-humble-desktop \
        ros-dev-tools \
        gz-harmonic \
        ros-humble-ros-gzharmonic \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# --- PX4 Autopilot 소스 빌드 ---
# WORKDIR을 사용하여 명령 실행 위치를 명확히 한다.
WORKDIR /
RUN git clone --recursive https://github.com/PX4/PX4-Autopilot.git /PX4-Autopilot
WORKDIR /PX4-Autopilot
# 특정 버전을 체크아웃하여 빌드의 재현성을 보장한다.
RUN git checkout v1.15.4 && \
    git submodule update --init --recursive && \
    pip3 install --no-cache-dir -r./Tools/setup/requirements.txt
# ccache를 사용하여 SITL 타겟을 빌드한다.
RUN ccache make px4_sitl

# --- 사용자 ROS 2 워크스페이스 빌드 (예시) ---
WORKDIR /
RUN mkdir -p /ros2_ws/src
# 실제 프로젝트에서는 이 부분에 소스 코드를 COPY한다.
# COPY --chown=vscode:vscode./my_ros_app_src /ros2_ws/src/my_ros_app
# RUN. /opt/ros/humble/setup.sh && cd /ros2_ws && colcon build --symlink-install

# ==============================================================================
# Stage 3: development - 대화형 개발을 위한 기능이 풍부한 이미지
#
# 이 스테이지는 `builder` 스테이지를 그대로 상속받는다.
# 모든 소스 코드, 빌드 도구, 컴파일러, 의존성이 포함되어 있다.
# `devcontainer.json`에서 이 스테이지를 타겟으로 지정한다.
# ==============================================================================
FROM builder AS development

# --- 최종 환경 설정 ---
USER vscode
WORKDIR /ros2_ws

# entrypoint 스크립트를 복사하고 실행 권한을 부여한다.
# 이 스크립트는 컨테이너 시작 시 ROS 및 기타 환경을 자동으로 설정한다.
COPY --chown=vscode:vscode .devcontainer/entrypoint.sh /usr/local/bin/entrypoint.sh
RUN sudo chmod +x /usr/local/bin/entrypoint.sh

ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
CMD ["/bin/bash"]

# ==============================================================================
# Stage 4: production - 최소화되고 안전하며 최적화된 최종 이미지
#
# 이 스테이지는 최소한의 CUDA `runtime` 이미지를 기반으로 한다.
# `builder` 스테이지에서 컴파일된 실행 파일만 선택적으로 복사하여
# 이미지 크기를 최소화하고 보안을 강화한다.
# ==============================================================================
FROM nvidia/cuda:12.2.2-runtime-ubuntu22.04 AS production

# --- base 스테이지에서 공통 환경 설정 복사 ---
COPY --from=base /etc/environment /etc/environment
COPY --from=base /etc/locale.gen /etc/locale.gen
COPY --from=base /etc/default/locale /etc/default/locale
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Asia/Seoul
ENV LANG=en_US.UTF-8

# --- base 스테이지에서 생성된 사용자 상속 (sudo 제외) ---
COPY --from=base /etc/passwd /etc/passwd
COPY --from=base /etc/group /etc/group
COPY --from=base /etc/shadow /etc/shadow
COPY --from=base --chown=vscode:vscode /home/vscode /home/vscode

# --- 런타임 의존성 설치 ---
# 애플리케이션 실행에 필요한 최소한의 패키지만 설치한다.
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl gnupg lsb-release && \
    curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null && \
    apt-get update && \
    # 프로덕션에서는 GUI가 없는 `ros-base`를 설치하여 이미지 크기를 줄인다.
    apt-get install -y --no-install-recommends \
        ros-humble-ros-base \
        libfuse2 \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# --- Builder 스테이지에서 아티팩트 복사 ---
# 멀티스테이지 빌드의 핵심: 컴파일된 결과물만 최종 이미지로 가져온다.
COPY --from=builder /PX4-Autopilot/build/px4_sitl_default/bin/px4 /usr/local/bin/px4
# 예시: 사용자의 컴파일된 ROS 2 애플리케이션 복사
# COPY --from=builder --chown=vscode:vscode /ros2_ws/install /opt/ros2_ws/install
# 개발 환경과 동일한 entrypoint 스크립트를 사용한다.
COPY --from=development --chown=vscode:vscode /usr/local/bin/entrypoint.sh /usr/local/bin/entrypoint.sh


# --- 최종 환경 설정 ---
USER vscode
WORKDIR /home/vscode

ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
# 실제 배포 시에는 애플리케이션 실행 명령으로 교체해야 한다.
# 예: CMD ["ros2", "launch", "my_ros_app", "my_app.launch.py"]
CMD ["/bin/bash"]
```


- **Stage 1: `base-deps` - 공통 기반:** `nvidia/cuda:12.2.2-devel-ubuntu22.04` 이미지를 기반으로 시작한다. `devel` 태그는 CUDA 툴킷과 컴파일러가 포함되어 있어 CUDA 가속 코드 빌드 및 Gazebo와 같은 도구에 필수적이다. ROS 2의 요구사항에 따라 UTF-8 로케일을 설정하고, `build-essential`, `cmake`, GUI 렌더링에 필요한 `libgl1-mesa-glx` 등을 설치한다.
- **Stage 2: `px4-deps` - 선언적 의존성 관리:** 사용자의 명시적 요구사항에 따라 `ubuntu.sh` 스크립트를 실행하는 대신, 스크립트가 설치하는 패키지들을 `apt-get install` 명령어로 직접 명시한다. 이는 PX4 v1.15.x 버전의 `ubuntu.sh` 스크립트와 관련 GitHub 이슈를 분석하여 도출된 의존성 목록이다. 이러한 선언적 방식은 환경의 투명성과 재현성을 보장하며, 이는 현대 DevOps의 핵심 원칙인 '코드로서의 인프라(Infrastructure as Code)'를 실천하는 것이다.
- **Stage 3: `robotics-stack` - ROS 2와 Gazebo의 비표준 조합:** 이 단계에서는 ROS 2 Humble과 Gazebo Harmonic을 설치한다. 표준 `ros:humble-desktop` 이미지는 Gazebo Fortress를 포함하므로, `apt` 패키지 충돌을 피하기 위해 `base-deps`에서부터 직접 스택을 구축한다. OSRF 리포지토리를 추가하여 `gz-harmonic`을 설치한 후, `ros-humble-ros-gzharmonic` 패키지를 설치하는 것이 핵심이다. 이 패키지는 ROS 2 Humble과 Gazebo Harmonic을 연결하는 브릿지 역할을 한다.
- **Stage 4: `px4-builder` - PX4 Autopilot 컴파일:** `git`을 사용하여 PX4-Autopilot 소스 코드를 클론하고, `v1.15.4` 태그로 정확히 체크아웃한 후 모든 서브모듈을 초기화한다. `make px4_sitl`을 실행하여 Software-in-the-loop(SITL) 펌웨어를 사전 컴파일한다.
- **Stage 5: `dev-image` - 최종 개발 환경:** 이 이미지는 `px4-builder`를 기반으로 하여 모든 개발 도구와 소스 코드를 포함한다. 보안 강화를 위해 UID/GID가 1000인 `vscode`라는 non-root 사용자를 생성하고, 패키지 설치 등 개발 편의성을 위해 암호 없는 `sudo` 권한을 부여한다. `USER vscode` 명령어로 기본 사용자를 전환하여 컨테이너 내부의 모든 프로세스가 non-root 권한으로 실행되도록 한다. 마지막으로, 컨테이너 시작 시 ROS 2 및 PX4 환경 변수를 자동으로 설정해주는 `entrypoint.sh` 스크립트를 구성한다.
- **Stage 6: `prod-image` - 최소 프로덕션 환경:** `ros:humble-ros-base`와 같이 컴파일러나 GUI 도구가 없는 최소한의 이미지를 기반으로 한다. `dev-image`와 동일한 non-root 사용자를 생성하여 보안 일관성을 유지한다. `COPY --from=px4-builder`를 사용하여 `px4-builder` 단계에서 컴파일된 `px4` 실행 파일과 같이 필요한 아티팩트만 복사한다.

다음 표는 멀티스테이지 빌드를 통해 생성된 개발 이미지와 프로덕션 이미지의 주요 차이점을 비교 분석한 것이다.

**표 2: 개발용 이미지와 프로덕션용 이미지 비교 분석**

| 특성           | 개발용 이미지 (dev-image)               | 프로덕션용 이미지 (prod-image)           |
| -------------- | --------------------------------------- | ---------------------------------------- |
| 기반 이미지    | `nvidia/cuda:12.2.2-devel-ubuntu22.04`  | `ros:humble-ros-base`                    |
| 예상 크기      | 수 GB ~ 수십 GB                         | 수백 MB ~ 수 GB                          |
| 포함된 도구    | 컴파일러(GCC, NVCC), GDB, CMake, Git 등 | 없음 (최소 런타임 라이브러리만 포함)     |
| 소스 코드      | PX4 전체 소스 코드 포함                 | 포함되지 않음 (컴파일된 바이너리만 존재) |
| 보안 공격 표면 | 넓음 (다양한 개발 도구 존재)            | 최소화됨 (불필요한 모든 요소 제거)       |
| 주요 사용 사례 | 코드 개발, 빌드, 디버깅, 시뮬레이션     | 실제 로봇에 배포, 애플리케이션 실행      |


`devcontainer.json` 파일은 VS Code에게 Dockerfile로 정의된 컨테이너를 어떻게 빌드하고 실행하며 상호작용할지를 지시하는 제어판과 같다. 이 파일을 통해 하드웨어 가속, GUI 포워딩, IDE 확장 프로그램 설치 등을 자동화하여 원클릭으로 완벽한 개발 환경을 구성할 수 있다.


```JSON
{
    "name": "PX4-Humble-Harmonic-Dev",
    "build": {
        "dockerfile": "./Dockerfile",
        "context": "..",
        "target": "development",
        "args": {
        }
    },
    // ======================================================================
    // 호스트-컨테이너 폴더 연결 설정 (핵심 부분)
    // ======================================================================
    // 1. 호스트의 프로젝트 폴더를 컨테이너의 특정 경로에 마운트(연결)
    //    - source=${localWorkspaceFolder}: 호스트의 현재 프로젝트 폴더
    //    - target=/ros2_ws: 컨테이너 내부의 연결될 폴더
    "workspaceMount": "source=${localWorkspaceFolder},target=/ros2_ws,type=bind,consistency=cached",
    // 2. 컨테이너에 접속했을 때 VS Code가 열람할 기본 폴더
    //    workspaceMount의 target과 일치시켜야 함
    "workspaceFolder": "/ros2_ws",
    "runArgs": [
        "--gpus=all",
        "--network=host",
        "--ipc=host",
        "-v", "/tmp/.X11-unix:/tmp/.X11-unix:rw",
        "--volume=/dev/dri:/dev/dri"
    ],
    "containerEnv": {
        "DISPLAY": "${localEnv:DISPLAY}",
        "NVIDIA_VISIBLE_DEVICES": "all",
        "NVIDIA_DRIVER_CAPABILITIES": "all",
        "QT_X11_NO_MITSHM": "1"
    },
    "remoteUser": "vscode",
    "customizations": {
        "vscode": {
            "settings": {
                "terminal.integrated.defaultProfile.linux": "bash",
                "python.defaultInterpreterPath": "/usr/bin/python3"
            },
            "extensions": [
                "google.geminicodeassist",
                "mhutchie.git-graph",
                "ms-vscode.cpptools-extension-pack",
                "ms-iot.vscode-ros",
                "ms-vscode.cmake-tools",                
                "redhat.vscode-xml",
                "streetsidesoftware.code-spell-checker",
                "eamodio.gitlens",
                "dotjoshjohnson.xml",
                "ms-azuretools.vscode-containers"
            ]
        }
    },
    "postCreateCommand": "if [ ! -f /etc/ros/rosdep/sources.list.d/20-default.list ]; then sudo rosdep init; fi && rosdep update"
}
```


- **`build`:** 이 객체는 VS Code에게 컨테이너 빌드 방법을 알려준다.

  - `"dockerfile": "../Dockerfile"`: 사용할 Dockerfile의 경로를 지정한다.
  - `"target": "dev-image"`: 핵심 설정이다. 멀티스테이지 Dockerfile에서 `dev-image`라는 이름의 단계까지만 빌드하도록 지시한다. 이를 통해 개발 시에는 프로덕션 이미지 빌드 과정을 건너뛰어 시간을 절약한다.

- **`runArgs`:** VS Code가 `docker run` 명령을 실행할 때 전달할 인자들의 배열이다. 하드웨어 및 GUI 기능 활성화의 핵심이다.

  - `"--gpus=all"`: 호스트의 모든 GPU를 컨테이너에 할당하여 CUDA 및 OpenGL 가속을 활성화한다.
  - `"--network=host"`: 컨테이너가 호스트의 네트워크 스택을 직접 사용하게 한다. 이는 ROS 2 노드 간의 DDS 통신과 X11 포워딩을 가장 간단하고 안정적으로 설정하는 방법이다.
  - `"-v", "/tmp/.X11-unix:/tmp/.X11-unix:rw"`: 호스트의 X11 소켓 디렉토리를 컨테이너에 마운트하여 GUI 애플리케이션이 호스트의 디스플레이 서버에 연결되도록 한다.
  - `"--volume=/dev/dri:/dev/dri"`: Direct Rendering Infrastructure(DRI) 장치를 마운트하여 Gazebo와 같은 애플리케이션에서 OpenGL 3.3 이상을 지원하고 하드웨어 렌더링을 가능하게 한다.

- **환경 및 사용자 설정:**

  - `"remoteUser": "vscode"`: VS Code 서버 프로세스와 터미널이 Dockerfile에서 생성한 non-root 사용자인 `vscode`로 실행되도록 지정한다. 이는 파일 권한 문제 해결과 보안에 매우 중요하다.
  - `"containerEnv"`: 컨테이너 내부에 환경 변수를 설정한다.
    - `"DISPLAY": "${localEnv:DISPLAY}"`: 컨테이너의 `DISPLAY` 변수를 호스트의 `DISPLAY` 변수 값으로 동적으로 설정하여 X11 포워딩의 안정성을 높인다.
    - `"QT_X11_NO_MITSHM": "1"`: RViz2나 Gazebo와 같이 Qt 기반 GUI 애플리케이션에서 발생할 수 있는 렌더링 오류를 방지하여 안정성을 향상시킨다.

- VS Code 커스터마이징 (customizations):

  이 객체는 IDE 자체의 설정을 자동화하여 모든 개발자에게 일관된 경험을 제공한다.

  - `"extensions"`: 컨테이너 내부에 자동으로 설치할 확장 프로그램 ID의 목록이다. 이를 통해 C/C++ 개발, ROS 지원, CMake 도구 등을 즉시 사용할 수 있다.
  - `"settings"`: 터미널 프로필이나 포맷터 등 워크스페이스에 적용할 기본 VS Code 설정을 정의한다.

- **`mounts`:** 호스트의 프로젝트 폴더를 컨테이너 내부의 `/home/vscode/ros2_ws`로 마운트한다. 이를 통해 호스트에서 코드를 수정하면 즉시 컨테이너에 반영되어 개발-빌드-테스트 주기를 단축할 수 있다.


컨테이너가 시작될 때 ROS 2 및 PX4 환경 변수를 자동으로 설정하여 터미널을 열 때마다 수동으로 `source` 명령을 실행할 필요가 없도록 합니다.

```Bash
#!/bin/bash
set -e

# ROS 2 환경 설정
source /opt/ros/humble/setup.bash

# PX4 Gazebo 환경 설정
source /PX4-Autopilot/Tools/setup_gazebo.bash /PX4-Autopilot /PX4-Autopilot/build/px4_sitl_default

# ROS 2 워크스페이스 환경 설정 (존재할 경우)
if [ -f /ros2_ws/install/setup.bash ]; then
  source /ros2_ws/install/setup.bash
fi

echo "ROS 2 Humble & PX4 Environment Sourced"

# 전달된 명령 실행
exec "$@"
```

**표 3: DevContainer를 위한 권장 VS Code 확장 프로그램**

| 확장 프로그램 이름   | 확장 프로그램 ID                        | 프로젝트에서의 역할                                          |
| -------------------- | --------------------------------------- | ------------------------------------------------------------ |
| C/C++ Extension Pack | `ms-vscode.cpptools-extension-pack`     | C++ 코드에 대한 IntelliSense, 디버깅, 코드 브라우징 제공.    |
| ROS                  | `ms-iot.vscode-ros`                     | ROS 2 빌드(colcon), 실행, 디버깅, 시각화 등 핵심 기능 지원.  |
| CMake Tools          | `ms-vscode.cmake-tools`                 | CMake 프로젝트에 대한 향상된 IntelliSense 및 빌드/디버그 지원. |
| Code Spell Checker   | `streetsidesoftware.code-spell-checker` | 코드와 주석의 오타를 검사하여 코드 품질 향상.                |
| GitLens              | `eamodio.gitlens`                       | 코드 라인별 Git 기록(blame), 커밋 정보 등을 시각화하여 협업 효율 증대. |


완벽하게 구성된 환경을 실제로 사용하는 방법을 안내한다. 이 섹션에서는 VS Code의 `tasks.json`과 `launch.json` 파일을 활용하여 사용자의 핵심 요구사항인 GUI 기반의 원활한 빌드 및 디버깅 워크플로우를 구현하는 데 초점을 맞춘다.


`tasks.json` 파일은 VS Code 내에서 실행할 수 있는 사용자 정의 빌드 및 실행 명령을 정의합니다. 이를 통해 터미널에 직접 명령어를 입력할 필요 없이 단축키나 명령어 팔레트를 통해 빌드 및 시뮬레이션 실행을 수행할 수 있습니다.

```JSON
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "colcon build (debug)",
            "type": "shell",
            "command": "source /opt/ros/humble/setup.bash && colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Debug",
            "group": "build",
            "problemMatcher": ["$gcc"]
        },
        {
            "label": "PX4: Launch SITL",
            "type": "shell",
            "command": "make -C /PX4-Autopilot px4_sitl_default gazebo",
            //"problemMatcher": 
        }
    ]
}
```

- **`colcon build (debug)`:** 디버깅을 위한 빌드 태스크다. `--cmake-args -DCMAKE_BUILD_TYPE=Debug` 인자를 추가하여 컴파일러가 바이너리에 디버깅 심볼을 포함하도록 지시한다. 디버거가 소스 코드와 실행 파일을 매핑하기 위해 이 심볼은 필수적이다.
- **`PX4: Launch SITL`:** PX4 SITL과 Gazebo 시뮬레이터를 한 번에 실행하는 태스크다. 이를 통해 개발자는 VS Code 내에서 시뮬레이션을 빠르게 시작할 수 있다.


`launch.json` 파일은 VS Code의 디버거를 설정하는 가장 중요한 파일이다. ROS 2 런치 파일로 시작되는 C++ 노드를 원클릭으로 디버깅하는 환경을 구성한다. 이 설정의 핵심은 VS Code ROS 확장 프로그램이 제공하는 고수준 추상화인 `"type": "ros"`를 사용하는 것이다.

```JSON
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "ROS2 Launch File",
            "type": "ros",
            "request": "launch",
            "target": "/path/to/your/ros2_ws/src/your_package_name/launch/your_launch_file.launch.py"
        },
        {
            "name": "Attach to C++ Node",
            "type": "cppdbg",
            "request": "attach",
            "program": "/path/to/your/ros2_ws/install/your_package_name/lib/your_package_name/your_cpp_node_name",
            "processId": "${command:pickProcess}",
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ]
        }
    ]
}
```

- **`"type": "ros"`:** VS Code가 ROS 확장 프로그램의 디버그 제공자를 사용하도록 지시한다.
- **`"target"`:** 디버깅할 ROS 2 런치 파일의 전체 경로를 지정한다.
- **`"launch": ["gzserver", "gzclient"]"`:** 강력하고 유용한 옵션이다. 디버거가 Gazebo 관련 프로세스(`gzserver`, `gzclient`)는 단순히 실행만 하고 디버거를 연결하지 않도록 설정한다. 이를 통해 개발자는 자신이 작성한 커스텀 노드에만 집중하여 디버깅할 수 있다.

**디버깅 워크플로우:**

1. 디버깅할 C++ 노드의 소스 코드에 중단점(breakpoint)을 설정한다.
2. VS Code의 터미널에서 "colcon build (debug)" 태스크를 실행하여 디버그 심볼이 포함된 빌드를 완료한다.
3. VS Code의 "실행 및 디버그(Run and Debug)" 패널로 이동한다.
4. 드롭다운 메뉴에서 "ROS: Launch and Debug" 구성을 선택하고 F5 키를 누르거나 녹색 시작 버튼을 클릭한다.
5. ROS 확장 프로그램이 지정된 런치 파일을 실행하고, C++ 노드 프로세스를 식별하여 GDB를 자동으로 연결한다. 코드 실행은 설정된 중단점에서 멈추게 되며, 이때부터 변수 확인, 스택 추적 등 모든 디버깅 기능을 사용할 수 있다.


모든 설정이 올바르게 완료되었는지 확인하는 가장 확실한 방법은 컨테이너 내에서 GUI 애플리케이션을 직접 실행해보는 것이다.

1. VS Code에서 통합 터미널을 연다 (이 터미널은 컨테이너 내부의 셸이다).
2. `rviz2`를 입력하고 실행한다. 호스트 데스크톱에 RViz2 창이 나타나야 한다.
3. `gz sim`을 입력하고 실행한다. 호스트 데스크톱에 Gazebo 시뮬레이터 창이 나타나야 한다.

이 두 애플리케이션이 성공적으로 실행되면, `devcontainer.json`에 설정한 X11 포워딩과 GPU 가속이 모두 정상적으로 작동하고 있음을 의미한다.


이 마지막 섹션에서는 프로덕션 수준의 보안과 별도의 프로덕션 컨테이너에 대한 사용자의 요구사항을 종합적으로 다룬다. Dockerfile 전반에 걸쳐 구현된 보안 관행을 요약하고, 실제 배포를 위한 명확한 경로를 제시한다.


본 구성의 핵심 보안 요소는 non-root 사용자(`vscode`)를 기본으로 사용하는 것이다. 이는 최소 권한 원칙을 적용한 것으로, 컨테이너 내부의 애플리케이션이 침해되더라도 공격자는 컨테이너 내에서 루트 권한을 갖지 못한다. 따라서 악성 패키지를 즉시 설치하거나 시스템 파일을 수정하는 등의 추가 공격을 어렵게 만들어 피해를 최소화할 수 있다. 개발 환경에서는 `sudo` 권한을 부여하여 편의성을 높였지만, 프로덕션 이미지에서는 이 `sudo` 권한마저 제거하여 보안을 더욱 강화했다.


멀티스테이지 Dockerfile 덕분에 개발과 프로덕션 빌드를 동일한 파일에서 관리할 수 있다. 프로덕션용으로 설계된 최소화된 `prod-image`를 빌드하려면 다음 명령어를 사용한다.

```Bash
docker build --target prod-image -t my-drone-app:prod-1.0.
```

`--target prod-image` 플래그는 Docker에게 `prod-image` 단계에서 빌드를 중단하라고 지시한다. 이 명령어 하나로 개발에 사용된 모든 도구와 소스 코드가 배제된, 작고 안전한 이미지가 생성된다.


개발 이미지와 프로덕션 이미지를 비교하면 멀티스테이지 빌드의 이점이 명확해진다 (표 2 참조).

- **크기 감소:** 컴파일러, 디버거, 소스 코드 등이 모두 제거되어 이미지 크기가 수 GB에서 수백 MB로 극적으로 감소한다. 이는 배포 속도를 높이고 스토리지 비용을 절감한다.
- **공격 표면 감소:** 이미지에 포함된 라이브러리와 실행 파일의 수가 적을수록 공격자가 악용할 수 있는 잠재적 취약점도 줄어든다.
- **관심사 분리:** 빌드 시에만 필요한 비밀(예: private 리포지토리 접근 토큰)이나 의존성이 최종 결과물에 포함되지 않아 더욱 안전하다.


환경의 보안을 검증하는 것은 매우 중요하다. Trivy는 컨테이너 이미지의 OS 패키지 및 애플리케이션 의존성에서 알려진 보안 취약점을 스캔하는 간단하고 포괄적인 오픈소스 도구이다.

프로덕션 이미지를 빌드한 후, 다음 명령어를 사용하여 `HIGH` 또는 `CRITICAL` 등급의 심각한 취약점이 있는지 스캔할 수 있다.

```Bash
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
    -v $HOME/trivy-cache:/root/.cache/ \
    aquasec/trivy:latest image --severity HIGH,CRITICAL my-drone-app:prod-1.0
```

이러한 스캐닝을 CI/CD 파이프라인에 통합하면, 취약한 이미지가 레지스트리에 푸시되는 것을 원천적으로 차단하여 개발 수명 주기를 훨씬 더 견고하게 만들 수 있다.


이 보고서에서 제시한 개발 환경 구성은 VS Code DevContainer를 중심으로 최신 로보틱스 개발의 모범 사례를 통합한 결과물이다. 멀티스테이지 Dockerfile을 통해 개발과 프로덕션의 요구사항을 동시에 만족시키는 재현 가능하고 안전한 환경을 구축했으며, `devcontainer.json`을 통해 복잡한 하드웨어 가속 및 IDE 설정을 자동화했다. 또한, `tasks.json`과 `launch.json`을 활용하여 VS Code 내에서 빌드부터 C++ 노드 디버깅까지 끊김 없는 워크플로우를 구현했다.

이 구성은 단순한 개발 환경 설정을 넘어, 전문적인 DevOps 파이프라인의 첫 단계를 제시한다. 여기서 정의된 Dockerfile은 개발자의 로컬 머신에서부터 시작하여 지속적 통합(CI) 서버를 거쳐 최종적으로 실제 로봇에 배포되는 핵심 아티팩트(artifact) 역할을 할 수 있다. 이 가이드를 따름으로써 개발팀은 환경 불일치로 인한 문제를 해결하고, 새로운 팀원의 온보딩 시간을 단축하며, 궁극적으로는 더 빠르고 안정적으로 로봇 애플리케이션을 개발하고 배포할 수 있을 것이다.


1. Installing/Configuring Nvidia Container Toolkit for Docker on Linux Mint 21.1 (Based on Ubuntu 22.04) - GitHub Gist, accessed July 26, 2025, https://gist.github.com/practical-dreamer/79c26931d99bc6a7f28271b3612907a9
2. Dev Containers - Part 1: Quick Start - Basic Setup and Usage, accessed July 26, 2025, https://dev.to/graezykev/dev-containers-part-1-quick-start-basic-setup-and-usage-51l6
3. Installing the NVIDIA Container Toolkit - NVIDIA Docs Hub, accessed July 26, 2025, https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html
4. Getting Started with Development Containers in VS Code | Ready-to-use Presentations, accessed July 26, 2025, https://microsoft.github.io/workshop-library/full/using-dev-containers-vscode/
5. Running GUI Applications in a Linux Docker Container - Baeldung, accessed July 26, 2025, https://www.baeldung.com/linux/docker-container-gui-applications
6. GUI apps within a Docker container | by samriddhipaliwal | Medium, accessed July 26, 2025, https://medium.com/@paliwalsamriddhi/gui-apps-within-a-docker-container-971681838fda
7. Top 20 Dockerfile best practices - Sysdig, accessed July 26, 2025, https://sysdig.com/learn-cloud-native/dockerfile-best-practices/
8. ros - Official Image | Docker Hub, accessed July 26, 2025, https://hub.docker.com/_/ros
9. Containerize your app using a multi-stage build | Docker Docs, accessed July 26, 2025, https://docs.docker.com/guides/cpp/multistage/
10. psaboia/devcontainer-nvidia-base: Example of how to setup a NVIDIA DevContainer with GPU Support for Tensorflow/Keras, that follows the page https://alankrantas.medium.com/setup-a-nvidia-devcontainer-with-gpu-support-for-tensorflow-keras-on-windows-d00e6e204630 - GitHub, accessed July 26, 2025, https://github.com/psaboia/devcontainer-nvidia-base
11. Ubuntu (deb packages) - ROS 2 Documentation: Humble ..., accessed July 26, 2025, https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debs.html
12. PX4-Autopilot/Tools/setup/ubuntu.sh at main - GitHub, accessed July 26, 2025, https://github.com/PX4/PX4-Autopilot/blob/main/Tools/setup/ubuntu.sh
13. [Bug] Installing PX4-Autopilot on Ubuntu22.04 Raspberry Pi 4 fails #21824 - GitHub, accessed July 26, 2025, https://github.com/PX4/PX4-Autopilot/issues/21824
14. gazebo harmonic and ros2 humble : r/ROS - Reddit, accessed July 26, 2025, https://www.reddit.com/r/ROS/comments/1lt0hmw/gazebo_harmonic_and_ros2_humble/
15. Installing Gazebo with ROS - Gazebo ionic documentation, accessed July 26, 2025, https://gazebosim.org/docs/latest/ros_installation/
16. Installing Gazebo with ROS - Gazebo ionic documentation, accessed July 26, 2025, https://gazebosim.org/docs/harmonic/ros_installation
17. PX4/PX4-Autopilot: v1.15.4 - Zenodo, accessed July 26, 2025, https://zenodo.org/records/14854151
18. PX4 Autopilot Software - GitHub, accessed July 26, 2025, https://github.com/PX4/PX4-Autopilot
19. Add a non-root user to a container - Visual Studio Code, accessed July 26, 2025, https://code.visualstudio.com/remote/advancedcontainers/add-nonroot-user
20. Leveraging VS Code Devcontainers for Efficient ROS 2 Development, accessed July 26, 2025, https://www.roboticscontentlab.com/2024/10/13/leveraging-vs-code-devcontainers-for-efficient-ros-2-development/
21. An Updated Guide to Docker and ROS 2 - Robotic Sea Bass, accessed July 26, 2025, https://roboticseabass.com/2023/07/09/updated-guide-docker-and-ros2/
22. Introduction to dev containers - Codespaces - GitHub Docs, accessed July 26, 2025, https://docs.github.com/en/codespaces/setting-up-your-project-for-codespaces/adding-a-dev-container-configuration/introduction-to-dev-containers
23. Dev Container metadata reference, accessed July 26, 2025, https://containers.dev/implementors/json_reference/
24. Using GPU in VS code container - Stack Overflow, accessed July 26, 2025, https://stackoverflow.com/questions/72129213/using-gpu-in-vs-code-container
25. Local Development With NVIDIA GPUs - Rendered.ai Support, accessed July 26, 2025, https://support.rendered.ai/development-guides/setting-up-the-development-environment/local-development-with-nvidia-gpus
26. Display forwarding from docker container / Issue #550 / microsoft/vscode-remote-release - GitHub, accessed July 26, 2025, https://github.com/microsoft/vscode-remote-release/issues/550
27. devcontainer, how to make X display work (mount graphics inside docker in visual studio code) - Stack Overflow, accessed July 26, 2025, https://stackoverflow.com/questions/60733288/devcontainer-how-to-make-x-display-work-mount-graphics-inside-docker-in-visual
28. Create a Dev Container - Visual Studio Code, accessed July 26, 2025, https://code.visualstudio.com/docs/devcontainers/create-dev-container
29. ROS 2 and VSCode - PickNik Robotics, accessed July 26, 2025, https://picknik.ai/vscode/docker/ros2/2024/01/23/ROS2-and-VSCode.html
30. ErickKramer/ros2_with_vscode: Vscode setup that I use to work with ROS2 - GitHub, accessed July 26, 2025, https://github.com/ErickKramer/ros2_with_vscode
31. vscode-ros/README.md at master - GitHub, accessed July 26, 2025, https://github.com/ms-iot/vscode-ros/blob/master/README.md
32. Debug ROS2 C++ node on VSCode (Ubuntu) - GitHub Gist, accessed July 26, 2025, https://gist.github.com/JADC362/a4425c2d05cdaadaaa71b697b674425f
33. What do I put in .vscode/launch.json to launch a ros2 C++ application from VSCdoe with debugger attached? - Robotics Stack Exchange, accessed July 26, 2025, https://robotics.stackexchange.com/questions/114934/what-do-i-put-in-vscode-launch-json-to-launch-a-ros2-c-application-from-vscdo
34. Securing Docker: Non-Root User Best Practices | by Kfir Gisman | Medium, accessed July 26, 2025, https://medium.com/@Kfir-G/securing-docker-non-root-user-best-practices-5784ac25e755

