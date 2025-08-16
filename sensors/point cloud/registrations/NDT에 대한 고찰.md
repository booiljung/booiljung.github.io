# 포인트 클라우드 정합을 위한 Normal Distributions Transform(NDT)에 대한 고찰

## 1.  정규분포변환의 기본 원리

포인트 클라우드 정합은 로보틱스 및 컴퓨터 비전 분야의 핵심 기술로, 서로 다른 시점이나 시간에 획득된 두 개 이상의 포인트 클라우드 데이터셋을 동일한 좌표계로 정렬하는 과정을 의미한다. 이 기술은 자율주행 차량의 라이다(LiDAR) 주행 기록계(Odometry), 동시적 위치추정 및 지도작성(SLAM), 3차원 객체 재구성 등 다양한 응용 분야의 기반을 형성한다.1 정합 문제는 크게 초기 추정값이 필요한 지역 정합(local registration)과 사전 정보 없이 정렬을 수행하는 전역 정합(global registration)으로 나뉘며, 정규분포변환(Normal Distributions Transform, NDT)은 주로 지역 정합에 사용되지만 전역 정합 프레임워크의 핵심 구성 요소로도 활용된다.1

### 1.1  포인트 클라우드 정합의 과제

포인트 클라우드 정합의 근본적인 목표는 소스 포인트 클라우드(‘P‘)를 타겟 포인트 클라우드(‘Q‘)에 최적으로 정렬하는 강체 변환(rigid body transformation) $T \in SE(3)$를 찾는 것이다.1 전통적인 방법인 반복 최근접점(Iterative Closest Point, ICP) 알고리즘은 소스 클라우드의 각 점에 대해 타겟 클라우드에서 가장 가까운 점을 찾는 대응점 탐색(correspondence search) 과정을 반복한다. 이 대응점 탐색은 계산 비용이 높을 뿐만 아니라, 데이터의 노이즈, 부분적 겹침, 그리고 구조가 부족한 환경에서 잘못된 대응점을 찾아 지역 최솟값(local minima)에 빠지기 쉬운 문제를 안고 있다.3

### 1.2  핵심 개념: 이산적 점에서 연속적, 확률적 표면으로

NDT는 이러한 전통적인 점 기반 접근 방식의 패러다임을 전환한다. 개별 점들을 직접 매칭하는 대신, 타겟 포인트 클라우드를 국소적인 확률 분포의 집합으로 모델링한다.4 이 접근법은 이산적인(discrete) 포인트 클라우드를 기저 표면(underlying surface)에 대한 조각별로 부드럽고(piecewise smooth) 미분 가능한(differentiable) 표현으로 변환한다.4 NDT 함수는 특정 공간 좌표에서 점이 샘플링될 확률을 제공하며, 이는 원본 데이터에 점이 존재하는 위치에만 국한되지 않는다.4

이러한 패러다임 전환의 가장 심오한 혁신은 정합 문제를 조합 탐색 문제(대응점 찾기)에서 연속 최적화 문제(가능도 함수 최대화)로 재구성했다는 점에 있다. ICP의 핵심 루프는 각 점에 대한 최근접 이웃을 찾는 이산적 탐색 연산에 의존하며, 이는 계산적으로 비싸고 직접적으로 미분하기 어렵다.3 반면 NDT는 이 탐색 과정을 확률 모델에 대한 질의, 즉 "변환된 소스 포인트가 타겟 모델에 존재할 확률은 얼마인가?"로 대체한다.4 이 확률은 점의 좌표에 대해 부드럽고 연속적이며 미분 가능한 함수인 가우시안 확률 밀도 함수(PDF)로 정의된다. 변환된 점의 좌표는 변환 파라미터(회전 및 이동)에 대해 미분 가능한 함수이므로, 전체 점수 함수(score function)가 변환 파라미터에 대해 미분 가능하게 된다. 이 덕분에 경사도(gradient)와 헤시안(Hessian) 정보를 활용하여 효율적으로 수렴하는 뉴턴 방법(Newton's method)과 같은 강력한 수치 최적화 기법을 직접 적용할 수 있게 된다.4 이는 ICP 계열 알고리즘과의 근본적인 차별점이다. 나아가 이러한 확률적 공식은 본질적으로 신뢰도 측정치를 제공한다. 최종 점수(`getFitnessScore()`, `getTransformationProbability()`)는 단순한 제곱 거리의 합이 아니라 정렬의 가능도(likelihood)와 관련된 통계적 해석을 가지므로, 보다 원칙에 입각한 평가 지표가 된다.8

### 1.3  역사적 배경과 초기 동기

NDT는 2003년 Peter Biber와 Wolfgang Straßer에 의해 2D SLAM에서의 지도 정합을 위해 처음 소개되었다.4 이후 Martin Magnusson 등에 의해 3D로 확장되었으며 1, 그 주된 동기는 대규모 데이터에 대해 빠르고 정확한 정합 알고리즘을 만드는 것이었다. 이는 ICP의 병목 지점인 명시적인 점 대 점 대응점 탐색을 피함으로써 달성하고자 했다.3

### 1.4  NDT 표현: 압축적이고 미분 가능한 모델

근본적으로 NDT는 데이터 압축 기법이다. 수백만 개에 이를 수 있는 포인트 클라우드를 훨씬 적은 수의 파라미터(평균과 공분산) 집합으로 압축한다.6 이 압축된 표현은 메모리 효율적이며, 이는 대규모 장기 응용 분야에서 위치 추정의 확장성을 확보하는 데 결정적인 역할을 한다.6

## 2.  NDT 알고리즘의 수학적 분석

이 섹션에서는 표준적인 Point-to-Distribution (P2D) NDT 알고리즘의 엄밀한 수학적 토대를 상세히 기술한다.

### 2.1  1단계: 타겟 포인트 클라우드의 복셀화

타겟 포인트 클라우드 $\boldsymbol{Q}$가 차지하는 3차원 공간은 지정된 `Resolution`을 갖는 균일한 격자인 복셀(voxel)로 이산화된다.4 이 복셀 격자 구조는 주변 점들을 효율적으로 그룹화하고 최적화 단계에서 빠른 조회를 가능하게 하는 핵심적인 역할을 한다.16

### 2.2  2단계: 각 복셀의 확률적 모델링

각 복셀 $i$에 대해, 그 안에 포함된 점들의 집합 $\boldsymbol{S_i} \subset \boldsymbol{Q}$는 국소적인 3D 가우시안 분포를 모델링하는 데 사용된다.1

먼저, 평균 벡터(mean vector) $\boldsymbol{\mu_i}$는 복셀 내 점들의 무게 중심으로 계산된다.5
$$
\boldsymbol{\mu_i} = \frac{1}{|\boldsymbol{S_i}|} \sum_{\boldsymbol{q} \in \boldsymbol{S_i}} \boldsymbol{q}
$$
다음으로, 공분산 행렬(covariance matrix) $\boldsymbol{\Sigma_i}$는 점들의 분포 형태와 방향을 설명하기 위해 계산된다.5
$$
\boldsymbol{\Sigma_i} = \frac{1}{|\boldsymbol{S_i}|-1} \sum_{\boldsymbol{q} \in \boldsymbol{S_i}} (\boldsymbol{q} - \boldsymbol{\mu_i})(\boldsymbol{q} - \boldsymbol{\mu_i})^T
$$
안정적인 공분산 행렬을 계산하기 위해서는 최소한의 점 개수(예: PCL 기본값 6개)가 필요하다.16 만약 복셀에 점이 너무 적으면 해당 복셀은 정합에 사용되지 않는다.

NDT 모델에서 가장 정보가 풍부한 구성 요소는 공분산 행렬 $\boldsymbol{\Sigma_i}$이다. 이는 국소적인 표면 기하학을 암묵적으로 인코딩한다. 예를 들어, 평평한 평면 위의 점들을 포함하는 복셀은 평면의 두 축을 따라 높은 분산을 갖고 법선 방향으로는 매우 낮은 분산을 가질 것이다. 이때 공분산 행렬의 고유벡터(eigenvectors)는 이러한 방향들과 정렬되고, 해당 고유값(eigenvalues)은 분산의 크기를 반영한다. 벽의 모서리와 같은 선 위의 점들을 포함하는 복셀의 경우, 분산은 선의 방향을 따라 높고 다른 두 수직 방향으로는 낮을 것이다. $\boldsymbol{\Sigma_i}$의 고유벡터와 고유값은 이 구조 또한 포착한다. 최적화 과정에서 마할라노비스 거리(Mahalanobis distance) 항인 $(\boldsymbol{x'_j}-\boldsymbol{\mu_i})^T \boldsymbol{\Sigma_i}^{-1} (\boldsymbol{x'_j}-\boldsymbol{\mu_i})$는 분산이 낮은 방향으로의 편차에 큰 페널티를 부과한다. 평면의 경우, 점을 평면에서 벗어나게 움직이는 것은 평면을 따라 움직이는 것보다 훨씬 더 큰 페널티를 받는다. 따라서 NDT는 단순히 점들을 평균에 맞추는 것이 아니라, 공분산에 의해 암묵적으로 정의된 국소 기하학적 구조에 맞추는 것이다. 이는 점-대-평면(point-to-plane) ICP 변형과 유사하지만, 명시적으로 특징을 추출하지 않고도 선과 같은 다른 구조를 모델링할 수 있어 더 일반적이다. 이러한 기하학적 인식 능력은 NDT의 강건함(robustness)에 대한 핵심적인 이유 중 하나이다.

### 2.3  3단계: 점수 함수와 최적화 문제

알고리즘의 목표는 소스 클라우드 $\boldsymbol{P}$를 타겟 NDT 모델에 정렬하는 최적의 6-DOF 변환 파라미터 $\boldsymbol{p} = [t_x, t_y, t_z, r_x, r_y, r_z]^T$를 찾는 것이다. 변환 함수 $T(\boldsymbol{p}, \boldsymbol{x})$가 파라미터 $\boldsymbol{p}$에 의해 정의된 변환을 점 $\boldsymbol{x}$에 적용한다고 하자.

변환된 단일 소스 포인트 $\boldsymbol{x'_j} = T(\boldsymbol{p}, \boldsymbol{x_j})$가 타겟 NDT 모델에서 관찰될 확률 밀도는 해당 점이 속한 복셀 $i$의 가우시안 PDF로 근사된다.4
$$
p(\boldsymbol{x'_j}) = \frac{1}{\sqrt{(2\pi)^D \det(\boldsymbol{\Sigma_i})}} \exp\left(-\frac{(\boldsymbol{x'_j}-\boldsymbol{\mu_i})^T \boldsymbol{\Sigma_i}^{-1} (\boldsymbol{x'_j}-\boldsymbol{\mu_i})}{2}\right)
$$
여기서 $D$는 차원(3D의 경우 3)이다. 최대화해야 할 전체 점수는 모든 소스 포인트에 대한 확률의 곱인 가능도(likelihood)이다. 수치적 안정성을 위해, 그리고 문제를 최소화 문제로 전환하기 위해 음의 로그 가능도(negative log-likelihood)가 사용된다. 이는 종종 지수부의 이차 형식(quadratic form)의 합으로 단순화된다.4
$$
\text{score}(\boldsymbol{p}) = -\sum_{j=1}^{|\boldsymbol{P}|} \log(p(T(\boldsymbol{p}, \boldsymbol{x_j}))) \propto \sum_{j=1}^{|\boldsymbol{P}|} (\boldsymbol{x'_j}-\boldsymbol{\mu_i})^T \boldsymbol{\Sigma_i}^{-1} (\boldsymbol{x'_j}-\boldsymbol{\mu_i})
$$
최종 최적화 문제는 이 점수 함수를 최소화하는 파라미터 $\boldsymbol{p}$를 찾는 것이다.4

## 3.  최적화 과정

이 섹션에서는 섹션 2에서 정의된 최소화 문제가 어떻게 해결되는지 상세히 설명한다.

### 3.1  뉴턴 방법을 이용한 최적화

원본 NDT 공식은 최적화를 위해 뉴턴 방법을 사용한다.4 이는 점수 함수의 1차 미분(경사도 또는 자코비안)과 2차 미분(헤시안)을 모두 사용하는 2차 최적화 방법이다. 변환 파라미터 $\boldsymbol{p}$에 대한 업데이트 단계는 다음과 같이 주어진다.
$$
\boldsymbol{p}_{k+1} = \boldsymbol{p}_k - \alpha \boldsymbol{H}^{-1} \boldsymbol{g}
$$
여기서 $\boldsymbol{g}$는 경사도(자코비안) 벡터, $\boldsymbol{H}$는 헤시안 행렬이며, 모두 $\boldsymbol{p}_k$에서 평가된다. $\alpha$는 라인 서치에 의해 결정되는 스텝 크기이다.

뉴턴 방법의 선택은 양날의 검과 같으며, 이는 NDT의 속도에 대한 상반된 보고서들을 직접적으로 설명해준다. 이는 반복 횟수와 반복당 비용 간의 핵심적인 트레이드오프를 강조한다. 뉴턴 방법은 최솟값 근처에서 이차적으로 수렴하므로, 일반적으로 1차 방법보다 훨씬 적은 반복 횟수를 요구한다.9 이것이 NDT의 이론적인 속도 이점이다. 그러나 각 반복은 ‘6×6‘ 헤시안 행렬을 계산하고 역행렬을 구하는 과정을 포함하며, 이는 1차 방법에서 사용되는 경사도 계산보다 훨씬 계산 비용이 높다.11 일부 사용자들이 NDT가 ICP보다 느릴 수 있다고 보고하는 것은 19 이러한 이유 때문이다. 이 모순은 구현의 세부 사항에 의해 해결된다. 순진한 단일 스레드 NDT 구현은 헤시안 계산 때문에 느릴 수 있다. 그러나 NDT_OMP와 같은 병렬화된 버전은 훨씬 빠를 수 있다.20 PCL의 초기 NDT 구현은 속도 문제가 있는 것으로 알려져 있다.20 결론적으로, NDT의 "속도"는 고유한 속성이 아니라 구현의 질, 특히 헤시안이 얼마나 효율적으로 계산되고 병렬화가 사용되는지에 따라 크게 달라진다. 더 적은 반복 횟수라는 이론적 이점은 반복당 비용이 효과적으로 관리될 때만 실제로 실현된다. 이것이 사용자가 최적화되지 않은 NDT 구현보다 간단한 ICP 구현이 더 빠르다고 느낄 수 있는 이유이다.

### 3.2  미분 계산: 자코비안과 헤시안 행렬

최적화의 핵심은 6-DOF 변환 파라미터 $\boldsymbol{p}$에 대한 점수 함수의 경사도 $\boldsymbol{g}$와 헤시안 $\boldsymbol{H}$를 해석적으로 계산하는 것이다. PCL 구현은 이 목적을 위해 `computeDerivatives`, `computeHessian`, `computePointDerivatives`와 같은 함수들을 상세히 정의한다.11

유도 과정은 연쇄 법칙(chain rule)을 포함한다. 점수는 변환된 점 $\boldsymbol{x'}$의 함수이고, 이는 다시 $\boldsymbol{p}$의 함수이다. 미분은 복잡하지만 해석적으로 계산될 수 있다. PCL 문서는 전체 수식을 위해 Magnusson의 2009년 논문을 참조한다.11 단일 점 $\boldsymbol{x_j}$에 대한 변환의 자코비안은 $3 \times 6$ 행렬이고,최종경사도벡터 $\boldsymbol{g}$는 $6 \times 1$ 벡터,헤시안 $\boldsymbol{H}$는 6×6 행렬이다.11

### 3.3  수렴 보장: More-Thuente 라인 서치

순수한 뉴턴 스텝은 최솟값을 지나칠 수 있다. 강건한 수렴을 보장하기 위해, NDT 구현은 종종 탐색 방향 $-\boldsymbol{H}^{-1} \boldsymbol{g}$를 따라 최적의 스텝 크기 $\alpha$를 찾는 라인 서치 알고리즘을 사용한다.10 More-Thuente 라인 서치는 각 반복에서 점수 함수의 충분한 감소를 보장하는 널리 사용되는 방법이다.11 PCL의 `StepSize` 파라미터는 이 탐색을 위한 최대 허용 스텝 길이를 설정한다.16

### 3.4  이산화 문제 해결: 삼선형 보간법

기본 NDT의 주요 한계 중 하나는 점수 함수가 복셀 경계에서 불연속적이라는 점이다.7 변환의 작은 변화가 점을 완전히 다른 가우시안을 가진 새로운 복셀로 이동시켜 점수 값의 급격한 변화를 유발할 수 있다.

이를 완화하기 위해 삼선형 보간법(trilinear interpolation)이 사용될 수 있다. 단일 복셀의 PDF만 사용하는 대신, 한 점에 대한 점수는 주변 8개 복셀의 PDF의 가중 평균으로 계산된다.7 8개의 이웃 셀 $b$ 각각에 대한 가중치 $w(\boldsymbol{x'}, \boldsymbol{\mu_b})$는 점과 셀 중심까지의 거리에 기반하여 결정되며, 이는 더 부드러운 점수 함수를 만든다.7 이러한 평활화(smoothing)는 "수렴의 계곡(valley of convergence)", 즉 초기 오차에 대한 강건성을 증가시키지만, 상당한 계산 비용을 수반한다 (예: 실행 시간이 450% 증가했다는 보고가 있음).7

## 4.  실제 적용: 구현 및 파라미터 설정

이 섹션은 이론과 실제를 연결하며, NDT를 효과적으로 사용하는 방법에 초점을 맞춘다.

### 4.1  주요 구현체

가장 널리 퍼진 오픈소스 구현체는 Point Cloud Library (PCL)이다.10 MATLAB의 Computer Vision Toolbox 또한 `pcregisterndt`라는 이름으로 구현을 제공한다.14 로봇 운영체제(Robot Operating System, ROS)는 다양한 위치 추정 및 지도 작성 패키지에서 PCL 구현을 활발히 활용한다.22

### 4.2  주요 구현 파라미터

NDT 성능을 튜닝하기 위한 핵심 파라미터들은 다음과 같다.

| 파라미터           | PCL 이름                   | MATLAB 이름     | 설명 및 역할                                                 | 주요 출처      |
| ------------------ | -------------------------- | --------------- | ------------------------------------------------------------ | -------------- |
| **복셀 해상도**    | `setResolution`            | `gridStep`      | NDT 격자의 복셀 크기를 정의. 가장 중요하고 스케일에 의존적인 파라미터. 세부 정보 포착(작은 해상도)과 안정적인 통계치를 위한 충분한 점 확보(큰 해상도) 사이의 트레이드오프를 제어. | `[10, 16, 21]` |
| **최대 스텝 크기** | `setStepSize`              | (내재됨)        | More-Thuente 라인 서치의 최대 스텝 길이. 큰 값은 먼 거리를 더 빨리 갈 수 있지만, 최적점을 지나치거나 지역 최솟값에 빠질 위험이 있음. | `[10, 11, 16]` |
| **변환 엡실론**    | `setTransformationEpsilon` | `Tolerance`     | 종료 조건. 반복 간 변환 파라미터의 변화량이 이 값보다 작아지면 알고리즘이 중지됨. 요구되는 정밀도를 정의. | `[10, 16, 21]` |
| **최대 반복 횟수** | `setMaximumIterations`     | `MaxIterations` | 안전 장치. 엡실론 기준으로 수렴하지 못할 경우 최적화기가 무한정 실행되는 것을 방지. | `[10, 16, 21]` |
| **이상치 비율**    | `setOutlierRatio`          | `OutlierRatio`  | 한 점이 이상치(균등 분포에서 발생)일 확률 대 내점(가우시안 분포에서 발생)일 확률을 모델링. 값이 클수록 노이즈와 이상치의 영향을 줄여 알고리즘을 더 강건하게 만듦. | `[11, 17, 21]` |

### 4.3  `Resolution` 튜닝의 기술

이 파라미터는 가장 영향력이 크다.16

- **지침 1:** 해상도는 비어있지 않은 대부분의 복셀이 안정적인 공분산 행렬을 형성하기에 충분한 수의 점(예: 5개 이상)을 포함하도록 충분히 커야 한다.16
- **지침 2:** 해상도는 환경의 고유한 기하학적 특징을 포착할 수 있을 만큼 충분히 작아야 한다. 너무 크면, 가까이 있는 두 개의 벽과 같은 별개의 특징들이 하나의 모호한 "덩어리" 복셀로 합쳐질 수 있다.16
- **실용적 조언:** 예상되는 환경 특징의 스케일과 거의 일치하는 값(예: 큰 방의 경우 1.0m, 복도의 경우 0.5m)으로 시작하여 성능에 따라 조정하는 것이 좋다.

NDT의 파라미터들은 독립적이지 않고 서로 결합된 시스템을 형성한다. 하나의 파라미터를 단독으로 튜닝하면 최적이 아닌 성능을 초래할 수 있다. `Resolution` 파라미터는 문제의 스케일을 정의하는 기본 파라미터이다.16

`StepSize`는 `Resolution`에 비례하여 조정되어야 한다. 해상도보다 훨씬 큰 스텝 크기는 단일 최적화 스텝이 여러 복셀을 뛰어넘게 하여 불안정한 동작을 유발할 수 있다. 합리적인 시작점은 `StepSize`를 `Resolution`과 같은 크기 수준으로 설정하는 것이다.16

`TransformationEpsilon`은 정지 정밀도를 정의하는데, 이 정밀도 역시 `Resolution`의 맥락에서 고려되어야 한다. 복셀 해상도가 2.0m일 때 0.001m의 변환 정밀도를 요구하는 것은 불필요하며 수렴을 방해할 수 있다. 효과적인 NDT 튜닝은 전체적인 접근을 요구한다. `Resolution`이 문제의 스케일을 정의하는 주 파라미터이며, 다른 모든 파라미터(`StepSize`, `TransformationEpsilon`)는 이 기본 스케일과 관련하여 설정되어야 수치적으로 안정적이고 의미 있는 최적화 과정을 보장할 수 있다. 이러한 상호 연결성은 종종 새로운 사용자들에게 혼란의 원인이 된다.22

### 4.4  초기 추정값의 중요성

NDT는 ICP보다 초기 오차에 더 강건하지만, 여전히 지역 최적화 방법이며 초기화에 민감하다.4 좋은 초기 추정값은 빠른 수렴을 달성하고 좋지 않은 지역 최솟값을 피하는 데 매우 중요하다.16 로보틱스에서는 이 초기 추정값이 보통 바퀴 주행기록계나 관성 측정 장치(IMU)와 같은 다른 센서에 의해 제공된다.16 이러한 이유로 NDT는 종종 "거친-정밀(coarse-to-fine)" 정렬 전략의 일부로 사용되며, 다른 방법(또는 매우 거친 해상도의 NDT)이 초기 추정값을 제공한다.4

## 5.  비교 분석: NDT 대 ICP

이 섹션에서는 가장 두드러진 두 지역 정합 알고리즘에 대한 균형 잡힌 증거 기반 비교를 제공한다.

### 5.1  대조적인 방법론

- **ICP:** 명시적인 점-대-점 또는 점-대-평면 대응점을 반복적으로 찾고 제곱 거리의 합을 최소화한다. 원본 포인트 클라우드에 직접 작동한다.3
- **NDT:** 타겟 클라우드를 확률적 모델로 변환하고 이 모델 내에서 소스 점들의 정렬을 최적화한다. 명시적인 대응점 탐색을 피한다.4

### 5.2  성능 벤치마킹: 속도, 정확도, 강건성

- **속도:** 이에 대한 합의는 복잡하고 종종 모순적이다.
  - 이론적으로 NDT는 ICP의 $O(N*M)$ 대응점 탐색을 피하고 더 빠른 수렴률(더 적은 반복 횟수)을 가지므로 더 빠를 수 있다.9
  - 실제로는, 순진한 NDT 구현은 복셀 격자 구축 및 각 단계에서의 헤시안 계산 오버헤드로 인해 최적화된 ICP보다 느릴 수 있다.19
  - 결론은 성능이 구현에 크게 의존한다는 것이다.20
- **정확도:** 일반적으로 ICP(특히 점-대-평면 변형)는 올바르게 수렴할 경우 직접적인 거리 메트릭을 최소화하므로 약간 더 정확한 것으로 간주된다.3 NDT의 정확도는 복셀 해상도에 의해 제한된다.
- **강건성 / 수렴의 계곡:** NDT는 더 큰 초기 정렬 오차에 대해 일관되게 더 강건하다고 보고된다. 부드러운 확률적 점수 함수는 더 넓은 유인 영역(basin of attraction)을 가져 ICP를 가두는 지역 최솟값을 극복할 수 있게 한다.7

NDT와 ICP가 비정형 환경과 정형 환경을 처리하는 방식의 근본적인 차이는 그들의 성능 특성을 설명한다. ICP는 올바른 최근접 이웃을 찾는 데 의존한다. 긴 복도나 평평한 벽과 같은 고도로 정형화된 환경에서는 최근접점이 잘 정의되고 종종 정확하다. 여기서 점-대-평면 ICP가 뛰어난 성능을 보인다. 반면, 나뭇잎이나 잔해 같은 비정형 환경이나 반복적인 기하학 구조를 가진 장면에서는 "가장 가까운 점"이 종종 기하학적으로 부정확하고 의미가 없다. 이는 ICP의 실패로 이어진다. 대조적으로, NDT는 복셀 내의 점들을 평균화한다.5 비정형 환경에서 이 평균화 과정은 부드럽고 덩어리 같은 가우시안 분포를 생성하여, 노이즈가 많고 모호한 기하학을 효과적으로 평탄화한다. 소스 점이 이 "덩어리"에 대해 정합될 때, 알고리즘은 본질적으로 구조의 국소 중심에 정합하는 것이며, 이는 그 안의 어떤 단일 점보다 훨씬 안정적인 특징이다. 결론적으로, ICP는 국소 기하학이 명확하고 모호하지 않을 때 가장 잘 작동한다. NDT의 성능 저하는 더 완만하며, "가장 가까운 점"이라는 개념이 무너지는 모호하고, 비정형적이거나, 특징이 희박한 환경에서 ICP에 비해 상대적으로 뛰어난 성능을 보인다.18 이는 NDT를 자연 환경에 더 나은 선택으로 만들고, 점-대-평면 ICP는 실내의 인공적인 장면에 더 선호될 수 있음을 시사한다.

### 5.3  하이브리드 접근법: NDT-ICP

많은 성공적인 시스템은 두 알고리즘의 강점을 활용하기 위해 2단계 접근법을 사용한다.12

- **전략 1 (NDT-거친, ICP-정밀):** NDT를 사용하여 신속하게 좋은 거친 정렬을 찾은 다음, 그 결과를 ICP의 초기 추정값으로 사용하여 최종적인 고정밀 미세 조정을 수행한다.24
- **전략 2 (ICP-거친, NDT-정밀):** 일부 연구에서는 그 반대의 경우에도 성공을 거두었지만, 이는 덜 일반적이다.12

| 특징            | 정규분포변환 (NDT)                                           | 반복 최근접점 (ICP)                                          |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **핵심 원리**   | 점-대-분포 정합; 확률적                                      | 점-대-점 또는 점-대-평면 정합; 기하학적                      |
| **데이터 표현** | 타겟 클라우드를 복셀화된 가우시안 분포로 변환                | 원본 점 좌표 (및 법선) 사용                                  |
| **대응점**      | 암묵적; 확률 모델 내 점의 위치에 기반                        | 명시적; 최근접 이웃 탐색                                     |
| **최적화**      | 경사도 기반 (뉴턴 방법); 가능도 점수 최소화                  | 교대 최적화; 제곱 거리 합 최소화                             |
| **속도**        | 반복 횟수는 적지만 반복당 비용이 높음. 구현에 따라 크게 다름. `[9, 20]` | 반복 횟수는 많지만 반복당 비용이 낮음. 순진한 구현에서 더 빠름. `[19]` |
| **정확도**      | 좋지만 복셀 해상도에 의해 제한됨. `12`                       | 좋은 초기 추정값이 제공되면 잠재적으로 더 높음. `3`          |
| **강건성**      | 더 넓은 수렴의 계곡; 좋지 않은 초기 추정값과 노이즈에 더 강건함. `[4, 7, 26]` | 더 좁은 수렴의 계곡; 초기 추정값에 매우 민감함. `7`          |
| **파라미터**    | `Resolution`, `StepSize`에 민감함. `16`                      | 덜 민감한 파라미터 (예: 최대 거리 임계값).                   |
| **메모리**      | 복셀 격자 저장을 위해 더 높은 메모리 사용.                   | 더 낮은 메모리 사용.                                         |

## 6.  NDT의 진화: 고급 변형과 현대 연구

이 섹션에서는 원본 NDT 프레임워크를 기반으로 구축된 풍부한 연구 환경을 탐색한다.

### 6.1  P2D 대 D2D: 두 가지 기본 형태

- **Point-to-Distribution (P2D-NDT):** 지금까지 설명된 "고전적인" NDT. 소스 점들을 타겟 분포에 정합한다.15 PCL에 구현된 버전이다.27
- **Distribution-to-Distribution (D2D-NDT):** 소스와 타겟 클라우드 *모두* NDT로 모델링되는 진화된 형태. 최적화는 가우시안 분포 쌍 간의 거리 메트릭(예: L2 거리)을 최소화한다.5
- **비교:** D2D는 분포의 수가 점의 수보다 훨씬 적기 때문에 합산 항의 수가 줄어들어 일반적으로 더 빠르다.26 또한 좋지 않은 초기 추정값에 덜 민감할 수 있다. 그러나 P2D는 때때로 비정형 환경이나 스캔 겹침이 적을 때 더 강건한 것으로 간주된다.18

### 6.2  시맨틱 보조 NDT (SE-NDT, EA-NDT)

- **동기:** 표준 NDT는 동적 객체(예: 자동차, 보행자)에 어려움을 겪으며, 단일 복셀이 다른 표면(예: 벽과 바닥)의 점들을 포함할 때 최적이 아닐 수 있다.2
- **SE-NDT (Semantic-assisted NDT):** NDT를 생성하기 전에 시맨틱 레이블(예: "건물", "도로", "식생")을 기반으로 포인트 클라우드를 분할한다. 각 시맨틱 클래스에 대해 별도의 NDT가 생성된다.5 이를 통해 정합이 안정적인 구조에 집중하고 동적인 구조는 무시할 수 있다.
- **EA-NDT (Environment-Aware NDT):** 한 걸음 더 나아가 경직된 격자 구조를 버린다. 시맨틱 분할과 클러스터링을 사용하여 기하학적으로 의미 있는 셀(예: 평면 패치를 위한 셀, 원통형 기둥을 위한 셀)을 형성한다.15 이는 훨씬 더 효율적이고 정확한 표현을 가져오며, 상당한 지도 압축으로 이어진다.15

### 6.3  전역 위치 추정을 위한 NDT (NDT-Transformer)

- **개념:** 이 접근법은 고전적인 정합과 현대 딥러닝 사이의 간극을 메운다.6 NDT 표현을 새로운 데이터 양식으로 취급한다.
- **방법:** 포인트 클라우드는 NDT 셀 집합으로 변환된다. 풍부한 기하학적 기술자인 이 셀들은 토큰의 "문장"으로 취급된다. 이 토큰 시퀀스는 원래 자연어 처리에서 유래한 모델인 트랜스포머 네트워크에 입력되어 전체 포인트 클라우드에 대한 전역 기술자(global descriptor)를 학습한다.6
- **응용:** 이 학습된 전역 기술자는 이전에 본 장소들의 데이터베이스에서 최상의 일치 항목을 찾아 대규모 장소 인식(루프 클로저 감지)에 사용될 수 있다.6

NDT의 진화는 로보틱스 인식 분야의 더 넓은 추세, 즉 순전히 기하학적인 처리에서 기하학과 고수준(시맨틱, 학습된) 정보의 융합으로의 이동을 반영한다. 원본 NDT와 그 변형들은 순전히 기하학적이다. 이들은 모든 점을 속한 객체에 관계없이 동등하게 취급한다.4 이러한 기하학적 방법의 주요 실패 모드는 동적 환경이다.2 장면을 통과하는 자동차는 정적 세계 가정을 위반하는 점들을 생성하여 정합을 손상시킨다. SE-NDT는 이를 해결하는 첫 단계이다.5 시맨틱 레이블을 사용하여 "자동차"나 "사람" 클래스에 속하는 점들을 식별하고 폐기함으로써 정합을 동적인 상황에 강건하게 만든다. 이는 기하학적 처리(NDT)와 시맨틱 이해의 직접적인 융합이다. EA-NDT는 이를 더 발전시킨다.15 필터링에만 시맨틱을 사용하는 것이 아니라, NDT 표현 자체를 근본적으로 재구성하여, 단순한 격자에서 지능적이고 객체 인식적인 셀로 이동한다. NDT-Transformer는 이 추세의 마지막 단계를 대표한다.6 기하학적 NDT 표현을 대규모 딥러닝 모델의 

*특징*으로 사용하여, 정합 문제를 완전히 추상화된 학습된 특징 정합 문제로 전환한다. 결론적으로, NDT는 더 이상 단일 알고리즘이 아니라 유연한 *표현 프레임워크*이다. 복셀 단위 가우시안 분포라는 핵심 구성 요소는 시맨틱으로 향상되거나, 새로운 데이터 구조에 사용되거나, 딥러닝 모델의 입력으로 사용될 수 있는 강력하고 압축적인 기하학적 기술자임이 입증되었다. 이러한 적응성이 NDT가 지속적으로 중요성을 갖는 핵심 이유이다.

## 7.  응용 및 한계

이 섹션에서는 논의를 실제 사용 사례에 기반을 두고 NDT의 경계에 대한 비판적 평가를 제공한다.

### 7.1  핵심 응용: 동시적 위치추정 및 지도작성 (SLAM)

- **라이다 주행 기록계:** NDT는 로봇의 증분 운동을 계산하기 위한 스캔-대-스캔 정합의 주력 기술로, 많은 SLAM 시스템의 프론트엔드를 형성한다.2
- **스캔-대-지도 위치 추정:** 일반적인 사용 사례는 현재 라이다 스캔을 미리 구축된 NDT 지도에 대해 위치를 추정하는 것이다. 이는 자율주행 차량의 기본 기능이다.8
- **그래프 기반 SLAM:** NDT에 의해 계산된 변환은 포즈 그래프에서 포즈(노드) 사이의 제약 조건(에지) 역할을 한다. 그런 다음 그래프 최적화기가 누적된 드리프트를 보정한다.2
- **루프 클로저:** NDT는 현재 스캔을 과거의 후보 스캔과 정합하여 루프를 감지하고 닫는 데 사용될 수 있으며, 이는 전역적으로 일관된 지도를 만드는 데 필수적이다.5

### 7.2  실제 환경에서의 과제

- **동적 환경:** 논의된 바와 같이, 움직이는 객체는 정적 세계 가정을 위반하며 성능을 심각하게 저하시킬 수 있다. 이는 교통량이 많은 도심 지역에서 주요 문제이다.2 시맨틱 기반 변형은 이 한계에 대한 직접적인 대응책이다.
- **특징이 부족한 환경:** 긴 터널이나 복도와 같은 자기 유사 환경에서는 NDT 점수 함수가 특정 방향으로 평평해져, 특히 이동 방향에서 높은 불확실성과 드리프트를 유발할 수 있다.7
- **확장성 및 장기 매핑:** NDT 표현은 압축적이지만, 대규모 지도를 시간이 지남에 따라 관리하고 업데이트하는 것은 과제를 제기한다. 처리 가능성을 유지하기 위해 종종 서브맵(submap) 기반 접근법이 사용된다.6

### 7.3  산업 및 검사 응용 분야의 NDT

이 보고서는 포인트 클라우드 정합 알고리즘에 초점을 맞추고 있지만, "NDT"라는 용어는 산업 검사에서 "비파괴 검사(Non-Destructive Testing)"를 의미하기도 한다. 로보틱스와 3D 스캐닝이 이러한 전통적인 비파괴 검사 작업을 자동화하는 데 사용되면서 흥미로운 융합이 일어나고 있다.

- **로봇 검사:** 센서(초음파, 와전류)를 장착한 로봇이 탱크나 파이프라인과 같은 대형 구조물에 대한 비파괴 검사를 수행하여 인간을 위험한 환경에서 벗어나게 한다.30
- **손상 평가를 위한 3D 스캐닝:** 고해상도 3D 스캐너는 부식, 찌그러짐 및 기타 기계적 손상을 평가하기 위해 구성 요소의 정밀한 디지털 모델을 생성하는 데 사용되며, 피트 게이지와 같은 수동 도구를 대체한다.32 이러한 스캔에서 생성된 포인트 클라우드는 NDT와 같은 알고리즘을 사용하여 시간 경과에 따른 손상 진행 상황을 추적하기 위해 정합될 수 있다.

NDT 알고리즘과 NDT 산업 프로세스 사이에는 재귀적인 관계가 존재한다. NDT 알고리즘에 의해 가능해진 SLAM 기능은 로봇이 물리적인 NDT 검사 작업을 자동화하는 데 필요한 3D 지도를 탐색하고 구축할 수 있게 하는 바로 그 기술이다. 전통적인 비파괴 검사(예: 대형 탱크의 초음파 검사)는 종종 위험하고 밀폐된 공간에서 인간 작업자를 필요로 한다.30 이를 자동화하기 위해 로봇은 탱크 내부 또는 주변을 탐색할 수 있어야 한다. 이러한 환경은 종종 GPS 신호가 닿지 않으므로 로봇은 자신을 위치시키고 지역을 매핑하기 위해 SLAM 알고리즘을 사용해야 한다.35 NDT 정합 알고리즘은 강건하고 환경의 지도(NDT 지도)를 구축할 수 있기 때문에 이 SLAM 시스템에 가장 적합한 후보다.2 로봇이 탱크의 자체 지도 내에서 위치를 파악하면, 물리적 검사를 수행하고 알려진 위치에서 데이터를 기록하기 위해 NDT 센서(예: UT 프로브)를 정밀하게 위치시킬 수 있다.31 결론적으로, NDT 알고리즘은 NDT 검사의 로봇 자동화를 위한 핵심 구현 기술이다. 이는 로봇이 주요 검사 작업을 수행하기 위한 전제 조건인 "나는 어디에 있는가?" 문제를 해결한다. 이것이 "NDT"의 두 가지 의미 사이에 강력한 시너지를 창출한다.

## 8.  종합 및 미래 방향

이 마지막 섹션에서는 주요 결과를 요약하고 NDT의 미래에 대해 전망한다.

### 8.1. 강점과 약점 요약

- **강점:** 속도(최적화된 구현에서), 초기 오차에 대한 강건성, 명시적 대응점 불필요, 원칙에 입각한 확률적 기반. 표현력은 적응성을 높인다.
- **약점:** `Resolution` 파라미터에 대한 민감성, 매우 동적이거나 특징 없는 환경에서의 성능 저하, 순진한 구현에서의 헤시안 계산 비용.

### 8.1  미래는 하이브리드: 기하학, 시맨틱, 학습의 융합

명백한 추세는 순수 NDT에서 벗어나는 것이다. 미래는 NDT의 기하학적 효율성과 시맨틱 분할의 문맥적 인식, 그리고 딥러닝의 패턴 인식 능력을 결합한 하이브리드 시스템에 있다. 지도 표현을 생성하기 위해 데이터 기반 방법을 사용하는 EA-NDT와 유사한 접근법과, NDT를 학습 기반 위치 추정 시스템의 프론트엔드로 사용하는 NDT-Transformer와 유사한 방법들에 대한 더 많은 연구가 기대된다.

### 8.2  결론

거의 20년 전에 소개된 NDT는 단일 알고리즘 이상임이 입증되었다. 그것은 포인트 클라우드 처리를 위한 기초적인 *표현 프레임워크*이다. 3D 공간을 압축적이고 미분 가능하게 모델링하는 능력은 그 지속적인 중요성을 보장했다. 원본 구현에는 한계가 있지만, 핵심 아이디어는 로보틱스와 자율 주행 연구의 최전선에 남아 있는 풍부하고 진화하는 알고리즘 군을 낳았다. NDT 프레임워크는 과거의 유물이 아니라 미래의 인식 시스템이 구축될 초석이다.

#### **참고 자료**

1. Fast Global Point Cloud Registration using Semantic NDT, accessed July 24, 2025, https://www.ipb.uni-bonn.de/wp-content/papercite-data/pdf/schirmer2024iros.pdf
2. Performance Analysis of NDT-based Graph SLAM for Autonomous Vehicle in Diverse Typical Driving Scenarios of Hong Kong - PubMed Central, accessed July 24, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC6263388/
3. A Benchmark for Point Clouds Registration Algorithms - arXiv, accessed July 24, 2025, https://arxiv.org/pdf/2003.12841
4. Normal distributions transform - Wikipedia, accessed July 24, 2025, https://en.wikipedia.org/wiki/Normal_distributions_transform
5. Semantic-assisted 3D Normal Distributions ... - ILIAD Project, accessed July 24, 2025, http://iliad-project.eu/wp-content/uploads/papers/iros_se_ndt.pdf
6. NDT-Transformer : large-scale 3D point cloud localisation using the normal distribution transform representation - White Rose Research Online, accessed July 24, 2025, https://eprints.whiterose.ac.uk/172335/1/NDT_TRANSFORMER_VLAD__Final_.pdf
7. Evaluation of 3D Registration Reliability and Speed–A Comparison ..., accessed July 24, 2025, https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=0185f6fb7a7a7859daf8ae326b485e9891a7461d
8. [Hello World!] – Localisation (NDT) - StreetDrone, accessed July 24, 2025, https://www.streetdrone.com/hello-world-localisation-ndt/
9. Total execution time of PM, ICP and NDT | Download Scientific Diagram - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/figure/Total-execution-time-of-PM-ICP-and-NDT_fig3_322664340
10. pcl/doc/tutorials/content/sources/normal_distributions_transform/normal_distributions_transform.cpp at master · PointCloudLibrary/pcl - GitHub, accessed July 24, 2025, https://github.com/PointCloudLibrary/pcl/blob/master/doc/tutorials/content/sources/normal_distributions_transform/normal_distributions_transform.cpp
11. pcl: pcl::NormalDistributionsTransform< PointSource, PointTarget ..., accessed July 24, 2025, http://docs.ros.org/hydro/api/pcl/html/classpcl_1_1NormalDistributionsTransform.html
12. Research on Point Cloud Registering Method of Tunneling Roadway Based on 3D NDT-ICP Algorithm - MDPI, accessed July 24, 2025, https://www.mdpi.com/1424-8220/21/13/4448
13. gitBook_PCL/Tutorial/Registration/How-to-use-Normal-Distributions-Transform.md at master, accessed July 24, 2025, https://github.com/adioshun/gitBook_PCL/blob/master/Tutorial/Registration/How-to-use-Normal-Distributions-Transform.md
14. show - Visualize normal distributions transform (NDT) map - MATLAB - MathWorks, accessed July 24, 2025, https://www.mathworks.com/help/vision/ref/pcmapndt.show.html
15. Towards High-Definition Maps: a Framework Leveraging Semantic ..., accessed July 24, 2025, https://arxiv.org/pdf/2301.03956
16. How to use Normal Distributions Transform — Point Cloud Library ..., accessed July 24, 2025, https://pointclouds.org/documentation/tutorials/normal_distributions_transform.html
17. pcl/registration/include/pcl/registration/ndt.h at master · PointCloudLibrary/pcl - GitHub, accessed July 24, 2025, https://github.com/PointCloudLibrary/pcl/blob/master/registration/include/pcl/registration/ndt.h
18. Beyond Points: Evaluating Recent 3D Scan-Matching Algorithms - SPENCER Project, accessed July 24, 2025, http://spencer.eu/papers/MagnussonICRA2015.pdf
19. Comparision time of ICP and NDT in Matching - Robotics Stack Exchange, accessed July 24, 2025, https://robotics.stackexchange.com/questions/84063/comparision-time-of-icp-and-ndt-in-matching
20. Why ICP is much faster than NDT in front_end? · Issue #127 · koide3/hdl_graph_slam, accessed July 24, 2025, https://github.com/koide3/hdl_graph_slam/issues/127
21. pcregisterndt - Register two point clouds using NDT algorithm - MATLAB - MathWorks, accessed July 24, 2025, https://www.mathworks.com/help/vision/ref/pcregisterndt.html
22. How to properly set the hyper parameters for PCL's NDT algorithm? - Stack Overflow, accessed July 24, 2025, https://stackoverflow.com/questions/77194736/how-to-properly-set-the-hyper-parameters-for-pcls-ndt-algorithm
23. abougouffa/ndtpso_slam: ROS package for NDT-PSO, a 2D Laser scan matching algorithm for SLAM - GitHub, accessed July 24, 2025, https://github.com/abougouffa/ndtpso_slam
24. Point Cloud Registration Algorithm Based on NDT and Feature Point Detection, accessed July 24, 2025, https://www.researching.cn/articles/OJe5f35bb9ef244d4c
25. Evaluation of 3D registration reliability and speed : a comparison of ICP and NDT - DiVA, accessed July 24, 2025, http://oru.diva-portal.org/smash/record.jsf?pid=diva2:274922
26. NDT literature review, accessed July 24, 2025, https://autowarefoundation.gitlab.io/autoware.auto/AutowareAuto/ndt-literature-review.html
27. [registration] Distribution-to-Distribution NDT (D2D-NDT) · Issue #5482 · PointCloudLibrary/pcl - GitHub, accessed July 24, 2025, https://github.com/PointCloudLibrary/pcl/issues/5482
28. (PDF) Beyond points: Evaluating recent 3D scan-matching algorithms - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/279911894_Beyond_points_Evaluating_recent_3D_scan-matching_algorithms
29. A Multi-Layered 3D NDT Scan-Matching Method for Robust Localization in Logistics Warehouse Environments - MDPI, accessed July 24, 2025, https://www.mdpi.com/1424-8220/23/5/2671
30. Case Study: Robotic Tech & UT Testing for Accuracy and Reduced Risk | Nexxis, accessed July 24, 2025, https://nexxis.com.au/case-study-robotic-tech-ut-testing-for-accuracy-and-reduced-risk/
31. (PDF) Integration of Robotics and Artificial Intelligence in Non-Destructive Testing: A Review of Applications and Challenges - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/372817358_Integration_of_Robotics_and_Artificial_Intelligence_in_Non-Destructive_Testing_A_Review_of_Applications_and_Challenges
32. 3D Scanning & Reverse Engineering Case Studies - Industrial Inspection & Consulting, accessed July 24, 2025, https://industrialinspection.com/3d-scanning-reverse-engineering-case-studies/
33. THE POWER OF 3D SCANNING FOR CORROSION AND MECHANICAL DAMAGE ASSESSMENT - Creaform, accessed July 24, 2025, https://www.creaform3d.com/en/resources/blog/the-power-of-3d-scanning-for-corrosion-and-mechanical-damage-assessment
34. 3D scanning in refineries: Boosting inspection speed and accuracy - Creaform, accessed July 24, 2025, https://www.creaform3d.com/en/resources/blog/improving-vessel-integrity-at-a-refinery-thanks-to-advanced-3d-scanning-technology
35. What is SLAM (Simultaneous Localisation and Mapping)? | Nexxis USA, accessed July 24, 2025, https://nexxis.com/what-is-slam-3d-mapping/



