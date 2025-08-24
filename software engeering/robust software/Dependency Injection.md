# 의존성 주입(Dependency Injection)
[견고한 소프트웨어](./index.md)


현대 소프트웨어 공학은 끊임없이 증가하는 시스템의 복잡성과의 싸움이다. 애플리케이션의 규모가 커지고 기능 요구사항이 다변화됨에 따라, 각 구성요소 간의 상호작용은 기하급수적으로 복잡해지며 이는 곧 유지보수 비용의 폭발적인 증가로 이어진다.1 이러한 복잡성의 중심에는 '의존성(dependency)'이라는 개념이 자리 잡고 있다. 전통적인 객체 지향 프로그래밍에서 한 객체가 다른 객체의 기능을 사용하기 위해 

`new` 키워드를 통해 직접 인스턴스를 생성하는 방식은 가장 직관적이지만, 동시에 가장 치명적인 문제를 내포한다. 이 행위는 '강한 결합(Tight Coupling)'을 야기하는데, 이는 특정 구현 클래스에 대한 직접적인 의존성을 코드에 명시적으로 새기는 것과 같다.1 강하게 결합된 코드는 유연성과 확장성이 현저히 떨어지며, 의존하는 객체를 격리할 수 없어 단위 테스트 작성을 극도로 어렵게 만든다.

이러한 강한 결합의 문제를 해결하기 위한 핵심 패러다임으로 '의존성 주입(Dependency Injection, DI)'이 등장했다. 의존성 주입은 객체가 자신의 의존성을 직접 생성하거나 찾는 능동적인 행위자에서, 외부로부터 필요한 의존성을 수동적으로 '주입'받는 수용자로 그 역할을 전환시키는 설계 원칙이다.3 이는 객체 간의 관계 설정 책임을 해당 객체 자신으로부터 분리하여 외부의 제3자에게 위임함으로써, 구성요소 간의 결합을 느슨하게 만들고 시스템 전체의 유연성을 극대화한다.

본 보고서는 의존성 주입을 '느슨한 결합을 위한 기술'이라는 피상적인 이해를 넘어, 그 이론적 근간을 이루는 제어의 역전(Inversion of Control, IoC) 원칙부터 심도 있게 탐구한다. 나아가 생성자, 세터, 필드 주입과 같은 핵심 구현 패턴들을 심층 비교 분석하고, 이 과정을 자동화하는 DI 컨테이너의 역할과 생명주기 관리 메커니즘을 규명한다. 또한, 서비스 로케이터(Service Locator) 및 팩토리(Factory) 패턴과 같은 관련 디자인 패턴과의 관계를 명확히 재정립하고, 의존성 관계를 방향성 비순환 그래프(DAG)라는 수학적 모델로 형식화하여 그 본질을 파헤친다. 마지막으로, Java(Spring), C#(.NET), Python(dependency-injector), TypeScript(NestJS) 등 주요 프레임워크에서의 구체적인 구현 전략과 순환 참조 문제 해결 방안을 제시함으로써, 이론과 실제를 아우르는 포괄적인 통찰을 제공하고자 한다.



의존성 주입을 이해하기 위한 첫걸음은 그 상위 개념인 제어의 역전(IoC) 원칙을 이해하는 것이다. IoC는 소프트웨어 설계의 근본적인 패러다임 전환을 의미한다.

전통적인 프로그래밍 모델에서 프로그램의 제어 흐름은 개발자가 작성한 코드에 의해 주도된다. 즉, 개발자의 코드가 능동적으로 라이브러리 함수를 호출하거나 다른 객체의 메서드를 실행하며 전체적인 작업 순서를 결정한다.1 그러나 IoC 패러다임에서는 이러한 제어권이 역전된다. 프레임워크나 컨테이너가 프로그램의 실행 흐름을 주도하고, 개발자가 작성한 코드는 프레임워크에 의해 필요한 시점에 호출되어 사용된다.5 이는 흔히 "헐리우드 원칙(Hollywood Principle)"으로 비유되는데, "우리가 당신을 부를 것이니, 먼저 우리에게 전화하지 마시오(Don't call us, we'll call you)"라는 말처럼, 개발자의 코드가 프레임워크의 호출을 기다리는 수동적인 형태가 되는 것이다.

IoC는 추상적인 원칙으로, 다양한 방식으로 구현될 수 있다. 예를 들어, 상위 클래스에서 전체적인 알고리즘의 뼈대를 정의하고 하위 클래스에서 특정 단계를 구체화하는 템플릿 메서드 패턴(Template Method Pattern) 역시 제어의 역전의 한 형태이다.6 이 외에도 전략 패턴(Strategy Pattern), 서비스 로케이터 패턴(Service Locator Pattern) 등 여러 디자인 패턴이 IoC 원칙을 구현한다.5

이러한 맥락에서 의존성 주입(DI)은 IoC 원칙을 구현하는 가장 대표적이고 구체적인 방법론 중 하나로 간주된다.5 IoC가 '누가 제어권을 갖는가'에 대한 거시적인 설계 원칙이라면, DI는 '객체 간의 의존 관계를 어떻게 설정하고 연결할 것인가'에 대한 미시적이고 구체적인 메커니즘이다.3 스프링(Spring)과 같은 현대적인 프레임워크는 IoC 컨테이너를 통해 애플리케이션 전반의 제어권을 획득하고(IoC), 이 권한을 바탕으로 객체의 생성, 생명주기 관리, 그리고 객체 간의 의존 관계 설정을 대신 처리해준다. 바로 이 '의존 관계를 설정해주는 행위'가 의존성 주입이다.4 즉, DI는 IoC라는 철학을 실현하기 위한 핵심적인 디자인 패턴인 것이다.


의존성 주입은 객체 지향 설계의 5대 원칙인 SOLID 중 하나인 의존성 역전 원칙(DIP)과 매우 깊은 관련을 맺고 있으며, 사실상 DIP를 실현하기 위한 가장 효과적인 도구이다.

DIP는 두 가지 핵심 명제로 정의된다.6

1. 상위 수준 모듈은 하위 수준 모듈에 의존해서는 안 된다. 둘 모두 추상화에 의존해야 한다.
2. 추상화는 세부 사항에 의존해서는 안 된다. 세부 사항이 추상화에 의존해야 한다.

간단히 말해, 변하기 쉬운 구체적인 구현 클래스에 직접 의존하지 말고, 안정적인 인터페이스나 추상 클래스와 같은 '추상화'에 의존하라는 것이다. 이는 소프트웨어의 유연성과 확장성을 확보하기 위한 핵심적인 아키텍처 원칙이다. 변화는 주로 시스템의 세부적인 구현 사항에서 발생하므로, 이러한 변화가 상위 수준의 정책이나 비즈니스 로직에 영향을 미치지 않도록 의존성의 방향을 '역전'시키는 것이 DIP의 목표이다.

그러나 DIP 원칙을 코드 수준에서 온전히 지키는 것은 생각보다 간단하지 않다. 개발자가 비즈니스 로직을 담고 있는 `UserService`가 데이터 접근을 담당하는 `UserRepository` 인터페이스에만 의존하도록 코드를 작성했다고 가정해 보자. 이는 DIP를 준수하는 것처럼 보인다. 하지만 프로그램이 실행되려면 어딘가에서는 반드시 `new MySqlUserRepository()`와 같이 구체적인 구현 클래스의 인스턴스를 생성해야만 한다. 만약 `UserService`가 이 생성 책임을 직접 담당한다면, `UserService`는 결국 `MySqlUserRepository`라는 구체 클래스에 의존하게 되어 DIP를 위반하게 된다.1

바로 이 지점에서 의존성 주입의 진정한 가치가 드러난다. DI는 이 '`new` 연산을 통한 객체 생성 및 연결 책임'을 클라이언트 코드(`UserService`)로부터 완전히 분리하여 외부의 제3자, 즉 DI 컨테이너에게 위임하는 메커니즘이다.1

`UserService`는 오직 `UserRepository` 인터페이스에만 의존하고, 실제 어떤 구현체가 사용될지는 외부의 설정(Configuration)에 의해 결정되며, DI 컨테이너가 런타임에 해당 구현체의 인스턴스를 생성하여 `UserService`에 '주입'해준다.

이처럼 DI는 단순한 코딩 기술이 아니라, DIP라는 중요한 아키텍처 원칙이 코드 수준에서 실현될 수 있도록 하는 구체적인 실천 방법론(praxis)이다. DI를 적용함으로써 개발자는 자연스럽게 추상화에 의존하는 코드를 작성하게 되고, 시스템은 변화에 유연하게 대응할 수 있는 구조적 강건함을 갖추게 된다.3 DI 없이는 DIP를 온전히 달성하기란 사실상 불가능에 가깝다.


의존성을 주입하는 방식에는 크게 생성자 주입, 세터 주입, 필드 주입의 세 가지 패턴이 존재한다. 각 패턴은 고유한 특성과 장단점을 가지며, 어떤 패턴을 선택하는가는 단순히 코딩 스타일의 문제가 아니라 객체의 설계 철학과 직결되는 중요한 결정이다.


생성자 주입은 객체를 생성하는 시점에 생성자의 인자를 통해 모든 필수 의존성을 주입받는 방식이다.3 이 방식의 가장 큰 특징은 객체가 생성되는 순간, 자신의 임무를 수행하는 데 필요한 모든 것을 갖춘 '완전한 상태'로 존재하게 된다는 점이다.

**장점:**

- **불변성(Immutability) 보장:** 생성자를 통해 주입받은 의존성을 `final`(Java) 또는 `readonly`(C#) 키워드를 사용하여 필드에 할당할 수 있다. 이는 객체가 생성된 이후 내부 상태(의존성)가 변경되지 않음을 보장하며, 예측 가능성을 높이고 멀티스레드 환경에서 스레드 안전성(thread-safety)을 확보하는 데 매우 유리하다.13
- **의존성 명시성:** 클래스의 생성자 시그니처 자체가 해당 클래스가 정상적으로 동작하기 위해 어떤 의존성들을 필요로 하는지를 명확하게 드러내는 계약(contract) 역할을 한다. 의존성이 누락될 경우, 컴파일 시점에 오류가 발생하거나 런타임에 객체 생성 자체가 실패하므로 문제를 조기에 발견할 수 있다.13
- **Null Pointer Exception (NPE) 방지:** 필수 의존성이 주입되지 않으면 객체 생성이 원천적으로 불가능하므로, 런타임에 의존성이 `null`이어서 발생하는 NPE를 효과적으로 방지할 수 있다.6
- **순환 참조 감지 용이성:** 대부분의 주요 DI 프레임워크는 생성자 주입 방식에서 발생하는 순환 참조(A가 B를, B가 A를 필요로 하는 상황)를 애플리케이션 구동 시점에 감지하여 예외를 발생시킨다. 이는 문제를 런타임의 특정 시점까지 미루지 않고 즉시 실패(fail-fast)하게 만들어 디버깅을 용이하게 한다.3

단점:

의존하는 객체의 수가 많아질수록 생성자의 인자 목록이 길어져 코드가 복잡해 보일 수 있다.17 하지만 이는 종종 해당 클래스가 너무 많은 책임을 지고 있어 단일 책임 원칙(Single Responsibility Principle, SRP)을 위반하고 있다는 신호(code smell)로 해석될 수 있으며, 리팩토링의 필요성을 알려주는 긍정적인 지표가 되기도 한다.


세터 주입은 객체가 기본 생성자를 통해 우선 생성된 후, 공개된 세터(setter) 메서드를 통해 의존성을 나중에 주입받는 방식이다.11

**장점:**

- **선택적 의존성(Optional Dependencies):** 의존성이 필수적이지 않고 선택적으로 제공될 수 있는 경우에 유용하다. 해당 의존성이 주입되지 않더라도 객체 자체는 생성될 수 있다.6
- **유연성:** 애플리케이션 실행 중에 의존성을 동적으로 변경하거나 재주입해야 하는 드문 시나리오에서 유연성을 제공한다.6

**단점:**

- **객체의 일시적 불완전 상태:** 객체는 생성 시점과 모든 세터 메서드가 호출 완료되는 시점 사이에 의존성이 완전히 갖춰지지 않은 '불완전한 상태'에 놓일 수 있다. 이 상태에서 객체를 사용하려 하면 예기치 않은 오류가 발생할 수 있다.17
- **NPE 발생 가능성:** DI 컨테이너가 특정 의존성에 대한 세터 메서드 호출을 누락하거나 개발자가 수동으로 주입하는 것을 잊었을 경우, 해당 의존성을 사용하는 코드에서 NPE가 발생할 위험이 있다.6
- **불변성 보장 불가:** 세터 메서드는 언제든지 호출될 수 있으므로, `final` 키워드를 사용하여 의존성의 불변성을 보장할 수 없다.6


필드 주입은 리플렉션(reflection) 기술을 사용하여 클래스의 private 필드에 직접 의존성을 주입하는 가장 간결한 방식이다. 주로 `@Autowired`(Spring)나 `@Inject`와 같은 어노테이션을 필드 선언 위에 직접 명시하여 사용한다.12

**장점:**

- **코드의 간결성:** 생성자나 세터 메서드를 작성할 필요가 없어 코드가 매우 짧아진다.17

**단점:**

- **프레임워크에 대한 강한 종속성:** DI 컨테이너 없이는 의존성을 주입할 방법이 전혀 없다. 이로 인해 순수한 단위 테스트(Unit Test) 작성이 매우 어려워진다. 테스트 코드에서는 리플렉션을 사용하여 강제로 필드 값을 설정하거나, 통합 테스트(Integration Test) 환경에 의존해야만 한다.17
- **숨겨진 의존성(Hidden Dependencies):** 의존성이 클래스의 공개된 API(생성자, 메서드)에 전혀 드러나지 않는다. 클래스 내부 구현을 들여다보지 않고서는 이 클래스가 어떤 의존성을 필요로 하는지 알 수 없어, 클래스의 재사용성과 유지보수성을 저해한다.
- **단일 책임 원칙(SRP) 위반 은폐:** 생성자 주입과 달리, 필드 주입은 의존성이 아무리 많아져도 코드상으로 복잡도가 드러나지 않는다. 이로 인해 클래스가 과도한 책임을 갖게 되는 SRP 위반 상황을 개발자가 인지하기 어렵게 만든다.20
- **불변성 보장 불가:** `final` 키워드를 사용할 수 없어 의존성의 불변성을 보장할 수 없다.20

각 주입 방식의 선택은 단순히 코드의 길이나 편의성 문제가 아니다. 이는 객체의 본질적인 설계 철학을 반영하는 행위이다. 생성자 주입은 "이 객체는 이러한 의존성들 없이는 존재할 수 없으며, 한번 정해진 의존성은 바뀌지 않는다"는 강력한 설계를 강제한다.14 반면 세터 주입은 "이 의존성은 선택 사항이거나, 나중에 변경될 수 있다"는 유연성을 부여한다.18 필드 주입은 이러한 설계적 고민을 생략하고 편의성을 극대화하지만, 그 대가로 명시성, 테스트 용이성, 불변성 등 중요한 가치를 희생한다.17 따라서 현대적인 객체 지향 설계 원칙에 부합하는 최선의 전략은, 

**필수적인 모든 의존성에 대해 생성자 주입을 기본으로 채택하고** 14, 선택적이거나 변경 가능한 극히 일부의 의존성에 한해 세터 주입을 고려하며, 필드 주입은 테스트 코드 등 매우 제한적인 경우를 제외하고는 지양하는 것이다.


| **평가 기준**             | **생성자 주입 (Constructor Injection)**             | **세터 주입 (Setter Injection)**                | **필드 주입 (Field Injection)**                 |
| ------------------------- | --------------------------------------------------- | ----------------------------------------------- | ----------------------------------------------- |
| **의존성 주입 시점**      | 객체 생성 시                                        | 객체 생성 후                                    | 객체 생성 후 (리플렉션)                         |
| **불변성 (Immutability)** | `final` 키워드로 보장 가능 (강력 권장)              | 보장 불가                                       | 보장 불가                                       |
| **의존성 누락 시**        | 컴파일 오류 또는 객체 생성 실패                     | 객체 생성은 성공, 사용 시 NPE 발생              | 객체 생성은 성공, 사용 시 NPE 발생              |
| **순환 참조 감지**        | 컴파일/애플리케이션 시작 시점에 감지                | 런타임에 실제 호출 시 `StackOverflowError` 발생 | 런타임에 실제 호출 시 `StackOverflowError` 발생 |
| **테스트 용이성**         | 매우 높음 (DI 컨테이너 없이 `new`로 객체 생성 가능) | 보통 (세터 메서드 호출 필요)                    | 매우 낮음 (DI 컨테이너 또는 리플렉션 필수)      |
| **코드 가독성/명시성**    | 높음 (생성자가 의존성을 명확히 표현)                | 보통 (세터 메서드를 통해 파악)                  | 낮음 (구현 내부를 봐야 파악 가능)               |
| **권장 사용 사례**        | 필수적이고 불변인 의존성 (대부분의 경우)            | 선택적이거나 변경 가능한 의존성                 | 권장하지 않음 (테스트 코드 등 예외적 허용)      |


의존성 주입 패턴을 수동으로 구현하는 것은 가능하지만, 애플리케이션의 규모가 커지면 객체를 생성하고 그들 간의 복잡한 의존 관계를 직접 설정하는 '조립기(Assembler)' 코드가 비대해지고 관리하기 어려워진다. DI 컨테이너는 이러한 과정을 자동화하여 개발자가 비즈니스 로직에 집중할 수 있도록 돕는 강력한 도구이다.


DI 컨테이너(또는 IoC 컨테이너)는 객체의 생성, 설정, 조립 및 생명주기 관리를 총괄하는 프레임워크 또는 라이브러리이다. Spring 프레임워크의 `ApplicationContext`가 가장 대표적인 예이다.3

DI 컨테이너의 핵심 역할은 다음과 같다.

1. **객체 생성 및 관리 (Bean Management):** 개발자가 `new` 키워드를 사용하여 직접 객체를 생성하는 대신, 컨테이너가 설정 정보를 바탕으로 객체(Spring에서는 'Bean'이라 칭함)를 생성하고 관리한다.3
2. **의존성 해결 (Dependency Resolution):** 컨테이너는 등록된 객체들 간의 의존 관계를 분석하여 하나의 거대한 '객체 그래프(Object Graph)'를 구성한다. 그리고 어떤 객체가 다른 객체를 필요로 할 때, 이 그래프를 참조하여 적절한 의존 객체를 찾아 자동으로 주입해준다.23
3. **생명주기 관리 (Lifecycle Management):** 객체의 생성부터 초기화(예: `@PostConstruct` 어노테이션이 붙은 메서드 호출), 사용, 그리고 소멸(예: `@PreDestroy` 어노테이션이 붙은 메서드 호출)에 이르는 전 과정을 관리한다.1

DI 컨테이너의 일반적인 동작 원리는 다음과 같은 단계를 거친다.

1. **설정 메타데이터 로딩:** 컨테이너는 시작 시점에 XML 파일, 어노테이션, 또는 Java 기반 설정 클래스와 같은 설정 메타데이터를 읽어들인다. 이 메타데이터에는 어떤 객체를 생성해야 하고, 그 객체들이 서로 어떻게 연결되어야 하는지에 대한 정보가 담겨 있다.9
2. **객체 그래프 생성:** 로딩된 메타데이터를 바탕으로, 컨테이너는 내부적으로 객체 간의 의존 관계를 나타내는 그래프 모델을 구축한다. 이 단계에서 의존성 해결 가능성, 순환 참조 여부 등을 검증할 수 있다.23
3. **객체 인스턴스화 및 주입:** 애플리케이션으로부터 특정 객체에 대한 요청이 들어오거나, 미리 정의된 순서에 따라 컨테이너는 의존성 그래프의 최하단(다른 객체에 의존하지 않는 객체)부터 객체를 생성하기 시작한다. 객체가 생성되면, 컨테이너는 해당 객체가 필요로 하는 다른 의존 객체들을 주입한 후, 완전히 구성된 객체를 반환한다.9


DI 컨테이너는 단순히 객체를 생성하고 주입하는 것을 넘어, 객체의 '생명주기(Lifecycle)' 또는 '스코프(Scope)'를 관리하는 중요한 역할을 수행한다. 스코프는 특정 객체 인스턴스가 얼마나 오래 유지되고 어느 범위까지 공유될지를 결정한다.

- **싱글톤 (Singleton):** 애플리케이션이 실행되는 동안 해당 타입의 객체는 단 하나만 생성된다. 이후 모든 의존성 주입 요청에 대해 동일한 인스턴스가 공유된다. 상태를 갖지 않는(stateless) 서비스나 리포지토리 객체에 주로 사용되며, 대부분의 DI 프레임워크(Spring 포함)에서 기본 스코프로 사용된다.4
- **트랜지언트 (Transient / Prototype in Spring):** 의존성 주입이 요청될 때마다 매번 새로운 인스턴스를 생성하여 반환한다. 객체가 고유한 상태를 가져야 하거나(stateful), 짧은 시간 동안 사용되고 버려져야 할 때 적합하다.23
- **스코프 (Scoped):** 미리 정의된 특정 범위 내에서만 싱글톤처럼 동작한다. 즉, 같은 스코프 내에서는 동일한 인스턴스가 공유되지만, 스코프가 달라지면 새로운 인스턴스가 생성된다. 웹 애플리케이션 환경에서 가장 흔한 예는 HTTP 요청(request) 스코프와 사용자 세션(session) 스코프이다. 요청 스코프는 하나의 HTTP 요청이 처리되는 동안 동일한 인스턴스를 보장하고, 세션 스코프는 한 사용자의 세션이 유지되는 동안 동일한 인스턴스를 보장한다.23


DI 컨테이너의 구현 방식은 애플리케이션의 성능과 개발 경험에 직접적인 영향을 미치는 중요한 아키텍처 결정 사항이다.

대부분의 주류 프레임워크(Spring,.NET Core 등)는 **런타임 DI** 방식을 채택한다. 이 방식은 애플리케이션이 시작될 때 리플렉션(Reflection) API를 사용하여 클래스, 메서드, 필드 정보를 동적으로 분석하고 의존성을 주입한다. 런타임 DI는 설정 변경에 유연하게 대응할 수 있고 비교적 배우기 쉽다는 장점이 있지만, 리플렉션 사용으로 인한 성능 오버헤드가 발생하여 애플리케이션 시작 시간이 길어질 수 있다. 또한, 의존성 설정 오류(예: 주입할 빈을 찾지 못하는 경우)가 런타임에 이르러서야 발견된다는 단점이 있다.27

반면, Dagger 2와 같은 프레임워크는 **컴파일 타임 DI** 방식을 사용한다. 이 방식은 컴파일 과정에서 어노테이션 프로세서(Annotation Processor)를 활용하여 의존성 주입에 필요한 모든 Java 코드를 미리 생성해버린다. 이렇게 생성된 코드는 리플렉션을 전혀 사용하지 않고, 마치 개발자가 직접 작성한 것처럼 일반적인 메서드 호출을 통해 의존성을 주입한다.27

컴파일 타임 DI의 장점은 명확하다.

- **성능 향상:** 리플렉션 오버헤드가 없어 런타임 성능이 우수하고 애플리케이션 시작 속도가 매우 빠르다.
- **컴파일 시점 오류 검증:** 의존성 그래프에 문제가 있거나 필요한 의존성을 제공하지 못하는 경우, 런타임이 아닌 컴파일 시점에 오류가 발생하여 안정성을 높인다.

물론, 빌드 시간이 길어지고 초기 학습 곡선이 가파를 수 있다는 트레이드오프가 존재한다.28 이러한 특성 때문에 빠른 시작 시간과 최적의 런타임 성능이 중요한 모바일 애플리케이션(Android)이나 서버리스 환경에서는 컴파일 타임 DI가 선호되는 경향이 있다. 반면, 전통적인 엔터프라이즈 애플리케이션에서는 런타임 DI가 제공하는 유연성과 개발 편의성이 더 높은 가치를 지닐 수 있다. 이처럼 DI 컨테이너의 내부 구현 방식은 단순한 기술 선택을 넘어, 대상 플랫폼과 애플리케이션의 특성을 고려한 신중한 아키텍처적 결정이 요구되는 영역이다.


의존성 주입은 객체 생성과 의존성 관리라는 공통된 관심사를 다루기 때문에, 서비스 로케이터나 팩토리 패턴과 자주 비교되거나 혼동된다. 각 패턴의 목적과 메커니즘을 명확히 구분하고 그 관계를 재정립하는 것은 DI를 깊이 있게 이해하는 데 필수적이다.


DI와 서비스 로케이터는 모두 의존성 역전 원칙(DIP)을 구현하여 결합도를 낮추려는 시도이지만, 그 접근 방식에서 근본적인 차이를 보인다.

- **개념적 차이:**
  - **의존성 주입 (DI):** 의존성이 외부에서 클래스로 '밀어 넣어지는(Push)' 수동적인 모델이다. 클래스는 생성자나 세터 메서드를 통해 자신이 필요로 하는 의존성을 명시적으로 선언하고, 외부의 조립기(DI 컨테이너)가 이를 제공해주기를 기다린다. 클래스는 의존성을 어디서 어떻게 가져오는지에 대해 전혀 알지 못한다.30
  - **서비스 로케이터 (Service Locator):** 클래스가 필요할 때마다 '끌어오는(Pull)' 능동적인 모델이다. 클래스는 서비스 로케이터라는 중앙 집중화된 레지스트리 객체에 직접 의존하며, 필요한 서비스가 생기면 이 로케이터에 명시적으로 요청하여(예: `locator.getService(MyService.class)`) 가져온다.28

이러한 메커니즘의 차이로 인해 서비스 로케이터는 종종 '안티패턴(Anti-Pattern)'으로 간주되는데, 그 이유는 다음과 같다.34

- **숨겨진 의존성 (Hidden Dependencies):** DI에서는 클래스의 생성자나 공개 메서드 시그니처만 봐도 어떤 의존성이 필요한지 명확하게 알 수 있다. 반면, 서비스 로케이터를 사용하면 이러한 의존성이 클래스 내부 구현 속에 숨겨진다. 클래스가 어떤 서비스들을 사용하는지 파악하려면 코드 내부의 모든 `locator.getService()` 호출을 일일이 찾아야만 한다. 이는 클래스의 API를 불투명하게 만들고 재사용성을 저해한다.28
- **전역 상태에 대한 의존:** 서비스 로케이터는 보통 싱글톤으로 구현되어 애플리케이션 전역에서 접근 가능한 일종의 전역 상태처럼 동작한다. 이는 모든 클라이언트 클래스가 특정 서비스 로케이터 구현체에 강하게 결합되도록 만든다. 특히 단위 테스트 시, 모든 테스트 케이스가 동일한 전역 로케이터 인스턴스와 상호작용해야 하므로 테스트 간 격리가 어려워지고 테스트 코드 작성이 복잡해진다.28
- **인터페이스 분리 원칙 (ISP) 위반:** 클래스는 자신이 실제로 사용하는 단 몇 개의 서비스만을 필요로 함에도 불구하고, 애플리케이션의 모든 서비스를 조회할 수 있는 거대한 서비스 로케이터 인터페이스 전체에 의존하게 될 수 있다. 이는 불필요한 의존성을 만들어내며, 관련 없는 서비스의 변경이 해당 클래스에 영향을 미칠 가능성을 낳는다.30

소프트웨어 설계의 대가인 마틴 파울러(Martin Fowler)는 DI와 서비스 로케이터의 선택보다 더 중요한 것은 '서비스 설정과 사용의 분리'라는 근본 원칙을 지키는 것이라고 강조했다.35 하지만 그는 의존성을 더 명확하고 명시적으로 드러낸다는 점에서 일반적으로 의존성 주입이 서비스 로케이터보다 더 나은 선택이라고 평가한다.


팩토리 패턴은 객체 생성과 관련된 디자인 패턴으로, DI와 목적과 역할에서 뚜렷한 차이를 보인다.

- **목적의 차이:**

  - **팩토리 패턴:** 객체를 생성하는 복잡한 로직을 캡슐화하는 것이 주된 목적이다. 클라이언트 코드가 `new` 키워드와 구체적인 생성 과정을 알 필요 없이, 팩토리에 요청하여 필요한 객체의 인스턴스를 얻도록 하는 데 중점을 둔다.5
  - **의존성 주입:** 객체 간의 의존 관계를 외부에서 설정하고 연결(wiring)하는 것이 주된 목적이다. 객체 생성 자체보다는, 이미 생성되었거나 생성될 객체들을 '어떻게 조립할 것인가'에 초점을 맞춘다.31

- **상호 보완적 관계:** DI와 팩토리 패턴은 서로 배타적인 관계가 아니라, 오히려 함께 사용될 때 강력한 시너지를 발휘하는 상호 보완적인 관계이다. DI 컨테이너는 그 자체로 거대한 '팩토리들의 팩토리(Factory of factories)'라고 볼 수 있다. 복잡한 객체 생성 로직이 필요한 경우, 개발자는 팩토리 패턴을 구현한 클래스(Spring에서는 `FactoryBean`)를 만들고 이를 DI 컨테이너에 빈으로 등록할 수 있다. 그러면 DI 컨테이너는 해당 팩토리를 사용하여 객체를 생성하고, 그렇게 생성된 객체를 다른 객체에 의존성으로 주입해준다.5 즉, 

  **DI는 팩토리를 관리하고 활용하는 상위 수준의 메커니즘**으로 동작할 수 있다.


| **구분**             | **의존성 주입 (Dependency Injection)**   | **서비스 로케이터 (Service Locator)**          | **팩토리 패턴 (Factory Pattern)**                |
| -------------------- | ---------------------------------------- | ---------------------------------------------- | ------------------------------------------------ |
| **핵심 목적**        | 객체 간의 의존성 연결 및 관리            | 중앙 레지스트리를 통한 서비스 조회             | 객체 생성 로직의 캡슐화                          |
| **의존성 확인 방식** | 명시적 (Explicit): 생성자/세터 시그니처  | 암시적 (Implicit): 코드 내부에서 로케이터 호출 | 클라이언트는 팩토리에만 의존                     |
| **결합도**           | 매우 낮음 (클라이언트는 추상화에만 의존) | 높음 (모든 클라이언트가 로케이터에 의존)       | 중간 (클라이언트는 팩토리에 의존)                |
| **테스트 용이성**    | 매우 높음 (Mock/Stub 객체 주입 용이)     | 낮음 (로케이터 자체를 Mocking 해야 함)         | 높음 (팩토리가 Mock 객체를 반환하도록 설정 가능) |
| **제어의 역전(IoC)** | 높음 (제어권이 컨테이너에 있음)          | 낮음 (클라이언트가 능동적으로 요청)            | 중간 (생성 제어권은 팩토리에 있음)               |


의존성 주입의 동작 원리를 더 깊이 이해하기 위해서는 소프트웨어 컴포넌트 간의 복잡한 의존 관계를 추상적이고 형식적인 모델로 표현하는 것이 유용하다. 수학의 그래프 이론은 이러한 의존성 구조를 모델링하고 분석하는 데 강력한 도구를 제공한다.38


소프트웨어 시스템 내의 모든 컴포넌트(클래스, 모듈, 빈 등)와 그들 간의 의존 관계는 하나의 방향성 그래프(Directed Graph)로 모델링될 수 있다.

이 의존성 그래프 `<code>$G$</code>`는 수학적으로 순서쌍 `<code>$(V, E)$</code>`로 형식화하여 정의할 수 있다.

- `<code>$V$</code>`는 정점(Vertex)들의 유한 집합으로, 시스템을 구성하는 각각의 컴포넌트를 나타낸다. 예를 들어, `UserService`, `UserRepository`, `DataSource` 등이 각각 하나의 정점이 된다.
- `<code>$E$</code>`는 간선(Edge)들의 집합으로, 정점 간의 의존 관계를 나타내는 순서쌍 `<code>$(u, v)$</code>`의 집합이다. 간선 `<code>$(u, v) \in E$</code>`는 정점 `<code>$u$</code>`가 정점 `<code>$v$</code>`에 의존한다는 것을 의미하며, 이는 화살표로 `<code>$u \rightarrow v$</code>`와 같이 표현된다. 이 간선은 방향성을 가지므로, `<code>$u$</code>`가 `<code>$v$</code>`에 의존하는 것과 `<code>$v$</code>`가 `<code>$u$</code>`에 의존하는 것은 명백히 다르다.43

예를 들어, `OrderService`가 `PaymentService`와 `OrderRepository`를 필요로 한다면, 의존성 그래프에는 `(OrderService, PaymentService)`와 `(OrderService, OrderRepository)`라는 두 개의 방향성 간선이 존재하게 된다.


건전하고 관리 가능한 소프트웨어 아키텍처는 그 의존성 그래프가 특별한 속성을 가져야 하는데, 바로 '순환(cycle)'이 없어야 한다는 것이다.

**순환 참조(Circular Dependency)**는 의존성 그래프에서 특정 정점에서 출발하여 여러 간선을 따라 이동했을 때 다시 자기 자신으로 돌아오는 경로가 존재하는 경우를 의미한다. 가장 단순한 예는 두 컴포넌트가 서로를 의존하는 경우(`<code>$A \rightarrow B$</code>` 이고 `<code>$B \rightarrow A$</code>`)이다. 이 경우, 그래프에는 `<code>$(A, B)$</code>`와 `<code>$(B, A)$</code>`라는 두 간선이 존재하여 `<code>$A \rightarrow B \rightarrow A$</code>` 라는 순환 경로가 형성된다.44

이러한 순환 참조는 DI 컨테이너에게 치명적인 문제를 야기한다. A를 생성하려면 B가 먼저 생성되어야 하고, B를 생성하려면 A가 먼저 생성되어야 하는 논리적 모순에 빠지기 때문이다. 이로 인해 DI 컨테이너는 어느 객체부터 생성해야 할지 순서를 결정할 수 없게 되어 의존성 해결에 실패하고 애플리케이션 구동을 중단시킨다.44

따라서, 잘 설계된 시스템의 의존성 그래프는 순환이 없는 **방향성 비순환 그래프(Directed Acyclic Graph, DAG)** 형태를 가져야 한다. 이는 로버트 C. 마틴(Robert C. Martin)이 제안한 **비순환 의존 원칙(Acyclic Dependencies Principle, ADP)**에서도 강조하는 바로, "컴포넌트 의존성 그래프에 순환이 있어서는 안 된다"는 설계 원칙이다.45


DI 컨테이너가 의존성 그래프를 바탕으로 객체들을 어떤 순서로 생성하고 초기화해야 하는지를 결정하는 과정은, 그래프 이론의 '위상 정렬(Topological Sorting)' 알고리즘을 통해 설명될 수 있다.

**위상 정렬**이란, DAG의 모든 정점을 일렬로 나열하는 것으로, 이때 그래프의 모든 방향성 간선 `<code>$(u, v)$</code>`에 대해 정점 `<code>$u$</code>`가 정점 `<code>$v$</code>`보다 항상 앞에 위치하도록 하는 정렬 방식이다.46 즉, 의존하는 쪽이 의존받는 쪽보다 항상 뒤에 오도록 순서를 정하는 것이다.

DI 컨테이너는 내부적으로 바로 이 위상 정렬 알고리즘을 수행하여 안전한 객체 생성 및 초기화 순서를 결정한다. 알고리즘은 일반적으로 다음과 같이 동작한다.

1. 의존성 그래프에서 자신을 가리키는 간선이 하나도 없는 정점(indegree가 0인 노드)을 찾는다. 이 정점들은 다른 어떤 컴포넌트에도 의존하지 않으므로 가장 먼저 생성될 수 있다.
2. 이 정점들을 생성하고, 정렬 결과 리스트에 추가한다.
3. 생성된 정점들과 그들로부터 나가는 모든 간선을 그래프에서 제거한다.
4. 이 과정으로 인해 새롭게 indegree가 0이 된 정점들이 나타나게 된다.
5. 그래프에 더 이상 정점이 남아있지 않을 때까지 1-4 과정을 반복한다.

이러한 관점에서 볼 때, DI 컨테이너의 '빈(Bean) 생성' 과정은 추상적인 마법이 아니라, 의존성 그래프에 대한 '위상 정렬' 알고리즘의 구체적인 응용 사례이다. 위상 정렬은 그래프에 순환이 없을 때, 즉 DAG일 때만 성공적으로 수행될 수 있다. 만약 그래프에 순환이 존재하면, indegree가 0인 정점이 더 이상 나타나지 않는 시점이 오게 되어 알고리즘이 중단된다. 이것이 바로 Spring 프레임워크에서 생성자 주입 방식의 순환 참조가 발생했을 때 `BeanCurrentlyInCreationException`과 같은 예외가 발생하는 근본적인 이유이다.49 이는 프레임워크의 동작이 잘 확립된 컴퓨터 과학 알고리즘에 기반하고 있음을 명확히 보여주는 사례라 할 수 있다.


의존성 주입의 원리와 패턴은 특정 언어나 프레임워크에 국한되지 않는 보편적인 개념이지만, 실제 구현 방식은 각 기술 스택의 특성에 따라 다르게 나타난다. 본 장에서는 Java, C#, Python, TypeScript 생태계의 대표적인 프레임워크에서 DI가 어떻게 구현되고 관리되는지 구체적인 전략을 살펴본다.


Spring 프레임워크는 DI 개념을 대중화시킨 가장 대표적인 사례로, 강력하고 유연한 DI 컨테이너를 제공한다.

- **설정 방식:**
  - **XML 기반 설정:** 초창기 Spring에서는 `<bean>`, `<constructor-arg>`, `<property>`와 같은 XML 태그를 사용하여 빈의 정의와 의존 관계를 명시했다. 이는 코드와 설정을 완전히 분리하는 장점이 있지만, 설정 파일이 비대해지고 관리하기 어려운 단점이 있다.19
  - **어노테이션 기반 설정:** `@Component`, `@Service`, `@Repository` 등의 스테레오타입 어노테이션으로 클래스를 스캔 대상 빈으로 지정하고, `@Autowired` 어노테이션을 사용하여 의존성을 주입하는 방식이다. 코드가 간결해지고 설정이 직관적이어서 현대 Spring 애플리케이션 개발의 표준으로 자리 잡았다.12
- **코드 예제 (어노테이션 기반 생성자 주입):**

```Java
// 의존성 대상 인터페이스
public interface MessageService {
    String getMessage();
}

// 구현체
@Component
public class EmailService implements MessageService {
    @Override
    public String getMessage() {
        return "Message from EmailService";
    }
}

// 의존성을 주입받는 클래스
@Component
public class MessageProcessor {
    private final MessageService messageService;

    // @Autowired는 생성자가 하나일 경우 생략 가능
    @Autowired
    public MessageProcessor(MessageService messageService) {
        this.messageService = messageService;
    }

    public void process() {
        System.out.println(messageService.getMessage());
    }
}
```

- 순환 참조 해결:

  Spring은 생성자 주입 방식에서 순환 참조가 발생하면 애플리케이션 구동 시점에 BeanCurrentlyInCreationException을 발생시켜 문제를 즉시 알린다.49 불가피하게 순환 참조를 해결해야 할 경우, 

  `@Lazy` 어노테이션을 사용할 수 있다. `@Lazy`는 의존성 주입 시점에 실제 빈 인스턴스 대신, 해당 빈을 처음 사용할 때 실제 인스턴스를 찾아주는 프록시(Proxy) 객체를 주입한다. 이로써 빈 생성 시점의 순환 고리를 끊을 수 있다.49 하지만 이는 근본적인 설계 문제를 임시로 가리는 해결책일 수 있으므로, 의존성 구조를 리팩토링하여 순환을 제거하는 것이 우선적으로 권장된다.52


.NET(구.NET Core)은 프레임워크 수준에서 DI 컨테이너를 기본으로 내장하고 있어 별도의 라이브러리 없이도 DI를 손쉽게 적용할 수 있다.

- **서비스 등록:** 애플리케이션의 시작점인 `Program.cs` 파일에서 `IServiceCollection` 인터페이스에 확장 메서드를 사용하여 서비스를 등록한다. 서비스의 생명주기에 따라 `AddSingleton()`, `AddScoped()`, `AddTransient()`를 사용한다.24

- **주입 방식 및 설정 기반 DI:**.NET은 생성자 주입을 강력하게 권장한다. 프레임워크는 등록된 서비스 타입과 일치하는 생성자 매개변수를 발견하면 자동으로 인스턴스를 주입해준다.25 또한, 

  `appsettings.json`과 같은 설정 파일의 값을 `IOptions<T>` 인터페이스를 통해 클래스에 주입받아, 코드 변경 없이 설정을 통해 서비스의 동작을 동적으로 구성하는 강력한 기능을 제공한다.24

- **코드 예제:**

```C#
// Program.cs - 서비스 등록
var builder = WebApplication.CreateBuilder(args);

// IMyDependency 인터페이스에 대한 구현으로 MyDependency를 스코프 생명주기로 등록
builder.Services.AddScoped<IMyDependency, MyDependency>();
builder.Services.AddRazorPages();

var app = builder.Build();
//...

// MyPage.cshtml.cs - 생성자 주입
public class MyPageModel : PageModel
{
    private readonly IMyDependency _myDependency;

    public MyPageModel(IMyDependency myDependency)
    {
        _myDependency = myDependency;
    }

    public void OnGet()
    {
        _myDependency.WriteMessage("MyPageModel.OnGet called.");
    }
}
```

- **순환 참조 해결:**.NET의 내장 DI 컨테이너 역시 순환 참조를 감지하면 `InvalidOperationException`을 발생시킨다. 이 문제를 해결하기 위해 `Lazy<T>` 클래스를 사용할 수 있다. `Lazy<T>`를 주입받으면, 해당 의존성의 인스턴스는 생성자 호출 시점이 아닌, `.Value` 속성에 처음 접근하는 시점에 생성(지연 로딩)되므로 순환 참조 고리를 끊을 수 있다.58


Python은 동적 타입 언어로, 정적 타입 언어와는 다른 방식의 DI 라이브러리가 발전했다. 그중 `dependency-injector`는 선언적인 방식으로 컨테이너를 구성하여 유연성과 가독성을 높인 대표적인 라이브러리이다.

- **컨테이너와 프로바이더:** `containers.DeclarativeContainer`를 상속하여 DI 컨테이너를 정의한다. 컨테이너 내부에는 객체의 생성 및 관리 방식을 정의하는 '프로바이더(Provider)'를 선언한다.
  - `providers.Configuration`: 설정 값(YAML,.ini 파일 등)을 주입한다.
  - `providers.Factory`: 호출될 때마다 새로운 객체를 생성한다.
  - `providers.Singleton`: 애플리케이션 내에서 유일한 인스턴스를 생성하고 공유한다.62
- **주입:** `@inject` 데코레이터를 함수나 메서드에 적용하여 의존성을 주입받을 수 있도록 표시한다. 이후 컨테이너의 `wire()` 메서드를 호출하여 지정된 모듈들에 실제 주입이 이뤄지도록 연결한다.63
- **코드 예제:**

```Python
# containers.py
from dependency_injector import containers, providers
from.services import ApiClient, SearchService

class Container(containers.DeclarativeContainer):
    config = providers.Configuration()
    api_client = providers.Singleton(ApiClient, api_key=config.api.key)
    search_service = providers.Factory(SearchService, api_client=api_client)

# views.py
from dependency_injector.wiring import inject, Provide
from.containers import Container
from.services import SearchService

@inject
def search_view(search_service: SearchService = Provide[Container.search_service]):
    results = search_service.search("DI in Python")
    #... render results
```

- **순환 참조 해결:** Python에서는 주로 모듈 임포트 시점에 순환 참조 문제가 발생한다. DI 패턴을 적용하면, 모듈이 다른 모듈을 직접 임포트하는 대신 런타임에 컨테이너로부터 객체를 주입받게 되므로, 임포트 시점의 강한 결합을 런타임 시점의 느슨한 객체 연결로 전환하여 이 문제를 완화할 수 있다.66

  `dependency-injector` 라이브러리 자체도 순환 의존성을 가진 프로바이더 그래프를 순회하고 해결할 수 있는 메커니즘을 내장하고 있다.65


NestJS는 TypeScript의 강력한 타입 시스템을 기반으로, 모듈(Module) 중심의 체계적인 DI 시스템을 제공하여 대규모 애플리케이션 개발에 적합하다.

- **모듈 기반 DI:** 애플리케이션은 `@Module` 데코레이터로 장식된 여러 모듈의 조합으로 구성된다. 각 모듈은 `providers`(서비스), `controllers`, `imports`(다른 모듈), `exports`(자신의 프로바이더를 외부에 노출)를 명시적으로 정의하여, 캡슐화와 명확한 의존성 경계를 강제한다.69
- **주입:** 클래스의 생성자에서 타입 힌트(type hint)를 사용하여 의존성을 선언하면, NestJS의 DI 시스템이 해당 타입에 맞는 프로바이더를 현재 모듈의 스코프 내에서 찾아 자동으로 주입해준다.
- **코드 예제:**

```TypeScript
// cats.service.ts
@Injectable()
export class CatsService {
    //...
}

// cats.controller.ts
@Controller('cats')
export class CatsController {
    constructor(private readonly catsService: CatsService) {}
    //...
}

// cats.module.ts
@Module({
    controllers: [CatsController],
    providers:,
})
export class CatsModule {}
```

- **순환 참조 해결:** NestJS에서 모듈 간 또는 프로바이더 간에 순환 참조가 발생할 경우, `forwardRef()` 함수를 사용하여 의존성 해결을 지연시킬 수 있다. `forwardRef()`는 아직 정의되지 않은 클래스를 참조할 수 있도록 하는 래퍼(wrapper) 함수로, 순환 관계에 있는 양쪽 모듈의 `imports` 배열이나 프로바이더의 `@Inject` 데코레이터 내에서 모두 사용되어야 한다.70

```TypeScript
// photo.service.ts
@Injectable()
export class PhotoService {
  constructor(
    @Inject(forwardRef(() => AlbumService)) // 순환 참조 해결
    private albumService: AlbumService,
  ) {}
}

// album.service.ts
@Injectable()
export class AlbumService {
  constructor(
    @Inject(forwardRef(() => PhotoService)) // 순환 참조 해결
    private photoService: PhotoService,
  ) {}
}
```


의존성 주입(DI)은 단순히 코드를 정리하고 객체를 생성하는 편리한 기술을 넘어, 현대 소프트웨어 공학의 복잡성에 대응하기 위한 핵심적인 아키텍처 원리이다. 본 보고서를 통해 살펴본 바와 같이, DI는 제어의 역전(IoC)이라는 거시적 패러다임과 의존성 역전 원칙(DIP)이라는 구체적인 설계 원칙에 깊이 뿌리내리고 있다. 그 본질은 객체 간의 의존 관계 설정 책임을 객체 자신으로부터 분리하여 외부의 조립자(DI 컨테이너)에게 위임함으로써, 시스템의 각 구성요소를 독립적으로 개발, 테스트, 교체, 재사용할 수 있도록 만드는 데 있다. 이는 결국 소프트웨어를 변화에 유연하고, 확장에 용이하며, 테스트하기 쉽게 만드는 근본적인 힘이 된다.2

생성자, 세터, 필드 주입과 같은 다양한 패턴의 비교 분석은 DI의 적용이 단순한 문법 선택이 아닌, 객체의 불변성, 완전성, 명시성에 대한 설계적 결정을 내포하고 있음을 보여주었다. 또한, 의존성 그래프의 형식적 모델링을 통해 DI 컨테이너의 동작이 '위상 정렬'이라는 확고한 컴퓨터 과학 알고리즘에 기반하고 있으며, 순환 참조와 같은 일반적인 문제가 왜 발생하는지에 대한 근본적인 이해를 제공했다.

성공적인 의존성 주입 적용을 위해 다음의 원칙들을 제언한다.

1. **생성자 주입을 기본으로 삼아라:** 객체의 불변성을 보장하고, 필요한 의존성을 명시적으로 드러내며, 객체가 항상 완전한 상태로 생성되도록 강제하라. 이는 가장 안전하고 예측 가능한 설계 방식이다.
2. **추상화에 의존하라:** 구체적인 구현 클래스가 아닌 인터페이스나 추상 클래스에 의존함으로써 의존성 역전 원칙(DIP)을 철저히 준수하라. 이는 구현의 교체를 용이하게 하고 시스템의 유연성을 극대화한다.
3. **순환 참조를 설계의 경고 신호로 받아들여라:** 순환 참조는 종종 클래스 간의 책임 분리가 제대로 이루어지지 않았다는 신호이다. `@Lazy`나 `forwardRef()`와 같은 프레임워크가 제공하는 기술적 해결책에 의존하기 전에, 먼저 의존성 구조를 재검토하고 리팩토링을 통해 순환을 근본적으로 제거할 수 없는지 심각하게 고민해야 한다.
4. **컨테이너는 조립 루트(Composition Root)에서만 구성하라:** DI 컨테이너의 설정과 객체 그래프의 조립은 애플리케이션의 시작점(예: `main` 메서드, `Program.cs`)과 같이 단일하고 책임 있는 위치에서만 이루어져야 한다. 비즈니스 로직을 담고 있는 코드 내에서 컨테이너에 직접 접근하여 의존성을 가져오는 서비스 로케이터 방식은 의존성을 다시 숨기는 결과를 초래하므로 지양해야 한다.

마이크로서비스, 서버리스, 모듈형 모놀리스 등 분산되고 독립적으로 배포 가능한 컴포넌트로 시스템을 구축하는 현대 아키텍처 패러다임에서, 각 컴포넌트 간의 명확한 계약과 느슨한 결합의 중요성은 그 어느 때보다 강조되고 있다. 이러한 환경에서 의존성 주입은 각 컴포넌트의 의존성을 명확하게 관리하고, 테스트 환경과 운영 환경에 따라 유연하게 구성을 변경할 수 있게 해주는 필수 불가결한 기술이다. 앞으로도 DI는 소프트웨어 아키텍처의 견고함과 유연성을 지탱하는 핵심 기둥으로서 그 중요성을 계속해서 더해갈 것이며, 컴파일 타임 DI와 같은 성능 최적화 기술의 발전은 더 넓은 영역으로의 확산을 가속화할 것이다.


1. ABOUT.Series (12) IoC (Inversion of Control; 제어의 역전) & DI (Dependency Injection; 의존성 주입) - THE DEVELOPER - 티스토리, 8월 16, 2025에 액세스, [https://bitkunst.tistory.com/entry/ABOUTSeries-12-IoC-Inversion-of-Control-%EC%A0%9C%EC%96%B4%EC%9D%98-%EC%97%AD%EC%A0%84-DI-Dependency-Injection-%EC%9D%98%EC%A1%B4%EC%84%B1-%EC%A3%BC%EC%9E%85](https://bitkunst.tistory.com/entry/ABOUTSeries-12-IoC-Inversion-of-Control-제어의-역전-DI-Dependency-Injection-의존성-주입)
2. DI의 장점과 단점 - K's dev log - 티스토리, 8월 16, 2025에 액세스, https://ken-dev-log.tistory.com/30
3. [Spring] IoC(제어의 역전)와 DI(의존성 주입)의 완벽 이해, 8월 16, 2025에 액세스, [https://developshrimp.com/entry/Spring-IoC%EC%A0%9C%EC%96%B4%EC%9D%98-%EC%97%AD%EC%A0%84%EC%99%80-DI%EC%9D%98%EC%A1%B4%EC%84%B1-%EC%A3%BC%EC%9E%85%EC%9D%98-%EC%99%84%EB%B2%BD-%EC%9D%B4%ED%95%B4](https://developshrimp.com/entry/Spring-IoC제어의-역전와-DI의존성-주입의-완벽-이해)
4. [Spring] 의존성 주입(Dependency Injection, DI)이란? 및 Spring이 의존성 주입을 지원하는 이유 - 망나니개발자, 8월 16, 2025에 액세스, https://mangkyu.tistory.com/150
5. IoC(제어 역전) / DI(의존성 주입) 개념 익히기 in Spring - 01, 8월 16, 2025에 액세스, https://cat-minzzi.tistory.com/75
6. [Spring] IoC(제어의 역전)와 DI(의존성 주입) 그리고 Spring - 오늘의 개발 - 티스토리, 8월 16, 2025에 액세스, https://oneul-losnue.tistory.com/364
7. shout-to-my-mae.tistory.com, 8월 16, 2025에 액세스, [https://shout-to-my-mae.tistory.com/425#:~:text=IoC%EB%8A%94%20%EA%B0%9D%EC%B2%B4%EC%9D%98%20%EC%83%9D%EC%84%B1,%EB%A5%BC%20%EC%A3%BC%EC%9E%85%ED%95%B4%EC%A3%BC%EB%8A%94%20%EA%B2%83%EC%9D%84%20%EB%A7%90%ED%95%9C%EB%8B%A4.](https://shout-to-my-mae.tistory.com/425#:~:text=IoC는 객체의 생성,를 주입해주는 것을 말한다.)
8. DI와 IOC, 그리고 Spring - velog, 8월 16, 2025에 액세스, [https://velog.io/@heyday_7/DI%EC%99%80-IOC](https://velog.io/@heyday_7/DI와-IOC)
9. [Spring] IoC Container - 아마란스 생각, 8월 16, 2025에 액세스, [https://amaran-th.github.io/Spring/[Spring\]%20IoC%20Container/](https://amaran-th.github.io/Spring/[Spring] IoC Container/)
10. DI와 서비스 로케이터 - Incheol's TECH BLOG, 8월 16, 2025에 액세스, https://incheol-jung.gitbook.io/docs/study/undefined/di
11. DI(Dependency Injection) 이란? / (포스팅 하나로 세부내용까지 총 정리) - 미니의 일상, 8월 16, 2025에 액세스, https://mininkorea.tistory.com/48
12. [Spring] 의존성 주입 시 @Autowired보다 생성자 주입을 권장하는 이유 - chaewss - 티스토리, 8월 16, 2025에 액세스, https://chaewsscode.tistory.com/206
13. [Spring] 의존성 주입(DI) 시 생성자 주입(Constructor Injection)을 사용해야하는 이유, 8월 16, 2025에 액세스, https://dev-jwblog.tistory.com/118
14. [DI] DI에서 생성자 주입과 setter 주입 중 더 선호되는 방식, 8월 16, 2025에 액세스, https://lifework-archive-reservoir.tistory.com/m/206
15. [Spring] 의존성 주입 3가지 방법 - (생성자 주입, Field 주입, Setter 주입) - 슬기로운 개발생활, 8월 16, 2025에 액세스, https://dev-coco.tistory.com/70
16. 생성자 vs setter vs field 의존성 주입 - 너도나도 함께 성장하기 - 티스토리, 8월 16, 2025에 액세스, https://escapefromcoding.tistory.com/719
17. Spring DI의 세 가지 방식: 생성자 주입, 세터 주입, 필드 주입, 8월 16, 2025에 액세스, https://programmer7895.tistory.com/87
18. 스프링 DI(Dependency Injection) 3가지 방법과 장단점 - I Luv Console, 8월 16, 2025에 액세스, https://rerewww.github.io/spring/spring-create-bean/
19. Dependency Injection :: Spring Framework, 8월 16, 2025에 액세스, https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html
20. DI(Dependency Injection)의 종류와 장단점 - EastHShin - 티스토리, 8월 16, 2025에 액세스, https://easthshin.tistory.com/9
21. 기술면접 스터디 - 2일차 (DI, Index), 8월 16, 2025에 액세스, https://jipang9-greedy-pot.tistory.com/170
22. 스프링 프레임워크에서의 의존성 주입(DI) 이해하기 - F-Lab, 8월 16, 2025에 액세스, https://f-lab.kr/insight/understanding-dependency-injection-in-spring-framework
23. [DI] DI 컨테이너의 동작 원리와 주요 기능 - ‍♀️ 삽질 기록 저장소, 8월 16, 2025에 액세스, https://lifework-archive-reservoir.tistory.com/220
24. Dependency injection in ASP.NET Core | Microsoft Learn, 8월 16, 2025에 액세스, https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-9.0
25. Dependency injection - .NET | Microsoft Learn, 8월 16, 2025에 액세스, https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection
26. Understanding Dependency Injection in .NET Core - Auth0, 8월 16, 2025에 액세스, https://auth0.com/blog/dependency-injection-in-dotnet-core/
27. [DI]DI 컨테이너의 성능에 영향을 미치는 요소와 개선 방법, 8월 16, 2025에 액세스, https://lifework-archive-reservoir.tistory.com/m/221
28. DI (Dependency Injection) 는 왜 써야할까? (2) – Android 에서의 DI - Newbie Developer, 8월 16, 2025에 액세스, [https://wonsohana.wordpress.com/2021/05/15/di-dependency-injection-%EB%8A%94-%EC%99%9C-%EC%8D%A8%EC%95%BC%ED%95%A0%EA%B9%8C-2-android-%EC%97%90%EC%84%9C%EC%9D%98-di/](https://wonsohana.wordpress.com/2021/05/15/di-dependency-injection-는-왜-써야할까-2-android-에서의-di/)
29. Dependency Injection with Dagger 2 | ITkonekt, 8월 16, 2025에 액세스, https://blog.itkonekt.com/2018/07/26/dependency-injection-with-dagger-2/
30. DI(Dependency Injection)와 서비스 로케이터 - leesche blog - 티스토리, 8월 16, 2025에 액세스, https://dev-leesh.tistory.com/84
31. 의존성 주입이랑 팩토리 패턴이랑 같은 거 아냐? : r/javahelp - Reddit, 8월 16, 2025에 액세스, https://www.reddit.com/r/javahelp/comments/15mcudc/is_dependency_injection_and_factory_pattern_the/?tl=ko
32. What's the difference between the Dependency Injection and Service Locator patterns?, 8월 16, 2025에 액세스, https://stackoverflow.com/questions/1557781/whats-the-difference-between-the-dependency-injection-and-service-locator-patte
33. Service Locator Pattern, 서비스 로케이터 패턴, 8월 16, 2025에 액세스, https://blog.mayleaf.dev/m/25
34. Service Locator is an Anti-Pattern - ploeh blog, 8월 16, 2025에 액세스, https://blog.ploeh.dk/2010/02/03/ServiceLocatorisanAnti-Pattern/
35. Inversion of Control Containers and the Dependency Injection pattern, 8월 16, 2025에 액세스, https://martinfowler.com/articles/injection.html
36. Service locator vs dependency injection. | Weird Scenes Inside The Goldmine, 8월 16, 2025에 액세스, https://guy-murphy.github.io/2014/11/24/service-locator-vs-dependency-injection/
37. What's the difference between the Service Locator and the Factory Design pattern?, 8월 16, 2025에 액세스, https://stackoverflow.com/questions/8325619/whats-the-difference-between-the-service-locator-and-the-factory-design-pattern
38. [논문]실제문제들의 그래프 이론을 이용한 수학적 모델링, 8월 16, 2025에 액세스, https://scienceon.kisti.re.kr/srch/selectPORSrchArticle.do?cn=DIKO0009941425
39. Structural equation modeling - Wikipedia, 8월 16, 2025에 액세스, https://en.wikipedia.org/wiki/Structural_equation_modeling
40. Topology Analysis of Software Dependencies - McGill School Of Computer Science, 8월 16, 2025에 액세스, https://www.cs.mcgill.ca/~martin/papers/tosem2008.pdf
41. (PDF) Using dependency models to manage complex software architecture - ResearchGate, 8월 16, 2025에 액세스, https://www.researchgate.net/publication/221321534_Using_dependency_models_to_manage_complex_software_architecture
42. Is there a database for tracking the dependencies of mathematical theorems?, 8월 16, 2025에 액세스, https://mathoverflow.net/questions/249383/is-there-a-database-for-tracking-the-dependencies-of-mathematical-theorems
43. Dependency Graph in Compiler Design - GeeksforGeeks, 8월 16, 2025에 액세스, https://www.geeksforgeeks.org/compiler-design/dependency-graph-in-compiler-design/
44. Dependency graph - Wikipedia, 8월 16, 2025에 액세스, https://en.wikipedia.org/wiki/Dependency_graph
45. Acyclic dependencies principle - Wikipedia, 8월 16, 2025에 액세스, https://en.wikipedia.org/wiki/Acyclic_dependencies_principle
46. What Is a Directed Acyclic Graph (DAG)? | IBM, 8월 16, 2025에 액세스, https://www.ibm.com/think/topics/directed-acyclic-graph
47. Directed Acyclic Graphs: DI (Day 3) | by Abou Zuhayr | Medium, 8월 16, 2025에 액세스, https://medium.com/@zuhayr.codes/directed-acyclic-graphs-di-day-3-3ee6b611e24b
48. Common Graph Algorithms: Directed Acyclic Graph (DAG) Algorithm - DEV Community, 8월 16, 2025에 액세스, https://dev.to/capnspek/common-graph-algorithms-directed-acyclic-graph-dag-algorithm-4bpl
49. Circular Dependencies in Spring - Baeldung, 8월 16, 2025에 액세스, https://www.baeldung.com/circular-dependencies-in-spring
50. Circular Dependencies in Spring - GeeksforGeeks, 8월 16, 2025에 액세스, https://www.geeksforgeeks.org/java/circular-dependencies-in-spring/
51. Spring Dependency Injection with Example - GeeksforGeeks, 8월 16, 2025에 액세스, https://www.geeksforgeeks.org/advance-java/spring-dependency-injection-with-example/
52. Solving Circular Dependency Issues in Spring Boot using @Lazy and Service Decomposition | by Balmik Prajapati | Medium, 8월 16, 2025에 액세스, https://medium.com/@balmikprajapati53/solving-circular-dependency-issues-in-spring-boot-using-lazy-and-service-decomposition-15f372cdaaee
53. spring - how to fix Circular Dependencies? - Stack Overflow, 8월 16, 2025에 액세스, https://stackoverflow.com/questions/77042552/how-to-fix-circular-dependencies
54. BeanCurrentlyInCreationException: Circular Dependencies in Spring - Gopi Gorantala, 8월 16, 2025에 액세스, https://www.ggorantala.dev/beancurrentlyincreationexception-circular-dependencies-in-spring/
55. Dependency Injection in ASP.NET Core Explained - codewithmukesh, 8월 16, 2025에 액세스, https://codewithmukesh.com/blog/dependency-injection-in-aspnet-core-explained/
56. NET Core Dependency Injection with Configuration - blogs.cninnovation.com, 8월 16, 2025에 액세스, https://csharp.christiannagel.com/2016/08/16/diwithconfiguration/
57. Configuration Driven Dependency Injection (DI) with .NET Core | by Zach Saw | Medium, 8월 16, 2025에 액세스, https://zach-saw.medium.com/configuration-driven-dependency-injection-di-with-net-core-d960965d7f8a
58. Lazily resolving services to fix circular dependencies in .NET Core - Thomas Levesque, 8월 16, 2025에 액세스, https://thomaslevesque.com/2020/03/18/lazily-resolving-services-to-fix-circular-dependencies-in-net-core/
59. c# - Circular dependency in dependency injection - Software Engineering Stack Exchange, 8월 16, 2025에 액세스, https://softwareengineering.stackexchange.com/questions/350762/circular-dependency-in-dependency-injection
60. Effortlessly Resolving Circular Dependencies in .NET with SmartInject - CodeProject, 8월 16, 2025에 액세스, https://www.codeproject.com/Tips/5369405/Effortlessly-Resolving-Circular-Dependencies-in-NE
61. Breaking Circular Dependencies in Microsoft DI with Lazy Resolution - Not a Designer, 8월 16, 2025에 액세스, https://notadesigner.com/breaking-circular-dependencies-in-microsoft-di-with-lazy-resolution/
62. Tutorials - Dependency Injector 4.48.1 documentation, 8월 16, 2025에 액세스, https://python-dependency-injector.ets-labs.org/tutorials/index.html
63. Injector Library and Exploring Dependency Injection in Python | by Luke Garzia | Medium, 8월 16, 2025에 액세스, https://medium.com/@garzia.luke/injector-library-and-exploring-dependency-injection-in-python-4ce10560cd24
64. Dependency Injection in Python | Better Stack Community, 8월 16, 2025에 액세스, https://betterstack.com/community/guides/scaling-python/python-dependency-injection/
65. How to handle circular dependency [non-string keys for Dict provider] / Issue #327 / ets-labs/python-dependency-injector - GitHub, 8월 16, 2025에 액세스, https://github.com/ets-labs/python-dependency-injector/issues/327
66. Untangling Circular Dependencies in Python | by Aman Deep | Jun, 2025 - Medium, 8월 16, 2025에 액세스, https://medium.com/@aman.deep291098/untangling-circular-dependencies-in-python-61316529c1f6
67. How to properly handle a circular module dependency in Python? - Stack Overflow, 8월 16, 2025에 액세스, https://stackoverflow.com/questions/20382403/how-to-properly-handle-a-circular-module-dependency-in-python
68. Container providers traversal - Dependency Injector 4.48.1 documentation, 8월 16, 2025에 액세스, https://python-dependency-injector.ets-labs.org/containers/traversal.html
69. 제어 역전(IoC)과 의존성 주입(DI) - velog, 8월 16, 2025에 액세스, [https://velog.io/@jazzcafe12/%EC%A0%9C%EC%96%B4-%EC%97%AD%EC%A0%84IoC%EA%B3%BC-%EC%9D%98%EC%A1%B4%EC%84%B1-%EC%A3%BC%EC%9E%85DI](https://velog.io/@jazzcafe12/제어-역전IoC과-의존성-주입DI)
70. Circular dependency | NestJS - A progressive Node.js framework, 8월 16, 2025에 액세스, https://docs.nestjs.com/fundamentals/circular-dependency
71. Circular dependency in Node.js and Nest.js - DEV Community, 8월 16, 2025에 액세스, https://dev.to/successgilli/circular-dependency-in-nodejs-and-nestjs-3e1d
72. Common errors - FAQ | NestJS - A progressive Node.js framework, 8월 16, 2025에 액세스, https://docs.nestjs.com/faq/common-errors
73. Nestjs circular dependency using forwardref - Stack Overflow, 8월 16, 2025에 액세스, https://stackoverflow.com/questions/76445874/nestjs-circular-dependency-using-forwardref
74. Understanding Circular Dependency in NestJS - DigitalOcean, 8월 16, 2025에 액세스, https://www.digitalocean.com/community/tutorials/understanding-circular-dependency-in-nestjs

