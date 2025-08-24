# NVIDIA 3DGUT 기술
[NVidia Omniverse](./index.md)


3D 장면 재구성 및 렌더링 분야는 최근 몇 년간 괄목할 만한 기술적 진보를 이루었다. 본 장에서는 3D 가우시안 스플래팅(3D Gaussian Splatting, 3DGS)이 가져온 혁신과 그로 인해 드러난 명확한 기술적 한계를 조망함으로써, 3D 가우시안 언센티드 변환(3D Gaussian Unscented Transform, 3DGUT)의 등장 배경을 심도 있게 논한다.


신경망을 이용한 뷰 합성(Novel View Synthesis) 기술은 NeRF(Neural Radiance Fields)의 등장으로 새로운 국면을 맞이했다. NeRF는 장면을 연속적인 5D 방사 휘도 함수로 모델링하여 극도로 높은 시각적 품질을 달성했으나, 볼류메트릭 레이 마칭(volumetric ray marching)에 의존하는 렌더링 방식 때문에 방대한 계산량을 요구했다. 이는 느린 학습 및 렌더링 속도로 이어져 실시간 상호작용이 필수적인 응용 분야에 적용하는 데 큰 제약으로 작용했다.1

이러한 상황에서 2023년에 등장한 3DGS는 패러다임을 전환하는 혁신을 가져왔다. 3DGS는 장면을 수많은 3D 가우시안 입자(Gaussian particle)의 집합으로 명시적으로 표현하고, 이를 고도로 최적화된 래스터화(rasterization) 파이프라인을 통해 렌더링한다. 이 접근법은 NeRF의 품질을 유지하면서도 실시간 렌더링 속도를 달성하여, 학계와 산업계 전반에 걸쳐 폭발적인 반응을 일으키며 빠르게 표준 기술 중 하나로 자리 잡았다.1


3DGS의 성공은 효율적인 래스터화에 기반하지만, 바로 이 점이 기술의 근본적인 한계를 규정하는 양날의 검으로 작용했다. 주요 한계는 다음과 같다.

- **카메라 모델 제약:** 3DGS의 핵심 렌더링 기법인 EWA(Elliptical Weighted Average) 스플래팅은 3차원 가우시안을 2차원 이미지 평면에 투영할 때, 비선형적인 투영 함수를 1차 테일러 급수 전개를 통해 선형으로 근사한다. 이 근사는 이상적인 핀홀(pinhole) 카메라 모델에서는 비교적 정확하게 작동하지만, 어안(fisheye) 렌즈나 광각 렌즈와 같이 비선형 왜곡이 심한 카메라 모델에서는 심각한 투영 오류를 유발하여 렌더링 품질을 저하시킨다.3
- **야코비안(Jacobian) 의존성:** EWA 스플래팅은 선형 근사를 위해 투영 함수의 야코비안 행렬을 명시적으로 계산해야 한다. 이는 새로운 종류의 카메라 모델을 지원하고자 할 때마다 해당 모델에 대한 전용 야코비안을 수학적으로 유도하고 코드로 구현해야 함을 의미한다. 이러한 방식은 확장성이 떨어지며, 복잡한 카메라 모델에 대한 정확한 야코비안 유도는 비현실적이거나 불가능할 수 있다.3
- **동적 효과(Rolling Shutter) 처리의 어려움:** 롤링 셔터 효과와 같이 시간 종속적인(time-dependent) 왜곡은 EWA의 정적인 선형 근사 프레임워크 내에서 모델링하기 매우 비직관적이고 어렵다. 이로 인해 고속으로 움직이는 카메라로 촬영된 영상 데이터의 정확한 3D 재구성이 제한된다.4
- **2차 광선 효과(Secondary Rays) 부재:** 래스터화는 본질적으로 카메라에서 시작되는 1차 광선(primary ray)만을 처리하는 기법이다. 따라서 반사(reflection), 굴절(refraction), 그림자(shadow)와 같은 사실적인 광학 현상을 시뮬레이션하는 데 필수적인 2차 광선을 자연스럽게 처리할 수 없다. 이는 렌더링 결과물의 사실성을 제한하는 주요 요인이다.4


앞서 언급된 3DGS의 근본적인 한계들은 특히 자율주행, 로보틱스, 증강현실(AR)과 같이 비표준 카메라와 동적 환경이 일반적인 실제 응용 분야에서 심각한 제약으로 작용한다.6 이러한 문제들을 해결하기 위해 NVIDIA는 3DGUT를 제안했다. 3DGUT는 EWA 스플래팅을 추정 이론(estimation theory) 분야에서 널리 사용되는 언센티드 변환(Unscented Transform, UT)으로 대체하는 혁신적인 접근법을 취한다. 이를 통해 래스터화의 높은 효율성을 유지하면서도, 기존의 한계였던 비선형 카메라 모델 지원, 동적 효과 처리, 그리고 2차 광선 효과 구현의 길을 열었다.1

3DGUT의 등장은 단순히 3DGS의 단점을 보완하는 점진적 개선을 넘어선다. 이는 컴퓨터 그래픽스 분야의 렌더링 패러다임이 '분석적(analytical) 모델링'에서 '통계적(statistical) 근사'로 전환되고 있음을 시사하는 중요한 사례다. EWA 스플래팅은 테일러 급수 전개를 이용한 1차 선형 근사, 즉 '분석적'으로 유도된 야코비안에 의존한다.3 이 방식은 핀홀 카메라처럼 모델이 명확하고 단순할 때는 효과적이지만, 현실 세계의 복잡하고 비선형성이 강한 센서(어안 렌즈, 롤링 셔터)에 대해서는 정확한 분석적 모델을 유도하는 것이 비현실적이거나 불가능에 가깝다.6 3DGUT는 이 문제를 해결하기 위해 UT를 도입했다.1 UT는 확률 분포 자체를 결정론적 샘플링(시그마 포인트)하여 비선형 변환을 통과시킨 후, 결과 분포를 '통계적으로' 재추정하는 방식이다. 이는 문제 해결의 철학을 '완벽한 수학 공식을 유도하겠다'는 접근에서 '결과 분포를 통계적으로 가장 잘 근사하겠다'는 실용적인 접근으로 전환한 것이다. 따라서 3DGUT는 향후 등장할 미지의 복잡한 센서 모델에 대해서도 코드 수정 없이 대응할 수 있는 유연성과 확장성을 확보하게 된다. 이는 NVIDIA가 Omniverse와 같은 거대 가상 세계 구축을 목표로 할 때, 다양한 데이터 소스를 통합하기 위한 필수적인 기술적 토대를 마련하는 전략적 움직임으로 해석될 수 있다.13


본 장에서는 3DGUT의 기술적 심장인 언센티드 변환의 수학적 원리를 상세히 분석하고, 이것이 어떻게 EWA 스플래팅의 근본적인 문제를 해결하는지 수식을 통해 명확히 설명한다.


3DGS에서 각 가우시안 입자는 평균 $\mu \in \mathbb{R}^3$와 공분산 $\Sigma \in \mathbb{R}^{3 \times 3}$를 갖는 3차원 정규분포 $\mathcal{N}(\mathbf{x} \mid \mu, \Sigma)$로 정의된다.1 이 3D 가우시안을 2D 이미지 평면으로 투영하는 변환 함수를 

$\mathbf{u} = \pi(\mathbf{x})$라 할 때, 이 함수는 원근 투영으로 인해 비선형적이다. EWA 스플래팅은 이 비선형 변환을 가우시안의 평균 $\mu$ 지점에서 1차 테일러 급수로 근사한다.
$$
\pi(\mathbf{x}) \approx \pi(\mu) + J(\mathbf{x} - \mu)
$$
여기서 $J$는 $\mu$에서 평가된 $\pi(\cdot)$의 야코비안 행렬이다.6 이 선형 근사를 통해, 투영된 2D 가우시안의 공분산 

$\Sigma'$은 다음과 같이 간단하게 계산된다.
$$
\Sigma' = J \Sigma J^T
$$
이 방식은 계산이 빠르지만, 카메라 왜곡이 심해져 비선형성이 커질수록 실제 투영 결과와의 오차가 기하급수적으로 증가하는 치명적인 단점을 가진다.3


언센티드 변환은 비선형 함수 자체를 근사하는 대신, 입력 확률 분포를 대표하는 소수의 샘플 포인트(시그마 포인트)를 통해 변환 후의 분포를 통계적으로 추정한다. 이는 "임의의 비선형 함수를 근사하는 것보다 가우시안 분포를 근사하는 것이 더 쉽다"는 핵심 철학에 기반한다.14 UT의 과정은 다음과 같다.

1. **시그마 포인트(Sigma Points) 생성:** $n$차원 가우시안 분포 $\mathcal{N}(\mu, \Sigma)$를 표현하기 위해, $2n+1$개의 시그마 포인트 $\mathcal{X}_i$와 해당 가중치 $W_i$를 결정론적으로 선택한다. 3D 가우시안의 경우 $n=3$이다.
   $$
   \begin{aligned}
   \mathcal{X}_0 &= \mu \\
   \mathcal{X}_i &= \mu + (\sqrt{(n+\lambda)\Sigma})_i \quad \text{for } i=1, \dots, n \\
   \mathcal{X}_{i+n} &= \mu - (\sqrt{(n+\lambda)\Sigma})_i \quad \text{for } i=1, \dots, n
   \end{aligned}
   $$
   여기서 $(\sqrt{(n+\lambda)\Sigma})_i$는 행렬 $(n+\lambda)\Sigma$의 행렬 제곱근(예: Cholesky 분해)의 i번째 열 벡터를 의미하며, $\lambda = \alpha^2(n+\kappa) - n$은 시그마 포인트의 퍼짐 정도를 조절하는 스케일링 파라미터다.9

2. **시그마 포인트의 비선형 변환:** 생성된 각 시그마 포인트를 비선형 투영 함수 $\pi(\cdot)$에 직접 통과시킨다.
   $$
   \mathcal{Y}_i = \pi(\mathcal{X}_i) \quad \text{for } i=0, \dots, 2n
   $$
   이 과정은 EWA와 달리 어떠한 근사도 없이 정확하게 계산된다는 것이 핵심이다.1

3. **결과 분포 재추정:** 변환된 포인트 $\mathcal{Y}_i$와 미리 정의된 가중치 $W_i$를 사용하여 결과 분포의 평균 $\mu'$과 공분산 $\Sigma'$을 통계적으로 계산한다.
   $$
   \begin{aligned}
   \mu' &\approx \sum_{i=0}^{2n} W_i^{(m)} \mathcal{Y}_i \\
   \Sigma' &\approx \sum_{i=0}^{2n} W_i^{(c)} (\mathcal{Y}_i - \mu')(\mathcal{Y}_i - \mu')^T
   \end{aligned}
   $$
   이렇게 얻어진 $\mu'$와 $\Sigma'$이 바로 래스터화에 사용될 최종 2D 가우시안의 파라미터가 된다.9


3DGUT는 3DGS의 최적화 프레임워크를 대부분 그대로 계승한다. 즉, 렌더링된 이미지와 실제 이미지 간의 재렌더링 손실(re-rendering loss, 주로 L1 손실과 D-SSIM 손실의 조합)을 최소화하는 방향으로 가우시안 입자의 파라미터(위치 $\mu$, 크기/회전 $\Sigma$, 불투명도 $\alpha$, 색상)를 최적화한다.1

핵심적인 차이점은 순전파(forward pass) 과정에서 3D 가우시안을 2D로 투영할 때 EWA 대신 UT 기반의 방식을 사용한다는 점이다. 이 전체 과정은 미분 가능(differentiable)하게 설계되어, 역전파(backpropagation)를 통해 손실 함수의 그래디언트가 가우시안 파라미터까지 효과적으로 전달될 수 있다. UT는 야코비안 계산이 필요 없는(derivative-free) 접근법이므로, 어떤 복잡한 비선형 카메라 모델이 주어져도 동일한 프레임워크를 수정 없이 적용할 수 있는 강력한 일반성을 제공한다.8

UT의 도입은 단순히 투영 정확도를 높이는 것을 넘어, 3DGS의 '렌더링 공식' 자체를 레이 트레이싱과 유사한 방식으로 재정의하여 하이브리드 렌더링(3DGRUT)의 기술적 토대를 마련하는 핵심적인 아키텍처 변경이다. 기존 3DGS는 투영된 2D 이미지 평면에서 픽셀에 대한 가우시안의 기여도를 계산하는 전형적인 래스터화 방식을 따른다.6 반면, 3DGRT와 같은 레이 트레이싱 방식은 3D 공간에서 광선(ray)과 가우시안 입자의 상호작용을 직접 계산한다.17 3DGUT는 UT를 사용한 후, 3DGRT의 접근법을 따라 광선을 따라 가장 큰 반응(maximum response)을 보이는 3D 지점에서 입자를 평가하는 방식을 채택했다.6 이러한 '3D 공간에서의 평가' 방식의 통일은 3DGUT로 학습된 가우시안 씬 표현이 3DGRT 렌더러와 완벽하게 호환되도록 만드는 결정적인 역할을 한다.3 결론적으로, UT의 도입은 단지 비선형 카메라 문제를 해결하기 위한 국소적인 해결책이 아니라, 3DGS의 렌더링 철학을 레이 트레이싱과 정렬(align)시켜 하나의 씬 표현을 두고 래스터라이저와 레이 트레이서가 선택적으로 작동할 수 있는 유연한 '하이브리드 렌더링 플랫폼'을 구축하기 위한 필수적인 사전 작업이었던 것이다.

**표 1: EWA 스플래팅과 Unscented Transform의 개념적 비교**

| 특징             | EWA Splatting (3DGS)                | Unscented Transform (3DGUT)                                |
| ---------------- | ----------------------------------- | ---------------------------------------------------------- |
| **근사 대상**    | 비선형 투영 함수 $\pi(\cdot)$       | 입력 확률 분포 $\mathcal{N}(\mu, \Sigma)$                  |
| **근사 방식**    | 1차 테일러 급수 전개 (선형화)       | 시그마 포인트를 이용한 통계적 샘플링                       |
| **야코비안 $J$** | 필수 (각 카메라 모델마다 유도 필요) | 불필요 (Derivative-free)                                   |
| **정확도**       | 비선형성이 클수록 급격히 저하       | 2차 모멘트까지 높은 정확도 유지                            |
| **지원 카메라**  | 이상적인 핀홀 카메라에 제한         | 임의의 비선형/왜곡 카메라 지원                             |
| **동적 효과**    | 모델링 매우 어려움                  | 시간 종속적 함수를 $\pi(\cdot)$에 통합하여 자연스럽게 지원 |


본 장에서는 3DGUT의 핵심 기능인 비선형 카메라 및 동적 효과 지원 능력을 실제 사례와 함께 구체적으로 분석하고, 이것이 3D 재구성 기술의 응용 범위를 어떻게 확장하는지 탐구한다.


UT의 강력한 비선형 함수 처리 능력 덕분에, 3DGUT는 어안 렌즈와 같이 극심한 방사형 왜곡(radial distortion)을 가진 카메라로 촬영된 이미지에서도 높은 품질의 3D 장면 재구성을 성공적으로 수행한다.6 기존 3DGS 방식은 이러한 이미지를 처리하기 위해 먼저 왜곡을 보정(rectify)하는 전처리 과정을 거쳐야 했다. 이 과정은 이미지 외곽부의 화각을 잘라내어 상당한 정보 손실을 유발하며, 특히 360도에 가까운 넓은 화각을 가진 어안 렌즈의 장점을 무력화시켰다.6 3DGUT는 왜곡된 원본 이미지를 직접 입력으로 사용하여 모든 픽셀 정보를 활용하므로, 더 완전하고 정확한 장면 재구성이 가능하다.

어안 렌즈 지원을 목표로 하는 경쟁 기술인 FisheyeGS와의 비교에서 3DGUT의 우수성은 명확히 드러난다. FisheyeGS는 특정 어안 모델에 대한 전용 야코비안을 유도해야 하므로 다른 종류의 왜곡 모델에는 적용하기 어려운 반면, 3DGUT는 범용적인 UT를 사용하므로 어떤 카메라 모델에도 쉽게 적응할 수 있다. 실험 결과, 3DGUT는 FisheyeGS보다 절반 이하의 가우시안 입자(0.38M vs 1.07M)를 사용하면서도 모든 지각적 품질 메트릭에서 월등한 성능을 보였다.6


롤링 셔터는 CMOS 센서가 이미지의 각 행(row)을 순차적으로 노출시키기 때문에 발생하는 현상이다. 카메라나 피사체가 빠르게 움직일 때 이미지 상단과 하단의 촬영 시점이 달라져 '젤리 현상'과 같은 기하학적 왜곡을 유발한다. 이는 자율주행 차량, 드론, 로봇 등 고속으로 움직이는 센서에서 얻은 데이터의 정확성을 심각하게 저해하는 문제다.

3DGUT는 이 동적 효과를 카메라의 투영 함수 $\pi(\cdot)$에 시간 변수를 포함시켜 자연스럽게 모델링한다. 즉, 각 시그마 포인트를 2D 이미지 평면에 투영할 때, 해당 픽셀이 노출되는 정확한 시간 $t$를 고려한 카메라의 포즈(위치 및 방향)를 적용하는 것이다: $\mathcal{Y}_i = \pi(\mathcal{X}_i, t)$. 이 방식은 EWA 스플래팅의 정적인 프레임워크에서는 구현하기 극히 어려운 시간 종속적 변환을 우아하게 해결한다.1 실제 자율주행 데이터셋인 Waymo에서의 실험 결과, 3DGUT는 롤링 셔터 효과를 효과적으로 처리하며 순수 레이 트레이싱 기반 방법론인 3DGRT와 대등하거나 그 이상의 성능을 달성했다. 이는 3DGUT가 실제 산업 현장에서 마주하는 복잡하고 동적인 데이터를 정확하게 처리할 수 있는 실용적인 기술임을 입증한다.6

3DGUT의 비선형 및 동적 카메라 지원 능력은 단순히 '더 많은 종류의 카메라를 지원한다'는 기술적 성취를 넘어, 3D 재구성 기술의 데이터 수집 및 전처리 파이프라인 전체를 혁신할 잠재력을 가진다. 기존의 3DGS 파이프라인은 COLMAP과 같은 SfM(Structure-from-Motion) 도구에 크게 의존하며, 이들 도구는 이상적인 핀홀 카메라 모델을 가정하거나 왜곡이 보정된 이미지를 입력으로 요구하는 경우가 많다.2 이 왜곡 보정 과정은 그 자체로 복잡하며, 카메라 내부 파라미터(intrinsics)에 대한 사전 지식이 필요하고 정보 손실을 필연적으로 유발한다.2 이는 데이터 수집 단계에서부터 상당한 제약 조건으로 작용했다. 3DGUT는 원본 왜곡 이미지(raw distorted image)를 직접 처리할 수 있으므로 6, 값비싼 전처리 과정과 그로 인한 정보 손실을 원천적으로 제거한다. 따라서 사용자는 더 이상 완벽하게 보정된 '학습용 데이터셋'을 만들기 위해 노력할 필요 없이, 저렴한 어안 렌즈나 일반 스마트폰 카메라로 촬영한 '날 것 그대로의' 데이터를 바로 고품질 3D 모델링에 활용할 수 있게 된다. 이는 3D 콘텐츠 제작의 민주화(democratization)를 가속화하고, 대규모 실세계 환경을 스캔하는 데 드는 비용과 시간을 획기적으로 줄여줄 수 있다. 예를 들어, 360도 카메라 한 대로 빠르게 촬영한 영상만으로도 고품질의 3D 공간을 생성하는 워크플로우가 현실화될 수 있다.19


본 장에서는 3DGUT와 3D 가우시안 레이 트레이싱(3D Gaussian Ray Tracing, 3DGRT)의 결합으로 탄생한 하이브리드 렌더링 시스템 3DGRUT의 개념을 설명하고, 이를 통해 어떻게 사실적인 광학 효과를 효율적으로 구현하는지 심층적으로 분석한다.


3DGRT는 3D 가우시안 입자 씬을 래스터화 대신 순수 레이 트레이싱 기법으로 렌더링하는 NVIDIA의 선행 연구다.4 레이 트레이싱은 광선을 직접 추적하므로 본질적으로 어떤 복잡한 카메라 모델이든 지원하며, 반사나 굴절 같은 2차 광선 효과를 물리적으로 정확하게 시뮬레이션할 수 있다. 그러나 이 방식은 래스터화보다 연산 비용이 훨씬 높고, 효율적인 실행을 위해 RT 코어와 같은 전용 하드웨어를 요구하는 단점이 있다.3

3DGUT의 설계에서 가장 중요한 측면 중 하나는 렌더링 공식을 3DGRT와 의도적으로 정렬(align)했다는 점이다. 제2장에서 분석했듯이, 3DGUT는 3D 공간에서 광선 방향을 따라 입자의 기여도를 평가하는 방식을 채택함으로써, 동일한 3D 가우시안 씬 표현(scene representation)을 3DGUT의 래스터라이저와 3DGRT의 레이 트레이서가 완벽하게 공유할 수 있도록 만들었다.1 이 기술적 호환성이 바로 3DGRUT 하이브리드 렌더링의 기반이 된다.


3DGRUT는 래스터화의 속도와 레이 트레이싱의 품질이라는 두 마리 토끼를 모두 잡기 위한 하이브리드 렌더링 시스템이다.9 렌더링 과정은 다음과 같이 두 단계로 나뉜다.

1. **1단계 (Primary Rays via Rasterization):** 먼저 카메라에서 시작되는 1차 광선에 대해서는 3DGUT의 고속 래스터화 방식을 사용하여 씬의 기본적인 가시성(visibility)과 색상을 계산한다. 이 단계는 이미지의 대부분 픽셀을 효율적으로 처리하여 실시간 성능을 보장한다.
2. **2단계 (Secondary Rays via Ray Tracing):** 1차 광선이 거울이나 유리처럼 반사율(reflectivity)이나 굴절률(refractivity)이 높은 표면 재질을 가진 가우시안 입자와 충돌하면, 해당 지점에서 물리 법칙에 따라 반사 또는 굴절 방향으로 새로운 2차 광선을 생성한다. 이 2차 광선들은 3DGRT의 레이 트레이싱 엔진을 통해 추적되어 주변 환경의 색상 정보를 가져오고, 최종 픽셀 색상에 더해진다.6

이러한 분리된 접근 방식은 대부분의 픽셀을 차지하는 확산(diffuse) 표면은 래스터화로 빠르게 처리하고, 시각적 사실성에 결정적인 영향을 미치는 소수의 정반사(specular) 및 투과(transmission) 효과만 레이 트레이싱으로 정밀하게 계산함으로써 성능과 품질 간의 최적의 균형을 맞춘다.


3DGRUT 파이프라인을 통해 기존 3DGS에서는 원천적으로 불가능했던 고품질의 물리 기반 광학 효과를 렌더링할 수 있다.4 예를 들어, 사용자는 3DGUT로 재구성된 현실 공간에 반사되는 금속 구나 굴절하는 유리 조각 같은 가상 객체를 삽입하고, 이 객체들이 주변 환경을 사실적으로 비추거나 왜곡시키는 모습을 실시간으로 확인할 수 있다.6 이는 3D 콘텐츠의 몰입감과 현실감을 극대화하는 핵심적인 기능이다.

3DGRUT의 등장은 3D 가우시안 표현이 단순한 '뷰 합성'을 위한 데이터 구조를 넘어, 물리 기반 렌더링(Physically Based Rendering, PBR) 및 디지털 트윈 시뮬레이션을 위한 '완전한 3D 씬 표현'으로 진화하고 있음을 의미한다. 초기 3DGS는 주어진 이미지들을 보간하여 새로운 뷰를 '그럴듯하게' 만들어내는 것에 중점을 둔 이미지 기반 렌더링에 가까웠다. 그러나 3DGRUT는 여기에 레이 트레이싱을 통한 물리적 광선 상호작용 시뮬레이션을 추가했다.6 이는 씬이 단순히 색상 값의 집합이 아니라, 빛과 상호작용하는 물리적 속성을 가진 공간으로 취급되기 시작했음을 의미한다. NVIDIA의 공식 GitHub 저장소에서 PBR 메시 및 환경 맵(environment map) 지원을 언급한 것은 이러한 방향성을 명확히 보여준다.17 이는 3D 가우시안 씬에 재질(material) 속성을 부여하고, 외부 조명의 영향을 물리적으로 정확하게 계산하려는 의도를 나타낸다. 이러한 기능들은 정확한 물리적 시뮬레이션이 필수적인 디지털 트윈, 자율주행 시뮬레이션 등의 응용 분야의 요구사항과 정확히 일치한다.13 따라서 3DGRUT는 3DGS를 '사진처럼 보이는 이미지를 만드는 기술'에서 '물리적으로 정확한 가상 세계를 구축하고 시뮬레이션하는 기술'로 격상시키는 결정적인 단계이며, 이는 NVIDIA의 Omniverse 플랫폼 전략과도 완벽하게 부합한다.13


본 장에서는 공개된 벤치마크 데이터를 바탕으로 3DGUT의 성능을 다각도로 분석하고, 주요 경쟁 기술들과의 심층 비교를 통해 그 우수성과 기술적 특징을 명확히 한다. 모든 성능 수치는 별도의 언급이 없는 한 NVIDIA RTX 5090 GPU에서 측정된 결과다.


3DGUT는 다양한 종류의 데이터셋에서 그 성능을 입증했다.

- **NeRF Synthetic Dataset:** 이상적인 핀홀 카메라를 사용하는 이 합성 데이터셋에서 3DGUT는 평균 PSNR 33.88, SSIM 0.970의 높은 이미지 품질을 달성했다. 평균 렌더링 속도는 846 FPS로 실시간 성능을 상회했으며, 평균 학습 시간은 214.6초를 기록했다.17 이 결과는 3DGUT가 기존 3DGS와 대등한 수준의 품질과 속도를 제공하며, UT 도입으로 인한 오버헤드가 거의 없음을 보여준다.3
- **MipNeRF360 Dataset:** 360도 시점의 복잡하고 넓은 실제 장면을 다루는 이 데이터셋에서도 3DGUT는 MCMC(Markov Chain Monte Carlo) 밀집화 전략 사용 시 평균 PSNR 27.78, SSIM 0.822, 렌더링 속도 308 FPS를 기록했다.17 이는 대규모 실외 장면에서도 3DGS와 유사한 지각적 품질을 유지할 수 있음을 의미한다.6
- **Scannet++ Dataset:** 실제 실내 환경을 스캔한 이 데이터셋에서는 평균 PSNR 29.22, 렌더링 속도 434 FPS를 기록하여, 복잡한 기하 구조와 재질을 가진 실내 환경 재구성 능력도 우수함을 입증했다.17
- **Waymo Open Dataset:** 롤링 셔터와 카메라 렌즈 왜곡이 포함된 실제 자율주행 데이터셋에서 3DGUT의 진가가 드러난다. 3DGUT(정렬된 렌더링 방식)는 PSNR 30.16을 기록하여, 순수 레이 트레이싱 방식인 3DGRT(29.99)를 미세하게 능가하는 결과를 보였다. 이는 왜곡이 보정된 이미지로만 학습이 가능한 3DGS(29.83)보다 월등히 우수한 성능이다.23


- **vs. 3DGS:** 핀홀 카메라를 사용하는 이상적인 조건의 데이터셋에서는 품질과 속도 면에서 대등한 성능을 보인다. 그러나 어안 카메라나 롤링 셔터와 같은 비이상적인 환경에서는 3DGUT가 구조적으로 우월하며, 왜곡을 직접 모델링하여 훨씬 높은 품질의 결과를 생성한다.3
- **vs. 3DGRT:** 3DGUT는 래스터화 기반이므로 렌더링 속도 면에서 순수 레이 트레이싱 기반인 3DGRT보다 3~4배 빠르다.8 Waymo 데이터셋 결과에서 볼 수 있듯이, 복잡한 실제 환경에서는 3DGUT가 품질 면에서도 3DGRT를 앞설 수 있다.23 3DGRT는 반사, 굴절과 같은 2차 광선 효과의 물리적 정확성에서 강점을 가진다.
- **vs. FisheyeGS:** 어안 렌즈 데이터셋에서 3DGUT는 모든 지각적 메트릭에서 FisheyeGS를 압도했다. 특히 3DGUT는 절반 이하의 가우시안 입자로 더 나은 결과를 생성하여, UT 기반 접근법의 범용성과 효율성을 명확히 입증했다.6


투영 방식의 정확도를 정량적으로 평가하기 위해, 연구팀은 몬테카를로 샘플링을 통해 얻은 '참 값'에 가까운 2D 투영 분포와, EWA 및 UT로 근사한 2D 투영 분포 간의 KL 발산을 측정했다. KL 발산 값이 낮을수록 두 분포가 유사함을 의미한다.6

실험 결과, 정적인 핀홀 카메라 조건에서는 두 방식의 KL 발산 분포가 유사했다. 그러나 정적인 어안 카메라 조건에서는 UT 기반 투영의 KL 발산이 EWA보다 현저히 낮았으며, 롤링 셔터가 추가된 동적 조건에서는 그 차이가 극명하게 벌어졌다. 롤링 셔터를 고려하지 않은 EWA의 근사는 완전히 실패하여 매우 높은 KL 발산 값을 보인 반면, 3DGUT의 UT 기반 투영은 동적 효과를 정확히 반영하여 낮은 KL 발산 값을 유지했다. 이는 UT가 비선형 변환을 훨씬 더 정확하게 근사함을 정량적으로 증명하는 강력한 증거다.6

3DGUT의 성능 프로파일은 단일 지표로 평가할 수 없는 '상황 적응적 우월성(context-adaptive superiority)'을 보여준다. 즉, '어떤 기술이 무조건 더 좋은가?'라는 질문은 올바르지 않다. 공개된 데이터 17를 종합해 보면, 3DGUT, 3DGS, 3DGRT는 각각 다른 시나리오에서 최적의 성능을 보인다. 스튜디오에서 핀홀 카메라로 촬영한 이상적인 데이터의 경우, 3DGS는 이미 충분히 빠르고 고품질의 결과를 제공하며 3DGUT는 여기서 대등한 성능을 유지하여 하위 호환성을 보장한다.3 반면, 어안 렌즈나 롤링 셔터가 포함된 실제 센서 데이터의 경우, 3DGS의 기본 가정이 무너지므로 3DGUT가 압도적인 우위를 점한다.6 마지막으로, 반사나 굴절과 같은 고품질 광학 효과가 필요한 경우에는 3DGRT(또는 3DGRUT)가 유일한 해결책이다.6 결국 NVIDIA는 하나의 기술로 모든 문제를 해결하려는 것이 아니라, 3DGRUT라는 통합 프레임워크 17 내에서 사용자가 자신의 요구사항(속도, 정확성, 사실성)에 따라 렌더링 모드를 선택할 수 있는 '기술 도구 상자(toolbox)'를 제공하고 있다. 이는 기술의 높은 성숙도를 보여주는 중요한 지표다.

**표 2: 주요 데이터셋 기반 3DGUT 성능 종합 비교**

| 데이터셋           | 모델           | PSNR ↑    | SSIM ↑    | FPS ↑ | 학습 시간(s) ↓ | 비고                                    |
| ------------------ | -------------- | --------- | --------- | ----- | -------------- | --------------------------------------- |
| **NeRF Synthetic** | 3DGUT          | 33.88     | 0.970     | 846   | 214.6          | 3DGS와 대등한 성능 17                   |
| **MipNeRF360**     | 3DGUT (MCMC)   | 27.78     | 0.822     | 308   | 596.7          | 대규모 씬 처리 능력 17                  |
|                    | 3DGS           | ~27.5     | ~0.81     | ~310  | ~680           | 3DGUT와 유사한 성능 6                   |
| **Waymo**          | 3DGUT (sorted) | **30.16** | **0.900** | -     | -              | 롤링 셔터, 왜곡 환경에서 최상급 성능 23 |
|                    | 3DGRT          | 29.99     | 0.897     | -     | -              | 레이 트레이싱 기반, 약간 낮은 품질 23   |
|                    | 3DGS           | 29.83     | 0.917     | -     | -              | 왜곡 보정된 이미지 사용, 한계 명확 23   |
| **Fisheye Data**   | 3DGUT          | **우수**  | **우수**  | -     | -              | 0.38M 파티클 6                          |
|                    | FisheyeGS      | 열세      | 열세      | -     | -              | 1.07M 파티클 6                          |


본 장에서는 3DGUT의 독보적인 기술적 특성이 실제 산업 분야에서 어떤 혁신적인 가치를 창출할 수 있는지 구체적인 사례를 통해 탐구한다.


자율주행차와 로봇은 주변 환경을 360도로 완벽하게 인식하기 위해 다수의 광각 및 어안 카메라를 핵심 센서로 사용한다. 이들 시스템은 고속으로 이동하기 때문에 롤링 셔터 왜곡이 필연적으로 발생한다.6 3DGUT는 이러한 '비이상적인' 센서 데이터를 전처리 없이 직접 처리하여, 더 정확하고 강건한 3D 환경 모델을 실시간으로 구축할 수 있게 해준다. 이는 차량의 인지(perception) 시스템의 성능을 획기적으로 향상시켜, 돌발 상황에 대한 대응 능력을 높이고 궁극적으로 주행 안전성을 확보하는 데 결정적인 기여를 할 수 있다.6


3DGRUT의 하이브리드 렌더링 능력은 물리적으로 정확한 조명 시뮬레이션을 가능하게 한다. 이는 제조, 건축, 도시 계획 분야에서 현실 세계와 동일한 가상 환경, 즉 디지털 트윈을 구축하고 다양한 시나리오를 시뮬레이션하는 데 필수적인 기술이다.13 예를 들어, 자율주행 AI 모델을 훈련시키기 위한 시뮬레이터에서 실제와 동일한 햇빛 반사, 젖은 노면의 빛 번짐, 터널 진입 시의 급격한 조도 변화 등을 정밀하게 재현할 수 있다. 이를 통해 AI 모델이 현실에서 마주칠 수 있는 다양한 엣지 케이스(edge case)에 대한 대응 능력을 가상 환경에서 안전하고 효율적으로 검증하고 강화할 수 있다.


가상현실(VR) 및 증강현실(AR) 헤드셋은 사용자에게 넓은 시야각(Field of View, FoV)을 제공해야 하므로 광각 또는 어안 렌즈를 사용한다. 3DGUT는 이러한 기기에서 캡처된 영상을 왜곡 없이 처리하여, 사용자가 보는 현실 공간 위에 자연스럽게 중첩되는 사실적인 가상 콘텐츠를 생성하는 데 활용될 수 있다.6 또한, 3DGUT는 기존 3DGS보다 적은 수의 이미지로도 고품질 3D 모델을 생성할 수 있는 잠재력을 보여준다.20 이는 사용자가 스마트폰으로 주변을 간단히 촬영하는 것만으로 자신만의 메타버스 공간이나 3D 에셋을 손쉽게 만들 수 있는 강력한 콘텐츠 제작 도구로 발전할 가능성을 시사한다.

3DGUT는 NVIDIA의 'AI 파운드리(AI Foundry)' 전략의 핵심 구성 요소로, 실세계 데이터를 캡처하여 Omniverse와 같은 가상 세계 시뮬레이션 플랫폼으로 주입하는 '데이터 인제스천(Data Ingestion)' 파이프라인의 병목을 해결하는 중추적인 역할을 수행한다. NVIDIA는 AI 모델 개발 및 배포를 위한 통합 플랫폼(AI Foundry, NIM, AI Enterprise)을 구축하고 있으며, 이 플랫폼의 성공은 방대한 양의 고품질 데이터에 달려있다.25 특히 디지털 트윈과 메타버스를 위해서는 실세계를 가상 세계로 정확하게 복제하는 기술이 필수적이다.13 기존의 3D 캡처 기술은 이상적인 카메라가 필요하고 왜곡 보정과 같은 복잡한 전처리 과정이 요구되어 대규모 확장에 어려움을 겪었다. 이것이 바로 '데이터 인제스천'의 병목 현상이다. 3DGUT는 저렴하고 일반적인 카메라(어안, 롤링 셔터)로 수집된 '오염된' 실세계 데이터를 직접 고품질 3D 에셋으로 변환함으로써 이 병목을 해결한다.6 이렇게 생성된 3D 에셋은 USDZ와 같은 표준 형식으로 익스포트되어 Omniverse 및 Isaac Sim과 같은 NVIDIA의 시뮬레이션 플랫폼에서 즉시 사용될 수 있다.17 결론적으로, 3DGUT는 단순한 렌더링 기술이 아니라, NVIDIA의 거대한 AI 및 시뮬레이션 생태계의 가장 앞단에서 실세계 데이터를 가상 세계로 공급하는 핵심적인 '연결고리'이자 '데이터 정제소' 역할을 수행하는 전략적 기술이다.


3DGUT는 3D 장면 재구성 분야에 혁신을 가져왔지만, 모든 기술과 마찬가지로 이론적 제약과 실제 구현상의 이슈를 가지고 있다. 본 장에서는 이러한 한계를 객관적으로 분석하고, 이를 바탕으로 향후 개선 및 연구 방향을 제시한다.


3DGUT의 핵심인 언센티드 변환은 그 자체로 몇 가지 이론적 가정을 내포하고 있으며, 이 가정이 깨질 경우 성능이 저하될 수 있다.

- **가우시안 가정의 한계:** 표준 UT는 변환 대상이 되는 확률 분포가 가우시안이거나 가우시안에 가깝다고 가정한다. 만약 실제 데이터의 노이즈나 불확실성이 심하게 비대칭적이거나 여러 개의 봉우리를 갖는 다중 모드(multi-modal) 등 비가우시안(non-Gaussian) 분포 특성을 강하게 띨 경우, $2n+1$개의 시그마 포인트만으로는 분포를 충분히 잘 표현하지 못해 근사 정확도가 저하될 수 있다.14
- **고도의 비선형성에 대한 강건성:** UT는 1차 선형 근사를 사용하는 확장 칼만 필터(EKF)보다 비선형성에 훨씬 강건하지만, 변환 함수가 불연속적이거나 비선형성이 극도로 심한 경우에는 여전히 상당한 근사 오차를 유발할 수 있다. 예를 들어, 갑작스러운 폐색(occlusion)이나 매우 복잡한 굴절 현상 모델링 시 한계를 보일 수 있다.14
- **차원의 저주(Curse of Dimensionality):** 시그마 포인트의 수는 입력 변수의 차원 $n$에 선형적으로 비례한다($2n+1$). 3D 가우시안의 위치를 다루는 현재의 경우 $n=3$으로 전혀 문제가 되지 않지만, 만약 위치, 속도, 재질 파라미터 등 더 높은 차원의 상태 변수를 동시에 추정하는 문제에 UT를 적용할 경우, 시그마 포인트의 수가 증가하여 계산 비용이 상승할 수 있다.14


공식 GitHub 저장소의 이슈 트래커와 커뮤니티 보고를 통해 실제 구현에서 몇 가지 문제점들이 알려졌다.

- **설치 및 환경 설정의 복잡성:** 공식 구현체는 초기에는 Linux 환경만 지원했으며, Windows 환경에서의 설치 과정이 복잡하여 많은 사용자들이 어려움을 겪었다. CUDA, PyTorch, Visual Studio 등 여러 의존성 라이브러리의 특정 버전 요구 사항은 기술의 대중적 채택을 저해하는 초기 장벽으로 작용했다.27
- **학습 불안정성:** 일부 사용자는 특정 조건(예: 기본 설정 외의 커스텀 플래그 사용)에서 학습을 진행할 때, 특정 반복 횟수(예: 7,000 스텝) 이후 재구성 결과가 갑자기 붕괴되어 흐릿해지는 현상을 보고했다. 이는 손실 함수가 발산하는 문제로, 모델이 특정 하이퍼파라미터에 민감하거나 구현에 수치적으로 불안정한 부분이 있을 수 있음을 시사한다.29
- **렌더링 아티팩트:** 렌즈 왜곡이 매우 심한 환경에서 결과물에 뾰족한 스파이크 형태의 아티팩트가 발생하거나, 특정 데이터셋에서 검은 이미지가 출력되는 등의 렌더링 관련 버그가 보고되었다. 이는 엣지 케이스에 대한 처리나 메모리 접근 등에서 발생할 수 있는 문제다.27


이러한 한계들을 극복하기 위해 다음과 같은 연구 방향을 제언할 수 있다.

- **일반화된 Unscented Transform(GenUT) 도입:** 비가우시안 분포를 더 정확하게 처리하기 위해, 분포의 왜도(skewness)와 첨도(kurtosis)까지 고려하여 시그마 포인트를 생성하는 GenUT와 같은 고차 UT 기법을 도입하는 연구가 필요하다. 이는 모델의 강건성을 높여 더 까다로운 실제 환경 데이터에 대한 성능을 향상시킬 수 있다.14
- **자동화된 하이퍼파라미터 튜닝 및 안정화:** 학습 불안정성 문제를 해결하기 위해 적응형 학습률(adaptive learning rate) 스케줄링, 그래디언트 클리핑, 그리고 논문에서 제안된 수치적으로 안정적인 미분 계산 기법 `23` 등을 적극적으로 적용하여 학습 과정을 안정화하는 연구가 요구된다.
- **생태계 확장 및 사용자 편의성 개선:** Nerfstudio와 같은 대중적인 통합 프레임워크에 3DGUT를 안정적으로 통합하고 7, Docker 등을 활용하여 설치 과정을 간소화하며, Windows 및 Vulkan API와 같은 다양한 플랫폼 지원을 완성하는 것은 기술 확산에 매우 중요하다.17

현재 보고된 3DGUT의 문제점들은 기술 자체의 근본적 결함이라기보다는, '연구 프로토타입'에서 '상용 수준의 안정적인 소프트웨어'로 전환되는 과정에서 겪는 전형적인 '성장통'으로 해석될 수 있다. 3DGUT는 CVPR 2025에 발표된 최신 연구 결과로 4, 학술적 참신성을 입증하는 것이 최우선 목표였을 것이다. GitHub에 보고된 이슈들 27은 대부분 설치, 특정 환경에서의 실행 오류, 특정 데이터에 대한 학습 실패 등 '엔지니어링' 및 '일반화'와 관련된 문제들이다. 이는 핵심 아이디어(UT의 적용)가 틀렸다는 증거라기보다는, 다양한 엣지 케이스에 대한 테스트와 폴리싱이 아직 부족함을 보여준다. 이러한 문제들은 오픈소스 커뮤니티의 피드백과 기여를 통해 점진적으로 해결될 수 있는 성격의 문제이며, 실제로 Nerfstudio와 같은 대형 오픈소스 프로젝트에서 3DGUT를 통합하려는 움직임 7은 이러한 성숙 과정을 가속화할 것이다. 더 나아가, UT의 이론적 한계와 실제 구현상의 불안정성 사이의 연관성을 파고드는 것은 새로운 학술 연구의 기회를 제공한다. 결론적으로, 3DGUT의 현재 한계는 기술의 종말이 아닌, 학계와 산업계가 함께 참여하여 기술을 더욱 성숙시킬 수 있는 새로운 시작점을 의미한다.


본 보고서는 NVIDIA의 3DGUT 기술에 대해 다각도로 심층 분석했다. 3DGUT는 기존 3DGS 기술이 가진 본질적인 한계를 극복하고, 3D 장면 재구성 및 렌더링 기술의 새로운 지평을 연 핵심적인 진보다.


3DGUT의 핵심 기여는 두 가지로 요약할 수 있다.

첫째, 렌더링 파이프라인의 심장부에서 EWA 스플래팅을 언센티드 변환(UT)으로 대체함으로써, 3DGS의 근본적인 한계였던 비선형 카메라 모델 제약을 극복했다. 이로 인해 어안 렌즈와 같은 심한 왜곡을 가진 카메라나 롤링 셔터와 같은 동적 효과까지 래스터화의 효율성을 유지하면서 자연스럽게 처리할 수 있게 되었다. 이는 기술의 적용 범위를 이상적인 환경에서 벗어나 복잡하고 동적인 현실 세계로 확장시켰다는 점에서 의의가 크다.

둘째, 3DGRT와의 렌더링 공식 정렬을 통해 단일 3D 가우시안 씬 표현을 공유하는 하이브리드 렌더링 프레임워크 3DGRUT를 실현했다. 이는 래스터화의 압도적인 속도와 레이 트레이싱의 물리적 정확성을 결합하여, 실시간 성능과 사실적인 광학 효과(반사, 굴절)를 동시에 달성하는 길을 열었다.


3DGUT는 자율주행, 로보틱스, 디지털 트윈, VR/AR 등 실세계 데이터를 직접 다루는 다양한 산업 분야에서 3D 재구성 기술의 실용성을 한 단계 끌어올린 '게임 체인저'다. 저렴하고 일반적인 센서로 수집된 데이터를 직접 고품질 3D 에셋으로 변환하는 능력은 3D 콘텐츠 제작의 장벽을 낮추고, 데이터 수집 비용과 시간을 획기적으로 절감시킬 것이다.

궁극적으로 3DGUT와 3DGRUT는 NVIDIA의 Omniverse 생태계를 강화하고, 물리 세계와 가상 세계의 경계를 허무는 데 기여할 핵심 기술로 자리매김할 것이다. 이 기술은 3D 가우시안 스플래팅이 단순히 주어진 뷰를 보간하는 '뷰 합성 기술'을 넘어, 빛과 상호작용하고 물리 법칙을 따르는 '완전한 물리 기반 3D 월드 표현'으로 나아가는 중요한 이정표다. 향후 3DGUT는 오픈소스 커뮤니티와의 협력을 통해 안정성을 높이고 생태계를 확장하며, 실시간 고품질 3D 렌더링의 미래를 선도할 것으로 전망된다.


1. 3DGUT: Enabling Distorted Cameras and Secondary Rays in Gaussian Splatting - arXiv, 8월 20, 2025에 액세스, https://arxiv.org/html/2412.12507v1
2. Evaluating Fisheye-Compatible 3D Gaussian Splatting Methods on Real Images Beyond 180° Field of View - arXiv, 8월 20, 2025에 액세스, https://arxiv.org/html/2508.06968v1
3. 3DGUT: Enabling Distorted Cameras and Secondary Rays in Gaussian Splatting | Read Paper on Bytez, 8월 20, 2025에 액세스, https://bytez.com/docs/cvpr/33729/paper
4. 3DGUT: Enabling Distorted Cameras and Secondary Rays in ..., 8월 20, 2025에 액세스, https://research.nvidia.com/labs/toronto-ai/publication/2025_cvpr_3dgut/
5. 3D Gaussian Splatting Reconstruction Patterns Across Scene Attributes - DiVA, 8월 20, 2025에 액세스, https://kth.diva-portal.org/smash/get/diva2:1985707/FULLTEXT01.pdf
6. 3DGUT: Enabling Distorted Cameras and Secondary Rays in Gaussian Splatting - Research at NVIDIA, 8월 20, 2025에 액세스, https://research.nvidia.com/labs/toronto-ai/3DGUT/
7. Nerfstudio Integrates 3DGUT into gsplat - Radiance Fields, 8월 20, 2025에 액세스, https://radiancefields.com/nerfstudio-integrates-3dgut
8. 3DGUT: Enabling Distorted Cameras and Secondary Rays in Gaussian Splatting - arXiv, 8월 20, 2025에 액세스, https://arxiv.org/html/2412.12507v2
9. [Revisión de artículo] 3DGUT: Enabling Distorted Cameras and Secondary Rays in Gaussian Splatting - Moonlight, 8월 20, 2025에 액세스, https://www.themoonlight.io/es/review/3dgut-enabling-distorted-cameras-and-secondary-rays-in-gaussian-splatting
10. 3DGUT: Enabling Distorted Cameras and Secondary Rays in Gaussian Splatting, 8월 20, 2025에 액세스, https://www.researchgate.net/publication/387141344_3DGUT_Enabling_Distorted_Cameras_and_Secondary_Rays_in_Gaussian_Splatting
11. 3DGUT: Enabling Distorted Cameras and Secondary Rays in Gaussian Splatting - Research at NVIDIA, 8월 20, 2025에 액세스, https://research.nvidia.com/labs/toronto-ai/3DGUT/res/3DGUT_ready_main.pdf
12. CVPR Poster 3DGUT: Enabling Distorted Cameras and Secondary Rays in Gaussian Splatting, 8월 20, 2025에 액세스, https://cvpr.thecvf.com/virtual/2025/poster/33729
13. Rapidly Generate 3D Assets for Virtual Worlds with Generative AI | NVIDIA Technical Blog, 8월 20, 2025에 액세스, https://developer.nvidia.com/blog/rapidly-generate-3d-assets-for-virtual-worlds-with-generative-ai/
14. Generalized unscented transformation for forecasting non-Gaussian processes | Phys. Rev. E - Physical Review Link Manager, 8월 20, 2025에 액세스, https://link.aps.org/doi/10.1103/PhysRevE.111.054135
15. A more robust unscented transform - MITRE Corporation, 8월 20, 2025에 액세스, https://www.mitre.org/sites/default/files/pdf/vanzandt_unscented.pdf
16. A Generalized Unscented Transformation for Probability Distributions - PMC - PubMed Central, 8월 20, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC8043458/
17. nv-tlabs/3dgrut: Ray tracing and hybrid rasterization of Gaussian particles - GitHub, 8월 20, 2025에 액세스, https://github.com/nv-tlabs/3dgrut
18. 3D Gaussian Ray Tracing: Fast Tracing of Particle Scenes | Request PDF - ResearchGate, 8월 20, 2025에 액세스, https://www.researchgate.net/publication/385966448_3D_Gaussian_Ray_Tracing_Fast_Tracing_of_Particle_Scenes
19. Digital Twins: Precision of 3D Gaussian Splatting - Plain Concepts, 8월 20, 2025에 액세스, https://www.plainconcepts.com/digital-twins-3d-gaussian-splatting/
20. The Biggest Breakthrough in Gaussian Splatting - A 3DGRUT First Impressions and Tutorial, 8월 20, 2025에 액세스, https://www.youtube.com/watch?v=FI5JluMoBDE
21. NVIDIA to Open Source 3DGRT and 3DGUT - Radiance Fields, 8월 20, 2025에 액세스, https://radiancefields.com/nvidia-to-open-source-3dgrt-and-3dgut
22. Qualitative comparison of our novel-view synthesis results against the baselines on the MipNERF360 dataset [1]. - ResearchGate, 8월 20, 2025에 액세스, https://www.researchgate.net/figure/Qualitative-comparison-of-our-novel-view-synthesis-results-against-the-baselines-on-the_fig1_387141344
23. 3DGUT: Enabling Distorted Cameras and Secondary Rays in Gaussian Splatting Supplementary Material, 8월 20, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2025/supplemental/Wu_3DGUT_Enabling_Distorted_CVPR_2025_supplemental.pdf
24. AAA-Gaussians: Anti-Aliased and Artifact-Free 3D Gaussian Rendering - arXiv, 8월 20, 2025에 액세스, https://arxiv.org/html/2504.12811v2
25. Generative AI Solutions | Smarter AI Runs on NVIDIA, 8월 20, 2025에 액세스, https://www.nvidia.com/en-us/solutions/ai/generative-ai/
26. GET3D: A Generative Model of High Quality 3D Textured Shapes Learned from Images, 8월 20, 2025에 액세스, https://research.nvidia.com/labs/toronto-ai/GET3D/
27. Issues · nv-tlabs/3dgrut - GitHub, 8월 20, 2025에 액세스, https://github.com/nv-tlabs/3dgrut/issues
28. I captured my kitchen with 3DGRUT using 180 degree fisheye images : r/GaussianSplatting, 8월 20, 2025에 액세스, https://www.reddit.com/r/GaussianSplatting/comments/1k033rk/i_captured_my_kitchen_with_3dgrut_using_180/
29. 3DGRUT - Breakdown of the reconstruction after 7k step if passing any custom flags · Issue #694 · nerfstudio-project/gsplat - GitHub, 8월 20, 2025에 액세스, https://github.com/nerfstudio-project/gsplat/issues/694
30. NVIDIA Open Sources 3D Gaussian Ray Tracing and 3D Gaussian Unscented Transform, 8월 20, 2025에 액세스, https://radiancefields.com/nvidia-releases-3d-gaussian-ray-tracing-and-3d-gaussian-unscented-transform

