# ROS 2 Humble의 rosbag2
[ROS2 Humble](./index.md)


`rosbag2`는 Robot Operating System 2 (ROS 2) 생태계에서 데이터의 기록, 재생, 분석을 담당하는 핵심 도구이다. 이는 단순한 로깅 유틸리티를 넘어, 데이터 기반의 로봇 애플리케이션 개발, 시뮬레이션 기반의 알고리즘 검증, 시스템 디버깅, 그리고 재현 가능한 테스트 환경 구축의 근간을 이룬다. `rosbag2`는 ROS 1 시절의 `rosbag`이 수행하던 역할을 계승하면서도, 그 구조와 철학에서 근본적인 발전을 이루었다.1

ROS 1의 `rosbag`이 모든 기능을 하나의 단일 파일 포맷(`.bag`)에 강하게 결합시킨 모놀리식(monolithic) 구조였다면, `rosbag2`는 패러다임의 전환을 보여준다. `rosbag2`의 가장 중요한 설계 철학은 '모듈성(modularity)'이다. 데이터가 저장되는 방식(스토리지), 메시지가 변환되는 방식(직렬화), 그리고 압축 방식 등이 각각 독립적인 플러그인(plugin) 형태로 분리되어 있다.2 이러한 플러그인 기반 아키텍처는 사용자에게 전례 없는 유연성을 제공한다. 개발자는 자신의 애플리케이션 요구사항에 맞춰 최적의 스토리지 백엔드를 선택하거나, 심지어 직접 개발하여 사용할 수도 있다. 이 구조적 변화는 `rosbag2`의 모든 기능과 특징을 관통하는 핵심 개념이며, ROS 2가 지향하는 유연하고 확장 가능한 시스템 설계 사상을 명확하게 보여준다.4

이 보고서는 ROS 2 Humble Hawksbill 배포판을 기준으로 `rosbag2`의 기능적 측면을 상세히 설명하는 것을 넘어, 이러한 구조적 변화가 실제 로보틱스 개발 환경에 어떤 영향을 미치는지, 그리고 개발자가 마주할 수 있는 실질적인 문제들을 어떻게 해결할 수 있는지에 대한 심도 깊은 분석과 실전 가이드를 제공하는 것을 목표로 한다.


`rosbag2`의 기능은 `ros2 bag`이라는 명령어를 통해 접근할 수 있다. 이 커맨드 라인 인터페이스(CLI)는 데이터 녹화, 재생, 분석을 위한 직관적인 방법을 제공하며, `rosbag2`를 처음 접하는 개발자에게 가장 기본적인 상호작용 수단이 된다.


데이터 녹화는 `ros2 bag record` 명령어를 통해 수행되며, 다양한 옵션을 통해 녹화 방식을 세밀하게 제어할 수 있다.


가장 간단한 녹화 방식은 현재 ROS 2 시스템에서 발행(publish)되고 있는 모든 토픽을 기록하는 것이다. `-a` 또는 `--all` 옵션을 사용하면 `rosbag2`는 실행 시점에 활성화된 모든 토픽을 자동으로 감지하여 녹화를 시작한다. 중요한 특징은 녹화가 진행되는 도중에 새로운 토픽이 시스템에 나타나더라도 이를 동적으로 감지하여 녹화 대상에 포함시킨다는 점이다.5 이는 시스템의 전체 상태를 빠짐없이 기록해야 할 때 매우 유용하다.

```Bash
ros2 bag record -a
```


시스템의 모든 데이터가 아닌, 특정 데이터 스트림에만 관심이 있을 경우, 녹화할 토픽의 이름을 명시적으로 지정할 수 있다. 여러 토픽은 공백으로 구분하여 나열한다.5

```Bash
ros2 bag record /turtle1/cmd_vel /turtle1/pose
```

이 방식은 녹화 시작 시점에 해당 토픽이 존재하지 않더라도 사용할 수 있다. `rosbag2`는 지정된 토픽이 활성화될 때까지 기다렸다가 녹화를 시작한다. 또한, ROS 2 Iron 버전부터는 정규 표현식(regex)을 이용한 필터링 기능이 추가되어, `/robot/camera_*`와 같이 특정 패턴을 가진 토픽들을 한 번에 지정하는 것도 가능해졌다.7


`ros2 bag record` 명령을 실행하면 기본적으로 현재 작업 디렉터리에 `rosbag2_YYYY_MM_DD-HH_MM_SS` 형식의 타임스탬프를 이름으로 하는 디렉터리가 생성된다. 이 디렉터리 안에는 실제 데이터가 담긴 파일(e.g., `.db3` 또는 `.mcap`)과 메타데이터 파일(`metadata.yaml`)이 저장된다.5 체계적인 데이터 관리를 위해서는 `-o` 또는 `--output` 옵션을 사용하여 bag 파일의 저장 경로와 디렉터리 이름을 명확하게 지정하는 것이 좋다.

```Bash
ros2 bag record /chatter -o my_first_bag
```


`-a` 옵션과 함께 사용될 때, `--no-discovery` 옵션은 녹화 시작 시점에 존재하는 토픽들만 기록하고, 녹화 중에 새로 생성되는 토픽들은 무시하도록 한다.6 이는 녹화 범위를 특정 시점으로 고정시키고 예상치 못한 토픽이 기록되는 것을 방지하여, 실험의 일관성을 유지하고 싶을 때 유용하게 사용될 수 있다.


녹화된 bag 파일은 `ros2 bag play` 명령어를 통해 재생할 수 있다. 이는 과거의 특정 시점에서 발생한 이벤트를 그대로 재현하여 알고리즘을 테스트하거나 시스템 동작을 분석하는 데 사용된다.


재생 명령어의 기본 형식은 bag 파일이 저장된 디렉터리 이름을 인자로 전달하는 것이다. `rosbag2`는 해당 디렉터리 내의 `metadata.yaml` 파일을 읽어 녹화된 토픽, 메시지 타입 등의 정보를 파악하고, 원본 타임스탬프에 맞춰 메시지를 다시 발행한다.5

```Bash
ros2 bag play my_first_bag
```


`-r` 또는 `--rate` 옵션은 재생 속도를 조절하는 데 사용된다. `-r 2.0`은 2배속으로, `-r 0.5`는 0.5배속으로 재생한다. 이 기능은 알고리즘의 성능 한계를 테스트하기 위해 시간을 압축하거나, 특정 구간을 느리게 재생하며 정밀하게 분석해야 할 때 필수적이다.11

```Bash
ros2 bag play my_first_bag -r 2.0
```


`--loop` 옵션을 사용하면 bag 파일의 재생이 끝나도 중단되지 않고, 처음부터 다시 반복하여 재생한다. 이 기능은 로봇 데모 환경을 구축하거나, 특정 시나리오에 대한 지속적인 회귀 테스트(regression test)를 수행할 때 유용하다.12

```Bash
ros2 bag play my_first_bag --loop
```


`ros2 bag info` 명령어는 녹화된 bag 파일의 내용을 재생하지 않고도 그 구조와 요약 정보를 확인할 수 있게 해준다. 이는 녹화가 의도대로 잘 수행되었는지 확인하는 첫 번째 단계이며, 데이터 분석의 시작점이 된다.

```Bash
ros2 bag info my_first_bag
```

이 명령어를 실행하면 다음과 같은 핵심 정보가 출력된다 5:

- **Files**: 데이터가 저장된 파일 목록 (e.g., `my_first_bag_0.db3`).
- **Bag Size**: bag 디렉터리의 총 크기.
- **Storage ID**: 사용된 스토리지 플러그인의 ID (`sqlite3`, `mcap` 등).
- **Duration**: 총 녹화 시간.
- **Start / End**: 녹화 시작 및 종료 시각.
- **Messages**: bag 파일에 저장된 총 메시지의 수.
- **Topic Information**: 각 토픽에 대한 상세 정보.
  - **Topic**: 토픽 이름.
  - **Type**: 메시지 타입.
  - **Count**: 해당 토픽으로 녹화된 메시지의 수.
  - **Serialization Format**: 사용된 직렬화 포맷 (일반적으로 `cdr`).

CLI는 `rosbag2`의 강력한 기능에 접근하기 위한 편리한 '진입점' 역할을 수행한다. 튜토리얼을 통해 `record`, `play`와 같은 기본 명령어를 배우는 것은 매우 쉽고 직관적으로 느껴질 수 있다.5 하지만 실제 로보틱스 애플리케이션, 특히 고대역폭의 센서 데이터를 다루는 환경에서는 이러한 단순함 이면에 숨겨진 복잡성이 드러나기 시작한다. 예를 들어, 대용량 데이터를 녹화할 때 메시지가 유실되는 현상 14이나, 

`Ctrl-C`로 녹화를 중단했을 때 프로세스가 응답하지 않고 멈추는(hang) 문제 15를 경험할 수 있다.

이러한 문제들은 CLI 명령어 자체의 결함이라기보다는, CLI가 추상화하고 있는 하부 시스템의 특성에서 기인한다. Humble의 기본 스토리지 백엔드인 SQLite3의 성능적 한계, 기본 QoS 설정의 한계, 그리고 ROS 2의 통신 근간을 이루는 DDS(Data Distribution Service)의 동작 방식 등이 복합적으로 작용한 결과이다. 이는 CLI를 능숙하게 사용하는 것을 넘어, 2장에서 다룰 파일 분할, 압축과 같은 고급 기능, 3장의 스토리지 아키텍처, 그리고 6장의 성능 최적화 전략을 깊이 이해하는 것이 `rosbag2`의 잠재력을 온전히 활용하기 위한 필수 과정임을 명확히 보여준다. 따라서 CLI 마스터링은 `rosbag2` 학습의 끝이 아닌, 더 깊은 이해를 위한 시작점이라 할 수 있다.

------


`rosbag2`는 기본적인 녹화 및 재생 기능을 넘어, 실제 로보틱스 개발 현장에서 마주하는 다양한 요구사항을 해결하기 위한 여러 고급 기능을 제공한다. 이러한 기능들은 대용량 데이터 관리, 저장 공간 효율화, 결정론적 테스트, 그리고 시스템 자동화 등 프로덕션 레벨의 애플리케이션 개발에 필수적이다.


자율주행 차량이나 장시간 운영되는 로봇 시스템에서는 몇 시간 동안 데이터를 녹화하는 경우가 흔하다. 이때 모든 데이터를 단일 파일에 저장하면 파일 크기가 수백 기가바이트에서 테라바이트에 달해 관리, 전송, 처리가 매우 비효율적이 된다. `rosbag2`의 파일 분할 기능은 이러한 문제를 해결하기 위해 제공된다.

- **크기 기준 분할 (`-b`, `--max-bag-size`)**: 이 옵션은 생성되는 개별 bag 파일의 최대 크기를 바이트(byte) 단위로 제한한다. 녹화 중인 파일의 크기가 지정된 값에 도달하면, `rosbag2`는 해당 파일을 닫고 새로운 파일(예: `my_bag_1.db3`, `my_bag_2.db3`)을 열어 녹화를 계속한다. 예를 들어, 100MB 단위로 파일을 분할하려면 `$100 * 1024 * 1024 = 104857600$` 바이트로 설정한다.1

  ```Bash
  ros2 bag record -a -b 104857600
  ```

- **시간 기준 분할 (`-d`, `--max-bag-duration`)**: 이 옵션은 각 파일의 최대 녹화 시간을 초(second) 단위로 제한한다. 지정된 시간이 경과하면 새로운 파일로 전환된다. 예를 들어, 10분(600초)마다 파일을 분할하려면 다음과 같이 명령한다.1

  ```Bash
  ros2 bag record -a -d 600
  ```

크기와 시간 기준 분할 옵션을 동시에 사용하면, 둘 중 먼저 충족되는 조건에 따라 파일이 분할된다.1 이 기능은 대용량 데이터를 관리 가능한 작은 단위로 나누어 후처리를 용이하게 하고, 데이터 전송 시 실패 위험을 줄여준다.


고해상도 카메라 이미지나 3D 라이다 포인트 클라우드와 같은 데이터는 막대한 저장 공간을 차지한다. `rosbag2`는 데이터 압축을 통해 이러한 부담을 줄일 수 있는 두 가지 수준의 메커니즘을 제공한다.

- **`rosbag2` 프레임워크 레벨 압축**: 이 방식은 스토리지 플러그인과 독립적으로 `rosbag2` 프레임워크 자체에서 압축을 처리한다.

  - `--compression-mode`: 압축 단위를 지정한다. `file`은 분할된 각 bag 파일 전체를 압축하는 방식이며, `message`는 개별 메시지 단위로 압축을 수행한다. 일반적으로 파일 분할 기능과 함께 `file` 모드를 사용하는 것이 권장된다.1
  - `--compression-format`: 사용할 압축 알고리즘을 지정한다. ROS 2 Humble에서는 `zstd`가 기본적으로 지원되며, 높은 압축률과 빠른 속도로 잘 알려져 있다.1

  ```Bash
  ros2 bag record -a --compression-mode file --compression-format zstd
  ```

- **스토리지 플러그인 네이티브 압축**: MCAP과 같은 최신 스토리지 플러그인은 프레임워크 레벨의 압축보다 훨씬 효율적인 자체 압축 메커니즘을 내장하고 있다. 예를 들어, MCAP은 데이터를 청크(chunk) 단위로 압축하는데, 이 방식은 파일 전체를 압축 해제하지 않고도 특정 데이터에 접근할 수 있는 인덱싱 기능을 유지할 수 있게 해준다. 이는 후처리 효율성 면에서 큰 장점을 가지므로, 스토리지 플러그인이 네이티브 압축을 지원한다면 이를 사용하는 것이 강력히 권장된다.1 (이 내용은 3장에서 더 자세히 다룬다.)


로봇 시스템의 테스트 및 디버깅에서 가장 중요한 요소 중 하나는 '결정론적 재현(deterministic reproduction)'이다. 즉, 동일한 입력 데이터에 대해 시스템이 항상 동일하게 동작해야 한다. 하지만 실제 시간(wall clock)을 기준으로 동작하는 시스템에서는 테스트를 실행하는 하드웨어의 성능이나 그 순간의 시스템 부하에 따라 결과가 미묘하게 달라질 수 있다.

`--use-sim-time` 파라미터는 이러한 문제를 해결하는 핵심적인 기능이다. ROS 노드를 실행할 때 이 파라미터를 `true`로 설정하면, 해당 노드는 시스템의 실제 시간을 무시하고 `/clock`이라는 특정 토픽으로 발행되는 시간을 자신의 시간 기준으로 삼는다.18

`ros2 bag play`를 실행할 때 `--use-sim-time` 옵션을 함께 사용하면, `player` 노드는 bag 파일에 기록된 메시지들의 원본 타임스탬프에 맞춰 `/clock` 토픽을 발행한다.1 이렇게 함으로써 bag을 재생하는 ROS 시스템 전체의 시간 흐름이 녹화 당시의 시간 흐름과 완벽하게 동기화된다. 이 메커니즘 덕분에 하드웨어 성능과 무관하게 언제나 동일한 순서와 시간 간격으로 이벤트를 재현할 수 있으며, 이는 신뢰성 있는 알고리즘 검증의 필수 전제 조건이다.19


`rosbag2`의 `recorder`와 `player`는 단순한 CLI 도구가 아니라, 그 자체로 하나의 ROS 2 노드이다. 따라서 이들은 ROS 2의 표준 통신 방식인 서비스를 통해 외부에서 제어될 수 있다. 이 기능은 `rosbag2`를 자동화된 테스트 프레임워크나 데이터 수집 시스템, 또는 그래픽 사용자 인터페이스(GUI)에 통합할 때 매우 강력한 유연성을 제공한다.

- **Recorder 서비스**: `recorder` 노드는 `~/pause` (녹화 일시 중지), `~/resume` (녹화 재개), `~/split_bagfile` (수동으로 파일 분할), 그리고 `~/snapshot` (특정 조건에서만 데이터를 저장하는 스냅샷 모드 제어)과 같은 서비스를 제공한다.1 예를 들어, 테스트 스크립트에서 특정 이벤트가 발생했을 때 `ros2 service call /<recorder_node_name>/pause std_srvs/srv/Trigger`와 같은 명령으로 녹화를 잠시 멈출 수 있다.
- **Player 서비스**: `player` 노드는 `~/pause`, `~/resume` 외에도 `~/seek` (특정 타임스탬프로 재생 위치 이동), `~/set_rate` (재생 속도 동적 변경), `~/stop` (재생 완전 중지) 등 훨씬 다양한 제어 기능을 서비스로 노출한다.1 이를 통해 복잡한 디버깅 시나리오를 자동화하거나, 사용자가 GUI를 통해 재생 과정을 정밀하게 제어하는 애플리케이션을 만들 수 있다.

이러한 고급 기능들은 `rosbag2`가 단순한 데이터 로깅 도구를 넘어, '데이터를 하나의 시스템 컴포넌트'로 취급하려는 ROS 2의 발전된 설계 철학을 명확하게 보여준다. 개발자는 처음에는 수동으로 CLI를 사용하여 데이터를 기록하고 재생하는 것으로 시작한다. 하지만 곧 테스트를 자동화하거나 복잡한 실험을 설계해야 하는 상황에 직면하게 된다. 이때 서비스 인터페이스는 `ros2 launch` 파일이나 자동화 스크립트에서 `rosbag2`의 동작을 프로그래밍 방식으로 제어할 수 있는 길을 열어준다.1

마찬가지로, 재현 테스트의 결과가 실행할 때마다 달라지는 문제를 겪게 되면, 그 원인이 각 노드가 사용하는 시간 기준의 불일치에 있음을 깨닫게 된다. `--use-sim-time` 옵션은 이 문제를 해결하고 결정론적인 테스트 환경을 구축하는 핵심 열쇠가 된다.19 또한, 수십, 수백 기가바이트에 달하는 데이터를 다루어야 하는 자율주행과 같은 분야에서는 파일 분할과 압축 기능이 없이는 효율적인 데이터 관리가 불가능하다.1

결론적으로, 이 기능들의 조합은 `rosbag2`가 단순한 '기록기'가 아님을 증명한다. `rosbag2`는 복잡한 로보틱스 시스템의 전체 개발 및 검증 수명주기에 깊숙이 통합되어 핵심적인 역할을 수행하는 정교한 '데이터 서브시스템(data subsystem)'으로 기능하도록 설계되었다.

------


`rosbag2`의 가장 혁신적인 특징 중 하나는 플러그인 기반 스토리지 아키텍처이다. 이는 데이터가 실제로 디스크에 저장되는 방식을 `rosbag2`의 핵심 로직과 분리하여, 사용자가 목적에 따라 다양한 저장소 백엔드를 선택할 수 있도록 한다. ROS 2 Humble에서는 SQLite3가 기본 스토리지로 사용되지만, 차세대 스토리지인 MCAP의 등장은 `rosbag2`의 성능과 활용성을 한 단계 끌어올렸다.


ROS 1의 `rosbag`은 `.bag`이라는 단일 파일 포맷에 모든 것이 고정되어 있었다. 반면, `rosbag2`는 스토리지 인터페이스(`storage_interfaces`)라는 추상화된 계층을 정의하고, 실제 구현은 별도의 플러그인 패키지에 위임한다.20 이 구조는 다음과 같은 장점을 가진다.

- **유연성**: 사용자는 애플리케이션의 요구사항(예: 쓰기 성능, 읽기 성능, 데이터 무결성, 실시간성)에 가장 적합한 스토리지 플러그인을 선택할 수 있다. 예를 들어, 고속 데이터 로깅에는 MCAP을, 기존 도구와의 호환성이 중요하다면 SQLite3를 사용할 수 있다.
- **확장성**: 새로운 파일 포맷이나 데이터베이스 기술이 등장하더라도, `rosbag2`의 코어 부분을 수정하지 않고 새로운 스토리지 플러그인을 개발하여 추가할 수 있다. 이는 `rosbag2`가 미래의 기술 변화에 쉽게 적응할 수 있게 해준다.3 LevelDB와 같은 다른 데이터베이스를 플러그인으로 도입하려는 제안도 있었다.21
- **분리된 개발**: 스토리지 플러그인은 독립적인 패키지로 개발 및 유지보수될 수 있어, 전체 `rosbag2` 프로젝트의 복잡성을 낮추고 개발을 가속화한다.


ROS 2 Humble 배포판을 설치하면 기본적으로 `rosbag2_storage_sqlite3` 플러그인이 함께 설치되며, 별도의 옵션 없이 `ros2 bag record`를 실행하면 이 플러그인이 사용된다.

- **특징**: SQLite3는 서버 없이 파일 하나로 전체 데이터베이스를 관리하는 경량 관계형 데이터베이스이다. `rosbag2`는 이 데이터베이스 파일(`.db3`)에 메시지 데이터를 저장하고, 별도의 `metadata.yaml` 파일에 메타데이터를 기록한다.5

- **성능과 안정성의 트레이드오프**: `rosbag2`는 SQLite3를 사용할 때, 기본적으로 쓰기 성능을 극대화하기 위해 다소 위험한 설정을 채택한다. 데이터베이스 저널링을 메모리에서 수행(`PRAGMA journal_mode=MEMORY`)하고, 디스크 동기화를 비활성화(`PRAGMA synchronous=OFF`)하는 것이다.2 이 설정은 빠른 쓰기 속도를 보장하지만, 녹화 중 시스템이 비정상적으로 종료될 경우 데이터베이스 트랜잭션이 완료되지 않아 

  `.db3` 파일 전체가 손상될 수 있는 심각한 위험을 내포하고 있다.

- **한계**:

  1. **성능 병목**: 대량의 메시지가 빠르게 유입될 경우, SQLite3의 트랜잭션 처리 오버헤드로 인해 쓰기 성능이 병목이 되어 메시지 유실로 이어질 수 있다.2
  2. **데이터 무결성 문제**: 앞서 언급한 기본 설정의 위험성 때문에 미션 크리티컬한 데이터를 안정적으로 로깅하기에는 부적합하다. 안정성을 높이는 `resilient` 프로파일을 사용할 수 있지만, 이는 쓰기 성능을 크게 저하시키는 대가를 치른다.2
  3. **이식성 부족**: SQLite3 기반의 bag 파일은 메시지 타입 정의(.msg)를 완전히 포함하지 않는다. 따라서 해당 bag 파일을 다른 환경에서 읽으려면, 녹화 당시 사용했던 모든 커스텀 메시지 패키지가 동일한 버전으로 설치되어 있어야 한다. 이는 데이터의 장기 보관이나 외부 협업 시 큰 걸림돌이 된다.23


이러한 SQLite3의 한계를 극복하기 위해 개발된 것이 바로 MCAP이다. MCAP은 로보틱스 데이터 로깅에 특화된 현대적인 컨테이너 파일 포맷으로, ROS 2 Iron Irwini 배포판부터 `rosbag2`의 기본 스토리지로 채택되었다.7 Humble 사용자도 `rosbag2_storage_mcap` 패키지를 설치하면 MCAP을 사용할 수 있다.

- **설계 철학**: MCAP은 고성능 쓰기, 효율적인 데이터 탐색, 데이터 무결성, 그리고 완벽한 이식성을 동시에 달성하는 것을 목표로 설계되었다.23
- **주요 장점**:
  - **단일 파일, 자기 완결적(Self-contained)**: 모든 메시지 데이터, 타임스탬프, 메타데이터뿐만 아니라, 녹화된 모든 토픽의 메시지 스키마(타입 정의)까지 단 하나의 `.mcap` 파일 안에 저장된다. 이 덕분에 ROS 환경이 전혀 없는 시스템에서도 MCAP 라이브러리만 있으면 파일의 내용을 완벽하게 해석하고 디코딩할 수 있다.23
  - **고성능 및 데이터 안정성**: 파일 끝에 데이터를 추가하기만 하는 Append-only 구조를 채택하여 쓰기 성능을 극대화한다. 또한, 녹화가 중간에 예기치 않게 중단되더라도 이미 기록된 부분은 손상되지 않아 데이터 안정성이 매우 높다.23
  - **효율적인 데이터 접근**: MCAP 파일은 내부적으로 데이터를 청크(Chunk) 단위로 관리하고, 각 메시지의 위치를 가리키는 인덱스(Message Index)를 포함할 수 있다. 덕분에 수백 기가바이트 크기의 파일이라도 전체를 스캔할 필요 없이 특정 시간대의 데이터에 매우 빠르게 접근(seek)할 수 있다.23
  - **네이티브 압축**: 청크 단위로 LZ4나 Zstd 같은 효율적인 압축 알고리즘을 적용할 수 있다. 중요한 점은, 압축된 상태에서도 인덱스를 활용한 빠른 데이터 탐색이 가능하다는 것이다. 이는 저장 공간 효율성과 데이터 분석 효율성을 동시에 만족시키는 핵심적인 특징이다.17


개발자가 자신의 애플리케이션 요구사항에 맞춰 최적의 스토리지를 선택할 수 있도록, 두 스토리지 백엔드의 주요 특징을 비교하면 다음과 같다.

| 기능 (Feature)          | SQLite3 (Humble 기본)                             | MCAP (Iron+ 기본)                                            | 핵심 차이 및 고려사항                                        |
| ----------------------- | ------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **파일 구조**           | 디렉터리 (`metadata.yaml` + `.db3` 파일)          | 단일 `.mcap` 파일                                            | MCAP이 관리와 이동이 훨씬 간편함.8                           |
| **기본 쓰기 성능**      | 높지만, 데이터 손상 위험 존재 (Default profile)   | 매우 높음                                                    | MCAP은 성능과 안정성을 동시에 제공.2                         |
| **데이터 무결성**       | 기본 설정에서 취약. 비정상 종료 시 파일 손상 가능 | 강함. Append-only 구조로 부분 파일 복구 용이                 | 장시간 미션 크리티컬 데이터 로깅에는 MCAP이 절대적으로 유리.17 |
| **데이터 탐색(Seek)**   | 비효율적. DB 쿼리에 의존                          | 매우 효율적. Chunk 및 Message Index 덕분                     | 대용량 bag 파일에서 특정 이벤트를 분석할 때 MCAP의 성능 우위가 두드러짐.23 |
| **자기 완결성(Schema)** | 불완전. 메시지 타입 정의(.msg)가 외부에 필요      | 완전. 파일 내에 스키마 포함                                  | ROS 환경이 없는 서버에서 분석하거나 장기 보관 시 MCAP이 필수적.23 |
| **압축**                | 프레임워크 레벨 압축만 지원 (비효율적)            | 네이티브 Chunk 압축 (LZ4, Zstd) 지원                         | MCAP은 압축 상태에서도 빠른 탐색이 가능하나, SQLite3는 파일 전체를 압축 해제해야 함.1 |
| **파일 크기 (압축 시)** | Zstd 압축 시 효율적                               | Zstd(slow) 옵션 사용 시 SQLite3+Zstd와 비슷하거나 더 우수    | 압축률과 CPU 사용량 간의 트레이드오프 존재.17                |
| **툴 생태계 지원**      | `rqt_bag` 등 기존 툴에서 잘 지원됨                | `rqt_bag` Humble 버전에선 미지원 이슈가 보고됨.17 Foxglove 등 최신 툴에서 완벽 지원.30 | Humble 환경에서는 MCAP 사용 시 `rqt_bag` 대신 다른 시각화 도구를 고려해야 할 수 있다. |


MCAP의 우수한 성능과 기능은 그 정교한 내부 구조에서 비롯된다. 파일은 고유한 매직 바이트(Magic Bytes)로 시작하고 끝나며, 그 사이는 일련의 레코드(Record)들로 채워진다.28 핵심 구성 요소는 다음과 같다.

- **Chunk (op=0x06)**: 메시지, 스키마, 채널 정보 등 여러 레코드들을 하나의 묶음으로 관리하는 컨테이너이다. Chunk는 MCAP에서 압축과 인덱싱이 이루어지는 기본 단위이다. 각 Chunk 레코드는 자신이 담고 있는 메시지들의 시작 시간(`message_start_time`)과 종료 시간(`message_end_time`) 정보를 가지고 있어, 특정 시간 범위에 해당하지 않는 Chunk 전체를 건너뛰는 시간 기반의 빠른 필터링을 가능하게 한다.28
- **Message Index (op=0x07)**: 각 Chunk 내에 저장된 메시지들을 효율적으로 찾기 위한 인덱스 정보이다. 이 레코드는 특정 채널(`channel_id`)에 속한 각 메시지의 타임스탬프와, 압축 해제된 Chunk 데이터 내에서의 상대적 위치(offset)를 매핑하는 목록을 담고 있다. 이 인덱스 덕분에 특정 메시지를 찾기 위해 Chunk 전체를 압축 해제하고 순차적으로 스캔할 필요 없이, 원하는 메시지의 위치로 직접 점프할 수 있다.28
- **Summary Section**: 파일의 끝부분에 위치하는 선택적 섹션으로, 파일 전체에 대한 요약 정보를 담고 있다. 여기에는 파일에 기록된 모든 채널(토픽)의 목록, 전체 메시지 수와 같은 통계 정보, 그리고 모든 Chunk Index의 위치 정보 등이 포함될 수 있다. 애플리케이션이 MCAP 파일을 열 때, 파일 전체를 읽는 대신 이 Summary Section만 먼저 읽으면 전체 파일의 구조를 매우 빠르게 파악할 수 있다.28

MCAP의 아키텍처는 '쓰기 최적화'와 '읽기 최적화'라는, 종종 상충될 수 있는 두 가지 목표를 동시에 달성하기 위해 매우 영리하게 설계되었다. 실시간 데이터 로깅 중에는 Append-only 방식으로 파일 끝에 데이터를 빠르게 추가하여 쓰기 성능을 극대화한다. 그리고 파일의 끝이나 후처리 과정을 통해 추가되는 인덱스와 요약 정보는, 나중에 데이터를 분석할 때 필요한 부분만 효율적으로 읽어낼 수 있도록 보장한다. 이는 실시간 로깅과 오프라인 분석이라는 로보틱스 개발의 두 가지 핵심 워크플로우를 모두 깊이 있게 고려한 결과물이라 할 수 있다.

------


`rosbag2`의 진정한 강력함은 CLI를 넘어 프로그래밍 방식으로 데이터 로깅과 처리를 제어할 때 드러난다. `rosbag2`는 Python(`rosbag2_py`)과 C++(`rosbag2_cpp`)를 위한 API를 제공하여, 개발자가 자신의 ROS 2 노드 내에서 직접 bag 파일을 생성하고 읽을 수 있게 해준다. 이를 통해 단순한 데이터 기록을 넘어, 조건부 로깅, 실시간 데이터 변환 및 분석 등 훨씬 더 정교하고 지능적인 데이터 파이프라인을 구축할 수 있다.


`rosbag2_py`는 Python 환경에서 `rosbag2`의 기능을 활용할 수 있게 해주는 라이브러리이다. CLI로는 불가능한, 사용자 정의 로직과 결합된 유연한 데이터 처리를 가능하게 한다.32


`SequentialWriter`는 메시지를 수신된 순서대로 bag 파일에 쓰는 클래스이다. 사용 과정은 다음과 같은 단계로 이루어진다.

1. **객체 생성 및 설정**: `SequentialWriter` 객체를 생성하고, `StorageOptions`와 `ConverterOptions`를 정의한다.

   - `StorageOptions`: `uri` (파일 경로 및 이름)와 `storage_id` ('sqlite3' 또는 'mcap')를 지정하는 것이 가장 중요하다. `storage_id`에 따라 생성되는 파일의 포맷이 결정된다.
   - `ConverterOptions`: 입력 메시지와 저장될 메시지의 직렬화 포맷을 지정한다. 특별한 경우가 아니면 입력과 출력을 모두 ROS 2의 표준 직렬화 포맷인 'cdr'로 설정한다.

   다음은 SQLite3와 MCAP 포맷으로 각각 쓰기 위한 `StorageOptions` 설정 예시이다.32

   ```Python
   import rosbag2_py
   
   # SQLite3 (Humble 기본)로 쓰기 위한 설정
   # 'my_bag_sqlite3'라는 디렉터리가 생성됨
   storage_options_sqlite3 = rosbag2_py.StorageOptions(
       uri='my_bag_sqlite3', 
       storage_id='sqlite3'
   )
   
   # MCAP으로 쓰기 위한 설정
   # 'my_bag.mcap'라는 단일 파일이 생성됨
   storage_options_mcap = rosbag2_py.StorageOptions(
       uri='my_bag.mcap', 
       storage_id='mcap'
   )
   
   converter_options = rosbag2_py.ConverterOptions(
       input_serialization_format='cdr',
       output_serialization_format='cdr'
   )
   ```

2. **파일 열기**: `writer.open(storage_options, converter_options)`를 호출하여 bag 파일을 열고 쓰기 준비를 한다.

3. **토픽 정보 등록**: `writer.create_topic()`을 사용하여 저장할 토픽의 메타데이터(토픽 이름, 메시지 타입, 직렬화 포맷)를 bag 파일에 등록한다. 이 과정은 bag 파일이 해당 토픽의 데이터를 올바르게 해석하는 데 필수적이다.

   ```Python
   topic_metadata = rosbag2_py.TopicMetadata(
       name='/chatter',
       type='std_msgs/msg/String',
       serialization_format='cdr'
   )
   writer.create_topic(topic_metadata)
   ```

4. **메시지 직렬화 및 쓰기**: ROS 2 노드에서 수신한 메시지 객체는 `rclpy.serialization.serialize_message()`를 통해 직렬화된 바이트 스트림으로 변환해야 한다. 그 후 `writer.write()` 메서드를 호출하여 직렬화된 데이터와 타임스탬프를 bag 파일에 쓴다.32

   ```Python
   from rclpy.serialization import serialize_message
   from std_msgs.msg import String
   
   #... writer 객체가 생성되고 열려있는 상태...
   
   msg = String()
   msg.data = 'Hello, rosbag2!'
   timestamp = node.get_clock().now().nanoseconds
   
   writer.write(
       '/chatter',
       serialize_message(msg),
       timestamp
   )
   ```


`SequentialReader`는 bag 파일에 저장된 메시지를 순차적으로 읽어오는 클래스이다.

1. **객체 생성 및 파일 열기**: `SequentialReader` 객체를 생성하고, `StorageOptions`와 `ConverterOptions`를 설정한 뒤 `reader.open()`을 호출하여 bag 파일을 연다. 읽기 시에도 `storage_id`를 명시해주는 것이 좋다.

2. **메시지 순회 및 읽기**: `reader.has_next()` 메서드로 읽을 메시지가 남아있는지 확인하고, `reader.read_next()`를 호출하여 `(topic_name, serialized_data, timestamp)` 형태의 튜플을 읽어온다.33

3. **메시지 역직렬화**: `read_next()`로 읽어온 데이터는 직렬화된 상태이므로, 이를 다시 파이썬 객체로 변환하는 역직렬화 과정이 필요하다.

   - 먼저 `reader.get_all_topics_and_types()`를 통해 bag에 저장된 토픽과 타입 정보를 얻는다.
   - 읽어온 `topic_name`에 해당하는 메시지 타입을 `rosidl_runtime_py.utilities.get_message()`를 이용해 동적으로 가져온다.
   - 마지막으로 `rclpy.serialization.deserialize_message()`를 사용하여 직렬화된 데이터(`serialized_data`)를 해당 타입의 파이썬 객체로 변환한다.33

   ```Python
   from rclpy.serialization import deserialize_message
   from rosidl_runtime_py.utilities import get_message
   
   #... reader 객체가 생성되고 열려있는 상태...
   
   # 토픽 이름과 메시지 타입 클래스를 매핑하는 딕셔너리 생성
   topic_types = reader.get_all_topics_and_types()
   type_map = {topic.name: get_message(topic.type) for topic in topic_types}
   
   while reader.has_next():
       (topic, data, timestamp) = reader.read_next()
       msg_type = type_map[topic]
       msg = deserialize_message(data, msg_type)
       print(f"[{timestamp}] {topic}: {msg.data}")
   ```


Python API가 편리함과 유연성을 제공한다면, C++ API는 최고의 성능이 요구되는 애플리케이션을 위한 선택지이다. `rosbag2_cpp`와 `rosbag2_transport` 라이브러리를 통해 C++ 노드에서 직접 bag 파일을 제어할 수 있다.35

C++ API는 `rosbag2_cpp::Writer`와 `rosbag2_cpp::Reader` 클래스를 중심으로 구성되며, Python API와 개념적으로 매우 유사한 인터페이스(e.g., `open`, `write`, `read_next`, `has_next`)를 제공한다.36 가장 큰 차이점은 C++ API가 `rclcpp::SerializedMessage`라는 객체를 직접 다루어, 직렬화 및 역직렬화 과정에서 발생하는 메모리 할당과 데이터 복사를 더 세밀하게 제어할 수 있다는 점이다. 이는 고주파, 대용량 센서 데이터를 처리할 때 발생할 수 있는 오버헤드를 최소화하는 데 결정적인 역할을 한다.


최고 수준의 녹화 성능을 달성하기 위한 가장 효과적인 방법은 `rosbag2`의 recorder를 다른 노드들과 동일한 프로세스 내에서 컴포넌트(component)로 실행하는 것이다. 이를 'Composable Node' 아키텍처라고 한다.

`rosbag2_transport::Recorder` 컴포넌트를 데이터 소스 노드(e.g., 카메라 드라이버)와 함께 컴포넌트 컨테이너에 로드하면, 두 컴포넌트 간의 메시지 전달은 ROS 2의 Intra-Process Communication(IPC) 메커니즘을 통해 이루어진다.1 IPC는 DDS 네트워크 스택을 완전히 우회하고, 직렬화/역직렬화 과정 없이 메시지 객체의 포인터(소유권)만을 전달하여 메모리를 직접 공유한다. 이 방식은 데이터 복사와 통신 오버헤드를 극단적으로 줄여주기 때문에, 시스템 리소스가 제한적이거나 데이터 처리량이 매우 높은 환경에서 메시지 유실 없이 안정적으로 데이터를 녹화하는 데 매우 효과적이다. ROS 2 Jazzy부터는 이 기능이 표준 `rosbag2` 프레임워크에 통합되었다.37

`rosbag2` API의 존재는 `rosbag2`의 위상을 단순한 '도구(tool)'에서 로보틱스 개발을 위한 '프레임워크(framework)'로 격상시킨다. 개발자는 더 이상 데이터 로깅을 시스템 개발과 분리된 후처리 작업으로 취급할 필요가 없다. 대신, 데이터 로깅을 애플리케이션의 핵심 로직의 일부로 통합할 수 있다. 예를 들어, 개발자는 CLI로 모든 데이터를 무차별적으로 녹화하고 나중에 필요한 부분을 찾는 대신, `rosbag2_py` API를 사용하여 특정 조건이 충족될 때만 데이터를 bag 파일에 쓰는 '스마트 레코더'를 구현할 수 있다.32 이는 이상 상황이 감지되었을 때만 주변 데이터를 저장하는 '블랙박스' 기능이나, 특정 실험 조건에서만 데이터를 수집하는 자동화된 워크플로우를 가능하게 한다.

더 나아가, Autoware 프로젝트에서 제공하는 `rosbag2_anonymizer`와 같은 도구는 이러한 가능성을 잘 보여준다.38 이 도구는 bag 파일을 프로그래밍 방식으로 읽고, 이미지 데이터에서 얼굴이나 번호판과 같은 개인정보를 감지하여 블러 처리한 뒤, 그 결과를 새로운 bag 파일로 쓰는 복잡한 데이터 변환 파이프라인을 구축한다. 최고 성능이 요구되는 시나리오에서는 C++ API와 Composable Node 아키텍처를 결합하여 시스템 리소스 사용을 최소화하면서도 미션 크리티컬한 데이터를 단 하나도 유실하지 않고 기록할 수 있다.37 이러한 활용 사례들은 `rosbag2`가 ROS 2가 지향하는 데이터 중심 아키텍처(data-centric architecture)의 핵심적인 구현체이며, 정교하고 지능적인 데이터 기반 로봇 애플리케이션을 구축하는 데 필수적인 기반 기술임을 증명한다.

------


ROS 2로의 전환을 고려하는 많은 개발팀에게 있어, 기존에 축적된 방대한 양의 ROS 1 `rosbag` 데이터 자산을 어떻게 활용할 것인가는 중요한 과제이다. `rosbag2`는 ROS 1 `rosbag`의 후속 기술이지만, 단순히 기능을 개선한 것이 아니라 아키텍처 자체가 근본적으로 다르기 때문에 직접적인 호환성은 없다. 이 장에서는 두 시스템의 구조적 차이를 분석하고, ROS 1 bag 파일을 ROS 2 환경에서 활용하기 위한 다양한 전략을 살펴본다.


두 `rosbag` 시스템 간의 가장 큰 차이점은 데이터 저장 방식에 대한 설계 철학에 있다.

- **ROS 1 `rosbag`**: ROS 1의 `.bag` 파일은 매우 잘 설계된 모놀리식(monolithic) 포맷이다. 하나의 `.bag` 파일 안에는 메시지 데이터뿐만 아니라, 해당 메시지의 타입 정의(schema)와 연결 정보까지 모두 포함되어 있었다.4 이러한 '자기 완결적(self-contained)' 특성 덕분에 `.bag` 파일 하나만 있으면 ROS 1이 설치된 어떤 시스템에서든 내용을 쉽게 확인할 수 있어 이식성이 매우 높았다. 하지만 이는 포맷 자체가 고정되어 있어 새로운 압축 방식이나 저장 기술을 도입하기 어려운 경직성을 내포하고 있었다.

- **ROS 2 `rosbag2`**: 반면, `rosbag2`는 앞서 설명했듯이 스토리지, 직렬화, 트랜스포트 계층을 모두 플러그인 형태로 분리한 유연한 아키텍처를 채택했다.3 이는 뛰어난 확장성을 제공하지만, 초기 버전(특히 SQLite3 백엔드)에서는 메시지 타입 정의가 bag 파일 외부에 존재해야 했기 때문에 ROS 1만큼 자기 완결적이지 못했다. 이 문제는 차세대 스토리지인 MCAP이 등장하면서 해결되었다. MCAP은 메시지 스키마를 파일 내에 포함시킴으로써 ROS 1 

  `rosbag`의 높은 이식성이라는 장점을 계승하고 `rosbag2`의 구조적 유연성을 결합한, 두 세계의 장점을 모두 취한 형태로 발전했다.


ROS 1 `.bag` 파일을 ROS 2 시스템에서 사용하기 위해, 사용자의 목적과 환경에 따라 선택할 수 있는 세 가지 주요 전략이 존재한다.


가장 간단하고 즉각적인 방법은 `ros1_bridge`를 활용하는 것이다. 이 방식은 두 개의 터미널을 사용한다. 하나의 터미널에서는 ROS 1 환경을 활성화하고 `rosbag play` 명령으로 `.bag` 파일을 재생한다. 다른 터미널에서는 ROS 2 환경을 활성화하고 `ros1_bridge`를 실행한다. `ros1_bridge`는 ROS 1과 ROS 2 네트워크 사이에서 다리 역할을 하며, ROS 1 토픽으로 발행되는 메시지들을 실시간으로 ROS 2 토픽으로 변환하여 전달한다.4

- **장점**: 설정이 비교적 간단하며, 데이터를 변환하는 과정 없이 즉시 ROS 2 노드에서 데이터를 구독하여 확인할 수 있다.
- **단점**: 이 방법을 사용하려면 하나의 시스템에 ROS 1과 ROS 2 환경이 모두 올바르게 설치되어 있어야 한다. 또한, 브릿지 자체에서 발생하는 통신 오버헤드와 잠재적인 성능 저하를 감수해야 한다.


이 방법은 `rosbag2`가 ROS 1 `.bag` 파일을 직접 읽고 재생할 수 있도록 해주는 특수한 스토리지 플러그인을 설치하는 것이다. `rosbag2_bag_v2_plugins` 패키지를 설치하면, `ros2 bag` 명령어에 `-s rosbag_v2`라는 스토리지 ID 옵션을 추가하여 ROS 1 bag 파일을 마치 ROS 2 bag처럼 다룰 수 있다.41

```Bash
ros2 bag play -s rosbag_v2 my_ros1_data.bag
```

- **장점**: `ros2 bag play`, `ros2 bag info` 등 ROS 2의 네이티브 CLI 도구를 그대로 사용하여 ROS 1 데이터를 다룰 수 있어 워크플로우 통합에 유리하다.
- **단점**: 이 플러그인은 내부적으로 ROS 1 메시지를 ROS 2 메시지로 변환하기 위해 `ros1_bridge`의 라이브러리에 의존한다. 따라서 이 플러그인을 설치하면 `ros1_bridge`와 그에 필요한 ROS 1 패키지들이 함께 설치되어, 시스템의 의존성 관계가 복잡해질 수 있다. 특히 커스텀 메시지를 사용하는 경우, ROS 1과 ROS 2 버전의 메시지 패키지가 모두 필요하여 관리가 어려워질 수 있다.40


가장 근본적이고 장기적인 해결책은 ROS 1 `.bag` 파일을 ROS 2 `rosbag2` 포맷(예: MCAP)으로 완전히 변환하는 것이다. `rosbags`와 같은 서드파티 라이브러리는 ROS 1과 ROS 2에 대한 의존성 없이 bag 파일을 변환할 수 있는 강력한 `rosbags-convert` CLI 도구를 제공한다.42

```Bash
# ROS 1.bag 파일을 ROS 2 디렉터리 포맷으로 변환
rosbags-convert my_ros1_data.bag

# ROS 2 디렉터리 포맷을 ROS 1.bag 파일로 변환
rosbags-convert my_ros2_data_dir --dst my_converted_data.bag
```

- **장점**: 일단 변환이 완료되면, 해당 데이터는 더 이상 ROS 1 환경에 대한 어떠한 의존성도 갖지 않는 완전한 ROS 2 데이터 자산이 된다. 이는 데이터의 장기 보관, 이식성, 그리고 미래 호환성 측면에서 가장 이상적인 방법이다.
- **단점**: 대용량 bag 파일을 변환하는 데는 상당한 시간과 컴퓨팅 자원이 소요될 수 있다.

ROS 1 `rosbag`과의 호환성 문제는 `rosbag2`가 단순히 새로운 버전이 아니라 근본적으로 다른 아키텍처를 채택했기 때문에 발생하는 필연적인 과제이다. 위에서 제시된 세 가지 해결책은 각각 다른 트레이드오프(편의성, 시스템 의존성, 데이터 이식성)를 가지고 있으며, 개발자의 구체적인 목적에 따라 전략적으로 선택되어야 한다.

예를 들어, ROS 2로 전환 중인 팀이 방대한 양의 ROS 1 bag 데이터 자산을 보유하고 있다고 가정해보자.4 가장 빠르게 기존 데이터를 활용하여 새로운 ROS 2 알고리즘을 테스트하고 싶다면, `ros1_bridge`를 이용한 실시간 브릿징이 가장 효율적인 '임시방편'이 될 수 있다.4 만약 ROS 2의 개발 워크플로우에 좀 더 깊이 통합하여 `ros2 launch` 등과 함께 사용하고 싶다면, `rosbag_v2` 플러그인을 설치하는 것을 고려할 수 있다.41 하지만 이는 시스템을 ROS 1과 ROS 2가 혼재된 "하이브리드" 상태로 만들어 장기적인 의존성 관리를 복잡하게 만들 수 있다.43

궁극적으로, 과거의 데이터 자산을 미래에도 온전히, 그리고 안정적으로 활용하기 위한 가장 현명한 '투자'는 데이터를 MCAP과 같은 ROS 2 네이티브 포맷으로 변환하여 보관하는 것이다. `rosbags-convert`와 같은 도구는 이러한 '데이터 마이그레이션'을 수행하는 핵심적인 역할을 한다.42 따라서 이 세 가지 방법은 단순한 옵션의 나열이 아니라, ROS 2로의 전환 과정에서 개발팀이 겪게 되는 '과도기적 분석', '통합 단계', 그리고 '완전한 마이그레이션'이라는 각기 다른 단계를 지원하는 전략적 도구들로 이해하는 것이 타당하다.

------


`rosbag2`는 강력한 도구이지만, 실제 개발 현장에서 사용하다 보면 다양한 문제에 직면하게 된다. 특히 대용량, 고주파 데이터를 다룰 때 발생하는 메시지 유실, 성능 저하, 그리고 비결정론적 재현과 같은 문제들은 많은 개발자들을 괴롭힌다. 이 장에서는 포럼과 GitHub 이슈 트래커 등에서 논의된 일반적인 문제들의 원인을 분석하고, 이를 해결하기 위한 구체적이고 실용적인 방안을 제시한다.


다음 표는 `rosbag2` 사용 시 흔히 겪는 문제들과 그 원인, 그리고 해결 방안을 체계적으로 정리한 것이다. 이는 개발자가 문제 상황에 직면했을 때 디버깅에 소요되는 시간을 획기적으로 줄여주는 실용적인 지식 베이스 역할을 한다.

| 문제 현상 (Symptom)                                 | 가능한 원인 (Root Cause)                                     | 해결 방안 및 명령어 예시 (Solution/Workaround)               | 관련 자료 |
| --------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | --------- |
| **메시지 유실 (Message Drops)**                     | 쓰기 속도 < 데이터 수신 속도, QoS 설정 미흡, 시스템 리소스 부족 | 1. QoS 프로파일을 `reliable` 및 `keep_all`로 설정. 2. 고성능 `mcap` 스토리지 사용 (`-s mcap`). 3. `--storage-config-file`로 스토리지 성능 튜닝(e.g., 청크 크기 증가). 4. Composable Recorder로 IPC 활용. | 2         |
| **`Ctrl-C` 입력 시 녹화 종료 안 됨 (Hang on Exit)** | DDS 노드 종료 시그널 처리 문제, 버퍼에 남은 데이터를 디스크에 쓰는(flush) 과정에서 대기 | 1. `Ctrl-C` 입력 후 `Enter` 키를 한 번 더 눌러본다. 2. 이 문제는 구버전의 알려진 이슈로, 최신 ROS 2 배포판으로 업데이트하면 해결될 수 있다. | 15        |
| **"file is not a database" 오류**                   | MCAP 포맷의 bag 파일을 `rqt_bag`과 같이 SQLite3 포맷만 지원하는 구버전 도구로 열려고 할 때 발생 | 1. `ros2 bag info/play` 등 표준 CLI를 사용한다. 2. Foxglove Studio와 같이 MCAP을 완벽하게 지원하는 최신 시각화 도구를 사용한다. | 17        |
| **Docker 환경에서 토픽 미발견**                     | Docker의 기본 네트워크 설정이 DDS의 노드 자동 발견(discovery) 메커니즘을 제한함 | 1. Docker 컨테이너 실행 시 `--ipc=host` 옵션을 추가하여 호스트의 IPC 네임스페이스를 공유한다. 2. DDS 설정에서 Super Client 프로파일을 사용하는 것을 고려한다. | 45        |
| **`rosbag_v2` 플러그인 설치 시 의존성 문제**        | 해당 플러그인이 의존하는 `ros1_bridge` 및 관련 ROS 1 패키지가 시스템에 설치되어 있지 않음 | 1. `ros1_bridge` 설치 가이드에 따라 필요한 ROS 1 패키지들을 먼저 설치한다. 2. ROS 1 bag 호환성이 필요 없다면, `rosbag2_bag_v2_plugins` 패키지를 제외하고 `rosbag2` 관련 패키지만 설치한다. | 43        |
| **`use_sim_time` 불일치로 인한 오작동**             | 시스템 내 일부 노드는 `use_sim_time:=True`로 실행되고, 다른 노드는 `False`(기본값)로 실행되어 시간 기준이 혼재됨 | `launch` 파일에서 시스템을 구성하는 모든 노드의 `use_sim_time` 파라미터를 `True`로 일관되게 설정하고, `ros2 bag play` 실행 시에도 `--use-sim-time` 옵션을 반드시 추가한다. | 18        |


메시지 유실은 `rosbag2` 사용 시 가장 심각하고 흔하게 발생하는 문제 중 하나이다. 특히 초당 수백 메가바이트에 달하는 데이터를 생성하는 고해상도 카메라나 라이다 센서를 사용할 때 이 문제는 두드러진다.

- **근본 원인**: 메시지 유실의 근본 원인은 데이터 파이프라인의 병목 현상에 있다. 즉, `rosbag2_transport` 노드의 내부 메시지 큐에 데이터가 쌓이는 속도가 스토리지 플러그인이 데이터를 디스크에 쓰는 속도를 초과할 때, 큐가 가득 차면서 새로운 메시지를 버리게 되는 것이다.2

이 문제를 해결하기 위해서는 다음과 같은 다단계 전략을 적용해야 한다.


가장 먼저 확인하고 적용해야 할 조치는 Quality of Service(QoS) 설정이다. QoS는 DDS 미들웨어 레벨에서 메시지 전달의 신뢰성을 보장하는 핵심 메커니즘이다.

- 데이터를 발행하는 Publisher 노드와 이를 수신하는 `rosbag2` Recorder 노드 양쪽 모두에서 QoS 설정을 일치시켜야 한다.
- 신뢰성이 중요한 데이터의 경우, `Reliability` 정책을 `RELIABLE`로, `History` 정책을 `KEEP_ALL`로 설정해야 한다. `RELIABLE`은 메시지 전달을 보장하도록 재전송을 시도하며, `KEEP_ALL`은 내부 큐가 가득 차지 않는 한 모든 메시지를 보관하도록 한다.
- `ros2 bag record` 명령에서는 `--qos-profile-overrides-path` 옵션을 사용하여, 특정 토픽에 대해 강제로 적용할 QoS 설정을 YAML 파일로 지정할 수 있다. 이는 Publisher의 QoS 설정을 변경할 수 없을 때 매우 유용하다.14

```YAML
# qos_overrides.yaml
/my_lidar/points:
  reliability: reliable
  history: keep_all
  depth: 100 # keep_all 사용 시 큐 크기를 충분히 크게 설정
```

```Bash
ros2 bag record /my_lidar/points --qos-profile-overrides-path qos_overrides.yaml
```


QoS를 `RELIABLE`로 설정했음에도 메시지 유실이 발생한다면, 이는 DDS 레벨이 아닌 `rosbag2` 내부, 특히 스토리지 쓰기 과정에서 병목이 발생하고 있다는 신호이다.

- **스토리지 선택**: 가장 효과적인 방법은 Humble의 기본 스토리지인 `sqlite3` 대신, 쓰기 성능이 월등히 뛰어난 `mcap`으로 변경하는 것이다. `-s mcap` 옵션을 추가하는 것만으로도 상당한 성능 향상을 기대할 수 있다.2
- **스토리지 설정 (`--storage-config-file`)**: MCAP 스토리지의 내부 동작 파라미터를 YAML 파일을 통해 직접 제어하여 성능을 더욱 미세하게 튜닝할 수 있다. 예를 들어, `chunk_size`를 늘리면 디스크에 접근하는 I/O 횟수가 줄어들어 쓰기 성능이 향상될 수 있다. `compressionLevel`을 속도 우선(`Fastest` 또는 `Fast`)으로 설정하는 것도 도움이 된다.26

```YAML
# mcap_performance_config.yaml
# 이 파일은 ros2 bag record --storage-config-file 옵션으로 전달
default: # MCAP writer는 'default' 프로파일을 찾음
  chunk_size: 4194304        # 4MB. 청크 크기를 늘려 I/O 횟수 감소
  compression: "Zstd"
  compressionLevel: "Fast"  # 압축률보다 속도 우선
  forceCompression: false
```


리소스가 극도로 제한된 임베디드 보드 등에서 최고의 쓰기 성능을 짜내야 할 때 `fastwrite` 프로파일을 사용할 수 있다.

```Bash
ros2 bag record -s mcap --storage-preset-profile fastwrite --all
```

이 프로파일은 MCAP의 주요 기능인 Chunking과 데이터 무결성 검사를 위한 CRC 계산을 비활성화하여 쓰기 오버헤드를 최소화한다.26

- **주의사항**: 이 프로파일로 생성된 bag 파일은 인덱스가 없기 때문에, 나중에 데이터를 읽을 때 특정 시간으로의 탐색(seek) 성능이 크게 저하된다. 따라서 장기 보관용 데이터나 정밀한 후처리가 필요한 데이터보다는, 데이터 유실을 감수할 수 없는 상황에서의 임시 고속 로깅에 적합하다.


데이터를 성공적으로 녹화했더라도, 이를 재현하는 과정에서 문제가 발생할 수 있다. `ros2 bag play`는 단순히 녹화된 메시지들을 원래의 시간 간격에 맞춰 발행하는 '오픈 루프(open-loop)' 시스템이다. 즉, 메시지를 구독하는 노드들이 이전 메시지를 모두 처리했는지 기다려주지 않고 다음 메시지를 계속해서 발행한다. 이로 인해 처리 능력이 부족한 시스템에서는 재생된 메시지를 놓치는 문제가 발생할 수 있다.19

신뢰성 있는 재현 환경을 구축하기 위한 핵심 전략은 다음과 같다.

1. **시간축 동기화**: 가장 중요한 것은 시스템 전체의 시간 기준을 통일하는 것이다. `launch` 파일에서 테스트에 관련된 모든 노드를 실행할 때 `use_sim_time` 파라미터를 `true`로 설정하고, `ros2 bag play`를 실행할 때도 반드시 `--use-sim-time` 옵션을 추가해야 한다. 이렇게 하면 모든 컴포넌트가 bag player가 발행하는 `/clock` 토픽을 기준으로 동작하여 시간축이 완벽하게 동기화된다.19
2. **시작 시점 동기화**: ROS 노드들이 초기화되는 데는 시간이 걸린다. `ros2 bag play`가 노드들이 완전히 준비되기 전에 메시지 발행을 시작하면, 초기 메시지들을 유실할 수 있다. `ros2 launch` 시스템의 이벤트 핸들러(Event Handlers)와 같은 메커니즘을 사용하여, 시스템의 모든 노드가 활성화된 것을 확인한 후에 `ros2 bag play` 프로세스를 시작하도록 실행 순서를 제어하는 것이 좋다.19
3. **정적 데이터 처리**: `/tf_static`과 같이 한 번만 발행되고 소멸하지 않는 `transient local` QoS를 사용하는 토픽은 bag 재생 시 불안정하게 동작하거나 유실될 수 있다. 이러한 정적 데이터는 bag에서 재생하는 것에 의존하기보다, `launch` 파일에서 별도의 `robot_state_publisher` 노드를 실행하여 직접 발행하는 것이 더 안정적이고 신뢰성 있는 방법일 수 있다.19

`rosbag2`의 성능 문제는 단일 컴포넌트의 결함이 아닌, '시스템 전체의 파이프라인 문제'로 접근해야 한다. 사용자가 메시지 유실을 처음 경험했을 때 14, 가장 먼저 `rosbag2` 자체의 버그나 성능을 의심하기 쉽다. 하지만 문제의 원인을 파고들다 보면, QoS 설정 14이 해결책으로 제시되는 것을 통해 이것이 DDS 미들웨어 계층의 문제일 수 있음을 알게 된다. 즉, 데이터가 `rosbag2` 노드에 도달하기도 전에 네트워크 레벨에서 유실될 수 있다는 것이다. QoS를 수정한 후에도 문제가 지속된다면, 이는 `rosbag2`의 내부 큐에서 병목이 발생하고 있다는 의미이며, 이는 스토리지의 쓰기 성능 문제 2로 귀결된다. 이 문제는 MCAP 스토리지 사용이나 스토리지 파라미터 튜닝을 통해 해결해야 한다. 마지막으로, 녹화는 성공했지만 재현 결과가 매번 달라진다면, 이는 ROS의 시간 관리 문제 19이며, `--use-sim-time`을 통해 해결할 수 있다.

결론적으로, `rosbag2`의 안정적인 운영은 `rosbag2`라는 단일 프로그램을 넘어, ROS 2 시스템을 구성하는 세 가지 핵심 메커니즘, 즉 통신(DDS), 데이터 저장(Storage), 그리고 시간 동기화(Time)에 대한 깊은 이해를 요구한다.

------


`rosbag2`는 ROS 1 `rosbag`의 단순한 계승자가 아니다. 이는 모듈성, 성능, 그리고 유연성에 초점을 맞춰 완전히 새롭게 설계된 데이터 관리 프레임워크이다. ROS 2 Humble 환경에서 `rosbag2`의 잠재력을 최대로 이끌어내기 위해서는 CLI의 기본 사용법을 숙지하는 것을 넘어, 파일 분할 및 압축과 같은 고급 기능, SQLite3와 MCAP으로 대표되는 스토리지 아키텍처의 차이, 그리고 `rosbag2_py`와 C++로 제공되는 프로그래밍 API를 종합적으로 이해하고 활용할 수 있어야 한다.

이 보고서에서 심층적으로 분석한 내용을 바탕으로, `rosbag2`를 효과적으로 활용하기 위한 핵심 권장사항을 다음과 같이 요약한다.

- **새로운 프로젝트에는 MCAP을 사용하라**: ROS 2 Humble의 기본 스토리지 백엔드는 SQLite3이지만, 성능, 데이터 안정성, 그리고 파일 이식성 등 거의 모든 면에서 MCAP이 월등한 장점을 보인다. `-s mcap` 옵션을 사용하여 MCAP 포맷으로 녹화하는 것을 표준으로 삼아야 한다. 이는 특히 데이터의 장기 보관이나 외부 협업, 그리고 고성능 로깅이 요구되는 환경에서 더욱 중요하다.2
- **QoS는 항상 명시적으로 관리하라**: 메시지 유실을 방지하기 위한 가장 기본적인 첫 번째 방어선은 QoS 설정이다. 데이터의 신뢰성이 중요하다면, Publisher와 Recorder 양쪽 모두에서 `Reliability: RELIABLE` 정책을 사용하는 것을 주저하지 말아야 한다. 특히 `ros2 bag record`의 `--qos-profile-overrides-path` 옵션을 적극적으로 활용하여, 녹화 대상 토픽에 대한 QoS 설정을 명시적으로 제어하는 습관을 들이는 것이 좋다.14
- **목적에 맞는 성능 프로파일을 선택하라**: 모든 상황에 맞는 단 하나의 최적 설정은 없다. 리소스가 제한된 환경에서 실시간으로 데이터를 유실 없이 기록하는 것이 최우선이라면 `fastwrite` 프로파일이 효과적일 수 있다. 반면, 저장 공간 효율성과 후처리 분석의 용이성까지 고려해야 한다면, 압축 옵션(`compression`, `compressionLevel`)과 `chunk_size`를 조절한 기본 MCAP 프로파일을 사용하는 것이 균형 잡힌 선택이다. `--storage-config-file` 옵션을 통해 자신의 시스템 환경과 데이터 특성에 맞게 스토리지 파라미터를 튜닝하는 것이 중요하다.26
- **단순 로깅을 넘어 API를 활용하라**: `rosbag2`의 진정한 가치는 프로그래밍 인터페이스를 통해 드러난다. `rosbag2_py`와 C++ API는 `rosbag2`를 단순한 로거에서 지능형 데이터 처리 파이프라인의 핵심 구성요소로 변모시킨다. 조건부 데이터 로깅, 실시간 데이터 익명화, 데이터 기반 시스템 검증 등 정교한 로봇 애플리케이션을 개발하고자 한다면, `rosbag2` API에 대한 학습은 선택이 아닌 필수이다.32

`rosbag2`는 ROS 2 생태계에서 데이터가 흐르는 방식을 정의하고 제어하는 강력한 수단이다. 이 보고서에서 제시된 심층적인 분석과 실전 가이드가 개발자들이 `rosbag2`를 더욱 효과적으로 활용하여, 더 신뢰성 있고 지능적인 로봇 시스템을 구축하는 데 기여할 수 있기를 바란다.


1. ros2/rosbag2 - GitHub, accessed July 26, 2025, https://github.com/ros2/rosbag2
2. Comparison of Rosbag2 Storage Plugins - MCAP, accessed July 26, 2025, https://mcap.dev/guides/benchmarks/rosbag2-storage-plugins
3. Rosbag2 - Rosbags documentation - GitLab, accessed July 26, 2025, https://ternaris.gitlab.io/rosbags/topics/rosbag2.html
4. Rosbag Backward Compatibility - ROS General - Open Robotics Discourse, accessed July 26, 2025, https://discourse.ros.org/t/rosbag-backward-compatibility/773
5. Recording and playing back data - ROS 2 Documentation: Rolling ..., accessed July 26, 2025, https://docs.ros.org/en/rolling/Tutorials/Beginner-CLI-Tools/Recording-And-Playing-Back-Data/Recording-And-Playing-Back-Data.html
6. rosbag2 - ROS Index, accessed July 26, 2025, https://index.ros.org/r/rosbag2/
7. Iron Irwini (iron) - ROS 2 Documentation: Rolling documentation, accessed July 26, 2025, https://docs.ros.org/en/rolling/Releases/Release-Iron-Irwini.html
8. ros2 bag - Save and Replay Topic Data - The Robotics Back-End, accessed July 26, 2025, https://roboticsbackend.com/ros2-bag-save-and-replay-topic-data/
9. Create and Replay a ROS2 Bag - YouTube, accessed July 26, 2025, https://www.youtube.com/watch?v=a-O1qM9_S7k
10. What are the steps to record and play with rosbag2? - Robotics Stack Exchange, accessed July 26, 2025, https://robotics.stackexchange.com/questions/90158/what-are-the-steps-to-record-and-play-with-rosbag2
11. Quick start - Autoware Documentation, accessed July 26, 2025, https://tier4.github.io/pilot-auto-ros2-iv-archive/tree/main/tutorial/QuickStart/
12. rosbag2_storage_broll: Rolling 0.1.1 documentation, accessed July 26, 2025, https://docs.ros.org/en/rolling/p/rosbag2_storage_broll/
13. Tutorials - ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/Tutorials.html
14. Detect messages dropped by ros2 bag record - Robotics Stack Exchange, accessed July 26, 2025, https://robotics.stackexchange.com/questions/100644/detect-messages-dropped-by-ros2-bag-record
15. rosbag2 doesn't write meta data file - ROS Answers archive, accessed July 26, 2025, https://answers.ros.org/question/397687/
16. ros2/rosbag2 - Gitee, accessed July 26, 2025, https://gitee.com/ros2_humble/rosbag2?skip_mobile=true
17. Using rosbag2_storage_mcap: Integrating MCAP for Efficient ROS 2 ..., accessed July 26, 2025, http://blog.us.fixstars.com/using-rosbag2_storage_mcap/
18. ROS 2 common issues and mistakes - Karelics, accessed July 26, 2025, https://karelics.fi/blog/2023/05/19/ros-2-common-issues-and-mistakes/
19. Fast, accurate, robust replay in ROS2 - ROS Discourse - Open Robotics, accessed July 26, 2025, https://discourse.openrobotics.org/t/fast-accurate-robust-replay-in-ros2/30406
20. rosbag2/docs/storage_plugin_development.md at rolling - GitHub, accessed July 26, 2025, https://github.com/ros2/rosbag2/blob/rolling/docs/storage_plugin_development.md
21. Tooling Proposal rosbag2 leveldb plugin - ROS General - Open Robotics Discourse, accessed July 26, 2025, https://discourse.openrobotics.org/t/tooling-proposal-rosbag2-leveldb-plugin/17072
22. Performance testing of rosbag2 / Issue #435 - GitHub, accessed July 26, 2025, https://github.com/ros2/rosbag2/issues/435
23. Introduction - MCAP, accessed July 26, 2025, https://mcap.dev/guides
24. MCAP as the ROS 2 Default Bag Format - Foxglove, accessed July 26, 2025, https://foxglove.dev/blog/mcap-as-the-ros2-default-bag-format
25. MCAP files, accessed July 26, 2025, https://mcap.dev/
26. ROS Package: rosbag2_storage_mcap - ROS Index, accessed July 26, 2025, https://index.ros.org/p/rosbag2_storage_mcap/
27. rosbag2_storage_mcap: Humble 0.15.15 documentation - the official ROS docs, accessed July 26, 2025, https://docs.ros.org/en/humble/p/rosbag2_storage_mcap/
28. MCAP Format Specification | MCAP, accessed July 26, 2025, https://mcap.dev/spec
29. MCAP Format Registry, accessed July 26, 2025, https://mcap.dev/spec/registry
30. ROS 2 - MCAP, accessed July 26, 2025, https://mcap.dev/guides/getting-started/ros-2
31. MCAP format spec for Kaitai Struct, accessed July 26, 2025, https://formats.kaitai.io/mcap/
32. Recording a bag from a node (Python) - ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/Tutorials/Advanced/Recording-A-Bag-From-Your-Own-Node-Py.html
33. Reading from a bag file (Python) - ROS 2 Documentation, accessed July 26, 2025, https://docs.ros.org/en/rolling/Tutorials/Advanced/Reading-From-A-Bag-File-Python.html
34. Reading and writing ROS 2 | MCAP, accessed July 26, 2025, https://mcap.dev/guides/python/ros2
35. Reading from a bag file (C++) - ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/Tutorials/Advanced/Reading-From-A-Bag-File-CPP.html
36. writer.hpp - ros2/rosbag2 - GitHub, accessed July 26, 2025, https://github.com/ros2/rosbag2/blob/rolling/rosbag2_cpp/include/rosbag2_cpp/writer.hpp
37. berndpfrommer/rosbag2_composable_recorder: ROS2 package that allows recording without interprocess communication - GitHub, accessed July 26, 2025, https://github.com/berndpfrommer/rosbag2_composable_recorder
38. Rosbag2 Anonymizer - Autoware Universe Documentation, accessed July 26, 2025, https://autowarefoundation.github.io/autoware-documentation/main/datasets/data-anonymization/
39. Rosbag1 - Rosbags documentation - GitLab, accessed July 26, 2025, https://ternaris.gitlab.io/rosbags/topics/rosbag1.html
40. Convert ROS1 bag to ROS2 bag - Wiki, accessed July 26, 2025, https://wiki.hanzheteng.com/development/ros2/convert-ros1-bag-to-ros2-bag
41. ros2/rosbag2_bag_v2: rosbag2 plugin for replaying ros1 version2 bag files - GitHub, accessed July 26, 2025, https://github.com/ros2/rosbag2_bag_v2
42. Developer and Internals » ROS1 to ROS2 Bag Conversion Guide - OpenVINS, accessed July 26, 2025, https://docs.openvins.com/dev-ros1-to-ros2.html
43. Unable to install rosbag2 - ROS Answers archive, accessed July 26, 2025, https://answers.ros.org/question/336160/
44. Rosbags documentation - GitLab, accessed July 26, 2025, https://ternaris.gitlab.io/rosbags/index.html
45. rosbag2_transport recorder fails to discover topics within Docker - Robotics Stack Exchange, accessed July 26, 2025, https://robotics.stackexchange.com/questions/115309/rosbag2-transport-recorder-fails-to-discover-topics-within-docker

