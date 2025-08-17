# VDB-GPDF

## 1. 서론

### 0.1  로보틱스 3D 환경 표현의 중요성

로봇이 주변 세계를 의미 있고 효과적으로 이해하며 상호작용하기 위해서는 환경에 대한 정밀한 표현이 필수적이다.1 지각(perception), 제어(control), 항법(navigation), 학습(learning), 조작(manipulation) 등 고차원적 임무 수행 능력은 로봇이 구축하는 내부 3D 모델의 품질에 직접적으로 의존한다.3 따라서 현대 로보틱스 연구의 핵심 과제 중 하나는 실제 환경의 물리적 특성을 높은 정확도로 반영하면서, 실시간 온라인 성능을 보장하고, 대규모 시나리오까지 확장 가능한 3D 환경 표현 기술을 개발하는 것이다.1

### 0.2  기존 3D 매핑 기술의 패러다임과 한계

지난 수십 년간 다양한 3D 환경 표현 기술이 제안되었으나, 각각은 뚜렷한 장점과 함께 본질적인 한계를 지녀왔다.

- **TSDF (Truncated Signed Distance Field)**: 이 방식은 실시간 성능과 밀집된(dense) 볼륨 표현 능력 덕분에 널리 채택되었다. 그러나 깊이 센서에서 직접 얻어지는 거리 값은 센서의 시선 방향에 따른 투영(projective) 또는 비투영(non-projective) 부호 거리로서, 실제 공간상의 최단 거리인 유클리드 거리(Euclidean Distance)를 과대평가하는 경향이 있다.1 이러한 거리 왜곡은 정밀한 충돌 회피나 로봇 팔 조작과 같은 작업에서 오차를 유발하는 근본적인 원인이 된다.

- **Octree**: 옥트리는 공간을 계층적으로 분할하여 희소한(sparse) 환경을 메모리 효율적으로 표현하는 데 강점을 보인다.5 하지만 특정 복셀에 접근하기 위해서는 트리 구조를 따라 순회(traversal)해야 하므로, 맵의 규모가 커지고 노드의 수가 증가할수록 데이터 접근 시간이 길어지는 확장성 문제를 내포한다.1 특히 계산 비용이 높은 알고리즘과 결합될 경우, 이러한 접근 시간 증가는 온라인 증분(incremental) 매핑의 심각한 병목 현상을 초래할 수 있다.2

- **GP (Gaussian Process) 기반 거리 필드**: 가우시안 프로세스는 비모수적 베이즈 모델로서, 환경을 확률적(probabilistic)이고 연속적인(continuous) 함수로 표현할 수 있게 한다.3 이는 측정 노이즈와 불확실성을 자연스럽게 모델링하고, 데이터가 없는 영역에 대한 추론을 가능하게 하는 등 이론적으로 매우 매력적인 특성을 제공한다.4 그러나 표준 GP 회귀의 계산 복잡도는 훈련 데이터 수의 세제곱에 비례(

  O(N3))하므로, 수백만 개의 포인트로 구성된 대규모 맵에 실시간으로 적용하는 것은 사실상 불가능에 가깝다.3

이러한 기술들의 발전 과정은 일종의 변증법적 흐름으로 해석될 수 있다. 초기의 연구들은 TSDF나 Octree와 같이 계산 효율성에 초점을 맞춘 결정론적, 구조적 방법론을 '정(Thesis)'으로 제시했다. 그러나 이들의 기하학적 부정확성이나 확장성 한계가 드러나면서, 이에 대한 대안으로 GPDF와 같이 확률론적이고 높은 정확도를 추구하는 모델이 '반(Antithesis)'으로 등장했다. 하지만 GPDF는 극심한 계산 비용이라는 새로운 문제를 야기하며 그 자체로 완벽한 해결책이 되지 못했다.

### 0.3  VDB-GPDF의 제안: 하이브리드 접근법을 통한 문제 해결

VDB-GPDF는 이러한 기술적 딜레마를 해결하기 위해 '정'과 '반'의 패러다임을 융합하는 '합(Synthesis)'의 단계에 해당하는 하이브리드 프레임워크를 제안한다.3 이 프레임워크의 핵심 가설은 

**가우시안 프로세스(GP)가 제공하는 정밀하고 확률적인 표현력**과 **OpenVDB 데이터 구조가 제공하는 빠른 접근 속도 및 우수한 확장성**을 결합함으로써, 기존 기술들의 장점은 극대화하고 단점은 상호 보완할 수 있다는 것이다.1 VDB-GPDF는 단순히 두 기술을 나란히 사용하는 것을 넘어, L-GPDF와 G-GPDF라는 이중 GP 구조를 통해 계산 문제를 창의적으로 해결하며 '정확성'과 '효율성'이라는 상충 관계(trade-off)를 극복하려는 시도라는 점에서 기술적 성숙의 새로운 단계를 제시한다.

## 1.  핵심 기술 배경

### 1.1  가우시안 프로세스 거리 필드 (Gaussian Process Distance Fields - GPDF)

#### 1.1.1  가우시안 프로세스의 수학적 정의

가우시안 프로세스(GP)는 임의의 유한한 변수 집합이 다변량 가우시안 분포를 따르는 확률 변수들의 집합으로 정의된다.7 이는 점(point)에 대한 분포가 아닌, 함수 전체에 대한 분포를 정의하는 강력한 비모수적(non-parametric) 베이즈(Bayesian) 모델이다.8 GP는 평균 함수 $m(\mathbf{x})$와 공분산 함수(covariance function) 또는 커널(kernel) $k(\mathbf{x}, \mathbf{x}')$에 의해 완전히 결정된다.
$$
f(\mathbf{x}) \sim \mathcal{GP}(m(\mathbf{x}), k(\mathbf{x}, \mathbf{x}'))
$$
여기서 $f(\mathbf{x})$는 입력 $\mathbf{x}$에 대한 함수 값을 나타내는 확률 변수다.

#### 1.1.2  거리 필드 모델링 및 GP 기반 방법론의 진화

거리 필드(Distance Field)는 공간상의 임의의 점 $\mathbf{x}$에서 가장 가까운 표면까지의 최단 거리를 값으로 갖는 스칼라 함수 $d(\mathbf{x})$로, 로봇의 충돌 감지, 경로 계획, 물체 조작 등 다양한 응용에 필수적인 정보를 제공한다.9 GP를 이용한 거리 필드 모델링은 여러 단계를 거쳐 발전해왔다.

- **GPIS (Gaussian Process Implicit Surfaces)**: 초기 연구인 GPIS는 표면(0-레벨셋) 근처에서는 부호 있는 거리(signed distance)를 잘 표현하지만, 표면에서 멀어질수록 함수 값이 0으로 수렴하는 경향이 있다. 이로 인해 전역적인 공간에 대한 거리 정보를 제공하지 못하여 장애물 회피와 같은 응용에 한계가 있었다.11
- **LogGPIS**: 이 한계를 극복하기 위해 제안된 LogGPIS는 GP의 출력에 로그 변환을 적용하여 거리 값이 항상 양수가 되도록 하고, 공간 전체에 걸쳐 유의미한 거리 필드를 정의할 수 있게 했다.11
- **VDB-GPDF의 GPDF 모델**: VDB-GPDF는 이전 연구들을 계승하여 GP 공식에 로그 변환을 적용하는 이론적으로 건전한(theoretically sound) 방식을 채택한다.3 잠재 함수 $h(\mathbf{x})$를 GP로 모델링하고, 이를 $d(\mathbf{x}) = \exp(h(\mathbf{x}))$와 같이 변환하여 최종 거리 값을 추론한다. 이 모델에서 잠재 함수는 보통 평균이 0인 GP를 따른다고 가정하며, 커널 함수로는 주로 제곱 지수(Squared Exponential) 커널이 사용된다.7

$$
k(\mathbf{x}_i, \mathbf{x}_j) = \sigma_f^2 \exp\left(-\frac{1}{2l^2} ||\mathbf{x}_i - \mathbf{x}_j||^2\right)
$$

이 식에서 하이퍼파라미터 l은 데이터 포인트 간의 상관관계 길이를 조절하는 길이 스케일(length-scale)이고, σf2는 함수 값의 전체적인 분산을 나타내는 신호 분산(signal variance)이다.

### 1.2  OpenVDB 데이터 구조

OpenVDB는 영화 산업의 시각 효과(VFX)를 위해 DreamWorks Animation에서 개발된 C++ 라이브러리로, 동적 토폴로지를 가진 고해상도의 희소 볼륨 데이터를 효율적으로 저장하고 조작하는 데 특화되어 있다.14

#### 1.2.1  핵심 철학 및 계층적 구조

OpenVDB의 핵심 철학은 사실상 무한한 3D 인덱스 공간을 지원하면서도 메모리 사용량을 최소화하고, 동시에 빠른 데이터 접근 속도를 보장하는 것이다.17 이를 위해 B+트리와 유사한, 넓고 얕은(wide and shallow) 계층적 트리 구조를 사용한다.1 일반적인 `5-4-3` 구성은 다음과 같은 노드들로 이루어진다 19:

- **Root Node**: 트리의 최상위 노드로, 동적인 분기 계수(dynamic branching factor)를 가지며 전체 공간에 흩어져 있는 하위 노드들을 관리한다.17
- **Internal Nodes**: 중간 노드들로, 고정된 크기의 자식 노드 배열을 가진다. 예를 들어, 5-노드는 32×32×32개의 자식 포인터를, 4-노드는 16×16×16개의 자식 포인터를 가진다.19
- **Leaf Nodes**: 트리의 말단 노드로, 실제 복셀(voxel) 데이터를 조밀한 배열 형태로 저장한다. 예를 들어, 3-노드는 8×8×8개의 복셀 값을 저장한다.16

#### 1.2.2  효율적 데이터 처리 원리

- **희소성(Sparsity)**: 데이터가 존재하지 않는 광대한 빈 공간에 대해서는 트리에 노드를 아예 할당하지 않는다. 이러한 비활성(inactive) 공간에 접근할 경우, 미리 정의된 단일 배경 값(background value)이 반환되어 메모리 낭비를 원천적으로 방지한다.17
- **상수 시간(O(1)) 접근**: 넓고 얕은 트리 구조 덕분에, 임의의 3D 좌표에 해당하는 복셀 데이터에 접근하는 데 걸리는 시간이 트리의 전체 크기와 무관하게 거의 상수 시간에 가깝다. 이는 깊이가 깊어질수록 접근 시간이 로그 스케일로 증가하는(O(logN)) Octree와 비교되는 VDB의 핵심 장점이다.1
- **복셀(Voxels)과 타일(Tiles)**: 데이터의 최소 단위는 리프 노드에 저장되는 **복셀**이다.16 만약 어떤 넓은 영역의 모든 복셀이 동일한 값을 가진다면, 상위 노드(Internal Node)에 직접 그 값을 저장할 수 있는데, 이를 **타일(tile)**이라 부른다. 타일은 해당 공간 블록 전체를 하나의 값으로 압축하여 표현하므로 저장 효율을 극대화한다.16
- **활성 상태(Active State)**: 모든 복셀과 타일은 '활성/비활성'의 이진 상태를 가진다. 이 상태 정보는 조밀한 비트마스크(bitmask)에 별도로 저장되어, '흥미로운' 데이터, 즉 활성 상태의 복셀들만 매우 효율적으로 순회(sparse iteration)하는 것을 가능하게 한다.17

이 두 기술의 결합은 단순한 '빠른 저장소'와 '정확한 모델'의 조합을 넘어선다. VDB의 계층적 공간 분할 구조는 GP 계산을 위한 자연스러운 '단위'를 제공한다. 구체적으로 VDB-GPDF는 각 VDB 리프 노드 내의 복셀 중심점들을 L-GPDF 학습을 위한 훈련 데이터로 사용한다.21 이는 전역 공간에 대해 수행하면 엄청난 비용이 드는 GP 추론을, 현재 센서 데이터가 존재하는 VDB 리프 노드 단위의 작은 지역 문제로 효과적으로 분할하고 제한하는 역할을 한다. 반대로, GP는 VDB의 각 복셀에 단순한 점유 여부가 아닌, 연속적인 유클리드 거리와 그에 대한 신뢰도(불확실성)라는 풍부한 의미론적 정보를 부여한다.1 이처럼 VDB의 구조적 특성과 GPDF의 모델링 특성은 서로의 단점을 보완하고 장점을 극대화하는 강력한 공생 관계를 형성한다.

## 2.  VDB-GPDF 프레임워크 아키텍처

VDB-GPDF는 GPDF의 표현력과 VDB의 효율성을 결합하기 위해 다단계 처리 파이프라인을 채택한다. 전체 시스템은 실시간 증분 업데이트와 고품질 전역 맵 생성을 모두 달성하도록 설계되었다.2

### 2.1  시스템 전체 흐름도

프레임워크의 전체적인 데이터 흐름은 다음과 같은 단계로 구성된다 21:

1. **지역 VDB 구조화**: LiDAR나 RGB-D 카메라로부터 들어온 현재 프레임의 센서 측정값을 월드 좌표계로 변환하고, 이를 복셀화하여 지역(local) VDB 구조를 생성한다.
2. **L-GPDF 학습 및 추론**: 지역 VDB의 리프 노드 내 복셀 중심점들을 훈련 데이터로 사용하여 임시적인 지역 GP 부호 거리 필드(Local GP Signed Distance Field, L-GPDF)를 학습시킨다.
3. **테스트 포인트 생성**: 현재 시야각 내에서 거리 추정이 필요한 지점들, 즉 테스트 포인트(testing points)를 생성한다.
4. **확률적 융합**: L-GPDF를 사용하여 테스트 포인트들의 거리와 불확실성을 추론하고, 이 결과를 가중 평균 방식을 통해 전역(global) VDB 맵에 융합(fusion)한다.
5. **G-GPDF 생성**: 융합이 완료된 전역 VDB 맵으로부터 표면 메시를 추출하고, 이를 기반으로 최종적인 전역 GP 부호 거리 필드(Global GP Signed Distance Field, G-GPDF)를 생성하여 다운스트림 애플리케이션에 제공한다.

### 2.2  지역 GP 부호 거리 필드 (L-GPDF) 생성 및 추론

L-GPDF는 시스템의 실시간 반응성을 담당하는 핵심 구성 요소다. 이는 전체 맵이 아닌, 현재 들어온 센서 데이터만을 처리하기 위한 임시적이고 잠재적인(temporary latent) 모델이다.1 현재 프레임의 포인트 클라우드가 지역 VDB 구조로 변환되면, 각 리프 노드 내의 복셀 중심점들이 해당 지역의 기하학적 구조를 학습하기 위한 GP의 훈련 데이터로 사용된다.21 이후, 광선 투사(ray-casting) 등을 통해 생성된 테스트 포인트 집합에 대해 L-GPDF는 각 포인트의 유클리드 거리(d^), 표면 속성(색상, 강도 등, c^), 그리고 이 추정치들의 불확실성을 나타내는 분산(σ2)을 빠르고 효율적으로 추론한다.3

### 2.3  확률적 융합 (Probabilistic Fusion)

확률적 융합은 L-GPDF가 추론한 새로운 정보를 기존의 전역 맵에 통합하는 과정이다. VDB-GPDF는 각 전역 복셀에 거리 값(d)과 누적 가중치(W)를 저장하며, 가중 평균(weighted average) 방식을 사용하여 맵을 점진적으로 업데이트한다.21

새로운 측정값(d^new)과 그에 해당하는 가중치(w)가 주어지면, 기존의 복셀 값(dold, Wold)은 다음 수식에 따라 갱신된다:
$$
W_{new} = W_{old} + w
$$

$$
d_{new} = \frac{W_{old} \cdot d_{old} + w \cdot \hat{d}_{new}}{W_{new}}
$$

이 과정의 핵심은 가중치 w를 어떻게 결정하느냐에 있다. VDB-GPDF는 L-GPDF로부터 추론된 불확실성(분산)을 기반으로 가중치를 설정함으로써 확률적 융합을 구현한다.1 즉, 추정치의 신뢰도가 높을수록(분산이 작을수록) 더 큰 가중치를 부여하여 맵 업데이트에 더 큰 영향을 미치도록 한다. 공개된 코드 저장소에서는 분산의 역수($w=1/σ2$)를 사용하는 표준적인 방식을 포함한 여러 가중치 전략을 제공한다.22

### 2.4  전역 GP 부호 거리 필드 (G-GPDF) 생성

G-GPDF는 최종적으로 사용자나 다른 로봇 모듈(예: 경로 계획기)에게 제공되는 고품질의 전역 맵이다.3 생성 과정은 다음과 같다. 먼저, 확률적 융합을 통해 업데이트가 완료된 전역 VDB 그리드에서 Marching Cubes와 같은 알고리즘을 사용하여 0-레벨셋, 즉 물체의 표면에 해당하는 밀집된 메시(mesh)를 복구한다.3 그 다음, 이 추출된 표면 메시의 정점(vertices)들을 새로운 훈련 데이터로 삼아 다시 한번 GP 모델을 학습시킨다. 이렇게 생성된 G-GPDF는 GP의 강력한 보간 및 생성 능력 덕분에 맵 상의 임의의 지점에 대해 매우 정확한 유클리드 거리와 해석적으로 계산된 그래디언트(analytical gradient)를 제공할 수 있다.3

VDB-GPDF의 가장 정교한 설계적 특징은 L-GPDF와 G-GPDF라는 이중 GP 구조를 채택한 점에 있다. 이는 '온라인 실시간 처리'와 '오프라인 고품질 표현'이라는 상이한 두 가지 요구사항을 분리하여 해결하는 지능적인 전략이다. 만약 단일 전역 GP 모델만을 사용했다면, 매 프레임마다 들어오는 수천 개의 새로운 데이터 포인트를 기존의 수백만 포인트에 추가하여 모델 전체를 재학습해야 했을 것이다. 이는 $O((N_{old}+N_{new})^3)$의 계산량으로 실시간 처리가 불가능하다. VDB-GPDF는 이 문제를 회피한다. 실시간 융합은 계산 비용이 저렴한 L-GPDF와 VDB의 가중 평균 업데이트를 통해 이루어진다.1 비싼 GP 계산은 현재 프레임의 제한된 데이터에 대해서만 지역적으로 수행된다. 반면, 최종 산출물인 G-GPDF는 필요할 때(on-demand) 또는 주기적으로, 누적되고 정제된 전역 표면 데이터로부터 생성되어 최고의 정확성과 표현력을 보장한다.3 이 이중 구조 덕분에 프레임워크는 매 프레임마다 전역 GP를 업데이트하는 계산적 재앙을 피하면서도, 최종적으로는 전역적으로 일관되고 연속적인 고품질 거리 필드를 제공할 수 있게 된다.

## 3.  성능 비교 및 분석

### 3.1  실험 환경 및 비교 대상

VDB-GPDF의 성능은 Voxblox, VDB-EDT, VDBFusion 등 최신 3D 매핑 프레임워크들과의 비교를 통해 검증되었다.1 평가는 실내 환경을 촬영한 RGB-D 데이터셋(예: Cow and Lady)부터 대규모 실외 환경을 포괄하는 LiDAR 데이터셋(예: KITTI, Newer College)에 이르기까지 다양한 시나리오에서 수행되었다.22

### 3.2  정량적 성능 비교

아래 표는 각 프레임워크의 핵심 특징과 성능을 정성적으로 요약한 것이다.

**표 1: 주요 3D 매핑 프레임워크 정량적 성능 비교**

| **프레임워크 (Framework)** | **핵심 데이터 구조** | **거리 필드 종류**   | **거리 정확도 (RMSE)** | **재구성 정확도** | **처리 시간**   |
| -------------------------- | -------------------- | -------------------- | ---------------------- | ----------------- | --------------- |
| **Voxblox** 4              | Hashed Voxel Grid    | ESDF (TSDF에서 전파) | 높음 (근사 오차)       | 보통              | 빠름            |
| **VDBFusion** 4            | OpenVDB              | TSDF (ESDF 미제공)   | 해당 없음              | 높음              | 매우 빠름       |
| **VDB-EDT** 1              | OpenVDB              | EDF (거리 변환)      | 보통                   | 보통              | 빠름            |
| **VDB-GPDF** 1             | OpenVDB + GPDF       | ESDF (GP 직접 추론)  | **매우 낮음**          | **높음**          | **경쟁력 있음** |

### 3.3  성능 분석

- **거리 추정 정확도**: VDB-GPDF는 비교 대상 프레임워크들 중에서 가장 낮은 거리 추정 오차를 기록했다.1 이는 TSDF로부터 파면 전파(wavefront propagation)를 통해 ESDF를 근사하는 Voxblox나, 점유 그리드에서 거리 변환을 수행하는 VDB-EDT와 근본적으로 다른 접근 방식 덕분이다. VDB-GPDF는 GPDF를 통해 유클리드 거리를 직접 확률적으로 추론하므로, 기하학적으로 더 정확한 값을 얻을 수 있다.1 특히 LiDAR 데이터셋을 사용한 실험에서 VDB-GPDF가 생성한 거리 오차는 다른 방법들보다 일관되게 낮고 부드러운(smoother) 분포를 보였다.1
- **재구성 품질**: 순수 TSDF 기반의 VDBFusion과 비교했을 때, VDB-GPDF는 더 완전하고 자연스러운(more complete and natural) 3D 모델을 재구성했다.1 이는 GP가 가진 생성 모델(generative model)로서의 특성 때문이다.4 GP는 데이터가 관측되지 않은 희소한 영역이나 노이즈가 심한 영역을 통계적 추론에 기반하여 부드럽게 보간(interpolate)하는 능력이 뛰어나, 결과적으로 구멍이 적고 실제와 유사한 표면을 만들어낸다.
- **계산 효율성**: OpenVDB 데이터 구조를 채택한 덕분에, VDB-GPDF는 VDBFusion이나 VDB-EDT와 유사한 수준의 높은 계산 효율성을 유지한다.4 GP 추론이라는 계산 집약적인 과정이 L-GPDF를 통해 현재 시야각 내의 작은 지역으로 한정되기 때문에, 전역 GP 모델을 유지하는 데 따르는 막대한 계산 부담 없이 온라인 증분 성능을 달성할 수 있다.3

VDB-GPDF의 성능 우위는 모든 조건에서 균일하게 나타나기보다는 특정 조건에서 더욱 극대화되는 비대칭성을 보인다. 그 진정한 강점은 평평하고 데이터가 밀집된 '쉬운' 환경보다, 데이터가 희소하거나(sparse) 센서가 표면과 평행하게 움직여 관측이 어려운 '어려운' 환경에서 더욱 두드러진다. VDBFusion과 같은 순수 TSDF 기반 방법은 관측된 깊이 값의 가중 평균에 의존하므로, 관측이 없는 영역이나 스쳐 지나가는 표면에 대한 정보를 생성하기 어렵다. 반면, VDB-GPDF의 GP는 이러한 영역을 확률적으로 추론하고 부드럽게 보간하여 더 완전한 모델을 생성한다.1 이는 GP가 데이터 포인트 사이의 공간을 통계적으로 채우고, 관측이 없는 영역에 대한 예측(평균과 분산)을 제공하는 본질적인 능력에서 기인한다.7 따라서 VDB-GPDF의 성능은 단순 평균 비교를 넘어, 데이터의 불확실성과 희소성이 높은 까다로운 시나리오에서의 강건함(robustness) 측면에서 더 큰 의미를 가진다.

## 4.  응용, 한계 및 미래 연구 방향

### 4.1  주요 응용 분야

VDB-GPDF가 제공하는 정확한 거리 필드, 그래디언트, 불확실성 정보는 기존 매핑 기술로는 접근하기 어려웠던 다양한 다운스트림 애플리케이션의 문을 열었다.

- **공간 음향화 (Spatial Sonification)**: 3D 환경의 기하학적 구조를 실시간으로 소리로 변환하여 시각장애인에게 공간 인지를 돕는 시스템에 VDB-GPDF가 핵심 엔진으로 사용되었다.6 이 응용은 VDB-GPDF의 효율적인 온라인 매핑 능력과 정확한 거리 정보 제공 능력이 결합되었기에 가능했다.21
- **동적 객체 재구성 (Dynamic Object Reconstruction)**: DynORecon과 같은 최신 시스템은 VDB-GPDF 프레임워크를 기반으로, 동적 SLAM 시스템이 추정한 움직이는 객체의 정보를 받아 해당 객체의 ESDF를 직접 추정한다.6 이를 통해 정적 배경과 동적 객체가 혼재된 복잡한 환경에서도 탐색에 필요한 정확한 맵을 생성할 수 있다.
- **안전한 로봇 제어 및 경로 계획**: GP가 제공하는 연속적이고 모든 지점에서 미분 가능한 거리 필드는 CHOMP와 같은 최적화 기반 경로 계획 알고리즘에 이상적인 입력이 된다.10 또한, 각 거리 추정치에 함께 제공되는 불확실성 정보는 로봇이 위험 지역을 회피하고 안전한 경로를 선택하도록 하는 확률적 계획 기법에 활용될 수 있다.26

### 4.2  기술적 한계 및 당면 과제

VDB-GPDF는 많은 발전을 이루었지만, 여전히 몇 가지 기술적 한계를 가지고 있다.

- **잔존하는 계산 비용**: L-GPDF를 통해 GP 추론을 지역화했지만, GP 자체의 계산 비용은 여전히 무시할 수 없는 부담으로 남아있다. 이로 인해 실시간 성능을 유지하기 위해 맵 업데이트 빈도를 조절해야 하는 등의 상충 관계가 발생한다.3
- **하이퍼파라미터 의존성**: GP 커널 함수의 성능은 길이 스케일(l)과 같은 하이퍼파라미터에 민감하게 반응한다. 이러한 파라미터를 다양한 환경에 맞게 자동으로 최적화하는 것은 여전히 도전적인 연구 주제다.
- **정적 세계 가정**: 현재 프레임워크는 기본적으로 정적인 환경을 가정한다. 오래된 데이터를 새로운 데이터로 덮어쓰는 방식으로 동적 객체를 암묵적으로 처리할 수는 있지만 1, 객체의 움직임을 명시적으로 모델링하거나 미래의 위치를 예측하지는 못한다.

### 4.3  향후 연구 방향

VDB-GPDF는 향후 여러 방향으로 확장될 수 있는 풍부한 잠재력을 가지고 있다.

- **SLAM/Odometry와의 강결합 (Tightly-coupled Integration)**: 현재는 외부의 SLAM 시스템으로부터 로봇의 위치 추정값을 입력받는 약결합(loosely-coupled) 방식이다. GPDF가 제공하는 정확한 거리 및 표면 정보를 SLAM의 최적화 과정에 직접 피드백하는 강결합 구조를 개발한다면, 위치 추정의 정확도와 맵의 일관성을 동시에 향상시키는 시너지를 창출할 수 있다.27
- **시공간 모델링 (Spatio-temporal Modeling)**: 현재의 공간 정보만을 다루는 GPDF를 시간 축까지 확장하여, 환경의 동적 변화를 모델링하는 **가우시안 프로세스 동적 모델(Gaussian Process Dynamical Model, GPDM)**로 발전시킬 수 있다.28 이는 시간에 따라 변화하는 환경을 예측하고, 움직이는 장애물에 대해 선제적으로 대응하는 등 한 차원 높은 수준의 로봇 지능을 구현하는 데 기여할 것이다.27
- **최신 렌더링 기술과의 융합**: VDB-GPDF의 정확한 기하학적 표현을 **3D 가우시안 스플래팅(3D Gaussian Splatting)**과 같은 최신 실시간 렌더링 기술과 결합하는 연구가 가능하다.31 이를 통해 사실적인 외형(appearance)과 정확한 물리적 구조(geometry)를 동시에 갖춘 고품질 디지털 트윈(Digital Twin)을 생성하고, 시뮬레이션 및 가상현실 응용 분야를 개척할 수 있다.27

이러한 확장 가능성은 VDB-GPDF가 단순히 더 나은 '맵'을 만드는 기술을 넘어, 로봇의 다양한 지능적 작업을 지원하는 '통합 표현(Unified Representation)'으로서의 잠재력을 가지고 있음을 시사한다.26 과거에는 매핑, 측위, 경로 계획, 조작 등 각 작업마다 별도의 특화된 데이터 표현이 필요했다.27 그러나 VDB-GPDF가 제공하는 [거리, 그래디언트, 표면 속성, 불확실성]이라는 풍부한 정보 집합은 이 모든 작업을 단일 표현 위에서 수행할 수 있는 강력한 이론적 기반을 제공한다. 예를 들어, 경로 계획기는 그래디언트를 따라 최적의 경로를 탐색하고, 충돌 감지기는 거리 값을 확인하며, 탐험 알고리즘은 불확실성(분산)이 높은 미지의 영역으로 로봇을 유도할 수 있다. 이 모든 정보가 단일 GPDF 모델에서 파생된다. 이는 로봇 소프트웨어 아키텍처의 복잡성을 획기적으로 줄이고, 각 기능 모듈 간의 유기적인 시너지를 극대화하여 보다 고차원적인 로봇 지능을 구현하는 중요한 초석이 될 수 있다.

## 5.  결론

### 6.1. VDB-GPDF의 핵심 기여 요약

VDB-GPDF는 가우시안 프로세스 거리 필드(GPDF)가 지닌 확률적 정확성과 연속성의 장점을 OpenVDB 데이터 구조의 탁월한 계산 효율성 및 확장성과 성공적으로 결합한 혁신적인 3D 매핑 프레임워크다. 이를 통해 기존 기술들이 직면했던 정확성, 효율성, 확장성 간의 고질적인 상충 관계를 효과적으로 완화하며, 온라인 증분 매핑 분야에서 새로운 가능성을 제시했다.

### 5.1  하이브리드 프레임워크로서의 의의

본 프레임워크는 단일 기술의 한계를 명확히 인식하고, 이종 기술의 장점을 융합하여 더 나은 해결책을 모색하는 현대 로보틱스 연구의 성공적인 사례로 평가될 수 있다. 특히, 실시간 처리를 위한 L-GPDF와 전역적 고품질 표현을 위한 G-GPDF의 이중 GP 구조는 계산적 제약 하에서 이론적 우아함과 실용적 성능을 모두 달성하기 위한 지능적인 설계의 정수를 보여준다.

### 5.2  미래 전망

VDB-GPDF는 정적인 3D 환경을 재구성하는 것을 넘어, 동적 환경 이해, 인간-로봇 상호작용, 고차원적 계획 등 미래 로보틱스 기술의 핵심 기반이 될 '통합 표현'으로서의 무한한 잠재력을 내포하고 있다. 향후 SLAM과의 강결합, 시공간 모델링으로의 확장, 그리고 최신 렌더링 기술과의 융합을 통해 그 기술적 영향력은 더욱 증대될 것으로 전망된다. 이는 궁극적으로 로봇이 복잡하고 불확실한 현실 세계와 더욱 정교하고 지능적으로 상호작용할 수 있도록 하는 데 결정적인 기여를 할 것이다.

#### 참고자료

1. (PDF) VDB-GPDF: Online Gaussian Process Distance Field with VDB Structure - ResearchGate, accessed August 6, 2025, https://www.researchgate.net/publication/382271448_VDB-GPDF_Online_Gaussian_Process_Distance_Field_with_VDB_Structure
2. VDB-GPDF: Online Gaussian Process Distance Field with VDB Structure - arXiv, accessed August 6, 2025, https://arxiv.org/html/2407.09649v1
3. VDB-GPDF: Online Gaussian Process Distance Field with VDB Structure - arXiv, accessed August 6, 2025, https://arxiv.org/html/2407.09649v3
4. VDB-GPDF: Online Gaussian Process Distance Field with VDB Structure - arXiv, accessed August 6, 2025, https://arxiv.org/html/2407.09649v2
5. [2407.09649] VDB-GPDF: Online Gaussian Process Distance Field with VDB Structure, accessed August 6, 2025, https://arxiv.org/abs/2407.09649
6. VDB-GPDF: Online Gaussian Process Distance Field with VDB Structure | Request PDF, accessed August 6, 2025, https://www.researchgate.net/publication/386122570_VDB-GPDF_Online_Gaussian_Process_Distance_Field_with_VDB_Structure
7. Gaussian process - Wikipedia, accessed August 6, 2025, https://en.wikipedia.org/wiki/Gaussian_process
8. Gaussian Processes for Dummies /, accessed August 6, 2025, https://katbailey.github.io/post/gaussian-processes-for-dummies/
9. [2302.13005] Accurate Gaussian Process Distance Fields with applications to Echolocation and Mapping - arXiv, accessed August 6, 2025, https://arxiv.org/abs/2302.13005
10. Gaussian Process Motion Planning, accessed August 6, 2025, https://homes.cs.washington.edu/~bboots/files/GPMP.pdf
11. Accurate Gaussian-Process-based Distance Fields with applications to Echolocation and Mapping - arXiv, accessed August 6, 2025, https://arxiv.org/html/2302.13005v3
12. Gaussian Process Implicit Surfaces for Shape Estimation and Grasping - Learning and Intelligent Systems @ TU Berlin, accessed August 6, 2025, https://argmin.lis.tu-berlin.de/papers/11-dragiev-ICRA.pdf
13. Probabilistic Implicit Surfaces for Localisation, Mapping and Planning - OPUS at UTS, accessed August 6, 2025, https://opus.lib.uts.edu.au/bitstream/10453/172297/1/thesis.pdf
14. OpenVDB - OpenVDB, accessed August 6, 2025, https://www.openvdb.org/documentation/doxygen/
15. OpenVDB - Ken Museth, accessed August 6, 2025, https://ken.museth.org/OpenVDB.html
16. What is OpenVDB and why should you care? [thinkingParticles Documentation], accessed August 6, 2025, https://cebas.com/manual/LifeLicenser/doku.php?id=reference_guide:thinkingparticles_nodes:operator_nodes:openvdb:what_is_openvdb
17. Frequently Asked Questions - OpenVDB, accessed August 6, 2025, https://www.openvdb.org/documentation/doxygen/faq.html
18. About OpenVDB, accessed August 6, 2025, https://www.openvdb.org/about/
19. Insight: VDB, a deep dive - JangaFX, accessed August 6, 2025, https://jangafx.com/insights/vdb-a-deep-dive
20. www.openvdb.org, accessed August 6, 2025, [https://www.openvdb.org/documentation/doxygen/faq.html#:~:text=OpenVDB%20stores%20voxel%20data%20in,all%20levels%20of%20the%20tree.](https://www.openvdb.org/documentation/doxygen/faq.html#:~:text=OpenVDB stores voxel data in,all levels of the tree.)
21. A Scene Representation for Online Spatial Sonification - arXiv, accessed August 6, 2025, https://arxiv.org/html/2412.05486v1
22. UTS-RI/VDB_GPDF - GitHub, accessed August 6, 2025, https://github.com/UTS-RI/VDB_GPDF
23. Comparison of ground truth and estimated EDF with a 2D horizontal slice... - ResearchGate, accessed August 6, 2025, https://www.researchgate.net/figure/Comparison-of-ground-truth-and-estimated-EDF-with-a-2D-horizontal-slice-09-m-above-the_fig3_382271448
24. A Scene Representation for Online Spatial Sonification - Bohrium, accessed August 6, 2025, https://www.bohrium.com/paper-details/a-scene-representation-for-online-spatial-sonification/1073851062688940033-108625
25. [2410.17831] Gaussian Process Distance Fields Obstacle and Ground Constraints for Safe Navigation - arXiv, accessed August 6, 2025, https://arxiv.org/abs/2410.17831
26. Lan Wu - CatalyzeX, accessed August 6, 2025, [https://www.catalyzex.com/author/Lan%20Wu](https://www.catalyzex.com/author/Lan Wu)
27. Exploring Probabilistic Distance Fields in Robotics - arXiv, accessed August 6, 2025, https://arxiv.org/html/2405.18965v1
28. Gaussian Process Dynamical Models - Gregory Gundersen, accessed August 6, 2025, https://gregorygundersen.com/blog/2020/07/24/gpdm/
29. Revisiting Gaussian Process Dynamical Models - IJCAI, accessed August 6, 2025, https://www.ijcai.org/Proceedings/15/Papers/152.pdf
30. Gaussian Process Dynamical Models, accessed August 6, 2025, https://www.dgp.toronto.edu/~jmwang/gpdm/nips05final.pdf
31. 3D Gaussian Splatting: Hands-on Course for Beginners - 3D Geodata Academy, accessed August 6, 2025, https://learngeodata.eu/3d-gaussian-splatting-hands-on-course-for-beginners/
32. Gaussian methods for 3D Reconstruction | by Carlo C. | AI monks.io - Medium, accessed August 6, 2025, https://medium.com/aimonks/gaussian-methods-for-3d-reconstruction-a5cff3c851fb