# SOLID 원칙

## 서론

객체 지향 설계(Object-Oriented Design, OOD)의 품질을 평가하고 향상시키기 위한 수많은 방법론이 존재하지만, 그중에서도 SOLID 원칙은 현대 소프트웨어 공학의 가장 근본적인 초석으로 자리 잡고 있다. SOLID는 다섯 가지 핵심 설계 원칙인 단일 책임 원칙(Single Responsibility Principle), 개방-폐쇄 원칙(Open-Closed Principle), 리스코프 치환 원칙(Liskov Substitution Principle), 인터페이스 분리 원칙(Interface Segregation Principle), 그리고 의존성 역전 원칙(Dependency Inversion Principle)의 머리글자를 따서 만든 두문자어(mnemonic acronym)이다.1 이 원칙들은 단순히 개별적인 코딩 지침의 집합을 넘어, 시간이 지나도 이해하기 쉽고(understandable), 변화에 유연하며(flexible), 유지보수가 용이한(maintainable) 소프트웨어 시스템을 구축하기 위한 핵심 철학을 담고 있다.3

이 원칙들의 체계화는 소프트웨어 장인정신(Software Craftsmanship)의 저명한 주창자인 로버트 C. 마틴(Robert C. Martin, "Uncle Bob")에 의해 2000년 그의 논문 "Design Principles and Design Patterns"에서 이루어졌다.1 마틴은 성공적인 소프트웨어는 필연적으로 진화해야 하지만, 그 과정에서 점차 복잡성이 증가하고 결국 경직성(rigidity), 취약성(fragility), 부동성(immobility), 점착성(viscosity)과 같은 특성을 띠게 되는 '소프트웨어 부패(software rot)' 현상을 목격했다.1 SOLID 원칙은 바로 이러한 소프트웨어의 고질적인 문제들을 해결하고, 시간이 지나도 건강하고 적응력 있는 시스템을 유지하기 위한 처방전으로 제시되었다. 'SOLID'라는 약어 자체는 2004년경 마이클 페더스(Michael Feathers)에 의해 처음 만들어져 널리 알려지게 되었다.3

본 보고서는 SOLID의 다섯 가지 원칙 각각에 대한 이론적 깊이를 탐구하는 것을 목표로 한다. 각 원칙의 정의와 철학적 기반을 명확히 하고, 실제 코드 예제를 통해 원칙의 위반 사례와 이를 해결하기 위한 리팩토링 과정을 상세히 분석할 것이다. 더 나아가, 이 원칙들이 어떻게 서로 유기적으로 상호작용하여 시너지 효과를 내는지, 그리고 객체 지향 패러다임을 넘어 함수형 프로그래밍과 마이크로서비스 아키텍처와 같은 현대적인 패러다임에 어떻게 적용될 수 있는지를 고찰한다. 마지막으로는 SOLID 원칙에 대한 비판적 시각을 견지하며, 과잉 공학의 위험성을 경고하고 다른 설계 원칙과의 비교를 통해 실용적인 적용을 위한 균형점을 제시하고자 한다.

보고서의 본격적인 논의에 앞서, 독자의 이해를 돕기 위해 SOLID 원칙의 핵심 개념과 목표를 요약한 표를 아래와 같이 제시한다.

| 원칙 (Principle)          | 정의 (Definition)                                            | 핵심 목표 (Core Objective)                                   |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **S**ingle Responsibility | 클래스는 변경되어야 할 단 하나의 이유만을 가져야 한다.       | 응집성 (Cohesion), 유지보수성 (Maintainability)              |
| **O**pen-Closed           | 소프트웨어 개체는 확장에 대해 열려 있고, 수정에 대해 닫혀 있어야 한다. | 확장성 (Extensibility), 안정성 (Stability)                   |
| **L**iskov Substitution   | 하위 타입은 상위 타입의 객체를 대체할 수 있어야 한다.        | 신뢰성 (Reliability), 다형성 (Polymorphism)                  |
| **I**nterface Segregation | 클라이언트는 자신이 사용하지 않는 인터페이스에 의존하도록 강요되어서는 안 된다. | 결합도 감소 (Decoupling), 모듈성 (Modularity)                |
| **D**ependency Inversion  | 상위 모듈은 하위 모듈에 의존해서는 안 되며, 둘 다 추상화에 의존해야 한다. | 결합도 감소 (Decoupling), 유연성 (Flexibility), 테스트 용이성 (Testability) |

이 표는 각 원칙이 고립된 규칙이 아님을 명확히 보여준다. 오히려 이들은 소프트웨어 설계의 근본적인 두 가지 목표, 즉 **응집도 증가(Increasing Cohesion)**와 **결합도 감소(Decreasing Coupling)**를 향해 유기적으로 수렴한다. 단일 책임 원칙은 모듈 내부의 응집도를 높이는 데 직접적으로 초점을 맞추는 반면, 나머지 네 원칙(OCP, LSP, ISP, DIP)은 다양한 메커니즘을 통해 모듈 간의 결합도를 낮추는 데 집중한다. 이러한 거시적인 관점을 견지할 때, 독자는 이어지는 상세 분석을 통해 각 원칙이 어떻게 전체적인 설계 품질 향상에 기여하는지에 대한 구조적인 이해를 얻을 수 있을 것이다.

## 1.  SOLID 원칙의 개별 심층 분석

### 1.1  단일 책임 원칙 (SRP: Single Responsibility Principle)

#### 1.1.1 정의와 철학적 기반

단일 책임 원칙(SRP)은 "클래스는 변경되어야 할 단 하나의 이유만을 가져야 한다(A class should have one, and only one, reason to change)"는 로버트 C. 마틴의 간결한 정의로 요약된다.1 이 원칙의 핵심은 '책임'의 단위를 올바르게 이해하는 데 있다. 이는 단순히 클래스가 하나의 메서드만 가져야 한다는 기계적인 해석이 아니다. 여기서 '변경의 이유'는 소프트웨어의 특정 기능이나 비즈니스 로직을 책임지는 단일 '액터(actor)' 또는 '컨텍스트(context)'와 밀접하게 연관된다.7 예를 들어, 재무팀과 인사팀은 서로 다른 액터이며, 이들의 요구사항 변경은 서로 다른 이유에 해당한다. 만약 하나의 클래스가 두 팀 모두의 요구사항을 처리한다면, 그 클래스는 두 가지 변경 이유를 가지게 되어 SRP를 위반하게 된다.

따라서 SRP의 근본적인 철학은 관련 있는 것들을 한데 모으고, 관련 없는 것들은 분리함으로써 **높은 응집도(High Cohesion)**를 달성하는 것이다.7 응집도가 높은 클래스는 그 목적이 명확하고, 이해하기 쉬우며, 변경의 파급 효과를 해당 클래스 내부로 국소화시킬 수 있다. 이는 관심사의 분리(Separation of Concerns)라는 더 넓은 소프트웨어 공학 원리와도 깊은 관련이 있다.8

#### 1.1.2 SRP 위반의 증상과 폐해

SRP를 위반한 클래스는 여러 가지 부정적인 증상을 나타내며, 이는 시스템의 건강성을 심각하게 저해한다.

- **다중 책임 클래스와 낮은 응집도:** 가장 흔한 위반 사례는 하나의 클래스가 서로 관련 없는 여러 책임을 동시에 수행하는 것이다. 예를 들어, 사용자 인증(authentication), 사용자 프로필 관리(profile management), 그리고 활동 로깅(activity logging)을 모두 처리하는 `UserAccountManager` 클래스를 생각해보자.9 인증 정책의 변경, 프로필 데이터 구조의 변경, 로깅 포맷의 변경은 각각 독립적인 이유로 발생하지만, 이 모든 변경이 

  `UserAccountManager`라는 단일 클래스에 집중된다. 이는 클래스의 응집도를 현저히 떨어뜨린다.

- **높은 결합도와 예상치 못한 부작용:** 책임이 분리되지 않은 코드는 내부적으로 서로 다른 기능들이 강하게 결합(tightly coupled)될 수밖에 없다. `UserAccountManager` 예제에서 로깅 로직을 수정하는 과정에서 실수로 인증 로직에 영향을 미칠 수 있다. 이처럼 하나의 변경이 전혀 예상치 못한 부분에서 버그를 유발하는 부작용(side effects)의 위험이 크게 증가한다.6

- **테스트 및 유지보수의 어려움:** 단일 책임을 가진 클래스는 그 책임에만 집중하여 단위 테스트를 작성하기 매우 용이하다.3 반면, 여러 책임을 가진 클래스는 테스트 케이스가 복잡해지고, 모든 책임의 조합을 고려해야 하므로 테스트 커버리지를 확보하기 어렵다. 또한, 새로운 개발자가 코드를 이해하고 수정하는 데 훨씬 더 많은 시간과 노력이 소요된다.4

- **병합 충돌(Merge Conflicts):** 팀 단위 개발 환경에서 SRP 위반은 버전 관리 시스템의 병합 충돌을 빈번하게 유발한다. 예를 들어, 한 개발자는 인증 기능을 수정하고 다른 개발자는 로깅 기능을 수정하기 위해 동일한 `UserAccountManager` 파일을 변경했다고 가정해보자. 이 두 개발자의 작업은 개념적으로는 완전히 독립적이지만, 물리적으로는 동일한 파일을 수정했기 때문에 병합 과정에서 충돌이 발생할 확률이 매우 높다. SRP를 준수했다면 이들은 애초에 서로 다른 파일을 수정했을 것이므로 충돌 자체가 발생하지 않았을 것이다.5

#### 1.1.3 리팩토링: 책임의 분리

SRP를 준수하기 위한 리팩토링의 핵심은 클래스가 가진 여러 책임을 식별하고, 각 책임을 독립된 클래스로 분리하는 것이다.

다음은 고객 정보(Customer entity)와 데이터베이스 저장 로직(Persistence logic)이 혼재된 클래스의 위반 사례와 해결책이다.7

**위반 전 코드 (Java):**

```Java
// SRP 위반: 고객 데이터와 데이터베이스 로직이 결합됨
class Customer {
    private int id;
    private String name;

    // 고객 관련 getter/setter...

    public void saveToDatabase() {
        // 데이터베이스 연결 및 저장 로직
        System.out.println("Saving customer " + name + " to the database.");
    }
}
```

위 `Customer` 클래스는 두 가지 변경 이유를 가진다. 첫째, 고객의 속성(예: 주소 추가)이 변경될 때. 둘째, 데이터 저장 방식(예: 데이터베이스에서 파일 시스템으로 변경)이 변경될 때. 이는 명백한 SRP 위반이다.

**리팩토링 후 코드 (Java):**

```Java
// SRP 준수: 고객 데이터 클래스
class Customer {
    private int id;
    private String name;

    // 고객 관련 getter/setter...
}

// SRP 준수: 고객 저장소 클래스
class CustomerRepository {
    public void save(Customer customer) {
        // 데이터베이스 연결 및 저장 로직
        System.out.println("Saving customer " + customer.getName() + " to the database.");
    }
}
```

리팩토링을 통해 `Customer` 클래스는 순수하게 고객 데이터를 표현하는 책임만을 가지게 되었고, 데이터 영속성과 관련된 책임은 `CustomerRepository` 클래스로 완전히 분리되었다. 이제 고객 데이터 모델의 변경은 `Customer` 클래스에만 영향을 미치고, 저장 방식의 변경은 `CustomerRepository` 클래스에만 영향을 미친다. 이로써 각 클래스는 단 하나의 변경 이유만을 가지게 되어 SRP를 준수하게 된다.

이처럼 SRP를 충실히 따르는 것은 단순히 코드를 나누는 행위를 넘어, 시스템의 변경 가능성을 예측하고 그 영향을 효과적으로 격리하는 중요한 설계 활동이다. SRP는 다른 모든 SOLID 원칙을 적용하기 위한 견고한 기반을 제공한다. 한 클래스가 여러 책임을 지게 되면, 그 클래스를 수정하지 않고 확장하는 것(OCP)은 거의 불가능해진다. 예를 들어, 보고서 생성 시 데이터 조회, 포맷팅, 출력을 모두 담당하는 `ReportGenerator` 클래스가 있다면, 새로운 출력 형식(예: CSV)을 추가하기 위해서는 기존 클래스의 코드를 직접 수정해야만 한다. 이는 OCP를 정면으로 위반하는 행위이다. 반면, SRP를 준수하여 책임을 `DataFetcher`, `ReportFormatter`, `ReportPrinter` 등으로 분리했다면, 새로운 `CSVFormatter` 클래스를 추가하고 이를 `ReportPrinter`에 주입하는 것만으로 기능 확장이 가능해진다.10 이처럼 SRP의 준수는 OCP 준수를 위한 강력한 전제 조건이 되며, 변경의 영향을 국소화시켜 시스템 전체의 안정성을 높이는 첫걸음이 된다.

### 1.2  개방-폐쇄 원칙 (OCP: Open-Closed Principle)

#### 1.2.1 정의와 철학적 기반

개방-폐쇄 원칙(OCP)은 1988년 버트런드 마이어(Bertrand Meyer)에 의해 처음 제시된 개념으로, "소프트웨어 개체(클래스, 모듈, 함수 등)는 확장에 대해서는 열려 있어야 하고(open for extension), 수정에 대해서는 닫혀 있어야 한다(closed for modification)"고 정의된다.3 이 원칙의 핵심은 시스템에 새로운 기능을 추가하거나 기존 기능의 동작을 변경해야 할 때, 이미 안정적으로 동작하고 있는 기존 코드를 수정하는 것을 최소화하거나 완전히 배제해야 한다는 것이다. 기존 코드를 수정하는 행위는 언제나 새로운 버그를 유발할 잠재적 위험을 내포하기 때문이다.4 대신, 시스템은 새로운 기능을 쉽게 '추가'할 수 있는 유연한 구조를 가져야 한다. OCP는 변화에 안정적인 시스템을 구축하여 장기적인 유지보수 비용을 절감하는 것을 궁극적인 목표로 한다.3

#### 1.2.2 구현 메커니즘: 추상화와 다형성

OCP를 달성하는 가장 핵심적인 기술적 메커니즘은 **추상화(Abstraction)**와 **다형성(Polymorphism)**이다.

- **추상화 의존:** 시스템의 핵심 로직(수정에 닫혀 있어야 할 부분)이 구체적인 구현 클래스에 직접 의존하는 대신, 인터페이스나 추상 클래스와 같은 추상화된 타입에 의존하도록 설계한다.4 이렇게 하면 핵심 로직은 안정적으로 유지되면서, 새로운 기능은 이 추상화를 구현하는 새로운 클래스를 추가함으로써 시스템에 통합될 수 있다.
- **다형성 활용:** 클라이언트 코드는 추상화된 타입(예: 인터페이스)을 통해 작업을 요청한다. 런타임에 이 인터페이스의 어떤 구체적인 구현 객체가 전달되느냐에 따라 실제 수행되는 행위가 달라진다. 이것이 다형성의 힘이다. 다형성 덕분에 클라이언트 코드를 전혀 수정하지 않고도, 새로운 구현 클래스를 추가하는 것만으로 시스템의 행위를 확장할 수 있다.

#### 1.2.3 OCP 위반 사례 분석

OCP 위반의 가장 전형적인 예는 조건 분기문(예: `if/else if` 또는 `switch`)을 사용하여 타입에 따라 다른 로직을 처리하는 코드이다. 여러 종류의 도형 면적을 계산하는 `AreaCalculator` 클래스를 예로 들어보자.2

**위반 전 코드 (Java):**

```Java
class Rectangle {
    public double width;
    public double height;
}

class Circle {
    public double radius;
}

// OCP 위반: 새로운 도형이 추가될 때마다 이 클래스를 수정해야 함
class AreaCalculator {
    public double calculateArea(Object shapes) {
        double totalArea = 0;
        for (Object shape : shapes) {
            if (shape instanceof Rectangle) {
                Rectangle rect = (Rectangle) shape;
                totalArea += rect.width * rect.height;
            } else if (shape instanceof Circle) {
                Circle circle = (Circle) shape;
                totalArea += Math.PI * circle.radius * circle.radius;
            }
            // 새로운 도형 'Triangle'을 추가하려면 여기에 'else if'를 추가해야 함
        }
        return totalArea;
    }
}
```

위 `AreaCalculator` 클래스는 새로운 도형 타입(예: 삼각형)을 지원해야 할 때마다 `calculateArea` 메서드 내부에 `else if` 블록을 추가하여 코드를 직접 **수정**해야 한다. 이는 "수정에 대해 닫혀 있어야 한다"는 OCP의 원칙을 명백히 위반하는 것이다.2 이러한 코드는 확장에 취약하며, 수정이 반복될수록 복잡해지고 버그 발생 가능성이 높아진다.

#### 1.2.4 리팩토링: 전략 패턴의 적용

OCP를 준수하도록 코드를 리팩토링하기 위해, 변화하는 부분(도형별 면적 계산 로직)과 변하지 않는 부분(모든 도형의 면적을 합산하는 로직)을 분리하고, 이 둘을 추상화를 통해 연결한다. 이는 디자인 패턴 중 전략 패턴(Strategy Pattern)의 아이디어와 일치한다.

**리팩토링 후 코드 (Java):**

```Java
// 1. 추상화(인터페이스) 정의
interface Shape {
    double area();
}

// 2. 구체적인 구현(전략) 클래스들
class Rectangle implements Shape {
    public double width;
    public double height;

    @Override
    public double area() {
        return width * height;
    }
}

class Circle implements Shape {
    public double radius;

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}

// OCP 준수: 새로운 도형이 추가되어도 이 클래스는 수정되지 않음
class AreaCalculator {
    public double calculateArea(Shape shapes) {
        double totalArea = 0;
        for (Shape shape : shapes) {
            totalArea += shape.area(); // 다형성을 통해 실제 면적 계산 로직이 호출됨
        }
        return totalArea;
    }
}

// 3. 새로운 기능 확장
class Triangle implements Shape {
    public double base;
    public double height;

    @Override
    public double area() {
        return 0.5 * base * height;
    }
}
```

리팩토링 후 `AreaCalculator`는 더 이상 구체적인 도형 클래스(`Rectangle`, `Circle`)를 알지 못한다. 대신 `Shape`라는 인터페이스에만 의존한다. 이제 새로운 `Triangle` 클래스를 추가하더라도, `Triangle`이 `Shape` 인터페이스를 구현하기만 하면 `AreaCalculator`의 코드는 단 한 줄도 수정할 필요 없이 새로운 도형을 처리할 수 있다.7 이것이 바로 '확장에 열려 있고, 수정에 닫혀 있는' OCP의 이상적인 모습이다.

OCP는 단순히 `if`문을 사용하지 말라는 표면적인 지침이 아니다. 그 본질은 **변화의 축(Axis of Change)을 예측하고, 그 축을 중심으로 추상화**하라는 깊이 있는 설계 요구사항이다. 시스템의 모든 부분을 추상화하는 것은 비현실적이며 과잉 공학(over-engineering)으로 이어질 수 있다.11 따라서 숙련된 설계자는 시스템에서 어떤 부분이 변할 가능성이 높은지, 그리고 어떤 부분이 안정적으로 유지될 것인지를 식별해야 한다. 예를 들어, '결제 수단'은 새로운 방식(예: 간편 결제)이 계속 추가될 가능성이 높은 변화의 축이다. 반면, '상품 가격 계산' 로직은 상대적으로 안정적일 수 있다. OCP의 진정한 실천은 이처럼 변화가 예상되는 '결제 수단'을 중심으로 `PaymentStrategy`와 같은 인터페이스를 설계하고, 이를 통해 시스템의 유연성을 확보하는 것이다. 결국 OCP를 효과적으로 적용하는 능력은 단순히 기술적인 코딩 스킬을 넘어, 비즈니스 요구사항의 변화 방향을 예측하고 시스템의 진화 지점을 파악하는 **설계적 통찰력**에 달려있다.

### 1.3  리스코프 치환 원칙 (LSP: Liskov Substitution Principle)

#### 1.3.1 정의와 철학적 기반

리스코프 치환 원칙(LSP)은 1987년 바버라 리스코프(Barbara Liskov)에 의해 처음 소개되었으며, 객체 지향 프로그래밍에서 올바른 상속 관계를 정의하는 핵심적인 기준을 제시한다.13 원칙의 공식적인 정의는 다음과 같다: "S 타입의 각 객체 o1에 대해 T 타입의 객체 o2가 있고, T 타입으로 정의된 모든 프로그램 P에서 o2를 o1으로 치환해도 P의 행위가 변하지 않는다면, S는 T의 하위 타입이다".13 이를 더 실용적인 언어로 풀어쓰면, **"하위 클래스는 언제나 자신의 상위 클래스를 대체할 수 있어야 한다"**는 의미이다.3

이 원칙의 핵심은 상속이 단순히 코드 재사용을 위한 구조적(syntactic) 관계가 아니라, 행위의 일관성을 보장하는 행위적(behavioral) 관계여야 함을 강조하는 데 있다. 즉, 하위 클래스는 상위 클래스가 가진 모든 특성과 행위를 온전히 계승하고 확장해야 하며, 상위 클래스의 행위를 임의로 변경하거나 축소해서는 안 된다. 클라이언트 코드는 상위 클래스 타입의 객체를 사용하면서 특정 행위를 기대하는데, 하위 클래스 객체로 교체되었을 때 그 기대가 깨진다면 LSP를 위반한 것이다. 이러한 관점에서 LSP는 OCP를 지키기 위한 중요한 전제 조건이 된다. 하위 클래스로의 치환이 프로그램의 정확성을 깨뜨린다면, 그것은 올바른 '확장'이라고 볼 수 없기 때문이다.16

#### 1.3.2 LSP 위반의 조건

하위 클래스가 상위 클래스의 행위적 계약을 위반하는 경우는 다음과 같은 구체적인 조건들로 나타난다.

- **사전조건 강화(Strengthening Preconditions):** 하위 클래스의 메서드가 상위 클래스의 동일한 메서드보다 더 강력한(즉, 더 좁은 범위의) 입력 조건을 요구하는 경우이다. 예를 들어, 상위 클래스 메서드가 모든 양수를 인자로 받을 수 있는데, 하위 클래스에서 100 이상의 양수만 받도록 제한한다면 이는 LSP 위반이다.4
- **사후조건 약화(Weakening Postconditions):** 하위 클래스의 메서드가 상위 클래스의 메서드보다 더 약한(즉, 더 넓은 범위의 또는 덜 보장된) 결과를 반환하는 경우이다. 상위 클래스가 정렬된 리스트를 반환하기로 약속했는데, 하위 클래스가 정렬되지 않은 리스트를 반환한다면 이는 사후조건을 약화시킨 것이다.14
- **불변식(Invariants) 위반:** 상위 클래스가 자신의 생명주기 동안 항상 유지해야 하는 상태의 일관성(불변식)을 하위 클래스가 깨뜨리는 경우이다. 이는 LSP 위반의 가장 흔하고 미묘한 형태 중 하나이다.13
- **새로운 예외 발생:** 상위 클래스의 메서드가 던지지 않던 타입의 예외를 하위 클래스의 메서드에서 던지는 경우이다. 클라이언트 코드는 상위 클래스가 던지는 예외만 처리하도록 작성되었기 때문에, 예기치 않은 예외로 인해 프로그램이 오작동할 수 있다.13

#### 1.3.3 심층 분석: 정사각형/직사각형 문제 (The Square/Rectangle Problem)

LSP를 설명하는 가장 고전적이고 유명한 예제는 '정사각형/직사각형 문제'이다. 수학적으로 정사각형은 너비와 높이가 같은 직사각형의 특수한 경우이므로, 객체 지향 모델링에서 `Square` 클래스를 `Rectangle` 클래스의 하위 클래스로 만드는 것은 직관적으로 타당해 보인다. 그러나 이는 LSP를 위반하는 대표적인 사례이다.18

`Rectangle` 클래스는 다음과 같은 암묵적인 불변식을 가진다: "너비(`width`)와 높이(`height`)는 서로 독립적으로 변경될 수 있다." 클라이언트 코드는 이 불변식을 신뢰하고 `Rectangle` 객체를 사용한다.

**LSP 위반 코드 (Java):**

```Java
class Rectangle {
    protected int width;
    protected int height;

    public void setWidth(int width) {
        this.width = width;
    }

    public void setHeight(int height) {
        this.height = height;
    }

    public int getArea() {
        return width * height;
    }
}

class Square extends Rectangle {
    @Override
    public void setWidth(int width) {
        this.width = width;
        this.height = width; // 정사각형의 불변식(너비=높이)을 유지하기 위해 높이도 변경
    }

    @Override
    public void setHeight(int height) {
        this.width = height; // 너비도 함께 변경
        this.height = height;
    }
}

// 클라이언트 코드
public class LspTest {
    public static void testArea(Rectangle r) {
        r.setWidth(5);
        r.setHeight(4);
        // 클라이언트는 r의 너비가 5, 높이가 4이므로 넓이가 20일 것이라고 기대한다.
        if (r.getArea()!= 20) {
            throw new AssertionError("Area calculation failed!");
        }
    }

    public static void main(String args) {
        Rectangle rect = new Rectangle();
        testArea(rect); // 성공

        Rectangle square = new Square();
        testArea(square); // AssertionError 발생!
    }
}
```

`Square` 클래스는 정사각형의 속성(너비와 높이가 항상 같음)을 유지하기 위해 `setWidth`나 `setHeight`가 호출될 때 너비와 높이를 모두 같은 값으로 설정한다. 이로 인해 `Rectangle`의 불변식("너비와 높이는 독립적이다")이 깨진다. `testArea` 메서드는 `Rectangle` 타입의 객체를 인자로 받으므로, `Square` 객체도 전달받을 수 있어야 한다. 하지만 `Square` 객체를 전달하면 `setHeight(4)`가 호출되는 순간 너비도 4로 변경되어 최종 넓이는 16이 된다. 이는 클라이언트의 기대(20)를 위반하며, 프로그램의 정확성을 깨뜨린다.17 따라서 `Square`는 `Rectangle`을 안전하게 대체할 수 없으며, LSP를 위반한다.

#### 1.3.4 리팩토링: 올바른 계층 구조 설계

이 문제를 해결하기 위해서는 행위적으로 'IS-A' 관계가 성립하지 않을 때 상속을 사용하지 말아야 한다는 교훈을 얻어야 한다. 대신, 공통의 추상화에 의존하는 구조로 재설계해야 한다.

**리팩토링 후 코드 (Java):**

```Java
// 공통 추상화
interface Shape {
    int getArea();
}

// Rectangle과 Square는 이제 형제 클래스
class Rectangle implements Shape {
    private int width;
    private int height;

    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public int getArea() {
        return width * height;
    }
}

class Square implements Shape {
    private int side;

    public Square(int side) {
        this.side = side;
    }

    @Override
    public int getArea() {
        return side * side;
    }
}
```

이제 `Rectangle`과 `Square`는 상속 관계가 아닌, 공통의 `Shape` 인터페이스를 구현하는 독립적인 클래스가 되었다.19 이 설계에서는 더 이상 LSP 위반 문제가 발생하지 않으며, 각 클래스는 자신의 불변식을 온전히 유지할 수 있다.

LSP는 본질적으로 **계약에 의한 설계(Design by Contract)** 개념을 상속 관계에 적용한 것이다. 상위 클래스의 public 메서드들은 클라이언트와 맺는 일종의 '서비스 계약'을 정의한다. 이 계약에는 메서드 시그니처와 같은 명시적 조건뿐만 아니라, 사전조건, 사후조건, 불변식과 같은 암묵적인 행위 조건도 포함된다.3 클라이언트 코드는 이 계약을 신뢰하고 상위 클래스 타입의 객체를 사용한다. 상속은 이 계약을 재사용하고 확장하는 메커니즘이다. LSP는 하위 클래스가 이 계약을 확장할 수는 있지만(예: 더 강력한 사후조건을 제공), 계약의 근본적인 내용을 위반해서는 안 된다고 규정한다. 정사각형/직사각형 문제의 핵심은 `Square`가 `Rectangle`의 '너비와 높이는 독립적'이라는 암묵적 계약을 파기했다는 점이다. 이처럼 LSP 위반은 단순한 버그를 넘어 시스템의 신뢰 기반을 무너뜨리는 **계약 파기** 행위이며, 다형성을 통한 유연한 확장의 근간을 흔들어 결국 OCP의 실패로 직결된다.

### 1.4  인터페이스 분리 원칙 (ISP: Interface Segregation Principle)

#### 1.4.1 정의와 철학적 기반

인터페이스 분리 원칙(ISP)은 "클라이언트는 자신이 사용하지 않는 메서드에 의존하도록 강요되어서는 안 된다(No client should be forced to depend on methods it does not use)"고 명시한다.3 이 원칙은 단일 책임 원칙(SRP)이 클래스의 책임을 다루는 것과 유사하게, 인터페이스의 책임을 다룬다.23 즉, 하나의 거대하고 범용적인 인터페이스보다는, 클라이언트의 특정 요구에 맞춰 잘게 분리된 여러 개의 구체적인 인터페이스가 더 낫다는 것이다. 이러한 작고 응집도 높은 인터페이스를 **역할 인터페이스(Role Interfaces)**라고도 부른다.22 ISP의 목표는 시스템의 결합도를 낮추어 변경에 용이하고, 재사용성이 높으며, 이해하기 쉬운 코드를 만드는 것이다.

#### 1.4.2 "비대한 인터페이스(Fat Interface)"의 문제점

ISP를 위반하는 주된 원인은 '비대한 인터페이스(Fat Interface)' 또는 '인터페이스 오염(interface pollution)'이다.25 이는 하나의 인터페이스가 서로 다른 클라이언트들이 필요로 하는 관련 없는 여러 메서드들을 한꺼번에 포함하고 있는 상황을 말한다. 비대한 인터페이스는 다음과 같은 심각한 문제들을 야기한다.

- **불필요한 구현 강요:** 인터페이스를 구현하는 클래스는 자신이 실제로 사용하지 않는 메서드까지도 의무적으로 구현해야 한다. 이는 종종 내용이 비어 있거나, `UnsupportedOperationException`과 같은 예외를 던지는 무의미한 코드를 양산하게 된다. 이러한 구현은 클라이언트에게 혼란을 줄 뿐만 아니라, 리스코프 치환 원칙(LSP)을 위반할 소지가 매우 크다.27
- **불필요한 결합과 경직성:** 비대한 인터페이스는 그것을 사용하는 모든 클라이언트와 구현하는 모든 클래스 간에 불필요한 결합을 만들어낸다. 예를 들어, A 클라이언트의 요구사항 변경으로 인해 인터페이스에 새로운 메서드가 추가되면, 그 메서드를 전혀 사용하지 않는 B, C 클라이언트와 모든 구현 클래스들까지 영향을 받게 된다. 최소한 재컴파일이 필요하며, 경우에 따라서는 불필요한 코드 수정을 유발할 수도 있다. 이는 시스템 전체를 경직되게 만들고 변경 비용을 증가시킨다.22
- **낮은 응집도와 가독성 저하:** 비대한 인터페이스는 여러 책임을 한 곳에 모아두었기 때문에 본질적으로 응집도가 낮다. 이로 인해 인터페이스의 목적을 한눈에 파악하기 어렵고, 코드를 이해하고 사용하는 데 더 많은 인지적 노력이 필요하게 된다.

#### 1.4.3 ISP 위반 사례 분석

인쇄, 스캔, 팩스 기능을 모두 제공하는 복합기를 모델링하는 상황을 가정해보자. ISP를 고려하지 않으면 다음과 같은 비대한 인터페이스를 설계하기 쉽다.

**위반 전 코드 (Java):**

```Java
// ISP 위반: 비대한 인터페이스
interface IMultiFunctionDevice {
    void print(Document d);
    void scan(Document d);
    void fax(Document d);
}

// 복합기는 모든 기능을 구현하므로 문제가 없어 보임
class MultiFunctionPrinter implements IMultiFunctionDevice {
    @Override
    public void print(Document d) { /* 인쇄 로직 */ }
    @Override
    public void scan(Document d) { /* 스캔 로직 */ }
    @Override
    public void fax(Document d) { /* 팩스 로직 */ }
}

// 하지만 단순 프린터는 스캔과 팩스 기능이 없음
class SimplePrinter implements IMultiFunctionDevice {
    @Override
    public void print(Document d) { /* 인쇄 로직 */ }

    @Override
    public void scan(Document d) {
        // 불필요한 구현 강요
        throw new UnsupportedOperationException("Scan not supported.");
    }

    @Override
    public void fax(Document d) {
        // 불필요한 구현 강요
        throw new UnsupportedOperationException("Fax not supported.");
    }
}
```

`SimplePrinter` 클래스는 오직 `print` 기능만 필요함에도 불구하고, `IMultiFunctionDevice` 인터페이스 때문에 불필요한 `scan`과 `fax` 메서드를 구현하도록 강요받는다.26 이는 ISP의 명백한 위반이며, LSP 또한 위반할 가능성을 내포한다.

#### 1.4.4 리팩토링: 역할 인터페이스로의 분리

이 문제를 해결하기 위해서는 비대한 인터페이스를 클라이언트의 역할에 따라 더 작고 응집도 높은 인터페이스로 분리해야 한다.

**리팩토링 후 코드 (Java):**

```Java
// ISP 준수: 역할 인터페이스로 분리
interface IPrinter {
    void print(Document d);
}

interface IScanner {
    void scan(Document d);
}

interface IFax {
    void fax(Document d);
}

// SimplePrinter는 필요한 인터페이스만 구현
class SimplePrinter implements IPrinter {
    @Override
    public void print(Document d) { /* 인쇄 로직 */ }
}

// 복합기는 여러 역할 인터페이스를 조합하여 구현
class MultiFunctionPrinter implements IPrinter, IScanner, IFax {
    @Override
    public void print(Document d) { /* 인쇄 로직 */ }
    @Override
    public void scan(Document d) { /* 스캔 로직 */ }
    @Override
    public void fax(Document d) { /* 팩스 로직 */ }
}

// 클라이언트는 자신이 필요한 최소한의 인터페이스에만 의존
class PrintClient {
    private final IPrinter printer;

    public PrintClient(IPrinter printer) {
        this.printer = printer;
    }

    public void doPrint(Document d) {
        printer.print(d);
    }
}
```

리팩토링을 통해 `IMultiFunctionDevice`는 `IPrinter`, `IScanner`, `IFax`라는 세 개의 역할 인터페이스로 분리되었다.22 이제 `SimplePrinter`는 `IPrinter` 인터페이스만 구현하면 되므로 더 이상 불필요한 메서드를 강요받지 않는다. `MultiFunctionPrinter`는 필요한 모든 인터페이스를 구현하여 자신의 다기능성을 표현한다. 또한 `PrintClient`와 같은 클라이언트는 이제 거대한 `IMultiFunctionDevice`가 아닌, 자신의 역할에 꼭 맞는 `IPrinter` 인터페이스에만 의존하게 되어 결합도가 현저히 낮아졌다.

ISP와 DIP(의존성 역전 원칙)는 함께 작용하여 유연하고 확장 가능한 **플러그인 아키텍처(Plugin Architecture)**를 가능하게 하는 핵심적인 메커니즘을 형성한다. DIP는 상위 모듈이 하위 모듈의 구체적인 구현이 아닌 '추상화'에 의존해야 한다고 요구하며, 이 '추상화'는 보통 인터페이스의 형태로 나타난다. 만약 이 인터페이스가 ISP를 위반한 '비대한 인터페이스'라면, 상위 모듈은 자신이 사용하지도 않는 수많은 메서드에 불필요하게 의존하게 되어 DIP가 목표로 하는 '느슨한 결합'의 효과가 반감된다. ISP는 바로 이 인터페이스를 역할에 따라 잘게 쪼개어, 상위 모듈이 '자신에게 꼭 필요한 최소한의 추상화'에만 의존하도록 보장한다. 이렇게 잘 분리된 인터페이스(ISP 준수)를 매개로 의존성을 역전(DIP 준수)시키면, 하위 모듈(구현체)은 마치 플러그인처럼 시스템에 쉽게 교체되거나 추가될 수 있다. 상위 모듈은 오직 자신이 필요로 하는 '역할'에만 관심을 가지기 때문이다. 결론적으로, ISP는 DIP가 최대의 효과를 발휘하기 위한 '인터페이스의 품질'을 보장하는 필수적인 파트너 원칙이라 할 수 있다.

### 1.5  의존성 역전 원칙 (DIP: Dependency Inversion Principle)

#### 1.5.1 정의와 철학적 기반

의존성 역전 원칙(DIP)은 견고하고 유연한 소프트웨어 아키텍처를 구축하기 위한 핵심적인 지침으로, 다음과 같은 두 가지 명제로 구성된다.3

1. **상위 수준 모듈은 하위 수준 모듈에 의존해서는 안 된다. 둘 다 추상화에 의존해야 한다.** (High-level modules should not depend on low-level modules. Both should depend on abstractions.)
2. **추상화는 세부 사항에 의존해서는 안 된다. 세부 사항이 추상화에 의존해야 한다.** (Abstractions should not depend on details. Details should depend on abstractions.)

여기서 '상위 수준 모듈'은 비즈니스 정책이나 핵심 로직과 같이 시스템의 본질적인 부분을 다루는 코드를 의미하며, '하위 수준 모듈'은 데이터베이스 접근, 네트워크 통신, 파일 시스템 I/O와 같이 이러한 정책을 구현하기 위한 구체적인 기술이나 메커니즘을 다루는 코드를 의미한다.

전통적인 절차적 프로그래밍에서는 제어 흐름과 의존성의 방향이 일치한다. 즉, 상위 수준 모듈이 하위 수준 모듈을 직접 호출하고 의존하는 구조를 가진다. DIP는 이러한 전통적인 의존성의 흐름을 '역전(inversion)'시켜야 한다고 주장한다. 의존성을 역전시킨다는 것은, 상위 수준 모듈이 자신이 필요로 하는 서비스에 대한 '추상 인터페이스'를 정의하고 소유하며, 하위 수준 모듈은 이 인터페이스를 구현하도록 만드는 것을 의미한다. 그 결과, 의존성의 방향이 '하위 수준 모듈 -->> 추상 인터페이스 ← 상위 수준 모듈'의 형태로 바뀌게 된다. 즉, 구체적인 세부 사항(하위 모듈)이 시스템의 핵심 정책(상위 모듈이 정의한 추상화)에 의존하게 되는 것이다. 이 원칙의 궁극적인 목표는 모듈 간의 결합도를 낮추어 시스템을 더 유연하고, 테스트하기 쉬우며, 유지보수하기 쉽게 만드는 것이다.3

#### 1.5.2 DIP 위반 사례 분석

DIP를 위반하는 코드는 상위 수준 모듈이 하위 수준 모듈의 구체적인 클래스를 직접 참조하고 생성하는 형태를 띤다. 예를 들어, 사용자에게 알림을 보내는 `NotificationService`를 생각해보자.

**위반 전 코드 (Java):**

```Java
// 하위 수준 모듈: 구체적인 이메일 발송 클래스
class EmailSender {
    public void sendEmail(String message) {
        System.out.println("Sending email: " + message);
    }
}

// 상위 수준 모듈: 비즈니스 로직
// DIP 위반: 상위 모듈이 하위 모듈의 구체 클래스(EmailSender)에 직접 의존
class NotificationService {
    private final EmailSender emailSender;

    public NotificationService() {
        this.emailSender = new EmailSender(); // 직접 생성 및 의존
    }

    public void sendNotification(String message) {
        emailSender.sendEmail(message);
    }
}
```

위 코드에서 `NotificationService`(상위 수준 모듈)는 `EmailSender`(하위 수준 모듈)라는 구체적인 클래스에 직접 의존하고 있다. 만약 알림 방식을 이메일에서 SMS로 변경해야 한다면, `NotificationService` 클래스의 내부 코드를 `SmsSender`를 사용하도록 직접 수정해야만 한다. 이는 OCP를 위반하는 경직된 설계이다.

#### 1.5.3 리팩토링: 추상화를 통한 의존성 역전

DIP를 적용하기 위해서는 상위 수준 모듈과 하위 수준 모듈 사이에 추상화 계층(인터페이스)을 도입하고, 의존성의 방향을 역전시켜야 한다.

**리팩토링 후 코드 (Java):**

```Java
// 1. 추상화 정의 (상위 모듈이 소유)
interface IMessageSender {
    void sendMessage(String message);
}

// 2. 하위 수준 모듈들은 추상화를 구현
class EmailSender implements IMessageSender {
    @Override
    public void sendMessage(String message) {
        System.out.println("Sending email: " + message);
    }
}

class SmsSender implements IMessageSender {
    @Override
    public void sendMessage(String message) {
        System.out.println("Sending SMS: " + message);
    }
}

// 3. 상위 수준 모듈은 추상화에만 의존
// DIP 준수: 상위 모듈이 추상 인터페이스(IMessageSender)에만 의존
class NotificationService {
    private final IMessageSender messageSender;

    // 의존성 주입(Dependency Injection)을 통해 구체적인 구현을 외부에서 받음
    public NotificationService(IMessageSender messageSender) {
        this.messageSender = messageSender;
    }

    public void sendNotification(String message) {
        messageSender.sendMessage(message);
    }
}

// 4. 조립(Composition Root)
public class Main {
    public static void main(String args) {
        // 이메일로 알림을 보내고 싶을 때
        IMessageSender emailSender = new EmailSender();
        NotificationService emailNotification = new NotificationService(emailSender);
        emailNotification.sendNotification("Hello via Email!");

        // SMS로 알림을 보내고 싶을 때
        IMessageSender smsSender = new SmsSender();
        NotificationService smsNotification = new NotificationService(smsSender);
        smsNotification.sendNotification("Hello via SMS!");
    }
}
```

리팩토링 후 `NotificationService`는 더 이상 `EmailSender`나 `SmsSender`와 같은 구체적인 클래스를 알지 못한다. 대신 `IMessageSender`라는 추상 인터페이스에만 의존한다. 실제 사용할 구현체는 생성자를 통해 외부에서 '주입(inject)'받는다. 이제 `NotificationService`의 코드를 전혀 변경하지 않고도 알림 방식을 자유롭게 교체할 수 있게 되었다. 이는 DIP를 통해 OCP를 달성하는 전형적인 예시이다.31

DIP, 의존성 주입(Dependency Injection, DI), 제어의 역전(Inversion of Control, IoC)은 종종 혼용되지만, 실제로는 서로 다른 수준의 개념이면서 강력한 인과 관계를 형성한다. 이들의 관계를 명확히 이해하는 것은 매우 중요하다.

1. **DIP는 '무엇을(What)'에 대한 설계 원칙(Principle)이다.** "추상화에 의존하라"는 아키텍처의 목표를 제시한다.30
2. **DI는 '어떻게(How)'에 대한 디자인 패턴(Pattern)이다.** DIP라는 원칙을 구현하기 위한 여러 구체적인 기술 중 하나이다. 클래스 외부에서 의존 객체를 생성하여 생성자, 세터 메서드, 또는 인터페이스를 통해 '주입'하는 방식을 말한다.30
3. **IoC는 '왜(Why)'에 대한 프로그래밍 패러다임(Paradigm)이다.** 프로그램의 제어 흐름을 전통적인 방식(내 코드가 프레임워크/라이브러리를 호출)에서 역전시키는(프레임워크가 내 코드를 호출) 더 광범위하고 근본적인 개념이다. 객체의 생성, 생명주기 관리, 메서드 호출 등의 제어권을 개발자가 아닌 프레임워크나 컨테이너가 갖는 것이 대표적인 예이다.30

이들의 인과 관계는 다음과 같이 정리할 수 있다: **DIP(원칙)**라는 목표를 달성하기 위해 **DI(패턴)**라는 수단을 사용하게 되고, DI를 시스템 전반에 걸쳐 체계적으로 적용하면(특히 Spring과 같은 프레임워크를 통해) 이는 곧 **IoC(패러다임)**를 구현하는 것이 된다. 이 세 가지 개념을 혼동하는 것은 설계의 목표(What), 수단(How), 그리고 결과(Why)를 뒤섞는 것과 같다.

## 2.  원칙의 통합과 확장

### 2.1  원칙 간의 상호작용과 시너지 효과

SOLID의 다섯 가지 원칙은 독립적으로 존재하는 규칙의 나열이 아니라, 하나의 유기적인 시스템처럼 서로를 보완하고 강화하며 작동한다. 이 원칙들이 개별적으로 적용될 때도 코드 품질 향상에 기여하지만, 함께 조화롭게 적용될 때 그 효과는 비선형적으로 증폭되어 견고하고 유연한 객체 지향 설계라는 공동의 목표를 달성한다.3 이들의 시너지는 마치 복잡한 화학 반응처럼, 개별 요소의 단순한 합을 넘어서는 전체적인 설계 품질의 고도화를 이끌어낸다.36

각 원칙 간의 상호 보완 관계는 다음과 같이 분석될 수 있다.

- **SRP와 OCP의 관계:** 단일 책임 원칙(SRP)은 개방-폐쇄 원칙(OCP)을 달성하기 위한 가장 기본적인 전제 조건이다. SRP를 위반하여 하나의 클래스가 여러 책임을 가지게 되면, 그중 어떤 책임에 대한 요구사항 변경이 발생하더라도 해당 클래스의 코드를 직접 수정해야만 한다. 이는 OCP를 정면으로 위반하는 것이다. 반면, SRP를 준수하여 책임을 잘게 분리하면, 각 클래스는 변경의 이유가 하나뿐이므로 변경의 영향 범위가 매우 좁아진다. 이처럼 변경의 영향이 국소화된 클래스는 '수정에 닫혀있기'가 훨씬 용이해지며, 새로운 기능은 새로운 클래스를 추가하는 방식으로 '확장'될 수 있다.10
- **LSP와 OCP의 관계:** 리스코프 치환 원칙(LSP)은 OCP가 의도한 '확장'이 시스템의 안정성을 해치지 않도록 보장하는 핵심적인 안전장치 역할을 한다. OCP는 주로 상속이나 인터페이스 구현을 통해 기능을 확장하는데, 만약 이 과정에서 만들어진 하위 타입이 상위 타입을 온전히 대체할 수 없다면(LSP 위반), 다형성을 신뢰할 수 없게 된다. 클라이언트 코드는 특정 하위 타입의 예외적인 동작을 처리하기 위해 `instanceof`와 같은 타입 검사 코드를 추가해야 하며, 이는 결국 기존 코드의 수정을 유발하여 OCP를 깨뜨린다. 따라서 LSP가 보장되어야만, 클라이언트 코드의 수정 없이 새로운 하위 타입을 안전하게 추가하는 진정한 의미의 '확장'이 가능해진다.10
- **ISP, DIP와 다른 원칙들의 관계:** 인터페이스 분리 원칙(ISP)과 의존성 역전 원칙(DIP)은 시스템의 결합도를 낮추는 데 결정적인 역할을 함으로써 다른 원칙들이 효과적으로 작동할 수 있는 토양을 마련한다. ISP는 클라이언트가 불필요한 의존성을 갖지 않도록 인터페이스를 작고 응집력 있게 유지한다. DIP는 이처럼 잘 정의된 인터페이스(추상화)를 매개로 모듈 간의 의존성 방향을 역전시켜, 구체적인 구현의 변경이 다른 모듈로 전파되는 것을 차단한다. 이렇게 결합도가 낮아진 시스템은 새로운 구현체를 추가하거나(OCP) 기존 구현체를 교체하는(LSP) 작업이 매우 용이해진다. 즉, ISP와 DIP는 변경에 유연한 구조를 만드는 데 필수적인 기반 공사와 같다.10

종합적으로 볼 때, SRP와 ISP는 주로 모듈 **내부**의 품질, 즉 **높은 응집도**를 확보하는 데 중점을 둔다. 반면, OCP, LSP, DIP는 모듈 **외부**, 즉 모듈 **간의 관계**에서 **낮은 결합도**를 유지하는 데 중점을 둔다. 이 다섯 원칙이 함께 어우러질 때, 시스템은 내부적으로는 견고하고 외부적으로는 유연한, 이상적인 아키텍처에 가까워진다.

이러한 관점에서 SOLID 원칙 전체는 **변경에 대한 체계적인 관리 전략(Strategy for Managing Change)**으로 요약될 수 있다. 소프트웨어의 본질은 끊임없는 '변화'에 있으며, 성공적인 소프트웨어는 반드시 진화의 과정을 거친다.1 모든 변경은 비용과 위험을 수반하므로, 좋은 설계의 목표는 이 비용과 위험을 최소화하는 것이다. SOLID는 이 목표를 달성하기 위한 구체적인 전술을 제공한다.

1. **SRP**는 변경이 발생할 지점을 하나의 클래스로 **국소화**한다.
2. **OCP**는 새로운 요구사항이 발생했을 때, 기존의 안정적인 코드를 건드리지 않고 변경(확장)할 수 있는 **안전한 경로**를 제공한다.
3. **LSP**는 OCP의 확장 경로(주로 상속)가 시스템의 안정성을 해치지 않도록 보장하는 **안전장치** 역할을 한다.
4. **ISP와 DIP**는 변경의 파급 효과가 다른 모듈로 전파되는 것을 차단하는 **방화벽** 역할을 한다. 추상화라는 완충 지대를 만들어 의존성을 효과적으로 격리시킨다.

결론적으로, SOLID는 미래에 발생할 가능성이 있는 변경을 예측하고, 그 변경이 시스템 전체에 미치는 영향을 최소화하도록 코드를 구조화하는 체계적이고 종합적인 방법론이라 할 수 있다.

### 2.2  SOLID의 경계 확장: 현대적 패러다임에의 적용

SOLID 원칙은 비록 객체 지향 프로그래밍(OOP)의 맥락에서 탄생하고 발전했지만, 그 근본 철학은 특정 프로그래밍 패러다임이나 기술에 얽매이지 않는다. 복잡성을 관리하고, 응집도를 높이며, 결합도를 낮추려는 이 원칙들의 핵심 아이디어는 함수형 프로그래밍(FP)이나 마이크로서비스 아키텍처(MSA)와 같은 현대적인 패러다임에서도 여전히 유효하며, 해당 패러다임의 언어와 도구에 맞게 재해석되어 적용될 수 있다.38

#### 2.2.1 함수형 프로그래밍(Functional Programming)에서의 SOLID

함수형 프로그래밍에는 클래스, 상속, 인터페이스와 같은 OOP의 핵심 구성 요소가 없거나 다른 형태로 존재한다. 그럼에도 불구하고 SOLID의 각 원칙은 다음과 같이 의미 있는 대응 관계를 찾을 수 있다.38

- **SRP (단일 책임 원칙):** 클래스 대신 **함수**에 적용된다. 함수형 프로그래밍에서 함수는 작고, 재사용 가능하며, 예측 가능한 단일 작업을 수행하는 것을 이상으로 삼는다. 특히 부수 효과가 없는 순수 함수(pure function)와 이들을 조합하여 더 복잡한 기능을 만드는 함수 합성(function composition)은 자연스럽게 SRP를 장려하는 강력한 도구가 된다.38
- **OCP (개방-폐쇄 원칙):** 상속을 통한 확장 대신 **고차 함수(Higher-Order Functions)**를 통해 달성된다. 고차 함수는 다른 함수를 인자로 받거나 함수를 결과로 반환할 수 있다. 이를 통해 기존 함수의 코드를 수정하지 않고, 동작을 변경하는 새로운 함수를 인자로 전달함으로써 행위를 유연하게 확장할 수 있다.38
- **LSP (리스코프 치환 원칙):** 클래스 계층 구조 대신 **파라메트릭 다형성(Parametric Polymorphism)**, 즉 제네릭(Generics)에 적용된다. 제네릭을 사용하는 함수는 어떤 구체적인 타입이 인자로 전달되더라도 그 타입의 내부 구조에 의존하지 않고 일관된 방식으로 동작해야 한다. 이는 특정 타입의 구현에 상관없이 치환 가능해야 한다는 LSP의 정신과 일치한다.38
- **ISP (인터페이스 분리 원칙):** 거대한 인터페이스 대신 **작고 구체적인 함수 시그니처**로 해석될 수 있다. 함수는 자신이 작업을 수행하는 데 꼭 필요한 최소한의 인자만을 받아야 한다. 거대한 데이터 구조 전체를 넘겨받는 대신, 그 구조에서 필요한 특정 값만 인자로 받는 것이 ISP를 준수하는 방법이다.40
- **DIP (의존성 역전 원칙):** 구체적인 구현에 의존하는 대신 **추상화(함수 그 자체)**에 의존하는 것으로 자연스럽게 나타난다. 함수형 프로그래밍에서는 함수 자체가 일급 객체이므로, 의존성을 주입하는 것은 단순히 필요한 동작을 수행하는 함수를 다른 함수에 인자로 전달하는 방식으로 매우 간단하게 이루어진다.38

#### 2.2.2 마이크로서비스 아키텍처(Microservices Architecture)에서의 SOLID

SOLID 원칙은 개별 마이크로서비스의 내부 설계뿐만 아니라, 여러 서비스로 구성된 전체 아키텍처의 구조를 설계하는 데에도 중요한 지침을 제공한다.39

- **SRP (단일 책임 원칙):** "마이크로서비스는 하나의 명확한 비즈니스 기능(business capability)에 대해서만 책임져야 한다." 이는 마이크로서비스의 경계를 설정하는 가장 핵심적인 원칙이다. 예를 들어, '주문 관리 서비스', '결제 서비스', '재고 관리 서비스'는 각각 독립적인 책임을 진다. 이는 도메인 주도 설계(DDD)의 경계 컨텍스트(Bounded Context) 개념과 매우 유사하게 활용된다.44
- **OCP (개방-폐쇄 원칙):** 전체 시스템의 기능을 향상시키기 위해 기존 서비스를 직접 수정하는 것이 아니라, 새로운 기능을 제공하는 신규 서비스를 추가(확장)하고 이를 기존 시스템과 통합하는 방식으로 이루어져야 한다. 이벤트 기반 아키텍처에서 새로운 구독자(서비스)를 추가하거나, API 게이트웨이를 통해 새로운 라우팅 규칙을 추가하는 것이 OCP를 구현하는 대표적인 방식이다.39
- **LSP (리스코프 치환 원칙):** 서비스의 새로운 버전은 반드시 이전 버전과 **하위 호환성(backward compatibility)**을 유지해야 한다. API의 계약(contract)을 변경하지 않음으로써, 클라이언트(다른 서비스)의 코드를 전혀 수정하지 않고도 새로운 버전의 서비스로 안전하게 교체(치환)할 수 있어야 한다. 이는 무중단 배포와 점진적인 시스템 업그레이드를 가능하게 하는 핵심 요소이다.39
- **ISP (인터페이스 분리 원칙):** 서비스가 노출하는 API는 특정 클라이언트나 유스케이스에 맞춰 세분화되어야 한다. 모든 기능을 한꺼번에 제공하는 거대한 단일 API 엔드포인트('God API')보다는, 목적에 따라 잘게 나뉜 작은 API들을 제공하는 것이 바람직하다. 이를 통해 클라이언트는 자신이 필요로 하는 기능에만 의존하게 되어 서비스 간 결합도를 낮출 수 있다.39
- **DIP (의존성 역전 원칙):** 마이크로서비스는 다른 서비스를 직접 호출(동기식 강결합)하는 것을 지양해야 한다. 대신 서비스 디스커버리, API 게이트웨이, 메시지 큐와 같은 **추상화된 중간 계층**을 통해 비동기적으로 통신하거나 간접적으로 상호작용해야 한다. 이는 한 서비스의 장애나 변경이 다른 서비스로 직접 전파되는 것을 막아 시스템 전체의 회복탄력성(resilience)을 높인다.39

이처럼 SOLID 원칙이 OOP를 넘어 다양한 패러다임에 걸쳐 적용될 수 있다는 사실은, 이 원칙들이 클래스나 상속과 같은 특정 기술에 대한 지침이 아니라, 복잡성을 관리하고 변화에 대응하기 위한 **보편적이고 추상적인 소프트웨어 설계 원리**임을 강력하게 시사한다. SRP는 '응집도', OCP는 '확장성', LSP는 '치환 가능성', ISP는 '최소 의존', DIP는 '추상화 의존'이라는 더 근본적인 개념으로 환원될 수 있다. 이는 SOLID가 OOP라는 '구현체'를 통해 표현된 설계의 '인터페이스'와 같다는 것을 의미한다. 따라서 앞으로 새로운 프로그래밍 패러다임이나 아키텍처 스타일이 등장하더라도, SOLID의 근본 철학은 그에 맞는 형태로 변용되어 계속해서 유효할 가능성이 높으며, 이는 SOLID 원칙의 시대를 초월하는 가치를 증명한다.

## 3.  비판적 고찰과 실용적 균형

SOLID 원칙은 의심할 여지 없이 훌륭한 소프트웨어 설계를 위한 강력한 지침이지만, 모든 상황에 무비판적으로 적용해야 하는 은탄환(silver bullet)은 아니다. 원칙의 근본적인 의도를 이해하지 못하고 기계적으로 적용할 경우, 오히려 코드의 복잡성을 증가시키고 개발 생산성을 저해하는 부작용을 낳을 수 있다. 따라서 성숙한 개발자는 원칙을 맹목적으로 추종하는 것을 넘어, 비판적인 시각을 견지하고 실용적인 균형점을 찾아야 한다.

### 3.1  SOLID 원칙에 대한 비판적 시각

SOLID 원칙의 남용은 여러 가지 문제점을 야기할 수 있으며, 그중 가장 대표적인 것이 과잉 공학(Over-engineering)의 위험이다.

- **과잉 공학과 복잡성 증가:** 원칙을 맹목적으로 적용하면 불필요한 클래스, 인터페이스, 추상화 계층이 시스템에 과도하게 추가될 수 있다. 이는 코드의 양을 늘리고 구조를 복잡하게 만들어 오히려 가독성과 유지보수성을 해치는 결과를 초래한다.11 예를 들어, SRP를 극단적으로 적용하여 모든 메서드를 별도의 클래스로 분리하는 '클래스 폭발(class explosion)' 현상은 오히려 개발자가 코드의 전체적인 흐름을 파악하기 어렵게 만든다.6 또한, OCP를 준수하기 위해 아직 발생하지도 않은 모든 미래의 변화를 예측하고 추상화 계층을 만드는 것은 비현실적이며, 결국 사용되지 않는 복잡한 코드만 남길 수 있다.12
- **성능 저하 가능성:** 추상화 계층의 도입과 의존성 주입을 통한 간접 호출(indirection)은 시스템의 유연성을 높이는 대가로 약간의 런타임 성능 저하를 유발할 수 있다. 대부분의 비즈니스 애플리케이션에서는 이 성능 저하가 무시할 수 있는 수준이지만, 고성능이 극도로 중요한 실시간 거래 시스템, 저지연(low-latency) 애플리케이션, 또는 자원이 제한된 임베디드 시스템에서는 이러한 오버헤드가 문제가 될 수 있다.11
- **단기 프로젝트 및 프로토타이핑의 비효율성:** 일회성 스크립트나 단기간 사용 후 폐기될 프로젝트, 또는 요구사항이 급격하게 변하는 초기 프로토타이핑 단계에서 SOLID 원칙을 엄격하게 적용하는 것은 불필요한 시간과 노력을 낭비하는 결과를 낳을 수 있다. 이러한 상황에서는 빠른 개발 속도와 신속한 피드백이 더 중요하므로, 어느 정도의 기술적 부채를 감수하고 단순하게 구현하는 것이 더 합리적인 선택일 수 있다.11
- **가독성 저하:** 과도한 분리와 추상화는 코드의 논리적 흐름을 추적하기 어렵게 만들 수 있다. 간단한 기능을 이해하기 위해 여러 개의 파일을 넘나들며 클래스 정의와 인터페이스, 의존성 주입 설정을 모두 확인해야 하는 상황은 개발자의 인지 부하(cognitive load)를 크게 높인다.11

### 3.2  다른 설계 원칙과의 비교: GRASP를 중심으로

SOLID 원칙의 실용적 가치를 더 잘 이해하기 위해, 또 다른 유명한 객체 지향 설계 원칙 모음인 GRASP(General Responsibility Assignment Software Patterns)와 비교해 볼 수 있다.

- **접근 방식의 차이:** SOLID는 "클래스는 단 하나의 책임만 가져야 한다" 또는 "추상화에 의존해야 한다"와 같이 무엇이 좋은 설계인지를 명확하게 규정하는 **규범적(prescriptive)** 성격이 강하다. 반면, GRASP는 'Information Expert'(정보 전문가), 'Creator'(생성자), 'Controller'(컨트롤러)와 같이 "객체에 책임을 할당할 때 고려할 수 있는 아홉 가지 패턴"을 제시하는 **서술적(descriptive)**이고 유연한 가이드라인에 가깝다.48
- **초점의 차이:** SOLID는 주로 개별 클래스와 인터페이스의 **정적인 구조와 품질**(낮은 결합도, 높은 응집도)에 집중하는 경향이 있다. 반면, GRASP는 객체들이 메시지를 주고받으며 협력하는 **동적인 상호작용** 속에서 "누가 이 책임을 져야 하는가?"라는 질문에 답하는 데 더 초점을 맞춘다.49
- **상호 보완 관계:** 두 원칙 집합은 서로 대립하는 것이 아니라 상호 보완적인 관계에 있다. 개발자는 먼저 GRASP 원칙(예: Information Expert)을 사용하여 특정 책임을 수행할 가장 적합한 객체를 결정할 수 있다. 그런 다음, 그 객체의 내부 구조를 설계할 때는 SOLID 원칙(예: SRP, OCP)을 적용하여 더 견고하고 유지보수하기 쉽게 다듬을 수 있다. 즉, GRASP가 책임 할당의 '전략'을 제시한다면, SOLID는 그 전략을 구현하는 '전술'을 제공한다고 볼 수 있다.50

### 3.3  실용적 적용을 위한 제언

SOLID 원칙을 성공적으로 활용하기 위해서는 원칙 자체를 아는 것을 넘어, 그것을 언제, 어떻게, 그리고 어느 수준까지 적용할지 판단하는 지혜가 필요하다.

- **원칙(Principles)이지 규칙(Rules)이 아니다:** SOLID는 모든 상황에 예외 없이 적용해야 하는 엄격한 법률이 아니라, 더 나은 설계 결정을 내리는 데 도움을 주는 **지침(guidelines)** 또는 **휴리스틱(heuristics)**으로 이해해야 한다.52
- **맥락의 중요성:** 원칙의 적용 여부와 수준은 프로젝트의 구체적인 맥락에 따라 달라져야 한다. 프로젝트의 예상 수명, 팀의 규모와 숙련도, 비즈니스 도메인의 변화 가능성 등을 종합적으로 고려해야 한다.53 "SOLID 원칙을 달성하기 위해 코딩하는 것이 아니라, 유지보수성을 얻기 위해 SOLID 원칙을 사용해야 한다(Do not aim to attain SOLID, use Solid to attain maintainability)"는 격언은 이러한 맥락적 사고의 중요성을 잘 보여준다.52
- **균형 감각과 경제적 사고:** 모든 설계 결정은 본질적으로 트레이드오프(trade-off)를 수반한다. SOLID 원칙을 적용하여 얻는 추상화와 유연성의 이점과, 그로 인해 발생하는 복잡성 증가와 초기 개발 비용이라는 대가 사이에서 끊임없이 균형을 잡아야 한다. YAGNI(You Ain't Gonna Need It: 필요하기 전까지는 만들지 말라)나 KISS(Keep It Simple, Stupid: 단순하게 유지하라)와 같은 다른 원칙들과 함께 고려하여, 현재 상황에 가장 적합한 결정을 내리는 것이 중요하다.16

결국 SOLID 원칙의 성공적인 적용을 위한 가장 중요한 상위 원칙은 **경제적 사고(Economic Thinking)**이다. 모든 설계 결정은 일종의 **비용-편익 분석(Cost-Benefit Analysis)**에 기반해야 한다. SOLID 원칙을 적용하는 행위(예: 인터페이스 추출, 클래스 분리)는 즉각적인 '비용'(개발 시간 증가, 코드 라인 수 증가, 인지적 복잡성)을 발생시킨다. 이 비용을 지불함으로써 얻고자 하는 '편익'은 미래에 발생할 '유지보수 비용 감소'와 '확장성 향상'이다. 만약 프로젝트의 수명이 짧아 '미래'가 없거나 11, 시스템의 해당 부분이 변경될 가능성이 거의 없어 '유지보수'나 '확장'의 필요성이 낮다면, 이 거래는 명백한 손해다. 즉, 비용이 편익보다 크다. 반대로, 수명이 길고 변화가 잦은 대규모 시스템에서는 초기에 지불한 추상화의 비용이 미래에 발생할 수 있는 엄청난 수정 비용을 절감해주므로 매우 이득인 투자가 된다.53 따라서 "SOLID를 적용해야 하는가?"라는 질문은 궁극적으로 "이 추상화에 대한 투자가 미래에 더 큰 이익으로 돌아올 것인가?"라는 경제적 질문으로 치환되어야 한다. 이것이 바로 원칙을 맹목적으로 따르는 것과 현명하게 활용하는 것을 가르는 진정한 기준점이다.

## 4. 결론

SOLID 원칙은 지난 수십 년간 객체 지향 설계의 핵심적인 지침으로서 그 가치를 증명해왔다. 이 원칙들은 특정 프로그래밍 언어나 기술, 시대적 유행에 얽매이지 않는, 잘 구조화된 소프트웨어를 구축하기 위한 근본적인 사고의 틀을 제공한다. 코드의 응집도를 높이고 결합도를 낮춤으로써, 변화에 유연하고 테스트와 유지보수가 용이한 시스템을 만드는 데 여전히 핵심적인 역할을 수행하고 있다.1 현대의 함수형 프로그래밍이나 마이크로서비스 아키텍처와 같은 새로운 패러다임에서도 그 기본 철학이 유효하다는 사실은 SOLID 원칙의 시대를 초월하는 보편성을 입증한다.

그러나 원칙 그 자체보다 중요한 것은 그것을 사용하는 개발자의 지혜와 통찰력이다. 진정한 소프트웨어 전문가는 원칙의 정의를 암기하는 것을 넘어, 각 원칙이 추구하는 근본적인 의도(예: 변경의 격리, 계약의 준수)를 깊이 이해해야 한다. 그리고 이를 바탕으로 현재 당면한 프로젝트의 구체적인 맥락 속에서 언제, 어떻게, 그리고 어느 수준까지 원칙을 적용할지, 혹은 적용하지 않을지를 판단하는 균형 감각을 갖추어야 한다. SOLID는 모든 문제에 대한 정답을 알려주는 상세한 지도가 아니라, 더 나은 설계를 향한 여정에서 올바른 방향을 제시하는 나침반과 같다.

소프트웨어 공학의 세계는 앞으로도 계속해서 변화하고 진화할 것이다. 새로운 언어, 프레임워크, 아키텍처 스타일이 끊임없이 등장할 것이다. 하지만 복잡성을 제어하고, 변화에 효과적으로 대응하며, 지속 가능한 시스템을 만들어야 한다는 근본적인 과제는 변하지 않을 것이다. 따라서 SOLID가 담고 있는 추상화, 모듈화, 책임 분리의 철학은 앞으로도 오랫동안 소프트웨어 장인(Software Craftsman)들이 끊임없이 연마하고 추구해야 할 중요한 가치로 남을 것이다.54

#### **참고 자료**

1. SOLID Principles in Object Oriented Design – BMC Software | Blogs, 8월 15, 2025에 액세스, https://www.bmc.com/blogs/solid-design-principles/
2. SOLID Design Principles Explained: Building Better Software Architecture - DigitalOcean, 8월 15, 2025에 액세스, https://www.digitalocean.com/community/conceptual-articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design
3. SOLID - Wikipedia, 8월 15, 2025에 액세스, https://en.wikipedia.org/wiki/SOLID
4. SOLID Principles in object-oriented design - DEV Community, 8월 15, 2025에 액세스, https://dev.to/muzammilnm/solid-principles-in-object-oriented-design-37pg
5. The SOLID Principles of Object-Oriented Programming Explained in Plain English, 8월 15, 2025에 액세스, https://www.freecodecamp.org/news/solid-principles-explained-in-plain-english/
6. SOLID Design Principles: The Single Responsibility Explained - Stackify, 8월 15, 2025에 액세스, https://stackify.com/solid-design-principles/
7. SOLID Principles: How to create a code that is easy to extend and ..., 8월 15, 2025에 액세스, https://www.thoughtworks.com/insights/blog/agile-engineering-practices/solid-principles-how-to-create-a-code-that-is-easy-to-extend-and-maintain-part-1
8. SOLID Principles: Improve Object-Oriented Design in Python, 8월 15, 2025에 액세스, https://realpython.com/solid-principles-python/
9. How the SOLID Principles Guide Object-Oriented Design: Examples of Violations and Their Consequences in Large-Scale Applications | by Youngjun Kim | Medium, 8월 15, 2025에 액세스, https://medium.com/@youngjun_kim/how-the-solid-principles-guide-object-oriented-design-examples-of-violations-and-their-8bacac9dda23
10. The Interrelation of SOLID Principles: How They Work Together | by Pushpraj Priy | Medium, 8월 15, 2025에 액세스, https://medium.com/@thepushp2/the-interrelation-of-solid-principles-how-they-work-together-72eca222db94
11. When Using Solid Principles May Not Be Appropriate | Baeldung on Computer Science, 8월 15, 2025에 액세스, https://www.baeldung.com/cs/solid-principles-avoid
12. Challenging the Gospel of SOLID Principles | by Jeremykhoo - Medium, 8월 15, 2025에 액세스, https://medium.com/@jeremykhoois/solid-principles-sucks-b5935b1235d7
13. Liskov substitution principle - Wikipedia, 8월 15, 2025에 액세스, https://en.wikipedia.org/wiki/Liskov_substitution_principle
14. What Is The Liskov Substitution Principle (LSP)? - ITU Online IT Training, 8월 15, 2025에 액세스, https://www.ituonline.com/tech-definitions/what-is-the-liskov-substitution-principle-lsp/
15. stackify.com, 8월 15, 2025에 액세스, [https://stackify.com/solid-design-liskov-substitution-principle/#:~:text=The%20Liskov%20Substitution%20Principle%20in,the%20objects%20of%20your%20superclass.](https://stackify.com/solid-design-liskov-substitution-principle/#:~:text=The Liskov Substitution Principle in,the objects of your superclass.)
16. S.O.L.I.D The first 5 principles of Object Oriented Design. - Jayamini samaratunga, 8월 15, 2025에 액세스, https://samaratungajs.medium.com/s-o-l-i-d-the-first-5-principles-of-object-oriented-design-de54bf91c03d
17. Liskov Substitution Principle (LSP) - GitHub Pages, 8월 15, 2025에 액세스, https://stg-tud.github.io/sedc/Lecture/ws13-14/3.3-LSP.html
18. stg-tud.github.io, 8월 15, 2025에 액세스, [https://stg-tud.github.io/sedc/Lecture/ws13-14/3.3-LSP.html#:~:text=The%20Rectangle%20%2F%20Square%20hierarchy%20violates,height%2Fwidth%20of%20a%20rectangle.](https://stg-tud.github.io/sedc/Lecture/ws13-14/3.3-LSP.html#:~:text=The Rectangle %2F Square hierarchy violates,height%2Fwidth of a rectangle.)
19. Square, Rectangle and the Liskov Substitution Principle | by Alexandre Dutertre - Medium, 8월 15, 2025에 액세스, https://medium.com/@alex24dutertre/square-rectangle-and-the-liskov-substitution-principle-ee1eb8433106
20. Is a square a rectangle? Liskov substitution principle in action - Steven Giesel, 8월 15, 2025에 액세스, https://steven-giesel.com/blogPost/67425427-01cd-4707-8572-991c3f9c5fc2
21. Simplifying the Liskov Substitution Principle of SOLID in C# - Infragistics, 8월 15, 2025에 액세스, https://www.infragistics.com/blogs/liskov-substitution-principle/
22. Interface segregation principle - Wikipedia, 8월 15, 2025에 액세스, https://en.wikipedia.org/wiki/Interface_segregation_principle
23. Design Goals and Principles of Object Oriented Programming - GeeksforGeeks, 8월 15, 2025에 액세스, https://www.geeksforgeeks.org/software-engineering/design-goals-and-principles-of-object-oriented-programming/
24. Interface Segregation Principle. and how to interpret it | by Vadim Samokhin - Medium, 8월 15, 2025에 액세스, https://medium.com/@wrong.about/interface-segregation-principle-bdf3f94f1d11
25. The Interface Segregation Principle, 8월 15, 2025에 액세스, https://condor.depaul.edu/dmumaugh/OOT/Design-Principles/isp.pdf
26. SOLID series: Understanding the Interface Segregation Principle (ISP) - LogRocket Blog, 8월 15, 2025에 액세스, https://blog.logrocket.com/interface-segregation-principle-isp/
27. Is Interface Segregation Principle Redundant? - Mayallo, 8월 15, 2025에 액세스, https://mayallo.com/solid-interface-segregation-principle/
28. Interface Segregation Principle: Everything You Need to Know - Reflectoring, 8월 15, 2025에 액세스, https://reflectoring.io/interface-segregation-principle/
29. Interface Segregation Principle (ISP) with TypeScript examples | by Oleksandr Khomyakov, 8월 15, 2025에 액세스, https://medium.com/@khomyakov/interface-segregation-principle-isp-with-typescript-examples-8f82f538a9b2
30. Dependency Injection, Dependency Inversion, IoC | SSENSE-TECH - Medium, 8월 15, 2025에 액세스, https://medium.com/ssense-tech/dependency-injection-vs-dependency-inversion-vs-inversion-of-control-lets-set-the-record-straight-5dc818dc32d1
31. Dependency Inversion vs Dependency Injection | by BuketSenturk - Medium, 8월 15, 2025에 액세스, https://medium.com/@buketsenturk/dependency-inversion-vs-dependency-injection-4a2dea766e6b
32. Dependency Inversion Principle VS Dependency Injection in C# - C# Corner, 8월 15, 2025에 액세스, https://www.c-sharpcorner.com/article/dependency-inversion-principle-vs-dependency-injection-in-c-sharp/
33. Spring - Difference Between Inversion of Control and Dependency Injection, 8월 15, 2025에 액세스, https://www.geeksforgeeks.org/springboot/spring-difference-between-inversion-of-control-and-dependency-injection/
34. Inversion of Control vs Dependency Injection - Stack Overflow, 8월 15, 2025에 액세스, https://stackoverflow.com/questions/6550700/inversion-of-control-vs-dependency-injection
35. How is Inversion of Control related to Dependency Inversion - Software Engineering Stack Exchange, 8월 15, 2025에 액세스, https://softwareengineering.stackexchange.com/questions/311674/how-is-inversion-of-control-related-to-dependency-inversion
36. Synergistic Oxygen Reduction Catalysis by Fe Nanoparticles and Atomic Fe-Nx Sites in MOF-Derived Carbon Nanotubes | ACS Sustainable Chemistry & Engineering, 8월 15, 2025에 액세스, https://pubs.acs.org/doi/10.1021/acssuschemeng.5c05924
37. Photon-Primed Organic Electrosynthesis Enabled by Oxidation of Photon-Induced Intermediates | Journal of the American Chemical Society, 8월 15, 2025에 액세스, https://pubs.acs.org/doi/10.1021/jacs.5c07822
38. Do the SOLID principles apply to Functional Programming? - DEV Community, 8월 15, 2025에 액세스, https://dev.to/patferraggi/do-the-solid-principles-apply-to-functional-programming-56lm
39. SOLID Microservices Design. Applying SOLID principles to... | by Adrien Nortain - Zenika, 8월 15, 2025에 액세스, https://medium.zenika.com/solid-microservices-design-dc6a4044a050
40. SOLID Principles in Functional Programming (FP) with examples - DEV Community, 8월 15, 2025에 액세스, https://dev.to/rosselli00/solid-principles-in-functional-programming-fp-with-examples-4bj4
41. SOLID principles in Functional Programming | by Maciej Kocik - Medium, 8월 15, 2025에 액세스, https://medium.com/@mkocik/solid-principles-in-functional-programming-b9b83aeddf80
42. Applying SOLID Principles Using a Functional Approach in Kotlin and Scala - Medium, 8월 15, 2025에 액세스, https://medium.com/@jorgegfx/applying-solid-principles-using-a-functional-approach-in-kotlin-and-scala-1096a5bb54a4
43. SOLID for functional programming - Stack Overflow, 8월 15, 2025에 액세스, https://stackoverflow.com/questions/5577054/solid-for-functional-programming
44. Applying SOLID principles to services | by David Van Couvering - Medium, 8월 15, 2025에 액세스, https://david-vancouvering.medium.com/applying-solid-principles-to-services-e56ef2382a26
45. How to Build Resilient Microservice Systems – SOLID Principles for Microservices, 8월 15, 2025에 액세스, https://www.freecodecamp.org/news/solid-principles-for-microservices/
46. Microservices: Designing Effective Microservices by following SOLID Design Principles | by Saurabh Gupta | Medium, 8월 15, 2025에 액세스, https://medium.com/@saurabh.engg.it/microservices-designing-effective-microservices-by-following-solid-design-principles-a995c3a033a0
47. Why I Don't Teach SOLID - Hacker News, 8월 15, 2025에 액세스, https://news.ycombinator.com/item?id=8929458
48. medium.com, 8월 15, 2025에 액세스, [https://medium.com/i-am-a-dummy-enlighten-me/solid-and-grasp-what-are-these-07a8acc03670#:~:text=SOLID%20principles%20are%20more%20prescriptive,impose%20more%20constraints%20and%20limitations.](https://medium.com/i-am-a-dummy-enlighten-me/solid-and-grasp-what-are-these-07a8acc03670#:~:text=SOLID principles are more prescriptive,impose more constraints and limitations.)
49. SOLID and GRASP: What are these?. Object-oriented design is a paradigm... | by Harpreet Singh Kalsi | I am a dummy, enlighten me! | Medium, 8월 15, 2025에 액세스, https://medium.com/i-am-a-dummy-enlighten-me/solid-and-grasp-what-are-these-07a8acc03670
50. Design patterns over SOLID and GRASP principles in real projects - ResearchGate, 8월 15, 2025에 액세스, https://www.researchgate.net/publication/379027734_Design_patterns_over_SOLID_and_GRASP_principles_in_real_projects
51. Why SOLID principles are still the foundation for modern software architecture - Reddit, 8월 15, 2025에 액세스, https://www.reddit.com/r/softwarearchitecture/comments/qsaahi/why_solid_principles_are_still_the_foundation_for/
52. Are the SOLID Principles problematic? - Florian Krämer, 8월 15, 2025에 액세스, https://florian-kraemer.net/software-architecture/2025/02/24/Are-the-SOLID-Principles-problematic.html
53. When to *not* use SOLID principles - Software Engineering Stack Exchange, 8월 15, 2025에 액세스, https://softwareengineering.stackexchange.com/questions/447532/when-to-not-use-solid-principles
54. Robert C. Martin Series - InformIT, 8월 15, 2025에 액세스, https://www.informit.com/imprint/series_detail.aspx?ser=348084

