# VDB-Mapping



VDB-Mapping은 3차원 공간을 모델링하고 이해하는 방식에 있어 중요한 진보를 대표하는 기술 프레임워크다. 본질적으로 VDB-Mapping은 희소(sparse)하고 동적으로 변화하는 대규모 볼류메트릭 데이터(volumetric data)를 극도로 효율적인 방식으로 표현, 저장, 조작 및 분석하기 위해 설계되었다.1 여기서 '희소하다'는 것은 표현하고자 하는 3차원 공간의 대부분이 비어있거나 관심 대상이 아닌 균일한 값으로 채워져 있음을 의미한다. 예를 들어, 광활한 하늘에 떠 있는 구름, 실내 공간의 가구 배치, 혹은 LiDAR 센서로 스캔한 도로 환경 등은 모두 데이터가 특정 영역에 집중된 희소한 특성을 보인다. VDB-Mapping은 바로 이러한 데이터의 내재적 희소성을 활용하여 메모리 사용량을 획기적으로 줄이고 데이터 접근 및 처리 속도를 비약적으로 향상시키는 것을 목표로 한다.1

본 보고서는 VDB-Mapping의 근간을 이루는 핵심 자료 구조의 철학적 배경과 구조적 독창성에서부터 출발하여, 이것이 로보틱스와 컴퓨터 그래픽스 분야에서 구체적으로 어떻게 구현되고 응용되는지를 심층적으로 분석한다. 나아가, 현재 기술이 직면한 한계를 진단하고, 딥러닝과 같은 최신 기술과의 융합을 통해 열리고 있는 새로운 연구 지평과 미래 발전 가능성까지 포괄적으로 고찰하는 것을 궁극적인 목적으로 한다. 이를 통해 VDB-Mapping 기술의 본질을 다각적으로 조명하고, 관련 분야의 연구자 및 개발자에게 깊이 있는 통찰을 제공하고자 한다.


VDB-Mapping을 논하기에 앞서, 용어의 잠재적 모호성을 해소하는 것은 필수적이다. 'VDB'라는 약어는 기술 분야에 따라 전혀 다른 두 가지 개념을 지칭하는 데 사용될 수 있어 혼동을 유발하기 쉽다.

첫째는 데이터베이스 가상화(database virtualization) 분야에서 사용되는 '가상 데이터베이스(Virtual Database)'다. Delphix나 Teiid와 같은 플랫폼에서 언급되는 VDB는 물리적인 데이터 소스(예: Oracle, MySQL)의 스냅샷을 기반으로 생성된, 읽고 쓰기가 가능한 논리적 데이터베이스 복사본을 의미한다.4 이러한 VDB는 주로 개발, 테스트, 분석 환경에서 데이터 관리의 민첩성과 효율성을 높이기 위한 목적으로 사용되며, 실제 데이터를 복제하지 않고 메타데이터와 포인터를 통해 가상 환경을 제공하는 기술이다.5

반면, 본 보고서에서 중점적으로 다룰 VDB는 완전히 다른 개념이다. 여기서의 VDB는 **Volumetric, Dynamic B+tree**의 약어로, 2013년 드림웍스 애니메이션(DreamWorks Animation)의 켄 무세스(Ken Museth)에 의해 개발된 혁신적인 3차원 데이터 구조를 지칭한다.1 이 자료 구조는 B+Tree의 효율적인 인덱싱 특성을 3차원 그리드에 적용하여, 희소 볼류메트릭 데이터를 압축적으로 저장하고 빠르게 접근하기 위해 탄생했다.1 이후 이 기술은 오픈소스 C++ 라이브러리인 OpenVDB로 공개되어 컴퓨터 그래픽스, 시뮬레이션, 로보틱스 분야의 산업 표준으로 자리 잡았다.9 심지어 OpenVDB의 공식 FAQ 문서에서도 초기에 'DB+Grid'라는 이름도 고려되었으나, 기존의 다른 희소 자료 구조와의 차별성을 강조하기 위해 'VDB'라는 이름이 채택되었음을 밝히고 있다.3

이처럼 두 VDB는 이름만 같을 뿐, 그 기반 기술, 목적, 응용 분야가 완전히 상이하다. 따라서 본 보고서의 전반에 걸쳐 'VDB' 및 'VDB-Mapping'은 후자인 볼류메트릭 데이터 구조 및 이를 활용한 매핑 기술을 지칭하는 것으로 한정한다. 이러한 명확한 구분을 통해 독자는 VDB-Mapping의 기술적 본질에 온전히 집중할 수 있을 것이다.

보고서의 구조 및 전개 방향 제시

본 보고서는 VDB-Mapping에 대한 체계적이고 심층적인 이해를 돕기 위해 다음과 같은 구조로 전개된다.

먼저, **II장 'VDB의 철학 및 핵심 자료 구조'**에서는 VDB-Mapping의 근간이 되는 OpenVDB 자료 구조의 설계 철학과 핵심 원리를 파헤친다. B+Tree와 그리드를 결합한 독창적인 계층 구조, 희소 데이터를 효율적으로 표현하는 메커니즘, 그리고 좌표계 시스템에 대해 상세히 분석한다.

다음으로, **III장 'VDB-Mapping의 구현 원리: TSDF 통합을 중심으로'**에서는 이론적 자료 구조가 실제 로보틱스 매핑에서 어떻게 구현되는지를 설명한다. 산업 표준 라이브러리인 OpenVDB의 역할과 함께, 센서 데이터를 맵에 통합하는 핵심 과정인 Ray-Casting과 TSDF(Truncated Signed Distance Field) 융합 규칙을 수식을 포함하여 구체적으로 다룬다.

**IV장 '주요 3D 매핑 기법과의 성능 비교 분석'**에서는 VDB-Mapping을 기존의 대표적인 3D 매핑 기술들(고정 그리드 맵, 옥트리, 서펠 기반 매핑)과 다각적으로 비교한다. 메모리 효율성, 연산 속도, 표현력 등의 측면에서 각 기술의 장단점을 명확히 제시하고 VDB-Mapping의 상대적 우위를 논증한다.

**V장 'VDB-Mapping의 응용 분야 확장'**에서는 VDB 기술이 탄생한 컴퓨터 그래픽스 분야를 시작으로 로보틱스, 3D 재구성, 그리고 자율주행차와 드론과 같은 자율 시스템에 이르기까지 광범위한 응용 사례를 구체적으로 살펴본다.

**VI장 'VDB-Mapping의 기술적 한계와 미래 전망'**에서는 현재 VDB-Mapping 기술이 직면한 기술적 과제들을 비판적으로 검토하고, 이를 극복하기 위한 최신 연구 동향을 소개한다. 특히 딥러닝과의 융합을 통해 열리고 있는 fVDB, NeuralVDB와 같은 차세대 기술의 가능성을 집중적으로 조명한다.

마지막으로, **VII장 '결론'**에서는 앞선 모든 논의를 종합하여 VDB-Mapping의 핵심 가치와 기술적 의의를 요약하고, 향후 발전 방향에 대한 종합적인 평가와 제언으로 보고서를 마무리한다.


VDB 자료 구조의 탁월함은 단순히 기존 기술을 개선한 것을 넘어, 희소 볼류메트릭 데이터를 바라보는 관점 자체를 전환했다는 데 있다. 전통적인 공간 분할 기법, 예를 들어 옥트리(Octree)가 공간을 우선적으로 균일하게 나누고 그 안에 데이터가 있는지를 확인하는 '공간 중심적(space-centric)' 접근 방식을 취하는 반면, VDB는 데이터가 존재하는 위치를 효율적으로 색인하고 그 외의 방대한 빈 공간은 암묵적으로 처리하는 '데이터 중심적(data-centric)' 접근 방식을 채택했다.1 이러한 철학적 전환은 VDB의 모든 구조적 설계에 깊이 스며들어 있으며, 이는 메모리 효율성과 연산 속도에서 혁신적인 성능 향상을 이끌어내는 근본적인 원인이 된다.


VDB의 가장 큰 구조적 특징은 데이터베이스 인덱싱에 널리 사용되는 B+Tree의 원리를 3차원 그리드 공간에 접목한 독창적인 하이브리드 구조라는 점이다.1 B+Tree는 모든 리프 노드(leaf node)가 동일한 깊이에 위치하여 균형 잡힌 탐색 경로를 보장하고, 노드 내 키(key)들이 정렬되어 있어 빠른 검색이 가능한 자료 구조다. VDB는 이러한 특성을 3차원 공간 인덱싱에 적용하여, 얕고 넓은(shallow and wide) 트리 구조를 형성함으로써 데이터 접근에 필요한 탐색 깊이를 최소화한다.11

VDB의 계층 구조는 일반적으로 최상위의 루트 노드(Root Node), 중간 단계의 내부 노드(Internal Node), 그리고 가장 하위의 리프 노드(Leaf Node)로 구성된다.3 이 구조의 핵심적인 특징은 다음과 같다.

- **가변적/고정적 분기율(Branching Factor)**: 루트 노드는 필요에 따라 동적으로 자식 노드를 추가할 수 있어, 사실상 무한한 공간으로의 확장을 지원한다. 반면, 내부 노드와 리프 노드는 컴파일 타임에 정의된 고정된 크기의 자식 그리드를 가진다.2 예를 들어, 표준적인 

  `Tree_float_5_4_3` 구성에서 최상위 내부 노드(5-노드)는 32×32×32개의 자식 포인터를, 그 다음 내부 노드(4-노드)는 16×16×16개의 포인터를, 그리고 리프 노드(3-노드)는 8×8×8개의 실제 복셀(voxel) 데이터를 담는 구조를 가진다.9

- **효율적인 좌표 계산**: 각 노드의 차원이 2의 거듭제곱으로 고정되어 있기 때문에, 특정 3차원 인덱스 좌표 $(i, j, k)$가 트리의 어느 노드, 어느 위치에 해당하는지를 비트 연산(bitwise operation)만으로 매우 빠르게 계산할 수 있다.2 이는 복잡한 계산 없이 좌표로부터 데이터 위치로의 직접적인 매핑을 가능하게 하여, VDB의 빠른 임의 접근(random access) 성능의 기반이 된다.

- **무한 인덱스 공간(Effectively Infinite Index Space)**: VDB는 32비트 또는 64비트 부호 있는 정수(signed integer)를 좌표로 사용한다. 이는 이론적으로 거의 무한대에 가까운 방대한 3차원 공간을 표현할 수 있음을 의미한다.1 특히 음수 좌표를 지원하기 때문에, 맵의 원점을 중심으로 모든 방향으로 제약 없이 공간을 확장해 나갈 수 있어, 크기나 위치가 동적으로 변하는 시뮬레이션이나 대규모 환경 매핑에 매우 유리하다.2

이러한 VDB의 구조는 옥트리와 근본적인 차이를 보인다. 옥트리는 공간을 8개의 하위 공간으로 재귀적으로 분할하며, 데이터가 없는 빈 공간이라도 표현하기 위해 노드가 존재해야 한다. 이로 인해 고해상도에서는 트리의 깊이가 매우 깊어질 수 있으며, 이는 탐색 시간의 증가로 이어진다.13 반면 VDB는 데이터가 있는 곳에만 노드를 생성하고, 얕은 트리 구조를 유지함으로써 방대한 희소 공간에서도 일관되게 빠른 성능을 제공한다. 이것이 바로 '공간 분할'에서 '데이터 인코딩'으로의 패러다임 전환이 가져온 실질적인 이점이다.


VDB의 설계 철학은 희소한 데이터를 얼마나 효율적으로 압축하고 처리할 수 있는가에 집중되어 있다. 이를 위해 VDB는 여러 독창적인 개념을 도입했다.

- **활성/비활성 복셀 (Active/Inactive Voxels)**: VDB 그리드 내의 모든 복셀은 '활성(active)' 또는 '비활성(inactive)' 상태를 가진다.12 활성 복셀은 사용자가 명시적으로 설정한 유의미한 데이터 값을 가지는 복셀들이다. 예를 들어, 연기 시뮬레이션에서는 밀도 값이 0보다 큰 영역, 레벨셋(level set)에서는 표면 주변의 좁은 밴드(narrow-band) 영역이 활성 복셀에 해당한다.14 반면, 비활성 복셀은 이러한 활성 영역을 제외한 나머지 모든 공간을 의미한다. VDB는 이 수많은 비활성 복셀들의 값을 개별적으로 저장하는 대신, 그리드 전체에 대해 단 한 번만 정의되는 '배경 값(background value)'을 사용한다.9 이 간단한 아이디어만으로도 전체 볼륨 데이터의 대부분을 차지하는 빈 공간에 대한 메모리 할당을 완전히 제거할 수 있다.

- **타일 (Tiles)**: VDB의 압축 효율성을 극대화하는 또 다른 핵심 개념은 '타일'이다.12 타일은 리프 노드가 아닌 상위 레벨의 노드(루트 노드 또는 내부 노드)에 직접 값을 저장하는 기능이다. 만약 어떤 내부 노드에 타일 값이 설정되면, 그 노드가 관장하는 모든 하위 공간(수백만, 수십억 개의 복셀을 포함할 수 있는)은 별도의 데이터를 저장하지 않고 모두 해당 타일 값으로 채워진 것으로 간주된다.2 예를 들어, 레벨셋 표현에서 표면으로부터 멀리 떨어진 외부 공간 전체를 나타내는 내부 노드에 최대 거리 값을 타일로 설정하면, 그 광대한 영역을 단 하나의 값으로 표현할 수 있다. 이는 균일한 값을 갖는 넓은 영역을 표현하는 데 있어 극도로 효율적인 압축 방식이다.

- **비트마스크 (Bitmasks)**: VDB의 빠른 접근 속도를 보장하는 숨겨진 무기는 각 노드 내에 존재하는 두 종류의 비트마스크다.2

  - **자식 마스크(Child Mask)**: 이 비트마스크는 해당 노드의 자식 노드들이 그리드 상의 어느 위치에 존재하는지를 비트 플래그로 압축하여 저장한다. 따라서 특정 자식 노드의 존재 여부를 확인하기 위해 포인터 배열 전체를 검색할 필요 없이, 비트마스크 조회만으로 즉시 알 수 있다.

  - 값 마스크(Value Mask): 이 비트마스크는 리프 노드 내의 복셀들 중 어떤 위치의 복셀이 활성 상태(즉, 배경 값과 다른 값)인지를 나타낸다.

    이 두 비트마스크 덕분에, VDB의 이터레이터(iterator)는 비어있는 공간이나 비활성 복셀을 건너뛰고 오직 유효한 데이터(활성 자식 노드 또는 활성 복셀)에만 직접적으로, 그리고 매우 빠르게 순차적으로 접근할 수 있다.12 이는 사실상 O(1)에 가까운 시간 복잡도로 희소 데이터 집합을 순회할 수 있게 만드는 VDB의 핵심적인 성능 비결이다.


VDB는 데이터를 효율적으로 저장하는 구조뿐만 아니라, 그 데이터를 물리 세계와 연결하는 유연한 시스템을 갖추고 있다. 이는 두 가지 주요 좌표계와 그 사이를 변환하는 메커니즘을 통해 이루어진다.14

- **인덱스 공간 (Index Space)**: VDB의 트리가 직접적으로 다루는 내부 좌표계로, $(i, j, k)$와 같은 3차원 정수 좌표로 구성된다.14 모든 복셀 데이터는 이 인덱스 공간 상의 고유한 좌표에 저장되고 접근된다. 이 공간은 본질적으로 이산적이지만, OpenVDB는 인덱스 좌표 사이의 위치(예: 

  (1.5,2.1,3.7))에 대한 값을 주변 복셀 값으로부터 보간(interpolation)할 수 있도록 연속적인 개념으로 확장하여 사용한다.

- **월드 공간 (World Space)**: 우리가 실질적으로 인식하는 물리적 공간으로, 미터(m)나 센티미터(cm)와 같은 단위를 가지는 (x,y,z) 좌표계다.14 로봇의 위치, 센서 데이터의 측정값 등은 모두 이 월드 공간 상에서 정의된다.

- **변환 (Transform)과 맵 (Map)**: VDB에서 `Grid` 객체는 트리(Tree) 데이터와 `Transform` 객체를 결합한 컨테이너다.14

  `Transform` 객체는 인덱스 공간의 좌표를 월드 공간의 물리적 위치로 매핑하는 역할을 한다. 이 변환은 단순한 스케일(복셀 크기 정의)과 이동(맵의 원점 위치 정의)일 수도 있고, 회전을 포함하는 더 복잡한 아핀 변환(affine transform)일 수도 있다.

이러한 분리된 접근 방식은 엄청난 유연성을 제공한다. 가장 중요한 응용 중 하나는 '인스턴싱(instancing)'이다.14 예를 들어, 하나의 잘 만들어진 나무 모델에 대한 VDB 트리 데이터가 있다고 가정하자. 이 트리 데이터를 여러 개의 다른 `Grid` 객체에서 공유하면서, 각각의 `Grid`에 서로 다른 `Transform`(다른 위치, 다른 크기, 다른 회전)을 적용할 수 있다. 이렇게 하면 단 하나의 데이터 복사본만으로 숲 전체를 표현할 수 있어, 메모리를 극도로 효율적으로 사용하면서 복잡한 장면을 구성하는 것이 가능해진다. 이는 VDB가 단순히 데이터를 저장하는 것을 넘어, 데이터를 재사용하고 변형하여 더 풍부한 정보를 생성하는 강력한 도구임을 보여준다.


VDB의 이론적 우수성은 OpenVDB라는 강력한 오픈소스 라이브러리를 통해 현실 세계의 문제 해결 도구로 구체화되었다. 특히 로보틱스 분야에서 VDB-Mapping은 센서로부터 들어오는 불완전하고 노이즈 섞인 데이터를 통합하여, 일관성 있고 정밀한 3차원 환경 모델을 구축하는 핵심 기술로 자리 잡았다. 이 과정의 중심에는 TSDF(Truncated Signed Distance Field)라는 정교한 데이터 표현 방식과 이를 VDB 그리드에 효율적으로 융합(fusion)하는 메커니즘이 있다. VDB의 등장은 기존의 단순 점유 격자(occupancy grid)를 넘어, 실시간으로 고품질의 표면을 재구성할 수 있는 TSDF와 같은 고급 표현 방식을 자원이 제한된 로봇 플랫폼에서도 실용적으로 만드는 결정적인 계기가 되었다. 이는 VDB가 단순히 기존 맵핑 방식의 저장소를 대체하는 것을 넘어, 맵핑의 질적 수준 자체를 한 단계 끌어올리는 '가능케 하는 기술(enabling technology)'임을 시사한다.


OpenVDB는 VDB 자료 구조를 실제로 사용 가능하게 만든 C++ 참조 구현체이자, 이제는 시각 효과(VFX) 및 시뮬레이션 산업의 사실상 표준(de facto standard)으로 인정받는 라이브러리다.9 아카데미 과학 기술상(Academy Award) 수상 경력이 그 기술적 성취와 영향력을 증명하며, 수많은 장편 영화와 소프트웨어에서 핵심 기술로 활용되고 있다.6

OpenVDB의 가장 큰 특징 중 하나는 C++의 템플릿(template) 기능을 극단적으로 활용한다는 점이다.3 이를 통해 사용자는 VDB 트리의 구조(예: 트리 레벨의 수, 각 레벨 노드의 차원)와 저장할 데이터의 타입(예: `float`, `double`, `int`, `vector`)을 컴파일 시점에 자유롭게 정의할 수 있다. 예를 들어, `openvdb::FloatGrid`는 단정밀도 부동소수점 값을 저장하는 그리드이고, `openvdb::Vec3SGrid`는 3차원 벡터를 저장하는 그리드다.15 이러한 '컴파일 타임 구성(compile-time configuration)'은 런타임에 동적으로 구조를 변경하는 유연성은 일부 포기하는 대신, 컴파일러가 최적화(inlining, template metaprogramming 등)를 최대한 수행할 수 있게 하여 극도로 높은 실행 성능을 보장한다. 이는 VDB의 핵심 설계 철학이기도 하다.3

라이브러리는 데이터 접근을 위해 두 가지 주요 인터페이스를 제공한다 15:

- **Accessor**: 좌표 기반의 임의 접근(random access)을 위한 고성능 인터페이스다. `accessor.setValue(xyz, value)`와 같이 특정 좌표의 복셀 값을 읽고 쓰는 데 사용되며, 내부적으로 캐싱 메커니즘을 활용하여 반복적인 접근 속도를 높인다.
- **Iterator**: 그리드 내의 데이터를 순차적으로 탐색(sequential access)하기 위한 인터페이스다. 특히 `ValueOnCIter`와 같이 '활성' 상태인 복셀들만 효율적으로 순회하는 이터레이터를 제공하여, 희소한 데이터 처리에 매우 유용하다.15

이처럼 OpenVDB는 고성능과 유연성을 겸비한 풍부한 도구 모음을 제공함으로써, 개발자가 VDB 자료 구조의 복잡한 내부 구현을 신경 쓰지 않고도 자신의 응용 분야에 맞게 강력한 볼류메트릭 데이터 처리 파이프라인을 구축할 수 있도록 지원한다.


로보틱스 매핑의 맥락에서, VDB-Mapping은 주로 LiDAR나 깊이 카메라와 같은 센서로부터 얻은 포인트 클라우드(point cloud) 데이터를 VDB 그리드에 통합하여 3차원 지도를 생성하는 과정을 의미한다. 이 과정의 가장 기본적인 메커니즘은 '레이캐스팅(Ray-Casting)'이다.16 `vdb_mapping_ros` 16나  `VDBFusion` 18과 같은 실제 구현체들이 이 방식을 채택하고 있다.

레이캐스팅 기반 업데이트 프로세스는 다음과 같은 단계로 이루어진다:

1. **광선 정의(Ray Definition)**: 각 센서 측정 포인트(pi)에 대해, 센서의 원점(oi)에서 해당 포인트까지 이어지는 가상의 광선(ray)을 정의한다.
2. **광선 투사(Ray Casting)**: 정의된 광선을 3차원 그리드 공간에 투사하여, 이 광선이 어떤 복셀들을 통과하는지 계산한다. OpenVDB는 이 계산을 효율적으로 수행하기 위해 DDA(Digital Differential Analyzer)와 같은 알고리즘을 내장하고 있다.19
3. **복셀 상태 업데이트(Voxel State Update)**: 광선이 통과한 복셀들의 상태를 업데이트한다. 일반적으로 광선의 종점, 즉 센서가 측정한 표면 근처의 복셀들은 '점유(occupied)' 상태로 간주된다. 반면, 센서 원점과 표면 사이의 경로상에 있는 복셀들은 장애물이 없는 '자유(free)' 공간으로 간주된다.16
4. **절단 거리 적용(Applying Truncation Distance)**: 실제로는 표면으로부터 무한히 떨어진 공간까지 모두 '자유'로 업데이트하는 것은 비효율적이며 불필요하다. 따라서 대부분의 TSDF 기반 매핑 시스템은 실제 측정된 표면 주변의 일정 거리, 즉 '절단 거리(truncation distance, $\tau$)' 내에 위치하는 복셀들만 업데이트 대상으로 삼는다.18 예를 들어, 측정된 거리가 $d$라면, $[d - \tau, d + \tau]$ 범위 내의 광선 경로상 복셀들만 계산에 포함시킨다. 이는 연산량을 크게 줄이면서도 표면 주변의 중요한 정보는 유지하는 효과적인 방법이다.19

이러한 레이캐스팅 과정은 수많은 센서 포인트에 대해 반복적으로 수행되며, VDB의 빠른 데이터 삽입 및 접근 성능 덕분에 실시간으로 대량의 데이터를 처리하는 것이 가능하다.


단순히 공간의 점유 여부만 판단하는 것을 넘어, 보다 정밀한 표면 모델을 얻기 위해 VDB-Mapping에서는 TSDF 표현이 널리 사용된다. TSDF는 각 복셀에 가장 가까운 표면까지의 거리를 저장하되, 그 부호(sign)를 통해 복셀이 표면의 안쪽에 있는지 바깥쪽에 있는지를 구분하는 방식이다.8 예를 들어, 표면 바깥은 양수, 안쪽은 음수 값을 가지며, 표면 자체는 0의 값을 갖는다. 'Truncated(절단된)'라는 이름에서 알 수 있듯이, 이 거리 값은 미리 정해진 절단 거리 $\tau$ 내에서만 유효하며, 그 밖의 영역은 $+\tau$ 또는 $-\tau$로 고정된다. 이는 센서 노이즈를 평균화하여 완화하고, 매끄러운 표면을 재구성하는 데 매우 효과적이다.21

새로운 센서 측정값이 들어올 때마다, VDB-Mapping 시스템은 이 정보를 기존의 TSDF 맵에 '융합(fusion)'하여 점진적으로 맵을 갱신한다. 이 과정은 1996년 Curless와 Levoy에 의해 정립된 고전적인 가중 평균(weighted average) 방식을 따른다.19

TSDF 융합은 두 개의 VDB 그리드를 사용하여 이루어진다: 하나는 거리 값(D)을 저장하고, 다른 하나는 각 복셀에 대한 누적 가중치(W)를 저장한다. 시간 t에 새로운 측정값(dt(x))과 그에 대한 가중치(wt(x))가 주어졌을 때, 복셀 x의 새로운 TSDF 값 $D_t(x)$와 누적 가중치 $W_t(x)$는 다음과 같은 수식으로 업데이트된다 19:
$$
D_t(x) = \frac{W_{t-1}(x)D_{t-1}(x) + w_t(x)d_t(x)}{W_{t-1}(x) + w_t(x)}
$$

$$
W_t(x) = W_{t-1}(x) + w_t(x)
$$

여기서 $D_{t-1}(x)$와 $W_{t-1}(x)$는 이전 시간 단계의 값들이다. 이 수식의 의미는, 새로운 측정값을 기존의 누적된 정보에 가중치를 고려하여 평균내는 것이다. 가중치(W)가 높은 복셀은 여러 번 관측되어 신뢰도가 높다는 의미이므로, 새로운 측정값이 전체 평균에 미치는 영향이 상대적으로 작아진다. 반대로 처음 관측되는 복셀은 새로운 측정값이 그대로 반영된다.

초기 TSDF 구현에서는 계산의 편의를 위해 센서 광선 방향으로의 거리인 '투영(projective) 거리'를 사용했다.21 하지만 이는 실제 가장 가까운 표면까지의 유클리드(Euclidean) 거리를 과대평가하는 경향이 있어 맵의 정확도를 저하시키는 원인이 되었다. 최근의 VDB-Mapping 기반 프레임워크들, 예를 들어 Voxfield와 같은 시스템들은 계산 비용이 더 들더라도 기하학적으로 더 정확한 '비투영(non-projective) 거리'를 계산하여 통합함으로써, 재구성된 표면의 품질을 크게 향상시키는 방향으로 발전하고 있다.21 이처럼 VDB의 효율성은 더 정교하고 계산 비용이 높은 알고리즘을 실시간 매핑에 적용할 수 있는 길을 열어주고 있다.


VDB-Mapping의 기술적 우수성을 객관적으로 평가하기 위해서는 기존의 주요 3D 매핑 기법들과의 다각적인 비교 분석이 필수적이다. 로보틱스와 컴퓨터 비전 분야에서는 환경을 표현하기 위해 다양한 자료 구조가 개발되고 활용되어 왔다. 본 장에서는 VDB-Mapping을 대표적인 세 가지 기법-고정 그리드 맵(Grid Map), 옥트리(OctoMap), 그리고 서펠(Surfel) 기반 매핑-과 비교하여, 각 기법의 장단점을 명확히 하고 VDB-Mapping이 제공하는 차별화된 가치를 조명한다. 비교 기준은 메모리 효율성, 연산 속도(데이터 접근 및 삽입), 맵의 정밀도 및 표현력, 그리고 동적인 환경 변화에 대한 대응 능력(동적 토폴로지 지원)을 중심으로 한다.


- vs. 고정 그리드 맵 (Grid Map)

  고정 그리드 맵은 3차원 공간을 균일한 크기의 복셀(voxel)로 나누고, 이를 거대한 3차원 배열로 표현하는 가장 직관적이고 단순한 방식이다.23 이 구조의 최대 장점은 데이터 접근 속도다. 특정 좌표 $(i, j, k)$에 해당하는 데이터는 배열 인덱싱을 통해 O(1)의 시간 복잡도로 즉시 접근할 수 있다.24 그러나 이 방식은 치명적인 단점을 내포하고 있다. 바로 메모리 비효율성이다. 그리드 맵은 실제 데이터의 존재 여부와 무관하게, 매핑하고자 하는 환경의 최대 경계 상자(bounding box) 전체에 해당하는 메모리를 사전에 할당해야 한다.23 따라서 대부분의 공간이 비어있는 희소한 환경(예: 넓은 실내 공간, 실외 도로)에서는 메모리의 대부분이 낭비된다. 또한, 맵의 크기가 한번 정해지면 확장이 매우 어렵고, 만약 로봇이 초기 설정된 경계를 벗어나면 맵 전체를 더 큰 배열로 복사하는 값비싼 연산이 필요하다.23 VDB-Mapping은 이러한 문제를 완벽하게 해결한다. 데이터가 존재하는 곳에만 노드를 동적으로 생성하고 배경 값과 타일을 이용해 빈 공간을 암묵적으로 처리함으로써 희소 환경에서 극도로 높은 메모리 효율을 달성한다. 또한, 사실상 무한한 인덱스 공간을 지원하여 맵의 경계에 대한 걱정 없이 동적으로 확장이 가능하다.2

- vs. 옥트리 (OctoMap)

  OctoMap은 로보틱스 커뮤니티에서 가장 널리 사용되는 확률적 3D 점유 격자 매핑 프레임워크로, 옥트리(Octree) 자료 구조를 기반으로 한다.13 옥트리는 공간을 재귀적으로 8개의 정육면체(octant)로 분할하며, 자식 노드들이 모두 동일한 상태(예: 모두 점유 또는 모두 비점유)일 경우 부모 노드에서 이를 통합하여 표현함으로써 메모리를 절약한다. 이 덕분에 OctoMap은 고정 그리드 맵에 비해 훨씬 뛰어난 메모리 효율성을 보인다. 하지만 옥트리 역시 한계를 가진다. 데이터에 접근하거나 새로운 데이터를 삽입하기 위해서는 루트 노드부터 시작하여 트리 구조를 따라 내려가는 탐색 과정이 필요하며, 이 시간은 트리의 깊이에 비례한다 (로그 시간 복잡도).20 맵의 해상도가 높아지거나 공간이 복잡해져 트리가 깊어지면 이 탐색 비용은 무시할 수 없는 수준이 되며, 특히 LiDAR와 같이 초당 수십만 개의 포인트를 쏟아내는 고밀도 센서를 처리할 때 병목 현상을 유발할 수 있다.13 VDB-Mapping은 B+Tree 기반의 얕고 넓은 구조, 좌표 기반의 빠른 위치 계산, 그리고 내부 캐싱 메커니즘을 통해 평균적으로 O(1)에 가까운 임의 접근 및 삽입 속도를 제공한다.2 실제 성능 평가에서 VDB 기반 매핑 시스템은 OctoMap보다 월등히 빠른 데이터 통합 속도를 보였으며, VDBFusion의 경우 특정 데이터셋에서 Octomap 대비 2~3배 빠른 성능을 기록했다.2

- vs. 서펠 기반 매핑 (Surfel-based Mapping)

  서펠(Surfel, Surface Element) 기반 매핑은 환경을 체적(volume)이 아닌 표면(surface)의 집합으로 모델링하는 접근 방식이다.26 각 서펠은 위치, 반경, 법선 벡터, 색상 등 표면의 지역적 특성을 나타내는 작은 디스크(disk)로 표현된다. 이 방식은 포인트 클라우드 정합(registration)을 통한 SLAM의 위치 추정이나 고품질의 표면 재구성에 강점을 보인다.26 하지만 서펠은 근본적으로 표면 정보만을 담고 있기 때문에, 로봇의 경로 계획에 필수적인 '자유 공간(free space)'이나 아직 탐사되지 않은 '미지 공간(unknown space)'에 대한 정보를 명시적으로 제공하지 않는다. 따라서 자율 항법과 같은 작업을 위해서는 서펠 맵으로부터 추가적인 처리를 통해 체적 정보를 추론해야 하는 번거로움이 있다. 또한, 맵의 충실도가 서펠의 수와 밀도에 크게 의존하므로, 대규모 환경을 상세하게 표현하기 위해서는 많은 수의 서펠이 필요하게 되어 확장성에 한계를 보일 수 있다.27 반면, VDB-Mapping은 본질적으로 체적 표현 방식이므로, 점유 공간, 자유 공간, 미지 공간을 모두 하나의 일관된 프레임워크 내에서 자연스럽게 표현할 수 있다. 이는 TSDF나 점유 확률과 같은 다양한 볼류메트릭 데이터를 저장할 수 있는 유연성을 제공하여, 3D 재구성부터 경로 계획까지 다양한 로보틱스 작업을 지원하는 통합된 환경 모델을 구축하는 데 더 적합하다.


이러한 분석을 종합하면, 각 3D 매핑 기법의 특성과 장단점을 명확하게 파악할 수 있다. 아래 표는 VDB-Mapping과 주요 경쟁 기술들을 핵심적인 성능 지표에 따라 비교하여 정리한 것이다. 이 표는 각 기술이 어떤 응용 분야에 더 적합한지에 대한 직관적인 이해를 돕는다.

| 특징 (Feature)     | VDB-Mapping (OpenVDB)                              | OctoMap                      | 고정 그리드 맵 (Grid Map)          | 서펠 기반 매핑 (Surfel-based)                   |
| ------------------ | -------------------------------------------------- | ---------------------------- | ---------------------------------- | ----------------------------------------------- |
| **핵심 자료 구조** | 계층적 B+Tree 유사 구조 (Hierarchical B+Tree-like) | 옥트리 (Octree)              | 고밀도 3차원 배열 (Dense 3D Array) | 표면 요소 집합 (Collection of Surface Elements) |
| **메모리 효율성**  | 매우 높음 (희소 데이터에 최적)                     | 높음                         | 낮음 (경계 상자 전체 할당)         | 중간 (표면 복잡도에 의존)                       |
| **접근/삽입 속도** | 빠름 (평균 O(1) 임의 접근)                         | 로그 시간 (트리 깊이에 비례) | 매우 빠름 (O(1))                   | 가변적                                          |
| **동적 토폴로지**  | 탁월함 (무한 공간, 동적 확장)                      | 좋음                         | 나쁨 (고정된 경계)                 | 좋음                                            |
| **데이터 표현**    | 체적 (SDF, 점유, 밀도 등)                          | 체적 (주로 확률적 점유)      | 체적 (점유)                        | 표면 (위치, 법선, 색상 등)                      |
| **주요 활용 사례** | VFX, 대규모 로보틱스, 시뮬레이션                   | 로보틱스 경로 계획, 탐사     | 단순 환경, 시뮬레이션              | 표면 재구성, SLAM 정합                          |
| **참고 자료**      | 1                                                  | 13                           | 23                                 | 26                                              |

결론적으로, VDB-Mapping은 희소하고 동적인 대규모 3D 환경을 다루는 데 있어 메모리 효율성과 연산 속도 측면에서 기존 기술들을 능가하는 뚜렷한 장점을 제공한다. 이는 VDB 기술이 단순한 점진적 개선이 아니라, 특정 문제 영역에서 게임의 판도를 바꾸는 혁신적인 솔루션임을 보여준다.


VDB 기술의 진정한 가치는 그것이 단일 분야에 국한되지 않고, 다양한 산업의 난제들을 해결하는 핵심 도구로 자리매김했다는 점에서 드러난다. VDB의 설계 철학인 '희소 데이터의 효율적 처리'는 특정 응용에만 유효한 것이 아니라, 본질적으로 희소한 특성을 갖는 3차원 데이터를 다루는 모든 분야에 적용될 수 있는 보편적인 원리다. 이로 인해 VDB는 그 탄생지인 시각 효과(VFX) 산업의 경계를 넘어 로보틱스, 3D 재구성, 자율 시스템 등 새로운 영역으로 빠르게 확산되었다.

이러한 확산 과정은 VDB가 단순히 기존 작업을 더 빠르게 만드는 '최적화 도구'를 넘어, 이전에는 계산 비용이나 메모리 제약으로 인해 비실용적이거나 불가능하다고 여겨졌던 문제들을 해결 가능한 영역으로 끌어들인 '가능케 하는 기술(enabling technology)'임을 명확히 보여준다. 예를 들어, 실시간으로 초고해상도 유체 시뮬레이션을 인터랙티브하게 조작하거나, 자원이 제한된 소형 로봇에서 대규모 환경의 3D 지도를 CPU만으로 실시간 생성하는 것은 VDB의 등장 이전에는 상상하기 어려웠다. 본 장에서는 VDB-Mapping 기술이 각 분야에서 어떻게 활용되며 새로운 가능성을 열고 있는지 구체적인 사례를 통해 살펴본다.


VDB 기술은 컴퓨터 그래픽스, 특히 영화의 시각 효과(VFX) 분야에서 태동하고 성장했다. 드림웍스 애니메이션은 영화 <장화신은 고양이(Puss in Boots)>에서 하늘을 가득 채운 거대하고 역동적인 구름을 표현하기 위해 기존 방식의 한계를 절감했고, 이를 해결하기 위해 Ken Museth가 VDB를 개발했다.1 VDB는 곧 연기, 불, 안개, 물과 같은 유체 시뮬레이션이나 폭발, 파괴 효과와 같이 희소하면서도 시간에 따라 형태가 급격히 변하는 볼류메트릭 데이터를 다루는 데 있어 업계의 표준으로 자리 잡았다.1

OpenVDB 라이브러리는 VFX 아티스트와 개발자를 위해 강력하고 직관적인 도구 모음을 제공한다 10:

- **필터링 및 노이즈 적용**: 가우시안 블러(Gaussian blur)와 같은 필터를 적용하여 볼륨을 부드럽게 만들거나, 절차적 노이즈(procedural noise)를 추가하여 구름이나 연기에 사실적인 디테일을 더하는 데 사용된다. 이는 드림웍스의 구름 모델링 툴셋의 기반이 되었다.10
- **모폴로지 연산(Morphological Operations)**: 팽창(dilation)이나 침식(erosion)과 같은 모폴로지 연산은 레벨셋(level set) 기반의 액체 표면 추적이나 시간에 따라 변하는 볼륨의 형태를 조작하는 데 필수적이다.10
- **CSG (Constructive Solid Geometry) 연산**: 합집합(union), 교집합(intersection), 차집합(difference)과 같은 불리언 연산을 매우 빠르게 수행할 수 있다. 이를 통해 여러 볼륨을 절차적으로 결합하거나, 특정 영역을 마스킹하여 필터를 적용하는 등 복잡한 형태를 손쉽게 만들어낼 수 있다.
- **변환 및 렌더링 지원**: 스칼라 및 벡터 필드에 대한 수학적 변환(예: 경사도, 라플라시안, 발산, 회전)을 지원하며, 효율적인 레이 트레이싱을 위한 가속 구조를 내장하고 있어 렌더링 시간을 단축시킨다.10

이처럼 VDB는 VFX 파이프라인에서 창의적인 표현의 한계를 확장하고, 아티스트가 이전보다 훨씬 더 크고 복잡한 시뮬레이션을 다룰 수 있게 만들었다.


VFX 분야에서 검증된 VDB의 효율성과 강력함은 자연스럽게 로보틱스 커뮤니티의 주목을 받았다. 로봇이 주변 환경을 인식하고 자율적으로 작업을 수행하기 위해서는 정밀한 3D 환경 모델이 필수적인데, VDB는 바로 이 문제를 해결할 이상적인 솔루션을 제공했다.30

- **점유 격자 맵핑 (Occupancy Mapping)**: 로봇 항법의 가장 기본적인 요구사항은 이동 가능한 공간과 장애물을 구분하는 것이다. VDB-Mapping은 고해상도의 실시간 점유 격자 맵을 생성하는 데 탁월한 성능을 보인다. 특히 최신 3D LiDAR 센서가 생성하는 방대한 양의 포인트 데이터를 실시간으로 처리할 때, 기존의 OctoMap 기반 시스템들이 데이터 삽입 속도의 한계로 인해 일부 데이터를 누락시키는 경우가 발생하는 반면, VDB 기반 시스템은 빠른 삽입 속도를 바탕으로 모든 데이터를 손실 없이 맵에 통합할 수 있다.2 이는 로봇이 더 정확하고 최신 상태의 지도를 기반으로 안전한 결정을 내릴 수 있게 한다.
- **TSDF 통합 및 표면 재구성**: VDB는 단순한 점유 여부를 넘어, TSDF와 같은 더 풍부한 정보를 담는 맵을 실시간으로 구축하는 것을 가능하게 했다. VDBFusion 18, VDBblox 34, Voxfield 21와 같은 프레임워크들은 VDB의 희소 데이터 구조를 기반으로 TSDF를 효율적으로 통합하여, 센서 노이즈가 제거된 고품질의 3D 표면 메시(mesh)를 실시간으로 재구성한다. 특히 VDBFusion은 고가의 GPU 없이 단일 코어 CPU만으로도 64채널 LiDAR 데이터를 초당 20프레임 이상 처리하는 놀라운 성능을 보여주었는데 19, 이는 VDB가 자원이 제한된 모바일 로봇 플랫폼에서도 고품질 3D 재구성을 실현할 수 있음을 입증한 중요한 사례다.


VDB-Mapping 기술은 자율주행차와 드론과 같은 고도로 동적인 자율 시스템의 핵심 역량을 강화하는 데 중요한 역할을 한다.

- **자율주행 (Autonomous Driving)**: 자율주행차의 안전한 운행은 주변 환경에 대한 초정밀 3D 지도(HD Map)와 실시간 장애물 인지에 달려있다.35 VDB는 이러한 요구사항을 충족시키는 데 이상적인 특성을 가진다.
  - **HD Map 생성**: VDB의 사실상 무한한 공간 표현 능력과 메모리 효율성은 도시 전체와 같은 광활한 도로 환경을 차선, 표지판, 연석 등 세밀한 정보까지 포함하여 정밀하게 모델링하는 데 적합하다.36
  - **동적 객체 레이어**: 정적인 HD Map 위에, VDB를 사용하여 실시간으로 감지되는 다른 차량, 보행자 등 동적 객체들의 위치와 이동 경로를 표현하는 동적 레이어를 구축할 수 있다. VDB의 빠른 업데이트 성능은 급변하는 교통 상황에 신속하게 대응하는 데 필수적이다.2
- **드론 항법 (Drone Navigation)**: 드론은 가볍고 배터리 용량이 제한되어 있어 계산 자원이 매우 제약적이다. 또한 GPS 신호가 닿지 않는 실내나 복잡한 구조물 사이를 비행해야 하는 경우가 많다.37 VDB-Mapping은 이러한 드론의 항법 문제를 해결하는 데 효과적으로 사용된다.
  - **실시간 3D 환경 모델링**: 드론에 장착된 깊이 카메라나 소형 LiDAR의 데이터를 VDB 맵으로 실시간 통합하여, 드론이 자신의 위치를 파악하고 충돌을 회피하며 경로를 계획할 수 있는 3D 지도를 생성한다.32
  - **GPU 가속 및 시뮬레이션**: NVIDIA의 NanoVDB는 VDB 자료 구조를 GPU에서 직접 처리할 수 있도록 최적화한 버전이다.10 이를 활용한 NanoMap과 같은 프레임워크는 GPU 가속을 통해 소형 드론과 같은 플랫폼에서도 매우 빠른 속도로 맵을 생성하고, 가상의 환경에서 드론의 비행을 시뮬레이션하는 기능까지 제공한다.39
  - **원격 통신**: `vdb_mapping_ros`와 같은 ROS 패키지는 드론이 현장에서 생성한 VDB 맵 데이터를 압축하여 지상의 원격 제어 스테이션으로 효율적으로 전송하는 기능을 제공한다.16 이를 통해 조종사는 드론의 시야를 넘어선 곳의 상황을 3D 지도로 파악하며 원격으로 임무를 수행할 수 있다.

이처럼 VDB-Mapping은 다양한 응용 분야에서 기존의 기술적 한계를 돌파하고 새로운 가능성을 제시하며 그 영향력을 지속적으로 확장해 나가고 있다.


VDB-Mapping은 희소 볼류메트릭 데이터 처리 분야에서 혁신을 이끌었지만, 기술이 성숙함에 따라 새로운 도전 과제와 한계점들이 드러나고 있다. 동시에, 딥러닝과 같은 파괴적인 기술의 등장은 VDB의 미래를 더욱 흥미로운 방향으로 이끌고 있다. 현재의 VDB는 주로 기하학적 구조를 정교하게 표현하는 데 초점이 맞춰져 있다. 그러나 진정한 지능형 시스템은 단순히 '어디에 무엇이 있는지'를 아는 것을 넘어, '그것이 무엇이며 어떻게 변하는지'를 이해해야 한다.

이러한 맥락에서 VDB의 발전 과정은 컴퓨터 과학의 더 넓은 흐름을 반영한다. 초기의 VDB는 인간의 통찰력으로 정교하게 설계된 순수한 알고리즘적, 명시적(explicit) 자료 구조였다.1 그 다음 단계에서는 이 강력한 도구가 로보틱스와 같은 새로운 분야에 적용되어 기존의 비효율적인 구조를 대체했다.2 그리고 현재, 우리는 세 번째 단계, 즉 VDB 구조 자체에 학습을 내장하는 하이브리드, 암시적(implicit) 표현으로의 전환을 목격하고 있다. 데이터 구조가 더 이상 수동적인 데이터 컨테이너가 아니라, 지능적인 인식 및 추론 파이프라인의 능동적인 구성 요소로 진화하고 있는 것이다. 본 장에서는 이러한 전환 과정 속에서 VDB-Mapping이 직면한 기술적 과제를 분석하고, 딥러닝과의 융합을 통해 열리고 있는 미래 연구 방향과 최신 동향을 심도 있게 탐색한다.


VDB-Mapping이 차세대 3D 인식 기술로 도약하기 위해 해결해야 할 주요 기술적 과제는 다음과 같다.

- **동적 환경 처리 (Handling Dynamic Environments)**: 대부분의 표준 VDB-Mapping 구현은 환경이 정적(static)이라는 암묵적인 가정을 기반으로 한다.41 이로 인해 로봇이나 차량, 보행자처럼 움직이는 객체가 지도상에 잔상(ghosting artifacts)으로 남아, 로봇의 경로 계획을 방해하거나 잘못된 판단을 유발할 수 있다. 이를 해결하기 위해 '스페이스 카빙(space carving)'과 같이, 센서 광선이 통과하는 영역을 비우는 기법이 사용되지만, 이는 계산 비용이 매우 높아 실시간 성능을 저하시키는 단점이 있다.17 따라서 동적 객체를 효율적으로 탐지, 추적하고 지도에서 분리하거나 혹은 동적인 상태 그대로 모델링하는 강건한 방법론이 여전히 중요한 연구 주제로 남아있다.
- **의미론적 정보의 부재 (Lack of Semantic Information)**: 기본적인 VDB 맵은 점유 확률, 밀도, 또는 표면까지의 거리와 같은 순수한 기하학적 정보만을 담고 있다.43 이 맵은 "좌표 (x,y,z)에 장애물이 있다"고 말해줄 수는 있지만, "그 장애물이 '의자'이며, '앉을 수 있는' 기능을 가졌다"는 의미론적(semantic) 정보는 제공하지 못한다. 이러한 의미 정보의 부재는 "부엌으로 가서 음료수를 가져와"와 같은 고수준의 인간-로봇 상호작용이나 복잡한 작업을 수행하는 데 있어 근본적인 한계로 작용한다.44 기하학적 3D 맵에 객체의 종류, 속성, 관계 등을 정의하는 의미론적 레이어를 어떻게 효율적으로 통합하고 추론할 것인가가 핵심 과제다.
- **대규모 확장성과 데이터 관리 (Large-Scale Scalability and Data Management)**: VDB는 뛰어난 메모리 효율성을 자랑하지만, 도시 전체나 국가 단위의 초거대 규모(ultra large-scale) 환경을 단일 VDB 맵으로 관리하는 것은 여전히 현실적인 도전이다.45 데이터의 크기가 일정 수준을 넘어서면 메모리 한계에 부딪히거나, 데이터 로딩 및 저장에 소요되는 시간이 길어져 실시간성을 해칠 수 있다. 따라서 맵 데이터를 여러 서버에 분산하여 저장하고 관리하는 기술, 또는 네트워크를 통해 필요한 부분만 효율적으로 스트리밍하고 업데이트하는 원격 매핑(remote mapping) 프레임워크에 대한 연구가 요구된다.16


이러한 기술적 과제들을 극복하고 VDB-Mapping의 지평을 넓히기 위한 연구가 딥러닝과의 융합을 중심으로 활발하게 진행되고 있다.

- **딥러닝과의 융합 (Fusion with Deep Learning)**: VDB의 미래는 딥러닝과의 깊은 통합에 달려있다. 이는 단순한 결합을 넘어, VDB 구조 자체를 딥러닝 프레임워크의 일부로 만드는 방향으로 나아가고 있다.
  - **fVDB**: VDB 구조 위에서 동작하는 미분 가능한(differentiable) 연산자들을 제공하는 혁신적인 프레임워크다.46 컨볼루션, 풀링, 레이 트레이싱과 같은 핵심 딥러닝 연산들을 VDB의 희소 그리드 상에서 직접 수행할 수 있게 함으로써, 3D 딥러닝 모델이 VDB의 메모리 효율성과 빠른 접근 속도의 이점을 그대로 누리며 학습하고 추론할 수 있게 한다. 이는 고해상도의 3D 데이터를 다루는 딥러닝 응용의 새로운 가능성을 연다.
  - **NeuralVDB**: VDB의 압축률을 한 차원 더 끌어올리는 기술로, VDB 트리의 하위 레벨 위상 정보와 복셀 값들을 명시적으로 저장하는 대신, 이를 표현하도록 학습된 작은 신경망(neural network)으로 대체한다.48 즉, 데이터 자체를 저장하는 것이 아니라 데이터를 생성하는 '함수'를 저장하는 것이다. 이 하이브리드 접근법은 VDB의 계층적 구조가 주는 공간 적응성의 장점은 유지하면서, 신경망의 강력한 표현력을 이용해 기존 OpenVDB 대비 메모리 사용량을 10배에서 최대 100배까지 획기적으로 줄일 수 있다. 이는 대규모 시뮬레이션이나 AI 기반 의료 영상 분석과 같이 극도의 메모리 효율성이 요구되는 분야에 큰 파급 효과를 가질 것이다.
- **의미론적 매핑 (Semantic Mapping)**: 기하학적 맵에 의미를 부여하는 연구는 VDB와 딥러닝의 결합이 가장 활발하게 이루어지는 분야 중 하나다. 딥러닝 기반의 객체 탐지(object detection) 및 의미론적 분할(semantic segmentation) 모델이 추출한 정보를 VDB의 기하학적 뼈대 위에 융합하는 방식이다.
  - NVIDIA의 `nvblox_torch`와 같은 최신 라이브러리는 VDB 기반의 3D 재구성 파이프라인에 비전 파운데이션 모델(Vision Foundation Model)이 추출한 딥 피처(deep feature)를 직접 융합하는 기능을 제공한다.49 이를 통해 생성된 맵은 기하학적 형태와 의미론적 내용을 동시에 담는 풍부한 표현이 된다.
  - RayFronts와 같은 연구는 한 걸음 더 나아가, 관측된 영역에는 의미론적 정보를 담은 복셀 맵을 구축하고, 아직 관측되지 않은 미지 영역의 경계(frontier)에서는 가능한 의미 정보를 예측하는 '의미론적 광선(semantic ray)'을 투사한다.50 이는 로봇이 단순히 보이는 것을 넘어, 보이지 않는 공간에 무엇이 있을지 추론하며 효율적으로 탐사할 수 있게 하는 진보된 형태의 의미론적 매핑이다.51
- **4D 시공간 매핑 (4D Spatio-temporal Mapping)**: 동적 환경 문제를 정면으로 다루기 위해, VDB-Mapping은 3차원 공간에 '시간' 축을 더한 4D 시공간으로 확장되고 있다. 이 접근법은 동적 객체를 노이즈로 간주하여 제거하는 대신, 시간에 따른 장면의 변화 자체를 모델링하는 것을 목표로 한다.
  - 최근 CVPR과 같은 최고 수준의 학회에서 발표되는 연구들은 신경망 표현(Neural Representation), 특히 NeRF(Neural Radiance Fields)나 신경 SDF(Neural SDF)를 활용하여 시간에 따라 변화하는 연속적인 TSDF를 학습한다.41 이 4D 표현으로부터 시간에 따라 변하지 않는 정적 배경(static background) 성분과 시간에 따라 움직이는 동적 성분을 분리해낼 수 있다. 이를 통해 동적 객체들로 인해 오염되지 않은, 매우 정확하고 상세한 정적 환경 맵을 추출하는 것이 가능해진다. 이는 자율주행을 위한 고정밀 지도 제작이나 디지털 트윈 구축에 있어 핵심적인 기술이 될 것이다.

이러한 최신 동향들은 VDB-Mapping이 기하학적 모델링 도구를 넘어, 주변 세계를 이해하고 예측하는 '지능형 공간 모델(Intelligent Spatial Model)'로 진화하고 있음을 명백히 보여준다. VDB의 효율적인 계층 구조는 이러한 고차원적이고 복잡한 정보를 담아내는 이상적인 뼈대(scaffold) 역할을 하고 있으며, 그 미래는 무한한 가능성을 향해 열려 있다.


본 보고서는 VDB-Mapping 기술에 대한 다각적이고 심층적인 고찰을 통해 그 핵심 원리, 응용 분야, 그리고 미래 발전 가능성을 조명했다. VDB-Mapping은 단순히 기존 3D 매핑 기술의 점진적 개선이 아니라, 희소하고 동적인 볼류메트릭 데이터를 처리하는 방식에 대한 근본적인 패러다임 전환을 이끈 혁신적인 기술 프레임워크다.


VDB-Mapping의 핵심 가치는 B+Tree의 효율적인 인덱싱과 3차원 그리드의 공간 표현력을 결합한 독창적인 자료 구조에 있다. 이 구조는 '데이터 중심적 인코딩'이라는 철학을 바탕으로, 방대한 빈 공간을 암묵적으로 처리하고 실제 데이터가 존재하는 영역에만 자원을 집중함으로써 메모리 효율성과 연산 속도를 극대화했다. 이러한 기술적 성취는 VDB-Mapping을 단순한 도구가 아닌, 여러 분야의 기술적 한계를 돌파하게 만든 '가능케 하는 기술(enabling technology)'의 반열에 올려놓았다.

컴퓨터 그래픽스 분야에서 VDB는 이전에는 불가능했던 규모와 복잡성의 시각 효과를 실현하며 창의적 표현의 지평을 넓혔다. 로보틱스 분야에서는 자원이 제한된 플랫폼에서도 고해상도의 3D 환경 모델을 실시간으로 구축하는 것을 현실화함으로써, 자율 항법과 3D 재구성 기술의 발전을 가속화했다. 이처럼 VDB-Mapping은 각 분야가 직면한 핵심적인 병목 현상을 해결하며 그 기술적 의의를 입증해왔다.


현재 VDB-Mapping은 기하학적 3D 맵을 구축하는 데 있어 매우 강력하고 성숙한 솔루션으로 확고히 자리매김했다. 그러나 기술의 발전은 새로운 도전을 요구하고 있다. 정적인 기하학 구조를 넘어, 동적으로 변화하는 객체를 이해하고, 공간에 존재하는 사물의 의미를 파악하는 능력은 차세대 지능형 시스템의 필수 요건이다.

VDB-Mapping의 미래는 딥러닝과의 깊은 융합에 달려 있다. fVDB와 NeuralVDB 같은 최신 연구 동향은 VDB가 단순한 데이터 저장소를 넘어, 딥러닝 모델의 학습과 추론 과정에 직접 참여하는 능동적인 구성 요소로 진화하고 있음을 보여준다. fVDB는 물리 기반 시뮬레이션과 딥러닝을 결합한 새로운 형태의 응용을 창출할 잠재력을 지니고 있으며, NeuralVDB는 메모리 제약이라는 오랜 난제를 해결하여 초거대 규모 데이터 처리를 가능하게 할 것이다.

따라서 향후 VDB-Mapping 연구는 다음의 방향으로 집중될 것을 제언한다. 연구 커뮤니티는 VDB의 검증된 효율성을 기반으로, 미분 가능하고(differentiable), 의미론적(semantic) 정보를 풍부하게 담으며, 시간의 흐름에 따른 동적 변화(4D)까지 포괄하는 통합된 시공간 표현(spatio-temporal representation)을 구축하는 데 노력을 경주해야 한다. 이러한 노력을 통해 VDB-Mapping은 주변 환경의 형태를 복제하는 것을 넘어, 그 본질을 이해하고 미래를 예측하는 진정한 '지능형 공간 모델(Intelligent Spatial Model)'로 거듭날 것이며, 이는 자율 로봇, 디지털 트윈, 가상현실 등 미래 기술의 발전에 핵심적인 기여를 할 것으로 확신한다.


1. VDB: High-resolution sparse volumes with dynamic ... - Ken Museth, accessed August 5, 2025, https://www.museth.org/Ken/Publications_files/Museth_TOG13.pdf
2. VDB-Mapping: A High Resolution and Real-Time Capable 3D Mapping Framework for Versatile Mobile Robots - ResearchGate, accessed August 5, 2025, https://www.researchgate.net/publication/355133596_VDB-Mapping_A_High_Resolution_and_Real-Time_Capable_3D_Mapping_Framework_for_Versatile_Mobile_Robots
3. Frequently Asked Questions - OpenVDB, accessed August 5, 2025, https://www.openvdb.org/documentation/doxygen/faq.html
4. Virtual database (VDB) management, accessed August 5, 2025, https://cd.delphix.com/docs/latest/virtual-database-vdb-management
5. Virtual Databases / GitBook, accessed August 5, 2025, https://teiid.github.io/teiid-documents/12.3.x/content/reference/vdb_guide.html
6. OpenVDB - Ken Museth, accessed August 5, 2025, https://ken.museth.org/OpenVDB.html
7. OpenVDB: An Open Source Data Structure and Toolkit for High-Resolution Volumes, accessed August 5, 2025, https://www.youtube.com/watch?v=7hUH92xwODg
8. Hierarchical, Dense and Dynamic 3D Reconstruction Based on VDB Data Structure for Robotic Manipulation Tasks - Frontiers, accessed August 5, 2025, https://www.frontiersin.org/journals/robotics-and-ai/articles/10.3389/frobt.2020.600387/full
9. Insight: VDB, a deep dive - JangaFX, accessed August 5, 2025, https://jangafx.com/insights/vdb-a-deep-dive
10. About OpenVDB, accessed August 5, 2025, https://www.openvdb.org/about/
11. Hierarchical, Dense and Dynamic 3D Reconstruction Based on VDB Data Structure for Robotic Manipulation Tasks - PMC, accessed August 5, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC7935507/
12. What is OpenVDB - cebas VISUAL TECHNOLOGY, accessed August 5, 2025, https://cebas.com/manual/finalRender/Reference_Guide/OpenVDB_NanoVDB/What_is_OpenVDB.htm
13. NanoMap: A GPU-Accelerated OpenVDB-Based Mapping and Simulation Package for Robotic Agents - MDPI, accessed August 5, 2025, https://www.mdpi.com/2072-4292/14/21/5463
14. OpenVDB Overview, accessed August 5, 2025, https://www.openvdb.org/documentation/doxygen/overview.html
15. OpenVDB Cookbook, accessed August 5, 2025, https://www.openvdb.org/documentation/doxygen/codeExamples.html
16. fzi-forschungszentrum-informatik/vdb_mapping_ros: ROS1 Wrapper für vdb_mapping - GitHub, accessed August 5, 2025, https://github.com/fzi-forschungszentrum-informatik/vdb_mapping_ros
17. Interactive Distance Field Mapping and Planning to Enable Human-Robot Collaboration, accessed August 5, 2025, https://arxiv.org/html/2403.09988v1
18. VDBFusion: Flexible and Efficient TSDF Integration of Range Sensor Data - ResearchGate, accessed August 5, 2025, https://www.researchgate.net/publication/358523933_VDBFusion_Flexible_and_Efficient_TSDF_Integration_of_Range_Sensor_Data
19. VDBFusion: Flexible and Efficient TSDF Integration of Range Sensor Data - MDPI, accessed August 5, 2025, https://www.mdpi.com/1424-8220/22/3/1296
20. Benchmarking Occupancy Mapping libraries - Nicolò Valigi, accessed August 5, 2025, https://nicolovaligi.com/articles/occupancy-mapping-benchmarks/
21. Voxfield: Non-Projective Signed Distance Fields for Online Planning and 3D Reconstruction, accessed August 5, 2025, https://www.ipb.uni-bonn.de/wp-content/papercite-data/pdf/pan2022iros.pdf
22. (PDF) Hierarchical, Dense and Dynamic 3D Reconstruction Based ..., accessed August 5, 2025, https://www.researchgate.net/publication/349471767_Hierarchical_Dense_and_Dynamic_3D_Reconstruction_Based_on_VDB_Data_Structure_for_Robotic_Manipulation_Tasks
23. OctoMap: An Efficient Probabilistic 3D Mapping Framework Based on Octrees - Washington, accessed August 5, 2025, https://courses.cs.washington.edu/courses/cse571/16au/slides/hornung13auro.pdf
24. MULTI-RESOLUTION FOR EFFICIENT, SCALABLE AND, accessed August 5, 2025, https://www.research-collection.ethz.ch/bitstream/handle/20.500.11850/679133/DoctoralThesisVictorReijgwart.pdf?sequence=1&isAllowed=y
25. Remote VDB-Mapping: A Level-Based Data Reduction Framework for Distributed Mapping | Request PDF - ResearchGate, accessed August 5, 2025, https://www.researchgate.net/publication/362929869_Remote_VDB-Mapping_A_Level-Based_Data_Reduction_Framework_for_Distributed_Mapping
26. Efficient Surfel-Based SLAM using 3D Laser Range Data in Urban Environments - Robotics, accessed August 5, 2025, https://www.roboticsproceedings.org/rss14/p16.pdf
27. CURL-SLAM: Continuous and Compact LiDAR Mapping - arXiv, accessed August 5, 2025, https://arxiv.org/html/2506.21077v1
28. Qualitative comparison between Octomap and VDBFusion. While Octomap... | Download Scientific Diagram - ResearchGate, accessed August 5, 2025, https://www.researchgate.net/figure/Qualitative-comparison-between-Octomap-and-VDBFusion-While-Octomap-clearly-models-the_fig10_358523933
29. VDB Analysis geometry node - SideFX, accessed August 5, 2025, https://www.sidefx.com/docs/houdini/nodes/sop/vdbanalysis.html
30. VoxelCache: Accelerating Online Mapping in Robotics and 3D Reconstruction Tasks, accessed August 5, 2025, https://www.researchgate.net/publication/367481067_VoxelCache_Accelerating_Online_Mapping_in_Robotics_and_3D_Reconstruction_Tasks
31. VoxelCache: Accelerating Online Mapping in Robotics and 3D Reconstruction Tasks - arXiv, accessed August 5, 2025, https://arxiv.org/pdf/2210.08729
32. (PDF) Efficient Global Occupancy Mapping for Mobile Robots using OpenVDB, accessed August 5, 2025, https://www.researchgate.net/publication/365209668_Efficient_Global_Occupancy_Mapping_for_Mobile_Robots_using_OpenVDB
33. Map of the FZI House of Living Labs created by the VDBMapping approach.... | Download Scientific Diagram - ResearchGate, accessed August 5, 2025, https://www.researchgate.net/figure/Map-of-the-FZI-House-of-Living-Labs-created-by-the-VDBMapping-approach-The-Data-was_fig1_355133596
34. VDBblox: Accurate and Efficient Distance Fields for Path Planning and Mesh Reconstruction, accessed August 5, 2025, https://www.semanticscholar.org/paper/e9523886dc7b0f62929b3ac90143e15227a9838e
35. Autonomous Cars | Solution | SINTRONES Technology Corp., accessed August 5, 2025, https://www.sintrones.com/solution/autonomous-cars/
36. Leveraging Mapping Data for Precise Localization in Autonomous Driving - Voxelmaps, accessed August 5, 2025, https://www.voxelmaps.com/news/leveraging-mapping-data-for-precise-localization-in-autonomous-driving
37. Navigation of Unmanned Aerial Vehicle Using Computer Vision in Raytheon Drone Competition - MavMatrix, accessed August 5, 2025, https://mavmatrix.uta.edu/cgi/viewcontent.cgi?article=1029&context=honors_spring2025
38. Enhancing Drone Navigation and Control: Gesture-Based Piloting, Obstacle Avoidance, and 3D Trajectory Mapping - MDPI, accessed August 5, 2025, https://www.mdpi.com/2076-3417/15/13/7340
39. NanoMap: A GPU-Accelerated OpenVDB-Based Mapping and Simulation Package for Robotic Agents - ResearchGate, accessed August 5, 2025, https://www.researchgate.net/publication/364974244_NanoMap_A_GPU-Accelerated_OpenVDB-Based_Mapping_and_Simulation_Package_for_Robotic_Agents
40. Wrote an occupany mapping library for use with robotics - Google Groups, accessed August 5, 2025, https://groups.google.com/g/openvdb-forum/c/J-kkRWz1V1M
41. 3D LiDAR Mapping in Dynamic Environments Using a 4D Implicit Neural Representation, accessed August 5, 2025, https://www.ipb.uni-bonn.de/wp-content/papercite-data/pdf/zhong2024cvpr.pdf
42. VDBFusion: Flexible and Efficient TSDF Integration of Range Sensor Data - PMC, accessed August 5, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC8838740/
43. VDB-based Spatially Grounded Semantics for Interactive Robots | Request PDF, accessed August 5, 2025, https://www.researchgate.net/publication/391693488_VDB-based_Spatially_Grounded_Semantics_for_Interactive_Robots
44. ViMantic, a distributed robotic architecture for semantic mapping in indoor environments, accessed August 5, 2025, https://www.researchgate.net/publication/354119331_ViMantic_a_distributed_robotic_architecture_for_semantic_mapping_in_indoor_environments
45. Voxel-Based Navigation: A Systematic Review of Techniques, Applications, and Challenges, accessed August 5, 2025, https://www.mdpi.com/2220-9964/13/12/461
46. fVDB: A Deep-Learning Framework for Sparse, Large-Scale, and High-Performance Spatial Intelligence - Research at NVIDIA, accessed August 5, 2025, https://research.nvidia.com/labs/prl/williams2024fVDB/fVDB.pdf
47. fVDB: A Deep-Learning Framework for Sparse, Large-Scale, and High-Performance Spatial Intelligence - arXiv, accessed August 5, 2025, https://arxiv.org/html/2407.01781v1
48. Optimizing Large-Scale Sparse Volumetric Data with NVIDIA NeuralVDB Early Access, accessed August 5, 2025, https://developer.nvidia.com/blog/optimizing-large-scale-sparse-volumetric-data-with-nvidia-neuralvdb-early-access/
49. R²D²: Building AI-based 3D Robot Perception and Mapping with NVIDIA Research, accessed August 5, 2025, https://developer.nvidia.com/blog/r2d2-building-ai-based-3d-robot-perception-and-mapping-with-nvidia-research/
50. [Literature Review] RayFronts: Open-Set Semantic Ray Frontiers for Online Scene Understanding and Exploration - Moonlight, accessed August 5, 2025, https://www.themoonlight.io/en/review/rayfronts-open-set-semantic-ray-frontiers-for-online-scene-understanding-and-exploration
51. [2109.01474] Real-Time Volumetric-Semantic Exploration and Mapping: An Uncertainty-Aware Approach - arXiv, accessed August 5, 2025, https://arxiv.org/abs/2109.01474
52. 3D LiDAR Mapping in Dynamic Environments using a 4D Implicit Neural Representation, accessed August 5, 2025, https://cvpr.thecvf.com/virtual/2024/poster/31240

