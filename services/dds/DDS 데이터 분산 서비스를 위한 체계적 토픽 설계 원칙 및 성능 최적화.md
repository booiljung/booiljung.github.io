# DDS 데이터 분산 서비스를 위한 체계적 토픽 설계 원칙 및 성능 최적화
[데이터 분산 서비스 DDS](./index.md)


Data Distribution Service (DDS)는 Object Management Group (OMG)에 의해 표준화된 통신 미들웨어로, 분산 시스템 환경에서 데이터 중심(Data-Centric)의 발행-구독(Publish-Subscribe) 통신 패러다임을 제공하는 것을 핵심 목표로 한다.1 이는 기존의 메시지 중심(Message-Centric) 또는 객체 요청 브로커(Object Request Broker) 방식과 근본적인 철학적 차이를 보인다. DDS는 중앙 집중식 서버나 브로커에 의존하지 않는 탈중앙화 아키텍처를 통해, 시스템 참여자(Participant)들이 동적으로 네트워크에 참여하고 이탈하는 환경에서 높은 수준의 실시간성, 신뢰성, 그리고 확장성을 보장하도록 설계되었다.2


DDS 통신의 근간은 글로벌 데이터 공간(Global Data Space, GDS)이라는 개념에 있다.6 GDS는 물리적으로 분산된 노드들이 논리적으로 공유하는 가상의 데이터 공간이다. 애플리케이션은 이 공간에 특정 종류의 데이터를 발행(Publish)하거나, 관심 있는 데이터를 구독(Subscribe)함으로써 상호작용한다. 이 모델에서 애플리케이션은 데이터의 최종 수신자가 누구인지, 어디에 있는지 알 필요가 없다. DDS 미들웨어는 데이터의 형식, 전달 방식, 수신자 결정 등 통신과 관련된 모든 복잡한 메커니즘을 자율적으로 처리한다.7 이러한 특성은 시스템 컴포넌트 간의 결합도를 극적으로 낮추어 유연하고 확장 가능한 아키텍처 구축을 가능하게 한다. 전통적인 메시지 중심 미들웨어가 '메시지'를 어떻게 전달할지에 초점을 맞춘다면, DDS는 '데이터' 자체의 상태를 어떻게 공유하고 동기화할지에 집중한다.4


GDS 내에서 데이터 흐름을 식별하고 정의하는 가장 기본적인 단위를 '토픽(Topic)'이라 칭한다.7 토픽은 단순한 메시지 채널의 이름이 아니라, 데이터 자체를 명세하는 세 가지 핵심 요소의 집합체로 정의된다.9

1. **토픽 이름 (Topic Name):** 도메인 내에서 데이터를 고유하게 식별하는 문자열이다. 예를 들어, `VehiclePosition` 또는 `SensorData_Radar`와 같이 데이터의 의미를 명확하게 나타내는 이름을 사용한다.10
2. **데이터 타입 (Topic Type):** 해당 토픽을 통해 교환될 데이터의 구조를 엄격하게 정의한다. 이 데이터 타입은 일반적으로 OMG의 인터페이스 정의 언어(Interface Definition Language, IDL)를 사용하여 플랫폼과 프로그래밍 언어에 독립적인 방식으로 명세된다.9
3. **서비스 품질 (QoS) 정책 (Quality of Service Policies):** 데이터의 비기능적 속성을 제어하는 규칙의 집합이다. 데이터 전달의 신뢰성, 데이터의 보존 기간(지속성), 데이터의 이력 관리, 우선순위 등 20가지가 넘는 세분화된 정책을 통해 애플리케이션의 요구사항에 맞춰 통신 동작을 정교하게 튜닝할 수 있다.6


데이터 중심 패러다임에서 토픽은 단순한 데이터 전달 통로 이상의 의미를 가진다. DDS 미들웨어는 토픽을 통해 '어떤 데이터인지(데이터 타입)', '어떤 형식인지(IDL)', 그리고 '어떻게 안전하고 신뢰성 있게 보낼 것인지(QoS)'에 대한 모든 정보를 관리한다.14 이로 인해 애플리케이션 개발자는 저수준의 네트워크 프로그래밍에서 해방되어 데이터 자체의 비즈니스 로직에만 집중할 수 있다.

이러한 관점에서 DDS 토픽을 올바르게 이해하는 것은 매우 중요하다. 토픽을 전통적인 메시징 시스템의 '메시지 큐'나 '채널'로 간주하는 것은 DDS의 잠재력을 제한하는 접근법이다. 대신, 토픽은 **'분산 데이터베이스의 테이블'**에 더 가깝다고 이해해야 한다. 이 비유는 DDS 토픽 설계의 본질을 꿰뚫는다. 데이터베이스 테이블이 명확한 스키마(Schema)를 갖는 것처럼, DDS 토픽은 IDL로 정의된 엄격한 데이터 타입을 가진다. 테이블이 기본 키(Primary Key)를 통해 여러 행(Row)을 관리하는 것처럼, DDS 토픽은 '키(Key)'를 사용하여 다수의 독립적인 데이터 '인스턴스(Instance)'를 식별하고 관리할 수 있다.15 또한, 데이터베이스가 데이터의 영속성(Persistence)을 보장하듯이, DDS는 `DURABILITY` QoS 정책을 통해 발행자(DataWriter)가 사라진 후에도 GDS에 데이터를 보존하여 나중에 참여하는 구독자(DataReader)에게 전달할 수 있다.17 따라서 DDS 토픽을 설계하는 행위는 단순히 통신 채널을 여는 것이 아니라, 분산 시스템 전체가 공유하고 동기화할 데이터 모델, 즉 '분산 데이터베이스의 스키마'를 설계하는 고도의 아키텍처 설계 활동이라 할 수 있다. 이 패러다임의 전환을 이해하는 것이 효과적인 DDS 시스템을 구축하는 첫걸음이다.


DDS의 데이터 중심 철학을 구현하는 기술적 근간은 바로 인터페이스 정의 언어(IDL)이다. IDL을 통해 데이터 타입을 명세하는 과정은 분산 시스템의 상호운용성, 확장성, 그리고 유지보수성을 결정짓는 핵심적인 단계이다.


IDL은 특정 프로그래밍 언어나 운영체제 플랫폼에 종속되지 않는 중립적인 방식으로 데이터 구조와 인터페이스를 정의하기 위한 명세 언어이다.18 DDS 환경에서 IDL의 역할은 절대적이다. C++로 작성된 발행자와 Java로 작성된 구독자가 원활하게 데이터를 교환할 수 있는 이유는, 양측 모두 IDL로 정의된 공통의 데이터 타입을 기준으로 DDS 미들웨어가 데이터의 직렬화(Serialization)와 역직렬화(Deserialization)를 자동으로 처리하기 때문이다.7 이는 ROS(Robot Operating System)와 같은 프레임워크에서도 마찬가지로, ROS1의 `.msg` 파일 형식은 ROS2에서 DDS 통신을 위해 내부적으로 `.idl` 형식으로 변환되어 사용된다.10

견고하고 잘 구조화된 IDL 설계는 시스템의 장기적인 생명주기에 큰 영향을 미친다. 시스템이 발전함에 따라 새로운 센서가 추가되거나 데이터 요구사항이 변경될 때, 잘 설계된 IDL은 기존 시스템의 수정을 최소화하면서 새로운 기능을 유연하게 추가할 수 있는 기반을 제공한다. 이는 전체 시스템의 재설계 없이도 확장이 용이한 아키텍처를 가능하게 한다.20


효과적인 IDL 설계를 위해서는 DDS가 지원하는 데이터 타입의 특성을 이해하고, 데이터의 논리적 구조를 명확하게 표현해야 한다.

- **기본 타입 (Primitive Types):** `short`, `long`, `double`, `boolean`, `char`, `octet`, `string` 등과 같은 기본 타입은 플랫폼 간 비트 수준의 호환성을 보장하므로 데이터 교환의 기본 단위로 사용된다. 설계 시에는 데이터의 범위와 정밀도를 고려하여 가장 적절한 타입을 선택해야 한다.21
- **구조체 (Struct):** 관련된 데이터 필드들을 논리적으로 그룹화하는 가장 일반적인 방법이다. 예를 들어, 3차원 위치를 표현하기 위해 `x`, `y`, `z` 필드를 갖는 `Point` 구조체를 정의할 수 있다. 구조체는 데이터의 응집도를 높이고 가독성을 향상시키는 방향으로 설계되어야 한다.9
- **유니언 (Union):** 여러 타입 중 단 하나의 값만 가질 수 있는 데이터를 모델링할 때 유용하다. 예를 들어, 센서 측정값이 정수일 수도 있고 부동소수점 수일 수도 있는 경우, 유니언을 사용할 수 있다. 이때, 현재 어떤 타입의 데이터가 저장되어 있는지를 명시하는 '식별자(discriminator)' 필드를 함께 사용하는 것이 필수적이다. 이는 수신 측에서 데이터를 안전하게 해석할 수 있도록 보장한다.21
- **시퀀스 (Sequence) 및 배열 (Array):** 가변 길이(시퀀스) 또는 고정 길이(배열)의 데이터 컬렉션을 표현한다. 시퀀스는 동적 메모리 할당을 통해 유연성을 제공하지만, 실시간성과 결정성이 중요한 임베디드 시스템에서는 예측 불가능한 메모리 할당 및 지연을 유발할 수 있다. 따라서 시퀀스를 사용하더라도 최대 길이를 명시적으로 제한(`sequence<long, 10>`)하여 리소스 사용량을 예측 가능하게 만드는 것이 권장된다.21


복잡한 시스템에서는 데이터 간의 관계를 모델링하는 것이 중요하다. IDL에서는 주로 두 가지 방식을 사용한다.

- **내장 방식 (Embedding):** 하나의 구조체 안에 다른 구조체를 멤버 필드로 직접 포함시키는 방식이다. 예를 들어, `Car` 구조체 안에 `Engine` 구조체를 포함할 수 있다. 이 방식은 데이터들이 항상 함께 생성되고 소비될 때 직관적이고 효율적이다. 하지만, `Engine` 데이터만 업데이트되었음에도 불구하고 전체 `Car` 데이터를 전송해야 하므로 데이터 중복과 대역폭 낭비를 유발할 수 있다.
- **참조 방식 (Foreign Key Pattern):** 데이터베이스 설계의 '외래 키' 개념을 차용한 방식이다. 각 데이터 모델을 별도의 토픽으로 정의하고, 한 토픽의 인스턴스가 다른 토픽 인스턴스를 식별할 수 있는 키(Key) 값을 필드로 가지도록 설계한다.25 예를 들어, `Carpool` 토픽과 `Car` 토픽을 별도로 정의하고, `Car` 구조체 안에 `long carpoolId;` 필드를 두어 자신이 속한 `Carpool` 인스턴스를 참조하게 할 수 있다. 이 방식은 데이터 중복을 제거하고 각 데이터의 독립적인 업데이트를 가능하게 하여 네트워크 효율성을 극대화한다. `Carpool`에 새로운 `Car`가 추가될 때, 전체 `Carpool` 데이터를 재전송할 필요 없이 새로운 `Car` 인스턴스 데이터만 발행하면 된다. 단점은 구독하는 애플리케이션 측에서 `carpoolId`를 기반으로 분리된 데이터들을 다시 조합(join)하는 로직을 구현해야 한다는 점이다.25


DDS X-Types 표준은 IDL에서 구조체 간의 상속(Inheritance)을 지원하여 데이터 모델의 재사용성과 확장성을 높인다.18 IDL에서 `struct Derived : Base {... };`와 같은 구문을 사용하여 기본 구조체를 확장하는 새로운 구조체를 정의할 수 있다.25 이를 통해 `Shape`라는 기본 타입을 정의하고, 이를 상속받아 `Circle`, `Rectangle` 등 구체적인 타입을 만들 수 있다.

그러나 상속 모델을 사용할 때는 중요한 주의사항이 있다. 구독자(DataReader)는 자신이 명시적으로 구독한 타입에 정의된 필드에만 접근할 수 있다. 예를 들어, 구독자가 `Shape` 타입으로 토픽을 구독하고 있을 때, 발행자(DataWriter)가 더 많은 필드를 가진 `Circle` 타입의 데이터를 발행하더라도, 해당 구독자는 `Shape`에 정의된 공통 필드만 읽을 수 있고 `Circle`에만 특화된 필드(예: `radius`)는 접근할 수 없다.26 따라서 상속은 데이터의 다형성(polymorphism)을 표현하는 강력한 도구이지만, 수신 측의 데이터 해석 능력을 반드시 고려하여 신중하게 적용해야 한다.

IDL을 통한 데이터 타입 설계는 단순히 통신 메시지의 페이로드(payload)를 정의하는 기술적인 작업을 넘어선다. 이는 분산 시스템에 참여하는 모든 노드가 공유하고 합의하는 **'공유 온톨로지(Shared Ontology)'**, 즉 개념 체계를 구축하는 과정이다. `struct Car`라는 정의는 단순히 바이트의 레이아웃을 명시하는 것이 아니라, 이 시스템 내에 '자동차'라는 개념적 실체가 존재하며, 그 실체는 '색상', 'ID'와 같은 속성으로 구성된다는 모든 참여자 간의 의미론적 계약(semantic contract)을 수립하는 것이다. 특히 참조(Foreign Key) 패턴을 사용할 때, `carpoolId` 필드는 단순한 정수 값이 아니라 '소속'이라는 관계를 명시하는 의미론적 연결고리 역할을 한다. 따라서 IDL을 설계할 때는 필드와 구조체의 이름, 그리고 이들 간의 관계가 시스템의 도메인 지식을 얼마나 명확하고 정확하게 반영하는지가 매우 중요하다. 잘 설계된 IDL은 그 자체로 시스템의 아키텍처를 설명하는 핵심적인 문서 역할을 하며, 시스템의 유지보수성과 신규 개발자의 이해도를 극적으로 향상시킨다.


| IDL Type        | C++11 Type       | 설명                                                         |
| --------------- | ---------------- | ------------------------------------------------------------ |
| `octet`         | `uint8_t`        | 8비트 부호 없는 정수. 바이트 데이터를 표현하는 데 사용된다.  |
| `short`         | `int16_t`        | 16비트 부호 있는 정수.                                       |
| `unsigned long` | `uint32_t`       | 32비트 부호 없는 정수.                                       |
| `long long`     | `int64_t`        | 64비트 부호 있는 정수.                                       |
| `double`        | `double`         | 64비트 부동소수점 수.                                        |
| `boolean`       | `bool`           | 논리값 (true/false).                                         |
| `string<N>`     | `std::string`    | 최대 길이 N을 갖는 문자열. N을 지정하여 리소스 제한을 명시할 수 있다. |
| `sequence<T>`   | `std::vector<T>` | 타입 T의 가변 길이 시퀀스.                                   |


DDS 토픽 설계에서 가장 중요하고 강력한 개념 중 하나는 '키(Key)'이다. 키는 토픽 내에서 개별 데이터 스트림을 식별하고 관리하는 핵심 메커니즘으로, 키의 유무는 토픽의 동작 방식을 근본적으로 변화시키며 시스템 전체 아키텍처에 지대한 영향을 미친다.


키는 토픽의 데이터 타입(IDL `struct`) 내에 존재하는 하나 이상의 필드로 구성된다. 이 필드들의 값 조합은 해당 토픽 내에서 데이터의 '인스턴스(instance)'를 고유하게 식별하는 역할을 한다.15 데이터베이스의 기본 키(Primary Key)와 매우 유사한 개념이다. IDL 명세에서는 `#pragma keylist` 지시문이나 필드 옆에 `@key` 어노테이션을 붙여 특정 필드가 키의 일부임을 명시한다.9

예를 들어, 항공 관제 시스템에서 항공기들의 위치 정보를 교환하기 위해 `FlightPlan`이라는 토픽을 정의한다고 가정하자. 이 토픽의 데이터 타입이 `flightID`, `position`, `altitude`, `speed` 필드를 포함하고 있을 때, `flightID`를 키로 지정하면 DDS 미들웨어는 각 `flightID` 값에 해당하는 항공기 정보를 별개의 독립적인 인스턴스로 취급하고 관리한다. 즉, `flightID`가 'KE001'인 데이터와 'OZ202'인 데이터는 같은 `FlightPlan` 토픽에 속하지만, 서로 다른 인스턴스로서 독립적인 라이프사이클을 가진다.


토픽은 키의 존재 유무에 따라 'Keyed Topic'과 'Keyless Topic'으로 나뉘며, 이 둘은 동작 방식과 적합한 사용 사례에서 명확한 차이를 보인다.

- **Keyless Topic (키 없는 토픽):**
  - 키가 정의되지 않은 토픽은 전체가 단 하나의 인스턴스만을 가진다.15 이는 싱글턴(Singleton) 객체와 유사한 동작 방식이다.
  - 새로운 데이터 샘플이 발행될 때마다 이 단일 인스턴스의 값은 최신 값으로 갱신된다. 이전 값은 (QoS 설정에 따라 다르지만) 일반적으로 덮어쓰인다.
  - **적용 사례:** 시스템 전체에 적용되는 공통 설정값, 특정 지역의 현재 기상 조건, 시스템의 전반적인 상태(예: '정상', '경고', '오류')와 같이 단일 정보 소스로 표현될 수 있는 데이터에 적합하다.
- **Keyed Topic (키 있는 토픽):**
  - 키가 정의된 토픽은 각 고유 키 값마다 별개의 인스턴스를 생성하고 관리한다.15
  - DDS 미들웨어는 각 인스턴스의 라이프사이클(생성, 데이터 업데이트, 소멸 등)을 개별적으로 추적한다.
  - 이러한 특성 덕분에, 물리적으로는 하나의 토픽이지만 논리적으로는 수많은 독립적인 데이터 객체들의 상태를 효율적으로 관리할 수 있다. 예를 들어, 수백만 대의 IoT 센서 데이터를 단일 `SensorReading` 토픽으로 관리하면서 각 센서의 `sensorID`를 키로 사용할 수 있다. 이는 수백만 개의 개별 토픽을 생성하는 것보다 훨씬 효율적이다.
  - **장점:** 수많은 데이터 스트림을 단일 토픽으로 통합함으로써, 시스템의 디스커버리(Discovery) 과정에서 발생하는 네트워크 및 CPU 오버헤드를 크게 줄일 수 있다. 이는 대규모 시스템의 확장성과 성능에 결정적인 이점을 제공한다.29


올바른 키 필드를 선정하는 것은 Keyed Topic 설계의 핵심이다.

- **선정 기준:** 키로 사용될 필드는 해당 데이터 스트림을 시스템의 전체 생명주기 동안 고유하게 식별할 수 있는 안정적인 값이어야 한다. 데이터베이스의 기본 키 선정 원칙과 마찬가지로, 값이 자주 변경되거나 중복될 가능성이 있는 필드는 키로 부적합하다.30
- **제약 조건:** DDS 표준에 따르면, 키 필드는 일반적으로 기본 타입(`long`, `double` 등), 열거형(enum), 또는 문자열(string)이어야 한다. 배열, 시퀀스, 또는 다른 구조체를 포함하는 복합 타입은 키 필드로 직접 사용될 수 없다.15

키(Key)의 도입은 DDS를 단순한 발행-구독 메시징 시스템에서 **'분산 객체 상태 동기화 프레임워크'**로 격상시키는 결정적인 역할을 한다. 키가 없는 토픽은 데이터를 불특정 다수에게 방송(broadcast)하는 채널에 가깝다. 그러나 키를 도입하는 순간, 토픽은 '객체의 논리적 집합'이라는 개념으로 변모한다.15

`VehicleState` 토픽은 더 이상 임의의 차량 상태 정보가 흐르는 스트림이 아니라, '시스템에 존재하는 모든 차량 객체들의 최신 상태를 담고 있는 분산 캐시' 또는 '가상 데이터베이스 테이블'이 된다. 이 관점에서 DataWriter가 특정 `vehicleID`를 키로 하여 데이터를 발행하는 행위는 단순히 메시지를 전송하는 것이 아니라, '해당 ID를 가진 분산 객체의 속성을 원격으로 업데이트'하는 행위로 해석될 수 있다. 마찬가지로 DataReader가 특정 키 값의 데이터를 읽는 것은 '특정 객체의 현재 상태를 조회'하거나 '해당 객체의 상태 변경 이벤트를 구독'하는 행위가 된다. 이러한 패러다임의 전환은 시스템 설계에 지대한 영향을 미친다. 개발자는 더 이상 메시지 포맷, 전송 순서, 재전송 로직과 같은 저수준의 통신 문제에 대해 고민하는 대신, 분산된 여러 노드에 걸쳐 '어떤 객체의 상태를 어떻게 일관성 있게 유지할 것인가'라는 고수준의 데이터 모델링 문제에 집중할 수 있게 된다. 이는 복잡한 분산 상태 관리 로직을 애플리케이션에서 분리하여 DDS 미들웨어에 위임함으로써, 애플리케이션 로직을 극적으로 단순화하고 시스템 전체의 안정성과 예측 가능성을 높이는 결과를 가져온다.


| 특성                | Keyless Topic                 | Keyed Topic                                                  |
| ------------------- | ----------------------------- | ------------------------------------------------------------ |
| **인스턴스 수**     | 단일 인스턴스 (Singleton)     | 고유 키 값당 하나의 인스턴스 (Multiple Instances)            |
| **데이터 모델**     | 단일 값 또는 상태             | 객체들의 집합, 데이터베이스 테이블                           |
| **리소스 사용량**   | 낮음 (단일 큐 관리)           | 높음 (인스턴스별 큐 및 메타데이터 관리)                      |
| **QoS 적용 단위**   | 토픽 레벨                     | 인스턴스 레벨 (예: `HISTORY`, `OWNERSHIP`)                   |
| **주요 용도**       | 시스템 전역 설정, 환경 데이터 | 다수 객체의 상태 추적, 센서 데이터 스트림 식별               |
| **네트워크 효율성** | -                             | 다수 데이터 스트림을 단일 토픽으로 통합하여 디스커버리 부하 감소 |


DDS의 가장 강력하고 차별화된 기능은 바로 서비스 품질(QoS) 정책이다. 20가지가 넘는 표준화된 QoS 정책은 데이터의 '무엇(What)'을 정의하는 IDL을 넘어, 데이터를 '어떻게(How)' 전달하고 관리할지를 매우 정교하게 제어할 수 있게 해준다. 효과적인 토픽 설계는 데이터 구조 정의와 함께, 해당 데이터의 특성에 맞는 최적의 QoS 정책을 조합하는 과정이라 할 수 있다.


DDS의 QoS는 발행자(DataWriter)와 구독자(DataReader) 간의 '요청-제공(Request-Offered)' 모델을 기반으로 동작한다.

- DataWriter는 자신이 제공할 수 있는 QoS 수준을 '제공(Offer)'한다.
- DataReader는 자신이 필요로 하는 QoS 수준을 '요청(Request)'한다.
- 두 엔티티 간의 통신 채널(Association)이 형성되려면, DataWriter가 제공하는 QoS가 DataReader가 요청하는 QoS와 호환되어야 한다. 호환성 규칙은 각 QoS 정책마다 정의되어 있다. 예를 들어, 신뢰성을 의미하는 `RELIABILITY` QoS 정책의 경우, DataReader가 `RELIABLE` (신뢰성 있는 전송)을 요청했다면 DataWriter는 반드시 `RELIABLE`을 제공해야 한다. 만약 DataWriter가 `BEST_EFFORT` (최선 노력 전송)만 제공한다면, 이 둘은 호환되지 않으므로 매칭은 실패하고 데이터 교환은 이루어지지 않는다.31

토픽 자체에도 QoS 정책을 설정할 수 있는데, 이는 해당 토픽에 대한 DataWriter나 DataReader를 생성할 때 기본값으로 사용되어 일관된 정책 적용을 돕는다.32


수많은 QoS 정책 중에서도 `RELIABILITY`, `HISTORY`, `DURABILITY` 세 가지는 데이터 전달의 보장 수준과 시간적 결합(temporal coupling)을 결정하는 가장 중요한 요소이며, 서로 밀접하게 연관되어 동작한다.34

- **신뢰성 (RELIABILITY):** 데이터 샘플이 반드시 전달되어야 하는지를 결정한다.
  - `BEST_EFFORT`: UDP와 유사하게 데이터 전송을 시도하지만, 네트워크 상황에 따라 유실될 수 있음을 가정한다. 재전송 메커니즘이 없으므로 오버헤드가 적고 지연 시간이 짧다. 고주파로 스트리밍되는 센서 데이터나 비디오 프레임과 같이 최신 데이터가 과거 데이터보다 훨씬 중요한 경우에 적합하다.10
  - `RELIABLE`: TCP와 유사하게 데이터의 수신을 보장한다. 수신 측에서 데이터 유실을 감지하면 송신 측에 재전송을 요청하는 메커니즘이 동작한다. 상태 변경, 제어 명령, 이벤트 알림 등 단 하나의 샘플도 놓쳐서는 안 되는 중요한 데이터에 필수적으로 사용된다.10
- **이력 관리 (HISTORY):** DataWriter가 구독자에게 전달하기 위해 얼마나 많은 과거 데이터 샘플을 보관할지를 결정한다. 이는 주로 `RELIABLE` 통신 시 재전송을 위해 사용된다.
  - `KEEP_LAST(depth)`: 가장 최근의 `depth`개 샘플만 보관한다. `depth`의 기본값은 1로, 이 경우 오직 가장 최신의 값만 유지된다. 이는 데이터의 현재 '상태'가 중요하고 과거 이력은 중요하지 않은 경우에 매우 효율적이다.33
  - `KEEP_ALL`: `RESOURCE_LIMITS` QoS 정책에 의해 설정된 리소스 한도에 도달할 때까지 생성된 모든 샘플을 보관한다. `RELIABLE` QoS와 함께 사용하면 모든 데이터의 순차적이고 완전한 전달을 보장한다. 하지만, 네트워크 문제 등으로 느린 구독자가 발생할 경우, 해당 구독자가 모든 데이터를 수신할 때까지 DataWriter의 송신 큐가 가득 차서 새로운 데이터 발행이 블로킹(blocking)될 수 있다는 점에 유의해야 한다.31
- **지속성 (DURABILITY):** DataWriter가 발행한 데이터를 얼마나 오래 보존할지를 결정한다. 이 정책은 구독자가 데이터를 구독하기 시작하기 '전에' 발행된 데이터, 즉 '과거 데이터'를 수신할 수 있는지 여부를 제어한다.
  - `VOLATILE`: 데이터는 DataWriter가 활성화되어 있는 동안에만 유효하며, 메모리에만 존재한다. 늦게 참여하는(late-joining) 구독자는 자신이 구독을 시작한 시점 이후에 발행된 데이터만 수신할 수 있다.17
  - `TRANSIENT_LOCAL`: DataWriter가 존재하는 동안에는 발행된 데이터를 자신의 메모리 공간에 보관한다. 늦게 참여한 구독자가 나타나면, DataWriter는 자신이 보관하고 있는 `HISTORY` 정책에 따른 과거 데이터를 전달해준다. 이는 새로운 노드가 시스템에 참여했을 때 즉시 현재 상태를 동기화해야 하는 경우에 매우 유용하다.17
  - `TRANSIENT` / `PERSISTENT`: DDS와 함께 동작하는 별도의 영속성 서비스(Persistence Service)를 통해 데이터를 비휘발성 메모리(RAM)나 디스크와 같은 영구 저장소에 보관한다. 이를 통해 DataWriter 애플리케이션이 완전히 종료된 후에도 데이터가 유지되며, 나중에 시스템이 재시작되거나 새로운 구독자가 참여했을 때 과거 데이터를 수신할 수 있다.17

시스템 설계 시 모든 데이터를 무조건 `RELIABLE`, `KEEP_ALL`, `PERSISTENT`로 설정하는 것은 심각한 안티-패턴이다. 이는 시스템에 불필요한 부하를 가중시키고, 복잡성을 증가시키며, 오히려 실시간성을 저해할 수 있다. 핵심은 토픽으로 전달될 데이터의 본질적인 특성, 즉 **'시간적 가치(Temporal Value)'와 '상태 일관성 요구사항'**을 깊이 분석하고, 그에 맞는 QoS 정책을 전략적으로 조합하는 것이다.

예를 들어, 초당 60회 업데이트되는 자율주행 차량의 라이다(LiDAR) 포인트 클라우드 데이터는 시간적 가치가 극히 짧다. 수 밀리초 전의 데이터는 거의 가치가 없으며, 한두 프레임을 놓치더라도 다음 프레임을 최대한 빨리 받는 것이 훨씬 중요하다. 이러한 데이터에는 `BEST_EFFORT`, `KEEP_LAST(depth=1)`, `VOLATILE` QoS 조합이 최적이다. 반대로, 은행 시스템의 '계좌 이체' 이벤트 데이터는 시간적 가치가 영구적이며, 단 하나의 유실이나 순서 뒤바뀜도 허용되지 않는다. 이 데이터는 `RELIABLE`, `KEEP_ALL`, `PERSISTENT` 조합으로 처리하여 절대적인 무결성을 보장해야 한다.

또 다른 예로, '차량의 현재 위치' 데이터는 최신 상태를 신뢰성 있게 아는 것이 중요하지만, 중간의 몇몇 업데이트가 유실되더라도 치명적이지는 않다. 또한, 새로운 관제 시스템이 네트워크에 접속했을 때 즉시 차량의 현재 위치를 알아야 한다. 이 경우, `RELIABLE`, `KEEP_LAST(depth=1)`, `TRANSIENT_LOCAL` 조합이 이상적이다. `RELIABLE`은 데이터 전달을 보장하고, `KEEP_LAST(1)`은 항상 최신 위치만 유지하여 리소스를 효율적으로 사용하며, `TRANSIENT_LOCAL`은 늦게 참여한 노드도 즉시 최신 상태를 동기화할 수 있게 해준다.

결론적으로, 토픽 설계는 IDL로 데이터 구조를 정의하는 것에서 그치지 않는다. 해당 토픽이 전달할 데이터의 비즈니스적 중요도, 시간적 민감도, 상태 유지의 필요성을 종합적으로 분석하여 그에 맞는 QoS '페르소나(Persona)'를 부여하는 과정이 반드시 수반되어야 한다. 이는 시스템의 요구사항인 성능과 안정성을 동시에 달성하는 가장 중요한 설계 활동이다.


| 시나리오            | `RELIABILITY` | `HISTORY`              | `DURABILITY`      | 동작 방식 및 주요 용도                                       |
| ------------------- | ------------- | ---------------------- | ----------------- | ------------------------------------------------------------ |
| **실시간 스트리밍** | `BEST_EFFORT` | `KEEP_LAST`, `depth=1` | `VOLATILE`        | 최신 데이터만 빠르게 전달. 데이터 유실 허용. (예: 비디오 스트림, 고주파 센서 데이터) |
| **최신 상태 공유**  | `RELIABLE`    | `KEEP_LAST`, `depth=1` | `TRANSIENT_LOCAL` | 최신 상태를 신뢰성 있게 전파. 늦게 참여한 노드도 즉시 최신 상태 획득. (예: 객체 위치, 시스템 상태) |
| **이벤트 알림**     | `RELIABLE`    | `KEEP_ALL`             | `VOLATILE`        | 모든 이벤트를 순서대로, 유실 없이 전달. DataWriter 활성 기간 동안만 유효. (예: 사용자 입력, 경보) |
| **영구적 로그**     | `RELIABLE`    | `KEEP_ALL`             | `PERSISTENT`      | 모든 데이터를 영구적으로 기록. 시스템 재시작 후에도 데이터 접근 가능. (예: 금융 거래 기록, 중요 시스템 로그) |


이론적 원칙을 바탕으로, 실제 DDS 기반 시스템을 설계할 때 자주 등장하는 효과적인 토픽 설계 패턴과, 반대로 시스템의 성능 저하와 복잡성 증가를 유발하는 대표적인 안티-패턴을 이해하는 것은 매우 중요하다.


- **상태 전파 (State Propagation) 패턴:** 이 패턴은 분산 시스템 내에 존재하는 다수 객체들의 상태를 지속적으로 동기화하는 데 사용된다. Keyed Topic을 활용하여 각 객체를 고유한 인스턴스로 모델링한다. 예를 들어, 다수의 로봇 상태를 관리하는 `RobotStatus` 토픽을 만들고 `robot_id`를 키로 사용한다. 각 로봇은 자신의 `robot_id`로 주기적으로 상태 정보를 발행한다. QoS 정책으로는 일반적으로 `RELIABILITY`를 `RELIABLE`로, `HISTORY`를 `KEEP_LAST(depth=1)`로, `DURABILITY`를 `TRANSIENT_LOCAL`로 설정한다. 이를 통해 모든 관제 시스템은 항상 각 로봇의 최신 상태를 신뢰성 있게 유지할 수 있으며, 새로 참여하는 노드도 즉시 모든 로봇의 현재 상태를 동기화할 수 있다.
- **이벤트 알림 (Event Notification) 패턴:** 이 패턴은 시스템에서 발생하는 불연속적인 사건(event)을 알리는 데 사용된다. 상태 전파가 '현재 상태가 무엇인가'에 초점을 맞춘다면, 이벤트 알림은 '무슨 일이 일어났는가'에 집중한다. 예를 들어, `SystemAlert` 토픽은 시스템의 오류나 경고 발생을 알리는 데 사용될 수 있다. 이러한 이벤트는 유실되어서는 안 되므로 `RELIABILITY`는 `RELIABLE`로 설정한다. 또한, 발생한 모든 이벤트를 순서대로 처리해야 할 수 있으므로 `HISTORY`는 `KEEP_ALL`로 설정하는 경우가 많다.
- **요청-응답 (Request-Reply / RPC) 패턴:** DDS는 본질적으로 비동기적인 발행-구독 모델이지만, 이를 응용하여 동기적인 원격 프로시저 호출(RPC)과 유사한 통신 패턴을 구현할 수 있다. 이를 위해서는 두 개의 토픽, 즉 요청 토픽과 응답 토픽이 필요하다.37
  1. 클라이언트는 고유한 `request_id`를 포함한 요청 데이터를 `ServiceName_Request` 토픽으로 발행한다.
  2. 동시에 클라이언트는 `ServiceName_Reply` 토픽을 구독하여 응답을 기다린다.
  3. 서버는 `ServiceName_Request` 토픽을 구독하고 있다가 요청을 수신한다.
  4. 요청을 처리한 후, 서버는 클라이언트가 보낸 `request_id`를 그대로 포함하여 `ServiceName_Reply` 토픽으로 결과 데이터를 발행한다.
  5. 클라이언트는 `ServiceName_Reply` 토픽에서 수신한 데이터의 `request_id`를 확인하여 자신의 요청에 대한 응답임을 식별한다.


- **만능 토픽 (The God Topic) 안티-패턴:**
  - **현상:** 시스템에서 교환되는 모든 종류의 데이터를, 서로 논리적 연관성이 없음에도 불구하고, 하나의 거대한 IDL 구조체로 묶어 단일 토픽으로 발행하는 설계 방식이다.38
  - **문제점:**
    1. **네트워크 대역폭 낭비:** 거대 구조체의 일부 필드만 변경되어도 전체 구조체를 전송해야 한다. 또한, 데이터의 일부만 필요한 구독자조차 불필요한 전체 데이터를 수신해야 하므로 심각한 대역폭 낭비를 유발한다.
    2. **강한 결합도 (Tight Coupling):** 데이터 구조의 사소한 변경이 해당 토픽과 관련된 모든 발행자 및 구독자 애플리케이션의 수정을 요구하게 되어 시스템의 유지보수성과 유연성을 크게 저해한다.
    3. **비효율적인 QoS 적용:** 데이터의 종류별로 상이한 QoS 요구사항(예: 일부는 신뢰성이 중요하고, 일부는 실시간성이 중요함)을 개별적으로 적용할 수 없다. 모든 데이터에 일괄적으로 가장 엄격한 QoS를 적용하게 되어 시스템 리소스를 낭비하게 된다.
  - **회피 전략:** 데이터를 논리적 응집도, 변경 주기, 그리고 소비 패턴을 기준으로 분리한다. 함께 생성되고, 함께 변경되며, 함께 소비되는 데이터들은 하나의 토픽으로 묶는 것이 합리적이다.38
- **과도한 토픽 분할 (Topic Proliferation) 안티-패턴:**
  - **현상:** 만능 토픽과 정반대로, 데이터 구조의 모든 개별 필드나 아주 작은 데이터 그룹을 각각 별개의 토픽으로 나누어 설계하는 방식이다.40
  - **문제점:**
    1. **디스커버리 부하 폭증:** DDS 참여자는 동일 도메인 내의 다른 모든 참여자, 그리고 그들이 생성한 모든 토픽, DataWriter, DataReader를 발견하고 관련 메타데이터를 교환하는 '디스커버리' 과정을 거친다. 토픽의 수가 수백, 수천 개로 늘어나면 이 과정에서 발생하는 네트워크 트래픽과 CPU 부하가 기하급수적으로 증가하여 시스템 시작 시간이 매우 길어지고 전체적인 확장성을 심각하게 저해한다.41
    2. **데이터 일관성 및 원자성 문제:** 여러 토픽에 걸쳐 분산된 관련 데이터들의 상태를 원자적(atomic)으로 업데이트하는 것이 불가능해진다. 이로 인해 구독자는 일시적으로 일관성이 깨진 데이터를 수신할 위험이 있다.
  - **회피 전략:** 논리적으로 강하게 연관된 데이터는 반드시 하나의 구조체로 묶어 단일 토픽으로 관리해야 한다. 특히, 다수의 유사한 데이터 스트림(예: 여러 센서의 측정값)을 관리해야 할 경우, 수십 개의 토픽을 만드는 대신 Keyed Topic을 활용하여 단일 토픽 내에서 여러 인스턴스로 관리하는 것을 최우선으로 고려해야 한다.29
- **키(Key)의 오용 안티-패턴:**
  - **현상:** Keyed Topic의 개념을 이해하지 못하거나 간과하여, 명백히 다수의 인스턴스로 모델링해야 하는 데이터를 Keyless Topic으로만 설계하는 경우. 예를 들어, 100대의 차량 상태를 추적하기 위해 `VehicleState_1`, `VehicleState_2`,..., `VehicleState_100`과 같이 100개의 Keyless Topic을 생성하는 것이다.
  - **문제점:** 이는 위에서 언급한 '과도한 토픽 분할' 안티-패턴과 정확히 동일한 문제를 야기한다. 디스커버리 부하, 리소스 낭비, 관리의 복잡성 등 모든 단점을 그대로 가지게 된다.
  - **회피 전략:** 토픽을 설계하기 전에, 해당 데이터가 '개별 인스턴스'의 개념을 가질 수 있는지 반드시 검토해야 한다. 만약 그렇다면, Keyed Topic을 사용하는 것이 DDS의 인스턴스 관리, 인스턴스별 QoS 적용 등 강력한 기능을 최대한 활용하고 시스템의 확장성을 확보하는 올바른 길이다.29

최적의 토픽 세분성(Granularity)을 찾는 것은 '만능 토픽'과 '과도한 토픽 분할'이라는 양 극단 사이에서 균형점을 찾는 과정이다. 이 균형점을 찾는 가장 중요한 기준은 데이터의 구조적 유사성이 아니라, **'데이터의 생명주기 및 소비 패턴'**이다. 즉, 데이터들이 함께 생성되고, 비슷한 주기로 변경되며, 동일한 구독자 그룹에 의해 함께 소비되는 경향이 있는지를 분석해야 한다. 예를 들어, 차량의 위치(`position`), 속도(`velocity`), 가속도(`acceleration`)는 모두 동일한 물리적 상태에서 파생되며, 관제 시스템과 같은 구독자는 이 정보들을 항상 함께 필요로 한다. 따라서 이들을 `VehicleKinematics`라는 단일 토픽으로 묶는 것은 데이터의 생명주기와 소비 패턴 관점에서 매우 합리적이다. 반면, 차량의 '엔진 오일 온도'와 '인포테인먼트 시스템의 현재 재생 목록'은 생성 및 변경 주기가 완전히 다르고, 이들을 소비하는 주체(엔진 제어 유닛 vs. 사용자 인터페이스) 또한 명확히 구분된다. 이들을 같은 토픽으로 묶는 것은 논리적 응집도를 해치고 불필요한 데이터 결합을 유발하는 '만능 토픽' 안티-패턴에 해당한다. 결국, 효과적인 토픽 설계는 데이터베이스 설계에서의 정규화(Normalization) 과정과 유사한 깊이 있는 분석을 요구한다. 데이터 필드 간의 함수적 종속성(functional dependency)과 생명주기를 분석하여 응집도는 높이고 결합도는 낮추는 방향으로 토픽의 경계를 결정해야 한다.


토픽 설계는 단순히 시스템의 논리적 데이터 구조를 결정하는 것을 넘어, 지연 시간(Latency), 처리량(Throughput), 확장성(Scalability)과 같은 시스템의 동적 성능 특성에 직접적이고 지대한 영향을 미친다. 따라서 시스템 아키텍트는 설계 단계에서부터 이러한 성능적 함의를 반드시 고려해야 한다.


DDS 시스템에서 새로운 참여자가 네트워크에 참여하면, 동일 도메인 내의 다른 모든 참여자 및 그들이 생성한 엔드포인트(DataWriter/DataReader)와 토픽 정보를 교환하는 디스커버리 과정을 거친다. 시스템 내 토픽의 수가 증가하면, 이 과정에서 교환되어야 할 메타데이터 패킷의 양이 크게 증가한다.41

특히 대규모 시스템에서는 수백 개의 노드가 각각 수십 개의 토픽을 생성하는 시나리오가 발생할 수 있다. 이 경우, 디스커버리 트래픽은 네트워크 대역폭을 상당 부분 점유하고 각 노드의 CPU에 상당한 부하를 주어, 시스템 전체의 초기화 시간을 지연시키고 새로운 노드의 참여를 방해하는 요인이 될 수 있다.42 일부 벤치마크 결과에 따르면, 다른 조건이 동일할 때 토픽 수가 증가함에 따라 메시지 지연 시간이 선형적으로 증가하는 경향이 관찰되기도 했다.43 이는 '과도한 토픽 분할' 안티-패턴이 왜 성능에 치명적인지를 명확히 보여준다. 따라서 시스템의 확장성을 확보하기 위해서는 Keyed Topic을 적극적으로 활용하여 불필요한 토픽 생성을 최소화하는 전략이 필수적이다.


- **데이터 크기 (Payload Size):** 데이터 샘플의 크기는 지연 시간과 처리량에 직접적인 영향을 미친다. 데이터 크기가 커질수록 IDL 구조체를 바이트 스트림으로 변환(직렬화)하고 다시 복원(역직렬화)하는 데 더 많은 CPU 시간이 소요된다. 또한, 네트워크를 통해 전송하는 데 걸리는 시간도 자연스럽게 증가한다. 특히, 데이터 크기가 네트워크의 MTU(Maximum Transmission Unit)를 초과하여 UDP 단편화(fragmentation)가 발생하는 지점(일반적으로 약 64KB)부터는 지연 시간이 급격히 증가하는 경향을 보인다.44 처리량 역시 데이터 복사 및 전송에 따르는 오버헤드로 인해 데이터 크기가 매우 커지면 한계에 도달하게 된다.45
- **구독자 수 (Number of Subscribers):** 단일 발행자에 대한 구독자의 수가 증가할수록 시스템의 부하가 증가하여 성능에 영향을 미칠 수 있다. 특히 `RELIABILITY` QoS를 `RELIABLE`로 설정한 경우, 발행자는 모든 구독자로부터 데이터 수신 확인(ACK) 메시지를 받아야 하고, 유실된 구독자에게는 데이터를 재전송해야 한다. 구독자 수가 많아지면 이러한 확인 및 재전송 관리 부담이 발행자에 집중되어 전체적인 지연 시간이 증가하고 처리량이 감소할 수 있다.43


DDS는 기본적으로 UDP/IP 위에서 동작하며, 유니캐스트(Unicast)와 멀티캐스트(Multicast) 통신 방식을 모두 지원한다.

- **멀티캐스트 (Multicast):** 발행자가 데이터를 단 한 번만 네트워크에 전송하면, 해당 멀티캐스트 그룹에 가입한 모든 구독자가 데이터를 수신할 수 있는 방식이다. 이는 동일한 데이터를 다수의 구독자에게 전송해야 할 때 네트워크 자원을 매우 효율적으로 사용할 수 있게 해준다.8 대부분의 DDS 시스템에서 기본 통신 방식으로 사용되며, 일대다(one-to-many) 데이터 전파에 이상적이다.47
- **유니캐스트 (Unicast):** 발행자가 각 구독자에게 개별적으로 데이터를 전송하는 일대일(one-to-one) 통신 방식이다. 구독자 수가 많아지면 동일한 데이터를 여러 번 전송해야 하므로 네트워크 부하가 증가한다.

일반적으로는 멀티캐스트가 더 효율적이지만, 모든 환경에서 최적의 성능을 보장하는 것은 아니다. Wi-Fi와 같이 멀티캐스트 지원이 불안정하거나, 복잡한 라우팅 정책이 적용된 기업 네트워크 환경에서는 오히려 유니캐스트가 더 안정적이거나 빠른 성능을 보일 수 있다.47 따라서 토픽 데이터의 소비 패턴(소수의 특정 구독자만 소비하는가, 아니면 다수의 불특정 구독자가 소비하는가)과 실제 배포될 네트워크 환경의 특성을 종합적으로 고려하여 토픽별로 통신 방식을 적절히 설정하는 것이 성능 최적화에 중요하다.

결론적으로, 토픽 설계는 시스템의 **'정적 아키텍처'뿐만 아니라 '동적 성능 특성'을 결정하는 핵심 제어 변수**이다. 지금까지의 논의가 주로 논리적 데이터 모델링에 초점을 맞추었다면, 실제 성능 측정 데이터는 이러한 논리적 설계 결정이 시스템의 물리적 성능에 직접적이고 측정 가능한 영향을 미친다는 것을 명백히 보여준다.43 예를 들어, '과도한 토픽 분할'은 단순히 관리의 불편함을 넘어, 디스커버리 시간을 증가시켜 42 시스템의 반응성을 저하시키는 실제적인 성능 병목 현상이다. 반대로, 관련 없는 데이터를 모두 묶은 '만능 토픽'은 단일 메시지의 지연 시간을 증가시키고 44 네트워크 처리량을 불필요하게 소모한다.45

따라서 시스템 아키텍트는 토픽을 설계할 때 두 가지 역할을 동시에 수행해야 한다. 하나는 '데이터 모델러'로서 "이 데이터 구조는 도메인 관점에서 논리적으로 타당한가?"를 질문해야 하고, 다른 하나는 '성능 엔지니어'로서 "이 구조가 초당 수천 개의 메시지와 수십 개의 구독자가 존재하는 운영 환경에서 어떤 지연 시간과 처리량 곡선을 그릴 것인가?"를 예측하고 검증해야 한다. 이는 토픽 설계가 개발 초기 단계의 일회성 활동으로 끝나는 것이 아니라, 시스템 통합 및 부하 테스트 단계에서 성능 프로파일링을 통해 지속적으로 검증되고 개선되어야 하는 반복적인 프로세스임을 의미한다.


| 설계 변수           | 값이 증가할 때 예상되는 영향                                 | 주요 고려사항 및 관련 QoS                                    |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **토픽 수**         | **지연 시간:** 증가 (디스커버리 부하)  **처리량:** 감소 가능 | Keyed Topic으로 토픽 수를 줄이는 전략 고려. Static Discovery 사용 검토. |
| **데이터 크기**     | **지연 시간:** 증가 (직렬화/전송 시간)  **처리량:** 특정 임계점 이후 감소 | 데이터 모델 최적화, 불필요한 필드 제거. UDP 단편화 크기 고려. |
| **구독자 수**       | **지연 시간:** 증가 (특히 `RELIABLE` 모드)  **처리량:** 감소 | 멀티캐스트 활용. `RELIABILITY` QoS의 `max_blocking_time` 설정. |
| **`HISTORY` depth** | **메모리 사용량:** 증가  **지연 시간:** `RELIABLE`+`KEEP_ALL`에서 증가 가능 | `RESOURCE_LIMITS` QoS와 함께 설정하여 시스템 자원 고갈 방지. |


DDS 기반의 고성능 실시간 분산 시스템을 성공적으로 구축하기 위한 핵심은 데이터 중심 패러다임에 대한 깊은 이해를 바탕으로 토픽을 체계적으로 설계하는 데 있다. 토픽 설계는 단순히 데이터 교환을 위한 채널을 정의하는 것을 넘어, 시스템 전체의 데이터 아키텍처, 성능, 확장성, 그리고 유지보수성을 결정짓는 근본적인 활동이다. 본 보고서에서 심층적으로 분석한 내용을 바탕으로, 최적의 DDS 시스템을 위한 종합적인 토픽 설계 원칙을 다음과 같이 요약할 수 있다.


1. **데이터 중심적으로 사고하라:** 토픽을 일시적인 메시지가 오가는 통로가 아닌, 상태를 가진 '분산 데이터 객체' 또는 '가상 데이터베이스 테이블'로 인식해야 한다. 시스템의 공유 데이터 모델을 설계한다는 관점에서 접근하여, 데이터의 상태와 라이프사이클 관리를 미들웨어에 위임하는 아키텍처를 지향해야 한다.
2. **IDL로 명확한 데이터 계약을 정의하라:** 프로그래밍 언어와 플랫폼에 독립적인 IDL을 사용하여 데이터 타입을 엄격하게 정의함으로써, 시스템의 상호운용성과 장기적인 유지보수성을 확보해야 한다. IDL은 단순한 스키마가 아니라, 모든 시스템 참여자가 동의하는 '공유 온톨로지'이자 의미론적 계약이다.
3. **키(Key)를 전략적으로 활용하라:** 다수의 독립적인 데이터 스트림이나 객체 상태를 관리해야 할 경우, 수많은 개별 토픽을 생성하는 대신 Keyed Topic을 사용하여 단일 토픽 내에서 여러 인스턴스로 관리해야 한다. 이는 디스커버리 부하를 극적으로 줄이고 시스템 확장성을 확보하는 가장 효과적인 방법이다.
4. **데이터의 특성에 맞는 QoS를 부여하라:** 모든 데이터에 획일적인 QoS 정책을 적용하는 것을 지양하고, 데이터의 시간적 가치, 비즈니스적 중요도, 상태 일관성 요구사항을 면밀히 분석하여 각 토픽에 최적화된 QoS '페르소나'를 부여해야 한다. 이는 시스템의 성능과 안정성 사이의 균형을 맞추는 핵심적인 과정이다.
5. **세분성(Granularity)의 균형을 찾아라:** '만능 토픽'과 '과도한 토픽 분할'이라는 양 극단의 안티-패턴을 피해야 한다. 데이터의 논리적 응집도, 변경 주기, 그리고 소비 패턴을 기준으로 함께 생성되고 함께 소비되는 데이터들을 묶어 최적의 토픽 단위를 결정해야 한다.
6. **성능을 측정하고 반복적으로 개선하라:** 토픽 설계는 일회성 활동이 아니다. 논리적으로 완벽해 보이는 설계라 할지라도, 실제 운영 환경의 부하 조건 하에서는 예상치 못한 성능 병목을 드러낼 수 있다. 따라서 시스템 개발 및 운영 전반에 걸쳐 성능을 지속적으로 프로파일링하고, 그 결과를 바탕으로 토픽의 데이터 구조, 키 설계, QoS 정책을 반복적으로 검증하고 개선해야 한다.


자율주행, 스마트 팩토리, 디지털 트윈, 대규모 IoT와 같이 수많은 디바이스가 동적으로 상호작용하는 미래의 분산 시스템 환경은 DDS 토픽 설계에 새로운 도전과 기회를 제시한다. 정적인 설계를 넘어, 시스템의 운영 상태와 네트워크 환경 변화에 따라 런타임에 동적으로 데이터 모델과 QoS를 조정하는 '적응형(adaptive) 토픽 설계'의 중요성이 더욱 부각될 것이다.

또한, DDS와 TSN(Time-Sensitive Networking)의 결합과 같이 18, 하드웨어 수준의 네트워크 인프라와 긴밀하게 연동하여 마이크로초 단위의 엄격한 실시간성과 결정성(determinism)을 보장하는 방향으로 토픽 설계 패러다임이 진화할 것이다. 이러한 미래 환경에서 본 보고서에서 제시한 체계적인 토픽 설계 원칙들은 더욱 복잡하고 지능화된 분산 시스템을 안정적이고 효율적으로 구축하기 위한 견고한 초석이 될 것이다.


1. KR20220027706A - 지능적 토픽 압축을 처리하는 dds 라우팅 서비스 장치의 동작 방법, accessed August 21, 2025, https://patents.google.com/patent/KR20220027706A/ko
2. DDS와 RTPS 개념정리 - Lab Note - 티스토리, accessed August 21, 2025, [https://lab-notes.tistory.com/entry/DDS-DDS%EC%99%80-RTPS-%EA%B0%9C%EB%85%90%EC%A0%95%EB%A6%AC](https://lab-notes.tistory.com/entry/DDS-DDS와-RTPS-개념정리)
3. 보안 DDS(Data Distribution Service)의 디스커버리 및 메시지 전송 성능 분석, accessed August 21, 2025, https://koreascience.kr/article/JAKO202117457596217.pdf
4. SDV 진짜 뼈대는 데이터 - RTI CTO가 말하는 DDS의 미래, accessed August 21, 2025, https://www.autoelectronics.co.kr/article/articleView.asp?idx=6233
5. DDS와 SOME/IP의 차이점에 대해 알아볼까요?, accessed August 21, 2025, https://jjeongil.tistory.com/189
6. DDS는 참여자(Participant)가 논리적 데이터 공간(GDS)에서 특정 토픽(Topic)을 정의하고, 토픽을 기반으로 참여자들간 직접 통신한다. 응용 프로그램은 토픽과 도메인 ID만으로도 통신이 가능하다. 아래 그림은 DDS의 통신 구조를 나타낸 그림이다 - 분산 이동컴퓨팅 연구실, accessed August 21, 2025, http://strauss.cnu.ac.kr/research_echodds.php
7. 데이터 분산 서비스 - 위키백과, 우리 모두의 백과사전, accessed August 21, 2025, [https://ko.wikipedia.org/wiki/%EB%8D%B0%EC%9D%B4%ED%84%B0_%EB%B6%84%EC%82%B0_%EC%84%9C%EB%B9%84%EC%8A%A4](https://ko.wikipedia.org/wiki/데이터_분산_서비스)
8. DDS 보안기술 - 한국전자통신연구원, accessed August 21, 2025, https://ettrends.etri.re.kr/ettrends/131/0905001659/26-5_112-122.pdf
9. An Introduction to DDS with Examples, accessed August 21, 2025, https://dds-demonstrators.readthedocs.io/en/latest/Teams/1.Hurricane/DDSguide.html
10. 03장 DDS란 무엇인가? | ROS2 하루에 입문하기, accessed August 21, 2025, https://robertchoi.gitbook.io/ros2/03-dds
11. PFDP 기반 DDS Security의 경량화 디스커버리 기법 - J-KICS, accessed August 21, 2025, https://engjournal.kics.or.kr/digital-library/manuscript/file/37317/NODE10505349.pdf
12. DDS(Data Distribution Service)란 무엇인가? - Perbiz, accessed August 21, 2025, [http://perbiz.co.kr/data/DDS(Data%20Distribution%20Service)%EB%9E%80.pdf](http://perbiz.co.kr/data/DDS(Data Distribution Service)란.pdf)
13. 가상 물리 시스템에서의 데이터 분배 서비스를 지원하기 위한 QoS 프로파일 생성 장치 및 방법, accessed August 21, 2025, https://patents.google.com/patent/KR101634322B1/ko
14. ROS2와 DDS란? - 개발하는 핑구 - 티스토리, accessed August 21, 2025, [https://ai-sinq.tistory.com/entry/ROS2%EC%99%80-DDS%EB%9E%80](https://ai-sinq.tistory.com/entry/ROS2와-DDS란)
15. 2. Topics, Domains and Partitions — The Data Distribution Service Tutorial, accessed August 21, 2025, https://download.zettascale.online/www/docs/OpenSplice/v6/html/ospl/DDSTutorial/topics-etc.html
16. Chapter 8 DDS Samples, Instances, and Keys, accessed August 21, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/users_manual/users_manual/DDS_Samples__Instances__and_Keys.htm
17. 59.9 DURABILITY QosPolicy - RTI Community, accessed August 21, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/users_manual/users_manual/DURABILITY_QosPolicy.htm
18. What's in the DDS Standard? - DDS Foundation, accessed August 21, 2025, https://www.dds-foundation.org/omg-dds-standard/
19. Extensible and Dynamic Topic Types for DDS - Object Management Group, accessed August 21, 2025, https://www.omg.org/spec/DDS-XTypes/1.1/PDF
20. [논문]데이터 분산 서비스를 활용한 실시간 시험자료 토픽 설계, accessed August 21, 2025, https://scienceon.kisti.re.kr/srch/selectPORSrchArticle.do?cn=JAKO201723839836781
21. Defining a data type via IDL - v3.0.0 - eProsima Safe DDS, accessed August 21, 2025, https://safe-dds.docs.eprosima.com/main/dds/safeddsgen/defining_types.html
22. What syntax and datatypes are supported in IDL - Eclipse Cyclone DDS - FAQ, accessed August 21, 2025, https://cyclonedds.io/content/guides/supported-idl.html
23. Import DDS Type Definitions from IDL and Create Domain Definitions Manually - MathWorks, accessed August 21, 2025, https://www.mathworks.com/help/dds/ug/import-dds-definitions-idl.html
24. DDS 모델에 식별된 유니온 정의 - IBM, accessed August 21, 2025, https://www.ibm.com/docs/ko/engineering-lifecycle-management-suite/design-rhapsody/10.0.1?topic=applications-defining-discriminated-unions-in-dds-model
25. How to model in idl for DDS - Stack Overflow, accessed August 21, 2025, https://stackoverflow.com/questions/15337914/how-to-model-in-idl-for-dds
26. Inherited dds topic structs | Data Distribution Service (DDS) Community RTI Connext Users, accessed August 21, 2025, https://community.rti.com/forum-topic/inherited-dds-topic-structs
27. KR101811728B1 - Dbms 기반의 dds 토픽 저장 방법 - Google Patents, accessed August 21, 2025, https://patents.google.com/patent/KR101811728B1/ko
28. Topic - v3.0.0, accessed August 21, 2025, https://safe-dds.docs.eprosima.com/main/dds/dds_layers/topic/topic.html
29. 3.5.1. Topics, keys and instances - 3.3.0 - eProsima Fast DDS, accessed August 21, 2025, https://fast-dds.docs.eprosima.com/en/stable/fastdds/dds_layer/topic/instances.html
30. publish subscribe - DDS Keyed Topics - Stack Overflow, accessed August 21, 2025, https://stackoverflow.com/questions/28072549/dds-keyed-topics
31. Quality of Service - OpenDDS 3.28.1, accessed August 21, 2025, https://opendds.readthedocs.io/en/dds-3.28.1/devguide/quality_of_service.html
32. A Study on an Improved DDS Discovery Method for a Large-scale System - Korea Journal Central, accessed August 21, 2025, https://journal.kci.go.kr/jksci/archive/articlePdf?artiId=ART002639375
33. QoS — Eclipse Cyclone DDS, 0.11.0, accessed August 21, 2025, https://cyclonedds.io/docs/cyclonedds/latest/api/qos.html
34. 5. Basic QoS — RTI Connext Getting Started documentation - RTI Community, accessed August 21, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/getting_started_guide/csharp/intro_qos.html
35. HISTORY QoS Parameter - DDS Foundation, accessed August 21, 2025, https://www.omgwiki.org/ddsf/doku.php?id=ddsf:public:guidebook:06_append:02_quality_of_service:history
36. Durability - OpenDDS 3.27.0, accessed August 21, 2025, https://opendds.readthedocs.io/en/dds-3.27/devguide/shapes/durability.html
37. 2. 차량 플랫폼 - SOME/IP와 DDS의 차이 - 만수르 코딩방 - 티스토리, accessed August 21, 2025, https://codingsuru0525.tistory.com/48
38. Should You Put Several Event Types in the Same Kafka Topic? | Confluent, accessed August 21, 2025, https://www.confluent.io/blog/put-several-event-types-kafka-topic/
39. Apache Kafka® Anti-Patterns and How To Avoid Them - Instaclustr, accessed August 21, 2025, https://www.instaclustr.com/blog/apache-kafka-anti-patterns/
40. Kafka Anti-Patterns: Common Pitfalls and How to Avoid Them | by Shailendra - Medium, accessed August 21, 2025, https://medium.com/@shailendrasinghpatil/kafka-anti-patterns-common-pitfalls-and-how-to-avoid-them-833cdcf2df89
41. DDS 미들웨어의 디스커버리 동작 시험을 위한 응용의 설계 및 구현, accessed August 21, 2025, https://koreascience.kr/article/CFKO201423965829156.pdf
42. [논문]A Study on an Improved DDS Discovery Method for a Large-scale System, accessed August 21, 2025, https://scienceon.kisti.re.kr/srch/selectPORSrchArticle.do?cn=JAKO202031458602981
43. Fast DDS TSC RMW report 2021 - GitHub Pages, accessed August 21, 2025, https://osrf.github.io/TSC-RMW-Reports/humble/eProsima-response.html
44. Latency Analysis of ROS2 Multi-Node Systems - arXiv, accessed August 21, 2025, https://arxiv.org/pdf/2101.02074
45. eProsima Fast DDS Performance, accessed August 21, 2025, https://www.eprosima.com/developer-resources/performance/eprosima-fast-dds-performance
46. ROS2 DDS 미들웨어의 통신 오버헤드 실험 분석, accessed August 21, 2025, https://ki-it.com/xml/39841/39841.pdf
47. Exploring the Effects of Multicast Communication on DDS Performance - arXiv, accessed August 21, 2025, https://arxiv.org/pdf/2209.09001
48. 실·가상 융합형 메타버스 서비스를 위한 DDS-TSN 계층 구조 설계 - AWS, accessed August 21, 2025, https://manuscriptlink-society-file.s3-ap-northeast-1.amazonaws.com/kips/conference/ack2022/presentation/KIPS_C2022B0323.pdf