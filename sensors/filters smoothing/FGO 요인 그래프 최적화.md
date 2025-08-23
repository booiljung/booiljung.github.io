# 요인 그래프 최적화(FGO)

본 보고서는 로봇 공학 및 컴퓨터 비전 분야에서 상태 추정 문제의 핵심적인 해결책으로 자리 잡은 요인 그래프 최적화(Factor Graph Optimization, FGO) 또는 스무딩(Smoothing)에 대한 심층적인 기술적 분석을 제공한다. 이 보고서는 요인 그래프의 확률적 기반부터 시작하여, 최대 사후 확률(MAP) 추정이 비선형 최소제곱 문제로 변환되는 과정, 이를 해결하기 위한 최적화 알고리즘, 그리고 대규모 문제의 계산 효율성을 높이는 희소성 활용 기법에 이르기까지 FGO의 전 과정을 체계적으로 다룬다. 또한, 전통적인 필터링 기법과의 근본적인 차이점을 비교하고, 동시적 위치 추정 및 지도 작성(SLAM)과 번들 조정(Bundle Adjustment)과 같은 핵심 응용 분야를 심층적으로 분석하며, 실제 구현 시 고려해야 할 사항과 최신 연구 동향을 조망한다.


요인 그래프 최적화를 이해하기 위해서는 먼저 그 근간을 이루는 확률적 그래픽 모델(Probabilistic Graphical Models, PGM)의 맥락을 파악해야 한다. 베이즈 네트워크(Bayesian Networks)나 마르코프 임의장(Markov Random Fields)과 같은 다른 PGM들 사이에서, 요인 그래프는 특히 복잡한 추정 문제의 구조를 명시적이고 유연하게 표현하는 능력 덕분에 로봇 공학 분야에서 강력한 도구로 부상했다.1 이는 문제의 구조를 직관적으로 시각화할 뿐만 아니라, 계산적으로 효율적인 해법을 설계하는 데 직접적인 통찰을 제공하기 때문이다.3


요인 그래프는 이분 그래프(bipartite graph)로, 두 종류의 노드로 구성된다: **변수 노드(variable nodes)**와 **요인 노드(factor nodes)**이다.4

- **변수 노드 ($X$)**: 우리가 추정하고자 하는 미지의 확률 변수들을 나타낸다. 로봇 공학의 맥락에서 이는 로봇의 시점별 자세(pose), 환경 내 랜드마크의 위치, 센서의 편향(bias), 속도 등과 같은 값들이다.5

- **요인 노드 ($F$)**: 변수들 간의 확률적 제약 또는 관계를 나타낸다. 각 요인 노드 $f_i(X_i)$는 센서 측정값이나 사전 지식(prior belief)으로부터 파생된 정보를 인코딩한다.5 요인 그래프의 구조적 특징은 간선(

  $E$)이 항상 변수 노드와 요인 노드 사이에만 존재한다는 점이다.4


요인 그래프의 근본적인 목적은 다변수 전역 함수(global, multivariate function)의 인수분해(factorization)를 표현하는 것이다. 확률적 추정 문제에서 이 전역 함수는 모든 변수들의 결합 확률 분포(joint probability distribution)에 해당한다.4

이 관계는 수학적으로 다음과 같이 표현된다. 모든 변수 $X$의 결합 확률 $P(X)$는 그래프 내 모든 요인들의 곱에 비례한다:
$$
P(X) \propto \prod_{i} f_i(X_i)
$$
여기서 $X_i$는 요인 $f_i$에 연결된 변수들의 부분 집합이다.4 이 인수분해는 하나의 거대하고 복잡한 고차원 문제를 다루기 쉬운 여러 개의 국소적인 관계들의 집합으로 분해하는 핵심적인 역할을 한다.

이러한 표현 방식의 힘은 단순히 문제를 시각화하는 데 그치지 않는다. 이는 조건부 독립성(conditional independence)을 명시적으로 드러내어 문제의 기저에 깔린 구조, 특히 희소성(sparsity)을 즉각적으로 파악할 수 있게 한다. 이는 상태 전반의 관계가 조밀한 공분산 행렬 내에 암묵적으로 포함되는 칼만 필터와 같은 전통적인 상태 공간 모델과의 근본적인 차이점이다. 예를 들어, 로봇의 주행 기록계(odometry) 측정값은 오직 두 개의 연속된 로봇 자세에만 직접적인 제약을 가하며, 특정 랜드마크 관측값은 현재의 로봇 자세와 해당 랜드마크 변수에만 연결된다.5 이러한 명시적이고 국소적인 연결성은 문제의 수학적 구조로 직접 이어진다. 따라서 요인 그래프를 모델링 도구로 선택하는 것은 대규모 문제를 효율적으로 해결하기 위한 첫걸음이며, 계산 솔버가 직접 활용할 수 있는 언어로 문제를 기술하는 과정이다.1


요인 그래프 최적화의 주된 목표는 이 결합 확률을 최대화하는 변수들의 집합 $X^*$를 찾는 것이다. 이를 최대 사후 확률(Maximum a Posteriori, MAP) 추정이라고 한다.4

MAP 추정 문제는 다음과 같이 공식화된다:
$$
X^* = \underset{X}{\operatorname{argmax}} P(X) = \underset{X}{\operatorname{argmax}} \prod_{i} f_i(X_i)
$$
이는 주어진 모든 정보(측정값과 사전 지식)를 종합적으로 가장 잘 설명하는 변수 값들의 집합을 찾는 것을 의미한다.4


확률적 추정 문제인 MAP 추정을 어떻게 결정론적 최적화 문제로 변환할 수 있을까? 이 전환의 핵심 열쇠는 측정 잡음(noise)에 대한 가정에 있다.


로봇 공학 및 센서 융합과 관련된 많은 실제 문제에서 센서 측정에 수반되는 잡음은 가우시안 분포(Gaussian distribution)로 상당히 잘 모델링된다.13 이 가정은 확률적 추론의 세계와 결정론적 최적화의 세계를 잇는 강력한 다리 역할을 한다.

이 가정 하에서, 각 요인 $f_i(X_i)$는 측정값 $z_i$와, 상태 변수가 주어졌을 때 측정값을 예측하는 비선형 측정 함수 $h_i(X_i)$에 기반한 가우시안 우도 함수(likelihood function)로 표현될 수 있다. 오차(error) 또는 잔차(residual)는 예측과 실제 측정값의 차이, 즉 $e_i(X_i) = h_i(X_i) - z_i$로 정의된다.

따라서 각 요인은 가우시안 확률 밀도 함수(PDF)에 비례하는 형태를 띤다:
$$
f_i(X_i) \propto \exp\left(-\frac{1}{2} \|h_i(X_i) - z_i\|_{\Sigma_i}^2\right)
$$
여기서 $\Sigma_i$는 측정 잡음의 공분산 행렬(covariance matrix)이며, $\|e\|_{\Sigma_i}^2 = e^T \Sigma_i^{-1} e$는 마할라노비스 거리(Mahalanobis distance)의 제곱으로, 측정의 불확실성을 고려하여 오차의 크기를 정규화한다.11 공분산의 역행렬인 $\Omega_i = \Sigma_i^{-1}$는 정보 행렬(information matrix)이라고 불리며, 측정값이 얼마나 확실한지에 대한 정보를 담고 있다.8


이 가우시안 요인을 MAP 추정 공식에 대입하면 비선형 최소제곱 문제로의 전환이 명확해진다.

1. MAP 추정 문제에서 시작:
   $$
   X^* = \underset{X}{\operatorname{argmax}} \prod_{i} f_i(X_i)
   $$
   가우시안 요인 대입:
   $$
   X^* = \underset{X}{\operatorname{argmax}} \prod_{i} \exp\left(-\frac{1}{2} \|h_i(X_i) - z_i\|_{\Sigma_i}^2\right)
   $$
   
2. **로그 변환**: 지수 함수의 곱은 다루기 어렵다. 하지만 로그 함수는 단조 증가 함수이므로, 어떤 함수를 최대화하는 것은 그 함수의 로그를 최대화하는 것과 동일하다. 이 성질을 이용해 곱을 합으로 변환한다:
   $$
   X^* = \underset{X}{\operatorname{argmax}} \log\left(\prod_{i} \exp\left(-\frac{1}{2} \|h_i(X_i) - z_i\|_{\Sigma_i}^2\right)\right) \\
   = \underset{X}{\operatorname{argmax}} \sum_{i} \left(-\frac{1}{2} \|h_i(X_i) - z_i\|_{\Sigma_i}^2\right)
   $$
   **최대화에서 최소화로 전환**: 음수 항의 합을 최대화하는 것은 해당 항의 양수 값의 합을 최소화하는 것과 같다. 따라서 $\operatorname{argmax}$를 $\operatorname{argmin}$으로 바꾸고 음수 부호와 상수 $-1/2$를 제거할 수 있다.

3. **비선형 최소제곱 문제 공식화**: 최종적으로 다음과 같은 비선형 최소제곱(Non-Linear Least Squares, NLLS) 문제의 표준 형태를 얻는다.11
   $$
   X^* = \underset{X}{\operatorname{argmin}} \sum_{i} \|h_i(X_i) - z_i\|_{\Sigma_i}^2
   $$
   이 변환은 매우 강력한 개념적 연결고리를 제공한다. "가장 가능성 있는" 상태를 찾는 확률적 문제를, 비용 함수를 최소화하는 해를 찾는 수치 최적화의 잘 정립된 방법론을 사용하여 풀 수 있게 해주기 때문이다. 요인 그래프는 *무엇을* 최적화할지(문제의 구조)를 정의하고, NLLS 공식은 *어떻게* 최적화할지(수학적 목표)를 알려준다. 따라서 FGO는 SLAM, 번들 조정 등 광범위한 로봇 공학 추정 문제를 우리가 효율적으로 풀 수 있는 표준 최적화 형식으로 체계적으로 변환하는 방법을 제공한다.11


실제 환경에서는 잘못된 데이터 연관 등으로 인해 가우시안 잡음 가정이 깨지는 이상치(outlier)가 발생할 수 있다. 이러한 이상치가 전체 추정치를 망가뜨리는 것을 방지하기 위해, 제곱 오차 비용 $e^T \Sigma^{-1} e$를 강인한 비용 함수(robust cost function) $\rho(e^T \Sigma^{-1} e)$로 대체할 수 있다. 후버(Huber)나 튜키(Tukey)와 같은 강인한 비용 함수는 오차가 특정 임계값을 초과할 때 그 영향을 줄여, 이상치에 덜 민감한 추정 결과를 얻게 해준다.5


NLLS 문제의 오차 함수 $e_i(X)$는 일반적으로 변수 $X$에 대해 비선형이기 때문에, 해를 한 번에 직접 계산할 수 없다. 대신, 초기 추정치 $X_0$에서 시작하여 최적해에 점진적으로 다가가는 반복적인(iterative) 방법을 사용해야 한다.17


모든 반복적인 비선형 최적화 기법의 핵심은 현재 추정치 주변에서 비선형 문제를 선형 문제로 근사하는 것이다. 이는 1차 테일러 전개(Taylor expansion)를 통해 이루어진다. 현재 추정치 $X_k$에서 약간의 변화량 $\Delta X$가 더해졌을 때의 오차 함수는 다음과 같이 근사할 수 있다:
$$
e_i(X_k \oplus \Delta X) \approx e_i(X_k) + J_i(X_k) \Delta X
$$
여기서 $J_i = \frac{\partial e_i}{\partial X}|_{X_k}$는 오차 함수 $e_i$를 해당 요인에 연결된 변수 $X$에 대해 편미분한 **자코비안 행렬(Jacobian matrix)**이다. 자코비안은 비선형 함수의 국소적인 선형 근사를 나타낸다.9 여기서 

$\oplus$ 연산자는 변수가 정의된 공간에 적합한 덧셈 연산을 의미하며, 유클리드 공간에서는 벡터 덧셈이지만 회전과 같은 비유클리드 공간(매니폴드)에서는 더 복잡한 연산(retraction)이 필요하다.11


선형화된 오차 함수를 NLLS 비용 함수에 대입하면, 이제 $\Delta X$에 대한 2차 함수(quadratic function)가 된다. 2차 함수는 그 최솟값을 직접 찾을 수 있다.
$$
\Delta X^* = \underset{\Delta X}{\operatorname{argmin}} \sum_{i} \|e_i(X_k) + J_i(X_k) \Delta X\|_{\Sigma_i}^2
$$
이 선형 최소제곱 문제를 풀면 다음과 같은 **정규 방정식(normal equations)**을 얻게 된다:
$$
(J^T \Omega J) \Delta X = -J^T \Omega e
$$
여기서 $J$, $\Omega$, $e$는 모든 요인들의 자코비안, 정보 행렬, 오차 벡터를 각각 쌓아 올린 거대한 행렬과 벡터이다. $H = J^T \Omega J$ 항은 NLLS 비용 함수의 헤시안 행렬(Hessian matrix)에 대한 근사이며, 항상 양의 준정부호(positive semi-definite) 행렬이다.9 이 선형 시스템을 풀어 최적의 업데이트 스텝  $\Delta X$를 구한 뒤, $X_{k+1} = X_k \oplus \Delta X$로 다음 추정치를 계산한다.

가우스-뉴턴 방법은 최적해 근처에서 매우 빠르게 수렴할 수 있지만, 초기 추정치가 좋지 않거나 문제의 비선형성이 매우 클 경우 스텝이 너무 커져 발산(diverge)할 위험이 있다.5


레벤버그-마르쿼트(Levenberg-Marquardt, LM) 알고리즘은 가우스-뉴턴의 불안정성을 개선하기 위해 고안된 기법이다. 이는 감쇠 인자(damping parameter) $\lambda$를 도입하여 가우스-뉴턴 방법과 경사 하강법(gradient descent)을 지능적으로 혼합한다.

수정된 정규 방정식은 다음과 같다:
$$
(J^T \Omega J + \lambda I) \Delta X = -J^T \Omega e
$$
여기서 $I$는 단위 행렬이다. (마르쿼트가 제안한 방식은 단위 행렬 $I$ 대신 $J^T \Omega J$의 대각 행렬 $\text{diag}(J^T \Omega J)$를 사용하기도 한다).22

이 감쇠 항은 최적화 과정에서 "신뢰 영역(trust region)"의 크기를 조절하는 역할을 한다.

- **$\lambda$가 작을 때**: $\lambda I$ 항의 영향이 미미해져 LM은 가우스-뉴턴과 거의 동일하게 동작한다. 이는 현재의 선형 근사를 "신뢰"하고, 빠르고 큰 스텝을 내딛는 것을 의미한다.
- **$\lambda$가 클 때**: $J^T \Omega J$에 비해 $\lambda I$ 항이 지배적이게 된다. 이 경우 업데이트 스텝 $\Delta X$는 비용 함수를 가장 가파르게 감소시키는 방향(steepest descent)으로의 작은 스텝에 가까워진다. 이는 현재의 선형 근사를 "불신"하고, 발산 위험이 있더라도 비용을 확실히 줄일 수 있는 안전한 방향으로 조금만 움직이는 것을 의미한다.22

LM 알고리즘은 각 반복 단계에서 계산된 스텝이 실제로 비용을 줄였는지 확인한다. 만약 비용이 감소했다면, 선형 근사가 유효했다고 판단하고 $\lambda$를 줄여 다음 스텝에서 더 가우스-뉴턴에 가깝게 만든다. 만약 비용이 증가했다면, 스텝을 기각하고 $\lambda$를 늘려 다음 스텝에서 더 경사 하강법에 가깝게 만들어 안정성을 확보한다.23 이 지능적인 적응 전략 덕분에 LM은 좋지 않은 초기 추정치에서도 훨씬 안정적으로 수렴하며, Ceres, g2o, GTSAM과 같은 대부분의 최적화 라이브러리에서 표준 솔버로 채택되고 있다.11

| 속성                | 가우스-뉴턴 (Gauss-Newton)                       | 레벤버그-마르쿼트 (Levenberg-Marquardt)                      |
| ------------------- | ------------------------------------------------ | ------------------------------------------------------------ |
| **업데이트 방정식** | $(J^T \Omega J) \Delta X = -J^T \Omega e$        | $(J^T \Omega J + \lambda I) \Delta X = -J^T \Omega e$        |
| **핵심 아이디어**   | 반복적인 선형 근사                               | 감쇠를 이용한 선형 근사, 가우스-뉴턴과 경사 하강법의 적응적 혼합 |
| **장점**            | 최적해 근처에서 2차 수렴 속도로 매우 빠름        | 나쁜 초기값에서도 강인한 수렴성, 높은 비선형성 문제 처리 능력 |
| **단점**            | 초기값이 나쁘거나 비선형성이 크면 발산할 수 있음 | 최적해 근처에서 가우스-뉴턴보다 느릴 수 있음, 감쇠 인자 $\lambda$ 조정 필요 |


SLAM이나 번들 조정과 같은 대규모 문제에서는 변수의 수가 수천에서 수백만에 이를 수 있다. 만약 정보 행렬 $H$가 조밀(dense)하다면, 정규 방정식 $H \Delta X = b$를 푸는 것은 계산적으로 불가능에 가깝다.26 다행히도, 이러한 문제들은 본질적으로 **희소(sparse)**하다는 특성을 가진다.


희소성은 센서 측정의 물리적 한계, 즉 국소성(locality)에서 비롯된다. 요인 그래프에서 하나의 요인(측정)은 전체 변수 중 극히 일부에만 연결된다.29

- **SLAM 예시**: 로봇의 주행기록계 측정값은 연속된 두 자세 $x_t$와 $x_{t+1}$만을 연결한다. 랜드마크 관측값은 현재 자세 $x_t$와 특정 랜드마크 $l_j$만을 연결한다. 이 측정값들은 $x_{t-100}$이나 다른 랜드마크 $l_{k \neq j}$와는 직접적인 관계가 없다.8

이러한 물리적 국소성은 요인 그래프 모델에 완벽하게 포착된다.3 그리고 이 그래프의 구조적 희소성은 자코비안 

$J$와 정보 행렬 $H$의 행렬 구조에 그대로 반영된다. 자코비안 행렬 $J$에서 한 요인에 해당하는 행은 그 요인에 연결된 변수들의 열에만 0이 아닌 블록을 가지며, 나머지 열은 모두 0이다.16 정보 행렬 

$H = J^T J$ 또한 희소 행렬이 된다. $H$의 $ij$번째 원소가 0이 아니려면, 변수 $i$와 $j$가 적어도 하나의 요인에 함께 연결되어 있어야 한다.30 이 희소 구조야말로 대규모 최적화를 가능하게 하는 핵심 열쇠이다.1


희소 선형 시스템 $H \Delta X = b$를 푸는 주된 방법은 **희소 촐레스키 분해(sparse Cholesky factorization)**를 이용하는 것이다. 정보 행렬 $H$는 대칭 양의 정부호 행렬이므로 $H = R^T R$ (또는 $L L^T$) 형태로 분해될 수 있으며, 여기서 $R$은 상삼각행렬(upper-triangular matrix)이다.16 해 $\Delta X$는 $R^T y = b$를 풀어 $y$를 구하고(전진 대입), $R \Delta X = y$를 풀어(후진 대입) 효율적으로 찾을 수 있다.

하지만 촐레스키 분해 과정에서 원래 0이었던 행렬의 원소가 0이 아닌 값으로 채워지는 **필인(fill-in)** 현상이 발생할 수 있다.16 필인이 많이 발생하면 분해된 행렬 $R$의 희소성이 파괴되어 계산 효율성이 저하된다. 놀랍게도, 필인의 양은 행렬 $H$의 행과 열의 순서, 즉 변수들의 순서에 매우 민감하게 반응한다. 따라서 분해에 앞서 필인을 최소화하는 방향으로 변수 순서를 재배열하는 **변수 순서화(variable ordering)** 과정이 필수적이다. 최적의 순서를 찾는 것은 NP-난해 문제이지만, COLAMD(Column Approximate Minimum Degree)나 METIS와 같은 휴리스틱 알고리즘들이 실제 문제에서 매우 효과적으로 동작한다.16

결론적으로, 대규모 FGO의 실현 가능성은 다음과 같은 인과 사슬에 의해 결정된다: **측정의 물리적 국소성 -->> 희소한 요인 그래프 구조 -->> 희소한 정보 행렬 -->> 희소 선형 대수 기법을 통한 효율적인 해법 -->> 대규모 SLAM/BA의 실현**. 이 사슬의 어느 한 고리라도 없었다면, 오늘날의 FGO 기반 방법들은 실용적이지 않았을 것이다.


번들 조정(BA) 문제에서 정보 행렬 $H$는 카메라와 3D 점 변수에 해당하는 특정 블록 구조를 가진다. 이때 **슈어 보수(Schur complement)** 트릭은 먼저 카메라 변수들만으로 구성된 더 작은 선형 시스템("reduced camera system")을 풀고, 그 결과를 이용해 점 변수들을 계산(후진 대입)하는 대수적 기법이다. 요인 그래프 관점에서 보면, 이는 모든 점 변수들을 먼저 소거(eliminate)한 후 카메라 변수들을 소거하는 특정 변수 소거 순서를 선택하는 것과 동일하다.35


로봇 공학의 상태 추정 문제는 역사적으로 필터링(filtering)과 스무딩(smoothing)이라는 두 가지 주요 패러다임에 의해 지배되어 왔다.


- **핵심 원리**: 시간 $t$까지의 모든 측정값($z_{1:t}$)을 사용하여 오직 **현재 시점의 상태** $x_t$만을 추정한다. 이는 강력한 마르코프 가정, 즉 현재 상태는 오직 직전 상태와 최신 측정값에만 의존한다는 가정에 기반한다.36
- **프로세스**: 재귀적인 단방향 전진 패스(forward pass)로 동작한다. 이전 상태와 동작 모델을 기반으로 현재 상태를 **예측(predict)**하고, 새로운 측정값을 사용하여 예측을 **갱신(update)**하는 두 단계를 반복한다.37
- **선형화**: 확장 칼만 필터(Extended Kalman Filter, EKF)와 같은 비선형 필터에서는 비선형 모델을 각 단계에서 **단 한 번만** 이전 추정치 주변에서 선형화한다. 이 과정에서 발생하는 선형화 오차는 그대로 다음 단계로 전파되며, 나중에 수정될 수 없다.9
- **장단점**: 계산적으로 매우 효율적이며 실시간 추정에 적합하다. 하지만 과거 상태를 소거(marginalize out)하고 과거의 선형화 결정을 재고하지 않기 때문에 정보를 폐기하게 되어 스무딩보다 정확도가 떨어진다.30


- **핵심 원리**: 특정 시간 구간(window) 또는 전체 궤적에 대한 모든 측정값($z_{1:T}$)을 사용하여 해당 구간의 **모든 상태** $x_{1:T}$를 동시에 추정한다.29 이는 배치(batch) 최적화 문제이다.
- **프로세스**: 단방향 전진 패스와 달리, 스무딩은 모든 제약 조건을 동시에 고려한다. 예를 들어 SLAM에서 현재 자세 $x_T$와 아주 오래전의 자세 $x_k$를 연결하는 루프 폐쇄(loop closure) 측정값이 들어오면, 이는 $x_k$와 $x_T$ 사이의 전체 경로에 영향을 미치는 강력한 제약이 된다. 최적화기는 그래프 내의 모든 변수를 조정하여 전역적으로 가장 일관된 해를 찾는다.8
- **재선형화**: FGO는 전체 궤적에 대한 현재의 최적 추정치 주변에서 문제를 **반복적으로 재선형화(re-linearize)**한다. 이는 선형화 오차를 크게 줄여 월등히 높은 정확도를 달성하는 핵심적인 이유다.38

필터링에서 스무딩으로의 전환은 상태 추정을 재귀적, 마르코프적 관점에서 전역적, 배치 최적화 관점으로 바라보는 패러다임의 전환이다. 스무딩은 본질적으로 더 정확하지만 계산 비용이 높다. 따라서 실용적인 스무딩 기법들은 배치 최적화의 정확도를 최대한 유지하면서 필터링의 실시간성을 되찾기 위한 영리한 공학적 해법들이다.

| 속성               | 필터링 (칼만 필터)            | 스무딩 (FGO)                   |
| ------------------ | ----------------------------- | ------------------------------ |
| **사용 데이터**    | 과거부터 현재까지 ($z_{1:t}$) | 특정 구간 전체 ($z_{1:T}$)     |
| **추정 변수**      | 현재 상태 $x_t$               | 전체 궤적 $x_{1:T}$            |
| **최적성**         | 선형-가우시안 시스템에서 최적 | 주어진 배치 데이터에 대해 최적 |
| **선형화**         | 각 단계에서 단일 선형화       | 반복적인 재선형화              |
| **계산 비용**      | 낮음 (스텝당 거의 일정)       | 높음 (궤적 길이에 따라 증가)   |
| **루프 폐쇄 처리** | 부정확, 과거 수정 불가        | 최적, 전역적 오차 보정         |


- **전체 스무딩 (Full Smoothing)**: 전체 궤적에 걸쳐 배치 최적화를 수행한다. 가장 정확하지만, 시스템이 계속 동작함에 따라 그래프가 무한정 커져 계산적으로 불가능해진다.29

- **고정 지연 스무딩 (Fixed-Lag Smoothing)**: 실시간 적용을 위한 현실적인 타협안으로, '슬라이딩 윈도우 최적화'라고도 불린다. 가장 최근 $N$개의 상태로 구성된 윈도우 내에서만 최적화를 수행한다. 윈도우보다 오래된 상태는 고정되거나 소거된다.7 이는 계산 비용을 한정시키지만, 윈도우 크기 

  $N$의 선택이 정확도와 효율성 사이의 트레이드오프를 결정한다.40

- **증분 스무딩 (Incremental Smoothing)**: iSAM, iSAM2와 같은 더 진보된 기법이다. 전체 그래프를 유지하되, 새로운 측정값이 들어올 때마다 영향을 받는 부분만 지능적으로 업데이트하여 처음부터 전체를 재계산하는 것을 피한다. 이를 위해 베이즈 트리(Bayes Tree)라는 자료 구조를 사용하여 영향을 받는 변수를 효율적으로 식별하고 이전 계산 결과를 재사용한다.3 이는 전체 스무딩에 가까운 정확도를 고정 지연 스무딩에 가까운 계산 비용으로 제공하는 매우 효과적인 방법이다.


FGO는 그 유연성과 정확성 덕분에 로봇 공학과 컴퓨터 비전의 여러 핵심 문제에 적용되고 있다.


SLAM은 로봇이 미지의 환경을 탐험하면서 지도를 작성하고, 동시에 그 지도 내에서 자신의 위치를 추적하는 문제이다.5 FGO는 SLAM 문제의 "백엔드(back-end)", 즉 센서 데이터로부터 추출된 제약 조건들을 바탕으로 최적의 궤적과 지도를 계산하는 역할을 한다.

- **그래프 구조**:
  - **변수 노드**: 로봇의 시간대별 자세 $x_1, x_2,..., x_T$와 환경 내 랜드마크의 위치 $l_1, l_2,..., l_M$.5
  - **요인 노드**:
    - **사전 요인(Prior Factor)**: 첫 번째 자세 $x_1$에 대한 사전 정보를 제공하여 전체 지도의 기준점을 고정시키는 단항(unary) 요인.46
    - **주행기록계 요인(Odometry Factor)**: 바퀴 엔코더나 IMU로부터 얻은 이동 정보를 나타내며, 연속된 두 자세($x_t, x_{t+1}$)를 연결하는 이진(binary) 요인.8
    - **랜드마크 관측 요인(Landmark Observation Factor)**: 카메라나 LiDAR 같은 센서가 특정 랜드마크를 관측했을 때의 정보를 나타내며, 자세 $x_t$와 랜드마크 $l_j$를 연결하는 요인.5
    - **루프 폐쇄 요인(Loop Closure Factor)**: 로봇이 이전에 방문했던 장소를 다시 인식했을 때 생성되는 요인. 현재 자세 $x_T$와 과거의 자세 $x_k$를 직접 연결하여, 그동안 누적된 오차(drift)를 전역적으로 보정하는 매우 중요한 제약 조건이다.8

FGO 솔버는 이 모든 제약 조건들을 동시에 가장 잘 만족시키는 로봇 자세들과 랜드마크 위치들의 집합을 찾는다. 이는 종종 랜드마크를 소거한 후 자세들만 최적화하는 "포즈 그래프 최적화(pose-graph optimization)" 형태로 수행되기도 한다.47


번들 조정(BA)은 컴퓨터 비전의 다중 시점 3차원 복원(Structure from Motion, SfM) 파이프라인의 핵심 최적화 단계이다. 이는 3차원 구조(점 구름)와 시점 파라미터(카메라 자세 및 내부 파라미터)를 **동시에** 정밀하게 조정하여, 실제 2D 이미지에 관측된 특징점과 3D 모델을 투영했을 때 예측되는 2D 점 사이의 재투영 오차(reprojection error)를 최소화하는 과정이다.50

- **SLAM과의 관계**: BA는 본질적으로 SLAM과 동일한 수학적 문제이다.53 특히 시각 SLAM(Visual SLAM)의 백엔드는 사실상 BA 그 자체이다.48 두 문제의 근본적인 차이는 응용 목표와 작동 방식에 있다. 전통적으로 BA는 주어진 이미지 묶음으로부터 최고의 정확도를 가진 3차원 모델을 생성하는 오프라인(offline) 배치 작업으로 간주되었다.54 반면 SLAM은 로봇의 실시간 항법을 위한 온라인(online) 문제로 발전해왔다.54 하지만 이 구분은 점차 흐려지고 있으며, 현대의 SLAM 시스템들은 정교한 증분 BA 기법을 사용한다.
- **BA의 그래프 구조**:
  - **변수 노드**: 카메라 자세 $C_1,..., C_N$와 3차원 점 $P_1,..., P_M$.
  - **요인 노드**: **재투영 오차 요인**. 각 요인은 하나의 카메라 $C_i$와 하나의 3차원 점 $P_j$을 연결하며, 카메라 $i$의 이미지 평면에서 관측된 2차원 특징점 $z_{ij}$에 의해 제약을 받는다. 오차는 실제 관측된 2D 점 $z_{ij}$와, 3D 점 $P_j$를 카메라 $C_i$로 투영하여 예측한 2D 점 사이의 유클리드 거리로 정의된다.50

FGO는 SLAM과 BA라는, 서로 다른 분야에서 발전해 온 두 문제를 하나의 통일된 수학적 프레임워크로 이해하게 해준다. 두 문제의 차이는 핵심 문제가 아니라 실시간성이라는 운영상의 제약에서 비롯되며, 이 제약이 SLAM에서 증분 솔버와 같은 특정 최적화 전략의 발전을 이끌었다.


로봇 공학에서 다루는 많은 변수들은 단순한 유클리드 벡터 공간에 존재하지 않는다. 예를 들어, 3차원 회전은 특수 직교군 SO(3)에 속하고, 3차원 자세(위치+회전)는 특수 유클리드군 SE(3)에 속한다. 이들은 비선형적인 기하학적 구조를 가진 **매니폴드(manifold)**이다.3

단순히 회전 행렬에 업데이트 벡터를 더하는 것은 더 이상 유효한 회전 행렬(즉, 직교 행렬이 아닌)을 만들 가능성이 높다. 이 문제를 원리적으로 해결하기 위해 리 군(Lie group) 이론이 도입된다. 최적화는 현재 추정치에서의 국소적인 접공간(tangent space) 상에서 수행된다. 접공간은 벡터 공간이므로 기존의 선형 대수 기법을 적용할 수 있다. 여기서 계산된 업데이트 벡터 $\Delta X$는 리 대수(Lie algebra, 예: $\mathfrak{so}(3)$)의 원소이다. 이 업데이트는 **지수 사상(exponential map)**을 통해 다시 원래의 매니폴드로 매핑되며, 이 과정을 통해 업데이트된 변수가 항상 매니폴드 상에 있도록(예: 유효한 회전 행렬이 되도록) 보장한다.22 이는 비유클리드 공간의 변수들을 다루는 원칙적이고 정확한 방법이다.9


FGO는 강력한 프레임워크이지만, 실제 시스템에 적용할 때는 여러 장단점과 현실적인 문제들을 고려해야 한다.


- **정확성**: 반복적인 재선형화와 (윈도우 내의) 전체 측정 이력을 활용하므로 필터링 기법보다 일반적으로 더 정확한 결과를 제공한다.38
- **유연성과 확장성**: 새로운 종류의 센서나 제약 조건을 추가하기가 매우 용이하다. 새로운 요인 타입을 정의하고 그래프에 추가하기만 하면 된다. 이러한 "플러그 앤 플레이" 기능은 IMU, GPS, 카메라, LiDAR 등 다중 센서를 융합하는 데 큰 장점이다.9
- **이상치에 대한 강인성**: 강인한 비용 함수와 결합될 때, FGO는 잘못된 루프 폐쇄와 같은 이상치 측정값을 효과적으로 식별하고 거부할 수 있어 장기간 자율 주행에 필수적이다.5
- **비선형성 및 비유클리드 공간 처리**: 매우 비선형적인 모델을 자연스럽게 다루며, SO(3)와 같은 매니폴드 상에서 최적화를 올바르게 수행한다.3


- **계산 복잡도**: 가장 주된 단점이다. 배치 최적화의 비용은 그래프의 크기에 따라 증가한다. 희소성과 증분 기법이 이를 완화하지만, 여전히 필터링보다 계산 집약적이다.26
- **초기값 민감성**: 비볼록(non-convex) 최적화 문제이므로, 전역 최적해(global minimum)로 수렴하기 위해서는 합리적으로 좋은 초기 추정치가 필요하다. 나쁜 초기값은 잘못된 지역 최적해(local minimum)로의 수렴을 야기할 수 있다.5 이것이 SLAM의 프론트엔드가 좋은 초기 추정치를 제공하기 위해 많은 노력을 기울이는 이유이다.
- **파라미터 튜닝**: 레벤버그-마르쿼트의 감쇠 인자 $\lambda$나 고정 지연 스무딩의 윈도우 크기 $N$과 같은 파라미터들은 최적의 성능을 위해 튜닝이 필요할 수 있다.23

이러한 장단점은 FGO 사용이 "공짜 점심"이 아님을 시사한다. FGO의 뛰어난 정확성과 유연성은 계산 복잡도와 좋은 초기값의 필요성이라는 대가를 치른다. FGO 기반 시스템을 설계하는 것은 단순히 라이브러리를 선택하는 것을 넘어, 정확도와 계산량 사이의 트레이드오프를 관리하고(예: 증분 스무딩 사용), 프론트엔드가 백엔드가 성공할 수 있도록 충분히 좋은 초기값을 제공하도록 보장하는 전체적인 시스템 설계의 문제이다.


FGO 생태계를 주도하는 여러 강력한 오픈소스 라이브러리가 존재한다.

- **GTSAM (Georgia Tech Smoothing and Mapping)**: 요인 그래프 개념에 가장 충실한 라이브러리. 명료한 코드, C++ 표현식 템플릿 사용, 매니폴드 최적화에 대한 강력한 지원으로 유명하다. 이 분야의 선구자들(Dellaert, Kaess)에 의해 개발되었다.11
- **Ceres Solver**: 구글에서 개발한 범용 NLLS 라이브러리. 고도로 최적화되어 있으며 구글 맵스와 같은 상용 시스템에서 널리 사용된다. 명시적인 요인 그래프 라이브러리는 아니지만, FGO가 공식화하는 기저의 NLLS 문제를 푸는 데 사용된다.11
- **g2o (General Graph Optimization)**: 로봇 공학 커뮤니티에서 널리 사용되는 C++ 프레임워크로, 특히 ORB-SLAM과 같은 유명한 SLAM 시스템에서 채택되었다. 정점(변수)과 간선(요인)을 정의하는 프레임워크를 제공한다.11


요인 그래프 최적화는 문제를 요인 그래프 상의 확률적 추론으로 모델링하는 강력하고 유연한 상태 추정 프레임워크이다. 일반적인 가우시안 잡음 가정 하에서, 이 확률적 MAP 문제는 결정론적 비선형 최소제곱 최적화 문제와 동일하며, 로봇 공학 문제의 본질적인 희소성은 희소 촐레스키 분해와 같은 기법을 통해 대규모 최적화를 효율적으로 풀 수 있게 한다. 스무딩 접근법은 필터링보다 더 정확한 결과를 제공하며, 증분 및 고정 지연 기법을 통해 실시간성의 한계를 극복하고 있다.

FGO는 이미 성숙한 기술이지만, 연구는 여전히 활발히 진행 중이며 다음과 같은 방향으로 나아가고 있다.

- **강인성과 비가우시안 추론**: 단순한 강인한 비용 함수를 넘어, 가우시안 혼합 모델(GMM) 등을 사용하여 더 복잡하고 비대칭적인 잡음 분포를 다루려는 연구가 진행 중이다.15
- **장기간 자율성과 지도 관리**: 로봇이 수일 또는 수개월 동안 작동하더라도 그래프가 관리 불가능할 정도로 커지지 않도록, 그래프를 지능적으로 희소화하고 요약하는 전략 개발이 중요하다.26
- **인식, 계획, 제어의 통합**: FGO 프레임워크를 상태 추정(인식)뿐만 아니라, 미래 궤적을 최적화하는 동작 계획 및 제어 문제까지 확장하여, 이들을 하나의 통일된 최적화 문제로 풀려는 시도가 이루어지고 있다.4
- **하드웨어 가속**: GPU나 그래프코어의 IPU와 같은 새로운 유형의 프로세서를 활용하여 FGO 솔버를 더욱 가속화하려는 연구가 활발하다.5

요인 그래프 최적화는 로봇과 지능형 시스템이 복잡하고 불확실한 세상과 상호작용하는 방식을 근본적으로 향상시키는 핵심 기술로, 앞으로도 그 중요성은 계속해서 커질 것이다.


1. Factor Graphs for Robot Perception - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/339504457_Factor_Graphs_for_Robot_Perception
2. Factor Graphs for Robot Perception - Now Publishers, accessed July 23, 2025, https://www.nowpublishers.com/article/Details/ROB-043
3. Factor Graphs for Robot Perception - CMU School of Computer Science, accessed July 23, 2025, https://www.cs.cmu.edu/~kaess/pub/Dellaert17fnt.pdf
4. Factor Graphs in Optimization-Based Robotic Control-A Tutorial and Review - ORBilu, accessed July 23, 2025, https://orbilu.uni.lu/bitstream/10993/64744/1/ACCESS3534993.pdf
5. Mastering Factor Graph Optimization - Number Analytics, accessed July 23, 2025, https://www.numberanalytics.com/blog/factor-graph-optimization-cognitive-robotics
6. Welcome to Factor Graphs - Emma Benjaminson, accessed July 23, 2025, https://sassafras13.github.io/FactorGraphs/
7. Optimize factor graph - MATLAB - MathWorks, accessed July 23, 2025, https://www.mathworks.com/help/nav/ref/factorgraph.optimize.html
8. (PDF) A tutorial on graph-based SLAM - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/231575337_A_tutorial_on_graph-based_SLAM
9. Factor Graphs for Navigation Applications: A Tutorial, accessed July 23, 2025, https://navi.ion.org/content/71/3/navi.653
10. Factor graph - Wikipedia, accessed July 23, 2025, https://en.wikipedia.org/wiki/Factor_graph
11. miniSAM: A Flexible Factor Graph Non-linear Least Squares ..., accessed July 23, 2025, https://project.inria.fr/ppniv19/files/2019/11/PPNIV19-paper_Dong.pdf
12. Factor Graphs for Robot Perception - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/319132170_Factor_Graphs_for_Robot_Perception
13. Factor Graphs for Navigation Applications: A Tutorial, accessed July 23, 2025, https://navi.ion.org/content/navi/71/3/navi.653.full.pdf
14. Least Squares Optimization: From Theory to Practice - MDPI, accessed July 23, 2025, https://www.mdpi.com/2218-6581/9/3/51
15. Robust Factor Graph Optimization – A Comparison for Sensor Fusion Applications - TU Chemnitz, accessed July 23, 2025, https://www.tu-chemnitz.de/etit/proaut/publications/etfa16.pdf
16. (PDF) Incremental Block Cholesky Factorization for Nonlinear Least ..., accessed July 23, 2025, https://www.researchgate.net/publication/306000554_Incremental_Block_Cholesky_Factorization_for_Nonlinear_Least_Squares_in_Robotics
17. Non-linear least squares - Wikipedia, accessed July 23, 2025, https://en.wikipedia.org/wiki/Non-linear_least_squares
18. ECE276A: Sensing & Estimation in Robotics Lecture 16: Batch Estimation, Factor Graphs - Nikolay A. Atanasov, accessed July 23, 2025, https://natanaso.github.io/ece276a2019/ref/ECE276A_16_BatchEstimation.pdf
19. Solving Non-linear Least Squares - Ceres Solver, accessed July 23, 2025, http://ceres-solver.org/nnls_solving.html
20. Tutorial - Ceres Solver, accessed July 23, 2025, http://ceres-solver.org/tutorial.html
21. Gradient Based Optimizations: Jacobians, Jababians & Hessians | by Jake Batsuuri | Computronium Blog | Medium, accessed July 23, 2025, https://medium.com/computronium/gradient-based-optimizations-jacobians-jababians-hessians-b7cbe62d662d
22. Gauss-Newton / Levenberg-Marquardt Optimization, accessed July 23, 2025, https://mat.uab.cat/~alseda/MasterOpt/optimization.pdf
23. Mastering Levenberg-Marquardt Algorithm - Number Analytics, accessed July 23, 2025, https://www.numberanalytics.com/blog/levenberg-marquardt-algorithm-guide
24. factor graphs and nonlinear least-squares problems on manifolds, accessed July 23, 2025, [http://wavelab.uwaterloo.ca/slam/2017-SLAM/Lecture6-factor_graphs_and_nlls_on_manifolds/Factor%20Graphs%20and%20NLLS%20on%20Manifolds.pdf](http://wavelab.uwaterloo.ca/slam/2017-SLAM/Lecture6-factor_graphs_and_nlls_on_manifolds/Factor Graphs and NLLS on Manifolds.pdf)
25. gtsam::LevenbergMarquardtOptimizer Class Reference, accessed July 23, 2025, https://gtsam.org/doxygen/a04380.html
26. Nonlinear Graph Sparsification for SLAM - Robotics, accessed July 23, 2025, https://www.roboticsproceedings.org/rss10/p40.pdf
27. Nonlinear Graph Sparsification for SLAM, accessed July 23, 2025, http://ais.informatik.uni-freiburg.de/publications/papers/mazuran14rss.pdf
28. Bundle Adjustment Revisited - arXiv, accessed July 23, 2025, https://arxiv.org/pdf/1912.03858
29. Factor Graphs and GTSAM, accessed July 23, 2025, https://gtsam.org/tutorials/intro.html
30. Graph SLAM - Washington, accessed July 23, 2025, [https://courses.cs.washington.edu/courses/cse571/23sp/slides/L09/Lecture09_Modern%20SLAM.pdf](https://courses.cs.washington.edu/courses/cse571/23sp/slides/L09/Lecture09_Modern SLAM.pdf)
31. Sparse Pose Graph Optimization in Cycle Space - arXiv, accessed July 23, 2025, https://arxiv.org/pdf/2203.15597
32. Factor descent optimization for sparsification in graph SLAM - Digital CSIC, accessed July 23, 2025, https://digital.csic.es/bitstream/10261/168330/1/factorSLAM.pdf
33. Factor Graphs for Robot Perception - Now Publishers, accessed July 23, 2025, https://www.nowpublishers.com/article/DownloadSummary/ROB-043
34. On the Minimum FLOPs Problem in the Sparse Cholesky Factorization - SIAM.org, accessed July 23, 2025, https://epubs.siam.org/doi/10.1137/130912438
35. Mining Structure Fragments for Smart Bundle Adjustment - Frank Dellaert, accessed July 23, 2025, https://dellaert.github.io/files/Carlone14bmvc.pdf
36. 1 Inference in Graphical Models, accessed July 23, 2025, https://dellaert.github.io/20S-3630/notes/1-hmm-inference.pdf
37. Kalman filter - Wikipedia, accessed July 23, 2025, https://en.wikipedia.org/wiki/Kalman_filter
38. A Comparative Study of Factor Graph Optimization-Based and Extended Kalman Filter-Based PPP-B2b/INS Integrated Navigation - MDPI, accessed July 23, 2025, https://www.mdpi.com/2072-4292/15/21/5144
39. Kalman Filtering and Kalman Smoothing in PROC UCM - SAS Support Communities, accessed July 23, 2025, https://communities.sas.com/t5/SAS-Communities-Library/Kalman-Filtering-and-Kalman-Smoothing-in-PROC-UCM/ta-p/338797
40. Factor graph optimization for GNSS/INS integration: A comparison with the extended Kalman filter | Request PDF - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/351802788_Factor_graph_optimization_for_GNSSINS_integration_A_comparison_with_the_extended_Kalman_filter
41. Factor Graph Based Incremental Smoothing in Inertial Navigation Systems - GitHub Pages, accessed July 23, 2025, https://indelman.github.io/ANPL-Website/Publications/Indelman12fusion_ppt.pdf
42. Motion Tracking with Fixed-lag Smoothing: Algorithm and Consistency Analysis - Department of Electrical and Computer Engineering - University of California, Riverside, accessed July 23, 2025, https://intra.ece.ucr.edu/~mourikis/tech_reports/fixed_lag.pdf
43. Chapter 9.3 - Fixed-Lag Smoothing - GlobalSpec, accessed July 23, 2025, https://www.globalspec.com/reference/19861/160210/chapter-9-3-fixed-lag-smoothing
44. Factor Graph Based Incremental Smoothing in Inertial Navigation Systems - Frank Dellaert, accessed July 23, 2025, https://dellaert.github.io/files/Indelman12fusion.pdf
45. A Tutorial on Graph-Based SLAM - Uni Freiburg, accessed July 23, 2025, http://www2.informatik.uni-freiburg.de/~stachnis/pdf/grisetti10titsmag.pdf
46. gtsam/matlab/gtsam_examples/PlanarSLAMExample_graph.m at master - GitHub, accessed July 23, 2025, https://github.com/haidai/gtsam/blob/master/matlab/gtsam_examples/PlanarSLAMExample_graph.m
47. A Tutorial on Graph-Based SLAM (vol 2, pg 31, 2010) - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/292377670_A_Tutorial_on_Graph-Based_SLAM_vol_2_pg_31_2010
48. Visual SLAM: Why Bundle Adjust? - arXiv, accessed July 23, 2025, https://arxiv.org/pdf/1902.03747
49. Difference between motion-only bundle adjustment and pose-graph optimization, accessed July 23, 2025, https://robotics.stackexchange.com/questions/22899/difference-between-motion-only-bundle-adjustment-and-pose-graph-optimization
50. Bundle adjustment - Wikipedia, accessed July 23, 2025, https://en.wikipedia.org/wiki/Bundle_adjustment
51. Bundle Adjustment - A Modern Synthesis - Johns Hopkins Computer Science, accessed July 23, 2025, https://www.cs.jhu.edu/~misha/ReadingSeminar/Papers/Triggs00.pdf
52. Bundle Adjustment and SLAM - Computer Vision and Geometry Group, accessed July 23, 2025, https://cvg.ethz.ch/lectures/3D-vision/assets/class06eth24.pdf
53. Bundle Adjustment (BA) - Steven Gong, accessed July 23, 2025, https://stevengong.co/notes/Bundle-Adjustment
54. Differences between SLAM, SfM, BA? : r/computervision - Reddit, accessed July 23, 2025, https://www.reddit.com/r/computervision/comments/ewlkat/differences_between_slam_sfm_ba/
55. What is the main difference between direct and indirect approaches of SLAM In terms of processing and representation and what is the best way to fuse? - Quora, accessed July 23, 2025, https://www.quora.com/What-is-the-main-difference-between-direct-and-indirect-approaches-of-SLAM-In-terms-of-processing-and-representation-and-what-is-the-best-way-to-fuse
56. Visual SLAM Tutorial: Bundle Adjustment, accessed July 23, 2025, https://www.cs.cmu.edu/~kaess/vslam_cvpr14/media/VSLAM-Tutorial-CVPR14-A13-BundleAdjustment-handout.pdf
57. question:What's the difference between factorGraph and poseGraph/3D? - MATLAB Answers - MathWorks, accessed July 23, 2025, https://www.mathworks.com/matlabcentral/answers/1875132-question-what-s-the-difference-between-factorgraph-and-posegraph-3d
58. Get Started | GTSAM, accessed July 23, 2025, https://gtsam.org/get_started/
59. Tutorials - GTSAM 4.0.2 documentation - Read the Docs, accessed July 23, 2025, https://doc-gtsam.readthedocs.io/en/latest/Tutorials.html
60. GTSAM by Example, accessed July 23, 2025, https://gtbook.github.io/gtsam-examples/
61. Tutorials | GTSAM, accessed July 23, 2025, https://gtsam.org/tutorials/
62. What Is Bundle Adjustment? | Baeldung on Computer Science, accessed July 23, 2025, https://www.baeldung.com/cs/computer-vision-bundle-adjustment
63. Factor Graph-Based Planning as Inference for Autonomous Vehicle Racing - arXiv, accessed July 23, 2025, https://arxiv.org/html/2203.03224v4
64. A Factor Graph Approach to Estimation and Model Predictive Control on Unmanned Aerial Vehicles, accessed July 23, 2025, https://repository.gatech.edu/bitstreams/d907d998-2214-4e01-b006-7d82793db1f0/download
65. Bundle Adjustment on a Graph Processor - CVF Open Access, accessed July 23, 2025, https://openaccess.thecvf.com/content_CVPR_2020/papers/Ortiz_Bundle_Adjustment_on_a_Graph_Processor_CVPR_2020_paper.pdf

