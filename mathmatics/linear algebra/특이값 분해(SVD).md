# 특이값 분해(SVD)


특이값 분해(Singular Value Decomposition, SVD)는 임의의 행렬에 적용 가능한 가장 보편적이고 강력한 행렬 분해 기법으로, 선형대수학의 '꽃(Highlight)'이라 불린다.1 SVD의 이러한 위상은 고유값 분해(Eigendecomposition, EVD)가 정방행렬(square matrix)이라는 제약 조건을 갖는 것과 달리 3, 모든 $m \times n$ 크기의 직사각 행렬(rectangular matrix)에 적용 가능하기 때문이다.2 이러한 범용성은 SVD가 현대 데이터 과학의 수많은 응용 분야에서 핵심적인 역할을 수행하게 하는 근간이 된다.

임의의 행렬 $A$로 표현되는 선형 변환은 기저(basis)의 회전(rotation), 각 축 방향으로의 크기 변환(scaling), 그리고 변환된 기저의 재회전(re-rotation)이라는 세 가지 근원적인 기하학적 변환의 조합으로 이해할 수 있다.2 SVD는 바로 이 직관을 $A = U \Sigma V^T$라는 수학적 형태로 정밀하게 기술하는 강력한 도구이다. 이 분해는 행렬 $A$가 수행하는 복잡한 선형 변환을 세 개의 단순한 변환, 즉 직교변환(orthogonal transformation), 대각변환(diagonal transformation), 그리고 또 다른 직교변환의 순차적 적용으로 해석할 수 있는 길을 열어준다. 본 보고서는 이 분해의 수학적 공리부터 심층적 응용까지를 체계적으로 고찰하고자 한다.


이 장에서는 SVD의 기본 정의를 확립하고, 분해를 구성하는 세 행렬 $U$, $\Sigma$, $V$의 개별적 특성과 상호 관계를 수학적으로 명확히 규명한다.


선형대수학의 기본 정리에 따르면, 임의의 $m \times n$ 실수 행렬 $A$는 다음과 같이 세 행렬의 곱으로 유일하게 분해될 수 있다.11
$$
A = U \Sigma V^T
$$
여기서 $U$는 $m \times m$ 직교 행렬, $\Sigma$는 $m \times n$ 직사각 대각 행렬, 그리고 $V^T$는 $n \times n$ 직교 행렬 $V$의 전치 행렬이다. 이 분해는 행렬 $A$가 수행하는 복잡한 선형 변환을 세 개의 단순한 변환의 순차적 적용으로 해석할 수 있는 수학적 기반을 제공한다.2


SVD를 구성하는 세 행렬은 각각 고유한 수학적 의미와 역할을 지닌다.


- $U$는 $m \times m$ 크기의 직교 행렬(orthogonal matrix)이다.11 직교 행렬의 정의에 따라, 그 열벡터들은 서로 정규 직교(orthonormal)하며, 이는 $U^T U = UU^T = I$ (여기서 $I$는 항등행렬)라는 성질을 만족함을 의미한다.13
- $U$의 열벡터 $u_i$들은 행렬 $A$의 열공간(column space)에 대한 정규 직교 기저를 형성한다.11 이 벡터들은 행렬 $AA^T$의 고유벡터(eigenvector)와 직접적으로 연관된다.16


- $\Sigma$는 $m \times n$ 크기의 직사각 대각 행렬(rectangular diagonal matrix)이다.11 이는 대각 성분 $\Sigma_{ii}$를 제외한 모든 원소가 0임을 의미한다.13
- 대각 성분 $\sigma_i = \Sigma_{ii}$는 행렬 $A$의 **특이값(singular value)**이라 불리며, 항상 0 이상의 실수 값을 갖는다 ($\sigma_i \ge 0$).3
- 특이값들은 관례적으로 큰 값에서 작은 값 순으로 정렬된다: $\sigma_1 \ge \sigma_2 \ge \dots \ge \sigma_r > 0$, 여기서 $r$은 행렬 $A$의 랭크(rank)이다.14


- $V$는 $n \times n$ 크기의 직교 행렬이다.11

  $V$의 열벡터 $v_i$들은 행렬 $A$의 행공간(row space)에 대한 정규 직교 기저를 형성한다.11

- 이 벡터들은 행렬 $A^T A$의 고유벡터와 직접적으로 연관된다.16

- 분해식에서는 $V$가 아닌 그것의 전치 행렬인 $V^T$가 사용된다는 점에 유의해야 한다.


특이값은 행렬의 근본적인 속성을 나타내는 핵심적인 지표이다. 특이값 $\sigma_i$는 $A^T A$ 또는 $AA^T$의 고유값 $\lambda_i$의 양의 제곱근($\sigma_i = \sqrt{\lambda_i}$)으로 정의된다.4

$A^T A$와 $AA^T$가 양의 준정부호 행렬(positive semi-definite matrix)이므로, 이들의 모든 고유값은 0 이상임이 보장되며, 따라서 모든 특이값은 실수가 된다.2

SVD의 중요한 성질 중 하나는 0이 아닌 특이값의 개수가 행렬 $A$의 랭크(rank)와 정확히 일치한다는 점이다.20 이는 SVD가 행렬의 근본적인 차원(effective dimensionality), 즉 행 또는 열 벡터들로 생성되는 공간의 실제 차원을 밝히는 강력한 도구임을 시사한다. 예를 들어, 랭크가 1인 행렬은 0이 아닌 특이값을 단 하나만 가진다.21


SVD는 계산 및 저장 효율성에 따라 두 가지 형태로 구분될 수 있다.

- **완전 SVD (Full SVD):** 위에서 정의된 $U (m \times m)$, $\Sigma (m \times n)$, $V (n \times n)$ 행렬을 모두 사용하는 형태를 지칭한다.23
- **축소 SVD (Reduced SVD / Thin SVD):** 행렬 $\Sigma$의 대각 성분 중 0이 되는 부분과, 이에 대응하는 $U$와 $V$의 열벡터들은 최종 행렬 $A$의 계산에 아무런 기여를 하지 않는다.1 이 불필요한 부분들을 제거하여 행렬의 크기를 줄인 형태를 축소 SVD라 한다. 만약 $\text{rank}(A) = r$이라면, $U$는 $m \times r$, $\Sigma$는 $r \times r$, $V^T$는 $r \times n$ 크기로 축소될 수 있다.1 이는 데이터 저장 공간을 절약하고 계산 효율성을 비약적으로 높이는 데 매우 중요하며, 대부분의 실제 응용에서는 축소 SVD가 사용된다.24

아래 표는 SVD를 구성하는 각 행렬의 속성을 요약한 것이다.

| 행렬          | 표기     | 크기         | 종류                    | 주요 속성 및 의미                                            |
| ------------- | -------- | ------------ | ----------------------- | ------------------------------------------------------------ |
| 원본 행렬     | $A$      | $m \times n$ | 임의의 실수 행렬        | 분해의 대상이 되는 행렬, $n$차원 벡터를 $m$차원 벡터로 변환하는 선형 변환 |
| 좌측 특이벡터 | $U$      | $m \times m$ | 직교 행렬 ($U^T U = I$) | $A$의 열공간(Column Space)과 좌측 영공간(Left Null Space)의 정규 직교 기저. $AA^T$의 고유벡터로 구성됨. |
| 특이값 행렬   | $\Sigma$ | $m \times n$ | 직사각 대각 행렬        | 대각 성분 $\sigma_i$는 특이값 ($\sigma_i \ge 0$). $A$의 선형 변환에 의한 각 축 방향의 크기 변환(scaling) 정도를 나타냄. |
| 우측 특이벡터 | $V$      | $n \times n$ | 직교 행렬 ($V^T V = I$) | $A$의 행공간(Row Space)과 영공간(Null Space)의 정규 직교 기저. $A^T A$의 고유벡터로 구성됨. |

SVD는 단순히 행렬 $A$를 분해하는 것을 넘어, 선형대수학의 기본 정리에 의해 모든 행렬이 갖는 4개의 기본 부분 공간(열공간, 행공간, 영공간, 좌측 영공간)에 대한 직교 기저를 한 번에 제공한다.15

$U$의 첫 $r$개 열은 열공간의 기저, 나머지 $m-r$개 열은 좌측 영공간의 기저가 된다.1 마찬가지로 $V$의 첫 $r$개 열은 행공간의 기저, 나머지 $n-r$개 열은 영공간의 기저가 된다.1 이는 SVD가 행렬의 구조를 가장 근본적인 수준에서 해부하는 '해부도'와 같음을 보여준다.

또한, 특이값이 큰 순서대로 정렬된다는 점 21은 단순한 관례가 아니다. 이는 각 특이값과 그에 대응하는 특이벡터 쌍($\sigma_i, u_i, v_i$)이 원본 행렬 $A$를 구성하는 데 기여하는 '정보량' 또는 '에너지'의 크기를 나타낸다. 행렬 $A$를 랭크-1 행렬들의 합으로 표현하면 $A = \sigma_1 u_1 v_1^T + \sigma_2 u_2 v_2^T + \dots$ 3 형태로 볼 수 있는데, 여기서 

$\sigma_1$이 가장 큰 비중을 차지하는 '주성분' 역할을 한다. 이 정보의 계층적 구조는 SVD가 데이터 압축 및 차원 축소에 왜 그토록 효과적인지에 대한 근본적인 이유를 제공한다.


이 장에서는 SVD가 어떻게 수학적으로 유도되는지를 단계별로 추적하고, 이를 통해 SVD가 갖는 깊은 기하학적 의미를 탐구한다.


SVD 유도의 출발점은 원본 행렬 $A$로부터 두 개의 특별한 정방행렬, 즉 $A^T A$와 $AA^T$를 구성하는 것이다.4 이 두 행렬은 SVD를 유도하는 데 필수적인 두 가지 중요한 성질을 지닌다.

첫째, 이 두 행렬은 항상 **대칭행렬(Symmetric Matrix)**이다. 예를 들어, $(A^T A)^T = A^T (A^T)^T = A^T A$이므로 $A^T A$는 대칭이다.3 대칭행렬은 항상 실수 고유값을 가지며, 서로 다른 고유값에 대응하는 고유벡터들은 서로 직교한다는 중요한 성질을 갖는다.20 이는 SVD의 구성 요소인 $U$와 $V$가 직교 행렬이 될 수 있는 이론적 기반이 된다.

둘째, 이 두 행렬은 **양의 준정부호 행렬(Positive Semi-definite Matrix)**이다. 임의의 영벡터가 아닌 벡터 $x$에 대해 $x^T (A^T A) x = (Ax)^T (Ax) = \|Ax\|^2 \ge 0$ 이기 때문이다.3 이 성질은 이 행렬들의 모든 고유값이 0 이상임을 보장하며, 따라서 특이값($\sigma_i = \sqrt{\lambda_i}$)이 항상 실수 값이 되게 하는 결정적인 역할을 한다.2 이처럼 다루기 힘든 일반 행렬 $A$의 문제를, 해법이 잘 알려진 대칭 행렬의 고유값 문제로 변환하여 푸는 것은 SVD 유도의 정교한 수학적 전략이다.


SVD는 $A^T A$의 고유값 분해로부터 체계적으로 유도될 수 있다.20

1. **$V$와 $\Sigma$ 계산:** 먼저 $n \times n$ 크기의 대칭행렬 $A^T A$의 고유값 분해를 수행한다.
   $$
   A^T A = V \Lambda V^T
   $$

   - $V$는 $A^T A$의 정규 직교 고유벡터들을 열로 갖는 $n \times n$ 직교 행렬이다. 이 고유벡터들이 바로 SVD의 우측 특이벡터 $v_i$가 된다.17
   - $\Lambda$는 $A^T A$의 고유값 $\lambda_i$를 대각 원소로 갖는 대각 행렬이다.
   - 특이값 $\sigma_i$는 고유값의 양의 제곱근, 즉 $\sigma_i = \sqrt{\lambda_i}$로 계산된다. 이들을 내림차순으로 정렬하여 $m \times n$ 크기의 특이값 행렬 $\Sigma$를 구성한다.4

2. **$U$ 계산:** 좌측 특이벡터 행렬 $U$의 열벡터 $u_i$는 이미 계산된 $v_i$와 $\sigma_i$를 통해 다음과 같이 유도된다.
   $$
   A v_i = \sigma_i u_i \implies u_i = \frac{1}{\sigma_i} A v_i \quad (\text{for } \sigma_i > 0)
   $$

   - 이 관계식은 $AV = U\Sigma$ 9라는 SVD의 핵심 관계로부터 직접 도출된다. 이 식을 통해 얻어진 

     $u_i$들이 $m \times m$ 행렬 $AA^T$의 고유벡터가 됨을 수학적으로 증명할 수 있다. $A(A^T A)v_i = A(\lambda_i v_i)$ 식의 양변을 정리하면 $(AA^T)(Av_i) = \lambda_i(Av_i)$가 되므로, $Av_i$는 고유값 $\lambda_i$에 대응하는 $AA^T$의 고유벡터임을 알 수 있다.20

3. **최종 결합:** 계산된 $U$, $\Sigma$, $V$를 결합하여 최종적인 SVD 형태를 완성한다. $AV = U\Sigma$의 양변 우측에 $V^T$를 곱하면 $AVV^T = U\Sigma V^T$가 된다. $V$가 직교 행렬이므로 $VV^T = I$이며, 따라서 $A = U\Sigma V^T$가 성립한다.9


SVD의 진정한 힘은 그 기하학적 해석에서 드러난다. 임의의 벡터 $x$에 대한 선형 변환 $Ax$는 $U(\Sigma(V^T x))$라는 세 단계의 순차적 변환 과정으로 해석될 수 있다.2

1. **$V^T$ (회전/반사):** $V^T$는 직교 변환이므로, 입력 벡터 $x$의 길이는 바꾸지 않고 좌표계를 회전(또는 반사)시킨다. 이는 입력 공간($R^n$)에서 변환의 본질을 가장 잘 설명할 수 있는 '올바른' 기저($v_i$들)를 선택하는 과정으로 볼 수 있다.2
2. **$\Sigma$ (크기 변환):** $\Sigma$는 대각 행렬이므로, $V^T$에 의해 회전된 벡터의 각 축 성분을 해당 특이값 $\sigma_i$만큼 늘리거나 줄인다(scaling). 이 단계에서 기하학적 형태의 왜곡(예: 원이 타원으로 변환)이 발생하며, $n$차원 공간에서 $m$차원 공간으로의 차원 변경도 일어날 수 있다.2
3. **$U$ (재회전/반사):** $U$ 역시 직교 변환으로, 크기가 조절된 벡터를 출력 공간($R^m$)의 최종 위치로 회전(또는 반사)시킨다. 이는 출력 공간에서 변환 결과를 가장 잘 설명하는 '올바른' 기저($u_i$들)를 선택하는 과정이다.2

이러한 기하학적 해석은 SVD가 단순히 행렬을 분해하는 계산 기법이 아님을 보여준다. 이는 모든 복잡한 선형 변환 $A$에 대해, 그 변환의 본질을 가장 단순하게 표현할 수 있는 최적의 입력 기저($V$)와 출력 기저($U$)가 항상 존재함을 증명한다. 이 최적의 기저들 위에서, 변환 $A$는 단지 축별 크기 조절($\Sigma$)이라는 지극히 단순한 행위로 귀결된다. 따라서 SVD는 모든 선형 변환의 '유전 암호' 또는 '표준 형식(Canonical Form)'을 해독하는 근본적인 도구로 볼 수 있다. 시각적으로, SVD는 입력 공간의 단위 구(unit sphere)가 선형 변환 $A$에 의해 어떻게 출력 공간의 타원체(ellipsoid)로 변환되는지를 설명한다.27 이때 

$V$의 열벡터($v_i$)는 타원체의 주축(principal axes) 방향으로 정렬되는 입력 벡터들이고, $U$의 열벡터($u_i$)는 변환된 타원체의 주축 방향이며, 특이값($\sigma_i$)은 각 주축의 길이가 된다.2


이 장에서는 SVD를 고유값 분해, 최적 근사 이론, 주성분 분석과 같은 선형대수학의 다른 핵심 개념들과 비교하고 연결함으로써 그 이론적 위치를 공고히 한다.


고유값 분해(EVD)는 정방행렬 $A$에 대해 $Av = \lambda v$를 만족하는 고유값 $\lambda$와 고유벡터 $v$를 찾는 과정이다. $A$가 $n$개의 선형 독립인 고유벡터를 가지면 $A = PDP^{-1}$로 분해할 수 있다.1 SVD는 이러한 EVD의 개념을 일반화한 형태로 볼 수 있다.4

- $A$의 우측 특이벡터($V$의 열)는 $A^T A$의 고유벡터이다.8
- $A$의 좌측 특이벡터($U$의 열)는 $AA^T$의 고유벡터이다.8
- $A$의 0이 아닌 특이값들은 $A^T A$와 $AA^T$의 고유값들의 양의 제곱근이다.8

$A$가 대칭 행렬($A = A^T$)일 경우, SVD와 EVD는 매우 밀접해진다. 이 경우 $A$의 특이값들의 절댓값은 $A$의 고유값들의 절댓값과 같다. 그러나 두 분해법은 근본적인 차이가 있으며, 아래 표는 이를 명확히 비교한다.

| 구분              | 특이값 분해 (SVD)                                 | 고유값 분해 (EVD)                                            |
| ----------------- | ------------------------------------------------- | ------------------------------------------------------------ |
| **적용 대상**     | 모든 $m \times n$ 행렬 (정방/직사각) 3            | 정방행렬 ($n \times n$)만 가능, 대각화는 선형 독립인 고유벡터가 존재해야 함 9 |
| **분해 형태**     | $A = U\Sigma V^T$                                 | $A = PDP^{-1}$                                               |
| **구성 행렬**     | $U$, $V$: 직교 행렬, $\Sigma$: (직사각) 대각 행렬 | $P$: 고유벡터 행렬 (일반적으로 비직교), $D$: 고유값 대각 행렬 |
| **기저의 직교성** | 항상 두 개의 직교 기저 ($U$, $V$)를 제공함        | 대칭 행렬일 경우에만 직교 기저($P$)를 보장함 17              |
| **대각 원소**     | 특이값 ($\sigma_i \ge 0$)                         | 고유값 ($\lambda_i$, 양/음/복소수 가능)                      |
| **핵심 의미**     | 변환의 기하학적 분해 (회전-스케일링-회전)         | 변환에 의해 방향이 불변인 벡터(고유벡터)와 그 스케일링 비율(고유값) 2 |

EVD는 $Av = \lambda v$라는 식에서 보듯, 하나의 변환 $A$가 자기 자신의 공간 내에서 어떻게 작용하는지(불변 방향)를 분석한다. 반면 SVD($Av_i = \sigma_i u_i$)는 입력 공간($R^n$의 기저 $v_i$)의 벡터가 출력 공간($R^m$의 기저 $u_i$)의 벡터로 어떻게 '매핑'되는지를 분석한다. 이 관점은 SVD가 두 개의 다른 공간(예: 사용자 공간과 아이템 공간, 단어 공간과 문서 공간) 사이의 관계를 모델링하는 데 왜 그토록 강력한지를 설명해준다.


SVD의 가장 중요한 응용 중 하나는 행렬을 더 낮은 랭크의 행렬로 근사하는 것이다. 에카르트-영(Eckart-Young) 정리는 이러한 근사가 '최적'임을 수학적으로 보장한다.

이 정리에 따르면, 주어진 행렬 $A$를 랭크 $k$ ($k < \text{rank}(A)$)인 행렬 $A_k$로 근사할 때, 프로베니우스 노름(Frobenius norm) $\|A - A_k\|_F$를 최소화하는 최적의 근사 행렬은 SVD의 가장 큰 $k$개의 특이값과 그에 대응하는 특이벡터들로 구성된다.9
$$
A_k = \sum_{i=1}^{k} \sigma_i u_i v_i^T
$$
이 정리는 SVD가 왜 데이터 압축과 노이즈 제거에 이론적으로 최적인지를 설명한다.9 단순히 상위 $k$개의 특이값만 선택하는 '절단된 SVD(Truncated SVD)' 19가 다른 어떤 랭크-$k$ 근사보다 원본 데이터를 가장 잘 보존함을 의미한다.


주성분 분석(PCA)은 데이터의 분산을 최대로 보존하는 새로운 직교 기저(주성분, Principal Components)를 찾는 대표적인 차원 축소 기법이다.31 전통적으로 PCA는 데이터의 공분산 행렬(Covariance Matrix)을 고유값 분해하여 수행되지만 31, SVD를 통해 더 직접적이고 수치적으로 안정적인 방법으로 수행할 수 있다.

평균이 0으로 맞춰진(mean-centered) 데이터 행렬 $X$ ($n$개의 샘플, $p$개의 변수)가 있다고 가정하자. 공분산 행렬 $C$는 $C = \frac{1}{n-1} X^T X$로 계산된다. 이제 $X$에 대한 SVD를 $X = U\Sigma V^T$라고 하면, 공분산 행렬 $C$는 SVD 항으로 다음과 같이 표현될 수 있다.34
$$
C = \frac{1}{n-1} (U\Sigma V^T)^T (U\Sigma V^T) = \frac{1}{n-1} (V\Sigma^T U^T U \Sigma V^T)
$$
$U$가 직교 행렬이므로 $U^T U = I$이다. 따라서,
$$
C = \frac{1}{n-1} V \Sigma^T \Sigma V^T = \frac{1}{n-1} V \Sigma^2 V^T
$$
이 식은 $C$의 고유값 분해 형태와 정확히 일치한다. 이로부터 SVD와 PCA 사이의 직접적인 관계를 도출할 수 있다.35

- **주 방향(Principal Directions):** 공분산 행렬 $C$의 고유벡터 행렬은 SVD의 우측 특이벡터 행렬 $V$와 동일하다. 즉, $V$의 열벡터($v_i$)들이 바로 데이터의 주 방향이다.34
- **주성분 분산:** $C$의 고유값 $\lambda_i$는 $A^T A$의 고유값과 같으므로 $\lambda_i = \sigma_i^2 / (n-1)$ 관계를 갖는다. 즉, 특이값의 제곱이 주성분의 분산에 비례한다.34
- **주성분 점수(Principal Component Scores):** 데이터 $X$를 새로운 주성분 축으로 사영(projection)한 결과는 $XV = (U\Sigma V^T)V = U\Sigma$이다. 즉, $U\Sigma$의 열들이 바로 주성분 점수이다.35

PCA의 핵심 목표인 '분산 최대화'는 기하학적으로 데이터 포인트들을 가장 잘 표현하는 저차원 부분 공간을 찾는 것과 동일하다.9 에카르트-영 정리에 따르면, 이 최적의 부분 공간은 SVD에 의해 자연스럽게 찾아진다. 따라서 SVD는 PCA의 이론적 정당성과 계산적 구현을 모두 제공하는 핵심 '엔진' 역할을 한다. 공분산 행렬을 명시적으로 계산하지 않고 데이터 행렬 

$X$에 직접 SVD를 적용하여 PCA를 수행하는 방식은 수치적으로 더 안정적이며, 특히 변수의 수가 샘플 수보다 훨씬 많은 고차원 데이터($p \gg n$)에서 계산적으로 유리하다.37


이 장에서는 앞서 다룬 SVD의 이론적 특성들이 어떻게 실제 데이터 과학 문제 해결에 적용되는지를 구체적인 사례를 통해 심층적으로 탐구한다. SVD는 이미지 처리, 추천 시스템, 자연어 처리, 수치 해석 등 다양한 분야에서 데이터의 잠재적 구조를 파악하고 문제를 해결하는 데 핵심적인 역할을 수행한다.


디지털 이미지는 픽셀 값으로 이루어진 행렬로 표현될 수 있다. 컬러 이미지의 경우, 빨강(R), 초록(G), 파랑(B) 각 채널별로 하나의 행렬이 존재한다.39 SVD는 이러한 이미지 행렬을 효율적으로 처리하는 데 탁월한 성능을 보인다.

- **압축 원리:** 에카르트-영 정리에 따라, SVD를 통해 얻은 상위 $k$개의 특이값과 특이벡터로 재구성한 랭크-$k$ 행렬 $A_k$는 원본 이미지 행렬 $A$의 최적 근사이다.9 원본 

  $m \times n$ 행렬을 저장하는 대신, $A_k = U_k \Sigma_k V_k^T$를 구성하는 요소들($m \times k$ 행렬 $U_k$, $k \times k$ 행렬 $\Sigma_k$, $k \times n$ 행렬 $V_k^T$)을 저장하면 훨씬 적은 데이터($m \cdot k + k + k \cdot n$)를 요구한다.1

  $k$값을 조절함으로써 압축률과 이미지 품질 사이의 균형을 맞출 수 있다.39

- **노이즈 제거 원리:** 이미지의 주요 정보(핵심 특징, 윤곽 등)는 큰 특이값에 집중되어 있고, 노이즈와 같은 미세하고 무작위적인 정보는 작은 특이값에 주로 분포한다.42 따라서 작은 특이값들을 0으로 만들고 이미지를 재구성하면(즉, 절단된 SVD를 사용하면) 이미지의 본질적인 구조는 유지하면서 노이즈가 효과적으로 제거된다.19


추천 시스템의 핵심 과제 중 하나는 사용자-아이템 평점 행렬 $R$의 비어있는 값을 예측하는 것이다. 이 행렬은 대부분의 사용자가 소수의 아이템만 평가하기 때문에 극도로 희소(sparse)한 특징을 갖는다.46

- **잠재 요인 모델(Latent Factor Model):** SVD는 $R$을 두 개의 저차원 행렬, 즉 사용자-잠재요인 행렬 $P$와 아이템-잠재요인 행렬 $Q$로 분해하는 행렬 분해(Matrix Factorization)의 이론적 기반을 제공한다. 목표는 $R \approx P \cdot Q^T$를 만족하는 $P$와 $Q$를 찾는 것이다.48

- **잠재 요인의 의미:** 여기서 $k$차원의 잠재 요인은 사용자의 취향(예: 특정 장르 선호도)이나 아이템의 속성(예: 영화의 장르, 배우)을 나타내는 추상적인 벡터이다.48 사용자 

  $u$의 아이템 $i$에 대한 예측 평점 $\hat{r}_{ui}$는 사용자 잠재 벡터 $p_u$와 아이템 잠재 벡터 $q_i$의 내적으로 계산된다: $\hat{r}_{ui} \approx p_u^T q_i$.46

- **SVD의 역할:** 전통적인 SVD는 희소 행렬에 직접 적용하기 어렵기 때문에, 실제 추천 시스템에서는 경사 하강법(Gradient Descent)이나 교대 최소 제곱법(Alternating Least Squares) 등을 사용하여 관측된 평점만을 이용해 $P$와 $Q$를 최적화하는 SVD 기반의 변형 알고리즘(예: Funk SVD, SVD++)이 널리 사용된다.51


단어-문서 행렬(Term-Document Matrix, TDM)이나 TF-IDF 행렬은 단어의 빈도수만을 기반으로 하므로, 동의어(synonymy)나 다의어(polysemy) 문제를 해결하기 어렵다.23 LSA는 SVD를 이용해 이 문제를 해결한다.

- **LSA의 원리:** LSA는 TDM에 절단된 SVD를 적용하여 단어와 문서를 저차원의 '잠재 의미 공간(latent semantic space)'으로 사영시킨다.30
  $$
  \text{TDM} \approx U_k \Sigma_k V_k^T
  $$

- **임베딩 벡터:** 이 분해에서 $U_k$의 행은 각 단어를 나타내는 '단어 임베딩' 벡터가 되고, $V_k^T$의 열(또는 $V_k$의 행)은 각 문서를 나타내는 '문서 임베딩' 벡터가 된다.30

  $\Sigma_k$의 대각 원소는 각 잠재 의미(토픽)의 중요도를 나타낸다.56

- **효과:** 이 잠재 공간에서는 문맥적으로 유사한 단어들(예: '자동차'와 '승용차')이 벡터 공간상에서 가깝게 위치하게 된다. 이를 통해 문서 간 유사도 계산(주로 코사인 유사도 사용)의 정확도를 높이고, 정보 검색 시스템의 성능을 향상시킬 수 있다.54


정방행렬이 아니거나 비가역적인(singular) 행렬 $A$는 역행렬을 갖지 않는다. 이 경우 선형 연립방정식 $Ax = b$의 해를 직접 구할 수 없다.57 의사역행렬은 이러한 문제에 대한 해법을 제공한다.

- **의사역행렬의 정의:** 역행렬의 개념을 일반화한 것으로, $A^+$로 표기한다. 이는 $Ax = b$에 대한 최적의 근사해, 즉 잔차 $\|Ax - b\|^2$를 최소화하는 최소 자승 해(least squares solution) $\hat{x} = A^+ b$를 제공한다.11

- **SVD를 이용한 계산:** $A = U\Sigma V^T$일 때, 의사역행렬 $A^+$는 다음과 같이 간단하고 안정적으로 계산된다.36
  $$
  A^+ = V \Sigma^+ U^T
  $$

- **$\Sigma^+$의 계산:** $\Sigma^+$는 $\Sigma$의 0이 아닌 대각 원소들의 역수를 취하고 행렬을 전치(transpose)하여 얻는다.36 즉, 대각 원소 $\sigma_i$는 $1/\sigma_i$가 되고, 행렬의 크기는 $n \times m$이 된다.

SVD는 어떤 행렬에 대해서도 의사역행렬을 안정적으로 계산할 수 있는 강력하고 일반적인 방법을 제공한다.61

이상의 응용 사례들은 서로 다른 분야의 문제처럼 보이지만, SVD의 관점에서는 모두 동일한 작업을 수행하고 있다. 즉, 원본 데이터 행렬에서 '핵심 개념' 또는 '잠재 구조'를 추출하고, '노이즈' 또는 '세부 정보'를 분리하는 것이다. 이미지에서는 '주요 형태', 추천에서는 '잠재 취향', LSA에서는 '주제(토픽)'가 바로 SVD가 추출하는 '개념'에 해당한다. 이처럼 SVD는 데이터의 종류에 구애받지 않고 저차원의 잠재 의미 공간을 발견하는 보편적인 수학적 프레임워크를 제공한다. 또한, 절단된 SVD에서 상위 $k$개의 특이값만 사용하는 것은 모델이 데이터의 가장 중요한 패턴에만 집중하고 덜 중요한 노이즈는 무시하도록 강제하는 것과 같다. 이는 모델의 복잡도(행렬의 랭크)를 인위적으로 제한하는 행위이므로, 머신러닝에서의 정규화(Regularization)와 유사한 효과를 가져와 과적합(overfitting)을 방지하는 데 기여한다.


빅데이터 시대에 접어들면서 SVD를 실용적으로 적용하기 위한 효율적인 알고리즘의 필요성이 대두되었다. 전통적인 SVD 알고리즘은 대규모 행렬에 적용하기에는 계산 비용이 너무 크기 때문이다.


$m \times n$ 크기의 행렬에 대한 전통적인 SVD 알고리즘의 계산 복잡도는 대략 $O(\min(m^2 n, mn^2))$ 수준이다. 이는 행렬의 행 또는 열의 크기가 커질수록 계산 비용이 기하급수적으로 증가함을 의미하며, 현대의 대규모 데이터셋에는 적용하기 어렵게 만드는 주요한 병목 현상이다.63


이러한 계산적 한계를 극복하기 위해 무작위 SVD와 같은 근사 알고리즘이 개발되었다.

- **핵심 아이디어:** 전체 행렬을 직접 분해하는 대신, 행렬의 열공간(column space)을 잘 대표하는 저차원 부분 공간을 무작위 샘플링을 통해 빠르게 찾고, 이 부분 공간에 투영된 작은 행렬에 대해서만 SVD를 수행하여 전체 행렬의 SVD를 근사하는 방식이다.63
- 알고리즘 단계 63:
  1. **샘플링(Sampling):** $n \times k$ 크기의 무작위 행렬 $P$를 생성한다. 여기서 $k$는 목표 랭크로, $n$보다 훨씬 작은 값이다.
  2. **스케치(Sketch) 형성:** $Z = A \cdot P$를 계산한다. $Z$는 $m \times k$ 크기의 작은 행렬로, $A$의 열공간에 대한 '스케치' 역할을 하며, 높은 확률로 $A$의 주요 정보를 담고 있다.
  3. **직교 기저 구축:** $Z$에 대해 QR 분해를 수행하여 정규 직교 기저 $Q$를 얻는다 ($Z = QR$). $Q$는 $A$의 열공간에 대한 근사 기저가 된다.
  4. **투영(Projection):** 원본 행렬 $A$를 기저 $Q$에 투영하여 더 작은 $k \times n$ 행렬 $Y = Q^T \cdot A$를 만든다.
  5. **소규모 SVD:** 이제 크기가 훨씬 작은 행렬 $Y$에 대해 전통적인 SVD를 수행한다: $Y = \tilde{U}\Sigma V^T$.
  6. **최종 $U$ 복원:** 최종 좌측 특이벡터 행렬 $U$는 $U = Q \cdot \tilde{U}$로 근사한다.
- **장점:** 계산 복잡도를 크게 낮추면서도, 원본 SVD와 매우 유사한 수준의 정확도를 제공한다. 특히 희소 행렬(sparse matrix) 처리에 효과적이며, 대규모 데이터 분석에서 SVD의 실용성을 크게 확장시켰다.64


특이값 분해는 임의의 행렬을 기하학적으로 의미 있는 세 가지 변환(회전-크기변환-회전)으로 분해하는 강력하고 우아한 수학적 도구이다. 이는 행렬의 4대 기본 부분 공간을 명확히 밝히고, 정보의 중요도에 따라 데이터를 계층적으로 구조화하며, 고유값 분해 및 주성분 분석과 같은 다른 핵심 선형대수 이론의 근간을 이룬다.

SVD의 이러한 이론적 완전성은 데이터 압축, 노이즈 제거, 추천 시스템, 잠재 의미 분석, 최소 자승 문제 해결 등 광범위한 응용 분야에서 실질적인 문제 해결 능력으로 발현된다. SVD는 복잡하고 노이즈가 많은 고차원 데이터 속에서 핵심적인 패턴과 잠재적인 구조를 추출하는 보편적인 메커니즘을 제공한다.

대규모 데이터 처리를 위한 무작위 SVD와 같은 현대적 알고리즘의 발전은 SVD의 적용 범위를 계속해서 넓히고 있으며, 앞으로도 SVD는 데이터 기반 과학과 공학의 발전에 있어 핵심적인 역할을 수행할 것으로 전망된다. 결론적으로, SVD는 단순한 행렬 분해 기법을 넘어, 복잡한 데이터 속에 숨겨진 구조와 의미를 발견하는 보편적인 '렌즈'라 할 수 있다.


1. SVD (특이값 분해) - 하고 싶은 일을 하자 - 티스토리, 8월 17, 2025에 액세스, https://dowhati1.tistory.com/7
2. [선형대수학 #4] 특이값 분해(Singular Value Decomposition, SVD)의 활용 - 다크 프로그래머, 8월 17, 2025에 액세스, https://darkpgmr.tistory.com/106
3. 특이값 분해(Singular Value Decompostion) - 생각과 고민., 8월 17, 2025에 액세스, https://gguguk.github.io/posts/SVD/
4. 특이값 분해(Singular Value Decomposition, SVD) - Studying data - 티스토리, 8월 17, 2025에 액세스, https://my-mindpalace.tistory.com/10
5. SVD의 이해 - 바보 같지만 괜찮아. - 티스토리, 8월 17, 2025에 액세스, https://onebyonebyone.tistory.com/74
6. 고윳값 분해, 특이값 분해(SVD) - 성장通 - 티스토리, 8월 17, 2025에 액세스, https://kevin-rain.tistory.com/157
7. 머신러닝을 위한 수학 정리: Matrix Decomposition - From I.E To NLP, 8월 17, 2025에 액세스, https://inhyeokyoo.github.io/project/mml/Matrix-Decomposition/
8. [선형대수] 특이값 분해 (SVD, Singular Value Decomposition) - R, Python 분석과 프로그래밍의 친구 (by R Friend) - 티스토리, 8월 17, 2025에 액세스, https://rfriend.tistory.com/185
9. [Linear Algebra] Singular Value Decomposition (SVD), Eckart ..., 8월 17, 2025에 액세스, https://yai-yonsei.tistory.com/24
10. 머신러닝 - 20. 특이값 분해(SVD) - 귀퉁이 서재 - 티스토리, 8월 17, 2025에 액세스, [https://bkshin.tistory.com/entry/%EB%A8%B8%EC%8B%A0%EB%9F%AC%EB%8B%9D-20-%ED%8A%B9%EC%9D%B4%EA%B0%92-%EB%B6%84%ED%95%B4Singular-Value-Decomposition](https://bkshin.tistory.com/entry/머신러닝-20-특이값-분해Singular-Value-Decomposition)
11. 특이값 분해(Singular Value Decomposition, SVD) - MINISTOP - 티스토리, 8월 17, 2025에 액세스, https://kxngmxn.tistory.com/36
12. kxngmxn.tistory.com, 8월 17, 2025에 액세스, [https://kxngmxn.tistory.com/36#:~:text=%ED%8A%B9%EC%9D%B4%EA%B0%92%20%EB%B6%84%ED%95%B4(Singular%20Value%20Decomposition%2C%20SVD)%EB%8A%94%20%EC%84%A0%ED%98%95,%EC%9C%BC%EB%A1%9C%20%EB%B6%84%ED%95%B4%ED%95%98%EB%8A%94%20%EB%B0%A9%EB%B2%95%EC%9D%B4%EB%8B%A4.](https://kxngmxn.tistory.com/36#:~:text=특이값 분해(Singular Value Decomposition%2C SVD)는 선형,으로 분해하는 방법이다.)
13. 특이값 분해(SVD) - 공돌이의 수학정리노트 (Angelo's Math Notes), 8월 17, 2025에 액세스, https://angeloyeo.github.io/2019/08/01/SVD.html
14. Singular value decomposition (SVD) - velog, 8월 17, 2025에 액세스, https://velog.io/@kimgeonhee317/Singular-value-decomposition-SVD
15. 44 Singular Value Decomposition SVD - YouTube, 8월 17, 2025에 액세스, https://www.youtube.com/watch?v=HFY6n4CfnSo
16. [ML] 특이값 분해(Singular Value Decomposition, SVD) - Deeppago's study note - 티스토리, 8월 17, 2025에 액세스, https://deeppago.tistory.com/87
17. 특이값 분해(SVD) - gaussian37, 8월 17, 2025에 액세스, https://gaussian37.github.io/math-la-svd/
18. 머신러닝 핵심 요약 5 (EVD, SVD, PCA, LDA) - Hajeong's log - 티스토리, 8월 17, 2025에 액세스, [https://jeong-devlog.tistory.com/entry/%EB%A8%B8%EC%8B%A0%EB%9F%AC%EB%8B%9D-%ED%95%B5%EC%8B%AC-%EC%9A%94%EC%95%BD-5-EVD-SVD-PCA-LDA](https://jeong-devlog.tistory.com/entry/머신러닝-핵심-요약-5-EVD-SVD-PCA-LDA)
19. 특이값 분해(SVD, Singular Value Decomposition) - Everything - 티스토리, 8월 17, 2025에 액세스, https://hyebiness.tistory.com/15
20. 특이값 분해(SVD)의 증명 - DeepCampus - 티스토리, 8월 17, 2025에 액세스, https://pasus.tistory.com/16
21. 특이값 분해(singular value decomposition) - DeepCampus - 티스토리, 8월 17, 2025에 액세스, https://pasus.tistory.com/15
22. SVD (Singular Vector Decomposition) - 파고파고 - 티스토리, 8월 17, 2025에 액세스, https://dippingtodeepening.tistory.com/40
23. LSA - 잠재 의미 분석 | SOOJLE, 8월 17, 2025에 액세스, https://soojle.gitbook.io/project/requirements/undefined-1/undefined-3/lsa
24. SVD(Singular Value Decomposition) - 특이값 분해 - velog, 8월 17, 2025에 액세스, [https://velog.io/@rkdtjdals97/SVDSingular-Value-Decomposition-%ED%8A%B9%EC%9D%B4%EA%B0%92-%EB%B6%84](https://velog.io/@rkdtjdals97/SVDSingular-Value-Decomposition-특이값-분)
25. [선형대수학] 6.5 Reduced SVD, 유사역행렬(Pseudoinverse) - 딥러닝 공부방 - 티스토리, 8월 17, 2025에 액세스, https://deep-learning-study.tistory.com/482
26. Singular Value Decomposition 의 개념 소개 - Soon's Blog, 8월 17, 2025에 액세스, https://meme2515.github.io/machine_learning/svd/
27. Singular value decomposition - Wikipedia, 8월 17, 2025에 액세스, https://en.wikipedia.org/wiki/Singular_value_decomposition
28. [LG AiMERS] Part 2-2 Matrix Decomposition(SVD, Eigen Decompostion) - 곰곰의 일지, 8월 17, 2025에 액세스, https://secundo.tistory.com/89
29. [차원축소][PCA의 원리] Principal component analysis - Science Vibe - 티스토리, 8월 17, 2025에 액세스, https://jrc-park.tistory.com/19
30. [NLP] 잠재 의미 분석 (LSA) - heehehe.log - 티스토리, 8월 17, 2025에 액세스, https://heehehe-ds.tistory.com/98
31. PCA(주성분분석, Principal Component Analysis)의 수학적 원리 - 통계학, 그리고 인공지능(R, python) - 티스토리, 8월 17, 2025에 액세스, https://rython.tistory.com/18
32. Principal component analysis - Wikipedia, 8월 17, 2025에 액세스, https://en.wikipedia.org/wiki/Principal_component_analysis
33. Mathematical understanding of Principal Component Analysis | by Yuki Shizuya | Intuition, 8월 17, 2025에 액세스, https://medium.com/intuition/mathematical-understanding-of-principal-component-analysis-6c761004c2f8
34. Chapter 5 Singular value decomposition and principal component analysis - CMU School of Computer Science, 8월 17, 2025에 액세스, https://www.cs.cmu.edu/~tom/10701_sp11/slides/pca_wall.pdf
35. dimensionality reduction - Relationship between SVD and PCA ..., 8월 17, 2025에 액세스, https://stats.stackexchange.com/questions/134282/relationship-between-svd-and-pca-how-to-use-svd-to-perform-pca
36. Equivalence of Pseudo-Inverse Matrix calculation and SVD ..., 8월 17, 2025에 액세스, https://2020machinelearning.medium.com/equivalence-of-pseudo-inverse-matrix-calculation-and-svd-singular-value-decomposition-62711ff6e91b
37. eigen decomposition과 함께 PCA, SVD 이해하기 - happip-jh - 티스토리, 8월 17, 2025에 액세스, https://happip-jh.tistory.com/20
38. Principal Component Analysis and Its Derivation From Singular Value Decomposition - Canadian Center of Science and Education, 8월 17, 2025에 액세스, https://ccsenet.org/journal/index.php/ijsp/article/download/0/0/38533/39149
39. Singular Value Decomposition: Applications to Image Processing, 8월 17, 2025에 액세스, https://www.lagrange.edu/academics/undergraduate/undergraduate-research/_images/18-Citations2020.Compton.pdf
40. Singular Value Decomposition applied to Images + Streamlit App | by Guillermo Velazquez | MCD-UNISON | Medium, 8월 17, 2025에 액세스, https://medium.com/mcd-unison/singular-value-decomposition-applied-to-images-streamlit-app-656e2467a25e
41. SVD-Demo: Image Compression - Tim Baumann, 8월 17, 2025에 액세스, https://timbaumann.info/svd-image-compression-demo/
42. [SAS 프로그래밍] SVD를 이용한 이미지 압축(Image compression), 8월 17, 2025에 액세스, [https://communities.sas.com/t5/SAS-Tech-Tip/SAS-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-SVD%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EC%9D%B4%EB%AF%B8%EC%A7%80-%EC%95%95%EC%B6%95-Image-compression/ta-p/662912](https://communities.sas.com/t5/SAS-Tech-Tip/SAS-프로그래밍-SVD를-이용한-이미지-압축-Image-compression/ta-p/662912)
43. [SAS 프로그래밍] SVD를 이용한 이미지 노이즈 제거(denoising an image), 8월 17, 2025에 액세스, [https://communities.sas.com/t5/SAS-Tech-Tip/SAS-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-SVD%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EC%9D%B4%EB%AF%B8%EC%A7%80-%EB%85%B8%EC%9D%B4%EC%A6%88-%EC%A0%9C%EA%B1%B0-denoising-an-image/ta-p/662968](https://communities.sas.com/t5/SAS-Tech-Tip/SAS-프로그래밍-SVD를-이용한-이미지-노이즈-제거-denoising-an-image/ta-p/662968)
44. [Computer Vision] SVD(Singular Value Decomposition)에 대하여 | byeol3325 Blog, 8월 17, 2025에 액세스, https://byeol3325.github.io/studying/SVD/
45. Singular Value Decomposition (SVD) - GeeksforGeeks, 8월 17, 2025에 액세스, https://www.geeksforgeeks.org/machine-learning/singular-value-decomposition-svd/
46. 추천 시스템 기본 - 협업 필터링(Collaborative Filtering) - ① - KM-Hana, 8월 17, 2025에 액세스, https://kmhana.tistory.com/31
47. Build a Recommendation Engine With Collaborative Filtering - Real Python, 8월 17, 2025에 액세스, https://realpython.com/build-recommendation-engine-collaborative-filtering/
48. 추천 시스템: 협업 필터링 (Collaborative filtering), 8월 17, 2025에 액세스, https://prierkt.github.io/machine-learning/Collaborative-filtering/
49. 추천 시스템 2 - Matrix Factorization(MF)를 이용한 협력 필터링 | by 김희규, 8월 17, 2025에 액세스, [https://heegyukim.medium.com/%EC%B6%94%EC%B2%9C-%EC%8B%9C%EC%8A%A4%ED%85%9C-2-matrix-factorization-mf-%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%ED%98%91%EB%A0%A5-%ED%95%84%ED%84%B0%EB%A7%81-11c10199a593](https://heegyukim.medium.com/추천-시스템-2-matrix-factorization-mf-를-이용한-협력-필터링-11c10199a593)
50. 추천 시스템 3장 : 모델 기반 협업 필터링 - velog, 8월 17, 2025에 액세스, [https://velog.io/@hwanii_00/%EC%B6%94%EC%B2%9C-%EC%8B%9C%EC%8A%A4%ED%85%9C-3%EC%9E%A5-%EB%AA%A8%EB%8D%B8-%EA%B8%B0%EB%B0%98-%ED%98%91%EC%97%85-%ED%95%84%ED%84%B0%EB%A7%81](https://velog.io/@hwanii_00/추천-시스템-3장-모델-기반-협업-필터링)
51. 3. 협업필터링 기반 추천시스템 - SGD - TEAM EDA - 티스토리, 8월 17, 2025에 액세스, https://eda-ai-lab.tistory.com/528
52. Collaborative filtering Recommender System with Python – from scratch, using SVD++, item-based, model-based approaches - Alpha Quantum, 8월 17, 2025에 액세스, https://www.alpha-quantum.com/blog/collaborative-filtering-recommender-systems/collaborative-filtering-recommender-system-with-python-from-scratch-using-svd-item-based-model-based-approaches/
53. NLP - 9. 토픽 모델링: 잠재 의미 분석(LSA) - 귀퉁이 서재 - 티스토리, 8월 17, 2025에 액세스, [https://bkshin.tistory.com/entry/NLP-9-%EC%BD%94%EC%82%AC%EC%9D%B8-%EC%9C%A0%EC%82%AC%EB%8F%84%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-%EC%98%81%ED%99%94-%EC%B6%94%EC%B2%9C-%EC%8B%9C%EC%8A%A4%ED%85%9C](https://bkshin.tistory.com/entry/NLP-9-코사인-유사도를-활용한-영화-추천-시스템)
54. 잠재 의미 분석 - 위키백과, 우리 모두의 백과사전, 8월 17, 2025에 액세스, [https://ko.wikipedia.org/wiki/%EC%9E%A0%EC%9E%AC_%EC%9D%98%EB%AF%B8_%EB%B6%84%EC%84%9D](https://ko.wikipedia.org/wiki/잠재_의미_분석)
55. 잠재 의미 분석(LSA)이란 무엇인가요? - Ranktracker, 8월 17, 2025에 액세스, https://www.ranktracker.com/ko/seo/glossary/latent-semantic-analysis-lsa/
56. TF-IDF 행렬의 특이값 분해를 통한 LSA(Latent Semantic Analysis)의 구현과 빈도 기반 토픽 모델의 한계, 8월 17, 2025에 액세스, https://songseungwon.tistory.com/135
57. 의사역행렬과 선형회귀 - 결국 빛나리라, 8월 17, 2025에 액세스, https://westshine-data-analysis.tistory.com/127
58. 다크 프로그래머 :: [선형대수학 #5] 선형연립방정식 풀이 - 티스토리, 8월 17, 2025에 액세스, https://darkpgmr.tistory.com/108
59. 의사역행렬의 기하학적 의미 - 공돌이의 수학정리노트 (Angelo's Math Notes), 8월 17, 2025에 액세스, https://angeloyeo.github.io/2020/11/11/pseudo_inverse.html
60. 유사 역행렬 (Pseudo Inverse Matrix) - DeepCampus - 티스토리, 8월 17, 2025에 액세스, https://pasus.tistory.com/31
61. 파이썬으로 SVD를 계산하는 방법 - 네피리티, 8월 17, 2025에 액세스, https://www.nepirity.com/blog/singular-value-decomposition-for-machine-learning/
62. Pseudo-inverse | Real Statistics Using Excel, 8월 17, 2025에 액세스, https://real-statistics.com/linear-algebra-matrix-topics/pseudo-inverse/
63. The Power of Randomized SVD. Randomized SVD is a method for ..., 8월 17, 2025에 액세스, https://medium.com/@adam.nik/the-power-of-randomized-svd-254cb0082e9e
64. [논문 리뷰] Algorithm xxx: Faster Randomized SVD with Dynamic Shifts, 8월 17, 2025에 액세스, https://www.themoonlight.io/ko/review/algorithm-xxx-faster-randomized-svd-with-dynamic-shifts
