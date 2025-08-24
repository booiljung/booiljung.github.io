[SLAM 동향](./index.md)
# 최신 SLAM 기술 동향 분석 보고서


SLAM(Simultaneous Localization and Mapping)은 로봇 공학 및 컴퓨터 비전 분야의 근본적인 문제로, 자율 시스템이 사전 정보가 없는 미지의 환경에서 자신의 위치를 실시간으로 추정(Localization)함과 동시에 주변 환경의 지도를 작성(Mapping)하는 기술을 의미한다.1 이 문제는 본질적으로 '닭과 달걀'의 관계로, 정확한 지도가 있어야 정확한 위치를 알 수 있고, 정확한 위치를 알아야 일관성 있는 지도를 만들 수 있다. 초기 SLAM은 확률적 접근법(probabilistic tools)에 기반하여 센서 측정값의 노이즈와 불확실성을 다루는 방식으로 발전해왔다.3

최근 수년간 SLAM 기술은 인공지능, 특히 딥러닝의 발전과 맞물려 폭발적인 진화를 거듭하고 있다. 이 보고서는 최신 SLAM 기술의 동향을 네 가지 핵심적인 진화 방향을 축으로 분석한다.

첫째, **정밀도의 정점**이다. 전통적인 기하학 기반 SLAM은 수십 년간 축적된 최적화 이론을 바탕으로 인간이 설계한 특징점(feature)과 기하학적 제약을 활용하여 정확도를 극한까지 끌어올렸다. 이 흐름의 정점에 있는 기술들을 통해 현대 SLAM의 성능 기준선을 이해한다.

둘째, **강인함의 추구**이다. 현실 세계는 실험실과 달리 조명 변화, 빠른 움직임, 텍스처 부족 등 예측 불가능한 변수로 가득하다. 딥러닝 기반의 학습 능력과 LiDAR, IMU 등 다양한 센서를 긴밀하게 융합(multi-sensor fusion)하는 기술이 어떻게 이러한 불확실성을 극복하고 SLAM의 강인함(robustness)을 새로운 차원으로 끌어올렸는지 탐구한다.4

셋째, **표현의 혁명**이다. SLAM의 핵심 구성 요소인 '맵'을 표현하는 방식 자체가 근본적으로 바뀌고 있다. 과거의 점, 선, 면과 같은 명시적(explicit) 표현이나 복셀(voxel) 기반의 이산적(discrete) 표현에서 벗어나, 3D 장면 전체를 하나의 연속적인 함수, 즉 신경장(Neural Fields)으로 표현하려는 시도가 새로운 패러다임을 열고 있다.5 이는 단순히 맵을 압축하는 것을 넘어, 렌더링을 통해 SLAM 문제 자체를 재정의하는 혁신을 가져왔다.

마지막으로, **이해의 심화**이다. 로봇이 단순히 공간의 기하학적 구조를 파악하는 것을 넘어, 그 공간에 존재하는 객체들의 의미(semantics)를 이해하는 것은 진정한 자율 지능의 필수 조건이다. 시맨틱 SLAM은 기하학적 지도에 '의자', '책상'과 같은 의미론적 정보를 통합하여, 로봇이 환경과 보다 지능적으로 상호작용할 수 있는 토대를 마련하고 있다.5

본 보고서는 이러한 네 가지 흐름을 따라, 각 분야를 대표하는 최신 SLAM 기술들을 발표 연도, 기술적 참고 관계, 핵심 특징, 장단점, 그리고 해결하고자 하는 문제의 관점에서 심층적으로 분석하여, 현재 SLAM 기술의 최전선(state-of-the-art)과 미래 발전 방향에 대한 통찰을 제공하고자 한다.


현대 SLAM 연구의 지형을 이해하기 위해서는, 서로 다른 철학을 바탕으로 각자의 분야에서 정점에 도달한 두 개의 상징적인 시스템을 먼저 분석해야 한다. 하나는 수십 년간 발전해 온 전통적인 최적화 방식의 완성형이며, 다른 하나는 딥러닝을 통해 SLAM의 가능성을 재정의한 혁신적인 시스템이다. 이 두 기술은 이후 등장하는 수많은 연구들의 성능을 비교하고 평가하는 중요한 기준점(baseline) 역할을 한다.


ORB-SLAM3는 전통적인 특징점(feature-based) 기반 SLAM 기술의 집대성이라 할 수 있는 시스템이다. 이는 단순히 하나의 알고리즘이 아니라, 단안(monocular), 스테레오(stereo), RGB-D 카메라를 모두 지원하며, 관성 측정 장치(IMU)와의 융합은 물론, 핀홀(pin-hole) 및 어안(fisheye) 렌즈 모델까지 포괄하는 최초의 통합 SLAM 라이브러리라는 점에서 그 의의가 크다.8 이 시스템은 이전 버전인 ORB-SLAM, ORB-SLAM2, 그리고 ORB-SLAM-VI의 계보를 잇는 최종 진화형으로, 과거의 성공적인 요소들을 계승하고 한계점들을 개선했다.9


ORB-SLAM3의 혁신은 기존의 검증된 기술들을 어떻게 계승하고 발전시켰는지 살펴보면 명확해진다.

- **DBoW2 (Bags of Binary Words)**: 장소 인식(Place Recognition)은 SLAM에서 누적 오차를 초기화하고 루프를 닫는(loop closure) 핵심 기능이다. ORB-SLAM3는 이전 버전들과 마찬가지로 DBoW2 라이브러리를 사용하지만, 그 방식을 근본적으로 개선했다.9 기존 DBoW2는 정밀도(precision)를 높이기 위해 3개의 연속된 키프레임이 동일한 장소로 인식되어야 하는 시간적 일관성(temporal consistency)을 요구했다. 이는 재현율(recall)을 희생시켜 루프를 너무 늦게 감지하는 문제를 낳았다. ORB-SLAM3는 이 순서를 뒤집어, 후보 키프레임에 대해 기하학적 일관성을 먼저 검사한 후, 이미 맵에 존재하는 공통 관측 키프레임(covisible keyframes)과의 지역적 일관성을 확인한다. 이 전략은 재현율을 극적으로 높여 더 빠르고 안정적인 루프 클로저를 가능하게 한다.12
- **ORB-SLAM-VI**: Visual-Inertial(VI) SLAM은 카메라만으로는 해결하기 어려운 빠른 움직임이나 스케일 모호성 문제를 IMU를 통해 해결한다. ORB-SLAM3는 이전 VI 버전의 아이디어를 계승하되, 최적화 방식을 완전히 새롭게 설계했다. 시스템의 모든 단계, 심지어 IMU 초기화 단계부터 **최대 사후 확률(Maximum-a-Posteriori, MAP) 추정**에만 전적으로 의존한다.8 이는 모든 측정값의 불확실성을 확률적으로 가장 타당한 단일 최적화 문제로 풀어내는 방식으로, 시스템의 정확도와 강인성을 이전과는 비교할 수 없는 수준으로 끌어올렸다.


- **MAP 기반 Visual-Inertial SLAM**: ORB-SLAM3의 가장 큰 혁신 중 하나는 VI-SLAM을 MAP 추정 프레임워크로 완전히 통합한 것이다. 특히 IMU 초기화 과정은 매우 중요한데, 여기서 속도, 중력 방향, IMU 바이어스가 정확하게 추정되어야 전체 시스템의 안정성이 보장된다. ORB-SLAM3는 단 2초 만에 정확한 초기값을 계산해내며, 이는 기존 시스템들이 수십 초를 소요하던 것과 비교해 매우 빠르고 정확하다.8 이 강력한 초기화와 지속적인 MAP 최적화 덕분에, ORB-SLAM3는 기존 VI-SLAM 시스템 대비 2배에서 10배 더 높은 정확도를 달성한다.8
- **Atlas 다중 맵 시스템**: 장기적인 자율주행이나 증강현실(AR) 시나리오에서는 로봇이 이전에 방문했던 장소를 다시 가거나, 시각 정보가 완전히 소실되었다가 다시 나타나는 경우가 빈번하다. 기존 SLAM 시스템은 이런 상황에서 트래킹을 완전히 놓치고(lost) 처음부터 다시 시작해야 했다. ORB-SLAM3는 **Atlas**라는 다중 맵 시스템을 도입하여 이 문제를 해결한다.8 트래킹을 놓치면, 시스템은 즉시 새로운 맵 생성을 시작한다. 이후 이전에 매핑했던 영역으로 돌아와 장소 인식이 성공하면, 두 맵을 끊김 없이(seamlessly) 하나의 일관된 맵으로 병합한다. 이는 단기적인 정보만 사용하는 Visual Odometry(VO) 시스템과 달리, 과거의 모든 정보를 재사용하여 장기적인 일관성과 정확성을 보장하는 진정한 의미의 SLAM을 구현한 것이다.8


ORB-SLAM3의 가장 큰 장점은 수많은 실험을 통해 검증된 **강인성**과 현존하는 시스템 중 최고 수준의 **정확도**이다.8 또한 다양한 센서 구성을 지원하는 범용성과 잘 관리된 오픈소스 코드 덕분에 학계와 산업계에서 표준적인 SLAM 시스템으로 널리 사용되고 있다.9

하지만 명확한 한계도 존재한다. 특징점 기반 방식의 태생적 한계로 인해, 벽이나 복도처럼 **텍스처가 부족한 환경**에서는 특징점을 충분히 추출하지 못해 트래킹에 실패할 수 있다.12 또한, 자동차 주행처럼 순수한 회전이 거의 없거나 매우 느리게 움직이는 시나리오에서는 IMU 센서가 충분한 관측 정보를 얻지 못해 초기화에 어려움을 겪을 수 있다.12 동적 객체(움직이는 사람, 차량 등)에 대한 명시적인 처리 기능이 부족하여, 이러한 객체들을 정적인 배경의 일부로 오인해 오차를 유발할 수도 있다.


ORB-SLAM3가 고전적인 최적화 방식의 정점을 보여줬다면, 같은 해에 발표된 DROID-SLAM은 SLAM 문제에 대한 완전히 다른 접근법, 즉 End-to-End 딥러닝의 가능성을 제시하며 학계에 큰 충격을 주었다. DROID-SLAM은 특징 추출, 매칭, 최적화 등으로 나뉘어 있던 복잡한 SLAM 파이프라인을, 미분 가능한(differentiable) 단일 심층 신경망으로 대체하려는 시도의 결정체다.13 핵심은 미분 가능한 **'Dense Bundle Adjustment (DBA) Layer'**를 통해 카메라 포즈와 영상의 모든 픽셀에 대한 깊이(depth) 값을 반복적으로 함께 업데이트하는 것이다.15


DROID-SLAM의 독창성은 기존 딥러닝 아키텍처와 고전 기하학 원리를 영리하게 융합한 데 있다.

- **RAFT (Recurrent All-Pairs Field Transforms)**: DROID-SLAM의 핵심 구성 요소 중 하나는 두 이미지 간의 조밀한(dense) 대응점, 즉 옵티컬 플로우를 찾는 것이다. 이를 위해 옵티컬 플로우 추정 분야에서 SOTA를 달성한 RAFT의 구조를 차용했다.17 RAFT는 순환 신경망(recurrent neural network)을 통해 초기 추정값으로부터 점진적으로 오차를 수정해나가는 반복적 업데이트(iterative updates) 구조를 가지고 있다. DROID-SLAM은 이 아이디어를 SLAM에 적용하여, 프레임 간의 조밀한 움직임을 매우 정확하게 추정한다.
- **BA-Net**: DROID-SLAM 이전에 미분 가능한 번들 조정(Bundle Adjustment) 레이어를 제안한 BA-Net과 같은 연구가 있었다.14 하지만 둘 사이에는 결정적인 차이가 있다. BA-Net은 미리 예측된 몇 개의 고정된 깊이 기저(depth basis)의 선형 조합으로 3D 구조를 표현했다. 이는 표현력에 명백한 한계를 가졌다. 반면, DROID-SLAM은 영상의 **모든 픽셀에 대한 깊이 값을 직접 최적화**한다. 이는 훨씬 더 유연하고 정확한 3D 재구성을 가능하게 하며, DROID-SLAM의 압도적인 성능의 기반이 된다.14


- **Dense Bundle Adjustment (DBA) Layer**: 이 레이어는 DROID-SLAM의 심장부다. 고전적인 BA는 3D 포인트의 재투영 오차(reprojection error)를 최소화하여 카메라 포즈와 3D 구조를 최적화하는 비선형 최소제곱 문제다. DROID-SLAM은 이를 미분 가능한 연산으로 구현했다. 구체적으로, 현재 추정된 옵티컬 플로우(프레임 간 픽셀의 움직임)와 현재 추정된 포즈 및 깊이로부터 계산된 픽셀의 움직임이 얼마나 일치하는지, 즉 호환성(compatibility)을 측정한다. 그리고 이 불일치를 줄이는 방향으로 카메라 포즈와 픽셀 단위 깊이에 대한 가우스-뉴턴(Gauss-Newton) 업데이트를 계산하여 적용한다.14 이처럼 강력한 기하학적 제약이 네트워크 내부에 통합되어 있기 때문에, DROID-SLAM은 놀랍게도 

  **단안(monocular) 비디오로만 학습했음에도 불구하고, 별도의 재학습 없이 스테레오나 RGB-D 입력을 자연스럽게 활용**하여 성능을 더욱 향상시킬 수 있다.14

- **강력한 일반화 성능**: 많은 딥러닝 기반 SLAM 시스템들이 학습 데이터셋에는 좋은 성능을 보이지만 새로운 환경에서는 성능이 급격히 저하되는 과적합(overfitting) 문제를 겪는다. DROID-SLAM은 이러한 한계를 극복하고 놀라운 일반화 성능을 보여준다. TartanAir, EuRoC, TUM-RGBD 등 다양한 표준 벤치마크 데이터셋에서 기존의 고전적인 방법(ORB-SLAM3 포함)과 학습 기반 방법을 모두 압도적인 차이로 능가했다.14 이는 특정 환경의 특징을 암기하는 것이 아니라, 움직임과 3D 구조 사이의 근본적인 기하학적 관계를 학습했음을 시사한다.


DROID-SLAM의 가장 큰 장점은 기존의 어떤 시스템과도 비교하기 힘든 **압도적인 정확도**와 **강인성**(catastrophic failures, 즉 치명적인 추적 실패가 현저히 적음)이다.14 또한 수많은 하이퍼파라미터 튜닝과 엔지니어링 트릭이 필요했던 기존 시스템들과 달리, 상대적으로 설계가 간결하다는 장점이 있다.17

하지만 이러한 성능은 상당한 대가를 치른다. 가장 큰 단점은 **높은 계산 비용**이다. 실시간에 가깝게 동작하기 위해서는 최소 11GB 이상의 메모리를 가진 고사양 GPU가 필수적이며, 학습에는 24GB 이상의 메모리를 가진 GPU 여러 장이 필요하다.19 특히 수십, 수백 개의 키프레임에 대한 포즈와 조밀한 깊이 맵을 동시에 최적화하는 전역 BA(Global BA)는 엄청난 계산량을 요구한다.17 이는 저전력 임베디드 시스템이나 모바일 기기에서의 활용을 어렵게 만드는 주요한 제약 조건이다.


ORB-SLAM3와 DROID-SLAM은 SLAM 문제에 대한 두 가지 상반된 철학적 접근 방식의 현재 위치를 명확히 보여준다. ORB-SLAM3는 '모델 기반(model-based)' 접근법의 정점으로, 수십 년간 컴퓨터 비전 연구자들이 쌓아 올린 기하학적 원리와 최적화 이론의 집대성이다. 이 시스템의 작동 원리는 명확하게 해석 가능하며, 그 성능은 수학적으로 보장된다. 반면, DROID-SLAM은 '데이터 기반(data-driven)' 접근법의 새로운 시대를 열었다. 복잡하고 단계적인 파이프라인을, 데이터로부터 직접 해법을 배우는 단일 학습 문제로 치환하려는 야심 찬 시도다.

이 두 시스템의 관계를 더 깊이 파고들면 흥미로운 지점이 발견된다. 전통적 SLAM, 즉 ORB-SLAM3와 같은 시스템이 실패하는 지점은 주로 '까다로운' 환경이다. 텍스처가 없거나, 조명이 급격히 변하거나, 움직임이 매우 빨라 모션 블러가 심한 경우, 인간이 설계한 ORB 특징점은 안정적으로 추출되고 매칭되기 어렵다.20 딥러닝은 바로 이 지점에서 해법을 제시한다. DROID-SLAM의 RAFT 기반 구조는 데이터로부터 대응점(correspondence)을 학습하기 때문에, 사람이 정의한 특징점보다 훨씬 더 강인하고 조밀한(dense) 매칭 정보를 제공할 수 있다.

하지만 DROID-SLAM 역시 기하학적 원리를 완전히 버린 것은 아니다. 그 이름에 'Dense Bundle Adjustment'가 포함된 것에서 알 수 있듯, 고전 기하학의 핵심 원리인 번들 조정을 미분 가능한 레이어 형태로 네트워크에 통합했다. 이는 순수 딥러닝이 가질 수 있는 '블랙박스' 문제를 완화하고, 강력한 기하학적 제약을 통해 학습 과정에 안정성을 부여하며, 결과적으로 뛰어난 일반화 성능을 확보하려는 영리한 전략이다.14

결론적으로, 이 두 시스템의 등장은 미래 SLAM의 방향이 순수한 모델 기반도, 순수한 데이터 기반도 아닌, 두 가지의 장점을 결합한 **'하이브리드(hybrid)'** 형태가 될 것임을 강력하게 시사한다. DROID-SLAM은 그 성공적인 초기 사례이며, ORB-SLAM3는 앞으로 등장할 모든 새로운 접근법들이 반드시 넘어서야 할, 매우 강력하고 신뢰할 수 있는 **'기하학적 기준선(geometric baseline)'**으로 확고히 자리매김했다.


Visual SLAM은 저렴한 카메라 센서를 사용하여 풍부한 환경 정보를 얻을 수 있다는 장점이 있지만, 조명 변화에 민감하고 스케일 모호성(scale ambiguity) 문제를 가지는 등 명확한 한계가 존재한다.21 이러한 한계를 극복하기 위해, 조명 변화에 강인하고 정확한 3차원 거리 측정이 가능한 LiDAR 센서의 활용이 필수적으로 여겨지고 있다. 이 장에서는 LiDAR를 중심으로 IMU, 카메라 등 여러 센서의 정보를 긴밀하게 결합(tightly-coupled)하여, 어떤 환경에서도 강인하고 정밀한 성능을 발휘하는 최신 다중 센서 융합 SLAM 기술들을 탐구한다.


카메라와 LiDAR는 상호보완적인 특성을 가진다. 카메라는 저렴하고 조밀한 텍스처 정보를 제공하지만 조명과 스케일에 취약하다. 반면 LiDAR는 주변광의 영향을 받지 않고 정확한 3D 구조 정보를 제공하지만, 데이터가 상대적으로 희소(sparse)하고 가격이 비싸다.21 여기에 고주파수(high-frequency)의 관성 정보를 제공하여 빠른 움직임을 포착하는 IMU를 융합하는 것이 현대 SLAM의 핵심 과제 중 하나가 되었다.4

센서 융합 기술은 초기에 각 센서의 추정 결과를 나중에 합치는 느슨한 결합(loosely-coupled) 방식에서 시작하여, 모든 센서의 원시 측정값(raw measurements)을 하나의 통합된 최적화 문제로 풀어내는 긴밀한 결합(tightly-coupled) 방식으로 발전해왔다.3 이 흐름에서 중요한 이정표가 된 기술들이 있다. **LIO-SAM (2020)**은 평활화 및 매핑(smoothing and mapping) 기법을 사용하는 그래프 최적화 기반의 LIO(LiDAR-Inertial Odometry) 시스템으로 널리 사용되었다.24 이후 **FAST-LIO2 (2022)**는 패러다임을 한 단계 더 발전시켰다. 이 시스템은 계산 비용이 많이 드는 특징 추출(feature extraction) 과정을 생략하고, LiDAR의 원시 포인트(raw points)를 맵에 직접 정합(register)하는 'Direct' 방식을 채택했다. 또한, 복잡한 그래프 최적화 대신 매우 효율적인 반복 칼만 필터(Error-State Iterated Kalman Filter, ESIKF)를 사용하여 실시간 성능과 정확도를 동시에 달성했다.25 이러한 기술적 진화의 최전선에 바로 FAST-LIVO2가 있다.


FAST-LIVO2는 FAST-LIO2의 성공적인 프레임워크를 기반으로, 여기에 Visual Odometry(VO)를 직접적(Direct)이고 긴밀하게(Tightly-coupled) 융합한 최신 LIVO(LiDAR-Inertial-Visual Odometry) 시스템이다.27 이 시스템은 LiDAR, IMU, 카메라로부터 들어오는 이종(heterogeneous)의 측정값들을 ESIKF 내에서 순차적으로 업데이트하는 방식으로 효율적이면서도 강인하게 융합한다.29


- **FAST-LIO2**: FAST-LIVO2는 FAST-LIO2의 핵심 철학을 그대로 계승한다. 특징 추출 없이 원시 포인트를 사용하는 'Direct' LiDAR Odometry 방식과, 증분 k-d 트리(ikd-Tree)를 이용한 효율적인 맵 관리 기법이 그것이다.25 이를 통해 LiDAR와 IMU 융합 부분의 높은 효율성과 정확도를 보장받는다.
- **FAST-LIVO**: 이전 버전인 FAST-LIVO는 비동기식(asynchronous) 업데이트 방식을 사용했는데, 이는 특정 상황에서 시스템의 강인성을 저해하는 요인이었다. FAST-LIVO2는 이 문제를 해결하기 위해, 차원과 특성이 다른 LiDAR와 Visual 측정값을 칼만 필터 내에서 **'순차적 업데이트(sequential update)'** 전략으로 처리한다. 이는 이종 센서 데이터 간의 차원 불일치(dimension mismatch) 문제를 해결하고 시스템의 안정성을 높이는 핵심적인 개선점이다.30


- **"Fast" & "Direct"**: 이름에서 알 수 있듯이, 'Fast'는 ESIKF를 기반으로 한 매우 효율적인 계산 성능을 의미한다. 'Direct'는 SLAM 파이프라인에서 전통적으로 사용되던 특징점(e.g., ORB, FAST corner)이나 특징면(e.g., edge, plane) 추출 단계를 완전히 생략하는 방식을 말한다.29 대신, LiDAR의 원시 포인트 클라우드는 맵에 직접 정합되고, 카메라 이미지는 픽셀의 **광도 오차(photometric error)**를 직접 최소화하는 방식으로 정렬된다. 특징 추출이라는 병목 구간을 제거함으로써 계산 효율을 극대화하고, 인간이 설계한 특징이 놓칠 수 있는 환경의 미묘한 기하학적, 시각적 정보까지 모두 활용하여 정확도를 높이는 것이 이 방식의 핵심 철학이다.26

- **Unified Voxel Map**: 이종 센서인 LiDAR와 Visual 정보를 어떻게 하나의 일관된 표현으로 엮어내는가는 LIVO 시스템의 핵심 과제다. FAST-LIVO2는 이를 **'통합 복셀 맵(Unified Voxel Map)'**이라는 영리한 구조로 해결한다.29 이 맵에서 LiDAR 모듈은 새로운 LiDAR 스캔을 등록하기 위한 3D 기하학적 구조(복셀 그리드)를 구축한다. 그러면 Visual 모듈은 이 맵을 구성하는 LiDAR 포인트들에, 해당 포인트를 관측했던 이미지의 작은 조각, 즉 **이미지 패치(patch)**를 붙인다. 이렇게 이미지 패치가 붙은 LiDAR 포인트를 'visual map points'라고 부른다. 새로운 이미지가 입력되면, 시스템은 이 'visual map points'를 현재 이미지에 투영하고, 투영된 위치의 픽셀 값과 저장된 이미지 패치의 픽셀 값 차이(광도 오차)를 최소화하는 방향으로 카메라의 포즈를 정밀하게 추정한다.

- **정확도 향상 기법**: FAST-LIVO2는 정확도를 더욱 높이기 위해 여러 정교한 기법들을 사용한다. 이미지 정렬 과정에서, 단순히 픽셀 값만 비교하는 것이 아니라 해당 픽셀 주변의 LiDAR 포인트들로부터 **평면 정보(plane prior)**를 추정하여 활용한다. 이는 3D 구조를 고려한 더 정확한 정렬을 가능하게 한다.30 또한, 가장 품질이 좋고 유용한 정보(e.g., 시차가 크고 텍스처가 풍부한)를 담고 있는 이미지 패치를 

  **레퍼런스 패치로 동적으로 업데이트**하며, 주변 환경의 조명 변화에 강인하게 대처하기 위해 실시간으로 카메라의 **노출 시간(exposure time)을 추정**하는 기능까지 포함하고 있다.30


FAST-LIVO2의 장점은 명확하다. 정확도, 강인성, 계산 효율성 모든 측면에서 기존의 SOTA 시스템들을 상당한 차이로 능가한다.30 특히 계산 효율이 매우 높아, 소형 무인 항공기(UAV)에 탑재하여 실시간 항법(onboard navigation)을 수행할 수 있을 정도다.31

하지만 이러한 고성능 시스템은 **하드웨어 동기화**에 매우 민감하다. LiDAR, 카메라, IMU 센서 간의 시간 정보가 물리적으로 정확하게 동기화되지 않으면, 센서 융합 과정에서 큰 오차가 발생하여 시스템 성능이 급격히 저하될 수 있다.31


대부분의 다중 센서 융합 연구가 LiDAR, 카메라, IMU의 조합에 집중하는 동안, RLI-SLAM은 새로운 센서인 **UWB(Ultra-Wideband)**를 도입하여 새로운 가능성을 제시했다.32 RLI-SLAM은 UWB의 거리 측정(ranging) 정보, LiDAR, IMU를 긴밀하게 결합한 SLAM 프레임워크다.32


- **드리프트 억제**: IMU는 단기적으로는 정확한 모션 정보를 제공하지만, 시간이 지남에 따라 오차가 누적되는 드리프트(drift) 문제가 심각하다. RLI-SLAM은 UWB 센서를 사용하여 앵커(anchor)까지의 절대 거리를 측정하고, 이 정보를 활용하여 IMU의 장기적인 드리프트를 효과적으로 억제한다. 이는 특히 장시간 동안 넓은 영역을 빠르게 움직이는 시나리오에서 LiDAR 포인트 클라우드의 왜곡을 보정하고, 전역적인 위치 추정 정확도를 높이는 데 결정적인 역할을 한다.32
- **혁신점**: 이 기술의 핵심 혁신점은 1) UWB 거리 측정 정보와 관성 센서 정보를 긴밀하게 융합하여 상호 보완하는 메커니즘과, 2) 이 융합 프레임워크에 효율적인 루프 클로저 모듈을 통합하여 전역적인 맵 일관성을 확보한 것이다.32


RLI-SLAM은 단 하나의 UWB 앵커로부터 거리 측정 정보만 받아도 높은 정확도와 강인성을 유지할 수 있다. 또한, 계산 복잡도가 FAST-LIO2와 유사한 수준으로 매우 낮아 실시간 적용에 유리하다.32


SLAM의 강인성은 결국 **'융합의 깊이'**에 비례한다는 것을 이 장의 기술들은 명확히 보여준다. 여러 센서를 단순히 병렬로 사용하는 것을 넘어, 각 센서의 원시 측정값이 다른 센서의 약점을 실시간으로, 그리고 근본적으로 어떻게 보완해주는지가 시스템 전체의 성능을 결정한다. FAST-LIVO2는 이러한 융합 철학의 정점에 서 있는 시스템이다.

이 기술들의 발전 과정을 따라가 보면, 융합의 패러다임이 어떻게 진화했는지 알 수 있다. LIO(LiDAR-Inertial Odometry)는 왜 강력한가? LiDAR의 정확한 3D 거리 측정과 IMU의 고주파 모션 추정이 결합되어, 시각 정보가 전혀 없는 어두운 환경이나 텍스처가 없는 환경에서도 강인한 위치 추정이 가능하기 때문이다. 그렇다면 LIVO(LiDAR-Inertial-Visual Odometry)는 왜 더 강력한가? LIO만으로는 표지판이나 벽의 그림과 같이 텍스처가 풍부하지만 기하학적으로는 단순한 평면의 정보를 충분히 활용할 수 없다. 여기에 카메라를 더하면, LiDAR가 3D 공간의 기하학적 뼈대(e.g., plane prior)를 제공하고, 카메라가 그 위에 색과 텍스처라는 풍부한 정보(e.g., photometric error)를 입혀, 훨씬 더 정교하고 강인한 위치 추정이 가능해진다.

FAST-LIVO2의 **'Unified Voxel Map'**은 이러한 융합 과정을 물리적으로 구현한 결정체다. 이 맵에서 LiDAR 포인트는 더 이상 단순한 3D 좌표의 나열이 아니다. 각 포인트는 자신을 관측했던 이미지의 조각(patch)을 '들고 있는' 앵커(anchor)가 된다. 이는 물리적으로, 그리고 정보적으로 완전히 다른 두 이종 센서의 데이터를 하나의 통합된 최적화 문제로 엮어주는 핵심적인 다리 역할을 한다.30

결론적으로, 최신 센서 퓨전 SLAM은 '어떤 센서를 추가로 사용하는가'의 문제를 넘어, **'여러 센서의 이질적인 데이터를 어떻게 하나의 일관된 표현(representation)으로 엮어내는가'**의 문제로 진화했다. FAST-LIVO2의 Unified Voxel Map은 이러한 '표현의 융합'을 보여주는 대표적인 성공 사례이며, RLI-SLAM의 UWB 도입은 미래에 Radar, Event Camera 등 더욱 다양한 센서들이 이 융합 프레임워크에 통합될 수 있는 무한한 가능성을 시사한다.


SLAM 기술의 역사에서 가장 혁신적인 변화는 종종 '맵(Map)'을 표현하는 방식 자체를 근본적으로 바꾸는 것에서 시작되었다. 최근 몇 년간, 3D 장면을 점, 선, 면의 집합이 아닌 하나의 연속적인 함수, 즉 **신경장(Neural Fields)**으로 표현하려는 시도가 SLAM 분야에 거대한 패러다임 전환을 가져오고 있다. 이 장에서는 3D 장면 표현의 혁명을 이끈 NeRF(Neural Radiance Fields)와 3D Gaussian Splatting(3DGS) 기술을 SLAM에 접목한 최신 연구들의 등장과 그 폭발적인 발전 과정을 추적한다.


- **NeRF (Neural Radiance Fields)**: 2020년에 처음 등장한 NeRF는 3D 장면을 표현하는 방식을 완전히 바꾸어 놓았다. NeRF는 3D 공간상의 한 점의 좌표 $(x, y, z)$와 그 점을 바라보는 2D 시선 방향 $(\theta, \phi)$을 입력받아, 그 지점의 색상(RGB)과 밀도($\sigma$)를 출력하는 간단한 신경망(MLP)으로 3D 장면 전체를 **암시적(implicitly)**으로 표현한다.6 그리고 **미분 가능한 볼륨 렌더링(differentiable volume rendering)**이라는 기술을 통해, 여러 각도에서 찍은 2D 이미지와 해당 이미지들의 카메라 포즈 정보만으로도 놀랍도록 사실적인 새로운 시점의 이미지를 생성하고 3D 장면을 재구성할 수 있다.33
- **3D Gaussian Splatting (3DGS)**: NeRF는 높은 품질의 결과물을 보여주었지만, 하나의 픽셀 색상을 계산하기 위해 광선(ray)을 따라 수많은 점들을 샘플링하고 신경망을 통과시켜야 하므로 렌더링 속도가 매우 느리다는 치명적인 단점이 있었다. 2023년에 등장한 3DGS는 이 문제를 해결하기 위해 다른 접근법을 취한다. 장면을 수많은 3D 가우시안(각각 위치, 모양, 색상, 불투명도 등의 파라미터를 가짐)의 집합으로 **명시적(explicitly)**으로 표현한다.5 그리고 이 가우시안들을 2D 이미지 평면에 투영하는 **미분 가능한 래스터라이제이션(differentiable rasterization)** 과정을 통해 NeRF보다 훨씬 빠른 속도로 실시간 렌더링과 학습을 달성했다.37

이 두 기술은 SLAM의 'M(Mapping)' 부분을 대체할 매우 강력한 후보로 떠올랐다. 맵 자체가 미분 가능한 신경 표현(neural representation)이 되면서, SLAM의 또 다른 축인 'L(Localization)', 즉 추적(Tracking) 역시 기존의 기하학적 오차 최소화 문제에서, 현재 포즈에서 렌더링한 이미지와 실제 센서 이미지 간의 차이, 즉 **'렌더링 오차(rendering loss)'**를 최소화하는 새로운 최적화 문제로 자연스럽게 변환되었다.6


- **iMAP (ICCV 2021)**: NeRF를 실시간 SLAM 시스템에 적용하려는 초기 시도 중 가장 대표적인 연구다. iMAP은 **단일 MLP**를 맵 전체의 표현으로 사용하고, RGB-D 카메라로부터 들어오는 영상 스트림을 이용해 카메라 포즈와 맵(즉, MLP의 가중치)을 실시간으로 동시에 최적화하는 대담한 시도를 했다.40
  - **문제점**: 하지만 이 방식은 명확한 한계를 가졌다. 단 하나의 MLP가 방 전체, 혹은 건물 전체의 모든 정보를 담아야 하므로, 장면이 조금만 커져도 MLP의 용량이 부족해져 세밀한 부분을 표현하지 못했다. 더 큰 문제는, 새로운 관측이 들어올 때마다 MLP 전체의 가중치가 업데이트되면서 이전에 학습했던 영역의 정보를 잊어버리는 **'파국적 망각(catastrophic forgetting)'** 현상이 발생한다는 점이었다.42
- **NICE-SLAM (CVPR 2022)**: iMAP이 제기한 확장성 문제를 해결하기 위해 등장한 시스템이다.
  - **기술적 참고 관계**: NICE-SLAM은 iMAP의 '신경 표현을 SLAM에 사용한다'는 핵심 아이디어를 계승하면서도, iMAP의 단일 MLP 구조를 근본적으로 바꾸었다.40 바로 **계층적 특징 그리드(hierarchical feature grid)**라는 개념을 도입한 것이다.42
  - **핵심 아이디어**: 장면 전체를 하나의 거대한 신경망으로 표현하는 대신, 3D 공간을 여러 해상도(Coarse, Mid, Fine)의 복셀 그리드로 나눈다. 그리고 각 복셀의 꼭짓점에 학습 가능한 작은 특징 벡터(feature vector)를 저장한다. 어떤 3D 좌표의 색상과 밀도를 알고 싶으면, 그 좌표 주변 8개 복셀의 특징 벡터들을 삼선형 보간(trilinear interpolation)하고, 이 보간된 특징 벡터를 아주 작은 MLP 디코더에 통과시켜 원하는 값을 얻는다. 이 구조 덕분에 맵의 업데이트가 더 이상 전역적(global)이지 않고, 카메라 주변의 **지역적(local)**인 복셀 그리드에 대해서만 이루어진다. 이로써 파국적 망각 문제를 해결하고 대규모 실내 환경으로의 확장이 가능해졌다.42


NICE-SLAM이 확장성의 문을 열자, 연구자들은 더 어려운 문제에 도전하기 시작했다.

- **NeRF-SLAM (IROS 2023)**: 이 연구는 단안 카메라(monocular) 환경이라는 SLAM의 가장 어려운 과제 중 하나를 NeRF와 결합하려 했다.
  - **해결 과제**: NeRF를 학습시키려면 정확한 카메라 포즈와 픽셀별 깊이 정보가 필수적인데, 단안 카메라만으로는 깊이를 직접 알 수 없다.
  - **핵심 아이디어**: 이 문제를 해결하기 위해, NeRF-SLAM은 고전적인 SLAM과 신경장 SLAM을 결합하는 영리한 하이브리드 전략을 사용한다. 먼저, DROID-SLAM과 같은 강력한 최신 **dense monocular SLAM**을 전단(front-end)으로 사용하여, 입력 비디오로부터 정확한 포즈와 함께 (불확실성을 포함한) 조밀한 깊이 맵을 추정한다.43 그리고 이렇게 얻은 포즈와 깊이 정보를 NeRF(후단, back-end)를 학습시키기 위한 **'의사 정답(pseudo-ground-truth)'**으로 활용한다.44 이는 각 분야의 장점만을 취한 성공적인 협력 사례다.
- **Point-SLAM (ICCV 2023)**: NICE-SLAM의 그리드 기반 표현이 가진 비효율성을 개선하기 위해 등장했다.
  - **기술적 참고 관계**: NICE-SLAM의 정적인 그리드 구조가 메모리 낭비를 초래한다는 점에 주목했다.46
  - **핵심 아이디어**: 고정된 복셀 그리드에 특징 벡터를 저장하는 대신, 입력 데이터의 정보 밀도(예: 이미지의 경계선이나 텍스처가 복잡한 부분)에 따라 동적으로 생성되는 **'신경 포인트 클라우드(neural point cloud)'**에 특징 벡터를 앵커링(anchoring)한다.46 즉, 벽과 같이 단조로운 영역에는 포인트를 적게 배치하고, 책상 위 복잡한 물체들 주변에는 포인트를 많이 배치하여 메모리와 계산 자원을 훨씬 효율적으로 사용하는 것이다.46
- **GS4 (2025 추정)**: 3DGS 기술을 기반으로, 신경장 SLAM의 근본적인 작동 방식을 바꾸려는 야심 찬 연구다.
  - **핵심 개념**: 지금까지의 모든 NeRF/3DGS SLAM은 특정 장면에 대해 **'per-scene optimization'**, 즉 테스트 시점에 해당 장면에 대해서만 처음부터 맵을 학습하고 최적화하는 방식을 사용했다. GS4는 이 패러다임을 깨고, 다양한 장면 데이터를 미리 학습한 **일반화 가능한(generalizable)** 단일 네트워크를 제안한다.35
  - **"Generalizable"의 의미**: 이 사전 학습된 피드포워드(feed-forward) 네트워크는, 이전에 한 번도 본 적 없는 새로운 장면에 대해서도 별도의 온라인 최적화나 미세 조정(fine-tuning) 없이, 입력 이미지를 보고 **제로샷(zero-shot)**으로 3D 가우시안들의 파라미터를 직접 예측해낸다.35
  - **Gaussian Refinement Network**: 여러 프레임에서 독립적으로 예측된 가우시안들을 효과적으로 병합하고, 중복되거나 불필요한 가우시안들을 제거하는 역할을 하는 보조 네트워크다. 기존 3DGS SLAM들이 사용하던 복잡한 규칙 기반(heuristic)의 가우시안 추가/제거(densification/pruning) 로직을, 학습 기반의 방식으로 대체하여 훨씬 더 적은 수의 가우시안으로 효율적인 장면 표현을 가능하게 한다.35


신경장 SLAM의 발전사는 **'표현의 효율성과 확장성'**을 향한 끊임없는 투쟁의 역사로 요약될 수 있다. 이 진화의 과정을 따라가 보면, 문제 해결 방식이 어떻게 점진적으로 정교해졌는지 명확히 보인다.

1. **문제의 시작 (iMAP)**: NeRF를 SLAM에 처음 도입하려 했을 때, 가장 단순한 방법은 전체 장면을 하나의 거대한 신경망에 넣는 것이었다. 하지만 이는 곧 비효율적이고 확장 불가능하다는 벽에 부딪혔다.42
2. **첫 번째 해결책 (NICE-SLAM)**: "전체를 한 번에 처리하지 말고, 공간을 잘게 쪼개서 로컬하게 다루자." 이 아이디어는 계층적 그리드를 탄생시켰고, 맵을 지역적으로 업데이트할 수 있게 만들어 대규모 환경으로의 확장성 문제를 해결했다.42
3. **새로운 문제의 발견**: "그런데 공간을 균일하게 쪼개는 것도 낭비 아닌가?" 텅 빈 흰 벽과 책상 위 복잡한 물체들에 동일한 양의 메모리와 계산 자원을 할당하는 것은 명백히 비효율적이었다.46
4. **두 번째 해결책 (Point-SLAM)**: "필요한 곳에만 자원을 집중하자." 정보가 풍부한 곳에만 특징 앵커(포인트)를 동적으로 배치함으로써, 표현의 효율성을 한 단계 더 끌어올렸다.46
5. **근본적인 질문 제기**: "그런데 왜 우리는 매번 새로운 장면을 볼 때마다 바닥부터 다시 학습해야 하는가?" Per-scene optimization 방식은 시간이 오래 걸릴 뿐만 아니라, 이전에 쌓아온 경험을 전혀 활용하지 못하는 근본적인 한계를 가지고 있었다.48
6. **궁극적인 해결책을 향한 시도 (GS4)**: "학습의 일반화를 통해, 경험을 재사용하자." 다양한 장면을 미리 학습해서, 새로운 장면을 보면 '추론'만으로 맵을 생성하자는 아이디어다. 이는 SLAM을 '온라인 최적화 문제'에서, 딥러닝 시대의 '인식/추론 문제'로 전환하려는 혁신적인 시도이며, 진정한 의미의 실시간, 저전력 SLAM을 향한 매우 중요한 걸음이다.35

결론적으로, 이 진화 과정은 SLAM이 점차 고전적인 기하학 기반 최적화 문제에서 벗어나, 현대 딥러닝의 인식(recognition) 및 생성(generation) 패러다임을 적극적으로 수용하고 있음을 명확하게 보여준다. GS4가 제시하는 '일반화'는 이 거대한 흐름의 최전선에 있으며, SLAM의 미래를 바꿀 잠재력을 가지고 있다.


지금까지의 SLAM은 로봇이 "내가 어디에 있는가?"와 "주변 공간은 어떻게 생겼는가?"라는 기하학적 질문에 답하는 데 집중해왔다. 하지만 로봇이 단순히 공간을 배회하는 것을 넘어, 인간과 의미 있는 상호작용을 하거나 복잡한 임무를 수행하기 위해서는 "저기 있는 저것이 무엇인가?"라는 질문에 답할 수 있어야 한다. 이 장에서는 기하학적 맵에 '의미(semantics)'를 부여하는 시맨틱 SLAM의 최신 동향을, 특히 앞서 살펴본 혁신적인 신경장 기술과의 융합을 중심으로 심도 있게 살펴본다.


시맨틱 SLAM은 전통적인 SLAM을 통해 생성된 기하학적 맵(점, 선, 면 등)에 시맨틱 정보(예: 의자, 책상, 문, 사람 등)를 통합하는 기술이다.5 이를 통해 로봇은 다음과 같은 고차원적인 능력을 갖게 된다.

- **지능적 상호작용**: "부엌으로 가서 컵을 가져와"와 같은 인간의 명령을 이해하고 수행할 수 있다. 이는 맵이 단순한 3D 좌표의 집합이 아니라, '부엌'과 '컵'이라는 의미를 가진 객체들로 구성되어 있기 때문에 가능하다.5
- **강인한 위치 추정**: 환경 내의 동적 객체(움직이는 사람, 차량 등)와 정적 객체(벽, 가구 등)를 구분할 수 있다. 동적 객체는 위치 추정에 방해가 되므로 이를 배제하고, 안정적인 정적 객체만을 랜드마크로 활용하여 SLAM의 강인성을 높일 수 있다.49
- **장면 이해**: 로봇이 현재 자신이 어떤 종류의 공간(사무실, 복도, 거실 등)에 있는지 이해하고, 그에 맞는 행동을 계획할 수 있다.

초기 시맨틱 SLAM은 SLAM 시스템과 딥러닝 기반 객체 탐지(Object Detection) 시스템(예: YOLO, Mask R-CNN)을 별도로 구동한 후, 그 결과를 후처리 과정에서 결합하는 방식이 주를 이루었다.49 하지만 최근 연구들은 두 작업을 분리하지 않고, 특징(feature) 레벨에서부터 깊게 융합하여 하나의 통합된 프레임워크 안에서 해결하려는 방향으로 나아가고 있다.5


신경장(NeRF, 3DGS)의 등장은 시맨틱 SLAM에도 새로운 가능성을 열어주었다. 맵 자체가 풍부한 정보를 담을 수 있는 신경망으로 표현되면서, 기하학적 정보와 시맨틱 정보를 훨씬 더 자연스럽고 깊게 통합할 수 있게 된 것이다.

- **SNI-SLAM (CVPR 2024)**: NeRF 기반 시맨틱 SLAM의 대표적인 최신 연구다.
  - **핵심 개념**: 정확한 3D 시맨틱 매핑, 고품질의 표면 재구성, 그리고 강인한 카메라 추적이라는 세 가지 목표를 동시에 달성하는 것을 목표로 한다.50
  - **기술적 참고 관계**: NICE-SLAM과 같은 NeRF 기반 SLAM 시스템을 기반으로 하되, 여기에 시맨틱 정보를 효과적으로 융합하는 방법에 집중한다.
  - **주요 특징**:
    - **계층적 시맨틱 표현**: 장면을 단순히 '의자', '책상'과 같은 단일 레벨의 시맨틱으로 이해하는 것을 넘어, '가구' -> '의자' -> '다리'와 같은 계층적인 구조로 이해할 수 있는 표현 방식을 도입했다.
    - **Cross-Attention 기반 특징 융합**: SNI-SLAM의 가장 핵심적인 아이디어다. 외형(appearance), 기하(geometry), 시맨틱(semantic)이라는 세 가지 종류의 특징을 독립적으로 처리하는 것이 아니라, **교차 어텐션(cross-attention) 메커니즘**을 통해 이들이 서로 정보를 주고받으며 상호 보강하도록 설계했다.50 예를 들어, 기하학적으로 평면임이 확실한 영역에서는 시맨틱 예측의 신뢰도를 높이고, 기하학적으로 불확실한 경계 영역에서는 시맨틱 예측의 영향을 줄이는 식이다. 이는 한 종류의 정보(예: 잘못된 시맨틱 예측)가 전체 시스템에 미치는 악영향을 최소화하고, 다각적인 정보를 바탕으로 종합적인 판단을 내리게 하는 매우 정교한 융합 방식이다.50
  - **검증**: 이 시스템은 Replica, ScanNet과 같은 표준 시맨틱 SLAM 데이터셋에서 기존의 다른 NeRF 기반 SLAM 방법들보다 매핑 및 추적 정확도에서 우수한 성능을 보임을 입증했다.50
- **GS4 (2025 추정) 재조명**: 3DGS 기반의 통합 시맨틱 SLAM.
  - **핵심 개념**: GS4는 시맨틱 정보를 별도의 모듈로 처리하지 않는다. 기하학적 정보(가우시안 파라미터)와 시맨틱 정보를 예측하는 과정을 하나의 **공유된 백본(shared backbone) 네트워크**에서 동시에 수행한다.35
  - **장점**: 이 방식은 외부의 시맨틱 분할(semantic segmentation) 알고리즘에 의존할 필요 없이, 매핑과 인식을 하나의 통합된 프레임워크에서 End-to-End로 처리할 수 있게 한다. 이는 시스템 전체의 효율성과 데이터 처리의 일관성을 크게 향상시키는 장점이 있다.


시맨틱 SLAM의 발전 과정은 **'어떻게 정보를 결합할 것인가'**라는 질문에 대한 답을 찾아가는 여정이었다.

1. **초기 접근법 (단순 결합)**: 기하학적 SLAM으로 3D 맵을 만들고(A), 딥러닝 모델로 2D 이미지에서 객체를 탐지한 후(B), 이 두 결과를 3D 공간에 투영하여 합치는(A+B) 방식이었다. 이 방식은 간단하지만, 두 모듈에서 발생한 오차가 그대로 중첩되고, 두 정보 간의 상호작용이 없어 일관성이 부족하다는 명확한 한계가 있었다.49
2. **SNI-SLAM의 진화 (특징 레벨 융합)**: "결과물이 아니라, 특징(feature) 레벨에서부터 융합하자." SNI-SLAM은 외형, 기하, 시맨틱 특징을 각각 추출한 뒤, 이들이 서로를 참조하고 가중치를 조절하도록(cross-attention) 만들었다. 이를 통해 시스템은 "기하학적으로 보니 이 부분은 평면인데, 외형적으로는 나무 질감이고, 따라서 시맨틱적으로는 '책상'일 확률이 매우 높다"와 같은 복합적이고 정교한 추론을 할 수 있게 되었다. 이는 단순 결합 방식보다 훨씬 더 깊은 수준의 융합이다.50
3. **GS4의 통합 (End-to-End)**: "애초에 분리해서 생각할 필요가 있는가?" GS4는 여기서 한 걸음 더 나아가, 입력 이미지로부터 기하 정보와 시맨틱 정보를 예측하는 과정 자체를 하나의 공유된 네트워크에서 동시에 수행한다. 이는 '공간 재구성(Mapping)'과 '공간 인식(Recognition)'이 본질적으로 분리될 수 없는 하나의 작업이라는 철학을 반영한다. 로봇이 공간을 이해하는 방식이 인간의 인식 과정과 더욱 유사해지는 것이다.35

결론적으로, 시맨틱 SLAM은 기하학적 SLAM 위에 지능을 '덧씌우는' 단계에서, 처음부터 지능적으로 공간을 **'인식'하고 '재구성'하는 방향**으로 진화하고 있다. SNI-SLAM과 GS4는 이러한 패러다임 전환의 선두에 서 있으며, 미래의 로봇이 인간과 같은 수준에서 환경을 이해하고 상호작용하는 데 필수적인 기술적 토대를 마련하고 있다.


이 보고서에서 분석한 최신 SLAM 기술들은 몇 가지 뚜렷한 기술적 흐름을 보여준다. 이러한 흐름을 요약하고, 여전히 남아있는 핵심 과제들을 조명하며, 미래 연구 방향을 전망해 본다.


- **표현의 진화 (Evolution of Representation)**: SLAM의 핵심인 '맵'의 표현 방식은 희소한 특징점(ORB-SLAM)에서 조밀한 픽셀 정보(DROID-SLAM)로, 다시 LiDAR의 명시적인 3D 구조(LIO)를 거쳐, 최종적으로는 장면 전체를 연속적인 함수로 모델링하는 암시적/명시적 신경장(NeRF/3DGS)으로 진화했다. 이는 이산적(discrete) 표현에서 연속적(continuous) 표현으로, 기하학적 표현에서 의미론적(semantic) 표현으로 나아가는 거대한 흐름이다.
- **최적화의 진화 (Evolution of Optimization)**: SLAM의 해를 찾는 방식 또한 변모했다. 특징점의 재투영 오차와 같은 기하학적 오차를 최소화하던 방식에서, 렌더링된 이미지와 실제 이미지의 차이를 줄이는 렌더링 오차 최소화로 전환되었다. 더 나아가, GS4와 같은 연구는 온라인 최적화 자체를 없애고 사전 학습된 모델을 통한 추론(inference)으로 대체하려는 시도를 하고 있다.
- **융합의 진화 (Evolution of Fusion)**: 여러 센서의 정보를 결합하는 방식은 각 센서의 결과를 나중에 합치는 느슨한 결합(loosely-coupled)에서, 모든 측정값을 하나의 문제로 푸는 긴밀한 결합(tightly-coupled)으로 발전했다. 최신 연구들은 여기서 더 나아가, 서로 다른 특징 공간(feature space)에서 정보를 융합하거나(SNI-SLAM), 아예 End-to-End 네트워크를 통해 모든 것을 통합하는(GS4) 방향으로 나아가고 있다.


이러한 눈부신 발전에도 불구하고, SLAM은 여전히 해결해야 할 많은 과제를 안고 있다.

- **확장성 및 실시간성 (Scalability & Real-time Performance)**: NeRF 및 3DGS 기반 방법들은 놀라운 품질의 맵을 생성하지만, 대규모 환경에서는 여전히 메모리 및 계산 비용 문제가 심각하다. 특히 저전력 임베디드 시스템에서 실시간으로 구동하는 것은 아직 큰 숙제로 남아있다.5
- **동적 환경 (Dynamic Environments)**: 대부분의 SLAM 시스템은 여전히 세상이 정적이라고 가정한다. 현실 세계의 수많은 동적 객체(사람, 차량 등)를 효과적으로 모델링하고, 이들로 인해 가려진 배경을 자연스럽게 복원하는 기술은 아직 초기 단계에 머물러 있다.6
- **재인식 및 전역 최적화 (Relocalization & Global Optimization)**: 신경장으로 표현된 맵은 한번 학습되고 나면 수정이 어렵다는 단점이 있다. 루프 클로저가 발생하여 과거의 포즈가 업데이트되었을 때, 이와 관련된 맵 전체를 효율적으로 다시 일관성 있게 만드는 것은 매우 어려운 문제다.38
- **일반화와 강인성 (Generalization & Robustness)**: GS4와 같은 일반화 모델이 등장했지만, 학습 데이터에 존재하지 않았던 새로운 형태의 환경이나 극한의 조명 조건, 심한 센서 노이즈 등 현실 세계의 무한한 다양성에 대해 얼마나 강인한 성능을 보일지는 아직 검증이 더 필요하다.


이러한 과제들을 해결하기 위한 미래 연구는 다음과 같은 방향으로 진행될 것으로 예상된다.

- **알고리즘-하드웨어 공동 설계 (Algorithm-Hardware Co-design)**: 소프트웨어 알고리즘의 최적화를 넘어, SLAM 연산에 특화된 하드웨어 가속기를 설계하여 임베디드 환경에서의 실시간 성능을 확보하려는 연구가 활발해질 것이다.5
- **하이브리드 접근법 (Hybrid Approaches)**: 고전적인 기하학 기반 SLAM이 가진 수학적 안정성과 해석 가능성, 그리고 딥러닝/신경장이 가진 강력한 표현력과 일반화 성능을 결합하는 하이브리드 SLAM 연구는 앞으로도 계속해서 중요한 흐름을 형성할 것이다.
- **평생/지속적 SLAM (Lifelong/Continual SLAM)**: 파국적 망각 문제 없이, 시간이 지남에 따라 새로운 정보를 지속적으로 학습하고 기존 맵을 효율적으로 갱신하며, 결코 멈추지 않는 진정한 의미의 평생 SLAM 기술이 요구된다.
- **암시적-명시적 하이브리드 표현 (Implicit-Explicit Hybrid Representations)**: NeRF가 가진 표현의 연속성과 압축률, 그리고 3DGS가 가진 렌더링 속도와 편집 용이성의 장점만을 결합한 새로운 형태의 하이브리드 신경 표현 방식이 등장할 가능성이 높다.


다음 표는 본 보고서에서 심층적으로 다룬 핵심 기술들의 특징을 한눈에 비교할 수 있도록 정리한 것이다.

| 기술 (발표 연도)      | 핵심 아이디어 / 맵 표현                       | 주요 센서          | 장점                                       | 단점 / 해결 과제                   | 기술적 계보 (참고)           |
| --------------------- | --------------------------------------------- | ------------------ | ------------------------------------------ | ---------------------------------- | ---------------------------- |
| **ORB-SLAM3 (2021)**  | 특징점 기반 MAP 추정 / Atlas 다중 맵          | Visual, V-I        | 높은 정확도, 강인성, 다중 센서/맵 지원     | 저텍스처/동적 환경 취약            | ORB-SLAM2, DBoW2 12          |
| **DROID-SLAM (2021)** | End-to-End 딥러닝 / Dense BA Layer            | Visual, V-I        | 압도적 정확도, 뛰어난 일반화               | 높은 GPU 요구사항, 복잡한 BA       | RAFT, BA-Net 14              |
| **FAST-LIVO2 (2024)** | Direct 방식의 긴밀한 결합 / Unified Voxel Map | LiDAR, IMU, Visual | 높은 효율성, 정확도, 강인성                | 하드웨어 동기화 필수               | FAST-LIO2, FAST-LIVO 30      |
| **NICE-SLAM (2022)**  | 계층적 특징 그리드 기반 NeRF                  | RGB-D              | 대규모 환경으로 확장 가능, 지역적 업데이트 | 그리드 해상도 한계, 빈 공간 샘플링 | iMAP, ConvONet 42            |
| **Point-SLAM (2023)** | 동적 신경 포인트 클라우드                     | RGB-D              | 메모리 효율성, 정보 밀도 기반 적응         | 포인트 관리 복잡성                 | NICE-SLAM, Point-NeRF 46     |
| **GS4 (2025 추정)**   | 일반화 가능한 3DGS / Feed-forward 예측        | RGB-D              | Per-scene 최적화 불필요, 제로샷 일반화     | 일반화 성능의 한계, 데이터 의존성  | 3DGS, Generalizable NeRFs 35 |
| **SNI-SLAM (2024)**   | Cross-Attention 기반 특징 융합 NeRF           | RGB-D              | 기하/외형/시맨틱의 깊은 융합               | 융합 메커니즘의 복잡성             | NICE-SLAM, Semantic Nets 50  |


1. A Comprehensive Survey of Visual SLAM Algorithms - MDPI, accessed July 23, 2025, https://www.mdpi.com/2218-6581/11/1/24
2. A survey on real-time 3D scene reconstruction with SLAM methods in embedded systems - arXiv, accessed July 23, 2025, https://arxiv.org/pdf/2309.05349
3. (PDF) The Simultaneous Localization and Mapping (SLAM)-An Overview - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/359698805_The_Simultaneous_Localization_and_Mapping_SLAM-An_Overview
4. A Review of Research on SLAM Technology Based on the Fusion of LiDAR and Vision, accessed July 23, 2025, https://www.researchgate.net/publication/389419323_A_Review_of_Research_on_SLAM_Technology_Based_on_the_Fusion_of_LiDAR_and_Vision
5. Is Semantic SLAM Ready for Embedded Systems ? A Comparative Survey - arXiv, accessed July 23, 2025, https://arxiv.org/html/2505.12384v1
6. SLAM Meets NeRF: A Survey of Implicit SLAM Methods - MDPI, accessed July 23, 2025, https://www.mdpi.com/2032-6653/15/3/85
7. Robust and Efficient Semantic SLAM with Semantic Keypoints - Y-Prize - University of Pennsylvania, accessed July 23, 2025, https://yprize.upenn.edu/wp-content/uploads/2021/01/Semantic-SLAM-paper.pdf
8. ORB-SLAM3: An Accurate Open-Source Library for Visual, Visual-Inertial and Multi-Map SLAM - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/343179441_ORB-SLAM3_An_Accurate_Open-Source_Library_for_Visual_Visual-Inertial_and_Multi-Map_SLAM
9. UZ-SLAMLab/ORB_SLAM3: ORB-SLAM3: An Accurate Open-Source Library for Visual, Visual-Inertial and Multi-Map SLAM - GitHub, accessed July 23, 2025, https://github.com/UZ-SLAMLab/ORB_SLAM3
10. ORB-SLAM3: An Accurate Open-Source Library for Visual, Visual–Inertial, and Multimap SLAM | Request PDF - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/351862633_ORB-SLAM3_An_Accurate_Open-Source_Library_for_Visual_Visual-Inertial_and_Multimap_SLAM
11. ORB-SLAM3 An Accurate Open-Source Library For Visual VisualInertial and Multimap SLAM - Scribd, accessed July 23, 2025, https://www.scribd.com/document/781310924/ORB-SLAM3-an-Accurate-Open-Source-Library-for-Visual-VisualInertial-and-Multimap-SLAM
12. [2007.11898] ORB-SLAM3: An Accurate Open-Source Library for ..., accessed July 23, 2025, https://ar5iv.labs.arxiv.org/html/2007.11898
13. DROID-SLAM: Deep Visual SLAM for Monocular, Stereo, and RGB-D Cameras | Request PDF - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/354115433_DROID-SLAM_Deep_Visual_SLAM_for_Monocular_Stereo_and_RGB-D_Cameras
14. DROID-SLAM: Deep Visual SLAM for Monocular, Stereo, and RGB-D Cameras, accessed July 23, 2025, https://proceedings.neurips.cc/paper/2021/file/89fcd07f20b6785b92134bd6c1d0fa42-Paper.pdf
15. DROID-SLAM: Deep Visual SLAM for Monocular, Stereo, and RGB-D Cameras - Papertalk, accessed July 23, 2025, https://papertalk.org/papertalks/37631
16. DROID-SLAM: Deep Visual SLAM for Monocular, Stereo, and RGB-D Cameras, accessed July 23, 2025, https://collaborate.princeton.edu/en/publications/droid-slam-deep-visual-slam-for-monocular-stereo-and-rgb-d-camera
17. DROID-SLAM: Deep Visual SLAM for Monocular, Stereo, and RGB-D Cameras, accessed July 23, 2025, https://openreview.net/forum?id=ZBfUo_dr4H
18. accessed January 1, 1970, [https.proceedings.neurips.cc/paper/2021/file/89fcd07f20b6785b92134bd6c1d0fa42-Paper.pdf](http://docs.google.com/https.proceedings.neurips.cc/paper/2021/file/89fcd07f20b6785b92134bd6c1d0fa42-Paper.pdf)
19. princeton-vl/DROID-SLAM - GitHub, accessed July 23, 2025, https://github.com/princeton-vl/DROID-SLAM
20. A Visual-Inertial SLAM Based on Point-Line Features and Efficient IMU Initialization - arXiv, accessed July 23, 2025, https://arxiv.org/html/2401.01081v2
21. LiDAR-based SLAM for robotic mapping: state of the art and new frontiers | Emerald Insight, accessed July 23, 2025, https://www.emerald.com/insight/content/doi/10.1108/ir-09-2023-0225/full/pdf?title=lidar-based-slam-for-robotic-mapping-state-of-the-art-and-new-frontiers
22. (PDF) A Comprehensive Survey of Visual SLAM Algorithms - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/358523574_A_Comprehensive_Survey_of_Visual_SLAM_Algorithms
23. A Review of Visual-LiDAR Fusion based Simultaneous Localization and Mapping - PMC, accessed July 23, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC7181037/
24. Online LiDAR-SLAM for Legged Robots with Robust Registration and Deep-Learned Loop Closure, accessed July 23, 2025, https://ori.ox.ac.uk/media/5601/2020icra_ramezani.pdf
25. 面向鲁棒的传感器融合地面SLAM, accessed July 23, 2025, https://www.xueshuxiangzi.com/downloads/2025_7_14/2507.08364.pdf
26. [2107.06829] FAST-LIO2: Fast Direct LiDAR-inertial Odometry - arXiv, accessed July 23, 2025, https://arxiv.org/abs/2107.06829
27. sjtuyinjie/awesome-LiDAR-Visual-SLAM - GitHub, accessed July 23, 2025, https://github.com/sjtuyinjie/awesome-LiDAR-Visual-SLAM
28. LIR-LIVO: A Lightweight,Robust LiDAR/Vision/Inertial Odometry with Illumination-Resilient Deep Features - arXiv, accessed July 23, 2025, [https://arxiv.org/pdf/2502.08676?](https://arxiv.org/pdf/2502.08676)
29. FAST-LIVO2: Fast, Direct LiDAR-Inertial-Visual Odometry | Request PDF - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/383428735_FAST-LIVO2_Fast_Direct_LiDAR-Inertial-Visual_Odometry
30. FAST-LIVO2: Fast, Direct LiDAR-Inertial-Visual Odometry - arXiv, accessed July 23, 2025, https://arxiv.org/html/2408.14035v2
31. hku-mars/FAST-LIVO: A Fast and Tightly-coupled Sparse-Direct LiDAR-Inertial-Visual Odometry (LIVO). - GitHub, accessed July 23, 2025, https://github.com/hku-mars/FAST-LIVO
32. RLI-SLAM: Fast Robust Ranging-LiDAR-Inertial Tightly-Coupled ..., accessed July 23, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11398178/
33. NeRFs in Robotics: A Survey - arXiv, accessed July 23, 2025, https://arxiv.org/html/2405.01333v2
34. Neural Radiance Fields for the Real World: A Survey - arXiv, accessed July 23, 2025, https://arxiv.org/html/2501.13104v1
35. arxiv.org, accessed July 23, 2025, https://arxiv.org/html/2506.06517v1
36. 3D Gaussian Splatting for Modern Architectural Heritage: Integrating UAV-Based Data Acquisition and Advanced Photorealistic 3D - AGILE-GISS, accessed July 23, 2025, https://agile-giss.copernicus.org/articles/6/51/2025/agile-giss-6-51-2025.pdf
37. A Survey on 3D Gaussian Splatting, accessed July 23, 2025, https://www.cs.jhu.edu/~misha/ReadingSeminar/Papers/Chen24.pdf
38. [2402.13255] How NeRFs and 3D Gaussian Splatting are Reshaping SLAM: a Survey, accessed July 23, 2025, https://arxiv.org/abs/2402.13255
39. arXiv:2503.18275v1 [cs.RO] 24 Mar 2025, accessed July 23, 2025, https://arxiv.org/pdf/2503.18275
40. 3D-Vision-World/awesome-NeRF-and-3DGS-SLAM - GitHub, accessed July 23, 2025, https://github.com/3D-Vision-World/awesome-NeRF-and-3DGS-SLAM
41. iMAP: Implicit Mapping and Positioning in Real ... - CVF Open Access, accessed July 23, 2025, https://openaccess.thecvf.com/content/ICCV2021/papers/Sucar_iMAP_Implicit_Mapping_and_Positioning_in_Real-Time_ICCV_2021_paper.pdf
42. NICE-SLAM: Neural Implicit Scalable Encoding ... - CVF Open Access, accessed July 23, 2025, https://openaccess.thecvf.com/content/CVPR2022/papers/Zhu_NICE-SLAM_Neural_Implicit_Scalable_Encoding_for_SLAM_CVPR_2022_paper.pdf
43. [2210.13641] NeRF-SLAM: Real-Time Dense Monocular SLAM with ..., accessed July 23, 2025, https://ar5iv.labs.arxiv.org/html/2210.13641
44. Publications - Luca Carlone - MIT, accessed July 23, 2025, https://lucacarlone.mit.edu/research/publications/
45. NeRF-SLAM: Real-Time Dense Monocular SLAM with Neural Radiance Fields | Request PDF - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/376510832_NeRF-SLAM_Real-Time_Dense_Monocular_SLAM_with_Neural_Radiance_Fields
46. Point-SLAM: Dense Neural Point Cloud-based ... - CVF Open Access, accessed July 23, 2025, https://openaccess.thecvf.com/content/ICCV2023/papers/Sandstrom_Point-SLAM_Dense_Neural_Point_Cloud-based_SLAM_ICCV_2023_paper.pdf
47. Point-SLAM Dense Neural Point Cloud-based SLAM | PDF | Rendering (Computer Graphics) - Scribd, accessed July 23, 2025, https://www.scribd.com/document/882125639/Point-SLAM-Dense-Neural-Point-Cloud-based-SLAM
48. [2506.06517] GS4: Generalizable Sparse Splatting Semantic SLAM - arXiv, accessed July 23, 2025, https://arxiv.org/abs/2506.06517
49. A Computationally Efficient Semantic SLAM Solution for Dynamic Scenes - MDPI, accessed July 23, 2025, https://www.mdpi.com/2072-4292/11/11/1363
50. SNI-SLAM: Semantic Neural Implicit SLAM | CVF Open Access, accessed July 23, 2025, https://openaccess.thecvf.com/content/CVPR2024/papers/Zhu_SNI-SLAM_Semantic_Neural_Implicit_SLAM_CVPR_2024_paper.pdf
51. How NeRFs and 3D Gaussian Splatting are Reshaping ... - Fabio Tosi, accessed July 23, 2025, https://fabiotosi92.github.io/files/survey-slam.pdf

