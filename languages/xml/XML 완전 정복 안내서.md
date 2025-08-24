[XML](./index.md)
# XML 완전 정복 안내서


XML(eXtensible Markup Language)은 많은 현대 개발자에게 익숙하면서도 종종 오해받는 기술이다. 그 본질을 이해하려면, 데이터를 단순히 화면에 **보여주는(display)** 데 초점을 맞춘 HTML과의 근본적인 차이점부터 알아야 한다. XML의 설계 목적은 데이터를 **저장하고 전달**하며, 데이터 자체의 **구조와 의미를 기술**하는 데 있다.1 이는 서로 다른 시스템, 특히 인터넷으로 연결된 이기종 시스템 간에 데이터를 원활하게 교환하기 위한 목적에서 출발했다.4

XML은 1998년 W3C(World Wide Web Consortium)에 의해 표준으로 제정되었으며, 그 뿌리는 SGML(Standard Generalized Markup Language)에 두고 있다. XML은 SGML의 복잡성을 대폭 줄인 실용적인 부분집합으로 설계되었다.4 가장 핵심적인 특징 중 하나는 특정 목적을 가진 마크업 언어를 만드는 데 사용되는 **메타 언어(meta-language)**라는 점이다.1 이름에 포함된 '확장 가능(eXtensible)'이라는 단어가 바로 이 특성을 의미한다. 개발자는 미리 정해진 태그만 사용하는 것이 아니라, 데이터의 의미에 맞게 새로운 태그를 직접 정의할 수 있다. 이러한 확장성 덕분에 수학을 위한 MathML, 화학을 위한 CML, 지리 정보를 위한 GML 등 수많은 산업 표준 파생 언어가 탄생할 수 있었다.1

2000년대 초반, XML은 플랫폼에 독립적인 텍스트 기반 형식이라는 장점 덕분에 이기종 시스템 간 데이터 교환의 사실상 표준으로 군림했다.1 유니코드를 완벽하게 지원하여 어떤 운영체제나 프로그래밍 언어 환경에서도 데이터의 손실 없이 정보를 주고받을 수 있었고, 이는 데이터 무결성을 보장하는 핵심 가치로 여겨졌다.5 그러나 기술의 흐름은 변했다. 최근 웹 API와 모바일 애플리케이션 개발에서는 더 가볍고 파싱 속도가 빠른 JSON(JavaScript Object Notation)이 주도권을 잡으면서, XML은 구식 기술로 치부되기도 한다.1

이러한 상황은 XML의 현재 위치를 정의하는 근본적인 긴장감을 만들어낸다. XML은 한편으로는 정부, 금융, 복잡한 문서 표준 등 광범위한 디지털 인프라를 지탱하는 기반 기술이지만, 다른 한편으로는 JSON이 지배하는 현대 웹 개발의 맥락에서는 '레거시' 또는 '구식' 형식으로 간주된다. 따라서 2025년 현재 XML을 이해한다는 것은 사라져가는 언어를 배우는 것이 아니다. 데이터의 무결성, 구조적 복잡성, 그리고 공식적인 유효성 검사가 무엇보다 중요한 특정 문제 영역에서, XML이 왜 여전히 가장 강력하고 올바른 선택인지를 이해하고 그 전문성을 습득하는 것을 의미한다. 이 안내서는 바로 그 지점을 파고들어, XML의 핵심 문법부터 현대적 활용 사례와 미래 전망까지 심층적으로 분석할 것이다.

------


XML을 제대로 활용하기 위해서는 그 구조와 엄격한 문법 규칙을 정확히 이해하는 것이 선행되어야 한다. HTML과 유사해 보이지만, XML의 규칙은 훨씬 더 엄격하며, 이 규칙을 지키지 않으면 문서는 처리되지 않는다.


모든 XML 문서는 논리적으로 계층적인 트리 구조를 가진다. 이 구조의 최상단에는 단 하나의 **루트 요소(Root Element)**가 존재해야 하며, 모든 다른 요소들은 이 루트 요소 안에 중첩되는 형태로 구성된다.2

문서의 가장 첫 줄에는 **XML 프롤로그(Prolog)**를 선언할 수 있다. 이는 필수는 아니지만, 문서의 속성을 명확히 하기 위해 강력히 권장된다.10

XML

```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
```

- **프롤로그 규칙**: 프롤로그는 반드시 문서의 첫 번째 줄에 위치해야 하며, 다른 어떤 태그나 주석보다도 앞에 와야 한다. 태그명 `xml`은 반드시 소문자로 작성해야 한다.10
- `version`: 사용된 XML의 버전을 명시한다. 현재 대부분 `1.0`을 사용한다.10
- `encoding`: 문서의 문자 인코딩 방식을 지정한다. 전 세계의 거의 모든 문자를 표현할 수 있는 `UTF-8`이 기본값이자 가장 널리 사용된다.4
- `standalone`: 문서가 외부 문서 정의(DTD 등)에 의존하는지 여부를 파서에게 알리는 속성이다. `yes`는 외부 의존성이 없음을, `no`는 외부 문서를 참조해야 함을 의미한다. 기본값은 `no`이다.12


XML 문서는 세 가지 핵심 구성 요소로 이루어진다.

- **요소(Element)**: XML의 기본 빌딩 블록으로, 시작 태그, 내용(콘텐츠), 종료 태그로 구성된다. 예를 들어, `<book>XML Guide</book>` 전체가 하나의 요소다.14

- **태그(Tag)**: 요소를 정의하기 위해 꺾쇠괄호(`<>`)로 감싼 이름이다. 모든 시작 태그(`<book>`)는 반드시 그에 대응하는 종료 태그(`</book>`)를 가져야 한다.12 만약 요소 내에 아무런 내용이 없다면, 

  `<br></br>` 대신 `<br/>`와 같이 하나의 태그로 축약하여 표현할 수 있다. 이를 빈 요소(empty element)라고 한다.12

- **속성(Attribute)**: 요소에 대한 추가적인 정보를 제공하는 역할을 한다. 속성은 항상 시작 태그 안에 `이름="값"`의 형태로 정의되며, 속성값은 반드시 큰따옴표(`"`) 또는 작은따옴표(`'`)로 감싸야 한다.12 하나의 요소는 여러 개의 속성을 가질 수 있지만, 같은 이름의 속성을 두 번 이상 사용할 수는 없다.14

XML

```
<book category="Programming" published="2024">
  <title>XML Guide</title>
</book>
```

위 예제에서 `book`은 `category`와 `published`라는 두 개의 속성을 가진 요소이며, `title`이라는 자식 요소를 포함하고 있다.


XML은 데이터를 표현하고 처리하는 과정에서 혼란을 막기 위해 매우 엄격한 문법 체계를 가지고 있다.

- **주석(Comment)**: ``로 끝난다. 주석 안에는 `--` 문자열을 포함할 수 없으며, 주석을 중첩시키는 것도 허용되지 않는다.4
- **CDATA 섹션**: Character Data의 약자로, 파서가 마크업으로 해석하지 않고 순수한 문자 데이터로 처리해야 하는 부분을 지정할 때 사용한다. HTML 태그나 스크립트 코드를 XML 문서 내에 그대로 포함시키고 싶을 때 유용하다. `<!]>`로 끝나며, `CDATA` 키워드는 반드시 대문자로 써야 한다.14
- **엔티티(Entity)**: XML 문법에서 특별한 의미를 가지는 문자들(예: `<`, `>`, `&`)을 텍스트 내용으로 사용하려면, 미리 정의된 엔티티로 변환해야 한다. 예를 들어, `<`는 `<`, `>`는 `>`, `&`는 `&`로 표현해야 한다.12

이 외에도 반드시 지켜야 할 핵심 규칙들이 있다. 이 규칙들은 XML을 HTML이나 JSON과 구별하는 중요한 특징이다.

- **모든 태그는 닫혀야 한다**: 시작 태그가 있다면 반드시 종료 태그가 있어야 한다.10
- **대소문자 구분**: 태그와 속성 이름은 대소문자를 엄격하게 구분한다. `<Book>`과 `<book>`은 완전히 다른 요소로 인식된다.12
- **올바른 중첩**: 태그는 수학의 괄호처럼 올바르게 중첩되어야 한다. 나중에 시작된 태그가 먼저 닫혀야 한다. 예를 들어, `<b><i>text</i></b>`는 올바르지만, `<b><i>text</b></i>`는 잘못된 중첩으로 오류를 발생시킨다.10
- **공백 처리**: HTML과 달리, XML은 텍스트 내의 띄어쓰기와 줄 바꿈을 있는 그대로 보존하고 인식한다.12
- **이름 규칙**: 요소와 속성의 이름은 반드시 문자(letter)나 밑줄(`_`)로 시작해야 한다. 이름 중간에 공백을 포함할 수 없으며, `xml`이라는 문자열(대소문자 무관)로 시작할 수 없다.14

| 규칙 (Rule)        | 설명 (Description)                                         | 올바른 예시 (Correct Example) | 잘못된 예시 (Incorrect Example)    |
| ------------------ | ---------------------------------------------------------- | ----------------------------- | ---------------------------------- |
| **루트 요소**      | 문서는 단 하나의 최상위 요소를 가져야 함                   | `<root><child/></root>`       | `<root/><another_root/>`           |
| **태그 닫기**      | 모든 시작 태그는 종료 태그를 갖거나, 빈 태그 형식이어야 함 | `<p>text</p>`, `<br/>`        | `<p>text`, `<br>`                  |
| **대소문자 구분**  | 태그와 속성 이름은 대소문자를 구분함                       | `<book>...</book>`            | `<book>...</Book>`                 |
| **중첩 순서**      | 나중에 열린 태그가 먼저 닫혀야 함                          | `<b><i>text</i></b>`          | `<b><i>text</b></i>`               |
| **속성 값 따옴표** | 모든 속성값은 따옴표로 감싸야 함                           | `<a href="url">`              | `<a href=url>`                     |
| **공백 처리**      | 텍스트 내의 공백과 줄 바꿈을 그대로 인식함                 | `<p>Hello   World</p>`        | (HTML과 달리 공백이 유지됨)        |
| **이름 규칙**      | 문자나 `_`로 시작, 공백 포함 불가, 'xml' 시작 불가         | `<_my-tag>`, `<item1>`        | `<1item>`, `<my tag>`, `<xml-tag>` |

------


XML 문서의 품질을 논할 때 '정형(Well-formed)'과 '유효(Valid)'라는 두 가지 중요한 개념이 등장한다. 이 둘의 차이를 이해하는 것은 XML의 설계 철학을 파악하는 데 있어 핵심적이다.


'정형(Well-formed)' XML이란 1장에서 설명한 모든 **구문 규칙(syntax rules)**을 완벽하게 준수한 문서를 의미한다.15 이는 XML 문서가 되기 위한 최소한의 자격 요건이다.

- 단 하나의 루트 요소를 가질 것
- 모든 시작 태그는 짝이 맞는 종료 태그를 가질 것
- 태그는 올바르게 중첩될 것
- 속성 값은 따옴표로 감싸여 있을 것
- 특수 문자는 엔티티로 올바르게 표현될 것

이러한 기본 문법을 하나라도 어기면 그 문서는 '정형'이 아니며, 기술적으로 더 이상 XML 문서로 취급되지 않는다. XML 파서(parser)는 정형이 아닌 문서를 만나면 즉시 처리를 중단하고 오류를 보고한다.16 따라서 정형성은 모든 XML 처리 과정의 가장 기본적인 전제 조건이다.


'유효(Valid)'한 XML 문서는 정형이라는 기본 조건을 만족하는 동시에, 한 단계 더 나아가 미리 정의된 **스키마(Schema)**의 규칙까지 준수하는 문서를 말한다.15 스키마는 특정 XML 문서가 가져야 할 구조적인 '설계도' 또는 '문법서'와 같다. 스키마는 다음과 같은 규칙들을 정의한다 18:

- 문서에 나타날 수 있는 요소와 속성의 목록
- 각 요소의 자식 요소와 그들의 순서 및 출현 횟수(예: `title` 요소는 반드시 하나만 있어야 하고, `author` 요소는 하나 이상 올 수 있다)
- 각 요소와 속성이 가질 수 있는 데이터의 타입(예: `price` 속성은 숫자여야 한다)

논리적으로 모든 유효한 문서는 반드시 정형이다. 하지만 모든 정형 문서가 유효한 것은 아니다. 유효성은 스키마라는 추가적인 계약을 만족해야만 얻을 수 있는 더 높은 수준의 품질 보증이다.16

이러한 정형성과 유효성의 구분은 단순한 기술적 구분이 아니라, XML의 핵심 설계 철학을 보여준다. 이는 데이터 품질을 보장하기 위한 2단계 시스템으로 작동한다. 1단계인 **정형성**은 최소한의 문법적 예측 가능성을 보장하여, 어떤 범용 XML 도구라도 문서를 일단 파싱할 수 있게 해준다. 2단계인 **유효성**은 특정 애플리케이션이나 시스템 간에 합의된 '계약'(스키마)에 따라 데이터의 의미적, 구조적 정확성을 보장한다. 이처럼 내장된 강력한 유효성 검사 기능이야말로, 외부 라이브러리에 의존해야 하는 JSON과 같은 형식에 비해 XML이 갖는 가장 큰 전략적 우위다.20


XML 문서의 유효성을 검사하기 위한 스키마를 정의하는 언어로는 대표적으로 DTD와 XML 스키마(XSD)가 있다.

- **DTD (Document Type Definition)**: XML 초창기부터 사용된 전통적인 스키마 정의 방식이다.18 XML 문서 상단에 

  `<!DOCTYPE...>` 선언을 통해 DTD를 명시하고, 이를 기준으로 유효성을 검사한다.13 하지만 DTD는 몇 가지 치명적인 단점을 가지고 있다. 첫째, DTD 자체의 문법이 XML이 아니어서 별도의 파싱 로직이 필요할 수 있다. 둘째, 데이터 타입을 정수, 날짜 등으로 세밀하게 정의하는 기능이 매우 빈약하다. 셋째, 태그 이름의 충돌을 방지하는 네임스페이스(Namespace)를 지원하지 않는다.15

- **XML 스키마 (XSD - XML Schema Definition)**: DTD의 한계를 극복하기 위해 W3C에서 새로 제정한 강력하고 현대적인 스키마 언어다.15 XSD는 다음과 같은 압도적인 장점을 가진다.

  - **XML 기반 문법**: XSD 파일 자체가 XML로 작성되기 때문에, 일반적인 XML 파서와 도구를 그대로 활용하여 스키마를 읽고 처리할 수 있다.
  - **강력한 데이터 타입**: 정수, 소수, 날짜, 시간과 같은 내장 데이터 타입은 물론, 정규식을 이용한 문자열 패턴 정의 등 매우 풍부하고 정교한 데이터 타입을 지원한다.
  - **네임스페이스 지원**: 네임스페이스를 완벽하게 지원하여, 여러 스키마를 조합하여 사용하는 복잡한 문서 구조도 명확하게 정의하고 검증할 수 있다.15

오늘날 대부분의 엔터프라이즈 환경에서는 DTD보다 월등한 기능과 유연성을 제공하는 XSD가 유효성 검증의 표준으로 사용되고 있으며, Xerces와 같은 주요 XML 파서들은 XSD 기반의 유효성 검사를 완벽하게 지원한다.15

------


XML은 단순히 데이터를 담는 그릇이 아니다. 그 데이터를 효과적으로 탐색하고, 변환하며, 질의하기 위한 강력한 기술 생태계를 함께 가지고 있다. XPath, XSLT, XQuery는 이 생태계의 삼총사라 할 수 있다.


XPath는 XML 문서 내의 특정 요소, 속성, 텍스트 노드 등 원하는 부분에 **접근하고 선택하기 위한 경로 언어(Path Language)**다.23 마치 파일 시스템에서 디렉터리 경로를 사용해 파일에 접근하듯, XPath는 

`/`와 `//` 같은 경로 표현식을 사용해 XML의 계층 구조를 자유롭게 탐색한다.25 XPath는 그 자체로도 유용하지만, 주로 XSLT와 XQuery의 핵심 엔진으로 작동하며, 이들 언어가 XML 데이터에 접근하는 기반을 제공한다.22

주요 경로 표현식은 다음과 같다.

- **절대 경로**: 문서의 루트 노드에서부터 시작하며, `/`로 경로를 구분한다. 예: `/bookstore/book/title`은 루트 요소인 `bookstore` 아래의 `book` 요소 아래의 `title` 요소를 선택한다.27
- **상대 경로**: `//`로 시작하며, 현재 위치에 상관없이 문서 내에서 특정 조건을 만족하는 모든 노드를 검색한다. 훨씬 유연하고 강력하다. 예: `//title`은 문서 어디에 있든 모든 `title` 요소를 선택한다.27
- **속성 선택**: 대괄호 ``와 `@` 기호를 사용하여 특정 속성이나 속성값을 가진 요소를 선택한다. 예: `//book[@category='COOKING']`은 `category` 속성값이 'COOKING'인 모든 `book` 요소를 찾는다.29
- **함수 활용**: `contains()`, `starts-with()`, `text()` 등 다양한 내장 함수를 통해 더욱 정교한 조건의 노드를 선택할 수 있다. 예: `//title[contains(text(), 'Java')]`는 텍스트 내용에 'Java'를 포함하는 모든 `title` 요소를 선택한다.28
- **축(Axes)**: `ancestor`(조상), `descendant`(자손), `following-sibling`(다음 형제), `parent`(부모) 등 현재 노드를 기준으로 다양한 관계에 있는 노드를 정밀하게 선택할 수 있는 고급 기능이다.27


XSLT는 XML 문서를 전혀 다른 형태의 문서, 예를 들어 HTML, 다른 구조의 XML, 또는 일반 텍스트 파일로 **변환(Transform)**하기 위해 설계된 언어다.31 XSLT는 절차적 프로그래밍 언어와 달리, **선언적(declarative)이고 템플릿 기반(template-based)**으로 동작한다.

XSLT의 처리 모델은 다음과 같은 핵심 요소로 구성된다.

- `<xsl:stylesheet>`: 모든 XSLT 문서의 루트 요소.
- `<xsl:template match="pattern">`: 변환 규칙의 핵심. `match` 속성에는 XPath 표현식이 들어가며, 소스 XML에서 해당 패턴과 일치하는 노드를 발견했을 때 이 템플릿 안의 내용을 적용하여 결과를 생성한다.33
- `<xsl:apply-templates>`: 현재 처리 중인 노드의 자식 노드들로 이동하여, 그 자식 노드에 맞는 템플릿을 다시 찾아 재귀적으로 변환 과정을 계속하도록 지시한다. 이를 'Push' 방식 처리라고 한다.31
- `<xsl:value-of select="expression">`: XPath 표현식으로 선택된 노드의 텍스트 값을 결과물에 출력한다.31
- `<xsl:for-each select="expression">`: 선택된 노드 집합을 순회하며 반복적으로 특정 작업을 수행한다. 이는 'Pull' 방식 처리에 해당한다.33

예를 들어, XML로 된 데이터 목록을 HTML 테이블로 변환하는 작업에 XSLT는 최적의 도구다.


XQuery는 이름에서 알 수 있듯 XML 데이터를 위한 **질의 언어(Query Language)**다.26 관계형 데이터베이스에 질의하기 위해 SQL을 사용하듯, XQuery는 방대한 XML 문서나 XML 데이터베이스에서 원하는 데이터를 효율적으로 추출하고 조작하기 위해 사용된다.35

XQuery의 심장은 **FLWOR 표현식**이다. 이는 SQL의 `SELECT-FROM-WHERE` 구문과 매우 유사한 구조를 가진다.22

- `For`: 변수를 특정 노드 시퀀스(집합)에 바인딩하며 순회한다.
- `Let`: 변수에 특정 값을 할당한다.
- `Where`: 결과를 필터링할 조건을 지정한다.
- `Order by`: 결과를 특정 기준으로 정렬한다.
- `Return`: 위 과정을 거쳐 최종적으로 생성할 결과 XML의 형태를 정의하고 반환한다.

XQuery는 단일 문서를 넘어 여러 XML 문서의 컬렉션을 대상으로 복잡한 조인, 필터링, 집계, 정렬 연산을 수행하는 데 매우 강력한 성능을 발휘한다.26


XML 문서를 프로그래밍 언어 내에서 처리(파싱)할 때는 크게 두 가지 API 모델, DOM과 SAX를 사용한다. 둘의 차이는 컴퓨터 과학의 근본적인 개념인 **공간-시간 트레이드오프(space-time tradeoff)**를 명확하게 보여준다.

- **DOM (Document Object Model)**
  - **처리 방식**: XML 문서 전체를 메모리에 **트리 구조로 완전히 로드**한 후, 각 노드에 자유롭게 접근(random access)하여 데이터를 처리하는 방식이다.37
  - **장점**: 일단 메모리에 로드되면, 특정 노드를 찾거나 수정, 추가, 삭제하는 작업이 매우 빠르고 편리하다. 개발자 입장에서 다루기 직관적이다.37
  - **단점**: 문서 전체를 메모리에 올려야 하므로, 기가바이트 단위의 대용량 XML 파일을 처리할 경우 엄청난 메모리를 소모하여 시스템 성능을 저하시키거나 `OutOfMemoryError`를 유발할 수 있다. 한 성능 테스트에 따르면, DOM은 SAX에 비해 약 5배에서 10배에 달하는 메모리를 사용한다.37
- **SAX (Simple API for XML)**
  - **처리 방식**: 문서를 처음부터 끝까지 순차적으로 한 줄씩 읽어 들이면서, 태그의 시작, 태그의 끝, 텍스트 내용 등을 만날 때마다 **이벤트(event)**를 발생시키는 스트리밍 방식이다.37
  - **장점**: 문서 전체를 메모리에 저장하지 않으므로, 매우 적은 양의 메모리만으로 아무리 큰 파일이라도 처리할 수 있다. 대용량 데이터 처리에 절대적으로 유리하다.37
  - **단점**: 처리가 한 방향(forward-only)으로만 진행되기 때문에 이미 읽고 지나간 데이터로 되돌아갈 수 없다. 따라서 문서의 구조를 변경하거나 특정 노드를 수정하는 작업이 매우 복잡하고 어렵다.37

이처럼 DOM은 메모리 공간(space)을 희생하여 처리 시간(time)과 개발 편의성을 얻는 전략이고, SAX는 처리 시간(더 복잡한 프로그래밍)과 유연성을 희생하여 메모리 공간을 아끼는 전략이다. 따라서 어떤 파서를 선택할지는 단순히 어떤 것이 더 좋고 나쁨의 문제가 아니라, 처리할 데이터의 크기와 애플리케이션의 요구사항을 분석하여 내리는 전략적인 아키텍처 결정에 해당한다.

| 구분 (Category)    | DOM 파서 (DOM Parser)                              | SAX 파서 (SAX Parser)                                        |
| ------------------ | -------------------------------------------------- | ------------------------------------------------------------ |
| **처리 방식**      | 문서 전체를 메모리에 트리 구조로 로드 (Tree-based) | 문서를 순차적으로 읽으며 이벤트 발생 (Event-based, Streaming) |
| **메모리 사용량**  | 매우 높음 (문서 크기의 약 10배) 37                 | 매우 낮음 (문서 크기와 무관하게 일정) 39                     |
| **처리 속도**      | 작은 파일에서는 빠르나, 로딩 시간이 김             | 대용량 파일에서 월등히 빠름                                  |
| **데이터 접근**    | 무작위 접근 가능 (Random Access)                   | 순방향 접근만 가능 (Forward-only)                            |
| **데이터 수정**    | 용이함 (노드 추가, 삭제, 변경이 자유로움)          | 매우 복잡하고 어려움                                         |
| **주요 장점**      | 데이터 구조 변경 및 탐색이 편리함                  | 메모리 효율성이 극도로 높음                                  |
| **주요 단점**      | 대용량 파일 처리 시 메모리 부족 문제 발생          | 데이터 구조를 파악하고 상태를 직접 관리해야 함               |
| **추천 사용 사례** | 크기가 작고 구조 변경이 잦은 설정 파일 처리        | 대용량 로그 파일, 데이터 피드, 실시간 스트림 처리            |

------


현대 웹 개발, 특히 API 통신에서 XML은 강력한 경쟁자인 JSON과 자주 비교된다. 두 기술은 데이터 교환이라는 공통된 목적을 가졌지만, 철학과 구조, 장단점에서 뚜렷한 차이를 보인다. 어떤 상황에서 어떤 기술을 선택해야 하는지 판단하려면 이 차이점을 명확히 이해해야 한다.


- **XML**: 시작 태그와 종료 태그를 모두 사용해야 하는 태그 기반 구조 때문에 문법이 장황(verbose)하다. 이는 필연적으로 파일 크기를 증가시키는 원인이 된다.6 하지만 태그 이름과 속성을 통해 데이터의 의미와 메타데이터를 명확하게 기술할 수 있다는 장점이 있다.19
- **JSON**: `키:값` 쌍으로 이루어진 객체와 값들의 리스트인 배열을 기반으로 한다. 문법이 매우 간결하여 사람이 읽고 쓰기 편하며, 동일한 데이터를 표현하더라도 파일 크기가 훨씬 작아 네트워크 전송 효율이 높다.20
- **주석 및 메타데이터**: XML은 `` 구문을 통해 주석을 공식적으로 지원하며, 속성(attribute)을 이용해 데이터 자체와 분리된 메타데이터를 표현하기에 매우 용이하다. 반면, JSON은 표준 명세에 주석 기능이 없으며, 모든 정보는 `키:값` 쌍으로 표현되어야 한다.19


- **파싱(Parsing)**: JSON은 JavaScript 문법의 일부(subset)에서 파생되었기 때문에, 특히 웹 브라우저를 포함한 JavaScript 환경에서 내장 함수(`JSON.parse()`)를 통해 매우 빠르고 간단하게 파싱된다.20 반면 XML은 구조의 복잡성으로 인해 전용 파서가 필요하며, 일반적으로 파싱 과정이 JSON보다 더 많은 연산을 요구하여 상대적으로 느리다.20
- **데이터 타입**: JSON은 숫자, 문자열, 불리언(true/false), 배열, 객체라는 단순하고 명확한 데이터 타입을 지원한다.6 XML 자체는 모든 것을 텍스트로 취급하지만, 스키마(XSD)와 결합하면 이야기가 달라진다. XSD를 통해 정수, 소수, 날짜, 시간 등 매우 풍부하고 사용자 정의된 데이터 타입을 엄격하게 정의하고 검증할 수 있다. 또한 네임스페이스, 이미지나 문서 같은 바이너리 데이터(Base64 인코딩) 등 훨씬 넓은 범위의 데이터를 포괄적으로 표현할 수 있다.6


어떤 데이터 형식이 더 안전한지에 대한 논의는 종종 오해를 불러일으킨다. XML과 JSON 중 어느 하나가 본질적으로 더 안전하거나 위험한 것은 아니다. 각각의 보안 위협은 데이터 형식 자체가 아닌, 그 형식을 처리하는 파서의 특정 기능이나 잘못된 구현 방식에서 비롯된다.

- XML 보안 위협: XXE (XML External Entity) Injection

  XML의 가장 심각하고 잘 알려진 보안 취약점은 XXE 공격이다. 이는 XML 파서가 DTD(문서 유형 정의)나 외부 엔티티(external entity) 참조를 처리하도록 설정되어 있을 때 발생한다. 공격자는 악의적으로 조작된 XML을 전송하여 파서가 서버의 로컬 파일을 읽게 하거나, 내부 네트워크의 다른 시스템을 스캔하거나, 서비스 거부(DoS) 공격을 유발할 수 있다.44 따라서 XML 파서를 사용할 때는 

  **DTD 처리와 외부 엔티티 로드를 명시적으로 비활성화**하는 것이 가장 중요한 보안 수칙이다.43

- JSON 보안 위협: Injection 및 Insecure Parsing

  JSON은 구조가 단순하여 XXE와 같은 복잡한 기능에서 비롯된 취약점은 없다. 그러나 처리 방식에서 문제가 발생할 수 있다.

  - **Insecure Parsing (안전하지 않은 파싱)**: 과거에 일부 개발자들이 JSON 문자열을 JavaScript 객체로 변환하기 위해 `eval()` 함수를 사용하는 경우가 있었다. 이는 치명적인 실수로, JSON 문자열에 악의적인 JavaScript 코드가 포함되어 있다면 그대로 실행되어 XSS(Cross-Site Scripting) 공격으로 이어질 수 있다. 항상 각 언어에서 제공하는 안전한 내장 파서(예: JavaScript의 `JSON.parse()`)를 사용해야 한다.45
  - **JSON Injection**: 애플리케이션이 사용자 입력을 제대로 검증하지 않고 JSON 데이터에 포함시킬 경우, 공격자가 의도치 않은 데이터를 주입하여 시스템의 논리를 교란시킬 수 있다.44
  - **JSONP (JSON with Padding)**: 다른 도메인 간 데이터 요청을 위해 과거에 사용되던 기술로, 현재는 CORS로 대체되었다. JSONP는 CSRF(Cross-Site Request Forgery) 공격에 취약할 수 있다.21

결론적으로, 보안은 데이터 형식을 선택하는 문제가 아니라, 개발자가 각 형식의 특성을 이해하고 안전한 구현 관행을 따르는지에 달려있다. XML의 위험성은 강력한 레거시 기능에 있고, JSON의 위험성은 JavaScript와의 긴밀한 관계와 부주의한 처리 방식에 있다.

| 기준 (Criteria)          | XML (eXtensible Markup Language)                             | JSON (JavaScript Object Notation)                            |
| ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **형식 (Format)**        | 태그 기반의 트리 구조                                        | 키-값 쌍(key-value pair) 기반의 맵과 배열 구조               |
| **구문 (Syntax)**        | 시작/종료 태그로 인해 장황함(verbose)                        | 간결하고 최소한의 문법 구조                                  |
| **가독성 (Readability)** | 구조가 명확하지만 태그 때문에 복잡해 보일 수 있음            | 매우 높음, 사람이 읽고 쓰기 쉬움                             |
| **파싱 (Parsing)**       | 전용 XML 파서 필요, 상대적으로 느림                          | 대부분 언어에 내장된 함수로 파싱, 매우 빠름                  |
| **스키마/유효성 검사**   | DTD, XSD를 통해 강력하고 내장된 유효성 검사 지원             | 스키마가 내장되어 있지 않음 (JSON Schema 등 외부 도구 필요)  |
| **데이터 타입**          | XSD를 통해 매우 풍부하고 사용자 정의된 타입 지원             | 숫자, 문자열, 불리언, 배열, 객체 등 제한된 기본 타입 지원    |
| **네임스페이스**         | 지원함 (태그 이름 충돌 방지)                                 | 지원하지 않음                                                |
| **주석 (Comments)**      | 공식적으로 지원함 (``)                                       | 공식적으로 지원하지 않음                                     |
| **보안 취약점**          | XXE(XML External Entity) Injection                           | Insecure Parsing(`eval`), JSON Injection, CSRF(JSONP 사용 시) |
| **주요 사용 사례**       | 복잡한 문서, 설정 파일, 데이터 무결성이 중요한 B2B/정부 데이터 교환 | 웹 API, 모바일 앱, 경량 데이터 교환, 실시간 통신             |

------


JSON이 웹의 새로운 표준으로 떠오른 지금, XML은 과연 과거의 유물이 되어가고 있는가? 답은 '아니오'다. XML은 범용 데이터 교환의 왕좌를 내주었을지 몰라도, 특정 전문 분야에서는 그 누구도 대체할 수 없는 독보적인 위치를 차지하며 여전히 활발하게 사용되고 진화하고 있다.


XML은 우리 주변의 수많은 핵심 기술과 시스템의 근간을 이루고 있다.

- **설정 파일(Configuration Files)**: XML의 계층적 구조, 주석 지원, 유효성 검사 기능은 복잡한 애플리케이션 설정을 기술하는 데 이상적이다. Java 생태계의 빌드 도구인 Maven의 `pom.xml`, Spring 프레임워크의 설정 파일, 그리고 수백만 개발자가 사용하는 안드로이드 애플리케이션의 `AndroidManifest.xml`과 UI 레이아웃 파일은 모두 XML을 표준으로 사용한다.46
- **벡터 그래픽(Vector Graphics)**: 웹에서 널리 사용되는 SVG(Scalable Vector Graphics)는 XML 기반의 W3C 표준이다. 이미지의 모든 모양, 경로, 색상을 XML 태그로 정의하기 때문에, 해상도에 상관없이 깨끗하게 확대/축소할 수 있으며, 텍스트 편집기로 직접 수정하거나 CSS와 JavaScript로 동적인 제어가 가능하다.46 안드로이드의 Vector Drawable 역시 SVG와 유사한 XML 기반 형식을 사용한다.50
- **오피스 문서(Office Documents)**: 우리가 매일 사용하는 Microsoft Office의 워드(.docx), 엑셀(.xlsx), 파워포인트(.pptx) 파일의 실체는 수많은 XML 파일과 리소스를 ZIP으로 압축한 것이다. 이 OOXML(Office Open XML) 형식은 복잡한 문서의 구조, 서식, 메타데이터, 포함된 객체 등을 표현하는 데 XML이 얼마나 강력한지를 보여주는 가장 대표적인 사례다.52
- **엔터프라이즈 및 정부 데이터 교환**: 데이터의 정확성과 무결성이 법적, 금전적 책임과 직결되는 분야에서 XML의 엄격한 스키마 기반 유효성 검사는 대체 불가능한 가치를 지닌다. 금융 파생상품 거래를 위한 FpML, 의료 정보 교환을 위한 HL7, 전자상거래를 위한 UBL과 같은 산업 표준은 모두 XML을 기반으로 한다.53 또한, 미국 국세청(IRS)의 세금 보고, 재무부(TreasuryDirect)의 채권 정보, Grants.gov의 정부 보조금 데이터 등 수많은 정부 기관의 공식 데이터 교환 형식으로 활발히 사용되고 있다.54


XML의 미래는 소멸이 아닌 **전문화(Specialization)**의 길을 걷고 있다. 범용 데이터 교환이라는 역할은 JSON에 넘겨주었지만, 특정 고부가가치 문제 영역에서는 오히려 그 입지를 더욱 공고히 하고 있다.

이러한 전망을 뒷받침하는 가장 강력한 증거는 주요 기관들의 지속적인 투자다. 단순히 기존 레거시 시스템을 유지하는 수준을 넘어, 2025년 이후에 적용될 **새로운 XML 스키마**가 지금도 활발하게 개발되고 있다는 사실은 XML의 장기적인 생명력을 증명한다. 예를 들어, 미국 재무부는 2025년 5월 이후 적용될 새로운 국채 매입 XML 형식을 발표했고 54, 국세청은 2025년 세금 보고를 위한 신규 XML 스키마를 제공하고 있으며 55, 교육부는 2025-26 학자금 지원을 위한 COD Common Record XML 스키마 버전 5.0c를 발표했다.57 이는 법규 준수와 데이터 무결성이 핵심인 영역에서 XML이 여전히 신뢰받는 표준 기술임을 명백히 보여준다.

따라서 XML의 미래는 다음과 같이 요약할 수 있다.

- **JSON의 지배력 확대**: 웹/모바일 API, 마이크로서비스 간 통신 등 경량 데이터 교환이 필요한 대부분의 신규 프로젝트에서는 JSON이 합리적인 기본 선택지로 남을 것이다.9
- **XML의 전문화 및 심화**: XML은 다음 영역에서 대체 불가능한 기술로 남을 것이다.
  - **데이터 무결성이 핵심인 분야**: 스키마를 통한 강력한 유효성 검사가 필수적인 금융, 정부, 의료, 법률 데이터 교환의 표준으로 계속 사용될 것이다.
  - **문서 중심(Document-centric) 애플리케이션**: 내용, 구조, 서식이 복잡하게 얽힌 디지털 출판, 기술 문서, 콘텐츠 관리 시스템(CMS)에서 XML은 여전히 최적의 솔루션이다.5
  - **레거시 시스템과의 상호운용성**: 수십 년간 축적된 수많은 엔터프라이즈 시스템이 XML을 기반으로 구축되었기 때문에, 이들과의 안정적인 연동을 위해 XML은 계속해서 중요한 역할을 수행할 것이다.58


결론적으로, "XML vs. JSON"은 더 이상 어느 한쪽을 선택해야 하는 제로섬 게임이 아니다. 현대 개발자에게 필요한 역량은 두 기술의 철학과 장단점을 모두 깊이 이해하고, 당면한 문제의 성격에 따라 **가장 적합한 도구를 선택하는 능력**이다.

- 새로운 REST API를 설계하고, 웹 클라이언트와 빠른 데이터를 주고받아야 한다면 **JSON**이 합리적인 첫 선택이다.
- 하지만 정부 기관의 공식 데이터를 파싱하거나, 금융 기관과 표준화된 거래 데이터를 교환하거나, SVG 그래픽을 동적으로 생성하거나, 복잡한 엔터프라이즈 애플리케이션의 설정을 관리해야 한다면, **XML**과 그 생태계(XPath, XSLT, XSD)에 대한 깊이 있는 지식은 당신을 평범한 개발자를 넘어선 문제 해결 전문가로 만들어 줄 것이다.

XML은 죽지 않았다. 단지 더 중요하고 복잡한 문제들을 해결하기 위해, 화려한 무대 위에서 조용한 무대 뒤로 자리를 옮겼을 뿐이다.


1. XML 개요 (개념, 설계목표) - 빨간색코딩 - 티스토리, accessed July 27, 2025, https://sjh836.tistory.com/117
2. XML 기초 문법 정리 - Inpa Dev ‍ - 티스토리, accessed July 27, 2025, [https://inpa.tistory.com/entry/XML-%F0%9F%93%91-XML-%EA%B8%B0%EC%B4%88-%EC%A0%95%EB%A6%AC](https://inpa.tistory.com/entry/XML-📑-XML-기초-정리)
3. XML이란 무엇인가? - 팔만코딩경 Mobile, accessed July 27, 2025, https://80000coding.oopy.io/e900f62f-3f47-454e-94a7-d4bc08b19219
4. XML - 위키백과, 우리 모두의 백과사전, accessed July 27, 2025, https://ko.wikipedia.org/wiki/XML
5. XML이란 무엇인가요? - Extensible Markup Language(XML) 설명 - AWS, accessed July 27, 2025, https://aws.amazon.com/ko/what-is/xml/
6. JSON과 XML 비교 - 데이터 표현 간의 차이점 - AWS, accessed July 27, 2025, https://aws.amazon.com/ko/compare/the-difference-between-json-xml/
7. XML다루기 - 6. XML 의 장점과 단점 - Kylog - 티스토리, accessed July 27, 2025, https://kylog.tistory.com/44
8. xml 장점 단점 - 재호와 함께하는 게임프로그래밍~~ㅋㅋ - 티스토리, accessed July 27, 2025, [https://jaehogame.tistory.com/entry/xml-%EC%9E%A5%EC%A0%90-%EB%8B%A8%EC%A0%90](https://jaehogame.tistory.com/entry/xml-장점-단점)
9. [프로그래밍] XML과 JSON의 차이 - 길은 가면, 뒤에 있다. - 티스토리, accessed July 27, 2025, https://12bme.tistory.com/202
10. [XML] prolog와 기본 문법, 구조 이해하기 - 코딩하는 핑구, accessed July 27, 2025, https://studyingpingu.tistory.com/63
11. XML 문법 - 코딩의 시작, TCP School, accessed July 27, 2025, https://tcpschool.com/xml/xml_basic_syntax
12. XML - Gray Cloud - 티스토리, accessed July 27, 2025, https://0707gray.tistory.com/55
13. [XML] XML 용법 - 구성요소, 문법 등 - Daisy's IT Blog - 티스토리, accessed July 27, 2025, https://webstudynote.tistory.com/110
14. [XML응용]XML 구조와 문법 정리 - 명우니닷컴 - 티스토리, accessed July 27, 2025, https://myeonguni.tistory.com/1087
15. XML Validation and Well-Formedness Check - Oxygen XML Editor, accessed July 27, 2025, https://www.oxygenxml.com/validation.html
16. Is there any difference between 'valid xml' and 'well formed xml'? - Stack Overflow, accessed July 27, 2025, https://stackoverflow.com/questions/134494/is-there-any-difference-between-valid-xml-and-well-formed-xml
17. What's the difference between "not well-formed" XML and "invalid" XML? - Stack Overflow, accessed July 27, 2025, https://stackoverflow.com/questions/16324243/whats-the-difference-between-not-well-formed-xml-and-invalid-xml
18. Valid and well-formed XML documents - - Users Guide, accessed July 27, 2025, https://docs.appeon.com/pb2025/pbug/ch06s12s01s01.html
19. XML vs JSON: A Comprehensive Comparison of Differences - Apidog, accessed July 27, 2025, https://apidog.com/articles/xml-vs-json/
20. [Web] XML과 JSON의 특징과 비교, accessed July 27, 2025, [https://velog.io/@falling_star3/Web-XML%EA%B3%BC-JSON](https://velog.io/@falling_star3/Web-XML과-JSON)
21. JSON vs XML: which one is faster and more efficient? - Imaginary Cloud, accessed July 27, 2025, https://www.imaginarycloud.com/blog/json-vs-xml
22. XSLT 2.0, XPath 2.0 및 XQuery 1.0의 주요 새 기능 - IBM, accessed July 27, 2025, https://www.ibm.com/docs/ko/SSAW57_8.5.5/com.ibm.websphere.nd.doc/ae/cins_xml_new_funcs.html
23. XPath의 이해 (초급편) - 이뫼장의 유익한 까망화면, accessed July 27, 2025, https://hahahax5.tistory.com/2
24. XPath 1.0 Tutorial - ZVON.org, accessed July 27, 2025, http://zvon.org/comp/r/tut-XPath_1.html
25. XPath, XQuery, XSLT - Stanford InfoLab, accessed July 27, 2025, http://infolab.stanford.edu/~ullman/fcdb/aut07/slides/xpath-xquery-xslt.pdf
26. XML - XQUERY - History making BLOG - 티스토리, accessed July 27, 2025, https://lawsnland.tistory.com/133
27. How to use XPath in Selenium? (using Text, Attributes, Logical Operators) - BrowserStack, accessed July 27, 2025, https://www.browserstack.com/guide/xpath-in-selenium
28. Introduction to XPath - GeeksforGeeks, accessed July 27, 2025, https://www.geeksforgeeks.org/javascript/introduction-to-xpath/
29. How to Use XPath in Selenium - A Complete Guide for Beginners - HeadSpin, accessed July 27, 2025, https://www.headspin.io/blog/using-xpath-in-selenium-effectively
30. How To Use XPath in Selenium: Complete Guide With Examples | LambdaTest, accessed July 27, 2025, https://www.lambdatest.com/blog/complete-guide-for-using-xpath-in-selenium-with-examples/
31. Introduction to XSLT - Digital humanities, accessed July 27, 2025, http://dh.obdurodon.org/xslt-basics.xhtml
32. XSLT Tutorial - Basics - EduTech Wiki, accessed July 27, 2025, https://edutechwiki.unige.ch/en/XSLT_Tutorial_-_Basics
33. XSLT Tutorial, accessed July 27, 2025, https://www.cs.ox.ac.uk/dan.olteanu/tutorials/xslt1.pdf
34. Hello, World! (XSLT) - Learn Microsoft, accessed July 27, 2025, https://learn.microsoft.com/en-us/previous-versions/windows/desktop/ms765388(v=vs.85)
35. Learn XQuery in 10 Minutes - Stylus Studio, accessed July 27, 2025, http://www.stylusstudio.com/whitepapers/Learn_XQuery_10.pdf
36. XQuery Quick Guide - Tutorialspoint, accessed July 27, 2025, https://www.tutorialspoint.com/xquery/xquery_quick_guide.htm
37. 13. XML과 JSON도 잘 쓰자 - velog, accessed July 27, 2025, [https://velog.io/@jsj3282/13.-XML%EA%B3%BC-JSON%EB%8F%84-%EC%9E%98-%EC%93%B0%EC%9E%90](https://velog.io/@jsj3282/13.-XML과-JSON도-잘-쓰자)
38. XML 파싱 시 DOM방식과 SAX방식 비교, accessed July 27, 2025, https://jayceepark.github.io/posts/2018-07-05/xml-parsing-dom-sax
39. 34일차 공부 XML Parsing, DOM Parser - MyITis - 티스토리, accessed July 27, 2025, https://myitis5212.tistory.com/38
40. DOM vs SAX - 알렉의 행복 산책 - 티스토리, accessed July 27, 2025, https://brainwave.tistory.com/349
41. xml, json, yaml 의 특징과 사용방법 - 프로그래밍 스터디, accessed July 27, 2025, [https://hobbylife.tistory.com/entry/xml-json-yaml-%EC%9D%98-%ED%8A%B9%EC%A7%95%EA%B3%BC-%EC%82%AC%EC%9A%A9%EB%B0%A9%EB%B2%95](https://hobbylife.tistory.com/entry/xml-json-yaml-의-특징과-사용방법)
42. XML, JSON 비교(알고쓰자) - SMG 프로그래머 - 티스토리, accessed July 27, 2025, https://smg7.tistory.com/103
43. JSON vs XML - Difference Between Data Representations - AWS, accessed July 27, 2025, https://aws.amazon.com/compare/the-difference-between-json-xml/
44. JSON Vs. XML for Web APIs: The Format Showdown | Zuplo Blog, accessed July 27, 2025, https://zuplo.com/blog/2025/04/30/json-vs-xml-for-web-apis
45. XML vs. JSON: A Security Perspective | by David Petty, accessed July 27, 2025, https://blog.securityevaluators.com/xml-vs-json-security-risks-22e5320cf529
46. [간단정리] {JSON},
47. Jetpack Compose vs XML: Deep Dive into Modern Android UI Development - Medium, accessed July 27, 2025, https://medium.com/@ipeksac.dogus.19/jetpack-compose-vs-xml-deep-dive-into-modern-android-ui-development-a1b2f7e2c3a7
48. The Future of XML - Why It Still Matters in 2025 and Beyond - MoldStud, accessed July 27, 2025, https://moldstud.com/articles/p-the-future-of-xml-why-it-still-matters-in-2025-and-beyond
49. What is an SVG File? – Pros, Cons, XML Code - Aspose Documentation, accessed July 27, 2025, https://docs.aspose.com/svg/net/what-is-an-svg-document/
50. Vector drawables overview | Views - Android Developers, accessed July 27, 2025, https://developer.android.com/develop/ui/views/graphics/vector-drawable-resources
51. Converting SVG file to Android Vector Drawable XML while keeping the group structure in place - Stack Overflow, accessed July 27, 2025, https://stackoverflow.com/questions/46456867/converting-svg-file-to-android-vector-drawable-xml-while-keeping-the-group-struc
52. The Case Against OOXML - NoOOXML, accessed July 27, 2025, http://noooxml.wdfiles.com/local--files/arguments/TheCaseAgainstOOXML.pdf
53. Best XML Converters by Use Case (2025 Guide) - Sonra, accessed July 27, 2025, https://sonra.io/xml-converters-by-use-case-bidirectional/
54. After May 29, 2025, new buyback announcement and results XML files on TreasuryDirect will have the following format, accessed July 27, 2025, https://treasurydirect.gov/instit/annceresult/press/preanre/2025/Buyback-XML-Changes.pdf
55. Tax year 2025 valid XML schemas and business rules for Form 2290 Modernized e-File (MeF) | Internal Revenue Service, accessed July 27, 2025, https://www.irs.gov/tax-professionals/tax-year-2025-valid-xml-schemas-and-business-rules-for-form-2290-modernized-e-file-mef
56. XML Extract | Grants.gov, accessed July 27, 2025, https://www.grants.gov/xml-extract
57. COD Common Record XML Schema Version 5.0c Now Available | Knowledge Center, accessed July 27, 2025, https://fsapartners.ed.gov/knowledge-center/library/electronic-announcements/2024-10-25/cod-common-record-xml-schema-version-50c-now-available
58. The Future of XML - Why It Remains Relevant in 2023 and Beyond - MoldStud, accessed July 27, 2025, https://moldstud.com/articles/p-the-future-of-xml-why-it-remains-relevant-in-2023-and-beyond

