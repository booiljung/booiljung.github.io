# PyTorch TorchScript에 대한 심층 고찰
[Pytorch](./index.md)



PyTorch의 초기 성공과 폭넓은 채택은 'define-by-run' 철학에 기반한 동적 계산 그래프, 즉 Eager Mode의 직관성과 유연성에서 비롯되었다.1 Eager Mode에서는 연산이 Python 코드에서 호출되는 즉시 실행된다.3 이러한 방식은 모델의 중간 상태를 쉽게 확인하고, `pdb`와 같은 표준 Python 디버깅 도구를 그대로 활용할 수 있게 하여 연구 및 프로토타이핑 과정에서의 개발자 경험을 극대화했다.5 연구자들은 복잡한 제어 흐름을 가진 모델을 Python의 모든 표현력을 동원하여 자유롭게 구현할 수 있었다.

그러나 이러한 유연성은 프로덕션 환경에서 명백한 한계를 드러냈다. 각 PyTorch 연산이 Python 인터프리터를 거쳐 실행되면서 상당한 오버헤드가 발생했고, 이는 모델의 추론 속도를 저하하는 주요 원인이 되었다.1 특히 Python의 전역 인터프리터 잠금(Global Interpreter Lock, GIL)은 다중 스레드 환경에서 진정한 병렬 처리를 방해하여, 여러 요청을 동시에 처리해야 하는 추론 서버의 확장성을 심각하게 저해했다.2 또한, 모델 실행이 Python 런타임에 강하게 종속되어 있어 C++, Java 등 다른 언어로 구축된 프로덕션 백엔드나, Python 환경을 갖추기 어려운 모바일 및 임베디드 시스템에 모델을 통합하는 데 큰 장벽으로 작용했다.8


프로덕션 환경은 연구 환경과는 다른 엄격한 요구사항을 가진다. 낮은 지연 시간(low latency), 높은 처리량(high throughput), 그리고 다양한 하드웨어 및 소프트웨어 플랫폼에서의 이식성(portability)이 핵심이다.2 이러한 요구를 충족시키기 위해 'define-and-run' 방식의 정적 계산 그래프, 즉 Graph Mode가 필수적이다. Graph Mode에서는 모델의 전체 연산 흐름을 사전에 하나의 완전한 그래프로 정의하고 컴파일한다.1

이처럼 모델의 전체 구조를 한눈에 파악할 수 있게 되면, 컴파일러 수준의 강력한 최적화가 가능해진다. 대표적인 예가 연산자 융합(operator fusion)으로, 여러 개의 개별 연산을 하나의 최적화된 커널로 통합하여 메모리 접근과 커널 실행 오버헤드를 줄인다.3 또한, 불필요한 연산을 제거하거나 메모리 할당을 최적화하는 등 전역적인 최적화를 통해 Eager Mode에서는 달성할 수 없는 수준의 성능 향상을 이끌어낼 수 있다. 특히 GPU와 같은 하드웨어 가속기의 연산 처리 속도가 메모리 접근 속도보다 훨씬 빠르게 발전하면서, Eager Mode의 인터프리터 오버헤드와 잦은 메모리 통신은 더욱 두드러진 병목이 되었고, 이는 Graph Mode의 필요성을 더욱 증대시켰다.6


TorchScript는 PyTorch의 핵심 철학인 유연성과 Pythonic함을 유지하면서 프로덕션 환경의 성능 및 이식성 요구를 만족시키기 위해 탄생한 기술이다.1 이는 PyTorch의 Eager Mode에서 작성된 동적 모델을 프로덕션에 적합한 정적 그래프 형태로 변환하는 교량 역할을 수행한다.

구체적으로 TorchScript는 PyTorch 모델의 중간 표현(Intermediate Representation, IR)으로, JIT(Just-In-Time) 컴파일러에 의해 최적화된다.15 이 IR은 Python 런타임에 대한 의존성 없이 직렬화(serialize)될 수 있으며, C++와 같은 고성능 환경에서 독립적으로 로드하고 실행할 수 있다.10 결과적으로 TorchScript는 개발자가 연구 단계에서는 Python의 모든 유연성을 누리고, 배포 단계에서는 Graph Mode의 성능 이점을 취할 수 있도록 하는, "연구에서 프로덕션까지(from research to production)"라는 통합 프레임워크를 지향하는 PyTorch의 핵심 전략적 도구였다.14

TorchScript의 등장은 PyTorch의 정체성과 프로덕션 배포의 현실적 요구 사이의 근본적인 긴장을 해결하려는 첫 번째 대규모 시도였다. PyTorch의 매력은 TensorFlow의 정적 그래프(define-and-run)와 대조되는 동적 그래프(define-by-run)에 있었다.8 그러나 프로덕션 환경에서는 TensorFlow가 가졌던 정적 그래프의 이점(최적화, 이식성)이 절실했다.6 PyTorch는 자신의 정체성을 잃지 않으면서 이 문제를 해결해야 했다. 즉, 'TensorFlow처럼 되는 것'이 아니라, 'Python 코드를 분석하여 TensorFlow와 같은 이점을 얻는 것'이라는 독자적인 접근법을 취했다. TorchScript는 바로 이 접근법의 구체적인 결과물이다. 

`torch.jit.script`는 Python 소스 코드를 직접 분석하고, `torch.jit.trace`는 실제 실행을 관찰하여 그래프를 추출한다. 이는 개발 경험은 Python에 가깝게 유지하면서, 실행 시점에는 정적 그래프의 이점을 누리려는 철학적 타협의 산물이었다. 이 역사적 맥락을 이해하는 것은 TorchScript의 역할과 한계를 정확히 파악하는 데 매우 중요하다.


TorchScript는 PyTorch 모델을 중간 표현(IR)으로 변환하기 위해 두 가지 상호 보완적인 메커니즘을 제공한다: 트레이싱(`torch.jit.trace`)과 스크립팅(`torch.jit.script`). 이 두 방식은 작동 원리와 장단점이 명확하여, 모델의 특성에 따라 적절한 방법을 선택하거나 조합하여 사용해야 한다.


트레이싱은 TorchScript 변환을 위한 가장 직관적이고 간단한 방법이다.


`torch.jit.trace`는 사용자가 제공한 예제 입력(example inputs)을 가지고 모델의 `forward` 함수를 실제로 한 번 실행시킨다. 그리고 이 실행 과정에서 발생하는 모든 텐서 연산들을 순서대로 기록(record)하여 정적인 계산 그래프를 생성한다.7 이 방식은 코드의 논리적 구조를 분석하는 것이 아니라, 특정 입력에 대한 '실행 결과'를 사진 찍듯이 캡처하는 방식에 가깝다.


트레이싱의 가장 큰 장점은 **단순성**이다. 기존 `nn.Module` 코드를 거의 또는 전혀 변경할 필요 없이, 모델 객체와 예제 텐서만 함수에 전달하면 되기 때문에 사용이 매우 간편하다.7 또한, 텐서 연산만을 추적하므로 `forward` 함수 내에 TorchScript가 지원하지 않는 임의의 Python 코드(예: 로깅, 외부 라이브러리 호출)가 포함되어 있어도 변환 자체는 성공할 수 있다.5

그러나 트레이싱에는 치명적인 단점이 존재하는데, 바로 **제어 흐름(Control Flow)의 소실**이다. 트레이싱은 '실행된 경로'만을 기록하기 때문에, 입력 데이터 값에 따라 실행 경로가 달라지는 `if-else` 문과 같은 분기문은 예제 입력이 통과한 한쪽 경로만 그래프에 포함시키고 나머지 경로는 완전히 무시한다.5 마찬가지로, `for`나 `while` 같은 반복문은 예제 입력의 길이나 조건에 맞춰 고정된 횟수만큼 연산이 펼쳐져(unrolled) 기록된다. 이로 인해 트레이싱된 모델은 예제 입력과 다른 특성을 가진 입력에 대해 의도치 않게 동작하거나 오류를 발생시키는 '일반화 실패' 문제를 겪게 된다.5


이러한 특성 때문에 트레이싱은 ResNet, VGG와 같이 입력 데이터의 형태나 값에 관계없이 항상 동일한 연산 그래프 구조를 가지는 정적인 모델에 매우 적합하다.21

```Python
import torch
import torch.nn as nn

class MyModelWithControlFlow(nn.Module):
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        if x.sum() > 10:
            return x * 2
        else:
            return x / 2

model = MyModelWithControlFlow()

# 입력 텐서의 합이 10보다 크므로 (5.0 * 4 = 20 > 10), 'if' 분기만 기록된다.
input_tensor_if = torch.full((2, 2), 5.0)
traced_model_if = torch.jit.trace(model, input_tensor_if)
print("--- Traced with input sum > 10 ---")
print(traced_model_if.code)
# 출력:
# def forward(self, x: Tensor) -> Tensor:
#   return torch.mul(x, 2)

# 입력 텐서의 합이 10보다 작으므로 (1.0 * 4 = 4 < 10), 'else' 분기만 기록된다.
input_tensor_else = torch.full((2, 2), 1.0)
traced_model_else = torch.jit.trace(model, input_tensor_else)
print("\n--- Traced with input sum <= 10 ---")
print(traced_model_else.code)
# 출력:
# def forward(self, x: Tensor) -> Tensor:
#   return torch.div(x, 2)
```

위 예제에서 볼 수 있듯이, `torch.jit.trace`는 `else` 분기의 존재 자체를 알지 못하고 오직 실행된 경로만을 기록한다.7


스크립팅은 트레이싱의 제어 흐름 문제를 해결하기 위해 도입된 더 강력한 변환 방식이다.


`torch.jit.script`는 JIT 컴파일러를 사용하여 Python 소스 코드를 직접 파싱하고, 추상 구문 트리(Abstract Syntax Tree, AST)를 분석하여 이를 TorchScript IR로 변환한다.7 이 방식은 코드의 모든 논리 구조, 즉 모든 조건 분기와 반복문을 그대로 보존하여 모델의 동적인 동작을 정확하게 표현한다.


스크립팅의 최대 장점은 **동적 제어 흐름을 완벽하게 보존**한다는 것이다. 데이터 값에 의존하는 `if-else` 문과 가변 길이의 `for` 루프를 모두 지원하므로, 모델의 동작이 입력에 따라 동적으로 변하는 경우에도 정확하게 동작한다.7 따라서 특정 예제 입력에 종속되지 않고 다양한 입력에 대해 강건하게 동작하는 

**일반화된 모델**을 생성할 수 있다.20

하지만 이러한 강력함에는 대가가 따른다. 스크립팅은 **제한된 Python 구문만을 지원**한다는 한계를 가진다. TorchScript는 Python의 모든 기능을 지원하는 것이 아니라, 딥러닝 모델 기술에 필수적인 기능들을 모아놓은 제한된 부분집합(subset)이다.5 따라서 지원되지 않는 Python 구문(예: `*args`, `**kwargs` 가변 인자, 일부 고급 리스트 컴프리헨션, 특정 내장 함수 등)을 사용하면 컴파일 에러가 발생한다.25 이로 인해 기존의 Python 코드를 TorchScript가 이해할 수 있도록 수정하거나 리팩토링해야 하는 경우가 빈번하게 발생한다.20


스크립팅은 RNN, LSTM, 그리고 어텐션 메커니즘을 사용하는 Transformer 모델과 같이 시퀀스 길이에 따라 반복 횟수가 달라지거나, Beam Search와 같이 복잡한 제어 로직이 포함된 모델에 필수적이다.10

```Python
# 위의 MyModelWithControlFlow 클래스를 그대로 사용
scripted_model = torch.jit.script(model)
print("--- Scripted Model ---")
print(scripted_model.code)
# 출력:
# def forward(self, x: Tensor) -> Tensor:
#   if torch.gt(torch.sum(x), 10):
#     _0 = torch.mul(x, 2)
#   else:
#     _0 = torch.div(x, 2)
#   return _0
```

`torch.jit.script`는 `if-else` 구문을 모두 포함하는 완전한 코드를 생성하여, 어떤 입력이 들어와도 모델이 올바르게 동작하도록 보장한다.7


실제 복잡한 모델에서는 일부 서브모듈은 정적인 구조를 가지고, 다른 일부는 동적인 제어 흐름을 가질 수 있다. TorchScript는 이러한 상황을 위해 두 방식을 유연하게 조합하여 사용할 수 있는 기능을 제공한다.7

예를 들어, `for` 루프를 포함하여 전체적인 흐름이 동적인 상위 모듈은 `torch.jit.script`로 변환하고, 그 안에서 호출되는 간단하고 정적인 CNN 블록과 같은 하위 모듈은 `torch.jit.trace`로 변환하여 포함시킬 수 있다. 반대로, 정적인 모델 내에서 특정 부분만 동적인 로직이 필요할 경우, 해당 부분만 `script` 모듈로 만들어 호출하는 것도 가능하다.10 이러한 하이브리드 접근법은 각 모듈의 특성에 가장 적합한 변환 방법을 적용함으로써 개발 편의성과 모델의 표현력을 모두 확보하는 효과적인 전략이다.1

트레이싱의 단순함과 스크립팅의 표현력 사이의 이러한 간극은 "Trace then Generalize"라는 실용적인 개발 패턴을 낳았다. 개발자는 보통 가장 쉬운 방법인 `trace`로 변환을 시작한다.20 단순한 CNN 모델의 경우 이것만으로 충분할 수 있다. 그러나 모델이 복잡해지거나, `training` 모드와 `eval` 모드에서 다르게 동작하는 `Dropout`이나 `BatchNorm`과 같은 레이어가 포함되면, 트레이스된 모델은 특정 모드에 고정되어 버리는 '일반화 문제'에 직면한다.5 이 문제를 해결하기 위해 개발자는 문제가 되는 부분, 즉 제어 흐름이 있거나 모드에 따라 동작이 바뀌는 서브모듈만 `script`로 변환하여 점진적으로 수정해 나간다. 하지만 이 패턴은 장기적으로 숨겨진 비용을 초래한다. 스크립팅의 엄격한 구문 제약 때문에, 개발자는 종종 깔끔하고 Pythonic한 코드를 포기하고 컴파일러가 이해할 수 있는 더 장황하고 비직관적인 코드를 작성해야 한다.20 예를 들어, `ModuleList`를 일반적인 `for i in range(...)` 루프로 순회하는 코드는 스크립팅이 불가능하여 `enumerate`나 이터레이터 방식으로 변경해야 한다.24 이러한 "TorchScript 친화적인" 코드가 쌓이면서 원본 연구 코드와의 괴리가 발생하고, 가독성과 유지보수성이 저하되어 결국 기술 부채로 남게 된다.20 이 경험은 TorchScript의 근본적인 한계를 드러내며, PyTorch가 `torch.compile`과 같은 더 나은 그래프 획득 방식으로 나아가게 된 핵심적인 동기가 되었다.

다음 표는 `torch.jit.trace`와 `torch.jit.script`의 핵심적인 차이점을 요약한다.

| 특징 (Feature)     | `torch.jit.trace` (트레이싱)                                | `torch.jit.script` (스크립팅)                      |
| ------------------ | ----------------------------------------------------------- | -------------------------------------------------- |
| **작동 원리**      | 예제 입력으로 모델을 실행하고 연산 흐름을 **기록** 7        | Python 소스 코드를 정적으로 **분석**하고 컴파일 23 |
| **제어 흐름 처리** | 데이터 의존적 분기(`if-else`) 및 루프(`for`) **소실** 5     | 제어 흐름을 완벽하게 **보존** 7                    |
| **입력 의존성**    | 예제 입력의 shape, type 등에 **강하게 의존** 20             | 입력에 독립적이며 **일반화** 가능 20               |
| **Python 호환성**  | 거의 모든 Python 코드 내에서 실행 가능 (텐서 연산만 기록) 5 | 제한된 Python **부분집합(subset)**만 지원 5        |
| **장점**           | 사용이 매우 간편하고, 기존 코드 수정 최소화                 | 동적 모델을 정확하게 표현, 높은 일반성             |
| **단점**           | 동적 모델에 부적합, 일반화 실패 위험                        | 학습 곡선 존재, 코드 리팩토링 필요 가능성          |
| **주요 사용 사례** | CNN 등 정적 그래프 구조를 가진 모델 21                      | RNN, Transformer, 제어 로직 포함 모델 10           |


TorchScript의 진정한 힘은 단순히 Python 코드를 다른 형식으로 변환하는 데 있는 것이 아니라, 변환된 중간 표현(IR)을 기반으로 JIT 컴파일러가 수행하는 강력한 최적화에 있다. 이 최적화는 Eager Mode에서는 불가능했던 성능 향상을 가능하게 한다.


TorchScript는 계산 그래프를 표현하기 위해 정적 단일 할당(Static Single Assignment, SSA) 형식의 중간 표현(IR)을 사용한다.10 SSA 형식은 모든 변수가 단 한 번만 할당되는 것을 보장하는 형태로, 컴파일러가 데이터 흐름을 분석하고 최적화를 적용하기에 매우 효율적인 구조다.

이 IR은 두 가지 주요 구성 요소로 이루어져 있다 10:

1. **ATen 연산자**: `aten::add`, `aten::matmul`, `aten::conv2d`와 같이 PyTorch의 핵심 C++ 백엔드인 'ATen' 라이브러리에 정의된 기본 연산자들이다. 실제 수치 계산은 이 연산자들을 통해 수행된다.
2. **기본(Primitive) 연산자**: `prim::Loop`, `prim::If`, `prim::Constant`와 같이 제어 흐름이나 상수 정의 등 계산 그래프의 구조를 형성하는 연산자들이다.

TorchScript로 변환된 모듈 객체의 `.graph` 속성을 통해 이 저수준 IR을 직접 확인할 수 있다. 각 명령어 라인은 일반적으로 `%출력변수.고유번호 : 타입 = 네임스페이스::연산자(%입력변수들) # 원본_소스.py:줄:칸` 과 같은 형식을 따른다. 이 구조는 각 연산의 입출력 관계와 데이터 타입을 명확히 보여주며, 원본 소스 코드 위치 정보까지 포함하여 디버깅을 용이하게 한다.7 반면, `.code` 속성은 이 저수준 IR을 사람이 더 이해하기 쉬운 Python 유사 구문으로 재해석하여 보여준다.7


JIT 컴파일러가 수행하는 가장 중요하고 효과적인 최적화 중 하나는 연산자 융합(Operator Fusion)이다.


연산자 융합은 계산 그래프 상에서 연속적으로 나타나는 여러 개의 개별 연산자(커널)를 하나의 더 큰 연산자(융합된 커널)로 병합하는 최적화 기법이다.12 딥러닝 모델, 특히 GPU에서 실행될 때의 성능은 순수한 연산 속도(FLOPs)보다 메모리 대역폭에 의해 제한되는 경우가 많다. 연산자 융합은 바로 이 문제를 정면으로 해결한다.

1. **메모리 접근 오버헤드 감소**: `Convolution -> BatchNorm -> ReLU`와 같은 일련의 연산을 생각해보자. 융합이 없다면, 각 연산은 GPU의 전역 메모리(DRAM)에서 입력을 읽고, 계산을 수행한 뒤, 중간 결과를 다시 전역 메모리에 쓰는 과정을 반복한다. DRAM 접근은 매우 비용이 높은 작업이다. 연산자 융합은 이러한 중간 결과를 DRAM에 쓰지 않고, GPU 칩 내의 훨씬 빠른 온칩 메모리(SRAM 캐시나 레지스터)에 유지한 채로 다음 연산을 연속적으로 수행한다. 이를 통해 비용이 큰 메모리 읽기/쓰기 횟수를 극적으로 줄여 성능을 향상시킨다.3
2. **커널 실행 오버헤드 감소**: GPU에서 각 연산(커널)을 실행하기 위해서는 CPU가 GPU에 명령을 내리는 과정이 필요하며, 이 '커널 실행(kernel launch)' 자체에도 무시할 수 없는 오버헤드가 발생한다. 3개의 연산을 1개의 융합된 커널로 합치면, 커널 실행 횟수가 1/3로 줄어들어 이 오버헤드 또한 감소한다.3

연산자 융합은 크게 두 가지 유형으로 나눌 수 있다. **수직적 융합(Vertical Fusion)**은 한 연산의 출력이 다음 연산의 입력으로 바로 사용되는 순차적인 연산들을 묶는 것(예: `Conv -> ReLU`)을 말하며, **수평적 융합(Horizontal Fusion)**은 동일한 연산이 여러 다른 입력에 독립적으로 적용될 때, 이들을 하나의 큰 연산으로 묶어 처리하는 것을 의미한다.3 TorchScript의 JIT 컴파일러는 IR 그래프를 분석하여 이러한 융합 가능한 패턴을 자동으로 감지하고, 이를 `prim::FusionGroup`이라는 특수한 노드로 묶어 하드웨어에 최적화된 코드를 생성한다.13


연산자 융합 외에도 JIT 컴파일러는 다양한 표준 컴파일러 최적화 기법을 적용하여 성능을 개선한다 13:

- **상수 전파(Constant Propagation)**: 코드 내에서 변하지 않는 상수 값들을 이용한 계산을 컴파일 시점에 미리 수행하여 런타임의 계산 부담을 줄인다.
- **데드 코드 제거(Dead Code Elimination)**: 최종 출력에 아무런 영향을 미치지 않는 불필요한 연산이나 코드 블록을 그래프에서 완전히 제거한다.
- **공통 부표현식 제거(Common Subexpression Elimination)**: 계산 그래프 내에서 반복적으로 나타나는 동일한 계산을 식별하여, 단 한 번만 계산하고 그 결과를 재사용함으로써 중복 계산을 방지한다.

TorchScript의 진정한 가치는 단순히 Python 의존성을 제거하는 것을 넘어선다. Eager Mode에서는 각 연산이 독립적으로, 순차적으로 실행되기 때문에 프레임워크는 `torch.add`가 실행되는 시점에 다음에 `torch.relu`가 올 것이라는 사실을 알 수 없다.3 따라서 `add`의 결과를 메모리에 쓰고, `relu`가 이 결과를 다시 메모리에서 읽어오는 과정이 불가피하다. 최적화의 범위가 '단일 연산자' 수준에 머무를 수밖에 없다.

반면, TorchScript는 모델 전체를 하나의 완전한 '그래프'로 표현한다.7 JIT 컴파일러는 이 그래프 전체를 조망하며 연산자들 간의 상호 관계를 분석할 수 있다. 이 '전체 그림을 보는 능력' 덕분에 컴파일러는 " `add`의 출력이 오직 `relu`의 입력으로만 사용되고, 둘 다 element-wise 연산이므로, 중간 결과를 DRAM에 저장할 필요 없이 하나의 커널로 합쳐버리자"와 같은 전역적인 최적화(Global Optimization)를 수행할 수 있다.12 이처럼 연산자들의 '상호작용'을 최적화하는 능력이야말로 Eager Mode의 근본적인 한계를 극복하고 하드웨어의 성능을 최대한으로 끌어내는 열쇠이며, TorchScript와 같은 그래프 기반 방식의 핵심적인 성능 우위의 원천이다.


TorchScript의 궁극적인 목표 중 하나는 Python으로 개발된 모델을 Python이 없는 고성능 C++ 환경에 배포하는 것이다. 이 과정은 모델 직렬화, C++ 환경 설정, 그리고 LibTorch API를 사용한 추론 코드 작성의 단계로 이루어진다.


TorchScript로 변환된 모듈은 `.save()` 메소드를 통해 단일 파일로 직렬화(serialization)할 수 있다. 관례적으로 `.pt` 또는 `.ts` 확장자가 사용된다.7 이 직렬화된 파일은 단순한 가중치 저장을 넘어선다. 파일 내부에는 모델의 아키텍처를 정의하는 코드, 학습된 파라미터(가중치와 편향), 그리고 모델에 저장된 버퍼 및 속성 정보가 모두 포함된다. 따라서 이 파일 하나만으로 모델을 완벽하게 복원할 수 있는 **독립적인 아티팩트(self-contained artifact)**가 된다.7

이는 PyTorch에서 일반적으로 사용되는 `torch.save(model.state_dict(), 'model_weights.pth')` 방식과 근본적인 차이를 보인다. `state_dict`는 모델의 상태(주로 가중치)만을 담고 있는 Python 딕셔너리이므로, 이를 로드하기 위해서는 먼저 Python 환경에서 해당 모델의 클래스(`class MyModel(nn.Module):...`)를 정의하고 인스턴스화해야 한다.33 반면, TorchScript 파일은 C++ 환경에서 Python 소스 코드나 클래스 정의 없이 `torch::jit::load()` 함수 하나만으로 즉시 로드하여 사용할 수 있다.21


직렬화된 TorchScript 모델을 C++에서 사용하기 위해서는 LibTorch가 필요하다. LibTorch는 PyTorch의 C++ 배포판으로, 모델을 로드하고 실행하는 데 필요한 모든 C++ 헤더 파일(`.h`, `.hpp`)과 공유 라이브러리 파일(`.so`, `.dll`, `.dylib`)을 포함하고 있다.22 PyTorch 공식 웹사이트에서 자신의 운영체제와 CUDA 버전에 맞는 LibTorch 배포판을 다운로드하여 압축을 해제하면 된다.37

C++ 프로젝트에서 LibTorch를 연동하는 가장 표준적이고 권장되는 방식은 빌드 시스템인 CMake를 사용하는 것이다.21 프로젝트 루트에 `CMakeLists.txt` 파일을 생성하고 다음과 같이 기본적인 내용을 구성한다:

1. **CMake 최소 버전 및 프로젝트 이름 정의**: `cmake_minimum_required(VERSION 3.0 FATAL_ERROR)`와 `project(my_inference_app)`를 명시한다.
2. **LibTorch 패키지 찾기**: `find_package(Torch REQUIRED)` 명령어를 사용한다. CMake가 LibTorch를 찾을 수 있도록, `cmake` 실행 시 `-DCMAKE_PREFIX_PATH=/path/to/libtorch` 옵션을 통해 LibTorch의 경로를 지정해주어야 한다.38
3. **실행 파일 생성 및 라이브러리 링크**: `add_executable(my_inference_app main.cpp)`로 실행 파일을 정의하고, `target_link_libraries(my_inference_app "${TORCH_LIBRARIES}")`를 통해 필요한 LibTorch 라이브러리들을 실행 파일에 링크한다.21


LibTorch와 CMake 설정이 완료되면, C++ 애플리케이션에서 모델 추론을 수행하는 코드를 작성할 수 있다. 전체적인 흐름은 다음과 같다.

1. **필수 헤더 포함**: `#include <torch/script.h>`를 통해 LibTorch의 핵심 기능을 가져온다.21

2. **모델 로딩**: `torch::jit::script::Module module;`로 모듈 객체를 선언하고, `try-catch` 블록 내에서 `module = torch::jit::load("path/to/your_model.pt");` 코드를 통해 직렬화된 TorchScript 모델을 C++ 객체로 로드한다.21

3. **디바이스 설정 (선택 사항)**: 기본적으로 모델과 텐서는 CPU에 로드된다. GPU를 사용하려면, `torch::Device device(torch::kCUDA);`로 디바이스를 정의하고 `module.to(device);`를 호출하여 모델을 GPU 메모리로 이동시킨다.21 이 경우, 모델에 입력될 텐서 또한 

   `.to(device)`를 통해 동일한 GPU로 이동시켜야 데이터 타입 불일치로 인한 런타임 에러를 방지할 수 있다.41

4. **입력 텐서 생성**: LibTorch는 `torch::randn({1, 3, 224, 224})`, `torch::ones(...)` 등 PyTorch Python API와 매우 유사한 C++ API를 제공하여 입력 텐서를 쉽게 생성할 수 있다.21 실제 애플리케이션에서는 OpenCV와 같은 이미지 처리 라이브러리를 사용하여 이미지를 읽고, 전처리한 후 LibTorch 텐서로 변환하는 과정이 일반적이다.38

5. **추론 실행**: TorchScript 모듈의 `forward` 메소드는 `std::vector<torch::jit::IValue>` 타입의 입력을 받는다. `IValue`는 텐서, 정수, 튜플 등 다양한 타입을 담을 수 있는 wrapper 클래스다. 생성한 입력 텐서를 `std::vector<torch::jit::IValue> inputs; inputs.push_back(input_tensor);`와 같이 벡터에 담은 후, `at::Tensor output = module.forward(inputs).toTensor();`를 호출하여 추론을 실행한다.37

6. **출력 처리**: `forward` 메소드의 반환값은 `IValue`이므로, `.toTensor()` 메소드를 호출하여 `at::Tensor` (LibTorch의 텐서 타입)로 명시적으로 변환해야 한다. 이후, `.slice()`, `.accessor<float, N>()` 등 다양한 메소드를 사용하여 결과 텐서의 데이터에 접근하고 후처리할 수 있다.37


전체 워크플로우는 다음과 같이 요약할 수 있다.

1. **1단계 (Python)**: `torchvision.models.resnet18`과 같은 사전 학습된 모델을 로드한다. `torch.jit.script` 또는 `torch.jit.trace`를 사용하여 모델을 TorchScript 모듈로 변환한다. 마지막으로 `scripted_model.save("resnet18_scripted.pt")`를 호출하여 모델을 파일로 저장한다.21
2. **2단계 (C++)**: 위에서 설명한 대로 `CMakeLists.txt`를 설정한다. C++ 메인 함수에서 `resnet18_scripted.pt` 파일을 로드하고, 임의의 더미 입력 텐서(예: `torch::ones({1, 3, 224, 224})`)를 생성하여 추론을 수행한 뒤, 출력 텐서의 일부를 콘솔에 출력하는 완전한 코드를 작성한다. GitHub의 다양한 예제 리포지토리들이 이 과정을 상세한 코드와 함께 제공하고 있다.22

TorchScript와 LibTorch의 조합이 제공하는 가장 본질적인 가치는 '개발 환경'과 '배포 환경'의 완벽한 분리(decoupling)에 있다. 모델 개발 및 학습은 Python의 풍부한 생태계와 빠른 프로토타이핑 능력 위에서 '유연성'과 '실험 속도'를 중시하며 이루어진다. 반면, 모델 배포 및 서빙은 C++ 기반의 고성능 웹 서버나 임베디드 시스템과 같이 Python이 없거나 사용하기 부적합한 환경에서 '성능', '안정성', '낮은 리소스 사용량'을 목표로 이루어진다.2

이 두 개의 이질적인 세계를 연결하는 것이 바로 TorchScript 파일(`.pt`)이다. 이 파일은 두 팀 간의 명확한 '인터페이스' 또는 '계약(contract)' 역할을 한다. Python으로 모델을 개발하는 ML 엔지니어링 팀은 이 파일만 C++로 시스템을 구축하는 소프트웨어 엔지니어링 팀에게 전달하면 된다. C++ 팀은 모델의 내부 구현이나 학습 과정에 대해 전혀 알 필요 없이, 오직 약속된 입출력 형식에 맞춰 LibTorch API를 사용하여 모델을 로드하고 실행하기만 하면 된다. Python 버전 충돌, 패키지 의존성 지옥과 같은 문제로부터 완전히 자유로워진다.2 이러한 명확한 역할 분리는 각 팀이 자신의 전문 분야에 집중할 수 있게 하여 전체 MLOps 워크플로우를 더욱 모듈화되고 강건하게 만들며, 이것이 TorchScript가 프로덕션 환경에서 핵심적인 기술로 자리 잡았던 근본적인 이유이다.


TorchScript는 강력한 도구이지만, 실제 프로젝트에 적용하다 보면 다양한 제약 사항과 오류에 직면하게 된다. 성공적인 활용을 위해서는 이러한 한계를 명확히 인지하고 효과적인 디버깅 전략을 갖추는 것이 필수적이다.


TorchScript는 Python의 모든 동적 특성을 지원하지 않는, 본질적으로 정적인 타입 시스템을 가진 언어다. 이로 인해 여러 제약이 발생한다.

- **지원되지 않는 Python 기능**: TorchScript 컴파일러는 Python의 모든 구문을 이해하지 못한다. 대표적으로 가변 인자(`*args`, `**kwargs`)를 받는 함수는 직접 변환할 수 없으며, 명시적인 파라미터로 수정해야 한다.26 또한, 리스트 컴프리헨션 내의 

  `if` 조건절, 일부 내장 함수, 그리고 NumPy나 SciPy와 같은 서드파티 라이브러리 호출 등은 직접 지원되지 않는다.25

- **정적 타입 시스템**: TorchScript는 변수의 타입이 코드 실행 중에 변하지 않을 것을 기대한다. 따라서 Python에서는 자연스러운 동적 타이핑 코드가 TorchScript에서는 오류를 유발할 수 있다.14 이를 해결하기 위해 Python의 

  `typing` 모듈을 사용하여 변수의 타입을 명시적으로 어노테이션(`x: torch.Tensor`, `y: List[int]`)하는 것이 매우 중요하며, 때로는 필수적이다. 특히 `None` 값을 가질 수 있는 변수는 `Optional`와 같이 명확히 선언해야 한다.24

- **클래스 및 속성 제약**: `nn.Module` 내에 정의된 속성(attribute) 또한 TorchScript의 타입 시스템을 따라야 한다. 만약 TorchScript가 이해할 수 없는 사용자 정의 클래스 타입을 속성의 타입으로 어노테이션하면, 해당 속성이 `forward` 메소드에서 전혀 사용되지 않더라도 스크립팅 과정에서 컴파일 오류가 발생할 수 있다.25


실제 개발 과정에서 자주 마주치는 대표적인 오류와 그 해결책은 다음과 같다.

- **`ModuleList`/`Sequential` 인덱싱 오류**: `for i in range(len(self.convs)): output = self.convs[i](input)`와 같은 코드는 `RuntimeError: indexing is only supported with integer literals` 오류를 발생시킨다. TorchScript 컴파일러는 루프 변수 `i`의 값을 정적으로 추론할 수 없기 때문이다. 이 문제는 루프를 `for conv_layer in self.convs: output = conv_layer(input)`와 같이 이터레이터를 직접 순회하는 방식으로 변경하거나, `enumerate`를 사용하여 해결할 수 있다.24
- **동적 속성 할당으로 인한 `AttributeError`**: `if self.training: self.dropout = nn.Dropout()`과 같이 특정 조건 하에서만 모듈의 속성을 초기화하면, 해당 조건이 충족되지 않는 실행 경로에서 속성이 존재하지 않아 `AttributeError`가 발생한다. TorchScript는 모든 가능한 실행 경로에 대해 속성이 정의되어 있기를 기대한다. 따라서 `__init__` 메소드에서 `self.dropout = nn.Identity()`나 `self.dropout = None`과 같이 모든 경우에 대해 속성을 미리 초기화해두어야 한다.24
- **`Optional` 타입 처리 미흡**: `Optional`로 선언된 변수를 사용하는 코드 블록에서, 컴파일러는 해당 변수가 `None`일 가능성을 배제할 수 없어 타입 에러를 일으킬 수 있다. 이를 해결하기 위해서는 `if x is not None:`과 같은 명시적인 조건문을 사용하거나, `assert x is not None` 구문을 코드 블록 시작 부분에 추가하여 컴파일러에게 해당 분기 내에서는 변수가 절대 `None`이 아님을 알려주어야 한다.24


TorchScript 코드의 오류를 해결하는 것은 때로 까다로울 수 있지만, 다음과 같은 체계적인 디버깅 기법을 활용하면 효율적으로 문제를 해결할 수 있다.

- **JIT 컴파일 비활성화**: 가장 강력한 디버깅 도구는 JIT 컴파일을 일시적으로 끄는 것이다. 터미널에서 `export PYTORCH_JIT=0` 환경 변수를 설정하고 스크립트를 실행하면, `@torch.jit.script`나 `torch.jit.trace`와 같은 TorchScript 관련 호출이 모두 무시되고 코드가 순수한 Python으로 실행된다. 이를 통해 `pdb`와 같은 표준 Python 디버거를 사용하여 코드의 논리적 오류를 먼저 찾아내고 수정할 수 있다.10

- **`.code` 및 `.graph` 속성 검사**: 컴파일이 성공했지만 예상과 다르게 동작할 경우, 변환된 TorchScript 모듈의 `.code` 속성을 출력하여 컴파일러가 코드를 어떻게 해석했는지 Python 유사 구문으로 확인할 수 있다.7 더 깊은 분석이 필요하다면, 

  `.graph` 속성을 통해 저수준 IR을 직접 검사하여 JIT 컴파일러의 변환 과정을 정밀하게 추적할 수 있다.7

- **점진적 변환 및 문제 국소화**: 전체 대형 모델을 한 번에 스크립팅하려고 시도하기보다는, 가장 작은 단위의 서브모듈부터 점진적으로 변환하며 테스트하는 것이 효과적이다. 오류가 발생하면, 문제가 되는 모듈을 특정하고 해당 모듈 내부에서 문제를 해결하는 방식으로 범위를 좁혀나갈 수 있다.9

TorchScript를 사용할 때 개발자들이 겪는 가장 큰 실용적인 문제는 기술의 '취약성(brittleness)'이다. 이는 문법적으로 사소해 보이는 변경이 예측 불가능한 컴파일 오류를 유발하는 현상을 의미한다.20 예를 들어, 잘 동작하던 모델 코드에 Python 관점에서는 완전히 합법적인 작은 수정(예: 데이터 구조를 튜플에서 

`NamedTuple`로 변경, 간단한 리스트 컴프리헨션 추가)을 가했을 때, 갑자기 TorchScript 컴파일러가 모호한 에러 메시지와 함께 실패하는 경우가 많다.20 이 순간부터 개발자의 목표는 '모델 개선'에서 '컴파일러 만족시키기'로 변질되며, 생산성을 저해하고 종종 가독성이 떨어지는 코드를 양산하게 된다. 이러한 경험이 반복되면 개발자들은 TorchScript 코드 수정을 꺼리게 되고, 프로젝트는 경직되어 "작동은 하지만, 아무도 건드리고 싶어하지 않는" 레거시 코드가 되어버린다. 이 '취약성' 문제는 TorchScript가 Python의 모든 의미를 완벽하게 이해하지 못하는 '제한된 부분집합'이라는 근본적인 한계에서 비롯된다. 

`torch.compile`의 TorchDynamo는 이 문제를 'graph break'라는 훨씬 우아한 방식으로 해결함으로써, 개발자가 더 이상 컴파일러의 눈치를 보지 않고 자유롭게 Python 코드를 작성할 수 있도록 했다. 이는 단순한 기술적 진보를 넘어, 개발자 경험의 패러다임을 바꾼 중요한 변화이다.


TorchScript는 PyTorch 모델을 프로덕션에 배포하기 위한 중요한 도구였지만, 유일한 선택지는 아니다. 모델의 이식성을 위한 ONNX와, PyTorch 2.0에서 도입된 새로운 컴파일러 스택인 `torch.compile`과의 비교를 통해 TorchScript의 위치와 장단점을 더 명확히 이해할 수 있다.


TorchScript와 ONNX는 모두 모델을 배포 가능한 형식으로 변환하는 것을 목표로 하지만, 그 철학과 생태계가 근본적으로 다르다.

- **목표와 철학**:

  - **TorchScript**: PyTorch 생태계 내에서의 완결성에 초점을 맞춘다. PyTorch로 개발된 모델을 C++ 환경으로 가장 자연스럽게 포팅하는 경로를 제공하며, PyTorch의 동적인 특성(예: 제어 흐름)을 최대한 지원하려 노력한다.19
  - **ONNX**: 프레임워크에 구애받지 않는 '모델 교환'을 위한 개방형 표준을 지향한다. PyTorch, TensorFlow, Keras 등 다양한 프레임워크에서 훈련된 모델을 ONNX 형식으로 변환하면, ONNX Runtime, TensorRT, OpenVINO 등 다양한 추론 엔진에서 실행할 수 있는 최고의 상호 운용성(interoperability)을 제공하는 것이 목표다.19

- 성능 비교:

  어느 쪽이 절대적으로 더 빠르다고 단정하기는 어렵다. 성능은 특정 모델 아키텍처, 하드웨어, 그리고 사용하는 런타임에 따라 크게 달라진다. 일반적으로 ONNX는 TensorRT나 OpenVINO와 같은 고도로 최적화된 실행 프로바이더(Execution Provider)와 결합될 때, 순수 LibTorch로 실행하는 TorchScript 모델보다 더 높은 성능을 보이는 경우가 많다.47 하지만 PyTorch에서 ONNX로 모델을 변환하는 과정에서 일부 연산자가 비효율적으로 분해되거나 지원되지 않아 오히려 성능이 저하되는 경우도 보고된다.49

- 유연성 및 기능:

  TorchScript의 가장 큰 장점은 torch.jit.script를 통해 데이터 의존적인 제어 흐름을 강력하게 지원한다는 점이다. 이는 ONNX가 기본적으로 정적인 방향성 비순환 그래프(Directed Acyclic Graph, DAG)를 가정하는 것과 대조된다.47 반면, 

  **ONNX**는 `dynamic_axes` 옵션을 통해 동적인 입력 shape를 지원하는 등 유연성을 확보하려 노력하지만, 설정이 다소 번거로울 수 있다. 과거에는 이 문제로 인해 가변 크기 이미지를 다루기 위해 TorchScript를 선호하는 경우도 있었다.47

- **선택 가이드**:

  - **TorchScript**는 PyTorch 생태계 내에서 작업을 완결하고 싶고, 주된 배포 타겟이 C++(LibTorch)이며, 모델에 복잡한 제어 흐름이 포함된 경우에 가장 직접적이고 자연스러운 선택이다.
  - **ONNX**는 다양한 하드웨어 백엔드(NVIDIA GPU, Intel CPU, ARM 프로세서 등)에 모델을 배포해야 하거나, 다른 프레임워크와의 협업이 필요한 경우, 즉 이식성과 상호 운용성이 최우선 순위일 때 강력한 선택지가 된다.


`torch.compile`은 PyTorch 2.0의 핵심 기능으로, TorchScript의 경험을 바탕으로 한 단계 진보한 새로운 컴파일 패러다임을 제시한다.

- **패러다임의 전환**:

  - **TorchScript**: 모델 전체를 하나의 완전한 정적 그래프로 변환하려는 '전체 프로그램(whole-program)' 컴파일 방식을 따른다. 이로 인해 지원되지 않는 Python 구문을 하나라도 만나면 변환 전체가 실패하는 'all-or-nothing' 접근법에 가깝다.50
  - **`torch.compile`**: JIT 컴파일러인 TorchDynamo를 사용하여, 컴파일 가능한 부분은 그래프로 캡처(graph capture)하고, 동적 제어 흐름이나 지원되지 않는 코드 등 컴파일이 불가능한 부분은 Eager Mode로 실행하도록 남겨두는(graph break) 유연한 하이브리드 방식을 채택한다.50

- 그래프 획득 방식의 혁신:

  TorchScript는 소스 코드를 직접 변환(script)하거나 실행을 추적(trace)하는 방식에 의존하여 불안정하고 여러 제약이 따랐다.51 반면, 

  `torch.compile`의 핵심인 **TorchDynamo**는 CPython의 Frame Evaluation API라는 저수준 기능을 활용하여 Python 바이트코드를 실행 중에 안전하게 후킹(hooking)한다. 이 혁신적인 접근법 덕분에, 기존 Python 코드를 단 한 줄도 변경할 필요 없이 99% 이상의 높은 성공률로 계산 그래프를 획득할 수 있다. 이는 TorchScript의 50% 남짓한 성공률과 비교할 수 없는 엄청난 발전이다.51

- **성능, 유연성, 사용 편의성**:

  - **성능**: `torch.compile`은 TorchInductor라는 기본 백엔드를 통해 OpenAI의 Triton과 같은 최신 커널 생성 기술을 활용한다. 이를 통해 다양한 벤치마크에서 TorchScript를 능가하는 상당한 성능 향상(HuggingFace, TIMM 등 163개 모델 대상 평균 43% 학습 속도 향상)을 보여준다.51
  - **유연성**: 'Graph break' 기능 덕분에 개발자는 임의의 Python 코드를 거의 그대로 사용할 수 있다. 더 이상 TorchScript의 엄격한 구문 제약에 얽매일 필요가 없어져 개발 자유도가 극적으로 향상되었다.52
  - **사용 편의성**: `model = torch.compile(model)` 단 한 줄의 코드를 추가하는 것만으로 적용이 가능하여, 사용자 경험이 획기적으로 개선되었다.51

TorchScript와 `torch.compile`의 가장 근본적인 차이는 사용자에게 요구하는 패러다임의 차이에 있다. TorchScript는 사용자에게 'TorchScript라는 새로운 언어(Python의 제한된 부분집합)의 규칙을 배우고 따를 것'을 요구했다. 개발자는 항상 "이 코드가 TorchScript에서 컴파일될까?"를 고민해야 했으며, 이는 창의성과 생산성을 저해하는 인지적 부담이었다.20 반면, 

`torch.compile`은 '기존의 Python 코드를 그대로 사용하면, 우리가 알아서 가속화해주겠다'는 새로운 패러다임으로 전환했다. 컴파일 가능 여부에 대한 고민을 개발자로부터 컴파일러로 이전시킨 것이다.51 이는 '명시적(explicit) 변환'에서 '암묵적(implicit) 가속화'로의 전환을 의미하며, PyTorch가 프레임워크의 핵심 가치인 'Pythonic'함을 희생하지 않으면서도 Graph Mode의 성능을 제공하는, 두 마리 토끼를 모두 잡는 데 성공했음을 보여준다. TorchScript가 '타협안'이었다면, `torch.compile`은 '통합된 해결책'에 가깝다.

다음 표는 TorchScript와 `torch.compile`의 주요 특징을 비교한다.

| 특징 (Feature)     | TorchScript (`torch.jit`)                        | `torch.compile` (PyTorch 2.0+)                            |
| ------------------ | ------------------------------------------------ | --------------------------------------------------------- |
| **핵심 철학**      | Python 코드를 정적 그래프 IR로 **변환**          | Python 코드를 동적으로 **가속화**                         |
| **그래프 획득**    | `trace` (실행 기록) 또는 `script` (정적 분석) 7  | TorchDynamo (CPython Frame API 후킹) 51                   |
| **Python 호환성**  | 제한된 부분집합만 지원, 코드 수정 필요 20        | Graph Break를 통해 거의 모든 Python 코드 지원 50          |
| **사용 편의성**    | 상대적으로 복잡, `trace`와 `script` 중 선택 필요 | `model = torch.compile(model)` 한 줄로 적용 51            |
| **에러 핸들링**    | All-or-Nothing, 컴파일 실패 시 전체 실패 50      | 컴파일 불가 코드는 Eager로 폴백(fallback) 실행 51         |
| **성능**           | 연산자 융합 등으로 성능 향상, 그러나 한계 존재   | TorchInductor, Triton 등 최신 기술로 더 높은 성능 달성 51 |
| **주된 사용 목적** | **Python 독립적인 배포** (특히 C++/모바일)       | **학습 및 추론 속도 향상** (Python 환경 내에서)           |


TorchScript는 PyTorch의 역사에서 중요한 이정표를 세웠지만, PyTorch 2.0의 등장과 함께 새로운 국면을 맞이했다. TorchScript의 현재 상태, 미래, 그리고 그것이 남긴 유산을 종합적으로 평가하는 것은 PyTorch 생태계의 발전 방향을 이해하는 데 필수적이다.


PyTorch 2.0 이후, TorchScript는 공식적으로 **사용 비권장(Deprecated)** 상태가 되었다.55 PyTorch 공식 문서와 튜토리얼에서는 새로운 C++ 배포 및 직렬화 워크플로우를 위해 `torch.export`를 사용할 것을 명확히 권장하고 있다.

그러나 'Deprecated'가 즉각적인 '제거(Removed)'를 의미하는 것은 아니다. PyTorch 팀은 기존에 TorchScript와 LibTorch를 기반으로 구축된 수많은 프로덕션 시스템의 안정성을 고려하여, "예측 가능한 미래까지(for the foreseeable future)" TorchScript에 대한 지원을 계속할 것이라고 밝혔다.56 이는 활발한 신규 기능 개발은 중단되지만, 중요한 버그 수정 및 유지보수는 지속될 것임을 시사한다. 따라서 기존 시스템을 급하게 마이그레이션할 필요는 없지만, 신규 프로젝트에 TorchScript를 채택하는 것은 지양해야 한다.


PyTorch 2.x 시대에 TorchScript의 역할을 계승할 공식적인 후계자는 `torch.export`이다.55

`torch.export`는 `torch.compile`의 핵심 기술인 TorchDynamo를 활용하여 계산 그래프를 획득한다. 이는 TorchScript의 `trace`나 `script`보다 훨씬 더 안정적이고 광범위한 Python 코드를 캡처할 수 있음을 의미한다.

`torch.export`를 통해 생성된 모델은 ATen IR 기반의 직렬화 가능한 형태로 추출되며, 이는 서버사이드 C++ 추론(LibTorch)이나 모바일/엣지 환경을 위한 경량 런타임인 ExecuTorch에서 사용될 수 있다. 즉, `torch.export`는 TorchScript가 수행했던 'Python 독립적인 배포 모델 생성'이라는 핵심 역할을 더욱 발전된 기술로 대체하는 것이다. 현재 `torch.export`는 아직 불안정(unstable) API로 분류되어 있지만, 점차 안정화되어 향후 C++ 배포의 표준 경로가 될 것으로 예상된다.57


TorchScript는 PyTorch를 단순한 연구용 프레임워크에서 프로덕션급 프레임워크로 격상시킨 일등공신이다. Python 런타임에 대한 의존성 없이 모델을 직렬화하고 C++에서 추론하는 개념을 PyTorch 생태계에 처음 도입하고 정착시킴으로써, PyTorch의 적용 범위를 비약적으로 확장시켰다.

동시에, TorchScript의 한계와 그로 인한 개발자들의 고충은 PyTorch 컴파일러 기술이 나아갈 방향에 대한 중요한 교훈을 남겼다. **'그래프 획득의 어려움'**과 **'Python 호환성 유지의 중요성'**이라는 두 가지 큰 교훈은 TorchDynamo와 `torch.compile`이 개발되는 직접적인 동기가 되었다. TorchScript의 불안정성과 사용성의 한계를 극복하려는 노력이 역설적으로 PyTorch 2.0의 혁신을 이끌어낸 것이다. 즉, TorchScript의 실패 경험은 PyTorch 2.0 성공의 귀중한 밑거름이 되었다.

PyTorch 2.0의 진정한 혁신은 TorchScript라는 거대한 단일 솔루션(monolithic solution)을 '언번들링(unbundling)'하여, 각각의 기능을 더 작고 모듈화된 전문 도구들로 분리한 데 있다. TorchScript는 그래프 획득, IR 정의, 최적화, 직렬화, C++ 로딩 등 너무 많은 역할을 한 번에 수행하려 했고, 이로 인해 시스템은 복잡하고 경직되었다.58 PyTorch 2.0은 이 기능들을 ▲그래프 획득(TorchDynamo) ▲프로그램 변환(FX) ▲자동 미분(AOTAutograd) ▲최적화 및 코드 생성(TorchInductor) ▲직렬화 및 배포(`torch.export`) 등으로 명확히 분리했다. 이러한 모듈화는 마치 마이크로서비스 아키텍처가 모놀리식 애플리케이션을 대체하는 것처럼, 각 구성 요소가 독립적으로 발전하고 교체될 수 있는 유연성과 확장성을 제공한다. 이는 향후 머신러닝 컴파일러가 나아갈 방향을 제시하며, TorchScript가 남긴 가장 중요한 기술적 유산이라 할 수 있다.


향후 TorchScript의 역할은 매우 제한적일 것이다. 신규 프로젝트에서는 `torch.compile`을 통한 학습/추론 가속화와 `torch.export`를 통한 배포가 표준이 될 것이다. 다만, `torch.export`가 완전히 안정화되기 전까지의 과도기나, 기존에 구축된 방대한 TorchScript 기반의 레거시 시스템을 유지보수하는 데에는 여전히 필요할 것이다. 또한, 모바일 및 엣지 디바이스 환경을 위한 솔루션으로 그 초점이 전환될 것이라는 논의가 있었으나 58, 이 영역 역시 `torch.export`와 ExecuTorch가 주도권을 가져갈 가능성이 높다.

결론적으로, TorchScript는 PyTorch의 발전에 지대한 공헌을 하고 역사의 뒤안길로 사라져 가는 과도기적 기술로 평가할 수 있다. 그 성공과 실패의 역사는 현재 PyTorch 2.x 시대를 연 중요한 자산으로 남아있다.


1. Understanding PyTorch Eager and Graph Mode | by Hey Amit - Medium, 8월 24, 2025에 액세스, https://medium.com/@heyamit10/understanding-pytorch-eager-and-graph-mode-e0b81c1f2396
2. PyTorch Models Are Not Deployment-Friendly! Supercharge Them With TorchScript., 8월 24, 2025에 액세스, https://www.dailydoseofds.com/pytorch-models-are-not-deployment-friendly-supercharge-them-with-torchscript/
3. Optimizing Production PyTorch Models' Performance with Graph ..., 8월 24, 2025에 액세스, https://pytorch.org/blog/optimizing-production-pytorch-performance-with-graph-transformations/
4. Concepts — ExecuTorch 0.7 documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/executorch/stable/concepts.html
5. JIT — PyTorch Training Performance Guide, 8월 24, 2025에 액세스, https://residentmario.github.io/pytorch-training-performance-guide/jit.html
6. PyTorch 2.0 release explained - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/pytorch/comments/zk6kc4/pytorch_20_release_explained/
7. TorchScript 소개 — 파이토치 한국어 튜토리얼 (PyTorch tutorials in ..., 8월 24, 2025에 액세스, https://tutorials.pytorch.kr/beginner/Intro_to_TorchScript_tutorial.html
8. JIT / Torch Script : 1. TRACE MODE , 2. SCRIPT MODE - 코딩고양이 - 티스토리, 8월 24, 2025에 액세스, https://cat-uni.tistory.com/34
9. 빠른 배포를 위한 YOLO11 모델의 TorchScript로 내보내기, 8월 24, 2025에 액세스, https://docs.ultralytics.com/ko/integrations/torchscript/
10. TorchScript — PyTorch 2.8 documentation, 8월 24, 2025에 액세스, https://pytorch.org/docs/stable/jit.html
11. 딥러닝 모델 배포하기 #02 - TorchScript & Pytorch JIT - Happy Jihye, 8월 24, 2025에 액세스, https://happy-jihye.github.io/dl/torch-2/
12. Faster Models with Graph Fusion: How Deep Learning Frameworks ..., 8월 24, 2025에 액세스, https://arikpoz.github.io/posts/2025-05-07-faster-models-with-graph-fusion-how-deep-learning-frameworks-optimize-your-computation/
13. Optimizing CUDA Recurrent Neural Networks with TorchScript - PyTorch, 8월 24, 2025에 액세스, https://pytorch.org/blog/optimizing-cuda-rnn-with-torchscript/
14. TorchScript for Model Optimization and Model Serving - Secure Machinery, 8월 24, 2025에 액세스, https://securemachinery.com/2023/11/12/torchscript-uses-for-model-optimization-and-serving/
15. tutorials.pytorch.kr, 8월 24, 2025에 액세스, [https://tutorials.pytorch.kr/recipes/torchscript_inference.html#:~:text=TorchScript**%EB%8A%94%20C%2B%2B%20%EA%B0%99%EC%9D%80,JIT%20Compiler%2C%20%EC%97%90%EC%84%9C%20%EC%82%AC%EC%9A%A9%EB%90%A9%EB%8B%88%EB%8B%A4.](https://tutorials.pytorch.kr/recipes/torchscript_inference.html#:~:text=TorchScript**는 C%2B%2B 같은,JIT Compiler%2C 에서 사용됩니다.)
16. www.geeksforgeeks.org, 8월 24, 2025에 액세스, [https://www.geeksforgeeks.org/deep-learning/pytorch-jit-and-torchscript-a-comprehensive-guide/#:~:text=TorchScript%20is%20the%20intermediate%20representation,Python%20may%20not%20be%20available.](https://www.geeksforgeeks.org/deep-learning/pytorch-jit-and-torchscript-a-comprehensive-guide/#:~:text=TorchScript is the intermediate representation,Python may not be available.)
17. PyTorch JIT and TorchScript: A Comprehensive Guide - GeeksforGeeks, 8월 24, 2025에 액세스, https://www.geeksforgeeks.org/deep-learning/pytorch-jit-and-torchscript-a-comprehensive-guide/
18. Research to Production: PyTorch JIT/TorchScript Updates - Michael Suo - YouTube, 8월 24, 2025에 액세스, https://www.youtube.com/watch?v=St3gdHJzic0
19. 실무에서 Pytorch를 C++로 변환하는 작업에 대한 질문. - 파이토치 한국 사용자 모임, 8월 24, 2025에 액세스, https://discuss.pytorch.kr/t/pytorch-c/3604
20. TorchScript: Tracing vs. Scripting - Yuxin's Blog, 8월 24, 2025에 액세스, https://ppwwyyxx.com/blog/2022/TorchScript-Tracing-vs-Scripting/
21. C++에서 TorchScript 모델 로딩하기 - PyTorch 한국어 튜토리얼, 8월 24, 2025에 액세스, https://tutorials.pytorch.kr/advanced/cpp_export.html
22. Wizaron/pytorch-cpp-inference: Serving PyTorch 1.0 ... - GitHub, 8월 24, 2025에 액세스, https://github.com/Wizaron/pytorch-cpp-inference
23. custom model, torch.jit.script 따라가기 - velog, 8월 24, 2025에 액세스, [https://velog.io/@sjj995/custom-model-torch.jit.script-%EB%94%B0%EB%9D%BC%EA%B0%80%EA%B8%B0](https://velog.io/@sjj995/custom-model-torch.jit.script-따라가기)
24. TorchScript? torch.jit.script 에러 유형 및 해결 방법, 8월 24, 2025에 액세스, https://byeongjo-kim.tistory.com/33
25. TorchScript cannot handle declared module attributes of Python types #53753 - GitHub, 8월 24, 2025에 액세스, https://github.com/pytorch/pytorch/issues/53753
26. Torchscript throws *args and *kwargs error while converting a model : r/pytorch - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/pytorch/comments/sgbxph/torchscript_throws_args_and_kwargs_error_while/
27. docs/source/jit_unsupported.rst · master - pytorch - GitLab EPFL, 8월 24, 2025에 액세스, https://gitlab.epfl.ch/hugon/pytorch/-/blob/master/docs/source/jit_unsupported.rst
28. Introduction to TorchScript — PyTorch Tutorials 2.0.1+cu117 documentation, 8월 24, 2025에 액세스, https://pytorch-tutorials-preview.netlify.app/beginner/intro_to_torchscript_tutorial
29. TorchScript Unsupported PyTorch Constructs, 8월 24, 2025에 액세스, https://docs.pytorch.org/docs/stable/jit_unsupported.html
30. Dynamic Fusion in PyTorch: The Future of Accelerated Deep Learning with JIT, TorchScript, and Quantization | gganbu marketplace, 8월 24, 2025에 액세스, https://gganbumarketplace.com/machine-learning/dynamic-fusion-in-pytorch-the-future-of-accelerated-deep-learning-with-jit-torchscript-and-quantization/
31. Operator Fusion in XLA: Analysis and Evaluation - Daniel Snider, 8월 24, 2025에 액세스, https://danielsnider.ca/papers/Operator_Fusion_in_XLA_Analysis_and_Evaluation.pdf
32. Distributed Optimizer with TorchScript support - PyTorch 한국어 튜토리얼, 8월 24, 2025에 액세스, https://tutorials.pytorch.kr/recipes/distributed_optim_torchscript.html
33. 모델 저장하기 & 불러오기 - PyTorch 한국어 튜토리얼 - 파이토치 한국 사용자 모임, 8월 24, 2025에 액세스, https://tutorials.pytorch.kr/beginner/saving_loading_models.html
34. [Pytorch] checkpoint vs torchscript vs onnx 모델 속도 비교 - 지미뉴트론 개발일기 - 티스토리, 8월 24, 2025에 액세스, https://jimmy-ai.tistory.com/394
35. PyTorch에서 모델을 저장하고 로드하는 방법 | korean - Wandb, 8월 24, 2025에 액세스, https://wandb.ai/wandb_fc/korean/reports/PyTorch---VmlldzoxNTEwNzIx
36. [PytorchToC++] 02. TorchScript 분석 - 또오겠습니다 - 티스토리, 8월 24, 2025에 액세스, https://data-gardner.tistory.com/106
37. 파이토치 모델을 C++에서 사용하기 -1- - BW, 8월 24, 2025에 액세스, https://busterworld.tistory.com/116
38. BIGBALLON/PyTorch-CPP: PyTorch C++ inference with LibTorch - GitHub, 8월 24, 2025에 액세스, https://github.com/BIGBALLON/PyTorch-CPP
39. PyTorch 모델을 C++ 에 빌드 및 컴파일하기 - 모딩의 코딩블로그, 8월 24, 2025에 액세스, [https://moding.tistory.com/entry/3-C-%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC-%EB%B9%8C%EB%93%9C-%EC%BB%B4%ED%8C%8C%EC%9D%BC](https://moding.tistory.com/entry/3-C-라이브러리-빌드-컴파일)
40. TorchScript for Deployment — PyTorch Tutorials 1.8.1+cu102 documentation, 8월 24, 2025에 액세스, https://h-huang.github.io/tutorials/recipes/torchscript_inference.html
41. Advice for debugging a TorchScript module from C++ - PyTorch Forums, 8월 24, 2025에 액세스, https://discuss.pytorch.org/t/advice-for-debugging-a-torchscript-module-from-c/58223
42. ultralytics/examples/YOLOv8-LibTorch-CPP-Inference/main.cc at main - GitHub, 8월 24, 2025에 액세스, https://github.com/ultralytics/ultralytics/blob/main/examples/YOLOv8-LibTorch-CPP-Inference/main.cc
43. dk-liang/pytorch2libtorch: A demo for using C++ to inference the pytorch model - GitHub, 8월 24, 2025에 액세스, https://github.com/dk-liang/pytorch2libtorch
44. koba-jon/pytorch_cpp: Deep Learning sample programs using PyTorch in C++ - GitHub, 8월 24, 2025에 액세스, https://github.com/koba-jon/pytorch_cpp
45. What are ONNX and TorchScript? - Medium, 8월 24, 2025에 액세스, https://medium.com/@sujathamudadla1213/what-are-onnx-and-torchscript-dcc878a053b1
46. PyTorch란 무엇인가요? - IBM, 8월 24, 2025에 액세스, https://www.ibm.com/kr-ko/think/topics/pytorch
47. [D] TorchScript model vs ONNX : r/MachineLearning - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/MachineLearning/comments/strteu/d_torchscript_model_vs_onnx/
48. mtszkw/fast-torch: Comparing PyTorch, JIT and ONNX for inference with Transformers, 8월 24, 2025에 액세스, https://github.com/mtszkw/fast-torch
49. Performance Discrepancy Between PyTorch and ONNX Model Conversion: Unexpected Increase in Latency : r/computervision - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/computervision/comments/1en829d/performance_discrepancy_between_pytorch_and_onnx/
50. A beginner's guide to PyTorch 2.0 and torch.compile | by Ishan Kumar - Medium, 8월 24, 2025에 액세스, https://medium.com/@ishankumar216/a-beginners-guide-to-pytorch-2-0-and-torch-compile-7bd8a27ef7bd
51. PyTorch 2.x, 8월 24, 2025에 액세스, https://pytorch.org/get-started/pytorch-2-x/
52. Project Report - Torch compile and PyTorch 2.2.0 | GSoC 2024 - DeepChem, 8월 24, 2025에 액세스, https://forum.deepchem.io/t/project-report-torch-compile-and-pytorch-2-2-0-gsoc-2024/1441
53. Introduction to torch.compile — PyTorch Tutorials 2.8.0+cu128 documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/tutorials/intermediate/torch_compile_tutorial.html
54. torch.compile - Hugging Face, 8월 24, 2025에 액세스, https://huggingface.co/docs/transformers/perf_torch_compile
55. 
56. Consequences of PyTorch 2.0 on libtorch? - C++ - PyTorch Forums, 8월 24, 2025에 액세스, https://discuss.pytorch.org/t/consequences-of-pytorch-2-0-on-libtorch/167576
57. Questions about TorchScript support · Issue #30 · NVIDIA/cuEquivariance - GitHub, 8월 24, 2025에 액세스, https://github.com/NVIDIA/cuEquivariance/issues/30
58. Next Steps for PyTorch Compilers, 8월 24, 2025에 액세스, https://dev-discuss.pytorch.org/t/next-steps-for-pytorch-compilers/309

