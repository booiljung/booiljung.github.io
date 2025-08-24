[Deno](./index.md)
# Deno



Deno는 Node.js의 원작자인 라이언 달(Ryan Dahl)이 Node.js 개발 과정에서 인지한 여러 설계적 오류를 교정하고자 만든 새로운 런타임 환경이다.1 라이언 달은 Node.js가 가졌던 주요 한계점으로 불안정한 보안 모델, 복잡한 모듈 시스템, 콜백 기반의 비동기 처리 방식, 그리고 비효율적인 GYP 빌드 시스템 등을 지적한 바 있다.1 Node.js는 기본적으로 리소스에 대한 무제한 접근 권한을 부여하여 심각한 보안 문제를 초래할 수 있었으며 1, `node_modules` 디렉토리와 `package.json` 파일에 의존하는 중앙 집중형 패키지 관리 방식은 복잡성과 비효율성을 야기한다는 비판을 받았다.2 또한, 초기 Node.js의 비동기 호출은 콜백 API에 기반을 두어 이른바 '콜백 지옥'을 유발하는 경향이 있었고 2, 크롬에서조차 버려진 GYP 빌드 시스템을 계속 사용하면서 빌드 속도 저하와 기술적 복잡성이 가중되었다.2

Deno는 이러한 Node.js의 단점을 보완하고, 현대 웹 기술의 발전을 반영하며, 더욱 안전하고 효율적인 런타임 환경을 제공하는 것을 목표로 한다.3 Node.js의 창시자가 자신의 과거 설계에 대해 "후회"를 표명한 것은 단순한 개인적 회고를 넘어선다.2 이는 Node.js가 초기 설계 단계에서 가졌던 기술적 선택, 특히 Promises의 부재와 GYP 빌드 시스템과 같은 결정이 장기적으로 시스템의 확장성과 유지보수성에 미친 부정적 영향을 시사한다. Deno는 이러한 "기술 부채"를 처음부터 지지 않고, 현대 웹 기술 스택과 설계 원칙을 적용하여 더욱 견고하고 미래 지향적인 런타임을 구축하려는 시도로 평가할 수 있다. 이는 단순히 새로운 런타임을 만드는 것을 넘어, JavaScript 서버 사이드 생태계의 패러다임을 개선하려는 의지를 보여준다.


Deno는 Google의 V8 JavaScript 엔진을 기반으로 JavaScript 및 TypeScript 코드를 실행한다.1 V8 엔진은 Chrome 웹 브라우저에서도 사용되는 고성능 엔진으로, Deno가 빠른 실행 속도를 제공하는 데 기여한다.1

런타임의 핵심은 Rust 언어로 작성되었으며, 이는 Node.js가 C++로 개발된 것과 대조된다.1 Deno가 Node.js의 C++ 대신 Rust를 핵심 언어로 채택한 것은 단순한 언어 변경을 넘어선다.1 Rust는 메모리 안전성을 보장하고 동시성 프로그래밍에 강점을 가지며, C++에 비해 개발자 경험이 개선될 수 있다. 이는 Deno 런타임 자체의 안정성과 성능을 극대화하려는 전략적 선택으로, 특히 Node.js에서 지적되었던 안정성 문제 3를 근본적으로 해결하려는 의지로 해석된다. Rust의 엄격한 타입 시스템과 소유권 모델은 런타임 수준에서 잠재적인 버그를 줄이고, 장기적인 유지보수 비용을 절감하는 데 기여할 수 있다.

비동기 작업을 처리하기 위해 Tokio 프레임워크를 이벤트 루프로 활용한다.1 Tokio는 Rust 기반의 비동기 런타임으로, Node.js의 `libuv`와 유사한 역할을 수행하며 효율적인 비동기 I/O 처리를 가능하게 한다.10

또한, Deno는 TypeScript를 기본적으로 내장 지원하여, 별도의 설정이나 컴파일러 설치 없이 TypeScript 코드를 바로 실행할 수 있다.1 이는 개발 생산성을 향상시키고 더 강력한 정적 타입 검사를 제공하여 대규모 프로젝트 개발에 특히 유리하다.5



Deno는 기본적으로 모든 코드 실행에 대해 엄격한 보안 정책을 적용하며, 샌드박스 환경에서 애플리케이션을 실행한다.3 이는 Node.js가 기본적으로 파일 시스템, 네트워크, 환경 변수 등 리소스에 대한 무제한 접근 권한을 부여하는 것과 대조된다.5 Deno는 파일 시스템, 네트워크 연결, 환경 변수 접근과 같은 민감한 리소스에 대한 접근 시 명시적인 권한 부여를 요구한다.3

이러한 권한은 `deno run` 명령어 실행 시 `--allow-read`, `--allow-write`, `--allow-net`, `--allow-env`, `--allow-run` 등의 플래그를 통해 제어된다.3 예를 들어, 파일 읽기 권한은 `--allow-read` 플래그로, 네트워크 접근 권한은 `--allow-net` 플래그로 부여한다.3 특정 경로 또는 도메인에 대한 세분화된 권한 부여도 가능하여, 애플리케이션의 공격 표면을 최소화할 수 있다.25 모든 권한을 허용하는 `--allow-all` 플래그도 존재하나 12, 이는 샌드박스 보안을 완전히 비활성화하므로 신중한 사용이 요구된다.27

Deno의 핵심 가치 중 하나는 "기본적으로 안전함" 25이라는 원칙이다. 이는 Node.js의 무제한 접근 모델 5에 대한 직접적인 반작용으로 볼 수 있다. 그러나 `--allow-all` 플래그의 존재 12는 개발 편의성과 엄격한 보안 사이의 내재된 절충점을 명확히 보여준다. 이 플래그는 Node.js 개발자에게 익숙한 "무제한 접근" 환경을 제공하여 전환을 용이하게 할 수 있지만, Deno가 지향하는 보안 모델의 핵심을 훼손할 위험이 있다. 즉, 개발자가 보안에 대한 깊은 이해 없이 이 플래그를 남용할 경우, Deno의 주요 장점인 공급망 공격 완화 21 등의 이점을 상실할 수 있음을 의미한다. 따라서 이 플래그의 사용에 대한 강력한 경고와 함께, 필요한 최소한의 권한을 부여하는 "최소 권한 원칙" 14을 강조하는 것이 중요하다.

Deno의 샌드박싱은 V8 엔진의 내장 기능을 활용하여 코드 실행을 격리하며, 외부 자원에 대한 접근을 제한하여 잠재적으로 위험한 작업을 방지한다.25 이는 악성 코드나 취약한 서드파티 모듈로부터 시스템을 보호하는 데 핵심적인 역할을 한다.21 Deno의 보안 모델은 단순히 "권한을 요구한다"는 표면적인 설명을 넘어선다. V8의 샌드박싱 기능 25을 활용하여 코드를 격리하고, 기본적으로 모든 I/O를 차단하는 "최소 권한" 14 원칙을 적용하는 것은 Node.js의 "무제한 접근" 25과 근본적인 차이를 만든다. 특히, 특정 디렉토리나 도메인에 대한 세분화된 권한 부여 25는 애플리케이션의 공격 표면을 최소화하고, npm 생태계에서 빈번하게 발생하는 공급망 공격 21의 위험을 효과적으로 줄일 수 있다. 이는 개발자가 보안에 대한 깊은 지식 없이도 상대적으로 안전한 애플리케이션을 구축할 수 있도록 돕는 Deno의 가장 강력한 장점 중 하나이다.


| 플래그                         | 설명                               | 예시                                     |
| ------------------------------ | ---------------------------------- | ---------------------------------------- |
| `--allow-read[=<paths>...]`    | 파일 시스템 읽기 허용              | `deno run --allow-read=./data app.ts`    |
| `--allow-write[=<paths>...]`   | 파일 시스템 쓰기 허용              | `deno run --allow-write=/tmp app.ts`     |
| `--allow-net[=<hosts>...]`     | 네트워크 접근 허용                 | `deno run --allow-net=google.com app.ts` |
| `--allow-env[=<variables>...]` | 환경 변수 접근 허용                | `deno run --allow-env=API_KEY app.ts`    |
| `--allow-run[=<commands>...]`  | 서브 프로세스 실행 허용            | `deno run --allow-run=git app.ts`        |
| `--allow-ffi`                  | 라이브러리 동적 로딩 허용          | `deno run --allow-ffi app.ts`            |
| `--allow-hrtime`               | 고해상도 시간 측정 허용            | `deno run --allow-hrtime app.ts`         |
| `--allow-all` / `-A`           | 모든 권한 허용 (샌드박스 비활성화) | `deno run --allow-all app.ts`            |

Deno의 보안 모델은 권한 플래그를 통해 구현된다. 이 표는 독자가 Deno의 보안 메커니즘을 구체적으로 이해하고, 자신의 애플리케이션에 필요한 최소한의 권한을 어떻게 부여할지 명확하게 파악하도록 돕는다. 각 플래그의 기능과 더불어, 특정 경로/호스트/변수 지정 옵션(`=<paths>...` 등)을 명시함으로써, Deno의 세분화된 권한 제어 능력을 부각할 수 있다. 이는 Deno의 "Secure by Default" 원칙을 실질적으로 보여주는 중요한 정보이다.


Deno는 웹 플랫폼 기능을 제공하고 웹 표준을 따른다.3 따라서 ES Modules, Web Workers, `fetch()` API 등을 Node.js와 달리 추가 설정 없이 그대로 사용할 수 있다.3 이는 브라우저 환경에서 작성된 JavaScript 코드가 Deno 런타임에서 거의 수정 없이 실행될 수 있는 "동형(isomorphic)" 애플리케이션 개발을 가능하게 한다.1 Deno는 `window` 객체 지원과 같은 브라우저 호환 기능을 제공하여 상호 운용성을 높인다.4

Deno는 TC39 (ECMAScript 표준화 위원회) 및 WinterCG (웹 호환성 그룹)에 적극적으로 참여하여 웹의 발전을 돕는다.22 이는 Deno가 최신 웹 표준을 빠르게 수용하고, 서버 사이드 JavaScript 환경과 브라우저 환경 간의 격차를 줄이려는 노력을 보여준다. Deno의 웹 표준 준수는 단순히 기술적 선택을 넘어선다.6 이는 프론트엔드 개발자가 서버 사이드 개발로 전환할 때 겪는 학습 곡선을 현저히 줄이는 효과 18를 가져온다. 브라우저에서 익숙한 API와 모듈 시스템을 서버에서도 동일하게 사용할 수 있게 함으로써, 개발자는 새로운 개념을 익히는 데 드는 시간을 절약하고, 클라이언트와 서버 간의 코드 재사용성을 극대화할 수 있다. 이는 개발 생산성 향상 8에 직접적으로 기여하며, 풀스택 개발 역량 강화에 중요한 역할을 한다.


Deno는 단일 실행 파일로 배포되며, 외부 의존성이 없어 설치 및 업데이트가 간편하다.5 이 단일 실행 파일 안에는 TypeScript 컴파일러, 코드 포매터 (`deno fmt`), 린터 (`deno lint`), 테스트 러너 (`deno test`), 디버거/인스펙터, 번들러 (`deno bundle`), 코드 커버리지 도구 등 다양한 개발 도구가 내장되어 있다.7

이러한 내장 도구들은 별도의 설치 및 설정 없이 프로젝트를 구성하고 관리할 수 있게 하여 개발 생산성을 향상시킨다.8 Node.js 환경에서 개발을 시작할 때 개발자들은 종종 린팅, 포매팅, 테스트를 위한 여러 외부 도구를 설치하고 `package.json`이나 `tsconfig.json`과 같은 복잡한 설정 파일을 구성하는 데 상당한 시간을 할애한다.4 Deno의 "단일 실행 파일" 14과 "내장 도구 통합" 32은 이러한 초기 설정 오버헤드를 완전히 제거하여 개발자가 즉시 코드 작성에 집중할 수 있도록 한다. 이는 특히 소규모 유틸리티 스크립트나 새로운 프로젝트를 시작할 때 개발자의 진입 장벽을 낮추고, "제로 설정 TypeScript" 22와 같은 경험을 제공하여 개발 효율성을 극대화한다. 

`deno fmt`는 JavaScript, TypeScript, JSON, Markdown 파일까지 포매팅하며 22, `deno lint`는 코드의 잠재적 문제를 분석한다.32 `deno test`는 테스트 파일을 자동으로 검색하고 실행하며, 코드 커버리지 리포트 생성 기능도 제공한다.32


Deno는 Node.js가 CommonJS와 ES Modules를 모두 지원하는 것과 달리, 브라우저와 같이 ES Modules만을 유일한 모듈 시스템으로 사용한다.1 Node.js 호환성 레이어를 통해 CommonJS 모듈을 사용할 수도 있다.10

Deno는 모듈 로드를 위해 `npm`과 같은 중앙 집중형 패키지 매니저를 사용하지 않고, HTTP URL이나 파일 경로를 통해 직접 모듈을 가져온다.1

`deno.land/x`는 GitHub에 호스팅된 오픈소스 Deno 모듈을 캐싱하고 쉬운 URL로 제공하는 서비스이다.40 또한  `esm.sh`, `jsr.io`, `jsDelivr` 등 다양한 CDN에서 모듈을 가져올 수 있다.39 Deno는 모듈을 처음 실행할 때 로컬에 캐싱하여 이후 오프라인 환경에서도 실행 가능하게 한다.1

Deno의 초기 URL 기반 모듈 시스템은 `node_modules`의 복잡성 2을 해결하려는 혁신적인 시도였으나, 간접 의존성의 버전 관리 41나 패키지 발견의 어려움 38과 같은 새로운 도전 과제에 직면했다. 이에 Deno는 JSR 39이라는 공식 레지스트리를 도입하고, Deno 2에서 `npm:` 접두사를 통한 `npm` 패키지 직접 임포트 18를 강화하는 등, 이상과 현실 사이에서 실용적인 절충점을 찾아가고 있다. 이는 Deno가 순수주의를 넘어 사용자 채택을 위한 유연성을 확보하려는 전략적 움직임으로 해석된다. 

`deno add` 명령어를 통해 `deno.json` 파일에 의존성을 명시적으로 추적하고 `deno.lock` 파일로 버전을 고정할 수 있다.39

`import map`을 사용하여 모듈 경로를 관리할 수도 있다.38


Deno의 모든 비동기 작업은 Promise를 반환하며, `async/await` 문법을 기본적으로 지원한다.3 Node.js가 초기에는 콜백 기반 API를 사용했던 것과 달리, Deno는 Promises를 고집하여 콜백 지옥(callback hell)을 피하고 코드를 더 명확하고 간결하게 작성할 수 있도록 한다.2

`Top-Level await`를 지원하여 `async` 함수로 감싸지 않고도 최상위 레벨에서 `await`를 사용할 수 있어 코드 가독성을 높인다.3

Deno는 처리되지 않은 Promise 거부(unhandled promise rejections) 발생 시 애플리케이션을 즉시 종료시키는 엄격한 정책을 가진다.4 이는 Node.js가 경고를 발생시키고 종료하지 않는 것과 대조된다.4 Promises와 

`async/await`는 현대 JavaScript 비동기 프로그래밍의 표준으로 자리 잡았다. Node.js의 창시자가 Promises를 초기에 삭제했던 것을 후회한 점 2은 Deno가 이 부분을 설계 원칙으로 삼은 배경이 된다. Deno가 모든 비동기 작업을 Promise로 통일하고, 처리되지 않은 Promise 거부에 대해 애플리케이션을 즉시 종료시키는 정책 13을 채택한 것은 단순한 문법적 선호를 넘어선다. 이는 비동기 코드의 안정성을 강화하고, 잠재적인 오류를 조기에 발견하여 애플리케이션의 예측 불가능한 동작을 방지하려는 의도로 볼 수 있다. 이는 개발자가 더 견고하고 신뢰할 수 있는 비동기 애플리케이션을 구축하도록 유도한다.



Deno는 TypeScript를 일급 언어로 지원하며, 별도의 설정이나 `tsconfig.json` 파일 없이 TypeScript 코드를 바로 실행할 수 있다.3 내장된 TypeScript 컴파일러(TSC와 SWC 조합)가 TypeScript 코드를 JavaScript로 변환하여 V8 엔진에서 실행한다.12 이 과정은 개발자가 인지하지 못하도록 백그라운드에서 원활하게 처리된다.12

Deno는 별도의 도구 없이 TypeScript 코드에 대한 타입 검사를 수행할 수 있으며 (`deno check`), 필요에 따라 원격 모듈이나 npm 패키지까지 포함하여 검사할 수 있다.16 타입 검사는 `deno run` 명령 시 `--check` 플래그를 통해 명시적으로 활성화할 수 있으며, 타입 오류 발생 시 코드 실행 전에 프로세스가 종료된다.16 특히  `deno test` 실행 시에는 기본적으로 타입 검사가 활성화되어 코드 품질을 높이는 데 기여한다.16

TypeScript의 내장 지원 16은 Deno의 핵심적인 강점이다. Node.js에서 TypeScript를 사용하려면 `tsconfig.json` 설정과 외부 컴파일러 4가 필요하여 초기 설정이 번거롭다. Deno는 이러한 복잡성을 제거하여 개발자가 즉시 타입 안정적인 코드를 작성할 수 있게 한다. 타입 검사가 기본적으로 활성화되고 (`deno check`), 특히 `deno test` 시에는 기본적으로 타입 검사가 수행되는 점 16은 코드 품질을 높이고 대규모 애플리케이션 개발 5에서 발생할 수 있는 오류를 사전에 방지하는 데 크게 기여한다. 이는 개발 생산성 8뿐만 아니라 장기적인 프로젝트 유지보수성 45에도 긍정적인 영향을 미친다.


Deno는 `npm`이나 `package.json` 없이 URL을 통해 모듈을 직접 임포트하는 방식을 사용한다.1 이는  `node_modules` 디렉토리의 복잡성과 크기 문제를 해결하고, 중앙 집중형 패키지 레지스트리에 대한 의존성을 줄이려는 의도이다.2`deno.land/x`는 GitHub에 호스팅된 오픈소스 Deno 모듈을 캐싱하고 쉬운 URL로 제공하는 서비스이다.40 또한 `esm.sh`, `jsr.io`, `jsDelivr` 등 다양한 CDN에서 모듈을 가져올 수 있다.39 Deno는 모듈을 처음 실행할 때 로컬에 캐싱하여 이후 오프라인 환경에서도 실행 가능하게 한다.1

Deno 2부터는 `npm:` 접두사를 사용하여 npm 패키지를 직접 임포트할 수 있게 되었으며, `package.json` 및 `node_modules` 지원이 강화되었다.18 이는 Node.js 생태계와의 호환성을 크게 향상시킨다.30

`deno add` 명령어를 통해 `deno.json` 파일에 의존성을 명시적으로 추적하고 `deno.lock` 파일로 버전을 고정할 수 있다.39

`import map`을 사용하여 모듈 경로를 관리할 수도 있다.38

Deno의 초기 URL 기반 모듈 시스템은 `node_modules`의 복잡성 2을 해결하려는 혁신적인 시도였으나, 간접 의존성의 버전 관리 41나 패키지 발견의 어려움 38과 같은 실질적인 문제에 직면했다. Deno 2에서 `npm:` 접두사를 통한 `npm` 패키지 지원 43과 

`package.json` 통합 18은 Deno의 가장 큰 약점이었던 생태계 규모 18를 보완하기 위한 전략적 움직임이다. 이는 Deno가 Node.js를 완전히 대체하기보다는, Node.js의 방대한 라이브러리를 활용하면서도 Deno의 보안 및 개발자 경험 6 이점을 유지하는 "실용적인 대안" 4으로 포지셔닝하려는 의도를 보여준다. JSR 43은 Deno-native 패키지의 중앙 레지스트리 역할을 하며, `deno add` 39와 `deno.lock` 39은 이러한 하이브리드 환경에서 의존성 관리를 체계화하는 데 기여한다.


Deno는 "Secure by Default" 원칙을 통해 기본적으로 파일 시스템, 네트워크, 환경 변수 등 시스템 I/O에 대한 접근을 거부한다.22 이는 Node.js와 달리 애플리케이션이 불필요한 권한을 가지는 것을 방지하여 공격 표면을 최소화한다.25

권한은 `--allow-read`, `--allow-write`, `--allow-net`, `--allow-env`, `--allow-run`과 같은 커맨드라인 플래그를 통해 명시적으로 부여해야 한다.3 예를 들어, 

```
deno run --allow-read=./data --allow-net=api.example.com app.ts
```

와 같이 특정 경로 또는 도메인에 대한 세분화된 권한 부여도 가능하다.25 이는 개발자가 스크립트가 어떤 리소스에 접근할 수 있는지 명확히 이해하고 제어할 수 있도록 돕는다.25

샌드박스 실행은 V8의 내장 기능을 활용하여 코드 실행을 격리하고, 민감한 리소스에 대한 직접적인 접근을 방지한다.25 이러한 격리 메커니즘은 악성 코드나 취약한 서드파티 모듈이 시스템에 미치는 영향을 제한하여 공급망 공격으로부터 시스템을 보호하는 데 핵심적인 역할을 한다.21 Deno는 기본적으로 사용자 코드를 신뢰하지 않고, 일반적인 프로세스가 작동하는 방식처럼 Web Worker를 통해 다른 프로세스를 구동할 수도 있다.1 또한, Deno는 캐싱 및 KV 스토리지 API를 통해 동일한 애플리케이션의 여러 호출이 데이터를 공유할 수 있는 메커니즘을 제공하지만, 다른 애플리케이션은 서로의 데이터를 볼 수 없도록 하여 데이터 격리를 보장한다.27


Deno는 `deno fmt` (코드 포매팅), `deno lint` (코드 린팅), `deno test` (테스트 실행)와 같은 도구를 CLI에 내장하고 있다.12 이 도구들은 별도의 설치나 설정 없이 바로 사용할 수 있으며, JavaScript, TypeScript, JSON, Markdown 파일까지 지원한다.12

`deno fmt`는 Deno의 코드 규칙과 관례에 따라 코드를 자동으로 포매팅하여 일관성을 유지한다.22

`deno lint`는 코드의 잠재적 문제나 모범 사례 위반 여부를 분석하며, ESLint와 유사한 기능을 제공한다.32

`deno test`는 테스트 파일을 자동으로 검색하고 실행하며, 코드 커버리지 리포트 생성 기능도 제공한다.32 Node.js 생태계에서는 코드 품질 유지를 위해 ESLint, Prettier, Jest/Mocha 등 다양한 외부 도구를 조합하고 설정해야 한다. 이는 개발 환경 설정에 상당한 시간과 노력을 요구하며, 팀 내에서 코드 관례를 통일하는 데도 어려움이 있을 수 있다. Deno가 이러한 핵심 개발 도구들을 내장하고 32, 심지어 마크다운 내 코드 스니펫까지 포매팅하는 기능 32을 제공하는 것은 개발 워크플로우를 극적으로 간소화하고 표준화하려는 의도를 보여준다. 이는 개발자에게 일관된 경험을 제공하고, 프로젝트의 초기 설정 비용을 줄이며, 코드 품질을 쉽게 유지할 수 있도록 돕는다.


Deno는 Go 언어의 표준 라이브러리를 모델로 한 자체 표준 라이브러리(`deno.land/std`)를 제공한다.5 이 표준 라이브러리는 파일 시스템 작업, HTTP 서버, 데이터 인코딩 등 일반적인 작업을 위한 모듈을 포함하며, Deno 코어 팀에 의해 감사되고 검증되어 일관성과 안정성을 보장한다.6 많은 표준 라이브러리 패키지는 Node.js 및 Cloudflare Workers와 같은 다른 JavaScript 환경과도 호환된다.42

Deno는 `fetch()`, `WebSockets`, `Web Workers`, `BroadcastChannel` 등 다양한 웹 표준 API를 내장하여 브라우저 환경과의 호환성을 높인다.3 이는 개발자가 브라우저와 서버 환경 간에 코드를 재사용하고, 웹 개발 지식을 서버 사이드 개발에 직접 적용할 수 있도록 지원한다.21 Node.js는 최소한의 코어 API를 제공하고 대부분의 기능을 npm 생태계에 의존한다. 반면 Deno는 "코어 API 최소화, 대규모 표준 라이브러리 제공" 10이라는 철학을 따른다. 이는 개발자가 외부 의존성 없이도 일반적인 작업을 수행할 수 있도록 하여, `node_modules`와 같은 문제 2를 피하고, 보안 감사 6를 거친 신뢰할 수 있는 모듈을 제공한다. 웹 표준 API의 적극적인 통합 29은 Deno가 브라우저 환경과의 일관성을 유지하려는 노력을 보여주며, 이는 개발자가 학습한 지식을 다양한 환경에서 재활용할 수 있게 하여 효율성을 높인다.



| 특징                | Deno                                                         | Node.js                                                      |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **언어 지원**       | TypeScript, JavaScript, WebAssembly 기본 지원                | JavaScript 기본, TypeScript는 별도 설정 필요                 |
| **런타임 핵심**     | Rust, Tokio                                                  | C++, libuv                                                   |
| **모듈 시스템**     | ES Modules 전용, URL/JSR 기반 임포트, `npm:` 접두사로 npm 호환 | CommonJS 및 ES Modules 지원, npm을 통한 로컬 패키지 관리     |
| **패키지 관리**     | 내장 패키지 매니저 역할, `package.json` 및 `node_modules` 불필요 (Deno 2부터 `package.json` 지원) | npm을 통한 중앙 집중식 패키지 관리, `package.json` 및 `node_modules` 사용 |
| **보안 모델**       | 기본적으로 샌드박스, 권한 기반 접근, 명시적 허용 필요        | 기본적으로 무제한 접근                                       |
| **비동기 처리**     | Promise 기반, Top-Level await 지원, 처리되지 않은 Promise 즉시 종료 | 콜백 및 Promise 혼용, Top-Level await 최근 지원, 처리되지 않은 Promise 경고 후 종료 |
| **내장 도구**       | 포매터, 린터, 테스트 러너, 번들러 등 내장                    | 대부분 외부 서드파티 도구 필요                               |
| **브라우저 호환성** | 웹 표준 API 적극 지원, 동형 코드 가능                        | 일부 웹 API 미지원, 폴리필 필요                              |
| **생태계 규모**     | 상대적으로 작고 성장 중                                      | 방대하고 성숙함                                              |
| **단일 실행 파일**  | 단일 실행 파일로 배포 가능                                   | 별도 번들링 도구 필요                                        |

이 표는 Deno와 Node.js의 핵심적인 차이점을 한눈에 비교할 수 있도록 하여 독자의 이해를 돕는 데 필수적이다. 각 항목별로 명확한 대조를 통해 Deno가 Node.js의 한계를 어떻게 극복하려 했는지, 그리고 어떤 새로운 접근 방식을 취했는지 직관적으로 보여준다. 특히 Deno 2의 `npm` 호환성 강화와 같은 최신 동향을 반영하여, 단순한 과거 비교가 아닌 현재의 변화된 환경을 제시한다.


Deno는 Node.js와 비교하여 여러 차별화된 장점을 제공한다. 첫째, **보안** 측면에서 Deno는 기본적으로 모든 I/O 접근을 제한하고 명시적인 권한을 요구함으로써 Node.js의 주요 보안 취약점을 해결한다.4 이는 애플리케이션의 공격 표면을 줄이고 공급망 공격으로부터 보호하는 데 효과적이다.21

둘째, **TypeScript 기본 지원**은 Deno의 강력한 강점이다. 별도의 설정 없이 TypeScript를 즉시 사용할 수 있어 개발 생산성을 높이고 타입 안정성을 확보한다.4 이는 특히 대규모 애플리케이션 개발 시 코드의 견고성을 높이는 데 기여한다.

셋째, **간소화된 모듈 관리** 방식이다. Deno는 URL 기반 임포트와 내장 캐싱, 그리고 JSR 및 `npm:` 호환성을 통해 `node_modules`의 복잡성을 제거하고 의존성 관리를 단순화한다.3 이는 프로젝트 구조를 깔끔하게 유지하고 잠재적인 버전 충돌 문제를 줄이는 데 도움이 된다.

넷째, **향상된 개발 경험**을 제공한다. 단일 실행 파일에 내장된 포매터, 린터, 테스트 러너 등의 도구는 개발 환경 설정을 간소화하고 일관된 워크플로우를 제공한다.12 또한 Promise 기반 API와 Top-Level await 지원으로 비동기 코드 작성이 더 직관적이고 간결해진다.3 Deno의 여러 장점은 궁극적으로 개발자의 "행복" 6과 생산성 9을 극대화하는 데 초점을 맞춘다. Node.js가 시장의 필요에 의해 빠르게 성장하며 기능적 확장에 집중했다면, Deno는 Node.js의 경험을 바탕으로 개발자가 겪는 불편함과 비효율성(예: 복잡한 설정, 보안 우려, `node_modules` 문제)을 직접적으로 해결하려 한다. 이는 단순히 기술적 우위를 넘어, 개발자가 더 적은 마찰로 더 안전하고 효율적인 코드를 작성할 수 있도록 지원하는 "개발자 중심" 14의 설계 철학을 반영한다.


Deno는 여러 장점에도 불구하고 현재 몇 가지 한계점을 가지고 있다. 첫째, **생태계 규모**이다. Node.js에 비해 Deno의 생태계는 아직 작고 성숙도가 낮다. 방대한 npm의 서드파티 라이브러리들을 모두 지원하지는 않는다.5 Deno 2에서 npm 호환성을 크게 개선했음에도 불구하고, 여전히 Node.js의 광범위한 라이브러리 지원에는 미치지 못하는 부분이 존재한다.47

둘째, **학습 곡선**이다. Node.js에 익숙한 개발자는 Deno의 새로운 개념, API, 권한 모델 등에 적응하는 데 시간과 노력이 필요할 수 있다.18 특히 Deno의 권한 기반 보안 모델은 Node.js의 무제한 접근 방식에 익숙한 개발자에게는 초기 진입 장벽으로 작용할 수 있다.

셋째, **API 안정성**이다. 일부 Deno API는 아직 불안정(unstable) 상태이며, Node.js의 안정된 기능에 비해 부족한 부분이 있을 수 있다.47 이는 상용 프로젝트에 Deno를 도입할 때 잠재적인 위험 요소로 작용할 수 있다.

넷째, **커뮤니티 지원**이다. Node.js에 비해 Deno의 커뮤니티 규모가 작아 정보나 문제 해결에 어려움이 있을 수 있다.13 비록 Deno 커뮤니티가 친절하고 미래 지향적인 분위기를 가지고 있다는 평가도 있지만 50, 문제 발생 시 즉각적인 해결책을 찾기 어려울 수 있다는 점은 여전히 고려해야 할 부분이다.

Deno의 기술적 장점에도 불구하고, "생태계 규모" 18는 여전히 가장 큰 한계로 작용한다. 이는 기술 채택 곡선에서 "초기 사용자"와 "주류 시장" 사이의 "캐즘(Chasm)" 47을 연상시킨다. Deno 2에서 Node.js 및 npm 호환성을 대폭 강화한 것은 18 이러한 캐즘을 극복하고 더 넓은 개발자층을 유인하기 위한 전략적 시도로 볼 수 있다. 기존 Node.js 프로젝트를 Deno로 점진적으로 마이그레이션하거나, npm의 방대한 라이브러리를 Deno 환경에서 활용할 수 있게 함으로써, Deno는 기술적 우수성뿐만 아니라 실용적인 유용성 18을 동시에 추구하고 있다.

Deno의 주요 활용 사례 및 적용 분야


Deno는 `Deno.serve()` API를 통해 간단하고 빠르게 HTTP 서버를 구축할 수 있으며, 이는 Node.js의 내장 HTTP 서버와 유사한 기능을 제공한다.11

`oak`, `hono`, `fresh`와 같은 미들웨어 프레임워크를 사용하여 복잡한 웹 애플리케이션 및 RESTful API 서버를 개발할 수 있다.17 특히 `Fresh`는 서버 측 렌더링(SSR)과 "클라이언트에 JavaScript를 전송하지 않는(zero JavaScript to client)" 아키텍처를 지향하는 차세대 웹 프레임워크로 21, 뛰어난 성능과 개발자 경험을 제공한다.34

`Fresh`는 빌드 단계가 필요 없고 34, 설정 없이 동작하며 34, JIT 렌더링을 지원하여 빠르고 가볍다는 특징을 가진다.51

Deno가 웹 서버 및 API 개발에 강점을 보이는 것은 단순히 Node.js의 대체재 역할을 넘어선다. `Fresh`와 같은 프레임워크 34가 "제로 JavaScript to client"를 기본으로 하고 서버 측 렌더링을 강조하는 것은, 웹 성능 최적화와 사용자 경험 향상이라는 현대 웹 개발의 주요 트렌드를 반영한다. 이는 Deno가 웹 개발의 새로운 패러다임을 선도하고, Edge 컴퓨팅 환경 52에 최적화된 아키텍처를 제공하려는 비전을 가지고 있음을 시사한다. Deno는 Astro, Remix, Vite 등 Node.js 기반의 인기 프레임워크도 `npm:` 접두사를 통해 사용할 수 있도록 호환성을 제공하여 기존 개발자들의 진입 장벽을 낮춘다.17


Deno는 Bash나 Python으로 작성되었을 수 있는 유틸리티 스크립트를 대체하는 데 적합하다.18 내장된 파일 시스템 작업, 네트워크 요청, 환경 변수 접근 등과 같은 시스템 유틸리티를 안전하게 사용할 수 있으며 19, 단일 실행 파일로 컴파일하여 배포하기 용이하다.19

`deno compile` 명령어를 통해 자가 포함(self-contained)된 크로스 플랫폼 실행 파일을 생성할 수 있다.12 이는 배포 및 관리의 편의성을 크게 높인다.

전통적으로 시스템 관리나 자동화 스크립트는 Bash, Python 등으로 작성되었다. Deno는 이러한 스크립팅 영역에서 JavaScript/TypeScript를 활용할 수 있는 강력한 대안을 제공한다. 특히, Deno의 "기본 보안" 19과 "권한 기반 접근" 26은 스크립트가 시스템에 미칠 수 있는 잠재적 위험을 최소화하면서도, `deno compile` 43을 통해 단일 실행 파일로 배포할 수 있어 배포 및 관리의 편의성을 크게 높인다. 이는 스크립팅 환경의 현대화뿐만 아니라 보안 수준을 한 단계 끌어올리는 중요한 의미를 가진다.


Deno Deploy는 Deno 기반 애플리케이션을 전 세계적으로 분산된 서버리스 엣지 환경에 배포할 수 있는 플랫폼이다.21 이는 사용자에게 더 가까운 곳에서 코드를 실행하여 낮은 지연 시간, 향상된 성능, 대역폭 비용 절감 등의 이점을 제공한다.21 Deno Deploy는 Deno KV (키-밸류 스토어), Queues, Cron과 같은 클라우드 네이티브 기본 요소를 내장하여 서버리스 애플리케이션 개발을 간소화한다.21 이는 세션 데이터, 피처 플래그, 협업 상태 등에 적합하며, 설정이 필요 없는 전역 일관성을 보장하는 간단한 API를 제공한다.53

Cloudflare Workers, AWS Lambda@Edge 등 기존 엣지 플랫폼의 대안으로 활용될 수 있다.52 서버리스와 엣지 컴퓨팅은 현대 클라우드 아키텍처의 중요한 축을 형성한다. Deno Deploy 52는 이러한 환경에 Deno를 최적화하여, 개발자가 복잡한 인프라 설정 없이도 고성능의 분산 애플리케이션을 쉽게 배포할 수 있도록 지원한다. 특히, Deno KV 53와 같은 내장된 상태 관리 기능은 서버리스 환경에서 데이터 지속성을 확보하는 데 중요한 역할을 한다. 이는 Deno가 단순히 런타임을 넘어, 분산 컴퓨팅 시대의 핵심 인프라 구성 요소로서 자리매김하려는 야심찬 비전을 가지고 있음을 보여준다. Deno Deploy는 전체 애플리케이션 호스팅 지향으로 진화 중이며, 서브프로세스, 백그라운드 작업, OpenTelemetry, 빌드 파이프라인, 자체 호스팅 리전 등을 지원할 예정이다.30


Deno 생태계는 다양한 웹 프레임워크와 라이브러리를 포함하며, 이는 Deno의 활용성을 높인다.

- **Fresh**: Deno에서 공식적으로 추진하는 풀스택 웹 프레임워크로, Preact를 기반으로 하며 기본적으로 클라이언트에 JavaScript를 전송하지 않는(zero JavaScript to client) 아일랜드 아키텍처를 특징으로 한다.21 빌드 단계가 필요 없고 34, 설정 없이 동작하며 34, JIT 렌더링을 지원하여 빠르고 가볍다.51

  `Fresh`는 서버 측에서 대부분의 렌더링을 수행하고, 클라이언트는 작은 상호작용 영역만 재렌더링하는 모델을 사용한다.17 런타임의 성공은 그 위에서 구축되는 프레임워크와 라이브러리 생태계의 활성화에 크게 좌우된다. 

  `Fresh` 34와 같은 Deno-native 프레임워크의 등장은 Deno가 단순한 런타임을 넘어 실제 웹 애플리케이션 개발에 활용될 수 있는 성숙한 플랫폼으로 진화하고 있음을 보여주는 중요한 지표이다. 특히 `Fresh`의 "제로 JS" 34와 "빌드 불필요" 51와 같은 혁신적인 접근 방식은 Deno의 설계 철학(웹 표준, 간소화)과 일치하며, Deno가 웹 개발의 미래 방향을 제시하려는 노력을 반영한다.

- **Oak**: Deno의 HTTP 미들웨어 프레임워크로, Node.js의 Express.js와 유사한 역할을 수행한다.17

- **Hono**: 웹 표준을 기반으로 구축된 웹 프레임워크로, Deno에서도 사용할 수 있다.17

- **JSR**: Deno의 공식 패키지 레지스트리로, `deno.land/std`와 같은 표준 라이브러리 패키지를 호스팅하며, TypeScript 및 npm 호환성을 지원한다.39

Deno는 Astro, Remix, Vite 등 Node.js 기반의 인기 프레임워크도 `npm:` 접두사를 통해 사용할 수 있도록 호환성을 제공한다.17 이는 기존 Node.js 개발자들이 Deno로 전환하거나 Deno 환경에서 기존 프로젝트를 활용하는 데 용이하게 한다.



Deno는 GitHub에서 10만 개 이상의 스타를 받았으며, 40만 명 이상의 활성 사용자를 보유하고 있다.22 이는 꾸준한 성장세를 보여주는 지표이다. 커뮤니티 지원은 Discord 채널을 통해 이루어지며, 유료 플랜에서는 이메일 지원 및 온보딩 지원도 제공된다.48 Deno 커뮤니티는 작지만 친절하고 미래 지향적인 분위기를 가지고 있다는 평가도 있다.50 Deno의 사용자 수 22와 GitHub 스타 22는 꾸준한 양적 성장을 보여준다. 그러나 Node.js의 방대한 생태계 5에 비하면 아직 규모가 작다는 점은 Deno의 주요 한계로 지적된다.18 Deno 커뮤니티의 "친절하고 미래 지향적인" 50 특성은 기술적 논의의 깊이와 혁신을 추구하는 데 긍정적인 영향을 미칠 수 있지만, 대규모 채택을 위해서는 더 많은 개발자의 유입과 활발한 기여가 필요하다. 이는 Deno가 양적 성장과 더불어 생태계의 질적 성숙을 동시에 추구해야 함을 의미한다.


Deno는 지속적인 업데이트를 통해 플랫폼을 발전시키고 있다.

- **Deno 2 출시**: 2024년 10월에 출시된 Deno 2는 Node.js 및 npm과의 하위 호환성을 크게 향상시켰다.10 이는 

  `package.json` 및 `node_modules` 지원, CommonJS 호환성 개선을 포함한다.18 이 업데이트는 Deno의 가장 큰 채택 장벽 중 하나를 제거하는 데 기여했다.30

- **JSR (JavaScript Registry) 도입**: JSR은 Deno Land에서 개발한 현대적이고 오픈소스 JavaScript 레지스트리로, TypeScript 소스 코드 직접 게시, 런타임 및 환경 간 모듈 로딩 처리, JSDoc 기반 문서 자동 생성, npm 호환성 등을 지원한다.42 JSR은 Deno-native 모듈의 중앙 허브 역할을 수행하며, `deno add`와 같은 내장 패키지 관리 명령어를 통해 사용된다.39

- **npm 호환성 강화**: Deno 2는 `npm:` 접두사를 통한 npm 패키지 임포트 기능을 대폭 개선하여, 기존 npm 라이브러리들을 Deno 환경에서 더 쉽게 사용할 수 있게 했다.18 이는 `@grpc/grpc-js`, `playwright`, `@google-cloud`, `mysql2` 등 다양한 npm 패키지를 Deno에서 사용할 수 있게 함으로써 Deno의 활용 범위를 크게 확장했다.43

- **`deno bundle` 재도입**: `esbuild`를 기반으로 `deno bundle` 명령어가 다시 도입되어, 단일 파일 JavaScript/TypeScript 번들링이 가능해졌다.35 향후 런타임 번들링 API 및 플러그인 지원이 계획되어 있어 유연성이 더욱 증대될 것이다.35

- **OpenTelemetry 안정화**: Deno 2.4에서 OpenTelemetry 지원이 안정화되어, 로그, 메트릭, 트레이스 수집을 위한 자동 계측이 가능하다.35 이는 Deno 애플리케이션의 모니터링 및 디버깅을 용이하게 한다.

- **`deno update` 명령어**: 의존성을 최신 버전으로 쉽게 업데이트할 수 있는 `deno update` 명령어가 추가되었다.35 이는 npm 및 JSR 의존성 모두에 적용된다.36

- **기타 개선 사항**: `DENO_COMPAT=1` 환경 변수를 통해 Node.js 호환성을 높이는 기능 35, `rusty_v8` 라이브러리 안정화 43, LSP(Language Server Protocol) 개선 36 등이 이루어졌다.

Deno의 발전 로드맵 35은 Node.js의 "대안" 4이라는 초기 포지셔닝을 넘어 "주류" 런타임으로 도약하려는 전략적 변화를 보여준다. Deno 2에서 Node.js 및 npm 호환성을 대폭 강화한 것은 18 이러한 캐즘을 극복하고 더 넓은 개발자층을 유인하기 위한 전략적 시도로 볼 수 있다. 기존 Node.js 프로젝트를 Deno로 점진적으로 마이그레이션하거나, npm의 방대한 라이브러리를 Deno 환경에서 활용할 수 있게 함으로써, Deno는 기술적 우수성뿐만 아니라 실용적인 유용성 18을 동시에 추구하고 있다. Deno는 단순한 런타임을 넘어 완전한 JavaScript 시스템 플랫폼으로 진화하고 있으며, 한 도구 체계로 작성, 실행, 테스트, 배포, 모니터링 등 통합 수행을 목표로 한다.30


Deno는 Node.js의 창시자가 기존 런타임의 한계를 극복하고자 설계한 현대적인 JavaScript 및 TypeScript 런타임이다. Rust 기반의 견고한 아키텍처, TypeScript 기본 지원, 웹 표준 준수, 그리고 단일 실행 파일에 통합된 개발 도구들은 Deno의 핵심 강점이다. 특히, 기본적으로 모든 I/O 접근을 제한하고 명시적인 권한을 요구하는 강력한 보안 모델은 Node.js와 비교하여 Deno의 가장 큰 차별점이며, 공급망 공격과 같은 보안 위협에 대한 효과적인 방어책을 제공한다.

URL 기반 모듈 시스템은 `node_modules`의 복잡성을 해소하려는 혁신적인 시도였으며, Deno 2에서 `npm:` 접두사를 통한 npm 패키지 호환성 강화와 JSR 레지스트리 도입은 Deno 생태계의 실용성과 확장성을 크게 높였다. 이는 Deno가 순수주의를 넘어 기존 JavaScript 생태계와의 상호 운용성을 확보하려는 전략적 움직임으로 평가된다. Promise 기반의 비동기 처리 방식과 Top-Level await 지원은 코드의 가독성과 안정성을 향상시키며, 개발자에게 더 나은 경험을 제공한다.

Deno는 웹 서버 및 API 개발, 명령줄 유틸리티 스크립트, 그리고 서버리스 및 엣지 컴퓨팅 환경에서 강력한 대안으로 부상하고 있다. `Fresh`와 같은 Deno-native 프레임워크의 발전은 Deno가 웹 개발의 새로운 패러다임을 선도하고 있음을 보여준다.

그러나 Deno는 Node.js에 비해 아직 생태계 규모가 작고, 일부 API의 성숙도가 낮으며, 기존 Node.js 개발자에게는 새로운 학습 곡선이 존재한다는 한계점도 명확하다. 그럼에도 불구하고, Deno는 지속적인 업데이트와 `npm` 호환성 강화를 통해 이러한 격차를 줄여나가고 있으며, 단순한 Node.js의 "대안"을 넘어 "주류" 런타임으로 도약하려는 야심찬 비전을 가지고 있다. Deno의 미래는 JavaScript 생태계의 발전 방향과 밀접하게 연관되어 있으며, 보안, 성능, 개발자 경험 측면에서 지속적인 혁신을 통해 그 잠재력을 실현할 것으로 기대된다.


1. Node 제작자가 만든 Deno: 자바스크립트의 새로운 접근 - ull.im, 8월 3, 2025에 액세스, https://blog.ull.im/engineering/2019/04/14/deno-ryan-dahl-2019-04-04.html
2. Node.js에 관한 10가지 후회 - 라이언 달과 Deno.js - 로켓 - 티스토리, 8월 3, 2025에 액세스, https://kingofbackend.tistory.com/112
3. [Deno] 신생 JS & TS 런타임 데노(Deno) - 개발 & 공부 일지, 8월 3, 2025에 액세스, https://laikhan-workshop.tistory.com/54
4. Deno vs Node.js: a comparison - Imaginary Cloud, 8월 3, 2025에 액세스, https://www.imaginarycloud.com/blog/deno-vs-node
5. Deno의 모든 것 : Node.js와 간단하게 비교해보기 - 한장현입니다., 8월 3, 2025에 액세스, https://han41858.tistory.com/55
6. Comparison of Deno vs Node.js, 8월 3, 2025에 액세스, https://bluenodesolutions.com/comparison-of-deno-vs-node-js/
7. Deno vs Node.js - 알아야 할 비교 - Reddit, 8월 3, 2025에 액세스, https://www.reddit.com/r/node/comments/nx9qqr/deno_vs_nodejs_a_comparison_you_need_to_know/?tl=ko
8. Deno: 안전하고 최신 기능을 지원하는 JavaScript 및 TypeScript 런타임 - Dak.so, 8월 3, 2025에 액세스, https://www.dak.so/deno
9. 디노 (소프트웨어) - 위키백과, 우리 모두의 백과사전, 8월 3, 2025에 액세스, [https://ko.wikipedia.org/wiki/%EB%94%94%EB%85%B8_(%EC%86%8C%ED%94%84%ED%8A%B8%EC%9B%A8%EC%96%B4)](https://ko.wikipedia.org/wiki/디노_(소프트웨어))
10. Deno (software) - Wikipedia, 8월 3, 2025에 액세스, https://en.wikipedia.org/wiki/Deno_(software)
11. denoland/deno: A modern runtime for JavaScript and TypeScript. - GitHub, 8월 3, 2025에 액세스, https://github.com/denoland/deno
12. Deno 살펴보기, 8월 3, 2025에 액세스, https://blog.outsider.ne.kr/1623
13. Deno - 개발자 박진 블로그 - 티스토리, 8월 3, 2025에 액세스, https://jinn-blog.tistory.com/115
14. 1.3 About Deno - The Internals of Deno - GitBook, 8월 3, 2025에 액세스, https://choubey.gitbook.io/internals-of-deno/introduction/about
15. Deno Basics for Beginners - Daily.dev, 8월 3, 2025에 액세스, https://daily.dev/blog/deno-basics-for-beginners
16. TypeScript support - Deno Docs, 8월 3, 2025에 액세스, https://docs.deno.com/runtime/fundamentals/typescript/
17. Web development - Deno Docs, 8월 3, 2025에 액세스, https://docs.deno.com/runtime/fundamentals/web_dev/
18. Deno Vs Node.js: Which One Is Better In 2025? - DevsData, 8월 3, 2025에 액세스, https://devsdata.com/deno-vs-node-which-one-is-better-in-2025/
19. [Deno] Node.js의 대안!! Deno 알아보기 - Dev. DY, 8월 3, 2025에 액세스, https://kdydesign.github.io/2022/02/17/deno-tutorial/
20. 2020년과 이후 JavaScript의 동향 - JavaScript(ECMAScript) - NAVER D2, 8월 3, 2025에 액세스, https://d2.naver.com/helloworld/4268738
21. Building websites and apps with Deno, 8월 3, 2025에 액세스, https://deno.com/learn/websites-apps
22. Deno, the next-generation JavaScript runtime, 8월 3, 2025에 액세스, https://deno.com/
23. Deno 애플리케이션을 배포하는 방법은 무엇인가요?, 8월 3, 2025에 액세스, [https://blog.back4app.com/ko/deno-%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98%EC%9D%84-%EB%B0%B0%ED%8F%AC%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95%EC%9D%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80%EC%9A%94/](https://blog.back4app.com/ko/deno-애플리케이션을-배포하는-방법은-무엇인가요/)
24. 데노 (Deno) - web - 넥스트티, 8월 3, 2025에 액세스, [https://www.next-t.co.kr/web/web-wiki/%EB%8D%B0%EB%85%B8-deno/](https://www.next-t.co.kr/web/web-wiki/데노-deno/)
25. Deno Security: Building Trustworthy Applications - DZone, 8월 3, 2025에 액세스, https://dzone.com/articles/deno-security-building-trustworthy-applications
26. Deno를 사용해보자! - Coding Groot - 티스토리, 8월 3, 2025에 액세스, https://coding-groot.tistory.com/137
27. Security and permissions - Deno Docs, 8월 3, 2025에 액세스, https://docs.deno.com/runtime/fundamentals/security/
28. Deno - 나무위키, 8월 3, 2025에 액세스, https://namu.wiki/w/Deno
29. Deno examples and tutorials, 8월 3, 2025에 액세스, https://docs.deno.com/examples/
30. Reports of Deno's Demise Have Been Greatly Exaggerated, 8월 3, 2025에 액세스, https://deno.com/blog/greatly-exaggerated
31. Deno Docs, 8월 3, 2025에 액세스, https://docs.deno.com/runtime/
32. All-in-one tooling - Deno Docs, 8월 3, 2025에 액세스, https://docs.deno.com/examples/all-in-one_tooling/
33. [AskJS] Bun / Deno / NodeJS - what do you use and why? : r/javascript - Reddit, 8월 3, 2025에 액세스, https://www.reddit.com/r/javascript/comments/1jcwygf/askjs_bun_deno_nodejs_what_do_you_use_and_why/
34. Fresh - The simple, approachable, productive web framework., 8월 3, 2025에 액세스, https://fresh.deno.dev/
35. Deno FINALLY Reintroduces Bundle Everyone Has Been Asking For | HackerNoon, 8월 3, 2025에 액세스, https://hackernoon.com/deno-finally-reintroduces-bundle-everyone-has-been-asking-for
36. Deno 2.4: deno bundle is back, 8월 3, 2025에 액세스, https://deno.com/blog/v2.4
37. DENO - velog, 8월 3, 2025에 액세스, https://velog.io/@rinm/DENO
38. Deno 패키지 관리 방법 - Dak.so, 8월 3, 2025에 액세스, https://www.dak.so/deno/how-to-use-package
39. Dependency Management in Deno - Kevin Cunningham, 8월 3, 2025에 액세스, https://www.kevincunningham.co.uk/posts/intro-to-jsr
40. Deno Third Party Modules, 8월 3, 2025에 액세스, https://deno.land/x
41. Deno는 간접적인 의존성의 버전을 관리할 수 있게 해줘요? 아니면 기본적으로 직접적인 의존성이 사용하는 정확한 버전을 사용해야 하나요? - Reddit, 8월 3, 2025에 액세스, https://www.reddit.com/r/Deno/comments/glerhe/does_deno_let_you_manage_versions_for_indirect/?tl=ko
42. Standard Library - Deno Docs, 8월 3, 2025에 액세스, https://docs.deno.com/runtime/fundamentals/standard_library/
43. Deno in 2024, 8월 3, 2025에 액세스, https://deno.com/blog/deno-in-2024
44. deno install, 8월 3, 2025에 액세스, https://docs.deno.com/runtime/reference/cli/install/
45. 객체지향 설계 원칙 5가지 - DevAndy, 8월 3, 2025에 액세스, https://youngjinmo.github.io/2021/04/principles-of-oop/
46. 2023년의 Deno, 8월 3, 2025에 액세스, https://news.hada.io/topic?id=13221
47. Deno vs. Node: No One is Ready for the Move - Reddit, 8월 3, 2025에 액세스, https://www.reddit.com/r/node/comments/13mckrr/deno_vs_node_no_one_is_ready_for_the_move/
48. Deno Deploy Pricing, 8월 3, 2025에 액세스, https://deno.com/deploy/pricing
49. Support and Feedback - Deno Docs, 8월 3, 2025에 액세스, https://docs.deno.com/deploy/early-access/support/
50. deno의 미래는 어떨 것 같아?, 8월 3, 2025에 액세스, https://www.reddit.com/r/Deno/comments/16svugk/what_is_the_future_of_deno/?tl=ko
51. Deno (r65 판) - 나무위키, 8월 3, 2025에 액세스, https://namu.wiki/w/Deno?uuid=716e3dba-826e-45ee-aa2a-3c64fed925b9
52. Deno Deploy Use Cases, 8월 3, 2025에 액세스, https://docs.deno.com/deploy/manual/use-cases/
53. Deno의 침체에 대한 소문은 크게 과장된 것입니다 - GeekNews, 8월 3, 2025에 액세스, https://news.hada.io/topic?id=21027