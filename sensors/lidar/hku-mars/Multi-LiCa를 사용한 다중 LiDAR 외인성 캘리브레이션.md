[HKU-MaRS](./index.md)
# Multi-LiCa를 사용한 다중 LiDAR 외인성 캘리브레이션에 대한 고찰



자율주행 시스템의 안전하고 신뢰성 있는 운행은 주변 환경에 대한 포괄적이고 정확한 인지 능력에 달려있다. 그러나 단일 종류의 센서만으로는 이러한 요구사항을 충족시키기 어렵다. 각 센서는 고유의 물리적 원리에 기반하므로 본질적인 한계를 내포하고 있기 때문이다.1 예를 들어, 카메라는 고해상도의 색상 및 텍스처 정보를 제공하지만 조명 변화나 악천후에 취약하며, 정확한 3차원 거리 정보를 직접 얻기 어렵다.2 반면, 라이다(LiDAR)는 레이저 펄스를 이용하여 주변 환경의 정확한 3차원 형상 정보를 획득할 수 있고 조명 조건에 거의 영향을 받지 않지만, 데이터가 희소(sparse)하고 악천후 시 성능이 저하될 수 있다.1 레이더(Radar)는 악천후 조건에서도 강인한 탐지 성능을 보이지만, 해상도가 낮아 객체의 세밀한 형상을 파악하기는 어렵다.3

이러한 개별 센서의 한계를 극복하고 인지 시스템의 강인성(robustness)과 신뢰도를 극대화하기 위한 핵심 전략이 바로 다중 센서 융합(multi-sensor fusion)이다.4 서로 다른 종류의 센서 데이터를 통합함으로써, 각 센서의 장점은 강화하고 단점은 상호 보완하는 시너지를 창출할 수 있다.2 즉, 센서 융합은 자율주행 차량이 어떠한 주행 환경에서도 안정적인 인지 성능을 유지하기 위한 필수불가결한 기술이라 할 수 있다.


다중 센서 융합을 성공적으로 수행하기 위한 가장 근본적인 전제 조건은 각 센서 데이터의 공간적 정합, 즉 외인성 캘리브레이션(extrinsic calibration)이다.1 외인성 캘리브레이션이란, 다중 센서 시스템을 구성하는 각 센서의 좌표계(coordinate system) 간의 상대적인 기하학적 관계, 즉 위치(translation)와 방향(rotation)을 나타내는 6자유도(6-DoF) 강체 변환(rigid-body transformation) 행렬을 정밀하게 추정하는 과정을 의미한다.

이 변환 행렬의 정확도는 전체 자율주행 시스템의 성능을 좌우하는 '초석(cornerstone)'과도 같다.8 만약 캘리브레이션에 미세한 오차라도 존재한다면, 이는 센서로부터 멀리 떨어진 지점에서 심각한 투영 오차(reprojection error)로 증폭될 수 있다. 예를 들어, 불과 몇 도의 회전 오차나 몇 퍼센트의 이동 오차가 5m 거리에서 20cm 이상의 위치 오차를 유발할 수 있다.10 이러한 오차는 객체 탐지 실패, 장애물의 위치 오인식, 나아가 잘못된 주행 경로 계획 및 제어로 이어져 시스템의 안전을 심각하게 위협할 수 있다.8 따라서, 고정밀 외인성 캘리브레이션은 신뢰성 있는 센서 융합과 안전한 자율주행을 위한 가장 기본적인 선결 과제이다.


외인성 캘리브레이션 기술은 크게 타겟 기반(target-based) 방식과 타겟리스(targetless) 방식으로 분류할 수 있다.

**타겟 기반 방법론**은 체커보드(checkerboard), ChArUco 보드, 구체(sphere) 등 사전에 기하학적 정보가 정확히 알려진 인공물을 사용하여 캘리브레이션을 수행한다.8 이 방식은 명확한 기하학적 제약 조건을 통해 3D-2D 또는 3D-3D 대응점(correspondence)을 명시적으로 찾을 수 있어, 통제된 환경에서는 잠재적으로 매우 높은 정확도를 달성할 수 있다는 장점이 있다.12 그러나 타겟을 정밀하게 설치하고 여러 위치에서 데이터를 수집하는 과정에 많은 수동적 개입이 필요하며 6, 이는 시간과 비용을 증가시키는 요인이 된다. 또한, 주행 중 발생하는 차량의 진동이나 온도 변화로 인해 센서의 정렬 상태가 미세하게 틀어지는 현상을 실시간으로 보정하는 온라인(online) 적용이 거의 불가능하다는 근본적인 한계를 가진다.14

**타겟리스 방법론**은 이러한 한계를 극복하기 위해 별도의 타겟 없이 주변 환경에 존재하는 자연적인 특징(natural features)을 활용한다.8 환경에 존재하는 평면, 모서리, 혹은 점군 데이터의 정보량(information content) 등을 이용하여 센서 간의 변환 관계를 추정하는 방식이다. 이 접근법은 자동화가 용이하고 별도의 장비가 필요 없어 유연성이 높으며, 온라인 캘리브레이션으로 확장될 잠재력을 지닌다.14 하지만 환경의 구조에 크게 의존한다는 단점이 있다. 예를 들어, 기하학적 특징이 부족한 평지나 대칭적인 구조를 가진 터널과 같은 환경에서는 알고리즘이 실패하거나 부정확한 결과를 도출할 수 있다.16 또한, 많은 타겟리스 방법들은 변환 행렬에 대한 합리적인 초기 추정치(initial guess)를 요구하며, 이 초기치가 부정확할 경우 지역 최적해(local minima)에 수렴하여 캘리브레이션에 실패할 위험이 있다.9


이러한 배경 속에서 Multi-LiCa(Multi-LiDAR Calibration) 프레임워크가 등장했다. 그 직접적인 동기는 기존 캘리브레이션 방법론으로는 해결하기 어려운 새로운 형태의 센서 아키텍처가 출현했기 때문이다. 뮌헨 공과대학교(TUM)의 자율주행 연구용 차량인 'EDGAR(Excellent Driving Garching)'는 전통적인 차량 지붕 중앙에 고가의 360° LiDAR 하나를 장착하는 방식에서 탈피했다.20 대신, 2대의 회전식 중거리 LiDAR(Ouster OS-1)와 2대의 고정형(solid-state) 장거리 LiDAR(Seyond Falcon)를 차량 루프 프레임 주위에 비스듬히(angled) 분산 배치하는 독특하고 비정형적인(atypical) 구성을 채택했다.20

이러한 설계는 차량 주변의 사각지대를 최소화하고 360° 전방위 인지 능력을 확보하기 위한 전략적 선택이었지만, 동시에 중대한 기술적 난제를 야기했다. 즉, 시스템을 구성하는 모든 LiDAR 센서가 동시에 공유하는 시야(Field of View, FOV)가 존재하지 않는다는 문제이다.12 기존의 많은 다중 LiDAR 캘리브레이션 도구들은 하나의 주 센서(main LiDAR)를 기준으로 다른 모든 센서와의 FOV 중첩을 가정하고 설계되었기 때문에, EDGAR와 같은 비정형 구성에는 적용할 수 없었다.20

따라서, 모든 센서 간의 직접적인 FOV 중첩이나 변환 행렬에 대한 초기 추정치 없이도, 다양한 센서 배치를 강인하게 처리할 수 있는 새로운 자동 캘리브레이션 프레임워크의 필요성이 절실해졌다. Multi-LiCa는 바로 이러한 현실적인 문제에 대한 해답으로 제안되었다.20 이는 Multi-LiCa가 단순히 기존 알고리즘의 성능을 소폭 개선한 것이 아니라, 자율주행차 센서 아키텍처 설계 패러다임의 변화에 발맞춘 필연적인 기술적 진보임을 시사한다. 고가의 중앙 집중식 센서에서 벗어나, 비용 효율적이고 특화된 다수의 센서를 유연하게 조합하는 새로운 설계 철학이 부상함에 따라, 이를 뒷받침할 수 있는 소프트웨어 기술의 중요성이 부각된 것이다. Multi-LiCa는 바로 이 '비중첩 FOV'라는 새로운 문제를 해결함으로써, 미래의 유연하고 확장 가능한 센서 아키텍처 설계의 실용성을 뒷받침하는 핵심 기술(enabling technology)로서의 가치를 지닌다.



Multi-LiCa 프레임워크는 캘리브레이션 과정에서 사용자의 개입을 최소화하고, 어떠한 환경에서도 일관된 성능을 제공하는 것을 목표로 설계되었다. 이를 위해 세 가지 핵심 철학을 채택했다.12

1. **모션리스 (Motionless):** 캘리브레이션을 위한 데이터 수집 과정에서 차량이나 센서의 어떠한 움직임도 요구하지 않는다. 이는 캘리브레이션을 위해 특정 경로를 주행하거나 복잡한 움직임을 수행할 필요 없이, 정지된 상태에서 단 한 번의 스캔 데이터만으로 캘리브레이션을 완료할 수 있음을 의미한다.24
2. **타겟리스 (Targetless):** 체커보드와 같은 인공적인 캘리브레이션 타겟을 전혀 사용하지 않는다. 대신, 주차장, 건물 주변 등 일상적인 환경에 존재하는 자연적인 기하학적 구조(벽, 기둥, 차량 등)를 특징으로 활용한다.20
3. **초기 추정치 불필요 (No Initial Guess):** Multi-LiCa의 가장 강력한 특징 중 하나로, 센서 간의 상대적 위치 및 방향에 대한 어떠한 사전 정보나 초기 추정값도 필요로 하지 않는다. 이는 사용자가 대략적인 변환 행렬을 수동으로 입력해야 하는 부담을 없애고, 초기 추정치의 정확도에 따라 성능이 크게 좌우되는 기존 알고리즘들의 근본적인 한계를 극복한다.20

이 세 가지 원칙은 Multi-LiCa를 매우 실용적이고 강인한 도구로 만들어주며, 전문 지식이 없는 사용자도 쉽고 빠르게 정확한 캘리브레이션을 수행할 수 있도록 한다.


Multi-LiCa는 복잡한 점군 정합 문제를 해결하기 위해, 효과성이 입증된 '조대-정밀(Coarse-to-Fine)' 2단계 전략을 채택한다.12 이는 마치 안개 속에서 목표 지점을 찾을 때, 먼저 넓은 시야로 대략적인 방향을 잡고(조대 정합), 목표 지점에 가까워졌을 때 세밀하게 위치를 조정하는(정밀 정합) 과정과 유사하다.

1. **1단계 - 조대 정합 (Coarse Alignment):** 두 점군 간의 상대적 자세 차이가 매우 크고, 수많은 잘못된 대응점(outliers)이 존재할 수 있는 상황에서 대략적인 변환 관계를 추정한다. 이 단계의 핵심 목표는 후속 정밀 정합 알고리즘이 지역 최적해(local minimum)에 빠지지 않고 전역 최적해(global minimum)로 안정적으로 수렴할 수 있는 영역(basin of convergence) 내로 변환 행렬을 초기화하는 것이다. 이를 위해 FPFH(Fast Point Feature Histograms) 특징 기술자와 전역적으로 강인한 TEASER++ 정합 알고리즘이 사용된다.24
2. **2단계 - 정밀 정합 (Fine Registration):** 조대 정합을 통해 얻은 양질의 초기 변환값을 바탕으로, 점군의 미세한 기하학적 구조를 고려하여 변환 행렬을 정밀하게 최적화한다. 이 단계에서는 점군을 국소적인 평면의 집합으로 간주하여 정합하는 Generalized-ICP(GICP) 알고리즘을 사용하여 높은 정확도를 달성한다.20

이러한 2단계 구조는 각 단계가 후속 단계의 근본적인 약점을 보완하도록 설계된 유기적인 시스템이다. ICP 계열 알고리즘(GICP 포함)의 가장 큰 약점은 초기 추정치에 대한 높은 민감성이다.9 Multi-LiCa는 이 문제를 해결하기 위해, 1단계에서 극심한 이상치 환경에서도 전역적으로 강인한 해를 제공하는 TEASER++를 배치했다.27 즉, 1단계는 2단계의 성공적인 수렴을 보장하는 '안전장치' 역할을 수행하며, 이 구조 덕분에 전체 프레임워크는 '초기 추정치 불필요'라는 강력한 장점을 가질 수 있게 된다. 이는 '전역적으로 강인한 해 탐색기'가 '국소적으로 정밀한 최적화기'를 안내하는 구조로, 1단계가 2단계의 실패 확률을 원천적으로 차단하는 상호 보완적 설계의 정수라 할 수 있다.


Multi-LiCa는 단순히 두 개의 LiDAR를 캘리브레이션하는 것을 넘어, 여러 개의 LiDAR를 하나의 일관된 좌표계로 통합하기 위한 지능적인 전략을 포함한다.12

정밀 정합(GICP)을 수행한 후에는 두 점군이 얼마나 잘 정렬되었는지를 나타내는 '적합도 점수(fitness score)'가 계산된다.25 Multi-LiCa는 이 점수를 일종의 비용(cost)으로 간주하여 정합의 성공 여부를 판단한다. 만약 적합도 점수가 사전에 정의된 임계값(예: 0.2)을 넘으면 해당 캘리브레이션은 성공한 것으로 간주된다.12

성공적으로 정합된 소스(source) 점군은 계산된 변환 행렬을 통해 타겟(target) 점군의 좌표계로 변환된 후, 기존 타겟 점군과 하나로 병합(merge)된다. 이렇게 생성된 '병합 점군'은 더 넓은 영역을 커버하고 더 풍부한 기하학적 정보를 담게 되며, 다음 캘리브레이션 단계에서 새로운 타겟 점군 역할을 수행한다. 이 과정을 반복함으로써, 기준이 되는 타겟 LiDAR와 직접적인 FOV 중첩이 없는 센서들까지도 마치 징검다리를 건너듯 연쇄적으로 캘리브레이션할 수 있게 된다.12


조대 정합 단계의 목표는 센서 간의 상대적 자세에 대한 사전 정보가 전혀 없는 상태에서, 대략적이지만 전역적으로 올바른 변환 행렬을 추정하는 것이다. 이 과정은 데이터 전처리, 특징 추출, 강인한 매칭의 세 단계로 구성된다.


원시(raw) LiDAR 점군은 수백만 개의 포인트로 구성될 수 있어 직접 처리하기에는 계산 비용이 매우 높다. 또한, 센서로부터의 거리에 따라 포인트의 밀도가 불균일하게 나타난다. 이러한 문제를 해결하기 위해 Multi-LiCa는 복셀화(voxelization)를 통한 다운샘플링을 수행한다.25

복셀화는 3차원 공간을 정육면체 형태의 작은 격자, 즉 복셀(voxel)로 나눈 뒤, 각 복셀 내에 포함된 모든 포인트들의 기하학적 중심점(centroid) 하나로 해당 복셀을 대표하게 하는 기법이다. 이를 통해 전체 포인트 수를 크게 줄여 계산 효율성을 높이는 동시에, 점군의 밀도를 공간적으로 균일하게 만드는 정규화 효과를 얻을 수 있다. Multi-LiCa에서는 경험적으로 0.35m 크기의 복셀을 사용하여 효과적인 다운샘플링을 수행한다.25 이 과정을 통해 생성된 중심점들이 특징 추출을 위한 키포인트(keypoint)가 된다.


다음 단계는 각 키포인트의 주변 기하학적 특징을 수치적으로 표현하는 것이다. 이를 위해 Multi-LiCa는 FPFH(Fast Point Feature Histograms)라는 강력한 3D 특징 기술자(feature descriptor)를 사용한다.24 FPFH는 특정 포인트 주변의 곡률과 표면 법선(surface normal)의 변화를 히스토그램 형태로 인코딩한 것으로, 점군의 이동(translation) 및 회전(rotation)에 대해 불변(invariant)하는 특성을 가진다.29 이 불변성 덕분에 두 점군의 상대적 자세가 어떻든 간에, 유사한 기하학적 형상을 가진 지점들은 유사한 FPFH 값을 갖게 되어 효과적인 매칭이 가능하다.

FPFH의 계산 과정은 두 단계로 이루어진다 30:

1. **SPFH (Simplified Point Feature Histogram) 계산:** 먼저, 각 키포인트 $p_q$와 그 주변의 k-개의 이웃 포인트 $p_k$들 간의 기하학적 관계(법선 벡터 간의 3가지 각도 차이)를 계산하여 단순화된 히스토그램인 SPFH를 생성한다. 이 단계는 $p_q$와 직접적인 이웃 간의 관계만을 고려하여 계산량을 줄인다.

2. **FPFH 최종 계산:** 그 후, 키포인트 $p_q$의 최종 FPFH는 자신의 SPFH와 그 이웃 포인트들의 SPFH를 거리 기반 가중치를 적용하여 합산함으로써 계산된다. 수학적으로는 다음과 같이 표현할 수 있다 30:
   $$
   \text{FPFH}(p_q) = \text{SPFH}(p_q) + \frac{1}{k} \sum_{i=1}^{k} \frac{1}{w_i} \text{SPFH}(p_{k_i})
   $$
   여기서 $k$는 이웃 포인트의 수, $w_i$는 $p_q$와 이웃 포인트 $p_{k_i}$ 간의 거리에 따른 가중치이다. 이 2단계 접근법은 계산 효율성을 크게 높이면서도(기존 PFH의 계산 복잡도 $O(nk^2)$를 $O(nk)$로 감소) 31, 주변 영역의 정보를 종합적으로 반영하여 특징의 표현력을 유지한다.


두 점군의 키포인트들로부터 FPFH 디스크립터가 계산되면, 각 소스 점군의 키포인트에 대해 타겟 점군에서 가장 유사한 FPFH 값을 가진 키포인트를 찾아 대응점(correspondence) 쌍을 생성한다. 그러나 이 과정에서는 필연적으로 수많은 잘못된 매칭, 즉 이상치(outlier)가 발생한다. 특히, 환경에 반복적인 구조가 많을 경우 이상치 비율은 99%를 넘을 수도 있다.

Multi-LiCa는 이러한 극심한 이상치 환경에서도 정확한 변환 행렬을 추정하기 위해 TEASER++라는 최첨단 알고리즘을 사용한다.24 TEASER++는 '빠르고 증명 가능한(fast and certifiable)' 3D 정합을 목표로 개발된 알고리즘으로, 대규모 이상치에 대한 탁월한 강인성을 자랑한다.27

TEASER++의 핵심 전략은 다음과 같다 28:

1. **변환 분리 (Decoupling):** 6-DoF 변환 문제를 스케일(scale), 회전(rotation), 이동(translation)의 세 가지 하위 문제로 분리하여 순차적으로 해결한다.
2. **스케일 및 이상치 제거:** 먼저 모든 대응점 쌍에 대해 이동 및 회전에 불변인 거리 비율을 계산한다. 이 값들은 이론적으로 모두 동일한 스케일 값을 가져야 한다. 이 정보를 바탕으로 그래프를 구성하고, 최대 클리크 찾기(maximum clique finding)와 같은 기법을 통해 스케일 값에 일관성을 보이는 최대 규모의 정상치(inlier) 집합을 찾아낸다. 이 과정에서 대부분의 이상치가 효과적으로 제거된다.
3. **회전 추정:** 앞서 찾아낸 정상치 집합을 이용하여 회전 행렬을 추정한다. 이 문제는 본질적으로 비볼록(non-convex) 최적화 문제이지만, TEASER++는 이를 반정부호 계획법(Semidefinite Programming, SDP)이라는 볼록 최적화 문제로 완화(relaxation)하여 전역 최적해를 구한다. 또한, Graduated Non-Convexity (GNC)라는 기법을 통해 훨씬 더 빠른 속도로 해를 구하고, 그 해의 최적성을 수학적으로 증명(certify)할 수도 있다.33
4. **이동 추정:** 스케일과 회전이 결정된 후에는, 이동 벡터를 간단한 최소 자승법(least squares)을 통해 빠르고 정확하게 계산할 수 있다.

TEASER++를 조대 정합 단계에 채택한 것은 Multi-LiCa가 '실패하지 않는' 캘리브레이션을 지향함을 보여주는 중요한 대목이다. 일반적인 RANSAC 기반 접근법은 이상치 비율이 특정 임계치를 넘으면 실패 확률이 급격히 증가하는 반면, TEASER++는 반환된 해가 전역 최적해임을 수학적으로 보장하거나 최적해와의 오차 한계를 제공하는 '증명 가능한' 알고리즘이다.28 이는 안전이 최우선인 자율주행 분야에서 캘리브레이션 결과의 신뢰도를 보증하는 매우 중요한 특성이다. 즉, Multi-LiCa는 속도나 단순함보다 '신뢰성'과 '보증된 성능'에 더 높은 가치를 둔 설계 철학을 반영한다.


조대 정합 단계를 통해 두 점군 간의 대략적인 변환 관계가 설정되면, 정밀 정합 단계에서는 이 초기값을 바탕으로 더욱 세밀하고 정확한 변환 행렬을 계산한다. Multi-LiCa는 이 단계를 위해 GICP(Generalized Iterative Closest Point) 알고리즘을 사용한다.


전통적인 점군 정합 알고리즘인 ICP(Iterative Closest Point)는 그 변종들과 함께 GICP의 필요성을 이해하는 데 좋은 배경을 제공한다.

- **표준 ICP (Point-to-Point):** 소스 점군의 각 포인트를 타겟 점군에서 가장 가까운 포인트에 대응시킨 후, 이 대응점들 사이의 유클리드 거리 제곱의 합을 최소화하는 변환 행렬을 반복적으로 계산한다. 이 방식은 직관적이고 구현이 간단하지만, 점군이 가지는 표면의 기하학적 구조를 전혀 고려하지 않기 때문에 노이즈에 민감하고 수렴 속도가 느릴 수 있다.26
- **Point-to-Plane ICP:** 한 단계 발전하여, 소스 점군의 포인트와 타겟 점군의 대응 포인트가 놓여 있는 국소 평면(local plane) 사이의 거리를 최소화한다. 이는 표면 정보를 활용하므로 표준 ICP보다 일반적으로 더 빠르고 정확하게 수렴한다. 하지만 소스 점군의 표면 구조는 무시하고 타겟 점군의 구조만 고려하는 비대칭적인 접근법이라는 한계가 있다.26
- **GICP (Generalized-ICP, Plane-to-Plane):** ICP 계열 알고리즘의 최종 진화형으로, 소스와 타겟 양쪽 점군 모두에서 국소적인 표면 구조를 고려한다. 각 포인트 주변의 분포를 확률적 모델, 즉 가우시안 분포로 근사하고, 이 분포를 나타내는 공분산 행렬을 추정한다. 정합 과정은 두 점군에 있는 대응되는 평면 패치(planar patch) 간의 거리를 최소화하는 방식으로 이루어진다. 이 대칭적이고 확률적인 접근 방식 덕분에 GICP는 노이즈, 이상치, 그리고 서로 다른 밀도를 가진 점군 데이터에 대해 매우 강인한 성능을 보인다.25

자율주행 환경의 LiDAR 데이터는 도로, 건물 벽 등 평면적인 구조를 많이 포함하고 있으며, EDGAR 차량처럼 서로 다른 종류의 LiDAR가 혼합된 경우 각 센서가 동일한 평면을 서로 다른 밀도와 패턴으로 스캔하게 된다. 이러한 '구조화되어 있지만 희소한(structured but sparse)' 데이터의 특성을 최대한 활용하기 위해, GICP는 점과 점을 맞추는 것을 넘어 '이 점들이 어떤 평면을 형성하는가'라는 고차원적인 정보를 이용한다. 따라서 GICP는 Multi-LiCa의 정밀 정합 단계에 가장 적합한 선택이라 할 수 있다.


GICP의 핵심 아이디어는 두 점군 $A = \{a_i\}$와 $B = \{b_i\}$를 정합할 때, 각 포인트 $a_i$, $b_i$를 단순한 3차원 좌표가 아닌, 주변 포인트 분포를 나타내는 가우시안 확률 분포로 모델링하는 것이다. 이 분포는 평균(포인트의 위치)과 공분산 행렬 $\mathbf{C}_i^A$, $\mathbf{C}_i^B$로 표현되며, 공분산 행렬은 해당 지점의 표면 구조(예: 평면의 방향성)를 인코딩한다.

알고리즘의 목표는 두 점군 사이의 강체 변환 $\mathbf{T}$를 찾는 것이며, 이는 다음 비용 함수(cost function)를 최소화함으로써 달성된다 34:
$$
\mathbf{T}^* = \underset{\mathbf{T}}{\arg\min} \sum_i \mathbf{d}_i(\mathbf{T})^T (\mathbf{C}_i^B + \mathbf{T}\mathbf{C}_i^A\mathbf{T}^T)^{-1} \mathbf{d}_i(\mathbf{T})
$$
여기서 $\mathbf{d}_i(\mathbf{T}) = \mathbf{b}_i - \mathbf{T}\mathbf{a}_i$는 변환된 소스 포인트 $\mathbf{T}\mathbf{a}_i$와 타겟 포인트 $\mathbf{b}_i$ 사이의 잔차(residual) 벡터이다. 이 비용 함수는 두 확률 분포 간의 마할라노비스 거리(Mahalanobis distance)의 제곱합과 유사한 형태를 띤다. 즉, 단순히 유클리드 거리를 최소화하는 것이 아니라, 각 포인트가 가지는 불확실성(표면의 형태)을 모두 고려하여 가중치를 부여함으로써 최적의 변환을 찾는 것이다. 이 비선형 최적화 문제는 일반적으로 BFGS와 같은 준-뉴턴 방법(Quasi-Newton method)을 사용하여 반복적으로 해결된다.34


GICP 알고리즘은 초기 추정치에 따라 지역 최적해에 빠질 수 있는 비볼록 최적화 문제이다. Multi-LiCa에서는 이 문제를 1단계 조대 정합을 통해 해결한다. TEASER++가 출력한 변환 행렬은 GICP 알고리즘의 초기 변환값(`InitialTransform`)으로 직접 사용된다.24 이 초기값은 이미 전역 최적해에 매우 근접해 있으므로, GICP는 지역 최적해의 위험 없이 빠르고 안정적으로 수렴하여 최종적으로 매우 정밀한 캘리브레이션 파라미터를 도출할 수 있다.



EDGAR 차량과 같이 복잡하고 분산된 센서 구성을 가진 시스템의 가장 큰 난제는 모든 센서가 동시에 바라보는 영역, 즉 완전한 시야 중첩(full FOV overlap)이 존재하지 않는다는 점이다.12 예를 들어, 차량 좌측의 LiDAR와 우측의 LiDAR는 서로의 시야가 전혀 겹치지 않을 수 있다. 이는 전통적인 다중 센서 캘리브레이션 방법론, 즉 하나의 기준 좌표계에 모든 센서를 직접 정합하는 방식의 기본 가정을 위배한다.


Multi-LiCa는 이 문제를 해결하기 위해 '연쇄적 정합 및 병합(Cascaded Registration and Merging)'이라는 독창적인 전략을 사용한다. 이 전략은 복잡한 N개의 센서 캘리브레이션 문제를 N-1개의 단순한 쌍별(pairwise) 캘리브레이션 문제로 효과적으로 분해한다. 이는 마치 정보의 다리를 놓는(information bridging) 과정과 같다. 인접한 센서 간의 캘리브레이션을 통해 정보를 순차적으로 전달하여, 결국 직접적인 연결이 없는 센서들까지 하나의 거대한 좌표계로 통합하는 것이다.

알고리즘의 구체적인 절차는 다음과 같다 12:

1. **초기화:** 캘리브레이션할 전체 LiDAR 목록에서 하나의 LiDAR를 기준이 되는 '타겟 LiDAR'로 지정한다. 나머지 LiDAR들은 '소스 LiDAR' 목록에 포함된다.

2. 반복 루프: 소스 LiDAR 목록이 비워질 때까지 다음 과정을 반복한다.

   a. 중첩 쌍 탐색: 현재의 타겟 점군(초기에는 타겟 LiDAR의 점군, 이후에는 병합된 점군)과 시야가 일부라도 겹치는 LiDAR를 소스 LiDAR 목록에서 모두 찾아낸다.

   b. 쌍별 정합 수행: 찾아낸 각 소스-타겟 쌍에 대해 Multi-LiCa의 2단계(조대+정밀) 정합 파이프라인을 실행하여 최적의 변환 행렬을 계산한다.

   c. 성공 여부 평가: 각 정합 결과에 대해 GICP의 '적합도 점수(fitness score)'를 평가한다. Multi-LiCa에서는 경험적으로 0.2 이상의 점수를 성공적인 정합으로 간주한다.12

   d. 점군 변환 및 병합: 적합도 점수가 임계값을 넘는, 즉 성공적으로 캘리브레이션된 각 소스 LiDAR에 대해 다음을 수행한다:

   i. 해당 소스 LiDAR의 점군을 계산된 변환 행렬을 이용해 타겟의 좌표계로 변환한다.

   ii. 변환된 점군을 기존 타겟 점군에 병합하여, 더 넓은 영역을 포괄하고 더 풍부한 기하학적 정보를 담은 새로운 '병합 타겟 점군'을 생성한다.

   e. 목록 업데이트: 성공적으로 캘리브레이션 및 병합이 완료된 LiDAR는 소스 LiDAR 목록에서 제거한다.

3. **종료:** 소스 LiDAR 목록의 모든 LiDAR가 성공적으로 캘리브레이션되면 프로세스를 종료한다.

이러한 '성공한 캘리브레이션의 연쇄적 의존성(cascading dependency)' 구조를 통해, 타겟 LiDAR와 직접적인 시야 중첩이 없는 센서라 할지라도, 중간에 위치한 다른 센서들을 '징검다리' 삼아 간접적으로 캘리브레이션이 가능해진다.24

이 전략은 계산적 복잡성을 줄일 뿐만 아니라, 시스템의 모듈성(modularity)과 확장성(scalability)을 크게 향상시킨다. 예를 들어, 시스템에 새로운 센서를 추가할 경우, 전체 시스템을 처음부터 다시 캘리브레이션할 필요 없이, 새로 추가된 센서와 기존의 병합된 점군(또는 가장 가까운 인접 센서) 간의 캘리브레이션만 수행하면 되기 때문이다. 이는 다중 센서 시스템의 유지보수와 업그레이드를 매우 용이하게 만드는 실용적인 공학적 해결책이다.



Multi-LiCa는 자율주행 기술 연구 및 개발 생태계의 표준 플랫폼인 ROS 2(Robot Operating System 2)에 완벽하게 통합되도록 설계되었다.20 사용자는 ROS 2 워크스페이스 내에서 `colcon build`를 통해 패키지를 빌드하고, `ros2 launch` 명령어를 사용하여 간단하게 캘리브레이션 노드를 실행할 수 있다. 캘리브레이션 과정에서 계산된 변환 행렬은 `/tf_static` 토픽을 통해 실시간으로 게시되어 Rviz와 같은 시각화 도구에서 즉시 확인할 수 있다.24

동시에, ROS 2 환경에 의존하지 않는 독립형(standalone) 애플리케이션으로도 사용할 수 있도록 구현되어, ROS를 사용하지 않는 다른 연구나 산업 환경에서도 Multi-LiCa의 핵심 기능을 활용할 수 있는 유연성을 제공한다.12


Multi-LiCa의 모든 동작은 하나의 YAML 설정 파일(`config/params.yaml`)을 통해 제어된다. 이를 통해 사용자는 자신의 센서 구성과 데이터 특성에 맞게 알고리즘을 세밀하게 조정할 수 있다.24 주요 설정 파라미터는 다음과 같다:

- **LiDAR 구성:** 캘리브레이션할 LiDAR들의 목록을 정의한다. ROS 2 환경에서는 메시지 토픽(topic) 이름을, 독립 실행 환경에서는 `.pcd` 파일의 경로를 지정한다.
- **알고리즘 파라미터:** 조대 정합과 정밀 정합 단계에서 사용되는 각 알고리즘의 하이퍼파라미터를 설정한다. 예를 들어, FPFH 특징 계산 시 사용할 이웃 검색 반경, TEASER++의 노이즈 바운드, GICP의 최대 반복 횟수 및 수렴 허용 오차 등을 조정할 수 있다.
- **LiDAR-to-Ground 캘리브레이션:** 지면 대비 센서의 피치(pitch) 각도와 높이를 보정하는 부가 기능의 활성화 여부와 RANSAC 알고리즘 관련 파라미터를 설정한다.
- **출력 설정:** 캘리브레이션 결과를 저장할 형식을 지정할 수 있다. 기본적으로 변환 행렬이 텍스트 파일로 저장되며, ROS의 로봇 상태 표현에 직접 사용할 수 있는 URDF(Unified Robot Description Format) 파일로 저장하는 옵션도 제공된다.24


Multi-LiCa는 LiDAR 간의 상대적 자세를 맞추는 것 외에도, 기준이 되는 타겟 LiDAR와 차량의 베이스(또는 지면) 간의 관계를 캘리브레이션하는 부가 기능을 제공한다.24 이 기능은 주로 센서가 지면을 기준으로 얼마나 기울어져 있는지(pitch)와 얼마나 떨어져 있는지(z-offset)를 보정하는 데 사용된다.

이 기능은 지면이 평평하다는 강한 가정 하에 동작한다. RANSAC(RANdom SAmple Consensus) 알고리즘을 사용하여 점군 데이터에서 가장 큰 평면, 즉 지면을 강인하게 추출한다. 추출된 지면 평면의 법선 벡터와 센서 좌표계의 z축 사이의 각도를 통해 피치 오차를 계산하고, 지면과 센서 원점 사이의 거리를 통해 높이 오차를 보정한다.24 단, 이 기능을 정확하게 사용하기 위해서는 차량의 베이스 좌표계 대비 센서의 x, y 위치와 롤(roll), 요(yaw) 각도가 사전에 정밀하게 알려져 있어야 한다는 전제 조건이 따른다.24


Multi-LiCa는 강력하고 유연한 도구이지만, 그 성능을 보장하기 위해 몇 가지 중요한 가정과 제약사항을 전제로 한다 24:

1. **정지 상태 (Motionless Calibration):** 캘리브레이션에 사용되는 점군 데이터는 차량과 주변 환경이 모두 완벽하게 정지된 상태에서 수집되어야 한다. 데이터 수집 중 차량이 움직이거나 주변에 동적 객체가 존재할 경우, 이는 점군 데이터에 왜곡을 유발하여 캘리브레이션 정확도를 심각하게 저하시킨다.
2. **부분적 FOV 중첩 (Partial FOV Overlap Requirement):** 캘리브레이션하고자 하는 모든 LiDAR는 서로 연결된 시야 체인(chain of overlapping FOVs)을 형성해야 한다. 즉, 어떤 LiDAR든 직접적이든 혹은 다른 LiDAR를 거쳐서든 기준이 되는 타겟 LiDAR와 시야가 연결되어야 한다. 완전히 고립되어 다른 어떤 센서와도 시야가 겹치지 않는 센서는 이 프레임워크로 캘리브레이션할 수 없다.
3. **정적 환경 및 평평한 지면 (Static Environment & Flat Ground Assumption):** 타겟리스 방식의 특성상, 캘리브레이션 데이터가 수집되는 동안 환경의 기하학적 구조가 변하지 않아야 한다. 또한, LiDAR-to-Ground 캘리브레이션 기능을 사용할 경우에는 지면이 충분히 넓고 평평하다고 가정한다.

이러한 제약사항들은 Multi-LiCa의 명확한 운영 설계 영역(Operational Design Domain, ODD)을 정의한다. 즉, 이 도구는 주행 중에 실시간으로 발생하는 미세한 오차를 보정하기 위한 '온라인/연속적 캘리브레이션' 도구가 아니라, 차량 운행 전 차고나 정비소와 같은 통제된 환경에서 시스템의 기준(baseline) 캘리브레이션을 정확하고 자동으로 설정하는 데 특화된 '오프라인 설정 및 정비 도구'로 포지셔닝된다. 이는 단점이 아니라, 도구의 목적과 사용 시나리오를 명확히 하는 중요한 특징이다.



Multi-LiCa의 특징과 장점을 명확히 이해하기 위해, 다른 주요 LiDAR 캘리브레이션 방법론과 비교 분석하는 것은 매우 유용하다. 아래 표는 Multi-LiCa, 전통적인 타겟 기반 방식, 그리고 또 다른 타겟리스 방식인 NDT(Normal Distributions Transform) 기반 방법의 핵심 특성을 요약한 것이다.

| 특성 (Attribute)                | Multi-LiCa                                              | 타겟 기반 (Target-Based)                                     | NDT 기반 (e.g., Ridecell)                          |
| ------------------------------- | ------------------------------------------------------- | ------------------------------------------------------------ | -------------------------------------------------- |
| **방법론 (Methodology)**        | FPFH+TEASER++ (조대) + GICP (정밀)                      | 알려진 타겟의 기하학적 특징 추출 (예: 체커보드 코너, ChArUco ID) | 정규 분포 변환 (Normal Distributions Transform)    |
| **타겟 요구사항 (Target Req.)** | 없음 (Targetless)                                       | 필수 (체커보드, ChArUco, 구체 등) 8                          | 없음 (Targetless)                                  |
| **자동화 수준 (Automation)**    | 완전 자동 (Fully Automatic) 20                          | 반자동 (타겟 설치 및 데이터 수집에 수동 개입 필요) 15        | 완전 자동 20                                       |
| **초기 추정치 (Initial Guess)** | 불필요 (Not Required) 24                                | 불필요 (Not Required)                                        | 필요 (Required) 20                                 |
| **이상치 강인성 (Robustness)**  | 매우 높음 (TEASER++의 전역 최적화 덕분) 28              | 중간 (타겟 탐지 알고리즘의 정확도에 의존) 8                  | 중간 (초기 추정치에 민감) 20                       |
| **핵심 장점 (Key Advantage)**   | 비정형/비중첩 FOV 구성에 강인하며 자동화 수준이 높음 12 | 명확한 제약조건으로 인해 잠재적으로 가장 높은 정확도 달성 가능 11 | 구조화된 환경에서 빠른 정합 속도                   |
| **주요 한계 (Key Limitation)**  | 정적 환경 가정, 환경 특징 부족 시 성능 저하 가능성 24   | 수동 개입 필수, 동적 환경 적용 불가, 타겟 폐색/조명에 취약 8 | 초기 추정치에 민감하며 비구조적 환경에서 성능 저하 |

이 비교를 통해 Multi-LiCa가 추구하는 가치를 명확히 알 수 있다. 타겟 기반 방법이 이론적인 '최고 정확도'를 목표로 한다면, Multi-LiCa는 '충분히 높은 정확도'를 '완전 자동화'와 '비정형 환경에 대한 유연성'이라는 실용적 가치와 균형을 맞추려 한다. EDGAR와 같이 복잡한 센서 시스템에서는 모든 센서에 동시에 보이는 단일 타겟을 배치하는 것 자체가 비현실적일 수 있다. 이러한 맥락에서 Multi-LiCa는 현실적인 제약 조건 하에서 가장 강인하고 효율적인 해결책을 제공함으로써, 이론적 최적성보다 공학적 실용성을 우선시하는 설계 철학을 보여준다.


Multi-LiCa의 성능은 기존의 타겟리스 캘리브레이션 방법인 CROON과의 비교를 통해 검증되었다. 실험 결과, Multi-LiCa는 CROON과 동등하거나 더 나은 캘리브레이션 정확도를 보였으며, 특히 CROON이 실패하는 비중첩 FOV 시나리오에서도 안정적으로 작동하여 그 강인성을 입증했다.25

그러나 Multi-LiCa와 같은 타겟리스 방법론은 특정 환경에서 실패할 가능성을 내포하고 있다. 일반적인 실패 시나리오는 다음과 같다:

- **기하학적 특징 부족:** 넓은 공터나 특징 없는 벽면과 같이 기하학적 구조가 단조로운 환경에서는 FPFH와 같은 특징 기술자가 변별력을 잃게 된다. 이로 인해 조대 정합 단계에서 올바른 대응점을 찾지 못해 캘리브레이션에 실패할 수 있다.
- **대칭적 환경:** 긴 터널이나 복도와 같이 대칭적인 구조를 가진 환경은 알고리즘에 혼란을 줄 수 있다. 대칭성으로 인해 수학적으로는 유효하지만 실제와는 다른 다수의 정합 해(multiple valid solutions)가 존재할 수 있기 때문이다.16
- **악천후 조건:** 비, 안개, 눈과 같은 악천후는 LiDAR 센서 데이터의 품질을 심각하게 저하시킨다. 대기 중의 입자(빗방울, 눈송이 등)에 의해 레이저 빔이 산란되거나 흡수되어 점군에 심각한 노이즈가 발생하고, 유효 포인트 수가 감소하며, 반사 강도 값이 왜곡된다.38 이러한 데이터 품질 저하는 FPFH 특징 계산의 신뢰도를 떨어뜨리고 GICP의 평면 추정을 방해하여 캘리브레이션 성능에 치명적인 악영향을 미칠 수 있다.40



Multi-LiCa는 다중 LiDAR 시스템의 외인성 캘리브레이션 분야에서 중요한 기술적 진보를 이루었다. 그 핵심 기여는 다음과 같이 요약할 수 있다.

1. **강인성과 유연성 확보:** 초기 변환 추정치나 모든 센서 간의 직접적인 시야 중첩이라는 까다로운 제약 조건 없이도, 다양한 종류와 배치를 가진 다중 LiDAR 시스템을 완전 자동으로 캘리브레이션할 수 있는 강인하고 유연한 프레임워크를 제공했다.12
2. **최신 알고리즘의 효과적 통합:** FPFH, TEASER++, GICP와 같은 최신 기하학 기반 컴퓨터 비전 알고리즘들을 '조대-정밀' 구조로 효과적으로 통합하였다. 이를 통해 타겟리스 방식이 가지는 초기치 의존성 및 지역 최적해 문제를 극복하고 높은 수준의 정확도와 신뢰성을 동시에 달성했다.24
3. **새로운 센서 아키텍처 지원:** TUM의 EDGAR 연구 차량과 같이, 기존의 패러다임에서 벗어난 비정형적이고 복잡한 센서 아키텍처의 실용화를 가능하게 하는 핵심 기반 기술로서의 가치를 입증했다.20
4. **오픈소스 생태계 기여:** 전체 소스 코드를 공개함으로써 12, 학계와 산업계의 관련 연구자들이 이를 활용하고 개선하며, 자율주행 기술 발전에 기여할 수 있는 토대를 마련했다.


Multi-LiCa는 오프라인 캘리브레이션 분야에서 괄목할 만한 성과를 이루었지만, 완전한 자율주행 시스템을 위해서는 여전히 해결해야 할 과제들이 남아있다.

- **동적 환경으로의 확장:** 현재 Multi-LiCa는 차량과 환경이 정지된 상태를 가정한다.24 그러나 실제 주행 중에는 엔진의 열이나 도로의 진동으로 인해 센서의 정렬 상태가 미세하게 변할 수 있다. 이러한 동적 변화를 실시간으로 감지하고 보정할 수 있는 '온라인/연속적(Online/Continuous)' 캘리브레이션 기술로의 확장이 필요하다.42
- **딥러닝 기반 방법론과의 융합:** 최근에는 딥러닝 기술을 이용하여 센서 데이터를 직접 입력받아 외인성 파라미터를 회귀(regression)하는 연구가 활발히 진행되고 있다.6 이러한 데이터 기반 접근법은 특정 환경에 대한 가정이 적다는 장점이 있지만, 학습 데이터에 과적합(overfitting)될 위험이 있고 결과의 신뢰성을 보장하기 어렵다는 단점이 있다.19
- **악천후 조건에서의 강인성 확보:** 악천후는 LiDAR 데이터 품질을 심각하게 저하시키는 가장 큰 요인 중 하나이다.38 현재의 기하학 기반 특징들은 이러한 극심한 노이즈 환경에서 쉽게 무너질 수 있다. 악천후 속에서도 안정적으로 작동하는 새로운 특징 기술자나 노이즈 제거 기술과 결합된 캘리브레이션 알고리즘 개발이 시급한 과제이다.

Multi-LiCa와 같은 고전적 기하학 기반 방법론의 미래는 딥러닝 기반 방법론과의 '하이브리드(Hybrid)' 접근 방식에서 찾을 수 있다. Multi-LiCa는 수학적으로 증명 가능하고 신뢰도 높은 초기 캘리브레이션 값을 생성하는 데 탁월한 성능을 보인다. 이렇게 얻어진 정확한 초기값을 일종의 '의사-레이블(pseudo-label)' 또는 '기준점(anchor)'으로 활용하여, 주행 중 발생하는 작은 변위를 실시간으로 예측하고 보정하는 경량화된 딥러닝 모델을 지도 학습(supervised learning)시키는 시나리오를 구상해 볼 수 있다. 이러한 하이브리드 접근법은 기하학 기반 방법이 제공하는 '정확성과 신뢰성'과 딥러닝 기반 방법이 제공하는 '실시간성과 적응성'이라는 두 마리 토끼를 모두 잡는, 차세대 캘리브레이션 패러다임으로 발전할 잠재력을 지니고 있다. 즉, Multi-LiCa는 그 자체로 완성도 높은 도구일 뿐만 아니라, 미래의 지능형 온라인 캘리브레이션 시스템을 위한 '고품질 학습 데이터 생성기'로서 새로운 역할을 수행할 수 있을 것이다.


1. Automatic Extrinsic Calibration of 3D LIDAR and Multi-Cameras Based on Graph Optimization - PMC, accessed August 7, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC8954836/
2. Autonomous Vehicle Perception: Sensor Fusion based on Structured Learning Methods - Chair of Automotive Technology - TUM Mobility Systems Engineering, accessed August 7, 2025, https://www.mos.ed.tum.de/en/ftm/main-research/intelligent-vehicle-systems/autonomous-vehicle-perception-sensor-fusion-based-on-structured-learning-methods/
3. Self-Driving Car Technology for a Reliable Ride - Waymo Driver, accessed August 7, 2025, https://waymo.com/waymo-driver/
4. Automatic Target-Less Camera-LiDAR Calibration From Motion and Deep Point Correspondences - arXiv, accessed August 7, 2025, https://arxiv.org/html/2404.17298v1
5. RobustCalib: Robust Lidar-Camera Extrinsic Calibration with Consistency Learning - arXiv, accessed August 7, 2025, https://arxiv.org/html/2312.01085v1
6. CalibFormer: A Transformer-based Automatic LiDAR-Camera Calibration Network - arXiv, accessed August 7, 2025, https://arxiv.org/html/2311.15241v2
7. How does multi-sensor calibration work in autonomous vehicles? - EE World Online, accessed August 7, 2025, https://www.eeworldonline.com/how-does-multi-sensor-calibration-work-in-autonomous-vehicles/
8. A Target-based Multi-LiDAR Multi-Camera Extrinsic Calibration System - arXiv, accessed August 7, 2025, https://arxiv.org/html/2507.16621v1
9. GMMCalib: Extrinsic Calibration of LiDAR Sensors using GMM-based Joint Registration, accessed August 7, 2025, https://arxiv.org/html/2404.03427v1
10. BEVCalib: LiDAR-Camera Calibration via Geometry-Guided Bird's-Eye View Representations - arXiv, accessed August 7, 2025, https://arxiv.org/html/2506.02587v1
11. (PDF) Improvements to Target-Based 3D LiDAR to Camera Calibration - ResearchGate, accessed August 7, 2025, https://www.researchgate.net/publication/343113840_Improvements_to_Target-Based_3D_LiDAR_to_Camera_Calibration
12. (PDF) Multi-LiCa: A Motion and Targetless Multi LiDAR-to-LiDAR Calibration Framework, accessed August 7, 2025, https://www.researchgate.net/publication/388231076_Multi-LiCa_A_Motion_and_Targetless_Multi_LiDAR-to-LiDAR_Calibration_Framework
13. Extrinsic calibration between a multi-layer lidar and a camera - ResearchGate, accessed August 7, 2025, https://www.researchgate.net/publication/224338006_Extrinsic_calibration_between_a_multi-layer_lidar_and_a_camera
14. DF-Calib: Targetless LiDAR-Camera Calibration via Depth Flow - ResearchGate, accessed August 7, 2025, https://www.researchgate.net/publication/390439904_DF-Calib_Targetless_LiDAR-Camera_Calibration_via_Depth_Flow
15. Automatic Targetless LiDAR-Camera Calibration: A Survey - ResearchGate, accessed August 7, 2025, https://www.researchgate.net/publication/363244175_Automatic_Targetless_LiDAR-Camera_Calibration_A_Survey
16. Online, Target-Free LiDAR-Camera Extrinsic Calibration via Cross-Modal Mask Matching, accessed August 7, 2025, https://arxiv.org/html/2404.18083v2
17. [Literature Review] MFCalib: Single-shot and Automatic Extrinsic Calibration for LiDAR and Camera in Targetless Environments Based on Multi-Feature Edge - Moonlight, accessed August 7, 2025, https://www.themoonlight.io/en/review/mfcalib-single-shot-and-automatic-extrinsic-calibration-for-lidar-and-camera-in-targetless-environments-based-on-multi-feature-edge
18. koide3/direct_visual_lidar_calibration: A toolbox for target-less LiDAR-camera calibration [ROS1/ROS2] - GitHub, accessed August 7, 2025, https://github.com/koide3/direct_visual_lidar_calibration
19. What Really Matters for Learning-based LiDAR-Camera Calibration - arXiv, accessed August 7, 2025, https://arxiv.org/html/2501.16969v1
20. Multi-LiCa: A Motion- and Targetless Multi - LiDAR-to-LiDAR Calibration Framework - arXiv, accessed August 7, 2025, https://arxiv.org/html/2501.11088v1
21. (PDF) Multi-LiDAR Localization and Mapping Pipeline for Urban Autonomous Driving, accessed August 7, 2025, https://www.researchgate.net/publication/375408399_Multi-LiDAR_Localization_and_Mapping_Pipeline_for_Urban_Autonomous_Driving
22. Multi-LiCa: A Motion- and Targetless Multi - LiDAR-to-LiDAR Calibration Framework, accessed August 7, 2025, https://www.researchgate.net/publication/384797483_Multi-LiCa_A_Motion-_and_Targetless_Multi_-_LiDAR-to-LiDAR_Calibration_Framework
23. Multi-LiCa: A Motion and Targetless Multi LiDAR-to-LiDAR Calibration Framework, accessed August 7, 2025, https://openreview.net/forum?id=tyT8eQ3Jum
24. TUMFTM/Multi_LiCa: Multi - LiDAR-to-LiDAR calibration framework for ROS2 and non-ROS applications - GitHub, accessed August 7, 2025, https://github.com/TUMFTM/Multi_LiCa
25. [Literature Review] Multi-LiCa: A Motion and Targetless Multi LiDAR-to-LiDAR Calibration Framework - Moonlight, accessed August 7, 2025, https://www.themoonlight.io/en/review/multi-lica-a-motion-and-targetless-multi-lidar-to-lidar-calibration-framework
26. Understanding Iterative Closest Point (ICP) Algorithm with Code - LearnOpenCV, accessed August 7, 2025, https://learnopencv.com/iterative-closest-point-icp-explained/
27. MIT-SPARK/TEASER-plusplus: A fast and robust point cloud registration library - GitHub, accessed August 7, 2025, https://github.com/MIT-SPARK/TEASER-plusplus
28. MIT Open Access Articles TEASER: Fast and Certifiable Point Cloud Registration, accessed August 7, 2025, https://dspace.mit.edu/bitstream/handle/1721.1/134163/2001.07715.pdf?sequence=2&isAllowed=y
29. Point Cloud Registration Based on Fast Point Feature Histogram Descriptors for 3D Reconstruction of Trees - MDPI, accessed August 7, 2025, https://www.mdpi.com/2072-4292/15/15/3775
30. Fast Point Feature Histograms (FPFH) descriptors - Point Cloud Library 0.0 documentation, accessed August 7, 2025, https://pcl.readthedocs.io/projects/tutorials/en/master/fpfh_estimation.html
31. The New Fast Point Feature Histograms Algorithm Based on Adaptive Selection - Journal of Applied Science and Engineering, accessed August 7, 2025, http://jase.tku.edu.tw/articles/jase-202006-23-2-0006.pdf
32. Fast Point Feature Histograms (FPFH) for 3D Registration - 3D Vision Laboratory, accessed August 7, 2025, https://www.cvl.iis.u-tokyo.ac.jp/class2016/2016w/papers/6.3DdataProcessing/Rusu_FPFH_ICRA2009.pdf
33. [2001.07715] TEASER: Fast and Certifiable Point Cloud Registration - arXiv, accessed August 7, 2025, https://arxiv.org/abs/2001.07715
34. Generalized ICP - Wiki, accessed August 7, 2025, https://wiki.hanzheteng.com/algorithm/slam/generalized-icp
35. Visually Bootstrapped Generalized ICP, accessed August 7, 2025, https://svl.stanford.edu/assets/publications/pdfs/gpandey-2011b.pdf
36. Generalized-ICP - Robotics, accessed August 7, 2025, https://www.roboticsproceedings.org/rss05/p21.pdf
37. US9933515B2 - Sensor calibration for autonomous vehicles - Google Patents, accessed August 7, 2025, https://patents.google.com/patent/US9933515B2/en
38. Survey on LiDAR Perception in Adverse Weather Conditions - ResearchGate, accessed August 7, 2025, https://www.researchgate.net/publication/370684027_Survey_on_LiDAR_Perception_in_Adverse_Weather_Conditions
39. Survey on LiDAR Perception in Adverse Weather Conditions - arXiv, accessed August 7, 2025, https://arxiv.org/pdf/2304.06312
40. Empirical Analysis of Autonomous Vehicle's LiDAR Detection Performance Degradation for Actual Road Driving in Rain and Fog, accessed August 7, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10051412/
41. Effect of Weather on the Performance of Autonomous Vehicle LiDAR Sensors | Request PDF, accessed August 7, 2025, https://www.researchgate.net/publication/358111485_Effect_of_Weather_on_the_Performance_of_Autonomous_Vehicle_LiDAR_Sensors
42. Uncertainty-Aware Online Extrinsic Calibration: A Conformal Prediction Approach - arXiv, accessed August 7, 2025, https://arxiv.org/html/2501.06878v1
43. Continuous extrinsic online calibration for stereo cameras | Request PDF - ResearchGate, accessed August 7, 2025, https://www.researchgate.net/publication/305998117_Continuous_extrinsic_online_calibration_for_stereo_cameras
44. Automatic Online Calibration Between Lidar and Camera - DiVA portal, accessed August 7, 2025, http://www.diva-portal.org/smash/get/diva2:1415878/FULLTEXT01.pdf