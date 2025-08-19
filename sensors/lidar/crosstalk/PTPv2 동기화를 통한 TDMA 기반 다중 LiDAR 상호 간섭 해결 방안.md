# PTPv2 동기화를 통한 TDMA 기반 다중 LiDAR 상호 간섭 해결 방안

## 서론: 자율주행 시대, LiDAR 간섭 문제의 부상

Light Detection And Ranging (LiDAR) 센서는 정밀한 3차원 공간 정보를 제공하는 능력 덕분에 자율주행 시스템의 핵심 센서로 자리매김했다.1 레이저 펄스를 이용하여 주변 환경과의 거리를 정확하게 측정하고, 이를 통해 고해상도 포인트 클라우드(Point Cloud)를 생성함으로써, 차량은 객체를 인식하고, 위치를 파악하며, 안전한 주행 경로를 계획할 수 있다. 그러나 자율주행 기술이 발전하고 도로 위에 LiDAR를 장착한 차량의 수가 기하급수적으로 증가함에 따라, 과거에는 크게 고려되지 않았던 새로운 기술적 난제가 수면 위로 떠올랐다. 바로 ‘LiDAR 상호 간섭(Mutual Interference)’ 문제이다.1

다수의 LiDAR가 밀집된 환경에서 동시에 작동할 때, 한 LiDAR에서 방출된 레이저 펄스가 다른 LiDAR의 수신기에 감지되면서 데이터에 심각한 오류를 유발할 수 있다. 이는 최악의 경우, 반드시 인지해야 할 보행자나 다른 차량을 놓치거나, 존재하지 않는 장애물을 인식하는 ‘허상(Ghost)’을 만들어내어 치명적인 사고로 이어질 수 있다.1 따라서 안전하고 신뢰도 높은 자율주행 시스템을 구현하기 위해 LiDAR 상호 간섭 문제는 반드시 해결해야 할 선결 과제이다.

현재까지 코드 분할 다중 접속(CDMA), 주파수 변조(FMCW), 신호 후처리 등 다양한 간섭 완화 기술이 제안되었지만, 각각 장단점과 기술적 한계를 가지고 있다. 본 보고서는 이 문제에 대한 해결책 중 하나로, 정밀 시각 프로토콜 버전 2(Precision Time Protocol v2, PTPv2)를 이용한 시간 동기화를 기반으로 시분할 다중 접속(Time-Division Multiple Access, TDMA) 방식을 적용하는 기술에 대해 심층적으로 고찰하고자 한다. TDMA의 근본 원리부터 PTPv2의 정밀 동기화 메커니즘, 두 기술의 시너지와 실제 구현 사례, 그리고 다른 대안 기술과의 종합적인 비교 분석을 통해 이 접근법의 기술적 타당성, 장점, 그리고 실용적 한계를 명확히 규명하는 것을 목표로 한다.

## 1.  다중 LiDAR 시스템의 상호 간섭: 근본 원인과 정량적 영향 분석

### 1.1  LiDAR 신호의 직교성 부재와 간섭 발생 메커니즘

상용 LiDAR 시스템, 특히 시장의 주류를 이루는 ToF(Time-of-Flight) 방식 LiDAR에서 상호 간섭이 발생하는 근본적인 이유는 신호 간의 ‘직교성(Orthogonality)’이 보장되지 않기 때문이다.1 대부분의 ToF LiDAR는 동일한 파장(예: 905nm)과 유사한 형태의 짧은 레이저 펄스를 방출한다. 수신기는 레이저 펄스가 방출된 후 되돌아오기까지의 시간을 측정하여 거리를 계산하는데, 이때 수신된 펄스가 자신이 보낸 것인지, 아니면 근처의 다른 LiDAR가 보낸 것인지를 명확히 구분할 방법이 없다. 이는 마치 여러 사람이 동시에 똑같은 단어를 외칠 때, 누가 외친 소리인지 구분하기 어려운 것과 같은 원리이다.

이러한 간섭은 도로 위의 다른 차량에 장착된 LiDAR뿐만 아니라, 센서 융합을 위해 한 차량에 여러 개의 LiDAR를 장착하는 최근 경향에 따라 동일 차량 내에서도 심각한 문제를 야기할 수 있다.4 미래에 자율주행차가 대중화되어 도로가 LiDAR 센서로 포화 상태가 되면, 이러한 간섭 문제는 시스템의 신뢰성을 위협하는 가장 큰 요인 중 하나가 될 것이다.2

### 1.2  간섭의 유형 및 포인트 클라우드에 미치는 영향

LiDAR 상호 간섭은 포인트 클라우드 데이터에 두 가지 치명적인 형태로 영향을 미친다.

첫째, **허상(Ghost) 포인트 생성**이다. 이는 다른 LiDAR(간섭원)에서 방출된 레이저 펄스를 자신의 수신기(피해자)가 수신하고, 이를 자신이 보낸 펄스가 특정 지점에서 반사되어 돌아온 것으로 오인하는 경우에 발생한다.3 이 경우, 시스템은 실제로는 아무것도 없는 공간에 장애물이 존재하는 것처럼 잘못된 3D 포인트를 생성하게 된다. 이러한 허상 포인트는 자율주행 시스템의 인지 알고리즘에 혼란을 주어 급제동이나 불필요한 회피 기동과 같은 오작동을 유발할 수 있다.2

둘째, **신호 대 잡음비(SNR, Signal-to-Noise Ratio) 저하 및 탐지 가능 영역 감소**이다. 간섭 펄스들은 시스템에 배경 노이즈(background noise)로 작용한다. 이 노이즈 레벨이 높아지면, 멀리 있는 물체에서 반사되어 돌아온 미약한 실제 신호가 노이즈에 묻혀버려 구분하기 어려워진다.2 이는 곧 SNR의 저하를 의미하며, 결과적으로 LiDAR의 최대 탐지 거리를 감소시킨다. 심각한 경우, 간섭으로 인해 특정 방향의 감지가 완전히 불가능해지는 ‘센서 블라인딩(blinding)’ 현상이 발생하여, 눈앞의 보행자나 차량, 벽과 같은 핵심적인 장애물을 인식하지 못하는 최악의 상황을 초래할 수 있다.1

이러한 영향은 정량적으로도 매우 심각하다. 연구에 따르면, 펄스당 10~20 포톤 수준의 매우 미약한 간섭만으로도 거리 측정 정확도가 수 센티미터(cm)나 저하될 수 있으며, 여러 간섭원이나 직사광선과 같은 강한 간섭은 센서를 완전히 무력화시킬 수 있다.5 또 다른 연구에서는 간섭이 발생했을 때 깊이 추정 오차(RMSE)가 기준선(baseline)의 38.7 cm에서 60.6 cm로 약 57%나 급증하는 것을 실험적으로 보여주었다.6

### 1.3  FMCW LiDAR의 특수 간섭 조건: 세 가지 정렬

최근 주목받는 FMCW(Frequency-Modulated Continuous-Wave) LiDAR는 코히런트(coherent) 검출 방식을 사용하여 일반적인 ToF LiDAR보다 간섭에 강한 것으로 알려져 있다.2 하지만 FMCW LiDAR 역시 완벽하게 간섭으로부터 자유로운 것은 아니며, 다음과 같은 세 가지 조건이 동시에 충족될 때 간섭이 발생할 확률이 높아진다.2

1. **공간적 정렬 (Spatial Alignment):** 두 LiDAR의 레이저 빔이 물리적으로 같은 공간을 통과하거나 같은 지점을 조준할 때 발생한다. 간섭원의 빔이 피해자의 수신기에 직접 들어오거나, 두 빔이 동일한 객체에서 반사되어 피해자 수신기로 들어가는 경우를 모두 포함한다.2
2. **스펙트럼적 정렬 (Spectral Alignment):** 두 LiDAR가 사용하는 레이저의 중심 파장과 주파수 변조(처프, chirp) 파라미터(대역폭, 스윕 속도 등)가 매우 유사하여, 간섭 신호가 피해 LiDAR의 광학 필터 대역을 통과할 때 발생한다.2
3. **시간적 정렬 (Temporal Alignment):** 간섭 신호가 피해 LiDAR가 신호를 수신하도록 설정된 시간 창(time window 또는 range gate) 내에 도달할 때 발생한다.2

이 세 가지 조건이 우연히 일치하고 간섭 신호의 세기가 충분히 강할 때, FMCW LiDAR에서도 허상이나 SNR 저하와 같은 간섭 현상이 나타날 수 있다. 이는 간섭 문제가 특정 기술 방식에 국한된 것이 아니라, 다수의 센서가 동일한 물리적 매체를 공유할 때 발생하는 보편적인 문제임을 시사한다. 결국, 간섭은 항상 발생하는 결정론적 현상이 아니라, 여러 독립적인 조건이 맞아떨어질 때 발생하는 '확률적 사건'으로 이해해야 한다. 이러한 관점은 간섭 문제를 해결하기 위한 전략 수립에 중요한 시사점을 제공한다. 즉, 간섭을 100% 원천 봉쇄하는 결정론적 접근법(예: TDMA)뿐만 아니라, 간섭 발생 '확률' 자체를 무시할 수 있을 만큼 낮추는 확률론적 접근법(예: RMCW) 역시 매우 유효한 해결책이 될 수 있음을 의미한다.

## 2.  시분할 다중 접속(TDMA)의 원리 및 간섭 회피 메커니즘

### 2.1  TDMA의 기본 개념: 시간 슬롯 할당

시분할 다중 접속(Time-Division Multiple Access, TDMA)은 다수의 사용자가 하나의 공유된 통신 채널을 간섭 없이 사용하기 위한 고전적이면서도 강력한 다중 접속 기술이다. 그 핵심 원리는 매우 간단하다: **시간을 나누어 쓰는 것**이다.8 TDMA는 전체 시간을 아주 작은 단위의 '시간 슬롯(time slot)'으로 잘게 나누고, 각각의 LiDAR 센서에게 고유한 시간 슬롯을 독점적으로 할당한다.

LiDAR 시스템에 이를 적용하면, 각 LiDAR는 오직 자신에게 할당된 시간 슬롯 동안에만 레이저를 방출하고 반사 신호를 수신한다. 다른 LiDAR들은 그 시간 동안에는 완전히 침묵(idle) 상태를 유지한다. 이 과정을 모든 LiDAR가 순차적으로 반복함으로써, 물리적으로 두 개 이상의 LiDAR 펄스가 동시에 공기 중에 존재하는 상황 자체를 원천적으로 차단한다.8 이는 마치 여러 사람이 돌아가면서 한 명씩 발언권을 얻어 말하는 것과 같아서, 대화가 겹치거나 혼선을 빚을 일이 원천적으로 사라지는 것과 동일한 효과를 낸다.

### 2.2  TDMA의 명확한 장점과 치명적 단점

TDMA 방식은 간섭 회피라는 측면에서 매우 명확한 장점과 그에 상응하는 치명적인 단점을 동시에 가지고 있다.

**장점 (Pros):**

- **완벽한 간섭 회피:** 이론적으로, 모든 LiDAR의 시간 동기화가 완벽하게 이루어진다면 시간 슬롯은 절대로 겹치지 않는다. 따라서 상호 간섭을 100% 회피할 수 있다. 이는 복잡한 신호 처리나 필터링 없이도 매우 높은 통신 및 감지 신뢰성을 보장한다.9
- **구현의 단순성:** CDMA처럼 복잡한 코드를 설계하거나 FMCW처럼 주파수를 정교하게 제어할 필요 없이, 오직 '시간 축'만을 제어하면 되므로 개념적으로 매우 단순하고 직관적이다.8

**단점 (Cons):**

- **프레임 속도 및 데이터 처리량 저하:** 이것이 TDMA의 가장 치명적인 약점이다. 시스템에 N개의 LiDAR가 있다면, 각 LiDAR는 전체 시간의 1/N만큼만 작동할 수 있다. 이는 곧 각 LiDAR의 데이터 갱신 주기(프레임 속도)가 이론적으로 1/N으로 감소함을 의미한다.10 예를 들어, 10Hz로 작동하는 LiDAR 4개를 TDMA로 묶으면 각 LiDAR는 2.5Hz로 작동하게 된다. 고속으로 주행하며 주변 환경을 실시간으로 인지해야 하는 자율주행차에게 이러한 데이터 처리량 감소는 안전에 직결되는 심각한 문제일 수 있다.
- **보호 구간(Guard Intervals)으로 인한 효율 저하:** 서로 다른 시간 슬롯이 미세한 시간 오차로 인해 충돌하는 것을 방지하기 위해, 각 슬롯 사이에는 아무도 데이터를 전송하지 않는 짧은 '보호 구간'을 반드시 두어야 한다. 이 보호 구간은 시스템의 안정성을 위해 필수적이지만, 그 자체로는 아무런 정보를 전달하지 않는 낭비되는 시간이므로 전체 시스템의 유효 데이터 처리량(throughput)을 추가로 감소시키는 요인이 된다.8
- **제한된 유연성:** 모든 LiDAR에 고정된 길이의 시간 슬롯을 할당하는 정적(static) TDMA 방식은 각 LiDAR의 역할이나 중요도 변화에 유연하게 대응하기 어렵다. 예를 들어, 전방의 중요한 객체를 추적하는 LiDAR와 측면의 변화 없는 환경을 감시하는 LiDAR가 동일한 시간 자원을 할당받는 것은 비효율적이다.8

이러한 장단점을 종합해 볼 때, TDMA가 제공하는 '안전(완벽한 간섭 회피)'과 시스템이 얻는 '정보량(데이터 처리량)' 사이에는 명백한 제로섬 게임(zero-sum game) 관계가 성립한다. 안전을 위해 간섭을 완벽히 차단하면 정보량이 줄어들고, 정보량을 늘리기 위해 시간 슬롯을 조밀하게 배치하면 충돌 위험이 커진다. 이 트레이드오프는 TDMA 방식의 본질적인 한계이며, LiDAR 수가 적고(예: 2~3개) 비교적 정적인 환경에서 작동하는 산업용 로봇 등에는 적합할 수 있으나 11, 수많은 LiDAR가 장착되고 예측 불가능한 동적 환경에 놓이는 자율주행차와 같은 복잡한 시스템에는 정적 TDMA 방식이 부적합할 수 있음을 시사한다.

### 2.3  동적 TDMA(Dynamic TDMA)의 가능성

정적 TDMA의 경직성을 극복하기 위한 대안으로 동적 TDMA(Dynamic TDMA)가 제안된다.8 이 방식은 고정된 시간 슬롯을 할당하는 대신, 각 LiDAR의 실시간 데이터 요구량이나 중요도에 따라 시간 슬롯의 수나 길이를 동적으로 조절한다.10

예를 들어, 자율주행차가 고속도로를 직진할 때는 전방을 주시하는 장거리 LiDAR에 더 많은 시간 슬롯을 집중적으로 할당하여 원거리 탐지 능력을 극대화하고, 복잡한 시내 교차로에 진입했을 때는 전방 및 측방 LiDAR에 시간 슬롯을 균등하게 분배하여 주변 상황을 넓게 파악하도록 할 수 있다. 이러한 동적 자원 할당은 시스템 전체의 효율성과 상황 대응 능력을 크게 향상시킬 수 있는 잠재력을 가지고 있다.

하지만 동적 TDMA를 구현하기 위해서는 각 LiDAR의 상태와 주변 환경을 실시간으로 판단하고, 이에 맞춰 시간 슬롯 스케줄을 동적으로 재구성하는 고도의 제어 로직이 필요하다. 그리고 이 모든 과정의 근간에는 TDMA의 가장 근본적인 요구사항, 즉 '정밀한 시간 동기화'가 자리 잡고 있다. TDMA의 이론적 장점은 모든 참여자가 완벽하게 동기화된 시계를 가지고 있다는 가정 하에서만 유효하다. 동기화가 조금이라도 틀어지면 시간 슬롯이 충돌하여 시스템 전체가 무너질 수 있다.8 따라서 TDMA 시스템 구현의 성패는 시간 분할 로직 자체가 아니라, 모든 LiDAR의 클럭을 얼마나 정밀하고, 안정적으로, 그리고 낮은 오버헤드로 맞출 수 있느냐에 달려있다. 바로 이 지점에서 PTPv2 프로토콜의 역할이 결정적으로 중요해진다.

## 3.  정밀 시각 동기화의 핵심: PTPv2 (IEEE 1588) 심층 분석

### 3.1  PTPv2의 작동 원리: 마스터-슬레이브 구조와 메시지 교환

정밀 시각 프로토콜(Precision Time Protocol, PTP)은 이더넷과 같이 패킷 지연이 불규칙한 네트워크 환경에서도 분산된 장치들 간의 시계를 매우 정밀하게 동기화하기 위해 설계된 IEEE 1588 표준 프로토콜이다.12 PTP는 하드웨어의 지원을 받을 경우, 수십 나노초(nanosecond) 수준의 오차 범위 내에서 시간을 일치시킬 수 있어, 통신, 금융, 전력망, 그리고 자율주행과 같은 실시간 제어가 필수적인 시스템에 널리 사용된다.14

PTP의 기본 구조는 마스터-슬레이브(Master-Slave) 방식에 기반한다. 네트워크에 연결된 PTP 지원 장치들은 '최고 마스터 시계 알고리즘(Best Master Clock Algorithm, BMCA)'이라는 절차를 통해 스스로 통신하여 네트워크 내에서 가장 안정적이고 정밀한 시계를 가진 장치를 찾아낸다. 이렇게 선출된 장치가 '그랜드마스터(Grandmaster, GM)' 시계가 되며, 나머지 모든 장치들은 '슬레이브(Slave)'가 되어 이 그랜드마스터의 시간에 자신의 시계를 맞춘다.14

현재 널리 사용되는 버전은 PTPv2로 알려진 IEEE 1588-2008 표준이다. 이는 초기 버전인 PTPv1(IEEE 1588-2002)에 비해 정확성, 정밀성, 그리고 다양한 네트워크 환경에 대한 강건성을 크게 향상시켰다. 다만, PTPv1과는 하위 호환성이 없다.14 가장 최신 버전은 2019년에 발표된 PTPv2.1(IEEE 1588-2019)이다.16

### 3.2  나노초 정밀도의 비밀: 4개의 타임스탬프 (T1, T2, T3, T4)

PTPv2가 나노초 수준의 정밀도를 달성하는 핵심 비결은 마스터와 슬레이브 간에 교환되는 일련의 메시지와, 이 메시지들이 오고 갈 때 찍히는 4개의 정밀한 타임스탬프에 있다. 이 과정을 통해 슬레이브는 자신과 마스터 시계 사이의 시간 차이(offset)와 두 장치 간의 네트워크 전송 지연(path delay)을 정확하게 계산해낼 수 있다.12

동기화 과정은 다음과 같은 2단계 메시지 교환으로 이루어진다 12:

1. **1단계: Sync 메시지 교환**
   - 마스터는 `Sync` 메시지를 네트워크에 브로드캐스트(또는 멀티캐스트)하면서, 메시지를 전송한 정확한 시각 **T1**을 기록한다.
   - 슬레이브는 이 `Sync` 메시지를 수신하고, 메시지를 받은 정확한 시각 **T2**를 자신의 시계 기준으로 기록한다.
   - 이때, 마스터는 `Sync` 메시지 자체에 T1을 포함시키거나(1-step clock), 별도의 `Follow_Up` 메시지를 통해 T1 정보를 전달한다(2-step clock).
2. **2단계: Delay Request-Response 메시지 교환**
   - 슬레이브는 동기화 계산을 위해 `Delay_Req` 메시지를 마스터에게 보내면서, 메시지를 전송한 시각 **T3**를 기록한다.
   - 마스터는 `Delay_Req` 메시지를 수신하고, 메시지를 받은 시각 **T4**를 자신의 시계 기준으로 기록한다.
   - 마스터는 이 T4 정보를 `Delay_Resp` 메시지에 담아 다시 슬레이브에게 보내준다.

이제 슬레이브는 T1, T2, T3, T4라는 4개의 타임스탬프 정보를 모두 확보했다. PTP는 마스터에서 슬레이브로 가는 경로의 지연 시간과 슬레이브에서 마스터로 가는 경로의 지연 시간이 동일하다는 '경로 대칭성(Path Symmetry)'을 가정한다. 이 가정을 바탕으로 슬레이브는 다음 공식을 이용해 오프셋과 지연을 계산한다 12:

- 왕복 지연 (Round Trip Delay) = (T2−T1)+(T4−T3)
- 편도 지연 (One-way Delay, d) = 2(T2−T1)+(T4−T3)
- 마스터-슬레이브 간 시간 오프셋 (Offset, o) = 2(T2−T1)−(T4−T3)

슬레이브는 이렇게 계산된 오프셋 값을 자신의 시계에 적용하여 마스터 시계와 동기화한다. 이 과정은 주기적으로 반복되어, 시계의 미세한 흐름(drift)이나 네트워크 상태 변화로 인한 오차를 지속적으로 보정한다.

### 3.3  PTPv2 구현의 현실적 요구사항

PTPv2의 이론적 정밀도를 현실 세계에서 구현하기 위해서는 몇 가지 중요한 요구사항이 충족되어야 한다.

- **하드웨어 타임스탬핑 (Hardware Timestamping):** 최고 수준의 정밀도를 위해서는 타임스탬프가 소프트웨어(예: 운영체제 커널) 레벨이 아닌, 네트워크 인터페이스 카드(NIC)와 같은 물리적 하드웨어 레벨에서 직접 생성되어야 한다. 이는 운영체제의 스케줄링 지연이나 프로토콜 스택 처리 시간 등 예측 불가능한 소프트웨어 지연 요소를 원천적으로 배제하기 위함이다.12
- **투명 클럭 (Transparent Clock, TC):** 마스터와 슬레이브 사이에 스위치나 라우터 같은 네트워크 장비가 존재할 경우, 이 장비 내부에서 패킷이 머무는 시간(residence time)이 동기화 오차의 주된 원인이 된다. 투명 클럭은 이 지연 시간을 정밀하게 측정하여 PTP 메시지에 누적 기록함으로써, 슬레이브가 종단 간(end-to-end) 지연을 계산할 때 이 오차를 보상할 수 있도록 해주는 기능이다.15 TC 기능을 지원하는 스위치는 동기화 정밀도를 크게 향상시키지만, 일반 스위치보다 가격이 비싸다. Ouster와 같은 LiDAR 제조사들은 정밀 동기화를 위해 PTP 호환 스위치(TC 또는 Boundary Clock 지원) 사용을 명시적으로 권장하고 있다.18
- **PTP 프로파일 (PTP Profiles):** PTP 표준은 다양한 옵션을 제공하는데, 특정 응용 분야의 요구사항에 맞게 이 옵션들을 조합하여 표준화한 것이 '프로파일'이다. 예를 들어, 자동차 이더넷 환경에서는 `gPTP(IEEE 802.1AS)` 프로파일이 주로 사용되며, 이 외에도 `Default`, `Automotive-Slave` 등 다양한 프로파일이 존재한다. 시스템 내의 모든 장치(마스터, 슬레이브, 스위치)는 동일한 프로파일을 사용하도록 설정되어야 원활한 동기화가 가능하다.19

결론적으로, PTPv2의 나노초급 정밀도는 단순히 프로토콜을 지원하는 장치를 사용하는 것만으로 보장되지 않는다. 그 성능은 네트워크 토폴로지, 스위치의 성능(특히 TC 지원 여부), 마스터와 슬레이브 간의 홉(hop) 수, 그리고 네트워크 케이블링 품질과 같은 전체 네트워크 인프라의 질에 강하게 종속된다.17 특히 스위치 내부에서 발생하는 '큐잉 지연 비대칭(Queue-Induced Delay Asymmetry, QIDA)' 문제는 경로 대칭성 가정을 깨뜨려 동기화 정밀도를 저하시키는 주요 요인으로 지목된다.17 따라서 PTPv2 기반의 TDMA 솔루션을 고려할 때는 LiDAR 센서 자체의 비용뿐만 아니라, 차량 내 전체 이더넷 아키텍처를 PTP에 최적화하여 설계하고 구축하는 데 드는 숨겨진 비용과 복잡성을 반드시 함께 평가해야 한다.

## 4.  PTPv2와 TDMA의 결합: 시너지, 구현 및 실증 사례

### 4.1  기술적 시너지: PTPv2가 TDMA를 완성하는 방법

TDMA와 PTPv2는 각기 다른 목적을 가진 기술이지만, 두 기술이 결합될 때 강력한 시너지를 발휘한다. TDMA의 근본적인 전제는 시스템 내 모든 참여자가 '동일한 시간'을 공유하고, 그 시간을 기준으로 자신의 행동(전송) 순서를 결정하는 것이다. 만약 이 '동일한 시간'에 대한 기준이 모호하거나 부정확하다면, TDMA 스케줄은 쉽게 무너지고 시간 슬롯 간의 충돌이 발생하여 시스템 전체의 신뢰성을 잃게 된다.

바로 이 지점에서 PTPv2가 결정적인 역할을 수행한다. PTPv2는 이더넷 네트워크를 통해 모든 LiDAR 센서에게 나노초 수준의 정밀도로 동기화된 공통의 시간 기준을 제공한다.12 즉, PTPv2는 TDMA 스케줄링을 위한 '공통의 박자(metronome)'를 제공하는 핵심 기반 기술이 되는 것이다.

PTPv2를 통해 정밀하게 동기화된 각 LiDAR는 글로벌 타임라인 상에서 자신에게 할당된 레이저 발광(emission) 시작 시점과 종료 시점을 오차 없이 정확하게 제어할 수 있게 된다. 이는 TDMA 시스템 설계 시, 시간 슬롯 간의 충돌을 막기 위해 설정해야 하는 보호 구간(guard interval)을 최소화하면서도 안정적인 운영을 가능하게 한다. 결과적으로 PTPv2는 TDMA의 이론적 장점인 완벽한 간섭 회피를 현실 세계에서 구현 가능하게 만드는 동시에, 보호 구간으로 인한 비효율을 줄여 시스템 성능을 최적화하는 데 기여한다.

### 4.2  구현 아키텍처 및 설정

PTPv2로 동기화된 TDMA 시스템의 일반적인 구현 아키텍처는 다음과 같은 요소로 구성된다.

1. **그랜드마스터(GM) 클럭 설정:** 차량의 중앙 처리 장치(ECU), 이더넷 스위치, 또는 전용 타임 서버 중 하나가 PTP 그랜드마스터 역할을 맡는다. 이 GM은 시스템 전체 시간의 기준점이 된다. 외부 시간 소스(예: GPS 수신기에서 제공하는 PPS 신호)와 동기화하여 절대 시간을 기준으로 삼거나, 외부 소스가 없을 경우 자체적으로 가장 안정적인 클럭을 가진 장치가 GM으로 선출되어 상대적인 동기화를 수행한다.19
2. **LiDAR 슬레이브 동기화:** 네트워크에 연결된 PTP 지원 LiDAR 센서들은 별도의 복잡한 설정 없이도 네트워크상의 PTP 마스터를 자동으로 감지하고 슬레이브 모드로 전환된다. 이후 주기적인 PTP 메시지 교환을 통해 자신의 내부 시계를 GM의 시간에 지속적으로 동기화한다. Livox, Ouster와 같은 최신 LiDAR 제품들은 이러한 자동 동기화 기능을 기본적으로 지원한다.19
3. **TDMA 스케줄러:** 모든 LiDAR가 PTP를 통해 하나의 시간 축을 공유하게 되면, 'TDMA 스케줄러'가 각 LiDAR의 발광 시간 슬롯을 할당하고 관리한다. 이 스케줄러는 중앙 컨트롤러(ECU)에 구현되어 모든 LiDAR의 스케줄을 통합 관리하는 중앙 집중형 방식일 수도 있고, 각 LiDAR가 PTP 시간을 기준으로 사전에 정의된 규칙에 따라 스스로 자신의 발광 타이밍을 결정하는 분산형 방식일 수도 있다.

### 4.3  실증 사례 분석: Ouster의 Phase Lock 기능

PTPv2 동기화 기반의 시간 분할 기술이 실제로 어떻게 적용될 수 있는지를 보여주는 좋은 사례는 Ouster LiDAR의 'Phase Lock' 기능이다.24 이 기능은 여러 개의 Ouster LiDAR를 동기화하여 사용할 때, 각 센서의 프레임 시작 시점을 의도적으로 다르게 설정할 수 있도록 해준다.

예를 들어, 차량의 전후좌우에 4개의 LiDAR를 90도 간격으로 설치했다고 가정하자. 각 LiDAR에 `phase lock offset` 값을 각각 0, 90000, 180000, 270000 마이크로초(μs)로 설정하면, 각 LiDAR는 자신의 회전 각도가 0도, 90도, 180도, 270도에 도달했을 때 프레임 데이터를 생성하도록 시작 시점이 순차적으로 지연된다. 이는 결과적으로 각 LiDAR가 레이저를 집중적으로 발사하는 시간을 시간 축 상에서 분리시키는 효과를 가져온다. 즉, 한 LiDAR가 특정 방향을 스캔하는 동안 다른 LiDAR는 다른 방향을 스캔하게 되어, 직접적인 빔 충돌 가능성을 크게 낮춘다.

이 기능은 순수한 의미의 TDMA(한 센서가 작동할 때 다른 모든 센서가 멈추는)와는 다르지만, PTPv2를 통해 정밀하게 동기화된 시간을 기반으로 각 센서의 작동 타이밍을 조절하여 간섭을 회피한다는 점에서 TDMA의 실용적인 변형으로 볼 수 있다.24 Ouster의 Phase Lock 기능은 본래 여러 LiDAR의 시야가 겹치는 영역에서 데이터의 시간적 정렬을 맞추어 인식 성능을 높이는 것이 주 목적이지만, 부수적으로 간섭 회피 효과까지 얻을 수 있는 것이다.24 이는 PTPv2와 같은 정밀 시간 동기화 기술이 단순히 시간을 맞추는 것을 넘어, 다중 센서 시스템의 성능을 최적화하는 다양한 응용의 기반이 될 수 있음을 보여준다.

### 4.4  실용적 과제 및 고려사항

PTPv2-TDMA 결합 방식은 이론적으로 강력하지만, 실제 차량 환경에 적용하기 위해서는 몇 가지 현실적인 과제를 고려해야 한다.

- **네트워크 부하 및 지터:** PTP 동기화 메시지 자체는 네트워크 대역폭을 많이 차지하지 않는다.14 그러나 고해상도 LiDAR는 센서당 수십에서 수백 Mb/s에 달하는 막대한 양의 데이터를 생성한다.25 이 대량의 데이터 트래픽이 PTP 제어 메시지와 동일한 이더넷 네트워크를 공유할 경우, 스위치에서 병목 현상이 발생하여 PTP 패킷의 전송 지연 및 지터(jitter, 지연 변동)가 커질 수 있다. 이는 PTP의 경로 대칭성 가정을 깨뜨려 동기화 정밀도를 저하시키는 요인이 된다.
- **동기화 안정성:** 차량이 터널에 진입하여 GPS 신호가 끊기거나, 네트워크 스위치의 일시적인 오류로 PTP 메시지가 유실될 경우, PTP 동기화 상태가 불안정해지거나 `UNCALIBRATED` 상태에 빠질 수 있다.18 이 경우 TDMA 스케줄이 무너져 일시적으로 간섭이 발생할 수 있으므로, 시스템은 이러한 예외 상황에 대처할 수 있는 강건한 로직을 갖추어야 한다.
- **확장성 및 복잡성:** 차량에 장착되는 LiDAR, 카메라, 레이더 등 시간 동기화가 필요한 센서의 수가 증가할수록, TDMA의 시간 슬롯은 더욱 잘게 쪼개져야 하고 PTP 네트워크 토폴로지는 복잡해진다. 이는 시스템 전체의 관리 복잡성과 비용을 증가시키는 요인이 된다.

결론적으로, PTPv2-TDMA 방식은 간섭 회피를 위한 강력한 솔루션이지만, 그 성능은 네트워크 인프라의 품질과 시스템 전체의 통합 설계 수준에 크게 좌우된다. 따라서 성공적인 도입을 위해서는 센서 자체의 성능뿐만 아니라 차량 내 이더넷 아키텍처 전반에 대한 깊은 이해와 신중한 설계가 필수적이다.



## 5.  종합적 비교 분석: PTPv2-TDMA 방식과 타 간섭 완화 기술



PTPv2-TDMA 방식의 효용성을 객관적으로 평가하기 위해서는 다른 대안 기술들과의 다각적인 비교가 필수적이다. 간섭 완화 기술은 크게 신호 변조 방식, 하드웨어 기반 방식, 그리고 소프트웨어 후처리 방식으로 나눌 수 있으며, 각각의 접근법은 성능, 시스템 영향, 복잡도, 비용 측면에서 뚜렷한 장단점을 가진다.



### 5.1  LiDAR 상호 간섭 완화 기술 비교표



아래 표는 주요 간섭 완화 기술들의 핵심적인 특징을 요약하여 비교한 것이다.

| 기술 (Technique)                    | 완화 원리 (Mitigation Principle)                             | 성능/효과 (Performance/Effectiveness)  | 시스템 영향 (System Impact)       | 복잡도 (Complexity)                    | 비용 (Cost Implication) |
| ----------------------------------- | ------------------------------------------------------------ | -------------------------------------- | --------------------------------- | -------------------------------------- | ----------------------- |
| **PTPv2-TDMA**                      | 시간 분할 (Temporal Separation)                              | 완벽한 간섭 회피 (이론상) 9            | 데이터 처리량/프레임 속도 감소 10 | H/W: 중(PTP 스위치), S/W: 중(스케줄러) | 중 (PTP 인프라) 17      |
| **CDMA**                            | 코드 직교성 (Code Orthogonality)                             | 높은 간섭 억제 27                      | 고속 디지털 신호 처리 요구 1      | H/W: 고, S/W: 고                       | 고                      |
| **RMCW (PRBS)**                     | 신호 무작위성 (Signal Randomness)                            | 매우 높은 간섭 억제 (면역에 가까움) 28 | 상관관계(Correlation) 처리 부하   | H/W: 중, S/W: 중                       | 중                      |
| **True-Random LiDAR**               | 물리적 무작위성 (Physical Randomness)                        | 매우 높은 간섭 억제 1                  | 아날로그 상관기 필요              | H/W: 고, S/W: 중                       | 고                      |
| **FMCW (Frequency Hopping 등)**     | 스펙트럼/시간 무작위화 (Spectral/Temporal Randomization)     | 높은 간섭 억제 2                       | 시스템 복잡도 및 비용 증가 2      | H/W: 고, S/W: 중                       | 고                      |
| **후처리 필터링 (Post-processing)** | 통계/기하학적 노이즈 제거 (Statistical/Geometric Filtering)  | 부분적 완화 (Outlier 제거) 29          | 처리 지연, 실시간성 저하          | H/W: 저, S/W: 중                       | 저                      |
| **AI/ML 기반 (D2SR 등)**            | 학습 기반 탐지/완화/복구 (Learning-based Detect/Mitigate/Recover) | 높은 완화 및 데이터 복구 6             | 높은 컴퓨팅 부하 (Edge AI)        | H/W: 고(GPU/NPU), S/W: 고              | 고 (AI 가속기)          |



### 5.2  변조 기반 방식과의 비교: CDMA & RMCW



PTPv2-TDMA가 '시간'을 분할하여 간섭을 회피하는 반면, 변조 기반 방식들은 신호 자체의 '형태'를 다르게 만들어 간섭을 억제한다.

- **CDMA (Code Division Multiple Access):** 각 LiDAR에 고유한 디지털 코드(예: Optical Orthogonal Codes)를 할당하여 레이저 펄스 자체를 변조하는 방식이다.3 수신기는 들어온 모든 신호 중에서 자신의 고유 코드와 일치하는 패턴을 가진 신호만을 상관(correlation) 연산을 통해 걸러낸다. 이는 마치 여러 사람이 각자 다른 언어로 동시에 말할 때, 자신이 아는 언어만 알아듣는 것과 유사하다. CDMA는 이론적으로 높은 간섭 억제 성능을 제공하지만, 나노초 단위의 매우 빠른 디지털 신호를 생성하고 처리해야 하므로 하드웨어와 소프트웨어의 복잡도가 매우 높고, 이는 비용 상승으로 이어진다.1
- **RMCW (Random-Modulated Continuous-Wave):** 의사 난수 이진 시퀀스(Pseudo-Random Binary Sequence, PRBS)와 같이 예측 불가능한 무작위 신호로 레이저의 강도나 위상을 변조하는 방식이다.28 PRBS는 자기 자신과 비교할 때만 매우 뾰족한 상관관계 피크(peak)를 보이고, 다른 PRBS나 다른 형태의 신호와는 거의 상관관계가 나타나지 않는 특성이 있다. 이 덕분에 RMCW LiDAR는 다른 LiDAR의 신호는 물론, 햇빛이나 가로등과 같은 주변 광원에 대해서도 거의 면역에 가까운 강력한 간섭 저항성을 보인다.28 TDMA와 달리 데이터 처리량을 희생하지 않으면서도 높은 간섭 억제력을 달성할 수 있어 매우 유망한 기술로 평가받는다.

TDMA와 비교했을 때, 이들 변조 기반 방식들은 데이터 처리량 손실이 없다는 결정적인 장점을 가진다. 하지만 신호의 생성과 처리가 훨씬 복잡하여 센서 자체의 가격과 시스템 복잡도를 높이는 단점이 있다.



### 5.3  후처리 및 AI 기반 방식과의 비교



이 접근법들은 간섭을 사전에 '회피'하는 것이 아니라, 간섭이 이미 발생한 데이터를 사후에 '보정'하는 방식이다.

- **후처리 필터링 (Post-processing):** 이미 수집된 포인트 클라우드 데이터에서 간섭으로 인해 발생한 것으로 의심되는 노이즈 포인트를 제거하는 알고리즘이다. 주변 포인트들과의 거리 분포를 분석하는 통계적 이상치 제거(Statistical Outlier Removal), 점들의 군집을 분석하는 유클리드 클러스터링(Euclidean Clustering), 또는 레이저 반사율(reflectivity) 정보를 활용하여 비정상적인 반사율을 가진 포인트를 제거하는 방법 등이 있다.29 이 방식들은 구현이 비교적 간단하고 추가적인 하드웨어 비용이 들지 않는다는 장점이 있지만, 간섭을 원천적으로 막는 것이 아니므로 한계가 명확하다. 특히, 실제 객체와 유사한 형태로 나타나는 군집성 간섭이나, SNR 저하로 인해 아예 측정되지 않은 '누락된 데이터'는 복구할 수 없다. 또한, 처리 과정에 시간이 소요되어 실시간성에 영향을 줄 수 있다.
- **AI/ML 기반 접근법:** 최근에는 딥러닝 기술을 활용하여 간섭 문제를 해결하려는 시도가 활발하다. 예를 들어, D2SR이라는 연구는 3단계 접근법을 제안한다: (a) 컨볼루션 신경망(CNN)을 이용해 간섭의 영향을 받은 데이터 프레임을 정확하게 '탐지'하고, (b) 탐지된 경우 LiDAR의 작동 주기를 동적으로 시간 이동시켜 간섭을 '완화'하며, (c) 그럼에도 불구하고 손상된 데이터 영역을 딥러닝 모델로 '복구'한다.6 이 방식은 간섭을 회피하고, 피해를 최소화하며, 손상된 데이터까지 복원하는 가장 진보된 형태의 해결책이다. 하지만 이러한 복잡한 AI 모델을 실시간으로 구동하기 위해서는 엔비디아 Jetson과 같은 고성능 엣지 AI 가속기가 필수적이며, 이는 시스템의 비용과 전력 소모를 크게 증가시킨다.

결론적으로, 간섭 완화 기술은 '사전 회피(Proactive Avoidance)'와 '사후 복구(Reactive Recovery)'라는 두 가지 큰 축을 중심으로 발전하고 있다. TDMA, CDMA, RMCW와 같은 기술은 전자에 속하며, 후처리 필터링과 AI 기반 기술은 후자에 속한다. PTPv2-TDMA는 '사전 회피' 전략 중 가장 확실하지만 데이터 손실이라는 대가가 따르는 옵션인 반면, RMCW는 데이터 손실 없이 높은 회피 성능을 제공하는 균형 잡힌 옵션이다. AI 기반 기술은 가장 강력한 '사후 복구' 능력을 제공하지만 가장 높은 비용을 요구한다. 미래의 가장 강건한 자율주행 시스템은 이들 중 하나의 기술에만 의존하는 것이 아니라, 여러 기술을 결합한 다층적(multi-layered) 방어 전략을 채택할 가능성이 높다.



## 6.  결론 및 미래 전망: 지능형 동기화와 하이브리드 접근법





### 6.1. PTPv2-TDMA의 최적 적용 시나리오 요약



PTPv2 동기화를 통한 TDMA 방식은 다중 LiDAR 환경에서의 상호 간섭을 해결하기 위한 매우 직관적이고 강력한 방법론이다. 이론적으로 완벽한 간섭 회피를 보장한다는 점에서, 시스템의 안전성과 신뢰성이 다른 어떤 성능 지표보다 우선시되는 특정 응용 분야에서 그 가치를 발휘한다.

이 기술의 최적 적용 시나리오는 다음과 같이 요약할 수 있다.

- **소수의 센서가 고정된 환경을 감시하는 경우:** 스마트 팩토리의 자동화 로봇, 특정 구역을 감시하는 보안 시스템, 또는 항만의 컨테이너 관리 시스템과 같이, 제한된 수의 LiDAR가 예측 가능한 환경에서 작동하는 경우가 해당된다. 이러한 환경에서는 데이터 처리량 감소가 큰 문제가 되지 않으며, 간섭으로 인한 오작동을 원천적으로 차단하는 것이 더 중요하다.

반면, 수십 개의 센서가 예측 불가능한 동적 환경에서 상호작용하는 자율주행차의 경우, 순수한 형태의 TDMA는 데이터 처리량과 프레임 속도를 심각하게 저하시키므로 주력 기술로 채택되기에는 한계가 명확하다. 하지만 Ouster의 Phase Lock 기능처럼, PTPv2 동기화를 기반으로 한 변형된 시간 분할 방식이 특정 시나리오(예: 복잡한 교차로 진입 시 전방 LiDAR와 측면 LiDAR 간의 간섭을 일시적으로 최소화하는 모드)에서 보조적인 수단으로 활용될 가능성은 충분하다.



### 6.1  미래 기술 전망: AI 기반 동적 TDMA



PTPv2-TDMA 방식의 미래는 정적인 시간 분할을 넘어, 지능적이고 동적인 자원 할당으로 나아갈 것이다. PTPv2가 제공하는 정밀한 공통 시간 축 위에서, 인공지능(AI)이 시스템의 '지휘자' 역할을 수행하는 **'AI 기반 동적 TDMA'** 개념을 상상해 볼 수 있다.8

이러한 시스템에서 AI는 차량의 주행 상태(고속도로, 시내, 주차), 주변 교통 흐름, 기상 조건, 그리고 각 LiDAR 센서의 역할과 중요도를 실시간으로 종합 판단한다. 그리고 그 판단 결과에 따라 각 LiDAR에 할당되는 시간 슬롯의 길이와 빈도를 동적으로, 그리고 지능적으로 재분배한다.

예를 들어, 한적한 고속도로를 주행할 때는 전방 장거리 LiDAR에 90%의 시간 슬롯을 할당하여 원거리 위험 감지 능력을 극대화하고, 보행자와 차량이 얽힌 복잡한 시내 교차로에서는 전방, 좌측, 우측 LiDAR에 시간 슬롯을 거의 균등하게 분배하여 360도 주변 인지 능력을 강화하는 식의 상황인지형(context-aware) 자원 관리가 가능해진다. 이는 TDMA의 단점인 경직성과 비효율성을 극복하고, 시스템의 성능을 주어진 상황에 맞게 최적화하는 궁극적인 해결책이 될 수 있다.



### 6.2  하이브리드 간섭 완화 전략의 부상



궁극적으로, 다중 LiDAR 환경의 상호 간섭 문제는 어떤 단일 기술만으로 모든 시나리오를 완벽하게 해결할 수 없다. 미래의 가장 강건하고 신뢰도 높은 시스템은 여러 기술의 장점을 결합한 **하이브리드 간섭 완화 전략(Hybrid Interference Mitigation Strategy)**을 채택하게 될 것이다.31

이는 마치 성을 방어하기 위해 성벽, 해자, 망루 등 다층적인 방어 시스템을 구축하는 것과 같다. LiDAR 시스템 역시 다음과 같은 다층적 구조를 가질 수 있다.

1. **1차 방어선 (물리 계층):** RMCW나 CDMA와 같은 변조 기술을 사용하여 센서 자체가 본질적으로 간섭에 강한 신호를 생성한다. 이를 통해 대부분의 간섭을 1차적으로 걸러낸다.
2. **2차 방어선 (MAC 계층):** PTPv2-TDMA(특히 AI 기반 동적 TDMA)를 통해 최악의 직접적인 충돌 상황을 회피하고, 시스템 자원을 효율적으로 관리한다.
3. **3차 방어선 (소프트웨어 계층):** 그럼에도 불구하고 발생하는 미세한 잔여 간섭이나 데이터 손상은 AI 기반 후처리 알고리즘이 실시간으로 탐지하고 복구하여 최종적인 데이터의 무결성을 보장한다.

이러한 하이브리드 아키텍처에서 PTPv2는 각기 다른 원리로 작동하는 이 모든 기술 구성 요소들을 하나의 공통된 시간 축 위에 정렬시키고, 이들의 협력적 동작을 가능하게 하는 필수적인 **'접착제(glue)'** 또는 **'기반 인프라(foundational infrastructure)'**로서의 역할을 수행할 것이다. 따라서 PTPv2 동기화 기술에 대한 깊은 이해와 활용은 미래 자율주행 시스템의 안전성과 신뢰성을 한 단계 끌어올리는 핵심 열쇠가 될 것이다.



#### **참고 자료**

1. True-random LiDAR에 대한 상호간섭 영향 분석 - DSpace at KOASAS, accessed July 22, 2025, https://koasas.kaist.ac.kr/handle/10203/284448
2. Effects and conditions of mutual interference in FMCW lidars, accessed July 22, 2025, https://www.spiedigitallibrary.org/conference-proceedings-of-spie/13472/134720I/Effects-and-conditions-of-mutual-interference-in-FMCW-lidars/10.1117/12.3053799.full?webSyncID=9d902bc9-993c-ed26-8bea-d49a28531ab0&sessionGUID=d3a6b03c-97c0-09b6-bcd7-46b80d8e8abe
3. True-random LiDAR에 대한 상호간섭 영향 분석 = Analysis of the impacts of mutual interference on the true-random LiDAR - 카이스트 도서관, accessed July 22, 2025, http://library.kaist.ac.kr/search/detail/view.do?bibCtrlNo=924536&flag=dissertation
4. CFAR-Based Interference Mitigation for FMCW Automotive Radar Systems - TU Delft Research Portal, accessed July 22, 2025, https://research.tudelft.nl/files/154907760/CFAR_Based_Interference_Mitigation_for_FMCW_Automotive_Radar_Systems.pdf
5. LiDAR Sensor Interference Prevention - Xray - GreyB, accessed July 22, 2025, https://xray.greyb.com/lidar/interference-avoidance
6. D2SR: Decentralized detection, de-synchronization, and recovery of LiDAR interference - InK@SMU.edu.sg, accessed July 22, 2025, https://ink.library.smu.edu.sg/cgi/viewcontent.cgi?article=10227&context=sis_research
7. Ambient light immunity of a frequency-modulated continuous-wave (FMCW) LiDAR chip, accessed July 22, 2025, https://opg.optica.org/oe/abstract.cfm?URI=oe-32-3-3997
8. Time-division multiple access - Wikipedia, accessed July 22, 2025, https://en.wikipedia.org/wiki/Time-division_multiple_access
9. What is Time Division Multiple Access (TDMA) in the World of GNSS/GPS - Novotech, accessed July 22, 2025, https://novotech.com/pages/time-division-multiple-access-tdma
10. Interference Avoidance for Two Time-of-Flight Cameras Using ..., accessed July 22, 2025, https://www.researchgate.net/publication/341923604_Interference_Avoidance_for_Two_Time-of-Flight_Cameras_Using_Autonomous_Optical_Synchronization
11. Time Division Multiple Access (TDMA) time division scheme. - ResearchGate, accessed July 22, 2025, https://www.researchgate.net/figure/Time-Division-Multiple-Access-TDMA-time-division-scheme_fig3_325626565
12. 정밀 시각 프로토콜 - 위키백과, 우리 모두의 백과사전, accessed July 22, 2025, [https://ko.wikipedia.org/wiki/%EC%A0%95%EB%B0%80_%EC%8B%9C%EA%B0%81_%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C](https://ko.wikipedia.org/wiki/정밀_시각_프로토콜)
13. IEEE 1588 Version 2, accessed July 22, 2025, https://www.ieee802.org/1/files/public/docs2008/as-garner-1588v2-summary-0908.pdf
14. Precision Time Protocol - Wikipedia, accessed July 22, 2025, https://en.wikipedia.org/wiki/Precision_Time_Protocol
15. 데이터 수집 및 테스트를 위한 PTP(Precision Time Protocol) - HBM, accessed July 22, 2025, https://www.hbm.com/kr/5143/precision-time-protocol/
16. A Survey on IEEE 1588 Implementation for RISC-V Low-Power Embedded Devices - MDPI, accessed July 22, 2025, https://www.mdpi.com/2079-9292/13/2/458
17. Implementation and Analysis of IEEE 1588 PTP Daemon Based on Embedded System, accessed July 22, 2025, https://www.researchgate.net/publication/344766102_Implementation_and_Analysis_of_IEEE_1588_PTP_Daemon_Based_on_Embedded_System
18. PTP Stuck in UNCALIBRATED - Hardware - Ouster | Community ..., accessed July 22, 2025, https://community.ouster.com/t/ptp-stuck-in-uncalibrated/99
19. 1.3.3. Time Synchronization Instructions - Livox wiki - Read the Docs, accessed July 22, 2025, https://livox-wiki-en.readthedocs.io/en/latest/tutorials/new_product/common/time_sync.html
20. IEEE 1588v2/PTP Testing - Quick Reference Guide - Knowledge Base - VeEX Inc., accessed July 22, 2025, https://kb.veexinc.com/en/knowledge/ieee-1588v2/ptp-quick-reference-guide
21. PTP Profiles Guide - Ouster Sensor Docs documentation, accessed July 22, 2025, https://static.ouster.dev/sensor-docs/image_route1/image_route2/appendix/ptp-profile-guide.html
22. LiDAR integration | SBG Systems, accessed July 22, 2025, https://support.sbg-systems.com/sc/hp/latest/how-to-articles/lidar-integration
23. Time Synchronization using PTP/IEEE1588 - Ruixiang's Notes, accessed July 22, 2025, https://notes.rdu.im/system/network/time-sync-using-ptp/
24. Multi-LiDAR time synchronization with phase_lock - Software ..., accessed July 22, 2025, https://community.ouster.com/t/multi-lidar-time-synchronization-with-phase-lock/323
25. The Future of AI Using LiDAR | Dell Technologies Info Hub, accessed July 22, 2025, https://infohub.delltechnologies.com/p/the-future-of-ai-using-lidar/
26. Driveworks, Orin : Camera Lidar Time Alignment via PTP - NVIDIA Developer Forums, accessed July 22, 2025, https://forums.developer.nvidia.com/t/driveworks-orin-camera-lidar-time-alignment-via-ptp/264656
27. 2-D Optical-CDMA Modulation in Automotive Time-of-Flight LIDAR Systems - University of Strathclyde, accessed July 22, 2025, https://pureportal.strath.ac.uk/files/105854771/Kwong_etal_ICTON_2020_2_D_optical_CDMA_modulation_in_automotive_time_of_flight.pdf
28. Coherent Random-Modulated Continuous-Wave LiDAR Based on ..., accessed July 22, 2025, https://www.mdpi.com/2304-6732/8/11/475
29. Algorithm for Point Cloud Dust Filtering of LiDAR for Autonomous Vehicles in Mining Area, accessed July 22, 2025, https://www.mdpi.com/2071-1050/16/7/2827
30. Behind the Scenes of LiDAR Data Processing: A Mission Coordinator's Perspective, accessed July 22, 2025, https://flyguys.com/behind-the-scenes-of-lidar-data-processing/
31. CPC COOPERATIVE PATENT CLASSIFICATION, accessed July 22, 2025, https://www.cooperativepatentclassification.org/sites/default/files/cpc/scheme/H/scheme-H04B.pdf
32. 分類対照ツール, accessed July 22, 2025, [https://www.jpo.go.jp/cgi/cgi-bin/search-portal/narabe_tool/table.cgi?keyword=H04B&type=](https://www.jpo.go.jp/cgi/cgi-bin/search-portal/narabe_tool/table.cgi?keyword=H04B&type)

