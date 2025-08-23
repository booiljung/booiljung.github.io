# PRIDE-PPPAR



정밀 단독 측위(Precise Point Positioning, PPP)는 전 지구 위성 항법 시스템(Global Navigation Satellite System, GNSS)을 활용한 고정밀 위치 결정 기술의 한 축을 담당해왔다. 이 기술의 가장 큰 장점은 단일 수신기만으로 전 지구적 범위에서 유연하게 운용할 수 있다는 점이다.1 그러나 기존의 PPP 방식은 센티미터 수준의 정확도를 달성하기 위해 약 30분이라는 긴 수렴 시간(convergence time)을 요구하는 치명적인 한계를 내포하고 있다.1 이러한 긴 초기화 시간은 실시간 및 시간 제약적인 응용 분야에서 PPP 기술의 활용을 심각하게 제한하는 요인으로 작용했다.1 이 문제의 근원은 위상 반송파(carrier phase) 측정치에 포함된 미지정수(ambiguity)에 있다. 표준 PPP 모델에서 미지정수는 위성과 수신기의 하드웨어 바이어스(bias)에 의해 오염되어 본래의 정수적 특성을 잃고 실수 값의 ‘부동해(float solution)’로 추정될 수밖에 없었다.4


이러한 기존 PPP의 한계를 극복하기 위한 기술적 돌파구로 미지정수 결정(Ambiguity Resolution, AR) 기술, 즉 PPP-AR이 등장했다. PPP-AR의 핵심 개념은 위상 반송파 미지정수가 본질적으로 정수라는 정보를 활용하는 것이다.5 미지정수에 포함된 하드웨어 바이어스를 정밀하게 추정하고 보정함으로써 미지정수의 정수적 특성을 복원하고, 이를 가장 가능성 있는 정수 값으로 '고정(fixing)'하는 과정을 거친다. 이 기술은 보정 정보 제품인 비보정 위상 지연(Uncalibrated Phase Delays, UPD) 또는 관측별 신호 바이어스(Observable-Specific Biases, OSB) 등을 통해 구현된다.3 미지정수를 정수로 고정함으로써 PPP-AR은 수렴 시간을 15분 이내로 단축시키고, 위치 결정 정확도를 기존의 데시미터 수준에서 센티미터, 나아가 밀리미터 수준까지 획기적으로 향상시킨다.1 이처럼 부동해를 '고정해(fixed solution)'로 전환함으로써 기존 PPP보다 훨씬 더 정확하고 안정적인 결과를 얻을 수 있게 되었다.2


본 보고서의 주제인 PRIDE PPP-AR은 이러한 PPP-AR 기술을 구현하는 핵심적인 소프트웨어 중 하나이다. 이는 우한대학교(Wuhan University) PRIDE 연구실에서 개발한 다중 GNSS PPP-AR을 위한 오픈소스 소프트웨어 패키지이다.7 이 소프트웨어의 중요성은 국제 GNSS 서비스(International GNSS Service, IGS)의 PPP-AR 워킹 그룹에서 정밀 보정 정보 제품의 검증을 위한 권장 소프트웨어로 지정되었다는 점에서 명확히 드러난다.9 이는 PRIDE PPP-AR이 측지학 커뮤니티 내에서 하나의 기준 도구로 인정받고 있음을 의미한다. 소프트웨어 개발의 명시적인 목표는 지각 변동 연구와 같은 지구과학 분야의 고정밀 응용 연구를 지원하는 것이다.7

결론적으로 PPP에서 PPP-AR로의 기술 발전은 고정밀 GNSS 분야의 근본적인 패러다임 전환을 의미한다. 기존 PPP 기술이 가진 느린 수렴 시간이라는 실용적 장벽을 1 미지정수 결정 기술이 해결하였고 3, 이 복잡한 기술을 연구자들이 쉽게 접근하고 활용할 수 있도록 구현한 것이 바로 PRIDE PPP-AR이다. 따라서 이 소프트웨어는 단순히 또 하나의 처리 프로그램이 아니라, 기술적 한계로 인해 제약을 받던 과학 연구를 가능하게 하는 핵심적인 동력원으로서 그 가치를 지닌다.



GNSS 위상 반송파 측정에서 미지정수는 위성과 수신기 사이의 거리에 포함된 파장의 온전한 개수, 즉 본질적으로 정수 값을 가진다. 그러나 표준 PPP 모델에서는 이 정수적 특성이 유지되지 않는데, 그 이유는 추정된 미지정수 파라미터가 위성과 수신기 양쪽에서 발생하는 하드웨어 지연에 의한 비정수적 바이어스에 오염되기 때문이다.4 따라서 PPP-AR 기술의 핵심 과제는 이러한 바이어스 성분을 정확히 분리하고 제거하여 미지정수의 순수한 정수적 특성을 복원하는 것이다.2


분리 시계 모델(Decoupled Clock Model, DCM)은 비차분(undifferenced) 미지정수 결정을 가능하게 하는 핵심적인 이론적 프레임워크로 제시된다.5 단일 시계 파라미터를 사용하는 표준 모델과 달리, DCM은 의사거리(pseudorange, 코드) 측정치와 위상 반송파 측정치에 대해 수신기와 위성 시계 오차 파라미터를 각각 분리하여 추정한다.5 이러한 분리 접근법은 상대적으로 잡음이 큰 코드 바이어스가 위상 기반 시계 추정치에 영향을 미치는 것을 방지하여, 위상 관측 방정식 내에서 미지정수의 정수적 특성을 보존하는 결정적인 역할을 한다.12 이 모델은 초기에 GPS를 위해 개발되었으나, 이후 다중 GNSS 시스템으로 성공적으로 확장되었다.12


DCM과 같은 이론적 모델을 실제 데이터 처리에 적용하기 위해서는 정밀한 보정 정보가 필수적이다. 관측별 신호 바이어스(Observable-Specific Biases, OSB)가 바로 그 역할을 하는 보정 정보 제품이다. OSB는 IGS 분석 센터(Analysis Centers, ACs)에서 전 지구적 기준국 네트워크의 데이터를 기반으로 산출된다.3 이 제품은 사용자의 측정치에서 위성 측 하드웨어 바이어스를 효과적으로 제거하는 데 필요한 보정 값을 제공함으로써, 단일 수신기만으로 미지정수 결정을 가능하게 한다.1 특히 프랑스 국립우주연구센터(CNES) 등에서 제공하는 실시간 OSB 제품의 등장은 실시간 PPP-AR을 현실화하는 데 결정적인 기여를 했다.1 우한대학교의 다중 GNSS 실험 분석 센터(WUM) 또한 전 지구적 PPP-AR을 지원하기 위한 고정밀 OSB 제품을 생성하고 있다.15


바이어스가 보정되고 미지정수의 '부동해' 추정치가 얻어지면, 마지막으로 가장 가능성 있는 정수 값을 찾아내는 단계가 필요하다. 이 과정에서 업계 표준 알고리즘으로 사용되는 것이 바로 LAMBDA(Least-squares AMBiguity Decorrelation Adjustment) 기법이다.5 LAMBDA 기법은 서로 높은 상관관계를 가지는 부동해 미지정수 추정치들을 상관성이 낮은 새로운 집합으로 변환하는 과정을 거친다. 이 변환 과정을 통해 미지정수를 올바른 정수 값으로 고정할 확률이 극적으로 높아진다.5

성공적인 PPP-AR 구현은 이론적 발전, 전 지구적 인프라, 그리고 알고리즘의 정교화가 긴밀하게 결합된 결과물이다. 첫째, DCM과 같은 이론적 혁신은 12 바이어스를 미지정수로부터 분리할 수 있는 수학적 기틀을 마련했다. 둘째, 이러한 이론은 IGS의 전 지구적 기준국 네트워크와 WUM, CNES, CODE, GFZ와 같은 분석 센터들이 OSB 보정 정보를 계산하고 배포하는 인프라 없이는 실용화될 수 없었다.1 셋째, LAMBDA 기법은 정수 고정이라는 마지막 핵심 단계를 수행하는 강력한 알고리즘적 해법을 제공했다.5 PRIDE PPP-AR은 바로 이 세 가지 흐름이 만나는 지점에 위치한다. 즉, DCM을 운용하고, OSB 제품을 사용하며, LAMBDA 기법을 구현하는 소프트웨어 도구인 것이다. 이는 소프트웨어의 성능이 자체적으로 완결되는 것이 아니라, 더 큰 과학적/운용적 생태계와의 통합을 통해 발휘됨을 보여준다. 동시에 이러한 의존성은 외부 제품의 품질과 가용성에 성능이 직접적으로 좌우될 수 있다는 잠재적 취약점이기도 하다.



PRIDE PPP-AR 소프트웨어는 마오롱 거(Maorong Ge) 박사의 초기 연구 노력에서 시작되어, 이후 우한대학교 GNSS 연구센터의 장후이 겅(Jianghui Geng) 교수팀에 의해 개발 및 개선되었다.7 이 소프트웨어의 핵심 철학 중 하나는 GNU 일반 공중 사용 허가서(GPLv3)에 따라 배포되는 오픈소스라는 점이다. 이는 신진 연구자들에게 혜택을 주고 측지학 및 지구물리학적 응용 연구를 장려하려는 명확한 목표를 가지고 있다.8 이 프로젝트는 중국 국립 자연과학재단의 지원을 받으며 국제측지학협회(IAG)의 후원 하에 진행되고 있다.8


소프트웨어의 내부 구조는 크게 두 가지 주요 모듈로 구성된다: 비차분 GNSS 처리 모듈과 단일 기준국 미지정수 결정 모듈이다.17 전체 작업 흐름은 여러 핵심적인 명령줄 프로그램을 통해 이루어진다.19

- `tedit`: 데이터 전처리 및 편집 단계로, 주기 슬립(cycle slip) 탐지 등을 수행한다.
- `spp`: 표준 단독 측위(Standard Point Positioning)를 통해 초기 좌표를 생성한다.
- `lsq`: 핵심적인 최소제곱조정 엔진으로, 미지정수의 부동해를 산출한다.
- `rc`: 외부에서 제공된 바이어스 제품을 이용하여 최종적인 단일 기준국 미지정수 결정을 수행하는 모듈이다.

이 소프트웨어는 Fortran 95로 작성되었으며, Linux 및 macOS 플랫폼에서 성공적으로 테스트되었다.17


PRIDE PPP-AR 소프트웨어의 다양한 기능은 연구자와 전문가가 특정 응용 분야에 맞춰 정밀한 데이터 처리를 수행할 수 있도록 지원한다. 아래 표는 소프트웨어의 핵심적인 기술적 역량을 요약하여 보여준다. 이는 여러 소프트웨어 옵션을 평가하는 사용자에게 중요한 의사결정 도구로 기능하며, GitHub 문서 7 와 여러 과학 논문 20 에 흩어져 있는 정보를 통합적으로 제공한다.

**표 3.1: PRIDE PPP-AR의 주요 기능 (v3.0 이상 기준)**

| 기능 분류       | 세부 내용                                                    | 출처 |
| --------------- | ------------------------------------------------------------ | ---- |
| **GNSS 지원**   | GPS, GLONASS, Galileo, BDS-2/3, QZSS 다중 GNSS 시스템 처리   | 7    |
| **주파수 지원** | 모든 주파수에 대한 PPP-AR 지원 (임의의 이중 주파수 무이온층 조합), L5, E6, B1C 등 신규 신호 포함 | 7    |
| **처리 모드**   | 정지(Static), 이동(Kinematic), 서브-데일리(Sub-daily) 해 산출 | 17   |
| **데이터 속도** | 최대 50 Hz의 고속 데이터 처리 지원                           | 7    |
| **플랫폼 역학** | 항공 사진 측량, 선상 중력 측정 등 고기동성 플랫폼 지원       | 7    |
| **고급 응용**   | 저궤도(LEO) 위성의 이동체 궤도 결정 (예: GRACE), 시각 및 주파수 전송(IPPP) | 7    |
| **장기 처리**   | 최대 108일간의 연속 데이터 처리 및 일 경계 불연속성 완화     | 7    |
| **보정 모델**   | 대류층 모델링(VMF1/VMF3), 2차 전리층 보정, 해양 조석 하중 보정 | 7    |
| **제품 호환성** | RINEX 4, Bias-SINEX 등 최신 IGS 데이터 형식 지원 및 시변 OSB 추정치 처리 | 7    |


소프트웨어의 실제 사용 환경은 Linux용 명령줄 인터페이스(CLI)와 macOS/Windows용 그래픽 사용자 인터페이스(GUI) 도구를 모두 제공하여 접근성을 높였다.21 설치 과정은 

`gfortran`, `make` 등 Linux와 유사한 환경을 필요로 한다.8 소프트웨어 매뉴얼과 교육 과정 자료를 보면 8, 대상 사용자는 GNSS에 대한 배경 지식을 갖추고 명령줄 사용에 익숙하며 RINEX 파일, 정밀 제품(SP3, CLK, Bias-SINEX), 설정 파일 등의 개념을 이해하고 있는 것으로 가정된다. UNAVCO/EarthScope와 같은 기관에서 정기적인 교육 과정을 제공하는 것은 사용자 교육과 지원에 대한 의지를 보여준다.21

PRIDE PPP-AR은 고급 기능과 커뮤니티 구축을 균형 있게 추구하는 전략적 접근 방식을 구현하고 있다. 소프트웨어는 최첨단 고정밀 처리 기능(예: 모든 주파수 AR, LEO 처리)을 제공하면서도 7, 오픈소스 정책 8 과 CLI/GUI 인터페이스 21 를 통해 상용 소프트웨어 대비 재정적, 기술적 진입 장벽을 낮춘다. 더 나아가, 주요 IGS 분석 센터인 우한대학교 개발팀은 소프트웨어뿐만 아니라 이에 필요한 핵심적인 WUM 바이어스 제품을 직접 제공하고 7, 국제 워크숍을 통해 교육과 훈련에 적극적으로 참여한다.21 이는 단순히 소프트웨어를 배포하고 끝내는 것이 아니라, 채택을 장려하고, 도구와 데이터를 함께 제공하여 고품질 결과를 보장하며, 숙련된 사용자 기반을 구축하는 통합적인 생태계를 조성하는 전략이다. 이러한 접근 방식은 광범위한 커뮤니티에 힘을 실어줌으로써 과학적 발견을 가속화하고, IGS 워킹 그룹의 권장 사례에서 볼 수 있듯이 9 소프트웨어를 사실상의 표준으로 자리매김하게 한다.



미지정수 결정(AR)이 가져오는 성능 향상은 정량적 데이터로 명확히 입증된다. 고정해는 부동해에 비해 위치 추정치의 RMS(Root Mean Square) 오차를 극적으로 감소시킨다. 예를 들어, 한 연구에서는 실시간 이동 측위 테스트에서 동(East), 북(North), 상(Up) 방향 성분의 RMS 오차가 부동해에서 각각 13.7 cm, 7.1 cm, 11.4 cm였던 것이 고정해에서는 0.8 cm, 0.9 cm, 2.5 cm로 크게 줄어들었다.2 정지 측위 해 역시 유사한 성능 향상을 보이며, 최종적으로 밀리미터 수준의 정확도에 도달한다.2


여러 GNSS 시스템을 함께 사용하는 것은 상당한 이점을 제공한다. GPS, Galileo, BDS와 같은 위성항법 시스템을 결합하면 가시 위성의 수가 증가하여 기하학적 강도가 향상된다. 이는 특히 도심 협곡과 같이 위성 신호 수신이 어려운 환경에서 수렴을 가속화하고 정확도를 높이는 데 결정적인 역할을 한다.5 실험 결과에 따르면 GPS+Galileo 조합이 종종 최고의 성능을 보이며, 정지 측위 수렴 시간을 GPS 단독 사용 시의 약 11.4분에서 약 8.0분으로 단축시켰다.1 여기에 BDS를 추가하면 수렴 시간을 0.5분에서 1.6분가량 더 줄일 수 있다.6


실제 PPP-AR을 수행하는 사용자에게 가장 중요한 고려사항 중 하나는 어떤 분석 센터(AC)의 정밀 제품을 사용할 것인가이다. 소프트웨어의 성능은 입력되는 궤도, 시계, 바이어스 제품의 품질에 직접적으로 의존하기 때문이다. 여러 연구에서 PRIDE PPP-AR과 유사한 방법론을 사용하여 주요 IGS 분석 센터의 제품 성능을 비교했으며, 그 결과를 종합하면 다음과 같다.

**표 4.1: IGS 분석 센터별 제품을 이용한 PPP-AR 성능 비교**

| 분석 센터                                                    | GNSS 조합 | 처리 모드 | 수렴 시간 (분) | 최초 고정 시간 (TTFF) (분) | 위치 정확도 RMS (E/N/U, cm) |      |
| ------------------------------------------------------------ | --------- | --------- | -------------- | -------------------------- | --------------------------- | ---- |
| **WUM**                                                      | G+E       | 정지      | 1.7 - 2.2      | 3.1 - 3.3                  | 1.2-1.3 / 1.1-1.6 / 2.6-2.8 |      |
|                                                              | G+E+C     | 정지      | 0.8 - 1.0      | 1.6 - 1.7                  | 1.1 / 1.0 / 2.3             |      |
| **CODE**                                                     | G+E       | 정지      | 1.7 - 2.2      | 3.1 - 3.3                  | 1.2-1.3 / 1.1-1.6 / 2.6-2.8 |      |
| **GFZ**                                                      | G+E       | 정지      | 1.7 - 2.2      | 3.1 - 3.3                  | 1.2-1.3 / 1.1-1.6 / 2.6-2.8 |      |
|                                                              | G+E+C     | 정지      | 0.8 - 1.0      | 1.6 - 1.7                  | (WUM과 유사)                |      |
| **CNES**                                                     | G+E+C     | 정지      | 2.0 - 2.3      | 3.5                        | (WUM/GFZ 대비 낮은 성능)    |      |
| 주: G(GPS), E(Galileo), C(BDS)를 의미. 수치는 여러 연구 6 결과를 종합하여 대표적인 범위를 나타냄. |           |           |                |                            |                             |      |

분석 결과를 종합하면 다음과 같은 경향이 나타난다.

- WUM 제품은 특히 다중 GNSS 조합에서 뛰어난 성능을 보이며, 종종 최고의 수렴 정확도와 빠른 TTFF를 달성한다.24
- CODE 제품은 일부 정지 PPP-AR 테스트에서 다른 제품들보다 우수한 성능을 보인다고 보고되었다.6
- GFZ 제품은 WUM 및 CODE와 비슷한 성능을 보이며, 특히 GPS/Galileo/BDS 조합에서 강점을 나타낸다.6
- CNES 제품은 실시간 AR의 기반을 마련했지만 1, 일부 후처리 다중 GNSS 비교에서는 자체적인 기준국 네트워크나 모델의 차이로 인해 수렴 속도와 정확도 측면에서 다소 낮은 성능을 보일 때가 있다.24
- 한 가지 중요한 점은, 부동해의 성능은 제품 선택에 큰 영향을 받지 않지만, 고정해의 성능은 어떤 AC의 제품을 사용하느냐에 따라 크게 달라진다는 것이다.6


실시간 PPP와 후처리 PPP의 성능에는 차이가 존재한다. 스트리밍된 보정 정보를 사용하는 실시간 PPP는 일반적으로 최종 정밀 제품을 사용하는 후처리 해보다 약간 더 긴 수렴 시간과 낮은 정확도를 보인다.25 그럼에도 불구하고, 실시간 다중 GNSS(GREC) 이동 측위는 5.2 cm 미만의 위치 결정 오차를 달성할 수 있는 수준에 도달했다.25

PPP-AR의 성능은 단일한 척도로 평가될 수 없으며, 소프트웨어, GNSS 조합, 그리고 결정적으로 보정 제품의 출처라는 복합적인 함수 관계에 있다. 여러 연구에서 특정 AC가 '최고'라고 보고하는 결과가 상충되어 보이는 것은 6, 평가 기준(수렴 속도 대 최종 정확도), GNSS 조합(예: G+E+C에서 GFZ가 우세, G 단독에서 CODE가 우세), 처리 모드(정지 대 이동)에 따라 최적의 선택지가 달라지기 때문이다. 이러한 가변성은 바로 IGS PPP-AR 워킹 그룹이 설립된 이유, 즉 여러 제품 간의 *상호운용성*을 연구하고 개선하기 위함과 직결된다.9 최종 목표는 서로 다른 AC의 제품을 사용하더라도 일관되고 신뢰할 수 있는 결과를 얻도록 하는 것이다. 사용자에게 이는 자신의 특정 응용 목적에 맞춰 AC 제품을 신중하게 선택해야 함을 시사한다. 예를 들어, 실시간 이동 측위 GPS+Galileo 사용자는 CNES 제품을, 후처리 정지 G+E+C 분석 연구자는 WUM이나 GFZ 제품을 선호할 수 있다. 단일한 승자가 없다는 사실은 오히려 IGS 커뮤니티 내의 건전한 경쟁과 전문화를 반영하며, 최종 사용자에게는 다양한 고품질 옵션을 제공하는 긍정적인 결과로 이어진다.



PRIDE PPP-AR이 설계된 주요 응용 분야 중 하나는 지각 변동 모니터링이다.7 GNSS는 판 구조 운동 및 단층 변형 축적과 관련된 지구 지각의 느리고 미세한 움직임을 측정하는 데 있어 근본적인 도구로 사용된다.27 정지 PPP-AR 해가 제공하는 밀리미터 수준의 정확도는 PRIDE PPP-AR을 이러한 연구에 매우 강력한 도구로 만들어주며, 이는 중국 내륙 지진 분석 연구에 활용된 사례에서도 입증된다.29


PRIDE PPP-AR은 지진 연구, 특히 지진 발생 시의 동적인 변위를 포착하는 데 핵심적인 역할을 한다. 소프트웨어의 고속(1 Hz 이상) 데이터 처리 능력은 지진 발생 중 및 직후의 급격한 지표면 움직임을 포착하는 데 필수적이다.27 PPP-AR이 제공하는 잡음이 적은 이동 측위 해는 표준 PPP에 비해 동지진 변위(coseismic offset)를 훨씬 더 명확하게 분석할 수 있게 해준다.4 실제로 2023년 튀르키예에서 발생한 MW 7.8-7.5 연쇄 지진의 동지진 변위를 고속 GNSS 관측 데이터로 분석한 연구에서 PRIDE PPP-AR이 처리 소프트웨어로 명시적으로 언급되었다.27 비록 다른 연구들이 InSAR이나 광학 위성 기법을 사용하여 동일한 지진을 분석했지만 30, 핵심적인 GNSS 기반 연구에서 이 소프트웨어가 인용되었다는 사실은 최첨단 지진학 연구에서의 그 관련성과 실제 적용 사례를 명확히 보여준다.


PRIDE PPP-AR의 다재다능함은 문서 및 교육 자료에서 언급된 다른 고급 응용 분야들을 통해 더욱 부각된다. 여기에는 항공 사진 측량, 항공/선상 중력 측정, 대기 탐사(대류층 지연 추정), 그리고 공학 측량 등이 포함된다.7

PRIDE PPP-AR의 주요 응용 분야들은 PPP-AR 기술이 가진 특정 장점(고정확도, 빠른 수렴, 고속 처리 능력, 적은 인프라 요구)이 기존 기술의 한계를 직접적으로 극복하는 영역과 일치한다. 예를 들어, 지진학이나 지각 변동 연구는 종종 외딴 지역을 포함한 넓은 영역에 걸쳐 밀리미터에서 센티미터 수준의 정확도를 요구한다.28 기존의 상대 측위(RTK) 기술은 매우 정확하지만 촘촘한 기준국 네트워크를 필요로 하여 넓거나 외진 지역에는 비실용적이다.3 반면, 표준 PPP는 전 지구적 커버리지를 갖지만 동적인 현상을 분석하기에는 너무 느리고 정확도가 부족하다.1 PPP-AR, 그리고 이를 구현하는 PRIDE PPP-AR은 이 두 세계의 장점을 결합한 해법을 제공한다. 즉, 인근 기준국 없이도 RTK 수준의 높은 정확도를 달성할 수 있는 것이다.3 따라서 PRIDE PPP-AR이 이러한 분야에서 성공적으로 활용되는 것은 기술적 공백을 메우는 능력에 기인하며, 이는 이전에는 대규모로 수행하기 어렵거나 불가능했던 과학 연구를 가능하게 한다. 오픈소스라는 특성은 이러한 기술을 전 지구적 학술 커뮤니티에 개방함으로써 그 파급 효과를 더욱 증폭시킨다.



본 보고서의 분석을 종합하면 PRIDE PPP-AR의 강점과 한계는 다음과 같다.

- **강점:** PRIDE PPP-AR은 최첨단 다중 GNSS PPP-AR 기술을 구현하는 오픈소스 기반의 과학 주도형 소프트웨어 패키지이다. 주요 IGS 분석 센터인 우한대학교의 지원을 받으며, IGS 워킹 그룹의 인정을 통해 광범위한 과학 커뮤니티의 신뢰를 얻고 있다. 또한, 측지학 및 지구물리학의 다양한 응용 분야를 지원하는 높은 다재다능함을 갖추고 있다.
- **한계:** 소프트웨어의 성능은 근본적으로 외부 정밀 제품의 품질과 가용성에 의존한다. 또한, GNSS 데이터 처리와 Linux 환경에 익숙하지 않은 사용자에게는 상당한 학습 곡선을 요구한다. 여러 분석 센터 제품 간의 미묘한 성능 차이는 신규 사용자에게 혼란의 원인이 될 수 있다.


PRIDE PPP-AR의 사례는 과학 연구에서 오픈소스 모델이 갖는 중요성을 잘 보여준다. 이 모델은 투명성, 재현성, 협업을 촉진하여 커뮤니티 주도의 검증과 개선을 가능하게 한다. 이는 내부 작동 원리를 알 수 없는 '블랙박스' 형태의 상용 소프트웨어와 대조되며, 현대 과학에서 점차 중요성이 커지고 있는 추세이다.


PRIDE PPP-AR의 변경 기록과 문서는 이미 향후 개발 방향을 시사하고 있다. 새로운 GNSS 신호에 대한 모든 주파수 PPP-AR 지원과 같은 기능 강화가 지속적으로 이루어지고 있다.7 또한, 제품 품질과 상호운용성을 개선하려는 IGS PPP-AR 워킹 그룹의 지속적인 노력은 이 소프트웨어를 통해 달성할 수 있는 성능을 직접적으로 향상시킬 것이다.9 궁극적인 목표는 전 지구 어디에서든 거의 즉각적으로 센티미터 수준의 위치 결정을 달성하는 것이며, PRIDE PPP-AR은 이러한 미래 기술을 개발하고 보급하는 데 있어 핵심적인 플랫폼 역할을 계속할 것이다.


1. Evaluation of Real-time Precise Point Positioning with Ambiguity Resolution Based on Multi-GNSS OSB Products from CNES - MDPI, accessed July 17, 2025, https://www.mdpi.com/2072-4292/14/19/4970
2. Rapid Integer Ambiguity Resolution in GPS Precise Point Positioning - - Nottingham ePrints, accessed July 17, 2025, https://eprints.nottingham.ac.uk/12116/1/Thesis_of_GENG_Jianghui.pdf
3. What is PPP-AR, Fast PPP-AR, and PPP-RTK? - Tersus GNSS, accessed July 17, 2025, https://www.tersus-gnss.com/tech_blog/what-is-ppp-ar-fast-ppp-ar-and-ppp-rtk
4. (PDF) Precise Point Positioning With Ambiguity Resolution In Real ..., accessed July 17, 2025, https://www.researchgate.net/publication/265190333_Precise_Point_Positioning_With_Ambiguity_Resolution_In_Real-Time
5. Multi-GNSS Ambiguity Resolution For Signal Obstruction in PPP - Inside GNSS - Global Navigation Satellite Systems Engineering, Policy, and Design, accessed July 17, 2025, https://insidegnss.com/multi-gnss-ambiguity-resolution-for-signal-obstruction-in-ppp/
6. Assessing the performance of GNSS PPP-AR using OSB products from different analysis centers | Request PDF - ResearchGate, accessed July 17, 2025, https://www.researchgate.net/publication/381665926_Assessing_the_performance_of_GNSS_PPP-AR_using_OSB_products_from_different_analysis_centers
7. PrideLab/PRIDE-PPPAR: An open‑source software for Multi-GNSS PPP ambiguity resolution - GitHub, accessed July 17, 2025, https://github.com/PrideLab/PRIDE-PPPAR
8. PRIDE PPP-AR Ⅲ MANUAL, accessed July 17, 2025, http://pride.whu.edu.cn/ueditor/jsp/upload/file/20231124/1700793919577054879.pdf
9. Precise Point Positioning with Ambiguity Resolution - International GNSS Service, accessed July 17, 2025, https://igs.org/wg/ppp-ar/
10. PRIDE PPP-AR II MANUAL, accessed July 17, 2025, http://pride.whu.edu.cn/ueditor/jsp/upload/file/20230419/1681885924256092664.pdf
11. Evaluation the static precise point positioning and fixing rate with phase ambiguity resolution using different analysis centers products | PLOS One, accessed July 17, 2025, https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0322622
12. Multi-GNSS Precise Point Positioning with Ambiguity Resolution Based on the Decoupled Clock Model - MDPI, accessed July 17, 2025, https://www.mdpi.com/2072-4292/16/16/2999
13. Undifferenced GPS Ambiguity Resolution using the Decoupled Clock Model and Ambiguity Datum Fixing - PPP-Wizard, accessed July 17, 2025, http://www.ppp-wizard.net/Articles/Collins_Navigation_v57n2_2010_accepted.pdf
14. On the interoperability of IGS products for precise point positioning with ambiguity resolution - Bohrium, accessed July 17, 2025, https://www.bohrium.com/paper-details/on-the-interoperability-of-igs-products-for-precise-point-positioning-with-ambiguity-resolution/812696106549903362-45
15. Undifferenced Ambiguity Resolution for Precise Multi-GNSS Products to Support Global PPP-AR - MDPI, accessed July 17, 2025, https://www.mdpi.com/2072-4292/17/8/1451
16. Undifferenced GPS Ambiguity Resolution Using the Decoupled Clock Model and Ambiguity Datum Fixing - The Institute of Navigation, accessed July 17, 2025, https://www.ion.org/publications/abstract.cfm?articleID=102523
17. (PDF) PRIDE PPP-AR: an open-source software for GPS PPP ambiguity resolution, accessed July 17, 2025, https://www.researchgate.net/publication/334264802_PRIDE_PPP-AR_an_open-source_software_for_GPS_PPP_ambiguity_resolution
18. PRIDE PPP-AR: an Open-source Software for GPS PPP Ambiguity Resolution, accessed July 17, 2025, https://beta.ngs.noaa.gov/gps-toolbox/PRIDE.htm
19. GNSS-PPP Course - Day 1 - YouTube, accessed July 17, 2025, https://www.youtube.com/watch?v=wyVGkaOq4SY
20. PRIDE PPP-AR: mitigating multipath and day-boundary discontinuities for geodesy and geophysics | Scilit, accessed July 17, 2025, https://www.scilit.com/publications/0e6a8f755d5d5576ff57c27de47fad48
21. 2024 Technical Short Course: Multi-GNSS Precise Point Positioning ..., accessed July 17, 2025, https://www.earthscope.org/event/2024-technical-short-course-multi-gnss-precise-point-positioning-and-pride-ppp-ar/
22. 2022 Short Course: Multi-GNSS Precise Point Positioning and PRIDE PPP-AR – NSF GAGE, accessed July 17, 2025, https://www.unavco.org/event/2022-short-course-multi-gnss-precise-point-positioning-and-pride-ppp-ar/
23. Evaluation the static precise point positioning and fixing rate with phase ambiguity resolution using different analysis centers products, accessed July 17, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC12088527/
24. Multi-GNSS and Multi-Frequency Precise Point Positioning Based on High Accuracy Products from IGS Analysis Centers - 武汉大学学报( 信息科学版), accessed July 17, 2025, http://ch.whu.edu.cn/en/article/doi/10.13203/j.whugis20240430
25. Performance Assessment of Multi-GNSS Real-Time Products from Various Analysis Centers, accessed July 17, 2025, https://www.researchgate.net/publication/366668315_Performance_Assessment_of_Multi-GNSS_Real-Time_Products_from_Various_Analysis_Centers
26. Performance Assessment of Multi-GNSS Real-Time Products from Various Analysis Centers, accessed July 17, 2025, https://www.mdpi.com/2072-4292/15/1/140
27. Coseismic deformation and kinematic rupture process of the 2023 ..., accessed July 17, 2025, https://academic.oup.com/gji/article/240/3/1363/7928134
28. Crustal Deformation Monitoring | U.S. Geological Survey - USGS.gov, accessed July 17, 2025, https://www.usgs.gov/programs/earthquake-hazards/crustal-deformation-monitoring
29. Is There Accelerated Crustal Deformation Before Large Earthquakes ..., accessed July 17, 2025, https://www.equsci.org.cn/article/id/45563256-43d2-4001-9eca-377a3019a47e?viewType=citedby-info
30. Three-Dimensional Deformation of the 2023 Turkey Mw 7.8 and Mw 7.7 Earthquake Sequence Obtained by Fusing Optical and SAR Images - MDPI, accessed July 17, 2025, https://www.mdpi.com/2072-4292/15/10/2656
31. Impact and GNSS-based deformation analysis of the earthquake in Türkiye on 6 February, 2023: a case study of the TAG (Tarsus–Adana–Gaziantep) motorway and Hatay airport - Frontiers, accessed July 17, 2025, https://www.frontiersin.org/journals/built-environment/articles/10.3389/fbuil.2025.1572885/full
32. (PDF) Three-Dimensional Deformation of the 2023 Turkey Mw 7.8 and Mw 7.7 Earthquake Sequence Obtained by Fusing Optical and SAR Images - ResearchGate, accessed July 17, 2025, https://www.researchgate.net/publication/370919405_Three-Dimensional_Deformation_of_the_2023_Turkey_Mw_78_and_Mw_77_Earthquake_Sequence_Obtained_by_Fusing_Optical_and_SAR_Images
33. Cumulative coseismic deformation of the 2023 Turkey earthquake sequence... - ResearchGate, accessed July 17, 2025, https://www.researchgate.net/figure/Cumulative-coseismic-deformation-of-the-2023-Turkey-earthquake-sequence-observed-by-InSAR_fig3_370883726
34. Coseismic Faulting Model and Post-Seismic Surface Motion of the 2023 Turkey–Syria Earthquake Doublet Revealed by InSAR and GPS Measurements - MDPI, accessed July 17, 2025, https://www.mdpi.com/2072-4292/15/13/3327
35. Study on coseismic surface deformation of the 2023 Turkey MW7.8 and MW7.5 double strong earthquakes using optical image correlation method - 地质力学学报, accessed July 17, 2025, https://journal.geomech.ac.cn/en/article/doi/10.12090/j.issn.1006-6616.2023144
36. Precise Point Positioning with Ambiguity Resolution in Real-Time, accessed July 17, 2025, https://www.ion.org/publications/abstract.cfm?articleID=7969



