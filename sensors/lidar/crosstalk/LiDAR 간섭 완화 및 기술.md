---
layout: page
title: LiDAR 간섭 완화 및 기술 동향
permalink: /sensors/lidar/crosstalk/LiDAR 간섭 완화 및 기술
---



센서가 밀집된 환경이 보편화되면서, 특히 자율주행 자동차, 로보틱스, 스마트 인프라 분야에서 다중 LiDAR(Light Detection and Ranging) 시스템 간의 상호 간섭은 더 이상 무시할 수 없는 핵심적인 기술적 과제로 부상했다. 이 보고서는 2024년과 2025년의 최신 기술 동향을 바탕으로 LiDAR 간섭의 근본 원인, 다양한 완화 전략, 그리고 시장의 경쟁 구도를 심층적으로 분석한다.

본 분석의 핵심 발견 사항은 다음과 같다. 첫째, LiDAR 간섭은 공간적, 스펙트럼적, 시간적 정렬이라는 세 가지 조건이 동시에 충족될 때 발생하는 예측 가능한 물리적 현상이다. 이로 인해 신호 대 잡음비(SNR) 저하, 고스트 포인트(허상) 생성 등 심각한 데이터 품질 저하가 발생하며, 이는 안전에 직결되는 시스템의 신뢰도를 위협한다.

둘째, 간섭 완화 기술은 크게 세 가지 패러다임으로 발전하고 있다. **하드웨어 중심 전략**은 FMCW(주파수 변조 연속파), RMCW(랜덤 변조 연속파)와 같은 고유한 파형 설계를 통해 간섭에 대한 내성을 근본적으로 높이는 방식이다. **프로토콜 기반 접근법**은 TDMA(시분할 다중 접속)나 CDMA(코드분할 다중 접속)와 같이 통신 기술에서 차용한 원리를 적용하여 신호 충돌을 회피한다. 마지막으로 **소프트웨어 및 AI 기반 솔루션**은 이미 오염된 데이터를 후처리하여 간섭을 탐지, 제거, 나아가 복구하는 역할을 한다.

셋째, 시장은 단일 기술로 간섭을 완벽히 '회피'하려는 시도에서 벗어나, 하드웨어의 내재적 강건성과 AI 기반 소프트웨어의 복원력을 결합한 하이브리드 '간섭 관리' 시스템으로 명확히 이동하고 있다. 이는 더 이상 센서의 하드웨어 사양만으로 경쟁하는 시대가 끝나고, 전체 인식 스택의 지능과 복원력이 핵심 경쟁력으로 부상했음을 의미한다.

본 보고서는 이러한 분석을 바탕으로 기술 아키텍트와 투자 전략가에게 다음과 같은 핵심 권고 사항을 제시한다. 시스템 아키텍트는 단일 솔루션에 의존하지 말고, 하드웨어, 프로토콜, 소프트웨어를 아우르는 다층적 방어 전략을 채택해야 한다. 특히 나노초 수준의 정밀 동기화를 제공하는 PTP/gPTP 네트워크는 이제 안전에 필수적인 핵심 유틸리티로 설계되어야 한다. 기술 투자자 및 전략가는 개별 LiDAR의 하드웨어 성능을 넘어, 기업의 소프트웨어 역량, 특히 AI 기반 복구 기술과 지적 재산(IP) 포트폴리오의 강점을 평가의 핵심 기준으로 삼아야 한다. 향후 LiDAR 시장의 승자는 가장 정교한 하드웨어를 만드는 기업이 아니라, 불확실성을 가장 지능적으로 관리하는 시스템을 구축하는 기업이 될 것이다.

------



LiDAR 기술의 확산은 자율주행차, 로보틱스, 드론 등 다양한 산업 분야에서 전례 없는 수준의 환경 인식 능력을 제공했다. 그러나 이러한 확산은 동시에 피할 수 없는 기술적 난제를 낳았는데, 바로 다중 LiDAR 시스템 간의 상호 간섭, 즉 누화(Crosstalk) 문제다. 간섭의 근본 원인은 LiDAR의 기본 작동 원리, 즉 빛을 방출하고 반사된 신호를 측정하는 행위 자체에 있다.1 자율주행차가 보편화되고, 각 차량에 여러 개의 LiDAR 센서가 장착되며, 로봇과 드론이 도심을 활보하는 미래 환경은 수많은 비협조적인 광원(LiDAR)이 공존하는 '빛의 포화 상태'가 될 것이다.3

이 문제는 더 이상 이론적인 가능성이 아니다. 실제 도로 환경에서 다수의 LiDAR가 동시에 작동할 때, 다른 차량의 센서에서 방출된 레이저 펄스가 자신의 수신기로 들어와 심각한 측정 오류를 유발하는 현상이 관찰되고 있다.5 연구에 따르면, 펄스당 10-20개의 광자라는 비교적 낮은 수준의 간섭만으로도 거리 측정 정확도가 수 센티미터까지 저하될 수 있으며, 여러 소스로부터의 강한 간섭이나 직사광선은 센서를 일시적으로 완전히 마비시킬 수도 있다.6 이는 LiDAR가 다른 차량 신호의 간섭으로 인해 레이더가 오작동하는 상황에서도 신뢰할 수 있는 데이터를 제공해야 한다는 본연의 역할에 정면으로 배치된다.7 따라서 다가올 센서 밀집 환경에서 LiDAR 시스템의 안전성과 신뢰성을 보장하기 위해서는 간섭 문제에 대한 깊이 있는 이해와 해결이 필수 불가결하다.


LiDAR 간섭은 무작위적인 노이즈가 아니라, 세 가지 특정 조건이 동시에 충족될 때 발생하는 예측 가능한 물리적 현상이다. 이 '3대 정렬(Three Alignments)' 모델을 이해하는 것은 효과적인 완화 전략을 수립하는 데 있어 가장 기본적인 출발점이다.3

- **공간적 정렬 (Spatial Alignment):** 간섭이 발생하기 위한 가장 기본적인 조건이다. 이는 간섭원의 레이저 빔이 피해자 LiDAR의 수신기 시야(Field-of-View) 내로 직접 들어오거나, 혹은 두 LiDAR가 동일한 물체를 조준하여 간섭원의 반사광이 피해자 수신기에 도달하는 경우를 포함한다.3 LiDAR는 레이더에 비해 빔 발산각이 매우 좁지만, 도로 위 수많은 차량에 장착된 센서들을 고려할 때 공간적 중첩은 충분히 발생할 수 있다.3
- **스펙트럼적 정렬 (Spectral Alignment):** 두 LiDAR 시스템이 동일하거나 매우 유사한 파장 대역에서 작동할 때 발생한다. 간섭원의 레이저 파장이 피해자 LiDAR의 광학 대역 통과 필터(Optical Band-pass Filter) 범위를 통과해야만 신호로 인식될 수 있다. 만약 두 시스템이 동일한 중심 파장과 변조 파라미터(예: FMCW 시스템의 처프(chirp) 특성)를 사용한다면, 피해자 수신기는 자신의 신호와 간섭 신호를 구분하기 매우 어려워진다.3
- **시간적 정렬 (Temporal Alignment):** 피해자 LiDAR의 수신기가 신호를 수신하기 위해 열려 있는 시간, 즉 '거리 게이트(Range Gate)' 내에 간섭 신호가 정확히 도달해야 함을 의미한다.3 ToF(Time-of-Flight) 시스템에서 이는 간섭 펄스가 예상 반사 신호의 도착 시간과 겹치는 것을 의미하며, FMCW 시스템에서는 간섭 신호의 비트 주파수가 피해자 시스템의 측정 대역 내에 들어오는 것을 뜻한다.

이 세 가지 정렬 조건이 모두 충족되고 간섭 신호가 충분한 에너지를 가지고 수신기에 도달할 때, 비로소 간섭 현상이 발생한다. 이는 간섭 완화 전략이 이 세 가지 정렬 조건 중 하나 이상을 깨뜨리는 방향으로 설계되어야 함을 시사한다. 즉, 완벽한 간섭 '제거'보다는 세 가지 정렬의 동시 발생 '확률'을 안전 기준치 이하로 낮추는 것이 보다 현실적이고 전략적인 목표가 된다.


LiDAR 간섭이 시스템에 미치는 영향은 단순히 약간의 노이즈가 추가되는 수준을 넘어선다. 그 영향은 크게 두 가지 치명적인 형태로 나타나며, 이는 자율주행 시스템의 안전성에 직접적인 위협이 된다.

첫 번째는 **신호 대 잡음비(Signal-to-Noise Ratio, SNR) 저하**다. 간섭 신호는 시스템의 배경 잡음 레벨을 크게 높인다. 이로 인해 멀리 있거나 반사율이 낮은 실제 물체(예: 어두운 색의 차량, 보행자)로부터 돌아오는 미약한 신호가 높아진 잡음 바닥에 묻혀버릴 수 있다.3 결과적으로 시스템의 최대 탐지 거리가 줄어들고, 중요한 장애물을 인지하지 못하는 '탐지 실패(Missed Detection)'로 이어질 수 있다. FMCW 시스템에서도 간섭은 잡음 플로어를 증가시켜 약한 목표물을 가리고 탐지 확률을 감소시킨다.10

두 번째이자 더 위험한 영향은 **허상(Ghost Points) 또는 팬텀 장애물(Phantom Obstacles)의 생성**이다. 이는 시스템이 간섭 펄스를 자신의 펄스에 대한 유효한 반사 신호로 오인하여, 실제로는 존재하지 않는 위치에 물체가 있는 것처럼 포인트를 생성하는 현상이다.3 이러한 허상 포인트들은 포인트 클라우드 데이터에 무작위로 흩어져 나타나며, 심각한 경우 이들이 모여 실제 장애물처럼 보이는 '팬텀 장애물'을 형성할 수 있다.1 자율주행 시스템이 이러한 팬텀 장애물에 반응하여 불필요한 급제동을 하거나, 반대로 허상 포인트들로 인해 실제 장애물의 형상이 왜곡되어 이를 제대로 인식하지 못하는 '오인(Misclassification)'을 유발할 수 있다. 이는 반드시 인식해야 할 사람, 자전거, 다른 차량 등을 보이지 않게 만드는 최악의 시나리오로 이어질 수 있어, LiDAR 간섭 문제가 반드시 해결되어야 하는 이유를 명확히 보여준다.1

------



간섭 문제에 대한 가장 근본적인 해결책 중 하나는 LiDAR 센서 자체의 신호, 즉 파형(waveform)을 정교하게 설계하는 것이다. 이 분야에서 가장 주목받는 기술은 주파수 변조 연속파(FMCW)와 그 발전형인 랜덤 변조 연속파(RMCW)다.

**FMCW (Frequency-Modulated Continuous-Wave):** 전통적인 펄스형 ToF LiDAR가 짧은 펄스를 사용하는 것과 달리, FMCW LiDAR는 시간에 따라 주파수가 선형적으로 변하는 연속적인 파(chirp)를 방출한다.9 수신된 신호와 기준 신호의 주파수 차이(비트 주파수)를 측정하여 거리를 계산하는 이 방식은 몇 가지 내재적인 간섭 저항성을 가진다. 지속적으로 주파수가 변하기 때문에, 외부의 단일 주파수 간섭 신호는 특정 비트 주파수에 고정되지 않고 넓게 퍼져나가 잡음처럼 처리될 수 있다.3 그러나 이러한 장점에도 불구하고 FMCW는 완벽한 해결책이 아니다. 만약 다른 FMCW LiDAR가 유사한 시작 주파수와 처프 기울기(chirp slope)를 가지고 동시에 작동한다면, 스펙트럼적, 시간적 정렬이 발생하여 심각한 간섭을 유발할 수 있다.3

**RMCW (Random-Modulated Continuous-Wave):** 이러한 FMCW의 한계를 극복하기 위해 등장한 것이 RMCW다. RMCW는 선형적인 주파수 변조 대신, 의사 난수 이진 시퀀스(PRBS, Pseudo-Random Binary Sequence)를 사용하여 광파의 위상이나 진폭을 무작위로 변조한다.12 이 시스템의 핵심 원리는 PRBS가 자기 자신과만 높은 상관관계(autocorrelation)를 갖고, 다른 신호(다른 LiDAR의 신호, 햇빛 등)와는 매우 낮은 상호상관관계(cross-correlation)를 갖는다는 점이다.12 수신기는 반사된 신호에서 PRBS를 재구성하고, 이를 원본 PRBS와 비교하여 시간 지연(거리)을 정확히 찾아낸다. 다른 LiDAR에서 온 신호는 원본 PRBS와 상관관계가 없으므로 효과적으로 걸러져, 매우 높은 수준의 간섭 면역성을 제공한다.12

**True-Random LiDAR:** RMCW에서 한 걸음 더 나아간 개념은 '진정한 무작위(True-Random)' 신호를 사용하는 것이다. 이 방식은 PRBS와 같은 디지털적으로 생성된 시퀀스가 아닌, LED의 자발적 방출(spontaneous emission)과 같은 물리적 현상에서 발생하는 예측 불가능한 아날로그 랜덤 신호를 직접 광원으로 사용한다.1 두 개 이상의 센서가 통계적으로 동일한 무작위 신호를 생성할 확률은 사실상 0에 가깝기 때문에, 이는 이론적으로 가장 완벽한 간섭 저항성을 제공한다. 또한, 고속의 디지털 신호 처리 없이 아날로그 영역에서 상관관계 검출을 수행할 수 있어 잠재적인 비용 및 처리 속도 이점을 가진다.2

이러한 파형 공학의 발전은 간섭 문제를 해결하기 위한 패러다임의 전환을 보여준다. 단순하고 예측 가능한 신호에서 복잡하고 예측 불가능한 신호로의 이동은, 간섭 신호가 우연히라도 스펙트럼적, 시간적으로 정렬될 확률 자체를 근본적으로 낮추는 전략이다. 이는 더 이상 간섭을 피하는 수준이 아니라, 신호 자체에 고유한 '지문'을 부여하여 능동적으로 구별해내는 방식으로, 기술적 해자(moat)를 구축하는 핵심 전략이 되고 있다.


LiDAR 시스템의 성능과 간섭 저항성에 영향을 미치는 또 다른 핵심 하드웨어 요소는 사용하는 레이저의 파장(wavelength)이다. 현재 시장은 주로 905nm와 1550nm 파장으로 양분되어 있으며, 각각 명확한 장단점을 가진다.

**905nm 파장:** 현재 가장 널리 사용되는 파장대로, 가장 큰 장점은 **비용 효율성**이다. 905nm 파장은 성숙한 기술인 실리콘(Silicon) 기반 광검출기(photodetector)로 수신할 수 있으며, 이 검출기는 대량 생산을 통해 가격이 매우 저렴하다.13 이로 인해 905nm LiDAR는 전체 시스템 비용을 낮추는 데 유리하여 대중적인 채택을 이끌고 있다. 하지만 두 가지 주요한 단점이 존재한다. 첫째, 905nm는 태양광 스펙트럼에서 복사 조도(solar irradiance)가 상대적으로 높은 영역에 속해 있어, 낮 시간 동안 태양광에 의한 배경 잡음(background noise)의 영향을 더 많이 받는다.15 둘째, 인체, 특히 눈의 망막에 상대적으로 덜 안전하기 때문에 국제 안전 표준(eye-safety regulations)에 따라 방출할 수 있는 레이저의 출력이 제한된다. 이 출력 제한은 곧 센서의 최대 탐지 거리를 제약하는 요인으로 작용한다.13

**1550nm 파장:** 1550nm 파장은 **성능과 안전성**에서 강점을 보인다. 이 파장대의 빛은 사람의 눈에서 각막과 수정체에 대부분 흡수되어 망막에 도달하지 않기 때문에 '눈에 안전한(eye-safe)' 파장으로 분류된다. 이 덕분에 905nm 대비 훨씬 높은 출력(일부 자료에 따르면 최대 40배)의 레이저를 사용할 수 있으며, 이는 곧 더 긴 탐지 거리와 안개, 먼지 등 악천후 조건에서의 우수한 투과 성능으로 이어진다.13 또한, 1550nm 대역은 태양광 복사 조도가 905nm 대역보다 현저히 낮아, 태양광으로 인한 배경 잡음이 적다는 근본적인 SNR 이점을 가진다.15 간섭의 관점에서 보면, 1550nm를 사용하는 것만으로도 시장의 주류인 905nm LiDAR들과의 스펙트럼적 분리가 이루어져 자연스러운 간섭 회피 효과를 얻는다.15 주요 단점은 인듐-갈륨-비소(InGaAs) 기반의 고가 검출기를 사용해야 한다는 점이었으나, 기술 발전으로 인해 실리콘 검출기와의 가격 격차는 점차 줄어들고 있다.13 Luminar와 같은 기업들이 이 파장대를 주력으로 내세우고 있다.18

결론적으로, 파장 선택은 비용과 성능 간의 전략적 트레이드오프다. 905nm는 비용에 민감한 대중 시장을 공략하는 데 유리한 반면, 1550nm는 장거리 탐지와 열악한 환경에서의 신뢰성이 요구되는 고성능 자율주행 시스템에 더 적합한 솔루션으로 자리매김하고 있다.


FMCW와 RMCW 외에도, 간섭 완화와 성능 향상을 목표로 하는 새로운 파형 공학 기술들이 지속적으로 연구되고 있다. 이 중 위상 변조 연속파(PhMCW)와 코드분할 다중 접속(CDMA) LiDAR, 그리고 혼돈(Chaotic) LiDAR가 주목할 만한 차세대 기술이다.

**PhMCW (Phase-Modulated Continuous-Wave):** FMCW가 주파수를 변조하는 것과 달리, PhMCW는 연속파 레이저의 **위상(phase)**을 변조하여 거리 정보를 인코딩한다. 주로 PRBS와 같은 코드를 사용하여 위상을 변조하며, 수신된 신호와의 위상차를 분석해 ToF를 계산한다.19 PhMCW는 FMCW에 비해 몇 가지 잠재적 이점을 가진다. 일부 연구에 따르면 장거리 직접 탐지 시나리오에서 FMCW 대비 3dB의 SNR 이점을 가질 수 있으며, 주파수 선형성 문제에서 비교적 자유로워 FMCW의 복잡한 레이저 소스 및 변조 회로 없이도 구현이 가능하다. 이는 비용 효율적이면서도 실시간 성능이 중요한 애플리케이션에 매력적인 대안이 될 수 있다.19

**CDMA (Code-Division Multiple Access) LiDAR:** 통신 기술인 CDMA 원리를 LiDAR에 직접 적용한 방식이다. 이 시스템은 방출하는 각각의 레이저 펄스를 고유한 코드 시퀀스(예: Optical Orthogonal Codes)로 인코딩한다.1 수신기는 자신에게 할당된 코드와 일치하는 신호를 찾기 위해 정합 필터(matched filter)를 사용한다. 자신의 센서에서 반사된 신호는 필터를 통과하며 높은 상관관계 피크(autocorrelation peak)를 생성하여 탐지되는 반면, 다른 LiDAR에서 온 간섭 신호는 코드가 다르기 때문에 낮은 상호상관관계 값만 생성하여 효과적으로 걸러진다.2 이 방식은 강력한 간섭 저항성뿐만 아니라, 각 센서에 고유한 'ID'를 부여하여 어떤 신호가 어느 센서에서 왔는지 식별할 수 있게 해주는 부가적인 장점도 제공한다. 더 나아가 파장 도약(wavelength-hopping)과 시간 확산(time-spreading)을 결합한 2차원 OCDMA 기술은 훨씬 더 많은 수의 센서를 지원할 수 있는 잠재력을 보여준다.20

**Chaotic LiDAR:** 예측 불가능하고 비주기적인 아날로그 신호를 사용한다는 점에서 True-Random LiDAR와 유사한 철학을 공유한다. Chaotic LiDAR는 레이저 자체를 혼돈(chaos) 상태로 만들어 매우 광대역의 복잡한 신호를 생성한다.1 이 아날로그 방식은 RMCW나 CDMA에서 요구되는 고속 디지털 신호 처리 과정을 피할 수 있다는 장점이 있다. 하지만 레이저의 혼돈 상태를 실제 주행 환경과 같은 다양한 외부 조건 하에서 안정적으로 유지하는 것이 기술적으로 매우 어려워, 아직 상용화에는 한계가 있다.1

이러한 신기술들은 파형의 복잡성과 고유성을 극대화하여 간섭 신호와의 스펙트럼 및 시간적 정렬 확률을 최소화하려는 공통된 목표를 가지고 있다. 이는 하드웨어 수준에서 간섭 문제에 대한 보다 근본적인 해결책을 찾으려는 업계의 지속적인 노력을 반영한다.

------



하드웨어 파형 설계와 더불어, 여러 LiDAR 센서가 공유된 공간이라는 '채널'을 어떻게 효율적으로 나누어 사용할 것인지에 대한 프로토콜 수준의 접근법도 활발히 연구되고 있다. 그중 가장 직관적이고 확실한 방법이 시분할 다중 접속(TDMA, Time-Division Multiple Access)이다.

**원리:** TDMA의 핵심 원리는 '시간'을 나누는 것이다. 사용 가능한 시간을 아주 작은 시간 슬롯(time slot)으로 분할하고, 각 LiDAR 센서에 고유한 전송 슬롯을 할당한다.21 한 센서가 자신에게 할당된 시간 슬롯 동안 레이저를 방출하고 신호를 수신하는 동안, 다른 모든 센서는 침묵을 지킨다. 이처럼 각 센서의 작동 시간을 물리적으로 분리함으로써, 두 개 이상의 센서가 동시에 신호를 방출하여 발생하는 상호 간섭을 원천적으로 차단한다.22 이는 통신에서 여러 사용자가 동일한 주파수 채널을 간섭 없이 공유하기 위해 사용하는 원리와 동일하다.21

**구현:** TDMA 시스템을 구현하기 위해서는 모든 센서 간의 정밀한 시간 동기화가 필수적이다. 이는 중앙의 마스터 동기화 장치가 각 센서에 전송 시작을 명령하는 방식으로 구현될 수도 있고, 보다 진보된 시스템에서는 각 센서가 자율적으로 동기화하는 방식을 사용하기도 한다. 예를 들어, 각 카메라(또는 LiDAR)에 포토다이오드를 추가하여 다른 센서의 조명 활동을 감지하고, 채널이 비어 있을 때만 작동하도록 스스로 타이밍을 조절하는 '자율 광학 동기화' 방식이 제안되었다.22

**장점과 단점:** TDMA의 가장 큰 장점은 간섭을 매우 효과적으로, 그리고 확실하게 제거하여 측정 데이터의 신뢰성을 보장한다는 것이다.24 하지만 치명적인 단점도 존재한다. 바로 시스템에 참여하는 센서의 수가 늘어날수록 각 센서가 신호를 보낼 수 있는 기회가 줄어들어, 개별 센서의 유효 프레임 속도(effective frame rate)가 감소한다는 것이다.21 예를 들어, 4개의 센서가 네트워크에 있다면 각 센서는 전체 시간의 1/4만 사용할 수 있으므로 프레임 속도도 1/4로 줄어든다. 이는 빠른 움직임을 감지해야 하는 자율주행과 같은 실시간 애플리케이션에 심각한 제약이 될 수 있다. 또한, 슬롯 간의 충돌을 방지하기 위한 보호 구간(guard interval)이 필요하며, 이는 전체 시스템의 처리량(throughput)을 다소 감소시키는 요인이 된다.21


정적 TDMA(Static TDMA)의 고정된 시간 슬롯 할당 방식과 그로 인한 프레임 속도 저하라는 명백한 한계를 극복하기 위해, 보다 유연하고 지능적인 TDMA 변형 기술들이 개발되고 있다.

**동적 TDMA (Dynamic TDMA, dTDMA):** 이 방식은 모든 센서에 고정된 길이의 시간 슬롯을 영구적으로 할당하는 대신, 각 센서의 실시간 데이터 전송 요구량에 따라 시간 슬롯의 수나 길이를 동적으로 조절한다.21 예를 들어, 전방을 주시하는 LiDAR는 고속 주행 시 더 높은 프레임 속도가 필요하므로 더 많은 시간 슬롯을 할당받고, 측후방 감지용 LiDAR는 상대적으로 적은 슬롯을 할당받을 수 있다. 이를 통해 시스템 전체의 자원을 보다 효율적으로 활용하고, 중요한 센서의 성능 저하를 최소화할 수 있다.

**분산형 비동기화 (Decentralized De-synchronization):** 중앙 제어 장치 없이도 TDMA와 유사한 효과를 내기 위한 접근법이다. D2SR 프레임워크에서 제안된 이 방식은 각 LiDAR가 명시적인 통신이나 협의 없이 독립적으로 자신의 활성 기간(active period)을 무작위로 시간 이동(time-shifting)시킨다.25 이는 모든 센서가 완벽하게 다른 시간에 작동하는 것을 보장하지는 않지만, 여러 센서의 활성 기간이 우연히 겹칠 '확률'을 통계적으로 크게 낮춘다. 이는 일종의 확률적 TDMA로, 구현이 간단하면서도 상당한 간섭 감소 효과를 보인다. 연구에 따르면, 2개의 LiDAR 환경에서 간섭 프레임의 비율을 70% 이상 줄일 수 있었다.25

**AI 기반 스케줄링 (AI-Driven Scheduling):** TDMA의 미래는 인공지능과의 결합에 있다. 차량의 주행 상황, 주변 환경의 복잡도, 다른 차량의 예상 경로 등을 종합적으로 판단하는 중앙 AI 시스템이 실시간으로 TDMA 스케줄을 최적화하는 시나리오를 상상할 수 있다. 예를 들어, 복잡한 교차로에 진입할 때는 모든 센서의 데이터를 종합적으로 분석하기 위해 TDMA 스케줄을 잠시 완화하거나 변경하고, 한적한 고속도로에서는 전방 LiDAR의 프레임 속도를 극대화하기 위해 시간 슬롯을 집중시키는 방식이다. 드론 스웜(drone swarm)과 같이 수많은 에이전트가 협력해야 하는 환경에서는, AI가 주도하는 동적 네트워크 토폴로지(예: Opportunistic Resilient Network Topology)가 각 드론의 LiDAR 작동 시간을 조율하여 상호 간섭을 최소화하는 데 핵심적인 역할을 할 것이다.26 이러한 지능형 접근법은 TDMA를 단순한 충돌 회피 프로토콜에서 시스템 성능을 극대화하는 동적 자원 관리 도구로 발전시킬 잠재력을 가지고 있다.


TDMA 외에도 CDMA(Code-Division Multiple Access)와 같은 다른 다중 접속 방식들이 LiDAR 간섭 완화를 위해 고려되고 있으며, 각 방식은 뚜렷한 장단점을 가진다. 시스템 설계자는 데이터 속도, 확장성, 시스템 복잡도라는 세 가지 핵심 축을 기준으로 이들을 평가해야 한다.

**TDMA:**

- **데이터 속도 및 확장성:** TDMA는 간섭을 완벽하게 제거하지만, 센서 수가 증가함에 따라 각 센서의 데이터 속도(유효 프레임 속도)가 반비례하여 감소하는 확장성 문제를 안고 있다.21 이는 센서 밀도가 높은 환경에서는 심각한 성능 저하로 이어질 수 있다.
- **시스템 복잡도:** 개념적으로는 간단하지만, 모든 센서 간의 나노초 수준의 정밀한 시간 동기화가 필수적이며, 이를 위한 PTP/gPTP 네트워크 인프라가 요구된다.22 동기화 실패는 시스템 전체의 실패로 이어질 수 있다.

**CDMA:**

- **데이터 속도 및 확장성:** CDMA는 여러 센서가 동일한 시간에 동시에 작동할 수 있도록 허용하므로, 센서 수가 증가해도 각 센서의 데이터 속도가 크게 저하되지 않는다. 이는 확장성 측면에서 TDMA보다 우수하다.20
- **시스템 복잡도:** CDMA는 송신단에서 신호를 고유 코드로 변조하고, 수신단에서 이를 복조하고 다른 코드의 신호를 걸러내기 위한 복잡한 신호 처리 과정(예: 정합 필터링)을 필요로 한다.1 이는 하드웨어 및 소프트웨어의 복잡도와 비용을 증가시키는 요인이다. 또한, 간섭을 완벽히 제거하는 것이 아니라 관리 가능한 수준의 배경 잡음으로 바꾸는 방식이므로, 간섭 신호가 매우 강할 경우 잡음 바닥(noise floor)이 상승하여 약한 신호를 탐지하는 데 어려움을 겪을 수 있다.20

하이브리드 접근법 (Hybrid Approaches):

이러한 상반된 특성 때문에, TDMA와 CDMA의 장점을 결합하려는 하이브리드 방식이 가장 전략적으로 유망한 경로로 부상하고 있다. 예를 들어, 'CDMA-TDMA' 하이브리드 시스템은 전체 시간을 여러 개의 TDMA 슬롯으로 나누되, 각 슬롯에 단 하나의 센서가 아닌, 서로 다른 CDMA 코드를 가진 여러 센서 그룹을 할당할 수 있다. 이 방식은 순수 TDMA보다 주어진 시간 내에 더 많은 센서를 작동시켜 데이터 속도 문제를 완화하고, 동시에 각 그룹 내에서는 CDMA 코드를 통해 상호 간섭을 방지한다. 이러한 하이브리드 간섭 완화 기법에 대한 특허 출원 동향은 이 접근법이 단순한 학술적 논의를 넘어 실제 산업계에서 심도 있게 고려되고 있음을 보여준다.28 이는 결국, 간섭 완화가 단일 기술의 선택 문제가 아니라, 주행 시나리오와 시스템 요구사항에 따라 동적으로 자원을 할당하고 프로토콜을 조합하는 시스템 수준의 최적화 문제로 진화하고 있음을 의미한다.

------



다중 LiDAR 시스템의 간섭 완화 전략, 특히 TDMA와 같은 협력적 프로토콜을 구현하기 위한 가장 근본적인 전제 조건은 모든 센서와 제어 장치들이 하나의 기준 시간에 맞춰 정밀하게 동기화되는 것이다. 이 역할을 수행하는 핵심 기술이 바로 IEEE 1588 정밀 시간 프로토콜(PTP, Precision Time Protocol)이다.

**목적과 성능:** PTP는 이더넷과 같은 패킷 기반 네트워크를 통해 분산된 장치들의 내부 클럭을 동기화하기 위해 설계된 프로토콜이다.30 PTP의 가장 큰 특징은 하드웨어 타임스탬핑을 활용하여 소프트웨어 기반의 NTP(Network Time Protocol)가 제공하는 밀리초 수준을 훨씬 뛰어넘는, 마이크로초 이하, 심지어 나노초 수준의 동기화 정확도를 달성할 수 있다는 점이다.30 이는 데이터 융합의 정확도를 높이고, TDMA의 시간 슬롯을 정밀하게 제어하는 데 필수적이다.

**작동 메커니즘:** PTP는 마스터-슬레이브(Master-Slave) 구조로 작동한다. 네트워크 내에서 가장 정밀한 클럭을 가진 장치가 그랜드마스터(Grandmaster)가 되어 시간의 기준점을 제공한다. 동기화 과정은 마스터와 슬레이브 간의 일련의 메시지 교환을 통해 이루어진다.33

1. 마스터는 `Sync` 메시지를 전송하면서 송신 시각($T_1$)을 기록한다. 이 $T_1$ 정보는 `Sync` 메시지 자체에 포함되거나(1-step), 후속 `Follow_Up` 메시지로 전송된다(2-step).30
2. 슬레이브는 `Sync` 메시지를 수신하면서 자신의 수신 시각($T_2$)을 기록한다.
3. 슬레이브는 마스터에게 `Delay_Req` 메시지를 전송하면서 자신의 송신 시각($T_3$)을 기록한다.
4. 마스터는 `Delay_Req` 메시지를 수신하면서 수신 시각($T_4$)을 기록하고, 이 $T_4$ 정보를 `Delay_Resp` 메시지에 담아 슬레이브에게 다시 보낸다.

이 네 개의 타임스탬프($T_1, T_2, T_3, T_4$)를 모두 확보한 슬레이브는 네트워크 전송 지연 시간(delay)과 마스터와의 시간 오프셋(offset)을 계산할 수 있다. 핵심 계산식은 다음과 같다: $Offset = (T_2 + T_3 - T_1 - T_4) / 2$이다.30 이는 

$T_2 - T_1 = o + d$와 $T_4 - T_3 = -o + d$ 두 식을 빼서 유도된다.

**핵심 가정:** 이 프로토콜의 정확성은 두 가지 중요한 가정에 의존한다. 첫째, 메시지 교환이 일어나는 짧은 시간 동안 마스터와 슬레이브 간의 클럭 오프셋은 일정하다고 간주된다. 둘째, 마스터에서 슬레이브로의 네트워크 경로 지연과 슬레이브에서 마스터로의 경로 지연이 대칭적(symmetrical)이라고 가정한다.30


표준 PTPv2가 다양한 산업 분야를 위한 범용 프로토콜이라면, 자동차 산업의 엄격한 요구사항을 충족시키기 위해 특화된 프로파일이 등장했다. 바로 시간 민감형 네트워킹(TSN, Time-Sensitive Networking)의 핵심 구성 요소인 IEEE 802.1AS, 통칭 gPTP(generalized PTP)다.

**gPTP (Generalized PTP):** gPTP는 IEEE 802.1AS 표준에 정의된 PTP의 특정 프로파일로, 자동차, 산업 자동화, 전문 오디오/비디오 브리징(AVB)과 같은 TSN 환경에 최적화되어 있다.34 gPTP는 PTPv2의 다양한 옵션들을 제한하고 단순화하여, 서로 다른 제조업체의 장비 간 상호운용성을 높이는 데 중점을 둔다.35 이는 복잡한 차량 내 네트워크에서 다양한 센서와 ECU(Electronic Control Unit)가 원활하게 동기화되도록 보장하는 데 매우 중요하다.

**IEEE 802.1DG-2025 Automotive Profile:** 2025년 중반에 발표된 이 최신 표준은 차량 내 이더넷 통신을 위한 TSN 프로파일을 명확하게 정의한다.36 이 표준은 gPTP를 기반으로 보안, 고가용성, 신뢰성, 그리고 가장 중요한 결정론적 지연 시간(bounded latency)에 대한 자동차 산업의 요구사항을 충족시키는 구체적인 가이드라인을 제공한다. 이는 다중 센서 데이터의 실시간 융합과 안전에 직결되는 제어 신호 전달을 위한 표준화된 프레임워크를 마련했다는 점에서 큰 의미를 가진다.

**최신 개정 동향 (2020, 2024):** gPTP 표준은 지속적으로 발전하고 있다. IEEE 802.1AS-2020 개정판에서는 여러 시간 도메인(multiple time domains)을 지원하고 동기화 트리(synchronization tree)를 명시적으로 정의하는 기능과 같은 중요한 개선이 이루어졌다.35 '다중 시간 도메인' 지원은 예를 들어, 안전에 민감한 LiDAR 및 레이더 데이터 동기화를 위한 시간 도메인과 인포테인먼트 시스템을 위한 시간 도메인을 분리하여 운영할 수 있게 해, 시스템의 안정성과 보안을 강화한다. 또한, 2024년 10월에 발표된 IEEE 802.1ASdm-2024 수정안은 gPTP를 위한 

**핫 스탠바이 이중화(hot-standby redundancy)** 솔루션을 정의하고, 네트워크 홉(hop) 수가 많은 환경에서도 정확도를 높이는 향상된 기능을 포함한다.35

이러한 표준의 진화는 시간 동기화가 더 이상 단순한 편의 기능이 아니라, 차량 아키텍처의 핵심적인 안전 필수 요소로 자리 잡고 있음을 명확히 보여준다. 이중화, 다중 도메인 지원과 같은 기능들은 동기화 네트워크의 장애가 곧 시스템 전체의 치명적인 오류로 이어질 수 있는 자율주행 환경의 요구를 반영한 결과물이다.


PTP/gPTP를 통해 나노초 수준의 정밀 동기화를 달성하는 것은 이론처럼 간단하지 않으며, 실제 구현에는 여러 가지 현실적인 과제가 따른다.

**하드웨어 지원의 필수성:** PTP의 높은 정확도는 **하드웨어 타임스탬핑(hardware timestamping)**에 크게 의존한다. 이는 PTP 메시지의 송수신 타임스탬프를 네트워크 인터페이스 카드(NIC)의 물리 계층(PHY)에서 직접 기록하는 것을 의미한다.30 만약 타임스탬프가 운영체제나 소프트웨어 스택의 상위 계층에서 기록된다면, 예측 불가능한 처리 지연(jitter)이 추가되어 동기화 정확도가 크게 저하된다. 따라서 고정밀 동기화가 필요한 모든 장치(LiDAR, ECU, 스위치 등)는 하드웨어 타임스탬핑을 지원해야 한다.

**PTP 지원 네트워크 스위치의 중요성:** PTP의 핵심 가정 중 하나인 '경로 지연의 대칭성'을 유지하기 위해서는 네트워크 인프라, 특히 스위치의 역할이 매우 중요하다. 일반적인 이더넷 스위치는 패킷을 버퍼에 저장했다가 전달하는 'Store-and-Forward' 방식으로 작동하며, 이때 발생하는 큐잉 지연(queuing delay)은 예측 불가능하고 비대칭적일 수 있다. 이는 PTP의 정확도를 심각하게 훼손한다.38 이 문제를 해결하기 위해 PTP 지원 스위치는 

**투명 클럭(Transparent Clock, TC)** 기능을 사용한다. TC 스위치는 PTP 패킷이 스위치 내부에 머무른 시간(residence time)을 측정하여, 패킷의 보정 필드(correction field)에 이 값을 누적시킨다. 이를 통해 패킷이 여러 스위치를 거치면서 발생하는 지연을 보상하여 종단 간(end-to-end) 동기화 정확도를 유지할 수 있다.33 Ouster LiDAR와 같은 장비가 PTP 동기화 상태에서 'UNCALIBRATED'에 머무는 문제는 종종 PTP를 지원하지 않는 스위치 사용으로 인해 

`Delay_Req/Resp` 메시지가 제대로 전달되지 않기 때문에 발생한다.28

**실제 구현 사례:** Ouster, Livox, Velodyne과 같은 주요 LiDAR 제조업체들은 자사 제품의 다중 센서 동기화 솔루션으로 PTP/gPTP를 적극적으로 채택하고 있다.39 실제 시스템을 구축할 때는 차량의 주 컴퓨터나 전용 타임 서버를 그랜드마스터 클럭으로 설정하고, 네트워크에 연결된 모든 LiDAR 센서와 스위치가 동일한 PTP 도메인 번호와 프로파일(예: default, gPTP, automotive-slave)을 사용하도록 구성해야 한다.34 이러한 정밀한 설정과 하드웨어 요구사항은 차량 내 네트워크가 단순한 데이터 전송 경로를 넘어, 전체 시스템의 성능과 안전을 좌우하는 결정론적 타이밍 유틸리티로 설계되어야 함을 시사한다.

------



하드웨어와 프로토콜 수준에서 간섭을 완화하려는 노력에도 불구하고, 일부 간섭 신호는 여전히 포인트 클라우드 데이터에 침투할 수 있다. 이를 해결하기 위한 마지막 방어선이 바로 소프트웨어 기반의 후처리(post-processing) 기술이다. 전통적인 방식은 통계적, 공간적 특성을 이용한 필터링 알고리즘에 의존한다.

**공간 영역 필터링 (Spatial Domain Filtering):** 이 접근법은 포인트의 3D 공간 좌표 정보를 기반으로 노이즈를 식별한다. 가장 대표적인 예는 **통계적 이상치 제거(Statistical Outlier Removal)** 필터다. 이 필터는 각 포인트에 대해 주변의 이웃 포인트들과의 평균 거리를 계산하고, 이 거리가 설정된 임계값을 초과하는 포인트를 이상치(outlier), 즉 노이즈로 간주하여 제거한다.44 또 다른 간단한 방법은 **통과 필터(Pass-Through Filter, PTF)**로, 특정 좌표 범위를 벗어나는 모든 포인트를 제거하는 방식이다. 이러한 방법들은 주로 배경이나 명백히 잘못된 위치에 생성된 노이즈를 제거하는 데 사용된다.

**반사율/강도 정보 활용 (Reflectivity/Intensity Filtering):** LiDAR는 거리 정보뿐만 아니라 반사된 레이저의 강도(intensity) 또는 반사율(reflectivity) 정보도 함께 수집한다. 이 정보는 간섭을 필터링하는 데 유용한 단서를 제공한다. 예를 들어, 도로 위의 먼지나 안개 입자는 일반적인 고체 장애물과 다른 반사율 특성을 보일 수 있다. 한 연구에서는 레이저 반사율을 기준 템플릿으로 사용하여 신뢰도 맵을 생성하고, 이를 통해 희소한 먼지 포인트 클라우드를 1차적으로 걸러내는 방법을 제안했다.44 이처럼 반사율 정보는 물리적 특성이 다른 간섭원과 실제 물체를 구분하는 데 효과적으로 사용될 수 있다.45

**한계점:** 이러한 고전적인 필터링 기법들은 구현이 비교적 간단하고 계산 부하가 적다는 장점이 있지만, 명확한 한계를 가진다. 대부분 고정된 임계값에 의존하기 때문에, 다양한 환경과 시나리오에 유연하게 대처하기 어렵다.44 너무 공격적인 필터링은 멀리 있는 작은 실제 물체(a small object at far distance)의 유효한 포인트까지 노이즈로 오인하여 제거할 수 있는 위험이 있다.4 반대로 너무 보수적인 필터링은 모든 간섭 포인트를 제거하지 못한다. 특히, 간섭으로 인해 생성된 오류가 특정 영역에 집중적으로 나타나는 경우, 단순한 이상치 제거 알고리즘으로는 이를 효과적으로 처리하기 어렵다.25


고전적인 필터링 방식의 한계를 극복하기 위해, 인공지능(AI), 특히 딥러닝(Deep Learning) 기술이 LiDAR 간섭 처리 분야에 혁명적인 변화를 가져오고 있다. AI 모델은 인간이 정의한 규칙이 아닌, 방대한 데이터로부터 간섭의 복잡하고 비선형적인 패턴을 스스로 학습하여, 훨씬 더 정교하고 강건한 탐지 및 제거 성능을 보여준다.

**간섭 탐지 (Interference Detection):** 간섭 완화의 첫 단계는 간섭이 발생했음을 인지하는 것이다. 경량화된 컨볼루션 신경망(CNN, Convolutional Neural Network)을 사용하여 LiDAR의 전체 프레임이 간섭으로 오염되었는지를 분류하는 연구가 성공적으로 수행되었다. 한 연구에서는 이러한 CNN 기반 탐지기가 92%의 높은 F1-score를 달성하여, 간섭이 발생한 프레임에 대해서만 후속 완화 또는 복구 알고리즘을 선택적으로 활성화할 수 있음을 보여주었다.25 이는 불필요한 계산을 줄여 실시간 시스템에 적용하는 데 유리하다.

**간섭 제거/완화 (Interference Removal/Mitigation):** 딥러닝 모델은 오염된 신호나 포인트 클라우드를 직접 처리하여 간섭 성분만을 선택적으로 제거하도록 학습될 수 있다.

- **시간-주파수 영역 접근:** 일부 기법들은 간섭이 포함된 원시 비트 신호(beat signal)를 단시간 푸리에 변환(STFT) 등을 통해 시간-주파수 영역의 이미지(스펙트로그램)로 변환한다. 이 이미지에서 간섭은 독특한 패턴(예: 짧은 처프 형태)으로 나타나고, 실제 목표물은 일관된 선으로 나타난다. 딥러닝 기반의 이미지 복원(image restoration) 또는 디노이징(denoising) 기술(예: 복소수 값 완전 컨볼루션 네트워크, 이중-트리 복소 웨이블릿 변환)을 적용하여 간섭 패턴만을 제거한 후, 다시 시간 영역 신호로 복원하는 방식이다.10
- **트랜스포머 기반 아키텍처 (Transformer-based Architectures):** 트랜스포머 모델은 데이터 내의 장거리 의존성(long-range dependencies)을 포착하는 데 탁월한 능력을 보인다. 이 특성은 주기적인 목표물 신호의 패턴을 학습하고, 비주기적인 간섭 신호로부터 목표 신호를 재구성하는 데 매우 효과적이다. 'RIMformer'와 같은 모델이 이러한 접근법의 예시다.47
- **생성 모델 (Generative Models):** 생성적 적대 신경망(GAN, Generative Adversarial Network)과 같은 생성 모델도 간섭 완화에 활용된다. 생성자(Generator)는 간섭이 제거된 깨끗한 신호를 생성하도록 학습하고, 판별자(Discriminator)는 생성된 신호가 실제 깨끗한 신호와 구별되지 않도록 학습하는 과정을 통해, 간섭이 포함된 입력으로부터 깨끗한 신호를 생성하는 능력을 갖추게 된다.47

이러한 AI 기반 접근법들은 기존 알고리즘보다 훨씬 더 높은 정확도와 유연성을 제공하며, 간섭 완화 기술의 패러다임을 바꾸고 있다.


가장 진보된 AI 시스템은 단순히 간섭으로 오염된 포인트를 '제거'하는 것을 넘어, 손실된 정보를 '복구'하고 '재구성'하는 단계로 나아가고 있다. 이는 소프트웨어의 역할을 수동적인 데이터 정제에서 능동적인 정보 창출로 격상시키는 중요한 변화다.

**D2SR 프레임워크 (Detection, Mitigation, Recovery):** 이 프레임워크는 AI 기반 간섭 처리의 전체 파이프라인을 보여주는 대표적인 사례다. 3단계로 구성된다: (a) **탐지(Detection)** 단계에서 CNN을 사용하여 간섭 프레임을 식별하고, (b) **완화(Mitigation)** 단계에서 분산형 비동기화 기법으로 간섭 발생 확률을 줄인 후, (c) 마지막 **복구(Recovery)** 단계가 핵심이다. 이 단계에서 통합된 심층 신경망(DNN)은 먼저 간섭으로 인해 값이 손상된 영역을 식별하는 '간섭 마스크(interference mask)'를 생성한다. 그리고 이 마스크 내의 픽셀(포인트)들에 대해 주변의 유효한 데이터를 기반으로 원래의 올바른 깊이 값을 추론하여 재생성한다.25 이는 단순히 데이터를 버리는 것이 아니라, 손상된 데이터를 유용한 정보로 되살리는 과정이다.

**INTACT 프레임워크:** 2025년에 발표된 INTACT(INTegrating Adversarial Curriculum Training) 프레임워크는 메타 학습(meta-learning)과 적대적 커리큘럼 학습(adversarial curriculum training)을 결합하여 LiDAR 노이즈에 대한 DNN의 강건성을 극대화하는 데 초점을 맞춘다. 이 프레임워크는 다양한 노이즈 조건에 효과적으로 일반화될 수 있는 강력한 초기 표현을 학습하고, 점차 복잡성이 증가하는 노이즈 패턴을 훈련 과정에 도입하여 실제 환경의 데이터 품질 저하 시나리오를 모방한다. 그 결과, 가우시안 노이즈 환경에서 객체 추적 정확도(MOTA)를 12.4%나 향상시키는 등, 기존 학습 방식보다 월등한 성능을 보이며 AI 모델 자체가 데이터 손상에 내성을 갖도록 만드는 새로운 패러다임을 제시했다.48

**VFM 기반 접근법 (Visual Foundation Models):** 또 다른 최신 연구 동향은 3D 포인트 클라우드 자체의 노이즈 문제를 우회하는 것이다. 2025년 연구에서는 Ouster 센서가 제공하는 고품질 2D 파노라마 이미지(깊이, 신호 강도, 주변광 이미지)에 Florence 2와 같은 시각 파운데이션 모델(VFM)을 직접 적용하여 객체 탐지 및 캡셔닝을 수행하는 방법을 제안했다.49 이 방식은 계산 비용이 높고 노이즈 처리가 까다로운 3D 포인트 클라우드 처리 과정을 생략하고, 보다 정제된 2D 이미지 데이터를 활용함으로써 잠재적인 데이터 손상에도 불구하고 강건한 인식을 달성하는 새로운 경로를 열었다.

이러한 복구 및 재구성 기술의 등장은 매우 중요한 전략적 의미를 가진다. 이는 소프트웨어의 힘으로 하드웨어의 물리적 한계를 보완할 수 있음을 의미하기 때문이다. 강력한 AI 복구 파이프라인을 갖추고 있다면, 다소 간섭에 취약하지만 저렴한 LiDAR 센서를 사용하더라도 시스템 전체적으로는 높은 성능을 유지할 수 있게 된다. 이는 센서 하드웨어와 소프트웨어 스택 전체를 아우르는 새로운 비용-성능 최적화의 가능성을 열어주며, LiDAR 시장의 경쟁 구도를 바꿀 수 있는 핵심적인 차별화 요소가 될 것이다.

------



지금까지 논의된 다양한 LiDAR 간섭 완화 전략들은 각각 뚜렷한 장점과 단점을 가지고 있다. 특정 애플리케이션에 가장 적합한 솔루션을 선택하기 위해서는 성능, 비용, 복잡성, 확장성 등 다각적인 기준에 따라 이들을 종합적으로 비교 분석하는 것이 필수적이다. 이 섹션에서는 주요 완화 기법들을 이러한 핵심 지표를 기준으로 평가하여, 기술 전략가와 시스템 설계자가 정보에 입각한 의사결정을 내릴 수 있도록 지원하는 프레임워크를 제시한다.

예를 들어, FMCW나 RMCW와 같은 파형 공학 기반의 하드웨어 솔루션은 근본적인 간섭 저항성을 제공하지만, 일반적으로 높은 개발 비용과 시스템 복잡성을 수반한다.3 반면, TDMA와 같은 프로토콜 기반 접근법은 개념이 간단하고 간섭 제거 효과가 확실하지만, 센서 수가 증가할수록 확장성에 한계를 보인다.21 소프트웨어 기반의 AI 솔루션은 기존 하드웨어의 한계를 보완할 수 있는 강력한 유연성을 제공하지만, 막대한 개발 노력과 높은 실시간 연산 능력을 요구한다.25 이러한 트레이드오프 관계를 명확히 이해하는 것은 성공적인 시스템 설계를 위한 첫걸음이다.


아래 표는 주요 LiDAR 간섭 완화 기술들을 핵심 평가 기준에 따라 종합적으로 비교한 것이다. 이 표는 각 기술의 본질적인 특성과 전략적 가치를 한눈에 파악할 수 있도록 설계되었으며, 복잡한 기술적 내용을 구조화하여 의사결정을 지원하는 핵심 도구 역할을 한다. 이를 통해 기술 전략가는 특정 시나리오에서 어떤 기술 조합이 최적의 비용-성능 균형을 제공할지 신속하게 판단할 수 있다. 예를 들어, 이 표는 RMCW 시스템의 높은 초기 투자 비용과 AI 복구 파이프라인의 낮은 하드웨어 비용 및 높은 연산 비용 간의 근본적인 차이를 명확하게 보여준다.

| **완화 기술**          | **기본 원리**                                             | **성능 및 효과**                                             | **장점**                                                    | **단점**                                                | **비용 및 복잡성**                           | **주요 플레이어 / 지지자**        |
| ---------------------- | --------------------------------------------------------- | ------------------------------------------------------------ | ----------------------------------------------------------- | ------------------------------------------------------- | -------------------------------------------- | --------------------------------- |
| **FMCW**               | 주파수 변조; 코히런트 감지.                               | 우수한 내재적 저항성, 그러나 유사/동기화된 간섭원에는 취약.3 | 속도를 직접 측정; 우수한 SNR.11                             | 완벽히 면역되지 않음; 복잡하고 비싼 레이저/변조 회로.8  | 높음.3                                       | (신흥 기업, 일부 연구 집중)       |
| **RMCW / True-Random** | 랜덤/의사-랜덤 변조; 상관관계 기반 감지.2                 | 간섭에 대해 매우 높거나 거의 완벽한 면역성.12                | 극도로 강건함; 높은 보안성.                                 | 고속 디지털/아날로그 처리 요구; 시스템 비용 증가 가능.1 | 높음.3                                       | (첨단 연구, IP 해자 가능성)       |
| **1550nm 파장**        | 눈에 안전하고 태양광 잡음이 적은 대역에서 작동.15         | 높음. 더 높은 출력을 허용하여 장거리 및 악천후 성능 향상. 905nm 장치와 본질적인 스펙트럼 분리.13 | 눈에 안전함; 장거리; 태양광 간섭 적음.                      | 역사적으로 InGaAs 검출기로 인한 높은 비용.13            | 중간-높음 (가격 격차 감소 중).13             | Luminar 18                        |
| **TDMA**               | 시간 슬롯 할당; 한 번에 하나의 센서만 전송.21             | 탁월함; 조정된 센서 간의 간섭을 완전히 제거.                 | 간단하고 효과적인 간섭 제거.24                              | 센서당 유효 프레임 속도 감소; 정밀한 동기화 필요.21     | 낮음-중간 (동기화 방식에 따라 다름).         | (다중 카메라/로보틱스에서 일반적) |
| **CDMA**               | 각 센서에 고유 코드 변조; 수신기에서 상관 필터링.20       | 좋음; 간섭을 낮은 수준의 잡음으로 변환.                      | 다수의 동시 사용자로 확장 가능; 센서 식별.20                | 신호 처리 복잡성 증가; 잡음 플로어 상승 가능.2          | 중간-높음 (복잡한 처리).                     | (연구 중심, 특허 활동)            |
| **AI 기반 복구**       | DNN이 오염된 데이터 포인트/영역을 탐지, 마스킹, 재구성.25 | 높고 지속적으로 개선 중. 데이터를 제거하는 것이 아니라 복구 가능. 모델과 훈련 데이터에 성능이 크게 의존. | 하드웨어 한계 보완 가능; 적응형; 시간이 지남에 따라 개선됨. | 높은 계산 비용; 방대한 훈련 데이터 필요; 개발 복잡성.   | 낮음 (하드웨어) / 높음 (소프트웨어 및 연산). | NVIDIA, 다양한 연구 기관 25       |


위의 비교 분석에서 명확히 드러나듯이, 어떤 단일 기술도 모든 시나리오에서 완벽한 해결책을 제공하지는 못한다. 따라서 미래의 가장 강건하고 신뢰성 있는 시스템은 단일 기술에 의존하는 대신, 여러 완화 기법을 결합한 **다층적 방어(defense-in-depth)** 또는 **하이브리드(hybrid)** 전략을 채택할 것이다.

이러한 하이브리드 시스템은 하드웨어, 프로토콜, 소프트웨어의 각 계층에서 독립적인 방어선을 구축하여 전체 시스템의 강건성을 극대화한다. 예를 들어, 다음과 같은 하이브리드 시스템을 구상할 수 있다:

1. **하드웨어 계층:** 1550nm 파장의 RMCW LiDAR를 사용하여 근본적인 간섭 저항성과 우수한 SNR을 확보한다. 이는 시스템의 1차 방어선 역할을 한다.
2. **프로토콜 계층:** 이 LiDAR들을 동적 TDMA 스케줄에 따라 작동시켜, 대부분의 시간 동안 물리적인 신호 충돌을 회피한다. 이는 2차 방어선이다.
3. **소프트웨어 계층:** 그럼에도 불구하고 발생하는 미미한 간섭이나 데이터 손실은 최종적으로 AI 기반 복구 파이프라인을 통과하면서 처리된다. 이 소프트웨어 계층은 시스템의 최종 방어선이자, 예상치 못한 상황에 대처하는 유연성을 제공한다.

이처럼 각기 다른 원리로 작동하는 여러 방어 계층을 중첩함으로써, 하나의 계층이 뚫리더라도 다른 계층이 시스템을 보호할 수 있다. 이러한 하이브리드 간섭 완화 기법에 대한 특허 분류의 존재는 이 방향성이 업계의 주요한 흐름임을 뒷받침한다.28 결국, 미래의 간섭 완화는 '하나의 완벽한 기술'을 찾는 것이 아니라, '여러 좋은 기술들을 지능적으로 조합'하는 시스템 엔지니어링의 영역으로 귀결될 것이다.

------



LiDAR 시장의 주요 기업들은 간섭 완화 문제에 대해 각기 다른 전략적 접근법을 취하고 있으며, 이는 그들의 기술적 배경과 시장 포지셔닝을 반영한다.

- **Hesai:** 성능과 신뢰성을 강조하며, 보도자료 등에서 자사의 "간섭 방지 기술(anti-interference technology)"을 꾸준히 언급하고 있다.52 특히, 이들은 적응형 코딩(adaptive coding), 다중 펄스 시퀀스, 해상도 향상 알고리즘과 관련된 다수의 특허를 보유하고 있어 55, 정교한 신호 처리와 하드웨어 설계를 통해 간섭 문제를 해결하려는 강력한 의지를 엿볼 수 있다. 이는 하드웨어 기반의 근본적인 해결책을 중시하는 전략으로 해석된다.
- **Luminar:** 1550nm 파장을 자사의 핵심 차별화 요소로 내세우는 전략을 고수하고 있다. Luminar는 1550nm 파장이 제공하는 높은 출력, 긴 탐지 거리, 눈 안전성 등의 장점이 간섭 저항성에도 기여한다고 강조한다.18 더 높은 출력은 신호의 SNR을 높여 간섭 신호 대비 우위를 점하게 하고, 주류인 905nm와 다른 파장 대역을 사용하는 것 자체가 스펙트럼 분리를 통한 효과적인 간섭 회피 전략이 된다.18
- **Ouster:** "디지털 LiDAR" 아키텍처를 기반으로 한 접근법을 강조한다. 이들은 PTP 동기화 기능과 같은 시스템 통합의 편의성을 부각시키며 34, 최근에는 "물리적 AI(Physical AI)"라는 브랜딩을 통해 유능한 하드웨어와 강력한 소프트웨어 처리 계층의 결합을 전략의 핵심으로 내세우고 있다.60 이는 간섭을 포함한 다양한 현실 세계의 노이즈를 지능적인 소프트웨어로 극복하겠다는 의지를 보여준다. 또한, 자사의 IP 포트폴리오를 적극적으로 방어하며 기술적 리더십을 지키려 하고 있다.62
- **Velodyne:** LiDAR 시장의 선구자 중 하나인 Velodyne은 최신 제품인 Alpha Prime에서 "향상된 센서 간 간섭 완화(advanced sensor-to-sensor interference mitigation)" 기능을 명시적으로 내세우고 있다.63 이는 시장의 요구에 부응하여 간섭 완화 기술에 대한 지속적인 연구개발을 진행하고 있음을 시사하며, 오랜 기간 축적된 기술력을 바탕으로 문제에 접근하고 있음을 보여준다.


자동차 산업의 Tier-1 공급업체인 Bosch와 Continental은 개별 LiDAR 센서 개발보다는 전체 '인식 시스템(perception system)' 통합자로서의 역할을 수행하며 간섭 문제에 접근한다.

이들의 초점은 단일 센서의 성능을 넘어, 여러 센서(LiDAR, 레이더, 카메라)의 데이터를 융합하고, 시스템 전체의 안전성과 신뢰성을 보장하는 데 있다. 예를 들어, Continental은 Aurora와의 파트너십을 통해 자율주행 트럭 시스템을 개발하면서, 강력한 연산 능력을 위해 NVIDIA, Ambarella와 같은 칩 전문가들과 협력하고 있다.65 이는 간섭으로 오염된 데이터를 실시간으로 처리하고 복구하는 데 필요한 고성능 컴퓨팅 파워를 확보하는 것이 중요함을 인식하고 있다는 증거다.

Bosch는 MEMS(미세전자기계시스템) 기술의 강자로서, 센서 자체에 AI 기능을 내장하는 '센서 레벨 AI'에 강점을 보인다.66 이는 간섭 신호를 센서단에서부터 지능적으로 필터링하거나 식별할 수 있는 잠재력을 의미한다. 또한, 이들 기업은 학계 및 연구 기관과 협력하여 새로운 간섭 완화 알고리즘 개발에도 참여하며 67, 시스템 통합자로서 하드웨어와 소프트웨어 양쪽에서 최적의 솔루션을 찾아내려는 노력을 기울이고 있다. 결국 Tier-1 공급업체들은 특정 LiDAR 기술에 종속되기보다는, 다양한 센서 기술들을 조합하여 가장 강건하고 비용 효율적인 전체 시스템을 구축하는 데 핵심적인 역할을 할 것이다.


LiDAR 시장이 성숙기에 접어들면서, 기술 경쟁은 이제 법정에서의 지적 재산(IP) 분쟁으로 확산되고 있다. 이 중 가장 상징적인 사건은 Ouster가 미국 국제무역위원회(ITC)에 Hesai를 상대로 제기한 특허 침해 소송이다.62

Ouster는 Hesai가 자사의 핵심 LiDAR 기술과 관련된 5개의 특허를 침해했다고 주장하며, 해당 제품의 미국 내 수입 금지를 요청했다. 이 소송은 몇 가지 중요한 전략적 함의를 가진다. 첫째, 이는 LiDAR 시장이 초기 기술 탐색의 '와일드 웨스트' 시대를 지나, 시장 점유율과 수익성이 중요해지는 성숙 산업으로 진입했음을 알리는 신호탄이다. Luminar가 Volvo에, Hesai가 SAIC Motor에 대규모 공급 계약을 체결하는 등 18, 시장 규모가 수십억 달러에 달하면서 핵심 기술을 보호하려는 동기가 그 어느 때보다 강해졌다.

둘째, 이는 독자적인 간섭 완화 기술과 같은 핵심 기술에 대한 특허가 강력한 경쟁 우위이자 '기술적 해자'가 될 수 있음을 보여준다. Hesai 역시 적응형 코딩과 같은 분야에서 자체 특허 포트폴리오를 구축하고 있으며 55, 앞으로 기업들은 단순히 기술을 개발하는 것을 넘어, 이를 법적으로 방어하고 경쟁사의 진입을 막는 수단으로 활용할 것이다.

결과적으로, LiDAR 산업에 참여하는 모든 기업(OEM, Tier-1, 센서 제조업체)에게 IP 실사(due diligence)는 이제 선택이 아닌 필수가 되었다. 기술 로드맵을 수립할 때 기술적 타당성뿐만 아니라 특허 침해 리스크를 반드시 검토해야 하며, 이는 향후 파트너십, M&A 등 기업 전략 전반에 큰 영향을 미칠 것이다.


LiDAR 기술은 현재의 과제를 해결하는 동시에, 미래의 새로운 가능성을 향해 끊임없이 진화하고 있다. 2025년 이후의 기술 궤적을 형성할 몇 가지 중요한 트렌드는 다음과 같다.

- **양자 센싱 (Quantum Sensing):** 물리학의 최전선에서는 양자 현상을 이용한 차세대 LiDAR 연구가 진행되고 있다. 예를 들어, '압축광(squeezed light)'과 같은 양자 상태의 빛을 사용하면, 고전적인 한계를 뛰어넘는 정밀도로 거리와 속도를 동시에 측정할 수 있다.68 아직 초기 연구 단계이지만, 양자 센싱은 잡음과 간섭을 처리하는 새로운 방식을 제시하며 센서 성능의 근본적인 도약을 이끌 잠재력을 가지고 있다.
- **드론 스웜 조정 (Drone Swarm Coordination):** 수백, 수천 개의 드론으로 구성된 스웜(swarm)이 동시에 작전을 수행하는 시나리오는 LiDAR 간섭 문제의 극단적인 예시를 보여준다.27 이러한 환경에서는 중앙 집중식 제어가 불가능하므로, 각 드론이 자율적으로 통신하고 협력하여 서로의 LiDAR 신호를 방해하지 않도록 하는 고도의 분산형 조정 전략이 필수적이다. 이를 위해 AI 기반의 동적 네트워크 토폴로지 기술, 예를 들어 미 국방부가 연구 중인 ACT(Autonomous Collaborative Teaming)나 ORIENT(Opportunistic Resilient Network Topology)와 같은 개념이 핵심적인 역할을 할 것이다.27
- **차세대 표준의 진화:** PTP/gPTP 동기화 표준의 지속적인 발전은 미래 다중 센서 시스템의 근간이 될 것이다. IEEE 802.1AS-2020, 802.1ASdm-2024, 그리고 802.1DG-2025와 같은 최신 표준들은 단순한 시간 동기화를 넘어, 이중화(redundancy), 보안, 다중 시간 도메인 지원 등 고도의 신뢰성과 강건성을 요구하는 자율주행 환경의 요구사항을 반영하고 있다.35 이러한 표준화된 프레임워크는 더욱 복잡하고 정교한 협력적 간섭 완화 기술을 구현하는 데 필수적인 기반을 제공할 것이다.

이러한 미래 기술들은 LiDAR가 단순한 거리 측정 센서를 넘어, 지능적으로 상호작용하고 협력하는 네트워크의 일부로 진화하고 있음을 보여준다.

------



본 보고서의 심층 분석을 통해 도출된 결론은 명확하다. LiDAR 상호 간섭 문제는 더 이상 단일 기술로 해결할 수 있는 단순한 과제가 아니며, 시장은 필연적으로 **하이브리드, AI 중심의 간섭 관리** 패러다임으로 이동하고 있다.

과거에는 하드웨어 파형 설계(예: FMCW)나 프로토콜(예: TDMA) 중 하나를 선택하여 간섭을 '회피'하는 데 중점을 두었다. 그러나 센서 밀도가 기하급수적으로 증가하는 현실 세계에서 완벽한 회피는 불가능하며, 각 기술은 성능, 비용, 확장성 측면에서 명백한 트레이드오프를 가진다. 따라서 미래의 가장 강건한 시스템은 하드웨어의 내재적 저항성, 프로토콜의 협력적 회피, 그리고 소프트웨어의 지능적 복구를 결합한 다층적 방어 전략을 채택할 것이다.

특히 주목해야 할 변화는 혁신의 무게 중심이 순수한 하드웨어 성능에서 불확실성을 관리하고 손상된 데이터를 복원하는 소프트웨어 및 AI 계층으로 이동하고 있다는 점이다. 가장 성공적인 시스템은 간섭을 완벽하게 피해야 할 문제로 보는 것이 아니라, 지능적으로 '관리'해야 할 환경 조건의 일부로 받아들일 것이다. 이는 LiDAR 기업의 핵심 경쟁력이 더 이상 '얼마나 멀리, 얼마나 정밀하게 보는가'에만 있지 않고, '얼마나 지저분한 환경에서도 신뢰할 수 있는 정보를 만들어내는가'로 확장되고 있음을 의미한다.


이러한 패러다임 전환에 따라, 자율 시스템을 설계하는 아키텍트와 엔지니어는 다음과 같은 전략적 접근을 취해야 한다.

- **다층적 방어(Layered Defense) 설계:** 단일 솔루션의 한계를 인정하고, 여러 독립적인 완화 계층을 설계에 포함시켜야 한다. 예를 들어, RMCW와 같은 강력한 하드웨어 기반 위에 TDMA 프로토콜을 얹고, 최종적으로 AI 복구 파이프라인을 두는 방식이다. 하나의 방어선이 실패하더라도 다른 방어선이 시스템을 보호할 수 있도록 설계해야 한다.
- **동기화 백본(Synchronization Backbone)의 우선순위화:** PTP/gPTP를 지원하는 이더넷 네트워크를 단순한 데이터 통로가 아닌, 시스템의 안전에 직결되는 핵심 유틸리티로 취급해야 한다. 자동차 등급의 PTP 지원 스위치와 하드웨어 타임스탬핑 기능이 내장된 프로세서에 대한 투자는 비용이 아니라 필수적인 안전 요구사항이다.
- **소프트웨어 보상을 고려한 설계:** 하드웨어를 선택할 때, 그 약점을 소프트웨어로 어떻게 보완할 수 있는지를 함께 고려해야 한다. 약간의 간섭에 더 취약하지만 가격이 저렴한 센서가 강력한 AI 복구 파이프라인과 결합될 때, 전체 시스템 관점에서는 더 비싼 고성능 센서보다 최적의 비용-성능 솔루션이 될 수 있다.
- **실시간 성능 확보:** 소프트웨어 기반, 특히 복잡한 AI 모델 기반의 완화 기술을 도입할 때, 이것이 NVIDIA Jetson과 같은 엣지 컴퓨팅 하드웨어에서 허용 불가능한 지연 시간(latency)을 유발하지 않는지 철저히 검증해야 한다.25 실시간성이 담보되지 않는 솔루션은 안전에 민감한 애플리케이션에서는 무용지물이다.


LiDAR 시장의 가치를 평가하고 투자 결정을 내리는 전략가와 투자자는 다음과 같은 관점을 견지해야 한다.

- **하드웨어 사양을 넘어서라:** 기업을 평가할 때, 단순히 센서의 탐지 거리나 해상도와 같은 하드웨어 사양에만 집중해서는 안 된다. AI 기반 완화 및 복구 능력을 포함한 전체 인식 스택의 강점과 소프트웨어 로드맵을 함께 평가해야 한다. 차세대 레이저 기술만큼이나 차세대 AI 완화 알고리즘이 기업의 미래를 좌우할 것이다.
- **지적 재산(IP) 포트폴리오를 면밀히 검토하라:** 독창적인 변조 방식, 정밀 동기화 기술, 고유한 AI 알고리즘 등에 대한 강력하고 방어 가능한 특허 포트폴리오는 장기적인 경쟁 우위의 핵심 지표다. 최근의 특허 소송 동향은 IP가 시장의 판도를 바꿀 수 있는 강력한 무기임을 보여준다.
- **생태계 플레이어를 식별하라:** 미래는 개별 LiDAR 센서가 아닌, 이를 둘러싼 생태계의 경쟁이다. NVIDIA, Ambarella와 같은 칩 제조사, 소프트웨어 플랫폼 기업, 그리고 Continental, Bosch와 같은 Tier-1 공급업체와 강력한 파트너십을 구축하는 기업이 시장에서 성공할 가능성이 더 높다.65
- **'간섭 관리'로의 전환을 주시하라:** 승리하는 패러다임은 간섭을 단순히 '회피'하는 것이 아니라 '관리'하는 것이다. 깨지기 쉬운 회피 전용 시스템을 구축하는 기업보다, 변화하는 환경에 적응하고 스스로를 복원하는 강건한 시스템을 만드는 기업, 즉 간섭 관리에 대한 깊은 이해를 보여주는 기업에 주목해야 한다.


1. True-random LiDAR에 대한 상호간섭 영향 분석 - DSpace at KOASAS, accessed July 22, 2025, https://koasas.kaist.ac.kr/handle/10203/284448
2. True-random LiDAR에 대한 상호간섭 영향 분석 = Analysis of the impacts of mutual interference on the true-random LiDAR - 카이스트 도서관, accessed July 22, 2025, http://library.kaist.ac.kr/search/detail/view.do?bibCtrlNo=924536&flag=dissertation
3. Effects and conditions of mutual interference in FMCW lidars, accessed July 22, 2025, https://www.spiedigitallibrary.org/conference-proceedings-of-spie/13472/134720I/Effects-and-conditions-of-mutual-interference-in-FMCW-lidars/10.1117/12.3053799.full?webSyncID=9d902bc9-993c-ed26-8bea-d49a28531ab0&sessionGUID=d3a6b03c-97c0-09b6-bcd7-46b80d8e8abe
4. (PDF) Real-time Mitigation of LiDAR Mutual Interference, accessed July 22, 2025, https://www.researchgate.net/publication/392872786_Real-time_Mitigation_of_LiDAR_Mutual_Interference
5. Do multiple LIDAR systems in same area interfere? - Robotics Stack Exchange, accessed July 22, 2025, https://robotics.stackexchange.com/questions/10770/do-multiple-lidar-systems-in-same-area-interfere
6. LiDAR Sensor Interference Prevention - Xray - GreyB, accessed July 22, 2025, https://xray.greyb.com/lidar/interference-avoidance
7. ADAS 및 LiDAR과의 관계 이해 - Neuvition | 고체 라이더, 라이더 센서 공급 업체, 라이더 기술, 라이더 센서, accessed July 22, 2025, https://www.neuvition.com/ko/media/blog/understanding-adas-and-its-relationship-with-lidar.html
8. Generative AI for Autonomous Driving: Frontiers and Opportunities - arXiv, accessed July 22, 2025, https://arxiv.org/html/2505.08854v1
9. Ambient light immunity of a frequency-modulated continuous-wave (FMCW) LiDAR chip, accessed July 22, 2025, https://opg.optica.org/oe/abstract.cfm?URI=oe-32-3-3997
10. CFAR-Based Interference Mitigation for FMCW Automotive Radar Systems - TU Delft Research Portal, accessed July 22, 2025, https://research.tudelft.nl/files/154907760/CFAR_Based_Interference_Mitigation_for_FMCW_Automotive_Radar_Systems.pdf
11. Frequency Modulation Control of an FMCW LiDAR Using a Frequency-to-Voltage Converter, accessed July 22, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10223902/
12. Coherent Random-Modulated Continuous-Wave LiDAR Based on ..., accessed July 22, 2025, https://www.mdpi.com/2304-6732/8/11/475
13. Why have 905 and 1550 nm become the standard for LiDARs? - Inertial Labs, accessed July 22, 2025, https://inertiallabs.com/why-have-905-and-1550-nm-become-the-standard-for-lidars/
14. Advances in LiDAR Hardware Technology: Focus on Elastic LiDAR for Solid Target Scanning - MDPI, accessed July 22, 2025, https://www.mdpi.com/1424-8220/24/22/7268
15. LiDAR Comparison: 905nm vs. 1550nm - Lumimetric, accessed July 22, 2025, https://www.lumimetric.com/en/new/905nm-and-1550nm-LiDAR-Laser-Comparison.html
16. Time of Flight vs. FMCW LiDAR: A Side-by-Side Comparison - AEye, accessed July 22, 2025, https://www.aeye.ai/resources/white-papers/time-of-flight-vs-fmcw-lidar-a-side-by-side-comparison/
17. AND90257 - Analytical Evaluation of Signal to Noise Ratios for SiPM and APD in dToF LiDAR Applications - Onsemi, accessed July 22, 2025, https://www.onsemi.com/download/application-notes/pdf/and90257-d.pdf
18. Luminar, accessed July 22, 2025, https://www.luminartech.com/
19. Comparison among various LiDAR technologies. - ResearchGate, accessed July 22, 2025, https://www.researchgate.net/figure/Comparison-among-various-LiDAR-technologies_tbl3_370323659
20. 2-D Optical-CDMA Modulation in Automotive Time-of-Flight LIDAR Systems - University of Strathclyde, accessed July 22, 2025, https://pureportal.strath.ac.uk/files/105854771/Kwong_etal_ICTON_2020_2_D_optical_CDMA_modulation_in_automotive_time_of_flight.pdf
21. Time-division multiple access - Wikipedia, accessed July 22, 2025, https://en.wikipedia.org/wiki/Time-division_multiple_access
22. Interference Avoidance for Two Time-of-Flight Cameras Using ..., accessed July 22, 2025, https://www.researchgate.net/publication/341923604_Interference_Avoidance_for_Two_Time-of-Flight_Cameras_Using_Autonomous_Optical_Synchronization
23. Analyses of Time Division Multiple Access (TDMA) Schemes for Global Mobile Satellite Communications (GMSC) - TransNav Journal, accessed July 22, 2025, https://www.transnav.eu/Article2_Analyses_of__Time_Division_Multiple_Access_(TDMA)_Schemes_for_Global_Mobile_Satellite_Communications_(GMSC)_Ilcev,56,1066.html
24. What is Time Division Multiple Access (TDMA) in the World of GNSS/GPS - Novotech, accessed July 22, 2025, https://novotech.com/pages/time-division-multiple-access-tdma
25. D2SR: Decentralized detection, de-synchronization, and recovery of LiDAR interference - InK@SMU.edu.sg, accessed July 22, 2025, https://ink.library.smu.edu.sg/cgi/viewcontent.cgi?article=10227&context=sis_research
26. The Future of AI Using LiDAR | Dell Technologies Info Hub, accessed July 22, 2025, https://infohub.delltechnologies.com/p/the-future-of-ai-using-lidar/
27. Drone Wars: Developments in Drone Swarm Technology - Defense ..., accessed July 22, 2025, https://dsm.forecastinternational.com/2025/01/21/drone-wars-developments-in-drone-swarm-technology/
28. PTP Stuck in UNCALIBRATED - Hardware - Ouster | Community ..., accessed July 22, 2025, https://community.ouster.com/t/ptp-stuck-in-uncalibrated/99
29. 分類対照ツール, accessed July 22, 2025, [https://www.jpo.go.jp/cgi/cgi-bin/search-portal/narabe_tool/table.cgi?keyword=H04B&type=](https://www.jpo.go.jp/cgi/cgi-bin/search-portal/narabe_tool/table.cgi?keyword=H04B&type)
30. 정밀 시각 프로토콜 - 위키백과, 우리 모두의 백과사전, accessed July 22, 2025, [https://ko.wikipedia.org/wiki/%EC%A0%95%EB%B0%80_%EC%8B%9C%EA%B0%81_%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C](https://ko.wikipedia.org/wiki/정밀_시각_프로토콜)
31. Precision Time Protocol - Wikipedia, accessed July 22, 2025, https://en.wikipedia.org/wiki/Precision_Time_Protocol
32. What Is IEEE 1588v2? Why PTPv2 Is Needed? - FS Community, accessed July 22, 2025, https://community.fs.com/blog/ieee-1588v2.html
33. 데이터 수집 및 테스트를 위한 PTP(Precision Time Protocol) - HBM, accessed July 22, 2025, https://www.hbm.com/kr/5143/precision-time-protocol/
34. PTP Profiles Guide - Ouster Sensor Docs documentation, accessed July 22, 2025, https://static.ouster.dev/sensor-docs/image_route1/image_route2/appendix/ptp-profile-guide.html
35. High-precision Time Synchronization with PTP and gPTP | ZHAW Institute of Embedded Systems InES, accessed July 22, 2025, https://www.zhaw.ch/en/engineering/institutes-centres/ines/communication-network-engineering/high-precision-time-synchronization-with-ptp-and-gptp-new-page
36. P802.1DG – TSN Profile for Automotive In-Vehicle Ethernet ..., accessed July 22, 2025, https://1.ieee802.org/tsn/802-1dg/
37. High-precision Time Synchronization with PTP and gPTP | ZHAW ..., accessed July 22, 2025, https://www.zhaw.ch/en/engineering/institutes-centres/ines/communication-network-engineering/high-precision-time-synchronization-with-ptp-and-gptp-new-page/
38. Implementation and Analysis of IEEE 1588 PTP Daemon Based on Embedded System, accessed July 22, 2025, https://www.researchgate.net/publication/344766102_Implementation_and_Analysis_of_IEEE_1588_PTP_Daemon_Based_on_Embedded_System
39. 1.3.3. Time Synchronization Instructions - Livox wiki - Read the Docs, accessed July 22, 2025, https://livox-wiki-en.readthedocs.io/en/latest/tutorials/new_product/common/time_sync.html
40. Time Synchronization using PTP/IEEE1588 - Ruixiang's Notes, accessed July 22, 2025, https://notes.rdu.im/system/network/time-sync-using-ptp/
41. Multi-LiDAR time synchronization with phase_lock - Software ..., accessed July 22, 2025, https://community.ouster.com/t/multi-lidar-time-synchronization-with-phase-lock/323
42. Lidar using ptp only at the start to synchronize timestamp - Forum, accessed July 22, 2025, https://forum.ouster.at/d/142-lidar-using-ptp-only-at-the-start-to-synchronize-timestamp
43. Synchronization of cameras and lidar with a frame trigger - DRIVE AGX Orin General, accessed July 22, 2025, https://forums.developer.nvidia.com/t/synchronization-of-cameras-and-lidar-with-a-frame-trigger/305527
44. Algorithm for Point Cloud Dust Filtering of LiDAR for Autonomous Vehicles in Mining Area, accessed July 22, 2025, https://www.mdpi.com/2071-1050/16/7/2827
45. Mastering LiDAR with DJI Enterprise: An Introductory Booklet - Insights, accessed July 22, 2025, https://enterprise-insights.dji.com/blog/lidar-basic-guide
46. An Interference Mitigation Method for FMCW Radar Based on Time–Frequency Distribution and Dual-Domain Fusion Filtering - MDPI, accessed July 22, 2025, https://www.mdpi.com/1424-8220/24/11/3288
47. Real-Time Interference Mitigation for Reliable Target Detection with FMCW Radar in Interference Environments - MDPI, accessed July 22, 2025, https://www.mdpi.com/2072-4292/17/1/26
48. Inducing Noise Tolerance through Adversarial Curriculum Training for LiDAR-based Safety-Critical Perception and Autonomy - arXiv, accessed July 22, 2025, https://arxiv.org/pdf/2502.1896
49. TexLiDAR: Automated Text Understanding for Panoramic LiDAR Data, accessed July 22, 2025, https://www.arxiv.org/pdf/2502.04385
50. CPC COOPERATIVE PATENT CLASSIFICATION, accessed July 22, 2025, https://www.cooperativepatentclassification.org/sites/default/files/cpc/scheme/H/scheme-H04B.pdf
51. Classification Explorer CPC - PatBase, accessed July 22, 2025, [http://www.patbase.com/linkclass.asp?CCC=CPC&CLASS=H04B7%2F04](http://www.patbase.com/linkclass.asp?CCC=CPC&CLASS=H04B7/04)
52. Hesai Hyper-Hemispherical Mini 3D Lidar for Robotics Unveiled at CES - Highways Today, accessed July 22, 2025, https://highways.today/2025/01/14/hesai-mini-3d-lidar/
53. Hesai Unveils Mini Hyper-Hemispherical 3D Lidar Series for Robotics at CES, accessed July 22, 2025, https://www.hesaitech.com/hesai-unveils-mini-hyper-hemispherical-3d-lidar-series-for-robotics-at-ces-hesai/
54. Hesai Secures ADAS Lidar Design Win for Multiple New Vehicles with SAIC Motor, accessed July 22, 2025, https://www.hesaitech.com/hesai-secures-adas-lidar-design-win-for-multiple-new-vehicles-with-saic-motor/
55. Patents Assigned to HESAI TECHNOLOGY CO., LTD., accessed July 22, 2025, https://patents.justia.com/assignee/hesai-technology-co-ltd
56. US10444356B2 - Lidar system and method - Google Patents, accessed July 22, 2025, https://patents.google.com/patent/US10444356B2
57. US20240036202A1 - Lidars and ranging methods - Google Patents, accessed July 22, 2025, https://patents.google.com/patent/US20240036202A1/zh
58. April 2025 Wrap-Up - Luminar, accessed July 22, 2025, https://www.luminartech.com/updates/april-2025-wrap-up
59. Luminar Technologies: Q2 2025 Update Reveals Progress and Persistent Challenges in the LiDAR Race - AInvest, accessed July 22, 2025, https://www.ainvest.com/news/luminar-technologies-q2-2025-update-reveals-progress-persistent-challenges-lidar-race-2507/
60. Ouster Digital Lidar Approved by Defense Department for Unmanned Aircraft, accessed July 22, 2025, https://investors.ouster.com/node/10906/pdf
61. Ouster Announces Strong Operating Results for First Quarter 2025, accessed July 22, 2025, https://investors.ouster.com/node/10811/pdf
62. ITC Institutes Investigation into Hesai's Unlawful Use of Ouster's Patents | Thu, 05/18/2023, accessed July 22, 2025, https://investors.ouster.com/news-releases/news-release-details/itc-institutes-investigation-hesais-unlawful-use-ousters-patents
63. Alpha Prime (VLS-128) - Mapix technologies, accessed July 22, 2025, https://www.mapix.com/lidar-scanner-sensors/velodyne/alpha-prime/
64. Velodyne Lidar Debuts Alpha Prime, the Most Advanced Lidar Sensor on the Market, accessed July 22, 2025, https://lidarmag.com/2019/11/15/velodyne-lidar-debuts-alpha-prime-the-most-advanced-lidar-sensor-on-the-market/
65. How Continental is Shaping the Future of Autonomous Mobility, accessed July 22, 2025, https://www.continental-automotive.com/en/news/2025/06-17-one-plus-one-equals-three-continental-autonomous-mobility.html
66. CES 2025, Part 2: Power, sensors and wireless - Electronic Products, accessed July 22, 2025, https://www.electronicproducts.com/ces-2025-part-2-power-sensors-and-wireless/
67. IV 2025 Program | Wednesday June 25, 2025 - ITS Papercept, accessed July 22, 2025, https://its.papercept.net/conferences/scripts/rtf/IV25_ContentListWeb_5.html
68. arXiv:2311.14546v2 [quant-ph] 8 Jan 2025, accessed July 22, 2025, https://arxiv.org/pdf/2311.14546
69. IEEE 1588 Working Group - Documents, accessed July 22, 2025, https://sagroups.ieee.org/1588/public-documents/
70. News - IEEE 1588-2019 Version 2.1 officially published - Oregano Systems, accessed July 22, 2025, https://www.oreganosystems.at/about-us/news/ieee-1588-2019-version-21-officially-published
71. IEEE 802.1DG-2025, accessed July 22, 2025, https://standards.ieee.org/ieee/802.1DG/7480/