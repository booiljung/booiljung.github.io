# 주파수 도약 확산 스펙트럼(FHSS) 기술


확산 스펙트럼(Spread Spectrum, SS) 통신 기술은 전송하고자 하는 정보 신호의 대역폭보다 훨씬 넓은 주파수 대역으로 신호를 의도적으로 확산시켜 전송하는 기법이다.1 협대역 통신 시스템이 주어진 정보량을 전송하기 위해 최소한의 주파수 대역폭을 사용하는 것과 대조적으로, 확산 스펙트럼은 '정보량 대비 과도한 대역폭 사용'을 통해 부가적인 이득을 얻는 것을 목표로 한다.3 이는 샤논의 채널 용량 정리 $C = B \log_2(1 + S/N)$에서 대역폭($B$)을 인위적으로 증가시켜 신호 대 잡음비($S/N$)가 매우 낮은 열악한 환경에서도 신뢰성 있는 통신을 가능하게 하는 공학적 해법으로 해석될 수 있다.

이 기술의 핵심 목표는 다양하다. 첫째, 의도적이거나 비의도적인 전파 방해(재밍, Jamming) 및 간섭에 대한 강인성을 확보하는 것이다.2 둘째, 신호의 전력 스펙트럼 밀도(spectral power density)를 잡음 레벨 이하로 낮추어 신호의 존재 자체를 숨기는 신호 은닉(signal hiding) 및 낮은 탐지 확률(Low Probability of Intercept, LPI) 특성을 통해 통신 보안을 극대화한다.6 셋째, 동일한 주파수 대역을 여러 사용자가 공유할 수 있도록 하는 다중 접속(Multiple Access) 기능을 제공한다.6 이러한 특성 덕분에 확산 스펙트럼 기술은 군용 통신에서 시작하여 GPS, 3G 이동통신, 무선랜 등 현대 무선 통신의 핵심 기술로 자리 잡았다.5

주파수 도약 확산 스펙트럼(Frequency-Hopping Spread Spectrum, FHSS)은 이러한 확산 스펙트럼 철학을 구현하는 대표적인 방식 중 하나이다. FHSS는 "정해진 시간에 따라 반송파 주파수를 불연속적으로 이동시키면서 통신하는 방법"으로 정의된다.8 마치 토끼가 뛰어다니는 것처럼 주파수가 역동적으로 변화한다고 하여 '주파수 도약'이라는 이름이 붙었다.8 FHSS 통신의 근간에는 송신기와 수신기가 사전에 약속된 의사난수(Pseudo-Noise, PN) 시퀀스를 공유하고, 이 시퀀스에 따라 생성된 동일한 도약 패턴(hopping pattern)으로 완벽하게 동기화하여 주파수를 변경한다는 원리가 있다.1 이 약속된 패턴을 알지 못하는 제3자에게 FHSS 신호는 예측 불가능한 광대역 임펄스 잡음처럼 인식되므로, 신호의 감청이나 해독이 극히 어렵다.7

FHSS와 직접 시퀀스 확산 스펙트럼(Direct Sequence Spread Spectrum, DSSS)은 동일한 확산 스펙트럼 철학을 공유하지만, 간섭에 대응하는 전략에서 근본적인 차이를 보인다. FHSS는 협대역 간섭이 존재하는 특정 주파수를 '회피(avoidance)'하여 다른 깨끗한 주파수로 도약하는 전략을 사용한다. 반면, DSSS는 수신 과정에서 협대역 간섭 신호를 넓은 대역으로 '희석(dilution)'시켜 그 영향을 무력화하는 전략을 취한다. 이러한 전략의 차이는 두 기술의 성능, 복잡도, 그리고 적합한 응용 분야를 결정짓는 핵심적인 요인으로 작용한다.

본 보고서는 FHSS 기술에 대한 포괄적이고 심층적인 분석을 제공하는 것을 목표로 한다. 이를 위해 먼저 FHSS의 역사적 배경과 이론적 기반을 살펴본다. 이후 시스템의 핵심 구조와 동작 원리를 PN 시퀀스 생성, 도약 방식 등을 중심으로 기술적으로 분석한다. 다음으로 처리 이득과 재밍 마진과 같은 핵심 성능 지표를 수학적으로 고찰하고, 시스템 구현의 최대 난제인 동기화 과정을 포착과 추적 단계로 나누어 심도 있게 다룬다. 또한, DSSS와의 비교 분석을 통해 FHSS의 고유한 특성을 명확히 하고, 근원 문제와 같은 실제적 문제점과 적응형 주파수 도약과 같은 고급 기법들을 논한다. 최종적으로 군용 통신부터 블루투스, LoRa에 이르는 다양한 응용 분야와 인지 무선 등 미래 기술과의 융합 가능성을 조망함으로써 FHSS 기술의 본질과 가치에 대한 깊이 있는 이해를 도모하고자 한다.


FHSS 기술의 역사는 '발명(Invention)', '구현(Implementation)', 그리고 '혁신(Innovation)'의 단계를 거치며 발전해왔다. 주파수를 변경하며 통신한다는 개념적 발명은 20세기 초반부터 존재했으나, 이를 구체적인 시스템으로 구현하려는 시도와 반도체 기술의 발전에 힘입어 실용적인 혁신으로 이어진 과정은 기술 발전의 본질을 잘 보여준다.


FHSS 기술의 대중적 기원은 제2차 세계대전 중이던 1941년, 할리우드 배우 헤디 라마(Hedy Lamarr)와 아방가르드 작곡가 조지 앤타일(George Antheil)의 독창적인 아이디어에서 찾을 수 있다.5 이들은 적의 전파 방해로 인해 유도 시스템이 교란되는 것을 막기 위해 연합군 어뢰를 위한 안전한 무선 유도 시스템을 고안했다.12

그들의 시스템은 당시 피아노가 88개의 건반을 가졌다는 사실에 착안하여 88개의 서로 다른 주파수 채널을 사용했다.14 시스템의 핵심은 송신기와 수신기의 주파수 변경을 완벽하게 동기화하는 방법이었는데, 앤타일은 자신의 작곡 경험을 살려 자동 연주 피아노에 사용되는 피아노 롤(player-piano roll)을 동기화 메커니즘으로 제안했다.12 동일한 구멍 패턴이 뚫린 두 개의 피아노 롤을 송신기와 수신기에 각각 장착하고 동시에 회전시키면, 구멍의 위치에 따라 지정된 주파수로 동시에 도약할 수 있다는 원리였다. 이는 오늘날 전자적으로 구현되는 PN 시퀀스 생성기의 기계적인 원형으로 볼 수 있다. 이 아이디어는 "비밀 통신 시스템(Secret Communication System)"이라는 이름으로 1942년 8월 11일, 미국 특허 번호 2,292,387로 등록되었다.5


라마와 앤타일의 특허는 FHSS 기술 역사에서 중요한 이정표이지만, 주파수 도약이라는 개념 자체의 최초 발명은 아니다. 최근의 연구에 따르면, 미국 특허청은 심사 과정에서 이들의 특허 중 가장 포괄적인 개념인 '송수신기의 동시적인 주파수 변경을 통한 비밀 통신 방법'에 대한 청구항 7번(claim 7)을 선행 기술(prior art)이 존재한다는 이유로 기각했다.17

실제로 주파수를 변경하여 통신 보안을 확보하려는 아이디어는 이전부터 존재했다. 대표적으로 니콜라 테슬라(Nikola Tesla)는 1903년 특허에서 신호가 방해받거나 가로채이지 않도록 여러 주파수를 사용하는 시스템을 기술했으며 10, 1930년대에는 네덜란드의 빌렘 브로에티에스(Willem Broertjes)와 독일 지멘스(Siemens)의 마르틴 베제케(Martin Baesecke)가 주파수 변경에 관한 특허를 출원한 바 있다.17 따라서 라마와 앤타일의 진정한 기여는 주파수 도약 개념의 창시라기보다는, 피아노 롤이라는 독창적인 매체를 통해 '동기화'라는 구체적인 기술적 난제를 해결하려 시도한 특정 시스템의 발명으로 재해석되어야 한다.14


라마와 앤타일이 제안한 시스템은 당시 기술 수준의 한계로 인해 즉각적인 실용화로 이어지지 못했다. 미 해군은 "어뢰에 피아노를 탑재할 수 없다"는 식의 비판적인 시각으로 이 제안을 채택하지 않았다.15 기술의 실용화는 개념적 발명뿐만 아니라 이를 뒷받침하는 관련 기반 기술의 성숙도에 크게 의존하는데, 당시에는 피아노 롤과 같은 기계적 장치를 소형 어뢰에 탑재할 만큼 기술이 발전하지 못했던 것이다.

FHSS 기술이 본격적으로 활용되기 시작한 것은 트랜지스터와 집적 회로(IC) 기술의 발달로 전자적인 주파수 합성기와 PN 시퀀스 생성기를 소형화할 수 있게 된 수십 년 후의 일이다. 특히 1960년대 쿠바 미사일 위기 당시, 미 해군 함정들은 통신 보안을 강화하기 위해 FHSS 기술이 적용된 통신 장비를 탑재하기 시작했다.12 이 시점을 계기로 FHSS는 군사 통신의 핵심 기술로 자리 잡았고, 이후 기술이 민간에 이전되면서 블루투스, 무선랜 등 다양한 상업적 응용 분야로 확산되는 혁신의 길을 걷게 되었다.

이 역사적 과정에서 '동기화(Synchronization)'는 FHSS 기술의 탄생 시점부터 현재까지 가장 핵심적인 기술적 난제로 남아있다. 라마와 앤타일이 피아노 롤이라는 기계적 장치를 고안한 것 자체가, 물리적으로 분리된 두 장치가 동일한 의사난수 패턴을 시간 오차 없이 생성하고 추종해야 하는 문제의 근본적인 어려움을 상징적으로 보여준다. 이 동기화 문제는 현대 FHSS 시스템에서도 여전히 포착(acquisition)과 추적(tracking)이라는 복잡한 신호 처리 과정의 형태로 남아 기술의 성능과 복잡도를 결정하는 핵심 요소로 작용하고 있다.


FHSS 시스템은 의사난수(PN) 시퀀스에 의해 제어되는 주파수 합성기를 통해 반송파 주파수를 동적으로 변경함으로써 동작한다. 이 과정의 핵심은 송신기와 수신기가 동일한 도약 패턴을 시간적으로 정확히 일치시켜 유지하는 데 있다. 본 장에서는 FHSS 시스템의 구성 요소, PN 시퀀스 생성 원리, 그리고 주요 도약 방식에 대해 상세히 분석한다.


FHSS 시스템은 크게 송신기와 수신기로 구성되며, 각각의 내부는 정보 신호를 변조하고 주파수를 도약시키는, 또는 그 역과정을 수행하는 기능 블록들로 이루어진다.


FHSS 송신기는 정보 데이터를 협대역 신호로 변조한 후, 이를 PN 시퀀스에 따라 결정된 반송파 주파수에 실어 전송하는 역할을 한다. 그 구조는 다음과 같다.1

1. **정보 소스 (Information Source):** 전송할 디지털 데이터(비트열)를 생성한다.
2. **협대역 변조기 (Narrowband Modulator):** 입력된 비트열을 특정 변조 방식에 따라 협대역 아날로그 신호로 변환한다. FHSS 시스템에서는 M-ary 주파수 편이 변조(MFSK)가 일반적으로 사용되는데, 이는 주파수 기반의 변조 방식이 주파수를 계속 변경하는 시스템의 특성과 잘 부합하기 때문이다.21
3. **PN 시퀀스 생성기 (PN Sequence Generator):** 미리 약속된 규칙(예: 원시 다항식)과 초기값(seed)을 바탕으로 도약 패턴을 결정하는 의사난수 비트 시퀀스를 생성한다. 이 시퀀스가 통신의 보안성과 항재밍 성능을 결정하는 핵심 요소이다.21
4. **주파수 합성기 (Frequency Synthesizer):** PN 시퀀스 생성기로부터 특정 시간 간격(홉 주기, hop period)마다 새로운 비트 그룹을 입력받아, 해당 값에 매핑되는 특정 반송파 주파수를 생성한다. 주파수 합성기의 성능, 특히 주파수 전환 속도(settling time)와 정확성은 FHSS 시스템의 전체 성능을 좌우한다.20
5. **혼합기 (Mixer) 및 대역 통과 필터 (BPF):** 협대역 변조된 신호를 주파수 합성기에서 생성된 반송파와 혼합(곱셈)한다. 이 과정을 통해 변조 신호의 중심 주파수가 도약 주파수로 이동하게 된다. 이후 대역 통과 필터는 원하는 주파수 성분만을 통과시켜 최종 송신 신호를 생성한다.20
6. **안테나 (Antenna):** 최종 생성된 FHSS 신호를 무선 채널로 방사한다.

이처럼 FHSS 시스템의 핵심은 '주파수 합성기'의 성능에 집약된다고 할 수 있다. PN 시퀀스가 '어떤 주파수로 도약할지'를 결정하는 논리적 두뇌라면, 주파수 합성기는 이를 물리적 신호로 얼마나 빠르고 정확하게 구현하는지를 결정하는 '심장'과 같은 역할을 수행한다.


FHSS 수신기는 송신 과정의 정확한 역순을 수행하여 원래의 정보 데이터를 복원한다. 이를 위해 송신기와 완벽하게 동기화된 PN 시퀀스 생성기와 주파수 합성기를 보유해야 한다.1

1. **안테나 (Antenna):** 무선 채널로부터 FHSS 신호를 수신한다.
2. **PN 시퀀스 생성기 및 주파수 합성기:** 송신기와 동일한 PN 시퀀스 생성기가 동일한 초기값으로 동작하며, 주파수 합성기는 이로부터 동일한 도약 주파수 패턴을 생성한다. 이 과정의 성공 여부는 동기화에 달려있다.20
3. **혼합기 (Mixer) 및 대역 통과 필터 (BPF):** 수신된 신호를 로컬 주파수 합성기에서 생성된 주파수와 혼합한다. 만약 송수신기가 완벽히 동기화되었다면, 이 과정을 통해 주파수 도약 성분이 제거되고 원래의 협대역 변조 신호가 일정한 중간 주파수(IF) 또는 기저대역(baseband) 신호로 복원된다. 이 과정을 역도약(Dehopping)이라 한다.20 동기가 맞지 않는 신호는 이 과정을 거쳐도 여전히 광대역 잡음으로 남게 되어 후속 필터에서 걸러진다.1
4. **협대역 복조기 (Narrowband Demodulator):** 역도약된 협대역 신호를 복조하여 원래의 디지털 데이터(비트열)를 복원한다.19


FHSS의 보안성과 성능은 도약 패턴의 예측 불가능성에 크게 의존하며, 이는 의사난수(PN) 시퀀스의 특성에 의해 결정된다. 이상적인 PN 시퀀스는 다음과 같은 조건을 만족해야 한다.7

- **랜덤성 (Randomness):** 시퀀스의 통계적 특성이 실제 난수와 유사하여 다음 비트를 예측하기 어려워야 한다.
- **긴 주기 (Long Period):** 시퀀스가 반복되기까지의 길이가 매우 길어야 외부에서 패턴을 파악하기 어렵다.
- **우수한 자기상관 (Good Autocorrelation):** 시퀀스를 시간적으로 이동시키지 않았을 때(zero shift)의 상관 값은 최대가 되고, 그 외의 모든 이동에 대해서는 상관 값이 매우 낮아야 한다. 이 특성은 동기화 과정에서 정확한 시간 정렬을 찾는 데 결정적인 역할을 한다.

이러한 조건을 만족하는 대표적인 PN 시퀀스로 최대 길이 시퀀스(m-sequence)가 있으며, 이는 주로 선형 귀환 시프트 레지스터(Linear Feedback Shift Register, LFSR)를 통해 생성된다.


LFSR은 `n`개의 플립플롭(D-플립플롭)으로 구성된 시프트 레지스터와, 레지스터의 특정 위치(탭, tap)에 있는 값들을 XOR(배타적 논리합) 연산하여 피드백하는 구조로 이루어져 있다.23 이 간단한 구조는 매우 긴 주기의 결정론적 시퀀스를 효율적으로 생성할 수 있다.26

LFSR의 동작을 결정하는 핵심은 피드백 경로를 정의하는 '특성 다항식(characteristic polynomial)'이다. `n`차 다항식 중에서 특정 조건을 만족하는 '원시 다항식(primitive polynomial)'을 사용하면, LFSR은 가능한 모든 비제로(non-zero) 상태를 단 한 번씩만 거친 후 초기 상태로 돌아오는 최대 주기, 즉 $2^n - 1$의 길이를 갖는 m-시퀀스를 생성할 수 있다.25 예를 들어, 원시 다항식 $P(x) = x^4 + x + 1$은 4단 LFSR의 4번 탭(출력)과 1번 탭의 출력을 XOR하여 첫 번째 플립플롭의 입력으로 피드백하는 구조를 의미한다. 이 경우 생성되는 m-시퀀스의 주기는 $2^4 - 1 = 15$가 된다. 송신기와 수신기가 동일한 원시 다항식과 초기 상태(seed)를 공유함으로써, 완전히 동일한 도약 패턴을 생성하고 유지할 수 있게 된다.24


FHSS 시스템은 도약 속도($R_h$, hops/sec)와 데이터 심볼 속도($R_s$, symbols/sec)의 상대적 관계에 따라 저속 주파수 도약(SFH)과 고속 주파수 도약(FFH)으로 분류된다. 이 선택은 시스템의 항재밍 성능, 복잡도, 그리고 에러 제어 전략에 직접적인 영향을 미친다.21


SFH는 하나의 주파수 채널에 머무는 동안 여러 개의 데이터 심볼이 전송되는 방식, 즉 $R_h < R_s$인 경우를 의미한다.21 예를 들어, 한 번의 주파수 도약 동안 10개의 심볼이 전송된다면 이는 SFH에 해당한다.

- **장점:** 주파수 합성기가 상대적으로 느리게 동작해도 되므로 시스템 구현이 간단하고 비용이 저렴하다.
- **단점:** 만약 현재 사용 중인 주파수 채널이 깊은 페이딩(fading)이나 강력한 협대역 재밍에 노출되면, 해당 홉 주기 동안 전송되는 모든 심볼이 한꺼번에 손실될 수 있다. 이는 '버스트 에러(bursty error)'를 유발하며, 이를 복구하기 위해 인터리빙(interleaving)이나 강력한 순방향 에러 정정(Forward Error Correction, FEC) 코드와 같은 복잡한 채널 코딩 기술이 요구된다.30


FFH는 하나의 데이터 심볼이 전송되는 시간 동안 여러 번의 주파수 도약이 발생하는 방식, 즉 $R_h > R_s$인 경우를 말한다.20 예를 들어, 하나의 심볼을 전송하기 위해 주파수를 4번 도약시킨다면 이는 FFH에 해당한다.

- **장점:** 각 심볼의 에너지가 여러 주파수 채널에 분산되어 전송되므로 '주파수 다이버시티(frequency diversity)' 효과를 얻을 수 있다.21 이 덕분에 특정 주파수가 재밍에 노출되더라도 심볼의 일부 에너지 만이 손실되므로, 협대역 재밍이나 주파수 선택적 페이딩에 대해 매우 강인한 성능을 보인다. 에러가 시간 축에 분산되어 발생하므로 상대적으로 간단한 FEC 코드로도 효과적인 에러 제어가 가능하다.
- **단점:** 데이터 심볼보다 빠른 속도로 주파수를 전환해야 하므로 매우 빠른 고성능의 주파수 합성기가 필요하다. 이는 시스템의 구현 복잡도와 비용을 크게 증가시키는 요인이 된다.

결론적으로, SFH와 FFH의 선택은 시스템 설계 시 중요한 트레이드오프 관계를 형성한다. SFH는 구현의 단순성을 얻는 대신 버스트 에러에 대응하기 위한 강력한 채널 코딩을 요구하는, 즉 데이터 링크 계층의 부담을 늘리는 방식이다. 반면, FFH는 물리 계층의 복잡도를 높여 주파수 다이버시티를 확보함으로써 채널 코딩의 부담을 줄이는 방식이다. 이처럼 물리 계층(PHY)의 특성과 데이터 링크 계층(DLL)의 에러 제어 전략은 상호 밀접하게 연관되어 있다.


확산 스펙트럼 시스템의 성능은 외부 간섭이나 재밍 환경 하에서 얼마나 신뢰성 있는 통신을 유지할 수 있는가에 의해 평가된다. FHSS 시스템의 항재밍 성능을 정량적으로 나타내는 핵심 지표는 처리 이득(Processing Gain)과 재밍 마진(Jamming Margin)이다.


처리 이득은 확산 스펙트럼 기술을 사용함으로써 얻게 되는 신호 대 잡음비(SNR)의 개선 정도를 나타내는 척도이다.31 이는 시스템이 얼마나 효과적으로 간섭 신호를 억제할 수 있는지를 보여준다. FHSS 시스템에서 처리 이득은 전체 시스템이 사용하는 총 주파수 대역폭($W_{ss}$, Spread Spectrum Bandwidth)을 원래 정보 신호의 대역폭($R_s$, Symbol Rate 또는 Information Bandwidth)으로 나눈 비율로 정의된다.31
$$
PG = \frac{W_{ss}}{R_s}
$$
이 값은 시스템이 사용할 수 있는 총 주파수 채널의 수($N$)와 거의 동일한 의미를 갖는다. 예를 들어, `k` 비트의 PN 시퀀스 출력이 하나의 도약 주파수를 결정한다면, 시스템은 최대 $2^k$개의 채널을 사용할 수 있으며, 이 값이 곧 처리 이득이 된다.29 예를 들어, 전체 도약 대역폭이 10 MHz이고 정보 심볼 속도가 1 ksps(정보 대역폭 1 kHz)인 FHSS 시스템이 있다면, 처리 이득은 $10,000,000 / 1,000 = 10,000$이며, 이를 데시벨(dB)로 환산하면 $10 \log_{10}(10000) = 40 \text{ dB}$가 된다.31 이는 수신기에서 역도약 과정을 거친 후의 SNR이 입력단 SNR보다 이론적으로 최대 40 dB까지 개선될 수 있음을 의미한다.

FHSS의 처리 이득 개념은 DSSS와는 다른 방식으로 이해되어야 한다. DSSS는 수신단의 상관기(correlator)를 통해 간섭 신호의 전력을 실시간으로 넓은 대역에 분산시켜 처리 이득을 얻는 '신호 처리적 이득'이다. 반면, FHSS의 처리 이득은 '시간 평균적(time-averaged)' 개념에 가깝다. 즉, 협대역 재머가 특정 주파수 $f_j$에 존재할 때, FHSS 신호는 전체 $N$개의 채널 중 오직 하나의 채널에서만 재머와 충돌한다. 따라서 전체 통신 시간 중 재머의 영향을 받는 시간의 비율은 평균적으로 $1/N$에 불과하다.31 이처럼 간섭을 겪는 시간을 최소화함으로써 평균적인 간섭 효과를 줄이는 것이 FHSS 처리 이득의 본질이다. 이러한 동작 방식의 차이는 FHSS가 간헐적인 버스트 에러를 발생시키는 반면, DSSS는 배경 잡음 레벨이 전반적으로 상승하는 형태로 간섭의 영향을 받는 특성의 차이를 낳는다.


재밍 마진은 처리 이득이라는 이론적 지표를 바탕으로, 실제 시스템의 손실과 요구 성능을 고려하여 시스템이 견딜 수 있는 최대 간섭의 수준을 나타내는 보다 실질적인 항재밍 성능 지표이다.29 재밍 마진은 보통 재밍 신호의 전력($J$)과 유효 신호의 전력($S$)의 비율($J/S$)로 표현되며, 이 값이 클수록 더 강한 재밍을 견딜 수 있음을 의미한다.

재밍 마진($M_J$)은 처리 이득($G_p$), 시스템 구현 과정에서 발생하는 총 손실($L_{sys}$), 그리고 목표 비트 에러율(BER)을 달성하기 위해 복조기 출력단에서 요구되는 최소 신호 대 잡음비($(SNR)_{out}$ 또는 $(E_b/N_0)_{min}$)에 의해 결정된다.29 각 항목을 dB 단위로 표현할 때의 관계식은 다음과 같다.
$$
M_J \text{} = G_p \text{} - L_{sys} \text{} - (SNR)_{out} \text{}
$$
예를 들어, 어떤 FHSS 시스템의 처리 이득이 40 dB이고, 시스템의 총 손실이 2 dB, 그리고 사용된 변조 방식(예: BPSK)이 $10^{-5}$의 BER을 만족하기 위해 11 dB의 $(SNR)_{out}`을 요구한다고 가정하자. 이 시스템의 재밍 마진은 $M_J = 40 - 2 - 11 = 27 \text{ dB}$로 계산된다. 이는 재밍 신호의 전력이 유효 신호의 전력보다 27 dB (약 500배) 더 강하더라도 시스템이 목표 성능을 유지하며 통신할 수 있다는 것을 의미한다.33

이 재밍 마진 공식은 항재밍 시스템 설계 시 발생하는 근본적인 트레이드오프 관계를 명확히 보여준다. 재밍 마진을 높이기 위해 처리 이득($G_p$)을 증가시키려면 더 넓은 주파수 대역폭($W_{ss}$)이 필요하며, 이는 비용 증가와 주파수 규제 문제를 야기한다. 반대로, 높은 데이터 전송률을 위해 스펙트럼 효율이 좋은 고차 변조 방식(예: 16-QAM)을 사용하면, 요구되는 $(SNR)_{out}` 값이 커지므로 재밍 마진이 감소하게 된다. 따라서 시스템 설계자는 주어진 환경과 요구사항 하에서 통신의 '강인성(robustness)'과 '효율성(efficiency)' 사이의 최적점을 찾아야 하는 과제에 직면하게 된다.


FHSS 시스템의 이론적 성능을 실제로 구현하기 위한 가장 중요하고 어려운 기술적 과제는 동기화(Synchronization)이다. 동기화란 수신기에 내장된 로컬 PN 시퀀스 생성기의 상태(타이밍과 패턴)를 수신된 신호에 포함된 PN 시퀀스의 상태와 정확하게 일치시키는 과정이다.35 동기화가 이루어지지 않으면 역도약 과정이 실패하여 신호 복원이 불가능하므로, FHSS 통신은 성립될 수 없다.35

동기화 과정은 일반적으로 두 단계, 즉 초기 동기를 획득하는 '포착(Acquisition)' 단계와 획득된 동기를 정밀하게 유지하는 '추적(Tracking)' 단계로 구성된다.36


포착은 수신기가 송신기의 PN 시퀀스 타이밍에 대한 아무런 정보가 없는 상태에서 시작하여, 시간과 주파수의 불확실성 영역(uncertainty region)을 탐색하여 두 시퀀스 간의 시간 오차를 대략적으로 칩(chip) 단위 이내로 줄이는 과정이다.36 동기화 불확실성을 야기하는 주요 요인으로는 송수신기 간의 거리로 인한 전파 지연, 양단 클럭의 불안정성으로 인한 시간 드리프트, 그리고 상대적인 움직임으로 인한 도플러 효과 등이 있다.36

포착 과정은 '탐색 공간(Search Space)'의 크기와 '탐색 시간(Search Time)' 간의 근본적인 트레이드오프 관계를 보여준다. 이 트레이드오프를 어떻게 해결하느냐에 따라 다양한 포착 기법이 존재한다.


직렬 탐색은 가장 기본적인 포착 방식으로, 하나의 상관기(correlator) 또는 정합 필터를 사용하여 가능한 모든 코드 위상(code phase)을 순차적으로 하나씩 검사하는 방법이다.36 수신기는 로컬 PN 코드의 위상을 특정 단위(보통 칩 길이의 1/2)만큼 이동시킨 후, 일정 시간(search dwell time) 동안 수신 신호와의 상관 값을 계산한다. 이 상관 값이 미리 설정된 임계값(threshold)을 넘어서면 포착이 성공한 것으로 간주하고 탐색을 멈춘 뒤 추적 단계로 넘어간다. 만약 임계값을 넘지 못하면, 로컬 코드의 위상을 다시 이동시켜 다음 위상을 검사하는 과정을 반복한다.36

이 방식은 하드웨어 구현이 매우 간단하고 저렴하다는 큰 장점이 있지만, 탐색해야 할 불확실성 영역이 넓을 경우 모든 위상을 순차적으로 검사해야 하므로 포착에 매우 오랜 시간이 걸릴 수 있다는 치명적인 단점이 있다.36


병렬 탐색은 직렬 탐색의 느린 포착 속도 문제를 해결하기 위한 방식으로, 탐색 공간 내의 가능한 모든 코드 위상에 해당하는 상관기 또는 정합 필터를 병렬로 배치하여 동시에 검사하는 방법이다.36 모든 위상에 대한 상관 값이 동시에 계산되므로, 이 중 가장 큰 값을 가지는 출력을 선택함으로써 단 한 번의 연산으로 최적의 위상을 찾아낼 수 있다.

이 방식은 이론적으로 가장 빠른 포착 속도를 제공하지만, 불확실성 영역에 포함된 위상의 수만큼 상관기를 구현해야 하므로 하드웨어 복잡도, 크기, 그리고 비용이 기하급수적으로 증가하여 실제 시스템에 전면적으로 적용하기는 어렵다.36 따라서 실제 시스템에서는 제한된 수의 병렬 채널을 사용하는 하이브리드 방식이나, 특정 동기화 패턴을 사용하여 탐색 공간을 줄이는 기법 등이 함께 사용된다.36


포착 단계를 통해 대략적인 동기가 이루어지면, 시스템은 추적 단계로 전환하여 미세한 시간 오차를 지속적으로 감지하고 보정함으로써 동기를 정밀하게 유지한다.36 추적은 일반적으로 제어 이론의 음성 피드백(Negative Feedback) 원리를 이용하는 폐쇄 루프(closed-loop) 시스템으로 구현된다. 시스템은 현재의 시간 오차를 측정하여 '에러 신호'를 생성하고, 이 에러 신호를 이용해 로컬 PN 시퀀스 생성기의 클럭 주파수를 미세 조정함으로써 오차를 0으로 수렴시킨다.


DLL은 가장 널리 사용되는 추적 루프 구조이다.39 DLL은 기존의 로컬 PN 시퀀스 외에, 이보다 시간적으로 약간 앞선 '조금 이른(early)' 시퀀스와 약간 뒤처진 '조금 늦은(late)' 시퀀스를 추가로 생성한다. 일반적으로 early와 late 시퀀스는 기준 시퀀스로부터 각각 칩 길이의 절반($T_c/2$)만큼 떨어져 있다.36

수신된 신호는 early 시퀀스와 late 시퀀스 각각과 상관 연산을 수행한다. 만약 로컬 시퀀스가 수신 신호와 완벽히 정렬되어 있다면, early 상관 값과 late 상관 값의 크기는 동일할 것이다. 만약 로컬 시퀀스가 수신 신호보다 앞서 있다면 late 상관 값이 더 커질 것이고, 뒤처져 있다면 early 상관 값이 더 커질 것이다. DLL은 이 두 상관 값의 차이를 계산하여 에러 신호를 생성한다. 이 에러 신호는 루프 필터를 거쳐 전압 제어 발진기(VCO)의 입력으로 전달되어 로컬 시퀀스 생성기의 클럭 속도를 조절한다. 이 피드백 과정을 통해 루프는 시간 오차를 지속적으로 최소화하며 안정된 추적 상태를 유지한다.36 DLL의 한 가지 단점은 early와 late 신호 경로(arm) 간의 이득이 완벽하게 균형을 이루어야 한다는 점이다. 이득 불균형이 존재할 경우, 실제 오차가 0임에도 불구하고 에러 신호가 0이 아닌 값을 갖게 되어 추적 오차를 유발할 수 있다.36


TDL은 DLL의 이득 불균형 문제를 해결하기 위해 고안된 구조이다.39 TDL은 두 개의 상관기를 사용하는 대신, 단 하나의 상관기를 시분할 방식으로 사용한다. 즉, 로컬 PN 시퀀스 생성기가 'dithering' 신호에 따라 early와 late 상태를 빠르게 번갈아 가며(toggle) 생성하고, 상관기는 이에 맞춰 상관 값을 계산한다. 이후 동기식 검출기를 통해 early와 late에 해당하는 상관 값을 분리하고 차이를 계산하여 에러 신호를 생성한다.

하나의 신호 경로만을 사용하므로 이득 불균형 문제가 원천적으로 해결된다는 장점이 있다. 하지만, 한정된 시간 동안 early와 late 상태를 모두 처리해야 하므로 신호 당 유효 적분 시간이 절반으로 줄어들어, 동일한 조건의 DLL에 비해 잡음 성능이 이론적으로 3 dB 저하되는 단점이 있다.40


FHSS와 DSSS는 확산 스펙트럼 기술을 구현하는 양대 산맥으로, 동일한 목표를 추구하지만 상이한 접근 방식을 취한다. 각 기술의 특성을 이해하는 것은 특정 응용 분야에 어떤 기술이 더 적합한지를 판단하는 데 매우 중요하다.7 본 장에서는 두 기술을 다양한 기준으로 비교 분석한다.

두 기술의 가장 근본적인 차이는 '간섭 대응 방식'에 있다. FHSS는 간섭이 존재하는 주파수를 능동적으로 '회피'하는 전략을 사용한다. 이는 마치 지뢰밭에서 지뢰가 없는 곳으로만 뛰어다니는 것과 같다. 반면, DSSS는 간섭 신호를 수신 과정에서 넓은 대역으로 '희석'시켜 그 영향을 최소화하는 전략을 사용한다. 이는 물 한 컵에 잉크 한 방울을 떨어뜨려 전체적으로 옅게 만드는 것과 유사하다. 이러한 전략의 차이가 아래 표에 요약된 다양한 성능 특성의 차이를 유발한다.

| **항목 (Category)**      | **주파수 도약 확산 스펙트럼 (FHSS)**                         | **직접 시퀀스 확산 스펙트럼 (DSSS)**                         | **관련 자료 (Snippets)** |
| ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------ |
| **기본 원리**            | 협대역 신호를 의사난수 패턴에 따라 여러 주파수 채널로 빠르게 '도약'시킴. | 정보 비트열을 더 빠른 속도의 PN 코드로 곱하여 광대역 신호로 '직접' 확산시킴. | 4                        |
| **대역폭 사용**          | 순간적으로는 협대역을 점유하지만, 시간 평균적으로 넓은 대역을 사용. 대역폭 효율성이 상대적으로 높음. | 항상 넓은 대역폭을 동시에 점유. 대역폭 효율성이 상대적으로 낮음. | 7                        |
| **데이터 전송률**        | 일반적으로 DSSS보다 낮음 (예: 초기 802.11에서 ~3 Mbps).      | 일반적으로 FHSS보다 높음 (예: 802.11b에서 ~11 Mbps).         | 30                       |
| **간섭 저항성**          | **협대역 간섭에 매우 강함 (회피 전략).** 간섭 채널을 피해 도약하므로 강력한 단일 톤 재밍에 효과적임. | **광대역 잡음 및 약한 간섭에 강함 (희석 전략).** 처리 이득만큼 간섭 신호의 전력 밀도를 낮춤. | 6                        |
| **다중 경로 페이딩**     | 주파수를 계속 바꾸므로 특정 주파수의 널(null)에 빠질 확률이 낮아 다중 경로 페이딩에 덜 민감함. | 레이크(Rake) 수신기를 통해 다중 경로 성분을 에너지원으로 활용할 수 있으나, 구현이 복잡함. | 7                        |
| **동기화**               | 주파수 동기화가 필요하지만, 코드 위상 동기화는 DSSS보다 상대적으로 포착 시간이 짧고 용이함. | 긴 PN 코드 전체의 위상을 정밀하게 정렬해야 하므로 초기 포착 시간이 길고 복잡함. | 31                       |
| **근원 문제 (Near-Far)** | 상대적으로 덜 민감함. 사용자들이 다른 주파수 채널을 사용할 확률이 높아 강한 신호의 간섭을 피할 수 있음. | **매우 민감함.** 가까운 송신기의 강한 신호가 먼 송신기의 약한 신호를 완전히 덮어버릴 수 있어 정밀한 전력 제어가 필수적임. | 6                        |
| **보안성/LPI**           | 도약 패턴을 모르면 신호의 존재 자체를 탐지하기 어려워 보안성이 우수함. | 신호가 넓은 대역에 걸쳐 잡음처럼 분포하여 전력 밀도가 낮으므로 LPI 특성이 우수함. | 1                        |
| **구현 복잡도/비용**     | 고속 주파수 합성기(synthesizer)의 성능이 중요하며, 이로 인해 비용이 증가할 수 있음. | 고속 디지털 신호 처리(상관기)가 핵심이며, 반도체 기술 발달로 저가 구현이 용이해짐. | 1                        |
| **주요 응용**            | 군용 통신, 블루투스, LoRa, 초기 IEEE 802.11                  | CDMA 셀룰러 통신, Wi-Fi (802.11b/g/n), GPS                   | 1                        |

이러한 기술적 특성의 차이는 각 기술이 채택되는 네트워크의 토폴로지 및 운용 환경과 밀접한 관련이 있다. 예를 들어, FHSS는 근원 문제에 강하고 상대적으로 동기화가 용이하여, 중앙 집중적인 전력 제어가 어려운 애드혹(Ad-hoc) 또는 P2P(Peer-to-Peer) 네트워크 환경에 매우 적합하다. 블루투스가 다수의 기기가 근거리에서 자율적으로 연결되는 환경에서 FHSS를 채택한 것이 대표적인 사례이다.1 반면, DSSS는 기지국이라는 강력한 중앙 제어 노드가 존재하여 모든 단말기의 송신 전력을 정밀하게 제어할 수 있는 셀룰러 네트워크 환경에서 그 잠재력을 최대한 발휘할 수 있다. CDMA 이동통신 시스템은 DSSS 기술과 정밀한 전력 제어를 결합하여 높은 시스템 용량을 달성한 성공적인 사례이다.42

또한, 두 기술의 간섭 대응 방식의 차이는 혼잡한 주파수 환경에서의 공존(coexistence) 능력에도 영향을 미친다. FHSS는 다른 시스템이 사용하는 특정 주파수 채널을 '블랙리스트'로 만들어 도약 패턴에서 제외하는 적응형 기법을 적용하기 용이하다. 이는 블루투스의 적응형 주파수 도약(AFH) 기능에서 볼 수 있듯이, 정적인 간섭원에 효과적으로 대응하는 능력을 제공한다.44 반면, DSSS는 간섭원의 특성을 파악하기보다는 자신의 처리 이득을 이용해 간섭을 '견디는' 수동적인 방식을 취한다. 이는 일정 수준 이하의 간섭에 대해서는 안정적인 성능을 보장하지만, 처리 이득을 초과하는 강력한 간섭에 대해서는 시스템 전체가 마비될 수 있는 취약점을 가진다.


FHSS 시스템은 이론적으로 우수한 성능을 제공하지만, 실제 무선 채널 환경에서 운용될 때는 여러 가지 문제점에 직면하게 된다. 이러한 문제들을 해결하고 성능을 최적화하기 위해 다양한 고급 구현 기법들이 개발되었다. 본 장에서는 대표적인 문제점인 근원 문제와 이를 해결하기 위한 기법, 그리고 FHSS 기술의 지능적 진화 형태인 적응형 주파수 도약에 대해 논한다.


근원 문제란 다중 접속 환경에서 수신기에 가까이 위치한 송신기(near user)의 강한 신호가 멀리 위치한 송신기(far user)의 약한 신호를 압도하여, 수신기가 약한 신호를 정상적으로 복조하지 못하는 현상을 말한다.42 이는 수신 전력이 거리에 따라 급격히 감쇠하는 무선 채널의 기본적인 물리 특성 때문에 발생한다.

FHSS 시스템은 사용자들이 서로 다른 주파수 채널을 사용하므로 DSSS/CDMA 시스템에 비해 근원 문제에 본질적으로 더 강인한 특성을 보인다.6 그러나 여러 사용자가 우연히 동일한 주파수 채널에서 동시에 전송하는 경우(이를 '히트(hit)'라고 한다), 근원 문제가 발생할 수 있다. 이 경우, 수신기는 일반적으로 더 강한 신호에 동기화되어 해당 신호만을 복조하게 되는데, 이를 캡처 효과(capture effect)라고 한다.42

이러한 문제를 완화하기 위해 다음과 같은 기법들이 사용된다.

- **전력 제어 (Power Control):** 가장 근본적이고 효과적인 해결책으로, 모든 송신기가 자신의 송신 전력을 동적으로 조절하여 각 신호가 수신기에 거의 동일한 전력 수준으로 도달하도록 만드는 기법이다.42 이를 통해 약한 신호가 강한 신호에 의해 가려지는 현상을 방지할 수 있다. 하지만 애드혹 네트워크와 같이 중앙 제어 노드가 없는 환경에서는 정밀한 전력 제어를 구현하기 어렵다는 한계가 있다.
- **다중 접속 프로토콜과의 결합:** FHSS를 다른 다중 접속 방식과 결합하여 충돌 자체를 줄이는 방법이다. 예를 들어, FH-TDMA(Time Division Multiple Access) 시스템에서는 각 사용자가 자신에게 할당된 시간 슬롯에서만 전송하므로, 동일 주파수에서의 동시 전송 확률을 크게 낮출 수 있다.42 이는 GSM 이동통신 시스템에서 성공적으로 사용된 방식이다.

FHSS의 근원 문제에 대한 상대적 강인성은 '직교성(Orthogonality)' 개념으로 설명될 수 있다. 이상적인 FHSS 시스템에서 서로 다른 사용자는 서로 다른 주파수 채널(주파수 직교성)을 사용하거나, TDMA와 결합될 경우 서로 다른 시간 슬롯(시간 직교성)을 사용한다. 이처럼 자원을 직교적으로 분할함으로써 사용자 간 간섭을 원천적으로 차단할 수 있다. 반면, DSSS는 동일한 시간과 주파수를 공유하며 오직 '코드(Code)'를 통해 사용자를 구분한다. 사용되는 PN 코드가 완벽한 직교성을 갖지 못하고 상호상관(cross-correlation) 값이 존재하기 때문에(준직교성, quasi-orthogonality), 다중 접속 간섭(MAI)이 항상 존재하며 이것이 근원 문제의 형태로 심각하게 발현되는 것이다.47


전통적인 FHSS 시스템은 외부 환경과 무관하게 미리 정해진 고정된 도약 패턴을 사용한다. 그러나 실제 통신 환경, 특히 2.4 GHz ISM(Industrial, Scientific, and Medical) 대역과 같이 비면허 대역에서는 Wi-Fi, 전자레인지 등 다양한 간섭원이 존재한다. 이러한 환경에서 고정된 패턴을 사용하면 특정 간섭 주파수에 반복적으로 충돌하여 통신 품질이 심각하게 저하될 수 있다.

적응형 주파수 도약(AFH)은 이러한 문제를 해결하기 위해 등장한 지능형 기법이다.18 AFH의 핵심 아이디어는 채널 상태를 지속적으로 감지하고 학습하여, 간섭이 심하거나 채널 품질이 나쁜 '불량 채널(bad channels)'을 식별하고 이를 도약 패턴에서 동적으로 제외하는 것이다.49

블루투스는 AFH를 성공적으로 구현한 대표적인 사례이다.44 블루투스 시스템에서 마스터 장치는 각 주파수 채널의 통신 품질을 패킷 에러율(PER), 수신 신호 강도 지표(RSSI) 등 다양한 지표를 통해 지속적으로 모니터링한다. 이 정보를 바탕으로 각 채널을 'Good' 또는 'Bad'로 분류하는 채널 맵(channel map)을 유지한다.44 통신 시에는 이 채널 맵을 참조하여 'Bad'로 분류된 채널을 건너뛰고 'Good' 채널들로만 구성된 새로운 도약 시퀀스를 생성하여 사용한다. 이 채널 맵은 피코넷 내의 슬레이브 장치들과 공유되어 모든 기기가 동일한 적응형 패턴으로 동작하게 된다.44

AFH는 FHSS 기술의 중요한 진화로 평가받는다. 이는 FHSS를 단순한 물리 계층(PHY) 기술에서 '인지 무선(Cognitive Radio)'의 초기 형태로 발전시켰기 때문이다. AFH는 주변 전파 환경을 '감지(sense)'하고, 채널 상태를 '판단(decide)'하며, 이에 따라 자신의 동작(도약 패턴)을 동적으로 '적응(adapt)'하는 인지 사이클의 기본 요소를 모두 포함하고 있다.51 이처럼 AFH는 정적인 규칙에 따라 동작하던 시스템을 동적인 환경과 상호작용하는 지능형 시스템으로 변화시키는 중요한 전환점이 되었다.


FHSS 기술은 그 고유한 강인성과 보안성을 바탕으로 군용 통신에서부터 일상생활의 무선 기기에 이르기까지 다양한 분야에서 핵심적인 역할을 수행해왔다. 또한, 차세대 통신 기술과의 융합을 통해 그 가치를 계속해서 확장해 나가고 있다.

8.1 주요 응용 사례

FHSS 기술의 생명력은 '강인성(Robustness)'과 '단순성(Simplicity)'의 독특한 조합에서 나온다. 초고속 데이터 전송률보다는 저전력, 저비용, 그리고 안정적인 연결이 더 중요한 특정 응용 분야에서 FHSS는 대체 불가능한 솔루션을 제공한다.

- **군용 통신:** FHSS의 태생적 목적으로, 강력한 항재밍 성능과 낮은 탐지 확률(LPI) 특성은 적의 전자전 공격으로부터 아군의 통신망을 보호하는 데 필수적이다. 전술 통신, 무인기 제어, 보안 데이터 링크 등 다양한 군사 응용 분야에서 여전히 핵심 기술로 사용된다.1
- **초기 무선 LAN (IEEE 802.11):** 1997년에 제정된 최초의 Wi-Fi 표준은 물리 계층 규격으로 DSSS와 함께 FHSS를 포함했다.1 당시 FHSS는 DSSS보다 다중 접속 환경에서의 간섭에 더 강인하다는 장점이 있었으나, 이후 더 높은 데이터 전송률을 제공하는 DSSS 기반의 802.11b 표준이 시장의 주류가 되면서 점차 사용되지 않게 되었다.47
- **블루투스 (Bluetooth):** FHSS 기술이 가장 성공적으로 상용화된 사례이다. 블루투스는 스마트폰, 이어폰, 키보드 등 다양한 기기들이 근거리에서 자율적으로 연결되는 개인 통신망(WPAN)을 목표로 설계되었다. 이러한 애드혹 환경에서는 중앙 집중적인 전력 제어가 어려워 근원 문제에 취약한 DSSS보다 FHSS가 훨씬 적합했다.1 블루투스는 2.4 GHz ISM 대역의 79개 채널을 초당 1600번 도약하는 빠른 도약(Fast Hopping) 방식을 채택하여, 혼잡한 전파 환경에서도 안정적인 연결을 제공한다.44
- **LoRa (Long Range):** 사물 인터넷(IoT)을 위한 대표적인 저전력 광역 통신망(LPWAN) 기술 중 하나이다. LoRa는 기본적으로 처프 확산 스펙트럼(CSS)을 사용하지만, 일부 지역의 규제(예: 채널 최대 점유 시간 제한)를 준수하고 통신의 신뢰성을 높이기 위해 FHSS 모드를 선택적으로 사용한다. 긴 데이터 패킷을 전송해야 할 경우, 패킷을 여러 조각으로 나누어 각 조각을 다른 주파수 채널로 도약하며 전송함으로써 채널 점유 시간 규정을 만족시키고 간섭에 대한 강인성을 확보한다.11


미래 통신 환경에서 FHSS의 역할은 단독 기술로서보다는 다른 첨단 기술과 융합된 '구성 요소(building block)'로서 더욱 중요해질 전망이다. FHSS의 '도약'이라는 핵심 메커니즘은 다른 기술의 약점을 보완하고 새로운 기능을 부여하는 강력한 도구가 될 수 있다.

- **인지 무선(Cognitive Radio)과 동적 스펙트럼 접속(DSA):** 한정된 주파수 자원의 효율적 사용을 위해, 기존 사용자가 없는 비어있는 주파수 대역(spectrum hole)을 2차 사용자가 동적으로 찾아 사용하는 동적 스펙트럼 접속(DSA) 패러다임에서 FHSS는 핵심적인 역할을 할 수 있다.55 인지 무선 단말기는 지속적으로 스펙트럼을 감지하여 사용 가능한 채널 목록을 동적으로 구성하고, 이 채널 목록을 hopset으로 사용하여 주파수 도약 통신을 수행할 수 있다. 이를 통해 1차 사용자와의 충돌을 지능적으로 회피하면서 통신 기회를 극대화할 수 있다.51
- **하이브리드 시스템 (Hybrid Systems):** FHSS를 다른 변조 및 다중 접속 기술과 결합하여 각 기술의 장점을 취하는 하이브리드 시스템에 대한 연구가 활발히 진행 중이다. 예를 들어, FH-OFDM(Orthogonal Frequency Division Multiplexing)은 OFDM의 높은 데이터 전송률과 다중 경로 페이딩에 대한 강인성에 FHSS의 항재밍 및 보안 특성을 결합한 형태이다. 이 시스템에서는 다수의 부반송파로 구성된 OFDM 심볼 블록 전체가 더 넓은 대역 내에서 주기적으로 도약한다. 이는 광대역 재밍과 협대역 재밍 모두에 효과적으로 대응할 수 있어, 보안이 특히 중요한 산업용 IoT(IIoT)나 차세대 군용 통신 등에서 유망한 기술로 주목받고 있다.56

이처럼 FHSS는 미래 통신 환경에서 '주연'이 아닌, 시스템 전체의 강인성과 보안, 신뢰성을 책임지는 핵심 '조연'으로서 그 가치를 지속적으로 발휘할 것으로 전망된다.


본 보고서는 주파수 도약 확산 스펙트럼(FHSS) 기술에 대한 다각적이고 심층적인 고찰을 수행했다. FHSS는 의사난수(PN) 시퀀스에 따라 반송파 주파수를 불연속적으로 도약시켜 통신하는 기법으로, 이를 통해 강력한 항재밍 성능, 낮은 탐지 확률에 기반한 높은 보안성, 그리고 다중 접속 환경에서의 공존 능력을 확보하는 핵심적인 확산 스펙트럼 기술임을 확인했다.

역사적으로 헤디 라마와 조지 앤타일의 군사적 목적의 발명에서 시작된 FHSS는, 동기화라는 근본적인 기술적 난제를 극복하는 과정과 반도체 기술의 발전에 힘입어 실용화되었다. 시스템의 핵심은 PN 시퀀스 생성기와 주파수 합성기의 정밀한 협업에 있으며, 도약 속도에 따라 저속(SFH) 및 고속(FFH) 도약 방식으로 나뉘어 각각 다른 성능적 트레이드오프를 제공한다. 처리 이득과 재밍 마진이라는 정량적 지표는 FHSS가 간섭 환경에서 얼마나 강인한지를 수학적으로 증명하며, 포착과 추적으로 구성된 복잡한 동기화 과정은 이 기술을 구현하는 데 있어 가장 중요한 과제임을 보여주었다.

DSSS와의 비교 분석을 통해, FHSS는 특히 협대역 간섭과 근원 문제에 강인한 특성을 지녀 중앙 제어가 어려운 애드혹 네트워크에 적합하다는 점이 부각되었다. 이러한 특성은 블루투스와 같은 성공적인 상업적 응용으로 이어졌으며, 적응형 주파수 도약(AFH)과 같은 지능형 기법의 도입은 FHSS를 단순 물리 계층 기술에서 인지 무선의 초기 형태로 진화시키는 계기가 되었다.

결론적으로, DSSS 기반 기술이 데이터 통신 시장의 주류를 형성하고 있음에도 불구하고, FHSS는 군용 통신, 블루투스, 그리고 LoRa와 같은 LPWAN 분야에서 보여주듯이, 절대적인 데이터 전송률보다 강인성, 보안성, 저전력, 저비용이 우선시되는 특정 응용 분야에서 여전히 대체 불가능한 핵심 기술로서 확고한 입지를 다지고 있다.

미래 통신 환경에서 FHSS는 단독 기술로 사용되기보다는, 인지 무선 환경에서의 동적 스펙트럼 접속을 위한 핵심 메커니즘으로 활용되거나, OFDM과 같은 다른 고효율 기술과 융합된 하이브리드 시스템의 구성 요소로 기능할 가능성이 높다. 이처럼 FHSS는 끊임없이 진화하며 차세대 통신 시스템의 신뢰성과 보안을 담보하는 핵심 기반 기술로서 그 중요성을 이어나갈 것으로 전망된다.


1. FHSS(Frequency Hopping Spread Spectrum)란??, 8월 3, 2025에 액세스, https://rf-yeolmu.tistory.com/115
2. Spread Spectrum Communication Techniques | www.dau.edu, 8월 3, 2025에 액세스, https://www.dau.edu/cop/e3/resources/spread-spectrum-communication-techniques
3. Spread spectrum techniques | Advanced Signal Processing Class Notes | Fiveable, 8월 3, 2025에 액세스, https://library.fiveable.me/advanced-signal-processing/unit-11/spread-spectrum-techniques/study-guide/CNvKBLosgcwbfs3W
4. What is Spread Spectrum? - GeeksforGeeks, 8월 3, 2025에 액세스, https://www.geeksforgeeks.org/computer-networks/what-is-spread-spectrum/
5. An Introduction to Spread-Spectrum Communications | Analog Devices, 8월 3, 2025에 액세스, https://www.analog.com/en/resources/technical-articles/introduction-to-spreadspectrum-communications--maxim-integrated.html
6. 무선 LAN의 기술적 고찰, 8월 3, 2025에 액세스, http://ebook.pldworld.com/_eBook/-Telecommunications,Networks-/NETWORK_DOCUMENTs/semina/my/wlan/wlan.htm
7. Spread Spectrum Technology & Communications - EE Times, 8월 3, 2025에 액세스, https://www.eetimes.com/tutorial-on-spread-spectrum-technology/
8. ko.wikipedia.org, 8월 3, 2025에 액세스, [https://ko.wikipedia.org/wiki/%EC%A3%BC%ED%8C%8C%EC%88%98_%EB%8F%84%EC%95%BD_%ED%99%95%EC%82%B0_%EC%8A%A4%ED%8E%99%ED%8A%B8%EB%9F%BC#:~:text=%EC%A3%BC%ED%8C%8C%EC%88%98%20%EB%8F%84%EC%95%BD%20%ED%99%95%EC%82%B0%20%EC%8A%A4%ED%8E%99%ED%8A%B8%EB%9F%BC(Frequency,'%EC%A3%BC%ED%8C%8C%EC%88%98%20%EB%8F%84%EC%95%BD'%EC%9D%B4%EB%9D%BC%EA%B3%A0%20%ED%95%9C%EB%8B%A4.](https://ko.wikipedia.org/wiki/주파수_도약_확산_스펙트럼#:~:text=주파수 도약 확산 스펙트럼(Frequency,'주파수 도약'이라고 한다.)
9. DSSS & FHSS - 대학원부터 시작한 첫번째 블로그 - 티스토리, 8월 3, 2025에 액세스, https://bluebamus.tistory.com/entry/DSSS-FHSS
10. 주파수 도약 - IT 위키, 8월 3, 2025에 액세스, [https://itwiki.kr/w/%EC%A3%BC%ED%8C%8C%EC%88%98_%EB%8F%84%EC%95%BD](https://itwiki.kr/w/주파수_도약)
11. 주파수 호핑 확산 스펙트럼 통신(FHSS)의 원리, 8월 3, 2025에 액세스, https://www.hdv-fiber.com/ko/news/the-principle-of-frequency-hopping-spread-spectrum-communication-fhss/
12. Hedy Lamarr and Frequency Hopping - Cade Museum for Creativity & Invention, 8월 3, 2025에 액세스, https://cademuseum.org/podcast/hedy-lamarr-and-frequency-hopping/
13. Hedy Lamarr - National Inventors Hall of Fame®, 8월 3, 2025에 액세스, https://www.invent.org/inductees/hedy-lamarr
14. Random Paths to Frequency Hopping | American Scientist, 8월 3, 2025에 액세스, https://www.americanscientist.org/article/random-paths-to-frequency-hopping
15. Hedy Lamarr - DPMA, 8월 3, 2025에 액세스, https://www.dpma.de/english/our_office/publications/ingeniouswomen/hedylamarr-erfinderischefemmefatale/index.html
16. Hedy Lamarr's patent | National Air and Space Museum, 8월 3, 2025에 액세스, https://airandspace.si.edu/multimedia-gallery/4790640jpg
17. The Seventh Claim | American Scientist, 8월 3, 2025에 액세스, https://www.americanscientist.org/blog/the-long-view/the-seventh-claim
18. Frequency-hopping spread spectrum - Wikipedia, 8월 3, 2025에 액세스, https://en.wikipedia.org/wiki/Frequency-hopping_spread_spectrum
19. Block diagram of the (a) transmitter and (b) receiver of the FHSS system - ResearchGate, 8월 3, 2025에 액세스, https://www.researchgate.net/figure/Block-diagram-of-the-a-transmitter-and-b-receiver-of-the-FHSS-system_fig3_329286286
20. DESIGN AND SIMULATION OF FREQUENCY HOPPING TECHNIQUE IN MATLAB, 8월 3, 2025에 액세스, http://dsp.vscht.cz/konference_matlab/matlab12/full_paper/059_Olsovsky.pdf
21. unit iii frequency hopped spread spectrum systems - Sathyabama, 8월 3, 2025에 액세스, https://www.sathyabama.ac.in/sites/default/files/course-material/2020-10/UNIT3.PDF
22. KR100707287B1 - 무선 통신을 위한 편파 상태 기술들 - Google Patents, 8월 3, 2025에 액세스, https://patents.google.com/patent/KR100707287B1/ko
23. Design and Implementation of FHSS and DSSS for Secure Data Transmission, 8월 3, 2025에 액세스, https://www.ijsps.com/uploadfile/2015/0915/20150915101611816.pdf
24. Spread Spectrum Communications - Definition & Techniques - NI, 8월 3, 2025에 액세스, https://www.ni.com/en/solutions/aerospace-defense/communications-navigation/understanding-spread-spectrum-for-communications.html
25. Linear Feedback Shift Registers (LFSR) - GeeksforGeeks, 8월 3, 2025에 액세스, https://www.geeksforgeeks.org/digital-logic/linear-feedback-shift-registers-lfsr/
26. Feedback Shift Register Sequences - SciSpace, 8월 3, 2025에 액세스, https://scispace.com/pdf/feedback-shift-register-sequences-12pxb97rmq.pdf
27. Tutorial Linear Feedback Shift Registers | PDF | Digital Electronics - Scribd, 8월 3, 2025에 액세스, https://www.scribd.com/document/456939420/Tutorial-Linear-Feedback-Shift-Registers
28. Linear Feedback Shift Registers (LFSRs), 8월 3, 2025에 액세스, https://www.eng.auburn.edu/~nelson/courses/elec4200/ClassMaterial/lfsrs.pdf
29. Frequency-Hop Spread Spectrum 499, 8월 3, 2025에 액세스, https://www.uoanbar.edu.iq/eStoreImages/Bank/2690.pdf
30. Difference between FHSS and DSSS - GeeksforGeeks, 8월 3, 2025에 액세스, https://www.geeksforgeeks.org/computer-networks/fhss-vs-dsss/
31. Technical Comparison of Frequency Hopping and Direct Sequence Spread Spectrum, 8월 3, 2025에 액세스, https://www.qsl.net/n9zia/wireless/fhss_vs_dsss.html
32. Process gain - Wikipedia, 8월 3, 2025에 액세스, https://en.wikipedia.org/wiki/Process_gain
33. Spread Spectrum Communications and Jamming Prof. Debarati Sen G S Sanyal School of Telecommunications Indian Institute of Techno, 8월 3, 2025에 액세스, http://elearn.psgcas.ac.in/nptel/courses/video/117105136/lec4.pdf
34. 5.9: Jamming Spread Spectrum Signals - GlobalSpec, 8월 3, 2025에 액세스, https://www.globalspec.com/reference/61137/203279/5-9-jamming-spread-spectrum-signals
35. Research on Synchronization Technology of Frequency Hopping Communication System - AIP Publishing, 8월 3, 2025에 액세스, https://pubs.aip.org/aip/acp/article-pdf/doi/10.1063/1.5039025/14160664/020053_1_online.pdf
36. UNIT IV SYNCHRONIZATION OF SS SYSTEMS - Sathyabama, 8월 3, 2025에 액세스, https://www.sathyabama.ac.in/sites/default/files/course-material/2020-10/UNIT4_10.pdf
37. A Synchronization Acquisition Method for the Dual-Sequence-Frequency-Hopping Communication System Based on Software Defined Radi, 8월 3, 2025에 액세스, https://www.jsoftware.us/vol16/448-JSW15484.pdf
38. Spread Spectrum Communications and Jamming Prof. Debarati Sen G S Sanyal School of Telecommunications Indian Institute of Techno, 8월 3, 2025에 액세스, http://elearn.psgcas.ac.in/nptel/courses/video/117105136/lec44.pdf
39. Tracking of Spread Spectrum Signals, 8월 3, 2025에 액세스, http://webmail.aast.edu/~mangoud/bb7.pdf
40. Double dither loop for pseudonoise code tracking - NASA Technical Reports Server (NTRS), 8월 3, 2025에 액세스, https://ntrs.nasa.gov/citations/19780039500
41. Comparing FHSS vs. DSSS | System Analysis Blog | Cadence, 8월 3, 2025에 액세스, https://resources.system-analysis.cadence.com/blog/msa2022-comparing-fhss-vs-dsss
42. Near–far problem - Wikipedia, 8월 3, 2025에 액세스, [https://en.wikipedia.org/wiki/Near%E2%80%93far_problem](https://en.wikipedia.org/wiki/Near–far_problem)
43. Near-far problem impact on mobile radiolocation accuracy in CDMA wireless cellular networks - ResearchGate, 8월 3, 2025에 액세스, https://www.researchgate.net/publication/4316595_Near-far_problem_impact_on_mobile_radiolocation_accuracy_in_CDMA_wireless_cellular_networks
44. Bluetooth-WLAN Coexistence - MATLAB & Simulink - MathWorks, 8월 3, 2025에 액세스, https://www.mathworks.com/help/wlan/ug/bluetooth_wlan_coexistence.html
45. Analysis of Near-Far Problem using Power Control Technique for GNSS based Applications - Research Inventy, 8월 3, 2025에 액세스, https://www.researchinventy.com/papers/v4i11/A04110108.pdf
46. Channelization Protocols Explained | Baeldung on Computer Science, 8월 3, 2025에 액세스, https://www.baeldung.com/cs/channelization-protocols
47. Solving the near–far problem in CDMA-based ad hoc networks, 8월 3, 2025에 액세스, https://wireless.ece.arizona.edu/sites/default/files/2023-03/adhocNJ_nearfar_alaa2003.pdf
48. On the Research of Adaptive Frequency Hopping in the Wireless Communication - Atlantis Press, 8월 3, 2025에 액세스, https://www.atlantis-press.com/article/4953.pdf
49. (PDF) Enhanced Adaptive Frequency Hopping for Wireless Personal Area Networks in a Coexistence Environment - ResearchGate, 8월 3, 2025에 액세스, https://www.researchgate.net/publication/221286987_Enhanced_Adaptive_Frequency_Hopping_for_Wireless_Personal_Area_Networks_in_a_Coexistence_Environment
50. Dynamic Adaptive Frequency Hopping for Mutually Interfering Wireless Personal Area Networks - CiteSeerX, 8월 3, 2025에 액세스, https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=f6fdd9ef791590e86342fce1eb2d3adc712e800d
51. Spectrum Sensing for Dynamic Spectrum Access in Cognitive Radio - DiVA portal, 8월 3, 2025에 액세스, https://www.diva-portal.org/smash/get/diva2:1511883/FULLTEXT01.pdf
52. IEEE Standards Supporting Cognitive Radio and Networks, Dynamic Spectrum Access, and Coexistence - ResearchGate, 8월 3, 2025에 액세스, https://www.researchgate.net/publication/3200393_IEEE_Standards_Supporting_Cognitive_Radio_and_Networks_Dynamic_Spectrum_Access_and_Coexistence
53. Wi-Fi: Overview of the 802.11 Physical Layer and Transmitter Measurements - Tektronix, 8월 3, 2025에 액세스, https://download.tek.com/document/37W-29447-2_LR.pdf
54. How Frequency Hopping Spread Spectrum Revolutionizes Wireless Communic - Wray Castle, 8월 3, 2025에 액세스, https://wraycastle.com/blogs/knowledge-base/frequency-hopping-spread-spectrum
55. DYNAMIC SPECTRUM ACCESS: FROM ... - GW Engineering, 8월 3, 2025에 액세스, https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=5c51473b75bf4ca1e310a1310a53a92484247e2d
56. OFDM With Hybrid Number and Index Modulation - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/abs/1911.03747
57. [2407.07721] An hybrid framework OTFS OFDM based on mobile speed estimation - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/abs/2407.07721
58. Theory and Terminology - FHSS vs DSSS - Banner Engineering, 8월 3, 2025에 액세스, https://info.bannerengineering.com/cs/groups/public/documents/literature/tt_fhssvsdsss.pdf
59. Frequency-Hopping Spread Spectrum in Wireless Networks - GeeksforGeeks, 8월 3, 2025에 액세스, https://www.geeksforgeeks.org/ethical-hacking/frequency-hopping-spread-spectrum-in-wireless-networks/