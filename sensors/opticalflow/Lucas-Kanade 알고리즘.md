# Lucas-Kanade 알고리즘



광학 흐름(Optical Flow)은 컴퓨터 비전 분야에서 동적 장면(dynamic scene)을 이해하기 위한 근본적인 개념이다. 이는 시간의 흐름에 따라 연속적으로 입력되는 이미지 프레임들 사이에서 관찰되는 객체, 표면, 그리고 엣지(edge)의 겉보기 움직임 패턴(apparent motion pattern)으로 정의된다.1 본질적으로, 3차원 실제 공간에서 발생하는 물리적 움직임이 카메라의 투영(projection) 과정을 거쳐 2차원 이미지 평면에 나타나는 결과물이다.4 이 겉보기 움직임은 각 픽셀 또는 이미지 상의 특정 지점이 다음 프레임에서 어느 위치로 이동했는지를 나타내는 2차원 변위 벡터 필드(displacement vector field)로 표현된다.1

광학 흐름의 중요성은 그것이 이미지 시퀀스에 내재된 시간적 정보를 정량화하여 다양한 고수준 컴퓨터 비전 작업을 가능하게 하는 기반 기술이라는 점에서 비롯된다. 예를 들어, 광학 흐름 정보는 특정 객체의 위치를 프레임마다 추적하는 객체 추적(object tracking), 장면 내에서 움직이는 영역과 정적인 영역을 구분하는 움직임 감지(motion detection), 비디오 데이터의 시간적 중복성(temporal redundancy)을 제거하여 압축 효율을 높이는 비디오 압축(video compression), 그리고 두 대의 카메라로 촬영한 이미지 간의 대응점(correspondence)을 찾아 깊이 정보를 추론하는 스테레오 비전(stereo vision) 등 수많은 응용 분야의 핵심적인 요소로 활용된다.1 이처럼 광학 흐름은 정적인 이미지를 분석하는 것을 넘어, 동적인 세계를 기계가 인식하고 해석하게 하는 교량 역할을 수행한다.


광학 흐름을 추정하는 기법은 계산 범위와 결과의 밀도에 따라 크게 희소 방식과 밀집 방식으로 분류된다.

**희소 광학 흐름 (Sparse Optical Flow):** 이 접근법은 이미지 전체가 아닌, 추적하기에 용이한 특정 '흥미로운' 지점들, 예를 들어 코너(corner)나 뚜렷한 텍스처를 가진 특징점(feature points)들의 움직임만을 계산한다.8 모든 픽셀을 처리하지 않기 때문에 계산 효율성이 매우 높아, 제한된 컴퓨팅 자원을 가진 환경이나 실시간 처리가 필수적인 응용 분야에 적합하다.8 본 보고서에서 중점적으로 다룰 루카스-카나데(Lucas-Kanade) 알고리즘은 이러한 희소 광학 흐름을 추정하는 가장 대표적이고 고전적인 방법론이다.9

**밀집 광학 흐름 (Dense Optical Flow):** 반면, 밀집 방식은 이미지 내의 모든 픽셀에 대해 움직임 벡터를 계산하여, 조밀하고 상세한 모션 필드를 생성한다.4 이 방식은 이미지 분할(image segmentation)이나 3D 장면 재구성(3D scene reconstruction)과 같이 모든 픽셀의 움직임 정보가 필요한 작업에 유용하다. 하지만 모든 픽셀을 계산해야 하므로 희소 방식에 비해 계산 비용이 현저히 높다는 단점이 있다.4 Horn-Schunck 알고리즘이나 Farneback 알고리즘이 전통적인 밀집 광학 흐름 기법에 해당하며, 최근에는 딥러닝 기반 모델들이 이 분야에서 최첨단 성능을 보이고 있다.11


루카스-카나데 알고리즘은 1981년 Bruce D. Lucas와 Takeo Kanade에 의해 처음 제안된 이래로 컴퓨터 비전 분야에서 가장 영향력 있고 널리 사용되는 기술 중 하나로 자리매김했다.6 이 알고리즘은 이미지의 밝기 값 변화율, 즉 그래디언트(gradient)를 이용하는 미분 기반(differential method) 접근법에 속하며, 전체 이미지가 아닌 특정 픽셀 주변의 작은 영역에만 집중하는 지역적(local) 방법론이다.6

알고리즘의 핵심 아이디어는 매우 직관적이면서도 강력하다. 단일 픽셀의 정보만으로는 움직임을 정확히 특정하기 어렵다는 '애퍼처 문제(aperture problem)'를 해결하기 위해, "특정 픽셀을 중심으로 하는 작은 이웃(neighborhood) 또는 윈도우(window) 내의 모든 픽셀은 동일한 움직임을 갖는다"고 가정한다.15 이 '공간적 일관성(spatial coherence)' 가정을 통해, 윈도우 내 여러 픽셀로부터 다수의 방정식을 얻어낼 수 있으며, 이를 최소제곱법(least squares method)으로 풀어 가장 적합한 단일 움직임 벡터를 찾아낸다. 여러 픽셀의 정보를 통합하는 이러한 방식은 개별 픽셀 단위로 계산하는 점 단위(point-wise) 방식에 비해 이미지 노이즈에 훨씬 강인한 특성을 보인다.15

루카스-카나데 알고리즘의 역사적 의의는 그것이 제시한 설계 철학에서 찾을 수 있다. 알고리즘이 탄생한 1980년대 초반의 컴퓨팅 자원은 매우 제한적이었고, 이미지의 모든 픽셀에 대해 복잡한 계산을 수행하는 밀집 광학 흐름은 실용적이지 않았다.9 Lucas와 Kanade는 문제의 범위를 '추적하기 좋은' 특징점과 그 '주변 지역'으로 의도적으로 한정함으로써, 전체 문제를 여러 개의 작고 독립적인 최적화 문제로 분해했다.8 이러한 접근 방식은 알고리즘의 계산 복잡도를 전체 픽셀 수에 선형적으로 비례하도록 만들어, 당시의 기술 수준에서도 실용적인 속도를 달성할 수 있게 했다.16 따라서 이 알고리즘의 '희소성'과 '지역성'은 기술적 한계라기보다는, 실용성을 확보하기 위한 핵심적인 설계적 타협(design trade-off)이었으며, 바로 이 효율성 덕분에 수십 년이 지난 오늘날까지도 객체 추적 및 비디오 안정화와 같은 수많은 실시간 응용 분야에서 초석이 되는 기술로 확고히 자리 잡고 있다.6


루카스-카나데 알고리즘은 물리적 세계의 복잡한 움직임을 수학적으로 다루기 쉬운 모델로 변환하기 위해 세 가지 강력한 기본 가정 위에 구축된다. 이 가정들은 각각 독립적으로 작용하는 것이 아니라, 서로 유기적으로 연결되어 알고리즘의 성립을 가능하게 하는 연쇄적인 논리 구조를 형성한다.


알고리즘의 가장 근본이 되는 첫 번째 가정은 '밝기 항상성'이다. 이는 추적 대상이 되는 객체 표면의 특정 지점은 조명 조건이 일정하다면, 연속된 프레임 간에 그 지점의 픽셀 밝기(intensity) 값이 변하지 않는다는 가정이다.6 즉, 객체가 움직여서 이미지 상의 위치가 변하더라도, 그 픽셀 값 자체는 그대로 유지된다는 것이다.

이를 수학적으로 표현하면 다음과 같다. 시간 $t$에 위치 $(x, y)$에 있던 픽셀의 밝기 값을 $I(x, y, t)$라고 할 때, 이 픽셀이 짧은 시간 $dt$ 후에 $(dx, dy)$만큼 이동하여 $(x+dx, y+dy)$ 위치로 갔다면, 시간 $t+dt$에서의 밝기 값은 이전과 동일해야 한다.18
$$
I(x, y, t) = I(x + dx, y + dy, t + dt)
$$
이 가정은 알고리즘의 출발점을 제공하는 대전제 역할을 한다. 하지만 이는 그림자, 반사광의 변화, 객체 자체의 색상 변화 등 실제 환경에서 흔히 발생하는 조명 변화가 없다는 이상적인 상황을 전제로 한다. 따라서 실제 응용에서는 이 가정이 깨지면서 알고리즘의 오차를 유발하는 주요 원인이 되기도 한다.6


두 번째 가정은 프레임 간 픽셀의 이동량이 매우 작다, 통상적으로 1픽셀 미만이라고 가정하는 '작은 움직임' 가정이다.14 이 가정은 일반적으로 비디오가 초당 20프레임 이상으로 촬영되어, 연속된 프레임 사이의 시간 간격이 매우 짧기 때문에 객체의 움직임이 미미할 것이라는 관찰에 기반한다.19

이 가정은 수학적 유도 과정에서 결정적인 역할을 한다. '밝기 항상성' 가정을 통해 얻은 방정식 $I(x, y, t) = I(x + dx, y + dy, t + dt)$는 변위 $(dx, dy)$에 대해 비선형적(non-linear)이어서 직접 풀기가 어렵다. 여기서 '작은 움직임' 가정, 즉 $dx$와 $dy$가 매우 작다는 조건이 도입되면, 방정식의 우변을 1차 테일러 급수(first-order Taylor series)를 이용해 선형적으로 근사하는 것이 가능해진다.1 움직임이 크면 테일러 급수의 고차항을 무시할 수 없게 되어 이 근사가 부정확해지므로, 이 가정은 알고리즘의 유효 범위를 정의하는 중요한 제약 조건이 된다. 이처럼 '작은 움직임' 가정은 다루기 힘든 비선형 문제를 다루기 쉬운 선형 문제로 변환하는 열쇠이다.


세 번째 가정은 '공간적 일관성'으로, 추적하려는 한 픽셀을 중심으로 하는 특정 크기(예: 3x3, 5x5)의 윈도우(window) 내에 위치한 모든 픽셀들은 동일한 움직임 벡터 $(u, v)$를 공유한다는 가정이다.4 여기서 $u$와 $v$는 각각 x축과 y축 방향의 속도(velocity)를 의미한다. 이 가정은 물리적으로 동일한 객체의 표면에 있는 인접한 점들은 강체(rigid body)처럼 함께 움직일 것이라는 직관에 근거한다.19

이 가정의 수학적 중요성은 앞선 두 가정을 통해 도출된 선형 방정식의 한계를 극복하는 데 있다. 테일러 근사를 통해 얻어진 방정식은 미지수가 속도 벡터의 두 성분 $(u, v)$인데, 식은 단 하나뿐인 부정(underdetermined) 시스템이다. 이는 '애퍼처 문제'로 알려진 현상으로, 단일 지점의 정보만으로는 움직임의 모든 성분을 결정할 수 없음을 의미한다.1 '공간적 일관성' 가정은 이 문제를 해결하기 위해 윈도우 내의 N개의 모든 픽셀에 대해 동일한 $(u, v)$를 적용하여 $N$개의 방정식을 생성한다. 이를 통해 미지수는 2개인데 방정식은 $N$개인 과결정 시스템(overdetermined system)을 구축할 수 있게 되며, 이는 최소제곱법과 같은 통계적 방법을 통해 유일한 최적 해를 구할 수 있는 기반을 마련한다.16

결론적으로, 이 세 가지 가정은 루카스-카나데 알고리즘의 논리적 흐름을 완벽하게 구성한다. '밝기 항상성'이 해결해야 할 문제를 수학적으로 정의하고, '작은 움직임'이 이 문제를 풀기 쉬운 선형 형태로 변환하며, 마지막으로 '공간적 일관성'이 선형화된 문제의 해를 유일하게 결정하는 구조를 완성한다. 이 세 가지 가정 중 어느 하나라도 없다면 알고리즘의 수학적 유도는 성립할 수 없다.


루카스-카나데 알고리즘의 수학적 근간은 앞서 설명한 세 가지 가정을 바탕으로 광학 흐름을 나타내는 선형 방정식을 유도하고, 이를 최소제곱법을 통해 푸는 과정에 있다.


유도의 시작점은 '밝기 항상성' 가정이다. 시간 $t$에서 위치 $(x, y)$에 있던 픽셀이 시간 $dt$ 후 속도 $(u, v)$로 이동하여 $(x+u \cdot dt, y+v \cdot dt)$에 위치한다고 가정하자. 밝기 항상성 원리에 따라 다음 식이 성립한다.
$$
I(x, y, t) = I(x+u \cdot dt, y+v \cdot dt, t+dt)
$$
'작은 움직임' 가정에 따라 $u \cdot dt$와 $v \cdot dt$가 매우 작으므로, 위 식의 우변을 점 $(x, y, t)$에 대해 1차 테일러 급수(Taylor series)로 전개하여 근사할 수 있다.1
$$
I(x+u \cdot dt, y+v \cdot dt, t+dt) \approx I(x,y,t) + \frac{\partial I}{\partial x}(u \cdot dt) + \frac{\partial I}{\partial y}(v \cdot dt) + \frac{\partial I}{\partial t}dt
$$
편의를 위해 편미분 $\frac{\partial I}{\partial x}$, $\frac{\partial I}{\partial y}$, $\frac{\partial I}{\partial t}$를 각각 $I_x$, $I_y$, $I_t$로 표기하자. 위 식을 정리하면 다음과 같다.
$$
I(x, y, t) \approx I(x,y,t) + I_x u \cdot dt + I_y v \cdot dt + I_t dt
$$
양변의 $I(x,y,t)$ 항을 소거하고 $dt$로 나누면, 최종적으로 '광학 흐름 제약 방정식(Optical Flow Constraint Equation)'을 얻게 된다.1
$$
I_x u + I_y v + I_t = 0
$$
또는,
$$
I_x u + I_y v = -I_t
$$
이 방정식은 단일 픽셀에서 이미지의 공간적 밝기 변화율($I_x, I_y$)과 시간적 밝기 변화율($I_t$)이 속도 벡터 $(u, v)$와 맺는 관계를 나타내는 선형 제약 조건이다.


광학 흐름 제약 방정식은 미지수 $u$와 $v$ 두 개를 포함하지만 방정식은 하나뿐이므로, 이 자체만으로는 해를 구할 수 없다. 여기서 '공간적 일관성' 가정이 도입된다. 추정하고자 하는 픽셀 $p$를 중심으로 하는 $n \times n$ 크기의 윈도우 $W$ 내에 있는 모든 픽셀 $q_1, q_2, \dots, q_N$ (여기서 $N=n^2$)은 모두 동일한 속도 벡터 $(u, v)$를 갖는다고 가정한다.1

이 가정을 적용하면, 윈도우 내의 각 픽셀 $q_i$에 대해 광학 흐름 제약 방정식을 하나씩 세울 수 있다.
$$
\begin{cases}
I_x(q_1)u + I_y(q_1)v = -I_t(q_1) \\
I_x(q_2)u + I_y(q_2)v = -I_t(q_2) \\
\quad \vdots \\
I_x(q_N)u + I_y(q_N)v = -I_t(q_N)
\end{cases}
$$
이 $N$개의 선형 방정식 시스템은 행렬 형태로 간결하게 표현할 수 있다. $Av = b$.1
$$
\begin{cases}
I_x(q_1)u + I_y(q_1)v = -I_t(q_1) \\
I_x(q_2)u + I_y(q_2)v = -I_t(q_2) \\
\quad \vdots \\
I_x(q_N)u + I_y(q_N)v = -I_t(q_N)
\end{cases}
$$
일반적으로 윈도우 크기는 $3 \times 3$ 이상이므로 $N > 2$가 되어, 이 시스템은 미지수의 개수(2)보다 방정식의 개수($N$)가 많은 과결정(overdetermined) 시스템이 된다.


과결정 시스템 $Av = b$는 일반적으로 모든 방정식을 동시에 만족시키는 정확한 해 $v$를 갖지 않는다. 이는 이미지 노이즈나 모델의 근사 오차 때문이다. 따라서, 오차 벡터 $Av - b$의 크기를 최소화하는 최적의 해 $v$를 찾는 것이 합리적인 접근법이다. 최소제곱법은 이 오차 벡터의 L2-norm 제곱, 즉 오차 제곱의 합(Sum of Squared Errors) $E(u, v)$를 최소화하는 해를 찾는다.16
$$
E(u, v) = \|Av - b\|^2 = \sum_{i=1}^{N} (I_x(q_i)u + I_y(q_i)v + I_t(q_i))^2
$$
이 최소제곱 문제는 잘 알려진 '정규 방정식(Normal Equation)'을 통해 해결할 수 있다.1
$$
A^T A v = A^T b
$$
이러한 접근 방식은 통계적 관점에서 중요한 의미를 갖는다. 만약 이미지의 노이즈가 평균이 0인 가우시안 분포를 따른다고 가정하면(이는 일반적인 가정이다 15), 최소제곱법으로 구한 해는 통계적으로 분산이 가장 작은 선형 불편 추정량(Best Linear Unbiased Estimator, BLUE)이 된다. 이는 루카스-카나데 알고리즘이 단순히 방정식을 푸는 것을 넘어, 노이즈가 존재하는 현실 데이터로부터 가장 신뢰할 수 있는 움직임 벡터를 추정하는 통계적으로 최적화된 방법임을 시사하며, 알고리즘이 노이즈에 비교적 강인한 이유를 수학적으로 뒷받침한다.15


정규 방정식을 $v$에 대해 풀면, 루카스-카나데 알고리즘의 최종 해를 얻을 수 있다.1
$$
v = (A^T A)^{-1} A^T b
$$
여기서 행렬 $A^T A$는 $2 \times 2$ 크기의 대칭 행렬로, 그 형태는 다음과 같다.
$$
A^T A =
\begin{bmatrix}
\sum_{i=1}^{N} I_x(q_i)^2 & \sum_{i=1}^{N} I_x(q_i) I_y(q_i) \\
\sum_{i=1}^{N} I_x(q_i) I_y(q_i) & \sum_{i=1}^{N} I_y(q_i)^2
\end{bmatrix}
$$
이 행렬은 윈도우 내 픽셀들의 그래디언트 분포, 즉 이미지의 지역적 구조(local structure)에 대한 정보를 담고 있기 때문에 '구조 텐서(Structure Tensor)'라고 불린다. 또한, 해리스 코너 검출(Harris Corner Detection)에서 사용되는 행렬과 동일한 형태를 가지므로 '해리스 행렬(Harris Matrix)'이라고도 한다.15

벡터 $A^T b$는 다음과 같이 계산된다.
$$
A^T b =
\begin{bmatrix}
-\sum_{i=1}^{N} I_x(q_i) I_t(q_i) \\
-\sum_{i=1}^{N} I_y(q_i) I_t(q_i)
\end{bmatrix}
$$
결론적으로, 루카스-카나데 알고리즘은 특정 픽셀 주변의 지역적 이미지 구조($A^T A$)와 밝기 변화($A^T b$)를 분석하여, 해당 지역의 움직임 벡터 $v$를 선형 대수적으로 계산하는 정교한 프레임워크를 제공한다.


루카스-카나데 알고리즘은 강력하지만, 그 성능은 기반 가정이 성립하는 특정 조건 하에서만 보장된다. 알고리즘의 성공과 실패는 수학적으로 구조 텐서 $A^T A$의 특성과 밀접하게 연관되어 있으며, 이는 이미지의 지역적 특성에 의해 결정된다.


최소제곱 해 $v = (A^T A)^{-1} A^T b$가 유일하게 존재하기 위한 수학적 필요충분조건은 행렬 $A^T A$가 가역(invertible)이어야 한다는 것이다.1 만약 $A^T A$가 비가역(non-invertible) 또는 특이(singular) 행렬이라면, 역행렬이 존재하지 않아 해를 구할 수 없다.

$A^T A$는 $2 \times 2$ 실수 대칭 행렬이므로, 항상 두 개의 실수 고유값(eigenvalue) $\lambda_1$과 $\lambda_2$를 가진다. 행렬이 가역이기 위한 조건은 이 두 고유값이 모두 0이 아니어야 한다는 것과 동치이다. 더 나아가, $A^T A$는 준양정부호(positive semi-definite) 행렬이므로 고유값은 항상 0 이상이다. 따라서 가역 조건은 두 고유값이 모두 0보다 커야 한다는 조건, 즉 $\lambda_1 \ge \lambda_2 > 0$으로 구체화된다.1

실제 구현에서는 수치적 안정성과 노이즈의 영향을 고려하여 더 엄격한 조건을 적용한다. 즉, 두 고유값이 단순히 0보다 큰 것이 아니라, 특정 양수 임계값 $\tau$보다 커야 한다는 조건을 사용한다: $\lambda_1 \ge \lambda_2 > \tau$.15 이 임계값은 계산된 움직임 벡터의 신뢰도를 판단하는 기준으로 작용한다.


구조 텐서 $A^T A$의 고유값 $\lambda_1, \lambda_2$의 크기와 분포는 해당 윈도우 내 이미지의 지역적 특성을 정량적으로 나타낸다. 이는 윈도우가 평탄한 영역, 엣지, 또는 코너에 위치하는지를 판별하는 기준으로 사용될 수 있다.

- **평탄 영역 (Flat region):** 윈도우 내에 밝기 변화가 거의 없는 균일한 영역이다. 이 경우, 모든 방향으로의 그래디언트 $I_x$와 $I_y$가 거의 0에 가깝다. 결과적으로 구조 텐서의 모든 원소가 0에 가까워지고, 두 고유값 $\lambda_1$과 $\lambda_2$가 모두 매우 작게 된다. 이 상황에서는 $A^T A$의 역행렬 계산이 수치적으로 매우 불안정해지며, 의미 있는 움직임을 추정할 수 없다.16
- **엣지 (Edge):** 윈도우가 선이나 경계와 같은 구조에 위치한 경우이다. 엣지는 한 방향(엣지에 수직인 방향)으로만 강한 그래디언트를 갖고, 다른 방향(엣지와 평행한 방향)으로는 그래디언트가 거의 없다. 예를 들어, 수직 엣지의 경우 $I_x$는 크지만 $I_y$는 0에 가깝다. 이로 인해 하나의 고유값($\lambda_1$)은 크고 다른 고유값($\lambda_2$)은 매우 작은, 즉 $\lambda_1 \gg \lambda_2$인 분포를 보인다. 이 경우 $A^T A$ 행렬은 조건이 나빠져(ill-conditioned), 역행렬 계산 시 작은 노이즈에도 결과가 크게 변동할 수 있다. 이는 후술할 애퍼처 문제의 직접적인 원인이 된다.19
- **코너 (Corner) 또는 텍스처가 풍부한 영역:** 윈도우가 두 개 이상의 엣지가 만나는 코너나 복잡한 텍스처를 포함하는 영역이다. 이 경우, 모든 방향으로 강한 그래디언트가 존재하여 $I_x$와 $I_y$가 모두 유의미한 값을 가진다. 결과적으로 두 고유값 $\lambda_1$과 $\lambda_2$가 모두 크고 서로 비슷한 값을 갖게 된다. 이때 $A^T A$ 행렬은 잘 조건화(well-conditioned)되어 안정적으로 역행렬을 계산할 수 있으며, 가장 신뢰할 수 있는 움직임 벡터를 추정할 수 있다.1

이러한 분석은 루카스-카나데 알고리즘의 성공 조건이 '추적하기 좋은 특징(good features to track)'의 정의와 정확히 일치함을 보여준다. 해리스 코너 검출기는 구조 텐서의 고유값들을 이용하여 코너 응답 함수를 계산하고, Shi-Tomasi 코너 검출기는 $\min(\lambda_1, \lambda_2)$가 큰 지점을 직접 찾는다.8 이는 루카스-카나데가 가장 잘 작동하는 지점을 찾는 것과 동일한 원리이다. 따라서 특징점 검출과 특징점 추적은 별개의 문제가 아니라, 구조 텐서라는 동일한 수학적 기반을 공유하는 강하게 결합된 문제이다. 애퍼처 문제와 같은 한계점은 역설적으로 어떤 특징을 추적해야 하는지에 대한 명확한 수학적 지침을 제공하며, 이는 컴퓨터 비전에서 검출과 추적의 공생 관계를 잘 보여주는 사례이다.


애퍼처 문제는 작은 구멍(aperture)이나 창을 통해 움직이는 객체의 일부만을 관찰할 때 발생하는 인식의 모호성을 의미한다.19 관찰자는 창의 경계에 수직인 방향의 움직임 성분만 인지할 수 있고, 경계와 평행한 방향의 움직임 성분은 감지할 수 없다.

**시각적 예시:** 무한히 긴 수직선이 오른쪽 아래 대각선 방향으로 움직이고 있다고 상상해보자. 만약 이 장면을 작은 원형 창(aperture)을 통해 본다면, 관찰자는 선이 단순히 오른쪽으로 수평 이동하는 것처럼 인식할 것이다.19 선이 아래로 내려가는 움직임 성분은 창 안에서 선의 모양 변화를 일으키지 않기 때문에 감지되지 않는다.

루카스-카나데 알고리즘에서는 분석 윈도우가 바로 이 '애퍼처' 역할을 한다. 윈도우가 '엣지' 영역에 위치할 때 이 문제가 발생한다. 엣지와 평행한 방향으로는 그래디언트가 거의 없기 때문에, 해당 방향의 움직임은 광학 흐름 제약 방정식 $I_x u + I_y v = -I_t$에 거의 영향을 주지 못한다. 즉, 해당 방향의 움직임을 제약할 정보가 없는 것이다. 수학적으로 이는 구조 텐서 $A^T A$의 한 고유값($\lambda_2$)이 0에 가까워지는 현상으로 나타나며, 행렬의 조건이 나빠져 속도 벡터 $(u, v)$를 유일하게 결정할 수 없게 만든다. 결과적으로 엣지에 수직인 방향의 속도 성분만 신뢰할 수 있게 계산되고, 평행한 방향의 속도 성분은 불확실해진다.15


알고리즘의 다른 주요 한계점들은 핵심 가정들이 위배될 때 발생한다.

- **큰 변위(Large Displacement):** 알고리즘은 '작은 움직임' 가정에 기반하여 1차 테일러 근사를 사용한다. 만약 프레임 간 객체의 실제 움직임이 윈도우 크기를 벗어날 정도로 크다면, 이 근사는 더 이상 유효하지 않게 되어 알고리즘은 잘못된 결과를 내거나 추적에 완전히 실패한다.15
- **텍스처 부족(Textureless) 영역:** 앞서 설명한 '평탄 영역'과 같이 텍스처가 부족한 영역에서는 그래디언트 정보가 거의 없다. 이로 인해 구조 텐서의 고유값들이 모두 작아져, 안정적인 해를 구할 수 없게 된다. 알고리즘은 이러한 영역에서 움직임을 감지하지 못한다.6

이러한 한계점들을 극복하기 위해 다양한 개선 기법들이 제안되었으며, 다음 장에서 이를 상세히 다룬다.


기본적인 루카스-카나데 알고리즘은 그 자체로 효율적이지만, 앞서 논의된 한계점들로 인해 실제 응용에서는 성능이 저하될 수 있다. 이러한 문제들을 해결하고 알고리즘의 정확성과 강인함(robustness)을 높이기 위해 다양한 개선 및 확장 기법들이 개발되었다


기본 알고리즘은 '공간적 일관성' 가정에 따라 윈도우 내의 모든 픽셀이 움직임 추정에 동일하게 기여한다고 본다. 하지만 직관적으로 생각해보면, 윈도우의 중심에 있는 픽셀이 가장자리 픽셀보다 추정하려는 지점의 움직임과 더 밀접한 관련이 있을 가능성이 높다. 가중 윈도우 기법은 이러한 직관을 반영하여, 윈도우 내 각 픽셀에 차등적인 가중치를 부여하는 방식이다.15

일반적으로 윈도우 중심으로부터의 거리가 멀어질수록 가중치가 점진적으로 감소하는 2차원 가우시안 함수(Gaussian function)가 가중치 $w_i$로 널리 사용된다.15 이를 적용하면 오차 함수는 가중 최소제곱(Weighted Least Squares) 형태로 수정된다.
$$
E(u, v) = \sum_{i=1}^{N} w_i (I_x(q_i)u + I_y(q_i)v + I_t(q_i))^2
$$
이 오차 함수를 최소화하는 정규 방정식은 $A^T W A v = A^T W b$가 되며, 최종 해는 다음과 같이 표현된다. 여기서 $W$는 가중치 $w_i$를 대각 원소로 갖는 $N \times N$ 대각 행렬이다.15
$$
v = (A^T W A)^{-1} A^T W b
$$
가중 윈도우를 사용하면 윈도우 경계 부근의 픽셀이나 노이즈의 영향을 줄여, 보다 안정적이고 정확한 움직임 추정이 가능하다.


'작은 움직임' 가정은 알고리즘의 핵심적인 제약 조건이다. 반복적 정제 기법은 이 가정을 완화하고 더 큰 움직임에 대해서도 정확한 해를 찾기 위해 사용된다. 이는 일종의 예측-보정(predict-correct) 방식으로, 초기 추정치를 점진적으로 개선해나가는 과정이다.15

알고리즘의 절차는 다음과 같다.

1. **초기 움직임 추정:** 첫 번째 프레임 $I_1$과 두 번째 프레임 $I_2$에 대해 기본 루카스-카나데 알고리즘을 적용하여 초기 움직임 벡터 $v_0 = (u_0, v_0)$를 계산한다.
2. **이미지 워핑(Warping):** 계산된 움직임 벡터 $v_0$를 이용해 $I_1$을 $I_2$ 방향으로 이동시켜 보정된 이미지 $I_1'$를 생성한다. 워핑은 이미지의 각 픽셀을 추정된 변위만큼 이동시키는 과정이다.
3. **잔여 움직임 계산:** 이제 $I_1'$과 $I_2$ 사이에는 초기 움직임이 보정되었으므로 더 작은 '잔여 움직임(residual motion)'만 남아있을 것이다. 이 두 이미지에 대해 다시 루카스-카나데 알고리즘을 적용하여 잔여 움직임 벡터 $\Delta v_0$를 계산한다.
4. **추정치 업데이트:** 전체 움직임 벡터를 $v_1 = v_0 + \Delta v_0$로 업데이트한다.
5. **반복:** $\Delta v_k$가 미리 정의된 임계값보다 작아져 수렴할 때까지 2~4단계를 반복한다.17

이 과정은 본질적으로 비선형 최적화 문제인 $\arg\min \sum (I_1(x) - I_2(W(x; p)))^2$를 풀기 위한 뉴턴-랩슨(Newton-Raphson) 또는 가우스-뉴턴(Gauss-Newton) 방법과 동일한 원리를 따른다.19 이를 통해 초기 추정치가 다소 부정확하더라도 점차 실제 해에 가깝게 수렴시킬 수 있다.


큰 변위 문제를 해결하기 위한 가장 표준적이고 효과적인 접근법은 이미지 피라미드(image pyramid)를 활용하는 것이다.4

**원리:** 먼저 원본 이미지(피라미드의 레벨 0)를 가우시안 필터링 후 다운샘플링하는 과정을 반복하여, 점차 해상도가 낮아지는 이미지들의 계층적 집합, 즉 이미지 피라미드를 생성한다.33 예를 들어, 레벨 1 이미지는 레벨 0 이미지의 가로, 세로 해상도가 절반이 되고, 레벨 2는 레벨 1의 절반이 된다.

**Coarse-to-Fine 추적:** 추적 과정은 피라미드의 최상층, 즉 가장 해상도가 낮은(coarsest) 레벨에서 시작된다.33

1. **최상위 레벨:** 이 레벨에서는 원본 이미지에서의 큰 움직임이 몇 픽셀 이내의 작은 움직임으로 축소되어 나타난다. 따라서 기본 루카스-카나데 알고리즘을 적용하여 움직임 벡터를 안정적으로 추정할 수 있다.
2. **결과 전파:** 최상위 레벨에서 계산된 움직임 벡터에 2를 곱하여(해상도 차이 보정) 한 단계 아래(finer) 레벨의 초기 추정치로 사용한다.
3. **잔여 움직임 정제:** 아래 레벨에서는 이 초기 추정치를 이용해 이미지를 미리 워핑한 후, 남아있는 미세한 잔여 움직임만을 루카스-카나데 알고리즘으로 계산한다.
4. **반복:** 계산된 잔여 움직임을 이전 레벨의 결과에 더해 현재 레벨의 최종 움직임 추정치를 업데이트하고, 이 과정을 원본 해상도인 레벨 0에 도달할 때까지 반복한다.33

이러한 계층적 접근 방식은 '작은 움직임'이라는 기본 가정을 각 레벨에서 유지하면서도, 전체적으로는 매우 큰 변위를 효과적으로 처리할 수 있게 한다.37 더 나아가, 이 Coarse-to-Fine 전략은 최적화 관점에서 중요한 이점을 제공한다. 저해상도 이미지는 고주파 성분(세부 텍스처)이 제거되어 오차 함수(error function)가 더 부드러운 형태를 띤다. 이러한 환경에서는 지역 최솟값(local minima)에 빠질 위험이 줄어들어 전역 최솟값(global minimum)에 더 가까운 해를 찾을 가능성이 높아진다. 즉, 피라미드 기법은 큰 움직임을 처리하는 동시에 최적화 문제의 안정성을 높이는 이중의 효과를 거둔다.


기본 루카스-카나데 알고리즘은 윈도우 내의 모든 픽셀이 동일한 병진(translation) 운동을 한다는, 즉 2개의 파라미터 $(u, v)$로 모델링되는 단순한 움직임을 가정한다. 하지만 실제 세계의 객체는 회전, 크기 변화, 기울어짐(shear) 등 더 복잡한 움직임을 보인다.39

이러한 복잡한 움직임을 모델링하기 위해, 움직임을 일반화된 워프 함수(warp function) $W(x; p)$로 표현할 수 있다. 여기서 $x$는 픽셀 좌표이고, $p$는 움직임을 기술하는 파라미터 벡터이다.31

- **아핀 변환(Affine Transformation):** 6개의 파라미터를 사용하여 병진, 회전, 크기 변화, 기울어짐을 표현할 수 있다.
- **투영 변환(Projective Transformation / Homography):** 8개의 파라미터를 사용하여 원근 왜곡(perspective distortion)까지 모델링할 수 있다.

이러한 일반화된 모델을 적용하면, 알고리즘의 목표는 최적의 움직임 벡터 $(u, v)$를 찾는 것에서, 오차를 최소화하는 최적의 파라미터 증분 $\Delta p$를 찾는 것으로 확장된다.31 이는 루카스-카나데의 프레임워크가 단순한 2D 병진 추적을 넘어, 훨씬 더 넓은 범위의 이미지 정합(image registration) 문제에 적용될 수 있음을 의미한다.


루카스-카나데 알고리즘은 이론적 우수성뿐만 아니라 구현의 용이성과 효율성 덕분에 다양한 실제 컴퓨터 비전 시스템에서 핵심적인 '구성 요소(building block)'로 기능한다. 이 알고리즘은 단독으로 사용되기보다는 특징점 검출, 움직임 모델링, 필터링 등 다른 기술과 결합하여 특정 응용 문제에 최적화된 솔루션을 구축하는 데 활용된다.


희소(sparse) 방식의 피라미드 루카스-카나데 알고리즘을 구현하는 일반적인 단계는 다음과 같다.

1. **입력:** 연속된 두 그레이스케일 프레임 $I_1$(이전 프레임)과 $I_2$(현재 프레임)를 입력으로 받는다.14
2. **특징점 검출 (선택 사항):** 첫 프레임이거나 추적 중인 특징점의 수가 부족할 경우, $I_1$에서 추적할 특징점들을 검출한다. 일반적으로 Shi-Tomasi 코너 검출기($goodFeaturesToTrack$)가 선호되는데, 이는 루카스-카나데 알고리즘이 잘 작동하는, 즉 구조 텐서의 최소 고유값이 큰 지점을 찾아주기 때문이다.40
3. **이미지 피라미드 생성:** $I_1$과 $I_2$에 대해 각각 이미지 피라미드를 생성한다.
4. **Coarse-to-Fine 추적:**
   - 피라미드의 최상위 레벨(가장 저해상도)에서 시작하여, 이전 프레임의 특징점 위치에 대해 루카스-카나데 알고리즘을 적용하여 움직임 벡터를 계산한다.
   - 계산된 벡터를 다음 레벨(고해상도)의 초기 추정치로 삼아, 해당 레벨에서 잔여 움직임을 계산하고 추정치를 정제한다. 이 과정을 원본 해상도까지 반복한다.
5. **각 레벨에서의 계산:** 각 레벨의 각 특징점에 대해 다음을 수행한다.
   - **이미지 편미분 계산:** 특징점 주변 윈도우 내의 픽셀들에 대해 공간 그래디언트 $I_x$, $I_y$와 시간 그래디언트 $I_t$를 계산한다. $I_x, I_y$는 Sobel 필터와 같은 미분 필터로, $I_t$는 두 프레임 간의 단순 차이($I_2 - I_1$)로 근사할 수 있다.14
   - **행렬 구성 및 풀이:** 윈도우 내 그래디언트 값들을 이용하여 구조 텐서 $A^T A$와 벡터 $A^T b$를 구성한다.25
   - **신뢰도 검사 (고유값):** $A^T A$의 최소 고유값이 미리 설정된 임계값 $\tau$보다 작은지 확인한다. 만약 작다면 해당 특징점은 추적이 불확실한 것으로 판단하고 '추적 실패'로 표시한다.25
   - **움직임 벡터 계산:** 신뢰도 검사를 통과한 경우, 선형 시스템 $v = (A^T A)^{-1} A^T b$를 풀어 움직임 벡터 $(u, v)$를 구한다.14
6. **출력:** 최종적으로 각 특징점에 대한 $I_2$에서의 새로운 위치, 그리고 각 점의 추적 상태(성공 또는 실패)를 나타내는 플래그를 출력한다.11


알고리즘의 성능은 몇 가지 주요 파라미터에 의해 크게 좌우된다.

- **윈도우 크기 (Window Size):** '공간적 일관성' 가정이 적용되는 영역의 크기를 결정한다.
  - **작은 윈도우 (예: 3x3, 5x5):** 윈도우 내 픽셀들이 동일한 움직임을 가질 확률이 높아져 공간적 일관성 가정이 더 잘 성립한다. 미세하고 국부적인 움직임을 잘 포착하지만, 이미지 노이즈에 민감하고 애퍼처 문제에 취약할 수 있다.10
  - **큰 윈도우 (예: 21x21, 31x31):** 더 많은 픽셀 정보를 통합하므로 노이즈에 강인하고 안정적인 추정이 가능하다. 하지만 윈도우 내에 서로 다른 움직임을 갖는 객체나 경계가 포함될 경우, 움직임이 평균화되어 부정확한 결과를 낳을 수 있다.25
- **고유값 임계값 (Eigenvalue Threshold, $\tau$):** 추적할 특징점의 '품질'을 결정하는 기준으로, '좋은 특징'을 정의한다.
  - **높은 임계값:** 구조 텐서의 두 고유값이 모두 큰, 즉 매우 뚜렷한 코너 특징점만을 추적 대상으로 삼는다. 추적의 신뢰도는 높아지지만, 추적되는 특징점의 수가 줄어들어 전역적인 움직임을 추정하기 어려울 수 있다.
  - **낮은 임계값:** 더 많은 특징점(덜 뚜렷한 코너나 강한 엣지 포함)을 추적할 수 있게 하지만, 신뢰도가 낮은 추적 결과를 포함할 위험이 커진다.25


KLT(Kanade-Lucas-Tomasi) 트래커는 루카스-카나데 알고리즘의 가장 유명하고 직접적인 응용 사례이다.45 이는 Shi-Tomasi의 특징점 검출 방식과 Kanade-Lucas의 추적 방식을 결합한 것이다.

**원리:** KLT 트래커는 먼저 Shi-Tomasi 알고리즘을 사용하여 '추적하기 좋은 특징점'들을 프레임에서 검출한다. 그 후, 피라미드 기반의 루카스-카나데 알고리즘을 이용하여 연속된 프레임 간에 이 특징점들의 위치를 효율적으로 추적한다.26 사용자가 특정 객체가 포함된 영역(Region of Interest, ROI)을 지정하면, KLT는 해당 영역 내에서 특징점들을 찾아내고, 이 점들의 집단적인 움직임을 통해 객체의 전체적인 이동, 회전, 크기 변화 등을 추정할 수 있다.31 KLT는 루카스-카나데가 단독으로는 '무엇을' 추적해야 할지 결정할 수 없는 반면, Shi-Tomasi 검출기가 '추적할 대상'을 제공하는 완벽한 협력 관계를 보여준다.


비디오 안정화는 촬영 중 발생한 카메라의 불필요한 흔들림을 제거하여 부드러운 영상을 만드는 기술이다.47 루카스-카나데 알고리즘은 이 과정에서 프레임 간 움직임을 추정하는 핵심 모듈로 사용된다.

**파이프라인:** 일반적인 비디오 안정화 파이프라인은 다음과 같은 단계로 구성된다.47

1. **특징점 추적:** KLT 트래커와 같은 기법을 사용하여 연속된 프레임 간에 다수의 안정적인 특징점을 추적한다.
2. **전역 움직임 추정:** 추적된 특징점들의 변위로부터 프레임 전체의 전역적인 움직임(global motion)을 나타내는 변환 행렬(예: 아핀 변환, 호모그래피)을 추정한다. 이 행렬은 카메라의 흔들림을 수학적으로 모델링한 것이다.
3. **움직임 평활화(Smoothing):** 추정된 변환 행렬들의 시계열 데이터(움직임 궤적)에서 손떨림과 같은 고주파 성분의 노이즈를 제거한다. 이를 위해 이동 평균 필터(moving average filter)나 칼만 필터(Kalman filter)와 같은 신호 처리 기법이 사용된다. 이를 통해 의도적인 카메라 움직임은 유지하면서 부드러운 카메라 경로를 생성한다.
4. **프레임 보정(Warping):** 원본 프레임에, 원래의 흔들리는 경로와 새롭게 생성된 부드러운 경로 간의 차이를 보상하는 역변환을 적용한다. 이 과정을 통해 각 프레임이 부드러운 경로에 맞춰 재정렬되어 안정화된 비디오가 생성된다.48 이 파이프라인에서 루카스-카나데는 안정화 자체를 수행하는 것이 아니라, 후속 단계에 필요한 정확한 움직임 정보를 제공하는 필수적인 역할을 담당한다.


현대 비디오 압축 표준(예: MPEG, H.26x)은 연속된 프레임 간의 유사성, 즉 시간적 중복성을 제거하여 데이터를 효율적으로 압축한다. 이 과정의 핵심이 움직임 예측 및 보상(Motion Estimation and Compensation)이다.50

루카스-카나데와 같은 광학 흐름 알고리즘은 현재 프레임의 특정 블록(block)이나 픽셀이 참조 프레임(보통 이전 프레임)의 어느 위치로부터 이동해 왔는지를 나타내는 움직임 벡터(motion vector)를 예측하는 데 활용될 수 있다.1 인코더는 이 움직임 벡터와, 움직임 보상 후에도 남아있는 미세한 차이 정보(잔차, residual)만을 전송한다. 디코더는 참조 프레임에 움직임 벡터를 적용하여 현재 프레임을 예측하고, 여기에 잔차 정보를 더해 원본 프레임을 복원한다. 이 방식을 통해 전체 프레임 데이터를 전송하는 대신 훨씬 적은 양의 데이터(움직임 벡터 + 잔차)만으로 비디오를 표현할 수 있어 압축 효율을 극대화할 수 있다.50

이처럼 루카스-카나데 알고리즘의 본질은 '움직임 추정'이라는 매우 근본적이고 일반적인 문제를 해결하는 데 있다. 이 일반성 덕분에 특정 응용에 맞춰 다른 전문화된 모듈과 쉽게 조합하여 객체 추적, 비디오 안정화, 비디오 압축 등 다양한 고수준 시스템을 구축할 수 있다. 따라서 루카스-카나데의 진정한 가치는 독립적인 솔루션으로서가 아니라, 다양한 컴퓨터 비전 파이프라인에서 신뢰성 있고 효율적인 '움직임 정보 제공자'로서의 모듈성에 있다.


루카스-카나데 알고리즘의 특성과 위치를 명확히 이해하기 위해서는 다른 주요 광학 흐름 추정 알고리즘들과의 비교가 필수적이다. 비교 대상은 크게 고전적인 전역적/밀집 방식 알고리즘과 최신 딥러닝 기반 모델로 나눌 수 있다.


Horn-Schunck 알고리즘은 루카스-카나데와 함께 광학 흐름 분야를 개척한 양대 산맥으로 꼽히는 고전적인 방법론이다. 두 알고리즘은 동일한 광학 흐름 제약 방정식에서 출발하지만, 문제 해결을 위한 추가적인 가정에서 근본적인 차이를 보인다.

- **최적화 범위:** 루카스-카나데는 각 픽셀 주변의 작은 윈도우 내에서만 최적화를 수행하는 '지역적(local)' 방식이다.12 각 윈도우는 서로 독립적으로 계산된다. 반면, Horn-Schunck는 이미지 전체에 걸쳐 단일 에너지 함수(energy functional)를 정의하고 이를 최소화하는 '전역적(global)' 방식을 채택한다.4
- **핵심 가정:** Horn-Schunck의 에너지 함수는 두 개의 항으로 구성된다. 첫 번째 항은 루카스-카나데와 동일한 광학 흐름 제약 조건(데이터 항)이고, 두 번째 항은 전체 이미지에 걸쳐 추정된 움직임 필드가 공간적으로 부드러워야 한다는 '평활성 제약(smoothness constraint)'이다.4 이 평활성 제약이 두 알고리즘을 구분하는 가장 큰 특징이다.
- **결과 차이:** 이러한 차이로 인해 결과물의 성격이 달라진다. 루카스-카나데는 텍스처가 풍부한 코너와 같은 특정 지점에서만 신뢰할 수 있는 결과를 내놓는 '희소한(sparse)' 광학 흐름을 생성한다.4 텍스처가 없는 평탄한 영역에서는 해를 구하지 못한다. 반면, Horn-Schunck는 평활성 제약 덕분에 텍스처가 없는 영역의 움직임을 주변 영역의 정보를 통해 보간(interpolate)하여, 이미지의 모든 픽셀에서 움직임 벡터를 계산하는 '조밀한(dense)' 결과를 생성한다.4 일반적으로 Horn-Schunck가 더 정확하고 상세한 모션 필드를 제공하지만, 전역 최적화를 위해 반복적인 계산이 필요하여 계산 비용이 훨씬 높다.8


Gunnar Farneback 알고리즘은 또 다른 고전적인 밀집 광학 흐름 알고리즘으로, 현재 OpenCV와 같은 라이브러리에서 루카스-카나데와 함께 표준 옵션으로 제공된다.11

- **접근 방식:** Farneback 알고리즘은 각 픽셀의 이웃을 2차 다항식(quadratic polynomial)으로 근사하여 신호의 지역적 모델을 구축한다. 그런 다음, 이 다항식 모델이 두 프레임 간에 어떻게 변위되는지를 분석하여 움직임을 추정한다. 이는 그래디언트 기반의 미분 방식과는 다른 접근법이다.
- **결과 밀도:** 루카스-카나데가 본질적으로 희소 방식인 것과 달리, Farneback 알고리즘은 이미지의 모든 픽셀에 대해 움직임 벡터를 계산하는 밀집 방식이다. 따라서 시각적으로 풍부하고 연속적인 모션 필드를 제공하지만, 희소 방식인 루카스-카나데보다 계산량이 많다.13


최근 몇 년간, FlowNet, PWC-Net, RAFT와 같은 딥러닝 기반 모델들이 광학 흐름 추정 분야에서 최첨단(state-of-the-art) 성능을 달성했다.8 이 모델들은 고전적인 방법들과는 패러다임이 다르다.

- **학습 방식:** 딥러닝 모델은 대규모의 광학 흐름 데이터셋(실제 데이터 또는 합성 데이터)을 이용하여, 두 이미지 프레임을 입력받아 밀집 광학 흐름 필드를 출력하는 복잡한 비선형 함수를 종단간(end-to-end) 방식으로 학습한다. 이들은 명시적인 수학적 가정(밝기 항상성 등)에 의존하기보다는 데이터에 내재된 패턴을 학습한다.
- **성능 및 강인함:** 딥러닝 모델들은 조명 변화, 큰 변위, 객체 가림(occlusion), 흐릿한 경계 등 고전적 방법들이 어려움을 겪는 매우 도전적인 시나리오에서 훨씬 더 높은 정확도와 강인함을 보인다.8
- **자원 요구사항:** 고전적 방법, 특히 루카스-카나데는 CPU 환경에서도 실시간 처리가 가능할 정도로 계산 효율이 높다.8 반면, 딥러닝 모델은 방대한 양의 파라미터를 처리하기 위해 추론(inference) 과정에서 강력한 GPU를 거의 필수적으로 요구한다.8
- **데이터 의존성:** 고전적 방법은 특정 데이터셋 없이도 수학적 원리에 기반하여 작동한다. 반면, 딥러닝 모델의 성능은 학습에 사용된 데이터의 양과 질, 그리고 다양성에 크게 의존한다. 학습 데이터에 존재하지 않는 유형의 장면에 대해서는 성능이 저하될 수 있다.

이러한 비교는 '최고'의 알고리즘을 찾는 것이 아니라, 특정 '응용'의 제약 조건(정확도, 속도, 자원)에 맞는 '최적'의 도구를 선택하는 문제임을 시사한다. 딥러닝 모델이 정확도에서 압도적 우위를 보이지만, 자율주행차의 특정 기능이나 모바일 증강현실 앱과 같이 실시간 반응이 중요하고 계산 자원이 제한적인 환경에서는 딥러닝 모델의 추론 시간이 너무 길거나 전력 소모가 클 수 있다.6 이러한 환경에서는 수십 년간 검증되어 예측 가능한 성능을 보이는 루카스-카나데의 낮은 계산 비용과 빠른 속도가 여전히 강력한 경쟁력을 가진다. 따라서 딥러닝의 시대에도 루카스-카나데는 '사라진 기술'이 아니라, 특정 문제 영역(실시간, 저전력, 희소 추적)에서 여전히 유효하고 최적의 선택일 수 있는 '전문화된 도구'로서의 가치를 유지하고 있다.


다음 표는 위에서 논의된 알고리즘들의 주요 특징을 요약하여 비교한다.

| 특징 (Feature)                       | **Lucas-Kanade**                    | **Horn-Schunck**                  | **딥러닝 기반 (e.g., RAFT)**          |
| ------------------------------------ | ----------------------------------- | --------------------------------- | ------------------------------------- |
| **방식 (Method)**                    | 지역적, 희소 (Local, Sparse)        | 전역적, 밀집 (Global, Dense)      | 종단간 학습, 밀집 (End-to-End, Dense) |
| **정확도 (Accuracy)**                | 중간 (Moderate)                     | 높음 (High)                       | 매우 높음 (Very High)                 |
| **계산 복잡도 (Computational Req.)** | 낮음 (Low)                          | 높음 (High)                       | 중간 (Moderate, GPU 필요)             |
| **주요 가정 (Key Assumption)**       | 지역적 움직임 일관성                | 전역적 움직임 평활성              | 데이터로부터 학습된 패턴              |
| **강점 (Strength)**                  | 속도, 실시간성, 구현 용이성 6       | 상세한 모션 필드, 이론적 정교함 4 | 복잡한 움직임, 조명/가림에 강인함 8   |
| **약점 (Weakness)**                  | 큰 변위, 애퍼처 문제, 텍스처 부족 6 | 계산 비용, 움직임 경계 흐림 4     | 대규모 학습 데이터 및 GPU 필요 8      |


루카스-카나데 알고리즘은 40년이 넘는 시간 동안 컴퓨터 비전 분야에서 꾸준히 연구되고 활용되어 온, 시대를 초월하는 기술이다. 딥러닝이라는 새로운 패러다임이 많은 분야를 지배하는 오늘날에도, 이 고전적인 알고리즘의 원리를 이해하고 그 가치를 재평가하는 것은 여전히 중요하다.


루카스-카나데 알고리즘의 지속적인 생명력은 그 명확한 장점에서 기인한다.

- **장점:** 가장 큰 장점은 단연 **높은 계산 효율성**이다. 지역적 접근과 희소한 특징점 추정을 통해 계산량을 최소화하여, CPU만으로도 실시간 처리가 가능하다.6 이는 임베디드 시스템이나 모바일 기기와 같이 자원이 제한된 환경에서 특히 중요하다. 또한, 알고리즘의 수학적 원리가 명확하고 

  **구현이 비교적 단순**하며, 여러 픽셀의 정보를 통합하는 최소제곱법 덕분에 **이미지 노이즈에 대해 상대적으로 강인한** 특성을 보인다.6

반면, 알고리즘은 그 근간을 이루는 가정들로부터 비롯되는 근본적인 한계 또한 명확히 가지고 있다.

- **단점:** **밝기 항상성 가정**은 실제 환경의 조명 변화에 매우 민감하게 만든다.6

  **'작은 움직임' 가정**은 빠른 움직임이나 낮은 프레임률의 비디오에서 성능 저하를 야기하며, 이는 피라미드 기법으로 상당 부분 완화될 수 있지만 완벽한 해결책은 아니다.6 또한, 

  **텍스처가 부족한 영역**에서는 움직임을 추정할 수 없으며, 엣지에서는 **애퍼처 문제**로 인해 불완전한 정보만을 얻게 된다.6


이러한 장단점을 고려할 때, 루카스-카나데 알고리즘의 현대적 의의는 최첨단 성능을 내는 단독 솔루션으로서가 아니라, 더 큰 시스템 내에서 특정 역할을 수행하는 효율적인 구성 요소(component) 또는 강력한 기본 원리로서의 가치에 있다.

첫째, 실시간 응용 분야에서 루카스-카나데는 여전히 대체 불가능한 역할을 수행한다. 실시간 SLAM(Simultaneous Localization and Mapping), 증강 현실(AR)에서의 객체 트래킹, 로보틱스 등 수많은 시스템에서 빠르고 가벼운 움직임 추정 모듈이 필요하며, KLT 트래커의 형태로 구현된 루카스-카나데는 이러한 요구사항을 충족시키는 검증된 솔루션이다.

둘째, 최신 딥러닝 기술과의 융합을 통해 새로운 가능성을 열고 있다. 예를 들어, 복잡한 딥러닝 기반 광학 흐름 모델의 계산 비용을 줄이기 위해, 루카스-카나데로 계산된 희소한 움직임 필드를 네트워크의 초기값(initial guess)으로 제공하거나, 학습 과정에서 물리적 제약을 가하는 정규화(regularization) 항으로 활용하는 연구가 진행되고 있다.54 이러한 하이브리드 방식은 고전적 알고리즘의 효율성과 딥러닝 모델의 강인함을 결합하여 시너지를 창출할 수 있다.

셋째, 교육적 가치가 매우 높다. 루카스-카나데 알고리즘을 유도하는 과정은 테일러 급수, 최소제곱법, 선형 대수, 고유값 분석 등 컴퓨터 비전의 핵심적인 수학적 도구들을 종합적으로 이해하는 훌륭한 사례를 제공한다. 애퍼처 문제와 구조 텐서를 통해 특징점 검출과 추적의 근본적인 관계를 통찰하게 함으로써, 컴퓨터 비전 연구자에게 필수적인 기초를 다져준다.

결론적으로, 루카스-카나데 알고리즘은 비록 최신 딥러닝 모델만큼 모든 상황에서 높은 정확도를 보장하지는 않지만, 그 효율성과 단순성, 그리고 명확한 이론적 기반이라는 독보적인 장점을 바탕으로 현대 컴퓨터 비전 기술 생태계에서 여전히 중요한 한 축을 담당하고 있다. 이는 특정 응용 분야의 제약 조건에 맞는 최적의 도구를 선택하는 것이 중요함을 보여주는 살아있는 증거이며, 미래의 컴퓨터 비전 기술 또한 이러한 고전적 지혜의 토대 위에서 발전해 나갈 것임을 시사한다.


1. Lecture #17: Motion - Stanford Computer Vision Lab, 8월 17, 2025에 액세스, http://vision.stanford.edu/teaching/cs131_fall1718/files/17_notes.pdf
2. Optical flow, 8월 17, 2025에 액세스, https://homepages.inf.ed.ac.uk/rbf/CVonline/LOCAL_COPIES/OWENS/LECT12/node4.html
3. Optical Flow: Lucas-Kanade Method | Baeldung on Computer Science, 8월 17, 2025에 액세스, https://www.baeldung.com/cs/optical-flow-lucas-kanade-method
4. Optical Flow in OpenCV (C++/Python) - LearnOpenCV, 8월 17, 2025에 액세스, https://learnopencv.com/optical-flow-in-opencv/
5. An Insight into Optical Flow and Its Application using Lucas-Kanade Algorithm - Medium, 8월 17, 2025에 액세스, https://medium.com/@guptachinmay321/an-insight-into-optical-flow-and-its-application-using-lucas-kanade-algorithm-c12f9f6e3773
6. Mastering Lucas-Kanade Method - Number Analytics, 8월 17, 2025에 액세스, https://www.numberanalytics.com/blog/mastering-lucas-kanade-method
7. Lucas–Kanade method - Wikipedia, 8월 17, 2025에 액세스, [https://en.wikipedia.org/wiki/Lucas%E2%80%93Kanade_method](https://en.wikipedia.org/wiki/Lucas–Kanade_method)
8. Horn-Schunck and Lucas Kanade=1See last slide for copyright information. - School of Computer Science, 8월 17, 2025에 액세스, https://www.cs.auckland.ac.nz/~rklette/CCV-CIMAT/pdfs/B08-HornSchunck.pdf
9. Comparison of Optical Flow Algorithms for Speed Determination of Moving Objects - CiteSeerX, 8월 17, 2025에 액세스, https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=5bef85844b401cdde9cc32ff60882abef20c74ee
10. The Horn & Schunck method, 8월 17, 2025에 액세스, https://www.cvl.isy.liu.se/en/education/undergraduate/tsbb15/reading-material/Horn-Schunck.pdf
11. Optical Flow - Lucas-Kanade Method - YouTube, 8월 17, 2025에 액세스, https://www.youtube.com/watch?v=6wMoHgpVUn8&pp=0gcJCfwAo7VqN5tD
12. Lucas-Kanade Optical Flow - Carnegie Mellon University, 8월 17, 2025에 액세스, https://www.cs.cmu.edu/~16385/s15/lectures/Lecture21.pdf
13. Lucas–Kanade method Description, 8월 17, 2025에 액세스, [https://classes.engineering.wustl.edu/ese461/FinalProject/OpticalFlow/Optical%20Flow%20Description-Lucas%C3%82%C2%ADKanade.pdf](https://classes.engineering.wustl.edu/ese461/FinalProject/OpticalFlow/Optical Flow Description-LucasÂ­Kanade.pdf)
14. Motion and Optical Flow - University of Washington, 8월 17, 2025에 액세스, https://homes.cs.washington.edu/~shapiro/EE596/notes/Optical_Flow.pdf
15. Determining Constant Optical Flow - People | MIT CSAIL, 8월 17, 2025에 액세스, https://people.csail.mit.edu/bkph/articles/Fixed_Flow.pdf
16. Optical Flow Estimation, 8월 17, 2025에 액세스, http://www.cs.toronto.edu/~fleet/courses/2503/fall11/Handouts/opticalFlow.pdf
17. Lucas-Kanade in a Nutshell - Freie Universität Berlin, 8월 17, 2025에 액세스, http://www.inf.fu-berlin.de/inst/ag-ki/rojas_home/documents/tutorials/Lucas-Kanade2.pdf
18. homepages.inf.ed.ac.uk, 8월 17, 2025에 액세스, [https://homepages.inf.ed.ac.uk/rbf/CVonline/LOCAL_COPIES/OWENS/LECT12/node4.html#:~:text=Ixu%20%2B%20Iy,v%20of%20the%20optical%20flow.](https://homepages.inf.ed.ac.uk/rbf/CVonline/LOCAL_COPIES/OWENS/LECT12/node4.html#:~:text=Ixu %2B Iy,v of the optical flow.)
19. Lucas-Kanade Motion Estimation, 8월 17, 2025에 액세스, https://www.cs.washington.edu/education/courses/455/05wi/notes/LucasKanade.pdf
20. opticalFlowLK - Object for estimating optical flow using Lucas-Kanade method - MathWorks, 8월 17, 2025에 액세스, https://www.mathworks.com/help/vision/ref/opticalflowlk.html
21. Can someone explain the Lucas Kanade algorithm in plain English? I'm having a hard time grasping it. - Reddit, 8월 17, 2025에 액세스, https://www.reddit.com/r/computervision/comments/760f68/can_someone_explain_the_lucas_kanade_algorithm_in/
22. [Optical Flow/04] Lucas-Kanade Algorithm, Part I - Thread Through, 8월 17, 2025에 액세스, https://youngj-baek.github.io/cv_optical_flow/Lucas-Kanade-Algorithm-Part-1/
23. Lucas/Kanade Meets Horn/Schunck: Combining Local and Global Optic Flow Methods - Mathematical Image Analysis Group, 8월 17, 2025에 액세스, https://www.mia.uni-saarland.de/Publications/bruhn-ijcv05c.pdf
24. Large Displacement Detection Using Improved Lucas–Kanade Optical Flow - PMC, 8월 17, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC10058884/
25. Pyramidal Implementation of the Affine Lucas Kanade Feature ..., 8월 17, 2025에 액세스, http://robots.stanford.edu/cs223b04/algo_affine_tracking.pdf
26. The Lucas-Kanade method with pyramids, where the optical flow is... - ResearchGate, 8월 17, 2025에 액세스, https://www.researchgate.net/figure/The-Lucas-Kanade-method-with-pyramids-where-the-optical-flow-is-calculated-at-the-top-of_fig1_287025502
27. Lucas-Kanade Motion Estimation - University of Washington, 8월 17, 2025에 액세스, https://courses.cs.washington.edu/courses/cse576/06sp/notes/MotionLK.pdf
28. Revisiting Lucas-Kanade and Horn-Schunck, 8월 17, 2025에 액세스, https://www-2.dc.uba.ar/trabajosFinalesOrga2/2018_PAZOS_REYES/tp/reads/JCEI10004-20130715-145221-8984-16871.pdf
29. Motion Estimation - cs.wisc.edu, 8월 17, 2025에 액세스, https://pages.cs.wisc.edu/~dyer/cs766/slides/motion/motion-4up.pdf
30. Iterative Lucas-Kanade algorithm - Signal Processing Stack Exchange, 8월 17, 2025에 액세스, https://dsp.stackexchange.com/questions/13234/iterative-lucas-kanade-algorithm
31. Lecture 30: Video Tracking: Lucas-Kanade - Penn State, 8월 17, 2025에 액세스, https://www.cse.psu.edu/~rtc12/CSE486/lecture30.pdf
32. 14.4 Alignment - LucasKanade - Carnegie Mellon University, 8월 17, 2025에 액세스, https://www.cs.cmu.edu/~16385/s17/Slides/14.4_Alignment__LucasKanade.pdf
33. Lucas-Kanade 20 Years On: A Unifying Framework Part 1: The Quantity Approximated, the Warp Update Rule, and the Gradient Descent - CMU Robotics Institute, 8월 17, 2025에 액세스, https://www.ri.cmu.edu/pub_files/pub3/baker_simon_2004_1/baker_simon_2004_1.pdf
34. Horn-Schunck Method Explained - Number Analytics, 8월 17, 2025에 액세스, https://www.numberanalytics.com/blog/horn-schunck-method-explained
35. Comparison Between The Optical Flow Computational Techniques, 8월 17, 2025에 액세스, https://ijettjournal.org/assets/volume-4/issue-10/IJETT-V4I10P142.pdf
36. Horn-Schunck Optical Flow | Teaching myself Haskell - WordPress.com, 8월 17, 2025에 액세스, https://teachingmyselfhaskell.wordpress.com/2014/01/25/horn-schunck-optical-flow/
37. What is the difference between different Optical flow classes? - MATLAB Answers, 8월 17, 2025에 액세스, https://www.mathworks.com/matlabcentral/answers/381514-what-is-the-difference-between-different-optical-flow-classes
38. 014 Calculating Sparse Optical flow using Lucas Kanade method - Master Data Science, 8월 17, 2025에 액세스, https://datahacker.rs/calculating-sparse-optical-flow-using-lucas-kanade-method/
39. Optical Flow - OpenCV Documentation, 8월 17, 2025에 액세스, https://docs.opencv.org/3.4/d4/dee/tutorial_optical_flow.html
40. Video Stabilization Using Optical Flow - ijarcce, 8월 17, 2025에 액세스, https://ijarcce.com/wp-content/uploads/2025/06/IJARCCE.2025.14657.pdf
41. Robust video stabilisation algorithm using feature point selection and delta optical flow - QUT ePrints, 8월 17, 2025에 액세스, https://eprints.qut.edu.au/29612/1/IET_CV09.pdf
42. Real-Time Image Stabilization Method Based on Optical Flow and Binary Point Feature Matching - MDPI, 8월 17, 2025에 액세스, https://www.mdpi.com/2079-9292/9/1/198
43. Video stabilization using Lucas-Kanade method - Signal Processing Stack Exchange, 8월 17, 2025에 액세스, https://dsp.stackexchange.com/questions/40441/video-stabilization-using-lucas-kanade-method
44. Temporal Stereo Disparity Estimation with Graph Cuts - APSIPA, 8월 17, 2025에 액세스, http://www.apsipa.org/proceedings_2015/pdf/54.pdf
45. lucaskanade81.pdf - Computer Science, 8월 17, 2025에 액세스, https://cseweb.ucsd.edu/classes/sp02/cse252/lucaskanade81.pdf
46. Depth Aware Lucas Kanade Method - DiVA portal, 8월 17, 2025에 액세스, http://www.diva-portal.org/smash/get/diva2:1887877/FULLTEXT01.pdf
47. Lucas-Kanade Method Explained - Number Analytics, 8월 17, 2025에 액세스, https://www.numberanalytics.com/blog/lucas-kanade-method-explained

