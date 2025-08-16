# Dynamic-LIO 심층 분석

------

## I. 서론: 동적 환경 SLAM의 난제와 LIO 프레임워크의 진화

### 0.1 A. 문제 정의: 정적 세계 가정의 붕괴

전통적인 동시적 위치 추정 및 지도 작성(Simultaneous Localization and Mapping, SLAM) 및 LiDAR-Inertial Odometry(LIO) 알고리즘의 근간에는 '정적 세계(static world)'라는 암묵적인 가정이 존재한다.1 이 가정은 환경 내 모든 객체가 움직이지 않고 고정되어 있다는 전제를 의미하며, 알고리즘은 이러한 정적 구조물로부터 일관된 기하학적 제약(geometric constraints)을 추출하여 자신의 위치를 추정하고 지도를 작성한다. 그러나 이러한 가정은 차량, 보행자, 자전거 등 수많은 동적 객체(dynamic objects)가 존재하는 실제 도심 주행 환경에서는 명백히 붕괴된다.1

정적 세계 가정이 깨졌을 때, LIO 시스템은 세 가지 치명적인 문제에 직면하게 된다.

1. **프론트엔드 오염 (Front-end Contamination):** LIO의 프론트엔드는 연속된 LiDAR 스캔 간의 상대적인 움직임을 추정하는 역할을 한다. 동적 객체에서 얻어진 포인트 클라우드는 잘못된 위치 정보를 포함하므로, 이를 기반으로 계산된 기하학적 제약은 실제 움직임과 무관한 오차를 유발한다. 이는 스캔 매칭(scan matching) 과정의 정확도를 심각하게 저하시켜, Odometry 추정의 초기 단계부터 오차를 누적시키는 원인이 된다.1
2. **맵 오염 (Map Contamination):** 프론트엔드에서 필터링되지 않은 동적 객체의 포인트들은 전역 지도(global map)에 그대로 누적된다. 이는 지도 상에 '유령 흔적(ghost tracks)'을 남겨 맵의 품질을 저하시킬 뿐만 아니라, 기존에 존재하던 정적 구조물의 특징을 가리거나 왜곡하는 결과를 초래한다.1 이렇게 오염된 지도는 이후의 위치 추정이나 경로 계획과 같은 다운스트림 작업에 심각한 악영향을 미친다. 특히, 로봇이 이전에 방문했던 장소를 다시 방문하여 루프를 닫으려 할 때, 동적 객체의 부재나 위치 변화로 인해 재인식(place recognition)에 실패하게 된다.2
3. **백엔드 실패 (Back-end Failure):** 백엔드는 루프 클로저(loop closure) 탐지나 전역 최적화(global optimization)를 통해 프론트엔드에서 누적된 오차를 보정하는 역할을 한다. 하지만 오염된 맵과 잘못된 프론트엔드 제약은 잘못된 루프 클로저 탐지로 이어질 수 있으며, 이는 맵 전체를 왜곡시키는 결과를 낳는다. 결국 시스템은 누적된 오차를 보정할 기회를 상실하고, 궤적 추정의 신뢰도는 급격히 하락한다.2

이러한 문제들은 단순히 LIO 시스템의 성능 저하에 그치지 않고, 자율주행 차량이나 로봇의 안정성과 안전에 직접적인 위협이 되기 때문에, 동적 환경에서의 강인성(robustness) 확보는 LIO 연구의 핵심 과제로 부상했다.4

### 0.2 B. 해결을 위한 접근법: 동적 객체 필터링 기법의 분류

동적 객체 문제를 해결하기 위한 연구는 크게 여러 갈래로 발전해왔다. 이러한 접근법들을 이해하는 것은 Dynamic-LIO가 채택한 방식의 독창성과 그 전략적 배경을 파악하는 데 필수적이다.4

- **가시성 기반 (Visibility-based) 접근법:** 이 방법은 현재 스캔에서 관측된 포인트가 이전에 구축된 맵의 포인트를 가리는 현상을 이용한다. 만약 현재 레이저 빔이 통과하는 경로 상에 이전에 관측된 맵 포인트가 존재한다면, 그 맵 포인트는 동적 객체로 간주될 수 있다는 원리다. 계산 비용이 상대적으로 낮다는 장점이 있지만, 시점 변화나 객체에 의한 가림(occlusion) 현상에 취약하며, 뒤에 정적 배경이 없는 큰 동적 객체를 탐지하기 어렵다는 한계가 있다.3
- **점유 격자/레이 트레이싱 기반 (Occupancy Grid/Ray-tracing-based) 접근법:** 공간을 작은 3차원 격자(voxel)로 나누고, 각 격자가 레이저 빔에 의해 '점유(occupied)'되는지 혹은 '통과(free)'되는지를 확률적으로 모델링한다. 정적 객체는 반복적으로 동일한 격자를 점유하지만, 동적 객체는 일시적으로만 점유하므로, 이 확률 값의 변화를 추적하여 동적 객체를 식별할 수 있다. 높은 정확도를 기대할 수 있지만, 방대한 격자 지도를 실시간으로 유지하고 레이저 빔을 추적하는 데 막대한 계산 비용이 소요되어 실시간 LIO 시스템에 직접 적용하기에는 부담이 크다.3
- **세분화 기반 (Segmentation-based) 접근법:** 딥러닝 기술을 활용하여 포인트 클라우드에서 '차량', '보행자', '자전거' 등 의미론적(semantic) 정보를 직접 추출한다. 이후, 동적으로 움직일 가능성이 있는 카테고리(예: 차량, 보행자)에 속한 포인트들을 제거하는 방식이다. 높은 탐지 정확도를 보이지만, 대규모로 레이블링된 학습 데이터셋과 고성능 GPU 자원에 크게 의존한다. 또한, 학습 데이터에 포함되지 않은 새로운 형태의 동적 객체는 탐지하지 못하는 한계를 지닌다.2

### 0.3 C. LIO 프레임워크의 발전: Tightly-coupled 방식의 부상

초기 LIO 시스템들은 LiDAR와 IMU(관성 측정 장치, Inertial Measurement Unit) 센서 데이터를 비교적 독립적으로 처리하는 Loosely-coupled(느슨한 결합) 방식을 주로 사용했다. 이 방식은 구현이 간단하지만, 두 센서가 제공하는 정보의 상호 보완성을 극대화하지 못하는 단점이 있었다.9

이후 LIO 기술이 발전하면서, LIO-SAM, FAST-LIO와 같은 최신 고성능 LIO 시스템들은 Tightly-coupled(강한 결합) 방식을 표준으로 채택하게 되었다. Tightly-coupled 프레임워크는 두 센서의 원시 측정값(raw measurement)을 하나의 최적화 문제로 통합하여 푼다. 구체적으로, IMU의 높은 주파수(high-frequency) 측정값을 이용해 LiDAR 스캔 과정에서 발생하는 포인트의 왜곡(motion distortion)을 정밀하게 보정한다. 동시에, LiDAR의 정확한 기하학적 측정값을 이용해 IMU의 바이어스(bias)와 같은 내부 파라미터를 실시간으로 추정하고 보정한다. 이러한 상호 보완적인 융합을 통해 시스템의 전반적인 정확도와 강인성을 크게 향상시킬 수 있다.1

LIO 기술의 발전은 '정확도'와 '실시간성/효율성'이라는 두 축 사이의 끊임없는 트레이드오프(trade-off) 과정이었다. LOAM이 특징점 기반 LIO의 기틀을 마련했고, LIO-SAM은 팩터 그래프(factor graph) 최적화를 통해 전역 일관성을 확보했으며, FAST-LIO는 반복 확장 칼만 필터(iEKF)를 통해 프론트엔드의 속도를 극대화했다. 그러나 이들 모두 동적 환경이라는 근본적인 문제 앞에서는 한계를 보였다. LIO 시스템은 상태 추정과 맵 업데이트라는 핵심 작업을 수행하기 위해 이미 많은 계산 자원을 사용하고 있으므로, 동적 객체 제거와 같은 추가적인 작업을 위해 할당할 수 있는 계산 예산(computational budget)은 매우 제한적이다.1 기존의 동적 객체 제거 방식, 특히 세분화나 점유 격자 기반 방법들은 계산 비용이 너무 높아 LIO의 실시간성을 해칠 위험이 컸다.3

이러한 배경에서 Dynamic-LIO의 등장은 LIO 기술의 트레이드오프에 '동적 강인성(dynamic robustness)'이라는 새로운 축을 추가하려는 중요한 시도로 해석될 수 있다. Dynamic-LIO는 기존 방법들의 계산적 한계를 명확히 인지하고, LIO 파이프라인의 실시간성을 저해하지 않으면서 동적 객체를 효과적으로 필터링할 수 있는 경량화된 접근법을 채택했다. 이는 단순히 새로운 알고리즘의 제시를 넘어, 시스템 전체의 제약 조건을 고려한 전략적 설계의 결과물이라 할 수 있다.

------

## 1. II. Dynamic-LIO의 구조와 핵심 원리

### 1.1 A. 시스템 아키텍처 개요

Dynamic-LIO는 복잡한 도심 주행 시나리오와 같이 동적 객체가 많은 환경에서 강인한 성능을 발휘하도록 설계된 Tightly-coupled LiDAR-Inertial Odometry 시스템이다.10 이 시스템의 아키텍처는 완전히 새로운 프레임워크를 처음부터 구축하는 대신, 이미 성능이 검증된 두 개의 강력한 오픈소스 프로젝트를 기반으로 구성하여 효율성과 안정성을 동시에 추구했다.

- **프론트엔드 (Front-end):** Odometry 추정의 핵심인 프론트엔드는 **SR-LIO** 프레임워크를 기반으로 한다.10 SR-LIO는 반복 확장 칼만 필터(iEKF)를 사용하여 LiDAR와 IMU 데이터를 Tightly-coupled 방식으로 융합하는 고성능 LIO 시스템이다. 이를 통해 빠르고 정확한 실시간 궤적 추정이 가능하다.
- **백엔드 (Back-end):** 시간이 지남에 따라 누적되는 Odometry의 오차(drift)를 보정하기 위한 백엔드는 **SC-A-LOAM**의 루프 클로저 모듈을 활용한다.10 이 모듈은 'Scan Context'라는 강력한 장소 인식(place recognition) 기술자를 사용하여 로봇이 과거에 방문했던 위치를 전역적으로 탐지하고, 이를 기반으로 맵 전체의 일관성을 최적화한다.

Dynamic-LIO의 진정한 핵심 혁신은 이 두 강력한 구성 요소 사이에 위치한다. 바로 **'레이블 일관성 탐지(Label Consistency Detection)'** 기반의 동적 포인트 제거 모듈이다.11 이 모듈은 원시 LiDAR 스캔 데이터가 프론트엔드 Odometry로 입력되기 직전에 동적 객체에 해당하는 포인트들을 선별적으로 제거하는 필터 역할을 수행한다. 결과적으로 프론트엔드는 오염되지 않은, 즉 정적인 포인트만을 사용하여 위치 추정을 수행하게 되며, 이는 시스템 전체의 정확성과 강인성을 극대화하는 결정적인 역할을 한다.

### 1.2 B. 핵심 기술: 레이블 일관성 탐지 (Label Consistency Detection)

Dynamic-LIO의 가장 중요한 기여는 '레이블 일관성 탐지'라는 이름의 새롭고 매우 효율적인 동적 포인트 필터링 알고리즘에 있다. 이 기술의 철학은 기존 방법들과 근본적으로 다르다. 복잡한 기하학적 계산, 방대한 전역 통계 정보, 혹은 사전 학습된 시맨틱 정보를 사용하는 대신, 계산 비용이 매우 낮은 '질적(qualitative)'인 기준을 통해 동적 포인트를 식별한다.1

이 알고리즘은 도심 주행 시나리오에서 관찰되는 한 가지 강력한 경험적 사실에 기반한다: **"대부분의 동적 객체(차량, 보행자 등)는 지면 위에 위치한다"**는 것이다.11 이 단순하지만 효과적인 가정을 활용하여, 동적 객체 탐지라는 3차원 공간에서의 복잡한 문제를 1비트(1-bit) 레이블 정보의 불일치 문제로 치환한다. 이는 딥러닝 기반 시맨틱 분할이 제공하는 '차량', '보행자', '건물'과 같은 풍부한 정보를 포기하는 대신, LIO 시스템이 감당할 수 있는 '속도'와 '단순성'을 얻는 전략적 선택이다.

알고리즘은 다음과 같은 단계로 수행된다.

1. **바이너리 레이블링 (Binarized Labeling):** 먼저, 현재 입력된 LiDAR 스캔의 모든 포인트에 대해 '지면(ground)' 또는 '비-지면(non-ground)'이라는 이진 레이블을 부여한다. 이 과정은 포인트 클라우드를 2D 거리 이미지(range image)로 변환한 뒤, 빠른 2D 연결 요소 분석(connected components analysis)을 통해 효율적으로 수행된다. 이 단계는 동적 객체 탐지를 위한 사전 정보 구축 과정이다.1
2. **최근접 이웃 탐색 (Nearest Neighbor Search):** 다음으로, 현재 스캔의 각 '비-지면' 포인트에 대해, 이전에 스캔들로 구축된 전역 맵(global map)에서 최근접 이웃 포인트들을 찾는다. 전체 맵을 대상으로 한 고비용의 탐색을 피하기 위해, 포인트의 3D 좌표를 기반으로 해당 복셀(voxel) 주변을 탐색하는 효율적인 '복셀 위치 기반 탐색(voxel-location-based search)' 기법을 사용한다.11
3. **일관성 검사 및 동적 포인트 식별 (Consistency Check & Dynamic Point Identification):** 이 단계가 알고리즘의 핵심이다. 한 포인트가 동적이라고 판단하는 기준은 다음 두 가지 중 하나다 12:
   - **기준 1 (이웃 부재):** 맵에서 유의미한 최근접 이웃을 찾지 못한 포인트. 이는 정적 환경에는 존재하지 않던 새로운 구조물, 즉 동적 객체일 가능성이 매우 높다는 것을 의미한다. 정적인 벽이나 건물 위의 점이라면 과거 스캔에서도 비슷한 위치에 점이 존재해야 하기 때문이다.
   - **기준 2 (레이블 불일치):** 최근접 이웃을 찾았지만, 그들의 레이블이 현재 포인트의 레이블과 일치하지 않는 경우. 가장 대표적인 예시는 현재 스캔의 포인트가 '비-지면'(예: 차량의 지붕)인데, 맵에서 찾은 최근접 이웃 포인트들이 대부분 '지면'인 상황이다. 이 레이블의 불일치는 정적인 지면 위에 이질적인 '비-지면' 객체가 나타났음을 강력하게 시사하며, 이는 곧 동적 객체의 존재를 의미한다.1

이 두 가지 기준에 따라 동적으로 식별된 포인트들은 이후의 Odometry 계산이나 맵 업데이트 과정에서 완전히 배제된다.1 이 전체 과정은 스캔 당 평균 1~9ms라는 극히 낮은 계산 오버헤드로 완료될 수 있어, 시스템의 실시간 성능을 전혀 해치지 않는다.11 이처럼 '레이블 일관성 탐지'는 도메인 특화된 가정을 바탕으로 복잡한 문제를 단순화하여, 실시간성과 강인성이라는 두 마리 토끼를 모두 잡는 데 성공한 매우 실용적인 해결책이라 할 수 있다. 물론 이 접근법의 한계 또한 명확하다. 핵심 가정이 깨지는 환경, 예를 들어 지면과 무관하게 움직이는 드론이나 공중 장애물에는 취약할 수밖에 없다.

------

## 2. III. Dynamic-LIO의 구성 요소 심층 분석

Dynamic-LIO의 아키텍처는 단순히 새로운 알고리즘 하나를 추가한 것을 넘어, 각 분야에서 최고 수준으로 평가받는 오픈소스 컴포넌트들을 유기적으로 결합한 실용주의적 접근법의 정수를 보여준다. 이는 연구 개발의 효율성을 극대화하는 동시에, 각 컴포넌트가 가진 강점을 온전히 활용하는 현명한 전략이다. Dynamic-LIO의 성능을 제대로 이해하기 위해서는 그 근간을 이루는 프론트엔드(SR-LIO)와 백엔드(SC-A-LOAM)의 핵심 원리를 깊이 있게 살펴볼 필요가 있다.

### 2.1 A. 프론트엔드: SR-LIO와 스윕 재구성 (Sweep Reconstruction)

Dynamic-LIO의 Odometry 추정은 SR-LIO(Sweep Reconstruction-based LIO)라는 고성능 프레임워크에 기반한다.10 SR-LIO는 FAST-LIO 계열과 마찬가지로 반복 확장 칼만 필터(iEKF)를 핵심 엔진으로 사용하지만, '스윕 재구성'이라는 독창적인 기법을 통해 성능을 한 단계 끌어올렸다.15

- **스윕 재구성 (Sweep Reconstruction) 메커니즘:**
  - **동기:** 일반적인 회전식 3D LiDAR 센서는 1초에 10번 회전하며, 10Hz(즉, 100ms 간격)로 하나의 완성된 스캔(sweep)을 출력한다. iEKF 기반 LIO 시스템에서는 이 100ms 동안 수집된 IMU 측정값들을 적분하여 다음 스캔 시점의 상태를 예측한다. 문제는 이 100ms라는 시간이 로봇의 움직임을 예측하기에는 비교적 길다는 점이다. 이 시간 동안 IMU 예측 오차는 계속해서 누적되고, 이는 상태 추정의 정확도를 저하시키는 직접적인 원인이 된다.15
  - **프로세스:** '스윕 재구성'은 이 문제를 해결하기 위해 LiDAR 데이터의 출력 주파수를 인위적으로 높이는 기법이다. 회전식 LiDAR가 360도 스캔을 '연속적으로(continuously)' 수행한다는 점에 착안하여, 100ms 주기의 원본 스캔들을 더 작은 시간 단위의 세그먼트(segment)로 분할한다. 그리고 이 세그먼트들을 '멀티플렉싱(multiplexing)' 방식으로 재조합하여 더 높은 주파수의 새로운 '재구성된 스캔(reconstructed sweeps)'을 생성한다. 예를 들어, 시간 `[0ms, 100ms]`에 걸쳐 수집된 스캔 `S_j`와 `[100ms, 200ms]`에 걸쳐 수집된 스캔 `S_j+1`이 있다면, 이 둘의 데이터를 조합하여 `[50ms, 150ms]` 구간에 해당하는 새로운 스캔 `P_i`를 만들어내는 방식이다. 이 과정을 반복하면, 원본 10Hz 스캔으로부터 50ms 간격의 20Hz 재구성 스캔을 얻을 수 있다.15
  - **효과:** 스캔 주파수가 10Hz에서 20Hz로 두 배가 되면, iEKF의 상태 업데이트(state update) 간격이 100ms에서 50ms로 줄어든다. 이는 IMU 예측 오차의 누적 시간을 절반으로 단축시켜, 상태 추정의 정확도와 강인성을 크게 향상시키는 결과를 가져온다.15 Dynamic-LIO가 SR-LIO를 프론트엔드로 채택한 것은 바로 이 높은 정확도와 빠른 업데이트 주기의 이점을 얻기 위함이다.
- **SR-LIO++로의 발전:** SR-LIO는 주파수를 높이는 대가로 계산량이 선형적으로 증가하여 저전력 플랫폼에 적용하기 어렵다는 단점이 있었다. 후속 연구인 SR-LIO++는 인접한 재구성 스캔들이 상당 부분의 세그먼트를 공유한다는 점에 착안하여, 중복 계산을 피하기 위한 캐싱(caching) 메커니즘을 도입했다. 또한, 맵 포인트를 64비트 실수에서 8비트 정수로 양자화(quantization)하여 메모리 사용량과 계산 비용을 극적으로 줄였다. 그 결과, 라즈베리 파이 4B와 같은 저전력 임베디드 보드에서도 20Hz의 LIO 출력이 가능해졌다.17

### 2.2 B. 백엔드: SC-A-LOAM과 Scan Context

아무리 정밀한 프론트엔드 Odometry라도 장시간, 장거리를 이동하면 미세한 오차가 누적되어 전체 궤적이 실제와 어긋나는 드리프트(drift) 현상은 피할 수 없다. 이를 보정하기 위해, 이전에 방문했던 장소를 다시 인식하고(place recognition), 이를 강력한 제약 조건으로 사용하여 전체 궤적과 지도를 최적화하는 루프 클로저(loop closure) 백엔드가 필수적이다.2 FAST-LIO2와 같은 순수 Odometry 시스템은 이 기능이 없어 장거리 주행 시 오차 누적이 심각한 단점으로 지적된다.20 Dynamic-LIO는 이 문제를 해결하기 위해 SC-A-LOAM의 백엔드, 즉 Scan Context 기반의 루프 클로저를 채택했다.

- **Scan Context 개요:**
  - Scan Context는 하나의 3D LiDAR 스캔 전체를 대표하는 고유한 '지문'과 같은 전역 기술자(global descriptor)다.22
  - **생성 방식:** LiDAR 좌표계 중심에서 바라본 360도 주변 환경을 2차원 매트릭스로 표현한다. 이 매트릭스의 각 열(column)은 특정 방위각(azimuth)을, 각 행(row)은 특정 반경(radial) 거리를 나타낸다. 그리고 각 셀(bin)에는 해당 구역에 속한 포인트들 중 가장 높은 지점의 z-좌표(height) 값이 저장된다. 이렇게 생성된 2D 높이 맵이 바로 'Scan Context'이며, 이는 스캔의 3차원 구조 정보를 압축적으로 담고 있다.23
- **Scan Context의 장점 및 역할:**
  - **시점 불변성 (Viewpoint Invariance):** Scan Context의 가장 큰 장점은 회전 변화에 매우 강인하다는 것이다. 로봇의 방위각(yaw)이 변하면 Scan Context 매트릭스의 열이 원형으로 이동(circular shift)하는 형태로 나타난다. 따라서 두 Scan Context를 비교할 때, 하나의 열을 계속 이동시키며 가장 유사도가 높은 지점을 찾으면 원래의 방위각 차이를 계산할 수 있다. 이 덕분에 로봇이 왔던 길을 반대 방향으로 되돌아가는 경우에도 루프를 성공적으로 탐지할 수 있다.23
  - **빠른 탐색:** 과거 키프레임들에서 생성된 Scan Context들은 KD-Tree와 같은 효율적인 자료구조에 저장된다. 새로운 스캔이 들어오면, 그 Scan Context를 생성하여 KD-Tree에서 가장 유사한 후보들을 매우 빠르게 검색할 수 있다. 이는 실시간 루프 탐지를 가능하게 하는 핵심 요소다.22
  - **Dynamic-LIO에서의 역할:** Dynamic-LIO는 SC-A-LOAM의 이 Scan Context 모듈을 사용하여 현재 위치와 유사한 과거 위치(루프 클로저 후보)를 효율적으로 찾는다. 루프가 탐지되면, 두 키프레임 간의 상대적 변환 관계를 나타내는 팩터(factor)를 생성하고, 이를 GTSAM(Georgia Tech Smoothing and Mapping)이라는 팩터 그래프 최적화 라이브러리를 이용해 전체 궤적에 반영한다. 이 과정을 통해 누적된 오차를 효과적으로 제거하고 전역적으로 일관된 맵을 유지할 수 있다.10

결론적으로 Dynamic-LIO의 아키텍처는 각 분야에서 검증된 최고의 부품들을 영리하게 조합한 결과물이다. 프론트엔드로는 SR-LIO의 정확성과 속도를, 백엔드로는 Scan Context의 강인한 루프 클로저 능력을 가져왔다. 그리고 이 두 강력한 컴포넌트가 제대로 작동하기 위해 반드시 필요한 '깨끗한 데이터'를 공급하는 역할, 즉 동적 객체 필터링을 자신들의 핵심 기여분인 '레이블 일관성 탐지'가 담당한다. 이는 두 부모 프레임워크가 해결하지 못했던 특정 문제를 해결하는 '접착제' 역할을 수행하며 시스템 전체의 완성도를 높이는, 매우 효율적이고 실용적인 설계 철학을 보여준다.

------

## 3. IV. 성능 평가 및 비교 분석

어떤 LIO 시스템의 가치는 결국 실제 환경에서의 성능으로 증명된다. Dynamic-LIO는 실제 도심 주행 환경을 기록한 여러 공개 데이터셋에서의 엄격한 테스트를 통해 그 우수성을 입증했다. 특히, 다른 최신 LIO 시스템들과의 직접적인 비교는 Dynamic-LIO가 가진 독특한 강점과 그 위치를 명확히 보여준다.

### 3.1 A. 정량적 성능 평가 (Quantitative Performance Evaluation)

Dynamic-LIO의 성능은 주로 홍콩 도심의 복잡한 환경을 담은 ULHK-CA 데이터셋과 UrbanNav 데이터셋을 통해 검증되었다.1 이러한 평가에서 핵심적으로 사용되는 지표는 **절대 궤적 오차(Absolute Trajectory Error, ATE)**이다. ATE는 시스템이 추정한 전체 궤적과 GPS-RTK 등으로 측정한 실제 참값(Ground Truth) 궤적 간의 전역적인 차이를 나타내며, LIO 시스템의 종합적인 정확도를 평가하는 가장 표준적인 척도다. 보통 RMSE(Root Mean Square Error) 값으로 결과를 요약한다.26

- **ULHK 및 UrbanNav 데이터셋 결과:**
  - 관련 연구 논문1에 제시된 결과에 따르면, Dynamic-LIO는 LIO-SAM, A-LOAM, 그리고 동적 필터링 기능이 없는 원본 SR-LIO와 같은 기존의 최첨단(State-of-the-Art, SOTA) 시스템들과 비교했을 때 ATE 측면에서 일관되게 우수한 성능을 보였다.
  - 이러한 성능 향상은 특히 차량과 보행자와 같은 동적 객체가 밀집된 시퀀스(예: ULHK-CA-RE, Urban-Nav-HK-TST-2)에서 더욱 두드러졌다. LIO-SAM과 같은 시스템들이 동적 객체로 인해 발생하는 잘못된 제약 조건 때문에 궤적이 크게 벗어나는(drift) 구간에서도, Dynamic-LIO는 '레이블 일관성 탐지' 모듈 덕분에 안정적인 궤적을 유지하는 모습을 보여주었다. 이는 동적 객체 필터링이 실제 주행 환경의 정확도에 미치는 결정적인 영향을 명확히 증명하는 결과다.
- **계산 효율성:**
  - Dynamic-LIO의 핵심인 동적 포인트 탐지 및 제거 모듈의 성능은 단순히 정확도뿐만 아니라 계산 효율성 측면에서도 평가되었다. 실험 결과, 이 모듈은 LiDAR 스캔 한 장을 처리하는 데 평균적으로 1~9ms라는 매우 낮은 시간만을 소요하는 것으로 나타났다.11 이는 10Hz LiDAR의 한 스캔 처리 시간인 100ms에 비해 극히 일부에 불과하며, LIO 시스템의 전체적인 실시간성을 전혀 해치지 않음을 의미한다. 이 낮은 계산 오버헤드는 Dynamic-LIO가 다른 무거운 동적 객체 제거 기법들과 차별화되는 가장 큰 실용적 장점이다.

### 3.2 B. 주요 LIO 시스템과의 비교 분석

Dynamic-LIO의 설계 철학과 장단점을 보다 명확히 이해하기 위해서는, 현재 LIO 분야를 대표하는 다른 두 시스템, 즉 LIO-SAM과 FAST-LIO2와의 비교 분석이 필수적이다. 이 세 시스템은 SLAM 기술이 추구하는 주요 가치인 '정확도', '강인성', '속도'라는 삼각 트레이드오프에서 각기 다른 지향점을 대표한다.

아래 표는 세 시스템의 핵심적인 특징을 요약하여 비교한 것이다. 이 비교를 통해 각 시스템의 아키텍처 선택이 어떤 장단점으로 이어지는지 직관적으로 파악할 수 있다.

| **특징 (Feature)**     | **Dynamic-LIO**                                             | LIO-SAM 21                                                 |                                                             | FAST-LIO2 15 |      |
| ---------------------- | ----------------------------------------------------------- | ---------------------------------------------------------- | ----------------------------------------------------------- | ------------ | ---- |
| **코어 프레임워크**    | **iEKF 기반 프론트엔드** + 팩터 그래프 백엔드               | **팩터 그래프 (iSAM2)** 기반 최적화                        | **반복 확장 칼만 필터 (iEKF)**                              |              |      |
| **동적 객체 처리**     | **레이블 일관성 탐지** (경량, 실시간) 11                    | 기본적으로 없음. 외부 모듈(e.g., 시맨틱 분할) 추가 필요 2  | 기본적으로 없음. 동적 객체에 취약.                          |              |      |
| **백엔드/루프 클로저** | **Scan Context** 기반 전역 루프 클로저 10                   | **GPS 및 루프 클로저 팩터** (팩터 그래프에 통합)           | **없음** (Odometry 전용, 전역 최적화 부재) 20               |              |      |
| **맵 표현 및 매칭**    | 복셀 기반 맵, 포인트-평면 매칭 (SR-LIO 기반)                | 특징점(코너, 평면) 기반 Scan-to-Map                        | 원본 포인트(raw points) 직접 Scan-to-Map, **ikd-Tree** 사용 |              |      |
| **주요 강점**          | **동적 환경에서의 강인성**, 실시간성, 전역 일관성           | **전역 궤적 일관성**, 다중 센서(GPS) 융통성                | **압도적인 계산 효율성 및 속도**, 높은 출력 주파수          |              |      |
| **내재적 한계**        | '지면 위 객체' 가정에 의존, 복잡한 3D 동적 상황에 취약 가능 | 동적 객체에 대한 내장 처리 부재, 상대적으로 높은 계산 비용 | 장거리 주행 시 **심각한 누적 오차(drift)** 발생             |              |      |

이 비교표는 세 시스템의 근본적인 철학 차이를 드러낸다.

- **FAST-LIO2의 철학:** '속도'가 최우선이다. 이를 위해 매우 효율적인 칼만 필터(iEKF)와 데이터 구조(ikd-Tree)를 사용하고, 계산 비용이 많이 드는 전역 최적화(루프 클로저)를 과감히 포기했다.20 그 결과 타의 추종을 불허하는 속도를 얻었지만, 장거리 주행 시의 드리프트라는 명확한 대가를 치른다.
- **LIO-SAM의 철학:** '전역 일관성'이 최우선이다. 팩터 그래프(iSAM2)를 기반으로 하여 루프 클로저, GPS 등 다양한 종류의 제약 조건을 유연하게 통합할 수 있다.27 이 덕분에 장거리에서도 누적 오차를 효과적으로 보정하지만, 최적화 과정의 계산 비용이 상대적으로 높으며, 동적 객체 문제에 대한 내장된 해결책은 없다.
- **Dynamic-LIO의 철학 (종합):** Dynamic-LIO는 위 두 시스템의 장점을 결합하고 단점을 보완하려는 시도로 볼 수 있다. 즉, 어떻게 하면 필터 기반 프론트엔드의 '속도'와 그래프 기반 백엔드의 '전역 일관성'을 모두 가지면서, 동시에 '동적 강인성'까지 확보할 수 있을까? 라는 질문에 대한 답이다.
  1. **프론트엔드:** 속도와 정확도를 위해 필터 기반 접근법을 채택하되, 단순한 iEKF를 넘어선 진보된 프레임워크인 SR-LIO를 선택했다.10 이는 FAST-LIO2의 속도적 이점을 계승한다.
  2. **백엔드:** 전역 일관성을 위해 그래프 기반 루프 클로저를 채택하되, 강인한 장소 인식 성능이 검증된 Scan Context 기반의 SC-A-LOAM 백엔드를 선택했다.10 이는 LIO-SAM의 드리프트 보정 능력을 계승한다.
  3. **핵심 추가 요소:** 두 부모 철학 모두에서 약점이었던 동적 객체 문제를 해결하기 위해, 자체 개발한 경량 필터인 '레이블 일관성 탐지'를 시스템의 가장 앞단에 배치했다.11

결론적으로, Dynamic-LIO는 기존의 트레이드오프 스펙트럼 상의 한 점이 아니라, 각 시스템의 강점을 전략적으로 융합하여 새로운 균형점을 만들어내려는 하이브리드 시스템이다. 이는 빠르면서, 전역적으로 일관되고, 동적 환경에서도 강인한 LIO를 목표로 하는, 매우 진보적인 설계라 평가할 수 있다.

------

## 4. V. 결론 및 전망

### A. Dynamic-LIO의 기여 요약

Dynamic-LIO는 LiDAR-Inertial Odometry 분야, 특히 실제와 같은 동적 환경에서의 응용에 있어 몇 가지 중요한 기여를 했다.

첫째, Tightly-coupled LIO 프레임워크에 **계산적으로 매우 효율적인 동적 객체 제거 모듈**을 성공적으로 통합함으로써, 실제 도심 주행 환경에서 LIO 시스템의 위치 추정 정확도와 맵 품질을 획기적으로 개선했다.1 이는 동적 환경 강인성이라는 LIO 연구의 오랜 난제에 대한 실용적이고 효과적인 해법을 제시했다는 점에서 의의가 크다.

둘째, 핵심 기술인 **'레이블 일관성 탐지'**는 새로운 패러다임을 제시했다. 복잡한 딥러닝 기반의 시맨틱 정보나 무거운 기하학적 모델링에 의존하는 대신, '지면/비-지면'이라는 극도로 단순화된 바이너리 레이블의 불일치라는 질적 기준을 이용해 동적 객체를 효과적으로 필터링할 수 있음을 보여주었다.11 이는 LIO 시스템의 제한된 계산 예산 내에서 실시간으로 동적 객체를 처리할 수 있는 길을 열었다.

셋째, SR-LIO와 SC-A-LOAM이라는, 이미 성능이 검증된 최고 수준의 오픈소스 컴포넌트들을 영리하게 조합하여 **높은 성능과 개발 효율성을 동시에 달성**한 실용적인 시스템을 구현했다.10 이는 학술적 기여를 넘어, 커뮤니티가 쉽게 활용하고 발전시킬 수 있는 견고한 기반을 제공했다는 점에서도 가치가 높다.

### 4.1 B. 한계 및 미래 연구 방향

Dynamic-LIO의 성공과 한계는 모두 그 핵심 철학인 '단순화와 효율성'에서 비롯된다. 이 단순함이 실시간 성능을 가능하게 했지만, 동시에 그 핵심 가정이 깨지는 '코너 케이스(corner cases)'에 대한 내재적 취약점을 안고 있다. 이러한 한계는 다음과 같은 미래 연구 방향을 제시한다.

- **한계점:**
  - **강한 가정에 대한 의존성:** '레이블 일관성 탐지' 알고리즘은 '동적 객체는 지면 위에 있다'는 강한 가정에 기반한다. 따라서 이 가정이 성립하지 않는 환경, 예를 들어 터널 천장에 매달려 움직이는 유지보수 로봇, 교량 아래를 지나는 선박, 혹은 드론과 같이 3차원 공간에서 자유롭게 움직이는 동적 객체가 많은 환경에서는 필터링 성능이 크게 저하될 수 있다.
  - **악천후 및 센서 저하에 대한 취약성:** 비, 안개, 눈, 먼지와 같은 악천후 상황에서는 LiDAR 데이터의 품질이 심각하게 저하될 수 있다.6 이는 포인트 클라우드 기반의 지면 분할(ground segmentation) 알고리즘의 실패로 이어질 수 있으며, 부정확한 '지면/비-지면' 레이블링은 동적 객체 탐지 성능에 직접적인 악영향을 미친다.
- **미래 연구 방향:**
  - **다중 센서 융합 (Multi-modal Fusion):** Dynamic-LIO의 한계를 보완하기 위한 가장 유력한 방향은 상호 보완적인 특성을 가진 다른 센서와의 융합이다. 예를 들어, 연기, 안개, 악천후에 강인한 **4D 밀리미터파 레이더(4D mmWave Radar)**를 통합하는 것을 고려할 수 있다. 이러한 시스템은 LiDAR의 성능이 저하되는 환경을 스스로 감지하고, 시스템을 일시적으로 레이더-관성 오도메트리(RIO)로 전환하는 적응형 융합(adaptive fusion) 방식을 통해 전반적인 강인성을 확보할 수 있다.6 또한, 극심한 모션 블러에 강한 **이벤트 카메라(Event Camera)**와의 융합은 매우 동적인 움직임에 대한 대응 능력을 향상시킬 수 있다.30
  - **경량 시맨틱 정보의 결합:** 완전한 딥러닝 기반 세분화 모델의 무거운 계산 비용을 피하면서도, '레이블 일관성 탐지'의 가정을 보완할 수 있는 하이브리드 접근법이 유망하다. 예를 들어, 계산 비용이 낮은 경량화된 네트워크를 사용하여 포인트를 '움직일 수 있는 객체(potentially movable)'와 '완전한 정적 구조물(static structure)'과 같이 더 넓은 범주로 분류하고, 이를 레이블 일관성 검사의 사전 정보(prior)로 활용하는 방식이다.
  - **온라인 자가 학습 및 적응:** 시스템이 동작하는 동안 마주치는 동적 객체들의 패턴(예: 크기, 속도, 주로 나타나는 위치)을 온라인으로 학습하고, 이를 필터링 기준에 동적으로 반영하여 특정 환경에 대한 적응성을 높이는 연구도 가능하다.

궁극적으로, 미래의 강인한 SLAM 시스템은 단 하나의 완벽한 '만능(silver bullet)' 알고리즘이 아닐 가능성이 높다. 오히려 Dynamic-LIO의 빠르고 효율적인 휴리스틱을 기본으로 삼고, 이 휴리스틱의 가정이 위배되는 상황을 감지했을 때 다른 센서나 경량화된 학습 모델과 같은 '안전망(safety net)'이 작동하는, 보다 모듈화되고, 회복탄력적이며, 상황을 인지하는(context-aware) 아키텍처로 발전할 것이다. 이는 SLAM 분야의 큰 흐름이 '단일 알고리즘의 완벽성 추구'에서 '다양한 알고리즘의 협력적 융합을 통한 강인성 확보'로 전환되고 있음을 반영한다. Dynamic-LIO는 이러한 흐름 속에서 중요한 이정표를 제시한 시스템으로 평가받을 자격이 충분하다.

#### **참고 자료**

1. A Fast Dynamic Point Detection Method for LiDAR-Inertial Odometry in Driving Scenarios, accessed July 30, 2025, https://arxiv.org/html/2407.03590v1
2. LIO-CSI: LiDAR inertial odometry with loop closure combined with semantic information, accessed July 30, 2025, https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0261053
3. DY-LIO: Tightly-coupled LiDAR-Inertial Odometry for dynamic environments - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/381580695_DY-LIO_Tightly-coupled_LiDAR-Inertial_Odometry_for_dynamic_environments
4. A Review of Dynamic Object Filtering in SLAM Based on 3D LiDAR, accessed July 30, 2025, https://www.mdpi.com/1424-8220/24/2/645
5. LIO-SAM++: A Lidar-Inertial Semantic SLAM with Association Optimization and Keyframe Selection - PMC, accessed July 30, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11644182/
6. AF-RLIO: Adaptive Fusion of Radar-LiDAR-Inertial Information for Robust Odometry in Challenging Environments - arXiv, accessed July 30, 2025, https://arxiv.org/html/2507.18317v1
7. LiDAR Inertial Odometry Based on Indexed Point and Delayed Removal Strategy in Highly Dynamic Environments - MDPI, accessed July 30, 2025, https://www.mdpi.com/1424-8220/23/11/5188
8. LIO-SAM++: A Lidar-Inertial Semantic SLAM with Association Optimization and Keyframe Selection - MDPI, accessed July 30, 2025, https://www.mdpi.com/1424-8220/24/23/7546
9. KN-LIO: Geometric Kinematics and Neural Field Coupled LiDAR-Inertial Odometry - arXiv, accessed July 30, 2025, https://arxiv.org/html/2501.04263v1
10. ZikangYuan/dynamic_lio: [IROS 2025] A LiDAR-inertial ... - GitHub, accessed July 30, 2025, https://github.com/ZikangYuan/dynamic_lio
11. LiDAR-Inertial Odometry in Dynamic Driving Scenarios using Label Consistency Detection, accessed July 30, 2025, https://arxiv.org/html/2407.03590v2
12. LiDAR-Inertial Odometry in Dynamic Driving Scenarios using Label Consistency Detection, accessed July 30, 2025, https://arxiv.org/abs/2407.03590
13. [Literature Review] LiDAR-Inertial Odometry in Dynamic Driving, accessed July 30, 2025, https://www.themoonlight.io/review/lidar-inertial-odometry-in-dynamic-driving-scenarios-using-label-consistency-detection
14. A Fast Dynamic Point Detection Method for LiDAR-Inertial Odometry in Driving Scenarios, accessed July 30, 2025, https://paperswithcode.com/paper/a-fast-dynamic-point-detection-method-for
15. SR-LIO: LiDAR-Inertial Odometry with Sweep Reconstruction - arXiv, accessed July 30, 2025, https://arxiv.org/pdf/2210.10424
16. ZikangYuan/sr_lio: [IROS 2024] A LiDAR-inertial odometry (LIO) package that can adjust the execution frequency beyond the sweep frequency - GitHub, accessed July 30, 2025, https://github.com/ZikangYuan/sr_lio
17. [2503.22926] SR-LIO++: Efficient LiDAR-Inertial Odometry and Quantized Mapping with Sweep Reconstruction - arXiv, accessed July 30, 2025, https://www.arxiv.org/abs/2503.22926
18. SR-LIO++: Efficient LiDAR-Inertial Odometry and Quantized Mapping with Sweep Reconstruction | Request PDF - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/390354696_SR-LIO_Efficient_LiDAR-Inertial_Odometry_and_Quantized_Mapping_with_Sweep_Reconstruction
19. SR-LIO++: Efficient LiDAR-Inertial Odometry and Quantized Mapping with Sweep Reconstruction - arXiv, accessed July 30, 2025, https://arxiv.org/html/2503.22926v2
20. Visualization of trajectory comparison between A-LOAM, FAST-LIO, and... - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/figure/sualization-of-trajectory-comparison-between-A-LOAM-FAST-LIO-and-our-method-on-KITTI_fig5_378109927
21. LiDAR Inertial Odometry Based on Indexed Point and Delayed ..., accessed July 30, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10255994/
22. gisbi-kim/scancontext: backup of irapkaist/scancontext - GitHub, accessed July 30, 2025, https://github.com/gisbi-kim/scancontext
23. Scan Context: Egocentric Spatial Descriptor for Place ... - Giseop Kim, accessed July 30, 2025, https://gisbi-kim.github.io/publications/gkim-2018-iros.pdf
24. gisbi-kim/PyICP-SLAM: Full-python LiDAR SLAM using ICP and Scan Context - GitHub, accessed July 30, 2025, https://github.com/gisbi-kim/PyICP-SLAM
25. gisbi-kim/SC-LIO-SAM: LiDAR-inertial SLAM - GitHub, accessed July 30, 2025, https://github.com/gisbi-kim/SC-LIO-SAM
26. [1811.09895] Benchmarking and Comparing Popular Visual SLAM Algorithms - ar5iv - arXiv, accessed July 30, 2025, https://ar5iv.labs.arxiv.org/html/1811.09895
27. LIO-SAM + Octomap + Occupancy Grid in Dynamic Environments - GitHub, accessed July 30, 2025, https://github.com/mohdomama/LIO-SAM
28. Performance Analysis of LIO-SAM - Ronia Peterson, accessed July 30, 2025, http://www.roniapeterson.com/LIO-SAM.pdf
29. [2107.06829] FAST-LIO2: Fast Direct LiDAR-inertial Odometry - arXiv, accessed July 30, 2025, https://arxiv.org/abs/2107.06829
30. Strong Robust LIO System Based on Event Camera Assistance, accessed July 30, 2025, https://isprs-annals.copernicus.org/articles/X-4-2024/477/2024/isprs-annals-X-4-2024-477-2024.pdf

