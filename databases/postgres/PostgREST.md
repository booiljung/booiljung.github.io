---
layout: page
title: PostgREST
permalink: /databases/postgres/PostgREST
---


현대의 애플리케이션 개발에서 백엔드 API를 구축하는 것은 필수적인 과정이지만, 이 과정은 종종 반복적인 CRUD(Create, Read, Update, Delete) 로직의 구현, 데이터베이스 스키마와 애플리케이션 비즈니스 로직 간의 불일치, 그리고 객체-관계 매핑(ORM)의 추상화 누수(leaky abstraction)로 인한 성능 문제 등 다양한 도전에 직면한다.1 PostgREST는 이러한 문제들에 대한 근본적인 해결책을 제시하며, 단순히 새로운 도구를 넘어 백엔드 아키텍처에 대한 패러다임 전환을 제안하는 독립형 웹 서버이다.1 Haskell로 작성된 이 서버는 PostgreSQL 데이터베이스를 직접 RESTful API로 변환하는 기능을 수행한다.3


전통적인 다층(multi-tier) 아키텍처에서는 데이터 유효성 검사, 접근 제어, 비즈니스 규칙과 같은 핵심 로직이 애플리케이션 서버 계층에 구현되는 경우가 일반적이다. 이로 인해 데이터베이스에 이미 정의된 제약 조건(예: `NOT NULL`, `CHECK`, `FOREIGN KEY`)들이 애플리케이션 코드에서 중복으로 구현되는 비효율이 발생한다.2 예를 들어, 데이터베이스 테이블의 특정 컬럼에 `NOT NULL` 제약이 있음에도 불구하고, 애플리케이션 코드에서 해당 필드에 대한 null 값 검사를 별도로 수행하는 식이다. 이러한 구조는 데이터베이스와 애플리케이션 간의 일관성을 유지하기 어렵게 만들고, 유지보수 비용을 증가시킨다.

PostgREST의 핵심 철학은 이러한 중복을 제거하고, 데이터베이스 자체를 유일하고 선언적인 '신뢰의 원천(Single Source of Truth)'으로 확립하는 것이다.1 이 패러다임 하에서 데이터베이스는 더 이상 수동적인 데이터 저장소에 머무르지 않고, API의 명세(specification)와 비즈니스 규칙을 정의하는 능동적인 주체가 된다. PostgreSQL이 제공하는 구조적 제약 조건과 권한 시스템이 API의 엔드포인트와 작업을 직접 결정한다.1

`CHECK` 제약 조건은 데이터 유효성 검사 규칙이 되고, `FOREIGN KEY`는 리소스 간의 관계를 정의하며, PostgreSQL의 `ROLE` 시스템은 API의 인증 및 권한 부여 체계의 근간을 이룬다.1

이러한 접근법은 '애플리케이션이 데이터를 소유하고 제어한다'는 전통적 관점에서 '데이터베이스가 비즈니스 규칙을 정의하고, 애플리케이션은 이를 소비한다'는 관점으로의 근본적인 전환을 의미한다. 결과적으로 아키텍처는 극도로 단순화되며, 개발 및 유지보수 오버헤드는 현저히 감소한다.7 데이터베이스 스키마가 곧 API 문서가 되는 강력한 일관성을 확보할 수 있다.


PostgREST는 수동 CRUD 프로그래밍의 명령형(imperative) 접근 방식에 대한 대안으로 선언적(declarative) 패러다임을 채택한다.1 명령형 프로그래밍이 작업의 '방법(how)'을 단계별로 명시하는 반면, 선언적 프로그래밍은 '무엇(what)'을 원하는지에 집중하고 실제 실행은 하부 시스템에 위임한다.

예를 들어, 수동으로 구현된 백엔드에서 `GET /users/{id}/posts`와 같은 요청을 처리하기 위해서는, 먼저 `users` 테이블에서 특정 사용자를 조회하고, 그 결과로 얻은 `user_id`를 사용하여 `posts` 테이블에서 해당 사용자의 게시글들을 조회하는 두 번 이상의 데이터베이스 쿼리를 순차적으로 실행한 뒤, 애플리케이션 메모리에서 이 두 데이터셋을 조합하는 명령형 로직이 필요하다.

반면, PostgREST에서는 리소스 임베딩(Resource Embedding) 기능을 통해 `GET /users?id=eq.{id}&select=*,posts(*)`와 같은 단일의 선언적 HTTP 요청으로 동일한 결과를 얻을 수 있다. 이 요청은 "ID가 {id}인 사용자의 모든 필드와, 그와 관련된 모든 게시글을 함께 반환하라"는 목표를 선언할 뿐, 데이터를 어떻게 조인하고 조합할지에 대한 절차는 명시하지 않는다. PostgREST는 이 요청을 해석하여 PostgreSQL의 고도로 최적화된 쿼리 플래너가 가장 효율적인 실행 계획을 수립하고 실행하도록 단일 SQL 쿼리를 생성하여 위임한다.1

이러한 선언적 접근은 권한 관리와 데이터 유효성 검사에도 동일하게 적용된다. 애플리케이션 컨트롤러에 복잡한 조건문으로 접근 제어 로직(guard)을 추가하는 대신, 데이터베이스 객체에 `GRANT` 문으로 권한을 선언적으로 할당하는 것이 더 단순하고 강력하다.1 마찬가지로, 코드 곳곳에 유효성 검사 로직을 흩어놓는 대신, 테이블에 `CHECK` 제약 조건을 선언하는 것이 데이터 무결성을 보장하는 더 견고한 방법이다. PostgREST를 채택하는 것은 단순히 코딩 스타일을 바꾸는 것을 넘어, 문제 해결 방식을 근본적으로 전환하여 개발자가 비즈니스 로직의 본질에 더 집중할 수 있도록 돕는다.


객체-관계 매핑(ORM) 기술은 객체 지향 프로그래밍 언어와 관계형 데이터베이스 사이의 패러다임 불일치, 이른바 '임피던스 불일치(impedance mismatch)' 문제를 해결하기 위해 등장했다. 그러나 ORM은 종종 '누수 있는 추상화(leaky abstraction)'로 비판받으며, 개발자가 데이터베이스의 실제 동작을 이해하지 못하게 만들고 N+1 쿼리 문제와 같이 비효율적인 SQL을 생성하여 성능 저하를 유발하는 원인이 되기도 한다.2

PostgREST는 의도적으로 ORM을 배제함으로써 '누수 없는 추상화'를 지향한다.1 이는 개발자와 데이터베이스 사이에 불필요한 추상화 계층을 두지 않고, HTTP 요청을 가능한 한 직접적으로 SQL 쿼리로 변환함을 의미한다.8 API 요청의 성능은 PostgreSQL 쿼리의 성능과 거의 일대일로 대응하게 되며, 이는 API 성능에 대한 예측 가능성을 극적으로 높인다.

새로운 데이터 뷰(view)를 생성하는 작업은 성능적 함의가 명확한 SQL 문으로 직접 이루어진다. 개발자는 `EXPLAIN ANALYZE`와 같은 PostgreSQL의 강력한 내장 도구를 사용하여 API 엔드포인트의 성능을 직접 분석하고, 인덱스 추가나 쿼리 재구성 등의 최적화를 수행할 수 있다.1 이러한 투명성은 ORM이 숨기는 복잡한 쿼리 생성 과정과 그로 인한 성능 저하 문제로부터 개발자를 해방시킨다. API와 데이터베이스 간의 성능 관련 피드백 루프가 단축되고, 문제 해결 과정이 단순화된다. 결과적으로 데이터베이스 관리자(DBA)나 SQL에 능숙한 개발자는 별도의 백엔드 프로그래밍 언어에 대한 깊은 지식 없이도 고성능 API를 설계하고 구축할 수 있게 된다.1


PostgREST는 PostgreSQL 데이터베이스 스키마를 기반으로 동적으로 RESTful API를 생성하는 정교한 메커니즘을 갖추고 있다. 이 과정의 핵심에는 효율적인 내부 아키텍처, 스키마 인트로스펙션, 그리고 데이터베이스 객체를 API 리소스로 변환하는 명확한 규칙이 자리 잡고 있다.


PostgREST는 높은 성능과 안정성을 목표로 설계되었으며, 이를 위해 기술 스택으로 Haskell 프로그래밍 언어와 고성능 비동기 웹 서버인 Warp을 채택했다.4 컴파일된 언어인 Haskell은 런타임 효율성을 보장하며, Warp 서버는 경량 스레드(lightweight threads)를 통해 수천 개의 동시 연결을 효율적으로 처리할 수 있는 기반을 제공한다.

HTTP 요청이 PostgREST 서버에 도달하면, 이는 여러 내부 모듈을 거치는 체계적인 파이프라인을 통해 처리된다. 각 모듈은 '관심사의 분리(Separation of Concerns)' 원칙에 따라 명확하게 정의된 단일 책임을 수행하며, 이는 시스템 전체의 안정성과 유지보수성을 높이는 데 기여한다.9

요청 처리 흐름은 다음과 같은 주요 단계로 구성된다:

1. **Main/CLI**: 프로그램의 진입점(entry point)으로, 서버를 시작하고 명령줄 인터페이스(CLI)를 통해 전달된 설정 파일 경로와 같은 인자를 처리한다.9
2. **App**: PostgREST 애플리케이션의 전체 구조를 구성하는 역할을 한다. 다른 핵심 모듈들을 조합하여 완전한 웹 애플리케이션 미들웨어 스택을 형성한다.9
3. **Auth**: 인증(Authentication)을 전담하는 모듈이다. HTTP 요청의 `Authorization` 헤더를 검사하여 JWT(JSON Web Token)의 유효성을 검증한다. 토큰의 서명, 만료 시간 등을 확인하고, 유효한 경우 페이로드에서 `role`과 같은 핵심 클레임을 추출한다.9
4. **ApiRequest**: HTTP 요청의 구문 분석을 담당한다. PostgREST 고유의 URL 쿼리 문자열(예: `age=gt.18`, `select=*,posts(*)`), 요청 헤더(예: `Accept`, `Prefer`), 그리고 요청 본문(body)을 파싱하여 내부적으로 사용 가능한 구조로 변환한다. 지원되지 않는 미디어 타입이나 HTTP 메서드와 같은 유효하지 않은 요청은 이 초기 단계에서 즉시 거부된다.9
5. **Plan**: 파싱된 요청과 인증 정보를 바탕으로, 실제 SQL 쿼리를 생성하기 위한 실행 계획을 수립한다. 이 모듈은 스키마 캐시를 참조하여 요청된 리소스(테이블, 뷰)의 존재 여부, 컬럼 정보, 그리고 리소스 간의 관계(외래 키)를 확인한다. 예를 들어, 리소스 임베딩 요청 시 존재하지 않는 관계를 참조하면 이 단계에서 요청이 거부된다. 즉, 구문적으로는 유효하지만 논리적으로 유효하지 않은 요청을 걸러내는 역할을 한다.9
6. **Query**: `Plan` 모듈에서 수립된 계획을 바탕으로, 최종적으로 PostgreSQL에서 실행될 SQL 쿼리를 생성한다. 이 과정에서 모든 사용자 입력은 매개변수화(parameterization)되어 SQL 인젝션 공격으로부터 안전한 쿼리가 만들어진다. 이 단계에서 비로소 데이터베이스 커넥션 풀에서 유휴 연결(idle connection)을 획득하여 쿼리를 실행하고 결과를 반환받는다.9

이러한 단계적 처리 방식은 복잡한 HTTP 요청을 체계적으로 SQL로 변환하며, 각 단계에서 오류를 조기에 발견하고 명확한 피드백과 함께 요청을 거부할 수 있게 하여 시스템의 전반적인 견고함을 보장한다.


PostgREST의 가장 큰 특징 중 하나는 API를 정의하기 위한 별도의 설정 파일이나 코드 작성이 필요 없다는 점이다. 대신, 서버가 시작될 때 `db-schemas` 설정에 지정된 데이터베이스 스키마를 동적으로 분석하는 '스키마 인트로스펙션' 과정을 거친다.1 이 과정에서 PostgREST는 `pg_catalog`과 같은 PostgreSQL의 시스템 카탈로그를 쿼리하여 테이블, 뷰, 컬럼, 데이터 타입, 제약 조건, 외래 키 관계, 함수, 그리고 역할과 권한에 대한 모든 메타데이터를 수집한다.

수집된 메타데이터는 인메모리 '스키마 캐시(Schema Cache)'에 저장된다.2 이 캐시는 후속 API 요청을 처리할 때 데이터베이스에 메타데이터를 다시 쿼리할 필요 없이, 메모리상에서 빠르게 테이블 구조나 관계를 조회할 수 있게 해준다. 스키마 캐시는 PostgREST의 높은 성능을 유지하는 데 핵심적인 역할을 한다. 만약 캐시가 없다면, 모든 API 요청마다 시스템 카탈로그를 조회하는 오버헤드가 발생하여 성능이 크게 저하될 것이다.

그러나 이 캐싱 메커니즘은 운영상 중요한 고려사항을 동반한다. 개발자가 데이터베이스 스키마를 변경했을 때(예: `CREATE TABLE`이나 `ALTER FUNCTION` 실행), 실행 중인 PostgREST 인스턴스는 이 변경을 자동으로 인지하지 못한다. 변경된 스키마를 API에 반영하기 위해서는 스키마 캐시를 수동으로 새로고침해야 한다. PostgREST는 이를 위해 세 가지 방법을 제공한다 13:

1. **서버 재시작**: 가장 간단한 방법이지만, 서비스 중단을 유발할 수 있다.
2. **`SIGUSR2` 시그널 전송**: PostgREST 프로세스에 `SIGUSR2` 시그널을 보내면, 서비스 중단 없이 스키마 캐시를 다시 로드한다. (예: `killall -SIGUSR2 postgrest`)
3. **`NOTIFY` 채널**: 데이터베이스 내에서 `NOTIFY pgrst, 'reload schema'`와 같은 명령을 실행하면, PostgREST의 리스너가 이를 감지하여 캐시를 새로고침한다.

따라서, 데이터베이스 마이그레이션을 자동화하는 CI/CD 파이프라인을 구축할 때는 마이그레이션 스크립트 실행 후 PostgREST 스키마 캐시를 새로고침하는 단계를 반드시 포함해야 한다. 이는 프로덕션 환경에서 PostgREST를 안정적으로 운영하기 위한 필수적인 DevOps 절차이다.


스키마 인트로스펙션이 완료되면, PostgREST는 명확하고 일관된 규칙에 따라 데이터베이스 객체를 HTTP API 리소스로 매핑한다. 이 자동 매핑 과정은 '규약을 통한 설정(Convention over Configuration)' 원칙을 따르므로, 개발자는 라우팅 코드를 작성할 필요 없이 표준 SQL DDL(Data Definition Language)을 통해 API를 설계하게 된다.7

매핑 규칙은 다음과 같다:

- **테이블과 뷰**: `db-schemas` 설정에 포함된 스키마 내의 모든 테이블과 뷰는 자동으로 RESTful 리소스로 노출된다. 테이블 또는 뷰의 이름이 엔드포인트의 경로가 된다. 예를 들어, `api` 스키마에 `todos`라는 테이블이 있다면, 이는 `http://localhost:3000/todos`라는 엔드포인트로 매핑된다.7
- **함수 (저장 프로시저)**: 스키마 내의 함수들은 원격 프로시저 호출(RPC)을 위한 엔드포인트로 노출된다. 테이블/뷰와의 경로 충돌을 피하기 위해 `/rpc/`라는 고정된 접두사가 사용된다. 예를 들어, `api` 스키마의 `add_them(a int, b int)` 함수는 `http://localhost:3000/rpc/add_them` 엔드포인트로 매핑된다.12
- **HTTP 메서드와 SQL 권한**: 클라이언트가 사용하는 HTTP 메서드는 데이터베이스 역할(role)에 부여된 SQL 권한과 직접적으로 연결된다.
  - `GET`, `HEAD`: `SELECT` 권한이 필요하다.
  - `POST`: `INSERT` 권한이 필요하다.
  - `PATCH`, `PUT`: `UPDATE` 권한이 필요하다.
  - `DELETE`: `DELETE` 권한이 필요하다.
  - `OPTIONS`: 권한 없이 항상 허용되며, 해당 엔드포인트에서 사용 가능한 메서드 목록을 반환한다.7

예를 들어, 개발자가 `CREATE TABLE api.users (...)`를 실행하고 스키마 캐시를 새로고침하면, PostgREST는 즉시 `/users` 엔드포인트를 생성한다. 이어서 `GRANT SELECT ON api.users TO web_anon;` 명령을 실행하면, `web_anon` 역할로 요청하는 익명 사용자는 `GET /users`를 통해 데이터를 조회할 수 있게 된다. 하지만 `INSERT` 권한이 없으므로 `POST /users` 요청은 `401 Unauthorized` 또는 `403 Forbidden` 오류를 반환하게 된다. 이처럼 API의 구조, 동작, 그리고 접근 제어가 전적으로 데이터베이스 스키마와 권한 설정에 의해 결정된다. 이는 API 개발 과정을 데이터베이스 설계 과정과 긴밀하게 통합시켜, 개발 속도를 비약적으로 향상시키는 강력한 추상화를 제공한다.


PostgREST의 보안 모델은 그 철학의 핵심을 이루며, 모든 보안 관련 결정을 데이터베이스에 위임함으로써 강력하고 일관된 보안 체계를 구축한다. 이 모델은 인증(Authentication)과 권한 부여(Authorization)라는 두 가지 개념을 명확히 분리하여 각각의 책임을 PostgREST 서버와 PostgreSQL 데이터베이스에 할당한다.


PostgREST 보안 모델의 근간은 인증과 권한 부여의 명확한 분리이다.19 이 두 개념은 종종 혼용되지만, PostgREST는 이들을 엄격하게 구분하여 처리한다.

- **인증 (Authentication)**: "클라이언트가 자신이 누구라고 주장하는지를 검증하는 과정"이다. 이는 PostgREST 서버의 핵심 책임 중 하나이다. PostgREST는 주로 상태 비저장(stateless) 방식인 JWT(JSON Web Token)를 사용하여 이 과정을 처리한다. 클라이언트가 제공한 JWT가 유효한지(서명이 올바른지, 만료되지 않았는지 등)를 확인하여 신원을 확인한다.6
- **권한 부여 (Authorization)**: "인증된 클라이언트가 특정 리소스에 대해 어떤 작업을 수행할 수 있는지를 결정하는 과정"이다. 이 책임은 전적으로 PostgreSQL 데이터베이스에 위임된다. PostgREST는 인증 과정에서 확인된 사용자의 신원(주로 데이터베이스 역할)을 데이터베이스에 전달할 뿐, 해당 사용자가 데이터를 읽거나 쓸 수 있는지에 대한 판단은 내리지 않는다. 모든 권한 부여 결정은 PostgreSQL의 역할(Role) 시스템, 권한(Privileges), 그리고 행 수준 보안(Row-Level Security, RLS) 정책에 의해 이루어진다.6

이러한 책임의 분리는 보안 로직의 '신뢰의 단일 원천'을 데이터베이스로 강제하는 효과를 낳는다. 전통적인 백엔드 아키텍처에서는 권한 검사 로직이 애플리케이션 코드의 여러 계층(컨트롤러, 서비스, 미들웨어 등)에 흩어져 존재할 수 있어, 일관성을 유지하기 어렵고 보안 허점을 만들 가능성이 높다. 반면, PostgREST 모델에서는 데이터베이스 관리자가 모든 데이터 접근 정책을 SQL을 통해 중앙에서 선언적으로 관리할 수 있다. 이는 보안 정책의 가시성을 높이고, 감사(auditing)를 용이하게 하며, 시스템 전체의 보안 일관성을 보장하는 강력한 장점이 된다.


PostgREST는 상태 비저장 인증을 위해 표준 기술인 JWT를 채택했다. 클라이언트는 인증이 필요한 모든 요청에 `Authorization: Bearer <jwt>` HTTP 헤더를 포함하여 JWT를 서버에 전송해야 한다.21

PostgREST 서버는 요청을 받으면 `postgrest.conf` 설정 파일에 명시된 `jwt-secret` 값을 사용하여 토큰의 서명을 검증한다. 이 시크릿은 대칭키 암호화 방식(HMAC-SHA256)을 위한 공유 비밀번호일 수도 있고, 비대칭키 암호화 방식(RSA, ECDSA 등)을 위한 JWK(JSON Web Key) 또는 JWKS(JSON Web Key Set) 형식의 공개키일 수도 있다.20

JWT의 페이로드(payload)에는 여러 클레임(claim)이 포함될 수 있으며, PostgREST는 그중 몇 가지를 특별하게 해석한다:

- **`role` (필수)**: 이 클레임은 PostgREST가 해당 요청을 처리하는 동안 데이터베이스 세션에서 전환(impersonate)할 PostgreSQL 역할의 이름을 지정한다. 이는 PostgREST의 권한 부여 메커니즘에서 가장 핵심적인 역할을 한다.21
- **`exp` (만료 시간)**: 토큰이 만료되는 시간을 나타내는 유닉스 타임스탬프(Unix epoch time)이다. PostgREST는 만료된 토큰이 포함된 요청을 자동으로 거부하여 시간 기반의 보안을 강제한다.23
- **기타 커스텀 클레임**: `user_id`, `email`, `tenant_id` 등 애플리케이션에 특화된 추가 정보를 클레임으로 포함할 수 있다. 이러한 커스텀 클레임은 데이터베이스 함수나 행 수준 보안 정책 내에서 `current_setting('request.jwt.claims', true)::json->>'email'`과 같은 구문을 통해 접근할 수 있어, 매우 유연하고 정교한 권한 부여 로직을 구현하는 데 사용될 수 있다.22

JWT의 상태 비저장 특성은 PostgREST의 수평적 확장성(horizontal scalability)에 직접적으로 기여한다. 서버가 클라이언트의 세션 상태를 메모리에 유지할 필요가 없기 때문에, 어떤 PostgREST 인스턴스라도 동일한 `jwt-secret`만 공유하면 모든 요청을 독립적으로 처리할 수 있다. 이는 로드 밸런서 뒤에 여러 PostgREST 인스턴스를 배치하여 손쉽게 시스템 처리량을 늘릴 수 있음을 의미한다.6 또한, 매 요청마다 데이터베이스에 사용자 정보를 조회하는 오버헤드를 제거하여 API 응답 시간을 단축시키고 데이터베이스 부하를 줄이는 효과도 있다.


PostgREST는 PostgreSQL의 강력한 역할 시스템을 기반으로 세 가지 유형의 역할을 구분하여 사용한다.20 이 역할 시스템은 '최소 권한 원칙(Principle of Least Privilege)'을 아키텍처 수준에서 구현하는 핵심적인 메커니즘이다.

1. **`authenticator` 역할**: PostgREST 서버 프로세스가 PostgreSQL 데이터베이스에 연결할 때 사용하는 역할이다. 이 역할의 자격 증명(사용자명, 비밀번호)은 `db-uri` 설정에 명시된다.26

   `authenticator` 역할은 `LOGIN` 권한 외에는 데이터베이스 객체에 대한 어떠한 권한도 가지지 않는 것이 원칙이다. 이 역할의 유일한 특권은 다른 사용자 역할로 전환(`SET ROLE`)할 수 있는 능력이다. 이를 위해 데이터베이스 관리자는 `GRANT user_role TO authenticator;`와 같은 명령을 통해 역할 전환을 명시적으로 허용해야 한다.21 이 구조는 PostgREST 서버 프로세스 자체가 탈취되더라도 데이터에 직접 접근할 수 없도록 하여, 강력한 보안 경계를 형성한다.

2. **`anonymous` 역할**: 인증되지 않은 요청, 즉 유효한 JWT가 없는 요청을 처리할 때 사용되는 역할이다. `db-anon-role` 설정 파라미터를 통해 어떤 데이터베이스 역할을 익명 역할로 사용할지 지정한다.26 일반적으로 이 역할은 공개적으로 접근 가능한 데이터에 대한 

   `SELECT` 권한만을 가지도록 제한된다.

3. **`user` 역할**: 인증된 요청을 처리할 때 사용되는 역할이다. PostgREST는 유효한 JWT의 `role` 클레임에서 이 역할의 이름을 읽어온다. 해당 HTTP 요청의 트랜잭션 동안, 데이터베이스 세션은 `SET LOCAL ROLE 'user_role';` 명령을 통해 이 사용자 역할로 전환된다. 이후 모든 SQL 쿼리는 이 역할이 가진 권한 하에서 실행된다. 트랜잭션이 종료되면 세션은 다시 원래의 `authenticator` 역할로 돌아가므로, 각 요청은 완벽하게 격리된 권한 컨텍스트에서 실행된다.

이 3중 역할 시스템은 PostgREST가 클라이언트 요청의 대리인(proxy)으로서만 동작하도록 강제한다. 서버 자체는 최소한의 권한만을 가지며, 실제 데이터 접근 권한은 전적으로 클라이언트가 제시한 신원(JWT)에 의해 결정된다.


PostgreSQL의 행 수준 보안(Row-Level Security, RLS)은 `GRANT` 문을 통한 테이블/컬럼 단위의 권한 부여를 넘어, 개별 행(row) 단위의 매우 정교한 데이터 접근 제어를 가능하게 하는 강력한 기능이다.28 PostgREST는 이 RLS 기능을 완벽하게 지원하며, 둘의 조합은 데이터베이스 중심 보안 모델의 정점을 보여준다.

RLS는 `CREATE POLICY` 명령을 사용하여 테이블에 적용된다. 정책은 특정 SQL 명령(`SELECT`, `INSERT`, `UPDATE`, `DELETE`)과 특정 역할에 대해, 어떤 행이 보이거나 수정될 수 있는지를 결정하는 불리언(boolean) 표현식(`USING` 절)과, 새로 삽입되거나 수정될 행이 정책을 만족하는지 검사하는 표현식(`WITH CHECK` 절)으로 구성된다.28

PostgREST와 함께 RLS를 사용할 때, 이 불리언 표현식은 현재 HTTP 요청의 컨텍스트를 활용하여 동적으로 데이터를 필터링할 수 있다. 이를 위해 두 가지 주요 PostgreSQL 기능을 사용한다:

- `current_user`: 현재 세션의 역할을 반환한다. PostgREST에서는 JWT의 `role` 클레임에 의해 설정된 사용자 역할에 해당한다.
- `current_setting('request.jwt.claims', true)`: PostgREST가 현재 요청의 JWT 클레임을 JSON 형태로 저장해두는 세션 변수이다. `->>` 연산자를 사용하여 특정 클레임 값(예: `user_id`, `tenant_id`)을 추출할 수 있다.10

이 기능들을 활용하면 "사용자는 자신의 게시물만 수정할 수 있다"와 같은 복잡한 비즈니스 규칙을 애플리케이션 코드 없이 순수 SQL 정책으로 선언할 수 있다. 예를 들어, `posts` 테이블에 `author_id` 컬럼이 있고, JWT에 `user_id` 클레임이 포함되어 있다고 가정하자. 다음과 같은 RLS 정책을 생성할 수 있다:

```SQL
-- posts 테이블에 RLS 활성화
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;

-- 자신의 게시물만 수정/삭제할 수 있도록 하는 정책
CREATE POLICY posts_author_policy ON posts
FOR ALL USING (author_id = (current_setting('request.jwt.claims', true)::json->>'user_id')::uuid)
WITH CHECK (author_id = (current_setting('request.jwt.claims', true)::json->>'user_id')::uuid);
```

이제 어떤 사용자가 `PATCH /posts/123` 요청을 보내면, PostgreSQL은 `UPDATE` 쿼리를 실행하기 전에 자동으로 이 RLS 정책을 적용한다. 데이터베이스는 123번 게시물의 `author_id`가 요청 JWT의 `user_id` 클레임 값과 일치하는지 확인하고, 일치하지 않으면 마치 해당 행이 존재하지 않는 것처럼 처리하여 업데이트를 거부한다. 이 모든 과정은 개발자가 `WHERE author_id =?` 조건을 쿼리에 명시적으로 추가하지 않아도 데이터베이스에 의해 투명하게 이루어진다.

특히, RLS는 다중 테넌시(Multi-Tenancy) SaaS 애플리케이션에서 테넌트 간의 데이터를 완벽하게 격리하는 데 이상적인 솔루션이다. 모든 데이터 테이블에 `tenant_id` 컬럼을 추가하고, `USING (tenant_id = (current_setting('request.jwt.claims', true)::json->>'tenant_id')::uuid)`와 같은 공통 정책을 적용하면, 애플리케이션 코드의 변경 없이 데이터베이스 수준에서 견고한 데이터 격리를 보장할 수 있다.30


PostgREST는 단순한 CRUD 작업을 넘어, 클라이언트가 데이터를 정교하게 요청하고 조작할 수 있도록 강력하고 유연한 고급 쿼리 기능을 제공한다. 이러한 기능들은 대부분 URL 쿼리 파라미터를 통해 제어되며, 내부적으로는 효율적인 SQL로 변환된다. 대표적인 기능으로는 수평적 필터링, 정렬, 리소스 임베딩, 그리고 원격 프로시저 호출(RPC)이 있다.


수평적 필터링은 SQL의 `WHERE` 절에 해당하는 기능으로, 특정 조건을 만족하는 행(row)들만 선택적으로 조회할 수 있게 한다. PostgREST는 이를 위해 `column=operator.value` 형식의 직관적인 쿼리 파라미터 문법을 제공한다.34

PostgREST가 지원하는 연산자는 PostgreSQL의 풍부한 연산자 체계를 대부분 반영하고 있어 매우 표현력이 높다. 다음 표는 주요 연산자와 그 사용법을 요약한 것이다.

| PostgREST 연산자 | PostgreSQL 연산자 | 의미                                   | 사용 예시 (URL)                     |
| ---------------- | ----------------- | -------------------------------------- | ----------------------------------- |
| `eq`             | `=`               | 같음 (equals)                          | `?age=eq.30`                        |
| `neq`            | `<>` 또는 `!=`    | 같지 않음 (not equal)                  | `?status=neq.archived`              |
| `gt`             | `>`               | 보다 큼 (greater than)                 | `?price=gt.100`                     |
| `gte`            | `>=`              | 크거나 같음 (greater than or equal)    | `?stock=gte.10`                     |
| `lt`             | `<`               | 보다 작음 (less than)                  | `?age=lt.18`                        |
| `lte`            | `<=`              | 작거나 같음 (less than or equal)       | `?rating=lte.5`                     |
| `like`           | `LIKE`            | 패턴 매칭 (대소문자 구분)              | `?name=like.John*`                  |
| `ilike`          | `ILIKE`           | 패턴 매칭 (대소문자 무시)              | `?name=ilike.john*`                 |
| `in`             | `IN`              | 목록에 포함                            | `?id=in.(1,5,10)`                   |
| `is`             | `IS`              | `NULL`, `TRUE`, `FALSE` 확인           | `?manager_id=is.null`               |
| `fts`            | `@@`              | 전문 검색 (Full-Text Search)           | `?document=fts.search-term`         |
| `cs`             | `@>`              | 포함 (contains, for array/jsonb)       | `?tags=cs.{news,tech}`              |
| `cd`             | `<@`              | 포함됨 (contained by, for array/jsonb) | `?range=cd.[2024-01-01,2024-01-31]` |

이러한 기본 연산자들은 논리 연산자 `and`, `or`, `not`과 결합하여 더욱 복잡한 조건을 구성할 수 있다. 여러 쿼리 파라미터는 기본적으로 `AND`로 연결된다. `OR` 조건을 사용하려면 `or=(condition1,condition2)`와 같은 형식을 사용한다. 예를 들어, 18세 미만이거나 65세 이상인 사용자를 찾으려면 `?or=(age.lt.18,age.gte.65)`와 같이 요청할 수 있다.35

정렬은 `order` 파라미터를 통해 제어된다. 쉼표로 구분하여 여러 정렬 기준을 지정할 수 있으며, 각 기준마다 정렬 방향(`asc` 또는 `desc`)과 `NULL` 값의 처리 방식(`nullsfirst` 또는 `nullslast`)을 명시할 수 있다.35 예를 들어, `?order=priority.desc.nullslast,created_at.asc`는 우선순위가 높은 순으로, 우선순위가 같다면 생성된 순서대로 정렬하되, 우선순위가 `NULL`인 항목은 맨 뒤로 보낸다.

이러한 강력한 필터링 및 정렬 기능은 클라이언트가 필요한 데이터를 매우 정확하게 요청할 수 있도록 하여, 불필요한 데이터 전송(over-fetching)을 방지하고 클라이언트 측의 데이터 처리 로직을 크게 단순화시킨다. 이는 GraphQL이 해결하고자 하는 핵심 문제 중 하나를 RESTful한 방식으로 우아하게 해결하는 접근법이다.


REST API의 고질적인 문제 중 하나는 관련된 리소스를 가져오기 위해 여러 번의 HTTP 요청이 필요한, 이른바 'N+1 쿼리 문제'이다. 예를 들어, 게시글 목록과 각 게시글의 작성자 정보를 함께 표시하려면, 먼저 게시글 목록을 가져온 뒤(`GET /posts`), 각 게시글에 대해 작성자 정보를 개별적으로 요청(`GET /users/{id}`)해야 할 수 있다.

PostgREST의 리소스 임베딩 기능은 이 문제를 해결하는 강력한 메커니즘이다. 데이터베이스에 정의된 외래 키(Foreign Key) 관계를 자동으로 인식하여, 단 한 번의 API 요청으로 관련된 리소스들을 중첩된 JSON 구조로 함께 반환할 수 있다.37

임베딩은 select 쿼리 파라미터를 통해 명시적으로 요청된다. 예를 들어, films 테이블과 directors 테이블이 외래 키로 연결되어 있을 때, 영화 정보와 감독 정보를 함께 가져오려면 다음과 같이 요청한다:

```
GET /films?select=title,year,director:directors(first_name,last_name)
```

이 요청에 대한 응답은 다음과 같은 구조를 가질 것이다:

```JSON
[
  {
    "title": "Inception",
    "year": 2010,
    "director": {
      "first_name": "Christopher",
      "last_name": "Nolan"
    }
  }
]
```

PostgREST는 일대다(one-to-many), 다대일(many-to-one), 다대다(many-to-many) 관계를 모두 지원하며, 관계의 종류에 따라 중첩된 데이터는 JSON 객체(to-one) 또는 JSON 배열(to-many)로 표현된다.37 또한, 임베드된 리소스에 대해서도 필터링, 정렬, 개수 제한이 가능하다. 예를 들어, 특정 영화의 출연 배우 중 성이 'D'로 시작하는 배우만 알파벳 순으로 5명까지 가져오려면 다음과 같이 요청할 수 있다:

```
GET /films?id=eq.123&select=title,actors(first_name,last_name)&actors.last_name=like.D*&actors.order=last_name.asc&actors.limit=5
```

내부적으로 PostgREST는 이러한 임베딩 요청을 효율적인 단일 SQL 쿼리로 변환한다. 이를 위해 PostgreSQL의 `JOIN` 구문과 `json_agg`, `json_build_object`와 같은 JSON 집계 함수를 적극적으로 활용한다. 이 방식은 네트워크 왕복 횟수를 최소화하고, 데이터베이스가 가장 효율적으로 데이터를 집계하도록 하여 전체적인 애플리케이션 성능을 크게 향상시킨다.


PostgREST는 테이블과 뷰를 통한 데이터 중심의 접근뿐만 아니라, PostgreSQL 함수(저장 프로시저)를 직접 호출하여 복잡한 비즈니스 로직을 수행할 수 있는 원격 프로시저 호출(RPC) 기능을 제공한다.16 노출된 스키마에 정의된 모든 함수는 `/rpc/{function_name}` 형태의 엔드포인트를 통해 API로 노출된다.

RPC는 단순한 CRUD를 넘어서는 작업들을 캡슐화하는 데 매우 유용하다. 예를 들어, '사용자 등록'과 같은 작업은 단순히 `users` 테이블에 행을 하나 추가하는 것으로 끝나지 않는다. 관련 테이블(예: `user_profiles`)에 기본 데이터를 생성하고, 환영 이메일을 발송하기 위해 큐에 작업을 넣는 등 여러 단계의 로직이 포함될 수 있다. 이러한 여러 단계를 개별 API 호출로 처리하면 트랜잭션의 원자성을 보장하기 어렵다.

PostgREST의 RPC를 사용하면, 이 모든 로직을 `signup(email text, password text)`와 같은 단일 PostgreSQL 함수 안에 캡슐화할 수 있다. 함수 내에서 트랜잭션 블록(`BEGIN...COMMIT/ROLLBACK`)을 사용하여 모든 작업의 원자성을 완벽하게 보장할 수 있다. 클라이언트는 단순히 `POST /rpc/signup` 엔드포인트에 필요한 파라미터를 JSON 본문으로 전송하는 것만으로 이 복잡한 워크플로우를 안전하게 실행할 수 있다.16

```SQL
-- 사용자 등록을 처리하는 PostgreSQL 함수 예시
CREATE OR REPLACE FUNCTION api.signup(email text, password text)
RETURNS void AS $$
DECLARE
  new_user_id uuid;
BEGIN
  -- 1. users 테이블에 새로운 사용자 삽입
  INSERT INTO auth.users (email, password_hash)
  VALUES (email, crypt(password, gen_salt('bf')))
  RETURNING id INTO new_user_id;

  -- 2. user_profiles 테이블에 기본 프로필 생성
  INSERT INTO public.user_profiles (user_id, username)
  VALUES (new_user_id, split_part(email, '@', 1));

  -- 3. 이메일 발송 큐에 작업 추가 (가상의 테이블)
  INSERT INTO jobs.email_queue (type, payload)
  VALUES ('welcome_email', jsonb_build_object('user_id', new_user_id));
END;
$$ LANGUAGE plpgsql VOLATILE SECURITY DEFINER;
```

또한, 함수가 테이블 타입(`SETOF table_name` 또는 `RETURNS TABLE(...)`)을 반환하는 경우, 해당 RPC 엔드포인트는 일반 테이블 엔드포인트처럼 필터링, 정렬, 리소스 임베딩 등 모든 고급 쿼리 기능을 동일하게 지원한다.16 이는 데이터베이스를 단순한 저장소를 넘어, 강력하고 유연한 애플리케이션 서버로 활용할 수 있는 가능성을 열어준다.

## 제4장: 성능 최적화 전략

PostgREST의 성능은 그 아키텍처 철학에 깊이 뿌리박고 있으며, 대부분의 경우 하부에 위치한 PostgreSQL 데이터베이스의 성능에 직접적으로 의존한다. 따라서 PostgREST 기반 시스템의 성능을 최적화하는 것은 PostgREST 자체의 튜닝보다는 PostgreSQL 데이터베이스를 효율적으로 설계하고 튜닝하는 과정과 거의 동일하다.

### PostgREST 고유의 성능 특성

PostgREST 자체는 매우 낮은 오버헤드를 갖도록 설계되었다. 이러한 성능 우위는 몇 가지 핵심적인 기술 선택에서 비롯된다:

- **컴파일된 언어 및 고성능 웹 서버**: PostgREST는 컴파일된 언어인 Haskell로 작성되었으며, 경량 스레드를 통해 높은 동시성 처리가 가능한 비동기 웹 서버 Warp 위에서 실행된다. 이는 인터프리터 언어로 작성된 많은 백엔드 프레임워크에 비해 본질적으로 더 나은 CPU 및 메모리 효율성을 제공한다.4
- **상태 비저장(Stateless) 아키텍처**: JWT 기반 인증을 통해 서버 측에 세션 상태를 유지할 필요가 없으므로, 각 PostgREST 인스턴스는 독립적으로 요청을 처리할 수 있다. 이는 로드 밸런서 뒤에 여러 인스턴스를 배치하여 수평적으로 손쉽게 확장(scale out)할 수 있게 해준다.6
- **효율적인 데이터베이스 통신**: 내부적으로 Hasql 라이브러리를 사용하여 PostgreSQL의 효율적인 바이너리 프로토콜로 통신하며, 이는 텍스트 기반 프로토콜에 비해 데이터 전송 오버헤드를 줄여준다.6

이러한 특성들로 인해, API 요청 처리 과정에서 PostgREST 자체가 병목이 되는 경우는 드물다. 성능 문제는 거의 항상 PostgREST가 생성하여 실행하는 SQL 쿼리와, 그 쿼리를 처리하는 PostgreSQL 데이터베이스의 상태에서 발생한다. 따라서 최적화의 초점은 데이터베이스 계층에 맞춰져야 한다.

### PostgreSQL로의 작업 위임: 성능의 핵심

PostgREST의 성능 철학은 "데이터가 있는 곳에서 코드를 실행한다"는 원칙을 따른다. 즉, 가능한 한 많은 계산과 로직을 데이터베이스 계층으로 위임하여, 네트워크를 통해 대량의 원시 데이터를 애플리케이션 계층으로 전송하는 비용을 최소화한다.6

- **JSON 직렬화**: 클라이언트가 중첩된 JSON 응답을 요청하면(리소스 임베딩), PostgREST는 여러 테이블의 데이터를 애플리케이션 메모리로 가져와 조립하는 대신, PostgreSQL의 내장 JSON 함수(`json_agg`, `json_build_object` 등)를 활용하여 데이터베이스 내에서 최종 JSON 구조를 생성하는 단일 쿼리를 실행한다. 이 방식은 네트워크 트래픽을 극적으로 줄이고, 고도로 최적화된 PostgreSQL의 C 함수를 통해 직렬화를 수행하므로 매우 효율적이다.
- **데이터 유효성 검사 및 권한 부여**: 데이터의 유효성은 `CHECK` 제약 조건으로, 접근 권한은 역할(Role)과 RLS 정책으로 데이터베이스에서 직접 처리된다. 이는 모든 요청에 대해 애플리케이션 계층에서 추가적인 검증 쿼리나 로직을 실행할 필요가 없음을 의미하며, 응답 지연 시간(latency)을 줄이는 데 기여한다.

100만 건의 판매 기록에서 월별 매출 합계를 계산하는 시나리오를 생각해보자. 전통적인 방식은 100만 건의 데이터를 모두 애플리케이션 서버로 가져와 메모리에서 집계하는 반면, PostgREST는 `SUM()`과 `GROUP BY`를 포함한 SQL 쿼리를 데이터베이스로 보내고 최종 결과인 12개의 행만 돌려받는다. 데이터 규모가 커질수록 이 차이는 기하급수적으로 벌어지며, PostgREST 접근법의 성능 우위가 명확해진다.

### 연결 풀링(Connection Pooling) 최적화 방안

PostgreSQL은 클라이언트 연결마다 별도의 서버 프로세스를 생성하기 때문에 연결 설정 비용이 상대적으로 높다. 따라서 높은 동시성을 처리하기 위해서는 연결 풀링(Connection Pooling)이 필수적이다.40

PostgREST는 자체적으로 데이터베이스 연결 풀을 내장하고 관리하여, 들어오는 API 요청에 신속하게 데이터베이스 연결을 할당하고 재사용한다.6 대부분의 중소 규모 워크로드에서는 이 내장 풀만으로도 충분하다.

그러나 수천 개 이상의 동시 연결이 발생하는 대규모 시스템에서는 PgBouncer와 같은 외부 전문 연결 풀러를 사용하는 것이 더 효율적일 수 있다.40 PgBouncer는 트랜잭션 풀링(transaction pooling) 모드에서 매우 가볍게 동작하여 훨씬 더 많은 수의 클라이언트 연결을 적은 수의 실제 데이터베이스 연결로 다중화(multiplexing)할 수 있다.

PostgREST와 PgBouncer를 트랜잭션 풀링 모드로 함께 사용할 때는 중요한 설정 조정이 필요하다. PostgREST는 기본적으로 성능 향상을 위해 준비된 구문(prepared statements)을 사용하는데, 이는 세션(연결)에 종속적이다. 트랜잭션 풀링 모드에서는 각 트랜잭션이 풀 내의 다른 실제 데이터베이스 연결을 사용할 수 있으므로, 한 트랜잭션에서 준비된 구문을 다른 트랜잭션에서 사용하려 할 때 호환성 문제가 발생할 수 있다. 이 문제를 해결하기 위해, PostgREST의 설정 파일에서 `db-prepared-statements = false`로 설정해야 한다.42 이는 약간의 성능 저하를 감수하는 대신, 대규모 동시성 처리 능력을 확보하기 위한 중요한 절충안(trade-off)이다.

### PostgREST 환경을 위한 PostgreSQL 인덱싱 전략 및 튜닝 모범 사례

PostgREST API의 성능은 PostgreSQL 쿼리 성능과 직결되므로, 효과적인 인덱싱 전략이 무엇보다 중요하다.

- **인덱싱 전략**:
  - **B-Tree 인덱스**: `WHERE` 절에 해당하는 모든 필터링 파라미터(예: `?user_id=eq.123`)와 `ORDER BY`에 사용되는 컬럼에 B-Tree 인덱스를 생성하는 것이 가장 기본적이고 효과적이다.43
  - **외래 키 인덱스**: 리소스 임베딩(JOIN)에 사용되는 외래 키 컬럼에는 반드시 인덱스를 생성해야 한다. 이는 중첩된 데이터를 조회할 때 발생하는 조인 성능을 극적으로 향상시킨다.44
  - **복합 인덱스(Composite Indexes)**: 여러 컬럼이 함께 필터링 조건으로 자주 사용되는 경우, 해당 컬럼들을 포함하는 복합 인덱스를 생성하는 것이 효과적이다. 인덱스 내 컬럼의 순서는 쿼리 패턴과 일치시키는 것이 중요하다.45
  - **GIN/GiST 인덱스**: JSONB 컬럼에 대한 `cs` (`@>`) 연산자나 전문 검색(`fts`)과 같은 고급 필터링 기능을 사용하는 경우, 해당 컬럼에 GIN 또는 GiST 인덱스를 생성하면 성능을 크게 향상시킬 수 있다.43
- **PostgreSQL 파라미터 튜닝**: `postgresql.conf` 파일의 주요 파라미터를 워크로드에 맞게 조정하는 것이 중요하다. 다음 표는 PostgREST 환경에서 특히 중요한 파라미터와 권장 사항을 요약한 것이다.

| 파라미터               | 설명                                                         | 기본값 | 권장 값 및 고려사항                                          |
| ---------------------- | ------------------------------------------------------------ | ------ | ------------------------------------------------------------ |
| `shared_buffers`       | PostgreSQL이 데이터 캐싱을 위해 사용하는 주 메모리 영역.     | 128MB  | 시스템 전체 RAM의 25% ~ 40%. 너무 높게 설정하면 OS 캐시와 경쟁하여 오히려 성능이 저하될 수 있다.40 |
| `work_mem`             | 각 정렬(sort), 해시(hash), 조인(join) 작업이 사용하는 메모리. | 4MB    | 복잡한 쿼리(JOIN, ORDER BY)가 많을수록 높게 설정. `max_connections` 수에 비례하여 총 메모리 사용량이 증가하므로 주의해야 한다.47 |
| `maintenance_work_mem` | `VACUUM`, `CREATE INDEX` 등 유지보수 작업에 사용되는 메모리. | 64MB   | 인덱스 생성이나 대규모 데이터 로딩 속도를 높이기 위해 512MB ~ 2GB 정도로 충분히 높게 설정하는 것이 좋다.47 |
| `random_page_cost`     | 비순차적 디스크 읽기의 비용 추정치.                          | 4.0    | 스토리지로 SSD나 NVMe를 사용하는 경우, 순차 읽기와 비순차 읽기의 비용 차이가 거의 없으므로 1.1과 같이 낮게 설정하여 쿼리 플래너가 인덱스 스캔을 더 선호하도록 유도한다.47 |
| `autovacuum` 관련      | `autovacuum_max_workers`, `autovacuum_vacuum_scale_factor` 등. | 다양함 | 쓰기 작업이 많은(high-write) 워크로드에서는 `scale_factor`를 낮추어 `VACUUM`이 더 자주 실행되도록 조정하여 테이블 블로트(bloat)를 방지하는 것이 중요하다.47 |

PostgREST 사용자는 API의 응답이 느릴 경우, 애플리케이션 코드를 디버깅하는 대신 `EXPLAIN ANALYZE`를 통해 해당 API 요청이 생성하는 SQL 쿼리의 실행 계획을 분석해야 한다. 문제의 원인이 'Seq Scan'이나 'filesort'로 밝혀지면, 해결책은 코드 수정이 아니라 적절한 인덱스를 추가하거나 PostgreSQL 파라미터를 튜닝하는 것이 될 것이다. 이처럼 문제 해결 과정이 전적으로 데이터베이스 내에서 이루어지는 것이 PostgREST 성능 최적화의 핵심이다.

## 제5장: 대안 기술과의 비교 분석

PostgREST는 독특한 철학과 아키텍처를 가지고 있어, 다른 백엔드 기술들과 비교했을 때 명확한 장단점과 트레이드오프를 보인다. 기술 스택을 선택하는 아키텍트는 PostgREST를 수동으로 구현하는 전통적인 백엔드, GraphQL 기반의 자동 API 생성 도구, 그리고 통합 BaaS(Backend-as-a-Service) 플랫폼과 비교하여 그 전략적 가치를 평가해야 한다.

### PostgREST vs. 수동 CRUD 백엔드 (예: Node.js, Express)

가장 전통적인 백엔드 개발 방식은 Node.js와 Express.js 같은 프레임워크를 사용하여 API의 모든 계층-라우팅, 컨트롤러, 서비스, 모델-을 직접 코딩하는 것이다.49

- **개발 속도 및 생산성**: 이 측면에서 PostgREST는 압도적인 우위를 보인다. 데이터베이스 스키마를 정의하는 것만으로 즉시 완전한 기능의 CRUD API가 생성되므로, 반복적인 보일러플레이트 코드를 작성할 필요가 전혀 없다.7 반면, Node.js/Express 스택에서는 각 엔드포인트에 대한 라우팅 로직, 데이터베이스 쿼리, 요청/응답 처리, 유효성 검사 등을 모두 수동으로 구현해야 하므로 상당한 시간과 노력이 소요된다.49
- **유연성 및 제어**: 수동 백엔드는 거의 무한한 유연성을 제공한다. 복잡한 비즈니스 워크플로우, 여러 외부 API와의 통합, 정교한 미들웨어(로깅, 모니터링, 캐싱) 구현 등 PostgREST의 범위를 벗어나는 요구사항을 자유롭게 처리할 수 있다.52 PostgREST는 복잡한 로직을 데이터베이스 함수(RPC)에 의존해야 하며, 특히 데이터베이스 외부의 자원(예: 파일 시스템, 외부 네트워크)에 접근하는 데에는 본질적인 제약이 있다.3
- **성능**: 일반적인 CRUD 워크로드에서, 잘 튜닝된 PostgREST는 Node.js 기반 API보다 훨씬 적은 CPU와 메모리를 사용하면서 더 높은 처리량(throughput)을 보이는 경향이 있다.5 이는 컴파일된 언어(Haskell)의 효율성과 데이터베이스로의 작업 위임을 통해 네트워크 및 애플리케이션 계층의 오버헤드를 최소화하기 때문이다.
- **유지보수**: PostgREST는 애플리케이션 코드베이스가 거의 없거나 전무하므로 유지보수 부담이 현저히 적다. 데이터베이스 스키마의 변경이 곧 API의 변경으로 직결되어 일관성이 강제된다.7 수동 백엔드는 데이터베이스 스키마 변경 시 관련 애플리케이션 코드(모델, 서비스 등)를 모두 찾아 동기화하고 수정해야 하는 추가적인 유지보수 비용이 발생한다.

결론적으로, 이 둘 사이의 선택은 '생산성/단순성'과 '유연성/제어' 사이의 근본적인 트레이드오프에 달려 있다. 애플리케이션의 핵심 기능이 데이터 중심의 CRUD 작업이라면 PostgREST가 월등한 이점을 제공한다. 반면, 복잡한 비즈니스 프로세스, 다수의 외부 시스템 연동, 고도의 커스터마이징이 필요하다면 수동 백엔드가 더 적합한 선택이다.

### PostgREST vs. Hasura: REST와 GraphQL 중심 접근법 비교

Hasura는 PostgREST와 마찬가지로 데이터베이스 스키마를 기반으로 API를 자동 생성하는 도구이지만, 핵심적인 철학과 기능 범위에서 차이를 보인다.

- **API 패러다임**: PostgREST는 이름 그대로 RESTful API를 생성하는 데 중점을 둔다.1 반면, Hasura는 주로 GraphQL API를 자동 생성하는 데 초점을 맞추고 있으며, 이를 통해 클라이언트가 필요한 데이터만 정확하게 요청할 수 있는 유연성을 제공한다.53
- **데이터 소스 지원**: PostgREST는 오직 PostgreSQL 데이터베이스만을 지원한다.56 Hasura의 가장 큰 강점 중 하나는 PostgreSQL 외에도 다수의 SQL/NoSQL 데이터베이스와 외부 REST/GraphQL 서비스를 데이터 소스로 지원한다는 점이다. 더 나아가, 이러한 여러 데이터 소스를 하나의 통합된 GraphQL 스키마로 묶는 '데이터 연합(Data Federation)' 기능을 제공한다.54
- **커스텀 로직 확장**: PostgREST에서 커스텀 비즈니스 로직은 PostgreSQL 함수(RPC)를 통해 데이터베이스 내에서 구현된다.16 Hasura는 'Actions' 또는 'Remote Schemas'라는 기능을 통해 외부의 웹훅 엔드포인트나 서버리스 함수를 호출하여 비즈니스 로직을 통합하는 방식을 사용한다. 이는 로직을 데이터베이스 외부에서 관리할 수 있게 해준다.54
- **사용자 경험 및 도구**: Hasura는 데이터베이스 스키마 관리, 관계 설정, 권한 정의, API 탐색 등을 위한 강력하고 직관적인 웹 기반 관리 UI를 제공한다.57 이는 개발자의 생산성을 크게 향상시킨다. PostgREST는 순수한 CLI 기반 도구로, 별도의 관리 UI를 제공하지 않는다.3

Hasura는 PostgREST보다 더 넓은 범위의 문제를 해결하려는 'API 플랫폼' 또는 '데이터 게이트웨이'에 가깝다. 여러 분산된 데이터 소스를 통합하고 프론트엔드 개발자에게 GraphQL의 강력한 기능을 제공하는 데 중점을 둔다. 반면, PostgREST는 '하나의 일을 잘하자(Do One Thing Well)'는 유닉스 철학에 따라, PostgreSQL을 REST API로 노출하는 단일 목적에 집중하며, 더 가볍고 미니멀한 접근 방식을 유지한다.

### PostgREST vs. Supabase: 독립 실행형 도구와 BaaS(Backend-as-a-Service) 플랫폼 비교

PostgREST와 Supabase는 경쟁 관계가 아니라, '엔진'과 그 엔진을 탑재한 '완성차'의 관계에 가깝다.58

- **핵심 관계**: Supabase는 PostgREST를 자사 플랫폼의 핵심 구성 요소로 사용하여 'Instant APIs'라는 기능을 제공한다. 즉, Supabase의 데이터베이스 API는 내부적으로 PostgREST에 의해 구동된다.39
- **제공 범위**: PostgREST는 단일 목적을 가진 독립 실행형(standalone) 웹 서버 바이너리이다.1 사용자는 이를 직접 자신의 서버에 설치하고, 기존 또는 새로운 PostgreSQL 데이터베이스에 연결하여 운영해야 한다. 반면, Supabase는 PostgREST를 포함하여 완전한 백엔드 솔루션을 제공하는 통합 개발 플랫폼(BaaS)이다. Supabase는 관리형 PostgreSQL 데이터베이스 호스팅, 사용자 인증(Authentication), 파일 스토리지(Storage), 실시간(Realtime) 데이터 구독, 서버리스 함수(Edge Functions) 등 애플리케이션 개발에 필요한 거의 모든 백엔드 구성 요소를 통합하여 제공한다.59
- **관리 주체 및 편의성**: PostgREST를 사용한다는 것은 인프라(서버, 데이터베이스, 네트워크 등)를 직접 관리하고 설정해야 함을 의미한다. Supabase는 이 모든 것을 관리형 서비스로 제공하여, 개발자가 인프라 걱정 없이 애플리케이션 로직 개발에만 집중할 수 있도록 돕는다.60

선택의 기준은 제어 수준과 편의성 사이에서 결정된다. 빠르게 프로토타입을 구축하거나 백엔드 인프라 관리 경험이 적은 개발자에게는 모든 것이 통합된 Supabase가 이상적인 솔루션이다. 반면, 이미 자체 PostgreSQL 인프라를 보유하고 있거나, 특정 클라우드 환경에 대한 요구사항이 있거나, BaaS가 제공하지 않는 세밀한 수준의 데이터베이스 제어가 필요한 경우에는 독립 실행형 PostgREST를 직접 배포하여 사용하는 것이 더 적합하다. Supabase의 성공은 PostgREST의 데이터베이스 중심 API 패러다임이 얼마나 강력하고 실용적인지를 입증하는 중요한 사례이기도 하다.

다음 표는 네 가지 백엔드 아키텍처 접근법의 주요 특징을 요약하여 비교한 것이다.

| 기준                     | PostgREST                        | 수동 CRUD (Node.js)                     | Hasura                              | Supabase                             |
| ------------------------ | -------------------------------- | --------------------------------------- | ----------------------------------- | ------------------------------------ |
| **API 패러다임**         | REST, RPC                        | REST, GraphQL 등 자유롭게 구현          | GraphQL 우선, REST 지원             | REST, RPC (PostgREST 기반), Realtime |
| **주요 데이터 소스**     | PostgreSQL 전용                  | 모든 데이터베이스 (ORM 통해)            | 다중 DB, REST/GraphQL 서비스 연합   | 관리형 PostgreSQL                    |
| **개발 속도 (CRUD)**     | 최상 (스키마 정의 시 즉시 생성)  | 낮음 (모든 계층 수동 구현)              | 매우 높음 (UI 통해 즉시 생성)       | 최상 (BaaS 플랫폼으로 즉시 제공)     |
| **비즈니스 로직 유연성** | 중간 (DB 함수/RPC에 의존)        | 최상 (언어/프레임워크의 모든 기능 활용) | 높음 (외부 웹훅/서버리스 함수 연동) | 높음 (Edge Functions, DB 함수 활용)  |
| **보안 모델**            | DB 중심 (Role, RLS)              | 애플리케이션 중심 (미들웨어, 코드)      | 플랫폼 중심 (UI 기반 권한 규칙)     | 플랫폼 중심 (Auth, RLS)              |
| **운영 복잡도**          | 중간 (Self-hosted, DB 튜닝 필요) | 높음 (전체 스택 직접 관리)              | 중간 (Self-hosted 또는 Cloud)       | 매우 낮음 (완전 관리형 서비스)       |
| **학습 곡선**            | PostgreSQL 및 SQL 숙련도 요구    | Node.js, 프레임워크, ORM 학습           | GraphQL 및 Hasura 개념 학습         | 플랫폼 사용법 학습 (상대적으로 낮음) |
| **생태계/확장성**        | PostgreSQL 생태계에 의존         | Node.js(npm)의 방대한 생태계            | GraphQL 생태계, 커스텀 확장         | 플랫폼 내 통합 기능, 확장 프로그램   |

## 결론: PostgREST의 전략적 가치와 한계

PostgREST는 PostgreSQL 데이터베이스를 중심으로 API를 구축하는 혁신적이고 강력한 패러다임을 제시한다. 데이터베이스를 신뢰의 단일 원천으로 삼고, 선언적인 방식으로 API를 자동 생성함으로써, 전통적인 백엔드 개발의 많은 복잡성과 비효율을 해결한다. 그러나 모든 문제를 해결하는 만병통치약이 아니라, 특정 종류의 문제를 매우 효과적으로 해결하는 '날카로운 도구(sharp tool)'로 이해해야 한다.1

### PostgREST가 최적의 선택이 되는 사용 사례

PostgREST의 전략적 가치는 다음과 같은 시나리오에서 극대화된다:

- **신속한 프로토타이핑 및 MVP(Minimum Viable Product) 개발**: 데이터 모델을 SQL로 정의하는 즉시 완벽하게 작동하는 API를 확보할 수 있어, 아이디어를 검증하고 시장에 출시하는 속도를 비약적으로 단축시킨다.7
- **데이터 중심 내부 도구 및 대시보드**: 복잡한 비즈니스 로직보다는 데이터의 조회, 생성, 수정이 주된 목적인 관리자 패널, 데이터 분석용 대시보드, 내부 CRM 등의 백엔드로 이상적이다. HTTP와 cURL, jq 같은 표준 도구만으로도 강력한 데이터 조작이 가능하다.56
- **마이크로서비스 아키텍처**: 특정 데이터 도메인(예: 사용자, 제품, 주문)을 전담하는 경량 마이크로서비스를 구축할 때, PostgREST는 해당 서비스의 API 계층 역할을 매우 효율적으로 수행할 수 있다. 이는 서비스의 코드베이스를 최소화하고 유지보수를 용이하게 한다.5
- **모바일 및 SPA(Single Page Application) 백엔드**: 클라이언트가 직접 데이터베이스와 상호작용하는 듯한 단순한 2계층(2-tier) 아키텍처를 구축하고자 할 때, PostgREST는 안전하고 강력한 중간 계층을 제공한다. PostgreSQL의 정교한 보안 모델이 데이터베이스를 직접 노출하는 위험을 완벽하게 차단한다.58

### 고려해야 할 제약 조건 및 한계

PostgREST를 도입하기 전에 반드시 그 한계를 명확히 인지해야 한다:

- **복잡한 비즈니스 로직의 구현**: 단순한 CRUD를 넘어서는 복잡한 워크플로우나 비즈니스 규칙은 모두 PostgreSQL 함수(RPC)로 구현해야 한다. 이는 PL/pgSQL과 같은 데이터베이스 프로그래밍에 대한 높은 수준의 전문성을 요구하며, 모든 종류의 로직을 SQL로 표현하는 것은 비효율적이거나 불가능할 수 있다.3
- **외부 시스템 연동의 어려움**: 데이터베이스 함수 내에서 외부 API를 호출하거나, 이메일을 발송하거나, 파일 시스템에 접근하는 등의 작업은 PostgreSQL에서 직접적으로 지원하지 않는다. 이러한 요구사항이 있을 경우, `LISTEN/NOTIFY`와 외부 워커 프로세스를 결합하는 비동기적인 방식이나 별도의 마이크로서비스를 구축하는 등 우회적인 아키텍처가 필요하다.63
- **스키마와의 강한 결합(Tight Coupling)**: API가 데이터베이스 스키마에 직접적으로 결합되어 있기 때문에, 스키마의 변경(예: 컬럼 이름 변경, 테이블 구조 변경)이 의도치 않게 API의 브레이킹 체인지(breaking change)로 이어질 수 있다.51 이러한 결합도를 낮추고 안정적인 API 인터페이스를 제공하기 위해서는, 실제 테이블 대신 뷰(View)를 API의 공개 계층으로 사용하는 전략이 적극 권장된다.6
- **PostgreSQL에 대한 종속성**: PostgREST는 이름에서 알 수 있듯이 PostgreSQL에 완전히 종속적이다. 프로젝트의 미래에 다른 종류의 데이터베이스로 마이그레이션할 가능성이 조금이라도 있다면 PostgREST는 적합한 선택이 아니다.10

### 미래 아키텍처를 위한 제언

PostgREST의 진정한 가치는 그것을 단독 솔루션으로 보기보다는, 현대적인 분산 시스템 아키텍처의 강력한 구성 요소 중 하나로 인식할 때 발현된다. 모든 백엔드 로직을 PostgREST 하나로 해결하려는 시도보다는, 각 도구가 가장 잘하는 일에 집중하도록 하는 하이브리드 접근법이 훨씬 더 효과적이고 확장 가능하다.

예를 들어, 데이터 중심의 CRUD 및 복잡한 조회 작업은 PostgREST에 맡겨 개발 생산성과 성능을 극대화한다. 동시에, 외부 시스템과의 연동, 계산 집약적인 비동기 작업, 실시간 통신 등 PostgREST가 처리하기 어려운 비즈니스 프로세스는 Node.js, Go, Python 등으로 작성된 별도의 경량 마이크로서비스가 담당하도록 역할을 분담할 수 있다.

이러한 아키텍처에서 PostgREST는 '데이터 서비스'로서의 역할을 수행하며, 다른 마이크로서비스들은 '비즈니스 로직 서비스'로서 PostgREST API를 소비하거나, 독립적인 역할을 수행한다. 이 접근법은 PostgREST가 제공하는 개발 속도와 성능상의 이점을 온전히 누리면서도, 커스텀 백엔드가 제공하는 무한한 유연성을 확보하는 균형 잡힌 시스템을 구축할 수 있게 한다. 결국 PostgREST는 개발자에게 더 많은 선택지를 제공하며, 데이터베이스의 잠재력을 최대한 활용하여 더 단순하고, 빠르며, 안전한 API를 구축할 수 있는 새로운 길을 열어주는 강력한 도구이다.

#### **참고 자료**

1. PostgREST Documentation - PostgREST 13.0 documentation, 8월 3, 2025에 액세스, https://postgrest.org/
2. PostgREST 12.2 documentation, 8월 3, 2025에 액세스, https://docs.postgrest.org/en/v12/index.html
3. Automatic APIs With PostgREST for PostgreSQL - DreamFactory Blog, 8월 3, 2025에 액세스, https://blog.dreamfactory.com/postgrest-for-postgresql-pros-and-cons
4. Introducing PostgREST, a REST API for any PostgreSQL database written in Haskell - Packt, 8월 3, 2025에 액세스, https://www.packtpub.com/de-my/learning/tech-news/introducing-postgrest-a-rest-api-for-any-postgresql-database-written-in-haskell
5. PostgREST 10.2 documentation, 8월 3, 2025에 액세스, https://docs.postgrest.org/en/v10/
6. PostgREST/postgrest: REST API for any Postgres database - GitHub, 8월 3, 2025에 액세스, https://github.com/PostgREST/postgrest
7. Create a CRUD API With PostgREST - GeeksforGeeks, 8월 3, 2025에 액세스, https://www.geeksforgeeks.org/dbms/create-a-crud-api-with-postgrest/
8. Stop calling PostgREST "MAGIC"! - freeCodeCamp, 8월 3, 2025에 액세스, https://www.freecodecamp.org/news/stop-calling-postgrest-magic-8f3e1d5e5dd1/
9. Architecture - PostgREST 12.2 documentation, 8월 3, 2025에 액세스, https://docs.postgrest.org/en/v12/explanations/architecture.html
10. Modern Web Architecture Without a Backend - Using PostgREST | ZenStack, 8월 3, 2025에 액세스, https://zenstack.dev/blog/postgrest
11. Building a REST API with PostgreSQL and PostgREST (No ORM Needed) | by Rizqi Mulki, 8월 3, 2025에 액세스, https://medium.com/@rizqimulkisrc/building-a-rest-api-with-postgresql-and-postgrest-no-orm-needed-4c34a987ec4a
12. API - PostgREST 12.2 documentation, 8월 3, 2025에 액세스, https://docs.postgrest.org/en/v12/references/api.html
13. Configuration - PostgREST 12.2 documentation, 8월 3, 2025에 액세스, https://docs.postgrest.org/en/v12/references/configuration.html
14. Configuration - PostgREST 10.2 documentation, 8월 3, 2025에 액세스, https://docs.postgrest.org/en/v10/configuration.html
15. API - PostgREST 13.0 documentation, 8월 3, 2025에 액세스, https://docs.postgrest.org/en/stable/references/api.html
16. Functions as RPC - PostgREST 12.2 documentation, 8월 3, 2025에 액세스, https://docs.postgrest.org/en/v12/references/api/functions.html
17. Stored Procedures - PostgREST 11.2 documentation, 8월 3, 2025에 액세스, https://postgrest.org/en/v11/references/api/stored_procedures.html
18. OPTIONS method - PostgREST 12.2 documentation, 8월 3, 2025에 액세스, https://docs.postgrest.org/en/v12/references/api/options.html
19. docs.postgrest.org, 8월 3, 2025에 액세스, [https://docs.postgrest.org/en/v12/references/auth.html#:~:text=PostgREST%20is%20designed%20to%20keep,the%20database%20authorize%20client%20actions.](https://docs.postgrest.org/en/v12/references/auth.html#:~:text=PostgREST is designed to keep,the database authorize client actions.)
20. Authentication - PostgREST 12.2 documentation, 8월 3, 2025에 액세스, https://docs.postgrest.org/en/v12/references/auth.html
21. Authentication - PostgREST 13.0 documentation, 8월 3, 2025에 액세스, https://docs.postgrest.org/en/v13/references/auth.html
22. Database Authorization - PostgREST 12.2 documentation, 8월 3, 2025에 액세스, https://docs.postgrest.org/en/v12/explanations/db_authz.html
23. Tutorial 1 - The Golden Key - PostgREST 12.2 documentation, 8월 3, 2025에 액세스, https://docs.postgrest.org/en/v12/tutorials/tut1.html
24. Overview of Role System - PostgREST 10.2 documentation, 8월 3, 2025에 액세스, https://docs.postgrest.org/en/v10/auth.html
25. get_settings('request.jwt.claims.role') not working as described in the documentation #469, 8월 3, 2025에 액세스, https://github.com/PostgREST/postgrest/issues/2057
26. Tutorial 0 - Get it Running - PostgREST devel documentation, 8월 3, 2025에 액세스, https://docs.postgrest.org/en/latest/tutorials/tut0.html
27. Tutorial 0 - Get it Running - PostgREST 11.2 documentation, 8월 3, 2025에 액세스, https://postgrest.org/en/v11/tutorials/tut0.html
28. Documentation: 17: 5.9. Row Security Policies - PostgreSQL, 8월 3, 2025에 액세스, https://www.postgresql.org/docs/current/ddl-rowsecurity.html
29. PostgreSQL Row Level Security (RLS): Basics and Examples - Satori Cyber, 8월 3, 2025에 액세스, https://satoricyber.com/postgres-security/postgres-row-level-security/
30. Mastering PostgreSQL Row-Level Security (RLS) for Rock-Solid Multi-Tenancy, 8월 3, 2025에 액세스, https://ricofritzsche.me/mastering-postgresql-row-level-security-rls-for-rock-solid-multi-tenancy/
31. Row-Level Security (RLS) in PostgreSQL for Multi-Tenant SaaS Apps | by Anand Thakkar, 8월 3, 2025에 액세스, https://medium.com/@anand_thakkar/row-level-security-rls-in-postgresql-for-multi-tenant-saas-apps-ef8c324031d0
32. Row-level security recommendations - AWS Prescriptive Guidance, 8월 3, 2025에 액세스, https://docs.aws.amazon.com/prescriptive-guidance/latest/saas-multitenant-managed-postgresql/rls.html
33. Multi-tenant data isolation with PostgreSQL Row Level Security | AWS Database Blog, 8월 3, 2025에 액세스, https://aws.amazon.com/blogs/database/multi-tenant-data-isolation-with-postgresql-row-level-security/
34. Tables and Views - PostgREST 12.2 documentation, 8월 3, 2025에 액세스, https://docs.postgrest.org/en/v12/references/api/tables_views.html
35. Tables and Views - PostgREST 13.0 documentation, 8월 3, 2025에 액세스, https://docs.postgrest.org/en/v13/references/api/tables_views.html
36. Filter PostgreSQL Data: WHERE, GROUP BY, HAVING, and LIMIT - Prisma, 8월 3, 2025에 액세스, https://www.prisma.io/dataguide/postgresql/reading-and-querying-data/filtering-data
37. Resource Embedding - PostgREST 12.2 documentation, 8월 3, 2025에 액세스, https://docs.postgrest.org/en/v12/references/api/resource_embedding.html
38. Resource Embedding - PostgREST devel documentation, 8월 3, 2025에 액세스, https://docs.postgrest.org/en/latest/references/api/resource_embedding.html
39. PostgREST a Little Gem: Supabase Postgres Functions with PostgREST | by balaji bal | STREAM-ZERO | Medium, 8월 3, 2025에 액세스, https://medium.com/stream-zero/postgrest-a-little-gem-supabase-postgres-functions-with-postgrest-d7819377d453
40. PostgreSQL Performance Tuning Guide: Settings That Make a Difference - Percona, 8월 3, 2025에 액세스, https://www.percona.com/blog/tuning-postgresql-database-parameters-to-optimize-performance/
41. optimize connection pooling in PostgreSQL Flexible Server - Microsoft Q&A, 8월 3, 2025에 액세스, https://learn.microsoft.com/en-us/answers/questions/2103126/optimize-connection-pooling-in-postgresql-flexible
42. Hardening PostgREST - PostgREST 10.2 documentation, 8월 3, 2025에 액세스, https://docs.postgrest.org/en/v10/admin.html
43. Documentation: 17: 11.2. Index Types - PostgreSQL, 8월 3, 2025에 액세스, https://www.postgresql.org/docs/current/indexes-types.html
44. PostgreSQL Performance Tuning: Optimizing Database Indexes | TigerData, 8월 3, 2025에 액세스, https://www.tigerdata.com/learn/postgresql-performance-tuning-optimizing-database-indexes
45. PostgreSQL Index Best Practices for Faster Queries - Mydbops, 8월 3, 2025에 액세스, https://www.mydbops.com/blog/postgresql-indexing-best-practices-guide
46. PostgreSQL Indexes - Neon, 8월 3, 2025에 액세스, https://neon.com/postgresql/postgresql-indexes
47. PostgreSQL Performance Tuning Best Practices 2025 - Mydbops, 8월 3, 2025에 액세스, https://www.mydbops.com/blog/postgresql-parameter-tuning-best-practices
48. PostgreSQL tuning: 6 things you can do to improve DB performance - NetApp Instaclustr, 8월 3, 2025에 액세스, https://www.instaclustr.com/education/postgresql/postgresql-tuning-6-things-you-can-do-to-improve-db-performance/
49. CRUD REST API with Node.js, Express, and PostgreSQL - LogRocket Blog, 8월 3, 2025에 액세스, https://blog.logrocket.com/crud-rest-api-node-js-express-postgresql/
50. Building a Robust Backend from Scratch: A Complete Guide to Node.js, Express, and PostgreSQL | by Kashanaqeel | Medium, 8월 3, 2025에 액세스, https://medium.com/@kashanaqeel74/building-a-robust-backend-from-scratch-a-complete-guide-to-node-js-express-and-postgresql-b679a6cd3ecc
51. Reactive Spring Boot and PostgREST: Simplifying Database-Driven API Development, 8월 3, 2025에 액세스, https://medium.com/@mbanaee61/reactive-spring-boot-and-postgrest-simplifying-database-driven-api-development-e2e20d29b141
52. Building REST APIs with PostgreSQL: A Practical Guide to Scalable, and Secure API Development - Liam ERD, 8월 3, 2025에 액세스, https://liambx.com/blog/building-rest-apis-postgresql-practical-guide
53. Comparison of Apache AGE, PostGraphile, and Hasura | Blog, 8월 3, 2025에 액세스, https://age.apache.org/blog/2024-04-23-comparison-of-apache-age-postgraphile-and-hasura/
54. Hasura vs. DreamFactory: A Comprehensive Comparison, 8월 3, 2025에 액세스, https://blog.dreamfactory.com/hasura-vs-dreamfactory
55. Create GraphQL APIs on PostgreSQL in 2 minutes - Hasura, 8월 3, 2025에 액세스, https://hasura.io/graphql/database/postgresql
56. Postgrest alternative - Fusio, 8월 3, 2025에 액세스, https://www.fusio-project.org/comparison/postgrest
57. A year after we chose to go with PostGraphile over Hasura in production, 8월 3, 2025에 액세스, https://emergence-engineering.com/blog/hasura-vs-postgraphile
58. REST API | Supabase Docs, 8월 3, 2025에 액세스, https://supabase.com/docs/guides/api
59. Supabase | The Postgres Development Platform., 8월 3, 2025에 액세스, https://supabase.com/
60. Understanding Your Backend Options in Flutter: Supabase vs. PostgreSQL - DhiWise, 8월 3, 2025에 액세스, https://www.dhiwise.com/post/understand-backend-options-in-flutter-supabase-vs-postgresql
61. Database | Supabase Docs, 8월 3, 2025에 액세스, https://supabase.com/docs/guides/database/overview
62. Using PostgREST for development, troubleshooting, and data endpoint - lucas dicioccio, 8월 3, 2025에 액세스, https://dicioccio.fr/role-of-postgrest.html
63. Community Tutorials - PostgREST 12.2 documentation, 8월 3, 2025에 액세스, https://docs.postgrest.org/en/v12/ecosystem.html
64. Community Tutorials - PostgREST 10.2 documentation, 8월 3, 2025에 액세스, https://docs.postgrest.org/en/v10/ecosystem.html

