# Iterative Closest Point (ICP) 포인트 클라우드 정합
[포인트 클라우드 정합](./index.md)



점군(Point Cloud)은 3차원 공간상에 분포하는 점들의 집합으로, LiDAR(Light Detection and Ranging), RGB-D 카메라, 구조광 스캐너 등 다양한 3D 센서를 통해 현실 세계의 기하학적 정보를 포착한 핵심적인 데이터 형태이다.1 이 데이터는 각 점의 3차원 좌표($x, y, z$)를 기본으로 하며, 때로는 색상(RGB), 반사 강도(intensity), 법선 벡터(normal vector) 등의 추가 정보를 포함할 수 있다.

점군 정합(Point Cloud Registration)이란, 서로 다른 시점, 시간 또는 다른 센서로부터 취득되어 각기 다른 로컬 좌표계를 갖는 두 개 이상의 점군을 하나의 일관된 전역 좌표계(global coordinate system)로 정렬하는 과정을 의미한다.3 정합의 궁극적인 목표는 한 점군(소스, source)을 다른 점군(타겟, target)에 최적으로 맞추는 강체 변환(Rigid Transformation)을 찾는 것이다. 강체 변환은 객체의 형태를 왜곡시키지 않는 변환으로, 오직 회전(Rotation)과 이동(Translation)만으로 구성된다.4

점군 정합 기술은 수많은 첨단 기술 분야의 근간을 이루는 핵심 요소이다. 로봇 공학에서는 로봇이 미지의 환경을 탐험하며 자신의 위치를 추정하고 동시에 지도를 작성하는 SLAM(Simultaneous Localization and Mapping) 기술의 핵심이며, 이를 통해 신뢰성 있는 경로 계획을 수립한다.5 3D 모델링 및 역공학 분야에서는 여러 각도에서 스캔한 객체의 부분적인 점군들을 병합하여 완전한 디지털 3D 모델을 생성하는 데 사용된다.5 자율 주행 자동차는 LiDAR 점군 정합을 통해 고정밀 지도(HD Map) 내에서 자신의 위치를 정밀하게 파악하고 주변 환경을 인식한다.8 의료 영상 분야에서는 CT, MRI 등 여러 영상 데이터를 정합하여 환자의 해부학적 구조를 다각적으로 분석하고 수술 계획을 수립하는 데 활용된다.10 이처럼 점군 정합은 디지털 세계와 물리적 세계를 연결하는 교량 역할을 수행하며, 그 중요성은 날로 증대되고 있다.


1992년, Besl과 McKay 6, 그리고 거의 동시에 Chen과 Medioni 5에 의해 제안된 Iterative Closest Point (ICP) 알고리즘은 점군 정합 문제를 해결하는 고전적이면서도 가장 영향력 있는 방법론으로 자리매김했다. ICP의 핵심 철학은 정합 문제에 내재된 "닭과 달걀 문제(Chicken-and-Egg Problem)"를 독창적인 반복적 접근법으로 해결하는 데 있다.

이 문제는 다음과 같이 요약될 수 있다 12:

1. 만약 두 점군 사이의 정확한 대응점(correspondence), 즉 어떤 소스 점이 어떤 타겟 점에 해당하는지를 미리 알고 있다면, 두 점군 집합 간의 오차를 최소화하는 최적의 변환 행렬(회전 및 이동)을 계산하는 것은 간단한 최소 제곱 문제(least-squares problem)로 귀결된다.12
2. 반대로, 만약 두 점군을 완벽하게 정렬하는 최적의 변환 행렬을 알고 있다면, 소스 점군을 변환시킨 후 타겟 점군에서 가장 가까운 점을 찾는 방식으로 정확한 대응점을 매우 쉽게 찾을 수 있다.12

이처럼 대응점 관계와 최적 변환은 서로가 서로를 필요로 하는 순환 의존성을 가진다. ICP는 이 딜레마를 정면으로 돌파하는 대신, 두 단계를 번갈아 가며 점진적으로 해를 개선해 나가는 현명한 전략을 채택했다.4 ICP의 반복 루프는 다음과 같은 두 가지 핵심 단계로 구성된다.

1. **가정 (Assume):** 현재까지 계산된 변환을 기준으로, 소스 점군의 각 점에 대해 타겟 점군에서 기하학적으로 가장 가까운 점(Closest Point)이 올바른 대응점일 것이라고 **가정**한다.
2. **최적화 (Optimize):** 이렇게 가정된 대응점 쌍들의 오차(일반적으로 제곱 거리의 합)를 최소화하는 새로운 최적의 변환을 **계산**한다.
3. **반복 (Iterate):** 계산된 새로운 변환을 소스 점군에 적용하고, 이 과정이 미리 정의된 수렴 조건(예: 오차의 변화가 미미해지거나 최대 반복 횟수에 도달)을 만족할 때까지 1번과 2번 단계를 **반복**한다.

이러한 반복적 접근법은 초기 추정치가 합리적인 범위 내에 있다면, 점차적으로 오차를 줄여나가며 매우 정밀한 정합 결과를 도출할 수 있다. ICP는 이처럼 직관적이면서도 강력한 철학 덕분에 지난 30년간 수많은 변형과 개선을 거치며 점군 정합 분야의 표준 알고리즘으로 군림해왔다.


본 보고서는 ICP 알고리즘을 다각도에서 심층적으로 고찰하는 것을 목표로 한다. 먼저 II장에서는 고전적인 Point-to-Point ICP의 수학적 원리와 SVD(특이값 분해)를 이용한 해법을 상세히 분석한다. III장에서는 Point-to-Plane, G-ICP와 같은 주요 변형 알고리즘들의 핵심 아이디어와 장단점을 비교 분석한다. IV장에서는 ICP가 가진 근본적인 한계점들(지역 최솟값, 아웃라이어 등)을 명확히 정의하고, V장에서는 이러한 한계를 극복하기 위한 전역 정합(Global Registration)과 같은 핵심 전략들을 다룬다. VI장에서는 전통적인 방식을 넘어, 딥러닝을 활용한 현대적인 점군 정합 방법론의 동향과 가능성을 탐구한다. VII장에서는 SLAM, 3D 모델링, 의료 영상 등 주요 응용 분야에서 ICP가 어떻게 활용되고 변형되는지를 구체적인 사례를 통해 고찰한다. 마지막으로 VIII장에서는 전체 논의를 종합하여 ICP의 현재 위상을 평가하고 미래 연구 방향을 전망하며 보고서를 마무리한다.


고전적 ICP, 또는 표준 ICP로 불리는 Point-to-Point ICP는 알고리즘의 가장 기본적인 형태이다. 그 이름에서 알 수 있듯이, 이 방식은 소스 점군(source point cloud)의 각 점과 타겟 점군(target point cloud)에서 가장 가까운 점 사이의 직접적인 거리(point-to-point distance)를 최소화하는 것을 목표로 한다.4 이 장에서는 Point-to-Point ICP의 단계별 파이프라인과 그 핵심을 이루는 수학적 원리를 상세히 기술한다.


Point-to-Point ICP의 전체 과정은 일반적으로 4개의 주요 단계로 구성되며, 이 단계들이 수렴할 때까지 반복된다.4 소스 점군을 
$$
P = \{p_1, p_2, \dots, p_{N_p}\}
$$
, 타겟 점군을
$$
Q = \{q_1, q_2, \dots, q_{N_q}\}
$$
라 하자.

1. **초기화 (Initialization):** ICP 알고리즘은 무(無)에서 시작하지 않는다. 소스 점군 $P$를 타겟 점군 $Q$에 대략적으로 정렬하는 초기 변환 $T_0 = (R_0, t_0)$가 필요하다.4 이 초기 추정치의 품질은 ICP의 최종 결과에 결정적인 영향을 미친다. 만약 초기 추정치가 실제 정답과 너무 멀리 떨어져 있다면, 알고리즘은 잘못된 지역 최솟값(local minimum)에 수렴할 위험이 매우 크다.4 이 초기 추정치는 다양한 방법으로 얻을 수 있다. 예를 들어, 로봇에 부착된 GPS/IMU 센서로부터 얻는 대략적인 위치 정보, 사용자가 직접 두 점군을 수동으로 맞춰보는 방법, 또는 다음에 다룰 전역 정합(Global Registration) 알고리즘의 결과를 사용하는 방법 등이 있다.4

2. **대응점 탐색 (Finding Correspondences):** 이 단계는 ICP의 'Closest Point'라는 이름이 유래한 부분이다. 현재의 변환 $T_k = (R_k, t_k)$가 적용된 소스 점군 $P'_k = \{R_k p_i + t_k\}$의 각 점 $p'_{i,k}$에 대해, 타겟 점군 $Q$ 내에서 유클리드 거리가 가장 가까운 점 $q_i$를 찾는다. 이 과정을 통해 $N_p$개의 대응점 쌍 집합 $C_k = \{(p_i, q_i)\}$가 생성된다. 모든 점에 대해 전체 타겟 점군을 탐색하는 것은 계산 비용이 $O(N_p N_q)$로 매우 높다. 이 계산 병목 현상을 해결하기 위해, 정적인 타겟 점군 $Q$에 대해 k-차원 트리(k-d tree)라는 자료구조를 미리 구축하는 것이 일반적이다.4 k-d 트리는 3차원 공간을 재귀적으로 분할하여, 최근접 이웃 탐색(Nearest Neighbor Search)의 시간 복잡도를 평균적으로  $O(\log N_q)$까지 획기적으로 줄여준다. 이로써 전체 대응점 탐색 단계의 속도를 비약적으로 향상시킬 수 있다.4

3. **변환 추정 (Estimating Transformation):** 2단계에서 찾아낸 대응점 쌍 집합 $C_k$를 사용하여, 이들의 오차를 최소화하는 최적의 회전 행렬 $R_{k+1}$과 이동 벡터 $t_{k+1}$을 계산한다. Point-to-Point ICP에서는 이 오차를 대응점 간의 제곱 유클리드 거리의 합으로 정의한다. 이 최적화 문제는 다행히 해석적 해(closed-form solution)가 존재하며, 특이값 분해(SVD)를 통해 안정적으로 풀 수 있다. 자세한 수학적 유도 과정은 2.3절에서 상세히 다룬다.4

4. **적용 및 수렴 판단 (Applying Transformation and Convergence Check):** 3단계에서 계산된 새로운 변환 $T_{k+1} = (R_{k+1}, t_{k+1})$을 소스 점군에 적용하여 업데이트한다. 일반적으로 변환을 누적하여 적용하는 방식을 사용한다 ($T_{final} = T_{k+1} \circ T_k \circ \dots \circ T_0$). 이후, 미리 정의된 수렴 조건을 확인한다. 일반적인 수렴 조건은 다음과 같다 2:

   - 대응점 쌍의 평균 제곱근 오차(RMSE)가 특정 임계값 이하로 떨어질 때.

   - 이전 반복 단계와 현재 반복 단계에서 계산된 변환 행렬의 차이가 매우 작아질 때.

   - 미리 설정된 최대 반복 횟수에 도달했을 때.

     이 조건 중 하나라도 만족되면 알고리즘은 종료되고, 최종 변환 행렬을 반환한다. 그렇지 않으면 2단계로 돌아가 과정을 반복한다.14


Point-to-Point ICP의 목표는 명확하다: 주어진 대응점 쌍에 대해, 변환된 소스 점과 타겟 점 사이의 거리 제곱의 합을 최소화하는 것이다. 이를 수학적으로 표현한 비용 함수(Cost Function) 또는 오차 함수(Error Function) $E(R, t)$는 다음과 같이 정의된다.1
$$
E(R, t) = \sum_{i=1}^{N} \| (Rp_i + t) - q_i \|^2
$$
여기서 $p_i$는 소스 점군 $P$의 $i$번째 점, $q_i$는 현재 변환된 $p_i$에 가장 가까운 타겟 점군 $Q$의 점, $R$은 회전 행렬, $t$는 이동 벡터, 그리고 $N$은 찾아낸 대응점 쌍의 총 개수를 의미한다. ICP의 변환 추정 단계는 바로 이 $E(R, t)$를 최소화하는 $R$과 $t$를 찾는 과정이다.


비용 함수 $E(R, t)$를 최소화하는 문제는 해석적으로 풀 수 있으며, 특이값 분해(Singular Value Decomposition, SVD)는 이를 위한 가장 강인하고 널리 사용되는 방법이다.1 해를 구하는 과정은 다음과 같은 단계로 이루어진다.

1. **중심점(Centroid) 계산:** 먼저, 두 대응점 집합 $\{p_i\}$와 $\{q_i\}$의 기하학적 중심(무게중심)을 각각 계산한다.
   $$
   \mu_p = \frac{1}{N} \sum_{i=1}^{N} p_i, \quad \mu_q = \frac{1}{N} \sum_{i=1}^{N} q_i
   $$
   1

2. **중심점 제거 (Centering):** 각 점 집합에서 해당 집합의 중심점을 빼서, 두 점군 집합의 중심을 모두 원점(0,0,0)으로 이동시킨다. 이렇게 중심화된 점들을 $p'_i$와 $q'_i$라고 하자.
   $$
   p'_i = p_i - \mu_p, \quad q'_i = q_i - \mu_q
   $$
   1 이렇게 하면 이동 벡터 

   $t$와 회전 행렬 $R$을 분리하여 풀 수 있게 되어 문제가 단순화된다.

3. **공분산 행렬 (Covariance Matrix) H 계산:** 중심화된 두 점군 집합 사이의 상호 관계를 나타내는 $3 \times 3$ 공분산 행렬(또는 교차-공분산 행렬) $H$를 계산한다.
   $$
   H = \sum_{i=1}^{N} p'_i (q'_i)^T
   $$
   1 이 행렬은 두 점군이 정렬된 방향에 대한 정보를 압축하여 담고 있다. 참고로, 문헌에 따라 

   $H = \sum q'_i (p'_i)^T$ 로 정의하기도 하는데, 이 경우 최종 회전 행렬 $R$의 계산식이 달라지므로 주의가 필요하다.19

4. **SVD 수행:** 행렬 $H$에 대해 특이값 분해를 수행한다. SVD는 임의의 행렬을 세 개의 특별한 행렬의 곱으로 분해하는 강력한 선형대수 기법이다.
   $$
   H = U \Sigma V^T
   $$
   1 여기서 

   $U$와 $V$는 직교 행렬(orthogonal matrices)이고, $\Sigma$는 특이값(singular values)들을 대각 원소로 갖는 대각 행렬이다.

5. **최적 회전 행렬 R 계산:** SVD 결과로부터 최적의 회전 행렬 $R$을 다음과 같이 직접 계산할 수 있다.
   $$
   R = V U^T
   $$
   1 이 공식은 E(R,t)$를 최소화하는 회전을 보장한다.

   - **반사 변환(Reflection) 처리:** 한 가지 중요한 예외 처리가 필요하다. SVD로 계산된 $R$이 드물게 반사(reflection) 변환을 포함하는 경우가 있다 (즉, $det(R) = -1$). 이는 기하학적으로 올바른 '회전'이 아니다. 이 경우, $R$이 올바른 강체 회전($det(R) = 1$)이 되도록 보정해주어야 한다. 일반적인 방법은 $V$ 행렬의 마지막 열의 부호를 뒤집은 후($V' = V \cdot diag(1, 1, -1)$), $R = V' U^T$로 다시 계산하는 것이다.4

6. **최적 이동 벡터 t 계산:** 최적의 회전 행렬 $R$을 구했다면, 최적의 이동 벡터 $t$는 1단계에서 계산한 중심점들을 이용하여 간단히 계산할 수 있다.
   $$
   t = \mu_q - R \mu_p
   $$
   1 이는 변환된 소스 점군의 중심($R\mu_p + t$)이 타겟 점군의 중심($\mu_q$)과 일치하도록 만드는 직관적인 해이다.

이처럼 SVD를 이용한 해법은 수학적으로 매우 안정적이고 우아하며, 고전적 ICP가 초기에 널리 채택된 주요한 이유가 되었다. 그러나 이 수학적 완벽성은 '대응점이 올바르다'는 강력한 가정 하에서만 유효하다는 점을 기억해야 한다. 실제 세계에서는 이 가정이 쉽게 깨지며, 이것이 바로 고전적 ICP의 본질적인 한계이자 다양한 변형 알고리즘이 등장하게 된 배경이다.


고전적인 Point-to-Point ICP는 그 단순성과 명확성에도 불구하고 몇 가지 근본적인 한계를 지닌다. 특히 평평하거나 특징이 없는 표면에서의 느린 수렴 속도와 잘못된 대응점에 대한 민감성은 실제 응용에서 종종 문제로 작용한다. 이러한 단점을 극복하기 위해 지난 수십 년간 수많은 ICP 변형 알고리즘이 제안되었다. 이 장에서는 그중 가장 중요하고 널리 사용되는 두 가지 변형인 Point-to-Plane ICP와 Generalized-ICP(G-ICP)를 심층적으로 분석하고, 이들의 철학적, 수학적 차이를 명확히 비교한다. ICP의 발전사는 본질적으로 '대응점 간의 오차'를 어떻게 더 정교하고 강인하게 정의하는가의 역사와 같다.


Point-to-Plane ICP는 Chen과 Medioni에 의해 제안된 방식으로, 고전적 ICP의 성능을 크게 개선하여 현재 많은 라이브러리에서 표준적인 선택지로 자리 잡았다.17

- **핵심 아이디어:** 이 방식의 핵심은 오차를 측정하는 기준을 '점과 점(point-to-point)' 사이의 유클리드 거리에서, '소스 점과 타겟 점에서의 접평면(tangent plane)' 사이의 수직 거리로 변경한 것이다.4 즉, 소스 점 

  $p_i$를 변환시킨 $(Rp_i+t)$가 타겟 대응점 $q_i$와 정확히 일치해야 한다는 엄격한 조건 대신, 변환된 소스 점이 $q_i$를 지나가는 평면 위에 놓이도록 유도한다. 이 접근법은 특히 평평한 표면을 따라 미끄러지는(sliding) 움직임에 대해 불필요한 페널티를 부여하지 않기 때문에, 훨씬 더 빠르고 안정적인 수렴을 가능하게 한다.14

- **비용 함수:** Point-to-Plane의 비용 함수를 정의하기 위해서는 타겟 점군 $Q$의 각 점 $q_i$에서의 법선 벡터(normal vector) $n_i$가 필요하다. 이 법선 벡터는 주변 점들을 이용하여 추정할 수 있다. 비용 함수 $E(R, t)$는 변환된 소스 점과 타겟 점을 잇는 벡터를 타겟 점의 법선 벡터에 투영(projection)한 길이의 제곱 합으로 정의된다.17
  $$
  E(R, t) = \sum_{i=1}^{N} \left( ((Rp_i + t) - q_i) \cdot n_i \right)^2
  $$
  이 식은 변환된 소스 점 $(Rp_i+t)$와 타겟 점 $q_i$가 정의하는 접평면 사이의 수직 거리를 의미한다.

- **최적화 문제와 선형화(Linearization):** Point-to-Point 비용 함수와 달리, 이 비용 함수는 회전 행렬 $R$의 삼각함수 항들로 인해 비선형(non-linear) 형태를 띤다. 따라서 SVD와 같은 닫힌 형식(closed-form)의 해가 존재하지 않는다.20 이 비선형 최적화 문제는 일반적으로 Levenberg-Marquardt(LM)와 같은 반복적인 수치 최적화 기법으로 풀 수 있다. 그러나 더 효율적인 방법은 작은 각도 근사(small angle approximation)를 이용한 선형화이다. 두 점군 간의 회전이 작다고 가정하면(

  $\theta \approx 0$), 회전 행렬 $R$을 작은 회전 각도 ($(\alpha, \beta, \gamma)$)에 대한 선형 식으로 근사할 수 있다.20 예를 들어, 

  $sin(\theta) \approx \theta$, $cos(\theta) \approx 1$ 과 같은 근사를 적용하면, 비선형 비용 함수는 미지의 변환 파라미터 ($(\alpha, \beta, \gamma, t_x, t_y, t_z)$)에 대한 선형 최소 제곱 문제 $Ax=b$ 형태로 변환된다. 이 선형 시스템은 SVD나 정규 방정식(Normal Equation)을 통해 매우 효율적으로 풀 수 있다.20

- **장단점:** Point-to-Plane ICP의 가장 큰 장점은 Point-to-Point 방식에 비해 훨씬 적은 반복 횟수로 더 정확한 정합 결과에 도달하는 빠른 수렴 속도이다.14 이는 많은 실제 응용에서 매우 중요한 이점이다. 반면, 단점으로는 타겟 점군의 법선 벡터를 추가로 계산해야 한다는 점과, 각 반복 단계에서의 계산이 (선형화를 사용하더라도) Point-to-Point의 SVD 해법보다 약간 더 복잡하다는 점을 들 수 있다.17 그럼에도 불구하고 전체적인 성능 향상 효과가 워낙 뛰어나, PCL, Open3D와 같은 주요 라이브러리에서 널리 사용되고 있다.17


Generalized-ICP(G-ICP)는 Segal 등에 의해 제안되었으며, 기존의 ICP 변형들을 하나의 통합된 확률적 관점에서 재해석한 매우 강력한 프레임워크이다.22

- **핵심 아이디어:** G-ICP는 Point-to-Point와 Point-to-Plane을 개별적인 알고리즘이 아닌, 동일한 확률 모델의 다른 표현으로 간주한다. 핵심 아이디어는 점군 데이터가 실제 표면에서 샘플링될 때 발생하는 불확실성(uncertainty)과 센서 노이즈를 명시적으로 모델링하는 것이다.22 G-ICP는 두 점군 모두에서 지역적으로 평면적인 구조(locally planar structure)를 가정하고, 이를 공분산 행렬(covariance matrix)을 통해 표현한다. 이를 통해 단순히 점 대 점이나 점 대 평면이 아닌, '평면 대 평면(plane-to-plane)'에 가까운 정합을 수행한다.22

- **확률 모델과 비용 함수:** G-ICP는 각 측정된 점 $p_i$가, 알 수 없는 실제 표면 위의 점 $\hat{p}_i$를 중심으로 하는 가우시안 분포에서 샘플링되었다고 가정한다. 이 분포의 형태는 공분산 행렬 $\Sigma_{p_i}$로 표현된다. 만약 점 $p_i$가 평평한 표면 위에 있다면, 이 공분산 행렬은 표면의 법선 방향으로는 작은 분산(높은 확실성)을 갖고, 평면과 접하는 방향으로는 큰 분산(낮은 확실성)을 갖는 타원체 형태가 될 것이다.22 G-ICP는 두 대응점 $p_i$, $q_i$ 사이의 유클리드 거리가 아닌, 두 점과 연관된 확률 분포 사이의 마할라노비스 거리(Mahalanobis distance)를 최소화하는 변환 $T=(R,t)$를 찾는다.25
  $$
  d(p_i, q_i) = (T p_i - q_i)^T (\Sigma_{q_i} + T \Sigma_{p_i} T^T)^{-1} (T p_i - q_i)
  $$
  이 비용 함수는 각 점의 불확실성을 고려하여 오차를 계산하므로, 노이즈나 잘못된 대응점에 훨씬 강인하다.

- **통합적 관점:** G-ICP 프레임워크 내에서 기존 ICP들은 다음과 같이 해석될 수 있다.22

  - **Point-to-Point ICP:** 타겟 점군 $Q$는 불확실성이 없는 완벽한 데이터($\Sigma_q = 0$)이고, 소스 점군 $P$의 각 점은 모든 방향으로 동일한 불확실성($\Sigma_p = I$, 단위 행렬)을 갖는다고 가정한 특수한 경우이다.
  - **Point-to-Plane ICP:** 타겟 점군 $Q$는 완벽한 데이터($\Sigma_q = 0$)이고, 소스 점군 $P$는 법선 방향으로만 불확실성을 갖고 평면 방향으로는 불확실성이 없다고 가정한 경우이다.

- **장점:** G-ICP의 가장 큰 장점은 강인성(robustness)이다. 두 점군의 구조적 정보를 대칭적으로 모두 활용하므로 한쪽 데이터에 편향되지 않는다.22 특히, 잘못된 대응점의 영향을 효과적으로 줄여주는 능력이 탁월하다. 만약 두 대응점의 지역 평면 방향이 서로 일치하지 않는다면(즉, 잘못된 매칭일 가능성이 높다면), 공분산 행렬의 상호작용으로 인해 해당 쌍의 비용 함수에 대한 기여도가 자연스럽게 낮아진다.22 이 덕분에 다양한 종류의 센서 데이터와 환경에 대해 일반적으로 우수한 성능을 보여주며, 복잡한 시나리오에서 첫 번째 선택지로 고려할 만하다.24


아래 표는 세 가지 핵심적인 ICP 변형 알고리즘의 본질적인 차이점을 요약하여 보여준다. 이는 특정 응용에 어떤 변종이 더 적합할지 판단하는 데 유용한 기준을 제공한다.

| 항목 (Criteria)   | Point-to-Point ICP             | Point-to-Plane ICP                        | Generalized-ICP (G-ICP)                                    |
| ----------------- | ------------------------------ | ----------------------------------------- | ---------------------------------------------------------- |
| **핵심 아이디어** | 대응점 간 유클리드 거리 최소화 | 소스 점에서 타겟 평면까지의 거리 최소화   | 두 점군 표면의 확률적 모델 간 거리 최소화 (Plane-to-Plane) |
| **비용 함수**     | $\sum | (Rp_i + t) - q_i |^2$  | $\sum ( ((Rp_i + t) - q_i) \cdot n_i )^2$ | 마할라노비스 거리 기반 확률적 비용 함수                    |
| **핵심 가정**     | 최근접점이 실제 대응점         | 최근접점의 접평면이 유효                  | 점군은 지역적 평면에서 샘플링된 확률 분포                  |
| **해법**          | 닫힌 형식 (SVD)                | 비선형 (선형 근사로 해결)                 | 비선형 (반복적 최적화)                                     |
| **장점**          | 구현 용이, 각 반복이 빠름      | 빠른 수렴 속도, 높은 정확도               | 강인함, 잘못된 대응점 영향 감소, 대칭성                    |
| **단점**          | 느린 수렴, 지역 최솟값에 취약  | 법선 벡터 계산 필요, 비선형 최적화        | 공분산 추정 필요, 개념적 복잡성                            |

이러한 진화 과정은 ICP가 단순한 거리 최소화 문제를 넘어, 데이터의 기하학적 구조와 통계적 불확실성을 점진적으로 통합하며 발전해왔음을 명확히 보여준다. Point-to-Plane은 성능과 구현 복잡도 사이의 훌륭한 절충점을 제공하며, G-ICP는 최고의 강인성을 목표로 하는 진일보한 프레임워크라 할 수 있다.


ICP 알고리즘과 그 변형들은 3D 점군 정합 분야에서 지대한 성공을 거두었지만, 그 성공은 특정 가정들이 충족될 때라는 전제 조건 하에 이루어진다. 이 장에서는 ICP가 실제 세계의 복잡한 데이터에 적용될 때 직면하는 근본적인 한계와 도전 과제들을 체계적으로 분석한다. 이러한 한계를 이해하는 것은 ICP를 효과적으로 사용하고, 더 나아가 차세대 정합 알고리즘이 나아가야 할 방향을 이해하는 데 필수적이다.


ICP의 가장 치명적인 약점은 본질적으로 지역 최적화(local optimization) 알고리즘이라는 데 있다.4 이는 알고리즘이 탐색을 시작한 지점 근처에서만 최적해를 찾을 수 있다는 의미이다. 비용 함수의 표면은 수많은 '골짜기'(지역 최솟값)를 가지고 있는데, ICP는 한번 특정 골짜기로 들어서면 다른, 더 깊은 골짜기(전역 최솟값, global minimum)가 존재하더라도 그곳으로 빠져나오지 못한다.

따라서 알고리즘의 성공은 전적으로 초기 추정치(initial alignment)의 품질에 달려있다.10 만약 초기 추정치가 실제 정답(전역 최솟값)이 위치한 골짜기 근처에서 시작한다면, ICP는 빠르고 정확하게 최적해로 수렴할 것이다. 하지만 초기 추정치가 좋지 않아 엉뚱한 골짜기에서 시작한다면, 알고리즘은 그 지역 최솟값에 수렴하여 완전히 잘못된 정합 결과를 내놓게 된다.4 이는 두 점군이 멀리 떨어져 있거나 큰 회전 변환을 갖는 경우에 빈번하게 발생하며, ICP를 단독으로 사용하기 어렵게 만드는 가장 큰 이유이다.


실제 센서 데이터는 완벽하지 않다. 측정 과정에서의 오류, 센서의 한계, 또는 환경 내의 동적 객체(움직이는 사람, 차량 등)로 인해 점군 데이터에는 항상 노이즈(noise)와 아웃라이어(outliers)가 포함된다.

- **노이즈:** 점의 위치가 실제 표면에서 미세하게 벗어나는 현상으로, 최소 제곱법(least-squares) 기반의 ICP는 가우시안 노이즈에는 비교적 강인하지만, 노이즈가 클 경우 정합의 정밀도를 저하시킨다.
- **아웃라이어:** 정상적인 데이터 분포에서 크게 벗어난 점들이다. 아웃라이어는 대응점 탐색 단계에서 심각한 문제를 일으킨다. 소스 점군의 한 점이 타겟 점군의 아웃라이어와 잘못 대응되면, 이 하나의 잘못된 대응점 쌍이 최소 제곱 비용 함수에 매우 큰 오차를 발생시켜 전체 최적화 과정을 크게 왜곡할 수 있다.10

이러한 문제에 대처하기 위해, 대응점 쌍 중에서 거리가 특정 임계값을 초과하는 쌍을 제거하거나, 전체 대응점 중 신뢰도가 높은 일부 비율(inlier ratio)만을 사용하는 등의 필터링 전략이 필요하다.5 하지만 이러한 방법들도 완벽하지 않으며, 아웃라이어는 여전히 ICP의 강인성을 위협하는 주요 도전 과제이다.


- **부분적 겹침 (Partial Overlap):** 3D 스캐너의 시야각 제한이나 물체에 의한 가림(occlusion) 현상 때문에, 두 점군은 완벽하게 동일한 영역을 포함하지 않는 경우가 대부분이다.15 즉, 소스 점군의 일부 점들은 타겟 점군에 대응하는 영역이 아예 존재하지 않는다. 이러한 겹치지 않는 영역의 점들은 대응점 탐색 과정에서 필연적으로 잘못된 대응점을 찾게 되며, 이는 아웃라이어와 유사하게 정합 성능을 저하시키는 요인으로 작용한다.22
- **대칭성 및 특징 부족 (Symmetry and Lack of Features):** 정합 알고리즘은 기하학적 구조의 독특함(uniqueness)을 기반으로 올바른 위치를 찾는다. 그러나 환경이 매우 대칭적이거나 특징이 없는 경우, 알고리즘은 올바른 정렬 상태를 구분하기 어려워진다. 예를 들어, 긴 복도, 넓은 평면, 완벽한 구와 같은 구조는 여러 방향으로 정렬해도 기하학적 오차가 유사하게 측정될 수 있다.28 이러한 '퇴화(degenerate)' 환경에서는 비용 함수의 최솟값이 명확하게 하나로 정의되지 않고, 넓고 평평한 계곡 형태를 띠게 된다. 이로 인해 알고리즘은 잘못된 해로 쉽게 미끄러지거나, 특정 방향으로의 변환을 제대로 추정하지 못하는 문제가 발생한다.


현대의 LiDAR 스캐너는 초당 수십만에서 수백만 개의 점을 생성할 수 있다. 점군의 크기가 커질수록 ICP의 계산 비용은 급격히 증가한다.

- **대응점 탐색:** k-d 트리를 사용하면 최근접 이웃 탐색의 시간 복잡도가 평균 $O(\log N_q)$로 줄어들지만, 점의 개수 $N_p$가 매우 클 경우 전체 대응점 탐색 단계($O(N_p \log N_q)$)는 여전히 상당한 시간을 소요한다.4
- **실시간 요구사항:** 특히 로봇의 실시간 SLAM과 같은 응용에서는 매 프레임마다 수십 밀리초(ms) 내에 정합을 완료해야 한다. 대규모 점군에 대해 ICP를 직접 실행하는 것은 이러한 실시간 요구사항을 충족시키기 어렵다.

이 문제를 해결하기 위해, 원본 점군을 복셀 그리드(voxel grid)나 무작위 샘플링(random sampling)을 통해 다운샘플링(down-sampling)하여 점의 개수를 줄이는 기법이 널리 사용된다.7 그러나 이는 필연적으로 원본 데이터의 정밀한 정보를 일부 손실시키는 결과를 낳으며, 속도와 정확도 사이의 트레이드오프(trade-off) 관계를 야기한다.

결론적으로, ICP의 이러한 근본적인 한계들은 "No Free Lunch" 원리를 명확히 보여준다. 빠르고 간단한 지역 최적화 알고리즘은 필연적으로 전역 최적성을 보장하지 못하며, 외부의 도움이(좋은 초기치)나 추가적인 강인한 메커니즘 없이는 복잡하고 불완전한 실제 데이터 앞에서 실패할 수밖에 없다. 바로 이러한 한계점들이 다음에 논의될 전역 정합 및 딥러닝 기반 방법론의 등장을 촉발한 핵심적인 원동력이 되었다.


IV장에서 논의된 ICP의 근본적인 한계, 특히 지역 최솟값 문제에 대한 민감성은 ICP를 단독으로 사용하는 것을 매우 위험하게 만든다. 이 문제를 해결하기 위해 컴퓨터 비전 및 로보틱스 커뮤니티는 '탐색(Exploration)'과 '정제(Refinement)'라는 두 가지 작업을 명확히 분리하는 전략을 개발했다. 이 장에서는 ICP가 자신의 장점인 정밀한 지역 최적화에만 집중할 수 있도록, 강인하게 대략적인 초기 정렬을 제공하는 전역 정합(Global Registration) 방법론을 중심으로 한계 극복 전략을 심층적으로 다룬다.


ICP의 지역 최솟값 문제를 해결하기 위한 가장 표준적이고 효과적인 접근 방식은 Coarse-to-Fine (조밀하지 않은 것에서 조밀한 것으로) 패러다임이다.3 이 패러다임은 정합 문제를 두 단계로 나누어 해결한다.

1. **1단계: Coarse Registration (전역 정합):** 이 단계의 목표는 두 점군의 초기 위치나 방향에 관계없이, 전역적인 탐색을 통해 대략적인 정렬(coarse alignment)을 찾는 것이다. 전역 정합 알고리즘은 일반적으로 계산 비용이 높고 최종 정합 결과가 ICP만큼 정밀하지는 않지만, ICP가 수렴할 수 있는 '올바른 골짜기' 근처로 점군을 이동시켜주는 결정적인 역할을 한다.29 즉, ICP를 위한 합리적인 초기 추정치를 제공하는 것이 주된 목적이다.
2. **2단계: Fine Registration (정밀 정합):** 1단계에서 얻은 대략적인 변환을 초기값으로 사용하여 ICP(주로 성능이 우수한 Point-to-Plane ICP)를 실행한다. 이제 ICP는 전역 최솟값이 있는 유망한 영역에서 탐색을 시작하므로, 지역 최솟값에 빠질 위험이 크게 줄어들고 빠르고 정밀하게 최종 정렬을 다듬을 수 있다.17

이 Coarse-to-Fine 접근법은 각기 다른 장점을 가진 알고리즘들을 현명하게 조합한 대표적인 성공 사례이다. 전역 정합이 '탐색'을, 지역 정합(ICP)이 '정제'를 담당함으로써, 전체 정합 파이프라인의 강인성과 정확성을 동시에 확보한다.


가장 널리 사용되는 전역 정합 파이프라인 중 하나는 지역 기하학적 특징을 이용하는 방법이며, 그 대표적인 예가 FPFH와 RANSAC의 조합이다.29 이 파이프라인은 다음과 같은 단계로 구성된다.

1. **다운샘플링 및 특징 추출 (Downsampling & Feature Extraction):** 먼저, 계산 효율성을 위해 원본 소스 및 타겟 점군을 복셀 그리드 다운샘플링(voxel grid downsampling)을 통해 점의 수를 줄인다.29 그 후, 다운샘플링된 각 점에 대해 지역 기하학적 특징을 기술하는 디스크립터(descriptor)를 계산한다. FPFH(Fast Point Feature Histogram)는 이 목적으로 널리 사용되는 특징 디스크립터이다.30 FPFH는 한 점과 그 이웃 점들 간의 기하학적 관계(거리, 각도 등)를 히스토그램으로 표현한 33차원의 벡터이다. 기하학적으로 유사한 구조를 가진 점들은 서로 다른 위치에 있더라도 유사한 FPFH 값을 갖게 된다.29
2. **초기 대응점 탐색 (Initial Correspondence Search):** 두 점군의 FPFH 특징들을 비교하여 잠재적인 대응점 쌍을 찾는다. 소스 점군의 각 점에 대해, 타겟 점군에서 FPFH 특징 벡터가 가장 유사한(예: 유클리드 거리가 가장 가까운) 점을 찾아 초기 대응점으로 설정한다.29 이 단계에서는 기하학적 위치가 아닌, 33차원의 특징 공간(feature space)에서 최근접 이웃을 탐색한다. 이 과정에서 필연적으로 많은 잘못된 대응점(outliers)이 포함된다.
3. **강인한 변환 추정 (Robust Transformation Estimation with RANSAC):** RANSAC(RANdom SAmple Consensus)은 수많은 노이즈와 아웃라이어가 포함된 데이터 속에서 원하는 모델의 파라미터를 강인하게 추정하는 강력한 알고리즘이다.27 FPFH로 찾은 수많은 잠재적 대응점 쌍 중에서, 일관된 강체 변환을 지지하는 올바른 쌍(inliers)을 찾아내는 데 사용된다.
   - **가설 생성 (Hypothesis Generation):** 전체 대응점 집합에서 무작위로 최소한의 샘플(강체 변환을 계산하기 위해 보통 3쌍)을 추출한다. 이 샘플만을 사용하여 변환 행렬(가설)을 계산한다.
   - **가설 검증 (Hypothesis Verification):** 생성된 가설(변환 행렬)을 전체 대응점 집합에 적용해 본다. 변환된 소스 점과 타겟 점 사이의 거리가 특정 임계값 이내인 대응점의 개수(inlier count)를 센다.
   - **반복 및 선택:** 위 두 과정을 수천에서 수만 번 반복하여, 가장 많은 inlier를 확보한 가설을 최적의 변환 행렬로 채택한다.29 이 과정을 통해 아웃라이어들은 투표에서 소외되어 자연스럽게 걸러진다.

이 FPFH + RANSAC 파이프라인은 ICP가 해결할 수 없는 큰 변위와 회전 문제를 효과적으로 처리하여, ICP를 위한 견고한 출발점을 제공한다.


또 다른 접근법은 ICP 자체를 전역적으로 최적화하려는 시도이다. Go-ICP는 이러한 방법의 대표적인 예이다.10

- **핵심 아이디어:** Go-ICP는 분기 한정(Branch-and-Bound)이라는 탐색 기법을 사용하여, 가능한 모든 3D 변환 공간($SE(3)$)을 체계적으로 탐색한다. 이 과정에서 특정 영역이 최적해를 포함할 가능성이 없다고 판단되면 해당 영역의 탐색을 중단(pruning)함으로써 효율성을 높인다. 이 방법은 계산이 완료되면 반드시 전역 최적해를 찾는 것을 보장한다.10
- **장단점:** Go-ICP는 ICP의 지역 최솟값 문제를 근본적으로 해결한다는 점에서 이론적으로 매우 강력하다. 그러나 변환 공간 전체를 탐색해야 하므로 계산 비용이 매우 높아서, SLAM과 같은 실시간 응용에는 적용하기 어렵다.30 주로 오프라인에서의 고정밀 정합이 요구되거나, 다른 알고리즘의 성능을 평가하기 위한 기준(ground truth)을 설정하는 데 사용된다.

결론적으로, ICP의 한계를 극복하는 가장 실용적인 전략은 Coarse-to-Fine 패러다임을 채택하는 것이다. FPFH와 RANSAC을 결합한 특징 기반 전역 정합은 강인함과 효율성 사이의 균형을 잘 맞춘 접근법으로, ICP가 정밀한 정제 작업에만 집중할 수 있는 이상적인 환경을 만들어준다. 이는 단일 알고리즘의 한계를 인정하고, 여러 알고리즘의 장점을 조합하여 더 어려운 문제를 해결하는 공학적 지혜의 좋은 예시이다.


지난 십여 년간 딥러닝 기술이 컴퓨터 비전 분야에 혁명을 일으키면서, 점군 정합 분야에도 근본적인 패러다임의 전환이 일어나고 있다. 전통적인 방법들이 인간이 직접 설계한 기하학적 특징(hand-crafted features)과 최적화 알고리즘에 의존했던 반면, 딥러닝 기반 접근법은 대규모 데이터를 통해 특징 표현, 대응 관계, 심지어 변환 자체를 직접 학습한다.13 이 장에서는 딥러닝이 점군 정합 문제를 어떻게 재정의하고 있으며, 주요 방법론과 그 장단점은 무엇인지 고찰한다.


딥러닝 기반 점군 정합의 핵심 철학은 '표현 학습(Representation Learning)'에 있다. 전통적인 FPFH와 같은 특징은 고정된 규칙에 따라 계산되지만, 딥러닝 모델, 특히 PointNet 34이나 Transformer 35와 같은 아키텍처는 점군의 원시 데이터(raw data)로부터 정합 작업에 가장 유용한 '특징'이 무엇인지를 스스로 학습한다.

- **장점:**
  - **강인성 (Robustness):** 딥러닝 모델은 다양한 노이즈, 아웃라이어, 밀도 변화를 포함한 대규모 데이터셋으로 학습되기 때문에, 이러한 교란 요인에 대해 전통적인 방법보다 훨씬 강인한 특징을 학습할 수 있다.32
  - **성능 (Performance):** 3DMatch, KITTI와 같은 표준 벤치마크 데이터셋에서 최신 딥러닝 기반 방법들은 전통적인 방법들의 성능을 크게 상회하는 결과를 보여준다.35
  - **속도 (Speed):** 한번 학습된 모델은 추론(inference) 시에는 매우 빠른 속도로 동작할 수 있다. 복잡한 반복 최적화 과정 없이, 한 번의 순전파(feed-forward pass)만으로 정합 결과를 도출하는 엔드투엔드 모델도 존재한다.13
- **단점:**
  - **데이터 의존성 (Data Dependency):** 딥러닝 모델의 성능은 학습 데이터의 양과 질에 크게 의존한다. 고품질의 3D 정합 데이터셋을 구축하는 것은 비용과 시간이 많이 소요되는 작업이다.32
  - **일반화 문제 (Generalization Issue):** 특정 데이터셋(예: 실내 가구)에서 학습된 모델이 완전히 다른 종류의 데이터(예: 자율주행 LiDAR)에 대해서는 성능이 저하되는 도메인 이동(domain shift) 문제가 발생할 수 있다.30
  - **해석 불가능성 (Lack of Interpretability):** 딥러닝 모델은 종종 '블랙박스(black box)'로 동작하여, 왜 특정 정합 결과를 내놓았는지 그 결정 과정을 이해하기 어렵다.13


딥러닝 기반 정합 방법은 크게 '하이브리드 방법'과 '엔드투엔드 방법'으로 나눌 수 있다.

- **하이브리드 방법 (Hybrid Methods):** 이 접근법은 전통적인 Coarse-to-Fine 파이프라인의 일부 구성 요소를 딥러닝 모듈로 대체하여 성능을 극대화하는 전략이다.37
  - **학습 기반 특징 디스크립터:** 파이프라인의 첫 단계인 특징 추출 부분을 딥러닝 모델로 대체하는 것이 가장 일반적이다. 예를 들어, FCGF(Fully Convolutional Geometric Features) 38나 SpinNet 30과 같은 모델은 FPFH보다 훨씬 더 구별력 높고 강인한 특징 벡터를 학습한다. 이렇게 학습된 특징을 추출한 뒤, 후속 단계인 RANSAC과 ICP는 그대로 사용하여 전체 파이프라인의 성능을 향상시킨다.37 이 방식은 딥러닝의 강력한 표현력과 전통적 방법의 기하학적 강인함을 결합한, 현재 매우 실용적이고 효과적인 접근법으로 평가받는다.
- **엔드투엔드 방법 (End-to-End Methods):** 이 방법은 두 점군을 입력으로 받아 최종 변환 행렬을 직접 출력하는 것을 목표로 한다.
  - **PointNetLK:** PointNet 34을 특징 추출기로 사용하여 각 점군을 하나의 전역 특징 벡터로 압축하고, 고전적인 Lucas-Kanade(LK) 알고리즘의 아이디어를 차용하여 두 특징 벡터 간의 차이를 최소화하는 변환을 반복적으로 추정한다.30
  - **Deep Closest Point (DCP):** PointNet과 유사한 인코더로 각 점의 특징을 추출하고, Transformer의 어텐션(attention) 메커니즘을 사용하여 두 점군 사이의 부드러운(soft) 대응 관계를 계산한다. 마지막으로, 미분 가능한(differentiable) SVD 모듈을 통해 이 대응 관계로부터 최적의 변환을 한 번에 계산한다.10
  - **최신 동향:** 최근에는 Transformer 아키텍처를 더욱 적극적으로 활용하여 점들 간의 장거리 의존성 및 컨텍스트를 효과적으로 포착하려는 연구(예: GeoTransformer 35)가 주류를 이루고 있다. 이들은 부분적 겹침이나 대칭적인 구조에서 특히 강력한 성능을 보인다.


아래 표는 점군 정합 분야에서 일어난 패러다임 전환을 명확히 보여준다. 연구자나 개발자가 자신의 문제(데이터 가용성, 실시간 요구사항, 일반화 필요성 등)에 어떤 접근법이 더 적합한지 판단하는 데 중요한 기준을 제공한다.

| 항목 (Aspect)     | 전통적 방법 (e.g., FPFH+RANSAC, ICP)   | 딥러닝 기반 방법 (e.g., FCGF+RANSAC, DCP)          |
| ----------------- | -------------------------------------- | -------------------------------------------------- |
| **특징 추출**     | 수작업 설계 (Hand-crafted, e.g., FPFH) | 데이터 기반 학습 (Learned features)                |
| **대응점 탐색**   | 기하학적/특징적 유사도 기반            | 학습된 유사도 또는 어텐션 메커니즘                 |
| **데이터 의존성** | 낮음 (알고리즘 자체로 동작)            | 높음 (대규모 학습 데이터셋 필요)                   |
| **일반화 성능**   | 특정 데이터에 편향되지 않음            | 학습 데이터 분포에 따라 좌우됨 (Domain shift 문제) |
| **성능**          | 벤치마크에서 종종 딥러닝에 뒤처짐      | 최신 모델들이 SOTA 성능 달성                       |
| **속도**          | RANSAC 등으로 인해 느릴 수 있음        | 추론 시 매우 빠르지만, 학습에 긴 시간 소요         |
| **해석 가능성**   | 높음 (각 단계가 명확함)                | 낮음 (Black box)                                   |

결론적으로, 딥러닝은 특징 공학이라는 어려운 수작업 과정을 자동화하고, 그 결과로 종종 더 높은 성능과 강인성을 달성함으로써 점군 정합 분야를 한 단계 발전시키고 있다. 현재로서는 딥러닝으로 강력한 특징을 추출하고 이를 강인한 고전적 파이프라인(RANSAC)과 결합하는 하이브리드 방식이 실용성과 성능 면에서 가장 균형 잡힌 접근법으로 여겨진다. 앞으로의 연구는 레이블 없는 데이터로 학습하는 비지도/자기지도 학습(unsupervised/self-supervised learning)을 통해 일반화 성능을 높이고, 실시간성과 정확성을 모두 만족시키는 경량화된 엔드투엔드 모델을 개발하는 방향으로 나아갈 것으로 전망된다.33


ICP 알고리즘의 진정한 가치는 그 유연성과 확장성에 있다. 지난 30년간 ICP는 단순한 학술적 알고리즘을 넘어, 로보틱스, 3D 모델링, 의료 등 다양한 산업 현장의 실제 문제를 해결하는 핵심 도구로 자리 잡았다. 이는 ICP의 기본 프레임워크가 각 응용 분야의 고유한 요구사항과 제약 조건에 맞게 끊임없이 수정되고 발전해왔기 때문에 가능했다. 이 장에서는 주요 응용 분야별로 ICP가 어떻게 활용되고 있으며, 각 분야의 특성이 알고리즘에 어떤 영향을 미쳤는지 심층적으로 고찰한다.


자율 주행 자동차와 이동 로봇 분야에서 ICP는 SLAM(Simultaneous Localization and Mapping) 기술의 심장과 같은 역할을 수행한다. SLAM은 로봇이 GPS와 같은 외부 도움 없이, 오직 자신의 센서 데이터만으로 미지의 환경을 탐험하며 지도(Map)를 생성하고, 동시에 그 지도 내에서 자신의 위치(Localization)를 추정하는 기술이다.9

- **핵심 역할: 스캔 매칭 (Scan Matching):** ICP는 SLAM의 여러 구성 요소 중 특히 스캔 매칭 단계에서 핵심적으로 사용된다.42
  - **주행 기록계 추정 (Odometry Estimation):** 로봇이 움직이면서 연속적으로 얻는 LiDAR 스캔(예: 시간 $t$의 스캔과 $t-1$의 스캔)을 ICP로 정합한다. 이 정합 결과로 얻어지는 변환 행렬은 로봇이 그 시간 동안 얼마나 이동하고 회전했는지를 나타내는 정밀한 이동량(odometry) 정보가 된다.5 이는 바퀴 회전 수에 기반한 주행 기록계가 미끄러운 지면 등에서 오차가 누적되는 문제를 보완한다.5
  - **루프 클로저 (Loop Closure):** 로봇이 오랜 시간 주행하다가 이전에 방문했던 장소로 다시 돌아왔을 때, 누적된 주행 기록계 오차로 인해 지도가 뒤틀리는 문제가 발생한다. 루프 클로저는 현재의 센서 스캔과 과거에 생성된 지도의 해당 부분을 ICP로 정합하여, '루프가 닫혔음'을 인식하고 그동안 누적된 오차를 전역적으로 보정하는 매우 중요한 과정이다.6
- **중요성 및 선택 기준:** ICP 기반 스캔 매칭은 특히 GPS 신호가 닿지 않는 실내, 터널, 고층 빌딩이 밀집한 도심 환경에서 로봇이 자신의 위치를 잃지 않도록 하는 강인한 측위 수단을 제공한다.5 또한, 자율 주행을 위한 고정밀 3D 지도(HD Map)를 제작하거나, 도로 공사 등으로 변경된 부분을 탐지하여 지도를 업데이트하는 데에도 ICP가 활용된다.8 SLAM 응용에서는 요구사항에 따라 적합한 ICP 변형을 선택하는 것이 중요하다. 예를 들어, 실시간으로 동작해야 하는 온라인 SLAM에서는 계산 효율성이 높은 ICP 변형이 요구되는 반면, 오프라인으로 고정밀 지도를 제작할 때는 시간이 더 걸리더라도 정확도가 높은 변형(예: G-ICP)을 사용하는 것이 유리하다.9


문화유산의 디지털 아카이빙, 산업 부품의 역공학, 가상현실(VR) 콘텐츠 제작 등에서 완전한 3D 모델을 생성하는 것은 필수적이다. ICP는 여러 각도에서 촬영하여 얻은 부분적인 3D 스캔들을 하나의 완전한 모델로 정합하는 표준적인 방법으로 사용된다.5

- **파이프라인:** 이 분야의 작업 흐름은 전형적인 Coarse-to-Fine 패러다임을 따른다.
  1. **데이터 획득:** 3D 스캐너를 이용해 대상 물체를 여러 방향에서 스캔하여 다수의 부분 점군(partial scans)을 얻는다.
  2. **초기 정합 (Coarse Registration):** 사용자가 직접 대응점을 몇 개 지정하여 수동으로 대략적인 정렬을 하거나, V장에서 설명한 FPFH+RANSAC과 같은 전역 정합 알고리즘을 사용하여 자동화된 초기 정렬을 수행한다.4
  3. **정밀 정합 (Fine Registration):** 초기 정합으로 얻은 변환을 시작점으로 하여 ICP를 실행, 스캔들 간의 미세한 오차를 최소화하고 매끄럽게 연결된 최종 3D 모델을 완성한다.7
- **오픈소스 생태계:** ICP는 이 분야에서 워낙 기본적인 기술이기 때문에, MeshLab, CloudCompare, PCL(Point Cloud Library), Open3D 등 널리 사용되는 오픈소스 소프트웨어 및 라이브러리에 표준 기능으로 구현되어 있어 연구자 및 개발자들이 쉽게 접근하고 활용할 수 있다.5


의료 영상 분야에서 ICP는 서로 다른 시간, 다른 장비, 또는 다른 모달리티(modality)로 촬영된 의료 영상(예: CT, MRI, PET)에서 추출한 해부학적 구조(뼈, 장기 등)의 표면을 정렬하는 데 널리 사용된다.10 이를 통해 질병의 진행 상황을 추적하거나, 수술 계획을 시뮬레이션하고, 방사선 치료의 정확도를 높일 수 있다.

- **핵심 도전 과제: 비강체 변형 (Non-rigid Deformation):** 의료 영상 정합이 다른 분야와 구별되는 가장 큰 특징은 바로 '비강체 변형' 문제이다. 산업용 부품이나 건물과 달리, 인체의 부드러운 조직(soft tissue)은 호흡, 심장 박동, 환자의 자세 변화, 병리학적 변화 등으로 인해 시간이 지남에 따라 그 형태가 변한다.45 강체 변환(회전, 이동)만을 가정하는 표준 ICP 알고리즘은 이러한 비강체 변형을 제대로 처리할 수 없다는 본질적인 한계를 가진다. 심지어, 종양의 성장과 같이 국소적인 변화가 클 경우, ICP는 이 변화를 '정합 오차'로 간주하여 이를 최소화하려다 오히려 주변의 정상 조직 정렬을 망가뜨리는 역설적인 결과를 낳기도 한다.45
- **해결 전략:** 이러한 문제를 해결하기 위해 의료 영상 분야에서는 ICP를 그대로 사용하기보다는, 도메인 지식을 결합한 다양한 변형 및 보완 전략이 개발되었다.
  - **안정 영역 기반 정합 (Stable Region-based Registration):** 뇌의 두개골이나 척추뼈와 같이 상대적으로 변형이 적은 강체 구조물을 기준으로 먼저 ICP 정합을 수행하고, 이를 바탕으로 부드러운 조직의 비강체 변형을 추정하는 단계적 접근법을 사용한다.45
  - **데이터 차감 (Data Subtraction):** 두 시점의 영상을 ICP로 정합한 후, 정합된 두 영상의 픽셀(복셀) 값이나 표면 데이터를 서로 빼서 그 차이(residual)만을 분석하는 기법이다. 이 방법은 ICP로 인해 발생하는 미세한 정합 오차를 상쇄시키고, 실제 조직의 변화량만을 더 명확하게 시각화하는 데 도움을 준다.45
  - **수정된 ICP 및 하이브리드 접근:** ICP 알고리즘 자체에 비강체 변형을 허용하는 항을 추가하거나, ICP를 초기 정렬에만 사용하고 이후 비강체 정합 알고리즘(예: Deformable ICP, Coherent Point Drift)으로 전환하는 하이브리드 방식을 사용한다.

이처럼 ICP는 다양한 응용 분야의 특성에 맞춰 진화해왔다. 이는 ICP의 기본 철학이 얼마나 유연하고 확장 가능한지를 보여주는 증거이다. 동시에, 의료 영상 분야의 사례는 성공적인 응용이 단순히 알고리즘의 기하학적 성능뿐만 아니라, 해당 분야의 고유한 문제(도메인 지식)를 깊이 이해하고 이를 알고리즘 설계에 반영하는 능력에 달려있음을 명확히 시사한다.


Iterative Closest Point 알고리즘은 지난 30여 년간 3차원 컴퓨터 비전과 로보틱스 분야에서 가장 근본적이고 영향력 있는 기술 중 하나로 그 자리를 굳건히 지켜왔다. 본 보고서는 ICP의 수학적 원리에서부터 시작하여, 주요 변형 알고리즘, 근본적인 한계와 이를 극복하기 위한 전략, 딥러닝 기반의 현대적 접근법, 그리고 다양한 응용 분야에서의 활용까지 다각도로 심층적인 고찰을 수행했다.


ICP의 여정은 '정합'이라는 근본적인 문제를 해결하기 위한 끊임없는 진화의 과정이었다. 초기의 Point-to-Point ICP는 SVD를 이용한 우아한 수학적 해법을 제시했지만, '가장 가까운 점이 곧 대응점'이라는 단순한 가정에 발목이 잡혔다. 이에 대한 해결책으로, Point-to-Plane ICP는 1차 미분 정보인 '법선 벡터'를 도입하여 기하학적 맥락을 추가했고, G-ICP는 한 걸음 더 나아가 2차 통계 정보인 '공분산'을 통해 불확실성까지 모델링하는 확률적 프레임워크를 제시했다. 이는 오차의 정의를 점차 정교화하며 강인성을 확보해 온 명백한 발전의 궤적이다.

그러나 ICP는 본질적으로 지역 최적화 알고리즘이라는 태생적 한계를 벗어날 수 없었다. 이 문제를 해결하기 위해 FPFH+RANSAC과 같은 특징 기반 전역 정합 알고리즘이 ICP의 '눈' 역할을 하는 Coarse-to-Fine 패러다임이 표준으로 자리 잡았다. 이 파이프라인에서 ICP는 전역 탐색의 부담을 덜고, 자신이 가장 잘하는 '정밀한 지역 정제' 역할에 집중하게 되었다.

최근 딥러닝의 등장은 이 패러다임에 또 한 번의 변화를 가져왔다. 딥러닝은 수작업으로 특징을 설계하던 과정을 자동화하고, 데이터로부터 직접 더 강력하고 강인한 특징을 학습함으로써 정합 성능을 새로운 차원으로 끌어올렸다. 현재 ICP는 단독으로 사용되기보다는, 딥러닝으로 추출된 우수한 특징과 RANSAC의 강인한 추정 능력, 그리고 ICP의 정밀한 정제 능력이 결합된 하이브리드 파이프라인의 핵심 부품으로서 그 위상을 유지하고 있다.


ICP와 점군 정합 분야는 여전히 해결해야 할 도전 과제와 함께 흥미로운 연구 기회들을 안고 있다. 미래 연구는 다음과 같은 방향으로 나아갈 것으로 전망된다.

- **딥러닝과의 깊은 융합:** 현재의 하이브리드 방식을 넘어, 레이블이 없는 대규모 데이터셋을 활용하여 스스로 학습하는 비지도(unsupervised) 및 자기지도(self-supervised) 학습 기반의 엔드투엔드 정합 모델 개발이 가속화될 것이다.40 이는 데이터 구축 비용을 절감하고, 학습되지 않은 환경에 대한 일반화 성능을 높이는 데 핵심적인 역할을 할 것이다.
- **의미론적 정보의 활용 (Semantic-Aware Registration):** 기존의 정합이 순수한 기하학적 형태에만 의존했다면, 미래의 정합은 '이것은 문이다', '저것은 의자다'와 같은 객체의 의미(semantics)를 이해하고 이를 정합 과정에 활용할 것이다.41 의미론적 정보는 기하학적으로 유사하지만 다른 객체들을 구분하고, 대응점 탐색의 정확도를 비약적으로 향상시켜 정합의 강인성을 한 차원 높일 수 있다.
- **퇴화 환경 극복:** 긴 복도나 넓은 터널과 같이 기하학적 제약이 부족한 퇴화(degenerate) 환경은 여전히 SLAM 및 정합 기술의 주요 실패 원인 중 하나이다. Point-to-Point와 Point-to-Plane의 장점을 동적으로 결합하는 GenZ-ICP와 같은 하이브리드 비용 함수 접근법 28이나, 관성 측정 장치(IMU)와 같은 다른 센서 정보를 융합하여 기하학적 모호성을 해결하려는 연구가 더욱 중요해질 것이다.
- **실시간 대규모 정합:** 증강 현실(AR) 및 자율 주행과 같은 실시간 응용을 위해, 수백만 개 이상의 점으로 구성된 대규모 점군을 지연 없이 처리하는 것은 여전히 큰 도전이다. 이를 위해 알고리즘의 경량화, 효율적인 자료구조의 개발, 그리고 GPU 및 전용 하드웨어를 활용한 가속 연구가 지속적으로 이루어질 것이다.
- **다중 모달리티 융합 (Multi-modal Fusion):** LiDAR 점군의 정밀한 3D 구조 정보와 카메라 이미지의 풍부한 텍스처 및 색상 정보를 함께 사용하여 정합의 정확도와 강인성을 높이는 연구가 더욱 활발해질 것이다.23 서로 다른 센서 데이터의 장점을 상호 보완적으로 활용하는 것은 악천후나 조명 변화가 심한 환경에서의 성능을 개선하는 데 필수적이다.

결론적으로, ICP는 죽지 않는다. 다만 끊임없이 진화할 뿐이다. ICP라는 이름이나 그 핵심적인 반복 정제 철학은 앞으로도 오랫동안 살아남을 것이다. 미래의 지능형 정합 시스템은 아마도 딥러닝으로 의미론적 대응점을 찾고, 확률적 프레임워크로 불확실성을 모델링하며, 최종적으로 ICP의 반복적 정제 과정을 거쳐 완벽에 가까운 결과를 도출하는 형태가 될 가능성이 높다. 즉, ICP는 단독 주연의 자리에서는 내려올지라도, 더 크고 지능적인 시스템의 신뢰성을 보장하는 필수적인 '마지막 1마일(last mile)' 정제 모듈로서 그 중요한 역할을 계속해서 수행해 나갈 것이다.


1. NOTES ON ITERATIVE CLOSEST POINT ALGORITHM - EVLM, accessed July 24, 2025, http://evlm.stuba.sk/APLIMAT2018/proceedings/Papers/0876_Prochazkova_Martisek.pdf
2. A K-Dimensional Tree–Iterative Closest Point Algorithm for Overbreak and Underbreak Assessment of Mountain Tunnels - MDPI, accessed July 24, 2025, https://www.mdpi.com/2076-3417/15/2/566
3. Point cloud registration: a mini-review of current state, challenging issues and future directions - AIMS Press, accessed July 24, 2025, https://www.aimspress.com/article/doi/10.3934/geosci.2023005?viewType=HTML
4. Understanding Iterative Closest Point (ICP) Algorithm with Code - LearnOpenCV, accessed July 24, 2025, https://learnopencv.com/iterative-closest-point-icp-explained/
5. Iterative closest point - Wikipedia, accessed July 24, 2025, https://en.wikipedia.org/wiki/Iterative_closest_point
6. ICP Algorithm: Theory, Practice And Its SLAM-oriented Taxonomy - arXiv, accessed July 24, 2025, https://arxiv.org/pdf/2206.06435
7. 3D registration process of ICP using linear constraint - SciSpace, accessed July 24, 2025, https://scispace.com/pdf/3d-registration-process-of-icp-using-linear-constraint-1bl9qv22.pdf
8. SLAM for Autonomous Driving: Concept and Analysis | Encyclopedia MDPI, accessed July 24, 2025, https://encyclopedia.pub/entry/history/compare_revision/96146
9. (PDF) ICP Algorithm: Theory, Practice and Its SLAM-oriented ..., accessed July 24, 2025, https://www.researchgate.net/publication/375869995_ICP_Algorithm_Theory_Practice_and_Its_SLAM-oriented_Taxonomy
10. What is Iterative Closest Point (ICP) - Activeloop, accessed July 24, 2025, https://www.activeloop.ai/resources/glossary/iterative-closest-point-icp/
11. Medical image registration using modified iterative closest points | Request PDF, accessed July 24, 2025, https://www.researchgate.net/publication/230274102_Medical_image_registration_using_modified_iterative_closest_points
12. learnopencv.com, accessed July 24, 2025, [https://learnopencv.com/iterative-closest-point-icp-explained/#:~:text=The%20intuitive%20idea%20behind%20ICP,and%20they%20will%20perfectly%20align.](https://learnopencv.com/iterative-closest-point-icp-explained/#:~:text=The intuitive idea behind ICP,and they will perfectly align.)
13. SANDRO: a Robust Solver with a Splitting Strategy for Point Cloud Registration - arXiv, accessed July 24, 2025, https://arxiv.org/html/2503.07743v1
14. Point Cloud Alignment Alignment of 3D Data Points Scan Alignment in Mapping Iterative Closest Point (I, accessed July 24, 2025, https://www.ipb.uni-bonn.de/html/teaching/msr2-2020/sse2-03-icp.pdf
15. An Iterative Closest Points Algorithm for Registration of 3D Laser Scanner Point Clouds with Geometric Features, accessed July 24, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC5580094/
16. pcregistericp - Register two point clouds using ICP algorithm - MATLAB - MathWorks, accessed July 24, 2025, https://www.mathworks.com/help/vision/ref/pcregistericp.html
17. ICP registration - Open3D 0.19.0 documentation, accessed July 24, 2025, https://www.open3d.org/docs/release/tutorial/pipelines/icp_registration.html
18. Iterative Closest Point Algorithm Introduction to Mobile Robotics - GMU CS Department, accessed July 24, 2025, https://cs.gmu.edu/~kosecka/cs685/cs685-icp.pdf
19. ICP & Point Cloud Registration - Part 1: Known Data Association & SVD (Cyrill Stachniss, 2021) - YouTube, accessed July 24, 2025, https://www.youtube.com/watch?v=dhzLQfDBx2Q
20. Linear Least-Squares Optimization for Point-to ... - NUS Computing, accessed July 24, 2025, https://www.comp.nus.edu.sg/~lowkl/publications/lowk_point-to-plane_icp_techrep.pdf
21. ICP point-to-plane odometry algorithm - OpenCV Documentation, accessed July 24, 2025, https://docs.opencv.org/4.x/d7/dbe/kinfu_icp.html
22. Generalized-ICP - Robotics, accessed July 24, 2025, https://www.roboticsproceedings.org/rss05/p21.pdf
23. Visually Bootstrapped Generalized ICP, accessed July 24, 2025, https://svl.stanford.edu/assets/publications/pdfs/gpandey-2011b.pdf
24. Point-to-Plane and Generalized ICP - 5 Minutes with Cyrill - YouTube, accessed July 24, 2025, https://www.youtube.com/watch?v=2hC9IG6MFD0
25. Generalized ICP - Wiki, accessed July 24, 2025, https://wiki.hanzheteng.com/algorithm/slam/generalized-icp
26. An Iterative Closest Points Algorithm for Registration of 3D Laser Scanner Point Clouds with Geometric Features - MDPI, accessed July 24, 2025, https://www.mdpi.com/1424-8220/17/8/1862
27. (PDF) Centralized RANSAC Based Point Cloud Registration with Fast Convergence and High Accuracy - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/378195884_Centralized_RANSAC_Based_Point_Cloud_Registration_with_Fast_Convergence_and_High_Accuracy
28. GenZ-ICP: Generalizable and Degeneracy-Robust LiDAR Odometry Using an Adaptive Weighting - arXiv, accessed July 24, 2025, https://arxiv.org/html/2411.06766v1
29. Global registration - Open3D 0.19.0 documentation, accessed July 24, 2025, https://www.open3d.org/docs/release/tutorial/pipelines/global_registration.html
30. Comparison of Point Cloud Registration Techniques on Scanned ..., accessed July 24, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11014384/
31. An Improved RANSAC-ICP Method for Registration of SLAM and UAV-LiDAR Point Cloud at Plot Scale - MDPI, accessed July 24, 2025, https://www.mdpi.com/1999-4907/15/6/893
32. Deep Learning-Based Point Cloud Registration: A Comprehensive Survey and Taxonomy, accessed July 24, 2025, https://arxiv.org/html/2404.13830v3
33. A review of rigid point cloud registration based on deep learning - Frontiers, accessed July 24, 2025, https://www.frontiersin.org/journals/neurorobotics/articles/10.3389/fnbot.2023.1281332/full
34. A Comprehensive Survey and Taxonomy on Point Cloud Registration Based on Deep Learning - arXiv, accessed July 24, 2025, https://arxiv.org/html/2404.13830v2
35. Deep learning-based point cloud registration: a comprehensive investigation | Request PDF, accessed July 24, 2025, https://www.researchgate.net/publication/380437128_Deep_learning-based_point_cloud_registration_a_comprehensive_investigation
36. DeepMatch: Toward Lightweight in Point Cloud Registration - Frontiers, accessed July 24, 2025, https://www.frontiersin.org/journals/neurorobotics/articles/10.3389/fnbot.2022.891158/pdf
37. Review on Deep Learning Algorithms and Benchmark Datasets for Pairwise Global Point Cloud Registration - MDPI, accessed July 24, 2025, https://www.mdpi.com/2072-4292/15/8/2060
38. Deep Global Registration - CVF Open Access, accessed July 24, 2025, https://openaccess.thecvf.com/content_CVPR_2020/papers/Choy_Deep_Global_Registration_CVPR_2020_paper.pdf
39. DGR vs FCGF / Issue #7 / chrischoy/DeepGlobalRegistration - GitHub, accessed July 24, 2025, https://github.com/chrischoy/DeepGlobalRegistration/issues/7
40. [Literature Review] Deep Learning-Based Point Cloud Registration: A Comprehensive Survey and Taxonomy - Moonlight, accessed July 24, 2025, https://www.themoonlight.io/en/review/deep-learning-based-point-cloud-registration-a-comprehensive-survey-and-taxonomy
41. A Review of SLAM Techniques and Security in Autonomous Driving - Advanced Robotics and Automation (ARA) Laboratory - University of Nevada, Reno, accessed July 24, 2025, https://ara.cse.unr.edu/wp-content/uploads/2014/12/Singandhupe_La_IRC18.pdf
42. Implementation of ICP Slam Algorithm on Fire Bird V for Mapping of an Indoor Environment - Atlantis Press, accessed July 24, 2025, https://www.atlantis-press.com/article/125960843.pdf
43. Simultaneous Localization and Mapping (SLAM) for Autonomous Driving: Concept and Analysis - MDPI, accessed July 24, 2025, https://www.mdpi.com/2072-4292/15/4/1156
44. Medical Image Registration Based on Inertia Matrix and Iterative Closest Point - NADIA, accessed July 24, 2025, http://article.nadiapub.com/IJSIP/vol8_no8/3.pdf
45. King's Research Portal - the King's College London Research Portal, accessed July 24, 2025, https://kclpure.kcl.ac.uk/portal/files/245163528/1-s2.0-S0300571224000332-main.pdf

