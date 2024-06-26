[GraphQL](https://graphql.org/learn/)

# Caching

> 객체 식별자를 제공하면 클라이언트가 풍부한 캐시를 구축할 수 있습니다.

엔드포인트 기반 API에서 클라이언트는 HTTP 캐싱을 사용하여 리소스 리페칭을 쉽게 피하고, 두 리소스가 동일한 경우 식별할 수 있습니다. 이러한 API의 URL은 클라이언트가 캐시를 구축하는 데 활용할 수 있는 **글로벌 고유 식별자**입니다. 하지만 GraphQL에는 주어진 객체에 대해 이 전역 고유 식별자를 제공하는 URL과 유사한 프리미티브가 없습니다. 따라서 클라이언트가 사용할 수 있도록 API에서 이러한 식별자를 노출하는 것이 가장 좋은 방법입니다.

## Globally Unique IDs

이를 위한 한 가지 가능한 패턴은 `id`와 같은 필드를 전역적으로 고유한 식별자로 예약하는 것입니다. 이 문서 전체에 사용된 예제 스키마는 이 접근 방식을 사용합니다:

```
{
  starship(id: "3003") {
    id
    name
  }
  droid(id: "2001") {
    id
    name
    friends {
      id
      name
    }
  }
}
```

```json
{
  "data": {
    "starship": {
      "id": "3003",
      "name": "Imperial shuttle"
    },
    "droid": {
      "id": "2001",
      "name": "R2-D2",
      "friends": [
        {
          "id": "1000",
          "name": "Luke Skywalker"
        },
        {
          "id": "1002",
          "name": "Han Solo"
        },
        {
          "id": "1003",
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

이것은 클라이언트 개발자에게 전달할 수 있는 강력한 도구입니다. 리소스 기반 API의 URL이 전역적으로 고유한 키를 제공하는 것과 마찬가지로 이 시스템의 `id` 필드는 전역적으로 고유한 키를 제공합니다.

백엔드에서 식별자로 UUID와 같은 것을 사용하는 경우 이 전역 고유 ID를 노출하는 것은 매우 간단할 수 있습니다! 백엔드에 모든 객체에 대한 전역 고유 ID가 아  직 없는 경우 GraphQL 계층에서 이를 구성해야 할 수 있습니다. 대개는 ID에 유형 이름을 추가하고 이를 식별자로 사용하는 것만큼 간단합니다. 그런 다음 서버는 해당 ID를 base64 인코딩하여 불투명하게 만들 수 있습니다.

선택적으로, 이 ID를 사용하여 [글로벌 객체 식별](https://graphql.org/learn/global-object-identification)의 `node` 패턴으로 작업할 수 있습니다.

## Compatibility with existing APIs

이러한 목적으로 `id` 필드를 사용할 때 한 가지 우려되는 점은 GraphQL API를 사용하는 클라이언트가 기존 API와 어떻게 작동할 것인가 하는 점입니다. 예를 들어 기존 API는 유형별 ID를 허용했지만 GraphQL API는 전 세계적으로 고유한 ID를 사용하는 경우 두 가지를 한꺼번에 사용하는 것이 까다로울 수 있습니다.

이러한 경우 GraphQL API는 이전 API의 ID를 별도의 필드에 노출할 수 있습니다. 이렇게 하면 두 가지 장점을 모두 누릴 수 있습니다:

- GraphQL 클라이언트는 전 세계적으로 고유한 ID를 얻기 위한 일관된 메커니즘에 계속 의존할 수 있습니다.
- 이전 API로 작업해야 하는 클라이언트도 객체에서 `previousApiId`를 가져와서 사용할 수 있습니다.

## Alternatives

글로벌 고유 ID는 과거에 강력한 패턴으로 입증되었지만, 이 패턴이 유일한 패턴은 아니며 모든 상황에 적합한 것도 아닙니다. 클라이언트가 필요로 하는 정말 중요한 기능은 캐싱을 위해 전역적으로 고유한 식별자를 도출하는 기능입니다. 서버가 식별자를 도출하면 클라이언트의 작업이 간소화되지만, 클라이언트도 식별자를 도출할 수 있습니다. 대개는 객체의 유형(`__typename`으로 쿼리)과 유형 고유 식별자를 결합하는 것만큼 간단합니다.

또한 기존 API를 GraphQL API로 대체하는 경우, GraphQL의 모든 필드가 동일하지만 전역적으로 고유하게 변경된 `id`를 제외하면 혼동될 수 있습니다. 이는 `id`를 전역 고유 필드로 사용하지 않는 또 다른 이유가 될 수 있습니다.