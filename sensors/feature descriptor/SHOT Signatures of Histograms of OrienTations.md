# SHOT (Signatures of Histograms of OrienTations)
[특징점 기술](./index.md)



3D 컴퓨터 비전의 여러 분야에서 핵심적인 과제 중 하나는 서로 다른 시점에서 얻은 3D 데이터를 정합(align)하거나, 장면 속에서 특정 객체를 인식하는 것이다. 이러한 과제를 해결하기 위해 점구름(point cloud) 데이터에서 추출된 특징점(keypoint) 주변의 지역적 기하학(local geometry) 정보를 압축된 수치 벡터로 표현하는 3D 형상 기술자(descriptor)가 필수적이다.1 기술자는 객체 인식, 3D 재구성, 로봇 공학, 자율 주행 등 다양한 고수준 응용의 근간을 이룬다.3 이상적인 기술자는 객체의 강체 변환(rigid transformation), 즉 회전과 이동에 대해 불변(invariant)해야 하며, 실제 환경에서 흔히 발생하는 노이즈, 일부가 가려지는 폐색(occlusion), 그리고 데이터 샘플링 밀도의 변화에 대해 강건(robust)한 특성을 지녀야 한다.6


SHOT이 등장하기 이전의 3D 기술자들은 크게 두 가지 패러다임으로 분류될 수 있었다: 시그니처(Signatures)와 히스토그램(Histograms).3

- **시그니처 (Signatures)**: 이 접근법은 특징점 주변에 지역 참조 프레임(Local Reference Frame, LRF)을 설정하고, 이 프레임을 기준으로 이웃 점들의 기하학적 속성(예: 좌표, 거리)을 인코딩한다. 시그니처 방식은 표면의 세부적인 구조를 정밀하게 묘사할 수 있어 높은 표현력(descriptiveness)을 갖지만, 데이터의 노이즈나 미세한 변형에 민감하게 반응하는 경향이 있다.6
- **히스토그램 (Histograms)**: 이 접근법은 특징점 주변 영역에서 특정 기하학적 속성(예: 점들 간의 각도, 거리)의 분포를 히스토그램으로 표현한다. 통계적 분포를 사용하므로 개별 점의 위치 변화나 노이즈에 덜 민감하여 높은 강건성(robustness)을 보인다. 그러나 이 과정에서 정밀한 공간적 정보가 일부 손실되어 표현력이 시그니처 방식보다 떨어질 수 있다.6

이 두 패러다임은 '표현력'과 '강건성' 사이의 근본적인 트레이드오프 관계를 형성했다. 연구자들은 응용 분야의 요구에 따라 두 가지 장점 중 하나를 선택해야 하는 딜레마에 직면했다. SHOT의 개발자들은 이 두 접근법이 상호 배타적인 것이 아니라, 현명하게 결합될 때 시너지를 낼 수 있다고 보았다. SHOT은 히스토그램의 강건성을 기반으로 하면서도, 시그니처처럼 공간적으로 지역화된 정보를 담기 위해 구조화된 그리드 내에서 여러 개의 지역 히스토그램을 계산하는 하이브리드 구조를 제안했다.8 이는 2D 이미지 특징 기술 분야에서 큰 성공을 거둔 SIFT(Scale-Invariant Feature Transform) 기술자의 철학, 즉 공간 그리드 위에서 방향성 히스토그램을 계산하는 방식을 3D 공간으로 확장한 것으로, 검증된 아이디어에 기반한 전략적 설계였다.8


이러한 배경 속에서 F. Tombari, S. Salti, L. Di Stefano는 2010년 유럽 컴퓨터 비전 학회(ECCV)에서 SHOT(Signatures of Histograms of OrienTations)을 처음 제안했다.9 그 이름은 '방향성 히스토그램들의 시그니처'를 의미하며, 알고리즘의 하이브리드 철학을 명확하게 드러낸다. SHOT은 매우 독창적이고 반복 가능한 LRF와 새로운 3D 기술자를 포함하는 포괄적인 표면 표현 방법을 제안함으로써, 3D 특징 기술 분야에서 중요한 성능 기준(benchmark)이자 연구의 이정표로 빠르게 자리 잡았다.3


SHOT 기술자의 계산 과정은 크게 세 단계로 구성된다: (1) 회전 불변성을 보장하는 로컬 참조 프레임(LRF)의 확립, (2) LRF를 기준으로 한 구형 공간 분할 및 지역 히스토그램 생성, (3) 최종 기술자 벡터 구성 및 정규화.


LRF는 기술자가 객체의 3D 회전에 영향을 받지 않도록(rotation invariance) 만드는 가장 핵심적인 요소이다.5 SHOT의 뛰어난 성능은 LRF의 고유성(uniqueness)과 반복성(repeatability)에 크게 의존한다. 즉, 동일한 표면에 대해 노이즈나 시점 변화가 있더라도 항상 거의 동일한 LRF가 계산되어야 한다.8 SHOT의 LRF 계산 과정은 다음과 같다.


특징점 `$\mathbf{p}$`를 중심으로 반경 `$R$` 내에 포함되는 모든 이웃점 `$\{\mathbf{p}_i\}$` 집합을 고려한다. 기존의 공분산 분석(Covariance Analysis, CA) 기반 LRF 방법들과 달리, SHOT은 중심점 `$\mathbf{p}$`로부터의 거리가 먼 점에 더 낮은 가중치를 부여하는 방식을 도입했다. 이는 지지 영역(support region)의 경계에 위치한 점들이 공분산 행렬 계산에 미치는 영향을 줄여, 노이즈와 점 밀도 변화에 대한 LRF의 강건성을 크게 향상시키는 핵심적인 개선점이다.8 가중 공분산 행렬 `$Cov(\mathbf{p})$`는 다음과 같이 정의된다:
$$
Cov(\mathbf{p}) = \frac{1}{\sum_{i=1}^{n} w_i} \sum_{i=1}^{n} w_i (\mathbf{p}_i - \bar{\mathbf{p}})(\mathbf{p}_i - \bar{\mathbf{p}})^T
$$
여기서 `$\bar{\mathbf{p}}$`는 가중치를 고려한 이웃점들의 중심(centroid)이며, `$w_i = R - ||\mathbf{p}_i - \mathbf{p}||$`는 중심점 `$\mathbf{p}$`와의 거리에 반비례하는 선형 가중치이다.14


계산된 공분산 행렬 `$Cov(\mathbf{p})$`에 대해 고유값 분해(Eigenvalue Decomposition, EVD)를 수행한다. 이를 통해 3개의 고유값(eigenvalues)과 그에 해당하는 3개의 직교하는 고유벡터(eigenvectors)를 얻는다. 이 고유벡터들이 LRF의 초기 축 후보(`$\mathbf{X}^+$`, `$\mathbf{Y}^+$`, `$\mathbf{Z}^+$`)가 된다. 고유벡터들은 고유값의 크기 순서대로 정렬된다.8


EVD를 통해 얻은 고유벡터는 방향(부호)이 모호하다. 예를 들어, `$\mathbf{X}^+$`가 유효한 고유벡터라면 `$-\mathbf{X}^+$` 또한 유효하다. 이 부호 모호성을 해결하지 않으면 동일한 표면에 대해서도 LRF가 180도 뒤집힐 수 있어 기술자의 일관성이 깨지고 매칭 성능이 급격히 저하된다.8 SHOT은 이 문제를 해결하기 위해 독창적인 부호 결정 방법을 사용한다. 각 축에 대해, 해당 축의 양의 방향과 음의 방향 중 어느 쪽으로 더 많은 이웃점 벡터(`$\mathbf{p}_i - \mathbf{p}$`)가 향하는지를 가중치 합으로 계산하여 최종 방향을 결정한다. 예를 들어, X축의 최종 방향 `$\mathbf{X}_{final}$`은 다음과 같이 결정된다 8:
$$
S_x^+ = \sum_{i: ||\mathbf{p}_i - \mathbf{p}|| \le R \land (\mathbf{p}_i - \mathbf{p}) \cdot \mathbf{X}^+ \ge 0} w_i
\\
S_x^- = \sum_{i: ||\mathbf{p}_i - \mathbf{p}|| \le R \land (\mathbf{p}_i - \mathbf{p}) \cdot (-\mathbf{X}^+) > 0} w_i
\\
\mathbf{X}_{final} = \begin{cases} \mathbf{X}^+ & \text{if } S_x^+ \ge S_x^- \\ -\mathbf{X}^+ & \text{otherwise} \end{cases}
$$
Z축(`$\mathbf{Z}_{final}$`)도 동일한 절차를 통해 부호 모호성을 해결한다. 마지막으로, Y축(`$\mathbf{Y}_{final}$`)은 오른손 좌표계를 형성하도록 외적(cross product)을 통해 `$\mathbf{Y}_{final} = \mathbf{Z}_{final} \times \mathbf{X}_{final}$`로 계산된다.8

이 LRF 설계는 SHOT의 성공에 결정적인 기여를 했다. 여기서 주목할 점은, 이 LRF가 순수한 기하학적 의미보다는 '반복성'이라는 실용적 목표에 극도로 최적화된 공학적 산물이라는 것이다. 예를 들어, 많은 LRF 방법들은 Z축을 표면 법선(surface normal)과 일치시키려 하지만, SHOT의 LRF에서 Z축(세 번째 고유벡터)은 법선과 반드시 일치하지 않을 수 있다.8 개발자들은 이것이 성능에 영향을 미치지 않는다고 명시적으로 밝히며, 중요한 것은 기하학적 해석이 아니라 어떤 상황(노이즈, 회전 등)에서도 항상 동일하게 계산되는 '매우 반복적인 직교 삼중항(highly repeatable triplet of orthogonal directions)'을 얻는 것이라고 강조했다.8 이는 SHOT이 이론적 순수성보다 실제 응용에서의 강건성을 우선시하는 실용주의적 설계 철학을 가지고 있음을 보여주며, 이 LRF는 SHOT의 성공을 이끈 가장 중요한 혁신 중 하나로 평가받는다.14


안정적인 LRF가 확립되면, 특징점 `$\mathbf{p}$`를 원점으로 하는 지역 좌표계가 정의된다. 이 좌표계를 기준으로 특징점 주변의 구형 지지 영역(spherical support)은 32개의 공간적 볼륨(volume)으로 분할된다.13 이 분할은 등방성 구면 그리드(isotropic spherical grid) 구조를 따르며, 구체적으로는 **방위각(azimuth) 8개 구역, 고도각(elevation) 2개 구역, 반경(radius) 2개 구역**으로 나뉜다 (`$8 \times 2 \times 2 = 32`).8

각 32개의 볼륨 내부에서 지역 히스토그램(local histogram)이 독립적으로 계산된다. 이 히스토그램이 담는 정보는 특징점의 법선 벡터 `$\mathbf{n}_u$`와 해당 볼륨 내에 위치한 이웃점의 법선 벡터 `$\mathbf{n}_v$` 사이의 각도 `$\theta_i$`이다. SHOT은 이 각도 자체를 사용하지 않고, 각도의 코사인 값, 즉 `$\cos(\theta_i)$`를 변수로 사용한다.8

`$\cos(\theta_i)$`를 사용하는 데는 두 가지 중요한 이유가 있다 8:

1. **빠른 계산**: 두 단위 벡터의 코사인 값은 벡터의 내적(dot product)과 같으므로, `$\cos(\theta_i) = \mathbf{n}_u \cdot \mathbf{n}_v$`로 매우 빠르게 계산할 수 있다.
2. **효과적인 비균등 양자화**: 코사인 값의 범위인 [-1, 1]을 균일한 간격으로 나누어 히스토그램 빈(bin)을 만들면, 원래 각도 `$\theta_i$`의 범위인 `$[0, \pi]$`에 대해서는 비균일한 간격으로 나뉘는 효과가 발생한다. 구체적으로, `$\theta_i$`가 `$\pi/2$`에 가까울수록(즉, 두 법선이 거의 수직일 때) 코사인 값의 변화가 크므로 더 세밀한 빈에 할당되고, `$\theta_i$`가 0이나 `$\pi$`에 가까울수록(두 법선이 거의 평행할 때) 코사인 값의 변화가 작아 더 넓은 빈에 할당된다. 이는 법선에 수직인 방향, 즉 표면의 곡률 변화가 가장 잘 드러나는 방향의 미세한 차이를 더 잘 포착하여 기술자의 표현력을 높이는 역할을 한다.

각 지역 히스토그램은 일반적으로 11개의 빈으로 구성된다.



한 점이 여러 공간 볼륨이나 히스토그램 빈의 경계에 위치할 경우, 미세한 위치 변화만으로도 완전히 다른 빈에 할당되어 기술자의 안정성을 해칠 수 있다. 이러한 경계 효과(boundary effects)를 완화하기 위해, SHOT은 사선형 보간법(quadrilinear interpolation)을 사용한다. 각 이웃점의 기여분은 그 점이 속한 공간 볼륨뿐만 아니라, 방위각, 고도각, 반경 방향으로 인접한 다른 볼륨들에도 분배된다. 마찬가지로 히스토그램에 기여할 때도 인접한 빈에 가중치를 두어 분배함으로써 부드러운 전환을 유도한다.8


32개의 각 공간 볼륨에서 계산된 11-bin 지역 히스토그램들을 순서대로 모두 연결(concatenate)하여 하나의 긴 벡터를 형성한다. 이렇게 생성된 최종 기술자의 차원은 다음과 같이 계산된다:
$$
32 \text{ (공간 볼륨)} \times 11 \text{ (히스토그램 빈)} = 352 \text{ 차원}
$$
이것이 Point Cloud Library (PCL)와 같은 주요 라이브러리에서 `SHOT352`라는 이름으로 알려진 표준 SHOT 기술자이다.18


마지막으로, 생성된 352차원의 벡터 전체에 대해 L2-norm 정규화를 수행하여 벡터의 크기를 1로 만든다. 이 정규화 과정은 점구름의 밀도 변화에 대한 강건성을 확보하는 데 중요한 역할을 한다. 예를 들어, 특정 영역의 점 밀도가 높아져 히스토그램의 전체적인 크기가 커지더라도, 정규화를 통해 상대적인 분포는 동일하게 유지된다. 또한, 이는 잠재적인 조명 변화에도 어느 정도의 불변성을 제공한다.8


SHOT의 기본 프레임워크는 유연하여 다양한 정보를 통합하도록 확장될 수 있다. 그중 가장 대표적인 확장은 색상 정보를 결합한 CSHOT과 계산 효율성을 극대화한 B-SHOT이다.



SHOT이 제안될 무렵, Microsoft Kinect와 같은 저렴한 RGB-D 센서가 대중화되기 시작했다. 이 센서들은 3D 깊이(형상) 정보와 함께 RGB 색상(텍스처) 정보를 제공했다.3 형상 정보만으로는 구분이 어려운 객체(예: 동일한 모양의 다른 색상 상자)나 텍스처가 풍부한 평면 등을 구별하기 위해 색상 정보를 활용하는 것은 자연스러운 발전 방향이었다. CSHOT(Color-SHOT)은 이러한 요구에 부응하여 기존의 형상 기반 SHOT 기술자에 색상 정보를 담은 히스토그램 시그니처를 추가로 결합한 형태이다.20


CSHOT의 계산 과정은 LRF 설정과 공간 분할까지는 SHOT과 동일하다. 그러나 32개의 각 공간 볼륨에서 형상 히스토그램(법선 벡터 간의 각도)을 계산하는 것과 별도로, 색상 히스토그램을 추가로 계산한다. 색상 히스토그램은 특징점의 색상 값(예: CIELab 색 공간에서의 값)과 이웃점의 색상 값 간의 차이를 기반으로 생성된다.20 CSHOT은 SHOT의 파라미터 외에 색상 히스토그램의 빈 개수를 결정하는 `Color Step` (`$SC$`)이라는 추가 파라미터를 가진다.20

형상 정보와 색상 정보가 결합되면서 기술자의 차원은 크게 증가한다. PCL 라이브러리의 `SHOTColorEstimation` 클래스가 생성하는 `SHOTColor1344` 기술자가 대표적인 예이다.7 이 1344차원은 표준 SHOT의 352차원 형상 정보에 각 볼륨마다 추가된 색상 히스토그램 정보가 더해져 구성된다. CSHOT은 형상만 사용하는 SHOT에 비해, 특히 텍스처가 풍부하거나 형상이 모호한 객체의 인식에서 더 높은 정확도를 보이는 것으로 입증되었다.6



표준 SHOT 기술자는 352개의 부동소수점(float) 값으로 구성되어 1408 바이트의 메모리를 차지한다. 수많은 특징점에 대해 기술자를 저장하고, 유클리드 거리(Euclidean distance)를 기반으로 매칭을 수행하는 것은 특히 실시간 처리가 요구되는 로보틱스나 모바일 응용에서 상당한 계산 부담이 될 수 있다. 이러한 문제를 해결하기 위해 SHOT의 이진(binary) 버전인 B-SHOT(Binary-SHOT)이 제안되었다.17 B-SHOT의 목표는 SHOT 기술자를 훨씬 작은 메모리 공간에 저장하고, 매우 빠른 속도로 매칭하는 것이다.

B-SHOT은 352차원의 실수 벡터인 SHOT을 352비트의 이진 벡터로 양자화(quantization)한다. 이 변환은 각 지역 히스토그램(11-bin)을 더 작은 하위 그룹으로 나누고, 각 그룹 내 값들의 분포와 합계에 따라 특정 이진 코드를 할당하는 복잡한 규칙 기반의 인코딩 방식을 통해 이루어진다.17


B-SHOT은 SHOT에 비해 메모리 사용량을 32배 줄이고(1408 바이트 vs 44 바이트), 특징 매칭 속도를 6배 향상시킨다. 이는 매칭 시 비용이 많이 드는 유클리드 거리 계산 대신, 비트 연산으로 매우 빠르게 계산할 수 있는 해밍 거리(Hamming distance)를 사용하기 때문이다.17

B-SHOT의 존재는 3D 기술자 설계에서 '정확도'와 '효율성' 간의 또 다른 중요한 트레이드오프를 보여준다. B-SHOT의 양자화 과정은 필연적으로 정보 손실을 유발한다. 예를 들어, 서로 다른 실수 값들이 동일한 이진 코드로 매핑되어 미세한 차이가 사라질 수 있다.17 이 정보 손실로 인해 B-SHOT은 SHOT보다 약간 더 적은 수의 정확한 대응점(correspondences)을 찾는 경향이 있다. 그러나 개발자들은 이 정도의 대응점만으로도 ICP(Iterative Closest Point)와 같은 후처리 알고리즘을 위한 대략적인 초기 변환을 추정하기에는 충분하다고 주장한다.17 이는 모든 응용이 최고의 정확도를 필요로 하는 것은 아니며, 실시간 SLAM이나 자원 제약이 있는 모바일 로봇과 같은 특정 작업에서는 속도와 메모리 효율성이 미세한 정확도 향상보다 더 중요할 수 있음을 시사한다.



SHOT은 제안 당시 최첨단(state-of-the-art) 지역 기술자들을 능가하는 종합적인 성능을 보였으며, 특히 노이즈, 클러터, 폐색이 존재하는 현실적인 시나리오에서 뛰어난 강점을 보였다.3 이는 고유하고 반복 가능한 LRF 설계와 히스토그램의 통계적 특성, 그리고 시그니처의 공간적 구별력을 결합한 하이브리드 구조 덕분이다. 여러 독립적인 비교 연구에서도 SHOT은 전반적으로 가장 우수한 성능을 보이는 기술자 중 하나로 지속적으로 평가되었으며, 특히 ground truth가 명확한 합성 데이터를 사용한 평가에서 높은 강건성을 입증했다.15


SHOT의 성능을 객관적으로 평가하기 위해, 동시대에 영향력이 컸던 다른 수작업 기술자들과 비교하는 것이 중요하다.

- **SHOT vs. FPFH (Fast Point Feature Histogram)**: FPFH는 특징점과 그 이웃점들 간의 기하학적 관계(법선 간 각도, 거리 등)를 계산하여 단순화된 히스토그램으로 표현한다. FPFH는 PFH의 계산 복잡도를 개선하여 속도가 빠르지만, SHOT보다 표현력이 낮고 노이즈에 덜 강건한 경향이 있다.15 가장 큰 차이점은 FPFH가 명시적인 LRF 없이 특징 자체의 불변성에 의존하는 반면, SHOT은 견고한 LRF를 통해 회전 불변성을 명시적으로 보장한다는 점이다. 그러나 두 기술자 모두 낮은 중첩률(low overlap)이나 큰 회전 차이가 있는 까다로운 시나리오에서는 성능이 저하되는 한계를 공유한다.3
- **SHOT vs. Spin Images**: Spin Image는 특징점 주변의 3D 점들을 2D 이미지(spin image) 형태의 히스토그램으로 변환하는 고전적인 기술자이다. 개념이 직관적이지만, 점 밀도 변화에 민감하고 폐색에 취약하다는 단점이 있다. SHOT은 안정적인 LRF와 최종 기술자 정규화 단계를 통해 이러한 문제에 대해 더 나은 강건성을 제공하도록 설계되었다.15

아래 표는 주요 3D 지역 기술자들의 핵심 특성을 요약하여 비교한 것이다. 이 표는 각 기술자의 장단점을 신속하게 파악하고, 특정 응용에 어떤 기술자가 더 적합할지 판단하는 데 도움을 준다.

| 특징                 | SHOT (Signature of Histograms of OrienTations)               | FPFH (Fast Point Feature Histogram)                         | Spin Image                                                   |
| -------------------- | ------------------------------------------------------------ | ----------------------------------------------------------- | ------------------------------------------------------------ |
| **기본 원리**        | LRF 기반 구형 그리드 내 지역 법선 방향 히스토그램들의 시그니처 8 | 점 쌍 간의 단순화된 기하학적 관계(각도, 거리) 히스토그램 24 | 특징점 축을 중심으로 회전시킨 2D 투영 평면에서의 점 밀도 히스토그램 |
| **LRF 사용**         | 예 (핵심 요소) 15                                            | 아니요                                                      | 아니요 (법선 벡터를 축으로 사용)                             |
| **주요 불변성**      | 회전, 이동 13                                                | 회전, 이동                                                  | 회전, 이동                                                   |
| **강건성 (노이즈)**  | 높음 15                                                      | 중간                                                        | 낮음                                                         |
| **강건성 (점 밀도)** | 높음 (L2 정규화 덕분) 8                                      | 중간 (가중치 부여) 17                                       | 낮음                                                         |
| **강건성 (폐색)**    | 중간                                                         | 낮음                                                        | 낮음                                                         |
| **기술자 차원**      | 352 (표준) 18                                                | 33 (표준)                                                   | 153 (일반적 설정)                                            |
| **계산 복잡도**      | 높음 (LRF 계산)                                              | 낮음                                                        | 중간                                                         |


SHOT의 뛰어난 성능과 강건성은 다양한 3D 컴퓨터 비전 및 로보틱스 응용 분야에서 성공적으로 활용되었다.


SHOT의 가장 대표적인 응용 분야는 장면 내에서 특정 객체 모델을 찾거나, 대규모 데이터베이스에서 질의 형상과 유사한 객체를 검색하는 것이다.3 이 과정은 일반적으로 (1) 장면과 모델에서 특징점 추출, (2) 각 특징점에 대해 SHOT 기술자 계산, (3) 기술자 간의 거리(예: 유클리드 거리)를 기반으로 대응점 매칭, (4) RANSAC과 같은 기법을 사용하여 잘못된 매칭을 제거하고 대응점 그룹화(correspondence grouping)를 통해 객체의 6자유도(6DOF) 포즈를 추정하는 파이프라인을 따른다.4


여러 시점에서 촬영된 단편적인 점구름들을 정렬하여 하나의 완전하고 일관된 3D 모델을 만드는 3D 재구성 과정에서 점구름 정합은 핵심적인 단계이다.3 SHOT은 서로 다른 점구름 간의 초기 대응점을 찾는 데 효과적으로 사용된다. SHOT으로 찾은 신뢰도 높은 대응점들을 기반으로 초기 변환 행렬을 추정한 뒤, ICP(Iterative Closest Point) 알고리즘을 적용하여 정합을 정밀하게 미세 조정하는 것이 표준적인 접근 방식이다.2


SHOT의 강건성과 효율성은 실시간 처리가 중요한 로보틱스 분야에서도 그 가치를 입증했다.

- **SLAM (Simultaneous Localization and Mapping)**: 대표적인 예로 'ORB-SHOT SLAM'이 있다. 이는 널리 사용되는 ORB-SLAM을 3D로 확장한 시스템으로, 로봇이 이전에 방문했던 장소를 다시 인식하여 누적된 궤적 오차를 수정하는 루프 클로징(loop closing) 과정에서 SHOT 기술자를 사용한다. 3D 점구름의 기하학적 정보를 활용하여 더 정확하고 강건한 장소 인식을 가능하게 한다.27
- **로봇 조작 (Robotic Manipulation)**: 로봇이 물체를 잡거나 옮기는 등의 조작 작업을 수행하기 위해서는 먼저 대상 객체를 정확히 인식하고 그 위치와 자세를 파악해야 한다. SHOT은 이러한 객체 인식 및 포즈 추정 모듈의 핵심 구성 요소로 활용될 수 있으며, 로봇이 비정형 환경에서 자율적으로 작업을 수행하는 데 필요한 시각적 인지 능력을 제공한다.28



SHOT은 수작업(handcrafted) 특징 기술의 정점에 있는 성과 중 하나이지만, 근본적인 한계 또한 가지고 있다. 인간의 경험과 기하학적 직관에 기반하여 설계된 알고리즘은 다음과 같은 까다로운 시나리오에서 성능이 크게 저하될 수 있다.16

- **극한 환경**: 두 점구름 간의 중첩 영역이 매우 작거나, 회전 차이가 극심할 때 대응점을 안정적으로 찾기 어렵다.3
- **센서 이질성**: 서로 다른 종류의 센서(예: LiDAR와 구조광 스캐너)로 촬영된 데이터는 점 밀도, 노이즈 패턴, 데이터 누락 방식이 매우 다르다. 이러한 이종(cross-source) 데이터 간의 정합은 수작업 기술자에게 매우 어려운 문제이다.30
- **설계의 경직성**: 알고리즘이 특정 유형의 기하학적 구조나 노이즈 모델을 가정하고 설계되었기 때문에, 예상치 못한 데이터나 매우 복잡한 실제 환경의 모든 변화에 유연하게 대응하기 어렵다.


최근 몇 년간 3D 컴퓨터 비전 분야는 딥러닝 기술의 발전으로 큰 변화를 겪었다. 딥러닝 기반 기술자는 수작업 기술자와 근본적으로 다른 접근 방식을 취한다.

- **표현 학습 (Representation Learning)**: 딥러닝 모델은 대규모 데이터셋으로부터 특징 표현 자체를 직접 학습한다. 이는 엔지니어가 수동으로 특징을 설계하는 과정을 대체하며, 종종 데이터에 더 최적화된, 강력한 표현을 찾아낸다.31 그 결과, 일반적으로 딥러닝 기반 방법들이 대규모 벤치마크에서 SHOT과 같은 수작업 기술자보다 우수한 성능을 보인다.24
- **단점**: 그러나 딥러닝 모델은 대규모의 정제된 학습 데이터가 필요하며, 학습 과정이 '블랙박스'처럼 작동하여 결과를 해석하기 어렵고, 막대한 계산 자원을 요구하는 단점이 있다.32


이러한 딥러닝의 부상 속에서 SHOT과 같은 고전적 기술자가 완전히 '대체'되었다고 보는 것은 단편적인 시각이다. 오히려 SHOT의 역할은 '재배치'되었으며, 오늘날에도 여전히 중요한 가치를 지닌다.

첫째, SHOT은 **강력한 성능 기준선(baseline)** 역할을 한다. 새로운 딥러닝 기반 기술자는 SHOT과 같은 검증된 고전적 방법보다 얼마나 뛰어난지를 입증해야 하며, SHOT은 여전히 수많은 연구에서 공정한 비교 대상으로 사용된다.3

둘째, **일반화 능력의 역설**이 존재한다. 일부 연구에서는 딥러닝 모델이 학습 데이터의 분포에서 멀어지는 미지의 환경(Out-of-Distribution)에 놓였을 때, 오히려 잘 설계된 수작업 특징이 더 나은 일반화 성능을 보일 수 있음을 시사한다.34 이는 학습 데이터가 부족하거나 예측 불가능한 환경에서는 SHOT이 더 신뢰성 있는 선택일 수 있음을 의미한다.

셋째, SHOT은 **하이브리드 시스템의 중요한 구성 요소**이자 **새로운 아키텍처의 영감**이 되고 있다. SHOT의 핵심 아이디어, 특히 견고한 LRF 계산 방식은 딥러닝 프레임워크에 통합되어 회전 불변성을 부여하는 데 사용된다.5 또한, ORB-SHOT SLAM과 같이 고전적 특징과 SHOT을 결합한 하이브리드 시스템은 각 기술의 장점을 극대화하는 성공적인 사례이다.27 복잡한 학습 기반 기술자의 지식을 더 빠르고 간단한 네트워크로 '증류(distill)'하는 연구에서도 SHOT의 원리가 여전히 유효함을 보여준다.5

결론적으로, SHOT은 단순히 과거의 유물이 아니라, 현재 진행형으로 3D 비전 분야에 영향을 미치고 있는 살아있는 기술이다.


SHOT(Signatures of Histograms of OrienTations)은 3D 지역 기술자 설계의 역사에서 중요한 이정표를 세운 알고리즘이다. '시그니처'의 높은 표현력과 '히스토그램'의 뛰어난 강건성이라는 두 패러다임의 장점을 성공적으로 결합함으로써, 3D 표면 정합 및 객체 인식의 성능을 한 단계 끌어올렸다.

SHOT의 성공은 무엇보다도 독창적이고 매우 반복 가능한 로컬 참조 프레임(LRF) 설계에 기인한다. 부호 모호성을 해결하고 노이즈에 강건한 LRF를 통해 완전한 회전 불변성을 달성했으며, 이는 이후 많은 연구에 깊은 영감을 주었다. 또한, CSHOT과 B-SHOT과 같은 확장을 통해 변화하는 센서 기술(RGB-D)과 응용 요구사항(실시간 처리)에 유연하게 대응하며 진화해왔다.

딥러닝 기술이 많은 영역에서 SHOT의 성능을 능가하는 시대를 맞이했지만, SHOT의 가치는 퇴색되지 않았다. 오늘날 SHOT은 새로운 알고리즘의 성능을 가늠하는 강력한 기준선으로, 데이터가 부족하거나 예측 불가능한 환경에서의 신뢰성 있는 대안으로, 그리고 더 발전된 하이브리드 시스템의 핵심 구성 요소로서 그 학술적, 실용적 명맥을 이어가고 있다. SHOT은 3D 컴퓨터 비전 분야에서 잘 설계된 기하학적 원리가 얼마나 강력하고 지속적인 영향력을 가질 수 있는지를 보여주는 대표적인 증거로 남아있다.


1. What is descriptor in computer vision? - Zilliz Vector Database, accessed July 24, 2025, https://zilliz.com/ai-faq/what-is-descriptor-in-computer-vision
2. 3D POINT CLOUD PROCESSING USING SPIN IMAGES FOR OBJECT DETECTION A Thesis Presented to the - ScholarWorks, accessed July 24, 2025, https://scholarworks.calstate.edu/downloads/fj236383k
3. SHOT: Unique Signatures of Histograms for Surface and Texture Description | Request PDF, accessed July 24, 2025, https://www.researchgate.net/publication/262111100_SHOT_Unique_Signatures_of_Histograms_for_Surface_and_Texture_Description
4. 3D Object Recognition based on Correspondence Grouping - Point Cloud Library 0.0 documentation - Compiling PCL, accessed July 24, 2025, https://pcl.readthedocs.io/projects/tutorials/en/master/correspondence_grouping.html
5. Distilling 3D distinctive local descriptors for 6D pose estimation - arXiv, accessed July 24, 2025, https://arxiv.org/html/2503.15106v1
6. SHOT: Unique signatures of histograms for surface and texture description - Bohrium, accessed July 24, 2025, https://www.bohrium.com/paper-details/shot-unique-signatures-of-histograms-for-surface-and-texture-description/813144549273632769-2613
7. PCL/OpenNI tutorial 4: 3D object recognition (descriptors) - robotica ..., accessed July 24, 2025, https://robotica.unileon.es/index.php?title=PCL/OpenNI_tutorial_4:_3D_object_recognition_(descriptors)
8. Unique Signatures of Histograms for Local Surface Description - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/221304551_Unique_Signatures_of_Histograms_for_Local_Surface_Description
9. fedassa/SHOT: C++ implementation of the SHOT 3D descriptor - GitHub, accessed July 24, 2025, https://github.com/fedassa/SHOT
10. pcl::SHOTEstimationOMP< PointInT, PointNT, PointOutT, PointRFT > Class Template Reference - Point Cloud Library, accessed July 24, 2025, http://pointclouds.org/documentation/classpcl_1_1_s_h_o_t_estimation_o_m_p.html
11. pcl::SHOTEstimation< PointInT, PointNT, PointOutT, PointRFT > Class Template Reference - the official ROS docs, accessed July 24, 2025, http://docs.ros.org/hydro/api/pcl/html/classpcl_1_1SHOTEstimation.html
12. (PDF) Fast 3D Keypoints Detector and Descriptor for View-Based 3D Objects Recognition, accessed July 24, 2025, https://www.researchgate.net/publication/281049939_Fast_3D_Keypoints_Detector_and_Descriptor_for_View-Based_3D_Objects_Recognition
13. Feature / PCL - adioshun, accessed July 24, 2025, https://adioshun.gitbooks.io/pcl/content/Tutorial/Feature/
14. SliceLRF: A Local Reference Frame Sliced along the Height on the 3D Surface - MDPI, accessed July 24, 2025, https://www.mdpi.com/1424-8220/23/7/3483
15. A Study on the Robustness of Shape Descriptors to Common Scanning Artifacts - MVA Organization, accessed July 24, 2025, https://www.mva-org.jp/Proceedings/2015USB/papers/14-26.pdf
16. A Novel HPNVD Descriptor for 3D Local Surface Description - MDPI, accessed July 24, 2025, https://www.mdpi.com/2227-7390/13/1/92
17. B-SHOT : A Binary Feature Descriptor for Fast and Efficient Keypoint Matching on 3D Point Clouds - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/280803085_B-SHOT_A_Binary_Feature_Descriptor_for_Fast_and_Efficient_Keypoint_Matching_on_3D_Point_Clouds
18. pcl/features/include/pcl/features/shot.h at master / PointCloudLibrary/pcl - GitHub, accessed July 24, 2025, https://github.com/PointCloudLibrary/pcl/blob/master/features/include/pcl/features/shot.h
19. pcl/features/shot.h Source File - Point Cloud Library, accessed July 24, 2025, https://pointclouds.org/documentation/shot_8h_source.html
20. A Combined Texture-Shape Descriptor for Enhanced ... - CiteSeerX, accessed July 24, 2025, https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=2c93afdbecd3029500cb47a0bcd73ed3aace01e7
21. (PDF) A combined texture-shape descriptor for enhanced 3D feature matching, accessed July 24, 2025, https://www.researchgate.net/publication/221127673_A_combined_texture-shape_descriptor_for_enhanced_3D_feature_matching
22. BRAND: A Robust Appearance and Depth Descriptor for RGB-D Images - UFMG, accessed July 24, 2025, https://homepages.dcc.ufmg.br/~erickson/publications/nascimento_iros2012.pdf
23. Establishment and Extension of a Fast Descriptor for Point Cloud Registration - MDPI, accessed July 24, 2025, https://www.mdpi.com/2072-4292/14/17/4346
24. Fast Point Feature Histograms (FPFH) for 3D registration | Request PDF - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/224557255_Fast_Point_Feature_Histograms_FPFH_for_3D_registration
25. 3D Object Recognition based on Correspondence Grouping - Point Cloud Library 0.0 documentation - Compiling PCL, accessed July 24, 2025, https://pcl.readthedocs.io/projects/tutorials/en/pcl-1.12.0/correspondence_grouping.html
26. SHOT descriptor computation for one point | Download Scientific Diagram - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/figure/SHOT-descriptor-computation-for-one-point_fig9_267390917
27. ORB-SHOT SLAM: Trajectory Correction by 3D Loop Closing Based on Bag-of-Visual-Words (BoVW) Model for RGB-D Visual SLAM - Fuji Technology Press, accessed July 24, 2025, https://www.fujipress.jp/jrm/rb/robot002900020365/
28. Amodal 3D Reconstruction for Robotic Manipulation via Stability and Connectivity - University of Washington, accessed July 24, 2025, https://homes.cs.washington.edu/~pedrod/papers/corl20.pdf
29. A Comprehensive Review on Handcrafted and Learning-Based Action Representation Approaches for Human Activity Recognition - MDPI, accessed July 24, 2025, https://www.mdpi.com/2076-3417/7/1/110
30. Learning a 3D descriptor for cross-source point cloud registration from synthetic data - ar5iv, accessed July 24, 2025, https://ar5iv.labs.arxiv.org/html/1708.08997
31. A Review on Deep Learning Techniques for 3D Sensed Data Classification - MDPI, accessed July 24, 2025, https://www.mdpi.com/2072-4292/11/12/1499
32. Deep Learning vs. Traditional Computer Vision - arXiv, accessed July 24, 2025, https://arxiv.org/pdf/1910.13796
33. Ensemble Learning, Deep Learning-Based and Molecular Descriptor-Based Quantitative Structure–Activity Relationships - MDPI, accessed July 24, 2025, https://www.mdpi.com/1420-3049/28/5/2410
34. Comparing Handcrafted Features and Deep Neural Representations for Domain Generalization in Human Activity Recognition, accessed July 24, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9572241/
35. Comparative Evaluation of Hand-Crafted and Learned Local Features - CVF Open Access, accessed July 24, 2025, https://openaccess.thecvf.com/content_cvpr_2017/papers/Schonberger_Comparative_Evaluation_of_CVPR_2017_paper.pdf
36. AI in Drug Discovery 2023 - A Highly Opinionated Literature Review (Part I), accessed July 24, 2025, http://practicalcheminformatics.blogspot.com/2024/01/ai-in-drug-discovery-2023-highly.html

