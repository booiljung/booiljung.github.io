# OverlayFS 파일 시스템


OverlayFS는 리눅스를 위한 현대적 유니온 마운트(union mount) 파일 시스템으로, 여러 파일 시스템이나 디렉터리 트리를 계층적으로 중첩하여 사용자에게 단일 통합 뷰(unified view)를 제공하는 강력한 기술이다.1 이 파일 시스템의 핵심 철학은 하나 이상의 읽기 전용(read-only) 하위 레이어(lower layer) 위에 읽기-쓰기(read-write)가 가능한 상위 레이어(upper layer)를 겹쳐 놓는 것이다.3 이 구조를 통해 발생하는 모든 변경 사항은 상위 레이어에만 기록되며, 하위 레이어는 원본 상태를 그대로 유지하게 된다.1 이러한 특성 덕분에 OverlayFS는 시스템의 불변성(immutability)을 보장하면서도 유연한 변경을 허용하는 독특한 능력을 갖추게 된다.

본 보고서는 OverlayFS의 내부 아키텍처, 핵심 동작 원리인 Copy-on-Write(CoW), 실제 시스템에서의 활용 방안, 그리고 성능에 영향을 미치는 주요 요인들을 심층적으로 분석한다. 더 나아가, POSIX 표준과의 비호환성 문제, 주요 보안 취약점 사례, 그리고 Aufs와 같은 다른 유니온 파일 시스템과의 비교를 통해 OverlayFS의 기술적 한계와 장점을 명확히 규명한다. 최종적으로, 시스템 아키텍트, 데브옵스 엔지니어, 그리고 보안 전문가가 이 기술을 시스템 설계, 성능 최적화, 보안 감사 등의 실무에 효과적으로 적용할 수 있도록 종합적이고 심도 있는 통찰을 제공하는 것을 목표로 한다.

OverlayFS는 단순한 파일 시스템 기술의 범주를 넘어선다. 이는 컨테이너화(containerization) 기술의 대명사인 도커(Docker)의 핵심 스토리지 드라이버로 채택되면서 현대 클라우드 네이티브 컴퓨팅 환경의 근간을 이루게 되었다.5 또한, Live CD나 IoT 기기와 같이 쓰기 횟수가 제한된 플래시 메모리를 사용하는 시스템에서도 원본 미디어를 보호하는 효율적인 솔루션을 제공한다.2 나아가, 서버 구성을 코드처럼 관리하고 배포 시마다 새로운 인스턴스로 교체하는 불변 인프라(Immutable Infrastructure) 패러다임을 파일 시스템 수준에서 완벽하게 지원하는 핵심 구현체로서 그 중요성이 날로 증대되고 있다.7 따라서 OverlayFS에 대한 깊이 있는 이해는 현대 IT 인프라를 구축하고 운영하는 모든 기술 전문가에게 필수적인 역량이라 할 수 있다.



유니온 파일 시스템은 여러 개의 독립된 디렉터리, 즉 브랜치(branch)의 내용을 물리적으로는 분리된 상태를 유지하면서, 논리적으로는 하나의 통합된 디렉터리처럼 보이도록 병합하는 파일 시스템 기술이다.2 사용자는 이 통합된 뷰를 통해 여러 소스에 흩어져 있는 파일과 디렉터리를 단일 경로에서 접근하고 조작할 수 있다.10 이러한 개념은 시스템의 유연성과 효율성을 극대화하는 데 중요한 역할을 한다.

OverlayFS의 등장은 리눅스 유니온 파일 시스템의 역사에서 중요한 전환점을 마련했다. 2010년, Miklos Szeredi에 의해 초기 RFC(Request for Comments) 패치셋이 리눅스 커널 커뮤니티에 제출되었으며, 수년간의 논의와 개선을 거쳐 2014년 리눅스 커널 버전 3.18에 공식적으로 병합되었다.2 이는 Aufs(Another Union File System)와 같이 커널 외부에서 별도의 패치를 통해 유지되던 기존 유니온 파일 시스템들과 근본적인 차이를 보이는 지점이다. 커널 메인라인에 포함됨으로써 OverlayFS는 광범위한 리눅스 배포판에서 표준으로 지원받게 되었고, 이는 안정성, 보안, 그리고 유지보수성 측면에서 큰 이점을 가져왔다. 이후 커널 4.0에서는 도커의 `overlay2` 스토리지 드라이버가 요구하는 기능 개선이 이루어지는 등 지속적인 발전을 거듭하며 사실상의 표준 유니온 파일 시스템으로 자리매김하게 되었다.2


OverlayFS의 동작을 이해하기 위해서는 네 가지 핵심 구성 요소의 역할과 상호작용을 명확히 파악해야 한다. 이 구성 요소들은 계층적 파일 시스템이라는 OverlayFS의 본질을 구현하는 기본 단위이다.

- `lowerdir` (하위 디렉터리): `lowerdir`는 변경되지 않는 원본 데이터를 담고 있는 하나 이상의 읽기 전용(Read-Only) 베이스 레이어(base layer)를 의미한다.5 컨테이너 이미지의 기반이 되는 OS 파일이나 애플리케이션 라이브러리 등이 여기에 해당한다. 마운트 시 여러 개의 `lowerdir`를 지정할 수 있으며, 이를 통해 다층적인 파일 시스템 구조를 형성할 수 있다.3 `lowerdir`는 마운트된 후에는 어떠한 쓰기 작업도 허용하지 않음으로써 원본 데이터의 무결성을 보장하는 역할을 한다.5

- `upperdir` (상위 디렉터리): `upperdir`는 파일 시스템 계층의 가장 위에 위치하는 읽기-쓰기(Read-Write)가 가능한 레이어다.3 사용자가 `merged` 디렉터리를 통해 수행하는 모든 변경 작업, 즉 파일 생성, 기존 파일 수정, 파일 삭제 등의 결과가 이 `upperdir`에 기록된다.5 `lowerdir`의 불변성을 유지하면서 동적인 변경을 수용하는 창구 역할을 수행한다.

- `merged` (병합 디렉터리): `merged` 디렉터리는 `lowerdir`와 `upperdir`의 내용이 논리적으로 통합되어 사용자에게 최종적으로 보여지는 가상의 마운트 지점이다.5 사용자는 이 디렉터리를 일반적인 파일 시스템처럼 사용하며, 내부적으로는 OverlayFS가 각 파일 연산을 적절한 레이어로 전달한다. 만약  `upperdir`와 `lowerdir`에 동일한 이름의 파일이 존재할 경우, 우선순위가 높은 `upperdir`의 파일이 `merged` 뷰에 나타나게 된다.5

- `workdir` (작업 디렉터리): `workdir`는 파일 시스템 연산의 원자성(atomicity)을 보장하기 위해 내부적으로 사용되는 임시 작업 공간이다.5 이 디렉터리는 반드시  `upperdir`와 동일한 파일 시스템 상에 위치해야 하는 제약 조건을 가진다.3

  `workdir`의 존재는 OverlayFS의 안정성과 데이터 무결성을 위한 핵심적인 설계 결정이다. `copy_up`과 같이 여러 단계로 이루어진 복잡한 파일 연산(예: 파일 생성, 메타데이터 복사, 데이터 복사)이 중간에 시스템 장애로 중단될 경우, `upperdir`에는 불완전하거나 손상된 파일이 남을 수 있다. `workdir`는 이러한 연산이 완전히 성공하기 전까지의 중간 상태를 처리하는 '준비 영역(staging area)' 역할을 수행한다. 모든 단계가 성공적으로 완료되면, `workdir`의 결과물은 `rename` 시스템 콜을 통해 원자적으로 `upperdir`로 이동된다. 만약 연산이 실패하면 `workdir`의 내용은 단순히 폐기되므로, `upperdir`의 일관성은 깨지지 않는다. 따라서 `workdir`는 단순한 임시 폴더가 아니라, 파일 시스템의 트랜잭션적 특성을 보장하여 신뢰성을 높이는 필수적인 안전장치로 기능한다.5


OverlayFS의 효율성과 불변성의 핵심에는 Copy-on-Write(CoW)라는 메커니즘이 자리 잡고 있다. 이는 데이터에 대한 수정 요청이 발생했을 때, 원본을 직접 수정하는 대신 복사본을 만들어 그 복사본을 수정하는 방식을 의미한다.


- **읽기(Read):** 파일 읽기 작업 시에는 CoW가 발생하지 않는다.11 시스템은 먼저 변경 사항이 저장되는 `upperdir`에서 요청된 파일을 찾는다. 만약 파일이 `upperdir`에 존재하지 않으면, `lowerdir` 계층을 우선순위에 따라 순차적으로 탐색하여 파일을 찾아 읽는다. 이 과정은 하위 파일 시스템으로 I/O 요청을 직접 전달하는 방식에 가깝기 때문에, 네이티브 파일 시스템과 거의 유사한 성능을 보인다.3

- **쓰기/수정(Write/Modify):** 진정한 CoW 메커니즘은 쓰기 또는 수정 작업 시에 발현된다. 만약 `lowerdir`에만 존재하는 파일을 처음으로 수정하려고 시도하면, 'copy-up'이라는 연산이 트리거된다.12 이 과정에서 해당 파일 전체가 

  `lowerdir`에서 `upperdir`로 복사된다. 복사가 완료된 후, 사용자의 수정 작업은 `upperdir`에 생성된 이 복사본에 적용된다.11 이후 해당 파일에 대한 모든 추가적인 수정 작업은 더 이상 

  `copy-up`을 유발하지 않고 `upperdir`의 파일에 직접 이루어진다. 이 메커니즘 덕분에 원본 `lowerdir`는 어떠한 변경도 겪지 않고 불변성을 유지할 수 있다.11


OverlayFS는 `lowerdir`의 불변성을 유지해야 하므로, 파일을 물리적으로 삭제하는 대신 논리적으로 보이지 않게 만드는 특별한 방법을 사용한다.

- `Whiteout` 파일: 사용자가 `merged` 뷰에서 `lowerdir`에 존재하는 파일을 삭제하면, OverlayFS는 `upperdir`에 해당 파일과 동일한 이름의 특수한 파일을 생성한다. 이 파일은 캐릭터 디바이스(character device) 형태이며, 주/부 번호(major/minor number)가 각각 0/0으로 설정되어 있다.14 이 특수 파일을 'whiteout' 파일이라고 부른다.5

  `merged` 뷰를 생성할 때, OverlayFS는 `upperdir`에서 whiteout 파일을 발견하면 `lowerdir`에 있는 동일한 이름의 원본 파일을 무시하고 사용자에게 보여주지 않는다. 즉, whiteout 파일은 하위 레이어의 파일을 가리는(masking) 역할을 하여 논리적인 삭제를 구현한다.11 실제 원본 파일은 

  `lowerdir`에 그대로 남아있다.

- `Opaque Directory` (불투명 디렉터리): 파일 단위의 마스킹 외에도, 디렉터리 단위로 병합을 차단하는 기능이 존재한다. `upperdir`의 특정 디렉터리에 `trusted.overlay.opaque`라는 확장 속성(extended attribute, xattr)을 'y' 값으로 설정하면, 해당 디렉터리는 '불투명(opaque)' 상태가 된다.14 이 경우, `merged` 뷰에서 해당 디렉터리를 조회할 때, `lowerdir`에 동일한 이름의 디렉터리가 존재하더라도 그 내용은 완전히 무시되고 `upperdir`의 내용만 보이게 된다.2 이는 특정 디렉터리의 내용을 하위 레이어와 무관하게 완전히 새로운 것으로 대체하고자 할 때 유용하게 사용된다.



OverlayFS를 시스템에 적용하기 위해서는 `mount` 명령어를 통해 파일 시스템을 마운트해야 한다. 이때 다양한 옵션을 조합하여 특정 요구사항에 맞게 동작을 정밀하게 제어할 수 있다. 기본적인 마운트 명령어의 형식은 다음과 같다 3:

```sh
$ mount -t overlay overlay -o [options] /merged
```

여기서 `-t overlay`는 파일 시스템 유형을 지정하고, `overlay`는 소스 디바이스가 없음을 의미하는 더미 이름이다. 핵심적인 설정은 `-o` 플래그 뒤에 오는 옵션들을 통해 이루어진다. 각 옵션의 기능과 그로 인한 시스템 동작의 변화를 명확히 이해하는 것은 성능 최적화와 안정성 확보에 필수적이다. 주요 마운트 옵션과 그 역할은 다음 표와 같다.

**Table 2.1: 주요 OverlayFS 마운트 옵션**

| 옵션 (Option)                  | 설명                                                         | 주요 사용 사례 및 영향                                       |
| ------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `lowerdir=[dir1]:[dir2]...`    | 하나 이상의 읽기 전용 하위 레이어를 지정한다. 복수 지정 시 `:`로 구분하며, 오른쪽 디렉터리가 가장 낮은 계층이 된다.3 | 도커 이미지 레이어 구성, Live CD 베이스 시스템 등 계층적 구조가 필요한 경우에 사용된다. 레이어 순서가 병합 결과에 직접적인 영향을 미친다. |
| `upperdir=[dir]`               | 읽기-쓰기가 가능한 상위 레이어를 지정한다. `merged` 디렉터리에서 발생하는 모든 변경 사항이 이 디렉터리에 기록된다.3 | 컨테이너의 쓰기 가능 공간, 사용자 세션 동안의 변경사항 저장 등 동적인 데이터 처리에 필수적이다. |
| `workdir=[dir]`                | 원자적 파일 연산을 보장하기 위한 내부 작업 디렉터리. 반드시 `upperdir`와 동일한 파일 시스템에 위치한 빈 디렉터리여야 한다.3 | 파일 시스템의 무결성을 보장하는 핵심 요소로, 쓰기 가능한 OverlayFS 마운트 시 필수적으로 지정해야 한다. |
| `xino=[on|auto|off]`           | 여러 다른 하위 파일 시스템에 걸쳐 inode 번호의 고유성과 지속성을 보장하는 기능. `st_ino`와 `fsid`를 조합하여 고유 ID를 생성한다.15 | NFS 환경과의 호환성을 높이고, 일부 파일 시스템 스캐너나 백업 도구가 inode 번호 변경으로 인해 오작동하는 것을 방지한다. |
| `volatile`                     | `upperdir`에 대한 모든 `sync` 시스템 콜을 생략하여 쓰기 성능을 가속화한다. 데이터 손실의 위험이 있다.17 | 데이터 영속성이 중요하지 않은 임시 빌드 컨테이너나 테스트 환경에서 I/O 성능을 극대화하기 위해 사용된다. |
| `redirect_dir=[on|off|follow]` | `lowerdir`에 있는 디렉터리에 대한 `rename(2)` 시스템 콜의 동작 방식을 제어한다. `on`으로 설정 시 이름 변경을 지원한다.15 | POSIX 표준과의 호환성을 개선하여, `rename` 시 발생하는 `EXDEV` 오류를 방지하고 애플리케이션의 정상적인 동작을 돕는다. |
| `index=[on|off]`               | 하드 링크가 있는 파일을 `copy_up`할 때, 링크 관계를 `upperdir`에서도 유지하도록 한다. 비활성화 시 링크가 깨진다.16 | 여러 이름으로 참조되는 파일의 일관성을 보장해야 하는 경우에 필수적이다. |
| `metacopy=[on|off]`            | `chmod`, `chown` 등 메타데이터만 변경되는 경우, 파일 데이터 전체를 복사하지 않고 메타데이터만 `copy_up`하도록 한다.18 | 메타데이터 변경이 잦은 워크로드에서 불필요한 데이터 복사를 방지하여 `copy_up` 오버헤드를 크게 줄이고 성능을 향상시킨다. |


OverlayFS의 가장 강력한 기능 중 하나는 `lowerdir` 옵션에 복수의 디렉터리를 지정하여 계층적인 읽기 전용 레이어 스택(stack)을 구성할 수 있다는 점이다.3 디렉터리들은 콜론(`:`)으로 구분되며, 마운트 옵션 문자열에서 왼쪽에 위치한 디렉터리가 더 높은 우선순위를 갖는다. 예를 들어, `lowerdir=/app:/os`로 지정하면 `/app` 레이어가 `/os` 레이어 위에 겹쳐지게 된다.14 만약 두 레이어에 동일한 경로의 파일이 존재한다면, 더 높은 우선순위를 가진 `/app` 레이어의 파일이 `merged` 뷰에 나타난다.5

이러한 계층적 구조는 현대적인 소프트웨어 배포 모델에서 매우 유용하게 활용된다. 대표적인 시나리오는 다음과 같다 21:

1. **가장 낮은 레이어 (`lower2`):** 최소한의 운영 체제 파일들을 포함하는 베이스 OS 이미지를 배치한다. (예: `ubuntu:22.04` 이미지)
2. **중간 레이어 (`lower1`):** 베이스 OS 위에 특정 애플리케이션을 실행하는 데 필요한 런타임과 라이브러리를 포함하는 이미지를 배치한다. (예: `python:3.10` 이미지)
3. **최상위 레이어 (`upperdir`):** 사용자의 애플리케이션 코드나 설정 파일 변경사항, 실행 중 생성되는 로그 파일 등을 저장한다.

이 구조를 통해 OS, 런타임, 애플리케이션 코드를 논리적으로 분리하고 독립적으로 관리할 수 있다. 여러 애플리케이션이 동일한 OS와 런타임 레이어를 공유함으로써 스토리지 공간을 획기적으로 절약할 수 있으며, 업데이트 시에는 해당 레이어만 교체하면 되므로 배포 및 관리가 매우 효율적이다.23 이는 도커 이미지 레이어링이 동작하는 핵심 원리이기도 하다.


OverlayFS는 오늘날 컨테이너 생태계의 '속도'와 '밀도'를 결정하는 핵심 기반 기술이라고 해도 과언이 아니다. 만약 OverlayFS와 같은 유니온 파일 시스템이 없었다면, 컨테이너를 시작할 때마다 수 기가바이트에 달하는 전체 OS와 애플리케이션 파일을 매번 복사해야 했을 것이다. 이는 막대한 시간과 디스크 공간을 소모하여 컨테이너의 핵심 가치인 민첩성과 효율성을 심각하게 저해했을 것이다. OverlayFS의 CoW 및 레이어 공유 메커니즘은 이러한 비효율적인 복사 과정을 제거함으로써 현대적인 컨테이너 기술을 가능하게 했다. 컨테이너 시작은 단순히 새로운 `upperdir`를 생성하고 기존 `lowerdir` 스택 위에 마운트하는 과정으로 축소되어 거의 즉각적으로 이루어진다. 또한, 1GB의 베이스 이미지를 사용하는 100개의 컨테이너는 1GB(베이스 이미지)에 각 컨테이너의 미미한 변경분(`$`α`$`)을 더한 공간만을 차지하므로, 스토리지 비용을 획기적으로 절감하고 단일 호스트에 더 많은 컨테이너를 배포할 수 있게 한다. 이처럼 OverlayFS의 아키텍처는 컨테이너의 민첩성과 효율성이라는 비즈니스 가치를 기술적으로 직접 구현한 것이며, 이것이 없었다면 오늘날의 컨테이너 생태계는 존재하기 어려웠을 것이다.11


도커 이미지는 `Dockerfile`에 정의된 명령어들을 기반으로 생성되는 여러 개의 읽기 전용 레이어의 집합이다.24

`Dockerfile`의 `RUN`, `COPY`, `ADD`와 같은 각 명령어는 이전 레이어를 기반으로 파일 시스템에 변경을 가하고, 그 결과를 새로운 레이어로 저장한다.6 이 과정이 반복되면서 이미지의 계층 구조가 만들어진다.

`docker pull` 명령어를 통해 이미지를 다운로드하면, 이 레이어들은 도커 호스트의 `/var/lib/docker/overlay2/` 디렉터리 아래에 각각 독립적인 디렉터리로 저장된다.24 도커의 `overlay2` 스토리지 드라이버는 이 이미지 레이어 디렉터리들을 OverlayFS의 `lowerdir` 스택으로 직접 매핑한다. 즉, 도커 이미지의 각 레이어는 OverlayFS의 읽기 전용 `lowerdir` 하나에 해당한다.24


`docker run` 명령어를 통해 컨테이너를 실행할 때, 도커 엔진은 다음과 같은 과정을 통해 컨테이너의 파일 시스템을 구성한다:

1. 선택된 이미지에 속한 모든 레이어 디렉터리들을 식별하여 OverlayFS의 `lowerdir` 스택으로 지정한다.
2. 해당 컨테이너만을 위한 고유한 디렉터리들을 새로 생성하여, 하나는 `upperdir`로, 다른 하나는 `workdir`로 지정한다. 이 `upperdir`가 바로 컨테이너의 쓰기 가능한 레이어(container layer)가 된다.6
3. 이 세 가지 디렉터리(`lowerdir` 스택, `upperdir`, `workdir`)를 사용하여 OverlayFS를 마운트하고, 그 결과를 컨테이너의 루트 파일 시스템(`merged` 뷰)으로 제공한다.

이러한 방식을 통해, 수백 개의 컨테이너가 동일한 베이스 이미지(공유된 `lowerdir` 스택)를 메모리와 디스크에서 공유하면서도, 각 컨테이너 내부에서 발생하는 모든 파일 시스템 변경(파일 생성, 수정, 삭제)은 격리된 고유의 `upperdir`에만 기록된다. 이는 스토리지 효율성을 극대화하는 동시에 컨테이너 간의 완벽한 격리성을 보장하는 매우 효율적인 구조다.11


OverlayFS는 그 구조적 특성으로 인해 독특한 성능 프로파일을 보인다. 읽기 작업은 거의 네이티브에 가까운 성능을 보이지만, 쓰기 작업, 특히 첫 쓰기 시에는 상당한 오버헤드가 발생할 수 있다. 반면, 메모리 관리 측면에서는 페이지 캐시 공유라는 강력한 이점을 제공한다.


- **읽기 성능:** `merged` 뷰를 통해 `lowerdir`에만 존재하는 파일을 읽을 때, 파일이 열린 후의 모든 I/O 작업은 VFS(Virtual File System) 계층을 거쳐 하위 파일 시스템으로 직접 전달된다.3 이 과정에서 약간의 메타데이터 조회 오버헤드가 발생할 수는 있지만, 실제 데이터 전송 성능은 하위 파일 시스템의 네이티브 성능에 매우 가깝다.
- **쓰기 성능:** OverlayFS의 성능 병목 현상은 주로 첫 쓰기 작업 시 발생하는 `copy_up` 연산에서 비롯된다. `lowerdir`에 있는 파일을 처음으로 수정하려고 할 때, OverlayFS는 해당 파일 전체를 `upperdir`로 복사해야 한다.12 이 복사 작업은 파일의 크기에 비례하는 지연 시간(latency)을 유발하며, 특히 수 기가바이트에 달하는 대용량 파일의 경우 이 오버헤드는 매우 두드러질 수 있다.25 일단 `copy_up`이 완료된 후의 후속 쓰기 작업은 `upperdir`에서 직접 이루어지므로 빠르지만, 최초의 지연은 피할 수 없다.
- **메타데이터 연산:** `find.` 또는 `ls -lR`과 같이 디렉터리 트리를 순회하며 많은 파일의 메타데이터를 조회하는 작업은 OverlayFS에서 성능 저하를 보일 수 있다. 이는 `merged` 뷰를 구성하기 위해 `upperdir`와 모든 `lowerdir` 계층을 순차적으로 탐색하고 그 결과를 병합해야 하기 때문이다.12 각 파일에 대한 조회는 잠재적으로 여러 하위 디렉터리에 대한 조회를 유발하므로, 네이티브 파일 시스템에 비해 더 많은 I/O 및 CPU 자원을 소모하게 된다.


OverlayFS의 가장 중요한 성능 이점 중 하나는 페이지 캐시 공유 기능이다. 이는 특히 동일한 베이스 이미지를 기반으로 수많은 컨테이너가 동시에 실행되는 고밀도 환경에서 시스템 전체의 메모리 효율성을 극대화한다.25 예를 들어, 100개의 우분투 컨테이너가 동일한 호스트에서 실행될 때, 모든 컨테이너는 공통적으로 `/lib/x86_64-linux-gnu/libc.so.6`와 같은 핵심 공유 라이브러리를 사용한다. OverlayFS 환경에서는 이 `libc.so.6` 파일의 데이터가 물리적 메모리(RAM)의 페이지 캐시에 단 한 번만 로드된다.26 이후 100개의 모든 컨테이너 프로세스는 이 공유된 단일 페이지 캐시 엔트리를 참조하여 파일을 읽게 되므로, 메모리 사용량이 획기적으로 감소한다.28

이러한 효율성은 OverlayFS가 파일 시스템 수준에서 동작하기에 가능하다. 반면, Device Mapper와 같은 블록 기반 스토리지 드라이버는 각 컨테이너에 고유한 가상 블록 디바이스를 할당한다. 커널의 VFS 계층 관점에서 보면, 각 컨테이너의 `libc.so.6` 파일은 서로 다른 디바이스에 존재하는 별개의 inode를 가진 완전히 다른 파일로 인식된다. 따라서 커널은 이들을 동일한 파일로 간주하지 않고, 각 컨테이너를 위해 별도의 페이지 캐시를 생성하게 된다. 결과적으로 100개의 컨테이너는 100개의 `libc.so.6` 복사본을 메모리에 올리게 되어 심각한 메모리 낭비를 초래한다.26 이처럼 OverlayFS의 페이지 캐시 공유 능력은 '읽기 전용' `lowerdir` 설계에서 파생된 직접적인 결과이며, 메모리 효율성 측면에서 블록 기반 스토리지 드라이버에 비해 압도적인 우위를 점하게 하는 근본적인 차별점이다. 이는 고밀도 PaaS(Platform as a Service) 환경 구축에 OverlayFS가 더 적합한 이유를 설명해준다.


OverlayFS의 성능을 최적화하기 위해서는 백엔드 파일 시스템 선택, 커널 버전, 그리고 마운트 옵션이라는 세 가지 요소를 종합적으로 고려해야 한다.

- **백엔드 파일 시스템 선택:** Docker를 비롯한 다수의 컨테이너 플랫폼은 `overlay2` 드라이버의 백엔드 파일 시스템으로 XFS를 권장한다. 단, XFS 파일 시스템 생성 시 `ftype=1` 옵션(`mkfs.xfs -n ftype=1`)을 반드시 활성화해야 한다.4 이 옵션은 디렉터리 엔트리(dentry)에 파일 유형 정보(일반 파일, 디렉터리, 심볼릭 링크 등)를 포함시키는 `d_type` 지원을 활성화한다. OverlayFS는 이 정보를 활용하여 불필요한 `stat` 시스템 콜을 생략할 수 있어, 특히 메타데이터 집약적인 작업의 성능을 향상시킨다. XFS는 대용량 파일 처리와 병렬 I/O에 강점을 보여 일반적으로 선호되지만 30, 작은 파일이 매우 많은 특정 워크로드에서는 Ext4가 더 나은 성능을 보일 수도 있다는 보고도 있다.31 따라서 실제 운영 환경의 워크로드 특성을 기반으로 벤치마킹하여 최적의 파일 시스템을 선택하는 것이 바람직하다.
- **커널 버전의 중요성:** OverlayFS는 리눅스 커널의 일부이므로, 커널의 발전과 함께 지속적으로 개선된다. 최신 리눅스 커널을 사용하는 것은 성능 향상과 새로운 기능 활용을 위해 매우 중요하다. 예를 들어, Linux 5.10 커널에서는 `volatile` 마운트 옵션이 도입되어 특정 상황에서 쓰기 성능을 극대화할 수 있게 되었고 17, Linux 5.15에서는 RCU(Read-Copy-Update) 기반의 조회를 활성화하여 ACL(Access Control List) 조회와 관련된 성능 병목을 해결했다.33 따라서 안정성이 검증된 최신 LTS(Long-Term Support) 커널을 사용하는 것이 OverlayFS의 잠재력을 최대한 활용하는 길이다.
- **마운트 옵션 활용:**
  - `volatile`: 데이터의 영속성보다 극단적인 성능이 요구되는 시나리오(예: CI/CD 파이프라인의 임시 빌드 컨테이너)에서 이 옵션을 사용하면, `upperdir`에 대한 모든 `fsync` 호출을 생략하여 쓰기 I/O를 크게 줄이고 성능을 향상시킬 수 있다.17 단, 시스템 비정상 종료 시 데이터가 유실될 수 있음을 명확히 인지해야 한다.
  - `xino`: 여러 물리적 디바이스에 걸쳐 `lowerdir`와 `upperdir`가 분산되어 있는 복잡한 환경에서 `xino=on` 또는 `xino=auto` 옵션을 사용하면, OverlayFS가 파일 시스템 ID와 inode 번호를 조합하여 고유하고 지속적인 inode 번호를 생성해준다.16 이는 NFS를 통해 OverlayFS 볼륨을 익스포트하거나, 파일 시스템 무결성을 검사하는 도구를 사용할 때 발생할 수 있는 호환성 문제를 해결하는 데 도움이 된다.15


OverlayFS는 강력하고 효율적인 파일 시스템이지만, 모든 기술과 마찬가지로 특정 한계점과 잠재적인 보안 위험을 내포하고 있다. 특히 POSIX 표준과의 완전한 호환성을 제공하지 않는다는 점은 특정 애플리케이션에서 예기치 않은 동작을 유발할 수 있다.


OverlayFS는 성능과 단순성을 위해 POSIX 파일 시스템 표준의 일부를 의도적으로 다르게 구현하거나 지원하지 않는다. 이는 대부분의 애플리케이션에서는 문제가 되지 않지만, 파일 디스크립터나 파일 시스템의 저수준 동작에 의존하는 일부 소프트웨어에서는 호환성 문제를 일으킬 수 있다.


가장 잘 알려진 비호환성 문제 중 하나는 `open(2)` 시스템 콜의 동작과 관련이 있다. POSIX 표준에 따르면, 동일한 파일을 여러 번 열었을 때 반환되는 파일 디스크립터들은 동일한 기본 파일 객체를 참조해야 한다. 그러나 OverlayFS에서는 이 가정이 깨질 수 있다.25

시나리오는 다음과 같다:

1. 애플리케이션이 `lowerdir`에만 존재하는 파일 `foo`를 읽기 전용(`O_RDONLY`)으로 연다. 이 호출은 성공하고 파일 디스크립터 `fd1`을 반환한다. `fd1`은 `lowerdir`에 있는 원본 파일의 inode를 가리킨다.
2. 잠시 후, 동일한 애플리케이션이 같은 파일 `foo`를 읽기-쓰기(`O_RDWR`) 모드로 다시 연다.
3. 이 두 번째 `open` 호출 시, OverlayFS는 쓰기 권한이 요청되었으므로 `copy_up` 연산을 트리거한다. 파일 `foo`는 `upperdir`로 복사된다.
4. 두 번째 호출은 `upperdir`에 새로 생성된 파일의 inode를 가리키는 파일 디스크립터 `fd2`를 반환한다.

결과적으로, `fd1`과 `fd2`는 이름은 같지만 서로 다른 inode를 가진 별개의 파일을 참조하게 되는 비정상적인 상태가 된다.29 이는 파일 잠금이나 데이터 일관성에 의존하는 정교한 애플리케이션에서 예측 불가능한 버그를 유발할 수 있다. 이 문제를 해결하기 위한 일반적인 해결 방안은, `yum-plugin-ovl` 패키지가 사용하는 방식처럼, 파일을 본격적으로 사용하기 전에 `touch`와 같은 명령어로 파일에 먼저 쓰기 작업을 가하여 `copy_up`을 강제로 미리 수행시키는 것이다.25


`rename(2)` 시스템 콜 역시 OverlayFS에서 제한적으로 지원된다. `lowerdir`에만 존재하는 디렉터리나 파일을 `rename`하려고 시도하면, OverlayFS는 이를 다른 파일 시스템 간의 이동으로 간주하여 `EXDEV` ("Invalid cross-device link") 오류를 반환하는 것이 기본 동작이다.15 이는 `rename`의 원자성을 보장하기 위한 방어적인 설계이지만, POSIX 표준을 기대하는 애플리케이션을 중단시킬 수 있다.25

이에 대한 대응 전략으로, 애플리케이션 개발자는 `rename` 호출이 `EXDEV` 오류로 실패할 경우를 대비하여, 파일을 새 위치에 수동으로 복사한 후 원본을 삭제하는 "copy and unlink" 방식의 대체 로직을 구현해야 한다.25 시스템 관리자 수준에서는 커널 설정(`CONFIG_OVERLAY_FS_REDIRECT_DIR=y`)이나 마운트 옵션(`redirect_dir=on`)을 사용하여 커널이 하위 레이어 디렉터리의 이름 변경을 제한적으로 처리하도록 할 수 있다. 이 옵션은 `upperdir`에 리디렉션을 나타내는 특별한 메타데이터를 기록하는 방식으로 동작한다.15


- **파일 잠금(File Locking) 및 `fanotify`:** `copy_up` 연산은 파일의 inode를 근본적으로 변경시키기 때문에, 파일 잠금(예: `fcntl`)이나 파일 시스템 이벤트 알림(`fanotify`)과 같은 inode 기반 메커니즘에서 의미론적 모호성을 야기한다. 예를 들어, `lowerdir`의 inode에 파일 잠금이 설정된 상태에서 해당 파일이 `copy_up` 되면, 그 잠금이 `upperdir`에 생성된 새로운 inode로 자동으로 전이되어야 하는가에 대한 명확한 답을 내리기 어렵다.26

  `fanotify` 역시 마찬가지로, `lowerdir` inode에 설정된 감시가 `copy_up` 이후의 변경 사항을 감지하지 못할 수 있다. 이러한 복잡성 때문에 초기 커널 버전에서는 관련 기능이 제한적으로 동작하거나, 잠금 시도 시 `ENOLCK` 오류를 반환하는 등의 방어적인 접근이 이루어지기도 했다.


CVE-2023-0386은 OverlayFS가 리눅스의 다른 핵심 기능인 사용자 네임스페이스(User Namespace)와 상호작용하는 과정의 허점을 파고드는 대표적인 권한 상승 취약점이다.

- **취약점의 근원:** 이 취약점의 핵심은 OverlayFS의 `copy_up` 로직이 파일의 소유자 정보(UID/GID)를 처리할 때, 해당 UID/GID가 현재 컨텍스트의 사용자 네임스페이스 내에 유효하게 매핑되어 있는지를 검증하지 않는다는 데에 있다.18 사용자 네임스페이스는 컨테이너와 같은 격리된 환경에서 프로세스가 호스트와 다른 사용자 및 그룹 ID를 갖도록 해주는 기능인데, 이 경계에서의 검증 누락이 공격의 빌미를 제공했다.
- **공격 시나리오:** 공격은 다음과 같은 단계로 진행된다 18:
  1. **네임스페이스 생성:** 공격자는 `unshare` 시스템 콜을 사용하여 권한 없는 사용자로서 새로운 사용자 네임스페이스를 생성한다. 이 네임스페이스 안에서 공격자는 가상의 `root` 권한(UID 0)을 갖게 된다.
  2. **악성 `lowerdir` 준비:** 생성된 네임스페이스 내부에서, 공격자는 FUSE(Filesystem in Userspace) 등을 이용해 가상 파일 시스템을 마운트하고, 그 안에 SUID(Set-User-ID) 비트가 설정된 실행 파일을 생성한다. 이 파일의 소유자는 네임스페이스 내부에서는 `root` (UID 0)이지만, 호스트 시스템 관점에서는 공격자의 원래 UID에 매핑된다.
  3. **OverlayFS 마운트 및 `copy_up` 트리거:** 공격자는 이 가상 파일 시스템을 `lowerdir`로, 일반 디렉터리를 `upperdir`로 하여 OverlayFS를 마운트한다. 이후 `chmod`와 같은 메타데이터 변경 명령을 통해 SUID 파일에 대한 `copy_up` 연산을 의도적으로 유발한다.
  4. **취약점 악용:** 취약한 커널의 OverlayFS 구현체는 `copy_up`을 수행하면서 파일의 메타데이터(소유권 및 SUID 비트 포함)를 `upperdir`로 복사한다. 이때, 파일의 소유자 UID가 현재 사용자 네임스페이스의 매핑 범위를 벗어나는지 확인하는 검증 절차가 누락되어, SUID 비트가 그대로 보존된 채 파일이 복사된다.
  5. **권한 상승:** 공격자는 네임스페이스를 빠져나와 호스트 시스템으로 돌아온다. 이제 `upperdir`에는 호스트 시스템의 실제 `root`가 소유하고 SUID 비트가 설정된 실행 파일이 생성되어 있다. 공격자가 이 파일을 실행하면, 프로세스는 `root` 권한으로 실행되어 시스템 전체에 대한 통제권을 장악하게 된다.

이 취약점은 컨테이너 보안이 단순히 컨테이너 이미지나 네트워크 정책을 관리하는 것을 넘어, 컨테이너 런타임과 호스트 커널 간의 복잡한 상호작용에 대한 깊은 이해를 요구함을 명확히 보여준다. 컨테이너가 제공하는 격리는 커널이라는 공유 자원 위에 구축되므로, 커널 자체의 취약점, 특히 OverlayFS, 네임스페이스, cgroup과 같은 컨테이너 핵심 기술과 관련된 취약점은 격리 경계를 무너뜨리는 치명적인 공격 벡터가 될 수 있다. 따라서 진정한 컨테이너 보안(Defense-in-Depth)을 위해서는 Seccomp, AppArmor/SELinux와 같은 커널 보안 모듈을 통해 컨테이너의 시스템 콜 접근을 최소화하고, 비인가된 네임스페이스 생성을 제한하며, 무엇보다 호스트 커널을 항상 최신 상태로 유지하는 등 다층적인 방어 전략이 필수적이다. 이는 제로 트러스트(Zero Trust) 원칙을 커널과 유저스페이스의 상호작용에 적용하는 것과 같은 맥락이다.38


OverlayFS를 안전하게 사용하기 위해서는 다음과 같은 모범 사례를 준수해야 한다.

- **지속적인 커널 업데이트:** CVE-2023-0386과 같은 알려진 취약점은 커널 업데이트를 통해 패치된다. 따라서 시스템의 리눅스 커널을 항상 안정적인 최신 버전으로 유지하는 것이 가장 기본적인 보안 조치이다.
- **네임스페이스 사용 제한:** 신뢰할 수 없는 사용자가 임의로 사용자 네임스페이스를 생성하는 것을 방지하기 위해, 시스템 전역적으로 커널 파라미터 `kernel.unprivileged_userns_clone=0`을 설정하는 것을 고려할 수 있다. 또는 Seccomp 프로파일을 통해 컨테이너가 `unshare`나 `clone` 시스템 콜을 특정 플래그와 함께 호출하는 것을 차단할 수 있다.
- **신뢰할 수 있는 이미지 사용:** 컨테이너 환경에서는 공식적으로 검증된 베이스 이미지만을 사용하고, 출처가 불분명한 이미지를 `lowerdir`로 사용하는 것을 피해야 한다.
- **보안 마운트 옵션 활용:** 컨테이너 외부에서 OverlayFS를 직접 사용할 경우, `nosuid`, `nodev`와 같은 보안 관련 마운트 옵션을 적절히 활용하여 위험을 완화해야 한다.


OverlayFS는 유니온 파일 시스템의 역사 속에서 여러 경쟁 기술과의 상호작용을 통해 현재의 위치에 도달했다. 특히 Aufs와의 관계는 컨테이너 기술 초기의 기술 표준 경쟁을 상징적으로 보여준다.


Aufs(Another Union File System)는 Docker 초창기에 기본 스토리지 드라이버로 사용되며 널리 알려졌다. 기술적으로 Aufs는 OverlayFS보다 일부 더 유연한 기능을 제공했지만, 결정적인 차이로 인해 결국 주류에서 밀려나게 되었다. 두 파일 시스템의 핵심적인 차이점은 다음 표와 같다.

**Table 5.2: 유니온 파일 시스템 비교 (OverlayFS vs. Aufs)**

| 특징 (Feature)         | Aufs (Another Union File System)                             | OverlayFS (Overlay File System)                              |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **커널 메인라인 통합** | **미포함.** 배포판별로 별도의 커널 패치가 필요하여 유지보수가 어렵고, 커널 업데이트에 대한 대응이 느리다.9 | **포함 (Linux 3.18+).** 리눅스 커널의 표준 기능으로, 모든 주요 배포판에서 안정적으로 지원된다.2 |
| **구현 복잡성**        | 코드베이스가 상대적으로 크고 복잡하며, 기능이 풍부하다.9     | 구현이 더 작고 단순하여 커널과의 통합 및 유지보수가 용이하다. |
| **동적 레이어 수정**   | 이미 마운트된 파일 시스템에 동적으로 브랜치(레이어)를 추가하거나 제거하는 유연한 기능을 지원한다.39 | 기본적으로 지원하지 않는다. 레이어 구성은 마운트 시점에 고정되며, 변경을 위해서는 언마운트 후 재마운트해야 한다. |
| **성능**               | 많은 수의 레이어가 중첩될 경우 메타데이터 조회 성능이 저하될 수 있다.9 | `copy_up` 오버헤드는 존재하지만, 페이지 캐시 공유 효율이 뛰어나고 전반적인 성능이 우수하다. |
| **xattr 지원**         | 지원한다.42                                                  | 지원한다.                                                    |
| **현재 상태**          | 대부분의 리눅스 배포판에서 지원이 중단되었으며, 사실상 사용되지 않는(deprecated) 기술로 간주된다.9 | Docker의 공식 권장 스토리지 드라이버이며, 컨테이너 환경의 사실상의 표준(de facto standard)이다. |

Aufs가 제공하는 동적 레이어 수정 기능은 특정 사용 사례에서 매우 매력적이었지만, 그 복잡한 코드베이스는 리눅스 커널 메인라인에 병합되는 데 큰 걸림돌이 되었다.9 커널 개발 커뮤니티는 유지보수성과 안정성을 이유로 Aufs의 통합을 거부했다. 반면, OverlayFS는 더 단순하고 명료한 설계를 바탕으로 커널에 성공적으로 통합될 수 있었다. 장기적인 관점에서 볼 때, 커널과의 통합은 안정적인 업데이트, 넓은 호환성, 그리고 커뮤니티의 지속적인 지원을 의미한다. 결국 이러한 '커널 메인라인'이라는 결정적 요인이 Docker를 포함한 전체 컨테이너 생태계가 Aufs를 버리고 OverlayFS를 표준으로 채택하게 만든 가장 큰 이유가 되었다.9 이는 기술 생태계에서 개별 기능의 우수성보다 표준화와 지속 가능성이 얼마나 중요한지를 보여주는 대표적인 사례다.


UnionFS는 리눅스에서 유니온 마운트 개념을 처음으로 구현한 선구자적인 파일 시스템이다.10 여러 디렉터리를 하나로 합쳐 보여준다는 혁신적인 아이디어를 제시했지만, 초기 구현은 성능과 안정성 측면에서 여러 문제점을 안고 있었다.9 이러한 한계를 극복하기 위해 Aufs가 개발되었고, 이후 커널 통합을 목표로 더 단순화된 접근법을 취한 OverlayFS가 등장하게 되었다. 즉, OverlayFS는 UnionFS의 핵심 아이디어를 계승하되, 리눅스 VFS와의 더 나은 통합과 실용적인 설계를 통해 안정성과 성능을 확보한 결과물이라고 볼 수 있다. UnionFS는 비록 현재는 거의 사용되지 않지만, 그 개념적 유산은 OverlayFS와 현대 컨테이너 기술 속에 깊이 남아있다.



불변 인프라는 현대 클라우드 네이티브 운영의 핵심 패러다임 중 하나로, 운영 중인 프로덕션 서버를 직접 수정(패치, 설정 변경)하는 대신, 변경 사항이 적용된 새로운 서버 이미지를 생성하여 기존 인스턴스를 완전히 교체하는 방식을 의미한다.44 이 접근법은 '구성 드리프트(configuration drift)'를 방지하고, 배포의 예측 가능성과 재현성을 높이며, 롤백을 단순화하는 등 수많은 이점을 제공한다.46

OverlayFS의 아키텍처는 이러한 불변 인프라의 철학과 완벽하게 부합한다. 변경이 불가능한 읽기 전용 `lowerdir`는 테스트와 검증이 완료된 '골든 이미지(golden image)'에 해당하며, 읽기-쓰기가 가능한 `upperdir`는 각 인스턴스가 실행되면서 발생하는 고유한 변경 사항이나 상태(state)를 저장하는 공간에 해당한다.7 이 구조는 '기반은 고정하고 차이점만 관리한다'는 불변 인프라의 핵심 원칙을 파일 시스템 수준에서 자연스럽게 구현한 것이다.

따라서 OverlayFS는 단순히 컨테이너 기술을 넘어, 클라우드 네이티브 운영 모델 전체를 지탱하는 기술적 초석으로 평가받아야 한다. 클라우드 네이티브의 핵심 원칙 중 하나는 개별 서버를 애완동물(Pets)처럼 관리하는 대신 가축(Cattle)처럼 취급하는 불변성이다. 이 원칙을 구현하기 위해서는 인프라를 코드(Infrastructure as Code, IaC)로 정의하고, 배포 단위를 이미지화해야 한다. OverlayFS는 바로 이 '이미지'를 효율적으로 저장하고 실행하는 기술적 기반을 제공한다. 베이스 이미지를 `lowerdir`로, 실행 중 변경 사항을 `upperdir`로 분리함으로써 이미지의 불변성을 보장하고, CoW를 통해 새로운 인스턴스를 빠르고 효율적으로 생성할 수 있게 한다. 이러한 특성 덕분에 OverlayFS는 Docker 컨테이너뿐만 아니라, 가상머신 이미지 관리, IoT 기기의 안전한 펌웨어 업데이트, 서버리스 컴퓨팅 환경의 실행 레이어 구성 등 불변성을 요구하는 모든 현대적 배포 모델에서 핵심적인 역할을 수행할 잠재력을 지니고 있다.


리눅스 커널 개발 커뮤니티는 OverlayFS의 기능과 성능을 지속적으로 개선하고 있다. 최근 커널 릴리스에서는 다음과 같은 발전이 이루어졌다:

- **성능 개선:** RCU 기반 조회 도입, `volatile` 마운트 옵션 추가 등을 통해 특정 워크로드에서의 성능을 꾸준히 향상시키고 있다.17
- **호환성 및 기능 확장:** 대소문자를 구분하지 않는(case-insensitive) 파일 시스템과의 호환성 작업 47, 비특권 사용자도 OverlayFS를 마운트할 수 있도록 하는 기능 추가 48 등 사용성과 호환성을 높이기 위한 노력이 계속되고 있다.
- **메타데이터 처리 개선:** 메타데이터 전용 `copy_up`(`metacopy`) 기능 도입으로 불필요한 데이터 복사를 줄이는 등 효율성을 높이고 있다.19

향후에는 현재 지원되지 않는 하위 레이어의 변경 사항을 상위 레이어에 병합하는 'merge-down' 기능이나, 파일 시스템 수준의 데이터 중복 제거(deduplication)와의 통합 등 더 진보된 기능에 대한 논의가 이루어질 가능성이 있다.


OverlayFS는 단순성, 우수한 성능, 그리고 리눅스 커널 메인라인 통합이라는 강력한 이점을 바탕으로 현대 리눅스 환경에서 가장 중요한 유니온 파일 시스템으로 확고히 자리 잡았다. 특히 컨테이너화된 환경에서 스토리지 효율성, 빠른 배포 속도, 그리고 강력한 격리성을 제공하는 핵심 기술로서 그 가치가 입증되었다. 또한, 불변 인프라라는 현대적 IT 운영 패러다임을 구현하는 데 있어 가장 이상적인 파일 시스템 수준의 해결책을 제시한다.

그러나 OverlayFS를 효과적으로 활용하기 위해서는 그 한계를 명확히 인지하고 전략적으로 접근해야 한다. 시스템 아키텍트와 엔지니어는 다음 사항을 반드시 고려해야 한다:

1. **성능 특성 이해:** `copy_up`으로 인한 첫 쓰기 작업의 오버헤드를 인지하고, 쓰기 집약적인 워크로드에 대해서는 성능 테스트를 통해 영향을 평가해야 한다.
2. **호환성 문제 대비:** `open(2)` 및 `rename(2)` 시스템 콜의 비표준 동작이 애플리케이션에 미칠 수 있는 영향을 파악하고, 필요시 애플리케이션 수정이나 우회 방안을 마련해야 한다.
3. **보안에 대한 지속적인 경계:** OverlayFS와 다른 커널 기능(특히 사용자 네임스페이스)의 상호작용에서 발생하는 보안 취약점에 유의해야 한다. 시스템 커널을 항상 최신 상태로 유지하고, 최소 권한 원칙에 따라 컨테이너의 권한을 제한하는 등 다층적인 보안 전략을 수립하는 것이 필수적이다.
4. **전략적 도구로서의 인식:** OverlayFS를 단순히 파일 시스템으로만 볼 것이 아니라, 컨테이너화, CI/CD, 불변 인프라와 같은 현대적인 아키텍처를 구현하고 가속화하는 핵심적인 전략적 도구로 인식하고 시스템 설계에 적극적으로 반영해야 한다.

결론적으로, OverlayFS는 현대 컴퓨팅 환경의 복잡성과 요구사항을 해결하는 데 있어 필수불가결한 기술이다. 그 구조와 동작 원리, 그리고 잠재적 위험에 대한 깊이 있는 이해를 바탕으로 신중하게 적용할 때, 시스템의 효율성, 안정성, 그리고 보안을 한 차원 높은 수준으로 끌어올릴 수 있을 것이다.


1. 21장. 파일 시스템 | 7.2 릴리스 노트 | Red Hat Enterprise Linux | 7, 8월 15, 2025에 액세스, https://docs.redhat.com/ko/documentation/red_hat_enterprise_linux/7/html/7.2_release_notes/technology-preview-file_systems
2. OverlayFS - Wikipedia, 8월 15, 2025에 액세스, https://en.wikipedia.org/wiki/OverlayFS
3. Overlay filesystem - ArchWiki, 8월 15, 2025에 액세스, https://wiki.archlinux.org/title/Overlay_filesystem
4. Chapter 21. File Systems | 7.2 Release Notes | Red Hat Enterprise Linux | 7, 8월 15, 2025에 액세스, https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/7.2_release_notes/technology-preview-file_systems
5. [Linux] Overlayfs 개념 및 사용 방법 - Jun_ : Pwn - 티스토리, 8월 15, 2025에 액세스, https://she11.tistory.com/241
6. Docker's Internal - 티스토리, 8월 15, 2025에 액세스, https://zbvs.tistory.com/14
7. Desacralizing the Linux overlay filesystem in Docker - Adaltas, 8월 15, 2025에 액세스, https://www.adaltas.com/en/2021/06/03/linux-overlay-filesystem-docker/
8. Immutable infrastructure, containers & the future of microservices [Adam Miller] - YouTube, 8월 15, 2025에 액세스, https://www.youtube.com/watch?v=Qvr_RxaAz78
9. What is AUFS (Another Union File System)? - Artoon Solutions, 8월 15, 2025에 액세스, https://artoonsolutions.com/glossary/aufs/
10. UnionFS - Wikipedia, 8월 15, 2025에 액세스, https://en.wikipedia.org/wiki/UnionFS
11. 컨테이너 이미지 내 overlayFS에서 Copy-on-Write가 구체적으로 ..., 8월 15, 2025에 액세스, https://potato-pg-journey.tistory.com/43
12. Demystifying Overlay File Systems - InfluentCoder, 8월 15, 2025에 액세스, https://influentcoder.com/posts/overlayfs/
13. Prevention of a DoS Attack with Copy-on-write in the Overlay Filesystem - KSL, 8월 15, 2025에 액세스, https://www.ksl.ci.kyutech.ac.jp/papers/2021/satou-dasc2021.pdf
14. Documentation/filesystems/overlayfs.txt - kernel/hikey-linaro - Git at Google, 8월 15, 2025에 액세스, https://android.googlesource.com/kernel/hikey-linaro/+/master/Documentation/filesystems/overlayfs.txt
15. overlayfs.txt - The Linux Kernel Archives, 8월 15, 2025에 액세스, https://www.kernel.org/doc/Documentation/filesystems/overlayfs.txt
16. Overlay Filesystem - The Linux Kernel documentation, 8월 15, 2025에 액세스, https://docs.kernel.org/filesystems/overlayfs.html
17. OverlayFS Adds A "Volatile" Option - Faster Performance But All Syncs Are Omitted, 8월 15, 2025에 액세스, https://www.phoronix.com/news/OverlayFS-Linux-5.10
18. [1-day] CVE-2023-0386(OverlayFS) 분석 - Jun_ : Pwn - 티스토리, 8월 15, 2025에 액세스, https://she11.tistory.com/242
19. UBIFS & OverlayFS Updates Hit The Linux 4.19 Kernel - Phoronix, 8월 15, 2025에 액세스, https://www.phoronix.com/news/Linux-4.19-UBIFS-OverlayFS
20. How to use multiple lower layers in overlayfs - Stack Overflow, 8월 15, 2025에 액세스, https://stackoverflow.com/questions/31044982/how-to-use-multiple-lower-layers-in-overlayfs
21. How to Use OverlayFS? | Baeldung on Linux, 8월 15, 2025에 액세스, https://www.baeldung.com/linux/overlayfs-usage
22. a practical look into overlayfs - ops.tips, 8월 15, 2025에 액세스, https://ops.tips/notes/practical-look-into-overlayfs/
23. Understanding Container Images, Part 3: Working with Overlays - Cisco Blogs, 8월 15, 2025에 액세스, https://blogs.cisco.com/developer/373-containerimages-03
24. 만들면서 이해하는 도커(Docker) 이미지: 도커 이미지 빌드 원리와 ..., 8월 15, 2025에 액세스, https://www.44bits.io/ko/post/how-docker-image-work
25. Use the OverlayFS storage driver | Docker ドキュメント - GitHub Pages, 8월 15, 2025에 액세스, https://matsuand.github.io/docs.docker.jp.onthefly/storage/storagedriver/overlayfs-driver/
26. Overlayfs issues and experiences [LWN.net], 8월 15, 2025에 액세스, https://lwn.net/Articles/636943/
27. Would same file from various docker images be page-cached in k8s node just once?, 8월 15, 2025에 액세스, https://stackoverflow.com/questions/59742644/would-same-file-from-various-docker-images-be-page-cached-in-k8s-node-just-once
28. Chapter 5. Optimizing persistent storage | Scaling and Performance Guide | OpenShift Container Platform | 3.10 | Red Hat Documentation, 8월 15, 2025에 액세스, https://docs.redhat.com/en/documentation/openshift_container_platform/3.10/html/scaling_and_performance_guide/scaling-performance-optimizing-storage
29. OverlayFS storage driver | Docker Docs, 8월 15, 2025에 액세스, https://docs.docker.com/engine/storage/drivers/overlayfs-driver/
30. XFS vs. Ext4: Which Linux File System Is Better? | Pure Storage Blog, 8월 15, 2025에 액세스, https://blog.purestorage.com/purely-educational/xfs-vs-ext4-which-linux-file-system-is-better/
31. 리눅스 파일 시스템 EXT4 vs XFS, 뭘 골라야 할까, 뭐가 더 좋지? : r/DataHoarder - Reddit, 8월 15, 2025에 액세스, https://www.reddit.com/r/DataHoarder/comments/11ar65f/linux_filesystems_ext4_vs_xfs_what_to_choose_what/?tl=ko
32. Splunk Indexers - ext4 vs XFS filesystem performance, 8월 15, 2025에 액세스, https://community.splunk.com/t5/Community-Blog/Splunk-Indexers-ext4-vs-XFS-filesystem-performance/ba-p/693424
33. OverlayFS On Linux 5.15 Improves Performance, Copies Up More Attributes - Phoronix, 8월 15, 2025에 액세스, https://www.phoronix.com/news/Linux-5.15-OverlayFS
34. Use the OverlayFS storage driver - Docker Documentation, 8월 15, 2025에 액세스, https://docker-docs.uclv.cu/storage/storagedriver/overlayfs-driver/
35. OverlayFS limitation - HackMD, 8월 15, 2025에 액세스, https://hackmd.io/@TuI2ECfeSiii4sWQjNf9GA/SkV_pj-18
36. Comparing aufs vs overlay - Page 2 - Puppy Linux Discussion Forum, 8월 15, 2025에 액세스, https://www.forum.puppylinux.com/viewtopic.php?t=1910&start=30
37. CISA Warns of Active Exploitation of Linux Kernel Privilege ..., 8월 15, 2025에 액세스, https://thehackernews.com/2025/06/cisa-warns-of-active-exploitation-of.html
38. Zero Trust Overlays - DoD CIO, 8월 15, 2025에 액세스, https://dodcio.defense.gov/Portals/0/Documents/Library/ZeroTrustOverlays.pdf
39. Comparing aufs vs overlay - Puppy Linux Discussion Forum, 8월 15, 2025에 액세스, https://forum.puppylinux.com/viewtopic.php?t=1910
40. Which union filesystem is best supported? - Ask Ubuntu, 8월 15, 2025에 액세스, https://askubuntu.com/questions/39850/which-union-filesystem-is-best-supported
41. how about overlayfs? / Issue #21 / Tomas-M/linux-live - GitHub, 8월 15, 2025에 액세스, https://github.com/Tomas-M/linux-live/issues/21
42. AkihiroSuda/issues-docker: :whale: Docker Issues and Tips (aufs/overlay/btrfs..) - GitHub, 8월 15, 2025에 액세스, https://github.com/AkihiroSuda/issues-docker
43. Hmm... how is Overlayfs and Unionfs different? From the explanation I can't find... | Hacker News, 8월 15, 2025에 액세스, https://news.ycombinator.com/item?id=21569168
44. Part 2: Immutable Infrastructure: Best Practices for Network Professionals - ZPE Systems, 8월 15, 2025에 액세스, https://zpesystems.com/immutable-infrastructure-best-practices-zs/
45. REL08-BP04 Deploy using immutable infrastructure - Reliability Pillar - AWS Documentation, 8월 15, 2025에 액세스, https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/rel_tracking_change_management_immutable_infrastructure.html
46. What is Mutable vs. Immutable Infrastructure? - HashiCorp, 8월 15, 2025에 액세스, https://www.hashicorp.com/resources/what-is-mutable-vs-immutable-infrastructure
47. New Patch Series Allows OverlayFS To Work With Casefolding - Phoronix, 8월 15, 2025에 액세스, https://www.phoronix.com/news/OverlayFS-Case-Insensitive
48. FUSE, OverlayFS, Ceph Ready With Improvements For Linux 5.11 - Phoronix, 8월 15, 2025에 액세스, https://www.phoronix.com/news/Linux-5.11-FUSE-Ceph-OverlayFS