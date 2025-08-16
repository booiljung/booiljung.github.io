# LB-LIOSAM에 대한 종합적 기술 고찰: 아키텍처 통합, 성능 분석 및 연구 맥락





### 초록



본 보고서는 개선된 LiDAR-관성 SLAM 프레임워크인 LB-LIOSAM에 대한 포괄적인 기술 분석을 제공한다. 이 시스템은 기반이 되는 LIO-SAM 아키텍처에 LinK3D 특징 추출 알고리즘과 BoW3D 폐루프 검출 알고리즘을 통합하여 성능을 향상시킨 구조를 가지고 있다. 본 보고서는 대규모 및 특징이 부족한 환경에서 발생하는 LIO-SAM의 주요 한계점, 특히 누적 오차 문제를 해결하는 데 있어 이러한 구성 요소들이 수행하는 구체적인 역할을 해부한다. 또한, LB-LIOSAM의 성능 주장을 비판적으로 평가하고, 관련 시스템들의 기존 벤치마크와 비교하여 그 맥락을 파악하며, 재현을 위한 실질적인 가이드를 제공한다. 마지막으로, SLAM 연구의 광범위한 지형 속에서 LB-LIOSAM의 독창성을 평가하고 향후 연구 방향을 제안한다.

------



## 1.  LIO-SAM으로부터의 진화: LiDAR-관성 SLAM의 핵심 과제 해결





### 1.1  LIO-SAM 패러다임: 최첨단 기술의 기준점



LIO-SAM(Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping)은 로봇 공학 커뮤니티에서 가장 영향력 있고 널리 인정받는 최첨단 SLAM(Simultaneous Localization and Mapping) 프레임워크 중 하나로 확고히 자리 잡았다.1 이 시스템의 핵심은 LiDAR(Light Detection and Ranging)와 관성 측정 장치(IMU) 데이터를 요인 그래프(factor graph) 기반으로 강결합(tightly-coupled)하여 실시간으로 모바일 로봇의 궤적을 정밀하게 추정하고 지도를 구축하는 데 있다.2

LIO-SAM의 아키텍처는 두 개의 독립적인 요인 그래프를 유지 관리하는 독특한 구조를 가진다. 첫 번째 그래프는 'imuPreintegration.cpp' 모듈에서 관리되며, IMU와 LiDAR 주행 거리 측정(odometry) 요인을 최적화하여 IMU 바이어스를 추정한다. 이 그래프는 주기적으로 초기화되어 IMU 주파수에서 실시간 주행 거리 측정 추정을 보장한다. 두 번째 그래프는 'mapOptimization.cpp' 모듈에서 관리되며, LiDAR 주행 거리 측정 요인과 GPS 요인을 최적화하여 전체 테스트 기간 동안 일관성을 유지하는 전역 지도를 생성한다.5

이러한 요인 그래프는 네 가지 주요 유형의 제약 조건을 통합하여 최적화를 수행한다.1

1. **IMU 사전적분 요인(IMU Preintegration Factors):** 연속적인 LiDAR 스캔 사이의 IMU 측정값을 사전적분하여 로봇의 모션 제약을 제공한다.
2. **LiDAR 주행 거리 측정 요인(LiDAR Odometry Factors):** 연속적인 LiDAR 키프레임 간의 스캔 매칭을 통해 얻은 상대적인 변환을 나타낸다.
3. **GPS 요인(GPS Factors):** (선택 사항) GPS 수신기로부터 얻은 절대 위치 정보를 제공하여 전역적인 드리프트(drift)를 보정한다.
4. **폐루프 요인(Loop Closure Factors):** 로봇이 이전에 방문했던 장소를 다시 인식했을 때 생성되는 제약으로, 누적된 오차를 크게 줄여 전역 지도의 일관성을 보장한다.

이러한 구조는 LIO-SAM이 높은 정확도와 실시간 성능을 달성할 수 있게 하는 기반이 되며, 이후 LB-LIOSAM이 어떤 부분을 어떻게 개선했는지 이해하는 데 중요한 기준점을 제공한다.



### 1.2  도전적인 환경에서의 LIO-SAM의 취약점 분석



LIO-SAM은 수많은 환경에서 뛰어난 성능을 입증했지만, 특정 조건에서는 명백한 한계를 드러낸다. 이러한 취약점은 LB-LIOSAM과 같은 개선된 알고리즘의 개발을 촉발하는 직접적인 동기가 되었다. 가장 두드러지는 문제는 대규모 환경, 특히 구조적으로 반복되거나(repetitive structures) 기하학적 특징이 부족한(feature-degenerate) 환경에서 강건성(robustness)이 저하되고 상당한 누적 오차가 발생하는 것이다.6 이는 긴 복도, 넓은 개활지, 또는 유사한 건물이 반복되는 도시 환경 등에서 흔히 발생하는 SLAM 시스템의 일반적인 실패 모드이다.7

이러한 실패의 근본 원인은 LIO-SAM의 핵심 구성 요소 두 가지에서 찾을 수 있다.

첫째, **특징 추출의 취약성**이다. LIO-SAM의 네이티브 특징 추출 방식은 각 점의 국소 영역 내 '거칠기(roughness)'를 계산하여 단순한 평면(planar) 특징과 모서리(edge) 특징을 식별하는 것에 의존한다.3 이 방법은 구조적으로 명확한 환경에서는 효과적이지만, 기하학적으로 모호한 환경에서는 신뢰할 수 있는 특징을 충분히 추출하지 못한다. 예를 들어, 특징이 없는 긴 터널이나 벽에서는 안정적인 평면 및 모서리 특징을 찾기 어려워 주행 거리 측정 요인의 정확도가 저하되고, 이는 직접적으로 위치 추정 오차의 누적으로 이어진다.6

둘째, **폐루프 검출의 불충분성**이다. LIO-SAM의 기본 폐루프 검출 메커니즘은 "단순하지만 효과적인 유클리드 거리 기반" 검색 방식이다.3 이 방식은 현재 위치와 가까운 과거의 키프레임을 찾는 데 초점을 맞춘다. 하지만 장거리 주행으로 인해 드리프트가 크게 누적된 경우, 실제로는 같은 위치임에도 불구하고 추정된 좌표 상의 거리는 멀어져 루프를 검출하지 못할 수 있다. 또한, 시각적으로 유사하지만 실제로는 다른 장소(perceptual aliasing)를 잘못된 루프로 인식할 위험도 내포하고 있다.6 이처럼 정확한 폐루프 제약 조건을 요인 그래프에 추가하지 못하면, 누적된 오차를 바로잡을 기회를 잃게 되어 결국 지도 왜곡과 부정확한 전역 위치 추정으로 귀결된다.



### 1.3  연구의 필요성: 강화된 특징 표현과 장소 인식의 요구



앞서 분석한 LIO-SAM의 취약점들은 하나의 명확한 연구 과제로 귀결된다. "대규모의 도전적인 시나리오에서 성능을 향상시키기 위해, 어떻게 LIO-SAM 프레임워크의 프론트엔드 특징 추출과 백엔드 폐루프 검출을 보다 강건한 모듈로 강화할 수 있는가?"라는 질문이 바로 그것이다. 이 질문에 대한 해답을 찾는 과정에서 LB-LIOSAM이 탄생했다.

LIO-SAM 프레임워크의 근간은 강력한 최적화 라이브러리인 GTSAM(Georgia Tech Smoothing and Mapping) 위에 구축되어 있어 매우 견고하다.5 시스템의 약점은 최적화 엔진 자체가 아니라, 그 엔진에 입력되는 데이터, 즉 '요인(factor)'의 품질에 있다. 특히 환경적 도전에 가장 민감한 것이 바로 LiDAR 주행 거리 측정 요인과 폐루프 요인이다. 따라서 전체 시스템의 성능을 가장 효과적으로 향상시키는 방법은 이 '가장 약한 연결고리'인 특징 추출기와 루프 검출기를 더 강력하고 전문화된 모듈로 교체하는 것이다. LB-LIOSAM은 이러한 가설을 직접적으로 구현한 결과물이다.

이러한 접근 방식은 로봇 공학 연구에서 흔히 볼 수 있는 패턴으로, LIO-SAM과 같이 견고한 기반 시스템은 LVI-SAM이나 FAST-LIO-SAM과 같은 다른 변종들에서 볼 수 있듯이 추가적인 혁신을 위한 플랫폼 역할을 한다.5 따라서 LB-LIOSAM의 개발은 완전한 재설계가 아닌, 목표 지향적인 모듈식 업그레이드로 이해할 수 있다. 이는 기존의 강력한 프레임워크를 활용하면서 특정 약점을 집중적으로 보완하여 전체 시스템의 성능을 한 단계 끌어올리는 효율적인 연구 전략의 좋은 예시이다.

------



## 2.  아키텍처 해부: LinK3D와 BoW3D의 LIO-SAM 요인 그래프 통합





### 2.1  시스템 개요 및 명명 규칙



LB-LIOSAM은 "LinK3D, BoW3D, 그리고 LIO-SAM을 통합하여 개선된 매핑 및 측위 방법"으로 공식적으로 정의된다.6 이 시스템은 LIO-SAM의 견고한 요인 그래프 최적화 프레임워크를 기반으로 하되, 누적 오차에 취약했던 두 가지 핵심 모듈을 교체하여 성능을 극대화하는 것을 목표로 한다.

시스템의 명칭은 이러한 통합 구조를 명확하게 반영한다. 이름의 'L'은 프론트엔드 특징 추출을 담당하는 **L**inK3D에서, 'B'는 백엔드 폐루프 검출을 담당하는 **B**oW3D에서 가져왔으며, 이를 기본 프레임워크인 **LIO-SAM**과 결합한 것이다.6

LB-LIOSAM의 전체적인 데이터 흐름은 다음과 같이 개념적으로 요약할 수 있다. 센서(LiDAR, IMU, GPS)로부터 입력된 데이터는 먼저 LIO-SAM의 전통적인 IMU 사전적분 및 포인트 클라우드 왜곡 보정 단계를 거친다. 그 후, 왜곡이 보정된 포인트 클라우드는 기존의 특징 추출 모듈 대신 LinK3D 모듈로 전달되어 고품질의 특징점을 생성한다. 이 특징점들은 LiDAR 측정 요인을 생성하는 데 사용된다. 동시에, 이 특징점들은 BoW3D 모듈로 전달되어 폐루프 검출을 위한 데이터베이스를 구축하고 쿼리하는 데 사용된다. BoW3D가 폐루프를 성공적으로 검출하면, 해당 정보는 폐루프 요인으로 변환되어 중앙 요인 그래프에 추가된다. 최종적으로, 이 강화된 요인 그래프는 GTSAM을 통해 최적화되어 더 정확한 전역 포즈와 지도 추정치를 산출한다.



### 2.2  프론트엔드 강화: 강건한 특징 추출을 위한 LinK3D





#### 2.2.1  LinK3D의 원리



LB-LIOSAM의 프론트엔드 강화의 핵심은 LinK3D(Linear Keypoints Representation for 3D LiDAR Point Cloud) 알고리즘의 도입에 있다. LIO-SAM이 점의 '거칠기'와 같은 단순한 기하학적 속성을 사용하는 것과 달리, LinK3D는 각 키포인트(keypoint)를 주변의 이웃 키포인트들과의 관계를 인코딩하여 설명하는 훨씬 정교한 접근 방식을 취한다.14 구체적으로, LinK3D는 각 키포인트에 대해 180차원의 벡터 디스크립터(descriptor)를 생성한다. 이 디스크립터는 키포인트의 로컬 기하학적 구조를 풍부하게 표현하도록 설계되어, 대규모 LiDAR 스캔에서 흔히 발생하는 점의 희소성(sparsity), 불균일한 분포, 그리고 가려짐(occlusion) 현상에 대해 높은 강건성을 보인다.15

LinK3D는 기존의 히스토그램 기반 수동 특징(hand-crafted features)이나 학습 기반 특징(learning-based features)과는 다른 전략을 사용한다. 이는 대규모 실외 환경의 LiDAR 데이터 특성을 깊이 고려한 결과이다. 예를 들어, 고정된 크기의 공간에 점이 충분하지 않으면 효과적인 통계적 기술자를 생성하기 어렵고, 동적 객체(차량, 보행자)로 인해 동일한 장소의 로컬 표면이 시간에 따라 다르게 보일 수 있다. LinK3D는 이러한 문제에 대응하기 위해 견고한 집계 키포인트를 먼저 추출하고, 이 키포인트들을 기반으로 기술자를 생성하여 설명의 일관성을 높인다.15 이 알고리즘의 독창성과 기술적 세부 사항은 원본 논문에서 더 자세히 확인할 수 있다.14



#### 2.2.2  LiDAR 측정 요인으로서의 통합



LB-LIOSAM은 LIO-SAM의 기존 특징 추출 모듈을 LinK3D로 완전히 대체한다. 새로운 LiDAR 스캔이 들어오면, 시스템은 LinK3D를 사용하여 특징점을 추출하고, 연속적인 키프레임 간에 이 특징점들을 매칭한다. LinK3D는 정확한 점 대 점(point-to-point) 매칭 결과를 실시간으로 생성할 수 있는 능력을 갖추고 있다.14

이렇게 생성된 고품질의 특징점 매칭 결과는 LIO-SAM의 요인 그래프에 "LiDAR에 대한 거리 요인(range factors for the LiDAR)"으로 추가된다.6 이 요인들은 본질적으로 LIO-SAM의 원래 LiDAR 주행 거리 측정 요인과 동일한 역할을 수행한다. 즉, 연속된 키프레임 간의 기하학적 제약을 나타낸다. 하지만 그 기반이 되는 데이터가 훨씬 더 정확하고 신뢰할 수 있기 때문에, 결과적으로 주행 거리 측정의 정확도가 크게 향상되고 드리프트가 감소한다. 이는 특히 특징이 부족한 환경에서 LIO-SAM이 겪던 성능 저하 문제를 직접적으로 해결하는 핵심적인 개선 사항이다.



### 2.3  백엔드 최적화: 고신뢰도 폐루프를 위한 BoW3D





#### 2.3.1  BoW3D의 원리



LB-LIOSAM의 백엔드 최적화는 BoW3D(Bag of Words for Real-Time Loop Closing in 3D LiDAR SLAM) 알고리즘을 통해 이루어진다. BoW3D는 시각 SLAM 분야에서 큰 성공을 거둔 BoW(Bag-of-Words) 모델을 3D LiDAR SLAM에 적용한 것이다.16 이 알고리즘의 핵심 아이디어는 3D "단어(word)"들의 모음, 즉 "단어 가방(bag of words)"을 구축하여 장소를 효율적으로 인식하는 것이다.

BoW3D에서 각 "단어"는 프론트엔드에서 추출된 LinK3D 특징 디스크립터에 해당한다.18 시스템이 새로운 키프레임을 처리할 때마다, 해당 프레임에서 추출된 LinK3D 디스크립터들을 BoW3D 데이터베이스에 추가한다. 이 데이터베이스는 해시 테이블을 기반으로 구축되어 빠른 검색과 업데이트가 가능하다.16 이러한 구조 덕분에, 시스템은 새로운 장소를 탐색하면서도 과거에 방문했던 장소와 유사한 "단어" 구성을 가진 곳을 실시간으로 찾아낼 수 있다. 이 과정이 바로 장소 인식(place recognition)이며, 폐루프 검출의 첫 단계이다.



#### 2.3.2  폐루프 요인으로서의 통합



BoW3D는 LIO-SAM의 백엔드에 통합되어 기존의 유클리드 거리 기반 폐루프 검출 방식을 대체한다. 새로운 키프레임이 생성되면, 그 프레임의 LinK3D 특징들은 BoW3D 데이터베이스를 쿼리하는 데 사용된다. 만약 데이터베이스 검색을 통해 현재 키프레임과 과거의 특정 키프레임 간에 충분한 유사성이 발견되면(즉, 재방문한 장소로 판단되면), BoW3D는 루프 후보를 식별한다.

단순히 장소를 인식하는 것을 넘어, BoW3D는 LinK3D 특징들의 정확한 점 대 점 매칭 정보를 활용하여 현재 키프레임과 과거 키프레임 간의 완전한 6자유도(6-DoF) 상대 변환(relative pose)을 계산할 수 있다.16 이 계산된 상대 변환은 요인 그래프에 "폐루프 요인(loop closure factor)"으로 추가된다.6 이 요인은 두 시간대의 포즈 노드 사이에 강력한 전역 제약 조건을 형성하여, 그동안 누적된 모든 드리프트를 보정하는 역할을 한다. 이는 LIO-SAM이 대규모 환경에서 겪던 지도 왜곡과 전역 포즈 오차 문제를 해결하는 데 결정적인 기여를 한다.



### 2.4  LB-LIOSAM의 통합된 요인 그래프



앞서 설명한 개선 사항들을 종합하면, LB-LIOSAM의 최종적인 요인 그래프는 다음과 같은 네 가지 핵심 제약 요인들을 공동으로 최적화하는 강력한 시스템으로 구성된다.6

1. **IMU 사전적분 요인:** LIO-SAM 원본과 동일하게 유지되며, 고주파 모션 정보를 제공한다.
2. **GPS 요인:** (선택 사항) LIO-SAM 원본과 동일하게 유지되며, 전역적인 위치 기준을 제공한다.
3. **LiDAR 측정 요인:** LIO-SAM의 단순한 기하학적 특징 대신, 훨씬 더 강건하고 설명력이 풍부한 **LinK3D** 매칭으로부터 생성된다. 이는 주행 거리 측정의 정확도를 크게 향상시킨다.
4. **폐루프 요인:** LIO-SAM의 거리 기반 검색 대신, 더 신뢰성 높은 **BoW3D** 장소 인식으로부터 생성된다. 이는 대규모 환경에서 누적 오차를 효과적으로 보정한다.

이러한 구조를 면밀히 살펴보면, LB-LIOSAM의 설계가 단순히 독립적인 고성능 알고리즘 두 개를 무작위로 선택한 것이 아님을 알 수 있다. 연구 자료에 따르면, BoW3D는 애초에 LinK3D 특징을 입력 "단어"로 사용하도록 명시적으로 설계되었다.16 이는 LB-LIOSAM의 개발자들이 두 알고리즘 간의 시너지 효과를 인지하고 의도적으로 이 조합을 선택했음을 시사한다. LinK3D는 강건하고 포즈에 불변하는 특징(어휘)을 제공하고, BoW3D는 이를 위한 효율적인 데이터베이스 및 검색 메커니즘(사전)을 제공한다. 이러한 지능적인 구성 요소 선택은 통합 과정을 단순화하고 두 모듈이 효과적으로 함께 작동하도록 보장하여 성능 향상을 극대화했을 가능성이 높다. 따라서 LB-LIOSAM의 기여는 단순히 모듈을 연결하는 것을 넘어, 널리 사용되는 프레임워크 내의 알려진 문제를 해결하기 위해 기존의 강력한 모듈 조합을 식별하고 그 효과를 입증한 데 있다.

------



## 3.  성능 평가 및 비교 맥락





### 3.1  발표된 성능 주장 및 실험 설정



LB-LIOSAM에 대한 주요 연구 논문은 해당 알고리즘의 성능에 대해 강력한 질적 주장을 제시한다.6 이 연구의 핵심 결과는 LB-LIOSAM이 기존 LIO-SAM과 비교했을 때 "누적 오차를 현저히 줄이고", "더 우수한 매핑 및 측위 효과"를 보인다는 것이다. 이는 제안된 아키텍처 개선이 목표했던 바를 성공적으로 달성했음을 시사한다.

실험은 "본 논문에서 제안된 실험 시나리오"에서 "전기 제어 자동차"를 사용하여 수행되었다.6 이 실험 설정은 알고리즘이 특정 환경 조건 하에서 어떻게 작동하는지 평가하기 위해 설계된 것으로 보인다. 하지만 논문에서는 이 실험 시나리오의 구체적인 특성(예: 환경의 크기, 특징의 밀도, 루프의 존재 여부 등)이나 사용된 센서의 사양에 대한 자세한 정보를 제공하지 않고 있다. 이는 실험 결과의 일반화 가능성을 평가하는 데 한계로 작용한다.



### 3.2  평가 방법론에 대한 비판적 분석



LB-LIOSAM의 아키텍처 개선은 이론적으로 매우 타당하지만, 그 성능을 입증하는 경험적 증거는 몇 가지 중요한 한계를 가지고 있다. 가장 결정적인 부분은 표준화된 정량적 벤치마킹의 부재이다.

주요 연구 자료 6는 "현저한 감소"나 "우수한 효과"와 같은 강력한 질적 주장을 하지만, SLAM 커뮤니티에서 널리 사용되는 표준적인 정량적 평가 지표를 제시하지 않는다. 예를 들어, 절대 궤적 오차(Absolute Trajectory Error, ATE)나 상대 포즈 오차(Relative Pose Error, RPE)와 같은 수치적 데이터가 없다. 이러한 지표는 알고리즘의 정확도를 객관적으로 평가하고 다른 시스템과 직접 비교하는 데 필수적이다.

더욱이, 이 연구는 "본 논문에서 제안된 실험 시나리오"라는 비공개(private) 데이터셋을 사용했다.6 이는 연구의 재현성을 심각하게 저해한다. 다른 연구자들이 동일한 조건에서 결과를 검증하거나, 자신들의 알고리즘과 성능을 비교하는 것이 불가능하다. 많은 선행 연구들이 KITTI 19, UrbanLoco 19, 또는 M2DGR 14과 같은 공개 데이터셋을 사용하여 자신들의 알고리즘을 벤치마킹하는 것과는 대조적이다.

결론적으로, LB-LIOSAM의 아키텍처적 우수성에도 불구하고, 제공된 경험적 증거는 일화적(anecdotal)이며 과학적 검증이 부족하다고 평가할 수 있다. 이 알고리즘의 진정한 가치를 평가하기 위해서는, 널리 인정된 공개 데이터셋을 사용하여 LIO-SAM 원본 및 다른 최신 알고리즘들과의 엄격한 정량적 비교가 반드시 필요하다.



### 3.3  성능 기준점 설정: LIO-SAM 및 관련 변종들



LB-LIOSAM 논문 자체에 정량적 데이터가 부족하기 때문에, 그 성능 주장의 타당성을 평가하기 위해서는 관련 시스템들의 성능을 기준으로 삼아 맥락을 파악하는 것이 필수적이다. 이를 위해 LIO-SAM 및 그 주요 변종들의 아키텍처와 알려진 성능을 검토하여 LB-LIOSAM이 위치하는 기술적 지형을 그려볼 수 있다.

LIO-SAM은 그 자체로도 강력한 성능을 보이지만, 특정 환경에서는 한계를 보이며, 이를 개선하기 위한 여러 연구가 진행되었다.1 예를 들어, 한 연구에서는 ID-LIO라는 알고리즘이 특정 도시 데이터셋(UrbanLoco-CAMarketStreet, UrbanNav-HK-Medium-Urban-1)에서 LIO-SAM 대비 ATE를 각각 67%와 85% 개선했다고 보고했다.19 이는 "현저한" 성능 향상이 어느 정도의 수치를 의미하는지에 대한 구체적인 기준점을 제공한다. LB-LIOSAM이 이와 유사하거나 더 나은 수준의 개선을 이루었는지를 평가하기 위해서는 이러한 정량적 데이터가 필요하다.

아래 두 개의 표는 LB-LIOSAM과 그 관련 시스템들을 체계적으로 비교하여, 독자가 LB-LIOSAM의 독창적인 기여와 잠재적 성능을 더 명확하게 이해하는 데 도움을 줄 것이다.



#### 3.3.1 표 1: LIO-SAM 변종들의 아키텍처 비교 분석



이 표는 LB-LIOSAM과 그와 관련된 주요 SLAM 시스템들의 설계 철학을 명확하게 비교하여, 각 알고리즘의 핵심 기여를 구조적으로 보여준다.

| 알고리즘 명칭       | 주요 센서 입력     | 특징 추출 방식                 | 폐루프 검출 메커니즘             | 핵심 기여                                   |
| ------------------- | ------------------ | ------------------------------ | -------------------------------- | ------------------------------------------- |
| **LIO-SAM** 3       | LiDAR, IMU, GPS    | 모서리/평면 특징 (거칠기 기반) | 유클리드 거리 기반 검색          | 강결합 LiDAR-관성 주행 거리 측정의 기반     |
| **LVI-SAM** 12      | LiDAR, Visual, IMU | 시각 특징 + LiDAR 모서리/평면  | 시각적 장소 인식 + 기하학적 정제 | LiDAR와 시각 시스템의 강건한 융합           |
| **FAST-LIO-SAM** 13 | LiDAR, IMU         | (FAST-LIO2로부터 암시적)       | 반경 검색 + ICP                  | 빠른 LIO 프론트엔드와 LIO-SAM 백엔드의 결합 |
| **LB-LIOSAM** 6     | LiDAR, IMU, GPS    | **LinK3D**                     | **BoW3D**                        | 특징 추출 및 폐루프 검출 강건성 강화        |



#### 3.3.2 표 2: 공개 데이터셋에서의 LIO-SAM 및 관련 시스템의 정량적 성능



이 표는 LB-LIOSAM의 질적 성능 주장을 평가할 수 있는 구체적인 수치 기준을 제공한다. 아래 데이터는 관련 연구에서 보고된 값들을 예시로 구성한 것이다.

| 알고리즘     | 데이터셋                   | 평가 지표            | 보고된 값                    | 출처 |
| ------------ | -------------------------- | -------------------- | ---------------------------- | ---- |
| LIO-SAM      | UrbanLoco-CAMarketStreet   | ATE RMSE [m]         | ~1.5 - 2.0 (추정치)          | 19   |
| ID-LIO       | UrbanLoco-CAMarketStreet   | ATE RMSE [m]         | 0.49 (LIO-SAM 대비 67% 향상) | 19   |
| LIO-SAM      | UrbanNav-HK-Medium-Urban-1 | ATE RMSE [m]         | ~3.0 - 3.5 (추정치)          | 19   |
| ID-LIO       | UrbanNav-HK-Medium-Urban-1 | ATE RMSE [m]         | 0.51 (LIO-SAM 대비 85% 향상) | 19   |
| LIO-SAM      | KITTI Odometry 00          | ATE RMSE [m]         | 1.83 (예시 값)               | 1    |
| FAST-LIO2    | KITTI Odometry 05          | Trajectory Drift [%] | 0.81                         | 13   |
| FAST-LIO-SAM | KITTI Odometry 05          | Trajectory Drift [%] | 0.59                         | 13   |

*참고: 위 표의 LIO-SAM 추정치는 ID-LIO의 개선율을 바탕으로 역산한 값이며, 실제 성능은 실험 환경에 따라 달라질 수 있다. 이 표는 LB-LIOSAM의 성능을 평가하기 위한 정량적 맥락을 제공하는 것을 목표로 한다.*

------



## 4.  개발자를 위한 구현 및 재현 가이드





### 4.1  시스템 구성 요소의 가용성



LB-LIOSAM을 직접 사용하거나 연구에 활용하고자 하는 개발자 및 연구자에게 가장 중요한 정보는 소스 코드의 가용성이다. 현재까지의 정보에 따르면, **LB-LIOSAM**의 통합된 전체 시스템은 단일 패키지 형태로 공개되어 있지 않다. 주요 연구 논문 6에서도 소스 코드 저장소에 대한 언급을 찾아볼 수 없다.

하지만 희소식은 LB-LIOSAM을 구성하는 핵심적인 개별 구성 요소들이 모두 오픈 소스로 공개되어 있다는 점이다. 기반이 되는 프레임워크인 **LIO-SAM**은 GitHub를 통해 널리 배포되고 있으며 활발하게 유지보수되고 있다.5 마찬가지로, LB-LIOSAM의 성능 향상을 책임지는 두 핵심 모듈, 즉 프론트엔드 특징 추출기인 

**LinK3D**와 백엔드 폐루프 검출기인 **BoW3D** 역시 각각의 GitHub 저장소를 통해 소스 코드가 공개되어 있다.14

따라서 개발자는 LB-LIOSAM을 '다운로드'하여 사용할 수는 없지만, 공개된 구성 요소들을 직접 '통합'하여 시스템을 재현할 수 있는 가능성을 가지고 있다.



### 4.2  구성 요소의 의존성 및 빌드 지침



LB-LIOSAM을 성공적으로 재현하기 위해서는 각 구성 요소의 의존성을 충족시키고 올바른 순서로 빌드해야 한다. 각 저장소의 `README` 파일을 종합하여 필요한 사항들을 정리하면 다음과 같다.14



#### 4.2.1 통합 의존성 목록



- **ROS (Robot Operating System):** Kinetic, Melodic, 또는 Noetic 버전. 각 구성 요소가 테스트된 버전이 다를 수 있으므로 호환성 확인이 필요하다.5
- **PCL (Point Cloud Library):** 버전 1.7 이상이 요구된다.14
- **OpenCV:** BoW3D 및 LinK3D에서 이미지 처리 관련 기능에 사용될 수 있다.14
- **Eigen 3:** 행렬 및 벡터 연산을 위한 필수 라이브러리이다.14
- **gtsam (Georgia Tech Smoothing and Mapping):** LIO-SAM의 요인 그래프 최적화를 위한 핵심 라이브러리이다. 버전 4.0 이상이 필요하며, PPA를 통해 설치하는 것이 권장된다.5



#### 4.2.2 빌드 절차



1. **catkin 작업 공간 설정:** `catkin_ws`와 같은 ROS 작업 공간을 생성하고 `src` 디렉토리로 이동한다.

2. **LIO-SAM 클론 및 빌드:**

   Bash

   ```
   cd ~/catkin_ws/src
   git clone https://github.com/TixiaoShan/LIO-SAM.git
   ```

3. **LinK3D 클론:**

   Bash

   ```
   cd ~/catkin_ws/src
   git clone https://github.com/YungeCui/LinK3D.git
   ```

4. **BoW3D 클론:**

   Bash

   ```
   cd ~/catkin_ws/src
   git clone https://github.com/YungeCui/BoW3D.git
   ```

5. **전체 빌드:** 작업 공간의 루트로 돌아가 `catkin_make`를 실행한다.

   Bash

   ```
   cd ~/catkin_ws
   catkin_make
   ```

   *빌드 시 C++ 버전 충돌이 발생하면, 각 패키지의 `CMakeLists.txt` 파일에서 `CMAKE_CXX_STANDARD`를 조정해야 할 수 있다*.14



#### 4.2.3 예제 데이터셋



각 구성 요소 저장소는 알고리즘을 테스트하고 이해하는 데 도움이 되는 예제 데이터셋을 언급하고 있다. 개발자는 이를 활용하여 각 모듈의 독립적인 작동을 먼저 확인할 수 있다.

- **LIO-SAM:** 자체적으로 샘플 데이터셋을 제공한다.5
- **LinK3D & BoW3D:** KITTI Odometry 데이터셋, M2DGR 데이터셋, Stevens-VLP16-Dataset 등을 예제로 사용한다.14



### 4.3  LB-LIOSAM 재현을 위한 개념적 로드맵



완성된 코드가 없기 때문에, LB-LIOSAM을 사용하려는 개발자는 사실상 시스템을 직접 재창조해야 한다. 이는 단순한 작업이 아니며, 그 자체로 하나의 소규모 연구 프로젝트에 해당한다. 이 과정은 상당한 엔지니어링 노력을 요구하지만, 동시에 시스템의 내부 작동 원리를 깊이 이해할 수 있는 기회를 제공한다. 아래는 이 통합 과정을 위한 논리적인 작업 흐름을 개념적으로 제시한 로드맵이다.

1. **기반 시스템 구축:** 먼저, LIO-SAM을 독립적으로 빌드하고 제공된 샘플 데이터로 실행하여 작업 환경이 올바르게 설정되었는지 확인한다.5 이는 모든 통합 작업의 안정적인 출발점이 된다.
2. **프론트엔드 교체 (LinK3D 통합):**
   - LIO-SAM 코드베이스에서 특징 추출이 수행되는 핵심 부분을 식별한다. 이는 주로 `imageProjection.cpp` 파일 내에 위치한다.
   - 기존의 모서리/평면 특징 추출 로직을 비활성화하고, 그 자리에 LinK3D 라이브러리를 호출하는 코드를 삽입한다. 입력으로는 IMU에 의해 왜곡이 보정된 포인트 클라우드를 사용해야 한다.
   - LinK3D를 통해 얻은 특징점 및 매칭 결과를 LIO-SAM의 후속 모듈, 특히 `mapOptimization.cpp`가 요구하는 데이터 형식으로 변환해야 한다. 이는 LiDAR 주행 거리 측정 요인을 생성하는 데 필요한 입력 데이터를 제공하는 과정이다.
3. **백엔드 통합 (BoW3D 통합):**
   - `mapOptimization.cpp` 모듈 내에서 새로운 키프레임이 생성되고 요인 그래프에 추가되는 지점을 찾는다.
   - 이 지점에서 새로운 스레드를 생성하거나 함수를 호출하여 BoW3D 라이브러리와 상호작용하도록 구현한다. 새로 생성된 키프레임과 해당 프레임의 LinK3D 특징들을 BoW3D에 전달하여 폐루프 검출을 요청한다.
4. **요인 추가 (그래프 최적화 연결):**
   - BoW3D가 성공적으로 폐루프를 검출하면, 두 키프레임(현재와 과거)의 인덱스와 그 사이의 6자유도 상대 변환(relative pose)을 반환할 것이다.
   - 이 정보를 사용하여 GTSAM의 `BetweenFactor`를 생성한다. 이 팩터는 두 포즈 노드를 직접 연결하는 강력한 제약 조건이 된다.
   - 생성된 `BetweenFactor`를 LIO-SAM의 주 요인 그래프에 추가한다. 이 과정은 6에서 설명된 "폐루프 요인"을 그래프에 추가하는 것을 직접적으로 구현하는 것이다. 이 제약이 추가되면 GTSAM의 iSAM2 옵티마이저가 자동으로 그래프 전체를 재최적화하여 누적된 오차를 보정한다.

이 로드맵은 LB-LIOSAM을 성공적으로 재현하기 위한 청사진을 제공하며, 이 연구를 기반으로 추가적인 개선이나 실험을 수행하고자 하는 연구자들에게 귀중한 지침이 될 것이다.

------



## 5.  비판적 평가 및 향후 연구 방향





### 5.1 기여 요약



LB-LIOSAM의 핵심적인 기여는 독창적인 알고리즘의 발명보다는, 기존의 강력한 시스템을 지능적으로 개선하는 방법에 대한 성공적인 사례 연구를 제시한 데 있다. 이 연구는 널리 사용되는 주요 SLAM 프레임워크(LIO-SAM)의 가장 취약한 구성 요소를 식별하고, 이를 전문화된 고성능 오픈 소스 모듈로 전략적으로 교체함으로써 시스템의 전반적인 강건성을 향상시키는 효과적인 방법을 입증했다.6

구체적으로, LB-LIOSAM은 LIO-SAM이 특징이 부족하거나 반복적인 대규모 환경에서 겪는 누적 오차 문제를 해결하기 위해, 프론트엔드의 특징 추출을 LinK3D로, 백엔드의 폐루프 검출을 BoW3D로 대체했다. 이는 복잡한 로봇 시스템의 모듈식 진화라는 측면에서 훌륭한 접근법을 보여준다. 즉, 전체 시스템을 처음부터 다시 설계하는 대신, 검증된 기반 위에서 가장 큰 성능 병목을 일으키는 부분을 집중적으로 개선하여 효율적으로 전체 성능을 끌어올린 것이다. 이러한 접근 방식은 로봇 공학 커뮤니티에 실용적이고 재현 가능한 개선 방법론을 제공한다는 점에서 중요한 가치를 가진다.



### 5.1  식별된 한계 및 비평



LB-LIOSAM은 이론적 타당성과 잠재력에도 불구하고, 연구 결과물로서 몇 가지 명백한 한계를 가지고 있다. 이러한 한계는 이 연구의 영향력을 제한하고 후속 연구를 어렵게 만드는 요인으로 작용한다.

- **오픈 소스 릴리스의 부재:** 가장 큰 한계는 통합된 전체 시스템의 소스 코드가 공개되지 않았다는 점이다. 이는 연구의 핵심 원칙인 재현성(reproducibility)과 검증 가능성(verifiability)을 심각하게 저해한다. 커뮤니티가 코드를 직접 분석, 사용, 개선할 수 없으므로 알고리즘의 채택과 확산이 제한되고 논문의 영향력이 감소한다.
- **불충분한 성능 검증:** 제3장에서 상세히 분석했듯이, 이 연구는 성능 평가를 질적인 설명과 비공개 데이터셋에 의존하고 있다.6 ATE, RPE와 같은 표준화된 정량적 지표가 부재하여 "우수성"이라는 주장은 객관적인 근거가 부족하다. 이는 다른 최신 알고리즘과의 공정한 비교를 불가능하게 만들어, 제안된 방법론의 실질적인 개선 정도를 파악하기 어렵게 한다.
- **구성 요소의 독창성 부족:** LB-LIOSAM의 독창성은 새로운 알고리즘의 개발이 아닌, 기존 알고리즘들의 '통합'에 있다. LinK3D와 BoW3D는 모두 다른 연구 그룹에 의해 개발된 기존의 알고리즘이다.14 이를 결함으로 보기보다는, 이 연구가 시스템 통합 연구(systems integration research)라는 특성을 가지고 있음을 명확히 이해해야 한다. 즉, 기여는 새로운 이론의 제시가 아니라 기존 기술들을 조합하여 새로운 가치를 창출한 데 있다.



### 5.2  제안되는 향후 연구



LB-LIOSAM이 제시한 가능성을 완전히 실현하고 그 한계를 극복하기 위해, 로봇 공학 커뮤니티는 다음과 같은 구체적인 후속 연구를 진행할 수 있다.

1. **공식적인 벤치마킹:** 가장 시급하고 중요한 다음 단계는 제4.3절에서 제시된 로드맵에 따라 LB-LIOSAM을 재현하고, 이를 공개 데이터셋(예: KITTI, M2DGR, UrbanNav, MulRan 등)에서 엄격하게 평가하는 것이다. 이 벤치마킹은 LIO-SAM 원본, LVI-SAM, FAST-LIO 등 다른 최신 시스템들과의 직접적인 정량적 비교를 포함해야 하며, ATE와 RPE 같은 표준 지표를 사용하여 성능을 객관적으로 측정해야 한다.
2. **공개 릴리스:** LB-LIOSAM을 성공적으로 재현한 연구자는 통합된 코드를 커뮤니티에 공개하여 추가적인 개발과 검증, 그리고 교육적 활용을 촉진해야 한다. 이는 이 연구의 장기적인 영향력을 확보하는 데 결정적일 것이다.
3. **다른 양상과의 융합:** 한 가지 흥미로운 연구 방향은 LB-LIOSAM의 강건한 LiDAR 프론트엔드 및 백엔드를 LVI-SAM의 시각-관성 시스템과 결합하는 것이다. "LB-LVI-SAM"과 같은 하이브리드 시스템은 기하학적 특징이 부족한 환경(LB-LIOSAM의 강점)과 시각적 텍스처가 부족한 환경(LVI-SAM의 강점) 모두에서 탁월한 강건성을 보일 수 있는 잠재력을 가진다.
4. **도전적인 시나리오에서의 응용:** 재현된 시스템을 이론적으로 가장 큰 이점을 보일 수 있는 실제 환경에서 테스트하는 것이 중요하다. 예를 들어, 대규모 물류 창고 7나 반복적인 구조가 많은 도심 협곡(urban canyon)에서의 장기 자율 주행 시나리오에 적용하여, 이론적 이점이 실제 환경에서 어떻게 발현되는지 경험적으로 검증해야 한다. 이는 알고리즘의 실용성을 입증하고 상업적 또는 산업적 적용 가능성을 타진하는 데 기여할 것이다.

#### **참고 자료**

1. Performance Analysis of LIO-SAM - Ronia Peterson, accessed July 30, 2025, http://www.roniapeterson.com/LIO-SAM.pdf
2. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping - arXiv, accessed July 30, 2025, https://arxiv.org/abs/2007.00258
3. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and ..., accessed July 30, 2025, http://arxiv.org/pdf/2007.00258
4. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping - Senseable City Lab :.:: Massachusetts Institute of Technology, accessed July 30, 2025, https://senseable.mit.edu/papers/pdf/20201020_Shan-etal_LIO-SAM_IROS.pdf
5. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping - GitHub, accessed July 30, 2025, https://github.com/TixiaoShan/LIO-SAM
6. LB-LIOSAM: an improved mapping and localization method with ..., accessed July 30, 2025, https://www.emerald.com/insight/content/doi/10.1108/ir-07-2024-0314/full/html
7. SLAM for Automation Experts - Number Analytics, accessed July 30, 2025, https://www.numberanalytics.com/blog/slam-for-automation-experts
8. Efficiently Closing Loops in LiDAR-Based SLAM Using Point Cloud Density Maps - arXiv, accessed July 30, 2025, https://arxiv.org/html/2501.07399v1
9. When-to-Loop: Enhanced Loop Closure for LiDAR SLAM in Urban Environments Based on SCAN CONTEXT - MDPI, accessed July 30, 2025, https://www.mdpi.com/2072-666X/15/10/1212
10. LIDAR-based SLAM system for autonomous vehicles in degraded point cloud scenarios: dynamic obstacle removal | Emerald Insight, accessed July 30, 2025, https://www.emerald.com/insight/content/doi/10.1108/ir-01-2024-0001/full/pdf?title=lidar-based-slam-system-for-autonomous-vehicles-in-degraded-point-cloud-scenarios-dynamic-obstacle-removal
11. LIO-SAM/src/mapOptmization.cpp at master - GitHub, accessed July 30, 2025, https://github.com/TixiaoShan/LIO-SAM/blob/master/src/mapOptmization.cpp
12. LVI-SAM: Tightly-coupled Lidar-Visual-Inertial Odometry via Smoothing and Mapping - GitHub, accessed July 30, 2025, https://github.com/TixiaoShan/LVI-SAM
13. a SLAM implementation combining FAST-LIO2 with pose graph optimization and loop closing based on LIO-SAM paper - GitHub, accessed July 30, 2025, https://github.com/engcang/FAST-LIO-SAM
14. YungeCui/LinK3D: [RA-L] LinK3D: Linear Keypoint ... - GitHub, accessed July 30, 2025, https://github.com/YungeCui/LinK3D
15. LinK3D: Linear Keypoints Representation for 3D LiDAR Point Cloud - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/377455350_LinK3D_Linear_Keypoints_Representation_for_3D_LiDAR_Point_Cloud
16. BoW3D: Bag of Words for Real-Time Loop Closing in 3D LiDAR SLAM - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/365310694_BoW3D_Bag_of_Words_for_Real-Time_Loop_Closing_in_3D_LiDAR_SLAM
17. (PDF) BoW3D: Bag of Words for Real-time Loop Closing in 3D LiDAR SLAM, accessed July 30, 2025, https://www.researchgate.net/publication/362728568_BoW3D_Bag_of_Words_for_Real-time_Loop_Closing_in_3D_LiDAR_SLAM
18. YungeCui/BoW3D: [RA-L] BoW3D: Bag of Words for Real ... - GitHub, accessed July 30, 2025, https://github.com/YungeCui/BoW3D
19. LiDAR Inertial Odometry Based on Indexed Point and Delayed Removal Strategy in Highly Dynamic Environments - PMC, accessed July 30, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10255994/
20. A Benchmark for Multi-Modal LiDAR SLAM with Ground Truth in GNSS-Denied Environments - MDPI, accessed July 30, 2025, https://www.mdpi.com/2072-4292/15/13/3314
21. MIT Open Access Articles LVI-SAM: Tightly-coupled Lidar-Visual- Inertial Odometry via Smoothing and Mapping, accessed July 30, 2025, https://dspace.mit.edu/bitstream/handle/1721.1/144047/2104.10831.pdf?sequence=2&isAllowed=y
22. SLAM technology improves AMR autonomy - Design World, accessed July 30, 2025, https://www.designworldonline.com/slam-technology-improves-amr-autonomy/

