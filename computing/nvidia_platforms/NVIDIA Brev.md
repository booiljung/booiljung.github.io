# NVIDIA Brev Blackwell 시대의 AI 개발 패러다임


생성형 인공지능(Generative AI) 시대의 본격적인 도래는 AI 모델의 규모와 복잡성을 기하급수적으로 증대시켰다. 수천억에서 조 단위에 이르는 파라미터를 가진 거대 언어 모델(LLM)이 표준으로 자리 잡으면서, 이를 개발하고 훈련, 배포하는 과정은 전례 없는 도전에 직면했다. 이러한 도전의 중심에는 세 가지 핵심적인 병목 현상이 존재한다. 첫째, 최신 고성능 GPU 자원의 확보 경쟁이 심화되면서 개발자들이 필요한 컴퓨팅 파워에 접근하기 어려워졌다. 둘째, CUDA, Docker, Python 라이브러리 등 복잡하게 얽힌 개발 환경을 구성하고 유지하는 데 과도한 시간과 노력이 소요된다. 셋째, 로컬 개발 환경에서 작성한 코드를 강력한 클라우드 인프라로 이전하고 확장하는 과정에서의 워크플로우 단절은 개발 생산성을 심각하게 저해하는 요인으로 작용하고 있다.

이러한 시대적 과제에 대한 NVIDIA의 전략적 해답이 바로 NVIDIA Brev 플랫폼이다. 2024년 NVIDIA가 인수한 Brev.dev에 기반한 이 플랫폼은 단순히 클라우드 GPU 인스턴스를 제공하는 서비스를 넘어선다. Brev는 NVIDIA의 압도적인 하드웨어 성능을 개발자가 가장 직관적이고 마찰 없이 활용할 수 있도록 설계된 '소프트웨어 관문(Software Gateway)'이다. 복잡한 인프라 구성의 부담을 제거하고, 클릭 몇 번만으로 최적화된 AI 개발 환경을 즉시 제공함으로써, Brev는 AI 개발의 접근성을 민주화하고 있다. 본 보고서는 NVIDIA Brev의 기술적 본질과 핵심 기능을 심층적으로 분석하고, NVIDIA의 거시적인 클라우드 및 AI 생태계 전략 안에서 Brev가 차지하는 전략적 가치를 조명한다. 나아가 Brev를 통해 접근 가능한 차세대 하드웨어의 심장, 즉 Blackwell 아키텍처의 혁신적인 기술들을 상세히 파헤치고, 경쟁 구도 속에서 NVIDIA가 구축하고 있는 기술적 해자를 분석하여 AI 개발의 미래 패러다임 변화를 전망하고자 한다.



NVIDIA Brev의 가장 근본적인 정체성은 클라우드 GPU 자원의 '용량 집계 플랫폼(Capacity Aggregator)'이라는 점에서 출발한다.1 Brev는 자체적인 데이터센터나 물리적 클라우드 인프라를 소유하지 않는다. 대신, Amazon Web Services (AWS), Google Cloud Platform (GCP) 등 전 세계 주요 클라우드 서비스 제공업체(CSP)들이 보유한 방대한 NVIDIA GPU 자원을 하나의 통합된 인터페이스를 통해 제공하는 중개자 역할을 수행한다.2

이러한 애그리게이터 모델은 개발자에게 두 가지 핵심적인 가치를 제공한다. 첫째는 '유연성'이다. 개발자는 특정 CSP에 종속될 필요 없이, 프로젝트의 요구사항, 실시간 가용성, 그리고 비용을 종합적으로 고려하여 최적의 GPU 인스턴스를 동적으로 선택할 수 있다.2 예를 들어, 특정 모델의 훈련에는 AWS의 A100 인스턴스가 비용 효율적일 수 있고, 다른 추론 작업에는 GCP의 L4 인스턴스가 더 적합할 수 있다. Brev는 이러한 선택을 단일 플랫폼 내에서 원활하게 할 수 있도록 지원하여, 자원 확보의 유연성을 극대화한다.

둘째는 '효율성'이다. Brev는 단순히 가상 머신(VM) 인스턴스를 할당하는 것을 넘어, AI 및 머신러닝 개발에 필수적인 소프트웨어 환경을 자동으로 프로비저닝한다.4 사용자가 인스턴스를 생성하면, 최신 버전의 Python, CUDA 드라이버, Docker, 그리고 Jupyter Lab과 같은 핵심 도구들이 사전에 완벽하게 구성된 상태로 제공된다.5 이는 개발자가 인프라 설정이라는 부차적이고 시간 소모적인 작업에서 해방되어, 본질적인 목표인 모델 개발과 실험에 즉시 집중할 수 있도록 돕는다. 이러한 자동화된 환경 구성은 개발 착수 시간을 획기적으로 단축시키고, 전체 개발 사이클의 효율성을 높이는 데 결정적인 기여를 한다.


2024년 7월, NVIDIA는 AI 개발자 플랫폼 스타트업인 Brev.dev를 공식적으로 인수했다고 발표했다.2 이 인수는 단순한 기술 확보를 넘어, 하드웨어 중심 기업에서 소프트웨어 및 서비스 플랫폼 기업으로 진화하려는 NVIDIA의 거시적인 클라우드 전략을 명확하게 보여주는 상징적인 사건이다. NVIDIA는 지난 10여 년간 CSP에 GPU를 공급하는 하드웨어 판매에 주력해왔지만, 최근 몇 년 사이 DGX Cloud와 같은 자체 클라우드 서비스를 출시하며 AI 개발 및 배포와 관련된 클라우드 인프라 시장에서 직접적인 영향력을 확대하려는 움직임을 보여왔다.2 Brev.dev 인수는 이러한 전략의 핵심적인 퍼즐 조각을 맞추는 행보로 해석할 수 있다.

Brev는 NVIDIA의 방대한 AI 생태계 안에서 각 요소들을 유기적으로 연결하고, 최종 개발자에게 통합된 경험을 제공하는 최전선의 접점 역할을 수행한다. 예를 들어, NVIDIA의 추론 최적화 및 배포를 위한 마이크로서비스인 NVIDIA NIMs는 Brev 인스턴스에 손쉽게 배포될 수 있으며, Llama 3와 같은 최신 모델을 즉시 실행할 수 있는 환경을 제공한다.4 또한, 로컬과 클라우드를 아우르는 통합 AI 개발 환경인 NVIDIA AI Workbench와 Brev는 완벽하게 연동된다.3 개발자는 로컬 AI Workbench에서 작업을 시작한 뒤, 더 강력한 컴퓨팅 자원이 필요할 때 Brev를 원격 개발 환경으로 매끄럽게 확장할 수 있다.

결론적으로 Brev는 NVIDIA의 하드웨어(GPU), 소프트웨어(AI Enterprise, NIMs), 그리고 개발 도구(AI Workbench)를 하나로 묶어 개발자에게 가장 접근하기 쉬운 형태로 제공하는 '전략적 추상화 계층(Strategic Abstraction Layer)'이다. NVIDIA의 최신 하드웨어인 Blackwell 아키텍처는 전례 없는 성능을 제공하지만, 동시에 극도로 높은 전력 소비와 액체 냉각 요구사항 등 운영의 복잡성 또한 급증시켰다. 이러한 복잡성은 개별 개발자나 소규모 팀이 최신 하드웨어를 직접 도입하는 데 높은 진입 장벽으로 작용한다. Brev는 바로 이 지점에서 복잡한 클라우드 인프라와 하드웨어 구성을 추상화하여, 개발자에게는 마치 자신의 로컬 컴퓨터를 사용하는 것처럼 간단하고 친숙한 인터페이스만을 제공한다. 이는 NVIDIA가 하드웨어의 압도적인 성능 우위를 소프트웨어 생태계의 지배력으로 전환시키는 핵심적인 연결고리이며, 가장 넓은 개발자층을 자사의 플랫폼으로 유인하는 강력한 전략적 도구이다.



Brev 플랫폼의 가장 혁신적인 기능 중 하나는 'Launchables'이다.5 Launchable은 특정 AI 프로젝트를 실행하는 데 필요한 모든 구성 요소를 하나의 공유 가능한 링크로 패키징한 개념이다. 이 패키지 안에는 필요한 GPU 리소스 사양, 특정 버전의 라이브러리가 설치된 Docker 컨테이너 이미지, 소스 코드가 담긴 GitHub 저장소, 그리고 즉시 실행 가능한 Jupyter Notebook 파일 등이 모두 포함된다.5 개발자는 이 링크를 클릭하는 것만으로 복잡한 설정 과정 없이 사전 구성된 완전한 개발 환경을 몇 분 안에 자신의 Brev 계정에 복제하고 실행할 수 있다.

NVIDIA는 이 Launchable 개념을 더욱 발전시켜 'NVIDIA Blueprints'라는 형태로 제공한다.7 Blueprints는 특정 AI 활용 사례를 위한 레퍼런스 워크플로우로, NVIDIA가 최적화하여 사전 구축한 Launchable이다. 예를 들어, 'PDF to Podcast' Blueprint는 PDF 파일을 입력받아 오디오 콘텐츠를 생성하는 AI 연구 보조 프로그램을 즉시 실행할 수 있게 해주며, 'AI Voice Assistant' Blueprint는 지능형 가상 비서를 구축하는 데 필요한 모든 환경을 제공한다.5 이러한 Blueprints는 최신 AI 프레임워크뿐만 아니라 NVIDIA NIM 마이크로서비스와 같은 고급 도구들을 포함하고 있어, 개발자들이 최신 기술을 손쉽게 테스트하고 자신의 프로젝트에 적용해볼 수 있도록 돕는다.5

Launchable과 Blueprint의 진정한 가치는 AI 프로젝트의 초기 설정 시간을 수일 또는 수 시간에서 단 몇 분 단위로 단축시킨다는 점에 있다. 이는 아이디어의 신속한 프로토타이핑을 가능하게 하고, 팀 내 협업이나 커뮤니티와의 기술 공유를 극적으로 간소화한다. 또한, Launchable을 생성한 개발자는 대시보드를 통해 자신의 Launchable이 얼마나 사용되고 있는지 사용량 메트릭을 모니터링할 수도 있어, 기술 전파의 영향력을 측정하는 데에도 유용하다.5


NVIDIA Brev는 로컬 개발 환경과의 단절 문제를 해결하기 위해 NVIDIA AI Workbench와 긴밀하게 통합되어 있다.7 AI Workbench는 개발자의 로컬 PC나 워크스테이션에서 AI 프로젝트를 관리하고 실행하기 위한 통합 데스크톱 애플리케이션이다.3 Brev는 이 AI Workbench의 '원격 위치(Remote Location)'로 손쉽게 등록될 수 있어, 로컬과 클라우드를 넘나드는 매끄러운(seamless) 개발 워크플로우를 구현한다.

이 통합 워크플로우는 매우 직관적이다. 개발자는 먼저 자신의 로컬 머신에 설치된 AI Workbench에서 프로젝트를 시작한다. 데이터 전처리나 소규모 모델 테스트와 같은 초기 작업은 로컬 RTX GPU를 활용하여 진행할 수 있다. 이후 대규모 모델 훈련이나 분산 학습과 같이 더 강력한 컴퓨팅 파워가 필요한 시점이 오면, 개발자는 AI Workbench 내에서 간단한 CLI 명령어나 GUI 조작을 통해 Brev 인스턴스를 생성하고 현재 프로젝트의 원격 실행 환경으로 연결할 수 있다.3

예를 들어,

```
$ nvwb create context --brev-instance-name <my-brev-instance>
```

와 같은 명령어를 사용하면 현재 실행 중인 Brev 인스턴스를 AI Workbench의 새로운 컨텍스트로 즉시 추가할 수 있다.7

이 연동의 가장 큰 장점은 클라우드 자원 관리를 자동화하여 개발자의 부담을 최소화한다는 점이다. 개발자가 AI Workbench 데스크톱 앱에서 특정 Brev 위치를 열면, 해당 Brev 인스턴스는 백그라운드에서 자동으로 시작된다. 개발이 완료된 후, 해당 위치에 연결된 모든 AI Workbench 창을 닫으면 인스턴스는 자동으로 중지 상태로 전환된다.3 이 기능은 개발자가 인스턴스를 끄는 것을 잊어버려 발생하는 불필요한 유휴 시간 비용을 원천적으로 방지해준다. 다만, 일부 클라우드 제공업체(예: Lambda Labs)의 경우 인스턴스 시작/중지 기능을 지원하지 않아 AI Workbench 통합에서 제외되며, 만일의 경우를 대비해 Brev 웹 콘솔에서 인스턴스가 확실히 종료되었는지 재확인하는 것이 권장된다.3


Brev는 다양한 성향의 개발자들을 모두 만족시키기 위해 강력한 커맨드 라인 인터페이스(CLI)와 직관적인 웹 콘솔을 모두 제공한다.

Brev CLI는 터미널 환경에 익숙하고 자동화를 선호하는 개발자들을 위한 핵심 도구이다.6

```
$ brev start <instance-name>`, `$ brev stop <instance-name>`, `$ brev delete <instance-name>
```

과 같은 간단한 명령어를 통해 인스턴스의 전체 생명주기를 관리할 수 있다. 또한, `$ brev port-forward` 명령어를 사용하면 원격 인스턴스에서 실행 중인 서비스(예: Jupyter Lab, 웹 애플리케이션)의 포트를 로컬 머신으로 손쉽게 전달하여, 마치 로컬에서 실행되는 것처럼 접근할 수 있다.6 이러한 CLI의 존재는 CI/CD 파이프라인과의 통합, 반복적인 작업을 위한 셸 스크립트 작성 등 고급 활용 사례의 문을 열어준다.

반면, 웹 콘솔은 플랫폼의 모든 기능을 시각적으로 관리하고자 하는 사용자에게 최적화된 환경을 제공한다.6 사용자는 웹 브라우저를 통해 새로운 인스턴스나 Launchable을 생성하고, GPU 유형과 디스크 크기를 설정하며, 현재 실행 중인 자원의 상태를 한눈에 파악할 수 있다. 또한, 팀 관리 기능으로 동료를 프로젝트에 초대하거나, 빌링 대시보드를 통해 현재까지의 사용 요금을 투명하게 확인하고 예산을 관리할 수 있다.1 Launchable의 사용 현황을 보여주는 메트릭 모니터링 기능 역시 웹 콘솔을 통해 제공되어, 공유된 프로젝트의 영향력을 시각적으로 분석할 수 있다.5

이처럼 Brev는 단순히 컴퓨팅 자원을 제공하는 것을 넘어, 개발 과정의 모든 마찰을 제거하고 '즉각적인 생산성(Instant Productivity)'이라는 통합된 경험을 제공하는 데 초점을 맞추고 있다. 전통적인 클라우드 GPU 사용 경험은 인스턴스 선택, OS 구성, 드라이버 및 CUDA 설치, 라이브러리 종속성 해결 등 길고 지루한 과정을 수반하며 개발자의 집중력을 흩트린다. Brev의 핵심 기능들은 이러한 과정을 모두 자동화하고 추상화함으로써, 개발자가 아이디어를 떠올린 순간부터 코드를 실행하기까지 걸리는 시간을 극단적으로 단축시킨다. 이는 개발자의 '몰입 상태(flow state)'를 방해하는 인프라 관련 문제를 제거하여 생산성을 극대화하는 효과를 낳으며, 결과적으로 개발자들을 NVIDIA 생태계에 더욱 깊이 유인하는 강력한 무기로 작용한다.


NVIDIA Brev 플랫폼을 통해 제공되는 최첨단 컴퓨팅 성능의 근원은 NVIDIA의 차세대 GPU 아키텍처인 Blackwell에 있다. 수학자 데이비드 해럴드 블랙웰의 이름을 딴 이 아키텍처는 단순한 성능 향상을 넘어, 생성형 AI 시대가 요구하는 극단적인 규모의 연산을 처리하기 위한 근본적인 패러다임 전환을 제시한다.9


Blackwell 아키텍처의 가장 두드러진 물리적 특징은 듀얼 다이(Dual-Die) 설계이다. 지난 수십 년간 반도체 산업을 지배해 온 무어의 법칙이 물리적 한계에 가까워지면서, 단일 실리콘 다이의 크기는 리소그래피 공정의 최대 크기인 '레티클 한계(reticle limit)'에 도달했다.10 이전 세대인 Hopper GH100 다이가 이미 이 한계에 근접했기 때문에, NVIDIA는 성능 확장을 위해 새로운 접근법을 모색해야 했다.

Blackwell은 이 한계를 극복하기 위해, 레티클 한계 크기로 제작된 두 개의 GPU 다이를 하나의 패키지 안에서 유기적으로 연결하는 혁신적인 설계를 채택했다.10 이 두 다이는 초당 10 테라바이트(TB/s)라는 경이적인 대역폭을 가진 'NV-High Bandwidth Interface (NV-HBI)'라는 칩-투-칩 인터커넥트를 통해 연결된다.9 이 연결은 매우 긴밀하여 두 다이 간에 완전한 캐시 일관성(cache coherency)을 유지하며, 소프트웨어나 프로그래머의 관점에서는 완벽하게 하나의 거대한 단일 GPU처럼 인식되고 동작한다.10

이러한 구조적 혁신은 맞춤형 TSMC 4NP 공정을 기반으로 구현되었다. 4NP 공정은 Hopper 아키텍처에 사용된 4N 공정의 향상된 버전으로, 더 높은 집적도를 가능하게 한다.10 이를 통해 Blackwell GPU는 두 개의 다이를 합쳐 총 2,080억 개의 트랜지스터를 집적했는데, 이는 Hopper 아키텍처의 800억 개 대비 약 2.6배에 달하는 수치이다.11 이처럼 Blackwell은 물리적 한계를 창의적인 엔지니어링으로 돌파하며 AI 연산 능력의 새로운 지평을 열었다.


생성형 AI, 특히 LLM의 추론 성능을 극대화하기 위해 Blackwell은 '2세대 트랜스포머 엔진'을 탑재했다.11 이 엔진의 핵심은 새로운 4비트 부동소수점(FP4) 데이터 형식을 하드웨어 수준에서 완벽하게 지원한다는 점이다.11 AI 모델의 가중치나 활성화 값을 표현하는 데 필요한 비트 수를 줄이면(양자화), 동일한 메모리 공간에 더 큰 모델을 저장할 수 있고, 동일한 연산 유닛에서 더 많은 데이터를 처리할 수 있어 처리량(throughput)이 향상된다. FP4는 FP8 대비 데이터 크기가 절반이므로, 이론적으로 2배의 성능 향상과 2배의 모델 수용 능력 증가를 가져올 수 있다.15

하지만 정밀도를 극단적으로 낮추면 모델의 정확도가 심각하게 손상될 위험이 있다. Blackwell은 이 문제를 해결하기 위해 '마이크로텐서 스케일링(Micro-tensor Scaling)'이라는 정교한 기술을 도입했다.17 이는 거대한 텐서 전체에 단일 스케일링 팩터를 적용하는 대신, 텐서 내부를 16개 또는 32개의 값으로 이루어진 작은 블록으로 나누고, 각 블록마다 최적화된 개별 스케일링 팩터를 적용하는 방식이다.19 AI 모델의 가중치와 활성화 값은 동적 범위가 매우 넓어 큰 값과 작은 값이 혼재하는데, 마이크로텐서 스케일링은 이러한 국소적인 데이터 분포 특성에 맞춰 정밀하게 양자화를 수행함으로써 정보 손실을 최소화한다.

특히 NVIDIA는 E2M1(부호 1비트, 지수 2비트, 가수 1비트) 구조를 가지면서, 16개 값으로 구성된 마이크로 블록마다 더 높은 정밀도인 E4M3 FP8 형식의 스케일 팩터를 공유하는 독자적인 'NVFP4' 형식을 구현했다.19 이 고정밀 스케일 팩터는 2의 거듭제곱 형태가 아닌 분수 형태의 스케일링을 가능하게 하여, 데이터의 실제 분포에 더욱 가깝게 근사할 수 있게 해준다. 이러한 하드웨어와 소프트웨어의 결합을 통해 Blackwell은 정확도 저하를 최소화하면서 FP4의 성능 이점을 극대화하며, 이는 NVIDIA가 주장하는 '최대 30배 추론 성능 향상'의 핵심 기술적 기반이 된다.20


조 단위 파라미터 모델의 등장은 개별 GPU의 성능을 넘어, 수백, 수천 개의 GPU를 얼마나 효율적으로 연결하여 하나의 거대한 가속기처럼 사용할 수 있는지가 AI 인프라의 핵심 경쟁력임을 명확히 했다. Blackwell 아키텍처는 이 문제에 대한 해답으로 '5세대 NVLink'와 'NVLink 스위치 시스템'을 제시한다.

5세대 NVLink는 단일 Blackwell GPU 당 18개의 링크를 통해 총 1.8 TB/s의 압도적인 양방향 대역폭을 제공한다.21 이는 4세대 NVLink(900 GB/s) 대비 2배, 그리고 업계 표준 인터페이스인 PCIe Gen5 대비 14배 이상 빠른 속도이다.11 하지만 Blackwell의 진정한 혁신은 이 고속 인터페이스를 'NVLink 스위치' 칩을 통해 랙 스케일, 나아가 데이터센터 스케일로 확장했다는 점에 있다.

대표적인 예가 'GB200 NVL72' 랙 스케일 시스템이다. 이 시스템은 단일 랙 안에 72개의 Blackwell GPU와 36개의 Grace CPU를 5세대 NVLink 스위치 시스템으로 완벽하게 연결하여, 72개의 GPU가 마치 하나의 거대한 GPU처럼 동작하는 'NVLink 도메인'을 형성한다.13 이 도메인 내에서 모든 GPU는 다른 모든 GPU와 최대 속도로 통신할 수 있으며(non-blocking), 총 GPU 통신 대역폭은 130 TB/s에 달한다.23 이는 사실상 '랙 자체가 하나의 거대한 컴퓨터'가 되는 패러다임의 전환을 의미한다.

또한, NVLink 스위치에는 NVIDIA의 네트워킹 기술인 SHARP(Scalable Hierarchical Aggregation and Reduction Protocol) 엔진이 내장되어 있다.11 이는 분산 훈련에서 빈번하게 발생하는 All-Reduce와 같은 집단 통신(collective communication) 연산을 GPU가 아닌 스위치 네트워크 단에서 직접 처리하여 통신 오버헤드를 획기적으로 줄여준다. 이처럼 Blackwell은 개별 칩의 성능 경쟁을 넘어, '데이터센터 스케일 아키텍처' 경쟁으로 패러다임을 전환시켰다. 칩, 시스템, 네트워킹, 소프트웨어를 모두 수직적으로 통합하는 이러한 접근 방식은 경쟁사가 단순히 칩 성능만으로는 따라잡기 어려운 강력한 기술적 해자(moat)를 구축한다.


Blackwell 아키텍처가 달성한 경이로운 성능은 막대한 전력 소비라는 대가를 수반한다. Blackwell B200 GPU의 열 설계 전력(TDP)은 구성 및 냉각 방식에 따라 1000W에서 최대 1200W에 이른다.11 이는 이전 세대인 Hopper H100의 700W TDP를 훨씬 상회하는 수치이다.

이러한 개별 GPU의 높은 전력 소비는 고밀도 랙 시스템에서 더욱 증폭된다. 72개의 GPU가 집적된 GB200 NVL72 시스템의 경우, 전체 랙의 총 전력 소비는 120kW를 초과할 수 있다.26 이는 전통적인 공랭식 데이터센터가 일반적으로 감당할 수 있는 랙당 30~40kW 수준을 3~4배가량 뛰어넘는 엄청난 전력 밀도이다.26 이러한 수준의 발열을 기존의 공기 냉각 방식으로 제어하는 것은 물리적으로 불가능에 가깝다. 공기 흐름을 극단적으로 높이면 엄청난 소음과 비효율적인 HVAC 시스템 운영 비용이 발생하며, 공기 온도를 낮추는 데는 한계가 있다.26

따라서 Blackwell의 최대 성능을 안정적으로 이끌어내기 위해서는 액체 냉각(Liquid Cooling) 솔루션의 도입이 선택이 아닌 필수가 되었다. 특히, 냉각액을 파이프를 통해 GPU 칩에 직접 접촉시켜 열을 제거하는 직접 칩 냉각(Direct-to-Chip Liquid Cooling) 방식은 열 스로틀링(thermal throttling) 없이 GPU가 최대 클럭 속도를 유지할 수 있게 해주는 가장 효과적인 방법이다.13 액체 냉각은 단순히 열을 식히는 것을 넘어, 데이터센터의 공간 효율성(더 높은 랙 밀도 가능)과 에너지 효율성(공조 시스템 전력 감소)을 높여 총소유비용(TCO)을 절감하는 데에도 결정적인 역할을 한다.29 결국, Blackwell의 등장은 고성능 AI 컴퓨팅의 미래가 액체 냉각과 불가분의 관계에 있음을 명확히 보여준다.

아래 표는 Blackwell 아키텍처의 핵심인 GB200 슈퍼칩과 이를 랙 스케일로 확장한 GB200 NVL72 시스템의 주요 제원을 요약한 것이다. 이를 통해 개별 칩의 성능이 NVLink 스위치 시스템을 통해 어떻게 거대한 시스템 수준의 성능으로 확장되는지를 직관적으로 확인할 수 있다.

**표 1: NVIDIA GB200 NVL72 랙 스케일 시스템 제원**

| 항목                     | GB200 Grace Blackwell 슈퍼칩 | GB200 NVL72 랙 시스템          |      |
| ------------------------ | ---------------------------- | ------------------------------ | ---- |
| 구성                     | 1 Grace CPU, 2 Blackwell GPU | 36 Grace CPU, 72 Blackwell GPU |      |
| FP4 성능 (희소성)        | 40 PFLOPS                    | 1,440 PFLOPS (1.44 EFLOPS)     |      |
| FP8/FP6 성능 (희소성)    | 20 PFLOPS                    | 720 PFLOPS                     |      |
| INT8 성능 (희소성)       | 20 POPS                      | 720 POPS                       |      |
| FP16/BF16 성능 (희소성)  | 10 PFLOPS                    | 360 PFLOPS                     |      |
| TF32 성능 (희소성)       | 5 PFLOPS                     | 180 PFLOPS                     |      |
| GPU 메모리 (총합)        | 384 GB HBM3e                 | 13.5 TB HBM3e                  |      |
| GPU 메모리 대역폭 (총합) | 16 TB/s                      | 576 TB/s                       |      |
| NVLink 대역폭            | 3.6 TB/s (칩 내)             | 130 TB/s (도메인 내)           |      |
| CPU 코어                 | 72 Arm Neoverse V2           | 2,592 Arm Neoverse V2          |      |
| CPU 메모리 (총합)        | 480 GB LPDDR5X               | 17 TB LPDDR5X                  |      |

주: 성능 수치는 희소성(sparsity) 기능을 활용했을 때를 기준으로 함. 자료: 20



Blackwell 아키텍처는 이전 세대인 Hopper 대비 모든 AI 워크로드에서 비약적인 성능 향상을 달성했다. 이러한 발전은 객관적인 벤치마크 결과를 통해 명확히 입증된다.

LLM 훈련 성능 측면에서, 업계 표준 벤치마크인 MLPerf Training v4.1 결과는 Blackwell의 압도적인 우위를 보여준다. 8개의 Blackwell GPU로 구성된 HGX B200 시스템은 8개의 Hopper GPU로 구성된 HGX H100 시스템과 비교했을 때, GPT-3 175B 모델 사전 훈련(pre-training)에서 GPU 당 2배, Llama 2 70B 모델의 LoRA(Low-Rank Adaptation) 미세 조정(fine-tuning)에서 2.2배 더 높은 성능을 기록했다.32 이러한 성능 향상은 단순히 연산 유닛의 증가뿐만 아니라, 더 넓은 대역폭을 가진 HBM3e 메모리, 2배 빨라진 5세대 NVLink, 그리고 Tensor Memory Accelerator(TMA)와 같은 하드웨어 기능을 최대한 활용하는 cuDNN 라이브러리의 소프트웨어 최적화가 결합된 결과이다.32

LLM 추론 성능에서는 그 격차가 더욱 극적으로 벌어진다. NVIDIA는 랙 스케일 시스템인 GB200 NVL72가 H100 기반 시스템 대비 조 단위 파라미터 모델의 실시간 추론에서 최대 30배의 성능 향상을 달성했다고 발표했다.20 이 놀라운 수치는 Blackwell의 핵심 혁신 기술인 FP4 정밀도 지원에 기인한다. 즉, H100이 FP8 정밀도로 연산하는 것과 Blackwell이 FP4 정밀도로 연산하는 것을 비교한 결과로, 아키텍처의 근본적인 발전과 데이터 형식의 혁신이 동시에 이루어졌을 때 가능한 성능 도약이다.16

에너지 효율성과 총소유비용(TCO) 측면에서도 Blackwell은 중요한 진전을 이루었다. 액체 냉각 방식이 적용된 GB200 NVL72 시스템은 기존의 H100 공랭식 인프라와 비교했을 때, 동일한 전력을 소비하면서 25배 더 높은 성능을 제공한다.20 이는 데이터센터의 운영 비용(OPEX)을 크게 절감하고, AI 컴퓨팅의 지속 가능성을 높이는 데 결정적인 기여를 할 수 있음을 의미한다.37

아래 표는 Hopper 아키텍처의 H100, H200과 Blackwell 아키텍처의 B200의 주요 기술 제원을 비교하여 세대 간 기술적 진보를 정량적으로 보여준다.

**표 2: NVIDIA GPU 아키텍처별 주요 제원 비교**

| 항목                    | NVIDIA B200 (Blackwell) | NVIDIA H200 (Hopper) | NVIDIA H100 (Hopper) |      |
| ----------------------- | ----------------------- | -------------------- | -------------------- | ---- |
| 아키텍처                | Blackwell (Dual-Die)    | Hopper (Monolithic)  | Hopper (Monolithic)  |      |
| 트랜지스터 수           | 2,080억 개              | 800억 개             | 800억 개             |      |
| 메모리 종류             | HBM3e                   | HBM3e                | HBM3                 |      |
| 메모리 용량             | 192 GB                  | 141 GB               | 80 GB                |      |
| 메모리 대역폭           | 8.0 TB/s                | 4.8 TB/s             | 3.35 TB/s            |      |
| FP4 성능 (희소성)       | 20 PFLOPS               | N/A                  | N/A                  |      |
| FP8 성능 (희소성)       | 10 PFLOPS               | 4 PFLOPS             | 4 PFLOPS             |      |
| FP16/BF16 성능 (희소성) | 5 PFLOPS                | 2 PFLOPS             | 2 PFLOPS             |      |
| NVLink                  | 5세대 (1.8 TB/s)        | 4세대 (900 GB/s)     | 4세대 (900 GB/s)     |      |
| TDP                     | 1000W - 1200W           | 700W                 | 700W                 |      |

자료: 12


NVIDIA Blackwell이 AI 가속기 시장에서 독보적인 위치를 점하고 있지만, AMD와 Intel 역시 각자의 전략으로 경쟁 구도를 형성하고 있다.

NVIDIA Blackwell vs. AMD Instinct MI300X/MI350:

AMD의 Instinct MI300X는 192GB의 방대한 HBM3 메모리 용량을 강점으로 내세워, 단일 카드에 거대 모델을 수용할 수 있는 능력을 강조한다.42 이는 모델 병렬화의 복잡성을 줄여줄 수 있는 잠재력을 가진다. MLPerf Inference v4.1 벤치마크에서 MI300X는 NVIDIA H100과 Llama 2 70B 모델 추론에서 대등한 성능을 보여주며 경쟁력을 입증했다.44 그러나 이는 Blackwell의 이전 세대와의 비교이며, FP4를 지원하는 Blackwell B200과는 상당한 성능 격차가 존재한다. AMD의 차세대 MI350은 288GB의 HBM3e 메모리와 더 높은 대역폭을 예고하며 Blackwell과의 격차를 줄이려 하고 있지만 46, AMD의 가장 큰 약점은 여전히 소프트웨어 생태계에 있다. NVIDIA의 CUDA는 지난 수십 년간 축적된 라이브러리, 최적화된 도구, 방대한 개발자 커뮤니티를 통해 AI 개발의 표준으로 자리 잡았다. 반면, AMD의 ROCm은 빠르게 발전하고 있음에도 불구하고, 실제 대규모 훈련 환경에서의 안정성, 버그, 프레임워크 호환성 문제 등에서 여전히 어려움을 겪고 있다는 평가가 지배적이다.47

NVIDIA Blackwell vs. Intel Gaudi 3:

Intel은 Gaudi 3를 통해 Blackwell과의 직접적인 최고 성능 경쟁보다는, H100 대비 우수한 '가격 대비 성능'을 제공하는 비용 효율적인 대안으로 시장을 공략하고 있다.48 Intel은 8개의 Gaudi 3 칩으로 구성된 플랫폼을 NVIDIA의 8-GPU HGX H100 플랫폼 가격의 3분의 2 수준인 125,000달러에 공급한다고 밝히며 가격 경쟁력을 전면에 내세웠다.49 성능 면에서 Gaudi 3는 특정 LLM 추론 워크로드, 특히 긴 응답을 생성하는 시나리오에서 H100을 능가하는 결과를 보여주기도 했다.48 하지만 전반적인 연산 처리량(TFLOPS)은 Blackwell에 크게 미치지 못하며, Blackwell의 핵심 무기인 FP4 정밀도를 지원하지 않는다는 명백한 한계를 가진다.51 또한, Gaudi 3는 칩 간 및 보드 간 인터커넥트로 표준 이더넷을 사용하여 개방성을 강조하지만, 이는 NVIDIA의 독점 고성능 NVLink에 비해 대역폭과 지연 시간 측면에서 성능적 한계를 가질 수밖에 없다.52

결론적으로, 경쟁사들이 개별 칩의 특정 사양(메모리 용량, 가격)에서 NVIDIA와 경쟁을 시도하고 있지만, AI 워크로드의 실제 성능은 하드웨어와 소프트웨어 스택 전체의 성숙도와 최적화 수준에 의해 결정된다. NVIDIA는 CUDA 생태계를 통해 이 부분에서 압도적인 우위를 점하고 있으며, Blackwell의 NVLink 스위치 시스템은 경쟁의 장을 개별 서버 노드에서 수백, 수천 개의 GPU를 연결하는 '클러스터 스케일'로 확장시켰다. 이는 네트워킹 하드웨어(InfiniBand, Spectrum-X)와 소프트웨어(NCCL, Magnum IO)까지 수직 통합한 NVIDIA만이 제공할 수 있는 차별점이다. 경쟁사들이 '점'(개별 칩)의 성능을 따라잡으려 노력하는 동안, NVIDIA는 '선'(서버 내 NVLink)과 '면'(데이터센터 스케일 패브릭)을 장악하며 경쟁의 차원을 바꾸고 있다.

아래 표는 주요 AI 가속기들의 핵심 사양을 비교하여 시장 내 경쟁 구도를 명확히 보여준다.

**표 3: 주요 AI 가속기 경쟁 구도 비교**

| 항목              | NVIDIA B200         | AMD Instinct MI300X | Intel Gaudi 3 |      |
| ----------------- | ------------------- | ------------------- | ------------- | ---- |
| 아키텍처          | Blackwell           | CDNA 3              | Gaudi         |      |
| 메모리 용량       | 192 GB HBM3e        | 192 GB HBM3         | 128 GB HBM2e  |      |
| 메모리 대역폭     | 8.0 TB/s            | 5.3 TB/s            | 3.7 TB/s      |      |
| FP8 성능 (밀집)   | ~5 PFLOPS           | ~1.3 PFLOPS         | 1.835 PetaOPs |      |
| BF16 성능 (밀집)  | ~2.5 PFLOPS         | ~1.3 PFLOPS         | 1.835 PFLOPS  |      |
| 인터커넥트        | NVLink 5 (1.8 TB/s) | Infinity Fabric     | 24 x 200GbE   |      |
| TDP               | 1000W - 1200W       | 750W                | 900W (OAM)    |      |
| 소프트웨어 생태계 | CUDA                | ROCm                | SynapseAI     |      |

자료: 42



Blackwell 아키텍처는 조 단위 파라미터를 가진 초거대 언어 모델(LLM)의 훈련과 추론 패러다임을 근본적으로 바꾸고 있다. 훈련 측면에서, Blackwell의 향상된 Tensor Core 연산 능력과 5세대 NVLink를 통한 압도적인 스케일 아웃 성능은 LLM의 훈련 시간을 획기적으로 단축시킨다.32 MLPerf 벤치마크에서 GPT-3 175B 모델 훈련에 256개의 Hopper GPU가 필요했던 작업을 단 64개의 Blackwell GPU로 수행할 수 있게 되었다는 사실은 33, 모델 개발 주기를 수개월에서 수주 단위로 단축시킬 수 있는 잠재력을 보여준다.

특히 Blackwell은 전문가 혼합(Mixture-of-Experts, MoE) 모델 아키텍처에 최적화되어 있다. MoE 모델은 추론 시 입력 토큰에 따라 관련된 일부 '전문가(expert)' 서브네트워크만 활성화하여 전체 모델 파라미터 수 대비 연산량을 크게 줄이는 효율적인 구조이다.56 하지만 전문가들 간의 결과를 취합하고 다음 레이어로 전달하는 과정에서 발생하는 통신이 심각한 병목 현상을 유발할 수 있다.18 Blackwell의 2세대 트랜스포머 엔진과 고대역폭 NVLink 스위치 시스템은 이러한 통신 오버헤드를 최소화하여, 10조 개 이상의 파라미터를 가진 거대한 MoE 모델의 훈련 및 실시간 추론을 경제적으로 실현 가능하게 만든다.17

추론 영역에서 Blackwell의 혁신은 더욱 두드러진다. FP4 정밀도의 도입은 추론 처리량을 극대화하여, DeepSeek-R1(671B)과 같은 거대 모델을 기반으로 한 AI 서비스의 운영 비용을 크게 절감할 수 있게 한다.58 이는 단순히 응답 속도를 높이는 것을 넘어, 더 복잡하고 긴 '사고(thinking)' 과정이 필요한 AI 에이전트나 실시간 추론(reasoning) 애플리케이션의 실현을 앞당기는 중요한 기술적 기반이 된다.59


Blackwell은 AI뿐만 아니라 전통적인 고성능 컴퓨팅(HPC) 및 과학 연구 분야에서도 혁신을 주도하고 있다. Blackwell GPU는 과학 및 공학 시뮬레이션에서 여전히 중요한 FP64(배정밀도) 및 FP32(단정밀도) 연산 성능을 Hopper 대비 30% 향상시켰다.57 이를 통해 회로 시뮬레이션, 전산 유체 역학(CFD), 기후 모델링과 같은 전통적인 HPC 워크로드의 실행 시간을 단축하고 더 높은 정밀도의 시뮬레이션을 가능하게 한다.60

더 나아가 Blackwell은 AI와 시뮬레이션을 융합하는 새로운 과학 연구 패러다임을 가속화한다. NVIDIA Parabricks 소프트웨어와 결합된 Blackwell은 유전체학 분석 시간을 획기적으로 단축시키며 8, 신약 개발, 양자 컴퓨팅 시뮬레이션, 재료 과학 등 다양한 분야에서 AI를 활용한 발견의 속도를 높인다.57 특히 Brev 플랫폼의 Blueprints 기능을 통해, 생물정보학이나 분자 동역학과 같이 복잡한 설정이 필요한 과학 연구 워크플로우를 클라우드에서 손쉽게 배포하고 실행할 수 있게 되었다.8

또한, Blackwell에 새롭게 내장된 하드웨어 감압 엔진(Decompression Engine)은 데이터 분석 워크로드에 큰 변화를 가져온다. 이 엔진은 압축된 데이터를 GPU에서 직접 고속으로 풀어낼 수 있어, CPU 대비 최대 18배, H100 GPU 대비 6배 빠른 데이터베이스 쿼리 성능을 제공한다.11 이는 대규모 실험 데이터나 시뮬레이션 결과를 분석하는 시간을 크게 단축시켜 과학자들이 더 빠르게 통찰력을 얻을 수 있도록 돕는다.


본 보고서에서 심층적으로 분석한 바와 같이, NVIDIA Brev와 Blackwell 아키텍처는 각각 독립된 기술이 아니라, AI 개발의 전 과정을 아우르는 NVIDIA의 거대한 '완전 통합 스택(Full-Stack)' 전략의 핵심 구성 요소이다. Blackwell은 개별 칩의 성능을 넘어 데이터센터 전체를 하나의 거대한 컴퓨팅 플랫폼으로 재정의하는 하드웨어의 비전을 제시한다. 그러나 이러한 플랫폼은 그 자체로 매우 강력하지만 동시에 운영과 접근이 복잡하다는 양면성을 가진다.

바로 이 지점에서 Brev가 그 최종적인 역할을 수행한다. Brev는 이처럼 강력하지만 복잡한 인프라를 최종 개발자가 단 몇 줄의 코드나 몇 번의 클릭만으로 접근하고 활용할 수 있게 만드는 '최종 사용자 인터페이스(End-User Interface)'이자 생태계로 들어오는 '관문'이다. 이는 마치 골드러시 시대에 단순히 성능 좋은 곡괭이(GPU)를 파는 것을 넘어, 아무나 접근하기 힘든 거대한 금광(Blackwell 데이터센터)을 소유하고, 그곳까지 누구나 쉽고 빠르게 도달할 수 있도록 잘 닦인 고속도로(Brev 플랫폼)를 건설하는 것과 같다. Launchables는 목적지별 내비게이션이며, AI Workbench 연동은 하이패스와 같다.

NVIDIA는 실리콘(Blackwell)부터 시스템(DGX/NVL72), 네트워킹(NVLink/InfiniBand), 핵심 소프트웨어(CUDA/NIM), 그리고 클라우드 접근 플랫폼(Brev)에 이르기까지 AI 개발에 필요한 모든 요소를 수직적으로 완벽하게 통합했다. 이러한 완전 통합 스택 전략은 경쟁사들이 단순히 칩 성능만으로는 따라오기 힘든 강력하고 지속 가능한 기술적 해자를 구축한다. 이는 AI 시대의 '운영체제'로서 NVIDIA의 지위를 공고히 할 것이며, Brev는 이 거대한 생태계의 관문으로서 더 많은 개발자를 유인하고 NVIDIA의 기술 리더십을 미래에도 지속 가능하게 만드는 핵심 동력이 될 것으로 전망된다.


1. Second-day experience Brev's on-demand GPU-enabled AI/ML containers/VMs for build and train - YouTube, 8월 20, 2025에 액세스, https://www.youtube.com/watch?v=eTyF47muSRo
2. Nvidia Expands AI Cloud Capabilities With Fourth Acquisition This Year - CRN, 8월 20, 2025에 액세스, https://www.crn.com/news/ai/2024/nvidia-makes-fourth-software-acquisition-this-year-with-brev-dev
3. NVIDIA Brev - AI Workbench Locations, 8월 20, 2025에 액세스, https://docs.nvidia.com/ai-workbench/user-guide/latest/locations/brev.html
4. Getting Started — NVIDIA Brev, 8월 20, 2025에 액세스, https://docs.nvidia.com/brev/latest/getting-started.html
5. Brev | NVIDIA Developer, 8월 20, 2025에 액세스, https://developer.nvidia.com/brev
6. NVIDIA Brev, 8월 20, 2025에 액세스, https://docs.nvidia.com/brev/latest/index.html
7. Streamline Collaboration Across Local and Cloud Systems with NVIDIA AI Workbench, 8월 20, 2025에 액세스, https://developer.nvidia.com/blog/streamline-collaboration-across-local-and-cloud-systems-with-nvidia-ai-workbench/
8. Shrink Genomics and Single-Cell Analysis Time to Minutes with NVIDIA Parabricks and NVIDIA AI Blueprints, 8월 20, 2025에 액세스, https://developer.nvidia.com/blog/shrink-genomics-and-single-cell-analysis-time-to-minutes-with-nvidia-parabricks-and-nvidia-blueprints/
9. NVIDIA Blackwell GPUs: Architecture, Features, Specs - NexGen Cloud, 8월 20, 2025에 액세스, https://www.nexgencloud.com/blog/performance-benchmarks/nvidia-blackwell-gpus-architecture-features-specs
10. Blackwell (microarchitecture) - Wikipedia, 8월 20, 2025에 액세스, https://en.wikipedia.org/wiki/Blackwell_(microarchitecture)
11. The Engine Behind AI Factories | NVIDIA Blackwell Architecture, 8월 20, 2025에 액세스, https://www.nvidia.com/en-us/data-center/technologies/blackwell-architecture/
12. NVIDIA Blackwell vs NVIDIA Hopper: A Detailed Comparison - NexGen Cloud, 8월 20, 2025에 액세스, https://www.nexgencloud.com/blog/performance-benchmarks/nvidia-blackwell-vs-nvidia-hopper-a-detailed-comparison
13. Comparing NVIDIA Blackwell Configurations - AMAX Engineering, 8월 20, 2025에 액세스, https://www.amax.com/comparing-nvidia-blackwell-configurations/
14. NVIDIA Blackwell GPU architecture: Unleashing next‑gen AI performance | genai-research, 8월 20, 2025에 액세스, https://wandb.ai/onlineinference/genai-research/reports/NVIDIA-Blackwell-GPU-architecture-Unleashing-next-gen-AI-performance--VmlldzoxMjgwODI4Mw
15. Is NVIDIA comparing FP4 vs FP8 in the consumer site' Specs table without a disclaimer/clearly stating it? - TechPowerUp, 8월 20, 2025에 액세스, https://www.techpowerup.com/forums/threads/is-nvidia-comparing-fp4-vs-fp8-in-the-consumer-site-specs-table-without-a-disclaimer-clearly-stating-it.333784/
16. Deep dive into NVIDIA Blackwell Benchmarks — where does the 4x training and 30x inference performance gain, and 25x reduction in energy usage come from? - adrian cockcroft, 8월 20, 2025에 액세스, https://adrianco.medium.com/deep-dive-into-nvidia-blackwell-benchmarks-where-does-the-4x-training-and-30x-inference-0209f1971e71
17. NVIDIA Blackwell Architecture | Exxact Blog, 8월 20, 2025에 액세스, https://www.exxactcorp.com/blog/hpc/nvidia-blackwell-architecture
18. Nvidia's Blackwell Offers FP4, Second-Gen Transformer Engine - EE Times, 8월 20, 2025에 액세스, https://www.eetimes.com/nvidias-blackwell-gpu-offers-fp4-transformer-engine-sharp/
19. Introducing NVFP4 for Efficient and Accurate Low-Precision ..., 8월 20, 2025에 액세스, https://developer.nvidia.com/blog/introducing-nvfp4-for-efficient-and-accurate-low-precision-inference/
20. GB200 NVL72 | NVIDIA, 8월 20, 2025에 액세스, https://www.nvidia.com/en-us/data-center/gb200-nvl72/
21. NVLink & NVSwitch: Fastest HPC Data Center Platform | NVIDIA, 8월 20, 2025에 액세스, https://www.nvidia.com/en-us/data-center/nvlink/
22. Fifth-Generation NVIDIA NVLink - AMAX Engineering, 8월 20, 2025에 액세스, https://www.amax.com/fifth-generation-nvidia-nvlink/
23. NVIDIA DGX GB200 - AMAX Engineering, 8월 20, 2025에 액세스, https://www.amax.com/nvidia-dgx-gb200-nvl72/
24. NVIDIA's full-spec Blackwell B200 AI GPU uses 1200W of power, up from 700W on Hopper H100 - TweakTown, 8월 20, 2025에 액세스, https://www.tweaktown.com/news/97059/nvidias-full-spec-blackwell-b200-ai-gpu-uses-1200w-of-power-up-from-700w-on-hopper-h100/index.html
25. Nvidia turns up the AI heat with 1200W Blackwell GPUs - The Register, 8월 20, 2025에 액세스, https://www.theregister.com/2024/03/18/nvidia_turns_up_the_ai/
26. Beyond Air: The TCO and Density Imperative of Liquid Cooling for the NVIDIA Blackwell Era, 8월 20, 2025에 액세스, https://www.gocodeo.com/post/beyond-air-the-tco-and-density-imperative-of-liquid-cooling-for-the-nvidia-blackwell-era
27. NVIDIA GB200 NVL72 | Continuum Labs, 8월 20, 2025에 액세스, https://training.continuumlabs.ai/infrastructure/servers-and-chips/nvidia-gb200-nvl72
28. Nvidia's Blackwell Showcases the Future of AI Is Water-Cooled – For Now - HPCwire, 8월 20, 2025에 액세스, https://www.hpcwire.com/2024/12/11/nvidias-blackwell-showcases-the-future-of-ai-is-water-cooled-for-now/
29. Chill Factor: NVIDIA Blackwell Platform Boosts Water Efficiency by Over 300x, 8월 20, 2025에 액세스, https://blogs.nvidia.com/blog/blackwell-platform-water-efficiency-liquid-cooling-data-centers-ai-factories/
30. NVIDIA GB200 NVL72 - AI server, 8월 20, 2025에 액세스, https://aiserver.eu/product/nvidia-gb200-nvl72/
31. Nvidia Blackwell variants comparison table : r/hardware - Reddit, 8월 20, 2025에 액세스, https://www.reddit.com/r/hardware/comments/1biesok/nvidia_blackwell_variants_comparison_table/
32. NVIDIA Blackwell Doubles LLM Training Performance in MLPerf Training v4.1, 8월 20, 2025에 액세스, https://developer.nvidia.com/blog/nvidia-blackwell-doubles-llm-training-performance-in-mlperf-training-v4-1/
33. NVIDIA B200 "Blackwell" Records 2.2x Performance Improvement Over its "Hopper" Predecessor | TechPowerUp Forums, 8월 20, 2025에 액세스, https://www.techpowerup.com/forums/threads/nvidia-b200-blackwell-records-2-2x-performance-improvement-over-its-hopper-predecessor.328792/
34. NVIDIA Blackwell | Products - CoreWeave, 8월 20, 2025에 액세스, https://www.coreweave.com/products/nvidia-blackwell
35. Hopper or Blackwell? Choosing the Right GPU Architecture - Hyperbolic, 8월 20, 2025에 액세스, https://www.hyperbolic.ai/blog/comparing-nvidia-s-hopper-and-blackwell-architectures-for-ai-ml-practitioners
36. NVIDIA Blackwell Architecture Technical Overview, 8월 20, 2025에 액세스, https://resources.nvidia.com/en-us-blackwell-architecture
37. How Much Could OpenAI Save by Switching from NVIDIA Hopper to Blackwell o3? - Reddit, 8월 20, 2025에 액세스, https://www.reddit.com/r/singularity/comments/1hjc47p/how_much_could_openai_save_by_switching_from/
38. B200 vs H200: Best GPU for LLMs, vision models, and scalable training | Blog - Northflank, 8월 20, 2025에 액세스, https://northflank.com/blog/b200-vs-h200
39. NVIDIA H100 vs H200 vs B200: Complete GPU Comparison Guide 2025 - Introl, 8월 20, 2025에 액세스, https://introl.com/blog/h100-vs-h200-vs-b200-choosing-the-right-nvidia-gpus-for-your-ai-workload
40. Blackwell B200 vs. Hopper H200 vs. H100: What's the Difference - Server Simply, 8월 20, 2025에 액세스, https://www.serversimply.com/blog/blackwell-b200-and-hopper-h200
41. Comparing NVIDIA's B200 and H100: What's the difference? - Civo.com, 8월 20, 2025에 액세스, https://www.civo.com/blog/comparing-nvidia-b200-and-h100
42. MI300X vs Blackwell : Who Will Wear the 2025‑26 “LLM GPU Crown”? - INFINITIX | AI-Stack, 8월 20, 2025에 액세스, https://ai-stack.ai/en/blackwell-vs-mi300x
43. Comparison of the Latest AI Chips: NVIDIA H100, AMD MI300, Intel Gaudi3, and Apple M3, 8월 20, 2025에 액세스, https://dolphinstudios.co/comparing-the-ai-chips-nvidia-h100-amd-mi300/
44. The First AI Benchmarks Pitting AMD Against Nvidia - The Next Platform, 8월 20, 2025에 액세스, https://www.nextplatform.com/2024/09/03/the-first-ai-benchmarks-pitting-amd-against-nvidia/
45. AMD MI300X Accelerators are Competitive with NVIDIA H100, Crunch MLPerf Inference v4.1 | TechPowerUp Forums, 8월 20, 2025에 액세스, https://www.techpowerup.com/forums/threads/amd-mi300x-accelerators-are-competitive-with-nvidia-h100-crunch-mlperf-inference-v4-1.326052/
46. AMD MI350 vs. NVIDIA Blackwell: A Comparative Analysis of Next-Generation Chips - Zoomax Low Vision Aids, 8월 20, 2025에 액세스, https://zoomax.com/low-vision-information/amd-mi350-vs-nvidia-blackwell-a-comparative-analysis-of-next-generation-chips/
47. MI300X vs H100 vs H200 Benchmark Part 1: Training – CUDA Moat Still Alive - SemiAnalysis, 8월 20, 2025에 액세스, https://semianalysis.com/2024/12/22/mi300x-vs-h100-vs-h200-benchmark-part-1-training/
48. Outrun By Nvidia, Intel Pitches Gaudi 3 Chips For Cost-Effective AI Systems - CRN, 8월 20, 2025에 액세스, https://www.crn.com/news/ai/2024/outrun-by-nvidia-intel-pitches-gaudi-3-chips-for-cost-effective-ai-systems
49. Analysis: As Nvidia Takes AI Victory Lap, AMD Doubles The Trouble For Intel - CRN, 8월 20, 2025에 액세스, https://www.crn.com/news/components-peripherals/2024/analysis-as-nvidia-takes-ai-victory-lap-amd-doubles-the-trouble-for-intel
50. Intel Gaudi 3 vs. Nvidia H100: Enterprise AI Inference Price-Performance Comparative Analysis - FiberMall, 8월 20, 2025에 액세스, https://www.fibermall.com/blog/intel-gaudi3-vs-nvidia-h100.htm
51. Stacking Up Intel Gaudi Against Nvidia GPUs For AI - The Next Platform, 8월 20, 2025에 액세스, https://www.nextplatform.com/2024/06/13/stacking-up-intel-gaudi-against-nvidia-gpus-for-ai/
52. Will Intel Gaudi 3 catapult the company to data-center AI relevance? - XPU.pub, 8월 20, 2025에 액세스, https://xpu.pub/2024/05/02/intel-gaudi-3/
53. Nvidia Blackwell vs. MI300X : r/AMD_Stock - Reddit, 8월 20, 2025에 액세스, https://www.reddit.com/r/AMD_Stock/comments/1bk2ngz/nvidia_blackwell_vs_mi300x/
54. Intel Gaudi3 vs NVIDIA H100: A Comprehensive Comparison | by Paul Goll | Medium, 8월 20, 2025에 액세스, https://medium.com/@paulgoll/intel-gaudi3-vs-nvidia-h100-a-comprehensive-comparison-61cbcf378c13
55. NVIDIA Blackwell Delivers up to 2.6x Higher Performance in MLPerf Training v5.0, 8월 20, 2025에 액세스, https://developer.nvidia.com/blog/nvidia-blackwell-delivers-up-to-2-6x-higher-performance-in-mlperf-training-v5-0/
56. Applying Mixture of Experts in LLM Architectures | NVIDIA Technical Blog, 8월 20, 2025에 액세스, https://developer.nvidia.com/blog/applying-mixture-of-experts-in-llm-architectures/
57. NVIDIA Blackwell Platform Pushes the Boundaries of Scientific Computing, 8월 20, 2025에 액세스, https://blogs.nvidia.com/blog/blackwell-scientific-computing/
58. NVIDIA Blackwell Delivers World-Record DeepSeek-R1 Inference Performance, 8월 20, 2025에 액세스, https://developer.nvidia.com/blog/nvidia-blackwell-delivers-world-record-deepseek-r1-inference-performance/
59. NVIDIA Blackwell Ultra for the Era of AI Reasoning, 8월 20, 2025에 액세스, https://developer.nvidia.com/blog/nvidia-blackwell-ultra-for-the-era-of-ai-reasoning/
60. NVIDIA RTX PRO Servers With Blackwell Coming to World's Most Popular Enterprise Systems, 8월 20, 2025에 액세스, https://nvidianews.nvidia.com/news/nvidia-rtx-pro-servers-with-blackwell-coming-to-worlds-most-popular-enterprise-systems
61. Applied Digital Among First Cloud Service Providers to Use New NVIDIA Blackwell Platform, 8월 20, 2025에 액세스, https://ir.applieddigital.com/news-events/press-releases/detail/90/applied-digital-among-first-cloud-service-providers-to-use

