# PPP-AR을 활용한 고정밀 위치 추정
[GNSS](./index.md)


현대 사회는 자율 주행 자동차, 정밀 농업, 지각 변동 감시, 무인 항공기(UAV)를 이용한 지도 제작 등 다양한 분야에서 센티미터, 나아가 밀리미터 수준의 고정밀 위치 정보를 요구하고 있다.1 이러한 수요에 부응하기 위해 위성 항법 시스템(Global Navigation Satellite System, GNSS) 기술은 끊임없이 발전해왔다. 초기 단독 측위(Single Point Positioning, SPP) 기술에서부터 기준국과의 상대적 위치를 계산하여 정밀도를 높이는 실시간 이동 측위(Real-Time Kinematic, RTK) 기술을 거쳐, 단일 수신기만으로 전 지구적 정밀 위치 결정이 가능한 정밀 단독 측위(Precise Point Positioning, PPP) 기술에 이르렀다.4

PPP 기술은 지역적 기준국 인프라 없이도 정밀한 위성 궤도 및 시각 보정 정보를 이용하여 높은 정확도를 달성할 수 있다는 점에서 혁신적이었으나, 치명적인 한계를 내포하고 있었다. 바로 수 분에서 수십 분, 때로는 한 시간 이상 소요되는 긴 수렴 시간(convergence time)이다.7 이러한 긴 수렴 시간은 반송파 위상 관측값에 포함된 미지정수(ambiguity)를 실수(real number) 형태의 '플로트(float)' 해로 추정하는 과정에서 발생하는 필연적인 문제였으며, 동적이고 실시간 응답성이 중요한 응용 분야로의 확산을 가로막는 주된 병목 현상으로 작용했다.9

이러한 한계를 극복하기 위해 등장한 기술이 바로 정수 미지정수 결정 기능이 탑재된 정밀 단독 측위, 즉 PPP-AR(Precise Point Positioning with Ambiguity Resolution)이다.4 PPP-AR은 반송파 위상 미지정수가 본질적으로 정수(integer)라는 물리적 특성을 복원하여 결정함으로써, 기존 PPP의 긴 수렴 시간을 획기적으로 단축하고 위치 결정 정확도를 한 단계 더 높은 수준으로 끌어올렸다.11 이 기술의 등장은 단순히 PPP의 성능을 개선한 것을 넘어, 고정밀 GNSS 기술의 패러다임을 전환시키는 계기가 되었다.

PPP-AR의 성공은 단순히 사용자 측의 알고리즘 개선만으로 이루어진 것이 아니다. 이는 전 지구적 규모의 기술 생태계 구축을 통해 가능해졌다. PPP가 주로 측지학자들을 위한 후처리 기술로 인식되었던 반면, PPP-AR은 동적인 실시간 측위 서비스로의 변모를 이끌었다. 이 변화의 핵심에는 미지정수의 정수성을 훼손하는 위성과 수신기의 하드웨어 바이어스(bias)를 분리하고 보정하는 과정이 있다.9 이러한 바이어스 분리는 단일 수신기만으로는 불가능하며, 국제 GNSS 서비스(International GNSS Service, IGS)와 같은 기관 산하의 여러 분석 센터(Analysis Center, AC)들이 전 세계에 분포된 상시 관측소 네트워크 데이터를 기반으로 정밀한 바이어스 보정 정보를 생성하고 사용자에게 배포하는 협력적 인프라가 필수적이다.12 따라서 PPP-AR의 발전은 알고리즘의 혁신과 더불어, 이러한 글로벌 협력 인프라의 구축이 함께 이루어낸 성과라 할 수 있다. 이로써 전 세계 어디서든 단일 수신기 사용자가 RTK와 유사한 수준의 정확도를 얻을 수 있는 기반이 마련되었으며, 이는 GNSS 기술 지형의 근본적인 변화를 의미한다.

본 보고서는 PPP-AR 기술의 이론적 배경과 수학적 모델부터 핵심 알고리즘, 성능에 영향을 미치는 요인, 타 기술과의 비교, 그리고 다양한 응용 분야에 이르기까지 포괄적이고 심도 있는 분석을 제공하고자 한다. 이를 통해 측지학, 항법 시스템, 그리고 관련 공학 분야의 연구자 및 전문가들에게 PPP-AR 기술에 대한 깊이 있는 이해와 통찰을 제공하는 것을 목표로 한다.


PPP-AR의 수학적 기반을 이해하기 위해서는 GNSS 수신기가 수집하는 원시 관측값(raw observation)에 대한 정밀한 모델링이 선행되어야 한다. 다양한 모델링 방식 중, 모든 원시 관측 정보를 그대로 유지하여 다중 주파수 및 다중 위성군(multi-GNSS) 환경에서 가장 유연하게 적용될 수 있는 비차분 비조합(Undifferenced and Uncombined, UDUC) 모델이 PPP-AR의 근간을 이룬다.14


UDUC 모델은 각 위성-수신기 쌍에 대해 각 주파수별 코드(code)와 반송파 위상(carrier phase) 관측 방정식을 개별적으로 구성한다. 수신기 `r`이 위성 `s`로부터 주파수 `j`의 신호를 수신할 때, 코드 및 반송파 위상 관측 방정식은 다양한 오차 요인을 포함하여 다음과 같이 표현될 수 있다.14

| 관측 방정식                  | 공식                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| 코드 관측값 ($P_{r,j}^s$)    | $P_{r,j}^s = \rho_r^s + c \cdot (dt_r - dt^s) + T_r^s + I_{r,j}^s + d_{r,j} - d_j^s + M_{r,j,P}^s + \epsilon_{r,j,P}^s$ |
| 위상 관측값 ($\Phi_{r,j}^s$) | $\Phi_{r,j}^s = \rho_r^s + c \cdot (dt_r - dt^s) + T_r^s - I_{r,j}^s + \lambda_j (N_{r,j}^s + b_{r,j} - b_j^s) + M_{r,j,\Phi}^s + \epsilon_{r,j,\Phi}^s$ |

위 방정식의 각 항은 다음 표와 같이 정의된다.

| 기호                                        | 설명                                                         |
| ------------------------------------------- | ------------------------------------------------------------ |
| $P_{r,j}^s, \Phi_{r,j}^s$                   | 각각 코드 및 반송파 위상 관측값 (단위: 미터)                 |
| $\rho_r^s$                                  | 위성 안테나 위상 중심과 수신기 안테나 위상 중심 간의 기하학적 거리 |
| $c$                                         | 진공 중 빛의 속도                                            |
| $dt_r, dt^s$                                | 각각 수신기 및 위성의 시계 오차                              |
| $T_r^s$                                     | 대류층(troposphere)에 의한 신호 지연                         |
| $I_{r,j}^s$                                 | 전리층(ionosphere)에 의한 신호 지연 (주파수 `j`에 의존적)    |
| $d_{r,j}, d_j^s$                            | 각각 수신기 및 위성의 코드 하드웨어 바이어스                 |
| $b_{r,j}, b_j^s$                            | 각각 수신기 및 위성의 위상 하드웨어 바이어스 (Phase Bias)    |
| $\lambda_j$                                 | 주파수 `j`의 파장                                            |
| $N_{r,j}^s$                                 | 반송파 위상 미지정수 (정수 값)                               |
| $M_{r,j,P}^s, M_{r,j,\Phi}^s$               | 각각 코드 및 위상 관측값의 다중경로(multipath) 오차          |
| $\epsilon_{r,j,P}^s, \epsilon_{r,j,\Phi}^s$ | 각각 코드 및 위상 관측값의 측정 잡음 및 기타 모델링되지 않은 오차 |


UDUC 모델의 핵심은 모든 오차 요인을 개별 파라미터로 명시적으로 모델링하는 데 있다. 이는 정보의 손실을 최소화하는 장점이 있지만, 동시에 심각한 문제점을 야기하는데, 바로 '등급 결손(rank deficiency)' 문제이다.14 관측 방정식에서 수신기 시계 오차($dt_r$), 위성 시계 오차($dt^s$), 하드웨어 바이어스($d_{r,j}, d_j^s, b_{r,j}, b_j^s$), 그리고 미지정수($N_{r,j}^s$) 항들은 서로 선형 종속 관계에 있다. 예를 들어, 수신기 시계 오차와 수신기 하드웨어 바이어스는 분리하여 추정할 수 없으며, 이들의 조합만이 추정 가능하다. 이러한 파라미터 간의 상관관계 때문에 독립적인 해를 구할 수 없게 된다. 이 문제를 해결하기 위해 특정 파라미터를 기준(datum)으로 고정하거나, 파라미터 간의 특정 조합을 새로운 추정 변수로 정의하는 기법(예: S-basis 이론)이 사용된다.14

PPP-AR의 관점에서 가장 중요한 등급 결손은 위상 미지정수($N_{r,j}^s$)와 위상 하드웨어 바이어스($b_{r,j}, b_j^s$) 간의 선형 결합이다. 위상 관측 방정식에서 이 두 항은 $\lambda_j (N_{r,j}^s + b_{r,j} - b_j^s)$ 형태로 묶여 있어, 순수한 정수 $N_{r,j}^s$를 직접 추정하는 것이 불가능하다. 즉, 하드웨어 바이어스가 미지정수의 정수성을 '오염'시키는 것이다.15

이러한 UDUC 모델의 구조적 한계는 역설적으로 PPP-AR 기술의 핵심 원리를 명확히 보여준다. 전통적인 PPP에서 널리 사용되던 전리층 소거(Ionosphere-Free, IF) 조합은 1차 전리층 오차를 제거하는 데 효과적이지만, 서로 다른 주파수의 하드웨어 바이어스를 복잡하게 결합시켜 새로운 비정수(non-integer) 형태의 미지정수 항($N_{IF}$)을 만들어낸다.16 이로 인해 IF 조합 모델에서는 미지정수의 정수성을 회복하기가 매우 어렵다.

반면, UDUC 모델은 비록 복잡하고 등급 결손 문제를 내포하지만, 원시 관측값의 구조를 그대로 보존함으로써 미지정수와 바이어스의 관계를 명확하게 드러낸다.14 위상 바이어스 항($b_{r,j}, b_j^s$)은 시간에 따라 매우 안정적인 특성을 가지므로, 전 지구적 관측망을 통해 사전에 정밀하게 추정될 수 있다.10 PPP-AR은 바로 이 지점에서 '분할 정복(divide and conquer)' 전략을 취한다. 즉, 글로벌 분석 센터가 위성 위상 바이어스(

$b_j^s$)를 정밀하게 추정하여 보정 정보 형태로 제공하고, 사용자는 이 보정 정보를 자신의 관측 방정식에 적용하여 미지정수 항에서 바이어스 성분을 분리해낸다. 이 과정을 통해 사용자는 '정화된' 미지정수, 즉 정수 특성을 회복한 $N_{r,j}^s$를 얻게 되며, 비로소 정수 결정 알고리즘을 적용할 수 있게 된다. 결국, UDUC 모델의 복잡성은 다중 주파수 신호에 대한 유연성을 극대화하고, 바이어스 분리라는 PPP-AR의 핵심 과제를 수행하기 위한 논리적 토대를 제공하는 강점으로 작용한다.


PPP-AR의 성능은 오염된 위상 미지정수에서 하드웨어 바이어스를 얼마나 효과적으로 분리하고, 그 결과로 얻어진 정수 후보군 중에서 얼마나 빠르고 정확하게 참값(true integer)을 찾아내는지에 달려 있다. 이를 위해 두 가지 핵심 기술, 즉 '위상 바이어스 보정 모델'과 '정수 최소 자승(Integer Least Squares, ILS) 문제 해결 알고리즘'이 유기적으로 결합된다.


앞서 언급했듯이, 원시 위상 관측값에 포함된 미지정수는 위성 및 수신기의 하드웨어에서 발생하는 위상 바이어스로 인해 정수성을 상실한다.18 따라서 정수 결정을 위한 첫 단계는 이러한 바이어스를 정밀하게 보정하여 미지정수의 정수 특성을 복원하는 것이다. 이를 위해 여러 가지 모델이 개발되었으며, 대표적인 모델은 다음과 같다.

| 구분                   | 분수 위상 바이어스 (FCB/UPD)                         | 관측별 신호 바이어스 (OSB)                                   | 정수 복원 시계 (IRC)                                        |
| ---------------------- | ---------------------------------------------------- | ------------------------------------------------------------ | ----------------------------------------------------------- |
| **원리**               | 미지정수의 실수 추정치에서 분수 부분을 분리하여 보정 | 각 원시 관측 신호(예: GPS L1C/A)에 대한 바이어스를 직접 보정 | 위성 시계 오차에 협대역(NL) 위상 바이어스를 흡수시켜 보정   |
| **형태**               | 광대역(WL) 및 협대역(NL) 조합에 대한 보정 정보 제공  | 각 신호별 바이어스 보정 정보 제공                            | 바이어스가 포함된 새로운 시계 보정 정보 제공                |
| **사용자 유연성**      | 낮음 (제공자가 사용한 주파수 조합에 종속적)          | 높음 (사용자가 임의의 주파수 조합을 자유롭게 구성 가능)      | 중간 (사용자 측 모델이 단순화되나, 특정 시계 제품에 종속적) |
| **다중 위성군 적합성** | 낮음 (조합 수가 기하급수적으로 증가)                 | 높음 (신호별 보정으로 확장성이 뛰어남)                       | 중간 (위성군별 시계 처리가 복잡해질 수 있음)                |
| **주요 제공 기관**     | (초기 모델)                                          | IGS, WUM, CNES 등                                            | CNES 등                                                     |


FCB 모델은 PPP-AR 초기에 널리 사용된 방식으로, 비차분 미지정수 실수해(float solution)의 분수 부분(fractional part)을 바이어스로 간주하고 이를 추정하는 모델이다.19 이 모델은 계산의 편의성을 위해 보통 파장이 길고 전리층 오차의 영향이 적은 광대역(Wide-Lane, WL) 조합과, 기하 정보에 민감한 협대역(Narrow-Lane, NL) 조합으로 나누어 각각의 FCB를 추정한다.21 사용자는 분석 센터로부터 제공받은 WL 및 NL FCB 값을 이용하여 자신의 미지정수 추정치를 보정하고 정수성을 복원한다. 이 방식은 이중 주파수 환경에서 효과적이지만, 사용자가 반드시 분석 센터에서 사용한 것과 동일한 주파수 조합을 사용해야 한다는 제약이 있다.22


다중 위성군, 다중 주파수 시대가 도래하면서 FCB 모델의 한계는 명확해졌다. 수많은 주파수 조합에 대해 각각의 FCB를 생성하고 배포하는 것은 비효율적이다.22 이러한 문제를 해결하기 위해 등장한 것이 OSB 모델이다.12 OSB는 특정 주파수 조합이 아닌, 각각의 원시 관측 신호(예: GPS L1 C/A, L2W, L5Q; Galileo E1, E5a)에 대한 고유한 바이어스 값을 제공한다.15 사용자는 이 OSB 보정 정보를 이용하여 자신이 사용하는 모든 신호의 바이어스를 개별적으로 보정한 후, 원하는 어떠한 조합이든 자유롭게 구성하여 정수 결정을 수행할 수 있다. 이러한 유연성 덕분에 OSB는 현재 IGS를 비롯한 대부분의 분석 센터에서 채택하는 표준 방식으로 자리 잡았다.12 이는 GNSS 보정 정보 서비스가 제공자 중심에서 사용자 중심으로 전환되었음을 보여주는 중요한 변화이다. 즉, 사용자가 가용한 모든 위성 신호의 잠재력을 최대한 활용할 수 있도록 기반을 마련해 준 것이다.


IRC는 바이어스를 직접 보정하는 대신, 위성 시계 보정 정보에 협대역 위상 바이어스를 의도적으로 흡수시키는 방식이다.15 이를 통해 사용자 측에서는 별도의 위상 바이어스 보정 없이도 IRC 시계 제품을 적용하는 것만으로 협대역 미지정수의 정수성을 복원할 수 있다. 모델이 단순화되는 장점이 있지만, 특정 시계 제품에 종속된다는 특징이 있다.


위상 바이어스 보정을 통해 미지정수의 정수성이 복원되면, 문제는 가장 가능성 높은 정수 벡터를 찾는 '정수 최소 자승(Integer Least Squares, ILS)' 문제로 귀결된다. 즉, 미지정수의 실수 추정치 벡터를 $\hat{a}$, 그 공분산 행렬을 $Q_{\hat{a}}$라 할 때, 다음의 이차 형식(quadratic form)을 최소화하는 정수 벡터 $a$를 찾는 것이다.24
$$
\min_{a \in Z^n} (\hat{a} - a)^T Q_{\hat{a}}^{-1} (\hat{a} - a)
$$
이 문제는 단순히 $\hat{a}$를 반올림하는 것으로는 최적해를 보장할 수 없으며, 특히 미지정수 간의 상관관계가 높을 경우 오답을 찾을 확률이 매우 높다. 이 ILS 문제를 효율적이고 강건하게 해결하기 위해 가장 널리 사용되는 알고리즘이 바로 LAMBDA(Least-squares AMBiguity Decorrelation Adjustment) 기법이다.24 LAMBDA 기법의 핵심은 어려운 탐색 문제를 그대로 풀기보다, 문제를 쉬운 형태로 변환한 후에 탐색을 수행하는 데 있다. 이는 두 단계로 구성된다.

1. 비상관화 (Decorrelation) 단계: Z-변환

   LAMBDA 기법의 가장 독창적인 부분은 탐색을 시작하기 전에 문제 자체의 구조를 바꾸는 전처리 과정에 있다. ILS 문제가 어려운 근본적인 이유는 미지정수 추정치들 간의 높은 상관관계, 즉 공분산 행렬 $Q_{\hat{a}}$의 비대각 성분들이 큰 값을 갖기 때문이다. 기하학적으로 이는 탐색 공간이 길고 찌그러진 형태의 타원체(ellipsoid) 모양임을 의미하며, 이러한 공간에서 최적의 정수 격자점을 찾는 것은 매우 비효율적이다.27

   Z-변환은 이러한 문제를 해결하기 위해, 정수 가우스 변환(integer Gauss transformations)과 순열(permutation)을 조합하여 구성된 특별한 정수 행렬 $Z$를 사용하여 미지정수 벡터를 새로운 벡터 $z = Za$로 변환한다.24 이 변환은 미지정수의 정수적 성질을 그대로 유지하면서, 변환된 미지정수의 공분산 행렬 $Q_{\hat{z}} = Z Q_{\hat{a}} Z^T$의 비대각 성분을 최소화하여 상관관계를 극적으로 낮춘다. 결과적으로 길고 찌그러졌던 탐색 공간은 구(sphere)에 가까운 형태로 바뀌고, 탐색의 효율성이 비약적으로 향상된다.27 이는 문제의 본질인 '상관관계'를 직접적으로 공격하여, 어려운 최적화 문제를 간단한 문제로 재구성하는 탁월한 접근 방식이다.

2. 탐색 (Search) 단계

   비상관화 단계를 통해 탐색 공간이 효율적인 형태로 변환되면, 이 새로운 공간 내에서 최적의 정수해를 찾는 이산 탐색(discrete search)을 수행한다.26 변환된 미지정수들은 상관관계가 매우 낮으므로, 순차적인 조건부 최소 자승법에 기반한 간단한 탐색만으로도 매우 빠르게 최적의 정수 후보를 찾을 수 있다. 즉, 첫 번째 변환된 미지정수를 특정 정수 값으로 고정하면, 두 번째 미지정수의 탐색 범위가 크게 줄어드는 방식으로 효율적인 탐색이 이루어진다.26 이렇게 찾아낸 최적의 변환된 정수 벡터 $\check{z}$는 다시 역변환($\check{a} = Z^{-1}\check{z}$)을 통해 원래의 미지정수 정수해 $\check{a}$를 얻는 데 사용된다.

결론적으로, LAMBDA 기법의 핵심 철학은 빠른 탐색 알고리즘을 고안하는 것보다, '탐색을 위한 준비' 즉, 문제의 구조를 변환하는 데 더 큰 중점을 둔다는 점이다. 이는 복잡한 최적화 문제 해결에 있어 문제 자체의 변환이 얼마나 중요한지를 보여주는 대표적인 사례라 할 수 있다.


PPP-AR의 최종적인 위치 결정 정확도, 정밀도, 그리고 수렴 시간은 다양한 요인들의 복합적인 상호작용에 의해 결정된다. 이러한 성능 결정 요인은 크게 위성 기하배치, 대기 조건, 데이터 품질 및 지역 환경, 그리고 정밀 보정 정보의 품질로 나눌 수 있다.


위성의 기하학적 배치는 위치 결정의 강건성(strength)을 결정하는 가장 기본적인 요소이다. 이는 보통 정밀도 희석 계수(Dilution of Precision, DOP)로 정량화된다.28 DOP 값이 낮을수록(즉, 위성들이 하늘에 넓게 분포할수록) 기하배치가 좋으며, 이는 미지정수 실수해의 불확실성을 줄여 정수 결정의 성공률을 높이고 수렴 시간을 단축시킨다.30 반대로 가용 위성 수가 적거나 특정 방향에 밀집되어 DOP 값이 높으면, 미지정수 추정의 정확도가 떨어져 정수 결정이 실패하거나 잘못된 정수해를 선택할 위험이 커진다.

다중 위성군(Multi-GNSS)의 활용은 위성 기하배치를 개선하는 가장 효과적인 방법이다. GPS, GLONASS, Galileo, BDS 등 여러 위성 시스템을 동시에 활용하면 가시 위성의 수가 크게 증가하여 DOP 값이 현저히 낮아지고, 특히 도심 협곡이나 숲과 같이 위성 신호가 차폐되는 환경에서도 안정적인 측위를 가능하게 한다.9 향후 저궤도(Low Earth Orbit, LEO) 위성군이 GNSS에 통합되면, LEO 위성의 빠른 움직임으로 인해 위성 기하배치가 급격하게 변하면서 수렴 시간을 수 초 단위로 단축시킬 수 있을 것으로 기대된다.30


GNSS 신호는 지구 대기권을 통과하면서 지연을 겪으며, 이는 PPP-AR의 주요 오차 요인으로 작용한다.

- **전리층 지연 (Ionospheric Delay):** UDUC 모델에서는 각 주파수별 전리층 지연을 직접 추정한다. 전리층은 태양 활동에 따라 변화가 심하며, 특히 태양 활동 극대기나 지자기 폭풍 발생 시에는 예측하기 어려운 불규칙한 지연을 유발한다.33 이러한 비정상적인 전리층 지연은 모델링 오차를 증가시켜 미지정수 추정에 악영향을 미칠 수 있다.28
- **대류층 지연 (Tropospheric Delay):** 대류층 지연은 건조 성분(hydrostatic)과 습윤 성분(wet)으로 나뉜다. 건조 성분은 압력과 온도로 비교적 정확하게 모델링할 수 있지만, 수증기량에 따라 급격히 변하는 습윤 성분은 예측이 매우 어렵다.35 이 습윤 성분 지연을 정밀하게 모델링하지 못하면, 남은 오차가 수직 위치 성분이나 미지정수 추정치에 흡수되어 시스템 전체를 오염시킨다.37 특히, 부정확한 대류층 모델은 위상 관측값의 잔차에 시스템적인 바이어스를 유발하여, LAMBDA와 같은 정수 결정 알고리즘이 올바른 정수해를 찾는 것을 방해하고 결국 정수 결정 실패로 이어진다.35

흥미롭게도, 대류층 모델링과 정수 결정 사이에는 중요한 피드백 루프가 존재한다. 정밀한 대류층 모델은 성공적인 정수 결정을 가능하게 하는 전제 조건이다. 역으로, 일단 정수 결정에 성공하면 미지정수 파라미터가 더 이상 미지수가 아닌 알려진 정수 값으로 고정된다. 이는 전체 추정 시스템의 강건성을 크게 향상시켜, 남아있는 다른 미지수들, 특히 대류층 천정 총 지연(Zenith Total Delay, ZTD) 파라미터를 더욱 정확하게 추정할 수 있게 해준다. 즉, 좋은 대기 모델이 정수 결정을 가능하게 하고, 성공적인 정수 결정은 다시 대기 파라미터 추정의 정확도를 높이는 선순환 구조를 형성한다. 이는 PPP 기술이 기상학 연구(GNSS Meteorology)에 활용되는 이론적 기반이 되기도 한다.3


수신기에서 수집되는 데이터의 품질과 주변 환경 역시 PPP-AR 성능에 직접적인 영향을 미친다.

- **다중경로 오차 (Multipath):** 위성에서 온 직접 신호 외에 주변 건물이나 지형에 반사된 신호가 함께 수신되는 현상으로, 특히 도심 협곡이나 수면 근처에서 심각한 오차를 유발한다.37 다중경로 오차는 코드와 위상 관측값 모두를 오염시키며, 정밀 측위의 정확도를 제한하는 가장 큰 난제 중 하나로 꼽힌다.39 다중경로 반구 모델(Multipath Hemispherical Map, MHM)과 같은 고급 모델링 기법을 통해 일부 완화할 수 있다.37
- **주기 슬립 (Cycle Slip):** 신호 차폐 등으로 인해 반송파 위상 추적이 순간적으로 끊기면서 미지정수 값이 정수만큼 도약하는 현상이다. 주기 슬립을 정확하게 탐지하고 복구하지 못하면 미지정수 추정의 연속성이 깨져 정수 결정이 불가능해진다.


PPP-AR은 외부에서 제공되는 정밀 보정 정보에 절대적으로 의존한다. 위성 궤도, 시계, 그리고 위상 바이어스 보정 정보의 정확도는 사용자 측 위치 결정 정확도의 상한선을 결정한다.28 만약 분석 센터에서 제공하는 보정 정보에 오차가 있거나, 서로 다른 보정 정보(예: 시계와 바이어스) 간에 일관성이 결여될 경우, 이는 그대로 사용자 측의 위치 오차로 전파된다.13 따라서 IGS와 같은 기구는 여러 분석 센터의 산출물을 종합하고 상호 검증하여 높은 품질과 일관성을 유지하는 역할을 수행하며, 이는 PPP-AR 생태계의 신뢰성을 담보하는 핵심적인 활동이다.13


PPP-AR은 고정밀 GNSS 측위 기술의 한 축을 담당하며, 전통적인 강자인 RTK 및 새로운 하이브리드 기술인 PPP-RTK와 고유한 장단점을 바탕으로 공존하고 있다. 이들 기술의 근본적인 차이를 이해하는 것은 특정 응용 분야에 가장 적합한 기술을 선택하는 데 있어 매우 중요하다.

이들 기술의 핵심적인 차이는 단순히 '절대 측위'와 '상대 측위'의 구분을 넘어선다. 더 근본적인 차이는 보정 정보를 생성하고 사용자에게 전달하는 방식, 즉 상태 공간 표현(State Space Representation, SSR)과 관측 공간 표현(Observation Space Representation, OSR)의 철학적 차이에 있다.41

- **OSR (RTK):** RTK는 기준국에서 수집한 원시 관측값 또는 그로부터 파생된 보정치를 로버에게 직접 전달한다. 이 정보는 특정 기준국의 '관측 공간'에 종속적이며, 기준국에서 멀어질수록 공간 상관성이 낮은 오차(예: 대기 오차)를 소거하는 능력이 급격히 저하된다.42 OSR 방식은 개념적으로 단순하고 빠른 초기화를 보장하지만, 광역 서비스를 위해서는 촘촘한 기준국 네트워크와 높은 대역폭의 통신이 필요하여 확장성이 떨어진다.
- **SSR (PPP 계열):** PPP와 PPP-AR은 전 지구적 네트워크를 통해 위성 궤도, 시계, 바이어스 등 GNSS 시스템 전체의 '상태'를 모델링하고, 이 상태 보정 정보를 사용자에게 전달한다.41 이 SSR 보정 정보는 특정 기준국에 종속되지 않고 전 세계 모든 사용자에게 동일하게 적용될 수 있다. 데이터가 간결하여 위성 L-band와 같은 저대역폭 통신으로도 효율적인播송이 가능하며, 이는 전 지구적 확장성의 핵심이다.

이러한 관점에서 볼 때, PPP-RTK의 등장은 단순히 두 기술의 장점을 결합한 것을 넘어, 고정밀 측위 서비스의 아키텍처가 확장성이 뛰어난 SSR 방식으로 진화하고 있음을 시사한다. 이는 향후 수백만 대의 자율 주행차나 IoT 기기를 지원하기 위한 필수적인 변화 방향이다.

다음 표는 각 기술의 주요 특징을 요약하여 비교한 것이다.

| 구분                | PPP                            | PPP-AR                           | RTK                             | PPP-RTK                           |
| ------------------- | ------------------------------ | -------------------------------- | ------------------------------- | --------------------------------- |
| **측위 원리**       | 절대 측위                      | 절대 측위                        | 상대 측위                       | 절대 측위 (지역 보강)             |
| **보정 정보**       | SSR (궤도, 시계)               | SSR (궤도, 시계, 바이어스)       | OSR (원시 관측 데이터)          | SSR (궤도, 시계, 바이어스, 대기)  |
| **인프라 요구사항** | 희소한 글로벌 네트워크         | 희소한 글로벌 네트워크           | 조밀한 지역 네트워크 (기준국)   | 지역 네트워크                     |
| **작동 범위**       | 전 지구                        | 전 지구                          | 기준국 반경 20~30 km 이내       | 수백 km                           |
| **수렴 시간**       | 수십 분 ~ 수 시간              | 수 분 ~ 15분                     | 수 초 (거의 즉시)               | 수 초 ~ 수 분                     |
| **일반 정확도**     | 데시미터 ~ 센티미터            | 센티미터 ~ 밀리미터              | 센티미터                        | 센티미터                          |
| **주요 장점**       | 기준국 불필요, 글로벌 커버리지 | PPP 대비 빠른 수렴, 높은 정확도  | 즉각적인 수렴, 높은 상대 정확도 | RTK급 성능, 넓은 커버리지         |
| **주요 단점**       | 긴 수렴 시간                   | 수렴 시간 필요, 보정 서비스 의존 | 작동 범위 제한, 통신 링크 필수  | 지역 네트워크 및 보정 서비스 의존 |


PPP-AR은 단일 수신기가 IGS와 같은 분석 센터에서 제공하는 SSR 형태의 글로벌 보정 정보(정밀 궤도, 시계, 위상 바이어스)를 이용하여 국제 지구 기준 좌표계(ITRF) 상의 절대 좌표를 결정하는 방식이다.44 지역적인 기준국 인프라로부터 완전히 독립적이므로, 해양, 사막, 산간 지역 등 인프라 구축이 어려운 곳에서도 고정밀 측위가 가능하다는 것이 가장 큰 장점이다.1 그러나 바이어스 보정을 통해 수렴 시간을 단축했음에도 불구하고, 여전히 대기 오차 등을 추정하기 위한 초기 수렴 시간이 필요하다는 점과 보정 정보를 수신하기 위한 인터넷 또는 위성 통신 링크가 필요하다는 단점이 있다.4


RTK는 로버 수신기가 근거리(< 20-30 km)에 위치한 기준국의 원시 관측 데이터를 실시간으로 수신하여 이중 차분(double difference)을 형성하는 방식이다.1 이중 차분 과정에서 위성 시계 및 궤도 오차, 대기 지연 오차 등 공통 오차 대부분이 효과적으로 제거되므로, 거의 즉각적으로 미지정수를 결정하고 센티미터 수준의 매우 높은 상대 정확도를 얻을 수 있다.41 하지만 기준국과의 거리가 멀어지면 대기 오차의 공간적 비상관성으로 인해 성능이 급격히 저하되며, 기준국과의 지속적인 통신 링크 유지가 필수적이라는 점에서 작동 범위가 매우 제한적이다.6


PPP-RTK는 PPP의 글로벌 SSR 프레임워크와 RTK의 지역적 오차 보정 개념을 결합한 하이브리드 기술이다.4 이 기술은 PPP-AR의 표준 SSR 보정 정보에 더하여, 지역 기준국 네트워크에서 추정한 정밀한 대기 상태(전리층 및 대류층) 보정 정보를 함께 사용자에게 제공한다.47 이 추가적인 대기 보정 정보를 통해 사용자는 수렴 과정의 가장 큰 병목 현상을 해소하고, RTK와 유사하게 거의 즉각적인 정수 결정 및 고정밀 측위를 달성할 수 있다. PPP-RTK는 전통적인 RTK보다 훨씬 넓은 지역(수백 km)을 커버하면서도 RTK급 성능을 제공할 수 있어, 차세대 고정밀 측위 서비스의 핵심 기술로 주목받고 있다.4


PPP-AR 기술은 지리적 제약 없이 높은 정확도를 제공하는 독보적인 장점을 바탕으로, 기존의 RTK 기술이 접근하기 어려웠던 광역 또는 원격지 응용 분야에서 새로운 가능성을 열고 있다. 이는 단순히 위치를 측정하는 것을 넘어, 다양한 과학 및 산업 분야에서 새로운 가치를 창출하는 '기반 기술(enabling technology)'로서의 역할을 수행한다. PPP-AR의 핵심 가치는 기준국 의존성을 제거함으로써 얻게 되는 운영상의 자유와 확장성에 있다.


PPP-AR은 전 지구적 규모의 현상을 연구하는 측지학 분야에서 필수적인 도구로 자리 잡았다. 단일 상시 관측소의 데이터만으로도 국제 지구 기준 좌표계(ITRF)에 직접 연결되는 밀리미터 수준의 정밀한 좌표 시계열을 생산할 수 있다.49 이는 지각판의 움직임, 지진 발생 후의 지각 변위, 화산 활동 감시 등 대규모 지구 동역학 현상을 효율적으로 모니터링하는 데 활용된다.5 과거에는 이러한 연구를 위해 조밀한 관측망을 구축하고 차분 처리를 수행해야 했지만, PPP-AR은 단일 관측소의 독립적인 운영을 통해 전 지구적 연구에 기여할 수 있는 길을 열었다. 특히 미지정수 결정은 기존 PPP float 해에 비해 동쪽(East) 방향 성분의 정확도를 크게 향상시키는 것으로 알려져 있다.49


UAV를 이용한 항공 사진 측량에서 지상 기준점(Ground Control Point, GCP)의 설치 및 측량은 전체 작업 시간과 비용의 상당 부분을 차지한다. PPP-AR은 UAV에 탑재된 GNSS 수신기의 이동 궤적을 센티미터 수준의 정확도로 직접 결정(Direct Georeferencing)함으로써 GCP의 필요성을 줄이거나 완전히 제거할 수 있다.51 이는 특히 접근이 어려운 산악 지역이나 광활한 지역의 매핑 작업에서 RTK/PPK 방식의 대안으로 부상하고 있다. 기준국 설치라는 물류적 부담 없이 고정밀 매핑이 가능해져 작업의 효율성과 경제성을 획기적으로 개선할 수 있다.6


자율 주행 자동차, 무인 선박, 드론 등 자율 시스템의 안전한 운용을 위해서는 차선 수준의 구분이 가능한 절대 위치 정보가 필수적이다.17 PPP-AR은 지역적 인프라에 의존하지 않고 전 지구적으로 일관된 고정밀 좌표를 제공하므로, 장거리 운행이나 글로벌 서비스를 목표로 하는 자율 시스템에 이상적인 솔루션을 제공한다.4 다만, 도심 협곡과 같이 위성 신호의 단절과 다중경로가 빈번한 환경은 여전히 큰 도전 과제로 남아 있으며, 이러한 환경에서의 강건성을 확보하기 위한 필터링 기술 연구가 활발히 진행되고 있다.17


광활한 농경지에서 트랙터나 콤바인과 같은 농기계의 자동 조향(auto-steering)을 통해 파종, 시비, 수확 작업을 정밀하게 수행하는 것은 생산성 향상과 자원 낭비 감소에 직결된다.1 RTK는 기준국과의 거리 제한 때문에 대규모 농장 전체를 커버하기 어려운 반면, PPP-AR은 거리 제한 없이 농장 전역에서 일관된 센티미터급 정확도를 제공하여 정밀 농업에 매우 적합하다.2 특히 숲 경계선 등 위성 가시성이 일부 제한되는 농업 환경에서도 다중 위성군을 활용한 PPP-AR은 안정적인 성능을 유지할 수 있다.2


교량, 댐, 고층 빌딩과 같은 대형 구조물의 미세한 변위를 실시간으로 감시하거나, 지진 발생 시 지표면의 움직임을 포착하는 데 고속(1 Hz 이상) PPP-AR 기술이 활용된다.4 이 기술은 수 센티미터 수준의 동적 변위를 실시간으로 감지할 수 있어, 구조 안전 진단 및 지진 조기 경보 시스템에 중요한 데이터를 제공한다.5



본 보고서는 PPP-AR 기술에 대한 포괄적인 고찰을 통해 그 기본 원리, 핵심 알고리즘, 성능 요인, 응용 분야 및 미래 전망을 심도 있게 분석하였다. PPP-AR은 전 지구적 바이어스 보정 정보 인프라를 기반으로 반송파 위상 미지정수의 정수성을 복원함으로써, 기존 PPP 기술의 고질적인 문제였던 긴 수렴 시간을 획기적으로 단축하고 정확도를 향상시켰다. 이를 통해 단일 수신기만으로 전 세계 어디서든 센티미터급 정밀 측위를 가능하게 하여, 측지학에서부터 자율 주행, 정밀 농업에 이르기까지 광범위한 분야에서 새로운 가능성을 열었다. PPP-AR은 지역적 제약이 있는 RTK 기술과 상호 보완적인 관계를 형성하며 고정밀 GNSS 기술의 중요한 축으로 자리매김하였다.


PPP-AR 기술은 괄목할 만한 발전을 이루었지만, 대중적인 실시간 서비스로 자리 잡기까지 해결해야 할 여러 도전 과제가 남아있다.

- **실시간 서비스의 안정성 및 상호 운용성:** 안정적인 실시간 PPP-AR 서비스를 위해서는 정밀 보정 정보(SSR)를 지연 없이 사용자에게 전달하는 것이 중요하다. 통신 지연이나 데이터 단절은 측위 성능의 저하 또는 재수렴을 유발할 수 있으며, 이를 극복하기 위한 보정 정보 예측 알고리즘 연구가 활발히 진행 중이다.23 또한, 여러 분석 센터에서 제공하는 보정 정보 간의 상호 운용성을 확보하는 것은 IGS와 같은 국제 기구의 주요 과제 중 하나이다.13
- **열악한 환경에서의 성능 저하:** 도심 협곡이나 숲과 같이 위성 신호 차폐와 다중경로가 심한 환경은 여전히 PPP-AR의 성능을 저하시키는 가장 큰 요인이다.9 이러한 환경에서도 강건한 측위 성능을 유지하기 위한 고급 필터링 기법 및 다중 센서 융합 기술에 대한 연구가 요구된다.17
- **다중 위성군 통합의 복잡성:** 다중 위성군 활용은 기하배치를 개선하지만, 시스템 간 시각 차이(Inter-System Bias) 및 GLONASS의 주파수 분할 다중 접속(FDMA) 방식과 같은 이질적인 신호 특성을 일관되게 처리해야 하는 복잡성을 증가시킨다.9


이러한 도전 과제에도 불구하고, PPP-AR 기술의 미래는 매우 밝으며, 다음과 같은 신기술들이 그 발전을 더욱 가속화할 것이다.

- **저궤도(LEO) 위성군과의 융합:** LEO 위성들은 기존 GNSS 위성보다 훨씬 낮은 고도에서 빠르게 이동하므로, 위성 기하배치를 급격하게 변화시킨다. LEO 위성 신호를 GNSS와 함께 활용하면 PPP-AR의 수렴 시간을 현재의 수 분 단위에서 수 초, 혹은 단일 에포크 수준으로 단축시킬 수 있을 것으로 기대된다. 이는 PPP-AR 기술의 '게임 체인저'가 될 잠재력을 가지고 있다.30
- **다중 주파수 신호의 활용:** 최신 GNSS 위성들은 3개 이상의 주파수 신호를 송출하고 있다. 추가적인 주파수 정보를 활용하면 파장이 더 긴 새로운 미지정수 조합(예: Extra-Wide-Lane)을 구성할 수 있어, 정수 결정의 속도와 신뢰성을 한층 더 높일 수 있다.8
- **저가형 수신기로의 확산:** 과거 전문가용 고가 장비의 전유물이었던 PPP-AR 기술이 점차 저가형 수신기 칩셋에도 구현되고 있다.59 성능은 아직 측량 등급 수신기에는 미치지 못하지만, 그 격차가 빠르게 줄어들고 있어 자율 주행, 드론, 모바일 기기 등 대중 시장으로의 확산을 견인할 것이다.

궁극적으로 고정밀 측위 기술의 미래는 PPP-AR과 RTK 간의 경쟁이 아닌, 융합을 통한 다층적 서비스 모델로 진화할 것이다. 전 지구를 커버하는 PPP-AR이 센티미터급 정확도의 기본 서비스를 제공하고, 도심과 같이 더 높은 성능이 요구되는 지역에서는 PPP-RTK가 지역 대기 보정 정보를 추가하여 즉각적인 RTK급 성능을 제공하는 형태가 될 것이다. 여기에 LEO 위성군이 가세하여 전 지구적인 즉시 수렴 가능성을 현실화할 것이다. 이러한 유연하고 확장 가능한 SSR 기반의 다층적 아키텍처는 원격지의 농업용 드론부터 도심의 자율주행 택시에 이르기까지, 미래 사회의 다양한 고정밀 측위 수요를 충족시키는 가장 효율적인 해법이 될 것이다. 이들 기술은 서로 경쟁하는 것이 아니라, 하나의 포괄적인 시스템을 구성하는 통합된 요소로 발전해 나갈 것이다.


1. High-Precision GNSS: RTK vs. PPP Positioning | Unmanned Systems Technology, 8월 17, 2025에 액세스, https://www.unmannedsystemstechnology.com/feature/high-precision-gnss-rtk-vs-ppp-positioning/
2. (PDF) Multi-GNSS precise point positioning for precision agriculture - ResearchGate, 8월 17, 2025에 액세스, https://www.researchgate.net/publication/323749495_Multi-GNSS_precise_point_positioning_for_precision_agriculture
3. 2023 Multi-GNSS Precise Point Positioning and PRIDE PPP-AR Short Course, 8월 17, 2025에 액세스, https://www.earthscope.org/event/2023-multi-gnss-precise-point-positioning-and-pride-ppp-ar-short-course/
4. Comparison of PPP, PPP-AR and PPP-RTK - Wisdom Beidou, 8월 17, 2025에 액세스, https://www.wisdombeidou.com/news_detail/21.html
5. Comparison of PPP-AR solution vs. PPP standard approach with two... - ResearchGate, 8월 17, 2025에 액세스, https://www.researchgate.net/figure/Comparison-of-PPP-AR-solution-vs-PPP-standard-approach-with-two-frequencies-Doy_fig12_289533965
6. Accuracy Assessment of RTK, PPK, and PPP-AR Techniques for Direct Georeferencing in UAV-Based Photogrammetric Mapping, 8월 17, 2025에 액세스, https://isprs-archives.copernicus.org/articles/XLVIII-M-6-2025/325/2025/isprs-archives-XLVIII-M-6-2025-325-2025.pdf
7. On Enhanced PPP with Single Difference Between-Satellite Ionospheric Constraints, 8월 17, 2025에 액세스, https://navi.ion.org/content/69/1/navi.505
8. (PDF) Multi-frequency multi-GNSS PPP: A comparison of two ambiguity resolution methods, 8월 17, 2025에 액세스, https://www.researchgate.net/publication/340237039_Multi-frequency_multi-GNSS_PPP_A_comparison_of_two_ambiguity_resolution_methods
9. Multi-GNSS Ambiguity Resolution For Signal Obstruction in PPP - Inside GNSS - Global Navigation Satellite Systems Engineering, Policy, and Design, 8월 17, 2025에 액세스, https://insidegnss.com/multi-gnss-ambiguity-resolution-for-signal-obstruction-in-ppp/
10. Global PPP with Ambiguity Resolution providing improved accuracy and instant position convergence, 8월 17, 2025에 액세스, https://dynamic-positioning.com/proceedings/dp2015/Sensors_Russell_2015.pdf
11. Optimizing PPP-AR with BDS-3 and GPS: Positioning Performance Across Diverse Geographical Regions Under Mostly Quiet Space Weather Conditions - MDPI, 8월 17, 2025에 액세스, https://www.mdpi.com/2073-4433/16/3/288
12. Undifferenced Ambiguity Resolution for Precise Multi-GNSS Products to Support Global PPP-AR - MDPI, 8월 17, 2025에 액세스, https://www.mdpi.com/2072-4292/17/8/1451
13. Precise Point Positioning with Ambiguity Resolution - International GNSS Service, 8월 17, 2025에 액세스, https://igs.org/wg/ppp-ar/
14. (PDF) PPP–RTK functional models formulated with undifferenced ..., 8월 17, 2025에 액세스, https://www.researchgate.net/publication/358583048_PPP-RTK_functional_models_formulated_with_undifferenced_and_uncombined_GNSS_observations
15. BDS-3/GPS/Galileo OSB Estimation and PPP-AR Positioning Analysis of Different Positioning Models - MDPI, 8월 17, 2025에 액세스, https://www.mdpi.com/2072-4292/14/17/4207
16. PPP Fundamentals - Navipedia - GSSC, 8월 17, 2025에 액세스, https://gssc.esa.int/navipedia/index.php/PPP_Fundamentals
17. Robust PPP-AR Using M-Estimators for Multi-Fault Scenarios, 8월 17, 2025에 액세스, https://elib.dlr.de/212752/1/Belles_ION_PLANS_2025_Paper.pdf
18. Review of code and phase biases in multi-GNSS positioning - DiVA portal, 8월 17, 2025에 액세스, https://www.diva-portal.org/smash/get/diva2:1142904/FULLTEXT01.pdf
19. M_FCB: an open‑source software for multi‑GNSS fractional cycle bias estimation, 8월 17, 2025에 액세스, https://www.researchgate.net/publication/387872498_M_FCB_an_open-source_software_for_multi-GNSS_fractional_cycle_bias_estimation
20. An Improved Fast Estimation of Satellite Phase Fractional Cycle Biases - MDPI, 8월 17, 2025에 액세스, https://www.mdpi.com/2072-4292/14/2/334
21. Multi-frequency and multi-GNSS PPP phase bias estimation and ambiguity resolution - SciSpace, 8월 17, 2025에 액세스, https://scispace.com/pdf/multi-frequency-and-multi-gnss-ppp-phase-bias-estimation-and-u3lb0pxwp5.pdf
22. GNSS observable-specific phase biases for all-frequency PPP ambiguity resolution | Request PDF - ResearchGate, 8월 17, 2025에 액세스, https://www.researchgate.net/publication/358366210_GNSS_observable-specific_phase_biases_for_all-frequency_PPP_ambiguity_resolution
23. Evaluation of Real-time Precise Point Positioning with Ambiguity Resolution Based on Multi-GNSS OSB Products from CNES - ResearchGate, 8월 17, 2025에 액세스, https://www.researchgate.net/publication/364302947_Evaluation_of_Real-time_Precise_Point_Positioning_with_Ambiguity_Resolution_Based_on_Multi-GNSS_OSB_Products_from_CNES
24. MLAMBDA: A Modified LAMBDA Method for Integer Ambiguity Determination - McGill School Of Computer Science, 8월 17, 2025에 액세스, https://www.cs.mcgill.ca/~chang/pub/ChangYangZhou.pdf
25. Solving Integer Ambiguity Based on an Improved Ant Lion Algorithm - PMC, 8월 17, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC11860462/
26. THE LAMBDA-METHOD FOR FAST GPS SURVEYING - Satellite ..., 8월 17, 2025에 액세스, https://gnss.curtin.edu.au/wp-content/uploads/sites/21/2016/04/Teunissen1995LAMBDA.pdf
27. The LAMBDA method for integer ambiguity estimation: implementation aspects, 8월 17, 2025에 액세스, https://www.researchgate.net/publication/2790708_The_LAMBDA_method_for_integer_ambiguity_estimation_implementation_aspects
28. The effects of various parameters on the accuracy of kinematic PPP in different platforms - Semantic Scholar, 8월 17, 2025에 액세스, https://pdfs.semanticscholar.org/b27b/5f66f318ace27203b0b00d316e0b0d0d93ed.pdf
29. (PDF) Assessing the influence of differential code bias and satellite geometry on GNSS ambiguity resolution through MANS-PPP software package - ResearchGate, 8월 17, 2025에 액세스, https://www.researchgate.net/publication/371958250_Assessing_the_influence_of_differential_code_bias_and_satellite_geometry_on_GNSS_ambiguity_resolution_through_MANS-PPP_software_package
30. Improvements in PPP by Integrating GNSS with LEO Satellites: A Geometric Simulation, 8월 17, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC12298146/
31. Improvements in PPP by Integrating GNSS with LEO Satellites: A Geometric Simulation, 8월 17, 2025에 액세스, https://www.researchgate.net/publication/393762281_Improvements_in_PPP_by_Integrating_GNSS_with_LEO_Satellites_A_Geometric_Simulation
32. A Rapid-Convergence Precise Point Positioning Approach Using Double Augmentations of Low Earth Orbit Satellite and Atmospheric Information - MDPI, 8월 17, 2025에 액세스, https://www.mdpi.com/2072-4292/15/22/5265
33. (PDF) The Influence of Varying Atmospheric and Space Weather Conditions on the Accuracy of Position Determination - ResearchGate, 8월 17, 2025에 액세스, https://www.researchgate.net/publication/369059065_The_Influence_of_Varying_Atmospheric_and_Space_Weather_Conditions_on_the_Accuracy_of_Position_Determination
34. The Influence of Varying Atmospheric and Space Weather Conditions on the Accuracy of Position Determination - PMC, 8월 17, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC10007505/
35. Assessment of tropospheric modeling on PPP-AR performances under brazilian atmospheric condition - SciELO, 8월 17, 2025에 액세스, https://www.scielo.br/j/bcg/a/HdWMv3cp8zdXvbLWqmVqcHv/?format=pdf&lang=en
36. Assessment of tropospheric modeling on PPP-AR performances under brazilian atmospheric condition - SciELO, 8월 17, 2025에 액세스, https://www.scielo.br/j/bcg/a/HdWMv3cp8zdXvbLWqmVqcHv/
37. Flowchart of PPP multipath mitigation | Download Scientific Diagram - ResearchGate, 8월 17, 2025에 액세스, https://www.researchgate.net/figure/Flowchart-of-PPP-multipath-mitigation_fig2_358078224
38. Challenges in Assessing PPP performance - Garrett Seepersad, 8월 17, 2025에 액세스, https://garrett.seepersad.org/publications/Seepersad_et_al_2014_JAG.pdf
39. PPP-AR positioning errors (m) of four multi-GNSS combination before and... - ResearchGate, 8월 17, 2025에 액세스, https://www.researchgate.net/figure/PPP-AR-positioning-errors-m-of-four-multi-GNSS-combination-before-and-after-multipath_fig3_375136067
40. Performance Analysis of Multi-GNSS Real-Time PPP-AR Positioning Considering SSR Delay - MDPI, 8월 17, 2025에 액세스, https://www.mdpi.com/2072-4292/16/7/1213
41. RTK, PPP and autonomous Types of positioning solutions - GNSS store, 8월 17, 2025에 액세스, https://gnss.store/blog/post/rtk-ppp-and-autonomous.html
42. Differences between RTK and PPP - Tersus GNSS, 8월 17, 2025에 액세스, https://tersus-gnss.com/tech_blog/-Differences-between-RTK-and-PPP
43. GNSS Precise Point Posi.oning (PPP) - UNOOSA, 8월 17, 2025에 액세스, https://www.unoosa.org/documents/pdf/icg/2018/ait-gnss/16_PPP.pdf
44. RTK vs PPP accuracy : r/Surveying - Reddit, 8월 17, 2025에 액세스, https://www.reddit.com/r/Surveying/comments/1jz87a1/rtk_vs_ppp_accuracy/
45. Precise Point Positioning (PPP) - NovAtel, 8월 17, 2025에 액세스, https://novatel.com/an-introduction-to-gnss/resolving-errors/ppp
46. What is PPP-AR, Fast PPP-AR, and PPP-RTK? - Tersus GNSS, 8월 17, 2025에 액세스, https://www.tersus-gnss.com/tech_blog/what-is-ppp-ar-fast-ppp-ar-and-ppp-rtk
47. Application of Atmospheric Augmentation for PPP-RTK with Instantaneous Ambiguity Resolution in Kinematic Vehicle Positioning - MDPI, 8월 17, 2025에 액세스, https://www.mdpi.com/2072-4292/16/15/2864
48. Recent advances and perspectives in GNSS PPP-RTK - TU Delft Research Portal, 8월 17, 2025에 액세스, https://research.tudelft.nl/files/148508274/Hou_2023_Meas._Sci._Technol._34_051002.pdf
49. Contribution of PPP with Ambiguity Resolution to the Maintenance of Terrestrial Reference Frame - MDPI, 8월 17, 2025에 액세스, https://www.mdpi.com/2072-4292/17/7/1183
50. Mitigation of Unmodeled Error to Improve the Accuracy of Multi-GNSS PPP for Crustal Deformation Monitoring :: GFZpublic, 8월 17, 2025에 액세스, https://gfzpublic.gfz-potsdam.de/pubman/faces/ViewItemFullPage.jsp?itemId=item_4892888_5
51. A New Precise Point Positioning with Ambiguity Resolution (PPP-AR) Approach for Ground Control Point Positioning for Photogrammetric Generation with Unmanned Aerial Vehicles - MDPI, 8월 17, 2025에 액세스, https://www.mdpi.com/2504-446X/8/9/456
52. Investigation of accuracy of PPP and PPP-AR methods for direct georeferencing in UAV photogrammetry - ResearchGate, 8월 17, 2025에 액세스, https://www.researchgate.net/publication/364312793_Investigation_of_accuracy_of_PPP_and_PPP-AR_methods_for_direct_georeferencing_in_UAV_photogrammetry
53. Smart PPP-RTK strengthens the future autonomous driving - The Innovation, 8월 17, 2025에 액세스, https://www.the-innovation.org/article/doi/10.59717/j.xinn-geo.2023.100039
54. Mitigation of Unmodeled Error to Improve the Accuracy of Multi-GNSS PPP for Crustal Deformation Monitoring - MDPI, 8월 17, 2025에 액세스, https://www.mdpi.com/2072-4292/11/19/2232
55. Positioning performance of GNSS-PPP and PPP-AR methods for determining the vertical displacements | Request PDF - ResearchGate, 8월 17, 2025에 액세스, https://www.researchgate.net/publication/357025925_Positioning_performance_of_GNSS-PPP_and_PPP-AR_methods_for_determining_the_vertical_displacements
56. Improving Performance of Uncombined PPP-AR Model with Ambiguity Constraints - MDPI, 8월 17, 2025에 액세스, https://www.mdpi.com/2072-4292/16/23/4537
57. From Network to Rover: Near Real-Time PPP Solution with Ambiguity Resolution, 8월 17, 2025에 액세스, https://www.ion.org/publications/abstract.cfm?articleID=18512
58. PrideLab/PRIDE-PPPAR: An open‑source software for Multi-GNSS PPP ambiguity resolution - GitHub, 8월 17, 2025에 액세스, https://github.com/PrideLab/PRIDE-PPPAR
59. Performance Evaluation of Low-Cost and Real-Time Multi-GNSS Advanced Demonstration Tool for Orbit and Clock Analysis-Precise Point Positioning (MADOCA-PPP) Receiver Systems - ResearchGate, 8월 17, 2025에 액세스, https://www.researchgate.net/publication/346509581_Performance_Evaluation_of_Low-Cost_and_Real-Time_Multi-GNSS_Advanced_Demonstration_Tool_for_Orbit_and_Clock_Analysis-Precise_Point_Positioning_MADOCA-PPP_Receiver_Systems
60. Bahel-Multiband-GNSS-low-cost-receiver-and-their-performance-in-accuracy-jtis-Vol-4-N°-1-2024.pdf, 8월 17, 2025에 액세스, [https://jtis-htttcubuea.com/wp-content/uploads/2024/04/Bahel-Multiband-GNSS-low-cost-receiver-and-their-performance-in-accuracy-jtis-Vol-4-N%C2%B0-1-2024.pdf](https://jtis-htttcubuea.com/wp-content/uploads/2024/04/Bahel-Multiband-GNSS-low-cost-receiver-and-their-performance-in-accuracy-jtis-Vol-4-N°-1-2024.pdf)

