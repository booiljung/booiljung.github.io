# 원형 그리드 패턴을 이용한 Zhang의 캘리브레이션

## 1. 서론

### 0.1 카메라 캘리브레이션의 중요성과 응용 분야

카메라 캘리브레이션(Camera Calibration)은 카메라의 물리적, 기하학적 특성을 정밀하게 측정하고 모델링하는 과정을 의미한다. 이 과정의 본질은 카메라가 획득한 2차원 이미지와 실제 3차원 세계 사이의 관계를 수학적으로 정립하는 데 있다.1 카메라 렌즈와 이미지 센서는 이상적인 핀홀(pinhole) 모델과 달리 고유한 왜곡을 포함하고 있으며, 제조상의 미세한 차이로 인해 모든 카메라가 서로 다른 특성을 가진다. 캘리브레이션은 이러한 카메라 내부 파라미터(intrinsic parameters)와 렌즈 왜곡 계수(distortion coefficients)를 추정하여, 이미지에 나타나는 왜곡을 보정하고 2차원 픽셀 좌표를 3차원 공간 좌표로, 혹은 그 역으로 정확하게 변환할 수 있는 기반을 제공한다.1

정확한 카메라 파라미터는 현대 컴퓨터 비전 기술의 근간을 이루며, 다양한 첨단 기술 분야에서 필수적인 전처리 단계로 자리 잡았다. 대표적인 응용 분야는 다음과 같다.

- **3차원 재구성 (3D Reconstruction)**: 여러 시점에서 촬영된 2D 이미지들로부터 대상의 3차원 형상을 복원하는 기술로, 정확한 카메라 파라미터는 각 이미지의 상대적인 위치와 방향을 파악하고, 이미지 점들을 3D 공간에 정확히 위치시키는 데 결정적인 역할을 한다.4
- **증강 현실 (Augmented Reality, AR)**: 현실 세계의 이미지 위에 가상의 3D 객체를 자연스럽게 겹쳐 보이게 하려면, 현실과 가상 공간의 좌표계를 정밀하게 일치시켜야 한다. 캘리브레이션은 이러한 정합(registration)의 정확도를 보장하여 사용자에게 이질감 없는 AR 경험을 제공한다.4
- **로보틱스 및 자율주행 (Robotics and Autonomous Driving)**: 로봇 팔이 물체를 정확하게 집거나(bin-picking), 자율주행차가 주변 환경을 인식하고 거리를 측정하기 위해서는 비전 센서가 제공하는 정보를 실제 세계의 미터 단위로 변환해야 한다. 캘리브레이션은 이러한 '눈-손(eye-hand)' 또는 '눈-세상(eye-to-world)' 좌표계 변환의 정확도를 담보한다.6
- **정밀 계측 (Metrology)**: 머신 비전 시스템을 이용해 부품의 크기나 결함을 검사하는 산업 현장에서, 픽셀 단위의 측정값을 실제 물리적 단위(mm, μm)로 환산하기 위해 정밀한 캘리브레이션이 선행되어야 한다.3

### 0.2 Zhang의 평면 기반 캘리브레이션 방법론의 의의

과거의 카메라 캘리브레이션 기법은 크게 두 가지 범주로 나눌 수 있었다. 첫째는 **포토그래메트릭 캘리브레이션(photogrammetric calibration)**으로, 3차원 공간상의 위치가 매우 정밀하게 알려진 캘리브레이션 장비(예: 서로 직교하는 두세 개의 평면)를 관찰하여 파라미터를 추정하는 방식이다. 이 방법은 높은 정확도를 보장하지만, 고가의 장비와 정교한 설치 환경을 요구하여 일반적인 사용자가 접근하기 어려웠다.10 둘째는 **자가 캘리브레이션(self-calibration)**으로, 별도의 캘리브레이션 패턴 없이 정적인 장면을 카메라로 움직이며 촬영한 이미지들로부터 카메라 파라미터를 추정하는 방식이다. 유연성은 높지만, 추정해야 할 파라미터가 많아 결과의 안정성과 강인성이 부족한 경우가 많았다.10

2000년에 발표된 Zhengyou Zhang의 방법론은 이 두 접근법의 간극을 메우며 캘리브레이션 기술의 대중화를 이끈 혁신적인 기법으로 평가받는다.10 Zhang의 방법은 고가의 3D 장비 대신, 프린터로 쉽게 제작할 수 있는 

**평면 패턴(planar pattern)** 하나만을 사용한다. 사용자는 이 평면 패턴을 서로 다른 여러 각도와 위치에서 촬영하기만 하면 된다. 이 방식은 3D 장비 없이 2D 계측 정보만을 활용하면서도, 자가 캘리브레이션보다 훨씬 높은 정확도와 강인성을 달성했다.10 특별한 전문 지식 없이도 누구나 손쉽게 캘리브레이션을 수행할 수 있게 만든 이 유연성과 실용성 덕분에, Zhang의 방법은 컴퓨터 비전 연구실을 넘어 다양한 산업 현장과 실제 응용 분야로 3D 비전 기술이 확산되는 데 결정적인 기여를 했다.

### 0.3 보고서의 목적: 원형 그리드 패턴 사용에 대한 심층적 고찰

Zhang의 방법론은 발표 초기부터 지금까지 주로 **체커보드(checkerboard)** 패턴을 표준처럼 사용해왔다. 체커보드의 코너는 검출이 용이하고 투영 변환 시 편향이 적다는 장점이 있기 때문이다. 그러나 특정 응용 분야, 특히 고정밀 계측이나 블러(blur)가 심한 환경에서는 **원형 그리드(circular grid)** 패턴이 잠재적으로 더 나은 성능을 보일 수 있다는 가능성이 제기되어 왔다.5 원의 중심점은 이론적으로 다수의 픽셀 정보를 종합하여 결정되므로 노이즈에 강인하고 더 높은 아픽셀(sub-pixel) 정밀도를 가질 수 있다.

본 보고서는 Zhang의 캘리브레이션 원리를 근간으로 하되, 캘리브레이션 패턴으로 **원형 그리드**를 사용하는 경우에 발생하는 이론적, 실용적 문제들을 심층적으로 고찰하는 것을 목표로 한다. 특히, 원형 그리드 사용 시 발생하는 고질적인 문제인 **'투영 중심 편향(projection center bias)'** 현상을 집중적으로 분석할 것이다. 이 편향은 원근 투영(perspective projection)과 렌즈 왜곡(lens distortion)에 의해 발생하며, 원형 그리드의 이론적 정밀도를 심각하게 저해하는 주요 원인이다.

따라서 본 보고서는 이 편향 문제의 수학적 원인을 파헤치고, 이를 해결하기 위한 최신 연구 동향, 특히 2024년 CVPR 등에서 발표된 **'비편향 추정기(unbiased estimator)'** 기술을 상세히 다룰 것이다.5 이를 통해 원형 그리드 패턴의 올바른 사용법과 그 잠재력을 최대한 활용하기 위한 기술적 통찰을 제공하고자 한다.

## 1.  카메라 모델의 수학적 원리

카메라 캘리브레이션을 이해하기 위해서는 먼저 카메라가 3차원 세계를 2차원 이미지로 변환하는 과정을 수학적으로 모델링하는 방법을 알아야 한다. 이 모델은 여러 좌표계 간의 변환과 카메라 고유의 특성, 그리고 렌즈의 물리적 한계로 인한 왜곡을 포함하는 계층적 구조를 가진다.

### 1.1 핀홀 카메라 모델과 투영 변환

가장 단순하고 이상적인 카메라 모델은 **핀홀 카메라 모델(pinhole camera model)**이다.3 이 모델은 렌즈 없이 아주 작은 구멍(pinhole)을 통해 빛이 들어와 반대편의 이미지 평면(image plane)에 상을 맺는 원리를 기반으로 한다. 3차원 공간의 한 점 $M =^T$가 핀홀(카메라 원점)을 지나 이미지 평면에 투영되는 과정은 기하학적으로 유사 삼각형(similar triangles) 원리를 통해 설명할 수 있다.14

카메라의 광학 중심(optical center)을 원점으로 하고, 광축(optical axis)을 Z축으로 하는 카메라 좌표계에서, 초점 거리(focal length)가 $f$일 때, 3D 점 $M =^T$는 이미지 평면 위의 점 $m = [x, y]^T$에 다음과 같이 투영된다 16:
$$
x = f \frac{X_c}{Z_c}, \quad y = f \frac{Y_c}{Z_c}
$$
이 관계는 깊이 정보 $Z_c$에 의한 나눗셈 때문에 비선형적이다. 컴퓨터 비전에서는 이러한 비선형 변환을 선형 대수적으로 다루기 위해 **동차 좌표계(homogeneous coordinates)**를 도입한다.18 2D 점 $[u, v]^T$는 $[u, v, 1]^T$로, 3D 점 $^T$는 $^T$로 표현된다. 동차 좌표계를 사용하면, 3D 점 $\tilde{M}$과 2D 이미지 점 $\tilde{m}$ 사이의 투영 변환(projective transformation)을 다음과 같은 단일 행렬 곱으로 표현할 수 있다 10:
$$
s \tilde{m} = P \tilde{M}
$$
여기서 $s$는 임의의 0이 아닌 스케일 팩터(scale factor)이며, $P$는 3x4 크기의 **카메라 투영 행렬(camera projection matrix)**이다. 이 행렬 $P$는 카메라의 모든 기하학적 정보를 담고 있으며, 내부 파라미터와 외부 파라미터의 조합으로 구성된다.

### 1.2 좌표계 시스템: 월드, 카메라, 이미지, 픽셀

캘리브레이션 과정은 여러 좌표계 간의 변환 관계를 규명하는 과정이며, 주로 다음 네 가지 좌표계가 사용된다 19:

1. **월드 좌표계 (World Coordinate System, $O_w-X_wY_wZ_w$)**: 3차원 공간에 대한 절대적인 기준을 제공하는 좌표계이다. 사용자가 문제에 맞게 임의로 설정할 수 있으며, 예를 들어 캘리브레이션 패턴의 한쪽 모서리를 원점으로 삼을 수 있다.19
2. **카메라 좌표계 (Camera Coordinate System, $O_c-X_cY_cZ_c$)**: 카메라의 광학 중심을 원점으로 하는 3차원 좌표계이다. Z축은 일반적으로 카메라가 바라보는 방향인 광축과 일치하며, X축과 Y축은 이미지 평면과 나란하다.19
3. **이미지 평면 좌표계 (Image Plane Coordinate System, $o-xy$)**: 카메라 좌표계의 Z축에 수직인 2D 평면으로, 물리적인 단위(예: mm)로 좌표가 표현된다. 카메라 내부 파라미터의 영향을 제거한 **정규화된 이미지 좌표(normalized image coordinates)**라고도 불리며, 초점 거리가 1인 가상의 평면으로 생각할 수 있다.19
4. **픽셀 좌표계 (Pixel Coordinate System, $o'-uv$)**: 디지털 이미지에서 픽셀 단위로 위치를 나타내는 2D 좌표계이다. 일반적으로 이미지의 좌측 상단 모서리를 원점으로 하며, u축은 오른쪽으로, v축은 아래쪽으로 증가한다.19

캘리브레이션의 최종 목표는 월드 좌표계의 3D 점이 어떤 변환 과정을 거쳐 픽셀 좌표계의 2D 점으로 나타나는지를 완벽하게 모델링하는 것이다. 이 과정은 크게 월드 -->> 카메라 -->> 이미지 -->> 픽셀 좌표계 순으로 변환된다.

### 1.3 카메라 내부 파라미터 (Intrinsic Parameters)

내부 파라미터는 카메라 자체의 고유한 광학적, 기하학적 특성을 나타내며, 3D 카메라 좌표계에서 2D 픽셀 좌표계로의 변환을 정의한다.1 이 파라미터들은 한 번 캘리브레이션되면 카메라가 바뀌지 않는 한 변하지 않는다. 내부 파라미터는 3x3 크기의 **카메라 행렬(Camera Matrix) $K$**로 표현된다.3
$$
K = \begin{bmatrix} f_x & \gamma & c_x \\ 0 & f_y & c_y \\ 0 & 0 & 1 \end{bmatrix}
$$
각 파라미터의 의미는 다음과 같다.

- **초점 거리 ($f_x, f_y$)**: 렌즈의 초점 거리 $F$를 픽셀 단위로 환산한 값이다. $f_x = F \cdot m_x$와 $f_y = F \cdot m_y$로 계산되며, 여기서 $m_x$와 $m_y$는 각각 이미지의 u축과 v축 방향으로 단위 길이(예: mm)당 픽셀 수를 나타내는 스케일 팩터이다.3 이미지 센서의 물리적인 셀(cell)이 완벽한 정사각형이 아닐 경우 $f_x$와 $f_y$는 다른 값을 가질 수 있다.1
- **주점 ($c_x, c_y$)**: 광축이 이미지 센서와 만나는 점, 즉 주점(principal point)의 픽셀 좌표이다. 이상적으로는 이미지의 정중앙에 위치해야 하지만, 카메라 조립 과정에서 렌즈와 이미지 센서가 미세하게 어긋나 중심에서 벗어날 수 있다.1
- **비대칭 계수 ($\gamma$, skew coefficient)**: 이미지 센서 셀 배열의 y축이 기울어진 정도를 나타낸다. 현대의 디지털 카메라는 제조 공정이 매우 정밀하여 이 값이 거의 0에 가깝기 때문에, 대부분의 캘리브레이션 모델에서는 $\gamma=0$으로 가정하고 무시한다.3

### 1.4 카메라 외부 파라미터 (Extrinsic Parameters)

외부 파라미터는 월드 좌표계에 대한 카메라의 3차원 공간상 위치(position)와 방향(orientation)을 정의한다. 이는 월드 좌표계에서 정의된 점을 카메라 좌표계로 변환하는 강체 변환(rigid transformation)에 해당한다.1 외부 파라미터는 카메라가 움직이거나 월드 좌표계의 기준이 바뀌면 함께 변한다.21

외부 파라미터는 3x3 **회전 행렬(Rotation Matrix) $R$**과 3x1 **이동 벡터(Translation Vector) $t$**로 구성된다. 월드 좌표계의 점 $M_w =^T$는 다음 식을 통해 카메라 좌표계의 점 $M_c =^T$로 변환된다.
$$
M_c = R \cdot M_w + t
$$
회전 행렬 $R$은 3개의 회전축(roll, pitch, yaw)에 대한 정보를 담고 있으며, 이동 벡터 $t$는 월드 좌표계의 원점에서 카메라 좌표계의 원점까지의 변위를 나타낸다. 이 둘을 합쳐 $$로 표기하며, 3개의 회전과 3개의 이동으로 총 6 자유도(6-DoF)를 가진다.1

### 1.5 렌즈 왜곡 모델 (Lens Distortion Model)

이상적인 핀홀 모델은 렌즈가 없다고 가정하지만, 실제 카메라는 렌즈를 사용하여 빛을 모으기 때문에 다양한 형태의 왜곡이 발생한다. 이 왜곡은 이미지의 기하학적 정확성을 떨어뜨리는 주된 요인으로, 특히 이미지의 가장자리에서 두드러지게 나타난다.3 캘리브레이션 과정에서는 이러한 왜곡을 모델링하고 보정하기 위한 왜곡 계수(distortion coefficients)를 함께 추정한다. 왜곡은 주로 방사 왜곡과 접선 왜곡으로 나뉜다.

- **방사 왜곡 (Radial Distortion)**: 렌즈의 곡률 때문에 발생하는 왜곡으로, 렌즈 중심에서 멀어질수록 빛이 더 많이 휘어져 발생한다. 이로 인해 직선이 바깥쪽으로 휘는 **배럴 왜곡(barrel distortion, $k_1 > 0$)**이나 안쪽으로 휘는 **핀쿠션 왜곡(pincushion distortion, $k_1 < 0$)**이 나타난다.3 왜곡 보정은 정규화된 이미지 좌표 $(x_n, y_n)$에 왜곡 계수 $k_1, k_2, k_3, \dots$를 적용하여 수행된다. 왜곡된 좌표 $(x_d, y_d)$는 다음과 같이 모델링된다 3:
  $$
  \begin{aligned}
  x_d &= x_n (1 + k_1 r^2 + k_2 r^4 + k_3 r^6 + \dots) \\
  y_d &= y_n (1 + k_1 r^2 + k_2 r^4 + k_3 r^6 + \dots)
  \end{aligned}
  $$
  여기서 $r^2 = x_n^2 + y_n^2$ 이다. 일반적으로 $k_1, k_2$ 두 개의 계수만으로도 충분하지만, 어안 렌즈(fisheye lens)와 같이 왜곡이 심한 경우 $k_3$까지 사용하기도 한다.3

- **접선 왜곡 (Tangential Distortion)**: 렌즈와 이미지 센서가 완벽하게 평행하게 조립되지 않았을 때 발생하는 왜곡이다.3 이 왜곡은 이미지를 특정 방향으로 기울어지게 만든다. 접선 왜곡은 두 개의 계수 $p_1, p_2$를 사용하여 모델링된다 3:
  $$
  \begin{aligned}
  x_d &= x_n + [2 p_1 x_n y_n + p_2(r^2 + 2x_n^2)] \\
  y_d &= y_n + [p_1(r^2 + 2y_n^2) + 2 p_2 x_n y_n]
  \end{aligned}
  $$
  이처럼 카메라 모델은 이상적인 기하학적 투영(핀홀 모델)에서 시작하여, 실제 디지털 카메라의 제조 특성(내부 파라미터), 공간적 배치(외부 파라미터), 그리고 렌즈의 물리적 한계(왜곡 모델)를 순차적으로 반영하는 계층적 구조를 갖는다. 캘리브레이션은 바로 이 모델의 모든 파라미터를 역으로 추정하여 카메라의 '정체'를 밝히는 과정이라 할 수 있다.

## 2.  Zhang의 평면 기반 캘리브레이션 알고리즘

Zhang의 방법은 복잡한 3차원 캘리브레이션 장비 없이, 누구나 쉽게 만들 수 있는 평면 패턴을 이용하여 카메라의 내부 및 외부 파라미터, 그리고 렌즈 왜곡 계수까지 정확하게 추정할 수 있는 획기적인 절차를 제시한다. 그 핵심에는 평면이라는 기하학적 제약을 통해 복잡한 투영 문제를 선형 대수 문제로 변환하는 아이디어가 있다.

### 2.1 핵심 원리: 평면 패턴과 호모그래피

Zhang 방법의 가장 중요한 전제는 캘리브레이션에 사용되는 모든 3D 특징점들이 하나의 평면 위에 존재한다는 것이다.10 월드 좌표계를 이 평면 위에 설정하여 모든 특징점의 Z 좌표를 0으로 만들면($Z_w=0$), 3D 월드 좌표와 2D 이미지 좌표 사이의 투영 관계가 크게 단순화된다.

3차원 점 $M =^T$와 그에 대응하는 이미지 점 $m = [u, v]^T$의 동차좌표 $\tilde{M} =^T$와 $\tilde{m} = [u, v, 1]^T$ 사이의 일반적인 투영 관계는 다음과 같다:
$$
s \tilde{m} = K \tilde{M} = K \begin{bmatrix} \mathbf{r}_1 & \mathbf{r}_2 & \mathbf{r}_3 & \mathbf{t} \end{bmatrix} \begin{bmatrix} X \\ Y \\ Z \\ 1 \end{bmatrix}
$$
여기서 $R$의 열 벡터를 $\mathbf{r}_1, \mathbf{r}_2, \mathbf{r}_3$라고 하자. 만약 모든 점이 $Z=0$ 평면에 있다면, 이 식은 다음과 같이 단순화된다 10:
$$
s \begin{bmatrix} u \\ v \\ 1 \end{bmatrix} = K \begin{bmatrix} \mathbf{r}_1 & \mathbf{r}_2 & \mathbf{r}_3 & \mathbf{t} \end{bmatrix} \begin{bmatrix} X \\ Y \\ 0 \\ 1 \end{bmatrix} = K \begin{bmatrix} \mathbf{r}_1 & \mathbf{r}_2 & \mathbf{t} \end{bmatrix} \begin{bmatrix} X \\ Y \\ 1 \end{bmatrix}
$$
이 식은 3D 공간의 평면 위 점($^T$)과 2D 이미지 평면의 점($[u, v, 1]^T$) 사이의 관계가 3x3 행렬에 의해 선형적으로 매핑됨을 보여준다. 이 3x3 변환 행렬을 **호모그래피 행렬(Homography Matrix) $H$**라고 정의한다.30
$$
H = \lambda K \begin{bmatrix} \mathbf{r}_1 & \mathbf{r}_2 & \mathbf{t} \end{bmatrix}
$$
여기서 $\lambda$는 임의의 스케일 팩터이다. 즉, 평면 패턴을 사용하면 복잡한 3D-2D 투영 문제를 훨씬 다루기 쉬운 2D-2D 호모그래피 문제로 치환할 수 있다.

### 2.2 호모그래피 행렬 H 추정

호모그래피 행렬 $H$는 9개의 원소를 가지지만, 스케일에 대해 불변이므로 8개의 자유도(Degrees of Freedom, DoF)를 가진다.31 하나의 3D-2D 점 대응 쌍은 $H$의 원소에 대한 두 개의 독립적인 선형 방정식을 제공한다. 따라서 최소 4개의 점 대응 쌍이 있으면 $H$를 유일하게 결정할 수 있다.28

실제로는 노이즈의 영향을 줄이기 위해 4개보다 많은 점(예: 체커보드의 모든 코너)을 사용한다. 각 점 대응 $(X_i, Y_i) \leftrightarrow (u_i, v_i)$로부터 얻은 여러 개의 선형 방정식을 모아 $Ah=0$ 형태의 동차 선형 시스템(homogeneous linear system)을 구성한다. 여기서 $h$는 $H$ 행렬의 9개 원소를 일렬로 나열한 벡터이다. 이 시스템은 **Direct Linear Transform (DLT)** 알고리즘을 통해 풀 수 있으며, 일반적으로는 특이값 분해(Singular Value Decomposition, SVD)를 이용하여 $A^TA$의 최소 고유값에 해당하는 고유벡터를 구함으로써 $h$의 해를 찾는다.28

### 2.3 단일 호모그래피로부터의 제약 조건 도출

추정된 호모그래피 $H = [\mathbf{h}_1, \mathbf{h}_2, \mathbf{h}_3]$는 카메라의 내부 파라미터 $K$와 외부 파라미터 $R, \mathbf{t}$와 다음과 같은 관계를 가진다:
$$
[\mathbf{h}_1, \mathbf{h}_2, \mathbf{h}_3] = \lambda K [\mathbf{r}_1, \mathbf{r}_2, \mathbf{t}]
$$
여기서 $\mathbf{h}_1, \mathbf{h}_2, \mathbf{h}_3$는 $H$의 열 벡터이다. 이 식으로부터 다음 관계를 유도할 수 있다:
$$
\mathbf{r}_1 = \lambda^{-1} K^{-1} \mathbf{h}_1 \quad \text{and} \quad \mathbf{r}_2 = \lambda^{-1} K^{-1} \mathbf{h}_2
$$
Zhang의 핵심적인 통찰은 회전 행렬 $R$의 열 벡터들이 가지는 기하학적 속성, 즉 서로 직교(orthonormal)한다는 사실을 이용하는 것이다.10

1. $\mathbf{r}_1$과 $\mathbf{r}_2$는 서로 직교한다: $\mathbf{r}_1^T \mathbf{r}_2 = 0$
2. $\mathbf{r}_1$과 $\mathbf{r}_2$의 크기는 1이다: $\|\mathbf{r}_1\| = \|\mathbf{r}_2\| = 1$, 즉 $\mathbf{r}_1^T \mathbf{r}_1 = \mathbf{r}_2^T \mathbf{r}_2 = 1$

이 두 가지 기하학적 진실을 호모그래피의 열 벡터와 내부 파라미터 $K$에 대한 대수적 제약 조건으로 변환할 수 있다 10:
$$
\begin{aligned}
\mathbf{h}_1^T K^{-T} K^{-1} \mathbf{h}_2 &= 0 \\
\mathbf{h}_1^T K^{-T} K^{-1} \mathbf{h}_1 &= \mathbf{h}_2^T K^{-T} K^{-1} \mathbf{h}_2
\end{aligned}
$$
이 두 방정식은 한 장의 평면 패턴 이미지(즉, 하나의 호모그래피)로부터 얻을 수 있는, 카메라 내부 파라미터에 대한 근본적인 제약 조건이다.

### 2.4 다중 이미지를 이용한 내부 파라미터 행렬 K 해석적 해법

위의 제약 조건을 더 다루기 쉬운 형태로 만들기 위해, 새로운 행렬 $B$를 다음과 같이 정의한다 29:
$$
B = K^{-T} K^{-1} = \begin{bmatrix} B_{11} & B_{12} & B_{13} \\ B_{12} & B_{22} & B_{23} \\ B_{13} & B_{23} & B_{33} \end{bmatrix}
$$
$K$가 상삼각행렬(upper triangular matrix)이므로 $K^{-1}$도 상삼각행렬이고, 따라서 $B$는 대칭 행렬(symmetric matrix)이다. $B$는 6개의 독립적인 미지수($B_{11}, B_{12}, B_{22}, B_{13}, B_{23}, B_{33}$)로 결정된다. 이 미지수들을 벡터 $\mathbf{b}$로 표현하자: $\mathbf{b} =^T$.

$B$를 이용하면 앞서 유도한 두 제약 조건은 $\mathbf{b}$에 대한 두 개의 동차 선형 방정식으로 표현될 수 있다. $i$번째 호모그래피의 $j$번째 열 벡터를 $\mathbf{h}_{ij}$라 할 때, $v_{ij}^T \mathbf{b} = \mathbf{h}_i^T B \mathbf{h}_j$ 관계를 이용하여 다음과 같이 쓸 수 있다 30:
$$
\begin{bmatrix} v_{12}^T \\ (v_{11} - v_{22})^T \end{bmatrix} \mathbf{b} = \mathbf{0}
$$
한 장의 이미지는 $\mathbf{b}$에 대한 두 개의 방정식을 제공한다. $\mathbf{b}$는 6개의 미지수를 가지므로, 스케일을 제외하고 유일한 해를 구하기 위해서는 최소 3개의 독립적인 방정식 쌍, 즉 **최소 3장**의 서로 다른 방향에서 촬영한 평면 패턴 이미지가 필요하다.10 만약 비대칭 계수 $\gamma=0$과 같은 추가 제약이 있다면 2장만으로도 충분하다.18

$n$장의 이미지로부터 $n$개의 호모그래피를 추정하고, 각각으로부터 2개의 선형 방정식을 얻어 총 $2n \times 6$ 크기의 행렬 $V$를 구성하여 동차 선형 시스템 $V\mathbf{b}=0$을 만든다. 이 시스템의 해 $\mathbf{b}$는 $V^TV$의 가장 작은 고유값에 해당하는 고유벡터를 찾음으로써 구할 수 있다.29

일단 벡터 $\mathbf{b}$가 구해지면, 이를 다시 대칭 행렬 $B$로 변환한다. 마지막으로, $B = K^{-T} K^{-1}$ 관계와 $K$가 상삼각행렬이라는 사실을 이용하여 $B$로부터 내부 파라미터 $f_x, f_y, c_x, c_y, \gamma$를 해석적으로 계산할 수 있다. 이 과정은 Cholesky 분해와 유사한 방식으로 수행된다.30

### 2.5 최대우도추정 기반 비선형 최적화

앞선 과정으로 구한 해석적 해는 이미지의 노이즈나 렌즈 왜곡을 전혀 고려하지 않았기 때문에 완벽하지 않다. 따라서 이 해는 더 정확한 값을 찾기 위한 **초기 추정치(initial guess)**로 사용된다.10

캘리브레이션의 최종 단계는 모든 파라미터(5개의 내부 파라미터, 각 이미지에 대한 6개의 외부 파라미터, 그리고 왜곡 계수들)를 동시에 최적화하여 모델의 예측과 실제 측정값 사이의 오차를 최소화하는 것이다.1 이 과정은 통계적으로 최대우도추정(Maximum Likelihood Estimation)에 해당하며, 최소화할 목적 함수는 모든 이미지의 모든 특징점에 대한 **재투영 오차(Reprojection Error)**의 제곱 합이다.35
$$
\sum_{i=1}^{n} \sum_{j=1}^{m} \| \mathbf{m}_{ij} - \hat{\mathbf{m}}(K, D, R_i, \mathbf{t}_i, M_j) \|^2
$$
여기서 $n$은 이미지의 수, $m$은 각 패턴에 있는 특징점의 수, $\mathbf{m}_{ij}$는 $i$번째 이미지에서 관측된 $j$번째 특징점의 2D 좌표, $\hat{\mathbf{m}}(\cdot)$은 3D 점 $M_j$를 추정된 파라미터들($K$, 왜곡 계수 $D$, $i$번째 이미지의 외부 파라미터 $R_i, \mathbf{t}_i$)을 이용해 이미지 평면에 투영한 예측 좌표이다.

이러한 비선형 최소 제곱 문제(non-linear least squares problem)를 풀기 위해 컴퓨터 비전 분야에서는 **Levenberg-Marquardt (LM) 알고리즘**이 표준처럼 사용된다.1 LM 알고리즘은 해에서 멀리 떨어져 있을 때는 안정적인 경사 하강법(Gradient Descent)처럼 동작하고, 해에 가까워지면 빠른 수렴 속도를 보이는 가우스-뉴턴법(Gauss-Newton)처럼 동작하여, 좋은 초기 추정치가 주어졌을 때 빠르고 안정적으로 최적해를 찾는다.38 3D 구조와 카메라 파라미터를 동시에 최적화하는 이 과정을 일반적으로 **번들 조정(Bundle Adjustment)**이라고도 부른다.11 이 2단계 접근법(해석적 초기해 + 비선형 최적화)은 계산의 효율성과 최종 결과의 정확성을 모두 만족시키는 매우 강력하고 실용적인 전략이다.

## 3.  캘리브레이션 패턴 비교 분석: 체커보드 vs. 원형 그리드

Zhang의 방법론은 평면 패턴을 기반으로 하며, 이론적으로는 어떤 패턴이든 사용할 수 있다. 그러나 실제로는 특징점을 얼마나 쉽고 정확하게 검출할 수 있느냐에 따라 캘리브레이션의 전체 성능이 좌우된다. 가장 널리 사용되는 두 가지 패턴인 체커보드와 원형 그리드는 각각 뚜렷한 장단점을 가진다.

### 3.1 특징점 검출 방식

- **체커보드 (Checkerboard)**: 체커보드의 특징점은 흑백 사각형이 만나는 **코너(corner)**, 더 정확하게는 **안장점(saddle point)**이다.5 이 점들은 수평선과 수직선, 두 직선의 교점으로 명확하게 정의된다. 따라서 특징점 검출 알고리즘은 이미지에서 직선 성분을 찾고 그 교차점을 계산하는 방식으로 작동하며, 이를 통해 높은 아픽셀(sub-pixel) 수준의 정확도를 얻을 수 있다.41 OpenCV의 `findChessboardCorners`와 같은 함수는 이러한 원리를 기반으로 구현되어 널리 사용된다.1
- **원형 그리드 (Circular Grid)**: 원형 그리드의 특징점은 패턴에 그려진 각 원의 **중심(center)**이다.5 이미지 상에서는 원이 타원(ellipse)으로 투영되므로, 특징점은 이 타원의 중심 또는 **무게중심(centroid)**으로 정의된다.42 원의 중심은 하나의 점이 아니라 원을 구성하는 수많은 경계 픽셀들의 정보를 종합하여 계산된다. 이 때문에 이론적으로 노이즈에 더 강인하고 정밀한 위치 측정이 가능하다.5 특징점 검출 과정은 일반적으로 이미지 이진화(binarization), 연결 요소 분석(connected component analysis)을 통한 블롭(blob) 검출, 그리고 검출된 블롭에 대한 타원 피팅(ellipse fitting) 또는 무게중심 계산 순으로 이루어진다.9

### 3.2 정확도, 강인성, 계산 복잡도 비교

체커보드와 원형 그리드는 특징점 검출 방식의 차이로 인해 정확도, 강인성, 계산 복잡도 측면에서 상호 보완적인 특성을 보인다.

- **정확도 (Accuracy)**:
  - **원형 그리드**는 수많은 경계 픽셀 정보를 평균화하여 중심점을 계산하므로, 이론적으로 체커보드 코너보다 더 높은 아픽셀 정밀도(최대 0.01 픽셀 수준)를 달성할 잠재력을 가진다.12 이러한 특성 때문에 초정밀 측정이 요구되는 산업용 포토그래메트리(industrial photogrammetry) 분야에서 선호되어 왔다.12
  - 그러나 이는 **투영 왜곡으로 인한 편향이 완벽하게 보정되었을 때**만 유효한 이야기다. 기존의 단순한 무게중심 계산 방식은 심각한 시스템 오차를 유발하여, 오히려 체커보드보다 낮은 성능을 보이는 경우가 많았다.5
  - 반면 **체커보드**의 코너는 선의 교차점으로 정의되어 투시 변환이나 렌즈 왜곡에 대해 본질적으로 편향이 적은(unbiased) 특징을 가진다.41 따라서 복잡한 보정 없이도 안정적인 정확도를 제공한다.
- **강인성 (Robustness)**:
  - **원형 그리드**는 이미지 블러(blur)나 초점 이탈(defocusing)에 대해 체커보드보다 강인하다.42 코너점은 이미지가 흐릿해지면 그 위치가 급격히 불분명해지는 반면, 타원의 전체적인 형태와 무게중심은 상대적으로 안정적으로 유지되기 때문이다. 이미지 노이즈에 대해서도 평균화 효과 덕분에 더 강인한 경향이 있다.
  - **체커보드**는 일부가 가려지는 **폐색(occlusion)**에 대해 더 강인한 면모를 보인다. 패턴의 일부가 보이지 않더라도, 보이는 부분의 직선들을 연장하여 보이지 않는 교차점의 위치를 추론할 수 있기 때문이다.41 반면, 대칭적인 원형 그리드는 어떤 원이 가려졌는지 식별하기 어렵고, 180도 회전 모호성(ambiguity) 문제가 발생할 수 있어 스테레오 캘리브레이션 등에 사용하기 까다롭다.41
- **계산 복잡도 (Computational Complexity)**:
  - **체커보드** 코너 검출은 상대적으로 간단하고 최적화가 잘 된 알고리즘들이 널리 보급되어 있어 계산적으로 효율적이다.12
  - **원형 그리드**의 경우, 단순한 블롭 검출은 빠르지만 부정확하다. 앞서 언급한 편향 문제를 해결하고 높은 정확도를 얻기 위해서는 복잡한 수학적 모델링과 추가적인 보정 계산이 필요하여 계산 복잡도가 증가한다.12

아래 표는 두 패턴의 특성을 종합적으로 비교한 것이다.

| 특징 (Feature)                 | 체커보드 패턴 (Checkerboard Pattern)                         |                                                              | 원형 그리드 패턴 (Circular Grid Pattern)                     |                                                              |
| ------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **특징점 (Control Point)**     | 사각형의 교차점 (코너/안장점)                                |                                                              | 원의 중심 (무게중심)                                         |                                                              |
| **정확도 (Accuracy)**          | **장점:** 투시/왜곡 변환에 대한 편향이 적음 (unbiased).      | **단점:** 특징점이 소수 픽셀에 의존하여 이론적 정밀도 한계 존재. | **장점:** 다수의 경계 픽셀을 이용하므로 이론적 아픽셀 정밀도 높음.5 | 단점: 투시 및 렌즈 왜곡으로 인한 중심점 편향이 심각하여 보정 없이는 정확도 저하.5 |
| **강인성 (Robustness)**        | **장점:** 일부 가려짐(occlusion)에 대해 선형 특징을 이용한 추론 가능.41 | **단점**: 이미지 블러, 노이즈, 저해상도에서 코너 검출 성능 저하.5 | **장점:** 이미지 블러 및 노이즈에 상대적으로 강인함.42       | **단점**: 대칭적 구조로 인해 가려짐에 취약하고 180도 모호성 발생 가능.46 |
| **계산 복잡도 (Complexity)**   | **장점:** 특징점 검출 알고리즘이 비교적 간단하고 성숙함.41   |                                                              | **장점**: 단순 블롭 검출은 빠르지만 부정확함.단점: 정확한 중심 검출 및 편향 보정을 위해 복잡한 수학적 모델링과 계산 필요.12 |                                                              |
| **핵심 과제 (Core Challenge)** | 저해상도, 블러, 열화상 카메라 등에서 안정적인 코너 검출의 어려움.5 |                                                              | 중심점의 투영 편향 문제 해결.5                               |                                                              |

## 4.  원형 그리드 패턴의 핵심 과제: 투영 왜곡과 편향

원형 그리드 패턴이 가진 높은 이론적 정밀도에도 불구하고, 오랫동안 체커보드만큼 널리 사용되지 못한 근본적인 이유는 바로 '투영 중심 편향(projection center bias)' 문제 때문이다. 이 문제는 단일한 원인이 아니라, 투시 변환과 렌즈 왜곡이라는 두 가지 다른 기하학적 원리가 복합적으로 작용하여 발생한다. 이 두 편향의 원인을 명확히 구분하여 이해하는 것이 문제 해결의 첫걸음이다.

### 4.1 투시 변환에 의한 편향

3차원 공간에 놓인 원형 패턴을 카메라로 비스듬히 촬영하면, 원근법에 의해 원은 이미지 평면에 타원(ellipse)으로 투영된다.7 이때 중요한 사실은, 3D 공간에 있던 원래 원의 중심점이 투영된 점과, 2D 이미지에 나타난 타원의 기하학적 중심(장축과 단축의 교점)이 일반적으로 일치하지 않는다는 점이다.42

이러한 불일치는 순수하게 투시 변환(perspective transformation)의 기하학적 특성 때문에 발생하며, **편심 오차(eccentricity error)**라고도 불린다.48 이 오차는 패턴이 카메라의 광축에 대해 더 많이 기울어질수록, 즉 투시 효과가 강해질수록 커진다. 이는 캘리브레이션 과정에서 체계적인 오차(systematic error)를 유발하며, 정확도를 떨어뜨리는 원인이 된다.48 Heikkila를 비롯한 초기 연구들은 이 문제를 인식하고, 이를 보정하기 위한 기하학적 모델을 제시하기도 했다.47 만약 렌즈 왜곡이 없다면, 이 편심 오차는 예측 가능하고 수학적으로 보정할 수 있는 문제이다.

### 4.2 렌즈 왜곡에 의한 편향

원형 그리드 사용의 **가장 근본적이고 치명적인 한계**는 바로 렌즈 왜곡에 의한 편향이다.5 실제 카메라 렌즈에 의해 발생하는 방사 왜곡(radial distortion)과 같은 변환은 비선형(non-linear) 변환이다. 이 비선형 변환이 투시 변환에 의해 생성된 타원에 적용되면, 타원의 

**코닉(conic) 속성이 파괴**된다. 즉, 이미지 센서에 최종적으로 맺히는 형상은 더 이상 완벽한 2차 곡선인 타원이 아니라, '일그러진 타원(distorted ellipse)' 형태가 된다.5

이것이 결정적인 문제인 이유는 다음과 같다:

1. **타원 피팅의 실패**: 이미지에 나타난 형상이 더 이상 타원이 아니므로, 타원 방정식을 기반으로 중심을 찾는 타원 피팅(ellipse fitting) 알고리즘은 원리적으로 정확한 중심을 찾을 수 없다.
2. **무게중심의 편향**: 가장 간단한 방법은 이 '일그러진 타원'의 무게중심(centroid)을 계산하여 특징점으로 사용하는 것이다. 그러나 이 무게중심은 렌즈 왜곡이 없는 이상적인 투영점으로부터 예측 불가능한 큰 편향을 갖게 된다.
3. **오차의 심각성**: 연구에 따르면, 이 렌즈 왜곡으로 인한 편향 오차는 체커보드 코너 검출 시 발생하는 측정 오차보다 훨씬 더 심각할 수 있다.5 결국 이 편향은 원형 패턴이 본래 가지고 있던 아픽셀 정밀도의 잠재력을 완전히 무력화시키는 결과를 초래한다.

이처럼 렌즈 왜곡이 코닉의 기하학적 구조 자체를 파괴하기 때문에, 이 문제를 해석적으로 다룰 수 있는 효과적인 방법이 오랫동안 부재했다. 이것이 바로 많은 연구자들이 원형 패턴 대신 체커보드 패턴의 개선(예: ChArUco 패턴 개발)에 더 집중해 온 역사적 배경이다.5

## 5.  편향 보정을 위한 최신 접근법: 비편향 타원 중심 추정

원형 그리드 캘리브레이션의 정확도를 획기적으로 높이기 위해서는 투시 변환과 렌즈 왜곡에 의해 발생하는 복합적인 중심점 편향을 근본적으로 해결해야 한다. 최근 컴퓨터 비전 최상위 학회들(CVPR 등)에서 발표된 연구들은 이 문제에 대한 새로운 해석적 해법을 제시하며 큰 주목을 받고 있다. 그 핵심에는 **기하학적 모멘트(moment)** 이론이 있다.

### 5.1 기존 보정 방법들의 한계

과거의 편향 보정 시도들은 몇 가지 근본적인 한계를 가지고 있었다.

- **왜곡 무시 또는 근사**: 일부 연구는 렌즈 왜곡을 무시하고 투시 변환에 의한 편심 오차만 보정하거나 51, 동심원(concentric circles)의 기하학적 특성을 이용해 편향을 근사적으로 상쇄하려 했다.7 하지만 이는 왜곡이 심한 렌즈에서는 정확도가 크게 떨어졌다.
- **반복적 최적화**: 왜곡 보정된 이미지에서 타원을 찾거나, 관측된 점과 모델 예측점 사이의 오차를 반복적으로 최소화하는 방식도 제안되었다.9 그러나 이 방법들은 계산 비용이 매우 높고, 초기 추정치에 따라 지역 최적해(local minima)에 빠질 위험이 있으며, 수렴을 보장하기 어렵다는 단점이 있었다.
- **근본적 한계**: 이 모든 방법들은 렌즈 왜곡으로 인해 코닉의 고유한 속성이 파괴된다는 점을 해석적으로, 즉 닫힌 형태의 수학 공식(closed-form solution)으로 다루지 못했다.5

### 5.2 모멘트 이론 기반의 비편향 추정기

최근 Song 등이 CVPR 2024에서 발표한 연구는 **모멘트 이론**을 도입하여 이 문제를 해결할 수 있는 길을 열었다.5

- **핵심 아이디어**: 형상의 기하학적 특성을 나타내는 모멘트를 사용하여, 비선형 왜곡 변환 하에서도 중심점의 위치를 편향 없이(unbiased) 추적하는 것이다. 이 연구의 가장 핵심적인 수학적 발견은 다음과 같다: **"임의의 다항식(polynomial) 함수로 표현되는 변환이 가해지더라도, 변환 후 형상의 1차 모멘트(즉, 무게중심)는 변환 전 형상의 모멘트들의 선형 결합으로 항상 표현될 수 있다."** 13
- **수학적 원리**:
  1. 카메라 렌즈의 방사 왜곡은 일반적으로 다항식 모델로 근사된다 (II장 참조).3
  2. 모멘트 이론에 따르면, 이 다항식 왜곡 모델에 의해 변환된 '일그러진 타원'의 무게중심 좌표는, 왜곡이 없다고 가정한 이상적인 타원의 다양한 차수의 모멘트들($M^{i,j}$)과 왜곡 계수($k_1, k_2, \dots$)의 선형 결합으로 정확하게 표현될 수 있다.13
  3. 이를 통해, 이미지에서 관측된 왜곡된 형상으로부터 그 무게중심을 계산하는 것이 아니라, 3D 모델(원)과 추정된 카메라 파라미터($K, R, t, D$)로부터 왜곡된 형상의 무게중심이 어디에 위치할지를 **해석적으로 예측**하는 것이 가능해진다. 이것이 바로 **비편향 투영 모델(unbiased projection model)**이다.
  4. 이 모델은 투시 변환에 의한 편심 오차와 렌즈 왜곡에 의한 비선형 편향을 모두 하나의 통합된 수학 공식으로 다루기 때문에, 원형 패턴의 이론적 정밀도를 손실 없이 온전히 활용할 수 있게 한다.5

### 5.3 캘리브레이션 파이프라인에의 통합 및 성능 향상

이러한 비편향 추정기는 Zhang의 캘리브레이션 파이프라인에 다음과 같이 통합될 수 있다:

1. **특징점 검출 및 호모그래피 추정**: 여러 장의 이미지에서 원형 패턴의 위치를 대략적으로 찾는다.

2. **비선형 최적화 (Bundle Adjustment)**: 재투영 오차를 계산하는 단계에서, 3D 모델 점을 이미지에 투영할 때 단순 핀홀 모델을 사용하는 대신, 위에서 설명한 **비편향 투영 모델**을 사용한다. 즉, 목적 함수 자체가 편향을 고려하도록 재정의된다.
   $$
   \sum_{i=1}^{n} \sum_{j=1}^{m} \| \mathbf{c}_{ij} - \hat{\mathbf{c}}(K, D, R_i, \mathbf{t}_i, M_j) \|^2
   $$
   여기서 $\mathbf{c}_{ij}$는 이미지에서 관측된 $j$번째 원의 (단순) 무게중심이고, $\hat{\mathbf{c}}(\cdot)$는 비편향 투영 모델을 통해 예측된 무게중심 좌표이다. Levenberg-Marquardt 알고리즘은 이 오차를 최소화하는 파라미터($K, D, R_i, \mathbf{t}_i$)를 찾는다.

실제 실험 결과, 이 접근법은 기존의 체커보드 기반 방법이나 편향을 고려하지 않은 원형 그리드 방법에 비해 월등히 낮은 재투영 오차를 보이며 캘리브레이션 정확도를 크게 향상시켰다. 특히 렌즈 왜곡이 심하거나, 블러와 노이즈가 많은 열화상(Thermal Infrared, TIR) 카메라 이미지와 같이 기존 방법들이 어려움을 겪는 환경에서 그 효과가 두드러지게 나타났다.5

더 나아가, 최근에는 특징점의 위치가 완벽한 하나의 점이 아니라 확률 분포를 가진다고 가정하고, 이 **불확실성(uncertainty)**을 모델링하여 캘리브레이션의 강인성을 더욱 향상시키려는 연구도 활발히 진행되고 있다.5 이처럼 비편향 추정기의 등장은 오랫동안 원형 그리드 패턴의 발목을 잡아온 핵심 문제를 해결함으로써, 고정밀 캘리브레이션 분야에서 원형 그리드가 새로운 표준으로 자리 잡을 가능성을 열었다고 평가할 수 있다.

## 6.  종합 고찰 및 실용적 권장사항

지금까지 Zhang의 캘리브레이션 원리부터 원형 그리드 패턴의 특징, 핵심 과제, 그리고 최신 해결책까지 심층적으로 분석했다. 이를 바탕으로, 실제 응용에서 최상의 캘리브레이션 결과를 얻기 위한 전략적 활용 방안과 실용적인 권장사항을 제시한다. 고정밀 캘리브레이션은 단순히 '최고의 패턴'을 선택하는 문제가 아니라, 패턴 선택, 촬영 환경, 절차, 알고리즘을 포함한 전체 파이프라인의 '오차 예산(error budget)'을 종합적으로 관리하는 과정임을 이해하는 것이 중요하다.

### 6.1 원형 그리드 패턴의 전략적 활용

원형 그리드 패턴은 모든 상황에서 체커보드보다 우월한 만능 해결책은 아니다. 그러나 특정 조건에서는 분명한 이점을 제공하므로, 응용 분야의 요구사항에 맞춰 전략적으로 선택해야 한다.

- **적합한 응용 분야**: 원형 그리드는 블러(blur)나 초점 이탈(defocusing)에 강인하고, 이론적으로 높은 아픽셀 정밀도를 제공한다. 따라서 다음과 같은 분야에서 체커보드의 효과적인 대안 또는 그 이상의 성능을 기대할 수 있다:
  - **고정밀 산업 계측**: 부품의 미세한 치수나 형태를 측정하는 비전 검사 시스템.7
  - **로보틱스**: 특히 로봇 팔이 어두운 상자 안의 물체를 집는 빈 피킹(bin-picking)과 같이 조명이 불균일하고 블러가 발생하기 쉬운 환경.8
  - **의료 영상**: 내시경 영상과 같이 해상도가 낮고 왜곡이 심하며, 블러가 잦은 환경에서의 3D 재구성.6
- **필수 전제 조건**: 원형 그리드의 잠재력을 최대한 활용하기 위해서는 반드시 **비편향 중심 추정 알고리즘**을 적용해야 한다. OpenCV와 같은 표준 라이브러리에서 제공하는 `findCirclesGrid` 함수는 기본적으로 이러한 편향을 완벽하게 보정하지 않을 수 있으므로, 사용자는 적용된 알고리즘의 원리를 명확히 이해하거나, 비편향 모델을 지원하는 최신 구현체를 사용해야 한다.44 편향 보정 없이 원형 그리드를 사용하는 것은 오히려 체커보드보다 낮은 정확도를 초래할 수 있다.45

### 6.2 고정밀 캘리브레이션을 위한 촬영 환경 및 절차

캘리브레이션의 정확도는 알고리즘뿐만 아니라 입력 데이터, 즉 촬영된 이미지의 품질에 크게 좌우된다. 다음은 패턴 종류와 무관하게 고정밀 캘리브레이션을 위해 반드시 준수해야 할 모범 사례이다.

- **패턴 품질**: 캘리브레이션 패턴은 물리적으로 완벽에 가까워야 한다.
  - **평탄성 및 강성**: 패턴은 구겨지거나 휘지 않도록 매우 평평하고(ultra flat) 단단한(rigid) 재질(예: Dibond, 폼보드)에 부착하거나 인쇄해야 한다.9
  - **표면 처리**: 빛 반사를 최소화하기 위해 무광(anti-reflective) 코팅된 표면을 사용하는 것이 이상적이다. 이는 특징점 검출의 안정성을 높인다.55
- **조명 환경**:
  - **균일성**: 전체 패턴에 걸쳐 조명이 균일하게 분포하도록 **확산광(diffuse light)**을 사용해야 한다. 강한 단일 광원(점 광원)은 부분적인 하이라이트나 짙은 그림자를 만들어 특징점 검출을 실패하게 만드는 주된 원인이다.44
- **이미지 획득 절차**:
  - **이미지 수량 및 다양성**: 최소 6장에서 15장 이상의 이미지를 촬영하는 것이 권장된다.6 특히 고차 왜곡 모델을 사용하는 경우 더 많은 이미지가 필요하다. 카메라는 다양한 각도와 거리, 위치에서 패턴을 촬영해야 한다.
  - **이미지 커버리지**: 패턴이 이미지의 중앙뿐만 아니라 **네 모서리와 가장자리까지** 전체 영역을 고르게 커버하도록 촬영해야 한다. 이는 특히 렌즈 왜곡 계수를 정확하게 추정하는 데 결정적으로 중요하다.44
  - **기울기**: 초점 거리를 정확하게 추정하기 위해서는 원근에 의한 단축(foreshortening) 효과를 관찰해야 한다. 이를 위해 카메라와 평행한 정면(fronto-parallel) 이미지뿐만 아니라, 수평 및 수직 방향으로 **최대 45도까지 기울어진 이미지**를 반드시 포함해야 한다. 45도 이상 과도하게 기울이면 특징점의 형태가 심하게 왜곡되어 검출 정확도가 오히려 떨어질 수 있다.44
  - **고정된 초점**: 캘리브레이션은 실제 응용 프로그램이 사용될 **작업 거리(working distance)**에서 수행되어야 한다. 카메라 초점을 작업 거리에 맞춘 후, 캘리브레이션 과정과 이후 사용 중에 초점을 변경해서는 안 된다. 초점 변경은 내부 파라미터, 특히 초점 거리를 변화시킨다.44

### 6.3 패턴 선택 가이드라인: 응용 분야별 최적의 선택

모든 상황에 맞는 단 하나의 완벽한 패턴은 없다. 응용 분야의 특성과 요구사항을 고려하여 최적의 패턴을 선택해야 한다.

- **일반적인 용도**: 구현의 용이성, 높은 안정성, 그리고 풍부한 레퍼런스를 고려할 때, **체커보드**는 대부분의 응용 분야에서 여전히 가장 신뢰할 수 있고 훌륭한 첫 번째 선택이다.35
- **최고 정밀도 요구**: 산업용 정밀 계측, 고정밀 로보틱스 등에서 0.1 픽셀 이하의 오차를 목표로 하는 경우, **비편향 추정기가 적용된 원형 그리드**가 최적의 선택이 될 수 있다. 특히 블러가 심한 환경에서 그 진가를 발휘한다.5
- **폐색이 잦은 환경**: 로봇 팔이나 다른 물체에 의해 패턴의 일부가 자주 가려지는 환경에서는 일반 체커보드나 원형 그리드보다 **ChArUco**나 **AprilGrid**와 같은 하이브리드 패턴이 훨씬 강인한 성능을 보인다. 이 패턴들은 내부에 고유 ID를 가진 마커(fiducial marker)를 포함하고 있어, 일부만 보여도 전체 패턴의 포즈를 인식할 수 있다.46
- **광각/어안 렌즈**: 왜곡이 매우 심한 렌즈를 캘리브레이션할 때는 이미지 가장자리까지 특징점을 고르게 분포시키는 것이 중요하다. 이를 위해 특징점 밀도가 높은(denser grids) 패턴이나, 가장자리 검출에 유리한 ChArUco와 같은 패턴이 효과적일 수 있다.12

결론적으로, 캘리브레이션의 성공은 패턴, 조명, 촬영 절차, 그리고 사용되는 알고리즘의 총체적인 조화에 달려있다. 각 단계에서 발생할 수 있는 오차 요인을 이해하고 이를 최소화하려는 노력이 고정밀 캘리브레이션의 핵심이다.

## 7.  결론

### 핵심 내용 요약 및 연구의 의의

본 보고서는 원형 그리드 패턴을 이용한 Zhang의 카메라 캘리브레이션 방법론에 대해 심층적으로 고찰했다. Zhang의 방법은 평면 패턴과 호모그래피라는 간결한 아이디어를 통해 고가의 장비 없이도 높은 정확도의 캘리브레이션을 가능하게 함으로써 컴퓨터 비전 기술의 대중화에 크게 기여했음을 확인했다.

전통적인 체커보드 패턴과 원형 그리드 패턴의 특성을 비교 분석한 결과, 원형 그리드는 다수의 픽셀 정보를 활용하여 이론적으로 더 높은 아픽셀 정밀도를 달성할 잠재력을 가지고 있으나, 투시 변환과 렌즈 왜곡으로 인해 발생하는 '투영 중심 편향'이라는 고질적인 문제로 인해 그 잠재력이 제대로 발휘되지 못했음을 밝혔다. 특히 렌즈의 비선형 왜곡은 투영된 원의 코닉 속성을 파괴하여 기존의 기하학적 보정 방법들을 무력화시키는 근본적인 원인이었다.

그러나 최근 제안된 **모멘트 기반 비편향 추정기**는 이러한 한계를 해석적으로 극복할 수 있는 새로운 길을 열었다. 이 기술은 왜곡 변환 하에서도 형상의 1차 모멘트(무게중심)를 편향 없이 예측할 수 있게 함으로써, 원형 그리드 패턴의 이론적 정밀도를 온전히 활용할 수 있게 만들었다. 이는 고정밀 캘리브레이션 분야에서 중요한 기술적 진전이며, 원형 그리드 패턴의 위상을 재정립할 수 있는 계기가 될 것으로 기대된다.

### 7.1 향후 연구 방향 제언

원형 그리드 캘리브레이션은 비편향 추정기의 등장으로 큰 발전을 이루었지만, 카메라 캘리브레이션 분야는 여전히 해결해야 할 과제와 새로운 가능성을 안고 있다. 향후 연구는 다음과 같은 방향으로 나아갈 수 있을 것이다.

- **딥러닝 기반 캘리브레이션**: 최근 딥러닝 기술을 이용하여 캘리브레이션 패턴 없이 단일 이미지로부터 직접 카메라의 내부 파라미터나 왜곡을 추정하려는 연구가 활발히 진행되고 있다.56 이러한 방법들은 편의성 면에서 큰 장점을 가지며, 향후 전통적인 패턴 기반 방법과 상호 보완적으로 발전할 가능성이 있다.
- **다양한 센서 융합 캘리브레이션 (Multi-modal Sensor Calibration)**: 자율주행차나 로봇 시스템에서는 카메라뿐만 아니라 LiDAR, IMU, Radar 등 다양한 센서를 함께 사용한다. 이 센서들 간의 상대적인 위치와 방향, 즉 외부 파라미터를 정밀하게 교정하는 것은 시스템 전체 성능에 매우 중요하다.22 이 과정에서 각 센서의 개별적인 고정밀 캘리브레이션이 선행되어야 하므로, 본 보고서에서 다룬 기술들이 더욱 중요해질 것이다.
- **실시간 및 자동 캘리브레이션 (Real-time and Automatic Calibration)**: 로봇이나 드론과 같이 동적인 환경에서 운용되는 카메라는 충격이나 온도 변화 등으로 인해 파라미터가 미세하게 변할 수 있다. 이러한 변화를 실시간으로 감지하고, 별도의 오프라인 절차 없이 시스템 스스로 캘리브레이션을 수행하거나 보정하는 기술은 강인한 비전 시스템을 위한 필수적인 연구 분야가 될 것이다.

이러한 연구들은 카메라 캘리브레이션 기술을 더욱 정밀하고, 편리하며, 강인하게 만들어 3D 컴퓨터 비전 기술의 응용 범위를 더욱 확장시키는 데 기여할 것이다.

#### **참고 자료**

1. [CV] 카메라 캘리브레이션 & 카메라 파라미터 | 2D 이미지와 3D 월드 ..., 8월 3, 2025에 액세스, https://mvje.tistory.com/81
2. 카메라 캘리브레이션 (Camera Calibration) - 다크 프로그래머 - 티스토리, 8월 3, 2025에 액세스, https://darkpgmr.tistory.com/32
3. What Is Camera Calibration? - MATLAB & Simulink - MathWorks, 8월 3, 2025에 액세스, https://www.mathworks.com/help/vision/ug/camera-calibration.html
4. Camera Calibration Demystified: A Step-by-Step Guide to Obtaining Intrinsic and Extrinsic Parameters and Correcting Lens Distortion - Medium, 8월 3, 2025에 액세스, https://medium.com/@zigzagyc/camera-calibration-demystified-a-step-by-step-guide-to-obtaining-intrinsic-and-extrinsic-b47bfc9ff748
5. Camera Calibration via Circular Patterns: A Comprehensive Framework with Measurement Uncertainty and Unbiased Projection Model - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/html/2506.16842v1
6. Camera Calibration Explained: Enhancing Accuracy in Computer Vision Applications | by Md Faruk Alam | Perceptron Perspectives | Medium, 8월 3, 2025에 액세스, https://medium.com/perceptron-perspectives/camera-calibration-explained-enhancing-accuracy-in-computer-vision-applications-8ad1494cc5f2
7. Iterative Camera Calibration Method Based on Concentric Circle Grids - MDPI, 8월 3, 2025에 액세스, https://www.mdpi.com/2076-3417/14/5/1813
8. Camera Calibration in High-Speed Robotic Assembly Operations - MDPI, 8월 3, 2025에 액세스, https://www.mdpi.com/2076-3417/14/19/8687
9. Discorpy: algorithms and software for camera calibration and correction - IUCr Journals, 8월 3, 2025에 액세스, https://journals.iucr.org/s/issues/2025/03/00/mo5303/mo5303.pdf
10. A Flexible New Technique for Camera Calibration - Microsoft, 8월 3, 2025에 액세스, https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/tr98-71.pdf
11. A binocular camera calibration method based on circle detection - PMC, 8월 3, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC11467626/
12. Which pattern (circle pattern or checkerboard pattern) should be ..., 8월 3, 2025에 액세스, https://www.researchgate.net/post/Which-pattern-circle-pattern-or-checkerboard-pattern-should-be-used-for-automotive-camera-calibration-fisheye-wide-webcam
13. Unbiased Estimator for Distorted Conics in ... - CVF Open Access, 8월 3, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2024/papers/Song_Unbiased_Estimator_for_Distorted_Conics_in_Camera_Calibration_CVPR_2024_paper.pdf
14. Image formation and pinhole camera model - Notebook - Ontario Tech University, 8월 3, 2025에 액세스, https://csundergrad.science.uoit.ca/courses/cv-notes/notebooks/01-image-formation.html
15. CS231A Course Notes 1: Camera Models, 8월 3, 2025에 액세스, https://web.stanford.edu/class/cs231a/course_notes/01-camera-models.pdf
16. 1.2. The Pinhole Camera Matrix - Homepages of UvA/FNWI staff, 8월 3, 2025에 액세스, https://staff.fnwi.uva.nl/r.vandenboomgaard/IPCV20162017/LectureNotes/CV/PinholeCamera/PinholeCamera.html
17. Pinhole Camera Model | HediVision, 8월 3, 2025에 액세스, https://hedivision.github.io/Pinhole.html
18. Camera resectioning - Wikipedia, 8월 3, 2025에 액세스, https://en.wikipedia.org/wiki/Camera_resectioning
19. What are Intrinsic and Extrinsic Camera Parameters in Computer Vision?, 8월 3, 2025에 액세스, https://towardsdatascience.com/what-are-intrinsic-and-extrinsic-camera-parameters-in-computer-vision-7071b72fb8ec/
20. [Stereo Vision] 카메라 캘리브레이션의 내부 및 외부 매개변수(intrinsic, extrinsic parameters), 8월 3, 2025에 액세스, https://eehoeskrap.tistory.com/511
21. [영상 처리] 카메라 캘리브레이션 (Camera Calibration) - Day to_day - 티스토리, 8월 3, 2025에 액세스, https://day-to-day.tistory.com/42
22. Camera Intrinsic, Extrinsic and Distortion in Camera Calibration - BasicAI, 8월 3, 2025에 액세스, https://docs.basic.ai/docs/camera-intrinsic-extrinsic-and-distortion-in-camera-calibration
23. (카메라) 카메라 캘리브레이션에 대하여... - ROBO-STORY, 8월 3, 2025에 액세스, https://robo-story.tistory.com/7
24. Distortion - Imatest, 8월 3, 2025에 액세스, https://www.imatest.com/imaging/distortion/
25. Camera Calibration Theory - Calcam documentation - GitHub Pages, 8월 3, 2025에 액세스, https://euratom-software.github.io/calcam/html/intro_theory.html
26. Camera Calibration - OpenCV, 8월 3, 2025에 액세스, https://docs.opencv.org/4.x/dc/dbb/tutorial_py_calibration.html
27. Real-Time radial and tangential lens distortion with OpenGL - Medium, 8월 3, 2025에 액세스, https://medium.com/@vlh2604/real-time-radial-and-tangential-lens-distortion-with-opengl-a55b7493e207
28. Camera Calibration: Zhang's Method, 8월 3, 2025에 액세스, https://www.ipb.uni-bonn.de/html/teaching/photo12-2021/2021-pho1-22-Zhang-calibration.pptx.pdf
29. RBE/CS 549 Computer Vision HomeWork 1: Auto-Calib - PeAR WPI, 8월 3, 2025에 액세스, https://pear.wpi.edu/img/teaching/rbe549/spring2024/studentoutputs/hw1/shettypuneet.pdf
30. Camera Calibration --- Zhang's Algorithm, 8월 3, 2025에 액세스, https://engineering.purdue.edu/kak/computervision/ECE661Folder/Lecture21.pdf
31. Basic concepts of the homography explained with code - OpenCV Documentation, 8월 3, 2025에 액세스, https://docs.opencv.org/4.x/d9/dab/tutorial_homography.html
32. 41 Homographies - Foundations of Computer Vision, 8월 3, 2025에 액세스, https://visionbook.mit.edu/homography.html
33. How to compute homography matrix H from corresponding points (2d-2d planar Homography) - Mathematics Stack Exchange, 8월 3, 2025에 액세스, https://math.stackexchange.com/questions/494238/how-to-compute-homography-matrix-h-from-corresponding-points-2d-2d-planar-homog
34. anaghad01/cv-rbe549-camera-calibration: Camera calibration using Zhang's method., 8월 3, 2025에 액세스, https://github.com/anaghad01/cv-rbe549-camera-calibration
35. 2차원 평면 형태의 심층학습 기반 깊이 카메라 캘리브레이션, 8월 3, 2025에 액세스, http://journal.ksae.org/xml/20100/20100.pdf
36. SUPPLEMENTING BUNDLE ADJUSTMENT WITH EVOLUTIONARY ALGORITHMS - Fernuni Hagen, 8월 3, 2025에 액세스, https://www.fernuni-hagen.de/mci/download/pub/c2006_heywuepet_vie.pdf
37. (PDF) Is Levenberg-Marquardt the Most Efficient Optimization Algorithm for Implementing Bundle Adjustment?. - ResearchGate, 8월 3, 2025에 액세스, https://www.researchgate.net/publication/221111908_Is_Levenberg-Marquardt_the_Most_Efficient_Optimization_Algorithm_for_Implementing_Bundle_Adjustment
38. Camera calibration, 8월 3, 2025에 액세스, https://www.csie.ntu.edu.tw/~cyy/courses/vfx/05spring/lectures/handouts/lec07_calibration_4up.pdf
39. Leveraging the Levenberg-Marquardt Method on Vanishing Point Estimation | by Oleg Boev, 8월 3, 2025에 액세스, https://medium.com/@olegboev/leveraging-the-levenberg-marquardt-method-on-vanishing-point-estimation-in-computer-vision-5b10668b1ad5
40. Calibration Patterns Explained - calib.io, 8월 3, 2025에 액세스, https://calib.io/blogs/knowledge-base/calibration-patterns-explained
41. camera calibration: why chessboards? - Signal Processing Stack Exchange, 8월 3, 2025에 액세스, https://dsp.stackexchange.com/questions/24734/camera-calibration-why-chessboards
42. a camera calibration technique using targets of circular features - ResearchGate, 8월 3, 2025에 액세스, https://www.researchgate.net/publication/250669007_A_CAMERA_CALIBRATION_TECHNIQUE_USING_TARGETS_OF_CIRCULAR_FEATURES
43. Sub-pixel estimation, 8월 3, 2025에 액세스, https://homepages.inf.ed.ac.uk/rbf/PAPERS/spedraft.pdf
44. How do I get the most accurate camera calibration? - Signal Processing Stack Exchange, 8월 3, 2025에 액세스, https://dsp.stackexchange.com/questions/1567/how-do-i-get-the-most-accurate-camera-calibration
45. 개선된 화질의 영상을 이용한 열화상 카메라 캘리브레이션 - Korea Science, 8월 3, 2025에 액세스, https://koreascience.kr/article/JAKO202113855736594.pdf
46. Calibration Target Types: Which One Do You Need? - Foamcoreprint.com, 8월 3, 2025에 액세스, https://www.foamcoreprint.com/blog/calibration-target-types-which-one-do-you-need
47. calibration using circle grid pattern has perspective bias / Issue #7312 - GitHub, 8월 3, 2025에 액세스, https://github.com/opencv/opencv/issues/7312
48. Eccentricity Correction Methods for Circular Targets in Perspective Projection, 8월 3, 2025에 액세스, https://isprs-archives.copernicus.org/articles/XLVIII-2-W7-2024/65/2024/isprs-archives-XLVIII-2-W7-2024-65-2024.pdf
49. A Four-step Camera Calibration Procedure with Implicit Image Correction, 8월 3, 2025에 액세스, https://www.tjhsst.edu/~rlatimer/techlab03/eherbst03/heikkila_article.pdf
50. Unbiased Estimator for Distorted Conics in Camera Calibration - ResearchGate, 8월 3, 2025에 액세스, https://www.researchgate.net/publication/384235876_Unbiased_Estimator_for_Distorted_Conics_in_Camera_Calibration
51. CVPR Poster Unbiased Estimator for Distorted Conics in Camera Calibration, 8월 3, 2025에 액세스, https://cvpr.thecvf.com/virtual/2024/poster/29503
52. Unbiased Estimator for Distorted Conics in Camera Calibration - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/html/2403.04583v2
53. Intuitive bin picking software: suitable for beginners | KUKA AG, 8월 3, 2025에 액세스, https://www.kuka.com/en-de/products/robot-systems/software/application-software/kuka_smart-bin-picking
54. A novel hand-eye calibration method of picking robot based on TOF camera - PMC, 8월 3, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC9888730/
55. Target Grid | Why You Need a High Precision Calibration Target - Foamcoreprint.com, 8월 3, 2025에 액세스, https://www.foamcoreprint.com/circle-targets
56. Deep Learning for Camera Calibration and Beyond: A Survey - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/html/2303.10559v2

