[GraphQL](https://graphql.org/learn/)

# Thinking in Graphs

## It's Graphs All the Way Down [*](https://en.wikipedia.org/wiki/Turtles_all_the_way_down)

> GraphQL을 사용하면 비즈니스 도메인을 그래프로 모델링할 수 있습니다.

그래프(graphs)는 우리의 자연스러운 정신 모델(natural mental models)과 기저 프로세스(underlying process)에 대한 언어적 설명(verbal descriptions)과 유사하기 때문에 많은 실제 현상(real-world phenomena)을 모델링하는 데 강력한 도구입니다. GraphQL을 사용하면 스키마를 정의하여 비즈니스 도메인을 그래프로 모델링하고, 스키마 내에서 다양한 유형의 노드와 노드가 서로 연결/관련되는 방식을 정의할 수 있습니다. 클라이언트에서는 객체 지향 프로그래밍과 유사한 패턴, 즉 다른 유형을 참조하는 유형이 생성됩니다. 서버에서는 GraphQL이 인터페이스만 정의하기 때문에 모든 백엔드(신규 또는 레거시!)에서 자유롭게 사용할 수 있습니다.

## Shared Language

> 명명(naming)은 직관적인 API를 구축하는 데 있어 어렵지만 중요한 부분입니다.

GraphQL 스키마는 팀과 사용자를 위한 표현력 있는 공유 언어라고 생각하세요. 좋은 스키마를 구축하려면 비즈니스를 설명하는 데 사용하는 일상적인 언어를 살펴보세요. 예를 들어 이메일 앱을 평이한 영어로 설명해 보겠습니다:

- 사용자는 여러 개의 이메일 계정을 가질 수 있습니다.
- 각 이메일 계정에는 주소, 받은 편지함, 임시 보관함, 삭제된 항목, 보낸 항목이 있습니다.
- 각 이메일에는 발신자, 수신 날짜, 제목 및 본문이 있습니다.
- 사용자는 수신자 주소가 없으면 이메일을 보낼 수 없습니다.

이름을 짓는 것은 직관적인 API를 구축하는 데 있어 어렵지만 중요한 부분이므로 문제 도메인과 사용자에게 적합한 이름이 무엇인지 신중하게 생각해 보세요. GraphQL 스키마에서 노드와 관계에 대해 직관적이고 내구성 있는 이름을 선택해야 하므로 팀원들이 이러한 비즈니스 도메인 규칙에 대한 공유된 이해와 합의를 도출해야 합니다. 실행하고자 하는 몇 가지 쿼리를 상상해 보세요:

내 모든 계정의 받은 편지함에서 읽지 않은 이메일 수 가져오기:

```
{
  accounts {
    inbox {
      unreadEmailCount
    }
  }
}
```

메인 계정에서 처음 20개의 초안에 대한 '미리 보기 정보'를 가져옵니다.

```
{
  mainAccount {
    drafts(first: 20) {
      ...previewInfo
    }
  }
}

fragment previewInfo on Email {
  subject
  bodyPreviewSentence
}
```

## 비즈니스 로직 계층

> 비즈니스 로직 계층은 비즈니스 도메인 규칙을 적용하기 위한 단일 소스 역할을 해야 합니다.

실제 비즈니스 로직을 어디에 정의해야 할까요? 유효성 검사 및 권한 확인은 어디에서 수행해야 할까요? 정답은 전용 비즈니스 로직 레이어 내부입니다. 비즈니스 로직 계층은 비즈니스 도메인 규칙을 적용하기 위한 신뢰할 수 있는 단일 소스 역할을 해야 합니다.

![Business Logic Layer Diagram](https://graphql.org/img/diagrams/business_layer.png)

위의 다이어그램에서 시스템에 대한 모든 진입점(REST, GraphQL 및 RPC)은 동일한 유효성 검사, 권한 부여 및 오류 처리 규칙으로 처리됩니다.

### 레거시 데이터로 작업하기

> 레거시 데이터베이스 스키마를 미러링하는 대신 클라이언트가 데이터를 사용하는 방법을 설명하는 GraphQL 스키마를 구축하는 것을 선호합니다.

클라이언트가 데이터를 사용하는 방식을 완벽하게 반영하지 않는 레거시 데이터 소스로 작업하는 경우가 있습니다. 이러한 경우에는 레거시 데이터베이스 스키마를 미러링하는 대신 클라이언트가 데이터를 사용하는 방법을 설명하는 GraphQL 스키마를 구축하는 것이 좋습니다.

'무엇'이 아닌 '어떻게'를 표현하도록 GraphQL 스키마를 구축하세요. 그러면 기존 클라이언트와의 인터페이스를 손상시키지 않고 구현 세부 사항을 개선할 수 있습니다.

## 한 번에 한 단계씩

> 유효성 검사 및 피드백을 더 자주 받기

전체 비즈니스 도메인을 한 번에 모델링하려고 하지 마세요. 그보다는 한 번에 하나의 시나리오에 필요한 스키마 부분만 구축하세요. 스키마를 점진적으로 확장하면 검증과 피드백을 더 자주 받아 올바른 솔루션을 구축하는 데 도움이 됩니다.

---

[Serving over HTTP](008_serving_over_http.md)