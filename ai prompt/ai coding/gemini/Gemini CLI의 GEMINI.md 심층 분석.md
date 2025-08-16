# Gemini CLI의 GEMINI.md 심층 분석

### 섹션 1: 서론: 구성을 넘어서 - 에이전트의 기억 장치로서의 `GEMINI.md`

개발자 도구의 패러다임이 단순한 명령어 실행에서 지능적인 문제 해결로 전환됨에 따라, 터미널 환경은 새로운 국면을 맞이하고 있습니다. 이러한 변화의 중심에 Google의 Gemini CLI가 있으며, 이는 단순한 커맨드 라인 유틸리티를 넘어선 에이전트 기반의 개발 동반자입니다.1 Gemini CLI는 Gemini 2.5 Pro 모델의 강력한 성능과 1백만 토큰이라는 방대한 컨텍스트 창을 터미널에 직접 통합하여, 개발자가 자연어를 통해 코드를 이해하고, 파일을 조작하며, 동적으로 문제를 해결할 수 있는 환경을 제공합니다.2

#### 0.0.1 `GEMINI.md`: 에이전트의 시스템 프롬프트

이러한 에이전트 기능의 핵심에는 `GEMINI.md` 파일이 자리 잡고 있습니다. 이 파일은 단순한 설정 파일이 아니라, Gemini 에이전트의 행동을 규정하는 "지시적 컨텍스트(instructional context)" 또는 "기억(memory)"의 역할을 수행합니다.4 즉, `GEMINI.md`는 개발자가 에이전트의 페르소나, 행동 규칙, 프로젝트에 대한 사전 지식 등을 프로그래밍하는 수단입니다. 이를 통해 Gemini CLI는 수동적인 도구에서 벗어나, 능동적이고 컨텍스트를 인지하는 개발 파트너로 거듭납니다. Gemini CLI는 `GEMINI.md`와 같은 시스템 프롬프트 메커니즘을 통해 확장 가능하도록 설계되었으며, 이는 개발자가 에이전트와의 소통 방식을 자신의 필요에 맞게 조정할 수 있게 합니다.3

#### 0.0.2 선언적 에이전트 프로그래밍으로의 전환

`GEMINI.md`의 진정한 가치는 그것이 개발자와 AI 에이전트 간의 상호작용 방식을 근본적으로 바꾼다는 점에 있습니다. 전통적인 스크립팅이 특정 *행동*을 순서대로 나열하는 명령형(imperative) 방식이었다면, `GEMINI.md`는 원하는 *상태*를 선언하는 선언형(declarative) 접근법을 도입합니다. 개발자는 `GEMINI.md` 파일에 프로젝트의 목표, 코딩 표준, 아키텍처 패턴과 같은 규칙과 제약 조건을 선언합니다.6 그러면 Gemini 에이전트는 자신의 추론 엔진을 사용하여 이 선언된 상태에 도달하기 위해 필요한 구체적인 행동들을 스스로 결정합니다.

이러한 패러다임 전환의 중요성은 실제 사용자 경험을 통해 명확히 드러납니다. 한 사용자는 `GEMINI.md`에 "함수 a와 b를 생성하라"는 명령형 지시를 내렸을 때, 나중에 수동으로 추가한 함수 `c`를 에이전트가 삭제하려는 예상치 못한 제안을 하는 것을 발견했습니다.8 이는 에이전트가 

`GEMINI.md`를 프로젝트 상태의 유일한 "진실의 원천"으로 간주하고, 파일에 명시되지 않은 함수 `c`를 불필요한 요소로 판단했기 때문입니다.

반면, "코드베이스가 함수 a, b, c를 포함하도록 보장하라"는 선언형 지시로 바꾸자 문제는 해결되었습니다. 이 접근법은 에이전트의 목표를 고정된 작업 목록 실행에서 '원하는 상태 유지'로 전환시켰으며, 이는 프로젝트가 진화하는 방식과 훨씬 더 잘 부합합니다. 이러한 방식은 사용자가 YAML 파일에 원하는 상태를 정의하면 시스템이 그 상태에 도달하는 방법을 스스로 찾아내는 Kubernetes나 Terraform과 같은 선언적 시스템의 핵심 원리와 일치합니다.

결론적으로, `GEMINI.md`를 마스터한다는 것은 새로운 스크립팅 언어를 배우는 것이 아니라, 에이전트의 행동에 대해 선언적으로 사고하는 법을 배우는 것입니다. 이는 GitHub에서 완전한 `/persona` 관리 시스템에 대한 기능 요청이 제기된 것에서도 알 수 있듯이, 사용자들이 단순한 지시를 넘어 재사용 가능한 에이전트 행동을 정의하는 방향으로 자연스럽게 나아가고 있음을 보여줍니다.9 이 능력은 팀 기반의 AI 개발 환경에서 효율성을 극대화하는 핵심적인 고차원 기술이라 할 수 있습니다.

### 0.1  계층적 컨텍스트 시스템: AI 행동의 범위 지정

Gemini CLI는 개발자가 에이전트의 행동을 다양한 수준에서 정밀하게 제어할 수 있도록 다층적인 컨텍스트 로딩 메커니즘을 채택하고 있습니다. 이 시스템은 현재 작업 디렉토리에서 시작하여 상위 디렉토리를 거쳐 사용자의 홈 디렉토리까지 `GEMINI.md` 파일을 탐색하고, 발견된 모든 파일의 내용을 하나로 결합하여 에이전트의 최종 지침 세트를 구성합니다.4

#### 0.1.1 컨텍스트의 세 가지 계층

이 계층적 시스템은 다음과 같은 세 가지 주요 수준으로 구성됩니다.

1. **전역 컨텍스트 (`~/.gemini/GEMINI.md`):**

   - **목적:** 이 파일은 사용자의 모든 프로젝트에 걸쳐 적용되는 전반적인 규칙, 기본 에이전트 페르소나, 그리고 보편적인 선호도를 설정하기 위해 사용됩니다.4 사용자의 홈 디렉토리 내의 

     `.gemini` 폴더에 위치합니다.

   - **사용 예시:** "당신은 시니어 스태프 엔지니어입니다. 항상 간결하고 전문가 수준의 조언을 제공하세요. 모든 신규 프로젝트에는 TypeScript와 Jest를 선호합니다." 와 같이 전역적인 페르소나와 기술 스택 선호도를 정의할 수 있습니다.11

2. **프로젝트 컨텍스트 (`<project-root>/GEMINI.md`):**

   - **목적:** 특정 프로젝트의 아키텍처, 기술 스택, 팀 전체의 코딩 표준, 주요 파일 위치 등 프로젝트에 국한된 규칙을 정의합니다.4 이 파일은 일반적으로 프로젝트의 루트 디렉토리에서 

     `gemini init` 명령을 실행하여 생성됩니다.2

   - **사용 예시:** "이 프로젝트는 Tailwind CSS를 사용하는 Next.js 프로젝트입니다. 모든 컴포넌트는 React Hooks를 사용하는 함수형 컴포넌트여야 합니다. 상태 관리는 Zustand로 처리하며, 주요 파일은 `@/components`와 `@/lib`에 있습니다." 와 같이 프로젝트의 기술적 특성을 명시할 수 있습니다.6

3. **로컬 컨텍스트 (`<sub-directory>/GEMINI.md`):**

   - **목적:** 프로젝트 내의 특정 모듈, 컴포넌트 또는 복잡한 작업에 대한 매우 구체적인 지침을 제공하기 위해 사용됩니다.4 하위 디렉토리에 위치하여 가장 세분화된 제어를 가능하게 합니다.
   - **사용 예시:** `src/api/` 디렉토리 내에 위치한 `GEMINI.md` 파일에 "이 모듈의 모든 엔드포인트는 요청 본문에 대해 Joi 유효성 검사를 포함해야 하며, `../../lib/errors.ts`의 전역 오류 처리기를 사용해야 합니다." 와 같은 세부 규칙을 정의할 수 있습니다.12

#### 0.1.2 병합 및 우선순위

CLI는 이 세 계층에서 발견된 모든 `GEMINI.md` 파일의 내용을 병합하여 최종적인 지시적 컨텍스트를 형성합니다.4 여기서 중요한 규칙은 더 구체적인 파일(로컬)의 컨텍스트가 더 일반적인 파일(전역)의 컨텍스트를 보충하거나 덮어쓴다는 것입니다.12 이 우선순위 규칙 덕분에 개발자는 전역 규칙에 대한 예외를 두거나 특정 모듈에 대해서만 다른 행동을 지시하는 등 정밀한 제어가 가능해집니다.

#### 0.1.3 컨텍스트 스택 디버깅

에이전트가 어떤 지침을 따르고 있는지 정확히 파악하는 것은 매우 중요합니다. 이를 위해 Gemini CLI는 에이전트의 "기억"을 검사할 수 있는 내장 명령어를 제공합니다.

- `/memory show`: 현재 로드된 모든 `GEMINI.md` 파일의 병합된 전체 내용을 표시합니다. 이를 통해 사용자는 에이전트가 현재 어떤 "생각"을 하고 있는지 정확히 확인할 수 있습니다.12
- `/memory refresh`: 활성 세션 중에 `GEMINI.md` 파일을 수정한 후, 변경 사항을 즉시 적용하기 위해 모든 컨텍스트 파일을 다시 로드합니다.12

이러한 계층적 구조와 디버깅 도구는 `GEMINI.md`를 단순한 지침 파일을 넘어, 에이전트의 행동을 체계적으로 설계하고 관리할 수 있는 강력한 시스템으로 만들어 줍니다.

**표 1: 계층적 컨텍스트 로딩 규칙**

| 범위                   | 파일 경로                   | 의도된 목적                         | 우선순위 |
| ---------------------- | --------------------------- | ----------------------------------- | -------- |
| **전역 (Global)**      | `~/.gemini/GEMINI.md`       | 사용자 전반의 페르소나 및 규칙 설정 | 낮음     |
| **프로젝트 (Project)** | `<project-root>/GEMINI.md`  | 프로젝트별 표준 및 아키텍처 정의    | 중간     |
| **로컬 (Local)**       | `<sub-directory>/GEMINI.md` | 모듈 또는 컴포넌트별 세부 지침      | 높음     |

### 0.2  효과적인 `GEMINI.md` 파일 작성법: 프롬프트 엔지니어링 실전 가이드

이 섹션에서는 효과적인 `GEMINI.md` 파일을 작성하기 위한 구체적이고 실용적인 방법을 다룹니다. 기본적인 내용 구성부터 정교한 프롬프트 엔지니어링 전략까지, 에이전트의 성능을 극대화하는 방법을 단계별로 안내합니다.

#### 0.2.1 핵심 콘텐츠 청사진

효과적인 `GEMINI.md` 파일은 명확한 구조와 구체적인 지침을 담고 있어야 합니다. 다음은 주요 지침 카테고리에 대한 예시와 모범 사례입니다.

- **프로젝트 목표 및 아키텍처:** "이 프로젝트는 마이크로서비스 기반의 전자상거래 백엔드입니다. `auth-service`는 JWT를 처리하고, `product-service`는 재고를 관리하며, 두 서비스는 RabbitMQ를 통해 통신합니다. 파일 변경 전에는 항상 전체 아키텍처를 요약하여 설명하세요." 6
- **기술 스택 및 의존성:** "이 프로젝트는 Poetry로 의존성을 관리하는 Python 3.11 프로젝트입니다. `pyproject.toml`에 명시된 라이브러리만 사용하세요. 동시성 작업에는 `asyncio`를 선호합니다." 6
- **코딩 표준 및 스타일 가이드:** "Google TypeScript 스타일 가이드를 준수하세요. 2칸 들여쓰기를 사용하고, 모든 함수에는 JSDoc 주석을 포함해야 합니다. 기본 내보내기(default export)는 사용하지 마세요." 4
- **에이전트 페르소나 및 어조:** "당신은 유용하지만 간결한 코딩 어시스턴트입니다. 불필요한 대화체 표현은 사용하지 말고, 바로 본론으로 들어가세요. 코드를 제공할 때, 질문받지 않는 한 기능에 대해 설명하지 마세요." 15
- **도구 사용 정책:** "커밋을 요청받으면, 커밋 메시지는 Conventional Commits 명세를 따라야 합니다. 새 파일을 작성하기 전에는 사용자에게 위치를 확인받으세요." 17

#### 0.2.2 지시의 기술: 선언적 프롬프트 vs. 명령적 프롬프트

앞서 언급했듯이, `GEMINI.md`를 작성할 때는 선언적 접근법이 매우 중요합니다. 이 개념을 더 깊이 이해하기 위해 실제 사례를 살펴보겠습니다.

- **사례 연구: 유령 코드 삭제 사건:** 한 사용자가 `GEMINI.md`에 "함수 a와 b를 생성하라"는 명령적 지시를 포함했을 때, 에이전트는 나중에 수동으로 추가된 함수 `c`를 삭제하자는 제안을 했습니다.8 이는 에이전트가 

  `GEMINI.md`를 프로젝트의 최종 상태에 대한 '선언'으로 해석하고, 명시되지 않은 함수 `c`를 제거해야 할 대상으로 판단했기 때문입니다.

- **해결책과 분석:** 이 문제에 대한 해결책은 지시를 선언적으로 바꾸는 것이었습니다. "코드베이스가 함수 a, b, c를 포함하도록 보장하라"는 프롬프트는 에이전트의 목표를 '고정된 작업 수행'에서 '원하는 상태 유지'로 전환시킵니다. 이 방식은 프로젝트가 시간이 지남에 따라 유기적으로 변화하는 실제 개발 과정과 훨씬 더 잘 부합합니다.

- **모범 사례:** `GEMINI.md`를 작성할 때는 "마치 쿠버네티스 YAML을 적용하는 것처럼 생각하라"는 조언을 따르는 것이 좋습니다.8 또한, 에이전트가 코드를 변경할 때 

  `GEMINI.md` 파일 자체도 업데이트하도록 지시하여, 시스템이 스스로를 문서화하는 선순환 구조를 만드는 것도 효과적인 전략입니다.

#### 0.2.3 LLM 이해를 돕는 구문 및 서식

`GEMINI.md`는 마크다운 파일이므로, 그 구조를 활용하여 LLM의 주의를 유도하고 지침의 중요도를 전달할 수 있습니다. 이는 프롬프트 엔지니어링의 핵심 원리 중 하나입니다.19

- **헤더 (`#`, `##`):** 지침의 명확한 계층 구조를 만듭니다.
- **목록 (글머리 기호/번호 매기기):** 복잡한 규칙이나 순차적인 단계를 나누어 설명합니다.
- **코드 블록 (```):** 에이전트가 따라야 할 코드 패턴이나 명령어 구문의 명확한 예시를 제공합니다.
- **구분선 (`---`):** 지침의 여러 섹션을 명확하게 분리하여 컨텍스트 혼동을 방지합니다.11

이러한 서식들을 전략적으로 사용하면, 에이전트는 지침의 의도를 더 정확하게 파악하고 일관된 결과를 생성할 수 있습니다.

**표 2: `GEMINI.md` 콘텐츠 청사진 예시**

| 프로젝트 유형           | 주요 포함 내용                                               |
| ----------------------- | ------------------------------------------------------------ |
| **TypeScript 웹 앱**    | 프레임워크: React/Next.js 명시스타일링: Tailwind CSS 사용 규칙상태 관리: Zustand/Redux 패턴테스트: Jest/Playwright 테스트 작성 가이드라인린팅: ESLint/Prettier 규칙 준수 |
| **Python 데이터 과학**  | 패키지 관리자: Poetry/Pip 사용법핵심 라이브러리: Pandas, NumPy, Scikit-learn 사용 패턴데이터 소스: 데이터 파일 위치 및 접근 방식노트북 규칙: Jupyter 노트북 작성 컨벤션 |
| **Android 앱 (Kotlin)** | 대상 SDK: 타겟 및 컴파일 SDK 버전 명시아키텍처: MVVM 패턴 준수핵심 라이브_S: Jetpack Compose, Retrofit, Room 사용법의존성: build.gradle.kts 파일 스캔 및 업데이트 확인 지시 20 |
| **Go 마이크로서비스**   | Go 버전: 사용할 Go 버전 명시핵심 모듈: 주요 모듈 및 패키지 설명데이터베이스: 데이터베이스 상호작용 및 ORM 사용 패턴오류 처리: 표준 오류 처리 관행 정의 |

### 0.3  고급 기술 및 에이전트 패턴

`GEMINI.md`는 정적인 프로젝트 구성을 넘어, 복잡하고 동적인 워크플로우를 구현하는 데 사용될 수 있습니다. 이 섹션에서는 `GEMINI.md`를 활용한 정교한 에이전트 패턴들을 탐구합니다.

#### 0.3.1 자체 문서를 통한 "유동적 기억" 생성

`GEMINI.md`의 가장 혁신적인 활용법 중 하나는 에이전트가 자신의 추론 과정을 스스로 기록하게 만드는 것입니다. 이 기술은 에이전트를 상태 없는(stateless) 도구에서 지속적이고 감사 가능한 기억을 가진 파트너로 변모시킵니다.8

- **프롬프트 예시:** "`GEMINI.md`에 다음과 같이 지시할 수 있습니다: '기술적 결정(예: 라이브러리 선택)을 내릴 때, 그 이유를 설명하고 `AI_REASONING.md` 파일에 추가하세요. 다음 호출 시에는 이 파일을 읽어 일관된 논리를 유지하세요.'"
- **의의:** 이 패턴을 통해 에이전트의 결정 과정을 버전 관리 시스템으로 추적하고 팀원들과 공유할 수 있게 됩니다. 이는 일회성 세션을 넘어선 장기적인 컨텍스트 유지를 가능하게 하는, 사용자가 직접 만들어낸 강력한 기억 메커니즘입니다.

#### 0.3.2 즉석 RAG (검색 증강 생성) 활용

`GEMINI.md` 파일 내에 외부 문서나 API 명세서의 URL을 포함시켜, 에이전트가 실시간 외부 정보에 기반하여 응답하도록 만들 수 있습니다.8

- **프롬프트 예시:** "우리 회사의 인증 흐름을 이해하려면, 먼저 `https://docs.our-auth-provider.com/v2/api-spec`에 있는 문서를 읽어야 합니다. 모든 로그인 관련 기능을 구현할 때는 이 문서를 참조하세요."
- **의의:** 이 방법은 복잡한 벡터 데이터베이스 설정 없이도 경량의 RAG(Retrieval-Augmented Generation)를 구현하는 효과적인 방법입니다. 에이전트는 더 이상 자신의 훈련 데이터나 로컬 코드베이스에만 의존하지 않고, 최신 외부 정보를 바탕으로 더 정확하고 신뢰도 높은 결과를 생성할 수 있습니다.

#### 0.3.3 교차 에이전트 오케스트레이션

`GEMINI.md`와 같은 컨텍스트 파일은 여러 AI 에이전트로 구성된 시스템을 조율하는 원시적인 오케스트레이션 레이어로 기능할 수 있습니다. 한 사용자의 창의적인 활용 사례는 이 가능성을 명확하게 보여줍니다.

이 사용자는 컨텍스트 창이 작은 대신 특정 작업에 강점이 있다고 판단한 Claude CLI와, 방대한 컨텍스트 창으로 전체 코드베이스 분석에 유리한 Gemini CLI를 함께 사용하고자 했습니다.14 이를 위해 그는 

`CLAUDE.md` 파일에 다음과 같은 지침을 추가했습니다: "코드베이스의 넓은 부분을 분석해야 할 경우, 비대화형 모드에서 `gemini -p` 명령을 실행하여 요약 정보를 얻으세요." 21

이 사례는 컨텍스트 파일이 단순히 정보를 제공하는 것을 넘어, AI 에이전트 간의 상호작용 프로토콜을 정의하는 데 사용될 수 있음을 보여주는 획기적인 발견입니다. `CLAUDE.md`는 하나의 AI가 다른 AI에게 작업을 위임하도록 프로그래밍하는 'API 문서' 역할을 한 것입니다. 이는 컨텍스트 파일이 향후 버전 관리 가능하고 사용자 친화적인 다중 에이전트 시스템을 구축하는 데 핵심적인 역할을 할 수 있음을 시사합니다.

#### 0.3.4 MCP (Model Context Protocol) 서버 행동 정의

Gemini CLI는 MCP를 통해 외부 도구와 통합하여 기능을 확장할 수 있습니다. 여기서 `GEMINI.md`와 `settings.json` 파일은 각기 다른 역할을 수행합니다.

- `~/.gemini/settings.json`: 이 파일은 MCP 서버를 *구성*합니다. 즉, CLI에게 어떤 MCP 서버가 존재하며 어떻게 실행해야 하는지를 알려줍니다.11
- `GEMINI.md`: 이 파일은 에이전트에게 해당 서버가 제공하는 도구들을 *언제, 어떻게* 사용해야 하는지를 *지시*합니다.22
- **`GEMINI.md` 지침 예시:** "작업이 GitHub와 관련될 경우, `github` MCP 도구를 사용하세요. 새로운 기능을 생성할 때는 먼저 `github.create_branch`를 사용하고, 코드를 작성한 후, 마지막으로 변경 사항 요약과 함께 `github.create_pull_request`를 사용하세요." 25

이처럼 `GEMINI.md`는 MCP 도구의 단순한 호출을 넘어, 여러 도구를 조합하여 복잡한 워크플로우를 자동화하는 스크립트 역할을 수행할 수 있습니다.

### 0.4  비교 분석: `GEMINI.md` vs. `CLAUDE.md`

Gemini CLI와 Anthropic의 Claude Code는 현재 가장 주목받는 두 개의 에이전트 기반 CLI 도구입니다. 두 도구 모두 파일 기반 컨텍스트 시스템을 채택하고 있으며, 이를 비교 분석함으로써 각 도구의 설계 철학과 실용적인 차이점을 이해할 수 있습니다.

#### 0.4.1 핵심 철학 및 기능적 유사성

두 도구는 놀라울 정도로 유사한 핵심 기능을 공유합니다. Gemini CLI의 `GEMINI.md`와 Claude Code의 `CLAUDE.md`는 모두 계층적 구조를 가진 마크다운 파일을 사용하여 에이전트에게 지속적인 컨텍스트를 제공하는 중심 메커니즘으로 작동합니다.4 또한, 두 시스템 모두 전역 사용자 수준 파일과 프로젝트 수준 파일을 지원하여 컨텍스트의 범위를 조절할 수 있게 합니다.4 이러한 공통된 패턴은 버전 관리가 가능하고 인간이 읽을 수 있는(human-readable) 컨텍스트 파일이 에이전트 기반 CLI 분야에서 핵심적인 기능으로 자리 잡았음을 시사합니다.29

#### 0.4.2 차별점 및 생태계의 영향

유사점에도 불구하고, 두 도구는 각자의 생태계와 모델 특성에 따라 뚜렷한 차이점을 보입니다.

- **도구 통합 및 확장성:** Gemini CLI는 MCP(Model Context Protocol)를 개방형 표준으로 강력하게 내세우며 확장성을 강조합니다.11 반면, Claude Code는 사용자 정의 슬래시 명령어(

  `/`)와 자체 도구 생태계와의 긴밀한 통합에 더 중점을 두는 경향이 있습니다.18

- **기반 모델의 강점:** 사용자들의 평가에 따르면, 기반 LLM의 강점이 다릅니다. Gemini 2.5 Pro는 방대한 1백만 토큰 컨텍스트 창과 폭넓은 지식 기반으로 유명하여, `GEMINI.md`에 광범위한 아키텍처 개요를 제공하는 데 이상적입니다.3 반면, Claude 모델은 강력한 추론, 계획 수립, 도구 사용 능력에서 높은 평가를 받아, 

  `CLAUDE.md`는 복잡한 다단계 워크플로우나 테스트 주도 개발(TDD) 프로세스를 정의하는 데 더 적합할 수 있습니다.32

- **사용자 경험(UX) 및 성숙도:** 사용자들은 Gemini CLI가 강력하지만 초기 버전으로서 때때로 장황하거나 버그가 있다고 평가합니다.16 반면 Claude Code는 "생각 모드(thinking mode)"나 더 정교한 권한 확인 프롬프트와 같은 기능 덕분에 더 "프리미엄"하고 세련된 경험을 제공한다는 평이 많습니다.34 이러한 UX의 차이는 각 

  `.md` 파일에 작성되는 지침의 스타일에 영향을 미칠 수 있습니다 (예: `GEMINI.md`에 더 명시적인 오류 처리 지침 포함).

- **비용 및 접근성:** Gemini CLI는 관대한 무료 티어를 제공하여 사용자들이 `GEMINI.md`를 자유롭게 실험해 볼 수 있도록 장려합니다.1 반면, Claude Code는 유료 구독 모델에 기반하므로, 사용자들은 

  `CLAUDE.md`를 작성할 때 비용 효율성을 고려한 더 집중적인 프롬프트 엔지니어링을 수행할 가능성이 있습니다.36

이러한 차이점들은 어느 한쪽이 절대적으로 우월하다기보다는, 개발자의 특정 요구사항과 프로젝트의 성격에 따라 어떤 도구가 더 적합한지를 결정하는 중요한 요소가 됩니다.

**표 3: 기능 비교 매트릭스: `GEMINI.md` vs. `CLAUDE.md`**

| 기능                | Gemini CLI (`GEMINI.md`)                                     | Claude Code (`CLAUDE.md`)                                    |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **계층적 컨텍스트** | 전역, 프로젝트, 로컬의 3계층 구조 지원 4                     | 전역 및 프로젝트 수준의 2계층 구조 지원 18                   |
| **도구 정의**       | `settings.json`에서 MCP 서버를 구성하고, `GEMINI.md`에서 사용법을 지시 23 | `.claude/commands/` 디렉토리에 마크다운 파일로 사용자 정의 슬래시 명령어 생성 18 |
| **프롬프팅 스타일** | 선언적 상태 정의를 강조. "Ensure X"와 같은 구문이 효과적 8   | TDD, 계획 수립 등 다단계 워크플로우를 명시적으로 지시. "think hard"와 같은 키워드 지원 34 |
| **확장성 모델**     | 개방형 표준인 MCP(Model Context Protocol)를 통한 확장성 강조 30 | 자체적인 도구 생태계 및 사용자 정의 명령어를 통한 긴밀한 통합에 중점 31 |
| **비용 모델**       | 개인 Google 계정으로 관대한 무료 티어 제공 (일 1,000회 요청) 3 | 유료 구독 플랜(Pro 이상) 필요 36                             |

### 0.5  전략적 권장 사항 및 향후 전망

이 보고서의 분석 결과를 바탕으로, `GEMINI.md`를 효과적으로 도입하고 활용하기 위한 전략적 권장 사항을 제시하고, 에이전트 기반 CLI의 미래를 전망합니다.

#### 0.5.1 도입을 위한 모범 사례

`GEMINI.md`의 잠재력을 최대한 활용하기 위해 다음의 단계별 접근법을 권장합니다.

1. **전역 파일로 시작하라:** 먼저 `~/.gemini/GEMINI.md` 파일에 개인적인 페르소나, 선호하는 기술 스택, 전반적인 코딩 스타일 등 자신만의 기본 설정을 확립하십시오. 이는 모든 프로젝트에서 일관된 기반을 제공합니다.
2. **모든 프로젝트에 `gemini init`을 사용하라:** 새로운 리포지토리를 시작할 때마다 루트 디렉토리에서 `gemini init`을 실행하여 프로젝트별 `GEMINI.md` 파일을 생성하십시오. 이 파일에 해당 프로젝트의 고유한 아키텍처, 의존성, 팀 규칙을 정의하여 컨텍스트를 명확히 해야 합니다.
3. **명령이 아닌, 상태를 선언하라:** "X를 하라"는 명령형 지시 대신 "Y 상태를 보장하라"는 선언형 지시를 사용하십시오. 이는 프로젝트의 예기치 않은 변경을 방지하고 에이전트의 행동을 더 예측 가능하게 만듭니다.
4. **`GEMINI.md`를 코드로 취급하고 반복적으로 개선하라:** `GEMINI.md` 파일을 버전 관리 시스템(Git 등)으로 추적하고, 변경 사항을 팀과 함께 검토하며, 에이전트의 실제 성능에 기반하여 지침을 지속적으로 개선하십시오. 에이전트의 현재 컨텍스트가 의도와 다른 경우, `/memory show` 명령을 사용하여 디버깅해야 합니다.
5. **생태계를 포용하라:** `GEMINI.md`에 단순히 코드 관련 컨텍스트만 제공하는 데 그치지 마십시오. CI/CD 파이프라인, MCP 서버, 그리고 다른 통합 도구들을 에이전트가 어떻게 사용해야 하는지에 대한 지침을 포함시켜 워크플로우 자동화의 범위를 넓혀야 합니다.

#### 0.5.2 알려진 한계 및 해결 방안

`GEMINI.md`는 강력한 도구이지만, 현재 몇 가지 한계점이 보고되고 있습니다.

- **성능의 "기복(Spikiness)":** 사용자들은 Gemini 모델의 성능이 일관되지 않을 수 있다고 보고합니다.40 이에 대한 해결책은 

  `GEMINI.md`에 매우 구체적이고 명확한 선언적 프롬프트를 제공하여 에이전트의 행동 범위를 좁히고 모호성을 줄이는 것입니다.

- **요청 제한(Rate Limiting) 버그:** CLI가 무한 요청 루프에 빠져 무료 티어의 사용 한도를 초과하는 버그가 보고되었습니다.35 현재로서는 이 현상이 발생할 때 프로세스를 강제 종료하고 다시 시작하는 것이 유일한 해결책입니다. 이 버그를 유발할 수 있는 지나치게 복잡하거나 모호한 프롬프트를 피하는 것이 좋습니다.

- **프롬프트 엔지니어링의 중요성:** `GEMINI.md`의 효과는 전적으로 사용자의 프롬프트 엔지니어링 능력에 달려 있습니다. 이 파일은 마법의 해결책이 아니며, 원하는 결과를 얻기 위해서는 지속적인 실험과 개선이 필요합니다.4

#### 0.5.3 앞으로의 길: 에이전트 제어의 진화

`GEMINI.md`와 같은 파일 기반 컨텍스트 시스템은 에이전트 제어 기술의 시작에 불과합니다. 향후 다음과 같은 방향으로 진화할 것으로 예상됩니다.

- **공식적인 페르소나 관리:** GitHub에서 제기된 `/persona create`, `/persona list`와 같은 완전한 페르소나 관리 시스템에 대한 기능 요청은 사용자들이 이미 유기적으로 수행하고 있는 작업을 공식화하는 자연스러운 다음 단계가 될 것입니다.9
- **더 긴밀한 IDE 및 Git 통합:** 경쟁 도구인 Claude Code에서 이미 제공하는 심층적인 Git 통합(예: `create_pr` 도구) 및 세션 관리 기능은 Gemini CLI의 로드맵에 포함될 가능성이 높습니다.10
- **오케스트레이션의 미래:** 궁극적으로 `GEMINI.md`와 같은 파일은 단일 에이전트에 대한 지침을 넘어, 다중 에이전트 오케스트레이션을 위한 사용자 친화적이고 버전 관리가 가능한 "지휘자의 악보" 역할을 하게 될 것입니다. 미래는 전문화된 에이전트 팀이 어떻게 협력하는지를 정의하는 데 있으며, `.md` 파일은 그 중심에서 핵심적인 역할을 수행할 것입니다.

#### **참고 자료**

1. Gemini CLI Tutorial Series - Medium, accessed July 12, 2025, https://medium.com/google-cloud/gemini-cli-tutorial-series-77da7d494718
2. Gemini CLI: The AI-Powered Command Line Revolution for Developers - DEV Community, accessed July 12, 2025, https://dev.to/shahidkhans/gemini-cli-the-ai-powered-command-line-revolution-for-developers-a7e
3. Google announces Gemini CLI: your open-source AI agent, accessed July 12, 2025, https://blog.google/technology/developers/introducing-gemini-cli-open-source-ai-agent/
4. Gemini CLI Tutorial Series — Part 2 : Gemini CLI Command line ..., accessed July 12, 2025, https://medium.com/google-cloud/gemini-cli-tutorial-series-part-2-gemini-cli-command-line-parameters-e64e21b157be
5. Everything You Need to Know About Google Gemini CLI: Features, News, and Expert Insights - TS2 Space, accessed July 12, 2025, https://ts2.tech/en/everything-you-need-to-know-about-google-gemini-cli-features-news-and-expert-insights/
6. Practical Tips for Using Gemini CLI in Real Projects - Momen, accessed July 12, 2025, https://momen.app/blogs/practical-tips-for-using-gemini-cli-in-real-projects/
7. google-gemini/gemini-cli-action - GitHub, accessed July 12, 2025, https://github.com/google-gemini/gemini-cli-action
8. Vibe coding my first Chrome Extension with Gemini CLI ! | by Riccardo Carlesso - Medium, accessed July 12, 2025, https://medium.com/google-cloud/vibe-coding-my-first-chrome-extension-with-gemini-cli-da5630d00434
9. Add a conversational persona management system to Gemini CLI · Issue #3777 - GitHub, accessed July 12, 2025, https://github.com/google-gemini/gemini-cli/issues/3777
10. Feature Enhancements for Gemini CLI · Issue #1585 - GitHub, accessed July 12, 2025, https://github.com/google-gemini/gemini-cli/issues/1585
11. Give Gemini CLI the Ability to Generate Images and Video, Work with GitHub Repos, and Use Other Tools | by Dazbo (Darren Lester) | Google Cloud - Medium, accessed July 12, 2025, https://medium.com/google-cloud/give-gemini-cli-the-ability-to-generate-images-and-video-work-with-github-repos-and-use-other-482172571f99
12. Use agentic chat as a pair programmer | Gemini Code Assist - Google for Developers, accessed July 12, 2025, https://developers.google.com/gemini-code-assist/docs/use-agentic-chat-pair-programmer
13. OpenAI Platform, accessed July 12, 2025, https://platform.openai.com/docs/guides/prompt-engineering/tactic-ask-the-model-to-adopt-a-persona
14. gemini.md to teach Claude to use google's gemini cli as his tool ..., accessed July 12, 2025, https://gist.github.com/steipete/20ed650822f1ac835144bfd328c872b7
15. Incorporate similar instructions to Gemini CLI · community · Discussion #164230 - GitHub, accessed July 12, 2025, https://github.com/orgs/community/discussions/164230
16. Gemini CLi vs. Claude Code : The better coding agent - Composio, accessed July 12, 2025, https://composio.dev/blog/gemini-cli-vs-claude-code-the-better-coding-agent
17. Quickstart - Anthropic API, accessed July 12, 2025, https://docs.anthropic.com/en/docs/claude-code/quickstart
18. Claude Code Implementation - Empathy First Media, accessed July 12, 2025, https://empathyfirstmedia.com/claude-code-implementation/
19. Best practices for prompt engineering with the OpenAI API, accessed July 12, 2025, https://help.openai.com/en/articles/6654000-best-practices-for-prompt-engineering-with-the-openai-api
20. Free Top-rated Agent AI: Boost Android Development with the Gemini CLI in 5 Minutes | by Mickco Lai | ProAndroidDev, accessed July 12, 2025, https://proandroiddev.com/boosting-android-development-with-gemini-cli-quick-start-in-5-minute-e3ca01cb90fb
21. Gemini CLI is awesome! But only when you make Claude Code use it as its bitch. - Reddit, accessed July 12, 2025, https://www.reddit.com/r/ChatGPTCoding/comments/1lm3fxq/gemini_cli_is_awesome_but_only_when_you_make/
22. Gemini CLI Full Tutorial - DEV Community, accessed July 12, 2025, https://dev.to/proflead/gemini-cli-full-tutorial-2ab5
23. Complete Gemini CLI Setup Guide for Your Terminal - HackerNoon, accessed July 12, 2025, https://hackernoon.com/complete-gemini-cli-setup-guide-for-your-terminal
24. /mcp don't show installed MCP servers · Issue #3813 · google-gemini/gemini-cli - GitHub, accessed July 12, 2025, https://github.com/google-gemini/gemini-cli/issues/3813
25. Gemini CLI + MCP: Complete Install in Minutes (Tutorial) - YouTube, accessed July 12, 2025, https://www.youtube.com/watch?v=Gm8XjcNQ1Vk
26. Gemini CLI + MCP Documentation Generator - GitHub, accessed July 12, 2025, https://github.com/Paraskevi-KIvroglou/Gemini-CLI-GitHub-MCP
27. How to Install Claude Code - AI-Powered Terminal Coding Assistant, accessed July 12, 2025, https://jsonobject.hashnode.dev/how-to-install-claude-code-ai-powered-terminal-coding-assistant
28. How to Master Claude MD Files in Claude Code: The Developer's Complete Guide to AI-Powered Documentation - Empathy First Media, accessed July 12, 2025, https://empathyfirstmedia.com/claude-md-file-claude-code/
29. Inspiration from Claude Code CLI Prompts | by 洪群崴 - Medium, accessed July 12, 2025, https://medium.com/@william31525/inspiration-from-claude-code-cli-prompts-dd460b6cb54e
30. google-gemini/gemini-cli: An open-source AI agent that brings the power of Gemini directly into your terminal. - GitHub, accessed July 12, 2025, https://github.com/google-gemini/gemini-cli
31. zebbern/claude-code-guide: Full guide on claude tips and tricks and how you can optimise your claude code the best & strive to find every command possible even hidden ones! - GitHub, accessed July 12, 2025, https://github.com/zebbern/claude-code-guide
32. Gemini CLI Team AMA : r/Bard - Reddit, accessed July 12, 2025, https://www.reddit.com/r/Bard/comments/1lms62v/gemini_cli_team_ama/
33. Gemini 2.5: Our most intelligent AI model - Google Blog, accessed July 12, 2025, https://blog.google/technology/google-deepmind/gemini-model-thinking-updates-march-2025/
34. Claude Code: Best practices for agentic coding - Anthropic, accessed July 12, 2025, https://www.anthropic.com/engineering/claude-code-best-practices
35. Issue #1881 · google-gemini/gemini-cli - Rate Limit Exceeded - GitHub, accessed July 12, 2025, https://github.com/google-gemini/gemini-cli/issues/1881
36. GEMINI CLI NOT RECONGIZING GEMINI CODE ASSIST SUBSCRIPTION - RATE LIMITS AND REVERTS BACK TO FLASH 2.5 · Issue #2711 - GitHub, accessed July 12, 2025, https://github.com/google-gemini/gemini-cli/issues/2711
37. Google accidentally published the Gemini CLI blog post. The release is probably soon. : r/Bard - Reddit, accessed July 12, 2025, https://www.reddit.com/r/Bard/comments/1ljzqxc/google_accidentally_published_the_gemini_cli_blog/
38. Compare the Top 5 Agentic CLI Coding Tools - AI - GetStream.io, accessed July 12, 2025, https://getstream.io/blog/agentic-cli-tools/
39. The Gemini CLI Github is LIVE : r/Bard - Reddit, accessed July 12, 2025, https://www.reddit.com/r/Bard/comments/1lk5i3c/the_gemini_cli_github_is_live/
40. How do I Monitor Gemini CLI Usage - Google Help, accessed July 12, 2025, https://support.google.com/gemini/thread/355460179/how-do-i-monitor-gemini-cli-usage?hl=en

