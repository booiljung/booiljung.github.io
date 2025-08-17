# 직접 시퀀스 대역 확산(DSSS) 기술

## 1. 서론: DSSS의 개념과 역사

### 0.1  대역 확산 통신의 정의와 DSSS의 기본 원리

대역 확산(Spread Spectrum) 통신은 전송하고자 하는 정보 신호가 점유하는 주파수 대역폭보다 훨씬 더 넓은 대역폭에 걸쳐 신호의 에너지를 의도적으로 분산시켜 전송하는 기술을 총칭한다.1 이 과정의 핵심은 신호의 총에너지는 보존하되, 단위 주파수당 에너지, 즉 전력 스펙트럼 밀도(Power Spectral Density, PSD)를 현저히 낮추는 데 있다. 결과적으로, 확산된 신호는 외부 관찰자에게 광대역 잡음(wideband noise)처럼 보이게 되어 신호의 존재 자체를 인지하기 어렵게 만든다.3

직접 시퀀스 대역 확산(Direct-Sequence Spread Spectrum, DSSS)은 이러한 대역 확산을 구현하는 가장 대표적이고 널리 사용되는 방식이다. DSSS의 기본 원리는 정보 데이터 신호에 그보다 훨씬 빠른 속도를 갖는 특별한 부호, 즉 '확산 부호(Spreading Code)' 또는 '칩 시퀀스(Chipping Sequence)'를 직접 곱하는 것이다.1 정보 데이터의 각 비트(bit)는 확산 부호의 전체 시퀀스와 곱해지며, 이 확산 부호의 각 단위를 칩(chip)이라고 칭한다. 정보 비트의 속도(bit rate)에 비해 칩의 속도(chip rate)가 월등히 높기 때문에, 원래의 좁은 대역폭을 가졌던 정보 신호는 확산 부호가 점유하는 넓은 대역폭으로 자연스럽게 확산된다.8

송신단에서 이 과정을 통해 광대역 신호로 변환되어 전송된 신호는 수신단에서 정교한 복원 과정을 거친다. 수신단은 송신단에서 사용한 것과 정확히 동일하고, 시간적으로 완벽하게 동기화된 확산 부호를 생성한다. 그리고 수신된 광대역 신호에 이 복제된 확산 부호를 다시 곱한다. 이 과정을 상관(correlation) 또는 역확산(despreading)이라고 부른다. 이 역확산 과정을 통해, 넓게 퍼져 있던 신호 에너지는 다시 원래의 좁은 대역폭으로 집중되어 본래의 정보 데이터 신호가 복원된다.1 이처럼 DSSS는 확산 부호라는 '비밀 키'를 공유하는 송수신단 사이에서만 의미 있는 통신을 가능하게 하는 강력한 메커니즘을 제공한다.

### 0.2  DSSS의 역사적 배경과 기술적 의의

DSSS 기술의 탄생과 발전은 군사적 필요성과 밀접한 관련이 있다. 그 기원은 제2차 세계대전 당시, 적의 통신 감청(eavesdropping)과 고의적인 전파 방해, 즉 재밍(jamming)으로부터 아군의 통신을 보호해야 하는 절박한 요구에서 출발했다.9 당시의 협대역 통신 방식은 특정 주파수에 강한 신호를 집중시켰기 때문에 적에게 쉽게 탐지되고, 해당 주파수에 강력한 방해 전파를 송출하는 재밍 공격에 매우 취약했다.

이러한 문제를 해결하기 위해 DSSS는 신호의 전력 스펙트럼 밀도를 의도적으로 낮춰 광대역 잡음처럼 위장하는 전략을 채택했다. 이는 신호 탐지 자체를 어렵게 만드는 LPI(Low Probability of Intercept) 특성을 부여했으며, 설령 신호가 탐지되더라도 사전에 약속된 확산 부호를 알지 못하면 정보를 해독할 수 없게 하여 강력한 보안성을 제공했다.3 스위스 발명가 구스타브 구아넬라(Gustav Guanella)가 비밀 신호 전송 방법으로 이 기술을 제안했으며 11, 헐리우드 여배우 헤디 라머(Hedy Lamarr)와 작곡가 조지 앤타일(George Antheil)이 어뢰의 무선 조종을 위한 주파수 도약(Frequency Hopping, FHSS) 기술에 대한 특허를 획득한 일화는 대역 확산 기술의 초기 역사에서 상징적인 사건으로 남아있다.12

냉전 시대를 거치며 군사 통신의 핵심 기술로 발전한 DSSS는 이후 통신 기술의 패러다임이 민간 상업 분야로 이동하면서 새로운 국면을 맞이했다. 제한된 주파수 자원을 다수의 사용자가 효율적으로 공유해야 하는 상업 이동통신 환경에서 DSSS의 고유한 특성들이 재해석되기 시작한 것이다.8 군사적 관점에서의 '항재밍' 능력은 상업적 관점에서 '다중 사용자 간 간섭(multi-user interference) 억제' 능력으로, '보안을 위한 비밀 키(확산 부호)'는 '사용자를 구분하기 위한 고유 식별자'로 그 의미가 확장되었다.

이러한 기술적 의의의 전환은 코드 분할 다중 접속(CDMA)이라는 혁신적인 다중 접속 기술의 탄생으로 이어졌다. 또한, DSSS의 정밀한 시간 분해능과 간섭 저항성은 위성 항법 시스템(GPS)의 핵심 원리가 되었고, 초기 무선 근거리 통신망(WLAN, Wi-Fi)과 저전력 사물 인터넷(IoT) 통신(Zigbee) 등 현대 무선 통신 기술의 근간을 이루는 핵심 기술로 확고히 자리매김하게 되었다.2 이는 군사적 필요에 의해 탄생한 기술이 민간의 요구에 부응하여 성공적으로 전환(spin-off)되고 상업적으로 꽃피운 대표적인 사례라 할 수 있다.

## 1.  DSSS의 이론적 기반과 수학적 모델링

### 1.1  섀넌의 채널 용량 정리에 기반한 DSSS의 정당성

DSSS 기술의 이론적 타당성은 현대 정보 이론의 아버지 클로드 섀넌(Claude Shannon)이 정립한 채널 용량 정리(Channel Capacity Theorem)에서 찾을 수 있다.12 이 정리는 특정 채널에서 오류 없이 전송할 수 있는 정보의 최대 전송률, 즉 채널 용량 $C$(단위: bits per second, bps)가 채널의 대역폭 $B$(단위: Hertz, Hz)와 신호 대 잡음비(Signal-to-Noise Ratio, $S/N$)에 의해 결정됨을 수학적으로 명시한다. 섀넌-하틀리(Shannon-Hartley) 정리에 따른 채널 용량 공식은 다음과 같다.3
$$
C = B \log_2(1 + S/N)
$$
이 공식은 통신 시스템 설계에 있어 근본적인 통찰을 제공한다. 즉, 채널 용량 $C$라는 목표 성능을 달성하기 위해, 시스템 설계자는 대역폭 $B$와 신호 대 잡음비 $S/N$라는 두 가지 자원을 서로 교환(trade-off)할 수 있다는 것이다.3 예를 들어, 잡음이 매우 심하여 

$S/N$ 값이 극도로 낮은 열악한 통신 환경이라 할지라도, 사용 가능한 대역폭 $B$를 충분히 넓게 확보할 수 있다면 이론적으로는 원하는 채널 용량 $C$를 유지하는 것이 가능하다. DSSS는 바로 이 원리를 적극적으로 활용하는 기술이다. 신호의 전력을 높이는 대신, 사용 대역폭을 극단적으로 넓힘으로써 통신의 신뢰성을 확보한다.

특히 $S/N \ll 1$, 즉 신호의 전력이 잡음 전력보다 훨씬 낮은 극한의 환경을 가정하여 위 공식을 근사하면 DSSS의 원리가 더욱 명확해진다. 로그의 밑을 2에서 자연상수 $e$로 변환하고, $x$가 매우 작을 때 $\ln(1+x) \approx x$라는 매클로린 급수(Maclaurin series) 근사를 적용하면 다음과 같은 관계식을 유도할 수 있다.12
$$
\frac{C}{B} = \frac{1}{\ln 2} \ln(1 + S/N) \approx \frac{1.443}{1} \left( \frac{S}{N} \right)
$$
이를 정리하면, $S/N \approx C/B$ 라는 매우 중요한 근사식을 얻는다. 이 식은 시스템이 요구하는 신호 대 잡음비 $S/N$가 정보 전송률 $C$와 확산된 대역폭 $B$의 비율에 의해 결정됨을 의미한다. 즉, 대역폭 $B$를 정보 전송률 $C$에 비해 매우 크게 만들면(대역 확산), 요구되는 $S/N$을 0보다 작은 값(음수 dB)으로 낮출 수 있다. 이것이 바로 DSSS 신호가 잡음 레벨 이하의 매우 낮은 전력 스펙트럼 밀도를 가지고도 통신이 가능한 이론적 배경이 된다.12

### 1.2  확산 및 역확산 과정의 수학적 모델

DSSS의 핵심 과정인 확산과 역확산은 수학적으로 매우 간결한 모델로 표현할 수 있다. 이 모델은 확산 부호의 대수적 특성을 기반으로 한다.

#### 1.2.1  송신단의 확산 과정

송신단에서는 전송하고자 하는 원본 정보 데이터 신호 $d(t)$에 확산 부호 $c(t)$를 직접 곱한다. 여기서 $d(t)$는 비트율이 $R_b$인 협대역 신호이며, $c(t)$는 칩 속도가 $R_c$ ($R_c \gg R_b$)인 광대역 신호다. 두 신호는 일반적으로 $+1$과 $-1$의 값을 갖는 극성(polar) NRZ(Non-Return-to-Zero) 파형으로 표현된다. 전송될 확산 신호 $s(t)$는 다음과 같이 정의된다.1
$$
s(t) = d(t) \cdot c(t)
$$
시간 영역에서의 곱셈은 주파수 영역에서 컨볼루션(convolution)에 해당하므로, $d(t)$의 좁은 스펙트럼은 $c(t)$의 넓은 스펙트럼과 컨볼루션되어 결과적으로 $c(t)$와 유사한 넓은 대역폭을 갖게 된다. 디지털 회로에서는 이 곱셈 과정이 주로 XOR(Exclusive OR) 논리 연산으로 구현된다.1

#### 1.2.2  수신단의 역확산 과정

확산된 신호 $s(t)$는 무선 채널을 통과하면서 가산성 백색 가우시안 잡음(AWGN) $n(t)$과 의도적이거나 비의도적인 간섭 신호 $j(t)$에 의해 오염된다. 따라서 수신기에 도달하는 신호 $r(t)$는 다음과 같다.
$$
r(t) = s(t) + n(t) + j(t) = d(t) \cdot c(t) + n(t) + j(t)
$$
수신단에서는 이 수신 신호 $r(t)$에 송신단에서 사용된 것과 동일하고 완벽하게 동기화된 확산 부호 $c(t)$를 다시 곱하여 역확산 과정을 수행한다. 역확산된 신호 $z(t)$는 다음과 같다.1
$$
z(t) = r(t) \cdot c(t) = [d(t) \cdot c(t) + n(t) + j(t)] \cdot c(t)
$$
여기서 확산 부호의 중요한 대수적 특성이 작용한다. 확산 부호의 각 칩은 $+1$ 또는 $-1$의 값을 가지므로, 자기 자신과의 곱은 항상 1이 된다. 즉, $c(t) \cdot c(t) = c^2(t) = 1$ 이다. 이 특성을 이용하여 위 식을 전개하면 다음과 같이 정리된다.20
$$
z(t) = d(t) \cdot c^2(t) + n(t) \cdot c(t) + j(t) \cdot c(t) = d(t) + n'(t) + j'(t)
$$
이 결과는 매우 중요한 의미를 갖는다. 원래의 정보 신호 $d(t)$는 $c^2(t)=1$에 의해 손상 없이 완벽하게 복원된다. 반면, 잡음 $n(t)$과 간섭 $j(t)$는 확산 부호 $c(t)$와 곱해져 각각 새로운 광대역 신호 $n'(t)$와 $j'(t)$로 변환된다. 즉, 원하는 신호는 '역확산(despread)'되고, 원치 않는 신호들은 오히려 '확산(spread)'되는 차별적인 과정이 일어나는 것이다.11

### 1.3  간섭 신호 억제 원리

DSSS의 강력한 간섭 억제 능력은 바로 이 역확산 과정의 차별적 특성에서 비롯된다. 역확산된 신호 $z(t)$는 후단의 협대역 필터(일반적으로 데이터 비트 주기에 맞춰 적분 후 결정하는 정합 필터 또는 상관기 형태)를 통과하게 된다.

- **원 신호의 복원:** 복원된 정보 신호 $d(t)$는 원래의 좁은 대역폭을 가지고 있으므로, 이 협대역 필터를 대부분의 에너지 손실 없이 통과한다.
- **간섭 신호의 확산 및 제거:** 반면, 확산된 잡음 $n'(t)$와 간섭 $j'(t)$는 매우 넓은 대역폭에 걸쳐 에너지가 분산되어 있다. 따라서 협대역 필터는 이들의 에너지 중 극히 일부만을 통과시키고 대부분을 차단한다.20

특히, 특정 주파수에 강한 에너지가 집중된 협대역 재밍(narrowband jamming) 신호 $j(t)$의 경우, 역확산 과정에서 그 에너지가 확산 부호의 넓은 대역폭으로 퍼지면서 전력 스펙트럼 밀도가 급격히 낮아진다. 결과적으로 수신 필터를 통과한 후의 간섭 전력은 현저하게 감소하여, 정보 신호의 검출에 거의 영향을 미치지 못하게 된다.11 이처럼 DSSS는 송신단과 수신단이 공유하는 확산 부호라는 '대칭적 키'를 이용하여, 이 키를 모르는 외부 간섭 신호에 대해서는 '비대칭적'으로 불리한 변조를 가함으로써 간섭을 효과적으로 억제하는 정교한 메커니즘을 구현한다.

## 2.  핵심 요소: 의사 잡음(PN) 부호

DSSS 시스템의 성능과 특성을 결정하는 가장 핵심적인 요소는 바로 확산 과정에 사용되는 의사 잡음(Pseudo-Noise, PN) 부호이다. PN 부호는 단순한 이진 수열이 아니라, 시스템의 동기 성능, 다중 접속 용량, 보안 수준을 규정하는 설계의 근간이라 할 수 있다.

### 2.1  PN 부호의 정의와 요구 특성

PN 부호는 이름에서 알 수 있듯이, 통계적 특성은 백색 잡음(white noise)과 매우 유사하지만, 실제로는 결정론적인(deterministic) 알고리즘에 의해 생성되는 주기적인 이진 시퀀스를 의미한다.10 즉, 생성 규칙과 초기값을 알면 얼마든지 동일한 시퀀스를 재현할 수 있지만, 규칙을 모르는 관찰자에게는 무작위적인 잡음처럼 보인다. 이러한 '예측 가능한 무작위성'이 DSSS의 핵심 원리를 가능하게 한다.

이상적인 PN 부호가 갖추어야 할 주요 통계적 특성은 다음과 같다 19:

1. **균형성 (Balance Property):** 시퀀스의 한 주기 내에서 $+1$의 개수와 $-1$(또는 '0')의 개수가 거의 동일해야 한다. 최대 길이 시퀀스(m-sequence)의 경우, 그 차이는 정확히 1이다. 이 특성은 시퀀스의 DC 성분을 최소화하여, 변조 과정에서 반송파(carrier)를 효과적으로 억제하고 전력 효율을 높이는 데 기여한다.19
2. **런 길이 분포 (Run-length Distribution):** '런(run)'이란 시퀀스 내에서 동일한 비트가 연속적으로 나타나는 구간을 의미한다. 좋은 PN 부호는 런의 길이가 잡음의 통계적 분포와 유사한 패턴을 보여야 한다. 예를 들어, 전체 런의 절반은 길이가 1이고, 1/4은 길이가 2, 1/8은 길이가 3인 방식으로 분포한다. 이 특성은 시퀀스가 특정 패턴에 치우치지 않고 충분히 무작위적임을 보장하는 척도이다.19
3. **상관 특성 (Correlation Property):** PN 부호의 가장 중요한 특성으로, 자기상관(autocorrelation)과 상호상관(cross-correlation)으로 나뉜다.
   - **자기상관:** 시퀀스 자신과 시간적으로 지연된 버전 간의 유사도를 나타낸다. 이상적인 PN 부호는 시간 지연이 없을 때만 매우 큰 값을 갖고, 그 외의 모든 지연에 대해서는 매우 작은 값을 가져야 한다. 이 뾰족한(impulsive) 자기상관 특성은 수신기가 송신 신호와 정확한 시간 동기를 맞추는 데 결정적인 역할을 한다.
   - **상호상관:** 서로 다른 두 PN 부호 간의 유사도를 나타낸다. CDMA와 같이 다수의 사용자가 동일한 주파수 자원을 공유하는 시스템에서는, 각 사용자에게 할당된 PN 부호들 간의 상호상관 값이 모든 시간 지연에 대해 가능한 한 0에 가까워야 한다. 이는 다른 사용자의 신호가 간섭으로 작용하는 것을 최소화하기 위함이다.

### 2.2  PN 부호의 생성: 선형 궤환 시프트 레지스터(LFSR)

이러한 복잡한 통계적 특성을 만족하는 긴 주기의 PN 부호는 선형 궤환 시프트 레지스터(Linear Feedback Shift Register, LFSR)라는 비교적 간단한 디지털 회로를 통해 효율적으로 생성될 수 있다.25 LFSR은 n개의 플립플롭으로 구성된 시프트 레지스터와, 레지스터의 특정 위치(탭, tap)의 출력을 입력으로 하는 XOR(Exclusive-OR) 게이트의 조합으로 이루어진다.25

매 클럭 펄스가 인가될 때마다 레지스터의 모든 비트는 한 칸씩 옆으로 이동(shift)하고, 탭으로 지정된 위치의 비트들이 XOR 연산된 결과가 레지스터의 첫 번째 입력으로 들어간다(feedback). 이 피드백 구조와 탭의 위치는 수학적으로 '생성 다항식(Generator Polynomial)'으로 표현되며, 어떤 생성 다항식을 사용하느냐에 따라 생성되는 PN 시퀀스의 주기와 특성이 결정된다.21 LFSR은 피드백을 구성하는 방식에 따라 크게 두 가지 구조로 나뉜다.25

- **피보나치(Fibonacci) 구조:** 다수의 탭 출력을 하나의 XOR 게이트 묶음으로 모아서 레지스터의 첫 단에 피드백하는 직렬 구조이다.
- **갈루아(Galois) 구조:** 각 탭 위치에서 XOR 연산이 일어나 다음 단의 입력에 영향을 주는 병렬 구조이다. 갈루아 구조는 신호의 전파 지연(propagation delay)이 적어 더 높은 클럭 속도에서 동작할 수 있는 장점을 가진다.25

### 2.3  PN 부호의 상관 특성

상관 특성은 PN 부호의 성능을 평가하는 가장 중요한 척도이며, 자기상관과 상호상관으로 구분된다.

#### 2.3.1  자기상관 (Autocorrelation)

자기상관 함수는 주기 $N$을 갖는 PN 시퀀스 $c(t)$와 이를 시간 $\tau$만큼 지연시킨 $c(t-\tau)$ 간의 유사도를 측정한다. 이산 시퀀스 $\{c_i\}$ ($c_i \in \{+1, -1\}$)에 대한 주기적 자기상관 함수 $R_c(\tau)$는 다음과 같이 정의된다.19
$$
R_c(\tau) = \sum_{i=0}^{N-1} c_i c_{i-\tau}
$$
여기서 인덱스는 모두 모듈로 $N$ 연산된다. DSSS 시스템에 사용되는 우수한 PN 부호, 특히 m-시퀀스는 '이상적인 2-레벨(ideal two-level) 자기상관' 특성을 보인다.24 이는 자기상관 함수 값이 단 두 개의 값만을 갖는다는 의미이다.
$$
R_c(\tau) = \begin{cases} N, & \text{if } \tau \equiv 0 \pmod{N} \\ -1, & \text{if } \tau \not\equiv 0 \pmod{N} \end{cases}
$$
이 수식은 시간 지연이 없는 경우($\tau=0$), 즉 시퀀스가 완벽하게 정렬되었을 때 상관 값은 시퀀스의 주기 $N$과 동일한 최대 피크(peak) 값을 갖고, 1 칩(chip)이라도 시간 지연이 발생하는 모든 경우($\tau \neq 0$)에는 $-1$이라는 매우 작고 일정한 부엽(sidelobe) 값을 가짐을 의미한다.23 이처럼 뾰족한 델타 함수(delta function)와 유사한 자기상관 특성은 수신기가 수많은 다중 경로 신호와 잡음 속에서도 정확한 동기를 획득하고 유지하는 데 결정적인 역할을 한다.19

#### 2.3.2  상호상관 (Cross-correlation)

상호상관 함수는 주기 $N$을 갖는 서로 다른 두 PN 시퀀스 $\{c_i\}$와 $\{d_i\}$ 간의 유사도를 측정하며, 다음과 같이 정의된다.27
$$
R_{c,d}(\tau) = \sum_{i=0}^{N-1} c_i d_{i-\tau}
$$
CDMA와 같은 다중 접속 시스템에서는 서로 다른 사용자에게 고유한 PN 부호를 할당하여 채널을 공유한다. 이때, 특정 사용자의 수신기는 자신이 사용하는 PN 부호와는 높은 자기상관 값을, 다른 사용자의 PN 부호와는 낮은 상호상관 값을 가져야 원하는 신호만을 선택적으로 복원할 수 있다. 따라서 CDMA 시스템에 사용되는 PN 부호 집합은 임의의 두 부호 간 상호상관 값이 모든 시간 지연 $\tau$에 대해 가능한 한 작게 유지되도록 설계되어야 한다.11

### 2.4  주요 PN 부호 계열 분석

이러한 요구 특성을 만족시키기 위해 여러 종류의 PN 부호 계열이 연구되고 사용되어 왔다.

- m-시퀀스 (Maximal-Length Sequences):

  $n$단의 LFSR이 생성할 수 있는 가장 긴 주기인 $N = 2^n - 1$을 갖는 시퀀스이다.20 m-시퀀스는 앞서 언급한 균형성, 런 길이 분포, 그리고 이상적인 2-레벨 자기상관 특성을 모두 만족하는, 이론적으로 가장 우수한 유사 잡음 특성을 지닌다.23 그러나 주어진 $n$에 대해 생성할 수 있는 m-시퀀스의 수가 제한적이어서, 많은 수의 사용자를 수용해야 하는 대규모 CDMA 시스템에는 부적합한 단점이 있다.

- 골드 부호 (Gold Codes):

  m-시퀀스의 제한된 개수 문제를 해결하기 위해 고안된 부호 계열이다. 골드 부호는 '선호 쌍(preferred pair)'이라 불리는 특정 조건을 만족하는 두 개의 m-시퀀스를 XOR(modulo-2 addition)하여 생성한다.32 하나의 m-시퀀스를 고정하고 다른 하나의 m-시퀀스를 시간 이동시키면서 XOR 연산을 반복하면, 원래의 두 m-시퀀스를 포함하여 총 

  $N+2$개의 골드 부호 시퀀스를 얻을 수 있다. 골드 부호의 자기상관 특성은 3-레벨로 m-시퀀스보다 다소 열화되지만, 임의의 두 부호 간 상호상관 값이 예측 가능하고 낮은 수준으로 제어된다는 큰 장점이 있다. 이러한 특성 덕분에 많은 수의 고유 코드가 필요한 GPS의 C/A 코드나 3G 이동통신(WCDMA)과 같은 대규모 다중 접속 시스템에서 널리 사용된다.32

- 바커 부호 (Barker Codes):

  비주기(aperiodic) 자기상관 함수의 부엽(sidelobe) 크기가 모든 시간 지연에 대해 최대 1을 넘지 않는 특별한 이진 시퀀스이다.29 바커 부호는 그 길이가 13을 초과하는 것은 존재하지 않는 것으로 알려져 있으며, 현재까지 알려진 것은 2, 3, 4, 5, 7, 11, 13의 길이를 갖는 것들뿐이다. 길이가 짧고 비주기 자기상관 특성이 매우 우수하여, 패킷 통신에서 패킷의 시작점을 빠르고 정확하게 검출하는 초기 동기 획득(acquisition) 용도로 매우 유용하다. 초기 WLAN 표준인 IEEE 802.11b의 1 Mbps 및 2 Mbps 전송 속도에서 사용된 11-chip 바커 코드(`10110111000`)가 가장 대표적인 예이다.18

## 3.  DSSS 시스템 성능 분석

DSSS 시스템의 성능은 주로 외부의 간섭이나 잡음을 얼마나 효과적으로 억제하고 안정적인 통신을 유지할 수 있는가에 의해 평가된다. 이러한 성능을 정량적으로 나타내는 핵심 지표가 바로 처리 이득(Processing Gain)과 재밍 마진(Jamming Margin)이다.

### 3.1  처리 이득 (Processing Gain, PG)

처리 이득은 대역 확산 기술을 사용함으로써 얻게 되는 신호 대 잡음비(SNR)의 개선 효과를 나타내는 가장 근본적인 지표이다. 이는 확산된 신호의 대역폭($W_{ss}$)과 원래 정보 신호의 대역폭($W_{data}$)의 비율로 정의된다.3 실제 시스템에서는 칩 속도($R_c$)와 데이터 비트 속도($R_b$)의 비율로 간단히 계산할 수 있다.21
$$
PG = \frac{W_{ss}}{W_{data}} \approx \frac{R_c}{R_b}
$$
처리 이득은 보통 데시벨(dB) 단위로 표현되며, 다음과 같이 계산된다.
$$
PG_{dB} = 10 \log_{10} \left( \frac{R_c}{R_b} \right)
$$
예를 들어, IS-95 CDMA 시스템에서 9.6 kbps의 음성 데이터가 1.2288 Mcps의 칩 속도로 확산될 때, 처리 이득은 $1,228,800 / 9,600 = 128$이며, 이를 dB로 환산하면 $10 \log_{10}(128) \approx 21$ dB가 된다.38

처리 이득의 물리적 의미는 역확산 과정에서 명확하게 드러난다. 수신단에서 역확산을 수행하면, 원하는 정보 신호의 에너지는 원래의 좁은 대역폭으로 다시 모이면서 신호 전력이 $PG$배만큼 증폭되는 효과를 얻는다. 반면, 수신단에 함께 들어온 협대역 간섭 신호는 역확산 과정에서 오히려 그 에너지가 넓은 대역으로 퍼지면서 전력 스펙트럼 밀도가 $PG$배만큼 감소하게 된다.11 결과적으로, 수신 필터를 통과한 후의 신호 대 간섭비(Signal-to-Interference Ratio, SIR)는 처리 이득만큼 개선된다. 따라서 처리 이득이 높을수록 시스템의 간섭 억제 능력이 강력해진다.

### 3.2  재밍 마진 (Jamming Margin, Mj)

재밍 마진은 DSSS 시스템이 특정 통신 품질(일반적으로 목표 비트 오류율, BER)을 유지하면서 견뎌낼 수 있는 최대 간섭(재밍) 전력의 수준을 정량적으로 나타내는 지표이다.39 이는 시스템의 강인성(robustness)을 직접적으로 보여주는 중요한 성능 파라미터이다.

재밍 마진은 시스템의 처리 이득, 통신 품질 유지를 위해 요구되는 최소 신호 품질, 그리고 시스템 내부의 손실에 의해 결정된다. 수학적으로 재밍 마진($M_j$)은 다음과 같이 표현된다.41
$$
M_j (dB) = PG (dB) - (E_b/N_0)_{req} (dB) - L (dB)
$$
여기서 각 항의 의미는 다음과 같다.

- $PG (dB)$: 시스템의 처리 이득 (dB).
- $(E_b/N_0)_{req} (dB)$: 목표 BER을 달성하기 위해 복조기(demodulator) 입력단에서 요구되는 최소 비트 에너지 대 잡음 전력 스펙트럼 밀도 비 (dB). 이는 사용된 변조 방식(예: BPSK, QPSK)과 채널 코딩에 의해 결정된다.
- $L (dB)$: 시스템 구현 과정에서 발생하는 각종 손실(loss) (dB).

이 공식은 DSSS 시스템 설계의 핵심적인 트레이드오프 관계를 명확히 보여준다. 재밍 마진을 높여 시스템의 강인성을 향상시키기 위해서는, 처리 이득($PG$)을 높이거나, 보다 효율적인 변복조 및 채널 코딩 기술을 사용하여 요구 $E_b/N_0$ 값을 낮추어야 한다. 처리 이득을 높이는 것은 곧 칩 속도를 높이거나 데이터 속도를 낮추는 것을 의미하며, 이는 더 넓은 대역폭을 요구하거나 전송 효율을 희생해야 함을 시사한다.41

### 3.3  장점 및 단점 종합 분석

DSSS 기술은 그 고유한 원리로부터 비롯되는 명확한 장점과 그에 따르는 필연적인 단점을 동시에 가지고 있다.

#### 3.3.1  장점

- **강력한 항재밍 및 간섭 억제 능력:** DSSS의 가장 큰 장점으로, 처리 이득에 비례하여 고의적이거나 비고의적인 간섭 신호의 영향을 효과적으로 억제할 수 있다.2
- **높은 보안성 및 낮은 탐지 확률(LPI):** 확산된 신호는 전력 스펙트럼 밀도가 매우 낮아 광대역 잡음처럼 보이기 때문에 신호의 존재 자체를 탐지하기 어렵다. 또한, 사전에 약속된 확산 부호를 알지 못하면 정보 복원이 사실상 불가능하여 도청에 강하다.1
- **다중 경로 페이딩에 대한 강인성:** 신호가 여러 경로를 통해 시간차를 두고 수신되는 다중 경로 환경에서, DSSS는 각 경로의 신호를 분리할 수 있는 높은 시간 분해능을 제공한다. RAKE 수신기와 같은 기술을 사용하면, 이들 다중 경로 신호를 개별적으로 복조한 후 동위상으로 결합하여 오히려 수신 신호의 품질을 향상시킬 수 있다.
- **코드 분할 다중 접속(CDMA) 지원:** 사용자마다 상호상관이 낮은 고유의 확산 부호를 할당함으로써, 다수의 사용자가 동일한 주파수 대역과 시간을 공유하면서 동시에 통신하는 것이 가능하다. 이는 주파수 자원 활용 효율을 극대화한다.

#### 3.3.2  단점

- **정밀한 동기화 요구:** DSSS 시스템이 제대로 동작하기 위해서는 수신단에서 생성하는 확산 부호가 수신된 신호의 확산 부호와 칩(chip) 단위로 매우 정밀하게 시간 동기화되어야 한다. 이 때문에 초기 동기를 획득(acquisition)하고 이를 지속적으로 추적(tracking)하기 위한 회로가 매우 복잡하며, 특히 긴 PN 코드를 사용할 경우 동기 획득에 소요되는 시간이 길어질 수 있다.3
- **넓은 주파수 대역폭 점유:** 기술의 본질상, 원래 정보 신호의 대역폭보다 처리 이득만큼 넓은 주파수 대역을 필요로 한다. 이는 주파수 자원이 제한적인 환경에서 단점으로 작용할 수 있다.
- **근원 문제(Near-Far Problem):** CDMA 시스템에서 발생하는 고질적인 문제로, 기지국에 가까운 사용자의 강한 신호가 멀리 있는 사용자의 약한 신호를 덮어버려 통신을 불가능하게 만드는 현상이다. 이를 해결하기 위해서는 모든 사용자의 신호가 기지국에 거의 동일한 전력으로 도달하도록 만드는 매우 정교하고 빠른 전력 제어(power control) 메커니즘이 필수적으로 요구된다.3

### 3.4 표 1: DSSS와 FHSS 기술 비교

DSSS는 대역 확산 기술의 한 종류이며, 또 다른 대표적인 방식인 주파수 도약 대역 확산(Frequency Hopping Spread Spectrum, FHSS)과 비교함으로써 그 특성을 더욱 명확히 이해할 수 있다. 두 기술은 동일한 목표를 다른 방식으로 달성하며, 이는 각 기술의 장단점과 적합한 응용 분야를 결정한다.3

| 특징            | DSSS (Direct Sequence Spread Spectrum)                       | FHSS (Frequency Hopping Spread Spectrum)                     |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **확산 방식**   | 고속의 PN 코드를 데이터에 직접 곱하여 넓은 대역에 고정적으로 확산 47 | 정해진 패턴에 따라 좁은 대역의 반송파 주파수를 빠르게 변경(도약) 48 |
| **대역폭 점유** | 항상 넓은 대역폭을 모두 사용                                 | 순간적으로는 좁은 대역폭을 사용하나, 시간적으로 넓은 대역을 사용 |
| **간섭 회피**   | 처리 이득을 통해 간섭 신호를 억제 (간섭에 맞서 싸움) 3       | 간섭이 있는 주파수를 피해 다른 주파수로 도약 (간섭을 피함) 49 |
| **동기 획득**   | 칩 단위의 정밀한 동기가 필요하여 복잡하고 시간이 오래 걸림 3 | 홉(hop) 단위 동기화로 상대적으로 간단하고 빠름 3             |
| **다중 경로**   | RAKE 수신기를 통해 다중 경로 성분을 효과적으로 처리 가능 (강점) 3 | 다중 경로에 상대적으로 취약                                  |
| **근원 문제**   | 존재함. 정밀한 전력 제어 필요 3                              | 존재하지 않음 (서로 다른 주파수 사용)                        |
| **주요 응용**   | CDMA, GPS, IEEE 802.11b, Zigbee 7                            | Bluetooth, 초기 IEEE 802.11, 일부 무선 전화기                |

## 4.  주요 통신 시스템별 DSSS 적용 사례 분석

DSSS 기술은 단일한 형태로 존재하는 것이 아니라, 각 통신 시스템의 고유한 목표와 요구사항에 맞춰 PN 부호, 변조 방식, 시스템 파라미터 등이 정교하게 조율된 '적응형 프레임워크'로서 다양한 분야에 적용되어 왔다. 각 시스템이 DSSS를 어떻게 변용하여 최적의 성능을 달성했는지 살펴보는 것은 이 기술에 대한 깊이 있는 이해를 제공한다.

### 4.1  코드 분할 다중 접속 (CDMA: IS-95)

DSSS의 다중 접속 능력을 상업적으로 가장 성공시킨 사례가 바로 2세대 이동통신 표준인 IS-95 CDMA이다.1 IS-95는 모든 사용자가 1.25 MHz라는 동일한 주파수 대역을 공유하면서 통화 채널을 설정하는 혁신적인 방식을 도입했다.50 이는 순방향 링크와 역방향 링크에서 서로 다른 DSSS 전략을 구사함으로써 가능했다.

#### 4.1.1  순방향 링크 (기지국 -->> 단말기)

기지국에서 다수의 단말기로 신호를 보내는 순방향 링크는 모든 신호가 하나의 송신점에서 출발하므로 완벽한 동기화가 가능하다. IS-95는 이 특성을 활용하여 직교 코드를 통한 채널 구분을 구현했다.

- **직교 채널화 (Orthogonal Channelization):** 순방향 링크는 총 64개의 채널로 구성된다. 이 채널들은 길이가 64인 월시 부호(Walsh Code) 집합을 통해 서로 완벽하게 직교적으로 분리된다.50 월시 부호는 상호상관 값이 0인 특성을 가지므로, 동기만 정확히 맞는다면 서로 다른 월시 부호로 확산된 신호들은 서로에게 전혀 간섭을 일으키지 않는다.30 이 64개의 월시 부호는 파일럿, 동기, 페이징, 트래픽 채널 등 각각의 용도에 따라 할당된다.
- **스크램블링 (Scrambling):** 월시 부호로 채널화된 64개의 신호를 모두 합산한 후, 기지국(또는 섹터)을 고유하게 식별하기 위한 짧은 PN 부호(short PN code)로 최종적으로 스크램블링한다. 이 과정은 인접한 기지국에서 오는 신호와의 간섭을 잡음처럼 만들어 억제하고, 단말기가 자신이 속한 셀을 식별할 수 있도록 돕는 역할을 한다.53

**표 2: IS-95 순방향 채널의 월시 부호 할당 예**

| 월시 부호 번호           | 채널 종류                      | 기능                                                         |
| ------------------------ | ------------------------------ | ------------------------------------------------------------ |
| W0                       | 파일럿 채널 (Pilot Channel)    | 모든 단말기가 동기를 맞추고 채널 상태를 추정하기 위한 기준 신호 |
| W32                      | 동기 채널 (Sync Channel)       | 시스템 시간 및 기지국 정보를 담고 있는 브로드캐스트 채널     |
| W1 ~ W7                  | 페이징 채널 (Paging Channels)  | 단말기를 호출(page)하거나 시스템 정보를 전달하는 채널        |
| 나머지 (W8-W31, W33-W63) | 트래픽 채널 (Traffic Channels) | 사용자의 음성 및 데이터 통신을 위한 개별 채널                |

#### 4.1.2  역방향 링크 (단말기 -->> 기지국)

여러 단말기에서 하나의 기지국으로 신호를 보내는 역방향 링크는 각 단말기의 위치와 이동성이 달라 완벽한 동기화가 불가능하다. 따라서 직교성을 보장할 수 없는 월시 부호 대신, 비동기 환경에 더 강인한 PN 부호를 사용자 구분에 사용한다.

- **사용자 구분:** 각 단말기는 `2^42-1`이라는 천문학적인 주기를 갖는 고유한 긴 PN 부호(long PN code)로 자신의 데이터를 확산한다.53 이 긴 PN 부호는 사실상 모든 사용자가 서로 다른 코드를 사용하는 것과 같은 효과를 내어, 기지국이 각 단말기의 신호를 구분할 수 있게 한다.
- **변조 방식:** 역방향 링크는 OQPSK(Offset Quadrature Phase Shift Keying) 변조 방식을 채택했다. OQPSK는 QPSK의 I(In-phase) 채널과 Q(Quadrature) 채널 신호를 반 칩(half-chip) 시간만큼 의도적으로 어긋나게 전송하는 방식이다. 이로 인해 신호의 위상이 한 번에 180도 급격하게 변하는 것을 방지하고 신호 파형의 진폭 변동을 최소화할 수 있다. 이는 전력 효율이 중요한 배터리 기반의 단말기에서 전력 증폭기(Power Amplifier)의 부담을 줄여주는 효과가 있다.53

#### 4.1.3  근원 문제(Near-Far Problem)와 전력 제어

CDMA 시스템의 가장 큰 난제는 '근원 문제'이다. 기지국에 가까이 있는 단말기(Near)의 수신 신호는 강하고, 멀리 있는 단말기(Far)의 수신 신호는 약하다. 만약 각 단말기가 동일한 전력으로 송신한다면, 기지국에서는 강한 신호가 약한 신호를 완전히 덮어버리는 간섭으로 작용하여 약한 신호의 통신을 불가능하게 만든다.45 이 문제를 해결하고 시스템 용량을 최대화하기 위해, IS-95는 모든 단말기의 신호가 기지국에 거의 동일한 세기로 도달하도록 만드는 정밀한 전력 제어(Power Control) 메커니즘을 도입했다.45

- **개방 루프 전력 제어 (Open Loop Power Control):** 단말기가 기지국으로부터 수신하는 순방향 신호의 세기를 스스로 측정하여, 그에 반비례하게 자신의 송신 전력을 조절하는 방식이다. 거리에 따른 경로 손실(path loss)과 같은 느린 채널 변화에 대응한다.45
- **폐쇄 루프 전력 제어 (Closed Loop Power Control):** 기지국이 각 단말기로부터 수신하는 신호의 품질(Eb/No)을 1.25ms마다 측정하고, 목표 품질과 비교하여 단말기에게 송신 전력을 1dB씩 높이거나 낮추라는 명령(Power Control Bit)을 지속적으로 보낸다. 이는 다중 경로 페이딩과 같은 빠른 채널 변화에 실시간으로 대응하는 핵심적인 역할을 수행한다.45

### 4.2  위성 항법 시스템 (GPS)

GPS는 지구 상공 약 20,000 km에 떠 있는 위성에서 송신하는 매우 미약한 신호를 수신하여 위치를 계산하는 시스템이다. DSSS 기술은 이러한 극한의 저전력 환경에서 신호를 안정적으로 검출하고, 나노초(nanosecond) 수준의 정밀한 시간 지연을 측정하여 위성까지의 거리를 계산하는 데 핵심적인 역할을 한다.13

#### 4.2.1  C/A 코드 (Coarse/Acquisition Code) 생성

민간용 GPS 수신기가 사용하는 L1 반송파 신호는 C/A 코드라는 DSSS 신호로 변조된다. C/A 코드는 주기가 1023 칩이고 1ms마다 반복되는 골드 부호(Gold Code)이다.32 이 코드는 각 위성을 고유하게 식별하는 ID 역할을 한다.

C/A 코드는 두 개의 10단 LFSR, 즉 G1과 G2 레지스터의 출력을 XOR하여 생성된다.60

- **G1 레지스터:** 피드백 탭이 3번과 10번으로 고정되어 있다.
- **G2 레지스터:** 피드백 탭이 2, 3, 6, 8, 9, 10번으로 고정되어 있다.

각 위성(PRN 번호로 식별)마다 고유한 C/A 코드를 생성하기 위해, G2 레지스터의 출력 탭 조합을 다르게 설정한다. 예를 들어, PRN 1번 위성은 G2의 2번과 6번 탭을, PRN 2번 위성은 3번과 7번 탭을 출력으로 사용한다. 이 선택된 G2의 출력과 G1의 고정된 출력(10번 탭)을 XOR하여 최종 C/A 코드를 생성한다.60

**표 3: GPS 위성 PRN 번호별 G2 레지스터 탭 조합**

| PRN 번호 | G2 출력 탭 조합 | PRN 번호 | G2 출력 탭 조합 |
| -------- | --------------- | -------- | --------------- |
| 1        |                 | 17       |                 |
| 2        |                 | 18       |                 |
| 3        |                 | 19       |                 |
| 4        |                 | 20       |                 |
| 5        |                 | 21       |                 |
| 6        |                 | 22       |                 |
| 7        |                 | 23       |                 |
| 8        |                 | 24       |                 |
| 9        |                 | 25       |                 |
| 10       |                 | 26       |                 |
| 11       |                 | 27       |                 |
| 12       |                 | 28       |                 |
| 13       |                 | 29       |                 |
| 14       |                 | 30       |                 |
| 15       |                 | 31       |                 |
| 16       |                 | 32       |                 |

#### 4.2.2  BPSK 변조를 통한 L1 반송파 확산

생성된 C/A 코드(칩 속도 1.023 Mcps)는 50 bps의 느린 속도로 전송되는 항법 메시지(위성 궤도, 시간 정보 등)와 XOR 연산된다. 이 결과 신호를 이용해 1575.42 MHz의 L1 반송파를 BPSK(Binary Phase Shift Keying) 방식으로 변조한다.61 이 과정을 통해, 원래 수십 Hz에 불과했던 항법 메시지 신호의 대역폭이 C/A 코드의 대역폭인 약 2.046 MHz(Null-to-Null)로 광대역 확산된다.34

GPS 수신기는 내부적으로 모든 위성의 C/A 코드를 생성할 수 있다. 수신된 미약한 광대역 신호와 내부에서 생성한 각 위성의 C/A 코드를 상관(correlation) 연산하여, 특정 위성의 C/A 코드와 일치할 때 발생하는 뾰족한 자기상관 피크를 검출한다. 이 피크가 검출된 시간과 신호에 담긴 시간 정보를 비교하여 위성으로부터 신호가 도달하기까지 걸린 시간을 극도의 정밀도로 측정하고, 이를 거리로 환산하여 위치 계산에 사용한다.

### 4.3  무선 근거리 통신망 (WLAN: IEEE 802.11b)

IEEE 802.11b는 2.4 GHz ISM 대역에서 동작하는 초기 Wi-Fi 표준으로, DSSS를 채택하여 다중 경로와 간섭이 심한 실내 환경에서 데이터 전송의 신뢰성을 확보했다.2 802.11b는 시장의 요구에 부응하여 전송 속도를 높이기 위해 DSSS 기술을 단계적으로 발전시킨 좋은 예시를 보여준다.

#### 4.3.1  1 Mbps 및 2 Mbps 전송

초기 802.11 표준에서 정의된 1 Mbps 및 2 Mbps의 기본 전송 속도는 강인성(robustness)에 초점을 맞추었다.

- **변조 및 확산:** 1 Mbps에서는 DBPSK(Differential BPSK), 2 Mbps에서는 DQPSK(Differential QPSK)로 데이터를 변조한다. 변조된 각 데이터 심볼은 11-chip 길이의 바커 부호(`+1, -1, +1, +1, -1, +1, +1, +1, -1, -1, -1`)와 곱해져 확산된다.18 칩 속도는 11 Mcps이므로, 처리 이득은 1 Mbps에서 11(10.4 dB), 2 Mbps에서 5.5(7.4 dB)가 된다.
- **바커 부호의 역할:** 바커 부호는 비주기 자기상관 특성이 매우 우수하여, 부엽(sidelobe)의 크기가 작다. 이는 수신기가 패킷의 시작을 명확하게 감지하고, 다중 경로로 인해 지연된 신호의 영향을 최소화하여 안정적인 동기를 획득하는 데 큰 도움을 준다.29

#### 4.3.2  5.5 Mbps 및 11 Mbps 전송

더 높은 데이터 전송률에 대한 요구가 커지면서, 802.11b 표준은 처리 이득을 다소 희생하는 대신 심볼 당 더 많은 정보 비트를 전송하는 상보 부호 키잉(Complementary Code Keying, CCK) 방식을 도입했다.18

- **CCK 원리:** CCK는 8-chip 길이의 상보적인 특성을 갖는 코드 집합을 사용한다. 바커 부호처럼 고정된 하나의 코드로 확산하는 대신, 전송할 데이터 비트 값 자체를 이용하여 64개의 코드워드(codeword) 중 하나를 동적으로 선택하여 변조와 확산을 동시에 수행한다.63
- **11 Mbps 모드:** 8개의 정보 비트를 하나의 CCK 심볼로 처리한다. 이 중 6비트는 64개의 코드워드 중 하나를 선택하는 데 사용되고, 나머지 2비트는 전체 심볼의 위상을 결정하는 DQPSK 변조에 사용된다. 결과적으로 하나의 8-chip 심볼에 8비트의 정보가 실리게 되어 $11 \text{ Mcps} / 8 \text{ chips/symbol} \times 8 \text{ bits/symbol} = 11 \text{ Mbps}`의 전송률을 달성한다.
- **5.5 Mbps 모드:** 4개의 정보 비트를 하나의 CCK 심볼로 처리한다. 2비트는 4개의 코드워드 중 하나를 선택하고, 나머지 2비트는 DQPSK 위상을 결정한다. 이로써 $11 \text{ Mcps} / 8 \text{ chips/symbol} \times 4 \text{ bits/symbol} = 5.5 \text{ Mbps}`의 전송률을 달성한다.63

이처럼 802.11b는 동일한 DSSS 프레임워크 내에서 확산 코드를 바커 부호에서 CCK로 전환함으로써, 시스템의 강인성과 데이터 처리량 사이의 균형점을 시장의 요구에 맞게 조절하는 기술적 진화를 보여주었다.

**표 4: IEEE 802.11b 전송 속도별 변조 및 확산 방식**

| 데이터 속도 | 변조 방식 | 확산 방식 | 확산 부호            | 칩 속도 | 처리 이득 (PG) |
| ----------- | --------- | --------- | -------------------- | ------- | -------------- |
| 1 Mbps      | DBPSK     | DSSS      | 11-chip Barker Code  | 11 Mcps | 11 (10.4 dB)   |
| 2 Mbps      | DQPSK     | DSSS      | 11-chip Barker Code  | 11 Mcps | 5.5 (7.4 dB)   |
| 5.5 Mbps    | DQPSK     | CCK       | 8-chip CCK Codewords | 11 Mcps | 2 (3 dB)       |
| 11 Mbps     | DQPSK     | CCK       | 8-chip CCK Codewords | 11 Mcps | 1 (0 dB)       |

5.4. 저전력 단거리 통신 (Zigbee: IEEE 802.15.4)

Zigbee, Thread 등 저전력, 저비용, 저속 통신을 목표로 하는 사물 인터넷(IoT) 프로토콜은 IEEE 802.15.4 물리 계층(PHY) 표준을 기반으로 한다. 이 표준의 2.4 GHz 대역 PHY는 DSSS 기술을 채택하여, 간섭이 많은 환경에서도 신뢰성 있는 통신을 저전력으로 구현하는 데 중점을 둔다.47

- **2.4 GHz 대역 PHY 명세:** 전 세계적으로 사용 가능한 2.4 GHz ISM 대역에서 250 kbps의 고정 데이터 전송률을 제공한다.67 Zigbee의 DSSS 구현 방식은 데이터 처리량보다는 신뢰성과 전력 효율에 최적화되어 있다.
  1. **데이터-심볼 매핑:** 먼저, 4개의 데이터 비트(a nibble)를 하나의 데이터 심볼로 매핑한다. 4비트는 총 16가지($2^4$)의 경우의 수를 가지므로, 16개의 심볼 중 하나가 선택된다.
  2. **심볼-칩 확산:** 선택된 각 심볼은 32-chip 길이의 고유한 PN 시퀀스로 대체(확산)된다.47 즉, 4비트의 정보가 32개의 칩으로 표현되므로, 비트당 8개의 칩이 할당되는 셈이다. 따라서 처리 이득은 8, 즉 9 dB가 된다.
  3. **변조 방식:** 최종적으로 32-chip PN 시퀀스는 O-QPSK(Offset QPSK)를 사용하여 변조된다.68 앞서 CDMA 역방향 링크에서 언급했듯이, O-QPSK는 신호 파형의 진폭 변동을 최소화하여 전력 증폭기의 선형성 요구사항을 낮추고 전력 효율을 높일 수 있다. 이는 배터리 수명이 기기의 가치를 결정하는 저전력 IoT 장치에 매우 적합한 선택이다.70

이처럼 Zigbee는 DSSS의 간섭 저항성과 신뢰성이라는 장점을 취하면서도, O-QPSK 변조와 같은 저전력 기술을 결합하여 IoT 환경의 특수한 요구사항을 만족시키는 맞춤형 설계를 보여준다.

## 5.  결론: DSSS 기술의 현재와 미래

### 6.1. DSSS 기술의 핵심 가치 요약

직접 시퀀스 대역 확산(DSSS) 기술은 지난 수십 년간 무선 통신 기술의 발전에 지대한 공헌을 해왔다. 이는 단순히 신호의 대역폭을 넓히는 변조 기법을 넘어, 통신 시스템의 강인성, 보안성, 그리고 다중 접속 효율성을 근본적으로 향상시키는 핵심적인 패러다임으로 자리 잡았다.

DSSS의 본질적 가치는 '결정론적 무작위성'을 지닌 의사 잡음(PN) 부호를 사용하는 데 있다. 이 PN 부호라는 '열쇠'를 공유하는 합법적인 송수신자에게는 명확하고 신뢰성 있는 통신 채널을 제공하는 반면, 이 열쇠가 없는 외부의 간섭, 재밍, 도청 시도자에게는 의미 없는 광대역 잡음만을 전달한다. 이러한 차별적 특성은 군사 통신의 보안 요구에서 출발하여, 상업 통신의 간섭 문제 해결과 주파수 공유 효율성 증대라는 과제로 성공적으로 확장되었다.

또한, DSSS는 '처리 이득(Processing Gain)'이라는 명확한 정량적 지표를 통해 통신 시스템 설계의 근본적인 트레이드오프, 즉 성능(신뢰성, 강인성)과 자원(대역폭, 복잡성) 간의 관계를 체계적으로 다룰 수 있는 강력한 이론적 프레임워크를 제공했다. 처리 이득과 재밍 마진의 관계는 시스템이 외부 위협에 얼마나 견딜 수 있는지를 예측하고 설계할 수 있게 해주었으며, 이는 오늘날 GPS와 같은 필수적인 사회 기반 시설의 안정적인 운영을 가능하게 하는 기반이 되었다.

### 5.1  타 기술(OFDM 등)과의 융합 및 향후 발전 방향 전망

현대의 고속 광대역 무선 통신 시스템, 예를 들어 4G LTE, 5G, 그리고 최신 Wi-Fi(802.11n/ac/ax) 표준에서는 DSSS보다 주파수 이용 효율이 높고 고속 데이터 전송 시 발생하는 다중 경로 문제를 주파수 영역에서 보다 유연하게 해결할 수 있는 직교 주파수 분할 다중(OFDM) 기술이 주류로 채택되었다.71 OFDM은 다수의 직교 부반송파를 사용하여 데이터를 병렬로 전송함으로써, DSSS가 직면했던 복잡한 시간 영역 등화기나 RAKE 수신기의 필요성을 줄이고 스펙트럼 효율을 극대화했다.

그러나 이는 DSSS 기술의 종말을 의미하지 않는다. 오히려 DSSS는 자신의 핵심 가치가 가장 잘 발휘될 수 있는 영역에서 여전히 대체 불가능한 기술로 확고히 자리 잡고 있다. GPS와 같은 위성 항법 시스템, 높은 보안성과 항재밍 성능이 요구되는 군사 및 특수 목적 통신, 그리고 Zigbee와 같이 저전력과 신뢰성이 핵심인 IoT 분야에서는 DSSS의 원리가 여전히 핵심적인 역할을 수행하고 있다.

향후 DSSS 기술의 발전은 단독 기술로서의 진화보다는 다른 기술과의 융합을 통해 새로운 가치를 창출하는 방향으로 전개될 가능성이 높다. 예를 들어, OFDM의 높은 주파수 효율과 DSSS의 우수한 다중 접속 및 간섭 저항 능력을 결합한 MC-CDMA(Multi-Carrier CDMA)와 같은 하이브리드 기술은 두 기술의 장점을 모두 취하려는 시도이다.48 또한, 최근 급격히 발전하고 있는 인공지능 및 머신러닝 기술을 DSSS 시스템에 접목하는 연구도 활발히 진행되고 있다. 머신러닝 기반의 지능형 간섭 탐지 및 제거 알고리즘을 통해 기존의 필터링 방식보다 훨씬 더 복잡하고 동적인 간섭 환경에 효과적으로 대응하거나 73, 시스템 환경에 최적화된 새로운 PN 부호를 동적으로 설계하는 등의 연구는 DSSS의 성능을 한 단계 더 끌어올릴 잠재력을 가지고 있다.

결론적으로, DSSS는 그 자체로 완성된 과거의 기술이 아니라, 새로운 통신 환경의 요구와 신기술의 등장에 맞춰 끊임없이 진화하고 융합될 수 있는 근본적인 기술 요소(fundamental building block)로서 그 생명력을 이어갈 것이다. DSSS가 제공하는 강인성과 보안성이라는 핵심 가치는 앞으로 다가올 초연결 시대의 통신 인프라를 더욱 견고하고 신뢰성 있게 만드는 데 지속적으로 기여할 것으로 전망된다.

#### **참고 자료**

1. DSSS(Direct Sequence Spread Spectrum)란?? - RF열무의 라이프 스터디 블로그 - 티스토리, 8월 5, 2025에 액세스, https://rf-yeolmu.tistory.com/31
2. 직접 확산 방식(Direct Sequence Spread Spectrum, DSSS) - velog, 8월 5, 2025에 액세스, [https://velog.io/@agnusdei1207/%EC%A7%81%EC%A0%91-%ED%99%95%EC%82%B0-%EB%B0%A9%EC%8B%9DDirect-Sequence-Spread-Spectrum-DSSS](https://velog.io/@agnusdei1207/직접-확산-방식Direct-Sequence-Spread-Spectrum-DSSS)
3. 무선 LAN의 기술적 고찰, 8월 5, 2025에 액세스, http://ebook.pldworld.com/_eBook/-Telecommunications,Networks-/NETWORK_DOCUMENTs/semina/my/wlan/wlan.htm
4. 순환 주파수 추정을 통한 직접 시퀀스 대역 확산 신호 탐지 알고리즘 복잡도 저감, 8월 5, 2025에 액세스, https://ki-it.com/xml/32013/32013.pdf
5. [논문]직접대역확산방식 기술조사 및 분석, 8월 5, 2025에 액세스, https://scienceon.kisti.re.kr/srch/selectPORSrchArticle.do?cn=JAKO201164359117887
6. seo.goover.ai, 8월 5, 2025에 액세스, [https://seo.goover.ai/report/202501/go-public-report-ko-73fdb46f-acc9-462e-87a3-8cc194e45e63-0-0.html#:~:text=DSSS%EC%9D%98%20%EA%B8%B0%EB%B3%B8%20%EC%9B%90%EB%A6%AC%EB%8A%94,%EC%9D%98%20%EC%8B%A0%ED%98%B8%EB%A1%9C%20%EB%B3%B5%EA%B5%AC%EB%90%A9%EB%8B%88%EB%8B%A4.](https://seo.goover.ai/report/202501/go-public-report-ko-73fdb46f-acc9-462e-87a3-8cc194e45e63-0-0.html#:~:text=DSSS의 기본 원리는,의 신호로 복구됩니다.)
7. DSSS (Direct Sequence Spread Spectrum) - ITPE * JackerLab, 8월 5, 2025에 액세스, https://itpe.jackerlab.com/m/entry/DSSS-Direct-Sequence-Spread-Spectrum
8. 직접 대역확산 통신 시스템의 직교 특성과 확산 코드의 중요성 - Goover, 8월 5, 2025에 액세스, https://seo.goover.ai/report/202501/go-public-report-ko-73fdb46f-acc9-462e-87a3-8cc194e45e63-0-0.html
9. 직접 시퀀스 확산 스펙트럼(DSSS) 통신 - 통신 원리, 8월 5, 2025에 액세스, https://www.hdv-fiber.com/ko/news/direct-sequence-spread-spectrum-dsss-communication-communication-principle/
10. 대역 확산 통신 방식, 8월 5, 2025에 액세스, https://koreascience.kr/article/JAKO199111920663294.pdf
11. Direct-sequence spread spectrum - Wikipedia, 8월 5, 2025에 액세스, https://en.wikipedia.org/wiki/Direct-sequence_spread_spectrum
12. An Introduction to Spread-Spectrum Communications | Analog Devices, 8월 5, 2025에 액세스, https://www.analog.com/en/resources/technical-articles/introduction-to-spreadspectrum-communications--maxim-integrated.html
13. 직접 시퀀스 확산 스펙트럼 - 위키백과, 우리 모두의 백과사전, 8월 5, 2025에 액세스, [https://ko.wikipedia.org/wiki/%EC%A7%81%EC%A0%91_%EC%8B%9C%ED%80%80%EC%8A%A4_%ED%99%95%EC%82%B0_%EC%8A%A4%ED%8E%99%ED%8A%B8%EB%9F%BC](https://ko.wikipedia.org/wiki/직접_시퀀스_확산_스펙트럼)
14. Unlocking DSSS: The Ultimate Guide - Number Analytics, 8월 5, 2025에 액세스, https://www.numberanalytics.com/blog/ultimate-guide-to-dsss
15. Direct Sequence Spread Spectrum (DSSS) Digital ... - SunCam, 8월 5, 2025에 액세스, https://www.suncam.com/miva/downloads/docs/079.pdf
16. Direct Sequence Spread Spectrum in Wireless Networks - GeeksforGeeks, 8월 5, 2025에 액세스, https://www.geeksforgeeks.org/ethical-hacking/direct-sequence-spread-spectrum-in-wireless-networks/
17. DSSS: The Ultimate Guide - Number Analytics, 8월 5, 2025에 액세스, https://www.numberanalytics.com/blog/ultimate-guide-dsss-data-communications
18. 802.11b White Paper - VOCAL Technologies, 8월 5, 2025에 액세스, https://www.vocal.com/wp-content/uploads/2012/05/802.11b_wp1.pdf
19. Spread Spectrum (SS), 8월 5, 2025에 액세스, https://www.ee.columbia.edu/~ywang/MSS/HW2/ss_intro.pdf
20. UNIT V- SPREAD SPECTRUM MODULATION Introduction - Sathyabama, 8월 5, 2025에 액세스, https://www.sathyabama.ac.in/sites/default/files/course-material/2020-10/UNIT5_1.pdf
21. Processing Gain for Direct Sequence Spread Spectrum ... - QSL.net, 8월 5, 2025에 액세스, https://www.qsl.net/n9zia/AN9633.pdf
22. [GPS] GPS 이론: GNSS, RTK 등 - velog, 8월 5, 2025에 액세스, [https://velog.io/@717lumos/GPS-GPS-%EC%9D%B4%EB%A1%A0-GNSS-RTK-%EB%93%B1](https://velog.io/@717lumos/GPS-GPS-이론-GNSS-RTK-등)
23. Maximum length sequence - Wikipedia, 8월 5, 2025에 액세스, https://en.wikipedia.org/wiki/Maximum_length_sequence
24. M-Sequences with Pairwise-Prime Periods - Michigan State University, 8월 5, 2025에 액세스, https://www.egr.msu.edu/~tongli/teaching/ece802/Papers/Globecom04.pdf
25. CDMA 이동통신을 위한 PN 코드의 성능분석, 8월 5, 2025에 액세스, https://ettrends.etri.re.kr/ettrends/24/0905001092/HJTOCM_1992_v7n2_32.pdf
26. Sebuah Kajian Pustaka: - Indonesian Journal of Electrical Engineering and Computer Science, 8월 5, 2025에 액세스, https://ijeecs.iaescore.com/index.php/IJEECS/article/viewFile/21562/14118
27. Correlation Functions of m−Sequences of Different Lengths - International Journal of Network Security, 8월 5, 2025에 액세스, http://ijns.jalaxy.com.tw/contents/ijns-v22-n1/ijns-2020-v22-n1-p171-176.pdf
28. Autocorrelation - Wikipedia, 8월 5, 2025에 액세스, https://en.wikipedia.org/wiki/Autocorrelation
29. Barker code - Wikipedia, 8월 5, 2025에 액세스, https://en.wikipedia.org/wiki/Barker_code
30. Code selection, 8월 5, 2025에 액세스, http://www.wu.ece.ufl.edu/books/EE/communications/CDMA/CDMA.htm
31. US6345073B1 - Convolutional despreading method for rapid code phase determination of chipping codes of spread spectrum systems - Google Patents, 8월 5, 2025에 액세스, https://patents.google.com/patent/US6345073B1/en
32. Gold code - Wikipedia, 8월 5, 2025에 액세스, https://en.wikipedia.org/wiki/Gold_code
33. Gold Code Generators:. Demystifying Pseudorandom Sequences | by Dasika Sri Bhuvana Vaishnavi | Medium, 8월 5, 2025에 액세스, https://medium.com/@dasikavaishnavi/gold-code-generators-d784e2b87984
34. EP1808967A1 - A method for code alignment for DSSS signal processing - Google Patents, 8월 5, 2025에 액세스, https://patents.google.com/patent/EP1808967A1/en
35. SPS Gold Code Generation and Implementation for IRNSS User Receiver - IJERA, 8월 5, 2025에 액세스, https://www.ijera.com/papers/Vol7_issue5/Part-3/G0705034853.pdf
36. Gold code | Error Correction Zoo, 8월 5, 2025에 액세스, https://errorcorrectionzoo.org/c/gold
37. PN Code - [정보통신기술용어해설], 8월 5, 2025에 액세스, http://www.ktword.co.kr/test/view/view.php?no=2609
38. 확산이득(Processing Gain) - RFDH, 8월 5, 2025에 액세스, http://www.rfdh.com/tech/cdma/cdma_spread_gain.htm
39. Direct Sequence Spread Spectrum - DTIC, 8월 5, 2025에 액세스, https://apps.dtic.mil/sti/tr/pdf/ADA387425.pdf
40. Performance Analysis of DSSS and FHSS under Various Jamming Scenarios - Swami Vivekananda University, 8월 5, 2025에 액세스, [https://www.swamivivekanandauniversity.ac.in/jiaef/img/pdf/5.%20Neelakshi_paper%202.pdf](https://www.swamivivekanandauniversity.ac.in/jiaef/img/pdf/5. Neelakshi_paper 2.pdf)
41. Part 4 Spread-Spectrum Modulation, 8월 5, 2025에 액세스, https://video.ocw.nycu.edu.tw/pub/idc082/Handout/q4-2020.pdf
42. DSSS Jamming Margin - EEWeb, 8월 5, 2025에 액세스, https://www.eeweb.com/dsss-jamming-margin/
43. Jamming resistance in spread spectrum systems - Signal Processing Stack Exchange, 8월 5, 2025에 액세스, https://dsp.stackexchange.com/questions/64196/jamming-resistance-in-spread-spectrum-systems
44. [텀즈] DSSS (direct sequence spread spectrum), 8월 5, 2025에 액세스, http://www.terms.co.kr/DSSS.htm
45. 전력제어 - RFDH, 8월 5, 2025에 액세스, http://www.rfdh.com/tech/cdma/cdma_power.htm
46. Unlocking DSSS: The Ultimate Guide - Number Analytics, 8월 5, 2025에 액세스, https://www.numberanalytics.com/blog/ultimate-guide-to-dsss-in-signal-processing
47. DSSS & FHSS - 대학원부터 시작한 첫번째 블로그 - 티스토리, 8월 5, 2025에 액세스, https://bluebamus.tistory.com/entry/DSSS-FHSS
48. FH - [정보통신기술용어해설], 8월 5, 2025에 액세스, http://www.ktword.co.kr/test/view/view.php?no=1573
49. [네트워크 용어] DSSS, FHSS - 공부 노트 - 티스토리, 8월 5, 2025에 액세스, https://ncookie.tistory.com/6
50. UNIT III CDMA CHANNELS, 8월 5, 2025에 액세스, [https://cs.uok.edu.in/Files/79755f07-9550-4aeb-bd6f-5d802d56b46d/Custom/uniit%203.pdf](https://cs.uok.edu.in/Files/79755f07-9550-4aeb-bd6f-5d802d56b46d/Custom/uniit 3.pdf)
51. CDMA Basics Is-95 Forward & Reverse Channel | PDF | Code Division Multiple Access | Bit Rate - Scribd, 8월 5, 2025에 액세스, https://www.scribd.com/document/131926305/CDMA-Basics-is-95-Forward-Reverse-Channel
52. CDMA Techniques - Tutorialspoint, 8월 5, 2025에 액세스, https://www.tutorialspoint.com/cdma/cdma_techniques.htm
53. chapter 8 - cdma technology, is-95, and imt-2000 - CWINS, 8월 5, 2025에 액세스, http://www.cwins.wpi.edu/publications/pown/chapter_8.pdf
54. IS95: CDMA Cellular Telephony, Forward channel - Wireless Communication, 8월 5, 2025에 액세스, http://www.wirelesscommunication.nl/reference/chaptr01/telephon/is95/is95fwd.htm
55. IS-95 Channels: cdmaOne Data Channels - Electronics Notes, 8월 5, 2025에 액세스, https://www.electronics-notes.com/articles/connectivity/cdmaone-cdma2000/is95-data-channels.php
56. Walsh 코드와 PN코드 - RFDH, 8월 5, 2025에 액세스, http://www.rfdh.com/tech/cdma/cdma_walsh.htm
57. IS-95 Air Interface: Forward & Reverse Links - Electronics Notes, 8월 5, 2025에 액세스, https://www.electronics-notes.com/articles/connectivity/cdmaone-cdma2000/is95-air-radio-access-interface.php
58. A simple model for GPS C/A-code self-interference - The Institute of Navigation, 8월 5, 2025에 액세스, https://navi.ion.org/content/67/2/319
59. The C/A Code | GEOG 862: GPS and GNSS for Geospatial Professionals - Dutton Institute, 8월 5, 2025에 액세스, https://www.e-education.psu.edu/geog862/node/1742
60. The GPS PRN (Gold Codes) - natronics, 8월 5, 2025에 액세스, https://natronics.github.io/blag/2014/gps-prn/
61. Spread Spectrum and Code Modulation of L1 GPS Carrier | GEOG ..., 8월 5, 2025에 액세스, https://www.e-education.psu.edu/geog862/node/1753
62. Wi-Fi: Overview of the 802.11 Physical Layer and Transmitter Measurements | Tektronix, 8월 5, 2025에 액세스, https://www.tek.com/en/documents/primer/wi-fi-overview-80211-physical-layer-and-transmitter-measurements
63. A Brief Tutorial on the PHY and MAC layers of the IEEE 802.11b Standard - eirp.org, 8월 5, 2025에 액세스, http://www.eirp.org/webtut.pdf
64. Redalyc.Design of a digital modulator and demodulator for reader-less RFID Tag in 0.18 Mm CMOS process, 8월 5, 2025에 액세스, https://www.redalyc.org/pdf/3032/303229362008.pdf
65. Complementary Code Keying (CCK) - Tutorialspoint, 8월 5, 2025에 액세스, https://www.tutorialspoint.com/complementary-code-keying-cck
66. CCK vs DSSS vs OFDM: Modulation Techniques Compared | RF Wireless World, 8월 5, 2025에 액세스, https://www.rfwireless-world.com/terminology/cck-vs-dsss-vs-ofdm
67. IEEE 802.15.4 - Wikipedia, 8월 5, 2025에 액세스, https://en.wikipedia.org/wiki/IEEE_802.15.4
68. IEEE 802.15.4 ZigBee - Keysight, 8월 5, 2025에 액세스, [https://helpfiles.keysight.com/csg/n7610b/Content/Main/IEEE%20802.15.4%20ZigBee.htm](https://helpfiles.keysight.com/csg/n7610b/Content/Main/IEEE 802.15.4 ZigBee.htm)
69. What is the modulation technique of Zigbee? (Zigbee Module Accessories Modules) - Vesternet, 8월 5, 2025에 액세스, https://www.vesternet.com/a/answers/5689162/What-is-the-modulation-technique-of-Zigbee
70. The ZigBee Alliance has been developing a standard-based wireless network platform aimed at, 8월 5, 2025에 액세스, [http://nice.kaist.ac.kr/files/attach/filebox/711/003/International%20Journals/1578587.pdf](http://nice.kaist.ac.kr/files/attach/filebox/711/003/International Journals/1578587.pdf)
71. SOFTWARE IMPLEMENTATION OF IEEE 802.11B WIRELESS LAN STANDARD Suyog D. Deshpande (Sr. MTS, 8월 5, 2025에 액세스, https://www.wirelessinnovation.org/assets/Proceedings/2004/2004-sdr04-4-1-2-deshpande.pdf
72. IEEE 802.11 - Wikipedia, 8월 5, 2025에 액세스, https://en.wikipedia.org/wiki/IEEE_802.11
73. The Ultimate Guide to Interference Cancellation, 8월 5, 2025에 액세스, https://www.numberanalytics.com/blog/ultimate-guide-interference-cancellation
74. Interference suppression in DSSS(Direct sequence spread spectrum) using kalman filter - MATLAB Answers - MathWorks, 8월 5, 2025에 액세스, https://www.mathworks.com/matlabcentral/answers/1910200-interference-suppression-in-dsss-direct-sequence-spread-spectrum-using-kalman-filter

