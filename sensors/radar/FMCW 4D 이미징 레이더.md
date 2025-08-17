# FMCW 4D 이미징 레이더

## 1.  기초 원리 및 핵심 기술

이 보고서의 첫 번째 부분에서는 이후의 모든 논의의 기초가 되는 기본적인 물리 및 신호 처리 원리를 확립한다. 연속파의 기본 개념에서부터 시작하여, 인식의 처음 세 가지 차원을 추출하는 정교한 신호 처리 파이프라인까지 다룬다.

### 1.1  FMCW 레이더의 물리학

본 장에서는 주파수 변조 연속파(Frequency-Modulated Continuous-Wave, FMCW) 방법론을 심층적으로 분석하고, 이를 기존의 펄스 레이더와 대조하여 왜 이 기술이 현대 자동차 및 산업용 애플리케이션에서 지배적인 선택이 되었는지 설명한다.

#### 1.1.1 제 1.1절: 연속파 철학: 펄스 레이더와의 대비

레이더 기술은 크게 펄스 방식과 연속파(Continuous Wave, CW) 방식으로 나뉜다. 펄스 레이더는 짧은 시간 동안 높은 출력의 전자기파 펄스를 방사하고, 이 펄스가 목표물에 반사되어 돌아오는 비행 시간(Time-of-Flight)을 직접 측정하여 거리를 계산한다.1 반면, FMCW 레이더는 이름에서 알 수 있듯이 연속적인 파형을 송신한다.3 FMCW 레이더의 핵심 측정 방식은 송신 신호와 수신 신호 간의 주파수 또는 위상 차이를 분석하는 것이다.1

이러한 근본적인 차이는 FMCW 방식에 상당한 아키텍처적 이점을 제공한다. 펄스 레이더는 높은 첨두 전력(peak power)을 요구하지만, FMCW는 낮은 전력으로 연속적인 신호를 방사하므로 방사선 안전성 측면에서 유리하며, 하드웨어 구현이 상대적으로 단순하다.1 이러한 특성 덕분에 FMCW 레이더는 경량화 및 저비용화가 가능하며, 이는 자동차 센서와 같이 대량 생산이 필수적인 애플리케이션에 매우 적합하다.5 실제로 자동차 산업에서 FMCW 방식이 널리 채택된 주된 이유는 기존 펄스 레이더에 비해 신호 처리 하드웨어의 복잡도가 낮고 비용 효율적이기 때문이다.4

이러한 경제적, 기술적 우위는 단순한 공학적 선호를 넘어, 4D 이미징 레이더라는 새로운 시장의 탄생을 가능하게 한 근본적인 동력이 되었다. FMCW의 내재적 특성인 저비용, 저전력, 그리고 비교적 간단한 하드웨어로 거리와 속도를 동시에 측정할 수 있는 능력은 레이더가 대중적인 자동차 센서로 자리매김하는 데 필수적인 조건이었다. 만약 개별 레이더 유닛이 펄스 시스템처럼 고가였다면, 여러 개의 칩을 계단식으로 연결하여 고해상도 이미징 레이더를 구현하는 현재의 방식은 경제적으로 불가능했을 것이다. 따라서 저비용으로 확장이 가능한 FMCW 아키텍처는 4D 이미징 레이더 산업 전체를 지탱하는 경제적 기반이라고 할 수 있다.

#### 1.1.2 제 1.2절: 처프 신호: 시간과 주파수의 인코딩

FMCW 레이더는 송신하는 연속파의 주파수를 시간에 따라 변조한다. 가장 일반적인 방식은 주파수를 선형적으로 증가 또는 감소시키는 것으로, 이렇게 변조된 신호를 "처프(Chirp)"라고 부른다.3 처프 신호는 주로 선형 주파수 변조(Linear Frequency Modulation, LFM) 파형을 사용하며, 톱니파(sawtooth wave)나 삼각파(triangular wave) 형태가 대표적이다.9 이 신호는 정해진 시간(T) 동안 특정 대역폭(BW)에 걸쳐 주파수를 스윕(sweep)하는데, 예를 들어 자동차 레이더에서는 76 GHz에서 81 GHz 사이의 주파수 대역을 사용한다.8

이러한 주파수 변조는 송신 신호에 일종의 "타임 스탬프"를 부여하는 효과를 가진다.3 즉, 신호가 반사되어 돌아오는 데 걸리는 시간 지연(Δt)이 송신 신호와 수신 신호 간의 주파수 차이(Δf)로 나타나게 된다. 따라서 처프 신호는 시간 정보를 주파수 정보로 인코딩하는 핵심적인 역할을 수행한다. 처프의 주요 파라미터인 대역폭(BW)은 레이더의 거리 해상도(range resolution)를 결정하는 가장 중요한 요소이다.1 더 넓은 대역폭을 사용할수록 더 정밀하게 가까운 두 물체를 분리하여 인식할 수 있다.

#### 1.1.3 제 1.3절: 신호 경로: 신호 발생기에서 믹서까지

FMCW 레이더 시스템의 신호 경로는 신호 발생기(Synthesizer)에서 시작된다. 신호 발생기는 정밀하게 설계된 처프 신호를 생성하고, 이 신호는 증폭기(Amplifier)를 거쳐 송신 안테나(Tx antenna)를 통해 공간으로 방사된다.8 방사된 전자기파는 목표물에 부딪혀 반사되고, 이 반사파는 수신 안테나(Rx antenna)에 의해 수신된다.

가장 결정적인 단계는 믹서(Mixer)에서 일어난다. 믹서는 수신된 반사 신호와 송신단에서 분기된 원본 처프 신호, 이 두 신호를 입력으로 받는다.8 믹서는 시간 영역에서 두 신호를 곱하는 연산을 수행하는데, 이는 주파수 영역에서 두 신호의 주파수 차이에 해당하는 새로운 주파수 성분을 생성하는 과정과 같다. 이 과정을 통해 시간 지연 정보가 주파수 차이 정보로 변환되며, 이는 후속 신호 처리 과정을 매우 단순화시키는 핵심적인 역할을 한다.

#### 1.1.4 제 1.4절: 비트 주파수: 측정의 기초

믹서의 출력 신호는 "비트 주파수(beat frequency)" 또는 "중간 주파수(Intermediate Frequency, IF)"라고 불리는 저주파 신호이다.1 이 비트 신호의 주파수(fbeat)는 송신 신호와 수신 신호 사이의 시간 지연(\tau)에 정비례하며, 따라서 목표물까지의 거리(R)에 비례한다.1 이 관계는 다음 식으로 표현될 수 있다:

fbeat=K\tau=Kc2R

여기서 K는 처프의 주파수 변화율(slope), c는 빛의 속도이다. 만약 목표물이 레이더에 대해 상대 속도를 가지고 움직인다면, 도플러 효과(Doppler effect)에 의해 수신 신호의 주파수가 변하고, 이 도플러 주파수 편이(fD)가 비트 신호에 더해지거나 빼져 속도 정보를 담게 된다.1

이렇게 생성된 아날로그 IF 신호는 아날로그-디지털 변환기(Analog-to-Digital Converter, ADC)를 통해 디지털 신호로 샘플링된 후, 디지털 신호 처리 장치(Digital Signal Processor, DSP)나 마이크로컨트롤러(MCU)에서 분석되어 거리, 속도, 각도 등의 정보를 추출하게 된다.3 나노초 단위의 짧은 비행 시간을 직접 측정해야 하는 고속, 고가의 회로 대신, 상대적으로 낮은 주파수의 비트 신호를 분석하는 방식을 통해 FMCW 레이더는 시스템의 복잡성과 비용을 획기적으로 낮출 수 있었다.1

### 1.2  신호 처리 파이프라인: 레이더 큐브의 구성

본 장에서는 원시 ADC 데이터로부터 인식의 세 가지 기본 차원을 추출하여 "레이더 데이터 큐브(Radar Data Cube)"를 구성하는 일련의 신호 처리 단계를 상세히 설명한다. 이 과정은 주로 고속 푸리에 변환(Fast Fourier Transform, FFT)을 통해 이루어진다.

#### 1.2.1 제 2.1절: 1차원: 고속 푸리에 변환을 통한 거리 추정 (Range-FFT)

신호 처리 파이프라인의 첫 번째 단계는 단일 처프 동안 수집된 디지털 IF 신호에 대해 FFT를 수행하는 것이다. 이 과정은 매우 짧은 시간(처프 지속 시간) 동안 발생하기 때문에 "고속 시간(fast time)" FFT라고도 불린다.3 FFT의 결과로 얻어지는 주파수 스펙트럼에서는 뚜렷한 피크(peak)들이 나타나는데, 각 피크의 주파수는 특정 목표물로부터 반사된 신호의 비트 주파수에 해당한다.11

앞서 설명했듯이 비트 주파수는 목표물까지의 거리에 정비례하므로, 이 FFT 결과는 사실상 해당 장면의 거리 프로파일(range profile)을 나타낸다. 즉, FFT 출력의 각 "빈(bin)"은 특정 거리 값에 대응된다.3 따라서 스펙트럼에서 피크를 찾는 것만으로 장면에 있는 모든 목표물의 거리를 동시에 측정할 수 있다. 인접한 두 목표물을 별개의 것으로 구분할 수 있는 능력, 즉 거리 해상도(ΔR)는 송신 신호의 대역폭(B)에 의해 결정되며, 그 관계는 다음과 같다:

ΔR=2Bc

이것이 레이더 큐브의 첫 번째 차원인 '거리'를 구성하는 과정이다.

#### 1.2.2 제 2.2절: 2차원: 도플러-FFT를 통한 속도 추정

속도를 측정하기 위해 레이더는 N개의 연속적인 처프를 하나의 "프레임(frame)"으로 묶어 송신한다.11 만약 목표물이 움직이고 있다면, 도플러 효과로 인해 각 처프에서 수신되는 IF 신호의 위상(phase)이 미세하게 변하게 된다.4 동일한 거리에 있지만 속도가 다른 두 목표물은 Range-FFT 결과에서는 동일한 거리 빈에 나타나 구분이 불가능하다.

이들을 구분하기 위해 두 번째 FFT가 수행된다. 먼저 프레임 내의 모든 처프에 대해 각각 Range-FFT를 수행하여 거리 정보를 추출한다. 그 다음, 각 거리 빈에 대해 N개의 처프에 걸쳐 수집된 데이터(위상 변화 정보 포함)를 대상으로 다시 한번 FFT를 수행한다. 이 두 번째 FFT는 상대적으로 긴 시간(프레임 지속 시간)에 걸쳐 수행되므로 "저속 시간(slow time)" 또는 "도플러-FFT(Doppler-FFT)"라고 불린다.3

도플러-FFT의 출력은 도플러 주파수 편이를 나타내며, 이는 목표물의 상대 속도에 정비례한다. 이 과정을 통해 거리와 속도 정보를 동시에 보여주는 2차원 '거리-도플러 맵(Range-Doppler map)'이 생성된다.4 이 맵을 통해 동일한 거리에 있는 여러 목표물을 속도에 따라 분리할 수 있다. 이것이 레이더 큐브의 두 번째 차원인 '속도'이다.

#### 1.2.3 제 2.3절: 3차원: 다중 안테나 배열을 통한 각도 추정 (Angle-FFT)

신호가 어느 방향에서 왔는지, 즉 도래각(Angle of Arrival, AoA)을 추정하기 위해서는 공간적으로 분리된 여러 개의 수신(Rx) 안테나가 필요하다.8 특정 목표물에서 반사된 전자기파는 각 Rx 안테나에 미세한 시간차를 두고 도달하며, 이는 각 안테나에서 수신된 IF 신호의 위상차로 나타난다.11

이 위상차를 측정하기 위해 세 번째 FFT가 수행된다. 거리-도플러 맵의 각 셀(특정 거리와 속도를 갖는 지점)에 대해, 여러 Rx 안테나 채널에서 수집된 데이터에 걸쳐 FFT를 수행한다. 이를 "앵글-FFT(Angle-FFT)"라고 한다.8 이 FFT의 결과로 나타나는 위상차(Δϕ)는 안테나 간의 거리(d)와 신호의 입사각(θ)에 의해 결정된다. 이 관계를 이용하여 목표물의 방위각(azimuth, 수평각)을 계산할 수 있다.8

이로써 거리, 속도, 방위각이라는 세 가지 차원으로 구성된 전통적인 3D '레이더 데이터 큐브'가 완성된다. 이 데이터 큐브는 레이더가 주변 환경에 대한 2차원 공간 지도를 생성하고, 탐지된 객체들을 정확한 위치에 배치할 수 있게 해준다.

이처럼 순차적인 FFT 프로세스를 통해 구성되는 레이더 큐브는 계산적으로는 우아하지만 물리적으로는 여러 제약 조건에 묶여 있는 데이터 구조이다. 각 차원의 성능 지표들은 서로 긴밀하게 연결되어 있어, 한 차원의 해상도를 향상시키려는 시도는 종종 다른 차원에서의 성능 저하 또는 계산 부하의 급증을 초래한다. 예를 들어, 더 높은 속도 해상도를 얻기 위해서는 프레임 시간(더 많은 처프)을 늘려야 하는데, 이는 빠르게 움직이는 목표물에 대해 모션 블러(motion blur) 또는 거리 이동(range migration) 현상을 유발할 수 있다.5 마찬가지로, 더 높은 각도 해상도를 위해 안테나 채널 수를 늘리면 앵글-FFT에 필요한 데이터 양이 기하급수적으로 증가하여 더 강력하고 전력 소모가 많은 DSP가 요구된다.3 또한, 최대 탐지 가능 거리와 최대 측정 가능 속도는 서로 반비례 관계에 있는 등, 레이더 시스템 설계는 각 차원의 성능을 독립적으로 최적화하는 것이 아니라, FMCW 신호의 물리적 특성과 실시간 처리라는 제약 조건 하에서 복잡한 다차원적 절충 관계를 탐색하는 과정이다. 이러한 이유로 단거리, 중장거리 등 특정 애플리케이션에 최적화된 다양한 종류의 레이더가 존재하는 것이다.13

## 2.  4D 이미징으로의 진화

이 부분에서는 기존의 3D 레이더에서 4D 이미징 레이더로 도약하는 데 결정적인 역할을 한 기술적 발전을 설명한다. 네 번째 차원을 정의하고, '이미징'을 가능하게 하는 하드웨어 및 아키텍처의 변화를 상세히 다룬다.

### 2.1  네 번째 차원의 정의와 '이미징' 패러다임

본 장에서는 '4D'와 '이미징'이 이 기술의 맥락에서 무엇을 의미하는지 명확히 하고, 단순한 점 기반 탐지 시스템에서 고해상도 환경 매핑 도구로 어떻게 발전했는지 설명한다.

#### 2.1.1 제 3.1절: '4D'의 차원: 거리, 속도, 방위각, 그리고 고도각

전통적인 자동차 레이더는 세 가지 차원의 정보를 제공했다: 목표물까지의 거리(range), 수평 방향(azimuth), 그리고 상대 속도(Doppler)이다.15 4D 이미징 레이더는 여기에 네 번째 핵심 차원인 '고도각(elevation)', 즉 수직 방향 정보를 추가한다.10 일부에서는 이 네 가지 차원을 X(수평), Y(수직), Z(깊이), 그리고 V(속도)로 표현하기도 한다.16

이러한 고도각 정보의 획득은 안테나 소자를 기존의 수평 배열뿐만 아니라 수직으로도 배열함으로써 가능해진다.15 3D 레이더가 수평으로만 안테나를 배열하여 방위각만 측정할 수 있었던 반면, 4D 레이더는 2차원 평면에 안테나를 배열하여 수평각과 수직각을 모두 측정할 수 있다.

고도각 정보의 추가는 그야말로 '게임 체인저'이다. 이를 통해 레이더는 도로 위에 멈춰 있는 차량과 그 위를 지나는 고가도로 표지판이나 다리 구조물을 명확히 구분할 수 있게 된다. 이는 3D 레이더가 종종 혼동하던 시나리오이다.15 또한 도로의 윤곽, 연석, 포장도로의 이음새와 같은 낮은 구조물도 식별할 수 있게 되어, 차량이 주행 가능한 공간을 더욱 정확하게 판단할 수 있다.15 이러한 능력은 운전자의 개입 없이 차량 스스로 복잡한 판단을 내려야 하는 레벨 3 이상의 고도화된 자율주행 기능을 구현하는 데 필수적이다.15

#### 2.1.2 제 3.2절: 포인트 클라우드에서 '이미지'로: 고해상도 데이터의 중요성

'이미징' 레이더라는 용어는 이 센서가 반환하는 데이터의 풍부함과 밀도에서 유래한다.15 방위각과 고도각 모두에서 높은 해상도를 갖춤으로써, 레이더는 단일 물체로부터 다수의 반사점을 탐지할 수 있다. 이 점들을 3차원 공간에 매핑하면, 마치 저해상도 이미지처럼 물체의 형태나 윤곽을 나타내는 밀도 높은 '포인트 클라우드(point cloud)'가 형성된다.15

이것이 바로 '4D 레이더'와 '4D *이미징* 레이더'를 구분하는 핵심적인 차이점이다. 기본적인 4D 레이더도 고도각 측정 기능은 가질 수 있지만, 그 해상도가 낮아 물체의 형태를 제대로 파악하기 어렵다. 반면, '이미징' 레이더는 원거리에서도 물체의 형상을 분해할 수 있을 만큼 충분한 각도 해상도(일반적으로 1도 이하)를 가져야 한다.19 이러한 고밀도 포인트 클라우드를 통해 시스템은 단순히 물체의 존재를 탐지하는 것을 넘어, 보행자, 자전거, 자동차 등 도로 사용자의 유형을 구분할 수 있게 된다.17 이처럼 풍부한 데이터는 카메라 데이터와의 융합을 더욱 의미 있게 만들고, 더 발전된 인식 알고리즘의 기반이 된다.21

이러한 맥락에서 '4D 이미징 레이더'라는 용어는 단순한 기술 사양을 넘어, 전략적인 마케팅 및 기술적 개념으로 자리 잡았다. 이 용어는 레이더가 '이미지와 유사한' 포인트 클라우드를 제공할 수 있음을 강조함으로써, 라이다(LiDAR)의 직접적인 경쟁 기술로 재포지셔닝하기 위해 만들어졌다. 이러한 프레임 전환은 레이더의 고유한 장점(비용, 전천후 성능)을 유지하면서도, 자율주행차 센서 경쟁 구도를 근본적으로 바꾸어 놓았다. 기존에 레이더는 보조 센서로 인식되었으나, '이미징'이라는 개념을 통해 라이다의 역할을 위협하는 핵심 인식 센서로 격상되었고, 이는 업계의 투자, 연구개발 방향, 그리고 완성차 업체의 센서 구성 결정에 지대한 영향을 미치고 있다.

#### 2.1.3 제 3.3절: 4D 이미징 레이더의 아키텍처 기반

높은 각도 해상도를 달성하기 위해서는 넓은 안테나 개구(aperture)가 필요하다. 이를 물리적으로 작은 폼팩터 내에서 구현하기 위해 다중 입출력(Multiple-Input Multiple-Output, MIMO) 기술이 사용된다.22 MIMO 레이더는 P개의 송신(Tx) 안테나와 Q개의 수신(Rx) 안테나에서 나오는 신호를 조합하여, 마치 P×Q개의 소자를 가진 하나의 큰 '가상 배열(virtual array)'처럼 작동하게 만든다.24

이미징에 필요한 수백 개 이상의 가상 채널을 확보하기 위해, 기업들은 여러 개의 레이더 MMIC(Monolithic Microwave Integrated Circuit) 칩을 계단식으로 연결(cascading)하는 방식을 사용한다.19 예를 들어, 각각 3개의 Tx와 4개의 Rx 채널을 가진 칩 4개를 배열하면, 총 12개의 Tx와 16개의 Rx 채널을 갖는 시스템이 되어 12×16=192개의 가상 채널을 구현할 수 있다.16 이스라엘의 스타트업인 Arbe Robotics는 한 걸음 더 나아가, 48개의 Tx와 48개의 Rx 채널을 확장하여 2,304개의 가상 채널을 생성하는 칩셋을 개발했다.19

MIMO 기술과 칩 캐스케이딩을 통해 거대한 가상 배열을 생성하는 능력은 4D 이미징 레이더의 핵심적인 하드웨어 구현 기술이다. 이는 물리적으로 비현실적인 크기의 안테나를 만들지 않고도, 해상도의 물리적 한계인 레일리 기준(Rayleigh Criterion)을 극복하는 실질적인 메커니즘이다.19 그러나 이러한 기술적 경로는 필연적으로 데이터 양의 폭증과 계산 복잡성이라는 새로운 과제로 이어진다.

## 3.  성능, 응용 분야 및 시장 환경

이 부분에서는 4D 이미징 레이더를 실제 세계에 적용하여 다른 주요 센서들과의 성능을 비교하고, 자동차 산업을 필두로 다양한 산업 분야에서의 응용 사례를 상세히 설명하며, 기술 개발과 채택을 주도하는 기업 생태계를 분석한다.

### 3.1  인식 센서 비교 분석

본 장에서는 4D 이미징 레이더, 라이다, 카메라에 대한 비판적이고 다각적인 비교를 통해 각 센서의 상호 보완적인 특성과 센서 융합의 당위성을 설명한다.

#### 3.1.1 제 4.1절: 4D 이미징 레이더 대 라이다

4D 이미징 레이더와 라이다는 첨단 자율주행 인식 시스템의 핵심 자리를 놓고 경쟁하는 가장 중요한 두 센서 기술이다.

- **작동 원리:** 레이더는 밀리미터파(mmWave) 대역(76-81 GHz)의 전파를 사용하고, 라이다는 근적외선 파장의 레이저 빛을 사용한다.26 이 근본적인 차이가 두 센서의 모든 성능 특성을 결정한다.
- **전천후 성능:** 레이더의 가장 큰 장점은 악천후 조건에서의 강인함이다. 전파는 비, 안개, 눈, 먼지 등을 쉽게 투과할 수 있어 날씨에 거의 영향을 받지 않는다.6 반면, 라이다는 빛을 사용하기 때문에 이러한 대기 중 입자에 의해 신호가 산란되거나 흡수되어 성능이 크게 저하된다.31
- **해상도 및 포인트 클라우드:** 일반적으로 라이다가 월등히 높은 각도 해상도와 포인트 클라우드 밀도를 제공한다.26 이를 통해 매우 상세하고 정밀한 3차원 지도를 생성할 수 있다. 4D 이미징 레이더의 해상도가 극적으로 향상되었지만, 그 포인트 클라우드는 여전히 라이다에 비해 희소하고 노이즈가 많은 경향이 있다.20 두 기술의 정확도는 동일한 수준으로 보기 어렵다.20
- **비용:** 4D 이미징 레이더는 라이다에 비해 훨씬 저렴하다. 일부 분석에 따르면 라이다 비용의 10-20% 수준에 불과하다.18 이는 대량 생산되는 차량에 탑재하는 데 있어 결정적인 장점이다. 다만, 최고 성능의 이미징 레이더는 일부 저가형 라이다보다 비쌀 수도 있다.20
- **속도 측정:** 레이더는 도플러 효과를 통해 목표물의 상대 속도를 직접적이고 즉각적으로 측정할 수 있다. 라이다는 일반적으로 연속된 프레임에서 감지된 포인트의 위치 변화를 계산하여 속도를 추정하므로, 간접적이고 순간적인 속도 측정에는 레이더보다 불리할 수 있다.

#### 3.1.2 제 4.2절: 4D 이미징 레이더 대 카메라

카메라는 가장 보편적인 차량용 센서이며, 레이더와는 매우 상호 보완적인 특성을 가진다.

- **작동 방식:** 레이더는 자체적으로 신호를 방사하고 수신하는 능동형 센서(active sensor)인 반면, 카메라는 주변의 빛에 의존하는 수동형 센서(passive sensor)이다.32
- **성능:** 카메라는 고해상도의 컬러 및 텍스처 정보를 제공하여, 교통 표지판 인식, 신호등 색상 구분 등 객체의 의미론적 분류(semantic classification)에 매우 뛰어나다.32 하지만 성능이 조명 조건(어둠, 역광)과 날씨에 크게 좌우된다는 치명적인 단점이 있다.17 레이더는 조명과 날씨에 관계없이 일관된 성능을 보인다.
- **거리 및 속도 측정:** 단안 카메라는 물체까지의 거리를 정확하게 측정할 수 없다.33 반면, 레이더는 거리와 속도 측정에 탁월한 성능을 보인다.
- **사생활 보호:** 레이더가 생성하는 포인트 클라우드에는 개인을 식별할 수 있는 정보가 포함되지 않는다. 따라서 차량 실내 모니터링이나 사생활 보호가 중요한 공공 감시 분야에 이상적이다.32 카메라는 이러한 응용 분야에서 심각한 사생활 침해 문제를 야기할 수 있다.

#### 3.1.3 제 4.3절: 센서 융합: 필연적인 경로

단일 센서만으로는 안전한 자율주행에 필요한 모든 요구사항을 충족시킬 수 없다.37 각 센서는 고유한 장점과 약점을 가지고 있기 때문이다.35 따라서 업계의 지배적인 접근 방식은 센서 융합(sensor fusion)이다. 이는 카메라, 레이더, 라이다 등 여러 센서로부터 얻은 데이터를 결합하여 단일의, 강건하고 정확한 환경 모델을 생성하는 기술이다.15

센서 융합은 시스템에 중복성(redundancy)을 제공한다. 예를 들어, 안개 속에서 카메라의 기능이 마비되더라도 레이더나 라이다가 그 역할을 대신할 수 있다.17 그러나 센서 융합의 가장 큰 과제는 각 센서가 서로 상충되는 정보를 제공할 때 발생한다. "카메라는 장애물이 없다고 하고, 라이다는 있다고 할 때, 어떤 센서를 신뢰해야 하는가?"와 같은 문제를 해결하기 위한 정교한 알고리즘이 필요하다.35

4D 이미징 레이더의 역할은 독립적인 '만능 센서'가 되는 것이 아니라, 정교한 센서 융합 엔진에 더 강력하고 신뢰할 수 있는 데이터를 제공하는 것이다. 4D 이미징 레이더가 제공하는 밀도 높은 4차원 포인트 클라우드는 라이다 및 카메라 데이터와의 융합 과정을 더욱 정확하고 의미 있게 만들어, 전체 인식 시스템의 성능을 한 단계 끌어올린다.

자율주행 센서 구성을 둘러싼 논쟁, 예를 들어 테슬라의 카메라 중심 접근법과 웨이모의 라이다 중심 접근법은 근본적으로 '코너 케이스(corner case)'를 어떻게 해결하고 위험을 관리할 것인가에 대한 철학의 차이이다. 4D 이미징 레이더는 이 두 가지 접근법의 약점을 모두 보완할 수 있는 강력한 '중간 지대' 기술로 부상했다. 카메라 중심 시스템에는 악천후에서도 강건한 깊이 및 속도 데이터를 추가해주고, 라이다 중심 시스템에는 더 저렴한 비용으로 전천후 중복성과 직접적인 속도 측정 능력을 제공한다. 자율주행의 안전성은 드물지만 치명적인 코너 케이스에서의 성능에 의해 정의된다.40 카메라와 라이다는 모두 악천후라는 중대한 코너 케이스에 취약하며, 레이더는 바로 이 지점에서 강점을 보인다.17 따라서 4D 이미징 레이더는 단순히 또 하나의 센서 옵션이 아니라, 자율주행 인식 기술의 두 가지 주요 경쟁 철학의 핵심적인 약점을 해결하는 전략적 기술이며, 장기적으로 센서 구성의 수렴을 이끌 가능성이 높다.

##### 3.1.3.1 표 4.1: 인식 센서 기술 비교 매트릭스

| 특성               | 카메라             | 3D 레이더       | 4D 이미징 레이더  | 라이다                |      |
| ------------------ | ------------------ | --------------- | ----------------- | --------------------- | ---- |
| **작동 원리**      | 가시광선 (수동)    | 전파 (능동)     | 전파 (능동)       | 레이저 (능동)         |      |
| **각도 해상도**    | 매우 높음          | 낮음            | 중간 ~ 높음       | 매우 높음             |      |
| **최대 탐지 거리** | 중간               | 높음            | 매우 높음 (300m+) | 높음                  |      |
| **속도 측정**      | 간접적 (추정)      | 직접적 (도플러) | 직접적 (도플러)   | 간접적 (추정)         |      |
| **악천후 성능**    | 취약               | 매우 강함       | 매우 강함         | 취약                  |      |
| **조명 독립성**    | 낮음 (빛 필요)     | 높음            | 높음              | 높음                  |      |
| **상대 비용**      | 매우 낮음          | 낮음            | 중간              | 높음                  |      |
| **전력 소모**      | 낮음               | 낮음            | 중간              | 높음                  |      |
| **사생활 보호**    | 취약               | 우수            | 우수              | 우수                  |      |
| **객체 분류 능력** | 우수 (색상/텍스처) | 제한적          | 중간 (형태)       | 좋음 (형태)           |      |
| **형태/크기 인식** | 중간 (2D 기반)     | 거의 불가       | 좋음 (3D 포인트)  | 매우 우수 (고밀도 3D) |      |
| 데이터 출처: 6     |                    |                 |                   |                       |      |

### 3.2  주요 응용 분야 및 산업적 영향

본 장에서는 4D 이미징 레이더의 다양한 응용 분야를 탐색하며, 자동차 산업을 필두로 여러 시장 부문에서 이 기술의 고유한 역량이 어떻게 가치를 창출하고 있는지 상세히 설명한다.

#### 3.2.1 제 5.1절: 자동차: 핵심 동력 (ADAS 및 자율주행)

자동차 산업은 4D 이미징 레이더 시장의 가장 큰 수요처이자 기술 발전을 이끄는 핵심 동력이다.14 이 기술은 첨단 운전자 보조 시스템(Advanced Driver-Assistance Systems, ADAS)의 성능을 향상시키고, 레벨 2+부터 레벨 5에 이르는 자율주행 기술의 핵심 구현 요소로 간주된다.15

- **레벨 2+ 기능:** 4D 이미징 레이더는 보행자 및 자전거 탑승자 감지 성능이 향상된 자율 긴급 제동(AEB), 지능형 순항 제어(ACC), 사각지대 감지(BSD), 차선 변경 보조(LCA)와 같은 기존 ADAS 기능의 정밀도를 크게 높인다.14 특히 '레벨 2+'는 운전자의 법적 책임은 유지하면서도 레벨 3에 가까운 편의 기능을 제공할 수 있어, 향후 10년간 자동차 시장의 주된 경쟁 영역이 될 것으로 전망된다.40
- **레벨 3~5 자율주행:** 운전자가 운전에서 완전히 손을 떼는 고도 자율주행 단계에서는 차량이 주변 환경을 완벽하게 이해하고 스스로 판단해야 한다. 4D 이미징 레이더는 고속도로 주행 시 300m 이상의 원거리 탐지를 통해 위험을 조기에 예측하고 17, 고해상도 환경 매핑을 통해 주행 가능 영역을 판단하며, 3D 레이더로는 감지하기 어려운 작은 도로 위 장애물을 식별하는 데 결정적인 역할을 한다.15

#### 3.2.2 제 5.2절: 산업, 로보틱스, 드론

4D 이미징 레이더의 강인한 특성은 산업 현장에서도 그 가치를 발휘한다. 먼지나 증기가 많은 환경과 같이 시각 센서가 제 기능을 못하는 곳에서 자율 로봇의 장애물 감지, 스마트 팩토리의 작업자 안전 모니터링 등에 활용될 수 있다.14 구체적인 응용 사례로는 정유 탱크의 액체 수위 측정 6, 공항 지상 장비나 건설 중장비의 정밀 유도 29 등이 있다. 드론 분야에서는 지면 상태를 감지하여 안전한 착륙을 돕거나, 비행 중 충돌을 방지하는 시스템에 적용된다.46

#### 3.2.3 제 5.3절: 보안, 감시 및 국방

보안 및 감시 분야에서 4D 이미징 레이더는 카메라와 달리 사생활 침해 우려 없이 360도 연속 모니터링을 제공하여 침입자 경보, 국경 감시, 주요 시설 보호에 사용된다.14 특히, 작고 빠르게 움직이는 드론을 모든 기상 조건에서 탐지하고 추적하는 대(對)드론(Counter-UAS) 시스템의 핵심 기술로 부상하고 있다.36 국방 분야에서는 항공기 항법, 미사일 유도, 지형 매핑 시스템의 성능을 향상시키며, 특히 GPS가 차단된 환경에서 그 중요성이 더욱 커진다.14

#### 3.2.4 제 5.4절: 신흥 분야: 헬스케어 및 스마트 인프라

4D 이미징 레이더는 예상치 못한 분야에서도 혁신을 이끌고 있다. 헬스케어 분야에서는 비접촉식 환자 모니터링을 가능하게 한다. 웨어러블 기기나 침습적인 카메라 없이도 환자의 호흡, 심박수와 같은 미세한 생체 신호를 감지하고 11, 낙상을 예측하거나 감지하며, 보행 패턴을 분석할 수 있다.14 이는 병원, 요양원뿐만 아니라 사생활 보호가 중요한 화장실 같은 공간에서도 유용하다.36

스마트 시티 인프라에서는 지능형 교통 관리 시스템에 활용된다. 실시간으로 교통량, 속도, 도로 점유율 데이터를 수집하고, 도로 위 정차 차량이나 역주행과 같은 돌발 상황을 즉시 감지하여 교통 흐름을 최적화하고 안전을 증진시킨다.17

이러한 다양한 응용 분야의 확산은 자동차 산업에서 시작된 '선순환 구조'에 기인한다. 자동차 시장의 엄격한 안전(ASIL) 및 신뢰성 요구사항과 대량 생산에 따른 비용 절감 압력은 4D 이미징 레이더 기술(칩셋, 알고리즘, 제조 공정)의 성숙을 강제했다. 이렇게 성숙하고 대량 생산을 통해 가격이 낮아진 기술은, 독자적으로는 개발 자금을 감당하기 어려웠을 다양한 비(非)자동차 산업 분야로 확산되어 새로운 시장을 창출하고 있다. 한국의 스마트레이더시스템이나 이스라엘의 Vayyar와 같은 기업들은 자동차용으로 개발된 핵심 기술을 헬스케어, 산업 안전 등 다른 분야에 적극적으로 적용하며 이러한 경향을 주도하고 있다.51 따라서 비자동차 분야의 미래 성장은 자동차 시장의 지속적인 기술 발전과 비용 절감 노력에 크게 의존할 것이다.

### 3.3  글로벌 시장 생태계

본 장에서는 칩 설계사부터 시스템 통합업체, 혁신적인 스타트업에 이르기까지 4D 이미징 레이더 가치 사슬의 주요 기업들을 분석하고, 시장을 형성하는 지역별 역학 관계를 살펴본다.

#### 3.3.1 제 6.1절: 반도체 거대 기업과 그들의 플랫폼

4D 이미징 레이더 시장의 기반은 핵심적인 레이더 시스템 온 칩(SoC)과 트랜시버를 설계하는 소수의 주요 반도체 기업들이 장악하고 있다. 상위 5개 기업이 전체 시장의 65%에서 90%를 차지할 정도로 집중도가 높다.54

- **NXP 반도체 (네덜란드):** 시장의 선두주자로, SAF8xxx 시리즈 RFCMOS 트랜시버와 S32R 제품군의 레이더 프로세서를 포함하는 확장 가능한 포트폴리오를 제공한다.44 최신 S32R47 프로세서는 16nm FinFET 공정으로 제작되어 레벨 2+에서 레벨 4 시스템을 목표로 한다.55
- **텍사스 인스트루먼츠 (미국):** 45nm RFCMOS 공정 기반의 AWR 및 IWR 시리즈 단일 칩 밀리미터파 센서(예: AWR2944)로 시장의 핵심 플레이어이다. 이들 칩은 트랜시버와 처리 장치를 통합하여 소형화와 저비용화를 실현했다.12
- **인피니언 테크놀로지스 (독일):** 또 다른 유럽의 강자로, 28nm 공정 기반의 RASIC CTRX8191F와 같은 차세대 이미징 레이더용 MMIC를 공급한다.49
- **로버트 보쉬 (독일):** 주요 반도체 제조사이자 세계적인 자동차 부품 공급업체(Tier 1)로서 독특한 위치를 차지하고 있다.54

이들 기업은 4D 이미징 레이더의 근간이 되는 '두뇌'와 '감각기관'을 제공한다. 이들의 기술 로드맵, 예를 들어 공정 미세화나 통합 수준의 향상은 미래 레이더 시스템의 성능, 전력 효율, 그리고 비용을 직접적으로 결정한다.

#### 3.3.2 제 6.2절: 자동차 1차 협력사 (Tier 1)

1차 협력사들은 반도체 기업들로부터 핵심 부품을 공급받아 완성된 레이더 모듈과 시스템으로 통합하여 완성차 업체(OEM)에 납품한다. 이 분야의 주요 기업으로는 2억 개 이상의 레이더 센서를 생산했으며 새로운 4D 장거리 이미징 레이더를 출시 중인 **콘티넨탈 AG (독일)** 54, **마그나 인터내셔널 (캐나다)** 54, **ZF 프리드리히스하펜 AG (독일)** 43, 그리고 **앱티브 (Aptiv)** 15 등이 있다. 이들은 시스템 통합, 소프트웨어 개발, 검증 및 인증 과정에서 부가가치를 창출하며, 반도체 기업과 완성차 업체 사이의 중요한 가교 역할을 한다.

#### 3.3.3 제 6.3절: 혁신가들: 주요 스타트업 프로파일

활발한 스타트업 생태계는 레이더 기술의 한계를 넘어서는 혁신을 주도하고 있다.

- **Arbe (이스라엘):** 초고해상도 레이더 분야의 선두주자로, 48개의 Tx와 48개의 Rx 채널(2,304 가상 채널)을 갖춘 독자적인 칩셋과 프로세서를 개발했다. 'Phoenix' 레이더는 1도의 방위각 해상도를 제공한다.21
- **Vayyar Imaging (이스라엘):** 고도로 통합된 단일 칩 솔루션에 집중하여, 차량 실내 모니터링부터 헬스케어, 산업 안전에 이르는 다기능성을 목표로 한다.43
- **Uhnder (미국):** 세계 최초로 자동차용으로 인증받은 양산형 4D 디지털 레이더 온 칩(Radar-on-Chip)을 선보인 선구자이다.49
- **비트센싱 (대한민국):** 다중 칩 캐스케이딩 기술을 활용하여 192개의 가상 채널을 구현하는 등 첨단 4D 이미징 레이더를 개발하는 한국의 스타트업이다. 자동차 및 스마트 시티 분야에서 기술력을 인정받고 있다.16
- **스마트레이더시스템 (대한민국):** 비균일 배열 안테나 설계와 AI 기반 신호 처리 기술에 강점을 가진 또 다른 한국의 혁신 기업이다. 모빌리티와 비모빌리티 시장 모두를 공략하고 있다.43

이러한 스타트업들은 대기업보다 민첩하게 파괴적 혁신(급진적인 칩셋 아키텍처, AI 기반 소프트웨어 등)에 집중하며, 종종 1차 협력사나 완성차 업체의 주요 파트너십 또는 인수 대상이 된다.

##### 3.3.3.1 표 6.1: 4D 이미징 레이더 시장의 주요 기업 및 제품

| 회사명                  | 국가     | 분류            | 주요 제품/기술                             | 특징 및 파트너십                                   |      |
| ----------------------- | -------- | --------------- | ------------------------------------------ | -------------------------------------------------- | ---- |
| **NXP 반도체**          | 네덜란드 | 반도체          | S32R 프로세서, SAF8xxx 트랜시버            | 16nm FinFET 공정 S32R47, 확장 가능한 포트폴리오    |      |
| **텍사스 인스트루먼츠** | 미국     | 반도체          | AWR/IWR 시리즈 단일 칩 센서 (AWR2944)      | 45nm RFCMOS 공정, 프로세서 및 트랜시버 통합        |      |
| **인피니언**            | 독일     | 반도체          | RASIC 시리즈 MMIC                          | 28nm 공정 기반 차세대 이미징 레이더 칩             |      |
| **콘티넨탈 AG**         | 독일     | Tier 1          | ARS540, 차세대 4D 장거리 레이더            | 누적 2억 개 이상 레이더 생산, 주요 OEM 공급        |      |
| **Arbe Robotics**       | 이스라엘 | 스타트업/혁신가 | Phoenix 레이더, 독자 칩셋                  | 48Tx x 48Rx (2304 가상 채널), 초고해상도           |      |
| **Vayyar Imaging**      | 이스라엘 | 스타트업/혁신가 | Radar-on-Chip (RoC)                        | 단일 칩 다기능 솔루션 (차량 내부, 헬스케어)        |      |
| **Uhnder**              | 미국     | 스타트업/혁신가 | 디지털 코드 변조(DCM) RoC                  | 세계 최초 양산형 디지털 4D 레이더 온 칩            |      |
| **비트센싱**            | 대한민국 | 스타트업/혁신가 | AIR 4D                                     | 다중 칩 캐스케이딩, 192 가상 채널, 스마트시티 적용 |      |
| **스마트레이더시스템**  | 대한민국 | 스타트업/혁신가 | RETINA                                     | 비균일 배열 안테나, AI 기반 신호 처리              |      |
| **Ambarella (Oculii)**  | 미국     | 반도체/혁신가   | CV3 AI 도메인 컨트롤러, Oculii AI 알고리즘 | 소프트웨어 정의 레이더, 중앙 집중식 처리 아키텍처  |      |
| 데이터 출처: 16         |          |                 |                                            |                                                    |      |

#### 3.3.4 제 6.4절: 지역별 시장 역학 및 규제 동인

4D 이미징 레이더 시장의 성장은 지역별로 다른 양상을 보인다. 유럽은 유로 NCAP(Euro NCAP)과 같은 엄격한 안전 규제와 BMW, 폭스바겐 등 주요 완성차 업체 및 콘티넨탈, 보쉬와 같은 강력한 1차 협력사들의 존재로 시장을 선도하고 있다.43 북미 역시 GM, 포드 등 대형 완성차 업체와 신기술에 대한 높은 수용성을 바탕으로 주요 시장을 형성하고 있다.43

아시아-태평양 지역, 특히 중국, 일본, 대한민국은 가장 빠르게 성장하는 시장이다. 토요타, 현대차, BYD와 같은 주요 제조사들이 안전 및 자율주행 기능 강화를 위해 4D 이미징 레이더 기술을 적극적으로 도입하고 있다.43 시장의 성장은 단순히 기술적 우위뿐만 아니라, 일본의 어린이 차량 방치 감지 의무화와 같은 지역별 안전 규제, 소비자 수요, 그리고 각국 자동차 산업의 전략적 우선순위에 의해 크게 영향을 받는다.54

시장의 경쟁 구도는 '수직적 통합'과 '수평적 전문화' 사이의 흥미로운 긴장 관계를 보여준다. 보쉬와 같은 기업은 반도체 칩부터 완성된 시스템까지 가치 사슬의 상당 부분을 내부적으로 통제한다. 반면, TI나 NXP와 같은 기업은 핵심 실리콘 플랫폼을 제공하는 데 전문화되어 있다. 이와 동시에 Arbe나 Vayyar 같은 스타트업들은 칩셋부터 소프트웨어까지 독자적인 풀스택 솔루션을 개발하여 기존 강자들에게 도전하고 있다. 이는 기업들이 서로 파트너이자 경쟁자가 되는 복잡한 협력-경쟁(co-opetition) 구도를 형성하며, 궁극적으로는 와트당, 달러당 최고의 성능을 제공하는 솔루션이 시장의 승자가 될 것이다. 이러한 역동적인 경쟁이 기술 혁신을 가속화하고 있다.

## 4.  심화 주제, 과제 및 미래 전망

이 마지막 부분에서는 4D 이미징 레이더 기술의 가장 복잡하고 미래 지향적인 측면을 탐구한다. 내재된 기술적 한계를 다루고, AI와 소프트웨어의 심대한 영향을 분석하며, 기술의 미래 궤적을 전망한다.

### 4.1  내재된 기술적 과제와 완화 전략

본 장에서는 4D 이미징 레이더의 성능을 발전시키기 위해 극복해야 할 근본적인 공학적 과제들을 이론적 한계에서부터 실제 구현 문제에 이르기까지 탐색한다.

#### 4.1.1 제 7.1절: 해상도의 장벽: 레일리 기준의 극복

레이더의 각도 해상도는 물리적으로 레일리 기준(Rayleigh Criterion)에 의해 제한된다. 이 기준에 따르면, 각도 해상도는 사용되는 전파의 파장에 비례하고 안테나 개구(aperture)의 크기에 반비례한다.19 즉, 더 좋은 해상도를 얻으려면 파장을 줄이거나(더 높은 주파수 사용) 안테나를 키워야 한다. 77 GHz 대역에서 이미징에 필요한 약 1° 수준의 해상도를 달성하려면, 전통적인 방식으로는 가로세로 28cm에 달하는 비현실적으로 큰 안테나가 필요하게 된다.19 이는 차량 전면 범퍼에 통합하기 매우 어렵다.

이 물리적 한계를 극복하기 위해 다음과 같은 하드웨어 및 소프트웨어적 접근법이 사용된다.

- **하드웨어:** 앞서 설명한 MIMO 가상 배열과 여러 칩을 연결하는 캐스케이딩 방식을 통해, 물리적 크기는 작게 유지하면서도 수천 개의 가상 채널을 만들어 매우 큰 유효 개구(effective aperture)를 구현한다.19
- **소프트웨어:** '초해상도(Super-resolution)' 알고리즘을 사용하여 물리적 한계를 넘어 계산적으로 해상도를 향상시킨다. MUSIC(Multiple Signal Classification)이나 ESPRIT(Estimation of Signal Parameters via Rotational Invariance Techniques)과 같은 고전적인 신호 처리 기법이나, 최근에는 딥러닝 기반의 모델들이 사용된다. 이러한 기법들은 해상도를 크게 향상시킬 수 있지만, 막대한 계산 비용을 수반하는 단점이 있다.19

결국 4D 이미징 레이더 기술 개발의 역사는 이 물리 법칙과의 싸움이라고 할 수 있다. 혁신은 차량의 크기, 비용, 전력이라는 제약 조건 내에서 레일리 기준을 '우회'할 수 있는 영리한 하드웨어와 소프트웨어의 조합을 찾는 데 있다.

##### 4.1.1.1 표 7.1: 상용 4D 이미징 레이더 시스템 기술 사양

| 시스템 / 회사          | 각도 해상도 (방위각/고도각) | 최대 탐지 거리 | 시야각 (FoV, 방위각/고도각) | 가상 채널 (Tx x Rx) | 핵심 기술                    |      |
| ---------------------- | --------------------------- | -------------- | --------------------------- | ------------------- | ---------------------------- | ---- |
| **Arbe Phoenix**       | 1° / 1.5°                   | 300 m          | 100° / 30°                  | 2304 (48x48)        | 독자 칩셋, 초고해상도        |      |
| **Continental ARS540** | N/A (고해상도)              | 300 m+         | N/A                         | N/A                 | 장거리 이미징, L2+ 지원      |      |
| **ZF / SAIC Radar**    | N/A (고해상도)              | 350 m          | N/A                         | 192                 | 16x 일반 시스템 채널 수      |      |
| **Ambarella / Oculii** | 0.5° - 1°                   | 300 - 500 m+   | N/A                         | 6x8 (저채널)        | AI 기반 동적 파형, 중앙 처리 |      |
| **Bitsensing AIR 4D**  | 1.5°                        | 300 m+         | 120°                        | 192 (12x16)         | 다중 칩 캐스케이딩           |      |
| **Steradian IRU**      | 1.2° / N/A                  | N/A            | 120° / 30°                  | 256                 | 16채널 E-band MIMO 트랜시버  |      |
| 데이터 출처: 16        |                             |                |                             |                     |                              |      |

#### 4.1.2 제 7.2절: 간섭 문제: 혼잡한 주파수 대역에서의 공존

수백만 개의 레이더가 동일한 77-81 GHz 주파수 대역에서 동시에 작동하게 되면서, 레이더 간 상호 간섭은 심각하고 점증하는 문제로 대두되고 있다.68 간섭은 레이더의 민감도를 저하시키고, 목표물 탐지를 놓치게 하거나, 존재하지 않는 목표물을 탐지하는 '고스트 타겟(ghost target)'을 생성하여 안전에 직접적인 위협이 될 수 있다.70

이 문제를 해결하기 위한 다양한 완화 기술이 연구되고 있다.

- **파형 설계:** 위상 코딩이나 의사 난수(pseudo-random) 파형을 사용하여 다른 레이더의 신호와 직교성을 갖도록 설계함으로써 간섭 확률 자체를 낮춘다.69
- **신호 처리:** 주 FFT 처리 이전에 시간 영역이나 시간-주파수 영역에서 간섭 신호를 식별하고 제거한다. 간섭으로 오염된 샘플을 0으로 만들거나(zeroing), 해당 구간을 무시하는(gating) 방식이 대표적이다.68
- **적응형 빔포밍:** 안테나의 방사 패턴을 실시간으로 조절하여 간섭 신호가 들어오는 방향으로 '널(null)'을 형성함으로써 공간적으로 간섭을 차단한다.70
- **학습 기반 기법:** 딥러닝 모델(예: GAN)을 사용하여 간섭 신호의 통계적 특성을 학습하고, 수신된 신호에서 간섭 성분만을 정교하게 분리하여 제거한다.73

차량 밀도가 높은 도심 주행 환경이 보편화됨에 따라, 강건한 간섭 완화 기술은 센서의 원시 성능만큼이나 중요해질 것이다.

#### 4.1.3 제 7.3절: 데이터 폭증: 계산 복잡성과 실시간 처리

4D 이미징 레이더는 기존 레이더보다 훨씬 방대한 양의 원시 데이터를 생성한다.74 Arbe의 고해상도 시스템은 초당 3 테라비트(Tb/sec)에 달하는 처리량을 요구할 수 있다.21 이 방대한 데이터를 초당 30프레임과 같은 실시간 요구사항에 맞춰 처리하려면 막대한 계산 능력이 필요하다.21 이는 전력 소모와 비용에 제약이 있는 차량용 온보드 프로세서에 큰 부담을 준다.65

다단계 FFT 연산, DBSCAN과 같은 클러스터링 알고리즘, 확장 칼만 필터(EKF)와 같은 추적 필터 등 전통적인 신호 처리 파이프라인 자체의 계산 부하도 상당하다.37 여기에 딥러닝 기반의 인식 알고리즘이 추가되면서 계산 요구사항은 더욱 높아진다.76 이러한 계산 복잡성 문제는 레이더의 성능이 안테나뿐만 아니라 데이터를 해석하는 프로세서의 능력에 의해 제한됨을 의미한다. 이는 NXP의 S32R 시리즈나 TI의 HWA(Hardware Accelerator)와 같이 레이더 처리 전용 하드웨어 가속기를 탑재한 특수 프로세서의 개발과 전력 효율성에 대한 집중적인 연구를 촉발시키는 주요 원인이 되고 있다.

해상도, 간섭, 계산 복잡성이라는 세 가지 주요 과제는 독립적인 문제가 아니라 서로 얽혀 있는 '삼중고(trilemma)'를 형성한다. 하나의 문제를 해결하려는 시도가 종종 다른 문제를 악화시킨다. 예를 들어, 더 많은 MIMO 채널을 사용하여 해상도를 높이면 처리해야 할 데이터 양이 늘어나 계산 부하가 증가하고, 더 넓어진 안테나의 부엽(sidelobe)으로 인해 더 많은 방향에서 간섭에 취약해질 수 있다.19 간섭을 완화하기 위해 복잡한 파형을 사용하면 수신단에서 이를 처리하기 위한 계산량이 더욱 증가한다. 따라서 4D 이미징 레이더 기술의 발전은 안테나, 파형, 신호 처리 알고리즘, 그리고 기반이 되는 컴퓨팅 하드웨어를 동시에 고려하는 '공동 설계(co-design)'를 통해서만 이루어질 수 있다.

### 4.2  미래는 소프트웨어 정의 및 AI 주도형

본 장에서는 고정된 기능의 하드웨어에서 유연하고 지능적인 소프트웨어 중심 시스템으로의 전환에 초점을 맞춰, 차세대 레이더 기술을 정의하는 최첨단 트렌드를 탐색한다.

#### 4.2.1 제 8.1절: 중앙 집중식 및 소프트웨어 정의 아키텍처로의 전환

처리된 객체 목록을 출력하는 '스마트' 엣지 레이더 모듈이라는 전통적인 모델은 도전을 받고 있다. 새로운 트렌드는 여러 개의 '단순한(dumb)' 또는 프로세서가 없는 레이더 헤드가 원시 ADC 데이터나 텐서 데이터를 강력한 중앙 도메인 컨트롤러(예: Nvidia Orin, Ambarella CV3)로 스트리밍하는 중앙 집중식 아키텍처이다.63 이러한 소프트웨어 정의(software-defined) 접근 방식은 여러 이점을 제공한다.

- **유연성:** 중앙 프로세서의 자원을 주행 시나리오에 따라 동적으로 할당할 수 있다. 예를 들어, 고속도로 주행 시에는 전방 장거리 레이더에 처리 능력을 집중시킬 수 있다.64
- **업그레이드 가능성:** 차량 수명 주기 동안 무선 업데이트(Over-the-Air, OTA)를 통해 성능과 기능을 지속적으로 개선할 수 있다.64
- **비용 및 전력:** 엣지 레이더 헤드의 비용과 전력 소모를 줄이고 설계를 단순화한다.64
- **우수한 융합:** '저수준(low-level)' 또는 '원시 데이터 융합(raw data fusion)'을 가능하게 한다. 이는 각 센서가 생성한 객체 목록을 융합하는 것보다 훨씬 더 풍부하고 강건한 환경 모델을 생성할 수 있는 방식이다.38

이것은 자동차 센싱 분야에서 가장 심오한 아키텍처 변화일 것이다. 이는 핵심 지적 재산과 가치를 하드웨어 센서에서 중앙 처리 소프트웨어로 이동시켜, 완성차 업체가 인식 스택에 대한 더 많은 통제권을 갖게 함을 의미한다. Ambarella의 아키텍처가 이러한 트렌드의 대표적인 예이다.63

#### 4.2.2 제 8.2절: 처리 체인 내 AI 및 머신러닝의 역할

인공지능(AI)과 머신러닝(ML)은 더 이상 최종 인식 단계(객체 분류)에만 머무르지 않고, 레이더 신호 처리 체인의 모든 단계에 통합되고 있다.

- **신호 처리:** 딥러닝 모델은 레이더 데이터의 노이즈를 제거하고, 간섭을 완화하며, 초해상도를 달성하는 데 사용되어 종종 전통적인 알고리즘을 능가하는 성능을 보인다.65 Ambarella의 Oculii 기술은 AI를 사용하여 레이더 파형 자체를 주변 환경에 맞게 동적으로 조정한다.63
- **포인트 클라우드 생성:** 포인트를 탐지하는 전통적인 CFAR(Constant False Alarm Rate) 알고리즘은 정보 손실을 유발할 수 있다. 학습 기반 탐지기는 원시 레이더 텐서로부터 더 밀도 높고 정확한 포인트 클라우드를 생성할 수 있다.79
- **종단간(End-to-End) 인식:** 일부 접근법은 중간 단계인 포인트 클라우드 생성을 완전히 건너뛰고, 원시 레이더 텐서(예: 거리-도플러-각도 큐브)에서 직접 객체 탐지나 점유 격자(occupancy grid)와 같은 인식 결과물을 출력하는 심층 신경망을 사용한다.81 이는 더 많은 정보를 보존하지만 계산 요구사항이 매우 높다.

AI를 저수준 신호 처리에 통합하는 것은 패러다임의 전환이다. 이를 통해 시스템은 수작업으로 만든 결정론적 알고리즘으로는 불가능했던 방식으로, 복잡하고 노이즈가 많은 실제 신호에 적응하고 학습할 수 있게 된다. 이는 원시 센서 데이터의 잠재력을 최대한 발휘하기 위한 핵심 열쇠이다.

#### 4.2.3 제 8.3절: 원시 데이터 융합: 환경 모델링의 새로운 지평

중앙 집중식 아키텍처의 궁극적인 목표는 원시 데이터 융합을 가능하게 하는 것이다.38 각 센서가 자체적으로 객체 목록을 만들고 이를 조정하려 애쓰는 대신, 카메라 픽셀, 레이더 텐서, 라이다 포인트와 같은 원시 데이터 스트림을 저수준에서 결합한다. 이를 통해 통일되고 고밀도의 3차원 세계 모델을 생성할 수 있다.38 이 접근법은 신호 대 잡음비를 개선하고, 더 작은 객체의 탐지를 가능하게 하며, 단일 센서의 고장에 대해 더 강건하다.38 최근에는 라이다와 4D 레이더 텐서를 직접 융합하고, 날씨 조건에 따라 각 센서 정보의 가중치를 적응적으로 조절하는 프레임워크도 개발되고 있다.83

원시 데이터 융합은 인식 기술의 '성배'로 여겨진다. 이는 가장 정확하고 강건한 환경 모델을 약속한다. 밀도 높은 4D 텐서를 출력하는 4D 이미징 레이더는 전통적인 레이더의 희소한 객체 목록보다 이러한 유형의 융합에 훨씬 더 적합한 데이터 소스이다.

중앙 집중식 아키텍처, AI 주도 처리, 그리고 원시 데이터 융합의 결합은 '소프트웨어 정의 센서'라는 새로운 개념을 창조하고 있다. 물리적인 레이더 헤드는 점차 범용 데이터 수집 장치가 되어가고, '센서'의 진정한 성능과 차별화된 기능은 중앙 컴퓨터에서 실행되는 소프트웨어에 의해 정의된다. 이는 자동차 공급망과 비즈니스 모델을 근본적으로 재편할 것이다. 완성차 업체는 차량을 판매한 후, 소프트웨어 구독이나 유료 업데이트를 통해 인식 성능 업그레이드를 제공할 수 있게 된다. 센서는 더 이상 정적인 하드웨어가 아니라, 동적으로 진화하는 플랫폼이 되는 것이다. 이는 경제 모델을 일회성 하드웨어 판매에서 반복적인 소프트웨어 수익 모델로 전환시키며, 완성차 업체에게 고객 경험과 차량 성능을 제어할 수 있는 막강한 힘을 부여할 것이다.

### 4.3  결론 및 전략적 전망

본 장에서는 보고서의 분석 결과를 종합하고, FMCW 4D 이미징 레이더의 전략적 중요성에 대한 미래 지향적인 관점을 제공한다.

#### 4.3.1 제 9.1절: 분석 결과 종합: 4D 이미징 레이더의 진화하는 역할

FMCW 4D 이미징 레이더는 단순한 거리/속도 센서에서 출발하여 고해상도 인식 센서로 진화했다. 이러한 발전은 비용 효율적인 FMCW 원리에서 시작되어, MIMO 안테나 배열과 칩 캐스케이딩을 통해 고도화되었으며, 현재는 AI와 소프트웨어 정의 아키텍처에 의해 혁신이 가속화되고 있다. 이 기술은 전천후 강인성, 직접적인 속도 측정 능력, 그리고 지속적으로 향상되는 해상도를 대량 생산이 가능한 비용으로 제공하는 독보적인 조합을 자랑한다.

이 기술의 역할은 보조적인 ADAS 센서에서 레벨 2+ 이상의 자율주행에 필수적인 주 인식 센서로 전환되고 있다. 특히 카메라와 라이다가 취약한 악천후 관련 코너 케이스를 해결하는 핵심 열쇠로서, 안전한 자율주행 시스템에 요구되는 중복성과 신뢰성을 확보하는 데 결정적인 요소로 자리매김하고 있다.

#### 4.3.2 제 9.2절: 전략적 전망 및 미래 궤적

- **기술 궤적:** 앞으로도 더욱 진보된 MIMO 기술과 AI 기반 초해상도 기법을 통해 해상도는 지속적으로 향상될 것이다. AI는 핵심 신호 처리 과정에 더욱 깊숙이 통합되어 더 강건하고 적응력이 뛰어난 센서를 탄생시킬 것이다. 반도체 기술 또한 더 작은 공정 노드에서의 추가적인 통합과 전력 감소를 통해 발전을 거듭할 것이다.
- **시장 궤적:** 자동차 시장이 계속해서 핵심 동력 역할을 할 것이며, 특히 레벨 2+ 시스템이 향후 10년간 대중적으로 채택될 전망이다.40 기술 비용이 하락함에 따라 산업, 헬스케어, 보안 시장으로의 기술 이전이 가속화될 것이다. 경쟁 구도는 전문 부품 공급업체와 풀스택 솔루션 제공업체 간의 역동적인 긴장 관계 속에서 계속 진화할 것이다.
- **최종 결론:** FMCW 4D 이미징 레이더는 단순한 점진적 개선이 아닌, 변혁적인 기술이다. 모든 조건에서 풍부하고 신뢰할 수 있는 인식 데이터를 제공하는 능력은 소프트웨어 정의 아키텍처의 유연성과 결합하여, 자동차와 트럭에서부터 로봇과 드론에 이르기까지 미래 자율 기계의 초석이 될 것이다.

#### **참고 자료**

1. Frequency-Modulated Continuous-Wave Radar (FMCW Radar) - Radartutorial.eu, accessed July 1, 2025, [https://www.radartutorial.eu/02.basics/Frequency%20Modulated%20Continuous%20Wave%20Radar.en.html](https://www.radartutorial.eu/02.basics/Frequency Modulated Continuous Wave Radar.en.html)
2. 자율주행 레이더 센서의 종류 및 작동 원리[대역폭,펄스레이더,연속파레이더,FMCW,FSK,FFT,CFAR] - 짐승Lab, accessed July 1, 2025, https://beast1251.tistory.com/136
3. Basics of FMCW Radar | Renesas, accessed July 1, 2025, https://www.renesas.com/en/blogs/basics-fmcw-radar
4. 현유진; 진영석, accessed July 1, 2025, http://journal.ksae.org/_common/do.php?a=full&bidx=707&aidx=9462
5. jees :: Journal of Electromagnetic Engineering and Science, accessed July 1, 2025, https://www.jees.kr/m/journal/view.php?doi=10.26866/jees.2022.3.r.79
6. FMCW radar simulation in HFSS SBR+ - (주)모아소프트, accessed July 1, 2025, https://moasoftware.co.kr/ansys/fmcw-radar-simulation-in-hfss-sbr/
7. FMCW (Frequency-Modulated Continuous Wave) Radar - 단순하게 - 티스토리, accessed July 1, 2025, https://talktato.tistory.com/71
8. FMCW 레이더의 원리 - Waveturtle - 티스토리, accessed July 1, 2025, https://waveturtle94.tistory.com/4
9. A Novel Signal Processing Technique for Ku-Band Automobile FMCW Fully Polarimetric SAR System Using Triangular LFM - KAIST, accessed July 1, 2025, http://ma.kaist.ac.kr/wp-content/uploads/2021/03/09146876-2.pdf
10. 4D Radar: Extraordinary sensors for the cars of tomorrow - Avnet, accessed July 1, 2025, https://www.avnet.com/apac/resources/article/4d-radar/
11. 딥 러닝: mmWave FMCW 레이더를 사용한 엣지 장치 1부 - 신호 처리 ..., accessed July 1, 2025, [https://hackernoon.com/lang/ko/mmwave-fmcw-%EB%A0%88%EC%9D%B4%EB%8D%94-1%EB%B6%80-%EC%8B%A0%ED%98%B8-%EC%B2%98%EB%A6%AC-%EA%B8%B0%EB%8A%A5%EC%9D%84-%EA%B0%96%EC%B6%98-%EB%94%A5-%EB%9F%AC%EB%8B%9D-%EC%97%A3%EC%A7%80-%EC%9E%A5%EC%B9%98](https://hackernoon.com/lang/ko/mmwave-fmcw-레이더-1부-신호-처리-기능을-갖춘-딥-러닝-엣지-장치)
12. AWR2944 data sheet, product information and support | TI.com, accessed July 1, 2025, https://www.ti.com/product/AWR2944
13. Four-Dimensional (4D) Imaging Radar Market, By Type, By Application, By Country, and By Region - GII Research, accessed July 1, 2025, https://www.giiresearch.com/report/anvi1514957-four-dimensional-4d-imaging-radar-market-by-type.html
14. Future of 4D Imaging Radar Industry: Driving Precision Across Automotive, Healthcare, and Defense - Reportsnreports, accessed July 1, 2025, https://www.reportsnreports.com/semiconductor-and-electronics/future-of-4d-imaging-radar-industry-driving-precision-across-automotive-healthcare-and-defense/
15. What Is 4D Imaging Radar? - Aptiv, accessed July 1, 2025, https://www.aptiv.com/en/insights/article/what-is-4d-imaging-radar
16. Reimagining Radar in the Form of High-Resolution 4D Sensing Systems - EEJournal, accessed July 1, 2025, https://www.eejournal.com/article/reimagining-radar-in-the-form-of-high-resolution-4d-sensing-systems/
17. The Advent of High-Resolution Radar - GSMA, accessed July 1, 2025, https://www.gsma.com/get-involved/gsma-foundry/wp-content/uploads/2024/09/High-Resolution-Radar.pdf
18. Ansys AVxcelerate를 이용한 자율주행 4D 이미징 레이더 시뮬레이션 - 디바인테크놀로지, accessed July 1, 2025, https://divinetechnology.tistory.com/36
19. Automotive Radar Future - Miniaturising Size & Maximising Performance - IDTechEx, accessed July 1, 2025, https://www.idtechex.com/en/research-article/automotive-radar-future-miniaturising-size-and-maximising-performance/30177
20. 4D Imaging Radar Will Replace LiDAR? - Neuvition | solid-state ..., accessed July 1, 2025, https://www.neuvition.com/media/4d-imaging-radar-and-lid.html
21. 4D IMAGING RADAR | Arbe Robotics, accessed July 1, 2025, https://arberobotics.com/wp-content/uploads/2021/05/4D-Imaging-radar-product-overview.pdf
22. A Point Cloud Improvement Method for High-Resolution 4D mmWave Radar Imagery - MDPI, accessed July 1, 2025, https://www.mdpi.com/2072-4292/16/15/2856
23. Antenna Failure Resilience: Deep Learning-Enabled Robust DOA Estimation with Single Snapshot Sparse Arrays This work was supported in part by U.S. National Science Foundation (NSF) under Grants CCF-2153386 and ECCS-2340029. - arXiv, accessed July 1, 2025, https://arxiv.org/html/2405.02788v1
24. Letter - jees :: Journal of Electromagnetic Engineering and Science, accessed July 1, 2025, https://www.jees.kr/m/journal/view.php?number=3711
25. Simulate an Automotive 4D Imaging MIMO Radar - MATLAB & Simulink - MathWorks, accessed July 1, 2025, https://www.mathworks.com/help/radar/ug/simulate-an-automotive-4d-imaging-mimo-radar.html
26. LiDAR vs. Radar: What's the Difference? - JOUAV, accessed July 1, 2025, https://www.jouav.com/blog/lidar-vs-radar.html
27. LiDAR vs. RADAR : A comprehensive comparison - YellowScan, accessed July 1, 2025, https://www.yellowscan.com/knowledge/lidar-vs-radar/
28. 4D Imaging Radar Market Size, Share, Industry Forecast by 2032 - Emergen Research, accessed July 1, 2025, https://www.emergenresearch.com/industry-report/four-dimensional-imaging-radar-market
29. 4D Radar Improves Safety and Accuracy in Automotive and Industrial Applications, accessed July 1, 2025, https://www.eetimes.eu/4d-radar-improves-safety-and-accuracy-in-automotive-and-industrial-applications/
30. [오토저널] 자율주행을 위한 4D 이미징 레이더의 진화 - Daum, accessed July 1, 2025, https://v.daum.net/v/zawFEbOKs1
31. [로그人] 이재은 대표 "4D이미징 레이더, 라이다 대체 가능" - IT조선, accessed July 1, 2025, https://it.chosun.com/news/articleView.html?idxno=2022011500402
32. A detailed comparison of LiDAR, Radar and Camera Technology - Insights | Outsight, accessed July 1, 2025, https://insights.outsight.ai/how-does-lidar-compares-to-cameras-and-radars/
33. A Multi-modal Dataset with Dual 4D Radar for Autononous Driving - PMC, accessed July 1, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11907064/
34. SMURF: Spatial Multi-Representation Fusion for 3D Object Detection with 4D Imaging Radar - arXiv, accessed July 1, 2025, https://arxiv.org/pdf/2307.10784
35. Camera versus Lidar - The Last Driver License Holder, accessed July 1, 2025, https://thelastdriverlicenseholder.com/2025/06/25/camera-versus-lidar/
36. "AI 탑재 4D 이미지 레이더, 자율주행 넘어 모든 분야 본다"...김용환 ..., accessed July 1, 2025, https://www.aitimes.com/news/articleView.html?idxno=138692
37. Integrated Sensor Fusion Based on 4D Mimo Radar and Camera - Nxtbook Media, accessed July 1, 2025, https://read.nxtbook.com/ieee/vehicular_technology/vehiculartechnology_dec_2022/integrated_sensor_fusion_base.html
38. Sensor Fusion and Perception Technology Fundamentals - FAQ - LeddarTech, accessed July 1, 2025, https://leddartech.com/sensor-fusion-perception-technology-fundamentals-faq/
39. A cool guide to how how cameras, radar and LiDAR work, pros and cons - Reddit, accessed July 1, 2025, https://www.reddit.com/r/coolguides/comments/171fsoo/a_cool_guide_to_how_how_cameras_radar_and_lidar/
40. L2+ 자율주행 구현을 위한 핵심 센서 '4D 이미징 레이더(RADAR)' - 비전시스템, accessed July 1, 2025, https://visionsystem.kr/issue-news?tpf=board/view&board_code=2&code=5274
41. 세계의 4D 이미징 레이더 시장 보고서(2025년) - 글로벌인포메이션, accessed July 1, 2025, https://www.giikorea.co.kr/report/tbrc1686441-four-dimensional-4d-imaging-radar-global-market.html
42. 4D Imaging Radar Industry worth $1,206.9 million by 2030 - MarketsandMarkets, accessed July 1, 2025, https://www.marketsandmarkets.com/PressReleases/4d-imaging-radar.asp
43. 4D Imaging Radar Market By Size Share Future Forecast [Latest], accessed July 1, 2025, https://www.strategicmarketresearch.com/market-report/4d-imaging-radar-market
44. The Importance of Imaging Radar - NXP Semiconductors, accessed July 1, 2025, https://www.nxp.com/company/about-nxp/smarter-world-blog/BL-THE-IMPORTANCE-OF-IMAGING-RADAR
45. 완전 자율주행 시대를 위한 센서 솔루션, 4D 이미징 RADAR - LG이노텍, accessed July 1, 2025, https://www.lginnotek.com/news/insightsView.do?idx=96
46. '자율주행 4D 레이더 기술' 스마트레이더시스템, 48억 투자 유치 - IT조선, accessed July 1, 2025, https://it.chosun.com/news/articleView.html?idxno=2019051301347
47. 스마트레이더시스템 "4D 레이다분야 글로벌 톱티어 목표" - 데일리한국, accessed July 1, 2025, https://daily.hankooki.com/news/articleView.html?idxno=986914
48. 4D Radar for Defense & Security - Hyper Tech, accessed July 1, 2025, https://www.hypertech.co.il/product/4d-radar-for-defense-security/
49. 4D Imaging Radar Market Revenue Trends and Growth Drivers - MarketsandMarkets, accessed July 1, 2025, https://www.marketsandmarkets.com/Market-Reports/4d-imaging-radar-market-226035909.html
50. 자율주행 혁신: 레이더와 카메라 융합 - Goover, accessed July 1, 2025, https://seo.goover.ai/report/202410/go-public-report-ko-a8d153bc-d3f2-4281-86c7-5fbf4eececc1-0-0.html
51. Vayyar: Imaging Radar From The Global Leader, accessed July 1, 2025, https://vayyar.com/
52. 스마트레이더시스템(424960), accessed July 1, 2025, https://ssl.pstatic.net/imgstock/upload/research/company/1712099157378.pdf
53. [제조혁신 이노비즈]4D이미징레이더로 특장차/자율주행 세계시장 석권 '스마트레이더시스템', accessed July 1, 2025, https://www.etnews.com/20230713000211
54. 4D Imaging Radar Companies - Texas Instruments Incorporated (US) and NXP Semiconductors (Netherlands) are the Key Players - MarketsandMarkets, accessed July 1, 2025, https://www.marketsandmarkets.com/ResearchInsight/4d-imaging-radar-market.asp
55. NXP Unveils Third-Generation Imaging Radar Processors for Level 2+ to 4 Autonomous Driving, accessed July 1, 2025, https://www.nxp.com/company/about-nxp/newsroom/NW-NXP-UNVEILS-THIRD-GENERATION-RADAR
56. NXP Unveils Third-Generation Imaging Radar Processors for Level 2+ to 4 Autonomous Driving - GlobeNewswire, accessed July 1, 2025, https://www.globenewswire.com/news-release/2025/05/08/3076849/0/en/NXP-Unveils-Third-Generation-Imaging-Radar-Processors-for-Level-2-to-4-Autonomous-Driving.html
57. AWR2943/AWR2944 Single-Chip 76 to 81GHz FMCW Radar Sensor datasheet (Rev. D) - Texas Instruments, accessed July 1, 2025, https://www.ti.com/lit/ds/symlink/awr2944.pdf
58. Continental Reaches 200 Million Radar Milestone for Greater Safety and the Mobility of Tomorrow, accessed July 1, 2025, https://www.continental.com/en-us/press/press-releases/continental-reaches-200-million-radar-milestone-for-greater-safety-and-the-mobility-of-tomorrow/
59. Meet the radar and lidar innovators at CES: Tech highlights | Segments.ai, accessed July 1, 2025, https://segments.ai/blog/radar-and-lidar-innovators-at-ces/
60. 4D imaging radar to solve growing complexity of in-cabin safety | Automotive World, accessed July 1, 2025, https://www.automotiveworld.com/articles/4d-imaging-radar-could-solve-growing-complexity-of-in-cabin-safety/
61. [인터뷰] 이재은 비트센싱 대표 "진짜 자율주행엔 이미징 레이더가 필수...주요 자동차 업체와 사업 논의 중", accessed July 1, 2025, https://www.thelec.kr/news/articleView.html?idxno=16322
62. [서울대학교 시흥캠퍼스본부 창업보육센터 입주기업] 4D 이미징 레이더 센서를 개발하는 스타트업 '스마트레이더시스템' - 매거진한경, accessed July 1, 2025, https://magazine.hankyung.com/job-joy/article/202402207542d
63. Ambarella Reveals 4D Imaging Radar Architecture for Autonomous Driving - Robotics 24/7, accessed July 1, 2025, https://www.robotics247.com/article/ambarella_reveals_4d_imaging_radar_architecture_for_autonomous_driving
64. Ambarella Unveils World's First Centrally Processed 4D Imaging Radar Architecture for Autonomous Mobility Systems, accessed July 1, 2025, https://www.ambarella.com/news/ambarella-unveils-worlds-first-centrally-processed-4d-imaging-radar-architecture-for-autonomous-mobility-systems/
65. 4D High-Resolution Imagery of Point Clouds for Automotive mmWave Radar | Request PDF, accessed July 1, 2025, https://www.researchgate.net/publication/369459190_4D_High-Resolution_Imagery_of_Point_Clouds_for_Automotive_mmWave_Radar
66. 4D Radar Improves Safety and Accuracy in Automotive and Industrial Applications - EEWeb, accessed July 1, 2025, https://www.eeweb.com/4d-radar-improves-safety-and-accuracy-in-automotive-and-industrial-applications/
67. Lotus Deploys Ambarella's Oculii™ AI 4D Imaging Radar Technology in L2+ Semi-Autonomous Systems for Eletre SUV and Emeya Hyper-GT Electric Vehicles, accessed July 1, 2025, https://investor.ambarella.com/news-releases/news-release-details/lotus-deploys-ambarellas-oculiitm-ai-4d-imaging-radar-technology
68. arXiv:2402.14018v1 [eess.SP] 21 Feb 2024, accessed July 1, 2025, https://arxiv.org/pdf/2402.14018
69. Waveform Design for Mutual Interference Mitigation in Automotive Radar - arXiv, accessed July 1, 2025, https://arxiv.org/pdf/2208.04398
70. INTERFERENCE MITIGATION TECHNIQUES IN FMCW AUTOMOTIVE RADARS Muhammad Rameez - DiVA portal, accessed July 1, 2025, http://www.diva-portal.org/smash/get/diva2:1421631/FULLTEXT02.pdf
71. Incoherent Interference Detection and Mitigation for Millimeter-Wave FMCW Radars - MDPI, accessed July 1, 2025, https://www.mdpi.com/2072-4292/14/19/4817
72. Radar Interference Mitigation: A Review - Preprints.org, accessed July 1, 2025, https://www.preprints.org/manuscript/202412.2182/v1
73. Unsupervised Deep Interference Mitigation for Automotive Radar | Request PDF - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/362198807_Unsupervised_Deep_Interference_Mitigation_for_Automotive_Radar
74. 4D Millimeter-Wave Radar in Autonomous Driving: A Survey - :::::: AEL ::::::, accessed July 1, 2025, http://ael.chungbuk.ac.kr/lectures/graduate/antenna-measurement/2024-1/08-4d-imaging-radar.pdf
75. 4D Millimeter-Wave Radar in Autonomous Driving: A Survey - arXiv, accessed July 1, 2025, https://arxiv.org/html/2306.04242v3
76. RadarOcc: Robust 3D Occupancy Prediction with 4D Imaging Radar - OpenReview, accessed July 1, 2025, [https://openreview.net/forum?id=2oZea6pKhl&referrer=%5Bthe%20profile%20of%20Fangqiang%20Ding%5D(%2Fprofile%3Fid%3D~Fangqiang_Ding1)](https://openreview.net/forum?id=2oZea6pKhl&referrer=[the+profile+of+Fangqiang+Ding](/profile?id%3D~Fangqiang_Ding1))
77. Enabling Higher-Performance 4D Imaging Radar With Central AI Processing of Raw Data, accessed July 1, 2025, https://auto-sens.com/watch/enabling-higher-performance-4d-imaging-radar-with-central-ai-processing-of-raw-data/
78. Radar - Greenerwave, accessed July 1, 2025, https://www.greenerwave.com/solutions/radar/
79. 4DR P2T: 4D Radar Tensor Synthesis with Point Clouds - arXiv, accessed July 1, 2025, https://arxiv.org/html/2502.05550v1
80. DenserRadar: A 4D millimeter-wave radar point cloud detector based on dense LiDAR point clouds - arXiv, accessed July 1, 2025, https://arxiv.org/pdf/2405.05131
81. [2505.12310] DNOI-4DRO: Deep 4D Radar Odometry with Differentiable Neural-Optimization Iterations - arXiv, accessed July 1, 2025, https://arxiv.org/abs/2505.12310
82. [2405.14014] RadarOcc: Robust 3D Occupancy Prediction with 4D Imaging Radar - arXiv, accessed July 1, 2025, https://arxiv.org/abs/2405.14014

Towards Robust 3D Object Detection with LiDAR and 4D Radar Fusion in Various Weather Conditions - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content/CVPR2024/papers/Chae_Towards_Robust_3D_Object_Detection_with_LiDAR_and_4D_Radar_CVPR_2024_paper.pdf