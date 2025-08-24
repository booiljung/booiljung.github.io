[Blox](./index.md)
# Dynablox (복잡한 동적 환경에서의 실시간 객체 탐지를 위한 기하학적 접근법)


자율 로봇 기술이 공장이나 물류창고와 같은 고도로 통제된 정적 환경을 벗어나, 인간과 예측 불가능한 요소들이 공존하는 일상 공간으로 확장됨에 따라 새로운 차원의 기술적 난제가 부상하고 있다. 가정, 쇼핑몰, 도심 거리와 같이 끊임없이 변화하는 동적 환경에서 로봇이 안전하고 효율적으로 임무를 수행하기 위해서는, 주변의 움직이는 객체(사람, 동물, 차량 등)를 실시간으로 정확하게 인지하고 그에 맞춰 행동을 계획하는 능력이 필수적이다. 이러한 동적 객체들은 단순히 회피해야 할 장애물을 넘어, 로봇 자신의 위치를 추정(Localization)하고 주변 환경의 지도를 작성(Mapping)하는 SLAM(Simultaneous Localization and Mapping) 프로세스의 근간을 뒤흔드는 주요 오차 요인으로 작용한다.1 동적 요소의 존재는 SLAM의 정적 세계(static world) 가정을 위배하여, 위치 추정 오차를 최대 40%까지 증가시킬 수 있다는 연구 결과는 이 문제의 심각성을 방증한다.1

이러한 도전에 대응하기 위해 지난 수십 년간 다양한 동적 객체 탐지 기술이 연구되어 왔으며, 크게 두 가지 패러다임으로 분류할 수 있다. 첫 번째는 **외형 기반(Appearance-based) 접근법**이다. 이 방식은 주로 딥러닝 기반의 이미지 또는 포인트 클라우드 분류기를 사용하여 '사람', '자동차' 등 사전에 정의되고 학습된 특정 클래스의 객체를 식별한다.4 이 접근법은 학습된 객체에 대해서는 높은 정확도를 보이지만, 몇 가지 근본적인 한계를 내포한다. 학습 데이터에 존재하지 않는 비정형 객체, 예를 들어 굴러가는 공, 서핑보드를 들고 가는 사람, 흔들리는 문과 같은 객체는 탐지하지 못하는 일반화 성능의 부재가 가장 큰 문제다.5 또한, 많은 외형 기반 방법론들은 지면을 분리하여 제거하는 전처리 과정에 의존하는데, 이는 지면이 평평하다는 암묵적인 가정을 전제한다. 따라서 계단, 경사로, 불규칙한 지형 등 복잡한 3D 구조를 가진 현실 세계 환경에서는 그 강인함이 급격히 저하된다.5

두 번째 패러다임은 **스캔-투-스캔(Scan-to-Scan) 또는 스캔-투-맵(Scan-to-Map) 비교 접근법**이다. 이 방식은 연속된 센서 데이터(스캔) 간의 차이점이나, 현재의 스캔과 사전에 구축된 정적 지도(맵) 사이의 불일치 지점을 찾아내어 움직임을 탐지한다. 객체의 외형이나 클래스에 구애받지 않는다는 명백한 장점이 있지만, 이 역시 뚜렷한 한계를 가진다. 스캔 간 비교는 계산 비용이 매우 높아 실시간 처리에 부담을 주며, 특히 느리게 움직이는 객체나 보행자처럼 개별 신체 부위가 독립적으로 움직이는 관절형 객체(articulated objects)를 안정적으로 탐지하는 데 어려움을 겪는다.5 한편, 지도 기반 비교 방식은 신뢰할 수 있는 고품질의 정적 지도가 사전에 존재해야 한다는 강력한 제약을 가진다. 이는 로봇이 완전히 새로운 미지의 환경을 탐사하며 실시간으로 지도를 생성하고 동시에 동적 객체를 탐지해야 하는 온라인 시나리오에는 원천적으로 부적합하다.3

이러한 기존 패러다임의 한계에 직면하여, 스위스 취리히 연방 공과대학교(ETH Zurich)의 연구팀이 발표한 **Dynablox**는 동적 객체 탐지 문제에 대한 근본적인 질문을 새롭게 던진다.7 "객체의 정체성(identity)이나 외형적 특징(appearance)을 미리 알지 못하더라도, 순수한 시공간적 기하학(spatio-temporal geometry) 정보만을 이용해 '움직임'이라는 현상 그 자체를 강인하게 탐지할 수 있는가?" 이 질문은 동적 객체 탐지를 '무엇'을 찾는 분류(classification)의 문제에서, '어떻게' 움직임이 발생하는지를 규명하는 기하학적 일관성 검증(geometric consistency validation)의 문제로 패러다임을 전환시킨다.5 기존 연구들이 주로 딥러닝 모델의 성능을 향상시켜 더 많은 '종류'의 객체를 더 정확하게 '분류'하는 데 집중했다면, Dynablox는 이러한 접근이 필연적으로 마주하는 개방형 세계(open-world)의 무한한 가능성 앞에서 한계를 가질 수밖에 없다고 본다. 대신, "움직임이란 이전에 비어있다고 '확신'했던 공간을 점유하는 물리적 현상"이라는 가장 기본적인 정의로 회귀한다. 이러한 관점의 전환은 특정 객체 클래스에 대한 의존성을 원천적으로 제거함으로써, 이론적으로는 움직이는 모든 것을 탐지할 수 있는 강력한 일반성(generality)을 확보하는 철학적 기반이 된다.


Dynablox의 작동 원리는 복잡한 딥러닝 모델이나 정교한 추적 알고리즘이 아닌, 직관적이고 강력한 하나의 물리적 원칙에 기반한다. 바로 **'자유 공간 침범(Free-Space Intrusion)' 원칙**이다. 이 원칙은 "과거의 시점들에서 수집된 센서 데이터를 통해 높은 신뢰도로 비어있다고(free-space) 확인된 3차원 공간에, 현재 시점의 센서 측정값이 존재한다면, 그 측정값은 정적인 세계의 일부가 아니라 그 공간으로 '이동'해 온 동적 객체에 해당한다"는 명제로 요약할 수 있다.5 이 아이디어 자체는 새로운 것이 아니지만, Dynablox의 독창성은 이 원칙이 현실 세계에서 강인하게 작동하기 위한 핵심 전제 조건을 만족시키는 방법에 있다.

그 핵심 전제 조건은 바로 **'높은 신뢰도(High-Confidence)'**의 확보다. 만약 자유 공간에 대한 추정이 조금이라도 부정확하여, 실제로는 벽이나 기둥이 있는 공간을 '비어있다'고 잘못 판단(false free-space)한다면, 그 이후 해당 정적 구조물을 관측할 때마다 계속해서 동적 객체로 오인하는 심각한 오탐(false positive)이 누적될 것이다.5 이러한 오탐은 한번 발생하면 시스템이 스스로 교정하기 매우 어렵기 때문에, 자유 공간으로 판정하는 기준은 극도로 보수적이고 신중해야 한다.

Dynablox는 이 '높은 신뢰도'를 확보하기 위해, 로봇이 온라인으로 지도를 작성하는 과정에서 필연적으로 발생하는 모든 종류의 **불확실성을 명시적으로 모델링하고 정량적으로 다루는** 전략을 취한다. 이는 다음과 같은 세 가지 주요 불확실성 소스를 포함한다.

1. **센서 불확실성 (Sensing Uncertainty):** 3D LiDAR 센서는 완벽하지 않다. 측정된 포인트에는 노이즈가 포함되어 있으며, 센서로부터 거리가 멀어질수록 포인트 클라우드는 점점 희소(sparse)해져 관측의 신뢰도가 떨어진다. Dynablox는 이러한 노이즈와 데이터 희소성을 고려하여 섣불리 공간을 비어있다고 판단하지 않는다.5
2. **상태 추정 불확실성 (State Estimation Uncertainty):** 로봇은 자신의 위치와 자세를 추정하기 위해 주행 거리계(Odometry)를 사용하지만, 이는 시간이 지남에 따라 오차가 누적되는 드리프트(drift) 현상을 피할 수 없다. 이 드리프트로 인해 로봇의 좌표계가 실제 세계와 미세하게 틀어지면, 정적인 배경조차 움직이는 것처럼 관측될 수 있다. Dynablox는 이러한 드리프트의 영향을 모델링하여 오탐을 억제한다.5
3. **매핑 불확실성 (Mapping Uncertainty):** 로봇이 처음 탐사하는 환경에서는 지도가 점진적으로 구축된다. 아직 완전히 탐사되지 않은 미지(unknown)의 공간과 이미 관측된 공간의 경계에서는 지도의 정확도가 낮을 수밖에 없다. Dynablox는 이러한 경계 영역에서의 불확실성을 인지하고, 충분한 관측이 누적되기 전까지는 해당 영역을 신뢰할 수 있는 자유 공간으로 간주하지 않는다.5

이처럼 다양한 불확실성을 체계적으로 고려하여 극도로 보수적인 자유 공간 추정치를 얻어낸 결과, Dynablox는 객체의 색상, 형태, 질감과 같은 외형 정보나 '사람', '자동차'와 같은 의미론적(semantic) 정보에 전혀 의존하지 않는 **외형에 무관한(appearance-agnostic) 강인함**을 획득한다. 오직 시공간적 점유 정보, 즉 '이전에 비어있었는가, 지금은 채워져 있는가'라는 기하학적 사실만을 판단 기준으로 삼는다. 이로 인해 굴러가는 공, 흔들리는 문, 상자를 든 사람, 심지어 서핑보드로 몸을 가린 사람 등 그 어떤 형태의 움직임도 동일한 원리로 탐지할 수 있는 강력한 일반성을 갖추게 된다.5

이러한 접근 방식은 동적 객체 탐지를 바라보는 관점을 근본적으로 바꾼다. 전통적인 로봇 시스템 아키텍처에서는 SLAM 모듈, 동적 객체 탐지 모듈, 경로 계획 모듈 등이 개별적으로 설계되고 정보를 주고받는 형태로 구성되는 경우가 많았다.2 하지만 Dynablox는 동적 객체 탐지를 독립적인 퍼셉션 모듈이 아닌, 

**'강인한 동적 지도 작성(Robust Dynamic Mapping)' 문제의 자연스러운 결과물**로 간주한다. 즉, Voxblox라는 기존의 효율적인 매핑 프레임워크를 확장하여, 동적 객체 탐지 기능을 매핑 프로세스 내부에 깊숙이 통합시켰다.5 탐지를 위해 별도의 복잡한 계산 파이프라인을 추가하는 대신, 지도를 업데이트하는 과정에서 발생하는 '기하학적 불일치'를 자연스럽게 동적 정보로 해석하는 것이다. 이로 인해 기존 매핑 스택에 최소한의 계산 오버헤드만을 추가하면서도 강력한 탐지 성능을 얻을 수 있다.5 이는 '인식'과 '지도 작성'이 분리될 수 없는 동전의 양면과 같다는 철학을 시스템 아키텍처로 구현한 것이다. 강인한 지도가 있어야 강인한 인식이 가능하고, 강인한 인식은 다시 더 나은 지도를 만드는 선순환 구조를 제안하며, 이는 미래의 고성능 로보틱스 시스템이 더욱 통합적이고 긴밀하게 결합된 형태로 발전할 것임을 시사한다.


Dynablox의 핵심 철학을 구현하는 기술적 방법론은 시공간적 불확실성을 정량적으로 모델링하고 제어하는 과정에 집약되어 있다. 이 방법론은 효율적인 3D 공간 표현을 위한 기반 구조 위에, 신뢰도 높은 자유 공간을 추정하기 위한 다단계 필터링 메커니즘을 얹는 형태로 구성된다.


Dynablox는 실시간 성능과 메모리 효율성을 확보하기 위해, 이미 그 성능이 입증된 3D 매핑 프레임워크인 **Voxblox**를 기반으로 채택하고 이를 확장하는 방식을 택했다.1 Voxblox는 무한한 3차원 공간을 효율적으로 다루기 위해 **계층적 복셀 해싱(hierarchical voxel hashing)** 구조를 사용한다. 전체 공간을 일정한 크기(예: 3.2m x 3.2m x 3.2m)의 '블록(Block)' 단위로 가상으로 나누고, 센서 데이터가 실제로 관측된 블록만을 동적으로 메모리에 할당한다.

각 블록의 내부는 다시 더 작은 단위인 '복셀(Voxel)'(예: 20cm x 20cm x 20cm)의 밀집된 그리드로 구성된다. 각 복셀은 주변 환경의 기하학적 정보를 표현하기 위해 **TSDF(Truncated Signed Distance Field)** 값을 저장한다. TSDF는 복셀의 중심점에서 가장 가까운 표면까지의 거리를 나타내는데, 표면 바깥(자유 공간)은 양수, 표면 안쪽은 음수로 표현되며, 일정 거리(truncation distance) 이상 떨어진 값은 최대/최소값으로 제한된다. 이 방식은 단순한 점유/비점유 그리드맵에 비해 훨씬 정밀하고 부드러운 표면 재구성이 가능하며, 메모리 효율성과 계산 속도 면에서 큰 이점을 제공한다.5

Dynablox는 이 견고한 Voxblox의 복셀 구조에 시간적 동역학을 추적하고 신뢰도를 판단하기 위한 새로운 속성들을 추가하여, 각 복셀의 상태를 단순한 TSDF 값에서 다음과 같은 **5-튜플(5-tuple)로 확장**하여 정의한다.
$$
v(t) = \{d(t), w(t), t_o(t), t_d(t), f(t)\}
$$
여기서 각 요소의 의미는 다음과 같다 5:

- $d(t)$: 시간 $t$에서의 TSDF 거리 값
- $w(t)$: 해당 TSDF 값의 신뢰도를 나타내는 가중치 (관측 횟수가 많을수록 증가)
- $t_o(t)$: 해당 복셀이 마지막으로 '점유(occupied)'된 것으로 관측된 시각 (프레임 번호)
- $t_d(t)$: 해당 복셀이 연속적으로 점유 상태를 유지한 기간 (프레임 수)
- $f(t)$: Dynablox의 핵심 속성으로, 해당 복셀이 **'높은 신뢰도의 자유 공간(high-confidence free-space)'**인지 여부를 나타내는 이진 플래그 ($1$이면 참, $0$이면 거짓)


이 부분이 Dynablox 알고리즘의 심장부로, 앞서 언급된 다양한 불확실성을 극복하고 신뢰할 수 있는 자유 공간 플래그 $f$를 결정하는 과정을 수학적 모델을 통해 체계적으로 구현한다. 이 과정은 여러 단계의 보수적인 검증을 거친다.5


가장 먼저, 각 복셀이 현재 시점 $t$에 '점유'되었는지를 판단해야 한다. 이를 위해 현재 프레임의 최신 정보와 과거에 누적된 지도 정보를 모두 활용한다. 어떤 복셀 $v$가 현재 프레임의 포인트 클라우드 $P(t)$에 속한 포인트 $p_i$를 포함하거나($\exists p_i \in P(t): K(p_i) = v$), 또는 누적된 TSDF 거리 값 $d(t)$가 표면에 매우 가깝다는 것을 나타낼 때($d(t) < \tau_d$, 여기서 $\tau_d$는 보수적인 임계값), 해당 복셀의 마지막 점유 시각 $t_o$를 현재 시각 $t$로 갱신한다.
$$
t_{o}^{(t+1)} = \begin{cases} t & \text{if } d(t) < \tau_d \lor \exists p_i \in P(t) : K(p_i) = v \\ t_{o}^{(t)} & \text{otherwise} \end{cases}
$$
5


원거리나 특정 각도에서는 LiDAR 빔이 성기게(sparse) 되어, 이전 프레임에서 관측되었던 객체가 이번 프레임에서는 관측되지 않을 수 있다. 이러한 일시적인 미관측으로 인해 점유 상태가 불필요하게 끊어지는 것을 방지하기 위해, 마지막 점유 시각 $t_o$가 현재로부터 일정 시간($\tau_s$, 예: 2 프레임) 이내라면 점유가 지속된 것으로 간주하여 점유 지속 시간 $t_d$를 1 증가시킨다.
$$
t_{d}^{(t+1)} = \begin{cases} t_{d}^{(t)} + 1 & \text{if } t_{o}^{(t)} \ge t - \tau_s \\ 0 & \text{otherwise} \end{cases}
$$
5


이제 가장 중요한 단계인 자유 공간 플래그 $f$를 결정한다. 오탐을 막기 위해 극도로 보수적인 기준을 적용한다.

- **시간적 조건 (Temporal Condition):** 센서 노이즈로 인해 특정 공간이 아주 잠깐 비어 보이는 현상을 걸러내기 위해, 해당 공간이 유의미한 시간 창($\tau_w$) 이상 지속적으로 비어있었어야만 자유 공간 후보가 될 수 있다. 즉, 마지막 점유 시각이 현재보다 $\tau_w$만큼 과거여야 한다 ($t_o(v_0) < t - \tau_w$).
- **공간적 조건 (Spatial Condition):** 단 하나의 복셀만 비어있다고 해서 그곳을 자유 공간으로 신뢰할 수는 없다. 특히 미탐사 지역의 경계면에서는 지도 정보가 부정확할 수 있다. 따라서 특정 복셀 $v$와 그 주변의 모든 이웃 복셀($\forall v_0 \in \mathcal{N}(v)$)이 동시에 위의 시간적 조건을 만족하고, 또한 모든 이웃 복셀이 한 번 이상 관측되어 유효한 가중치($w(t)(v_0) > 0$)를 가져야만 한다.

이 두 조건을 모두 만족할 때 임시 플래그 $\hat{f}$가 1이 되며, 한번 자유 공간으로 판정된($f=1$) 복셀은 이 상태를 유지한다.
$$
\hat{f} = \mathbb{I}(t_o(v_0) < t - \tau_w \land w(t)(v_0) > 0, \forall v_0 \in \mathcal{N}(v))
$$

$$
f^{(t+1)} = \max(f^{(t)}, \hat{f})
$$

5


로봇의 주행 거리계 오차(drift)가 누적되면, 정적인 배경이 미세하게 움직이는 것처럼 관측되어 이전에 자유 공간으로 설정된 영역을 침범할 수 있다. 이는 대규모의 지속적인 오탐을 유발하는 주된 원인이다. Dynablox는 이 현상이 일반적으로 '느리게' 점진적으로 발생한다는 특성에 착안하여, 특정 복셀의 점유 지속 시간 $t_d$가 예상되는 드리프트 속도를 기반으로 계산된 임계 시간 $\tau_r$을 초과하면, 이는 실제 동적 객체가 아니라 드리프트로 인한 현상일 가능성이 높다고 판단한다. 이 경우, 해당 복셀의 자유 공간 플래그를 강제로 리셋($f=0$)하여 잘못된 신뢰 정보를 초기화한다.
$$
f^{(t+1)} = \begin{cases} 0 & \text{if } t_d^{(t)} > \tau_r \\ \max(f^{(t)}, \hat{f}) & \text{otherwise} \end{cases}
$$
5

이러한 수학적 모델들은 단순한 규칙의 나열이 아니라, 로보틱스 시스템이 현실 세계에서 겪는 주요 불확실성 요인들-센서 노이즈, 데이터 희소성, 매핑 경계의 불확실성, SLAM의 드리프트-을 체계적으로 정량화하고 이에 직접 대응하도록 설계된 '물리 기반 신뢰도 모델'이라고 할 수 있다. 각 수식은 실제 로봇 운영 시나리오에서 발생하는 구체적인 문제점 하나하나에 직접적으로 대응하며, 이는 Dynablox가 수많은 실험과 경험을 통해 다듬어진 매우 실용적인 알고리즘임을 방증한다.


위의 엄격한 과정을 통해 높은 신뢰도의 자유 공간($f=1$)이 결정되면, 동적 객체 탐지는 매우 간단해진다.

1. 현재 프레임의 포인트 클라우드 $P(t)$에서, 높은 신뢰도의 자유 공간으로 레이블링된 복셀 또는 그 이웃 복셀에 떨어진 모든 포인트들을 동적 객체의 '시드(seed)' 포인트로 간주한다.
2. 이 시드 포인트를 포함하는 복셀들($V_{dyn}$)을 대상으로 **연결 요소 분석(Connected Component Analysis)**을 수행하여 공간적으로 인접한 복셀들을 하나의 객체 클러스터($C$)로 묶는다.
3. 마지막으로, 단일 노이즈 포인트를 걸러내기 위해 클러스터의 크기가 특정 임계값($\tau_c$)보다 작은 경우는 무시하고, 남은 유효한 클러스터에 속하는 모든 포인트들을 최종적인 동적 객체 포인트 집합($P_{dyn}$)으로 출력한다.5


Dynablox의 우수성은 체계적인 정량적 평가와 다양한 시나리오에서의 정성적 검증을 통해 입증되었다. 평가는 알고리즘의 정확도, 다양한 환경 및 객체에 대한 일반성, 그리고 실제 로봇에 탑재되기 위한 실시간 성능에 초점을 맞추어 진행되었다.


정량적 평가는 주로 공개 벤치마크 데이터셋인 **DOALS(Dynamic Object Annotated Lidar Scans)**를 사용하여 수행되었다.5 이 데이터셋은 고해상도 LiDAR 센서로 수집되었으며, 주로 개방된 평평한 환경에서 움직이는 보행자들에 대한 정확한 레이블(ground truth)을 포함하고 있다. 성능 비교를 위한 핵심 지표로는 탐지된 동적 포인트 집합과 실제 동적 포인트 집합 간의 중첩도를 나타내는 **IoU(Intersection over Union)**가 사용되었다.

Dynablox는 여러 최신 기술(SOTA, State-of-the-art)들과 직접 비교되었다. 비교 대상에는 미래의 정보까지 모두 사용하여 성능의 이론적 상한선을 보여주는 오프라인 점유 기반 방식(Occupancy-based approach), 그리고 대표적인 경쟁 패러다임인 딥러닝 기반의 외형 기반 분류기(Appearance-based approach)가 포함되었다.

실험 결과, 일반적인 로봇 운영 환경을 가정한 탐지 거리 20m 제한 조건에서 Dynablox는 **86.0%의 IoU**를 달성했다. 이는 외형 정보를 전혀 사용하지 않고 순수 기하학적 단서만으로도 최신 외형 기반 분류기(82.0% @ 전체 범위)를 능가하고, 오프라인 방식(88.0% @ 전체 범위)의 성능에 근접하는 매우 인상적인 결과다.1 이는 Dynablox의 핵심 철학, 즉 외형에 대한 가정 없이도 강인한 탐지가 가능하다는 주장을 강력하게 뒷받침한다.


이 표는 Dynablox의 핵심 성능 주장을 객관적인 수치로 뒷받침하는 가장 중요한 증거다. 오프라인 방식(성능의 상한선), 외형 기반 방식(주요 경쟁 패러다임), 그리고 단순 자유 공간 쿼리(자체 방법론의 필요성 입증)와의 직접 비교를 통해 Dynablox의 기술적 위치와 우수성을 명확하게 보여준다.

| 접근 방식 (Approach)            | 탐지 거리 (Range) | IoU (%)  |
| ------------------------------- | ----------------- | -------- |
| Occupancy (Offline Upper Bound) | ∞                 | 88.0     |
| Appearance-based (3D-MiniNet)   | ∞                 | 82.0     |
| **Ours (Dynablox)**             | **∞**             | **83.8** |
| LC Free Space (Baseline)        | 20m               | 30.7     |
| **Ours (Dynablox)**             | **20m**           | **86.0** |

Source: 5 (Table I)


Dynablox의 성공이 단 하나의 기술이 아닌, 여러 불확실성 요소를 체계적으로 모델링한 결과임을 보이기 위해 구성 요소별 제거 연구(Ablation Study)가 수행되었다. 이 연구는 Dynablox를 구성하는 핵심 요소들을 하나씩 제거했을 때 성능(IoU)이 얼마나 저하되는지를 측정한다.

분석 결과, **TSDF 큐(TSDF Cue)**, 즉 여러 프레임의 측정값을 융합하여 노이즈를 줄이는 기능과, **시간적 신뢰도 창($\tau_w$)**, 즉 공간을 자유롭다고 선언하기 전에 일정 시간 기다리는 보수적인 정책이 성능에 가장 결정적인 영향을 미치는 것으로 나타났다. 이들을 제거했을 때 IoU는 각각 62.4%, 74.4%나 급감했다. 이는 불확실성을 신중하고 보수적으로 다루는 것이 Dynablox의 강인함에 얼마나 중요한지를 정량적으로 증명하는 결과다.5


이 표는 각 구성 요소의 기여도를 수치화함으로써 저자들의 설계 결정이 타당했음을 입증하고, 어떤 부분이 알고리즘의 핵심인지를 명확히 한다.

| 제거된 구성 요소 (Ablated Component) | IoU (%)  | 성능 저하 (%) |
| ------------------------------------ | -------- | ------------- |
| **Full Model (Ours)**                | **86.0** | **-**         |
| w/o Temporal Window ($\tau_w$)       | 11.6     | -74.4         |
| w/o TSDF Cue                         | 23.6     | -62.4         |
| w/o Spatial Margin ($\mathcal{N}$)   | 38.1     | -47.9         |
| w/o Sparsity Compensation ($\tau_s$) | 83.3     | -2.7          |

Source: 5 (Table II)


많은 딥러닝 기반 모델들은 특정 벤치마크 데이터셋에서는 높은 수치를 기록하지만, 데이터셋의 분포를 벗어나는 시나리오에서는 성능이 급격히 저하되는 경향이 있다. Dynablox는 이러한 한계를 극복하고 알고리즘의 근본적인 '일반성(generality)'을 입증하기 위해, DOALS 데이터셋의 한계를 보완하는 자체 데이터셋을 구축하고 공개했다.9 이 데이터셋은 

**계단, 경사로 등 복잡한 3D 구조의 실내 환경**과 **굴러가는 공, 흔들리는 문, 상자나 서핑보드를 운반하는 사람** 등 사전 학습이 불가능한 다양한 비정형 동적 객체들을 포함한다.

이 데이터셋에서의 정성적 실험 결과, Dynablox는 지면이 평평하지 않은 복잡한 3D 지형에서도 안정적으로 작동했으며, 외형이나 클래스에 관계없이 움직이는 모든 객체를 매우 강인하게 탐지해냈다.5 이는 Dynablox의 높은 성능이 특정 데이터셋에 과적합(overfitting)된 것이 아니라, 알고리즘의 근본 철학에서 비롯된 것임을 시각적으로 증명한다. 이러한 접근은 실제 로봇이 마주할 예측 불가능한 현실 세계에서 신뢰성을 확보하기 위해서는 벤치마크 점수뿐만 아니라, 다양한 엣지 케이스(edge case)에 대한 강인성이 더욱 중요하다는 강력한 메시지를 전달한다.


아무리 정확한 알고리즘이라도 실시간으로 작동하지 않으면 실제 로봇에 탑재될 수 없다. Dynablox는 효율성 측면에서도 뛰어난 성능을 보였다. 노트북급 CPU(AMD-4800U) 환경에서 평균 **17.2 FPS(초당 프레임 수)**로 구동되어 온라인 및 실시간 요구사항을 충분히 만족시켰다.5

더 중요한 것은, Dynablox가 기존의 3D 볼륨 매핑 스택(TSDF 통합)에 추가하는 계산 오버헤드가 약 **39%** 수준에 불과하다는 점이다.5 이는 동적 객체 탐지를 위해 완전히 별개의 무거운 알고리즘을 실행하는 대신, 기존 매핑 프로세스에 자연스럽게 통합되는 경량화된 설계를 통해 높은 효율성을 달성했음을 의미한다.


Dynablox는 발표 이후 학계와 산업계 모두에 상당한 파급 효과를 미쳤다. 이는 단순히 높은 성능 때문만이 아니라, 연구 결과를 공유하는 방식과 알고리즘 자체의 실용성 덕분이다.


Dynablox 연구팀은 논문 발표와 동시에 알고리즘의 **ROS(Robot Operating System) 기반 소스 코드**와, 실험에 사용된 **새로운 데이터셋**을 모두 오픈소스로 공개했다.9 이러한 개방적인 접근은 동적 SLAM 및 객체 탐지 분야의 후속 연구자들이 손쉽게 결과를 재현하고, 자신들의 연구와 성능을 비교하며, 나아가 Dynablox를 기반으로 새로운 아이디어를 발전시킬 수 있는 견고한 발판을 마련해주었다. 그 결과, 다수의 후속 연구에서 Dynablox는 기하학 기반 동적 객체 탐지 방법론의 성능을 비교하기 위한 표준 **베이스라인(baseline)**으로 활발하게 인용되고 있다.1 이는 Dynablox가 해당 분야에서 하나의 새로운 기준점을 제시했음을 의미한다.


Dynablox가 이룬 가장 중요하고 상징적인 성과는, 학술적 논의를 넘어 실제 산업 현장에서 그 가치를 인정받았다는 점이다. Dynablox의 핵심 알고리즘은 세계적인 GPU 기업이자 AI 및 로보틱스 플랫폼의 선두주자인 **NVIDIA의 공식 로보틱스 3D 재구성 라이브러리인 nvblox**에 **'Dynamic Reconstruction'**이라는 기능으로 공식 통합되었다.9

NVIDIA의 nvblox 공식 기술 문서는 'Dynamic Reconstruction' 섹션에서 Dynablox 논문을 직접적으로 인용하며, 그 알고리즘의 작동 원리를 다음과 같이 명확히 설명하고 있다: "이 알고리즘은 **높은 신뢰도의 자유 공간(high-confidence freespace) 레이어**를 유지 관리한다. 어떤 객체가 이 자유 공간으로 진입할 때마다, 그 객체는 동적으로 식별된 후 별도의 **동적 점유 레이어(dynamic occupancy layer)**로 통합된다".12 이는 Dynablox의 핵심 철학 및 방법론과 정확히 일치하는 설명이다.

이 통합 과정에서 중요한 기술적 진보가 이루어졌다. 원본 Dynablox는 CPU 기반으로 구현되었지만, nvblox는 이를 NVIDIA의 병렬 컴퓨팅 플랫폼인 **CUDA를 통해 GPU에 최적화**했다. 이를 통해 알고리즘의 내재된 병렬성을 극대화하여, 훨씬 더 높은 해상도의 복셀 맵을 사용하면서도 더욱 빠른 속도로 동적 객체를 탐지할 수 있게 되었다.9

NVIDIA와 같은 거대 기술 기업이 자신들의 공식 로보틱스 라이브러리에 특정 학술 연구의 알고리즘을 채택했다는 사실은 매우 큰 의미를 가진다. 이는 Dynablox의 접근 방식이 학술적 논의 수준을 넘어, 산업 현장에서 요구하는 **성능, 효율성, 그리고 안정성**이라는 세 가지 핵심 요건을 모두 만족시키는 매우 실용적인 솔루션임을 시장의 리더가 공인한 것과 같다. 이는 수십 편의 학술적 인용보다도 더 강력한 기술적 검증(validation)이라고 평가할 수 있다.

이 성공적인 산업계 채택은 '학계의 선도적 아이디어'가 '산업계의 대규모 플랫폼'을 만나 기술 확산의 임계점을 넘은 대표적인 사례로 분석될 수 있다. 이 과정은 다음과 같은 단계로 이루어졌다. 첫째, ETH Zurich의 연구팀은 뛰어난 아이디어를 고품질의 ROS 기반 오픈소스 코드로 구현하여 공개했다.9 둘째, NVIDIA의 Isaac ROS 팀은 자신들의 로보틱스 개발자 생태계를 위해 실시간 3D 재구성 및 동적 환경 인지 기능이라는 명확한 수요를 가지고 있었다.14 셋째, 이들은 이미 공개되어 있고 성능이 검증되었으며, GPU 병렬화에 적합한 구조를 가진 Dynablox 알고리즘을 발견하고 이를 자신들의 nvblox 라이브러리에 통합하기로 결정했다.12 만약 Dynablox 코드가 비공개였다면 이러한 신속하고 성공적인 기술 이전은 불가능했을 것이다. 이 통합을 통해 Dynablox의 아이디어는 개별 연구실 수준을 넘어, NVIDIA의 Jetson 임베디드 플랫폼과 Isaac Sim 시뮬레이터를 사용하는 전 세계 수많은 개발자들에게 손쉽게 접근 가능한 표준 기능이 되었다.14 결국, 뛰어난 아이디어, 고품질의 오픈소스 구현, 그리고 산업계의 명확한 수요라는 세 가지 요소가 시너지를 일으켜 학술적 성과가 산업 표준으로 비상하는 성공적인 경로를 만들어낸 것이다. 이는 현대 기술 연구 개발에서 오픈소스 전략이 갖는 중요성을 강력하게 시사한다.


Dynablox는 기하학 기반의 동적 객체 탐지 분야에서 중요한 이정표를 세웠지만, 완벽한 해결책은 아니며 몇 가지 내재적 한계와 함께 미래 연구를 위한 새로운 과제를 제시한다.


Dynablox의 한계는 대부분 그 핵심 철학인 '순수 기하학'에 의존하는 데서 비롯된다.

- **기하학적 표현의 한계:** 논문에서도 인정하듯이, Dynablox는 복셀 기반의 맵 표현을 사용하기 때문에 특정 종류의 객체를 다루는 데 어려움이 있다. 예를 들어, 로프나 케이블처럼 **매우 얇은 객체**나, 유리창이나 거울처럼 LiDAR 빔이 투과하거나 불규칙하게 반사되는 **투명/반사성 표면**으로 이루어진 객체는 복셀 맵 상에서 안정적으로 표현되기 어렵다. 이로 인해 탐지에 실패하거나 오히려 정적인 반사면을 동적 객체로 오인하는 오탐을 유발할 수 있다.5
- **단기적 관점의 한계:** Dynablox의 핵심 원리는 '이전 프레임 대비 현재 프레임'의 변화를 추적하는 것이다. 이는 실시간으로 발생하는 **단기적인(short-term) 움직임**을 탐지하는 데는 매우 효과적이다. 그러나 로봇이 특정 장소를 떠났다가 한참 뒤에 다시 돌아왔을 때, 그 사이에 가구가 재배치되거나 물건이 다른 위치로 옮겨진 것과 같은 **장기적인(long-term) 또는 시야 밖(out-of-sight)에서의 변화**를 인지하고 처리하는 데는 근본적인 한계를 가진다.17 Dynablox의 관점에서는, 옮겨진 후 가만히 놓여 있는 의자는 그저 새로운 정적 객체일 뿐, 그것이 '이동'했다는 맥락을 파악할 수 없다.


Dynablox의 한계를 극복하고 더 높은 수준의 환경 이해를 달성하기 위한 연구는 크게 두 가지 방향으로 진행되고 있으며, 이는 Dynablox의 기하학적 기반 위에 새로운 정보 계층을 추가하는 형태로 볼 수 있다.


최근 SLAM 연구의 가장 중요한 흐름 중 하나는 3차원 공간 지도에 '이것은 의자', '저것은 사람'과 같이 **의미(semantics)**를 부여하는 **Semantic SLAM**이다.2 Semantic SLAM은 동적 객체 탐지에 새로운 가능성을 제시한다. 예를 들어, '사람'이나 '자동차'와 같이 본질적으로 움직일 확률이 높은 객체 클래스는 더 낮은 기하학적 증거만으로도 동적으로 분류할 수 있고, 반대로 '건물'이나 '책상'처럼 정적일 확률이 매우 높은 객체는 웬만한 기하학적 불일치가 있어도 정적으로 유지하려는 경향을 부여할 수 있다.20

이는 Dynablox의 '외형에 무관한' 철학과 언뜻 상충되는 것처럼 보일 수 있지만, 실제로는 매우 강력한 상호 보완 관계를 형성할 수 있다. 즉, Dynablox의 강인한 기하학적 탐지를 1차적인 증거로 사용하되, 의미론적 정보를 일종의 '사전 확률(prior)'처럼 활용하여 최종 판단의 신뢰도를 더욱 높이는 하이브리드 접근이 가능하다. Dynablox가 "무언가 움직였다"는 사실(fact)을 높은 신뢰도로 알려준다면, Semantic SLAM은 "움직인 것은 사람이다"라는 해석(interpretation)을 제공하여 로봇이 더 지능적인 판단을 내리도록 도울 수 있다.


로봇이 며칠, 몇 주, 몇 달에 걸쳐 같은 환경에서 반복적으로 작동할 때 발생하는 변화(가구 재배치, 계절에 따른 식물 변화 등)를 인지하고 지도에 반영하는 연구 분야가 **Long-term SLAM**이다.22 이러한 연구들은 Dynablox가 다루지 못하는 '과거에는 움직였지만 현재는 정지해 있는 객체(moved but currently static objects)'나 '사라진 객체(removed objects)', '새로 생긴 객체(newly placed objects)'를 체계적으로 관리할 수 있다.

Long-term SLAM은 시간에 따라 변화하는 지도의 여러 버전을 관리하거나, 각 객체의 시공간적 이력을 추적하는 방식을 사용한다. 미래의 강인한 동적 SLAM 시스템은 Dynablox와 같은 실시간 단기 탐지 모듈이 현재의 동적인 위협을 처리하고, Long-term SLAM 모듈이 그 결과를 바탕으로 영구적인 지도 변화를 관리하고 업데이트하는 계층적 구조를 가질 것으로 전망된다.17

결론적으로, Dynablox는 동적 객체 탐지의 **필요조건**인 '순수한 움직임'을 감지하는 문제에 대해 매우 강력하고 실용적인 해법을 제시했다. 그러나 로봇이 인간과 같은 수준의 지능적인 상황 이해를 하기 위한 **충분조건**을 만족시키기 위해서는 의미론적, 시간적 맥락 정보와의 결합이 필수적이다. 이런 관점에서 Dynablox의 성공은 동적 환경 인지 연구의 끝이 아니라, 더 높은 수준의 지능을 구축하기 위한 견고한 첫 번째 발판을 완성했다는 데에 가장 큰 의의가 있다. 미래의 완전한 동적 환경 이해 시스템은 Dynablox의 기하학적 엔진을 기반으로, 그 위에 Semantic Layer와 Long-term Memory Layer를 쌓아 올리는 계층적 아키텍처가 될 가능성이 높다.


1. Dynamic Object Detection in Range data using Spatiotemporal Normals - OPUS at UTS, accessed August 13, 2025, [https://opus.lib.uts.edu.au/bitstream/10453/173898/2/Dynamic%20Object%20Detection%20in%20Range%20data%20using%20Spatiotemporal%20Normals.pdf](https://opus.lib.uts.edu.au/bitstream/10453/173898/2/Dynamic Object Detection in Range data using Spatiotemporal Normals.pdf)
2. DS-SLAM: A Semantic Visual SLAM towards Dynamic Environments - arXiv, accessed August 13, 2025, https://arxiv.org/pdf/1809.08379
3. LiDAR-based Real-Time Object Detection and Tracking in Dynamic Environments - arXiv, accessed August 13, 2025, https://arxiv.org/html/2407.04115v1
4. Dynamic and Real-Time Object Detection Based on Deep Learning for Home Service Robots - MDPI, accessed August 13, 2025, https://www.mdpi.com/1424-8220/23/23/9482
5. Dynablox: Real-time Detection of Diverse Dynamic Objects in Complex Environments, accessed August 13, 2025, https://www.researchgate.net/publication/370153018_Dynablox_Real-time_Detection_of_Diverse_Dynamic_Objects_in_Complex_Environments
6. Dynablox: Real-Time Detection of Diverse Dynamic Objects in Complex Environments, accessed August 13, 2025, https://www.researchgate.net/publication/373150784_Dynablox_Real-time_Detection_of_Diverse_Dynamic_Objects_in_Complex_Environments
7. Dynablox: Real-time Detection of Diverse Dynamic Objects in Complex Environments - arXiv, accessed August 13, 2025, https://arxiv.org/abs/2304.10049
8. zhuhu00/Awesome_Dynamic_SLAM: Dynamic SLAM, Life-long SLAM Research(Lidar, Visual, Sensor Fusion etc.) - GitHub, accessed August 13, 2025, https://github.com/zhuhu00/Awesome_Dynamic_SLAM
9. ethz-asl/dynablox: Real-time detection of diverse dynamic objects in complex environments. - GitHub, accessed August 13, 2025, https://github.com/ethz-asl/dynablox
10. Dynablox: Real-time Detection of Diverse Dynamic Objects in Complex Environments, accessed August 13, 2025, https://projects.asl.ethz.ch/datasets/doku.php?id=dynablox
11. Robust Real-time Moving Object Segmentation in LiDAR Point Clouds - DiVA portal, accessed August 13, 2025, http://www.diva-portal.org/smash/get/diva2:1892088/FULLTEXT01.pdf
12. Technical Details - isaac_ros_docs documentation - NVIDIA Isaac ROS, accessed August 13, 2025, https://nvidia-isaac-ros.github.io/concepts/scene_reconstruction/nvblox/technical_details.html
13. nvidia-isaac/nvblox: A GPU-accelerated TSDF and ESDF library for robots equipped with RGB-D cameras. - GitHub, accessed August 13, 2025, https://github.com/nvidia-isaac/nvblox
14. Isaac ROS Nvblox - isaac_ros_docs documentation, accessed August 13, 2025, https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_nvblox/index.html
15. 5.9.4.2. NVIDIA ROS 2 Projects - Vulcanexus 1.0.0 documentation, accessed August 13, 2025, https://docs.vulcanexus.org/en/iron/ros2_documentation/source/Related-Projects/Nvidia-ROS2-Projects.html
16. 3D Scene Reconstruction | NVIDIA Isaac ROS Nvblox | Jetson Orin Nano - YouTube, accessed August 13, 2025, https://www.youtube.com/watch?v=EDQiwWbjnws
17. Constraint-Based Modeling of Dynamic Entities in 3D Scene Graphs for Robust SLAM, accessed August 13, 2025, https://arxiv.org/html/2503.02050v1
18. Khronos: A Unified Approach for Spatio-Temporal Metric-Semantic SLAM in Dynamic Environments - Robotics, accessed August 13, 2025, https://www.roboticsproceedings.org/rss20/p081.pdf
19. Learning from Feedback: Semantic Enhancement for Object SLAM Using Foundation Models - arXiv, accessed August 13, 2025, https://arxiv.org/html/2411.06752v1
20. A Semantic SLAM Approach for Dynamic Scenes Based on LiDAR Point Clouds - arXiv, accessed August 13, 2025, https://arxiv.org/abs/2402.18318
21. A 3D Object Detection based Semantic SLAM towards Dynamic Indoor Environments - arXiv, accessed August 13, 2025, https://arxiv.org/html/2310.06385
22. Long-Term Simultaneous Localization and Mapping in Dynamic Environments - Perceptual Robotics Laboratory (PeRL), accessed August 13, 2025, http://robots.engin.umich.edu/publications/ncarlevaris-phdthesis.pdf
23. Long-term Object-based SLAM in Low-dynamic Environments - DSpace@MIT, accessed August 13, 2025, https://dspace.mit.edu/handle/1721.1/153705

