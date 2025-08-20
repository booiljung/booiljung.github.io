# ROS 2 Humble package.xml 완벽 안내서

## 1. `package.xml`의 진정한 역할

ROS 2 패키지를 다루다 보면 `package.xml` 파일을 마주하게 된다. 많은 개발자들이 이를 단순히 패키지 이름, 버전, 작성자 정보 등을 담는 메타데이터 파일로 취급하곤 한다. 하지만 이는 `package.xml`의 역할을 과소평가하는 것이다. 이 파일은 단순한 정보의 나열이 아니라, ROS 2 생태계 전체와 소통하는 패키지의 **공식 명세서(public API)**이자 **계약서**다.

`package.xml`은 패키지의 정체성을 정의하고, 시스템의 다른 부분들과 어떻게 상호작용할지를 명시하는 핵심적인 역할을 수행한다. 예를 들어, 빌드 도구인 `colcon`은 이 파일을 읽어 패키지 간의 빌드 순서를 결정한다. 의존성 관리 도구인 `rosdep`은 이 파일을 참조하여 필요한 시스템 라이브러리를 설치한다. `ros2 launch`나 `ros2 run` 같은 커맨드 라인 도구들 역시 이 파일을 통해 패키지의 위치와 실행에 필요한 정보를 파악한다.1

이처럼 `package.xml`은 ROS 2의 다양한 도구들이 각자의 역할을 수행할 수 있도록 표준화된 인터페이스를 제공하는 중심축이다. `colcon`이 `CMakeLists.txt`나 `setup.py`의 복잡한 문법을 직접 해석할 필요 없이 빌드 순서를 알 수 있는 것도, `rosdep`이 C++ 코드의 내용을 몰라도 필요한 라이브러리를 설치할 수 있는 것도 모두 `package.xml`이라는 잘 정의된 계약서 덕분이다. 이러한 설계는 ROS 2 생태계의 각 도구들이 서로 느슨하게 결합(decoupled)되어 독립적으로 발전할 수 있게 만드는 핵심 원리다.4

이 안내서는 `package.xml`의 모든 것을 다룬다. 기본적인 파일 구조와 필수 태그부터 시작하여, 복잡한 의존성 관계를 정의하는 방법, 빌드 시스템과의 연동, 고급 기능 활용법, 그리고 실제 프로젝트에서 적용할 수 있는 모범 사례와 흔히 발생하는 문제들의 해결책까지 깊이 있게 파고들 것이다. 이 문서를 끝까지 읽고 나면, `package.xml`을 단순한 설정 파일이 아닌, ROS 2 패키지의 잠재력을 최대한으로 이끌어내는 강력한 도구로 활용할 수 있게 될 것이다.

## 2.  `package.xml` 포맷 3: 해부학

ROS 2 Humble은 `package.xml` 포맷 3을 표준으로 사용한다. 이 포맷은 REP-149에 명시되어 있으며, 이전 포맷에 비해 조건부 의존성 선언과 같은 강력한 기능들이 추가되었다.7 패키지를 생성하면 다음과 같은 기본 구조를 가진 `package.xml` 파일이 만들어진다.1

```XML
<?xml version="1.0"?>
<?xml-model href="http://download.ros.org/schema/package_format3.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
<package format="3">
  <name>my_package</name>
  <version>0.0.0</version>
  <description>TODO: Package description</description>
  <maintainer email="user@todo.todo">user</maintainer>
  <license>TODO: License declaration</license>

  <buildtool_depend>ament_cmake</buildtool_depend>

  <test_depend>ament_lint_auto</test_depend>
  <test_depend>ament_lint_common</test_depend>

  <export>
    <build_type>ament_cmake</build_type>
  </export>
</package>
```

### 2.1  XML 프롤로그와 스키마 선언

파일의 가장 첫 두 줄은 XML 문서의 표준 선언과 스키마 정의를 담고 있다.

- `<?xml version="1.0"?>`: 이 파일이 표준 XML 1.0 형식임을 선언한다.
- `<?xml-model href="http://download.ros.org/schema/package_format3.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>`: 이 줄은 `package.xml` 파일의 구조와 유효성을 검증하는 데 사용될 XSD(XML Schema Definition) 파일의 위치를 지정한다.8 즉, 이 파일은 `package.xml`이 따라야 할 "설계도"와 같다. `ament_xmllint`와 같은 린팅 도구는 이 스키마를 기준으로 파일의 문법적 정확성을 검사하므로, 이 선언은 매우 중요하다.11

### 2.2  루트 태그: `<package format="3">`

모든 내용은 `<package>` 루트 태그 안에 위치한다. 여기서 `format="3"` 속성은 이 파일이 REP-149에 명시된 포맷 3 사양을 따른다는 것을 명확히 한다.7 이 속성이 있어야만 `<group_depend>`나 `condition` 속성 같은 포맷 3의 새로운 기능들을 올바르게 사용할 수 있다. ROS 2는 이전 버전인 포맷 2도 지원하지만, 특별한 이유가 없다면 Humble에서는 포맷 3을 사용하는 것이 표준이다.13

### 2.3  필수 태그: 패키지의 신분증

모든 ROS 2 패키지는 자신의 정체성을 나타내는 최소한의 정보를 반드시 포함해야 한다. 이 정보들은 패키지의 "신분증"과 같은 역할을 한다.7

- **`<name>`**: 패키지의 고유한 이름이다. 이 이름은 ROS 생태계 전체에서 유일해야 하며, 다른 패키지들이 이 패키지를 참조하거나 `ros2 run <package_name> <executable_name>`과 같이 커맨드 라인에서 사용할 때 식별자로 사용된다. 패키지 이름은 소문자, 숫자, 그리고 밑줄(`_`)만으로 구성하는 것이 규칙이다.1

- **`<version>`**: 패키지의 현재 버전을 나타낸다. ROS 커뮤니티는 유의적 버전(Semantic Versioning)을 따를 것을 강력히 권장한다. 버전은 `MAJOR.MINOR.PATCH` (예: `1.2.3`) 형식으로 구성되며, 각 숫자는 API 호환성과 변경 사항의 종류를 나타낸다.14

  `bloom`과 같은 배포 도구는 이 버전 정보를 사용하여 릴리스를 관리하고 Git 태그와 연동하므로 정확하게 관리하는 것이 매우 중요하다.7

- **`<description>`**: 패키지의 기능과 목적을 설명하는 텍스트다. 다른 개발자들이 이 패키지가 무엇을 하는지 빠르고 명확하게 이해할 수 있도록 작성해야 한다. 여러 줄에 걸쳐 상세한 설명도 가능하다.7

- **`<maintainer>`**: 패키지를 유지보수하는 책임자를 명시한다. `email` 속성은 필수로 포함해야 하며, 패키지에 문제가 발생했을 때 사용자들이 연락할 수 있는 공식적인 창구 역할을 한다. 프로젝트의 규모가 크거나 여러 사람이 관리하는 경우, 여러 개의 `<maintainer>` 태그를 사용하여 다수의 책임자를 지정할 수 있다.7

- **`<license>`**: 패키지에 적용되는 소프트웨어 라이선스를 명시한다. 오픈소스 프로젝트의 경우, 라이선스를 명확히 하는 것은 법적으로 매우 중요하다. Apache-2.0, BSD, MIT 등 SPDX 표준 라이선스 식별자를 사용하는 것이 일반적이다. `ros2 pkg create` 명령어에 `--license Apache-2.0`와 같은 옵션을 주면 자동으로 해당 라이선스 태그와 `LICENSE` 파일이 생성된다.1

### 2.4  선택적 메타데이터 태그

필수 태그 외에도 패키지에 대한 추가 정보를 제공하여 사용자의 이해를 돕고 커뮤니티 참여를 유도하는 여러 선택적 태그가 있다.

- **`<author>`**: 패키지를 처음 개발한 사람이나 기여자를 명시한다. 유지보수 담당자(`maintainer`)와 작성자는 다를 수 있다. 여러 명의 작성자가 있다면 여러 개의 `<author>` 태그를 사용할 수 있다.17
- **`<url>`**: 패키지와 관련된 웹사이트 주소를 제공한다. `type` 속성을 사용하여 URL의 성격을 명확히 할 수 있다.
  - `website`: 패키지 공식 웹사이트나 위키 페이지 (예: `http://wiki.ros.org/my_package`)
  - `repository`: 소스 코드가 저장된 Git 저장소 주소 (예: `https://github.com/my_org/my_package`)
  - bugtracker: 버그나 이슈를 보고할 수 있는 이슈 트래커 주소 (예: https://github.com/my_org/my_package/issues) 이러한 정보는 사용자가 패키지에 대한 더 깊은 정보를 얻거나 프로젝트에 기여하고자 할 때 매우 유용하다.7

## 3.  핵심 기능: 의존성(Dependency) 선언

`package.xml`의 가장 중요하고 핵심적인 기능은 바로 의존성 관리다. 패키지가 올바르게 빌드되고 실행되기 위해 필요한 다른 패키지나 시스템 라이브러리가 무엇인지 명시하는 역할을 한다.

### 3.1  왜 두 번 선언해야 하는가? (`package.xml` vs. `CMakeLists.txt`/`setup.py`)

ROS 개발을 처음 접하는 사람들이 가장 흔하게 던지는 질문 중 하나는 "왜 `package.xml`과 `CMakeLists.txt`에 의존성을 두 번씩이나 선언해야 하는가?"이다. 이는 중복 작업처럼 보일 수 있지만, 실제로는 두 파일이 서로 다른 대상과 목적을 위해 의존성을 선언하기 때문에 발생하는 의도된 설계다.4

- package.xml의 역할: 생태계 도구를 위한 선언

  package.xml에 명시된 의존성은 ROS 2 생태계의 고수준 도구들을 위한 것이다.

  - **`rosdep`**: `rosdep install --from-paths src --ignore-src -y` 명령을 실행하면, `rosdep`은 `src` 폴더 내 모든 패키지의 `package.xml` 파일을 스캔한다. 그리고 파일에 선언된 의존성 '키(key)'들을 `rosdistro` 데이터베이스와 비교하여, 현재 운영체제에 맞는 시스템 패키지(예: `libboost-dev`, `python3-numpy`)를 `apt`나 `dnf` 같은 시스템 패고지 관리자를 통해 자동으로 설치해준다.6
  - **`colcon`**: `colcon build` 명령을 실행하면, `colcon` 역시 `package.xml`을 읽어 워크스페이스 내 패키지들 간의 의존성 그래프를 그린다. 이 그래프를 바탕으로 어떤 패키지를 먼저 빌드해야 하는지 위상 정렬(topological sort)을 수행하여 올바른 순서로 빌드를 진행한다. 예를 들어, 패키지 A가 패키지 B에 의존한다면, `colcon`은 반드시 B를 먼저 빌드한 후에 A를 빌드한다.4

  이처럼 `package.xml`은 패키지 외부의 도구들에게 "이 패키지를 빌드하고 실행하려면 **무엇이** 필요한가?"에 대한 정보를 제공하는 **선언적 명세**다.

- CMakeLists.txt / setup.py의 역할: 빌드 시스템을 위한 선언

  반면, CMakeLists.txt의 find_package()나 setup.py의 install_requires는 실제 컴파일러와 링커, 즉 빌드 시스템 자체를 위한 것이다.

  - **`find_package(rclcpp REQUIRED)` (in `CMakeLists.txt`)**: 이 구문은 CMake에게 `rclcpp` 패키지를 찾아달라고 요청한다. CMake는 이 요청을 받아 해당 패키지가 제공하는 헤더 파일의 경로, 라이브러리 파일의 경로 등의 정보를 찾아내어 컴파일러와 링커에게 전달한다.
  - **`install_requires=['rclpy']` (in `setup.py`)**: 이 구문은 Python의 `setuptools`에게 이 패키지가 `rclpy` 라이브러리를 필요로 하니, 설치 시 함께 고려해달라고 알려준다.

  이들은 생태계 도구들이 이미 준비해준 의존성을 "**어떻게** 찾아서 코드와 연결할 것인가?"에 대한 구체적인 **절차적 지시**다.

이러한 이중 선언 구조는 ROS 2의 유연성과 확장성의 핵심이다. `colcon`과 `rosdep`은 `package.xml`이라는 표준화된 명세만 이해하면 되므로, 내부적으로 CMake를 쓰든, Python setuptools를 쓰든, 혹은 미래에 나올 새로운 빌드 시스템을 쓰든 상관없이 일관되게 동작할 수 있다. 이 설계 덕분에 ROS 2는 ROS 1의 `catkin`이 CMake에 강하게 종속되었던 한계를 극복하고 다양한 빌드 시스템을 포용하는 현대적인 빌드 환경을 구축할 수 있었다.20

### 3.2  의존성 태그 상세 분석

`package.xml`에는 여러 종류의 의존성 태그가 있으며, 각 태그는 의존성이 필요한 시점과 범위에 따라 명확히 구분된다. 올바른 태그를 사용하는 것은 패키지의 의도를 명확히 하고, 불필요한 의존성 전파를 막으며, 효율적인 패키징을 위해 매우 중요하다.6

- **`<buildtool_depend>`**: 패키지를 빌드하는 데 사용되는 **빌드 도구** 자체에 대한 의존성을 선언한다. C++ 패키지의 경우, `ament`가 제공하는 CMake 매크로들을 사용하기 위해 거의 항상 `<buildtool_depend>ament_cmake</buildtool_depend>`를 포함한다.22 순수 Python 패키지의 경우, 빌드 타입이 `<export>` 태그 안에 `ament_python`으로 명시되므로 이 태그는 보통 사용되지 않는다.

- **`<build_depend>`**: **빌드 시에만** 필요한 의존성을 선언한다. 이 의존성은 패키지의 소스 코드(`.cpp`)나 내부적으로만 사용되는 비공개 헤더 파일(`.hpp`)에서만 필요하며, 패키지를 사용하는 다른 패키지에게는 노출되지 않는다. 예를 들어, 어떤 알고리즘 구현을 위해 `Boost` 라이브러리를 내부적으로만 사용하고 그 내용이 공개 API(헤더 파일)에 드러나지 않는다면 `<build_depend>`가 적합하다. 이렇게 선언된 의존성은 최종적으로 생성된 바이너리 패키지를 설치할 때 함께 설치될 필요가 없다.6

- **`<build_export_depend>`**: **전이 의존성(Transitive Dependency)**을 관리하기 위한 매우 중요한 태그다. 만약 내 패키지(P1)의 **공개 헤더 파일**(`include/p1/my_header.hpp`)이 다른 패키지(P2)의 헤더 파일(예: `geometry_msgs/msg/pose.hpp`)을 `#include`하고 있다면, 내 패키지(P1)를 사용하는 제3의 패키지(P3)는 P1을 빌드하기 위해 P2의 헤더 파일도 필요로 하게 된다. 이 관계를 명시하지 않으면 P3는 컴파일 오류를 겪게 된다. `<build_export_depend>`는 바로 이럴 때 "P1을 빌드하기 위해 사용하는 패키지는 P2도 필요하다"는 정보를 P3에게 전달(export)하는 역할을 한다. 주로 헤더 파일이나 CMake 설정 파일을 통해 다른 패키지의 기능이 전파될 때 사용된다.6

- **`<exec_depend>`**: **실행 시에만** 필요한 의존성을 선언한다. 빌드 과정과는 무관하지만, 노드를 실행할 때 필요한 라이브러리나 다른 패키지의 실행 파일 등이 여기에 해당한다. 대표적인 예는 다음과 같다:

  - 순수 Python 패키지가 `import rclpy`나 `import numpy`를 할 때, `rclpy`와 `numpy`는 실행 시점에 필요하다.

  - 런치 파일(`*.launch.py`)에서 다른 패키지의 노드(예: `demo_nodes_cpp`의 `talker`)를 실행할 때, `demo_nodes_cpp`는 실행 시점 의존성이다.24

  - 실행 시 동적으로 로드되는 공유 라이브러리(.so 파일)가 필요할 때.

    순수 Python 패키지는 컴파일 단계가 없으므로 대부분의 의존성을 <exec_depend>로 선언한다.6

- **`<test_depend>`**: **테스트 코드를 빌드하고 실행할 때만** 필요한 의존성을 선언한다. `gtest`나 `pytest` 같은 테스트 프레임워크, `ament_lint_auto`나 `ament_lint_common` 같은 코드 품질 검사 도구들이 여기에 해당한다.25 이 태그로 선언된 의존성은 일반적인 빌드나 실행에는 전혀 영향을 주지 않으며, 오직 

  `colcon test`와 같은 테스트 관련 작업을 할 때만 사용된다. `<build_depend>`나 `<exec_depend>`에 이미 선언된 패키지를 `<test_depend>`에 중복해서 선언해서는 안 된다.6

- **`<depend>`**: **빌드, 빌드-익스포트, 실행 시점 모두**에 필요한 의존성을 한 번에 선언하는 가장 강력하고 일반적인 태그다. C++ 패키지에서 `rclcpp`나 `std_msgs`와 같이 헤더 파일을 인클루드하여 빌드하고, 라이브러리에 링크하며, 실행 시점에도 해당 기능이 필요한 경우에 사용한다. 어떤 태그를 써야 할지 헷갈릴 때는 일단 `<depend>`를 사용하면 대부분의 경우 문제가 해결되므로 안전한 선택지가 될 수 있다. 하지만 패키지의 의도를 명확하게 하고 불필요한 의존성 전파를 막기 위해서는 각 목적에 맞는 특정 태그를 사용하는 것이 권장된다.6

#### 3.2.1 의존성 태그 선택 가이드

다음 표는 다양한 상황에서 어떤 의존성 태그를 선택해야 하는지에 대한 빠른 참조 가이드다.

| 태그                    | 목적 (언제 사용하는가?)                                 | C++ 패키지 예시                    | Python 패키지 예시               |
| ----------------------- | ------------------------------------------------------- | ---------------------------------- | -------------------------------- |
| `<buildtool_depend>`    | 패키지를 빌드하는 데 필요한 도구 자체                   | `ament_cmake`                      | (사용 안 함)                     |
| `<build_depend>`        | 소스코드(.cpp)나 비공개 헤더에서만 필요한 의존성        | `Boost` (내부 구현용)              | (거의 사용 안 함)                |
| `<build_export_depend>` | 공개 헤더(.hpp)에서 `#include`하는 의존성 (전이 의존성) | `geometry_msgs` (공개 API에 사용)  | (거의 사용 안 함)                |
| `<exec_depend>`         | 노드나 스크립트를 실행할 때만 필요한 의존성             | 런치 파일에서 실행하는 다른 패키지 | `rclpy`, `numpy`                 |
| `<depend>`              | 빌드, 링크, 실행 전반에 걸쳐 모두 필요한 의존성         | `rclcpp`, `std_msgs`               | (사용 안 함, `exec_depend` 선호) |
| `<test_depend>`         | 테스트를 빌드하고 실행할 때만 필요한 의존성             | `ament_cmake_gtest`                | `ament_flake8`, `pytest`         |

### 3.3  고급 의존성 속성

포맷 3에서는 의존성을 더욱 정교하게 제어할 수 있는 속성들을 제공한다.

- **버전 제한 (`version_lt`, `version_lte`, `version_eq`, `version_gte`, `version_gt`)**: 특정 패키지의 버전을 제한할 수 있다. 이는 의존하는 패키지의 API가 변경되어 하위 호환성이 깨졌을 때 매우 유용하다. 예를 들어, `my_lib` 패키지의 `2.0.0` 버전부터 API가 변경되었다면 다음과 같이 선언하여 호환되지 않는 버전의 사용을 막을 수 있다.7

  ```XML
  <depend version_lt="2.0.0">my_lib</depend>
  ```

  - `version_lt`: ~보다 작은 (`<`)
  - `version_lte`: ~보다 작거나 같은 (`<=`)
  - `version_eq`: ~와 같은 (`==`)
  - `version_gte`: ~보다 크거나 같은 (`>=`)
  - `version_gt`: ~보다 큰 (`>`)

- **조건부 의존성 (`condition`)**: `package.xml` 포맷 3의 가장 강력한 기능 중 하나로, 특정 조건이 만족될 때만 의존성이 활성화되도록 할 수 있다. 이 조건은 Python 표현식으로 평가되며, `$ROS_VERSION`과 같은 환경 변수를 사용할 수 있다. 가장 대표적인 사용 사례는 ROS 1과 ROS 2를 동시에 지원하는 패키지를 만드는 것이다.7

  ```XML
  <depend condition="$ROS_VERSION == 1">roscpp</depend>
  <depend condition="$ROS_VERSION == 2">rclcpp</depend>
  ```

  `ros_environment` 패키지를 설치하면 ROS 환경을 소싱할 때 `$ROS_VERSION` 환경 변수가 `1` 또는 `2`로 설정된다. 이를 통해 하나의 `package.xml` 파일로 두 ROS 버전에 대한 의존성을 깔끔하게 관리할 수 있다.28

## 4.  생태계와의 연결고리: `<export>` 태그

`<export>` 태그는 패키지가 자신의 정보를 ROS 생태계의 다른 부분(빌드 시스템, 툴 등)에 "수출(export)"하는 데 사용된다. 이 태그 안에 포함된 정보는 패키지가 어떻게 빌드되어야 하는지, 어떤 특별한 속성을 가지고 있는지 등을 알려주는 중요한 메타데이터다.19

### 4.1  빌드 타입 선언: `<build_type>`

`<export>` 섹션에서 가장 중요하고 필수적인 태그는 `<build_type>`이다. 이 태그는 `colcon`에게 이 패키지를 어떤 빌드 시스템을 사용하여 빌드해야 하는지를 명시적으로 알려준다.3

- **`<build_type>ament_cmake</build_type>`**: 이 패키지가 `ament_cmake`를 사용하는 C++ 기반 패키지임을 나타낸다. `colcon`은 이 태그를 보고 해당 패키지 디렉토리에서 `CMakeLists.txt` 파일을 찾아 CMake 빌드 프로세스를 실행한다.3
- **`<build_type>ament_python</build_type>`**: 이 패키지가 순수 Python 패키지임을 나타낸다. `colcon`은 이 태그를 보고 `setup.py`와 `setup.cfg` 파일을 사용하여 Python의 표준 `setuptools` 기반으로 패키지를 빌드하고 설치한다.20

만약 C++과 Python 코드가 함께 있는 하이브리드 패키지를 만들고 싶다면, 기본 빌드 타입은 `ament_cmake`로 설정해야 한다. 그리고 `CMakeLists.txt` 안에서 `ament_cmake_python` 패키지가 제공하는 함수들을 사용하여 Python 코드를 처리하도록 설정한다. 이 경우 `package.xml`에는 `<buildtool_depend>ament_cmake_python</buildtool_depend>` 의존성을 추가해야 한다.26

### 4.2  그룹 의존성: `<member_of_group>`과 `<group_depend>`

이는 `package.xml` 포맷 3에서 도입된 고급 기능으로, 여러 패키지를 논리적인 그룹으로 묶어 관리할 수 있게 해준다. 개별 패키지 이름을 일일이 나열하는 대신, 특정 "그룹"에 대한 의존성을 선언하는 방식이다.7

- **`<member_of_group>`**: 특정 패키지가 어떤 그룹의 멤버임을 선언한다.
- **`<group_depend>`**: 특정 그룹에 속한 모든 멤버 패키지들에 대한 의존성을 선언한다.

가장 대표적인 사용 사례는 ROS 인터페이스(메시지, 서비스, 액션) 패키지들이다.

1. `std_msgs`, `geometry_msgs`와 같이 `.msg`, `.srv`, `.action` 파일을 포함하는 모든 인터페이스 패키지들은 자신의 `package.xml`에 다음과 같이 선언한다:

   ```XML
   <export>
     <member_of_group>rosidl_interface_packages</member_of_group>
   </export>
   ```

2. `ros1_bridge`나 `rosidl_default_generators`와 같이 워크스페이스에 있는 모든 인터페이스 정의를 알아야 하는 패키지들은 개별 인터페이스 패키지 이름을 나열하는 대신, 다음과 같이 그룹 전체에 대한 의존성을 선언한다:

   ```XML
   <group_depend>rosidl_interface_packages</group_depend>
   ```

이 메커니즘을 통해 `colcon`은 `rosidl_interface_packages` 그룹의 멤버인 모든 패키지들을 먼저 빌드한 후, 이 그룹에 의존하는 `ros1_bridge` 같은 패키지를 빌드하도록 순서를 보장할 수 있다. 이는 워크스페이스에 어떤 인터페이스 패키지가 추가되거나 제거되더라도 `ros1_bridge`의 `package.xml`을 수정할 필요가 없게 만들어주므로, 시스템의 모듈성과 확장성을 크게 향상시킨다.7

### 4.3  기타 `<export>` 태그

- **`<architecture_independent>`**: 이 태그가 존재하면, 해당 패키지가 CPU 아키텍처에 종속적이지 않음을 나타낸다. 순수 Python 패키지, 런치 파일이나 설정 파일만 담고 있는 패키지, 메시지 정의만 담고 있는 패키지 등이 여기에 해당한다. 빌드 팜에서는 이 정보를 활용하여 여러 아키텍처에 대한 빌드를 한 번만 수행하는 등 최적화를 할 수 있다.

  ```XML
  <export>
    <architecture_independent/>
  </export>
  ```

- **`<deprecated>`**: 패키지가 더 이상 사용되지 않거나 새로운 패키지로 대체되었음을 알릴 때 사용한다. 이 태그가 있으면 `colcon`이 빌드 시 경고 메시지를 출력하여 사용자에게 알려준다.

  ```XML
  <export>
    <deprecated>This package is deprecated, please use 'new_package' instead.</deprecated>
  </export>
  ```

- **Pluginlib Export**: ROS 1에서는 플러그인을 등록하기 위해 `package.xml`의 `<export>` 태그 안에 플러그인 설명 XML 파일의 경로를 명시했다. 하지만 ROS 2에서는 이 방식이 변경되었다. 이제는 `CMakeLists.txt` 파일 안에서 `pluginlib_export_plugin_description_file()`이라는 CMake 매크로를 사용하여 플러그인을 등록하고 내보낸다. 따라서 ROS 2에서 `pluginlib`를 사용할 때 `package.xml`의 `<export>` 태그는 직접적인 역할을 하지 않는다. 이 변경점은 빌드 시스템과의 통합을 더 강화하기 위한 것이다.35

## 5.  유형별 `package.xml` 실전 예제

이론적인 설명을 넘어, 실제 프로젝트에서 사용되는 `package.xml` 예제를 통해 각 태그가 어떻게 활용되는지 살펴보자.

### 5.1  `ament_cmake` C++ 패키지 예제

간단한 `rclcpp` 기반의 퍼블리셔 노드를 포함하는 C++ 패키지의 `package.xml` 예제다.

```XML
<?xml version="1.0"?>
<?xml-model href="http://download.ros.org/schema/package_format3.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
<package format="3">
  <name>cpp_pubsub</name>
  <version>1.0.0</version>
  <description>A simple C++ publisher and subscriber example.</description>
  <maintainer email="dev@example.com">Jane Doe</maintainer>
  <license>Apache-2.0</license>

  <buildtool_depend>ament_cmake</buildtool_depend>

  <depend>rclcpp</depend>
  <depend>std_msgs</depend>

  <test_depend>ament_cmake_gtest</test_depend>
  <test_depend>ament_lint_auto</test_depend>
  <test_depend>ament_lint_common</test_depend>

  <export>
    <build_type>ament_cmake</build_type>
  </export>
</package>
```

**해설**:

- `<buildtool_depend>ament_cmake</buildtool_depend>`: `ament_cmake`가 제공하는 CMake 함수들(예: `ament_target_dependencies`, `ament_package`)을 `CMakeLists.txt`에서 사용하기 때문에 필수적이다.5
- `<depend>rclcpp</depend>`와 `<depend>std_msgs</depend>`: 소스 코드에서 `rclcpp/rclcpp.hpp`와 `std_msgs/msg/string.hpp`를 `#include`하고, 생성된 실행 파일은 `rclcpp`와 `std_msgs` 라이브러리에 링크되며, 실행 시점에도 이 라이브러리들이 필요하다. 따라서 빌드, 익스포트, 실행 모든 단계에 걸친 의존성이므로 `<depend>` 태그가 가장 적합하다.38
- `<test_depend>`: `ament_cmake_gtest`는 Google Test 프레임워크를 CMake와 통합해주는 역할을 한다. `ament_lint_auto`와 `ament_lint_common`은 `cpplint`, `cppcheck` 등 다양한 린터를 실행하여 코드 스타일과 잠재적 오류를 검사한다. 이들은 제품 코드의 빌드나 실행과는 무관하므로 `<test_depend>`로 선언하는 것이 올바르다.25
- `<export>`의 `<build_type>`: `colcon`이 이 패키지를 빌드할 때 `cmake` 명령어를 사용해야 함을 알려준다.3

### 5.2  `ament_python` Python 패키지 예제

간단한 `rclpy` 기반의 노드를 포함하는 순수 Python 패키지의 `package.xml` 예제다.

```XML
<?xml version="1.0"?>
<?xml-model href="http://download.ros.org/schema/package_format3.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
<package format="3">
  <name>py_pubsub</name>
  <version>1.0.0</version>
  <description>A simple Python publisher and subscriber example.</description>
  <maintainer email="dev@example.com">John Doe</maintainer>
  <license>Apache-2.0</license>

  <exec_depend>rclpy</exec_depend>
  <exec_depend>std_msgs</exec_depend>

  <test_depend>ament_copyright</test_depend>
  <test_depend>ament_flake8</test_depend>
  <test_depend>ament_pep257</test_depend>
  <test_depend>python3-pytest</test_depend>

  <export>
    <build_type>ament_python</build_type>
  </export>
</package>
```

**해설**:

- `<exec_depend>rclpy</exec_depend>`와 `<exec_depend>std_msgs</depend>`: Python 패키지는 별도의 컴파일/링크 단계가 없다. 의존성은 스크립트가 실행될 때 `import` 구문을 통해 해결된다. 따라서 빌드 시점 의존성인 `<build_depend>`는 필요 없고, 실행 시점 의존성인 `<exec_depend>`를 사용한다. 이것이 C++ 패키지와의 가장 큰 차이점이다.6
- `<test_depend>`: Python 코드의 테스트에는 `pytest`가 주로 사용되며, 코드 스타일 검사를 위해 `flake8`, 문서 문자열(docstring) 검사를 위해 `pep257` 등을 사용한다. 이들은 모두 테스트 과정에서만 필요하다.
- `<export>`의 `<build_type>`: `colcon`이 이 패키지를 빌드할 때 `setup.py`를 실행해야 함을 알려준다.31

### 5.3  메타패키지(Metapackage) 예제

ROS 2에서 메타패키지는 여러 관련 패키지를 하나의 논리적 단위로 묶어 사용자가 편리하게 설치하고 관리할 수 있도록 돕는 역할을 한다. ROS 1에서는 `<metapackage/>`라는 전용 태그가 있었지만, ROS 2에서는 이 태그가 사라졌다. 대신, 실제 코드나 빌드 로직 없이 오직 다른 패키지에 대한 실행 시점 의존성(`exec_depend`)만을 포함하는 일반 패키지로 메타패키지를 구성한다.13

`ros_gz` 메타패키지의 `package.xml` 예제를 살펴보자. 이 패키지는 ROS 2와 Gazebo 시뮬레이터를 연동하는 데 필요한 핵심 패키지들을 묶어준다.40

```XML
<?xml version="1.0"?>
<?xml-model href="http://download.ros.org/schema/package_format2.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
<package format="2"> <name>ros_gz</name>
  <version>2.1.6</version>
  <description>Meta-package containing interfaces for using ROS 2 with Gazebo simulation.</description>
  <maintainer email="adityapande@intrinsic.ai">Aditya Pande</maintainer>
  <license>Apache 2.0</license>

  <buildtool_depend>ament_cmake</buildtool_depend>

  <exec_depend>ros_gz_bridge</exec_depend>
  <exec_depend>ros_gz_sim</exec_depend>
  <exec_depend>ros_gz_sim_demos</exec_depend>
  <exec_depend>ros_gz_image</exec_depend>

  <test_depend>ament_lint_auto</test_depend>
  <test_depend>ament_lint_common</test_depend>

  <export>
    <build_type>ament_cmake</build_type>
  </export>
</package>
```

**해설**:

- **핵심은 `<exec_depend>`**: 이 패키지는 `ros_gz_bridge`, `ros_gz_sim` 등 Gazebo 연동에 필수적인 패키지들을 `<exec_depend>`로 포함하고 있다. 사용자가 `sudo apt install ros-humble-ros-gz` 명령을 실행하면, 패키지 관리자는 이 의존성 목록을 보고 명시된 모든 패키지를 함께 설치해준다.
- **빈 `CMakeLists.txt`**: 메타패키지의 `CMakeLists.txt`는 거의 비어 있으며, `ament_package()`를 호출하여 ROS 2 패키지 시스템에 자신을 등록하는 최소한의 내용만 담고 있다. 컴파일할 소스 코드가 없기 때문이다.41
- **용도**: `navigation2`, `slam_toolbox`, `moveit` 등 복잡한 기능을 제공하는 대규모 프로젝트들은 사용자가 수십 개의 개별 패키지를 일일이 설치하는 불편을 겪지 않도록 최상위 메타패키지를 제공하는 것이 일반적이다.

## 6.  모범 사례(Best Practices) 및 흔한 실수

잘 작성된 `package.xml`은 프로젝트의 유지보수성, 재사용성, 협업 효율성을 크게 향상시킨다. 다음은 다년간의 ROS 개발 경험을 통해 정립된 모범 사례와 초보자들이 흔히 저지르는 실수들이다.

### 6.1  패키지 구성 전략

효과적인 `package.xml` 작성은 올바른 패키지 구성에서부터 시작된다.

- **단일 책임 원칙 (Single Responsibility Principle)**: 하나의 패키지는 하나의 명확한 목적을 가져야 한다. 예를 들어, 로봇의 URDF 모델, 실제 하드웨어 드라이버, 시뮬레이션 설정, SLAM 알고리즘, 경로 계획 알고리즘은 각각 별개의 패키지로 분리하는 것이 좋다. 너무 많은 기능을 하나의 패키지에 담으면 의존성이 복잡해지고, 일부 기능만 재사용하기가 어려워지며, 여러 개발자가 동시에 작업할 때 충돌이 발생하기 쉽다.42
- **인터페이스 패키지 분리**: 로봇 프로젝트에서 사용되는 커스텀 메시지(`.msg`), 서비스(`.srv`), 액션(`.action`) 정의는 `my_robot_msgs`와 같은 별도의 전용 패키지에 모아두는 것이 매우 좋은 전략이다. 이렇게 하면, 인터페이스 정의와 이를 사용하는 로직(C++ 또는 Python 코드)이 분리된다. 만약 인터페이스와 로직이 같은 패키지에 있다면, 두 패키지가 서로의 메시지를 사용해야 할 경우 피할 수 없는 순환 의존성(circular dependency)이 발생하게 된다. 인터페이스를 별도 패키지로 분리하면 모든 로직 패키지들이 이 인터페이스 패키지에만 의존하는 단방향 의존성 구조를 만들 수 있어 프로젝트 전체의 안정성이 높아진다.42
- **런치 파일 및 설정 패키지 분리**: 로봇을 구동하기 위한 복잡한 런치 파일들과 파라미터 설정 파일(`.yaml`)들은 `my_robot_bringup`과 같은 전용 패키지에 모아서 관리하는 것이 좋다. 각 기능 패키지마다 런치 파일을 흩어 놓으면, 하나의 런치 파일이 다른 패키지의 런치 파일을 `include`하고, 또 다른 패키지의 설정 파일을 참조하는 등 의존 관계가 거미줄처럼 얽히게 된다. 모든 시작 절차를 `bringup` 패키지에 중앙화하면, 시스템 전체의 구동 방식을 한눈에 파악하기 쉽고, 의존성 관리가 단순해진다. 이 `bringup` 패키지의 `package.xml`에는 런치 파일에서 실행하는 모든 노드가 속한 패키지들을 `<exec_depend>`로 명시해야 한다.24

### 6.2  의존성 관리 모범 사례

- **최소한의 원칙**: 패키지에 꼭 필요한 최소한의 의존성만 선언해야 한다. 불필요한 의존성은 빌드 시간을 늘리고, 패키지 배포를 복잡하게 만들며, 다른 패키지와의 충돌 가능성을 높인다.
- **정확한 태그 사용**: `<depend>`는 편리하지만 남용해서는 안 된다. 의존성의 성격에 맞게 `<build_depend>`, `<build_export_depend>`, `<exec_depend>`, `<test_depend>`를 정확히 구분하여 사용하면 패키지의 의도를 명확하게 전달할 수 있다. 이는 특히 바이너리 데비안 패키지를 생성할 때, 실행에 불필요한 개발용 라이브러리(-dev 패키지)가 함께 패키징되는 것을 막아주어 더 가볍고 효율적인 배포를 가능하게 한다.6
- **순환 의존성 금지**: 패키지 A가 패키지 B에 의존하고, 동시에 패키지 B가 패키지 A에 의존하는 순환 의존성 구조는 절대적으로 피해야 한다. `colcon`은 이러한 구조의 워크스페이스를 빌드할 수 없다. 순환 의존성이 발생했다면, 이는 패키지 설계가 잘못되었다는 강력한 신호이며, 공통 기능을 별도의 패키지로 분리하는 등 리팩토링이 필요하다.42

### 6.3  버전 관리와 배포

- **유의적 버전(Semantic Versioning) 준수**: `package.xml`의 `<version>` 태그는 유의적 버전 규칙을 따라야 한다.14

  - `PATCH` 버전 (x.y.Z): 하위 호환성을 유지하는 버그 수정.

  - `MINOR` 버전 (x.Y.z): 하위 호환성을 유지하면서 새로운 기능이 추가됨.

  - MAJOR 버전 (X.y.z): 하위 호환성이 깨지는 API 변경이 발생함.

    이 규칙을 따르면 다른 사용자들이 버전 번호만 보고도 업데이트의 심각성을 파악하고 자신의 코드에 미칠 영향을 예측할 수 있다.45

- **`package.xml`과 Git 태그의 동기화**: ROS 패키지를 공식적으로 배포할 때는 `bloom`이라는 도구를 사용한다. `bloom`은 Git 저장소의 태그를 기준으로 릴리스를 생성한다. 일반적인 워크플로우는 다음과 같다 15:

  1. `CHANGELOG.rst` 파일을 업데이트하여 변경 사항을 기록한다.

  2. `package.xml` 파일의 `<version>` 태그를 새로운 릴리스 버전(예: `1.1.0`)으로 수정한다.

  3. 변경된 두 파일을 커밋한다.

  4. 해당 커밋에 `package.xml`의 버전과 동일한 이름의 Git 태그(예: `1.1.0`)를 생성한다.

  5. 이 태그를 원격 저장소에 푸시한 후 bloom-release 명령을 실행한다.

     이 과정을 통해 코드의 특정 상태(Git 태그)와 패키지 버전(package.xml)이 명확하게 연결되어, 재현 가능하고 신뢰할 수 있는 배포가 가능해진다.

### 6.4  흔히 저지르는 실수와 해결책

- **실수: 잘못된 의존성 태그 사용**
  - **증상**: Python 노드 패키지의 `rclpy` 의존성을 `<build_depend>`로 선언했다. `colcon build`는 통과하지만 `ros2 run`으로 노드를 실행하면 `ModuleNotFoundError`가 발생한다.
  - **원인**: Python 의존성은 실행 시점에 필요하므로 `<exec_depend>`로 선언해야 한다.
  - **해결**: `<build_depend>rclpy</build_depend>`를 `<exec_depend>rclpy</exec_depend>`로 수정한다.
- **실수: 전이 의존성 누락**
  - **증상**: 내 라이브러리 패키지(P1)는 잘 빌드되지만, 이 라이브러리를 사용하는 다른 패키지(P3)를 빌드할 때 `geometry_msgs/msg/pose.hpp: No such file or directory`와 같은 컴파일 오류가 발생한다.
  - **원인**: P1의 공개 헤더 파일이 `geometry_msgs`의 헤더를 `#include`하고 있지만, `package.xml`에 `<build_export_depend>geometry_msgs</build_export_depend>`를 선언하지 않았다.
  - **해결**: P1의 `package.xml`에 `<build_export_depend>geometry_msgs</build_export_depend>`를 추가하여 의존성 정보를 P3에게 올바르게 전파한다.
- **실수: `package.xml`과 빌드 스크립트의 불일치**
  - **증상**: `rosdep install`은 성공적으로 완료되지만, `colcon build`를 실행하면 `CMake Error: Could not find a package configuration file provided by "some_package"` 오류가 발생한다.
  - **원인**: `package.xml`에는 `<depend>some_package</depend>`가 선언되어 있지만, `CMakeLists.txt`에서 `find_package(some_package REQUIRED)` 호출을 누락했다.
  - **해결**: `package.xml`과 `CMakeLists.txt`(또는 `setup.py`)의 의존성 목록을 항상 일치시킨다.
- **실수: XML 문법 오류**
  - **증상**: `colcon build` 실행 시 패키지를 아예 인식하지 못하거나 XML 파싱 오류가 발생한다.
  - **원인**: 태그를 닫지 않았거나(`</depend>`), 속성 값에 따옴표를 빠뜨리는 등 단순한 문법 오류가 있다.
  - **해결**: `ament_lint`의 일부인 `ament_xmllint`를 사용하여 `package.xml` 파일의 유효성을 검사할 수 있다. `colcon test`를 실행하면 이 검사가 자동으로 수행된다. 또한, XSD 스키마를 지원하는 VS Code와 같은 편집기를 사용하면 실시간으로 오류를 발견하는 데 도움이 된다.11

## 7.  문제 해결(Troubleshooting) 가이드

개발 과정에서 `package.xml`과 관련된 다양한 문제에 직면할 수 있다. 다음은 가장 흔하게 발생하는 오류들과 그 원인 및 해결 방법이다.

### 7.1  `rosdep install` 실패: "Cannot locate rosdep definition for [key_name]"

이 오류는 `rosdep`이 `package.xml`에 명시된 의존성 키에 해당하는 시스템 패키지를 `rosdistro` 데이터베이스에서 찾지 못할 때 발생한다.49

- 원인 1: 단순 오타 또는 잘못된 키 이름

  가장 흔한 원인이다. rosdep 키는 실제 시스템 패키지 이름(예: libpcl-dev)과 다를 수 있다. 예를 들어, libpcl-dev에 대한 올바른 rosdep 키는 pcl_ros 또는 libpcl-all-dev일 수 있으며, 이는 ROS 배포판과 상황에 따라 다르다.

  - **해결책**:
    1. 의심되는 키 이름에 오타가 없는지 확인한다.
    2. `rosdep keys <key_name>` 명령어로 해당 키가 존재하는지, 존재한다면 어떤 시스템 패키지에 매핑되는지 확인한다.
    3. ROS Index 웹사이트(index.ros.org)나 `rosdistro` GitHub 저장소의 `rosdep/base.yaml` 또는 `rosdep/python.yaml` 파일을 직접 검색하여 올바른 키 이름을 찾는다.52

- 원인 2: rosdistro 데이터베이스에 없는 키

  비교적 새로운 라이브러리나 ROS 생태계에서 자주 사용되지 않는 라이브러리는 rosdistro에 등록되어 있지 않을 수 있다.

  - **해결책**:
    1. **(권장)** `rosdistro` 저장소에 풀 리퀘스트(Pull Request)를 보내 새로운 `rosdep` 규칙을 추가한다. 이는 커뮤니티에 기여하는 가장 좋은 방법이다.6
    2. **(임시/로컬)** 로컬 `rosdep` 소스 파일을 만들어 개인적으로 사용하는 키를 정의할 수 있다. 예를 들어, `my_rules.yaml` 파일을 만들고 `/etc/ros/rosdep/sources.list.d/50-local.list` 파일에 해당 파일의 경로를 추가한 후 `rosdep update`를 실행하면 로컬 규칙이 적용된다.55

- 원인 3: ROS_DISTRO 환경 변수 누락

  rosdep은 어떤 ROS 배포판을 기준으로 의존성을 해결해야 하는지 알아야 한다. 이 정보가 없으면 ROS 관련 키를 찾지 못한다.

  - **해결책**:
    1. 터미널에서 `source /opt/ros/humble/setup.bash` 명령을 실행하여 ROS 환경을 올바르게 설정했는지 확인한다. `echo $ROS_DISTRO`를 실행했을 때 `humble`이 출력되어야 한다.
    2. 환경 설정이 어려운 경우, `rosdep install` 명령어에 `--rosdistro humble` 옵션을 명시적으로 추가하여 배포판을 지정해준다.57

### 7.2  `colcon build` 실패: "Package 'package_name' not found"

`colcon`이 워크스페이스 내에서 특정 패키지를 인식하지 못할 때 발생하는 오류다.

- 원인 1: 잘못된 워크스페이스 구조

  colcon은 특정 디렉토리 구조를 기대한다. 패키지 소스 코드는 워크스페이스 루트의 src 디렉토리 아래에 위치해야 한다.

  - **해결책**: 패키지가 `~/ros2_ws/src/my_package`와 같이 올바른 위치에 있는지 확인하고, `colcon build` 명령은 워크스페이스의 루트 디렉토리(`~/ros2_ws`)에서 실행해야 한다.

- 원인 2: package.xml 파일의 부재 또는 오류

  colcon은 디렉토리 안에 package.xml 파일이 있어야만 해당 디렉토리를 ROS 패키지로 인식한다.

  - **해결책**:
    1. 패키지 디렉토리 최상단에 `package.xml` 파일이 존재하는지 확인한다.
    2. 파일 이름이 정확한지 확인한다.
    3. `package.xml` 파일의 내용에 심각한 XML 문법 오류가 없는지 확인한다. 특히 `<name>` 태그의 이름이 실제 패키지 디렉토리 이름과 달라서 발생하는 경우가 많다. `package.xml`의 `<name>`과 `CMakeLists.txt`의 `project()` 이름, 그리고 실제 폴더 이름은 일치시키는 것이 좋다.58

- 원인 3: 의존하는 패키지의 빌드 실패

  --packages-select 옵션으로 특정 패키지만 빌드할 때, 해당 패키지가 의존하는 다른 패키지가 아직 빌드되지 않았거나 빌드에 실패한 경우 이 오류가 발생할 수 있다.

  - **해결책**: `--packages-up-to <package_name>` 옵션을 사용하면 해당 패키지와 그 패키지가 의존하는 모든 패키지들을 함께 빌드해주므로 이 문제를 해결할 수 있다.

### 7.3  `colcon build` 실패: CMake/Python 관련 오류

빌드 과정 자체에서 발생하는 오류로, `package.xml`과 빌드 스크립트 간의 불일치로 인해 발생하는 경우가 많다.

- **원인: `package.xml`과 빌드 스크립트의 의존성 불일치**
  - `CMake Error at CMakeLists.txt:10 (find_package): Could not find a package configuration file...`: `package.xml`에는 의존성을 선언했지만, `CMakeLists.txt`에서 `find_package()`를 호출하는 것을 잊었거나, 그 반대의 경우다.
  - `ModuleNotFoundError: No module named 'some_ros_package'`: Python 패키지에서 발생하는 경우로, `package.xml`에 `<exec_depend>`를 선언했지만 `setup.py`의 `install_requires`에 누락되었거나, 혹은 그 반대의 경우다.
  - **해결책**: `package.xml`의 의존성 목록과 `CMakeLists.txt`의 `find_package()` 목록, `setup.py`의 `install_requires` 목록이 항상 일관성을 유지하도록 철저히 확인해야 한다. `catkin_lint`와 같은 ROS 1 도구가 ROS 2에서는 아직 완벽하게 지원되지 않으므로, 이 부분은 개발자가 직접 주의를 기울여야 한다.11

## 8. 결론: 잘 작성된 `package.xml`의 가치

지금까지 ROS 2 Humble의 `package.xml`에 대해 깊이 있게 탐구했다. 이 파일이 단순한 메타데이터의 집합이 아니라, ROS 2 생태계의 모든 도구와 소통하며 패키지의 빌드, 테스트, 배포, 실행의 모든 과정을 관장하는 핵심적인 "계약서"임을 확인했다.

정확하고 명료하게 작성된 `package.xml`은 다음과 같은 가치를 제공한다.

- **자동화된 의존성 관리**: `rosdep`을 통해 단 한 번의 명령으로 복잡한 시스템 의존성을 해결하여 개발 환경 설정을 극적으로 단순화한다.
- **신뢰할 수 있는 빌드**: `colcon`이 패키지 간의 관계를 명확히 파악하여 항상 올바른 순서로 빌드를 수행하도록 보장하며, 순환 의존성과 같은 잠재적 문제를 사전에 방지한다.
- **향상된 협업과 재사용성**: 패키지의 목적, 라이선스, 유지보수 정보, 그리고 의존성을 명확히 함으로써 다른 개발자들이 패키지를 쉽게 이해하고 안전하게 재사용할 수 있는 기반을 마련한다.
- **체계적인 배포**: 유의적 버전 관리와 `bloom` 도구와의 연동을 통해, 코드의 특정 버전을 안정적으로 패키징하고 배포하는 체계적인 릴리스 파이프라인을 구축할 수 있게 한다.

결론적으로, `package.xml`을 능숙하게 다루는 능력은 ROS 2 개발자의 핵심 역량 중 하나다. 이 안내서에서 다룬 원칙과 모범 사례들을 꾸준히 적용한다면, 더 안정적이고, 유지보수하기 쉬우며, 확장 가능한 로봇 소프트웨어를 개발하는 데 큰 도움이 될 것이다. `package.xml`은 사소한 설정 파일이 아니라, 훌륭한 로봇 공학 프로젝트를 지탱하는 견고한 초석이다.

#### **참고 자료**

1. Creating a package - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Creating-Your-First-ROS2-Package.html
2. Creating a package - ROS 2 Documentation: Foxy documentation, accessed July 27, 2025, https://docs.ros.org/en/foxy/Tutorials/Beginner-Client-Libraries/Creating-Your-First-ROS2-Package.html
3. The build system - ROS 2 Documentation: Iron documentation, accessed July 27, 2025, https://docs.ros.org/en/iron/Concepts/Advanced/About-Build-System.html
4. ros - Dependency management in ROS2: CMakeLists.txt, package ..., accessed July 27, 2025, https://robotics.stackexchange.com/questions/96942/dependency-management-in-ros2-cmakelists-txt-package-xml-colcon-build-make
5. The build system - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Concepts/Advanced/About-Build-System.html
6. Managing Dependencies with rosdep - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/Rosdep.html
7. REP 149 -- Package Manifest Format Three Specification (ROS.org), accessed July 27, 2025, https://ros.org/reps/rep-0149.html
8. ROS Cheat Sheet | Clearpath Robotics Documentation, accessed July 27, 2025, https://docs.clearpathrobotics.com/docs/ros/tutorials/cheat_sheet
9. https://ros2-tutorial.readthedocs.io/en/latest/_downloads/d105360c187244c8f4ad65b892cff4c1/package.xml, accessed July 27, 2025, https://ros2-tutorial.readthedocs.io/en/latest/_downloads/d105360c187244c8f4ad65b892cff4c1/package.xml
10. ros1_bridge/package.xml at master - GitHub, accessed July 27, 2025, https://github.com/ros2/ros1_bridge/blob/master/package.xml
11. How do you validate your package.xmls in ROS2? - General - ROS Discourse, accessed July 27, 2025, https://discourse.ros.org/t/how-do-you-validate-your-package-xmls-in-ros2/23875
12. ROS2 C++ Package Creation Guide | ROS2 Tutorial - The Construct, accessed July 27, 2025, https://www.theconstruct.ai/ros2-cpp-package-creation-guide-ros2-tutorial/
13. Migrating Packages - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/How-To-Guides/Migrating-from-ROS1/Migrating-Packages.html
14. Semantic Versioning 2.0.0 | Semantic Versioning, accessed July 27, 2025, https://semver.org/
15. How to release a ROS 2 binary package - Part 3 - The Construct, accessed July 27, 2025, https://www.theconstruct.ai/how-to-release-a-ros-2-binary-package-part-3/
16. nav2_behavior_tree/package.xml / main / undefined / GitLab, accessed July 27, 2025, https://git.tu-berlin.de/ecschuetz/navigation2/-/blob/main/nav2_behavior_tree/package.xml?ref_type=heads
17. example_interfaces/package.xml at rolling / ros2 ... - GitHub, accessed July 27, 2025, https://github.com/ros2/example_interfaces/blob/rolling/package.xml
18. pluginlib - ROS Wiki, accessed July 27, 2025, http://wiki.ros.org/pluginlib
19. Package manifest XML tags reference - ROS, accessed July 27, 2025, http://ros.org/doc/independent/api/rospkg/html/manifest_xml.html
20. The build system "ament_cmake" and the meta build tool "ament_tools" - ROS2 Design, accessed July 27, 2025, https://design.ros2.org/articles/ament.html
21. Managing Dependencies with rosdep - ROS 2 Documentation: Foxy documentation, accessed July 27, 2025, https://docs.ros.org/en/foxy/Tutorials/Intermediate/Rosdep.html
22. catkin/package.xml - ROS Wiki, accessed July 27, 2025, http://wiki.ros.org/catkin/package.xml
23. Migration guide from ROS 1 - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://ftp.udx.icscoe.jp/ros/ros_docs_mirror/en/humble/Contributing/Migration-Guide.html
24. ROS2 XML Launch File Example - The Robotics Back-End, accessed July 27, 2025, https://roboticsbackend.com/ros2-xml-launch-file/
25. Guidelines and Best Practices, accessed July 27, 2025, https://autowarefoundation.gitlab.io/autoware.auto/AutowareAuto/contributor-guidelines.html
26. ament_cmake_python user documentation - the official ROS docs, accessed July 27, 2025, https://docs.ros.org/en/humble/How-To-Guides/Ament-CMake-Python-Documentation.html
27. Ability to query build or exec dependencies only / Issue #587 / ros-infrastructure/rosdep, accessed July 27, 2025, https://github.com/ros-infrastructure/rosdep/issues/587
28. Use identical package.xml for ROS1 and ROS2 pkgs - Robotics Stack Exchange, accessed July 27, 2025, https://robotics.stackexchange.com/questions/86231/use-identical-package-xml-for-ros1-and-ros2-pkgs
29. Support for format three package.xml conditions / Issue #653 / ros-infrastructure/rosdep - GitHub, accessed July 27, 2025, https://github.com/ros-infrastructure/rosdep/issues/653
30. About the build system - ROS 2 Documentation: Foxy documentation, accessed July 27, 2025, https://docs.ros.org/en/foxy/Concepts/About-Build-System.html
31. Migrating a Python Package Example - ROS 2 Documentation ..., accessed July 27, 2025, https://docs.ros.org/en/humble/How-To-Guides/Migrating-from-ROS1/Migrating-Python-Package-Example.html
32. Create a ROS2 package for Both Python and Cpp Nodes - The Robotics Back-End, accessed July 27, 2025, https://roboticsbackend.com/ros2-package-for-both-python-and-cpp-nodes/
33. ament_cmake_python user documentation - ROS 2 Documentation, accessed July 27, 2025, https://ftp.udx.icscoe.jp/ros/ros_docs_mirror/en/humble/How-To-Guides/Ament-CMake-Python-Documentation.html
34. Dependency from rmw_cyclonedds_cpp on fastrtps via rmw_dds_common #16 - GitHub, accessed July 27, 2025, https://github.com/ros2/rmw_dds_common/issues/16
35. Creating and Using Plugins (C++) - ROS 2 Documentation: Xin 文档, accessed July 27, 2025, https://daobook.github.io/ros2-docs/xin/Tutorials/Pluginlib.html
36. Question about how to export plugin.xml files / Issue #198 / ros/pluginlib - GitHub, accessed July 27, 2025, https://github.com/ros/pluginlib/issues/198
37. Creating and using plugins (C++) - ROS 2 Documentation: Foxy documentation, accessed July 27, 2025, https://docs.ros.org/en/foxy/Tutorials/Beginner-Client-Libraries/Pluginlib.html
38. ament_cmake user documentation - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/How-To-Guides/Ament-CMake-Documentation.html
39. ROS 2 metapackages/variants support / Issue #408 - GitHub, accessed July 27, 2025, https://github.com/ros2/ros2/issues/408
40. package.xml - gazebosim/ros_gz - GitHub, accessed July 27, 2025, https://github.com/gazebosim/ros_gz/blob/ros2/ros_gz/package.xml
41. Using variants - Galactic documentation - the official ROS docs, accessed July 27, 2025, https://docs.ros.org/en/galactic/How-To-Guides/Using-Variants.html
42. Package Organization For a ROS Stack [Best Practices] - The Robotics Back-End, accessed July 27, 2025, https://roboticsbackend.com/package-organization-for-a-ros-stack-best-practices/
43. ROS2 Part 9 - ROS2 Launch Files - RoboticsUnveiled, accessed July 27, 2025, https://www.roboticsunveiled.com/ros2-launch-files/
44. REP 140 -- Package Manifest Format Two Specification (ROS.org), accessed July 27, 2025, https://www.ros.org/reps/rep-0140.html
45. Using Git Tags for Semantic Versioning (Step-by-Step Guide) - Medium, accessed July 27, 2025, https://medium.com/@wealthiscertain/using-git-tags-for-semantic-versioning-step-by-step-guide-02b18d73a7b9
46. Managing Releases with Semantic Versioning and Git Tags - GitKraken, accessed July 27, 2025, https://www.gitkraken.com/gitkon/semantic-versioning-git-tags
47. Bloom release from a specific tag in a specific branch - ROS Discourse, accessed July 27, 2025, https://discourse.ros.org/t/bloom-release-from-a-specific-tag-in-a-specific-branch/37922
48. Invalid package.xml suppresses error and silently succeeds if setup.py present / Issue #146, accessed July 27, 2025, https://github.com/colcon/colcon-ros/issues/146
49. How to solve the ERROR Cannot locate rosdep definition for [PCL] in ROS2 Humble?, accessed July 27, 2025, https://robotics.stackexchange.com/questions/107920/how-to-solve-the-error-cannot-locate-rosdep-definition-for-pcl-in-ros2-humble
50. ERROR: the following packages/stacks could not have their rosdep keys resolved to system dependencies: I'm using Ubuntu 18.04; ROS Melodic - Reddit, accessed July 27, 2025, https://www.reddit.com/r/ROS/comments/1ac4t7f/error_the_following_packagesstacks_could_not_have/
51. Cannot locate rosdep definition - ROS Answers archive, accessed July 27, 2025, http://answers.ros.org/question/320734/
52. Managing Dependencies with rosdep - ROS 2 Documentation: Rolling documentation, accessed July 27, 2025, https://docs.ros.org/en/rolling/Tutorials/Intermediate/Rosdep.html
53. Managing Dependencies with rosdep - documentación de ROS 2 Documentation, accessed July 27, 2025, https://ros-spanish-users-group.github.io/ros2_documentation/humble/Tutorials/Intermediate/Rosdep.html
54. rosdistro/REVIEW_GUIDELINES.md at master - GitHub, accessed July 27, 2025, https://github.com/ros/rosdistro/blob/master/REVIEW_GUIDELINES.md
55. rostooling/cc-rosdep - Docker Image, accessed July 27, 2025, https://hub.docker.com/r/rostooling/cc-rosdep
56. Generate deb from dependent res package locally - Robotics Stack Exchange, accessed July 27, 2025, https://robotics.stackexchange.com/questions/84788/generate-deb-from-dependent-res-package-locally
57. Cannot locate rosdep definition for [ament_lint_common] - ROS Answers archive, accessed July 27, 2025, https://answers.ros.org/question/371969/
58. colcon can not find my package - ROS Answers archive, accessed July 27, 2025, https://answers.ros.org/question/364278/
59. [feature] CI job to verify dependencies package.xml / Issue #29685 / ros/rosdistro - GitHub, accessed July 27, 2025, https://github.com/ros/rosdistro/issues/29685