---
layout: page
title: MQTT 토픽 설계 원칙
permalink: /services/mqtt/MQTT 토픽 설계 원칙
---


MQTT(Message Queuing Telemetry Transport) 프로토콜은 발행-구독(Publish-Subscribe) 패러다임을 기반으로 동작하며, 이 구조의 근간을 이루는 것이 바로 '토픽(Topic)'이다.1 토픽은 단순히 메시지가 전달될 주소를 나타내는 문자열을 넘어, 전체 사물 인터넷(IoT) 시스템 아키텍처의 청사진이자 데이터 흐름의 논리적 경로를 정의하는 핵심 설계 요소이다. MQTT의 발행-구독 모델은 메시지를 보내는 발행자(Publisher)와 메시지를 받는 구독자(Subscriber)가 서로의 존재를 알 필요 없이 중앙의 브로커(Broker)를 통해 통신하도록 설계되었다.3 이러한 아키텍처는 구성 요소 간의 결합도를 현저히 낮추는 '분리(Decoupling)'를 가능하게 하는데, 바로 이 분리를 매개하는 유일한 연결고리가 토픽이다.5 발행자와 구독자는 공간적(서로의 IP 주소나 포트를 알 필요 없음), 시간적(동시에 온라인 상태일 필요 없음), 그리고 동기적(서로의 작업을 중단시키지 않음)으로 완벽하게 분리된다.3

이러한 완전한 분리 환경에서, 데이터 생산자와 소비자가 상호작용할 수 있는 유일한 공통분모는 '토픽 이름'과 그에 담기는 '메시지 페이로드 형식'뿐이다. 따라서 잘 설계된 토픽 구조는 시스템의 데이터 생산자와 소비자 간의 암묵적인 인터페이스, 즉 'API 계약(API Contract)'으로서의 역할을 수행한다. 토픽 구조를 변경하는 것은 RESTful API에서 엔드포인트 URL이나 데이터 스키마를 변경하는 것과 동일한 파급 효과를 가지며, 이는 토픽 설계가 초기 아키텍처 단계에서 얼마나 신중하게 다루어져야 하는지를 시사한다.

반면, 체계적인 원칙 없이 설계된 토픽 구조는 시스템 전반에 걸쳐 다차원적인 악영향을 미친다.

첫째, 불필요하게 길거나 비효율적인 토픽 이름은 모든 메시지에 포함되어 네트워크 대역폭을 낭비하고, 특히 리소스가 제한된 저전력 IoT 디바이스에 심각한 성능 저하를 유발한다.6

둘째, 경직된 토픽 구조는 새로운 디바이스나 기능이 추가될 때마다 기존 구조를 변경해야 하는 문제를 야기하여 시스템의 확장성을 심각하게 제약한다.6

셋째, 지나치게 광범위하거나 예측 불가능한 토픽 구조는 세분화된 접근 제어 정책(Access Control List, ACL) 적용을 어렵게 만들어, 인가되지 않은 데이터 접근을 허용하는 보안 취약점으로 작용할 수 있다.8 마지막으로, 토픽의 의미가 모호하거나 일관성이 결여된 경우, 시스템의 동작을 이해하고 문제를 해결하는 디버깅 과정이 매우 복잡해져 유지보수 비용이 급증하게 된다.6

본 보고서는 이러한 문제들을 해결하기 위해, MQTT 토픽의 기본 문법부터 시작하여 견고하고 확장 가능한 시스템 구축을 위한 핵심 설계 원칙, 실용적인 설계 패턴, 그리고 반드시 피해야 할 안티패턴까지 아우르는 종합적인 가이드라인을 제공하는 것을 목표로 한다. 이를 통해 시스템 아키텍트와 개발자가 성능, 확장성, 보안, 유지보수성을 극대화하는 토픽 구조를 스스로 설계할 수 있는 역량을 갖추도록 돕고자 한다.


효과적인 토픽 설계를 위해서는 먼저 MQTT 토픽이 가지는 고유한 구조와 문법적 제약사항을 명확히 이해해야 한다. 이러한 규칙들은 단순한 권장사항을 넘어, 시스템의 장기적인 안정성과 여러 시스템 간의 상호운용성을 담보하는 중요한 아키텍처 원칙으로 기능한다.


MQTT 토픽의 가장 큰 특징은 슬래시(`/`) 문자를 구분자로 사용하여 파일 시스템의 디렉토리와 유사한 계층적 구조를 형성한다는 점이다.3 이 계층 구조는 물리적 공간, 논리적 그룹, 장치의 종류 등 다양한 기준에 따라 수많은 IoT 디바이스와 여기서 생성되는 데이터를 체계적으로 분류하고 관리하는 데 매우 효과적이다.14

예를 들어, 스마트 홈 환경에서 거실에 위치한 온도 센서의 데이터를 나타내는 토픽은 다음과 같이 직관적으로 구성할 수 있다.

```
smart-home/living-room/temperature-sensor/value
```

이러한 구조는 토픽 이름만으로 데이터의 출처와 의미를 명확하게 파악할 수 있게 하며, 이후 설명할 와일드카드를 활용하여 특정 그룹의 데이터를 효율적으로 구독하는 기반이 된다.2


MQTT 토픽 이름은 특정 규칙과 제약사항을 따른다. 이러한 제약들은 표면적으로는 가독성이나 디버깅의 용이성을 위한 것처럼 보이지만, 그 본질은 서로 다른 클라이언트 구현체, 브로커, 백엔드 시스템 간의 상호운용성 문제를 원천적으로 차단하고 미래에 발생할 수 있는 예측 불가능한 통합 문제를 방지하는 '기술 부채 관리' 전략이다.

- **문자 집합**: 토픽 이름은 UTF-8 문자열로 구성된다.13 하지만 실제 시스템 구축 시에는 디버깅의 용이성과 시스템 간 호환성을 보장하기 위해 영문 소문자, 숫자, 그리고 하이픈(-)만으로 구성된 ASCII 문자 집합을 사용하는 것이 강력히 권장된다.6 비-ASCII 문자는 특정 로깅 시스템이나 분석 도구에서 예기치 않게 손상되어 표시될 수 있기 때문이다.6
- **대소문자 구분**: 토픽 이름은 대소문자를 엄격하게 구분한다. 따라서 `home/temperature`와 `home/Temperature`는 완전히 별개의 토픽으로 취급된다.6 이러한 혼동을 방지하기 위해 프로젝트 전반에 걸쳐 일관된 명명 규칙(예: 모든 토픽 레벨은 소문자로 작성)을 수립하고 강제하는 것이 매우 중요하다.18
- **길이 제한**: 토픽은 최소 한 글자 이상이어야 유효하다.6 MQTT 프로토콜 명세상 토픽 이름의 최대 길이는 65,535바이트로 정의되어 있지만, 실제 상용 브로커나 클라우드 IoT 플랫폼에서는 더 엄격한 제한을 두는 경우가 많다. 예를 들어, IBM MQ는 10,240자로 제한하며, AWS IoT Core는 256바이트의 UTF-8 문자열로 제한한다.18 따라서 사용하고자 하는 브로커의 제약사항을 사전에 확인해야 한다.
- **금지된 패턴**:
  - 메시지를 발행(Publish)할 때 토픽 이름에 와일드카드 문자(`+`, `#`)를 포함할 수 없다. 와일드카드는 오직 메시지를 구독(Subscribe)할 때만 사용 가능하다.18
  - 연속된 슬래시로 인해 발생하는 빈 레벨(예: `home//temperature`)은 유효하지 않은 토픽 구조이다.13
  - 토픽 이름에 NULL 문자(`\u0000`)를 포함할 수 없다.19


달러 기호(`$`)로 시작하는 토픽은 일반적인 데이터 교환 목적이 아닌, 브로커의 내부 상태 모니터링이나 특정 기능을 위해 예약된 특수한 시스템 토픽이다.21

가장 대표적인 예는 `$SYS` 토픽으로, 브로커의 버전, 현재 연결된 클라이언트 수, 초당 처리 메시지 수, CPU 부하 등 다양한 운영 통계 정보를 발행하는 데 사용된다.21 예를 들어, `$SYS/broker/clients/total` 토픽을 구독하면 현재 브로커에 연결된 클라이언트의 총 수를 실시간으로 확인할 수 있다.

`$`로 시작하는 토픽의 중요한 특징 중 하나는, 일반적인 다중 레벨 와일드카드(`#`)를 사용하여 모든 토픽을 구독하더라도 이 시스템 토픽들은 구독 결과에 포함되지 않는다는 점이다. `$SYS` 관련 토픽을 구독하기 위해서는 반드시 `$SYS/#`와 같이 명시적으로 해당 경로를 지정해야 한다.23


서비스 품질(Quality of Service, QoS)은 토픽 설계 자체의 일부는 아니지만, 각 토픽을 통해 전달되는 메시지의 신뢰성 수준을 결정하는 중요한 개념이다. MQTT는 세 가지 QoS 레벨을 제공하며, 이는 메시지 전달의 보장 수준과 시스템 성능 간의 트레이드오프 관계를 나타낸다. 시스템 아키텍트는 각 토픽의 목적과 중요도에 따라 적절한 QoS 레벨을 선택해야 한다.

| QoS 레벨  | 전달 보장 수준                  | 메시지 중복 가능성 | 성능 오버헤드    | 주요 사용 사례                                               |
| --------- | ------------------------------- | ------------------ | ---------------- | ------------------------------------------------------------ |
| **QoS 0** | **최대 한 번 (At most once)**   | 메시지 유실 가능   | 낮음 (가장 빠름) | 주기적으로 반복 전송되는 센서 데이터 (예: 온도, 습도) 2      |
| **QoS 1** | **최소 한 번 (At least once)**  | 중복 발생 가능     | 중간             | 상태 변경 알림, 반드시 전달되어야 하지만 중복 처리가 가능한 경우 |
| **QoS 2** | **정확히 한 번 (Exactly once)** | 중복 없음          | 높음 (가장 느림) | 결제 처리, 원격 제어 명령 등 데이터의 유실이나 중복이 허용되지 않는 경우 |

QoS 0은 메시지를 한 번만 보내고 전달 여부를 확인하지 않는 'Fire and Forget' 방식이다.13 QoS 1은 수신 확인(ACK)을 통해 최소 한 번의 전달을 보장하지만, 네트워크 문제 등으로 ACK가 유실되면 메시지가 재전송되어 중복이 발생할 수 있다.12 QoS 2는 4단계 핸드셰이킹 과정을 통해 정확히 한 번의 전달을 보장하지만, 이 과정에서 발생하는 오버헤드로 인해 성능이 가장 낮다.13 따라서 중요한 제어 명령을 전달하는 토픽에는 QoS 1 또는 2를, 빈번하게 발생하는 원격 측정(telemetry) 데이터 토픽에는 QoS 0 또는 1을 사용하는 것이 일반적인 설계 패턴이다.


MQTT의 와일드카드는 단일 구독 요청으로 여러 토픽에 대한 메시지를 수신할 수 있게 해주는 강력하고 유연한 기능이다.2 이는 시스템에 동적으로 추가되는 디바이스나 센서 데이터를 별도의 코드 수정 없이 처리할 수 있게 하고, 구독 로직의 복잡성을 크게 줄여준다.27


MQTT 아키텍처의 유연성은 데이터의 '생산 패턴'과 '소비 패턴'이 분리되어 있다는 점에서 비롯된다. 발행자는 항상 구체적이고 완전한 토픽 이름(와일드카드 없음)을 사용하여 메시지를 발행해야 한다.20 이는 데이터가 '어디서', '무엇에 대한' 정보인지를 명확하게 정의하는 '생산의 원칙'이다. 반면, 구독자는 와일드카드를 사용하여 자신의 '관심사'에 따라 데이터를 유연하게 소비할 수 있다.27 이것이 바로 '소비의 원칙'이다.

이 두 원칙의 분리는 시스템의 유연성을 극대화한다. 발행자는 구독자가 데이터를 어떻게 소비할지 전혀 알 필요가 없다. 예를 들어, `smart-home/living-room/temperature` 토픽에 온도 데이터가 발행될 때, 한 구독자는 이 토픽을 직접 구독하여 거실 온도만 확인할 수 있다. 동시에 다른 분석 시스템은 `smart-home/+/temperature`를 구독하여 모든 방의 온도를 집계할 수 있으며, 또 다른 로깅 시스템은 `smart-home/#`를 구독하여 집 안에서 발생하는 모든 이벤트를 기록할 수 있다. 이처럼 토픽 설계 시 '가장 구체적인 단위로 데이터를 발행'하는 원칙을 지키면, 다양한 소비 요구사항을 와일드카드를 통해 유연하게 충족시킬 수 있다.


단일 레벨 와일드카드인 플러스 기호(`+`)는 정확히 하나의 토픽 레벨을 대체한다.2 이는 특정 위치의 레벨 값은 상관없지만 전체적인 구조는 일치하는 토픽들을 구독하고자 할 때 유용하다.

- **구독 토픽 필터**: `smart-home/+/temperature`
- **매치되는 토픽**:
  - `smart-home/living-room/temperature`
  - `smart-home/bed-room/temperature`
  - `smart-home/kitchen/temperature`
- **매치되지 않는 토픽**:
  - `smart-home/temperature` (레벨 개수 불일치)
  - `smart-home/living-room/main/temperature` (레벨 개수 불일치)

단일 레벨 와일드카드는 토픽 필터 내 여러 위치에 중복하여 사용할 수도 있다. 예를 들어, `country/+/city/+/sensor` 필터는 `country/korea/city/seoul/sensor`와 `country/usa/city/newyork/sensor` 토픽 모두와 매치된다.


다중 레벨 와일드카드인 해시 기호(`#`)는 0개 이상의 하위 토픽 레벨 전체를 대체한다.2 이 와일드카드는 반드시 토픽 필터의 가장 마지막 문자로 위치해야 하며, 그 앞에는 슬래시(

`/`)가 있어야 한다.

- **구독 토픽 필터**: `smart-home/living-room/#`
- **매치되는 토픽**:
  - `smart-home/living-room/temperature`
  - `smart-home/living-room/light/status`
  - `smart-home/living-room/light/brightness/set`
- **매치되지 않는 토픽**:
  - `smart-home/bed-room/temperature` (시작 경로 불일치)

만약 토픽 필터를 `#` 단독으로 사용하면, 해당 브로커로 전달되는 모든 애플리케이션 메시지(단, `$SYS`로 시작하는 시스템 토픽 제외)를 구독하게 된다.6 이는 디버깅 목적 등 특수한 경우를 제외하고는 브로커와 클라이언트 양쪽에 심각한 성능 부하를 유발할 수 있으므로 실제 운영 환경에서는 사용을 지양해야 하는 대표적인 안티패턴으로 간주된다.6


와일드카드는 매우 유용한 기능이지만, 남용할 경우 시스템에 부정적인 영향을 미칠 수 있다.

- **성능**: 브로커는 메시지를 수신할 때마다 해당 메시지의 토픽과 매치되는 모든 구독(와일드카드 구독 포함)을 찾아내야 한다. 따라서 복잡하고 광범위한 와일드카드 구독이 많아질수록 브로커의 메시지 라우팅 부하가 증가하게 된다. 특히, 광범위한 와일드카드 구독은 클라이언트가 의도치 않은 대량의 메시지를 수신하게 만들어 클라이언트 측의 CPU, 메모리, 네트워크 리소스를 고갈시킬 수 있다.6
- **보안**: 접근 제어 목록(ACL)을 설계할 때 와일드카드를 이용한 구독 권한은 매우 신중하게 부여해야 한다. 예를 들어, 한 사용자에게 `users/+/private-data` 토픽에 대한 구독 권한을 부여하면, 해당 사용자는 다른 모든 사용자의 개인 데이터에 접근할 수 있게 된다. 따라서 와일드카드 구독 권한은 반드시 필요한 최소한의 범위로 제한해야 한다.


성공적인 MQTT 시스템의 기반은 잘 정의된 토픽 구조에 있다. 다음 5가지 핵심 원칙은 시스템의 성능, 확장성, 유지보수성을 보장하는 토픽 설계를 위한 지침을 제공한다. 그러나 이 원칙들은 상호 보완적이면서도 때로는 상충하는 관계에 있다. 예를 들어, '명확성'을 추구하면 토픽이 길어져 '간결성' 원칙과 충돌할 수 있다. 따라서 최적의 설계란 이들 원칙 사이의 '균형점(Trade-off)'을 시스템의 특정 요구사항에 맞게 의식적으로 찾아가는 과정이며, 이것이 바로 아키텍트의 핵심적인 역할이다.


토픽 계층은 가장 일반적인 정보에서 시작하여 가장 구체적인 정보 순으로, 즉 왼쪽에서 오른쪽으로 갈수록 범위가 좁아지도록 구성해야 한다.18 이 원칙은 데이터를 논리적으로 구조화하고 와일드카드를 이용한 필터링을 극대화하는 데 필수적이다.

- **구조**: `[최상위 그룹]/[하위 그룹]/.../[개별 식별자]/[데이터 종류]`
- **좋은 예**: `usa/california/building-15/floor-3/hvac-unit-719/temperature`
- **나쁜 예**: `hvac-unit-719/temperature/floor-3/building-15/california/usa`

'General-to-Specific' 구조를 따르면, 구독자는 와일드카드를 사용하여 원하는 수준의 데이터를 매우 효율적으로 필터링할 수 있다. 예를 들어, `usa/california/building-15/#` 구독으로 특정 건물의 모든 데이터를 수집하거나, `usa/california/+/+/hvac-unit-719/temperature` 구독으로 캘리포니아 내 모든 건물의 특정 HVAC 유닛 온도 데이터만을 집계하는 것이 가능하다.


토픽 이름은 그 자체로 데이터의 의미와 맥락을 명확하게 전달해야 한다. 개발자가 추가적인 문서 없이는 이해하기 어려운 약어나 모호한 이름은 피해야 한다.6

또한, 하나의 토픽은 하나의 명확한 목적만을 가져야 한다. 여러 종류의 데이터를 하나의 일반적인 토픽(예: `device/123/data`)에 JSON 형태로 묶어 보내는 것은 대표적인 안티패턴이다.6 이 방식은 발행 클라이언트의 구현을 단순화시킬 수는 있지만, 특정 데이터(예: 온도)만 필요한 구독자도 불필요한 모든 데이터(습도, 압력 등)를 수신하여 파싱해야 하므로 비효율적이다. 대신 다음과 같이 데이터를 구체적으로 분리해야 한다.

- `device/123/temperature`
- `device/123/humidity`
- `device/123/pressure`

이렇게 토픽을 분리하면 구독자는 자신이 필요한 데이터만 정확히 구독할 수 있어 네트워크 트래픽과 클라이언트의 처리 부하를 최소화할 수 있다.


IoT 시스템은 시간이 지남에 따라 새로운 종류의 센서, 장치, 기능이 추가되며 진화한다. 따라서 초기 토픽 구조 설계 시, 이러한 미래의 확장을 염두에 두어야 한다.6 경직된 구조는 새로운 요구사항이 발생했을 때 전체 토픽 체계를 재설계해야 하는 큰 비용을 초래할 수 있다.

확장성을 확보하는 한 가지 방법은 API 버전 관리와 유사하게 토픽에 버전 정보를 포함하는 것이다.

- **예시**: `v2/devices/{deviceId}/telemetry/temperature`

이 방식은 메시지 페이로드의 스키마가 변경되는 등 이전 버전과 호환되지 않는 큰 변화가 있을 때, 기존 시스템에 영향을 주지 않고 새로운 버전의 토픽을 점진적으로 도입할 수 있게 해준다.


토픽 구조에는 메시지를 발행하는 주체, 즉 디바이스를 고유하게 식별할 수 있는 ID를 포함하는 것이 매우 중요하다.6 이 ID는 디바이스의 시리얼 번호, MAC 주소, 또는 시스템에서 부여한 고유한 클라이언트 ID(Client ID)나 사물 이름(Thing Name)이 될 수 있다.

- **예시**: `devices//status`

토픽에 고유 식별자를 포함하면 다음과 같은 장점이 있다.

1. **디버깅**: 메시지의 출처를 명확히 알 수 있어 문제 발생 시 원인 추적이 용이하다.
2. **보안**: 브로커의 접근 제어(ACL) 기능과 연동하여, 특정 클라이언트 ID를 가진 디바이스는 자신의 ID가 포함된 토픽에만 메시지를 발행할 수 있도록 강제하는 강력한 보안 정책을 구현할 수 있다.10 예를 들어, `client-A`는 `devices/client-A/status`에는 발행할 수 있지만, `devices/client-B/status`에는 발행할 수 없도록 제한할 수 있다.


토픽 이름은 발행되는 모든 메시지에 포함되어 전송되므로, 가능한 한 짧고 간결하게 유지해야 한다.6 이는 네트워크 오버헤드를 줄이는 데 직접적인 영향을 미치며, 특히 대역폭이 제한되고 비용이 비싼 셀룰러나 LPWAN(Low-Power Wide-Area Network) 환경에서 운영되는 디바이스에게는 매우 중요한 요소이다.7

예를 들어, `building` 대신 `bldg`, `temperature` 대신 `temp`, `production` 대신 `prod`와 같이 프로젝트 내에서 일관되게 사용되는 약어를 정의하여 적용할 수 있다. 단, 이 원칙은 '명확성' 원칙과 상충될 수 있으므로, 의미를 해치지 않는 선에서 균형을 맞추는 것이 중요하다.

| 원칙       | 핵심 목표                                   | 설계 고려사항                                                | 모범 예시                                   |
| ---------- | ------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------- |
| **계층성** | 데이터의 논리적 구조화 및 효율적 필터링     | 일반적인 정보에서 구체적인 정보 순으로 구성되었는가?         | `country/city/building/floor/device/metric` |
| **명확성** | 토픽만으로 데이터의 의미와 맥락 전달        | 약어나 모호함 없이 이름이 직관적인가? 하나의 토픽이 하나의 목적만 가지는가? | `.../air-conditioner/temperature`           |
| **확장성** | 미래의 기능 및 장치 추가에 대한 유연성 확보 | 새로운 센서나 데이터 유형을 쉽게 추가할 수 있는 구조인가? 버전 관리가 필요한가? | `v1/devices/{id}/telemetry/pressure`        |
| **식별성** | 메시지 출처의 명확화 및 보안 정책 적용      | 토픽에 발행자의 고유 ID가 포함되어 있는가?                   | `devices/{clientID}/events/status`          |
| **간결성** | 네트워크 오버헤드 최소화                    | 명확성을 해치지 않으면서 최대한 짧게 표현되었는가?           | `bldg/{id}/temp` (프로젝트 내 약어 합의 시) |


앞서 논의한 핵심 원칙들을 실제 시스템에 적용하는 방법을 이해하기 위해, 일반적으로 사용되는 설계 패턴과 구체적인 시나리오 기반의 사례 연구를 살펴본다.


IoT 시스템에서 디바이스와의 통신은 크게 두 가지 방향으로 나뉜다. 하나는 디바이스가 자신의 상태나 센서 값을 주기적으로 보고하는 '데이터(Telemetry)' 흐름이고, 다른 하나는 외부 시스템이 디바이스의 상태를 변경하거나 특정 동작을 수행하도록 지시하는 '명령어(Command)' 흐름이다. 이 두 가지 유형의 메시지는 목적, 중요도, 보안 요구사항이 다르므로 토픽 구조상에서 명확하게 분리해야 한다.18

일반적으로 토픽의 첫 번째 레벨을 사용하여 이를 구분하는 것이 효과적이다.

- **데이터 토픽 예시**: `dt/smart-home/living-room/light-01/status`
- **명령어 토픽 예시**: `cmd/smart-home/living-room/light-01/set`

이러한 분리는 다음과 같은 실질적인 이점을 제공한다.

- **차등적 QoS 적용**: 중요한 명령어 토픽(`cmd/...`)에는 전달을 보장하는 QoS 1 또는 2를 적용하고, 빈번하게 발생하는 데이터 토픽(`dt/...`)에는 성능을 고려하여 QoS 0 또는 1을 적용할 수 있다.
- **세분화된 보안 정책**: 접근 제어(ACL)를 통해 일반 사용자는 모든 데이터 토픽(`dt/#`)을 구독할 수 있지만, 오직 관리자만이 명령어 토픽(`cmd/#`)에 메시지를 발행할 수 있도록 권한을 분리할 수 있다.18


MQTT는 본질적으로 비동기적인 발행-구독 프로토콜이지만, 특정 명령을 내린 후 그 결과를 동기적으로 확인해야 하는 경우가 있다. 예를 들어, 디바이스의 현재 설정 값을 요청하고 그 응답을 받아야 하는 시나리오이다. 이를 위해 명령어 메시지의 페이로드에 '응답을 받을 토픽(response topic)'의 주소를 포함시키는 패턴을 널리 사용한다.

1. **요청 발행**: 요청자(클라이언트 A)가 특정 디바이스의 설정 값을 얻기 위해 명령어 토픽으로 메시지를 발행한다. 이때 페이로드에 자신이 응답을 수신할 고유한 토픽 주소를 포함시킨다.
   - **발행 토픽**: `cmd/device-123/get-config`
   - **페이로드**: `{ "response_topic": "res/client-A/request-456", "correlation_id": "xyz-789" }`
2. **응답 발행**: 요청을 수신한 디바이스 123은 자신의 설정 값을 조회한 후, 요청 페이로드에 명시된 `response_topic`으로 결과를 발행한다. `correlation_id`를 함께 보내주면 요청자가 여러 요청을 비동기적으로 처리할 때 어떤 요청에 대한 응답인지 구분할 수 있다.
   - **발행 토픽**: `res/client-A/request-456`
   - **페이로드**: `{ "correlation_id": "xyz-789", "config": { "interval": 60, "enabled": true } }`

이 패턴을 통해 비동기적인 MQTT 환경 위에서 동기적인 요청/응답 상호작용을 효과적으로 구현할 수 있다.


가장 일반적인 IoT 적용 사례인 스마트 홈을 통해 토픽 설계 원칙과 패턴을 적용해 본다.26

- **요구사항**: 거실과 침실에 각각 조명, 온도/습도 센서, 전동 블라인드가 있다. 각 장치는 개별적으로 제어되어야 하며, 방 단위 또는 집 전체 단위의 일괄 제어도 가능해야 한다. 사용자는 모바일 앱을 통해 현재 상태를 모니터링하고 제어할 수 있다.
- **설계 예시**:

| 장치 (Device)  | 데이터/상태 토픽 (Data/Status Topic)             | 명령어 토픽 (Command Topic)            | 페이로드 예시 (Payload Example)       |
| -------------- | ------------------------------------------------ | -------------------------------------- | ------------------------------------- |
| 거실 조명      | `dt/my-home/living-room/light-01/status`         | `cmd/my-home/living-room/light-01/set` | `"ON"`, `"OFF"`, `{"brightness": 80}` |
| 침실 온도 센서 | `dt/my-home/bed-room/temp-sensor-01/temperature` | (해당 없음)                            | `23.5`                                |
| 침실 습도 센서 | `dt/my-home/bed-room/temp-sensor-01/humidity`    | (해당 없음)                            | `45.2`                                |
| 거실 블라인드  | `dt/my-home/living-room/blind-01/position`       | `cmd/my-home/living-room/blind-01/set` | `50` (0~100), `"OPEN"`, `"CLOSE"`     |

- **활용 시나리오**:
  - **개별 제어**: 사용자가 모바일 앱에서 거실 조명을 켜면, 앱은 `cmd/my-home/living-room/light-01/set` 토픽으로 `"ON"` 메시지를 발행한다.
  - **상태 모니터링**: 대시보드 애플리케이션은 `dt/my-home/+/+/temperature` 토픽을 구독하여 집 안 모든 방의 현재 온도를 실시간으로 표시한다.
  - **그룹 제어**: '외출 모드' 자동화 기능이 실행되면, 시스템은 `cmd/my-home/+/light-01/set` 토픽으로 `"OFF"` 메시지를 발행하여 집 안의 모든 조명을 한 번에 소등한다.


공장 자동화와 같은 산업 환경에서는 더욱 체계적이고 구조화된 토픽 설계가 요구된다.18 산업 자동화 표준인 ISA-95에서 정의한 계층(Enterprise → Site → Area → Line → Cell)을 토픽 구조에 반영하면 매우 효과적이다.

- **요구사항**: 여러 도시에 위치한 공장(Site) 내의 특정 구역(Area)에 있는 생산 라인(Line)의 CNC 기계(Cell) 상태(압력, 온도, 진동)와 생산량 데이터를 실시간으로 수집하고, 필요시 원격으로 비상 정지 명령을 내릴 수 있어야 한다.
- **설계 예시**:
  - **압력 데이터**: `dt/enterprise/site-A/area-3/line-2/cnc-5/telemetry/pressure`
  - **진동 데이터**: `dt/enterprise/site-A/area-3/line-2/cnc-5/telemetry/vibration`
  - **생산량 데이터**: `dt/enterprise/site-A/area-3/line-2/cnc-5/metrics/production-count`
  - **비상 정지 명령**: `cmd/enterprise/site-A/area-3/line-2/cnc-5/emergency-stop`
- **활용 시나리오**:
  - **중앙 관제**: 중앙 관제 시스템은 `dt/enterprise/site-A/#` 토픽을 구독하여 A 공장에서 발생하는 모든 데이터를 모니터링한다.
  - **예지 보전**: 예지 보전 분석 시스템은 `dt/enterprise/+/+/+/+/telemetry/vibration` 토픽을 구독하여 모든 공장의 모든 기계로부터 진동 데이터를 수집하고, 이상 징후를 분석하여 고장을 예측한다.
  - **긴급 대응**: 관리자가 B 공장의 모든 생산 라인을 즉시 중단시켜야 할 경우, `cmd/enterprise/site-B/+/+/+/emergency-stop` 토픽으로 `true` 메시지를 발행한다.


좋은 설계 원칙을 따르는 것만큼이나 잘못된 설계, 즉 안티패턴을 인지하고 피하는 것이 중요하다. 안티패턴은 종종 단기적인 구현의 '편의성'이라는 함정에서 비롯되지만, 장기적으로는 시스템의 성능, 확장성, 안정성을 심각하게 저해하는 기술 부채로 돌아온다.


토픽 이름을 선행 슬래시(`/`)로 시작하는 것(예: `/myhome/livingroom`)은 반드시 피해야 할 대표적인 안티패턴이다.6 이는 루트 디렉토리를 연상시켜 직관적으로 보일 수 있지만, 실제로는 `""`(빈 문자열)이라는 의미 없는 최상위 레벨을 추가하는 결과를 낳는다.10

이로 인해 와일드카드 사용 시 혼란이 발생한다. 예를 들어, `+/livingroom`이라는 와일드카드 구독은 `/myhome/livingroom` 토픽과 매치되지 않는다. 또한 불필요한 레벨 탐색으로 인해 브로커에 미세한 오버헤드를 추가하며, MQTT 표준에서 권장하지 않는 방식이므로 시스템 간 상호운용성을 저해할 수 있다.10


토픽 이름에 공백 문자를 사용하는 것은 절대적으로 금지해야 한다.6 공백은 여러 시스템에서 특별한 의미를 가지는 문자로, URL 인코딩 문제, 예기치 않은 파싱 오류, 디버깅의 어려움 등 수많은 문제를 야기할 수 있다.

마찬가지로, 특수문자나 비-ASCII 문자(예: 한글, 이모지)의 사용도 지양해야 한다.6 이러한 문자들은 특정 클라이언트 라이브러리, 브로커, 로깅 시스템, 데이터베이스 등에서 올바르게 처리되지 않아 데이터 손상이나 시스템 오작동의 원인이 될 수 있다. 항상 가장 보편적이고 안전한 문자 집합(영문 소문자, 숫자, 하이픈)을 사용하는 것이 장기적인 안정성을 보장하는 길이다.


클라이언트, 특히 리소스가 제한된 임베디드 디바이스가 다중 레벨 와일드카드 `#`를 단독으로 구독하는 것은 매우 위험한 안티패턴이다.6 이는 "필요한 데이터를 일일이 구독하는 대신, 모든 것을 다 받아서 클라이언트에서 필터링하자"는 편리함의 유혹에서 비롯되지만, 실제로는 브로커를 통과하는 모든 트래픽을 해당 클라이언트로 집중시키는 결과를 낳는다.

이로 인해 클라이언트는 처리 용량을 초과하는 메시지 폭탄을 맞게 되어 다운되거나, 네트워크 대역폭이 고갈되어 전체 시스템에 영향을 미칠 수 있다. 모든 메시지를 로깅하거나 분석해야 하는 요구사항이 있다면, 이는 클라이언트 레벨이 아닌 브로커의 확장 기능(플러그인)이나 AWS IoT Rules Engine과 같은 백엔드 통합 서비스를 통해 처리하는 것이 올바른 아키텍처 접근 방식이다.6


수많은 디바이스(예: 수천 대의 센서)가 하나의 공유 토픽에 데이터를 발행하고, 단 하나의 클라이언트(예: 모니터링 대시보드)가 그 토픽을 구독하는 '팬인(Fan-in)' 패턴은 해당 구독 클라이언트에 엄청난 부하를 유발한다.18

이 패턴은 해당 클라이언트의 메시지 처리 능력, 네트워크 수신 버퍼, CPU 성능 등을 쉽게 초과하여 메시지 유실이나 시스템 불안정을 초래할 수 있다. 이러한 대규모 데이터 집계 요구사항은 단일 클라이언트가 아닌, Amazon Kinesis, Kafka, 또는 RabbitMQ와 같이 수평적으로 확장이 가능한 백엔드 데이터 처리 시스템으로 라우팅되도록 설계해야 한다.30 MQTT 5.0의 공유 구독 기능 또한 이 문제에 대한 해결책이 될 수 있다.

| 요구사항 (Requirement)         | 안티패턴 (Anti-Pattern)                                      | 문제점 (Problem)                                             | 모범 사례 (Best Practice)                                    |
| ------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **모든 센서 데이터 수집**      | 최종 클라이언트에서 `#` 구독                                 | 클라이언트 과부하, 네트워크 폭증, 시스템 불안정 유발         | 브로커 확장 기능 또는 백엔드 통합(Rules Engine 등) 사용      |
| **다양한 종류의 데이터 전송**  | `device/123/data`와 같이 단일 토픽에 모든 정보를 JSON으로 전송 | 불필요한 데이터 수신으로 인한 대역폭 낭비 및 클라이언트 파싱 부하 증가 | `device/123/temperature`, `device/123/humidity` 등 데이터 종류별로 토픽 분리 |
| **토픽 계층의 시작**           | `/my-system/devices/...` (선행 슬래시 사용)                  | 의미 없는 빈 최상위 레벨 생성, 와일드카드 매칭 혼란, 비표준  | `my-system/devices/...` (선행 슬래시 제거)                   |
| **수천 개 장치의 데이터 집계** | 단일 애플리케이션 클라이언트가 `devices/+/status` 구독 (Fan-in) | 구독 클라이언트 병목 현상, 메시지 유실                       | 확장 가능한 백엔드 서비스(Kinesis, Kafka 등)로 라우팅 또는 MQTT 5.0 공유 구독 활용 |


MQTT 5.0 명세는 기존 버전의 한계를 개선하고 더 복잡한 시스템 요구사항을 지원하기 위한 여러 새로운 기능을 도입했다. 이 중 '토픽 별칭'과 '공유 구독'은 토픽 설계 방식과 전체 시스템 아키텍처에 상당한 영향을 미친다. 이러한 기능들은 MQTT가 단순한 'IoT 프로토콜'을 넘어, 클라우드 백엔드를 포함한 '범용 분산 시스템 메시징 백본'으로 진화하고 있음을 보여준다. 따라서 아키텍트는 이제 디바이스 관점뿐만 아니라 시스템 전체 관점에서 메시징 아키텍처를 설계해야 한다.


MQTT 5.0에서 도입된 토픽 별칭은 긴 토픽 이름을 짧은 2바이트 정수(별칭)에 매핑하여 통신하는 기능이다.31

- **동작 방식**: 클라이언트가 특정 토픽으로 메시지를 처음 발행할 때, PUBLISH 패킷에 전체 토픽 이름과 함께 앞으로 사용할 정수 값의 '토픽 별칭'을 포함하여 보낸다. 브로커는 이 매핑 정보를 해당 연결 세션 동안 기억한다. 이후 클라이언트는 동일한 토픽으로 메시지를 보낼 때, 긴 토픽 이름 대신 이 2바이트짜리 별칭만 포함하여 패킷을 전송한다.
- **설계에 미치는 영향**: 이 기능은 특히 토픽 이름이 길고 동일한 토픽으로 메시지를 반복적으로 발행하는 저대역폭 환경에서 네트워크 오버헤드를 획기적으로 줄여준다.7 이는 제3장의 '간결성' 원칙에 대한 압박을 상당 부분 완화시켜주는 효과가 있다. 즉, 아키텍트는 네트워크 효율성을 위해 토픽의 '명확성'이나 '계층성'을 과도하게 희생할 필요가 줄어든다. 더 서술적이고 명확한 토픽 이름을 사용하면서도, 토픽 별칭 기능을 통해 통신 성능을 유지할 수 있는 유연성을 확보하게 된다.


전통적인 MQTT 모델에서는 하나의 토픽을 여러 클라이언트가 구독할 경우, 해당 토픽으로 발행된 메시지는 모든 구독자에게 복제되어 전달(브로드캐스트)된다. 이는 상태 동기화에는 유용하지만, 여러 작업자(worker)가 작업을 분산 처리하는 데는 비효율적이다.

MQTT 5.0의 공유 구독은 이러한 문제를 해결한다.

- **동작 방식**: 클라이언트는 `$share/{group_name}/{topic_filter}` 형식으로 토픽을 구독한다.32 동일한 

  `{group_name}`을 사용하는 구독자들은 하나의 구독 그룹을 형성한다. 이 그룹의 토픽 필터와 일치하는 메시지가 발행되면, 브로커는 그룹 내의 구독자 중 단 한 명에게만 메시지를 전달한다. 메시지 분배 방식은 브로커 구현에 따라 라운드 로빈, 랜덤, 최소 부하 등 다양할 수 있다.

- **설계에 미치는 영향**: 공유 구독은 전통적인 메시지 큐(Message Queue)의 핵심 기능인 '워크로드 분산'을 MQTT 프로토콜 수준에서 지원한다. 이는 제5장에서 언급된 '팬인(Fan-in)' 안티패턴에 대한 강력한 해결책이 된다. 이전에는 이러한 요구사항을 해결하기 위해 MQTT 브로커와 별도의 메시지 큐 시스템(예: RabbitMQ, SQS)을 연동해야 했지만, 이제는 MQTT 브로커를 중심으로 더 단순하고 확장성 있는 백엔드 아키텍처를 구축할 수 있게 되었다. 이는 MQTT가 IoT 엣지(edge)뿐만 아니라, 클라우드 백엔드에서의 복잡한 분산 처리 시나리오까지 포괄하는 프로토콜로 발전했음을 의미한다.


본 보고서는 MQTT 토픽 설계가 단순한 명명 규칙 제정을 넘어, 전체 IoT 시스템의 성능, 확장성, 보안, 그리고 유지보수성을 결정하는 핵심적인 아키텍처 활동임을 논증했다. 성공적인 MQTT 시스템을 구축하는 것은 단순히 프로토콜을 사용하는 것을 넘어, 그 중심 철학인 '토픽'을 어떻게 설계하고 관리하는지에 달려있다.

보고서 전반에 걸쳐 제시된 5가지 핵심 설계 원칙—**계층성, 명확성, 확장성, 식별성, 간결성**—은 견고한 토픽 구조를 위한 근본적인 지침을 제공한다. 그러나 가장 중요한 것은 이 원칙들 사이의 본질적인 트레이드오프 관계를 이해하고, 주어진 시스템의 고유한 요구사항과 제약 조건에 맞춰 의식적으로 균형점을 찾아가는 것이다. 예를 들어, 극도의 저전력 환경에서는 '간결성'이, 복잡한 권한 관리가 필요한 엔터프라이즈 환경에서는 '식별성'과 '계층성'이 더 높은 우선순위를 가질 수 있다.

또한, 토픽 구조는 한번 설계하고 끝나는 정적인 결과물이 아니라, 시스템의 진화와 함께 지속적으로 관리되어야 하는 동적인 자산이다. 이를 위해 **토픽 거버넌스 및 문서화**는 필수적이다. 토픽은 시스템의 모든 구성 요소가 공유하는 '공공 API'와 같다. 따라서 조직 내에 모든 팀원이 참조할 수 있는 중앙화된 '토픽 레지스트리'나 위키를 구축해야 한다.18 이 레지스트리에는 각 토픽의 정확한 경로, 목적, 페이로드 스키마, 권장 QoS 수준, 그리고 주요 발행자와 구독자가 명확하게 문서화되어야 한다. 이러한 거버넌스 체계가 부재할 경우, 시스템이 복잡해짐에 따라 토픽 구조는 필연적으로 무질서해지고, 결국 막대한 기술 부채로 이어질 것이다.

최종적으로, 아키텍트와 개발자는 본 보고서에서 제시한 원칙, 패턴, 그리고 안티패턴에 대한 지식을 바탕으로 현재의 요구사항을 충실히 만족시키는 동시에, MQTT 5.0의 새로운 기능들을 활용하여 미래의 변화에 유연하게 대응할 수 있는 지속 가능한 토픽 아키텍처를 구축해야 한다. 이것이 바로 성공적인 MQTT 기반 IoT 시스템을 향한 핵심 경로이다.


1. MQTT 개념 (1) - wishlee Log - 티스토리, accessed August 21, 2025, https://wishlee0204.tistory.com/23
2. MQTT 프로토콜: 개념, 동작 원리, 실전 활용과 장단점 완전정리, accessed August 21, 2025, https://notavoid.tistory.com/374
3. What is MQTT? - MQTT Protocol Explained - AWS, accessed August 21, 2025, https://aws.amazon.com/what-is/mqtt/
4. MQTT Publish/Subscribe Architecture (Pub/Sub) – MQTT Essentials: Part 2 - HiveMQ, accessed August 21, 2025, https://www.hivemq.com/blog/mqtt-essentials-part2-publish-subscribe/
5. MQTT와 EMQX - 깊이있게 - 티스토리, accessed August 21, 2025, https://titlejjk.tistory.com/322
6. MQTT Topics, Wildcards, & Best Practices – MQTT Essentials: Part 5, accessed August 21, 2025, https://www.hivemq.com/blog/mqtt-essentials-part-5-mqtt-topics-best-practices/
7. MQTT: A Conceptual Deep-Dive | Hacker News, accessed August 21, 2025, https://news.ycombinator.com/item?id=20526564
8. 무선 압력 및 온도 센서에 MQTT 프로토콜을 어떻게 적용하나요? - SenTec, accessed August 21, 2025, https://cdsentec.com/ko/how-mqtt-protocol-application-in-wireless-pressure-or-temperature-sensor/
9. MQTT 클라이언트를 사용하여 AWS IoT MQTT 메시지 보기, accessed August 21, 2025, https://docs.aws.amazon.com/ko_kr/iot/latest/developerguide/view-mqtt-messages.html
10. Good way to design mqtt topic? - Stack Overflow, accessed August 21, 2025, https://stackoverflow.com/questions/48238365/good-way-to-design-mqtt-topic
11. IOT 통신 프로토콜 MQTT란 무엇일까? (개념/ 송수신 방식/ 예제) - 나의 과거일지, accessed August 21, 2025, https://jeongkyun-it.tistory.com/188
12. Publish MQTT Messages and Subscribe to Message Topics - MATLAB & Simulink, accessed August 21, 2025, https://www.mathworks.com/help/simulink/supportpkg/raspberrypi_ref/publish-and-subscribe-to-mqtt-messages.html
13. MQTT Protocol 이란? - velog, accessed August 21, 2025, [https://velog.io/@iamdudumon/MQTT-Protocol-%EC%9D%B4%EB%9E%80](https://velog.io/@iamdudumon/MQTT-Protocol-이란)
14. MQTT 개념 및 특징 - 공대베짱이, accessed August 21, 2025, https://dejavuhyo.github.io/posts/mqtt-concept/
15. MQTT 소개, accessed August 21, 2025, https://www.joinc.co.kr/w/man/12/MQTT/Tutorial
16. MQTT Publish/Subscribe with Mosquitto Pub/Sub examples | Cedalo, accessed August 21, 2025, https://cedalo.com/blog/mqtt-subscribe-publish-mosquitto-pub-sub-example/
17. [통신 이론] MQTT, MQTT Protocol (MQTT 프로토콜) 이란? - 1 (이론편) - 공대생의 차고, accessed August 21, 2025, https://underflow101.tistory.com/22
18. MQTT design best practices - Designing MQTT Topics for AWS IoT ..., accessed August 21, 2025, https://docs.aws.amazon.com/whitepapers/latest/designing-mqtt-topics-aws-iot-core/mqtt-design-best-practices.html
19. MQTT 클라이언트의 토픽 문자열 및 토픽 필터 - IBM, accessed August 21, 2025, https://www.ibm.com/docs/ko/ibm-mq/9.4.x?topic=concepts-topic-strings-topic-filters-in-mqtt-clients
20. MQTT 주제 - AWS IoT Core, accessed August 21, 2025, https://docs.aws.amazon.com/ko_kr/iot/latest/developerguide/topics.html
21. MQTT Topics and Wildcards: essentials & usage examples | Cedalo, accessed August 21, 2025, https://cedalo.com/blog/mqtt-topics-and-mqtt-wildcards-explained/
22. 5 MQTT: Topics, Wildcards, & Best Practices | Part 5 - DEV Community, accessed August 21, 2025, https://dev.to/hivemq_/mqtt-topics-wildcards-best-practices-part-5-87g
23. MQTT Topic Tree Design best practices, tips & examples - pi3g.com, accessed August 21, 2025, https://pi3g.com/mqtt-topic-tree-design-best-practices-tips-examples/
24. MQTT(Message Queueing Telemetry Transport) - 정보 보관소 - 티스토리, accessed August 21, 2025, https://blaxsior-repository.tistory.com/280
25. MQTT 프로토콜에 대하여 - 수무무일기 - 티스토리, accessed August 21, 2025, https://soobindairy.tistory.com/15
26. [MQTT] MQTT란? (개념, 특징, 장단점, 사용 사례) - 니나노, accessed August 21, 2025, [https://yuna-ninano.tistory.com/entry/MQTT-MQTT%EB%9E%80-%EA%B0%9C%EB%85%90-%ED%8A%B9%EC%A7%95-%EC%9E%A5%EB%8B%A8%EC%A0%90-%EC%82%AC%EC%9A%A9-%EC%82%AC%EB%A1%80](https://yuna-ninano.tistory.com/entry/MQTT-MQTT란-개념-특징-장단점-사용-사례)
27. MQTT 실전 응용하기 — WildCard, accessed August 21, 2025, [https://medium.com/@jspark141515/mqtt-%EC%8B%A4%EC%A0%84-%EC%9D%91%EC%9A%A9%ED%95%98%EA%B8%B0-wildcard-9643877262f2](https://medium.com/@jspark141515/mqtt-실전-응용하기-wildcard-9643877262f2)
28. MQTT Topic Trees - Raspberry Valley, accessed August 21, 2025, https://raspberry-valley.azurewebsites.net/MQTT-Topic-Trees/
29. Arduino 센서와 MQTT를 이용한 스마트 홈 구축 가이드 -, accessed August 21, 2025, [https://byulhablog.com/arduino-%EC%84%BC%EC%84%9C%EC%99%80-mqtt%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EC%8A%A4%EB%A7%88%ED%8A%B8-%ED%99%88-%EA%B5%AC%EC%B6%95-%EA%B0%80%EC%9D%B4%EB%93%9C/](https://byulhablog.com/arduino-센서와-mqtt를-이용한-스마트-홈-구축-가이드/)
30. MQTT communication patterns - Designing MQTT Topics for AWS IoT Core, accessed August 21, 2025, https://docs.aws.amazon.com/whitepapers/latest/designing-mqtt-topics-aws-iot-core/mqtt-communication-patterns.html
31. Topic Alias - MQTT 5.0 new features | EMQ, accessed August 21, 2025, https://www.emqx.com/en/blog/mqtt5-topic-alias
32. Azure Event Grid의 MQTT 브로커 기능에서 지원하는 MQTT 기능 - Microsoft Learn, accessed August 21, 2025, https://learn.microsoft.com/ko-kr/azure/event-grid/mqtt-support
33. 공유 구독 - IBM, accessed August 21, 2025, https://www.ibm.com/docs/ko/SSWMAJ_5.0.0.1/com.ibm.ism.doc/Developing/devsharedsubscriptions.html