# FAST-LIVO2



자율 로봇 및 차량의 핵심 기술인 동시적 위치 추정 및 지도 작성(Simultaneous Localization and Mapping, SLAM)은 단일 센서의 본질적인 한계로 인해 다중 센서 융합이라는 패러다임으로 발전해왔다. 각 센서는 고유한 장점을 가지지만, 특정 환경에서는 치명적인 단점을 드러내기 때문이다.

시각 SLAM(Visual SLAM)은 저렴한 카메라를 사용하여 주변 환경의 풍부한 텍스처와 색상 정보를 획득할 수 있다. 그러나 조명 변화에 매우 민감하며, 텍스처가 부족하거나 없는 환경(예: 흰 벽, 어두운 복도)에서는 특징점 추출 및 매칭에 실패하여 추적을 놓치기 쉽다.1 또한, 카메라만으로는 깊이(depth)를 직접 측정할 수 없어 스케일 모호성(scale ambiguity) 문제가 발생하며, 3차원 맵 포인트를 생성하기 위한 삼각측량(triangulation) 등의 과정에서 상당한 계산 부하가 발생한다.3

반면, 라이다 SLAM(LiDAR SLAM)은 레이저를 이용하여 주변 환경까지의 거리를 직접적이고 정밀하게 측정하므로 조명 변화에 강인하고 정확한 기하학적 구조를 획득할 수 있다. 하지만 복도나 넓은 평원과 같이 기하학적 특징이 부족한 퇴화 환경(degenerate environment)에서는 위치 추정에 어려움을 겪는다.1 또한, 라이다로 생성된 포인트 클라우드 맵은 색상이나 텍스처 정보가 없어 현실감 있는 맵을 제공하지 못하며, 구조화되지 않은 시나리오에서는 Z축 드리프트(drift)가 누적되는 경향이 있다.1

관성 측정 장치(Inertial Measurement Unit, IMU)는 가속도계와 자이로스코프를 통해 매우 높은 주기로 시스템의 움직임 정보를 제공한다. 이는 라이다나 카메라와 같은 저주파 센서 측정 간의 상태를 예측(propagate)하고, 라이다 스캔 시 발생하는 움직임 왜곡을 보정하는 데 필수적이다. 그러나 IMU는 측정 오차가 시간에 따라 빠르게 누적되기 때문에, 다른 외부 센서(exteroceptive sensor)에 의해 주기적으로 보정되지 않으면 단독으로 사용할 수 없다.

이러한 각 센서의 상호 보완적인 특성 때문에 라이다-관성-시각 주행계(LiDAR-Inertial-Visual Odometry, LIVO) 시스템의 필요성이 대두되었다. LIVO는 세 가지 센서의 장점을 결합하여 단일 센서로는 작동이 불가능한 열악한 환경에서도 강건하고 정확한 위치 추정을 가능하게 하는 시너지 효과를 창출한다.1


FAST-LIVO2는 독립적으로 탄생한 기술이 아니라, 홍콩대학교 MARS 연구실에서 수년간 진행해 온 연구의 정점에 있는 산물이다. 이 시스템의 혁신성을 이해하기 위해서는 그 이전 버전들의 발전 과정을 살펴보는 것이 필수적이다. 이 발전 과정은 특정 문제점을 해결하기 위한 반복적인 개선의 역사로 요약될 수 있다.

1. **FAST-LIO:** 이 프레임워크의 시작은 계산적으로 효율적이고 강건한 라이다-관성 주행계(LIO) 패키지인 FAST-LIO였다. 이는 긴밀하게 결합된 반복 확장 칼만 필터(iterated extended Kalman filter)를 사용하여 LIO 문제를 해결했으며, 이 필터 기반 접근 방식은 이후 프레임워크의 핵심 골격이 되었다.6
2. **FAST-LIO2:** 다음 단계는 기존 LOAM 계열 SLAM의 특징점 추출 방식이 갖는 비효율성과 특정 환경에서의 한계를 극복하는 것이었다. FAST-LIO2는 라이다의 원시 포인트(raw points)를 특징점 추출 과정 없이 직접 맵에 정합하는 '직접 방식(direct method)'을 도입했다. 또한, 맵 관리를 위해 증분 k-d 트리(incremental k-d tree, ikd-Tree)라는 효율적인 자료 구조를 사용하여 시스템을 더 빠르고 정확하며, 다양한 종류의 라이다에 적용할 수 있도록 만들었다.6 이 '직접 방식' 철학은 FAST-LIVO2의 핵심적인 기반이 되었다.
3. **FAST-LIVO:** 라이다-관성 시스템만으로는 기하학적으로 퇴화된 환경에서의 강건성 확보와 컬러 맵 생성에 한계가 있었다. 이를 해결하기 위해 FAST-LIVO는 FAST-LIO2 프레임워크에 처음으로 시각 정보를 통합했다. 이 시스템은 맵 포인트에 이미지 패치(patch)를 부착하고, 직접 방식의 VIO 서브시스템을 추가했다.5 하지만 FAST-LIVO는 비동기적 업데이트(asynchronous update) 방식을 사용했는데, 이는 서로 다른 센서 데이터를 융합하는 데 있어 최적의 강건성을 제공하지 못하는 약점으로 지적되었다.3
4. **FAST-LIVO2:** FAST-LIVO2는 바로 이 약점을 개선하기 위해 등장했다. FAST-LIO2의 직접 방식과 필터 기반 접근법, 그리고 FAST-LIVO의 라이다-시각 융합 개념을 계승하면서, 이종(heterogeneous) 센서 데이터를 보다 강건하게 처리하기 위한 **순차적 업데이트(sequential update)** 전략과 더욱 정교해진 **통합 맵(unified map)** 관리 기법을 도입하여 완성도를 높였다.3

이러한 발전은 HKU-MARS 연구실이 구축한 포괄적인 생태계 덕분에 가속화될 수 있었다. 공개된 소스 코드, 자체 제작한 데이터셋, 하드웨어 설계 도면, 그리고 센서 교정 툴킷 등은 엄격한 테스트와 빠른 반복 개발을 가능하게 하는 선순환 구조를 만들었다.10


FAST-LIVO2의 우수성은 상태 추정 엔진, 직접 방식의 센서 처리, 그리고 통합된 맵 표현이라는 세 가지 핵심 기술 요소의 유기적인 결합에 있다.


FAST-LIVO2의 심장은 상태 추정기, 즉 필터에 있다. 이 시스템은 오차 상태 반복 칼만 필터(Error-State Iterated Kalman Filter, ESIKF)를 채택하고, 여기에 독창적인 순차적 업데이트 전략을 결합했다. 이는 이전 버전 및 최적화 기반 경쟁 시스템들과 FAST-LIVO2를 구분 짓는 가장 큰 아키텍처적 특징이다.

- **ESIKF 프레임워크:** 시스템은 IMU, 라이다, 이미지 측정값을 ESIKF를 통해 융합한다.3 '오차 상태(Error-State)' 방식은 크고 비선형적인 전체 상태(global state)와 작고 거의 선형적인 오차 상태(error state)를 분리하여 다룬다. 이로 인해 필터 업데이트 과정이 수치적으로 더 안정적이고 계산적으로 효율적이게 된다. '반복(Iterated)' 필터는 업데이트된 상태 추정치를 중심으로 측정 모델을 다시 선형화하는 과정을 반복함으로써, 강한 비선형성을 갖는 시스템에서도 높은 정확도를 달성할 수 있게 한다.

- **순차적 업데이트 전략:** 이는 FAST-LIVO의 비동기 방식에 대한 핵심적인 개선점이다.3 라이다 스캔(수천 개의 포인트)과 시각 제약(소수의 패치) 사이의 현저한 차원 불일치를 효과적으로 다루기 위해, FAST-LIVO2는 단일 필터 내에서 상태를 순차적으로 업데이트한다. 구체적인 과정은 다음과 같다: 먼저 IMU 예측을 통해 얻은 사전 상태(prior state)를 라이다 측정값으로부터 얻은 기하학적 제약 조건을 이용해 1차로 업데이트한다. 그 다음, 이렇게 

  **새롭게 정제된 상태**를 즉시 카메라 이미지로부터 얻은 광도(photometric) 제약 조건을 이용해 2차로 업데이트한다.3 이 방식은 시각 업데이트가 가장 최신의 기하학 정보를 활용하도록 보장하여 센서 간의 결합을 더욱 긴밀하게 만들고 시스템 전체의 강건성을 향상시킨다.


FAST-LIVO2의 높은 효율성과 열악한 환경에서의 강건성은 추상화된 특징점을 사용하지 않고 원시 센서 데이터를 직접 활용하는 '직접 방식'을 일관되게 채택한 데서 기인한다.

- **직접 라이다 주행계:** 라이다 모듈은 LOAM이나 LIO-SAM처럼 명시적인 기하학적 특징(모서리, 평면 등)을 추출하지 않는다. 대신, 원시 포인트 클라우드를 기존의 복셀 맵에 직접 정합하여 포인트-평면 간의 잔차(residual)를 최소화한다.3 이는 특징점 추출에 드는 계산 비용과 실패 가능성을 원천적으로 제거하여, 특히 복잡하거나 비구조적인 환경에서 큰 이점을 제공한다.8
- **직접 시각 주행계:** 유사하게, 시각 모듈도 ORB-SLAM이나 VINS-Fusion처럼 ORB나 FAST 같은 특징점을 추출하고 매칭하지 않는다. 대신, 입력 이미지와 맵에 저장된 참조 패치(reference patch) 간의 직접적인 광도 오차(photometric error)를 최소화하는 방식을 사용한다.3 이 방식은 소수의 희소한 코너점뿐만 아니라 이미지 내의 모든 밝기 변화(intensity gradient) 정보를 활용할 수 있어, **텍스처가 부족한 환경에서 유리할 수 있다**.


이종 센서들을 긴밀하게 결합하는 능력은 라이다와 비전을 위한 별도의 맵을 유지하고 동기화하는 복잡성을 피하는 통합 맵 표현 방식 덕분에 가능하다. 이러한 설계는 '통합을 통한 효율성' 원칙을 명확히 보여준다. 별도의 맵을 관리하는 것은 복잡한 데이터 연관 및 동기화 문제를 야기하며, 시각 특징점을 독립적으로 삼각측량하는 것은 스케일 드리프트와 계산 비용을 증가시킨다. 따라서 시각 모듈이 기하학적으로 정확한 라이다 포인트를 직접 재사용하게 함으로써 이러한 문제들을 근본적으로 해결한다. 이는 시각 주행계의 품질이 라이다 맵의 품질에 직접적으로 의존하는 강력한 인과 관계를 형성한다.

- **구조:** FAST-LIVO2는 단일 통합 복셀 맵(unified voxel map)을 사용하며, 이 맵을 효율적으로 저장하기 위해 해시 인덱싱된 옥트리(hash-indexed octree) 구조를 활용한다.3
- **라이다의 역할:** 라이다 모듈은 이 맵의 기하학적 구조를 구축하고 유지하는 책임을 진다. 각 복셀은 지역 표면 정보를 담고 있으며, 이는 주로 평면 특징(surfel)으로 표현된다.3
- **시각 모듈의 역할:** 시각 모듈은 라이다 맵 포인트를 자신의 시각적 랜드마크로 **재사용**한다. 자체적으로 3D 포인트를 삼각측량하는 대신, 복셀 맵 내의 기하학적으로 정확한 라이다 포인트에 참조 이미지 패치를 부착한다. 새로운 이미지가 들어오면, 이 패치들을 현재 뷰로 투영하고 광도 오차를 최소화하여 이미지 정렬을 수행한다.3 시각 추적을 위해 라이다의 기하학 정보를 직접 재사용하는 이 방식이야말로 긴밀한 결합의 핵심이다.


핵심 아키텍처 외에도, FAST-LIVO2는 정확도와 강건성의 한계를 넘어서기 위한 여러 정교한 기술들을 포함하고 있다.

- **라이다 평면 사전 정보 활용:** 모든 패치 내 픽셀이 동일한 깊이를 갖는다고 가정한 FAST-LIVO와 달리, FAST-LIVO2는 맵의 라이다 포인트로부터 얻은 지역 평면 정보를 활용하여 이미지 정렬 시 더 정확한 아핀 변환(affine warping)을 수행한다. 심지어 이 평면 정보를 업데이트 과정에서 함께 정제하여 정확도를 크게 향상시켰다.3
- **동적 패치 업데이트:** 외형 변화에 대응하기 위해, 맵 포인트에 부착된 참조 이미지 패치는 새로운 이미지가 성공적으로 정렬된 후에 동적으로 업데이트된다.12
- **주문형 레이캐스팅:** 라이다 데이터가 희소하거나 카메라 시야에는 있지만 라이다 시야에는 없는 사각지대의 경우, 시스템은 '주문형 레이캐스팅(on-demanding raycast)'을 통해 복셀 맵에서 시각 맵 포인트를 가져와 시각 추적을 지속할 수 있도록 한다.12
- **실시간 노출 시간 추정:** 직접 방식 시각 주행계의 주된 실패 원인인 조명 변화에 대응하기 위해, FAST-LIVO2는 이미지 노출 시간을 상태 벡터의 일부로 포함하여 실시간으로 추정함으로써 강건성을 높인다.12

이러한 직접 방식의 선택은 양날의 검과 같다. 효율성과 텍스처가 부족한 장면에서의 성능이라는 큰 이점을 제공하는 동시에, 초기 포즈 추정치와 센서의 외부 파라미터 교정(extrinsic calibration)에 대한 강한 의존성을 낳는다. 외부 파라미터 교정의 작은 오차는 광도 및 기하학적 잔차 계산에서 지속적인 오차로 나타나게 된다. 따라서 알고리즘 자체는 환경 변화에 강건하지만, 그 구현 및 성능은 세심한 사전 교정에 크게 좌우되며, 사용자들이 겪는 어려움이 주로 교정 문제에 집중되는 이유가 여기에 있다.17


FAST-LIVO2의 가치는 이론적 우수성뿐만 아니라 실제 환경에서의 성능으로 증명된다. 특히 계산 효율성과 위치 추정 정확도는 이 시스템의 핵심적인 성과이다.


FAST-LIVO2의 주요 목표 중 하나는 고성능 LIVO 시스템을 자원이 제한된 플랫폼에 탑재할 수 있도록 만드는 것이었고, 이는 성공적으로 달성되었다.

- **실시간 온보드 UAV 항법:** FAST-LIVO2는 LIVO 시스템 중 세계 최초로 완전한 온보드 자율 비행 UAV에 적용된 사례로 알려져 있다.18 이는 모든 상태 추정, 경로 계획, 제어 알고리즘이 UAV의 제한된 온보드 컴퓨터에서 실시간으로 실행되어야 하므로, 시스템의 효율성을 입증하는 가장 강력한 증거다.
- **ARM 플랫폼에서의 성능:** 이 시스템은 Jetson Orin NX, RK3588과 같은 저전력 ARM 기반 플랫폼에서도 실시간으로 작동함이 시연되었다.19
- **정량적 효율성 지표:** "FAST-LIVO2 on Resource-Constrained Platforms" 논문은 중요한 벤치마크를 제공한다. 이 논문에서 제안된 경량화 버전은 원본 FAST-LIVO2 대비 Hilti 데이터셋에서 프레임당 실행 시간을 33%, 메모리 사용량을 47% 감소시키면서도, 정확도 손실(RMSE 증가)은 3 cm에 불과했다.2 이는 원본 FAST-LIVO2조차도 최적화의 여지가 있는 상당한 계산 부하를 가지고 있었음을 시사한다. 아래 표 3은 x86 및 ARM 플랫폼에서의 구체적인 실행 시간과 메모리 사용량을 보여준다.

이러한 효율성-정확도 간의 상충 관계는 고정된 것이 아니라 유연하게 조절될 수 있다. 경량화 버전의 '적응형 시각 프레임 선택기'는 라이다 정보가 충분하고 신뢰할 수 있을 때는 값비싼 시각 업데이트 빈도를 동적으로 줄인다. 즉, 기하학적으로 반드시 필요할 때만 시각 정보 처리 비용을 지불함으로써, 자원이 제한된 응용 분야를 위한 더 나은 균형점을 찾을 수 있음을 보여준다.

**표 3: Hilti 데이터셋에서의 계산 및 메모리 사용량 분석**

| **아키텍처**                | **ARM**          |                  | **x86 PC**       |                  |      |
| --------------------------- | ---------------- | ---------------- | ---------------- | ---------------- | ---- |
| **SLAM 시스템**             | **경량화 버전**  | **FAST-LIVO2**   | **경량화 버전**  | **FAST-LIVO2**   |      |
| **프레임당 실행 시간 (ms)** |                  |                  |                  |                  |      |
| 라이다 파트                 | 53.83 / 5.35     | 56.50 / 5.56     | 23.36 / 2.36     | 25.05 / 2.41     |      |
| 시각 파트                   | 3.99 / 1.80      | 19.36 / 4.75     | 2.64 / 1.12      | 10.61 / 2.68     |      |
| **총계**                    | **57.82 / 6.75** | **75.87 / 9.77** | **25.99 / 3.22** | **35.66 / 4.88** |      |
| **최대 메모리 사용량 (GB)** |                  |                  |                  |                  |      |
| **평균**                    | **1.7**          | **2.5**          | -                | -                |      |

실행 시간은 평균 / 표준 오차로 표시됨. 메모리 사용량 데이터는 ARM 플랫폼에서 수집됨. 데이터 출처: 2


FAST-LIVO2는 특히 센서 성능이 저하되는 도전적인 환경에서 기존의 많은 LIO 및 LIVO 시스템을 능가하는 최첨단 위치 추정 정확도를 달성한다.

- **Hilti 데이터셋 성능:** Hilti SLAM 챌린지 데이터셋은 시스템 성능을 평가하는 핵심 벤치마크다. 아래 표 2에서 볼 수 있듯이, FAST-LIVO2는 여러 시퀀스에서 매우 낮은 절대 궤적 오차(Absolute Trajectory Error, ATE)를 기록했다. 경량화 버전은 일부 시퀀스(예: Cupola, Attic to Upper Gallery)에서 원본보다 정확도가 다소 하락했지만, 여전히 FAST-LIO2(LIO-only)나 다른 LIVO 시스템들을 능가하는 경쟁력 있는 성능을 보여준다.2 특히 긴 복도(Long Corridor)와 같이 기하학적 특징이 부족한 환경에서 시각 정보 융합이 드리프트를 억제하는 데 명확한 이점을 제공함을 알 수 있다. 반면, 조명이 좋지 않은 환경에서는 시각 정보의 기여도가 감소할 수 있다.
- **M2DGR 데이터셋:** FAST-LIVO2 논문은 시스템 평가를 위해 다양하고 도전적인 시나리오를 포함하는 M2DGR 데이터셋을 사용했다고 명시적으로 언급한다.13 이는 시스템의 강건성을 입증하는 데 중요한 부분이다.
- **지도 품질:** 시스템은 실시간으로 사진처럼 사실적인 고밀도 컬러 포인트 클라우드를 생성할 수 있다.19 그 정확도는 '픽셀 수준(pixel-level)'으로 묘사되는데 9, 이는 메쉬(mesh) 생성, NeRF, 3D 가우시안 스플래팅(Gaussian Splatting)과 같은 후속 3D 모델 렌더링 응용 분야에 매우 중요하다.12 이러한 높은 지도 품질은 단순히 융합 알고리즘의 결과물이 아니라, 정밀한 하드웨어 동기화, 엄격한 외부 파라미터 교정, 그리고 정확한 노출 시간 복구 등 전체 시스템 엔지니어링의 성공에 달려 있다.3

**표 2: Hilti SLAM 챌린지 데이터셋 성능 비교 (ATE, RMSE in meters)**

| **데이터셋** | **시퀀스**              | **경량화 버전** | **FAST-LIVO2** | **FAST-LIO2** | **FAST-LIVO** | **R3LIVE** | **SDV-LOAM** | **LVI-SAM** |      |
| ------------ | ----------------------- | --------------- | -------------- | ------------- | ------------- | ---------- | ------------ | ----------- | ---- |
| Hilti ’22    | Construction Ground     | 0.010           | **0.010**      | 0.013         | 0.022         | 0.021      | 25.121       | ×           |      |
|              | Construction Multilevel | 0.023           | **0.020**      | 0.044         | 0.052         | 0.024      | 12.561       | ×           |      |
|              | Construction Stairs     | 0.170           | **0.016**      | 0.320         | 0.241         | 0.784      | 9.212        | 9.142       |      |
|              | Long Corridor           | **0.054**       | 0.067          | 0.064         | 0.065         | 0.061      | 19.531       | 6.312       |      |
|              | Cupola                  | 0.220           | **0.121**      | 0.250         | 0.182         | 2.142      | 9.321        | ×           |      |
|              | Lower Gallery           | 0.018           | **0.007**      | 0.024         | 0.022         | 0.008      | 11.232       | 2.281       |      |
|              | Attic to Upper Gallery  | 0.180           | **0.069**      | 0.720         | 0.621         | 2.412      | 4.551        | ×           |      |
|              | Outside Building        | 0.041           | **0.035**      | 0.028         | 0.052         | 0.029      | 2.622        | 0.952       |      |
| Hilti ’23    | Floor 0                 | 0.032           | **0.021**      | 0.031         | 0.022         | 0.024      | 4.621        | ×           |      |
|              | Floor 1                 | 0.018           | 0.023          | 0.031         | 0.022         | 0.024      | 7.951        | 8.682       |      |
|              | Floor 2                 | 0.038           | **0.022**      | 0.083         | 0.048         | 0.046      | 7.912        | ×           |      |
|              | Basement                | 0.024           | **0.016**      | 0.038         | 0.035         | 0.024      | 6.151        | ×           |      |
|              | Stairs                  | **0.012**       | 0.018          | 0.170         | 0.152         | 0.110      | 9.032        | 3.584       |      |
|              | Parking 3x Floors Down  | 0.095           | **0.032**      | 0.320         | 0.356         | 0.462      | 19.952       | ×           |      |
|              | Large room              | 0.028           | **0.026**      | 0.028         | 0.031         | 0.035      | 16.781       | 0.563       |      |
|              | Large room (dark)       | 0.044           | 0.046          | **0.040**     | 0.053         | 0.059      | 15.012       | ×           |      |
| **평균**     |                         | 0.063           | **0.034**      | 0.138         | 0.123         | 0.391      | 11.347       | 4.502       |      |
|              |                         |                 |                |               |               |            |              |             |      |

×는 시스템 실패를 의미. 굵은 글씨는 최고 성능을 나타냄. 데이터 출처: 2


FAST-LIVO2를 SLAM 기술의 전체적인 맥락에서 이해하기 위해서는 다른 주요 시스템들과의 철학적, 구조적 차이를 비교하는 것이 중요하다.


SLAM 커뮤니티는 백엔드(backend) 구현 방식에 따라 크게 필터 기반(예: 칼만 필터)과 최적화 기반(예: 팩터 그래프)의 두 진영으로 나뉜다. FAST-LIVO2는 현대적인 필터 기반 접근법의 강력한 지지자인 반면, LIO-SAM이나 VINS-Fusion과 같은 시스템은 최적화 기반 진영을 대표한다.

- **필터링 방식 (FAST-LIVO2):** 측정값을 순차적으로 처리하며 현재 상태와 그 불확실성만을 유지한다. 과거 상태는 한계화(marginalize)시켜 버리기 때문에 매 스텝당 계산 복잡도가 일정하게 유지된다. 이는 실시간성과 자원 제약이 중요한 응용 분야에 이상적이다. 하지만 루프 폐쇄(loop closure)와 같은 새로운 정보가 들어왔을 때 과거 상태 전체를 다시 최적화할 수 없어 전역 일관성 면에서는 불리할 수 있다.
- **최적화 방식 (LIO-SAM, VINS-Fusion):** 과거의 상태와 측정값들을 팩터 그래프(factor graph)라는 형태로 모두 유지한다. 새로운 정보(특히 루프 폐쇄)가 들어오면, 전체 궤적을 대상으로 전역 최적화(global optimization)를 수행하여 매우 높은 전역적 일관성을 달성할 수 있다.23 그러나 시간이 지남에 따라 그래프의 크기가 계속 커져 계산 비용이 증가하므로, 실시간성을 유지하기 위해 키프레임(keyframe) 선정이나 슬라이딩 윈도우(sliding window) 같은 전략이 필수적이다.

이러한 프론트엔드 철학은 백엔드 아키텍처 선택에 직접적인 영향을 미친다. 특징점 기반 방식은 희소하지만 강건한 제약 조건(키포인트 매칭)을 생성하므로, 많은 제약 조건들을 모아 전역 최적해를 찾는 팩터 그래프에 적합하다. 반면, 직접 방식은 조밀하지만 노이즈가 많을 수 있는 잔차(광도/기하학적 오차)를 생성하는데, 이를 모두 전역 최적화에 포함하는 것은 계산적으로 비효율적이다. 따라서 측정값을 순차적으로 처리하고 과거를 한계화시키는 필터링 프레임워크가 직접 방식과 자연스럽게 어울린다.

**표 1: 최신 다중 센서 SLAM 시스템 아키텍처 비교**

| **항목**                | **FAST-LIVO2**                   | **LIO-SAM**                         | **VINS-Fusion**                 |      |
| ----------------------- | -------------------------------- | ----------------------------------- | ------------------------------- | ---- |
| **핵심 철학**           | 필터 기반, 직접 방식             | 최적화 기반, 특징점 방식            | 최적화 기반, 특징점 방식        |      |
| **백엔드 방식**         | ESIKF (오차 상태 반복 칼만 필터) | 팩터 그래프 최적화 (GTSAM)          | 비선형 최적화 (슬라이딩 윈도우) |      |
| **프론트엔드 (라이다)** | 직접 방식 (원시 포인트 정합)     | 특징점 방식 (모서리/평면점)         | 해당 없음                       |      |
| **프론트엔드 (시각)**   | 직접 방식 (광도 오차 최소화)     | 해당 없음 (LVI-SAM에서 VINS 사용)   | 특징점 방식 (KLT 코너 추적)     |      |
| **맵 표현**             | 통합 복셀 맵 (라이다+시각)       | 서브-키프레임 집합으로 구성된 맵    | 희소 특징점 맵                  |      |
| **센서 융합 전략**      | ESIKF 내 순차적 업데이트         | 팩터 그래프에 요인(factor)으로 추가 | 슬라이딩 윈도우 내 공동 최적화  |      |
| **루프 폐쇄**           | (논문에서 상세히 다루지 않음)    | 팩터 그래프에 루프 폐쇄 요인 추가   | DBoW2 기반 시각 장소 인식       |      |

데이터 출처: 3


두 시스템 모두 LIO/LIVO 계열이지만, 그 구조는 근본적으로 달라 뚜렷한 성능 차이를 보인다.

- **백엔드:** FAST-LIVO2는 ESIKF를 사용한다.3 반면, LIO-SAM은 GTSAM 라이브러리를 기반으로 한 팩터 그래프를 사용하여 스무딩 및 매핑(smoothing and mapping)을 수행한다.23
- **프론트엔드:** FAST-LIVO2는 원시 포인트를 직접 사용하는 반면, LIO-SAM은 포인트 클라우드에서 모서리(corner)와 평면(planar) 특징점을 추출하여 사용한다.3
- **융합 방식:** FAST-LIVO2는 통합 복셀 맵과 순차적 ESIKF 업데이트를 통해 센서를 융합한다.3 LIO-SAM은 IMU 사전 적분, 라이다 주행계, GPS, 루프 폐쇄 등 각 측정값을 독립적인 '요인(factor)'으로 모델링하여 팩터 그래프에 추가하는 방식을 사용한다.23 이 방식은 GPS와 같이 새로운 센서를 추가하는 데 더 유연할 수 있다.
- **강건성:** FAST-LIVO2는 시각 정보를 활용하여 라이다 퇴화 환경에서 강건성을 확보한다.19 LIO-SAM은 양호한 기하학적 특징에 크게 의존하며 드리프트가 발생할 수 있지만, 전역 최적화와 루프 폐쇄를 통해 이를 보정할 수 있다.27 한 사용자는 FAST-LIVO2가 대부분의 퇴화 문제를 해결하지만 완전한 암흑 환경에서는 어려움을 겪는다고 지적했다.28


VINS-Fusion은 최적화 기반 시각-관성 SLAM의 대표적인 시스템으로, FAST-LIVO2와의 비교는 시각 데이터를 처리하고 융합하는 방식의 차이를 명확히 보여준다.

- **주요 센서:** VINS-Fusion은 근본적으로 VIO/VINS 시스템으로, 카메라가 주된 외부 센서 역할을 한다.24 반면 FAST-LIVO2는 LIO 시스템에서 진화했으며, 라이다가 주된 기하학적 구조를 제공한다.
- **백엔드:** FAST-LIVO2는 ESIKF를, VINS-Fusion은 키프레임의 슬라이딩 윈도우에 대한 비선형 최적화를 사용한다.3
- **프론트엔드:** FAST-LIVO2는 직접적인 광도 정렬을, VINS-Fusion은 KLT(Kanade-Lucas-Tomasi) 코너를 추적하는 특징점 기반 방식을 사용한다.3
- **센서 지원:** VINS-Fusion은 모노, 스테레오, 스테레오+IMU 등 다양한 센서 구성을 지원하여 유연성이 높다.24 FAST-LIVO2는 라이다-관성-시각 조합에 특화되어 설계되었다.



FAST-LIVO2는 학술적 연구에 그치지 않고, 강력한 오픈소스 생태계를 기반으로 실제 세계에 영향을 미치는 실용적인 시스템이다.

- **UAV 자율 항법:** 대표적인 응용 사례는 UAV의 완전한 온보드 자율 항법이다. 라이다나 비전 단독으로는 실패할 수 있는 열악한 환경에서도 안정적인 비행이 가능하다.18

- **항공 매핑:** 항공 측량에도 효과적으로 사용된다. 특히 지상고가 높아 라이다 성능이 저하될 때 시각 정보가 드리프트를 보정하는 데 도움을 준다.19

- **3D 재구성 및 디지털 트윈:** 빠르고 조밀하며 정확한 컬러 포인트 클라우드를 생성하는 능력은 메쉬 재구성, NeRF, 3D 가우시안 스플래팅과 같은 후속 응용을 위한 이상적인 데이터 소스가 된다.13 고대 건축물이나 풍경을 스캔하여 UE5와 같은 게임 엔진용 에셋을 제작하는 데 사용되기도 했다.19

- **오픈소스 프로젝트:** 소스 코드는 GPLv2 라이선스로 깃허브(GitHub)에 공개되어 있다.10 HKU-MARS 연구실은 관련 데이터셋(

  `FAST-LIVO2-Dataset`), 교정 툴킷(`FAST-Calib`), 휴대용 장비 설계도(`LIV_handhold`) 등 전체 생태계를 함께 제공한다.10 이러한 개방성은 다른 연구자들이 시스템을 쉽게 도입하고, 테스트하며, 개선하는 것을 가능하게 하여 전체 분야의 발전을 가속화하고 원본 연구의 중요성을 공고히 하는 선순환을 만든다.


강력한 성능에도 불구하고, FAST-LIVO2는 성공적인 적용을 위해 반드시 이해해야 할 본질적인 한계와 실용적인 과제를 안고 있다.

- **계산 및 메모리 요구량:** 원본 시스템은 상당한 계산 및 메모리 자원을 필요로 한다. 이는 "FAST-LIVO2 on Resource-Constrained Platforms" 연구의 주된 동기가 되었다.2 이는 기본 버전이 별도의 최적화 없이는 초저전력 임베디드 시스템에 적합하지 않을 수 있음을 의미한다.
- **외부 파라미터 교정에 대한 민감성:** 이는 매우 중요한 실용적 과제다. 직접 방식을 사용하는 시스템으로서, 성능이 라이다, 카메라, IMU 간의 정확한 사전 교정 값에 크게 의존한다. 한 깃허브 이슈에서는 이것이 "사용자들이 재현 시 직면할 가장 큰 문제"일 수 있다고 지적했다.17
- **극한의 시각적 열화 환경:** 많은 도전에 강건하지만, 극심한 시각적 조건에서는 여전히 영향을 받을 수 있다. 한 사용자는 완전한 암흑 속에서 항법 문제를 해결하지는 못한다고 언급했다.28 Hilti 데이터셋 결과에서도 조명이 좋지 않은 시나리오에서는 라이다 전용 시스템인 FAST-LIO2에 비해 성능이 소폭 하락하는 것을 보여준다.15
- **고수준 의미론적 이해의 부재:** 시스템은 저수준의 기하학적, 광도적 데이터에 기반하여 작동한다. 최신 시맨틱 SLAM 시스템들처럼 동적 객체와 정적 객체를 구분하여 다르게 처리하는 등의 의미론적 정보를 통합하지는 않는다.29


FAST-LIVO2는 종착점이 아니라 디딤돌이다. 이 시스템의 강건한 주행계는 이제 고품질 장면 표현에 초점을 맞춘 차세대 SLAM 시스템의 핵심 구성 요소로 활용되고 있다.

- **가우시안 스플래팅과의 통합:** 다수의 최신 연구들이 FAST-LIVO2의 주행계 파이프라인을 기반으로 실시간 3D 가우시안 스플래팅 맵을 생성하는 SLAM 시스템을 구축하고 있다. GS-LIVO와 Gaussian-LIC2가 대표적인 예다.20 이들은 단순한 광도 오차 최소화를, 가우시안 맵에서 렌더링된 이미지와 실제 카메라 이미지를 비교하는 더 복잡한 렌더링 손실(rendering loss)로 대체한다.
- **'서비스로서의 주행계(Odometry as a Service)' 트렌드:** 이는 강건한 상태 추정(주행계)이 더 복잡한 매핑 및 장면 이해 백엔드를 위한 모듈화된 서비스가 되어가는 더 큰 흐름을 보여준다. 주행계의 역할은 정확한 포즈를 제공하고, 백엔드의 역할은 그 포즈를 사용하여 풍부하고 사실적인 월드 모델을 구축하는 것으로 분업화되는 것이다. SLAM 시스템이 점점 더 복잡해지면서, 모든 것을 다 하는 단일(monolithic) 시스템은 비현실적이 되고 있다. 따라서 고도로 최적화된 주행계 프론트엔드와 고품질 매핑 백엔드로 자연스럽게 아키텍처가 분리되는 것은, 이 분야가 성숙한 공학 분야로 발전하고 있다는 신호다.
- **향후 연구:** 한 리뷰에서는 FAST-LIVO2와 같은 시스템의 향후 연구 방향으로 더 정교한 루프 폐쇄나 전역 최적화 기법을 통합하여 장기적인 드리프트를 해결하는 것을 제안했으며 14, 이는 필터링과 최적화 접근법 사이의 경계를 더욱 허물게 될 것이다.


FAST-LIVO2는 현대적인 필터 기반, 직접 방식 SLAM의 정점을 대표하는 매우 효율적이고 정확하며 강건한 LIVO 시스템이다. 순차적 ESIKF와 통합 복셀 맵이라는 핵심 혁신은 긴밀한 센서 결합과 자원이 제한된 하드웨어에서의 실시간 성능을 가능하게 했다. UAV 자율 비행과 같은 까다로운 실제 응용 분야에서의 성공은 그 실용성을 입증한다.

그러나 이 시스템의 강력한 성능은 세심한 외부 파라미터 교정이라는 실용적인 전제 조건에 크게 의존하며, 원본 시스템은 상당한 계산 자원을 요구하는 한계를 가진다.

궁극적으로 FAST-LIVO2의 가장 중요한 가치는 그 자체의 성능을 넘어, 차세대 SLAM 기술의 발전을 위한 견고한 기반이 되고 있다는 점에 있다. 3D 가우시안 스플래팅과 같은 고품질 맵 표현 기술이 부상함에 따라, FAST-LIVO2와 같은 강건한 주행계는 복잡한 백엔드를 위한 신뢰할 수 있는 '포즈 서버'로서의 역할을 수행하며 SLAM 기술의 모듈화 및 전문화 시대를 이끌고 있다. 이는 FAST-LIVO2가 현재의 최첨단 기술일 뿐만 아니라, 미래 SLAM 연구의 필수적인 구성 요소임을 시사한다.


1. LPVIMO-SAM: Tightly-coupled LiDAR/Polarization Vision/Inertial/Magnetometer/Optical Flow Odometry via Smoothing and Mapping - arXiv, accessed July 19, 2025, https://arxiv.org/html/2504.20380v1
2. FAST-LIVO2 on Resource-Constrained Platforms: LiDAR-Inertial-Visual Odometry with Efficient Memory and Computation - arXiv, accessed July 19, 2025, https://arxiv.org/html/2501.13876v1
3. FAST-LIVO2: Fast, Direct LiDAR-Inertial-Visual Odometry - arXiv, accessed July 19, 2025, https://arxiv.org/html/2408.14035v2
4. Student Projects - ETH Zurich - Autonomous Systems Lab, accessed July 19, 2025, https://asl.ethz.ch/v4rl/education/student-projects.html
5. LIR-LIVO: A Lightweight,Robust LiDAR/Vision/Inertial Odometry with Illumination-Resilient Deep Features - arXiv, accessed July 19, 2025, https://arxiv.org/html/2502.08676v1
6. Ericsii/FAST_LIO_ROS2: ROS2 version of FAST_LIO2. Welcome to the technical communication discord server discord: https://discord.gg/U3B65MGH8m - GitHub, accessed July 19, 2025, https://github.com/Ericsii/FAST_LIO_ROS2
7. hku-mars/FAST_LIO: A computationally efficient and robust LiDAR-inertial odometry (LIO) package - GitHub, accessed July 19, 2025, https://github.com/hku-mars/FAST_LIO
8. FAST-LIO2: Fast Direct LiDAR-inertial Odometry - YouTube, accessed July 19, 2025, https://www.youtube.com/watch?v=2OvjGnxszf8
9. hku-mars/FAST-LIVO: A Fast and Tightly-coupled Sparse-Direct LiDAR-Inertial-Visual Odometry (LIVO). - GitHub, accessed July 19, 2025, https://github.com/hku-mars/FAST-LIVO
10. FAST-LIVO2: Fast, Direct LiDAR-Inertial-Visual Odometry - GitHub, accessed July 19, 2025, https://github.com/hku-mars/FAST-LIVO2
11. HKU-Mars-Lab - GitHub, accessed July 19, 2025, https://github.com/hku-mars
12. FAST-LIVO2: Fast, Direct LiDAR-Inertial-Visual Odometry | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/383428735_FAST-LIVO2_Fast_Direct_LiDAR-Inertial-Visual_Odometry
13. FAST-LIVO2: Fast, Direct LiDAR-Inertial-Visual Odometry | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/385964386_FAST-LIVO2_Fast_Direct_LiDAR-Inertial-Visual_Odometry
14. [Literature Review] FAST-LIVO2: Fast, Direct LiDAR-Inertial-Visual ..., accessed July 19, 2025, https://www.themoonlight.io/en/review/fast-livo2-fast-direct-lidar-inertial-visual-odometry
15. [2501.13876] FAST-LIVO2 on Resource-Constrained Platforms ..., accessed July 19, 2025, https://ar5iv.labs.arxiv.org/html/2501.13876
16. [2408.14035] FAST-LIVO2: Fast, Direct LiDAR-Inertial-Visual Odometry - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2408.14035
17. Will the accuracy of this method be higher than FAST LIO2? / Issue #62 - GitHub, accessed July 19, 2025, https://github.com/hku-mars/FAST-LIVO2/issues/62
18. Fast-LIVO对重建的计算机有什么需求？ #168 - GitHub, accessed July 19, 2025, https://github.com/hku-mars/FAST-LIVO/issues/168
19. FAST-LIVO2: Fast, Direct LiDAR-Inertial-Visual Odometry - YouTube, accessed July 19, 2025, https://www.youtube.com/watch?v=6dF2DzgbtlY
20. GS-LIVO: Real-Time LiDAR, Inertial, and Visual Multi-sensor Fused Odometry with Gaussian Mapping - arXiv, accessed July 19, 2025, https://arxiv.org/html/2501.08672v1
21. FAST-LIVO2 on Resource-Constrained Platforms: LiDAR-Inertial-Visual Odometry with Efficient Memory and Computation | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/388354428_FAST-LIVO2_on_Resource-Constrained_Platforms_LiDAR-Inertial-Visual_Odometry_with_Efficient_Memory_and_Computation
22. FAST-LIVO: Fast and Tightly-coupled Sparse-Direct LiDAR-Inertial-Visual Odometry | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/366611115_FAST-LIVO_Fast_and_Tightly-coupled_Sparse-Direct_LiDAR-Inertial-Visual_Odometry
23. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and ..., accessed July 19, 2025, http://arxiv.org/pdf/2007.00258
24. HKUST-Aerial-Robotics/VINS-Fusion: An optimization ... - GitHub, accessed July 19, 2025, https://github.com/HKUST-Aerial-Robotics/VINS-Fusion
25. An Effective LiDAR-Inertial SLAM-Based Map Construction Method for Outdoor Environments - MDPI, accessed July 19, 2025, https://www.mdpi.com/2072-4292/16/16/3099
26. Visual-Inertial SLAM Comparison - Bharat Joshi - GitHub Pages, accessed July 19, 2025, https://joshi-bharat.github.io/projects/visual_slam_comparison/
27. Mapping results of LOAM, LIOM, and LIO-SAM using the Walking dataset.... - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/figure/Mapping-results-of-LOAM-LIOM-and-LIO-SAM-using-the-Walking-dataset-The-map-of-LOAM-in_fig3_362412728
28. When will FAST-LIVO3 be released? / Issue #328 / hku-mars/FAST-LIVO2 - GitHub, accessed July 19, 2025, https://github.com/hku-mars/FAST-LIVO2/issues/328
29. A Stereo Visual-Inertial SLAM Algorithm with Point-Line Fusion and Semantic Optimization for Forest Environments - MDPI, accessed July 19, 2025, https://www.mdpi.com/1999-4907/16/2/335
30. Gaussian-LIC2: LiDAR-Inertial-Camera Gaussian Splatting SLAM - arXiv, accessed July 19, 2025, https://arxiv.org/html/2507.04004v2

