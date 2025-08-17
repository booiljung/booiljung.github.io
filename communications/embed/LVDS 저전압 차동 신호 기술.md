# 저전압 차동 신호(LVDS) 기술

## 서론

현대 전자 시스템은 기하급수적으로 증가하는 데이터를 처리하고 전송해야 하는 본질적인 과제에 직면해 있다. 인공지능, 고해상도 디스플레이, 자율주행 기술의 발전은 시스템 내부 및 시스템 간의 데이터 처리량을 한계까지 밀어붙이고 있다. 이러한 고속 데이터 전송 요구는 필연적으로 신호 무결성(Signal Integrity) 문제를 야기하며, 동시에 모바일 기기 및 고밀도 집적 회로 환경의 확산은 저전력 소모와 전자파 장해(EMI, Electromagnetic Interference) 억제를 시스템의 성능과 안정성을 좌우하는 핵심 설계 제약 조건으로 부상시켰다.1 고속화와 저전력/저잡음이라는 상충되어 보이는 두 가지 요구사항을 동시에 만족시키는 것은 오랫동안 전자공학 분야의 난제였다.

이러한 기술적 난제를 해결하기 위한 혁신적인 솔루션으로 1994년, 미국 통신 산업 협회(TIA)와 전자 산업 연맹(EIA)에 의해 ANSI/TIA/EIA-644 표준으로 공식화된 저전압 차동 신호(LVDS, Low-Voltage Differential Signaling) 기술이 등장했다.2 LVDS는 매우 낮은 전압 스윙(swing)을 사용하는 차동 신호 전송 방식을 채택함으로써, 저렴한 꼬임쌍선(twisted-pair) 구리 케이블을 통해서도 수백 Mbps를 넘어 수 Gbps에 이르는 고속 데이터 전송을 지극히 낮은 전력 소모와 최소한의 EMI 방출로 구현해냈다. 지난 수십 년간 LVDS는 노트북 컴퓨터의 디스플레이 인터페이스를 시작으로 디지털 TV, 통신 시스템, 서버, 산업용 제어 장비, 그리고 자동차 전장 시스템에 이르기까지 실로 광범위한 분야에서 핵심적인 물리 계층(Physical Layer) 기술로 확고히 자리매김했다.1

본 보고서는 이처럼 현대 전자 기술의 발전에 지대한 공헌을 한 LVDS 기술의 모든 측면을 심층적으로 고찰하는 것을 목표로 한다. 제1장에서는 LVDS의 근간을 이루는 차동 신호와 전류 모드 구동의 기본 원리를 설명한다. 제2장에서는 TIA/EIA-644 표준에 명시된 상세한 전기적 특성을 분석하고, 제3장에서는 실제 반도체 회로 수준에서의 드라이버와 수신기 구현 방식을 탐구한다. 제4장에서는 LVDS가 갖는 명확한 장점들과 기술적 한계를 분석하고, 제5장에서는 다른 주요 고속 인터페이스 기술과의 비교를 통해 LVDS의 기술적 위상을 조명한다. 제6장에서는 실제 시스템 설계 시 반드시 고려해야 할 PCB 레이아웃 가이드라인을 제시하며, 제7장에서는 주요 응용 분야와 미래 발전 동향을 전망한다. 마지막으로 결론에서는 LVDS 기술의 핵심 가치와 기술사적 의의를 종합적으로 정리하며 보고서를 마무리한다.

## 1.  LVDS의 기본 원리

### 1.1  차동 신호 전송(Differential Signaling)의 개념

LVDS 기술의 본질을 이해하기 위해서는 그 근간을 이루는 차동 신호 전송 방식에 대한 이해가 선행되어야 한다. 이는 기존의 단일 종단 신호 방식이 가졌던 근본적인 한계를 극복하기 위해 고안된 매우 효과적인 해결책이다.

#### 1.1.1 단일 종단(Single-Ended) 방식의 한계

전통적인 디지털 신호 전송 방식인 단일 종단 신호(Single-Ended Signaling)는 하나의 신호선과 모든 신호가 공유하는 공통 접지(common ground)를 기준으로 전압 레벨을 정의하여 '0'과 '1'을 구분한다. 대표적으로 TTL(Transistor-Transistor Logic)이나 CMOS(Complementary Metal-Oxide-Semiconductor) 로직이 이 방식을 사용한다. 단일 종단 방식은 회로 구현이 간단하고 비용이 저렴하다는 장점이 있지만, 데이터 전송 속도가 빨라지고 전송 거리가 길어질수록 여러 가지 심각한 문제에 직면하게 된다.5

가장 큰 문제는 노이즈에 매우 취약하다는 점이다. 신호선 주변에서 발생하는 전자기적 노이즈나 전원 공급단의 노이즈가 신호선에 유입되면, 수신단에서는 원래의 신호와 노이즈를 분리할 방법이 없다. 특히 고속으로 신호가 스위칭될 때 발생하는 접지 바운스(Ground Bounce) 현상은 송신단과 수신단의 접지 전위를 다르게 만들어 신호의 기준 레벨 자체를 흔들어 버린다. 이로 인해 데이터 전송 오류가 발생할 확률이 급격히 높아지며, 전송 거리가 길어질수록 신호 감쇠와 왜곡이 누적되어 신뢰성 있는 데이터 복원이 거의 불가능해진다.5

#### 1.1.2 차동 신호의 우월성

차동 신호(Differential Signaling) 방식은 이러한 단일 종단 방식의 한계를 극복하기 위해 두 개의 상보적인(complementary) 신호선을 사용한다. 즉, 하나의 데이터를 전송하기 위해 정위상(non-inverting, +) 신호와 이와 정확히 반대 위상을 갖는 역위상(inverting, -) 신호를 동시에 전송한다.5 수신단에서는 두 신호선 각각의 절대 전압을 측정하는 것이 아니라, 두 신호선 간의 '전압 차이'를 감지하여 논리 상태를 판단한다.

이 방식의 가장 큰 장점은 공통 모드 노이즈(Common-Mode Noise) 제거 능력에 있다. 전송 과정에서 외부로부터 유입되는 노이즈는 물리적으로 인접한 두 신호선에 거의 동일한 크기와 위상으로 영향을 미치게 된다. 이를 공통 모드 노이즈라고 한다. 수신단에서는 두 신호의 차이를 계산하므로, 양쪽에 공통으로 더해진 노이즈 성분은 수학적으로 완벽하게 상쇄되어 제거된다.1

#### 1.1.3 공통 모드 노이즈 제거 원리

이 원리를 수식으로 간단히 표현할 수 있다. 송신단에서 전송한 정위상 신호를 $V_P$, 역위상 신호를 $V_N$이라 하고, 전송 경로에서 유입된 공통 모드 노이즈를 $V_{noise}$라고 가정하자. 수신단에 도달하는 각 신호는 $V_{P,rx} = V_P + V_{noise}$ 와 $V_{N,rx} = V_N + V_{noise}$ 가 된다. 차동 수신기는 이 두 신호의 차이를 증폭하므로, 수신기가 인식하는 최종 신호 $V_{diff}$는 다음과 같다.
$$
V_{diff} = V_{P,rx} - V_{N,rx} = (V_P + V_{noise}) - (V_N + V_{noise}) = V_P - V_N
$$
결과적으로 노이즈 성분인 $V_{noise}$는 완벽하게 제거되고, 송신단에서 보낸 순수한 차동 신호($V_P - V_N$)만이 남게 된다. 이 원리 덕분에 LVDS는 노이즈가 심한 환경에서도 매우 높은 신호 품질을 유지할 수 있으며, 이는 LVDS 기술의 가장 핵심적인 우월성 중 하나이다.6

### 1.2  LVDS의 정의와 역사

LVDS(Low-Voltage Differential Signaling)는 앞서 설명한 차동 신호 전송 방식을 기반으로 하되, 매우 낮은 전압 스윙(Low-Voltage Swing)을 결합하여 고속, 저전력, 저잡음 특성을 극대화한 기술이다.1 이는 특정 통신 프로토콜이나 데이터 포맷을 정의하는 것이 아니라, 송신 드라이버(Driver)와 수신기(Receiver)의 전기적 특성만을 규정하는 물리 계층(Physical Layer) 표준이다.3 이 덕분에 LVDS는 다양한 상위 프로토콜(예: 비디오 데이터, 제어 신호 등)을 실어 나르는 범용 고속 데이터 전송 통로로 활용될 수 있다.

LVDS 기술은 1994년, TIA/EIA(Telecommunications Industry Association/Electronic Industries Alliance)에 의해 **ANSI/TIA/EIA-644**라는 공식 표준으로 제정되었다.2 이 표준의 등장은 당시의 기술적 요구에 대한 명확한 응답이었다. 1990년대 초반, ECL(Emitter-Coupled Logic)과 같은 기존의 고속 인터페이스들은 빠른 속도를 제공했지만 전력 소모가 매우 크고, 주로 음의 전원 전압을 사용하여 다른 로직 시스템과의 통합이 어려웠다.4 한편, 반도체 기술의 발전으로 로직 IC의 주 전원 전압은 5V에서 3.3V, 나아가 2.5V로 빠르게 낮아지고 있었다. LVDS는 바로 이러한 배경 속에서 ECL의 높은 전력 소모 문제를 해결하고, 3.3V 이하의 저전압 시스템과 완벽하게 호환되는 새로운 고속 인터페이스로 개발된 것이다.4

### 1.3  전류 모드 드라이버(Current-Mode Driver)의 동작 원리

LVDS가 다른 차동 신호 기술과 구별되는 또 하나의 핵심적인 특징은 바로 '전류 모드 드라이버(Current-Mode Driver)'를 사용한다는 점이다. 일반적인 전압 모드 드라이버가 직접적으로 HIGH 또는 LOW 전압을 출력하는 것과 달리, LVDS 드라이버는 일정한 크기의 전류를 출력하되 그 '방향'을 제어하는 방식으로 동작한다.1

LVDS 드라이버의 핵심은 약 $3.5\text{mA}$의 정전류원(constant current source)이다.1 드라이버는 논리 입력 값에 따라 내부 스위치를 조작하여 이 $3.5\text{mA}$의 전류가 차동 쌍(differential pair)의 어느 쪽으로 흐를지를 결정한다. 이 전류는 꼬임쌍선 케이블과 같은 전송 매체를 통해 수신단으로 전달되고, 수신단에 위치한 $100\Omega$의 종단 저항(termination resistor)을 통과하게 된다. 옴의 법칙($V = I \times R$)에 따라, 이 $100\Omega$ 저항 양단에는 약 $350\text{mV}$ ($3.5\text{mA} \times 100\Omega$)의 차동 전압($V_{OD}$)이 형성된다.1

논리 '1'을 전송할 때는 전류가 정위상(+) 라인으로 흘러 종단 저항을 거친 뒤 역위상(-) 라인을 통해 드라이버로 되돌아온다. 반대로 논리 '0'을 전송할 때는 전류의 방향이 반대가 되어 역위상(-) 라인으로 흘러가 정위상(+) 라인으로 되돌아온다.10 이처럼 드라이버, 전송선로, 종단 저항, 그리고 다시 드라이버로 이어지는 하나의 완전한 폐쇄 전류 루프(closed current loop)가 형성되는 것이 LVDS의 기본 동작 모델이다.13

이러한 전류 모드 구동 방식은 LVDS의 저전력, 저잡음 특성에 결정적인 기여를 한다. 드라이버는 항상 $3.5\text{mA}$의 정전류를 출력하고 단지 그 경로만 바꾸기 때문에, 신호가 고속으로 스위칭될 때에도 전원 공급 장치에서 끌어오는 전류량의 변화가 거의 없다. 이는 스위칭 시 큰 돌입 전류(inrush current)를 발생시켜 심각한 전원 노이즈와 EMI를 유발하는 CMOS 로직과 극명한 대조를 이룬다.3 결과적으로 LVDS는 동작 주파수가 증가해도 전력 소모가 거의 일정하게 유지되는 뛰어난 특성을 갖게 된다.14

LVDS의 성공은 단순히 차동 신호 방식을 채택했기 때문만은 아니다. 물론 차동 신호 방식은 외부에서 유입되는 노이즈를 효과적으로 상쇄하는 탁월한 메커니즘을 제공한다. 하지만 LVDS의 진정한 혁신성은 여기에 '전류 모드 구동'이라는 개념을 결합한 데 있다. 전류 모드 드라이버는 스위칭 시에도 전원 공급 라인에 거의 부담을 주지 않는 '정전류' 특성을 갖는다. 이는 시스템 내부에서 발생하는 노이즈의 근원 자체를 억제하는 역할을 한다. 즉, LVDS는 차동 신호를 통해 '외부 노이즈에 대한 방어력'을 높이는 동시에, 전류 모드 구동을 통해 '내부 노이즈 발생량'을 최소화하는 이중의 안전장치를 갖춘 셈이다. 이 두 가지 핵심 원리의 완벽한 시너지가 바로 LVDS를 수십 년간 고속 인터페이스 시장에서 독보적인 위치에 올려놓은 근본적인 동력이라 할 수 있다.

## 2.  LVDS의 전기적 특성 및 표준

LVDS 기술의 신뢰성과 상호 호환성은 TIA/EIA-644 표준에 명시된 엄격한 전기적 규격에 의해 보장된다. 이 장에서는 드라이버와 수신기의 핵심적인 전기적 파라미터들을 상세히 분석하고, 이들이 LVDS의 성능에 어떻게 기여하는지를 살펴본다.

### 2.1  TIA/EIA-644 표준 상세

TIA/EIA-644 표준은 LVDS 인터페이스를 구성하는 드라이버(송신단)와 수신기(수신단)가 만족해야 할 전기적 요구사항을 명확하게 정의한다.9

#### 2.1.1 드라이버 출력 특성 (Driver Output)

- **차동 출력 전압 ($V_{OD}$)**: 드라이버가 출력하는 두 신호선 간의 전압 차이를 의미한다. 표준은 $100\Omega$ 종단 저항의 양단에서 측정했을 때, 이 값의 절대값($|V_{OD}|$)이 $247\text{mV}$ 이상 $454\text{mV}$ 이하의 범위에 있어야 한다고 규정한다. 일반적으로 약 $350\text{mV}$의 값을 갖는다.1 이 낮은 전압 스윙은 LVDS의 저전력 및 저EMI 특성의 핵심이다.
- **공통 모드 오프셋 전압 ($V_{OS}$)**: 두 출력 신호 전압의 평균값으로, 신호의 직류(DC) 기준 레벨을 의미한다. 표준은 이 값이 접지(Ground)를 기준으로 $1.125\text{V}$ 이상 $1.375\text{V}$ 이하의 범위에 있도록 요구하며, 일반적인 값은 $1.2\text{V}$ 또는 $1.25\text{V}$이다.1

#### 2.1.2 수신기 입력 특성 (Receiver Input)

- **입력 감도 (Sensitivity)**: 수신기가 유효한 논리 상태로 인식할 수 있는 최소한의 차동 전압을 의미한다. 표준에 따르면 수신기는 $\pm100\text{mV}$의 차동 입력 전압($|V_{ID}|$)을 감지할 수 있어야 한다.9 이는 드라이버의 최소 출력 전압(

  $247\text{mV}$)과 비교했을 때 상당한 노이즈 마진(Noise Margin)을 제공함을 의미한다.

- **입력 전압 범위**: 수신기의 각 입력 핀이 손상 없이 받아들일 수 있는 절대 전압 범위를 말한다. 이 범위는 $0\text{V}$에서 $2.4\text{V}$까지로 규정되어 있다.1

- **공통 모드 전압 허용 범위**: 이는 LVDS의 강건함(robustness)을 보여주는 매우 중요한 지표이다. 표준은 드라이버와 수신기 사이에 최대 $\pm1\text{V}$의 접지 전위차(Ground Shift) 또는 공통 모드 노이즈가 존재하더라도 수신기가 정상적으로 동작해야 한다고 명시한다.9 예를 들어, 드라이버의 

  $V_{OS}$가 $1.2\text{V}$일 때, 여기에 $\pm1\text{V}$의 노이즈가 더해져도 수신기 입력 신호의 공통 모드 전압은 $0.2\text{V}$에서 $2.2\text{V}$ 사이가 된다. 이 값은 수신기의 입력 전압 범위($0\text{V}$ ~ $2.4\text{V}$) 내에 있으므로 안전하게 신호를 수신할 수 있다.1 일부 제조사는 이 허용 범위를 $-4\text{V}$에서 $5\text{V}$까지 확장한 고성능 수신기 칩을 제공하여 훨씬 더 열악한 노이즈 환경에 대응할 수 있도록 한다.9

### 2.2  신호 레벨 및 전력 소모 분석

LVDS의 전기적 특성은 고속 동작과 저전력 구현에 직접적으로 기여한다.

- **낮은 전압 스윙의 이점**: 약 $350\text{mV}$에 불과한 매우 낮은 차동 전압 스윙은 여러 가지 장점을 제공한다. 첫째, 신호의 논리 상태를 바꾸기 위해 필요한 전압 변화 폭이 작기 때문에 신호의 상승 시간(rise time)과 하강 시간(fall time)을 크게 단축시킬 수 있다. 이는 곧 더 높은 데이터 전송 속도를 달성할 수 있음을 의미한다.5 둘째, 신호 스위칭 시 전압 변화율(

  $dV/dt$)이 작아져 주변으로 방출되는 전자기파, 즉 EMI를 현저히 줄일 수 있다.14

- **전력 소모 계산**: LVDS 시스템의 주된 전력 소모는 드라이버가 전류를 흘려보내는 수신단의 $100\Omega$ 종단 저항에서 발생한다. 이 저항에서 소모되는 전력($P$)은 옴의 법칙과 전력 공식을 통해 간단히 계산할 수 있다.
  $$
  P = I^2 \times R = (3.5 \times 10^{-3} \text{A})^2 \times 100 \Omega = 0.00001225 \times 100 \text{ W} = 1.225 \text{ mW}
  $$
  이 계산 결과인 약 $1.2\text{mW}$는 다른 고속 인터페이스 기술과 비교했을 때 매우 낮은 수치이다. 예를 들어, 유사한 차동 신호 기술인 RS-422는 약 $90\text{mW}$의 전력을 소모하는데, 이는 LVDS보다 수십 배나 높은 값이다.1 이처럼 압도적인 저전력 특성 덕분에 LVDS는 발열에 민감한 고밀도 시스템이나 배터리 수명이 중요한 휴대용 기기에 이상적인 솔루션으로 각광받았다.19

### 2.3  100Ω 종단 저항(Termination Resistor)의 역할

LVDS 시스템에서 수신단에 위치하는 $100\Omega$ 종단 저항은 선택 사항이 아닌, 시스템이 올바르게 동작하기 위한 필수 구성 요소이다. 이 저항은 세 가지 핵심적인 역할을 수행한다.

1. **임피던스 매칭 (Impedance Matching)**: 고속 신호가 전송되는 PCB 트레이스나 케이블은 고유의 '특성 임피던스(Characteristic Impedance)'를 갖는다. LVDS 시스템에서는 이 특성 임피던스를 $100\Omega$에 가깝게 설계하는 것이 일반적이다.20 신호가 전송선로의 끝에 도달했을 때, 전송선로의 임피던스와 부하의 임피던스가 다르면 에너지의 일부가 반사되어 송신측으로 되돌아가는 '신호 반사(Signal Reflection)' 현상이 발생한다. 이 반사파는 원래의 신호와 중첩되어 심각한 왜곡을 일으키고, 이는 데이터 오류의 주된 원인이 된다. 수신단에 전송선로의 특성 임피던스와 동일한 

   $100\Omega$ 종단 저항을 배치하면, 신호 에너지가 저항에서 모두 소모되어 반사가 발생하지 않는다. 이를 임피던스 매칭이라 하며, 신호 무결성(Signal Integrity)을 유지하는 데 가장 중요한 요소이다.9

2. **차동 전압 생성**: LVDS 드라이버는 본질적으로 전류를 제어하는 '전류원'이다. 따라서 전류만으로는 수신기가 논리 상태를 인식할 수 없다. 이 $3.5\text{mA}$의 전류가 $100\Omega$ 종단 저항을 통과해야만 비로소 옴의 법칙에 따라 약 $350\text{mV}$의 차동 '전압'이 생성된다. 즉, 종단 저항은 전류 신호를 수신기가 감지할 수 있는 전압 신호로 변환하는 역할을 한다.1

3. **전류 루프 완성**: LVDS는 드라이버에서 나온 전류가 한쪽 신호선을 통해 종단 저항으로 흐르고, 다른 쪽 신호선을 통해 다시 드라이버로 되돌아오는 완전한 폐쇄 루프를 형성하여 동작한다. $100\Omega$ 종단 저항은 이 전류 루프의 반환점을 제공하여 회로를 완성시키는 필수적인 다리 역할을 한다.12

LVDS가 등장한 1990년대 중반은 반도체 산업에서 전원 전압이 $5\text{V}$에서 $3.3\text{V}$로, 그리고 $2.5\text{V}$로 빠르게 전환되던 중요한 시기였다. LVDS 표준에서 공통 모드 오프셋 전압($V_{OS}$)을 $1.2\text{V}$라는 비교적 낮은 값으로 설정한 것은 이러한 기술적 흐름을 완벽하게 고려한 전략적 결정이었다. $1.2\text{V}$의 $V_{OS}$를 중심으로 약 $\pm175\text{mV}$($350\text{mV}/2$)의 신호가 스윙하면, 각 신호선의 전압은 약 $1.025\text{V}$에서 $1.375\text{V}$ 사이의 매우 좁은 범위에 머무르게 된다.1 여기에 표준이 허용하는 최대 $\pm1\text{V}$의 공통 모드 노이즈가 더해지더라도, 수신기 입력 핀에 가해지는 실제 전압은 $0.025\text{V}$에서 $2.375\text{V}$ 사이가 된다. 이 범위는 수신기의 절대 입력 전압 범위인 $0\text{V}$ ~ $2.4\text{V}$를 벗어나지 않는다.9 이 설계 덕분에 LVDS는 $5\text{V}$ 시스템은 물론, 당시 최신 기술이었던 $3.3\text{V}$나 $2.5\text{V}$의 저전압 로직 IC와도 아무런 문제 없이 직접 연결될 수 있었다.3 결국 $1.2\text{V}$라는 $V_{OS}$ 값은 LVDS가 단순히 또 하나의 고속 인터페이스에 머무르지 않고, 서로 다른 전압 도메인을 갖는 복잡한 시스템들을 유연하게 연결하는 핵심 '브리지' 기술로 자리매김하게 만든 결정적인 한 수였다. 이는 노트북, 서버, 통신 장비 등 다양한 전압이 혼재하는 시스템에서의 채택을 폭발적으로 증가시키는 계기가 되었다.

## 3.  LVDS 회로 구현 및 분석

LVDS의 우수한 전기적 특성은 실제 반도체 칩 내부의 정교한 아날로그 회로 설계를 통해 구현된다. 이 장에서는 LVDS 드라이버와 수신기의 대표적인 회로 구조와 동작 원리를 분석하고, SerDes와의 결합이 가져오는 시너지 효과를 살펴본다.

### 3.1  LVDS 드라이버 회로 구조

LVDS 드라이버의 가장 보편적인 구현 방식은 정전류원과 스위칭 소자를 결합한 형태이다. 그중에서도 H-브리지 토폴로지가 널리 사용된다.

#### 3.1.1 H-브리지(H-Bridge) 토폴로지

이 구조는 4개의 MOSFET 트랜지스터(M1, M2, M3, M4)가 알파벳 'H'자 형태로 배치되고, 그 상단에 정전류원이 연결된 형태를 띤다.10 H-브리지의 양쪽 수직 다리 중간 지점에서 차동 출력($V_{out+}$, $V_{out-}$)이 나온다. 동작 원리는 다음과 같다 10:

- **논리 '1' 출력**: 입력 데이터 D가 '1'이면(따라서 D-bar는 '0'), 대각선 방향의 스위치 쌍인 M1과 M4가 켜지고, M2와 M3는 꺼진다. 이로 인해 정전류원에서 공급되는 전류는 M1을 거쳐 $V_{out+}$ 단자로 흘러나가고, 전송선로와 종단 저항을 거쳐 $V_{out-}$ 단자로 돌아온 뒤 M4를 통해 접지로 흐른다.
- **논리 '0' 출력**: 입력 데이터 D가 '0'이면, 반대쪽 대각선 쌍인 M2와 M3가 켜지고 M1과 M4는 꺼진다. 전류는 M2를 통해 $V_{out-}$ 단자로 흘러나가고, $V_{out+}$ 단자를 통해 돌아와 M3를 거쳐 접지로 흐른다.

이 H-브리지 구조는 비교적 적은 수의 트랜지스터로 전류의 방향을 정밀하게 제어할 수 있으며, 스위치가 꺼졌을 때 전류 누설이 적어 효율적이다.10

#### 3.1.2 공통 모드 피드백(CMFB) 회로

드라이버의 출력 공통 모드 전압($V_{OS}$)을 TIA/EIA-644 표준에서 요구하는 $1.2\text{V}$ 근처로 안정적으로 유지하는 것은 매우 중요하다. 만약 $V_{OS}$가 이 범위를 벗어나면 수신기가 신호를 제대로 인식하지 못할 수 있다. 공통 모드 피드백(Common-Mode Feedback, CMFB) 회로는 바로 이 역할을 수행하는 핵심적인 아날로그 회로 블록이다.10

CMFB 회로는 두 출력 단자($V_{out+}$, $V_{out-}$)의 평균 전압, 즉 실제 공통 모드 전압을 지속적으로 감지한다. 그리고 이 감지된 전압을 밴드갭 레퍼런스(Bandgap Reference) 등에서 생성된 정밀한 기준 전압($V_{REF}$, 약 $1.2\text{V}$)과 비교한다. 만약 실제 공통 모드 전압이 기준 전압보다 높으면, CMFB 회로는 피드백 신호를 생성하여 H-브리지 상단의 정전류원이 공급하는 전류량을 미세하게 줄인다. 반대로 실제 공통 모드 전압이 낮으면 전류량을 늘린다. 이러한 음성 피드백 루프(negative feedback loop)를 통해 $V_{OS}$는 외부의 전원 전압 변동, 온도 변화, 반도체 공정 편차(PVT, Process-Voltage-Temperature variation)에도 불구하고 항상 일정한 값으로 정밀하게 제어된다.10

### 3.2  LVDS 수신기 회로 구조

LVDS 수신기는 두 입력 신호 간의 미세한 전압 차이를 감지하여 표준 디지털 로직 레벨(CMOS/TTL)로 변환하는 역할을 한다.

#### 3.2.1 기본 구조 및 넓은 공통 모드 입력 범위

수신기의 핵심은 고입력 임피던스를 갖는 차동 증폭기(differential amplifier)이다.12 입력 임피던스가 매우 높기 때문에(수백 $\text{k}\Omega$ 이상), 드라이버에서 온 $3.5\text{mA}$의 전류는 대부분 수신기 입력단을 거치지 않고 병렬로 연결된 $100\Omega$ 종단 저항으로 흐르게 된다.10 수신기는 이 저항 양단에 걸리는 전압 차이만을 감지한다.

표준 LVDS 수신기는 $0\text{V}$에서 $2.4\text{V}$에 이르는 넓은 공통 모드 입력 범위를 지원해야 한다. 이를 구현하기 위해 많은 수신기들은 NMOS 차동 쌍과 PMOS 차동 쌍을 병렬로 연결한 레일-투-레일(Rail-to-Rail) 입력단을 채택한다.24 입력 공통 모드 전압이 낮을 때는 PMOS 쌍이 주로 동작하고, 높을 때는 NMOS 쌍이 동작하여 전체 입력 범위에 걸쳐 안정적인 증폭이 가능하도록 한다.

#### 3.2.2 Fail-Safe 기능

실제 시스템에서는 케이블이 연결되지 않거나(open-circuit), 두 신호선이 서로 붙어버리거나(short-circuit), 혹은 송신 드라이버의 전원이 꺼지는 등 유효한 차동 신호가 입력되지 않는 상황이 발생할 수 있다. 이러한 경우, 일반적인 차동 수신기의 두 입력 단자 전압은 거의 같아져 출력이 '0'과 '1' 사이를 불안정하게 빠르게 진동(oscillation)하는 문제가 발생할 수 있다.

Fail-Safe 기능은 이러한 비정상적인 상황에서 수신기 출력을 항상 예측 가능한 특정 논리 상태(일반적으로 HIGH)로 유지시켜주는 매우 중요한 기능이다.13 이는 수신기 입력단 회로에 의도적으로 미세한 비대칭성(asymmetry)을 주어 구현된다. 예를 들어, 입력단에 작은 전류원을 추가하여 바이어스를 인가함으로써, 외부 입력이 없을 때에도 인위적으로 수 

$\text{mV}$ 정도의 작은 양의 차동 전압이 발생하도록 설계한다. 이 작은 전압은 수신기를 항상 특정 상태로 유지시키기에 충분하며, 시스템의 안정성을 크게 향상시킨다.9

### 3.3  SerDes(Serializer/Deserializer)와의 결합

LVDS는 그 자체로도 훌륭한 인터페이스 기술이지만, SerDes 기술과 결합될 때 그 가치가 극대화된다. SerDes는 'Serializer(직렬화기)'와 'Deserializer(병렬화기)'의 합성어로, 다수의 저속 병렬 데이터 라인을 소수의 고속 직렬 데이터 라인으로 변환하고, 이를 다시 원래의 병렬 데이터로 복원하는 기술을 의미한다.

이 조합에서 LVDS는 고속 직렬 데이터를 안정적으로 전송하는 물리 계층(PHY)의 역할을 담당한다. 예를 들어, 24비트 RGB 비디오 데이터와 4개의 제어 신호를 포함하는 총 28비트의 병렬 데이터를 전송해야 한다고 가정해보자. 이를 병렬로 전송하려면 최소 28개의 신호선이 필요하다. 하지만 SerDes 칩을 사용하면 이 28비트 데이터를 4개의 고속 직렬 채널로 직렬화할 수 있다. 그리고 이 4개의 직렬 채널을 각각 LVDS 차동 쌍으로 전송하는 것이다.25 수신단에서는 4개의 LVDS 수신기가 신호를 받은 후, Deserializer가 이를 다시 28비트 병렬 데이터로 복원한다.

이러한 SerDes와 LVDS의 결합은 다음과 같은 막대한 이점을 제공한다:

- **케이블 및 커넥터 감소**: 필요한 신호선의 수가 28개에서 8개(4쌍)로 획기적으로 줄어들어 케이블의 두께, 무게, 비용을 크게 절감할 수 있다.
- **스큐(Skew) 문제 해결**: 병렬 데이터 전송에서 가장 큰 문제 중 하나는 각 신호선 간의 미세한 길이 차이나 전기적 특성 차이로 인해 신호 도착 시간이 달라지는 스큐(skew) 현상이다. 직렬화는 이 문제를 근본적으로 해결하여 데이터 전송의 정확도를 높인다.1
- **EMI 감소**: 신호선의 수가 줄어들면 그만큼 외부로 방출되는 EMI의 총량도 감소한다.

이러한 SerDes와 LVDS의 강력한 조합을 가장 성공적으로 상용화한 사례가 바로 **FPD-Link(Flat Panel Display Link)**이다. FPD-Link는 그래픽 컨트롤러에서 출력되는 수십 개의 병렬 RGB 비디오 데이터를 몇 쌍의 LVDS 라인으로 직렬화하여 LCD 패널로 전송하는 데 사용되었으며, 노트북 컴퓨터와 평판 디스플레이 시장의 성장을 견인한 핵심 기술이 되었다.3

시스템의 실제 운영 환경에서는 전원이 켜진 상태에서 케이블을 연결하거나 분리하는 '핫 플러깅(Hot Plugging)'이 요구되는 경우가 많다. 수신기의 Fail-Safe 기능은 이러한 상황에서 시스템 안정성을 보장하는 핵심적인 역할을 한다. 만약 케이블이 분리된 상태에서 Fail-Safe 기능이 없다면, 수신기 입력은 전기적으로 불안정한 플로팅(floating) 상태가 되어 미세한 노이즈에도 출력이 심하게 진동할 수 있다. 이는 후속 로직 회로에 치명적인 오작동을 유발하고, 심한 경우 과도한 전류로 인해 칩이 손상될 수도 있다. Fail-Safe 기능은 케이블이 분리되었을 때 수신기 출력을 안정된 논리 상태로 즉시 고정시켜준다.13 이 덕분에 시스템은 전기적 스트레스나 로직 에러 없이 안전하게 케이블의 연결 및 분리를 감지하고 처리할 수 있게 된다. 따라서 Fail-Safe는 단순한 예외 처리 기능을 넘어, LVDS가 견고한 시스템 인터페이스로서 핫 플러깅과 같은 고급 기능을 지원할 수 있게 만드는 필수 불가결한 설계 요소라 할 수 있다.14

## 4.  LVDS의 장점과 한계

모든 공학 기술과 마찬가지로, LVDS 역시 뚜렷한 장점과 함께 특정 응용 분야에서는 한계를 보인다. 이 장에서는 LVDS의 핵심적인 장점들을 심층적으로 분석하고, 기술적 한계와 이를 극복하기 위해 등장한 파생 기술들을 살펴본다.

### 4.1  핵심 장점 심층 분석

LVDS는 네 가지 핵심적인 장점을 통해 고속 인터페이스 시장에서 독보적인 위치를 차지했다.

#### 4.1.1 높은 노이즈 내성 (High Noise Immunity)

이는 LVDS의 가장 근본적인 장점이다. 제1장에서 설명했듯이, 차동 신호 방식은 전송 경로에서 공통으로 유입되는 노이즈를 수신단에서 효과적으로 상쇄시킨다.1 여기에 더해, TIA/EIA-644 표준이 보장하는 $\pm1\text{V}$의 넓은 공통 모드 전압 허용 범위는 송신기와 수신기 간에 상당한 수준의 접지 전위차가 발생하더라도 통신 링크가 끊어지지 않도록 보호한다.14 이러한 특성은 모터나 릴레이 등 전기적 노이즈가 많은 산업 환경이나, 여러 전자 장치가 밀집된 자동차 환경에서 LVDS가 선호되는 주된 이유이다.

#### 4.1.2 낮은 EMI 방출 (Low EMI Emission)

LVDS는 다른 고속 인터페이스에 비해 '조용한' 기술로 알려져 있다. 이는 두 가지 주요 메커니즘에 기인한다. 첫째, 차동 쌍의 두 신호선에는 크기가 같고 방향이 반대인 전류가 흐른다. 오른나선 법칙에 따라 각 도선에서 발생하는 자기장은 서로 반대 방향이 되어 상쇄되는 효과를 낳는다. 두 도선이 꼬임쌍선(twisted pair)처럼 긴밀하게 결합될수록 이 상쇄 효과는 극대화되어 외부로 방출되는 전자기파(EMI)가 최소화된다.3 둘째, 정전류원(constant-current) 드라이버의 특성상, 신호가 0에서 1로 또는 1에서 0으로 스위칭될 때에도 전원 공급 라인에서 끌어오는 전류의 변화가 거의 없다. 이는 전원 라인에 급격한 전류 스파이크를 발생시켜 EMI의 주요 원인이 되는 CMOS나 TTL 로직과 대조되는 점이다.3 마지막으로, $350\text{mV}$에 불과한 낮은 전압 스윙 자체도 전압 변화율($dV/dt$)을 낮춰 EMI 방출을 줄이는 데 기여한다.1

#### 4.1.3 저전력 소모 (Low Power Consumption)

LVDS의 저전력 특성은 타의 추종을 불허한다. 드라이버가 $3.5\text{mA}$의 정전류를 $100\Omega$ 종단 저항에 흘려보낼 때 소모되는 전력은 약 $1.2\text{mW}$에 불과하다.1 이는 다른 고속 인터페이스 기술에 비해 월등히 낮은 수치이다. 더욱 중요한 점은, LVDS의 전력 소모는 동작 주파수에 거의 영향을 받지 않는다는 것이다. CMOS 기술의 경우, 주파수가 증가함에 따라 커패시터를 충전하고 방전하는 데 필요한 동적 전력(dynamic power)이 기하급수적으로 증가하는 반면, LVDS는 정전류 구동 방식 덕분에 수백 $\text{MHz}$, 수 $\text{GHz}$로 동작해도 전력 소모가 거의 일정하게 유지된다.14 이러한 특성은 발열 관리가 중요한 고밀도 서버나 백플레인, 그리고 배터리 사용 시간이 핵심인 노트북과 같은 휴대용 기기에 매우 큰 이점을 제공한다.19

#### 4.1.4 고속 전송 능력 (High-Speed Capability)

낮은 전압 스윙은 신호의 논리 레벨을 바꾸는 데 필요한 시간을 단축시켜 매우 빠른 신호 천이(transition)를 가능하게 한다. 이는 데이터 전송 속도를 높이는 데 직접적인 원인이 된다.5 TIA/EIA-644 표준 자체는 최대 $655\text{Mbps}$의 데이터 전송률을 권장하고 이론상 $1.9\text{Gbps}$까지 가능하다고 명시하지만 5, 이는 1990년대 기술을 기준으로 한 것이다. 현대의 진보된 반도체 공정 기술 덕분에, 현재 시중에는 $3\text{Gbps}$를 초과하는 데이터 전송 속도를 지원하는 LVDS 기반 제품들이 다수 출시되어 있다.1

### 4.2  기술적 한계 및 파생 기술

이처럼 많은 장점에도 불구하고, 표준 LVDS는 특정 토폴로지에서 명확한 한계를 가진다.

#### 4.2.1 Point-to-Point 구조 최적화

표준 LVDS는 하나의 송신 드라이버와 하나의 수신기가 1:1로 연결되는 **Point-to-Point** 토폴로지에 가장 최적화되어 있다.15 이 구성은 송신단, 전송선로, 수신단 종단 저항으로 이어지는 임피던스 경로가 가장 단순하고 예측 가능하여 신호 반사를 최소화하고 신호 무결성을 극대화할 수 있다.

#### 4.2.2 Multi-Drop 및 Multipoint의 한계

- **Multi-Drop**: 하나의 드라이버에 여러 개의 수신기를 병렬로 연결하는 **Multi-Drop** 구성은 표준 LVDS로도 구현이 가능하지만 제약이 따른다. 주 전송선로에서 각 수신기로 분기되는 짧은 선로를 '스터브(stub)'라고 하는데, 이 스터브는 임피던스 불일치를 유발하는 안테나처럼 작용하여 고속 신호에서 심각한 반사를 일으킨다. 따라서 Multi-Drop 구성에서는 스터브 길이를 극도로 짧게 유지해야 하며, 연결 가능한 수신기의 수와 전체 전송 거리가 제한된다.9
- **Multipoint**: 여러 개의 드라이버가 하나의 버스를 공유하여 양방향 통신을 하는 **Multipoint** 구성(또는 버스 구성)은 표준 LVDS로는 지원되지 않는다. 표준 LVDS 드라이버는 항상 전류를 출력하도록 설계되어 있어, 두 개 이상의 드라이버가 동시에 버스에 연결되면 충돌이 발생하기 때문이다.

#### 4.2.3 파생 기술의 등장

이러한 한계를 극복하고 버스 환경에 대응하기 위해 LVDS의 기본 원리를 계승하면서도 특성을 개선한 여러 파생 기술이 개발되었다.

- **M-LVDS (Multipoint-LVDS)**: TIA/EIA-899 표준으로 제정된 M-LVDS는 Multipoint 애플리케이션을 위해 특별히 설계되었다. 표준 LVDS보다 더 높은 구동 전류를 갖는 드라이버를 사용하여, 버스의 양 끝에 위치한 두 개의 종단 저항(double termination)을 구동할 수 있다. 이를 통해 최대 32개의 노드(드라이버 또는 수신기)를 하나의 버스에 연결할 수 있다. 또한, 어떤 드라이버도 동작하지 않는 버스 유휴(idle) 상태에서 수신기 출력이 불안정해지는 것을 막기 위해, 입력 임계값에 오프셋을 준 'Type 2' 수신기를 정의하는 등 버스 환경에 특화된 기능들을 포함한다.2
- **Bus LVDS (BLVDS)**: M-LVDS와 유사한 개념으로, 주로 고밀도 백플레인과 같이 부하가 큰 다중 접속 환경을 위해 드라이버의 구동 능력을 강화한 기술이다. M-LVDS와 마찬가지로 다중 종단 버스를 구동할 수 있으며, 활성화된 버스에 카드를 삽입하거나 제거하는 '핫 플러깅' 기능을 지원하도록 설계되었다.28

LVDS의 Point-to-Point 구조적 한계는 실패가 아니라, 오히려 '고속 신호 무결성'이라는 명확한 목표를 위해 다른 요소들을 희생한 '최적화의 결과'로 해석해야 한다. 고속 신호를 왜곡 없이 전송하기 위한 가장 이상적인 조건은 임피던스 경로가 단순하고 예측 가능한 1:1 연결이다. Multi-Drop이나 Multipoint 구성은 필연적으로 스터브나 다중 부하로 인한 임피던스 불연속점을 만들어내고, 이는 고속 신호에 치명적인 반사를 유발한다. 따라서 LVDS가 Point-to-Point에 최적화된 것은 기술적 한계라기보다는 '설계 철학의 전략적 선택'이었다. 하지만 시장은 버스 구조의 유연성을 요구했고, 이러한 요구는 LVDS의 핵심 원리(저전압, 차동, 전류 구동)는 그대로 계승하되, 구동 전류를 높이고 수신기 특성을 변경하여 버스 환경에 맞게 '재조정'한 M-LVDS와 BLVDS라는 새로운 파생 기술의 탄생으로 이어졌다.2 이는 하나의 성공적인 기술이 어떻게 시장의 다양한 요구사항에 맞춰 분화하고 발전해 나가는지를 보여주는 전형적인 사례이며, LVDS의 구조적 한계가 오히려 기술 진화의 동력으로 작용했음을 시사한다.

## 5.  고속 인터페이스 기술 비교

LVDS 기술의 특성과 위상을 보다 명확하게 이해하기 위해서는, 동시대에 사용되었거나 현재 경쟁하고 있는 다른 주요 고속 인터페이스 기술들과의 비교 분석이 필수적이다. 각 기술은 고유의 장단점을 가지며, 특정 애플리케이션의 요구사항에 따라 그 적합성이 달라진다. 아래 표는 LVDS를 단일 종단 방식(TTL/CMOS), PECL, CML과 같은 주요 기술들과 핵심 지표를 기준으로 비교 분석한 것이다. 이 표는 엔지니어가 시스템 설계 시 직면하는 트레이드오프(trade-off)를 이해하고, 주어진 조건에 가장 적합한 기술을 선택하는 데 유용한 가이드라인을 제공한다.

| 특성 (Specification)          | 단일 종단 (Single-Ended)                 | LVDS (Low-Voltage Differential Signaling)                    | PECL/LVPECL (Positive Emitter-Coupled Logic) | CML (Current-Mode Logic)              |
| ----------------------------- | ---------------------------------------- | ------------------------------------------------------------ | -------------------------------------------- | ------------------------------------- |
| **신호 방식**                 | 단일 종단 (vs. GND)                      | 차동 (Differential)                                          | 차동 (Differential)                          | 차동 (Differential)                   |
| **대표 표준**                 | -                                        | **TIA/EIA-644**                                              | - (사실상 표준)                              | - (주로 Vendor-specific)              |
| **버스 구조**                 | Point-to-Point                           | Point-to-Point, Multi-drop (M-LVDS/BLVDS는 Multipoint 지원)  | Point-to-Point, Multi-drop                   | Point-to-Point                        |
| **데이터 속도**               | $\sim$수백 $\text{Mbps}$                 | **$\sim3 \text{Gbps}$** (표준은 $655\text{Mbps}$, 구현에 따라 다름) | $>10 \text{Gbps}$                            | **$>10 \text{Gbps}$**                 |
| **전력 소모**                 | 낮음~중간                                | **매우 낮음 (Very Low)**                                     | 높음 (High)                                  | 중간 (Medium)                         |
| **출력 전압 스윙**            | $V_{cc}$에 따라 다름 (예: $3.3\text{V}$) | **$\sim350 \text{mV}$** (가장 작음)                          | $\sim800 \text{mV}$                          | $\sim800 \text{mV}$                   |
| **공통 모드 전압 ($V_{OS}$)** | N/A                                      | **$\sim1.2 \text{V}$**                                       | $V_{cc} - 1.3\text{V}$                       | $V_{cc}$에 가까움                     |
| **종단 방식**                 | 다양함                                   | **수신단 $100\Omega$ 종단 (필수)**                           | 수신단 Pull-down 저항 및 $50\Omega$ 종단     | 주로 $50\Omega$ 종단 (내부 구현 가능) |
| **주요 공정**                 | CMOS                                     | **CMOS, BiCMOS**                                             | Bipolar, SiGe                                | Bipolar, CMOS, SiGe                   |
| **주요 장점**                 | 단순함, 저비용                           | **저전력, 저EMI, 높은 노이즈 내성**                          | 초고속, 안정적인 출력                        | 초고속, 간단한 AC 커플링              |
| **주요 단점**                 | 노이즈에 취약, 고속 한계                 | Multipoint 지원 제한                                         | 높은 전력 소모, 음수 전원 필요 가능성        | 표준 부재, 높은 전력 소모             |
| **자료 출처**                 | 5                                        | 1                                                            | 27                                           | 27                                    |

**비교 분석 심층 해설:**

- **단일 종단 (TTL/CMOS)**: 가장 기본적인 방식으로, 저속의 보드 내부 신호 전송에 적합하다. 구현이 간단하고 비용이 저렴하지만, 노이즈에 매우 취약하고 고속 전송에 한계가 명확하여 수백 $\text{Mbps}$ 이상의 속도에서는 거의 사용되지 않는다.5
- **LVDS**: 이 표에서 LVDS의 독보적인 위치는 '균형'에 있다. 속도, 전력, 노이즈 특성 모든 면에서 우수한 성능을 보인다. 특히 **매우 낮은 전력 소모**와 **가장 작은 출력 전압 스윙**은 LVDS의 핵심 정체성이다. 이는 전력 효율과 EMI 억제가 중요한 대부분의 애플리케이션에서 LVDS를 매력적인 선택지로 만든다. $10\text{Gbps}$급의 초고속이 필요하지 않은 거의 모든 영역에서 가장 합리적인 솔루션이라 할 수 있다.1
- **PECL/LVPECL**: 속도 면에서는 LVDS보다 우수하며, $10\text{Gbps}$ 이상의 초고속 통신이 가능하다. 이는 주로 Bipolar 공정을 기반으로 하여 매우 빠른 스위칭 속도를 달성하기 때문이다. 하지만 가장 큰 단점은 **높은 전력 소모**이다. 또한, 드라이버 출력이 오픈 에미터(open emitter) 구조이므로 항상 외부 종단 저항이 필요하며, 전통적인 ECL은 음수 전원을 요구하여 시스템 설계가 복잡해질 수 있다.27
- **CML**: CML은 PECL과 함께 초고속 인터페이스 시장을 양분하는 기술이다. $10\text{Gbps}$를 훌쩍 넘는 데이터 속도를 지원하며, 특히 드라이버 출력이 $50\Omega$ 부하에 맞춰져 있어 별도의 종단 네트워크 없이 $50\Omega$ 전송선로에 직접 연결할 수 있다는 장점이 있다. 이는 AC 커플링을 단순화시킨다. 하지만 PECL과 마찬가지로 전력 소모가 LVDS보다 크며, 아직 업계 표준이 명확하지 않아 제조사별로 전기적 특성이 다를 수 있다는 단점이 있다.27

결론적으로, 극도의 저전력과 우수한 노이즈 내성이 요구되는 기가비트급 이하의 애플리케이션에서는 LVDS가 최적의 선택이다. 반면, $10\text{Gbps}$ 이상의 최첨단 속도가 반드시 필요한 광통신 모듈이나 고성능 백플레인과 같은 분야에서는 전력 소모를 감수하더라도 PECL이나 CML이 사용된다. 이처럼 각 기술은 명확한 트레이드오프 관계를 가지며, LVDS는 그중에서도 가장 넓은 범용성과 균형 잡힌 성능을 제공하는 기술로 평가할 수 있다.

## 6.  LVDS 시스템 설계 및 PCB 레이아웃 가이드라인

LVDS의 뛰어난 이론적 성능을 실제 하드웨어에서 온전히 구현하기 위해서는 세심한 시스템 설계와 PCB(Printed Circuit Board) 레이아웃이 필수적이다. 고속으로 동작하는 LVDS 신호는 단순한 디지털 펄스가 아닌, 아날로그 파형으로 취급해야 하며, 전송선로 이론에 기반한 설계 원칙을 철저히 준수해야 한다.

### 6.1  차동 임피던스 제어

LVDS 시스템 설계에서 가장 중요한 개념은 '임피던스 제어(Impedance Control)'이다.

- **중요성**: 수백 $\text{MHz}$ 이상의 고주파 신호에게 PCB 트레이스는 단순한 구리선이 아니라 고유의 전기적 특성을 갖는 '전송선로(Transmission Line)'로 동작한다. 만약 전송선로의 특성 임피던스가 드라이버의 출력 임피던스나 수신단의 종단 저항과 일치하지 않으면, 임피던스가 변하는 지점에서 신호 에너지가 반사된다. 이 반사파는 원래 신호에 중첩되어 파형을 왜곡시키고, 이는 데이터 오류율(Bit Error Rate)을 높이는 직접적인 원인이 된다.32

- **100Ω 차동 임피던스**: LVDS 시스템은 전체 신호 경로가 $100\Omega$의 차동 임피던스를 갖도록 설계하는 것을 표준으로 한다.20 차동 임피던스는 차동 쌍을 이루는 두 신호선 사이에 신호가 인가될 때 나타나는 임피던스를 의미한다. 이는 일반적으로 각 단일 신호선이 접지면(ground plane)에 대해 

  $50\Omega$의 특성 임피던스를 갖도록 설계하고, 두 신호선을 적절한 간격으로 배치함으로써 달성된다.11

- **임피던스 결정 요소**: PCB 트레이스의 특성 임피던스는 물리적인 구조에 의해 결정된다. 주요 요소는 다음과 같다:

  - 트레이스의 폭 (W, Width)
  - 트레이스와 기준면(접지 또는 전원면) 사이의 유전체 두께 (H, Height)
  - PCB 유전체의 유전 상수 (Er, Dielectric Constant)
  - 차동 쌍의 경우, 두 트레이스 사이의 간격 (S, Spacing)

  PCB 설계자는 원하는 임피던스 값을 얻기 위해 PCB 제작사와 협력하여 이러한 파라미터들을 정밀하게 계산하고 제어해야 한다.32

### 6.2  차동 쌍 배선 규칙

차동 신호의 장점을 극대화하고 신호 왜곡을 최소화하기 위해, 차동 쌍(P/N) 배선 시에는 다음과 같은 규칙을 반드시 준수해야 한다.

- **등-길이 배선 (Length Matching)**: 차동 쌍을 구성하는 두 트레이스(Positive/Negative)의 물리적 길이는 전송 경로 전체에 걸쳐 가능한 한 정확하게 일치해야 한다. 만약 두 트레이스의 길이가 다르면 신호가 수신기에 도달하는 시간에 차이가 발생하는데, 이를 '스큐(skew)'라고 한다. 스큐는 두 신호의 위상차를 발생시켜 차동 신호의 진폭을 감소시키고, 일부는 공통 모드 노이즈로 변환되어 시스템의 노이즈 내성을 약화시킨다. 일반적으로 허용되는 길이 차이는 매우 작으며, 고속 설계에서는 $5 \text{ mils}$(약 $0.127\text{mm}$) 이내로 정밀하게 일치시키는 것이 권장된다. 길이가 짧은 쪽 트레이스에 의도적으로 구불구불한 패턴(serpentine/meander)을 추가하여 길이를 맞추는 기법이 사용된다.32
- **밀접하고 평행한 배선 (Tightly Coupled Parallel Routing)**: 두 트레이스는 드라이버에서 수신기까지 가능한 한 가깝고 일정한 간격을 유지하며 평행하게 배선해야 한다.18 이는 두 가지 중요한 이유 때문이다. 첫째, 일정한 간격은 전송 경로상의 차동 임피던스를 일정하게 유지하는 데 필수적이다. 둘째, 두 트레이스가 밀접하게 결합(coupling)될수록 외부에서 유입되는 노이즈가 양쪽에 거의 동일하게 영향을 미치게 되어 공통 모드 노이즈 제거 효과가 극대화된다. 또한, 두 트레이스에서 방출되는 EMI가 서로를 효과적으로 상쇄시키는 데에도 유리하다.35
- **90도 꺾임 금지**: 신호 경로에 90도 직각 꺾임을 사용하는 것은 반드시 피해야 한다. 꺾이는 지점에서 트레이스의 폭이 실효적으로 넓어져 특성 임피던스가 급격히 낮아지고, 이는 심각한 신호 반사를 유발한다. 대신, 두 개의 45도 꺾임을 사용하거나, 135도 이상의 완만한 각도를 사용하거나, 가장 이상적으로는 부드러운 호(arc) 형태로 배선해야 한다.32
- **비아(Via) 사용 최소화 및 대칭적 사용**: 비아는 PCB의 다른 층으로 신호를 연결하는 통로이지만, 그 구조상 주변 트레이스와 다른 임피던스 특성을 갖는 대표적인 불연속점이다. 따라서 LVDS와 같은 고속 신호 경로에서는 비아 사용을 가급적 피하는 것이 좋다. 부득이하게 사용해야 할 경우, 차동 쌍의 P/N 트레이스 모두에 대칭적으로 비아를 사용해야 하며, 신호의 리턴 전류가 흐를 수 있도록 신호 비아 바로 옆에 하나 이상의 접지 스티칭 비아(ground stitching via)를 함께 배치해야 한다.36

### 6.3  크로스토크 및 노이즈 저감

- **타 신호와의 이격**: LVDS 차동 쌍은 인접한 다른 신호선, 특히 전압 스윙이 큰 TTL이나 CMOS 신호로부터 충분한 거리를 두고 배선해야 한다. 인접 신호의 스위칭이 LVDS 라인에 노이즈를 유발하는 현상을 '크로스토크(crosstalk)'라고 한다. 이를 방지하기 위해 LVDS 쌍과 다른 신호선 사이에 최소 $12\text{mm}$의 거리를 두거나, 아예 다른 레이어에 배치하여 접지면으로 분리하는 것이 권장된다.32
- **견고한 접지면 (Solid Ground Plane)**: LVDS 트레이스가 배선되는 레이어의 바로 아래 또는 위에는 가능한 한 넓고 연속적인 접지면을 배치해야 한다. 이 접지면은 신호 전류의 귀환 경로(return path) 역할을 하는데, 이 경로가 짧고 안정적일수록 임피던스가 일정하게 유지되고 노이즈에 대한 내성이 강해진다. 신호 경로 아래의 접지면이 갈라져 있거나(split plane), 구멍이 뚫려 있는(void) 구간은 반드시 피해야 한다. 신호가 이런 구간을 지나가면 리턴 경로가 길게 우회하게 되어 심각한 신호 왜곡과 EMI를 유발한다.35
- **전원 디커플링 (Power Supply Decoupling)**: LVDS 드라이버 및 수신기 IC의 전원(VCC) 핀과 접지(GND) 핀 사이에는 가능한 한 가깝게 디커플링 커패시터를 배치해야 한다. 일반적으로 $0.1\mu\text{F}$, $0.01\mu\text{F}$ 등 다양한 용량의 커패시터를 병렬로 연결하여 넓은 주파수 대역의 노이즈를 효과적으로 제거한다. 이 커패시터들은 칩이 고속으로 스위칭할 때 필요한 순간적인 전류를 공급하고, 전원 라인을 통해 유입되는 노이즈를 걸러내는 중요한 역할을 한다.32

LVDS PCB 설계 가이드라인에 나열된 모든 규칙들, 즉 등-길이 배선, 평행 배선, 비아의 대칭적 사용, 임피던스 제어 등을 관통하는 단 하나의 핵심 원칙은 바로 '대칭성(Symmetry)'이다. LVDS 기술의 모든 이론적 장점은 차동 쌍을 이루는 두 신호선(P와 N)이 전기적으로 완벽하게 상보적이라는 가정에서 출발한다. 외부에서 노이즈가 유입될 때, 두 선에 '동일하게(symmetrically)' 영향을 주어야만 수신기에서 효과적으로 상쇄될 수 있다. 만약 두 선의 길이가 다르거나(asymmetric length), 한쪽 선만 다른 신호에 더 가깝게 배선되거나(asymmetric environment), 리턴 경로가 비대칭적이라면, 두 선에 미치는 영향이 달라져 공통 모드 노이즈가 제거되지 않고 오히려 차동 노이즈로 변환(mode conversion)되는 최악의 결과를 낳는다. 따라서 성공적인 LVDS PCB 설계의 목표는 단순히 규칙 목록을 따르는 것을 넘어, 물리적, 전기적 대칭성을 시스템 전체에 걸쳐 최대한 유지하는 것이다. 설계자는 모든 배선 결정 과정에서 "이 결정이 P/N 쌍의 대칭성을 깨뜨리는가?"라는 질문을 끊임없이 자문해야 한다.

## 7.  LVDS의 주요 응용 분야 및 발전 동향

LVDS는 1990년대 중반 등장한 이래, 그 뛰어난 특성을 바탕으로 다양한 전자 산업 분야에 깊숙이 뿌리내렸다. 이 장에서는 LVDS가 핵심적인 역할을 수행해 온 주요 응용 분야를 살펴보고, 최신 기술 동향과 시장 전망을 통해 LVDS의 현재와 미래를 조망한다.

### 7.1  디스플레이 인터페이스 (FPD-Link & LDI)

LVDS 기술의 성공 신화는 디스플레이 인터페이스 분야에서 시작되었다고 해도 과언이 아니다.

- **초기 시장 지배와 FPD-Link**: 1996년, National Semiconductor(이후 Texas Instruments에 인수)는 LVDS 기술을 기반으로 한 **FPD-Link(Flat Panel Display Link)** 칩셋을 출시했다.26 이는 당시 막 성장하던 노트북 컴퓨터 시장의 요구에 완벽하게 부응하는 솔루션이었다. FPD-Link는 메인보드의 그래픽 컨트롤러에서 출력되는 수십 가닥의 병렬 RGB 비디오 신호를 몇 쌍의 LVDS 라인으로 직렬화하여, 얇고 유연해야 하는 노트북의 힌지(hinge) 부분을 통해 LCD 패널로 안정적으로 전송할 수 있게 해주었다. 이 기술은 곧 노트북 디스플레이 인터페이스의 사실상의 표준(de facto standard)으로 자리 잡았고, 노트북의 슬림화와 고해상도화에 결정적인 기여를 했다.3

- **기술의 진화**: FPD-Link는 시장의 요구에 따라 지속적으로 발전했다. 초기 FPD-Link는 데이터 채널과 별도의 클럭 채널을 사용했지만, 이후 클럭 신호를 데이터 신호에 임베딩하여 필요한 케이블 수를 더욱 줄인 **FPD-Link II**가 등장했다.26 2010년에 소개된 

  **FPD-Link III**는 한 걸음 더 나아가, 동일한 차동 쌍을 통해 비디오 데이터 전송과 함께 양방향 제어 신호(예: I2C)까지 주고받을 수 있는 기능을 통합했다. 이를 통해 터치스크린 정보나 디스플레이 설정 제어를 위한 별도의 케이블이 필요 없게 되었다.26 흥미로운 점은, FPD-Link III부터는 

  $3\text{Gbps}$ 이상의 더 높은 데이터 전송 속도를 지원하기 위해 물리 계층을 LVDS에서 CML(Current-Mode Logic)로 전환하기 시작했다는 것이다.37

- **현재의 위상**: 오늘날 소비자용 노트북이나 스마트폰 디스플레이 인터페이스 시장의 주도권은 더 높은 대역폭과

#### 참고자료

1. LVDS(Low voltage differential signaling) 란?, accessed August 9, 2025, https://hanbulkr.tistory.com/11
2. 낮은 전압 차분 신호 - 위키백과, 우리 모두의 백과사전, accessed August 9, 2025, [https://ko.wikipedia.org/wiki/%EB%82%AE%EC%9D%80_%EC%A0%84%EC%95%95_%EC%B0%A8%EB%B6%84_%EC%8B%A0%ED%98%B8](https://ko.wikipedia.org/wiki/낮은_전압_차분_신호)
3. Low-voltage differential signaling - Wikipedia, accessed August 9, 2025, https://en.wikipedia.org/wiki/Low-voltage_differential_signaling
4. LVDS based FPD-Link spans industries with Gigabits @ milliwatts! - Texas Instruments, accessed August 9, 2025, https://www.ti.com/lit/pdf/snla205
5. LVDS, accessed August 9, 2025, https://onea01.tistory.com/288
6. LVDS의 장단점 - 지식 - Xiamen Kabasi Electric Co., Ltd, accessed August 9, 2025, https://ko.kbs-connector.com/info/advantages-and-disadvantages-of-lvds-63999383.html
7. CameraLink 1. LVDS 기술 - KDH Blog - 티스토리, accessed August 9, 2025, https://eastwind17.tistory.com/68
8. ko.lcdtftlcd.com, accessed August 9, 2025, [https://ko.lcdtftlcd.com/news-show-287.html#:~:text=LVDS(%EC%A0%80%EC%A0%84%EC%95%95%20%EC%B0%A8%EB%8F%99%20%EC%8B%A0%ED%98%B8)%EB%8A%94,%EC%A0%84%EB%A0%A5%20%EC%86%8C%EB%B9%84%EB%A5%BC%20%EB%8B%AC%EC%84%B1%ED%95%A9%EB%8B%88%EB%8B%A4.](https://ko.lcdtftlcd.com/news-show-287.html#:~:text=LVDS(저전압 차동 신호)는,전력 소비를 달성합니다.)
9. Interface Circuits for TIA/EIA-644 (LVDS) (Rev. B) - Texas Instruments, accessed August 9, 2025, https://www.ti.com/lit/pdf/slla038
10. High Speed LVDS Driver for SERDES - MIT, accessed August 9, 2025, http://web.mit.edu/magic/Public/papers/05441164.pdf
11. LVDS 신호의 형태와 송수신 구조의 예시 - 망고토마토, accessed August 9, 2025, https://mangto.tistory.com/31
12. Theory of Operation and VDD Fault Scenario for LVDS Products APPLICATION N OTE - Frontgrade, accessed August 9, 2025, https://www.frontgrade.com/sites/default/files/documents/App-Note-LVDS-TheoryOfOperation.pdf
13. Low-Voltage Differential Signaling (LVDS) - TestWorld Inc., accessed August 9, 2025, https://www.testworld.com/wp-content/uploads/Low-Voltage-Differential-Signaling-LVDS.pdf
14. LVDS케이블이란?, accessed August 9, 2025, http://cabletronkorea.com/bbs/board.php?bo_table=harness&wr_id=48
15. AN-5017 LVDS Fundamentals, accessed August 9, 2025, https://www.onsemi.jp/download/application-notes/pdf/an-5017.pdf
16. LVDS Signals Crashcourse | Advanced PCB Design Blog | Cadence, accessed August 9, 2025, https://resources.pcb.cadence.com/blog/2023-lvds-signals-crashcourse
17. AN-1177: LVDS and M-LVDS Circuit Implementation Guide | Analog Devices, accessed August 9, 2025, https://www.analog.com/en/resources/app-notes/an-1177.html
18. 디스플레이 LVDS 인터페이스 기술 원리 및 세부 소개 - 뉴스, accessed August 9, 2025, https://ko.lcdtftlcd.com/news-show-287.html
19. LVDS 케이블과 HDMI 케이블의 장단점은 무엇입니까? - 지식, accessed August 9, 2025, https://ko.top-cables.com/info/what-s-the-advantage-and-disadvantage-between-87712085.html
20. LVDS 케이블 어셈블리 란?-지식-Ecocables Electronics Co., Ltd, accessed August 9, 2025, https://ko.wire-harness.net/info/what-are-lvds-cable-assemblies-49389814.html
21. Reason for 100 Ohm Resistor on Transmitter of the NI 6585 - Support - National Instruments, accessed August 9, 2025, https://knowledge.ni.com/KnowledgeArticleDetails?id=kA00Z0000004AShSAM
22. Gbps급 LVDS I/O에 관한 연구 - 초고속 회로 및 시스템 연구실, accessed August 9, 2025, http://tera.yonsei.ac.kr/publication/pdf/MS_2003_colee.pdf
23. LVDS driver schematic. | Download Scientific Diagram - ResearchGate, accessed August 9, 2025, https://www.researchgate.net/figure/LVDS-driver-schematic_fig6_224344651
24. LVDS Driver & Receiver - CERN Indico, accessed August 9, 2025, https://indico.cern.ch/event/72160/contribution/2082283/attachments/1036626/1477154/LVDS-V1.pdf
25. 고속 인터페이스 LVDS Serdes란, 그 기본원리와 특징｜자인일렉트로닉스 - THine, accessed August 9, 2025, https://www.thine.co.jp/ko/contents/detail/serdes-lvds.html
26. FPD-Link - Wikipedia, accessed August 9, 2025, https://en.wikipedia.org/wiki/FPD-Link
27. ECL vs LVDS vs CML: A Comparative Analysis | RF Wireless World, accessed August 9, 2025, https://www.rfwireless-world.com/terminology/ecl-vs-lvds-vs-cml
28. LVDS Product Family Introductions, accessed August 9, 2025, https://www.asc.ohio-state.edu/physics/cms/cfeb/datasheets/LVDS_Note0.html
29. The ECS Inc. Guide to Oscillator Output Types, accessed August 9, 2025, https://ecsxtal.com/store/pdf/output-signal-types.pdf
30. LVPECL, PECL, ECL Logic and Termination, accessed August 9, 2025, https://www.mouser.com/pdfDocs/peclclocksandtermination.pdf
31. Interface between High speed signal interfaces - 망고토마토 - 티스토리, accessed August 9, 2025, https://mangto.tistory.com/97
32. Board Design Guidelines for LVDS Systems, accessed August 9, 2025, https://doc.inmys.ru/open?hash=d5f65801c881592cfa6b99f0a1d128c2&fn=wp_lvdsboard.pdf
33. PCB Design Guidelines for LVDS Technology - UCL HEP Group, accessed August 9, 2025, https://www.hep.ucl.ac.uk/~mp/ELECTRONICS/LVDS_PCBs_App_note_AN-1035.pdf
34. High Speed LVDS PCB Design Guidelines - MADPCB, accessed August 9, 2025, https://madpcb.com/glossary/lvds/
35. LVDS PCB Layout Guidelines | Advanced PCB Design Blog | Cadence, accessed August 9, 2025, https://resources.pcb.cadence.com/blog/2023-lvds-pcb-layout-guidelines
36. AN11088 PTN3460 DP to LVDS PCB layout guidelines - NXP Semiconductors, accessed August 9, 2025, https://www.nxp.com/docs/en/application-note/AN11088.pdf
37. What is an FPD-link? - MD Elektronik, accessed August 9, 2025, https://www.md-elektronik.com/en/fpd-link/
38. FPD-Link SerDes | TI.com - Texas Instruments, accessed August 9, 2025, https://www.ti.com/product-category/interface/high-speed-serdes/fpd-link-serdes/overview.html
39. FPD-Link - SiTime, accessed August 9, 2025, https://www.sitime.com/applications/automotive/fpd-link
40. LVDS Interface IC Market Size, Share & Forecast to 2030, accessed August 9, 2025, https://www.researchandmarkets.com/report/lvds-interface-ic
41. Understanding Consumer Behavior in LVDS Interface ICs Market: 2025-2033, accessed August 9, 2025, https://www.datainsightsmarket.com/reports/lvds-interface-ics-913374