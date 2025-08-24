[견고한 소프트웨어](./index.md)
# DRY(Don't Repeat Yourself) 원칙


본 보고서는 소프트웨어 공학의 초석 중 하나인 DRY(Don't Repeat Yourself) 원칙에 대한 포괄적이고 심층적인 분석을 제공하는 것을 목표로 한다. 소프트웨어 개발 분야에서 DRY 원칙은 가장 널리 알려져 있으면서도 가장 자주 오해받는 개념 중 하나이다. 많은 개발자들이 이를 단순히 '코드 중복 제거'라는 표면적 수준에서 이해하지만, 이는 원칙의 본질적인 깊이와 적용 범위를 심각하게 축소하는 해석이다. 본 보고서는 이러한 피상적 이해를 넘어, '지식의 단일한, 권위 있는 표현'이라는 원칙의 본질을 탐구하고, 그것이 소프트웨어 시스템의 전 계층에 걸쳐 어떻게 적용되고 다른 핵심 개발 원칙들과 상호작용하는지를 다각적으로 조명할 것이다. 이를 통해 독자들이 DRY 원칙을 보다 정교하고 현명하게 실제 프로젝트에 적용할 수 있는 통찰과 실용적 지침을 얻게 되기를 기대한다.


이 장에서는 DRY 원칙이 탄생한 배경과 그 핵심 철학을 깊이 있게 탐구한다. 많은 개발자들이 오해하는 '코드 중복'과의 차이점을 명확히 하고, '지식의 중복'이라는 더 근본적인 문제에 초점을 맞춘다.


DRY 원칙은 앤디 헌트(Andy Hunt)와 데이브 토마스(Dave Thomas)가 그들의 저서 *The Pragmatic Programmer*에서 처음으로 공식화하며 널리 알려졌다.1 그들은 소프트웨어 개발을 지속적인 '유지보수' 과정으로 보았으며, 이러한 관점에서 시스템의 유지보수성을 저해하는 가장 큰 요인 중 하나로 '지식의 중복'을 지목했다.4

이들이 제시한 DRY 원칙의 핵심 정의는 다음과 같다:

> "모든 지식 조각은 시스템 내에서 단일하고, 모호하지 않으며, 권위 있는 표현을 가져야 한다(Every piece of knowledge must have a single, unambiguous, authoritative representation within a system)." 1

이 문장은 단순한 지침을 넘어선 철학적 선언에 가깝다. 각 단어는 신중하게 선택되었으며 깊은 의미를 내포한다.

- **모든 지식 조각(Every piece of knowledge):** 이는 코드로 한정되지 않는다. 데이터베이스 스키마, 테스트 계획, 빌드 시스템, 심지어 문서에 이르기까지 시스템을 구성하는 모든 정보 단위를 포함한다.5
- **단일한(Single):** 지식은 시스템 내 단 한 곳에만 존재해야 한다. 이는 정보의 출처가 유일해야 함을 의미하며, 변경이 필요할 때 수정해야 할 곳이 단 한 곳임을 보장한다.1
- **모호하지 않으며(Unambiguous):** 그 표현은 해석의 여지가 없이 명확해야 한다.
- **권위 있는(Authoritative):** 해당 지식에 대한 최종적인 '진실'의 원천(Source of Truth)으로 인정되어야 한다. 다른 모든 부분은 이 권위 있는 표현을 참조하거나 그것으로부터 파생되어야 한다.

결론적으로, DRY 원칙의 핵심은 단순히 코드의 외형적 반복을 피하는 것이 아니라, 시스템이 의존하는 근본적인 '지식' 또는 '정보'를 중앙에서 관리하여 일관성을 유지하고 변경의 파급 효과를 최소화하려는 아키텍처 수준의 지식 관리 전략이라 할 수 있다.7


DRY 원칙에 대한 가장 흔한 오해는 이를 '코드 중복 금지'와 동일시하는 것이다. 물론 많은 경우에 코드의 중복은 지식의 중복을 의미하지만, 이 둘이 항상 일치하는 것은 아니다.8 DRY 원칙의 진정한 초점은 코드의 구문적(syntactic) 반복이 아니라, 시스템의 의도(intent)와 지식(knowledge)의 의미론적(semantic) 반복에 있다.9

예를 들어, 시스템에 두 개의 서로 다른 유효성 검사 규칙이 있다고 가정해 보자. 하나는 '사용자의 나이는 19세 이상이어야 한다'는 규칙이고, 다른 하나는 '주문 가능한 최대 상품 수량은 19개'라는 규칙이다. 이 두 규칙은 현재 우연히 동일한 숫자 `19`를 사용하지만, 이들은 완전히 다른 비즈니스 개념을 나타낸다. 전자는 법적 규제에 의해, 후자는 재고 관리 정책에 의해 변경될 수 있다. 이 두 곳의 코드에서 숫자 `19`를 하나의 상수로 통합하는 것은 DRY 원칙의 잘못된 적용이다. 왜냐하면 코드는 비슷해 보일지라도 그 안에 담긴 '지식'-'법적 음주 가능 연령'과 '최대 주문 수량'-은 서로 다르기 때문이다.12

반대로, 코드의 형태가 다르더라도 동일한 지식을 표현하고 있다면 이는 DRY 원칙 위반이다. 예를 들어, 한 곳에서는 정규 표현식을 사용해 이메일 형식을 검증하고, 다른 곳에서는 문자열 분할(split)과 포함(includes) 메서드를 조합하여 동일한 검증 로직을 구현했다면, 이는 '이메일 형식 유효성 검사 규칙'이라는 단일 지식이 두 가지 다른 방식으로 중복 표현된 것이다.

따라서 DRY 원칙을 올바르게 적용하기 위해서는 "이 코드가 같은가?"를 묻는 대신 "이 코드가 표현하는 지식과 의도가 같은가?"를 물어야 한다.13


DRY 원칙은 종종 SPOT(Single Point of Truth) 또는 SSOT(Single Source of Truth) 원칙과 동의어로 사용되거나 그 구체적인 구현 형태로 간주된다.14 SPOT 원칙은 시스템 내의 모든 데이터 요소가 단 하나의 출처에서 비롯되어야 한다는, 보다 데이터 중심적인 개념이다.

DRY는 이 개념을 코드, 로직, 설정을 포함한 시스템의 모든 '지식'으로 확장한다. 예를 들어, '피자는 최소 하나의 토핑이 필요하다'는 비즈니스 규칙이 있다고 가정하자. 이 '진실'은 시스템 내에서 단 한 곳에만 코드로 표현되어야 한다. 만약 이 규칙이 프론트엔드 유효성 검사 로직과 백엔드 API 서버의 비즈니스 로직에 각각 별도로 구현되어 있다면, 이는 SPOT 원칙과 DRY 원칙을 모두 위반한 것이다. 향후 정책이 '최소 두 개의 토핑'으로 변경될 경우, 두 곳을 모두 수정해야 하며, 하나라도 누락하면 시스템은 데이터 불일치 상태에 빠지게 된다.14

이처럼 DRY 원칙은 시스템의 모든 지식에 대해 단일 진실 지점(SPOT)을 강제함으로써, 시스템의 일관성과 예측 가능성을 보장하는 핵심적인 역할을 수행한다.


DRY 원칙은 단순히 코드 라인을 줄이는 기법이 아니다. 이는 시스템 설계의 전반에 걸쳐 적용될 수 있는 강력한 원칙으로, 코드, 데이터, 프로세스, 문서화 등 다양한 계층에서 그 가치를 발휘한다. 원칙 적용의 핵심 기준은 '현재의 외형적 동일성'이 아니라 '미래의 동시적 변경 가능성'에 있다. 즉, "이 정보가 변경될 때, 함께 변경되어야 하는 다른 곳이 있는가?"라는 질문이 DRY 원칙 위반 여부를 판단하는 시금석이 된다.


코드 계층은 DRY 원칙이 가장 명시적으로 적용되는 영역이다. 반복되는 로직이나 값을 추상화하고 재사용 가능한 단위로 만드는 것이 핵심이다.

- **함수 및 메서드:** 가장 기본적인 DRY 적용은 반복되는 코드 블록을 함수나 메서드로 추출하는 것이다.1 예를 들어, 여러 곳에서 사용자의 전체 이름을 표현하기 위해 

  `user.firstName + " " + user.lastName`과 같은 코드를 반복적으로 작성하고 있다면, 이를 `User` 클래스 내의 `getFullName()` 메서드로 캡슐화해야 한다. 이렇게 하면 이름 표시 정책이 변경되더라도(예: `lastName`을 먼저 표시) `getFullName()` 메서드 한 곳만 수정하면 되므로 유지보수성이 극적으로 향상된다.11

- **클래스와 상속/구성:** 객체지향 프로그래밍에서 상속과 구성은 DRY 원칙을 구현하는 강력한 도구다. 여러 클래스에서 공통으로 나타나는 상태(속성)와 행위(메서드)를 식별하여 부모 클래스로 추상화하거나, 별도의 컴포넌트 클래스로 분리하여 구성을 통해 재사용할 수 있다.16 예를 들어, 

  `Customer`와 `Employee` 클래스가 모두 주소 정보를 가지고 있다면, `Address` 클래스를 별도로 정의하고 두 클래스가 `Address` 객체를 멤버로 갖도록 구성하는 것이 좋다.

- **설정(Configuration):** 코드 내에 하드코딩된 값, 소위 '매직 스트링(magic strings)'이나 '매직 넘버(magic numbers)'는 DRY 원칙의 주요 위반 사례다. 데이터베이스 연결 문자열, API 엔드포인트 URL, 특정 비즈니스 규칙의 임계값(예: 최대 시도 횟수) 등은 별도의 설정 파일(예: `.properties`, `JSON`, `YAML`)이나 상수(Constants) 클래스로 중앙화하여 관리해야 한다.18 이를 통해 설정 변경 시 코드를 재컴파일할 필요가 없어지고, 여러 환경(개발, 스테이징, 프로덕션)에 따른 설정을 유연하게 관리할 수 있다.


데이터베이스 설계에서 데이터 정규화(Normalization) 과정은 그 자체로 데이터 계층에서 DRY 원칙을 체계적으로 적용하는 행위다.6 정규화의 핵심 목표는 데이터의 중복성을 최소화하여 데이터 무결성을 보장하고, 데이터의 삽입, 수정, 삭제 시 발생할 수 있는 이상 현상(anomaly)을 방지하는 것이다.21

예를 들어, 제3정규형(3NF)은 이행적 종속성(transitive dependency)을 제거할 것을 요구한다. 이는 기본 키가 아닌 다른 컬럼에 종속적인 컬럼을 별도의 테이블로 분리하는 것을 의미한다. 가령, `Orders` 테이블에 `CustomerID`, `CustomerName`, `CustomerAddress`가 모두 포함되어 있다면, `CustomerName`과 `CustomerAddress`는 기본 키인 `OrderID`가 아니라 `CustomerID`에 종속된다. 이는 고객 정보라는 '지식'이 모든 주문 레코드에 중복 저장되는 결과를 낳는다. 3NF에 따라 `Customers` 테이블을 별도로 만들고 `Orders` 테이블에서는 `CustomerID`만을 외래 키로 참조하도록 구조를 변경하면, 고객 정보는 시스템 내에서 단일하고 권위 있는 표현을 갖게 되어 DRY 원칙을 만족시킨다.21


현대 소프트웨어 개발에서 빌드 및 배포 프로세스는 점점 더 복잡해지고 있다. Maven, Gradle과 같은 빌드 자동화 도구를 사용하는 환경에서도 DRY 원칙은 매우 중요하다.23

다중 모듈 프로젝트에서 각 모듈의 `build.gradle` 파일에 공통적으로 사용되는 라이브러리 의존성과 버전, 플러그인 설정 등을 반복적으로 선언하는 것은 흔한 DRY 위반 사례다. 이러한 중복은 버전 불일치로 인한 호환성 문제를 야기하고, 라이브러리 업그레이드 시 모든 모듈 파일을 일일이 수정해야 하는 유지보수 부담을 가중시킨다.23

이에 대한 DRY 해결책은 공통 설정을 한 곳으로 중앙화하는 것이다. Gradle에서는 루트 프로젝트의 `subprojects` 블록을 사용하거나, `buildSrc` 디렉토리, 또는 컨벤션 플러그인(convention plugin)을 만들어 공통 의존성 및 설정을 정의하고 각 모듈에서 이를 적용하도록 할 수 있다. 이를 통해 라이브러리 버전 관리가 단일 지점에서 이루어지므로 일관성이 보장되고 유지보수가 용이해진다.23


DRY 원칙은 코드뿐만 아니라 기술 문서에도 동일하게 적용된다. 시스템의 기능, API 명세, 아키텍처 설명 등이 여러 문서에 걸쳐 복사-붙여넣기 방식으로 작성되면, 코드 변경 시 관련된 모든 문서를 찾아 수정하는 것은 거의 불가능에 가깝다. 이는 결국 문서와 실제 시스템 간의 불일치를 초래하여 문서의 신뢰성을 떨어뜨리는 주된 원인이 된다.9

문서화에서 DRY를 적용하는 방법은 '단일 출처(single-sourcing)' 기법을 활용하는 것이다. 예를 들어, API 엔드포인트에 대한 상세 설명은 소스 코드의 주석(예: Javadoc, OpenAPI/Swagger 어노테이션)에 단 한 번만 작성하고, 빌드 시점에 이 주석으로부터 API 문서를 자동으로 생성하는 방식을 사용할 수 있다. 또한, 공통적인 개념 설명은 위키(Wiki)와 같은 중앙화된 저장소에 작성하고, 다른 문서에서는 해당 페이지로 링크를 거는 방식을 사용해야 한다.26

다만, 문서화의 경우 예외적인 고려가 필요하다. 코드와 달리 문서는 사용자의 편의성과 가독성이 매우 중요하다. 때로는 사용자가 여러 문서를 넘나들며 정보를 찾는 것을 방지하기 위해, 명확성을 높일 목적으로 핵심적인 정보를 의도적으로 반복하는 것이 더 효과적일 수 있다. 이러한 접근 방식을 ARID(Accepts some Repetition In Documentation) 원칙이라고 하며, DRY 원칙과 상충하는 것처럼 보이지만 실제로는 문서의 사용 맥락을 고려한 실용적인 절충안이라 할 수 있다.27

다음 표는 각 시스템 계층에서 DRY 원칙을 적용하는 구체적인 방법을 요약한 것이다.

| 계층         | 핵심 목표                      | 구체적 기법                                           | 성공 사례                                      | 주의할 점 (관련 원칙)                     |
| ------------ | ------------------------------ | ----------------------------------------------------- | ---------------------------------------------- | ----------------------------------------- |
| **코드**     | 로직의 재사용성 및 일관성 확보 | 함수/클래스 추출, 상속/구성, 디자인 패턴, 설정 중앙화 | 공통 유틸리티 라이브러리, 기반 컨트롤러 클래스 | 과잉 추상화, 거짓 중복 (AHA, KISS)        |
| **데이터**   | 데이터 무결성 및 중복 최소화   | 데이터베이스 정규화 (1NF, 2NF, 3NF, BCNF)             | 고객 정보와 주문 정보를 분리한 테이블 설계     | 과도한 정규화로 인한 조인 성능 저하       |
| **프로세스** | 빌드/배포의 일관성 및 자동화   | 빌드 스크립트 모듈화, 공통 의존성 관리                | Gradle의 `buildSrc` 또는 컨벤션 플러그인 활용  | 설정의 복잡성 증가 (KISS)                 |
| **테스트**   | 테스트 로직의 재사용           | 헬퍼 함수, 데이터 생성기, 기반 테스트 클래스          | `createUser` 헬퍼를 통한 테스트 데이터 생성    | 테스트 간 결합도 증가, 가독성 저하 (DAMP) |
| **문서**     | 정보의 최신성 및 일관성 유지   | 단일 출처 문서화, 링크/참조 활용                      | API 명세를 코드로 생성 (Swagger/OpenAPI)       | 가독성을 위한 의도적 반복 허용 (ARID)     |


DRY 원칙은 강력하지만, 맹목적이거나 기계적으로 적용될 때 오히려 시스템의 건강을 해치는 독이 될 수 있다. 특히 '거짓 중복'을 진정한 중복으로 오인하여 섣부른 추상화를 감행하는 것은 유연성을 저해하고 불필요한 복잡성을 야기하는 대표적인 오용 사례다. 올바른 DRY 원칙의 적용은 '변경의 축(Axis of Change)'을 정확히 예측하고, 동일한 축 상에 있는 지식만을 함께 묶는 것에 달려있다. 잘못된 추상화는 본질적으로 서로 다른 변경 축을 가진 개념들을 하나의 운명 공동체로 억지로 묶어버리는 행위와 같다.


거짓 중복(False Duplication), 또는 우발적 중복(Accidental Duplication)은 두 개 이상의 코드 조각이 현재 시점에서는 구문적으로 동일하거나 매우 유사해 보이지만, 그들이 표현하는 근본적인 도메인 지식이나 비즈니스 맥락이 서로 다른 경우를 의미한다.10 이러한 코드는 단지 '우연히' 같을 뿐이며, 미래에는 서로 다른 이유로, 다른 시점에, 다른 방향으로 변경될 가능성이 매우 높다.

예를 들어, 한 온라인 쇼핑몰 시스템에서 상품의 '정가(Regular Price)'와 '특별 할인가(Discounted Price)'를 검증하는 로직이 있다고 가정하자. 현재 정책상 두 가격 모두 "0원 이상이어야 한다"는 규칙을 가지고 있어, 다음과 같은 코드가 두 곳에 존재할 수 있다.28

```Java
// 정가 검증 로직
if (regularPrice < 0) {
    throw new IllegalArgumentException("가격은 0원 이상이어야 합니다.");
}

// 할인가 검증 로직
if (discountedPrice < 0) {
    throw new IllegalArgumentException("가격은 0원 이상이어야 합니다.");
}
```

이 코드는 외형상 완벽한 중복처럼 보인다. 그러나 '정가 정책'과 '할인가 정책'은 서로 다른 비즈니스 개념이다. 미래에 '정가는 반드시 100원 이상이어야 한다'는 새로운 정책이 생기거나, '할인가는 정가보다 높을 수 없다'는 규칙이 추가될 수 있다. 이 두 지식은 서로 독립적인 '변경의 축'을 따라 진화한다. 따라서 이들을 섣불리 하나의 공통 검증 함수로 묶는 것은 거짓 중복을 제거하려다 더 큰 문제를 야기하는 전형적인 사례가 된다.


거짓 중복을 제거하기 위해 성급하게 추상화를 도입하면, 본질적으로 관련 없는 도메인 개념들 사이에 인위적인 의존성이 생겨 강하게 결합(tightly coupled)되는 문제가 발생한다.28 이렇게 되면 한쪽의 변경이 다른 쪽에 의도치 않은 영향을 미치게 되어 시스템의 유연성이 크게 저하된다.

앞선 가격 예시를 확장하여, '일반 할인 가격'과 '여름 특별 할인 가격'을 계산하는 두 클래스가 있다고 가정하자. 초기에는 두 할인 정책이 모두 '정가에서 4,000원을 할인'하는 동일한 로직을 가질 수 있다.28

```Java
// 초기: 두 클래스의 로직이 동일함
public class RegularDiscountedPrice {
    //...
    public RegularDiscountedPrice(final RegularPrice regularPrice) {
        int discountedAmount = regularPrice.getValue() - 4000;
        //...
    }
}

public class SummerDiscountedPrice {
    //...
    public SummerDiscountedPrice(final RegularPrice regularPrice) {
        int discountedAmount = regularPrice.getValue() - 4000;
        //...
    }
}
```

이 중복을 제거하기 위해 `SummerDiscountedPrice`가 `RegularDiscountedPrice`를 상속하도록 리팩토링할 수 있다.

```Java
// 잘못된 추상화: 상속을 통한 중복 제거
public class SummerDiscountedPrice extends RegularDiscountedPrice {
    public SummerDiscountedPrice(RegularPrice regularPrice) {
        super(regularPrice);
    }
}
```

이제 코드는 더 짧아졌지만, '여름 할인 정책'은 '일반 할인 정책'에 강하게 종속되었다. 만약 비즈니스 요구사항이 변경되어 "여름 할인은 고정 금액이 아닌 5% 정률 할인으로 변경한다"고 결정되면 어떻게 될까? `SummerDiscountedPrice`는 더 이상 부모 클래스의 할인 로직을 사용할 수 없게 된다. 결국 이 상속 관계는 깨져야만 하며, 개발자는 상속 구조를 제거하고 `SummerDiscountedPrice` 클래스의 로직을 처음부터 다시 작성해야 한다.28 이는 잘못된 추상화가 어떻게 미래의 변화에 대응하는 비용을 증가시키는지를 명확히 보여준다.


DRY 원칙에 대한 과도한 집착은 종종 불필요하고 지나치게 복잡한 추상화 계층, 즉 과잉 추상화(over-abstraction)를 낳는다.10 이러한 추상화는 코드의 줄 수를 줄일 수는 있지만, 시스템의 전체적인 동작을 파악하기 어렵게 만들고, 간단한 변경조차 여러 계층을 거쳐야 하는 복잡한 작업으로 만들어 버린다. 결국 이는 소프트웨어 설계의 또 다른 핵심 원칙인 KISS(Keep It Simple, Stupid)를 정면으로 위반하는 결과를 초래한다.30

예를 들어, 내비게이션 메뉴에 'About', 'Contact', 'Buy'라는 세 개의 버튼을 렌더링하는 코드가 있다고 가정하자. 각 버튼은 약간씩 다른 속성을 가질 수 있다. 이 작은 중복을 제거하기 위해 개발자가 모든 경우를 처리할 수 있는 하나의 거대한 `ButtonFactory` 컴포넌트를 만들기로 결정할 수 있다.31

초기에는 이 팩토리가 깔끔해 보일 수 있다. 하지만 곧이어 "Buy 버튼에만 빨간색 테두리를 추가해달라"는 요구사항이 들어온다. 이 추상화는 모든 버튼을 동일하게 유지하는 데 최적화되어 있었기 때문에, 이 작은 변화를 수용하기 위해 팩토리 함수에 `if (type === 'buy')`와 같은 조건 분기나 새로운 파라미터(`borderColor`)를 추가해야 한다. 이러한 요구사항이 누적됨에 따라, 이 팩토리 함수는 수많은 파라미터와 복잡한 내부 로직으로 가득 찬 '괴물'이 되어버린다. 결국, 이 복잡한 추상화를 이해하고 사용하는 비용이, 약간의 코드를 복사-붙여넣기 하는 비용보다 훨씬 커지게 된다.31 이는 중복 그 자체가 문제가 아니라, 중복을 제거하기 위해 도입된 '잘못된 추상화'가 더 큰 문제임을 보여주는 사례다.


소프트웨어 개발의 세계에서 어떤 원칙도 고립되어 존재하지 않는다. DRY 원칙 역시 마찬가지다. 그 가치는 다른 중요한 원칙들과의 관계 속에서 정의되며, 때로는 이들과 긴장 관계를 형성하기도 한다. 성숙한 개발자는 이 원칙들을 맹목적인 규칙으로 따르는 것이 아니라, 각 상황의 맥락 속에서 이들의 상호작용을 이해하고 최적의 균형점을 찾는 변증법적 사고를 해야 한다. DRY, AHA, Rule of Three, KISS, YAGNI 등은 서로 대립하는 규칙이 아니라, '지속 가능한 소프트웨어 개발'이라는 공동 목표를 위한 상호 보완적인 '휴리스틱(Heuristics)'의 집합이다.


WET은 DRY 원칙의 명백한 안티테제(antithesis)로, 주로 "Write Everything Twice(모든 것을 두 번 작성하라)"의 약자로 통용된다. 때로는 "We Enjoy Typing(우리는 타이핑을 즐긴다)" 또는 "Waste Everyone's Time(모두의 시간을 낭비한다)"과 같은 풍자적인 의미로도 사용된다.11

WET한 코드는 명백한 비용을 초래한다. 다계층 아키텍처에서 사용자가 입력한 '댓글(comment)'을 처리하는 기능을 구현한다고 가정해 보자. 'comment'라는 텍스트 문자열은 UI의 레이블, HTML 태그의 `id` 속성, 프론트엔드 상태 관리 변수명, API 요청 객체의 필드명, 백엔드 DTO(Data Transfer Object)의 속성명, 데이터베이스 테이블의 컬럼명, 관련 로직을 담은 함수명 등 시스템의 여러 계층에 걸쳐 반복적으로 나타날 수 있다.5

만약 '댓글'을 '리뷰(review)'로 변경해야 하는 요구사항이 발생한다면, 개발자는 이 모든 곳을 찾아 정확하게 수정해야 한다. 하나라도 누락하면 시스템의 일관성이 깨지고 버그가 발생한다. 이것이 바로 WET한 코드가 초래하는 유지보수 비용 증가, 버그 발생 가능성 증대, 그리고 일관성 파괴의 전형적인 예시다.33 DRY 접근 방식은 이러한 중복을 프레임워크나 코드 생성기, 혹은 단일 설정 파일을 통해 제거하여, 변경이 필요한 지점을 단 한 곳으로 집중시킨다.5


AHA(Avoid Hasty Abstractions) 원칙은 DRY 원칙에 대한 중요한 안전장치 역할을 한다. 이 원칙의 핵심은 "잘못된 추상화보다 차라리 중복이 낫다"는 것이다.5 AHA는 중복을 발견했을 때 즉시 추상화하려는 충동을 억제하고, 해당 중복의 본질과 그 이면에 있는 패턴을 충분히 이해할 때까지 기다려야 한다는 실용적인 지침을 제공한다.35

이는 개발 과정이 불확실성 속에서 진행된다는 사실을 인정하는 것이다. 초기 단계에서는 두 코드 조각이 진정한 중복인지, 아니면 우발적인 거짓 중복인지 명확하지 않을 수 있다. AHA 원칙은 이러한 불확실성 하에서 섣부른 결정(잘못된 추상화)을 내리는 위험을 감수하기보다, 일단 중복을 용납하고 시간이 지나면서 더 많은 정보가 드러나기를 기다리는 것이 더 현명한 전략일 수 있음을 시사한다.37


그렇다면 언제 추상화를 적용하는 것이 적절한가? 여기에 대한 구체적인 경험 법칙(rule of thumb)으로 '세 번의 법칙(Rule of Three)'이 널리 알려져 있다.7

이 법칙은 다음과 같이 요약된다:

1. 어떤 코드를 처음 작성할 때는 그냥 작성한다.
2. 비슷한 코드를 두 번째로 작성해야 할 때도, 아직은 추상화하지 않고 중복을 허용한다.
3. 하지만 세 번째로 동일하거나 매우 유사한 코드를 작성하게 될 때, 비로소 이는 우연이 아닌 명백한 패턴으로 간주하고 리팩토링하여 추상화한다.

'세 번의 법칙'은 AHA 원칙을 실천하는 구체적인 방법론을 제시한다. 두 번의 사례만으로는 올바른 추상화 패턴을 도출하기에 정보가 부족할 수 있지만, 세 번의 사례가 모이면 공통점과 차이점이 더 명확해져 더 견고하고 유연한 추상화를 설계할 수 있게 된다.38 이 법칙은 성급한 추상화의 위험을 줄이고, 더 나은 설계 결정을 내릴 수 있는 시간을 벌어주는 효과적인 지연 전략이다.


소프트웨어 설계는 종종 DRY, KISS(Keep It Simple, Stupid), YAGNI(You Ain't Gonna Need It)라는 세 가지 원칙 사이에서 섬세한 줄타기를 하는 것과 같다.

- **DRY vs. KISS:** DRY 원칙을 추구하기 위한 추상화는 때때로 코드를 더 복잡하게 만들 수 있다. 앞서 살펴본 '과잉 추상화' 사례처럼, 중복을 제거하기 위해 도입된 복잡한 디자인 패턴이나 상속 구조는 시스템을 이해하고 수정하기 어렵게 만들어 KISS 원칙을 위반할 수 있다.30 이 경우, 개발자는 추상화로 얻는 유지보수의 이점과 그로 인해 발생하는 복잡성의 비용을 저울질해야 한다. 종종 약간의 중복을 감수하는 더 단순한 설계가 장기적으로 더 나은 선택일 수 있다.30
- **DRY vs. YAGNI:** YAGNI 원칙은 "당신은 그것이 필요하지 않을 것이다"라고 말하며, 현재 시점에 명확히 요구되지 않는 기능은 구현하지 말라고 경고한다.41 이는 미래에 필요할 것이라는 '예측'에 기반하여 과도하게 유연하고 일반적인 추상화를 만드는 행위를 비판한다. DRY 원칙은 '현재 존재하는' 명백한 중복을 제거하는 데 초점을 맞춰야지, '미래에 발생할지도 모르는' 가상의 중복을 미리 해결하기 위해 YAGNI를 위반해서는 안 된다.10

이 세 원칙의 균형을 맞추기 위한 실용적인 작업 흐름은 다음과 같이 제안될 수 있다:

"먼저 현재 요구사항을 만족하는 가장 간단한 코드를 작성하라(KISS & YAGNI). 개발을 진행하면서 반복되는 패턴이 세 번 이상 명확하게 나타나면(Rule of Three), 그 때 비로소 해당 중복을 리팩토링하여 제거하라(DRY). 단, 이 과정에서 현재 필요하지 않은 미래의 유연성까지 고려하여 시스템을 과도하게 설계하지는 마라(YAGNI)." 40

다음 표는 주요 소프트웨어 개발 원칙들의 핵심 목표와 장단점을 비교 분석하여, 이들 간의 관계를 종합적으로 이해하는 데 도움을 준다.

| 원칙              | 핵심 목표                               | 접근 방식                                               | 장점                                         | 단점 / 오용 시 위험                      |
| ----------------- | --------------------------------------- | ------------------------------------------------------- | -------------------------------------------- | ---------------------------------------- |
| **DRY**           | 지식의 중복 제거를 통한 유지보수성 향상 | 공통 로직/데이터를 단일 출처로 추상화                   | 일관성, 버그 감소, 변경 용이성               | 과잉 추상화, 거짓 중복으로 인한 강결합   |
| **WET**           | (DRY의 안티테제)                        | 모든 것을 필요할 때마다 반복 작성                       | (의도적 적용 시) 초기 개발 속도, 낮은 결합도 | 유지보수 재앙, 버그 확산, 불일치 발생    |
| **AHA**           | 성급하고 잘못된 추상화 방지             | 중복을 허용하며 패턴이 명확해질 때까지 추상화 유보      | 더 나은 추상화 도출, 유연성 확보             | 코드 중복으로 인한 단기적 유지보수 비용  |
| **Rule of Three** | 추상화 시점에 대한 구체적 가이드        | 세 번째 중복이 발생했을 때 리팩토링                     | AHA를 실천하는 경험적 기준 제공              | 모든 상황에 적용되는 절대적 규칙은 아님  |
| **KISS**          | 시스템의 단순성 유지                    | 불필요한 복잡성을 피하고 가장 간단한 해결책 모색        | 이해/디버깅 용이, 인지 부하 감소             | 과도한 단순화로 인한 확장성 부족         |
| **YAGNI**         | 불필요한 기능 개발 방지                 | 현재 요구사항에만 집중하고 미래 예측에 기반한 개발 지양 | 개발 시간 단축, 코드베이스 간결화            | 미래 확장 시 대규모 리팩토링 필요 가능성 |


DRY 원칙은 소프트웨어 개발의 거의 모든 영역에서 긍정적으로 받아들여지지만, 유독 단위 테스트(Unit Test) 코드에 적용될 때에는 격렬한 논쟁의 대상이 된다. 이는 프로덕션 코드와 테스트 코드가 추구하는 근본적인 '목표 함수(Objective Function)'가 다르기 때문이다. 프로덕션 코드의 목표 함수가 '유지보수성'과 '효율성'에 중점을 둔다면, 테스트 코드의 목표 함수는 '신뢰성'과 '가독성(문서로서의 가치)'에 더 큰 비중을 둔다. 이 차이를 이해하지 못하고 프로덕션 코드의 원칙을 테스트 코드에 기계적으로 적용하는 것이 문제의 근원이다.


단위 테스트에 DRY 원칙을 과도하게 적용하면 여러 가지 부작용이 발생할 수 있다.

- **가독성 저하:** 테스트 케이스는 흔히 준비(Arrange), 실행(Act), 단언(Assert)의 3단계 구조를 따른다. 여러 테스트에서 공통적으로 사용되는 준비(Arrange) 단계의 로직을 별도의 `Setup` 함수나 기반 클래스로 추출하면, 코드의 양은 줄어들 수 있다. 하지만 이는 각 테스트 케이스를 이해하기 위해 여러 파일이나 함수를 넘나들며 컨텍스트를 파악해야 하는 부담을 준다. 결국 테스트 케이스 하나가 그 자체로 완결된 스토리를 전달하지 못하게 되어 가독성이 심각하게 저하된다.42
- **테스트 간 결합도 증가:** 공통 셋업 로직이나 데이터를 여러 테스트가 공유하게 되면, 테스트 간에 보이지 않는 결합(coupling)이 발생한다. 예를 들어, 테스트 A를 위해 공통 `Setup` 함수를 약간 수정했는데, 이로 인해 전혀 관련 없어 보이던 테스트 B, C, D가 갑자기 실패하는 상황이 발생할 수 있다. 이는 테스트의 독립성이라는 핵심 원칙을 위배하며, 실패의 원인을 추적하기 어렵게 만든다.42
- **디버깅의 어려움:** 특정 테스트가 실패했을 때, 그 원인이 해당 테스트 케이스 자체의 로직에 있는지, 아니면 공유되는 셋업 로직의 미묘한 상태 변화 때문인지 파악하기가 매우 어려워진다. 특히 여러 테스트가 순차적으로 실행되면서 공유된 상태를 변경하는 경우, 특정 테스트는 단독으로 실행하면 성공하지만 전체 테스트 스위트 내에서는 실패하는, 소위 '깜빡이는 테스트(flaky test)'의 원인이 되기도 한다.44


이러한 문제에 대한 대안으로 테스트 코드에서는 DAMP(Descriptive and Meaningful Phrases) 원칙이 강조된다.42 DAMP 원칙은 테스트 코드가 최대한 서술적이고 의미가 명확해야 한다는 점을 우선시한다. 이는 약간의 코드 중복을 감수하더라도, 각 테스트 케이스가 독립적으로 이해될 수 있는 하나의 완결된 시나리오를 보여주어야 함을 의미한다.

"테스트는 테스트가 없다"는 격언이 있다.45 이는 프로덕션 코드와 달리 테스트 코드는 그 자체의 정확성을 보장해 줄 또 다른 테스트가 없으므로, 사람이 직접 눈으로 읽고 그 정확성을 쉽게 검증할 수 있어야 한다는 의미다. DAMP 원칙을 따르는 테스트 코드는 시스템의 특정 동작에 대한 '살아있는 문서'로서의 역할을 수행한다. 새로운 개발자가 프로젝트에 합류했을 때, 잘 작성된 DAMP한 테스트 코드를 읽는 것만으로도 시스템의 요구사항과 동작 방식을 빠르고 정확하게 파악할 수 있다.


그렇다면 테스트 코드에서는 DRY를 완전히 포기하고 모든 것을 중복 작성해야 하는가? 반드시 그렇지는 않다. DRY와 DAMP는 상호 배타적인 개념이 아니며, 두 원칙의 장점을 모두 취하는 조화로운 접근이 가능하다. 핵심은 테스트 시나리오의 '무엇을(What)'과 그 구현의 '어떻게(How)'를 분리하여 생각하는 것이다.42

- **DAMP 적용 대상 (What-to's):** 테스트가 '무엇을' 검증하는지에 대한 서술적인 단계들은 각 테스트 케이스 내에 명시적으로 남아있어야 한다. 이는 테스트의 가독성과 시나리오의 명확성을 보장한다. 예를 들어, `given_a_user_is_logged_in()`이나 `when_an_item_is_added_to_the_cart()`와 같은 서술적인 단계는 테스트 메서드 본문에 직접 드러나는 것이 좋다.
- **DRY 적용 대상 (How-to's):** 각 단계를 '어떻게' 구현하는지에 대한 복잡하고 반복적인 로직은 헬퍼(helper) 함수나 팩토리(factory) 메서드로 추출하여 재사용할 수 있다. 예를 들어, '로그인한 사용자'를 생성하는 과정이 복잡하다면(데이터베이스에 사용자 레코드 생성, 세션 설정 등), 이 로직을 `createLoggedInUser()`라는 헬퍼 함수로 캡슐화할 수 있다.

이러한 접근법을 통해 테스트 코드는 다음과 같은 형태를 띠게 된다:

```C#
// DAMP와 DRY를 조화시킨 예시
[Fact]
public void Adding_item_to_cart_succeeds_for_logged_in_user()
{
    // Arrange (What): 시나리오가 명확히 드러남
    User loggedInUser = CreateLoggedInUser("testuser");
    Product product = CreateProduct("Laptop", price: 1500);
    ShoppingCart cart = new ShoppingCart(loggedInUser);

    // Act (What)
    cart.AddItem(product);

    // Assert (What)
    Assert.Equal(1, cart.ItemCount);
    Assert.Equal(1500, cart.TotalPrice);
}

// DRY 적용 (How): 구현 세부사항은 헬퍼 함수로 추출
private User CreateLoggedInUser(string username)
{
    // 사용자 생성 및 로그인 상태 설정에 대한 복잡한 로직
    // 이 로직은 여러 테스트에서 재사용될 수 있음
    var user = new User(username);
    _userService.Login(user);
    return user;
}
```

이처럼 'What-to'와 'How-to'를 분리함으로써, 테스트 케이스의 가독성(DAMP)을 해치지 않으면서도 복잡한 준비 과정의 중복(DRY)을 효과적으로 제거할 수 있다. 이는 프로덕션 코드와 테스트 코드의 서로 다른 목표 함수를 모두 만족시키려는 현명한 타협안이다.


DRY 원칙은 소프트웨어 장인 정신의 핵심에 놓여 있지만, 그 자체로 목적이 될 수는 없다. 그것은 더 나은, 즉 더 유지보수하기 쉽고, 더 이해하기 쉬우며, 더 견고한 소프트웨어를 만들기 위한 강력한 도구이자 지침이다. 본 보고서의 논의를 종합하여, 개발자가 DRY 원칙을 현명하게 활용하기 위한 몇 가지 최종적인 제언을 제시한다.


가장 중요한 것은 DRY를 맹목적으로 따라야 할 경직된 규칙(Rule)이 아니라, 주어진 상황과 맥락에 맞게 신중하게 해석하고 적용해야 할 원칙(Principle)으로 이해하는 것이다.29 소프트웨어 개발에는 은탄환(silver bullet)이 없으며, 모든 원칙에는 예외와 트레이드오프가 존재한다. 때로는 의도적인 중복이 섣부른 추상화보다 더 나은 선택일 수 있으며, 원칙을 어기는 것이 더 실용적인 결과를 낳을 수도 있다. 독단주의를 경계하고 항상 실용주의적인 관점을 유지하는 것이 성숙한 개발자의 자세다.29


DRY 원칙을 적용할지 여부를 결정하는 가장 효과적인 사고 모델은 '변경의 축'을 중심으로 생각하는 것이다. 코드의 외형적 유사성에 현혹되지 말고, 다음과 같은 질문을 스스로에게 던져야 한다: "이 코드 조각들은 동일한 이유로, 함께 변경될 운명인가?" 만약 그 답이 '그렇다'라면, 이는 진정한 지식의 중복이며 추상화의 강력한 후보다. 만약 '아니다'라면, 그것은 단지 우발적인 거짓 중복일 가능성이 높으므로 그대로 두는 것이 시스템의 유연성을 위해 더 나은 결정일 수 있다.


잘못된 추상화는 관리되지 않는 중복보다 더 큰 해악을 끼칠 수 있다. 다음은 의도적인 중복이 더 나은 선택일 수 있는 대표적인 경우들이다.

- **거짓 중복:** 서로 다른 도메인 개념이 우연히 같은 형태의 코드를 가질 때.
- **복잡성 증가:** 추상화로 인해 발생하는 복잡성의 비용이 중복을 관리하는 비용보다 더 크다고 판단될 때 (KISS 원칙 우선).
- **초기 개발 단계:** 시스템의 패턴이 아직 안정화되지 않았을 때 ('세 번의 법칙'에 따라 추상화를 유보).
- **테스트 코드:** 각 테스트의 명료성과 독립성이 재사용성보다 더 중요할 때 (DAMP 원칙 우선).


완벽하게 DRY한 시스템은 도달해야 할 최종 상태가 아니라, 끊임없이 추구해야 할 이상향에 가깝다. 소프트웨어는 살아있는 유기체처럼 끊임없이 변화하고 진화하므로, DRY를 달성하는 것 역시 한 번의 작업으로 끝나는 것이 아니라 지속적인 리팩토링을 통해 점진적으로 개선해 나가는 여정이다.47

'깨진 창문 이론'처럼, 시스템 내의 작은 중복을 발견했을 때 이를 방치하지 않고 즉시 개선하려는 자세, 즉 '보이스카우트 규칙(Boy-Scout Rule: 언제나 처음 왔을 때보다 캠프장을 더 깨끗하게 만들고 떠나라)'을 실천하는 문화가 DRY한 코드베이스를 유지하는 비결이다.47

궁극적으로 DRY 원칙의 현명한 적용은 단순히 기술적인 기교를 넘어선다. 그것은 개발 중인 시스템의 본질을 깊이 이해하고, 비즈니스의 변화 방향을 예측하며, 미래의 불확실성에 유연하게 대응할 수 있는 아키텍처를 구축하려는 깊은 통찰에서 비롯된다. DRY는 코드를 작성하는 기술이 아니라, 지식을 구조화하는 예술에 더 가깝다.


1. [Software] DRY원칙이란?(Don't Repeat Yourself) - kim.dragon, 8월 16, 2025에 액세스, https://kim-dragon.tistory.com/256
2. en.wikipedia.org, 8월 16, 2025에 액세스, [https://en.wikipedia.org/wiki/Don%27t_repeat_yourself#:~:text=The%20DRY%20principle%20is%20stated,their%20book%20The%20Pragmatic%20Programmer.](https://en.wikipedia.org/wiki/Don't_repeat_yourself#:~:text=The DRY principle is stated,their book The Pragmatic Programmer.)
3. The DRY Principle: Embracing Efficiency in Software Engineering | by Michael Egger, 8월 16, 2025에 액세스, https://medium.com/@mesw1/the-dry-principle-embracing-efficiency-in-software-engineering-56e8efa62c07
4. DRY Principle | Software development - LiteBreeze, 8월 16, 2025에 액세스, https://litebreeze.com/software-development/dry-dont-repeat-yourself/
5. Don't repeat yourself - Wikipedia, 8월 16, 2025에 액세스, [https://en.wikipedia.org/wiki/Don%27t_repeat_yourself](https://en.wikipedia.org/wiki/Don't_repeat_yourself)
6. Don't repeat yourself - Wikipedia, 8월 16, 2025에 액세스, https://en.wikipedia.org/wiki/Write_everything_twice?useskin=vector
7. DRY principles: How to write efficient SQL - dbt Labs, 8월 16, 2025에 액세스, https://www.getdbt.com/blog/dry-principles
8. [ 개인공부 ] DRY 원칙, 8월 16, 2025에 액세스, [https://sykeem.tistory.com/entry/%EA%B0%9C%EC%9D%B8%EA%B3%B5%EB%B6%80-DRY-%EC%9B%90%EC%B9%99](https://sykeem.tistory.com/entry/개인공부-DRY-원칙)
9. The Pragmatic Programmer highlighted points: Ch 2, A Pragmatic Approach, 8월 16, 2025에 액세스, https://dev.to/chenxeed/the-pragmatic-programmer-highlighted-points-ch-2-a-pragmatic-approach-505l
10. 코드를 성급하게 DRY하지 마세요 - GeekNews, 8월 16, 2025에 액세스, https://news.hada.io/topic?id=15102
11. [소프트웨어 3대 개발 원칙] DRY에 대해서 알아보자. (Feat. WET), 8월 16, 2025에 액세스, https://iwillcomplete.tistory.com/90
12. What should I consider when the DRY and KISS principles are incompatible?, 8월 16, 2025에 액세스, https://softwareengineering.stackexchange.com/questions/400183/what-should-i-consider-when-the-dry-and-kiss-principles-are-incompatible
13. DRY (Don't Repeat Yourself) / Just Do It! And Then Some! - Backend Developer, 8월 16, 2025에 액세스, https://jaeseongdev.github.io/development/2021/04/09/DRY/
14. DRY is an over-rated programming principle? - Hacker News, 8월 16, 2025에 액세스, https://news.ycombinator.com/item?id=32010699
15. 클린 코드를 위한 소프트웨어 설계 원칙 - F-Lab, 8월 16, 2025에 액세스, https://f-lab.kr/insight/software-design-principles-for-clean-code
16. DRY 원칙 - 기계인간 John Grib, 8월 16, 2025에 액세스, https://johngrib.github.io/wiki/jargon/dry-principle/
17. Best Practices for Writing DRY (Don't Repeat Yourself) Code - PixelFreeStudio Blog, 8월 16, 2025에 액세스, https://blog.pixelfreestudio.com/best-practices-for-writing-dry-dont-repeat-yourself-code/
18. The DRY Principle - Codefinity, 8월 16, 2025에 액세스, https://codefinity.com/blog/The-DRY-Principle
19. The Don't-Repeat-Yourself (DRY) design principle in .NET Part 1, 8월 16, 2025에 액세스, https://dotnetcodr.com/2013/10/17/the-dont-repeat-yourself-dry-design-principle-in-net-part-1/
20. Does the DRY principle apply to database normalization, or is it applicable only to code?, 8월 16, 2025에 액세스, https://www.quora.com/Does-the-DRY-principle-apply-to-database-normalization-or-is-it-applicable-only-to-code
21. Mastering Normalization for Optimal SQL Server Schema Design | MoldStud, 8월 16, 2025에 액세스, https://moldstud.com/articles/p-master-normalization-for-sql-server-schema-design
22. DRY SQL – Writing simple, maintainable queries | Clean Database Development, 8월 16, 2025에 액세스, https://cleandatabase.wordpress.com/2017/08/31/dry-sql-writing-simple-maintainable-queries/
23. DRY Principle in Action: Streamlining Dependencies with Gradle Plugins, 8월 16, 2025에 액세스, https://iamjosephmj.medium.com/dry-principle-in-action-streamlining-dependencies-with-gradle-plugins-736ff16d4bbb
24. Gradle and Maven Comparison, 8월 16, 2025에 액세스, https://gradle.org/maven-and-gradle/
25. [개발서적] 실용주의 프로그래머 핵심 내용 정리 및 요약 - 망나니개발자 - 티스토리, 8월 16, 2025에 액세스, https://mangkyu.tistory.com/315
26. What Is The DRY Principle? - ITU Online IT Training, 8월 16, 2025에 액세스, https://www.ituonline.com/tech-definitions/what-is-the-dry-principle/
27. 좋은 문서를 만드는 8가지 원칙 - 한빛+, 8월 16, 2025에 액세스, https://m.hanbit.co.kr/channel/view.html?cmscode=CMS6877759475
28. [객체지향] 잘못된 DRY 원칙 적용 - Falcon - 티스토리, 8월 16, 2025에 액세스, https://m-falcon.tistory.com/744
29. DRY에 대해 어떻게 생각하세요? : r/Frontend - Reddit, 8월 16, 2025에 액세스, https://www.reddit.com/r/Frontend/comments/ry1vid/what_is_your_opinion_on_dry/?tl=ko
30. Clean Code Essentials: YAGNI, KISS, DRY - DEV Community, 8월 16, 2025에 액세스, https://dev.to/juniourrau/clean-code-essentials-yagni-kiss-and-dry-in-software-engineering-4i3j
31. [번역] DRY - 잘못된 추상화의 일반적인 원인 - velog, 8월 16, 2025에 액세스, https://velog.io/@eunbinn/dry-the-common-source-of-bad-abstractions
32. JAVA_DRY원칙. DRY원칙은 개발자들이 중요하게 생각하는 원칙입니다. | by zzory_bbong, 8월 16, 2025에 액세스, [https://medium.com/@duddk1551/java-dry%EC%9B%90%EC%B9%99-18abd5433b7f](https://medium.com/@duddk1551/java-dry원칙-18abd5433b7f)
33. 소프트웨어 개발의 3개의 KEY 원칙 : KISS,YAGNI,DRY, 8월 16, 2025에 액세스, https://hongjinhyeon.tistory.com/136
34. Avoid Hasty Abstractions (AHA).md - Artistudio, 8월 16, 2025에 액세스, https://brain.artistudio.xyz/artistudio/technical/resources/terms-concepts/principles/avoid-hasty-abstractions-aha-.md
35. Avoid Hasty Abstractions (AHA) - Francisco Moretti, 8월 16, 2025에 액세스, https://www.franciscomoretti.com/blog/avoid-hasty-abstractions-aha
36. Avoid Hasty Abstractions (AHA) - Epic Web Dev, 8월 16, 2025에 액세스, https://www.epicweb.dev/principles/craft/architecting-projects/aha
37. Avoid Hasty Abstractions (AHA) - mbark, 8월 16, 2025에 액세스, https://mbarkt3sto.hashnode.dev/avoid-hasty-abstractions-aha
38. Rule of three (computer programming) - Wikipedia, 8월 16, 2025에 액세스, https://en.wikipedia.org/wiki/Rule_of_three_(computer_programming)
39. The Rule of Three - Andrew Brookins, 8월 16, 2025에 액세스, https://andrewbrookins.com/technology/the-rule-of-three/
40. KISS vs. DRY vs. YAGNI: Understanding Key Software Development ..., 8월 16, 2025에 액세스, https://rrmartins.medium.com/kiss-vs-dry-vs-yagni-understanding-key-software-development-principles-e307b7419636
41. KISS, DRY, SOLID, YAGNI - A Simple Guide to Some Principles of Software Engineering and Clean Code | by HlfDev | Medium, 8월 16, 2025에 액세스, https://medium.com/@hlfdev/kiss-dry-solid-yagni-a-simple-guide-to-some-principles-of-software-engineering-and-clean-code-05e60233c79f
42. DRY vs DAMP in Unit Tests / Enterprise Craftsmanship, 8월 16, 2025에 액세스, https://enterprisecraftsmanship.com/posts/dry-damp-unit-tests/
43. DRY and Unit Tests don't mix well - Matthew Manela, 8월 16, 2025에 액세스, [https://matthewmanela.com/blog/dry-and-unit-tests-don%E2%80%99t-mix-well-2/](https://matthewmanela.com/blog/dry-and-unit-tests-don’t-mix-well-2/)
44. Why practicing DRY in tests is bad for you - DEV Community, 8월 16, 2025에 액세스, https://dev.to/mbarzeev/why-practicing-dry-in-tests-is-bad-for-you-j7f
45. Testing on the Toilet: Tests Too DRY? Make Them DAMP! - Google Testing Blog, 8월 16, 2025에 액세스, https://testing.googleblog.com/2019/12/testing-on-toilet-tests-too-dry-make.html
46. DRY and DAMP in Tests - Code With Arho, 8월 16, 2025에 액세스, https://www.arhohuttunen.com/dry-damp-tests/
47. The Pragmatic Programmer Quick Reference Guide, 8월 16, 2025에 액세스, [https://www.khoury.northeastern.edu/home/lieber/courses/csg110/sp08/Pragmatic%20Quick%20Reference.htm](https://www.khoury.northeastern.edu/home/lieber/courses/csg110/sp08/Pragmatic Quick Reference.htm)
48. Software Design Principles (Basics) | DRY, YAGNI, KISS, etc - workat.tech, 8월 16, 2025에 액세스, https://workat.tech/machine-coding/tutorial/software-design-principles-dry-yagni-eytrxfhz1fla

