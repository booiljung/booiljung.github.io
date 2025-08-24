# Nginx 마스터 클래스
[Nginx](./index.md)


Nginx(엔진엑스로 발음)는 오늘날 웹 인프라의 핵심 구성 요소로 자리 잡았다. 그러나 Nginx를 단순히 '웹 서버'로만 정의하는 것은 그 본질을 충분히 설명하지 못한다. Nginx의 등장은 웹 기술 발전의 필연적인 결과였으며, 그 설계 철학은 현대적인 웹 애플리케이션 아키텍처의 방향성을 제시했다.

1990년대 후반, 인터넷 사용자가 폭발적으로 증가하면서 웹 서버는 이전에 경험하지 못했던 도전에 직면했다. 당시 웹 서버들은 1만 개 이상의 동시 연결(Concurrent Connections)을 효율적으로 처리하지 못했는데, 이 문제를 'C10K 문제(C10K Problem)'라고 부른다.1 이 문제의 근원은 하드웨어 성능 부족이 아니었다. 당시 웹 서버 시장을 지배하던 아파치(Apache)와 같은 서버들은 클라이언트 연결이 발생할 때마다 새로운 프로세스(Process)나 스레드(Thread)를 할당하는 구조를 가졌기 때문이다.3 이러한 구조는 동시 연결 수가 적을 때는 안정적이고 개발이 용이했지만, 수천, 수만 개의 연결이 동시에 발생하자 메모리 사용량이 기하급수적으로 증가하고, 잦은 컨텍스트 스위칭(Context Switching)으로 인해 CPU에 심각한 부하를 유발하며 성능 저하를 일으켰다.3

이러한 아키텍처적 한계를 극복하기 위해 러시아의 소프트웨어 엔지니어 이고르 시쇼브(Igor Sysoev)는 2002년 Nginx 개발을 시작했다.6 그는 대규모 트래픽을 처리하던 러시아의 포털 사이트 램블러(Rambler)의 요구사항을 충족시키기 위해 완전히 새로운 구조의 웹 서버를 구상했다.7 2년간의 개발 끝에 2004년, Nginx는 오픈 소스로 세상에 공개되었다.1 초기 Nginx는 아파치 서버의 앞단에서 정적 콘텐츠(HTML, CSS, 이미지 등)를 처리하여 아파치의 부하를 덜어주는 보조적인 역할로 사용되기 시작했으나, 점차 그 자체로 완전한 기능을 갖춘 웹 서버로 발전했다.6

Nginx의 핵심 설계 철학은 명확했다: **'높은 성능, 높은 동시성, 낮은 메모리 사용량'**.6 이는 다양한 기능과 유연성, 그리고 `.htaccess` 파일을 통한 분산 설정의 편의성에 중점을 둔 아파치와는 근본적으로 다른 접근 방식이었다.4 한 개발자의 비유는 이 철학의 차이를 명확하게 보여준다: "아파치는 백만 가지 옵션을 가진 Microsoft Word와 같지만, 우리는 6가지만 필요하다. Nginx는 그 6가지 작업을 수행하며, 그중 5가지는 아파치보다 50배 빠르다".13

결론적으로 Nginx의 등장은 단순히 더 빠른 웹 서버의 출현이 아니었다. 이는 웹 서버 설계 패러다임의 전환을 의미했다. 과거의 웹 서버들이 시스템 자원이 충분하다는 가정 하에 기능적 풍부함을 추구했다면, Nginx는 폭증하는 트래픽 속에서 제한된 자원을 얼마나 효율적으로 사용할 것인가에 대한 해답을 제시했다. 웹 트래픽의 양적 변화가 웹 서버 아키텍처의 질적 변화를 촉발한 역사적 사례이며, 이는 이후 마이크로서비스나 비동기 프로그래밍 패러다임이 각광받게 된 기술적 배경과도 맥을 같이 한다.

| 항목                   | Nginx                                   | Apache                                   |
| ---------------------- | --------------------------------------- | ---------------------------------------- |
| **아키텍처**           | 이벤트 기반 (Event-Driven)              | 프로세스/스레드 기반                     |
| **연결 처리 방식**     | 단일 스레드 내 다중 연결 비동기 처리    | 연결당 프로세스 또는 스레드 할당         |
| **성능 (정적 콘텐츠)** | 매우 빠름, 적은 리소스 사용             | 상대적으로 느림, 리소스 사용량 많음      |
| **성능 (동적 콘텐츠)** | 외부 프로세서(FastCGI, uWSGI 등)에 위임 | 내장 모듈(`mod_php` 등)로 직접 처리 가능 |
| **설정 방식**          | 중앙 집중식 (`nginx.conf`)              | 분산형 (`.htaccess` 파일 지원)           |
| **모듈 로딩**          | 정적 로딩 (컴파일 시 포함)              | 동적 로딩 (실행 중 필요시 로드)          |
| **OS 지원**            | 모든 Unix 계열, Windows 지원 제한적     | 모든 OS 완벽 지원                        |


Nginx가 수만 개의 동시 연결을 적은 메모리로 처리할 수 있는 비결은 그 독창적인 아키텍처에 있다. Nginx는 전통적인 서버 모델을 답습하는 대신, '이벤트 기반 비동기 논블로킹(Event-driven, Asynchronous, Non-blocking)' 아키텍처를 채택했다. 이 섹션에서는 Nginx 성능의 근간을 이루는 이 아키텍처를 마스터-워커 프로세스 구조와 운영체제(OS) 커널과의 상호작용을 중심으로 심층 분석한다.


Nginx의 가장 큰 특징은 요청마다 새로운 프로세스나 스레드를 생성하지 않는다는 점이다.4 대신, 소수의 워커 프로세스(Worker Process)가 수천, 수만 개의 연결을 동시에 효율적으로 처리한다.14 이는 '이벤트'를 중심으로 동작하기에 가능하다. 여기서 이벤트란 네트워크 연결 수립, 클라이언트로부터 데이터 수신, 데이터 전송 준비 완료 등과 같은 상태 변화를 의미한다.5 워커 프로세스는 이러한 이벤트가 발생했는지 지속적으로 감시하고, 이벤트가 발생한 연결에 대해서만 필요한 작업을 수행한 후 즉시 다른 이벤트를 처리하러 이동한다. 이 방식은 하나의 스레드가 여러 작업을 번갈아 가며 처리하는 비동기(Asynchronous) 모델의 핵심이다.16


Nginx는 실행 시 여러 프로세스를 생성하는데, 이들은 명확한 역할 분담을 통해 안정성과 효율성을 동시에 달성한다.

- **마스터 프로세스(Master Process)**: Nginx가 시작되면 가장 먼저 생성되는 프로세스로, 주로 관리자 역할을 수행한다. 루트(root) 권한으로 실행되어 설정 파일(`nginx.conf`)을 읽고, 80번이나 443번 같은 시스템 포트를 바인딩(binding)한다. 그 후, 실제 요청 처리를 담당할 워커 프로세스들을 생성하는데, 이때 보안을 위해 권한이 낮은 일반 사용자 계정으로 워커 프로세스를 실행시킨다. 마스터 프로세스는 직접 클라이언트 요청을 처리하지 않으며, 대신 워커 프로세스들의 상태를 감시하고, 설정 파일 리로드(`reload`)나 정상 종료(`quit`)와 같은 외부 제어 신호를 받아 워커 프로세스들을 관리하는 역할을 한다.15
- **워커 프로세스(Worker Process)**: 실제 클라이언트의 요청을 받아 처리하는 주체다. 일반적으로 서버의 CPU 코어 수만큼 생성하는 것이 권장되는데, 이는 각 워커 프로세스를 특정 CPU 코어에 고정(pinning)시켜 프로세스 간 컨텍스트 스위칭 비용을 최소화하고 하드웨어 자원을 최대한 효율적으로 사용하기 위함이다.5 각 워커 프로세스는 독립적인 이벤트 루프(Event Loop)를 가지고 있으며, 이 루프를 통해 수천 개의 연결에서 발생하는 이벤트를 비동기적으로 처리한다.15
- **보조 프로세스**: Nginx는 필요에 따라 캐시 관련 보조 프로세스를 실행하기도 한다. `cache loader` 프로세스는 Nginx 시작 시 디스크에 저장된 캐시를 메모리로 미리 불러오는 역할을 하며, `cache manager` 프로세스는 주기적으로 캐시 크기를 확인하여 설정된 용량을 초과하지 않도록 오래된 데이터를 삭제하는 역할을 수행한다.11


워커 프로세스가 단일 스레드로 수많은 연결을 동시에 처리할 수 있는 핵심 기술은 논블로킹 I/O(Non-blocking I/O)와 OS 커널이 제공하는 고성능 이벤트 알림 메커니즘에 있다.

전통적인 블로킹(blocking) 방식에서는 특정 I/O 작업(예: 네트워크 소켓에서 데이터를 읽는 작업)을 요청하면, 데이터가 도착할 때까지 해당 스레드는 다른 어떤 일도 하지 못하고 대기해야 한다. 만약 1만 개의 연결을 이런 방식으로 처리한다면 1만 개의 스레드가 필요하게 되고, 대부분의 스레드는 아무 일도 하지 않고 대기하는 상태로 자원을 낭비하게 된다.

반면, Nginx의 워커 프로세스는 논블로킹 소켓을 사용한다.19 데이터를 읽으라고 요청했을 때 즉시 읽을 데이터가 없으면, 대기하지 않고 즉시 반환받아 다른 작업을 처리한다. 그렇다면 언제 데이터가 도착했는지 어떻게 알 수 있을까? 바로 이 지점에서 OS 커널의 역할이 빛을 발한다. Nginx는 `epoll`(Linux), `kqueue`(BSD 계열)과 같은 OS의 고성능 이벤트 알림 시스템 콜을 사용한다.15 워커 프로세스는 자신이 관리하는 모든 소켓을 OS 커널에 등록하며 "이 소켓들 중 어느 하나라도 읽을 데이터가 생기거나, 쓸 준비가 되거나, 새로운 연결이 들어오는 등의 이벤트가 발생하면 나에게 알려달라"고 요청한다.

이후 워커 프로세스는 자신만의 루프를 도는 것이 아니라, 커널이 알려주는 이벤트가 발생할 때까지 대기한다. 커널은 하드웨어 수준에서 효율적으로 수많은 소켓의 상태 변화를 감지하다가, 이벤트가 발생한 소켓들의 목록을 워커 프로세스에게 한 번에 전달해준다. 워커 프로세스는 이 목록을 받아 순차적으로 해당 소켓에 대한 논블로킹 I/O 작업을 수행하고, 다시 커널의 다음 알림을 기다린다.

이러한 아키텍처는 '작업 위임' 철학의 정수라 할 수 있다. Nginx는 수만 개의 연결을 일일이 감시하고 관리하는 부담스러운 작업을 자신이 직접 처리하는 대신, 이 작업을 가장 효율적으로 수행할 수 있는 OS 커널에게 완전히 위임한다. 그리고 자신은 커널이 알려주는 '실제로 처리할 일'에만 집중함으로써 시스템 전체의 효율을 극대화한다. 이것이 바로 Nginx가 적은 리소스로 높은 성능을 낼 수 있는 근본적인 이유다.


Nginx의 강력한 아키텍처를 이해했다면, 이제 실제 서버에 설치하고 운영하는 방법을 알아볼 차례다. 이 섹션에서는 가장 널리 사용되는 리눅스 배포판인 Ubuntu와 CentOS를 기준으로 설치 과정과 서비스 관리 방법을 단계별로 설명한다.


Ubuntu 및 Debian 계열 배포판에서는 `apt` 패키지 관리자를 통해 매우 간단하게 Nginx를 설치할 수 있다.

1. **패키지 목록 갱신**: 설치에 앞서 로컬 패키지 인덱스를 최신 상태로 업데이트한다.

   ```Bash
   sudo apt update
   ```

   22

2. **Nginx 설치**: 다음 명령어로 Nginx와 관련 의존성 패키지들을 설치한다.

   ```Bash
   sudo apt install nginx
   ```

   22

3. **방화벽 설정**: 서버에 `ufw`(Uncomplicated Firewall) 방화벽이 활성화되어 있다면, 외부에서 웹 서버에 접근할 수 있도록 HTTP 및 HTTPS 트래픽을 허용해야 한다. Nginx는 설치 시 `ufw`에 몇 가지 애플리케이션 프로필을 등록하므로 쉽게 설정할 수 있다.

   - 사용 가능한 프로필 확인:

     ```Bash
     sudo ufw app list
     ```

     출력 결과에 `Nginx HTTP`(80번 포트), `Nginx HTTPS`(443번 포트), `Nginx Full`(80, 443번 포트 모두)이 포함된 것을 확인할 수 있다.22

   - HTTP와 HTTPS 트래픽 모두 허용:

     ```Bash
     sudo ufw allow 'Nginx Full'
     ```

     24

   - 방화벽 상태 확인:

     ```Bash
     sudo ufw status
     ```

     규칙이 정상적으로 추가되었는지 확인한다.22


CentOS 및 RHEL 계열 배포판에서는 `yum` 또는 `dnf` 패키지 관리자를 사용한다. 공식 저장소에 Nginx가 포함되어 있지 않을 수 있으므로, EPEL(Extra Packages for Enterprise Linux) 저장소를 먼저 추가해야 한다.

1. **EPEL 저장소 설치**: 다음 명령어로 EPEL 저장소를 시스템에 추가한다.

   ```Bash
   sudo yum install epel-release
   ```

   (CentOS 8 이상에서는 `sudo dnf install epel-release`) 25

2. **Nginx 설치**: EPEL 저장소가 추가되면 `yum` 또는 `dnf`를 통해 Nginx를 설치할 수 있다.

   ```Bash
   sudo yum install nginx
   ```

   (CentOS 8 이상에서는 `sudo dnf install nginx`) 26

3. **방화벽 설정**: CentOS/RHEL은 기본적으로 `firewalld` 방화벽을 사용한다.

   - HTTP 및 HTTPS 서비스 영구적으로 허용:

     ```Bash
     sudo firewall-cmd --permanent --add-service=http
     sudo firewall-cmd --permanent --add-service=https
     ```

     26

   - 방화벽 설정 다시 불러오기:

     ```Bash
     sudo firewall-cmd --reload
     ```

     27


Nginx는 `systemd`를 통해 서비스로 관리된다. 다음은 `systemctl` 명령어를 사용한 주요 관리 작업이다.

- **서비스 시작**: `sudo systemctl start nginx` 24
- **서비스 중지**: `sudo systemctl stop nginx` 24
- **서비스 상태 확인**: `sudo systemctl status nginx` 22
- **부팅 시 자동 시작 설정**: `sudo systemctl enable nginx` 23
- **부팅 시 자동 시작 비활성화**: `sudo systemctl disable nginx` 23


Nginx 설정을 변경한 후 이를 적용하기 위해 `restart`와 `reload` 두 가지 명령어를 사용할 수 있다. 이 둘의 차이를 이해하는 것은 운영 환경에서 매우 중요하다.

- `sudo systemctl restart nginx`: 이 명령어는 Nginx 서비스를 완전히 중지시켰다가 다시 시작한다. 즉, 기존의 마스터 프로세스와 모든 워커 프로세스를 종료(kill)하고, 새로운 프로세스들을 생성한다. 이 과정에서 아주 짧은 시간 동안 서비스가 중단(downtime)될 수 있다.23
- `sudo systemctl reload nginx`: 이 명령어는 서비스 중단 없이 설정을 적용하는 '우아한(graceful)' 재시작 방식이다. 내부적으로는 실행 중인 마스터 프로세스에 `SIGHUP` 시그널을 보낸다.29 시그널을 받은 마스터 프로세스는 다음과 같이 동작한다:
  1. 새로운 설정 파일을 읽고 문법적 유효성을 검사한다.
  2. 문법에 이상이 없으면, 새로운 설정을 적용한 새로운 워커 프로세스들을 생성한다.
  3. 기존 워커 프로세스들에게는 더 이상 새로운 연결을 받지 말고, 현재 처리 중인 요청들만 모두 완료한 후 정상적으로 종료하라는 신호를 보낸다.
  4. 이 과정을 통해 기존 연결은 끊김 없이 처리되면서 새로운 연결은 새 설정이 적용된 워커 프로세스가 처리하게 되어, 무중단으로 설정을 변경할 수 있다.29

이러한 무중단 `reload` 기능은 Nginx의 마스터-워커 아키텍처가 제공하는 핵심적인 운영상의 이점이다. 따라서 운영 중인 서비스에 설정 변경을 적용할 때는 특별한 경우가 아니라면 항상 `restart` 대신 `reload`를 사용하는 것이 원칙이다.


Nginx의 모든 동작은 `nginx.conf`라는 텍스트 기반 설정 파일을 통해 제어된다. 이 파일의 구조와 문법을 이해하는 것은 Nginx를 효과적으로 활용하기 위한 필수 관문이다. Nginx 설정은 단순한 키-값 쌍의 나열이 아니라, 명확한 계층 구조와 상속 규칙을 가진 논리적인 시스템이다.


Nginx 설정 파일은 중괄호(`{}`)로 둘러싸인 블록들의 집합으로 구성된다. 이 중 다른 지시어(Directive)들을 내부에 포함할 수 있는 특별한 블록을 **컨텍스트(Context)**라고 부른다.30 컨텍스트는 설정을 기능별, 영역별로 구분하는 역할을 하며, 다음과 같은 계층 구조를 가진다.

```
main (global context)
├── events {... }
└── http {
   ...
    ├── server {
    │  ...
    │   └── location {... }
    │   }
    └── server {
       ...
    }
}
```

이 구조의 핵심 원리는 **상속(Inheritance)**과 **재정의(Override)**다. 상위 컨텍스트에 정의된 지시어의 값은 하위 컨텍스트로 상속된다. 만약 하위 컨텍스트에서 동일한 지시어가 다시 정의되면, 하위 컨텍스트의 값이 상위의 값을 재정의하여 적용된다.32

그러나 이 규칙에는 중요한 예외가 있다. 예를 들어, `proxy_set_header`와 같은 일부 지시어는 하위 컨텍스트에 단 하나라도 정의되면 상위 컨텍스트의 모든 `proxy_set_header` 설정이 무시되고 상속되지 않는다.35 이는 단순한 재정의가 아니라 상속 체계 자체가 차단되는 것으로, "가장 구체적인(specific) 블록의 설정이 해당 요청에 대한 모든 권한을 가진다"는 Nginx의 설계 철학을 보여준다. 이러한 특성을 이해하지 못하면 예상치 못한 설정 오류를 겪을 수 있으므로, 공통 설정은 `include` 지시어를 사용하여 별도 파일로 관리하고 필요한 곳에서 불러오는 것이 좋은 방법이다.



설정 파일의 최상위 영역으로, 특정 컨텍스트 블록에 속하지 않는 전역 설정을 담당한다.32

- `user`: 워커 프로세스가 실행될 사용자 계정과 그룹을 지정한다. 보안을 위해 `nginx`나 `www-data`와 같은 권한이 낮은 사용자를 지정하는 것이 일반적이다.36
- `worker_processes`: 실행할 워커 프로세스의 개수를 지정한다. `auto`로 설정하면 시스템의 CPU 코어 수에 맞게 자동으로 조정된다.30
- `error_log`: 에러 로그를 기록할 파일 경로와 로깅 레벨(예: `warn`, `error`, `debug`)을 지정한다.36
- `pid`: 마스터 프로세스의 프로세스 ID(PID)가 저장될 파일 경로를 지정한다.30


네트워크 연결 처리 방식에 대한 전반적인 설정을 정의한다.32

- `worker_connections`: 워커 프로세스 하나가 동시에 처리할 수 있는 최대 연결 수를 설정한다. 서버가 처리할 수 있는 총 동시 연결 수는 이 값에 `worker_processes` 값을 곱한 것과 같다 (`worker_processes * worker_connections`).36
- `use`: 사용할 이벤트 처리 모델을 지정한다. 리눅스에서는 `epoll`, BSD 계열에서는 `kqueue`가 주로 사용되며, 보통 Nginx가 OS에 맞춰 자동으로 최적의 모델을 선택하므로 명시적으로 설정할 필요는 거의 없다.40


웹 서버 기능과 관련된 대부분의 설정을 포함하는 가장 중요한 컨텍스트다.32

- `include`: 외부 설정 파일을 불러와 설정을 모듈화한다. 예를 들어, `include /etc/nginx/mime.types;`는 파일 확장자에 따른 MIME 타입을 정의하고, `include /etc/nginx/conf.d/*.conf;`는 `conf.d` 디렉토리의 모든 `.conf` 파일을 포함시켜 가상 호스트 설정을 분리 관리하는 데 사용된다.36
- `access_log`: 클라이언트의 접근 로그를 기록할 파일 경로와 포맷을 지정한다.41
- `sendfile`, `tcp_nopush`, `keepalive_timeout` 등 다양한 성능 최적화 관련 지시어들이 이 컨텍스트에 위치한다.16


하나의 가상 호스트(예: `example.com`이라는 웹사이트)를 정의하는 단위다.18

`http` 컨텍스트 내에 여러 개의 `server` 블록을 정의하여 하나의 Nginx 인스턴스로 여러 웹사이트를 호스팅할 수 있다.

- `listen`: 서버가 요청을 수신할 IP 주소와 포트를 지정한다. `listen 80;`은 HTTP, `listen 443 ssl;`은 HTTPS 요청을 의미한다.42
- `server_name`: 이 `server` 블록이 어떤 도메인 이름으로 들어오는 요청을 처리할지 지정한다. 여러 도메인을 공백으로 구분하여 나열하거나, 와일드카드(`*.example.com`) 또는 정규표현식(`~^www\d+\.example\.com$`)을 사용할 수 있다.42


`server` 블록 내에서 특정 요청 URI(URL의 경로 부분)를 어떻게 처리할지 세부적으로 정의한다.30 Nginx의 라우팅 기능의 핵심이다.

- **URI 매칭 규칙**: `location` 지시어는 URI와 매칭되는 방식에 따라 다양한 수정자(modifier)를 가질 수 있다.
- **매칭 우선순위**: 클라이언트 요청이 들어왔을 때, Nginx는 정해진 우선순위에 따라 URI와 가장 잘 맞는 `location` 블록 하나를 선택하여 요청을 처리한다. 그 순서는 다음과 같다 43:
  1. `=` 수정자를 사용한 정확히 일치하는 `location` 블록.
  2. `^~` 수정자를 사용한 가장 긴 접두사 일치 `location` 블록. (이 블록이 선택되면 정규식 검사는 더 이상 수행되지 않는다.)
  3. `~` 또는 `~*` 수정자를 사용한 정규식 일치 `location` 블록. (설정 파일에 정의된 순서대로 검사하며, 처음 일치하는 블록이 선택된다.)
  4. 수정자가 없는 가장 긴 접두사 일치 `location` 블록.

이러한 `location` 블록의 매칭 규칙과 우선순위를 정확히 이해하는 것은 복잡한 라우팅 로직을 구현하는 데 매우 중요하다.

| 우선순위 | 수정자     | 설명                                                         | 예시                   | 비고                                                         |
| -------- | ---------- | ------------------------------------------------------------ | ---------------------- | ------------------------------------------------------------ |
| **1**    | `=`        | URI가 문자열과 정확히 일치해야 함                            | `location = /login`    | 가장 먼저 처리되며, 일치 시 검색 종료                        |
| **2**    | `^~`       | URI가 이 접두사로 시작할 경우, 이 블록이 선택되고 정규식 검사를 중단함 | `location ^~ /images/` | 정적 파일 경로에 사용하여 불필요한 정규식 검사를 피할 때 유용 |
| **3**    | `~` / `~*` | 정규식 일치 (각각 대소문자 구분/미구분)                      | `location ~ \.php$`    | 설정 파일에 정의된 순서대로 검사하여 첫 번째 일치 항목 사용  |
| **4**    | `(없음)`   | URI가 접두사와 일치함                                        | `location /`           | 모든 정규식이 일치하지 않을 경우, 가장 길게 일치하는 접두사 블록 사용 |


이론적 배경을 바탕으로, 이제 Nginx의 핵심 기능들을 실제 설정 파일 예제와 함께 구체적으로 다룬다. 정적 콘텐츠 제공부터 리버스 프록시, 로드 밸런싱, 캐싱, HTTPS 설정, Gzip 압축까지 실제 운영 환경에서 필수적인 기능들을 상세히 설명한다.


Nginx의 가장 기본적인 역할이자 가장 뛰어난 성능을 발휘하는 분야는 정적 파일(HTML, CSS, JavaScript, 이미지 등)을 클라이언트에게 제공하는 것이다.


가장 간단한 정적 웹사이트 설정은 `server` 블록 내에 `root`와 `index` 지시어를 사용하는 것이다.

```Nginx
server {
    listen 80;
    server_name example.com www.example.com;

    root /var/www/example.com/html;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

- `root /var/www/example.com/html;`: 이 `server` 블록으로 들어오는 요청에 대한 파일 시스템의 최상위 경로를 지정한다.42
- `index index.html index.htm;`: 클라이언트가 `/` 와 같이 디렉토리를 요청했을 때 기본으로 보여줄 파일을 순서대로 지정한다.29
- `location / {... }`: 모든 요청에 대한 기본 처리 방식을 정의한다. `try_files $uri $uri/ =404;`는 먼저 요청된 URI와 일치하는 파일(`$uri`)을 찾고, 없으면 디렉토리(`$uri/`)를 찾고(이때 `index` 지시어에 명시된 파일이 사용됨), 그마저도 없으면 404 에러를 반환하라는 의미다.42


`location` 블록 내에서 파일 경로를 지정할 때 `root`와 `alias`는 비슷해 보이지만 결정적인 차이가 있어 주의해야 한다.

- **`root`**: `location`에서 일치한 URI 경로가 `root`로 지정된 경로 뒤에 그대로 **덧붙여진다**.

  ```Nginx
  location /images/ {
      root /data/www;
  }
  # 요청: http://example.com/images/logo.png
  # 실제 파일 경로: /data/www/images/logo.png
  ```

  위 예시에서 `/images/`가 `/data/www` 뒤에 붙어 최종 경로가 형성된다.49

- **`alias`**: `location`에서 일치한 URI 경로가 `alias`로 지정된 경로로 **대체된다**.

  ```Nginx
  location /images/ {
      alias /data/www/assets/;
  }
  # 요청: http://example.com/images/logo.png
  # 실제 파일 경로: /data/www/assets/logo.png
  ```

  위 예시에서는 `/images/` 부분이 `/data/www/assets/`로 대체되었다.50

  `alias`는 주로 URI 구조와 실제 파일 시스템 구조가 다를 때 유용하게 사용된다.


리버스 프록시는 클라이언트의 요청을 직접 처리하지 않고, 내부 네트워크의 다른 서버(백엔드 서버 또는 WAS)로 전달하고 그 응답을 다시 클라이언트에게 반환하는 중개자 역할을 한다. 이를 통해 백엔드 서버의 IP를 숨겨 보안을 강화하고, 여러 서버로 부하를 분산하며, SSL 암호화/복호화 작업을 대신 처리하는 등 다양한 이점을 제공한다.1


`location` 블록 내의 `proxy_pass` 지시어를 사용하여 리버스 프록시를 설정한다.

```Nginx
location /api/ {
    proxy_pass http://127.0.0.1:8080;
}
```

위 설정은 `/api/`로 시작하는 모든 요청을 로컬호스트의 8080 포트에서 실행 중인 백엔드 서버로 전달한다.55


`proxy_pass`에 지정된 URL 끝에 슬래시(`/`)가 있는지 없는지에 따라 백엔드로 전달되는 URI가 달라지므로 매우 주의해야 한다.57

1. **슬래시가 없는 경우**: `location`에서 일치한 URI가 그대로 전달된다.

   ```Nginx
   location /app/ {
       proxy_pass http://backend;
   }
   # 요청: /app/page -> 백엔드 전달 URI: /app/page
   ```

2. **슬래시가 있는 경우**: `location`에서 일치한 부분이 `proxy_pass`의 경로로 대체된 후 나머지 URI가 붙는다.

   ```Nginx
   location /app/ {
       proxy_pass http://backend/;
   }
   # 요청: /app/page -> 백엔드 전달 URI: /page
   ```


백엔드 서버는 클라이언트가 아닌 Nginx로부터 요청을 받기 때문에, 클라이언트의 원래 정보를 알 수 없다. `proxy_set_header` 지시어를 사용해 필요한 정보를 HTTP 헤더에 담아 전달해야 한다.

```Nginx
location / {
    proxy_pass http://app_server;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

- `proxy_set_header Host $host;`: 클라이언트가 요청한 원래의 호스트 이름(도메인)을 전달한다.56
- `proxy_set_header X-Real-IP $remote_addr;`: 클라이언트의 실제 IP 주소를 `X-Real-IP` 헤더에 담아 전달한다.59
- `proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;`: 요청이 여러 프록시를 거칠 경우, 거쳐온 모든 프록시 서버의 IP 주소 목록을 전달한다. `$proxy_add_x_forwarded_for` 변수는 기존 `X-Forwarded-For` 헤더 값에 Nginx의 IP를 추가해준다.61
- `proxy_set_header X-Forwarded-Proto $scheme;`: 클라이언트가 Nginx에 요청한 원래 프로토콜(http 또는 https)을 전달한다. 백엔드에서 HTTPS 리디렉션 등을 처리할 때 필요하다.63


로드 밸런싱은 들어오는 트래픽을 여러 대의 백엔드 서버에 분산시켜 단일 서버의 과부하를 방지하고, 서비스의 가용성과 확장성을 높이는 기술이다. Nginx는 `upstream` 모듈을 통해 강력한 로드 밸런싱 기능을 제공한다.


`http` 컨텍스트 내에 `upstream` 블록을 정의하여 로드 밸런싱할 서버 그룹을 만든다.

```Nginx
http {
    upstream my_backend {
        server backend1.example.com;
        server 192.168.0.2:8080;
        server unix:/tmp/backend.sock;
    }

    server {
        listen 80;
        location / {
            proxy_pass http://my_backend;
        }
    }
}
```

`upstream` 블록에 `my_backend`라는 이름의 서버 그룹을 정의하고, `server` 지시어로 그룹에 속할 서버들을 나열했다. 서버 주소는 도메인, IP 주소, 유닉스 소켓 등 다양한 형태로 지정할 수 있다. `proxy_pass` 지시어에는 개별 서버 주소 대신 `upstream` 블록의 이름을 지정한다.64


Nginx는 여러 가지 트래픽 분산 알고리즘을 제공한다.

- **Round Robin (기본값)**: 특별한 지시어가 없으면 기본으로 사용된다. 서버 목록을 위에서부터 아래로 순서대로 돌아가며 요청을 분배한다. 가장 간단하고 예측 가능한 방식이다.65

  - 서버 Si가 선택될 확률 $P(S_i)$는 가중치 wi에 비례한다. (가중치가 모두 1일 경우)

    $P(S_i) = \frac{1}{N}$

- **Least Connections**: `least_conn;` 지시어를 `upstream` 블록에 추가한다. 현재 활성 연결(active connections) 수가 가장 적은 서버로 다음 요청을 보낸다. 각 요청의 처리 시간이 다를 때 부하를 더 공평하게 분산시킬 수 있다.65

  - 다음 요청은 $\min(C_1, C_2,..., C_N)$을 만족하는 서버로 전달된다. 여기서 Ci는 서버 i의 현재 연결 수다.

    $\text{Next Server} = \arg\min_{S_i} (C_i)$

- **IP Hash**: `ip_hash;` 지시어를 추가한다. 클라이언트의 IP 주소를 해싱하여 그 결과값에 따라 특정 서버에만 요청을 보낸다. 이를 통해 특정 클라이언트는 항상 같은 서버에 연결되도록 보장할 수 있다(세션 고정 또는 Sticky Session). 사용자의 로그인 정보와 같이 세션 상태를 서버 메모리에 저장하는 애플리케이션에 유용하다.65

  - 서버 인덱스는 $H(\text{IP}_{\text{client}}) \pmod{N}$으로 결정된다. 여기서 H는 해시 함수, N은 서버의 총 개수다.

    $\text{Server Index} = H(\text{IP}_{\text{client}}) \pmod{N}$

| 알고리즘              | 동작 방식                              | 장점                                                  | 단점                                        | 추천 사용 사례                                               |
| --------------------- | -------------------------------------- | ----------------------------------------------------- | ------------------------------------------- | ------------------------------------------------------------ |
| **Round Robin**       | 서버 목록을 순차적으로 순환            | 설정 간단, 예측 가능                                  | 서버의 현재 부하 상태를 고려하지 않음       | Stateless 애플리케이션, 모든 서버 사양이 동일한 환경         |
| **Least Connections** | 활성 연결이 가장 적은 서버 우선        | 처리 시간이 불규칙한 요청에 대해 부하를 공평하게 분산 | 세션 유지가 보장되지 않음                   | 파일 다운로드, DB 쿼리 등 처리 시간이 다양한 서비스          |
| **IP Hash**           | 클라이언트 IP 해시 기반 고정 서버 할당 | 세션 유지(Sticky Session) 가능                        | 특정 IP 대역에서 트래픽 쏠림 현상 발생 가능 | 로그인, 장바구니 등 세션 유지가 필수적인 Stateful 애플리케이션 |


`server` 지시어에 다양한 옵션을 추가하여 로드 밸런싱 동작을 세밀하게 제어할 수 있다.

- `weight=number`: 서버별로 가중치를 부여한다. 가중치가 높은 서버가 더 많은 요청을 비례하여 받게 된다. 서버의 처리 성능이 다를 때 유용하다.72
- `max_fails=number` 및 `fail_timeout=time`: `fail_timeout` 시간 동안 `max_fails` 횟수만큼 서버 연결에 실패하면 해당 서버를 장애 상태로 간주하고 로드 밸런싱에서 제외한다.66
- `backup`: 다른 모든 서버가 장애 상태일 때만 요청을 받는 예비 서버로 지정한다.73


Nginx는 강력한 콘텐츠 캐싱 기능을 제공한다. 반복적으로 요청되는 콘텐츠를 Nginx에 임시 저장(캐시)해두고, 동일한 요청이 다시 들어오면 백엔드 서버까지 가지 않고 Nginx가 직접 응답함으로써 응답 속도를 획기적으로 개선하고 백엔드 서버의 부하를 줄일 수 있다.75


캐시 설정은 주로 두 개의 지시어를 통해 이루어진다.

1. **`proxy_cache_path`**: `http` 컨텍스트 내에서 캐시의 물리적인 속성을 정의한다.

   ```Nginx
   http {
       proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=1g inactive=60m;
      ...
   }
   ```

   - `/var/cache/nginx`: 캐시된 파일이 저장될 디스크 경로.

   - `levels=1:2`: 캐시 파일을 저장할 하위 디렉토리 구조. 효율적인 파일 검색을 위해 사용된다.

   - `keys_zone=my_cache:10m`: 캐시 키(어떤 요청이 캐시되었는지에 대한 정보)와 메타데이터를 저장할 공유 메모리 공간의 이름(`my_cache`)과 크기(`10m`)를 지정한다. 이 메모리 공간은 빠른 캐시 조회를 위해 사용된다.

   - `max_size=1g`: 디스크에 저장될 캐시의 최대 크기를 1GB로 제한한다.

   - inactive=60m: 60분 동안 접근되지 않은 캐시 데이터는 삭제된다.

     59

2. **`proxy_cache`**: `server` 또는 `location` 블록 내에서 캐시를 활성화한다.

   ```Nginx
   server {
      ...
       location / {
           proxy_pass http://my_backend;
           proxy_cache my_cache; # 'my_cache'라는 이름의 keys_zone을 사용
       }
   }
   ```

   `proxy_cache_path`에서 정의한 `keys_zone`의 이름을 지정하여 해당 캐시 설정을 사용하도록 한다.59


- `proxy_cache_valid`: HTTP 응답 코드별로 캐시의 유효 기간을 설정한다.

  ```Nginx
  proxy_cache_valid 200 302 10m;  # 200, 302 응답은 10분간 캐시
  proxy_cache_valid 404      1m;   # 404 응답은 1분간 캐시
  ```

  78

- `proxy_cache_key`: 캐시를 구분하는 고유 키를 생성하는 규칙을 정의한다. 기본값 외에 쿠키 값이나 다른 변수를 추가하여 더 세밀한 캐싱 제어가 가능하다.

  ```Nginx
  proxy_cache_key "$scheme$proxy_host$request_uri$is_args$args"; # 기본값
  proxy_cache_key "$host$request_uri$cookie_user"; # user 쿠키 값에 따라 캐시를 분리
  ```

  59

- **캐시 상태 확인**: 디버깅을 위해 응답 헤더에 캐시 상태(HIT, MISS, BYPASS, EXPIRED 등)를 추가하면 유용하다.

  ```Nginx
  add_header X-Cache-Status $upstream_cache_status;
  ```

  82


현대 웹에서 HTTPS는 선택이 아닌 필수다. Nginx는 Let's Encrypt와 Certbot을 통해 무료로 SSL/TLS 인증서를 발급받고 적용하여 손쉽게 HTTPS 통신을 구축할 수 있도록 지원한다.


HTTPS 서버를 구성하려면 `server` 블록의 `listen` 지시어에 `ssl` 파라미터를 추가하고, 인증서와 개인 키 파일의 경로를 지정해야 한다.

```Nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    # 기타 SSL/TLS 최적화 설정
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'HIGH:!aNULL:!MD5';
}
```

- `listen 443 ssl;`: 443번 포트에서 SSL/TLS 연결을 수신하도록 설정한다.83
- `ssl_certificate`: 인증 기관으로부터 발급받은 인증서 파일의 경로를 지정한다. Let's Encrypt의 경우 보통 `fullchain.pem` 파일을 사용한다.85
- `ssl_certificate_key`: 인증서에 대한 개인 키 파일의 경로를 지정한다. Let's Encrypt의 경우 `privkey.pem` 파일이다.85


Certbot은 Let's Encrypt 인증서의 발급, 설치, 갱신 과정을 자동화해주는 클라이언트 프로그램이다.87

1. **Certbot 설치 (Ubuntu 기준)**: Certbot과 Nginx 플러그인을 설치한다.

   ```Bash
   sudo apt update
   sudo apt install certbot python3-certbot-nginx
   ```

   88

2. **인증서 발급 및 Nginx 설정 자동화**: 다음 명령어를 실행하면 Certbot이 Nginx 설정 파일을 분석하여 `server_name`에 맞는 도메인에 대한 인증서를 발급하고, HTTPS 설정을 자동으로 추가해준다.

   ```Bash
   sudo certbot --nginx -d example.com -d www.example.com
   ```

   89

   이 과정에서 이메일 주소 입력, 서비스 약관 동의, 그리고 HTTP 요청을 HTTPS로 자동 리디렉션할지 여부를 묻는다. 'Redirect' 옵션을 선택하는 것이 일반적이다.

3. **자동 갱신**: Let's Encrypt 인증서는 유효 기간이 90일로 짧지만, Certbot은 설치 시 시스템의 `cron`이나 `systemd timer`에 자동 갱신 스크립트를 등록한다. 이 스크립트는 하루에 두 번 실행되어 만료가 임박한(보통 30일 이내) 인증서를 자동으로 갱신하므로, 사용자가 신경 쓸 필요가 거의 없다.88 자동 갱신이 정상적으로 동작하는지 테스트하려면 다음 명령어를 사용한다.

   ```Bash
   sudo certbot renew --dry-run
   ```

   88


Gzip은 텍스트 기반의 콘텐츠(HTML, CSS, JavaScript, JSON, XML 등)를 서버에서 클라이언트로 전송하기 전에 압축하여 데이터 전송량을 줄이는 기술이다. 이를 통해 페이지 로딩 속도를 개선하고 네트워크 대역폭을 절약할 수 있다.93


`http` 블록 내에 다음과 같은 지시어들을 추가하여 Gzip 압축을 설정할 수 있다.

```Nginx
http {
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_min_length 256;
    gzip_types
        text/plain
        text/css
        application/json
        application/javascript
        application/x-javascript
        text/xml
        application/xml
        application/xml+rss
        text/javascript
        image/svg+xml;
   ...
}
```

- `gzip on;`: Gzip 압축 기능을 활성화한다.96
- `gzip_types...;`: 압축을 적용할 콘텐츠의 MIME 타입을 지정한다. `text/html`은 `gzip on;`만으로도 기본적으로 압축되지만, CSS, JavaScript, JSON 등 다른 텍스트 기반 파일들은 여기에 명시해주어야 한다.97
- `gzip_comp_level 6;`: 압축 레벨을 1에서 9 사이의 값으로 설정한다. 숫자가 높을수록 압축률이 좋아지지만, 서버의 CPU 사용량도 함께 증가한다. 일반적으로 5 또는 6이 성능과 압축률 사이의 좋은 절충점으로 여겨진다.97
- `gzip_min_length 256;`: 지정된 바이트 크기보다 작은 파일은 압축하지 않는다. 매우 작은 파일은 압축해도 크기 감소 효과가 미미하거나 오히려 커질 수 있기 때문에, 불필요한 CPU 낭비를 막기 위한 설정이다.97
- `gzip_vary on;`: 응답 헤더에 `Vary: Accept-Encoding`을 추가한다. 이는 중간 프록시 서버(예: CDN)가 압축된 버전과 압축되지 않은 버전을 별도로 캐시하도록 하여, Gzip을 지원하지 않는 소수의 클라이언트에게 압축된 콘텐츠가 잘못 전달되는 문제를 방지한다.94
- `gzip_proxied any;`: Nginx가 리버스 프록시로 동작할 때, 백엔드 서버로부터 받은 응답에 대해서도 Gzip 압축을 수행하도록 설정한다.97


본 자습서를 통해 Nginx의 탄생 배경부터 핵심 아키텍처, 설치 및 관리, 그리고 주요 기능별 상세 설정 방법까지 포괄적으로 살펴보았다. Nginx의 성공은 단순히 빠른 속도 때문이 아니라, C10K 문제라는 시대적 과제를 해결하기 위해 고안된 혁신적인 **이벤트 기반 비동기 아키텍처**에 근간을 두고 있음을 확인했다. 이 아키텍처는 제한된 서버 자원으로 수많은 동시 연결을 효율적으로 처리하는 능력을 부여했으며, 이는 고성능 정적 콘텐츠 서빙, 유연한 리버스 프록시, 강력한 로드 밸런싱 등 Nginx의 모든 핵심 기능의 기반이 되었다.

Nginx의 역할은 시대의 변화와 함께 계속해서 진화해왔다. 오늘날 웹 아키텍처의 주류로 자리 잡은 마이크로서비스(Microservices)와 컨테이너(Container) 환경에서 Nginx는 과거의 '웹 서버'라는 역할을 넘어 훨씬 더 중추적인 임무를 수행하고 있다.

- **마이크로서비스 API 게이트웨이**: 분산된 여러 마이크로서비스들의 단일 진입점(Single Point of Entry) 역할을 하는 API 게이트웨이로서 Nginx가 널리 채택되고 있다. Nginx는 라우팅, 인증, 로깅, 속도 제한(rate limiting) 등 공통적인 횡단 관심사(cross-cutting concerns)를 처리하며, 백엔드 서비스의 복잡성을 클라이언트로부터 숨기는 중요한 역할을 한다.13
- **컨테이너 환경 (Kubernetes)**: 쿠버네티스(Kubernetes) 생태계에서 Nginx는 외부 트래픽을 클러스터 내부의 서비스로 라우팅하는 **인그레스 컨트롤러(Ingress Controller)**의 사실상 표준 구현체로 사용된다. 이는 Nginx가 컨테이너화된 동적 환경에서 트래픽 관리의 핵심 요소임을 증명한다.18

이러한 변화는 Nginx의 역할이 '웹 서버'에서 **'애플리케이션 딜리버리 컨트롤러(Application Delivery Controller, ADC)'**로 진화했음을 시사한다. 과거 웹의 중심이 정적인 페이지를 '제공'하는 것이었다면, 현대 웹의 중심은 복잡하게 분산된 애플리케이션 컴포넌트 간의 트래픽을 '연결'하고 '제어'하는 것으로 이동했다. Nginx는 바로 이 지점에서 핵심적인 가치를 제공한다.

Nginx 생태계 또한 F5에 인수된 이후 NGINX Plus와 같은 상용 제품을 통해 엔터프라이즈급 기능과 지원을 강화하고 있으며, 다중 언어 애플리케이션 서버인 NGINX Unit, 서비스 메시 솔루션인 NGINX Service Mesh 등으로 그 영역을 확장하고 있다.7 동시에, 최근 F5의 정책에 대한 비판으로 freenginx와 같은 새로운 포크(fork) 프로젝트가 등장하는 등, Nginx를 둘러싼 오픈소스 커뮤니티의 역동성 또한 여전히 살아있다.7

결론적으로, Nginx는 지난 20년간 웹 기술의 발전을 이끌어온 핵심 동력이었으며, 앞으로도 클라우드 네이티브와 마이크로서비스 시대의 복잡한 애플리케이션 환경에서 트래픽을 제어하고 전달하는 중추적인 인프라로서 그 중요성을 더욱 키워나갈 것이다.


1. NGINX: Everything you need to know about this open source web server - DataScientest, 8월 3, 2025에 액세스, https://datascientest.com/en/nginx-everything-you-need-to-know-about-this-open-source-web-server
2. medium.com, 8월 3, 2025에 액세스, [https://medium.com/@caophuc799/nginx-architecture-and-why-can-nginx-handle-thousands-of-requests-concurrently-b8686b558f86#:~:text=The%20C10k%20problem%20is%20the,client%20at%20the%20same%20time.](https://medium.com/@caophuc799/nginx-architecture-and-why-can-nginx-handle-thousands-of-requests-concurrently-b8686b558f86#:~:text=The C10k problem is the,client at the same time.)
3. Nginx History - velog, 8월 3, 2025에 액세스, https://velog.io/@dev_leewoooo/Nginx-History
4. NGINX vs Apache: Picking Best Web Server for Your Business, 8월 3, 2025에 액세스, https://www.liquidweb.com/blog/nginx-vs-apache/
5. Nginx는 무엇이고 왜 사용할까? - 애송이 개발 블로그, 8월 3, 2025에 액세스, https://parkmuhyeun.github.io/project/2023-01-07-nginx/
6. The Architecture of Open Source Applications (Volume 2)nginx, 8월 3, 2025에 액세스, https://aosabook.org/en/v2/nginx.html
7. Nginx - Wikipedia, 8월 3, 2025에 액세스, https://en.wikipedia.org/wiki/Nginx
8. NGINX (r228 판) - 나무위키, 8월 3, 2025에 액세스, https://namu.wiki/w/NGINX?uuid=feab8b74-4afb-474b-be17-6cd17b0e2839
9. A detailed comparison between Apache and Nginx web server - Site24x7, 8월 3, 2025에 액세스, https://www.site24x7.com/learn/web-server/apache-vs-nginx-web-server.html
10. wonit.tistory.com, 8월 3, 2025에 액세스, https://wonit.tistory.com/333
11. NGINX vs Apache – Choosing the Best Web Server in 2025 - Hostinger, 8월 3, 2025에 액세스, https://www.hostinger.com/tutorials/nginx-vs-apache
12. Apache와 Nginx 비교 및 추천 - 어떤 웹 서버가 더 나을까?, 8월 3, 2025에 액세스, https://www.devhypnos.com/posts/74
13. Apache vs NGINX: 완벽 비교, 8월 3, 2025에 액세스, [https://nginxstore.com/blog/nginx/apache-vs-nginx-%EC%99%84%EB%B2%BD-%EB%B9%84%EA%B5%90/](https://nginxstore.com/blog/nginx/apache-vs-nginx-완벽-비교/)
14. NGINX Architecture, 8월 3, 2025에 액세스, https://blog.nginx.org/nginx-architecture
15. Handling concurrent connections at scale with NGINX event driven architecture - Gurzu Inc, 8월 3, 2025에 액세스, https://gurzu.com/blog/handling-concurrent-connections-at-scale-with-nginx-event-driven-architecture/
16. nginx 기본 정리 (기능, 설치, 구조, 실행) - Develop Me, 8월 3, 2025에 액세스, [https://jinaon.tistory.com/entry/nginx-%EA%B8%B0%EB%B3%B8-%EC%A0%95%EB%A6%AC-%EA%B8%B0%EB%8A%A5-%EC%84%A4%EC%B9%98-%EA%B5%AC%EC%A1%B0-%EC%8B%A4%ED%96%89](https://jinaon.tistory.com/entry/nginx-기본-정리-기능-설치-구조-실행)
17. Nginx 기본정리, 8월 3, 2025에 액세스, [https://velog.io/@rlaghwns1995/Nginx-%EA%B8%B0%EB%B3%B8%EC%A0%95%EB%A6%AC](https://velog.io/@rlaghwns1995/Nginx-기본정리)
18. Understanding NGINX: Architecture, Configuration & Alternatives | Solo.io, 8월 3, 2025에 액세스, https://www.solo.io/topics/nginx
19. Understanding Nginx Worker Architecture - Mohit Mishra, 8월 3, 2025에 액세스, https://mohitmishra786.github.io/chessman/2024/12/29/Understanding-NGINX-Worker-Architecture.html
20. NGINX vs. Apache: Web Server Performance & Features - is*hosting Blog, 8월 3, 2025에 액세스, https://blog.ishosting.com/en/nginx-vs-apache
21. nginx, 8월 3, 2025에 액세스, https://nginx.org/en/
22. How to Install Nginx on Ubuntu 22.04 - phoenixNAP, 8월 3, 2025에 액세스, https://phoenixnap.com/kb/install-nginx-ubuntu-22-04
23. How to install nginx - Ubuntu Server documentation, 8월 3, 2025에 액세스, https://documentation.ubuntu.com/server/how-to/web-services/install-nginx/
24. How to Install Nginx Web Server on Ubuntu 24.04 - phoenixNAP, 8월 3, 2025에 액세스, https://phoenixnap.com/kb/install-nginx-on-ubuntu
25. How to Install and Configure Nginx on CentOS and Rocky Linux - phoenixNAP, 8월 3, 2025에 액세스, https://phoenixnap.com/kb/how-to-install-nginx-on-centos-7
26. How To Install Nginx on CentOS 7 - DigitalOcean, 8월 3, 2025에 액세스, https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-centos-7
27. How to Install Nginx on CentOS 8 - DigitalOcean, 8월 3, 2025에 액세스, https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-centos-8
28. Installing Nginx in CentOS 7 - GitHub Gist, 8월 3, 2025에 액세스, https://gist.github.com/b44af8e84d1e5d63b990
29. NGINX 기초 - 알고풀자 - 티스토리, 8월 3, 2025에 액세스, https://algopoolja.tistory.com/108
30. Beginner's Guide - nginx, 8월 3, 2025에 액세스, http://nginx.org/en/docs/beginners_guide.html
31. nginx를 이용한 로드밸런싱 및 무중단 배포 - 너도나도 함께 성장하기 - 티스토리, 8월 3, 2025에 액세스, https://escapefromcoding.tistory.com/597
32. Understanding the Nginx Configuration File Structure and Configuration Contexts, 8월 3, 2025에 액세스, https://www.digitalocean.com/community/tutorials/understanding-the-nginx-configuration-file-structure-and-configuration-contexts
33. NGINX - What is the difference between a 'block directive' and a 'context'? - Stack Overflow, 8월 3, 2025에 액세스, https://stackoverflow.com/questions/47423348/nginx-what-is-the-difference-between-a-block-directive-and-a-context
34. NGINX App Protect DoS Directives and Policy, 8월 3, 2025에 액세스, https://docs.nginx.com/nginx-app-protect-dos/directives-and-policy/learn-about-directives-and-policy/
35. K000151885: NGINX proxy_set_header directives configured in http or server contexts have no effect - MyF5 | Support, 8월 3, 2025에 액세스, https://my.f5.com/manage/s/article/K000151885
36. NGINX Configuration file - velog, 8월 3, 2025에 액세스, https://velog.io/@dogfootbirdfoot/nginx.conf
37. [Nginx] Nginx의 핵심 파일 nginx.conf 파일은 어떻게 구성될까? 분석해보자! - Wonit, 8월 3, 2025에 액세스, https://wonit.tistory.com/335
38. Core functionality - nginx, 8월 3, 2025에 액세스, http://nginx.org/en/docs/ngx_core_module.html
39. Nginx architecture. Nginx has modular and event driven... | by natnat - Medium, 8월 3, 2025에 액세스, https://natnat1.medium.com/nginx-architecture-c9c93058e8a0
40. Example nginx configuration, 8월 3, 2025에 액세스, http://nginx.org/en/docs/example.html
41. Module ngx_http_core_module - nginx, 8월 3, 2025에 액세스, http://nginx.org/en/docs/http/ngx_http_core_module.html
42. How to configure nginx - Ubuntu Server documentation, 8월 3, 2025에 액세스, https://documentation.ubuntu.com/server/how-to/web-services/configure-nginx/
43. Configuring NGINX and NGINX Plus as a Web Server, 8월 3, 2025에 액세스, https://docs.nginx.com/nginx/admin-guide/web-server/web-server/
44. Understanding Nginx Server and Location Block Selection Algorithms - DigitalOcean, 8월 3, 2025에 액세스, https://www.digitalocean.com/community/tutorials/understanding-nginx-server-and-location-block-selection-algorithms
45. Nginx location directive examples - DigitalOcean, 8월 3, 2025에 액세스, https://www.digitalocean.com/community/tutorials/nginx-location-directive
46. Nginx Location Directive Explained - KeyCDN Support, 8월 3, 2025에 액세스, https://www.keycdn.com/support/nginx-location-directive
47. NGINX Configuration: Directives, Examples, and 4 Mistakes to Avoid | Solo.io, 8월 3, 2025에 액세스, https://www.solo.io/topics/nginx/nginx-configuration
48. Serving Static Content with NGINX - DEV Community, 8월 3, 2025에 액세스, https://dev.to/imsushant12/serving-static-content-with-nginx-26ih
49. Serve Static Content | NGINX Documentation, 8월 3, 2025에 액세스, https://docs.nginx.com/nginx/admin-guide/web-server/serving-static-content/
50. Understanding the difference between the root and alias directives in Nginx - Techcoil, 8월 3, 2025에 액세스, https://www.techcoil.com/blog/understanding-the-difference-between-the-root-and-alias-directives-in-nginx/
51. Nginx -- static file serving confusion with root & alias - Stack Overflow, 8월 3, 2025에 액세스, https://stackoverflow.com/questions/10631933/nginx-static-file-serving-confusion-with-root-alias
52. Nginx -- Static File Serving Confusion With Root & Alias | Better Stack Community, 8월 3, 2025에 액세스, https://betterstack.com/community/questions/static-file-serving-confusion-with-root-alias/
53. [서버] Nginx란? (개념, 특징, 장단점, 면접 대비) - 니나노 - 티스토리, 8월 3, 2025에 액세스, [https://yuna-ninano.tistory.com/entry/%EC%84%9C%EB%B2%84-Nginx%EB%9E%80-%EA%B0%9C%EB%85%90-%ED%8A%B9%EC%A7%95-%EC%9E%A5%EB%8B%A8%EC%A0%90-%EB%A9%B4%EC%A0%91-%EB%8C%80%EB%B9%84](https://yuna-ninano.tistory.com/entry/서버-Nginx란-개념-특징-장단점-면접-대비)
54. How To Set Up a Reverse Proxy (for Nginx & Apache) - Kinsta, 8월 3, 2025에 액세스, https://kinsta.com/blog/reverse-proxy/
55. How To Configure Nginx as a Reverse Proxy on Ubuntu 22.04 - DigitalOcean, 8월 3, 2025에 액세스, https://www.digitalocean.com/community/tutorials/how-to-configure-nginx-as-a-reverse-proxy-on-ubuntu-22-04
56. NGINX Reverse Proxy | NGINX Documentation, 8월 3, 2025에 액세스, https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/
57. Nginx reverse proxy + URL rewrite - Server Fault, 8월 3, 2025에 액세스, https://serverfault.com/questions/379675/nginx-reverse-proxy-url-rewrite
58. Nginx: Everything about proxy_pass - DEV Community, 8월 3, 2025에 액세스, https://dev.to/danielkun/nginx-everything-about-proxypass-2ona
59. Module ngx_http_proxy_module - nginx, 8월 3, 2025에 액세스, http://nginx.org/en/docs/http/ngx_http_proxy_module.html
60. Module ngx_http_realip_module - nginx, 8월 3, 2025에 액세스, http://nginx.org/en/docs/http/ngx_http_realip_module.html
61. Forwarding real remote IP to proxied server with nginx, 8월 3, 2025에 액세스, https://serverfault.com/questions/837915/forwarding-real-remote-ip-to-proxied-server-with-nginx
62. NGINX and X-Forwarded-For Header (XFF) - Loadbalancer.org, 8월 3, 2025에 액세스, https://www.loadbalancer.org/blog/nginx-and-x-forwarded-for-header/
63. How to Use Nginx as a Load Balancer for Your Application | by elaurichenickson | Medium, 8월 3, 2025에 액세스, https://medium.com/@elaurichetoho/how-to-use-nginx-as-a-load-balancer-for-your-application-d80ca40f28d8
64. 4 Setting Up Load Balancing by Using NGINX - Oracle Help Center, 8월 3, 2025에 액세스, https://docs.oracle.com/en/operating-systems/oracle-linux/8/balancing/balancing-SettingUpLoadBalancingbyUsingNGINX.html
65. Using nginx as HTTP load balancer, 8월 3, 2025에 액세스, http://nginx.org/en/docs/http/load_balancing.html
66. Nginx를 로드밸런서로 사용하기, 8월 3, 2025에 액세스, https://daydayplus.tistory.com/103
67. A Deep Dive into Nginx Load Balancing Algorithms | by Raj Ranjan Sinha - Medium, 8월 3, 2025에 액세스, https://medium.com/@raj.ranjan.sinha/a-deep-dive-into-nginx-load-balancing-algorithms-0b27f5b1a3f3
68. [Network] 로드밸런싱 알고리즘 종류 - 우노 - 티스토리, 8월 3, 2025에 액세스, https://wooono.tistory.com/714
69. High-Performance Load Balancing with NGINX | by Shamoda Jayasekara | Tech It Out, 8월 3, 2025에 액세스, https://medium.com/tech-it-out/high-performance-load-balancing-with-nginx-b3a31acd88a3
70. [Network] 로드 밸런서(LB) - 정의, 역할, 로드밸런싱 알고리즘, 종류 - 無테고리 인생살이, 8월 3, 2025에 액세스, https://chunsubyeong.tistory.com/106
71. IP Hash Load Balancing: NGINX Configuration Guide - Coherence, 8월 3, 2025에 액세스, https://www.withcoherence.com/articles/ip-hash-load-balancing-nginx-configuration-guide
72. Nginx를 사용한 로드밸런싱 환경 구축 - hudi.blog, 8월 3, 2025에 액세스, https://hudi.blog/load-balancing-with-nginx/
73. NGINX - Upstream Module (Part 01) | by Nethmini Romina | FAUN.dev, 8월 3, 2025에 액세스, https://faun.pub/nginx-upstream-module-part-01-e7433abcbf90
74. HTTP Load Balancing | NGINX Documentation, 8월 3, 2025에 액세스, https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/
75. Nginx 소개, 8월 3, 2025에 액세스, [https://velog.io/@harperkwon/Nginx-%EC%86%8C%EA%B0%9C](https://velog.io/@harperkwon/Nginx-소개)
76. High-Performance Caching with NGINX & NGINX Plus - F5 Networks, 8월 3, 2025에 액세스, https://www.f5.com/content/dam/f5/corp/global/pdf/ebooks/High-Performance-Caching-NGINX-Plus.pdf
77. NGINX 기본 캐시 설정 - HYEL & TSAR - 티스토리, 8월 3, 2025에 액세스, https://5hyel.tistory.com/40
78. NGINX Content Caching | NGINX Documentation, 8월 3, 2025에 액세스, https://docs.nginx.com/nginx/admin-guide/content-cache/content-caching/
79. [Nginx] Cache 적용 - 돼민이 - 티스토리, 8월 3, 2025에 액세스, https://jink1982.tistory.com/216
80. Nginx cache example config - GitHub Gist, 8월 3, 2025에 액세스, https://gist.github.com/Friz-zy/3dd2a670d8f14eac717e100d0dc7696f
81. Middleware (Cache-1) Nginx reverse proxy 로 Cache 기능활용 - TECH | 클럭스, 8월 3, 2025에 액세스, http://www.chlux.co.kr/bbs/board.php?bo_table=board02&wr_id=135&sca=Middleware&page=1
82. NGINX Config Rewind: Serverion Revives the Lost Art of Proxy Cache Tuning, 8월 3, 2025에 액세스, https://www.serverion.com/uncategorized/nginx-config-rewind-serverion-revives-the-lost-art-of-proxy-cache-tuning/
83. Configuring HTTPS servers - nginx, 8월 3, 2025에 액세스, http://nginx.org/en/docs/http/configuring_https_servers.html
84. NGINX SSL Termination | NGINX Documentation, 8월 3, 2025에 액세스, https://docs.nginx.com/nginx/admin-guide/security-controls/terminating-ssl-http/
85. Module ngx_http_ssl_module - nginx, 8월 3, 2025에 액세스, http://nginx.org/en/docs/http/ngx_http_ssl_module.html
86. How to create a CSR using OpenSSL & install your SSL certificate on a Nginx server, 8월 3, 2025에 액세스, https://knowledge.digicert.com/tutorials/how-to-create-a-csr-using-openssl-and-install-your-ssl-certificate-on-a-nginx-server
87. Nginx에 Let's Encrypt SSL(https) 보안 인증서 적용하기(with certbot 인증서 자동갱신), 8월 3, 2025에 액세스, https://deoking.tistory.com/7
88. Install Let's Encrypt SSL on Ubuntu with Certbot | InMotion Hosting, 8월 3, 2025에 액세스, https://www.inmotionhosting.com/support/website/ssl/lets-encrypt-ssl-ubuntu-with-certbot/
89. How To Secure Nginx with Let's Encrypt on Ubuntu 20.04 - DigitalOcean, 8월 3, 2025에 액세스, https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-20-04
90. [Ubuntu] Nginx, Let's Encrypt를 사용하여 HTTPS 설정하기 (+에러 해결), 8월 3, 2025에 액세스, https://maemae22.tistory.com/109
91. How To Install Let's Encrypt On Nginx - UpCloud, 8월 3, 2025에 액세스, https://upcloud.com/resources/tutorials/install-lets-encrypt-nginx/
92. Let's Encrypt 인증서로 NGINX SSL 설정하기, 8월 3, 2025에 액세스, [https://nginxstore.com/blog/nginx/lets-encrypt-%EC%9D%B8%EC%A6%9D%EC%84%9C%EB%A1%9C-nginx-ssl-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0/](https://nginxstore.com/blog/nginx/lets-encrypt-인증서로-nginx-ssl-설정하기/)
93. Configure Gzip Compression on Apache and Nginx - Vultr Docs, 8월 3, 2025에 액세스, https://docs.vultr.com/configure-gzip-compression-on-apache-and-nginx
94. nginx gzip 옵션 - 나만의 인덱스 - 티스토리, 8월 3, 2025에 액세스, https://yangbongsoo.tistory.com/4
95. [Nginx] Nginx에 간단하게 gzip 압축 설정하기, 8월 3, 2025에 액세스, https://twpower.github.io/48-set-up-gzip-in-nginx
96. How to enable GZIP on Nginx - SpeedyCache, 8월 3, 2025에 액세스, https://speedycache.com/docs/miscellaneous/how-to-enable-gzip-on-nginx
97. Module ngx_http_gzip_module - nginx, 8월 3, 2025에 액세스, http://nginx.org/en/docs/http/ngx_http_gzip_module.html
98. Gzip configuration for Nginx - GitHub Gist, 8월 3, 2025에 액세스, https://gist.github.com/sydcanem/3e00c09b3361927b2fd1
99. nginx gzip - velog, 8월 3, 2025에 액세스, https://velog.io/@dudtjr913/nginx-gzip
100. Compression on Nginx - Junhyunny's Devlogs, 8월 3, 2025에 액세스, https://junhyunny.github.io/information/nginx/compression-on-nginx/
101. How To Improve Website Performance Using gzip and Nginx on Ubuntu 20.04, 8월 3, 2025에 액세스, https://www.digitalocean.com/community/tutorials/how-to-improve-website-performance-using-gzip-and-nginx-on-ubuntu-20-04