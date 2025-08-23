# Gemini CLI 완벽 사용 가이드


Gemini CLI는 단순히 터미널에서 AI와 채팅하는 도구가 아니다. 이것은 개발자의 터미널 환경에 상주하며, 스스로 생각하고 행동하는 'AI 에이전트'다.1 기존의 AI 도구들이 웹 브라우저 안에서 답변을 생성하는 데 그쳤다면, Gemini CLI는 개발자의 가장 핵심적인 작업 공간인 터미널에 직접 통합되어 코드베이스를 이해하고, 파일을 수정하며, 명령어를 실행하는 등 실제적인 작업을 수행한다.1 이 가이드는 Gemini CLI의 핵심 개념부터 실전 활용까지 모든 것을 다룬다.


Gemini CLI의 가장 큰 차별점은 '생각하고 행동하는(Reason and Act, ReAct)' 능력에 있다.4 사용자가 "이 버그를 수정해줘"와 같이 높은 수준의 목표를 제시하면, Gemini CLI는 다음과 같은 ReAct 루프를 통해 작업을 자율적으로 수행한다.4

1. **Reason (추론):** 사용자의 요청과 현재 프로젝트의 컨텍스트를 분석하여 문제 해결을 위한 계획을 수립한다.
2. **Act (행동):** 계획의 첫 단계를 실행하기 위해 파일 읽기, 웹 검색, 셸 명령어 실행 등 내장된 도구(tool)를 사용한다.
3. **Observe (관찰):** 도구 실행 결과를 관찰하고, 그 결과를 바탕으로 초기 계획이 여전히 유효한지, 수정이 필요한지를 판단한다.
4. **Repeat (반복):** 목표가 달성될 때까지 이 과정을 반복한다.

이러한 작동 방식은 개발자가 AI에게 작업의 '방법'을 일일이 지시하는 기존의 방식에서 벗어나, '목표'를 위임하는 새로운 개발 패러다임을 제시한다.2 개발자는 "API 키를 찾아서 환경 변수로 바꾸는 코드를 작성해줘"라고 명령하는 대신, "하드코딩된 API 키를 환경 변수에서 읽어오도록 리팩토링해줘"라고 작업을 위임할 수 있다. 그러면 에이전트는 스스로 파일을 검색하고(`grep`), 코드를 수정하며(`edit`), 필요하다면 테스트까지 실행하는(`shell`) 모든 과정을 알아서 처리한다. 이는 개발자의 인지 부하를 획기적으로 줄여, 문제 해결의 본질에 더 집중할 수 있게 해준다.


Gemini CLI가 강력한 에이전트로서 기능할 수 있는 배경에는 다음과 같은 핵심 특징들이 있다.

- **100만 토큰 컨텍스트 창:** Gemini 2.5 Pro 모델을 기반으로, 약 150만 단어 또는 750페이지 분량의 방대한 정보를 한 번에 처리할 수 있다.1 이는 대규모 프로젝트의 전체 코드베이스를 컨텍스트에 넣고 작업하는 것을 가능하게 하며, 대화 도중 이전 지시사항을 '잊어버리는' 문제를 근본적으로 해결한다.2
- **로컬 파일 시스템 접근:** Gemini CLI는 사용자의 컴퓨터에서 직접 실행된다. 이는 코드, 민감한 데이터, 비즈니스 정보가 외부 서버로 전송되지 않아 보안과 개인정보 보호 측면에서 큰 이점을 제공한다.2 또한 파일 읽기, 쓰기, 수정, 셸 명령어 실행 등 로컬 환경과 직접 상호작용하며 실제 개발 작업을 수행할 수 있다.6
- **오픈소스 (Apache 2.0):** 소스 코드가 GitHub 저장소 `google-gemini/gemini-cli`에 공개되어 있어 누구나 작동 방식을 투명하게 검토하고 보안을 확인할 수 있다.1 버그 리포트, 기능 제안, 코드 기여 등 전 세계 개발자 커뮤니티의 참여를 통해 지속적으로 발전하고 있다.1


`gemini-cli`라는 동일한 이름으로, 현재는 개발이 중단되고 보관(archived)된 Go 언어 기반의 다른 프로젝트(`eliben/gemini-cli`)가 존재한다.13 해당 프로젝트는 단순히 Gemini API를 호출하는 간단한 클라이언트이며, 이 가이드에서 다루는 공식 Google Gemini CLI와는 전혀 다른 도구다. 이 가이드에서 설명하는 것은 Node.js 기반의 공식 AI 에이전트이므로 혼동하지 않도록 주의해야 한다.


Gemini CLI를 사용하기 위한 준비 과정은 간단하지만, 몇 가지 주의할 점이 있다. 운영체제와 환경에 맞춰 순조롭게 첫발을 내디딜 수 있도록 단계별로 안내한다.


Gemini CLI는 Node.js v18 또는 v20 이상 버전에서 실행된다.9 설치를 진행하기 전에 터미널에서 `node -v` 명령어를 실행하여 현재 설치된 버전을 반드시 확인해야 한다.

특히 Ubuntu 24.04 LTS와 같은 일부 운영체제는 기본적으로 구버전인 v18을 제공하는 경우가 있다.16 이 경우, Node Version Manager(nvm)나 NodeSource와 같은 도구를 사용하여 최신 버전의 Node.js로 업그레이드해야 한다. 

`nvm`을 사용하는 것이 여러 버전을 관리하기에 용이하여 권장된다.14


Gemini CLI를 설치하는 방법은 크게 세 가지이며, 각기 다른 장단점을 가진다.

- **npx (설치 없이 실행):** 가장 간단하고 권장되는 방법이다. 시스템에 별도의 파일을 설치하지 않고 실행 시점에 최신 버전을 다운로드하여 사용한다. 터미널에 다음 명령어를 입력하면 바로 실행된다.8

  ```Bash
  npx https://github.com/google-gemini/gemini-cli
  ```

- **npm (전역 설치):** 시스템 어디서든 `gemini`라는 짧은 명령어로 실행하고 싶을 때 사용한다. 다음 명령어로 전역 설치할 수 있다.12

  ```Bash
  npm install -g @google/gemini-cli
  ```

- **Homebrew (macOS 사용자):** macOS 사용자라면 Homebrew를 통해 더 간편하게 설치할 수 있다.9

  ```Bash
  brew install gemini-cli
  ```

설치 방법은 사용성과 업데이트 전략에 직접적인 영향을 미친다. `npx`는 항상 최신 버전을 사용하도록 보장하지만, 일부 환경(예: Ubuntu)에서는 매번 긴 명령어를 입력해야 하는 불편함이 보고되었다.17 이 경우, 셸 설정 파일(예: `~/.zshrc` 또는 `~/.bashrc`)에 `alias gemini='npx @google/gemini-cli'`와 같이 별칭(alias)을 등록해두면 편리하다. 반면, `npm` 전역 설치는 `gemini`라는 짧은 명령어를 제공하지만, 사용자가 주기적으로 `npm update -g @google/gemini-cli`를 실행하여 수동으로 업데이트하지 않으면 구버전의 버그나 제한된 기능에 머무를 수 있다. 빠르게 발전하는 도구인 만큼, 최신 기능을 놓치지 않으려면 업데이트에 신경 쓰는 것이 중요하다.


설치 후 터미널에서 `gemini` (또는 `npx @google/gemini-cli`)를 실행하면, 첫 실행 시 초기 설정 과정이 시작된다.16 키보드 화살표 키를 사용하여 원하는 항목을 선택하고 Enter 키로 확인하면 된다.

1. **테마 선택:** 터미널 인터페이스의 색상 테마를 선택한다.12
2. **인증 방법 선택:** Gemini API에 접근하기 위한 인증 방법을 선택한다.16


Gemini CLI는 세 가지 인증 방법을 지원하며, 사용자의 목적에 따라 적절한 방법을 선택해야 한다.

- **개인 구글 계정 (권장 시작 방법):** "Login with Google"을 선택하면 웹 브라우저가 열리며, 구글 계정으로 로그인하여 인증을 완료한다. 이 방법은 가장 쉽고 빠르며, 개인 개발자에게는 Gemini 2.5 Pro 모델을 포함한 매우 넉넉한 무료 사용량(분당 60회, 하루 1,000회 요청)을 제공한다.1

- **Gemini API 키:** 더 높은 사용량이 필요하거나, 특정 모델을 안정적으로 사용하고 싶을 때 선택한다. Google AI Studio에서 API 키를 발급받은 후, 터미널 환경 변수로 설정해야 한다.9

  ```Bash
  export GEMINI_API_KEY="YOUR_API_KEY"
  ```

- **Vertex AI API 키:** 기업 환경에서 Google Cloud와의 긴밀한 통합, 감사 로그, 통합 결제 등이 필요할 때 사용한다. Google Cloud 콘솔에서 API 키를 발급받고, 두 개의 환경 변수를 설정해야 한다.9

  ```Bash
  export GOOGLE_API_KEY="YOUR_API_KEY"
  export GOOGLE_GENAI_USE_VERTEXAI=true
  ```


Gemini CLI와의 상호작용은 크게 대화형 모드와 비대화형 모드로 나뉘며, 특수 기호(`@`, `/`, `!`)를 통해 다양한 부가 기능을 활용할 수 있다.


터미널에 `gemini`를 입력하면 시작되는 기본 모드로, Read-Eval-Print Loop(REPL) 환경을 제공한다.19

`>` 프롬프트가 나타나면 자연어 질문이나 명령을 입력하여 대화를 시작할 수 있다.11 이 모드의 가장 큰 특징은 이전 대화의 맥락을 기억한다는 점이다. 따라서 여러 단계에 걸쳐 점진적으로 문제를 해결하거나 복잡한 작업을 수행하기에 적합하다.13 예를 들어, 먼저 코드베이스에 대한 질문을 하고, 그 답변을 바탕으로 코드 수정을 요청하는 등의 연속적인 작업이 가능하다.


스크립팅이나 다른 도구와의 연동을 위해 비대화형 모드가 제공된다.

- **비대화형 모드 (`-p`):** `-p` 또는 `--prompt` 플래그 뒤에 프롬프트를 직접 전달하여 단일 요청을 보내고 응답만 받은 후 즉시 종료된다.19 셸 스크립트 내에서 Gemini의 능력을 활용하거나, 특정 작업을 자동화하는 데 매우 유용하다.21
  - 예시: `gemini -p "이 코드베이스의 주요 아키텍처를 설명해줘. @./src"`
- **파이핑:** Unix의 파이프(`|`)를 사용하여 다른 명령어의 표준 출력을 Gemini CLI의 입력으로 직접 전달할 수 있다.19 이는 Unix 철학에 부합하는 매우 강력한 기능으로, 기존의 CLI 도구들과 Gemini를 유기적으로 결합할 수 있게 해준다.
  - 예시: `git diff main...feature-branch | gemini -p "이 코드 변경 사항을 바탕으로 PR 설명을 마크다운 형식으로 작성해줘."`


Gemini CLI의 인터페이스는 세 가지 특수 기호를 통해 강력한 기능을 제공한다.

- **슬래시 명령어 (`/`):** CLI 자체의 메타 기능을 제어한다. 대화 중에 `/`를 입력하면 사용 가능한 명령어 목록을 볼 수 있다.19

- **컨텍스트 명령어 (`@`):** 로컬 파일이나 디렉터리를 현재 대화의 컨텍스트로 명시적으로 포함시킨다.12

  `@`를 입력하면 파일 탐색기가 나타나 쉽게 파일을 선택할 수 있다. 이는 Gemini가 로컬 파일 시스템을 인식하고 특정 파일에 대해 작업하도록 지시하는 가장 핵심적인 문법이다.

- **셸 명령어 (`!`):** Gemini CLI 세션을 종료하지 않고도 현재 터미널의 셸 명령어를 직접 실행할 수 있게 해준다.11

  `!`를 입력하면 셸 모드로 전환되며, `pwd`, `ls -l`, `npm install` 등 모든 셸 명령어를 실행할 수 있다. 셸 모드를 빠져나오려면 `ESC` 키를 누르면 된다.

다음 표는 이러한 핵심 명령어들을 요약한 것이다.

| 분류          | 명령어/구문       | 설명                                            | 사용 예시                                           |
| ------------- | ----------------- | ----------------------------------------------- | --------------------------------------------------- |
| **대화 모드** | `gemini`          | 대화형(REPL) 세션을 시작한다.                   | `gemini`                                            |
|               | `gemini -p "..."` | 비대화형 모드로 단일 요청을 실행한다.           | `gemini -p "Hello World를 한국어로 번역해줘"`       |
|               | `... | gemini`    | 다른 명령어의 출력을 파이프로 연결한다.         | `cat code.js | gemini -p "이 코드의 버그를 찾아줘"` |
| **슬래시**    | `/help`           | 도움말 및 사용 가능한 명령어를 보여준다.        | `/help`                                             |
|               | `/tools`          | 현재 사용 가능한 내장 도구 목록을 보여준다.     | `/tools`                                            |
|               | `/memory`         | `GEMINI.md`에서 로드된 컨텍스트를 관리한다.     | `/memory show`                                      |
|               | `/mcp`            | 설정된 MCP 서버 목록과 상태를 보여준다.         | `/mcp`                                              |
|               | `/restore`        | 파일 수정 사항을 이전 체크포인트로 되돌린다.    | `/restore <checkpoint_name>`                        |
|               | `/quit`           | Gemini CLI 세션을 종료한다.                     | `/quit`                                             |
| **컨텍스트**  | `@<file/dir>`     | 지정된 파일이나 디렉터리를 컨텍스트에 추가한다. | `> @./src/main.py 이 파이썬 코드를 설명해줘.`       |
| **셸**        | `!`               | 셸 모드로 전환하여 시스템 명령어를 실행한다.    | `!ls -al`                                           |


Gemini CLI의 응답 품질과 일관성을 극적으로 향상시키는 가장 중요한 기능은 바로 `GEMINI.md` 파일을 통한 '컨텍스트 엔지니어링(Context Engineering)'이다.22 이는 일회성으로 프롬프트를 잘 작성하는 것을 넘어, AI가 특정 프로젝트의 규칙과 맥락을 지속적으로 이해하고 일관된 결과물을 내도록 '환경' 자체를 설계하는 것을 의미한다. 

`GEMINI.md`는 AI 시대의 `.editorconfig`나 `Makefile`과 같은 역할을 하는 핵심 인프라다.


`GEMINI.md` 파일은 Gemini 모델에게 제공되는 일종의 영구적인 시스템 프롬프트다.1 개발자는 이 마크다운 파일 안에 프로젝트에 특화된 다양한 규칙과 지침을 자연어로 작성할 수 있다.19

- **코딩 스타일 가이드:** "모든 파이썬 코드는 PEP 8을 준수해야 한다", "새 파일의 들여쓰기는 공백 2칸을 사용한다".19
- **아키텍처 원칙:** "모든 API 엔드포인트는 FastAPI를 사용해 구현한다", "데이터베이스 상호작용은 리포지토리 패턴을 따라야 한다".
- **선호하는 기술 스택:** "프론트엔드 웹사이트는 React와 Bootstrap을 사용한다", "백엔드 API는 Node.js와 Express.js를 선호한다".10
- **기타 지침:** "모든 공개 함수에는 JSDoc 형식의 주석을 추가해야 한다", "절대로 `force push`를 사용하지 마라".

이렇게 `GEMINI.md`에 프로젝트의 '규칙'을 명시해두면, Gemini CLI는 코드 생성, 리팩토링, 버그 수정 등 모든 작업을 이 규칙에 맞춰 수행한다. 이는 팀 전체의 코드 일관성을 유지하고 AI가 생성하는 결과물의 품질을 보장하는 강력한 메커니즘이다.10


Gemini CLI는 여러 위치에 있는 `GEMINI.md` 파일을 계층적으로 로드하여 하나의 최종 컨텍스트로 결합한다. 이 때 더 구체적인(더 가까운) 위치의 파일이 더 일반적인 파일을 덮어쓴다.14

로딩 우선순위는 다음과 같다.

1. **하위 디렉터리 컨텍스트:** `현재_경로/하위_디렉터리/GEMINI.md` (가장 구체적, 특정 모듈에 대한 규칙)
2. **프로젝트/상위 컨텍스트:** `현재_경로/GEMINI.md` (CLI 실행 위치부터 상위로 탐색하며 발견되는 파일)
3. **전역 컨텍스트 (사용자 홈):** `~/.gemini/GEMINI.md` (모든 프로젝트에 공통으로 적용되는 개인적인 규칙)

예를 들어, 전역 `GEMINI.md`에 "들여쓰기는 4칸"이라고 설정했더라도, 프로젝트 루트의 `GEMINI.md`에 "이 프로젝트의 들여쓰기는 2칸"이라고 명시하면, 해당 프로젝트 내에서는 2칸 들여쓰기가 적용된다.

또한, `@` 구문을 사용하여 다른 마크다운 파일을 `GEMINI.md` 안으로 가져올(import) 수 있다. 이를 통해 스타일 가이드, API 문서 등 컨텍스트를 모듈화하여 체계적으로 관리할 수 있다.19



- 모든 Python 코드는 PEP 8을 준수해야 한다.
- 새 파일의 들여쓰기는 공백 2칸을 사용한다.


@./src/frontend/react-style-guide.md

@./src/backend/fastapi-style-guide.md

현재 세션에 어떤 컨텍스트가 최종적으로 결합되어 사용되고 있는지 확인하고 싶다면, 언제든지 `/memory show` 명령어를 사용하면 된다.19

`GEMINI.md`를 Git을 통해 팀원들과 공유하면, 모든 팀원의 Gemini CLI가 동일한 프로젝트 지식을 바탕으로 일관된 결과물을 생성하게 되므로, 이는 팀의 생산성과 코드 품질을 좌우하는 핵심적인 '인프라'로 관리되어야 한다.


Gemini CLI가 단순한 언어 모델을 넘어 실제 작업을 수행하는 '에이전트'가 될 수 있는 이유는 바로 로컬 환경과 상호작용하는 내장 도구(tool) 덕분이다. 이 도구들은 Gemini의 '손과 발'이 되어 파일 시스템을 조작하고, 웹에서 정보를 가져오며, 다른 프로그램을 실행한다.


로컬 코드베이스와 직접 상호작용하는 가장 기본적이고 핵심적인 도구들이다.10

- `ls` (또는 `ReadFolder`): 디렉터리의 파일 및 폴더 목록을 나열한다.
- `grep` (또는 `SearchText`): 특정 텍스트 패턴으로 파일 내용을 검색한다. "모든 `TODO` 주석을 찾아줘"와 같은 요청에 사용된다.
- `read-file`, `read-many-files` (`ReadFile`, `ReadManyFiles`): 하나 또는 여러 파일의 전체 내용을 읽는다. 코드 분석이나 요약의 기본이 된다.
- `write-file` (`WriteFile`): 새로운 파일을 생성한다. "이 내용으로 `README.md` 파일을 만들어줘"와 같은 요청을 처리한다.
- `edit` (`edit`): 기존 파일의 내용을 diff 형식으로 수정한다. Gemini는 수정 사항을 적용하기 전에 사용자에게 변경될 내용을 보여주고 승인을 받는다.
- `glob` (또는 `FindFiles`): `*.js`와 같은 와일드카드 패턴을 사용하여 파일을 찾는다.


외부 인터넷에 접근하여 최신 정보를 반영하거나 특정 웹 페이지의 데이터를 분석하는 데 사용된다.10

- `web-search` (또는 `GoogleSearch`): Google 검색을 수행하여 답변의 근거를 마련(grounding)한다. "이 에러 메시지의 원인이 뭐야?"와 같은 질문에 대해 최신 정보를 찾아 답변할 수 있다.
- `web-fetch` (`WebFetch`): 특정 URL의 HTML 또는 JSON 콘텐츠를 가져온다. "이 블로그 포스트를 요약해줘"와 같은 요청에 사용된다.


- `shell` (`Shell`): 터미널 셸 명령어를 직접 실행한다.10 이 도구 덕분에 

  `npm test`, `git status`, `docker build` 등 기존에 사용하던 모든 CLI 도구와 완벽하게 연동할 수 있다. Gemini CLI의 활용 범위를 무한대로 확장시켜주는 가장 강력한 도구 중 하나다.

- `memoryTool` (`Save Memory`): "나는 항상 async/await를 선호해" 또는 "내 프로젝트의 약칭은 'GCLI'야"와 같이 사용자 개인의 선호도나 특정 사실을 세션 간에 기억하도록 저장한다.10


이러한 강력한 도구들은 생산성을 극대화하지만, 동시에 시스템에 잠재적인 위험을 초래할 수도 있다. AI가 `rm -rf /`와 같은 치명적인 명령어를 실행할 가능성을 배제할 수 없기 때문이다. 따라서 Gemini CLI는 '신뢰하되, 검증하라(Trust, but verify)' 원칙에 기반한 여러 안전장치를 제공한다.

- **사용자 확인:** 기본적으로 파일 쓰기나 셸 실행처럼 시스템을 변경할 수 있는 민감한 작업을 수행하기 전에는 항상 사용자에게 실행 허락을 구한다.26 사용자는 한 번만 허용, 항상 허용, 또는 취소를 선택할 수 있다.

- **YOLO 모드 (`--yolo`):** 'You Only Live Once'의 약자로, 모든 도구 사용 확인 절차를 건너뛰고 자동으로 승인한다.24 매우 편리하지만 그만큼 위험하므로, 자신이 무엇을 하고 있는지 완벽하게 이해하고 통제할 수 있는 상황에서만 신중하게 사용해야 한다.

- **샌드박스 모드 (`--sandbox`):** 도구 실행 환경을 Docker나 Podman 컨테이너 내부로 격리시킨다.7 만약 AI가 위험한 명령을 실행하더라도 그 영향은 격리된 컨테이너 내부에 국한되므로, 실제 시스템을 안전하게 보호할 수 있다. 보안이 중요한 작업을 할 때 적극 권장되는 매우 중요한 안전장치다.

- **체크포인팅 (`--checkpointing`):** 이 플래그를 활성화하면, `edit`이나 `write-file` 도구가 파일을 수정하기 직전에 프로젝트의 스냅샷(체크포인트)을 저장한다.19 만약 AI의 수정이 마음에 들지 않거나 문제를 일으켰을 경우, 

  `/restore` 명령어를 사용하여 이전 상태로 쉽게 되돌릴 수 있다. 이는 일종의 '실행 취소' 기능을 제공하는 보험과 같다.

이러한 제어 장치들을 잘 이해하고 활용하는 것이 Gemini CLI를 안전하고 효과적으로 사용하는 핵심이다.


Gemini CLI의 가장 혁신적인 기능 중 하나는 텍스트를 넘어 이미지, PDF 등 다양한 형태의 데이터를 이해하고 처리하는 멀티모달(multimodal) 능력이다.12 이는 기획이나 디자인 단계의 시각적 아이디어를 실제 코드로 변환하는 과정을 획기적으로 단축시켜, 개발의 패러다임을 바꿀 잠재력을 지닌다.

전통적인 개발 과정은 디자이너의 시각적 아이디어를 개발자가 코드로 '번역'하는 과정을 포함하며, 이 과정에서 많은 정보 손실과 오해가 발생한다. Gemini CLI의 멀티모달 기능은 이 '번역' 과정을 자동화하여, AI가 시각적, 문서적 컨텍스트를 직접 이해하고 코드를 생성함으로써 인간 번역가(개발자)의 개입으로 인한 오류 가능성을 줄인다. 이는 개발자의 역할을 코드 '작성자'에서 AI가 생성한 코드의 '검토자' 및 '아키텍트'로 변화시킬 수 있다.


손으로 그린 와이어프레임이나 디지털 목업 이미지를 Gemini CLI에 제공하고, 이를 기반으로 실제 작동하는 UI 코드를 생성하도록 요청할 수 있다.6

- **예시 프롬프트:**

  ```
  > @./ui-sketches/login-page.jpg 이 손으로 그린 스케치를 보고, TypeScript와 Tailwind CSS를 사용한 React 로그인 페이지 컴포넌트를 만들어줘. 이메일과 비밀번호 입력 필드, 로그인 버튼, 그리고 '비밀번호 찾기' 링크를 포함해야 해.
  ```

  이 요청을 받은 Gemini는 이미지를 분석하여 레이아웃, 구성 요소(입력창, 버튼 등)를 파악한 뒤, 그에 맞는 JSX 코드와 스타일을 생성한다.


복잡한 API 명세서나 기술 요구사항이 담긴 PDF 문서를 분석하여, 해당 내용을 구현하는 코드를 자동으로 생성할 수 있다.6

- **예시 프롬프트:**

  ```
  > @./docs/api-specification-v2.pdf 이 API 명세서 PDF 파일을 읽고, 3.2 섹션에 기술된 '사용자 정보 조회' 엔드포인트를 호출하는 Python 함수를 작성해줘. 요청 파라미터와 반환값에 대한 타입 힌트도 포함해줘.
  ```

  Gemini는 PDF 내부의 텍스트, 표, 심지어 다이어그램까지 이해하여 28, 명세에 맞는 정확한 클라이언트 코드를 생성한다.


시스템 아키텍처나 데이터 흐름을 시각적으로 표현한 다이어그램 이미지를 입력받아, 그 구조를 분석하고 개선점을 제안받을 수 있다.6

- **예시 프롬프트:**

  ```
  > @./diagrams/system-architecture.png 이 시스템 아키텍처 다이어그램을 분석해줘. 주요 컴포넌트들은 무엇이고, 데이터는 어떻게 흐르는지 설명해줘. 그리고 잠재적인 보안 취약점이나 확장성 병목 지점이 있다면 알려줘.
  ```

  이 기능은 고수준의 시스템 설계를 구체적인 코드 구현과 연결하고, 잠재적인 문제를 조기에 발견하는 데 도움을 준다.


Gemini CLI는 `settings.json` 파일과 커스텀 명령어 기능을 통해 사용자의 작업 스타일과 프로젝트 요구사항에 맞게 동작을 미세 조정하고 자동화할 수 있는 강력한 방법을 제공한다. 이는 개인의 생산성을 높이는 동시에 팀의 작업 표준을 유지하는, '개인화'와 '표준화'라는 두 가지 목표를 동시에 달성하는 정교한 설계다.


`settings.json`은 Gemini CLI의 동작을 제어하는 핵심 설정 파일이다.19 이 파일은 두 가지 레벨로 존재하며, 계층적으로 적용된다.

- **전역 설정 (`~/.gemini/settings.json`):** 사용자의 홈 디렉터리에 위치하며, 모든 프로젝트에 공통으로 적용되는 개인적인 설정을 담는다.19 예를 들어, 선호하는 UI 테마, 자주 사용하는 텍스트 편집기 경로 등을 여기에 설정할 수 있다.
- **프로젝트 설정 (`프로젝트_폴더/.gemini/settings.json`):** 프로젝트의 루트 디렉터리에 위치하며, 해당 프로젝트 내에서만 적용되는 설정을 담는다.19 이 파일은 Git과 같은 버전 관리 시스템에 포함시켜 팀 전체가 동일한 설정을 공유하는 데 사용된다. 프로젝트 설정은 전역 설정을 덮어쓴다.

주요 설정 항목은 다음과 같다 19:

```JSON
{
  "theme": "GitHub",
  "autoAccept": false,
  "sandbox": "docker",
  "checkpointing": {
    "enabled": true
  },
  "fileFiltering": {
    "respectGitIgnore": true
  },
  "usageStatisticsEnabled": false,
  "excludeTools": [
    "run_shell_command(rm)"
  ],
  "mcpServers": {
    "myPythonServer": {
      "command": "python",
      "args": ["-m", "my_mcp_server"],
      "cwd": "./mcp_tools/python"
    }
  }
}
```

- `theme`: 터미널 UI의 색상 테마.
- `sandbox`: 도구 실행을 위한 샌드박스 환경 ("docker", "podman", `true`).
- `checkpointing`: 파일 수정 전 스냅샷 저장 기능 활성화.
- `autoAccept`: 안전한 읽기 전용 도구 호출을 자동으로 승인할지 여부.
- `fileFiltering.respectGitIgnore`: `.gitignore`에 명시된 파일을 무시할지 여부.
- `excludeTools`: 특정 도구나 명령어(예: `rm`)의 사용을 금지하여 안전성 확보.
- `mcpServers`: 외부 도구 연동을 위한 MCP 서버 설정 (다음 장에서 상세히 다룸).


자주 사용하는 복잡하고 긴 프롬프트를 `.toml` 형식의 파일로 저장하여, 간단한 커스텀 명령어로 만들어 재사용할 수 있다.19 이는 반복적인 작업을 자동화하고, 팀 내에서 표준화된 작업 절차를 공유하는 데 매우 효과적이다.30

커스텀 명령어는 `~/.gemini/commands/그룹명/명령어.toml` 경로에 파일을 생성하여 정의한다.

- **예시: Jest 테스트 생성 명령어 (`~/.gemini/commands/test/gen.toml`)**

  Ini, TOML

  ```
  # 사용법: /test:gen "로그인 버튼에 대한 유닛 테스트를 만들어줘"
  description = "주어진 설명에 기반하여 Jest 유닛 테스트 코드를 생성합니다."
  prompt = """
  당신은 10년차 시니어 테스트 엔지니어입니다.
  다음 요구사항에 기반하여 Jest 테스트 프레임워크를 사용한 포괄적인 유닛 테스트를 작성해주세요.
  엣지 케이스도 고려해야 합니다.
  
  요구사항: {{args}}
  
  오직 테스트 코드 블록만 반환해주세요.
  """
  ```

이 파일을 저장하면, Gemini CLI 세션 내에서 `/test:gen "요구사항..."` 과 같이 간단하게 호출하여 복잡한 프롬프트를 실행할 수 있다. `{{args}}` 부분은 명령어 뒤에 오는 문자열로 동적으로 치환된다.


Gemini CLI의 진정한 잠재력은 Model Context Protocol(MCP)을 통해 외부 도구 및 서비스와 연동하여 그 능력을 무한히 확장할 수 있다는 점에 있다.14 MCP는 AI 에이전트를 위한 일종의 '플러그인 아키텍처' 또는 'API' 역할을 하여, Gemini CLI가 핵심 기능에 집중하면서도 커뮤니티와 서드파티를 통해 생태계를 확장할 수 있게 해준다.


MCP는 AI 에이전트(Gemini CLI)가 외부의 전문화된 도구(Tool)와 표준화된 방식으로 통신하기 위한 프로토콜이다.1 개발자는 특정 기능을 수행하는 MCP 서버를 직접 만들거나 기존에 공개된 서버를 사용하여 Gemini CLI에 새로운 능력을 동적으로 추가할 수 있다. 예를 들어, 이미지 생성 모델인 Imagen, 사내 데이터베이스 조회, 특정 클라우드 서비스 제어 등의 기능을 MCP 서버로 구현하여 Gemini CLI와 연동시킬 수 있다.11


외부 MCP 서버를 연동하려면 `settings.json` 파일의 `mcpServers` 객체에 서버 정보를 등록해야 한다.19

- **설정 항목 예시:**

  JSON

  ```
  "mcpServers": {
    "githubServer": {
      "command": "node",
      "args": ["./mcp_servers/github/dist/index.js"],
      "cwd": "/path/to/github-mcp-server",
      "env": {
        "GITHUB_TOKEN": "$GITHUB_PAT"
      },
      "timeout": 15000,
      "trust": false
    }
  }
  ```

  - `command`, `args`, `cwd`: 서버를 실행하기 위한 명령어와 경로 정보.31
  - `env`: 서버 프로세스에 전달할 환경 변수. `$VAR_NAME` 구문을 통해 셸 환경 변수를 참조할 수 있다.19
  - `timeout`: 요청 타임아웃 (밀리초 단위).
  - `trust`: `true`로 설정하면 이 서버가 제공하는 모든 도구 사용 시 사용자 확인을 건너뛴다.19


- **GitHub 연동:** GitHub에서 공식적으로 제공하는 MCP 서버(`github/github-mcp-server`)를 설정하면, Gemini CLI 내에서 GitHub와 직접 상호작용할 수 있다.31 GitHub Personal Access Token(PAT)을 발급받아 환경 변수로 설정해야 한다.14
  - **예시 프롬프트:** `> 우리 팀의 'frontend' 저장소에서 'bug' 라벨이 붙은 최신 이슈 5개를 찾아줘.`
- **Figma 연동:** Figma용 MCP 서버를 연동하면, 디자인 시스템에 프로그래밍 방식으로 접근할 수 있다.8 Figma API 키가 필요하다.
  - **예시 프롬프트:** `> Figma 디자인 시스템에 정의된 'primary-button' 스타일과 똑같은 React 컴포넌트를 생성해줘.`
- **Netdata 연동:** Netdata MCP 서버를 설정하면, 인프라 모니터링 데이터를 실시간으로 조회하고 분석할 수 있다.32
  - **예시 프롬프트:** `> 지난 1시간 동안 'web-server-01'의 CPU 사용량에 이상 징후가 있었는지 분석해줘.`

`/mcp` 명령어를 사용하면 현재 설정된 MCP 서버 목록과 각 서버가 제공하는 도구들을 확인할 수 있다.31


지금까지 배운 기능들을 조합하여 실제 개발 현장에서 마주치는 문제들을 해결하는 구체적인 워크플로우와 프롬프트 '레시피'를 소개한다.


처음 접하는 대규모 코드베이스를 파악하는 것은 언제나 어려운 일이다. Gemini CLI를 사용하면 이 과정을 대화형으로 진행할 수 있다.

- **프롬프트:** `> 이 프로젝트의 전체적인 아키텍처를 설명해줘. 주요 디렉터리들의 역할은 뭐야? 그리고 어떤 외부 라이브러리에 의존하고 있는지 알려줘.` 9
- **워크플로우:**
  1. Gemini는 `ls -R`과 유사한 `glob` 도구를 사용하여 전체 파일 및 디렉터리 구조를 파악한다.
  2. `package.json`, `pom.xml`, `requirements.txt` 등 의존성 관리 파일을 `read-file` 도구로 읽어 사용된 라이브러리를 분석한다.
  3. 주요 소스 코드 파일을 읽고 분석하여 각 모듈의 역할을 추론한다.
  4. 이 모든 정보를 종합하여 구조화된 답변을 생성한다.


매번 커밋 메시지를 작성하고 Pull Request 설명을 채우는 것은 번거로운 작업이다. 이 과정을 자동화할 수 있다.

- **프롬프트 (파이핑 활용):** `git diff main | gemini -p "이 코드 변경 사항을 바탕으로, Conventional Commits 규칙에 맞는 커밋 메시지를 작성해줘. 타입은 feat, fix, refactor 중에서 자동으로 선택해줘."` 6
- **워크플로우:**
  1. `git diff` 명령어의 출력이 파이프를 통해 Gemini CLI로 전달된다.
  2. Gemini는 코드 변경 내용을 분석하여 새로운 기능 추가(feat), 버그 수정(fix), 리팩토링(refactor) 등 변경의 핵심 의도를 파악한다.
  3. 파악된 의도에 맞춰 제목과 본문으로 구성된 표준 커밋 메시지를 생성한다.


단순하지만 반복적인 파일 처리 작업을 셸 스크립트와 Gemini CLI를 결합하여 자동화할 수 있다.

- **프롬프트 (셸 스크립트 내에서 호출):** `> 이 디렉터리의 모든 JPG 이미지를 50% 품질의 WebP 형식으로 변환하고, 파일명은 그대로 유지해줘.` 9

- **워크플로우 (예시 셸 스크립트 `batch_convert.sh`):**

  ```Bash
  #!/bin/bash
  for file in *.jpg; do
    echo "Processing $file..."
    gemini -p "이 이미지('@$file')를 50% 품질의 WebP 형식으로 변환하고, 결과 파일을 '${file%.jpg}.webp'로 저장해줘."
  done
  echo "All files converted."
  ```

  이 스크립트는 디렉터리의 모든 `.jpg` 파일을 순회하며, 각 파일을 컨텍스트로 포함하여 Gemini CLI에 변환 작업을 요청한다.


`google-gemini/gemini-cli-action`을 사용하면 GitHub 워크플로우에 Gemini의 지능을 통합할 수 있다.34

- **이슈 자동 분류:**
  - **트리거:** 새 이슈가 생성될 때.
  - **워크플로우:** 이슈의 제목과 본문을 Gemini에 전달하여 내용을 분석하고, `bug`, `feature-request`, `documentation` 등 적절한 라벨을 자동으로 할당하도록 한다.34
- **PR 자동 리뷰:**
  - **트리거:** 새 PR이 열리거나, PR 코멘트에 `@gemini-cli /review`가 입력될 때.
  - **워크플로우:** 변경된 코드(`git diff`)를 Gemini에 전달하여 코드 리뷰를 요청한다. Gemini는 잠재적인 버그, 코딩 스타일 위반, 개선 제안 등을 찾아 PR에 코멘트로 남긴다.34
- **설정:** 이 기능을 사용하려면 GitHub Actions 워크플로우 `.yml` 파일에 해당 액션을 설정하고, `GEMINI_API_KEY`를 GitHub Repository Secret으로 안전하게 등록해야 한다.34


Gemini CLI를 효과적으로 사용하려면 사용 가능한 모델, 비용, 그리고 성능 제한에 대한 현실적인 이해가 필요하다. 특히 '무료'라는 말에 현혹되어 겪을 수 있는 성능 저하 문제에 대한 대처법을 아는 것이 중요하다.


Gemini CLI는 주로 두 가지 모델을 사용한다.

- **Gemini 2.5 Pro:** 기본적으로 사용되는 모델로, 강력한 추론 능력과 100만 토큰이라는 방대한 컨텍스트 창을 제공한다.1 복잡한 코드 분석, 대규모 리팩토링, 창의적인 콘텐츠 생성 등 고차원적인 작업에 적합하다.
- **Gemini 2.5 Flash:** 더 가볍고 빠른 모델이다.12 Pro 모델에 비해 추론 능력은 다소 낮지만, 응답 속도가 훨씬 빠르다. 간단한 질문, 코드 요약, 형식 변환 등 신속한 응답이 중요한 작업에 유리하다.


인증 방식에 따라 비용과 사용량 제한이 달라진다.

- **무료 등급 (개인 구글 계정 로그인):** 개인 구글 계정으로 로그인하면 Gemini Code Assist 라이선스가 무료로 제공된다. 이론적으로 Gemini 2.5 Pro 모델에 대해 분당 60회, 하루 1,000회라는 매우 넉넉한 요청 할당량을 제공한다.1 개인적인 학습이나 소규모 프로젝트, 도구 테스트 용도로는 충분하다.
- **유료 옵션 (API 키 또는 Vertex AI):** 무료 등급의 할당량을 초과하거나, 특정 모델(예: Pro)을 안정적으로 사용하고 싶을 때, 또는 기업용 기능(감사 로그, 통합 관리 등)이 필요할 때 사용한다.1 Google AI Studio나 Google Cloud에서 발급받은 API 키를 사용하며, 사용한 토큰 양에 따라 비용이 청구된다.


**현상:** 많은 사용자들이 무료 등급으로 Gemini CLI를 사용할 때, 몇 번의 프롬프트만으로도 응답 속도가 현저히 느려지거나, 예고 없이 더 성능이 낮은 `gemini-2.5-flash` 모델로 강제 전환되는 경험을 보고하고 있다.35

**원인:** 이는 버그라기보다는, 과도한 사용을 방지하고 안정적인 서비스를 위해 유료 플랜 사용을 유도하기 위한 구글의 의도된 정책일 가능성이 높다. 무료 등급의 '하루 1,000회'는 최대 이론치이며, 실제로는 짧은 시간 내에 집중적으로 사용하면 제한에 걸리기 쉽다.

**대처법:**

1. **기대치 조정:** 무료 등급은 '맛보기' 또는 '가벼운 용도'로 생각하는 것이 좋다. 중요한 업무나 지속적인 개발 작업에 사용하기에는 성능의 변동성이 크다.
2. **API 키 사용:** 안정적이고 예측 가능한 성능이 필요한 전문적인 개발 작업에는 **처음부터 Google AI Studio에서 API 키를 발급받아 사용하는 것을 강력히 권장한다.** 이것이 스트레스 없이 Gemini CLI의 모든 성능을 끌어내는 가장 확실한 방법이다.

다음 표는 인증 방식에 따른 차이점을 요약한 것이다.

| 인증 방식          | 주요 모델                                       | 비용                      | 주요 할당량                | 추천 사용 사례                                   |
| ------------------ | ----------------------------------------------- | ------------------------- | -------------------------- | ------------------------------------------------ |
| **개인 구글 계정** | Gemini 2.5 Pro (성능 제한적, Flash로 전환 가능) | 무료                      | 하루 최대 1,000회 (이론상) | 개인 학습, 취미, 기능 테스트                     |
| **Gemini API 키**  | Gemini 2.5 Pro / Flash (선택 가능)              | 사용량 기반 과금          | API 등급별 상이            | 전문 개발자, 소규모 프로젝트, 안정적인 성능 요구 |
| **Vertex AI 키**   | Gemini 2.5 Pro / Flash (선택 가능)              | 기업용 플랜 / 사용량 과금 | 프로젝트/조직 단위 공유    | 대규모 기업, 팀 단위 협업, 보안 및 거버넌스 요구 |

결론적으로, Gemini CLI는 터미널 기반 개발자에게 전례 없는 생산성 향상을 가져다줄 강력한 AI 에이전트다. 이 가이드에서 다룬 설치, 기본 사용법, 컨텍스트 엔지니어링, 도구 활용, 확장성, 그리고 현실적인 제약 사항들을 잘 이해한다면, 이 도구를 단순한 보조 수단을 넘어 개발 워크플로우의 핵심적인 파트너로 만들 수 있을 것이다.


1. Google announces Gemini CLI: your open-source AI agent, accessed July 25, 2025, https://blog.google/technology/developers/introducing-gemini-cli-open-source-ai-agent/
2. Gemini CLI: A comprehensive guide to understanding, installing, and leveraging this new Local AI Agent : r/GeminiAI - Reddit, accessed July 25, 2025, https://www.reddit.com/r/GeminiAI/comments/1lkojt8/gemini_cli_a_comprehensive_guide_to_understanding/
3. Google Gemini CLI: The New AI Dev Tool Shaking Up Your Terminal - Medium, accessed July 25, 2025, https://medium.com/@gowripaliwal/google-gemini-cli-the-new-ai-dev-tool-shaking-up-your-terminal-1aa9d4d318b2
4. Gemini CLI | Gemini Code Assist - Google for Developers, accessed July 25, 2025, https://developers.google.com/gemini-code-assist/docs/gemini-cli
5. Gemini CLI | Gemini for Google Cloud, accessed July 25, 2025, https://cloud.google.com/gemini/docs/codeassist/gemini-cli
6. Mastering the Gemini CLI. The Complete Guide to AI-Powered... | by ..., accessed July 25, 2025, https://medium.com/@creativeaininja/mastering-the-gemini-cli-cb6f1cb7d6eb
7. From Bash to Brain: How Gemini CLI Is Rethinking Developer Tools - Medium, accessed July 25, 2025, https://medium.com/@delimiterbob/from-bash-to-brain-how-gemini-cli-is-rethinking-developer-tools-f662ac354852
8. Gemini CLI tutorial - Will it replace Windsurf and Cursor? - LogRocket Blog, accessed July 25, 2025, https://blog.logrocket.com/gemini-cli-tutorial/
9. google-gemini/gemini-cli: An open-source AI agent that brings the power of Gemini directly into your terminal. - GitHub, accessed July 25, 2025, https://github.com/google-gemini/gemini-cli
10. Gemini CLI - Simon Willison's Weblog, accessed July 25, 2025, https://simonwillison.net/2025/Jun/25/gemini-cli/
11. Gemini CLI Full Tutorial - DEV Community, accessed July 25, 2025, https://dev.to/proflead/gemini-cli-full-tutorial-2ab5
12. Google Gemini CLI Tutorial: How to Install and Use It (With Images) - DEV Community, accessed July 25, 2025, https://dev.to/auden/google-gemini-cli-tutorial-how-to-install-and-use-it-with-images-4phb
13. eliben/gemini-cli: Access Gemini LLMs from the command-line - GitHub, accessed July 25, 2025, https://github.com/eliben/gemini-cli
14. Give Gemini CLI the Ability to Generate Images and Video, Work with GitHub Repos, and Use Other Tools | by Dazbo (Darren Lester) | Google Cloud - Medium, accessed July 25, 2025, https://medium.com/google-cloud/give-gemini-cli-the-ability-to-generate-images-and-video-work-with-github-repos-and-use-other-482172571f99
15. Using Gemini CLI Is Fucking Awesome. I'm having a blast building my personal AI's brain from the command line. : r/GeminiAI - Reddit, accessed July 25, 2025, https://www.reddit.com/r/GeminiAI/comments/1lnz4rf/using_gemini_cli_is_fucking_awesome_im_having_a/
16. Gemini's command line tool is a hidden productivity game changer - and it's free - ZDNET, accessed July 25, 2025, https://www.zdnet.com/article/geminis-command-line-tool-is-a-hidden-productivity-game-changer-and-its-free/
17. How to Install Gemini CLI on Ubuntu to Access AI from Your Terminal, accessed July 25, 2025, https://www.omgubuntu.co.uk/2025/07/how-to-install-gemini-cli-on-ubuntu
18. Gemini CLI: A Guide With Practical Examples - DataCamp, accessed July 25, 2025, https://www.datacamp.com/tutorial/gemini-cli
19. Google Gemini CLI Cheatsheet - Philschmid, accessed July 25, 2025, https://www.philschmid.de/gemini-cli-cheatsheet
20. Gemini CLI Tutorial Series - Medium, accessed July 25, 2025, https://medium.com/google-cloud/gemini-cli-tutorial-series-77da7d494718
21. Gemini CLI is awesome! But only when you make Claude Code use it as its bitch. - Reddit, accessed July 25, 2025, https://www.reddit.com/r/ChatGPTCoding/comments/1lm3fxq/gemini_cli_is_awesome_but_only_when_you_make/
22. gemini-cli / GitHub Topics, accessed July 25, 2025, https://github.com/topics/gemini-cli
23. Philschmid, accessed July 25, 2025, https://www.philschmid.de/
24. Gemini CLI Tutorial Series - Part 2 : Gemini CLI Command line parameters | by Romin Irani | Google Cloud - Medium, accessed July 25, 2025, https://medium.com/google-cloud/gemini-cli-tutorial-series-part-2-gemini-cli-command-line-parameters-e64e21b157be
25. Practical Gemini CLI: Tool calling | by Prashanth Subrahmanyam | Google Cloud - Medium, accessed July 25, 2025, https://medium.com/google-cloud/practical-gemini-cli-tool-calling-52257edb3f8f
26. Gemini CLI Tutorial Series - Part 4 : Built-in Tools | by Romin Irani | Google Cloud - Medium, accessed July 25, 2025, https://medium.com/google-cloud/gemini-cli-tutorial-series-part-4-built-in-tools-c591befa59ba
27. The Gemini CLI might change how I work. Here are four prompts that prove it., accessed July 25, 2025, https://seroter.com/2025/06/26/the-gemini-cli-might-change-how-i-work-here-are-four-prompts-that-prove-it/
28. 7 examples of Gemini's multimodal capabilities in action - Google Developers Blog, accessed July 25, 2025, https://developers.googleblog.com/en/7-examples-of-geminis-multimodal-capabilities-in-action/
29. Gemini CLI Tutorial Series - Part 3 : Configuration settings via settings.json and .env files, accessed July 25, 2025, https://medium.com/google-cloud/gemini-cli-tutorial-series-part-3-configuration-settings-via-settings-json-and-env-files-669c6ab6fd44
30. Supercharge Code Workflows with Gemini CLI + JSON Prompts Parameters - Medium, accessed July 25, 2025, https://medium.com/@and.gpch.dev/supercharge-code-workflows-with-gemini-cli-json-prompts-parameters-5ce9deaf15b2
31. Gemini CLI Tutorial Series - Part 5 : Github MCP Server | by Romin Irani | Google Cloud, accessed July 25, 2025, https://medium.com/google-cloud/gemini-cli-tutorial-series-part-5-github-mcp-server-b557ae449e6e
32. Gemini CLI - Learn Netdata, accessed July 25, 2025, https://learn.netdata.cloud/docs/ai-&-ml/devops-copilot/gemini-cli
33. Google Gemini CLI: AI in Your Terminal (Windows • Linux • macOS) - YouTube, accessed July 25, 2025, https://www.youtube.com/watch?v=xqvprnPocHs
34. google-gemini/gemini-cli-action - GitHub, accessed July 25, 2025, https://github.com/google-gemini/gemini-cli-action
35. Gemini CLI switch models? : r/GoogleGeminiAI - Reddit, accessed July 25, 2025, https://www.reddit.com/r/GoogleGeminiAI/comments/1llmde2/gemini_cli_switch_models/
36. Gemini CLI Model : r/Bard - Reddit, accessed July 25, 2025, https://www.reddit.com/r/Bard/comments/1lkcqmz/gemini_cli_model/

