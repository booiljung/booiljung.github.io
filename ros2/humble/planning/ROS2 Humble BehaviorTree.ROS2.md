# ROS2 Humble에서의 BehaviorTree.ROS2


로보틱스 분야의 핵심 과제는 복잡성의 관리다. 로봇이 수행해야 할 작업이 정교해지고, 예측 불가능한 동적 환경과 상호작용해야 함에 따라, 로봇의 행동을 결정하는 제어 아키텍처는 점점 더 중요해지고 있다. 전통적으로 이러한 제어 로직을 구현하는 데에는 유한 상태 머신(Finite State Machine, FSM)이 널리 사용되어 왔다. FSM은 시스템을 명확한 '상태(state)'와 그 상태들 사이의 '전이(transition)'로 모델링하는 직관적인 방식으로, 많은 엔지니어에게 익숙한 접근법이다.1

그러나 로봇의 자율성이 고도화되면서 FSM은 본질적인 한계에 부딪히게 되었다. 바로 '상태 폭발(state explosion)' 문제다. 새로운 행동이나 예외 처리 로직을 추가할 때마다 기하급수적으로 증가하는 상태와 전이의 수는 시스템을 이해하고 유지보수하기 어렵게 만드는 주된 원인이 된다.2 예를 들어, 자율 주행 로봇에 '장애물 회피', '경로 재계획', '배터리 충전'과 같은 다양한 부가 기능이 추가되면, 각 기능과 기존 상태 간의 모든 가능한 상호작용을 전이로 정의해야 하므로 제어 로직은 걷잡을 수 없이 복잡해진다.

이러한 배경 속에서 행동 트리(Behavior Tree, BT)는 FSM의 한계를 극복하기 위한 강력한 대안으로 부상했다. 본래 비디오 게임 산업에서 NPC(Non-Player Character)의 복잡한 인공지능을 모델링하기 위해 탄생한 BT는, 그 유연성과 확장성 덕분에 로보틱스 분야에서 빠르게 채택되었다.4 BT의 핵심 철학은 '상태'가 아닌 '행동(Action)' 중심의 설계에 있다. 즉, 정적인 상태와 그 사이의 전이를 관리하는 대신, 계층적이고 조합 가능하며 반응적인 방식으로 행동 콜백을 실행하는 데 초점을 맞춘다.2 이러한 모듈성과 확장성은 복잡한 자율 시스템을 구축하는 데 있어 BT가 가진 가장 큰 장점이다.2

ROS2(Robot Operating System 2) 생태계에서 BT의 채택은 단순한 기술적 선호를 넘어, 현대 로보틱스가 요구하는 복잡성을 해결하기 위한 필연적인 진화 과정으로 볼 수 있다. 초기 로봇 시스템이나 간단한 ROS1 애플리케이션은 SMACH와 같은 FSM으로 충분히 관리 가능했지만, Nav2의 정교한 내비게이션, MoveIt의 복잡한 조작(manipulation)과 같이 다단계의 반응형 작업이 요구되는 현대 로보틱스 시스템에서 FSM 방식은 취약성을 드러냈고, 8 Nav2와 같은 핵심 ROS2 프로젝트가 BT를 깊숙이 통합한 것은, 개발자들이 현대 자율성의 복잡성을 다루기 위해 내린 신중한 아키텍처적 결정이다. 이는 로보틱스 분야가 단순한 스크립트 기반의 동작에서 벗어나, 더욱 견고하고 확장 가능한 자율성으로 성숙해 가고 있음을 보여주는 명백한 증거다.

이 보고서는 ROS2 Humble Hawksbill 배포판을 기준으로, 사실상의 표준 C++ BT 구현체인 `BehaviorTree.ROS2` 패키지에 대한 심층적인 기술 분석을 제공하는 것을 목표로 한다. `BehaviorTree.CPP` 라이브러리의 근본적인 설계 철학부터 ROS2 통신과의 통합 패턴, 그리고 Nav2 내비게이션 스택에서의 실제 적용 사례에 이르기까지 포괄적으로 다룰 것이다. 이를 통해 개발자들이 자신의 ROS2 프로젝트에서 행동 트리를 효과적으로 사용하고, 확장하며, 디버깅하는 데 필요한 실질적인 지식을 제공하고자 한다.


`BehaviorTree.ROS2` 패키지를 이해하기 위해서는 그 기반이 되는 `BehaviorTree.CPP` 라이브러리의 핵심 원리와 설계 철학을 먼저 파악해야 한다. 이 라이브러리는 로보틱스와 AI 분야에서 널리 사용되는 C++ 행동 트리 구현체로, 유연성, 성능, 그리고 사용 편의성에 중점을 두고 설계되었다.4


`BehaviorTree.CPP`의 가장 근본적인 철학은 "상태가 아닌 행동의 관점에서 생각하라(Think in terms of Actions, not states)"는 것이다.4 이는 FSM과의 가장 큰 차이점이다. FSM이 시스템의 현재 '상태'가 무엇이고, 어떤 '이벤트'가 발생했을 때 다른 상태로 '전이'할 것인가에 집중하는 반면, BT는 '어떤 조건 하에서 어떤 행동을 실행할 것인가'에 집중한다.

BT에서 시스템의 '상태'는 명시적으로 정의되지 않는다. 대신, 현재 실행 중인 행동(들)의 결과로 나타나는 부수적인 속성(emergent property)으로 간주된다. 예를 들어, 로봇이 "NavigateToPose"라는 행동을 수행하고 있다면, 로봇은 '내비게이션 중'인 상태에 있다고 볼 수 있다. 이 행동이 성공적으로 끝나면 다음 행동으로 넘어가고, 실패하면 다른 대체 행동을 시도하게 된다. 이처럼 BT는 정해진 조건 하에서 콜백 함수를 호출하는 메커니즘으로 이해할 수 있으며, 이 콜백 안에서 어떤 작업을 수행할지는 전적으로 개발자의 몫이다.12


`BehaviorTree.CPP`는 행동 트리를 구성하는 몇 가지 기본 노드 유형을 제공한다. 이들은 트리의 논리적 흐름을 제어하고 실제 작업을 수행하는 기본 단위다.6 모든 노드는 실행이 끝나면 세 가지 상태 중 하나를 반환한다: 

`SUCCESS`(성공), `FAILURE`(실패), `RUNNING`(실행 중).


컨트롤 플로우 노드는 자식 노드들을 특정 규칙에 따라 실행하고, 그 결과를 조합하여 자신의 상태를 결정하는 역할을 한다. 이들은 트리의 가지(branch)에 해당한다.

- **`Sequence`**: 자식 노드들을 순서대로 실행한다. 자식 노드 중 하나라도 `FAILURE`를 반환하면 즉시 실행을 멈추고 `FAILURE`를 반환한다. 모든 자식 노드가 `SUCCESS`를 반환해야만 `SUCCESS`를 반환한다. 이는 논리적 AND 연산과 유사하며, 순차적인 작업 흐름을 정의하는 데 사용된다. 예를 들어, "문 열기 -> 물건 잡기 -> 문 닫기"와 같은 일련의 작업을 표현할 수 있다.

- **`Fallback` (또는 `Selector`)**: 자식 노드들을 순서대로 실행하다가 하나라도 `SUCCESS`를 반환하면 즉시 실행을 멈추고 `SUCCESS`를 반환한다. 모든 자식 노드가 `FAILURE`를 반환해야만 `FAILURE`를 반환한다. 이는 논리적 OR 연산과 유사하며, 여러 대안 중 하나를 선택하거나 실패 시 복구 동작을 정의하는 데 이상적이다. 예를 들어, "계획 A 시도 -> 실패 시 계획 B 시도 -> 실패 시 비상 정지"와 같은 로직을 구현할 수 있다.13

- **`Parallel`**: 모든 자식 노드를 "병렬적으로" 실행한다. 여기서 병렬은 실제 멀티스레딩이 아니라, 매 틱(tick)마다 모든 자식 노드를 순서대로 한 번씩 실행한다는 의미다. 이 노드는 사전에 정의된 조건(예: M개의 자식이 `SUCCESS`를 반환하면 `SUCCESS` 반환)에 따라 자신의 상태를 결정한다. 여러 작업을 동시에 모니터링하거나 수행해야 할 때 유용하다.

- **`Reactive Nodes` (`ReactiveSequence`, `ReactiveFallback`)**: 이들은 BT의 반응성을 극대화하는 매우 중요한 노드다. 일반적인 `Sequence`나 `Fallback`은 자식 노드 중 하나가 `RUNNING` 상태를 반환하면, 그 자식 노드가 `SUCCESS`나 `FAILURE`로 바뀔 때까지 다른 자식들을 실행하지 않는다. 하지만 `Reactive` 노드들은 다르다. 자식 노드가 `RUNNING` 상태일지라도, 매 틱마다 이전 자식 노드들(실행 중인 노드 포함)을 다시 실행(re-tick)한다.2 이 특성 덕분에 트리는 현재 실행 중인 장기 작업을 중단하지 않고도 환경 변화에 즉각적으로 반응할 수 있다. 예를 들어, 

  `ReactiveFallback`의 첫 번째 자식으로 "비상 정지 버튼 확인" 조건을 두고 두 번째 자식으로 "경로 주행" 액션을 두면, 로봇은 경로를 주행하는 중에도 매 틱마다 비상 정지 버튼 상태를 확인하여 즉시 멈출 수 있다.


실행 노드는 트리의 가장 끝에 위치하는 잎(leaf) 노드로서, 실제 작업을 수행한다.

- **`ActionNode`**: BT의 실질적인 "일꾼"이다. 시간이 걸릴 수 있는 작업을 수행하며, `SUCCESS`, `FAILURE`, 또는 `RUNNING` 상태를 반환할 수 있다. 로봇의 이동, 센서 데이터 처리, 외부 시스템과의 통신 등 대부분의 구체적인 작업이 `ActionNode`로 구현된다.
- **`ConditionNode`**: `ActionNode`의 특수한 형태로, 어떤 조건을 확인하는 역할을 한다. `tick()`이 호출되면 즉시 `SUCCESS` 또는 `FAILURE`를 반환하며, 절대로 `RUNNING` 상태를 반환할 수 없다. "배터리가 충분한가?", "목표 지점에 도달했는가?"와 같은 검사를 수행하는 데 사용된다.


데코레이터 노드는 단 하나의 자식 노드를 가지며, 그 자식 노드의 행동을 수정하거나 반환되는 상태를 변경하는 역할을 한다.

- **`Inverter`**: 자식 노드가 `SUCCESS`를 반환하면 `FAILURE`로, `FAILURE`를 반환하면 `SUCCESS`로 상태를 뒤집는다. `RUNNING`은 그대로 유지한다. "장애물이 없는지 확인(`SUCCESS`)"하는 조건을 "장애물이 있으면(`FAILURE`)"으로 바꾸고 싶을 때 유용하다.
- **`RetryUntilSuccessful`**: 자식 노드가 `FAILURE`를 반환하면, 지정된 횟수만큼 재시도한다. 재시도 중 `SUCCESS`를 반환하면 자신도 `SUCCESS`를 반환하고, 모든 재시도 후에도 실패하면 `FAILURE`를 반환한다.
- **`Timeout`**: 자식 노드가 지정된 시간 이상 `RUNNING` 상태를 유지하면, 자식 노드를 강제로 중지(`halt`)시키고 `FAILURE`를 반환한다.



BT의 실행은 '틱(tick)'이라는 신호에 의해 구동된다. 일정한 주기(예: 10 Hz)로 루트 노드에 틱 신호가 전달되면, 이 신호는 컨트롤 플로우 노드의 논리 규칙에 따라 자식 노드로 전파된다.6 예를 들어, `Sequence` 노드는 첫 번째 자식에게 틱을 보내고, 그 자식이 `SUCCESS`를 반환하면 두 번째 자식에게 틱을 보낸다. 이 틱 기반 실행 모델은 BT가 지속적으로 환경을 평가하고 반응적으로 행동을 결정하게 하는 핵심 메커니즘이다.


BT의 각 노드는 독립적으로 설계되지만, 종종 서로 데이터를 주고받아야 한다. 예를 들어, 경로를 계산하는 노드(`ComputePathToPose`)는 그 결과를 경로를 따라가는 노드(`FollowPath`)에 전달해야 한다. 이러한 데이터 공유를 위해 `BehaviorTree.CPP`는 '블랙보드(Blackboard)'라는 메커니즘을 제공한다.6

블랙보드는 모든 노드가 공유하는 간단한 키-값(key-value) 저장소다. 한 노드가 블랙보드의 특정 키에 값을 쓰면, 다른 노드는 그 키를 통해 값을 읽을 수 있다. 이 방식은 노드 간의 직접적인 의존성(tight coupling)을 제거하여 모듈성을 크게 향상시킨다. `ComputePathToPose`와 `FollowPath` 노드는 서로의 존재를 알 필요 없이, 단지 'path'라는 이름의 블랙보드 항목을 통해 데이터를 교환하면 된다. 이 덕분에 개발자는 `ComputePathToPose`를 파일에서 경로를 읽어오는 다른 노드로 쉽게 교체할 수 있으며, `FollowPath` 노드는 전혀 수정할 필요가 없다. 이는 진정한 모듈식 플러그-앤-플레이 아키텍처의 정수다.

'포트(Port)'는 각 노드가 필요로 하는 데이터(입력 포트)와 생성하는 데이터(출력 포트)를 명시적으로 선언하는 타입-안전(type-safe) 메커니즘이다.12 XML 파일에서 이 포트들은 블랙보드의 특정 항목에 매핑된다. 이를 통해 각 노드의 데이터 인터페이스가 명확해지고, 재사용성이 높아진다.


`BehaviorTree.CPP`의 가장 강력한 설계 패턴 중 하나는 '무엇을 할 것인가(What)'와 '어떻게 할 것인가(How)'를 분리하는 것이다. 행동의 논리적 흐름, 즉 트리의 구조는 XML 파일에 정의하고, 각 행동의 구체적인 구현은 C++ 코드로 작성한다.4

모든 노드는 `BT::TreeNode` 클래스를 상속받으며, 핵심 가상 함수인 `tick()`과 `halt()`를 구현해야 한다.15

`tick()`은 노드가 실행될 때 호출되는 함수이고, `halt()`는 상위 노드의 결정에 따라 현재 진행 중인 작업을 중단해야 할 때 호출된다. `BT::SyncActionNode`는 한 번의 `tick` 안에 완료되는 동기적 행동에 사용되고, `BT::StatefulActionNode`는 여러 `tick`에 걸쳐 상태를 유지해야 하는 비동기적 행동의 기반이 된다.8

이렇게 C++로 작성된 커스텀 노드들은 `BehaviorTreeFactory`에 등록된다. `BehaviorTreeFactory`는 일종의 중개자 역할을 하여, 등록된 C++ 노드 타입의 목록을 관리한다.12 런타임에 `BehaviorTreeFactory`는 XML 파일을 파싱하고, XML에 명시된 노드 이름에 해당하는 C++ 객체를 생성하여 트리 구조로 조립한다.

예를 들어, `ApproachObject`라는 커스텀 노드를 C++로 구현한 뒤, 팩토리에 등록하는 코드는 다음과 같다.

C++

```
// MyNodes.h
class ApproachObject : public BT::SyncActionNode
{
public:
    ApproachObject(const std::string& name, const BT::NodeConfiguration& config)
        : BT::SyncActionNode(name, config) {}

    static BT::PortsList providedPorts() { return {}; }

    BT::NodeStatus tick() override {
        std::cout << "Approaching the object..." << std::endl;
        return BT::NodeStatus::SUCCESS;
    }
};

// main.cpp
#include "behaviortree_cpp/bt_factory.h"
#include "MyNodes.h"

int main()
{
    BT::BehaviorTreeFactory factory;
    factory.registerNodeType<ApproachObject>("ApproachObject");

    // XML에서 트리를 로드
    auto tree = factory.createTreeFromFile("./my_tree.xml");

    // 트리를 실행
    tree.tickWhileRunning();

    return 0;
}
```

그리고 이 노드를 사용하는 XML 파일(`my_tree.xml`)은 다음과 같이 작성할 수 있다.

XML

```
<root BTCPP_format="4">
    <BehaviorTree ID="MainTree">
        <Sequence name="root_sequence">
            <ApproachObject/>
            </Sequence>
    </BehaviorTree>
</root>
```

이러한 '구현과 정의의 분리'는 C++ 개발자가 견고하고 재사용 가능한 행동 기본 단위(action primitives)를 만드는 데 집중하게 하고, 시스템 통합 담당자나 비프로그래머가 코드를 재컴파일하지 않고도 XML 파일 수정만으로 로봇의 행동을 재구성할 수 있게 해준다. 이는 개발 및 실험 속도를 극적으로 향상시키는 강력한 아키텍처 패턴이다.4


`BehaviorTree.CPP`가 강력한 범용 BT 라이브러리라면, `BehaviorTree.ROS2`는 이를 ROS2 생태계와 매끄럽게 연결하는 핵심적인 다리 역할을 하는 패키지다.17 이 패키지는 ROS2의 통신 메커니즘(Action, Service, Topic)을 BT 노드로 쉽게 래핑(wrapping)할 수 있는 표준화된 방법을 제공한다.


`BehaviorTree.ROS2`의 설계 목표는 명확하다. 첫째, ROS2와의 상호작용에 필요한 상용구 코드(boilerplate code)를 최소화하는 것이다. 둘째, 그리고 가장 중요한 것은, ROS2의 비동기 통신을 BT의 틱 기반 실행 모델에 맞춰 비차단(non-blocking) 방식으로 처리하는 것이다.17

만약 BT 노드의 `tick()` 함수가 ROS2 Action이나 Service의 응답을 기다리며 차단(block)된다면, 전체 BT의 실행 주기가 지연된다. 이는 BT의 가장 큰 장점인 '반응성'을 심각하게 훼손하는 결과를 낳는다. 로봇은 장시간의 작업(예: 1분간 이동)이 끝날 때까지 새로운 센서 데이터나 상위 우선순위의 이벤트에 반응할 수 없게 된다.8 따라서 

`BehaviorTree.ROS2`는 이러한 "임피던스 불일치(impedance mismatch)" 문제를 해결하는 데 중점을 둔다.

권장되는 시스템 아키텍처는 BT를 실행하는 중앙 집중식 '코디네이터(Coordinator)' 또는 '작업 플래너(Task Planner)' ROS 노드를 두고, 로봇의 다른 기능들(예: 경로 계획, 물체 인식)은 독립적인 Action 또는 Service 서버로 구현하는 것이다.18 BT는 이 서버들을 호출하여 전체 작업을 조율하는 역할을 맡는다.


`RosActionNode` 래퍼는 `BehaviorTree.ROS2`에서 가장 중요하고 정교한 구성 요소다. 이는 BT의 동기적, 폴링(polling) 기반의 `tick()`과 ROS2 Action의 비동기적, 이벤트 기반 통신 모델 사이의 근본적인 충돌을 우아하게 해결한다.

ROS2 Action은 목표 전송(send goal), 주기적인 피드백(feedback) 수신, 최종 결과(result) 수신의 비동기적 흐름을 가진다.18

`BT::RosActionNode<ActionT>` 템플릿 클래스는 이 비동기적 흐름을 BT의 `SUCCESS`, `FAILURE`, `RUNNING` 상태와 동기화하기 위해 내부적으로 비동기 상태 머신 패턴을 구현한다. 그 실행 흐름은 다음과 같다.

1. **첫 번째 `tick()` 호출**: BT가 이 노드를 처음으로 `tick`하면, 래퍼 클래스는 사용자가 재정의한 `setGoal()` 메서드를 호출한다. 이 메서드 안에서 개발자는 BT의 입력 포트(블랙보드)로부터 값을 읽어와 ROS2 Action의 목표 메시지를 채운다. 목표 설정이 성공적으로 완료되면, 래퍼는 내부적으로 ROS2 Action 클라이언트를 통해 목표를 서버로 전송하고, 즉시 `BT::NodeStatus::RUNNING`을 반환한다. 여기서 핵심은 **차단하지 않고 즉시 반환**한다는 점이다.18

2. **이후의 `tick()` 호출**: Action 서버가 작업을 처리하는 동안, BT는 주기적으로 이 노드를 계속 `tick`한다. 이때마다 노드는 단순히 자신의 내부 상태가 '진행 중'임을 확인하고 계속해서 `RUNNING`을 반환한다. 이는 상위 노드에게 "아직 작업이 끝나지 않았으니 기다려달라"는 신호를 보내는 것과 같다.

3. **피드백(Feedback) 처리**: 래퍼 클래스는 ROS2의 콜백 메커니즘을 통해 Action 서버로부터 오는 피드백 메시지를 백그라운드에서 수신한다. 피드백이 도착하면, 사용자가 선택적으로 재정의한 `onFeedback()` 콜백 함수가 호출된다. 개발자는 이 함수를 이용해 작업 진행 상황을 모니터링하거나, 특정 피드백 값에 따라 `SUCCESS` 또는 `FAILURE`를 반환하여 현재 Action을 선점적으로 종료시킬 수도 있다.18

4. **결과(Result) 처리**: Action 서버가 작업을 완료하고 최종 결과를 보내면, 래퍼의 내부 결과 콜백이 이를 수신한다. 이 콜백은 사용자가 재정의한 `onResultReceived()` 메서드를 호출한다. 이 메서드의 반환 값(`SUCCESS` 또는 `FAILURE`)이 바로 BT 노드의 최종 상태가 된다.18 예를 들어, 

   `onResultReceived()`에서 Action 결과가 성공적이라고 판단되면 `BT::NodeStatus::SUCCESS`를 반환하고, 이때부터 BT는 다음 노드를 실행하게 된다.

5. **중단(Halt) 처리**: 만약 트리의 다른 부분(예: 더 높은 우선순위의 `Fallback` 브랜치)이 활성화되어 현재 실행 중인 Action 노드가 중단되어야 할 경우, BT 엔진은 해당 노드의 `halt()` 메서드를 호출한다. `RosActionNode` 래퍼는 `halt()`가 호출되면 자동으로 Action 서버에 취소(cancel) 요청을 보낸다.18 이 기능은 BT의 반응성을 보장하는 데 필수적이다.

`Fibonacci` Action 예제를 기반으로 한 `RosActionNode` 구현 코드는 다음과 같다.18

```C++
#include <behaviortree_ros2/bt_action_node.hpp>
#include "example_interfaces/action/fibonacci.hpp"

class FibonacciAction : public BT::RosActionNode<example_interfaces::action::Fibonacci>
{
public:
    FibonacciAction(const std::string& name,
                    const BT::NodeConfiguration& conf,
                    const BT::RosNodeParams& params)
        : BT::RosActionNode<example_interfaces::action::Fibonacci>(name, conf, params) {}

    // 이 노드가 사용하는 포트들을 정의
    static BT::PortsList providedPorts()
    {
        return providedBasicPorts({BT::InputPort<unsigned>("order")});
    }

    // 목표 메시지를 설정하는 함수 (사용자 재정의)
    bool setGoal(Goal& goal) override
    {
        // 입력 포트에서 "order" 값을 읽어와 goal에 설정
        return getInput("order", goal.order);
    }

    // 결과 메시지를 받았을 때 호출되는 콜백 (사용자 재정의)
    BT::NodeStatus onResultReceived(const WrappedResult& wr) override
    {
        RCLCPP_INFO(node_->get_logger(), "Result received.");
        // Action의 결과 코드에 따라 BT 노드의 상태를 결정
        if (wr.code == rclcpp_action::ResultCode::SUCCEEDED) {
            return BT::NodeStatus::SUCCESS;
        }
        return BT::NodeStatus::FAILURE;
    }

    // 피드백 메시지를 받았을 때 호출되는 콜백 (선택적 재정의)
    BT::NodeStatus onFeedback(const std::shared_ptr<const Feedback> feedback) override
    {
        RCLCPP_INFO(node_->get_logger(), "Next number in sequence received: %d", feedback->partial_sequence.back());
        return BT::NodeStatus::RUNNING; // 피드백을 받아도 계속 실행
    }

    // 통신 오류 등 실패 시 호출되는 콜백 (사용자 재정의)
    BT::NodeStatus onFailure(BT::ActionNodeErrorCode error) override
    {
        RCLCPP_ERROR(node_->get_logger(), "Action failed with error code: %d", error);
        return BT::NodeStatus::FAILURE;
    }
};
```

이처럼 `RosActionNode` 래퍼는 BT의 `tick`과 ROS 통신 라이프사이클을 분리함으로써, 개발자가 복잡한 비동기 처리 로직을 직접 구현할 필요 없이 선언적인 방식으로 ROS2 Action을 BT에 통합할 수 있게 해준다. 이는 `BehaviorTree.ROS2`가 제공하는 가장 핵심적인 가치다.


ROS2 Service를 위한 래퍼인 `BT::RosServiceNode<ServiceT>`도 `RosActionNode`와 유사한 비차단 방식으로 작동한다.18 Service는 Action과 달리 피드백이나 취소 기능이 없지만, 요청-응답 모델 자체가 비동기적으로 처리될 수 있다.

`RosServiceNode`는 첫 번째 `tick`에서 Service 요청을 보내고 즉시 `RUNNING`을 반환한다. 백그라운드에서 Service 응답이 수신되면, 내부 콜백이 호출되고 노드의 상태는 `SUCCESS` 또는 `FAILURE`로 전환된다. 이 역시 `tick()` 함수가 차단되는 것을 방지하여 BT의 반응성을 유지한다.


`BehaviorTree.ROS2`는 Topic 통신을 위한 간단한 래퍼도 제공한다.17 이들은 보통 동기적인 노드로 구현된다.

- **`TopicPublisher` 노드**: `tick()`이 호출될 때 블랙보드 포트에서 데이터를 가져와 특정 토픽으로 발행(publish)하고 즉시 `SUCCESS`를 반환하는 형태로 구현될 수 있다.
- **`TopicSubscriber` 노드**: 특정 토픽에서 메시지가 수신될 때까지 기다리거나(`RUNNING` 반환), 메시지가 도착하면 그 내용을 블랙보드 포트에 쓰고 `SUCCESS`를 반환하는 방식으로 구현된다. 예를 들어, `as2_behavior_tree` 패키지의 `WaitForEvent` 노드는 지정된 토픽에 이벤트가 게시될 때까지 기다리는 역할을 한다.19

이러한 래퍼들을 통해 개발자는 ROS2의 모든 표준 통신 방식을 BT 내에서 일관되고 효율적으로 사용할 수 있다.


`BehaviorTree.ROS2`의 실제 적용 사례를 가장 잘 보여주는 것은 단연 ROS2의 표준 내비게이션 스택인 Nav2다.9 Nav2는 단순히 BT를 사용하는 것을 넘어, BT를 중심으로 전체 내비게이션 로직을 구성하고, 이를 위한 풍부한 도구와 플러그인을 제공하는 거대한 생태계다.


Nav2의 핵심 컴포넌트 중 하나인 `nav2_bt_navigator` 서버는 `BehaviorTree.CPP`를 사용하여 내비게이션의 전체 흐름을 조율한다.9

`nav2_bt_navigator`는 단순히 BT를 실행하는 역할만 하는 것이 아니라, `nav2_behavior_tree`라는 패키지를 통해 내비게이션에 특화된 방대한 양의 BT 노드 라이브러리를 제공한다.9

이 라이브러리에는 다음과 같은 다양한 노드들이 포함되어 있다:

- **Action Nodes**: `ComputePathToPose`(경로 계산), `FollowPath`(경로 추종), `Spin`(제자리 회전), `BackUp`(후진), `ClearEntireCostmap`(코스트맵 초기화) 등
- **Condition Nodes**: `IsStuck`(로봇이 끼었는지 확인), `IsPathValid`(경로 유효성 검사), `GoalReached`(목표 도달 확인) 등
- **Decorator Nodes**: `RateController`(자식 노드의 실행 주기 제어), `DistanceController`(이동 거리에 따라 자식 노드 실행), `SpeedController`(로봇 속도 제한) 등

개발자는 처음부터 이 모든 노드를 구현할 필요 없이, Nav2가 제공하는 검증된 노드들을 레고 블록처럼 조립하여 자신만의 내비게이션 동작을 만들 수 있다. 이는 커스텀 내비게이션 로직 개발의 진입 장벽을 극적으로 낮추는 효과를 가져온다. Nav2가 제공하는 기본 BT XML 파일(예: `navigate_w_replanning_and_recovery.xml`)을 살펴보면, 이러한 노드들이 어떻게 조합되어 복잡하고 강건한 행동을 만들어내는지 명확히 알 수 있다.10


Nav2가 어떻게 BT를 활용하여 강건한(robust) 시스템을 구축하는지 보여주는 대표적인 예가 바로 `RecoveryNode`다. 이는 실패 상황을 처리하고 복구 동작을 수행하기 위해 설계된 커스텀 컨트롤 노드다.10

FSM에서 내비게이션 실패를 처리하려면 `NAVIGATING` 상태에서 `RECOVERY_SPIN`, `RECOVERY_CLEAR_MAP` 등 다양한 복구 상태로 향하는 복잡한 전이들을 일일이 정의해야 한다. 이는 '스파게티 코드'처럼 얽혀 가독성과 유지보수성을 해치기 쉽다.2

`RecoveryNode`는 이러한 "시도-실패-복구-재시도" 로직 전체를 하나의 재사용 가능한 컨트롤 플로우 노드로 캡슐화한다. 그 작동 원리는 다음과 같다 10:

1. `RecoveryNode`는 '주요 행동(primary)' 자식과 '복구 행동(recovery)' 자식, 두 개의 자식을 가진다.
2. 먼저 주요 행동 자식을 `tick`한다. 만약 이 자식이 `SUCCESS`를 반환하면, `RecoveryNode`도 `SUCCESS`를 반환하며 정상적으로 종료된다.
3. 만약 주요 행동 자식이 `FAILURE`를 반환하면, `RecoveryNode`는 복구 행동 자식을 `tick`한다.
4. 복구 행동 자식이 `SUCCESS`를 반환하면(즉, 복구가 성공하면), `RecoveryNode`는 다시 주요 행동 자식을 `tick`하여 작업을 재시도한다. 이 과정은 `number_of_retries`라는 매개변수에 지정된 횟수만큼 반복될 수 있다.
5. `RecoveryNode`가 최종적으로 `FAILURE`를 반환하는 경우는, 주요 행동이 실패하고 복구 행동마저 실패하거나, 혹은 지정된 재시도 횟수를 모두 소진했을 때다.

Nav2는 이 `RecoveryNode`를 사용하여 주 내비게이션 루프를 구성한다. 주요 행동 자식으로는 경로 계획과 추종을 담당하는 `PipelineSequence`를 두고, 복구 행동 자식으로는 `Spin`, `BackUp`, `ClearCostmap` 등의 여러 복구 액션을 순서대로 시도하는 `Fallback` 또는 `RoundRobin` 노드를 둔다.10 이처럼 

`RecoveryNode`는 복잡한 오류 처리 로직을 명확하고 재사용 가능한 단위로 추상화하여, BT가 제공하는 더 높은 수준의 "문법"을 통해 복잡한 행동 패턴을 얼마나 효과적으로 표현할 수 있는지를 보여주는 훌륭한 사례다.22


Nav2의 강력함은 제공된 노드를 사용하는 데 그치지 않고, 사용자가 직접 새로운 BT 노드를 플러그인 형태로 쉽게 추가할 수 있다는 점에 있다. Nav2 공식 문서에는 이를 위한 상세한 튜토리얼이 제공된다.21 커스텀 플러그인 개발 과정은 다음과 같이 요약할 수 있다.

1. **C++ 클래스 생성**: 새로운 노드의 역할을 정의하는 C++ 클래스를 작성한다. ROS2 Action 클라이언트를 포함하는 노드라면 `nav2_behavior_tree::BtActionNode<ActionT>`를 상속받는 것이 편리하다.21
2. **가상 함수 구현**: `setGoal()`, `onResultReceived()` 등 필요한 가상 함수들을 재정의하여 노드의 구체적인 동작을 구현한다.
3. **플러그인 등록**: `pluginlib`를 사용하여 작성한 클래스를 플러그인으로 등록한다. 여기에는 `BT_REGISTER_NODES` 매크로를 사용하고, `package.xml` 파일에 플러그인을 export하는 과정이 포함된다.
4. **파라미터 설정**: `bt_navigator`의 파라미터 YAML 파일에 새로 만든 플러그인 라이브러리의 이름을 추가하여 런타임에 로드될 수 있도록 설정한다.
5. **XML에서 사용**: 커스텀 BT XML 파일에서 등록된 플러그인 이름을 태그로 사용하여 새로운 노드를 트리에 포함시킨다.21

이러한 플러그인 아키텍처 덕분에 Nav2는 단순한 BT의 '소비자'를 넘어, 커뮤니티와 함께 진화하는 '생산적인 생태계'가 된다. 개발자들은 Nav2가 제공하는 견고한 기반 위에서 자신만의 특수한 내비게이션 기능을 손쉽게 확장하고 공유할 수 있다.


로봇의 고수준 행동을 제어하기 위한 아키텍처를 선택할 때, ROS 개발자들은 전통적인 FSM 접근법인 SMACH와 현대적인 BT 접근법 사이에서 고민하게 된다. ROS2 Humble을 기준으로 두 기술을 여러 측면에서 비교 분석하면, 왜 BT가 복잡한 현대 로봇 시스템에 더 적합한 선택인지 명확해진다.

| 기능                             | BehaviorTree.ROS2 (`BehaviorTree.CPP`)                       | SMACH (`executive_smach`)                                    |
| -------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **핵심 패러다임**                | 계층적, 틱 기반의 작업 실행 (Hierarchical, tick-based task execution) 4 | 상태 전이 및 이벤트 기반 로직 (State transitions and event-driven logic) 1 |
| **모듈성**                       | **높음:** 서브트리(Subtree)를 통해 자연스럽게 구성 가능. 재사용성 극대화.4 | **보통:** 중첩된 상태 머신으로 구현 가능하나, 복잡한 연결 구조를 유발할 수 있다. |
| **확장성**                       | **높음:** 트리에 새로운 가지를 추가하는 방식으로 '상태 폭발' 문제를 우아하게 회피.2 | **낮음:** 상태와 전이의 수가 증가함에 따라 관리 불가능한 복잡성에 빠지기 쉽다.2 |
| **반응성**                       | **높음:** 반응형 노드(Reactive Nodes)는 조건의 지속적인 재평가를 통해 환경 변화에 신속한 대응이 가능하다.2 | **보통:** 반응성은 정의된 상태 전이에 국한된다. 본질적으로 '폴링' 방식이 아니다. |
| **데이터 관리**                  | 중앙 집중식 `Blackboard`를 통한 유연하지만 복잡할 수 있는 데이터 공유.2 | 상태 수준의 `userdata` 입출력. 더 명시적이고 격리되어 있으나, 장황해질 수 있다. |
| **디버깅 도구**                  | **우수함:** Groot2는 강력한 시각적 설계, 실시간 모니터링 및 디버깅 기능을 제공한다.26 | **양호(단, 주의 필요):** `smach_viewer`는 기능적이지만 ROS2 포팅 및 유지보수가 일관되지 않았다.28 |
| **ROS2 Humble 성숙도 및 생태계** | **높음:** 활발하게 유지보수되며 Nav2의 핵심 의존성. ROS2 생태계의 일등 시민으로 간주된다.9 | **보통:** Humble로 포팅되었으나 개발이 더뎠고, 최신 ROS2 시스템에서의 커뮤니티 모멘텀과 예제가 부족하다.28 |

**상세 분석:**

- **모듈성 및 확장성**: BT의 가장 큰 장점은 모듈성과 확장성이다. 특정 기능을 수행하는 노드들의 집합을 '서브트리(Subtree)'로 캡슐화하여 다른 트리에서 쉽게 재사용할 수 있다.7 새로운 행동을 추가하는 것은 트리에 새로운 가지(branch)를 추가하는 것만으로 간단히 해결되는 경우가 많다. 반면, SMACH에서 새로운 상태를 추가하려면 기존의 여러 상태와의 전이를 모두 고려해야 하므로, 시스템이 커질수록 복잡성이 기하급수적으로 증가하는 '상태 폭발' 문제에 직면하게 된다.2

- **반응성**: BT, 특히 `Reactive` 노드를 사용하면 시스템이 환경 변화에 훨씬 민첩하게 반응할 수 있다.2 틱 기반 실행 모델은 주기적으로 트리의 조건을 재평가하여, 장시간 실행되는 작업 도중에도 우선순위가 높은 이벤트(예: 안전 센서 감지)에 즉각적으로 대응할 수 있게 한다. SMACH는 본질적으로 이벤트가 발생하여 명시적인 전이를 트리거하기 전까지는 현재 상태에 머물러 있기 때문에, 이와 같은 수준의 지속적인 반응성을 구현하기가 더 까다롭다.

- **데이터 관리**: BT는 모든 노드가 접근할 수 있는 중앙 `Blackboard`를 사용한다. 이는 매우 유연한 데이터 공유를 가능하게 하지만, 여러 노드가 동시에 같은 데이터를 수정할 경우 스레드 안전성(thread-safety)과 같은 문제를 신중하게 관리해야 하는 복잡성이 있다.2 SMACH는 

  `userdata`라는 딕셔너리를 통해 상태 간에 데이터를 명시적으로 전달한다. 이는 데이터 흐름을 더 명확하게 하지만, 많은 데이터를 전달해야 할 경우 코드가 장황해질 수 있다.

- **ROS2 성숙도 및 모멘텀**: 기술 선택에 있어 커뮤니티의 지원과 생태계의 성숙도는 매우 중요한 요소다. `BehaviorTree.CPP`와 `BehaviorTree.ROS2`는 Nav2와 같은 핵심 프로젝트의 필수 구성 요소로서 활발하게 개발 및 유지보수되고 있다.17 반면, SMACH의 ROS2 버전은 Humble 배포판에서 사용 가능하게 되었지만, 개발 과정이 순탄치 않았고 Action 래퍼와 같은 핵심 기능에 문제가 있었던 점, 그리고 

  `smach_viewer`와 같은 시각화 도구의 지원이 늦어진 점 등은 커뮤니티의 관심과 모멘텀이 BT 쪽으로 이동했음을 시사한다.28

결론적으로, 간단하고 순차적인 작업을 위해서는 SMACH도 여전히 유효한 선택일 수 있지만, 복잡하고 동적인 환경에서 작동해야 하는 현대 로봇 애플리케이션의 고수준 작업 조율을 위해서는 `BehaviorTree.ROS2`가 기술적 우위와 생태계 성숙도 측면에서 명백히 더 나은 선택이다.


강력한 도구 생태계는 기술 채택을 가속화하는 중요한 촉매제다. `BehaviorTree.CPP`와 `BehaviorTree.ROS2`는 개발자의 생산성과 디버깅 효율을 크게 향상시키는 우수한 도구들을 갖추고 있으며, 그 중심에는 Groot2가 있다.


Groot2는 `BehaviorTree.CPP`를 위한 공식 IDE(통합 개발 환경)로, 추상적인 코딩 개념을 구체적이고 관리 가능한 시각적 시스템으로 변환하여 개발자의 인지 부하를 크게 낮춰준다.4

- **시각적 설계**: Groot2는 드래그-앤-드롭 인터페이스를 제공하여 개발자가 BT를 시각적으로 설계하고 편집할 수 있게 한다.4 복잡한 BT 구조를 순수한 XML이나 코드로만 파악하는 것은 매우 어렵지만, Groot2는 계층적 관계와 데이터 흐름을 직관적으로 보여주어 시스템의 "사고 과정"을 쉽게 이해하도록 돕는다.

- **노드 팔레트 생성**: Groot2가 커스텀 노드를 인식하게 하려면, C++ 코드로부터 'TreeNode 모델'을 생성하는 과정이 필수적이다. 개발자는 `BehaviorTreeFactory`에 모든 커스텀 노드를 등록한 후, `BT::writeTreeNodesModelXML(factory)` 함수를 호출하여 XML 형식의 노드 목록을 생성할 수 있다.26 이 파일을 Groot2에 임포트하면, 에디터의 팔레트에 커스텀 노드들이 나타나 드래그-앤-드롭으로 사용할 수 있게 된다.26

- **실시간 모니터링 및 디버깅**: Groot2의 가장 강력한 기능은 실행 중인 ROS2 애플리케이션에 연결하여 BT의 상태를 실시간으로 모니터링하는 것이다. 이를 위해 애플리케이션 코드에 `BT::Groot2Publisher`를 추가한다. 이 퍼블리셔는 BT의 구조와 각 노드의 상태(`IDLE`, `RUNNING`, `SUCCESS`, `FAILURE`) 변화를 주기적으로 Groot2에 전송한다.26 개발자는 Groot2 화면에서 각 노드의 색상이 실시간으로 변하는 것을 보며(예: 

  `RUNNING`은 노란색, `SUCCESS`는 녹색, `FAILURE`는 빨간색) 로봇의 행동 결정 과정을 직관적으로 추적하고 디버깅할 수 있다. 이는 단순히 텍스트 로그를 분석하는 것과는 차원이 다른 디버깅 경험을 제공한다.


`BehaviorTree.CPP`는 트리 내 노드들의 상태 변화를 구독할 수 있는 로거(Logger) 메커니즘을 제공한다.12

`nav2_behavior_tree` 패키지는 이를 활용한 `RosTopicLogger`라는 유용한 클래스를 제공한다.31

`RosTopicLogger`는 실행 중인 BT에 연결되어, 모든 노드의 상태가 변경될 때마다 타임스탬프, 노드 이름, 이전 상태, 현재 상태 등의 정보를 담은 `nav2_msgs::msg::BehaviorTreeLog` 메시지를 ROS2 토픽으로 발행한다.31

이 기능은 다음과 같은 다양한 활용 가능성을 열어준다:

- **프로그래밍 방식의 인트로스펙션**: 다른 ROS 노드에서 이 로그 토픽을 구독하여 BT의 상태를 프로그래밍 방식으로 감시하고 특정 상황에 대응할 수 있다.
- **사후 분석**: `ros2 bag`을 사용하여 로그 메시지를 기록해두면, 오프라인에서 시뮬레이션이나 실제 실험 중 발생한 문제의 원인을 정밀하게 분석할 수 있다.
- **커스텀 대시보드**: RQt이나 다른 시각화 도구를 사용하여 이 로그 데이터를 기반으로 한 커스텀 모니터링 대시보드를 구축할 수 있다.

Groot2가 실시간 시각적 디버깅에 강점이 있다면, `RosTopicLogger`는 장기적인 데이터 로깅과 자동화된 분석에 강점이 있다. 이 두 도구를 함께 사용하면 BT 기반 시스템의 개발 및 유지보수 효율을 극대화할 수 있다.


이 보고서는 ROS2 Humble 생태계 내에서 `BehaviorTree.ROS2` 패키지의 기술적 측면을 심층적으로 분석했다. 분석 결과, 행동 트리는 단순한 대안 기술을 넘어 복잡한 자율 로봇 시스템의 제어 아키텍처를 위한 현대적 표준으로 자리매김하고 있음을 확인했다.

**핵심 요약:**

- **설계 철학의 우수성**: BT는 '상태 폭발' 문제를 겪는 FSM과 달리, '행동 중심'의 모듈식, 계층적 접근법을 통해 복잡성을 효과적으로 관리하고 시스템의 확장성과 유지보수성을 크게 향상시킨다.
- **ROS2와의 완벽한 통합**: `BehaviorTree.ROS2` 패키지, 특히 `RosActionNode` 래퍼는 ROS2의 비동기 통신 모델과 BT의 틱 기반 실행 모델 간의 간극을 성공적으로 메워, 개발자가 반응성을 희생하지 않고도 복잡한 ROS2 기반 행동을 손쉽게 구현할 수 있도록 지원한다.
- **검증된 실용성**: Nav2 내비게이션 스택은 BT가 실제 로봇 애플리케이션에서 얼마나 강력하고 강건한 시스템을 구축하는 데 기여할 수 있는지를 명백히 증명하는 사례다. `RecoveryNode`와 같은 패턴은 정교한 오류 처리 로직을 우아하게 캡슐화한다.
- **성숙한 개발 생태계**: Groot2와 같은 시각적 IDE와 `RosTopicLogger`와 같은 디버깅 도구는 개발 경험을 획기적으로 개선하고, BT 기반 시스템의 설계, 테스트, 유지보수 전 과정의 효율성을 높인다.

**전문가 제언:**

1. **기본 아키텍처로 채택**: ROS2 Humble 이상의 버전에서 새롭고 복잡한 로봇 프로젝트를 시작한다면, 고수준 작업 조율을 위한 기본 아키텍처로 `BehaviorTree.ROS2`를 채택할 것을 강력히 권장한다. FSM 대비 초기 학습 곡선이 존재하지만, 확장성, 반응성, 유지보수성 측면에서 얻는 장기적인 이점이 훨씬 크다.
2. **Nav2 생태계 적극 활용**: 바퀴를 재발명하지 마라. 특히 내비게이션과 관련된 작업을 구현할 때는 Nav2의 `nav2_behavior_tree` 패키지가 제공하는 풍부하고 검증된 노드와 설계 패턴을 적극적으로 활용하고, 이를 기반으로 필요한 부분을 커스터마이징하는 것이 효율적이다.
3. **전체 도구 체인 도입**: 개발 초기 단계부터 Groot2를 사용하여 BT를 시각적으로 설계하고 디버깅하는 프로세스를 정착시켜라. 또한, `RosTopicLogger`와 같은 로깅 메커니즘을 시스템에 통합하여 장기적이고 견고한 모니터링 및 분석 체계를 구축해야 한다.

**미래 전망:**

행동 트리는 미래 로보틱스의 핵심 기반 기술로서, 점점 더 복잡하고 지능적이며 강건한 자율 시스템을 구현하는 데 필수적인 역할을 할 것이다. `BehaviorTree.CPP`와 `BehaviorTree.ROS2`를 둘러싼 활발한 개발과 강력한 커뮤니티 지원은 이 기술이 ROS 생태계 내에서 장기적인 생명력과 중요성을 가질 것임을 분명히 보여준다. 따라서 이 기술에 대한 깊이 있는 이해와 숙달은 모든 로보틱스 전문가에게 필수적인 역량이 될 것이다.


1. Behavior tree - Taobotics Robot, accessed July 23, 2025, https://docs.taobotics.com/docs/handsfree/handsfree-en/Tutorial/Advanced/Behavior-Tree/doc.html
2. State Machines vs Behavior Trees ... - Polymath Robotics Blog, accessed July 23, 2025, https://www.polymathrobotics.com/blog/state-machines-vs-behavior-trees
3. Behavior Trees and State Machines in Robotics Applications - Einar Broch Johnsen, accessed July 23, 2025, https://ebjohnsen.org/publication/23-tse/23-tse.pdf
4. BehaviorTree.CPP, accessed July 23, 2025, https://www.behaviortree.dev/
5. SMACC vs Behavior Trees, accessed July 23, 2025, https://smacc.dev/smacc-vs-behavior-trees/
6. Introduction to Behavior Trees - Robotic Sea Bass, accessed July 23, 2025, https://roboticseabass.com/2021/05/08/introduction-to-behavior-trees/
7. Behavior Trees in Robotics (Part 1 - Concept) - YouTube, accessed July 23, 2025, https://www.youtube.com/watch?v=kRp3eA09JkM
8. Behavior Trees | MoveIt Pro Docs, accessed July 23, 2025, https://docs.picknik.ai/7/concepts/behavior_trees/
9. nav2_behavior_tree: Humble 1.1.18 documentation - the official ROS docs, accessed July 23, 2025, https://docs.ros.org/en/humble/p/nav2_behavior_tree/
10. Nav2 Behavior Trees - Nav2 1.0.0 documentation, accessed July 23, 2025, https://docs.nav2.org/behavior_trees/index.html
11. Behavior Trees in Robotics (Part 2 - C++ Implementation) - YouTube, accessed July 23, 2025, https://www.youtube.com/watch?v=4PUiDmD5dkg
12. Your first Behavior Tree | BehaviorTree.CPP, accessed July 23, 2025, https://www.behaviortree.dev/docs/tutorial-basics/tutorial_01_first_tree/
13. 5 minute Behavior Tree tutorial - YouTube, accessed July 23, 2025, https://www.youtube.com/watch?v=KeShMInMjro
14. Blackboard and ports - BehaviorTree.CPP, accessed July 23, 2025, https://www.behaviortree.dev/docs/tutorial-basics/tutorial_02_basic_ports/
15. behaviortree_cpp: BT::TreeNode Class Reference - the official ROS docs, accessed July 23, 2025, http://docs.ros.org/indigo/api/behaviortree_cpp_v3/html/classBT_1_1TreeNode.html
16. Looking for AI & Behaviour Tree C++ Documentation or Sample Code!!! Lack of NON Blueprint Docs!!! - Unreal Engine Forums, accessed July 23, 2025, https://forums.unrealengine.com/t/looking-for-ai-behaviour-tree-c-documentation-or-sample-code-lack-of-non-blueprint-docs/56271
17. BehaviorTree.ROS2/README.md at humble / BehaviorTree/BehaviorTree.ROS2 / GitHub, accessed July 23, 2025, https://github.com/BehaviorTree/BehaviorTree.ROS2/blob/humble/README.md
18. Integration with ROS2 | BehaviorTree.CPP, accessed July 23, 2025, https://www.behaviortree.dev/docs/ros2_integration/
19. as2_behavior_tree: Humble 1.1.2 documentation - the official ROS docs, accessed July 23, 2025, https://docs.ros.org/en/humble/p/as2_behavior_tree/
20. nav2_behavior_tree: Jazzy 1.3.7 documentation - the official ROS docs, accessed July 23, 2025, https://docs.ros.org/en/jazzy/p/nav2_behavior_tree/
21. Writing a New Behavior Tree Plugin - Nav2 1.0.0 documentation, accessed July 23, 2025, https://docs.nav2.org/plugin_tutorials/docs/writing_new_bt_plugin.html
22. Finite State Machines are dead. Long life Behavior Trees by DAVIDE FACONTI - YouTube, accessed July 23, 2025, https://www.youtube.com/watch?v=22KUPktetzg
23. Getting Started with Custom Behavior Tree Plugins in Nav2: Seeking Guidance and Resources : r/ROS - Reddit, accessed July 23, 2025, https://www.reddit.com/r/ROS/comments/1ii9qso/getting_started_with_custom_behavior_tree_plugins/
24. Nav2 - How to create a custom behavior tree (BT) plugin/node in a separate ros2 package?, accessed July 23, 2025, https://robotics.stackexchange.com/questions/107553/nav2-how-to-create-a-custom-behavior-tree-bt-plugin-node-in-a-separate-ros2
25. Behavior tree plugin naming convention in .yaml config file - Robotics Stack Exchange, accessed July 23, 2025, https://robotics.stackexchange.com/questions/103219/behavior-tree-plugin-naming-convention-in-yaml-config-file
26. tutorial_11_groot2 - BehaviorTree.CPP, accessed July 23, 2025, https://www.behaviortree.dev/docs/tutorial-basics/tutorial_11_groot2/
27. Groot Tutorials - Nav2 1.0.0 documentation, accessed July 23, 2025, https://docs.nav2.org/tutorials/docs/using_groot.html
28. Porting SMACH to ROS2 / Issue #70 / ros/executive_smach - GitHub, accessed July 23, 2025, https://github.com/ros/executive_smach/issues/70
29. ROS Package: behaviortree_cpp, accessed July 23, 2025, https://index.ros.org/p/behaviortree_cpp/
30. BehaviorTree.CPP and Groot Version 3.0 released - Open Robotics Discourse, accessed July 23, 2025, https://discourse.openrobotics.org/t/behaviortree-cpp-and-groot-version-3-0-released/8068
31. Class RosTopicLogger - nav2_behavior_tree: Humble 1.1.18 documentation, accessed July 23, 2025, https://docs.ros.org/en/humble/p/nav2_behavior_tree/generated/classnav2__behavior__tree_1_1RosTopicLogger.html

