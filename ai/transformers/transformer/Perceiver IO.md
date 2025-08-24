# Perceiver IO
[트랜스포머 (Transformer)](./index.md)



트랜스포머(Transformer) 아키텍처는 인코더-디코더 구조를 기반으로 하며, 멀티-헤드 어텐션(Multi-Head Attention)과 위치별 피드포워드 네트워크(Position-wise Feed-Forward Network)를 핵심 구성 요소로 한다.1 이 아키텍처의 가장 혁신적인 부분은 셀프 어텐션(Self-Attention) 메커니즘으로, 이는 시퀀스 내의 모든 토큰 쌍 간의 관계를 직접적으로 모델링하여 문맥에 맞는 풍부한 표현을 학습한다.3 셀프 어텐션은 순환 신경망(RNN)의 순차적 처리 방식이 가지는 장기 의존성 문제와 병렬 처리의 어려움을 극복하며 자연어 처리 분야에 혁명을 일으켰다.1


그러나 트랜스포머의 강력한 표현력은 막대한 계산 비용을 수반한다. 셀프 어텐션 메커니즘의 시간 및 메모리 복잡도는 입력 시퀀스의 길이 $n$에 대해 제곱에 비례하여 증가하는, 이른바 '$O(n^2)$ 문제'를 내재하고 있다. 구체적으로, $n \times d$ 차원의 Query(Q) 행렬과 $d \times n$ 차원의 Key(K) 전치 행렬을 곱하여 $n \times n$ 크기의 어텐션 스코어 행렬을 생성하는 과정에서 계산 복잡도는 $O(n^2d)$가 된다.3

이러한 2차 복잡도는 모델의 확장성에 심각한 제약을 가한다. BERT와 같은 언어 모델이 최대 512 토큰 정도의 비교적 짧은 시퀀스에만 적용될 수 있는 근본적인 이유가 바로 여기에 있다.7 이 문제는 텍스트를 넘어 다른 데이터 양식으로 확장할 때 더욱 심각해진다. 예를 들어, 수만 개의 픽셀로 구성된 이미지, 수십만 개의 샘플로 이루어진 음성 신호, 혹은 비디오 데이터에 표준 트랜스포머를 직접 적용하는 것은 '2차 병목 현상(Quadratic Bottleneck)'으로 인해 현실적으로 불가능하다.8


이 병목 현상을 우회하기 위해 다양한 도메인 특화적(domain-specific) 기법들이 제안되었다. 대표적인 예로 Vision Transformer(ViT)는 이미지를 작은 패치(patch)들로 분할하여 시퀀스 길이를 인위적으로 줄이고 11, Wav2Vec2는 원시 음성 파형에서 특징 벡터 시퀀스를 추출하는 전처리 과정을 거친다.11

이러한 접근법들은 특정 데이터 양식에 대한 강력한 귀납적 편향(inductive bias)을 아키텍처에 주입하여 해당 도메인에서 높은 성능을 달성했다. 하지만 이는 모델의 범용성을 희생하는 대가로 얻어진 것이다. 각 데이터 양식에 맞춰 별도의 전처리 모듈과 아키텍처 수정을 요구하기 때문에, 모델은 특정 데이터 유형에 강하게 종속된다.13 따라서 트랜스포머의 문제를 단순히 계산적 비효율성으로만 볼 것이 아니라, 이 비효율성이 진정한 의미의 '범용 지각 모델(general perception model)'로의 발전을 가로막는 근본적인 장벽이라는 관점에서 접근할 필요가 있다. 이러한 배경 속에서, 도메인에 대한 가정을 최소화하면서도 다양한 종류의 대규모 입력을 효율적으로 처리할 수 있는 근본적인 아키텍처 혁신으로서 Perceiver가 등장하게 되었다.8



Perceiver 아키텍처는 트랜스포머의 2차 복잡도 문제를 해결하기 위해, 크기가 작고 고정된 학습 가능한 '잠재 배열(latent array)'을 도입하는 혁신적인 아이디어를 제시한다.13 이 잠재 배열은 거대한 입력 데이터와 심층적인 처리 네트워크 사이의 정보 병목(information bottleneck) 역할을 수행한다.

핵심 메커니즘은 '비대칭 어텐션(asymmetric attention)'이다. 표준 셀프 어텐션에서는 Query, Key, Value가 모두 동일한 소스(입력 시퀀스)에서 파생되지만, Perceiver에서는 이 관계가 비대칭적으로 재구성된다. 즉, 크기가 작은 잠재 배열이 Query를 생성하고, 크기가 매우 큰 입력 배열이 Key와 Value를 생성한다. 이는 본질적으로 교차 어텐션(cross-attention)의 한 형태로, Query와 Key/Value의 소스를 분리하여 정보 흐름을 제어하는 것이 핵심이다.12


이러한 비대칭 구조는 계산 복잡도에 극적인 변화를 가져온다. 입력 배열의 크기를 $M$, 잠재 배열의 크기를 $N$이라고 할 때($N \ll M$), 교차 어텐션의 계산 복잡도는 표준 셀프 어텐션의 $O(M^2)$에서 $O(MN)$으로 감소한다. 잠재 배열의 크기 $N$은 입력 크기와 무관한 하이퍼파라미터이므로, 이는 전체 계산량이 입력 크기 $M$에 대해 선형적으로 증가함을 의미한다.14 이 구조적 혁신 덕분에 Perceiver는 수만 개 이상의 픽셀이나 음성 샘플과 같은 고차원 입력을 직접 처리할 수 있는 능력을 갖추게 되었다.13


Perceiver의 설계는 단순히 계산량을 줄이는 것을 넘어, 더 심오한 구조적 이점을 제공한다. 바로 '모델의 깊이'와 '입력의 크기'를 분리(decoupling)한 것이다. 교차 어텐션을 통해 정보는 $N$ 크기의 잠재 표현으로 압축된다. 이후의 모든 심층적인 정보 처리는 이 작은 잠재 공간 내에서 셀프 어텐션 레이어들로 구성된 '잠재 트랜스포머 타워(latent transformer tower)'를 통해 이루어진다. 이 잠재 공간 내에서의 연산 복잡도는 $O(N^2)$으로, 입력 크기 $M$과는 완전히 무관하다.14

결과적으로 Perceiver 전체 아키텍처의 복잡도는 $O(MN + L \cdot N^2)$가 된다 (여기서 $L$은 잠재 트랜스포머의 깊이). 이는 입력 크기 $M$이 아무리 커지더라도 모델의 깊이 $L$을 늘리는 데 드는 추가 비용이 없음을 의미한다. 이 '이중 분리' 원칙 덕분에 Perceiver는 대규모 입력에 대해 전례 없이 깊은 모델을 구축할 수 있으며, 이는 ImageNet 실험에서 48개의 잠재 블록을 사용한 사례에서 입증되었다.13


단 한 번의 교차 어텐션만으로는 정보 병목이 너무 심해져 입력의 세부 정보를 충분히 포착하지 못할 수 있다. 이를 완화하기 위해 Perceiver는 교차 어텐션 블록과 잠재 셀프 어텐션 블록을 여러 번 반복하는 구조를 채택할 수 있다. 이를 통해 잠재 배열은 입력 데이터로부터 점진적으로 필요한 정보를 반복적으로 추출하게 된다.13 또한, 각 반복 블록 간에 가중치를 공유(weight sharing)함으로써 파라미터 효율성을 높일 수 있으며, 이는 모델이 마치 순환 신경망(RNN)처럼 작동하게 만든다.8


Perceiver는 다양한 종류의 대규모 입력을 처리하는 데 획기적인 성공을 거두었지만, 출력 생성 방식에는 명백한 한계가 있었다. 최종 잠재 배열을 전역 평균 풀링(global average pooling)한 후, 단일 선형 레이어를 통과시켜 클래스 로짓(logit)을 생성하는 방식에 머물렀기 때문이다.10 이로 인해 Perceiver의 적용 범위는 주로 분류(classification)와 같은 단순한 출력 공간을 가진 태스크에 국한되었다.7 언어 생성, 옵티컬 플로우 예측, 의미 분할 등과 같이 구조화된 고차원 출력을 요구하는 복잡한 실제 문제에 적용하기에는 역부족이었다. 이러한 한계는 Perceiver를 진정한 범용 아키텍처로 발전시키기 위한 다음 단계, 즉 Perceiver IO의 개발을 촉진하는 직접적인 동기가 되었다.15


Perceiver IO는 기존 Perceiver의 아이디어를 입력뿐만 아니라 출력까지 확장하여, 임의의 입력을 받아 임의의 구조화된 출력을 생성할 수 있는 완전한 '읽기(Read)-처리(Process)-쓰기(Write)' 프레임워크를 제안한다.21


Perceiver IO의 정보 흐름은 세 가지 명확한 단계로 구성된다.

- **읽기 (인코딩):** 다양한 양식의 거대한 입력 배열을 저차원의 고정된 잠재 공간으로 매핑한다.
- **처리:** 잠재 공간 내에서 깊은 셀프 어텐션 연산을 통해 정보를 정제하고 고차원적인 관계를 학습한다.
- **쓰기 (디코딩):** 정제된 잠재 표현으로부터 특정 태스크에 맞는 임의의 크기와 구조를 가진 출력 배열을 생성한다.


인코딩 단계는 기존 Perceiver의 핵심 메커니즘을 그대로 따른다. 다양한 모달리티(텍스트, 이미지, 음성 등)로부터 온 입력 데이터는 먼저 전처리기(Preprocessor)를 통해 임베딩된 바이트 배열($M \times C_{in}$)로 변환된다.11 데이터의 순서나 위치 정보가 중요한 경우, 푸리에 특징(Fourier features)과 같은 위치 인코딩이 입력 특징에 결합된다.11 그 후, 학습 가능한 잠재 배열($N \times D_{latent}$)이 Query를 생성하고 전처리된 입력 배열이 Key와 Value를 생성하여 교차 어텐션을 수행한다. 이 과정을 통해 방대한 입력 정보가 작은 잠재 배열로 효율적으로 '증류(distill)'된다.11


인코더를 통과한 잠재 배열은 깊은 트랜스포머 타워, 즉 프로세서의 입력이 된다. 이 프로세서는 다수의 셀프 어텐션 블록으로 구성되며, 모든 연산은 오직 잠재 배열 내에서만 이루어진다. 따라서 계산 복잡도는 입력 크기 $M$이나 출력 크기 $O$와는 완전히 무관한 $O(N^2)$을 유지한다.11 이 단계에서 모델은 입력으로부터 추출된 핵심 정보들 간의 복잡하고 추상적인 관계를 학습하며, 문제 해결에 필요한 고차원적인 표현을 형성한다.


Perceiver IO의 가장 핵심적인 혁신은 바로 디코딩 메커니즘에 있다.22 이 단계는 생성하고자 하는 출력의 구조와 의미를 명시적으로 정의하는 '출력 쿼리 배열(Output Query Array)' ($O \times C_{out}$)을 도입함으로써 기존 Perceiver의 한계를 극복한다.24

디코딩 과정은 인코딩과 대칭적인 교차 어텐션으로 이루어진다. 이번에는 출력 쿼리 배열이 Query를 생성하고, 프로세서를 거쳐 정제된 최종 잠재 배열이 Key와 Value를 생성한다.11 어텐션 연산의 출력은 항상 Query의 형태를 따르기 때문에, 이 디코더 교차 어텐션의 결과물은 $O \times D_{latent}$ 차원을 가지게 된다. 즉, 원하는 출력의 크기($O$)와 동일한 수의 요소를 가진 배열이 생성되는 것이다. 이 배열은 마지막으로 후처리기(Postprocessor)를 통해 각 태스크에 맞는 최종 출력 형태로 변환된다.11


출력 쿼리 배열의 설계는 Perceiver IO의 범용성을 뒷받침하는 핵심 요소이다. 이 쿼리는 태스크의 요구사항에 따라 매우 유연하게 구성될 수 있다.

- **분류:** 단 하나의 학습 가능한 임베딩 벡터를 쿼리로 사용하여 전체 잠재 공간의 정보를 요약하고 클래스별 로짓을 생성할 수 있다.11
- **언어 모델링:** 출력하고자 하는 텍스트 시퀀스의 각 위치에 해당하는 위치 인코딩을 쿼리로 사용하여, 잠재 공간에 "이 위치에 와야 할 토큰은 무엇인가?"를 질의한다.7
- **옵티컬 플로우:** 이미지의 모든 픽셀 좌표 $(x, y)$를 쿼리로 사용하여, 각 픽셀 위치에 대한 이동 벡터 $(dx, dy)$를 예측하도록 잠재 공간에 질의한다.26
- **멀티모달 자동인코딩:** 비디오 프레임의 픽셀 위치, 오디오 샘플의 시간 위치, 클래스 레이블을 위한 전역 쿼리 등 각기 다른 모달리티와 구조에 해당하는 쿼리들을 결합하여 단일 모델로부터 다양한 출력을 동시에 생성할 수 있다.11

이러한 출력 쿼리의 유연성은 잠재 공간을 일종의 고차원 정보 데이터베이스로, 출력 쿼리를 해당 데이터베이스에 대한 구조화된 '질의 인터페이스'로 기능하게 한다. 쿼리는 단순히 '무엇'(예: 옵티컬 플로우)을 원하는지를 명시할 뿐만 아니라 '어디에'(예: 픽셀 (x,y) 위치) 대한 정보인지를 지정한다. 이 방식은 특정 좌표를 쿼리하여 해당 위치의 값을 얻는 NeRF의 아이디어와도 맞닿아 있으며 25, Perceiver IO가 단일 아키텍처로 전혀 다른 종류의 구조화된 출력을 생성할 수 있는 원동력이 된다.



Perceiver IO의 모든 어텐션 연산은 스케일드 닷-프로덕트 어텐션(Scaled Dot-Product Attention)을 기반으로 한다. Query(Q), Key(K), Value(V) 세 개의 행렬이 주어졌을 때, 어텐션의 출력은 다음 수식으로 계산된다.4
$$
\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V
$$
여기서 $d_k$는 Key 벡터의 차원을 의미하며, $\sqrt{d_k}$로 나누어주는 스케일링 과정은 내적 값이 너무 커져 소프트맥스 함수의 그래디언트가 소실되는 것을 방지하는 역할을 한다.


Perceiver IO의 세 가지 핵심 어텐션 단계는 이 기본 수식을 서로 다른 입력 소스에 적용함으로써 구현된다. 입력 배열을 $X \in \mathbb{R}^{M \times C}$, 잠재 배열을 $Z \in \mathbb{R}^{N \times D}$, 출력 쿼리 배열을 $Y_q \in \mathbb{R}^{O \times E}$라고 정의하고 각 단계의 연산을 수식으로 나타내면 다음과 같다.

- **인코더 교차 어텐션 (정보 압축):**
  - $Q = Z W_q$, $K = X W_k$, $V = X W_v$
  - $Z_{\text{new}} = \text{Attention}(Q, K, V)$
  - 잠재 배열 $Z$가 Query를, 거대한 입력 배열 $X$가 Key와 Value를 생성하는 비대칭 구조이다. $M$ 차원의 정보가 $N$ 차원으로 압축된다.
- **프로세서 셀프 어텐션 (정보 정제):**
  - $Q = Z_{\text{proc}} W_q$, $K = Z_{\text{proc}} W_k$, $V = Z_{\text{proc}} W_v$
  - $Z_{\text{proc\_new}} = \text{Attention}(Q, K, V)$
  - Query, Key, Value가 모두 잠재 배열 $Z_{\text{proc}}$에서 파생되는 대칭적 구조이다. $N$ 차원의 잠재 공간 내에서 정보가 정제되고 관계가 재구성된다.
- **디코더 교차 어텐션 (정보 확장 및 투영):**
  - $Q = Y_q W_q$, $K = Z_{\text{final}} W_k$, $V = Z_{\text{final}} W_v$
  - $Y_{\text{out}} = \text{Attention}(Q, K, V)$
  - 출력 쿼리 $Y_q$가 Query를, 최종 잠재 배열 $Z_{\text{final}}$이 Key와 Value를 생성하는 또 다른 비대칭 구조이다. $N$ 차원의 정보가 $O$ 차원의 특정 구조를 가진 출력으로 확장 또는 투영된다.

이러한 '압축-정제-확장'의 대칭적 흐름은 Perceiver IO 아키텍처의 우아함을 보여준다. 각 단계는 명확한 역할을 수행하며, 이들의 유기적인 상호작용을 통해 정보 처리의 효율성과 표현력을 동시에 달성한다.

**Table 1: 어텐션 메커니즘별 계산 복잡도 비교**

| 구성 요소 (Component)             | 표준 트랜스포머 (Standard Transformer)        | Perceiver IO                                                 | 설명 (Description)                                           |
| --------------------------------- | --------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **입력 인코딩 (Input Encoding)**  | $O(M^2 \cdot d)$                              | $O(M \cdot N \cdot d)$                                       | Perceiver IO는 교차 어텐션을 통해 입력 크기 $M$에 대해 선형적인 복잡도를 달성한다. |
| **주요 처리 (Main Processing)**   | $O(L \cdot M^2 \cdot d)$                      | $O(L \cdot N^2 \cdot d)$                                     | Perceiver IO는 입력 크기 $M$과 무관한 작은 잠재 공간($N$)에서 깊은 처리를 수행한다. |
| **출력 디코딩 (Output Decoding)** | $O(O \cdot M \cdot d)$ (Encoder-Decoder Attn) | $O(O \cdot N \cdot d)$                                       | Perceiver IO는 잠재 공간을 쿼리하므로 출력 크기 $O$와 잠재 공간 크기 $N$에만 의존한다. |
| **전체 (Overall)**                | $O(L \cdot M^2 \cdot d)$                      | $O(M \cdot N \cdot d + L \cdot N^2 \cdot d + O \cdot N \cdot d)$ | $N \ll M, N \ll O$ 이므로, Perceiver IO는 입출력 크기에 대해 선형적인 확장성을 가진다. |

*주: $M$: 입력 시퀀스 길이, $N$: 잠재 배열 크기, $O$: 출력 시퀀스 길이, $L$: 처리 레이어 깊이, $d$: 모델 차원*


Perceiver IO의 아키텍처적 우수성은 다양한 도메인의 까다로운 벤치마크에서 거둔 강력한 실험 결과로 입증된다. 이는 모델의 일반성이 이론적 개념에 그치지 않고, 실제 문제 해결 능력으로 이어진다는 것을 보여주는 강력한 증거이다.


자연어 이해 능력 평가를 위한 GLUE 벤치마크에서 Perceiver IO는 주목할 만한 성과를 보였다. 표준 BERT 베이스라인과 유사한 계산량(FLOPs) 조건에서, 널리 사용되는 SentencePiece 토크나이저를 사용했을 때 BERT와 대등하거나 소폭 우월한 성능(평균 81.16점)을 기록했다.22

그러나 가장 인상적인 결과는 토크나이저 없이 원시 UTF-8 바이트(byte)를 직접 입력으로 사용했을 때 나타났다. 이 조건에서 Perceiver IO는 평균 80.95점을 획득하여, 토크나이저를 사용한 BERT와 거의 대등한 성능을 보였다.20 반면, 동일하게 바이트 입력을 받은 BERT 모델은 성능이 71.45점으로 크게 하락했다.25 이는 Perceiver IO가 언어에 대한 사전 지식인 토큰화 과정 없이도 데이터 자체로부터 언어의 구조적, 통계적 특성을 학습할 수 있는 강력한 일반화 능력을 갖추었음을 시사한다.22


컴퓨터 비전 분야에서 매우 어려운 태스크로 알려진 옵티컬 플로우(optical flow) 예측에서 Perceiver IO는 Sintel 벤치마크의 'final' 패스에서 당시 최고 수준(State-of-the-Art, SOTA)의 성능을 달성했다 (평균 종단점 오차 EPE: 2.42).20

이 성과가 특히 의미 있는 이유는, 기존의 옵티컬 플로우 모델들이 이미지의 다중 해상도 특징을 효과적으로 포착하기 위해 명시적으로 설계된 멀티스케일(multi-scale) 구조나 상관관계 볼륨(correlation volume)과 같은 도메인 특화적 장치에 크게 의존하는 반면, Perceiver IO는 이러한 아키텍처적 가정 없이도 뛰어난 성능을 달성했다는 점이다.22 이는 모델이 어텐션 메커니즘만을 통해 이미지 내의 장거리 의존성과 미세한 움직임 패턴을 데이터로부터 직접 학습했음을 보여준다.


Perceiver IO의 진정한 강점은 여러 데이터 양식을 동시에 처리하는 멀티모달(multimodal) 환경에서 드러난다.

- **Kinetics (비디오+오디오+레이블 자동인코딩):** 비디오 프레임, 오디오 파형, 클래스 레이블을 단일 입력으로 받아 이를 다시 재구성하는 복잡한 멀티모달 자동인코딩 태스크를 성공적으로 수행했다.22 이는 각 모달리티를 위한 별도의 인코더 '트렁크(trunk)' 없이, 단일 통합 아키텍처로 이종(heterogeneous) 데이터를 효과적으로 처리할 수 있음을 입증한 사례이다.22
- **AudioSet (오디오+비디오 분류):** 원시 오디오와 비디오 프레임을 함께 입력받아 사운드 이벤트를 분류하는 태스크에서 기존 Perceiver 모델보다 일관되게 향상된 성능(mAP 43.3)을 보였다.22 이는 Perceiver IO의 유연한 디코딩 메커니즘이 분류와 같은 단순 출력 태스크에서도 성능 향상에 기여할 수 있음을 시사한다.


- **StarCraft II:** 딥마인드의 게임 AI인 AlphaStar 에이전트의 핵심 구성 요소인 트랜스포머 모듈을 Perceiver IO로 대체한 결과, 계산량을 3분의 1로 줄이면서도 원본과 동일한 승률(87%)을 달성했다. 이는 복잡한 강화학습 환경의 실시간 의사결정에서도 Perceiver IO의 효율성과 성능이 유효함을 보여준다.22
- **ImageNet:** 2D 컨볼루션이나 이미지의 격자 구조에 대한 어떠한 사전 정보도 주지 않고 픽셀 자체만을 입력으로 사용했음에도 불구하고 강력한 분류 성능을 보였다. 특히 대규모 데이터셋(JFT)으로 사전 학습했을 경우, 기존의 이미지 특화 모델들과 견주어 손색없는 성능을 달성했다.22

**Table 2: 주요 벤치마크 성능 요약**

| 벤치마크 (Benchmark) | 태스크 (Task)      | 메트릭 (Metric)   | 베이스라인 모델 (Baseline Model) | Perceiver IO       | 비고 (Remarks)                              |
| -------------------- | ------------------ | ----------------- | -------------------------------- | ------------------ | ------------------------------------------- |
| **GLUE**             | 자연어 이해        | Average Score (↑) | BERT (SentencePiece)             | 81.16              | 토크나이저 사용 시 BERT 성능 상회           |
| **GLUE**             | 자연어 이해        | Average Score (↑) | BERT (UTF-8 Bytes)               | 80.95              | 토크나이저 없이도 BERT(w/ Tokenizer)와 대등 |
| **Sintel**           | 옵티컬 플로우      | EPE (final) (↓)   | RAFT (AutoFlow)                  | 2.42               | SOTA 달성                                   |
| **AudioSet**         | 오디오-비디오 분류 | mAP (↑)           | Perceiver                        | 43.3               | 유연한 디코더가 분류 성능도 향상            |
| **ImageNet**         | 이미지 분류        | Top-1 Acc. (↑)    | ResNet-50                        | 86.4 (pre-trained) | 이미지 특화 모델과 경쟁력 있는 성능         |
| **StarCraft II**     | 게임 플레이        | Win Rate (↑)      | AlphaStar                        | 0.87               | 3배 적은 FLOPs로 동일 성능 달성             |


Perceiver IO는 범용성과 효율성 측면에서 중요한 진전을 이루었지만, 완벽한 해결책은 아니며 몇 가지 내재적인 한계와 고려사항을 가지고 있다.


Perceiver IO의 효율성은 잠재 배열이라는 고정된 크기의 정보 병목(information bottleneck)을 통해 달성된다. 이는 필연적으로 입력 데이터에 포함된 모든 정보가 손실 없이 잠재 공간으로 완벽하게 압축될 수 없다는 것을 의미한다.8 특히 매우 세밀하고 국소적인 정보가 결정적인 역할을 하는 태스크에서는 이 병목 현상이 모델의 표현력을 제한하여 성능 저하의 원인이 될 수 있다. 잠재 배열의 크기 $N$은 계산 효율성과 정보 표현력 사이의 중요한 트레이드오프(trade-off) 관계에 있으며, 일부 실험에서는 잠재 차원이 특정 지점을 넘어 너무 커지면 오히려 성능이 저하되고 학습이 불안정해지는 현상이 관찰되기도 했다.8


Perceiver IO는 아키텍처 수준에서 높은 범용성을 지향하지만, 특정 도메인에서 최고 수준의 성능을 달성하기 위해서는 여전히 상당한 하이퍼파라미터 튜닝과 도메인 특화적인 엔지니어링 노력이 요구된다.25 예를 들어, 입력 데이터에 대한 최적의 전처리 방식(예: 푸리에 특징 vs. 컨볼루션), 태스크에 맞는 출력 쿼리의 정교한 설계, 잠재 배열의 크기와 모델 깊이의 결정 등은 여전히 전문가의 지식과 실험에 크게 의존한다. 이는 '하나의 아키텍처로 모든 것을 해결한다'는 이상과 실제 적용 사이의 간극을 보여준다. 따라서 Perceiver IO의 진정한 가치는 어떤 문제든 즉시 해결하는 '만능 해결사(silver bullet)'라기보다는, 다양한 문제에 맞게 유연하게 수정하고 최적화할 수 있는 강력한 '아키텍처 템플릿(template)'을 제공했다는 점에서 찾아야 한다.


Perceiver IO의 잠재 배열은 입력 시퀀스 전체를 보고 정보를 요약하는 방식으로 작동한다. 이러한 전역적인 정보 접근 방식은 각 타임스텝의 출력이 오직 이전 타임스텝의 정보에만 의존해야 하는 자기회귀(auto-regressive) 생성 태스크(예: 언어 모델의 텍스트 생성)에 직접 적용하기 어렵다는 구조적 한계를 가진다.25 표준 트랜스포머가 어텐션 마스크를 통해 쉽게 구현하는 인과적 구조(causal structure)를, 입력 전체를 압축한 잠재 공간에 직접 부과하는 것은 간단한 문제가 아니다. 이 한계를 해결하기 위해서는 Perceiver AR과 같이 자기회귀 생성을 위해 특별히 설계된 후속 연구가 필요했다.32


2차 복잡도 문제를 해결하려는 시도는 Perceiver IO 외에도 다수가 존재한다. Linformer, Reformer, Longformer와 같은 모델들은 주로 어텐션 행렬 자체를 저차원 행렬로 근사(low-rank approximation)하거나, 지역적(local) 또는 희소(sparse)한 패턴을 도입하여 계산을 효율화한다.33 이 모델들은 주로 긴 텍스트 시퀀스 처리에 초점을 맞추며 '어텐션의 계산 방식'을 바꾸는 접근법을 취한다. 반면, Perceiver IO는 '어텐션의 대상'을 소수의 잠재 벡터로 제한하는 근본적으로 다른 철학을 따른다. 이로 인해 Perceiver IO는 특정 모달리티나 시퀀스 구조에 덜 의존적인, 보다 범용적인 확장성을 확보하게 된다.



Perceiver IO는 인공지능 아키텍처 연구에 있어 중요한 이정표를 제시했다. 잠재 공간을 활용한 비대칭 어텐션과 유연한 출력 쿼리 메커니즘이라는 두 가지 핵심 아이디어를 통해, 트랜스포머의 고질적인 2차 복잡도 문제를 해결하고 입력과 출력 모두에서 선형적인 확장성을 달성했다. 더 중요하게는, 컨볼루션이나 토큰화와 같은 도메인 특화적인 아키텍처 가정을 최소화함으로써, 단일 모델로 이미지, 텍스트, 오디오, 비디오 등 다양한 데이터 양식을 아우르는 전례 없는 수준의 범용성을 입증했다.


Perceiver IO는 '효율적인 전역성'을 달성했지만, 순수한 전역 어텐션에만 의존하는 방식은 초고해상도 이미지나 장시간 비디오와 같이 입력 크기가 극단적으로 커질 경우 여전히 효율성의 한계를 보였다.36 이러한 한계를 극복하기 위해 후속 연구인 계층적 Perceiver(Hierarchical Perceiver, HiP)가 제안되었다.

HiP는 Perceiver의 범용성을 유지하면서도 아키텍처에 '지역성(locality)'을 다시 도입한다.36 입력을 계층적인 그룹으로 분할하고, 각 그룹 내에서 지역적으로 정보를 처리한 후, 그 결과를 다시 병합하여 상위 계층으로 전달하는 방식을 사용한다.38 이를 통해 계산 효율성과 성능을 한 단계 더 끌어올렸으며, 별도의 도메인 특화 전처리 없이도 고해상도 원시 데이터를 직접 처리할 수 있는 길을 열었다.37


Perceiver IO에서 HiP로 이어지는 연구의 흐름은 AI 아키텍처 발전의 변증법적 과정을 보여준다. 컨볼루션 신경망(CNN)이 '강한 지역성'을 가정했다면(정립, Thesis), 표준 트랜스포머와 Perceiver IO는 계산적 한계 속에서 '효율적인 전역성'을 추구하며 패러다임을 전환했다(반정립, Antithesis). 그리고 HiP는 Perceiver의 전역적 처리 능력에 계층적 지역성을 다시 통합함으로써 두 패러다임의 장점을 아우르는 새로운 종합(Synthesis)을 이루어냈다.

이는 미래의 범용 인공지능 아키텍처가 순수한 지역성이나 전역성 중 하나를 선택하는 것이 아니라, 이 둘 사이의 정교한 균형을 통해 다양한 스케일의 정보를 유연하게 처리하는 방향으로 진화할 것임을 시사한다. Perceiver IO가 제시한 잠재 공간을 통한 정보 병목, 쿼리 기반의 능동적 정보 추출, 그리고 입출력 양식에 구애받지 않는 처리 방식은 이러한 미래 아키텍처를 설계하는 데 있어 지속적인 영감과 핵심적인 구성 요소를 제공할 것으로 전망된다.


1. Transformer (deep learning architecture) - Wikipedia, 8월 16, 2025에 액세스, https://en.wikipedia.org/wiki/Transformer_(deep_learning_architecture)
2. Transformer Clear Explanation: Attention Is All You Need! - 2017 | by Jinpeng Zhang, 8월 16, 2025에 액세스, https://dataturbo.medium.com/transformer-attention-is-all-you-need-fe6205c5be33
3. Attention Mechanism Complexity Analysis | by Mridul Rao | Medium, 8월 16, 2025에 액세스, https://medium.com/@mridulrao674385/attention-mechanism-complexity-analysis-7314063459b1
4. Understanding and Coding the Self-Attention Mechanism of Large Language Models From Scratch - Sebastian Raschka, 8월 16, 2025에 액세스, https://sebastianraschka.com/blog/2023/self-attention-from-scratch.html
5. On The Computational Complexity of Self-Attention - Proceedings of Machine Learning Research, 8월 16, 2025에 액세스, https://proceedings.mlr.press/v201/duman-keles23a/duman-keles23a.pdf
6. Complexity of transformer attention network : r/LanguageTechnology - Reddit, 8월 16, 2025에 액세스, https://www.reddit.com/r/LanguageTechnology/comments/9gulm9/complexity_of_transformer_attention_network/
7. Perceiver - Hugging Face, 8월 16, 2025에 액세스, https://huggingface.co/docs/transformers/model_doc/perceiver
8. [D] Paper Explained - Perceiver: General Perception with Iterative Attention (Full Video Analysis) : r/MachineLearning - Reddit, 8월 16, 2025에 액세스, https://www.reddit.com/r/MachineLearning/comments/matg19/d_paper_explained_perceiver_general_perception/
9. Perceiver: General Perception with Iterative Attention (Google DeepMind Research Paper Explained) - YouTube, 8월 16, 2025에 액세스, https://www.youtube.com/watch?v=P_xeshTnPZg&pp=0gcJCfwAo7VqN5tD
10. The Annotated Perceiver. A detailed PyTorch tutorial for the... | by Curt Tigges - Medium, 8월 16, 2025에 액세스, https://medium.com/@curttigges/the-annotated-perceiver-74752113eefb
11. Perceiver IO: a scalable, fully-attentional model that works on any modality - Hugging Face, 8월 16, 2025에 액세스, https://huggingface.co/blog/perceiver
12. Perceiver: General Perception with Iterative Attention, 8월 16, 2025에 액세스, http://proceedings.mlr.press/v139/jaegle21a/jaegle21a.pdf
13. General Perception with Iterative Attention - Perceiver - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/pdf/2103.03206
14. Perceiver: General Perception with Iterative Attention | by Christian Lin | Medium, 8월 16, 2025에 액세스, https://medium.com/@crlc112358/multi-agent-reinforcement-learning-paper-reading-perceiver-general-perception-with-iterative-8d93fe255686
15. Building architectures that can handle the world's data - Google DeepMind, 8월 16, 2025에 액세스, https://deepmind.google/discover/blog/building-architectures-that-can-handle-the-worlds-data/
16. Paper page - Perceiver: General Perception with Iterative Attention - Hugging Face, 8월 16, 2025에 액세스, https://huggingface.co/papers/2103.03206
17. [2103.03206] Perceiver: General Perception with Iterative Attention - ar5iv - arXiv, 8월 16, 2025에 액세스, https://ar5iv.labs.arxiv.org/html/2103.03206
18. Perceiver: General Perception with Iterative Attention - The VITALab website, 8월 16, 2025에 액세스, https://vitalab.github.io/article/2021/07/22/Perceiver.html
19. From Set Transformer to Perceiver Sampler | Towards Data Science, 8월 16, 2025에 액세스, https://towardsdatascience.com/from-set-transformer-to-perceiver-sampler-2f18e741d242/
20. Perceiver IO: A General Architecture for Structured Inputs & Outputs - ResearchGate, 8월 16, 2025에 액세스, https://www.researchgate.net/publication/353634901_Perceiver_IO_A_General_Architecture_for_Structured_Inputs_Outputs
21. [2107.14795] Perceiver IO: A General Architecture for Structured Inputs & Outputs - ar5iv, 8월 16, 2025에 액세스, https://ar5iv.labs.arxiv.org/html/2107.14795
22. PERCEIVER IO: A GENERAL ARCHITECTURE FOR STRUCTURED INPUTS & OUTPUTS - OpenReview, 8월 16, 2025에 액세스, https://openreview.net/pdf?id=fILj7WpI-g
23. DeepMind Perceiver and Perceiver IO | Paper Explained - YouTube, 8월 16, 2025에 액세스, https://www.youtube.com/watch?v=WJWBq4NZfvY
24. Perceiver IO - Paper Summary, 8월 16, 2025에 액세스, https://medium.com/ml-summaries/perceiver-io-paper-summary-e8f28e451d21
25. Perceiver IO: A General Architecture for Structured Inputs & Outputs | OpenReview, 8월 16, 2025에 액세스, https://openreview.net/forum?id=fILj7WpI-g
26. Perceiver - Wikipedia, 8월 16, 2025에 액세스, https://en.wikipedia.org/wiki/Perceiver
27. Perceiver IO: A General Architecture for Structured Inputs & Outputs by Deepmind. Explained! | by Gaurav Chauhan | Analytics Vidhya | Medium, 8월 16, 2025에 액세스, https://medium.com/analytics-vidhya/perceiver-io-a-general-architecture-for-structured-inputs-outputs-4ad669315e7f
28. [R] DeepMind's Perceiver IO: A General Architecture for a Wide Variety of Inputs & Outputs - Reddit, 8월 16, 2025에 액세스, https://www.reddit.com/r/deeplearning/comments/p12pkt/r_deepminds_perceiver_io_a_general_architecture/
29. Perceiver IO: A General Architecture for Structured Inputs & Outputs | Papers With Code, 8월 16, 2025에 액세스, https://paperswithcode.com/paper/perceiver-io-a-general-architecture-for
30. [2107.14795] Perceiver IO: A General Architecture for Structured Inputs & Outputs - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/abs/2107.14795
31. Revision History for Response to Reviewer 2doh - OpenReview, 8월 16, 2025에 액세스, https://openreview.net/revisions?id=uz3IrYuMwly
32. General-purpose, long-context autoregressive modeling with Perceiver AR - Proceedings of Machine Learning Research, 8월 16, 2025에 액세스, https://proceedings.mlr.press/v162/hawthorne22a/hawthorne22a.pdf
33. Demystifying Sparse Attention: Longformer, BigBird, Reformer, and Linformer Explained | by Boopathi Raj | Jun, 2025 | Medium, 8월 16, 2025에 액세스, https://medium.com/@rajboopathiking/demystifying-sparse-attention-longformer-bigbird-reformer-and-linformer-explained-029b97588144
34. A Practical Survey on Faster and Lighter Transformers - Papers With Code, 8월 16, 2025에 액세스, https://paperswithcode.com/paper/a-practical-survey-on-faster-and-lighter/review/
35. Wayformer: Motion Forecasting via Simple & Efficient Attention Networks | Request PDF - ResearchGate, 8월 16, 2025에 액세스, https://www.researchgate.net/publication/361981954_Wayformer_Motion_Forecasting_via_Simple_Efficient_Attention_Networks
36. (PDF) Hierarchical Perceiver - ResearchGate, 8월 16, 2025에 액세스, https://www.researchgate.net/publication/358795676_Hierarchical_Perceiver
37. Hierarchical Perceiver - CatalyzeX, 8월 16, 2025에 액세스, https://www.catalyzex.com/paper/hierarchical-perceiver
38. HiP: Hierarchical Perceiver, 8월 16, 2025에 액세스, http://arxiv.org/pdf/2202.10890
39. DeepMind's Perceiver IO: A General Architecture for a Wide Variety of Inputs & Outputs, 8월 16, 2025에 액세스, https://syncedreview.com/2021/08/09/deepmind-podracer-tpu-based-rl-frameworks-deliver-exceptional-performance-at-low-cost-78/

