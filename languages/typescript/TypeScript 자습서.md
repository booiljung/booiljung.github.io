[TypeScript](./index.md)
# TypeScript 자습서



TypeScript는 Microsoft가 개발하고 유지 관리하는 오픈소스 프로그래밍 언어로, JavaScript의 상위 집합(superset)으로 정의된다.1 "상위 집합"이라는 용어는 TypeScript의 핵심 정체성을 나타낸다. 이는 TypeScript가 JavaScript를 대체하는 완전히 새로운 언어가 아니라, 기존 JavaScript의 모든 기능을 포함하면서 그 위에 추가적인 계층, 즉 정적 타입 시스템을 더한 언어임을 의미한다. 따라서 모든 유효한 JavaScript 코드는 문법적으로 유효한 TypeScript 코드이며, 이는 기존 JavaScript 프로젝트에 점진적으로 TypeScript를 도입할 수 있는 유연성을 제공한다.4

JavaScript는 `string`이나 `number`와 같은 원시 타입을 제공하지만, 변수에 일관되게 동일한 타입의 값이 할당되는지를 언어 차원에서 확인하지 않는다. 반면, TypeScript는 이 과정을 컴파일 시점에 확인하여 코드의 예기치 않은 동작을 줄이고 버그 발생 가능성을 낮춘다.3 이처럼 TypeScript는 JavaScript의 동적인 특성에 정적 타입 검사라는 강력한 도구를 결합하여 대규모 애플리케이션 개발의 안정성과 유지보수성을 향상시키는 것을 목표로 한다.


TypeScript의 등장은 JavaScript 생태계의 발전과 밀접한 관련이 있다. JavaScript는 웹 페이지에 간단한 상호작용을 추가하는 작은 스크립팅 언어로 시작했지만, 점차 프론트엔드와 백엔드를 아우르는 대규모 애플리케이션 개발의 핵심 언어로 성장했다.7 그러나 프로그램의 규모와 복잡성이 기하급수적으로 증가하는 동안, 코드 단위 간의 관계를 명확하게 표현하는 JavaScript의 능력은 그에 미치지 못했다.6

프로그래머가 작성하는 가장 흔한 종류의 오류는 타입 오류, 즉 특정 종류의 값이 예상과 다른 종류의 값이 필요한 곳에 사용될 때 발생한다.6 이러한 오류는 단순한 오타, 라이브러리 API에 대한 오해, 런타임 동작에 대한 잘못된 가정 등 다양한 원인으로 발생할 수 있다. TypeScript의 핵심 목표는 이러한 문제를 해결하기 위해 JavaScript 프로그램을 위한 

**정적 타입 검사기(static typechecker)** 역할을 하는 것이다.7 즉, 코드가 실행되기 전(정적)에 프로그램의 타입이 올바른지 확인(타입 검사)하는 도구로서 기능한다.6

이러한 설계 철학의 중요한 결과 중 하나는 타입 시스템이 컴파일 시점에만 존재하고, 최종적으로 생성되는 JavaScript 코드에서는 모든 타입 관련 구문이 제거된다는 점이다.1 이는 TypeScript가 JavaScript의 런타임 동작을 절대 변경하지 않음을 보장한다. 타입 시스템은 개발 단계에서만 오류를 검출하는 도구이며, 런타임에는 어떠한 성능 저하도 유발하지 않는 '제로 코스트 추상화(zero-cost abstraction)'를 제공한다. 그러나 이 설계는 중요한 함의를 가진다. TypeScript의 타입 검사는 개발 환경과 컴파일 과정에 국한되므로, API 입력값 유효성 검사와 같은 런타임 시점의 데이터 검증은 별도의 라이브러리(예: Zod, Yup)를 통해 반드시 수행해야 한다. TypeScript 타입만으로 런타임 안전성이 보장된다고 가정하는 것은 심각한 오류로 이어질 수 있다.


TypeScript는 JavaScript의 동적 타이핑(dynamic typing)과 약한 타이핑(weakly typed)이라는 본질적인 한계를 보완하기 위해 설계되었다.1 JavaScript에서는 변수의 타입이 런타임에 결정되고 쉽게 변경될 수 있어, 대규모 프로젝트에서는 예측 불가능한 오류의 원인이 되기도 한다. 예를 들어, JavaScript에서는 숫자와 문자열을 더하면 오류 없이 문자열로 자동 변환되어 연결되지만(`5 + "10"`은 `15`가 아닌 `"510"`이 된다), TypeScript에서는 이러한 타입 불일치를 컴파일 시점에 오류로 감지하여 버그를 사전에 방지한다.6

TypeScript의 가장 큰 장점은 정적 타입 정보가 개발 도구(IDE)의 기능을 극대화한다는 점이다.10 Visual Studio Code와 같은 최신 편집기는 TypeScript의 타입 정보를 활용하여 강력한 코드 자동 완성, 안전한 리팩토링, 실시간 오류 강조, 변수 정의로 즉시 이동하는 기능 등을 제공한다.10 이는 개발자의 생산성을 크게 향상시키고, 특히 여러 개발자가 협업하는 대규모 프로젝트에서 코드의 가독성과 유지보수성을 획기적으로 개선한다.4

이러한 개발자 경험의 향상은 TypeScript 생태계의 선순환 구조를 만들었다. 향상된 도구와 안정성은 더 많은 프로젝트와 개발팀이 TypeScript를 채택하도록 유도했다. 사용자 기반이 확대되자 기존 JavaScript 라이브러리 제작자들은 TypeScript 타입 정의 파일(`@types` 패키지)을 제공하기 시작했고, 이는 다시 TypeScript를 기존 프로젝트에 통합하는 것을 더욱 용이하게 만들어 채택을 가속화하는 결과를 낳았다. 이처럼 TypeScript의 성공은 단순히 기술적인 우위를 넘어, JavaScript 생태계와의 공생 관계 및 강력한 도구 지원을 통해 개발자 커뮤니티의 지지를 얻어낸 결과라고 분석할 수 있다.



TypeScript 개발을 시작하기 위해 먼저 TypeScript 컴파일러(`tsc`)를 설치해야 한다. 설치 방법은 크게 전역(global) 설치와 프로젝트별(local) 설치로 나뉜다.

- **전역 설치**: 간단한 테스트나 학습 목적으로 시스템 전반에서 `tsc` 명령어를 사용하고 싶을 때 유용하다. `npm`을 통해 다음 명령어로 설치할 수 있다.10

  ```Bash
  npm install -g typescript
  ```
  
- **프로젝트별 설치**: 팀 프로젝트나 실제 애플리케이션 개발에서는 이 방식이 강력히 권장된다.14 프로젝트마다 특정 버전의 TypeScript를 종속성으로 관리하여 모든 팀원이 동일한 환경에서 개발하고, 일관성 있고 재현 가능한 빌드(reproducible build)를 보장할 수 있다.14

  ```Bash
  npm install typescript --save-dev
  ```

프로젝트별로 설치한 경우, `tsc` 명령어는 `npx`를 통해 실행하는 것이 일반적이다.10

```Bash
npx tsc
```

가장 기본적인 컴파일러 사용법은 특정 TypeScript 파일(`.ts`)을 JavaScript 파일(`.js`)로 변환하는 것이다.

```Bash
npx tsc main.ts
```

이 명령을 실행하면 `main.ts` 파일과 동일한 디렉터리에 `main.js` 파일이 생성된다.8


실제 프로젝트에서는 `tsconfig.json`이라는 설정 파일을 통해 컴파일러의 동작을 체계적으로 관리한다. 이 파일은 TypeScript 프로젝트의 루트 디렉터리에 위치하며, `npx tsc --init` 명령어로 기본 설정 파일을 생성할 수 있다.15

`tsconfig.json` 파일은 TypeScript의 두 가지 핵심 역할, 즉 **타입 검사기**와 **트랜스파일러**의 설정을 모두 담고 있다. 이 두 역할을 이해하면 각 옵션의 목적을 명확히 파악할 수 있다.


이 옵션들은 TypeScript가 코드를 얼마나 엄격하게 검사할지를 결정한다.

- `strict`: `true`로 설정하는 것을 강력히 권장한다. 이 옵션은 `noImplicitAny`, `strictNullChecks` 등 모든 엄격한 타입 검사 규칙을 활성화하여 TypeScript의 안정성 기능을 최대한 활용하게 한다.15
- `noImplicitAny`: 타입이 명시되지 않은 변수나 매개변수를 TypeScript가 `any` 타입으로 추론하는 것을 방지한다. `true`로 설정하면 모든 변수에 명시적인 타입을 요구하여 코드의 명확성을 높인다.
- `strictNullChecks`: `null`과 `undefined`를 모든 타입에서 허용하지 않고, 해당 값을 사용하려면 명시적으로 유니언 타입(`string | null`)을 선언하도록 강제한다. 이는 런타임에 `null` 또는 `undefined` 관련 오류를 방지하는 데 매우 중요한 옵션이다.16


이 옵션들은 생성될 JavaScript 코드의 형식과 대상 환경을 제어한다.

- `target`: 생성될 JavaScript의 ECMAScript 버전을 지정한다. 예를 들어, `"ES2020"`으로 설정하면 해당 버전에 맞는 문법으로 코드가 변환된다.15 오래된 브라우저를 지원해야 한다면 

  `"ES5"`와 같이 낮은 버전을 선택할 수 있다.

- `module`: 생성될 JavaScript 파일이 사용할 모듈 시스템을 지정한다. Node.js 환경에서는 주로 `"CommonJS"`를 사용하고, 최신 브라우저나 Node.js 환경에서는 `"ES2020"` 또는 `"NodeNext"` 등을 사용한다.15

- `outDir`: 컴파일된 JavaScript 파일들이 저장될 출력 디렉터리를 지정한다. 일반적으로 `"./dist"`로 설정하여 소스 코드와 빌드 결과물을 분리한다.15

- `rootDir`: TypeScript 소스 파일들이 위치한 루트 디렉터리를 지정한다. 보통 `"./src"`로 설정한다.15

- `esModuleInterop`: `true`로 설정하면 CommonJS 모듈과 ES 모듈 간의 호환성을 높여, `import React from 'react'`와 같은 구문을 문제없이 사용할 수 있게 해준다.15

이처럼 `tsconfig.json`을 통해 프로젝트의 요구사항에 맞게 타입 검사의 강도와 출력될 JavaScript의 형태를 정밀하게 제어할 수 있다. 현대적인 TypeScript 개발에서는 `strict: true`를 기본으로 설정하는 것이 표준적인 관행이다.


Node.js 환경에서 TypeScript를 사용하는 것은 서버사이드 애플리케이션의 안정성을 높이는 효과적인 방법이다. Express 프레임워크와 함께 TypeScript 프로젝트를 설정하는 과정은 다음과 같다.

1. 프로젝트 초기화 및 의존성 설치:

   먼저 프로젝트 디렉터리를 만들고 npm 프로젝트로 초기화한다.

   ```Bash
   mkdir ts-node-express && cd ts-node-express
   npm init -y
   ```
   
   다음으로, `express`와 같은 런타임 의존성과 TypeScript 개발에 필요한 개발 의존성을 설치한다.15

   ```Bash
   # 런타임 의존성
   npm install express dotenv
   
   # 개발 의존성
   npm install -D typescript ts-node @types/node @types/express nodemon
   ```
   
   각 개발 의존성의 역할은 다음과 같다 15:
   
   - `typescript`: TypeScript 컴파일러.
   - `ts-node`: TypeScript 파일을 JavaScript로 미리 컴파일하지 않고 직접 실행할 수 있게 해주는 도구로, 개발 시 유용하다.
   - `@types/node`, `@types/express`: Node.js와 Express 프레임워크에 대한 타입 정의 파일이다. 이 패키지들은 원래 JavaScript로 작성된 라이브러리에 TypeScript 타입 정보를 제공하여, 해당 라이브러리의 함수와 객체에 대해 타입 검사와 자동 완성을 가능하게 한다. 이는 TypeScript 생태계의 핵심적인 부분이다.
   - `nodemon`: 개발 중에 파일이 변경될 때마다 서버를 자동으로 재시작해주는 도구이다.
   
2. tsconfig.json 생성 및 설정:

   npx tsc --init 명령어로 tsconfig.json 파일을 생성하고, 앞서 설명한 주요 옵션들(target, module, strict, outDir 등)을 프로젝트 환경에 맞게 설정한다.15

3. package.json 스크립트 작성:

   개발 및 빌드, 실행 과정을 자동화하기 위해 package.json 파일의 scripts 섹션을 다음과 같이 구성한다.17

   ```JSON
   "scripts": {
     "build": "npx tsc",
     "start": "node dist/index.js",
     "dev": "nodemon src/index.ts"
   }
   ```
   
   - `build`: `tsc`를 실행하여 `src` 디렉터리의 모든 `.ts` 파일을 컴파일하고 `dist` 디렉터리에 결과물을 저장한다.
   - `start`: 컴파일된 JavaScript 파일을 Node.js로 실행하여 프로덕션 서버를 구동한다.
   - `dev`: `nodemon`과 `ts-node`를 함께 사용하여, 소스 코드 변경 시 자동으로 서버를 재시작하며 개발을 진행한다.

이러한 설정을 통해 타입 안정성을 갖춘 Node.js 및 Express 애플리케이션을 효율적으로 개발하고 배포할 수 있는 기반이 마련된다.



TypeScript는 JavaScript의 기본 자료형에 해당하는 핵심적인 기본 타입(primitive types)을 제공한다. 이 타입들의 이름은 JavaScript의 `typeof` 연산자가 반환하는 문자열과 동일하다.

- `string`: `"Hello, world"`와 같은 문자열 값을 나타낸다.18
- `number`: `42`와 같은 숫자 값을 나타낸다. JavaScript에는 정수와 실수를 구분하는 별도의 타입이 없으므로, 모든 숫자는 `number` 타입으로 처리된다.18
- `boolean`: `true`와 `false` 두 가지 값을 나타낸다.18

타입을 명시할 때는 항상 소문자 `string`, `number`, `boolean`을 사용해야 한다. 대문자로 시작하는 `String`, `Number`, `Boolean`은 JavaScript의 특수한 내장 객체 타입을 참조하므로, 일반적인 타입 표기에는 사용하지 않는 것이 원칙이다.18

변수를 선언할 때 타입을 명시적으로 지정할 수 있다.

```TypeScript
let myName: string = "Alice";
```

하지만 대부분의 경우, TypeScript는 변수 초기화 시 할당된 값을 바탕으로 타입을 자동으로 추론(inference)하므로 타입 표기를 생략할 수 있다.

```TypeScript
let myName = "Alice"; // TypeScript는 myName을 'string' 타입으로 추론한다.
```

TypeScript의 타입 시스템에서 중요한 개념 중 하나는 **리터럴 타입(literal type)**이다. `let`으로 선언된 변수는 값이 변경될 수 있으므로 일반적인 기본 타입으로 추론되지만, `const`로 선언된 변수는 재할당이 불가능하므로 더 구체적인 리터럴 타입으로 추론된다.18

```TypeScript
let changingString = "Hello World"; // 타입: string
changingString = "Olá Mundo"; // 가능

const constantString = "Hello World"; // 타입: "Hello World"
// constantString = "Olá Mundo"; // 오류: "Hello World" 타입에 "Olá Mundo"를 할당할 수 없다.
```

`constantString`의 타입은 일반적인 `string`이 아니라, 값 자체가 타입이 되는 `"Hello World"`라는 리터럴 타입이다. 이 리터럴 타입은 유니언 타입과 결합될 때 강력한 기능을 제공한다. 예를 들어, 특정 문자열 값들만 허용하는 타입을 만들 수 있다.3

```TypeScript
type WindowStates = "open" | "closed" | "minimized";

function setWindowState(state: WindowStates) {
  //...
}

setWindowState("open"); // 가능
setWindowState("closed"); // 가능
// setWindowState("maximized"); // 오류: "maximized"는 WindowStates 타입에 할당할 수 없다.
```

이처럼 리터럴 타입은 API의 매개변수를 특정 값들로 제한하여 코드의 안정성과 가독성을 높이는 데 매우 유용하게 사용된다.


데이터를 모델링할 때, 개별 값뿐만 아니라 값의 집합이나 구조를 표현하는 것이 중요하다. TypeScript는 이를 위해 배열, 튜플, 열거형과 같은 데이터 구조 타입을 제공한다. 이들을 올바르게 선택하고 사용하는 것은 데이터의 의도를 명확히 표현하고 코드의 안정성을 높이는 핵심적인 설계 결정이다.


배열은 동일한 타입의 요소들을 순서대로 나열한 컬렉션이다. 주로 길이가 정해져 있지 않은 동적인 목록을 표현하는 데 사용된다. TypeScript에서 배열 타입은 두 가지 방식으로 선언할 수 있다.19

1. `타입` 구문:

   ```TypeScript
   let list: number = ;
   ```
   
2. 제네릭 `Array<타입>` 구문:

   ```TypeScript
   let list: Array<number> = ;
   ```

두 방식은 기능적으로 동일하며, `타입` 구문이 더 일반적으로 사용된다. TypeScript 배열은 JavaScript 배열의 모든 표준 메서드(`map`, `filter`, `push` 등)를 그대로 사용할 수 있으며, 타입 시스템이 요소의 타입을 검사하여 안정성을 보장한다.22


튜플은 길이가 고정되고 각 인덱스의 타입이 정해져 있는 배열이다. 서로 다른 타입의 값들을 정해진 순서에 따라 묶어서 표현할 때 유용하다. 이는 위치에 의미가 부여된 이종(heterogeneous) 데이터 구조를 가볍게 표현하는 방법이다.8

```TypeScript
// [string, number] 타입의 튜플 선언
let x: [string, number];

// 초기화
x = ["hello", 10]; // 정상

// 잘못된 초기화
// x = [10, "hello"]; // 오류: number는 string에, string은 number에 할당할 수 없다.
```

튜플은 배열과 달리 길이와 순서가 타입에 의해 보장된다. 따라서 정해진 인덱스 밖의 요소에 접근하려고 하면 컴파일 오류가 발생한다.24 예를 들어, `x`는 `string` 타입의 메서드를, `x`은 `number` 타입의 메서드를 안전하게 사용할 수 있다.


열거형은 TypeScript 고유의 기능으로, 관련된 상수 값들의 집합에 친숙한 이름을 부여하는 방법이다.19 "매직 넘버"나 "매직 스트링"을 코드에서 제거하고, 의도를 명확하게 표현하는 데 사용된다.

- **숫자 열거형 (Numeric Enum)**: 기본적으로 0부터 시작하여 1씩 증가하는 숫자 값을 가진다. 초기값을 지정하여 시작 번호를 변경할 수도 있다.25

  ```TypeScript
enum Direction {
    Up = 1,
    Down, // 2
    Left,  // 3
    Right, // 4
  }
  let dir: Direction = Direction.Left; // dir의 값은 3
  ```
  
  숫자 열거형의 독특한 특징은 **역방향 매핑(reverse mapping)**을 지원한다는 것이다. 값(value)을 통해 이름(key)을 조회할 수 있다 (`Direction`는 `"Left"`를 반환한다).25

- **문자열 열거형 (String Enum)**: 각 멤버를 명시적인 문자열 값으로 초기화해야 한다. 자동 증가 기능은 없지만, 런타임에 디버깅할 때 숫자보다 훨씬 명확하고 직관적인 값을 제공한다는 장점이 있다.25

  ```TypeScript
  enum LogLevel {
    Error = "ERROR",
    Warn = "WARN",
    Info = "INFO",
  }
  ```
  
  문자열 열거형은 역방향 매핑을 생성하지 않는다.25

이 세 가지 데이터 구조 타입은 각각 다른 목적을 가진다. 동질적인 목록에는 **배열**을, 위치에 의미가 있는 고정된 구조에는 **튜플**을, 명명된 상수 집합에는 **열거형**을 사용하는 것이 적절한 설계 방식이다.


TypeScript에는 일반적인 데이터 타입 외에 특별한 의미를 가지는 타입들이 존재한다. 이들은 타입 시스템의 경계를 다루거나 특정 상황을 명확하게 표현하는 데 사용된다.


`any` 타입은 TypeScript의 타입 검사를 완전히 비활성화하는 "탈출구"와 같다.18

`any`로 선언된 변수에는 어떤 타입의 값이든 할당할 수 있으며, 해당 변수의 속성에 접근하거나 함수처럼 호출하는 등 모든 작업이 컴파일 시점에 오류 없이 허용된다.27 이는 기존 JavaScript 코드와의 점진적인 통합을 위해 필요하지만, 타입 안정성을 포기하는 대가이므로 꼭 필요한 경우가 아니면 사용을 지양해야 한다.


`unknown`은 `any`의 타입 안전(type-safe) 버전이다.28

`any`와 마찬가지로 어떤 타입의 값이든 `unknown` 타입 변수에 할당할 수 있다. 하지만 `unknown` 타입의 변수를 사용하려면, 반드시 타입 가드(type guard)나 타입 단언(type assertion)을 통해 타입을 좁혀야(narrowing) 한다.29 이는 알 수 없는 타입의 값을 다룰 때 발생할 수 있는 런타임 오류를 컴파일 시점에 방지해준다.

```TypeScript
function processValue(value: unknown) {
  // value.toUpperCase(); // 오류: 'value'는 'unknown' 타입이다.

  if (typeof value === 'string') {
    // 이 블록 안에서 value는 'string' 타입으로 좁혀진다.
    console.log(value.toUpperCase()); // 정상
  }
}
```


`void`는 함수가 아무 값도 반환하지 않음을 명시적으로 나타내는 타입이다.19

`void`를 반환 타입으로 갖는 함수는 내부적으로 값을 반환할 수 있지만, 그 값은 외부에서 관찰되거나 사용되지 않음을 의미한다.

```TypeScript
function logMessage(message: string): void {
  console.log(message);
  // return 문이 없거나, return; 처럼 값을 반환하지 않는다.
}
```


`never` 타입은 절대 발생할 수 없는 값의 타입을 나타낸다.19 주로 다음과 같은 두 가지 경우에 사용된다.

1. 항상 예외를 던지는 함수.31
2. 무한 루프에 빠져 절대 반환하지 않는 함수.31

```TypeScript
function throwError(message: string): never {
  throw new Error(message);
}

function infiniteLoop(): never {
  while (true) {}
}
```

`never`는 모든 타입의 하위 타입(subtype)이므로 어떤 타입의 변수에도 할당할 수 있지만, `never` 자신을 제외한 어떤 타입도 `never` 타입 변수에 할당할 수 없다. 이 특성은 타입 시스템에서 모든 가능성을 소거했음을 보장하는 **철저한 검사(exhaustive checks)** 패턴에 유용하게 사용된다.32


| 기능                 | `any`                                                  | `unknown`                                                    | 설명                                                         |
| -------------------- | ------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **값 할당**          | 모든 타입의 값을 할당할 수 있다.                       | 모든 타입의 값을 할당할 수 있다.                             | 두 타입 모두 어떤 값이든 받을 수 있다.                       |
| **속성 접근**        | 타입 검사 없이 모든 속성에 접근 가능하다.              | 타입 검사 없이는 속성에 접근할 수 없다.                      | `unknown`은 사용 전에 타입을 좁혀야 한다.                    |
| **함수 호출**        | 타입 검사 없이 함수처럼 호출 가능하다.                 | 타입 검사 없이는 함수처럼 호출할 수 없다.                    | `unknown`은 `typeof value === 'function'`과 같은 검사가 필요하다. |
| **다른 타입에 할당** | `string` 등 다른 타입의 변수에 자유롭게 할당 가능하다. | `any` 또는 `unknown`을 제외한 다른 타입의 변수에 할당할 수 없다. | `unknown`은 타입 안전성을 유지하기 위해 할당을 제한한다.     |
| **핵심 철학**        | 타입 시스템의 검사를 의도적으로 회피한다.              | 알 수 없는 값을 타입 안전하게 표현한다.                      | `any`는 편리함을, `unknown`은 안전성을 우선한다.             |


JavaScript에서 `null`과 `undefined`는 값이 없음을 나타내는 두 가지 원시 값이다. TypeScript는 이 둘에 대해 명확한 의미론적 구분을 제안한다.

- `undefined`: 아직 초기화되지 않은 상태를 의미한다. 변수가 선언되었지만 값이 할당되지 않았을 때의 기본값이다.33
- `null`: 값이 의도적으로 비어 있음을 나타낸다. 개발자가 명시적으로 "값이 없음"을 표현하기 위해 사용한다.33

이러한 구분에도 불구하고, 실제 코드에서는 두 값을 유사하게 처리해야 하는 경우가 많다. 이 때 `== null` 비교 연산자를 사용하면 `null`과 `undefined`를 한 번에 확인할 수 있다. (`null == undefined`는 `true`이지만, 다른 falsy 값과는 `false`이다).33

```TypeScript
function processValue(value: string | null | undefined) {
  if (value!= null) {
    // 이 블록 안에서 value는 'string' 타입으로 확신할 수 있다.
    console.log(value.toUpperCase());
  }
}
```

TypeScript 개발에서 가장 중요한 컴파일러 옵션 중 하나는 `strictNullChecks`이다.16 이 옵션을 

`false`(기본값)로 두면, `null`과 `undefined`는 모든 타입의 값으로 간주되어 `number` 타입 변수에 `null`을 할당하는 것이 허용된다. 이는 JavaScript에서 흔히 발생하는 `Cannot read property 'x' of null`과 같은 런타임 오류의 주된 원인이 된다.

`strictNullChecks`를 `true`로 설정하면, `null`과 `undefined`는 각각 고유한 `null`과 `undefined` 타입을 가지게 되며, 다른 타입에 할당할 수 없게 된다.16

```TypeScript
// --strictNullChecks가 true일 때
let x: number;
x = 1;      // 정상
// x = null;   // 오류: 'null'은 'number' 타입에 할당할 수 없다.
// x = undefined; // 오류: 'undefined'는 'number' 타입에 할당할 수 없다.

let y: number | null;
y = 1;      // 정상
y = null;   // 정상
// y = undefined; // 오류: 'undefined'는 'number | null' 타입에 할당할 수 없다.
```

이 옵션을 활성화하면 개발자는 `null` 또는 `undefined`가 될 수 있는 모든 변수를 명시적으로 처리하도록 강제받게 된다. 이는 유니언 타입(`string | null`)과 타입 가드를 사용하여 코드의 안정성을 대폭 향상시킨다. 따라서 모든 현대 TypeScript 프로젝트에서는 `strictNullChecks`를 `true`로 설정하는 것이 표준으로 여겨진다.



TypeScript 타입 시스템의 강력함은 기존 타입들을 조합하여 새로운 타입을 만들어내는 능력에 있다. 유니언과 인터섹션은 가장 기본적인 타입 조합 도구이다.


유니언 타입은 여러 타입 중 하나가 될 수 있는 값을 나타낸다. 수직 막대(`|`) 기호를 사용하여 타입을 연결한다.3

```TypeScript
function printId(id: number | string) {
  console.log("Your ID is: " + id);
}

printId(101);       // 정상
printId("202");     // 정상
// printId({ myID: 22342 }); // 오류
```

유니언 타입의 변수를 다룰 때, TypeScript는 안정성을 위해 해당 유니언에 포함된 **모든 타입에 공통으로 존재하는 멤버에만** 접근을 허용한다.35 예를 들어 `string | number` 타입의 변수에는 `string` 타입에만 있는 `toUpperCase()` 메서드를 직접 호출할 수 없다. 특정 타입의 멤버에 접근하려면 타입 내로잉(narrowing) 과정이 필요하다.

식별 가능한 유니언 (Discriminating Unions)

이는 유니언 타입을 안전하고 효과적으로 다루기 위한 강력한 패턴이다. 유니언에 속한 모든 타입이 공통된 리터럴 타입 속성(구분자, discriminant)을 가지도록 설계하는 것이다.35

```TypeScript
type NetworkLoadingState = { state: "loading"; };
type NetworkFailedState = { state: "failed"; code: number; };
type NetworkSuccessState = { state: "success"; response: { title: string; }; };

type NetworkState = NetworkLoadingState | NetworkFailedState | NetworkSuccessState;

function handleState(state: NetworkState) {
  switch (state.state) {
    case "loading":
      console.log("Loading...");
      break;
    case "failed":
      console.error(`Error code: ${state.code}`); // state는 NetworkFailedState로 좁혀짐
      break;
    case "success":
      console.log(`Success: ${state.response.title}`); // state는 NetworkSuccessState로 좁혀짐
      break;
  }
}
```

위 예제에서 `state` 속성은 `"loading"`, `"failed"`, `"success"`라는 리터럴 타입을 가지며, 이 값을 기준으로 `switch` 문에서 타입을 좁힐 수 있다. 이를 통해 각 `case` 블록 내에서 해당 타입에만 존재하는 속성(`code` 또는 `response`)에 안전하게 접근할 수 있다.


인터섹션 타입은 여러 타입을 하나로 결합하여, 결합된 모든 타입의 기능을 갖는 단일 타입을 생성한다. 앰퍼샌드(`&`) 기호를 사용하여 타입을 연결한다.35

```TypeScript
interface Person {
  name: string;
  age: number;
}

interface Serializable {
  serialize(): string;
}

type SerializablePerson = Person & Serializable;

const person: SerializablePerson = {
  name: "Alice",
  age: 30,
  serialize: () => {
    return JSON.stringify({ name: "Alice", age: 30 });
  }
};
```

`SerializablePerson` 타입은 `Person`의 `name`과 `age` 속성, 그리고 `Serializable`의 `serialize` 메서드를 모두 가져야 한다. 이처럼 인터섹션은 여러 계약(contract)이나 특징을 하나의 객체에 모두 적용해야 할 때 유용하다.36


TypeScript에서 객체의 형태를 정의하는 데는 `type` 별칭과 `interface` 두 가지 주요 방법이 있다.18 둘은 많은 경우 서로 대체 가능하지만, 코드의 유지보수성, 확장성, 그리고 컴파일러의 동작에 영향을 미치는 중요한 차이점을 가진다.


1. **표현 가능한 타입의 범위**:

   - `interface`: 주로 객체의 형태(shape)를 정의하는 데 사용된다. 클래스가 구현해야 할 계약을 명시하는 데 특화되어 있다.

   - `type`: 객체 형태뿐만 아니라, 원시 타입, 유니언, 튜플 등 거의 모든 종류의 타입에 새로운 이름을 부여할 수 있어 훨씬 유연하다.38

     ```TypeScript
     type StringOrNumber = string | number; // interface로는 불가능
     type PointTuple = [number, number];   // interface로는 불가능
     ```
   
2. **확장성 (Extensibility)**:

   - `interface`: `extends` 키워드를 사용하여 다른 인터페이스를 상속받아 확장한다. 이는 객체 지향 프로그래밍의 상속 개념과 유사하여 직관적이다.38

     ```TypeScript
     interface Animal { name: string; }
     interface Bear extends Animal { honey: boolean; }
     ```
     
   - `type`: 인터섹션(`&`)을 사용하여 기존 타입을 확장한다. 기능적으로는 유사하지만, 구문상으로는 조합의 의미가 더 강하다.40

     ```TypeScript
     type Animal = { name: string; };
     type Bear = Animal & { honey: boolean; };
     ```
   
3. **선언 병합 (Declaration Merging)**:

   - `interface`: 동일한 이름으로 여러 번 선언될 경우, 모든 선언이 자동으로 하나의 인터페이스로 병합된다. 이는 특히 서드파티 라이브러리의 기존 타입 정의를 확장(augment)할 때 매우 유용한 기능이다.38

     ```TypeScript
     interface Window { title: string; }
     interface Window { ts: any; }
     // Window 인터페이스는 이제 title과 ts 속성을 모두 가진다.
     ```
     
   - `type`: 선언 병합을 지원하지 않는다. 동일한 이름으로 `type`을 두 번 선언하면 중복 식별자 오류가 발생한다.41 이 특성은 타입이 예기치 않게 변경되지 않음을 보장하여 코드의 명확성을 높이는 장점이 될 수도 있다.


`interface`와 `type` 사이의 선택은 단순한 스타일의 문제가 아니라 설계상의 결정이다. 커뮤니티와 TypeScript 팀 사이에서도 의견이 나뉘지만 42, 다음과 같은 실용적인 전략을 따르는 것이 좋다.

- **`interface`를 사용해야 할 때**:
  - 객체나 클래스의 형태를 정의할 때.
  - 라이브러리나 프레임워크의 공개 API를 정의할 때. `extends`를 통한 명확한 상속 구조와 선언 병합을 통한 확장 가능성이 중요하기 때문이다.
- **`type`을 사용해야 할 때**:
  - 유니언, 튜플, 원시 타입 등 객체가 아닌 타입에 별칭을 붙일 때.
  - 유틸리티 타입, 조건부 타입 등 복잡한 타입 조작이 필요할 때.

결론적으로, **"객체의 형태를 정의할 때는 `interface`를 우선적으로 사용하고, 그 외 모든 경우에는 `type`을 사용한다"**는 규칙은 대부분의 상황에서 명확하고 일관된 코드를 작성하는 데 도움이 되는 효과적인 지침이다.


| 기능                           | `interface`                     | `type` 별칭                                 |
| ------------------------------ | ------------------------------- | ------------------------------------------- |
| **객체 형태 정의**             | 가능                            | 가능                                        |
| **유니언/튜플/원시 타입 정의** | 불가능                          | 가능                                        |
| **확장 구문**                  | `extends`                       | `&` (인터섹션)                              |
| **선언 병합**                  | 가능 (자동 병합)                | 불가능 (오류 발생)                          |
| **클래스 구현**                | `implements` 가능               | `implements` 가능 (단, 유니언 타입은 불가)  |
| **권장 사용 사례**             | 객체/클래스 형태 정의, 공개 API | 유니언, 튜플, 맵드 타입 등 복잡한 타입 정의 |


TypeScript는 일반적인 타입 변환을 용이하게 하기 위해 여러 가지 내장 유틸리티 타입을 제공한다. 이들은 전역적으로 사용 가능하며, 기존 타입을 재사용하여 간결하고 유지보수하기 좋은 코드를 작성하는 데 필수적이다.44

- `Partial<T>`: 타입 `T`의 모든 속성을 선택적(optional)으로 만든 새로운 타입을 구성한다. 객체의 일부 속성만 업데이트하는 함수를 작성할 때 유용하다.46

  ```TypeScript
  interface Todo { title: string; description: string; }
  
  function updateTodo(todo: Todo, fieldsToUpdate: Partial<Todo>) {
    return {...todo,...fieldsToUpdate };
  }
  ```
  
- `Readonly<T>`: 타입 `T`의 모든 속성을 읽기 전용(`readonly`)으로 만든다. 초기화 이후 객체의 속성이 변경되는 것을 방지하고자 할 때 사용한다.48

  ```TypeScript
  interface User { name: string; }
  const user: Readonly<User> = { name: "Alice" };
  // user.name = "Bob"; // 오류: 읽기 전용 속성이므로 'name'에 할당할 수 없다.
  ```
  
- `Pick<T, K>`: 타입 `T`에서 속성 `K`의 집합을 선택하여 새로운 타입을 구성한다. 특정 속성만 필요한 새로운 타입을 만들 때 사용한다.50

  ```TypeScript
  interface Todo { title: string; description: string; completed: boolean; }
  type TodoPreview = Pick<Todo, "title" | "completed">;
  // TodoPreview 타입은 { title: string; completed: boolean; }
  ```
  
- `Omit<T, K>`: 타입 `T`의 모든 속성에서 `K`를 제거한 새로운 타입을 구성한다. `Pick`의 반대 개념으로, 특정 속성을 제외하고 싶을 때 사용한다.52

  ```TypeScript
  interface Todo { title: string; description: string; completed: boolean; }
  type TodoWithoutDescription = Omit<Todo, "description">;
  // TodoWithoutDescription 타입은 { title: string; completed: boolean; }
  ```
  
- `Record<K, T>`: 속성 키가 `K`이고 속성 값이 `T`인 객체 타입을 구성한다. 정적인 키와 동적인 값을 갖는 객체, 즉 딕셔너리나 맵과 같은 구조를 타입으로 정의할 때 매우 유용하다.54

  ```TypeScript
  type Page = 'home' | 'about' | 'contact';
  interface PageInfo { title: string; }
  
  const pageInfo: Record<Page, PageInfo> = {
    home: { title: "Home" },
    about: { title: "About Us" },
    contact: { title: "Contact" }
  };
  ```


조건부 타입(Conditional Types)은 TypeScript의 타입 시스템 내에서 조건문 로직을 구현할 수 있게 해주는 고급 기능이다. JavaScript의 삼항 연산자와 유사한 구문을 사용하여, 타입 관계에 따라 두 가지 가능한 타입 중 하나를 선택한다.56

```TypeScript
SomeType extends OtherType? TrueType : FalseType;
```

이 구문은 `SomeType`이 `OtherType`에 할당 가능하면 `TrueType`을, 그렇지 않으면 `FalseType`을 결과로 반환한다. 조건부 타입은 제네릭과 결합될 때 그 진정한 힘을 발휘한다.

예를 들어, `Promise`의 내부 타입을 추출하는 `UnpackPromise` 유틸리티 타입을 만들어 보자. 이를 위해서는 `infer` 키워드가 필요하다. `infer`는 조건부 타입의 `extends` 절 내에서만 사용될 수 있으며, 추론될 타입을 위한 새로운 제네릭 타입 변수를 동적으로 도입한다.57

```TypeScript
type UnpackPromise<T> = T extends Promise<infer U>? U : T;

// 사용 예시
type StringPromise = Promise<string>;
type NumberPromise = Promise<number>;
type NotAPromise = boolean;

type UnpackedString = UnpackPromise<StringPromise>; // 타입: string
type UnpackedNumber = UnpackPromise<NumberPromise>; // 타입: number
type UnpackedBoolean = UnpackPromise<NotAPromise>;   // 타입: boolean
```

`UnpackPromise<T>`는 `T`가 `Promise<...>` 형태를 따르는지 확인한다. 만약 그렇다면, `infer U`를 통해 프로미스가 감싸고 있는 내부 타입(`U`)을 추론하여 반환한다. 그렇지 않다면, 원래 타입 `T`를 그대로 반환한다.

또한, 조건부 타입이 제네릭 유니언 타입에 적용될 때는 **분배적 조건부 타입(distributive conditional types)**으로 동작한다. 이는 조건이 유니언의 각 멤버에 개별적으로 적용됨을 의미한다.56

```TypeScript
type ToArray<T> = T extends any? T : never;

type StringOrNumberArray = ToArray<string | number>; // 타입: string | number
```

`ToArray<string | number>`는 `ToArray<string> | ToArray<number>`로 분배되어, 최종적으로 `string | number`라는 유니언 타입을 생성한다. 이처럼 조건부 타입과 `infer`는 타입 수준에서 정교한 메타프로그래밍을 가능하게 하여, 매우 동적이고 지능적인 타입을 만들 수 있게 해준다.



TypeScript에서 함수는 JavaScript와 마찬가지로 일급 시민(first-class citizen)이며, 타입 시스템 내에서도 정교하게 다룰 수 있다. 함수의 타입을 정의하는 것은 API의 계약을 명확히 하고, 잘못된 사용을 방지하는 데 핵심적인 역할을 한다.


함수의 매개변수와 반환 값에 타입을 명시적으로 지정할 수 있다. 이는 함수의 입력과 출력을 명확하게 문서화하는 효과를 가진다.58

```TypeScript
function add(x: number, y: number): number {
  return x + y;
}
```

반환 타입은 대부분의 경우 TypeScript가 `return` 문을 기반으로 추론할 수 있으므로 생략 가능하지만, 명시적으로 작성하는 것이 코드의 가독성을 높일 수 있다.58


JavaScript와 마찬가지로, TypeScript 함수도 선택적 매개변수(optional parameters)와 기본 매개변수(default parameters)를 지원한다.

- **선택적 매개변수**: 매개변수 이름 뒤에 `?`를 붙여 해당 매개변수가 선택적임을 나타낸다. 선택적 매개변수는 함수 호출 시 생략될 수 있으며, 생략될 경우 `undefined` 값을 가진다. 모든 선택적 매개변수는 필수 매개변수 뒤에 위치해야 한다.59

  ```TypeScript
  function greet(name: string, greeting?: string): string {
    if (greeting) {
      return `${greeting}, ${name}!`;
    }
    return `Hello, ${name}!`;
  }
  greet("Alice"); // "Hello, Alice!"
  greet("Bob", "Good morning"); // "Good morning, Bob!"
  ```
  
- **기본 매개변수**: 매개변수에 기본값을 할당하여, 해당 인자가 제공되지 않거나 `undefined`로 전달될 경우 기본값을 사용하도록 한다. 기본 매개변수도 선택적으로 간주된다.60

  ```TypeScript
  function log(message: string, level: string = 'info') {
    console.log(`[${level.toUpperCase()}] ${message}`);
  }
  log("Server started"); // "[INFO] Server started"
  log("Database connection failed", "error"); // " Database connection failed"
  ```


함수 자체의 타입을 정의할 수도 있다. 이는 콜백 함수나 고차 함수를 다룰 때 특히 유용하다. 함수 타입은 화살표 함수와 유사한 구문으로 표현된다.63

```TypeScript
type MathOperation = (x: number, y: number) => number;

const add: MathOperation = (a, b) => a + b;
const subtract: MathOperation = (a, b) => a - b;
```

`MathOperation`이라는 타입 별칭은 두 개의 `number`를 인자로 받아 `number`를 반환하는 함수의 시그니처를 정의한다. 이를 통해 `add`와 `subtract` 함수가 동일한 계약을 따르도록 강제할 수 있다.


TypeScript는 클래스 기반 객체 지향 프로그래밍을 완벽하게 지원한다. 클래스는 속성(데이터)과 메서드(동작)를 캡슐화하는 청사진이다.64


- **속성 (Fields/Properties)**: 클래스 인스턴스가 가질 데이터를 정의한다. 타입 표기를 통해 각 속성의 타입을 명시할 수 있다.64
- **생성자 (Constructor)**: `new` 키워드로 클래스의 인스턴스를 생성할 때 호출되는 특수 메서드다. 주로 속성을 초기화하는 역할을 한다.64
- **메서드 (Methods)**: 클래스 인스턴스가 수행할 수 있는 동작을 정의하는 함수다.

```TypeScript
class Greeter {
  // 속성
  greeting: string;

  // 생성자
  constructor(message: string) {
    this.greeting = message;
  }

  // 메서드
  greet() {
    return "Hello, " + this.greeting;
  }
}

let greeter = new Greeter("world");
```


TypeScript는 클래스 멤버의 가시성을 제어하기 위해 `public`, `private`, `protected` 세 가지 접근 제어자를 제공한다. 이는 캡슐화 원칙을 강제하여 코드의 안정성을 높인다.65

- `public`: 모든 곳에서 접근 가능하다. 접근 제어자를 명시하지 않으면 기본적으로 `public`으로 간주된다.66
- `private`: 해당 클래스 내부에서만 접근 가능하다. 클래스 외부나 상속받은 자식 클래스에서는 접근할 수 없다.66
- `protected`: 해당 클래스 내부와 해당 클래스를 상속받은 자식 클래스 내부에서 접근 가능하다. 클래스 외부에서는 접근할 수 없다.66

중요한 점은 이 접근 제어자들이 TypeScript의 컴파일 시점 기능이라는 것이다. 최종적으로 생성되는 JavaScript 코드에서는 이 제어자들이 모두 제거되어 모든 멤버가 `public`처럼 동작한다.69 따라서 이들은 런타임의 데이터 보안을 위한 것이 아니라, 개발 단계에서 의도된 클래스 사용법을 강제하고 실수를 방지하기 위한 도구이다. 런타임 수준의 강력한 비공개성을 원한다면 ECMAScript 표준인 `#private` 필드를 사용해야 하며, TypeScript도 이를 지원한다.65


| 접근 위치                  | `public` | `private` | `protected` |
| -------------------------- | -------- | --------- | ----------- |
| **클래스 내부**            | 가능     | 가능      | 가능        |
| **자식 클래스**            | 가능     | 불가능    | 가능        |
| **클래스 인스턴스 (외부)** | 가능     | 불가능    | 불가능      |


TypeScript 클래스는 전통적인 객체 지향 디자인 패턴을 구현하기 위한 다양한 기능을 제공한다.


`extends` 키워드를 사용하여 한 클래스가 다른 클래스(부모 클래스)의 속성과 메서드를 물려받을 수 있다. 이를 통해 코드 재사용성을 높이고 클래스 간의 계층 구조를 형성할 수 있다.65

자식 클래스의 생성자에서는 `this` 키워드를 사용하기 전에 반드시 `super()`를 호출하여 부모 클래스의 생성자를 실행해야 한다. 이는 부모 클래스의 초기화 로직이 먼저 수행되도록 보장하기 위함이다.64 또한 `super.methodName()` 구문을 사용하여 자식 클래스에서 부모 클래스의 메서드를 호출할 수 있다.70

```TypeScript
class Animal {
  move(distance: number = 0) {
    console.log(`Animal moved ${distance}m.`);
  }
}

class Dog extends Animal {
  bark() {
    console.log('Woof! Woof!');
  }
  
  move(distance: number = 5) {
    console.log("Dog is moving...");
    super.move(distance); // 부모 클래스의 move 메서드 호출
  }
}

const dog = new Dog();
dog.bark();
dog.move(10);
```


`readonly` 키워드를 속성 앞에 붙이면 해당 속성을 읽기 전용으로 만들 수 있다. `readonly` 속성은 선언 시점이나 생성자 내부에서만 값을 할당할 수 있으며, 그 이후에는 재할당이 불가능하다.72 이는 객체의 불변성(immutability)을 보장하는 데 도움이 되며, 컴파일 시점에만 적용되는 제약 조건이다.74

```TypeScript
class Octopus {
  readonly name: string;
  readonly numberOfLegs: number = 8;
  constructor (theName: string) {
    this.name = theName;
  }
}
let dad = new Octopus("Man with the 8 strong legs");
// dad.name = "Man with the 3-piece suit"; // 오류: 'name'은 읽기 전용 속성이다.
```


`static` 키워드로 선언된 속성이나 메서드는 클래스의 인스턴스가 아닌 클래스 자체에 속한다. 따라서 인스턴스를 생성하지 않고 `ClassName.memberName` 형태로 직접 접근할 수 있다.75 정적 멤버는 모든 인스턴스가 공유하는 유틸리티 함수나 상수를 정의하는 데 유용하다.

```TypeScript
class Grid {
  static origin = {x: 0, y: 0};
  
  calculateDistanceFromOrigin(point: {x: number; y: number;}) {
    let xDist = (point.x - Grid.origin.x);
    let yDist = (point.y - Grid.origin.y);
    return Math.sqrt(xDist * xDist + yDist * yDist);
  }
}
```


`abstract` 키워드로 선언된 클래스는 직접 인스턴스화할 수 없으며, 다른 클래스가 상속받기 위한 기반(base) 클래스로만 사용된다.77 추상 클래스는 구현부가 없는 `abstract` 메서드를 포함할 수 있으며, 이 메서드들은 추상 클래스를 상속받는 모든 자식 클래스에서 반드시 구현해야 한다. 이는 관련된 클래스들의 공통적인 구조와 계약을 강제하면서도, 각 클래스의 구체적인 구현은 자유롭게 남겨두는 강력한 객체 지향 패턴이다.

```TypeScript
abstract class Department {
  constructor(public name: string) {}

  printName(): void {
    console.log("Department name: " + this.name);
  }

  abstract printMeeting(): void; // 자식 클래스에서 반드시 구현해야 함
}

class AccountingDepartment extends Department {
  constructor() {
    super("Accounting and Auditing");
  }

  printMeeting(): void {
    console.log("The Accounting Department meets each Monday at 10am.");
  }
}

// const department = new Department(); // 오류: 추상 클래스는 인스턴스화할 수 없다.
const accounting = new AccountingDepartment();
accounting.printName();
accounting.printMeeting();
```



제네릭(Generics)은 단일 타입이 아닌 다양한 타입에서 작동할 수 있는 재사용 가능한 컴포넌트를 생성하기 위한 핵심 도구이다.80 제네릭을 사용하면 타입 정보를 변수처럼 다룰 수 있어, 코드의 재사용성과 타입 안정성이라는 두 가지 목표를 동시에 달성할 수 있다.

제네릭이 해결하는 문제를 이해하기 위해, 입력값을 그대로 반환하는 `identity` 함수를 생각해보자.

1. **특정 타입 사용**: `number` 타입만 처리하는 함수는 재사용성이 떨어진다.

   ```TypeScript
   function identity(arg: number): number {
     return arg;
   }
   ```
   
2. **`any` 타입 사용**: 모든 타입을 받을 수 있지만, 타입 안정성을 잃게 된다. 함수에 `string`을 넣어도 반환 타입을 `number`로 간주하는 등의 오류가 발생할 수 있다.82

   ```TypeScript
   function identity(arg: any): any {
     return arg;
   }
   ```
   
3. **제네릭 사용**: 제네릭은 타입 변수(일반적으로 `T`로 표기)를 사용하여 이 문제를 해결한다. `<T>`는 이 함수가 제네릭이며, `T`라는 타입 변수를 사용함을 선언한다. 이 타입 변수는 입력(`arg: T`)과 출력(`: T`)의 타입이 동일해야 한다는 관계를 설정한다.80

   ```
   function identity<T>(arg: T): T {
     return arg;
   }
   
   let output = identity<string>("myString"); // 타입: string
   let anotherOutput = identity(123);        // 타입: number (타입 추론)
   ```

이처럼 제네릭은 함수가 호출될 때 전달된 인자의 타입을 `T`에 담아, 그 타입 정보를 함수 전체에서 일관되게 유지한다. 이를 통해 하나의 함수 정의로 다양한 타입을 안전하게 처리할 수 있다.


제네릭은 함수뿐만 아니라 인터페이스와 클래스 등 다양한 구조에 적용될 수 있다.


앞서 본 `identity` 함수처럼, 함수의 시그니처에 타입 변수를 선언하여 구현한다. 여러 타입 변수를 사용할 수도 있다.63

```TypeScript
function map<Input, Output>(arr: Input, func: (arg: Input) => Output): Output {
  return arr.map(func);
}
const parsed = map(["1", "2", "3"], (n) => parseInt(n)); // parsed의 타입: number
```


인터페이스 정의에 타입 변수를 사용하여, 해당 인터페이스를 구현하는 객체의 멤버 타입을 동적으로 지정할 수 있다.84

```TypeScript
interface Box<T> {
  contents: T;
}

let stringBox: Box<string> = { contents: 'Hello' };
let numberBox: Box<number> = { contents: 100 };
```

`Box<T>` 인터페이스는 `contents` 속성의 타입을 `T`로 정의한다. `Box`를 사용할 때 `string`이나 `number`와 같은 구체적인 타입을 지정하여 다양한 종류의 "상자"를 만들 수 있다.


클래스도 제네릭을 통해 다양한 타입의 데이터를 처리하도록 만들 수 있다. 클래스 이름 뒤에 타입 변수를 선언하여 클래스 내부의 속성이나 메서드에서 사용한다.80

```TypeScript
class DataStorage<T> {
  private data: T =;

  addItem(item: T) {
    this.data.push(item);
  }

  getItems(): T {
    return [...this.data];
  }
}

const textStorage = new DataStorage<string>();
textStorage.addItem("Hello");
textStorage.addItem("World");

const numberStorage = new DataStorage<number>();
numberStorage.addItem(1);
numberStorage.addItem(2);
```

`DataStorage<T>` 클래스는 어떤 타입 `T`의 데이터든 저장하고 관리할 수 있는 재사용 가능한 자료 구조를 제공한다.


제네릭은 유연하지만, 때로는 타입 변수가 특정 구조나 속성을 가지도록 강제해야 할 필요가 있다. 예를 들어, 타입 `T`의 속성인 `.length`에 접근하는 함수를 작성하려면, `T`가 반드시 `.length` 속성을 가지고 있음을 컴파일러에 알려주어야 한다. 이때 **제네릭 제약 조건(Generic Constraints)**을 사용한다.80


`extends` 키워드를 사용하여 타입 변수가 특정 인터페이스나 타입을 만족해야 한다는 제약을 추가할 수 있다.86

```TypeScript
interface Lengthwise {
  length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
  console.log(arg.length); // 이제.length 속성에 안전하게 접근 가능
  return arg;
}

loggingIdentity("hello"); // string은 length 속성을 가짐
loggingIdentity(); // array는 length 속성을 가짐
// loggingIdentity(123); // 오류: number는 length 속성이 없다.
```

`T extends Lengthwise`는 "타입 `T`는 `length: number` 속성을 가진 어떤 타입이든 될 수 있다"는 계약을 명시한다. 이 제약 조건 덕분에 함수 본문에서 `arg.length`를 안전하게 사용할 수 있다.


`keyof` 연산자는 객체 타입의 모든 키를 문자열 리터럴 유니언 타입으로 반환한다. 이를 `extends`와 함께 사용하면, 제네릭 타입 변수가 다른 타입의 키 중 하나여야 한다는 제약을 걸 수 있다. 이는 객체의 속성에 안전하게 접근하는 함수를 만들 때 매우 유용하다.87

```TypeScript
function getProperty<T, K extends keyof T>(obj: T, key: K) {
  return obj[key];
}

let x = { a: 1, b: 2, c: 3, d: 4 };

getProperty(x, "a"); // 정상, 반환 타입은 number
// getProperty(x, "m"); // 오류: "m"은 { a, b, c, d }의 키가 아니다.
```

`K extends keyof T` 제약은 두 번째 매개변수 `key`가 반드시 첫 번째 매개변수 `obj`에 존재하는 키여야 함을 보장한다. 이를 통해 존재하지 않는 속성에 접근하려는 시도를 컴파일 시점에 차단할 수 있다. 제네릭 제약 조건은 제네릭의 유연성과 타입 시스템의 안정성 사이의 균형을 맞추는 핵심적인 메커니즘이다.



TypeScript는 최신 JavaScript 표준인 ES 모듈 시스템을 채택하여 코드를 모듈 단위로 구성하고 관리한다. 모듈 내에 선언된 변수, 함수, 클래스 등은 기본적으로 해당 모듈의 스코프에 한정되며, 외부에서 사용하려면 반드시 `export` 키워드를 통해 명시적으로 내보내야 한다.89

다른 모듈에서 내보낸 기능을 사용하기 위해서는 `import` 키워드를 사용한다.

```TypeScript
// @filename: maths.ts
export const PI = 3.14;
export function absolute(num: number) {
  return num < 0? -num : num;
}

// @filename: app.ts
import { PI, absolute } from "./maths.js";

console.log(PI);
const absPhi = absolute(-1.618);
```

TypeScript는 ES 모듈 구문에 타입 관련 기능을 추가했다. `import type`과 `export type`은 타입 선언만을 가져오거나 내보낼 때 사용된다.91

```TypeScript
// @filename: animal.ts
export type Cat = { breed: string; yearOfBirth: number };
export interface Dog { breeds: string; yearOfBirth: number; }

// @filename: app.ts
import type { Cat, Dog } from "./animal.js";
type Animals = Cat | Dog;
```

`import type`의 가장 큰 장점은 컴파일러에게 해당 `import`가 타입 정보만을 위한 것임을 명확히 알려준다는 것이다. 그 결과, 이 `import` 구문은 최종적으로 생성되는 JavaScript 코드에서 완전히 제거된다. 이는 타입 정보 때문에 불필요한 모듈 로드가 발생하는 것을 방지하여, 런타임 성능과 번들 크기를 최적화하는 데 도움이 된다.


ES 모듈은 두 가지 주요 내보내기 방식을 제공한다: **Named Export(이름 있는 내보내기)**와 **Default Export(기본 내보내기)**.

- **Named Export**: 한 모듈에서 여러 개의 변수, 함수, 클래스를 각각의 이름으로 내보낼 수 있다. 가져올 때는 반드시 중괄호(`{}`) 안에 내보낸 이름과 동일한 이름을 사용해야 한다.93

  ```TypeScript
  // export
  export const PI = 3.14;
  export class Circle { /*... */ }
  
  // import
  import { PI, Circle } from "./shapes.js";
  ```
  
- **Default Export**: 모듈당 단 하나의 값만 `default`로 내보낼 수 있다. 가져올 때는 중괄호 없이 원하는 이름으로 자유롭게 지정할 수 있다.93

  ```TypeScript
  // export
  export default class Square { /*... */ }
  
  // import
  import MySquare from "./square.js"; // Square를 MySquare라는 이름으로 가져옴
  ```

두 방식의 선택은 프로젝트의 일관성과 유지보수성에 큰 영향을 미친다. `Default Export`는 가져오는 쪽에서 이름을 마음대로 바꿀 수 있어, 코드베이스 전체에서 동일한 모듈이 다른 이름으로 사용되는 불일치를 유발할 수 있다. 이는 리팩토링과 코드 탐색을 어렵게 만드는 요인이 된다.95

반면, `Named Export`는 항상 동일한 이름을 사용하도록 강제하므로 명시적이고 예측 가능하다. IDE의 자동 완성 및 자동 `import` 기능도 `Named Export`에서 훨씬 안정적으로 동작하며, 사용하지 않는 코드를 제거하는 트리 쉐이킹(tree shaking)에도 더 효과적이다.95

이러한 이유로, **`Named Export`를 기본으로 사용하고, `Default Export`는 프레임워크의 규칙(예: React 컴포넌트)이나 명확하게 모듈의 주된 기능을 나타낼 때만 제한적으로 사용하는 것이 현대적인 TypeScript 개발의 모범 사례로 권장된다.**


| 기능                | Named Export                | Default Export                  |
| ------------------- | --------------------------- | ------------------------------- |
| **모듈당 개수**     | 여러 개 가능                | 단 하나만 가능                  |
| **가져오기 구문**   | `import { name } from...`   | `import anyName from...`        |
| **리팩토링 안전성** | 높음 (이름이 일관됨)        | 낮음 (가져올 때 이름 변경 가능) |
| **IDE 자동 완성**   | 우수함                      | 불일치할 수 있음                |
| **트리 쉐이킹**     | 매우 효과적                 | 덜 효과적일 수 있음             |
| **재내보내기**      | `export * from...`으로 간편 | 명시적 이름 지정 필요           |


**타입 내로잉(Type Narrowing)**은 TypeScript 컴파일러가 코드의 흐름을 분석하여, 특정 조건문 블록 내에서 변수의 타입을 더 구체적인 타입으로 좁혀나가는 과정을 의미한다.97 이는 유니언 타입을 안전하게 다루기 위한 필수적인 메커니즘이다. 이 과정을 돕는 조건문을 **타입 가드(Type Guards)**라고 한다.

- **`typeof` 타입 가드**: 변수의 원시 타입을 확인하는 데 사용된다. `typeof`는 `"string"`, `"number"`, `"boolean"`, `"object"` 등의 문자열을 반환하며, TypeScript는 이를 인식하여 타입을 좁힌다.97

  ```TypeScript
  function process(value: string | number) {
    if (typeof value === 'string') {
      // value는 여기서 string 타입이다.
      return value.toUpperCase();
    }
    // value는 여기서 number 타입이다.
    return value.toFixed(2);
  }
  ```
  
- **`instanceof` 타입 가드**: 값이 특정 클래스의 인스턴스인지 확인한다. 클래스와 함께 사용될 때 유용하다.97

  ```TypeScript
  class Dog { bark() {} }
  class Cat { meow() {} }
  
  function makeSound(animal: Dog | Cat) {
    if (animal instanceof Dog) {
      // animal은 여기서 Dog 타입이다.
      animal.bark();
    } else {
      // animal은 여기서 Cat 타입이다.
      animal.meow();
    }
  }
  ```
  
- **`in` 연산자 타입 가드**: 객체에 특정 속성이 존재하는지 확인한다. 객체의 형태를 구분하는 데 사용된다.97

  ```TypeScript
  type Fish = { swim: () => void };
  type Bird = { fly: () => void };
  
  function move(animal: Fish | Bird) {
    if ("swim" in animal) {
      // animal은 여기서 Fish 타입이다.
      return animal.swim();
    }
    // animal은 여기서 Bird 타입이다.
    return animal.fly();
  }
  ```
  
- **사용자 정의 타입 가드 (User-Defined Type Guards)**: 더 복잡한 로직이 필요할 때, 개발자가 직접 타입 가드 함수를 만들 수 있다. 이 함수는 불리언 값을 반환하며, 반환 타입 시그니처에 `parameter is Type` 형태의 **타입 서술어(type predicate)**를 사용한다.101

  ```TypeScript
  interface Car { drive: () => void; }
  interface Bike { ride: () => void; }
  
  // 사용자 정의 타입 가드 함수
  function isCar(vehicle: Car | Bike): vehicle is Car {
    return (vehicle as Car).drive!== undefined;
  }
  
  function start(vehicle: Car | Bike) {
    if (isCar(vehicle)) {
      // vehicle은 여기서 Car 타입이다.
      vehicle.drive();
    } else {
      // vehicle은 여기서 Bike 타입이다.
      vehicle.ride();
    }
  }
  ```

`isCar` 함수가 `true`를 반환하면, TypeScript는 `if` 블록 내에서 `vehicle` 변수를 `Car` 타입으로 간주한다. 이처럼 타입 가드는 유니언 타입의 모호성을 제거하고, 각 타입에 맞는 로직을 안전하게 실행할 수 있도록 보장하는 핵심적인 도구이다.



TypeScript는 React 애플리케이션 개발에 널리 사용되며, 컴포넌트의 안정성과 유지보수성을 크게 향상시킨다. React 프로젝트에서 TypeScript를 효과적으로 사용하는 핵심 패턴은 다음과 같다.


JSX를 포함하는 파일은 `.tsx` 확장자를 사용해야 한다.20 함수형 컴포넌트의 `props`는 `interface`나 `type` 별칭을 사용하여 타입을 정의하는 것이 가장 기본적인 관행이다.

```TypeScript
import React from 'react';

// Props 타입을 interface로 정의
interface GreetingProps {
  name: string;
  messageCount?: number; // 선택적 prop
}

const Greeting: React.FC<GreetingProps> = ({ name, messageCount = 0 }) => {
  return (
    <div>
      <h1>Hello, {name}!</h1>
      {messageCount > 0 && <p>You have {messageCount} new messages.</p>}
    </div>
  );
};
```

`React.FC` (Function Component) 타입을 사용하면 `children` prop이 기본적으로 포함되며, 컴포넌트의 반환 타입이 유효한 React 요소임을 보장한다.


`@types/react` 패키지는 React의 내장 훅에 대한 타입 정의를 제공한다.20

- **`useState`**: `useState`는 초기값을 기반으로 상태의 타입을 추론한다. 더 복잡한 타입(예: 유니언 타입)의 경우, 제네릭을 사용하여 명시적으로 타입을 지정할 수 있다.20

  ```TypeScript
  type Status = "idle" | "loading" | "success" | "error";
  
  const = React.useState<Status>("idle");
  // setStatus는 이제 'idle', 'loading', 'success', 'error' 중 하나만 인자로 받는다.
  ```
  
- **`useReducer`**: `useReducer`의 상태와 액션 타입을 명확히 정의하여 복잡한 상태 로직을 안전하게 관리할 수 있다.

  ```TypeScript
  interface State { count: number; }
  type Action = { type: 'increment' } | { type: 'decrement' };
  
  function reducer(state: State, action: Action): State {
    switch (action.type) {
      case 'increment': return { count: state.count + 1 };
      case 'decrement': return { count: state.count - 1 };
      default: throw new Error();
    }
  }
  
  const [state, dispatch] = React.useReducer(reducer, { count: 0 });
  ```


DOM 이벤트 핸들러를 타이핑할 때는 React가 제공하는 이벤트 타입을 사용한다. 이를 통해 이벤트 객체(`e`)의 타입과 그 속성(`e.target.value` 등)에 대한 정확한 타입 정보를 얻을 수 있다.

```TypeScript
function MyForm() {
  const [value, setValue] = React.useState('');

  const handleChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    setValue(event.target.value);
  };

  const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    console.log('Submitted:', value);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" value={value} onChange={handleChange} />
      <button type="submit">Submit</button>
    </form>
  );
}
```

`React.ChangeEvent<HTMLInputElement>`는 `input` 요소의 `onChange` 이벤트에 대한 타입을, `React.FormEvent<HTMLFormElement>`는 `form` 요소의 `onSubmit` 이벤트에 대한 타입을 정확하게 정의한다.


TypeScript 타입 시스템의 가장 근본적인 특징 중 하나는 **구조적 타이핑(Structural Typing)**을 채택했다는 점이다. 이는 "덕 타이핑(duck typing)"이라고도 불리며, 타입의 호환성을 이름이 아닌 구조(즉, 객체가 가진 속성과 메서드의 형태)를 기준으로 판단하는 방식이다.3

예를 들어, `Point`라는 인터페이스가 있다고 가정해보자.

```TypeScript
interface Point {
  x: number;
  y: number;
}

function logPoint(p: Point) {
  console.log(`${p.x}, ${p.y}`);
}
```

이제 `logPoint` 함수를 호출할 때, 명시적으로 `Point` 타입으로 선언되지 않은 객체를 전달해도 문제가 없다. 해당 객체가 `Point` 인터페이스가 요구하는 구조, 즉 `number` 타입의 `x`와 `y` 속성을 가지고 있기만 하면 된다.

```TypeScript
const point = { x: 12, y: 26 };
logPoint(point); // 정상적으로 동작한다.
```

`point` 변수는 `Point` 타입으로 선언된 적이 없지만, 그 구조가 `Point` 인터페이스의 요구사항과 일치하므로 TypeScript는 이를 호환 가능하다고 판단한다.

이러한 구조적 타이핑은 Java나 C#과 같은 **명목적 타이핑(Nominal Typing)** 시스템과 근본적인 차이를 보인다. 명목적 타이핑에서는 두 타입이 동일한 구조를 가졌더라도 이름이 다르면 서로 호환되지 않는다. TypeScript가 구조적 타이핑을 채택한 것은 JavaScript의 유연하고 동적인 객체 생성 방식과 자연스럽게 어우러지기 위한 전략적인 설계 결정이다. JavaScript 개발자들은 클래스 선언 없이 객체 리터럴을 통해 즉석에서 객체를 만드는 데 익숙하며, 구조적 타이핑은 이러한 패턴을 타입 시스템 내에서 원활하게 지원한다.

더 나아가, TypeScript의 구조적 호환성은 필요한 속성의 **부분 집합(subset)**만 만족하면 성립된다. 즉, 대상 타입이 요구하는 모든 속성을 가지고 있다면, 추가적인 속성을 더 가지고 있어도 호환되는 것으로 간주된다.

```TypeScript
const point3 = { x: 12, y: 26, z: 89 };
logPoint(point3); // 정상적으로 동작한다. z 속성은 무시된다.
```

`point3` 객체는 `Point`가 요구하는 `x`와 `y`를 모두 가지고 있으므로 `logPoint` 함수에 전달될 수 있다.

이처럼 구조적 타이핑은 TypeScript가 타입을 어떻게 "생각"하는지를 보여주는 핵심 원리이다. 타입은 이름이나 선언이 아닌, 객체가 가진 **능력(capability)**과 **형태(shape)**에 의해 결정된다. 이 원리를 이해하는 것은 복잡한 상황에서 TypeScript의 동작을 정확히 예측하고, 유연하면서도 안정적인 코드를 작성하는 데 필수적이다.


1. A Complete Guide to Beginning with TypeScript - Frontend Masters, 8월 6, 2025에 액세스, https://frontendmasters.com/blog/a-complete-guide-to-beginning-with-typescript/
2. TypeScript vs JavaScript: Which One Is Better to Choose? - Radixweb, 8월 6, 2025에 액세스, https://radixweb.com/blog/typescript-vs-javascript
3. Documentation - TypeScript for JavaScript Programmers, 8월 6, 2025에 액세스, https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html
4. radixweb.com, 8월 6, 2025에 액세스, [https://radixweb.com/blog/typescript-vs-javascript#:~:text=TypeScript%20is%20better%20for%20projects,JavaScript%20is%20also%20valid%20TypeScript.](https://radixweb.com/blog/typescript-vs-javascript#:~:text=TypeScript is better for projects,JavaScript is also valid TypeScript.)
5. What is TypeScript and why should I use it instead of JavaScript? [closed] - Stack Overflow, 8월 6, 2025에 액세스, https://stackoverflow.com/questions/12694530/what-is-typescript-and-why-should-i-use-it-instead-of-javascript
6. TypeScript vs JavaScript: How are they different? | Hygraph, 8월 6, 2025에 액세스, https://hygraph.com/blog/typescript-vs-javascript
7. The TypeScript Handbook, 8월 6, 2025에 액세스, https://www.typescriptlang.org/docs/handbook/intro.html
8. Difference between TypeScript and JavaScript - GeeksforGeeks, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/typescript/difference-between-typescript-and-javascript/
9. Where TypeScript Differs from JavaScript During Runtime Calls, 8월 6, 2025에 액세스, https://medium.com/@AlexanderObregon/where-typescript-differs-from-javascript-during-runtime-calls-9714bc4c96a9
10. Documentation - The Basics - TypeScript, 8월 6, 2025에 액세스, https://www.typescriptlang.org/docs/handbook/2/basic-types.html
11. Top 6 Benefits of Implementing TypeScript - Strapi, 8월 6, 2025에 액세스, https://strapi.io/blog/benefits-of-typescript
12. medium.com, 8월 6, 2025에 액세스, https://medium.com/@chaewonkong/typescript-what-good-for-8240dc9173c7
13. What Is TypeScript? Pros and Cons of TypeScript vs. JavaScript - STX Next, 8월 6, 2025에 액세스, https://www.stxnext.com/blog/typescript-pros-cons-javascript
14. How to set up TypeScript - TypeScript, 8월 6, 2025에 액세스, https://www.typescriptlang.org/download/
15. How to set up TypeScript with Node.js and Express - LogRocket Blog, 8월 6, 2025에 액세스, https://blog.logrocket.com/express-typescript-node/
16. Documentation - TypeScript 2.0, 8월 6, 2025에 액세스, https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-0.html
17. Adding Typescript to your Node.js project | by Lois T. | Geek Culture - Medium, 8월 6, 2025에 액세스, https://medium.com/geekculture/adding-typescript-to-your-node-js-project-fe4ba08369c8
18. Documentation - Everyday Types - TypeScript, 8월 6, 2025에 액세스, https://www.typescriptlang.org/docs/handbook/2/everyday-types.html
19. Handbook - Basic Types - TypeScript, 8월 6, 2025에 액세스, https://www.typescriptlang.org/docs/handbook/basic-types.html
20. Using TypeScript - React, 8월 6, 2025에 액세스, https://react.dev/learn/typescript
21. TypeScript Array Type, 8월 6, 2025에 액세스, https://www.typescripttutorial.net/typescript-tutorial/typescript-array-type/
22. TypeScript Arrays - Tutorials Teacher, 8월 6, 2025에 액세스, https://www.tutorialsteacher.com/typescript/typescript-array
23. Array TS Basics for Beginners - Daily.dev, 8월 6, 2025에 액세스, https://daily.dev/blog/array-ts-basics-for-beginners
24. Playground Example - Tuples - TypeScript, 8월 6, 2025에 액세스, https://www.typescriptlang.org/play/typescript/type-primitives/tuples.ts.html
25. Handbook - Enums - TypeScript, 8월 6, 2025에 액세스, https://www.typescriptlang.org/docs/handbook/enums.html
26. Why I Don't Like Enums | Total TypeScript, 8월 6, 2025에 액세스, https://www.totaltypescript.com/why-i-dont-like-typescript-enums
27. 'any' and 'unknown' types and the differences between them in typescript? - Medium, 8월 6, 2025에 액세스, https://medium.com/@ishwarnaruka/any-and-unknown-types-and-the-differences-between-them-in-typescript-1cc89f5aa062
28. Documentation - TypeScript 3.0, 8월 6, 2025에 액세스, https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-0.html
29. TypeScript's any vs unknown vs never: When and How to Use Them - Frontend Master, 8월 6, 2025에 액세스, https://rahuulmiishra.medium.com/typescripts-any-vs-unknown-vs-never-when-and-how-to-use-them-35d81ea57c01
30. unknown vs any in TypeScript - Dmitri Pavlutin, 8월 6, 2025에 액세스, https://dmitripavlutin.com/typescript-unknown-vs-any/
31. Never Type | TypeScript Deep Dive - GitBook, 8월 6, 2025에 액세스, https://basarat.gitbook.io/typescript/type-system/never
32. A complete guide to TypeScript's never type - Zhenghao He, 8월 6, 2025에 액세스, https://www.zhenghao.io/posts/ts-never
33. Null vs. Undefined | TypeScript Deep Dive - GitBook, 8월 6, 2025에 액세스, https://basarat.gitbook.io/typescript/recap/null-undefined
34. Undefined vs null : r/typescript - Reddit, 8월 6, 2025에 액세스, https://www.reddit.com/r/typescript/comments/11dpu05/undefined_vs_null/
35. Handbook - Unions and Intersection Types - TypeScript, 8월 6, 2025에 액세스, https://www.typescriptlang.org/docs/handbook/unions-and-intersections.html
36. What are intersection types in Typescript ? - GeeksforGeeks, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/typescript/what-are-intersection-types-in-typescript/
37. Documentation - Object Types - TypeScript, 8월 6, 2025에 액세스, https://www.typescriptlang.org/docs/handbook/2/objects.html
38. Typescript interfaces vs type aliases - DEV Community, 8월 6, 2025에 액세스, https://dev.to/rem0nfawzi/typescript-interfaces-vs-type-alias-bpk
39. Interfaces vs Types in TypeScript - Stack Overflow, 8월 6, 2025에 액세스, https://stackoverflow.com/questions/37233735/interfaces-vs-types-in-typescript
40. Type Aliases vs Interfaces in TypeScript - DEV Community, 8월 6, 2025에 액세스, https://dev.to/toluagboola/type-aliases-vs-interfaces-in-typescript-3ggg
41. Interface vs Type alias in TypeScript 2.7 | by Martin Hochel | Medium, 8월 6, 2025에 액세스, https://medium.com/@martin_hotell/interface-vs-type-alias-in-typescript-2-7-2a8f1777af4c
42. Type vs Interface: Which Should You Use? - Total TypeScript, 8월 6, 2025에 액세스, https://www.totaltypescript.com/type-vs-interface-which-should-you-use
43. What's the best practice for when to use interface vs. class vs. type? : r/typescript - Reddit, 8월 6, 2025에 액세스, https://www.reddit.com/r/typescript/comments/qc6u5d/whats_the_best_practice_for_when_to_use_interface/
44. Documentation - Utility Types - TypeScript, 8월 6, 2025에 액세스, https://www.typescriptlang.org/docs/handbook/utility-types.html
45. TypeScript Utility Types - GeeksforGeeks, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/typescript/typescript-utility-types/
46. How to use TypeScript Partial Type? | Refine - Refine dev, 8월 6, 2025에 액세스, https://refine.dev/blog/typescript-partial-utility-type/
47. TypeScript Partial Type: Syntax, Use Cases, and Examples - Mimo, 8월 6, 2025에 액세스, https://mimo.org/glossary/typescript/partial-type
48. TypeScript Readonly
49. TypeScript Readonly Type: Syntax, Use Cases, and Examples - Mimo, 8월 6, 2025에 액세스, https://mimo.org/glossary/typescript/readonly-type
50. The TypeScript Pick utility type - Graphite, 8월 6, 2025에 액세스, https://graphite.dev/guides/typescript-pick-utility-type
51. "Pick" & "Omit" in TypeScript | Bits and Pieces, 8월 6, 2025에 액세스, https://blog.bitsrc.io/pick-omit-in-typescript-abcb5b6093f7
52. The TypeScript Omit utility type - Graphite, 8월 6, 2025에 액세스, https://graphite.dev/guides/typescript-omit-utility-type
53. TypeScript Omit
54. What is the Record Type in TypeScript ? - GeeksforGeeks, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/typescript/what-is-the-record-type-in-typescript/
55. TypeScript Record Type with Examples | Refine - Refine dev, 8월 6, 2025에 액세스, https://refine.dev/blog/typescript-record-type/
56. Documentation - TypeScript 2.8, 8월 6, 2025에 액세스, https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html
57. Documentation - Conditional Types - TypeScript, 8월 6, 2025에 액세스, https://www.typescriptlang.org/docs/handbook/2/conditional-types.html
58. Functions - TypeScript: Handbook, 8월 6, 2025에 액세스, https://www.typescriptlang.org/docs/handbook/functions.html
59. TypeScript Optional Parameter: Syntax, Usage, and Examples - Mimo, 8월 6, 2025에 액세스, https://mimo.org/glossary/typescript/optional-parameter
60. Optional Parameters | TypeScript Guide by Convex, 8월 6, 2025에 액세스, https://www.convex.dev/typescript/advanced/typescript-optional-parameters
61. TypeScript Optional and Default Parameters (with Examples) - HowToDoInJava, 8월 6, 2025에 액세스, https://howtodoinjava.com/typescript/typescript-optional-and-default-parameters/
62. TypeScript Default Parameters, 8월 6, 2025에 액세스, https://www.typescripttutorial.net/typescript-tutorial/typescript-default-parameters/
63. Documentation - More on Functions - TypeScript, 8월 6, 2025에 액세스, https://www.typescriptlang.org/docs/handbook/2/functions.html
64. Documentation - Classes - TypeScript, 8월 6, 2025에 액세스, https://www.typescriptlang.org/docs/handbook/2/classes.html
65. Handbook - Classes - TypeScript, 8월 6, 2025에 액세스, https://www.typescriptlang.org/docs/handbook/classes.html
66. Access Modifiers in TypeScript - GeeksforGeeks, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/typescript/access-modifiers-in-typescript/
67. Understanding Class Modifiers in TypeScript: Public, Private, and Protected - Medium, 8월 6, 2025에 액세스, https://medium.com/@AbbasPlusPlus/understanding-class-modifiers-in-typescript-public-private-and-protected-f93bf6202e5d
68. TypeScript Data Modifiers: public, private, protected, static, readonly - Tutorials Teacher, 8월 6, 2025에 액세스, https://www.tutorialsteacher.com/typescript/data-modifiers
69. Understanding "public" / "private" in typescript class - Stack Overflow, 8월 6, 2025에 액세스, https://stackoverflow.com/questions/38713052/understanding-public-private-in-typescript-class
70. TypeScript Inheritance, 8월 6, 2025에 액세스, https://www.typescripttutorial.net/typescript-tutorial/typescript-inheritance/
71. TypeScript Inheritance - GeeksforGeeks, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/typescript/typescript-inheritance/
72. TypeScript Data Modifiers: public, private, protected, static, readonly - Tutorials Teacher, 8월 6, 2025에 액세스, https://www.tutorialsteacher.com/typescript/typescript-readonly
73. TypeScript readonly, 8월 6, 2025에 액세스, https://www.typescripttutorial.net/typescript-tutorial/typescript-readonly/
74. Understanding the `readonly` modifier | Learn TypeScript, 8월 6, 2025에 액세스, https://learntypescript.dev/10/l1-readonly-modifier/
75. TypeScript Static - Tutorials Teacher, 8월 6, 2025에 액세스, https://www.tutorialsteacher.com/typescript/typescript-static
76. TypeScript Static Methods and Properties - TypeScript Tutorial, 8월 6, 2025에 액세스, https://www.typescripttutorial.net/typescript-tutorial/typescript-static-methods-and-properties/
77. What are Abstract Classes in TypeScript? - GeeksforGeeks, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/typescript/what-are-abstract-classes-in-typescript/
78. TypeScript classes - Graphite, 8월 6, 2025에 액세스, https://graphite.dev/guides/typescript-classes
79. TypeScript Abstract Classes - TypeScript Tutorial, 8월 6, 2025에 액세스, https://www.typescripttutorial.net/typescript-tutorial/typescript-abstract-classes/
80. Documentation - Generics - TypeScript, 8월 6, 2025에 액세스, https://www.typescriptlang.org/docs/handbook/2/generics.html
81. TypeScript generics - Graphite, 8월 6, 2025에 액세스, https://graphite.dev/guides/typescript-generics
82. Understanding Generics in TypeScript: A Guide with Practical Examples - Medium, 8월 6, 2025에 액세스, https://medium.com/@ignatovich.dm/understanding-generics-in-typescript-a-guide-with-practical-examples-e73b670b445d
83. What are the difference between generic Type(T) vs any in typescript - Stack Overflow, 8월 6, 2025에 액세스, https://stackoverflow.com/questions/44023061/what-are-the-difference-between-generic-typet-vs-any-in-typescript
84. TypeScript Generic Interfaces - TypeScript Tutorial, 8월 6, 2025에 액세스, https://www.typescripttutorial.net/typescript-tutorial/typescript-generic-interfaces/
85. Generic Interface in TypeScript - Tutorials Teacher, 8월 6, 2025에 액세스, https://www.tutorialsteacher.com/typescript/typescript-generic-interface
86. TypeScript Generic Constraints - GeeksforGeeks, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/typescript/typescript-generic-constraints/
87. Advanced Typescript Generics In Practice - Dennis O'Keeffe, 8월 6, 2025에 액세스, https://www.dennisokeeffe.com/blog/2023-06-20-advanced-typescript-generics-in-practice
88. TypeScript Generic Constraints - TypeScript Tutorial, 8월 6, 2025에 액세스, https://www.typescripttutorial.net/typescript-tutorial/typescript-generic-constraints/
89. Documentation - Modules - TypeScript, 8월 6, 2025에 액세스, https://www.typescriptlang.org/docs/handbook/2/modules.html
90. Documentation - Modules - Theory - TypeScript, 8월 6, 2025에 액세스, https://www.typescriptlang.org/docs/handbook/modules/theory.html
91. Documentation - TypeScript 3.8, 8월 6, 2025에 액세스, https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-8.html
92. Documentation - Modules - Reference - TypeScript, 8월 6, 2025에 액세스, https://www.typescriptlang.org/docs/handbook/modules/reference.html
93. Difference Between Default & Named Exports in JavaScript - GeeksforGeeks, 8월 6, 2025에 액세스, https://www.geeksforgeeks.org/javascript/difference-between-default-named-exports-in-javascript/
94. Named vs Default Exports in JavaScript - YouTube, 8월 6, 2025에 액세스, https://www.youtube.com/shorts/5V2BXwxaKkg
95. Avoid Export Default | TypeScript Deep Dive - GitBook, 8월 6, 2025에 액세스, https://basarat.gitbook.io/typescript/main-1/defaultisbad
96. Default Export vs. Named Export: Which is Better? | by Hillary Ibeanu - Medium, 8월 6, 2025에 액세스, https://medium.com/@ibeanuhillary/default-export-vs-named-export-which-is-better-51faa54a5937
97. Documentation - Narrowing - TypeScript, 8월 6, 2025에 액세스, https://www.typescriptlang.org/docs/handbook/2/narrowing.html
98. Advanced TypeScript (7/10): Guarding Types with typeof/instanceof/literals - Medium, 8월 6, 2025에 액세스, https://medium.com/@seanbridger/advanced-typescript-7-10-guarding-types-with-typeof-instanceof-literals-c46c2668df45
99. Using an `instanceof` type guard - Learn TypeScript, 8월 6, 2025에 액세스, https://learntypescript.dev/07/l4-instanceof-type-guard/
100. How to use type guards in TypeScript - LogRocket Blog, 8월 6, 2025에 액세스, https://blog.logrocket.com/how-to-use-type-guards-typescript/
101. WHAT IS TYPE GUARD `IS` IN TYPESCRIPT ? | by wessam aftab ..., 8월 6, 2025에 액세스, https://medium.com/@wisamkayani360/what-is-type-guard-is-in-typescript-eeee6822c413
102. User-defined Type Guards in Typescript | by Slawek Plamowski | Level Up Coding, 8월 6, 2025에 액세스, https://levelup.gitconnected.com/user-defined-type-guards-in-typescript-fad639e4944f
103. Interfaces - TypeScript: Handbook, 8월 6, 2025에 액세스, https://www.typescriptlang.org/docs/handbook/interfaces.html

