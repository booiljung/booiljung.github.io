#  직교 주파수 분할 다중 (OFDM)

## 서론

### 0.1  OFDM의 정의와 현대 통신 시스템에서의 위상

직교 주파수 분할 다중(Orthogonal Frequency Division Multiplexing, OFDM)은 고속의 데이터 스트림을 다수의 저속 부반송파(subcarrier)로 분할하여 병렬로 전송하는 디지털 다중 반송파 변조 방식이다.1 이 기술의 핵심은 다수의 부반송파가 주파수 영역에서 서로 밀접하게 중첩되면서도, 시간 영역에서는 수학적 '직교성(Orthogonality)'을 유지함으로써 상호 간섭 없이 데이터를 효율적으로 전송하는 데 있다.3

OFDM은 다중 경로 페이딩(multipath fading)에 대한 강인성과 높은 주파수 이용 효율이라는 두 가지 핵심 장점 덕분에 현대 유무선 광대역 통신 시스템의 근간을 이루는 핵심 기술로 자리 잡았다.3 오늘날 4G LTE, 5G NR(New Radio)과 같은 이동통신 시스템, Wi-Fi(IEEE 802.11a/g/n/ac/ax)로 대표되는 무선 근거리 통신망(WLAN), 그리고 유럽식 디지털 방송(DVB) 및 디지털 오디오 방송(DAB) 등 다양한 분야에서 표준 기술로 채택되어 널리 사용되고 있다.6 이는 OFDM이 변조(modulation)와 다중화(multiplexing)를 효과적으로 결합하여, 하나의 주 신호를 여러 개의 독립적인 부 채널로 분할한 후 이를 다시 다중화하는 방식을 통해 제한된 주파수 자원 하에서 고속 데이터 전송의 한계를 극복했기 때문이다.2

### 0.2  기술의 역사적 발전 과정과 핵심 철학

OFDM의 기본 개념은 1950년대에 처음 제안되었으며, 1966년 벨 연구소(Bell Labs)의 R. W. Chang에 의해 주파수 분할 다중(Frequency Division Multiplexing, FDM)의 특수한 형태로 구체화되었다.6 초기 연구는 복잡하고 비싼 등화기(equalizer)의 사용을 피하면서 다중 경로 페이딩과 펄스 형태의 잡음에 강한 통신 방식을 구현하는 데 초점을 맞추었다.6

1971년에는 변복조 과정에 이산 푸리에 변환(Discrete Fourier Transform, DFT)을 적용하여 다수의 부반송파를 효율적으로 생성하고 분리하는 획기적인 아이디어가 고안되었다.6 이는 복잡한 아날로그 필터 뱅크를 단일 디지털 신호 처리 알고리즘으로 대체할 수 있는 가능성을 열었으나, 당시의 반도체 기술 수준으로는 고속의 DFT 연산을 실시간으로 처리할 수 있는 초고밀도 집적회로(VLSI) 칩이 부재하여 상용화로 이어지지 못했다.

기술적 돌파구는 1990년대에 고속 푸리에 변환(Fast Fourier Transform, FFT) 칩 기술이 비약적으로 발전하면서 마련되었다.6 저렴하고 효율적인 FFT 연산이 가능해지면서 OFDM의 상용화가 급물살을 탔다. ADSL, HDSL과 같은 유선 디지털 가입자 회선 기술에 먼저 성공적으로 적용되었고, 이후 그 가능성을 인정받아 DAB, DVB-T, WLAN(IEEE 802.11a) 등 다양한 무선 통신 및 방송 표준으로 채택되며 오늘날의 위상을 갖추게 되었다.6 이처럼 OFDM의 성공은 '직교성'이라는 순수한 수학적 이상과 'FFT'라는 효율적인 공학적 구현의 성공적인 결합에 기인한다. 이론과 실제 사이의 간극을 디지털 신호 처리 기술의 발전이 메워준 대표적인 사례라 할 수 있다.

### 0.3  FDM, CDMA와의 비교를 통한 OFDM의 본질 이해

OFDM의 본질을 이해하기 위해서는 기존의 다중화 방식인 FDM 및 코드 분할 다중 접속(Code Division Multiple Access, CDMA)과의 비교가 유용하다.

- **FDM과의 비교**: 전통적인 FDM은 주파수 축을 여러 채널로 분할하되, 채널 간 상호 간섭을 방지하기 위해 상당한 폭의 보호 대역(Guard Band)을 삽입한다.3 이 보호 대역은 데이터를 전송하는 데 사용되지 않으므로 주파수 이용 효율을 떨어뜨리는 주요 원인이 된다. 반면, OFDM은 부반송파 간의 직교성을 이용하여 보호 대역 없이 스펙트럼을 서로 겹쳐서 사용한다. 이로 인해 이론적으로 주파수 효율을 최대 50%까지 향상시킬 수 있다.3
- **CDMA와의 비교**: 2G, 3G 이동통신에 사용된 CDMA는 서로 다른 사용자에게 직교하는 코드(orthogonal code)를 할당하여, 모든 사용자가 동일한 주파수와 시간 자원을 공유하면서도 자신의 신호만 분리해낼 수 있도록 한다.3 OFDM은 이러한 '직교성을 이용한 분리'라는 아이디어를 주파수 영역으로 가져왔다. 즉, CDMA가 코드 영역에서의 직교성을 활용한다면, OFDM은 주파수 영역에서 부반송파 간의 직교성을 활용하여 상호 간섭 없이 데이터를 병렬 전송한다.2

`표 1`은 이 세 가지 기술의 핵심적인 차이점을 요약한다.

**표 1: FDM, CDMA, OFDM 기술 비교**

| 비교 항목            | 주파수 분할 다중 (FDM) | 코드 분할 다중 접속 (CDMA)  | 직교 주파수 분할 다중 (OFDM) |
| -------------------- | ---------------------- | --------------------------- | ---------------------------- |
| **자원 분할 기준**   | 주파수                 | 코드                        | 주파수 (직교 부반송파)       |
| **보호 요소**        | 보호 대역 (Guard Band) | 직교 코드 (Orthogonal Code) | 순환 전치 (Cyclic Prefix)    |
| **주파수 효율**      | 낮음                   | 높음                        | 매우 높음                    |
| **주된 간섭**        | 인접 채널 간섭 (ACI)   | 다중 사용자 간섭 (MUI)      | 반송파 간 간섭 (ICI)         |
| **다중 경로 강인성** | 약함                   | 강함                        | 매우 강함 (등화 용이)        |

## 1.  OFDM의 수학적 원리: 직교성(Orthogonality)

### 1.1  부반송파의 개념과 다중 반송파 변조

OFDM은 전송하려는 전체 광대역 채널을 다수의 독립적인 협대역(narrowband) 부채널로 나누고, 각 부채널에 부반송파(subcarrier)를 할당하여 데이터를 전송하는 다중 반송파 변조(Multi-Carrier Modulation) 방식이다.10 이는 하나의 큰 트럭으로 모든 화물을 운송할 때 발생하는 위험(예: 사고 시 모든 화물 손실)을 여러 대의 작은 트럭으로 나누어 운송함으로써 분산시키는 것에 비유할 수 있다.2

이러한 접근 방식은 무선 통신의 고질적인 문제인 주파수 선택적 페이딩(frequency selective fading)을 효과적으로 해결한다. 주파수 선택적 페이딩은 채널의 특정 주파수 대역에서 신호 감쇠가 심하게 일어나는 현상으로, 광대역 단일 반송파 시스템에서는 전체 신호에 치명적인 왜곡을 유발할 수 있다. 하지만 OFDM에서는 광대역 채널이 다수의 협대역 부채널로 나뉘므로, 각 부채널은 거의 균일한 감쇠를 겪는 평탄 페이딩(flat fading) 채널처럼 동작하게 된다.5 따라서 일부 부반송파가 깊은 페이딩의 영향을 받더라도, 다른 부반송파들은 정상적으로 전송되어 전체 데이터 손실을 막고 수신단에서의 채널 등화를 훨씬 용이하게 만든다.

### 1.2  직교성의 수학적 정의와 조건

OFDM에서 '직교성'이란, 한 부반송파의 신호가 다른 부반송파의 신호 검출에 아무런 영향을 미치지 않는 성질을 의미한다. 수학적으로 두 함수가 직교한다는 것은 특정 구간에서 두 함수의 곱을 적분한 값, 즉 내적(inner product)이 0이 되는 것을 의미한다.9

`k`번째 부반송파와 `l`번째 부반송파를 각각 복소 지수 함수 $\phi_k(t) = e^{j2\pi f_k t}$와 $\phi_l(t) = e^{j2\pi f_l t}$로 표현할 때, 한 OFDM 심볼 주기 $T_s$ 동안 두 부반송파가 직교하기 위한 조건은 다음과 같다 ($k \neq l$일 때).7
$$
<\phi_k, \phi_l> = \int_{0}^{T_s} \phi_k(t) \phi_l^*(t) dt = \int_{0}^{T_s} e^{j2\pi f_k t} e^{-j2\pi f_l t} dt = \int_{0}^{T_s} e^{j2\pi(f_k - f_l)t} dt = 0
$$
위의 적분식이 0이 되기 위해서는 오일러 공식에 의해 $(f_k - f_l)T_s$가 0이 아닌 정수 값을 가져야 한다. 시스템 내의 모든 부반송파 쌍이 이 조건을 만족하려면, 인접한 부반송파 간의 주파수 간격 $\Delta f$가 심볼 주기의 역수와 같아야 한다.7
$$
\Delta f = \frac{1}{T_s}
$$
`이 조건은 `k`번째 부반송파의 주파수 $f_k$가 기본 주파수 $\Delta f$의 정수배, 즉 $f_k = k \cdot \Delta f = k/T_s$ (또는 기준 주파수 $f_0$를 더해 $f_0 + k/T_s$) 형태를 가져야 함을 의미한다.

### 1.3  주파수 영역에서의 스펙트럼 중첩과 직교성의 시각적 해석

주파수 영역에서 사각 펄스 형태인 시간 영역 심볼은 sinc 함수($\text{sinc}(f) = \sin(\pi f) / (\pi f)$) 형태의 스펙트럼을 가진다.11 OFDM에서는 다수의 부반송파를 사용하므로, 전체 스펙트럼은 각 부반송파의 sinc 함수 스펙트럼들이 서로 심하게 중첩된 형태로 나타난다.6

전통적인 FDM에서는 이러한 스펙트럼 중첩이 곧바로 인접 채널 간섭(ACI)을 의미하므로 보호 대역을 두어 스펙트럼을 분리해야만 했다. 그러나 OFDM에서는 직교 조건 $\Delta f = 1/T_s$가 마법과 같은 역할을 한다. 이 조건을 만족하면, 특정 부반송파의 스펙트럼에서 신호 에너지가 최대가 되는 지점(peak)이 다른 모든 부반송파의 스펙트럼에서는 정확히 에너지가 0이 되는 지점(null 또는 zero-crossing)과 일치하게 된다.3

따라서 수신기에서 특정 부반송파 $f_k$의 주파수 성분을 추출하기 위해 해당 주파수에서 샘플링(또는 상관 연산)을 수행하면, $f_k$ 자체의 신호 성분만 검출되고 다른 모든 부반송파 $f_l$ ($l \neq k$)의 기여분은 0이 되어 이론적으로 완벽하게 분리된다.3 이것이 바로 OFDM이 보호 대역 없이도 부반송파들을 매우 조밀하게 배치하여 높은 주파수 효율을 달성할 수 있는 근본적인 원리이다.

### 1.4  직교성 상실(Loss of Orthogonality)의 원인과 영향

OFDM의 직교성은 송신기와 수신기 간의 완벽한 동기화를 전제로 하는 이상적인 조건이다. 실제 통신 환경에서는 여러 요인으로 인해 직교성이 깨질 수 있으며, 이는 시스템 성능에 치명적인 영향을 미친다.

- **반송파 주파수 오프셋 (Carrier Frequency Offset, CFO):** 송신기와 수신기에서 사용하는 국부 발진기(local oscillator)의 미세한 주파수 차이나 부정확성으로 인해 발생한다.7 CFO는 수신된 신호의 전체 스펙트럼을 주파수 축에서 미세하게 이동시킨다. 이로 인해 한 부반송파의 피크가 더 이상 다른 부반송파의 영점에 위치하지 않게 되어 부반송파 간 간섭(Inter-Carrier Interference, ICI)을 유발하고, 신호 대 잡음비(SNR)를 급격히 감소시킨다.10
- **도플러 효과 (Doppler Shift):** 송신기나 수신기가 고속으로 이동할 때 발생하는 주파수 편이 현상이다.7 도플러 효과 역시 CFO와 마찬가지로 직교성을 파괴하여 ICI를 유발한다. 특히 다중 경로 채널 환경과 결합되면, 각기 다른 경로를 통해 들어오는 신호들이 서로 다른 도플러 편이를 겪게 되어 문제를 더욱 복잡하게 만들고 성능 저하를 심화시킨다.7

이처럼 OFDM의 근간을 이루는 '직교성'은 높은 효율을 부여하는 핵심 강점인 동시에, 외부 교란에 매우 민감하게 반응하는 치명적인 약점(Achilles' heel)이기도 하다. 이 이중적 특성으로 인해 OFDM 시스템 설계에서는 매우 정밀하고 복잡한 동기화 기술의 구현이 필수적이며, 이는 시스템 복잡도와 비용을 증가시키는 주요 요인이 된다.

## 2.  OFDM 시스템의 신호 처리 구조

OFDM 시스템은 송신기와 수신기의 복잡한 디지털 신호 처리 과정을 통해 구현된다. 이 과정의 핵심은 수학적으로 정의된 직교성을 효율적인 알고리즘인 IFFT와 FFT를 통해 실현하는 것이다.

### 2.1  송신기(Transmitter) 블록 다이어그램 상세 분석

OFDM 송신기는 입력된 디지털 비트 스트림을 무선 채널을 통해 전송할 수 있는 아날로그 RF 신호로 변환하는 일련의 과정을 수행한다.15

1. **채널 코딩 및 인터리빙 (Channel Coding & Interleaving):** 전송할 원본 데이터 비트에 순방향 오류 정정(Forward Error Correction, FEC) 부호를 적용하여 채널에서 발생할 수 있는 비트 오류에 대한 강인성을 확보한다.16 이후 인터리빙 과정을 통해 부호화된 비트들을 시간 및 주파수 영역에 걸쳐 재배치한다. 이는 특정 주파수 대역에 집중된 페이딩으로 인해 연속적인 비트 오류(burst error)가 발생하는 것을 방지하고, FEC의 오류 정정 능력을 극대화하는 역할을 한다.8

2. **직렬-병렬 변환 (Serial-to-Parallel, S/P):** 고속의 직렬 비트 스트림을 `N`개의 병렬 스트림으로 변환한다. 여기서 `N`은 사용할 부반송파의 수와 같다.15 이 과정을 통해 하나의 긴 심볼을 

   `N`개의 짧은 심볼로 나누는 대신, 하나의 짧은 심볼을 `N`개의 긴 심볼로 변환하는 효과를 얻는다. 즉, 각 부반송파를 통해 전송되는 심볼의 지속 시간은 `N`배로 길어지며, 이는 다중 경로 지연에 대한 내성을 높이는 데 기여한다.1

3. **심볼 매핑 (Symbol Mapping):** 각각의 병렬 비트 그룹을 QPSK, 16-QAM, 64-QAM 등과 같은 디지털 변조 방식의 성상도(constellation)에 따라 하나의 복소수 심볼($X_k$)로 변환한다.8 이 복소수 값은 해당 부반송파의 진폭과 위상 정보를 담고 있으며, 주파수 영역에서의 데이터를 나타낸다.11

4. **IFFT (Inverse Fast Fourier Transform) 변조:** `N`개의 주파수 영역 복소수 심볼 $X_k$를 입력받아 `N`포인트 IFFT 연산을 수행한다. 이 과정은 `N`개의 직교하는 정현파(부반송파)를 각각의 $X_k$ 값으로 변조하여 모두 합산하는 것과 수학적으로 동일하며, 결과적으로 `N`개의 샘플로 구성된 시간 영역 OFDM 심볼 $x[n]$을 생성한다.9 IFFT는 이 복잡한 변조 과정을 매우 효율적으로 구현하는 핵심 알고리즘이다. 시간 영역 신호 

   $x[n]$은 다음과 같이 표현된다.
   $$
   x[n] = \frac{1}{\sqrt{N}} \sum_{k=0}^{N-1} X_k e^{j2\pi \frac{kn}{N}}, \quad n=0, 1, \dots, N-1
   $$
   5

5. **순환 전치 (Cyclic Prefix, CP) 삽입:** IFFT를 통해 생성된 시간 영역 심볼의 마지막 $N_{CP}$개 샘플을 복사하여 심볼의 맨 앞에 덧붙인다.12 이 CP는 다중 경로 채널로 인한 심볼 간 간섭(ISI)과 반송파 간 간섭(ICI)을 방지하는 핵심적인 역할을 한다 (4장에서 상세히 후술).

6. **병렬-직렬 변환(P/S), DAC 및 상향 변환:** CP가 추가된 병렬 샘플들을 다시 하나의 직렬 디지털 스트림으로 변환하고 18, 이를 디지털-아날로그 변환기(DAC)를 통해 아날로그 파형으로 바꾼다.15 마지막으로, 이 기저대역(baseband) 신호를 믹서를 사용하여 원하는 RF 반송파 주파수로 상향 변환(up-conversion)한 후 증폭하여 안테나를 통해 송신한다.7

### 2.2  수신기(Receiver) 블록 다이어그램 상세 분석

수신기는 송신기의 과정을 정확히 역순으로 수행하여 원래의 데이터 비트를 복원한다.15

1. **하향 변환(Down-conversion) 및 ADC:** 안테나로 수신된 RF 신호를 기저대역 신호로 주파수를 낮추고 7, 아날로그-디지털 변환기(ADC)를 통해 디지털 샘플 스트림으로 변환한다.15

2. **동기화 (Synchronization):** 수신된 신호로부터 OFDM 심볼의 정확한 시작 위치를 찾는 시간 동기화와 송신기와 수신기 간의 주파수 오차를 추정하고 보상하는 주파수 동기화를 수행한다.1 이 과정은 직교성을 유지하고 시스템 성능을 보장하는 데 매우 중요하다.22

3. **순환 전치 (CP) 제거:** 각 OFDM 심볼의 앞부분에 삽입되었던 CP 구간을 제거하여, IFFT의 출력에 해당하는 원래의 `N`개 유효 데이터 샘플만을 남긴다.12

4. **FFT (Fast Fourier Transform) 복조:** CP가 제거된 `N`개의 시간 영역 샘플 $y[n]$에 `N`포인트 FFT 연산을 수행한다. 이는 시간 영역 신호를 다시 주파수 영역으로 변환하는 과정으로, 각 부반송파에 실려 있는 복소수 심볼 $Y_k$를 분리해내는 역할을 한다.19
   $$
   Y_k = \sum_{n=0}^{N-1} y[n] e^{-j2\pi \frac{kn}{N}}, \quad k=0, 1, \dots, N-1
   $$
   24

5. **채널 추정 및 등화 (Channel Estimation & Equalization):** 송신단에서 미리 약속된 위치에 삽입한 파일럿(pilot) 심볼을 이용하여, 무선 채널이 각 부반송파에 미친 왜곡(진폭 감쇠 및 위상 변화) $H_k$를 추정한다.5 이후, 추정된 채널 값 

   $H_k$의 역수를 수신된 심볼 $Y_k$에 곱하여($\hat{X}_k = Y_k / H_k$) 채널에 의한 왜곡을 보상한다. 이 과정을 등화라고 하며, OFDM에서는 각 부반송파별로 독립적인 복소수 나눗셈 연산으로 간단하게 구현된다.10

6. **디매핑, 병렬-직렬 변환, 디코딩:** 등화된 복소수 심볼 $\hat{X}_k$를 원래의 비트 그룹으로 변환(디매핑)하고, `N`개의 병렬 스트림을 다시 하나의 직렬 스트림으로 결합한 후 15, 채널 디코딩 과정을 거쳐 최종적으로 원본 데이터 비트를 복원한다.

OFDM 시스템의 송수신 과정은 본질적으로 '주파수 영역 처리' 시스템의 특성을 보인다. 데이터의 생성, 왜곡 보상, 복원 등 핵심적인 과정이 모두 주파수 영역에서 정의되고 처리된다. 시간 영역 신호는 단지 전송을 위한 매개체 역할을 할 뿐, 시스템의 근본적인 로직은 주파수 영역에서 이루어진다. 이 '주파수 중심적' 설계 철학이 OFDM의 복잡도 감소와 성능 향상의 근원이라 할 수 있다.

`표 2`는 OFDM 송수신기 주요 구성 요소의 기능과 신호의 형태를 정리한 것이다.

**표 2: OFDM 송수신기 주요 구성 요소 및 기능**

| 구분       | 블록            | 주요 기능                                            | 신호 영역           |
| ---------- | --------------- | ---------------------------------------------------- | ------------------- |
| **송신기** | S/P 변환        | 고속 직렬 스트림을 저속 병렬 스트림으로 변환         | 비트/심볼           |
|            | 심볼 매핑       | 비트 그룹을 복소수 QAM/PSK 심볼로 변환               | 주파수 영역, 디지털 |
|            | IFFT            | 주파수 영역 심볼을 시간 영역 샘플로 변환 (변조)      | 시간 영역, 디지털   |
|            | CP 삽입         | ISI/ICI 방지를 위해 심볼 끝부분을 복사하여 앞에 추가 | 시간 영역, 디지털   |
|            | DAC/상향변환    | 디지털 신호를 아날로그 RF 신호로 변환하여 전송       | 시간 영역, 아날로그 |
| **수신기** | ADC/하향변환    | 수신된 RF 신호를 디지털 기저대역 샘플로 변환         | 시간 영역, 디지털   |
|            | 동기화          | 심볼 타이밍 및 주파수 오프셋 추정 및 보상            | 시간 영역, 디지털   |
|            | CP 제거         | 보호 구간인 CP를 제거                                | 시간 영역, 디지털   |
|            | FFT             | 시간 영역 샘플을 주파수 영역 심볼로 변환 (복조)      | 주파수 영역, 디지털 |
|            | 채널 등화       | 채널 왜곡을 추정하고 보상                            | 주파수 영역, 디지털 |
|            | 디매핑/P/S 변환 | 복소수 심볼을 비트로 변환하고 직렬 스트림으로 재구성 | 비트/심볼           |

## 3.  다중 경로 채널 환경과 순환 전치의 역할

무선 통신 환경의 가장 큰 특징 중 하나는 신호가 단일 경로가 아닌 다수의 경로를 통해 수신기에 도달하는 다중 경로(multipath) 현상이다. 이로 인해 발생하는 신호 왜곡은 고속 데이터 통신의 성능을 저해하는 주된 요인이며, OFDM은 순환 전치(Cyclic Prefix, CP)라는 독창적인 해결책을 통해 이 문제를 극복한다.

### 3.1  다중 경로 페이딩과 심볼 간 간섭(ISI)

무선 환경에서 송신된 전파는 건물, 지형지물 등에 반사, 회절, 산란되어 여러 경로를 통해 수신기에 도달한다. 각 경로는 서로 다른 이동 거리와 위상 변화를 겪기 때문에, 수신 신호는 여러 개의 지연 시간(delay)과 감쇠(attenuation)를 가진 신호들의 합으로 나타난다.18 이러한 지연 시간의 분포를 지연 확산(delay spread)이라고 한다.

이 지연 확산으로 인해, 시간적으로 앞서 전송된 심볼의 끝부분이 시간적으로 뒤따르는 현재 심볼의 시작 부분을 침범하여 중첩되는 현상이 발생하는데, 이를 심볼 간 간섭(Inter-Symbol Interference, ISI)이라 한다.1 데이터 전송 속도가 높아질수록 각 심볼의 지속 시간은 짧아지므로, 동일한 지연 확산이라도 ISI의 영향은 상대적으로 더욱 심각해져 통신 품질을 크게 저하시킨다.10

### 3.2  순환 전치(CP)를 통한 ISI 및 ICI 방지 원리

OFDM은 각 OFDM 심볼 사이에 보호 구간(Guard Interval, GI)을 삽입하여 ISI에 대응한다. 이 보호 구간을 단순히 비워두는 대신, 해당 OFDM 심볼의 뒷부분 일부를 복사하여 채워 넣는데, 이것이 바로 순환 전치(CP)이다.1

- **ISI 방지**: CP의 길이($T_{CP}$)를 채널의 최대 지연 확산($\tau_{max}$)보다 길게 설정하면($T_{CP} > \tau_{max}$), 이전 심볼에서 비롯된 가장 긴 지연을 가진 다중 경로 신호조차도 현재 심볼의 유효 데이터 구간이 시작되기 전, 즉 CP 구간 내에서 모두 수신이 완료된다. 수신기는 이 CP 구간을 단순히 버리고 그 이후의 유효 데이터 구간만을 취하여 신호 처리를 수행한다. 이를 통해 이전 심볼로부터의 간섭이 현재 심볼에 영향을 미치는 것을 원천적으로 차단하여 ISI를 효과적으로 제거할 수 있다.20
- **ICI 방지**: 다중 경로 채널은 각 부반송파에 서로 다른 위상과 진폭 변화를 유발하여 부반송파 간의 직교성을 파괴하고 반송파 간 간섭(Inter-Carrier Interference, ICI)을 유발할 수 있다.27 CP는 아래 4.3절에서 설명할 원리를 통해, 다중 경로 채널의 영향이 주파수 영역에서 각 부반송파에 독립적인 곱셈 형태로 작용하도록 만든다. 이로써 부반송파 간의 직교성이 유지되고 ICI가 방지된다.20

### 3.3  선형 컨볼루션(Linear Convolution)의 원형 컨볼루션(Circular Convolution) 변환

송신 신호가 물리적인 채널을 통과하는 과정은 수학적으로 송신 신호와 채널 임펄스 응답(Channel Impulse Response, CIR) 간의 '선형 컨볼루션(Linear Convolution)'으로 모델링된다.28 그러나 OFDM의 핵심인 FFT 기반의 효율적인 처리를 위해서는 '원형 컨볼루션(Circular Convolution)' 관계가 필요하다.

CP는 이 두 컨볼루션 사이의 간극을 메우는 결정적인 역할을 한다. CP를 삽입함으로써 OFDM 심볼은 시간적으로 주기적인 신호와 같은 구조를 갖게 된다. 수신단에서 CP 구간을 버리고 유효 심볼 구간($T_s$)만을 관찰했을 때, 채널과의 선형 컨볼루션 결과가 수학적으로 원형 컨볼루션의 결과와 동일해지는 효과가 나타난다.28 이는 CP가 심볼의 끝과 시작을 자연스럽게 연결하여, 선형 컨볼루션 과정에서 발생하는 '꼬리' 부분이 심볼의 앞부분으로 감싸 들어오는 것처럼 보이게 만들기 때문이다.28

### 3.4  주파수 영역 등화의 단순화

이처럼 CP 덕분에 채널 효과를 원형 컨볼루션으로 간주할 수 있게 되면, 컨볼루션 정리(Convolution Theorem)를 적용할 수 있게 된다. 이 정리에 따르면, 시간 영역에서의 '원형 컨볼루션'은 주파수 영역에서의 '단순 곱셈'으로 변환된다.24

따라서 수신단에서 FFT를 수행하여 얻은 주파수 영역 신호 $Y[k]$는 송신 심볼 $X[k]$와 채널의 주파수 응답 $H[k]$의 단순한 곱셈 형태로 표현할 수 있다 ($Y[k] = H[k] \cdot X[k] + N[k]$, 여기서 $N[k]$는 잡음).

이 관계 덕분에 채널 등화 과정은 매우 간단해진다. 수신기는 각 부반송파별로 채널 응답 $H[k]$를 추정한 뒤, 수신된 신호 $Y[k]$를 $H[k]$로 나누어주기만 하면 원래의 송신 심볼 $X[k]$를 복원할 수 있다. 이러한 '단일 탭(one-tap)' 주파수 영역 등화는 시간 영역에서 복잡한 다중 탭 필터를 사용해야 하는 단일 반송파 시스템의 등화 방식에 비해 계산 복잡도를 획기적으로 낮추는 결정적인 장점이다.27

결론적으로, CP는 단순히 ISI를 막는 보호 구간을 넘어, 아날로그 물리 세계의 현상(선형 컨볼루션)을 디지털 신호 처리(FFT 기반 등화)가 가장 효율적으로 다룰 수 있는 수학적 형태(원형 컨볼루션)로 변환해주는 '인터페이스' 역할을 수행한다. CP가 없었다면 OFDM은 FFT 기반의 저복잡도 등화라는 가장 큰 실용적 장점을 잃게 되었을 것이다.

`표 3`은 OFDM 기술의 장점과 단점을 종합적으로 요약한 것이다.

**표 3: OFDM의 장점 및 단점 요약**

| 구분     | 항목                          | 설명                                                         |
| -------- | ----------------------------- | ------------------------------------------------------------ |
| **장점** | **다중 경로 페이딩 강인성**   | 광대역 채널을 다수의 협대역 평탄 페이딩 채널로 변환하여 주파수 선택적 페이딩에 강함.4 |
|          | **높은 주파수 효율**          | 직교성을 이용해 보호 대역 없이 부반송파를 중첩시켜 스펙트럼을 효율적으로 사용.4 |
|          | **간단한 채널 등화**          | CP를 이용하여 주파수 영역에서 간단한 단일 탭 등화가 가능하여 수신기 복잡도 감소.7 |
|          | **SFN 구현 용이**             | 여러 송신국이 동일 주파수를 사용해도 신호가 보강 간섭으로 작용하여 단일 주파수 망 구성에 유리.1 |
|          | **협대역 간섭에 대한 강인성** | 특정 주파수의 간섭이 일부 부반송파에만 영향을 미치므로 전체 데이터 손실 방지.4 |
| **단점** | **높은 PAPR**                 | 다수 부반송파의 합으로 인해 높은 피크 전력이 발생하여 전력 증폭기의 효율 저하 및 비선형 왜곡 유발.7 |
|          | **동기화에 대한 민감성**      | 주파수 및 시간 오차에 매우 민감하여 직교성이 쉽게 깨지고 ICI가 발생. 정밀한 동기화 회로 필요.7 |
|          | **CP로 인한 효율 손실**       | 데이터를 전송하지 않는 CP 구간으로 인해 전송률 및 전력 효율 감소.7 |
|          | **도플러 효과에 대한 취약성** | 고속 이동 환경에서 발생하는 도플러 편이가 직교성을 파괴하여 성능을 급격히 저하.7 |

## 4.  OFDM의 주요 기술적 과제와 해결 방안

OFDM은 수많은 장점에도 불구하고, 그 근본적인 구조에서 비롯되는 몇 가지 중요한 기술적 과제를 안고 있다. 그중 가장 대표적인 것이 높은 최대 전력 대 평균 전력 비(PAPR) 문제와 동기화에 대한 민감성이다.

### 4.1  최대 전력 대 평균 전력 비(PAPR) 문제

#### 4.1.1  PAPR 발생 원인 및 시스템에 미치는 영향

OFDM 신호는 다수의 독립적인 부반송파 신호들을 시간 영역에서 합산하여 생성된다. 이 과정에서 다수의 부반송파 신호들이 특정 시간 지점에서 우연히 모두 같은 위상으로 더해지는 보강 간섭이 발생할 수 있다. 이 경우, 순간적인 신호의 진폭(피크 전력)이 평균 전력에 비해 매우 커지는 현상이 나타나는데, 이를 높은 PAPR(Peak-to-Average Power Ratio) 문제라고 한다.31

높은 PAPR은 송신 시스템에 심각한 문제를 야기한다. 송신단의 전력 증폭기(Power Amplifier, PA)는 특정 입력 전력 범위 내에서만 신호를 선형적으로 증폭할 수 있다. PAPR이 높은 신호가 이 선형 동작 영역을 벗어나는 피크 값을 가지면, PA는 포화 상태에 이르러 신호를 비선형적으로 왜곡시킨다. 이 비선형 왜곡은 두 가지 문제를 일으킨다. 첫째, 대역 내 왜곡(in-band distortion)을 발생시켜 수신단에서의 복조 성능, 즉 비트 오류율(BER)을 저하시킨다. 둘째, 대역 외 방사(out-of-band radiation)를 유발하여 인접 채널에 간섭을 일으킨다.31

이러한 왜곡을 피하기 위해서는 PA가 항상 최대 피크 전력까지 선형적으로 동작하도록 설계해야 한다. 이는 PA의 동작점을 평균 전력보다 훨씬 높은 곳에 두어야 함을 의미하며, 결과적으로 PA의 전력 효율이 크게 감소하게 된다.7 또한, 높은 선형성을 요구하는 PA는 가격이 비싸 이동 단말기의 비용을 증가시키는 요인이 된다.32

#### 4.1.2  PAPR 저감 기법 비교 분석

높은 PAPR 문제를 해결하기 위해 지난 수십 년간 다양한 기법들이 연구되어 왔다. 이들은 크게 신호 왜곡 기법, 신호 스크램블링 기법, 코딩 기법 등으로 분류할 수 있다.

- **신호 왜곡(Signal Distortion) 기법:** 신호 자체를 변형시켜 피크 값을 직접적으로 줄이는 방식이다.
  - **클리핑(Clipping):** 가장 직관적이고 간단한 방법으로, 신호의 진폭이 미리 정해진 임계값을 초과하면 강제로 잘라내는 방식이다. 구현은 간단하지만, 신호를 의도적으로 왜곡시키므로 BER 성능 저하를 감수해야 한다.33 클리핑 이후 필터링을 적용하여 클리핑으로 인해 발생하는 대역 외 방사를 일부 줄일 수 있다.34
  - **컴팬딩(Companding):** 신호의 동적 범위를 비선형 함수를 이용해 압축(compressing)하여 전송하고, 수신기에서 역함수를 이용해 다시 확장(expanding)하는 기법이다. 진폭이 큰 신호는 압축하고 작은 신호는 확장하여 전체적인 PAPR을 낮춘다.33
- **신호 스크램블링(Signal Scrambling) 기법:** 원본 정보를 왜곡하지 않으면서 신호의 통계적 특성을 변화시켜 높은 피크가 발생할 확률 자체를 낮추는 방식이다.
  - **PTS (Partial Transmit Sequence):** 전체 부반송파를 여러 개의 부블록으로 나눈다. 각 부블록에 서로 다른 위상 회전 계수를 적용하여 여러 조합의 후보 신호를 생성하고, 그중 PAPR이 가장 낮은 신호를 선택하여 전송한다. PAPR 저감 성능은 우수하지만, 최적의 위상 조합을 찾는 과정의 계산 복잡도가 매우 높고, 선택된 위상 정보를 수신기에 알려주기 위한 부가 정보(Side Information, SI) 전송이 필요하여 데이터 전송 효율이 감소하는 단점이 있다.32
  - **SLM (Selected Mapping):** 원본 데이터 블록에 여러 개의 서로 다른 위상 시퀀스를 곱하여 다수의 독립적인 후보 신호를 생성하고, 그중 PAPR이 가장 낮은 신호를 전송한다. PTS와 마찬가지로 우수한 성능을 보이지만, 다수의 IFFT 연산으로 인한 계산 복잡도와 SI 전송의 부담이 존재한다.32
- **코딩(Coding) 기법:** 오류 정정 부호 설계 시, PAPR이 낮은 특성을 갖는 코드워드만을 사용하도록 제한하는 방식이다. 신호 왜곡이나 추가적인 계산 없이 PAPR을 줄일 수 있지만, 사용 가능한 코드워드가 제한되므로 코드율이 감소하여 데이터 전송률이 저하되는 단점이 있다.31

#### 4.1.3  최신 연구 동향: 머신러닝(ML) 기반 PAPR 저감

최근에는 기존 기법들의 한계를 극복하기 위해 인공지능, 특히 머신러닝(ML) 및 딥러닝(DL)을 PAPR 저감에 활용하는 연구가 활발히 진행되고 있다.35 오토인코더(Autoencoder)와 같은 심층 신경망(DNN) 모델을 사용하여 PAPR 저감과 신호 복원을 하나의 종단 간(end-to-end) 최적화 문제로 모델링하는 접근 방식이 대표적이다. 이 방식은 데이터로부터 PAPR 발생의 복잡한 비선형 패턴을 직접 학습하여, 기존 기법들보다 더 나은 성능과 복잡도 간의 균형점을 찾을 잠재력을 가지고 있다. 하지만 ML 기반 기법들 역시 모델의 계산 복잡성, 방대한 훈련 데이터 요구, 학습된 모델의 해석 가능성 부족 등 해결해야 할 과제를 안고 있다.35

`표 4`는 주요 PAPR 저감 기법들의 특성을 비교한다.

**표 4: PAPR 저감 기법 비교 분석**

| 기법        | PAPR 저감 성능 | 계산 복잡도      | 전력 증가 | 대역폭 효율 손실        | 신호 왜곡                  |
| ----------- | -------------- | ---------------- | --------- | ----------------------- | -------------------------- |
| **클리핑**  | 중간           | 낮음             | 없음      | 없음                    | 발생 (In-band/Out-of-band) |
| **PTS**     | 높음           | 매우 높음        | 없음      | 발생 (Side Information) | 없음                       |
| **SLM**     | 높음           | 높음             | 없음      | 발생 (Side Information) | 없음                       |
| **코딩**    | 낮음~중간      | 낮음             | 없음      | 발생 (Code Rate 감소)   | 없음                       |
| **ML 기반** | 높음 (잠재력)  | 높음 (훈련/추론) | 없음      | 없음                    | 최소화 가능 (학습)         |

### 4.2  동기화의 민감성

OFDM의 근간인 직교성은 다수의 부반송파를 중첩시키는 구조에서 비롯된 필연적인 결과로, PAPR 문제와 동기화 민감성이라는 두 가지 주요 과제를 야기한다. PAPR이 통계적 문제라면, 동기화는 결정론적(deterministic) 문제이다.

- **오프셋의 영향:** 앞서 2.4절에서 논의했듯이, 반송파 주파수 오프셋(CFO)은 부반송파 간 직교성을 직접적으로 파괴하여 심각한 ICI를 유발한다.7 샘플링 시간 오프셋(Sampling Time Offset, STO)은 FFT 윈도우의 위치를 부정확하게 만들어 각 부반송파에 위상 회전을 일으키고, 심한 경우 CP의 보호 효과를 무력화시켜 ISI를 다시 발생시킬 수 있다.29
- **동기화 기법:** 수신기는 이러한 오차를 보상하기 위해 정교한 동기화 절차를 수행한다. 일반적으로 전송 프레임의 시작 부분에 있는 프리앰블(preamble)이나 널리 알려진 동기 심볼과 같은 신호를 이용한다. 예를 들어, CP의 반복적인 특성을 활용한 자기 상관(auto-correlation) 기법으로 심볼의 정확한 시작점을 찾고(시간 동기화), 프리앰블 내의 반복 패턴을 이용하여 주파수 오차를 추정하고 보상한다(주파수 동기화).1 동기화가 획득된 이후에도, 데이터 심볼 사이에 주기적으로 삽입되는 파일럿 심볼을 지속적으로 추적하여 미세한 오차를 보상하고 동기를 유지한다.21

## 5.  OFDM 기반 다중 접속 및 차세대 파형 기술

OFDM은 그 자체로 변조 방식이지만, 시간과 주파수라는 2차원 자원을 유연하게 활용할 수 있는 특성 덕분에 다수의 사용자가 자원을 공유하는 다중 접속(Multiple Access) 기술의 기반으로 확장되었다. 또한, 기술의 발전에 따라 OFDM의 한계를 극복하기 위한 차세대 파형 기술들이 등장하고 있다.

### 5.1  OFDMA와 SC-FDMA

- **OFDMA (Orthogonal Frequency Division Multiple Access):** OFDMA는 OFDM의 전체 부반송파 집합을 더 작은 단위의 그룹으로 나누어, 각 그룹을 서로 다른 사용자에게 할당하는 다중 접속 방식이다.13 이는 시간과 주파수 자원을 2차원 격자(grid)로 보고, 특정 사용자에게 특정 시간 슬롯의 특정 부반송파 그룹, 즉 자원 블록(Resource Block, RB)을 할당하는 것과 같다.38 OFDMA의 가장 큰 장점은 자원 할당의 유연성이 매우 높다는 점이다. 각 사용자의 채널 상태가 좋은 주파수 대역이나 요구 데이터율에 맞춰 동적으로 자원을 할당함으로써, 시스템 전체의 주파수 효율과 용량을 극대화할 수 있다(다중 사용자 다이버시티 효과).13 이러한 장점 때문에 LTE와 5G NR의 하향링크(downlink, 기지국에서 단말기로의 전송)에 표준으로 채택되었다.37
- **SC-FDMA (Single-Carrier FDMA):** DFT-spread-OFDM이라고도 불리는 SC-FDMA는 기본적으로 OFDMA 송신기 구조의 심볼 매핑단과 IFFT단 사이에 DFT 연산 블록을 추가한 형태이다.40 이 추가적인 DFT 확산 과정은 여러 부반송파에 실리는 신호의 특성을 단일 반송파 신호의 특성과 유사하게 만들어준다. SC-FDMA의 결정적인 장점은 OFDMA에 비해 PAPR이 현저히 낮다는 것이다.40 특정 시뮬레이션 조건에서 약 3.1 dB의 PAPR 개선 효과가 보고된 바 있다.42 낮은 PAPR은 전력 효율이 매우 중요하고 고가의 선형적인 PA를 사용하기 어려운 이동 단말기에 큰 이점을 제공한다. 이러한 이유로 SC-FDMA는 LTE 및 5G NR의 상향링크(uplink, 단말기에서 기지국으로의 전송) 표준으로 채택되었다.37 반면, DFT 확산으로 인해 각 데이터 심볼이 여러 부반송파에 걸쳐 전송되므로, OFDMA만큼 세밀하고 유연한 주파수 자원 할당은 어렵다는 단점이 있다.43

`표 5`는 OFDMA와 SC-FDMA의 주요 특징을 비교한다.

**표 5: OFDMA와 SC-FDMA 비교**

| 특징                 | OFDMA                                | SC-FDMA (DFT-s-OFDM)           |
| -------------------- | ------------------------------------ | ------------------------------ |
| **기본 원리**        | 부반송파 그룹을 사용자별로 할당      | DFT 확산 후 부반송파 그룹 할당 |
| **PAPR 특성**        | 높음                                 | 낮음                           |
| **자원 할당 유연성** | 매우 높음 (부반송파 단위)            | 제한적 (연속된 부반송파 할당)  |
| **주 사용 링크**     | 하향링크 (Downlink)                  | 상향링크 (Uplink)              |
| **수신기 복잡도**    | 상대적으로 낮음                      | 상대적으로 높음 (IDFT 필요)    |
| **주요 장점**        | 높은 스펙트럼 효율, 유연한 자원 할당 | 낮은 PAPR, 높은 전력 효율      |
| **주요 단점**        | 높은 PAPR                            | 낮은 자원 할당 유연성          |

### 5.2  MIMO-OFDM

MIMO(Multiple-Input Multiple-Output)는 다수의 송신 안테나와 수신 안테나를 사용하여 공간을 또 다른 통신 자원으로 활용하는 기술이다. MIMO 기술은 OFDM과 결합하여 MIMO-OFDM이라는 강력한 시너지를 창출한다. 이 결합은 두 가지 주요 이득을 제공한다. 첫째, 공간 다이버시티(spatial diversity)를 통해 동일한 신호를 여러 경로로 전송하여 다중 경로 페이딩에 대한 강인성을 높이고 신호의 신뢰도를 향상시킨다. 둘째, 공간 다중화(spatial multiplexing)를 통해 서로 다른 데이터 스트림을 동일한 시간/주파수 자원을 사용하여 동시에 전송함으로써 데이터 전송률을 안테나 수에 비례하여 획기적으로 증가시킬 수 있다.21

OFDM은 각 부반송파가 평탄 페이딩을 겪도록 채널을 변환해주기 때문에, 각 부반송파에 대해 간단한 행렬 연산으로 MIMO 처리가 가능하여 두 기술의 결합이 매우 용이하다.5 STBC(Space-Time Block Code)-OFDM, SFBC(Space-Frequency Block Code)-OFDM 등 다양한 시공간 또는 시공간-주파수 부호화 기법이 MIMO-OFDM 시스템에 적용되어 성능을 극대화한다.21

### 5.3  주요 통신 표준 적용 사례

OFDM 기술의 발전사는 '경직성'에서 '유연성'으로, 그리고 '단일 최적화'에서 '다중 시나리오 적응'으로 나아가는 과정으로 요약될 수 있다. 초기 OFDM 표준들이 고정된 파라미터를 사용한 반면, 최신 표준들은 다양한 서비스와 환경에 동적으로 적응할 수 있는 유연성과 확장성을 핵심 설계 철학으로 삼고 있다.

- **WLAN (IEEE 802.11ax, Wi-Fi 6):** 이전 표준인 802.11ac 대비 FFT 크기를 4배 늘리고(최대 2048), 부반송파 간격을 1/4로 줄여(78.125 kHz) OFDM 심볼 길이를 4배(12.8 µs)로 늘렸다. 이는 다중 경로 지연에 대한 강인성을 크게 향상시켜 실외나 혼잡한 환경에서의 성능을 개선한다.45 또한, 핵심 기술로 OFDMA를 도입하여 전체 채널 대역폭을 더 작은 자원 단위(Resource Unit, RU)로 분할하고, 이를 다수의 사용자에게 동시에 할당함으로써 조밀한 사용자 환경에서의 전반적인 효율을 극대화했다.45

- **5G NR (New Radio):** 5G는 초고속(eMBB), 초저지연(URLLC), 초연결(mMTC)이라는 서로 상이한 요구사항을 가진 서비스들을 단일 시스템에서 지원하기 위해 '가변 뉴머롤로지(Flexible Numerology)' 개념을 도입했다.48 이는 부반송파 간격을 

  $15 \text{ kHz} \times 2^\mu$ ($\mu=0,1,2,3,4$)로 가변적으로 설정할 수 있게 한 것이다.48 부반송파 간격이 넓어지면 심볼 길이는 짧아져 저지연 통신(URLLC)에 유리하고, 간격이 좁아지면 심볼 길이가 길어져 보호 구간의 상대적 오버헤드가 줄고 넓은 커버리지(mMTC) 확보에 유리하다.48

- **DVB-T2 (Digital Video Broadcasting):** 디지털 지상파 방송 표준인 DVB-T2는 매우 긴 FFT 크기(최대 32k)와 다양한 CP 비율(1/128 ~ 1/4)을 지원한다. 이는 방송 환경의 특징인 단일 주파수 망(Single Frequency Network, SFN)에서 발생하는 매우 긴 지연 확산에 효과적으로 대응하기 위함이다.17 또한, 물리 계층 파이프(Physical Layer Pipe, PLP) 개념을 도입하여, 하나의 RF 채널 내에서 각기 다른 변조 및 코딩 방식(즉, 다른 수준의 견고성)을 갖는 여러 서비스(예: HD 방송과 SD 방송)를 동시에 전송할 수 있는 유연성을 제공한다.17

`표 6`은 주요 통신 표준별 OFDM 파라미터를 비교한다.

**표 6: 주요 통신 표준별 OFDM 파라미터 비교**

| 파라미터                | IEEE 802.11ax (Wi-Fi 6) | 5G NR                                | DVB-T2                                      |
| ----------------------- | ----------------------- | ------------------------------------ | ------------------------------------------- |
| **채널 대역폭**         | 20, 40, 80, 160 MHz     | 5~100 MHz (FR1), 50~400 MHz (FR2)    | 1.7, 5, 6, 7, 8, 10 MHz                     |
| **FFT 크기**            | 256, 512, 1024, 2048    | 가변 (뉴머롤로지에 따라 다름)        | 1k, 2k, 4k, 8k, 16k, 32k                    |
| **부반송파 간격**       | 78.125 kHz              | 15, 30, 60, 120, 240 kHz             | 대역폭/FFT 크기에 따라 결정                 |
| **심볼 길이 (CP 제외)** | 12.8 µs                 | 66.7 µs (@15kHz) ~ 4.17 µs (@240kHz) | ~4 ms (@32k, 8MHz)                          |
| **CP 길이**             | 0.8, 1.6, 3.2 µs        | Normal / Extended                    | 1/128, 1/32, 1/16,..., 1/4 (심볼 길이 비율) |
| **최대 변조 방식**      | 1024-QAM                | 256-QAM (UL/DL), 1024-QAM (DL 옵션)  | 256-QAM                                     |
| **다중 접속 방식**      | OFDMA                   | OFDMA(DL), SC-FDMA(UL)               | TDM (PLP 기반)                              |

### 5.4  차세대 파형(Waveform) 기술 동향

OFDM은 여전히 강력한 기술이지만, 특히 고속 이동 환경에서의 도플러 효과에 대한 취약성 등 근본적인 한계를 가지고 있다.14 이로 인해 6G와 같은 차세대 통신 시스템을 위한 새로운 파형 기술들이 활발히 연구되고 있다.

- **FBMC (Filter Bank Multi-Carrier):** 각 부반송파마다 매우 정교한 필터를 적용하여 대역 외 방사(OOB emission)를 획기적으로 줄인 기술이다. 이 덕분에 CP가 필요 없어 스펙트럼 효율이 높고, 비동기적인 다중 사용자 환경에 강하다. 하지만 필터 구현으로 인한 계산 복잡도와 지연 시간이 크고, MIMO 시스템과의 결합이 복잡하다는 단점이 있다.51
- **6G 후보 기술 (OTFS, AFDM):** 6G 시대의 핵심 요구사항 중 하나인 초고속 이동성 환경에 대응하기 위한 기술들이다.
  - **OTFS (Orthogonal Time Frequency Space):** 전통적인 시간-주파수 영역 대신, 채널의 변화가 더 느리게 나타나는 지연-도플러(Delay-Doppler) 영역에서 데이터를 직접 변조한다. 이를 통해 높은 이동성으로 인한 심각한 도플러 효과에 매우 강인한 특성을 보인다.14
  - **AFDM (Affine Frequency Division Multiplexing):** 이산 어파인 푸리에 변환(DAFT)이라는 일반화된 푸리에 변환을 기반으로 한다. OTFS와 유사한 수준의 도플러 강인성을 가지면서도, 기존 OFDM 시스템 구조와의 호환성이 높고 구현 복잡도가 낮다는 장점이 있어 유력한 6G 후보 기술로 부상하고 있다.14

## 6.  결론

### 7.1. OFDM 기술의 핵심적 가치와 한계 요약

OFDM은 지난 수십 년간 디지털 통신 기술의 발전을 견인해 온 핵심 동력이었다. 그 가치는 다음 두 가지로 요약할 수 있다. 첫째, '직교성'이라는 수학적 원리를 'FFT'라는 효율적인 디지털 알고리즘으로 구현하여 높은 주파수 효율을 달성했다. 둘째, '순환 전치'라는 독창적인 아이디어를 통해 다중 경로 채널이라는 무선 통신의 가장 큰 난제를 극복하고, 간단한 등화만으로 신뢰성 있는 고속 통신을 가능하게 했다.

그러나 이 기술은 명확한 한계 또한 내포하고 있다. 다수의 부반송파를 합산하는 구조에서 비롯되는 높은 PAPR 문제는 송신기의 전력 효율을 저하시키는 고질적인 단점이다. 또한, 직교성을 유지하기 위한 전제 조건인 완벽한 동기화에 대한 극심한 민감성은 시스템의 복잡도를 증가시킨다. 특히, 고속 이동 환경에서 발생하는 도플러 효과에 대한 취약성은 6G와 같은 차세대 통신 시스템이 극복해야 할 중요한 과제로 남아있다.

### 6.1  미래 무선 통신 시스템에서의 OFDM의 역할과 발전 방향 전망

이러한 한계에도 불구하고, OFDM은 그 견고함, 효율성, 그리고 무엇보다 기존 인프라와의 높은 호환성 덕분에 당분간 6G를 포함한 미래 시스템에서도 중요한 기반 기술로서의 역할을 계속할 것이다. 다만, 그 형태는 현재와는 다를 수 있으며, 발전 방향은 크게 두 가지로 예측된다.

첫째는 **'OFDM의 진화'**이다. 기존 OFDM 프레임워크에 윈도윙(Windowing)이나 필터링(Filtering) 같은 기술을 융합하여 스펙트럼 특성을 개선한 f-OFDM과 같은 변종 기술들이 특정 시나리오에 적용될 수 있다. 또한, 인공지능/머신러닝 기술을 접목하여 PAPR 저감, 동기화, 자원 할당과 같은 고질적인 문제들을 더욱 지능적이고 적응적으로 해결하려는 노력이 가속화될 것이다.

둘째는 **'새로운 파형으로의 전환'**이다. 지상과 위성을 통합하고, 차량, 드론 등 초고속 이동체를 지원하며, 통신과 감지 기능이 통합(ISAC)되는 6G의 새로운 요구사항을 만족시키기 위해 OFDM의 한계를 근본적으로 뛰어넘는 새로운 파형 기술들이 필요하다. OTFS나 AFDM과 같은 기술들은 특히 높은 이동성 환경에서 OFDM을 대체하거나 공존하며 특정 서비스 계층을 담당할 가능성이 높다.14

궁극적으로 미래 통신 시스템은 하나의 단일 파형이 모든 것을 지배하는 시대에서 벗어나, 다양한 서비스와 시나리오에 최적화된 여러 파형들을 유연하게 선택하고 조합하여 사용하는 '파형 툴박스(Waveform Toolbox)' 접근 방식을 취할 것으로 전망된다. 이 툴박스 안에서 OFDM은 여전히 가장 기본적이고 중요한 도구 중 하나로 그 자리를 지킬 것이며, 동시에 새로운 도구들과 함께 진화하며 통신 기술의 미래를 열어갈 것이다.

#### **참고 자료**

1. 지식덤프, 8월 4, 2025에 액세스, http://www.jidum.com/jidums/view.do?jidumId=502
2. Intuitive Guide to Principles of Communications www.complextoreal.com Orthogonal Frequency Division Multiplexing (OFDM) Modulat, 8월 4, 2025에 액세스, https://complextoreal.com/wp-content/uploads/2013/01/ofdm2.pdf
3. OFDM(Orthogonal Frequency Division Multiplexing)이란?? - RF열무의 라이프 스터디 블로그, 8월 4, 2025에 액세스, https://rf-yeolmu.tistory.com/35
4. OFDM (Orthogonal Frequency Division Multiplexing) - CableFree, 8월 4, 2025에 액세스, https://www.cablefree.net/wirelesstechnology/ofdm-introduction/
5. Orthogonal Frequency-Division Multiplexing (OFDM) Explained - MATLAB & Simulink, 8월 4, 2025에 액세스, https://www.mathworks.com/discovery/ofdm.html
6. 2.3GHz대역 주파수 활용을 위한 표준 기술소개, 8월 4, 2025에 액세스, https://www.tta.or.kr/data/androReport/ttaJnal/JTGHBY_2002_s84_86.pdf
7. Orthogonal frequency-division multiplexing - Wikipedia, 8월 4, 2025에 액세스, https://en.wikipedia.org/wiki/Orthogonal_frequency-division_multiplexing
8. OFDM(Orthogonal FDM : 직교 주파수 분할 다중) - CA 방송일반 - 『catv』기초상식, 8월 4, 2025에 액세스, https://cafe.daum.net/catvcableguy/4lpG/291?svc=cafeapi
9. [디지털통신] OFDM: (1) OFDM 기초 - 이것저것 테크블로그 - 티스토리, 8월 4, 2025에 액세스, https://ai-com.tistory.com/entry/OFDM-Orthogonal-Frequency-Division-Multiplexing
10. Introduction to OFDM, 8월 4, 2025에 액세스, https://comm.eelabs.technion.ac.il/wp-content/uploads/sites/12/2020/11/Booklet_Exp72-OFDM-Tutorial.pdf
11. Concepts of Orthogonal Frequency Division Multiplexing (OFDM) and 802.11 WLAN, 8월 4, 2025에 액세스, https://helpfiles.keysight.com/csg/89600B/Webhelp/Subsystems/wlan-ofdm/content/ofdm_basicprinciplesoverview.htm
12. Orthogonal Frequency Division Multiplexing - Devopedia, 8월 4, 2025에 액세스, https://devopedia.org/orthogonal-frequency-division-multiplexing
13. OFDMA - 특허 기술 동향 - ( - 미국, 8월 4, 2025에 액세스, http://www.cinelove.net/newtech/ofdm.htm
14. Overview and Performance Analysis of Various Waveforms in ... - arXiv, 8월 4, 2025에 액세스, https://arxiv.org/abs/2302.14224
15. OFDM transmitter and receiver block diagram explanation ..., 8월 4, 2025에 액세스, http://telcosought.com/4g-ran/ofdm-transmitter-ofdm-receiver/
16. Orthogonal Frequency Division Multiplexing and its Applications - ResearchGate, 8월 4, 2025에 액세스, https://www.researchgate.net/profile/Ankit-Chadha-5/publication/235943582_Orthogonal_Frequency_Division_Multiplexing_and_its_Applications/links/00b7d51496adc5c932000000/Orthogonal-Frequency-Division-Multiplexing-and-its-Applications.pdf
17. DVB-T2 - Wikipedia, 8월 4, 2025에 액세스, https://en.wikipedia.org/wiki/DVB-T2
18. OFDM Uncovered Part 1: The Architecture - EE Times, 8월 4, 2025에 액세스, https://www.eetimes.com/ofdm-uncovered-part-1-the-architecture/
19. OFDM (OFDMA) 원리 및 특징 장단점 - 오년 - 티스토리, 8월 4, 2025에 액세스, https://ttistoryy.tistory.com/15
20. Cyclic Prefix in OFDM: Definition and Role in Wireless Communication, 8월 4, 2025에 액세스, https://www.rfwireless-world.com/terminology/cyclic-prefix-ofdm-wireless-communication
21. [와이브로 大해부] (2) 「다중접속 방식」면밀히 살펴보기 - 지디넷코리아, 8월 4, 2025에 액세스, https://zdnet.co.kr/view/?no=00000039136689
22. ORTHOGONAL FREQUENCY DIVISION MULTIPLEXING FOR WIRELESS CHANNELS, 8월 4, 2025에 액세스, https://www.i3s.unice.fr/~deneire/mobile/ofdm_tutorial.pdf
23. 다중 모드를 사용하는 OFDM-CDIM 시스템 설계와 성능 평가 - 한국전자파학회논문지, 8월 4, 2025에 액세스, https://www.jkiees.org/archive/view_article?pid=jkiees-29-7-515
24. Basic OFDM modulation and demodulation - DSPIllustrations.com, 8월 4, 2025에 액세스, [https://dspillustrations.com/pages/pages/courses/course_OFDM/html/03%20-%20Basic%20OFDM%20modulation%20and%20demodulation.html](https://dspillustrations.com/pages/pages/courses/course_OFDM/html/03 - Basic OFDM modulation and demodulation.html)
25. OFDM transmission step-by-step - labAlive experiment, 8월 4, 2025에 액세스, https://www.etti.unibw.de/labalive/experiment/ofdmstepbystep/
26. Mathematical description of OFDM, 8월 4, 2025에 액세스, http://www.wirelesscommunication.nl/reference/chaptr05/ofdm/ofdmmath.htm
27. (PDF) Effects of cyclic prefix on OFDM system - ResearchGate, 8월 4, 2025에 액세스, https://www.researchgate.net/publication/220902525_Effects_of_cyclic_prefix_on_OFDM_system
28. The Cyclic Prefix (CP) in OFDM - DSPIllustrations.com, 8월 4, 2025에 액세스, https://dspillustrations.com/pages/posts/misc/the-cyclic-prefix-cp-in-ofdm.html
29. Advantages and Disadvantages of OFDM, 8월 4, 2025에 액세스, https://sna.csie.ndhu.edu.tw/~cnyang/MCCDMA/tsld021.htm
30. OFDM: Advantages and Disadvantages of Orthogonal Frequency Division Multiplexing, 8월 4, 2025에 액세스, https://www.rfwireless-world.com/terminology/ofdm-advantages-disadvantages
31. Review on PAPR Reduction and Improvement of OFDM System Performance Using Artificial Intelligence Machine Learning Algorithm, 8월 4, 2025에 액세스, https://ijsret.com/wp-content/uploads/2025/01/IJSRET_V11_issue1_102.pdf
32. Low-complexity peak-to-average power ratio reduction method for orthogonal frequency- division multiplexing communications, 8월 4, 2025에 액세스, https://web.njit.edu/~akansu/PAPERS/Wang-Akansu-OFDM-PAPR-Reduction-IET-Nov2015.pdf
33. A Comprehensive Study of PAPR Reduction Techniques for Deep Joint Source Channel Coding in OFDM Systems - arXiv, 8월 4, 2025에 액세스, https://arxiv.org/pdf/2309.11803
34. A Review: Analysis of PAPR Reduction Techniques of OFDM System - Crimson Publishers, 8월 4, 2025에 액세스, https://crimsonpublishers.com/rdms/pdf/RDMS.000686.pdf
35. A Survey of PAPR Techniques Based on Machine Learning - MDPI, 8월 4, 2025에 액세스, https://www.mdpi.com/1424-8220/24/6/1918
36. Cyclic Prefix (CP) in OFDM: Advantages and Disadvantages | RF Wireless World, 8월 4, 2025에 액세스, https://www.rfwireless-world.com/terminology/cyclic-prefix-advantages-disadvantages
37. FAQ | ShareTechnote, 8월 4, 2025에 액세스, https://www.sharetechnote.com/html/db/html/FAQ_4G_PhyProcessing_OFDMA_SCFDMA.html
38. LTE, OFDM, OFDMA - Zeung-il Kim - Medium, 8월 4, 2025에 액세스, https://endland.medium.com/lte-ofdm-ofdma-e89d7d722785
39. 5G Air Interface – Part II LTE and NR Physical layer, 8월 4, 2025에 액세스, https://www.dspcsp.com/tau/5G/05-air-if-2.pdf
40. Single-carrier FDMA - Wikipedia, 8월 4, 2025에 액세스, https://en.wikipedia.org/wiki/Single-carrier_FDMA
41. The Application of OFDMA and SC-FDMA in LTE - LNTwww, 8월 4, 2025에 액세스, https://en.lntwww.de/Mobile_Communications/The_Application_of_OFDMA_and_SC-FDMA_in_LTE
42. SC-FDMA vs. OFDM Modulation - MathWorks, 8월 4, 2025에 액세스, https://de.mathworks.com/help/comm/ug/scfdma-vs-ofdm.html
43. (PDF) OFDMA vs. SC-FDMA: performance comparison in local area imt-a scenarios, 8월 4, 2025에 액세스, https://www.researchgate.net/publication/224341122_OFDMA_vs_SC-FDMA_performance_comparison_in_local_area_imt-a_scenarios
44. Difference Between CDMA and OFDM - GeeksforGeeks, 8월 4, 2025에 액세스, https://www.geeksforgeeks.org/electronics-engineering/difference-between-cdma-and-ofdm/
45. Wi-Fi 6 - Wikipedia, 8월 4, 2025에 액세스, https://en.wikipedia.org/wiki/Wi-Fi_6
46. Introduction to 802.11ax High-Efficiency Wireless - NI, 8월 4, 2025에 액세스, https://www.ni.com/en/solutions/semiconductor/wireless-connectivity-test/introduction-to-802-11ax-high-efficiency-wireless.html
47. Performance and simulation analysis of 802.11ax OFDMA in contention-driven scenarios, 8월 4, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC11888924/
48. Designing for the future: the 5G NR physical layer - Ericsson, 8월 4, 2025에 액세스, https://www.ericsson.com/en/reports-and-papers/ericsson-technology-review/articles/designing-for-the-future-the-5g-nr-physical-layer
49. Understanding and Modeling the 5G NR Physical ... - MATLAB EXPO, 8월 4, 2025에 액세스, https://www.matlabexpo.com/content/dam/mathworks/mathworks-dot-com/images/events/matlabexpo/us/2019/understanding-and-modeling-the-5g-nr-physical-layer.pdf
50. DVB-T2 - ENENSYS Technologies, 8월 4, 2025에 액세스, https://www.enensys.com/technologies/dvb-t2/
51. FBMC vs OFDM Waveform Contenders for 5G Wireless Communication System, 8월 4, 2025에 액세스, https://www.scirp.org/journal/paperinformation?paperid=79772
52. OFDM vs FBMC: A Detailed Comparison - RF Wireless World, 8월 4, 2025에 액세스, https://www.rfwireless-world.com/terminology/ofdm-vs-fbmc
53. FBMC vs. OFDM Modulation - MATLAB & Simulink - MathWorks, 8월 4, 2025에 액세스, https://www.mathworks.com/help/comm/ug/fbmc-vs-ofdm-modulation.html
54. Affine Frequency Division Multiplexing (AFDM) for 6G: Properties, Features, and Challenges, 8월 4, 2025에 액세스, https://arxiv.org/html/2507.21704v1

