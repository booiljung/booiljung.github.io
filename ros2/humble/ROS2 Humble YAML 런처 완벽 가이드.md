# ROS2 Humble YAML 런처 완벽 가이드

## 1. ROS 2 런치 시스템의 핵심: 왜 런처를 사용하는가?

로보틱스 시스템은 수십 개의 노드가 여러 장비에 걸쳐 동시에 실행되는 복잡한 분산 시스템인 경우가 많다. 이러한 시스템을 관리하고 실행하는 것은 단순한 작업이 아니다. ROS 2 런치 시스템은 바로 이 복잡성을 해결하기 위해 설계된 강력한 도구다.

### 1.1 단일 노드 실행의 한계

ROS 2 튜토리얼을 처음 접하면 보통 `ros2 run <pkg> <exec>` 명령어를 사용하여 각 노드를 별도의 터미널에서 하나씩 실행한다.1 노드가 두세 개에 불과할 때는 이 방법이 가능하지만, 실제 로봇 애플리케이션에서는 금세 비효율적이고 오류가 발생하기 쉬운 방식이 된다.1 수많은 터미널을 열고, 각 노드에 필요한 설정을 반복적으로 입력하며, 전체 시스템을 한 번에 시작하거나 중지할 통합된 방법이 없다는 점은 명백한 한계다.1

### 1.2 ROS 2 런치 시스템의 등장

ROS 2 런치 시스템은 단일 명령어(`ros2 launch`)로 다수의 노드를 자동으로 실행하고 설정하기 위해 도입되었다.2 이는 단순히 여러 개의 `ros2 run` 명령을 묶어놓은 스크립트가 아니다. 런치 시스템은 애플리케이션 전체의 구성을 기술하는 프레임워크다. 어떤 프로그램을 실행할지, 어디서 실행할지, 어떤 인자를 전달할지, 그리고 ROS 고유의 통신 방식을 어떻게 설정하여 컴포넌트를 재사용할지 등을 모두 정의할 수 있다.2

더 나아가, 런치 시스템은 실행된 프로세스의 상태를 모니터링하고 변화에 대응하는 책임까지 진다.2 예를 들어, 특정 노드가 예기치 않게 종료되었을 때 자동으로 재시작하는 기능(ROS 1의 `respawn` 기능과 유사)을 구현할 수 있다.3 이처럼 런치 시스템은 단순한 실행 도구를 넘어 시스템의 생명주기를 관리하는 역할을 수행한다.

### 1.3 런치 파일: 시스템의 청사진

이 모든 설정은 "런치 파일"이라는 단일 파일에 기술된다.2 이 파일은 로봇 소프트웨어 스택의 청사진 역할을 하며, 시스템의 상태를 선언적으로 정의한다. ROS 2에서는 사용자의 필요와 선호에 따라 Python, XML, 그리고 이 보고서의 핵심 주제인 YAML, 세 가지 형식 중 하나로 런치 파일을 작성할 수 있다.2

여기서 중요한 점은 런치 시스템이 시스템의 *설명(description)*과 *실행(execution)*을 분리한다는 것이다. 런치 파일은 "무엇을 어떻게 실행할지"에 대한 정적인 설명을 생성하는 역할을 한다. 그러면 `ros2 launch` 도구가 이 설명을 해석하여 실제 시스템을 구동하고, 설명된 상태를 유지하기 위해 프로세스를 관리한다.5 이러한 아키텍처 덕분에 `ros2 launch --print`와 같은 정적 분석 도구나, 시스템 상태 변화에 따라 동적으로 동작을 변경하는 이벤트 기반의 고급 기능을 사용할 수 있게 된다. 이는 단순한 쉘 스크립트로는 불가능한 강력한 기능이다.

## 2. 런치 파일 형식 선택 가이드: Python vs. XML vs. YAML

ROS 2는 Python, XML, YAML 세 가지 런치 파일 형식을 지원하며, 각기 다른 장단점을 가진다. 어떤 형식을 선택할지는 프로젝트의 복잡성, 팀의 기술 선호도, 그리고 달성하고자 하는 목표에 따라 달라진다.4

### 2.1 Python (`.launch.py`): 최고의 유연성

- **강점**: Python 런치 파일은 완전한 기능을 갖춘 프로그래밍 언어다. 따라서 조건문(`if/else`), 반복문, 파일 입출력, 외부 라이브러리 활용 등 Python의 모든 기능을 런치 로직에 통합할 수 있다.6 또한, `ros2/launch` 및 `ros2/launch_ros` 패키지의 모든 기능에 직접 접근할 수 있어 XML이나 YAML에서는 노출되지 않은 저수준 제어가 가능하다.7
- **약점**: 간단한 작업을 수행하는 데도 코드가 장황하고 복잡해질 수 있다.8 일부 개발자들은 그 문법이 "Python으로 작성된 XML 같다"고 느끼며, Python 고유의 간결함과 직관성을 활용하기 어렵다고 평가하기도 한다.5

### 2.2 XML (`.launch.xml`): 익숙함과 간결함

- **강점**: ROS 1의 표준 형식이었기 때문에 ROS 1에서 마이그레이션하는 개발자에게 매우 익숙하다.6 선언적이고 계층적인 구조는 간단하거나 중간 정도 복잡도의 시스템을 기술할 때 동등한 Python 코드보다 훨씬 가독성이 높고 간결하다.9
- **약점**: Python에 비해 유연성이 떨어진다. 조건부 실행(`if`/`unless`)이나 변수 사용 등은 지원하지만, 복잡한 연산이나 프로그래밍 로직을 구현할 수는 없다.12

### 2.3 YAML (`.launch.yaml`): 최고의 가독성

- **강점**: YAML은 최소한의 문법으로 매우 깔끔하고 사람이 읽기 쉬운 구조를 가진다. 이 때문에 정적인 설정을 기술하는 데 탁월하다.13 기능적으로는 XML과 많은 부분을 공유한다.14
- **약점**: 세 형식 중 유연성이 가장 낮으며, 주로 정적인 시스템 설명에 사용된다. 커뮤니티에서는 Python이나 XML에 비해 공식 문서가 부족하다는 점이 단점으로 지적된다.10 또한, 그 문법이 자연스러운 YAML 관례를 따르기보다는 XML 구조를 그대로 모방하는 경우가 많아 직관적이지 않을 수 있다.15

### 2.4 비교표 및 선택 전략

아래 표는 세 가지 런치 파일 형식의 주요 특징을 요약한 것이다.

| 특징                 | Python (`.launch.py`)                            | XML (`.launch.xml`)                             | YAML (`.launch.yaml`)             |
| -------------------- | ------------------------------------------------ | ----------------------------------------------- | --------------------------------- |
| **문법 스타일**      | 프로그래밍 (명령형)                              | 마크업 (선언형)                                 | 마크업 (선언형)                   |
| **유연성**           | 매우 높음 (조건문, 반복문, 라이브러리 사용 가능) | 중간 (기본적인 조건부 실행 및 변수 지원)        | 낮음 (주로 정적 구성용)           |
| **가독성**           | 낮음 (코드가 장황해질 수 있음)                   | 중간 (계층 구조가 명확함)                       | 매우 높음 (간결하고 깔끔함)       |
| **고급 기능**        | 모든 기능에 직접 접근 가능                       | 제한적으로 노출됨                               | 제한적으로 노출됨                 |
| **문서화 수준**      | 높음                                             | 중간                                            | 낮음                              |
| **주요 사용 사례**   | 동적/조건부 로직이 필요한 복잡한 시스템          | ROS 1에서 마이그레이션, 중간 규모의 정적 시스템 | 간단한 정적 시스템, 파라미터 파일 |
| **ROS 1과의 유사성** | 낮음                                             | 높음                                            | 낮음                              |

단일 형식을 선택하는 것보다 더 효과적인 전략은 **하이브리드 접근법**을 취하는 것이다. ROS 2 런치 시스템의 강력한 기능 중 하나는 한 형식의 런치 파일에서 다른 형식의 런치 파일을 포함(`include`)할 수 있다는 점이다.16 이를 활용하면 각 형식의 장점을 극대화할 수 있다. 예를 들어, 최상위 런치 파일은 Python으로 작성하여 하드웨어 감지나 복잡한 설정 로직을 처리하고, 이 파일이 각 모듈을 실행하는 단순하고 가독성 높은 YAML 또는 XML 런치 파일을 포함하도록 구조화할 수 있다. 이러한 모듈식 설계는 복잡한 시스템의 확장성과 유지보수성을 크게 향상시키는 전문가 수준의 모범 사례다.17

## 3. YAML 런처 시작하기: 기본 구조와 문법

YAML 런처를 사용하기 위해서는 기본적인 파일 구조와 빌드 시스템과의 연동 방식을 이해해야 한다. 이는 "파일을 찾을 수 없음"과 같은 흔한 오류를 피하는 첫걸음이다.

### 3.1 패키지 구조: 관례와 모범 사례

ROS 2 패키지 내에서 런치 관련 파일들은 정해진 관례에 따라 배치하는 것이 좋다.

- **`launch/` 디렉터리**: 모든 런치 파일(`.launch.yaml`, `.launch.xml`, `.launch.py`)은 패키지 루트에 위치한 `launch/` 디렉터리 안에 저장한다.19
- **`config/` 디렉터리**: 노드 파라미터를 정의하는 YAML 파일들은 `config/` 디렉터리 안에 저장한다. 이는 런치 파일과 설정 파일을 명확히 분리하여 관리를 용이하게 한다.13
- **전용 `_bringup` 패키지**: 대규모 프로젝트에서는 모든 런치 파일과 설정을 중앙에서 관리하기 위해 `my_robot_bringup`과 같이 이름이 `_bringup`이나 `_support`로 끝나는 전용 패키지를 만드는 것이 일반적인 모범 사례다.21

### 3.2 런치 파일 설치: `colcon`이 파일을 찾는 방법

`ros2 launch` 명령어는 소스 코드(`src/`) 디렉터리에 있는 파일을 직접 실행하는 것이 아니라, 빌드 후 `install/` 디렉터리에 설치된 파일을 찾아 실행한다.20 따라서 런치 파일을 생성하거나 수정한 후에는 반드시 빌드 과정을 통해 설치해야 한다. 이 과정은 패키지 타입(`ament_python` 또는 `ament_cmake`)에 따라 다르다.

- **`ament_python` 패키지**: `setup.py` 파일을 수정하여 `data_files` 목록에 `launch/` 디렉터리를 추가해야 한다. `glob`을 사용하면 `launch/` 디렉터리 내의 모든 파일을 자동으로 설치하도록 지정할 수 있다.17

  ```Python
  # setup.py
  import os
  from glob import glob
  from setuptools import setup
  
  package_name = 'my_python_pkg'
  
  setup(
      #... 기타 설정...
      data_files=[
          ('share/ament_index/resource_index/packages',
              ['resource/' + package_name]),
          ('share/' + package_name, ['package.xml']),
          # launch 디렉터리의 모든 yaml 파일을 설치
          (os.path.join('share', package_name, 'launch'), glob('launch/*.yaml')),
          # config 디렉터리의 모든 yaml 파일을 설치
          (os.path.join('share', package_name, 'config'), glob('config/*.yaml')),
      ],
  )
  ```

- **`ament_cmake` 패키지**: `CMakeLists.txt` 파일에 `install(DIRECTORY...)` 명령어를 추가하여 `launch/` 디렉터리를 설치하도록 명시한다.19

  ```CMake
  # CMakeLists.txt
  install(DIRECTORY launch
    DESTINATION share/${PROJECT_NAME}
  )
  ```

이 설치 과정은 개발자들이 자주 겪는 혼란의 원인이므로 반드시 이해해야 한다. 개발 중에는 매번 `colcon build`를 실행하는 번거로움을 피하기 위해 `colcon build --symlink-install` 옵션을 사용하는 것이 좋다. 이 옵션은 파일을 복사하는 대신 심볼릭 링크를 생성하므로, 소스 파일을 수정하면 즉시 `install/` 디렉터리에 반영된다.22

### 3.3 최소 YAML 런치 파일 작성

모든 YAML 런치 파일은 정해진 기본 구조를 따른다.

1. 파일 최상단에 YAML 버전(`%YAML 1.2`)과 문서 시작 구분자(`---`)를 명시한다.7
2. 최상위 요소로 `launch:` 키를 사용한다. 이 키의 값은 실행할 *액션(action)*들의 리스트(배열)다.19
3. 가장 기본적인 액션은 `node`이며, 특정 노드를 실행하는 역할을 한다.

아래는 `demo_nodes_cpp` 패키지의 `talker` 노드를 실행하는 가장 간단한 YAML 런치 파일의 예다.

```YAML
# my_first_launch.yaml
%YAML 1.2
---
launch:
  - node:
      pkg: "demo_nodes_cpp"
      exec: "talker"
      name: "my_talker"
```

### 3.4 실행 및 확인

작성한 런치 파일을 실행하는 과정은 다음과 같다.

1. **빌드**: 워크스페이스 루트에서 `colcon build` 명령어로 패키지를 빌드하여 런치 파일을 설치한다.
2. **환경 설정**: 새 터미널에서 `source install/setup.bash` 명령어로 빌드된 환경을 활성화한다.
3. **실행**: `ros2 launch <package_name> <launch_file_name.yaml>` 명령어로 런치 파일을 실행한다.1
4. **확인**: 다른 터미널에서 `ros2 node list`를 실행하여 `my_talker` 노드가 실행 중인지 확인한다.

## 4. YAML 런처 고급 활용: 파라미터, 리매핑, 인자

기본적인 노드 실행을 넘어, YAML 런처는 노드의 동작을 세밀하게 제어하고 시스템을 동적으로 구성할 수 있는 다양한 고급 기능을 제공한다.

### 4.1 노드 속성 제어: 네임스페이스와 이름

`node` 액션 내에서 `name`과 `namespace` 속성을 지정하여 노드를 체계적으로 관리하고 토픽 이름 충돌을 방지할 수 있다.7

- `name`: 노드에 고유한 이름을 부여한다. 지정하지 않으면 ROS 2가 임의의 이름을 생성한다.
- `namespace`: 노드를 특정 네임스페이스에 할당한다. 해당 노드가 사용하는 모든 상대적인 토픽/서비스 이름 앞에 네임스페이스가 접두사로 붙는다.

```YAML
- node:
    pkg: "turtlesim"
    exec: "turtlesim_node"
    name: "sim"
    namespace: "turtlesim1"
```

### 4.2 런치 인자 (`arg`): 동적인 설정 값 전달

`arg` 액션은 런치 파일을 위한 변수를 선언한다. 이 변수는 런치 파일 실행 시 커맨드 라인에서 값을 설정하거나, 지정된 기본값을 사용하도록 할 수 있다.24 선언된 인자는 런치 파일 내에서 

`$(var <name>)` 대체(substitution) 구문을 통해 참조할 수 있다.12

- **인자 선언**:

  YAML

  ```
  - arg:
      name: "background_r"
      default: "0"
      description: "배경색의 Red 채널 값"
  ```

- **인자 사용**:

  YAML

  ```
  - node:
      pkg: "turtlesim"
      exec: "turtlesim_node"
      param:
        - name: "background_r"
          value: "$(var background_r)"
  ```

- 커맨드 라인에서 값 재정의:

  ros2 launch my_package my_launch.yaml background_r:=255.7

### 4.3 노드 파라미터 (`param`): 노드 동작 미세 조정

노드 파라미터는 노드 내부의 변수 값을 외부에서 설정하여 코드 수정 없이 동작을 변경할 수 있게 해준다. YAML 런치 파일에서 파라미터를 설정하는 방법은 두 가지다.

#### 4.3.1 방법 1: 인라인 파라미터 설정

`node` 액션 내에 `param` 키를 사용하여 파라미터를 직접 정의할 수 있다. 문법은 `name`과 `value`를 키로 갖는 딕셔너리의 리스트 형태다.24

```YAML
- node:
    pkg: "turtlesim"
    exec: "turtlesim_node"
    param:
      - name: "background_g"
        value: 255
      - name: "background_b"
        value: 255
```

이 문법은 다소 장황하게 느껴질 수 있는데, 이는 YAML 런처의 문법이 XML 런처의 구조를 그대로 따르기 때문이다. XML에서는 `<param name="..." value="..."/>` 태그를 반복해서 사용하며, 이것이 YAML에서는 딕셔너리의 리스트로 변환된 것이다.15 이 "XML의 그림자"를 이해하면 YAML 런처의 다소 비직관적인 문법을 이해하는 데 도움이 된다.

#### 4.3.2 방법 2: 외부 YAML 파일에서 로드

수많은 파라미터를 관리해야 할 경우, 별도의 YAML 파일에 파라미터를 정의하고 이를 불러오는 것이 훨씬 효율적이다.13 ROS 1의 `<rosparam>` 태그는 ROS 2에서 `from` 속성을 가진 `param` 액션으로 대체되었다.27

- 런치 파일 (my_launch.yaml):

  param 액션에 from 키를 사용하여 파라미터 파일의 경로를 지정한다. 이 때 $(find-pkg-share <pkg_name>) 대체를 사용하면 패키지 상대 경로로 파일을 지정할 수 있어 이식성이 높아진다.14

  ```YAML
  - node:
      pkg: "my_package"
      exec: "my_node"
      name: "my_node_instance"
      param:
        - from: "$(find-pkg-share my_package)/config/my_params.yaml"
  ```

- 파라미터 파일 (my_params.yaml):

  파라미터 파일은 런치 파일과 문법이 다르다는 점에 반드시 유의해야 한다. 노드이름: ros__parameters: {... } 구조를 따라야 한다.13 여기서 '노드이름'은 런치 파일에서 지정한 최종 노드 이름(네임스페이스 포함)과 정확히 일치해야 한다.

  ```YAML
  # config/my_params.yaml
  my_node_instance:
    ros__parameters:
      update_frequency: 10.0
      message: "Hello from file"
  ```

  또한, `/**:` 와일드카드를 사용하면 파일에 정의된 파라미터를 런치 파일 내 모든 노드에 일괄 적용할 수도 있다.17

### 4.4 토픽 리매핑 (`remap`): 노드 간 연결 재구성

`remap` 액션은 노드 소스 코드를 변경하지 않고 노드가 사용하는 토픽, 서비스, 액션의 이름을 바꿀 수 있게 해준다.28

`from`에 원래 이름을, `to`에 변경할 이름을 지정하는 딕셔너리 리스트 형식이다.

```YAML
- node:
    pkg: "demo_nodes_cpp"
    exec: "talker"
    remap:
      - from: "chatter"
        to: "my_custom_chatter"
```

## 5. 대규모 프로젝트를 위한 구조화: `include`와 `group`

복잡한 로봇 시스템은 수십 개의 노드로 구성된다. 이러한 시스템을 단일 런치 파일로 관리하는 것은 비효율적이고 유지보수를 어렵게 만든다. ROS 2 런치 시스템은 `include`와 `group` 액션을 통해 시스템을 모듈화하고 계층적으로 구성할 수 있는 강력한 기능을 제공한다.

### 5.1 재사용성을 위한 `include`

`include` 액션은 하나의 런치 파일이 다른 런치 파일을 포함할 수 있게 해준다. 이는 시스템을 기능 단위(예: 카메라 모듈, 모터 드라이버 모듈, 내비게이션 모듈)로 분할하고, 각 모듈을 별도의 런치 파일로 만들어 재사용성을 극대화하는 핵심 기능이다.17 특히 `include`는 포함하는 파일과 포함되는 파일의 형식이 달라도 상관없다. 예를 들어, Python 런치 파일이 YAML 런치 파일을 포함하거나 그 반대도 가능하다.7

```YAML
# main_launch.yaml
launch:
  - include:
      file: "$(find-pkg-share camera_module)/launch/camera.launch.yaml"

  - include:
      file: "$(find-pkg-share navigation_module)/launch/nav.launch.py"
```

### 5.2 `include`에 인자 전달

포함되는 런치 파일을 더욱 유연하게 사용하기 위해 `arg`를 전달할 수 있다. 이를 통해 동일한 런치 파일을 다른 설정으로 여러 번 재사용할 수 있다.25 예를 들어, 로봇의 왼쪽 카메라와 오른쪽 카메라에 대해 동일한 `camera.launch.yaml` 파일을 사용하면서 카메라 ID만 다르게 설정할 수 있다.

```YAML
# main_launch.yaml
launch:
  - include:
      file: "$(find-pkg-share my_robot_bringup)/launch/camera_module.launch.yaml"
      arg:
        - name: "camera_id"
          value: "left_camera"
        - name: "frame_rate"
          value: "15.0"

  - include:
      file: "$(find-pkg-share my_robot_bringup)/launch/camera_module.launch.yaml"
      arg:
        - name: "camera_id"
          value: "right_camera"
        - name: "frame_rate"
          value: "10.0"
```

### 5.3 `group`과 `push_ros_namespace`로 범위 지정

동일한 모듈을 여러 개 실행할 때, 각 모듈의 노드들이 서로 충돌하지 않도록 격리해야 한다. `group` 액션과 `push_ros_namespace`를 함께 사용하면 이 문제를 해결할 수 있다. `push_ros_namespace`는 `group` 내의 모든 액션에 대해 지정된 네임스페이스를 적용한다.7 이를 통해 동일한 로봇 모듈 런치 파일을 여러 번 `include`하더라도 각 인스턴스가 고유한 네임스페이스 내에서 동작하게 된다.

아래 예제는 두 대의 로봇에 대해 동일한 `robot.launch.yaml`을 사용하면서 각각 `robot1`과 `robot2` 네임스페이스로 격리하는 방법을 보여준다.

```YAML
# multi_robot_launch.yaml
launch:
  # 로봇 1 실행
  - group:
      - push_ros_namespace:
          namespace: "robot1"
      - include:
          file: "$(find-pkg-share my_robot_module)/launch/robot.launch.yaml"

  # 로봇 2 실행
  - group:
      - push_ros_namespace:
          namespace: "robot2"
      - include:
          file: "$(find-pkg-share my_robot_module)/launch/robot.launch.yaml"
```

이처럼 `include`, `arg`, `group`을 조합하는 것은 단순한 기능의 나열이 아니다. 이는 복잡한 시스템을 관리하기 위한 **"컴포지션 기반 런치 패턴(Compositional Launch Pattern)"**이라는 통합된 설계 방식이다. 이 패턴을 통해 대규모 시스템을 작고, 자율적이며, 재설정 가능한 컴포넌트로 분해하여 관리할 수 있다. 이는 실제 로보틱스 애플리케이션의 복잡성을 다루는 핵심 전략이다.

## 6. 종합 실습 예제: Talker/Listener 실행하기

지금까지 배운 개념들을 종합하여 실제 동작하는 예제를 만들어 본다. 이 예제는 `demo_nodes_cpp` 패키지의 `talker`와 `listener` 노드를 실행하되, 네임스페이스, 외부 파라미터 파일 로딩, 런치 인자, 리매핑을 모두 사용하는 완전한 YAML 런치 파일을 작성하는 것을 목표로 한다.7

### 6.1 프로젝트 설정

1. **패키지 생성**: `ament_python` 타입의 새 패키지를 생성한다.

   ```Bash
   cd ~/ros2_ws/src
   ros2 pkg create yaml_launch_example --build-type ament_python --dependencies demo_nodes_cpp
   ```

2. **디렉터리 생성**: 생성된 패키지 내에 `launch`와 `config` 디렉터리를 만든다.

   ```Bash
   cd yaml_launch_example
   mkdir launch config
   ```

3. **`setup.py` 설정**: `colcon`이 런치 파일과 설정 파일을 찾아서 설치할 수 있도록 `setup.py`를 수정한다.

   ```Python
   # yaml_launch_example/setup.py
   import os
   from glob import glob
   from setuptools import setup
   
   package_name = 'yaml_launch_example'
   
   setup(
       name=package_name,
       version='0.0.0',
       packages=[package_name],
       data_files=[
           ('share/ament_index/resource_index/packages',
               ['resource/' + package_name]),
           ('share/' + package_name, ['package.xml']),
           (os.path.join('share', package_name, 'launch'), glob('launch/*.yaml')),
           (os.path.join('share', package_name, 'config'), glob('config/*.yaml')),
       ],
       install_requires=['setuptools'],
       zip_safe=True,
       maintainer='user',
       maintainer_email='user@todo.todo',
       description='TODO: Package description',
       license='TODO: License declaration',
       tests_require=['pytest'],
       entry_points={
           'console_scripts': [
           ],
       },
   )
   ```

### 6.2 파라미터 파일 작성 (`config/params.yaml`)

`talker` 노드를 위한 커스텀 파라미터를 정의하는 파일을 `config` 디렉터리에 생성한다.

```YAML
# config/params.yaml
my_talker:
  ros__parameters:
    # talker 노드가 메시지를 발행하는 주기 (단위: 초)
    # 기본값은 1.0초. 이 파일을 통해 0.2초 (5Hz)로 변경한다.
    period_sec: 0.2
```

### 6.3 YAML 런치 파일 작성 (`launch/talker_listener.launch.yaml`)

이제 `launch` 디렉터리에 핵심 런치 파일을 작성한다. 이 파일은 다음 기능을 모두 포함한다.

- 커맨드 라인에서 설정 가능한 `topic_name` 인자 선언
- `talker` 노드를 `talker_ns` 네임스페이스에서 실행
- `config/params.yaml` 파일에서 `talker` 노드의 파라미터 로드
- `listener` 노드를 `listener_ns` 네임스페이스에서 실행
- `remap`을 사용하여 두 노드가 `topic_name` 인자로 지정된 토픽을 사용하도록 설정

```YAML
# launch/talker_listener.launch.yaml
%YAML 1.2
---
launch:
  # 커맨드 라인에서 토픽 이름을 변경할 수 있도록 인자(argument)를 선언한다.
  - arg:
      name: "topic_name"
      default: "chatter"
      description: "Talker와 Listener가 통신할 토픽 이름"

  # Talker 노드 실행
  - node:
      pkg: "demo_nodes_cpp"
      exec: "talker"
      name: "my_talker"
      namespace: "talker_ns"
      # 외부 YAML 파일에서 파라미터를 로드한다.
      param:
        - from: "$(find-pkg-share yaml_launch_example)/config/params.yaml"
      # 기본 'chatter' 토픽을 'topic_name' 인자의 값으로 리매핑한다.
      remap:
        - from: "chatter"
          to: "$(var topic_name)"

  # Listener 노드 실행
  - node:
      pkg: "demo_nodes_cpp"
      exec: "listener"
      name: "my_listener"
      namespace: "listener_ns"
      # Listener도 동일한 토픽을 구독하도록 리매핑한다.
      remap:
        - from: "chatter"
          to: "$(var topic_name)"
```

### 6.4 빌드 및 실행

1. **빌드**: 워크스페이스 루트로 이동하여 패키지를 빌드한다.

   ```Bash
   cd ~/ros2_ws
   colcon build --packages-select yaml_launch_example
   ```

2. **환경 설정**: 환경을 활성화한다.

   ```Bash
   source install/setup.bash
   ```

3. **실행 (기본값)**: 인자 없이 실행하면 `default` 값인 `chatter` 토픽이 사용된다.

   ```Bash
   ros2 launch yaml_launch_example talker_listener.launch.yaml
   ```

4. **실행 (인자 재정의)**: 커맨드 라인에서 `topic_name`을 변경하여 실행한다.

   ```Bash
   ros2 launch yaml_launch_example talker_listener.launch.yaml topic_name:=custom_topic
   ```

### 6.5 결과 확인

새 터미널을 열고 환경을 활성화한 후, 다음 명령어를 통해 시스템이 의도대로 동작하는지 확인한다.

- **노드 목록 확인**:

  ```Bash
  ros2 node list
  ```

  출력에 `/talker_ns/my_talker`와 `/listener_ns/my_listener`가 보여야 한다.

- **토픽 목록 확인** (`topic_name:=custom_topic`으로 실행했을 경우):

  ```Bash
  ros2 topic list
  ```

  출력에 `/custom_topic`이 보여야 한다. (네임스페이스가 토픽에 적용되지 않은 것은 `talker` 노드가 절대 경로 `/chatter`를 사용하도록 코딩되었기 때문이며, `remap`이 올바르게 동작한 결과다.)

- **파라미터 확인**:

  ```Bash
  ros2 param get /talker_ns/my_talker period_sec
  ```

  출력으로 `Double value is: 0.2`가 표시되어 파라미터 파일이 성공적으로 로드되었음을 확인할 수 있다.

## 7. 문제 해결 가이드: 흔한 오류와 해결책

YAML 런처는 간결하지만, 사소한 실수로 인해 예상치 못한 오류가 발생하기 쉽다. 다음은 자주 발생하는 문제와 그 해결책을 정리한 것이다.

### 7.1 구문 오류: YAML 들여쓰기 및 구조

- **문제**: YAML은 들여쓰기에 극도로 민감하다. 잘못된 들여쓰기는 가장 흔한 구문 오류의 원인이다.30 또한, 주석 안에 콜론(`:`)과 같은 특수 문자가 포함되면 파서가 혼동을 일으킬 수 있다.32
- **오류 메시지**: `bad indentation of a mapping entry`, `Parser Error`, `Couldn't parse parameter override rule` 등.
- **해결책**:
  - YAML은 들여쓰기에 탭(tab) 대신 스페이스(space)를 사용한다. 같은 계층의 모든 항목은 정확히 동일한 수의 스페이스로 들여써야 한다.
  - IDE의 YAML 플러그인이나 린터(linter)를 사용하여 문법 오류를 사전에 감지하는 것이 좋다.
  - 주석을 사용할 때 콜론(`:`)과 같은 문자가 문제를 일으킬 수 있으므로 주의한다.32

### 7.2 빌드 및 설치 오류

- **문제**: `ros2 launch` 명령어가 런치 파일을 찾지 못하거나, 파일을 수정해도 변경 사항이 적용되지 않는다.
- **오류 메시지**: `file '<file_name>' was not found in the share directory of package '<package_name>'`.34
- **해결책**: 이 문제는 `ros2 launch`가 `src/`가 아닌 `install/` 디렉터리에서 파일을 찾는다는 사실을 이해하면 해결된다. 아래 체크리스트를 확인한다.
  1. 파일을 생성하거나 수정한 후 `colcon build`를 실행했는가? 22
  2. `setup.py` (ament_python) 또는 `CMakeLists.txt` (ament_cmake)에 정확한 설치 규칙이 포함되어 있는가? 20
  3. **현재 터미널에서** `source install/setup.bash` (또는 `local_setup.bash`)를 실행했는가? 35
  4. 개발 중에는 `colcon build --symlink-install`을 사용하여 매번 빌드하는 번거로움을 피하고 있는가? 22

### 7.3 런타임 오류: 파라미터 및 대체

- **문제 1**: 파라미터 YAML 파일이 로드되지 않는데 아무런 오류도 표시되지 않는다.13
  - **원인**: 파라미터 파일에 명시된 노드 이름(예: `my_node:`)이 런치 파일에서 최종적으로 생성된 노드의 전체 이름(네임스페이스 포함)과 정확히 일치하지 않기 때문이다. ROS는 이 경우 조용히 실패한다.
  - **해결책**: `ros2 node list`와 `ros2 node info <node_name>` 명령어로 실제 실행된 노드의 전체 이름을 확인하고, 파라미터 파일의 키와 일치시킨다.
- **문제 2**: 대체(substitution)가 실패한다.
  - **오류 메시지**: `SubstitutionFailure`.38
  - **원인**: `$(var...)` 구문에서 변수 이름에 오타가 있거나, `$(command...)` 구문에서 실행된 명령 자체가 실패하는 경우에 발생한다.
  - **해결책**: 모든 `$(var...)` 변수 이름의 철자를 다시 확인한다. `$(command...)`를 사용하는 경우, 해당 명령어를 터미널에서 직접 실행하여 런치 시스템과 무관하게 독립적으로 디버깅한다.

### 7.4 "XML의 그림자"로 인한 혼란

- **문제**: `arg`의 `choices`와 같이 일부 기능의 YAML 문법이 매우 비직관적이고 이상하게 느껴진다.15

  - **원인**: 앞서 설명했듯이, YAML 런처의 파서는 XML 구조를 그대로 모방하여 설계되었기 때문이다.

  - **해결책**: 이해하기 어려운 YAML 문법에 직면했을 때, 해당 기능의 XML 런치 파일 예제를 찾아보는 것이 좋은 해결책이 될 수 있다. YAML 문법은 XML 태그 구조를 거의 일대일로 변환한 것일 가능성이 높다. 예를 들어, `arg`에 선택지를 제공하는 XML 문법은 다음과 같다.

    ```XML
    <arg name="my_arg">
      <choice value="option1"/>
      <choice value="option2"/>
    </arg>
    ```

    이는 아래와 같은 비직관적인 YAML 문법으로 변환된다.15

    ```YAML
    - arg:
        name: "my_arg"
        choice:
          - value: "option1"
          - value: "option2"
    ```

    이러한 연관성을 이해하면 문서화가 부족한 기능의 문법도 유추해낼 수 있다.

#### **참고 자료**

1. Launching nodes - ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Launching-Multiple-Nodes/Launching-Multiple-Nodes.html
2. Launch - ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/Concepts/Basic/About-Launch.html
3. ROS 2 Launch System - ROS2 Design, accessed July 26, 2025, https://design.ros2.org/articles/roslaunch.html
4. Creating a launch file - ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/Launch/Creating-Launch-Files.html
5. Why does the ROS2 Python launch files look like XML written in Python? - ROS Discourse, accessed July 26, 2025, https://discourse.ros.org/t/why-does-the-ros2-python-launch-files-look-like-xml-written-in-python/23679
6. Using XML, YAML, and Python for ROS 2 Launch Files, accessed July 26, 2025, https://docs.ros.org/en/rolling/How-To-Guides/Launch-file-different-formats.html
7. Using XML, YAML, and Python for ROS 2 Launch Files, accessed July 26, 2025, https://docs.ros.org/en/humble/How-To-Guides/Launch-file-different-formats.html
8. Using Python, XML, and YAML for ROS 2 Launch Files, accessed July 26, 2025, https://docs.ros.org/en/foxy/How-To-Guides/Launch-file-different-formats.html
9. ROS 2 launch: Python vs XML - Robotics Stack Exchange, accessed July 26, 2025, https://robotics.stackexchange.com/questions/96149/ros-2-launch-python-vs-xml
10. Which ROS 2 launch file format do you prefer Python or XML and why? - Reddit, accessed July 26, 2025, https://www.reddit.com/r/ROS/comments/1j6cu02/which_ros_2_launch_file_format_do_you_prefer/
11. ROS2 - Create a Launch File with XML - YouTube, accessed July 26, 2025, https://www.youtube.com/watch?v=Le1vx1_KUDQ
12. Migrating Launch Files - ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://ros2docs.robook.org/humble/How-To-Guides/Migrating-from-ROS1/Migrating-Launch-Files.html
13. ROS2 YAML For Parameters - The Robotics Back-End, accessed July 26, 2025, https://roboticsbackend.com/ros2-yaml-params/
14. ROS2: Adding parameters to YAML launch file - Robotics Stack Exchange, accessed July 26, 2025, https://robotics.stackexchange.com/questions/105910/ros2-adding-parameters-to-yaml-launch-file
15. Difficult to specify YAML argument choices / Issue #698 / ros2/launch - GitHub, accessed July 26, 2025, https://github.com/ros2/launch/issues/698
16. ROS2 - Include a Launch File in Another Launch File (Python, XML, YAML) - YouTube, accessed July 26, 2025, https://www.youtube.com/watch?v=sl0exwcg3o8&pp=0gcJCfwAo7VqN5tD
17. Managing large projects - ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/Launch/Using-ROS2-Launch-For-Large-Projects.html
18. Using ROS 2 launch for large projects - daobook 0.0.1 文档, accessed July 26, 2025, https://daobook.github.io/ros2-docs/galactic/Tutorials/Launch-Files/Using-ROS2-Launch-For-Large-Projects.html
19. Integrating launch files into ROS 2 packages, accessed July 26, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/Launch/Launch-system.html
20. Integrating launch files into ROS 2 packages - ROS documentation, accessed July 26, 2025, https://docs.ros.org/en/foxy/Tutorials/Intermediate/Launch/Launch-system.html
21. ros2-examples/launch_files.md at humble - GitHub, accessed July 26, 2025, https://github.com/IntelligentSystemsLabUTV/ros2-examples/blob/humble/launch_files.md
22. Launch Files - ROS Industrial Training - Read the Docs, accessed July 26, 2025, https://industrial-training-master.readthedocs.io/en/foxy/_source/session2/ros2/2-Launch-Files.html
23. Set parameters in ROS2 using a yaml file without needing to build the package every time, accessed July 26, 2025, https://robotics.stackexchange.com/questions/94711/set-parameters-in-ros2-using-a-yaml-file-without-needing-to-build-the-package-ev
24. Using XML, YAML, and Python for ROS 2 Launch Files, accessed July 26, 2025, https://ros2docs.robook.org/rolling/How-To-Guides/Launch-file-different-formats.html
25. Using substitutions - ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/Launch/Using-Substitutions.html
26. Create a Launch File with YAML - ROS2 - YouTube, accessed July 26, 2025, https://www.youtube.com/watch?v=2hFEfjB9kh0
27. Migrating Launch Files - ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/How-To-Guides/Migrating-from-ROS1/Migrating-Launch-Files.html
28. How to remap topic in a ROS2 launch file? - Robotics Stack Exchange, accessed July 26, 2025, https://robotics.stackexchange.com/questions/98873/how-to-remap-topic-in-a-ros2-launch-file
29. ROS2 - Remap Nodes, Topics, Services (with Command Line and Launch File) - YouTube, accessed July 26, 2025, https://www.youtube.com/watch?v=bjH5qmKKpRk
30. Indentation error in configuration.yaml - Home Assistant Community, accessed July 26, 2025, https://community.home-assistant.io/t/indentation-error-in-configuration-yaml/552278
31. ros2 - Error: Error opening YAML file, at ./src/parser.c:270, at ./src/rcl/arguments.c:406 - Stack Overflow, accessed July 26, 2025, https://stackoverflow.com/questions/79567627/error-error-opening-yaml-file-at-src-parser-c270-at-src-rcl-arguments-c
32. Parser Error Couldn't parse parameter override rule - loading tricycle_controller from ros2_control / Issue #295 / ros-controls/gazebo_ros2_control - GitHub, accessed July 26, 2025, https://github.com/ros-controls/gazebo_ros2_control/issues/295
33. rclcpp ROS2 params yaml file comments - Robotics Stack Exchange, accessed July 26, 2025, https://robotics.stackexchange.com/questions/109663/rclcpp-ros2-params-yaml-file-comments
34. Unable to launch my launch file - ROS2 Basics in 5 Days Humble (Python), accessed July 26, 2025, https://get-help.theconstruct.ai/t/unable-to-launch-my-launch-file/25548
35. ROS 2 common issues and mistakes - Karelics, accessed July 26, 2025, https://karelics.fi/blog/2023/05/19/ros-2-common-issues-and-mistakes/
36. [Bug] ROS2 can not find launch file - The Construct ROS Community, accessed July 26, 2025, https://get-help.theconstruct.ai/t/bug-ros2-can-not-find-launch-file/28327
37. ros2 launch not working after source my package - ROS Answers archive, accessed July 26, 2025, https://answers.ros.org/question/350137/
38. Running ros2_control on a real robot (Making a Mobile Robot Pt 13) - #39 by Afiq003, accessed July 26, 2025, https://discourse.articulatedrobotics.xyz/t/discussion-running-ros2-control-on-a-real-robot-making-a-mobile-robot-pt-13/107/39

