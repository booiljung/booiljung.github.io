# GRAND-SLAM에 대한 심층 고찰

## 1. 자율 이동 로봇을 위한 공간 인식의 패러다임

자율 이동 로봇 기술의 발전은 로봇이 주변 환경을 어떻게 인식하고 상호작용하는지에 대한 근본적인 질문과 맞닿아 있다. 이 질문에 대한 핵심적인 해답 중 하나가 바로 SLAM(Simultaneous Localization and Mapping) 기술이다.1 SLAM은 로봇이 사전 정보가 없는 미지의 환경에 놓였을 때, 자신의 위치를 실시간으로 추정함과 동시에 주변 환경의 지도를 점진적으로 작성해 나가는 과정을 의미한다. 이 기술은 GPS 신호가 도달하지 않는 실내, 지하, 심해와 같은 환경에서 로봇의 자율적 항법을 가능하게 하는 필수불가결한 요소로 자리 잡았다.4 나아가 증강 현실(AR), 가상 현실(VR), 자율 주행 자동차 등 다양한 미래 산업의 기반 기술로서 그 중요성이 날로 증대되고 있다.6

SLAM 기술의 여러 갈래 중에서도, 카메라를 주된 센서로 사용하는 Visual SLAM(vSLAM)은 저렴한 비용으로 풍부하고 밀도 높은 환경 정보를 획득할 수 있다는 장점 덕분에 학계와 산업계의 지대한 관심을 받고 있다.3 vSLAM의 발전사는 곧 3차원 공간을 어떻게 표현하고 이해할 것인가에 대한 기술적 진화의 역사와 궤를 같이한다. 초기 vSLAM 시스템들은 주로 로봇의 항법에 필요한 최소한의 기하학적 정보를 담기 위해, 환경을 소수의 뚜렷한 특징점들로 표현하는 희소(sparse) 포인트 클라우드 방식을 채택했다.6 그러나 기술이 발전하고 응용 분야가 확장됨에 따라, 단순한 항법 지도를 넘어 인간이 직관적으로 이해하고 상호작용할 수 있는, 보다 사실적이고 밀도 높은(dense) 맵에 대한 요구가 커지기 시작했다.

이러한 요구에 부응하여 최근 컴퓨터 비전 및 그래픽스 분야에서는 NeRF(Neural Radiance Fields)와 3D Gaussian Splatting(3DGS)과 같은 혁신적인 장면 표현 기술이 등장했다.6 이 기술들은 환경을 단순한 점의 집합이 아닌, 임의의 시점에서 사실적인 이미지를 렌더링할 수 있는 연속적이거나 미분 가능한 '장면(scene)'으로 표현한다. 이는 SLAM 기술의 패러다임을 '어떻게 더 정확하게 위치를 추정하고 기하학적 지도를 만들 것인가'라는 전통적인 목표에서 '어떻게 더 풍부하고 사실적인 디지털 세계를 재구성할 것인가'라는 새로운 차원으로 확장시켰다. SLAM의 목표가 단순히 로봇의 '항법'을 넘어, AR/VR과 같은 응용을 위한 '사실적 재구성'으로 진화하고 있는 것이다.

본 고찰의 목적은 이러한 기술적 흐름의 최전선에 있는 GRAND-SLAM (Gaussian Reconstruction via Multi-Agent Dense SLAM) 알고리즘을 심층적으로 분석하는 데 있다.12 GRAND-SLAM은 3DGS가 제공하는 뛰어난 장면 표현 능력을 기반으로, 이를 다수의 로봇이 협력하는 다중 에이전트(multi-agent) 시나리오와 광활한 대규모(large-scale) 환경으로 확장하려는 시도이다.6 이는 SLAM 기술이 직면한 전통적인 난제인 '확장성'과 '전역적 일관성'을 해결하면서 동시에 '사실적인 맵 재구성'이라는 새로운 목표를 달성하려는 야심 찬 도전이다. 따라서 GRAND-SLAM에 대한 분석은 현대 SLAM 기술의 현주소를 진단하고, 자율 로봇과 인간이 디지털화된 공간 정보를 공유하며 상호작용하는 미래를 조망하는 중요한 시금석이 될 것이다.

## 2.  Visual SLAM의 양대 산맥 - 간접 방식과 직접 방식

Visual SLAM은 카메라로부터 얻은 이미지 정보를 처리하여 위치 추정과 지도 작성을 수행하는 방식에 따라 크게 두 가지 접근법, 즉 간접 방식과 직접 방식으로 나뉜다. 이 두 방식은 상호 보완적인 장단점을 가지며, 현대 SLAM 시스템의 근간을 이루는 핵심 철학을 제공한다.

### 2.1  특징점 기반의 간접 방식 (Indirect Feature-Based Methods)

간접 방식은 이미지에서 시각적으로 두드러지는 소수의 특징점(keypoint)을 추출하고 이를 추적하여 카메라의 움직임을 추정하는 접근법이다.7 이 방식의 핵심 원리는 다음과 같다. 먼저, 이미지 내에서 코너나 블롭(blob)과 같이 반복적으로 검출 가능하고 구별되는 지점들을 ORB(Oriented FAST and Rotated BRIEF), SIFT(Scale-Invariant Feature Transform)와 같은 알고리즘을 사용해 키포인트로 추출한다. 각 키포인트는 주변 픽셀 정보를 요약한 고유한 기술자(descriptor)를 가지게 되며, 시스템은 연속된 이미지 프레임 간에 이 기술자의 유사도를 비교하여 동일한 3차원 점에 해당하는 키포인트들을 정합(matching)한다.14

일단 키포인트들의 대응 관계가 수립되면, SLAM 시스템은 3차원 공간상의 맵 포인트(map point)들을 현재 추정된 카메라 포즈로 2차원 이미지 평면에 재투영(reprojection)시킨다. 이때 실제 관측된 키포인트의 위치와 재투영된 위치 사이의 오차, 즉 기하학적 재투영 오차(geometric reprojection error)가 발생한다. 간접 방식의 목표는 이 재투영 오차의 총합을 최소화하는 카메라 포즈와 맵 포인트 위치를 찾는 것이다.15

이 분야의 가장 대표적인 알고리즘은 ORB-SLAM2이다. ORB-SLAM2는 매우 빠르고 회전 불변성을 갖는 ORB 특징점을 시스템의 모든 단계-추적, 지도 작성, 루프 클로저-에 일관되게 사용하여 높은 실시간 성능과 강인성을 달성했다.9 특히, 시스템을 추적(Tracking), 지역 매핑(Local Mapping), 루프 클로저(Loop Closing)의 세 가지 주요 스레드로 분리하여 병렬적으로 처리함으로써 효율성을 극대화했다.18

간접 방식의 가장 큰 장점은 특징점 기술자를 사용하기 때문에 조명 변화나 카메라의 자동 노출 보정 등 광도(photometric) 변화에 비교적 강인하다는 것이다.14 그러나 명확한 코너나 텍스처가 부족한(textureless) 환경, 예를 들어 흰 벽이나 복도와 같은 곳에서는 안정적으로 키포인트를 추출하고 추적하기 어려워 성능이 급격히 저하되는 명백한 한계를 가진다.14

### 2.2  픽셀 강도 기반의 직접 방식 (Direct Intensity-Based Methods)

직접 방식은 간접 방식과 달리 명시적인 특징점 추출 및 정합 과정을 생략하고, 이미지의 원시 픽셀 강도(intensity) 정보를 직접 활용하여 카메라 모션을 추정한다.7 이 방식의 핵심 아이디어는 '광도 불변성 가정(photometric consistency assumption)'에 기반한다. 즉, 동일한 3차원 지점은 다른 시점에서 보더라도 그 밝기 값이 거의 일정하게 유지된다는 가정이다.

직접 방식은 이 가정을 바탕으로, 한 프레임의 픽셀들을 현재 추정된 카메라 모션과 깊이 정보를 이용해 다른 프레임의 이미지 평면에 투영한다. 이후, 원본 픽셀의 강도 값과 투영된 위치의 픽셀 강도 값 사이의 차이, 즉 광도 오차(photometric error)를 계산한다. 직접 방식 SLAM의 목표는 이 광도 오차의 총합이 최소가 되는 카메라 포즈를 찾는 것이다.15

이 접근법의 대표적인 예는 DSO(Direct Sparse Odometry)이다. DSO는 이미지 전체에서 픽셀 강도의 경사도(gradient)가 충분히 큰 픽셀들을 희소하지만 균일하게 샘플링한다.20 이를 통해 키포인트 검출기 없이도 텍스처가 거의 없는 벽면의 미세한 음영 변화까지 활용하여 안정적인 추적을 수행할 수 있다. DSO의 핵심적인 기여는 카메라의 기하학적 파라미터(포즈)와 맵의 구조(역깊이, inverse depth), 그리고 카메라의 광학적 파라미터(노출 시간, 비네팅 등)까지 포함하는 통합된 확률 모델을 구축하고, 이 모든 변수들을 일관되게 공동으로 최적화(joint optimization)한다는 점이다.20

직접 방식의 주된 장점은 텍스처가 부족한 환경에서도 작동할 수 있다는 점으로, 간접 방식의 근본적인 한계를 보완한다.14 하지만 픽셀의 밝기 값에 직접적으로 의존하기 때문에, 급격한 조명 변화나 카메라의 자동 노출 변경과 같이 광도 불변성 가정이 깨지는 환경에서는 성능이 저하되는 취약점을 보인다.7

### 2.3  하이브리드 방식과 최신 동향

간접 방식과 직접 방식은 이처럼 명확히 상호 보완적인 장단점을 지닌다.14 따라서 최근 연구 동향은 두 방식의 장점을 결합하려는 하이브리드(hybrid) 방식으로 나아가고 있다. 예를 들어, 프레임 간의 미세한 움직임은 계산 효율이 높은 직접 방식으로 추정하고, 장기적인 일관성 확보를 위한 루프 클로저나 전역 최적화(global optimization)는 조명 변화에 강인한 간접 방식의 특징점을 활용하는 식이다.15

이러한 기술적 맥락 속에서 GRAND-SLAM의 추적 모듈을 살펴보면, 그 철학이 본질적으로 직접 방식에 깊이 뿌리내리고 있음을 알 수 있다. GRAND-SLAM은 3D 가우시안으로 표현된 맵을 현재 카메라 포즈로 렌더링하여 가상의 이미지를 생성하고, 이 가상 이미지와 실제 카메라로 입력된 이미지 간의 차이(렌더링 손실)를 최소화함으로써 카메라 포즈를 최적화한다.13 이 렌더링 손실은 색상(color)과 깊이(depth) 정보의 차이를 모두 포함하는데, 이는 픽셀 레벨의 정보를 직접 비교하여 오차를 최소화한다는 점에서 직접 방식의 핵심 아이디어와 정확히 일치한다. 즉, GRAND-SLAM의 프론트엔드는 3DGS라는 현대적인 장면 표현 방식을 활용하여 직접 방식의 원리를 정교하게 구현한 최신 사례로 해석될 수 있다.

| 평가 항목            | 간접 방식 (예: ORB-SLAM2)                                    | 직접 방식 (예: DSO)                                          |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **핵심 원리**        | 키포인트 추출 및 정합, 기하학적 재투영 오차 최소화 15        | 픽셀 강도 직접 사용, 광도 오차 최소화 15                     |
| **대표 알고리즘**    | ORB-SLAM2, PTAM 14                                           | DSO, LSD-SLAM 15                                             |
| **주요 최적화 대상** | 기하학적 재투영 오차 (Geometric Reprojection Error)          | 광도 오차 (Photometric Error)                                |
| **장점**             | 조명 및 시점 변화에 강인함, 개념적으로 명확함 14             | 텍스처가 부족한 환경에서도 작동, 더 밀도 높은 맵 생성 가능 14 |
| **단점**             | 텍스처 부족 환경에 취약, 특징점 추출이 계산 비용의 일부를 차지함 14 | 조명 변화, 카메라 노출 변화 등 광도 변화에 취약함 7          |
| **강인한 환경**      | 뚜렷한 텍스처와 특징점이 풍부한 환경                         | 텍스처가 부족하지만 음영 등 미세한 경사도가 존재하는 환경    |
| **취약한 환경**      | 텍스처가 없는 흰 벽, 반복적인 패턴이 많은 환경               | 급격한 조명 변화, 모션 블러가 심한 환경                      |

## 3.  전역 일관성 확보의 핵심 - 그래프 기반 SLAM과 최적화

vSLAM 시스템이 장시간, 넓은 영역을 탐사할 때 필연적으로 발생하는 문제는 오차의 누적, 즉 드리프트(drift)이다. 로봇이 움직임을 추정할 때마다 미세한 오차가 발생하고, 이 오차가 시간이 지남에 따라 쌓여 실제 위치와 추정 위치 간의 간극이 점점 커지게 된다. 그래프 기반 SLAM은 이러한 누적 오차를 효과적으로 보정하고 전역적으로 일관된 지도와 경로를 생성하기 위한 강력한 수학적 프레임워크를 제공한다.

### 3.1  포즈 그래프의 수학적 표현 (Mathematical Formulation of Pose Graphs)

그래프 기반 SLAM의 핵심 아이디어는 전체 SLAM 문제를 하나의 거대한 그래프 최적화 문제로 모델링하는 것이다.4 이 그래프는 다음과 같은 요소들로 구성된다.

- **노드 (Nodes):** 그래프의 각 노드($x_i$)는 특정 시점에서의 로봇의 상태, 즉 포즈(pose)를 나타낸다. 2D 환경에서는 위치 $(x, y)$와 방향 $\theta$로, 3D 환경에서는 위치 $(x, y, z)$와 회전(쿼터니언 또는 회전 행렬)으로 표현된다.5 경우에 따라서는 환경 내의 고정된 지점인 랜드마크(landmark) 또한 노드로 표현될 수 있다.

- **엣지 (Edges):** 엣지는 두 노드 간의 공간적 제약(spatial constraint)을 나타낸다. 이 제약은 센서 측정으로부터 얻어진다. 대표적인 엣지에는 두 가지 종류가 있다. 하나는 연속된 두 포즈($x_i$, $x_{i+1}$) 사이의 움직임을 나타내는 오도메트리(odometry) 제약($u_i$)이다. 다른 하나는 로봇이 과거에 방문했던 장소를 다시 인식했을 때 생성되는 루프 클로저(loop closure) 제약($z_{ij}$)으로, 시간적으로 멀리 떨어진 두 포즈($x_i$, $x_j$)를 직접 연결한다.4

- **오차 함수 (Error Function):** 모든 엣지에는 오차 함수 $e_{ij}(x_i, x_j)$가 연관되어 있다. 이 함수는 현재 추정된 노드들의 값(포즈 $x_i, x_j$)을 바탕으로 계산한 예상 측정값($\hat{z}_{ij}$)과 실제 센서가 측정한 값($z_{ij}$) 사이의 차이를 정량화한다.4 예를 들어, 포즈 

  $x_i$에서 $x_j$로의 상대적 변환을 측정했는데, 현재 추정된 $x_i$와 $x_j$의 값으로 계산한 상대 변환과 다르다면 그 차이가 오차가 된다.

- **정보 행렬 (Information Matrix):** 모든 측정에는 불확실성이 내재되어 있다. 정보 행렬 $\Omega_{ij}$는 이러한 측정의 신뢰도를 나타내는 가중치 역할을 한다. 수학적으로는 측정 오차의 공분산 행렬(covariance matrix) $\Sigma_{ij}$의 역행렬($\Omega_{ij} = \Sigma_{ij}^{-1}$)이다. 이는 불확실성이 작은, 즉 신뢰도 높은 측정값일수록 더 큰 가중치를 부여받아 최적화 과정에 더 강한 영향을 미치도록 만든다.5

### 3.2  포즈 그래프 최적화 (Pose Graph Optimization)

포즈 그래프가 구성되면, SLAM 문제는 그래프 전체의 정합성(consistency)이 가장 높아지는 노드(포즈)들의 최적 구성을 찾는 문제로 귀결된다. 이는 모든 엣지에서 발생하는 오차의 총합을 최소화하는 것과 같다. 확률적인 관점에서 이는 모든 관측값이 주어졌을 때, 가장 가능성이 높은 상태(포즈들의 집합)를 찾는 최대 사후 확률(Maximum a Posteriori, MAP) 추정 문제와 동일하다.23

측정 오차가 가우시안 분포를 따른다고 가정할 때, 이 MAP 추정 문제는 비선형 최소제곱(Non-linear Least Squares) 최적화 문제로 공식화될 수 있다.26 즉, 모든 엣지의 제곱 오차를 각각의 정보 행렬로 가중하여 더한 총 오차 함수 

$F(X)$를 최소화하는 포즈들의 집합 $X^*$를 찾는 것이다.
$$
X^* = \underset{X}{\text{argmin}} F(X) = \underset{X}{\text{argmin}} \sum_{\langle i,j \rangle \in \mathcal{C}} e_{ij}(x_i, x_j)^T \Omega_{ij} e_{ij}(x_i, x_j)
$$
위 식에서 $X = \{x_i\}$는 최적화할 모든 포즈의 집합이며, $\mathcal{C}$는 그래프 내 모든 제약 조건(엣지)의 집합이다.26

이 오차 함수는 로봇의 움직임과 관측 모델이 비선형적이기 때문에 일반적으로 비선형(non-linear)이며 볼록하지 않은(non-convex) 형태를 띤다. 따라서 해를 직접 구하기 어렵고, 가우스-뉴턴(Gauss-Newton) 방법이나 레벤버그-마쿼트(Levenberg-Marquardt) 알고리즘과 같은 반복적인(iterative) 수치 최적화 기법을 사용하여 해를 구한다.23 이러한 알고리즘들은 현재의 해 추정치 주변에서 비선형 오차 함수를 선형으로 근사하고, 이 선형화된 문제를 풀어 해를 조금씩 개선해 나가는 과정을 수렴할 때까지 반복한다.23

이 최적화 과정에서 가장 극적인 효과를 발휘하는 것이 바로 루프 클로저(loop closure)이다. 로봇이 긴 경로를 이동한 후 이전에 방문했던 장소를 다시 인식하게 되면, 시간적으로 멀리 떨어진 두 노드 사이에 강력한 제약 조건(엣지)이 추가된다. 이 새로운 엣지는 그동안 누적된 오도메트리 드리프트를 전체 그래프에 걸쳐 분산시키고 보정하는 역할을 한다. 그 결과, 지도의 왜곡이 수정되고 전역적인 일관성이 획기적으로 향상된다.1

### 3.3  구현 프레임워크: GTSAM의 역할과 구조

GTSAM(Georgia Tech Smoothing and Mapping)은 위에서 설명한 그래프 기반 최적화 문제를 효율적으로 해결하기 위해 조지아 공과대학교에서 개발한 오픈소스 C++ 라이브러리이다.27 GTSAM은 SLAM 문제를 인수 그래프(Factor Graph)라는 확률적 그래픽 모델을 사용하여 표현한다.27 인수 그래프는 변수(노드)와 인수(factor, 즉 확률적 제약)를 명시적으로 분리하여 문제의 구조를 더 명확하게 시각화하고 다룰 수 있게 해준다.

GTSAM을 사용한 일반적인 포즈 그래프 최적화 과정은 다음과 같다.

1. `gtsam::NonlinearFactorGraph` 객체를 생성하여 비선형 인수 그래프를 만든다.
2. `PriorFactor`를 사용하여 로봇의 초기 위치에 대한 제약(예: 원점)을 추가한다. 이는 전체 그래프의 기준점을 고정하는 역할을 한다.
3. 오도메트리 측정값과 루프 클로저 측정값을 `BetweenFactor` 형태로 그래프에 추가한다. 각 팩터에는 측정값과 그 불확실성을 나타내는 노이즈 모델(정보 행렬)이 함께 제공된다.
4. `gtsam::Values` 객체에 최적화할 모든 포즈 변수들의 초기 추정값을 삽입한다.
5. `gtsam::LevenbergMarquardtOptimizer` 또는 `gtsam::GaussNewtonOptimizer`와 같은 최적화 객체를 생성하고, `optimize()` 메소드를 호출하여 최적의 해를 구한다.28

GTSAM의 가장 큰 장점은 SLAM 문제가 가지는 희소성(sparsity)을 적극적으로 활용한다는 점이다. 대부분의 측정(엣지)은 소수의 변수(노드)에만 관련되므로, 최적화 과정에서 다루어야 하는 행렬(자코비안, 헤시안)은 대부분의 원소가 0인 희소 행렬이 된다. GTSAM은 이러한 희소 구조를 활용하는 선형대수 기법을 통해 수천, 수만 개의 변수를 갖는 대규모 최적화 문제도 매우 효율적으로 해결할 수 있다.27

이러한 그래프 기반 최적화의 원리는 GRAND-SLAM의 전역적 일관성 확보 메커니즘을 이해하는 데 결정적이다. GRAND-SLAM은 대규모 환경을 효율적으로 처리하기 위해 전체 맵을 여러 개의 서브맵(submap)으로 분할한다. 각 서브맵은 그 자체로는 지역적으로 일관성을 유지하지만, 서브맵들을 연결하는 과정에서 오도메트리 드리프트가 누적되어 전체 맵의 일관성은 깨지게 된다. GRAND-SLAM은 이 문제를 해결하기 위해 그래프 기반 최적화를 백엔드(back-end)로 채택한다. 시스템은 각 서브맵의 기준 포즈를 하나의 '슈퍼 노드(super-node)'처럼 취급하고, 서브맵 간의 연속적인 추적 결과와 루프 클로저에 의해 발견된 상대적 변환을 엣지로 하는 전역 포즈 그래프를 구성한다. 이 거대한 포즈 그래프는 GTSAM을 통해 최적화되며, 그 결과 모든 서브맵의 포즈가 전역적으로 가장 일관된 상태로 재조정된다.13 결국 GRAND-SLAM의 아키텍처는 '지역적 밀도 맵핑(Local Dense Mapping)'을 통한 실시간 처리 및 확장성 확보와 '전역적 희소 최적화(Global Sparse Optimization)'를 통한 일관성 보장이라는 두 가지 목표를 분리하여 달성하는, 현대 대규모 SLAM 시스템의 정석적인 구조를 따른다고 볼 수 있다.

## 4.  장면 표현의 혁신 - 3D 가우시안 스플래팅

SLAM 기술의 발전은 단순히 위치를 추정하는 알고리즘의 개선뿐만 아니라, 주변 환경, 즉 '맵'을 어떻게 표현할 것인가에 대한 고민과 함께 이루어져 왔다. 최근 이 분야에서 가장 주목받는 기술 중 하나가 바로 3D 가우시안 스플래팅(3D Gaussian Splatting, 3DGS)이다. 이 기술은 전례 없는 수준의 사실적인 렌더링 품질과 실시간 성능을 동시에 제공하며, SLAM의 맵 표현 방식에 새로운 가능성을 제시하고 있다.

### 4.1  3DGS의 기본 원리: NeRF를 넘어선 실시간 렌더링

3DGS는 3차원 장면을 수백만 개의 3D 가우시안(Gaussian)들의 집합으로 표현하는 명시적(explicit) 표현 방식이다.30 이는 이전에 각광받던 NeRF(Neural Radiance Fields)와 같은 암시적(implicit) 표현 방식과 대비된다. NeRF는 장면을 좌표를 입력받아 색상과 밀도를 출력하는 거대한 신경망으로 표현하여 매우 높은 품질의 이미지를 생성할 수 있었지만, 신경망을 학습시키고 렌더링하는 데 막대한 계산 비용이 소요되어 실시간 응용에는 한계가 있었다.30

3DGS는 이러한 한계를 극복하기 위해 제안되었다. 신경망 대신, 각기 고유한 속성을 가진 수많은 3D 가우시안으로 장면을 직접 모델링하고, 이를 매우 효율적인 미분 가능한(differentiable) 래스터라이제이션(rasterization) 파이프라인을 통해 렌더링한다. 그 결과, 3DGS는 NeRF 수준의 시각적 품질을 유지하면서도 1080p 해상도에서 초당 100 프레임 이상의 실시간 렌더링 속도를 달성했다.30 이처럼 빠른 렌더링과 학습 속도는 SLAM과 같이 실시간 처리가 필수적인 로보틱스 응용에 3DGS를 매우 매력적인 선택지로 만들었다.

### 4.2  3DGS의 수학적 정의

3DGS에서 장면을 구성하는 각각의 3D 가우시안은 다음과 같은 파라미터들로 정의된다 33:

- **위치 (Position, $\mu$):** 3차원 공간상에서 가우시안의 중심점을 나타내는 3차원 벡터 $\mu \in \mathbb{R}^3$이다.
- **공분산 (Covariance, $\Sigma$):** 가우시안의 형태(타원체의 방향)와 크기(타원체의 각 축 길이)를 결정하는 3x3 대칭 양의 정부호 행렬(symmetric positive definite matrix)이다. 최적화의 효율성과 안정성을 위해, 공분산 행렬 $\Sigma$는 3차원 스케일링 벡터 $s$와 회전을 나타내는 쿼터니언 $q$로 분해하여 저장하고 최적화한다. 회전 행렬 $R$이 쿼터니언 $q$로부터 얻어지면, 공분산은 $\Sigma = RSS^TR^T$로 계산된다.33
- **불투명도 (Opacity, $\alpha$):** 0과 1 사이의 값을 가지며, 해당 가우시안이 렌더링 과정에서 뒤쪽의 다른 가우시안들을 얼마나 가리는지를 결정한다.
- **색상 (Color):** 가우시안의 색상은 보는 방향에 따라 달라지는 현실 세계의 미묘한 광학 효과(view-dependent effects)를 표현하기 위해 구면 조화 함수(Spherical Harmonics, SH)의 계수들로 표현된다.33

렌더링 과정은 다음과 같다. 먼저, 모든 3D 가우시안들은 주어진 카메라의 월드-투-카메라 변환 행렬 $W$를 통해 카메라 좌표계로 변환된다. 이때 3D 공분산 행렬 $\Sigma$는 2D 이미지 평면에 투영되어 2D 공분산 행렬 $\Sigma'$로 변환된다. 이 변환은 투영 변환의 선형 근사인 자코비안 행렬 $J$를 사용하여 $\Sigma' = JW\Sigma W^T J^T$와 같이 계산된다.33 그 후, 고도로 최적화된 타일 기반의 래스터라이저가 이 2D 가우시안들을 이미지 평면에 '뿌리는(splatting)' 방식으로 각 픽셀의 최종 색상 값을 효율적으로 계산한다. 이 전체 과정은 미분 가능하도록 설계되어, 렌더링된 이미지와 실제 이미지 간의 오차를 역전파(backpropagation)하여 가우시안 파라미터들을 최적화할 수 있다.

### 4.3  SLAM에서의 3DGS 활용

3DGS의 등장은 SLAM 커뮤니티에 큰 반향을 일으켰다. 3DGS를 맵 표현 방식으로 사용하면 다음과 같은 장점들을 얻을 수 있다.

- **고품질 밀도 맵:** 기존의 희소 맵과 달리, 3DGS는 환경의 기하학적 구조뿐만 아니라 텍스처와 색상까지 매우 사실적으로 표현하는 밀도 높은 맵을 생성한다.
- **다양한 응용 가능성:** 이렇게 생성된 사실적인 맵은 로봇의 위치 추정을 넘어, 증강 현실(AR) 콘텐츠를 오버레이하거나, 가상 현실(VR) 환경을 구축하거나, 디지털 트윈(Digital Twin)을 생성하는 등 다양한 고급 응용의 기반이 된다.6
- **새로운 최적화 패러다임:** 미분 가능한 렌더러는 SLAM의 최적화 방식을 근본적으로 바꿀 수 있다. 기존의 기하학적 오차(예: 재투영 오차) 대신, 렌더링된 이미지와 실제 센서 이미지 간의 광도 오차(photometric error)를 직접 최소화하는 것이 가능해진다. 이는 1장에서 논의된 직접 방식 SLAM의 철학과 맞닿아 있으며, GS-SLAM, MonoGS 등 3DGS를 SLAM에 적용하려는 여러 선구적인 연구들로 이어졌다.11

### 4.4  3DGS 기반 SLAM의 기술적 과제

이러한 엄청난 잠재력에도 불구하고, 3DGS를 실제 SLAM 시스템에 적용하는 데에는 여러 기술적 과제가 존재한다.

- **메모리 및 계산 복잡도:** 고품질의 장면을 표현하기 위해서는 수백만 개에 달하는 3D 가우시안이 필요하며, 각 가우시안은 위치, 공분산, 불투명도, SH 계수 등 많은 파라미터를 가진다. 이는 막대한 메모리 사용량과 저장 공간을 요구하며, 최적화 과정의 계산 복잡도를 증가시킨다.10
- **기하학적 정확도:** 3DGS는 본질적으로 시각적 렌더링 품질에 최적화된 표현 방식이다. 따라서 TSDF(Truncated Signed Distance Function) 퓨전과 같이 기하학적 정확도에 초점을 맞춘 전통적인 밀도 맵핑 방식에 비해 표면의 정밀도나 기하학적 일관성이 떨어질 수 있다.34 이는 로봇의 충돌 회피나 정밀한 조작과 같이 정확한 기하 정보가 중요한 작업에 영향을 줄 수 있다.
- **동적 환경 처리:** 대부분의 초기 3DGS-SLAM 시스템들은 세상이 정적(static)이라는 강한 가정 하에 설계되었다. 따라서 실제 환경에 존재하는 사람, 차량 등 움직이는 객체들은 맵에 잔상(ghosting)을 남기거나 잘못된 기하 구조를 생성하여 맵을 오염시키고, 심각한 경우 추적 실패를 유발할 수 있다.35
- **센서 융합 및 강인성:** 많은 3DGS-SLAM 연구들이 RGB-D 카메라에 의존하고 있어, 깊이 정보를 얻을 수 없거나 조명이 어둡고 텍스처가 부족한 환경에서는 성능이 저하된다. LiDAR와 같은 다른 센서와의 효과적인 융합을 통해 다양한 환경에서의 강인성을 확보하는 것이 중요한 연구 과제로 남아있다.35

결론적으로, 3DGS는 SLAM의 맵 표현 방식을 혁신하고 최적화의 기준을 기하학에서 렌더링 품질로 확장시키는 패러다임 전환을 이끌고 있다. GRAND-SLAM은 이러한 3DGS의 잠재력을 인식하고, 위에서 언급된 기술적 과제들, 특히 메모리 문제와 대규모 환경으로의 확장성 문제를 해결하며 이를 다중 에이전트 협업이라는 새로운 영역으로 이끌어가는 중요한 시도라고 할 수 있다.

## 5.  GRAND-SLAM 심층 분석

GRAND-SLAM은 3D 가우시안 스플래팅(3DGS)이라는 혁신적인 장면 표현 기술을 기반으로, 기존 SLAM 시스템의 한계를 넘어 대규모 환경에서의 다중 에이전트 협업이라는 도전적인 목표를 달성하기 위해 설계된 정교한 시스템이다. 본 장에서는 GRAND-SLAM의 핵심 방법론을 구성 요소별로 심층적으로 분해하고 분석한다.

### 5.1  개요: 대규모, 다중 에이전트 환경을 위한 가우시안 재구성

GRAND-SLAM은 그 이름 'Gaussian Reconstruction via Multi-Agent Dense SLAM'에서 알 수 있듯이, 3D 가우시안을 이용한 밀도 높은 장면 재구성을 다수의 에이전트가 협력하여 수행하는 SLAM 시스템이다.12 이 시스템의 핵심 목표는 기존 3DGS 기반 SLAM 연구들이 주로 소규모의 통제된 실내 환경에 머물렀던 것과 달리, 이를 광활하고 복잡한 실외 환경 및 여러 로봇이 동시에 임무를 수행하는 다중 에이전트 시나리오로 확장하는 것이다.6

이러한 목표를 달성하기 위해 GRAND-SLAM은 SLAM의 고전적인 난제인 확장성(scalability)과 전역적 일관성(global consistency)을 동시에 해결해야 한다. 이를 위해 시스템은 세 가지 핵심적인 전략을 유기적으로 결합한다: (i) 계산 복잡도를 제어하기 위한 서브맵(submap) 기반의 지역적 최적화, (ii) 누적된 오차를 보정하기 위한 단일 및 다중 로봇 간의 강인한 루프 클로저, 그리고 (iii) 전체 맵의 일관성을 최종적으로 보장하기 위한 포즈 그래프 최적화 백엔드이다.6

### 5.2  핵심 방법론 1: 서브맵 기반의 지역적 최적화 (Local Optimization over Submaps)

대규모 환경을 단일 맵으로 관리하고 수백만, 수천만 개의 가우시안을 한 번에 최적화하는 것은 현실적으로 불가능하다. GRAND-SLAM은 이 확장성 문제를 해결하기 위해 '분할 정복(divide and conquer)' 전략을 채택한다. 즉, 전체 맵을 관리 가능한 크기의 여러 서브맵(submap) 단위로 분할하여 증분적으로 구축한다.6

- **서브맵 관리:** 새로운 서브맵은 에이전트가 현재 활성화된 서브맵의 시작점으로부터 미리 정의된 거리나 회전 각도 이상으로 이동했을 때 생성된다. 이 접근법은 카메라 추적(tracking)과 지도 작성(mapping)과 같은 계산 비용이 높은 작업의 범위를 현재 활성화된 서브맵으로 한정시킨다. 이를 통해 전체 환경의 크기가 증가하더라도 시스템의 계산 복잡도가 거의 일정하게 유지되어 뛰어난 확장성을 보장한다.13
- **지역 좌표계 최적화:** 대규모 환경에서 에이전트가 전역 원점(global origin)으로부터 매우 멀리 떨어지게 되면, 포즈 파라미터의 스케일 차이로 인해 최적화 과정에서 경사도 불균형(imbalanced gradients) 문제가 발생하여 수렴이 불안정해질 수 있다. GRAND-SLAM은 이 문제를 방지하기 위해, 카메라 포즈 최적화를 전역 좌표계가 아닌 현재 서브맵의 지역 좌표계(local frame)를 기준으로 수행한다. 이는 최적화의 수치적 안정성을 크게 향상시킨다.13
- **렌더링 손실 함수:** 서브맵 내의 3D 가우시안 파라미터들은 해당 서브맵에 속한 키프레임(keyframe)들을 얼마나 잘 재구성하는지를 기준으로 최적화된다. 이를 위해 GRAND-SLAM은 렌더링 손실 함수(Rendering Loss Function)를 정의한다. 이 손실은 현재 가우시안 맵을 특정 키프레임의 시점에서 렌더링한 가상 이미지와 실제 카메라로 촬영된 원본 이미지 사이의 차이를 측정한다. 구체적으로, L1 색상 오차와 깊이 오차의 가중 합으로 구성된다.13

$$
\mathcal{L}_{\text{render}} = \lambda \mathcal{L}_{\text{L1}} + (1 - \lambda) \mathcal{L}_{\text{depth}}
$$

여기서 $\mathcal{L}_{\text{L1}} = |C_k - \hat{C}*k|$는 픽셀별 색상 값의 L1 노름(norm)이고, $\mathcal{L}*{\text{depth}} = |D_k - \hat{D}_k|$는 픽셀별 깊이 값의 L1 노름이다. $C_k$와 $D_k$는 키프레임 $k$의 실제 색상 및 깊이 이미지이며, $\hat{C}_k$와 $\hat{D}_k$는 현재 맵을 렌더링하여 얻은 예측값이다. $\lambda$는 두 손실 간의 균형을 조절하는 가중치이다.13 이 손실 함수를 최소화하는 과정은 맵이 실제 관측을 가장 잘 설명하도록 가우시안들의 위치, 모양, 색상 등을 조정하는 역할을 한다.

### 5.3  핵심 방법론 2: 강인한 루프 클로저 (Robust Loop Closure)

지역적 최적화만으로는 시간이 지남에 따라 누적되는 오도메트리 드리프트를 막을 수 없다. 전역적 일관성을 확보하기 위해서는 로봇이 과거에 방문했던 장소를 다시 인식하고, 이를 통해 맵의 오차를 보정하는 루프 클로저(loop closure)가 필수적이다. GRAND-SLAM은 이 과정을 단일 에이전트 내(intra-agent)뿐만 아니라 서로 다른 에이전트 간(inter-agent)에도 수행할 수 있도록 설계되었다.6

루프 클로저 과정은 Coarse-to-Fine, 즉 거친 수준에서 정밀한 수준으로 점차 정합(alignment)의 정확도를 높여가는 방식으로 이루어진다.

1. **후보 탐색 (Candidate Detection):** 각 키프레임은 NetVLAD와 같은 장소 인식 기술자를 사용하여 표현된다. 시스템은 현재 키프레임의 기술자를 과거 키프레임들의 데이터베이스와 비교하여 유사도가 높은 후보들을 찾는다.13
2. **거친 정합 (Coarse Alignment):** 루프 클로저 후보가 발견되면, 두 서브맵 간의 초기 상대 변환($T_{ij}$)을 추정해야 한다. 특히 에이전트 간 루프나 시간 간격이 매우 긴 루프의 경우, 단순히 누적된 오도메트리 정보만으로는 신뢰할 수 있는 초기값을 얻기 어렵다. 이를 위해 GRAND-SLAM은 두 서브맵에 해당하는 밀도 높은 RGB-D 데이터를 직접 사용하여 정합(registration)을 수행하고, 이를 통해 강인한 초기 변환 값을 계산한다.13
3. **정밀 정합 (Fine Refinement):** 거친 정합으로 얻은 초기 변환 값은 Point-to-Plane ICP(Iterative Closest Point) 알고리즘을 통해 더욱 정밀하게 다듬어진다. 이 과정은 두 서브맵으로부터 생성된 포인트 클라우드 간의 기하학적 오차를 최소화하여 매우 정확한 상대 포즈 변환을 최종적으로 얻어낸다.13

### 5.4  핵심 방법론 3: 전역적 일관성을 위한 포즈 그래프 최적화 (Pose Graph Optimization for Global Consistency)

성공적으로 검증된 루프 클로저 제약은 시스템의 백엔드인 포즈 그래프 최적화 프레임워크에 통합된다. 이 백엔드는 2장에서 설명한 그래프 기반 SLAM의 원리를 그대로 따른다.

- **그래프 구성:** 전역 포즈 그래프의 노드는 각 서브맵의 기준 키프레임들의 전역 포즈($T_k$)를 나타낸다. 엣지는 두 종류의 제약으로 구성된다. 하나는 카메라 추적 과정에서 얻어지는 연속적인 서브맵 간의 상대 포즈 제약이고, 다른 하나는 루프 클로저를 통해 얻어지는 비연속적인(시간적으로 멀리 떨어진) 서브맵 간의 상대 포즈 제약이다.13

- **GTSAM 기반 최적화:** 이 포즈 그래프는 GTSAM 라이브러리를 사용하여 최적화된다.13 최적화의 목표는 모든 제약 조건(엣지)을 가장 잘 만족시키는 노드(서브맵 포즈)들의 전역 배치 

  $T^*$를 찾는 것이다. 이는 다음의 비선형 최소제곱 문제를 푸는 것과 같다.13

$$
T^* = \underset{T}{\text{argmin}} \sum_{(i,j) \in \mathcal{C}} \| \log((T_i^{-1} T_j) Z_{ij}^{-1}) \|^2_{\Sigma_{ij}}
$$

위 식에서 $\mathcal{C}$는 모든 제약(추적 및 루프 클로저)의 집합, $Z_{ij}$는 측정된 상대 변환, $\Sigma_{ij}$는 해당 측정의 공분산 행렬, $T_i, T_j$는 최적화할 서브맵의 전역 포즈이다. $\log(\cdot)$는 변환 행렬을 접선 공간(tangent space)의 벡터로 매핑하는 연산이다.

- **맵 업데이트:** 포즈 그래프 최적화가 완료되면, 모든 서브맵의 전역 포즈가 전역적으로 가장 일관된 값으로 업데이트된다. 마지막으로, 각 서브맵에 속한 모든 3D 가우시안들과 키프레임들은 이렇게 최적화된 서브맵의 전역 변환을 사용하여 지역 좌표계에서 일관된 전역 좌표계로 최종 변환된다. 이 과정을 통해 하나의 거대하고 일관된 3D 가우시안 맵이 완성된다.13

### 5.5  기여점 및 성능 평가

GRAND-SLAM의 핵심적인 학술적 기여는 다음과 같이 요약할 수 있다.6

1. 대규모 장면에 강인하게 확장 가능한, 지역적 서브맵 기반 최적화 방식을 갖춘 RGB-D 3DGS-SLAM 추적 모듈을 제안했다.
2. 광도 및 기하학적 오차를 모두 고려하는 Coarse-to-Fine 최적화 절차를 통해 단일 및 다중 에이전트 루프 클로저를 효과적으로 통합했다.
3. 포즈 그래프 최적화 백엔드를 통해 루프 클로저 제약을 전역적으로 반영함으로써, 장시간 동작 시 발생하는 맵의 드리프트와 추적 오차를 크게 감소시켰다.

이러한 기여를 바탕으로 GRAND-SLAM은 실험적으로도 뛰어난 성능을 입증했다. 대표적으로, 실내 환경 데이터셋인 Replica에서는 기존 다중 에이전트 방법 대비 28% 더 높은 PSNR(Peak Signal-to-Noise Ratio, 렌더링 품질 척도)을 달성했으며, 실제 대규모 실외 환경 데이터셋인 Kimera-Multi에서는 91% 더 낮은 다중 에이전트 추적 오차(ATE, Absolute Trajectory Error)를 기록했다. 이는 GRAND-SLAM이 목표했던 대로 대규모의 도전적인 환경에서 높은 정확도와 강인성을 모두 갖추었음을 보여주는 결과이다.12

SLAM 연구의 오랜 난제는 실시간성을 유지하면서(확장성) 동시에 전역적으로 일관된 맵을 만드는 것(일관성) 사이의 균형을 맞추는 것이었다. 모든 측정값을 고려하는 Full SLAM은 일관성은 높지만 맵이 커지면 계산량이 폭발하고, 최근 데이터만 고려하는 필터링 방식은 빠르지만 드리프트가 누적된다.5 GRAND-SLAM의 아키텍처는 이 딜레마에 대한 정교한 공학적 해법을 제시한다. 실시간성이 중요한 프론트엔드(추적, 지역 매핑)에서는 계산 범위를 서브맵으로 제한하여 확장성을 확보하고, 전역적 일관성을 책임지는 백엔드(포즈 그래프 최적화)는 루프 클로저가 발견될 때 비동기적으로 수행하여 전체 맵의 일관성을 보장한다. 이처럼 실시간 지역 최적화와 비동기적 전역 최적화를 분리하고 결합하는 이원적 구조는 대규모 SLAM 기술이 실용화되기 위한 현대적인 표준 해법을 명확히 보여준다.

| 평가 항목         | ORB-SLAM2                               | DSO                                        | LSG-SLAM                                      | Swarm-SLAM                                 | **GRAND-SLAM**                    |
| ----------------- | --------------------------------------- | ------------------------------------------ | --------------------------------------------- | ------------------------------------------ | --------------------------------- |
| **주요 센서**     | Monocular, Stereo, RGB-D                | Monocular                                  | Stereo                                        | Lidar, Stereo, RGB-D                       | RGB-D                             |
| **장면 표현**     | 희소 포인트 클라우드                    | 희소 포인트 클라우드                       | 3D 가우시안 스플래팅                          | 희소 포즈 그래프                           | 3D 가우시안 스플래팅              |
| **추적 방식**     | 간접 (특징점 기반)                      | 직접 (광도 오차 기반)                      | 하이브리드 (렌더링+특징)                      | 간접 (특징점 기반)                         | 직접 (렌더링 오차 기반)           |
| **백엔드 최적화** | 번들 조정, 포즈 그래프                  | 공동 최적화 (창 내)                        | 포즈 그래프 최적화                            | 분산 포즈 그래프 최적화                    | 포즈 그래프 최적화                |
| **다중 에이전트** | 미지원 (기본)                           | 미지원 (기본)                              | 미지원                                        | 지원 (분산형)                              | 지원 (협력형)                     |
| **주요 특징**     | ORB 특징점 기반의 강인한 실시간 SLAM 16 | 키포인트 없이 텍스처 부족 환경에 강인함 20 | 대규모 실외 환경을 위한 스테레오 3DGS SLAM 37 | 통신 효율적인 분산 협업 SLAM 프레임워크 38 | 대규모 다중 에이전트 3DGS SLAM 12 |

## 6.  종합 고찰 및 미래 전망

GRAND-SLAM은 로보틱스와 컴퓨터 비전 분야의 최신 기술들을 집약하여 SLAM의 새로운 가능성을 연 중요한 이정표이다. 본 장에서는 GRAND-SLAM의 기술적 의의를 종합적으로 고찰하고, 이를 통해 열리게 될 미래 응용 분야와 앞으로 해결해야 할 과제들을 전망한다.

### 6.1  GRAND-SLAM의 의의: 직접 방식과 그래프 기반 방식의 성공적 융합

GRAND-SLAM의 아키텍처는 Visual SLAM의 두 가지 핵심 철학, 즉 직접 방식과 그래프 기반 방식을 성공적으로 융합한 현대적 사례로 평가할 수 있다. 시스템의 프론트엔드, 즉 실시간 카메라 추적 모듈은 3DGS 맵의 렌더링 결과와 실제 이미지 간의 광도 및 깊이 오차를 최소화하는 방식으로 동작한다. 이는 1장에서 논의된 직접 방식의 원리를 최신 맵 표현 기술에 맞게 정교하게 발전시킨 것이다. 한편, 시스템의 백엔드는 2장에서 설명한 그래프 기반 최적화의 원리를 충실히 따른다. 서브맵들의 포즈를 노드로, 추적 및 루프 클로저 결과를 엣지로 하는 전역 포즈 그래프를 구성하고 최적화함으로써 누적된 오차를 보정하고 전역적 일관성을 확보한다.

이러한 이원적 구조는 3장에서 다룬 혁신적인 맵 표현 방식인 3DGS를 대규모, 다중 에이전트라는 실용적이고 도전적인 문제에 적용하기 위한 필연적인 설계 선택이었다. 3DGS의 높은 표현력과 계산 복잡도를 지역적으로 제어하면서, 그래프 최적화의 강력한 전역 일관성 보장 능력을 결합함으로써, GRAND-SLAM은 확장성, 일관성, 사실성이라는 세 마리 토끼를 동시에 잡으려는 시도를 성공적으로 보여주었다.

### 6.2  다중 에이전트 협업 SLAM의 미래: 응용 분야 확장

GRAND-SLAM과 같이 강력하고 확장 가능한 협업 SLAM(Collaborative SLAM, C-SLAM) 기술은 단일 로봇의 한계를 뛰어넘어 다양한 미래 응용 분야의 문을 열 것이다.

- **재난 대응 및 수색 구조:** 지진이나 건물 붕괴와 같이 GPS 사용이 불가능한 재난 현장에서, 다수의 드론이나 지상 로봇이 동시에 투입되어 신속하게 3차원 지도를 공동으로 작성할 수 있다. 이를 통해 구조대원들은 실시간으로 내부 구조를 파악하고, 위험 지역을 식별하며, 고립된 생존자의 위치를 효율적으로 탐색할 수 있다.39
- **대규모 AR 클라우드 및 디지털 트윈:** 도시 전체나 대형 쇼핑몰, 공항과 같은 광활한 공간에 대해, 수많은 사용자의 스마트폰과 AR 기기들이 협력하여 지속적으로 업데이트되는 공유 3D 맵을 구축할 수 있다. 이 'AR 클라우드'는 모든 사용자에게 위치에 기반한 끊김 없는(seamless) 증강 현실 경험을 제공하고, 현실 세계의 가상 복제인 '디지털 트윈'을 구축하여 시뮬레이션, 유지보수, 도시 계획 등에 활용될 수 있는 핵심 기반 기술이 될 것이다.42
- **자율 물류 및 우주 탐사:** 광대한 물류 창고에서 여러 대의 로봇이 협력하여 실시간으로 재고의 위치를 파악하고 최적의 경로로 물품을 운송할 수 있다.44 또한, 화성이나 달과 같은 미지의 행성 표면을 탐사할 때, 다수의 로버가 협력하여 광범위한 지역을 효율적으로 탐사하고 정밀한 3D 지도를 공동으로 작성함으로써 탐사 임무의 효율과 과학적 성과를 극대화할 수 있다.45

### 6.3  남은 과제와 향후 연구 방향

GRAND-SLAM이 제시한 방향성은 유망하지만, 완전한 실용화를 위해서는 여전히 해결해야 할 기술적 과제들이 남아있다.

- **통신 제약 및 분산 처리:** GRAND-SLAM은 루프 클로저 정보 공유 등에서 중앙 집중적인 서버의 존재를 암시한다. 그러나 실제 로봇 군집(swarm)이나 야외 환경에서는 로봇 간 통신이 불안정하거나 대역폭이 매우 제한적일 수 있다. 따라서 중앙 서버 없이 각 에이전트가 독립적으로 판단하고 최소한의 정보 교환만으로 협업할 수 있는 완전한 분산(decentralized) C-SLAM 알고리즘에 대한 연구가 필수적이다.38
- **실시간성과 계산 복잡도:** 3DGS 맵의 최적화와 렌더링은 여전히 높은 계산 자원을 요구한다. 특히 자원이 제한된 소형 드론이나 모바일 기기에 탑재되기 위해서는, 가우시안 모델 압축, 효율적인 자료구조 설계, 하드웨어 가속 등 계산 복잡도를 획기적으로 줄이기 위한 추가적인 연구가 필요하다.10
- **동적 환경 및 장기 자율성:** 현실 세계는 정적이지 않다. 움직이는 사람, 차량과 같은 동적 객체들을 효과적으로 감지하고, 이를 맵에서 분리하거나 혹은 별도로 추적하는 기술이 통합되어야 한다. 또한, 조명, 날씨, 계절의 변화와 같이 시간이 지남에 따라 환경의 외양이 변하는 상황에서도 강인하게 동작하며 수개월, 수년에 걸쳐 지도를 유지보수할 수 있는 장기(long-term) 자율성 확보가 중요한 미래 연구 방향이다.35

결론적으로, GRAND-SLAM과 같은 기술의 궁극적인 지향점은 단순히 로봇이 길을 잃지 않도록 돕는 것을 넘어선다. 이는 물리적 세계와 디지털 세계를 매끄럽게 융합하는, 지속적이고 공유 가능한 **'공간 인지 플랫폼(Spatial Cognition Platform)'**을 구축하는 거대한 비전의 일부이다. 단일 로봇의 SLAM이 로봇에게 '개인적인' 지도를 제공했다면, C-SLAM은 여러 로봇에게 '공유된' 지도를 제공하며 협업의 문을 열었다.41 GRAND-SLAM은 이 공유된 지도를 매우 사실적이고(3DGS), 전역적으로 일관되며(Graph Optimization), 대규모로 확장 가능하도록(Submaps) 만들었다.

이렇게 생성된 고품질의 공유 맵은 더 이상 로봇만을 위한 것이 아니다. 인간 작업자는 AR 헤드셋을 통해 이 맵에 접속하여 로봇의 시야를 공유하고 원격으로 작업을 지시할 수 있다. 인공지능 에이전트는 이 맵을 분석하여 최적의 물류 경로를 계획할 수 있다. 즉, 이 맵은 물리적 공간에 대한 '단일 진실 공급원(Single Source of Truth)'으로서, 로봇, 인간, AI를 포함한 모든 종류의 에이전트가 시공간적으로 일관된 디지털 정보를 바탕으로 상호작용하는 새로운 패러다임의 기반이 된다. 이는 AR 클라우드, 산업용 메타버스, 스마트 시티와 같은 미래 기술을 현실로 만드는 핵심 구성 요소가 될 것이며, GRAND-SLAM은 그 미래를 향한 중요한 한 걸음을 내디딘 것으로 평가할 수 있다.

#### 참고자료

1. SLAM-past-present-future.pdf - cs.wisc.edu, accessed August 6, 2025, https://pages.cs.wisc.edu/~jphanna/teaching/25spring_cs639/resources/SLAM-past-present-future.pdf
2. What Is SLAM (Simultaneous Localization and Mapping)? - MATLAB & Simulink, accessed August 6, 2025, https://www.mathworks.com/discovery/slam.html
3. Visual Simultaneous Localization and Mapping: A Survey - ResearchGate, accessed August 6, 2025, https://www.researchgate.net/publication/234081012_Visual_Simultaneous_Localization_and_Mapping_A_Survey
4. (PDF) A tutorial on graph-based SLAM - ResearchGate, accessed August 6, 2025, https://www.researchgate.net/publication/231575337_A_tutorial_on_graph-based_SLAM
5. A Tutorial on Graph-Based SLAM, accessed August 6, 2025, http://www2.informatik.uni-freiburg.de/~stachnis/pdf/grisetti10titsmag.pdf
6. Local Optimization for Globally Consistent Large-Scale Multi-Agent Gaussian SLAM - arXiv, accessed August 6, 2025, https://arxiv.org/html/2506.18885v1
7. Visual SLAM: What Are the Current Trends and What to Expect? - PMC, accessed August 6, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9735432/
8. A survey of state-of-the-art on visual SLAM | Request PDF - ResearchGate, accessed August 6, 2025, https://www.researchgate.net/publication/361091154_A_survey_of_state-of-the-art_on_visual_SLAM
9. raulmur/ORB_SLAM2: Real-Time SLAM for Monocular, Stereo and RGB-D Cameras, with Loop Detection and Relocalization Capabilities - GitHub, accessed August 6, 2025, https://github.com/raulmur/ORB_SLAM2
10. Compact 3D Gaussian Splatting For Dense Visual SLAM (2403.11247v2) - Emergent Mind, accessed August 6, 2025, https://www.emergentmind.com/articles/2403.11247
11. GS-SLAM: Dense Visual SLAM with 3D Gaussian Splatting - CVF Open Access, accessed August 6, 2025, http://openaccess.thecvf.com/content/CVPR2024/papers/Yan_GS-SLAM_Dense_Visual_SLAM_with_3D_Gaussian_Splatting_CVPR_2024_paper.pdf
12. [2506.18885] GRAND-SLAM: Local Optimization for Globally Consistent Large-Scale Multi-Agent Gaussian SLAM - arXiv, accessed August 6, 2025, https://arxiv.org/abs/2506.18885
13. [Literature Review] GRAND-SLAM: Local Optimization for Globally Consistent Large-Scale Multi-Agent Gaussian SLAM - Moonlight | AI Colleague for Research Papers, accessed August 6, 2025, https://www.themoonlight.io/en/review/grand-slam-local-optimization-for-globally-consistent-large-scale-multi-agent-gaussian-slam
14. Direct and Indirect vSLAM Fusion for Augmented Reality - PMC, accessed August 6, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC8404931/
15. Understanding how Direct Visual SLAM works | Kudan Global, accessed August 6, 2025, https://www.kudan.io/blog/direct-visual-slam/
16. 5.ORB_SLAM2 basics - Yahboom, accessed August 6, 2025, [http://www.yahboom.net/public/upload/upload-html/1700209418/5.ORB_SLAM2%20basics.html](http://www.yahboom.net/public/upload/upload-html/1700209418/5.ORB_SLAM2 basics.html)
17. 8. ORB_SLAM2 basics - Yahboom, accessed August 6, 2025, [http://www.yahboom.net/public/upload/upload-html/1672835847/8.%20ORB_SLAM2%20basics.html](http://www.yahboom.net/public/upload/upload-html/1672835847/8. ORB_SLAM2 basics.html)
18. A review on ORB-SLAM-2 paper. -By Kanishk Vishwakarma, SLAM... | by Sally Robotics, accessed August 6, 2025, https://medium.com/@sallyrobotics.blog/a-review-on-orb-slam-2-paper-3554d4fcaa7c
19. The Advantages of a Joint Direct and Indirect VSLAM in AR - Squarespace, accessed August 6, 2025, https://static1.squarespace.com/static/5c3f69e1cc8fedbc039ea739/t/5d029325d893b70001e4d593/1560449835995/27_younes_final.pdf
20. Visual SLAM - DSO: Direct Sparse Odometry - Computer Vision Group, accessed August 6, 2025, https://cvg.cit.tum.de/research/vslam/dso
21. Direct Sparse Odometry - Jakob Engel, accessed August 6, 2025, https://jakobengel.github.io/pdf/DSO.pdf
22. Direct Sparse Odometry SLAM - Artificial Intelligence Research, accessed August 6, 2025, https://ai-mrkogao.github.io/slam/DSOSLAM/
23. Graph SLAM: From Theory to Implementation - Federico Sarrocco, accessed August 6, 2025, https://federicosarrocco.com/blog/graph-slam-tutorial
24. Graph-based SLAM basics. Simultaneous Localization and Mapping... - Medium, accessed August 6, 2025, https://medium.com/@fatlip/graph-based-slam-basics-f84501525f24
25. ORB SLAM 2 under the hood. Brief SLAM undertsanding | by Suvasis Mukherjee | Medium, accessed August 6, 2025, https://medium.com/@suvasism/orb-slam-2-under-the-hood-ad120049eff2
26. Towards a Robust Back-End for Pose Graph SLAM - Niko Sünderhauf, accessed August 6, 2025, https://nikosuenderhauf.github.io/assets/papers/ICRA12-robustSLAM.pdf
27. Factor Graphs and GTSAM, accessed August 6, 2025, https://gtsam.org/tutorials/intro.html
28. Exercises - VNAV - MIT, accessed August 6, 2025, https://vnav.mit.edu/labs/lab7/exercises.html
29. PoseSLAM - GTSAM 4.0.2 documentation - Read the Docs, accessed August 6, 2025, https://gtsam-jlblanco-docs.readthedocs.io/en/latest/PoseSLAM.html
30. 3D Gaussian Splatting for Real-Time Radiance Field Rendering - Inria, accessed August 6, 2025, https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/
31. 3D Gaussian Splatting for Real-Time Radiance Field Rendering - ResearchGate, accessed August 6, 2025, https://www.researchgate.net/publication/372667406_3D_Gaussian_Splatting_for_Real-Time_Radiance_Field_Rendering
32. 3D Gaussian Splatting: Survey, Technologies, Challenges, and Opportunities - arXiv, accessed August 6, 2025, https://arxiv.org/html/2407.17418v1
33. 3D Gaussian Splatting for Real-Time Radiance Field Rendering | Qiang Zhang, accessed August 6, 2025, https://zhangtemplar.github.io/3d-gaussian-splatting/
34. Geometrically Constrained Gaussian Splatting SLAM - OpenReview, accessed August 6, 2025, https://openreview.net/forum?id=w1Pwcx5hPp
35. The 3D Gaussian Splatting SLAM System for Dynamic Scenes Based on LiDAR Point Clouds and Vision Fusion - MDPI, accessed August 6, 2025, https://www.mdpi.com/2076-3417/15/8/4190
36. GRAND-SLAM: Local Optimization for Globally Consistent Large-Scale Multi-Agent Gaussian SLAM - ResearchGate, accessed August 6, 2025, https://www.researchgate.net/publication/392942672_GRAND-SLAM_Local_Optimization_for_Globally_Consistent_Large-Scale_Multi-Agent_Gaussian_SLAM
37. (PDF) Large-Scale Gaussian Splatting SLAM - ResearchGate, accessed August 6, 2025, https://www.researchgate.net/publication/391776235_Large-Scale_Gaussian_Splatting_SLAM
38. (PDF) Swarm-SLAM: Sparse Decentralized Collaborative Simultaneous Localization and Mapping Framework for Multi-Robot Systems - ResearchGate, accessed August 6, 2025, https://www.researchgate.net/publication/375730572_Swarm-SLAM_Sparse_Decentralized_Collaborative_Simultaneous_Localization_and_Mapping_Framework_for_Multi-Robot_Systems
39. Slam Technology on Disaster Response - Scirp.org., accessed August 6, 2025, https://www.scirp.org/journal/paperinformation?paperid=135222
40. SLAM Drones for High-Precision Mapping and Exploration - Envision Beyond, accessed August 6, 2025, https://www.nvisionbeyond.com/slam/
41. Multi-UAV Collaborative Monocular SLAM - Research Collection, accessed August 6, 2025, https://www.research-collection.ethz.ch/bitstream/handle/20.500.11850/272499/eth-50606-01.pdf
42. Enhancing AR Applications with Sensor Fusion - The Critical Role of SLAM Technology, accessed August 6, 2025, https://moldstud.com/articles/p-enhancing-ar-applications-with-sensor-fusion-the-critical-role-of-slam-technology
43. CORB2I-SLAM: An Adaptive Collaborative Visual-Inertial SLAM for Multiple Robots - MDPI, accessed August 6, 2025, https://www.mdpi.com/2079-9292/11/18/2814
44. SwarmMap: Scaling Up Real-time Collaborative Visual SLAM at the Edge - USENIX, accessed August 6, 2025, https://www.usenix.org/system/files/nsdi22-paper-xu_jingao.pdf
45. Collaborative Vision Systems for Space Exploration | Annika Thomas | TEDxMIT - YouTube, accessed August 6, 2025, https://www.youtube.com/watch?v=dfqULX-AADA
46. MAGiC-SLAM: Multi-Agent Gaussian Globally Consistent SLAM - arXiv, accessed August 6, 2025, https://arxiv.org/html/2411.16785v1
47. S3E: A Multi-Robot Multimodal Dataset for Collaborative SLAM - arXiv, accessed August 6, 2025, https://arxiv.org/html/2210.13723v8
48. Swarm SLAM: Challenges and Perspectives - Frontiers, accessed August 6, 2025, https://www.frontiersin.org/journals/robotics-and-ai/articles/10.3389/frobt.2021.618268/full