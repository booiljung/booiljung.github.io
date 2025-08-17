# RC-Vox에서 브레제남 알고리즘



# RC-Vox 환경의 3D 복셀 그리드를 위한 브레제남 선 알고리즘 심층 분석





## I. 서론





### 0.1 복셀 패러다임의 부상과 직선 알고리즘의 중요성



현대 컴퓨터 그래픽스, 의료 영상, 물리 시뮬레이션, 컴퓨터 수치 제어(CNC) 등 다양한 첨단 기술 분야에서 복셀(Voxel, Volume Element) 기반의 공간 표현 방식이 핵심적인 패러다임으로 자리 잡고 있다.1 복셀은 3차원 공간을 균일한 격자(Grid)로 분할한 최소 단위로서, 복잡한 기하학적 형태나 불균일한 내부 데이터를 표현하는 데 있어 기존의 표면 기반 모델링(Surface-based Modeling) 방식이 갖는 한계를 극복하는 대안을 제시한다. 이러한 복셀 그리드 내에서 두 점을 연결하는 직선 경로를 효율적이고 정확하게 결정하는 것은 시스템의 성능과 기능성을 좌우하는 근본적인 연산이다. 이는 광선 투사(Ray-Casting), 가시선(Line-of-Sight) 검사, 인공지능의 경로 탐색, 그리고 절차적 콘텐츠 생성(Procedural Content Generation)과 같은 핵심 기능들의 근간을 이루기 때문이다.3

이러한 맥락에서, 브레제남 선 알고리즘은 단순한 '선 그리기' 기술을 넘어, 이산 공간(discrete space)을 탐색하는 근본적인 '순회(traversal)' 방법론으로 재해석될 수 있다. 본래 이 알고리즘의 목적은 디지털 플로터의 펜을 제어하거나 래스터 디스플레이의 픽셀을 활성화하여 시각적인 선을 '그리는' 것이었다.6 그러나 복셀 환경에서 '픽셀을 켠다'는 행위는 '특정 복셀을 선택하거나 방문한다'는 행위와 개념적으로 등가이다.3 따라서 알고리즘이 출력하는 선택된 픽셀 또는 복셀의 순차적 집합은 시작점과 끝점 사이를 잇는 이산적인 '경로'를 정의한다. 이 경로는 광선 투사의 경로를 근사하거나, 인공지능 캐릭터의 이동 경로를 설정하고, 물리 시뮬레이션에서 입자의 궤적을 추적하는 등 다양한 비-시각적 응용에 직접적으로 활용될 수 있다.4 결과적으로, 현대적 복셀 시스템에서 브레제남 알고리즘의 가치는 시각적 표현(rendering)만큼이나, 혹은 그 이상으로 공간적 계산(spatial computation)을 위한 효율적인 순회 경로 생성 능력에 있다.



### 0.2 연구의 목적 및 범위



본 보고서는 1965년 IBM의 Jack Bresenham에 의해 발표된 이 고전적이고 우아한 알고리즘이 3차원 복셀 그리드 환경, 특히 'RC-Vox'로 명명된 가상 시스템에 어떻게 적용되고 어떤 기술적 의미를 갖는지 심층적으로 분석하는 것을 목적으로 한다.6 본 보고서는 알고리즘의 근간을 이루는 2차원 수학적 원리부터 시작하여, 3차원 공간으로의 확장 과정, RC-Vox 환경에서의 구체적인 응용 사례, 그리고 주요 대안 알고리즘과의 성능 및 특성 비교를 통해 브레제남 알고리즘의 본질적인 효용성과 내재적 한계를 명확히 규명하고자 한다.



### 0.3 RC-Vox의 정의



본 보고서에서 'RC-Vox'는 특정 상용 제품을 지칭하는 것이 아니라, 복셀 데이터 구조를 기반으로 광선 투사(Ray-Casting) 및 공간 순회(Spatial Traversal) 연산이 빈번하게 발생하는 고성능 가상 시스템을 총칭하는 용어로 정의한다. 이는 `RVX - Retro VoXel graphics framework`와 같이 복셀 씬(scene)의 모델링, 렌더링, 조작을 위한 프레임워크의 핵심 원리를 차용한 개념적 모델이다.8 따라서 RC-Vox 환경은 실시간 렌더링, 동적 시뮬레이션, 상호작용적 콘텐츠 생성을 목표로 하며, 이러한 환경에서 브레제남 알고리즘의 역할과 성능을 분석하는 것은 매우 실질적인 의미를 가진다.



## 1. II. 2차원 브레제남 선 알고리즘의 수학적 원리





### 1.1  문제 정의: 래스터화의 본질



브레제남 선 알고리즘의 근본적인 과제는 두 개의 정수 좌표점 $(x_0, y_0)$와 $(x_1, y_1)$로 정의되는 이상적인 수학적 직선을 이산적인 픽셀 그리드 상에서 가장 잘 근사하는 픽셀들의 집합을 결정하는 것이다.6 알고리즘 설계의 핵심 목표는 이 과정을 부동소수점 연산의 부정확성과 비효율성을 완전히 배제하고, 오직 정수 덧셈, 뺄셈, 그리고 비트 시프트(bit shift) 연산만으로 수행하여 하드웨어 수준의 최고 속도를 달성하는 데 있다.6



### 1.2  직선의 암시적 방정식 (Implicit Equation of a Line)



알고리즘의 유도는 일반적으로 알려진 직선의 기울기-절편 형태(slope-intercept form)에서 시작한다.

```
$$y = mx + b$$
```

여기서 기울기 m=ΔxΔy=x1−x0y1−y0 이다. 이 식은 부동소수점인 기울기 m을 포함하므로 정수 연산에 적합하지 않다. 모든 계산을 정수 영역으로 가져오기 위해, 양변에 Δx를 곱하고 항들을 이항하여 다음과 같은 암시적 형태(implicit form)로 변환한다.6

```
$$F(x, y) = (\Delta y)x - (\Delta x)y + \Delta x b = 0$$
```

이 함수 $F(x, y)$는 점 $(x, y)$가 직선 상에 있을 때 정확히 0이 된다. 더 중요한 특성은 $F(x, y)$의 부호가 점 $(x, y)$가 직선의 어느 쪽에 위치하는지를 알려준다는 것이다. 예를 들어, Δx>0,Δy>0인 경우, F(x,y)<0인 점은 직선보다 위에(above) 위치하고, F(x,y)>0인 점은 직선보다 아래에(below) 위치한다.6 이 특성이 다음 픽셀을 선택하는 결정의 핵심적인 기반이 된다.



### 1.3  결정 변수의 유도 (Derivation of the Decision Variable)



알고리즘의 유도를 단순화하기 위해, 선의 기울기가 0과 1 사이(0≤m≤1)인 경우를 먼저 가정한다. 이 경우, 선은 x축을 따라 한 번에 한 픽셀씩 오른쪽으로 진행한다. 만약 현재 선택된 픽셀이 $(x_k, y_k)$라면, 다음 픽셀의 후보는 바로 오른쪽의 '동쪽(East)' 픽셀 $E(x_k+1, y_k)$와 대각선 위 오른쪽의 '북동쪽(North-East)' 픽셀 NE(xk+1,yk+1) 두 가지로 좁혀진다.11

어떤 후보 픽셀이 실제 직선에 더 가까운지를 판별하기 위한 기준으로, 두 후보 픽셀의 중간점(midpoint) $M(x_k+1, y_k+0.5)$을 사용한다.9 만약 실제 직선이 이 중간점 

M보다 아래를 지나간다면 E 픽셀이 더 가까울 것이고, M보다 위를 지나간다면 NE 픽셀이 더 가까울 것이다. 이 판별을 위해 중간점 M의 좌표를 앞서 유도한 암시적 방정식 $F(x, y)$에 대입한다. 이 값을 '결정 변수(decision variable)' dk로 정의한다.

```
$$d_k = F(x_k+1, y_k+0.5) = \Delta y(x_k+1) - \Delta x(y_k+0.5) + \Delta x b$$
```

$F(x, y)$의 부호 특성에 따라 dk의 부호로 다음 픽셀을 결정할 수 있다 6:

- 만약 dk<0 이면, 중간점 M이 직선보다 위에 있으므로 실제 선은 E에 더 가깝다. 따라서 다음 픽셀은 $E(x_k+1, y_k)$로 선택한다.
- 만약 dk≥0 이면, 중간점 M이 직선보다 아래에 있거나 직선 상에 있으므로 NE가 더 가깝다. 다음 픽셀은 $NE(x_k+1, y_k+1)$로 선택한다.



### 2.4. 결정 변수의 점진적 갱신 (Incremental Update of the Decision Variable)



매 단계마다 dk를 위 공식에 따라 새로 계산하는 것은 비효율적이다. 대신, $d_{k+1}$을 dk로부터 점진적으로(incrementally) 계산하는 갱신 규칙을 유도한다.

**경우 1 (E 선택 시):** 다음 픽셀이 $(x_{k+1}, y_{k+1}) = (x_k+1, y_k)$인 경우.

```
$$
\begin{align*}
d_{k+1} &= F(x_{k+1}+1, y_{k+1}+0.5) = F((x_k+1)+1, y_k+0.5) \\
&= \Delta y(x_k+2) - \Delta x(y_k+0.5) + \Delta x b \\
&= + \Delta y \\
&= d_k + \Delta y
\end{align*}
$$
```

**경우 2 (NE 선택 시):** 다음 픽셀이 $(x_{k+1}, y_{k+1}) = (x_k+1, y_k+1)$인 경우.

```
$$
\begin{align*}
d_{k+1} &= F(x_{k+1}+1, y_{k+1}+0.5) = F((x_k+1)+1, (y_k+1)+0.5) \\
&= \Delta y(x_k+2) - \Delta x(y_k+1.5) + \Delta x b \\
&= + \Delta y - \Delta x \\
&= d_k + \Delta y - \Delta x
\end{align*}
$$
```

이로써 매 단계에서 간단한 덧셈만으로 결정 변수를 갱신할 수 있게 되었다.



### 2.5. 정수 연산으로의 최종 변환



아직 해결해야 할 두 가지 문제가 남아있다: 초기 결정 변수 d0의 계산과, dk의 정의에 포함된 소수 0.5이다.

초기 결정 변수 d0는 $(x_0, y_0)$에서 시작할 때 계산된다.

```
$$d_0 = F(x_0+1, y_0+0.5) = \Delta y(x_0+1) - \Delta x(y_0+0.5) + \Delta x b$$
```

시작점 $(x_0, y_0)$는 직선 위의 점이므로 F(x0,y0)=Δyx0−Δxy0+Δxb=0이다. 이를 이용하여 식을 정리하면 다음과 같다.

```
$$d_0 = (\Delta y x_0 - \Delta x y_0 + \Delta x b) + \Delta y - 0.5 \Delta x = \Delta y - 0.5 \Delta x$$
```

여전히 존재하는 소수 0.5를 제거하기 위해, 결정 변수 dk와 모든 갱신식에 2를 곱한다. 이 새로운 정수 결정 변수를 Dk=2dk로 정의하자. 이 변환은 부호에 영향을 주지 않으므로 결정 과정은 동일하게 유지된다.6

- **초기값:** D0=2d0=2(Δy−0.5Δx)=2Δy−Δx
- **E 선택 시 갱신:** Dk+1=Dk+2Δy
- **NE 선택 시 갱신:** Dk+1=Dk+2Δy−2Δx

이제 초기값 계산과 모든 점진적 갱신 과정이 오직 정수 연산만으로 완벽하게 수행된다.9

브레제남 알고리즘의 진정한 혁신은 '오차의 누적과 보상'이라는 개념을 정수 연산으로 완벽하게 구현한 데 있다. 이는 단순한 반올림 트릭이 아니라, 매 단계의 결정이 이전까지 누적된 오차를 고려하여 다음 단계의 오차를 최소화하는 방향으로 이루어지는 동적인 프로세스다. 직선의 방정식 y=mx+b에서 x가 1 증가할 때 y는 기울기 m만큼 증가한다.13 매 단계에서 y값을 반올림하는 것은 비효율적인 부동소수점 연산을 요구한다.14 브레제남은 y의 실제 값과 픽셀 그리드 값 사이의 '오차(error)'를 추적하는 아이디어를 도입했다. 이 오차가 0.5를 넘어서면 y 좌표를 1 증가시키고, 오차 값에서 1.0을 빼서 보상하는 방식이다.9

`error += m`이라는 부동소수점 연산을 정수화하기 위해 양변에 2⋅Δx를 곱하면, `new_error = error * 2 * \Delta x`가 되고, 갱신식은 `new_error += 2 * \Delta y`가 된다. `error >= 0.5`라는 비교 조건은 `2 \cdot \Delta x \cdot error \ge \Delta x`로 변환되며, 이는 `2 \cdot \Delta x \cdot error - \Delta x \ge 0`과 같다. 여기서 `2 \cdot \Delta x \cdot error - \Delta x`가 바로 우리가 유도한 결정 변수 D와 동일한 형태를 띤다. 결론적으로, 결정 변수 D는 실제 직선과 현재 선택된 픽셀 경로 사이의 누적된 오차를 2⋅Δx 배율로 확대한 정수 값이다. D를 갱신하는 과정은 이 누적 오차에 다음 단계의 오차(즉, m에 비례하는 2Δy)를 더하고, y 좌표가 조정될 때 오차를 보상(즉, −1.0에 비례하는 −2Δx)하는 과정을 정수 연산만으로 완벽하게 시뮬레이션하는 것이다. 이는 단순한 최적화를 넘어, 이산 공간에서의 오차 제어에 대한 근본적인 통찰을 보여준다.



### 2.6. 일반화: 8분면 대칭성 (Generalization: Octant Symmetry)



지금까지의 유도는 기울기가 0≤m≤1인, 즉 1사분면의 첫 번째 8분면(octant)에 대해서만 유효하다.9 다른 7개의 8분면에 위치한 직선을 처리하기 위해, 좌표축을 교환하거나(

x↔y), 좌표의 증감 방향을 바꾸는 등의 대칭 변환을 적용한다. 예를 들어, 기울기가 1보다 큰 경우(m>1), y축이 x축보다 더 빠르게 변하므로 y를 주축으로 삼아 한 단계씩 증가시키고, x의 증가 여부를 결정 변수로 판단한다. 이러한 대칭성을 활용함으로써, 모든 방향의 직선을 동일한 핵심 로직을 재사용하여 효율적으로 처리할 수 있다.10



## III. 3차원 복셀 그리드로의 확장: 주축과 다중 결정 변수





### 3.1. 3차원 확장 개념: 주축의 도입



2차원 평면에서 3차원 공간으로 알고리즘을 확장할 때, 가장 큰 난관은 세 개의 좌표축(X, Y, Z)이 동시에 변화한다는 점이다. 2차원에서처럼 하나의 축을 기준으로 다른 축의 변화를 결정하는 단순한 방식은 일반화하기 어렵다. 이 문제를 해결하기 위한 핵심 아이디어는 '주축(Dominant Axis)'을 선정하는 것이다. 주축은 직선의 시작점과 끝점 사이의 좌표 변화량, 즉 Δx,Δy,Δz의 절댓값 중 가장 큰 값을 갖는 축으로 정의된다.10

알고리즘의 기본 전략은 이 주축을 따라 항상 1 복셀씩 일정하게 전진하는 것이다. 나머지 두 '비주축(Non-dominant Axes)'은 2차원 알고리즘에서의 y축과 유사한 역할을 수행한다. 즉, 각각 독립적인 결정 변수를 통해 현재 좌표를 유지할지, 아니면 1씩 증감할지를 매 단계마다 결정하게 된다.18 이 방식을 통해 3차원 직선의 복잡한 움직임을 세 개의 축에 대한 단순한 결정들의 조합으로 분해할 수 있다.



### 3.2. 다중 결정 변수의 정의와 갱신



X축이 주축이라고 가정하자 (즉, ∣Δx∣≥∣Δy∣ 이고 ∣Δx∣≥∣Δz∣). 이 경우, 두 개의 독립적인 결정 변수가 필요하다. 하나는 X-Y 평면에서의 투영된 선을 처리하기 위한 Dy이고, 다른 하나는 X-Z 평면에서의 투영된 선을 처리하기 위한 Dz이다.18

각 결정 변수는 2차원 알고리즘과 완전히 동일한 원리로 유도된다. 주축인 X가 2차원의 x 역할을 하고, 비주축인 Y와 Z가 각각 2차원의 y 역할을 한다.

- **초기값:**

  Code snippet

  ```
  $$
  D_{y,0} = 2|\Delta y| - |\Delta x| \\
  D_{z,0} = 2|\Delta z| - |\Delta x|
  $$
  ```

  (여기서 Δx,Δy,Δz는 부호를 포함한 값이지만, 유도 편의상 절댓값을 사용한 형태로 표현하였으며 실제 구현에서는 부호를 고려해야 한다.)

- 점진적 갱신:

  매 단계에서 X 좌표는 부호에 따라 1씩 증감한다. Y와 Z 좌표의 갱신은 각각 Dy와 Dz의 부호에 따라 독립적으로 결정된다.

  - Y 좌표 갱신:

    만약 Dy≥0 이면, Y 좌표를 부호에 따라 1 증감시키고, Dy를 Dy−2∣Δx∣ 로 갱신한다. 그 후, Dy에 $2|\Delta y|$를 더한다. 이를 하나의 식으로 표현하면 다음과 같다.

    - If Dy≥0: y←y+sign(Δy), Dy←Dy+2∣Δy∣−2∣Δx∣
    - Else: Dy←Dy+2∣Δy∣

  - Z 좌표 갱신:

    Y 좌표와 동일한 논리가 적용된다.

    - If Dz≥0: z←z+sign(Δz), Dz←Dz+2∣Δz∣−2∣Δx∣
    - Else: Dz←Dz+2∣Δz∣

이러한 갱신 규칙은 3차원 직선 그리기를 두 개의 독립적인 2차원 문제로 효과적으로 분해하여 해결하는 방식임을 보여준다.18

3D 브레제남 알고리즘은 본질적으로 2D 알고리즘을 두 개의 독립적인 평면(주축-비주축1, 주축-비주축2)에 투영하여 동시에 실행하는 것과 같다. 이 구조는 경로의 토폴로지 특성을 결정하며, '26-연결성'이라는 중요한 속성으로 귀결된다. 알고리즘은 주축을 따라 한 단계 전진할 때(예: x가 x+1로), y 좌표의 변화는 오직 x와 y의 관계(Dy)에 의해서만 결정되며 z의 값은 이에 영향을 주지 않는다. 마찬가지로, z 좌표의 변화는 오직 x와 z의 관계(Dz)에 의해서만 결정된다. 이는 마치 X-Y 평면에 2D 브레제남을 적용하고, 동시에 X-Z 평면에 또 다른 2D 브레제남을 적용하여 각 평면에서 얻은 y와 z 좌표를 조합하는 것과 같다. "직선이 두 평면의 움직임으로 분해된다"는 표현이 이를 뒷받침한다.20 한 단계에서 주축 좌표는 1만큼 변하고, 두 비주축 좌표는 각각 0 또는 1만큼 변할 수 있다. 가능한 모든 변화 조합은 

(±1,δy,δz) 형태이며, 여기서 $\delta y, \delta z \in {-1, 0, 1}$이다. 이는 복셀 그리드에서 한 복셀과 면, 모서리, 또는 꼭짓점을 공유하는 모든 이웃(총 26개)으로의 이동을 포함한다. 따라서, 알고리즘의 구조 자체가 필연적으로 26-연결성(26-connected) 경로를 생성하게 된다.1



### 3.3. 3D 브레제남 알고리즘 의사 코드



다음은 (x1, y1, z1)에서 (x2, y2, z2)까지의 복셀을 순회하는 3D 브레제남 알고리즘의 일반화된 의사 코드이다.18

**1. 초기화:**

```
dx = abs(x2 - x1)
dy = abs(y2 - y1)
dz = abs(z2 - z1)

xs = sign(x2 - x1)
ys = sign(y2 - y1)
zs = sign(z2 - z1)

x, y, z = x1, y1, z1
ListOfPoints.add(x, y, z)
```

**2. 주축에 따른 분기:**

```
// X축이 주축인 경우
if (dx >= dy && dx >= dz) {
    p1 = 2*dy - dx  // Y에 대한 결정 변수
    p2 = 2*dz - dx  // Z에 대한 결정 변수
    while (x!= x2) {
        x += xs
        if (p1 >= 0) {
            y += ys
            p1 -= 2*dx
        }
        if (p2 >= 0) {
            z += zs
            p2 -= 2*dx
        }
        p1 += 2*dy
        p2 += 2*dz
        ListOfPoints.add(x, y, z)
    }
} 
// Y축이 주축인 경우
else if (dy >= dx && dy >= dz) {
    p1 = 2*dx - dy
    p2 = 2*dz - dy
    while (y!= y2) {
        y += ys
        if (p1 >= 0) {
            x += xs
            p1 -= 2*dy
        }
        if (p2 >= 0) {
            z += zs
            p2 -= 2*dy
        }
        p1 += 2*dx
        p2 += 2*dz
        ListOfPoints.add(x, y, z)
    }
}
// Z축이 주축인 경우
else {
    p1 = 2*dy - dz
    p2 = 2*dx - dz
    while (z!= z2) {
        z += zs
        if (p1 >= 0) {
            y += ys
            p1 -= 2*dz
        }
        if (p2 >= 0) {
            x += xs
            p2 -= 2*dz
        }
        p1 += 2*dy
        p2 += 2*dx
        ListOfPoints.add(x, y, z)
    }
}
```



## IV. RC-Vox 환경에서의 적용: 복셀 순회와 성능 분석





### 4.1. 선 그리기에서 복셀 순회로



RC-Vox와 같은 현대적 복셀 엔진에서 3D 브레제남 알고리즘의 주된 용도는 화면에 시각적인 선을 그리는 것을 넘어, 두 점 사이의 경로를 구성하는 복셀의 이산적 시퀀스를 식별하는 '복셀 순회(Voxel Traversal)'에 있다.3 알고리즘이 생성하는 복셀의 목록은 그 자체로 중요한 공간 정보이며, 다양한 고급 기능의 기반이 된다.

- **광선 투사(Ray Casting):** 광선이 통과하는 복셀들을 순서대로 방문하여 가장 먼저 만나는 불투명 복셀을 찾아내는 데 사용된다. 이는 렌더링, 객체 선택, 환경 탐지 등의 핵심 기술이다.5
- **가시선(Line of Sight, LOS) 확인:** 두 유닛(예: 게임 캐릭터) 사이에 시야를 가리는 장애물이 있는지 검사하기 위해, 두 점을 잇는 경로상의 모든 복셀이 비어있는지(투명한지) 확인하는 데 사용된다.3
- **절차적 생성(Procedural Generation):** 강, 길, 동굴과 같은 자연 지형지물을 생성할 때, 시작점과 끝점을 연결하는 복셀 경로를 생성하고 이를 기반으로 지형을 변형시키는 데 활용될 수 있다.
- **물리 및 충돌 감지:** 발사체의 궤적을 시뮬레이션하거나, 두 움직이는 객체 사이의 충돌이 일어나는 복셀을 예측하는 등 실시간 물리 계산에 적용될 수 있다.4



### 4.2. 경로의 특성: 26-연결성과 그 의미



앞서 수학적 원리에서 분석했듯이, 3D 브레제남 알고리즘은 필연적으로 26-연결성(26-connected) 경로를 생성한다.1 이는 경로를 구성하는 연속된 두 복셀이 최소한 하나의 꼭짓점(vertex)을 공유함을 의미하며, 대각선 방향으로의 이동을 포함한다. 이러한 특성은 장점과 단점을 동시에 가진다.

- **장점:** 경로는 항상 끊어짐 없이 연결됨을 보장한다. 이는 얇은 벽이나 작은 장애물을 광선이 관통하는 '터널링(tunneling)' 현상을 방지하는 데 도움이 된다. 경로는 항상 '두껍게' 유지되어 연결성이 필요한 응용(예: 특정 영역이 완전히 차단되었는지 확인)에 신뢰성을 제공한다.1
- **단점:** 최소한의 복셀만 통과하는 6-연결성(6-connected, 면으로만 인접) 경로에 비해 더 많은 복셀을 방문하게 되어 연산량이 증가할 수 있다. 특히, 대각선 이동을 허용하지 않는 특정 물리 모델이나 게임 로직(예: 격자 기반 이동)과는 부합하지 않을 수 있다.3



### 4.3. 성능 분석: 정수 연산의 압도적 이점



브레제남 알고리즘의 가장 지속적이고 강력한 장점은 모든 계산이 하드웨어에서 가장 빠르게 처리되는 정수 덧셈, 뺄셈, 비트 시프트 연산으로만 구성된다는 점이다.6 이는 부동소수점 곱셈, 나눗셈, 그리고 반올림 연산이 필수적인 DDA(Digital Differential Analyzer)나 다른 부동소수점 기반 알고리즘에 비해 CPU 사이클을 극적으로 절약한다.15

이러한 특성 덕분에 브레제남 알고리즘은 저수준 하드웨어 제어 6 뿐만 아니라, 초당 수백만 번 이상의 순회 연산이 필요한 고성능 실시간 시뮬레이션 및 게임 엔진에서 여전히 강력한 경쟁력을 유지한다. 일부 연구에서는 특정 조건 하에서 브레제남이 Amanatides-Woo와 같은 알고리즘보다 20-500% 느렸다는 결과를 보고하기도 했지만 23, 이는 순수한 CPU 연산 속도 외에 메모리 접근 패턴, 분기 예측 실패, 캐시 미스 등 복합적인 요인이 작용한 결과로 해석될 수 있다. 알고리즘의 핵심 루프에 포함된 순수 산술 연산의 복잡도와 비용 측면에서는 브레제남이 명백한 우위를 점한다.

3D 브레제남 알고리즘의 '주축' 기반 접근법은 그 자체로 비등방성(anisotropic) 특성을 내포한다. 이는 생성된 경로가 좌표축에 대해 대칭적이지 않음을 의미하며, 특정 응용에서 미묘하지만 중요한 편향(bias)을 유발할 수 있다. 예를 들어, $(x_1, y_1, z_1)$에서 $(x_2, y_2, z_2)$로 가는 경로와, 좌표를 교환한 $(y_1, x_1, z_1)$에서 $(y_2, x_2, z_2)$로 가는 경로는 완전히 다른 복셀 집합을 생성할 수 있다. 그 원인은 '주축' 선택에 있다. 첫 번째 경우 $|\Delta x|$가 가장 커서 X가 주축이 될 수 있지만, 두 번째 경우 $|\Delta y|$가 가장 커서 Y가 주축이 될 수 있기 때문이다. 주축이 달라지면 알고리즘의 기본 스텝 방향과 오차 누적 방식이 근본적으로 변하여, 결과적인 복셀 경로도 달라진다. 이는 RC-Vox 환경에서 플레이어가 $(0,0,0)$에서 $(10, 5, 5)$를 향해 발사체를 쏘는 것과 $(0,0,0)$에서 $(5, 10, 5)$를 향해 쏘는 것이 물리적으로는 동일한 각도의 회전 변환임에도 불구하고, 브레제남 알고리즘은 서로 다른 복셀 경로를 생성할 수 있음을 의미한다. 이는 게임플레이에서 비일관적인 충돌 판정이나 예측 불가능한 동작으로 이어질 수 있으므로, 개발자는 이러한 비등방성 특성을 인지하고 필요에 따라 경로를 정규화하거나 다른 대칭적인 알고리즘의 사용을 고려해야 한다.



## V. 대안 알고리즘과의 비교 고찰





### 5.1. Amanatides-Woo 복셀 순회 알고리즘



1987년 John Amanatides와 Andrew Woo에 의해 제안된 이 알고리즘은 3D 그리드 순회를 위해 특별히 설계되었으며, 광선 추적(Ray Tracing) 분야에서 널리 사용된다.7

- **작동 원리:** 이 알고리즘은 광선을 파라메트릭 방정식 $P(t) = \text{origin} + t \cdot \text{direction}$으로 표현한다. 알고리즘의 핵심은 광선이 복셀 그리드를 구성하는 X, Y, Z축에 평행한 경계면들을 통과하는 파라미터 t 값들을 계산하는 것이다. 매 반복 단계에서, 현재 복셀에서 다음 X, Y, Z 경계면까지 도달하는 데 필요한 t의 증가량(`tMaxX`, `tMaxY`, `tMaxZ`) 중 가장 작은 값을 선택한다. 이 선택에 따라 해당 축 방향으로 한 복셀만큼 전진하고, 해당 축의 `tMax` 값을 갱신한다.7

- **특징:** 이 방식은 브레제남과 달리 '주축' 개념 없이 모든 축을 동등하게 취급하며, 광선의 실제 기하학적 경로를 더 정확하게 따른다.7 그러나 파라미터 

  t 값을 계산하고 비교하는 과정에서 부동소수점 연산이 필수적이다.7



### 5.2. 3D 브레제남 vs. Amanatides-Woo



두 알고리즘은 3D 복셀 순회를 위한 대표적인 방법론이지만, 설계 철학과 특성에서 명확한 차이를 보인다.

- **정확성 대 속도:** 브레제남은 모든 연산을 정수로 처리하여 순수 CPU 연산 속도에서 우위를 점하지만, 주축 기반의 단계적 진행으로 인해 실제 광선 경로에서 미세하게 벗어나는 근사치를 생성한다. 반면, Amanatides-Woo는 부동소수점 연산으로 인해 순수 연산 비용은 더 높을 수 있으나, 광선이 통과하는 모든 복셀을 기하학적으로 정확하게 식별한다.23
- **경로 특성:** 브레제남은 26-연결성의 '두꺼운' 경로를 생성하는 반면, Amanatides-Woo는 구현에 따라 면으로만 인접한 6-연결성 경로 또는 모든 접촉 복셀을 포함하는 supercover(26-연결성과 유사) 경로를 생성할 수 있다. 일반적으로 Amanatides-Woo는 더 '얇고' 최소한의 경로를 생성하는 데 유리하다.1
- **구현 복잡도:** 3D 브레제남은 주축에 따른 3가지 경우와 각 경우에 대한 8분면 대칭성을 모두 고려해야 하므로 구현이 다소 복잡하고 장황해질 수 있다. Amanatides-Woo는 초기화 과정에서 부동소수점 계산이 다소 복잡하지만, 메인 순회 루프는 개념적으로 더 간단하고 일반적이어서 확장성이 좋다.7



### 5.3. 기타 대안: Xiaolin Wu의 안티에일리어싱 알고리즘



시각적 품질이 최우선 순위인 렌더링 응용에서는 Xiaolin Wu의 안티에일리어싱(Anti-aliasing) 라인 알고리즘을 고려할 수 있다.25 이 알고리즘은 브레제남과 같이 이산적인 픽셀/복셀을 선택하는 것이 아니라, 이상적인 직선에 인접한 두 픽셀(복셀)을 모두 그리되, 직선과의 거리에 비례하는 명암(또는 투명도) 값을 각각에 적용한다.26 이를 통해 계단 현상(aliasing)이 완화된 부드러운 경계선을 표현할 수 있다. 하지만 이는 순회나 공간 계산 목적이 아닌, 고품질 렌더링을 위한 후처리 기법에 가까우며, 브레제남이나 Amanatides-Woo와는 근본적으로 다른 목적을 가진다.



### 표 1: 3D 라인 알고리즘 비교



다음 표는 본 보고서에서 논의된 주요 3D 라인 관련 알고리즘들의 핵심적인 특성을 요약하고 비교하여, 특정 응용 시나리오에 가장 적합한 알고리즘을 신속하게 판단할 수 있도록 돕는다.

| 특성 (Feature)     | 3D 브레제남 (3D Bresenham)    | Amanatides-Woo                         | Xiaolin Wu (3D 확장 시)      |
| ------------------ | ----------------------------- | -------------------------------------- | ---------------------------- |
| **주요 목적**      | 빠른 복셀 순회, 라인 래스터화 | 정확한 광선-복셀 순회 (Ray Traversal)  | 고품질 안티에일리어싱 렌더링 |
| **연산 방식**      | 정수 덧셈/뺄셈/시프트 전용    | 부동소수점 덧셈/비교/곱셈/나눗셈       | 부동소수점 연산              |
| **성능**           | 매우 높음 (CPU 연산 기준)     | 중간~높음 (GPU 친화적)                 | 상대적으로 낮음              |
| **경로 정확도**    | 근사치 (주축에 편향됨)        | 높음 (실제 광선 경로에 충실)           | 시각적 정확도에 초점         |
| **연결성**         | 26-연결성 (두꺼운 경로)       | 6-연결성 또는 Supercover(26) 선택 가능 | 해당 없음 (픽셀 밝기 조절)   |
| **구현 복잡도**    | 중간 (주축 및 8분면 처리)     | 중간 (초기화 로직)                     | 높음 (밝기 계산 추가)        |
| **주요 인용 자료** | 18                            | 7                                      | 25                           |



## VI. 결론 및 제언





### 연구 요약



본 보고서는 3D 브레제남 선 알고리즘이 반세기가 넘는 시간에도 불구하고 RC-Vox와 같은 현대적 복셀 환경에서 여전히 강력하고 유효한 도구임을 심층적으로 분석하고 입증했다. 알고리즘의 핵심인 정수 연산 기반의 결정 변수와 점진적 갱신 방식은 탁월한 연산 속도를 보장한다. 2D 원리의 수학적 유도를 통해 결정 변수가 누적된 오차를 정수 형태로 관리하는 메커니즘임을 밝혔으며, 주축 개념을 통한 3D 확장이 필연적으로 26-연결성 경로라는 고유한 토폴로지 특성을 낳음을 규명했다.



### 알고리즘의 효용성과 한계



- **효용성:** 브레제남 알고리즘은 순수 정수 연산에 기반한 압도적인 속도를 바탕으로, 내부적인 대량 계산, 수많은 인공지능 에이전트의 경로 탐색, 빠른 시각화 프로토타이핑 등 연산 속도가 최우선시되고 완벽한 기하학적 정확성이 요구되지 않는 시나리오에서 최적의 선택이 될 수 있다.
- **한계:** 그러나 주축 기반 경로 생성 방식에서 비롯되는 비등방성 편향과 실제 광선 경로와의 미세한 불일치는, 정밀한 광선 추적 렌더링이나 물리적으로 정확한 충돌 감지 애플리케이션에서는 무시할 수 없는 단점으로 작용할 수 있다.



### 실용적 제언



RC-Vox 시스템 개발자는 단일 알고리즘을 모든 상황에 적용하기보다, 응용 프로그램의 구체적인 요구사항에 따라 알고리즘을 선택적으로 사용하는 하이브리드 접근법을 채택할 것을 강력히 제언한다.

- **정밀성이 요구되는 경우 (예: 렌더링을 위한 1차 광선, 픽셀 단위의 정확한 충돌 판정):** 부동소수점 연산의 비용을 감수하더라도 기하학적으로 더 정확한 **Amanatides-Woo 알고리즘**을 사용하는 것이 바람직하다.
- **속도가 최우선인 경우 (예: 수백 개 AI 유닛의 실시간 경로 계산, 간단한 파티클 효과, 근사적인 가시선 확인):** **3D 브레제남 알고리즘**이 가장 효율적인 솔루션을 제공할 것이다.
- **시각적 품질이 중요한 경우 (예: 최종 렌더링 결과물):** **Xiaolin Wu**와 같은 안티에일리어싱 기법을 앞서 선택된 순회 알고리즘 위에 추가적으로 적용하여 시각적 완성도를 높이는 방안을 고려할 수 있다.



### 향후 연구 방향



본 보고서에서 다룬 고전적인 알고리즘들을 현대 GPU 아키텍처의 대규모 병렬 처리 능력(예: Compute Shader)에 최적화하여 구현하고, 실제 RC-Vox와 유사한 환경에서 대규모 복셀 데이터셋을 대상으로 실증적인 성능을 비교 분석하는 연구는 중요한 후속 과제이다. 또한, 두께를 가진 선(thick line)이나 원통을 복셀화하는 문제에 브레제남의 오차 누적 및 보상 원리를 창의적으로 적용하여 효율적인 알고리즘을 개발하는 연구 역시 흥미로운 향후 연구 방향이 될 것이다.28





#### **Works cited**

1. [2302.04031] Fast and Robust Lidar-Inertial Odometry by Tightly-Coupled Iterated Kalman Smoother and Robocentric Voxels - ar5iv, accessed August 4, 2025, https://ar5iv.labs.arxiv.org/html/2302.04031
2. FR-LIO: Fast and Robust Lidar-Inertial Odometry by Tightly-Coupled ..., accessed August 4, 2025, https://arxiv.org/pdf/2302.04031
3. VOX-LIO: An Effective and Robust LiDAR-Inertial Odometry System Based on Surfel Voxels, accessed August 4, 2025, https://www.mdpi.com/2072-4292/17/13/2214
4. Mergeable Probabilistic Voxel Mapping for LiDAR–Inertial–Visual Odometry - MDPI, accessed August 4, 2025, https://www.mdpi.com/2079-9292/14/11/2142
5. Bresenham algorithm - Deepnight Games, accessed August 4, 2025, https://deepnight.net/tutorial/bresenham-magic-raycasting-line-of-sight-pathfinding/
6. 3d "Greedy" Bresenham's Line Algorithm - Stack Overflow, accessed August 4, 2025, https://stackoverflow.com/questions/65753807/3d-greedy-bresenhams-line-algorithm
7. Ray Casting in 2D Grids - theshoemaker, accessed August 4, 2025, https://theshoemaker.de/posts/ray-casting-in-2d-grids
8. FR-LIO: Fast and Robust Lidar-Inertial Odometry by Tightly-Coupled Iterated Kalman Smoother and Robocentric Voxels - ResearchGate, accessed August 4, 2025, https://www.researchgate.net/publication/368361497_FR-LIO_Fast_and_Robust_Lidar-Inertial_Odometry_by_Tightly-Coupled_Iterated_Kalman_Smoother_and_Robocentric_Voxels
9. Bresenham's line algorithm - Wikipedia, accessed August 4, 2025, [https://en.wikipedia.org/wiki/Bresenham%27s_line_algorithm](https://en.wikipedia.org/wiki/Bresenham's_line_algorithm)
10. Bresenham's Line Algorithm | mbedded.ninja, accessed August 4, 2025, https://blog.mbedded.ninja/programming/algorithms-and-data-structures/bresenhams-line-algorithm/
11. Bresenham's Line Generation Algorithm - GeeksforGeeks, accessed August 4, 2025, https://www.geeksforgeeks.org/dsa/bresenhams-line-generation-algorithm/
12. BRESENHAM'S LINE DRAWING ALGORITHM | PPTX - SlideShare, accessed August 4, 2025, https://www.slideshare.net/slideshow/bresenhams-line-drawing-algorithm-150115027/150115027
13. Bresenham's Algorithm for 3-D Line Drawing - GeeksforGeeks, accessed August 4, 2025, https://www.geeksforgeeks.org/python/bresenhams-algorithm-for-3-d-line-drawing/
14. C++ Bresenham 3d Line Drawing Algorithm - GitHub Gist, accessed August 4, 2025, https://gist.github.com/yamamushi/5823518
15. Three-dimensional extension of Bresenham's algorithm and its application in straight-line interpolation | Request PDF - ResearchGate, accessed August 4, 2025, https://www.researchgate.net/publication/245386302_Three-dimensional_extension_of_Bresenham's_algorithm_and_its_application_in_straight-line_interpolation
16. A Faster Voxel Traversal Algorithm For Ray Tracing | PDF - Scribd, accessed August 4, 2025, https://www.scribd.com/document/6913705/A-faster-voxel-traversal-algorithm-for-ray-tracing
17. A Fast Voxel Traversal Algorithm for Ray Tracing, accessed August 4, 2025, http://www.cse.yorku.ca/~amana/research/grid.pdf
18. Cast ray to select block in voxel game - Game Development Stack Exchange, accessed August 4, 2025, https://gamedev.stackexchange.com/questions/47362/cast-ray-to-select-block-in-voxel-game
19. (PDF) A Fast Voxel Traversal Algorithm for Ray Tracing - ResearchGate, accessed August 4, 2025, https://www.researchgate.net/publication/2611491_A_Fast_Voxel_Traversal_Algorithm_for_Ray_Tracing
20. A Fast Voxel Traversal Algorithm for Ray Tracing - Portal de Periódicos da CAPES, accessed August 4, 2025, https://www.periodicos.capes.gov.br/index.php/acervo/buscador.html?task=detalhes&id=W1532701078
21. A Comparison of Optimal Scanline Voxelization ... - DiVA portal, accessed August 4, 2025, https://www.diva-portal.org/smash/get/diva2:1459330/FULLTEXT01.pdf
22. Bresenham's line algorithm: An exploration - Middle Engine Software, accessed August 4, 2025, https://www.middle-engine.com/blog/posts/2020/07/28/bresenhams-line-algorithm
23. Floating Point Demystified, Part 1 - Josh Haberman, accessed August 4, 2025, https://blog.reverberate.org/2014/09/what-every-computer-programmer-should.html
24. Fast and Robust LiDAR-Inertial Odometry by Tightly-Coupled Iterated Kalman Smoother and Robocentric Voxels - ResearchGate, accessed August 4, 2025, https://www.researchgate.net/publication/380301592_Fast_and_Robust_LiDAR-Inertial_Odometry_by_Tightly-Coupled_Iterated_Kalman_Smoother_and_Robocentric_Voxels
25. Two-Level Grids for Ray Tracing on GPUs - Computer Science, accessed August 4, 2025, https://webhome.cs.uvic.ca/~blob/downloads/tyler/paper1227_final.pdf
26. An Integer One-Pass Algorithm for Voxel Traversal - Bohrium, accessed August 4, 2025, https://www.bohrium.com/paper-details/an-integer-one-pass-algorithm-for-voxel-traversal/812344320181927936-2661
27. Why floating point operations are slower than integer? : r/compsci - Reddit, accessed August 4, 2025, https://www.reddit.com/r/compsci/comments/iioe7j/why_floating_point_operations_are_slower_than/
28. Why Bresenham's line algorithm more efficent then Naive algorithm - Stack Overflow, accessed August 4, 2025, https://stackoverflow.com/questions/8651947/why-bresenhams-line-algorithm-more-efficent-then-naive-algorithm
29. Floating point vs integer calculations on modern hardware - Stack Overflow, accessed August 4, 2025, https://stackoverflow.com/questions/2550281/floating-point-vs-integer-calculations-on-modern-hardware
30. Is there any performance benefit to using int vs. float on modern systems? - Reddit, accessed August 4, 2025, https://www.reddit.com/r/C_Programming/comments/12s8ede/is_there_any_performance_benefit_to_using_int_vs/
31. Mastering Computer Graphics: Bresenham's Line Algorithm Unlocked! - Medium, accessed August 4, 2025, https://medium.com/@PradumnaVerma/mastering-computer-graphics-bresenhams-line-algorithm-unlocked-965ef3b6ac97
32. Patterns | GPU branch divergence, accessed August 4, 2025, https://co-design.pop-coe.eu/patterns/gpu-branch-divergence.html
33. Reducing branch divergence to speed up parallel execution of unit testing on GPUs - Iowa State University, accessed August 4, 2025, [https://swapp.cs.iastate.edu/files/inline-files/bagies_ea_%20JS-May-2023-parallel%20execution%20of%20unit%20testing%20on%20GPUs.pdf](https://swapp.cs.iastate.edu/files/inline-files/bagies_ea_ JS-May-2023-parallel execution of unit testing on GPUs.pdf)
34. Branch handling on GPUs--why does screen partitioning help so much? : r/shaders - Reddit, accessed August 4, 2025, https://www.reddit.com/r/shaders/comments/18vibwf/branch_handling_on_gpuswhy_does_screen/
35. Is branch divergence really so bad? - Stack Overflow, accessed August 4, 2025, https://stackoverflow.com/questions/17223640/is-branch-divergence-really-so-bad
36. Efficient voxelization using projected optimal scanline - Tianjia Shao, accessed August 4, 2025, https://tianjiashao.com/Docs/2017/voxelization.pdf
37. Characterizing Perspective Error in Voxel-Based Lidar Scan Matching | NAVIGATION, accessed August 4, 2025, https://navi.ion.org/content/71/1/navi.627
38. Characterizing Perspective Error in Voxel-Based Lidar Scan Matching - NAVIGATION, accessed August 4, 2025, https://navi.ion.org/content/navi/71/1/navi.627.full.pdf

