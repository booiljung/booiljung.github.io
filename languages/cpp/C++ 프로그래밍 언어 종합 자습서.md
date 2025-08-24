[C++ 프로그래밍](./index.md)
# C++ 프로그래밍 언어 종합 자습서


C++는 운영체제, 게임 엔진, 금융 시스템, 고성능 컴퓨팅 등 성능이 시스템의 가치를 결정하는 거의 모든 분야에서 핵심적인 역할을 수행하는 프로그래밍 언어이다.1 C++ 학습은 단순히 하나의 언어를 배우는 것을 넘어, 컴퓨터 시스템의 근본적인 동작 원리와 현대 소프트웨어 공학의 정수를 이해하는 과정이다.

C++의 수십 년에 걸친 생명력과 영향력의 본질은 단순히 '빠른 속도'라는 단어로만 설명되지 않는다. 그 핵심은 C언어로부터 물려받은 하드웨어에 대한 직접적이고 정밀한 '통제력'을 유지하면서, 시대의 요구에 맞춰 객체 지향, 제네릭, 함수형 프로그래밍과 같은 고수준의 추상화 패러다임을 성공적으로 흡수해 온 '진화하는 능력'에 있다.2 1985년 처음 등장한 이래로 , C++는 C++11, C++14, C++17, C++20 등 지속적인 표준 개정을 통해 끊임없이 발전하며 현대 소프트웨어의 복잡한 요구사항을 해결하는 강력한 도구로 자리매김했다.2

따라서 C++를 배운다는 것은 과거의 유물을 탐구하는 것이 아니라, 저수준의 메모리 관리부터 고수준의 설계 패턴까지 아우르는 폭넓은 시야를 갖추는 것이다. 이 자습서는 C++의 문법적 지식을 넘어, 그 안에 담긴 설계 철학과 원리를 깊이 있게 전달함으로써 학습자가 C++를 통해 강력하고 효율적이며 우아한 소프트웨어를 구축할 수 있는 능력을 갖추도록 돕는 것을 목표로 한다.



C++의 역사를 이해하는 것은 언어의 설계 목표와 핵심 철학을 파악하는 첫걸음이다. C++는 1979년, 덴마크의 컴퓨터 과학자 비야네 스트롭스트룹(Bjarne Stroustrup)이 AT&T 벨 연구소에서 'C with Classes'라는 이름의 프로젝트로 시작했다. 당시 C언어는 시스템 프로그래밍에 있어 최고의 효율성을 자랑했지만, 소프트웨어의 규모가 커짐에 따라 코드 관리의 어려움이 대두되었다. 스트롭스트룹의 목표는 C언어의 강력한 성능과 저수준 제어 능력을 그대로 유지하면서, 대규모 소프트웨어 개발을 용이하게 하는 객체 지향 프로그래밍(Object-Oriented Programming, OOP) 개념을 도입하는 것이었다.4

이러한 배경 속에서 C++는 1985년에 공식적으로 발표되었으며 2, 다음과 같은 핵심 설계 목표를 가지고 발전해왔다.

- **C언어와의 호환성:** 기존의 방대한 C언어 코드 자산을 활용할 수 있도록 높은 수준의 호환성을 유지한다.4 비록 C99 표준 이후 C++이 C의 완전한 상위 집합(superset)은 아니게 되었지만, 여전히 대부분의 C 코드는 C++ 컴파일러에서 성공적으로 컴파일된다.2
- **객체 지향:** 캡슐화, 상속, 다형성과 같은 객체 지향 개념을 도입하여 코드의 재사용성을 높이고 대규모 소프트웨어의 유지보수를 용이하게 한다.4
- **엄격한 타입 체크:** 컴파일 시점에 타입 오류를 잡아내어 실행 시간(runtime)에 발생할 수 있는 오류의 가능성을 줄이고 디버깅을 돕는다.4
- **효율성 저하 최소화:** 새로운 기능(예: 가상 함수, 템플릿)을 추가하더라도 C언어 수준의 성능을 유지하기 위해 불필요한 오버헤드를 최소화한다. "사용하지 않는 기능에 대해서는 비용을 지불하지 않는다(You don't pay for what you don't use)"는 것이 C++의 핵심 철학 중 하나이다.4

이러한 철학을 바탕으로 C++는 1998년 첫 공식 국제 표준인 C++98을 시작으로 C++03, C++11, C++14, C++17, 그리고 최신 표준인 C++20에 이르기까지 주기적인 개정을 통해 현대 프로그래밍 환경의 요구를 지속적으로 반영하고 있다.2


C++ 프로그래밍을 시작하기 위해서는 코드를 작성할 편집기(Editor)와 작성된 코드를 기계어로 번역할 컴파일러(Compiler), 그리고 오류를 잡기 위한 디버거(Debugger)가 필요하다. 개발 환경을 구축하는 방법은 크게 두 가지로 나눌 수 있다.

개발 환경을 선택하는 것은 단순히 도구를 고르는 것을 넘어, 개발 워크플로우에 대한 두 가지 다른 철학 중 하나를 선택하는 것을 의미한다. Visual Studio와 같은 통합 개발 환경(IDE)은 디버거, 컴파일러, 프로젝트 관리 시스템이 긴밀하게 통합된 '올인원(All-in-One)' 솔루션으로, 특히 대규모 Windows 기반 프로젝트에서 생산성을 극대화한다.11 반면, VS Code와 MinGW를 조합하는 방식은 사용자가 컴파일러, 링커, 디버거 등 각 요소를 직접 구성하는 '모듈화된' 접근법으로, 경량성과 플랫폼 독립성을 중시하는 현대 개발 트렌드를 반영한다.14 초심자에게 후자의 방식은 빌드 프로세스에 대한 더 깊은 이해를 강제하는 효과적인 학습 도구가 될 수 있다.


Visual Studio는 Microsoft가 개발한 강력한 통합 개발 환경(IDE)으로, C++ 개발에 필요한 모든 도구를 한 번에 설치하고 관리할 수 있어 편리하다. 개인 개발자, 학생, 오픈소스 기여자는 무료 버전인 Community Edition을 사용할 수 있다.11

1. **설치 관리자 다운로드:**([https://visualstudio.microsoft.com/ko/)에](https://visualstudio.microsoft.com/ko/)에) 접속하여 'Visual Studio Community' 버전의 설치 관리자를 다운로드한다.17
2. **워크로드 선택:** 설치 관리자를 실행한 후 '워크로드' 탭에서 **"C++를 사용한 데스크톱 개발"**을 선택한다. 이 워크로드는 C++ 컴파일러, 표준 라이브러리, 디버거 등 필수적인 도구들을 포함한다.17
3. **설치:** '설치' 버튼을 클릭하여 설치를 진행한다. 설치가 완료되면 C++ 프로젝트를 생성하고 바로 코딩을 시작할 수 있다.


Visual Studio Code(VS Code)는 가볍고 확장성이 뛰어난 코드 편집기로, C++ 컴파일러를 별도로 설치하고 연동하여 개발 환경을 구축할 수 있다. Windows 환경에서는 MinGW를 통해 GCC(GNU Compiler Collection) 컴파일러를 사용할 수 있다.

1. **MinGW-w64 설치 (Windows):**
   - MinGW는 Windows에서 GCC 컴파일러, 링커, 디버거(GDB) 등을 사용할 수 있게 해주는 툴체인이다.(https://www.mingw-w64.org/downloads/) 등에서 자신의 운영체제(64비트 권장)에 맞는 버전을 다운로드한다.14
   - 다운로드한 파일의 압축을 해제하고, `mingw64` 폴더를 `C:\`와 같이 경로에 한글이나 공백이 없는 곳으로 이동시킨다.14
2. **환경 변수 설정 (Windows):**
   - 컴퓨터의 어느 위치에서든 컴파일러를 실행할 수 있도록 환경 변수를 설정해야 한다.
   - `C:\mingw64\bin`과 같이 `g++.exe` 파일이 위치한 경로를 복사한다.16
   - '시스템 속성' -> '고급' -> '환경 변수'로 이동하여 '시스템 변수' 목록의 `Path`를 편집한다.14
   - '새로 만들기'를 클릭하여 복사한 경로를 추가하고 모든 창에서 '확인'을 눌러 저장한다.
   - 명령 프롬프트(cmd)를 열고 `g++ --version`을 입력했을 때 버전 정보가 출력되면 성공적으로 설정된 것이다.16
3. **Visual Studio Code 설치 및 확장:**
   - ([https://code.visualstudio.com/)에서](https://code.visualstudio.com/)에서) 설치 파일을 다운로드하여 설치한다.14
   - VS Code를 실행하고, 왼쪽 사이드바의 '확장(Extensions)' 탭으로 이동한다.
   - `C/C++ Extension Pack`을 검색하여 설치한다. 이 확장팩은 코드 자동 완성(IntelliSense), 디버깅, 코드 탐색 등 필수적인 기능을 제공한다.14
4. **프로젝트 구성 (빌드 및 디버깅 설정):**
   - 작업할 폴더를 하나 생성하고 VS Code에서 해당 폴더를 연다.
   - C++ 소스 파일(예: `main.cpp`)을 생성한다.
   - VS Code는 빌드와 디버깅을 위해 `.vscode` 폴더 안에 JSON 형식의 설정 파일을 사용한다.
     - `c_cpp_properties.json`: 컴파일러 경로와 IntelliSense 설정을 담당한다.
     - `tasks.json`: 빌드 명령(컴파일)을 정의한다.
     - `launch.json`: 디버거 실행 방법을 정의한다.
   - VS Code의 메뉴에서 '터미널' -> '기본 빌드 작업 구성'을 선택하고 `C/C++: g++.exe 활성 파일 빌드`를 선택하면 `tasks.json` 파일이 자동으로 생성된다. 디버깅을 처음 시작(F5 키)할 때 `launch.json` 파일도 유사한 방식으로 생성할 수 있다.14


개발 환경 구축이 완료되었다면, 가장 기본적인 C++ 프로그램을 작성하고 실행하여 모든 것이 정상적으로 동작하는지 확인한다.

```C++
#include <iostream>

int main() 
{
    std::cout << "Hello, World!" << std::endl;
    return 0;
}
```

이 간단한 코드는 C++ 프로그램의 핵심적인 구조를 보여준다. 각 구성 요소를 자세히 살펴보자.

- `#include <iostream>`: **전처리기 지시문(Preprocessor Directive)**이다. 컴파일이 시작되기 전에 `#include`로 지정된 파일을 소스 코드에 포함시키도록 전처리기에 지시한다. `<iostream>`은 C++ 표준 라이브러리의 일부로, 콘솔 입출력(Input/Output Stream)에 필요한 기능들을 담고 있다.18
- `int main()`: 모든 C++ 프로그램의 **시작점(Entry Point)**이 되는 특별한 함수이다. 운영체제는 프로그램을 실행할 때 `main` 함수를 가장 먼저 호출한다. `int`는 이 함수가 정수(integer) 값을 반환함을 의미한다.19
- `{... }`: 중괄호는 함수의 시작과 끝을 나타내는 **코드 블록(Code Block)**이다.
- `std::cout`: **표준 출력 스트림(Standard Output Stream)** 객체이다. 기본적으로 콘솔 화면에 연결되어 있다. `std::`는 `cout`이 `std`라는 **이름 공간(namespace)**에 속해 있음을 명시하는 것이다.18 이름 공간은 이름 충돌을 방지하기 위해 관련 기능들을 그룹화하는 역할을 한다.
- `<<`: **스트림 삽입 연산자(Stream Insertion Operator)**이다. 오른쪽에 있는 데이터를 왼쪽의 스트림 객체(여기서는 `std::cout`)로 보낸다(삽입한다).22
- `"Hello, World!"`: 출력할 **문자열 리터럴(String Literal)**이다.
- `std::endl`: **End Line**의 약자로, 출력 스트림에 개행 문자(`\n`)를 삽입하고 출력 버퍼를 비우는(flush) 역할을 하는 **조작자(Manipulator)**이다.22
- `return 0;`: `main` 함수의 실행을 종료하고 운영체제에 값을 반환한다. 관례적으로 `0`은 프로그램이 성공적으로 종료되었음을 의미한다.19
- `;`: 세미콜론은 C++에서 하나의 **문장(Statement)**이 끝났음을 알리는 기호이다.

이 코드를 컴파일하고 실행하면 콘솔 화면에 "Hello, World!"가 출력될 것이다.



프로그래밍에서 데이터는 변수(variable)라는 이름이 붙은 메모리 공간에 저장된다.24 C++는 정적 타입 언어(statically-typed language)이므로, 변수를 사용하기 전에 반드시 어떤 종류의 데이터를 저장할 것인지 타입을 명시하여 선언해야 한다.


변수는 `타입 변수명;` 형식으로 선언한다. 선언과 동시에 값을 할당하는 것을 초기화(initialization)라고 한다.26

```C++
int age;          // 정수형 변수 age 선언
age = 30;         // 값 할당

double weight = 75.5; // 실수형 변수 weight 선언과 동시에 초기화

// C++11 이후 권장되는 유니폼 초기화(Uniform Initialization)
int score {100}; 
```

변수의 이름을 정할 때는 몇 가지 규칙이 있다. 이름은 영문자, 숫자, 밑줄(`_`)로 구성할 수 있지만, 숫자로 시작할 수 없으며 C++ 키워드(예: `int`, `if`, `class`)는 사용할 수 없다.24


C++는 다양한 종류의 데이터를 표현하기 위해 여러 내장 데이터 타입을 제공한다. 이들의 크기와 표현 범위는 시스템 아키텍처나 컴파일러에 따라 다를 수 있으나, 일반적으로 통용되는 기준은 다음과 같다.25

| 데이터 타입     | 크기 (Bytes) | 값의 범위                      | 설명                                      |
| --------------- | ------------ | ------------------------------ | ----------------------------------------- |
| `bool`          | 1            | `true`, `false`                | 논리적 참과 거짓을 표현한다.28            |
| `char`          | 1            | -128 ~ 127                     | 단일 문자를 저장한다 (ASCII 코드).27      |
| `unsigned char` | 1            | 0 ~ 255                        | 부호 없는 문자를 표현한다.                |
| `short`         | 2            | -32,768 ~ 32,767               | 작은 범위의 정수를 저장한다.              |
| `int`           | 4            | -2,147,483,648 ~ 2,147,483,647 | 가장 일반적으로 사용되는 정수 타입이다.28 |
| `unsigned int`  | 4            | 0 ~ 4,294,967,295              | 0과 양의 정수만을 표현한다.26             |
| `long long`     | 8            | 약 ±9.22e18                    | 매우 큰 범위의 정수를 저장한다.27         |
| `float`         | 4            | 약 ±3.4e±38 (유효숫자 7자리)   | 단정밀도 부동소수점 수를 저장한다.26      |
| `double`        | 8            | 약 ±1.7e±308 (유효숫자 15자리) | 배정밀도 부동소수점 수를 저장한다.26      |

`sizeof` 연산자를 사용하면 특정 데이터 타입이나 변수가 메모리에서 차지하는 크기를 바이트 단위로 확인할 수 있다.29


하나의 데이터 타입을 다른 타입으로 변환하는 것을 형 변환이라고 한다.

- **암시적 형 변환 (Implicit Conversion):** 컴파일러가 필요에 따라 자동으로 수행하는 변환이다. 예를 들어, `double d = 5;`라는 코드에서 정수 `5`는 `double` 타입인 `5.0`으로 자동 변환된다. 이 과정에서 데이터 손실이 발생할 수 있으므로 주의해야 한다 (예: `int i = 3.14;` -> `i`는 `3`이 됨).24
- **명시적 형 변환 (Explicit Conversion):** 프로그래머가 의도를 명확히 하여 강제로 타입을 변환하는 것이다. C++에서는 C-스타일 캐스팅보다 타입에 안전한 `static_cast`를 사용하는 것이 권장된다.29

```C++
double pi = 3.14159;
int integer_pi = static_cast<int>(pi); // 명시적 형 변환, integer_pi는 3이 된다.
```


연산자는 변수나 값에 대한 연산을 수행하는 기호이다. C++는 풍부한 연산자를 제공한다.

- **산술 연산자:** `+` (덧셈), `-` (뺄셈), `*` (곱셈), `/` (나눗셈), `%` (나머지).24
- **관계 연산자:** `==` (같음), `!=` (다름), `>` (큼), `<` (작음), `>=` (크거나 같음), `<=` (작거나 같음). 연산 결과는 `bool` 타입(`true` 또는 `false`)이다.30
- **논리 연산자:** `&&` (논리 AND), `||` (논리 OR), `!` (논리 NOT). `bool` 값을 피연산자로 사용한다.30
- **대입 연산자:** `=` (대입). `a += 5;` (`a = a + 5;`와 동일)와 같은 복합 대입 연산자도 있다.24
- **증감 연산자:** `++` (1 증가), `--` (1 감소). 변수 앞에 붙으면 **전위(prefix)**, 뒤에 붙으면 **후위(postfix)** 연산자가 된다. 전위는 값을 먼저 증감시킨 후 연산을 수행하고, 후위는 연산을 먼저 수행한 후 값을 증감시킨다.24
- **비트 연산자:** `&`, `|`, `^`, `~`, `<<`, `>>`. 데이터를 비트 수준에서 조작할 때 사용한다.
- **조건 연산자 (삼항 연산자):** `조건? 값1 : 값2`. 조건이 참이면 `값1`을, 거짓이면 `값2`를 반환한다.31

여러 연산자가 하나의 수식에 사용될 때는 **연산자 우선순위**와 **결합 규칙**에 따라 계산 순서가 결정된다. 예를 들어, 곱셈과 나눗셈은 덧셈과 뺄셈보다 우선순위가 높다. 복잡한 수식에서는 괄호 `()`를 사용하여 계산 순서를 명확히 지정하는 것이 코드의 가독성과 안정성을 위해 바람직하다.24


프로그램이 사용자와 상호작용하기 위한 가장 기본적인 방법은 표준 입출력 스트림을 사용하는 것이다.

- **`#include <iostream>`:** 표준 입출력 기능을 사용하기 위해 반드시 포함해야 하는 헤더 파일이다.18

- 표준 출력 std::cout:

  std::cout 객체와 스트림 삽입 연산자 <<를 사용하여 콘솔에 데이터를 출력한다. << 연산자는 연쇄적으로 사용하여 여러 데이터를 한 번에 출력할 수 있다.18 C언어의 `printf`와 달리 출력할 데이터의 타입에 따라 서식 지정자(`%d`, `%f` 등)를 바꿀 필요가 없어 편리하고 타입에 안전하다.18

  ```C++
  int age = 30;
  std::string name = "Bjarne";
  std::cout << "Name: " << name << ", Age: " << age << std::endl;
  ```

- 표준 입력 std::cin:

  std::cin 객체와 스트림 추출 연산자 >>를 사용하여 키보드로부터 입력을 받는다. >> 연산자는 기본적으로 공백(스페이스, 탭, 개행)을 구분자로 인식하여 데이터를 분리한다.18

  ```C++
  int number;
  std::cout << "Enter a number: ";
  std::cin >> number;
  std::cout << "You entered: " << number << std::endl;
  ```

- 이름 공간(Namespace)과 using 지시문:

  cout, cin, endl 등은 모두 std라는 이름 공간에 속해 있어, 사용할 때마다 std:: 접두사를 붙여야 한다. using namespace std; 지시문을 소스 코드 상단에 선언하면 이 접두사를 생략할 수 있다.18 하지만 이 방식은 대규모 프로젝트에서 이름 충돌을 일으킬 수 있으므로, `using std::cout;`, `using std::cin;`과 같이 사용할 기능만 개별적으로 선언하거나, 함수 내에서만 `using` 지시문을 사용하는 것이 더 안전한 방법으로 권장된다.35


제어 흐름 구문은 프로그램의 실행 경로를 조건에 따라 변경하거나 특정 코드 블록을 반복 실행하게 한다.


- if, else if, else:

  가장 기본적인 조건문으로, if 뒤의 조건식이 true이면 해당 코드 블록을 실행한다. if 조건이 false일 경우, 이어지는 else if의 조건을 검사하거나, 모든 조건이 false일 경우 else 블록을 실행한다.37

  ```C++
  int score = 85;
  if (score >= 90) {
      std::cout << "A" << std::endl;
  } else if (score >= 80) {
      std::cout << "B" << std::endl; // 이 블록이 실행된다.
  } else {
      std::cout << "C" << std::endl;
  }
  ```

  중첩된 `if` 문을 사용할 때 `else`는 가장 가까운 `if`에 연결되므로, 의도치 않은 동작을 피하기 위해 중괄호 `{}`를 명확하게 사용하는 습관이 중요하다.38

- switch-case:

  하나의 변수 값을 여러 상수 값과 비교하여 실행 경로를 결정할 때 유용하다. 비교 대상은 정수, 문자, 열거형(enum) 타입이어야 한다.39

  - `case`: 비교할 상수 값을 지정한다.
  - `break`: 해당 `case`의 실행을 마치고 `switch` 문을 빠져나간다. `break`가 없으면 다음 `case`의 코드가 연이어 실행되는 'fall-through' 현상이 발생하므로 주의해야 한다.37
  - `default`: 어떤 `case`와도 일치하지 않을 때 실행된다.39

  ```C++
  int choice = 2;
  switch (choice) {
      case 1:
          std::cout << "Menu 1 selected." << std::endl;
          break;
      case 2:
          std::cout << "Menu 2 selected." << std::endl; // 이 블록이 실행된다.
          break;
      default:
          std::cout << "Invalid choice." << std::endl;
          break;
  }
  ```

  조건의 수가 많을 경우, `switch`문은 내부적으로 점프 테이블을 사용하여 `if-else` 체인보다 더 효율적으로 동작할 수 있다.39


- for 루프:

  for (초기식; 조건식; 증감식)의 구조를 가지며, 주로 정해진 횟수만큼 코드를 반복할 때 사용된다. 초기식은 루프 시작 전 한 번만 실행되고, 매 반복마다 조건식을 검사하며, 반복이 끝날 때마다 증감식이 실행된다.43

  ```C++
  int sum = 0;
  for (int i = 1; i <= 10; ++i) {
      sum += i; // 1부터 10까지의 합을 계산한다.
  }
  ```

- 범위 기반 for 루프 (Range-based for loop):

  C++11부터 도입된 문법으로, 배열이나 STL 컨테이너의 모든 원소를 매우 간결하게 순회할 수 있다. for (원소타입 변수 : 컨테이너) 형태로 사용한다.46

  ```C++
  int numbers = {1, 2, 3, 4, 5};
  for (int num : numbers) {
      std::cout << num << " ";
  }
  ```

- while 루프:

  while (조건식)의 구조를 가지며, 조건식이 true인 동안 코드 블록을 계속해서 반복한다. 반복 횟수가 명확하지 않고 특정 조건이 만족될 때까지 반복해야 할 경우에 유용하다.43

  ```C++
  int n = 5;
  while (n > 0) {
      std::cout << n << " ";
      --n;
  } // 출력: 5 4 3 2 1
  ```

- do-while 루프:

  while 루프와 유사하지만, 조건을 나중에 검사하므로 코드 블록이 최소 한 번은 반드시 실행된다. do {... } while (조건식); 형태로 사용하며, while 뒤에 세미콜론이 붙는다는 점에 유의해야 한다.43

- **`break`와 `continue`:**

  - `break`: 반복문이나 `switch` 문을 즉시 탈출한다.
  - `continue`: 현재 반복의 나머지 부분을 건너뛰고, 다음 반복을 시작한다.



함수(Function)는 특정 작업을 수행하는 코드의 집합을 하나의 단위로 묶은 것이다. 함수를 사용하면 코드의 중복을 피하고, 프로그램을 논리적인 단위로 분해하여 재사용성과 유지보수성을 높일 수 있다.21


C++ 함수는 네 가지 주요 요소로 구성된다.21

1. **반환 타입 (Return Type):** 함수가 작업을 마친 후 호출한 곳으로 반환하는 값의 데이터 타입. 값을 반환하지 않는 경우 `void`를 사용한다.
2. **함수 이름 (Function Name):** 함수를 식별하는 고유한 이름.
3. **매개변수 목록 (Parameter List):** 함수가 작업을 수행하는 데 필요한 외부 데이터를 전달받는 통로. 괄호 `()` 안에 `타입 변수명` 형태로 선언한다.
4. **함수 몸체 (Function Body):** 중괄호 `{}` 안에 함수의 실제 동작을 정의하는 코드 블록.

```C++
// 함수의 정의(Definition)
int add(int a, int b) // 반환 타입: int, 함수 이름: add, 매개변수: int a, int b
{
    // 함수 몸체
    return a + b; // return 문을 통해 결과값 반환
}
```


- **함수 선언 (Declaration) 또는 프로토타입 (Prototype):** 컴파일러에게 함수의 이름, 반환 타입, 매개변수 목록을 미리 알려주는 것이다. `main` 함수보다 뒤에 함수를 정의하려면, `main` 함수 앞에 반드시 선언이 있어야 한다. 이는 컴파일러가 `main` 함수에서 해당 함수를 호출하는 코드를 만났을 때, 그 함수가 어떤 형태인지 알 수 있게 하기 위함이다.49

  ```C++
  #include <iostream>
  
  int add(int a, int b); // 함수 선언 (프로토타입)
  
  int main() {
      int result = add(5, 3); // 함수 호출
      std::cout << "Result: " << result << std::endl;
      return 0;
  }
  
  // 함수 정의
  int add(int a, int b) {
      return a + b;
  }
  ```

- **함수 정의 (Definition):** 함수의 실제 코드를 구현하는 부분이다.49


함수에 인자(argument)를 전달하는 방식은 크게 세 가지로 나눌 수 있으며, 이는 함수가 원본 데이터를 어떻게 다룰지를 결정하는 중요한 개념이다.

- 값에 의한 호출 (Call-by-Value):

  가장 기본적인 방식으로, 함수를 호출할 때 인자의 값이 복사되어 함수의 매개변수에 전달된다. 따라서 함수 내에서 매개변수의 값을 변경하더라도, 함수 호출에 사용된 원본 변수에는 아무런 영향을 미치지 않는다.51 이는 데이터의 안정성을 보장하지만, 구조체나 클래스처럼 크기가 큰 객체를 전달할 경우 복사로 인한 성능 저하가 발생할 수 있다.

  ```C++
  void increment(int x) {
      x++; // x의 복사본이 증가. 원본은 그대로.
  }
  int num = 5;
  increment(num); // num은 여전히 5
  ```

- 주소에 의한 호출 (Call-by-Address / Call-by-Pointer):

  인자의 메모리 주소를 포인터(pointer) 매개변수로 전달하는 방식이다. 함수는 전달받은 주소를 통해 원본 변수의 메모리 공간에 직접 접근할 수 있다. 따라서 역참조 연산자(*)를 사용하여 원본 변수의 값을 읽거나 수정하는 것이 가능하다.51

  ```C++
  void increment(int* p) {
      (*p)++; // p가 가리키는 주소의 값을 증가.
  }
  int num = 5;
  increment(&num); // num의 주소를 전달. num은 6이 된다.
  ```

- 참조에 의한 호출 (Call-by-Reference):

  C++에서 추가된 방식으로, 인자의 별명(alias) 역할을 하는 참조자(reference)를 매개변수로 사용한다. 함수 내에서 참조자 매개변수를 조작하면 원본 변수가 직접 변경된다. 포인터와 달리 주소 연산자(&)나 역참조 연산자(*) 없이 일반 변수처럼 사용할 수 있어 코드가 더 간결하고 직관적이다.52

  ```C++
  void increment(int& r) {
      r++; // r이 참조하는 원본 변수의 값을 증가.
  }
  int num = 5;
  increment(num); // num을 참조로 전달. num은 6이 된다.
  ```


함수 오버로딩은 C++의 다형성(polymorphism)을 구현하는 방법 중 하나로, **같은 이름의 함수를 여러 개 정의**할 수 있게 해주는 기능이다. 컴파일러는 함수의 이름뿐만 아니라 **매개변수의 개수나 타입**을 종합적으로 고려하여 호출할 함수를 결정한다.54

```C++
#include <iostream>

void print(int i) {
    std::cout << "Integer: " << i << std::endl;
}

void print(double d) {
    std::cout << "Double: " << d << std::endl;
}

void print(const char* s) {
    std::cout << "String: " << s << std::endl;
}

int main() {
    print(10);       // print(int) 호출
    print(3.14);     // print(double) 호출
    print("Hello");  // print(const char*) 호출
    return 0;
}
```

**주의사항:**

- 함수 오버로딩은 오직 **매개변수의 시그니처(signature)**, 즉 매개변수의 개수와 타입의 조합에 의해서만 구분된다. **반환 타입만 다른 것은 오버로딩으로 인정되지 않는다**.55
- 컴파일러가 함수 호출을 해석할 때, 모호한 경우(ambiguous match), 예를 들어 여러 변환 경로를 통해 두 개 이상의 오버로딩된 함수와 일치할 수 있는 경우에는 컴파일 오류가 발생한다.55


배열은 **동일한 데이터 타입의 원소들을 연속된 메모리 공간에 저장**하는 자료구조이다.

- **선언:** `타입 배열이름[크기];` 형태로 선언한다. C++의 고정 배열(fixed array)에서 `크기`는 반드시 컴파일 시점에 결정되는 상수여야 한다.60

  ```C++
  int scores; // 5개의 int를 저장할 수 있는 배열
  ```

- **초기화:** 선언과 동시에 중괄호 `{}`를 사용하여 초기화할 수 있다.60

  ```C++
  int numbers = {10, 20, 30, 40, 50};
  int fib = {0, 1, 1, 2, 3, 5}; // 크기를 생략하면 초기값 개수로 자동 결정
  int zeros = {0}; // 모든 원소를 0으로 초기화
  ```

  일부만 초기화하면 나머지 원소들은 0으로 자동 초기화된다.61

- **원소 접근:** 배열의 각 원소는 **인덱스(index)**를 통해 접근한다. C++에서 인덱스는 항상 **0부터 시작**하여 `크기-1`까지이다.60

  ```C++
  numbers = 5; // 첫 번째 원소에 5를 할당
  std::cout << numbers; // 세 번째 원소인 30을 출력
  ```

  **주의:** C++는 배열의 경계를 검사하지 않는다. `numbers`와 같이 유효 범위를 벗어난 인덱스에 접근하면 예측할 수 없는 동작(undefined behavior)을 일으켜 프로그램에 심각한 오류를 유발할 수 있다.60


구조체는 서로 다른 타입의 변수들을 하나의 논리적 단위로 묶어 새로운 사용자 정의 데이터 타입을 만드는 기능이다. 예를 들어 '학생'이라는 데이터를 표현하기 위해 이름(`string`), 학번(`int`), 학점(`double`)을 하나의 구조체로 묶을 수 있다.65

```C++
#include <string>
#include <iostream>

struct Student {
    std::string name;
    int studentID;
    double gpa;
};
```

C++의 `struct`는 단순한 데이터 묶음 이상의 의미를 지닌다. 이는 C언어와의 호환성을 유지하면서 객체 지향 프로그래밍으로 나아가는 중요한 징검다리 역할을 한다. C++에서 `struct`는 멤버 함수를 가질 수 있으며, 유일한 차이점은 기본 접근 지정자가 `struct`는 `public`이고 `class`는 `private`라는 점이다.68 이러한 설계는 C의 절차적 패러다임에서 C++의 객체 지향 패러다임으로 자연스럽게 개념을 확장하려는 의도를 보여준다.

- 변수 선언 및 멤버 접근:

  구조체 타입으로 변수를 선언한 뒤, **멤버 선택 연산자 . (점)**를 사용하여 각 멤버에 접근한다.65

  ```C++
  Student s1;
  s1.name = "Grace Hopper";
  s1.studentID = 19061209;
  s1.gpa = 4.5;
  ```

- **초기화:** 배열과 유사하게 중괄호 `{}`를 사용하여 선언과 동시에 초기화할 수 있다.66

  ```C++
  Student s2 = {"Alan Turing", 19120623, 4.4};
  ```

- 구조체와 포인터:

  구조체를 가리키는 포인터를 사용할 때는 **화살표 연산자 ->**를 사용하여 멤버에 접근한다. ptr->member는 (*ptr).member와 완전히 동일한 표현이다.70

  ```C++
  Student* ptr_s = &s2;
  std::cout << "Name: " << ptr_s->name << std::endl; // "Alan Turing" 출력
  ```



포인터는 C++의 가장 강력하면서도 복잡한 기능 중 하나로, 변수의 **메모리 주소**를 직접 저장하고 조작할 수 있게 해준다. 이를 통해 매우 효율적인 프로그램 작성이 가능하지만, 동시에 오류가 발생하기 쉬워 세심한 주의가 필요하다.

- **개념:** 포인터는 다른 변수가 저장된 메모리의 위치(주소)를 값으로 갖는 특별한 변수이다.72

- **주소 연산자 (`&`):** 변수 앞에 붙여 해당 변수의 메모리 주소를 얻는다.73

- **역참조 연산자 (`\*`):** 포인터 변수 앞에 붙여, 해당 포인터가 가리키는 메모리 주소에 저장된 **실제 값**에 접근한다. 이를 통해 값을 읽거나 수정할 수 있다.73

- **선언:** `타입* 포인터이름;` 형태로 선언한다. `*`는 해당 변수가 포인터임을 나타낸다.73

  ```C++
  int var = 10;
  int* ptr;      // int형 변수를 가리킬 포인터 ptr 선언
  
  ptr = &var;    // ptr에 var의 메모리 주소를 할당
  
  std::cout << "Value of var: " << var << std::endl;      // 출력: 10
  std::cout << "Address of var: " << &var << std::endl;   // var의 주소 출력
  std::cout << "Value of ptr: " << ptr << std::endl;      // ptr의 값(var의 주소) 출력
  std::cout << "Value pointed by ptr: " << *ptr << std::endl; // 출력: 10
  
  *ptr = 20;     // ptr이 가리키는 곳의 값을 20으로 변경
  std::cout << "New value of var: " << var << std::endl;  // 출력: 20
  ```

- **NULL 포인터:** 아무것도 가리키지 않는 상태를 나타내는 포인터이다. C++11부터는 `0`이나 `NULL` 매크로 대신 타입에 안전한 `nullptr` 키워드를 사용하는 것이 강력히 권장된다. 포인터를 사용하기 전에는 항상 `nullptr`인지 확인하는 습관이 중요하다.


참조자는 C++에 도입된 개념으로, 이미 존재하는 변수에 대한 **또 다른 이름(별명)**을 부여하는 기능이다.

- **개념:** 참조자는 선언될 때 참조하는 변수와 동일한 메모리 공간을 공유한다. 따라서 참조자를 수정하면 원본 변수도 함께 수정된다.53

- **선언:** `타입& 참조자이름 = 원본변수;` 형태로 선언하며, **선언과 동시에 반드시 초기화해야 한다**.76

  ```C++
  int original = 100;
  int& ref = original; // ref는 original의 별명
  
  std::cout << "Original: " << original << ", Ref: " << ref << std::endl; // 출력: 100, 100
  
  ref = 200; // 참조자를 통해 값을 변경
  
  std::cout << "Original: " << original << ", Ref: " << ref << std::endl; // 출력: 200, 200
  ```

참조자는 포인터의 강력한 기능(원본 데이터에 대한 직접 접근)은 유지하면서, 포인터 사용 시 발생할 수 있는 가장 흔한 오류의 원인(NULL 역참조, 엉뚱한 주소 가리키기)을 문법적으로 차단하기 위해 고안되었다. 이는 성능을 위해 저수준 제어를 허용하되, 더 안전하고 추상화된 도구를 제공하여 개발자의 실수를 줄이려는 C++의 실용적인 설계 철학을 반영한다. "사용할 수 있다면 참조자를, 어쩔 수 없다면 포인터를 써라" 77는 이러한 철학을 잘 보여주는 지침이다.


| 특징        | 포인터 (Pointer)                                          | 참조자 (Reference)                                        |
| ----------- | --------------------------------------------------------- | --------------------------------------------------------- |
| **NULL 값** | `nullptr`로 초기화 가능 (아무것도 가리키지 않음).75       | NULL을 허용하지 않음. 반드시 유효한 변수를 참조해야 함.75 |
| **재할당**  | 다른 변수의 주소를 가리키도록 재할당 가능.78              | 한번 초기화되면 다른 변수를 참조하도록 변경할 수 없음.76  |
| **초기화**  | 선언 후 나중에 주소를 할당할 수 있음.                     | 선언과 동시에 반드시 초기화해야 함.76                     |
| **문법**    | 주소 할당 시 `&`, 값 접근 시 `*` 또는 `->` 연산자 필요.77 | 일반 변수와 동일한 문법으로 사용. 별도의 연산자 불필요.53 |

이러한 차이점 때문에, 함수에 매개변수를 전달하여 원본을 수정하고자 할 때, NULL이 전달될 가능성이 없다면 포인터보다 참조자를 사용하는 것이 더 안전하고 코드가 간결해진다.


객체 지향 프로그래밍(Object-Oriented Programming)은 실제 세계의 개념들을 객체(Object)라는 단위로 모델링하여 프로그래밍하는 패러다임이다. C++는 C언어에 객체 지향 개념을 도입하여 탄생했으며, OOP는 C++의 핵심적인 정체성이다.


- **클래스 (Class):** 객체를 생성하기 위한 **설계도** 또는 **틀(template)**이다. 클래스는 객체가 가지게 될 데이터(속성)와 그 데이터를 조작하는 함수(동작)를 함께 정의한다. 클래스는 `struct`와 유사하지만, 기본 접근 지정자가 `private`이라는 점에서 차이가 있다.68
- **객체 (Object):** 클래스라는 설계도로부터 만들어진 **실체(instance)**이다. 메모리에 실제로 공간을 할당받아 존재하며, 클래스에 정의된 속성과 동작을 가진다.79
- **멤버 변수 (Member Variables):** 클래스 내에 선언된 변수로, 객체의 상태나 속성을 나타낸다.
- **멤버 함수 (Member Functions / Methods):** 클래스 내에 정의된 함수로, 객체의 동작이나 행위를 정의한다. 멤버 함수는 해당 객체의 멤버 변수에 접근하여 조작할 수 있다.81
- **`this` 포인터:** 모든 멤버 함수 내에서 암묵적으로 사용할 수 있는 포인터로, **현재 객체 자신을 가리킨다**. 멤버 변수와 매개변수의 이름이 같을 때 이를 구분하거나, 객체 자신의 주소를 반환할 때 사용된다.82

```C++
class Car {
public: // 접근 지정자
    // 멤버 변수
    std::string color;
    int speed;

    // 멤버 함수
    void accelerate() {
        this->speed += 10; // this 포인터를 사용하여 멤버 변수 speed에 접근
    }

    void brake() {
        speed -= 10;
    }
};

int main() {
    Car myCar; // Car 클래스의 객체(인스턴스) myCar 생성
    myCar.color = "blue";
    myCar.speed = 60;
    myCar.accelerate(); // myCar 객체의 accelerate 멤버 함수 호출
    
    return 0;
}
```


캡슐화와 정보 은닉은 객체 지향 프로그래밍의 핵심 원리 중 하나로, 객체의 내부 데이터를 보호하고 프로그램의 안정성을 높이는 기법이다.

- **캡슐화 (Encapsulation):** 관련된 데이터(멤버 변수)와 그 데이터를 처리하는 함수(멤버 함수)를 하나의 클래스로 묶는 것을 의미한다.83
- **정보 은닉 (Information Hiding):** 객체의 내부 구현(주로 멤버 변수)을 외부에 숨기고, 오직 공개된 인터페이스(public 멤버 함수)를 통해서만 접근을 허용하는 것이다. 이를 통해 외부에서 데이터를 임의로 변경하여 객체가 비정상적인 상태가 되는 것을 방지한다.83
- **접근 지정자 (Access Specifiers):** 정보 은닉을 구현하기 위해 사용된다.68
  - `public`: 클래스 외부 어디에서든 접근할 수 있다. 객체의 공개 인터페이스를 구성한다.
  - `private`: 해당 클래스의 멤버 함수 내에서만 접근할 수 있다. 외부 접근이 차단되어 정보 은닉을 실현한다.
  - `protected`: 해당 클래스와 이 클래스를 상속받은 파생 클래스 내에서만 접근할 수 있다.88

```C++
class BankAccount {
private: // 정보 은닉
    double balance;

public: // 공개 인터페이스
    void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }
    double getBalance() {
        return balance;
    }
};
```

위 예제에서 `balance`는 `private`으로 선언되어 외부에서 직접 접근할 수 없다. `myAccount.balance = -10000;`과 같은 위험한 조작이 불가능하다. 잔액을 변경하거나 조회하려면 반드시 `public`으로 공개된 `deposit`과 `getBalance` 함수를 통해서만 가능하다.


생성자와 소멸자는 객체의 **생명 주기(life cycle)**를 관리하는 특별한 멤버 함수이다.

- **생성자 (Constructor):**
  - **역할:** 객체가 생성될 때 **자동으로 호출**되어 멤버 변수를 초기화하거나 필요한 자원(예: 동적 메모리)을 할당하는 역할을 한다.89
  - **특징:** 클래스 이름과 동일하며, 반환 타입이 없다.89
  - **종류:**
    - **기본 생성자 (Default Constructor):** 매개변수가 없는 생성자. 사용자가 어떤 생성자도 정의하지 않으면 컴파일러가 아무 동작도 하지 않는 기본 생성자를 자동으로 만들어준다.89
    - **매개변수 있는 생성자:** 객체 생성 시 외부로부터 값을 받아 멤버를 초기화한다. 함수 오버로딩을 통해 여러 개의 생성자를 가질 수 있다.93
    - **복사 생성자 (Copy Constructor):** 같은 클래스 타입의 다른 객체를 인자로 받아, 그 객체를 복사하여 새로운 객체를 생성할 때 호출된다. 동적 할당된 멤버가 있을 경우, '깊은 복사(Deep Copy)'를 위해 직접 정의해야 할 수 있다.89
- **소멸자 (Destructor):**
  - **역할:** 객체가 소멸될 때(예: 함수가 끝나 지역 변수가 사라질 때, `delete` 연산자로 동적 할당된 객체를 해제할 때) **자동으로 호출**되어, 생성자에서 할당했던 자원을 해제하는 역할을 한다. 메모리 누수(memory leak)를 방지하는 데 필수적이다.92
  - **특징:** 클래스 이름 앞에 `~`(틸드)를 붙이며, 매개변수와 반환 타입이 없다. 클래스당 오직 하나만 존재할 수 있다.92

```C++
class MyArray {
private:
    int* arr;
    int size;
public:
    // 생성자: 동적 메모리 할당
    MyArray(int s) {
        size = s;
        arr = new int[size];
        std::cout << "Array created." << std::endl;
    }
    // 소멸자: 동적 메모리 해제
    ~MyArray() {
        delete arr;
        std::cout << "Array destroyed." << std::endl;
    }
};
```


상속은 기존 클래스(**기반 클래스** 또는 **부모 클래스**)의 멤버(변수, 함수)를 물려받아 새로운 클래스(**파생 클래스** 또는 **자식 클래스**)를 만드는 기능이다. 코드의 재사용성을 높이고 클래스 간의 계층 구조를 형성할 수 있게 한다.

- **문법:** `class 파생클래스명 : 접근지정자 기반클래스명 {... };`
- **상속 관계에서의 접근 제어:** 파생 클래스가 기반 클래스의 멤버에 어떻게 접근할 수 있는지를 결정한다.
  - **`public` 상속:** 기반 클래스의 `public` 멤버는 파생 클래스에서도 `public`으로, `protected` 멤버는 `protected`로 유지된다. "is-a" 관계(예: '학생'은 '사람'이다)를 표현할 때 주로 사용되는 가장 일반적인 상속 방식이다.
  - **`protected` 상속:** 기반 클래스의 `public`과 `protected` 멤버가 모두 파생 클래스에서 `protected` 멤버가 된다.
  - **`private` 상속:** 기반 클래스의 `public`과 `protected` 멤버가 모두 파생 클래스에서 `private` 멤버가 된다.
  - 어떤 상속 방식을 사용하든, 기반 클래스의 `private` 멤버는 파생 클래스에서 직접 접근할 수 없다.98

```C++
// 기반 클래스
class Person {
public:
    std::string name;
    void introduce() { std::cout << "My name is " << name << std::endl; }
};

// 파생 클래스
class Student : public Person { // Person을 public 상속
public:
    int studentID;
    void study() { std::cout << "I am studying." << std::endl; }
};

int main() {
    Student s;
    s.name = "John";      // 기반 클래스의 public 멤버 사용
    s.studentID = 12345;  // 파생 클래스의 멤버 사용
    s.introduce();        // 기반 클래스의 public 멤버 함수 사용
    s.study();            // 파생 클래스의 멤버 함수 사용
    return 0;
}
```


다형성(Polymorphism)은 '여러 가지 형태를 가짐'이라는 의미로, 객체 지향 프로그래밍에서 하나의 인터페이스(예: 함수 호출)가 서로 다른 타입의 객체에 대해 각기 다른 방식으로 동작하는 능력을 말한다.102


파생 클래스에서 기반 클래스로부터 물려받은 멤버 함수를 **동일한 이름, 동일한 매개변수, 동일한 반환 타입**으로 재정의하는 것을 오버라이딩이라고 한다.103


C++의 다형성은 주로 가상 함수를 통해 실현된다. Java와 같은 언어에서는 모든 메서드가 기본적으로 가상(동적 바인딩)이지만, C++는 기본적으로 **정적 바인딩(Static Binding)**을 사용한다. 즉, 컴파일 시점에 호출될 함수가 결정된다. 이는 "지불하지 않는 기능에 대해서는 비용을 치르지 않는다(You don't pay for what you don't use)"는 C++의 핵심 철학을 보여준다. 가상 함수 테이블과 간접 호출로 인한 미세한 성능 저하조차 다형성이 필요 없는 경우에는 허용하지 않겠다는 것이다.4

- **정적 바인딩의 문제:** 기반 클래스 타입의 포인터로 파생 클래스 객체를 가리킬 때, 해당 포인터를 통해 함수를 호출하면 포인터의 타입, 즉 기반 클래스의 함수가 항상 호출된다. 이는 우리가 기대하는 다형적 동작이 아니다.106
- **동적 바인딩 (Dynamic Binding):** 이 문제를 해결하기 위해 기반 클래스의 함수 선언 앞에 `virtual` 키워드를 붙인다. 이렇게 선언된 함수를 **가상 함수**라고 한다. `virtual` 함수가 호출될 때, 컴파일러는 실행 시간에 포인터가 **실제로 가리키고 있는 객체의 타입**을 확인하여 해당 타입에 맞게 오버라이딩된 함수를 호출한다. 이것이 동적 바인딩이다.103

```C++
class Animal {
public:
    virtual void speak() { std::cout << "Animal speaks" << std::endl; }
    virtual ~Animal() {} // 상속 계층에서는 소멸자를 가상으로 선언하는 것이 안전하다.
};

class Dog : public Animal {
public:
    void speak() override { // 'override' 키워드는 오버라이딩을 명시 (C++11)
        std::cout << "Woof!" << std::endl;
    }
};

class Cat : public Animal {
public:
    void speak() override {
        std::cout << "Meow!" << std::endl;
    }
};

void doSpeak(Animal* animal) {
    animal->speak(); // 동적 바인딩 발생
}

int main() {
    Animal* pDog = new Dog();
    Animal* pCat = new Cat();

    doSpeak(pDog); // 출력: Woof!
    doSpeak(pCat); // 출력: Meow!

    delete pDog;
    delete pCat;
    return 0;
}
```

- **순수 가상 함수와 추상 클래스:** `virtual void func() = 0;`과 같이 함수 몸체 없이 선언된 가상 함수를 **순수 가상 함수(Pure Virtual Function)**라고 한다. 순수 가상 함수를 하나 이상 포함하는 클래스는 **추상 클래스(Abstract Class)**가 되어 객체를 직접 생성할 수 없다. 추상 클래스를 상속받는 파생 클래스는 반드시 모든 순수 가상 함수를 오버라이딩해야만 객체를 생성할 수 있다. 이는 파생 클래스에게 특정 인터페이스의 구현을 강제하는 역할을 한다.101



템플릿은 데이터 타입에 의존하지 않는 **범용(Generic) 코드**를 작성하게 해주는 C++의 강력한 기능이다. 동일한 로직을 다른 데이터 타입에 대해 반복적으로 작성할 필요 없이, 하나의 템플릿으로 여러 타입에서 동작하는 함수나 클래스를 만들 수 있다.

- 함수 템플릿 (Function Template):

  특정 타입 대신 임의의 타입을 나타내는 템플릿 매개변수(예: T)를 사용하여 함수를 정의한다. template <typename T> 또는 template <class T> 구문을 사용한다.109 컴파일러는 함수가 호출될 때 전달되는 인자의 타입을 분석하여 해당 타입에 맞는 함수 코드를 자동으로 생성(인스턴스화)한다.109

  ```C++
  template <typename T>
  T getMax(T a, T b) {
      return (a > b)? a : b;
  }
  
  int main() {
      std::cout << getMax(5, 10) << std::endl;       // T가 int로 인스턴스화
      std::cout << getMax(3.14, 2.71) << std::endl; // T가 double로 인스턴스화
  }
  ```

- 클래스 템플릿 (Class Template):

  임의의 타입을 멤버 변수나 멤버 함수의 타입으로 사용하는 범용 클래스를 정의한다.109 클래스 템플릿으로 객체를 생성할 때는 꺾쇠괄호 `<>` 안에 사용할 구체적인 타입을 명시해야 한다.111

  ```C++
  template <typename T>
  class Box {
  private:
      T content;
  public:
      void setContent(T c) { content = c; }
      T getContent() { return content; }
  };
  
  int main() {
      Box<int> intBox;       // T가 int인 Box 클래스
      intBox.setContent(123);
  
      Box<std::string> stringBox; // T가 std::string인 Box 클래스
      stringBox.setContent("Hello Template");
  }
  ```


예외 처리는 프로그램 실행 중에 발생하는 예측 불가능한 오류(예: 파일 열기 실패, 메모리 할당 실패, 0으로 나누기 등)를 구조적으로 처리하는 메커니즘이다. `if` 문으로 오류를 처리하는 방식보다 코드의 가독성과 안정성을 높여준다.114

- **`try`, `catch`, `throw`:**
  - `try`: 예외가 발생할 가능성이 있는 코드를 `try` 블록으로 감싼다.114
  - `throw`: `try` 블록 내에서 예외적인 상황이 발생하면, `throw` 키워드를 사용하여 예외를 발생시킨다. `throw`는 정수, 문자열, 객체 등 어떤 타입의 값이든 던질 수 있다.114
  - `catch`: `throw`에 의해 던져진 예외를 받아서 처리하는 블록이다. `catch`는 처리할 예외의 타입을 명시해야 하며, 던져진 예외의 타입과 일치하는 `catch` 블록이 실행된다. `catch(...)`는 모든 타입의 예외를 처리할 수 있다.115

```C++
#include <iostream>
#include <stdexcept>

double divide(int a, int b) {
    if (b == 0) {
        throw std::runtime_error("Division by zero error!");
    }
    return static_cast<double>(a) / b;
}

int main() {
    try {
        double result = divide(10, 0);
        std::cout << "Result: " << result << std::endl;
    } catch (const std::runtime_error& e) {
        std::cerr << "Exception caught: " << e.what() << std::endl;
    }
    return 0;
}
```

위 예제에서 `divide` 함수는 0으로 나누려는 시도가 있을 때 `std::runtime_error` 타입의 예외를 `throw`한다. `main` 함수에서는 `try` 블록 안에서 `divide` 함수를 호출하고, 예외가 발생하면 `catch` 블록이 이를 잡아 처리한다.


C++에서는 `<fstream>` 헤더 파일을 통해 파일에 데이터를 쓰거나 파일로부터 데이터를 읽어오는 스트림 기반의 파일 입출력 기능을 제공한다.

- **파일 쓰기 (`ofstream`):** `ofstream` (Output File Stream) 객체를 사용하여 파일에 데이터를 쓴다. `cout`과 마찬가지로 `<<` 연산자를 사용한다.118
- **파일 읽기 (`ifstream`):** `ifstream` (Input File Stream) 객체를 사용하여 파일에서 데이터를 읽는다. `cin`과 마찬가지로 `>>` 연산자나 `getline` 함수를 사용한다.118
- **파일 열기와 닫기:**
  - 객체를 생성할 때 파일 경로를 인자로 전달하여 파일을 열 수 있다. `open()` 멤버 함수를 사용할 수도 있다.118
  - 파일을 열 때는 다양한 모드(예: `ios::app`은 이어쓰기, `ios::binary`는 바이너리 모드)를 지정할 수 있다.118
  - 파일 작업이 끝나면 `close()` 함수로 명시적으로 닫거나, 객체가 범위를 벗어나 소멸될 때 자동으로 닫힌다 (RAII 원칙).118

```C++
#include <iostream>
#include <fstream>
#include <string>

int main() {
    // 파일 쓰기
    std::ofstream outFile("example.txt");
    if (outFile.is_open()) {
        outFile << "This is a line." << std::endl;
        outFile << "This is another line." << std::endl;
        outFile.close();
    } else {
        std::cerr << "Unable to open file for writing." << std::endl;
    }

    // 파일 읽기
    std::ifstream inFile("example.txt");
    std::string line;
    if (inFile.is_open()) {
        while (getline(inFile, line)) {
            std::cout << line << std::endl;
        }
        inFile.close();
    } else {
        std::cerr << "Unable to open file for reading." << std::endl;
    }

    return 0;
}
```


표준 템플릿 라이브러리(Standard Template Library, STL)는 C++ 표준 라이브러리의 핵심으로, 프로그래머가 직접 구현할 필요 없이 바로 사용할 수 있는 효율적인 자료구조와 알고리즘의 집합이다.122

STL의 진정한 혁신은 단순히 유용한 자료구조와 알고리즘을 제공하는 데 있지 않다. 그 핵심은 **컨테이너(데이터 저장 방식), 반복자(데이터 접근 방식), 알고리즘(데이터 처리 방식)을 완전히 분리**한 설계에 있다.123 이 세 구성 요소의 '직교성(Orthogonality)' 덕분에, `sort`라는 단 하나의 알고리즘이 `vector`뿐만 아니라 `deque`, 심지어 C-스타일 배열에도 동일하게 적용될 수 있다. 필요한 것은 단지 컨테이너가 표준을 따르는 '반복자'를 제공하는 것뿐이다. 이는 n개의 컨테이너와 m개의 알고리즘이 있을 때 n*m개의 구현이 아닌 n+m개의 구현만으로 시스템을 구성할 수 있게 하는, 소프트웨어 공학적으로 매우 우아하고 효율적인 해결책이다.


1. **컨테이너 (Containers):** 데이터를 저장하고 관리하는 객체.
   - **순차 컨테이너 (Sequence Containers):** `vector`(동적 배열), `list`(이중 연결 리스트), `deque`(양방향 큐) 등 데이터를 선형 순서로 저장한다.125
   - **연관 컨테이너 (Associative Containers):** `map`(키-값 쌍), `set`(키 집합) 등 키를 기반으로 데이터를 저장하며, 자동으로 정렬된다.125
   - **비정렬 연관 컨테이너 (Unordered Associative Containers):** `unordered_map`, `unordered_set` 등 해시 테이블을 사용하여 정렬 없이 더 빠른 삽입/삭제/검색을 제공한다.125
2. **반복자 (Iterators):** 컨테이너의 원소를 가리키고 순회하는 포인터와 유사한 객체이다. 알고리즘이 특정 컨테이너의 내부 구조에 의존하지 않고 동작할 수 있도록 하는 추상화 계층을 제공한다.126
3. **알고리즘 (Algorithms):** `<algorithm>` 헤더에 정의된, 컨테이너의 데이터에 대해 정렬, 검색, 수정 등의 작업을 수행하는 템플릿 함수들이다. `sort`, `find`, `copy`, `for_each` 등이 있다.125


`vector`는 자동으로 크기가 조절되는 동적 배열로, STL에서 가장 널리 사용되는 컨테이너 중 하나이다.

- **선언:** `#include <vector>` 헤더가 필요하다. `vector<타입> 이름;` 형태로 선언한다.128
- **원소 추가/삭제:** `push_back()`으로 끝에 원소를 추가하고, `pop_back()`으로 끝의 원소를 제거한다.129
- **원소 접근:** 일반 배열처럼 `` 연산자로 접근하거나, 경계 검사를 수행하는 `at()` 함수를 사용할 수 있다.129
- **크기:** `size()` 함수로 현재 원소의 개수를 알 수 있다.129

```C++
#include <iostream>
#include <vector>
#include <string>

int main() {
    std::vector<std::string> names;

    names.push_back("Alice");
    names.push_back("Bob");
    names.push_back("Charlie");

    std::cout << "First name: " << names.front() << std::endl; // names과 동일
    std::cout << "Last name: " << names.back() << std::endl;

    // 범위 기반 for 루프를 이용한 순회
    for (const std::string& name : names) {
        std::cout << name << " ";
    }
    std::cout << std::endl;

    names.pop_back(); // "Charlie" 제거
    std::cout << "Size after pop: " << names.size() << std::endl;
    
    return 0;
}
```


`map`은 키(Key)와 값(Value)을 쌍으로 저장하는 연관 컨테이너이다. 키는 유일해야 하며, 키를 기준으로 자동 정렬된다.

- **선언:** `#include <map>` 헤더가 필요하다. `map<키타입, 값타입> 이름;` 형태로 선언한다.133
- **삽입:** `insert(make_pair(key, value))` 또는 `map[key] = value` 형태로 삽입한다. 후자의 방식이 더 간편하다.135
- **접근 및 검색:** `map[key]`로 값에 접근할 수 있다. `find(key)` 함수는 해당 키를 가진 원소의 반복자를 반환하며, 키가 없으면 `end()` 반복자를 반환한다.134

```C++
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<std::string, int> ages;

    ages["Alice"] = 30;
    ages = 25;
    ages.insert(std::make_pair("Charlie", 35));

    std::cout << "Bob's age: " << ages << std::endl;

    // map 순회 (키 기준 정렬)
    for (const auto& pair : ages) {
        std::cout << pair.first << " is " << pair.second << " years old." << std::endl;
    }

    return 0;
}
```


아래 표는 C++ 연산자들의 우선순위와 결합 규칙을 정리한 것이다. 우선순위가 높을수록 먼저 계산되며, 같은 우선순위의 연산자들은 결합 방향에 따라 계산 순서가 결정된다. 복잡한 표현식에서는 괄호 `()`를 사용하여 의도를 명확히 하는 것이 권장된다.31

| 우선순위 | 연산자                                                       | 설명                                                         | 결합 방향                              |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ | -------------------------------------- |
| 1        | `::`                                                         | 범위 지정                                                    | 없음                                   |
| 2        | `()` `` `.` `->` `++` `--` `typeid` `const_cast` `dynamic_cast` `reinterpret_cast` `static_cast` | 함수 호출, 배열 첨자, 멤버 접근, 후위 증감, 타입 식별, 타입 캐스팅 | 왼쪽 -->> 오른쪽                          |
| 3        | `!` `~` `++` `--` `+` `-` `&` `*` `sizeof` `new` `delete` `(type)` | 단항 연산자 (논리 NOT, 비트 NOT, 전위 증감, 부호, 주소/역참조, 크기, 동적 할당/해제), C-스타일 캐스트 | 오른쪽 -->> 왼쪽                          |
| 4        | `.*` `->*`                                                   | 멤버에 대한 포인터                                           | 왼쪽 -->> 오른쪽                          |
| 5        | `*` `/` `%`                                                  | 곱셈, 나눗셈, 나머지                                         | 왼쪽 -->> 오른쪽                          |
| 6        | `+` `-`                                                      | 덧셈, 뺄셈                                                   | 왼쪽 -->> 오른쪽                          |
| 7        | `<<` `>>`                                                    | 비트 시프트                                                  | 왼쪽 -->> 오른쪽                          |
| 8        | `<` `<=` `>` `>=`                                            | 관계 연산자                                                  | 왼쪽 -->> 오른쪽                          |
| 9        | `==` `!=`                                                    | 동등 연산자                                                  | 왼쪽 -->> 오른쪽                          |
| 10       | `&`                                                          | 비트 AND                                                     | 왼쪽 -->> 오른쪽                          |
| 11       | `^`                                                          | 비트 XOR                                                     | 왼쪽 -->> 오른쪽                          |
| 12       | `                                                            | `                                                            | 비트 OR                                |
| 13       | `&&`                                                         | 논리 AND                                                     | 왼쪽 -->> 오른쪽                          |
| 14       | `                                                            |                                                              | `                                      |
| 15       | `? :` `=` `+=` `-=` `*=` `/=` `%=` `<<=` `>>=` `&=` `^=` `   | =` `throw`                                                   | 조건(삼항), 대입, 복합 대입, 예외 발생 |
| 16       | `,`                                                          | 쉼표                                                         | 왼쪽 -->> 오른쪽                          |


1. C++ 언어 소개 - 안동민 개발노트, 8월 3, 2025에 액세스, https://andongmin.com/docs/cpp/ch1/ch1-1
2. C++ - 위키백과, 우리 모두의 백과사전, 8월 3, 2025에 액세스, https://ko.wikipedia.org/wiki/C%2B%2B
3. [C++] C++란, C++의 특징 - 잡동사니 - 티스토리, 8월 3, 2025에 액세스, https://bonsilrote.tistory.com/8
4. [C++] C++ 역사, 특징, 개발과정 - 블로그, 8월 3, 2025에 액세스, https://ehs2803.github.io/c++/cpp-start/
5. [C++ 독학] 알아도 쓸데없는 프로그래밍 언어의 역사 - NpsCause blog, 8월 3, 2025에 액세스, https://npscause.tistory.com/20
6. [기술의역사] C++ (C 플러스 플러스) - DEV_NUNU - 티스토리, 8월 3, 2025에 액세스, https://new93helloworld.tistory.com/23
7. C++ 이란? - tyoon9781 - 티스토리, 8월 3, 2025에 액세스, https://tyoon9781.tistory.com/entry/cpp-introduce
8. [C++] C++ 시작하기 - 특징, 객체 지향, 컴파일과 링킹 - HERSTORY, 8월 3, 2025에 액세스, https://4z7l.github.io/2020/07/16/cpp-1.html
9. C++ - C++ 언어의 특징 - velog, 8월 3, 2025에 액세스, [https://velog.io/@liz/C-C-%EC%96%B8%EC%96%B4%EC%9D%98-%ED%8A%B9%EC%A7%95](https://velog.io/@liz/C-C-언어의-특징)
10. C 언어와 C++ 언어의 호환성과 역사 - Tae_S - 티스토리, 8월 3, 2025에 액세스, https://security1.tistory.com/11
11. C++ 개발자를 위한 필수 도구와 IDE 소개 🛠️ - 재능넷, 8월 3, 2025에 액세스, https://www.jaenung.net/tree/8745
12. [C, C++] 무료 IDE 추천 - Coding Space By MIR - 티스토리, 8월 3, 2025에 액세스, https://holygod.tistory.com/25
13. Windows용 Visual Studio C/C++ IDE 및 컴파일러, 8월 3, 2025에 액세스, https://visualstudio.microsoft.com/ko/vs/features/cplusplus/
14. VSCode로 C++ 개발 환경 구축하기 (알고리즘) - Amylo's blog, 8월 3, 2025에 액세스, https://blog.amylo.diskstation.me/algorithm/Starting_Algorithm_with_VSCode_C++/
15. 【C/C++환경설정】Visual Studio Code에서 C/C++ 코딩 환경 구축하기[Configure Visual Studio Code for C/C++] - 신중앙컴퓨터학원 - 티스토리, 8월 3, 2025에 액세스, https://3dlecture.tistory.com/9
16. [Visual Studio Code] VS Code C++ 개발 환경 설정 (MinGW) - 모징이의 개발 경험치, 8월 3, 2025에 액세스, [https://mojing.tistory.com/entry/Visual-Studio-Code-VS-Code-C-%EA%B0%9C%EB%B0%9C-%ED%99%98%EA%B2%BD-%EC%84%A4%EC%A0%95](https://mojing.tistory.com/entry/Visual-Studio-Code-VS-Code-C-개발-환경-설정)
17. Visual Studio에서 C 및 C++ 지원 설치 | Microsoft Learn, 8월 3, 2025에 액세스, https://learn.microsoft.com/ko-kr/cpp/build/vscpp-step-0-installation?view=msvc-170
18. [C++] 입력문 / 출력문 (cin, cout) 사용법 & 예제 - 코딩팩토리 - 티스토리, 8월 3, 2025에 액세스, https://coding-factory.tistory.com/479
19. 따라하며 배우는 C++ 1. C++의 기초적인 사용법, 8월 3, 2025에 액세스, https://maxcomfem.tistory.com/2
20. 알고리즘 - C++ 기초 / 자주 쓰이는 문법들 - ChanBLOG, 8월 3, 2025에 액세스, https://chanhuiseok.github.io/posts/algo-2/
21. [C언어/C++] 함수(Function) 사용법 & 예제 - 코딩팩토리, 8월 3, 2025에 액세스, https://coding-factory.tistory.com/637
22. [C++] #1 C++ 에서의 입출력 (std::cout, std::cin, std::endl) - 요이로그 - 티스토리, 8월 3, 2025에 액세스, https://omyodevelop.tistory.com/33
23. [C++] cout,cin 사용법 - 돼민이 - 티스토리, 8월 3, 2025에 액세스, https://jink1982.tistory.com/37
24. [C++ 독학] 변수와 타입
25. C++ 변수와 자료형 - Gyong - 티스토리, 8월 3, 2025에 액세스, https://gyong0.tistory.com/101
26. C++ 문법 공부 - 데이터 타입(data type) - 쉬고 싶은 개발자 - 티스토리, 8월 3, 2025에 액세스, https://offbyone.tistory.com/115
27. [C/C++] C언어의 기본 자료형/데이터 타입(Data Type) 종류 - Logic - 티스토리, 8월 3, 2025에 액세스, [https://kimasill.tistory.com/entry/CC-C%EC%96%B8%EC%96%B4%EC%9D%98-%EA%B8%B0%EB%B3%B8-%EC%9E%90%EB%A3%8C%ED%98%95%EB%8D%B0%EC%9D%B4%ED%84%B0-%ED%83%80%EC%9E%85Data-Type-%EC%A2%85%EB%A5%98](https://kimasill.tistory.com/entry/CC-C언어의-기본-자료형데이터-타입Data-Type-종류)
28. (3) C++ 변수의 데이터 타입 & Variables(변수) - 제임스 지식 보고, 8월 3, 2025에 액세스, https://jisik-bogo.tistory.com/12
29. C++ : 변수와 자료형 (variable and data type) - 달나라 노트 - 티스토리, 8월 3, 2025에 액세스, https://cosmosproject.tistory.com/511
30. [C/C++ 프로그래밍] 5. 연산자, 8월 3, 2025에 액세스, https://gdngy.tistory.com/151
31. C++ 연산자 우선 순위 - MY블로그, 8월 3, 2025에 액세스, https://rhksgml78.tistory.com/8
32. [c/c++] 연산자 우선순위(Operator Priority) - 컴공생의 다이어리, 8월 3, 2025에 액세스, https://computer-science-student.tistory.com/156
33. 25.0 연산자 우선순위 알아보기 - C 언어 코딩 도장, 8월 3, 2025에 액세스, https://dojang.io/mod/page/view.php?id=188
34. C++ | 03.01 연산자 우선순위와 결합법칙(Operator precedence and associativity ), 8월 3, 2025에 액세스, https://heroine-day.tistory.com/30
35. [C++] 기본 입출력 ( cin | cout | scanf | printf ) - 뚜벅뚜벅 컴공맨 - 티스토리, 8월 3, 2025에 액세스, [https://where-i-am.tistory.com/entry/C-%EA%B8%B0%EB%B3%B8-%EC%9E%85%EC%B6%9C%EB%A0%A5-cin-cout-scanf-printf](https://where-i-am.tistory.com/entry/C-기본-입출력-cin-cout-scanf-printf)
36. [C++] 입출력 ( cin / cout ) - Rebro의 코딩 일기장, 8월 3, 2025에 액세스, https://rebro.kr/30
37. C 언어 / C++ 언어 제6강 조건문 (if문, else-if문, switch문) - 알버트의 개발하는 블로그, 8월 3, 2025에 액세스, https://yghan.tistory.com/9
38. C++ Chapter 5.1 : 조건 분기 (if문, switch-case문) - 평생 공부 블로그, 8월 3, 2025에 액세스, https://ansohxxn.github.io/cpp/chapter5-1/
39. [C++] 조건문 - 코딩이랑 이것저것, 8월 3, 2025에 액세스, https://wn42.tistory.com/85
40. C++ 06.02 - switch 문 (switch statement) - 소년코딩 - 티스토리, 8월 3, 2025에 액세스, https://boycoding.tistory.com/186
41. [C/C++] switch ~ case 문의 활용방법 - 흔한 실수까지, 8월 3, 2025에 액세스, https://reakwon.tistory.com/204
42. C++ Switch 구문 - 김코딩의 개발로그, 8월 3, 2025에 액세스, https://eccsck.tistory.com/54
43. C 언어 / C++ 언어 제7강 반복문 (for문, while문, do while문), 8월 3, 2025에 액세스, https://yghan.tistory.com/10
44. [C++] 반복문 (for문, while문) - AtoZ inventory - 티스토리, 8월 3, 2025에 액세스, [https://scene-inventory.tistory.com/entry/C-%EB%B0%98%EB%B3%B5%EB%AC%B8-for%EB%AC%B8-while%EB%AC%B8](https://scene-inventory.tistory.com/entry/C-반복문-for문-while문)
45. [카루의 C++ 강좌] 3-1. 반복문과 for, while, do~while, 8월 3, 2025에 액세스, https://karupro.tistory.com/26
46. C++의 반복처리(for, 범위 기반 for문, while, do~while) - 황민혁 트러블슈팅 모음집, 8월 3, 2025에 액세스, https://koey.tistory.com/287
47. 소년코딩 - C++ 06.04 - while 문 (while statement) - 티스토리, 8월 3, 2025에 액세스, https://boycoding.tistory.com/188
48. 코딩 기초 20편 _ C++ 코딩 반복문 while문, do/while문, for문 비교와 예제 코드 - 지식아일랜드, 8월 3, 2025에 액세스, https://knowledge-island.tistory.com/44
49. [C/C++ 프로그래밍] 8. 함수 - GDNGY의 함께 만들어가는 테크노트 (Technote), 8월 3, 2025에 액세스, https://gdngy.tistory.com/154
50. [C++ 기본 공부정리] 11-1. 함수(function) 기본 - 정리노트, 8월 3, 2025에 액세스, [https://min-zero.tistory.com/entry/C-%EA%B8%B0%EB%B3%B8-%EA%B3%B5%EB%B6%80%EC%A0%95%EB%A6%AC-11-1-%ED%95%A8%EC%88%98function-%EA%B8%B0%EB%B3%B8](https://min-zero.tistory.com/entry/C-기본-공부정리-11-1-함수function-기본)
51. [C++] 함수(Function) - 코딩이랑 이것저것, 8월 3, 2025에 액세스, https://wn42.tistory.com/91
52. [C++] call by value, call by address, call by reference 차이 - rehtorbs - 티스토리, 8월 3, 2025에 액세스, https://rehtorb-algorithm.tistory.com/11
53. C++ 참조자(Reference)의 이해 - A Game Programmer - 티스토리, 8월 3, 2025에 액세스, https://devshovelinglife.tistory.com/535
54. [C++] 함수재정의 오버로딩(overloading) - rehtorbs - 티스토리, 8월 3, 2025에 액세스, https://rehtorb-algorithm.tistory.com/13
55. 소년코딩 - C++ 08.07 - 함수 오버로딩 (Function overloading) - 티스토리, 8월 3, 2025에 액세스, https://boycoding.tistory.com/221
56. [C++ 기본 공부정리] 11-6. 함수 오버로딩(function overloading), 8월 3, 2025에 액세스, [https://min-zero.tistory.com/entry/C-%EA%B8%B0%EB%B3%B8-%EA%B3%B5%EB%B6%80%EC%A0%95%EB%A6%AC-11-6-%ED%95%A8%EC%88%98-%EC%98%A4%EB%B2%84%EB%A1%9C%EB%94%A9function-overloading](https://min-zero.tistory.com/entry/C-기본-공부정리-11-6-함수-오버로딩function-overloading)
57. [C++] 함수 오버로딩 ( Function overloading ) - Song 컴퓨터공학 - 티스토리, 8월 3, 2025에 액세스, https://songsite123.tistory.com/13
58. C++ 함수 오버로딩 - 열코의 프로그래밍 일기, 8월 3, 2025에 액세스, https://yeolco.tistory.com/118
59. 함수 오버로드 - Learn Microsoft, 8월 3, 2025에 액세스, https://learn.microsoft.com/ko-kr/cpp/cpp/function-overloading?view=msvc-170
60. [C/C++ 프로그래밍] 9. 배열, 8월 3, 2025에 액세스, https://gdngy.tistory.com/155
61. C/C++ 배열 사용법 - 열코의 프로그래밍 일기 - 티스토리, 8월 3, 2025에 액세스, https://yeolco.tistory.com/107
62. C++ 07.01 - 배열, Array (Part 1) - 소년코딩 - 티스토리, 8월 3, 2025에 액세스, https://boycoding.tistory.com/193
63. [c++] array 선언 초기화 - IagreeBUT - 티스토리, 8월 3, 2025에 액세스, https://iagreebut.tistory.com/297
64. [ c++ ] 배열 초기화, 8월 3, 2025에 액세스, https://yebeen-study-note.tistory.com/38
65. [C] 구조체(struct). 구조체의 개념과 배열 사용 예시 | by EunJin | Medium, 8월 3, 2025에 액세스, [https://jin0904.medium.com/c-%EA%B5%AC%EC%A1%B0%EC%B2%B4-struct-a82bae699581](https://jin0904.medium.com/c-구조체-struct-a82bae699581)
66. C++ 05.07 - 구조체, struct - 소년코딩, 8월 3, 2025에 액세스, https://boycoding.tistory.com/183
67. [C++ 기본 공부정리] 9. 구조체(struct) - 정리노트 - 티스토리, 8월 3, 2025에 액세스, [https://min-zero.tistory.com/entry/C-%EA%B8%B0%EB%B3%B8-%EA%B3%B5%EB%B6%80%EC%A0%95%EB%A6%AC-9-%EA%B5%AC%EC%A1%B0%EC%B2%B4struct](https://min-zero.tistory.com/entry/C-기본-공부정리-9-구조체struct)
68. [C++] 클래스와 객체 요약 - teinoi - 티스토리, 8월 3, 2025에 액세스, https://teinoi.tistory.com/24
69. [C언어/C++] 구조체 사용법 & 예제 총정리 - 코딩팩토리 - 티스토리, 8월 3, 2025에 액세스, https://coding-factory.tistory.com/639
70. C/C++ 구조체 사용법 및 예제 - 열코의 프로그래밍 일기, 8월 3, 2025에 액세스, https://yeolco.tistory.com/113
71. [C++] 구조체(Struct) - 코딩이랑 이것저것, 8월 3, 2025에 액세스, https://wn42.tistory.com/94
72. [C/C++ 프로그래밍] 11. 포인터 - GDNGY의 함께 만들어가는 테크노트 (Technote), 8월 3, 2025에 액세스, https://gdngy.tistory.com/167
73. [C++] 포인터(Pointer) 기초 - 코딩이랑 이것저것, 8월 3, 2025에 액세스, https://wn42.tistory.com/89
74. [C++ 때려잡기] C++ 기초강의 4-3 마침내 포인터, 포인터 기초 - SeeRoE 프로그래밍 기록, 8월 3, 2025에 액세스, https://see-ro-e.tistory.com/27
75. [C & C++] 참조자(Reference)와 포인터(Pointer)의 차이 - 이론과 실습 사이 - 티스토리, 8월 3, 2025에 액세스, https://min-310.tistory.com/56
76. [C++] 참조자(Reference)에 대해서 - 별준 - 티스토리, 8월 3, 2025에 액세스, https://junstar92.tistory.com/111
77. [C++] 포인터(Pointer)와 레퍼런스(Reference : 참조자)의 차이, 8월 3, 2025에 액세스, https://gracefulprograming.tistory.com/11
78. [C++] Reference 와 Pointer 의 차이점 설명해줄래 - Guru_Park의 블로그 - 티스토리, 8월 3, 2025에 액세스, https://guru.tistory.com/136
79. [C/C++ 프로그래밍 : 중급] 2. 클래스와 객체, 8월 3, 2025에 액세스, https://gdngy.tistory.com/174
80. [C++ 때려잡기] C++ 심화강의 1 객체 지향 프로그래밍과 클래스 - SeeRoE 프로그래밍 기록, 8월 3, 2025에 액세스, https://see-ro-e.tistory.com/33
81. [C++] 클래스와 메소드 - 고민보단 도전을 - 티스토리, 8월 3, 2025에 액세스, https://swblossom.tistory.com/5
82. [C++] 클래스(Class) 사용법 & 예제 총정리 - 코딩팩토리 - 티스토리, 8월 3, 2025에 액세스, https://coding-factory.tistory.com/697
83. [C++] 정보은닉과 캡슐화 (Information Hiding & Encapsulation), 8월 3, 2025에 액세스, [https://pacs.tistory.com/entry/C-%EC%A0%95%EB%B3%B4%EC%9D%80%EB%8B%89%EA%B3%BC-%EC%BA%A1%EC%8A%90%ED%99%94-Information-Hiding-Encapsulation](https://pacs.tistory.com/entry/C-정보은닉과-캡슐화-Information-Hiding-Encapsulation)
84. [11/23] 정보은닉 VS 캡슐화 - 개발새발, 8월 3, 2025에 액세스, https://bean-soup-99.tistory.com/89
85. 정보은닉(Data Hiding)과 캡슐화(Encapsulation) - 망스토리, 8월 3, 2025에 액세스, [https://themangs.tistory.com/entry/%EC%A0%95%EB%B3%B4%EC%9D%80%EB%8B%89Data-Hiding%EA%B3%BC-%EC%BA%A1%EC%8A%90%ED%99%94Encapsulation](https://themangs.tistory.com/entry/정보은닉Data-Hiding과-캡슐화Encapsulation)
86. C++ | 좋은 클래스의 조건, 정보은닉과 캡슐화, 8월 3, 2025에 액세스, https://koey.tistory.com/60
87. [자바 CS지식] 캡슐화와 정보 은닉의 차이점 - 소란한 블로그 - 티스토리, 8월 3, 2025에 액세스, https://bashu.tistory.com/22
88. C++ public, protected, private에 대한 설명 - HwanShell - 티스토리, 8월 3, 2025에 액세스, https://hwan-shell.tistory.com/25
89. [C++] 생성자(Constructor) - velog, 8월 3, 2025에 액세스, [https://velog.io/@leesu1012/C-%EC%83%9D%EC%84%B1%EC%9E%90Constructor](https://velog.io/@leesu1012/C-생성자Constructor)
90. [C++] 생성자 (Constructor) - 코딩이랑 이것저것 - 티스토리, 8월 3, 2025에 액세스, https://wn42.tistory.com/104
91. [C/C++ 프로그래밍 : 중급] 3. 생성자와 소멸자, 8월 3, 2025에 액세스, https://gdngy.tistory.com/175
92. [C++] 생성자와 소멸자 - 코딩하는 멍멍이 - 티스토리, 8월 3, 2025에 액세스, https://zheldajdajd.tistory.com/16
93. 소년코딩 - C++ 09.05 - 생성자 (Constructor) - 티스토리, 8월 3, 2025에 액세스, https://boycoding.tistory.com/244
94. 소멸자 (Destructor) - C++ 09.09 - 소년코딩 - 티스토리, 8월 3, 2025에 액세스, https://boycoding.tistory.com/249
95. [C++ 기본 공부정리] 14-5. OOP - 소멸자(destructor), 8월 3, 2025에 액세스, [https://min-zero.tistory.com/entry/C-%EA%B8%B0%EB%B3%B8-%EA%B3%B5%EB%B6%80%EC%A0%95%EB%A6%AC-14-4-OOP-%EC%86%8C%EB%A9%B8%EC%9E%90destructor](https://min-zero.tistory.com/entry/C-기본-공부정리-14-4-OOP-소멸자destructor)
96. 41. C++ 소멸자 destructor - YouTube, 8월 3, 2025에 액세스, https://www.youtube.com/watch?v=hWEz3C0_jKQ
97. 소멸자(destructor) - 코딩의 시작, TCP School, 8월 3, 2025에 액세스, https://tcpschool.com/cpp/cpp_conDestructor_destructor
98. [ C++ ] public 상속, protected 상속, private 상속 - 청포도 에이드, 8월 3, 2025에 액세스, [https://musket-ade.tistory.com/entry/C-public-%EC%83%81%EC%86%8D-protected-%EC%83%81%EC%86%8D-private-%EC%83%81%EC%86%8D](https://musket-ade.tistory.com/entry/C-public-상속-protected-상속-private-상속)
99. C++ :: protected 상속, private 상속, IS-A 관계, HAS-A 관계 - 도순씨의 코딩일지 - 티스토리, 8월 3, 2025에 액세스, https://dosundosun.tistory.com/30
100. [C++] 상속 두번째, 세가지 형태의 상속 (private, protected, public) - 티스토리, 8월 3, 2025에 액세스, [https://pacs.tistory.com/entry/C-%EC%83%81%EC%86%8D-%EB%91%90%EB%B2%88%EC%A7%B8-%EC%84%B8%EA%B0%80%EC%A7%80-%ED%98%95%ED%83%9C%EC%9D%98-%EC%83%81%EC%86%8D-private-protected-public](https://pacs.tistory.com/entry/C-상속-두번째-세가지-형태의-상속-private-protected-public)
101. 2. 접근 권한 키워드 - public, private, protected, 파생 클래스 _ C++ - 머리없는개발자, 8월 3, 2025에 액세스, https://nybot-house.tistory.com/59
102. [C/C++ 프로그래밍 : 중급] 6. 다형성, 8월 3, 2025에 액세스, https://gdngy.tistory.com/178
103. [C++ 객체지향] 다형성(Polymorphism) - bdfgdfg - 티스토리, 8월 3, 2025에 액세스, https://marmelo12.tistory.com/285
104. C++ 오버로딩과 오버라이딩 - Insert Brain Here - 티스토리, 8월 3, 2025에 액세스, https://hwan1402.tistory.com/87
105. 오버라이딩 (Overriding) - 꽈이의 게임개발 - 티스토리, 8월 3, 2025에 액세스, https://younggwan.tistory.com/29
106. C++ 다형성과 가상함수 - Insert Brain Here - 티스토리, 8월 3, 2025에 액세스, https://hwan1402.tistory.com/82
107. [C++] virtual function (가상 함수), function override(오버라이드) - 쓸쓸한 공부방 - 티스토리, 8월 3, 2025에 액세스, https://gloomystudy.tistory.com/40
108. [C++] 다형성(Polymorphism) - Dandi's Devlog, 8월 3, 2025에 액세스, https://choi-dan-di.github.io/cpp/polymorphism/
109. [C++]함수 템플릿, 클래스 템플릿 - Junk의 티스토리, 8월 3, 2025에 액세스, [https://junk-s.tistory.com/entry/C%ED%95%A8%EC%88%98-%ED%85%9C%ED%94%8C%EB%A6%BF-%ED%81%B4%EB%9E%98%EC%8A%A4-%ED%85%9C%ED%94%8C%EB%A6%BF](https://junk-s.tistory.com/entry/C함수-템플릿-클래스-템플릿)
110. [C++] 함수 템플릿(function template) - velog, 8월 3, 2025에 액세스, [https://velog.io/@octo__/C-%ED%95%A8%EC%88%98-%ED%85%9C%ED%94%8C%EB%A6%BFfunction-template](https://velog.io/@octo__/C-함수-템플릿function-template)
111. [C++] 템플릿 (Template) - 코딩이랑 이것저것 - 티스토리, 8월 3, 2025에 액세스, https://wn42.tistory.com/115
112. C++ - Template 템플릿, 동작과정, typename과 class 선언의 차이 - 공부 - 티스토리, 8월 3, 2025에 액세스, https://dlemrcnd.tistory.com/78
113. [C++] Class Template | 클래스 템플릿 - Archive - 티스토리, 8월 3, 2025에 액세스, https://dad-rock.tistory.com/175
114. [C++] 예외 처리 (Exception Handling) try, catch ,throw - 코딩팩토리 - 티스토리, 8월 3, 2025에 액세스, https://coding-factory.tistory.com/706
115. [ C++ ] 예외처리 메커니즘( try, catch, throw ) 총 정리 - 청포도 에이드, 8월 3, 2025에 액세스, [https://musket-ade.tistory.com/entry/C-%EC%98%88%EC%99%B8%EC%B2%98%EB%A6%AC-%EB%A9%94%EC%BB%A4%EB%8B%88%EC%A6%98-try-catch-throw-%EC%B4%9D-%EC%A0%95%EB%A6%AC](https://musket-ade.tistory.com/entry/C-예외처리-메커니즘-try-catch-throw-총-정리)
116. C++ ] 예외처리 try-catch와 throw - 개준생의 공부 일지 - 티스토리, 8월 3, 2025에 액세스, https://eteo.tistory.com/584
117. [C/C++ 프로그래밍 : 중급] 8. 예외 처리와 오류 처리, 8월 3, 2025에 액세스, https://gdngy.tistory.com/180
118. C++ fstream 객체를 통한 파일 입출력 - 냉정과 열정 사이, 8월 3, 2025에 액세스, https://psychoria.tistory.com/774
119. [C++] 파일입출력(ofstream, ifstream)에 대해서. - 가면 뒤의 기록 - 티스토리, 8월 3, 2025에 액세스, https://blockdmask.tistory.com/322
120. C++ 파일 입출력 정리 - Donghoon Note - 티스토리, 8월 3, 2025에 액세스, https://dhpark1212.tistory.com/entry/fopen-fread-fwrite-fstream-ifstream-ofstream
121. C++ 파일입출력 ( ifstream, ofstream ) - 성장하고 싶은 신입 개발자 - 티스토리, 8월 3, 2025에 액세스, https://dragondeok.tistory.com/49
122. 표준 템플릿 라이브러리 - 위키백과, 우리 모두의 백과사전, 8월 3, 2025에 액세스, [https://ko.wikipedia.org/wiki/%ED%91%9C%EC%A4%80_%ED%85%9C%ED%94%8C%EB%A6%BF_%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC](https://ko.wikipedia.org/wiki/표준_템플릿_라이브러리)
123. STL (Standard Template Library) - 새로시작하는 - 티스토리, 8월 3, 2025에 액세스, https://newclass.tistory.com/m/80
124. [C++] STL : Standard Template Library 1 -표준 템플릿 라이브러리란?, 8월 3, 2025에 액세스, https://shjz.tistory.com/13
125. C++ STL 정리 - 개요 - vel1024 - 티스토리, 8월 3, 2025에 액세스, https://sdy-study.tistory.com/59
126. [C++] STL의 개념, 구성요소, 특징, 장단점 - 깨작코딩, 8월 3, 2025에 액세스, [https://siloam72761.tistory.com/entry/C-STL%EC%9D%98-%EA%B0%9C%EB%85%90-%EA%B5%AC%EC%84%B1%EC%9A%94%EC%86%8C-%ED%8A%B9%EC%A7%95-%EC%9E%A5%EB%8B%A8%EC%A0%90](https://siloam72761.tistory.com/entry/C-STL의-개념-구성요소-특징-장단점)
127. STL 개요 | STL | 템플릿 | C++ 프로그래밍 - YouTube, 8월 3, 2025에 액세스, https://www.youtube.com/watch?v=AYYZQ9apCOk
128. [C++] STL vector 사용법 - 모징이의 개발 경험치 - 티스토리, 8월 3, 2025에 액세스, [https://mojing.tistory.com/entry/C-STL-vector-%EC%82%AC%EC%9A%A9%EB%B2%95](https://mojing.tistory.com/entry/C-STL-vector-사용법)
129. [C++] STL vector 사용법 & 예제 총정리 - 코딩팩토리, 8월 3, 2025에 액세스, https://coding-factory.tistory.com/596
130. C++ vector사용법 및 설명 (장&단점) - HwanShell - 티스토리, 8월 3, 2025에 액세스, https://hwan-shell.tistory.com/119
131. [C++][STL] Vector 기본 사용법 및 예제 활용 - 코딩젤리 - 티스토리, 8월 3, 2025에 액세스, https://life-with-coding.tistory.com/411
132. [C++ STL] Vector Container 사용 방법 & 관련 예제 총 정리 - nov.Zip, 8월 3, 2025에 액세스, [https://novlog.tistory.com/entry/C-STL-Vector-Container-%EC%82%AC%EC%9A%A9-%EB%B0%A9%EB%B2%95-%EA%B4%80%EB%A0%A8-%EC%98%88%EC%A0%9C-%EC%B4%9D-%EC%A0%95%EB%A6%AC-%EB%AF%B8%EC%99%84%EC%84%B1](https://novlog.tistory.com/entry/C-STL-Vector-Container-사용-방법-관련-예제-총-정리-미완성)
133. C++ STL 맵 기본 사용법과 예제 - Insert Brain Here - 티스토리, 8월 3, 2025에 액세스, https://hwan1402.tistory.com/88
134. [C++][STL] map 사용법 정리 - 코딩젤리 - 티스토리, 8월 3, 2025에 액세스, https://life-with-coding.tistory.com/305
135. C++ STL map 기본 사용법과 예제, 8월 3, 2025에 액세스, https://twpower.github.io/91-how-to-use-map-in-cpp
136. [C++] STL map Container 사용 방법 정리 (feat. map & [hash_map]unordered_map & multi_map) - nov.Zip, 8월 3, 2025에 액세스, [https://novlog.tistory.com/entry/C-STL-map-Container-%EC%82%AC%EC%9A%A9-%EB%B0%A9%EB%B2%95-%EC%A0%95%EB%A6%AC-feat-map-hashmapunorderedmap-multimap](https://novlog.tistory.com/entry/C-STL-map-Container-사용-방법-정리-feat-map-hashmapunorderedmap-multimap)
137. [C++ STL] map 사용법 - Joon - 티스토리, 8월 3, 2025에 액세스, https://kbj96.tistory.com/23
138. C++ 기본 제공 연산자, 우선 순위 및 결합성 | Microsoft Learn, 8월 3, 2025에 액세스, https://learn.microsoft.com/ko-kr/cpp/cpp/cpp-built-in-operators-precedence-and-associativity?view=msvc-170

