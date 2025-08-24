# Flutter 마스터클래스


이 장에서는 Flutter 프레임워크의 근간을 이루는 핵심 개념을 정립한다. Flutter가 무엇인지 정의하고, 그 핵심 가치를 설명하며, 기술적 장점과 단점에 대한 균형 잡힌 시각을 제공하여 학습의 토대를 마련한다.


Flutter는 구글(Google)이 개발한 오픈소스 사용자 인터페이스(UI) 소프트웨어 개발 키트(SDK)이다.1 그 핵심 가치는 단일 코드베이스(Single Codebase)를 사용하여 모바일(iOS, Android), 웹, 데스크톱(Windows, macOS, Linux), 그리고 임베디드 장치까지 아우르는 다중 플랫폼(Multi-platform)을 위한 아름답고 네이티브하게 컴파일된 애플리케이션을 구축하는 데 있다.3

여기서 '네이티브하게 컴파일된다'는 표현은 매우 중요하다. 이는 Flutter가 웹뷰(WebView)나 자바스크립트 브리지(JavaScript Bridge)를 통해 플랫폼과 통신하는 일부 크로스플랫폼 프레임워크와 근본적으로 다르다는 것을 의미한다.5 Flutter 코드는 각 플랫폼의 중앙 처리 장치(CPU)가 직접 이해할 수 있는 ARM 또는 Intel 기계어 코드로 컴파일되며, 웹의 경우 JavaScript로 컴파일된다.3 이러한 접근 방식은 애플리케이션의 성능을 네이티브 앱 수준으로 끌어올리는 결정적인 요소로 작용한다. 결과적으로 Flutter는 개발자에게는 생산성의 극대화를, 사용자에게는 뛰어난 성능과 일관된 경험을 제공하는 강력한 솔루션으로 자리매김하고 있다.


Flutter의 개발 경험과 성능은 세 가지 상호 연결된 핵심 특징에 의해 정의된다. 이 특징들은 서로 시너지를 일으켜 Flutter의 가치 제안인 "빠르고, 생산적이며, 유연한" 개발을 가능하게 한다.3


핫 리로드는 Flutter 개발 경험의 혁신적인 측면 중 하나다. 개발자가 코드에 변경 사항을 적용하면, 애플리케이션을 처음부터 다시 시작하거나 현재 상태를 잃지 않고도 거의 즉시 UI에 변경 사항이 반영되는 기능이다.4 예를 들어, 버튼의 색상을 변경하거나 텍스트를 수정하면, 수 초 내에 실행 중인 에뮬레이터나 실제 기기에서 결과를 확인할 수 있다. 이는 개발 및 반복(iteration) 주기를 극적으로 단축시키며, 디자이너와 개발자가 실시간으로 협업하며 UI를 실험하고 다듬는 것을 가능하게 한다.10


Flutter의 중심 철학은 "모든 것은 위젯이다(Everything is a widget)"라는 개념에 기반한다.11 UI를 구성하는 모든 요소가 위젯이다. 화면의 구조를 잡는 `Row`, `Column`과 같은 구조적 요소부터, 사용자에게 보이는 `Text`, `Button`과 같은 시각적 요소, 심지어 `Padding`이나 `GestureDetector`처럼 보이지 않는 레이아웃 및 기능 요소까지 모두 위젯으로 정의된다.14

Flutter는 구글의 머티리얼 디자인(Material Design)과 애플의 쿠퍼티노(Cupertino) 디자인 가이드라인을 따르는 풍부하고 사용자 정의 가능한 위젯 라이브러리를 기본적으로 제공한다.1 개발자는 이러한 기성 위젯들을 조합(composition)하여 복잡한 UI를 손쉽게 구축할 수 있으며, 필요에 따라 자신만의 위젯을 생성하는 것도 매우 간단하다. 이 위젯 기반 아키텍처는 UI 코드를 재사용 가능하고 구조화된 방식으로 관리할 수 있게 해준다.


Flutter의 뛰어난 성능은 자체 렌더링 엔진을 사용한다는 점에서 비롯된다. Flutter는 플랫폼의 네이티브 UI 컴포넌트에 의존하는 대신, C++로 작성된 고성능 2D 그래픽 엔진인 스키아(Skia)를 사용하여 UI를 화면에 직접 그린다.6 이 방식은 안드로이드와 iOS 간의 UI 불일치 문제를 원천적으로 해결하며, 플랫폼 업데이트에 UI가 영향을 받을 위험을 최소화한다.

또한, 앞서 언급했듯이 자바스크립트 브리지를 사용하지 않기 때문에 앱과 플랫폼 간의 통신 병목 현상이 없다.5 그 결과, 앱 시작 시간이 단축되고, 60fps(초당 프레임)의 부드러운 애니메이션과 빠른 반응성을 제공하여 사용자는 네이티브 앱과 구별하기 어려운 수준의 경험을 하게 된다.8

이 세 가지 특징의 상호작용은 Flutter 개발의 선순환 구조를 만든다. 개발자는 위젯 기반 아키텍처를 통해 UI를 선언적으로 구축하고, 핫 리로드를 통해 즉각적인 피드백을 받으며 빠르게 실험한다. 그리고 스키아 렌더링 엔진은 이렇게 만들어진 아름답고 복잡한 UI가 모든 플랫폼에서 일관되고 높은 성능으로 작동하도록 보장한다.


Flutter는 강력한 프레임워크이지만 모든 프로젝트에 완벽한 만능 해결책은 아니다. 그 장점과 단점을 명확히 이해하는 것은 기술 스택을 결정하는 데 있어 매우 중요하다.


- **빠른 개발 속도 및 시장 출시 시간(Time-to-Market) 단축**: 단일 코드베이스와 핫 리로드 기능은 개발 과정을 획기적으로 가속화한다. iOS와 Android 버전을 동시에 개발하고 출시할 수 있어 시장에 제품을 더 빨리 선보일 수 있다.4
- **비용 효율성**: iOS와 Android 개발팀을 별도로 운영할 필요가 없으므로 초기 개발 비용과 지속적인 유지보수 비용을 크게 절감할 수 있다.4 이는 특히 자원이 제한적인 스타트업에게 큰 이점이다.
- **일관된 UI 및 비즈니스 로직**: 하나의 코드로 여러 플랫폼을 지원하므로, 플랫폼 간에 동일한 사용자 경험과 비즈니스 로직을 보장할 수 있다. 이는 브랜드 정체성을 유지하고 사용자 혼란을 줄이는 데 도움이 된다.8
- **매력적이고 사용자 정의 가능한 UI**: Flutter의 위젯 시스템과 자체 렌더링 엔진은 픽셀 하나하나까지 제어할 수 있는 유연성을 제공한다. 이를 통해 플랫폼의 제약 없이 매우 아름답고 독창적인 UI를 구현할 수 있다.1
- **강력한 커뮤니티와 구글의 지원**: 구글이 적극적으로 지원하고 있으며, 전 세계적으로 활발한 오픈소스 커뮤니티가 형성되어 있다. 이는 풍부한 문서, 지속적인 업데이트, 그리고 `pub.dev`를 통한 방대한 패키지 생태계를 의미한다.4


- **큰 애플리케이션 크기**: Flutter 앱은 스키아 렌더링 엔진과 위젯 세트를 포함한 프레임워크 자체를 앱 내에 번들로 포함해야 한다. 이로 인해 네이티브 앱에 비해 최종 애플리케이션의 파일 크기가 더 클 수 있다.6
- **Dart 언어 학습 곡선**: Flutter는 Dart 언어를 사용한다. Dart는 Java, C# 등 다른 C 스타일 객체 지향 언어와 유사하지만, JavaScript나 Java, Swift/Kotlin에 비해 상대적으로 덜 대중적이다. 따라서 기존 개발자들에게 약간의 학습 곡선이 존재할 수 있다.1
- **제한적인 서드파티 라이브러리**: Flutter의 패키지 생태계는 빠르게 성장하고 있지만, 네이티브 iOS나 Android 생태계만큼 성숙하지는 않았다. 매우 특수하거나 최신 네이티브 기능을 위한 서드파티 라이브러리가 존재하지 않을 수 있다.1
- **웹에서의 SEO 문제**: Flutter for Web은 UI를 캔버스(Canvas)에 렌더링하는 방식으로 작동한다. 이로 인해 검색 엔진 크롤러가 웹 페이지의 콘텐츠를 쉽게 분석하기 어려워, 검색 엔진 최적화(SEO)가 중요한 콘텐츠 중심의 웹사이트에는 적합하지 않을 수 있다.16

이러한 장단점을 종합해 볼 때, Flutter의 가치는 특정 유형의 프로젝트에서 극대화된다. 예를 들어, 여러 플랫폼에서 일관된 브랜드 경험을 제공해야 하는 소비자용 앱, 빠른 프로토타이핑과 시장 출시가 중요한 MVP(최소 기능 제품), 또는 사내 비즈니스용 애플리케이션 개발에 있어 Flutter는 탁월한 선택이다. 반면, 앱의 크기가 극도로 중요하거나, 특정 네이티브 SDK에 깊이 의존해야 하며 해당 SDK의 Flutter 플러그인이 없는 경우에는 신중한 고려가 필요하다. 결국, 프로젝트의 목표와 제약 조건을 분석하여 개발 속도와 브랜드 일관성의 가치가 잠재적인 단점을 상쇄하는지를 판단하는 것이 현명한 아키텍처 결정의 핵심이다.


이 장에서는 Flutter 개발을 시작하기 위해 필요한 모든 소프트웨어를 설치하고 구성하는 과정을 운영체제별로 상세하게 안내한다. 성공적인 개발 환경 구축은 원활한 학습과 개발의 첫걸음이다.


각 운영체제에 맞는 Flutter SDK와 필수 도구들을 설치하는 방법을 단계별로 설명한다.


Windows 환경에서 Flutter를 설정하는 과정은 다음과 같다.17

1. **시스템 요구 사항 확인**: Flutter 공식 문서에서 최신 시스템 요구 사항(운영체제 버전, 디스크 공간 등)을 확인한다.
2. **Git for Windows 설치**: Flutter는 버전 관리를 위해 Git을 사용한다. Git for Windows를 다운로드하여 설치한다.
3. **Flutter SDK 다운로드**: Flutter 공식 사이트의 SDK 아카이브에서 Windows용 최신 안정 버전의 `.zip` 파일을 다운로드한다.17
4. **SDK 압축 해제**: 다운로드한 `.zip` 파일의 압축을 푼다. 이때, 설치 경로는 `C:\Program Files`와 같이 관리자 권한이 필요하거나 공백, 특수 문자가 포함된 디렉터리를 피해야 한다. `C:\src\flutter` 또는 `C:\Users\<사용자명>\development\flutter`와 같은 경로를 권장한다.17
5. **환경 변수 설정**: Flutter 커맨드 라인 도구를 어느 위치에서든 사용하려면 `Path` 환경 변수를 업데이트해야 한다.
   - '시스템 환경 변수 편집'을 검색하여 실행한다.
   - '환경 변수...' 버튼을 클릭한다.
   - '사용자 변수' 섹션에서 'Path'를 찾아 선택하고 '편집'을 클릭한다.
   - '새로 만들기'를 클릭하고, 압축을 해제한 Flutter 디렉터리 내의 `bin` 폴더 전체 경로(예: `C:\src\flutter\bin`)를 추가한다.19
   - 모든 창에서 '확인'을 눌러 변경 사항을 저장한다.


macOS 환경 설정은 Intel 기반 Mac과 Apple Silicon 기반 Mac에 따라 약간의 차이가 있을 수 있다.21

1. **시스템 요구 사항 확인**: macOS 버전 및 디스크 공간 요구 사항을 확인한다.

2. **Rosetta 2 설치 (Apple Silicon Mac 전용)**: Apple Silicon(M1, M2 등) 칩을 사용하는 Mac에서는 일부 도구와의 호환성을 위해 Rosetta 2를 설치해야 한다. 터미널에서 다음 명령을 실행한다.21

   ```Bash
   sudo softwareupdate --install-rosetta --agree-to-license
   ```

3. **Flutter SDK 다운로드**: 공식 사이트에서 자신의 프로세서(Intel 또는 Apple Silicon)에 맞는 Flutter SDK `.zip` 파일을 다운로드한다.24

4. **SDK 압축 해제**: 원하는 위치에 압축을 푼다. 예를 들어, 홈 디렉터리 아래에 `development` 폴더를 만들고 그 안에 압축을 해제할 수 있다.

   ```Bash
   mkdir ~/development
   unzip ~/Downloads/flutter_macos_*.zip -d ~/development
   ```

5. **Path 환경 변수 설정**: 터미널에서 Flutter 명령어를 사용하기 위해 `PATH`를 설정해야 한다. 최신 macOS는 기본적으로 Zsh를 사용하며, 공식 문서에서는 `.zshenv` 파일에 경로를 추가할 것을 권장한다.21

   - 터미널에서 `nano ~/.zshenv` 또는 선호하는 편집기로 파일을 연다 (파일이 없으면 새로 생성된다).

   - 파일 끝에 다음 줄을 추가한다 (경로는 실제 압축 해제한 경로에 맞게 수정한다).

     ```Bash
     export PATH="$HOME/development/flutter/bin:$PATH"
     ```

   - 파일을 저장하고 터미널을 재시작하여 변경 사항을 적용한다.

6. **iOS 개발 도구 설치**: iOS 앱을 개발하고 시뮬레이터에서 실행하려면 Xcode와 CocoaPods를 설치해야 한다.21

   - **Xcode**: Mac App Store에서 Xcode를 설치한다. 설치 후 처음 실행하여 라이선스에 동의하고 추가 컴포넌트 설치를 완료한다.

   - **CocoaPods**: 터미널에서 다음 명령어로 CocoaPods를 설치한다.

     ```Bash
     sudo gem install cocoapods
     ```


Linux에서는 `snapd`를 이용한 간편한 설치 방법과 수동 설치 방법을 모두 지원한다.2

1. **필수 도구 설치**: 개발에 필요한 기본 도구들을 `apt-get` (Debian/Ubuntu 기준)을 통해 설치한다.30

   ```Bash
   sudo apt-get install clang cmake ninja-build pkg-config libgtk-3-dev
   ```

2. **설치 방법 선택**:

   - **방법 1: `snapd` 사용 (권장)**: 가장 간단한 방법으로, 터미널에 다음 명령어를 입력하여 Flutter를 설치한다.2

     ```Bash
     sudo snap install flutter --classic
     ```

   - **방법 2: 수동 설치**:

     - 공식 사이트에서 Linux용 Flutter SDK `.tar.xz` 파일을 다운로드한다.

     - 원하는 위치에 압축을 푼다 (예: `~/development`).

       ```Bash
       mkdir ~/development
       tar xf ~/Downloads/flutter_linux_*.tar.xz -C ~/development
       ```

     - 사용하는 셸에 맞게 `PATH` 환경 변수를 설정한다. 예를 들어, `bash`를 사용한다면 `~/.bash_profile` 또는 `~/.bashrc` 파일에 다음 줄을 추가한다.30

       ```Bash
       export PATH="$HOME/development/flutter/bin:$PATH"
       ```


SDK 설치 후에는 개발 생산성을 높이기 위해 통합 개발 환경(IDE)을 설정하고, 모든 구성 요소가 올바르게 설치되었는지 검증해야 한다.


Flutter 개발에는 Visual Studio Code(VS Code), Android Studio, IntelliJ IDEA가 주로 사용된다.15 각 IDE는 Flutter 개발을 지원하는 전용 플러그인 또는 확장 프로그램을 제공하며, 코드 자동 완성, 구문 강조, 디버깅, 핫 리로드 실행 등의 기능을 통합하여 제공한다.24

특히 최근 Flutter 공식 문서에서는 VS Code를 통한 개발 환경 구축을 적극적으로 권장하고 있다. VS Code의 Flutter 확장 프로그램은 'Flutter: New Project' 명령을 통해 Flutter SDK의 다운로드, 압축 해제, 경로 설정까지 자동으로 처리해주는 편리한 기능을 제공한다.17 이는 초보 개발자들이 겪을 수 있는 복잡하고 오류가 발생하기 쉬운 수동 설정 과정을 크게 단순화시켜준다. 이러한 변화는 개발자 경험(Developer Experience, DevEx)을 개선하여 프레임워크의 진입 장벽을 낮추려는 Flutter 팀의 전략적 방향성을 보여준다. 개발자가 복잡한 설정에 시간을 낭비하는 대신, Flutter의 핵심 가치인 '빠른 UI 개발'을 즉시 경험하게 함으로써 프레임워크에 대한 긍정적인 첫인상을 심어주는 것이다. 따라서 이 자습서에서는 VS Code를 사용한 통합 설치 방법을 우선적으로 권장한다.


모바일 앱 개발을 위해서는 Android Studio와 Android SDK가 필수적이다.

1. Android Studio 공식 사이트에서 설치 파일을 다운로드하여 설치한다.
2. Android Studio를 실행하고 설정 마법사를 따라 최신 Android SDK, SDK Command-line Tools, Build-Tools 등을 설치한다.18
3. **Android 가상 기기(AVD) 생성**:
   - Android Studio에서 'AVD Manager'를 연다.
   - 'Create Virtual Device'를 클릭하여 원하는 기기 모델과 안드로이드 시스템 이미지를 선택하여 가상 기기를 생성한다. 이 가상 기기는 앱을 테스트할 에뮬레이터 역할을 한다.18
4. **실제 기기 연결 (선택 사항)**: 실제 안드로이드 기기를 USB로 연결하여 테스트할 수도 있다. 이를 위해서는 기기에서 '개발자 옵션'과 'USB 디버깅'을 활성화해야 한다.


모든 설치와 설정이 완료된 후, 터미널(또는 명령 프롬프트)을 열고 다음 명령어를 실행하여 개발 환경을 최종적으로 검증한다.

```Bash
flutter doctor
```

`flutter doctor`는 Flutter 개발 환경의 상태를 진단하는 강력한 도구다. 이 명령어는 Flutter SDK, 연결된 기기, Android toolchain, Xcode(macOS), Chrome(웹 개발용), VS Code 등 각 구성 요소의 설치 상태를 확인하고, 문제가 있거나 추가 설정이 필요한 부분(예: Android 라이선스 동의)을 친절하게 알려준다.18 출력 결과에서 모든 항목 앞에 녹색 체크 표시(`[✓]`)가 나타나면 개발 환경 구축이 성공적으로 완료된 것이다.


Flutter 애플리케이션은 Dart 프로그래밍 언어로 작성된다. 따라서 Flutter를 효과적으로 사용하기 위해서는 Dart의 핵심 문법과 개념에 대한 이해가 필수적이다. 이 장에서는 Flutter 개발에 가장 중요하고 빈번하게 사용되는 Dart의 기능들을 집중적으로 다룬다.


Dart는 정적 타입을 지원하는 객체 지향 언어로, C 스타일의 친숙한 문법 구조를 가지고 있다.34


변수는 데이터를 저장하는 공간이다. Dart에서 변수를 선언하는 주요 키워드는 다음과 같다.

- `var`: 변수를 선언할 때 사용하며, 초기화되는 값을 기반으로 컴파일러가 타입을 자동으로 추론(Type Inference)한다.34

  ```Dart
  var name = 'Flutter'; // String 타입으로 추론됨
  var year = 2024;     // int 타입으로 추론됨
  ```

- **명시적 타입**: 변수의 타입을 직접 지정하여 선언할 수도 있다.

  ```Dart
  String framework = 'Flutter';
  int version = 3;
  ```

- `final`: 단 한 번만 값을 할당할 수 있는 변수를 선언한다. 일단 할당되면 변경할 수 없다. 런타임에 값이 결정될 수 있다.38

  ```Dart
  final String appName = 'MyAwesomeApp';
  // appName = 'NewApp'; // 오류 발생
  ```

- `const`: `final`과 유사하지만, 컴파일 시점에 값이 결정되는 상수(Compile-time constant)를 선언한다. `const` 변수는 반드시 컴파일 시점 상수로 초기화되어야 한다.38

  ```Dart
  const double pi = 3.14159;
  ```


Dart의 모든 변수는 객체(Object)다. 숫자, 함수, `null`조차도 객체다.34 주요 내장 데이터 타입은 다음과 같다.37

- **Numbers**: `int` (정수), `double` (실수). 이 둘은 모두 `num` 타입을 상속한다.
- **Strings**: `String` (문자열). 작은따옴표(`'`) 또는 큰따옴표(`"`)로 감싸서 표현한다.
- **Booleans**: `bool` (`true` 또는 `false`).
- **Collections**:
  - `List`: 순서가 있는 값의 모음 (배열과 유사).
  - `Set`: 순서가 없고 중복을 허용하지 않는 값의 모음.
  - `Map`: 키(Key)와 값(Value)의 쌍으로 이루어진 모음.


Null 안전성은 최신 Dart의 핵심 기능으로, 코드에서 `null` 참조로 인한 런타임 오류를 방지하는 데 큰 도움을 준다.

- 기본적으로 모든 변수는 `null` 값을 가질 수 없다(Non-nullable).

  ```Dart
  int a = 42;
  // a = null; // 컴파일 오류 발생
  ```

- 변수가 `null` 값을 가질 수 있도록 하려면, 타입 뒤에 물음표(`?`)를 붙여야 한다(Nullable).34

  ```Dart
  String? name; // null일 수 있음
  name = 'Dart';
  name = null; // 가능
  ```

- Nullable 변수의 값을 사용해야 할 때, 해당 값이 `null`이 아님을 확신한다면 느낌표(`!`)를 사용하여 `null`이 아님을 단언(Assertion)할 수 있다. 만약 이때 값이 `null`이면 런타임 오류가 발생한다.34

  ```Dart
  int? maybeValue = 5;
  int value = maybeValue!; // maybeValue가 null이 아님을 단언
  ```


프로그램의 실행 흐름을 제어하기 위한 구문들은 다른 언어들과 매우 유사하다.

- **조건문 (Conditionals)**: `if-else`와 `switch-case`를 사용하여 조건에 따라 다른 코드를 실행한다.43

  ```Dart
  if (isReady) {
    print('Ready!');
  } else {
    print('Not ready.');
  }
  ```

- **반복문 (Loops)**: `for`, `for-in`, `forEach`, `while`, `do-while`을 사용하여 특정 코드 블록을 반복 실행한다.43 특히 

  `for-in` 루프는 리스트와 같은 반복 가능한(Iterable) 객체의 각 요소를 순회하는 데 유용하다.

  ```Dart
  var numbers = ;
  for (var n in numbers) {
    print(n);
  }
  ```

- **반복 제어**: `break`는 반복문을 즉시 중단하고, `continue`는 현재 반복을 건너뛰고 다음 반복으로 넘어간다.47

- **단언 (Assertions)**: `assert` 문은 개발 중에만 활성화되며, 주어진 조건이 `false`일 경우 실행을 중단시킨다. 이는 버그를 조기에 발견하는 데 도움이 된다.43

  ```Dart
  assert(text!= null);
  ```


함수는 특정 작업을 수행하는 코드의 묶음이다. Dart에서 함수는 일급 객체(first-class object)로 취급되어 변수에 할당하거나 다른 함수의 인자로 전달할 수 있다.49

- **기본 구문**: 반환 타입, 함수 이름, 매개변수 목록, 그리고 함수 본문으로 구성된다.50

  ```Dart
  int add(int a, int b) {
    return a + b;
  }
  ```

- **화살표 구문 (Arrow Syntax)**: 함수 본문이 단일 표현식으로 이루어진 경우, `=>`를 사용하여 코드를 간결하게 작성할 수 있다.49

  ```Dart
  int add(int a, int b) => a + b;
  ```

- **매개변수 (Parameters)**: Dart는 다양한 종류의 매개변수를 지원한다.

  - **위치 매개변수 (Positional Parameters)**: `int add(int a, int b)`와 같이 순서대로 전달해야 하는 필수 매개변수.

  - **이름 있는 매개변수 (Named Parameters)**: 중괄호 `{}`로 묶어 정의하며, 호출 시 매개변수 이름을 명시해야 한다. `@required` 또는 `required` 키워드로 필수로 만들 수 있으며, 기본값을 지정할 수도 있다.49

    ```Dart
    void enableFlags({bool? bold, bool? hidden}) {... }
    // 호출: enableFlags(bold: true, hidden: false);
    ```

  - **선택적 위치 매개변수 (Optional Positional Parameters)**: 대괄호 ``로 묶어 정의하며, 호출 시 생략 가능하다.49

- **익명 함수 (Anonymous Functions)**: 이름이 없는 함수로, 클로저(closure)라고도 불린다. 주로 다른 함수의 인자로 전달될 때 사용된다. Flutter에서 버튼의 `onPressed` 콜백과 같이 이벤트 핸들러를 정의할 때 매우 흔하게 사용된다.49

  ```Dart
  var numbers = ;
  numbers.forEach((number) {
    print(number * 2);
  });
  ```


Dart는 클래스 기반의 객체 지향 프로그래밍(OOP)을 완벽하게 지원한다. 이는 Flutter의 위젯 시스템을 이해하는 데 필수적이다.

- **클래스와 객체**: `class` 키워드를 사용하여 클래스를 정의한다. 클래스는 객체를 생성하기 위한 청사진(blueprint)이다. 객체는 클래스의 인스턴스(instance)이며, 상태를 나타내는 인스턴스 변수(필드)와 동작을 나타내는 메서드(함수)를 가진다.54

  ```Dart
  class Point {
    double x = 0;
    double y = 0;
  
    void move(double newX, double newY) {
      x = newX;
      y = newY;
    }
  }
  ```

- **생성자 (Constructors)**: 객체를 생성하고 초기화하는 특별한 메서드다. 클래스 이름과 동일한 이름을 가지며, `Point(this.x, this.y);`와 같이 간결한 초기화 문법을 제공한다. 이름 있는 생성자(`ClassName.identifier`)를 통해 여러 방식의 객체 생성을 지원할 수 있다.56

- **객체 생성 및 멤버 접근**: `new` 키워드는 선택 사항이며, 일반적으로 생략한다. 객체의 멤버(변수나 메서드)에 접근할 때는 점(`.`) 연산자를 사용한다.54

  ```Dart
  var p = Point(10, 20);
  p.move(100, 200);
  print(p.x);
  ```

Flutter의 모든 위젯은 사실상 클래스다. 예를 들어, `Text('Hello')`는 `Text` 클래스의 인스턴스를 생성하는 것이며, `'Hello'`는 생성자를 통해 `data`라는 인스턴스 변수에 전달된다. 이처럼 OOP 개념은 Flutter의 구조를 이해하는 데 직접적으로 연결된다.


현대 애플리케이션에서 네트워크 요청, 파일 읽기/쓰기, 데이터베이스 접근과 같은 작업은 필수적이다. 이러한 작업들은 시간이 걸릴 수 있으며, 완료될 때까지 UI를 멈추게(blocking) 해서는 안 된다. Dart는 단일 스레드 이벤트 루프 모델을 기반으로 비동기 프로그래밍을 처리한다.57

- **`Future`**: 비동기 작업의 결과를 나타내는 객체다. 작업이 시작되면 `Future` 객체가 즉시 반환된다. 이 `Future`는 미래의 특정 시점에 값(성공) 또는 오류(실패)로 '완료(completed)'될 것을 약속한다. 그전까지는 '미완료(uncompleted)' 상태에 있다.58

- **`async`와 `await`**: Dart는 비동기 코드를 동기 코드처럼 간결하고 읽기 쉽게 작성할 수 있도록 `async`와 `await` 키워드를 제공한다.

  - `async`: 함수 본문 앞에 `async`를 붙이면, 해당 함수는 비동기 함수가 되며 항상 `Future`를 반환한다.60
  - `await`: `async` 함수 내에서만 사용할 수 있다. `await` 키워드를 `Future`를 반환하는 표현식 앞에 붙이면, 해당 `Future`가 완료될 때까지 함수의 실행을 '일시 중지'시킨다. 이때 프로그램 전체가 멈추는 것이 아니라, 이벤트 루프는 다른 작업을 처리할 수 있다. `Future`가 완료되면, `await`는 그 결과를 반환하고 함수는 다음 코드를 계속 실행한다.58

  ```Dart
  Future<String> fetchData() async {
    // 네트워크 요청을 시뮬레이션
    await Future.delayed(Duration(seconds: 2));
    return 'Data fetched!';
  }
  
  void main() async {
    print('Fetching data...');
    String data = await fetchData();
    print(data); // 2초 후에 'Data fetched!'가 출력됨
  }
  ```

과거에는 `.then()` 콜백을 사용하여 `Future`의 결과를 처리했지만, 이는 코드를 중첩시키고 가독성을 떨어뜨리는 '콜백 지옥(callback hell)'을 유발할 수 있었다.62

`async/await`는 이러한 문제를 해결하기 위해 도입된 언어 기능으로, 비동기 코드의 논리적 흐름을 순차적으로 파악하기 쉽게 만들어준다. 오류 처리 역시 `try-catch` 구문을 사용하여 동기 코드와 동일한 방식으로 자연스럽게 처리할 수 있다.63 따라서 현대 Dart 및 Flutter 개발에서는 비동기 작업을 처리할 때 `async/await`를 사용하는 것이 표준적인 모범 사례로 간주된다. 이는 코드의 유지보수성과 안정성을 크게 향상시키는 핵심적인 패러다임 전환이다.


이 장에서는 Flutter의 가장 핵심적인 개념인 위젯에 대해 깊이 있게 탐구한다. 다양한 위젯의 종류를 구분하고, 위젯의 상태가 어떻게 변화하고 UI에 반영되는지에 대한 메커니즘을 명확히 설명한다.


Flutter의 UI 구축 패러다임은 "모든 것은 위젯이다(Everything is a widget)"라는 한 문장으로 요약될 수 있다.13 이는 단순히 화면에 보이는 버튼이나 텍스트뿐만 아니라, 눈에 보이지 않는 요소들까지 모두 위젯으로 취급한다는 의미다. 예를 들어, 화면의 중앙에 요소를 배치하는 `Center` 위젯, 자식 위젯 주변에 여백을 주는 `Padding` 위젯, 위젯들을 가로나 세로로 배열하는 `Row`와 `Column` 위젯, 그리고 사용자의 터치 입력을 감지하는 `GestureDetector` 위젯까지 모두 위젯의 한 종류다.13

Flutter의 위젯은 UI의 특정 부분을 어떻게 구성하고 어떻게 보여줄지에 대한 '불변의(immutable) 선언'이다.14 개발자는 작고 단일 목적을 가진 위젯들을 레고 블록처럼 조합(composition)하여 복잡하고 정교한 UI를 구축한다. 예를 들어, 아이콘과 텍스트를 나란히 표시하려면 `Icon` 위젯과 `Text` 위젯을 `Row` 위젯의 자식으로 포함시키면 된다. 이러한 조합적 접근 방식은 코드의 재사용성을 높이고 UI 구조를 직관적으로 이해할 수 있게 해준다. 이는 다른 프레임워크에서 흔히 볼 수 있는 뷰(View)와 뷰 컨트롤러(View Controller)를 분리하는 전통적인 패러다임과는 차별화되는 강력하고 유연한 방식이다.


Flutter의 위젯은 크게 두 가지 종류로 나뉜다: `StatelessWidget`과 `StatefulWidget`. 이 둘의 차이점을 이해하는 것은 Flutter 개발의 가장 중요한 첫걸음이다.

- **`StatelessWidget` (상태 없는 위젯)**: 이름 그대로, 내부적으로 변경 가능한 '상태(state)'를 가지지 않는 위젯이다. 일단 생성되면 위젯의 속성(properties)은 변하지 않는다. 즉, 불변(immutable)이다.65

  `StatelessWidget`은 오직 생성자를 통해 전달받은 데이터와 `BuildContext`에만 의존하여 자신의 모습을 결정한다. 화면에 정적인 콘텐츠를 표시할 때 주로 사용되며, `Text`, `Icon`, `Image`, `Container` 등이 대표적인 예다.

- **`StatefulWidget` (상태 있는 위젯)**: 동적으로 변경될 수 있는 상태를 가지는 위젯이다. 사용자의 상호작용(예: 버튼 클릭, 텍스트 입력)이나 데이터 수신 등에 따라 위젯의 모습이 바뀔 필요가 있을 때 사용된다.65

  `StatefulWidget`은 두 개의 클래스로 구성된다.

  1. `StatefulWidget`을 상속하는 위젯 클래스: 이 클래스 자체는 불변이다.

  2. `State`를 상속하는 상태 클래스: 이 클래스가 실제로 변경 가능한 상태 데이터를 보유하며, 위젯의 생명주기 동안 유지된다. `State` 객체 안에서 `setState()` 메서드를 호출하여 상태 변경을 프레임워크에 알리고 UI를 다시 그리도록 요청한다.68

     `Checkbox`, `TextField`, `Slider` 등이 대표적인 예다.

초보자들이 흔히 혼란을 겪는 지점은, 예를 들어 화면의 텍스트가 바뀔 때 `Text` 위젯 자체가 상태를 가져야 한다고 생각하는 것이다.70 그러나 Flutter의 선언적 패러다임은 다르게 작동한다. 핵심은, 상태가 변경되면 해당 상태를 가진 부모 `StatefulWidget`이 자신의 `build` 메서드를 다시 호출하여 *새로운* `StatelessWidget`을 생성한다는 점이다. 즉, 기존의 `Text` 위젯을 수정하는 것이 아니라, 새로운 내용이 담긴 `Text` 위젯으로 교체하는 것이다.70

이것이 효율적으로 작동하는 이유는 위젯 자체가 매우 가벼운 '설계도'에 불과하기 때문이다. 실제로 화면에 픽셀을 그리는 무거운 작업은 렌더링 엔진이 담당한다. Flutter 프레임워크는 상태 변경 시 생성된 새로운 위젯 트리와 이전 위젯 트리를 비교(diffing)하여, 실제 변경이 필요한 최소한의 부분만 다시 그리는 데 고도로 최적화되어 있다.14

`StatelessWidget`의 불변성은 이러한 효율적인 비교 및 렌더링 알고리즘을 가능하게 하는 전제 조건이다.

따라서 개발자는 가능한 한 `StatelessWidget`을 사용하고, 위젯이 외부에서 제공되지 않는 자신만의 내부 상태를 관리해야 할 때만 `StatefulWidget`을 사용해야 한다. 이는 "상태를 위로 끌어올리기(lift state up)"라는 Flutter 아키텍처의 중요한 원칙과도 연결된다.

| 기능                     | `StatelessWidget`                                            | `StatefulWidget`                                             |
| ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **상태 (State)**         | 불변(Immutable). 속성은 `final`로 선언됨.                    | 가변(Mutable). 상태는 별도의 `State` 객체에 저장됨.          |
| **생명주기 (Lifecycle)** | 생성자와 `build` 메서드로 구성됨.                            | `createState()`, `initState()`, `build()`, `setState()`, `dispose()` 등 더 복잡한 생명주기를 가짐. |
| **사용 사례**            | 자신의 설정과 `BuildContext`에만 의존하는 UI. (예: 정적 텍스트, 아이콘) | 사용자 상호작용이나 데이터 변경에 따라 동적으로 변하는 UI. (예: 체크박스, 슬라이더) |
| **성능**                 | 관리할 상태가 없어 더 가볍고 성능이 좋음.                    | 상태 객체를 유지하기 위한 약간의 오버헤드가 있음.            |
| **핵심 원리**            | UI의 정적인 부분을 기술함.                                   | UI의 동적인 부분을 기술하며, 새로운 상태로 다시 빌드될 수 있음. |


`setState()`는 `StatefulWidget`의 심장과도 같은 메서드다. `State` 클래스 내에서만 호출할 수 있으며, Flutter 프레임워크에게 "이 위젯의 내부 상태가 변경되었으니, UI를 업데이트해야 한다"고 알리는 신호 역할을 한다.71

`setState()`가 호출되면 다음과 같은 과정이 일어난다.

1. **상태 변경**: `setState()`에 전달된 콜백 함수 내부에서 상태 변수의 값을 변경한다. UI에 반영되어야 하는 모든 상태 변경은 반드시 이 콜백 함수 안에서 이루어져야 한다.71

   ```Dart
   int _counter = 0;
   
   void _incrementCounter() {
     setState(() {
       // 이 콜백 함수 안에서 상태를 변경한다.
       _counter++;
     });
   }
   ```

2. **위젯 '더티' 상태로 표시**: 프레임워크는 해당 `State` 객체를 '더티(dirty)' 상태로 표시하여 다음 프레임에 다시 빌드해야 함을 인지한다.

3. **재빌드(Rebuild) 예약**: 프레임워크는 해당 위젯의 `build()` 메서드를 다시 호출하도록 예약한다.

4. **UI 업데이트**: `build()` 메서드가 실행되면서 변경된 상태 값을 기반으로 새로운 위젯 트리가 생성되고, 프레임워크는 이전 트리와의 차이점을 계산하여 화면을 효율적으로 다시 그린다.68

**`setState()` 사용 시 주의사항:**

- **상태 변경은 콜백 내부에서**: 상태 변수를 `setState()` 콜백 밖에서 변경하면, 데이터는 바뀌지만 `build()` 메서드가 호출되지 않아 UI가 업데이트되지 않는다.

- **`build()` 메서드에서 호출 금지**: `build()` 메서드 내에서 `setState()`를 호출하면, 재빌드가 무한 반복되어 앱이 멈출 수 있다.72

  `setState()`는 버튼의 `onPressed` 콜백과 같이 사용자의 입력이나 이벤트에 대한 응답으로 호출되어야 한다.

- **불필요한 호출 자제**: `setState()`는 위젯과 그 하위 트리의 재빌드를 유발하므로, 불필요하게 자주 호출하면 성능 저하의 원인이 될 수 있다. 상태가 실제로 변경되었을 때만 호출해야 한다.73

`setState()`는 Flutter에서 가장 기본적인 로컬 상태 관리 메커니즘이다. 데이터의 변화와 시각적 변화를 연결하는 이 핵심적인 다리를 이해하는 것은 동적인 Flutter 앱을 만드는 데 있어 필수적이다.


이 장에서는 실제 애플리케이션을 만드는 데 필요한 핵심 위젯들을 실용적인 예제와 함께 다룬다. 기본적인 UI 요소부터 복잡한 레이아웃 구성, 동적 목록 표시에 이르기까지, Flutter UI 개발의 실질적인 기술을 익힌다.


모든 UI의 기초가 되는 가장 빈번하게 사용되는 위젯들이다.


`Container`는 Flutter에서 가장 다재다능한 위젯 중 하나로, 다른 위젯을 담는 상자 역할을 한다. 단순히 자식 위젯을 감싸는 것 외에도 다양한 스타일링, 위치 지정, 크기 조절 기능을 제공한다.74

- **주요 속성**:
  - `child`: 컨테이너 내부에 위치할 단일 위젯.
  - `color`: 컨테이너의 배경색.
  - `width`, `height`: 컨테이너의 너비와 높이를 명시적으로 지정.
  - `padding`: 컨테이너의 경계와 자식 위젯 사이의 내부 여백. `EdgeInsets` 객체를 사용한다.
  - `margin`: 컨테이너의 경계와 다른 위젯 사이의 외부 여백.
  - `alignment`: 컨테이너 내에서 자식 위젯을 정렬.
  - `decoration`: 컨테이너를 꾸미는 데 사용. `BoxDecoration`을 통해 배경색, 테두리(`border`), 그림자(`boxShadow`), 둥근 모서리(`borderRadius`) 등을 복합적으로 설정할 수 있다.

**중요 규칙**: `color` 속성과 `decoration` 속성은 동시에 사용할 수 없다. `decoration`을 사용하면서 배경색을 지정하려면 `BoxDecoration`의 `color` 속성을 사용해야 한다.74

```Dart
Container(
  width: 200,
  height: 100,
  padding: const EdgeInsets.all(16.0),
  margin: const EdgeInsets.all(10.0),
  alignment: Alignment.center,
  decoration: BoxDecoration(
    color: Colors.blue,
    borderRadius: BorderRadius.circular(12),
    border: Border.all(color: Colors.black, width: 2),
  ),
  child: const Text(
    'Hello Container',
    style: TextStyle(color: Colors.white),
  ),
)
```


`Text` 위젯은 화면에 텍스트 문자열을 표시하는 데 사용된다. Flutter UI에서 가장 기본적인 요소 중 하나다.77

- **주요 속성**:
  - `data`: 표시할 `String` 데이터.
  - `style`: 텍스트의 스타일을 지정. `TextStyle` 객체를 사용하여 `fontSize`, `fontWeight`, `color`, `fontFamily` 등을 설정한다.
  - `textAlign`: 텍스트의 정렬 방식(예: `TextAlign.center`).
  - `overflow`: 텍스트가 할당된 공간을 초과할 때 처리하는 방식(예: `TextOverflow.ellipsis`는 말줄임표 '...'로 표시).
  - `maxLines`: 텍스트가 표시될 최대 줄 수.

하나의 `Text` 위젯 안에서 여러 스타일을 혼합하여 사용하고 싶을 때는 `RichText` 위젯과 `TextSpan` 객체를 조합하여 사용한다.78

```Dart
Text(
  'This is a long text that might overflow.',
  textAlign: TextAlign.center,
  maxLines: 1,
  overflow: TextOverflow.ellipsis,
  style: TextStyle(
    fontSize: 24,
    fontWeight: FontWeight.bold,
    color: Colors.deepPurple,
  ),
)
```


- **`Image`**: 다양한 소스(네트워크, 로컬 자산, 메모리 등)로부터 이미지를 표시한다. `Image.network()`, `Image.asset()`, `Image.file()`과 같은 생성자를 사용하여 쉽게 이미지를 로드할 수 있다.
- **`Icon`**: 머티리얼 디자인 아이콘과 같은 아이콘 폰트의 글리프(glyph)를 표시한다. `Icons` 클래스에 미리 정의된 수많은 아이콘을 사용할 수 있다.


위젯들을 화면에 배치하고 정렬하는 방법을 결정하는 위젯들이다. Flutter의 레이아웃 시스템은 "제약 조건은 아래로 흐르고, 크기는 위로 흐르며, 부모가 위치를 설정한다(Constraints flow down. Sizes flow up. Parents set positions)"는 간단하지만 강력한 규칙에 의해 지배된다.11 이 규칙을 이해하는 것은 복잡한 레이아웃 문제를 해결하는 열쇠다. 위젯은 부모로부터 받을 수 있는 크기 제약(최소/최대 너비 및 높이) 내에서 자신의 크기를 결정하고, 그 크기를 부모에게 알린다. 그러면 부모 위젯이 최종적으로 자식의 위치를 결정한다. 따라서 레이아웃 문제를 디버깅할 때는 단일 위젯의 속성만이 아니라 전체 위젯 트리의 관계를 고려해야 한다.


`Row`와 `Column`은 UI를 구성하는 가장 기본적인 다중 자식 레이아웃 위젯이다.

- `Row`: 자식 위젯들을 수평(가로)으로 나란히 배치한다.
- `Column`: 자식 위젯들을 수직(세로)으로 차례대로 배치한다.
- **주요 속성**:
  - `children`: 배치할 위젯들의 `List`.
  - `mainAxisAlignment`: 주축(Row는 가로, Column은 세로) 방향으로 자식들을 정렬하는 방식. (예: `MainAxisAlignment.start`, `center`, `end`, `spaceBetween`, `spaceAround`, `spaceEvenly`).
  - `crossAxisAlignment`: 교차축(Row는 세로, Column은 가로) 방향으로 자식들을 정렬하는 방식. (예: `CrossAxisAlignment.start`, `center`, `end`, `stretch`).

```Dart
Row(
  mainAxisAlignment: MainAxisAlignment.spaceEvenly,
  children: <Widget>[
    Icon(Icons.star),
    Icon(Icons.star),
    Icon(Icons.star),
  ],
)
```


`Row`나 `Column` 내에서 특정 자식 위젯이 남은 공간을 모두 차지하도록 만들고 싶을 때 사용한다. 이는 반응형 레이아웃을 만드는 데 매우 유용하다.14

- **주요 속성**:
  - `flex`: 여러 `Expanded` 위젯이 있을 때, 남은 공간을 차지할 비율을 지정한다. `flex` 값이 클수록 더 많은 공간을 할당받는다.

```Dart
Row(
  children: <Widget>[
    Expanded(
      flex: 2,
      child: Container(color: Colors.blue),
    ),
    Expanded(
      flex: 1,
      child: Container(color: Colors.red),
    ),
  ],
)
```

위 예제에서 파란색 컨테이너는 빨간색 컨테이너보다 2배의 공간을 차지하게 된다.


위젯들을 겹쳐서 배치해야 할 때 `Stack` 위젯을 사용한다. 마치 여러 장의 종이를 쌓는 것처럼, `children` 리스트의 순서대로 위젯이 아래에서 위로 쌓인다.14

`Positioned` 위젯은 `Stack`의 자식으로만 사용될 수 있으며, `Stack`의 경계로부터 특정 위치(`top`, `bottom`, `left`, `right`)에 자식 위젯을 정밀하게 배치하는 데 사용된다.

```Dart
Stack(
  children: <Widget>[
    Container(
      width: 200,
      height: 200,
      color: Colors.red,
    ),
    Positioned(
      top: 20,
      left: 20,
      child: Container(
        width: 100,
        height: 100,
        color: Colors.blue,
      ),
    ),
  ],
)
```


많은 수의 아이템을 스크롤 가능한 목록 형태로 보여줄 때 사용되는 효율적인 위젯들이다.


`ListView.builder`는 화면에 보이는 아이템만 동적으로 생성(lazy loading)하므로, 수백, 수천 개의 아이템이 포함된 긴 목록을 표시할 때 매우 효율적이다.92

- **주요 속성**:
  - `itemCount`: 목록에 포함될 전체 아이템의 개수.
  - `itemBuilder`: 각 아이템의 인덱스를 받아 해당 위치에 표시될 위젯을 반환하는 함수. 이 함수는 화면에 아이템이 보여야 할 때만 호출된다.

```Dart
ListView.builder(
  itemCount: 100,
  itemBuilder: (BuildContext context, int index) {
    return ListTile(
      leading: Icon(Icons.list),
      title: Text('Item $index'),
    );
  },
)
```


`GridView`는 아이템을 2차원 격자 형태로 배열하는 스크롤 가능한 위젯이다. 갤러리나 상품 목록 같은 UI에 적합하다.94

- **주요 생성자**:
  - `GridView.count`: 교차축(스크롤 방향의 수직 방향)에 표시될 아이템의 개수를 고정한다.
  - `GridView.extent`: 교차축 방향으로 각 아이템의 최대 크기를 지정하면, 화면 크기에 따라 아이템 개수가 자동으로 조절되어 반응형 레이아웃에 유리하다.
  - `GridView.builder`: `ListView.builder`와 마찬가지로, 많은 수의 아이템을 효율적으로 표시하기 위해 아이템을 동적으로 생성한다.


Flutter는 머티리얼 디자인 시스템을 구현하는 풍부한 위젯 세트를 제공하여, 일관되고 아름다운 UI를 쉽게 만들 수 있도록 돕는다.


`Scaffold`는 머티리얼 디자인 앱의 기본적인 시각적 레이아웃 구조를 구현하는 위젯이다. 앱의 최상위 구조를 잡는 데 사용되며, `appBar`, `body`, `floatingActionButton`, `drawer` 등 다양한 UI 요소를 배치할 수 있는 '슬롯'을 제공한다.98


`AppBar`는 일반적으로 `Scaffold`의 `appBar` 속성에 사용되며, 화면 상단에 툴바를 표시한다. 주로 페이지 제목, 액션 버튼, 탭 등을 포함한다.99


`Card`는 관련된 정보를 하나의 그룹으로 묶어 표시하는 데 사용되는 머티리얼 디자인 컴포넌트다. 약간의 그림자 효과(`elevation`)와 둥근 모서리를 가지고 있어 콘텐츠를 시각적으로 구분하고 강조하는 데 효과적이다.100


애플리케이션이 복잡해짐에 따라, 여러 위젯과 화면에 걸쳐 데이터를 공유하고 일관성을 유지하는 것이 중요해진다. 이 장에서는 `setState()`를 넘어서는, 더 확장 가능하고 체계적인 상태 관리(State Management) 기법들을 다룬다.


앞서 배운 `setState()`는 단일 `StatefulWidget` 내의 지역적(ephemeral) 상태를 관리하는 데는 훌륭하다.106 하지만 앱의 규모가 커지면 여러 문제가 발생한다. 예를 들어, 사용자의 로그인 정보나 앱의 테마 설정과 같은 전역적(global) 상태는 앱의 여러 부분에서 접근하고 수정해야 한다.

이러한 상태를 최상위 위젯에 두고 생성자를 통해 하위 위젯으로 계속 전달하는 방식("prop drilling"이라고도 함)은 코드를 복잡하게 만들고 유지보수를 어렵게 한다.107 한 위젯의 상태 변경이 직접 관련 없는 수많은 중간 위젯들의 재빌드를 유발하여 성능 저하를 일으킬 수도 있다. 이러한 문제를 해결하기 위해, 상태를 UI 코드로부터 분리하고, 필요한 위젯에서만 효율적으로 상태에 접근할 수 있도록 돕는 전문적인 상태 관리 솔루션이 필요하게 된다.106


`provider`는 Flutter 팀에서 공식적으로 추천하는 간단하고 유연한 상태 관리 패키지 중 하나다. 특히 Flutter를 처음 접하는 개발자에게 권장되며, `InheritedWidget`을 기반으로 하지만 이를 더 쉽고 효율적으로 사용할 수 있도록 추상화한 래퍼(wrapper)다.106 `provider`를 사용하기 위해서는 세 가지 핵심 개념을 이해해야 한다.


`ChangeNotifier`는 상태를 보유하고, 그 상태가 변경될 때마다 '리스너(listener)'들에게 변경 사실을 알리는 역할을 하는 간단한 클래스다. 상태를 관리하고자 하는 클래스가 `ChangeNotifier`를 `extends`(상속)하거나 `with`(믹스인)하여 구현한다. 상태가 변경되는 로직이 포함된 메서드 내에서, 변경이 끝난 후 `notifyListeners()`를 호출하면 이 `ChangeNotifier`를 구독하고 있는 모든 위젯에게 업데이트 신호를 보낸다.107

```Dart
import 'package:flutter/foundation.dart';

class CounterModel extends ChangeNotifier {
  int _count = 0;
  int get count => _count;

  void increment() {
    _count++;
    notifyListeners(); // 상태 변경 후 리스너들에게 알림
  }
}
```


`ChangeNotifierProvider`는 `ChangeNotifier`의 인스턴스를 위젯 트리의 특정 지점부터 그 하위 위젯들에게 제공(provide)하는 역할을 하는 위젯이다. 일반적으로 앱의 최상단이나 상태를 공유해야 하는 위젯들의 공통 조상 위치에 배치한다. 이렇게 함으로써 하위 위젯 어디서든 해당 상태 모델에 접근할 수 있게 된다.111

```Dart
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (context) => CounterModel(),
      child: const MyApp(),
    ),
  );
}
```


하위 위젯에서 제공된 상태에 접근하고 사용하는 방법은 주로 두 가지다.

1. **`Consumer<T>` 위젯**: `Consumer` 위젯은 위젯 트리에서 특정 타입(`T`)의 `Provider`를 찾아, 그 상태가 변경될 때마다 자신의 `builder` 함수를 다시 실행하여 UI를 업데이트한다. `Consumer`는 전체 위젯을 재빌드하는 대신, UI의 특정 부분만 정확하게 재빌드할 수 있어 성능 최적화에 유리하다.110

   Dart

   ```
   Consumer<CounterModel>(
     builder: (context, counter, child) {
       return Text('Count: ${counter.count}');
     },
   )
   ```

2. **`Provider.of<T>(context)` 또는 확장 메서드**: `build` 메서드 내에서 `Provider.of<CounterModel>(context)`를 호출하여 상태 모델에 직접 접근할 수 있다. 더 간결한 방법으로 `context.watch<T>()`와 `context.read<T>()` 확장 메서드가 제공된다.113

   - `context.watch<T>()`: 상태를 '구독'한다. 상태가 변경될 때마다 해당 위젯의 `build` 메서드를 다시 실행시킨다. UI에 상태 값을 표시할 때 사용한다.
   - `context.read<T>()`: 상태를 '한 번만 읽는다'. 상태 변경을 구독하지 않으므로, UI를 재빌드하지 않는다. 버튼 클릭 시 상태를 변경하는 메서드를 호출하는 것처럼, 상태 변경에 반응할 필요가 없는 로직에 사용한다.

   Dart

   ```
   // UI에 값을 표시 (상태 변경 시 재빌드됨)
   Text('Count: ${context.watch<CounterModel>().count}')
   
   // 버튼 클릭 시 상태 변경 메서드 호출 (재빌드 불필요)
   ElevatedButton(
     onPressed: () => context.read<CounterModel>().increment(),
     child: const Text('Increment'),
   )
   ```


애플리케이션의 규모가 더욱 커지고 비즈니스 로직이 복잡해지면, `Provider`보다 더 구조화된 아키텍처 패턴이 필요할 수 있다. BLoC(Business Logic Component) 패턴은 이러한 요구에 부응하는 강력한 상태 관리 솔루션이다.

BLoC의 핵심 철학은 UI와 비즈니스 로직을 완전히 분리하는 것이다.114 UI는 오직 '이벤트(Event)'를 BLoC에 전달하고, BLoC가 처리한 결과인 '상태(State)'를 받아 화면에 그리기만 한다. UI는 비즈니스 로직이 어떻게 작동하는지 전혀 알 필요가 없다.

- **핵심 구성 요소**:
  - **Events**: UI에서 발생하는 사용자 입력이나 시스템 이벤트 (예: `IncrementButtonPressed`, `FetchDataEvent`).
  - **States**: UI가 특정 시점에 보여줘야 할 모습에 대한 정보 (예: `CounterInitial`, `CounterLoading`, `CounterSuccess(value: 1)`).
  - **BLoC**: 이벤트를 입력으로 받아 비즈니스 로직을 처리한 후, 새로운 상태를 출력으로 내보내는 중간 계층. BLoC는 스트림(Stream)을 사용하여 이벤트와 상태를 비동기적으로 처리한다.115

`flutter_bloc` 패키지는 `BlocProvider`, `BlocBuilder`, `BlocListener`와 같은 위젯을 제공하여 BLoC 패턴 구현을 용이하게 해준다.114

`Provider`와 BLoC의 선택은 프로젝트의 복잡성에 따라 결정된다. `Provider`는 배우기 쉽고 빠르게 적용할 수 있어 중소 규모의 앱에 적합하다. 반면, BLoC는 초기 설정이 다소 복잡하지만, 비즈니스 로직과 UI의 명확한 분리를 강제하여 대규모 애플리케이션의 테스트 용이성과 유지보수성을 크게 향상시킨다. 개발자의 여정은 `setState`로 시작하여, 상태 공유의 필요성이 생기면 `Provider`를 도입하고, 애플리케이션의 복잡도가 비즈니스 로직의 엄격한 분리를 요구하는 시점에 BLoC를 고려하는 점진적인 방식으로 이루어지는 것이 일반적이다.


현대 애플리케이션은 대부분 서버와 통신하여 데이터를 가져오거나 전송한다. 이 장에서는 Flutter 앱에서 네트워크 요청을 보내고, 서버로부터 받은 데이터를 처리하여 UI에 표시하는 전 과정을 다룬다.


`http` 패키지는 Flutter에서 HTTP 네트워크 요청을 처리하는 가장 표준적이고 간단한 방법이다.119


1. **의존성 추가**: 프로젝트의 `pubspec.yaml` 파일에 `http` 패키지를 추가한다. 터미널에서 다음 명령어를 실행하면 자동으로 추가 및 설치된다.

   ```Bash
   flutter pub add http
   ```

2. **인터넷 권한 추가**: 플랫폼별로 인터넷 사용 권한을 설정해야 한다.

   - **Android**: `android/app/src/main/AndroidManifest.xml` 파일에 다음 권한을 추가한다.119

     ```XML
     <uses-permission android:name="android.permission.INTERNET" />
     ```

   - **macOS**: `macos/Runner/DebugProfile.entitlements`와 `macos/Runner/Release.entitlements` 파일에 네트워크 클라이언트 권한을 추가한다.121

     ```XML
     <key>com.apple.security.network.client</key>
     <true/>
     ```

3. **패키지 임포트**: `http` 패키지를 사용할 파일 상단에 다음 코드를 추가한다. 관례적으로 `as http`를 사용하여 네임스페이스 충돌을 방지한다.

   ```Dart
   import 'package:http/http.dart' as http;
   ```

요청 보내기

- **GET 요청**: 서버로부터 데이터를 가져올 때 사용한다. `http.get()` 메서드는 `Future<http.Response>`를 반환한다.119

  ```Dart
  Future<void> fetchData() async {
    final url = Uri.parse('https://jsonplaceholder.typicode.com/posts/1');
    final response = await http.get(url);
  
    if (response.statusCode == 200) {
      // 성공
      print('Response body: ${response.body}');
    } else {
      // 실패
      throw Exception('Failed to load data');
    }
  }
  ```

- **POST 요청**: 서버에 새로운 데이터를 생성할 때 사용한다. `http.post()` 메서드는 `body`와 `headers`를 인자로 받는다.120

  ```Dart
  Future<void> createData() async {
    final url = Uri.parse('https://jsonplaceholder.typicode.com/posts');
    final response = await http.post(
      url,
      headers: <String, String>{
        'Content-Type': 'application/json; charset=UTF-8',
      },
      body: jsonEncode(<String, String>{
        'title': 'foo',
        'body': 'bar',
        'userId': '1',
      }),
    );
  
    if (response.statusCode == 201) { // 201 Created
      print('Response body: ${response.body}');
    } else {
      throw Exception('Failed to create data.');
    }
  }
  ```


서버와 데이터를 주고받을 때 가장 널리 사용되는 데이터 형식은 JSON(JavaScript Object Notation)이다. Flutter는 내장된 `dart:convert` 라이브러리를 통해 JSON 처리를 지원한다.123


JSON 문자열을 Dart 객체(주로 `Map` 또는 `List`)로 변환하는 과정이다. `jsonDecode()` 함수를 사용한다.123

```Dart
import 'dart:convert';

//...
if (response.statusCode == 200) {
  // JSON 문자열을 Map<String, dynamic>으로 변환
  final Map<String, dynamic> data = jsonDecode(response.body);
  print('Title: ${data['title']}');
}
```


`Map<String, dynamic>`을 직접 사용하는 것은 타입 안전성을 보장하지 못하고 오타에 취약하다. 따라서 JSON 구조에 맞는 모델 클래스를 만들고, `fromJson`이라는 이름의 팩토리 생성자(factory constructor)를 통해 `Map`을 타입이 명확한 Dart 객체로 변환하는 것이 좋다.124

```Dart
class Post {
  final int userId;
  final int id;
  final String title;
  final String body;

  Post({
    required this.userId,
    required this.id,
    required this.title,
    required this.body,
  });

  factory Post.fromJson(Map<String, dynamic> json) {
    return Post(
      userId: json['userId'],
      id: json['id'],
      title: json['title'],
      body: json['body'],
    );
  }
}

// 사용 예시
if (response.statusCode == 200) {
  final post = Post.fromJson(jsonDecode(response.body));
  print('Post Title: ${post.title}');
}
```


Dart 객체를 JSON 문자열로 변환하는 과정이다. `jsonEncode()` 함수를 사용한다. 모델 클래스에 `toJson()` 메서드를 구현해두면 `jsonEncode()`가 이를 자동으로 호출하여 직렬화한다.124


`FutureBuilder`는 비동기 작업(`Future`)의 상태에 따라 UI를 동적으로 구성하는 매우 유용한 위젯이다.119 네트워크 요청과 같이 시간이 걸리는 작업을 수행하고 그 결과를 화면에 표시해야 할 때, 로딩 중, 데이터 수신 성공, 오류 발생의 세 가지 상태를 깔끔하게 처리할 수 있다.

`FutureBuilder`는 `StatefulWidget`을 직접 만들어 로딩 상태, 데이터, 오류 상태를 수동으로 관리하는 복잡함을 추상화해준다. 이는 Flutter의 선언적 UI 철학을 잘 보여주는 예로, 개발자는 '어떤 상태일 때 어떤 UI를 보여줄지'만 정의하면 된다.

- **주요 속성**:
  - `future`: UI를 구성하는 데 사용할 `Future` 객체. 보통 API를 호출하는 함수가 된다.
  - `builder`: `BuildContext`와 `AsyncSnapshot`을 인자로 받는 함수. `AsyncSnapshot`은 `Future`의 현재 상태와 데이터를 담고 있다.
- **`AsyncSnapshot`의 주요 속성**:
  - `connectionState`: `Future`의 현재 연결 상태 (`ConnectionState.none`, `waiting`, `active`, `done`).
  - `hasData`: `Future`가 성공적으로 완료되어 데이터를 가지고 있는지 여부.
  - `data`: `Future`가 완료되었을 때의 결과 데이터.
  - `hasError`: `Future`가 오류와 함께 완료되었는지 여부.
  - `error`: `Future`가 완료되었을 때의 오류 객체.

```Dart
FutureBuilder<Post>(
  future: fetchPost(), // API를 호출하는 Future<Post> 타입의 함수
  builder: (context, snapshot) {
    if (snapshot.connectionState == ConnectionState.waiting) {
      // 로딩 중일 때
      return const CircularProgressIndicator();
    } else if (snapshot.hasError) {
      // 오류가 발생했을 때
      return Text('Error: ${snapshot.error}');
    } else if (snapshot.hasData) {
      // 데이터 수신에 성공했을 때
      return Text(snapshot.data!.title);
    } else {
      // 데이터가 없을 때 (일반적으로 발생하지 않음)
      return const Text('No data');
    }
  },
)
```

이처럼 `FutureBuilder`를 사용하면 비동기 데이터 흐름에 따른 UI 로직을 명확하고 간결하게 표현할 수 있다.


이 장에서는 앞에서 배운 모든 개념을 종합하여 실용적인 To-Do 리스트 애플리케이션을 처음부터 끝까지 만들어본다. 이 프로젝트를 통해 Flutter 개발의 전체적인 흐름을 경험하고, 각 기술 요소들이 어떻게 유기적으로 결합되는지 이해할 수 있다.


효율적인 개발과 유지보수를 위해 프로젝트를 시작하기 전에 기능과 구조를 설계한다.


개발할 To-Do 앱은 다음과 같은 핵심 기능을 갖는다.127

1. **할 일 추가**: 사용자가 새로운 할 일을 텍스트로 입력하여 목록에 추가할 수 있다.
2. **할 일 목록 조회**: 추가된 모든 할 일의 목록을 화면에 표시한다.
3. **할 일 완료 처리**: 각 할 일 항목의 완료 여부를 체크박스를 통해 변경할 수 있다.
4. **할 일 삭제**: 목록에서 특정 할 일을 제거할 수 있다.


프로젝트의 가독성과 확장성을 위해 다음과 같이 파일을 구조화한다. 이는 관심사의 분리(Separation of Concerns) 원칙을 따르는 좋은 습관이다.130

```
lib/
├── main.dart           # 앱의 진입점
├── models/
│   └── todo.dart       # Todo 데이터 모델 클래스
├── providers/
│   └── todo_provider.dart # 상태 관리를 위한 Provider 클래스
├── screens/
│   └── home_screen.dart  # 메인 UI 화면
└── widgets/
    └── task_tile.dart    # 개별 할 일 항목을 표시하는 위젯
```


`pubspec.yaml` 파일에 이 프로젝트에서 사용할 `provider`와 `http` 패키지를 추가한다.

```YAML
dependencies:
  flutter:
    sdk: flutter
  provider: ^6.0.0 # 버전은 최신 버전으로 확인
  http: ^1.0.0     # 버전은 최신 버전으로 확인
```

터미널에서 `flutter pub get` 명령을 실행하여 패키지를 설치한다.


사용자가 직접 상호작용할 화면을 위젯을 조합하여 구성한다.


이 화면은 앱의 메인 화면으로, 전체적인 구조를 잡는 `Scaffold` 위젯을 사용한다.

- `appBar`: `AppBar` 위젯을 사용하여 화면 상단에 'Todo List'라는 제목을 표시한다.
- `body`: `ListView.builder`를 사용하여 할 일 목록을 동적으로 표시한다. 목록이 비어있을 경우를 대비하여 중앙에 안내 메시지를 표시하는 로직을 추가할 수 있다.
- `floatingActionButton`: `FloatingActionButton`을 배치하여 사용자가 새로운 할 일을 추가하는 화면이나 다이얼로그를 띄울 수 있도록 한다.130


`ListView.builder`에서 각 할 일을 표시하기 위한 재사용 가능한 위젯을 만든다.

- `StatelessWidget`으로 생성하여 외부에서 `Todo` 객체를 전달받도록 한다.
- `ListTile` 또는 `Card` 위젯을 사용하여 구조를 잡는다.
- `leading`: `Checkbox`를 배치하여 할 일의 완료 상태를 표시하고 변경할 수 있게 한다.
- `title`: `Text` 위젯으로 할 일의 내용을 표시한다. 완료된 할 일은 취소선을 그어 시각적으로 구분한다.
- `trailing`: `IconButton`을 배치하여 해당 할 일을 삭제하는 기능을 연결한다.130


UI와 비즈니스 로직을 분리하기 위해 `Provider`를 사용하여 상태를 관리한다.


할 일 하나를 나타내는 데이터 클래스를 정의한다.

```Dart
class Todo {
  String id; // 고유 식별자
  String title;
  bool isDone;

  Todo({required this.id, required this.title, this.isDone = false});
}
```


애플리케이션의 상태(할 일 목록)와 관련 비즈니스 로직을 모두 포함하는 클래스를 만든다. 이 클래스는 `ChangeNotifier`를 상속한다.130

```Dart
import 'package:flutter/foundation.dart';
import '../models/todo.dart';

class TodoProvider extends ChangeNotifier {
  final List<Todo> _todos =;

  List<Todo> get todos => _todos;

  void addTodo(String title) {
    final newTodo = Todo(id: DateTime.now().toString(), title: title);
    _todos.add(newTodo);
    notifyListeners(); // 상태 변경 알림
  }

  void toggleTodoStatus(String id) {
    final todo = _todos.firstWhere((todo) => todo.id == id);
    todo.isDone =!todo.isDone;
    notifyListeners();
  }

  void deleteTodo(String id) {
    _todos.removeWhere((todo) => todo.id == id);
    notifyListeners();
  }
}
```


1. **`main.dart` 수정**: 앱의 최상위 위젯인 `MaterialApp`을 `ChangeNotifierProvider`로 감싸 `TodoProvider`를 전체 앱에 제공한다.131

   Dart

   ```
   void main() {
     runApp(
       ChangeNotifierProvider(
         create: (context) => TodoProvider(),
         child: const MyApp(),
       ),
     );
   }
   ```

2. **UI 위젯에서 상태 사용**: `home_screen.dart`와 `task_tile.dart`에서 `context.watch<TodoProvider>()`를 사용하여 할 일 목록을 가져와 화면에 표시하고, `context.read<TodoProvider>()`를 사용하여 `addTodo`, `toggleTodoStatus`, `deleteTodo`와 같은 메서드를 호출하여 상태를 변경한다.


실제 애플리케이션은 대부분 로컬 메모리가 아닌 원격 서버에 데이터를 저장한다. 이 심화 과정에서는 기존의 로컬 `List<Todo>`를 REST API와 연동하여 데이터를 관리하는 방법을 다룬다. JSONPlaceholder와 같은 공개 API를 예시로 사용할 수 있다.


네트워크 요청 로직을 별도의 클래스로 분리한다. 이 클래스는 `http` 패키지를 사용하여 CRUD(Create, Read, Update, Delete)에 해당하는 API 요청을 수행하는 메서드들을 포함한다.132

- `getTodos()`: `GET` 요청으로 모든 할 일 목록을 가져온다.
- `addTodo(Todo todo)`: `POST` 요청으로 새로운 할 일을 서버에 추가한다.
- `updateTodo(Todo todo)`: `PUT` 또는 `PATCH` 요청으로 기존 할 일의 상태(예: 완료 여부)를 업데이트한다.
- `deleteTodo(String id)`: `DELETE` 요청으로 특정 할 일을 서버에서 삭제한다.


기존 `TodoProvider`의 메서드들을 `async`로 변경하고, 로컬 리스트를 직접 조작하는 대신 위에서 만든 API 서비스 클래스의 메서드를 호출하도록 수정한다. API 호출이 성공적으로 완료된 후에 `notifyListeners()`를 호출하여 UI를 갱신한다. 또한, 데이터 로딩 상태를 관리하기 위한 `isLoading`과 같은 상태 변수를 추가할 수 있다.133


- `home_screen.dart`의 `initState` 또는 `FutureBuilder`를 사용하여 앱이 처음 시작될 때 `getTodos()`를 호출하여 서버로부터 데이터를 가져온다.
- 데이터를 가져오는 동안 로딩 인디케이터(`CircularProgressIndicator`)를 표시하여 사용자에게 작업이 진행 중임을 알린다.
- API 호출 중 발생할 수 있는 오류를 `try-catch`로 처리하고, 사용자에게 오류 메시지를 보여주는 로직을 추가한다.

이 과정을 통해 UI, 상태 관리, 데이터 소스(네트워크)가 명확하게 분리된, 실제 상용 앱과 유사한 견고한 아키텍처를 경험하게 된다.


이 자습서를 통해 Flutter의 기초부터 실전 프로젝트까지의 여정을 마쳤다. 하지만 Flutter의 세계는 훨씬 더 넓고 깊다. 이 장에서는 여러분이 Flutter 개발자로서 계속 성장해 나갈 수 있는 다음 단계와 유용한 자료들을 소개한다.


대부분의 앱은 여러 개의 화면(페이지)으로 구성된다. Flutter는 화면 간의 이동을 관리하기 위한 강력한 내비게이션 시스템을 제공한다.

- **기본 내비게이션**: `Navigator.push()`를 사용하여 새로운 화면을 스택에 쌓고, `Navigator.pop()`을 사용하여 이전 화면으로 돌아가는 것이 가장 기본적인 방식이다.

- **이름 있는 라우트 (Named Routes)**: 앱이 복잡해지면, 각 화면에 고유한 이름을 부여하고 이 이름을 통해 이동하는 '이름 있는 라우트' 방식이 코드를 더 깔끔하게 관리하는 데 도움이 된다.135

  `MaterialApp`의 `routes` 속성에 라우트 맵을 정의하여 사용할 수 있다.

- **고급 라우팅**: 더 복잡한 내비게이션 시나리오(예: 딥 링킹, 중첩 라우팅)를 위해서는 `go_router`와 같은 공식적으로 지원되는 패키지를 사용하는 것이 좋다.


견고하고 안정적인 애플리케이션을 만들기 위해서는 테스트 코드 작성이 필수적이다. Flutter는 포괄적인 테스트 기능을 지원하며, 크게 세 가지 유형의 테스트로 나뉜다.15

1. **유닛 테스트 (Unit Tests)**: 단일 함수, 메서드 또는 클래스의 기능을 테스트한다. 외부 의존성 없이 비즈니스 로직이 올바르게 작동하는지 검증하는 데 사용된다.
2. **위젯 테스트 (Widget Tests)**: 단일 위젯이 예상대로 렌더링되고 사용자의 상호작용에 올바르게 반응하는지 테스트한다. Flutter의 위젯 테스트 프레임워크는 실제 기기나 에뮬레이터 없이 가상 환경에서 위젯을 빌드하고 테스트할 수 있게 해준다.
3. **통합 테스트 (Integration Tests)**: 전체 앱 또는 앱의 주요 부분들이 함께 올바르게 작동하는지 테스트한다. 실제 기기나 에뮬레이터에서 앱을 실행하며 사용자의 시나리오를 자동화하여 검증한다.


Flutter의 가장 큰 강점 중 하나는 활발한 생태계다. 지속적인 학습과 문제 해결을 위해 다음 리소스들을 적극적으로 활용해야 한다.

- **`pub.dev`**: Dart와 Flutter 패키지의 공식 저장소다. 상태 관리, 네트워킹, 데이터베이스, 애니메이션 등 거의 모든 기능에 대한 수많은 오픈소스 패키지를 찾을 수 있다.136 패키지를 선택할 때는 '좋아요' 수, 'Pub Points', 그리고 '인기도'와 같은 패키지 점수와 'Flutter Favorites' 배지를 참고하여 품질과 신뢰도를 판단하는 것이 좋다.136
- **공식 문서 (Official Documentation)**: Flutter의 모든 것을 담고 있는 가장 정확하고 신뢰할 수 있는 자료다. API 레퍼런스, 상세한 가이드, 그리고 특정 문제를 해결하는 방법을 보여주는 'Cookbook' 레시피 등을 제공한다.11
- **샘플 및 커뮤니티**:
  - **Flutter Samples Repository**: Flutter 팀이 직접 관리하는 모범 사례를 보여주는 다양한 샘플 앱을 참고할 수 있다.
  - **YouTube**: Flutter 공식 채널의 'Widget of the Week' 시리즈를 비롯하여 수많은 개발자들이 유용한 튜토리얼과 팁을 공유한다.136
  - **Stack Overflow**: 개발 중 발생하는 구체적인 문제에 대한 해답을 찾고 질문할 수 있는 가장 큰 커뮤니티다.136

Flutter는 빠르게 발전하고 있는 프레임워크다. 공식 블로그와 뉴스레터를 구독하여 최신 릴리스 정보, 새로운 기능, 그리고 로드맵을 꾸준히 확인하며 학습을 이어나가는 것이 훌륭한 Flutter 개발자로 성장하는 길이다.


1. Flutter: Advantages, Disadvantages and Future Scopes - GeeksforGeeks, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/flutter/flutter-advantages-disadvantages-and-future-scopes/
2. How to Install Flutter on Linux? - GeeksforGeeks, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/installation-guide/how-to-install-flutter-on-linux/
3. Flutter - Build apps for any screen, 8월 6, 2025에 액세스, https://flutter.dev/
4. Is Flutter Good for App Development: Pros and Cons - Purrweb, 8월 6, 2025에 액세스, https://www.purrweb.com/blog/is-flutter-good-pros-and-cons/
5. Flutter Pros and Cons – Why Use It in 2025? - Netguru, 8월 6, 2025에 액세스, https://www.netguru.com/blog/benefits-of-flutter
6. Flutter pros and cons: Is it a good choice for your app? - Cogniteq, 8월 6, 2025에 액세스, https://www.cogniteq.com/blog/flutter-pros-and-cons-it-good-choice-your-app
7. Why Use Flutter: Pros and Cons of Flutter App Development - Waverley, 8월 6, 2025에 액세스, https://waverleysoftware.com/blog/why-use-flutter-pros-and-cons/
8. Introduction to Flutter: Key Features and Benefits - MageComp, 8월 6, 2025에 액세스, https://magecomp.com/blog/flutter-key-features-benefits/
9. What are the features of Flutter? - Educative.io, 8월 6, 2025에 액세스, https://www.educative.io/answers/what-are-the-features-of-flutter
10. 8 Benefits and Advantages of Flutter for Cross-Platform Development - Relevant Software, 8월 6, 2025에 액세스, https://relevant.software/blog/top-8-flutter-advantages-and-why-you-should-try-flutter-on-your-next-project/
11. Docs | Flutter, 8월 6, 2025에 액세스, https://docs.flutter.dev/
12. Flutter Documentation - Flutter - Beautiful native apps in record time, 8월 6, 2025에 액세스, https://flutter-website-staging.firebaseapp.com/docs/
13. Widgets - Flutter Documentation, 8월 6, 2025에 액세스, https://docs.flutter.dev/get-started/fundamentals/widgets
14. Building user interfaces with Flutter, 8월 6, 2025에 액세스, https://docs.flutter.dev/ui
15. FAQ - Flutter Documentation, 8월 6, 2025에 액세스, https://docs.flutter.dev/resources/faq
16. Flutter Pros and Cons: Why Choose Flutter for App Development? - LeanCode, 8월 6, 2025에 액세스, https://leancode.co/blog/flutter-pros-and-cons-summary
17. Make Android apps | Flutter, 8월 6, 2025에 액세스, https://docs.flutter.dev/get-started/install/windows/mobile
18. How to Install Flutter on Windows? - GeeksforGeeks, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/installation-guide/how-to-install-flutter-on-windows/
19. Install manually - Flutter Documentation, 8월 6, 2025에 액세스, https://docs.flutter.dev/install/manual
20. Start building Flutter native desktop apps on Windows - Flutter Documentation, 8월 6, 2025에 액세스, https://docs.flutter.dev/get-started/install/windows/desktop
21. How to install flutter on Mac step by step - FlutterMapp.com, 8월 6, 2025에 액세스, https://www.fluttermapp.com/articles/install-flutter-mac
22. Get Started: Install on macOS - Flutter, 8월 6, 2025에 액세스, https://flutter-website-staging.firebaseapp.com/setup-macos/
23. macOS - Flutter Documentation, 8월 6, 2025에 액세스, https://docs.flutter.dev/get-started/install/macos
24. Make iOS apps | Flutter, 8월 6, 2025에 액세스, https://docs.flutter.dev/get-started/install/macos/mobile-ios
25. Start building Flutter Android apps on macOS - Flutter Documentation, 8월 6, 2025에 액세스, https://docs.flutter.dev/get-started/install/macos/mobile-android
26. A Step-by-Step Guide: Installing Flutter on macOS | by Balaji Venkatachalam | Medium, 8월 6, 2025에 액세스, https://medium.com/@vbalaji165/a-step-by-step-guide-installing-flutter-on-macos-acc1b338f36
27. Install Flutter on macOS Apple Silicon - DEV Community, 8월 6, 2025에 액세스, https://dev.to/rubyc/install-flutter-on-macos-apple-silicon-52b8
28. Install Flutter on Linux | Snap Store - Snapcraft, 8월 6, 2025에 액세스, https://snapcraft.io/flutter
29. Linux - Flutter Documentation, 8월 6, 2025에 액세스, https://docs.flutter.dev/get-started/install/linux
30. Make Linux desktop apps | Flutter, 8월 6, 2025에 액세스, https://docs.flutter.dev/get-started/install/linux/desktop
31. How to Install Flutter on Windows, Mac, and Linux - Apidog, 8월 6, 2025에 액세스, https://apidog.com/blog/how-to-installing-flutter-on-windows-mac-and-linux/
32. Start building Flutter Android apps on Linux - Flutter Documentation, 8월 6, 2025에 액세스, https://docs.flutter.dev/get-started/install/linux/android
33. Flutter Installation Guide for Linux - GitHub Gist, 8월 6, 2025에 액세스, https://gist.github.com/asad-albadi/19d681bd4d8bc59f18aea2efccdf1dfc
34. Introduction to Dart, 8월 6, 2025에 액세스, https://dart.dev/language
35. Dart (programming language) - Wikipedia, 8월 6, 2025에 액세스, https://en.wikipedia.org/wiki/Dart_(programming_language)
36. Dart Tutorial - GeeksforGeeks, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/dart/dart-tutorial/
37. Data Types in Dart - Dart Tutorial, 8월 6, 2025에 액세스, https://dart-tutorial.com/introduction-and-basics/datatypes-in-dart/
38. Dart Variables and Data Types, 8월 6, 2025에 액세스, https://dartbook.dev/article/Dart_Variables_and_Data_Types.html
39. Variables & Data Types - Dart | Flutter ~ Episode 1.1 - YouTube, 8월 6, 2025에 액세스, https://www.youtube.com/watch?v=sp4YSF9rpkI
40. Dart - Data Types - GeeksforGeeks, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/dart/dart-data-types/
41. Built-in types - Dart, 8월 6, 2025에 액세스, https://dart.dev/language/built-in-types
42. Intro to Dart - Flutter Documentation, 8월 6, 2025에 액세스, https://docs.flutter.dev/get-started/fundamentals/dart
43. Flow Control in Dart - Scaler Topics, 8월 6, 2025에 액세스, https://www.scaler.com/topics/flutter-tutorial/flow-control-in-dart/
44. Dart control flow statements. - Medium, 8월 6, 2025에 액세스, https://medium.com/@MrArc/dart-control-flow-statements-d2d6005604
45. Dart Control Flow Statement - Part 2 | by Sadanand Gadwal - Medium, 8월 6, 2025에 액세스, https://medium.com/@sadanandgadwal/dart-control-flow-from-if-else-to-loops-and-more-part-2-5350f5268cf7
46. Mastering Control Flow and Conditional Statements in Dart: A Beginner's Guide, 8월 6, 2025에 액세스, https://blog.suneeldk.me/mastering-control-flow-and-conditional-statements-in-dart-a-beginners-guide
47. Loops - Dart, 8월 6, 2025에 액세스, https://dart.dev/language/loops
48. Dart - Loop Control Statements (Break and Continue) - GeeksforGeeks, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/dart/dart-loop-control-statements-break-and-continue/
49. Functions - Dart, 8월 6, 2025에 액세스, https://dart.dev/language/functions
50. Exploring Dart Functions: A Comprehensive Guide with Coding Examples | by Ahsi Dev, 8월 6, 2025에 액세스, https://medium.com/@ahsan-001/exploring-dart-functions-a-comprehensive-guide-with-coding-examples-a2b836f36d85
51. Functions in Dart, 8월 6, 2025에 액세스, https://dart-tutorial.com/dart-functions/functions-in-dart/
52. Dart Programming - Functions - Tutorialspoint, 8월 6, 2025에 액세스, https://www.tutorialspoint.com/dart_programming/dart_programming_functions.htm
53. Different Types of Functions in Dart - GeeksforGeeks, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/dart/different-types-of-functions-in-dart/
54. Dart Class - Dart Tutorial, 8월 6, 2025에 액세스, https://www.darttutorial.org/dart-tutorial/dart-class/
55. Dart - Classes And Objects - GeeksforGeeks, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/dart/dart-classes-and-objects/
56. Classes | Dart, 8월 6, 2025에 액세스, https://dart.dev/language/classes
57. Why Await? Futures in Dart & Flutter - QuickBird Studios, 8월 6, 2025에 액세스, https://quickbirdstudios.com/blog/futures-flutter-dart/
58. Asynchronous programming: futures, async, await - Dart, 8월 6, 2025에 액세스, https://dart.dev/libraries/async/async-await
59. Understanding Future, Async, and Await in Dart: A Comprehensive Guide - Medium, 8월 6, 2025에 액세스, https://medium.com/@bhavesh.sachala/understanding-future-async-and-await-in-dart-a-comprehensive-guide-897f875133a7
60. Async programming - Dart, 8월 6, 2025에 액세스, https://dart.dev/language/async
61. Using await async in Dart - GeeksforGeeks, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/dart/using-await-async-in-dart/
62. Why use async/await instead of Future.then? | by Alexey Inkin | Flutter Senior - Medium, 8월 6, 2025에 액세스, https://medium.com/flutter-senior/why-use-async-await-instead-of-future-then-2e9c340aabfb
63. dart:async, 8월 6, 2025에 액세스, https://dart.dev/libraries/dart-async
64. When to use await vs .then( ) : r/dartlang - Reddit, 8월 6, 2025에 액세스, https://www.reddit.com/r/dartlang/comments/a4da0q/when_to_use_await_vs_then/
65. Difference between Stateful and Stateless widgets (Dart, Flutter) - Codecademy Forums, 8월 6, 2025에 액세스, https://discuss.codecademy.com/t/difference-between-stateful-and-stateless-widgets-dart-flutter/756398
66. Understanding Flutter: Stateful vs. Stateless Widgets. | by Blup - Medium, 8월 6, 2025에 액세스, https://medium.com/@blup-tool/understanding-flutter-stateful-vs-stateless-widgets-26fae8bbe8d6
67. Understanding Flutter Widgets: Stateless vs. Stateful Widgets | by khanoor - Medium, 8월 6, 2025에 액세스, https://medium.com/@khannooralam31/understanding-flutter-widgets-stateless-vs-stateful-widgets-cae56ea2dd56
68. Add interactivity to your Flutter app, 8월 6, 2025에 액세스, https://docs.flutter.dev/ui/interactivity
69. flutter - What difference between stateless and stateful widgets? - Stack Overflow, 8월 6, 2025에 액세스, https://stackoverflow.com/questions/45936084/what-difference-between-stateless-and-stateful-widgets
70. What is state in flutter? Stateless vs stateful widgets? : r/FlutterDev - Reddit, 8월 6, 2025에 액세스, https://www.reddit.com/r/FlutterDev/comments/vo7l6e/what_is_state_in_flutter_stateless_vs_stateful/
71. Mastering setState() in StatefulWidgets - Flutter blog, 8월 6, 2025에 액세스, https://finitefield.hashnode.dev/mastering-setstate-in-statefulwidgets
72. Flutter setState - The simplest state management in Flutter - themobilecoder, 8월 6, 2025에 액세스, https://themobilecoder.com/flutter-setstate-the-simplest-state-management-in-flutter/
73. Mastering setState() in Flutter: The Key to Responsive UIs | by Harsh Kumar Khatri | Medium, 8월 6, 2025에 액세스, https://mailharshkhatri.medium.com/mastering-setstate-in-flutter-the-key-to-responsive-uis-f4217d321b0b
74. www.geeksforgeeks.org, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/flutter/container-class-in-flutter/
75. Flutter Container Widget: A Complete Guide, 8월 6, 2025에 액세스, https://www.fluttersolution.com/2023/03/flutter-container-widget-complete-guide.html?m=1
76. Flutter Container Widget - DroidBiz, 8월 6, 2025에 액세스, https://www.droidbiz.in/flutter/flutter-container-widget
77. Text widgets - Flutter Documentation, 8월 6, 2025에 액세스, https://docs.flutter.dev/ui/widgets/text
78. Flutter | Text Widgets - MageComp, 8월 6, 2025에 액세스, https://magecomp.com/blog/flutter-text-widgets/
79. Darting Through the Basics: A Definitive Guide to the Flutter Text Widget - DhiWise, 8월 6, 2025에 액세스, https://www.dhiwise.com/post/a-definitive-guide-to-the-flutter-text-widget
80. All about Text widget in Flutter - Medium, 8월 6, 2025에 액세스, https://medium.com/@omlondhe/all-about-text-widget-in-flutter-45998e54e84
81. Flutter Text Widgets Documentation | by Prasish Sharma - Medium, 8월 6, 2025에 액세스, https://medium.com/@prasishsharma4/text-widgets-in-flutter-c8b912e6721f
82. Understanding constraints - Flutter Documentation, 8월 6, 2025에 액세스, https://docs.flutter.dev/ui/layout/constraints
83. Expanded Widget in Flutter: How to Use It Effectively | by Sanjay Singhania - Medium, 8월 6, 2025에 액세스, https://medium.com/flutter-writers/expanded-widget-in-flutter-how-to-use-it-effectively-747e95c8ee83
84. Flutter's Expanded vs. Flexible Widgets: Demystifying Their Usage | by Naveen Jose, 8월 6, 2025에 액세스, https://medium.com/@naveenjose24/flutters-expanded-vs-flexible-widgets-demystifying-their-usage-c1ff0712c725
85. Expanded Widget - Flutter - GeeksforGeeks, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/flutter/flutter-expanded-widget/
86. Expanded (Flutter Widget of the Week) - YouTube, 8월 6, 2025에 액세스, https://www.youtube.com/watch?v=_rnZaagadyo
87. Flutter Stack: A Simple Guide for Overlapping Widgets - DhiWise, 8월 6, 2025에 액세스, https://www.dhiwise.com/post/flutter-stack-your-ultimate-guide-to-overlapping-widgets
88. Flutter Journey 010 - Stack, Position, Sized Box, and Align Widgets in Flutter, 8월 6, 2025에 액세스, https://zubairmz.medium.com/flutter-journey-010-stack-position-sized-box-and-align-widgets-in-flutter-acde298120b
89. How to use the positioned widget in Flutter - Educative.io, 8월 6, 2025에 액세스, https://www.educative.io/answers/how-to-use-the-positioned-widget-in-flutter
90. Flutter - Positioned Widget - GeeksforGeeks, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/flutter/flutter-positioned-widget/
91. Position Widgets On Top Of Each Other in Flutter | by Ranjan Kumar | Medium, 8월 6, 2025에 액세스, https://medium.com/@rk0936626/position-widgets-on-top-of-each-other-in-flutter-aa24d4aadee3
92. Listview.builder in Flutter - GeeksforGeeks, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/flutter/listview-builder-in-flutter/
93. Listview.builder in Flutter - Techdynasty - Medium, 8월 6, 2025에 액세스, https://techdynasty.medium.com/listview-builder-in-flutter-e54a8fa2c7a0
94. How to Create a Custom Gridview In Flutter? - Scaler Topics, 8월 6, 2025에 액세스, https://www.scaler.com/topics/gridview-builder-flutter/
95. Using Flutter's GridView for Creating Responsive Layouts - Stackademic, 8월 6, 2025에 액세스, https://blog.stackademic.com/using-flutters-gridview-for-creating-responsive-layouts-88d7c333c56a
96. Create a grid list - Flutter Documentation, 8월 6, 2025에 액세스, https://docs.flutter.dev/cookbook/lists/grid-lists
97. GridViews in Flutter - Karthik Ponnam - Medium, 8월 6, 2025에 액세스, https://karthikponnam.medium.com/gridviews-in-flutter-cadbff12f47c
98. What is Widgets in Flutter? - GeeksforGeeks, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/flutter/what-is-widgets-in-flutter/
99. Flutter Widgets Cheatsheet: Categories, Types and Basic Widgets - DhiWise, 8월 6, 2025에 액세스, https://www.dhiwise.com/post/flutter-widget-cheatsheet-basic-widgets-types
100. Understanding Flutter Layout Widgets: A Beginner's Guide. | by Blup - Medium, 8월 6, 2025에 액세스, https://medium.com/@blup-tool/understanding-flutter-layout-widgets-a-beginners-guide-2b3a36c9e4f4
101. Card | FlutterFlow Documentation, 8월 6, 2025에 액세스, https://docs.flutterflow.io/resources/ui/widgets/built-in-widgets/card/
102. Complete Guide to Card Widget in Flutter: Creating Beautiful Cards with Advanced Features | by Hamidreza Ramezani | Medium, 8월 6, 2025에 액세스, https://medium.com/@hamidrezadeveloper/complete-guide-to-card-widget-in-flutter-creating-beautiful-cards-with-advanced-features-982fc1ac32ba
103. How To Build Card Widget In Flutter? - Scaler Topics, 8월 6, 2025에 액세스, https://www.scaler.com/topics/cards-in-flutter/
104. Flutter - Card Widget - GeeksforGeeks, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/flutter/flutter-card-widget/
105. Flutter | Card Widget - MageComp, 8월 6, 2025에 액세스, https://magecomp.com/blog/flutter-card-widget/
106. State Management in Flutter: A Complete Guide | by Navaghan Dabhi | Medium, 8월 6, 2025에 액세스, https://medium.com/@dabhinavaghan1/state-management-in-flutter-a-complete-guide-6497b9bd3b7e
107. State management - Flutter Documentation, 8월 6, 2025에 액세스, https://docs.flutter.dev/get-started/fundamentals/state-management
108. Essential Guide on Flutter State Management – Best Practices - SolGuruz, 8월 6, 2025에 액세스, https://solguruz.com/blog/essential-guide-on-flutter-state-management/
109. A Complete Guide To Flutter State Management - Coditation, 8월 6, 2025에 액세스, https://www.coditation.com/blog/a-complete-guide-to-flutter-state-management
110. Provider Package - Flutter - GeeksforGeeks, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/flutter/flutter-provider-package/
111. Simple app state management - Flutter Documentation, 8월 6, 2025에 액세스, https://docs.flutter.dev/data-and-backend/state-mgmt/simple
112. Understanding Flutter's Provider Package For State Management: A Practical Guide with a Sample Project - Medium, 8월 6, 2025에 액세스, https://medium.com/@accnd163/understanding-flutters-provider-package-for-state-management-a-practical-guide-with-a-sample-e5134321b685
113. provider | Flutter package - Pub.dev, 8월 6, 2025에 액세스, https://pub.dev/packages/provider
114. How to Manage State in Flutter with BLoC Pattern? - GeeksforGeeks, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/flutter/how-to-manage-state-in-flutter-with-bloc-pattern/
115. Complete Flutter BLoC Tutorial: Understanding State Management in Flutter - DhiWise, 8월 6, 2025에 액세스, https://www.dhiwise.com/post/flutter-bloc-tutorial-understanding-state-management
116. Ultimate Flutter BLoC Guide: State Management Best Practices, 8월 6, 2025에 액세스, https://www.oneclickitsolution.com/blog/flutter-bloc-state-management-best-practices
117. Bloc vs. RiverPod: A Comprehensive Comparison in 2024, 8월 6, 2025에 액세스, https://www.xavor.com/blog/bloc-vs-riverpod/
118. Mastering State Management in Flutter with Bloc: A Comprehensive Guide - Medium, 8월 6, 2025에 액세스, https://medium.com/@dihsar/mastering-state-management-in-flutter-with-bloc-a-comprehensive-guide-1d03319ba7df
119. Fetch data from the internet - Flutter Documentation, 8월 6, 2025에 액세스, https://docs.flutter.dev/cookbook/networking/fetch-data
120. Flutter HTTP Tutorial - A Complete Guide - CodeSundar, 8월 6, 2025에 액세스, https://codesundar.com/flutter-http-tutorial/
121. Send data to the internet - Flutter Documentation, 8월 6, 2025에 액세스, https://docs.flutter.dev/cookbook/networking/send-data
122. Flutter and REST APIs: How to Manage HTTP Requests Efficiently | by Blup | Medium, 8월 6, 2025에 액세스, https://medium.com/@blup-tool/flutter-and-rest-apis-how-to-manage-http-requests-efficiently-6ebcc9d025bd
123. dart:convert | Dart, 8월 6, 2025에 액세스, https://dart.dev/libraries/dart-convert
124. JSON and serialization - Flutter Documentation, 8월 6, 2025에 액세스, https://docs.flutter.dev/data-and-backend/serialization/json
125. The best way to parse a JSON in Dart - Stack Overflow, 8월 6, 2025에 액세스, https://stackoverflow.com/questions/15866290/the-best-way-to-parse-a-json-in-dart
126. Parse JSON in the background - Flutter Documentation, 8월 6, 2025에 액세스, https://docs.flutter.dev/cookbook/networking/background-parsing
127. Learn State Management in Flutter by Building a Simple Todo App - freeCodeCamp, 8월 6, 2025에 액세스, https://www.freecodecamp.org/news/learn-state-management-in-flutter/
128. kuldeep2code/todo-app-getx-flutter: Create a todo application using getx micro-framework of flutter - GitHub, 8월 6, 2025에 액세스, https://github.com/kuldeep2code/todo-app-getx-flutter
129. navinAce/TODO_Flutter: TODO app developed with Flutter as a learning project. Includes source code for a task management app, demonstrating Flutter's fundamental concepts and widgets. To access the app, click the link below. You may receive a warning about unsafe content; press "Open" to proceed and then "Install". - GitHub, 8월 6, 2025에 액세스, https://github.com/navinAce/TODO_Flutter
130. Flutter Todo app step-by-step - Medium, 8월 6, 2025에 액세스, https://medium.com/@amd4iq/flutter-todo-app-step-by-step-1011e17f1895
131. Day 21: Mastering Provider State Management in Flutter | by Hemant Kumar Prajapati, 8월 6, 2025에 액세스, https://medium.com/@hemantkumarceo001/day-21-mastering-provider-state-management-in-flutter-0defd61f5d9c
132. Implementing Rest API in Flutter - GeeksforGeeks, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/flutter/implementing-rest-api-in-flutter/
133. Flutter Provider Http Get Request | Restful Api Example - Dbestech, 8월 6, 2025에 액세스, https://www.dbestech.com/tutorials/flutter-provider-http-get-request-restful-api-example
134. Rest API Call with http and Provider in Flutter - YouTube, 8월 6, 2025에 액세스, https://www.youtube.com/watch?v=vM1Rv2PuEqk
135. Flutter Routes & Navigation – Parameters, Named Routes, onGenerateRoute - YouTube, 8월 6, 2025에 액세스, https://www.youtube.com/watch?v=nyvwx7o277U
136. Learn - Flutter, 8월 6, 2025에 액세스, https://flutter.dev/learn
137. How to use Flutter's official documentation effectively for learning and reference, 8월 6, 2025에 액세스, https://dev.to/shadrachsamuel04/how-to-use-flutters-official-documentation-effectively-for-learning-and-reference-3dpc

