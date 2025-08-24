# GitHub Pages를 활용한 전문가용 기술 블로그 구축 및 운영
[Github Pages](./index.md)



GitHub Pages는 GitHub 저장소(Repository)에 포함된 정적 파일들, 즉 HTML, CSS, JavaScript 등을 웹 페이지로 직접 호스팅하고 게시해주는 무료 서비스다.1 이 서비스는 개발자들이 자신의 프로젝트, 포트폴리오, 기술 블로그 등을 손쉽게 웹에 공개할 수 있는 강력한 도구로 자리 잡았다.3 저장소의 최대 용량은 1GB로 제한되므로 대규모 프로젝트보다는 개인 블로그나 문서 사이트 운영에 최적화되어 있다.5

이 서비스의 핵심은 '정적(static) 호스팅'이라는 개념에 있다. 정적 사이트는 서버 측에서 데이터베이스 조회나 복잡한 로직 처리 없이, 미리 생성된(pre-built) 파일들을 사용자에게 그대로 전달하는 방식을 의미한다.6 이는 동적 사이트와 대비되는 개념으로, 다음과 같은 본질적인 장점을 가진다. 첫째, 서버 측 취약점이 없어 보안성이 매우 높다. 둘째, 복잡한 연산 과정이 생략되므로 사용자에게 콘텐츠를 전달하는 속도가 매우 빠르다. 셋째, 모든 콘텐츠와 웹사이트 구조가 Git 저장소 내 파일로 관리되므로 버전 관리가 용이하며, 변경 이력을 추적하고 이전 상태로 복원하는 것이 간단하다.

과거 개인 웹사이트나 블로그를 운영하기 위해서는 도메인 구매, 웹 호스팅 서비스 결제, 데이터베이스 및 서버 백엔드 관리 등 상당한 기술적, 재정적 장벽이 존재했다.6 GitHub Pages는 이러한 장벽을 제거함으로써 기술 퍼블리싱의 민주화를 이끌었다. 개발자와 연구자에게 이미 익숙한 도구인 Git과 GitHub 플랫폼을 기반으로 무료 호스팅을 제공함으로써, 누구나 최소한의 비용과 노력으로 자신의 지식을 공유하고, 포트폴리오를 구축하며, 오픈소스 프로젝트 문서를 배포할 수 있는 환경을 마련한 것이다.5 이는 단순한 기능을 넘어, 수많은 기술 전문가들이 자신의 브랜드를 만들고 개방형 지식 생태계에 기여하도록 촉진하는 강력한 원동력이 되었다.


GitHub Pages의 작동 메커니즘은 Git 워크플로와 긴밀하게 통합된 자동화된 CI/CD(Continuous Integration/Continuous Deployment) 파이프라인에 기반한다. 사용자가 특정 브랜치(예: `main` 또는 `gh-pages`)에 변경 사항을 `git push` 명령어로 전송하면, 이 이벤트가 GitHub Actions 워크플로를 자동으로 트리거한다.3

이 워크플로는 사전에 정의된 작업들을 순차적으로 실행한다. 먼저, 저장소의 소스 코드를 받아온다. 만약 Jekyll과 같은 정적 사이트 생성기(Static Site Generator, SSG)를 사용한다면, 마크다운(`*.md`) 파일과 템플릿 파일들을 HTML, CSS, JS 파일로 변환하는 '빌드(build)' 과정을 수행한다. 빌드가 완료되면, 생성된 정적 파일들을 웹 서버에 배포할 수 있는 형태로 패키징하여 지정된 호스팅 환경에 게시한다. 이 모든 과정은 사용자의 개입 없이 자동으로 이루어지며, 평균적으로 수 분 정도 소요될 수 있다.3

사용자는 GitHub 저장소의 `Actions` 탭에서 이 배포 과정의 진행 상황을 실시간으로 확인할 수 있다.3 워크플로가 성공적으로 완료되면, 배포된 웹사이트는 루트 디렉터리의 `index.html` 파일을 기본 페이지로 인식하여 사용자에게 제공한다.9 이처럼 

`git push`라는 익숙한 명령 하나만으로 웹사이트의 빌드부터 배포까지 전 과정이 자동화되는 것은 GitHub Pages의 가장 큰 특징이다. 이는 콘텐츠 업데이트 주기를 획기적으로 단축시키며, 모든 변경 사항이 Git 커밋 기록으로 남기 때문에 안정적인 운영과 문제 발생 시 빠른 추적을 가능하게 한다.


GitHub Pages는 목적과 URL 구조에 따라 두 가지 주요 유형으로 구분된다: 사용자/조직 페이지와 프로젝트 페이지.10 이 두 유형의 차이를 명확히 이해하는 것은 블로그 구축 초기 단계에서 가장 중요한 결정 사항 중 하나이며, 이는 향후 설정과 운영에 직접적인 영향을 미친다.

사용자/조직 페이지 (User/Organization Pages)

사용자 페이지는 개인 또는 조직의 공식적인 대표 사이트를 위해 설계되었다. 이 페이지를 생성하기 위해서는 저장소의 이름이 반드시 username.github.io 또는 organization-name.github.io라는 엄격한 명명 규칙을 따라야 한다.3 계정당 단 하나만 생성할 수 있으며, 일반적으로 `main` 브랜치의 루트 디렉터리에 있는 파일들이 호스팅된다. 배포가 완료되면, `https://username.github.io`라는 루트 도메인 형태의 깔끔한 URL을 갖게 된다.5

프로젝트 페이지 (Project Pages)

프로젝트 페이지는 개별 프로젝트의 문서, 데모 사이트, 또는 부가적인 블로그 등을 위해 사용된다. 일반적인 저장소에서 생성할 수 있으며, 저장소 이름에 특별한 제약이 없다. 전통적으로 프로젝트 페이지의 소스 코드는 gh-pages라는 별도의 브랜치에 관리되었으나, 현재는 main 브랜치의 특정 폴더(예: /docs)를 소스로 지정하는 것도 가능하다.10 프로젝트 페이지의 가장 큰 특징은 URL 구조다. 배포된 사이트는 사용자 페이지의 하위 경로(subpath) 형태로 제공되며, URL은 `https://username.github.io/repository-name`과 같은 형식을 띤다.3

이 두 페이지 유형의 선택은 단순히 URL의 미관상의 차이에 그치지 않는다. 이는 정적 사이트 생성기(SSG)의 핵심 설정 값에 직접적인 영향을 미치는 기술적 결정이다. 예를 들어, 프로젝트 페이지의 URL은 `/repository-name`이라는 하위 경로를 포함한다. 따라서 SSG는 CSS, JavaScript, 이미지와 같은 모든 정적 자산(asset)의 경로 앞에 이 하위 경로를 붙여주어야 한다. Jekyll에서는 이 설정을 `_config.yml` 파일의 `baseurl` 변수로 제어한다. 만약 사용자가 프로젝트 페이지를 만들면서 `baseurl`을 `"/repository-name"`으로 설정하지 않으면, 모든 자산 링크는 `https://username.github.io/assets/css/style.css`와 같이 잘못된 경로를 가리키게 되어 404 오류를 발생시키고 사이트의 스타일과 기능이 완전히 깨지게 된다. 반면, 사용자 페이지는 루트 도메인을 사용하므로 `baseurl`을 빈 문자열이나 `"/"`로 설정해야 한다. 이처럼 페이지 유형 선택은 SSG 설정의 정확성을 좌우하는 결정적인 요인이므로, 프로젝트 시작 단계에서 명확히 인지하고 그에 맞는 설정을 적용해야 한다.



Jekyll은 Ruby 프로그래밍 언어를 기반으로 동작하는 정적 사이트 생성기다.11 따라서 GitHub Pages에서 제공하는 자동 빌드 기능 외에, 로컬 컴퓨터에서 콘텐츠를 작성하고 디자인을 실시간으로 확인하며 개발하기 위해서는 Ruby 및 관련 개발 도구(RubyGems, Bundler)를 설치하는 과정이 반드시 선행되어야 한다.11 GitHub Pages 문서에서는 Jekyll을 마치 완벽하게 통합된 단순한 솔루션처럼 소개하지만, 실제 로컬 개발 과정은 Ruby라는 생소한 생태계와의 만남이며, 이는 특히 비-Ruby 개발자에게 상당한 진입 장벽으로 작용할 수 있다.14 운영체제별 특성을 고려한 정확한 설치 절차는 다음과 같다.


Windows는 Jekyll의 공식 지원 플랫폼이 아니기 때문에 설치 과정이 상대적으로 복잡하며, 몇 가지 추가적인 단계를 요구한다.13

1. **Ruby+Devkit 설치**: `RubyInstaller for Windows` 공식 사이트에서 `Ruby+Devkit` 버전의 설치 프로그램을 다운로드한다.16 Devkit은 C/C++로 작성된 Ruby 라이브러리(Gem)의 네이티브 확장을 Windows 환경에서 빌드하는 데 필수적인 도구 모음이다. 설치 마법사를 진행할 때, 'Add Ruby executables to your PATH' 옵션을 반드시 체크하여 어느 위치에서든 

   `ruby` 명령어를 실행할 수 있도록 한다.13

2. **MSYS2 개발 도구 체인 설치**: 설치 마지막 단계에서 'Run 'ridk install' to set up MSYS2...' 옵션이 나타나면 이를 실행한다. 터미널 창이 열리면, 제시되는 옵션 중 MSYS2와 MINGW 개발 도구 체인을 설치하는 항목(보통 3번 또는 기본 옵션)을 선택하여 엔터를 누른다.16 이 과정은 Jekyll이 의존하는 일부 Gem들을 컴파일하는 데 필요한 환경을 구성한다.

3. **Jekyll 및 Bundler 설치**: 설치가 완료된 후, 새로운 명령 프롬프트(Command Prompt) 창을 관리자 권한으로 연다. 다음 명령어를 입력하여 Jekyll과 Bundler를 설치한다. Bundler는 프로젝트별로 사용되는 Gem의 버전을 관리하여 의존성 충돌 문제를 방지하는 필수 도구다.16

   ```
   gem install jekyll bundler
   ```

4. **설치 확인**: 설치가 완료되면 다음 명령어를 통해 Jekyll 버전이 정상적으로 출력되는지 확인한다.11

   ```
   jekyll -v
   ```

   만약 명령어를 찾을 수 없다는 오류가 발생하면, 시스템을 재부팅하거나 환경 변수(PATH) 설정을 다시 확인해야 한다. 이처럼 Windows에서의 Jekyll 설치는 여러 단계와 잠재적인 컴파일 이슈를 내포하고 있어, 초심자에게는 상당한 어려움이 될 수 있다. 대안으로, WSL(Windows Subsystem for Linux)을 설치하고 그 안에서 Ubuntu 환경을 구성하여 Jekyll을 설치하는 것도 안정적인 방법 중 하나다.16


macOS는 Unix 기반 운영체제로 Ruby 개발 환경 구성이 비교적 용이하지만, 시스템에 내장된 Ruby를 직접 사용하는 것은 피해야 한다. 시스템 Ruby는 운영체제와 깊게 연관되어 있어, Gem을 설치할 때 권한 문제가 발생하거나 시스템 안정성을 해칠 수 있다.19 따라서 패키지 관리자를 통해 별도의 Ruby 버전을 설치하고 관리하는 것이 권장된다.

1. **Homebrew 설치**: Homebrew는 macOS용 패키지 관리자로, 각종 개발 도구를 손쉽게 설치할 수 있게 해준다. 터미널을 열고 Homebrew 공식 사이트의 설치 스크립트를 실행하여 설치한다.20

2. **Xcode Command-Line Tools 설치**: 일부 Gem은 컴파일을 위해 C 컴파일러 등 개발 도구를 필요로 한다. 다음 명령어를 실행하여 Xcode Command-Line Tools를 설치한다.21

   ```
   xcode-select --install
   ```

3. **Ruby 설치**: Homebrew를 사용하여 최신 버전의 Ruby를 설치한다.20

   ```
   brew install ruby
   ```

4. **환경 변수 설정**: Homebrew로 설치한 Ruby가 시스템 Ruby보다 우선적으로 사용되도록 셸 설정 파일(`~/.zshrc`, `~/.bash_profile` 등)에 다음 경로를 추가한다.

   ```
   export PATH="/usr/local/opt/ruby/bin:$PATH"
   ```

   설정 파일을 저장한 후, `source ~/.zshrc` 명령을 실행하거나 터미널을 재시작하여 변경 사항을 적용한다. `which ruby` 명령을 실행했을 때 `/usr/local/bin/ruby` 경로가 표시되면 정상적으로 설정된 것이다.20

5. **Jekyll 및 Bundler 설치**: 이제 새로 설치한 Ruby 환경에서 Jekyll과 Bundler를 설치한다.20

   ```
   gem install jekyll bundler
   ```

6. **설치 확인**: `jekyll -v` 명령어로 설치를 최종 확인한다.


Jekyll은 '관례에 의한 설정(Convention over Configuration)' 원칙에 따라 동작하며, 이는 정해진 디렉터리 구조와 파일 명명 규칙을 통해 대부분의 설정이 자동으로 이루어짐을 의미한다.12 이 구조를 이해하는 것은 Jekyll을 효과적으로 활용하기 위한 필수 전제 조건이다.

- `_config.yml`: 프로젝트의 루트에 위치하며, 사이트의 모든 전역 설정을 담고 있는 가장 중요한 파일이다.12 이 파일에서는 사이트의 제목(`title`), 설명(`description`), URL(`url`), 기본 경로(`baseurl`), 사용할 테마(`theme`), 플러그인 목록(`plugins`) 등을 YAML 형식으로 정의한다. 특히 `url`과 `baseurl`은 사이트의 링크와 자산 경로를 생성하는 데 사용되므로 정확하게 설정해야 한다. 이 파일을 수정한 후에는 변경 사항을 적용하기 위해 로컬 서버를 재시작해야 한다.12

- `_posts`: 블로그 게시물(포스트)이 저장되는 핵심 디렉터리다. 이 폴더 안에 있는 모든 마크다운 파일은 개별 블로그 글로 변환된다. 파일명은 반드시 `YYYY-MM-DD-title.MARKUP` 형식을 따라야 한다 (예: `2024-01-01-hello-world.md`). 파일명에 포함된 날짜는 게시물의 발행일로 사용되며, URL 생성에도 영향을 미친다. 각 포스트 파일의 최상단에는 '머리말(Front Matter)'이라 불리는 YAML 형식의 메타데이터 블록이 위치해야 한다. 이 머리말에는 글의 제목(`title`), 레이아웃(`layout`), 카테고리(`categories`), 태그(`tags`) 등 추가 정보를 정의할 수 있다.12

- `_layouts`: 웹 페이지의 전체적인 구조를 정의하는 HTML 템플릿 파일들이 위치한다. 예를 들어, `default.html`은 헤더, 푸터 등 모든 페이지에 공통적으로 적용되는 기본 골격을 담고, `post.html`은 개별 블로그 글의 내용을 표시하는 레이아웃을 정의한다. 포스트의 머리말에서 `layout: post`라고 지정하면, 해당 포스트의 콘텐츠는 `_layouts/post.html` 템플릿 안에 삽입되어 최종 HTML 페이지가 생성된다.12

- `_includes`: 여러 페이지에서 재사용되는 작은 HTML 조각 파일들을 보관하는 디렉터리다. 네비게이션 바, 사이드바, 댓글 섹션 등을 별도의 파일로 만들어두고, `{% include navigation.html %}`와 같은 Liquid 태그를 사용하여 필요한 위치에 삽입할 수 있다. 이는 코드의 중복을 줄이고 유지보수성을 높인다.12

- `assets`: CSS, JavaScript, 이미지, 폰트 등 웹사이트의 디자인과 기능을 담당하는 정적 자원들을 저장하는 폴더다.12

- `_site`: 이 디렉터리는 Jekyll이 소스 파일들을 빌드하여 생성한 최종 결과물, 즉 실제 웹 서버에 배포될 완전한 정적 웹사이트가 저장되는 곳이다. 사용자가 직접 이 폴더를 수정해서는 안 되며, 소스 코드를 변경하고 Jekyll을 다시 빌드할 때마다 이 폴더의 내용은 덮어쓰인다. 따라서 이 폴더는 버전 관리에서 제외하기 위해 `.gitignore` 파일에 반드시 추가해야 한다.12

- `Gemfile` 및 `Gemfile.lock`: Bundler가 사용하는 파일로, 프로젝트에 필요한 Ruby Gem들의 목록과 정확한 버전을 명시한다. 이를 통해 다른 개발 환경에서도 동일한 의존성을 유지할 수 있다.12


Jekyll 테마는 블로그의 시각적 디자인과 레이아웃을 결정하는 핵심 요소다. 잘 만들어진 테마를 사용하면 복잡한 프론트엔드 개발 없이도 전문적인 수준의 블로그를 빠르게 구축할 수 있다.

테마 적용 방법

Jekyll 테마를 적용하는 방법은 크게 세 가지로 나눌 수 있다.

1. **GitHub 지원 테마 사용**: GitHub Pages는 여러 공식 테마를 지원한다. 이 테마들은 `_config.yml` 파일에 단 한 줄을 추가하는 것만으로 매우 간단하게 적용할 수 있다. 예를 들어, 'Minimal' 테마를 사용하려면 다음과 같이 설정한다.23

   ```YAML
theme: jekyll-theme-minimal
   ```
   
   이 방식은 가장 간단하지만, 선택할 수 있는 테마가 제한적이다.

2. **원격 테마(Remote Theme) 사용**: GitHub에 공개된 대부분의 Jekyll 테마는 원격 테마 방식으로 사용할 수 있다. `jekyll-remote-theme` 플러그인을 사용하며, `_config.yml`에 다음과 같이 테마 저장소 정보를 지정한다.23

   ```YAML
remote_theme: owner/repository
   ```
   
   이 방법은 로컬 저장소에 테마 파일을 직접 포함하지 않아도 되므로 프로젝트를 깔끔하게 유지할 수 있는 장점이 있다.

3. **테마 파일 직접 복사/Fork**: 가장 유연한 방법으로, 마음에 드는 테마의 GitHub 저장소를 Fork하거나 파일 전체를 다운로드하여 자신의 프로젝트 저장소에 직접 복사하는 방식이다.11 이 경우 

   `_config.yml`의 `theme` 설정은 필요 없으며, 테마의 모든 파일(`_layouts`, `_includes`, `assets` 등)을 직접 수정하여 완벽하게 제어할 수 있다.

테마 커스터마이징

테마를 적용한 후에는 자신만의 스타일과 필요에 맞게 커스터마이징하는 과정이 필요하다. Jekyll은 '오버라이드(override)' 매커니즘을 제공하여 원본 테마 파일을 직접 수정하지 않고도 안전하게 변경 사항을 적용할 수 있게 한다.

- **CSS 커스터마이징**: 테마의 색상, 폰트, 간격 등 스타일을 변경하고 싶을 경우, `assets/css/style.scss` (또는 유사한 이름의 메인 CSS 파일) 파일을 생성한다. 이 파일의 최상단에 `@import "{{ site.theme }}";` 구문을 추가하여 원본 테마의 스타일을 먼저 불러온다. 그 다음 줄부터 자신만의 CSS 규칙을 추가하면, 이 규칙들이 원본 스타일을 덮어쓰거나 확장하게 된다.23
- **HTML 레이아웃 커스터마이징**: 헤더에 분석 스크립트를 추가하거나 푸터의 내용을 변경하는 등 HTML 구조를 수정해야 할 경우, 원본 테마 저장소에서 수정하고자 하는 파일(예: `_layouts/default.html` 또는 `_includes/header.html`)을 찾는다. 그리고 자신의 프로젝트에 동일한 경로와 파일명으로 새 파일을 생성한 뒤, 원본 파일의 내용을 복사하여 필요한 부분을 수정한다. Jekyll은 빌드 시 프로젝트 내의 파일을 우선적으로 사용하므로, 이렇게 생성된 파일이 원본 테마의 파일을 대체하게 된다.23 이 오버라이드 방식을 통해 원본 테마의 업데이트가 있더라도 내가 수정한 내용과 충돌 없이 안전하게 반영할 수 있다.


GitHub Pages 블로그 운영의 핵심은 '로컬 개발 및 테스트 -> Git을 통한 버전 관리 및 푸시 -> 원격 서버에서의 자동 배포'로 이어지는 체계적인 워크플로에 있다. 이 과정을 통해 콘텐츠의 품질을 보장하고 빠르고 안정적인 배포를 실현할 수 있다.

로컬 환경에서의 테스트

새로운 글을 작성하거나 블로그 디자인을 변경할 때, 변경 사항을 즉시 원격 저장소에 반영하는 것은 위험하다. 오타, 깨진 링크, 레이아웃 문제 등이 방문자에게 그대로 노출될 수 있기 때문이다. 이를 방지하기 위해 로컬 컴퓨터에서 Jekyll 서버를 구동하여 변경 사항을 실시간으로 미리 확인하는 과정은 필수적이다.25

프로젝트의 루트 디렉터리에서 터미널을 열고 다음 명령어를 실행한다.11

```
bundle exec jekyll serve
```

이 명령어는 Bundler를 통해 프로젝트의 `Gemfile.lock`에 명시된 정확한 버전의 Gem들을 사용하여 Jekyll 서버를 실행한다. 이는 로컬 환경과 GitHub Pages의 빌드 환경 간의 의존성 차이로 인한 문제를 최소화하는 데 도움이 된다.25

서버가 성공적으로 구동되면, 터미널에 `Server address: http://127.0.0.1:4000/`와 같은 메시지가 출력된다. 웹 브라우저에서 이 주소로 접속하면 현재 로컬 파일들을 기반으로 빌드된 블로그를 확인할 수 있다. Jekyll 서버는 파일 변경 사항을 자동으로 감지하여 사이트를 다시 빌드하므로(`--watch` 옵션이 기본 활성화), 소스 코드를 수정한 후 저장하면 잠시 뒤 브라우저를 새로고침하는 것만으로 변경된 내용을 즉시 확인할 수 있다. 이 빠른 피드백 루프는 효율적인 개발과 콘텐츠 작성을 가능하게 한다.11

GitHub Pages로의 배포

로컬 테스트를 통해 모든 변경 사항이 의도한 대로 작동하는 것을 확인했다면, 이제 이를 원격 GitHub 저장소에 반영하여 실제 웹사이트에 배포할 차례다.

1. **변경 사항 스테이징(Staging)**: Git을 사용하여 변경된 모든 파일(새로 작성한 포스트, 수정된 레이아웃 파일 등)을 스테이징 영역에 추가한다.

   ```
   git add.
   ```

2. **커밋(Commit)**: 변경 사항에 대한 의미 있는 메시지와 함께 로컬 저장소에 커밋한다.

   ```
   git commit -m "Add new post about advanced CI/CD"
   ```

3. **푸시(Push)**: 로컬 저장소의 커밋 내역을 GitHub 원격 저장소로 푸시한다.

   ```
   git push origin main
   ```

`git push` 명령이 완료되면, 앞서 설명한 대로 GitHub Actions 워크플로가 자동으로 트리거되어 빌드 및 배포 과정을 수행한다.4 몇 분 후, `https://username.github.io` 주소로 접속하면 방금 커밋한 변경 사항이 웹사이트에 반영된 것을 확인할 수 있다. 이처럼 체계적인 워크플로는 블로그 운영의 안정성과 효율성을 극대화하는 핵심적인 실천 방안이다.



기술 블로그, 특히 수학, 과학, 공학 분야의 콘텐츠를 다룰 때 복잡한 수식을 정확하고 미려하게 표현하는 능력은 필수적이다. Jekyll 블로그에서는 LaTeX 문법으로 작성된 수식을 웹 페이지에 렌더링하기 위해 주로 MathJax와 KaTeX라는 두 가지 JavaScript 라이브러리를 활용한다.30 두 라이브러리는 동일한 목표를 가지지만, 성능, 기능 범위, 작동 방식에서 뚜렷한 차이를 보이므로, 블로그의 특성과 요구사항에 맞춰 신중하게 선택해야 한다.

**MathJax**는 LaTeX 수식 렌더링의 표준으로 여겨질 만큼 가장 폭넓은 기능과 호환성을 자랑한다. 거의 모든 표준 LaTeX 매크로와 환경을 지원하여 복잡하고 긴 수식도 문제없이 표현할 수 있다.30 또한, 사용자가 렌더링된 수식을 우클릭하여 TeX 소스를 보거나 확대하는 등 다양한 인터랙티브 기능을 제공한다는 장점이 있다.30 하지만 이러한 풍부한 기능은 성능상의 트레이드오프를 동반한다. MathJax는 주로 클라이언트 측(사용자의 브라우저)에서 JavaScript를 실행하여 수식을 렌더링하므로, 수식이 많은 페이지에서는 로딩 속도가 상대적으로 느려질 수 있다.30

**KaTeX**는 Khan Academy에서 개발한 라이브러리로, '속도'에 최우선 가치를 둔다. KaTeX는 서버 사이드 렌더링을 지원하여 Jekyll 빌드 시점에 수식을 HTML과 CSS로 미리 변환할 수 있다. 이 방식은 클라이언트의 부담을 크게 줄여주어 매우 빠른 렌더링 속도를 제공한다.30 하지만 빠른 속도를 위해 기능 범위를 다소 제한하여, 일부 복잡한 LaTeX 환경이나 매크로는 지원하지 않을 수 있다.30

다음은 두 라이브러리의 핵심적인 특징을 비교한 표다.

| 기능 (Feature)                 | MathJax                                                  | KaTeX                                                |
| ------------------------------ | -------------------------------------------------------- | ---------------------------------------------------- |
| 렌더링 속도 (Rendering Speed)  | 상대적으로 느림, 클라이언트 측 렌더링 부하 30            | 매우 빠름, 서버 사이드 렌더링 가능 30                |
| 지원 기능 범위 (Feature Scope) | 광범위함, 거의 모든 LaTeX 매크로 지원 30                 | 핵심 기능에 집중, 일부 고급 환경 미지원 30           |
| 렌더링 방식 (Rendering Method) | 주로 Client-Side (JavaScript) 32                         | Server-Side (via Ruby gem) 또는 Client-Side 33       |
| 마크다운 파서와의 충돌         | `_` (아래첨자) 문자가 마크다운의 *italic*과 충돌 가능 36 | 충돌 가능성 존재하나, 서버사이드 렌더링 시 완화 가능 |
| 사용자 인터랙션                | 우클릭 메뉴 등 추가 기능 제공 30                         | 정적 렌더링 결과물만 제공                            |
| 테이블 내 `$`\rvert`$` 사용    | 필요 (파서 문제)                                         | 필요 (파서 문제)                                     |

**MathJax 적용 방법**

1. `_includes` 디렉터리에 `mathjax_support.html` 파일을 생성하고 MathJax 설정 스크립트를 추가한다. 이 스크립트는 인라인 수식을 위한 `$`...`$` 구분자와 블록 수식을 위한 `$$`...`$$` 구분자를 정의한다.35

2. `_layouts/default.html`과 같은 기본 레이아웃 파일의 `<head>` 태그 내에, 조건부 로직을 사용하여 수식이 필요한 페이지에서만 위 스크립트를 불러오도록 설정한다. 이는 불필요한 스크립트 로딩을 방지하여 성능을 최적화한다.37

   ```HTML
   {% if page.use_math %}
     {% include mathjax_support.html %}
   {% endif %}
   ```
   
3. 수식을 사용하려는 포스트의 머리말에 `use_math: true`를 추가한다. 본문에서는 인라인 수식은 `$E=mc^2$`와 같이, 블록 수식은 `$$...$$`로 감싸서 작성한다.37

**KaTeX 적용 방법 (서버 사이드)**

1. `Gemfile`의 `:jekyll_plugins` 그룹에 `gem 'kramdown-math-katex'`를 추가한다.33

2. 터미널에서 `bundle install`을 실행하여 Gem을 설치한다.

3. `_config.yml` 파일에서 마크다운 엔진 설정을 다음과 같이 변경하여 KaTeX를 활성화한다.33

   YAML

   ```
   kramdown:
     math_engine: katex
   ```

4. KaTeX의 CSS와 폰트 파일을 프로젝트의 `assets` 폴더에 직접 추가하거나, 레이아웃 파일에서 CDN 링크를 통해 로드한다. 이 파일들은 수식이 올바르게 렌더링되기 위해 필수적이다.33


마크다운으로 테이블을 작성할 때, 수직선(`|`) 문자는 셀(cell)을 구분하는 특별한 의미를 가진다. 이로 인해 수식 내에서 절댓값(`|x|`)이나 조건부 확률(`P(A|B)`)을 표현하기 위해 `|`를 사용하면, 마크다운 파서가 이를 셀의 끝으로 오인하여 테이블 구조를 파괴하는 심각한 문제가 발생한다.39

이 문제의 근본적인 원인은 마크다운 파서의 작동 순서에 있다. GFM(GitHub Flavored Markdown)을 포함한 대부분의 마크다운 명세는 문서를 파싱할 때 두 단계를 거친다. 첫 번째 단계에서는 테이블, 인용구, 코드 블록과 같은 블록(block) 수준의 구조를 먼저 분석한다. 두 번째 단계에서야 각 블록 내부의 텍스트를 분석하여 강조, 링크, 인라인 코드, 그리고 인라인 수식과 같은 인라인(inline) 요소를 처리한다.39

이러한 파싱 순서 때문에, 테이블 파서가 셀 구분자를 찾기 위해 `|` 문자를 스캔할 때, 그 문자가 `$…$`와 같은 수식 구분자 내부에 있는지 여부를 전혀 인지하지 못한다. 파서의 관점에서는 단순히 셀을 끝내는 문자로 보일 뿐이며, 그 결과 테이블의 열이 의도치 않게 분리되거나 구조 전체가 깨지게 된다. 일반적인 해결책으로 생각할 수 있는 백슬래시를 이용한 이스케이프(`\|`)는 마크다운 렌더러에 따라 동작이 일관되지 않아 불안정하며, 때로는 백슬래시 문자가 그대로 출력되는 부작용을 낳기도 한다.41

이 문제를 기술적으로 완벽하게 해결하는 방법은 마크다운 파서에게는 일반 텍스트로 인식되지만, LaTeX 렌더링 엔진에게는 수직선으로 인식되는 명령어를 사용하는 것이다. 그 해결책이 바로 `\lvert`와 `\rvert`이다.43 이 명령어들은 각각 왼쪽과 오른쪽 수직선을 의미하며, 시각적으로는 `|`와 동일한 결과를 렌더링한다.

예를 들어, 테이블 내에서 절댓값 `|V|`를 표현해야 할 경우, 다음과 같이 작성해야 한다.

| 변수                  | 설명                                           |
| --------------------- | ---------------------------------------------- |
| `$`\lvert V \rvert`$` | 그래프의 정점(Vertex) 집합의 크기(cardinality) |

위 예시에서 테이블 파서는 `$`\lvert V \rvert`$`라는 문자열을 파싱할 때 `|` 문자를 발견하지 못하므로, 셀을 정상적으로 인식한다. 이후 인라인 파싱 단계에서 MathJax나 KaTeX 엔진이 이 문자열을 처리하게 되면, `\lvert`와 `\rvert`를 올바른 수직선 기호로 렌더링하여 최종적으로 사용자는 `|V|`라는 정확한 수식을 보게 된다. 이는 단순한 문법 트릭이 아니라, 마크다운 파서의 작동 원리를 이해하고 이를 우회하는 근본적인 해결책이다. 따라서 기술 문서, 특히 LaTeX 수식이 포함된 테이블을 작성할 때는 `|` 대신 `\lvert`와 `\rvert`를 사용하는 것을 표준으로 삼아야 한다.


GitHub Pages는 기본적으로 `git push` 이벤트에 반응하여 사이트를 자동으로 빌드하고 배포하는 기능을 내장하고 있으며, 이 기능의 배후에는 GitHub Actions가 있다.3 사용자는 단순히 이 기본 기능을 사용하는 것을 넘어, GitHub Actions 워크플로우를 직접 정의하고 커스터마이징함으로써 훨씬 더 정교하고 강력한 배포 파이프라인을 구축할 수 있다.

저장소의 `Settings > Pages > Build and deployment` 메뉴에서 배포 소스를 'GitHub Actions'로 명시적으로 선택하면, GitHub가 제공하는 기본 Jekyll 배포 워크플로우 템플릿을 사용하거나 자신만의 워크플로우 파일을 작성할 수 있다.29 워크플로우 파일은 저장소 루트의 `.github/workflows/` 디렉터리에 YAML 형식으로 작성한다.

이러한 커스텀 워크플로우를 통해 다음과 같은 고급 기능들을 구현할 수 있다.

- **빌드 전 테스트**: 사이트를 빌드하기 전에 링크 체커(link checker)를 실행하여 깨진 링크가 있는지 검사하거나, HTML 유효성 검사 도구를 실행하여 마크업 오류를 사전에 방지할 수 있다.
- **의존성 제어**: 특정 Ruby 버전이나 Jekyll 버전을 명시하여 빌드 환경을 고정함으로써, GitHub Pages의 기본 빌드 환경 업데이트로 인해 발생할 수 있는 예기치 않은 빌드 실패를 방지할 수 있다.
- **외부 데이터 소스 연동**: GitHub Actions의 가장 강력한 활용 사례 중 하나는 외부 서비스와의 연동이다. 예를 들어, Notion을 콘텐츠 관리 시스템(CMS)으로 사용하는 경우를 생각해보자. Notion API를 사용하여 특정 데이터베이스의 페이지들을 가져와 마크다운 파일로 변환하는 스크립트를 작성할 수 있다. 그리고 GitHub Actions 워크플로우가 이 스크립트를 실행하여 변환된 마크다운 파일들을 `_posts` 디렉터리에 자동으로 추가한 뒤, Jekyll 빌드 및 배포 단계를 진행하도록 구성할 수 있다.44 이 방식을 사용하면 비개발자도 Notion의 편리한 편집 환경에서 글을 작성하고, '갱신' 버튼 하나만 누르면 블로그에 자동으로 글이 발행되도록 만들 수 있다. 이는 콘텐츠 작성 환경과 기술적인 배포 과정을 완벽하게 분리하는 매우 효율적인 방법이다.

이처럼 GitHub Actions를 활용하면 단순한 정적 사이트 호스팅을 넘어, 전문적인 퍼블리싱 플랫폼에 필적하는 고도로 자동화되고 안정적인 CI/CD 파이프라인을 구축할 수 있다. 이는 개인 기술 블로그의 운영 수준을 한 단계 격상시키는 핵심적인 기술이다.



Jekyll은 GitHub Pages와의 긴밀한 통합 덕분에 오랫동안 표준적인 선택지로 여겨져 왔지만, 블로그의 규모가 커지고 더 나은 개발 경험을 추구하는 사용자들이 늘어나면서 Hugo와 같은 현대적인 대안들이 주목받고 있다. Jekyll과 Hugo의 선택은 단순히 개인의 취향 문제가 아니라, 개발 철학과 우선순위의 차이를 반영하는 중요한 기술적 결정이다.

빌드 속도

가장 극적인 차이점은 빌드 속도다. Hugo는 Go 언어로 개발되어 사전 컴파일된 단일 바이너리 파일로 실행된다. 이 덕분에 빌드 속도가 압도적으로 빠르다. 수백 개의 포스트를 가진 블로그를 빌드할 때, Ruby 인터프리터를 통해 실행되는 Jekyll은 수십 초에서 길게는 수 분까지 소요될 수 있는 반면, Hugo는 동일한 작업을 수백 밀리초, 즉 눈 깜짝할 사이에 완료한다.14 이 속도 차이는 로컬 개발 환경에서의 생산성에 지대한 영향을 미친다. Hugo를 사용하면 파일 저장과 동시에 브라우저에 변경 사항이 거의 즉시 반영되는, 매우 짧고 쾌적한 피드백 루프를 경험할 수 있다.

의존성 및 설치

Jekyll은 Ruby 런타임과 Bundler, 그리고 여러 Gem들에 대한 의존성을 가진다. 이는 앞서 설명했듯이, 특히 Windows 환경에서 복잡한 설치 과정과 잠재적인 버전 충돌 문제를 야기한다.14 반면 Hugo는 외부 의존성이 없는 단일 실행 파일이다. 운영체제에 맞는 바이너리 파일 하나만 다운로드하여 PATH에 추가하면 설치가 끝난다. 이러한 단순함은 개발 환경을 설정하고 유지하는 데 드는 시간과 노력을 크게 줄여준다.14

테마 관리 및 유연성

Hugo는 Git Submodule을 표준적인 테마 관리 방식으로 채택하고 있다. 이를 통해 원본 테마 저장소를 내 프로젝트의 하위 모듈로 포함시키므로, 원본 테마의 소스 코드와 내가 수정한 커스터마이징 코드가 명확하게 분리된다. 이는 원본 테마의 업데이트를 손쉽게 반영하면서도 개인화된 내용을 안전하게 유지할 수 있게 해준다.14 반면 Jekyll은 테마 전체를 프로젝트에 복사하는 방식을 사용하는 경우가 많아, 원본 테마의 업데이트를 추적하고 병합하기가 상대적으로 번거롭다.14

GitHub Pages 연동

이 지점에서 두 도구의 철학적 차이가 명확히 드러난다. Jekyll은 GitHub Pages에 내장된 '마법' 같은 통합을 제공한다. 사용자는 마크다운 파일을 푸시하기만 하면 웹사이트가 나타나는 편리함을 누릴 수 있다.10 하지만 이 편리함은 느린 로컬 개발 경험과 복잡한 의존성 관리라는 비용을 수반한다.

반면 Hugo는 이러한 '마법'을 포기하는 대신, 압도적인 로컬 개발 경험을 우선시한다. Hugo를 GitHub Pages에 배포하기 위해서는 사용자가 직접 GitHub Actions 워크플로우를 설정하여, `hugo` 명령어로 사이트를 빌드하고 그 결과물(`public` 디렉터리)을 배포 브랜치(`gh-pages`)에 푸시하는 과정을 명시적으로 구성해야 한다.45 이는 초기 설정에 조금 더 노력이 필요하지만, 빌드와 배포의 전 과정을 사용자가 완벽하게 통제할 수 있다는 장점을 제공한다. 진지한 개발자들에게는 숨겨진 마법보다는 명시적인 제어가 장기적으로 더 선호되는 방식일 수 있다.

결론적으로, Jekyll은 GitHub Pages와의 간편한 통합을 원하는 입문자에게 적합할 수 있지만, Hugo는 빠른 빌드 속도와 쾌적한 로컬 개발 경험, 그리고 명확한 의존성 관리를 중시하는 숙련된 개발자에게 훨씬 더 매력적인 선택지다.


블로그의 규모가 성장하여 Jekyll의 빌드 속도가 생산성을 저해하는 단계에 이르면, Hugo로의 마이그레이션은 합리적인 선택이 된다. 다행히 Hugo는 Jekyll 사용자를 위해 공식적인 마이그레이션 도구와 명확한 가이드라인을 제공한다.

1단계: 자동 마이그레이션 도구 실행

Hugo는 hugo import jekyll이라는 커맨드라인 도구를 내장하고 있다.14 이 도구는 기존 Jekyll 프로젝트의 경로와 새로 생성할 Hugo 프로젝트의 경로를 인자로 받아 마이그레이션을 수행한다.

```
hugo import jekyll /path/to/jekyll/blog /path/to/new/hugo/site
```

이 명령을 실행하면, 도구는 다음과 같은 주요 작업들을 자동으로 처리한다.

- `_posts` 디렉터리의 모든 마크다운 파일을 Hugo의 콘텐츠 디렉터리(보통 `content/posts`)로 복사한다.
- Jekyll의 머리말(Front Matter) 형식을 Hugo가 인식하는 형식으로 변환한다. 예를 들어, `categories`와 `tags`를 Hugo의 분류법(Taxonomy)에 맞게 조정한다.
- `assets`나 `images`와 같은 정적 파일들을 Hugo의 `static` 디렉터리로 이동시킨다.14

이 자동화 도구 덕분에 기본적인 콘텐츠 이전 작업은 매우 빠르고 간단하게 완료될 수 있다.

2단계: 수동 후속 작업

자동 마이그레이션이 만능은 아니며, 성공적인 이전을 위해서는 몇 가지 중요한 수동 작업이 반드시 필요하다.

- **URL 일관성 유지**: 마이그레이션 과정에서 가장 중요한 것은 기존 포스트의 URL 구조를 그대로 유지하는 것이다. URL이 변경되면 검색 엔진의 색인 결과가 무효화되고, 외부 사이트에서 연결된 링크들이 모두 깨져 블로그의 SEO 자산에 치명적인 손실을 입을 수 있다. Hugo의 설정 파일(`config.toml` 또는 `config.yaml`)에서 `permalinks` 설정을 조정하여 Jekyll에서 사용하던 URL 구조(예: `/:categories/:year/:month/:day/:title/`)를 정확하게 재현해야 한다.14
- **테마 및 레이아웃 재구성**: Jekyll 테마는 Hugo와 호환되지 않으므로, 새로운 Hugo 테마를 선택하고 적용해야 한다. 기존 블로그의 디자인과 최대한 유사한 테마를 찾거나, 선택한 테마를 직접 커스터마이징하여 디자인의 연속성을 유지하는 노력이 필요하다.
- **플러그인 기능 대체**: Jekyll에서 플러그인을 통해 구현했던 기능들은 Hugo에서 다른 방식으로 대체해야 한다. 예를 들어, 관련 글 목록, 사이트맵 생성 등은 대부분 Hugo의 내장 기능으로 제공된다. Jekyll의 `include`와 유사한 기능은 Hugo의 'Partials'로, 복잡한 마크다운 확장은 'Shortcodes'를 사용하여 구현할 수 있다.45 기존에 사용하던 플러그인 목록을 검토하고, 각각에 해당하는 Hugo의 기능을 찾아 적용하는 과정이 필요하다.

이러한 전략적인 접근을 통해, 기존 콘텐츠와 SEO 자산을 안전하게 보존하면서 Hugo가 제공하는 월등한 성능과 개발 경험의 이점을 누릴 수 있다.



기술 블로그를 성공적으로 운영하는 것은 단순히 기술적 설정에 그치지 않고, 콘텐츠를 효율적으로 작성하고 체계적으로 관리하는 전략을 필요로 한다.

콘텐츠와 표현의 분리

마크다운을 핵심 작성 도구로 사용하는 것은 매우 현명한 전략이다. 마크다운은 콘텐츠의 구조(제목, 목록, 코드 블록 등)에만 집중하게 하고, 시각적인 표현(색상, 폰트 등)은 테마와 CSS에 위임한다. 이러한 '관심사의 분리'는 작성자가 글의 논리적 흐름에만 집중할 수 있게 하여 생산성을 높이고, 나중에 블로그 디자인 전체를 변경하더라도 콘텐츠를 일일이 수정할 필요가 없게 만들어 유지보수성을 극대화한다.49

Git 브랜치 전략의 활용

블로그의 모든 콘텐츠는 Git으로 관리되므로, 소프트웨어 개발에서 사용하는 정교한 브랜치 전략을 콘텐츠 관리에 도입할 수 있다. 예를 들어, 다음과 같은 워크플로를 구축할 수 있다.10

1. **초안 작성**: 새로운 글을 작성할 때는 `master`(또는 `main`) 브랜치에서 `draft/my-new-post`와 같은 이름의 새로운 브랜치를 생성한다. 이 브랜치에서 자유롭게 글을 쓰고 커밋을 쌓아나간다.
2. **리뷰 및 교정**: 글의 초안이 완성되면, 동료에게 리뷰를 요청하기 위해 Pull Request를 생성할 수 있다. 또는 `ready`와 같은 중간 브랜치에 병합하여 발행 대기 상태로 관리할 수도 있다.
3. **발행**: 모든 검토가 끝나고 발행할 준비가 되면, 해당 브랜치를 `master` 브랜치로 병합한다. 이 병합 작업이 `git push`를 통해 원격 저장소에 반영되면, GitHub Actions가 자동으로 사이트를 빌드하고 배포하여 글이 발행된다.

이러한 체계적인 접근은 여러 글을 동시에 작업하거나 협업을 할 때 매우 유용하며, 발행 실수를 줄이고 콘텐츠의 품질을 높이는 데 기여한다.

자산 관리 및 아이디어 추적

블로그에 사용되는 모든 이미지, 다이어그램, 코드 예제 등의 정적 자산은 콘텐츠와 함께 Git 저장소 내에서 관리하는 것이 바람직하다. 이는 콘텐츠와 자산 간의 일관성을 보장하고, 외부 이미지 호스팅 서비스의 장애나 정책 변경으로부터 자유롭게 해준다. 또한, GitHub 저장소의 이슈(Issues) 기능을 블로그 아이디어나 글감 목록을 관리하는 도구로 적극적으로 활용할 수 있다. 각 이슈는 하나의 글 주제가 될 수 있으며, 관련 자료나 메모를 댓글로 추가하며 아이디어를 구체화할 수 있다.


장기적인 관점에서 블로그를 안정적으로 운영하고 미래의 기술 변화에 유연하게 대응하기 위한 전략적 고려가 필요하다.

의존성 관리와 환경 일치

로컬 개발 환경과 GitHub Pages의 실제 빌드 환경 간의 미세한 차이, 특히 Ruby Gem들의 버전 차이는 예기치 않은 빌드 오류의 주된 원인이 된다.25 이러한 문제를 방지하기 위해, 

`Gemfile`에 `github-pages` Gem을 명시하고 `bundle update` 명령을 통해 주기적으로 최신 버전으로 유지하는 것이 중요하다. 더 나아가, `Gemfile.lock` 파일을 Git 저장소에 반드시 포함하여 로컬 환경과 원격 빌드 환경이 정확히 동일한 버전의 의존성을 사용하도록 강제해야 한다. 이는 환경 차이로 인한 문제를 최소화하는 가장 확실한 방법이다.12

SSG 비종속적 콘텐츠 전략

처음에는 Jekyll의 편리함 때문에 시작했더라도, 블로그의 규모가 커져 포스트 수가 수백, 수천 개에 이르면 Jekyll의 빌드 시간은 심각한 생산성 저하 요인이 될 수 있다. 따라서 장기적인 관점에서는 언제든지 Hugo와 같은 고성능 SSG로 마이그레이션할 수 있는 가능성을 열어두는 것이 현명하다. 이를 위해 콘텐츠를 최대한 SSG에 비종속적인 형태로 유지해야 한다. 즉, 콘텐츠는 순수한 표준 마크다운 문법으로만 작성하고, 특정 SSG의 플러그인이나 독자적인 기능에 대한 의존도를 최소화하는 것이다. 자산 역시 assets/images와 같은 표준적인 폴더 구조로 관리하여, 나중에 SSG를 변경하더라도 콘텐츠와 자산을 거의 그대로 이전할 수 있도록 설계해야 한다.

영구적인 정체성 확보: 커스텀 도메인

username.github.io라는 URL도 훌륭하지만, 진정으로 지속 가능한 블로그를 원한다면 초기 단계부터 커스텀 도메인(예: my-tech-blog.com)을 설정하는 것을 강력히 권장한다.10 커스텀 도메인을 사용하면, 향후 호스팅 플랫폼을 GitHub Pages에서 Netlify, Vercel 또는 자체 서버로 이전하더라도 블로그의 URL을 영구적으로 유지할 수 있다. 이는 검색 엔진 순위, 외부 링크, 그리고 독자들에게 각인된 블로그의 정체성을 온전히 보존하는 가장 중요한 장기 투자다. 하나의 페이지에는 하나의 커스텀 도메인만 할당할 수 있으므로, 신중하게 선택하고 처음부터 적용하는 것이 바람직하다.10


1. GitHub Pages 빠른 시작 - GitHub Docs, 8월 24, 2025에 액세스, https://docs.github.com/ko/pages/quickstart
2. [Git] Github Pages로 리액트 호스팅 - velog, 8월 24, 2025에 액세스, [https://velog.io/@tngusro/Git-Github-Pages%EB%A1%9C-%EB%A6%AC%EC%95%A1%ED%8A%B8-%ED%98%B8%EC%8A%A4%ED%8C%85](https://velog.io/@tngusro/Git-Github-Pages로-리액트-호스팅)
3. Github Pages - velog, 8월 24, 2025에 액세스, https://velog.io/@jaemaning/Github-Pages
4. GitHub Pages로 웹사이트 무료 호스팅하기, 8월 24, 2025에 액세스, https://www.daleseo.com/github-pages/
5. GitHub Pages 활성화 시켜 여러 repository 웹 호스팅하기., 8월 24, 2025에 액세스, https://cheershennah.tistory.com/216
6. 가입 및 초기 세팅 | Github.io로 나만의 블로그 만들기, 8월 24, 2025에 액세스, [https://sh-otherlife.tistory.com/entry/Githubio%EB%A1%9C-%EB%82%98%EB%A7%8C%EC%9D%98-%EB%B8%94%EB%A1%9C%EA%B7%B8-%EB%A7%8C%EB%93%A4%EA%B8%B0-1-%EA%B0%80%EC%9E%85-%EB%B0%8F-%EC%B4%88%EA%B8%B0-%EC%84%B8%ED%8C%85](https://sh-otherlife.tistory.com/entry/Githubio로-나만의-블로그-만들기-1-가입-및-초기-세팅)
7. GitHub 페이지를 사용하여 웹 사이트 만들기 및 호스트 - Training - Microsoft Learn, 8월 24, 2025에 액세스, https://learn.microsoft.com/ko-kr/training/modules/create-host-web-sites-github-pages/
8. [GitHub] GitHub Pages 기능 활용해 웹페이지 호스팅하기, 8월 24, 2025에 액세스, https://kotlinworld.com/291
9. [Git] 깃허브 페이지(GitHub Pages) 만들기 / 리액트 깃 페이지 - 이롭게 현명하게 - 티스토리, 8월 24, 2025에 액세스, https://devyihyun.tistory.com/203
10. GitHub의 페이지 기능 이용하기 - dogfeet, 8월 24, 2025에 액세스, https://dogfeet.github.io/articles/2012/github-pages.html
11. Windows 환경에서 Github pages와 jekyll 를 활용해 나만의 블로그 웹호스팅하기, 8월 24, 2025에 액세스, https://devcamus.tistory.com/20
12. [Jekyll Blog] GitHub 연동 및 Jekyll 설치 · TheoryDB, 8월 24, 2025에 액세스, https://theorydb.github.io/envops/2019/05/03/envops-blog-github-pages-jekyll/
13. Installing Jekyll on Windows - GitHub Pages, 8월 24, 2025에 액세스, https://treehouse.github.io/installation-guides/windows/jekyll-windows.html
14. 블로그 Jekyll to Hugo 마이그레이션 후기 - Morgenrøde, 8월 24, 2025에 액세스, https://ryanking13.github.io/2020/12/29/migrating-jekyll-to-hugo.html/
15. About GitHub Pages and Jekyll, 8월 24, 2025에 액세스, https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/about-github-pages-and-jekyll
16. Jekyll on Windows | Jekyll • Simple, blog-aware, static sites, 8월 24, 2025에 액세스, https://jekyllrb.com/docs/installation/windows/
17. Install Jekyll on Windows | CHMI services, 8월 24, 2025에 액세스, https://chmi-sops.github.io/mydoc_install_jekyll_on_windows.html
18. Installing Jekyll on Windows - GitHub Gist, 8월 24, 2025에 액세스, https://gist.github.com/arthurattwell/281a5e1888ffd89b08b4861a2e3c1b35
19. How To Install Jekyll on Mac (M1 or Intel), the Easy Way - Moncef Belyamani, 8월 24, 2025에 액세스, https://www.moncefbelyamani.com/how-to-install-jekyll-on-a-mac-the-easy-way/
20. Install Jekyll on Mac | Jekyll theme for documentation - Idratherbewriting.com, 8월 24, 2025에 액세스, https://idratherbewriting.com/documentation-theme-jekyll/mydoc_install_jekyll_on_mac.html
21. Jekyll Installation Guide (Book Edition) - Hyde Press, 8월 24, 2025에 액세스, https://hydepress.github.io/jekyll-install
22. Jekyll 기반의 Github Page로 블로그 만들기 - 끄적끄적 - 티스토리, 8월 24, 2025에 액세스, https://chaniii.tistory.com/2
23. Adding a theme to your GitHub Pages site using Jekyll, 8월 24, 2025에 액세스, https://docs.github.com/articles/adding-a-jekyll-theme-to-your-github-pages-site
24. Help installing jekyll theme on github pages, 8월 24, 2025에 액세스, https://talk.jekyllrb.com/t/help-installing-jekyll-theme-on-github-pages/5164
25. Jekyll을 사용하여 로컬로 GitHub Pages 사이트 테스트, 8월 24, 2025에 액세스, https://docs.github.com/ko/enterprise-cloud@latest/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll
26. Jekyll을 사용하여 로컬로 GitHub Pages 사이트 테스트, 8월 24, 2025에 액세스, https://docs.github.com/ko/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll
27. Jekyll: GitHub Pages 에 블로그 만들기, 8월 24, 2025에 액세스, https://xho95.github.io/blog/github-pages/jekyll/minima/theme/2017/03/05/Jekyll-Blog-with-Minima.html
28. Jekyll로 만든 GitHub Page 로컬에서 테스트 하기 - bluecolorsky - 티스토리, 8월 24, 2025에 액세스, https://bluecolorsky.tistory.com/112
29. Jekyll Chirpy(v6.0.1) 테마를 활용한 Github 블로그 만들기(2023.6 기준), 8월 24, 2025에 액세스, [https://jjikin.com/posts/Jekyll-Chirpy-%ED%85%8C%EB%A7%88%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-Github-%EB%B8%94%EB%A1%9C%EA%B7%B8-%EB%A7%8C%EB%93%A4%EA%B8%B0(2023-6%EC%9B%94-%EA%B8%B0%EC%A4%80)/](https://jjikin.com/posts/Jekyll-Chirpy-테마를-활용한-Github-블로그-만들기(2023-6월-기준)/)
30. github blog와 latex - eeeuns, 8월 24, 2025에 액세스, https://eeeuns.github.io/2020/12/10/githubblog/
31. [Jekyll/MathJax] MathJax로 수학식 표기 - jtrimind blog, 8월 24, 2025에 액세스, https://jtrimind.github.io/blog/mathjax/
32. Adding LaTeX Rendering to Our Website, Part 1 | by Manning Publications | CodeX, 8월 24, 2025에 액세스, https://manningbooks.medium.com/adding-latex-rendering-to-our-website-part-1-cedd84a24baa
33. Add LaTeX support to Jekyll site — - Younghoon, Jung, 8월 24, 2025에 액세스, https://younghoonjung.com/2020/09/add-latex-support-to-jekyll.html
34. KaTeX – The fastest math typesetting library for the web, 8월 24, 2025에 액세스, https://katex.org/
35. Render TeX/LaTeX Mathematics with MathJax - BookStack, 8월 24, 2025에 액세스, https://www.bookstackapp.com/hacks/mathjax-tex/
36. jekyll 마크다운과 maxjax 충돌 해결 방법, 8월 24, 2025에 액세스, https://emeraldgoose.github.io/jekyll/md-latex/
37. Jekyll Github 블로그에 MathJax 적용하기 - Seongsu, 8월 24, 2025에 액세스, https://baeseongsu.github.io/posts/apply-mathjax-to-jekyll-blog/
38. How to LaTeX in Jekyll using KaTeX - Xuning Yang, 8월 24, 2025에 액세스, https://www.xuningyang.com/blog/2021-01-11-katex-with-jekyll/
39. Mathematical absolute value symbol (`|`) is interpreted as end of cell (cell separator) in a table · Issue #318 · yzhang-gh/vscode-markdown - GitHub, 8월 24, 2025에 액세스, https://github.com/yzhang-gh/vscode-markdown/issues/318
40. Pipe character within $..$ in markdown table breaks equation across columns #185 - GitHub, 8월 24, 2025에 액세스, https://github.com/atom-community/markdown-preview-plus/issues/185
41. How to show the pipe "|" symbol in Markdown table? - Stack Overflow, 8월 24, 2025에 액세스, https://stackoverflow.com/questions/23723396/how-to-show-the-pipe-symbol-in-markdown-table
42. Pipe character is not properly escaped in table - Bug graveyard - Obsidian Forum, 8월 24, 2025에 액세스, https://forum.obsidian.md/t/pipe-character-is-not-properly-escaped-in-table/47494
43. Absolute Value Symbols - math mode - LaTeX Stack Exchange, 8월 24, 2025에 액세스, https://tex.stackexchange.com/questions/43008/absolute-value-symbols
44. Jekyll 기반 Github Pages와 Notion Page 연동 - LOURCODE, 8월 24, 2025에 액세스, [https://lourcode.kr/posts/Jekyll-%EA%B8%B0%EB%B0%98-Github-Pages%EC%99%80-Notion-Page-%EC%97%B0%EB%8F%99/](https://lourcode.kr/posts/Jekyll-기반-Github-Pages와-Notion-Page-연동/)
45. jekyll 에서 hugo 로 전환한 소감 - Soo Story, 8월 24, 2025에 액세스, https://findstar.pe.kr/2018/06/25/jekyll-to-hugo-thoughts/
46. Jekyll vs Hugo? 둘 중에 어떤 게 최고의 정적 사이트 생성기일까? - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/learnprogramming/comments/3dy5w1/jekyll_vs_hugo_best_static_site_generator_out_of/?tl=ko
47. Hugo로 github.io 블로그 만들기, 8월 24, 2025에 액세스, https://github.com/Integerous/Integerous.github.io
48. Jekyll 블로그에서 Hugo 블로그로 이전 후기 - DS Lab, 8월 24, 2025에 액세스, https://blog.dslab.kr/jekyll-to-hugo/
49. GitHub Pages를 이용한 기술 블로그 제작 후기, 8월 24, 2025에 액세스, [https://techblog.yogiyo.co.kr/github-pages%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EA%B8%B0%EC%88%A0-%EB%B8%94%EB%A1%9C%EA%B7%B8-%EC%A0%9C%EC%9E%91-%ED%9B%84%EA%B8%B0-77ce4b5e5564](https://techblog.yogiyo.co.kr/github-pages를-이용한-기술-블로그-제작-후기-77ce4b5e5564)

