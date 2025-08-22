---
layout: page
title: KISS-ICP 포인트 클라우드 정합
permalink: /sensors/point cloud/registrations/KISS-ICP
---



3차원 공간을 이해하고 상호작용하는 능력은 현대 로보틱스와 컴퓨터 비전 시스템의 핵심적인 요구사항이다. 이러한 능력의 근간에는 포인트 클라우드(Point Cloud) 데이터가 자리 잡고 있다. 포인트 클라우드는 LiDAR(Light Detection and Ranging) 센서, RGB-D 카메라, 스테레오 카메라 등 다양한 장비를 통해 수집된 3차원 공간상의 점들의 집합으로, 각 점은 $x, y, z$ 좌표를 기본 정보로 가진다.1 이 데이터는 주변 환경의 기하학적 형상을 매우 정밀하게 표현할 수 있어 자율주행, 로보틱스, 3D 재구성, 문화유산 디지털화 등 광범위한 분야에서 필수적으로 활용된다.1

그러나 단일 포인트 클라우드만으로는 완전한 환경 정보를 얻기 어렵다. 로봇이나 차량이 이동하면서 여러 시점에서 순차적으로 데이터를 획득하거나, 여러 센서를 동시에 사용하여 환경을 스캔하는 경우가 일반적이다. 이때 각기 다른 좌표계를 기준으로 생성된 여러 개의 포인트 클라우드 조각들을 하나의 일관된 전역 좌표계로 정렬하는 과정이 필요하며, 이를 '포인트 클라우드 정합(Point Cloud Registration)'이라 부른다.2 정합은 분리된 데이터 조각들을 마치 퍼즐처럼 맞춰 하나의 완전한 3D 모델을 생성하는 과정으로, 로봇의 위치를 추정(Localization)하고 주변 지도를 작성(Mapping)하는 SLAM(Simultaneous Localization and Mapping) 기술의 가장 기본적인 구성 요소이다.1


지난 수십 년간 포인트 클라우드 정합 분야를 지배해 온 고전적이면서도 강력한 알고리즘은 단연 ICP(Iterative Closest Point)이다.2 ICP의 기본 원리는 매우 직관적이다. 하나의 포인트 클라우드(소스, Source)를 점진적으로 변환(회전 및 이동)시켜, 다른 고정된 포인트 클라우드(타겟, Target)에 가장 잘 들어맞도록 반복적으로 최적화하는 방식이다.2

하지만 ICP는 초기 추정값에 민감하여 지역 최솟값(local minima)에 빠지기 쉽고, 데이터 내에 존재하는 이상치(outlier)에 취약하다는 근본적인 한계를 가지고 있다.1 이러한 한계를 극복하기 위해 로보틱스 및 SLAM 연구 커뮤니티는 점차 복잡성이 증가하는 방향으로 발전해왔다. 예를 들어, Point-to-Plane ICP와 같이 더 정교한 오차 척도를 사용하거나 7, FPFH(Fast Point Feature Histograms), SIFT(Scale-Invariant Feature Transform)와 같은 복잡한 기하학적 특징(feature)을 추출하여 대응점의 정확도를 높이려 시도했다.4 더 나아가, IMU(Inertial Measurement Unit)나 바퀴 주행기록계(Wheel Odometry)와 같은 추가적인 센서 정보를 융합하여 움직임을 예측하고, 최근에는 딥러닝 기술을 도입하여 특징 추출 및 정합 과정을 데이터 기반으로 학습시키려는 연구가 주류를 이루었다.5

이러한 복잡성의 증가는 특정 조건에서는 성능 향상을 가져왔지만, 동시에 새로운 문제를 야기했다. 시스템은 수많은 파라미터를 갖게 되었고, 이를 특정 센서, 환경, 로봇의 움직임 프로파일에 맞게 조정하는 것은 "지루하고 힘든(tedious)" 작업이 되었다.11 그 결과, 많은 시스템들이 특정 벤치마크 데이터셋에는 최적화되었지만 다른 환경에서는 잘 작동하지 않는 일반화(generalization) 능력의 부재라는 문제에 직면했다.13

이러한 배경 속에서, 2022년 Ignacio Vizzo 연구팀은 기존 연구 동향에 대한 근본적인 반론을 제기하며 KISS-ICP(Keep It Small and Simple ICP)를 제안했다.12 그들의 철학은 명확하다: "복잡성을 버리고 핵심으로 돌아가자.".15 KISS-ICP는 지난 30년간 제안된 수많은 복잡한 기법들을 의도적으로 배제하고, 1992년 Besl과 McKay에 의해 처음 제안된 가장 기본적인 Point-to-Point ICP 알고리즘으로 회귀한다.13 연구팀은 이 고전적인 방법이 '올바른 방식(if done the right way)'으로 구현될 경우, 여전히 최첨단(State-of-the-Art, SOTA) 성능을 달성할 수 있으며, 오히려 복잡한 시스템들보다 더 강건하고 범용적일 수 있음을 증명하고자 했다. 이는 기술 발전이 항상 복잡성 증가와 동의어가 아니며, 때로는 기본 원리에 대한 깊은 이해와 정교한 재해석이 더 큰 혁신을 가져올 수 있다는 중요한 시사점을 던진다. 기존 연구들이 ICP의 한계를 외부 정보나 복잡한 모델을 '더하여' 보완하려 했다면, KISS-ICP는 ICP 자체를 올바르게 작동시키기 위해 불필요한 요소들을 '빼는' 역발상을 통해 새로운 길을 제시한 것이다.


본 보고서는 포인트 클라우드 정합 분야에 중요한 화두를 던진 KISS-ICP 알고리즘을 심층적으로 고찰하는 것을 목표로 한다. 이를 위해 먼저 제2장에서 고전적 ICP 알고리즘의 원리와 수학적 모델, 그리고 그 한계를 재조명한다. 제3장에서는 KISS-ICP의 핵심 철학인 'Keep It Small and Simple'이 어떻게 네 가지 핵심 구성요소를 통해 구현되었는지 상세히 분석한다. 제4장에서는 KISS-ICP의 수학적 모델, 특히 강건한 비용 함수와 최적화 과정을 수식을 통해 구체적으로 탐구한다.

이어지는 제5장에서는 KISS-ICP를 G-ICP, NDT와 같은 다른 주요 정합 알고리즘과 다각적으로 비교하여 기술적 위치와 장단점을 명확히 규명한다. 제6장에서는 KITTI와 같은 표준 벤치마크 데이터를 기반으로 한 실증적 성능 평가 결과를 분석하고, 그 의미를 고찰한다. 마지막으로 제7장에서는 KISS-ICP의 광범위한 응용 분야와 명확한 한계점, 그리고 KISS 철학을 계승하는 미래 연구 방향을 논하며 보고서를 마무리한다. 본 보고서는 독자에게 KISS-ICP에 대한 깊이 있는 이해와 함께, 포인트 클라우드 정합 기술의 현재와 미래에 대한 균형 잡힌 시각을 제공하고자 한다.


KISS-ICP의 혁신성을 이해하기 위해서는 그 근간이 되는 고전적 ICP 알고리즘에 대한 깊이 있는 이해가 선행되어야 한다. ICP는 수많은 변종을 낳으며 지난 30년간 3D 정합 분야의 표준으로 자리매김해왔다.2 이 장에서는 표준 ICP의 작동 방식과 두 가지 주요 변형인 Point-to-Point 및 Point-to-Plane 방식의 수학적 원리를 탐구하고, 고전적 접근법이 가진 내재적 한계를 명확히 한다.


ICP 알고리즘은 이름에서 알 수 있듯이, 가장 가까운 점들을 찾아 점진적으로 정합 오차를 최소화하는 반복적인 프로세스다.1 이 과정은 일반적으로 다음과 같은 세 단계로 구성된다 2:

1. **대응점 탐색 (Correspondence Search):** 첫 번째 단계는 두 포인트 클라우드 간의 대응 관계를 설정하는 것이다. 소스 포인트 클라우드 $P = \{p_i\}$의 각 점 $p_i$에 대해, 타겟 포인트 클라우드 $Q = \{q_i\}$에서 가장 가까운 점 $q_i$를 찾는다. 모든 점 쌍에 대한 거리를 계산하는 것은 계산 비용이 매우 높기 때문에, 이 과정을 가속화하기 위해 일반적으로 KD-Tree와 같은 공간 분할 자료구조가 사용된다.1 KD-Tree를 타겟 포인트 클라우드에 대해 미리 구축해두면, 각 소스 포인트에 대한 최근접 이웃 탐색(Nearest Neighbor Search)을 

   $O(\log N)$의 시간 복잡도로 효율적으로 수행할 수 있다.

2. **변환 추정 (Transformation Estimation):** 대응점 탐색을 통해 얻어진 대응점 쌍의 집합 $C = \{(p_i, q_i)\}$가 주어지면, 이들 간의 정합 오차를 최소화하는 최적의 강체 변환(Rigid Transformation) 행렬 $T$를 계산한다. 이 변환 행렬은 3차원 회전을 나타내는 회전 행렬 $R \in SO(3)$과 3차원 이동을 나타내는 변환 벡터 $t \in \mathbb{R}^3$로 구성된다. 오차를 어떻게 정의하느냐에 따라 ICP의 변종이 나뉜다.

3. **적용 및 반복 (Apply and Iterate):** 계산된 변환 행렬 $T$를 소스 포인트 클라우드 $P$의 모든 점에 적용하여 새로운 위치로 업데이트한다 ($P' = T \cdot P$). 이후, 정합 오차(예: 대응점 간 평균 거리)를 계산하고, 이 오차가 미리 정의된 임계값 이하로 줄어들거나, 오차의 변화량이 매우 작아지거나, 최대 반복 횟수에 도달하는 등 특정 수렴 조건을 만족할 때까지 1~3단계를 반복한다.2


ICP의 성능과 특성은 2단계 '변환 추정'에서 어떤 목적 함수(objective function)를 최소화하는지에 따라 크게 달라진다. 가장 대표적인 두 가지 방식은 Point-to-Point와 Point-to-Plane이다.


가장 기본적이고 고전적인 ICP 방식이다.1 이 방식은 변환된 소스 포인트 $(R p_i + t)$와 그에 대응하는 타겟 포인트 $q_i$ 사이의 유클리드 거리(Euclidean distance) 제곱의 합을 최소화하는 것을 목표로 한다.1 목적 함수 $E_{p2p}(R, t)$는 다음과 같이 정의된다:
$$
E_{p2p}(R, t) = \sum_{i=1}^{N} \| (R p_i + t) - q_i \|^2
$$
이 목적 함수의 가장 큰 장점은 SVD(Singular Value Decomposition, 특이값 분해)를 통해 해석적 해(closed-form solution)를 효율적으로 구할 수 있다는 점이다.1 최적의 변환 

$(R^*, t^*)$를 찾는 과정은 다음과 같다 17:

1. 두 대응점 집합의 중심점(centroid) $\mu_p$와 $\mu_q$를 계산한다.
   $$
   \mu_p = \frac{1}{N} \sum_{i=1}^{N} p_i, \quad \mu_q = \frac{1}{N} \sum_{i=1}^{N} q_i
   $$
   각 포인트에서 중심점을 빼서 중심화된(centered) 포인트 집합 $p'_i = p_i - \mu_p$와 $q'_i = q_i - \mu_q$를 구한다.

2. 이 중심화된 포인트들로 3×3 공분산 행렬(covariance matrix) $W$를 계산한다.
   $$
   W = \sum_{i=1}^{N} q'_i (p'_i)^T
   $$
   행렬 $W$에 대해 SVD를 수행한다: $W = U \Sigma V^T$.

3. 최적의 회전 행렬 $R^*$은 $R^* = U V^T$로 계산된다. (단, $\det(R^*) = -1$인 반사(reflection)가 발생하는 특수한 경우, $V$의 마지막 열의 부호를 바꾸어 재계산하는 보정이 필요하다 17.

4. 최적의 이동 벡터 $t^*$는 $t^* = \mu_q - R^* \mu_p$로 계산된다.

이처럼 Point-to-Point ICP는 수학적으로 매우 명쾌하고 계산적으로 효율적이다. 하지만 이는 점과 점 사이의 거리 정보만을 사용하기 때문에, 포인트 클라우드가 나타내는 실제 표면의 기하학적 구조(예: 평면의 방향)는 전혀 고려하지 않는다는 한계가 있다.


Point-to-Plane ICP는 Point-to-Point의 한계를 극복하기 위해 타겟 포인트의 국소적인 표면 정보를 활용한다. 구체적으로, 타겟 포인트 $q_i$에서의 법선 벡터(normal vector) $n_i$를 이용한다.7 이 방식은 변환된 소스 포인트와 타겟 포인트 사이의 오차 벡터 $((R p_i + t) - q_i)$를 타겟 표면의 법선 방향으로 투영(projection)한 값의 제곱의 합을 최소화한다.19 이는 점과 점 사이의 거리가 아닌, 점과 평면 사이의 거리를 최소화하는 것과 유사하다. 목적 함수  $E_{p2pl}(R, t)$는 다음과 같다:
$$
E_{p2pl}(R, t) = \sum_{i=1}^{N} ( ((R p_i + t) - q_i) \cdot n_i )^2
$$
Point-to-Plane 방식은 특히 평면 구조가 많은 인공적인 환경에서 Point-to-Point 방식보다 더 빠른 수렴 속도와 높은 정확도를 보이는 것으로 알려져 있다.7 법선 정보를 활용함으로써, 표면을 따라 미끄러지는 움직임에 대해 더 강한 제약을 제공하기 때문이다. 그러나 이 방식은 몇 가지 단점을 가진다. 첫째, 모든 타겟 포인트에 대해 신뢰할 수 있는 법선 벡터를 추정해야 하며, 이 추정의 정확도가 전체 성능에 큰 영향을 미친다. 둘째, 목적 함수가 회전 행렬 R$에 대해 비선형적이기 때문에 SVD와 같은 해석적 해가 존재하지 않는다. 따라서 회전 각도가 작다고 가정하고 삼각함수를 선형화하는 등의 근사 기법을 사용하여 반복적으로 최적해를 찾아야 한다.

이 두 방식의 선택은 결국 '계산적 우아함'과 '기하학적 충실도' 사이의 근본적인 트레이드오프를 보여준다. Point-to-Point는 해석적 해법의 명쾌함을, Point-to-Plane은 기하학적 정보를 활용한 높은 정확도를 추구한다. 흥미롭게도 KISS-ICP는 의도적으로 더 단순한 Point-to-Point 방식을 채택하고도 SOTA 성능을 달성했는데, 이는 목적 함수의 복잡성을 높이는 대신, 입력 데이터 자체를 정제하고 통계적으로 강건한 최적화 기법을 적용하는 것이 더 효과적일 수 있음을 시사한다. 즉, 복잡한 기하학적 모델을 사용하는 대신, 영리한 데이터 필터링과 강건한 통계 기법을 통해 Point-to-Point라는 단순한 모델이 잘 작동할 수 있는 이상적인 환경을 인위적으로 조성하는 접근법의 승리라고 볼 수 있다.


앞서 설명한 ICP의 기본 원리와 변형들은 강력하지만, 실제 환경에서 사용될 때 다음과 같은 근본적인 한계에 부딪힌다 10:

- **지역 최솟값 (Local Minima):** ICP는 비볼록(non-convex) 최적화 문제이므로, 알고리즘의 성공 여부가 초기 추정값에 매우 크게 의존한다.1 소스와 타겟 포인트 클라우드가 초기에 서로 멀리 떨어져 있거나 회전 오차가 크면, ICP는 잘못된 대응점을 찾게 되고 결국 전역 최솟값이 아닌 엉뚱한 지역 최솟값에 수렴하여 정합에 실패한다.10 이 때문에 ICP는 보통 RANSAC 기반의 전역 정합(global registration) 알고리즘을 통해 대략적인 초기 정렬을 구한 뒤, 이를 미세 조정하는 지역 정합(local registration) 단계에서 사용된다.7
- **이상치 및 부분 중첩 (Outliers and Partial Overlap):** 최소제곱법에 기반한 ICP의 목적 함수는 이상치에 매우 민감하다. 두 포인트 클라우드 간에 실제로 대응점이 존재하지 않는 영역, 예를 들어 센서 노이즈, 스캔 범위의 차이로 인한 부분 중첩, 환경 내의 동적 객체(움직이는 사람이나 차량) 등은 잘못된 대응점을 유발하여 정합 정확도를 심각하게 저해한다.2
- **대응점 탐색의 모호성:** 특징이 없는 넓은 평면이나 반복적인 패턴, 대칭적인 구조를 가진 객체의 경우, 기하학적으로는 전혀 다른 위치의 점들이 가장 가까운 점으로 잘못 매칭될 수 있다.10 이는 잘못된 변환 추정으로 이어진다.

이러한 한계들은 지난 수십 년간 수많은 ICP 변종 연구를 촉발시킨 원동력이었으며, KISS-ICP가 해결하고자 했던 핵심 문제들이기도 하다.


KISS-ICP는 고전적 ICP가 가진 한계를 극복하기 위해 복잡한 모델을 추가하는 대신, 문제의 본질에 집중하고 파이프라인의 각 단계를 영리하게 재설계하는 접근법을 취한다. 이 장에서는 KISS-ICP의 핵심 철학을 살펴보고, 그 성공을 뒷받침하는 네 가지 핵심 구성요소를 심층적으로 분석한다.


KISS-ICP의 가장 중요한 특징은 그 이름에 담긴 철학, 즉 "Keep It Small and Simple" (작고 단순하게 유지하라)에 있다.12 이는 LiDAR Odometry 및 SLAM 시스템이 점차 복잡해지는 추세에 대한 명백한 반기이다. 연구팀은 기존 시스템들이 성능을 높이기 위해 추가했던 정교한 특징 추출기, 별도의 루프 클로저 모듈, IMU나 GPS와 같은 외부 센서와의 융합 등을 의도적으로 배제했다.12

이러한 '빼기의 혁신'을 통해 달성하고자 한 목표는 명확하다. 바로 최소한의 파라미터 튜닝으로 다양한 조건에서 강건하게 작동하는 범용적인 시스템을 구축하는 것이다.12 KISS-ICP는 동일한 파라미터 설정 하나만으로 고속도로를 주행하는 자율주행차, 복잡한 3차원 기동을 하는 드론(UAV), 사람이 직접 들고 다니는 핸드헬드 스캐너 등 전혀 다른 환경과 센서, 이동 프로파일을 가진 플랫폼에 적용될 수 있음을 실험적으로 증명했다.13 이는 특정 시나리오에 과적합되지 않는 높은 일반화 성능을 의미하며, 실제 로보틱스 응용에서 매우 중요한 가치를 지닌다.


KISS-ICP의 놀라운 성능과 강건성은 독립적으로 작동하는 네 가지 핵심 요소의 유기적인 결합, 즉 시너지 효과에 기반한다.15 이 네 가지 요소는 LiDAR Odometry가 실패하는 주요 원인들을 계층적으로 방어하는 '다층 방어 시스템(defense-in-depth)'처럼 작동한다. 각 단계는 다음 단계를 위한 데이터 정제 과정이며, 이들의 조합이 최종적으로 단순한 Point-to-Point ICP가 성공적으로 작동할 수 있는 토대를 마련한다.


LiDAR 센서는 한 번에 모든 방향의 데이터를 얻는 것이 아니라, 일정 시간 동안 회전하며 순차적으로 포인트를 획득한다. 이 스캔 시간 동안 로봇이나 차량이 움직이면, 포인트 클라우드는 실제 환경의 모습과 다르게 왜곡되는 '모션 왜곡(motion distortion)' 현상이 발생한다.20 이는 정합 오류의 가장 근본적인 원인 중 하나이다.

많은 시스템들이 이 문제를 해결하기 위해 고주파수 데이터를 제공하는 IMU 센서를 융합하여 각 포인트가 측정된 시점의 정확한 자세를 보정한다. 하지만 KISS-ICP는 '단순성' 원칙에 따라 IMU와 같은 추가 센서에 의존하지 않는다.11 대신, 오직 이전에 추정된 LiDAR Odometry 포즈 정보만을 사용하여 로봇의 움직임을 예측한다. 구체적으로, 최근 몇 개의 포즈로부터 등속 모델(constant velocity model)을 가정하여 현재 프레임에서의 각속도와 선속도를 추정한다.20 그리고 이 예측된 속도를 이용해 스캔이 진행되는 동안 각 포인트의 위치를 보정하여 왜곡을 제거(de-skewing)한다.20 이는 시스템의 복잡성과 의존성을 낮추면서도 데이터의 물리적 무결성을 확보하는 첫 번째 방어선 역할을 한다.


모션 왜곡이 보정된 포인트 클라우드는 여전히 수만에서 수십만 개의 점으로 구성되어 있어, 이를 직접 정합에 사용하는 것은 계산적으로 매우 비효율적이다. KISS-ICP는 계산 효율성과 정보량 보존 사이의 균형을 맞추기 위해 독특한 계층적 복셀 다운샘플링 전략을 사용한다.20

1. **맵 업데이트용 다운샘플링:** 먼저, 상대적으로 조밀한 복셀 크기(예: 논문에서는 0.5m)를 사용하여 원본 포인트 클라우드를 다운샘플링한다. 이 과정에서 생성된 포인트들은 정합이 완료된 후, 다음 프레임의 타겟이 될 지역 맵(local map)을 업데이트하는 데 사용된다. 이는 맵의 세부 정보를 최대한 보존하기 위함이다.
2. **정합용 다운샘플링:** 그 다음, 위에서 생성된 조밀한 포인트 클라우드를 대상으로 더 큰 복셀 크기(예: 논문에서는 1.0m)를 사용하여 한 번 더 다운샘플링한다. 이 희소한 포인트 클라우드가 실제 ICP 정합 과정의 소스 포인트로 사용된다.20

이러한 계층적 접근은 ICP의 계산량을 극적으로 줄여 실시간 성능을 보장하는 동시에, 맵 자체는 상대적으로 풍부한 정보를 유지할 수 있도록 하는 영리한 설계다. 이는 계산 복잡도라는 현실적인 제약을 관리하는 두 번째 방어선이다.


대부분의 ICP 변종들은 대응점을 찾을 때 고정된 최대 거리 임계값(max correspondence distance)을 사용한다. 이 임계값보다 멀리 있는 점은 대응점 후보에서 제외된다. 하지만 이 값은 환경이나 로봇의 속도에 따라 최적값이 달라지기 때문에 튜닝이 어렵다.

KISS-ICP는 이 문제를 해결하기 위해 고정된 값이 아닌 '적응형 임계값'을 사용한다.12 이 임계값은 1단계에서 사용된 등속 모델을 기반으로 동적으로 결정된다.20 즉, 로봇의 이전 움직임을 바탕으로 현재 프레임에서 예상되는 최대 변위(displacement)를 계산하고, 이를 기반으로 대응점 탐색 범위를 설정하는 것이다. 예를 들어, 로봇이 빠르게 움직이고 있을 때는 임계값을 크게 설정하여 더 넓은 영역에서 대응점을 찾고, 거의 정지 상태일 때는 임계값을 작게 설정하여 불필요한 탐색을 줄이고 잘못된 매칭을 방지한다.

이 운동학적(kinematic) 정보에 기반한 필터링은 물리적으로 불가능하거나 비현실적인 대응점들을 정합 초기 단계에서부터 효과적으로 배제한다. 이는 고정 임계값 방식보다 훨씬 지능적인 접근법으로, 탐색 공간을 크게 줄이고 정합의 강건성을 향상시키는 세 번째 방어선 역할을 수행한다.


위의 세 단계를 거치면서 데이터는 상당히 정제되지만, 여전히 예측 불가능한 이상치들이 남아있을 수 있다. 예를 들어, 갑자기 나타난 동적 객체(다른 차량, 보행자)나 센서 노이즈 등이 이에 해당한다. 표준 최소제곱법 기반의 ICP는 소수의 큰 오차를 가진 이상치에 의해 전체 정합 결과가 크게 왜곡될 수 있다.

KISS-ICP는 이러한 문제에 대한 최종 방어선으로 '강건한 커널(robust kernel)'을 최적화 과정에 도입한다.12 강건한 커널은 최소제곱법의 목적 함수를 수정하여, 특정 임계값을 초과하는 큰 오차(residual)를 가진 대응점의 영향력을 수학적으로 줄이는 역할을 한다. 즉, 오차가 작은 '좋은' 대응점(inlier)에는 거의 영향을 주지 않지만, 오차가 큰 '나쁜' 대응점(outlier)에는 낮은 가중치를 부여하여 전체 비용 함수에 미치는 영향을 억제한다. 이를 통해 소수의 이상치가 전체 정합 결과를 좌우하는 것을 방지하고 통계적 강건성을 확보한다.

결론적으로, KISS-ICP의 네 가지 핵심 요소는 순차적으로 데이터의 품질을 높이고 문제의 난이도를 낮춘다. 모션 보상으로 물리적 오류를 잡고, 다운샘플링으로 계산량을 줄이며, 적응형 임계값으로 비현실적 대응점을 걸러낸 뒤, 강건한 커널로 남은 이상치를 억제한다. 이처럼 잘 정제된 문제로 변환되었기 때문에, 최종적으로는 가장 단순한 Point-to-Point ICP만으로도 충분히 정확하고 강건한 해를 구할 수 있게 되는 것이다.


KISS-ICP의 철학과 구성요소를 이해했다면, 이제 이를 뒷받침하는 수학적 모델과 최적화 과정을 구체적으로 살펴볼 차례이다. KISS-ICP는 고전적인 Point-to-Point ICP의 틀을 유지하면서도, 강건한 통계 기법을 영리하게 통합하여 성능을 극대화한다.


KISS-ICP의 목표는 LiDAR Odometry 문제, 즉 연속적인 LiDAR 스캔을 통해 센서의 자세(pose) 변화를 추정하는 것이다. 시간 $t$에서 새로 들어온 LiDAR 스캔(소스 포인트 클라우드)을 $S_t = \{s_i | s_i \in \mathbb{R}^3\}$라 하자. 이 소스 클라우드는 이전 시간 $t-1$까지의 스캔들이 정합되어 누적된 지역 맵(local map, 타겟 포인트 클라우드) $Q_{t-1} = \{q_j | q_j \in \mathbb{R}^3\}$에 정합되어야 한다.21

이때 우리가 찾고자 하는 것은 소스 클라우드 $S_t$를 지역 맵 $Q_{t-1}$에 최적으로 정렬하는 강체 변환(rigid transformation) $T_t \in SE(3)$이다. 이 변환은 3차원 회전을 나타내는 회전 행렬 $R_t \in SO(3)$와 3차원 이동을 나타내는 변환 벡터 $t_t \in \mathbb{R}^3$로 구성된다. 소스 클라우드의 임의의 점 $s_i$는 이 변환에 의해 다음과 같이 변환된다:
$$
s'_i = T_t s_i = R_t s_i + t_t
$$




표준 Point-to-Point ICP가 유클리드 거리의 제곱합을 최소화하는 것과 달리, KISS-ICP는 이상치의 영향을 줄이기 위해 강건한 커널(robust kernel) 함수 $\rho(\cdot)$를 포함하는 비용 함수를 정의한다.12

먼저, 대응점 탐색 단계에서 소스 포인트 $s_i$에 대한 타겟 맵의 최근접 이웃점 $q_i$를 찾는다. 이렇게 형성된 대응점 쌍 $(s_i, q_i)$의 집합을 $C$라고 할 때, 변환 $T_t$에 대한 잔차(residual) 벡터 $r_i(T_t)$는 다음과 같이 정의된다 21:
$$
r_i(T_t) = T_t s_i - q_i
$$
KISS-ICP의 전체 비용 함수(cost function) $\chi(T_t)$는 이 잔차 벡터의 제곱 노름(squared norm)에 강건한 커널 $\rho$를 적용한 값들의 총합으로 정의된다:
$$
\chi(T_t) = \sum_{(s_i, q_i) \in C} \rho(\|r_i(T_t)\|^2) = \sum_{(s_i, q_i) \in C} \rho(\|T_t s_i - q_i\|^2)
$$
여기서 $\rho(x)$는 M-추정(M-estimator)에 사용되는 강건한 커널 함수이다. 다양한 커널 함수가 사용될 수 있으며, 예를 들어 Tukey biweight 커널은 특정 임계값 $c$를 기준으로 정의된다. 잔차의 제곱 노름 $x = \|r_i\|^2$에 대해 Tukey 커널은 다음과 같이 표현될 수 있다:
$$
\rho(x, c^2) =
\begin{cases}
\frac{c^2}{6} \left(1 - \left(1 - \frac{x}{c^2}\right)^3\right) & \text{if } x \le c^2 \\
\frac{c^2}{6} & \text{if } x > c^2
\end{cases}
$$
이 함수는 잔차의 크기가 임계값 $c$ 이하일 때는 이차 함수와 유사하게 행동하지만, 임계값을 초과하면 비용 증가가 멈추거나 둔화된다. 이로 인해 임계값을 크게 벗어나는 이상치들이 전체 비용 함수에 미치는 영향력이 효과적으로 제한된다.


강건한 커널이 포함된 비용 함수 $\chi(T_t)$는 비선형적(non-linear)이며 비볼록(non-convex)이기 때문에, 표준 Point-to-Point ICP처럼 SVD를 통해 한 번에 해석적 해를 구할 수 없다. KISS-ICP는 이 문제를 풀기 위해 반복적 재가중 최소제곱(IRLS)이라는 효율적인 최적화 기법을 사용한다.

IRLS의 핵심 아이디어는 각 반복 단계에서 비선형적인 강건한 최적화 문제를 선형적인 가중 최소제곱(Weighted Least Squares, WLS) 문제로 근사하여 푸는 것이다. 이는 각 대응점 쌍 $(s_i, q_i)$에 잔차의 크기에 따라 달라지는 가중치 $w_i$를 부여함으로써 달성된다. 가중치 $w_i$는 비용 함수를 잔차의 제곱 노름 $x_i = \|r_i\|^2$로 미분한 값, 즉 $\rho'(x_i)$로 정의된다. Tukey 커널의 경우, 가중 함수는 다음과 같다:
$$
w(x_i, c^2) = \rho'(x_i, c^2) =
\begin{cases}
\left(1 - \frac{x_i}{c^2}\right)^2 & \text{if } x_i \le c^2 \\
0 & \text{if } x_i > c^2
\end{cases}
$$
이 가중 함수는 잔차가 클수록 가중치가 0에 가까워지는 특성을 보인다. IRLS 알고리즘은 다음과 같은 절차를 따른다:

1. **초기화:** 변환 $T_0$를 초기 추정값(예: 등속 모델 예측값)으로 설정한다.

2. 반복 ($k=1, 2,...$):

   a.  가중치 계산: 현재 변환 추정치 $T_{k-1}$을 사용하여 모든 대응점에 대한 잔차 $r_i(T_{k-1})$와 그 제곱 노름 $x_i$를 계산한다. 그리고 이를 이용해 각 대응점의 가중치 $w_i^{(k-1)} = w(x_i, c^2)$를 계산한다.

   b.  가중 최소제곱 문제 풀이: 다음의 가중 최소제곱(WLS) 문제를 풀어 증분 변환(incremental transform) $\Delta T_k$를 찾는다.
   $$
   \Delta T_k = \underset{\Delta T}{\arg\min} \sum_{i} w_i^{(k-1)} \| (\Delta T \cdot T_{k-1}) s_i - q_i \|^2
   $$
   이 WLS 문제는 각 대응점에 가중치 $w_i$를 적용한 것을 제외하면 표준 Point-to-Point ICP 문제와 동일한 형태를 가지므로, 가중치가 적용된 SVD 해법을 통해 효율적으로 풀 수 있다.

   c.  변환 업데이트: 현재 변환 추정치를 업데이트한다: $T_k = \Delta T_k \cdot T_{k-1}$.

   d.  수렴 확인: 변환 $T_k$가 수렴 조건(예: $\Delta T_k$가 거의 단위 행렬에 가까워짐)을 만족하면 반복을 종료한다.

이 최적화 과정은 기존의 검증된 SVD 기반 해법의 구조를 대부분 유지하면서, IRLS라는 영리한 '포장'을 통해 강건성을 주입하는 방식이다. 이는 알고리즘의 핵심 계산부(SVD)를 재활용하여 계산 효율성을 높이는 동시에, 반복적인 가중치 재계산을 통해 이상치에 대한 저항력을 극대화하는 매우 실용적이고 효과적인 설계라 할 수 있다.


KISS-ICP의 기술적 위치를 명확히 하기 위해서는, 포인트 클라우드 정합 분야의 다른 주요 알고리즘들과의 비교 분석이 필수적이다. 본 장에서는 KISS-ICP를 Generalized-ICP(G-ICP) 및 Normal Distributions Transform(NDT)과 비교하여 각 알고리즘의 철학, 성능 특성, 그리고 적합한 사용 시나리오를 심층적으로 분석한다.


세 알고리즘은 동일한 '정합' 문제를 풀지만, 그 접근 방식과 철학에서 근본적인 차이를 보인다.

- **KISS-ICP (강건한 점-대-점, Robust Point-to-Point):** 이 알고리즘의 핵심 철학은 '모델의 단순성'과 '데이터의 정교함'이다. 가장 단순한 Point-to-Point 오차 척도, 즉 유클리드 거리를 사용하되, 그 약점을 보완하기 위해 파이프라인 전반에 걸쳐 데이터를 지능적으로 정제(모션 보상, 부표본화, 적응형 임계값)하고, 최종 최적화 단계에서 강건한 통계 기법(커널)을 적용한다.12 이는 '복잡한 모델' 대신 '정제된 데이터'를 통해 문제를 해결하려는 접근법이다.

- **G-ICP (확률론적 평면-대-평면, Probabilistic Plane-to-Plane):** G-ICP의 철학은 '기하학적 구조의 명시적 모델링'에 있다. 포인트 클라우드의 점들이 실제로는 어떤 표면에서 샘플링된 것이라는 사실에 착안하여, 각 점 주변의 국소적인 표면 구조를 확률적으로 모델링한다.19 구체적으로, 각 점에 대해 법선 방향으로는 작은 분산(uncertainty)을, 접평면 방향으로는 큰 분산을 갖는 공분산 행렬을 추정한다. 정합 오차는 두 대응점의 확률 분포(가우시안) 사이의 마할라노비스 거리(Mahalanobis distance)로 정의되며, 이는 사실상 '평면 대 평면'의 거리를 측정하는 것과 같다.19 이는 '모델 자체를 정교하게 만들어' 기하학적 정보를 적극적으로 활용하는 접근법이다. 목적 함수는 다음과 같이 표현된다 19:
  $$
  T = \underset{T}{\arg\min} \sum_{i} d(T)_i^T (C_{B_i} + T C_{A_i} T^T)^{-1} d(T)_i
  $$
  여기서 $d(T)_i = b_i - T a_i$이고, $C_{A_i}$와 $C_{B_i}$는 각각 점 $a_i$와 $b_i$의 공분산 행렬이다.

- **NDT (통계적 분포 변환, Normal Distributions Transform):** NDT는 가장 급진적인 접근법으로, '명시적인 대응점 탐색의 폐지'를 추구한다. 타겟 포인트 클라우드를 직접 사용하는 대신, 공간을 일정한 크기의 복셀(voxel)로 나누고 각 복셀에 포함된 점들의 통계적 분포(평균과 공분산)를 계산하여 하나의 정규분포(가우시안)로 표현한다.23 정합 과정은 소스 포인트 클라우드의 점들을 변환시켰을 때, 이들이 타겟 공간의 확률 밀도 함수(PDF) 상에서 가장 높은 확률(우도, likelihood)을 갖도록 하는 변환을 찾는 것이다.23 이는 개별 점들의 대응 관계가 아닌, 전체적인 분포의 유사성을 최적화하는 방식이다. 목적 함수는 변환된 소스 포인트들의 음의 로그 우도(negative log-likelihood) 합을 최소화하는 것이다 23:
  $$
  \underset{R, t}{\arg\min} \left\{ - \sum_{i} \text{NDT}(f_{R,t}(x_i)) \right\}
  $$

- 


각기 다른 철학은 실제 성능에서도 뚜렷한 차이로 나타난다.

- **속도:** 일반적으로 NDT가 가장 빠른 성능을 보인다. 대응점 탐색 과정이 없고, 최적화 과정이 병렬화에 매우 용이하기 때문이다. 특히 OpenMP와 같은 병렬 컴퓨팅 라이브러리를 활용하면 NDT의 속도는 극적으로 향상될 수 있다.25 KISS-ICP는 효율적인 다운샘플링과 SVD 기반 최적화 덕분에 매우 빠른 실시간 성능을 보인다.12 G-ICP는 각 점에 대한 공분산 행렬 계산과 비선형 최적화 과정으로 인해 세 알고리즘 중에서는 상대적으로 계산 비용이 높은 경향이 있다.25
- **정확도:** 정확도는 환경의 특성에 따라 달라진다. G-ICP는 빌딩 내부, 복도, 도심 등 인공적인 평면 구조가 풍부한 환경에서 표면 정보를 적극적으로 활용하므로 매우 높은 정확도를 보인다.22 실제로 특징이 없는 복도와 같은 환경에서 Point-to-Point 기반인 KISS-ICP가 실패하는 반면, G-ICP와 유사한 접근법은 성공할 수 있다는 연구 결과도 있다.26 NDT는 복셀 크기와 같은 파라미터가 환경에 맞게 잘 설정되고 초기 추정값이 좋을 경우 높은 정확도를 달성할 수 있다.25 KISS-ICP는 특정 구조에 의존하지 않기 때문에 다양한 환경에서 전반적으로 우수하고 안정적인 정확도를 제공하는 것을 목표로 한다.12
- **수렴성 및 강건성:** 수렴 범위(Valley of Convergence), 즉 얼마나 큰 초기 오차에서부터 정합에 성공할 수 있는지를 보면, NDT가 표준 ICP보다 더 넓은 수렴 범위를 갖는 것으로 알려져 있다.27 G-ICP는 확률적 모델링 덕분에 이상치에 대해 자연스러운 강건성을 가진다.19 KISS-ICP의 가장 큰 장점 중 하나는 바로 강건성이다. 적응형 임계값과 강건한 커널의 조합은 동적 객체가 많은 실제 주행 환경과 같은 시나리오에서 이상치의 영향을 효과적으로 억제한다. 또한, 파라미터 튜닝에 대한 민감도가 낮아 다양한 조건에서 안정적으로 작동하는 범용성을 확보했다.12


세 알고리즘의 특성을 종합하여 장단점과 추천 사용 시나리오를 정리하면 다음과 같다.

- **KISS-ICP:**
  - **장점:** 구현이 상대적으로 단순하고, 속도가 빠르며, 파라미터 튜닝이 거의 필요 없어 다양한 환경과 센서에 대한 범용성이 매우 높다. 특히 IMU와 같은 추가 센서 없이 LiDAR 데이터만으로 작동한다는 점은 큰 장점이다.12
  - **단점:** 기하학적 구조 정보를 명시적으로 사용하지 않으므로, 긴 터널이나 복도처럼 특징이 없는 환경에서는 성능 저하가 발생할 수 있다.26 또한, 설계상 다운샘플링에 의존하므로 빔(beam) 수가 적은 희소한 LiDAR 데이터에는 취약할 수 있다.28
  - **추천 시나리오:** 다양한 로봇 플랫폼(지상, 항공)을 위한 범용 LiDAR Odometry 프론트엔드, 빠른 프로토타이핑, IMU가 없거나 사용하기 어려운 시스템.
- **G-ICP:**
  - **장점:** 평면 구조가 풍부한 환경(예: 실내, 빌딩)에서 매우 높은 정합 정확도를 제공한다. 확률적 모델링은 이상치에 대한 내재적 강건성을 부여한다.19
  - **단점:** 모든 점에 대한 법선 벡터와 공분산 행렬을 계산해야 하므로 계산 복잡도가 높다. 또한, 법선 벡터 추정의 정확도에 성능이 크게 좌우된다.22
  - **추천 시나리오:** 고정밀 3D 모델링, 건축 및 측량, 구조화된 환경에서의 고정밀 SLAM.
- **NDT:**
  - **장점:** 대응점 탐색 과정이 없어 매우 빠르며, 병렬화에 용이하여 대규모 포인트 클라우드 처리에 적합하다.23
  - **단점:** 초기 추정값에 민감하여 수렴 범위가 제한될 수 있다. 성능이 복셀 크기라는 핵심 파라미터에 크게 의존하며, 공간의 이산화(discretization)로 인한 오류가 발생할 수 있다.23
  - **추천 시나리오:** 실시간성이 매우 중요한 응용, 대규모 포인트 클라우드의 빠른 거친 정렬(coarse registration), 다른 정합 알고리즘의 초기 추정값을 제공하는 용도.


세 알고리즘의 주요 특징을 요약하면 다음 표와 같다. 이 표는 각 알고리즘의 핵심 원리와 장단점을 한눈에 비교하여, 특정 응용에 가장 적합한 알고리즘을 선택하는 데 도움을 줄 수 있다.

| 특성            | KISS-ICP                                | G-ICP                                                | NDT                                                   |
| --------------- | --------------------------------------- | ---------------------------------------------------- | ----------------------------------------------------- |
| **핵심 원리**   | 강건한 점-대-점 (Robust Point-to-Point) | 확률론적 평면-대-평면 (Probabilistic Plane-to-Plane) | 통계적 정규분포 변환 (Normal Distributions Transform) |
| **오차 척도**   | 유클리드 거리 (강건 커널 적용)          | 마할라노비스 거리 (공분산 기반)                      | 확률 분포의 우도(Likelihood)                          |
| **대응점 탐색** | KD-Tree 기반 최근접 이웃 탐색           | KD-Tree 기반 최근접 이웃 탐색                        | 불필요 (암시적)                                       |
| **이상치 처리** | 강건한 커널, 적응형 임계값              | 확률적 가중치 (암시적)                               | 복셀 내 통계 처리 (간접적)                            |
| **주요 강점**   | 단순성, 속도, 범용성, 파라미터 둔감성   | 구조적 환경에서 높은 정확도                          | 매우 빠른 속도, 병렬화 용이                           |
| **주요 약점**   | 구조 정보 미활용, 희소 데이터에 취약    | 계산 복잡도, 법선 벡터 추정 의존                     | 초기 추정 민감, 복셀 크기 의존                        |
| **센서 의존성** | 낮음 (IMU 불필요)                       | 중간 (법선 추정을 위한 밀도 필요)                    | 중간 (복셀화를 위한 밀도 필요)                        |


알고리즘의 철학과 이론적 분석만으로는 그 실제 가치를 온전히 평가할 수 없다. 이 장에서는 표준화된 벤치마크 데이터셋을 기반으로 KISS-ICP의 성능을 정량적, 정성적으로 분석하고, 그 결과가 시사하는 바를 심도 있게 고찰한다.


자율주행 및 로보틱스 연구에서 알고리즘의 성능을 객관적으로 비교 평가하기 위해 가장 널리 사용되는 데이터셋 중 하나는 KITTI Odometry 벤치마크이다.29 독일 카를스루에 시내와 주변 고속도로 등 다양한 실제 환경에서 수집된 이 데이터셋은 고해상도 Velodyne LiDAR 데이터, 스테레오 카메라 이미지, 그리고 고정밀 GPS/IMU 기반의 참값(ground truth) 궤적 정보를 제공한다.29

KITTI Odometry 벤치마크는 총 22개의 시퀀스로 구성되며, 이 중 11개(00-10)는 참값 궤적이 함께 제공되어 알고리즘 학습 및 개발에 사용되고, 나머지 11개(11-21)는 평가를 위해 참값이 공개되지 않은 채로 제공된다. 연구자들은 자신의 알고리즘을 평가용 시퀀스에 적용하여 얻은 궤적을 제출하고, 공식 서버에서 평가를 받는다. 평가는 동일한 파라미터 세트를 모든 시퀀스에 적용해야 한다는 엄격한 규칙 하에 이루어지므로, 알고리즘의 범용성과 강건성을 측정하는 좋은 척도가 된다.29


KISS-ICP 논문과 KITTI 공식 리더보드에 따르면, KISS-ICP는 순수 Odometry 시스템임에도 불구하고 매우 뛰어난 성능을 기록했다. 주요 평가지표는 주로 경로의 일부 구간에 대한 상대적인 자세 오차(Relative Pose Error, RPE)이며, 이는 평균 상대 평 이동 오차(average relative translational error, %)와 평균 상대 회전 오차(average relative rotational error, deg/m)로 나뉜다.29

KISS-ICP의 성능은 루프 클로저(loop closure)와 전역 최적화(global optimization)와 같은 백엔드 모듈을 포함하는 완전한 SLAM 시스템들과 비교될 때 그 가치가 더욱 명확해진다. 다음 표는 KITTI Odometry 벤치마크(전체 시퀀스 00-21)에서 KISS-ICP와 주요 경쟁 알고리즘들의 성능을 비교한 것이다.

| 방법 (유형)            | 코드 공개 | 상대 평 이동 오차 (%) ↓ | 상대 회전 오차 (deg/m) ↓ |            |
| ---------------------- | --------- | ----------------------- | ------------------------ | ---------- |
| CT-ICP (SLAM) 21       | Yes       | 0.59                    | 0.0016                   |            |
| KISS-ICP (Odometry) 12 |           | **Yes**                 | **0.61**                 | **0.0017** |
| MOLA-LO (Odometry) 29  | Yes       | 0.62                    | 0.0017                   |            |
| SiMpLE (Odometry) 29   | Yes       | 0.62                    | 0.0015                   |            |
| PIN-SLAM (SLAM) 29     | Yes       | 0.64                    | 0.0015                   |            |

(Note: 수치는 29의 2024년 9월 기준 공식 리더보드 데이터를 반영함)

이 표가 보여주는 결과는 매우 시사하는 바가 크다. 첫째, KISS-ICP는 루프 클로저 없이 순수하게 프레임 간 정합만으로 누적 오차를 계산하는 Odometry 시스템임에도 불구하고, CT-ICP나 PIN-SLAM과 같은 완전한 SLAM 시스템들과 대등한 수준의 정확도를 달성했다. 이는 KISS-ICP가 프레임 간 정합(scan-to-scan 또는 scan-to-map)에서 발생하는 오차를 극도로 낮추었음을 의미한다.29

이러한 결과는 SLAM 시스템 설계에 대한 전통적인 관점에 중요한 질문을 던진다. 전통적인 SLAM은 필연적으로 발생하는 Odometry의 오차 누적(drift)을 루프 클로저를 통해 주기적으로 보정하는 'Odometry + Loop Closure' 구조를 기본으로 한다.8 하지만 KISS-ICP의 성공은 프론트엔드(Odometry)의 정확도가 충분히 높다면, 백엔드(Loop Closure)의 역할이 줄어들거나 심지어 특정 응용에서는 불필요할 수도 있음을 보여준다. 즉, '매우 정확한 프론트엔드'는 그 자체로 강력한 솔루션이 될 수 있으며, 이는 SLAM 시스템 설계의 무게 중심을 다시 프론트엔드의 정밀도를 높이는 방향으로 이동시킬 수 있는 중요한 발견이다. 많은 로보틱스 응용(예: 단거리 자율 배송, 공장 내 물류)에서는 전역 일관성(global consistency)보다 국소적인 정확성과 실시간성이 더 중요할 수 있다. 이런 경우, 복잡하고 계산 비용이 높은 SLAM 백엔드를 제거하고 KISS-ICP와 같은 고정밀 Odometry만 사용하는 것이 훨씬 효율적이고 실용적인 대안이 될 수 있다.


정량적인 수치 외에도 KISS-ICP가 생성하는 맵의 품질을 시각적으로 평가하는 것 또한 중요하다. 공개된 데모 영상과 논문의 결과들을 보면, KISS-ICP는 다양한 환경에서 매우 일관성 있고 깔끔한 포인트 클라우드 맵을 실시간으로 생성하는 것을 확인할 수 있다.15 고속도로, 복잡한 도심, 심지어 드론이나 핸드헬드 기기로 촬영된 비정형적인 데이터에서도 구조물들이 명확하게 구분되고 경로가 안정적으로 추정된다.

물론 실패 사례도 존재한다. 예를 들어, 시스템이 처음 시작될 때는 이전 움직임에 대한 정보가 없어 모션 예측이 불가능하므로, 초기 몇 프레임 동안은 잘못된 정렬이 발생할 수 있다. Waymo 데이터셋을 사용한 한 실험에서는 초기 5프레임 동안 정렬이 어긋나는 현상이 관찰되었으나, 이는 초기 임계값을 조정하거나 더 긴 시퀀스를 실행함으로써 알고리즘이 스스로 회복하는 모습을 보였다.32 이러한 동적인 회복 능력은 KISS-ICP의 강건성을 보여주는 또 다른 측면이다.


KISS-ICP는 그 철학과 성능 덕분에 학계를 넘어 산업계에서도 큰 주목을 받고 있다. 이 장에서는 KISS-ICP의 광범위한 응용 분야를 살펴보고, 동시에 명확한 기술적 한계와 실패 사례를 분석한다. 마지막으로, KISS 철학을 계승하고 확장하는 미래 연구 방향을 통해 포인트 클라우드 정합 기술의 전망을 조망한다.


KISS-ICP의 가장 큰 강점은 특정 플랫폼이나 환경에 국한되지 않는 뛰어난 범용성이다. 최소한의 파라미터 튜닝으로 다양한 시나리오에 즉시 적용할 수 있다는 점은 실제 로보틱스 제품 및 서비스 개발에 있어 매우 매력적인 특징이다.13

- **자율주행차:** KISS-ICP는 동적 객체가 많고 고속으로 주행하는 고속도로나 도심 환경에서도 강건한 주행 거리 측정을 제공한다. IMU 데이터 없이 순수 LiDAR만으로도 높은 정확도를 달성할 수 있어, 센서 고장 시 백업 시스템이나 경량화된 Odometry 솔루션으로 활용될 수 있다.13
- **무인 항공기 (UAV) 및 지상 로봇:** 드론과 같이 복잡한 3차원 움직임을 보이는 플랫폼이나, 세그웨이처럼 독특한 이동 프로파일을 가진 지상 로봇에도 동일한 파라미터로 적용 가능하다.13 이는 다양한 로봇을 개발하는 연구소나 기업에서 플랫폼마다 Odometry 시스템을 새로 튜닝해야 하는 부담을 크게 줄여준다.
- **핸드헬드 매핑 및 3D 재구성:** 사람이 직접 들고 다니며 실내외 공간을 스캔하는 핸드헬드 매핑 장비에도 KISS-ICP는 효과적이다. 실시간으로 정합 결과를 확인하며 고품질의 3D 포인트 클라우드 맵을 생성할 수 있어, 건축, 측량, 가상현실 콘텐츠 제작 등 다양한 분야에 응용될 수 있다.13

이러한 광범위한 적용 가능성은 KISS-ICP가 로보틱스 분야의 새로운 '표준 Odometry 베이스라인'이 될 수 있는 강력한 잠재력을 가지고 있음을 보여준다.15


모든 기술이 그러하듯, KISS-ICP 역시 만능이 아니며 명확한 한계와 실패 사례가 존재한다. 이러한 한계는 대부분 KISS-ICP의 '단순성'이라는 설계 철학에서 비롯된 필연적인 결과이다.

- **희소한 포인트 클라우드 (Sparse Data):** KISS-ICP는 정합 전에 다운샘플링을 수행하여 계산 효율성을 높인다. 이 전략은 Velodyne VLP-64와 같이 조밀한 포인트 클라우드에서는 효과적이지만, Ouster OS0-32와 같이 빔(beam) 수가 적어 원본 데이터 자체가 희소한 LiDAR 센서에서는 오히려 독이 될 수 있다.28 이미 부족한 정보를 더욱 희소하게 만들어 정합에 필요한 기하학적 특징을 잃게 만들기 때문이다. GitHub 이슈 포럼에서는 이러한 센서를 사용하는 경우, 다운샘플링 단계를 비활성화하는 등의 파라미터 조정이 필요하다는 논의가 있었다.28
- **기하학적으로 모호한 환경 (Geometrically Ambiguous Environments):** KISS-ICP는 Point-to-Point 오차 척도를 사용하므로, 기하학적 구조 정보를 명시적으로 활용하지 않는다. 이로 인해 긴 복도, 터널, 넓은 평원과 같이 한 방향으로 특징이 거의 없는 환경에서는 해당 축 방향의 이동(translation)을 정확히 추정하기 어렵다.26 이는 'aperture problem'과 유사한 현상으로, 알고리즘이 표면을 따라 미끄러지는 움직임을 제대로 제약하지 못하기 때문에 발생한다. 이 문제는 GenZ-ICP와 같은 후속 연구에서 Point-to-Plane과 같은 다른 오차 척도를 결합하여 해결하려는 시도의 동기가 되었다.26
- **특이한 움직임 (Unusual Motions):** KISS-ICP의 모션 보상 및 적응형 임계값은 등속 모델(constant velocity model)에 기반한다. 이는 일반적인 지상 로봇의 움직임에는 잘 맞지만, 급격한 가감속이나 순수한 Z축 방향의 수직 이동, 큰 Pitch/Roll 회전과 같이 등속 가정이 깨지는 특이한 움직임에서는 정합에 실패할 수 있다.28
- **초기화 문제 (Initialization Problem):** 시스템이 처음 시작되는 몇 프레임 동안은 이전 포즈 정보가 없어 모션 예측이 불가능하다. 이때는 변환 행렬을 단위 행렬(identity matrix)로 초기화하는데, 만약 초기 움직임이 크다면 잘못된 지역 최솟값에 수렴할 위험이 있다.32 이 문제는 보통 몇 프레임이 지나면 알고리즘이 스스로 회복하지만, 초기 정확도가 중요한 응용에서는 문제가 될 수 있다.


KISS-ICP의 성공은 그 자체로도 의미가 크지만, 더 중요한 것은 '단순함의 힘'이라는 철학을 증명하고 이를 바탕으로 한 풍부한 후속 연구의 길을 열었다는 점이다. KISS-ICP의 한계는 그 설계 철학의 필연적인 결과이며, 이를 극복하려는 후속 연구들은 '원칙 있는 복잡성의 재도입'이라는 흥미로운 패턴을 보인다. 이는 무분별하게 기능을 추가하던 과거와 달리, KISS-ICP라는 견고하고 검증된 기반 위에 필요한 모듈을 선별적으로, 그리고 체계적으로 쌓아 올리는 성숙한 개발 방향을 제시한다.

- **KISS-SLAM:** KISS-ICP가 가진 가장 명백한 한계는 누적 오차(drift)를 보정할 수 있는 백엔드가 없다는 점이다. KISS-SLAM은 이 문제를 해결하기 위해 KISS-ICP를 매우 정확한 프론트엔드로 사용하고, 여기에 효율적인 장소 인식(place recognition) 모듈과 포즈 그래프 최적화(pose graph optimization) 백엔드를 결합한 완전한 SLAM 시스템이다.21 이는 KISS-ICP의 강건한 Odometry 성능을 유지하면서 전역적으로 일관된 맵을 생성할 수 있게 한다.
- **KISS-Matcher:** KISS 철학을 Odometry 문제가 아닌 전역 정합(global registration) 문제로 확장한 연구이다. 전역 정합은 초기 추정값 없이, 서로 멀리 떨어져 있는 두 포인트 클라우드를 정합하는 더 어려운 문제이다. KISS-Matcher는 확장성(scalability)에 중점을 두어, 대규모 맵 레벨의 정합까지 효율적으로 처리하는 것을 목표로 한다.34
- **Semantic-KISS:** 정합 과정에 기하학적 정보뿐만 아니라 의미론적 정보(semantic information)를 통합하려는 시도이다.36 예를 들어, 딥러닝 기반의 분할(segmentation) 모델을 통해 각 포인트를 '도로', '빌딩', '자동차', '보행자' 등으로 분류한다. 그리고 정합 시 '자동차' 포인트는 '빌딩' 포인트와 대응될 수 없다는 강력한 제약을 추가하여 정확도와 강건성을 한 단계 더 높인다. 또한, 맵에서 주차된 차량과 같이 시간에 따라 변하는 동적 객체들을 필터링하여, 시간이 지나도 안정적으로 재사용할 수 있는 '영속적인(persistent)' 맵을 구축하는 데 활용될 수 있다.36

결론적으로, KISS-ICP는 포인트 클라우드 정합 분야에 중요한 전환점을 제시했다. 복잡성의 경쟁에서 벗어나 기본으로 돌아감으로써 더 강건하고 범용적인 해법을 찾을 수 있음을 증명했다. KISS-ICP 자체는 강력한 LiDAR Odometry 도구로서 그 가치가 충분하며, 동시에 그 한계와 철학은 KISS-SLAM, Semantic-KISS 등 더 발전된 차세대 SLAM 시스템을 위한 견고한 초석이 되고 있다. 이는 KISS-ICP가 단순한 알고리즘을 넘어, 로보틱스 및 컴퓨터 비전 분야의 연구 개발 방법론에 새로운 영감을 주는 성공적인 패러다임 전환의 사례임을 보여준다.


1. Understanding Iterative Closest Point (ICP) Algorithm with Code - LearnOpenCV, accessed July 24, 2025, https://learnopencv.com/iterative-closest-point-icp-explained/
2. Iterative closest point - Wikipedia, accessed July 24, 2025, https://en.wikipedia.org/wiki/Iterative_closest_point
3. Mastering ICP Algorithm in Metrology - Number Analytics, accessed July 24, 2025, https://www.numberanalytics.com/blog/ultimate-guide-icp-algorithm-metrology-inspection
4. The PCL Registration API - Point Cloud Library 0.0 documentation - Read the Docs, accessed July 24, 2025, https://pcl.readthedocs.io/projects/tutorials/en/master/registration_api.html
5. Point Cloud Registration: Beyond the Iterative Closest Point Algorithm - Think Autonomous, accessed July 24, 2025, https://www.thinkautonomous.ai/blog/point-cloud-registration/
6. 12.2: The Iterative Closest Point (ICP) Algorithm - Engineering LibreTexts, accessed July 24, 2025, https://eng.libretexts.org/Bookshelves/Mechanical_Engineering/Introduction_to_Autonomous_Robots_(Correll)/12%3A__RGB-D_SLAM/12.02%3A_The_Iterative_Closest_Point_(ICP)_Algorithm
7. ICP registration - Open3D 0.19.0 documentation, accessed July 24, 2025, https://www.open3d.org/docs/release/tutorial/pipelines/icp_registration.html
8. ICP Algorithm: Theory, Practice And Its SLAM-oriented Taxonomy - arXiv, accessed July 24, 2025, https://arxiv.org/pdf/2206.06435
9. Gentle Introduction to Global Point Cloud Registration using Open3D | by Amnah Ebrahim, accessed July 24, 2025, https://medium.com/@amnahhmohammed/gentle-introduction-to-point-cloud-registration-using-open3d-pt-2-18df4cb8b16c
10. Point Cloud Alignment in Open3D using the Iterative Closest Point (ICP) Algorithm - Medium, accessed July 24, 2025, https://medium.com/@BlanchR2/point-cloud-alignment-in-open3d-using-the-iterative-closest-point-icp-algorithm-22433693aa8a
11. KISS-ICP: In Defense of Point-to-Point ICP – Simple, Accurate, and Robust Registration If Done the Right Way, accessed July 24, 2025, https://www.ipb.uni-bonn.de/wp-content/papercite-data/pdf/vizzo2023ral.pdf
12. KISS-ICP: In Defense of Point-to-Point ICP – Simple, Accurate, and Robust Registration If Done the Right Way - arXiv, accessed July 24, 2025, https://arxiv.org/html/2209.15397
13. KISS-ICP: In Defense of Point-to-Point ICP Simple, Accurate, and Robust Registration If Done the Right Way - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/367101986_KISS-ICP_In_Defense_of_Point-to-Point_ICP_Simple_Accurate_and_Robust_Registration_If_Done_the_Right_Way
14. KISS-ICP: In Defense of Point-to-Point ICP -- Simple, Accurate, and Robust Registration If Done the Right Way - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/364110233_KISS-ICP_In_Defense_of_Point-to-Point_ICP_--_Simple_Accurate_and_Robust_Registration_If_Done_the_Right_Way
15. KISS-ICP: In Defense of Point-to-Point ICP - Accurate and Robust 3D Point Cloud Registration | by Cyrill Stachniss | StachnissLab | Medium, accessed July 24, 2025, https://medium.com/stachnisslab/kiss-icp-in-defense-of-point-to-point-icp-accurate-and-robust-3d-point-cloud-registration-bd8a21cae3d8
16. Comparing ICP Variants on Real-World Data Sets - Francis Colas, accessed July 24, 2025, https://www.franciscolas.net/publications/2013_Pomerleau_AutonomousRobots_Comparing.pdf
17. Pset #2 part 1 of 3: Iterative Closest Point (ICP) - Robotic Manipulation, accessed July 24, 2025, https://manipulation.csail.mit.edu/Fall2020/pset2/pset2_ICP.html
18. A Tutorial on Rigid Registration Iterative Closed Point (ICP), accessed July 24, 2025, https://www.sci.utah.edu/~shireen/pdfs/tutorials/Elhabian_ICP09.pdf
19. Generalized-ICP - Robotics, accessed July 24, 2025, https://www.roboticsproceedings.org/rss05/p21.pdf
20. KISS-ICP - Rerun, accessed July 24, 2025, https://rerun.io/examples/robotics/kiss-icp
21. KISS-SLAM: A Simple, Robust, and Accurate 3D LiDAR SLAM System With Enhanced Generalization Capabilities, accessed July 24, 2025, https://www.ipb.uni-bonn.de/wp-content/papercite-data/pdf/kiss2025iros.pdf
22. Generalized ICP - Wiki, accessed July 24, 2025, https://wiki.hanzheteng.com/algorithm/slam/generalized-icp
23. Normal distributions transform - Wikipedia, accessed July 24, 2025, https://en.wikipedia.org/wiki/Normal_distributions_transform
24. gitBook_PCL/Tutorial/Registration/How-to-use-Normal-Distributions-Transform.md at master, accessed July 24, 2025, https://github.com/adioshun/gitBook_PCL/blob/master/Tutorial/Registration/How-to-use-Normal-Distributions-Transform.md
25. Why ICP is much faster than NDT in front_end? / Issue #127 / koide3 ..., accessed July 24, 2025, https://github.com/koide3/hdl_graph_slam/issues/127
26. GenZ-ICP: Generalizable and Degeneracy-Robust LiDAR Odometry Using an Adaptive Weighting - arXiv, accessed July 24, 2025, https://arxiv.org/html/2411.06766v1
27. Evaluation of 3D Registration Reliability and Speed–A Comparison ..., accessed July 24, 2025, https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=0185f6fb7a7a7859daf8ae326b485e9891a7461d
28. Kiss-ICP struggles with sparse data ? / Issue #128 - GitHub, accessed July 24, 2025, https://github.com/PRBonn/kiss-icp/issues/128
29. KITTI Odometry Benchmark - Andreas Geiger, accessed July 24, 2025, https://www.cvlibs.net/datasets/kitti/eval_odometry.php
30. KITTI Benchmark (Point Cloud Registration) - Papers With Code, accessed July 24, 2025, https://paperswithcode.com/sota/point-cloud-registration-on-kitti
31. PRBonn/kiss-icp: A LiDAR odometry pipeline that just works - GitHub, accessed July 24, 2025, https://github.com/PRBonn/kiss-icp
32. Failure case study / Issue #151 / PRBonn/kiss-icp - GitHub, accessed July 24, 2025, https://github.com/PRBonn/kiss-icp/issues/151
33. KISS-ICP: In Defense of Point-to-Point ICP – Simple, Accurate, and Robust Registration If Done the Right Way - Bohrium, accessed July 24, 2025, https://www.bohrium.com/paper-details/kiss-icp-in-defense-of-point-to-point-icp-simple-accurate-and-robust-registration-if-done-the-right-way/864973736359493879-2536
34. KISS-Matcher: Fast and Robust Point Cloud Registration Revisited - arXiv, accessed July 24, 2025, https://arxiv.org/html/2409.15615v2
35. KISS-Matcher: Fast, Robust, and Scalable Registration + ROS2 SLAM examples - GitHub, accessed July 24, 2025, https://github.com/MIT-SPARK/KISS-Matcher
36. A Chefs KISS -- Utilizing semantic information in both ICP and SLAM framework - arXiv, accessed July 24, 2025, https://arxiv.org/abs/2504.02086



