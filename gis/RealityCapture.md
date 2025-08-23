# RealityCapture(RealityScan)


현실 세계를 정밀한 디지털 데이터로 변환하는 리얼리티 캡처(Reality Capture) 기술은 현대 산업의 패러다임을 바꾸고 있다.1 건축, 엔지니어링, 건설(AEC), 게임 개발, 영화 시각 효과(VFX), 문화유산 보존 등 다양한 분야에서 리얼리티 캡처는 기존의 수동적이고 시간 소모적인 데이터 수집 방식을 대체하며, 프로젝트의 효율성을 극대화하고 비용을 절감하며, 이해관계자 간의 원활한 협업을 가능하게 하는 핵심 동력으로 자리 잡았다.3 이 기술은 레이저 스캐닝(LiDAR), 사진측량(Photogrammetry), 구조광 스캐닝 등 다양한 방법을 통해 물리적 환경을 고도로 정확한 3D 모델, 포인트 클라우드, 또는 디지털 트윈(Digital Twin)으로 재현한다.1

이러한 기술적 흐름의 중심에 슬로바키아의 Capturing Reality사가 개발한 사진측량 소프트웨어 RealityCapture가 있다.6 RealityCapture는 순서 없이 촬영된 다수의 사진이나 레이저 스캔 데이터를 입력받아, 경계선 없는(seamless) 초고품질 3D 모델을 놀라운 속도로 생성하는 능력으로 시장의 주목을 받아왔다.7

소프트웨어의 운명은 2021년 3월, 게임 엔진의 거인 에픽게임즈(Epic Games)에 인수되면서 극적인 전환점을 맞이했다.9 이 인수는 단순한 기업 인수를 넘어, 에픽게임즈가 구상하는 거대한 3D 콘텐츠 제작 생태계 구축 전략의 핵심적인 한 수였다. 에픽게임즈는 자사의 주력 제품인 언리얼 엔진(Unreal Engine)을 중심으로 Quixel(3D 에셋 라이브러리), Twinmotion(실시간 시각화 툴), ArtStation(아티스트 커뮤니티) 등 관련 기술 기업들을 공격적으로 인수하며 3D 콘텐츠 제작 파이프라인의 수직 계열화를 완성해왔다.9 RealityCapture는 이미 인수 전부터 에픽의 핵심 자산인 Quixel Megascans 라이브러리 제작에 필수적인 도구로 사용되고 있었으며, 이는 양사 간의 기술적 시너지가 충분히 검증되었음을 의미한다.10

인수 이후 에픽게임즈는 RealityCapture의 가격 정책, 접근성, 그리고 생태계 내 역할을 근본적으로 재편했다. 가장 주목할 만한 변화는 2024년 4월에 단행된 리브랜딩과 새로운 가격 정책이다. 데스크톱용 전문가 소프트웨어인 'RealityCapture'는 'RealityScan'으로, 모바일 스캔 앱은 'RealityScan Mobile'로 명칭이 변경되었다.13 본 보고서는 혼동을 피하기 위해 최신 명칭인 'RealityScan'을 기준으로 서술할 것이다. 에픽게임즈의 비전은 명확하다. RealityScan을 단순한 개별 소프트웨어로 판매하여 수익을 내는 것이 아니라, 현실 세계를 디지털로 변환하는 '입력 게이트'로서 기능하게 하여 더 많은 창작자와 개발자를 언리얼 엔진 생태계로 유입시키고, 궁극적으로는 이들이 만들어낼 메타버스와 디지털 트윈의 미래를 선점하려는 것이다.10

본 보고서는 RealityScan의 기술적 핵심을 분석하고, 에픽게임즈 인수 이후 급변한 라이선스 및 도입 비용 구조를 총체적으로 해부하며, 실제 도입을 고려하는 전문가와 기업이 합리적인 의사결정을 내릴 수 있도록 실용적인 가이드를 제공하는 것을 목표로 한다.


RealityScan이 경쟁이 치열한 포토그래메트리 시장에서 독보적인 위치를 차지할 수 있었던 배경에는 타협 없는 품질과 압도적인 처리 속도를 양립시킨 핵심 기술이 있다. 이 장에서는 소프트웨어의 근간을 이루는 엔진 기술, 주요 기능과 워크플로우, 그리고 에픽게임즈 생태계와의 유기적인 통합이 어떻게 시너지를 창출하는지 심층적으로 분석한다.


RealityScan의 성능은 정교하게 설계된 포토그래메트리 파이프라인에 기반한다. 그 핵심에는 SfM(Structure-from-Motion)과 MVS(Multi-View Stereo)라는 두 가지 기본 원리가 자리 잡고 있다.16

- **SfM (Structure-from-Motion)**: 이 기술은 여러 각도에서 촬영된 2D 이미지들에서 공통된 특징점(feature points)을 식별하고 매칭하여, 각 이미지의 촬영 위치와 방향(카메라 포즈), 그리고 3차원 공간상의 특징점들의 위치(희소 포인트 클라우드, sparse point cloud)를 동시에 추정한다. RealityScan은 이 과정을 매우 빠르고 정확하게 처리하여 후속 단계의 견고한 기반을 마련한다.
- **MVS (Multi-View Stereo)**: SfM 단계에서 얻어진 정확한 카메라 정보를 바탕으로, 이미지 간의 픽셀 단위 비교를 통해 훨씬 더 조밀하고 상세한 3차원 정보를 복원한다. 이 과정을 통해 수백만 개 이상의 점으로 구성된 밀도 높은 포인트 클라우드(dense point cloud) 또는 삼각 메시(triangulated mesh)가 생성된다.16

RealityScan이 경쟁 제품 대비 월등한 처리 속도를 자랑하는 비결은 이러한 기본 원리를 하드웨어 가속 기술과 효율적으로 결합한 데 있다. 특히 NVIDIA GPU의 병렬 컴퓨팅 플랫폼인 CUDA(Compute Unified Device Architecture)를 적극적으로 활용하여, 연산 집약적인 MVS 및 텍스처링 과정을 극적으로 가속한다.6 이로 인해 RealityScan의 핵심 기능을 온전히 사용하기 위해서는 NVIDIA GPU가 필수적인 하드웨어 요구사항이 된다.18

더 나아가, RealityScan은 '아웃 오브 코어(out-of-core)' 알고리즘 설계를 채택했다는 점에서 기술적 우위를 점한다.8 이는 처리해야 할 데이터 전체를 시스템의 주 메모리(RAM)에 한 번에 로드할 필요 없이, 데이터를 분할하여 디스크와 메모리 간에 효율적으로 스왑하며 처리하는 방식이다. 덕분에 사용자는 시스템 RAM 용량의 물리적 한계를 넘어 수만 장의 고해상도 이미지나 대용량 레이저 스캔 데이터와 같은 방대한 데이터셋도 일반적인 사양의 워크스테이션에서 처리할 수 있다.18 예를 들어, 16GB RAM 환경에서도 수천 장의 이미지를 처리하는 것이 가능하다.8 이러한 설계는 대규모 프로젝트를 수행하는 전문가들에게 하드웨어 투자 비용의 부담을 줄여주는 실질적인 이점으로 작용한다.


RealityScan은 초보자부터 전문가까지 아우를 수 있는 유연하고 강력한 기능 세트를 제공한다. 사용자는 간단한 자동화 워크플로우를 따르거나, 각 단계를 수동으로 제어하며 결과물의 품질을 극대화할 수 있다.

데이터 입력의 유연성

소프트웨어의 가장 큰 장점 중 하나는 다양한 데이터 소스를 통합 처리할 수 있는 능력이다. 일반 DSLR이나 스마트폰으로 촬영한 사진, 드론을 이용한 항공 이미지, 심지어 비디오 영상까지 입력받아 3D 모델을 생성할 수 있다.7 여기에 더해, 정밀도가 높은 지상(terrestrial) 및 항공(aerial) LiDAR 레이저 스캔 데이터를 사진과 결합하여, 사진만으로는 얻기 힘든 정확한 스케일과 기하학적 구조를 가진 모델을 생성하는 하이브리드 워크플로우를 지원한다.1

자동화와 수동 제어의 조화

초보 사용자는 입력 데이터를 불러온 뒤 'Start' 버튼을 클릭하는 것만으로 이미지 정렬, 메시 생성, 텍스처링에 이르는 전 과정을 자동으로 수행할 수 있다.22 반면, 전문가는 워크플로우 탭을 따라 각 단계를 개별적으로 실행하며 세부 파라미터를 정밀하게 제어할 수 있다. 예를 들어, 이미지 정렬(Alignment) 품질을 조정하거나, 메시 모델(Mesh Model) 생성 시 디테일 수준을 선택하고, 텍스처링(Texturing) 시 해상도와 방식을 지정하는 것이 가능하다.21

핵심 기능 목록

RealityScan은 3D 재구성의 전 과정을 지원하는 포괄적인 도구 모음을 제공한다.21

- **모델 생성**: 사진 및/또는 레이저 스캔으로부터 희소/고밀도 포인트 클라우드와 3D 메시를 자동으로 생성한다.
- **모델 편집**: 생성된 메시의 폴리곤 수를 줄이는 '단순화(Simplify)', 불필요한 부분을 제거하는 '필터링(Filter)', 표면을 부드럽게 만드는 '스무딩(Smoothing)', 모델의 구멍을 메우는 '구멍 닫기(Close Holes)' 등 다양한 편집 도구를 제공한다.
- **텍스처링 및 색상화**: 고해상도 이미지를 기반으로 사실적인 텍스처를 생성하고, 정렬 후 이미지 간의 색상 차이를 보정하는 '색상 정규화(Color Normalization)' 기능을 지원한다. 또한, 고품질 모델의 텍스처를 단순화된 모델에 다시 투영하는 '텍스처 재투영(Texture Reprojection)'도 가능하다.
- **측정 및 지리 참조**: 모델의 스케일을 정확하게 설정하고, 거리, 면적, 부피를 측정할 수 있다. 지상 기준점(GCPs)이나 비행 로그를 사용하여 모델을 실제 지리 좌표계에 정밀하게 일치시키는 지리 참조(Georeferencing) 기능은 측량 및 매핑 분야에서 필수적이다.
- **데이터 입출력**: 최종 모델을 다양한 LOD(Level of Detail)로 내보내거나, 왜곡 보정된 이미지, 정사사진(Ortho-photo), 디지털 표면 모델(DSM) 등 다양한 형태의 결과물을 생성할 수 있다. 또한, 생성된 모델을 Sketchfab과 같은 온라인 플랫폼에 직접 공유하는 기능도 내장되어 있다.

RealityScan 2.0의 혁신

최신 2.0 업데이트는 AI 기술을 접목하여 워크플로우를 한층 더 효율적으로 만들었다.14

- **AI 마스킹**: AI 기반의 객체 분할 모델을 통해 이미지의 배경을 자동으로 제거해준다. 이는 수동 마스킹 작업에 소요되던 막대한 시간을 절약시키는 혁신적인 기능이다.
- **정렬 알고리즘 개선**: 특징점이 적은 평평한 표면 등에서 이미지 정렬 성공률을 높여, 여러 조각으로 나뉘는 컴포넌트 발생을 줄이고 모델의 일관성을 향상시켰다.
- **시각적 품질 분석**: 스캔 데이터의 커버리지가 충분한지, 부족한 부분이 어디인지 히트맵 형태로 시각화하여 보여준다. 이를 통해 사용자는 메시 생성 단계로 넘어가기 전에 데이터의 허점을 미리 파악하고 보완할 수 있어 재작업을 크게 줄일 수 있다.


RealityScan의 진정한 가치는 에픽게임즈의 거대한 생태계 안에서 다른 도구들과 결합될 때 발현된다. 이는 개별 소프트웨어의 기능을 넘어 '통합된 워크플로우'라는 강력한 경쟁력을 창출한다.

Unreal Engine과의 완벽한 연동

가장 핵심적인 시너지는 언리얼 엔진(UE)과의 연동에서 나온다. 전통적인 워크플로우에서는 고폴리곤 스캔 데이터를 게임 엔진에서 실시간으로 사용하기 위해 리토폴로지(retopology), UV 언래핑, 노멀 맵 베이킹 등 복잡하고 시간 소모적인 최적화 과정이 필수적이었다. 하지만 UE5의 혁신적인 가상화 지오메트리 시스템인 '나나이트(Nanite)'는 이 과정을 사실상 불필요하게 만들었다.16 사용자는 RealityScan에서 생성된 수백만, 수천만 폴리곤의 고밀도 메시를 별도의 최적화 없이 FBX 포맷으로 익스포트하여 UE5로 직접 임포트할 수 있다.24 Nanite는 이 고밀도 메시를 실시간으로 스트리밍하고 렌더링하여, 퍼포먼스 저하 없이 영화 수준의 디테일을 구현한다. 이 혁신적인 워크플로우는 CD PROJEKT RED가 'The Witcher 4' 테크 데모에서 사실적인 숲 환경을 구현하는 데 실제로 활용되며 그 강력함을 입증했다.27

MetaHuman 생성 파이프라인

RealityScan은 사실적인 디지털 휴먼 제작의 문턱을 획기적으로 낮췄다. 사용자는 RealityScan(또는 RealityScan Mobile)을 이용해 실제 인물의 얼굴을 스캔하고, 이 데이터를 UE의 'Mesh to MetaHuman' 플러그인으로 처리하여 즉시 애니메이션이 가능한 고품질 메타휴먼(MetaHuman) 아바타를 생성할 수 있다.28 이 워크플로우는 복잡한 3D 모델링 기술 없이도 자신이나 특정 인물을 닮은 디지털 캐릭터를 만들 수 있게 하여, 게임, 영화, 가상현실 등 다양한 분야에서 활용될 수 있는 무한한 가능성을 열어준다.31

Fab 마켓플레이스 및 Sketchfab 연동

생성된 3D 에셋은 에픽의 생태계 안에서 유통되고 활용된다. RealityScan은 3D 모델 공유 플랫폼인 Sketchfab으로 결과물을 직접 업로드하는 기능을 내장하고 있다.33 Sketchfab은 에픽게임즈가 인수한 또 다른 자회사로, 현재 Quixel Megascans, ArtStation 마켓플레이스와 함께 에픽의 통합 에셋 마켓플레이스인 '팹(Fab)'으로 통합되고 있다.13 이는 사용자가 RealityScan으로 현실 세계의 에셋을 캡처하고(생성), UE에서 콘텐츠를 제작하며(제작), 필요에 따라 Fab에서 다른 에셋을 구매하거나 자신의 에셋을 판매하는(유통) 완결된 순환 구조를 형성한다.

이러한 생태계 통합은 에픽게임즈의 치밀한 전략을 보여준다. RealityScan은 현실을 가상으로 옮기는 첫 관문 역할을 하며, 사용자를 자연스럽게 UE, MetaHuman, Fab으로 이어지는 파이프라인으로 유도한다. 경쟁사들이 개별 소프트웨어의 기능적 우수성을 내세울 때, 에픽게임즈는 '워크플로우 전체의 효율성과 통합성'이라는 더 큰 차원의 가치를 제공함으로써 시장의 규칙을 바꾸고 있다. 이는 개발자와 아티스트의 작업 효율을 극대화하는 동시에, 에픽 플랫폼에 대한 의존도를 심화시키는 강력한 락인(lock-in) 효과를 창출한다.


RealityScan 도입을 고려할 때, 소프트웨어 라이선스 비용뿐만 아니라 최적의 성능을 발휘하기 위한 하드웨어 및 데이터 수집 장비에 대한 투자를 포함하는 총 소유 비용(Total Cost of Ownership, TCO)을 종합적으로 분석하는 것이 필수적이다. 특히 에픽게임즈 인수 이후 라이선스 정책이 급격하게 변화했기 때문에, 과거와 현재의 비용 구조를 명확히 이해해야 한다.


RealityScan(구 RealityCapture)의 가격 정책은 '고가의 전문가용 툴'에서 '생태계 확장을 위한 미끼 상품'으로 그 성격이 변화해왔다. 이는 에픽게임즈의 비즈니스 모델 전환을 명확히 보여주는 지표다.

- **초기 모델 (에픽 인수 이전)**: 당시 RealityCapture는 소수의 전문가 시장을 겨냥한 전형적인 고가 소프트웨어였다. 영구 라이선스 가격은 €15,000(약 $16,500)에 달했으며, 이는 개인이나 소규모 스튜디오에게는 상당한 진입 장벽이었다.12 이와 함께 제공된 PPI(Pay-Per-Input) 모델은 사용한 만큼만 비용을 지불하는 방식으로, 입력되는 데이터의 양(사진 매수, 해상도, 레이저 스캔 포인트 수)에 따라 크레딧을 구매하고 결과물을 내보낼 때 차감하는 구조였다.36 이 모델은 저빈도 사용자에게는 합리적이었으나, 대규모 프로젝트를 수행하는 헤비 유저에게는 오히려 구독제보다 비싼 비용을 초래할 수 있었다.37
- **과도기 (2021년 인수 직후)**: 에픽게임즈는 인수 직후, 사용자 기반을 폭발적으로 늘리기 위한 공격적인 가격 인하를 단행했다. 영구 라이선스 가격을 $3,750로 대폭 인하하고, PPI 크레딧의 가치를 높여 사실상의 가격 인하 효과를 가져왔다.12 이는 Quixel 인수 때와 동일한 패턴으로, 에픽 생태계로의 사용자 유입을 최우선 과제로 삼는 전략의 시작이었다.
- **현재 모델 (2024년 4월 이후)**: 2024년 4월, RealityScan 1.4 버전 출시와 함께 에픽은 PPI와 영구 라이선스의 신규 판매를 전면 중단하고, 연간 총수익(annual gross revenue) $1M USD를 기준으로 무료 사용자와 유료 구독자를 구분하는 새로운 구독 모델을 도입했다.39 이 변화는 소프트웨어 자체의 판매 수익을 상당 부분 포기하는 대신, 잠재적인 언리얼 엔진 사용자를 최대한 확보하려는 의도를 명확히 보여준다. 사실상 대부분의 학생, 개인 개발자, 소규모 스튜디오에 소프트웨어를 무료로 제공함으로써 생태계의 저변을 넓히고, 안정적인 수익은 대기업으로부터 확보하는 이원화 전략이다.


현재 RealityScan의 라이선스는 사용자의 연간 총수익과 사용 목적에 따라 명확하게 구분된다.

- **무료 플랜**: 다음 조건에 해당하는 사용자는 RealityScan의 모든 기능을 기간 제한 없이 무료로 사용할 수 있다.43
  - 개인 또는 기업의 연간 총수익이 $1,000,000 USD 미만인 경우
  - 학생, 교육자 및 교육 기관 (수익 제한 없음)
  - 취미 사용자
- **유료 플랜 (연간 총수익 $1M USD 초과 기업 대상)**:
  - **RealityScan Seat**: 연간 $1,250 (사용자당). RealityScan 소프트웨어 단독 사용을 위한 구독 라이선스다.40
  - **Unreal Subscription**: 연간 $1,850 (사용자당). RealityScan, Unreal Engine, Twinmotion 세 가지 소프트웨어를 함께 사용할 수 있는 번들 구독이다.39 언리얼 엔진을 활용한 비게임 분야(건축 시각화, 영화, 시뮬레이션 등) 프로젝트를 진행하는 기업을 위한 종합 솔루션이다.

**기존 라이선스 보유자 정책**: 신규 판매는 중단되었지만, 기존 사용자의 권리는 보호된다.

- **PPI 크레딧**: 2024년 4월 23일 이후 12개월간 보유한 크레딧을 계속 사용할 수 있다.39
- **영구 라이선스**: 보유한 버전은 영구적으로 사용 가능하며, 1.4 버전 출시 후 12개월간 무료로 최신 버전 업데이트를 제공받는다. 그 이후 업데이트를 원할 경우, 새로운 구독 정책에 따라 라이선스를 구매해야 한다.42

아래 표는 현재의 라이선스 플랜을 한눈에 비교하여 사용자의 의사결정을 돕는다.

**표 1: RealityScan 라이선스 플랜 비교 (2024년 4월 이후)**

| 플랜명                  | 대상                                                         | 연간 비용 (사용자당) | 포함된 소프트웨어                          | 주요 특징                                                    |
| ----------------------- | ------------------------------------------------------------ | -------------------- | ------------------------------------------ | ------------------------------------------------------------ |
| **Free**                | - 연간 총수익 $1M USD 미만 개인/기업 - 학생, 교육자, 취미 사용자 | $0                   | RealityScan                                | - 모든 기능 사용 가능 - 상업적 이용 가능 (수익 조건 내)      |
| **RealityScan Seat**    | - 연간 총수익 $1M USD 초과 기업                              | $1,250               | RealityScan                                | - RealityScan 단독 사용 - 1년간 업데이트 포함                |
| **Unreal Subscription** | - 연간 총수익 $1M USD 초과 기업                              | $1,850               | - RealityScan - Unreal Engine - Twinmotion | - 에픽의 3대 크리에이티브 툴 포함 - 비게임 분야 개발사에 최적화된 번들 |

주: 모든 가격은 세금 별도이며, 지역에 따라 변동될 수 있다.43


RealityScan의 소프트웨어 라이선스 비용은 많은 사용자에게 '0'이 되었지만, 그 성능을 제대로 활용하기 위한 하드웨어 및 장비 투자는 여전히 중요한 비용 요소다. 1년차 총 소유 비용(TCO)은 다음과 같이 산출할 수 있다.
$$
TCO_{1-year} = C_{hardware} + C_{software} + C_{acquisition}
$$

- `$C_{hardware}$`: 워크스테이션 구축 비용
- `$C_{software}$`: 연간 구독 비용 (해당 시)
- `$C_{acquisition}$`: 카메라, 드론 등 데이터 수집 장비 비용

워크스테이션 요구사항

RealityScan은 CUDA 기반으로 작동하므로, NVIDIA 그래픽카드가 사실상 필수다. GPU가 없으면 이미지 정렬과 같은 일부 기능만 작동하고, 핵심인 3D 모델 생성과 텍스처링은 불가능하다.18

- **최소 사양**: 64-bit Windows, 8GB RAM, CUDA 3.5 지원 NVIDIA GPU (1GB VRAM).18
- **권장 사양**: 4코어 이상 CPU, 16GB 이상 RAM, **CUDA 6.1 이상 지원 NVIDIA GPU**, NVMe SSD.18
- **RAM**: 처리할 이미지 수에 직접적인 영향을 받는다. 기본 설정(이미지당 40k 특징점) 기준으로 약 4,000장의 이미지를 처리하려면 32GB, 8,000장은 64GB, 16,000장은 128GB의 RAM이 권장된다.19
- **스토리지**: OS와 소프트웨어 설치용, 프로젝트 데이터용, 캐시 파일용으로 빠른 속도의 NVMe SSD를 여러 개 사용하는 것이 워크플로우 속도 향상에 매우 효과적이다.19

데이터 수집 장비

결과물의 품질은 원본 데이터의 품질에 크게 좌우된다.

- **카메라 및 렌즈**: 고해상도 풀프레임 미러리스 카메라(예: Canon EOS R8, 약 $1,300)와 밝은 단렌즈(예: Sigma 35mm F1.4 DG DN, 약 $900) 조합은 고품질 객체 스캔에 이상적이다.50
- **드론**: 넓은 지역이나 건축물 스캔을 위해서는 항공 촬영용 드론(예: DJI Mavic 3 Pro, 약 $2,199)이 필수적이다.54
- **조명**: 제어된 환경에서 반사와 그림자를 최소화하기 위한 조명 장비(예: Godox SL60W 2-light kit, 약 $200~$370)도 중요한 투자 항목이다.56

아래 표는 사용 목적에 따른 워크스테이션 구성 예시와 예상 비용을 정리한 것이다. 이는 하드웨어 투자 규모를 가늠하는 데 실질적인 기준을 제공한다.

표 2: RealityScan 워크스테이션 구성 예시 및 예상 비용 (2025년 상반기 기준)

(환율: 1 USD = 1,350원 가정)

| 구분                                | **입문 전문가용 (Entry-Pro)**                       | **고급형 (High-End)**                                        | **최고사양 (Top-Tier)**                                      |
| ----------------------------------- | --------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **주요 용도**                       | 소규모 객체 스캔, 학습, UE5 연동 테스트             | 전문적인 3D 에셋 제작, 대규모 데이터셋 처리                  | VFX, 대규모 디지털 트윈, 연구개발                            |
| **CPU**                             | Intel Core i5-14600K (~$300 / 41만원)               | Intel Core i9-14900K (~$550 / 74만원)                        | AMD Threadripper PRO 7965WX (~$2,650 / 358만원)              |
| **GPU**                             | NVIDIA GeForce RTX 4070 SUPER 12GB (~$600 / 81만원) | NVIDIA GeForce RTX 4080 SUPER 16GB (~$1,000 / 135만원)       | 2 x NVIDIA GeForce RTX 4090 24GB (~$3,600 / 486만원)         |
| **RAM**                             | 32GB (2x16GB) DDR5 (~$100 / 14만원)                 | 64GB (2x32GB) DDR5 (~$180 / 24만원)                          | 128GB (4x32GB) DDR5 (~$360 / 49만원)                         |
| **Storage**                         | 2TB NVMe SSD (~$150 / 20만원)                       | 2TB NVMe SSD (OS/App) + 2TB NVMe SSD (Project) (~$300 / 40만원) | 4TB NVMe SSD (OS/App) + 4TB NVMe SSD (Project) (~$600 / 81만원) |
| **기타 (M/B, Cooler, PSU, Case)**   | ~$500 / 67만원                                      | ~$800 / 108만원                                              | ~$1,500 / 203만원                                            |
| **하드웨어 총계 (\**Chardware\**)** | **~$1,650 / 223만원**                               | **~$2,830 / 381만원**                                        | **~$8,710 / 1,177만원**                                      |

이처럼 RealityScan 소프트웨어 자체는 무료로 접근할 수 있지만, 그 잠재력을 최대한 활용하기 위해서는 최소 200만원 이상의 초기 하드웨어 투자가 필요하며, 전문적인 작업을 위해서는 400만원 이상의 시스템이 권장된다. 여기에 데이터 수집 장비 비용까지 고려하면, '무료'라는 단어 이면에 숨겨진 실질적인 투자 규모를 명확히 인지해야 한다.


RealityScan의 강력한 기능과 에픽 생태계와의 통합은 다양한 산업 분야에서 혁신적인 워크플로우를 가능하게 한다. 하지만 이러한 잠재력을 온전히 활용하기 위해서는 소프트웨어의 특성과 학습 과정에 대한 이해가 선행되어야 한다.


RealityScan은 특정 산업 분야에 국한되지 않고, 현실을 디지털로 옮기는 모든 작업에 적용될 수 있다.

- **게임 및 VFX**: 이 분야는 RealityScan의 가장 큰 수혜자다. CD PROJEKT RED가 'The Witcher 4' 테크 데모 제작에 RealityScan을 활용한 사례는 대표적이다.27 개발팀은 실제 숲의 바위, 나무, 지면 등을 스캔하여 고밀도 3D 에셋을 제작했다. AI 마스킹 기능으로 현장에서 배경을 신속하게 분리하고, 개선된 정렬 알고리즘으로 복잡한 유기체의 형태를 안정적으로 캡처했다. 이렇게 생성된 고품질 Nanite 메시는 별도의 최적화 과정 없이 언리얼 엔진 5에 바로 적용되어, 극사실적인 가상 세계를 구축하는 데 결정적인 역할을 했다. 이는 에셋 제작 파이프라인을 획기적으로 단축시키고, 아티스트가 창의적인 작업에 더 집중할 수 있게 만든다.
- **디지털 트윈 및 시뮬레이션**: SAS(통계 분석 소프트웨어 기업)와의 협력은 RealityScan이 엔터테인먼트를 넘어 산업 현장으로 확장되는 가능성을 보여준다.15 SAS는 RealityScan을 이용해 Georgia-Pacific 제지 공장의 설비를 정밀하게 스캔하여 언리얼 엔진 내에 디지털 트윈을 구축했다. 이 가상 공장에서 SAS의 AI 분석 솔루션을 통해 무인운반차(AGV)의 이동 경로 최적화, 생산 라인 효율성 증대 등을 시뮬레이션한다.59 실제 공장 가동을 중단하지 않고도 다양한 시나리오를 테스트하고 최적의 해결책을 찾아낼 수 있어, 비용 절감과 생산성 향상에 직접적으로 기여한다.
- **건축, 엔지니어링, 건설 (AEC)**: 건설 현장에서 RealityScan은 프로젝트의 생애주기 전반에 걸쳐 활용된다. 드론이나 360도 카메라를 이용해 현장을 주기적으로 스캔하면, 공정 진행 상황을 시각적으로 문서화하고 원격으로 관리할 수 있다.60 설계 도면(BIM 모델)과 실제 시공 결과(as-built)를 3D로 비교하여 품질을 검수하고, 잠재적인 오류를 조기에 발견하여 재작업 비용을 줄일 수 있다.3
- **문화유산 보존 및 학술 연구**: 고고학 유적지나 박물관의 유물을 고해상도로 스캔하여 영구적인 디지털 아카이브를 구축하는 데 사용된다.3 디지털화된 유물은 온라인 전시, 학술 연구, 파손 시 복원 데이터 등 다방면으로 활용될 수 있다. RealityScan의 비접촉식 데이터 수집 방식은 훼손되기 쉬운 유물을 안전하게 기록하는 데 매우 효과적이다.


에픽게임즈는 RealityScan의 사용자 기반 확대를 위해 방대한 양의 학습 자료를 무료로 제공하고 있다. 이는 소프트웨어 도입의 가장 큰 장벽 중 하나인 학습 비용을 크게 낮추는 요인이다.

- **무료 학습 자료**:
  - **Epic Developer Community**: RealityScan 전용 포럼과 학습 섹션이 마련되어 있어, 공식 튜토리얼, 사용자 가이드, FAQ 등을 찾아볼 수 있다. 초보자를 위한 'Making a Complete Model' 62부터 UE5 연동 워크플로우 23, Mesh to MetaHuman 가이드 29까지 다양한 주제의 자료가 지속적으로 업데이트된다.
  - **공식 YouTube 채널**: RealityScan(구 Capturing Reality) 공식 채널은 수십 개의 영상 튜토리얼 재생목록을 제공한다.64 각 기능에 대한 짧은 팁부터 전체 프로젝트를 다루는 긴 웨비나까지 다양한 형식의 콘텐츠를 통해 실질적인 기술을 습득할 수 있다.
  - **커뮤니티 기반 학습**: Reddit의 r/photogrammetry와 같은 커뮤니티에서는 사용자들이 서로의 노하우를 공유하고 문제 해결을 돕는다.66
- **유료 교육 과정**: 보다 체계적이고 심도 있는 학습을 원하는 사용자를 위한 유료 교육 과정도 존재한다.
  - **CyArk**: 문화유산 디지털 보존 전문 비영리 단체로, RealityCapture를 활용한 지상 및 드론 포토그래메트리 데이터 수집, 처리, 모델링에 대한 전문적인 단기 코스를 온라인으로 제공한다.67
  - **온라인 교육 플랫폼**: Udemy, Domestika 등에서는 다양한 강사들이 포토그래메트리 기초부터 특정 소프트웨어(Agisoft Metashape 등) 활용법까지 다양한 가격대의 강의를 제공한다.68 RealityScan 전문 강좌는 아직 많지 않지만, 포토그래메트리 일반 원리를 배우는 데 도움이 될 수 있다.
  - **전문 교육 기관**: Lightpoint Data와 같은 기관에서는 법의학 재구성 등 특정 분야에 초점을 맞춘 고급 포토그래메트리 온라인 과정을 운영한다.70


RealityScan의 학습 곡선은 이중적인 특성을 보인다. 입문은 쉽지만, 숙달은 어렵다.

소프트웨어의 인터페이스는 경쟁 제품인 Agisoft Metashape나 3DF Zephyr에 비해 다소 복잡하고 직관성이 떨어진다는 평가가 있다.71 특히 수많은 설정 항목들이 왼쪽 하단의 작은 패널에 집중되어 있어, 초보자가 원하는 설정을 찾고 이해하기가 쉽지 않을 수 있다.22 이는 소프트웨어가 본래 전문가용으로 설계되었기 때문이다.

하지만 에픽게임즈는 이러한 진입 장벽을 낮추기 위해 강력한 자동화 기능을 전면에 내세웠다. 사용자는 사진을 드래그 앤 드롭하고 'Start' 버튼을 누르는 것만으로도 꽤 괜찮은 품질의 3D 모델을 얻을 수 있다.22 이는 포토그래메트리 경험이 전혀 없는 사람도 기술의 맛을 볼 수 있게 해준다.

문제는 '괜찮은' 결과물을 '최상의' 결과물로 만드는 과정에 있다. 최적의 결과를 얻기 위해서는 좋은 품질의 원본 사진을 촬영하는 방법부터 시작하여, 각 처리 단계(정렬, 메시화, 텍스처링)의 세부 설정이 결과물에 미치는 영향을 깊이 이해하고, 상황에 맞게 파라미터를 조정할 수 있어야 한다. 예를 들어, 정렬 시 특징점 개수를 조절하거나, 메시 생성 시 디테일 수준을 'Normal'과 'High' 중에서 적절히 선택하는 판단이 필요하다. 'High' 옵션은 디테일을 높이는 대신 노이즈를 증폭시킬 수 있기 때문이다.22

결론적으로, RealityScan은 '누구나 쉽게 시작할 수 있도록' 문을 활짝 열어두었지만, 그 문을 통과한 사용자가 전문가 수준의 결과물을 내기까지는 상당한 학습과 경험 축적이 필요하다. 에픽게임즈의 전략은 이러한 학습 과정에서 사용자들이 자연스럽게 자사의 방대한 무료 학습 자료와 커뮤니티에 깊이 관여하도록 유도하여, 생태계에 대한 충성도를 높이는 효과를 낳는다.


RealityScan의 도입을 결정하기 위해서는 시장 내 다른 주요 소프트웨어와의 비교를 통해 그 장단점과 포지셔닝을 객관적으로 파악하는 것이 중요하다. 포토그래메트리 소프트웨어 시장은 각각 뚜렷한 강점을 가진 여러 플레이어들이 경쟁하는 구도를 보이고 있다.


Agisoft Metashape

과거 PhotoScan으로 알려진 Metashape는 RealityScan의 가장 직접적이고 강력한 경쟁자다.72

- **강점**: Metashape의 최대 강점은 **정확도와 제어 가능성**이다. 정밀한 카메라 보정, 상세한 포인트 클라우드 편집, 그리고 강력한 Python 스크립팅 지원을 통해 사용자가 워크플로우의 모든 단계를 세밀하게 제어할 수 있다.73 이로 인해 고고학, 문화유산 기록, 정밀 측량, 학술 연구 등 결과물의 과학적 정확성이 중요한 분야에서 높은 평가를 받는다.73 또한, 

  **영구 라이선스** 모델을 유지하고 있어, 초기 투자 비용은 높지만 장기적으로는 구독료 부담 없이 소프트웨어를 소유할 수 있다는 장점이 있다.75

- **약점**: 처리 속도가 RealityScan에 비해 상대적으로 느린 경향이 있으며, 특히 고밀도 클라우드 생성 단계에서 차이가 두드러진다.73 학습 곡선 또한 가파른 편으로 평가된다.74

Pix4D

Pix4D는 단일 소프트웨어라기보다는, 특정 산업 분야에 최적화된 제품군(PIX4Dmapper, PIX4Dfields, PIX4Dcloud 등)으로 구성된 솔루션 스위트다.78

- **강점**: **사용 편의성과 클라우드 기반 협업**에 강점을 보인다. 특히 드론을 이용한 대규모 매핑 워크플로우에 최적화되어 있으며, 직관적인 인터페이스와 자동화된 처리 과정은 초보자도 쉽게 접근할 수 있게 한다.74 PIX4Dcloud 플랫폼을 통해 팀원들과 원격으로 데이터를 공유하고 협업하는 기능은 건설, 농업과 같은 현장 중심 산업에서 매우 유용하다.79
- **약점**: **구독 기반**의 가격 정책은 장기 사용자에게 비용 부담이 될 수 있다.74 또한, Metashape만큼의 세밀한 수동 제어 옵션은 부족하여, 고도로 맞춤화된 처리가 필요한 연구 분야에는 적합하지 않을 수 있다.

3DF Zephyr

3Dflow사가 개발한 3DF Zephyr는 유연한 제품 라인업과 합리적인 가격 정책으로 폭넓은 사용자층을 확보하고 있다.

- **강점**: **접근성과 직관적인 UI**가 가장 큰 장점이다. 50장의 사진 제한이 있는 완전 무료 버전부터 시작하여, Lite, Pro, Aerial 버전까지 사용자의 필요와 예산에 맞춰 선택의 폭이 넓다.80 인터페이스가 사용자 친화적으로 설계되어 있어, 초보자도 쉽게 워크플로우를 따라갈 수 있다는 평가를 받는다.72 Zephyr 역시 

  **영구 라이선스**를 제공하여 초기 비용 부담이 상대적으로 적다.80

- **약점**: RealityScan만큼의 압도적인 처리 속도나 Metashape 수준의 정밀한 후처리 기능을 기대하기는 어렵다. 기능의 깊이와 생태계 통합 측면에서는 거대 기업이 지원하는 경쟁 제품들에 비해 열세에 있다.

아래 표는 이들 주요 소프트웨어의 핵심적인 특징을 비교하여 RealityScan의 시장 내 위치를 명확히 보여준다.

**표 3: 주요 포토그래메트리 소프트웨어 비교**

| 구분                | **RealityScan**                                              | **Agisoft Metashape**                                        | **PIX4Dmapper**                                         | **3DF Zephyr**                                    |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------- | ------------------------------------------------- |
| **가격 모델**       | 구독 (조건부 무료)                                           | 영구 라이선스                                                | 구독 / 영구                                             | 영구 라이선스 (무료 버전 존재)                    |
| **주요 가격 (Pro)** | $1,250/년 (매출 $1M 초과 시)                                 | $3,499 (영구)                                                | $3,500/년 또는 $5,990 (영구)                            | $3,200 (Pro, 영구)                                |
| **핵심 강점**       | - 압도적인 처리 속도 - UE 생태계 완벽 통합 - 조건부 무료 정책 | - 높은 정확도 및 제어 - 강력한 스크립팅 지원 - 영구 라이선스 | - 사용자 친화적 UI - 클라우드 기반 협업 - 산업별 솔루션 | - 쉬운 사용법 - 유연한 가격 정책 - 무료 버전 제공 |
| **핵심 약점**       | - 높은 하드웨어 요구 (NVIDIA 필수) - 에픽 생태계 종속성      | - 상대적으로 느린 속도 - 높은 초기 비용                      | - 구독료 부담 - 제한된 수동 제어                        | - 기능적 깊이 부족 - 상대적으로 낮은 성능         |
| **주 사용 분야**    | 게임, VFX, 실시간 시각화, 디지털 트윈                        | 측량, 문화유산, 학술 연구, 정밀 모델링                       | 건설, 농업, 드론 매핑, 긴급 대응                        | 일반 3D 모델링, 교육, 취미                        |
| **생태계 통합성**   | 최상 (UE, MetaHuman, Fab)                                    | 중간 (Python, 범용 포맷)                                     | 높음 (GIS/CAD, API)                                     | 낮음 (범용 포맷)                                  |


이러한 경쟁 구도 속에서 RealityScan의 시장 포지셔닝은 명확하다. 에픽게임즈는 RealityScan을 '파괴적 혁신자(Disruptor)'로 내세워 기존 시장의 규칙을 흔들고 있다. 전통적인 소프트웨어 벤더들이 라이선스 판매를 통해 수익을 창출하는 반면, 에픽은 소프트웨어를 거의 무료로 배포하여 사용자들을 자사의 거대한 생태계로 끌어들이는 데 집중한다.

RealityScan의 핵심 경쟁력은 **'속도'와 '생태계'** 두 단어로 요약된다. CUDA 가속을 통한 압도적인 처리 속도는 생산성을 중시하는 모든 사용자에게 매력적이다.73 그러나 더 강력한 무기는 언리얼 엔진과의 완벽한 통합이다. Nanite를 활용한 고폴리곤 메시의 실시간 렌더링 워크플로우는 다른 소프트웨어들이 쉽게 따라올 수 없는, 에픽 생태계 안에서만 가능한 독점적인 가치다.

이러한 전략은 필연적으로 포토그래메트리 시장의 양극화를 초래할 가능성이 높다.

- **에픽 생태계 중심 시장**: 게임, 영화, 건축 시각화 등 언리얼 엔진의 영향력이 절대적인 분야에서는 RealityScan이 표준 도구로 자리 잡을 가능성이 크다. 개발자들은 작업 파이프라인의 효율성을 극대화하기 위해 자연스럽게 RealityScan을 선택하게 될 것이다.
- **독립적 전문가 시장**: 반면, 언리얼 엔진 의존도가 낮고, 소프트웨어의 독립적인 기능과 과학적 정확성을 더 중시하는 정밀 측량, GIS, 학술 연구 분야에서는 Agisoft Metashape와 같은 전통적인 강자들이 여전히 강력한 입지를 유지할 것이다. 이들은 에픽의 생태계에 종속되는 것을 경계하며, 개방성과 제어 가능성을 더 높은 가치로 평가할 수 있다.

결론적으로, RealityScan은 단독 소프트웨어로서의 경쟁을 넘어, '에픽게임즈 플랫폼'의 일부로서 시장 전체의 판도를 바꾸는 게임 체인저 역할을 하고 있다.


지금까지 RealityScan의 기술적 특징, 비용 구조, 실무 적용 사례, 그리고 시장 내 경쟁 구도를 다각도로 분석했다. 이를 바탕으로 RealityScan 도입의 장단점을 종합적으로 평가하고, 잠재 사용자 유형에 따른 최적의 도입 전략을 제언한다.


**명 (Pros):**

- **압도적인 처리 속도**: CUDA 가속과 out-of-core 알고리즘을 통해 대용량 데이터셋을 경쟁 제품 대비 월등히 빠르게 처리하여 생산성을 극대화한다.6
- **혁신적인 무료 정책**: 연간 총수익 $1M 미만의 개인, 학생, 소규모 스튜디오는 모든 기능을 무료로 사용할 수 있어, 금전적 진입 장벽이 사실상 없다.44
- **완벽한 Unreal Engine 생태계 통합**: Nanite, MetaHuman, Fab 마켓플레이스와의 유기적인 연동은 특히 게임 및 실시간 콘텐츠 제작자에게 타의 추종을 불허하는 효율적인 워크플로우를 제공한다.23
- **강력하고 유연한 기능**: 사진뿐만 아니라 레이저 스캔 데이터까지 통합 처리할 수 있으며, 자동화 기능과 수동 제어 옵션을 모두 제공하여 초보자부터 전문가까지 아우를 수 있다.8
- **풍부한 무료 학습 자료**: 에픽게임즈가 제공하는 방대한 양의 공식 튜토리얼과 커뮤니티 지원은 학습 비용과 시간을 크게 절약해준다.62

**암 (Cons):**

- **높은 하드웨어 요구사항**: 핵심 기능 활용을 위해 고사양의 NVIDIA GPU가 필수적이며, 이는 초기 하드웨어 투자 비용을 증가시키는 주요 요인이다.18
- **가파른 학습 곡선 (전문가 활용 시)**: 직관성이 다소 떨어지는 UI와 방대한 설정 옵션으로 인해, 소프트웨어의 모든 잠재력을 끌어내기까지는 상당한 학습 노력이 필요하다.71
- **구독 모델의 한계**: 연 매출 $1M을 초과하는 대기업의 경우, 매년 반복적으로 발생하는 구독 비용이 부담이 될 수 있으며, 영구 라이선스를 선호하는 조직의 정책과 충돌할 수 있다.
- **생태계 종속성 심화**: 에픽 생태계 내에서의 효율성은 압도적이지만, 이는 반대로 에픽의 플랫폼과 정책에 대한 의존도를 높이는 결과를 낳는다.


사용자의 소속, 주력 분야, 예산, 기술적 요구사항에 따라 RealityScan의 도입 전략은 달라져야 한다.

- **개인, 학생, 소규모 스튜디오 (연 매출 < $1M)**
  - **결론: 강력 추천.** 비용 부담 없이 현존 최고 성능의 포토그래메트리 툴을 상업적 프로젝트에까지 활용할 수 있는 전례 없는 기회다. 유일한 장벽은 초기 하드웨어 투자 비용이므로, 표 2에서 제시된 '입문 전문가용' 워크스테이션 구축을 목표로 예산을 계획하는 것이 합리적이다.
- **게임/VFX/실시간 콘텐츠 제작사**
  - **결론: 필수 도입 고려.** 주력 개발 엔진이 언리얼 엔진이라면, RealityScan(또는 Unreal Subscription 번들)은 선택이 아닌 필수에 가깝다. Nanite와의 연동을 통한 파이프라인 혁신은 개발 기간 단축과 결과물 품질 향상에 직접적으로 기여하므로, 투자 가치가 매우 높다.
- **측량/건설/학술 연구 기관**
  - **결론: 신중한 비교 검토 후 결정.** 처리 속도가 프로젝트의 성패를 좌우하는 경우 RealityScan은 매우 매력적인 선택지다. 하지만 Python 스크립트를 통한 고도의 자동화 및 분석, 특정 포맷 지원, 오프라인 환경에서의 영구 라이선스 사용이 필수적이라면 Agisoft Metashape가 더 나은 대안일 수 있다.73 두 소프트웨어의 평가판을 모두 사용하여 실제 프로젝트 데이터로 테스트해보고, 조직의 워크플로우와 요구사항에 가장 부합하는 솔루션을 선택하는 것이 현명하다.
- **대기업 (연 매출 > $1M)**
  - **결론: TCO 기반의 전략적 결정.** 사용자당 연간 $1,250 또는 $1,850의 구독 비용이 발생하는 만큼, 장기적인 총 소유 비용을 신중하게 분석해야 한다. 5년간의 TCO를 산출하여 Metashape의 영구 라이선스 비용과 비교하고, 에픽 생태계 통합이 제공하는 생산성 향상 가치를 정량적으로 평가하여 도입 여부를 결정해야 한다. 25개 이상의 대량 라이선스 구매 시에는 에픽게임즈와의 맞춤형 라이선스 협상도 가능하다.43


에픽게임즈가 RealityScan을 통해 그리는 미래는 단순히 3D 에셋을 만드는 것을 넘어선다. 그들의 궁극적인 비전은 현실 세계 전체를 가상 공간에 정밀하게 복제하는 **'디지털 트윈'**과, 그 위에서 상호작용하는 **'메타버스'**를 구축하는 것이다.15

이 거대한 비전 속에서 RealityScan은 물리적 세계의 정보를 디지털 세계로 옮기는 가장 중요한 '입력 장치(Input Device)'이자, 모든 가상 세계의 근간이 되는 현실 기반 데이터(Ground Truth)를 생성하는 핵심 기술로 자리매김한다. 이미 제조(SAS), 게임(CDPR) 분야에서 그 가능성을 입증했으며, 앞으로 의료, 도시 계획, 자율주행 시뮬레이션 등 적용 분야는 무한히 확장될 것이다.15

따라서 RealityScan을 도입하고 활용하는 것은 단순히 현재의 3D 모델링 작업을 효율화하는 것을 넘어, 다가오는 디지털 트윈과 메타버스 시대의 새로운 기회를 선점하기 위한 전략적 투자로 이해될 수 있다. 에픽게임즈가 제공하는 강력하고 접근성 높은 도구를 통해 현실을 디지털로 변환하는 능력은 미래 디지털 경제의 핵심 경쟁력이 될 것이다. RealityScan은 그 경쟁의 출발선에 서 있는 가장 강력한 도구 중 하나다.


1. Reality Capture Process: A Comprehensive Guide - FlyPix AI, accessed July 23, 2025, https://flypix.ai/blog/reality-capture-process/
2. What is Reality Capture? | Latest Technology Trend and 5 Benefits - Autodesk, accessed July 23, 2025, https://www.autodesk.com/solutions/reality-capture
3. 3D scanning software for games, AEC, geospatial, simulation, media - RealityScan, accessed July 23, 2025, https://www.realityscan.com/en-US/uses
4. 3D Reality Capture Workflows for AEC and Industry40 - Cintoo, accessed July 23, 2025, https://cintoo.com/blog/3d-reality-capture
5. Benefits Of Reality Capture And Why You Should Use It - REBIM®, accessed July 23, 2025, https://rebim.io/benefits-of-reality-capture/
6. RealityCapture, Epic Games, Building the Ultimate Simulation - Mosaic51, accessed July 23, 2025, https://www.mosaic51.com/featured/reality-capture-epic-games-ultimate-simulation/
7. en.wikipedia.org, accessed July 23, 2025, https://en.wikipedia.org/wiki/RealityCapture
8. RealityCapture - MIT - Docubase, accessed July 23, 2025, https://docubase.mit.edu/tools/realitycapture/
9. Epic acquires Capturing Reality | GamesIndustry.biz, accessed July 23, 2025, https://www.gamesindustry.biz/epic-acquires-capturing-reality
10. Capturing Reality is now part of Epic Games, accessed July 23, 2025, https://www.epicgames.com/site/en-US/news/capturing-reality-is-now-part-of-epic-games
11. Capturing Reality Joins Epic Games - InvestGame, accessed July 23, 2025, https://investgame.net/news/capturing-reality-joins-epic-games/
12. Epic buys RealityCapture creator Capturing Reality - CG Channel, accessed July 23, 2025, https://www.cgchannel.com/2021/03/epic-games-buys-realitycapture-developer-capturing-reality/
13. My - Sign in - Capturing Reality, accessed July 23, 2025, https://www.capturingreality.com/my
14. RealityScan 2.0: New release brings powerful new features to a rebranded RealityCapture, accessed July 23, 2025, https://www.realityscan.com/en-US/news/realityscan-20-new-release-brings-powerful-new-features-to-a-rebranded-realitycapture
15. SAS digital twins on Unreal Engine transform manufacturing, accessed July 23, 2025, https://www.sas.com/en_ie/news/press-releases/2025/may/enhanced-digital-twins.html
16. From Reality to Virtual Worlds: The Role of Photogrammetry in Game Development - arXiv, accessed July 23, 2025, https://arxiv.org/html/2505.16951v1
17. Epic games buys RealityCapture developer - Peter Falkingham, accessed July 23, 2025, https://peterfalkingham.com/2021/03/10/epic-games-buys-realitycapture-developer/
18. Hardware and Operating System Requirements | Epic Developer Community, accessed July 23, 2025, https://dev.epicgames.com/community/learning/knowledge-base/DB58/realityscan-hardware-and-operating-system-requirements
19. Hardware Recommendations for RealityScan - Puget Systems, accessed July 23, 2025, https://www.pugetsystems.com/solutions/photogrammetry-workstations/realityscan/hardware-recommendations/
20. FAQ - RealityScan - RealityCapture Training, accessed July 23, 2025, https://www.realitycapture-training.com/en/faq-realityscan/
21. RealityScan Help, accessed July 23, 2025, https://rshelp.capturingreality.com/
22. No more Agisoft? Turn to Reality Capture with this guide - Peter Falkingham, accessed July 23, 2025, https://peterfalkingham.com/2024/11/05/no-more-agisoft-turn-to-reality-capture-with-this-guide/
23. RealityCapture to Unreal Engine: Beginner's Guide to Photogrammetry Workflow, accessed July 23, 2025, https://dev.epicgames.com/community/learning/tutorials/W4MR/realityscan-realitycapture-to-unreal-engine-beginner-s-guide-to-photogrammetry-workflow
24. RealityCapture to Unreal Engine: Beginner's Guide to Photogrammetry Workflow - YouTube, accessed July 23, 2025, https://www.youtube.com/watch?v=uwTLiE0PsF4
25. RealityCapture to Unreal Engine 5 - YouTube, accessed July 23, 2025, https://www.youtube.com/watch?v=kRD0rgCnOWQ
26. RealityCapture to UE5 - Workflow Tutorial - YouTube, accessed July 23, 2025, https://www.youtube.com/watch?v=WrCOhes1Zgg
27. How CD PROJEKT RED used RealityScan to help create 'The Witcher 4 Unreal Engine 5 Tech Demo', accessed July 23, 2025, https://www.realityscan.com/en-US/spotlights/how-cd-projekt-red-used-realityscan-to-create-the-witcher-4-unreal-engine-5-tech-demo
28. Community Tutorial: Making real people into MetaHumans: RealityScan to Unreal Engine 5 Mesh to MetaHuman, accessed July 23, 2025, https://forums.unrealengine.com/t/community-tutorial-making-real-people-into-metahumans-realityscan-to-unreal-engine-5-mesh-to-metahuman/1868619
29. Making real people into MetaHumans: RealityScan to Unreal Engine 5 Mesh to MetaHuman, accessed July 23, 2025, https://dev.epicgames.com/community/learning/tutorials/0Jey/realityscan-making-real-people-into-metahumans-realityscan-to-unreal-engine-5-mesh-to-metahuman
30. Scan yourself for Mesh to MetaHuman - Epic Games Developers, accessed July 23, 2025, https://dev.epicgames.com/community/learning/tutorials/Ovd2/realityscan-scan-yourself-for-mesh-to-metahuman
31. Scan Yourself for Mesh to MetaHuman | RealityCapture - - Toolfarm, accessed July 23, 2025, https://www.toolfarm.com/tutorial/scan-yourself-for-mesh-to-metahuman-realitycapture/
32. Tutorial: Scanning Yourself for Mesh to MetaHuman in RealityCapture - 80 Level, accessed July 23, 2025, https://80.lv/articles/tutorial-scanning-yourself-for-mesh-to-metahuman-in-realitycapture
33. (Sketchfab) RealityCapture - Sign in to Your Epic Games account, accessed July 23, 2025, https://support.fab.com/s/article/RealityCapture
34. Fab, Epic's New Unified Content Marketplace, Launches Today! - Sketchfab, accessed July 23, 2025, https://sketchfab.com/blogs/community/epics-unified-marketplace-fab-launches-today/
35. How To Use Fab Marketplace Inside Of Unreal Engine Projects (Tutorial) - YouTube, accessed July 23, 2025, https://www.youtube.com/watch?v=HD62M6MuTkw
36. RealityCapture gets new 'pay-per-input' pricing - CG Channel, accessed July 23, 2025, https://www.cgchannel.com/2019/09/realitycapture-gets-new-pay-per-input-pricing/
37. Capturing Reality introduces PPI (Pay Per Input) pricing - CGPress, accessed July 23, 2025, https://cgpress.org/archives/capturing-reality-introduces-ppi-pay-per-input-pricing.html
38. Capturing Reality licenses images instead of software - Geo Week News, accessed July 23, 2025, https://www.geoweeknews.com/news/capturing-reality-licenses-images-instead-of-software
39. RealityScan | Epic Developer Community, accessed July 23, 2025, https://dev.epicgames.com/community/realityscan/getting-started/realityscan
40. We are updating Unreal Engine, Twinmotion, and RealityCapture pricing in late April, accessed July 23, 2025, https://www.unrealengine.com/en-US/blog/we-are-updating-unreal-engine-twinmotion-and-realitycapture-pricing-in-late-april
41. RealityCapture is now free to indie artists and studios - CG Channel, accessed July 23, 2025, https://www.cgchannel.com/2024/04/realitycapture-1-4-makes-the-photogrammetry-app-free-to-indie-artists/
42. RealityCapture Price Change Incoming, Goes Free Starting April - All3DP, accessed July 23, 2025, https://all3dp.com/4/realitycapture-price-change-incoming-goes-free-starting-april/
43. RealityScan licensing options, accessed July 23, 2025, https://www.realityscan.com/en-US/license
44. RealityScan | 3D models from images and laser scans - RealityScan, accessed July 23, 2025, https://www.realityscan.com/
45. Download RealityScan, accessed July 23, 2025, https://www.realityscan.com/en-US/download
46. RealityScan - - Toolfarm, accessed July 23, 2025, https://www.toolfarm.com/buy/realityscan/
47. RealityCapture - CG River, accessed July 23, 2025, https://www.cgriver.com/products/capturing-reality-reality-capture-enterprise
48. Twinmotion Licensing and Pricing, accessed July 23, 2025, https://www.twinmotion.com/en-US/license
49. RealityCapture Requirements: What You Need for Optimal Performance, accessed July 23, 2025, https://flypix.ai/blog/reality-capture-requirements/
50. Shop Canon EOS R8 | Canon U.S.A., Inc., accessed July 23, 2025, https://www.usa.canon.com/shop/p/eos-r8
51. Canon EOS R8 Price Watch and Comparison, accessed July 23, 2025, https://www.cpricewatch.com/product/07796/Canon-EOS-R8-price.html?streetprice
52. Sigma 35mm f/1.4 DG DN Art Lens for Sony E - B&H, accessed July 23, 2025, https://www.bhphotovideo.com/c/product/1636130-REG/sigma_303965_35mm_f_1_4_dg_dn.html
53. SIGMA 35mm F1.4 DG DN | Art, accessed July 23, 2025, https://www.sigmaphoto.com/35mm-f1-4-dg-dn-a
54. Buy DJI Mavic 3 Pro - DSLRPros, accessed July 23, 2025, https://www.dslrpros.com/products/dji-mavic-3-pro
55. Buy DJI Mavic 3 Pro - Triple-Lens Flagship Camera Drone, accessed July 23, 2025, https://store.dji.com/product/dji-mavic-3-pro
56. Godox SL-60W Daylight LED Video 2-Light Kit with 2 Stand - 2 Softbox and Case, accessed July 23, 2025, https://thephotocenter.com/shop/godox-sl-60w-daylight-led-video-2-light-kit-with-2-stand-2-softbox-and-case/c12fee20-85bf-0139-be04-00163e90e196?variation=2956650
57. Godox SL60W Pro Photography Kit with Softbox, 5600K Studio LED Light, Bowens Mount, accessed July 23, 2025, https://www.walmart.com/ip/Godox-SL60W-Kit-with-Soft-Box-Softbox-5600K-Studio-Continuous-LED-Video-Light-Lamp-5600K-Bowens-Mount-for-Video-Recording/2700030253
58. SAS Unveils Digital Twins on Unreal Engine | Channel Insider, accessed July 23, 2025, https://www.channelinsider.com/news-and-trends/sas-epic-games-partnership/
59. SAS, Epic Games & Georgia-Pacific Redefine Digital Twins with Unreal Engine - YouTube, accessed July 23, 2025, https://www.youtube.com/watch?v=qudSxO2GWSg
60. Reality Capture 101 | Guide to Reality Capture in Construction - OpenSpace, accessed July 23, 2025, https://www.openspace.ai/blog/reality-capture-101/
61. Reality Capture in Construction: Benefits, Tools & More - Matterport, accessed July 23, 2025, https://matterport.com/blog/reality-capture-construction
62. Making a complete model in RealityCapture | Epic Developer Community, accessed July 23, 2025, https://dev.epicgames.com/community/learning/tutorials/ow4e/realityscan-making-a-complete-model-in-realitycapture
63. RealityCapture to UE5 - Workflow Tutorial | Epic Developer Community, accessed July 23, 2025, https://dev.epicgames.com/community/learning/tutorials/j9vV/unreal-engine-realityscan-realitycapture-to-ue5-workflow-tutorial
64. Reality Capture - YouTube, accessed July 23, 2025, https://www.youtube.com/playlist?list=PL8DLoIwEdbnm1ka8pqVvFpRReLlYIreZd
65. Tutorials - YouTube, accessed July 23, 2025, https://www.youtube.com/playlist?list=PL56jeA0rCS3LWuahdfIFWp1d0WDuEKVqe
66. Reality Capture tutorial ideas : r/photogrammetry - Reddit, accessed July 23, 2025, https://www.reddit.com/r/photogrammetry/comments/1bofphd/reality_capture_tutorial_ideas/
67. Photogrammetry Courses and Training - CyArk, accessed July 23, 2025, https://www.cyark.org/whatwedo/training/
68. Top Photogrammetry Courses Online - Updated [July 2025] - Udemy, accessed July 23, 2025, https://www.udemy.com/topic/photogrammetry/
69. Online Course - Introduction to 3D Photogrammetry (David Chumilla) - Domestika, accessed July 23, 2025, https://www.domestika.org/en/courses/666-introduction-to-3d-photogrammetry
70. Advanced Photogrammetry - Lightpoint Data, accessed July 23, 2025, https://lightpointdata.com/advanced-photogrammetry
71. Reality Capture - New photogrammetry software package., accessed July 23, 2025, https://forums.culturalheritageimaging.org/topic/452-reality-capture-new-photogrammetry-software-package/
72. What is the best photogrammetry software in 2025? - 3D Mag, accessed July 23, 2025, https://www.3dmag.com/reviews/photogrammetry-software/what-is-the-best-photogrammetry-software-in-2025/
73. Agisoft Metashape vs RealityCapture: Best Photogrammetry Software for 2025?, accessed July 23, 2025, https://www.agisoftmetashape.com/agisoft-metashape-vs-realitycapture-best-photogrammetry-software-for-2025/
74. Agisoft Metashape vs Pix4D: Which Photogrammetry Software Is Best, accessed July 23, 2025, https://www.agisoftmetashape.com/agisoft-metashape-vs-pix4d-which-photogrammetry-software-is-best/
75. Is Metashape Free to Use? Everything You Need to Know About Pricing and Trial Options, accessed July 23, 2025, https://www.agisoftmetashape.com/is-metashape-free-to-use-everything-you-need-to-know-about-pricing-and-trial-options/
76. Buy - Agisoft Metashape, accessed July 23, 2025, https://www.agisoft.com/buy/online-store/
77. Licensing Options - Agisoft Metashape, accessed July 23, 2025, https://www.agisoft.com/buy/licensing-options/
78. PIX4D Drone Software for Mapping & Surveying | Dronefly, accessed July 23, 2025, https://www.dronefly.com/collections/pix4d
79. Pricing plans for PIX4Dcloud platform | Pix4D, accessed July 23, 2025, https://www.pix4d.com/pricing/pix4dcloud
80. 3DF Zephyr Photogrammetry Software Review - 3D Mag, accessed July 23, 2025, https://www.3dmag.com/reviews/3df-zephyr-photogrammetry-software-review/
81. 3DF Zephyr - photogrammetry software - 3d models from photos - 3Dflow, accessed July 23, 2025, https://www.3dflow.net/3df-zephyr-photogrammetry-software/
82. RealityCapture Pricing: Everything You Need to Know - FlyPix AI, accessed July 23, 2025, https://flypix.ai/blog/reality-capture-pricing/
83. Epic Games Makes Digital Twins Efficient - XR Today, accessed July 23, 2025, https://www.xrtoday.com/mixed-reality/epic-games-makes-digital-twins-efficient/

