# PyTorch 학습서 (기초부터 모델 배포까지)
[Pytorch](./index.md)


PyTorch는 유연성과 속도를 핵심 가치로 삼는 오픈소스 머신러닝 프레임워크이다. 이는 두 가지 주요 기능을 통해 구현된다: 첫째, NumPy와 유사한 인터페이스를 가지면서도 강력한 GPU 가속을 지원하는 N차원 텐서(Tensor) 연산 기능이다.1 둘째, 연산 기록을 동적으로 추적하는 테이프 기반 자동 미분 시스템 위에 구축된 심층 신경망 라이브러리이다.1 이러한 설계는 PyTorch가 연구자들에게는 아이디어를 빠르고 유연하게 구현할 수 있는 실험 플랫폼을 제공하고, 개발자들에게는 GPU의 성능을 최대한 활용하는 고성능 계산 도구를 제공하게 한다.1

대부분의 머신러닝 워크플로우는 공통적인 단계를 따른다: (1) 데이터를 준비하고 불러오는 단계, (2) 데이터를 처리할 모델을 설계하고 생성하는 단계, (3) 모델의 예측과 실제 값의 차이를 줄이도록 파라미터를 최적화하는 단계, 그리고 (4) 학습이 완료된 모델을 저장하여 추후에 사용하는 단계이다.4 본 학습서는 이 전체 과정을 PyTorch를 사용하여 체계적으로 구현하는 방법을 안내한다. 예제로는 의류 이미지를 10개의 클래스(예: T-shirt, Trouser, Pullover) 중 하나로 분류하는 FashionMNIST 데이터셋을 활용하여, 데이터 준비부터 모델 저장까지의 전 과정을 단계별로 상세히 다룰 것이다.4



텐서(Tensor)는 PyTorch의 가장 기본적인 데이터 구조로, 배열이나 행렬과 매우 유사한 다차원 데이터 배열이다. PyTorch에서는 모델의 입력 데이터, 출력 결과, 그리고 학습 대상인 모델의 파라미터(가중치와 편향)를 모두 텐서 형태로 인코딩하여 다룬다.5

개념적으로 PyTorch 텐서는 과학 컴퓨팅 분야의 표준 라이브러리인 NumPy의 `ndarray`와 거의 동일하다.2 하지만 결정적인 차이점이 존재하는데, 바로 텐서는 GPU나 TPU와 같은 하드웨어 가속기에서 연산을 수행하여 계산 속도를 극적으로 향상시킬 수 있다는 점이다.5 이러한 특징 덕분에 PyTorch는 대규모 데이터셋과 복잡한 모델의 연산을 효율적으로 처리할 수 있다.

PyTorch 텐서 API가 NumPy와 유사하게 설계된 것은 전략적인 결정이다. 과학 컴퓨팅 및 데이터 분석 분야의 수많은 개발자와 연구자들은 이미 NumPy에 익숙하다. PyTorch는 이들에게 친숙한 API를 제공함으로써 새로운 프레임워크 학습에 대한 진입 장벽을 크게 낮추었다.5 결과적으로 사용자들은 프레임워크 자체의 사용법을 익히는 데 시간을 쏟기보다, 해결하고자 하는 머신러닝 문제에 더 집중할 수 있게 된다.


PyTorch는 다양한 방식으로 텐서를 생성하는 함수를 제공한다.

- **데이터로부터 직접 생성:** 파이썬의 리스트(list)나 NumPy 배열을 `torch.tensor()` 함수에 전달하여 텐서를 생성할 수 있다. 이 경우 데이터 타입은 입력 데이터로부터 자동으로 추론된다.5
- **특정 값으로 채워진 텐서 생성:** `torch.ones(shape)`, `torch.zeros(shape)`, `torch.rand(shape)`와 같은 함수들은 주어진 `shape`(형태)에 따라 모든 요소가 각각 1, 0, 또는 0과 1 사이의 균등 분포 무작위 값으로 채워진 텐서를 생성한다.5 또한, `torch.ones_like(input_tensor)`와 같은 `_like` 함수들은 기존 텐서의 `shape`과 `dtype` 속성을 그대로 유지하면서 새로운 텐서를 생성할 때 유용하다.5
- **시퀀스 데이터 생성:** `torch.arange(start, end, step)`는 주어진 범위 내에서 일정한 간격(`step`)을 갖는 1차원 텐서를 생성하며, `torch.linspace(start, end, steps)`는 시작점과 끝점 사이를 `steps` 개수만큼 균등하게 나눈 값들로 텐서를 생성한다.7


모든 PyTorch 텐서는 세 가지 중요한 속성(attribute)을 가진다.

- `shape`: 텐서의 각 차원의 크기를 나타내는 튜플이다. 예를 들어, (2, 3) `shape`은 2개의 행과 3개의 열을 가진 2차원 텐서를 의미한다.5
- `dtype`: 텐서에 저장된 데이터의 타입을 나타낸다. `torch.float32`, `torch.int64` 등이 있으며, 연산의 정밀도와 메모리 사용량에 영향을 미친다.6
- `device`: 텐서가 저장된 물리적 장치를 나타낸다. 기본값은 `'cpu'`이며, GPU를 사용할 경우 `'cuda'`로 지정된다.5

이 중 `device` 속성은 단순한 메타데이터가 아니라 PyTorch의 핵심 가치인 하드웨어 가속을 실현하는 관문이다. 딥러닝의 방대한 연산은 CPU만으로는 비효율적이며, GPU를 통한 병렬 처리가 필수적이다. PyTorch는 `tensor.to('cuda')`라는 직관적인 메서드를 통해 복잡한 GPU 메모리 할당 및 데이터 전송 과정을 추상화한다.5 개발자는 저수준의 CUDA 프로그래밍 지식 없이도 이 간단한 호출만으로 GPU의 강력한 연산 능력을 활용할 수 있으며, 이것이 PyTorch가 "GPU를 지원하는 NumPy"로 불리는 근본적인 이유이다.1


PyTorch 텐서는 풍부한 연산 기능을 지원하며, 대부분 NumPy와 유사한 문법을 사용한다.

- **인덱싱 및 슬라이싱:** 파이썬의 표준 리스트나 NumPy 배열처럼 대괄호(``)를 사용하여 텐서의 특정 요소나 부분 집합에 접근하고 수정할 수 있다.5
- **산술 연산:** `+`, `-`, `*`, `/`와 같은 표준 산술 연산자를 사용하여 요소별(element-wise) 연산을 수행할 수 있다. `torch.add()`, `torch.mul()`과 같은 함수를 사용할 수도 있다.5
- **행렬 곱:** `@` 연산자 또는 `torch.matmul()` 함수를 사용하여 두 텐서 간의 행렬 곱셈을 수행한다. 이는 신경망의 선형 계층에서 핵심적인 연산이다.5
- **차원 변경:** `reshape()`이나 `view()` 메서드를 사용하여 텐서의 `shape`을 변경할 수 있다. `unsqueeze()`는 특정 위치에 크기가 1인 차원을 추가하고, `squeeze()`는 크기가 1인 차원을 제거하는 데 사용된다.10


PyTorch는 NumPy와의 상호 운용성을 매우 중요하게 생각하며, 둘 사이의 변환은 매우 효율적이다.

- **텐서에서 NumPy로:** CPU에 위치한 텐서에 대해 `.numpy()` 메서드를 호출하면 NumPy `ndarray`로 변환할 수 있다. 이 때 새로운 메모리가 할당되는 것이 아니라, 두 객체가 동일한 메모리 공간을 공유한다. 따라서 한쪽에서 값을 변경하면 다른 쪽에도 즉시 반영된다.5
- **NumPy에서 텐서로:** `torch.from_numpy()` 함수를 사용하여 NumPy 배열을 PyTorch 텐서로 변환할 수 있다. 이 경우에도 메모리가 공유된다.5

이러한 메모리 공유 방식은 데이터 복사에 따른 오버헤드를 제거하여, 기존 NumPy 기반의 데이터 처리 파이프라인과 PyTorch를 원활하게 통합할 수 있도록 돕는다.



`torch.autograd`는 PyTorch의 자동 미분(automatic differentiation) 엔진으로, 신경망 학습의 핵심인 역전파(backpropagation) 알고리즘을 구동한다.12 신경망 학습은 크게 두 단계로 이루어진다. 첫 번째는 순방향 전파(forward propagation)로, 입력 데이터가 네트워크를 통과하여 예측 값을 생성하는 과정이다. 두 번째는 역방향 전파로, 예측 값과 실제 값 사이의 오차(loss)를 기반으로 모델의 파라미터(가중치와 편향)를 어떻게 조정해야 할지 결정하는 과정이다. 이 조정을 위해 각 파라미터에 대한 오차의 기울기(gradient)가 필요한데, `autograd`가 바로 이 기울기 계산을 자동으로 수행해준다.12


`autograd`는 텐서에 대한 연산이 실행될 때마다, 어떤 데이터(텐서)와 어떤 연산이 수행되었는지를 기록하는 계산 그래프를 생성한다.13 이 그래프는 방향성 비순환 그래프(Directed Acyclic Graph, DAG) 형태를 띠며, 입력 텐서가 잎(leaf) 노드가 되고 최종 출력 텐서가 루트(root) 노드가 된다.12

PyTorch의 계산 그래프는 '동적(dynamic)'이라는 매우 중요한 특징을 가진다. 이는 그래프가 코드가 실행되는 시점(runtime)에 즉석에서 생성된다는 의미이다.13 정적 그래프 방식의 프레임워크(예: 초기 버전의 TensorFlow)에서는 모델의 구조를 미리 정의하고 컴파일해야 했지만, PyTorch에서는 일반적인 파이썬 코드를 실행하듯이 모델을 실행하면 그 흐름에 따라 그래프가 자연스럽게 만들어진다.

이러한 동적 그래프 방식은 PyTorch가 연구 커뮤니티에서 폭발적인 지지를 얻은 핵심적인 이유이다. 연구 과정에서는 종종 입력에 따라 구조가 변하거나, 재귀적인 연산을 수행하거나, 조건부로 다른 경로를 타는 등 복잡하고 동적인 모델을 실험해야 한다. PyTorch는 파이썬의 `if`문, `for`문과 같은 네이티브 제어 흐름을 모델 코드에 그대로 사용할 수 있게 함으로써 이러한 요구를 완벽하게 충족시킨다.3 "코드가 곧 모델"이라는 이 직관적인 패러다임은 디버깅을 매우 용이하게 만든다. 개발자는 

`print()` 함수나 `pdb`와 같은 표준 파이썬 디버거를 사용하여 코드의 어느 지점에서든 중간 텐서 값을 확인하며 모델의 동작을 단계별로 추적할 수 있다.3 이러한 유연성과 투명성은 연구자들이 아이디어를 신속하게 프로토타이핑하고 검증하는 데 결정적인 이점을 제공한다.


`autograd`가 연산을 추적하게 하려면, 해당 텐서를 생성할 때 `requires_grad=True` 속성을 설정해야 한다.2 이 속성이 `True`로 설정된 텐서에 대해 연산이 수행되면, 그 결과로 생성되는 텐서는 `grad_fn`이라는 속성을 갖게 된다. `grad_fn`은 해당 텐서를 만들어낸 함수(연산)를 참조하며, 계산 그래프에서 노드 간의 연결고리 역할을 한다. 이를 통해 `autograd`는 연산의 흐름을 역으로 추적할 수 있다.14 입력 텐서와 같이 사용자가 직접 생성한 텐서는 `grad_fn`이 없다 (이들을 잎 노드라고 한다).


모델의 최종 출력인 손실(loss) 값은 스칼라 텐서이다. 이 손실 텐서에 대해 `.backward()` 메서드를 호출하면, `autograd` 엔진이 작동하기 시작한다.12 엔진은 `grad_fn`을 따라 계산 그래프를 루트에서부터 잎 노드 방향으로 거슬러 올라가며, 연쇄 법칙(chain rule)을 적용하여 그래프에 있는 모든 `requires_grad=True` 텐서에 대한 손실의 기울기를 계산한다.

계산된 기울기는 각 텐서의 `.grad` 속성에 누적(accumulate)되어 저장된다.2 옵티마이저는 이 `.grad` 값을 참조하여 모델의 파라미터를 업데이트하게 된다.

`backward()` 호출은 계산 그래프를 '소비'하는 행위로 이해할 수 있다. 순방향 전파 과정에서 `autograd`는 역전파 시 기울기 계산에 필요한 중간 값들(activations)을 메모리에 저장한다. `loss.backward()`가 호출되면, 이 저장된 값들을 사용하여 기울기를 계산하고, 계산이 완료되면 메모리 효율성을 위해 이 중간 값들을 해제한다. 만약 동일한 그래프에 대해 `backward()`를 두 번 호출하려고 하면, 필요한 중간 값들이 이미 사라졌기 때문에 `RuntimeError`가 발생한다. 이 '일회성 소비' 메커니즘을 이해하는 것은 PyTorch의 메모리 관리 방식을 파악하는 데 중요하며, 그래프를 유지해야 하는 특수한 경우(`retain_graph=True` 사용)에 발생하는 메모리 부담을 예측하는 데 도움이 된다.


모델을 평가(inference)하거나 검증(validation)할 때는 파라미터를 업데이트하지 않으므로 기울기를 계산할 필요가 없다. 이 경우 `with torch.no_grad():` 컨텍스트 관리자를 사용하면, 해당 블록 내에서 실행되는 모든 연산에 대한 추적을 일시적으로 비활성화할 수 있다.13 이는 불필요한 메모리 사용을 줄이고 계산 속도를 향상시키는 효과적인 방법이다.12



PyTorch에서 모든 신경망 모델은 `torch.nn.Module` 클래스를 상속받아 자신만의 클래스로 정의된다.17

`nn.Module`은 신경망을 구성하는 데 필요한 모든 기본 빌딩 블록(계층, 활성화 함수, 손실 함수 등)을 제공하는 핵심 컨테이너이다.17

PyTorch 모델의 중요한 특징은 중첩 구조(nested structure)이다. 즉, 하나의 신경망 모델(`nn.Module`)은 그 자체로 여러 개의 작은 모듈(예: `nn.Linear`, `nn.Conv2d`와 같은 계층)들로 구성된다. 이러한 계층적이고 모듈화된 설계 덕분에 복잡한 아키텍처도 체계적이고 직관적으로 구축하고 관리할 수 있다.17


`nn.Module`을 상속받는 커스텀 모델 클래스는 일반적으로 두 개의 핵심 메서드를 구현해야 한다.

- `__init__()`: 모델의 생성자(constructor)이다. 이 메서드 안에서 모델이 사용할 계층들을 초기화하고 정의한다. 예를 들어, `self.conv1 = nn.Conv2d(...)`와 같이 합성곱 계층이나 `self.fc1 = nn.Linear(...)`와 같이 완전 연결 계층을 클래스의 속성으로 생성한다. `__init__` 메서드의 첫 줄에서는 반드시 `super().__init__()`를 호출하여 부모 클래스인 `nn.Module`의 생성자를 실행해야 한다.17
- `forward()`: 모델의 순방향 전파 로직을 정의한다. 이 메서드는 입력 텐서 `x`를 인자로 받아, `__init__`에서 정의한 계층들을 순서대로 통과시킨 후 최종 출력 텐서를 반환한다. 모델 객체를 함수처럼 호출하면(예: `output = model(input_data)`), 내부적으로 이 `forward()` 메서드가 실행된다.17

`nn.Module`은 '상태를 가진 함수'라는 강력한 객체 지향적 추상화를 제공한다. `torch.nn.functional`이 순수한 수학적 연산(예: `F.relu`)을 제공하는 반면, `nn.Module`로 정의된 계층들은 내부에 학습 가능한 파라미터(가중치와 편향)라는 '상태(state)'를 캡슐화한다.3 예를 들어, `nn.Linear` 계층은 단순한 행렬 곱셈 연산이 아니라, 학습 과정에서 계속 변화하는 `weight`와 `bias`라는 상태를 내장한 객체이다. `nn.Module`은 이처럼 상태와 그 상태를 사용하는 연산(`forward` 로직)을 하나의 단위로 묶어 코드의 재사용성과 가독성을 높인다. 또한, `model.train()`과 `model.eval()`을 통해 모델의 동작 모드를 전환하는 기능 역시 이러한 상태 기반 설계의 중요한 예시이다.14


`torch.nn` 네임스페이스는 다양한 사전 정의된 계층을 제공하여 모델 구축을 용이하게 한다.

- `nn.Flatten`: 다차원 텐서를 1차원 벡터로 변환한다. 예를 들어, `(N, C, H, W)` 형태의 이미지 배치 텐서를 `(N, C*H*W)` 형태로 만들어, 합성곱 계층의 출력을 완전 연결 계층의 입력으로 전달할 때 주로 사용된다.17
- `nn.Linear`: 입력 데이터에 선형 변환 $y = xA^T + b$`를 적용한다. 완전 연결 계층(fully-connected layer)이라고도 불린다.17
- `nn.ReLU`: 대표적인 비선형 활성화 함수이다. 입력값이 0보다 작으면 0을, 0보다 크면 입력값을 그대로 출력한다. 선형 변환만으로는 표현할 수 없는 복잡한 함수를 학습할 수 있도록 모델에 비선형성을 도입하는 역할을 한다.17
- `nn.Sequential`: 여러 계층을 순서대로 담는 컨테이너 모듈이다. `__init__`에서 정의된 순서대로 입력 데이터가 각 계층을 통과한다. 간단한 순차적 구조의 모델을 빠르고 간결하게 정의할 때 매우 유용하다.17


`nn.Module`의 가장 강력한 기능 중 하나는 학습 가능한 파라미터를 자동으로 추적하고 관리하는 것이다. `__init__` 메서드 내에서 `nn.Linear`나 `nn.Conv2d`와 같은 `nn.Module`의 하위 클래스 객체를 클래스의 속성으로 할당하면, 해당 계층이 가진 모든 파라미터(가중치와 편향 텐서)가 부모 모듈에 자동으로 '등록'된다.3

이렇게 등록된 파라미터들은 `model.parameters()` 또는 `model.named_parameters()` 메서드를 통해 손쉽게 접근할 수 있다. 이 메서드들은 모델과 그 하위 모듈에 포함된 모든 학습 가능한 파라미터에 대한 반복자(iterator)를 반환한다.3

이 파라미터 자동 등록 기능은 `autograd`와 옵티마이저를 잇는 결정적인 연결고리 역할을 한다. 모델 학습의 목표는 손실 함수의 기울기를 계산하여(`autograd`) 파라미터를 업데이트하는(`optimizer`) 것이다. 이 과정이 원활하게 작동하려면, 옵티마이저는 어떤 텐서들이 업데이트 대상인지 정확히 알아야 한다. 개발자가 수백 개의 계층에 존재하는 모든 파라미터 텐서를 수동으로 목록화하여 옵티마이저에 전달하는 것은 매우 번거롭고 오류가 발생하기 쉽다. `nn.Module`은 이 과정을 자동화한다. `self.layer = nn.Linear(...)`와 같이 모듈을 속성으로 할당하는 순간, 해당 모듈의 모든 파라미터가 재귀적으로 수집된다. 따라서 개발자는 `optimizer = torch.optim.Adam(model.parameters(),...)` 단 한 줄의 코드로 전체 모델의 모든 학습 가능한 파라미터를 옵티마이저에 안전하고 편리하게 등록할 수 있다.


PyTorch의 아키텍처는 머신러닝 워크플로우의 핵심 관심사인 데이터 처리, 모델 구조, 학습 메커니즘을 체계적으로 분리하여 설계되었다. `Dataset`/`DataLoader`는 데이터 처리를, `nn.Module`은 모델 구조를, 그리고 손실 함수와 옵티마이저는 학습 메커니즘을 각각 책임진다. 이러한 명확한 책임 분리 덕분에 각 구성 요소를 독립적으로 교체하고 실험할 수 있어, 복잡한 프로젝트의 유연성과 재사용성이 극대화된다.


효율적인 데이터 처리는 성공적인 모델 학습의 기반이다. PyTorch는 이를 위해 `torch.utils.data.Dataset`과 `torch.utils.data.DataLoader`라는 두 가지 핵심 클래스를 제공한다.20

- **`Dataset`:** `Dataset`은 데이터와 그에 해당하는 레이블을 저장하고, 개별 데이터 샘플에 접근하는 방법을 정의하는 추상 클래스이다. 커스텀 데이터셋을 만들려면 `Dataset` 클래스를 상속받고, 데이터셋의 전체 크기를 반환하는 `__len__` 메서드와 특정 인덱스(`idx`)에 해당하는 데이터 샘플(보통 (데이터, 레이블) 튜플)을 반환하는 `__getitem__` 메서드를 반드시 구현해야 한다.18
- **`DataLoader`:** `DataLoader`는 `Dataset` 객체를 감싸는 반복자(iterator)이다. `DataLoader`는 학습 과정에 필요한 복잡한 데이터 처리 작업들을 자동화해준다. 주요 기능으로는 (1) 데이터를 지정된 크기의 미니배치(mini-batch)로 묶어주는 기능, (2) 매 에포크(epoch)마다 데이터를 무작위로 섞어(`shuffle=True`) 모델의 과적합을 방지하는 기능, (3) 여러 개의 CPU 코어를 사용하여 데이터 로딩을 병렬로 처리함으로써 학습 속도를 높이는 기능 등이 있다.19


손실 함수(Loss Function)는 모델의 예측 값이 실제 정답과 얼마나 다른지를 정량적으로 측정하는 함수이다. 신경망 학습의 목표는 이 손실 함수의 값을 최소화하는 파라미터를 찾는 것이다.22 즉, 손실 값은 모델의 예측이 얼마나 '틀렸는지'에 대한 벌점(penalty)으로 작용한다.23 문제의 종류에 따라 적절한 손실 함수를 선택하는 것이 중요하다.

- **회귀 손실 (Mean Squared Error):** `nn.MSELoss`는 연속적인 값을 예측하는 회귀(regression) 문제에서 널리 사용된다. 예측 값과 실제 값의 차이를 제곱한 후 평균을 내어 계산한다. 오차를 제곱하기 때문에, 큰 오차에 대해 더 큰 페널티를 부여하는 특징이 있다.22
- **분류 손실 (Cross-Entropy Loss):** `nn.CrossEntropyLoss`는 여러 개의 클래스 중 하나를 예측하는 다중 클래스 분류(multi-class classification) 문제의 표준적인 손실 함수이다. 이 함수는 내부적으로 `LogSoftmax` 활성화 함수와 `NLLLoss`(Negative Log Likelihood Loss)를 결합한 형태로, 모델의 원시 출력(logits)을 직접 입력으로 받아 확률 분포 간의 차이를 측정한다.24

| 손실 함수             | 주요 사용 사례    | LaTeX 수식                                                   |
| --------------------- | ----------------- | ------------------------------------------------------------ |
| `nn.MSELoss`          | 회귀 (Regression) | $ \text{MSE}(y, \hat{y}) = \frac{1}{N} \sum_{i=0}^{N - 1} (y_i - \hat{y}_i)^2 $` |
| `nn.CrossEntropyLoss` | 다중 클래스 분류  | $ \text{CE} = -\sum_{c=1}^{M} y_{o,c} \log(p_{o,c}) $`       |
| `nn.BCELoss`          | 이진 분류         | $ \text{BCE} = -(y \log(p) + (1-y) \log(1-p)) $`             |


옵티마이저(Optimizer)는 `autograd`를 통해 계산된 기울기 정보를 바탕으로, 손실 함수를 최소화하기 위해 모델의 파라미터를 업데이트하는 알고리즘이다.27

`optimizer.step()` 메서드를 호출하면 실제 파라미터 업데이트가 수행된다.

- **SGD (Stochastic Gradient Descent):** `torch.optim.SGD`는 가장 기본적이고 전통적인 옵티마이저이다. 미니배치에 대한 기울기를 계산하고, 이 기울기의 반대 방향으로 학습률(learning rate)만큼 파라미터를 이동시킨다.29
- **Adam (Adaptive Moment Estimation):** `torch.optim.Adam`은 현재 가장 널리 사용되는 옵티마이저 중 하나이다. 각 파라미터마다 독립적인 학습률을 적응적으로 조절하는 것이 특징이다. 과거 기울기의 지수 이동 평균(1차 모멘텀)과 과거 기울기 제곱 값의 지수 이동 평균(2차 모멘텀)을 함께 사용하여, 빠르고 안정적으로 최적점에 수렴하는 경향이 있다.29 SGD가 모든 파라미터에 동일한 학습률을 적용하는 반면, Adam은 각 파라미터의 업데이트 기록을 바탕으로 업데이트 강도를 동적으로 조절하기 때문에 하이퍼파라미터 튜닝에 덜 민감하고 일반적으로 좋은 성능을 보인다.

| 옵티마이저   | 핵심 아이디어                              | 파라미터 업데이트 LaTeX 수식                                 |
| ------------ | ------------------------------------------ | ------------------------------------------------------------ |
| `optim.SGD`  | 미니배치 기울기 방향으로 파라미터 업데이트 | $ \theta = \theta - \eta \cdot \nabla_\theta J( \theta; x^{(i)}; y^{(i)}) $` |
| `optim.Adam` | 모멘텀과 적응형 학습률 결합                | $ w_{t+1} = w_t - \frac{\alpha \cdot \hat{m}_t}{\sqrt{\hat{v}_t} + \epsilon} $` |



모델 학습은 에포크(epoch) 단위로 반복되는 학습 루프(training loop)를 통해 진행된다. 하나의 에포크는 전체 학습 데이터셋을 한 번 모두 사용하는 것을 의미한다. 각 에포크 내에서 데이터는 미니배치 단위로 처리되며, 각 미니배치마다 파라미터 업데이트가 한 번씩 일어난다. 하나의 학습 스텝은 다음과 같은 5단계의 명확한 순서로 구성된다.12

1. **기울기 초기화 (`optimizer.zero_grad()`):** PyTorch는 기본적으로 기울기를 누적(accumulate)하도록 설계되어 있다. 이는 RNN과 같은 특정 모델에서 유용할 수 있지만, 일반적인 경우에는 각 미니배치의 기울기가 이전 배치의 기울기에 영향을 주지 않도록 해야 한다. 따라서 매 학습 스텝이 시작될 때마다 `optimizer.zero_grad()`를 호출하여 모든 파라미터의 `.grad` 속성을 0으로 초기화해야 한다.
2. **순방향 전파 (`prediction = model(inputs)`):** 현재 미니배치의 입력 데이터를 모델에 통과시켜 예측 값을 계산한다. 이 과정에서 `autograd`는 연산 과정을 추적하여 계산 그래프를 생성한다.
3. **손실 계산 (`loss = loss_fn(prediction, targets)`):** 모델의 예측 값과 실제 정답 레이블을 손실 함수에 전달하여 손실 값을 계산한다. 이 손실 값은 모델이 얼마나 잘못 예측했는지를 나타내는 스칼라 텐서이다.
4. **역방향 전파 (`loss.backward()`):** 계산된 손실 값에 대해 `.backward()` 메서드를 호출한다. `autograd`는 생성된 계산 그래프를 따라 역으로 이동하며, 손실에 대한 모든 학습 가능 파라미터의 기울기를 계산하고 각 파라미터의 `.grad` 속성에 저장한다.
5. **파라미터 업데이트 (`optimizer.step()`):** 옵티마이저의 `step()` 메서드를 호출한다. 옵티마이저는 각 파라미터의 `.grad` 속성에 저장된 기울기 값을 참조하여, 자신이 구현하는 최적화 알고리즘(예: SGD, Adam)에 따라 파라미터 값을 업데이트한다.

이 5단계는 PyTorch의 핵심 컴포넌트들이 유기적으로 상호작용하는 과정을 보여준다. 데이터(`Tensor`)가 모델(`nn.Module`)을 통해 흐르며 순방향 전파가 일어나고, 이 과정을 `autograd`가 지켜보며 계산 그래프를 그린다. 손실 함수가 예측과 정답을 비교해 오차를 계산하면, `loss.backward()`를 통해 `autograd`가 기울기를 계산하여 각 파라미터의 `.grad` 속성을 채운다. 마지막으로 `optimizer`가 이 기울기 정보를 바탕으로 파라미터를 업데이트하며 하나의 학습 사이클이 완성된다. 이 긴밀한 인과 관계의 사슬을 이해하는 것이 PyTorch를 효과적으로 사용하는 열쇠이다.


모델이 학습 데이터에만 과적합되지 않고, 보지 못한 새로운 데이터에 대해서도 좋은 성능을 내는지(일반화 성능) 확인하기 위해 평가 루프를 사용한다. 평가 루프는 학습에 사용되지 않은 검증(validation) 데이터셋이나 테스트(test) 데이터셋으로 수행된다.

평가 루프는 학습 루프와 유사하지만 두 가지 중요한 차이점이 있다.

- **평가 모드 전환 (`model.eval()`):** 평가를 시작하기 전에 `model.eval()`을 호출하여 모델을 평가 모드로 전환해야 한다. 드롭아웃(dropout)이나 배치 정규화(batch normalization)와 같은 일부 계층들은 학습 시와 평가 시에 다르게 동작한다. `model.eval()`은 이러한 계층들이 평가에 적합한 방식으로 동작하도록 설정해준다.14
- **기울기 계산 비활성화 (`with torch.no_grad()`):** 평가 시에는 모델 파라미터를 업데이트하지 않으므로 기울기를 계산할 필요가 없다. `with torch.no_grad()` 컨텍스트 블록을 사용하여 기울기 계산을 비활성화하면, 불필요한 메모리 사용을 줄이고 계산 속도를 높일 수 있다.14



모델 학습에는 많은 시간과 컴퓨팅 자원이 소요되므로, 학습된 모델을 저장하고 나중에 재사용하는 것은 매우 중요하다. PyTorch에서 모델을 저장하는 가장 권장되는 방법은 모델의 `state_dict`를 사용하는 것이다. `state_dict`는 모델의 학습 가능한 모든 파라미터(가중치와 편향)를 각 계층의 이름과 텐서로 매핑하는 파이썬 딕셔너리(dictionary) 객체이다.33 이는 모델의 아키텍처(코드)와는 별개로, 학습을 통해 얻어진 모델의 '상태'만을 나타낸다.


`state_dict`를 이용한 모델 저장 및 불러오기는 간단한 함수 호출로 이루어진다.

- **저장:** `torch.save(model.state_dict(), PATH)`를 사용하여 모델의 `state_dict`를 디스크에 저장한다. `PATH`는 저장될 파일 경로이며, 일반적으로 `.pt` 또는 `.pth` 확장자를 사용한다.33
- **불러오기:** 먼저, 저장했을 때와 동일한 아키텍처를 가진 모델 클래스의 인스턴스를 생성해야 한다. 그 다음, `model.load_state_dict(torch.load(PATH))`를 호출하여 저장된 파라미터를 새로 생성한 모델 인스턴스에 로드한다.33 불러온 모델을 추론(inference)에 사용하기 전에는 반드시 `model.eval()`을 호출하여 평가 모드로 전환해야 일관된 예측 결과를 얻을 수 있다.33


PyTorch에서는 `torch.save(model, PATH)`와 같이 모델 객체 전체를 저장할 수도 있지만, `state_dict` 방식이 훨씬 선호된다. 그 이유는 유연성과 견고성 때문이다.

모델 객체 전체를 저장하는 것은 내부적으로 Python의 `pickle`을 사용하는데, 이 방식은 저장될 당시의 특정 클래스 정의와 디렉토리 구조에 강하게 종속된다.33 만약 나중에 프로젝트의 파일 구조를 변경하거나, 모델 클래스의 이름을 바꾸거나, 코드를 리팩토링하면 저장된 파일을 불러오는 데 실패할 수 있다.

반면, `state_dict`는 순수한 파라미터 데이터(딕셔너리)이기 때문에 코드의 구조적 변경에 영향을 받지 않는다. 이는 모델의 아키텍처(코드)와 학습된 가중치(데이터)를 명확하게 분리하는 좋은 소프트웨어 공학적 실천이다.33

`state_dict`는 모델의 영속성(persistence)을 코드의 생명주기(lifecycle)로부터 분리하는 핵심적인 추상화 계층이다. 소프트웨어로서의 모델 코드는 지속적으로 개선되고 리팩토링되지만, 막대한 비용을 들여 학습한 가중치는 중요한 자산으로 보존되어야 한다. `state_dict`는 이 둘을 분리함으로써, 모델 코드의 변경과 무관하게 학습된 가중치를 재사용하고 이식할 수 있게 한다. 예를 들어, 연구자는 새로운 아키텍처를 실험하면서 기존 모델의 가중치 일부를 전이 학습(transfer learning)에 활용할 수 있으며, 엔지니어는 안정화된 버전의 모델 코드와 특정 버전의 가중치를 결합하여 프로덕션 환경에 안정적으로 배포할 수 있다. 이는 협업, 배포, 장기적인 프로젝트 유지보수를 가능하게 하는 MLOps의 근간이 되는 개념이다.


`state_dict` 방식은 서로 다른 하드웨어 환경 간에 모델을 주고받을 때도 유연성을 제공한다. 예를 들어, GPU에서 학습하고 저장한 모델을 GPU가 없는 환경(CPU)에서 불러와야 할 경우가 있다. 이때 `torch.load()` 함수의 `map_location` 인자를 사용하면 된다. `torch.load(PATH, map_location=torch.device('cpu'))`와 같이 지정하면, GPU에 저장되었던 텐서들이 CPU 메모리로 매핑되어 로드된다.33 이와 유사하게 CPU에서 저장한 모델을 특정 GPU로 로드하는 등 다양한 시나리오에 유연하게 대처할 수 있다.33


본문에서 다룬 핵심적인 손실 함수와 옵티마이저의 수학적 정의는 다음과 같다.

| 항목           | 구분                              | LaTeX 수식                                                   |
| -------------- | --------------------------------- | ------------------------------------------------------------ |
| **손실 함수**  | Mean Squared Error (MSE)          | $ \text{MSE}(y, \hat{y}) = \frac{1}{N} \sum_{i=0}^{N - 1} (y_i - \hat{y}_i)^2 $` |
|                | Categorical Cross-Entropy         | $ \text{CE} = -\sum_{c=1}^{M} y_{o,c} \log(p_{o,c}) $`       |
| **옵티마이저** | Stochastic Gradient Descent (SGD) | $ \theta = \theta - \eta \cdot \nabla_\theta J( \theta; x^{(i)}; y^{(i)}) $` |
|                | Adam                              | $ w_{t+1} = w_t - \frac{\alpha \cdot \hat{m}_t}{\sqrt{\hat{v}_t} + \epsilon} $` |


1. Tensors and Dynamic neural networks in Python with strong GPU acceleration - GitHub, 8월 24, 2025에 액세스, https://github.com/pytorch/pytorch
2. Learning PyTorch with Examples — PyTorch Tutorials 2.8.0+cu128 documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/tutorials/beginner/pytorch_with_examples.html
3. PyTorch 101: Learn Deep Learning with PyTorch | DigitalOcean, 8월 24, 2025에 액세스, https://www.digitalocean.com/community/tutorials/pytorch-101-advanced
4. Learn the Basics — PyTorch Tutorials 2.8.0+cu128 documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/tutorials/beginner/basics/intro.html
5. Tensors — PyTorch Tutorials 2.8.0+cu128 documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/tutorials/beginner/basics/tensorqs_tutorial.html
6. Tensors — PyTorch Tutorials 2.8.0+cu128 documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/tutorials/beginner/blitz/tensor_tutorial.html
7. Introduction to PyTorch Tensors. Section 1 - Medium, 8월 24, 2025에 액세스, https://medium.com/@heyamit10/introduction-to-pytorch-tensors-16caaa0cd9b1
8. Creating a Tensor in Pytorch - Tutorialspoint, 8월 24, 2025에 액세스, https://www.tutorialspoint.com/creating-a-tensor-in-pytorch
9. Introduction to PyTorch Tensors, 8월 24, 2025에 액세스, https://docs.pytorch.org/tutorials/beginner/introyt/tensors_deeper_tutorial.html
10. PyTorch Tutorial - John Lambert, 8월 24, 2025에 액세스, https://johnwlambert.github.io/pytorch-tutorial/
11. Tensor Math Operations - Deep Learning with PyTorch 4 - YouTube, 8월 24, 2025에 액세스, https://www.youtube.com/watch?v=Ta3z9vZaoMc
12. A Gentle Introduction to torch.autograd — PyTorch Tutorials 2.8.0+ ..., 8월 24, 2025에 액세스, https://docs.pytorch.org/tutorials/beginner/blitz/autograd_tutorial.html
13. Understanding Autograd in PyTorch: The Core of Automatic Differentiation - Medium, 8월 24, 2025에 액세스, https://medium.com/@sahin.samia/understanding-autograd-in-pytorch-the-core-of-automatic-differentiation-ca9d98da980d
14. Autograd mechanics — PyTorch 2.8 documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/docs/stable/notes/autograd.html
15. The Fundamentals of Autograd — PyTorch Tutorials 2.8.0+cu128 documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/tutorials/beginner/introyt/autogradyt_tutorial.html
16. ducminhkhoi/autograd_pytorch: Building your own autograd mechanism based on PyTorch tensor only (not Variable, can be seen as numpy array) - GitHub, 8월 24, 2025에 액세스, https://github.com/ducminhkhoi/autograd_pytorch
17. Build the Neural Network — PyTorch Tutorials 2.8.0+cu128 ..., 8월 24, 2025에 액세스, https://docs.pytorch.org/tutorials/beginner/basics/buildmodel_tutorial.html
18. What is torch.nn really? — PyTorch Tutorials 2.8.0+cu128 documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/tutorials/beginner/nn_tutorial.html
19. Training with PyTorch — PyTorch Tutorials 2.8.0+cu128 documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/tutorials/beginner/introyt/trainingyt.html
20. Datasets & DataLoaders — PyTorch Tutorials 2.8.0+cu128 documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/tutorials/beginner/basics/data_tutorial.html
21. Quickstart — PyTorch Tutorials 2.8.0+cu128 documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/tutorials/beginner/basics/quickstart_tutorial.html
22. PyTorch Loss Functions: The Ultimate Guide - neptune.ai, 8월 24, 2025에 액세스, https://neptune.ai/blog/pytorch-loss-functions
23. Beginner's Guide to Pytorch Loss Functions | Zero To Mastery, 8월 24, 2025에 액세스, https://zerotomastery.io/blog/pytorch-loss-functions/
24. Loss Functions - PyTorch nn - Codecademy, 8월 24, 2025에 액세스, https://www.codecademy.com/resources/docs/pytorch/nn/loss-functions
25. Guide to PyTorch Loss Functions - Medium, 8월 24, 2025에 액세스, https://medium.com/biased-algorithms/guide-to-pytorch-loss-functions-90ab7ca85ec2
26. Understanding Categorical Cross-Entropy Loss, Binary Cross ..., 8월 24, 2025에 액세스, https://gombru.github.io/2018/05/23/cross_entropy_loss/
27. Custom Optimizers in Pytorch - GeeksforGeeks, 8월 24, 2025에 액세스, https://www.geeksforgeeks.org/machine-learning/custom-optimizers-in-pytorch/
28. Using Optimizers from PyTorch - MachineLearningMastery.com, 8월 24, 2025에 액세스, https://machinelearningmastery.com/using-optimizers-from-pytorch/
29. bentrevett/a-tour-of-pytorch-optimizers - GitHub, 8월 24, 2025에 액세스, https://github.com/bentrevett/a-tour-of-pytorch-optimizers
30. Gradient descent — Statistics and Machine Learning in Python 0.5 ..., 8월 24, 2025에 액세스, https://duchesnay.github.io/pystatsml/optimization/optim_gradient_descent.html
31. Optimizers in Pytorch | Quick Walkthrough | Tutorial for Beginners - YouTube, 8월 24, 2025에 액세스, https://www.youtube.com/watch?v=lB_8XL7fpSM
32. What is Adam Optimizer? - GeeksforGeeks, 8월 24, 2025에 액세스, https://www.geeksforgeeks.org/deep-learning/adam-optimizer/
33. Saving and Loading Models — PyTorch Tutorials 2.8.0+cu128 ..., 8월 24, 2025에 액세스, https://docs.pytorch.org/tutorials/beginner/saving_loading_models.html
34. Saving and Loading Models — PyTorch Tutorials 2.7.0+cu126 documentation, 8월 24, 2025에 액세스, [https://docs.pytorch.org/tutorials/beginner/saving_loading_models.html?highlight=torch%20save](https://docs.pytorch.org/tutorials/beginner/saving_loading_models.html?highlight=torch+save)