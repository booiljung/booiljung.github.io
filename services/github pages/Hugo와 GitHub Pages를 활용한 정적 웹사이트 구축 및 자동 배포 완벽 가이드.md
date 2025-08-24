[Github Pages](./index.md)
# Hugo와 GitHub Pages를 활용한 정적 웹사이트 구축 및 자동 배포 완벽 가이드



정적 사이트 생성기(Static Site Generator, SSG)는 현대 웹 개발의 중요한 패러다임으로 자리 잡았다. SSG의 기본 원리는 웹사이트를 구성하는 모든 페이지를 사전에 HTML, CSS, JavaScript 파일로 생성(build)해두는 것이다.1 사용자가 웹사이트에 접속하면, 서버는 이 미리 만들어진 정적 파일들을 그대로 전달하기만 한다.

이는 데이터베이스와 서버사이드 렌더링에 의존하는 동적 사이트와 근본적인 차이를 보인다. 동적 사이트(예: WordPress)는 사용자의 요청이 있을 때마다 서버에서 데이터베이스를 조회하고 템플릿을 조합하여 동적으로 HTML 페이지를 생성한다.3 이 방식은 사용자별 맞춤형 콘텐츠를 제공하는 데 유리하지만, 서버의 부하가 크고 응답 속도가 느려질 수 있으며, 데이터베이스와 서버 측 코드의 복잡성으로 인해 잠재적인 보안 취약점이 존재할 수 있다.

반면, 정적 사이트는 서버 측 로직이 없어 구조가 단순하고, 공격 표면이 적어 보안성이 뛰어나다. 무엇보다 미리 생성된 파일을 제공하므로 로딩 속도가 매우 빠르다.4 SSG를 선택하는 것은 단순히 기술 스택을 바꾸는 것을 넘어, 웹사이트의 핵심 목적인 '콘텐츠 전달'에 집중하여 속도, 보안, 관리 용이성을 극대화하려는 전략적 접근이라 할 수 있다.


수많은 SSG 중에서 Hugo는 독보적인 장점으로 주목받는다. Jekyll, Hexo 등 다른 인기 있는 SSG와 비교했을 때 Hugo의 가장 큰 차별점은 Go 언어로 작성되었다는 점이다.2 이로 인해 타의 추종을 불허하는 빌드 속도를 자랑한다. 수백, 수천 페이지에 달하는 대규모 사이트조차 수 초 내에 빌드를 완료할 수 있다.3

또한 Hugo는 단일 실행 파일(바이너리)로 배포되므로 Ruby(Jekyll)나 Node.js(Hexo)와 같은 별도의 런타임 환경에 대한 의존성이 없다(Zero Dependency).6 이는 개발 환경 설정의 복잡성을 획기적으로 낮추어, 어떤 운영체제에서든 Hugo 실행 파일 하나만으로 즉시 개발을 시작할 수 있게 한다. Jekyll의 경우, 가장 널리 사용되지만 페이지 수가 많아질수록 빌드 시간이 기하급수적으로 늘어난다는 단점이 꾸준히 지적되어 왔다.1

Hugo의 경이로운 빌드 속도는 단순한 성능 지표 이상의 의미를 가진다. 콘텐츠를 수정하고 저장하는 순간, 변경 사항이 거의 실시간으로 로컬 서버에 반영되는 것을 확인할 수 있다.6 이러한 즉각적인 피드백은 개발자의 생산성을 극대화한다. 글을 쓰거나 디자인을 수정하는 과정에서 발생하는 기술적 마찰을 최소화하여, 창작의 흐름이 끊기지 않도록 지원하는 것이다.


GitHub Pages는 개발자에게 가장 매력적인 정적 사이트 호스팅 솔루션 중 하나다. 가장 큰 장점은 GitHub 저장소(repository)를 기반으로 무료로 정적 웹사이트를 호스팅해준다는 점이다.4 이는 개인 블로그나 포트폴리오, 오픈소스 프로젝트 문서를 운영하는 데 있어 비용 부담을 완전히 제거해준다.

또한, 소스 코드를 관리하는 Git 저장소와 배포된 웹사이트가 하나의 플랫폼(GitHub)에서 통합 관리되므로 워크플로우가 매우 단순하고 직관적이다.8 별도의 웹 호스팅 서비스를 계약하고, FTP 클라이언트를 통해 파일을 업로드하는 등의 번거로운 과정이 필요 없다. 개발자는 자신이 가장 익숙한 도구인 Git과 GitHub만을 사용하여 웹사이트의 콘텐츠와 코드를 관리하고 배포할 수 있다. 이는 문서와 웹사이트를 코드처럼 취급하는 'Docs as Code' 철학을 실현하기에 가장 이상적인 환경을 제공한다.


Hugo와 GitHub Pages는 개별적으로도 훌륭한 도구지만, 이 둘의 조합은 단순한 합 이상의 시너지 효과를 창출한다. 바로 Hugo의 빠른 빌드 능력과 GitHub Actions의 강력한 자동화 기능을 결합하여, 완전 자동화된 콘텐츠 발행 파이프라인(CI/CD Pipeline)을 구축할 수 있다는 점이다.

이 파이프라인이 완성되면, 개발자는 로컬 환경에서 마크다운으로 콘텐츠를 작성한 뒤 `git push` 명령어 하나만 실행하면 된다. 그러면 GitHub Actions가 자동으로 Hugo를 실행하여 웹사이트를 빌드하고, 그 결과물을 GitHub Pages에 배포하는 모든 과정을 알아서 처리한다.9

이 조합은 단순한 '블로그 제작 도구'를 넘어선다. 이는 개인 개발자나 소규모 팀이 추가 비용 없이 전문적인 개발 및 배포 워크플로우를 경험하고 운영할 수 있도록 하는 하나의 '개발 생태계'를 제공한다. 과거에는 기업 수준의 인프라와 전문 DevOps 인력이 필요했던 CI/CD 자동화를, 이제 누구나 무료로, 단일 플랫폼 내에서 구축할 수 있게 된 것이다. 이는 기술 접근성의 민주화를 보여주는 대표적인 사례라 할 수 있다.



모든 버전 관리 작업의 출발점은 Git을 설치하고 초기 설정을 완료하는 것이다. Git은 소스 코드의 변경 이력을 추적하고 협업을 가능하게 하는 분산 버전 관리 시스템으로, Hugo 프로젝트와 GitHub 연동에 필수적이다.

운영체제에 따라 가장 효율적인 방법으로 Git을 설치한다.

- **Windows**:(https://git-scm.com/download/win) 공식 인스톨러를 다운로드하여 설치하는 것이 가장 일반적이다.11 설치 과정에서 대부분의 옵션은 기본값을 유지해도 무방하나, 'Adjusting your PATH environment' 단계에서는 "Git from the command line and also from 3rd-party software"를 선택하여 명령 프롬프트나 PowerShell 등 어디서든 

  `git` 명령어를 사용할 수 있도록 설정하는 것이 좋다.13

- **macOS**: macOS에는 기본적으로 Git이 포함되어 있는 경우가 많지만, 최신 버전을 사용하기 위해 패키지 매니저인 Homebrew를 통해 설치하는 것을 권장한다. Homebrew가 설치되어 있다면, 터미널에서 다음 명령어를 실행한다.11

  ```Bash
  brew install git
  ```

- **Linux**: 각 배포판에 내장된 패키지 매니저를 사용한다.

  - Debian/Ubuntu 계열: `sudo apt-get update && sudo apt-get install git` 11
  - Fedora/RHEL 계열: `sudo dnf install git` 또는 `sudo yum install git` 11

Git 설치가 완료되면, 버전 확인 명령어로 정상적으로 설치되었는지 확인한다.

```Bash
git --version
```

다음으로, 모든 커밋(commit)에 기록될 작성자 정보를 설정한다. 이 설정은 전역(`--global`)으로 한 번만 지정해두면 모든 로컬 저장소에 적용된다.

```Bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

이 단계는 단순한 도구 설치를 넘어, 이후의 모든 작업 내역을 추적하고 협업의 기반을 마련하는 필수적인 초기화 과정이다.


웹사이트를 생성하고 관리할 핵심 도구인 Hugo를 설치한다. Hugo는 일반 버전과 'Extended' 버전으로 나뉘는데, 많은 최신 테마들이 Sass/SCSS와 같은 고급 기능을 사용하기 때문에 처음부터 **Extended 버전**을 설치하는 것이 강력히 권장된다.5

패키지 매니저를 사용하면 설치와 버전 관리가 매우 용이하므로, 수동으로 바이너리를 다운로드하여 PATH 환경 변수를 설정하는 것보다 권장된다.


| 운영체제    | 권장 방법     | 명령어 예시                            | 비고                                                    |
| ----------- | ------------- | -------------------------------------- | ------------------------------------------------------- |
| **Windows** | Chocolatey    | `choco install hugo-extended -confirm` | Chocolatey 패키지 매니저가 미리 설치되어 있어야 한다.14 |
|             | Scoop         | `scoop install hugo-extended`          | Scoop 패키지 매니저가 미리 설치되어 있어야 한다.14      |
| **macOS**   | Homebrew      | `brew install hugo`                    | Homebrew는 기본적으로 Extended 버전을 설치한다.2        |
| **Linux**   | Snap          | `sudo snap install hugo`               | Snap을 통해 설치할 수 있다.19                           |
|             | Debian/Ubuntu | `sudo apt install hugo`                | 배포판 저장소의 버전이 최신이 아닐 수 있다.15           |
|             | Arch Linux    | `sudo pacman -S hugo`                  | Arch Linux의 공식 저장소를 통해 설치한다.19             |

과거 Ruby 기반의 Jekyll 같은 SSG는 Windows 환경에서 복잡한 의존성 문제로 인해 설치에 어려움을 겪는 경우가 많았다.1 하지만 Hugo는 Go 언어의 특성상 단일 바이너리로 컴파일되어 별도의 런타임이 필요 없으며 6, Chocolatey나 Scoop 같은 현대적인 Windows용 패키지 매니저를 공식적으로 지원한다.14 덕분에 사용자는 복잡한 환경 설정 없이 간단한 명령어 한 줄로 설치를 마칠 수 있다. 이는 Hugo의 플랫폼 접근성을 높이고 더 넓은 사용자층을 확보하는 데 기여한 중요한 요인이다.

설치 후, 다음 명령어로 Hugo가 정상적으로 설치되었는지, 그리고 Extended 버전이 맞는지 확인한다.

```Bash
hugo version
```

출력 결과에 `extended`라는 단어가 포함되어 있다면 성공적으로 설치된 것이다.



개발 환경 구축이 완료되면, Hugo 프로젝트의 기본 골격을 생성할 차례다. 원하는 상위 디렉터리로 이동한 후, 다음 명령어를 실행한다.

```Bash
hugo new site <프로젝트명>
```

예를 들어, `my-blog`라는 이름의 사이트를 생성하려면 `hugo new site my-blog`를 입력한다. 이 명령어는 `my-blog`라는 이름의 디렉터리를 생성하고, 그 안에 사이트 운영에 필요한 모든 하위 디렉터리와 기본 설정 파일(`hugo.toml`)을 자동으로 만들어준다.2 이는 개발자가 디렉터리 구조에 대한 고민 없이 즉시 콘텐츠 제작에 집중할 수 있도록 돕는 스캐폴딩(scaffolding) 기능이다.21


`hugo new site` 명령어로 생성된 디렉터리 구조는 '관례에 의한 설정(Convention over Configuration)' 철학을 따른다. 이는 각 파일과 디렉터리의 위치 자체가 특별한 의미를 가지므로, 개발자가 복잡한 설정을 하지 않아도 Hugo가 알아서 사이트를 구성한다는 의미다. 각 핵심 디렉터리의 역할은 다음과 같다.

- `archetypes/`: 새 콘텐츠 파일을 생성할 때(`hugo new content...`) 사용할 템플릿 파일(`default.md` 등)이 위치한다. 이 템플릿은 Front Matter의 기본값을 정의하여 콘텐츠 작성의 일관성을 유지해준다.22
- `content/`: 웹사이트의 모든 실제 콘텐츠(주로 마크다운 파일)가 저장되는 가장 중요한 디렉터리다. 이 디렉터리 내부의 폴더 구조는 그대로 웹사이트의 URL 경로 구조로 반영된다. 예를 들어, `content/posts/my-first-post.md` 파일은 `https://example.com/posts/my-first-post/`와 같은 URL로 생성된다.24
- `data/`: TOML, YAML, JSON 형식의 데이터 파일을 저장하는 곳이다. 이곳의 데이터는 템플릿 어디에서든 불러와 사용할 수 있어, 사이트 전반에 걸쳐 사용되는 정적인 정보(예: 저자 목록, 프로젝트 데이터)를 관리하기에 용이하다.22
- `layouts/`: 웹사이트의 시각적 표현을 담당하는 HTML 템플릿 파일들이 위치한다. Hugo는 `content/` 디렉터리의 콘텐츠를 이곳의 템플릿과 결합하여 최종 HTML 페이지를 생성한다. 프로젝트 루트의 `layouts` 디렉터리는 `themes` 디렉터리에 있는 테마의 템플릿 파일을 덮어쓰는(override) 역할을 하므로, 테마의 특정 부분만 수정하고 싶을 때 유용하다.22
- `static/`: 이미지, CSS, JavaScript, 폰트 파일 등 Hugo의 처리 과정 없이 최종 결과물 디렉터리(`public`)로 그대로 복사될 정적 자원들을 저장한다. `static/img/logo.png` 파일은 빌드 후 `public/img/logo.png`로 복사된다.22
- `themes/`: 외부에서 다운로드한 테마들이 위치하는 디렉터리다. 일반적으로 Git Submodule을 통해 관리된다.22
- `hugo.toml`: 사이트 전반의 설정을 담고 있는 핵심 파일이다. 사이트의 제목, 기본 URL, 사용할 테마 등 중요한 정보를 이곳에서 관리한다. Hugo는 TOML, YAML, JSON 형식을 모두 지원하며, 파일명은 각각 `hugo.toml`, `hugo.yaml`, `hugo.json`이 될 수 있다.7

이러한 디렉터리 구조의 암묵적인 규칙 덕분에 사용자는 최소한의 설정만으로도 체계적인 사이트를 구축할 수 있다. 예를 들어, `content/posts/my-post.md` 파일은 자동으로 'posts'라는 섹션(또는 타입)으로 분류되고, Hugo는 이 콘텐츠를 렌더링하기 위해 `layouts/posts/single.html` 템플릿을 우선적으로 찾는다.27 이처럼 파일의 위치가 콘텐츠의 종류와 사용할 템플릿을 결정하므로, 각 파일마다 별도의 설정을 명시할 필요가 없어 프로젝트가 커져도 일관성을 유지하기 쉽다.


`hugo.toml` 파일은 사이트의 '신분증'과 같다. 이 파일에 정의된 설정값들은 모든 페이지와 템플릿에서 `.Site`라는 전역 변수를 통해 접근할 수 있으며 34, 사이트 전체의 동작 방식을 제어하는 중앙 통제실 역할을 한다. 프로젝트 초기 단계에서 반드시 확인하고 설정해야 할 핵심 항목은 다음과 같다.

```Ini, TOML
baseURL = 'https://example.org/'
languageCode = 'en-us'
title = 'My New Hugo Site'
theme = 'ananke'
```

- `baseURL`: 배포될 사이트의 최종 루트 URL을 지정한다. 이 값은 사이트맵 생성, RSS 피드, 그리고 모든 절대 경로 링크 생성의 기준이 되므로 매우 중요하다. 특히 GitHub Pages에 배포할 때 이 값을 정확하게 설정하지 않으면 CSS나 이미지 파일 경로가 깨지는 주요 원인이 된다.31
- `languageCode`: 사이트의 기본 언어를 지정한다. 다국어 사이트를 구축할 때 중요한 기준이 된다.
- `title`: 웹 브라우저 탭이나 검색 결과에 표시될 사이트의 제목이다.
- `theme`: 적용할 테마의 이름을 지정한다. 이 이름은 `themes` 디렉터리 내에 있는 테마 폴더의 이름과 일치해야 한다.

이 외에도 수많은 설정 옵션이 있으며, 이는 사용할 테마의 문서나 Hugo 공식 문서를 통해 확인할 수 있다.36



Hugo의 가장 큰 장점 중 하나는 풍부하고 활발한 테마 생태계다. 공식 테마 사이트([themes.gohugo.io](https://themes.gohugo.io/))에는 수백 개의 무료 테마가 등록되어 있으며, 블로그, 포트폴리오, 문서 사이트, 기업 랜딩 페이지 등 다양한 목적에 맞는 테마를 쉽게 찾을 수 있다.2 또한, Gethugothemes와 같은 사이트에서는 전문적인 디자인과 기능을 갖춘 유료 테마도 제공한다.38

이러한 풍부한 선택지 덕분에 디자인에 전문성이 없는 개발자라도 약간의 설정만으로 미학적으로 완성도 높은 웹사이트를 빠르게 구축할 수 있다.


테마를 프로젝트에 통합하는 방법에는 여러 가지가 있지만, **Git Submodule**을 사용하는 것이 가장 권장되는 표준 방식이다. 이 방법은 프로젝트의 소스 코드와 외부 의존성인 테마를 명확하게 분리하여 관리할 수 있게 해준다.

테마를 직접 프로젝트 폴더에 복사해 넣으면, 원본 테마가 업데이트되었을 때 새로운 변경 사항을 수동으로 다시 복사해야 하는 번거로움이 있다. 반면, Git Submodule은 부모 저장소(내 프로젝트) 안에 다른 저장소(테마)의 특정 커밋을 가리키는 포인터를 저장하는 방식이다.40 이를 통해 `git submodule update --remote`와 같은 간단한 명령어로 테마를 항상 최신 버전으로 유지할 수 있다.40 이는 소프트웨어 개발에서 외부 라이브러리를 관리하는 방식과 동일하며, 웹사이트 프로젝트에 체계적인 의존성 관리 개념을 도입하는 것이다.

다음 단계를 따라 테마를 추가한다.

1. **프로젝트 디렉터리로 이동**: 터미널에서 생성한 Hugo 프로젝트의 루트 디렉터리로 이동한다.

   ```Bash
   cd my-blog
   ```

2. **Git 저장소 초기화**: 아직 Git 저장소로 만들지 않았다면 초기화한다.

   ```Bash
   git init
   ```

3. **Git Submodule로 테마 추가**: 공식 테마 사이트에서 마음에 드는 테마를 고른 후, 해당 테마의 GitHub 저장소 URL을 복사한다. 그리고 다음 명령어를 실행한다.

   ```Bash
   git submodule add <테마_저장소_URL> themes/<테마명>
   ```

   예를 들어, 'ananke' 테마를 추가하는 경우:

   ```Bash
   git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
   ```

   이 명령어는 `themes` 디렉터리 아래에 `ananke`라는 폴더를 만들고 해당 테마의 소스 코드를 복제한다.20

4. **`hugo.toml`에 테마 설정**: 프로젝트 루트의 `hugo.toml` 파일을 열고, 방금 추가한 테마를 사용하도록 설정한다.

   ```Ini, TOML
   theme = 'ananke'
   ```


대부분의 잘 만들어진 테마는 `exampleSite`라는 디렉터리를 포함하고 있다.43 이 디렉터리에는 해당 테마의 모든 기능을 보여주는 예제 콘텐츠와 완전한 설정 파일이 들어있다. 이는 테마 제작자가 제공하는 일종의 '모범 답안'이자 '사용 설명서'다.

가장 효율적인 초기 설정 방법은 이 `exampleSite`의 내용을 활용하는 것이다.

1. **설정 파일 복사**: 테마의 `exampleSite` 폴더에 있는 `config.toml`(또는 `hugo.toml`, `config.yaml`) 파일을 내 프로젝트의 루트 디렉터리로 복사하여 기존 파일을 덮어쓴다.
2. **설정 값 수정**: 복사한 설정 파일을 열어 `baseURL`, `title`, `author` 등 개인에 맞게 필요한 정보들을 수정한다.43

이 방법을 사용하면 해당 테마가 제공하는 메뉴, 파라미터 등 모든 설정 옵션을 빠르게 파악하고, 시행착오 없이 초기 설정을 마칠 수 있다. 테마 문서를 꼼꼼히 읽고 각 설정 항목의 의미를 파악하는 것이 중요하다.44



Hugo는 CLI 명령어를 통해 새로운 콘텐츠 파일을 손쉽게 생성할 수 있는 기능을 제공한다. 프로젝트 루트 디렉터리에서 다음 명령어를 실행한다.

```Bash
hugo new content <섹션>/<파일명>.md
```

예를 들어, 'posts' 섹션에 'my-first-post'라는 제목의 글을 작성하려면 다음과 같이 입력한다.

```Bash
hugo new content posts/my-first-post.md
```

이 명령어는 `content/posts/` 디렉터리 안에 `my-first-post.md`라는 마크다운 파일을 생성한다.45 이때 파일의 내용은 `archetypes` 디렉터리에 정의된 템플릿을 기반으로 자동 생성된다.28 Archetype 기능은 모든 게시물이 `title`, `date`, `draft` 등 일관된 메타데이터 구조(Front Matter)를 갖도록 보장하여, 콘텐츠 관리의 효율성을 높인다.


Front Matter는 마크다운 파일의 가장 상단에 위치하며, `---` (YAML) 또는 `+++` (TOML)로 감싸인 영역이다. 이 영역에는 해당 콘텐츠의 제목, 작성일, 태그, 초안 상태 등 다양한 메타데이터가 정의된다.

```YAML
---
title: "My First Post"
date: 2024-07-26T10:00:00+09:00
draft: true
tags: ["hugo", "blogging"]
---

여기에 마크다운으로 본문 내용을 작성한다.
```

- `title`: 페이지의 제목.
- `date`: 페이지의 발행일. 이 날짜를 기준으로 글이 정렬된다.
- `draft`: `true`로 설정되면 해당 페이지는 초안 상태로 간주되어, 일반적인 `hugo` 빌드 명령어 실행 시 최종 결과물에 포함되지 않는다.41
- `tags`, `categories`: 콘텐츠를 분류하기 위한 태그나 카테고리.

Front Matter는 콘텐츠 자체와 메타데이터를 명확하게 분리하는 핵심적인 개념이다. Hugo는 이 메타데이터를 읽어 페이지를 어떻게 처리하고 렌더링할지 결정하며, 템플릿 내에서 이 데이터들을 자유롭게 활용하여 페이지를 동적으로 구성할 수 있다.


콘텐츠를 작성하면서 변경 사항을 실시간으로 확인하기 위해 Hugo의 내장 개발 서버를 사용한다. 프로젝트 루트 디렉터리에서 다음 명령어를 실행한다.

```Bash
hugo server
```

이 명령어를 실행하면 Hugo는 사이트를 메모리상에서 빌드하고, `http://localhost:1313/` 주소로 접속 가능한 경량 웹 서버를 구동한다.41

Hugo 서버의 가장 강력한 기능은 **LiveReload**이다. 서버가 실행 중인 상태에서 `content`, `layouts`, `static` 등 프로젝트 내의 파일을 수정하고 저장하면, Hugo가 이를 즉시 감지하여 사이트를 자동으로 다시 빌드하고 웹 브라우저를 새로고침해준다.6 Hugo의 압도적으로 빠른 빌드 속도와 결합된 이 기능은 개발자에게 즉각적인 피드백을 제공하며, 코드를 수정하고, 저장하고, 브라우저를 새로고침하는 반복적인 작업을 제거하여 개발 경험을 극적으로 향상시킨다.


새로 생성된 콘텐츠는 기본적으로 `draft: true`로 설정되어 있어 일반 `hugo server` 명령어로는 보이지 않는다. 아직 발행 준비가 되지 않은 초안 상태의 글을 포함하여 전체 사이트의 모습을 미리 확인하려면 `-D` (또는 `--buildDrafts`) 플래그를 추가하여 서버를 실행해야 한다.

```Bash
hugo server -D
```

이 명령어를 사용하면 `draft` 상태인 글까지 모두 렌더링되어 로컬 서버에서 확인할 수 있다.20 이 기능은 아직 완성되지 않은 글이 실제 사이트 레이아웃 안에서 어떻게 보일지 미리 확인하고 다듬을 수 있게 해준다. 발행될 콘텐츠와 작업 중인 콘텐츠를 명확히 분리하여 관리할 수 있는 안전장치 역할을 한다. 글이 완성되어 발행할 준비가 되면, 해당 파일의 Front Matter에서 `draft` 값을 `false`로 변경하거나 해당 라인을 삭제하면 된다.



Hugo로 만든 사이트를 GitHub Pages에 배포하려면 먼저 GitHub에 원격 저장소(remote repository)를 생성해야 한다. GitHub Pages는 두 가지 주요 유형의 사이트를 제공하며, 어떤 유형을 선택하느냐에 따라 저장소 이름과 설정 방식이 달라지므로 프로젝트 시작 단계에서 명확히 결정하는 것이 중요하다.

1. **사용자 또는 조직 페이지 (User/Organization Pages)**
   - **저장소 이름**: 반드시 `<username>.github.io` 또는 `<orgname>.github.io` 형식이어야 한다. 여기서 `<username>`은 개인 GitHub 계정 이름, `<orgname>`은 조직 이름이다.53
   - **특징**: 이 유형의 저장소는 계정 또는 조직당 하나만 생성할 수 있다. 저장소의 `main` 브랜치(또는 설정된 기본 브랜치)에 있는 콘텐츠가 사이트의 루트에 직접 배포된다.
   - **URL**: `https://<username>.github.io/`
2. **프로젝트 페이지 (Project Pages)**
   - **저장소 이름**: 특별한 규칙 없이 자유롭게 지정할 수 있다 (예: `my-blog`, `project-docs`).
   - **특징**: 하나의 계정으로 여러 개의 프로젝트 페이지를 만들 수 있다. 배포는 주로 `gh-pages`라는 별도의 브랜치를 사용하거나, 최근에는 GitHub Actions를 통해 이루어진다.54
   - **URL**: `https://<username>.github.io/<repository-name>/`

개인 블로그나 포트폴리오를 만드는 경우, 일반적으로 **사용자 페이지**를 선택하는 것이 URL이 더 깔끔하고 관리하기 편리하다.


`baseURL` 설정은 Hugo 사이트를 GitHub Pages에 성공적으로 배포하는 데 있어 가장 중요하며, 가장 흔하게 실수가 발생하는 부분이다. 이 설정이 잘못되면 웹사이트의 CSS, JavaScript, 이미지 등 모든 정적 자원(static asset)의 경로가 깨져 스타일이 적용되지 않은 날 것의 HTML 페이지만 보이게 된다.35

`baseURL`은 Hugo가 모든 절대 경로를 생성하는 기준점이 된다. 프로젝트 페이지의 경우, URL 경로에 저장소 이름이 하위 디렉터리처럼 포함되는데, `baseURL`에 이 경로를 명시하지 않으면 Hugo는 사이트 루트(`/css/style.css`)를 기준으로 링크를 생성한다. 하지만 실제 리소스의 위치는 `/<repository-name>/css/style.css`이므로, 브라우저는 파일을 찾지 못해 404 오류를 반환한다.

따라서 `hugo.toml` 파일의 `baseURL`을 사이트 유형에 맞게 정확하게 설정해야 한다.

- **사용자 페이지의 경우**:

  ```Ini, TOML
  baseURL = "https://<username>.github.io/"
  ```

  35

- **프로젝트 페이지의 경우**:

  ```Ini, TOML
  baseURL = "https://<username>.github.io/<repository-name>/"
  ```

  35

**주의**: `baseURL` 값은 반드시 프로토콜(`https://`)로 시작하고, 경로의 끝은 슬래시(`/`)로 마무리해야 한다.57


로컬 Hugo 프로젝트를 방금 생성한 GitHub 원격 저장소와 연결해야 한다. 프로젝트 루트 디렉터리에서 다음 명령어들을 순서대로 실행한다.

1. **Git 저장소 초기화** (아직 하지 않았다면):

   ```Bash
   git init
   ```

2. **원격 저장소 추가**: GitHub에서 생성한 저장소의 URL을 복사하여 `origin`이라는 이름의 원격 저장소로 등록한다.

   ```Bash
   git remote add origin https://github.com/<username>/<repository-name>.git
   ```

   58

3. **파일 스테이징 및 커밋**: 프로젝트의 모든 파일을 Git의 관리 대상으로 추가하고 첫 번째 커밋을 생성한다.

   ```Bash
   git add.
   git commit -m "Initial commit"
   ```

4. **원격 저장소로 푸시**: 로컬의 커밋 내역을 GitHub 원격 저장소로 업로드한다.

   ```Bash
   git push -u origin main
   ```

   (기본 브랜치 이름이 `master`인 경우 `main`을 `master`로 변경)

이제 로컬 프로젝트의 소스 코드가 GitHub에 안전하게 백업되었으며, 다음 단계인 자동 배포를 위한 준비가 완료되었다.



GitHub Actions는 GitHub에 내장된 CI/CD(지속적 통합/지속적 배포) 플랫폼이다.59 이를 활용하면 코드 변경 사항이 저장소에 푸시(push)될 때마다 빌드, 테스트, 배포 등의 작업을 자동으로 실행하는 워크플로우(workflow)를 구성할 수 있다.

Hugo 프로젝트에서는 이 기능을 사용하여, 로컬에서 작성한 마크다운 콘텐츠를 `main` 브랜치에 푸시하기만 하면, GitHub의 가상 서버가 자동으로 Hugo 사이트를 빌드하고 그 결과물을 GitHub Pages에 배포하도록 만들 수 있다. 이를 통해 개발자는 반복적인 배포 작업에서 완전히 해방되어 콘텐츠 제작에만 집중할 수 있다.

워크플로우는 프로젝트 루트의 `.github/workflows/` 디렉터리 안에 위치한 YAML 파일로 정의된다.60 이 파일에는 언제(event), 어떤 환경에서(job), 무엇을(step) 실행할지가 명시된다.


GitHub Actions를 사용하여 GitHub Pages를 배포하기 위해서는, 먼저 저장소의 설정을 변경해야 한다.

1. GitHub 저장소 페이지로 이동하여 **Settings** 탭을 클릭한다.
2. 왼쪽 사이드바에서 **Pages** 메뉴를 선택한다.
3. **Build and deployment** 섹션의 **Source** 항목에서 드롭다운 메뉴를 클릭하고, **GitHub Actions**를 선택한다.10

이 설정은 GitHub Pages가 더 이상 특정 브랜치(예: `gh-pages`)의 내용을 그대로 호스팅하는 것이 아니라, GitHub Actions 워크플로우를 통해 생성되고 배포된 결과물(artifact)을 호스팅하도록 하는 중요한 변경이다. 이는 소스 코드와 배포 결과물을 논리적으로 분리하여 더 깔끔하고 전문적인 워크플로우를 가능하게 한다.


이제 자동 배포의 핵심인 워크플로우 파일을 작성할 차례다. 프로젝트 루트에 `.github/workflows/` 디렉터리를 생성하고, 그 안에 `hugo.yml` (또는 원하는 다른 이름) 파일을 만든다. Hugo 공식 문서에서 권장하는 최신 워크플로우는 다음과 같다.10

```YAML
#.github/workflows/hugo.yml

name: Deploy Hugo site to Pages

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

defaults:
  run:
    shell: bash

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.128.2 # 사용할 Hugo 버전 지정
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb

      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive # Git Submodule로 테마를 사용한다면 필수
          fetch-depth: 0

      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5

      - name: Build with Hugo
        env:
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          hugo --gc --minify --baseURL "${{ steps.pages.outputs.base_url }}/"

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path:./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

이 워크플로우 파일의 각 부분을 상세히 분석해 보자.

- `name`: GitHub Actions 탭에 표시될 워크플로우의 이름이다.
- `on`: 워크플로우를 트리거할 이벤트를 정의한다. `push: branches: [main]`은 `main` 브랜치에 코드가 푸시될 때마다 이 워크플로우를 실행하라는 의미다.61
- `permissions`: 워크플로우가 실행될 때 부여받는 권한을 설정한다. `pages: write`와 `id-token: write`는 GitHub Pages에 배포하는 데 필수적인 권한이다.10
- `jobs`: 워크플로우를 구성하는 작업 단위다. 이 예제는 `build`와 `deploy`라는 두 개의 잡(job)으로 구성되어 있다.

**`build` 잡의 단계(steps):**

1. **Install Hugo CLI**: `wget`을 사용해 지정된 버전(`HUGO_VERSION`)의 Hugo Extended 버전을 다운로드하고 설치한다. `peaceiris/actions-hugo` 같은 커뮤니티 액션을 사용하면 이 과정을 더 간결하게 만들 수도 있다.63

2. **Checkout**: `actions/checkout@v4` 액션을 사용하여 GitHub 저장소의 소스 코드를 가상 머신으로 가져온다.65

   `submodules: recursive` 옵션은 `themes` 디렉터리에 Git Submodule로 추가된 테마까지 함께 가져오도록 하는 필수 설정이다.

3. **Setup Pages**: `actions/configure-pages@v5` 액션을 통해 배포 환경을 설정하고, 배포될 URL(`base_url`)과 같은 정보를 출력 변수로 제공한다.

4. **Build with Hugo**: `hugo` 명령어를 실행하여 정적 사이트 파일을 생성한다. `--minify` 옵션은 HTML, CSS, JS 파일을 압축하여 성능을 최적화한다. 여기서 `baseURL`을 `${{ steps.pages.outputs.base_url }}/`로 동적으로 설정하는 것이 핵심이다.

5. **Upload artifact**: `actions/upload-pages-artifact@v3` 액션을 사용하여 빌드 결과물인 `public` 디렉터리를 GitHub Pages가 사용할 수 있는 아티팩트(artifact)로 업로드한다.62

**`deploy` 잡의 단계(steps):**

- `needs: build`: 이 잡이 실행되기 전에 `build` 잡이 반드시 성공적으로 완료되어야 함을 명시하는 의존성 설정이다.
- **Deploy to GitHub Pages**: `actions/deploy-pages@v4` 액션을 사용하여 `build` 잡에서 업로드한 아티팩트를 GitHub Pages에 최종적으로 배포한다.62

이처럼 빌드와 배포 잡을 분리하는 것은 CI/CD의 모범 사례다. 빌드 잡은 소스 코드를 실행 가능한 결과물(아티팩트)로 만드는 책임만 지고, 배포 잡은 그 결과물을 실제 서비스 환경에 배포하는 책임만 진다. 이러한 관심사의 분리(Separation of Concerns) 원칙은 워크플로우의 명확성을 높이고, 향후 테스트 잡 등을 추가할 때 유연한 파이프라인 확장을 가능하게 한다.

이 YAML 파일을 프로젝트에 추가하고 GitHub에 푸시하면, 이제부터 `main` 브랜치에 변경 사항을 푸시할 때마다 자동으로 웹사이트가 빌드되고 배포될 것이다.



학술적이거나 기술적인 내용을 다루는 웹사이트에서는 복잡한 수학 수식을 표현해야 할 필요가 있다. LaTeX는 이를 위한 표준적인 마크업 언어지만, 웹 브라우저는 기본적으로 LaTeX 문법을 해석하지 못한다. 이 문제를 해결하기 위해 MathJax나 KaTeX와 같은 JavaScript 라이브러리를 사용한다.69

이 라이브러리들은 웹 페이지가 로드될 때, 페이지 내의 특정 구분자(예: `$$...$$`)로 둘러싸인 LaTeX 코드를 찾아, 이를 브라우저가 표시할 수 있는 아름다운 수학 기호(SVG 또는 HTML+CSS)로 변환해준다. 서버는 순수한 텍스트 형태의 LaTeX 코드를 전달하고, 실제 렌더링은 클라이언트 측(사용자의 브라우저)에서 동적으로 이루어지는 것이다. KaTeX는 MathJax에 비해 렌더링 속도가 매우 빠르다는 장점이 있어 최근 널리 사용된다.71


Hugo에서 KaTeX를 원활하게 사용하려면, Hugo의 마크다운 렌더러가 LaTeX 문법을 훼손하지 않고 그대로 브라우저에 전달하도록 설정하는 과정이 필요하다. Hugo 공식 문서에서 권장하는 최신 방법은 다음과 같다.70

**1단계: Goldmark Passthrough 활성화**

Hugo의 기본 마크다운 렌더러인 Goldmark가 LaTeX 구분자 사이의 내용을 처리하지 않고 그대로 통과(passthrough)시키도록 설정한다. 이는 마크다운 파서가 `_`를 기울임꼴 태그(`<em>`)로 변환하는 등 LaTeX 문법을 의도치 않게 변경하는 것을 방지하기 위함이다. `hugo.toml` 파일에 다음 설정을 추가한다.

```Ini, TOML
[markup.goldmark.extensions.passthrough]
  enable = true
  [markup.goldmark.extensions.passthrough.delimiters]
    block = [['$$', '$$']]
    inline = [['$', '$']]
```

**2단계: KaTeX 로드를 위한 Partial 템플릿 생성**

KaTeX 라이브러리의 CSS와 JavaScript 파일을 웹 페이지에 삽입하는 역할을 할 partial 템플릿을 생성한다. `layouts/_partials/` 디렉터리 안에 `katex.html` 파일을 만들고 다음 내용을 추가한다.

```HTML
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/katex.min.css" integrity="sha384-n8MVd4RsNIU0KOVEMckMnHllPN8yLIJXXrqDHWw9vanRNkVCo+GMSyEeTe2KLgaK" crossorigin="anonymous">
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/katex.min.js" integrity="sha384-XjKyOOlGwcjNTAIQHIpgOno0Hl1YQqzUOEleOLALmuqehneUG+vnGctmUbKyJ//g" crossorigin="anonymous"></script>
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/contrib/auto-render.min.js" integrity="sha384-+VBxd3r6XgURycqtZ117nYw44OOcIax56Z4dCRWbxyPt0Koah1uHoK0o4+/RRE05" crossorigin="anonymous"
    onload="renderMathInElement(document.body, {
        delimiters: [
            {left: '$$', right: '$$', display: true},
            {left: '$', right: '$', display: false}
        ]
    });"></script>
```

71

**3단계: 기본 템플릿에 Partial 조건부 포함**

사이트의 모든 페이지에 공통적으로 적용되는 기본 레이아웃 템플릿(보통 `layouts/_default/baseof.html` 또는 테마의 유사한 파일)을 연다. `<head>` 태그가 닫히기 직전에 다음 코드를 삽입한다.

```html
{{ if.Params.math }}
  {{ partial "katex.html". }}
{{ end }}
```

70

이 코드는 각 페이지의 Front Matter에 `math: true`라는 파라미터가 설정된 경우에만 `katex.html` partial을 포함시킨다. 이 방식은 수식이 없는 페이지에 불필요한 JavaScript 라이브러리를 로드하지 않도록 하여 사이트 전체의 로딩 성능을 최적화하는 데 매우 중요하다.

이제 수식을 사용하고 싶은 마크다운 파일의 Front Matter에 `math: true`를 추가하면 해당 페이지에서 LaTeX 수식을 자유롭게 사용할 수 있다.


- **인라인 수식**: 문장 내에 수식을 포함할 때는 `$` 구분자를 사용한다.

  - 마크다운 소스: `아인슈타인의 질량-에너지 등가 원리는 $E=mc^2$로 표현된다.`
  - 렌더링 결과: 아인슈타인의 질량-에너지 등가 원리는 `$E=mc^2$`로 표현된다.

- **블록 수식**: 수식을 별도의 단락으로 표시할 때는 `$$` 구분자를 사용한다.

  - 마크다운 소스:

    ```latex
    $$
    \int_{-\infty}^{\infty} e^{-x^2} dx = \sqrt{\pi}
    $$
    ```

  - 렌더링 결과:

    ```latex
    $$
    \int_{-\infty}^{\infty} e^{-x^2} dx = \sqrt{\pi}
    $$
    ```

- **Markdown 테이블 내 수식 작성**: Markdown 테이블 내에서 `|` 문자는 열(column)을 구분하는 특수 문자로 해석된다. 따라서 조건부 확률 `$P(A|B)$`나 절대값 `$\lvert x \rvert$`와 같이 `|` 기호가 포함된 LaTeX 수식을 테이블 안에 직접 사용하면 테이블 구조가 깨지게 된다.73

  이 문제를 해결하는 가장 안정적인 방법은 `|` 문자 대신, 동일한 시각적 표현을 가지면서도 Markdown 파서에게는 일반 텍스트로 인식되는 LaTeX 명령어 `\rvert`, `\lvert`, `\vert` 등을 사용하는 것이다.75 이 명령어들은 Markdown 파싱 단계를 무사히 통과한 후, 브라우저의 KaTeX 렌더러에 의해 올바른 수직선 기호로 변환된다.


| 설명            | 문제되는 마크다운 소스 | 해결된 마크다운 소스   | 렌더링 결과         |
| --------------- | ---------------------- | ---------------------- | ------------------- |
| **조건부 확률** | `$| P(A|B) $`          | `$| P(A\rvert B) $`    | `$P(A\rvert B)$`    |
| **절대값**      | `$| |x| $`             | `$| \lvert x \rvert $` | `$\lvert x \rvert$` |
| **노름(Norm)**  | `$| | \mathbf{v} | $`  | `$| | \mathbf{v} | $`  | `$| \mathbf{v} |$`  |

이 표는 흔히 발생하는 문제를 명확히 보여주고 실용적인 해결책을 제시함으로써, 기술 문서 작성 시 마주할 수 있는 구체적인 어려움을 해결하는 데 도움을 준다.


이 가이드는 정적 사이트 생성기 Hugo와 GitHub Pages를 결합하여 현대적이고 자동화된 웹사이트를 구축하는 전 과정을 상세히 다루었다. 로컬 개발 환경 구축부터 Git과 Hugo 설치, 프로젝트 구조 이해, 테마 적용, 콘텐츠 작성, 그리고 GitHub Actions를 이용한 완전 자동화된 배포 파이프라인 구축까지의 여정을 살펴보았다. 또한, KaTeX를 연동하여 기술 문서에 필수적인 LaTeX 수식을 렌더링하는 고급 기법까지 다루었다.

이 과정을 통해 독자는 단순히 웹사이트 하나를 완성한 것을 넘어, 다음과 같은 핵심적인 현대 웹 개발의 개념과 기술을 습득했다.

- **정적 사이트 아키텍처**: 속도, 보안, 단순성을 극대화하는 웹 개발 패러다임을 이해했다.
- **버전 관리 시스템(Git)**: 콘텐츠와 코드를 체계적으로 관리하고 변경 이력을 추적하는 방법을 익혔다.
- **CI/CD 파이프라인**: `git push`만으로 빌드와 배포가 자동으로 이루어지는 효율적인 워크플로우를 구축했다.
- **인프라로서의 코드(Infrastructure as Code)**: YAML 파일을 통해 배포 환경을 코드로 정의하고 관리하는 경험을 했다.

웹사이트 구축은 끝이 아니라 새로운 시작이다. 이 가이드를 통해 마련된 견고한 기반 위에서 당신의 웹사이트를 더욱 풍성하게 발전시킬 수 있는 몇 가지 다음 단계들을 제안한다.

- **커스텀 도메인 연결**: `username.github.io` 주소 대신 개인 소유의 도메인(예: `yourdomain.com`)을 연결하여 전문성을 높일 수 있다. GitHub Pages는 커스텀 도메인 설정과 무료 HTTPS 인증서 발급을 지원한다.57
- **댓글 기능 추가**: 정적 사이트는 자체적으로 댓글 기능을 가질 수 없지만, Utterances, Giscus, Disqus와 같은 외부 서비스를 연동하여 독자와의 소통 창구를 마련할 수 있다. 특히 Utterances와 Giscus는 GitHub 이슈나 토론 기능을 활용하므로 GitHub 생태계와 자연스럽게 통합된다.79
- **검색 엔진 최적화(SEO) 및 분석**: Google Analytics를 연동하여 방문자 통계를 분석하고 5, Hugo의 SEO 관련 설정을 최적화하여 검색 결과에 더 잘 노출되도록 할 수 있다.
- **테마 커스터마이징**: `layouts` 디렉터리에 자신만의 템플릿 파일을 추가하여 기존 테마의 디자인을 부분적으로 수정하거나 완전히 새로운 디자인을 구현할 수 있다.

Hugo와 GitHub Pages 생태계는 지금도 활발하게 발전하고 있다. 새로운 기능이 추가되고 더 나은 방법론이 등장할 때, Hugo 공식 문서와 GitHub Actions 문서는 가장 신뢰할 수 있는 정보의 원천이 될 것이다. 또한, 활발한 커뮤니티 포럼을 통해 문제 해결에 도움을 받거나 새로운 아이디어를 얻을 수 있다. 이제 당신만의 지식과 경험을 세상과 공유할 준비가 되었다.


1. window 환경에서 hugo로 원하는 github.io 기술 블로그 만들기, 8월 24, 2025에 액세스, https://github.com/JeHa00/jeha00.github.io
2. 초보자 Hugo 블로그 구축기 - 202 Blue Note, 8월 24, 2025에 액세스, https://key4920.github.io/docs/ETC/Hugo/first_hugo_blog/
3. 블로그 구축기 1 (Hugo + github.io) - ialy's blog, 8월 24, 2025에 액세스, https://ialy1595.github.io/post/blog-construct-1/
4. [Github] github.io? github page는 뭘까? - Gorilla with Lion - 티스토리, 8월 24, 2025에 액세스, https://likelion-kgu.tistory.com/31
5. Hugo 개요 / 설치하기 - devkuma, 8월 24, 2025에 액세스, https://www.devkuma.com/docs/hugo/overview/
6. GH-Pages with HUGO - mo0oom - Han's blog, 8월 24, 2025에 액세스, https://mo0om.com/2021/gh-pages-with-hugo/
7. Hugo + Netlify + CMS 블로그 만들기 - -난 잠은 죽어서 자-, 8월 24, 2025에 액세스, [https://asd147asd147.netlify.app/p/hugo-netlify-cms-%EB%B8%94%EB%A1%9C%EA%B7%B8-%EB%A7%8C%EB%93%A4%EA%B8%B0/](https://asd147asd147.netlify.app/p/hugo-netlify-cms-블로그-만들기/)
8. Github Pages + Hugo를 사용한 블로그 구축기 - Billo Park, 8월 24, 2025에 액세스, https://blog.billo.io/devposts/blog_github_page/
9. Deploying a Hugo Website to GitHub Pages Using GitHub Actions - Curious Mints, 8월 24, 2025에 액세스, https://www.curiousmints.com/deploying-a-hugo-website-to-github-pages-using-github-actions/
10. Host on GitHub Pages - Hugo, 8월 24, 2025에 액세스, https://gohugo.io/host-and-deploy/host-on-github-pages/
11. Install Git | Atlassian Git Tutorial, 8월 24, 2025에 액세스, https://www.atlassian.com/git/tutorials/install-git
12. Git Guides - install git · GitHub, 8월 24, 2025에 액세스, https://github.com/git-guides/install-git
13. Chapter 6 Install Git - Happy Git and GitHub for the useR, 8월 24, 2025에 액세스, https://happygitwithr.com/install-git.html
14. Windows - Hugo, 8월 24, 2025에 액세스, https://gohugo.io/installation/windows/
15. Install Hugo on Ubuntu - Igor Baiborodine, 8월 24, 2025에 액세스, https://www.kiroule.com/article/install-hugo-on-ubuntu/
16. How to install Hugo on Windows - Documentation Portal, 8월 24, 2025에 액세스, https://tw-docs.com/docs/static-site-generators/hugo-install/
17. Install Hugo, 8월 24, 2025에 액세스, https://gohugobrasil.netlify.app/getting-started/installing/
18. macOS - Hugo, 8월 24, 2025에 액세스, https://gohugo.io/installation/macos/
19. Linux - Hugo, 8월 24, 2025에 액세스, https://gohugo.io/installation/linux/
20. Quick start - Hugo, 8월 24, 2025에 액세스, https://gohugo.io/getting-started/quick-start/
21. Creating A New Site | Hugo | - Giraffe Academy, 8월 24, 2025에 액세스, https://www.giraffeacademy.com/static-site-generators/hugo/hugo-directory-structure/
22. Directory Structure - Hugo - Netlify, 8월 24, 2025에 액세스, https://gohugobrasil.netlify.app/getting-started/directory-structure/
23. Archetypes - Hugo, 8월 24, 2025에 액세스, https://gohugo.io/content-management/archetypes/
24. Content organization - Hugo, 8월 24, 2025에 액세스, https://gohugo.io/content-management/organization/
25. Content Organization - Hugo - Netlify, 8월 24, 2025에 액세스, https://gohugobrasil.netlify.app/content-management/organization/
26. Content Organization | Hugo, 8월 24, 2025에 액세스, https://www.engino.co.uk/form-elements/organization/
27. Hugo's Directory Structure Explained - Jake Wiesler, 8월 24, 2025에 액세스, https://www.jakewiesler.com/blog/hugo-directory-structure
28. How to build a static site with Hugo | Hygraph, 8월 24, 2025에 액세스, https://hygraph.com/blog/hugo-static-site
29. hugo server not rendering images from static folder - Stack Overflow, 8월 24, 2025에 액세스, https://stackoverflow.com/questions/48023746/hugo-server-not-rendering-images-from-static-folder
30. Installing & Using Themes | Hugo | - Giraffe Academy, 8월 24, 2025에 액세스, https://www.giraffeacademy.com/static-site-generators/hugo/installing-using-themes/
31. Introduction - Hugo, 8월 24, 2025에 액세스, https://gohugo.io/configuration/introduction/
32. Configure Hugo | Hugo中文网, 8월 24, 2025에 액세스, https://hugo.zcopy.site/getting-started/configuration/
33. A Hugo Survival Guide - GitHub Gist, 8월 24, 2025에 액세스, https://gist.github.com/janert/4e22671044ffb06ee970b04709dd7d81
34. [Hugo] Hugo 문법 정리 - velog, 8월 24, 2025에 액세스, [https://velog.io/@doheek2/Hugo-Hugo-%EB%AC%B8%EB%B2%95-%EC%A0%95%EB%A6%AC](https://velog.io/@doheek2/Hugo-Hugo-문법-정리)
35. Hugo site doesn't display properly when deployed to github pages - Stack Overflow, 8월 24, 2025에 액세스, https://stackoverflow.com/questions/62294887/hugo-site-doesnt-display-properly-when-deployed-to-github-pages
36. Configuration - Hugo, 8월 24, 2025에 액세스, https://gohugo.io/configuration/
37. Hugo Themes, 8월 24, 2025에 액세스, https://themes.gohugo.io/
38. Gethugothemes: Premium Hugo Themes for Blazing Fast Website, 8월 24, 2025에 액세스, https://gethugothemes.com/
39. What are your go-to resources for finding Hugo Themes? : r/gohugo - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/gohugo/comments/1i5iyhz/what_are_your_goto_resources_for_finding_hugo/
40. Managing a Hugo theme using git submodules - Jan Tytgat, 8월 24, 2025에 액세스, https://jantytgat.com/posts/hugo/themes/managing-a-hugo-theme-with-git-submodules/
41. [Hugo] Quick Start - Dog foot print - 티스토리, 8월 24, 2025에 액세스, https://hokeydokey.tistory.com/150
42. A Practical Guide to Installing a Hugo Theme - DEV Community, 8월 24, 2025에 액세스, https://dev.to/daleymottley/a-practical-guide-to-installing-a-hugo-theme-53be
43. Create a static blog with Hugo and GitHub Pages - Marie Cruz, 8월 24, 2025에 액세스, http://www.testingwithmarie.com/posts/20241126-create-a-static-blog-with-hugo/
44. Hugo와 Github page로 블로그 구축 - Gromit.Blog, 8월 24, 2025에 액세스, https://gowoonsori.com/blog/projects/hugo/start/
45. Creating New Posts with Hugo | Blog about anything related to my learnings, 8월 24, 2025에 액세스, https://reshmeeauckloo.com/posts/hugo_create-new-post/
46. Add New Content | Documentation Contributor Guide, 8월 24, 2025에 액세스, https://docs.nvidia.com/networking-ethernet-software/contributor-guide/Make-a-Larger-Contribution/Adding_New_Content/
47. Draft - Hugo, 8월 24, 2025에 액세스, https://gohugo.io/methods/page/draft/
48. Basic usage - Hugo, 8월 24, 2025에 액세스, https://gohugo.io/getting-started/usage/
49. hugo-server(1) — Arch manual pages, 8월 24, 2025에 액세스, https://man.archlinux.org/man/hugo-server.1.en
50. Basic Usage - Hugo - Netlify, 8월 24, 2025에 액세스, https://gohugobrasil.netlify.app/getting-started/usage/
51. hugo server, 8월 24, 2025에 액세스, https://gohugo.io/commands/hugo_server/
52. Discreet Drafts in Hugo - Zachary Betz, 8월 24, 2025에 액세스, https://zwbetz.com/discreet-drafts-in-hugo/
53. Hugo — Github Pages 연동. 이전 블로그에서 가져온 글입니다. | by Kang Daniel | Daniel's Dev Blog | Medium, 8월 24, 2025에 액세스, [https://medium.com/daniel-dev/hugo-github-pages-%EC%97%B0%EB%8F%99-dad16c123aa](https://medium.com/daniel-dev/hugo-github-pages-연동-dad16c123aa)
54. GitHub의 페이지 기능 이용하기 - dogfeet, 8월 24, 2025에 액세스, https://dogfeet.github.io/articles/2012/github-pages.html
55. Quickstart for GitHub Pages - GitHub Docs, 8월 24, 2025에 액세스, https://docs.github.com/en/pages/quickstart
56. Deploying as a GitHub Project Page based site needs canonifyurls = true · Issue #7714 · gohugoio/hugo, 8월 24, 2025에 액세스, https://github.com/gohugoio/hugo/issues/7714
57. How to Create and Host a Website with Hugo and GitHub Pages | by Magsther - Medium, 8월 24, 2025에 액세스, https://medium.com/@magstherdev/github-pages-hugo-86ae6bcbadd
58. [Git] Github Pages로 리액트 호스팅 - velog, 8월 24, 2025에 액세스, [https://velog.io/@tngusro/Git-Github-Pages%EB%A1%9C-%EB%A6%AC%EC%95%A1%ED%8A%B8-%ED%98%B8%EC%8A%A4%ED%8C%85](https://velog.io/@tngusro/Git-Github-Pages로-리액트-호스팅)
59. Quickstart for GitHub Actions, 8월 24, 2025에 액세스, https://docs.github.com/en/actions/get-started/quickstart
60. Understanding GitHub Actions, 8월 24, 2025에 액세스, https://docs.github.com/articles/getting-started-with-github-actions
61. Workflow syntax for GitHub Actions, 8월 24, 2025에 액세스, https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax
62. Building and deploying a custom site using GitHub Actions and GitHub Pages, 8월 24, 2025에 액세스, https://til.simonwillison.net/github-actions/github-pages
63. Hugo setup · Actions · GitHub Marketplace, 8월 24, 2025에 액세스, https://github.com/marketplace/actions/hugo-setup
64. How to Set Up Hugo in GitHub Actions - Workflow Hub - CICube, 8월 24, 2025에 액세스, https://cicube.io/workflow-hub/peaceiris-actions-hugo/
65. GitHub Actions Checkout: Complete Guide to Faster CI/CD Workflows - Devzery, 8월 24, 2025에 액세스, https://www.devzery.com/post/github-actions-complete-guide-to-actions-checkout
66. How to use the checkout action in GitHub Actions - Graphite, 8월 24, 2025에 액세스, https://graphite.dev/guides/github-actions-checkout
67. How To Deploy Pages on GitHub Using Actions | by Marcelo Paternostro | Medium, 8월 24, 2025에 액세스, https://medium.com/@mpaternostro/how-to-deploy-pages-on-github-using-actions-a9281d03b345
68. GitHub Action to publish artifacts to GitHub Pages for deployments, 8월 24, 2025에 액세스, https://github.com/actions/deploy-pages
69. MathJax Support - Hugo - GitHub Pages, 8월 24, 2025에 액세스, https://bwaycer.github.io/hugo_tutorial.hugo/tutorials/mathjax/
70. Mathematics in Markdown - Hugo, 8월 24, 2025에 액세스, https://gohugo.io/content-management/mathematics/
71. KaTeX in Hugo - feature, 8월 24, 2025에 액세스, https://discourse.gohugo.io/t/katex-in-hugo/43274
72. Render LaTeX with KaTex in Hugo Blog - Edwin Kofler, 8월 24, 2025에 액세스, https://edwinkofler.com/posts/render-latex-with-katex-in-hugo-blog/
73. Pipe character within $..$ in markdown table breaks equation across columns #185 - GitHub, 8월 24, 2025에 액세스, https://github.com/atom-community/markdown-preview-plus/issues/185
74. Pipe Problems in tables: Math\Latex, Inline code and separator - Obsidian Forum, 8월 24, 2025에 액세스, https://forum.obsidian.md/t/pipe-problems-in-tables-math-latex-inline-code-and-separator/3692
75. Vertical bar symbol within a markdown table - Stack Overflow, 8월 24, 2025에 액세스, https://stackoverflow.com/questions/49809122/vertical-bar-symbol-within-a-markdown-table
76. Using \big| and \right| versus \bigr\rvert and \right\rvert - TeX - LaTeX Stack Exchange, 8월 24, 2025에 액세스, https://tex.stackexchange.com/questions/294753/using-big-and-right-versus-bigr-rvert-and-right-rvert
77. Deploy Hugo Blog to Github Pages via Github Actions w/ a Custom Domain - YouTube, 8월 24, 2025에 액세스, https://www.youtube.com/watch?v=_QSr2_pxIJs
78. Hugo and GitHub Pages Tutorial - Alex Zweber Leslie, 8월 24, 2025에 액세스, https://azleslie.com/projects/hugo-tutorial/
79. Hugo로 github.io 블로그 만들기, 8월 24, 2025에 액세스, https://ryan-han.com/post/dev/creating_static_blog/
80. Hugo로 github.io 블로그 만들기, 8월 24, 2025에 액세스, https://github.com/Integerous/Integerous.github.io

