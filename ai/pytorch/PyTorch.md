# PyTorch
[Pytorch](./index.md)


PyTorch는 현대 인공지능(AI) 개발의 패러다임을 바꾼 핵심적인 딥러닝 프레임워크다. 그 기원을 이해하는 것은 PyTorch가 왜 현재와 같은 모습으로 설계되었는지, 그리고 AI 커뮤니티에 어떤 영향을 미쳤는지를 파악하는 데 필수적이다. PyTorch는 단순한 기술적 산물이 아니라, 개발자 경험을 최우선으로 고려한 철학적 선택의 결과물이다. 본 장에서는 Torch 라이브러리로부터의 계승, 'Define-by-Run'이라는 혁신적인 패러다임 채택, 그리고 연구와 제품화의 간극을 메우기 위한 전략적 진화 과정을 심층적으로 분석한다.


PyTorch의 여정은 2016년, 당시 Facebook AI Research(FAIR, 현 Meta AI) 소속 연구원들에 의해 시작되었다.1 PyTorch는 Lua 언어 기반의 과학 컴퓨팅 프레임워크였던 Torch 라이브러리의 정신적 계승자로 탄생했다.1 Torch는 효율적인 C/C++ 백엔드와 강력한 GPU 가속 라이브러리를 통해 이미 학계에서 그 성능을 인정받고 있었다. PyTorch 개발팀은 Torch의 이러한 강력한 성능 기반은 그대로 유지하면서, 인터페이스를 당시 데이터 과학과 머신러닝 분야에서 폭발적인 인기를 얻고 있던 Python으로 전환하는 것을 핵심 목표로 삼았다.4

이러한 'Python-first' 철학은 PyTorch 설계의 근간을 이룬다. 이는 단순히 C++ 프레임워크에 Python 바인딩을 제공하는 수준을 넘어, 프레임워크 자체가 Python 생태계의 자연스러운 일부처럼 느껴지도록 설계되었음을 의미한다.7 예를 들어, PyTorch의 핵심 데이터 구조인 `Tensor`는 과학 컴퓨팅 분야의 표준 라이브러리인 NumPy의 `ndarray`와 매우 유사한 API를 제공한다.3 이 덕분에 기존에 NumPy를 사용하던 개발자들은 별도의 학습 곡선 없이도 쉽게 PyTorch에 적응할 수 있었다. 또한, PyTorch는 Python의 직관적인 문법과 프로그래밍 모델을 그대로 따르며, Python의 표준 디버깅 도구(예: `pdb`)를 아무런 제약 없이 활용할 수 있게 했다.6 이 접근법은 개발자들이 프레임워크의 복잡성에 얽매이지 않고, 오롯이 딥러닝 모델의 로직과 아이디어 구현에만 집중할 수 있는 환경을 제공했다.


PyTorch가 등장했을 때 딥러닝 커뮤니티에 가장 큰 충격을 준 특징은 바로 'Define-by-Run'이라는 새로운 패러다임이었다.11 이는 코드가 실행되는 시점(runtime)에 계산 그래프가 동적으로 생성되고, 필요에 따라 실시간으로 구조를 변경할 수 있음을 의미한다.13

이는 당시 시장을 지배하던 TensorFlow 1.x가 채택한 'Define-and-Run'(정의 후 실행) 방식과 근본적인 차이를 보였다.11 정적 계산 그래프 패러다임에서는 먼저 모델의 전체 구조를 하나의 거대한 그래프로 정의하고 컴파일한 다음, 이 고정된 그래프에 데이터를 주입하여 실행하는 방식이었다.16 이 방식은 전체 그래프를 미리 파악할 수 있어 전역적인 최적화에 유리하고, 모델을 다양한 환경에 이식하기 용이하다는 장점이 있었다.15 하지만 개발자에게는 상당한 제약으로 작용했다. 디버깅이 매우 어려웠고, 입력 데이터의 길이에 따라 네트워크 구조가 변하는 순환 신경망(RNN)과 같은 동적 모델을 구현하기가 매우 까다로웠다.15

반면, PyTorch의 동적 계산 그래프는 이러한 제약을 완전히 해소했다. 개발자는 일반적인 Python 코드를 작성하듯이 `for` 루프나 `if` 조건문을 사용하여 네트워크의 흐름을 자유롭게 제어할 수 있었다.14 각 연산이 실행될 때마다 그래프가 즉시 생성되므로, 코드의 어느 지점에서든 `print()` 함수를 사용해 텐서의 중간값을 확인하거나, Python 디버거를 이용해 단계별로 실행을 추적하는 것이 가능했다.1 이러한 극적인 유연성과 디버깅 편의성은 복잡하고 새로운 모델 아키텍처를 실험해야 하는 연구자들에게 폭발적인 호응을 얻었고, PyTorch가 학계와 연구 커뮤니티에서 빠르게 표준 프레임워크로 자리매김하는 결정적인 계기가 되었다.11

이러한 설계는 단순한 기술적 선택이 아니었다. 2016년 당시의 딥러닝 프레임워크들은 개발자에게 "프레임워크의 규칙"에 맞춰 코드를 작성하도록 강요했다. TensorFlow 1.x의 `tf.placeholder`와 `tf.Session` 같은 개념은 Python 개발자들에게는 생소하고 부자연스러운 장벽이었다.11 PyTorch는 이 패러다임을 뒤집어, "개발자가 이미 알고 있는 Python의 규칙"에 맞춰 프레임워크가 동작하도록 설계했다.7 이는 개발자의 인지 부하를 극적으로 낮추어 연구 생산성을 폭발적으로 증가시켰고, 결국 TensorFlow 2.0이 Eager Execution을 기본값으로 채택하며 PyTorch의 방식을 따라오게 만든 시장의 판도를 바꾼 혁신이었다.11

| 특징 (Feature)       | 정적 계산 그래프 (Define-and-Run)                 | 동적 계산 그래프 (Define-by-Run)                   |
| -------------------- | ------------------------------------------------- | -------------------------------------------------- |
| **대표 프레임워크**  | TensorFlow 1.x, Theano, Caffe                     | PyTorch, Chainer, TensorFlow 2.x (Eager)           |
| **그래프 정의 시점** | 코드 실행 전, 전체 그래프를 미리 정의 및 컴파일   | 코드 실행 중, 연산이 수행될 때마다 동적으로 생성   |
| **유연성**           | 낮음 (가변 길이 입력, 조건부 로직 처리 복잡)      | 높음 (Python 제어 흐름을 자연스럽게 사용 가능)     |
| **디버깅**           | 어려움 (Python 디버거 사용 불가, 별도 도구 필요)  | 쉬움 (표준 Python 디버깅 도구 `pdb`, `print` 사용) |
| **성능 최적화**      | 용이함 (전체 그래프를 보고 전역 최적화 수행 가능) | 상대적으로 어려움 (런타임 오버헤드 존재 가능)      |
| **사용자 경험**      | 비직관적 (별도의 세션, 플레이스홀더 개념 필요)    | 직관적 (일반적인 Python 프로그래밍과 유사)         |


초기 PyTorch는 연구에서의 강력한 유연성에도 불구하고, 실제 제품 환경에 모델을 배포하는 프로덕션(production) 측면에서는 약점을 보였다. 모바일 배포, 분산 학습, 고성능 추론 등에서는 Caffe2와 같은 프레임워크가 더 성숙한 솔루션을 제공하고 있었다.1 이러한 '연구용'이라는 인식을 극복하고 엔드투엔드(end-to-end) 프레임워크로 도약하기 위해 PyTorch는 전략적인 진화를 거듭했다.

그 첫 번째 중대한 이정표는 2018년 3월에 이루어진 Caffe2와의 통합이었다.1 이 통합은 PyTorch의 사용자 친화적인 Python 프론트엔드와 Caffe2의 고도로 최적화된 C++ 백엔드를 결합하는 것을 목표로 했다. 이를 통해 개발자들은 PyTorch의 직관적인 환경에서 모델을 실험하고 프로토타이핑한 후, 별도의 프레임워크 전환 없이 동일한 코드베이스를 사용하여 고성능의 프로덕션 환경에 원활하게 배포할 수 있게 되었다.1 이 통합의 결과물로 2018년 12월에 출시된 PyTorch 1.0은 TorchScript와 같은 핵심 기능을 도입했다. TorchScript는 PyTorch 모델을 Python 인터프리터에 의존하지 않는 정적 그래프 형태로 변환하여, C++ 환경에서의 고성능 추론 및 배포를 가능하게 했다.1

두 번째 이정표는 거버넌스의 변화였다. 2022년 9월, Meta는 PyTorch의 관리 권한을 리눅스 재단(Linux Foundation) 산하에 새로 설립된 독립적인 'PyTorch 재단(PyTorch Foundation)'으로 이관했다.1 이 결정은 PyTorch가 특정 기업의 전략에 종속되지 않는 중립적이고 개방적인 프로젝트임을 선언하는 중요한 의미를 가졌다. Meta, Microsoft, Amazon, Google, NVIDIA 등 AI 산업을 이끄는 주요 경쟁사들이 재단의 창립 멤버로 참여하여 PyTorch의 미래 방향성을 공동으로 결정하게 되었다.1 이는 PyTorch가 단순한 오픈소스 프로젝트를 넘어, AI 산업 전체를 지탱하는 핵심 인프라이자 공공재로 자리매김했음을 상징하는 사건이었다. Caffe2 통합이 기술적으로 연구와 제품화의 다리를 놓았다면, 재단 설립은 PyTorch를 '프로젝트'에서 '산업 표준 인프라'로 격상시키는 전략적 완성 단계였다.


PyTorch의 강력함과 유연성은 두 가지 핵심 구성 요소, 즉 `torch.Tensor`와 `autograd` 엔진에 기반한다. 텐서는 데이터를 표현하고 연산하는 기본 단위이며, `autograd`는 이 텐서들의 연산 기록을 바탕으로 복잡한 미분 과정을 자동화한다. 이 두 요소의 긴밀한 상호작용이 PyTorch의 동적 계산 그래프와 효율적인 신경망 학습을 가능하게 하는 원동력이다.


`torch.Tensor`는 PyTorch에서 모든 데이터와 모델 파라미터를 표현하는 핵심적인 다차원 배열 자료구조다.3 이는 개념적으로 NumPy의 `ndarray`와 매우 유사하여, 스칼라(0차원), 벡터(1차원), 행렬(2차원)뿐만 아니라 그 이상의 고차원 데이터를 다룰 수 있다.22 PyTorch는 `torch.float32`, `torch.int64` 등 다양한 데이터 타입을 지원하여 유연한 데이터 표현을 가능하게 한다.13

텐서는 다음과 같은 주요 속성을 통해 자신의 상태를 기술한다.

- `shape`: 텐서의 각 차원별 원소의 개수를 나타내는 튜플이다. 예를 들어, (64, 3, 224, 224)의 `shape`는 64개의 이미지가 각각 3개의 채널(RGB)과 224x224의 해상도를 가짐을 의미한다.22
- `dtype`: 텐서에 저장된 데이터의 타입을 명시한다. 연산의 정밀도와 메모리 사용량에 직접적인 영향을 미친다.22
- `device`: 텐서가 할당된 물리적 장치를 나타낸다. 기본값은 CPU(`'cpu'`)이며, `tensor.to('cuda')`와 같은 메서드를 통해 GPU로 데이터를 손쉽게 이동시킬 수 있다.22 이 속성은 단순한 위치 지정자를 넘어, 복잡한 시스템에서 데이터 흐름을 명시적으로 제어하고 디버깅을 용이하게 하는 PyTorch의 "Simple Over Easy" 설계 철학을 반영한다.7 프레임워크가 암묵적으로 최적의 장치를 결정하는 대신, 개발자에게 명시적인 제어권을 부여함으로써 이기종 컴퓨팅 환경에서의 예측 가능성과 안정성을 높인다.
- `requires_grad`: 이 불리언(boolean) 속성이 `True`로 설정되면, 해당 텐서에 가해지는 모든 연산이 `autograd` 엔진에 의해 추적된다. 이는 신경망의 학습 가능한 파라미터(가중치, 편향)에 대해 기울기를 계산하기 위한 필수적인 설정이다.8
- `grad`: `requires_grad=True`인 텐서에 대해 `.backward()`가 호출된 후, 계산된 기울기 값이 이 속성에 `Tensor` 형태로 저장되고 누적된다.8
- `grad_fn`: 텐서가 특정 연산의 결과로 생성되었을 때, 해당 연산을 수행한 `Function` 객체를 가리킨다. 이 속성은 계산 그래프를 역방향으로 추적하는 연결고리 역할을 하며, 텐서가 사용자에 의해 직접 생성된 경우(leaf tensor)에는 `None` 값을 가진다.25

PyTorch 텐서의 가장 큰 특징 중 하나는 하드웨어 가속 기능이다. NumPy 배열이 CPU 연산에 국한되는 반면, PyTorch 텐서는 NVIDIA GPU(CUDA)나 Apple Silicon(MPS)과 같은 가속기 위에서 연산을 수행할 수 있다.3 딥러닝에서 빈번하게 발생하는 대규모 행렬 곱셈과 같은 병렬 연산은 GPU에서 수행될 때 CPU 대비 엄청난 속도 향상을 가져온다.5

또한, PyTorch는 NumPy와의 뛰어난 호환성을 제공한다. CPU에 위치한 PyTorch 텐서와 NumPy 배열은 `torch.from_numpy()`나 `tensor.numpy()` 메서드를 통해 상호 변환될 때, 기본적으로 동일한 메모리 공간을 공유한다.22 이는 데이터 복사에 따른 오버헤드를 제거하여 효율적인 데이터 처리를 가능하게 한다.29 단, 이 메모리 공유는 텐서가 CPU에 있을 때만 유효하며, 텐서를 GPU로 이동시키는 순간(`tensor.to('cuda')`) 새로운 메모리가 GPU에 할당되면서 NumPy 배열과의 연결은 끊어진다.29

| 특징 (Feature)    | `torch.Tensor`                               | NumPy `ndarray`                                    |
| ----------------- | -------------------------------------------- | -------------------------------------------------- |
| **주요 용도**     | 딥러닝 및 수치 연산                          | 범용 과학 및 수치 연산                             |
| **하드웨어 가속** | GPU (CUDA, ROCm), MPS 등 지원                | CPU 전용 (GPU 지원은 CuPy 등 외부 라이브러리 필요) |
| **자동 미분**     | `autograd` 내장 지원 (`requires_grad=True`)  | 지원하지 않음                                      |
| **계산 그래프**   | 연산 기록을 통해 동적 계산 그래프 생성       | 계산 그래프 개념 없음                              |
| **생태계 통합**   | PyTorch 생태계(`nn`, `optim` 등)와 완벽 통합 | SciPy, Pandas, Scikit-learn 등과 통합              |
| **메모리 공유**   | CPU 상에서 NumPy 배열과 메모리 공유 가능     | CPU 상에서 PyTorch 텐서와 메모리 공유 가능         |


`torch.autograd`는 PyTorch가 신경망 학습을 자동화하는 핵심 엔진이다.20 개발자가 복잡한 미분 수식을 직접 코딩할 필요 없이, 순방향 연산만 정의하면 `autograd`가 역전파 과정을 통해 모든 파라미터의 기울기를 자동으로 계산해준다.3

`autograd`의 작동 원리는 동적 계산 그래프(DAG)에 기반한다. `requires_grad=True`로 설정된 텐서(일반적으로 모델의 파라미터와 입력 데이터)가 연산에 참여하면, PyTorch는 해당 연산과 입출력 텐서 간의 관계를 그래프 형태로 기록한다.5 이 그래프는 코드가 실행되는 순서에 따라 동적으로 구축되며, 잎(leaf) 노드는 입력 텐서, 중간 노드는 연산, 그리고 루트(root) 노드는 최종 결과(일반적으로 스칼라 값인 손실)가 된다.26 각 중간 텐서는 자신을 생성한 연산 함수를 `grad_fn` 속성을 통해 가리키며, 이 `grad_fn`들이 연결되어 전체 계산의 역사를 추적할 수 있는 경로를 형성한다.25

신경망 학습의 핵심인 기울기 계산은 최종 손실 텐서에 대해 `.backward()` 메서드를 호출함으로써 시작된다. 이 호출은 `autograd` 엔진에게 그래프를 루트에서부터 잎 노드 방향으로 역추적하며 각 파라미터에 대한 손실의 편미분 값을 계산하라고 지시하는 것과 같다. 이 과정의 수학적 기반은 미적분학의 연쇄 법칙(chain rule)이다.25

예를 들어, 손실 $L$이 모델의 출력 $y$에 의해 결정되고, $y$는 파라미터 $w$와 입력 $x$에 대한 함수 $f$의 결과($y=f(w, x)$)라고 가정하자. 연쇄 법칙에 따라 $L$을 $w$에 대해 편미분한 값은 다음과 같이 표현할 수 있다.
$$
\frac{\partial L}{\partial w} = \frac{\partial L}{\partial y} \frac{\partial y}{\partial w}
$$
`.backward()`가 호출되면, `autograd`는 먼저 루트 노드에서 자기 자신에 대한 기울기, 즉 $\frac{\partial L}{\partial L} = 1$에서 시작한다. 이 값을 `grad_fn`을 통해 이전 노드로 전달한다. 각 `grad_fn`은 입력으로 받은 상위 노드의 기울기(upstream gradient, $\frac{\partial L}{\partial y}$)에 자신의 연산에 대한 국소 기울기(local gradient, $\frac{\partial y}{\partial w}$)를 곱하여 하위 노드로 전달할 기울기(downstream gradient, $\frac{\partial L}{\partial w}$)를 계산한다.26 이 과정이 그래프를 따라 재귀적으로 반복되어 모든 잎 노드(학습 가능한 파라미터)에 도달하면, 각 파라미터 텐서의 `.grad` 속성에 최종적으로 계산된 기울기 값이 누적된다.26

이처럼 `autograd`와 동적 그래프의 결합은 '실행'과 '미분'을 하나의 통합된 과정으로 만들었다. 정적 그래프 프레임워크에서처럼 모델의 '정의' 단계와 '실행' 단계를 분리하고, '순방향 그래프'와 '역방향 그래프'를 별개의 개념으로 다룰 필요가 없어졌다.12 개발자는 그저 일반적인 Python 코드를 작성하여 순방향 연산을 수행하면, 미분은 `autograd`가 배경에서 알아서 처리해준다는 강력한 추상화를 제공받는다. 이는 특히 실행 경로가 데이터에 따라 달라지는 동적 모델의 구현을 극도로 단순화시켜 딥러닝 연구의 속도와 유연성을 획기적으로 향상시켰다.14


PyTorch에서 신경망을 구축하는 과정은 `torch.nn.Module`이라는 강력하고 유연한 클래스를 중심으로 이루어진다. `nn.Module`은 단순한 코드 컨테이너를 넘어, 모델의 구성 요소(레이어), 학습 가능한 파라미터, 그리고 데이터의 흐름을 정의하는 순방향 연산을 하나의 객체로 캡슐화하는 객체 지향적 추상화의 핵심이다. 본 장에서는 `nn.Module`의 구조와 역할을 분석하고, 이를 바탕으로 구성되는 표준적인 딥러닝 모델 학습 워크플로우를 단계별로 상세히 설명한다.


`torch.nn.Module`은 PyTorch에서 모든 신경망 아키텍처의 기본이 되는 부모 클래스다.34 사용자 정의 모델을 만들거나 개별 레이어를 구현할 때, 반드시 이 클래스를 상속받아야 한다. 

`nn.Module`을 상속받는 클래스는 일반적으로 두 개의 핵심 메서드를 구현해야 한다.37

- `__init__(self)`: 모델의 생성자 역할을 하는 메서드다. 이 메서드 내부에서 모델을 구성하는 데 필요한 레이어(예: `nn.Linear`, `nn.Conv2d`)나 다른 `nn.Module` 객체들을 정의하고 초기화한다. `super().__init__()`를 생성자 첫 줄에서 호출하여 부모 클래스의 초기화 로직을 실행하는 것이 필수적이다.34
- `forward(self, x)`: 모델의 순방향 연산(forward pass)을 정의하는 메서드다. 입력 데이터 `x`를 인자로 받아, `__init__`에서 정의된 레이어들을 순차적으로 또는 복잡한 로직에 따라 통과시킨 후 최종 출력값을 반환한다. 모델 객체를 함수처럼 호출할 때(예: `model(input_data)`), PyTorch는 내부적으로 이 `forward` 메서드를 실행한다. 직접 `model.forward(input_data)`를 호출하는 것은 권장되지 않는다.35

`nn.Module`의 본질은 상태(state)를 가지는 연산을 캡슐화하는 것이다. 여기서 상태란 신경망의 학습 가능한 가중치(weights)와 편향(biases)과 같은 파라미터를 의미한다. PyTorch가 제공하는 `nn.Linear`, `nn.Conv2d`, `nn.LSTM` 등 모든 내장 레이어들은 `nn.Module`을 상속받아 구현되어 있으며, 자체적으로 파라미터를 관리하고 순방향 연산을 정의한다.34


`nn.Module`의 가장 강력한 기능 중 하나는 모델 내의 모든 학습 가능한 파라미터를 체계적으로 관리하는 능력이다.

**파라미터 관리:** `nn.Module`의 속성으로 `nn.Parameter` 객체나 다른 `nn.Module` 객체를 할당하면, PyTorch는 이들을 자동으로 '등록'하여 추적한다.35

`nn.Parameter`는 `torch.Tensor`의 특별한 하위 클래스로, 모듈에 할당될 때 자동으로 `requires_grad=True` 속성을 가지게 된다. 이렇게 등록된 파라미터들은 다음과 같은 메서드를 통해 쉽게 접근할 수 있다.

- `model.parameters()`: 해당 모듈과 그 안에 포함된 모든 하위 모듈(submodules)의 학습 가능한 파라미터들을 이터레이터(iterator) 형태로 반환한다. 이 이터레이터는 옵티마이저를 초기화할 때 전달되어, 학습 과정에서 모든 가중치가 올바르게 업데이트되도록 하는 데 사용된다.34
- `model.named_parameters()`: 각 파라미터의 이름(예: `'layer1.weight'`)과 파라미터 객체 자체를 튜플 형태로 묶어 반환한다. 이는 특정 레이어의 가중치만 동결(freeze)하거나, 레이어별로 다른 학습률을 적용하는 등 세밀한 제어가 필요할 때 매우 유용하다.38

**계층적 구성:** `nn.Module`은 다른 `nn.Module`을 속성으로 포함할 수 있어, 마치 레고 블록을 조립하듯 복잡한 모델을 간단한 구성 요소들의 조합으로 만들 수 있다.34 이러한 계층적(nested) 구조는 코드의 가독성과 재사용성을 크게 향상시킨다.

- `nn.Sequential`: 가장 간단한 형태의 컨테이너로, 여러 모듈을 순서대로 담아 입력 데이터가 정의된 순서대로 각 모듈을 통과하도록 한다. 간단한 순차적 모델을 빠르게 구성할 때 유용하다.35
- `nn.ModuleList`와 `nn.ModuleDict`: Python의 표준 `list`나 `dict`와 유사하게 모듈들을 담을 수 있지만, 중요한 차이점은 이 컨테이너에 담긴 모듈들이 부모 `nn.Module`에 올바르게 등록된다는 것이다. 따라서 `model.parameters()` 호출 시 이 모듈들의 파라미터도 함께 반환된다. 모델의 레이어 수가 동적으로 결정되거나, 특정 조건에 따라 다른 활성화 함수를 사용해야 하는 등 유연한 아키텍처를 구현할 때 필수적이다.38

이러한 객체 지향적 설계 덕분에, 딥러닝 아키텍처를 '코드'가 아닌 '구성 요소(component)'의 조합으로 다룰 수 있게 된다. 예를 들어, ResNet의 기본 블록을 하나의 `nn.Module`로 구현하면, 이 블록을 재사용하여 전체 ResNet 아키텍처를 손쉽게 구축할 수 있다. 이는 복잡한 모델의 설계, 디버깅, 수정을 매우 용이하게 만들며, PyTorch의 직관성을 뒷받침하는 핵심적인 설계 원칙이다.


PyTorch를 사용한 모델 학습은 일반적으로 명확하게 정의된 일련의 단계를 따른다. 이 워크플로우는 데이터 준비부터 모델 정의, 학습 루프 실행에 이르기까지 체계적으로 구성되어 있다.40

1. 데이터 준비: Dataset과 DataLoader

효율적인 데이터 처리는 성공적인 모델 학습의 전제 조건이다. PyTorch는 이를 위해 Dataset과 DataLoader라는 두 가지 핵심 데이터 유틸리티를 제공한다. 이 둘의 역할을 분리하는 것은 데이터 처리 로직과 모델 학습 로직의 '관심사 분리(Separation of Concerns)' 원칙을 구현한 것으로, 코드의 모듈성과 재현성을 크게 향상시킨다.

- `torch.utils.data.Dataset`: 데이터셋의 샘플과 그에 해당하는 레이블을 저장하고, 특정 인덱스의 데이터에 접근하는 방법을 정의하는 추상 클래스다. 사용자는 `__len__` (데이터셋의 총 샘플 수 반환)와 `__getitem__` (주어진 인덱스(`idx`)에 해당하는 샘플과 레이블 반환)이라는 두 메서드를 구현하여 자신만의 커스텀 데이터셋 클래스를 만들 수 있다.43 이를 통해 데이터 소스가 이미지 파일, CSV, 데이터베이스 등 무엇이든 일관된 인터페이스로 모델에 제공할 수 있다.
- `torch.utils.data.DataLoader`: `Dataset` 객체를 입력으로 받아, 학습에 사용될 데이터 배치를 효율적으로 생성하는 이터레이터다. `DataLoader`는 미니배치(mini-batch) 구성, 데이터 셔플링(shuffling)을 통한 과적합 방지, `num_workers` 인자를 이용한 멀티프로세싱 기반 데이터 로딩 가속화 등 복잡한 데이터 공급 과정을 자동화한다.43
- 모델, 손실 함수, 옵티마이저 정의

데이터가 준비되면, 학습에 필요한 세 가지 핵심 요소를 정의해야 한다.

- **모델(Model):** `nn.Module`을 상속받아 신경망 아키텍처를 클래스로 정의한다.34
- **손실 함수(Loss Function):** 모델의 예측값과 실제 정답 사이의 오차를 측정하는 함수다. 이 오차(loss)는 모델이 얼마나 '잘못' 예측했는지를 나타내는 스칼라 값이며, 학습 과정에서 최소화해야 할 대상이다. `torch.nn` 모듈은 다양한 문제 유형에 맞는 손실 함수들을 제공한다. 예를 들어, 다중 클래스 분류 문제에는 `nn.CrossEntropyLoss`를, 회귀 문제에는 `nn.MSELoss`(Mean Squared Error)를 주로 사용한다.48
- **옵티마이저(Optimizer):** `torch.optim` 모듈에 구현되어 있으며, 손실 함수로부터 계산된 기울기(gradient)를 사용하여 모델의 학습 가능한 파라미터(가중치와 편향)를 업데이트하는 알고리즘이다. 가장 기본적인 `optim.SGD`(Stochastic Gradient Descent)부터, 더 정교한 적응형 학습률 알고리즘인 `optim.Adam`까지 다양한 옵티마이저를 선택할 수 있다.51
- 학습 루프 (Training Loop)

모든 준비가 끝나면, 실제 모델 학습이 이루어지는 학습 루프를 실행한다. 학습 루프는 전체 데이터셋을 여러 번 반복(epoch)하며, 각 에폭은 DataLoader가 제공하는 미니배치 단위로 다음의 5단계를 반복 수행한다.34

1. `optimizer.zero_grad()`: PyTorch는 기울기를 계산할 때마다 기존 값에 누적하는 방식으로 동작하므로, 새로운 미니배치에 대한 기울기를 계산하기 전에 이전 배치의 기울기 값을 명시적으로 0으로 초기화해야 한다.
2. `outputs = model(inputs)`: 현재 미니배치의 입력 데이터를 모델에 통과시켜 예측값을 계산한다 (순방향 전파).
3. `loss = criterion(outputs, labels)`: 모델의 예측값과 실제 정답 레이블을 손실 함수에 입력하여 손실값을 계산한다.
4. `loss.backward()`: 계산된 손실값에 대해 `.backward()` 메서드를 호출한다. 이 한 줄의 코드로 `autograd` 엔진이 활성화되어, 손실에 대한 모델의 모든 학습 가능한 파라미터의 기울기가 자동으로 계산된다 (역전파).
5. `optimizer.step()`: 옵티마이저가 `.grad` 속성에 저장된 기울기 값을 바탕으로, 정의된 최적화 알고리즘(예: SGD, Adam)에 따라 모델의 파라미터를 업데이트한다.

이 과정을 수많은 에폭과 미니배치에 걸쳐 반복함으로써, 모델은 점차 손실을 최소화하는 방향으로 파라미터를 조정하며 데이터의 패턴을 학습하게 된다.


딥러닝 모델의 크기와 학습 데이터의 양이 기하급수적으로 증가함에 따라, 단일 GPU의 연산 능력과 메모리 용량은 점차 한계에 부딪히게 되었다. 이러한 문제를 해결하기 위해 PyTorch는 다중 GPU 및 다중 노드 환경에서 학습을 효율적으로 수행할 수 있는 강력한 분산 학습(distributed training) 기능과 다양한 하드웨어 가속기 지원을 제공한다. PyTorch의 분산 학습 API 발전 과정은 모델 크기라는 제약을 극복하기 위한 메모리 효율성 증대 투쟁의 역사와 같다.


nn.DataParallel (DP)

nn.DataParallel은 단일 머신에 장착된 여러 개의 GPU를 활용하여 학습을 병렬화하는 가장 간단한 방법이다.53 사용법은 모델을 `nn.DataParallel(model)`로 감싸기만 하면 될 정도로 직관적이다.

DP는 단일 프로세스, 다중 스레드 방식으로 동작한다. 먼저, 지정된 주 GPU(master GPU, 보통 `cuda:0`)가 모델을 로드하고, 각 학습 단계마다 입력 데이터 배치를 여러 GPU로 분할하여 전송(scatter)한다. 모델은 각 GPU에 복제되며, 각 복제본은 할당된 데이터 조각에 대해 순방향 연산을 수행한다. 계산된 출력은 다시 주 GPU로 모아져(gather) 손실이 계산된다. 역전파 과정에서는 각 GPU에서 계산된 기울기가 주 GPU로 다시 모여 합산된 후, 주 GPU에서 파라미터 업데이트가 이루어진다. 마지막으로, 업데이트된 모델 파라미터가 다시 모든 GPU로 브로드캐스팅된다.54

이러한 구조는 몇 가지 심각한 성능적 한계를 가진다. 모든 데이터와 기울기가 주 GPU를 거쳐야 하므로 주 GPU에 심각한 병목 현상이 발생한다. 또한, Python의 전역 인터프리터 잠금(Global Interpreter Lock, GIL)으로 인해 다중 스레드의 효율이 저하되며, 매 이터레이션마다 모델을 복제하는 오버헤드가 발생한다.54 이러한 이유로 GPU 수가 늘어나도 성능 향상이 선형적으로 이루어지지 않으며, 현재는 `DistributedDataParallel`의 사용이 강력히 권장된다.55

nn.parallel.DistributedDataParallel (DDP)

DistributedDataParallel은 DP의 한계를 극복하기 위해 설계된 훨씬 더 강력하고 효율적인 분산 학습 도구다. DDP는 다중 프로세스 방식을 채택하여, 각 GPU마다 별도의 독립적인 프로세스를 생성한다.56 각 프로세스는 모델의 완전한 복제본을 가지고 있으며, 단일 노드뿐만 아니라 다중 노드(여러 머신) 환경으로의 확장을 완벽하게 지원한다.

DDP의 작동 방식은 DP와 근본적으로 다르다. 학습이 시작되면 각 프로세스는 전체 데이터셋의 서로 다른 일부를 `DataLoader`로부터 공급받는다. 각 프로세스는 독립적으로 순방향 및 역방향 연산을 수행한다. 역전파 과정에서 각 파라미터의 기울기 계산이 완료되는 즉시, DDP는 `torch.distributed` 패키지의 효율적인 집합 통신(collective communication) 연산, 특히 `AllReduce`를 사용하여 모든 프로세스의 기울기를 백그라운드에서 동기화(평균)한다.55 이 과정은 역전파 계산과 중첩(overlap)되어 통신 오버헤드를 최소화한다. 모든 프로세스가 동일한 평균 기울기 값을 가지게 되므로, 각 프로세스의 옵티마이저는 독립적으로, 하지만 동일하게 파라미터를 업데이트한다.

이러한 설계 덕분에 DDP는 주 GPU 병목 현상이 없으며, GIL의 제약을 받지 않고, 통신과 계산을 효율적으로 중첩시켜 DP에 비해 월등히 높은 성능과 확장성을 제공한다.53


DDP는 성능 면에서 매우 효율적이지만, 각 GPU가 모델 전체의 복사본과 옵티마이저 상태, 기울기를 모두 메모리에 유지해야 한다는 근본적인 한계를 가진다. 이로 인해 모델의 크기가 단일 GPU의 메모리 용량을 초과하면 DDP를 사용할 수 없게 된다.58 대규모 언어 모델(LLM)의 등장으로 모델 파라미터가 수십억, 수천억 개로 증가하면서 이 문제는 딥러닝의 가장 큰 장벽이 되었다.

FSDP(Fully Sharded Data Parallel)는 이러한 메모리 제약을 극복하기 위해 등장한 혁신적인 기술이다. Microsoft DeepSpeed의 ZeRO(Zero Redundancy Optimizer) Stage 3 전략에서 영감을 받아 PyTorch에 네이티브하게 구현되었다.60

FSDP의 핵심 아이디어는 DDP에서 중복으로 저장되던 모델 구성 요소들(파라미터, 기울기, 옵티마이저 상태)을 모든 GPU에 걸쳐 잘게 분할(shard)하여 저장하는 것이다.63 즉, 각 GPU는 모델 전체가 아닌, 모델의 일부 조각(shard)만을 소유한다.

- **순방향/역방향 연산:** 특정 레이어의 연산이 필요할 때, 해당 레이어의 파라미터 조각들이 모든 GPU로부터 `All-Gather` 통신을 통해 일시적으로 수집되어 완전한 형태의 파라미터로 재구성된다. 연산이 끝나면, 다른 GPU로부터 빌려온 파라미터 조각들은 즉시 메모리에서 해제되어 메모리 사용량을 최소화한다.66
- **기울기 계산:** 역전파 후, 각 GPU에서 계산된 전체 기울기는 `Reduce-Scatter` 연산을 통해 다시 분할되어, 각 GPU는 자신이 소유한 파라미터 조각에 해당하는 기울기 조각만을 저장하고 동기화한다.64

이러한 '필요할 때만 모으고, 사용 후 즉시 버리는' 전략을 통해 FSDP는 DDP 대비 GPU 메모리 사용량을 극적으로 줄일 수 있다. 이로써 단일 GPU 메모리 용량을 훨씬 뛰어넘는 수십억, 수천억, 심지어 조 단위 파라미터 스케일의 초거대 모델 학습이 가능해졌다.58 이는 LLM 시대를 기술적으로 가능하게 한 핵심 동력 중 하나로 평가받는다.

| 특징 (Feature)     | `nn.DataParallel` (DP)            | `nn.DistributedDataParallel` (DDP)    | `Fully Sharded Data Parallel` (FSDP)          |
| ------------------ | --------------------------------- | ------------------------------------- | --------------------------------------------- |
| **병렬화 모델**    | 단일 프로세스, 다중 스레드        | 다중 프로세스                         | 다중 프로세스                                 |
| **적용 범위**      | 단일 노드                         | 단일 노드, 다중 노드                  | 단일 노드, 다중 노드                          |
| **메모리 사용**    | 높음 (모든 GPU에 모델 전체 복제)  | 높음 (모든 GPU에 모델 전체 복제)      | 낮음 (파라미터, 기울기, 옵티마이저 상태 분할) |
| **통신 오버헤드**  | 높음 (주 GPU 병목 현상)           | 낮음 (AllReduce를 통한 효율적 동기화) | 상대적으로 높지만 계산과 중첩하여 최적화      |
| **성능/확장성**    | 낮음 (GPU 수 증가 시 성능 저하)   | 높음 (선형에 가까운 확장성)           | 매우 높음 (초거대 모델 학습 가능)             |
| **주요 사용 사례** | 간단한 프로토타이핑 (현재 비권장) | 대부분의 분산 학습 시나리오           | 단일 GPU 메모리를 초과하는 거대 모델 학습     |


PyTorch의 성능은 최신 하드웨어 가속기를 효율적으로 활용하는 능력에 크게 의존한다.

- **CUDA:** PyTorch는 초기부터 NVIDIA GPU를 위한 병렬 컴퓨팅 플랫폼인 CUDA를 완벽하게 지원해왔다. `tensor.to('cuda')`라는 간단한 명령어를 통해 텐서와 모델을 GPU 메모리로 이동시키면, 이후의 모든 연산은 GPU의 수천 개 코어를 활용하여 병렬로 처리된다.3 이는 딥러닝의 핵심인 대규모 행렬 연산에서 CPU 대비 수십에서 수백 배에 달하는 압도적인 성능 향상을 제공한다.5

- **Apple Silicon (MPS):** PyTorch 1.12 릴리스부터는 Apple의 M-시리즈 칩(Apple Silicon)에 내장된 GPU를 활용하기 위한 새로운 백엔드인 Metal Performance Shaders(MPS)를 지원하기 시작했다.70 개발자는 

  `device='mps'`를 지정하여 자신의 Mac 컴퓨터에서 직접 GPU 가속을 활용한 모델 학습과 추론을 수행할 수 있게 되었다. 이는 값비싼 클라우드 GPU 자원이나 별도의 NVIDIA GPU 시스템 없이도 로컬 환경에서 빠른 프로토타이핑과 모델 파인튜닝이 가능해졌음을 의미한다.71 특히 Apple Silicon의 통합 메모리 아키텍처는 CPU와 GPU가 동일한 메모리 풀에 접근하여 데이터 전송으로 인한 지연을 줄여주므로, 엔드투엔드 성능 향상에 기여한다.71

이처럼 PyTorch는 업계 표준인 CUDA 지원을 공고히 하는 동시에, Apple Silicon과 같은 새로운 하드웨어 플랫폼으로의 지원을 적극적으로 확장하며 다양한 개발 환경에서 고성능 딥러닝을 가능하게 하고 있다.


딥러닝 모델의 가치는 연구실에서 개발되는 것을 넘어 실제 제품과 서비스에 통합될 때 비로소 완성된다. PyTorch는 초기에는 연구용 프레임워크라는 인식이 강했지만, 지속적인 발전을 통해 모델을 프로덕션 환경에 배포하기 위한 강력하고 다양한 도구들을 제공하게 되었다. 본 장에서는 학습이 완료된 PyTorch 모델을 실제 서비스 환경으로 이전하는 데 사용되는 핵심 기술인 TorchScript와 TorchServe, 그리고 프레임워크 간 상호운용성을 위한 ONNX에 대해 다룬다.


프로덕션 환경은 종종 Python이 아닌 C++, Java와 같은 언어로 구성되거나, 성능상의 이유로 Python 인터프리터의 오버헤드를 피해야 하는 경우가 많다. TorchScript는 이러한 문제를 해결하기 위해 PyTorch 모델을 Python 런타임에 의존하지 않는, 직렬화(serialization) 가능하고 최적화된 중간 표현(Intermediate Representation, IR)으로 변환하는 기술이다.1 이렇게 변환된 모델은 C++ 애플리케이션에 직접 로드하여 고성능 추론을 수행하거나, 모바일 기기에 배포하는 등 다양한 프로덕션 시나리오에 활용될 수 있다.75

TorchScript는 '유연성'과 '성능'이라는 상충되는 가치를 모두 만족시키기 위해 두 가지 상호 보완적인 모델 변환 방식을 제공한다.

- Tracing (torch.jit.trace):

  Tracing은 가장 간단하고 직관적인 변환 방식이다. 실제 예제 입력 데이터를 모델에 한 번 통과시키고, 이 과정에서 실행되는 모든 텐서 연산들을 순서대로 기록하여 계산 그래프를 생성한다.73 이 방식은 기존 모델 코드를 거의 수정할 필요 없이 적용할 수 있어 매우 편리하다. CNN과 같이 데이터의 흐름이 입력에 따라 변하지 않는 정적인 모델에 특히 효과적이다.73

  하지만 Tracing의 치명적인 단점은 데이터에 의존적인 제어 흐름(예: if 문이나 for 루프)을 제대로 포착하지 못한다는 것이다.74 트레이싱 시 사용된 예제 입력이 통과한 특정 실행 경로만이 그래프에 고정되므로, 다른 입력을 사용했을 때 실행될 수 있는 다른 경로는 무시된다. 이는 모델의 의도치 않은 동작이나 오류로 이어질 수 있다.79

- Scripting (torch.jit.script):

  Scripting은 Tracing의 한계를 극복하기 위한 더 강력한 방식이다. 이 방법은 모델의 Python 소스 코드를 직접 정적으로 분석하고 파싱하여 TorchScript 코드로 변환한다.73 모델의 실행을 기록하는 대신 코드 자체를 이해하기 때문에, `if` 문, `for` 루프 등 데이터에 따라 동적으로 변하는 모든 제어 흐름과 로직을 정확하게 그래프에 포함시킬 수 있다.73 RNN이나 어텐션 기반 모델과 같이 복잡한 로직을 가진 모델을 변환하는 데 필수적이다.

  단점은 Scripting이 지원하는 Python 문법과 기능이 일부 제한되어 있다는 것이다. 따라서 기존의 Eager 모드 코드가 TorchScript 컴파일러와 호환되도록 일부 수정해야 할 필요가 있을 수 있다.74

실제로는 이 두 가지 방식을 혼합하여 사용하는 것이 효과적이다. 모델의 정적인 부분(예: CNN 백본)은 Tracing으로 간단하게 변환하고, 동적인 로직이 포함된 부분(예: 디코더의 빔 서치)은 Scripting으로 변환하여 하나의 TorchScript 모듈로 결합할 수 있다.73

| 특징 (Feature)     | Tracing (`torch.jit.trace`)                           | Scripting (`torch.jit.script`)                     |
| ------------------ | ----------------------------------------------------- | -------------------------------------------------- |
| **변환 방식**      | 예제 입력을 실행하여 연산 흐름을 기록 (Runtime-based) | 소스 코드를 정적으로 분석하여 변환 (Code-based)    |
| **제어 흐름 처리** | 처리 불가 (실행된 경로만 그래프에 고정됨)             | 처리 가능 (if, for 등 동적 로직 보존)              |
| **입력 의존성**    | 높음 (입력의 shape이나 type이 바뀌면 재추적 필요)     | 낮음 (입력과 무관하게 모델 구조 자체를 변환)       |
| **적용 용이성**    | 높음 (코드 수정 거의 불필요)                          | 낮음 (TorchScript 호환 코드로 수정 필요할 수 있음) |
| **적합한 모델**    | CNN 등 제어 흐름이 없는 정적인 모델                   | RNN, Transformer 등 동적 제어 흐름이 있는 모델     |
| **주요 한계**      | 동적 로직을 놓쳐 잘못된 그래프를 생성할 수 있음       | 지원하지 않는 Python 기능 사용 시 컴파일 오류 발생 |


모델이 직렬화되었다면, 이를 실제 서비스 요청에 응답할 수 있는 안정적이고 확장 가능한 환경에 배포해야 한다.

- TorchServe:

  TorchServe는 PyTorch 재단이 공식적으로 지원하는 모델 서빙 프레임워크로, AWS와의 협력을 통해 개발되었다.11 TorchServe는 프로덕션 환경에서 PyTorch 모델을 배포하고 관리하는 데 필요한 모든 기능을 갖추고 있다.

  주요 기능으로는 REST 및 gRPC API 엔드포인트를 자동으로 생성하여 외부 애플리케이션과의 연동을 쉽게 하고, 여러 모델을 동시에 서빙하거나 동일 모델의 여러 버전을 관리하는 기능, 들어오는 요청을 동적으로 배치(batching)하여 처리량을 높이는 기능, 그리고 서버의 상태와 모델 성능을 모니터링하기 위한 로깅 및 메트릭 수집 기능 등이 있다.80 또한, Kubernetes, Amazon SageMaker, Google Vertex AI와 같은 주요 클라우드 및 컨테이너 오케스트레이션 플랫폼과 원활하게 통합되어 대규모 분산 환경에서의 모델 배포를 지원한다.82

- ONNX (Open Neural Network Exchange):

  ONNX는 특정 딥러닝 프레임워크에 종속되지 않고 모델을 교환할 수 있도록 설계된 개방형 표준 포맷이다.3 PyTorch 모델을 ONNX 형식으로 내보내면(export), PyTorch가 설치되지 않은 환경에서도 모델을 실행할 수 있는 큰 장점이 있다.

  예를 들어, ONNX로 변환된 모델은 NVIDIA의 TensorRT나 Microsoft의 ONNX Runtime과 같은 고도로 최적화된 추론 엔진에서 실행될 수 있다. 이러한 엔진들은 특정 하드웨어(예: NVIDIA GPU)에 맞춰 연산을 융합하고 정밀도를 조정하는 등 추가적인 최적화를 수행하여 PyTorch 런타임보다 훨씬 빠른 추론 속도를 제공할 수 있다.11 또한, ONNX는 TensorFlow를 포함한 다른 딥러닝 프레임워크에서도 모델을 불러올 수 있게 하여, 기존에 TensorFlow Serving 등으로 구축된 배포 파이프라인에 PyTorch 모델을 통합해야 할 때 유용한 다리 역할을 한다.


PyTorch 2.0의 출시는 프레임워크 역사상 가장 중요한 변곡점 중 하나로 평가된다. 이는 단순한 버전 업데이트를 넘어, PyTorch의 핵심 작동 방식을 근본적으로 바꾸는 '컴파일러'라는 새로운 패러다임을 도입했기 때문이다. `torch.compile`의 등장은 Eager 모드의 직관적인 개발 경험과 Graph 모드의 높은 성능이라는, 그동안 양립하기 어려웠던 두 마리 토끼를 모두 잡으려는 야심 찬 시도였다.


2023년 3월에 공식 출시된 PyTorch 2.0의 핵심은 `torch.compile()`이라는 단일 함수 API다.3 사용자는 기존의 PyTorch 모델 코드를 단 한 줄도 수정할 필요 없이, `compiled_model = torch.compile(model)`이라는 한 줄의 코드를 추가하는 것만으로 상당한 성능 향상을 기대할 수 있다.

`torch.compile`의 핵심 목표는 명확하다. 바로 PyTorch의 최대 강점인 Eager 모드의 유연성과 Pythonic한 개발 경험을 그대로 유지하면서, 백그라운드에서 JIT(Just-In-Time) 컴파일을 통해 정적 그래프 프레임워크 수준의 실행 속도를 달성하는 것이다.85 이는 과거 TorchScript가 해결하고자 했던 '유연성 대 성능'의 트레이드오프 문제를 보다 근본적이고 사용자 친화적인 방식으로 해결하려는 시도다. 중요한 점은 `torch.compile`이 기존 코드와 100% 역호환성을 보장하는 완전히 선택적인(optional) 기능이라는 것이다. 컴파일에 실패하더라도 모델은 기존의 Eager 모드로 정상적으로 동작한다.85

이러한 변화의 배경에는 PyTorch의 개발 철학이 C++ 중심에서 Python 중심으로 전환되는 중대한 흐름이 있다. PyTorch 1.x 시대의 성능 최적화는 TorchScript나 C++ 확장을 통해 Python의 한계를 벗어나 C++의 영역으로 넘어가는 것을 의미했다.7 그러나 PyTorch 2.0의 핵심 기술들은 대부분 Python으로 작성되어, 방대한 Python 커뮤니티가 프레임워크의 핵심 컴파일러 기술에 더 쉽게 기여하고 이를 확장할 수 있는 길을 열었다.85 이는 '성능을 위해서는 C++로 가야 한다'는 통념을 깨고, 'Python-first' 철학을 성능 최적화의 영역까지 확장한 것이다.


`torch.compile`의 마법과 같은 성능 향상은 보이지 않는 곳에서 동작하는 여러 혁신적인 기술들의 유기적인 결합을 통해 이루어진다.85

- **TorchDynamo (그래프 캡처):** `torch.compile`의 가장 앞단에 위치한 프론트엔드다. TorchDynamo는 CPython의 Frame Evaluation API라는 저수준 기능을 활용하여, 실행 중인 Python 바이트코드를 분석하고 PyTorch 연산들을 안전하게 포착하여 FX Graph라는 중간 표현(IR)으로 변환한다.85 TorchDynamo의 가장 큰 혁신은 '그래프 분할(graph break)' 기능이다. 만약 코드 내에 컴파일이 불가능한 동적 Python 구문(예: 지원되지 않는 라이브러리 호출, 복잡한 데이터 구조)을 만나면, 전체 컴파일을 포기하는 대신 컴파일 가능한 부분까지만 그래프로 만들고, 지원되지 않는 부분은 기존의 Eager 모드로 실행하도록 남겨둔다. 이후 다시 컴파일 가능한 코드가 나타나면 새로운 그래프 캡처를 시작한다. 이 덕분에 사용자는 기존 코드를 거의 수정하지 않고도 부분적인 성능 향상을 누릴 수 있다.85
- **AOTAutograd (순방향/역방향 그래프 생성):** TorchDynamo가 순방향 연산 그래프를 캡처하면, AOTAutograd가 이 그래프를 입력으로 받아 역전파(backward pass)에 필요한 연산들을 미리(Ahead-Of-Time) 추적하여 완전한 순방향-역방향 결합 그래프를 생성한다. 이를 통해 학습 과정 전체(순방향과 역방향 모두)가 컴파일러의 최적화 대상이 될 수 있다.85
- **TorchInductor (백엔드 컴파일러):** `torch.compile`의 기본 백엔드 컴파일러로, AOTAutograd가 생성한 최적화 가능한 그래프를 받아 실제 하드웨어에서 실행될 고도로 최적화된 기계 코드를 생성하는 역할을 한다.
  - **GPU 환경:** NVIDIA 및 AMD GPU를 위해, OpenAI가 개발한 Triton 딥러닝 컴파일러를 활용한다. Triton은 Python과 유사한 언어로 고성능 GPU 커널을 작성할 수 있게 해주며, TorchInductor는 이를 이용해 여러 PyTorch 연산을 하나의 최적화된 커널로 융합(operator fusion)하거나, 메모리 접근 패턴을 최적화하는 등의 작업을 수행한다.85
  - **CPU 환경:** 병렬 처리를 위한 OpenMP 지시문이 포함된 C++ 코드를 생성하여 멀티코어 CPU의 성능을 최대한 활용한다.93

이러한 스택은 JAX와 같은 순수 JIT 컴파일 프레임워크와 PyTorch를 차별화한다. JAX가 전체 함수를 컴파일하는 것을 전제로 하여 강력한 성능을 제공하지만 코드 작성에 제약을 두는 반면, PyTorch는 TorchDynamo의 그래프 분할 기능을 통해 기존의 방대한 코드베이스와 Eager 모드의 자유도를 포용하면서도 점진적인 성능 개선 경로를 제공하는 실용주의적 접근을 택했다.91


`torch.compile`의 효과는 다양한 모델과 조건에서 검증되었다. PyTorch 팀이 165개의 오픈소스 모델을 대상으로 수행한 공식 벤치마크 결과에 따르면, `torch.compile`은 Eager 모드 대비 float32 정밀도에서 평균 20%, AMP(Automatic Mixed Precision) 정밀도에서는 평균 36%의 학습 속도 향상을 기록했다.86

성능 향상의 정도는 여러 요인에 따라 달라진다. 일반적으로 모델이 크고, 연산이 복잡하며, 배치 크기가 클수록 컴파일로 인한 이점이 커진다. 연산자 융합과 커널 실행 오버헤드 감소 효과가 더 크게 나타나기 때문이다. 이 때문에 `torch.compile`은 주로 학습(training) 시나리오에서 빛을 발한다.94

반면, 배치 크기가 1과 같이 매우 작은 실시간 추론(inference) 환경에서는 그 효과가 미미하거나, 경우에 따라서는 Eager 모드와 차이가 없을 수도 있다.94 이러한 시나리오에서는 ONNX Runtime이나 NVIDIA의 TensorRT와 같이 추론에 특화된 전문 엔진들이 더 공격적인 최적화를 통해 우수한 성능을 보이는 경우가 많다. 이는 `torch.compile`이 학습과 추론 모두를 지원하지만, 그 설계의 무게중심이 학습 성능 최적화에 더 기울어져 있음을 시사한다.

최상의 성능을 이끌어내기 위해서는 '그래프 분할'의 발생을 최소화하는 것이 중요하다. `torch._dynamo.explain` 유틸리티나 `torch.compile(model, fullgraph=True)` 옵션을 사용하면 코드의 어느 부분에서 그래프 분할이 발생하는지 진단할 수 있으며, 이를 바탕으로 코드를 수정하여 엔드투엔드 컴파일을 달성하고 성능을 극대화할 수 있다.90


PyTorch의 성공은 단순히 뛰어난 코어 기능 때문만이 아니라, 그 주위를 둘러싼 풍부하고 역동적인 생태계 덕분이다. 특정 도메인의 문제를 해결하는 전문 라이브러리부터, 코드의 복잡성을 줄여주는 상위 수준 프레임워크, 그리고 최신 AI 연구를 주도하는 커뮤니티에 이르기까지, PyTorch 생태계는 AI 개발의 모든 단계를 지원한다. 본 장에서는 PyTorch의 핵심 생태계 구성 요소를 살펴보고, LLM 시대에서의 역할, 경쟁 구도, 그리고 미래 발전 방향을 조망한다.


PyTorch는 핵심 프레임워크를 가볍고 범용적으로 유지하는 대신, 특정 AI 분야에 필요한 기능들을 별도의 공식 라이브러리로 분리하여 제공하는 모듈식 생태계 전략을 채택했다. 이는 각 도메인의 전문가들이 독립적으로 빠르게 발전할 수 있는 환경을 제공하며, 사용자는 필요한 라이브러리만 선택적으로 사용할 수 있다.

- **`TorchVision`:** 컴퓨터 비전(Computer Vision) 분야를 위한 필수 라이브러리다. CIFAR, ImageNet과 같은 유명 데이터셋을 쉽게 다운로드하고 사용할 수 있는 API, ResNet, VGG, Vision Transformer 등 사전 학습된(pre-trained) 최신 모델 아키텍처, 그리고 이미지 자르기, 회전, 정규화 등 다양한 이미지 전처리 및 데이터 증강을 위한 변환(transform) 유틸리티를 종합적으로 제공한다.20
- **`TorchText`:** 자연어 처리(NLP)를 위한 라이브러리로, 텍스트 데이터를 모델이 처리할 수 있는 텐서 형태로 변환하는 복잡한 전처리 파이프라인 구축을 돕는다. 텍스트 데이터셋 로딩, 문장을 단어로 나누는 토큰화(tokenization), 단어와 정수 인덱스를 매핑하는 단어장(vocabulary) 구축, 그리고 배치 생성 등의 기능을 지원한다.20
- **`TorchAudio`:** 오디오 및 음성 처리를 위한 라이브러리다. WAV, MP3 등 다양한 형식의 오디오 파일을 로드하고 저장하는 기능, 오디오 파형을 주파수 영역으로 변환하는 스펙트로그램(Spectrogram) 및 MFCC(Mel-Frequency Cepstral Coefficients) 생성, 그리고 오디오 데이터 증강을 위한 다양한 변환 함수와 사전 학습된 모델을 제공한다.20


PyTorch는 매우 유연하지만, 학습 루프, 분산 학습 설정, 하드웨어(GPU/TPU) 관리, 로깅 등 모든 것을 사용자가 직접 구현해야 하므로 반복적인 상용구(boilerplate) 코드가 많아지는 경향이 있다. PyTorch Lightning은 이러한 문제를 해결하기 위해 등장한 인기 있는 상위 수준 프레임워크다.103

Lightning은 PyTorch 코드의 구조를 표준화하여 '과학(science)' 코드와 '공학(engineering)' 코드를 분리한다.105 사용자는 `LightningModule`이라는 클래스에 모델 아키텍처, 학습 단계(`training_step`), 검증 단계(`validation_step`), 옵티마이저 설정 등 연구의 핵심 로직에만 집중하여 코드를 작성하면 된다. 그러면 `Trainer` 클래스가 다중 GPU 학습, 16비트 혼합 정밀도(mixed precision) 학습, 체크포인팅, 로깅 등 복잡한 엔지니어링 작업을 모두 자동으로 처리해준다.103 이를 통해 코드의 가독성, 재사용성, 재현성이 크게 향상되어 연구와 개발의 생산성을 높일 수 있다.105


대규모 언어 모델(Large Language Models, LLM) 시대의 도래는 PyTorch의 '연구 중심'이라는 정체성을 오히려 가장 강력한 '산업 경쟁력'으로 전환시켰다. 과거에는 '연구용 PyTorch'와 '산업용 TensorFlow'라는 이분법이 통용되었지만, LLM의 등장은 최첨단 연구 결과물이 곧바로 산업의 핵심 동력이 되는 시대를 열었다.108

대부분의 선구적인 LLM 연구가 PyTorch의 유연성과 강력한 분산 학습 기능(특히 FSDP) 위에서 이루어졌기 때문에, Meta의 Llama 시리즈 110, Mistral AI의 모델들 112, 그리고 수많은 오픈소스 LLM들이 자연스럽게 PyTorch를 기반으로 학습되고 공개되었다. Hugging Face Transformers 라이브러리와 같은 LLM 생태계의 핵심 도구들 역시 PyTorch를 중심으로 발전했으며, Hugging Face Hub에 등록된 모델의 90% 이상이 PyTorch 기반이라는 사실은 PyTorch의 압도적인 지배력을 보여준다.11

이제 기업들은 TensorFlow로 SOTA 모델을 재구현하는 대신, PyTorch 기반의 모델을 직접 파인튜닝하고 배포하는 것을 선호하게 되었다. `torch.compile`과 FSDP의 결합은 LLM 학습 효율을 더욱 끌어올리고 있으며 114, vLLM과 같은 고성능 추론 엔진이 PyTorch 생태계에 통합되면서 111 연구부터 배포까지의 전 과정이 PyTorch 내에서 완결되는 선순환 구조가 강화되고 있다.


PyTorch는 현재 AI 프레임워크 시장에서 가장 강력한 위치를 점하고 있지만, 여전히 TensorFlow, JAX와 같은 강력한 경쟁자들과 공존하고 있다.

- **TensorFlow:** Google의 지원을 받는 TensorFlow는 여전히 산업계, 특히 대규모 프로덕션 환경에서 강점을 보인다. TensorFlow Serving을 통한 안정적인 서빙, TFLite를 이용한 모바일 및 엣지 디바이스 배포, 그리고 TFX(TensorFlow Extended)를 통한 성숙한 MLOps 파이프라인 구축 능력은 TensorFlow의 차별화된 경쟁력이다.11 하지만 연구 커뮤니티에서의 영향력 감소와 가파른 학습 곡선은 약점으로 지적된다.9
- **JAX:** Google이 개발한 JAX는 NumPy와 유사한 API에 자동 미분과 XLA(Accelerated Linear Algebra)를 통한 JIT 컴파일을 결합한 고성능 수치 연산 라이브러리다.116 특히 TPU에서의 압도적인 성능과 `vmap`, `pmap`과 같은 함수 변환을 통한 우아한 병렬화 기능은 과학 계산 및 차세대 AI 연구 분야에서 큰 주목을 받고 있다.119 그러나 아직 생태계가 상대적으로 작고, 순수 함수형 프로그래밍 패러다임에 익숙해져야 하므로 학습 곡선이 가파르다는 단점이 있다.118

**미래 로드맵:** 매년 개최되는 PyTorch 컨퍼런스와 공식 발표들을 통해 PyTorch의 미래 방향성을 엿볼 수 있다. 향후 로드맵은 다음과 같은 핵심 분야에 집중될 것으로 보인다: (1) **컴파일러 성능 강화**: `torch.compile`의 안정성을 높이고 그래프 분할을 최소화하며, 동적 shape 지원을 강화하여 더 많은 모델에서 성능 향상을 이끌어낸다. (2) **이기종 하드웨어 지원 확대**: `torch.accelerator`라는 통합 API를 통해 NVIDIA GPU뿐만 아니라 다양한 AI 칩과 가속기를 원활하게 지원하는 생태계를 구축한다. (3) **분산 학습 및 추론 최적화**: FSDP v2와 같은 차세대 분산 학습 기술을 도입하고, vLLM과의 긴밀한 통합을 통해 LLM의 학습 및 추론 성능을 지속적으로 개선한다. (4) **글로벌 커뮤니티 확장**: 중국을 포함한 전 세계 개발자 커뮤니티와의 협력을 강화하고, 현지화된 자료와 이벤트를 통해 생태계를 확장한다.122

| 특징 (Feature)    | PyTorch                                            | TensorFlow                                               | JAX                                               |
| ----------------- | -------------------------------------------------- | -------------------------------------------------------- | ------------------------------------------------- |
| **핵심 패러다임** | 동적 그래프 (Eager) + JIT 컴파일 (`torch.compile`) | 동적 그래프 (Eager) + 정적 그래프 (Keras, `tf.function`) | 함수형 프로그래밍 + JIT 컴파일 (XLA)              |
| **주요 강점**     | 연구 유연성, 직관적 API, 강력한 LLM 생태계         | 프로덕션 배포, MLOps, 모바일/엣지(TFLite)                | 고성능 컴퓨팅(TPU), 뛰어난 병렬화(`vmap`, `pmap`) |
| **디버깅**        | 매우 쉬움 (Pythonic)                               | 쉬움 (Eager 모드)                                        | 상대적으로 어려움 (JIT 컴파일 추적 필요)          |
| **생태계 성숙도** | 매우 높음 (Hugging Face 등 커뮤니티 주도)          | 매우 높음 (Google 주도, 엔드투엔드 솔루션)               | 성장 중 (연구 커뮤니티 중심)                      |
| **주요 사용자층** | 연구자, 학계, LLM 개발자, 스타트업                 | 대기업, 프로덕션 환경, MLOps 엔지니어                    | 고성능 컴퓨팅 연구자, Google/DeepMind             |
| **미래 방향성**   | 컴파일러 성능 강화, 이기종 하드웨어 지원           | 통합 ML 플랫폼, 엔터프라이즈 솔루션 강화                 | 차세대 AI 연구를 위한 고성능 컴퓨팅               |

결론적으로 PyTorch는 개발자 친화적인 철학을 바탕으로 연구 커뮤니티를 장악했으며, 이를 발판으로 LLM 시대의 산업 표준으로까지 도약했다. `torch.compile`을 통한 성능 혁신과 개방적인 재단 거버넌스를 통해, PyTorch는 앞으로도 AI 기술의 발전을 이끄는 핵심 동력으로 기능할 것이다.


1. The Complete History and Evolution of PyTorch | Deep Learning Framework Timeline, 8월 24, 2025에 액세스, https://tensorgym.com/blog/pytorch-history
2. tensorgym.com, 8월 24, 2025에 액세스, [https://tensorgym.com/blog/pytorch-history#:~:text=PyTorch's%20journey%20began%20in%202016,Python%2Dbased%20deep%20learning%20framework.](https://tensorgym.com/blog/pytorch-history#:~:text=PyTorch's journey began in 2016,Python-based deep learning framework.)
3. PyTorch - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/PyTorch
4. What is PyTorch? - IBM, 8월 24, 2025에 액세스, https://www.ibm.com/think/topics/pytorch
5. What is PyTorch? | Data Science | NVIDIA Glossary, 8월 24, 2025에 액세스, https://www.nvidia.com/en-us/glossary/pytorch/
6. PyTorch: all about Facebook's Deep Learning framework - DataScientest, 8월 24, 2025에 액세스, https://datascientest.com/en/pytorch-all-about-this-framework
7. PyTorch Design Philosophy, 8월 24, 2025에 액세스, https://pytorch.org/docs/stable/community/design.html
8. Learning PyTorch with Examples — PyTorch Tutorials 2.8.0+cu128 documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/tutorials/beginner/pytorch_with_examples.html
9. PyTorch vs TensorFlow: Which Deep Learning Framework is Better in 2025 | DigitalOcean, 8월 24, 2025에 액세스, https://www.digitalocean.com/community/tutorials/pytorch-vs-tensorflow
10. www.codecademy.com, 8월 24, 2025에 액세스, [https://www.codecademy.com/article/pytorch-vs-tensorflow-choosing-the-best-framework-for-deep-learning#:~:text=PyTorch%20has%20a%20Pythonic%20syntax,be%20made%20on%20the%20go.](https://www.codecademy.com/article/pytorch-vs-tensorflow-choosing-the-best-framework-for-deep-learning#:~:text=PyTorch has a Pythonic syntax,be made on the go.)
11. Deep Learning Framework Showdown: PyTorch vs TensorFlow in 2025 - MarkTechPost, 8월 24, 2025에 액세스, https://www.marktechpost.com/2025/08/20/deep-learning-framework-showdown-pytorch-vs-tensorflow-in-2025/
12. Define-and-Run vs. Define-by-Run - Medium, 8월 24, 2025에 액세스, https://medium.com/@zzemb6/define-and-run-vs-define-by-run-b527d127e13a
13. Comprehensive Guide to PyTorch: Theory and Coding Notes | by Ishan Rastogi | Medium, 8월 24, 2025에 액세스, https://medium.com/@ishanrastogi26/comprehensive-guide-to-pytorch-theory-and-coding-notes-7fc2ed80e84f
14. Understanding PyTorch's Dynamic Computational Graphs | by ServerWala InfraNet FZ-LLC, 8월 24, 2025에 액세스, https://medium.com/@serverwalainfra/understanding-pytorchs-dynamic-computational-graphs-bf77ee51e5c8
15. Dynamic vs Static Computational Graphs - PyTorch and TensorFlow - GeeksforGeeks, 8월 24, 2025에 액세스, https://www.geeksforgeeks.org/deep-learning/dynamic-vs-static-computational-graphs-pytorch-and-tensorflow/
16. Section 5 (Week 5) - CS230 Deep Learning - Stanford University, 8월 24, 2025에 액세스, https://cs230.stanford.edu/section/5/
17. Static vs Dynamic Computational Graphs | by Abhishek Jain - Medium, 8월 24, 2025에 액세스, https://medium.com/@abhishekjainindore24/static-vs-dynamic-computational-graphs-5a49d1e3030b
18. PyTorch vs. TensorFlow: A Comprehensive Comparison - Rafay, 8월 24, 2025에 액세스, https://rafay.co/the-kubernetes-current/pytorch-vs-tensorflow-a-comprehensive-comparison/
19. A Comparative Survey of PyTorch vs TensorFlow for Deep Learning: Usability, Performance, and Deployment Trade-offs - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2508.04035v1
20. PyTorch: Origin, Evolution, and Core Features | by Alok Choudhary | Aug, 2025 - Medium, 8월 24, 2025에 액세스, https://medium.com/@alok05/pytorch-origin-evolution-and-core-features-a5545192a260
21. How Facebook's Development of PyTorch Revolutionized Personal AI Development, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/380850284_How_Facebook's_Development_of_PyTorch_Revolutionized_Personal_AI_Development
22. Tensors — PyTorch Tutorials 2.8.0+cu128 documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/tutorials/beginner/basics/tensorqs_tutorial.html
23. PyTorch Tensor vs NumPy Array - GeeksforGeeks, 8월 24, 2025에 액세스, https://www.geeksforgeeks.org/deep-learning/pytorch-tensor-vs-numpy-array/
24. 0. PyTorch Fundamentals, 8월 24, 2025에 액세스, https://www.learnpytorch.io/00_pytorch_fundamentals/
25. Autograd mechanics — PyTorch 2.8 documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/docs/stable/notes/autograd.html
26. AutoGrad Mechanics in PyTorch - Kaggle, 8월 24, 2025에 액세스, https://www.kaggle.com/code/aisuko/autograd-mechanics-in-pytorch
27. Tensors — PyTorch Tutorials 2.8.0+cu128 documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/tutorials/beginner/blitz/tensor_tutorial.html
28. PyTorch CUDA vs Numpy for arithmetic operations? Fastest? - Stack Overflow, 8월 24, 2025에 액세스, https://stackoverflow.com/questions/52526082/pytorch-cuda-vs-numpy-for-arithmetic-operations-fastest
29. PyTorch memory model: "torch.from_numpy()" vs "torch.Tensor()" - Stack Overflow, 8월 24, 2025에 액세스, https://stackoverflow.com/questions/48482787/pytorch-memory-model-torch-from-numpy-vs-torch-tensor
30. Understanding Autograd in PyTorch: The Core of Automatic Differentiation - Medium, 8월 24, 2025에 액세스, https://medium.com/@sahin.samia/understanding-autograd-in-pytorch-the-core-of-automatic-differentiation-ca9d98da980d
31. 3.4 Automatic Differentiation in PyTorch - Lightning AI, 8월 24, 2025에 액세스, https://lightning.ai/courses/deep-learning-fundamentals/3-0-overview-model-training-in-pytorch/3-4-automatic-differentiation-in-pytorch/
32. A Gentle Introduction to torch.autograd - PyTorch documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/tutorials/beginner/blitz/autograd_tutorial.html
33. PyTorch Autograd automatic differentiation feature - Stack Overflow, 8월 24, 2025에 액세스, https://stackoverflow.com/questions/51054627/pytorch-autograd-automatic-differentiation-feature
34. Introduction to PyTorch Modules and Layers | by ServerWala InfraNet FZ-LLC | Medium, 8월 24, 2025에 액세스, https://medium.com/@serverwalainfra/introduction-to-pytorch-modules-and-layers-9a30527df871
35. Build the Neural Network — PyTorch Tutorials 2.8.0+cu128 documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/tutorials/beginner/basics/buildmodel_tutorial.html
36. Initializing a Neural Network Model in PyTorch | CodeSignal Learn, 8월 24, 2025에 액세스, https://codesignal.com/learn/courses/building-a-neural-network-in-pytorch/lessons/initializing-a-neural-network-model-in-pytorch
37. Create Model using Custom Module in Pytorch - GeeksforGeeks, 8월 24, 2025에 액세스, https://www.geeksforgeeks.org/machine-learning/create-model-using-custom-module-in-pytorch/
38. PyTorch 101: Learn Deep Learning with PyTorch | DigitalOcean, 8월 24, 2025에 액세스, https://www.digitalocean.com/community/tutorials/pytorch-101-advanced
39. Modules — PyTorch 2.8 documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/docs/stable/notes/modules.html
40. 1. PyTorch Workflow Fundamentals, 8월 24, 2025에 액세스, https://www.learnpytorch.io/01_pytorch_workflow/
41. PyTorch Tutorial - GeeksforGeeks, 8월 24, 2025에 액세스, https://www.geeksforgeeks.org/deep-learning/pytorch-learn-with-examples/
42. How to Learn PyTorch From Scratch in 2025: An Expert Guide | DataCamp, 8월 24, 2025에 액세스, https://www.datacamp.com/blog/how-to-learn-pytorch
43. Datasets & DataLoaders — PyTorch Tutorials 2.8.0+cu128 documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/tutorials/beginner/basics/data_tutorial.html
44. PyTorch DataLoader Tutorial: Master torch.utils.data.DataLoader | Codecademy, 8월 24, 2025에 액세스, https://www.codecademy.com/article/how-to-use-pytorch-dataloader-custom-datasets-transformations-and-efficient-techniques
45. A Guide to the DataLoader Class and Abstractions in PyTorch | DigitalOcean, 8월 24, 2025에 액세스, https://www.digitalocean.com/community/tutorials/dataloaders-abstractions-pytorch
46. Pytorch Basics : Efficient data management with Dataset and Dataloader - Medium, 8월 24, 2025에 액세스, https://medium.com/@manindersingh120996/pytorch-basics-efficient-data-management-with-dataset-and-dataloader-e3aaebe61681
47. Training a Classifier — PyTorch Tutorials 2.8.0+cu128 documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/tutorials/beginner/blitz/cifar10_tutorial.html
48. PyTorch Loss Functions: The Ultimate Guide - neptune.ai, 8월 24, 2025에 액세스, https://neptune.ai/blog/pytorch-loss-functions
49. Beginner's Guide to Pytorch Loss Functions | Zero To Mastery, 8월 24, 2025에 액세스, https://zerotomastery.io/blog/pytorch-loss-functions/
50. Loss vs Loss Function vs Criterion in PyTorch | by Hey Amit - Medium, 8월 24, 2025에 액세스, https://medium.com/@heyamit10/loss-vs-loss-function-vs-criterion-in-pytorch-5f446afdb63c
51. SGD — PyTorch 2.8 documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/docs/stable/generated/torch.optim.SGD.html
52. ADAM optimization with PyTorch - AI Mind, 8월 24, 2025에 액세스, https://pub.aimind.so/adam-optimization-with-pytorch-a76195c85587
53. What is the difference between DataParallel and DistributedDataParallel? - PyTorch Forums, 8월 24, 2025에 액세스, https://discuss.pytorch.org/t/what-is-the-difference-between-dataparallel-and-distributeddataparallel/6108
54. How does pytorch's parallel method and distributed method work? - Stack Overflow, 8월 24, 2025에 액세스, https://stackoverflow.com/questions/53375422/how-does-pytorchs-parallel-method-and-distributed-method-work
55. Distributed and parallel training... explained - Part 1 (2020) - fast.ai Course Forums, 8월 24, 2025에 액세스, https://forums.fast.ai/t/distributed-and-parallel-training-explained/73892
56. Getting Started with Distributed Data Parallel - PyTorch documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/tutorials/intermediate/ddp_tutorial.html
57. DistributedDataParallel slower than DataParallel? - PyTorch Forums, 8월 24, 2025에 액세스, https://discuss.pytorch.org/t/distributeddataparallel-slower-than-dataparallel/152912
58. Introducing PyTorch Fully Sharded Data Parallel (FSDP) API, 8월 24, 2025에 액세스, https://pytorch.org/blog/introducing-pytorch-fully-sharded-data-parallel-api/
59. Scale LLMs with PyTorch 2.0 FSDP on Amazon EKS – Part 2 | Artificial Intelligence - AWS, 8월 24, 2025에 액세스, https://aws.amazon.com/blogs/machine-learning/scale-llms-with-pytorch-2-0-fsdp-on-amazon-eks-part-2/
60. PyTorch FSDP: Experiences on Scaling Fully Sharded Data Parallel - VLDB Endowment, 8월 24, 2025에 액세스, https://www.vldb.org/pvldb/vol16/p3848-huang.pdf
61. FullyShardedDataParallel — PyTorch 2.8 documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/docs/stable/fsdp.html
62. SimpleFSDP: Simpler Fully Sharded Data Parallel with torch.compile - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2411.00284v1
63. The Practical Guide to Distributed Training using PyTorch — Part 5: Distributing Everything using FSDP | by Siladittya Manna | The Owl | Jul, 2025 | Medium, 8월 24, 2025에 액세스, https://medium.com/the-owl/the-practical-guide-to-distributed-training-using-pytorch-part-5-distributing-everything-using-a6a0b8e83bb9
64. DDP vs FSDP in PyTorch: Unlocking Efficient Multi-GPU Training - Jellyfish Technologies, 8월 24, 2025에 액세스, https://www.jellyfishtechnologies.com/ddp-vs-fsdp-in-pytorch-unlocking-efficient-multi-gpu-training/
65. Efficient Large-Scale Training with Pytorch FSDP and AWS, 8월 24, 2025에 액세스, https://pytorch.org/blog/efficient-large-scale-training-with-pytorch/
66. [D] Pytorch FSDP is pipeline parallelism right? : r/MachineLearning - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/MachineLearning/comments/1bqsq3w/d_pytorch_fsdp_is_pipeline_parallelism_right/
67. Advanced Model Training with Fully Sharded Data Parallel (FSDP) - PyTorch documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/tutorials/intermediate/FSDP_advanced_tutorial.html
68. Advanced Model Training with Fully Sharded Data Parallel (FSDP) - PyTorch, 8월 24, 2025에 액세스, https://pytorch.org/tutorials/intermediate/FSDP_adavnced_tutorial.html
69. Getting Started with Fully Sharded Data Parallel (FSDP2) - PyTorch documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/tutorials/intermediate/FSDP_tutorial.html
70. Accelerated PyTorch training on Mac - Metal - Apple Developer, 8월 24, 2025에 액세스, https://developer.apple.com/metal/pytorch/
71. Introducing Accelerated PyTorch Training on Mac, 8월 24, 2025에 액세스, https://pytorch.org/blog/introducing-accelerated-pytorch-training-on-mac/
72. Accelerated PyTorch Training on Mac - Hugging Face, 8월 24, 2025에 액세스, https://huggingface.co/docs/accelerate/usage_guides/mps
73. Tracing and Scripting in ML Workflows: PyTorch, TorchScript, and JIT-ed Models, 8월 24, 2025에 액세스, https://gganbumarketplace.com/machine-learning/tracing-and-scripting-in-ml-workflows-pytorch-torchscript-and-jit-ed-models/
74. What are Torch Scripts in PyTorch? - GeeksforGeeks, 8월 24, 2025에 액세스, https://www.geeksforgeeks.org/deep-learning/what-are-torch-scripts-in-pytorch/
75. PyTorch - AI at Meta, 8월 24, 2025에 액세스, https://ai.meta.com/tools/pytorch/
76. TorchScript — PyTorch 2.7 documentation, 8월 24, 2025에 액세스, https://pytorch.org/docs/stable/jit.html
77. TorchScript - Hugging Face, 8월 24, 2025에 액세스, https://huggingface.co/docs/transformers/torchscript
78. When should I use Tracing rather than Scripting? - jit - PyTorch Forums, 8월 24, 2025에 액세스, https://discuss.pytorch.org/t/when-should-i-use-tracing-rather-than-scripting/53883
79. Mastering TorchScript: Tracing vs Scripting, Device Pinning, Direct Graph Modification, 8월 24, 2025에 액세스, https://paulbridger.com/posts/mastering-torchscript/
80. Model Serving in PyTorch: An Expert's Guide to Efficient Deployment | by Hey Amit - Medium, 8월 24, 2025에 액세스, https://medium.com/@heyamit10/model-serving-in-pytorch-an-experts-guide-to-efficient-deployment-3ce43343b3a0
81. Model Deployment and Serving - KodeKloud Notes, 8월 24, 2025에 액세스, https://notes.kodekloud.com/docs/Fundamentals-of-MLOps/Model-Deployment-and-Serving/Model-Deployment-and-Serving
82. Serve, optimize and scale PyTorch models in production - GitHub, 8월 24, 2025에 액세스, https://github.com/pytorch/serve
83. TorchServe — PyTorch/Serve master documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/serve/
84. Serve scalable LLMs on GKE with TorchServe | Kubernetes Engine - Google Cloud, 8월 24, 2025에 액세스, https://cloud.google.com/kubernetes-engine/docs/tutorials/scalable-ml-models-torchserve
85. PyTorch 2.x, 8월 24, 2025에 액세스, https://pytorch.org/get-started/pytorch-2-x/
86. PyTorch 2.0: Our next generation release that is faster, more Pythonic and Dynamic as ever, 8월 24, 2025에 액세스, https://pytorch.org/blog/pytorch-2-0-release/
87. Understanding PyTorch Eager and Graph Mode | by Hey Amit - Medium, 8월 24, 2025에 액세스, https://medium.com/@heyamit10/understanding-pytorch-eager-and-graph-mode-e0b81c1f2396
88. Pytorch 2.x Eager mode vs Graph mode | by Mohamed Amine Dhiab | Medium, 8월 24, 2025에 액세스, https://medium.com/@mohameddhiab/pytorch-2-x-eager-mode-vs-graph-mode-c942d7040042
89. torch.compiler — PyTorch 2.8 documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/docs/stable/torch.compiler.html
90. Introduction to torch.compile — PyTorch Tutorials 2.8.0+cu128 documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/tutorials/intermediate/torch_compile_tutorial.html
91. PyTorch 2.0 - HotChips 2023, 8월 24, 2025에 액세스, [https://www.hc2023.hotchips.org/assets/program/tutorials/ml/PyTorch%202.0.pdf](https://www.hc2023.hotchips.org/assets/program/tutorials/ml/PyTorch 2.0.pdf)
92. Maximizing AI/ML Model Performance with PyTorch Compilation - Towards Data Science, 8월 24, 2025에 액세스, https://towardsdatascience.com/maximizing-ai-ml-model-performance-with-pytorch-compilation/
93. Unleashing PyTorch Performance with TorchInductor: A Deep Dive | by Aadishagrawal, 8월 24, 2025에 액세스, https://medium.com/@aadishagrawal/unleashing-pytorch-performance-with-torchinductor-a-deep-dive-1f01e8b36efa
94. Benchmark of the newly launched PYTORCH 2.0 : r/learnmachinelearning - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/learnmachinelearning/comments/zd6krn/benchmark_of_the_newly_launched_pytorch_20/
95. torchvision: Models, Datasets and Transformations for Images - mlverse - R-universe, 8월 24, 2025에 액세스, https://mlverse.r-universe.dev/torchvision
96. pytorch/vision: Datasets, Transforms and Models specific to Computer Vision - GitHub, 8월 24, 2025에 액세스, https://github.com/pytorch/vision
97. torchtext.data - PyTorch documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/text/0.8.1/data.html
98. Text classification with the torchtext library - h-huang.github.io, 8월 24, 2025에 액세스, https://h-huang.github.io/tutorials/beginner/text_sentiment_ngrams_tutorial.html
99. Torchtext 0.4 with Supervised Learning Datasets: A Quick Introduction by George Zhang, 8월 24, 2025에 액세스, https://www.youtube.com/watch?v=SpSOOfVZ_ss
100. Introduction to Torchaudio in PyTorch | Scaler Topics, 8월 24, 2025에 액세스, https://www.scaler.com/topics/pytorch/torchaudio-in-pytorch/
101. faroit/torchaudio: simple audio I/O for pytorch - GitHub, 8월 24, 2025에 액세스, https://github.com/faroit/torchaudio
102. Data manipulation and transformation for audio signal processing, powered by PyTorch - GitHub, 8월 24, 2025에 액세스, https://github.com/pytorch/audio
103. Getting Started with PyTorch Lightning: Build and Train Models | Codecademy, 8월 24, 2025에 액세스, https://www.codecademy.com/article/guide-to-py-torch-lightning
104. Lightning-AI/pytorch-lightning: Pretrain, finetune ANY AI model of ANY size on multiple GPUs, TPUs with zero code changes. - GitHub, 8월 24, 2025에 액세스, https://github.com/Lightning-AI/pytorch-lightning
105. Why Should I Use PyTorch Lightning? | by Aaron (Ari) Bornstein, 8월 24, 2025에 액세스, https://devblog.pytorchlightning.ai/why-should-i-use-pytorch-lightning-488760847b8b
106. Use Pytorch Lightning to Decouple Science science code from engineering code - Medium, 8월 24, 2025에 액세스, https://medium.com/analytics-vidhya/use-pytorch-lightning-to-decouple-science-science-code-from-engineering-code-1e6f36cf6a1a
107. PyTorch Lightning: A Comprehensive Hands-On Tutorial - DataCamp, 8월 24, 2025에 액세스, https://www.datacamp.com/tutorial/pytorch-lightning-tutorial
108. Key Differences Between TensorFlow and PyTorch for Deep Learning | by Krish - Medium, 8월 24, 2025에 액세스, https://medium.com/@krishtech/key-differences-between-tensorflow-and-pytorch-for-deep-learning-3f2351bfbb72
109. Pytorch vs Tensorflow: The Ultimate Decision Guide - V7 Labs, 8월 24, 2025에 액세스, https://www.v7labs.com/blog/pytorch-vs-tensorflow
110. Together AI – The AI Acceleration Cloud - Fast Inference, Fine-Tuning & Training, 8월 24, 2025에 액세스, https://www.together.ai/
111. vLLM Joins PyTorch Ecosystem: Easy, Fast, and Cheap LLM Serving for Everyone, 8월 24, 2025에 액세스, https://pytorch.org/blog/vllm-joins-pytorch/
112. Finetuning Llama 2 and Mistral - Medium, 8월 24, 2025에 액세스, https://medium.com/@geronimo7/finetuning-llama2-mistral-945f9c200611
113. PyTorch vs TensorFlow in 2023 - AssemblyAI, 8월 24, 2025에 액세스, https://www.assemblyai.com/blog/pytorch-vs-tensorflow-in-2023
114. pytorch/torchtitan: A PyTorch native platform for training generative AI models - GitHub, 8월 24, 2025에 액세스, https://github.com/pytorch/torchtitan
115. Maximizing Training Throughput Using PyTorch FSDP and Torch.compile, 8월 24, 2025에 액세스, https://pytorch.org/blog/maximizing-training-throughput/
116. ML Engineer comparison of Pytorch, TensorFlow, JAX, and Flax - SoftwareMill, 8월 24, 2025에 액세스, https://softwaremill.com/ml-engineer-comparison-of-pytorch-tensorflow-jax-and-flax/
117. PyTorch vs TensorFlow vs JAX: The Ultimate Comparison - YouTube, 8월 24, 2025에 액세스, https://www.youtube.com/watch?v=mf2oCBeg7T8
118. JAX vs. PyTorch: Differences and Similarities [2025] - Geekflare, 8월 24, 2025에 액세스, https://geekflare.com/dev/jax-vs-pytorch/
119. JAX vs PyTorch: A simple transformer benchmark - Echo Nolan's Blog, 8월 24, 2025에 액세스, https://www.echonolan.net/posts/2021-09-06-JAX-vs-PyTorch-A-Transformer-Benchmark.html
120. Comparing PyTorch and JAX - DigitalOcean, 8월 24, 2025에 액세스, https://www.digitalocean.com/community/tutorials/pytorch-vs-jax
121. [D] Is it worth switching to JAX from TensorFlow/PyTorch? : r/MachineLearning - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/MachineLearning/comments/1b08qv6/d_is_it_worth_switching_to_jax_from/
122. PyTorch Day China Recap, 8월 24, 2025에 액세스, https://pytorch.org/blog/pytorch-day-china-recap/
123. PyTorch Conference | LF Events, 8월 24, 2025에 액세스, https://events.linuxfoundation.org/pytorch-conference/
124. PyTorch Conference 2025 Schedule Announcement, 8월 24, 2025에 액세스, https://pytorch.org/blog/pytorch-conference-2025-schedule-announcement/