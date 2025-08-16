# 개방-폐쇄 원칙(OCP)

### 서론: 변화를 수용하는 설계의 정수, 개방-폐쇄 원칙

소프트웨어 공학의 역사는 끊임없이 변화하는 요구사항과의 투쟁의 역사라 해도 과언이 아니다. 소프트웨어(Software)의 본질적 속성은 그 이름이 암시하듯 '부드러움(soft)', 즉 변경에 대한 유연성에 있다.1 그러나 현실의 많은 소프트웨어 시스템은 작은 요구사항 변경에도 연쇄적인 수정을 유발하며 시스템 전체를 위협하는 '경직성(rigidity)'과 '취약성(fragility)'을 드러낸다.2 이러한 소프트웨어의 경화를 방지하고, 시간이 지남에 따라 진화하는 시스템을 구축하기 위한 지침으로 로버트 C. 마틴(Robert C. Martin)은 객체 지향 설계의 다섯 가지 핵심 원칙, SOLID를 정립하였다.5

SOLID는 단일 책임 원칙(Single Responsibility Principle), 개방-폐쇄 원칙(Open-Closed Principle), 리스코프 치환 원칙(Liskov Substitution Principle), 인터페이스 분리 원칙(Interface Segregation Principle), 의존성 역전 원칙(Dependency Inversion Principle)의 첫 글자를 딴 약어이다.8 이 중 두 번째 원칙인 개방-폐쇄 원칙(Open-Closed Principle, 이하 OCP)은 변화를 관리하는 소프트웨어 설계의 핵심 철학을 담고 있으며, 많은 전문가들에 의해 객체 지향 설계의 가장 중요한 원칙 중 하나로 간주된다.4

OCP의 정의는 다음과 같다: "소프트웨어 개체(클래스, 모듈, 함수 등)는 확장에 대해 열려 있어야 하고, 수정에 대해서는 닫혀 있어야 한다(Software entities should be open for extension, but closed for modification)".10 이 정의는 언뜻 보기에 모순적으로 들린다. 어떻게 하나의 개체가 변화를 허용하면서('열려 있으면서') 동시에 변화를 거부할 수('닫혀 있으면서') 있는가? 이 역설적인 두 가지 목표를 동시에 달성하는 방법에 대한 탐구가 바로 OCP의 정수이다. '수정에 닫혀있다'는 것은 이미 검증되고 안정적으로 동작하는 기존 코드를 변경함으로써 발생할 수 있는 잠재적 버그와 예측 불가능한 부작용을 원천적으로 차단하려는 안정성 지향의 방어적 자세를 의미한다.11 반면, '확장에 열려있다'는 것은 새로운 기능 추가나 행위 변경과 같은 미래의 요구사항을 시스템의 근간을 흔들지 않고 수용하려는 진화 지향의 진보적 자세를 의미한다.11 OCP는 이 두 가지 상반된 가치를 조화시키는 설계 철학이며, 단순히 특정 코딩 기법을 넘어 미래의 불확실성에 대처하는 시스템 아키텍처의 근본적인 자세를 규정한다.

본 보고서는 OCP의 기원과 역사적 진화를 추적하고, 그 작동 원리를 가능하게 하는 핵심 메커니즘을 분석하며, 디자인 패턴을 통한 구체적 구현 방법론을 탐구한다. 나아가 다른 SOLID 원칙과의 유기적 관계를 조명하고, OCP에 대한 비판적 시각과 현실 세계에서의 실용적 적용 방안을 고찰할 것이다. 최종적으로 OCP가 현대 소프트웨어 아키텍처, 특히 플러그인 시스템의 근간을 이루는 핵심 사상임을 밝힘으로써, 지속 가능한 소프트웨어 설계를 위한 OCP의 현대적 의의를 심도 있게 논하고자 한다.

## 1. OCP의 기원과 진화: 마이어의 이상과 마틴의 현실

개방-폐쇄 원칙은 고정불변의 법칙이 아니라, 소프트웨어 공학의 발전과 함께 그 의미와 적용 방식이 진화해 온 살아있는 개념이다. OCP의 개념적 여정을 이해하기 위해서는 버트런드 마이어(Bertrand Meyer)가 제시한 초기 개념과 로버트 C. 마틴에 의해 재해석된 현대적 개념의 차이를 명확히 구분해야 한다. 이 패러다임의 전환은 OCP에 대한 이해의 핵심을 이룬다.

### 1.1 버트런드 마이어의 1988년 초기 정의

OCP라는 용어는 1988년 버트런드 마이어가 그의 저서 "객체 지향 소프트웨어 구성(Object-Oriented Software Construction)"에서 처음으로 제시하였다.10 마이어의 시대적 배경은 객체 지향 프로그래밍의 초기 단계로, 상속을 통한 코드 재사용이 핵심적인 가치로 여겨지던 시기였다. 그의 OCP 정의는 이러한 맥락에서 이해되어야 한다.

마이어에게 있어 **'닫힘(Closed)'**이란, 하나의 모듈이 성공적으로 컴파일되어 라이브러리 형태로 저장되고, 다른 클라이언트 모듈들이 안정적으로 사용할 수 있는 상태를 의미했다.10 이는 일단 완성되어 배포된 모듈의 소스 코드는 버그 수정을 제외하고는 변경되어서는 안 된다는 강력한 안정성의 요구를 담고 있다. 이는 데이비드 파르나스(David Parnas)의 정보 은닉(information hiding) 개념과도 맞닿아 있으며, 모듈의 인터페이스가 안정적인 계약으로서 기능해야 함을 강조한다.10

반면, **'열림(Open)'**은 이러한 안정성을 해치지 않으면서 모듈의 기능을 확장할 수 있는 가능성을 의미했다. 마이어는 이를 달성하기 위한 핵심 메커니즘으로 **구현 상속(implementation inheritance)**을 제시했다.10 즉, 기존의 클래스(부모 클래스)를 수정하는 대신, 이 클래스를 상속받는 새로운 클래스(자식 클래스)를 만들어 새로운 기능을 추가하거나 기존 기능을 변경(오버라이드)함으로써 시스템을 확장할 수 있다는 것이다.10 이 방식은 이미 존재하는 코드를 재사용하면서 새로운 요구사항에 대응할 수 있는 직관적인 방법을 제공했다.

하지만 마이어의 접근법은 근본적인 한계를 내포하고 있었다. 구현 상속은 부모 클래스의 내부 구현과 자식 클래스 사이에 강한 결합(tight coupling)을 형성한다.9 만약 부모 클래스의 내부 구현이 변경되면, 이를 상속받는 모든 자식 클래스들이 영향을 받아 연쇄적으로 수정되어야 할 위험이 존재했다. 이는 '수정에 닫혀있다'는 OCP의 근본적인 목표를 오히려 저해하는 역설적인 결과를 낳을 수 있었다.

### 1.2 로버트 C. 마틴의 1990년대 재해석

1990년대에 들어서면서 복잡한 소프트웨어 시스템의 유지보수와 진화가 개발의 주요 난제로 부상했다. 이러한 배경 속에서 로버트 C. 마틴은 1996년 자신의 논문 "The Open-Closed Principle"을 통해 OCP를 현대적인 관점에서 재해석하였고, 이 해석이 오늘날 OCP의 표준적인 의미로 받아들여지고 있다.3

마틴의 가장 중요한 기여는 OCP 구현의 핵심 메커니즘을 구현 상속에서 **추상화(abstraction)**로 전환한 것이다.3 그는 추상 클래스(abstract class)나 인터페이스(interface)를 통해 OCP를 달성하는 방식을 제창했다.

마틴의 관점에서 **'닫힘(Closed)'**의 대상은 구체적인 클래스의 소스 코드가 아니라, 시스템의 핵심적인 부분을 정의하는 **안정된 추상 인터페이스**이다. 이 인터페이스는 한번 정의되고 클라이언트들이 사용하기 시작하면, 가급적 변경되어서는 안 된다. 클라이언트 코드는 오직 이 추상 인터페이스에만 의존하므로, 인터페이스를 구현하는 구체적인 클래스들이 어떻게 변경되거나 추가되더라도 클라이언트 코드는 영향을 받지 않는다.

그리고 **'열림(Open)'**은 이 추상 인터페이스를 구현하는 새로운 구체 클래스(concrete class)를 추가함으로써 시스템의 동작을 **다형적(polymorphically)**으로 확장하는 것을 의미한다.10 클라이언트는 추상 인터페이스 타입으로 객체를 다루기 때문에, 런타임에 어떤 구체 클래스의 인스턴스가 제공되더라도 동일한 방식으로 상호작용할 수 있다. 새로운 기능이 필요하면, 새로운 구현 클래스를 만들어 시스템에 '플러그인'처럼 연결하기만 하면 된다.

이러한 마틴의 접근법은 추상화를 통해 클라이언트와 구현체 사이의 결합도를 극적으로 낮추고(loose coupling), 이를 통해 시스템의 유연성, 확장성, 테스트 용이성을 비약적으로 향상시켰다.18 마이어의 OCP가 '이미 만들어진 코드를 어떻게 효율적으로 재사용할 것인가'에 초점을 맞추었다면, 마틴의 OCP는 '미래의 불확실한 변화에 시스템이 어떻게 탄력적으로 대응할 것인가'라는 '변화 관리'와 '리스크 관리'의 관점으로 패러다임을 전환시켰다. 이는 소프트웨어 공학의 주요 관심사가 정적인 코드 재사용에서 동적인 시스템 진화로 이동했음을 보여주는 상징적인 변화라 할 수 있다.

다음 표는 마이어와 마틴의 OCP 해석에 대한 핵심적인 차이를 요약하여 보여준다.

| 구분              | 버트런드 마이어 (1988)                             | 로버트 C. 마틴 (1996)                                        |
| ----------------- | -------------------------------------------------- | ------------------------------------------------------------ |
| **핵심 목표**     | 라이브러리화된 코드의 재사용 및 안정성 확보        | 변화에 대한 유연성 및 유지보수성 극대화                      |
| **주요 메커니즘** | 구현 상속 (Implementation Inheritance)             | 추상화 (Interfaces, Abstract Classes)와 다형성 (Polymorphism) |
| **'닫힘'의 대상** | 컴파일된 구체 클래스의 소스 코드                   | 안정화된 추상 인터페이스의 명세                              |
| **'열림'의 방식** | 기존 클래스를 상속받는 **새로운 자식 클래스** 생성 | 인터페이스를 구현하는 **새로운 구체 클래스** 생성            |
| **장점**          | 코드 재사용                                        | 낮은 결합도, 높은 유연성, 테스트 용이성                      |
| **단점/한계**     | 강한 결합(Tight Coupling) 유발, 상속 계층의 복잡화 | 과도한 추상화 가능성, 디자인 복잡성 증가                     |

## 2. OCP의 작동 원리: 추상화와 다형성

현대적 의미의 OCP는 추상화(Abstraction)와 다형성(Polymorphism)이라는 객체 지향 프로그래밍의 두 가지 핵심 기둥 위에서 작동한다. 추상화는 '수정에 대해 닫힌' 방화벽을 구축하고, 다형성은 '확장에 대해 열린' 통로를 제공한다. 이 두 메커니즘이 어떻게 상호작용하여 OCP의 역설적인 목표를 달성하는지 구체적인 예제를 통해 심층적으로 분석한다.

### 2.1 추상화: '수정에 대해 닫히게' 만드는 방화벽

추상화는 시스템에서 변하는 부분과 변하지 않는 부분을 분리하는 가장 강력한 도구이다.21 OCP의 맥락에서 추상화는 클라이언트 코드와 구체적인 구현 세부사항 사이에 안정적인 계약(contract)을 설정하는 역할을 한다. 클라이언트는 변화 가능성이 높은 구체적인 구현 클래스에 직접 의존하는 대신, 변화 가능성이 낮은 안정적인 추상 인터페이스에 의존한다.3

이 추상 인터페이스는 일종의 '방화벽'처럼 기능한다. 구현 세부사항이 변경되거나, 새로운 구현체가 추가되거나, 심지어 기존 구현체가 완전히 다른 것으로 교체되더라도, 클라이언트는 오직 약속된 인터페이스만을 바라보기 때문에 이러한 변화로부터 완벽하게 격리된다. 이것이 바로 '수정에 대해 닫혀 있는' 상태의 본질이다. 이 개념은 "추상화에 의존하라, 구체화에 의존하지 말라(Depend upon abstractions, [not] concretes)"고 말하는 의존성 역전 원칙(DIP)과 직접적으로 연결되며, OCP를 구조적으로 뒷받침하는 핵심 사상이다.5

### 2.2 다형성: '확장에 대해 열리게' 만드는 통로

다형성은 동일한 인터페이스를 따르는 여러 다른 타입의 객체들이 동일한 메시지(메서드 호출)에 대해 각자의 방식으로 응답할 수 있는 능력을 의미한다.13 클라이언트 코드는 추상 인터페이스 타입의 변수를 통해 객체와 상호작용한다. 런타임에 이 변수에 어떤 구체 클래스의 인스턴스가 할당되더라도, 클라이언트는 이를 알 필요 없이 일관된 방식으로 작업을 요청할 수 있다.

이것이 바로 '확장에 대해 열려 있는' 상태를 구현하는 핵심 메커니즘이다.24 시스템에 새로운 기능이나 행위를 추가해야 할 때, 개발자는 기존 코드를 한 줄도 수정할 필요 없이, 단지 약속된 추상 인터페이스를 구현하는 새로운 클래스를 추가하기만 하면 된다. 이렇게 추가된 새로운 클래스는 기존 클라이언트 코드에 의해 아무런 변경 없이 즉시 활용될 수 있다. 다형성은 새로운 기능이 시스템에 자연스럽게 통합될 수 있도록 하는 유연한 '통로' 역할을 수행한다.

### 2.3 심층 분석 예제: '도형 그리기(Shape Drawer)' 문제

OCP의 작동 원리를 가장 명확하게 보여주는 고전적인 예제는 다양한 종류의 도형을 그리는 시스템을 구현하는 것이다. OCP를 위반한 설계와 준수한 설계가 새로운 요구사항에 어떻게 다르게 반응하는지 비교 분석한다.

#### 2.3.1 OCP 위반 설계 (절차적/조건 분기 방식)

OCP를 고려하지 않은 초기 설계는 종종 중앙 집중적인 제어 구조를 갖는다. `Drawer`라는 클래스가 모든 도형을 그리는 책임을 지며, 내부적으로 도형의 타입을 확인하여 적절한 그리기 로직을 호출하는 방식이다.

```Java
// OCP 위반 코드
public enum ShapeType { CIRCLE, SQUARE }

public class Shape {
    public ShapeType type;
    // 도형별 데이터...
}

public class Circle extends Shape {
    public double radius;
    public Point center;
    public Circle() { this.type = ShapeType.CIRCLE; }
}

public class Square extends Shape {
    public double side;
    public Point topLeft;
    public Square() { this.type = ShapeType.SQUARE; }
}

public class Drawer {
    public void drawAllShapes(List<Shape> shapes) {
        for (Shape shape : shapes) {
            if (shape.type == ShapeType.CIRCLE) {
                drawCircle((Circle) shape);
            } else if (shape.type == ShapeType.SQUARE) {
                drawSquare((Square) shape);
            }
            // 새로운 도형 'Triangle' 추가 시, 이 부분에 else if 블록을 추가해야 함 (수정 발생)
        }
    }

    private void drawCircle(Circle c) { /* 원 그리기 로직 */ }
    private void drawSquare(Square s) { /* 사각형 그리기 로직 */ }
}
```

이 설계의 치명적인 문제점은 새로운 요구사항에 극도로 취약하다는 것이다. 만약 '삼각형(Triangle)'을 그리는 기능을 추가해야 한다면, 개발자는 다음과 같은 일련의 수정을 강요받는다.8

1. `ShapeType` 열거형에 `TRIANGLE`을 추가해야 한다.
2. `Triangle` 클래스를 새로 만들어야 한다.
3. **가장 결정적으로, `Drawer` 클래스의 `drawAllShapes` 메서드 내부에 `else if (shape.type == ShapeType.TRIANGLE)` 분기문을 추가하고, `drawTriangle` 메서드를 구현해야 한다.**

`Drawer` 클래스는 이미 테스트를 거쳐 안정적으로 동작하던 코드였지만, 새로운 기능을 '확장'하기 위해 기존 코드를 '수정'해야만 했다. 이는 '수정에 닫혀 있어야 한다'는 OCP의 핵심 원칙을 명백하게 위반하는 행위이며, 이러한 수정은 새로운 버그를 유발할 위험을 내포한다.3

#### 2.3.2 OCP 준수 설계 (객체 지향적/다형성 방식)

OCP를 준수하는 설계는 '어떤 도형을 어떻게 그릴지'에 대한 중앙 집중적인 제어 로직을 각 도형 객체 자신에게로 분산시킨다. 이는 추상화와 다형성을 통해 이루어진다.

1. 먼저, 모든 도형이 따라야 할 공통 계약인 `Shape` 인터페이스를 정의한다. 이 인터페이스는 `draw()`라는 추상 메서드를 선언한다.
2. 각각의 구체적인 도형 클래스(`Circle`, `Square` 등)는 `Shape` 인터페이스를 구현하고, `draw()` 메서드 내에 자신을 그리는 로직을 책임지고 구현한다.
3. `Drawer` 클래스는 더 이상 개별 도형의 구체적인 타입을 알 필요가 없다. 오직 `Shape` 인터페이스에만 의존하며, 주어진 `Shape` 객체의 `draw()` 메서드를 호출하는 책임만 진다.

```Java
// OCP 준수 코드
public interface Shape {
    void draw();
}

public class Circle implements Shape {
    private double radius;
    private Point center;
    // 생성자...
    @Override
    public void draw() { /* 원 그리기 로직 */ }
}

public class Square implements Shape {
    private double side;
    private Point topLeft;
    // 생성자...
    @Override
    public void draw() { /* 사각형 그리기 로직 */ }
}

public class Drawer {
    public void drawAllShapes(List<Shape> shapes) {
        for (Shape shape : shapes) {
            shape.draw(); // 어떤 구체적인 도형인지 알 필요 없이, 다형적으로 draw() 호출
        }
    }
}
```

이 설계에서 '삼각형(Triangle)'을 추가하는 과정은 이전과 극명하게 달라진다.

1. `Shape` 인터페이스를 구현하는 `Triangle` 클래스를 **새로 추가**한다.

   ```Java
   public class Triangle implements Shape {
       // 필드 및 생성자...
       @Override
       public void draw() { /* 삼각형 그리기 로직 */ }
   }
   ```

이것으로 모든 작업이 끝난다. 기존에 존재하던 `Drawer`, `Shape`, `Circle`, `Square` 클래스는 단 한 줄의 코드도 수정할 필요가 없다. `Drawer` 클래스는 '수정'에 완벽하게 '닫혀' 있으면서도, 새로운 `Triangle` 클래스의 추가를 통해 시스템의 기능이 '확장'되었다. 이것이 바로 OCP를 완벽하게 준수하는 설계이다.3

이러한 설계 변화의 본질은 **책임의 이동(Responsibility Shift)**에 있다. OCP 위반 코드에서는 '어떤 도형을 어떻게 그릴지 결정하는 책임'이 `Drawer`라는 중앙 제어 모듈에 집중되어 있었다. 반면, OCP 준수 코드에서는 이 책임이 각 `Shape` 구현체 객체 자신에게로 위임되었다. `Drawer`는 이제 '그리라는 명령을 전달하는 책임'만 가질 뿐, '어떻게 그릴지'에 대한 구체적인 지식은 각 도형 객체가 스스로 책임진다. 이처럼 중앙 집중화된 제어 로직을 분산된 객체의 책임으로 전환하는 것이 OCP를 구현하는 핵심적인 설계 변환 과정이다.

## 3. IV. 디자인 패턴을 활용한 OCP의 구체화

개방-폐쇄 원칙은 추상적인 설계 사상에 머무르지 않고, 잘 알려진 디자인 패턴들을 통해 실제 코드에서 구체적인 형태로 구현될 수 있다. 특히 전략 패턴(Strategy Pattern)과 템플릿 메서드 패턴(Template Method Pattern)은 OCP를 구현하는 대표적인 예시로, 각각 합성(Composition)과 상속(Inheritance)이라는 다른 메커니즘을 통해 OCP의 목표를 달성하는 방법을 보여준다.

### 3.1 전략 패턴: 행위의 동적 교체

전략 패턴은 알고리즘(전략)의 계열을 각각의 클래스로 캡슐화하고, 이들을 상호 교체 가능하게 만드는 행위 디자인 패턴이다.27 클라이언트의 요청을 처리하는 컨텍스트(Context) 객체는 구체적인 알고리즘 구현에 직접 의존하지 않고, 모든 전략이 구현해야 하는 공통 인터페이스에만 의존한다. 실제 사용할 구체적인 전략 객체는 런타임에 외부로부터 컨텍스트에 주입(inject)된다.

이 구조는 OCP를 이상적으로 만족시킨다. 컨텍스트 클래스는 새로운 전략(알고리즘)이 추가되더라도 전혀 수정될 필요가 없으므로 **'수정에 닫혀'** 있다. 새로운 전략은 공통 인터페이스를 구현하는 새로운 클래스를 추가하는 것만으로 시스템에 통합될 수 있으므로 **'확장에 열려'** 있다.27 전략 패턴의 핵심은 상속이 아닌  **객체 합성**을 통해 행위를 유연하게 확장하고 변경하는 데 있다.

#### 3.1.1 코드 예시: 결제 시스템

다양한 결제 수단을 처리하는 결제 시스템을 예로 들어 전략 패턴을 통한 OCP 구현을 살펴본다.

1. **전략 인터페이스 정의:** 모든 결제 방식이 구현해야 할 `PaymentStrategy` 인터페이스를 정의한다.

   ```Java
   public interface PaymentStrategy {
       void pay(int amount);
   }
   ```

2. **구체적인 전략 구현:** 각 결제 수단에 대한 구체적인 클래스를 구현한다.

   ```Java
   public class CreditCardPayment implements PaymentStrategy {
       private String cardNumber;
       // 생성자...
       @Override
       public void pay(int amount) {
           System.out.println("Paid " + amount + " using Credit Card.");
       }
   }
   
   public class PaypalPayment implements PaymentStrategy {
       private String email;
       // 생성자...
       @Override
       public void pay(int amount) {
           System.out.println("Paid " + amount + " using PayPal.");
       }
   }
   ```

3. **컨텍스트 클래스 구현:** `PaymentProcessor`는 `PaymentStrategy` 인터페이스에만 의존하며, 실제 결제 로직을 해당 전략 객체에 위임한다.

   ```Java
   public class PaymentProcessor {
       private PaymentStrategy paymentStrategy;
   
       // 런타임에 전략을 설정(주입)받음
       public void setPaymentStrategy(PaymentStrategy paymentStrategy) {
           this.paymentStrategy = paymentStrategy;
       }
   
       public void processOrder(int amount) {
           // 구체적인 결제 방식을 알 필요 없이, 인터페이스의 메서드만 호출
           paymentStrategy.pay(amount);
       }
   }
   ```

이 시스템에 새로운 결제 수단으로 '비트코인 결제'를 추가해야 할 경우, 개발자는 `PaymentStrategy` 인터페이스를 구현하는 `BitcoinPayment` 클래스를 새로 작성하기만 하면 된다. 기존의 `PaymentProcessor`, `CreditCardPayment`, `PaypalPayment` 클래스는 전혀 수정할 필요가 없다.11 이처럼 전략 패턴은 행위의 변화를 완벽하게 캡슐화하여 OCP를 준수하는 유연한 설계를 가능하게 한다.

### 3.2 템플릿 메서드 패턴: 골격은 유지, 단계는 확장

템플릿 메서드 패턴은 알고리즘의 전체적인 골격(skeleton)을 슈퍼클래스의 하나의 메서드(템플릿 메서드)에 정의하고, 알고리즘의 특정 단계들은 서브클래스에서 구현하도록 위임하는 행위 디자인 패턴이다.31 템플릿 메서드는 일반적으로 `final`로 선언되어 서브클래스가 알고리즘의 구조 자체를 변경하는 것을 막는다.

이 패턴 역시 OCP를 잘 구현한다. 알고리즘의 전체적인 흐름과 변하지 않는 부분은 슈퍼클래스의 템플릿 메서드에 정의되어 있으므로 **'수정에 닫혀'** 있다. 반면, 알고리즘의 세부적인 단계들은 새로운 서브클래스를 만들고 추상 메서드를 구현하거나 훅(hook) 메서드를 오버라이드함으로써 얼마든지 다르게 구현될 수 있으므로 **'확장에 열려'** 있다.34 템플릿 메서드 패턴의 핵심은 **클래스 상속**을 통해 알고리즘의 일부를 확장하는 것이다.

#### 3.2.1 코드 예시: 음료 제조 과정

차(Tea)와 커피(Coffee)를 만드는 과정을 예로 들어 템플릿 메서드 패턴을 살펴본다. 두 음료는 '물을 끓인다', '내용물을 우려낸다', '컵에 따른다', '첨가물을 넣는다'는 공통적인 단계를 갖지만, '우려내는 방식'과 '첨가물'에서 차이가 있다.

1. **추상 슈퍼클래스 및 템플릿 메서드 정의:** `BeverageMaker` 추상 클래스에 알고리즘의 골격인 `makeBeverage()` 템플릿 메서드를 정의한다.

   ```Java
   public abstract class BeverageMaker {
       // 템플릿 메서드: 알고리즘의 골격을 정의하며, 변경을 막기 위해 final로 선언
       public final void makeBeverage() {
           boilWater();
           brew();
           pourInCup();
           addCondiments();
       }
   
       // 모든 서브클래스에서 공통으로 사용되는 구체 메서드
       private void boilWater() { System.out.println("Boiling water"); }
       private void pourInCup() { System.out.println("Pouring into cup"); }
   
       // 서브클래스에서 반드시 구현해야 하는 추상 메서드 (변하는 부분)
       protected abstract void brew();
       protected abstract void addCondiments();
   }
   ```

2. **구체적인 서브클래스 구현:** `BeverageMaker`를 상속받아 각 음료에 맞는 세부 단계를 구현한다.

   ```Java
   public class TeaMaker extends BeverageMaker {
       @Override
       protected void brew() { System.out.println("Steeping the tea"); }
       @Override
       protected void addCondiments() { System.out.println("Adding lemon"); }
   }
   
   public class CoffeeMaker extends BeverageMaker {
       @Override
       protected void brew() { System.out.println("Dripping coffee through filter"); }
       @Override
       protected void addCondiments() { System.out.println("Adding sugar and milk"); }
   }
   ```

이 시스템에 '핫초코'라는 새로운 음료를 추가해야 할 경우, 개발자는 `BeverageMaker`를 상속받는 `HotChocolateMaker` 클래스를 새로 만들고 `brew()`와 `addCondiments()` 메서드를 구현하기만 하면 된다. 기존의 `BeverageMaker`나 다른 서브클래스들은 전혀 수정할 필요가 없다.33 템플릿 메서드 패턴은 알고리즘의 일관성을 유지하면서 세부 구현의 다양성을 확보하는 방식으로 OCP를 효과적으로 지원한다.

결론적으로, 전략 패턴과 템플릿 메서드 패턴은 OCP라는 동일한 목표를 달성하기 위한 상보적인 두 가지 접근법을 대표한다. 전략 패턴은 '무엇을 할 것인가(What)'라는 알고리즘 전체를 객체 합성(Composition)을 통해 동적으로 교체함으로써 유연성을 극대화한다. 반면, 템플릿 메서드 패턴은 '어떻게 할 것인가(How)'의 전체적인 흐름은 클래스 상속(Inheritance)을 통해 고정하되, 그 안의 특정 '단계들(Steps)'을 위임함으로써 구조적 안정성과 확장성을 동시에 제공한다. 어떤 패턴을 선택할지는 변화의 단위가 알고리즘 전체인지, 아니면 알고리즘의 일부인지에 대한 설계자의 판단에 달려 있다.

## 4. V. OCP와 다른 SOLID 원칙의 시너지

개방-폐쇄 원칙은 독립적으로 존재하는 고립된 규칙이 아니라, 다른 SOLID 원칙들과 유기적으로 상호작용하며 그 가치가 극대화되는 시스템의 일부이다. 특히 리스코프 치환 원칙(LSP)과 의존성 역전 원칙(DIP)은 OCP를 가능하게 하고 그 구조를 강화하는 데 필수적인 역할을 한다. 이들 원칙 간의 시너지를 이해하는 것은 견고하고 유연한 객체 지향 설계를 구축하는 데 매우 중요하다.

### 4.1 리스코프 치환 원칙 (LSP): 신뢰성의 기반

리스코프 치환 원칙(LSP)은 바바라 리스코프(Barbara Liskov)에 의해 정의되었으며, 그 내용은 "S 타입의 객체 o1 각각에 대해 T 타입의 객체 o2가 하나씩 존재하여, T 타입을 이용해서 정의한 모든 프로그램 P에서 o2를 o1으로 치환하더라도 P의 행위가 변하지 않는다면, S는 T의 서브타입이다"로 요약될 수 있다.36 더 간단히 말해, **"서브타입은 언제나 그것의 기반 타입으로 교체될 수 있어야 한다"**는 것이다.5 이는 자식 클래스가 부모 클래스의 역할을 대체할 때, 프로그램의 정확성이나 행위의 일관성을 해치지 않아야 함을 의미한다.

OCP와 LSP의 관계는 매우 밀접하다. OCP는 다형성을 통해 '확장에 열려있는' 구조를 만든다. 클라이언트 코드는 기반 타입(추상 인터페이스 또는 추상 클래스)에만 의존하면서, 런타임에는 다양한 서브타입의 구체적인 객체들과 상호작용한다. 이 메커니즘이 원활하게 작동하기 위한 전제 조건이 바로 LSP이다.38

만약 어떤 서브타입이 LSP를 위반한다면, 즉 기반 타입으로 교체되었을 때 예기치 않은 동작을 하거나 예외를 발생시킨다면, 클라이언트 코드는 더 이상 안심하고 기반 타입에만 의존할 수 없게 된다. 클라이언트는 특정 서브타입을 구별하고 그에 대한 예외 처리를 하기 위해 `if (object instanceof SpecificSubType)`과 같은 타입 검사 코드를 추가해야만 한다.36 이러한 조건 분기문의 추가는 새로운 서브타입이 생길 때마다 클라이언트 코드의 '수정'을 유발하며, 이는 OCP를 정면으로 파괴하는 결과를 낳는다.37

예를 들어, `Rectangle` 클래스와 이를 상속하는 `Square` 클래스가 있다고 가정하자. `Rectangle`은 `setWidth`와 `setHeight` 메서드를 독립적으로 가진다. 그러나 `Square`는 정사각형의 속성을 유지하기 위해 `setWidth`를 호출하면 `height`도 함께 변경하고, `setHeight`를 호출하면 `width`도 함께 변경하도록 오버라이드했다. `Rectangle`을 기대하는 클라이언트 코드가 `setWidth(5)`와 `setHeight(4)`를 순차적으로 호출했을 때, `Square` 객체는 너비와 높이가 모두 4가 되는 예기치 않은 결과를 낳는다. 이는 LSP 위반이며, 이로 인해 클라이언트는 `Square` 타입을 특별히 처리해야 하므로 OCP도 함께 무너진다.36

결론적으로, LSP는 OCP가 의존하는 다형성이 '신뢰성 있게' 작동하기 위한 필수적인 품질 보증 장치이다. LSP가 지켜져야만 클라이언트는 안심하고 기반 타입을 통해 모든 서브타입을 일관되게 다룰 수 있으며, 비로소 '수정에 닫힌' 안정적인 상태를 유지할 수 있다.

### 4.2 의존성 역전 원칙 (DIP): 구조적 강화

의존성 역전 원칙(DIP)은 두 가지 핵심 명제로 구성된다: **"고수준 모듈은 저수준 모듈에 의존해서는 안 된다. 둘 모두 추상화에 의존해야 한다"** 그리고 **"추상화는 세부 사항에 의존해서는 안 된다. 세부 사항이 추상화에 의존해야 한다"**.5 여기서 고수준 모듈은 시스템의 핵심 비즈니스 로직이나 정책을 담고 있는 부분을, 저수준 모듈은 데이터베이스 접근, 파일 입출력 등 구체적인 구현 세부사항을 다루는 부분을 의미한다. DIP는 전통적인 하향식 의존성 흐름(고수준 → 저수준)을 '역전'시켜, 모든 모듈이 중앙의 안정적인 추상화 계층에 의존하도록 구조를 재편한다.

DIP와 OCP의 관계는 더욱 직접적이다. OCP를 올바르게 구현하기 위한 전제 조건은 클라이언트(고수준 모듈)가 변화하기 쉬운 구체적인 구현체(저수준 모듈)가 아닌, 안정적인 추상 인터페이스에 의존해야 한다는 것이다. 이는 DIP의 첫 번째 명제와 정확히 일치한다.8 즉, DIP는 OCP를 개별 클래스 설계를 넘어 시스템 전체의 아키텍처 수준으로 확장하고 구조화하는 원칙이라고 볼 수 있다.41

DIP를 따르는 설계는 자연스럽게 OCP를 만족시키는 구조를 갖게 된다. 고수준 모듈과 저수준 모듈 사이에 추상 인터페이스라는 '확장 포인트'가 마련되기 때문이다. 저수준 모듈의 구현이 변경되거나 완전히 새로운 구현으로 교체되더라도, 추상 인터페이스만 동일하게 구현한다면 고수준 모듈은 전혀 수정할 필요가 없다. 이는 OCP의 정의와 완벽하게 부합한다. 의존성 주입(Dependency Injection)과 같은 구체적인 기술 패턴은 DIP를 구현하는 효과적인 수단이며, 결과적으로 OIP를 만족하는 유연하고 테스트하기 쉬운 시스템을 구축하는 데 결정적인 역할을 한다.41

결론적으로 SOLID 원칙들은 개별적인 규칙의 집합이 아니라, '유연하고 유지보수 가능한 시스템'이라는 하나의 공동 목표를 향해 상호 보완적으로 작용하는 유기적인 시스템이다. 이 시스템 안에서 **DIP**는 의존성의 방향을 제어하여 '추상화에 의존하라'는 거시적인 아키텍처의 청사진을 제공한다. **OCP**는 그 청사진 위에서 '수정 없이 확장하라'는 미시적인 설계의 행동 지침을 제시한다. 그리고 **LSP**는 그 확장이 '안전하고 예측 가능하게' 이루어지도록 보장하는 구현의 품질 기준을 제공한다. 이 세 원칙은 각각 아키텍처, 설계, 구현의 관점에서 동일한 목표를 지원하며, 하나가 무너지면 다른 원칙들도 온전히 지켜지기 어려운 긴밀한 관계를 형성한다.

## 5. VI. 비판적 시각: OCP의 그림자와 실용적 균형

개방-폐쇄 원칙은 강력하고 이상적인 설계 목표를 제시하지만, 현실 세계의 프로젝트에 맹목적으로 적용될 경우 오히려 해가 될 수 있다. OCP를 교조적으로 추종할 때 발생하는 과잉 설계의 함정, 다른 실용주의적 원칙과의 충돌, 그리고 리팩토링과의 관계에 대한 비판적 검토는 OCP를 현명하게 사용하기 위한 필수적인 과정이다.

### 5.1 과잉 설계의 함정: 시기상조 추상화

OCP를 준수하기 위해서는 변화가 예상되는 지점을 식별하고 그 주변에 추상화 계층(인터페이스, 추상 클래스)을 구축해야 한다. 그러나 미래에 발생할 모든 변화를 정확하게 예측하는 것은 사실상 불가능하다.42 이러한 예측에 기반한 설계는 '시기상조 추상화(Premature Abstraction)'라는 과잉 설계의 함정에 빠지기 쉽다.43

개발자가 아직 구체적인 요구사항이 없음에도 불구하고 미래의 확장을 대비해 불필요한 인터페이스와 복잡한 클래스 계층 구조를 만드는 것은 여러 문제를 야기한다.

첫째, **디자인 복잡성**이 증가한다. 단순히 구현해도 될 코드를 여러 클래스와 인터페이스로 분리하면 전체적인 코드 구조를 이해하고 추적하기 어려워진다.42

둘째, **코드 오버헤드**가 발생한다. 더 많은 클래스와 파일을 관리해야 하며, 이는 시스템 성능에 미미한 영향을 주거나 개발 속도를 저하시킬 수 있다.42

셋째, **테스팅 및 디버깅**이 복잡해진다. 추상화 계층이 늘어날수록 의존 관계가 복잡해져 문제의 원인을 파악하기 어려워질 수 있다.42

더욱 심각한 문제는, 잘못 예측된 추상화는 미래의 변화를 돕기는커녕 오히려 방해하는 족쇄가 될 수 있다는 점이다.44 실제 요구사항이 예측과 다른 방향으로 전개될 경우, 기존의 잘못된 추상화를 수정하거나 우회하기 위해 더 큰 비용을 치러야 할 수도 있다.

### 5.2 YAGNI 원칙과의 충돌

YAGNI(You Aren't Gonna Need It)는 "지금 당장 필요하지 않은 기능은 만들지 말라"는 애자일 개발의 핵심 원칙 중 하나이다.45 이는 미래의 불확실한 요구사항을 위해 미리 코드를 작성하는 행위를 낭비로 간주하고, 현재의 명확한 요구사항에 집중할 것을 강조한다.

이러한 YAGNI의 관점은 미래의 확장을 대비해 미리 추상화 계층을 구축하라고 권장하는 OCP와 정면으로 충돌하는 것처럼 보인다.46 OCP는 미래의 변화를 '예측'하고 대비하라고 말하는 반면, YAGNI는 미래 예측을 '금지'하고 현재에 집중하라고 말한다. 이 둘 사이의 긴장 관계는 모든 개발자가 마주하는 현실적인 딜레마이다.47

이 충돌을 해결하기 위한 실용적인 접근법은 OCP의 적용 범위를 **맥락에 따라 전략적으로 조절**하는 것이다.

- **내부용 코드 / 단일 팀 개발 환경:** 이 환경에서는 YAGNI가 OCP보다 우선될 수 있다. 코드베이스 전체에 대한 통제권이 팀 내부에 있으므로, 변화가 필요할 때 리팩토링을 통해 구조를 개선하는 비용이 상대적으로 낮다. 따라서 처음에는 가장 단순하게 구현하고(KISS 원칙), 동일한 종류의 변경이 반복적으로 발생하여 변화의 축이 명확해지면 그 때 OCP를 적용하여 리팩토링하는 것이 효율적이다. 이는 '한 번 속는 건 괜찮지만, 두 번은 안 된다(Fool me once, shame on you; fool me twice, shame on me)'는 접근법과 일치한다.15
- **외부 공개 API / 라이브러리 / 프레임워크:** 이 환경에서는 OCP가 YAGNI보다 우선되어야 한다. 한번 공개된 API는 수많은 클라이언트 코드에 영향을 미치므로 수정하기가 매우 어렵다(하위 호환성 문제). 따라서 API 설계 단계에서 신중한 예측을 통해 향후 발생 가능성이 높은 확장을 위한 '확장 포인트'를 미리 설계해 두는 것이 필수적이다. 이 경우, OCP는 외부 개발자들이 라이브러리를 수정하지 않고도 원하는 기능을 추가할 수 있도록 하는 핵심적인 가이드라인이 된다.47

### 5.3 리팩토링과 OCP 위반의 경계

OCP를 '기존 코드를 절대 수정해서는 안 된다'는 경직된 규칙으로 오해해서는 안 된다. 버그 수정은 당연히 허용되는 수정 행위이다.26 더 중요한 것은, 코드의 가독성, 유지보수성, 구조적 건전성을 향상시키기 위해 기존 코드의 내부 구현을 변경하는 **리팩토링(Refactoring)**은 OCP 위반이 아니라는 점이다.

오히려, OCP를 준수하지 않는 설계를 OCP를 준수하도록 개선하는 과정 자체가 리팩토링의 중요한 활동 중 하나이다.50 예를 들어, 앞서 '도형 그리기' 예제에서 `if-else` 구조를 가진 `Drawer` 클래스를 `Shape` 인터페이스와 다형성을 사용하도록 변경하는 것은 OCP를 달성하기 위한 리팩토링이다. OCP는 새로운 '기능'을 추가할 때 기존 코드를 수정하지 말라는 것이지, 코드의 '구조' 자체를 더 나은 방향으로 개선하는 행위를 금지하는 원칙이 아니다.

### 5.4 실용적 OCP 적용 가이드라인

이러한 비판적 시각을 바탕으로, OCP를 현실 세계에 현명하게 적용하기 위한 몇 가지 가이드라인을 제시할 수 있다.

1. **변화 예측이 아닌, 변화 격리에 집중하라:** 미래의 모든 변화를 예측하려는 시도는 실패하기 쉽다. 대신, 변화가 발생했을 때 그 영향이 시스템의 다른 부분으로 퍼져나가지 않도록 '격리'하는 데 집중해야 한다. 이는 알리스터 콕번(Alistair Cockburn)이 제안한 '보호된 변화(Protected Variations)' 패턴의 핵심 사상과 일맥상통한다. 변화 가능성이 높은 부분을 식별하고 그 주위에 안정적인 인터페이스라는 방어벽을 치는 것이다.10
2. **세 번의 법칙(Rule of Three)을 활용하라:** 어떤 기능에 대해 유사한 종류의 변경 요청이 세 번 발생하기 전까지는 섣불리 추상화를 도입하지 않는 것이 좋다. 첫 번째 요청은 그냥 구현한다. 두 번째 유사한 요청이 왔을 때도 중복을 감수하고 구현할 수 있다. 하지만 세 번째 요청이 왔을 때, 이는 명백한 변화의 패턴을 의미하므로 이때 리팩토링을 통해 OCP를 적용하는 것이 시기상조 추상화를 피하는 좋은 경험 법칙이다.46
3. **KISS(Keep It Simple, Stupid) 원칙을 잊지 마라:** 항상 가장 단순한 해결책을 우선적으로 고려해야 한다. OCP를 적용하기 위한 추가적인 복잡성은 그로 인해 얻는 유연성과 유지보수성의 이득이 명백하게 비용을 상쇄할 때만 정당화될 수 있다.15

결론적으로, OCP는 모든 코드에 기계적으로 적용해야 하는 절대적인 '규칙'이 아니라, 우리가 지향해야 할 이상적인 '목표'로 이해해야 한다. 실제 설계 과정은 이 목표와 현재의 제약 조건(개발 시간, 비용, 요구사항의 명확성 등) 사이에서 최적의 균형점을 찾는 트레이드오프의 연속이다. 따라서 개발자는 "이 코드는 OCP를 위반했는가?"라고 묻기보다 **"이 코드에 OCP를 적용함으로써 얻는 이득이 복잡성 증가라는 비용을 능가하는가?"**라고 묻는 실용적인 자세를 견지해야 한다.

## 6. 결론: 플러그인 아키텍처의 초석이자 미래 지향적 설계 원칙

개방-폐쇄 원칙은 단순히 개별 클래스의 설계 지침을 넘어, 현대 소프트웨어 아키텍처를 관통하는 핵심 사상으로 확장된다. OCP의 정신을 가장 극적으로 구현한 형태가 바로 **플러그인(Plugin) 아키텍처**이며, 이를 통해 OCP가 지향하는 유연하고 진화 가능한 시스템의 비전을 엿볼 수 있다.

로버트 C. 마틴은 플러그인 시스템을 OCP의 '궁극적인 실현(ultimate consummation)'이자 '정점(apotheosis)'이라고 극찬했다.16 플러그인 아키텍처에서 시스템은 안정적인 핵심부(Core)와 확장 가능한 주변부(Plugins)로 명확히 분리된다. 시스템의 핵심부는 한번 안정화되면 가급적 수정되지 않으므로 **'수정에 닫혀'** 있다. 반면, 새로운 기능은 독립적으로 개발되고 배포될 수 있는 플러그인을 통해 시스템에 추가되므로 **'확장에 열려'** 있다.

이러한 구조를 가능하게 하는 핵심은 의존성의 방향이다. 플러그인은 시스템의 핵심부가 제공하는 안정적인 API(확장 포인트)에 의존하지만, 시스템의 핵심부는 개별 플러그인의 존재 자체를 알지 못한다. 즉, 의존성이 항상 주변부에서 핵심부로 향한다. 이는 의존성 역전 원칙(DIP)을 시스템 아키텍처 전체 수준에서 적용한 결과이다.16 Eclipse, Visual Studio Code와 같은 통합 개발 환경, Chrome이나 Firefox와 같은 웹 브라우저의 확장 프로그램, 심지어 Minecraft와 같은 게임의 모드(Mod) 시스템에 이르기까지, 수많은 현대 소프트웨어가 이 플러그인 아키텍처를 통해 놀라운 수준의 확장성과 생명력을 확보하고 있다.16

이러한 관점은 OCP의 본질을 '소유권'과 '제어권'의 분리 원칙으로 재해석하게 한다. 시스템의 핵심부를 개발하는 팀은 시스템 전체의 안정성과 일관성을 유지할 '소유권'과 '제어권'을 갖는다. 반면, 플러그인을 개발하는 외부 개발자나 미래의 개발자는 핵심을 변경할 권한 없이, 약속된 확장 포인트를 통해서만 새로운 기능을 추가할 수 있는 제한된 '제어권'을 부여받는다. 이 명확한 권한의 분리는 대규모 개발팀 간의 협업을 원활하게 하고, 시스템이 예측 불가능한 방향으로 훼손되는 것을 막으며, 장기적인 진화를 가능하게 하는 사회-기술적(socio-technical) 기반을 제공한다.

본 보고서는 개방-폐쇄 원칙에 대한 다각적인 고찰을 통해 다음과 같은 핵심 사항들을 밝혔다.

- OCP는 버트런드 마이어의 상속 기반 재사용 개념에서 출발하여, 로버트 C. 마틴의 추상화 기반 변화 관리 개념으로 진화하였다.
- 추상화와 다형성은 OCP를 기술적으로 구현하는 핵심 메커니즘이며, 전략 패턴과 템플릿 메서드 패턴은 이를 구체화하는 대표적인 디자인 패턴이다.
- OCP는 단독으로 존재하지 않으며, 특히 LSP와 DIP와 같은 다른 SOLID 원칙들과의 강력한 시너지를 통해 그 힘이 완성된다.
- OCP의 실용적 적용은 YAGNI, KISS와 같은 다른 원칙들과의 균형을 요구하며, 모든 상황에 적용해야 하는 절대 규칙이 아닌, 맥락에 따라 전략적으로 추구해야 할 설계 목표이다.

마이크로서비스 아키텍처(MSA)나 서버리스 컴퓨팅과 같은 현대적인 분산 시스템 설계 패러다임에서도 OCP의 정신은 여전히 유효하다. 각 서비스는 독립적으로 배포되고 확장될 수 있는 '닫힌' 단위로 기능하며, 잘 정의된 API 계약을 통해 서로 '열린' 방식으로 상호작용하고 협력한다.

결론적으로, 개방-폐쇄 원칙은 변화를 피할 수 없는 소프트웨어의 숙명 앞에서, 그 변화를 통제 불가능한 비용과 리스크가 아닌, 시스템의 가치를 증진시키는 기회로 전환시키는 강력하고 미래 지향적인 설계 사상이다. 불확실성이 가득한 미래의 요구사항에 흔들리지 않고, 지속적으로 성장하고 진화하는 견고한 소프트웨어를 구축하고자 하는 모든 소프트웨어 공학도에게 OCP에 대한 깊이 있는 이해와 실용적인 적용 능력은 핵심적인 역량이 될 것이다.

#### **참고 자료**

1. Benefits of the Open/Closed Principle | by Andrew Vathanakamsang | Medium, 8월 15, 2025에 액세스, https://medium.com/@a.vathanaka/benefits-of-the-open-closed-principle-dc9284d47598
2. Open Close Principle (OCP) Explained | by Liu Xiaoyu - Medium, 8월 15, 2025에 액세스, https://medium.com/@tkgs.liu.xiaoyu/open-close-principle-ocp-explained-16a877e77e67
3. The Open-Closed Principle, 8월 15, 2025에 액세스, https://courses.cs.duke.edu/fall07/cps108/papers/ocp.pdf
4. Clarify the Open/Closed Principle - Software Engineering Stack Exchange, 8월 15, 2025에 액세스, https://softwareengineering.stackexchange.com/questions/19627/clarify-the-open-closed-principle
5. SOLID - Wikipedia, 8월 15, 2025에 액세스, https://en.wikipedia.org/wiki/SOLID
6. SOLID Principles - Medium, 8월 15, 2025에 액세스, https://medium.com/@dev.alisamir/solid-principles-60a5d575232b
7. Robert C. Martin's Principle Collection, 8월 15, 2025에 액세스, http://principles-wiki.net/collections:robert_c._martin_s_principle_collection
8. SOLID Design Principles Explained: Building Better Software Architecture - DigitalOcean, 8월 15, 2025에 액세스, https://www.digitalocean.com/community/conceptual-articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design
9. Open Closed Principle Tutorial with Java Coding Example for Beginners - YouTube, 8월 15, 2025에 액세스, https://www.youtube.com/watch?v=8uj6IcR8kL0
10. Open–closed principle - Wikipedia, 8월 15, 2025에 액세스, [https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle](https://en.wikipedia.org/wiki/Open–closed_principle)
11. What Is The Open/Close Principle? - ITU Online IT Training, 8월 15, 2025에 액세스, https://www.ituonline.com/tech-definitions/what-is-the-open-close-principle/
12. 개방-폐쇄 원칙 - 위키백과, 우리 모두의 백과사전, 8월 15, 2025에 액세스, [https://ko.wikipedia.org/wiki/%EA%B0%9C%EB%B0%A9-%ED%8F%90%EC%87%84_%EC%9B%90%EC%B9%99](https://ko.wikipedia.org/wiki/개방-폐쇄_원칙)
13. Open-Closed Principle: Everything You Need to Know When ..., 8월 15, 2025에 액세스, https://www.alooba.com/skills/concepts/programming/object-oriented-programming/open-closed-principle/
14. The Open/Closed Principle: Designing Software for Scalability and Maintainability - Medium, 8월 15, 2025에 액세스, https://medium.com/@prajwal_ahluwalia/the-open-closed-principle-designing-software-for-scalability-and-maintainability-fa7a82e4eb20
15. SOLID Design in C#: The Open-Close Principle (OCP) - NDepend ..., 8월 15, 2025에 액세스, https://blog.ndepend.com/solid-design-the-open-close-principle-ocp/
16. Uncle Bob on the Open-Closed Principle - Clean Coder Blog, 8월 15, 2025에 액세스, http://blog.cleancoder.com/uncle-bob/2014/05/12/TheOpenClosedPrinciple.html
17. Open-closed Principle - PatternForge, 8월 15, 2025에 액세스, https://patternforge.org/sw-eng-instrument/open-closed-principle/
18. The Open-Closed Principle. and what hides behind it | by Vadim Samokhin - Medium, 8월 15, 2025에 액세스, https://medium.com/@wrong.about/the-open-closed-principle-c3dc45419784
19. Open-Closed Principle - Chandra Sivaraman, 8월 15, 2025에 액세스, https://chandrasivaraman.com/blog/open-closed-principle
20. The Open/Closed Principle with Code Examples - Stackify, 8월 15, 2025에 액세스, https://stackify.com/solid-design-open-closed-principle/
21. [SOLID] 개방 폐쇄 원칙(OCP)이란? - 느리더라도 꾸준하게 - 티스토리, 8월 15, 2025에 액세스, https://steady-coding.tistory.com/378
22. [Go] 개방 폐쇄 원칙 - Jinwoo's Blog, 8월 15, 2025에 액세스, https://bugoverdose.github.io/development/ocp-in-go/
23. What is the meaning and reasoning behind the Open/Closed Principle? - Stack Overflow, 8월 15, 2025에 액세스, https://stackoverflow.com/questions/59016/what-is-the-meaning-and-reasoning-behind-the-open-closed-principle
24. Open/Close principle and polymorphism - Stack Overflow, 8월 15, 2025에 액세스, https://stackoverflow.com/questions/40754311/open-close-principle-and-polymorphism
25. OCP(Open-Close-Principle) 개방 폐쇄 원칙이란? - devoong2 - 티스토리, 8월 15, 2025에 액세스, [https://devoong2.tistory.com/entry/OCPOpen-Close-Principle-%EA%B0%9C%EB%B0%A9-%ED%8F%90%EC%87%84-%EC%9B%90%EC%B9%99%EC%9D%B4%EB%9E%80](https://devoong2.tistory.com/entry/OCPOpen-Close-Principle-개방-폐쇄-원칙이란)
26. Understanding the Open/Closed Principle | by Steven | Medium, 8월 15, 2025에 액세스, https://medium.com/@seungbae2/understanding-the-open-closed-principle-5cf440c3efdf
27. Strategy - Refactoring.Guru, 8월 15, 2025에 액세스, https://refactoring.guru/design-patterns/strategy
28. The Open/Closed Principle and Strategy Pattern - DZone, 8월 15, 2025에 액세스, https://dzone.com/articles/the-openclosed-principle
29. Strategy pattern - Wikipedia, 8월 15, 2025에 액세스, https://en.wikipedia.org/wiki/Strategy_pattern
30. Methods to Implement the Open/Closed Principle (OCP) in Your Code | by Anh Trần Tuấn, 8월 15, 2025에 액세스, https://medium.com/@tuananhbk1996/methods-to-implement-the-open-closed-principle-ocp-in-your-code-c0d52cfcf5ad
31. Template method pattern - Wikipedia, 8월 15, 2025에 액세스, https://en.wikipedia.org/wiki/Template_method_pattern
32. Template Method - Refactoring.Guru, 8월 15, 2025에 액세스, https://refactoring.guru/design-patterns/template-method
33. Template Method Design Pattern - GeeksforGeeks, 8월 15, 2025에 액세스, https://www.geeksforgeeks.org/system-design/template-method-design-pattern/
34. 템플릿 메소드(Template Method) 패턴 - 완벽 마스터하기 - Inpa Dev ‍ - 티스토리, 8월 15, 2025에 액세스, [https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%ED%85%9C%ED%94%8C%EB%A6%BF-%EB%A9%94%EC%86%8C%EB%93%9CTemplate-Method-%ED%8C%A8%ED%84%B4-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EC%9E%90](https://inpa.tistory.com/entry/GOF-💠-템플릿-메소드Template-Method-패턴-제대로-배워보자)
35. Template Method Pattern: Define the Flow, Customize the Steps | by Maxim Gorin | Medium, 8월 15, 2025에 액세스, https://maxim-gorin.medium.com/template-method-pattern-define-the-flow-customize-the-steps-027d5c3cfcc6
36. SOLID Principles in Programming: Understand With Real Life Examples - GeeksforGeeks, 8월 15, 2025에 액세스, https://www.geeksforgeeks.org/system-design/solid-principle-in-programming-understand-with-real-life-examples/
37. object oriented - LSP vs OCP / Liskov Substitution VS Open Close ..., 8월 15, 2025에 액세스, https://softwareengineering.stackexchange.com/questions/178488/lsp-vs-ocp-liskov-substitution-vs-open-close
38. A Solid Guide to SOLID Principles - Baeldung, 8월 15, 2025에 액세스, https://www.baeldung.com/solid-principles
39. Help me understand the difference between LSP and OCP : r/csharp - Reddit, 8월 15, 2025에 액세스, https://www.reddit.com/r/csharp/comments/13tv8sw/help_me_understand_the_difference_between_lsp_and/
40. A quick guide on SOLID Principles for Object-Oriented design - Pragmatic Ways, 8월 15, 2025에 액세스, https://pragmaticways.com/5-solid-principles-all-software-engineers-need-to-follow/
41. How Dependency inversion is an extension of OCP?, 8월 15, 2025에 액세스, https://softwareengineering.stackexchange.com/questions/365087/how-dependency-inversion-is-an-extension-of-ocp
42. Disadvantages of the Open/Closed Principle (OCP) | by Nozibul ..., 8월 15, 2025에 액세스, https://medium.com/@nozibulislamspi/disadvantages-of-the-open-closed-principle-ocp-655228024276
43. SOLID vs. Avoiding Premature Abstraction - Software Engineering Stack Exchange, 8월 15, 2025에 액세스, https://softwareengineering.stackexchange.com/questions/65690/solid-vs-avoiding-premature-abstraction
44. The Open-Closed Principle, in review | Jon Skeet's coding blog, 8월 15, 2025에 액세스, https://codeblog.jonskeet.uk/2013/03/15/the-open-closed-principle-in-review/
45. My favorite software development principles: SOLID, DRY, KISS, & more, 8월 15, 2025에 액세스, https://christopherklint.com/blog/favorite-software-development-principles-solid-dry-kiss
46. Open/Closed principle: the thin line between predicting the future and being provident | by Daniele Scillia (Dan The Dev) - Medium, 8월 15, 2025에 액세스, https://medium.com/dan-the-dev/open-closed-principle-the-thin-line-between-predicting-the-future-and-being-provident-f28c2adb7b0a
47. OCP vs YAGNI · Enterprise Craftsmanship, 8월 15, 2025에 액세스, https://enterprisecraftsmanship.com/posts/ocp-vs-yagni/
48. Issues with the Open/Closed Principle? - Stack Overflow, 8월 15, 2025에 액세스, https://stackoverflow.com/questions/24614035/issues-with-the-open-closed-principle
49. Is the Open/Closed Principle a good idea? - Stack Overflow, 8월 15, 2025에 액세스, https://stackoverflow.com/questions/1416476/is-the-open-closed-principle-a-good-idea
50. Refactoring and Open / Closed principle - Software Engineering Stack Exchange, 8월 15, 2025에 액세스, https://softwareengineering.stackexchange.com/questions/170547/refactoring-and-open-closed-principle
51. Open-closed principle considered harmful? : r/ExperiencedDevs - Reddit, 8월 15, 2025에 액세스, https://www.reddit.com/r/ExperiencedDevs/comments/vbi9im/openclosed_principle_considered_harmful/
52. The Open-Close principle is backwards and broken! | by Ifeora Okechukwu | Medium, 8월 15, 2025에 액세스, https://isocroft.medium.com/the-open-close-principle-is-backwards-and-broken-930646d92f8b
53. SOLID vs. YAGNI [closed] - Stack Overflow, 8월 15, 2025에 액세스, https://stackoverflow.com/questions/3921904/solid-vs-yagni