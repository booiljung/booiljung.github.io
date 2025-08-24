# 3D 가우시안 스플래팅(3D Gaussian Splatting, 3DGS)
[Gaussian Splatting](./index.md)



컴퓨터 그래픽스 분야의 오랜 숙원은 3차원(3D) 모델로부터 현실 세계를 그대로 복제한 듯한 2차원(2D) 이미지를 생성하는 렌더링(Rendering) 기술의 완성에 있었습니다.1 이 목표를 달성하기 위해 전통적으로는 폴리곤 메시(Polygon Mesh)나 포인트 클라우드(Point Cloud)와 같은 명시적인(explicit) 3D 장면 표현 방식이 주를 이루었습니다. 그러나 이러한 방식들은 복잡한 기하학적 구조나 머리카락과 같은 미세한 디테일을 표현하는 데 본질적인 한계를 드러냈습니다. 메시는 복잡한 표면을 단순화하는 과정에서 구멍이나 균열과 같은 아티팩트(artifact)를 유발할 수 있으며, 포인트 클라우드는 불연속적이고 희소한 구조로 인해 복잡한 형태를 정확하게 재구성하기 어려웠습니다.1

이러한 상황 속에서, 2020년 등장한 Neural Radiance Fields(NeRF)는 새로운 시점 합성(Novel View Synthesis) 분야에 혁명을 일으켰습니다.3 NeRF는 3D 공간상의 위치 좌표($x,y,z$)와 2D 시선 방향($\theta, \phi$)이라는 5차원(5D) 입력을 받아 해당 지점의 색상(RGB)과 볼륨 밀도($\sigma$, volume density)를 출력하는 심층 신경망(MLP, Multi-Layer Perceptron)을 사용합니다.5 이 연속적인(continuous) 함수 표현을 통해 NeRF는 기존 방식으로는 불가능했던 수준의 극사실적인 렌더링 품질을 달성하며 3D 재구성 기술의 새로운 기준을 제시했습니다.


NeRF는 전례 없는 시각적 품질(visual fidelity)을 선보이며 학계와 산업계의 폭발적인 관심을 받았지만, 실용화를 가로막는 두 가지 핵심적인 한계를 명확히 드러냈습니다. 첫 번째는 극심한 연산 비용으로 인한 느린 학습 및 렌더링 속도였고, 두 번째는 이로 인해 실시간 상호작용이 거의 불가능하다는 점이었습니다.3

NeRF의 렌더링 과정은 각 픽셀에 대해 광선(ray)을 투사하고, 그 경로를 따라 수많은 지점을 샘플링한 후, 각 샘플링 지점마다 무거운 MLP 연산을 반복적으로 수행해야 합니다.7 이 볼륨 레이 마칭(volume ray marching) 방식은 본질적으로 높은 계산 비용을 수반합니다. Mip-NeRF360과 같은 후속 연구들은 렌더링 품질을 더욱 향상시켜 최고 수준에 도달했지만, 단일 장면을 학습하는 데 최대 48시간이 소요될 정도로 비효율적이었습니다.4 반대로 Instant-NGP와 같은 모델들은 해시 그리드(hash grid)와 같은 기법을 도입하여 학습 시간을 수 분 단위로 단축했지만, 이는 필연적으로 품질 저하를 동반하는 트레이드오프(trade-off) 관계에 있었습니다.4 결과적으로, 1080p 해상도에서 초당 30프레임(FPS) 이상의 실시간 렌더링은 여전히 달성하기 어려운 목표로 남아있었습니다.3


2023년 SIGGRAPH 학회에서 발표된 '3D Gaussian Splatting for Real-Time Radiance Field Rendering' 논문은 이러한 NeRF의 한계를 정면으로 돌파하며 새로운 패러다임을 제시했습니다.3 3D 가우시안 스플래팅(이하 3D-GS)은 Mip-NeRF360과 동등하거나 그 이상의 시각적 품질을 달성하면서도, 1080p 해상도에서 초당 100프레임 이상의 압도적인 실시간 렌더링 속도와 경쟁력 있는 학습 시간을 동시에 구현했습니다.1

3D-GS의 핵심 혁신은 NeRF의 근간이었던 신경망을 과감히 배제하고, 장면을 표현하는 기본 단위를 '미분 가능한 3D 가우시안(differentiable 3D Gaussians)'이라는 명시적 프리미티브(explicit primitive)로 대체한 것에 있습니다.1 그리고 이 가우시안들을 2D 이미지로 렌더링하는 전 과정을 GPU 아키텍처에 고도로 최적화된 미분 가능한 래스터화(differentiable rasterization) 파이프라인으로 설계했습니다. 이는 NeRF가 추구했던 '모든 것을 학습하는' 암시적 표현(implicit representation)에서, GPU의 병렬 처리 능력을 극대화하는 명시적 표현으로의 회귀이자 동시에 진화였습니다.

이러한 접근 방식의 전환은 단순히 NeRF의 속도 문제를 해결한 것을 넘어, 3D 장면 표현 방식에 대한 근본적인 철학의 변화를 의미합니다. NeRF의 핵심이 장면 전체를 하나의 거대한 신경망 가중치에 '인코딩'하는 것이었다면, 3D-GS는 GPU가 가장 잘하는 작업인 '프리미티브 그리기(래스터화)'에 집중했습니다.15 3D-GS 연구진은 "만약 렌더링할 프리미티브 자체를 미분 가능하게 만들 수 있다면, 신경망 없이도 경사 하강법(gradient descent)을 통해 장면을 최적화할 수 있지 않을까?"라는 질문에 대한 해답을 제시한 것입니다. 그 해답이 바로 위치, 모양, 색, 투명도를 파라미터로 갖는 미분 가능한 3D 가우시안이었고 16, 이를 2D로 투영(splatting)하는 과정 전체를 미분 가능하게 설계하여 학습 기반 최적화를 가능하게 했습니다.13 결과적으로, NeRF의 '느린 추론(per-ray sampling)'을 GPU에 최적화된 '빠른 그리기(rasterization)'로 대체하면서도 학습의 유연성을 유지한 것이 3D-GS의 성공 비결입니다. 이는 기술적 진보가 항상 복잡성 증가를 동반하는 것이 아니라, 기존 하드웨어의 강점을 재해석하고 핵심 원리에 집중함으로써 이루어질 수 있음을 보여주는 중요한 사례입니다.


3D-GS의 혁신은 세 가지 핵심 요소의 유기적인 결합으로 이루어집니다: 3D 가우시안을 이용한 장면 표현, 고속의 미분 가능한 래스터화 파이프라인, 그리고 학습 과정에서 가우시안의 밀도를 동적으로 조절하는 최적화 기법입니다.3


3D-GS는 3D 장면을 수백만 개의 3D 가우시안(3D Gaussians) 집합으로 표현합니다.12 이는 각 점이 위치와 색상 정보만 갖는 전통적인 포인트 클라우드와 달리, 각 '점'이 하나의 가우시안 분포를 나타내는 풍부한 파라미터를 가집니다.6 '스플랫(Splat)'이라는 용어는 이러한 3D 가우시안을 2D 이미지 평면에 물감을 흩뿌리듯 투영하는 과정을 의미합니다.2

각 3D 가우시안은 다음과 같은 핵심 파라미터로 정의됩니다 13:

- **위치 (Position/Mean, $\mu$):** 3D 공간상에서 가우시안의 중심점을 나타내는 3차원 벡터($XYZ$)입니다.
- **공분산 (Covariance, $\Sigma$):** 3×3 크기의 행렬로, 가우시안의 모양, 즉 크기(scale)와 방향(orientation)을 결정합니다. 이 공분산 행렬을 통해 가우시안은 완벽한 구(등방성, isotropic) 형태뿐만 아니라, 특정 방향으로 길게 늘어지거나 납작한 타원체(이방성, anisotropic) 형태를 가질 수 있습니다. 이 이방성 표현 능력은 길고 얇은 표면이나 평면과 같은 기하학적 구조를 적은 수의 가우시안으로 효율적으로 모델링하는 데 매우 중요합니다.4 최적화 과정에서 공분산 행렬 $\Sigma$가 항상 양의 준정부호(positive semi-definite)라는 수학적 제약을 만족하도록 하기 위해, 행렬 자체를 직접 학습하지 않습니다. 대신, 3차원 스케일링 벡터($S$)와 4차원 회전 쿼터니언($R$)으로 분해하여 이들을 독립적으로 최적화한 후, $\Sigma=RSS^TR^T$ 와 같이 재구성하는 방식을 사용합니다. 이는 학습의 안정성을 크게 높여줍니다.15
- **불투명도 (Opacity, $\alpha$):** 0(완전 투명)과 1(완전 불투명) 사이의 스칼라 값으로, 해당 가우시안이 빛을 얼마나 가리는지를 결정합니다. 이 값은 여러 가우시안이 겹쳤을 때 최종 색상을 혼합하는 알파 블렌딩(alpha blending) 과정에서 핵심적인 역할을 합니다.15
- **색상 (Color):** 가우시안의 색상은 구면 조화 함수(Spherical Harmonics, SH)의 계수로 표현됩니다.13 단순한 RGB 값 대신 SH를 사용함으로써, 바라보는 시선 방향에 따라 색상이 미묘하게 변하는 효과(view-dependent effects)를 모델링할 수 있습니다. 이는 금속의 반사광이나 물체의 하이라이트와 같은 복잡한 광학 현상을 사실적으로 표현하는 데 기여합니다.12


미분 가능한 래스터화 파이프라인은 3D-GS 기술의 심장부라 할 수 있습니다. 이 파이프라인은 3D 공간에 정의된 수백만 개의 가우시안 집합을 입력받아 최종 2D 이미지를 생성하는 전 과정을 담당하며, 가장 중요한 특징은 이 모든 과정이 미분 가능하다(differentiable)는 점입니다.13 덕분에 렌더링된 결과 이미지와 실제 촬영된 원본 이미지(ground truth) 사이의 오차(loss)를 계산하고, 이 오차를 최소화하기 위한 그래디언트(gradient)를 모든 가우시안 파라미터에 대해 역전파(backpropagate)하여 모델을 최적화할 수 있습니다.21

이 파이프라인은 다음과 같은 주요 단계로 구성됩니다 7:

1. **투영 (Projection):** 먼저, 3D 공간에 있는 모든 가우시안을 주어진 카메라의 시점(viewpoint)에서 바라본 2D 이미지 평면으로 투영합니다. 이 과정에서 3D 가우시안의 중심점 $\mu$는 간단한 변환을 통해 2D 좌표로 매핑됩니다. 더 복잡한 부분은 3D 공분산 행렬 $\Sigma$를 2D로 투영하는 것입니다. 이는 뷰 변환 행렬($W$)과 원근 투영의 자코비안 행렬($J$)을 사용하여 $\Sigma′=JW\Sigma W^TJ^T$ 라는 근사식을 통해 2D 공분산 행렬 $\Sigma'$를 계산함으로써 이루어집니다. 이렇게 얻어진 $\Sigma'$는 2D 이미지 평면상에서의 타원(splat)의 모양을 결정합니다.7

2. **타일 기반 래스터화 (Tile-based Rasterization):** 수백만 개의 스플랫을 픽셀 단위로 처리하는 것은 비효율적입니다. 이를 해결하기 위해, 렌더링할 화면을 16x16 픽셀과 같은 작은 타일(tile) 단위로 분할합니다.6 그 다음, 각 스플랫이 어떤 타일들과 겹치는지 빠르게 식별하고, 각 타일별로 렌더링에 영향을 미치는 스플랫들의 목록을 생성합니다. 하나의 스플랫이 여러 타일에 걸쳐 있을 경우, 해당 스플랫은 여러 타일 목록에 중복으로 포함됩니다.7 이 방식은 GPU의 수많은 코어를 활용하여 각 타일을 독립적으로 병렬 처리할 수 있게 해주어 래스터화 과정의 효율을 극대화합니다.13

3. **정렬 및 알파 블렌딩 (Sorting & Alpha Blending):** 각 타일 내부에서는 해당 타일에 할당된 스플랫들을 카메라로부터의 거리, 즉 깊이(depth)를 기준으로 정렬합니다.13 정렬은 일반적으로 빠른 정렬 알고리즘인 기수 정렬(radix sort)을 사용합니다. 정렬이 완료되면, 각 타일의 픽셀들을 순회하며 스플랫들을 앞에서 뒤로(front-to-back) 순서대로 렌더링합니다. 각 픽셀의 최종 색상은 다음의 알파 블렌딩 공식을 통해 누적 계산됩니다:
   $$
   C_\text{out}=C_\text{new}\alpha_\text{new}+C_\text{prev}(1−\alpha_\text{new})
   $$
   이 과정을 통해 반투명한 객체들이 자연스럽게 혼합된 결과를 얻을 수 있습니다.6

4. **경사 하강 흐름 (Gradient Flow):** 학습을 위한 역전파 과정에서는 이와 반대 방향으로 연산이 진행됩니다. 최종 픽셀 색상에 대한 손실로부터 시작하여, 뒤에서 앞으로(back-to-front) 순서로 각 스플랫이 최종 색상에 기여한 정도를 계산하고, 이를 통해 각 스플랫의 2D 파라미터(색상, 불투명도, 2D 공분산)에 대한 그래디언트를 구합니다. 마지막으로, 투영 과정의 역연산(chain rule)을 통해 이 2D 그래디언트를 다시 3D 가우시안의 원본 파라미터(SH 계수, 불투명도, 스케일, 회전)에 대한 그래디언트로 변환합니다.15 이 모든 복잡한 연산은 고성능 컴퓨팅을 위해 특별히 제작된 CUDA 커널을 통해 구현되어, 전체 학습 및 렌더링 파이프라인의 빠른 속도를 보장합니다.28


3D-GS는 단순히 고정된 수의 가우시안을 최적화하는 데 그치지 않고, 학습 과정에서 가우시안의 밀도를 동적으로 조절하는 '적응형 밀도 제어(Adaptive Density Control)'라는 독창적인 전략을 사용합니다. 이 전략은 모델이 스스로 장면의 복잡도를 파악하고 필요한 곳에 자원을 집중적으로 할당하게 만들어, 효율성과 최종 품질을 동시에 높이는 핵심적인 역할을 합니다.7

- **초기화 (Initialization):** 학습은 여러 장의 입력 이미지로부터 Structure from Motion(SfM) 알고리즘(예: COLMAP)을 통해 추출된 희소 포인트 클라우드(sparse point cloud)에서 시작됩니다. 이 포인트 클라우드의 각 점들은 3D 가우시안의 초기 위치(μ)로 사용됩니다.3
- **최적화 (Optimization):** 래스터화 파이프라인을 통해 생성된 이미지와 실제 원본 이미지 간의 차이를 측정하기 위해 L1 손실과 D-SSIM(differentiable Structural Similarity Index Measure) 손실을 결합한 손실 함수(loss function)를 사용합니다.17 이 손실을 최소화하는 방향으로 확률적 경사 하강법(Stochastic Gradient Descent, SGD)과 Adam 옵티마이저를 사용하여 모든 가우시안 파라미터(위치, 공분산, 불투명도, SH 계수)를 반복적으로 업데이트합니다.21
- **적응형 밀도 제어 (Adaptive Density Control):** 이 과정은 3D-GS 최적화의 백미로, 특정 주기(예: 매 100회 반복)마다 수행됩니다.
  - **밀도화 (Densification):** 이 단계의 목표는 재구성이 부족한 영역에 새로운 가우시안을 추가하여 디테일을 향상시키는 것입니다. 재구성이 부족한 영역은 공통적으로 '뷰 공간 위치 그래디언트(view-space positional gradient)'가 크다는 특징을 보입니다.15 즉, "이 가우시안의 위치를 조금만 옮겨도 렌더링 결과가 크게 향상된다"는 신호는 해당 영역에 모델의 표현력이 부족하다는 의미입니다. 이 신호를 기반으로 두 가지 전략이 사용됩니다.
    - **복제 (Cloning):** 재구성이 덜 된 영역(under-reconstruction)에 있는 작은 가우시안의 경우, 해당 가우시안을 그대로 복제(clone)하고 위치 그래디언트 방향으로 약간 이동시켜 새로운 가우시안을 생성합니다. 이는 기하학적으로 비어있는 공간을 채우는 효과를 줍니다.13
    - **분할 (Splitting):** 하나의 가우시안이 너무 넓은 영역을 차지하여 디테일을 뭉개는 경우(over-reconstruction), 해당 가우시안을 두 개의 더 작은 가우시안으로 분할(split)합니다. 새로 생성된 가우시안들의 크기는 실험적으로 결정된 비율(약 1.6)로 나누어 작게 만들고, 위치는 원래 가우시안의 확률 밀도 함수(PDF)를 기반으로 샘플링하여 결정합니다.13
  - **가지치기 (Pruning):** 특정 주기(예: 매 3000회 반복)마다 불투명도(α)가 거의 0에 가까워 렌더링에 거의 기여하지 않는 가우시안들을 시스템에서 영구적으로 제거합니다. 이는 불필요한 연산을 줄이고 모델을 경량화하는 데 기여합니다.7

이러한 적응형 밀도 제어 전략은 고정된 구조를 갖는 NeRF의 MLP와 근본적으로 다른 접근 방식입니다. 이는 마치 조각가가 처음에는 큰 덩어리로 전체적인 형태를 잡고, 점차 디테일이 필요한 부분에 정과 망치를 집중하여 세밀하게 조각해 나가는 과정과 유사합니다. 필요한 곳에만 '정보량(가우시안)'을 동적으로 집중시키는 데이터 기반(data-driven) 방식 덕분에, 3D-GS는 매우 효율적으로 고품질의 장면 표현을 학습할 수 있습니다.


3D-GS는 NeRF가 해결하고자 했던 새로운 시점 합성 문제를 동일하게 다루지만, 그 접근 방식과 성능 특성에서 근본적인 차이를 보입니다. 두 기술의 차이점을 표현 방식, 렌더링 패러다임, 그리고 정량적 성능 지표를 통해 심층적으로 비교 분석할 수 있습니다.


두 기술의 가장 근본적인 차이는 3D 장면을 컴퓨터 내에서 어떻게 표현하는지에 있습니다.

- **3D-GS (명시적 표현, Explicit Representation):** 3D-GS는 장면을 수백만 개의 개별적인 3D 가우시안이라는 '명시적' 프리미티브의 집합으로 표현합니다.1 각 가우시안은 위치, 공분산, 불투명도, 색상 등 직접 접근하고 수정할 수 있는 파라미터를 가진 독립적인 객체입니다.31 이는 전통적인 컴퓨터 그래픽스의 포인트 클라우드나 폴리곤 메시처럼, 장면을 구성하는 요소들이 데이터 구조 안에 명확하게 정의되어 있는 '객체 기반' 표현 방식과 맥을 같이합니다.8
- **NeRF (암시적 표현, Implicit Representation):** 반면, NeRF는 장면 전체를 하나의 거대한 신경망(MLP)의 가중치(weights) 안에 '암시적으로' 인코딩합니다.5 장면의 기하학적 구조나 외형 정보는 네트워크 파라미터 속에 숨겨져 있으며, 특정 3D 좌표를 네트워크에 '질의(query)'해야만 비로소 해당 지점의 색상과 밀도 값을 얻을 수 있습니다.9 이는 장면을 직접적으로 기술하는 것이 아니라, 장면을 생성하는 '함수' 자체를 학습하는 방식입니다.


장면 표현 방식의 차이는 자연스럽게 렌더링 방식의 차이로 이어집니다.

- **3D-GS (스플래팅/래스터화, Splatting/Rasterization):** 3D-GS는 3D 공간에 존재하는 모든 가우시안을 2D 이미지 평면으로 투영(splatting)하고, 이를 깊이 순으로 정렬하여 앞에서부터 차례로 겹쳐 그리는 래스터화 방식을 사용합니다.6 이 과정은 GPU가 병렬 처리에 극도로 최적화된 작업으로, 모든 프리미티브를 한 번에 효율적으로 처리할 수 있습니다.7
- **NeRF (레이 마칭/볼륨 렌더링, Ray Marching/Volume Rendering):** NeRF는 이미지의 각 픽셀에 대해 카메라 원점으로부터 광선을 투사하고, 이 광선 경로를 따라 다수의 지점을 샘플링합니다. 그리고 각 샘플 지점마다 무거운 MLP를 실행하여 색상과 밀도를 예측한 후, 이 값들을 광선 방향으로 적분(integration)하여 최종 픽셀 색상을 계산합니다.6 이는 본질적으로 픽셀별 순차적 연산이며, 수많은 MLP 추론 과정으로 인해 계산 비용이 매우 높습니다.7


이러한 근본적인 차이는 학습 시간, 렌더링 속도, 시각적 품질, 그리고 메모리 사용량 등 정량적 성능 지표에서 뚜렷한 트레이드오프를 만들어냅니다.

- **학습 시간 (Training Time):** 3D-GS는 일반적으로 NeRF, 특히 Mip-NeRF360과 같은 고품질 모델보다 훨씬 빠르게 학습을 완료합니다. 일반적인 장면에서 3D-GS는 약 30분에서 1시간 내외로 학습이 가능한 반면, Mip-NeRF360은 최대 48시간이 소요될 수 있습니다.4 Instant-NGP와는 비슷한 학습 시간을 보이지만, 3D-GS는 훨씬 높은 최종 품질을 달성합니다.4
- **렌더링 속도 (Rendering Speed):** 3D-GS의 가장 압도적인 장점입니다. 고도로 최적화된 CUDA 기반 래스터화 파이프라인 덕분에, 1080p 해상도에서도 초당 100프레임(FPS) 이상의 실시간 렌더링이 가능합니다.3 이는 사용자가 3D 장면과 원활하게 상호작용할 수 있음을 의미합니다. 반면, 대부분의 NeRF 모델은 실시간 렌더링에 어려움을 겪으며, 고품질 모델의 경우 1 FPS 미만의 속도를 보이는 경우도 많습니다.4
- **시각적 품질 (Visual Quality):** 여러 벤치마크 데이터셋에서 3D-GS는 최고 수준의 NeRF 모델인 Mip-NeRF360과 동등하거나 이를 능가하는 시각적 품질(PSNR, SSIM, LPIPS 기준)을 보여주었습니다.1 NeRF에서 종종 발생하는 안개 같은 '부유물(floaters)'이나 거친 노이즈 아티팩트가 적은 경향이 있습니다. 다만, 3D-GS 역시 완벽하지는 않아서, 입력 이미지에서 관측되지 않은 영역이나 텍스처가 부족한 영역에서는 가우시안이 길게 늘어지거나 얼룩덜룩해 보이는(splotchy) 아티팩트가 발생할 수 있습니다.4
- **메모리 및 저장 공간 (Memory & Storage):** 이 지표는 3D-GS의 명확한 단점입니다. 수백만 개에 달하는 가우시안의 모든 파라미터(위치, 공분산, 불투명도, SH 계수)를 명시적으로 저장해야 하므로, 최종 모델의 파일 크기는 수백 MB에서 수 GB에 달합니다.17 반면, NeRF 모델은 상대적으로 작은 크기의 신경망 가중치만 저장하면 되므로, 수십 MB 수준으로 훨씬 작고 가볍습니다.19

이러한 복합적인 비교를 통해 3D-GS와 NeRF의 관계는 단순한 '대체'가 아닌, 명확한 '트레이드오프(trade-off)' 관계에 있음을 알 수 있습니다. 아래 표는 주요 NeRF 변종들과 3D-GS의 성능을 정량적으로 요약한 것입니다.

**Table 1: 3D-GS vs. NeRF 정량적 성능 비교**

| **항목 (Metric)**                 | **3D Gaussian Splatting (3D-GS)**     | **Mip-NeRF360 (품질 기준)**                | **Instant-NGP (속도 기준)**       | **자료 출처 (Source)** |
| --------------------------------- | ------------------------------------- | ------------------------------------------ | --------------------------------- | ---------------------- |
| **품질 (Quality)**                | **최상급 (SOTA)** (PSNR, SSIM, LPIPS) | **최상급 (SOTA)** (3D-GS와 유사/근소 열위) | 중간 품질 (3D-GS보다 낮음)        | 1                      |
| **학습 시간 (Training Time)**     | **~30-60분**                          | ~24-48시간                                 | **~5-10분** (가장 빠름)           | 4                      |
| **렌더링 속도 (Rendering Speed)** | **실시간 (≥100 FPS @ 1080p)**         | 매우 느림 (<1 FPS)                         | 준실시간 (10-15 FPS)              | 3                      |
| **메모리/파일 크기**              | **큼 (수백MB ~ 1.5GB+)**              | **작음 (수십MB)**                          | **매우 작음 (~5-35MB)**           | 17                     |
| **장면 표현 방식**                | 명시적 (Explicit Primitives)          | 암시적 (Implicit MLP)                      | 암시적 (Implicit Hash Grid + MLP) | 1                      |

이 비교는 기술 선택이 최종 애플리케이션의 요구사항에 따라 결정되어야 함을 명확히 보여줍니다. 3D-GS는 '모든 정보를 명시적으로 펼쳐놓기' 때문에 렌더링은 빠르지만 메모리를 많이 차지하고 36, NeRF는 '모든 정보를 신경망에 압축'하기 때문에 메모리는 적지만 렌더링 시 압축을 푸는 과정(추론)이 느립니다.19 따라서 사용자가 3D 장면을 웹에서 실시간으로 탐험하는 VR/AR이나 인터랙티브 뷰어와 같은 애플리케이션에는 초기 로딩 시간이 다소 길더라도 일단 로드된 후에는 압도적으로 빠른 렌더링 속도를 제공하는 3D-GS가 절대적으로 유리합니다.2 반면, 수많은 3D 에셋을 클라우드에 저장하고 필요할 때마다 선택적으로 전송해야 하는 서비스나, 모델 자체의 배포가 중요한 상황에서는 NeRF의 작은 파일 크기가 여전히 장점으로 작용할 수 있습니다. 이러한 트레이드오프는 자연스럽게 3D-GS 연구의 다음 방향성, 즉 '메모리 및 저장 공간 문제 해결'의 중요성을 부각시키며, 이는 다음 장에서 다룰 성능 최적화 및 압축 기술 연구의 당위성으로 이어집니다.


3D-GS가 발표된 이후, 연구 커뮤니티는 그 잠재력을 최대한 발휘하기 위해 두 가지 주요 방향으로 성능 개선 노력을 집중해왔습니다. 첫째는 렌더링 및 학습 속도를 더욱 가속화하는 것이고, 둘째는 가장 큰 약점으로 지적되는 메모리 및 저장 공간 문제를 해결하는 것입니다.


3D-GS의 핵심 장점인 속도를 극한까지 끌어올리기 위한 다양한 최적화 연구가 진행되고 있습니다.

- **래스터라이저 개선:** 원본 3D-GS에 포함된 타일 기반 소프트웨어 래스터라이저는 CUDA로 고도로 최적화되어 있지만, 여전히 개선의 여지가 있습니다. 최근 연구들은 GPU에 내장된 하드웨어 래스터화 파이프라인을 직접 활용하여 렌더링의 순방향 패스(forward-pass) 속도를 크게 향상시키는 가능성을 보여주었습니다.24 그러나 하드웨어 파이프라인은 그래디언트 계산과 같은 역방향 패스(backward-pass)에 제약이 많기 때문에, 이를 미분 가능하게 만들어 학습에 온전히 활용하는 '미분 가능한 하드웨어 래스터라이저' 개발이 중요한 연구 주제로 부상하고 있습니다.24
- **알고리즘 최적화:** 렌더링 알고리즘 자체를 개선하려는 시도도 활발합니다. 예를 들어, 'Speedy-Splat' 연구에서는 2D로 투영된 가우시안 타원의 경계 상자를 더욱 정밀하게 계산하는 'SnugBox' 기법을 제안했습니다.39 기존 방식이 타원의 최대 반경을 기준으로 넓은 사각형 영역을 할당했던 것과 달리, SnugBox는 타원에 꼭 맞는 경계 상자를 계산하여 불필요한 타일 할당을 대폭 줄였습니다. 이를 통해 렌더링 속도를 평균 6.7배 향상시키는 성과를 거두었습니다.39 또 다른 연구에서는 마할라노비스 거리(Mahalanobis distance)를 이용하여 픽셀 중심에서 멀리 떨어진, 즉 픽셀 색상에 미치는 영향이 미미한 가우시안에 대한 연산을 조기에 종료하는 최적화를 통해 렌더링 속도를 약 15% 향상시키기도 했습니다.40
- **분산 학습 (Distributed Training):** 단일 GPU의 VRAM 용량은 고해상도 이미지를 사용하거나 매우 복잡하고 규모가 큰 장면을 학습할 때 병목이 됩니다. 이를 극복하기 위해 'Grendel'과 같은 분산 학습 시스템이 개발되었습니다.41 Grendel은 수백만 개의 가우시안 파라미터를 여러 GPU에 효율적으로 분할하고, 각 GPU가 렌더링에 필요한 가우시안 데이터를 서로 주고받는 희소 통신(sparse all-to-all communication)과 동적 부하 분산(dynamic load balancing)을 통해 학습 과정을 병렬화합니다. 이를 통해 단일 GPU로는 불가능했던 대규모 장면의 학습을 가능하게 하고, 더 많은 가우시안을 사용하여 최종 렌더링 품질까지 향상시킬 수 있습니다.41


3D-GS의 기가바이트(GB) 단위에 달하는 거대한 파일 크기는 모바일 기기, VR/AR 헤드셋과 같이 리소스가 제한된 환경에 배포하는 데 가장 큰 실질적인 장벽입니다.37 이 문제를 해결하기 위한 연구는 크게 '압축(Compression)'과 '컴팩션(Compaction)'이라는 두 가지 상호 보완적인 방향으로 진행되고 있습니다.43

- **압축 (Compression):** 이 접근법은 가우시안의 총 개수는 그대로 유지하되, 각 가우시안을 구성하는 파라미터(위치, 스케일, 회전, 색상 등)를 저장하는 데 필요한 비트(bit) 수를 줄이는 데 초점을 맞춥니다.45
  - **스칼라/벡터 양자화 (Scalar/Vector Quantization):** 가장 간단하고 효과적인 방법은 32비트 부동소수점(Float32)으로 저장되는 파라미터들을 16비트 부동소수점(Float16)이나 더 낮은 정밀도의 데이터 타입으로 변환하는 것입니다. 이だけで 저장 공간을 절반으로 줄일 수 있으며, 대부분의 경우 시각적 품질 저하는 거의 없습니다.38 더 나아가, 유사한 속성(예: 색상)을 가진 가우시안들을 그룹화하고, 대표값들로 구성된 코드북(codebook)을 만든 뒤 각 가우시안은 코드북의 인덱스만 저장하는 벡터 양자화 기법을 사용하여 압축률을 극대화할 수 있습니다.
  - **엔트로피 코딩 (Entropy Coding):** 양자화된 데이터를 더욱 압축하기 위해 허프만 코딩이나 산술 코딩과 같은 엔트로피 코딩 기법을 적용하여 데이터의 통계적 중복성을 제거합니다.
- **컴팩션 (Compaction):** 이 접근법은 불필요하거나 중복되는 가우시안의 '수' 자체를 줄이는 것을 목표로 합니다.44
  - **가지치기 (Pruning):** 학습이 완료된 후, 렌더링 결과에 거의 영향을 미치지 않는 가우시안을 식별하여 제거하는 후처리 기법입니다. 예를 들어, 불투명도가 매우 낮아 거의 보이지 않는 가우시안, 크기가 너무 작아 픽셀에 영향을 주지 못하는 가우시안, 또는 다른 불투명한 가우시안에 의해 완전히 가려지는 가우시안 등이 제거 대상이 됩니다. 앞서 언급된 'Speedy-Splat' 연구는 학습 파이프라인에 효율적인 실시간 가지치기 기법을 통합하여, 최종 가우시안 수를 90%까지 줄이면서도 원본과 거의 동일한 품질을 유지하는 데 성공했습니다.39
- **구조적 압축 (Structured Compression):** 이는 압축과 컴팩션의 아이디어를 결합한 가장 진보된 접근법으로, 가우시안들 간의 공간적 상관관계를 활용하여 데이터를 구조적으로 재구성함으로써 압축 효율을 높입니다. 예를 들어, 'Scaffold-GS'와 같은 앵커 기반(anchor-based) 방법은 소수의 '앵커' 가우시안을 정의하고, 주변의 다른 가우시안들은 이 앵커로부터의 상대적인 변위와 속성 차이만을 저장하는 방식으로 데이터 중복성을 크게 줄입니다.45

이러한 최적화 기법들은 3D-GS의 실용성을 크게 향상시키고 있습니다. 특히 컴팩션 기법은 단순히 파일 크기를 줄이는 것을 넘어, 렌더링 시 처리해야 할 프리미티브의 총량을 줄여주기 때문에 렌더링 속도까지 함께 향상시키는 '일석이조'의 효과를 가져옵니다. 이는 컴팩션이 단순한 저장소 최적화를 넘어, 3D-GS의 핵심 장점인 실시간 성능을 더욱 강화하는 중요한 전략임을 시사합니다.

**Table 2: 3D-GS 압축 및 컴팩션 기법 비교**

| **기법 분류 (Category)**     | **세부 기법 (Technique)**  | **핵심 아이디어 (Core Idea)**                                | **결과 (Result)**                                  | **자료 출처 (Source)** |
| ---------------------------- | -------------------------- | ------------------------------------------------------------ | -------------------------------------------------- | ---------------------- |
| **압축 (Compression)**       | 양자화 (Quantization)      | 파라미터의 데이터 타입을 저정밀도로 변환 (e.g., Float32 -> Float16) | 2배 압축. 품질 저하 거의 없음.                     | 38                     |
|                              | 벡터 양자화, 엔트로피 코딩 | 유사 파라미터를 코드북으로 대체, 데이터 중복성 제거          | 5x-12x 압축. 수용 가능한 품질 저하.                | 38                     |
| **컴팩션 (Compaction)**      | 가지치기 (Pruning)         | 학습 중/후에 시각적 기여도가 낮은 가우시안을 제거            | 가우시안 수 90% 감소, 렌더링 속도 향상, 품질 유지. | 39                     |
| **구조적 압축 (Structured)** | 앵커/그래프 기반           | 가우시안의 공간적 관계를 활용하여 구조적으로 압축            | 높은 압축률 달성 목표.                             | 45                     |


3D-GS의 성공은 자연스럽게 다음 질문으로 이어졌습니다: "이 강력한 기술을 정적인 장면을 넘어 움직이는 동적 장면(dynamic scene)에 적용할 수 없을까?" 이 질문에 대한 해답으로 등장한 것이 바로 4D 가우시안 스플래팅(4D-GS)입니다.


정적 장면에 초점을 맞춘 3D-GS를 동적 장면으로 확장하는 것은 시간의 흐름($t$)에 따른 객체의 움직임(motion)과 모양 변화(deformation)를 모델링해야 하는 새로운 차원의 도전입니다.46

가장 단순한 접근법은 비디오의 각 프레임마다 독립적인 3D-GS 모델을 학습하는 것이지만, 이는 엄청난 계산 비용과 저장 공간 낭비를 초래하며 프레임 간의 시간적 일관성을 보장하지 못합니다.47

이를 해결하기 위해 4D-GS 연구들은 보다 근본적인 접근법을 채택했습니다. 먼저, 시간 t=0에서의 기준이 되는 하나의 '정규(canonical)' 3D 가우시안 집합을 정의합니다. 그리고 시간($t$)을 추가 입력으로 받아, 이 정규 가우시안들을 해당 시점의 위치와 모양으로 변형시키는 '변형 필드(deformation field)'를 학습하는 것입니다.47 이 변형 필드는 특정 시점에서 각 정규 가우시안이 겪게 될 위치 이동($\Delta x,\Delta y,\Delta z$), 회전($\Delta R$), 크기 변화($\Delta S$) 등을 예측하는 함수 역할을 합니다.

이 변형 필드는 일반적으로 경량의 신경망으로 구현됩니다. 예를 들어, '4D Gaussian Splatting for Real-Time Dynamic Scene Rendering' 논문에서는 HexPlane에서 영감을 받은 4D 시공간 복셀 그리드(spatio-temporal voxel grid)와 작은 MLP를 결합한 구조를 제안했습니다.47 이 구조에서 4D 인코더는 특정 가우시안의 정규 위치와 현재 시간($t$)을 입력받아 고차원의 특징 벡터(feature vector)를 추출하고, MLP 디코더는 이 특징 벡터를 바탕으로 해당 가우시안에 적용될 구체적인 변형 파라미터들을 예측합니다. 이 방식을 통해 단 하나의 정규 가우시안 모델과 작은 변형 필드 네트워크만으로 전체 동적 장면을 효율적으로 표현할 수 있게 됩니다.


초기 4D-GS 접근법은 동적 장면을 성공적으로 모델링했지만, 곧 새로운 비효율성이 발견되었습니다. 대부분의 실제 동영상 장면은 일부 움직이는 객체(사람, 자동차 등)와 넓은 정적 배경(건물, 풍경 등)이 혼재되어 있습니다. 모든 가우시안을 4D로 모델링하는 것은, 전혀 움직이지 않는 배경에 속한 가우시안들에게도 불필요한 시간적 변형 파라미터를 할당하고 매 프레임 변형을 계산하게 만들어 메모리와 연산을 낭비하는 결과를 초래합니다.51

이러한 문제 인식을 바탕으로, 장면을 동적 요소와 정적 요소로 적응적으로 분리하여 처리하는 '하이브리드 3D-4D 가우시안 스플래팅(Hybrid 3D-4DGS)' 프레임워크가 제안되었습니다.51 이 접근법의 핵심 아이디어와 과정은 다음과 같습니다:

1. **초기 4D 모델링:** 학습 초기에는 장면의 모든 가우시안을 4D로 간주하고 변형 필드를 통해 최적화를 시작합니다.
2. **정적/동적 분류:** 일정 반복 횟수(예: 500회)가 지나면, 각 4D 가우시안의 시간적 변화량을 분석합니다. 시간의 흐름에 따른 위치, 회전, 크기 등의 변화가 특정 임계값 이하로 거의 없는 가우시안들을 '정적(static)'으로 분류합니다.52
3. **표현 변환:** '정적'으로 분류된 가우시안들은 시간 차원과 관련된 모든 파라미터를 제거하고, 순수한 3D 가우시안으로 변환됩니다. 이들은 더 이상 변형 필드의 영향을 받지 않고 고정된 상태로 렌더링 파이프라인에 참여합니다. 반면, '동적(dynamic)'으로 분류된 가우시안들은 계속해서 완전한 4D 표현을 유지하며 변형 필드를 통해 움직임을 표현합니다.52
4. **분리된 최적화:** 이후의 학습 과정에서는 3D 가우시안 그룹과 4D 가우시안 그룹이 분리되어 최적화됩니다. 이를 통해 정적 배경에 대한 중복 계산을 원천적으로 제거할 수 있습니다.

이 하이브리드 방식은 정적 영역에 대한 불필요한 계산을 제거함으로써 전체 학습 시간을 기존 4D-GS 대비 3~5배 단축하고, 메모리 요구량을 크게 줄이면서도 최종 렌더링 품질은 유지하거나 오히려 향상시키는 놀라운 효율성을 보여주었습니다.51

4D-GS, 특히 하이브리드 3D-4DGS로의 발전 과정은 3D-GS 연구 커뮤니티의 빠른 학습 및 문제 해결 능력을 보여주는 대표적인 사례입니다. 이는 '일단 되게 만들고(4D-GS), 그 다음 최적화한다(Hybrid 3D-4DGS)'는 실용적인 공학적 접근 방식의 전형을 보여줍니다. 복잡한 문제를 한 번에 완벽하게 해결하려 하기보다, 점진적으로 문제를 분해하고 각 부분에 맞는 최적의 도구를 적용하는 연구 개발의 패턴을 통해 기술이 빠르게 성숙해나가고 있음을 알 수 있습니다.


3D-GS의 뛰어난 실시간 렌더링 성능과 높은 시각적 품질은 발표 직후부터 다양한 산업 분야의 주목을 받았으며, 가상현실(VR), 디지털 트윈, 영화 및 게임 제작 등에서 실질적인 적용 사례들이 등장하고 있습니다. 각 산업의 고유한 요구사항은 3D-GS 기술의 특정 측면을 발전시키는 원동력이 되고 있습니다.


VR/AR 환경은 사용자의 시점 변화에 즉각적으로 반응해야 하므로, 초당 72~90프레임(Hz) 이상의 매우 높은 프레임률과 낮은 지연 시간(latency)이 몰입감 유지를 위해 필수적입니다. 3D-GS의 실시간 렌더링 능력은 이러한 요구사항을 충족시키는 이상적인 기술로 평가받습니다.5

그러나 데스크톱 환경과 달리 VR 헤드셋(HMD)의 넓은 시야각(FOV), 고해상도 디스플레이, 그리고 사용자의 끊임없는 머리 움직임은 3D-GS에 새로운 도전 과제를 안겨주었습니다.55

- **VR에서의 도전 과제:**
  - **시간적 아티팩트 (Temporal Artifacts):** 사용자가 머리를 미세하게 움직일 때, 렌더링되는 가우시안의 순서가 바뀌면서 특정 부분이 갑자기 나타나거나 사라지는 '파핑(popping)' 현상이 발생합니다. 이는 VR 환경에서 극심한 멀미와 몰입감 저하를 유발합니다.55
  - **플로터 (Floaters):** 3D 가우시안을 2D로 투영하는 과정에서 발생하는 왜곡으로 인해, 배경과 분리된 것처럼 보이는 부유물(floaters)이 나타날 수 있습니다. 이는 특히 양안시(stereo vision)를 사용하는 VR에서 깊이감을 해치고 시각적 불편함을 초래합니다.55
  - **성능 저하:** VR의 고해상도 렌더링은 처리해야 할 가우시안의 수를 기하급수적으로 증가시켜, 실시간 성능의 기준선인 72 FPS를 유지하기 어렵게 만듭니다.54
- **해결책 (VRSplat):** 이러한 VR 환경의 문제들을 종합적으로 해결하기 위해 'VRSplat'과 같은 특화된 연구가 진행되었습니다.55 VRSplat은 여러 최신 기술들을 유기적으로 결합하여 VR 경험을 최적화합니다.
  - **아티팩트 제거:** 'Mini-Splatting', 'StopThePop', 'Optimal Projection'과 같은 기존 연구들을 VR 환경에 맞게 수정하고 결합하여 파핑과 플로터 현상을 효과적으로 억제합니다.
  - **포비티드 래스터라이저 (Foveated Rasterizer):** 인간의 시각 시스템이 시선이 머무는 중심 영역(fovea)만 선명하게 보고 주변부(peripheral)는 흐릿하게 인식한다는 특성을 활용한 기술입니다. VRSplat은 효율적인 포비티드 래스터라이저를 구현하여, 사용자의 시선이 향하는 작은 영역은 고품질로 렌더링하고 나머지 넓은 주변부 영역은 저품질로 렌더링합니다. 특히 이 두 영역의 렌더링을 단일 GPU 실행(single pass)으로 처리하여 연산 효율을 극대화하고, 이를 통해 72 FPS 이상의 안정적인 프레임률을 달성했습니다.54

현재 'Scaniverse', 'Hyperscape', 'Gracia'와 같은 애플리케이션들이 Meta Quest와 같은 상용 VR 기기에서 3D-GS 콘텐츠를 탐색하는 경험을 제공하며 기술의 가능성을 보여주고 있습니다.57


디지털 트윈(Digital Twin)은 공장, 건물, 도시와 같은 물리적 자산이나 시스템의 가상 복제품을 만들고, 실시간 데이터를 연동하여 상태를 모니터링하고 시뮬레이션을 통해 미래를 예측하는 기술입니다.59 3D-GS는 현실 세계를 빠르고 사실적으로 3D 모델화할 수 있는 능력 덕분에 디지털 트윈 구축을 위한 핵심 시각화 기술로 큰 주목을 받고 있습니다.60

- **사례 연구 분석 (Plain Concepts):** 소프트웨어 컨설팅 기업 Plain Concepts는 3D-GS를 이용한 디지털 트윈의 정확도를 검증하기 위해 심층적인 사례 연구를 수행했습니다.60
  - **테스트 환경:** 연구는 세 가지 다른 규모의 환경에서 진행되었습니다: (1) 중간 규모 사무실(100m²), (2) 대형 가구(4.7m), (3) 360° 카메라를 이용한 대규모 시설(5000m²).
  - **정확도 결과:** 제어된 환경에서 3D-GS는 평균 수 cm 수준의 높은 정확도를 보였습니다. 특히, AprilTag와 같은 인쇄된 마커를 함께 사용하여 스케일을 보정했을 때, 4.7m 가구의 길이를 거의 오차 없이(4.78m) 측정하는 등 놀라운 정밀도를 보여주었습니다.
  - **한계점 발견:** 이 연구는 3D-GS의 한계 또한 명확히 드러냈습니다.
    - **특징 없는 표면 (Featureless Surfaces):** 흰 벽, 단색 바닥 등 텍스처 특징이 부족한 평면에서는 SfM 과정에서 특징점을 정확히 매칭하기 어려워, 가우시안이 실제 표면에서 벗어나 붕 뜨거나 확산되는(diffuse) 아티팩트가 발생했습니다.
    - **반사 및 투명 표면 (Reflective/Transparent Surfaces):** 유리창이나 광택 있는 표면의 경우, 3D-GS는 반사된 상을 실제 존재하는 기하학적 구조로 오인하여 유리창 너머 허공에 가우시안을 생성하는 오류를 보였습니다.
    - **캡처 조건의 영향:** 대규모 야외 환경을 360° 카메라로 장시간(9분) 촬영했을 때, 촬영 동안 태양의 위치가 변하면서 생긴 그림자의 이동이 렌더링 결과에 불일치를 유발하는 등, 일관된 조명 조건의 중요성이 확인되었습니다.
- **산업 적용:** 이러한 가능성과 한계 속에서 실제 산업 적용이 이루어지고 있습니다. Hiverlab은 자사의 디지털 트윈 플랫폼 'SpatialWork'에 3D-GS 편집기를 통합하여, 여러 스플랫 데이터를 합치고 그 위에 실시간 IoT 센서 데이터를 오버레이하는 솔루션을 제공하고 있습니다.64 또한, 'PerfCam' 프레임워크는 공장 생산 라인에 설치된 기존 CCTV 카메라 영상만으로 3D-GS 디지털 트윈을 구축하고, 여기에 딥러닝 객체 탐지 모델을 결합하여 생산 라인의 가동률, 성능 등 핵심 성과 지표(KPI)를 실시간으로 추출하는 시스템을 개발했습니다.61


영화 및 게임 산업에서 3D-GS는 가상 프로덕션(Virtual Production) 워크플로우를 혁신할 잠재력을 보여주고 있습니다.

- **워크플로우 통합:** 실제 촬영 장소나 세트를 핸드헬드 SLAM 스캐너 등으로 빠르게 스캔하여 3D-GS 에셋을 생성하고, 이를 Unreal Engine이나 Houdini와 같은 주요 디지털 콘텐츠 제작(DCC) 툴로 가져와 가상 로케이션 스카우팅, 사전 시각화(Pre-visualization), 디지털 배경 제작 등에 활용할 수 있습니다.66 이는 시간과 비용이 많이 드는 실제 로케이션 촬영을 줄이고, 기획 단계에서부터 최종 결과물을 시각적으로 확인하며 의사결정을 내릴 수 있게 돕습니다.
- **편집 및 제어의 중요성:** 3D-GS를 단순한 캡처 데이터를 넘어 창의적인 에셋으로 활용하기 위해서는 아티스트의 의도대로 편집하고 제어할 수 있는 도구가 필수적입니다. Houdini용 플러그인 'GSOPs'는 이러한 요구에 부응하는 강력한 기능을 제공합니다.69 아티스트는 GSOPs를 통해 3D-GS 데이터에서 특정 객체를 분리하거나 노이즈를 제거하고, 가우시안들을 직접 변형(deform)시켜 애니메이션을 적용하거나, 장면에 새로운 조명을 추가하는(relighting) 등 정교한 제어가 가능합니다.
- **장점 및 현재의 한계:** 3D-GS는 반사나 투명 재질을 포함한 복잡한 장면을 매우 사실적으로 빠르게 캡처하여 배경 에셋으로 활용하는 데 큰 장점을 가집니다.12 그러나 현재의 3D-GS는 본질적으로 폴리곤 메시(mesh)를 생성하지 않는 볼륨 표현 방식이기 때문에, 기존 게임 엔진의 물리 시뮬레이션, 충돌 감지(collision detection), 캐릭터 애니메이션 파이프라인과 직접적으로 호환되지 않는다는 한계가 있습니다.59 이 문제를 해결하기 위해 3D-GS로부터 고품질의 메시를 추출하는 연구(e.g., Gaussian Frosting, 2DGS)가 병행되고 있습니다.5

결론적으로, 각 산업 분야의 서로 다른 요구사항-VR의 '속도', 디지털 트윈의 '정확도', 미디어 제작의 '사실성 및 편집 가능성'-은 3D-GS 기술이 다각적으로 발전하도록 견인하는 역할을 하고 있습니다. 이는 3D-GS가 단일 기술에 머무르지 않고, 다양한 응용 분야와 상호작용하며 진화하는 하나의 거대한 기술 생태계를 형성하고 있음을 보여줍니다.


3D-GS 기술이 빠르게 확산되면서, 전문가부터 일반 사용자까지 누구나 이 기술을 활용할 수 있도록 돕는 다양한 저작(authoring) 및 편집 도구 생태계가 빠르게 형성되고 있습니다. 이 생태계는 '접근성'을 높이는 방향과 '전문성'을 강화하는 두 가지 방향으로 동시에 확장되고 있습니다.


복잡한 명령어나 설정 없이도 손쉽게 3D-GS 모델을 만들 수 있는 도구들이 등장하며 기술의 대중화를 이끌고 있습니다.

- **모바일 애플리케이션:** 최신 스마트폰에 탑재된 고성능 카메라와 LiDAR 센서를 활용하여 3D-GS 장면을 생성하는 앱들이 대표적입니다. 'Polycam', 'KIRI Engine', 'Scaniverse', 'Luma AI'와 같은 앱들은 사용자가 객체나 공간 주위를 돌며 비디오를 촬영하거나 여러 장의 사진을 찍기만 하면, 클라우드 서버나 기기 내(on-device) 연산을 통해 고품질의 3D-GS 모델(.ply 파일)을 생성해 줍니다.22 이는 3D 스캐닝에 대한 전문 지식이 없는 일반 사용자도 손쉽게 현실 공간을 디지털화할 수 있는 길을 열었습니다.
- **웹 기반 플랫폼:** 별도의 소프트웨어 설치 없이 웹 브라우저에서 모든 작업을 처리할 수 있는 플랫폼도 인기를 얻고 있습니다. 'Polycam Web'이나 'SuperSplat'과 같은 서비스는 사용자가 촬영한 이미지나 비디오 파일을 웹사이트에 업로드하면 서버에서 3D-GS 모델을 생성해주고, 결과를 웹 뷰어에서 바로 확인하고 공유할 수 있는 기능을 제공합니다.73
- **오픈소스 파이프라인:** 더 높은 수준의 제어와 커스터마이징을 원하는 개발자나 연구자들은 원본 3D-GS 논문에서 공개한 오픈소스 코드를 활용할 수 있습니다.11 이 파이프라인은 보통 (1) COLMAP과 같은 Structure from Motion(SfM) 도구를 사용하여 입력 이미지들로부터 카메라 포즈와 희소 포인트 클라우드를 추정하고, (2) 이 정보를 바탕으로 3D-GS 모델을 초기화한 뒤, (3) PyTorch 기반의 최적화 스크립트를 실행하여 모델을 학습시키는 전체 워크플로우를 포함합니다.


생성된 3D-GS 모델을 그대로 사용하는 경우는 드뭅니다. 원하는 결과물을 얻기 위해서는 불필요한 부분을 제거하거나 다른 에셋과 합성하는 등의 편집 과정이 필수적입니다.

- **범용 웹 편집기:** 'SuperSplat'은 3D-GS 편집을 위한 가장 대표적인 웹 기반 도구입니다.73 사용자는.ply 형식의 스플랫 파일을 브라우저로 드래그 앤 드롭하기만 하면 됩니다. 직관적인 인터페이스를 통해 스캔 과정에서 생긴 노이즈나 부유물(floaters)을 쉽게 제거하고, 원하는 부분만 잘라내는 크롭(crop) 기능을 사용할 수 있습니다. 또한, 모델을 압축하여 파일 크기를 줄이거나, 카메라 경로를 지정하여 간단한 애니메이션을 만들고, 편집된 결과를 새로운 파일로 내보내거나 웹 링크로 공유하는 등 다채로운 후처리 작업을 지원합니다.73
- **전문 DCC 플러그인:** 영화 및 게임 제작과 같은 전문 분야에서는 기존에 사용하던 디지털 콘텐츠 제작(DCC) 툴과의 통합이 매우 중요합니다. 이를 위해 주요 DCC 툴을 위한 3D-GS 플러그인들이 활발하게 개발되고 있습니다.
  - **Houdini (GSOPs):** 'GSOPs'는 SideFX Houdini를 위한 강력한 무료 플러그인으로, 3D-GS 데이터를 Houdini의 절차적(procedural) 워크플로우 안으로 완벽하게 통합합니다.69 아티스트는 가우시안의 위치, 색상, 크기 등의 속성을 다른 지오메트리 속성처럼 다룰 수 있으며, 이를 통해 모델을 변형시키거나 파티클 시스템과 연동하는 등 복잡하고 창의적인 애니메이션 및 시각 효과(VFX) 작업이 가능해집니다.
  - **Unreal Engine & Unity:** 게임 엔진을 위한 플러그인도 다수 개발되어, 3D-GS 에셋을 Unreal Engine이나 Unity 프로젝트로 직접 임포트하여 실시간으로 렌더링하고 게임 환경의 일부로 활용할 수 있게 되었습니다.46
- **인공지능 기반 편집 (AI-driven Editing):** 3D-GS 편집의 새로운 지평은 생성형 AI와의 결합에서 열리고 있습니다. 'Instruct-GS2GS'나 'GaussCtrl'과 같은 연구들은 텍스트 프롬프트(text prompt)를 사용하여 3D-GS 장면을 직관적으로 편집하는 방법을 제시합니다.77 이 기술들은 InstructPix2Pix와 같은 이미지 대 이미지(Image-to-Image) 변환 모델을 활용합니다. 사용자가 "벽을 파란색으로 만들어줘"와 같은 자연어 명령을 내리면, 시스템은 현재 3D-GS 모델에서 렌더링한 여러 시점의 이미지들을 이 명령에 따라 수정한 뒤, 수정된 이미지들을 새로운 학습 데이터로 삼아 원래의 3D-GS 모델을 다시 최적화하는 과정을 반복합니다. 이는 수백만 개의 가우시안을 직접 조작하는 대신, 인간의 언어라는 가장 직관적인 인터페이스를 통해 3D 장면을 편집할 수 있는 가능성을 보여줍니다.

이처럼 3D-GS 생태계는 일반 사용자를 위한 '접근성'과 전문가를 위한 '전문성'이라는 두 축을 따라 빠르게 성장하고 있습니다. 스마트폰 앱이 기술의 대중화를 이끄는 동시에, 전문 DCC 플러그인과 AI 기반 편집 기술은 3D-GS를 기존 콘텐츠 제작 파이프라인에 깊숙이 통합하고, 이전에는 상상하기 어려웠던 새로운 창작의 가능성을 열어가고 있습니다. 이러한 다각적인 생태계의 발전은 3D-GS가 일시적인 유행을 넘어, 다양한 사용자층을 아우르는 강력하고 실용적인 차세대 3D 기술로 자리매김하고 있음을 증명합니다.


3D 가우시안 스플래팅은 실시간 고품질 렌더링이라는 놀라운 성과를 거두었지만, 기술이 성숙해감에 따라 몇 가지 근본적인 한계점들이 명확해지고 있으며, 이는 곧 미래 연구의 방향성을 제시하고 있습니다. 3D-GS의 미래는 단순히 장면을 더 잘 '재구성(Reconstruction)'하는 것을 넘어, 재구성된 장면의 의미를 '이해(Understanding)'하고, 이를 바탕으로 새로운 것을 '생성(Generation)'하며, 사용자와 '상호작용(Interaction)'하는 방향으로 나아갈 것입니다.


현재 3D-GS 기술이 직면한 주요 도전 과제는 다음과 같습니다.

- **아티팩트 및 품질 문제:** 3D-GS는 입력 데이터에 크게 의존합니다. 따라서 촬영된 이미지의 수가 부족하거나(sparse views), 객체들이 서로 멀리 떨어져 있는 경우 품질이 저하됩니다. 특히, 텍스처가 없는 넓고 평평한 표면, 유리와 같은 반사체나 투명체는 SfM 단계에서의 특징점 매칭 실패와 렌더링 과정의 모호성으로 인해 가우시안이 길게 늘어지거나 얼룩지는 아티팩트, 또는 허공에 떠다니는 플로터(floaters)를 유발하는 고질적인 문제를 안고 있습니다.13
- **편집 및 제어의 어려움:** 3D-GS는 명시적 표현 방식임에도 불구하고, 수백만 개의 개별 가우시안을 의미론적으로(semantically) 편집하는 것은 여전히 매우 어려운 과제입니다.12 예를 들어, 장면에서 '의자'만을 선택하여 옮기거나 색을 바꾸고 싶을 때, 어떤 가우시안들이 '의자'에 해당하는지를 식별하는 과정이 직관적이지 않습니다. 현재의 편집 도구들은 대부분 기하학적 영역을 기반으로 한 수동 선택에 의존하고 있습니다.
- **기하학적 표현의 부재:** 3D-GS는 본질적으로 표면(surface)이 아닌 볼륨(volume)을 표현하는 점들의 집합입니다. 이로 인해 명확한 표면 메시(mesh)가 존재하지 않아, 기존 3D 파이프라인에서 당연하게 여겨졌던 작업들이 어렵습니다. 정확한 표면 법선(surface normal) 벡터를 추출하기 어렵고, 물리 기반 렌더링(PBR)에 필요한 재질 속성을 정의하기 힘들며, 게임 엔진에서의 충돌 감지나 물리 시뮬레이션 적용이 불가능합니다.5
- **대규모 장면 처리의 부담:** 압축 및 컴팩션 기술이 발전하고 있음에도 불구하고, VRAM 요구사항과 긴 학습 시간은 여전히 도시 전체와 같은 매우 큰 규모의 환경을 다루는 데 있어 주요 병목으로 작용합니다.17


이러한 한계점들을 극복하기 위한 연구들이 활발히 진행 중이며, 이는 3D-GS의 미래 발전 가능성을 엿보게 합니다.

- **정규화 및 사전 지식 도입 (Regularization and Priors):** 아티팩트를 줄이고, 관측되지 않은 영역을 보다 그럴듯하게 재구성하기 위해 추가적인 제약을 가하는 연구가 필요합니다. 예를 들어, 렌더링된 표면이 물리적으로 매끄럽도록 강제하는 기하학적 정규화(regularization) 항을 손실 함수에 추가하거나, 대규모 3D 데이터셋으로부터 학습된 '사전 지식(prior)'을 활용하여 장면의 구조를 예측하는 데이터 기반 접근법을 도입할 수 있습니다.13
- **생성 모델과의 결합 (Integration with Generative Models):** 3D-GS를 디코더로 사용하는 생성형 3D 모델의 등장은 중요한 전환점입니다.18 텍스트나 단일 이미지로부터 3D 장면을 직접 생성하는 모델들은 3D-GS를 통해 고품질의 실시간 렌더링이 가능한 결과물을 만들어낼 수 있습니다. 이는 3D 콘텐츠 제작의 패러다임을 '캡처'에서 '생성'으로 바꾸고, 더 나아가 텍스트 기반의 정교한 3D 편집을 가능하게 할 것입니다.82
- **실시간 재구성 (Real-time Reconstruction):** 현재의 3D-GS는 수십 분의 오프라인 학습 시간을 필요로 합니다. 하지만 로봇 원격 조작이나 실시간 디지털 트윈과 같은 애플리케이션을 위해서는 입력되는 비디오 스트림으로부터 즉각적으로 3D 장면을 재구성하는 기술이 필수적입니다. 이를 위해 RGB-D 카메라의 깊이 정보를 활용하거나, 사전 학습된 신경망을 통해 SfM과 최적화 과정을 단일 순방향 패스(single feed-forward pass)로 대체하려는 '실시간 3D-GS' 연구가 활발히 진행되고 있습니다.83
- **물리 시뮬레이션 및 상호작용 (Physics Simulation and Interaction):** 3D-GS의 기하학적 표현 부재 문제를 해결하기 위해, 3D-GS 표현으로부터 효율적으로 고품질의 메시를 추출하는 기술(e.g., SuGaR, Gaussian Frosting, 2DGS)이 발전하고 있습니다.5 더 나아가, 메시 변환 없이 3D-GS 표현 위에서 직접 물리적 상호작용을 시뮬레이션하는 연구가 성공한다면, 3D-GS는 단순한 시각화 도구를 넘어 동적이고 상호작용 가능한 차세대 가상 세계의 근간이 될 잠재력을 가지고 있습니다.

결론적으로, 3D 가우시안 스플래팅은 컴퓨터 그래픽스 분야에 실시간 사실주의 렌더링이라는 새로운 시대를 열었습니다. 현재의 기술은 주로 현실을 얼마나 잘 '복제'하는가에 초점이 맞춰져 있지만, 미래의 발전 방향은 명확합니다. 장면 파싱(scene parsing), 의미론적 분할(semantic segmentation)과 같은 컴퓨터 비전 기술과 결합하여 복제된 장면의 의미를 '이해'하고, 생성 모델링과 결합하여 새로운 장면을 '창조'하며, 물리 기반 렌더링 및 시뮬레이션 기술과 결합하여 사용자와 '상호작용'하는 것입니다. 이러한 융합적 발전이 이루어질 때, 3D-GS는 진정한 의미의 차세대 3D 플랫폼으로 자리매김할 것입니다.


1. 3D Gaussian Splatting for Real Time Radiance Field Rendering Using Insta360 Camera: - CORE, accessed July 19, 2025, https://core.ac.uk/download/617974924.pdf
2. 가우시안 스플래팅 (Gaussian Splatting): 3D 장면을 표현하는 새로운 패러다임, accessed July 19, 2025, https://nodiscard.tistory.com/230
3. 3D Gaussian Splatting for Real-Time Radiance Field Rendering - Inria, accessed July 19, 2025, https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/
4. 3D Gaussian Splatting for Real-Time Radiance Field Rendering - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/372989904_3D_Gaussian_Splatting_for_Real-Time_Radiance_Field_Rendering
5. Radiance Fields (Gaussian Splatting and NeRFs), accessed July 19, 2025, https://radiancefields.com/
6. 3D Gaussian Splatting - velog, accessed July 19, 2025, https://velog.io/@hsbc/3D-Gaussian-Splatting
7. [3DGS] 3D Gaussian Splatting for Real-Time Radiance Field ... - velog, accessed July 19, 2025, https://velog.io/@cjkangme/3D-Gaussian-Splattingfor-Real-Time-Radiance-Field-Rendering
8. 3D Gaussian Splatting 리뷰 (SIGGRAPH 2023) - velog, accessed July 19, 2025, [https://velog.io/@dusruddl2/3D-Gaussian-Splatting-%EA%B0%9C%EB%85%90-%EB%A6%AC%EB%B7%B0-SIGGRAPH-2023](https://velog.io/@dusruddl2/3D-Gaussian-Splatting-개념-리뷰-SIGGRAPH-2023)
9. [컴퓨터 그래픽스] 트렌드 연구 NeRF, 3D Gaussian Splatting, accessed July 19, 2025, [https://flyduckdev.tistory.com/entry/%EC%BB%B4%ED%93%A8%ED%84%B0-%EA%B7%B8%EB%9E%98%ED%94%BD%EC%8A%A4-%ED%8A%B8%EB%A0%8C%EB%93%9C-%EC%97%B0%EA%B5%AC-NeRF](https://flyduckdev.tistory.com/entry/컴퓨터-그래픽스-트렌드-연구-NeRF)
10. [논문리뷰] Gaussian Splatting with NeRF-based Color and Opacity - 전생했더니 인공지능이었던 건에 대하여, accessed July 19, 2025, [https://kimjy99.github.io/%EB%85%BC%EB%AC%B8%EB%A6%AC%EB%B7%B0/vdgs/](https://kimjy99.github.io/논문리뷰/vdgs/)
11. Original reference implementation of "3D Gaussian Splatting for Real-Time Radiance Field Rendering" - GitHub, accessed July 19, 2025, https://github.com/graphdeco-inria/gaussian-splatting
12. 3D Gaussian Splatting: A new frontier in rendering - Chaos, accessed July 19, 2025, https://www.chaos.com/blog/3d-gaussian-splatting-new-frontier-in-rendering
13. 3D Gaussian Splatting for Real-Time Radiance Field Rendering | Qiang Zhang, accessed July 19, 2025, https://zhangtemplar.github.io/3d-gaussian-splatting/
14. 3D Gaussian Splatting: Performant 3D Scene Reconstruction at Scale - AWS, accessed July 19, 2025, https://aws.amazon.com/blogs/spatial/3d-gaussian-splatting-performant-3d-scene-reconstruction-at-scale/
15. [논문 리뷰] 3D Gaussian Splatting (SIGGRAPH 2023) : 랜더링 속도/퀄리티 개선 - xoft, accessed July 19, 2025, https://xoft.tistory.com/51
16. 3D 가우시안 스플래팅 가이드 및 생성 방법 - Pixcap, accessed July 19, 2025, https://pixcap.com/kr/blog/gaussian-splatting
17. [논문리뷰] 3D Gaussian Splatting - 정리용 블로그 - 티스토리, accessed July 19, 2025, https://clean-dragon.tistory.com/13
18. What is it? - Hugging Face ML for 3D Course, accessed July 19, 2025, https://huggingface.co/learn/ml-for-3d-course/unit3/what-is-it
19. The Battle For Realism in 3D Rendering: A Brief Overview of NeRFs vs. Gaussian Splatting | by Aahana | Antaeus AR | Medium, accessed July 19, 2025, https://medium.com/antaeus-ar/the-battle-for-realism-in-3d-rendering-a-brief-overview-of-nerfs-vs-gaussian-splatting-580cff4d8801
20. Understanding and Exploring 3D Gaussian Splatting: A Comprehensive Overview, accessed July 19, 2025, https://logessiva.medium.com/understanding-and-exploring-3d-gaussian-splatting-a-comprehensive-overview-b4004f28ef1c
21. Introduction to 3D Gaussian Splatting - Hugging Face, accessed July 19, 2025, https://huggingface.co/blog/gaussian-splatting
22. 3D Gaussian Splatting: A Technical Guide to Real-Time Neural Rendering - KIRI Engine, accessed July 19, 2025, https://www.kiriengine.app/blog/3d-gaussian-splatting-a-technical-guide-to-real-time-neural-rendering
23. [2025 실감미디어 겨울학교] Gaussian Splatting 핵심 개념-최재열(성균관대학교/연구원), accessed July 19, 2025, https://www.youtube.com/watch?v=s7FkiQP0mEM
24. [2505.18764] Efficient Differentiable Hardware Rasterization for 3D Gaussian Splatting, accessed July 19, 2025, https://arxiv.org/abs/2505.18764
25. Efficient Differentiable Hardware Rasterization for 3D Gaussian Splatting - arXiv, accessed July 19, 2025, https://arxiv.org/html/2505.18764v1
26. Rasterization - gsplat documentation, accessed July 19, 2025, https://docs.gsplat.studio/main/apis/rasterization.html
27. [논문 리뷰] 3D Gaussian Splatting for Real-Time Radiance Field Rendering, accessed July 19, 2025, https://mole-starseeker.tistory.com/158
28. slothfulxtx/diff-gaussian-rasterization: Differentiable gaussian rasterization with depth, alpha, normal map and extra per-Gaussian attributes, also support camera pose gradient - GitHub, accessed July 19, 2025, https://github.com/slothfulxtx/diff-gaussian-rasterization
29. graphdeco-inria/diff-gaussian-rasterization - GitHub, accessed July 19, 2025, https://github.com/graphdeco-inria/diff-gaussian-rasterization
30. 3DGS-LM: Faster Gaussian-Splatting Optimization with Levenberg-Marquardt - GitHub Pages, accessed July 19, 2025, https://lukashoel.github.io/3DGS-LM/static/3DGS-LM_paper.pdf
31. A Survey on 3D Gaussian Splatting, accessed July 19, 2025, https://www.cs.jhu.edu/~misha/ReadingSeminar/Papers/Chen24.pdf
32. How NeRFs and 3D Gaussian Splatting are Reshaping SLAM: a Survey - arXiv, accessed July 19, 2025, https://arxiv.org/html/2402.13255v1
33. 3D Gaussian Splatting vs. NeRFs. What is the difference? | Fodev JEO, accessed July 19, 2025, https://akillness.github.io/posts/nerf-3d-gaussian-splatting/
34. Novel view synthesis, NeRF vs Gaussian splatting : r/computervision - Reddit, accessed July 19, 2025, https://www.reddit.com/r/computervision/comments/1if2w7f/novel_view_synthesis_nerf_vs_gaussian_splatting/
35. 3D Gaussian Splatting 완벽 분석 (feat. CUDA Rasterizer) - velog, accessed July 19, 2025, [https://velog.io/@gjghks950/3D-Gaussian-Splatting-%EC%99%84%EB%B2%BD-%EB%B6%84%EC%84%9D-feat.-CUDA-Rasterizer](https://velog.io/@gjghks950/3D-Gaussian-Splatting-완벽-분석-feat.-CUDA-Rasterizer)
36. Three Level Summary: Neural Radiance Fields vs. 3D Gaussian Splatting - Edward Ahn, accessed July 19, 2025, https://edwardahn.me/archive/2024/02/19/NeRFvs3DGS
37. 3D Gaussian Splatting, But Smaller - Radiance Fields, accessed July 19, 2025, https://radiancefields.com/3d-gaussian-splatting-but-smaller
38. Making Gaussian Splats smaller - Aras Pranckevičius, accessed July 19, 2025, https://aras-p.info/blog/2023/09/13/Making-Gaussian-Splats-smaller/
39. Speedy-Splat: Fast 3D Gaussian Splatting with Sparse Pixels and Sparse Primitives, accessed July 19, 2025, https://speedysplat.github.io/
40. Introduction to 3D Gaussian Splatting and Its Speedup - Fixstars Corporation Tech Blog, accessed July 19, 2025, https://blog.us.fixstars.com/introduction-to-3d-gaussian-splatting-and-its-speedup/
41. On Scaling Up 3D Gaussian Splatting Training - OpenReview, accessed July 19, 2025, https://openreview.net/forum?id=pQqeQpMkE7
42. 3DGS.zip: A survey on 3D Gaussian Splatting Compression Methods - arXiv, accessed July 19, 2025, https://arxiv.org/html/2407.09510
43. 3DGS.zip: A survey on 3D Gaussian Splatting Compression Methods - arXiv, accessed July 19, 2025, https://arxiv.org/html/2407.09510v5
44. 3DGS.zip: A survey on 3D Gaussian Splatting Compression Methods - Eurographics, accessed July 19, 2025, https://diglib.eg.org/bitstream/handle/10.1111/cgf70078/cgf70078.pdf
45. [Literature Review] Compression in 3D Gaussian Splatting: A Survey ..., accessed July 19, 2025, https://www.themoonlight.io/en/review/compression-in-3d-gaussian-splatting-a-survey-of-methods-trends-and-future-directions
46. MrNeRF/awesome-3D-gaussian-splatting - GitHub, accessed July 19, 2025, https://github.com/MrNeRF/awesome-3D-gaussian-splatting
47. 4D Gaussian Splatting for Real-Time Dynamic Scene Rendering - CVF Open Access, accessed July 19, 2025, https://openaccess.thecvf.com/content/CVPR2024/papers/Wu_4D_Gaussian_Splatting_for_Real-Time_Dynamic_Scene_Rendering_CVPR_2024_paper.pdf
48. 4D Gaussian Splatting for Real-Time Dynamic Scene Rendering - arXiv, accessed July 19, 2025, https://arxiv.org/html/2310.08528v3
49. [2310.08528] 4D Gaussian Splatting for Real-Time Dynamic Scene Rendering - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2310.08528
50. 4D Gaussian Splatting for Real-Time Dynamic Scene Rendering - Guanjun Wu, accessed July 19, 2025, https://guanjunwu.github.io/4dgs/
51. Hybrid 3D-4D Gaussian Splatting for Fast Dynamic Scene Representation - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2505.13215
52. Hybrid 3D-4D Gaussian Splatting for Fast Dynamic Scene Representation - arXiv, accessed July 19, 2025, https://arxiv.org/html/2505.13215v1
53. Hybrid 3D-4D Gaussian Splatting for Fast Dynamic Scene Representation - GitHub Pages, accessed July 19, 2025, https://ohsngjun.github.io/3D-4DGS/
54. VR-Splatting: Foveated Radiance Field Rendering via 3D Gaussian Splatting and Neural Points - Linus Franke, accessed July 19, 2025, https://lfranke.github.io/vr_splatting/
55. arxiv.org, accessed July 19, 2025, https://arxiv.org/abs/2505.10144
56. VRSplat: Fast and Robust Gaussian Splatting for Virtual Reality | AI, accessed July 19, 2025, https://www.aimodels.fyi/papers/arxiv/vrsplat-fast-robust-gaussian-splatting-virtual-reality
57. Four ways to explore 3D Gaussian splats with your Meta Quest - Scaniverse, accessed July 19, 2025, https://scaniverse.com/news/four-ways-to-explore-3d-gaussian-splats-on-meta-quest
58. These 3 Apps let you experience Gaussian Splatting on Quest : r/OculusQuest - Reddit, accessed July 19, 2025, https://www.reddit.com/r/OculusQuest/comments/1kav02v/these_3_apps_let_you_experience_gaussian/
59. HoloGaussian Digital Twin: Reconstructing 3D Scenes with Gaussian Splatting for Tabletop Hologram Visualization of Real Environments - MDPI, accessed July 19, 2025, https://www.mdpi.com/2072-4292/16/23/4591
60. Digital Twins: Precision of 3D Gaussian Splatting - Plain Concepts, accessed July 19, 2025, https://www.plainconcepts.com/digital-twins-3d-gaussian-splatting/
61. PerfCam: Digital Twinning for Production Lines Using 3D Gaussian Splatting and Vision Models - arXiv, accessed July 19, 2025, https://arxiv.org/html/2504.18165v1
62. Gaussian Splatting: The Future of Reality Capture - Heliguy, accessed July 19, 2025, https://www.heliguy.com/blogs/posts/gaussian-splatting-the-future-of-reality-capture/
63. 3D Gaussian Splatting - Plain Concepts, accessed July 19, 2025, https://www.plainconcepts.com/3d-gaussian-splatting/
64. Gaussian Splatting for Digital Twin new feature - Hiverlab, accessed July 19, 2025, https://hiverlab.com/gaussian-splatting-for-digital-twin-new-feature/
65. [2504.18165] PerfCam: Digital Twinning for Production Lines Using 3D Gaussian Splatting and Vision Models - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2504.18165
66. The rise of 3D Gaussian Splatting: what is it and how is it changing the immersive media industry? - Magnopus, accessed July 19, 2025, https://www.magnopus.com/blog/the-rise-of-3d-gaussian-splatting
67. [Workflow] 3D Gaussian Splatting Applied in Scouting and Pre-Production - YouTube, accessed July 19, 2025, https://www.youtube.com/watch?v=jIvmm3noB_s
68. 3D Gaussian Splatting from Hollywood Films! - YouTube, accessed July 19, 2025, https://www.youtube.com/watch?v=8CdLVVny9hc
69. Get This Versatile Gaussian Splats Editing Toolset For Houdini - 80 Level, accessed July 19, 2025, https://80.lv/articles/get-this-versatile-gaussian-splats-editing-toolset-for-houdini
70. GSOPS for SideFX Houdini - YouTube, accessed July 19, 2025, https://www.youtube.com/watch?v=CNo7H39OaE8
71. Free Gaussian Splatting Editor For Houdini - 80 Level, accessed July 19, 2025, https://80.lv/articles/get-this-versatile-gaussian-splats-editing-toolset-for-houdini/
72. Gaussian Splatting: When Does it Make Sense to Use and Why? - Rock Paper Reality, accessed July 19, 2025, https://rockpaperreality.com/insights/3d-vfx/gaussian-splatting-when-does-it-make-sense-to-use-and-why/
73. Gaussian splatting for beginners: The best tools for creating and editing - Design4Real, accessed July 19, 2025, https://design4real.de/en/gaussian-splatting-for-beginners-the-best-tools-for-creating-and-editing/
74. Free 3D Gaussian Splatting Tool - Polycam, accessed July 19, 2025, https://poly.cam/tools/gaussian-splatting
75. Top 5 tools for Gaussian Splatting Compared : r/photogrammetry - Reddit, accessed July 19, 2025, https://www.reddit.com/r/photogrammetry/comments/1kpu7h1/top_5_tools_for_gaussian_splatting_compared/
76. 3D Gaussian Splatting for Real-Time Radiance Field Rendering - Detailed Windows Install & Usage Instructions - GitHub, accessed July 19, 2025, https://github.com/PolarNick239/gaussian-splatting-Windows
77. Instruct-GS2GS: Editing 3D Gaussian Splatting Scenes with Instructions, accessed July 19, 2025, https://instruct-gs2gs.github.io/
78. GaussCtrl, accessed July 19, 2025, https://gaussctrl.active.vision/
79. 3D representation of Architectural Heritage: a comparative analysis of NeRF, Gaussian Splatting, and SfM-MVS reconstructions usi - ISPRS - The International Archives of the Photogrammetry, Remote Sensing and Spatial Information Sciences, accessed July 19, 2025, https://isprs-archives.copernicus.org/articles/XLVIII-2-W8-2024/93/2024/isprs-archives-XLVIII-2-W8-2024-93-2024.pdf
80. 2D Gaussian Splatting Review - 김선호 | Nerd's NeRF Team @가짜연구소 - YouTube, accessed July 19, 2025, https://www.youtube.com/watch?v=1P5gk8icwM0
81. Gsplat VRAM usage and optimisation? : r/GaussianSplatting - Reddit, accessed July 19, 2025, https://www.reddit.com/r/GaussianSplatting/comments/1iwvgbh/gsplat_vram_usage_and_optimisation/
82. Generative AI Digital Twins with 4D Gaussian Splats Training Data - Francesca Tabor, accessed July 19, 2025, https://www.francescatabor.com/articles/2024/5/8/pkg69fh3t5g8dvd8jqvo3n13cex8qy
83. Realtime Gaussian Splatting : r/GaussianSplatting - Reddit, accessed July 19, 2025, https://www.reddit.com/r/GaussianSplatting/comments/1iyz4si/realtime_gaussian_splatting/
84. RTG-SLAM: Real-time 3D Reconstruction at Scale Using Gaussian Splatting - gapszju, accessed July 19, 2025, https://gapszju.github.io/RTG-SLAM/static/pdfs/RTG-SLAM_arxiv.pdf
85. How I Built This 3D World in 11 Seconds - YouTube, accessed July 19, 2025, https://www.youtube.com/watch?v=i7fteOuIENE
86. DrivingGaussian: Composite Gaussian Splatting for Surrounding Dynamic Autonomous Driving Scenes, accessed July 19, 2025, https://agents4ad.github.io/assets/cvpr2024/papers/35.pdf
87. HDRSplat: Gaussian Splatting for High Dynamic Range 3D Scene Reconstruction from Raw Images - BMVA Archive, accessed July 19, 2025, https://bmva-archive.org.uk/bmvc/2024/papers/Paper_22/paper.pdf