[Java Spring](./index.md)
# Java Spring


2000년대 초반, Java를 사용한 엔터프라이즈 애플리케이션 개발 환경은 J2EE(Java 2 Platform, Enterprise Edition) 명세와 그 핵심 구현체인 EJB(Enterprise JavaBeans) 기술이 지배하고 있었다. 당시 개발자들은 EJB가 제공하는 트랜잭션 관리, 분산 기술, ORM(Object-Relational Mapping)과 같은 강력한 기능들을 활용하여 대규모 시스템을 구축했다.1 그러나 이러한 강력함의 이면에는 극심한 복잡성과 비효율성이 존재했다. 개발자들은 이를 'Java 진영의 추운 겨울'이라 묘사하며, 끝없는 야근과 기술적 장벽에 부딪혔다.2

EJB의 근본적인 문제는 그 침습적인(invasive) 특성과 과도한 복잡성에 있었다. 비즈니스 로직을 담고 있는 단순한 객체조차도 EJB 컨테이너가 요구하는 특정 인터페이스를 구현해야 했으며, 복잡한 배포 서술자(Deployment Descriptor) XML 파일을 작성해야만 했다. 이는 개발 과정을 번거롭게 만들었을 뿐만 아니라, 순수한 객체 지향 설계 원칙을 훼손했다. 또한, EJB 컴포넌트는 애플리케이션 서버라는 무거운 컨테이너 외부에서는 테스트하기가 거의 불가능하여, 개발 및 테스트 주기를 현저히 지연시키는 원인이 되었다.1 로드 존슨(Rod Johnson)은 EJB를 '오버엔지니어링의 산물'이라 비판하며, 단순한 기능을 구현하기 위해 불필요한 부분까지 강제로 대비하게 만드는 구조적 문제를 지적했다.3

이러한 기술적, 철학적 배경 속에서 Spring 프레임워크가 등장했다. Spring은 단순히 EJB의 대안을 제시하는 것을 넘어, 엔터프라이즈 Java 개발의 근본적인 패러다임 전환을 이끌었다. 그 핵심에는 단순성, POJO(Plain Old Java Object), 그리고 개발자 중심의 설계 철학이 자리 잡고 있었다. 본 보고서는 Spring이 어떻게 EJB의 겨울을 끝내고 Java 생태계에 새로운 봄을 가져왔는지, 그 핵심 원리와 아키텍처는 무엇인지, 그리고 끊임없는 진화를 통해 어떻게 현대 클라우드 네이티브 시대의 주역으로 자리매김했는지를 심층적으로 고찰하고자 한다. Spring의 성공은 단일 기술의 우수성을 넘어, 엔터프라이즈급 기능이 우수한 객체 지향 설계 원칙과 개발자 생산성을 희생시켜서는 안 된다는 철학적 승리였음을 규명하는 것이 본 보고서의 핵심 목표이다.



Spring 프레임워크의 사상적, 기술적 토대는 2002년 로드 존슨이 출간한 "Expert One-on-One J2EE Design and Development"라는 책에서 시작되었다.1 이 책은 단순히 J2EE와 EJB의 문제점을 나열하는 데 그치지 않고, 그 문제들을 해결하기 위한 구체적인 설계 원칙과 함께 약 3만 줄에 달하는 실용적인 코드를 제시했다. 이 코드는 오늘날 Spring 프레임워크의 핵심 개념인 `BeanFactory`, `ApplicationContext`, 의존성 주입(DI), 제어의 역전(IoC) 등의 원형을 포함하고 있었으며, 프레임워크의 기술적 청사진 역할을 했다.1


로드 존슨의 책이 개발자 커뮤니티에서 큰 반향을 일으키자, 유겐 휠러(Juergen Hoeller)와 얀 카로프(Yann Caroff)가 그에게 책의 코드를 기반으로 한 오픈소스 프로젝트를 제안했다.2 이 제안을 계기로 본격적인 프레임워크 개발이 시작되었으며, 이것이 Spring 프레임워크의 공식적인 탄생이다.

프레임워크의 이름인 "Spring"은 상징적인 의미를 담고 있다. 이는 J2EE와 EJB로 대표되던 복잡하고 어려운 '겨울'의 시대를 끝내고, 개발자들에게 사용하기 편리한 새로운 '봄'이 왔음을 의미하는 은유였다.1 이 이름은 Spring이 기술적 해결책을 넘어 개발자들에게 가져다준 해방감을 효과적으로 표현했다.


Spring의 가장 핵심적이고 혁명적인 철학은 바로 POJO(Plain Old Java Object) 중심 설계에 있다. POJO란 특정 프레임워크나 기술에 종속되지 않은, 순수한 Java 객체를 의미한다. EJB가 비즈니스 객체에게 `EJBObject`와 같은 특정 인터페이스의 구현을 강요했던 것과 달리, Spring은 비즈니스 로직이 오직 비즈니스 자체에만 집중할 수 있도록 어떠한 기술적 제약도 강요하지 않았다.2

이 POJO 중심 철학은 다음과 같은 중요한 이점을 가져왔다.

- **느슨한 결합 (Loose Coupling):** 컴포넌트들이 프레임워크와 강하게 결합되지 않으므로, 특정 기술의 변화에 유연하게 대처할 수 있으며 코드의 재사용성이 높아진다.
- **향상된 테스트 용이성 (Enhanced Testability):** 비즈니스 로직을 담은 POJO는 무거운 엔터프라이즈 컨테이너의 구동 없이, 단순한 JUnit 테스트만으로도 단위 테스트가 가능하다. 이는 개발 생산성과 코드 품질을 획기적으로 향상시켰다.
- **객체 지향 설계의 순수성 보존:** 개발자는 프레임워크의 제약에서 벗어나 SOLID와 같은 순수한 객체 지향 설계 원칙을 자유롭게 적용할 수 있었다. Spring은 DI와 같은 메커니즘을 통해 오히려 개방-폐쇄 원칙(OCP)이나 의존관계 역전 원칙(DIP)과 같은 객체 지향 원칙들을 더 잘 지킬 수 있도록 지원했다.2

결론적으로, POJO라는 철학은 Spring의 모든 기술적 결정의 근간이 되었다. 순수한 객체인 POJO가 어떻게 다른 객체와의 의존관계를 설정하고, 트랜잭션이나 보안과 같은 엔터프라이즈 서비스를 제공받을 수 있을까? 이 질문에 대한 해답이 바로 다음에 설명할 제어의 역전(IoC)과 관점 지향 프로그래밍(AOP)이다. 즉, IoC/DI와 AOP는 단순히 Spring의 기능이 아니라, POJO 중심 철학을 엔터프라이즈 환경에서 실현하기 위한 필연적인 기술적 귀결이었다.



전통적인 프로그래밍 방식에서 객체의 생성, 구성, 메소드 호출 등 프로그램의 흐름 제어권은 전적으로 개발자가 작성한 코드에 있었다.5 예를 들어, `A` 객체가 `B` 객체를 필요로 할 때, `A` 객체 내부에서 `new B()`와 같이 직접 `B` 객체를 생성하고 제어했다.

제어의 역전(IoC)은 이러한 제어 흐름의 패러다임을 뒤집는 설계 원칙이다. IoC 환경에서는 객체의 생성과 생명주기 관리 등 모든 제어권이 개발자의 코드에서 프레임워크(컨테이너)로 넘어가게 된다.5 개발자는 각 객체(컴포넌트)가 어떤 일을 할 것인지만 정의하고, 이 객체들을 어떻게 생성하고 서로 연결할 것인지는 프레임워크가 결정한다.


의존성 주입(DI)은 IoC 원칙을 구현하는 구체적인 디자인 패턴이다.8 객체가 필요로 하는 다른 객체(의존성)를 자기 자신이 내부에서 직접 생성하는 대신, 외부(IoC 컨테이너)로부터 전달받는(주입받는) 방식이다.6

예를 들어, `MenuController`가 `MenuService`를 필요로 할 때, `new MenuService()` 코드를 `MenuController` 내부에 작성하는 것은 두 클래스 간의 강한 결합(Tight Coupling)을 야기한다.5 만약 `MenuService`의 구현이 변경되면 `MenuController`의 코드도 수정해야 한다. DI를 사용하면 `MenuController`는 단지 `MenuService` 인터페이스에만 의존하고, 실제 구현 객체는 외부의 IoC 컨테이너가 생성하여 주입해준다. 이로써 두 클래스는 느슨한 결합(Loose Coupling) 관계를 유지하게 되어 유연하고 확장성 높은 설계가 가능해진다.5


Spring은 IoC와 DI를 구현하기 위한 핵심 엔진으로 IoC 컨테이너를 제공한다. Spring에서 관리하는 객체를 '빈(Bean)'이라고 부르며, IoC 컨테이너는 이 빈들의 생성, 관계 설정, 사용, 소멸까지 전 생명주기를 관리한다.6 Spring의 IoC 컨테이너는 주로 두 가지 인터페이스로 대표된다.

- **BeanFactory:** IoC 컨테이너의 가장 기본적인 형태이다. 빈을 생성하고 의존성을 주입하는 핵심 기능을 제공하지만, 그 외의 부가 기능은 거의 없다.11

- **ApplicationContext:** `BeanFactory` 인터페이스를 상속받아 확장한 고급 컨테이너이다.15

  `BeanFactory`의 모든 기능을 포함하며, 국제화(i18n) 지원, 이벤트 발행 및 구독, AOP와의 손쉬운 통합 등 엔터프라이즈 애플리케이션 개발에 필요한 다양한 부가 기능을 제공한다. 따라서 대부분의 Spring 애플리케이션에서는 `ApplicationContext`를 IoC 컨테이너로 사용한다.10


Spring IoC 컨테이너 내부에서 하나의 빈이 생성되고 소멸되기까지의 과정은 다음과 같은 정교한 단계를 거친다.

1. **컨테이너 초기화:** `new AnnotationConfigApplicationContext(AppConfig.class)`와 같이 `ApplicationContext` 구현체를 생성하며 컨테이너가 시작된다.18

2. **빈 정의(Bean Definition) 등록:** 컨테이너는 XML 설정 파일이나 `@Configuration` 어노테이션이 붙은 Java 클래스와 같은 설정 정보를 파싱한다. 그리고 각 빈에 대한 모든 메타데이터(클래스 이름, 스코프, 생성자 인자, 프로퍼티 값, 초기화/소멸 메소드 등)를 담고 있는 `BeanDefinition`이라는 객체를 생성하여 내부 레지스트리에 등록한다.18 이 단계는 실제 빈 인스턴스를 만들기 전의 '설계도'를 만드는 과정과 같다.

3. **빈 인스턴스화:** 컨테이너는 등록된 `BeanDefinition` 정보를 바탕으로, 리플렉션(Reflection)을 사용하여 빈의 실제 인스턴스를 생성한다.9

4. **프로퍼티 설정 (의존성 주입):** 인스턴스화된 빈 객체의 의존성을 주입한다. `BeanDefinition`에 명시된 정보를 바탕으로 생성자 주입, 세터 주입, 필드 주입 등의 방식으로 필요한 다른 빈들을 주입한다.7

5. **Aware 인터페이스 처리:** 만약 빈이 `BeanNameAware`, `ApplicationContextAware`와 같은 `Aware` 인터페이스를 구현했다면, 컨테이너는 해당 빈에게 자신의 이름이나 컨테이너 자신에 대한 참조를 알려주기 위해 `setBeanName()`, `setApplicationContext()`와 같은 메소드를 호출한다.21

6. **초기화 콜백 (Initialization Callbacks):** 의존성 주입까지 완료된 빈을 대상으로 초기화 로직을 수행할 수 있는 콜백 메소드를 호출한다. 호출 순서는 다음과 같다.21

   - `@PostConstruct` 어노테이션이 붙은 메소드

   - `InitializingBean` 인터페이스의 `afterPropertiesSet()` 메소드

   - 설정 정보에 init-method로 지정된 커스텀 초기화 메소드

     이 중 @PostConstruct는 Java 표준 어노테이션으로 Spring 프레임워크에 대한 의존성을 줄일 수 있어 가장 권장되는 방식이다.25

7. **빈 사용 준비 완료:** 모든 초기화 과정이 끝나면, 빈은 애플리케이션에서 사용될 준비가 완료된 상태로 컨테이너에 보관된다.

8. **소멸 콜백 (Destruction Callbacks):** 애플리케이션이 종료되어 컨테이너가 닫힐 때, 컨테이너는 관리하던 빈들을 소멸시킨다. 이때 소멸 전 특정 로직을 수행할 수 있는 콜백 메소드를 호출한다. 호출 순서는 다음과 같다.22

   - `@PreDestroy` 어노테이션이 붙은 메소드
   - `DisposableBean` 인터페이스의 `destroy()` 메소드
   - 설정 정보에 `destroy-method`로 지정된 커스텀 소멸 메소드

이처럼 실제 빈 인스턴스와 그 메타데이터인 `BeanDefinition`을 분리하는 것은 Spring 아키텍처의 핵심적인 설계 결정이다. 컨테이너는 실제 클래스가 아닌 추상화된 메타데이터를 기반으로 동작하기 때문에, 설정 정보의 출처(XML, Java 등)에 관계없이 일관된 방식으로 빈을 관리할 수 있다. 이러한 추상화 계층은 `BeanPostProcessor`와 같은 강력한 확장 포인트를 가능하게 하여, Spring 프레임워크 자체의 유연성과 확장성을 극대화하는 기반이 된다.



객체 지향 프로그래밍(OOP)은 데이터와 그 데이터를 처리하는 행위를 하나의 '객체'로 묶어 모듈화하는 데 매우 효과적이다. 그러나 애플리케이션을 개발하다 보면 여러 모듈에 공통적으로 나타나지만, 각 모듈의 핵심 비즈니스 로직과는 거리가 있는 부가 기능들이 존재한다. 로깅, 보안, 트랜잭션 관리, 성능 측정 등이 대표적인 예이다.26 이러한 기능들은 애플리케이션 전반에 걸쳐 횡단(cross-cut)으로 나타나기 때문에 '횡단 관심사(Cross-Cutting Concerns)'라고 불린다.

OOP만으로는 이러한 횡단 관심사를 깔끔하게 모듈화하기 어렵다. 그 결과, 관련 코드가 여러 클래스와 메소드에 흩어져 중복(code scattering)되고, 하나의 메소드 안에 비즈니스 로직과 부가 기능 코드가 뒤섞이는(code tangling) 문제가 발생한다. 이는 코드의 가독성을 떨어뜨리고 유지보수를 어렵게 만든다.


AOP는 이러한 횡단 관심사를 '관점(Aspect)'이라는 별도의 모듈로 분리하여 관리하는 프로그래밍 패러다임이다.26 AOP를 통해 개발자는 핵심 비즈니스 로직에만 집중할 수 있으며, 횡단 관심사는 필요한 곳에 선언적으로 적용할 수 있다. 이로써 코드의 중복을 제거하고 모듈성을 높여 재사용성과 유지보수성을 향상시킬 수 있다.27


Spring AOP를 이해하기 위해서는 다음과 같은 핵심 용어에 대한 이해가 필요하다.

- **Aspect (관점):** 횡단 관심사를 모듈화한 단위. 트랜잭션 관리나 로깅과 같은 부가 기능을 정의한 모듈이다. Spring에서는 `@Aspect` 어노테이션이 붙은 클래스로 표현된다.26
- **Join Point (조인 포인트):** 프로그램 실행 중 Aspect가 적용될 수 있는 모든 시점이나 위치. 메소드 실행, 생성자 호출, 필드 접근 등 다양하다. Spring AOP에서는 프록시 방식의 한계로 인해 **메소드 실행** 시점만이 Join Point가 될 수 있다.26
- **Advice (어드바이스):** Aspect가 Join Point에서 수행하는 실제 동작. 즉, 부가 기능 로직 그 자체이다. Advice의 종류에는 `@Before` (메소드 실행 전), `@After` (메 '소드 실행 후, 성공/실패 무관), `@AfterReturning` (메소드 정상 실행 후), `@AfterThrowing` (예외 발생 후), `@Around` (메소드 실행 전후) 등이 있다.26
- **Pointcut (포인트컷):** 수많은 Join Point 중에서 Advice를 적용할 대상을 선별하는 표현식. 즉, '어디에' 부가 기능을 적용할지를 정의한다.26
- **Target (타겟):** Advice가 적용되는 대상 객체. 핵심 비즈니스 로직을 구현하는 클래스이다.26
- **Weaving (위빙):** Aspect를 Target 객체에 적용하여 부가 기능이 추가된 새로운 프록시 객체를 만드는 과정이다. Spring AOP에서는 런타임에 Weaving이 일어난다.27


Spring AOP는 완전한 AOP 구현체인 AspectJ와 달리, 프록시(Proxy) 패턴을 기반으로 동작하는 실용적인 AOP 솔루션이다.27 Target 객체의 코드를 직접 수정하는 대신, 런타임에 Target 객체를 감싸는 프록시 객체를 생성한다.

클라이언트가 Target의 메소드를 호출하면, 실제로는 프록시 객체의 메소드가 호출된다. 이 프록시 객체는 Pointcut 설정에 따라 Advice(부가 기능)를 먼저 실행한 후, 실제 Target 객체의 메소드를 호출하거나, Target 메소드 호출 후에 Advice를 실행하는 방식으로 동작한다.

Spring은 두 가지 방식의 프록시 생성 기술을 사용한다.

- **JDK Dynamic Proxy:** Java 표준 라이브러리에서 제공하는 기술로, Target 객체가 하나 이상의 인터페이스를 구현하고 있을 때 사용된다. 런타임에 해당 인터페이스들을 모두 구현하는 새로운 프록시 클래스를 동적으로 생성한다. 인터페이스 기반으로만 프록시를 만들 수 있다는 제약이 있다.31
- **CGLIB (Code Generation Library):** Target 객체가 인터페이스를 구현하지 않는 순수 클래스일 경우 사용된다. CGLIB는 런타임에 Target 클래스를 상속받는 자식 클래스를 동적으로 생성하여 프록시로 사용한다. 따라서 `final` 클래스나 `private` 메소드에는 적용할 수 없다.32 과거에는 성능과 안정성 문제로 인터페이스가 있을 경우에만 제한적으로 사용되었으나, 기술이 발전하면서 Spring Boot 2.0부터는 기본 프록시 생성 방식으로 채택되었다.34

Spring AOP가 런타임 프록시 방식을 채택한 것은 실용적인 선택이었다. 컴파일 시점이나 클래스 로딩 시점에 바이트코드를 조작하는 AspectJ 방식보다 기능적으로는 제한적이지만(메소드 실행 조인 포인트만 지원), 별도의 빌드 과정이 필요 없고 Spring IoC 컨테이너와 완벽하게 통합된다는 큰 장점이 있다. IoC 컨테이너가 클라이언트에게 Target 빈을 직접 반환하는 대신, AOP가 적용된 프록시 빈을 감쪽같이 반환해주면 되기 때문이다. 이러한 설계는 AOP를 IoC의 자연스러운 확장 기능으로 만들어 개발자들이 매우 쉽게 AOP를 도입할 수 있도록 했다. 이는 복잡성을 피하고 실용성을 추구하는 Spring의 핵심 철학과 정확히 일치한다.


Spring 프레임워크는 단일 기능을 수행하는 거대한 프레임워크가 아니라, 독립적이면서도 유기적으로 연동되는 여러 모듈의 집합체로 설계되었다.3 이러한 모듈식 설계 덕분에 개발자는 프로젝트에 필요한 기능만 선택적으로 도입할 수 있으며, 이는 Spring이 '경량(lightweight) 프레임워크'라고 불리는 이유 중 하나이다. Spring이라는 단어 자체가 때로는 모호하게 사용될 수 있을 만큼 그 생태계는 방대하지만, 그 구조를 핵심 모듈 단위로 분해하면 명확한 아키텍처를 파악할 수 있다.2

Spring 프레임워크의 아키텍처는 크게 몇 개의 논리적인 계층으로 나눌 수 있으며, 각 계층은 여러 개의 모듈로 구성된다.11


| **Category (분류)**           | **Module/Artifact (모듈/아티팩트)**                          | **Core Function (핵심 기능)**                                |
| ----------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Core Container**            | `spring-core`, `spring-beans`, `spring-context`, `spring-expression` | IoC 컨테이너, DI, Bean 관리, SpEL(표현 언어) 제공 11         |
| **AOP & Instrumentation**     | `spring-aop`, `spring-aspects`, `spring-instrument`          | AOP 구현, AspectJ 통합, 클래스 계측 지원 12                  |
| **Data Access / Integration** | `spring-jdbc`, `spring-tx`, `spring-orm`, `spring-oxm`, `spring-jms` | JDBC 추상화, 선언적 트랜잭션 관리, ORM(JPA, Hibernate) 통합, Object/XML 매핑, JMS 지원 11 |
| **Web**                       | `spring-web`, `spring-webmvc`, `spring-websocket`            | 기본 웹 기능, Spring MVC 프레임워크, WebSocket 지원 11       |
| **Test**                      | `spring-test`                                                | 단위 및 통합 테스트 지원 (JUnit, Mockito 연동) 12            |
| **Messaging**                 | `spring-messaging`                                           | 메시지 기반 애플리케이션 구축을 위한 추상화 제공 12          |


Spring 아키텍처의 가장 중요한 특징은 모든 모듈이 **Core Container**를 기반으로 구축된다는 점이다.11 Core Container가 제공하는 IoC/DI 기능은 Spring 생태계의 모든 프로젝트와 모듈이 동작하는 근간을 이룬다. 예를 들어, Spring MVC의 `DispatcherServlet` 내부에서 사용되는 `HandlerMapping`이나 `Controller` 같은 컴포넌트들은 모두 Core Container에 의해 빈으로 관리되고 의존성이 주입된다. 마찬가지로, Data Access 계층의 `DataSource`나 `TransactionManager` 역시 컨테이너가 관리하는 빈이다. 이처럼 IoC 컨테이너는 Spring의 모든 기능을 하나로 묶고 조립하는 중심축 역할을 수행하며, Spring 프레임워크의 일관성과 확장성을 보장하는 핵심 요소이다.


Spring MVC는 서블릿 API를 기반으로 웹 애플리케이션을 개발하기 위한 강력하고 유연한 프레임워크이다. 그 중심에는 '프론트 컨트롤러(Front Controller)' 디자인 패턴을 구현한 `DispatcherServlet`이 있다.37


프론트 컨트롤러 패턴은 모든 클라이언트의 웹 요청을 단일 진입점(Servlet)에서 중앙집중식으로 처리하는 방식이다. `DispatcherServlet`은 이 진입점 역할을 수행하며, 요청을 받으면 바로 처리하는 것이 아니라, 요청 분석, 공통 작업 처리 후 실제 비즈니스 로직을 처리할 세부 컨트롤러에게 작업을 위임한다. 이 구조는 공통 로직의 중복을 피하고 애플리케이션의 구조를 깔끔하게 유지하는 데 도움을 준다.37


클라이언트로부터 HTTP 요청이 들어왔을 때, `DispatcherServlet` 내부에서는 다음과 같은 단계별 처리 과정이 일어난다.

1. **요청 접수:** 서블릿 컨테이너(예: Tomcat)는 들어온 HTTP 요청을 가장 먼저 `DispatcherServlet`에게 전달한다.40
2. **핸들러 매핑 (HandlerMapping):** `DispatcherServlet`은 등록된 `HandlerMapping` 구현체들에게 현재 요청(URI, HTTP 메소드 등)을 처리할 핸들러(일반적으로 `@Controller` 클래스의 메소드)가 누구인지 질의한다. 가장 널리 사용되는 `RequestMappingHandlerMapping`은 `@RequestMapping`, `@GetMapping` 등의 어노테이션 정보를 기반으로 적절한 컨트롤러 메소드를 찾아 반환한다.37
3. **핸들러 어댑터 (HandlerAdapter) 조회:** `HandlerMapping`을 통해 핸들러를 찾았다면, `DispatcherServlet`은 이 핸들러를 실행할 수 있는 `HandlerAdapter`를 찾는다. Spring MVC는 다양한 형태의 핸들러를 지원하기 때문에, 각 핸들러 유형에 맞는 어댑터가 필요하다. 어노테이션 기반 컨트롤러의 경우 `RequestMappingHandlerAdapter`가 선택된다.37
4. **컨트롤러 실행:** `HandlerAdapter`는 찾아낸 컨트롤러 메소드를 인자값 처리(예: `@RequestParam`, `@RequestBody` 등)와 함께 실행한다. 컨트롤러는 비즈니스 로직을 수행하고, 그 결과를 `Model` 객체에 담고, 뷰에 대한 논리적인 이름(예: "home")을 문자열로 반환한다.37
5. **뷰 리졸버 (ViewResolver) 호출:** `DispatcherServlet`은 컨트롤러로부터 반환받은 논리적 뷰 이름과 `Model` 데이터를 `ViewResolver`에게 전달한다.37
6. **뷰 해석 (View Resolution):** `ViewResolver`는 논리적 뷰 이름을 실제 물리적인 뷰 리소스(예: `/WEB-INF/views/home.jsp` 파일 또는 `templates/home.html` Thymeleaf 템플릿)와 매핑하여, 렌더링을 수행할 `View` 객체를 반환한다.37
7. **뷰 렌더링:** `DispatcherServlet`은 `View` 객체에게 `Model` 데이터를 전달하여 최종 응답 화면(예: HTML)을 렌더링하도록 요청한다. `View`는 `Model` 데이터를 사용하여 템플릿을 완성한다.37
8. **응답:** 렌더링된 최종 응답은 `DispatcherServlet`을 통해 클라이언트에게 전송된다.40

이러한 `DispatcherServlet`의 아키텍처는 객체 지향의 개방-폐쇄 원칙(Open-Closed Principle)을 훌륭하게 적용한 사례이다. `DispatcherServlet`의 핵심 요청 처리 흐름은 고정되어 있어 변경에 닫혀 있지만(`closed for modification`), `HandlerMapping`, `HandlerAdapter`, `ViewResolver`와 같은 전략 인터페이스의 새로운 구현체를 추가함으로써 프레임워크의 기능을 얼마든지 확장할 수 있다(`open for extension`). Spring이 어노테이션 기반 컨트롤러에서 나아가 WebFlux의 함수형 엔드포인트와 같은 새로운 프로그래밍 모델을 도입할 수 있었던 것도, `DispatcherServlet`의 핵심 코드를 변경하지 않고 새로운 전략 구현체를 제공하는 것만으로 가능했기 때문이다. 이처럼 전략 패턴에 기반한 유연한 설계는 Spring MVC가 오랜 시간 동안 변화에 적응하며 발전할 수 있었던 핵심 동력이다.


데이터 접근 계층(Data Access Layer, DAL)을 구현하는 것은 전통적으로 많은 상용구 코드(boilerplate code)를 필요로 하는 작업이었다. JDBC API를 직접 사용하든, JPA(Java Persistence API)를 사용하든, 개발자는 `EntityManager`를 획득하고, 트랜잭션을 시작하며, CRUD(Create, Read, Update, Delete) 쿼리를 작성하고, 리소스를 해제하는 반복적인 코드를 작성해야 했다. Spring Data JPA는 이러한 데이터 접근 계층의 개발을 극적으로 단순화하기 위해 등장한 프로젝트이다.45


Spring Data JPA의 핵심은 개발자가 구현 클래스 없이 인터페이스를 정의하는 것만으로 데이터 접근 로직을 완성할 수 있다는 점이다.47 예를 들어, `User` 엔티티에 대한 데이터 접근은 다음과 같은 인터페이스를 정의하는 것으로 충분하다.

```Java
public interface UserRepository extends JpaRepository<User, Long> {
    // 추가적인 구현 코드 없음
}
```


이것이 가능한 이유는 Spring Data JPA가 애플리케이션 시작 시점에 `JpaRepository`를 상속한 모든 인터페이스를 스캔하여, 런타임에 해당 인터페이스의 프록시(Proxy) 구현체를 동적으로 생성하기 때문이다. 그리고 이 프록시 객체를 Spring 컨테이너에 빈으로 등록한다. 따라서 개발자가 `UserRepository`를 주입받아 사용할 때, 실제로 주입되는 것은 Spring Data JPA가 만들어준 프록시 객체이다.47

이 프록시 객체는 모든 메소드 호출을 가로채서(intercept) 정해진 규칙에 따라 동작한다.

- **CRUD 메소드의 동작:** `save()`, `findById()`, `delete()`와 같이 `JpaRepository`에 기본적으로 정의된 메소드들은 `SimpleJpaRepository`라는 공통 구현 클래스에 위임된다. 이 클래스는 내부적으로 `EntityManager`를 사용하여 실제 데이터베이스 작업을 수행한다.46 특히 

  `save()` 메소드는 정교하게 동작하는데, 전달된 엔티티의 식별자(`@Id` 필드) 값이 `null`이면 새로운 엔티티로 간주하여 `EntityManager.persist()`를 호출하고, 식별자 값이 존재하면 이미 데이터베이스에 있는 데이터의 수정으로 간주하여 `EntityManager.merge()`를 호출한다.48

- **쿼리 메소드 (Query Derivation Mechanism):** Spring Data JPA의 가장 강력한 기능 중 하나는 메소드 이름을 분석하여 자동으로 JPQL(Java Persistence Query Language) 쿼리를 생성하는 것이다. 예를 들어, 개발자가 `UserRepository`에 `findByEmailAndNameOrderByCreatedDateDesc(String email, String name)`과 같은 메소드를 선언하면, 프록시 객체는 이 메소드 이름을 파싱한다. `FindBy`, `And`, `OrderBy`, `Desc`와 같은 키워드를 기준으로 메소드 이름을 분해하고, `Email`과 `Name`을 `User` 엔티티의 필드에 매핑하여 `SELECT u FROM User u WHERE u.email =?1 AND u.name =?2 ORDER BY u.createdDate DESC`와 같은 JPQL 쿼리를 동적으로 생성하여 실행한다.47

- **`@Query` 어노테이션:** 메소드 이름만으로 표현하기 어려운 복잡한 쿼리는 `@Query` 어노테이션을 사용하여 직접 JPQL이나 네이티브 SQL을 작성할 수 있다. 이 경우, 프록시는 메소드 이름 분석 대신 어노테이션에 명시된 쿼리를 실행한다.50

Spring Data JPA는 '설정보다 관례(Convention over Configuration)'라는 원칙을 데이터 접근 계층에 완벽하게 구현한 사례이다. 개발자는 정해진 명명 규칙(convention)을 따름으로써, JPQL 쿼리를 작성하는 설정(configuration) 작업에서 해방된다. 이 접근 방식은 개발자의 초점을 데이터베이스와의 상호작용이라는 기계적인 작업에서 비즈니스 로직에 필요한 데이터 연산의 '의도'를 정의하는 것으로 이동시켜, 생산성을 극적으로 향상시킨다.


Spring Security는 Java 기반 엔터프라이즈 애플리케이션의 보안(인증 및 인가)을 위한 사실상의 표준 프레임워크이다. 서블릿 필터(Servlet Filter) 기반으로 동작하며, 강력하고 유연한 보안 기능을 선언적으로 적용할 수 있도록 지원한다.


Spring Security의 동작을 이해하기 위해서는 두 가지 핵심 개념을 명확히 구분해야 한다.

- **인증 (Authentication):** 사용자가 누구인지 신원을 확인하는 과정이다. "당신은 누구인가?"에 대한 답을 찾는 과정으로, 일반적으로 아이디와 비밀번호를 통한 로그인이 여기에 해당한다.51
- **인가 (Authorization):** 인증된 사용자가 특정 리소스에 접근하거나 특정 기능을 실행할 수 있는 권한이 있는지 확인하는 과정이다. "당신은 이 작업을 수행할 수 있는가?"에 대한 답을 찾는 과정이다.51 인가는 반드시 인증 이후에 이루어진다.


Spring Security의 모든 기능은 서블릿 필터 체인을 통해 구현된다. 클라이언트의 모든 HTTP 요청은 애플리케이션의 `DispatcherServlet`에 도달하기 전에 Spring Security가 구성한 일련의 필터들을 거치게 된다.

그 중심에는 `DelegatingFilterProxy`와 `FilterChainProxy`가 있다. `DelegatingFilterProxy`는 서블릿 컨테이너의 생명주기에 의해 관리되는 표준 서블릿 필터로, 실제 로직은 직접 처리하지 않고 Spring 컨테이너에 등록된 `FilterChainProxy`라는 이름의 빈에게 모든 처리를 위임한다.54

`FilterChainProxy`는 실제 보안 로직을 담고 있는 여러 보안 필터들(예: `CsrfFilter`, `UsernamePasswordAuthenticationFilter`, `FilterSecurityInterceptor` 등)의 체인을 관리한다. 요청 URL 패턴에 따라 적용할 필터 체인을 결정하고, 해당 체인에 속한 필터들을 정해진 순서대로 실행시킨다.54


1. 사용자가 로그인 폼에 아이디와 비밀번호를 입력하고 제출하면, `UsernamePasswordAuthenticationFilter`가 이 요청을 가로챈다.52
2. 이 필터는 요청에서 추출한 아이디와 비밀번호를 사용하여, 아직 인증되지 않은 상태의 `Authentication` 객체(구체적으로는 `UsernamePasswordAuthenticationToken`)를 생성한다.51
3. 생성된 토큰은 인증을 처리하기 위해 `AuthenticationManager`에게 전달된다.
4. `AuthenticationManager`의 기본 구현체인 `ProviderManager`는 등록된 여러 `AuthenticationProvider` 중에서 현재 토큰을 처리할 수 있는 프로바이더(일반적으로 `DaoAuthenticationProvider`)를 찾아 인증을 위임한다.
5. `DaoAuthenticationProvider`는 `UserDetailsService`를 사용하여 데이터베이스 등에서 사용자 정보를 조회하고, `PasswordEncoder`를 사용하여 제출된 비밀번호와 저장된 암호화된 비밀번호를 비교한다.
6. 인증에 성공하면, `DaoAuthenticationProvider`는 사용자의 권한(Authorities, 예: 'ROLE_USER', 'ROLE_ADMIN') 정보까지 포함된, 완전히 인증된 `Authentication` 객체를 생성하여 반환한다.
7. 최종적으로 인증된 `Authentication` 객체는 `SecurityContext`에 저장되며, 이 `SecurityContext`는 `SecurityContextHolder`에 보관된다. `SecurityContextHolder`는 기본적으로 `ThreadLocal` 전략을 사용하여 현재 요청을 처리하는 스레드 내에서만 인증 정보를 유지시킨다.51


인증이 완료된 후, 사용자가 보호된 리소스에 접근하려고 할 때 인가 과정이 진행된다.

1. 필터 체인의 후반부에 위치한 `FilterSecurityInterceptor`가 요청을 가로챈다.
2. 이 필터는 `SecurityContextHolder`에서 현재 사용자의 `Authentication` 객체를 가져온다.
3. 요청된 URL과 애플리케이션의 보안 설정(예: `http.authorizeRequests().antMatchers("/admin/**").hasRole("ADMIN")`)을 비교한다.
4. `Authentication` 객체에 포함된 권한이 해당 리소스에 접근하기에 충분한지 확인한다. 만약 권한이 부족하면 `AccessDeniedException`을 발생시켜 접근을 거부한다.52

`SecurityContextHolder`와 `ThreadLocal`의 조합은 Spring Security 아키텍처의 핵심적인 설계 결정이다. 이 메커니즘 덕분에 보안 인프라와 비즈니스 로직이 완벽하게 분리된다. 애플리케이션의 어떤 계층(Controller, Service 등)에서도 `SecurityContextHolder.getContext().getAuthentication()`이라는 정적 메소드 호출만으로 현재 사용자의 정보와 권한을 얻을 수 있다. `HttpServletRequest` 객체를 모든 메소드에 일일이 전달할 필요가 없어지는 것이다. 이는 비즈니스 로직을 보안 관련 코드로부터 깨끗하게 유지시켜주는 강력한 디커플링(decoupling) 메커니즘이다.


Spring 프레임워크는 EJB의 복잡성을 해결하며 Java 개발에 혁신을 가져왔지만, 시간이 지나면서 프레임워크 자체의 생태계가 방대해짐에 따라 새로운 종류의 복잡성이 발생했다. 프로젝트를 시작하기 위해 처리해야 할 설정이 너무 많아진 것이다. 복잡한 XML 또는 Java 기반 설정, 수많은 라이브러리 간의 버전 호환성을 직접 관리해야 하는 의존성 문제, 그리고 별도의 웹 서버를 설치하고 WAR(Web Application Archive) 파일을 배포해야 하는 번거로움 등이 그것이다.57

Spring Boot는 이러한 Spring 프레임워크 자체의 복잡성을 해결하기 위해 탄생했다. 그 철학은 '설정보다 관례(Convention over Configuration)'로 요약되며, 개발자가 최소한의 설정만으로 독립 실행 가능한 상용 수준의 Spring 애플리케이션을 신속하게 구축할 수 있도록 돕는다.57


Spring Boot의 가장 핵심적인 기능으로, 애플리케이션의 클래스패스(classpath)와 개발자가 정의한 빈들을 분석하여 필요한 Spring 설정을 자동으로 구성해준다.

- **동작 원리:**
  1. Spring Boot 애플리케이션의 메인 클래스에 있는 `@SpringBootApplication` 어노테이션은 내부적으로 `@EnableAutoConfiguration`을 포함하고 있다.62
  2. `@EnableAutoConfiguration`이 활성화되면, Spring Boot는 클래스패스에 있는 모든 JAR 파일에서 `META-INF/spring.factories` 파일을 찾는다. 특히 `spring-boot-autoconfigure.jar` 내의 `spring.factories` 파일에는 수많은 `AutoConfiguration` 클래스들이 `org.springframework.boot.autoconfigure.EnableAutoConfiguration` 키에 매핑되어 있다.62
  3. Spring Boot는 이 목록에 있는 모든 `AutoConfiguration` 클래스를 로드하지만, 바로 빈으로 등록하지는 않는다. 각 `AutoConfiguration` 클래스에는 `@Conditional` 계열의 어노테이션(예: `@ConditionalOnClass`, `@ConditionalOnBean`, `@ConditionalOnMissingBean` 등)이 적용되어 있다.
  4. Spring 컨테이너는 이 조건들을 평가한다. 예를 들어, `DataSourceAutoConfiguration`은 클래스패스에 `DataSource` 클래스와 JDBC 드라이버가 존재하고(`@ConditionalOnClass`), 개발자가 직접 `DataSource` 타입의 빈을 등록하지 않았을 경우(`@ConditionalOnMissingBean`)에만 활성화되어 데이터베이스 연결에 필요한 빈들을 자동으로 등록해준다.62

이러한 조건부 자동 구성 덕분에 개발자는 필요한 라이브러리를 추가하는 것만으로 대부분의 설정을 끝낼 수 있으며, 필요 시 자신만의 설정을 추가하여 자동 구성을 손쉽게 오버라이드할 수 있다.


스타터는 특정 기능을 구현하는 데 필요한 의존성들의 묶음을 제공하는 편리한 디스크립터이다. 예를 들어, 웹 애플리케이션을 개발하기 위해 `pom.xml`이나 `build.gradle` 파일에 `spring-boot-starter-web` 하나만 추가하면, Spring MVC, 내장 톰캣 서버, Jackson(JSON 라이브러리) 등 웹 개발에 필요한 모든 라이브러리들이 서로 호환되는 버전으로 함께 추가된다.58 이는 개발자를 '의존성 지옥(dependency hell)'에서 해방시켜준다.


Spring Boot 애플리케이션은 기본적으로 Tomcat, Jetty, Undertow와 같은 웹 서버를 내장하고 있다. 그 결과, 애플리케이션은 별도의 외부 웹 서버에 배포할 필요 없이, `java -jar my-app.jar` 명령어 하나만으로 실행 가능한 독립적인 JAR 파일로 패키징될 수 있다.57 이는 개발, 테스트, 배포 과정을 극적으로 단순화시킨다.

결론적으로 Spring Boot는 Spring 프레임워크의 철학을 계승하고 완성한 결과물이다. Spring이 개발자를 EJB 컨테이너의 굴레에서 해방시켰다면, Spring Boot는 개발자를 Spring 프레임워크 자체의 상용구 설정과 환경 구축의 부담에서 해방시켰다. 이는 Ruby on Rails나 Node.js/Express와 같은 경쟁 프레임워크의 단순성에 대응하여, Spring 생태계가 스스로의 성공으로 인해 발생한 복잡성을 해결하려는 자정 노력의 산물이며, 개발자 생산성을 최우선으로 여기는 핵심 가치를 다시 한번 증명한 것이다.


전통적인 모놀리식(Monolithic) 아키텍처에서 벗어나, 독립적으로 배포하고 확장할 수 있는 작은 서비스들의 집합으로 애플리케이션을 구성하는 마이크로서비스 아키텍처(MSA)가 대두되면서 새로운 기술적 과제들이 등장했다. 분산 시스템 환경에서는 서비스 간의 동적 위치 탐색(Service Discovery), 장애 전파를 막기 위한 회복성(Fault Tolerance), 중앙화된 설정 관리, API 트래픽 관리 등 모놀리식 환경에서는 고려할 필요가 없었던 복잡한 문제들을 해결해야 한다.66

Spring Cloud는 이러한 분산 시스템의 문제들을 해결하기 위한 도구와 프레임워크를 제공하는 상위 프로젝트이다. 특정 기술을 새로 발명하기보다는, Netflix OSS와 같이 이미 산업계에서 검증된 강력한 오픈소스들을 Spring Boot의 개발 모델에 완벽하게 통합하여 제공하는 전략을 취한다.66


Spring Cloud는 MSA를 구축하는 데 필요한 다양한 패턴을 지원하는 컴포넌트들을 제공한다.

- **서비스 디스커버리 (Service Discovery) - Eureka:** MSA 환경에서는 서비스 인스턴스들이 동적으로 생성되고 소멸되기 때문에, 각 서비스의 IP 주소와 포트를 정적으로 관리하기 어렵다. 서비스 디스커버리 서버(예: Netflix Eureka)는 일종의 '전화번호부' 역할을 한다. 각 마이크로서비스는 시작할 때 Eureka 서버에 자신의 네트워크 위치 정보를 등록하고, 다른 서비스는 Eureka 서버에 질의하여 통신하고자 하는 서비스의 위치를 동적으로 찾아낸다.66
- **API 게이트웨이 (API Gateway) - Spring Cloud Gateway:** API 게이트웨이는 모든 외부 클라이언트 요청의 단일 진입점 역할을 한다. 클라이언트의 요청을 적절한 내부 마이크로서비스로 라우팅하고, 인증/인가, 로깅, 요청 제한(rate limiting)과 같은 공통적인 횡단 관심사를 중앙에서 처리한다. 이를 통해 내부 서비스의 복잡성을 숨기고 보안을 강화할 수 있다.66
- **중앙화된 설정 관리 (Centralized Configuration) - Spring Cloud Config:** 여러 마이크로서비스의 설정 정보(`application.properties` 또는 `application.yml`)를 Git과 같은 외부 저장소에서 중앙집중적으로 관리한다. 각 서비스는 시작 시 Config 서버로부터 자신의 설정 정보를 가져온다. 이를 통해 서비스 재배포 없이 설정을 동적으로 변경하고, 모든 서비스의 설정을 일관되게 관리할 수 있다.66
- **서킷 브레이커 (Circuit Breaker) - Resilience4j:** 분산 시스템에서는 하나의 서비스 장애가 연쇄적으로 다른 서비스의 장애를 유발하는 '연쇄 실패(cascading failure)'가 발생할 수 있다. 서킷 브레이커 패턴은 특정 서비스에 대한 호출이 계속 실패하면, 전기 회로의 차단기처럼 일시적으로 해당 서비스로의 모든 호출을 차단('open the circuit')한다. 이를 통해 장애가 전파되는 것을 막고, 장애가 발생한 서비스가 복구될 시간을 벌어주어 전체 시스템의 안정성과 회복성을 높인다.66
- **클라이언트 사이드 로드 밸런싱 (Client-Side Load Balancing) - Spring Cloud LoadBalancer:** 특정 서비스가 여러 인스턴스로 확장(scale-out)되었을 때, 해당 서비스를 호출하는 클라이언트(다른 마이크로서비스)가 서비스 디스커버리로부터 얻은 여러 인스턴스 목록을 보고 요청을 지능적으로 분산시키는 기술이다. 이를 통해 서비스의 가용성과 처리량을 높일 수 있다.66

Spring Cloud의 진정한 가치는 새로운 분산 시스템 알고리즘의 발명이 아니라, 이미 검증된 패턴들을 Spring Boot의 '마법'인 자동 구성(Auto-configuration)과 스타터(Starter) 의존성 모델로 완벽하게 통합했다는 데 있다. 예를 들어, 개발자는 `spring-cloud-starter-netflix-eureka-client` 의존성을 추가하고 `@EnableDiscoveryClient` 어노테이션 하나만 붙이면, 자신의 Spring Boot 애플리케이션이 자동으로 Eureka 서버에 등록되고 상태를 보고하는 복잡한 클라이언트 로직이 모두 자동으로 구성된다. 이 접근 방식은 복잡한 분산 시스템 기술의 진입 장벽을 극적으로 낮춤으로써, 평범한 애플리케이션 개발자도 견고한 마이크로서비스 아키텍처를 구축할 수 있도록 지원한다.



수십 년간 Java 웹 개발의 근간이었던 서블릿 API와 이를 기반으로 하는 Spring MVC는 근본적으로 동기(Synchronous), 블로킹(Blocking) I/O 모델에 기반한다. 이 모델은 '요청 당 스레드 하나(one-thread-per-request)' 방식으로 동작한다. 즉, 클라이언트로부터 요청이 들어오면 서블릿 컨테이너는 스레드 풀에서 스레드 하나를 할당하여 해당 요청의 처리를 맡긴다. 만약 이 스레드가 데이터베이스 조회나 외부 API 호출과 같은 I/O 작업을 수행하는 동안, 결과가 돌아올 때까지 해당 스레드는 아무 일도 하지 못하고 대기(block) 상태에 빠진다. 적은 수의 동시 요청 환경에서는 문제가 없지만, I/O 작업이 많은 고성능, 고시성(high-concurrency) 환경에서는 대기하는 스레드가 급증하여 시스템 자원을 비효율적으로 사용하고 성능 저하를 유발할 수 있다.69


이러한 한계를 극복하기 위해 반응형 프로그래밍 패러다임이 등장했다. 반응형 프로그래밍은 비동기(Asynchronous), 논블로킹(Non-blocking) 방식으로 동작하며, 데이터의 흐름(data stream)과 변화의 전파(propagation of change)를 중심으로 하는 이벤트 기반(event-driven) 프로그래밍 모델이다.69


Spring 프레임워크 5에서 새롭게 도입된 Spring WebFlux는 완전한 논블로킹, 반응형 웹 프레임워크이다. 이는 Spring MVC를 대체하는 것이 아니라, 특정 사용 사례(고시성, 스트리밍, I/O 집약적 작업)에 더 적합한 대안으로 제공된다.70

- **이벤트 루프 모델:** WebFlux는 요청 당 스레드 모델 대신, 적은 수의 고정된 스레드(이벤트 루프)를 사용하여 수많은 동시 연결을 처리한다. I/O 작업이 발생하면, 스레드는 결과를 기다리며 대기하는 대신 즉시 다른 이벤트 처리를 위해 반환된다. I/O 작업이 완료되면, 미리 등록된 콜백(callback) 함수가 이벤트 루프에 의해 실행되어 후속 작업을 처리한다. 이 방식을 통해 적은 수의 스레드로도 높은 동시성을 효율적으로 처리할 수 있다.69
- **Reactive Streams 명세:** WebFlux는 비동기 스트림 처리에 대한 표준 명세인 Reactive Streams를 기반으로 한다. 이 명세의 핵심 요소 중 하나는 '역압(Backpressure)'이다. 역압은 데이터를 소비하는 측(Consumer)이 자신이 처리할 수 있는 데이터의 양을 데이터를 생산하는 측(Producer)에게 능동적으로 알리는 메커니즘이다. 이를 통해 소비자가 감당할 수 없는 속도로 데이터가 밀려와 시스템이 불안정해지는 것을 방지한다.69


Spring은 Reactive Streams 명세를 구현한 라이브러리로 Project Reactor를 채택했다. Reactor는 두 가지 핵심적인 발행자(Publisher) 타입을 제공한다.

- **Mono`<T>`:** 0개 또는 1개의 데이터 항목만을 방출하는 스트림을 표현한다. 단일 결과를 반환하는 비동기 작업(예: 단일 객체를 반환하는 API 호출)에 적합하다.69
- **Flux`<T>`:** 0개부터 N개까지의 데이터 항목을 방출하는 스트림을 표현한다. 여러 개의 결과를 반환하는 비동기 작업(예: 데이터베이스에서 여러 행을 스트리밍하거나, 서버-전송 이벤트(SSE) 등)에 적합하다.69

WebFlux 기반의 컨트롤러 메소드는 실제 데이터 `T`를 반환하는 대신, `Mono<T>` 또는 `Flux<T>`를 반환하여 비동기적인 데이터의 흐름을 표현한다.

WebFlux의 등장은 Spring 생태계가 기존의 성공적인 모델을 버리지 않으면서도 완전히 새로운 프로그래밍 패러다임을 수용하고 통합할 수 있는 능력을 보여주는 명백한 증거이다. Spring MVC를 대체하는 대신, 병렬적인 기술 스택(Netty와 같은 논블로킹 서버 지원)을 처음부터 다시 구축하는 길을 택했다.70 그러면서도 

`@RestController`, `@GetMapping`과 같은 기존 어노테이션 기반의 개발 경험을 최대한 유지하여 개발자들이 새로운 반응형 모델에 쉽게 적응할 수 있도록 배려했다. 이러한 병렬 스택 접근 방식은 Spring의 아키텍처적 성숙도를 보여주며, 끊임없이 변화하는 기술 환경 속에서 지속적으로 관련성을 유지하며 발전할 수 있는 원동력이 되고 있다.



Spring은 지난 20여 년간 Java 엔터프라이즈 개발의 표준으로 자리매김하며 독보적인 생태계를 구축했다. 그 성공의 이면에는 명확한 강점과 함께 고려해야 할 약점도 존재한다.

- **강점:**
  - **포괄성 (Comprehensiveness):** 웹 개발, 데이터 접근, 보안, 배치 처리, 마이크로서비스 등 엔터프라이즈 애플리케이션 개발에 필요한 거의 모든 영역을 아우르는 포괄적인 솔루션을 제공한다.3
  - **거대한 생태계와 커뮤니티:** 방대한 문서, 수많은 예제, 활발한 커뮤니티 지원은 개발자가 마주치는 대부분의 문제에 대한 해결책을 쉽게 찾을 수 있도록 돕는다.
  - **유연성과 확장성:** IoC/DI와 AOP를 근간으로 하는 모듈식 설계는 높은 수준의 유연성과 확장성을 보장하여, 변화하는 요구사항에 쉽게 적응할 수 있다.3
  - **지속적인 혁신:** EJB의 대안으로 시작하여, Spring Boot를 통한 생산성 혁신, Spring Cloud를 통한 MSA 지원, WebFlux를 통한 반응형 패러다임 도입에 이르기까지, 시대의 변화에 맞춰 끊임없이 진화해왔다.
- **약점:**
  - **높은 학습 곡선 (Steep Learning Curve):** 생태계가 방대하고 기능이 많은 만큼, 초심자가 모든 개념을 이해하고 능숙하게 사용하기까지 상당한 시간과 노력이 필요할 수 있다.72
  - **과도한 추상화 ("Magic"):** Spring Boot의 자동 구성과 같은 기능은 매우 편리하지만, 내부 동작 원리를 이해하지 못하면 문제가 발생했을 때 해결하기 어려운 '마법'처럼 느껴질 수 있다.
  - **런타임 오버헤드:** 경량화를 지향하지만, 다양한 기능을 제공하기 위한 프레임워크 자체의 런타임 오버헤드는 매우 특화된 경량 프레임워크에 비해 상대적으로 클 수 있다.


Spring 생태계는 현재의 성공에 안주하지 않고, 클라우드 네이티브와 AI 시대를 선도하기 위한 혁신을 계속하고 있다. 최근의 기술 동향과 공식 발표들을 통해 Spring의 미래 방향성을 다음과 같이 전망할 수 있다.

- **GraalVM 네이티브 이미지 지원:** Spring은 AOT(Ahead-of-Time) 컴파일 기술에 막대한 투자를 하여 GraalVM을 공식적으로 지원한다. 이를 통해 Spring 애플리케이션을 JVM 없이 실행되는 네이티브 실행 파일로 컴파일할 수 있다. 그 결과, 수 밀리초 단위의 거의 즉각적인 시작 시간과 현저히 낮은 메모리 사용량을 달성할 수 있다. 이는 리소스 효율성이 극도로 중요한 서버리스(Serverless) 컴퓨팅이나 컨테이너 환경에서 Spring의 경쟁력을 극대화하는 핵심 기술이다.73
- **Project Loom과 가상 스레드 (Virtual Threads):** JDK에 공식적으로 도입된 가상 스레드는 기존의 동기/블로킹 방식 코드의 확장성 문제를 해결할 잠재력을 가지고 있다. Spring은 가상 스레드를 일급 시민으로 지원하여, 개발자들이 복잡한 반응형 프로그래밍으로 코드를 재작성하지 않고도 Spring MVC와 같은 익숙한 방식으로 I/O 집약적인 애플리케이션의 동시성을 대폭 향상시킬 수 있는 길을 열어주고 있다. 이는 '두 세계의 장점(익숙한 코드, 높은 확장성)'을 모두 취하는 실용적인 접근 방식이다.73
- **서버리스 및 스케일-투-제로 (Scale-to-Zero):** AWS Lambda와 같은 서버리스 플랫폼에서 Spring 애플리케이션을 보다 효율적으로 실행하기 위한 노력이 계속되고 있다. 이는 네이티브 이미지 기술과 결합하여, 평소에는 비용을 거의 발생시키지 않다가 필요할 때만 즉시 확장되는 '스케일-투-제로' 아키텍처 구현을 용이하게 할 것이다.73
- **Spring AI:** 최근 공개된 Spring AI 프로젝트는 애플리케이션에 대규모 언어 모델(LLM)과 같은 인공지능 서비스를 통합하는 과정을 단순화하려는 전략적 움직임을 보여준다. Spring Data가 데이터베이스 접근을 추상화했듯이, Spring AI는 다양한 AI 모델과의 상호작용을 위한 일관된 추상화 계층을 제공하여 개발자가 AI 기반 애플리케이션을 더 쉽게 구축할 수 있도록 지원할 것이다.73


Spring의 지난 20년은 Java 엔터프라이즈 개발의 역사 그 자체였다. 그 성공의 핵심 동력은 '개발자 생산성'이라는 확고한 철학을 바탕으로, 현실의 문제에 대한 실용적인 해결책을 제시하며 끊임없이 진화해 온 놀라운 적응력에 있다. EJB의 복잡성을 POJO의 단순함으로 대체하며 시작된 여정은, 마이크로서비스와 반응형 시스템이라는 새로운 패러다임을 수용하고, 이제는 클라우드 네이티브와 AI라는 미래를 향해 나아가고 있다. Spring은 단순히 과거의 유산이 아니라, Java 생태계의 미래를 지속적으로 정의하고 형성해 나가는 현재 진행형의 혁신임을 증명하고 있다.4


1. [스프링] 스프링의 역사 - 대니기록, 8월 15, 2025에 액세스, https://kdohyeon.tistory.com/46
2. 스프링의 역사 | 스프링은 왜 탄생했는가, 8월 15, 2025에 액세스, https://gaebalsogi.tistory.com/37
3. 스프링 프레임워크는 왜 등장했을까?, 8월 15, 2025에 액세스, https://psvm.kr/posts/spring/history-of-spring
4. 요즘 개발자들 필수 키워드 : '스프링'의 중요성과 학습 전략, 8월 15, 2025에 액세스, https://kernel360.co.kr/springdev_story
5. [Spring] 스프링 IoC와 DI란 무엇인가? (제어의 역전과 의존성 주입), 8월 15, 2025에 액세스, https://ittrue.tistory.com/212
6. [Spring] IoC(제어의 역전)와 DI(의존성 주입)의 완벽 이해 - 개발하는 새우, 8월 15, 2025에 액세스, [https://developshrimp.com/entry/Spring-IoC%EC%A0%9C%EC%96%B4%EC%9D%98-%EC%97%AD%EC%A0%84%EC%99%80-DI%EC%9D%98%EC%A1%B4%EC%84%B1-%EC%A3%BC%EC%9E%85%EC%9D%98-%EC%99%84%EB%B2%BD-%EC%9D%B4%ED%95%B4](https://developshrimp.com/entry/Spring-IoC제어의-역전와-DI의존성-주입의-완벽-이해)
7. [Spring] 스프링 컨테이너 & DI(Dependency Injection) - 인포꿀팁 - 티스토리, 8월 15, 2025에 액세스, [https://back-end-developer.tistory.com/entry/Spring-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88-DIDependency-Injection](https://back-end-developer.tistory.com/entry/Spring-스프링-컨테이너-DIDependency-Injection)
8. [Spring] IoC(제어의 역전)와 DI(의존성 주입) 그리고 Spring - 오늘의 개발 - 티스토리, 8월 15, 2025에 액세스, https://oneul-losnue.tistory.com/364
9. [DI 구현하기] 의존성 주입이 필요한 이유와 DI 컨테이너의 탄생 ..., 8월 15, 2025에 액세스, https://ksabs.tistory.com/262
10. 6/17 (금) Spring Framework - DI(Dependency Injection) 1️⃣ Spring Container와 Bean, 8월 15, 2025에 액세스, https://wjcodding.tistory.com/43
11. [Spring] Spring Framework Module (스프링 프레임워크 모듈), 8월 15, 2025에 액세스, [https://maenco.tistory.com/entry/Spring-Spring-Framework-Module-%EC%8A%A4%ED%94%84%EB%A7%81-%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC-%EB%AA%A8%EB%93%88](https://maenco.tistory.com/entry/Spring-Spring-Framework-Module-스프링-프레임워크-모듈)
12. [Spring] 스프링 프레임워크 주요 모듈 알아보기 - jay_velog, 8월 15, 2025에 액세스, https://reeeemind.tistory.com/90
13. [Spring Framework] IoC Container, ApplicationContext와 Bean - 개발자의 기록습관 - 티스토리, 8월 15, 2025에 액세스, https://ict-nroo.tistory.com/120
14. [Spring] IoC, DI, Spring Container, Bean 완벽하게 알아보자 - 메모장 희망편 - 티스토리, 8월 15, 2025에 액세스, https://esperer.tistory.com/6
15. [Spring] 스프링 IoC컨테이너 ApplicationContext - 차곡차곡. - 티스토리, 8월 15, 2025에 액세스, [https://deveun.tistory.com/entry/%ED%86%A0%EB%B9%84-%EC%8A%A4%ED%94%84%EB%A7%81-IoC%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88-ApplicationContext](https://deveun.tistory.com/entry/토비-스프링-IoC컨테이너-ApplicationContext)
16. [Spring] Spring에서 Bean은 어떤 자료구조로 관리될까요? - 기러기는 기록기록 - 티스토리, 8월 15, 2025에 액세스, https://kookiencream.tistory.com/117
17. 토비의스프링 vol.1 - 1장, 오브젝트와 의존관계 - 개발 공부할래? - 티스토리, 8월 15, 2025에 액세스, https://loopstudy.tistory.com/162
18. [스프링 핵심 원리 - 기본편] 3. 스프링 컨테이너와 스프링 빈, 8월 15, 2025에 액세스, https://daegwonkim.tistory.com/280
19. Spring Container and Bean | devHTak, 8월 15, 2025에 액세스, https://devhtak.github.io/spring/2021/03/09/Spring_Container.html
20. Spring Bean이란 - velog, 8월 15, 2025에 액세스, [https://velog.io/@smc2315/Spring-Bean%EC%9D%B4%EB%9E%80](https://velog.io/@smc2315/Spring-Bean이란)
21. Spring - Bean 초기화 및 생명주기 - I's Story - 티스토리, 8월 15, 2025에 액세스, https://isstory83.tistory.com/93
22. Spring Bean life cycle - joyfulviper님의 학습일기 - 티스토리, 8월 15, 2025에 액세스, https://joyfulviper.tistory.com/2
23. [Spring] 빈 생명 주기 콜백 (Bean LifeCycle) @PostConstruct / @PreDestroy - HS_dev_log, 8월 15, 2025에 액세스, https://innovation123.tistory.com/213
24. Spring Container와 Bean의 Lifecycle - 캔두 - 티스토리, 8월 15, 2025에 액세스, https://yein-lee.tistory.com/39
25. [Spring] 빈 생명주기와 빈 스코프 - velog, 8월 15, 2025에 액세스, [https://velog.io/@semi-cloud/Spring-%EB%B9%88-%EC%83%9D%EB%AA%85%EC%A3%BC%EA%B8%B0%EC%99%80-%EB%B9%88-%EC%8A%A4%EC%BD%94%ED%94%84](https://velog.io/@semi-cloud/Spring-빈-생명주기와-빈-스코프)
26. [Spring] 스프링 AOP (Spring AOP) 총정리 : 개념, 프록시 기반 AOP ..., 8월 15, 2025에 액세스, [https://engkimbs.tistory.com/entry/%EC%8A%A4%ED%94%84%EB%A7%81AOP](https://engkimbs.tistory.com/entry/스프링AOP)
27. [Spring] AOP (관점 지향 프로그래밍) - Hyem.log, 8월 15, 2025에 액세스, https://hyemin-jang.github.io/Spring/2-aop/
28. [Spring] AOP: 횡단 관심사 쉽게 이해하기 - 오늘도 개발중입니다 - 티스토리, 8월 15, 2025에 액세스, https://curiousjinan.tistory.com/entry/spring-aop-understand
29. engkimbs.tistory.com, 8월 15, 2025에 액세스, [https://engkimbs.tistory.com/entry/%EC%8A%A4%ED%94%84%EB%A7%81AOP#:~:text=AOP%EB%8A%94%20Aspect%20Oriented%20Programming,%EC%9C%BC%EB%A1%9C%20%EA%B0%81%EA%B0%81%20%EB%AA%A8%EB%93%88%ED%99%94%ED%95%98%EA%B2%A0%EB%8B%A4%EB%8A%94%20%EA%B2%83%EC%9D%B4%EB%8B%A4.](https://engkimbs.tistory.com/entry/스프링AOP#:~:text=AOP는 Aspect Oriented Programming,으로 각각 모듈화하겠다는 것이다.)
30. 스프링 부트(Spring Boot) - AOP와 트랜잭션(Transaction) - Let's develop - 티스토리, 8월 15, 2025에 액세스, https://congsong.tistory.com/25
31. JDK Dynamic Proxy와 CGLIB - Scribbleeeeee, 8월 15, 2025에 액세스, https://jiseunghyeon.com/jdk-dynamic-proxy-cglib
32. [Spring] AOP와 JDK Dynamic Proxy, CGLIB - 느리더라도 꾸준하게 - 티스토리, 8월 15, 2025에 액세스, https://steady-coding.tistory.com/608
33. [Spring] Spring의 AOP 프록시 구현 방법(JDK 동적 프록시, CGLib 프록시)과 @EnableAspectJAutoProxy의 proxyTargetClass - (3/3) - 망나니개발자, 8월 15, 2025에 액세스, https://mangkyu.tistory.com/175
34. [간단정리] JDK Dynamic Proxy vs CGLib Proxy - 넌 잘하고 있어, 8월 15, 2025에 액세스, https://hahahoho5915.tistory.com/76
35. [Spring/AOP] JDK Dynamic Proxy vs CGLIB Proxy - 연로그 - 티스토리, 8월 15, 2025에 액세스, https://yeonyeon.tistory.com/289
36. [Spring] 스프링 프레임워크의 주요 모듈들을 알아보자 / 아웃스탠딩보이 블로그, 8월 15, 2025에 액세스, https://outstanding1301.github.io/dev/2021/01/09/mastering-spring5-2/
37. Spring MVC Architecture - 표준프레임워크, 8월 15, 2025에 액세스, https://www.egovframe.go.kr/wiki/doku.php?id=egovframework:rte:ptl:spring_mvc_architecture
38. Spring MVC구조 - 내부HTTP요청처리과정 - 개발 아카이브, 8월 15, 2025에 액세스, https://kimdeveloper.tistory.com/46
39. [TIL] Spring MVC 요청 처리 과정 - Yeonnnnny - 티스토리, 8월 15, 2025에 액세스, https://yeonnnnny.tistory.com/m/159
40. DispatcherServlet 파헤치기 - velog, 8월 15, 2025에 액세스, [https://velog.io/@kimsei1124/DispatcherServlet-%ED%8C%8C%ED%97%A4%EC%B9%98%EA%B8%B0](https://velog.io/@kimsei1124/DispatcherServlet-파헤치기)
41. 요청이 왔을 때 스프링 MVC 내부적 흐름 과정 (DispatcherServlet 중심), 8월 15, 2025에 액세스, https://programmer-may.tistory.com/161
42. Spring DispatcherServlet(디스패처서블릿) 개념부터 동작 과정까지, 8월 15, 2025에 액세스, https://zzang9ha.tistory.com/441
43. [Spring] HTTP Request 를 처리하는 과정 - DispatcherServlet 원리 - 김예건 - 티스토리, 8월 15, 2025에 액세스, https://ibocon.tistory.com/208
44. [Java] Spring 프레임워크에서의 MVC 패턴과 레이어드 아키텍처 - 코딩하는 주노 이야기, 8월 15, 2025에 액세스, [https://co-no.tistory.com/entry/Java-Spring-%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC%EC%97%90%EC%84%9C%EC%9D%98-MVC-%ED%8C%A8%ED%84%B4%EA%B3%BC-Service-DAO-DTO-VO-%EA%B0%9C%EB%85%90](https://co-no.tistory.com/entry/Java-Spring-프레임워크에서의-MVC-패턴과-Service-DAO-DTO-VO-개념)
45. [Spring] Spring type(스프링의 종류) - 천천히 꾸준하게 - 티스토리, 8월 15, 2025에 액세스, https://yooseong12.tistory.com/37
46. [JPA] JPA, Spring Data JPA의 내부 동작 원리 알아보기 - 성장하는 성하 Blog - 티스토리, 8월 15, 2025에 액세스, https://ksh-coding.tistory.com/153
47. Spring Data JPA와 JpaRepository 인터페이스의 공통 ... - 경호의 공부방, 8월 15, 2025에 액세스, [https://ykh6242.tistory.com/entry/%EC%8A%A4%ED%94%84%EB%A7%81-%EB%8D%B0%EC%9D%B4%ED%84%B0-JPA%EC%99%80-JpaRepository-%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4%EC%9D%98-%EA%B3%B5%ED%86%B5-%EA%B8%B0%EB%8A%A5-%EC%A0%95%EB%A6%AC](https://ykh6242.tistory.com/entry/스프링-데이터-JPA와-JpaRepository-인터페이스의-공통-기능-정리)
48. Spring Data JPA - 코딩하는 털보 - 티스토리, 8월 15, 2025에 액세스, https://rockintuna.tistory.com/22
49. spring-data-jpa save 동작 원리 - velog, 8월 15, 2025에 액세스, [https://velog.io/@rainmaker007/spring-data-jpa-save-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC](https://velog.io/@rainmaker007/spring-data-jpa-save-동작-원리)
50. [Java/JPA] Spring Boot Data JPA 이해하기 -3 : JpaRepository 활용 방법(Query Method, @Query, NamedQuery) - Contributor9 - 티스토리, 8월 15, 2025에 액세스, https://adjh54.tistory.com/481
51. [Spring Security] 스프링 시큐리티 - 인증 프로세스 - velog, 8월 15, 2025에 액세스, [https://velog.io/@impala/Spring-Security-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0-%EC%9D%B8%EC%A6%9D-%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4](https://velog.io/@impala/Spring-Security-스프링-시큐리티-인증-프로세스)
52. Spring Security의 구조, 인증 인가 처리과정 - 우당탕탕 개발, 8월 15, 2025에 액세스, https://anythingis.tistory.com/112
53. [Spring Security] 스프링 시큐리티와 FilterChain - 개발 메모용 블로그 - 티스토리, 8월 15, 2025에 액세스, https://memodayoungee.tistory.com/134
54. JWT를 구현하기전, 인증/인가 동작 원리: FilterChain을 이해해보자! - JAN's History, 8월 15, 2025에 액세스, https://shinjaeeun.tistory.com/104
55. [Spring] Spring Security를 이용한 JWT 기반 인증 및 인가 설정 - chaewss - 티스토리, 8월 15, 2025에 액세스, https://chaewsscode.tistory.com/244
56. 쉽게 풀어 쓴 Spring Security 필터링 매커니즘, 8월 15, 2025에 액세스, https://seongsu.me/post/20
57. [스프링 부트(SpringBoot)] 스프링 부트 소개 - GDNGY의 함께 만들어가는 테크노트 (Technote), 8월 15, 2025에 액세스, https://gdngy.tistory.com/86
58. Spring Boot 의 탄생과 장점 - velog, 8월 15, 2025에 액세스, [https://velog.io/@wawakdev/Spring-Spring-Boot-%EC%9D%98-%ED%83%84%EC%83%9D%EA%B3%BC-%EC%9E%A5%EC%A0%90](https://velog.io/@wawakdev/Spring-Spring-Boot-의-탄생과-장점)
59. [Spring] SpringBoot의 등장배경 및 장점 - Better Programming, 8월 15, 2025에 액세스, https://ar-tec.tistory.com/131
60. Spring VS SpringBoot - velog, 8월 15, 2025에 액세스, https://velog.io/@ksu9704/Spring-VS-SpringBoot
61. 봄(Spring)과 스프링 부트(Spring Boot) 차이: 어떤 프레임워크를 선택해야 할까?, 8월 15, 2025에 액세스, https://hiteksoftware.co.kr/blog/difference-between-spring-and-spring-boot/
62. Spring Boot AutoConfiguration 자동 설정, 원리 - TheWing - 티스토리, 8월 15, 2025에 액세스, https://sujl95.tistory.com/58
63. [spring] SpringBoot의 AutoConfiguration 동작방식 - JH-Labs - 티스토리, 8월 15, 2025에 액세스, https://jh-labs.tistory.com/337
64. Spring Boot 핵심 기능 3 - 자동 구성(Auto Configuration) - Reference - 티스토리, 8월 15, 2025에 액세스, https://hanseom.tistory.com/333
65. SpringBoot AutoConfiguration을 대하는 자세 - Tecoble, 8월 15, 2025에 액세스, https://tecoble.techcourse.co.kr/post/2021-10-14-springboot-autoconfiguration/
66. [Spring Cloud] Spring Cloud란 무엇일까 (마이크로서비스 아키텍처 ..., 8월 15, 2025에 액세스, https://pixx.tistory.com/273
67. [Spring Cloud] MSA : (1) 마이크로서비스의 이해와 Spring Cloud 구조 - 데굴데굴 개발자의 기록, 8월 15, 2025에 액세스, https://sjh9708.tistory.com/119
68. Spring Cloud기반 마이크로서비스개발 - KOSTA EDU, 8월 15, 2025에 액세스, https://kosta.oopy.io/a3556c62-2bea-4d33-8f9c-4d5eb6e7a2c7
69. Spring의 Webflux - velog, 8월 15, 2025에 액세스, [https://velog.io/@qwerty1434/Spring%EC%9D%98-Webflux](https://velog.io/@qwerty1434/Spring의-Webflux)
70. 스프링 WebFlux로 반응형 웹 애플리케이션 개발 - 재능넷, 8월 15, 2025에 액세스, https://www.jaenung.net/tree/1510
71. [Spring Boot Webflux] 흐름 및 주요 특징 - 민진이네 개발노트, 8월 15, 2025에 액세스, https://minjin-note.tistory.com/19
72. 스프링 프레임워크(Spring Framework)의 탄생배경, 장단점 - Anything Is Possible - 티스토리, 8월 15, 2025에 액세스, https://hi5june.tistory.com/41
73. My SpringOne 2023 Recap, 8월 15, 2025에 액세스, https://spring.io/blog/2023/08/29/my-springone-2023-recap/
74. The Road to Spring Boot 4: What's Here, What's Next, and How to Prepare | by Gagan Jain, 8월 15, 2025에 액세스, https://medium.com/@gaganjain0302/the-road-to-spring-boot-4-whats-here-what-s-next-and-how-to-prepare-2cb85d7f6110
75. Runtime efficiency with Spring (today and tomorrow), 8월 15, 2025에 액세스, https://spring.io/blog/2023/10/16/runtime-efficiency-with-spring
76. The Future of Spring Applications: Leveraging Native Execution through Spring Native, 8월 15, 2025에 액세스, https://adria-bt.com/en/the-future-of-spring-applications-leveraging-native-execution-through-spring-native/
77. SpringOne 2023 Recap - Dan Vega - Spring Developer Advocate, YouTuber and Lifelong Learner, 8월 15, 2025에 액세스, https://www.danvega.dev/newsletter/spring-one-2023-recap/
78. From Spring Framework 6.2 to 7.0, 8월 15, 2025에 액세스, https://spring.io/blog/2024/10/01/from-spring-framework-6-2-to-7-0/
79. The Path Towards Spring Boot Native Applications on GraalVM - YouTube, 8월 15, 2025에 액세스, https://www.youtube.com/watch?v=DyWlThGX6Ic
80. SpringOne > News > Page #1 - InfoQ, 8월 15, 2025에 액세스, https://www.infoq.com/SpringOne/news/
81. Spring Spotlight: Sustainable Evolution with Spring (SpringOne 2024) - YouTube, 8월 15, 2025에 액세스, https://www.youtube.com/watch?v=7S6M8H2hz6w
82. 프레임워크의 특징과 사용법 (자바 스프링 예제) - 잡학서고, 8월 15, 2025에 액세스, https://www.gklibrarykor.com/1432/

