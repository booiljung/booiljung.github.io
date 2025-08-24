# NVIDIA Jetson Orin 플랫폼을 위한 샌드박스 수준 격리 기술 전략
[NVIDIA Jetson AGX Orin](./index.md)


NVIDIA Jetson Orin 플랫폼은 엣지 컴퓨팅 환경에서 전례 없는 수준의 AI 성능을 제공하며, 복잡하고 다양한 워크로드를 단일 시스템에 통합할 수 있는 가능성을 열었다. 이러한 통합 환경에서는 각 애플리케이션의 안정성, 보안, 리소스 관리를 보장하기 위한 강력한 격리 기술이 필수적이다. 본 섹션에서는 Jetson Orin의 핵심 하드웨어 아키텍처를 분석하고, 플랫폼에 내장된 보안 기능의 역할과 한계를 명확히 정의한다. 이를 통해 시스템 무결성 보호와 애플리케이션 런타임 샌드박싱 사이의 "격리 격차(Isolation Gap)"를 규명하고, 본 보고서에서 다룰 격리 기술의 필요성을 제시한다.


NVIDIA Jetson Orin 플랫폼은 엣지 AI 애플리케이션을 위해 설계된 고성능 시스템 온 모듈(SoM) 제품군이다. 이 플랫폼의 핵심은 강력한 이기종 컴퓨팅 아키텍처에 있으며, 이는 여러 AI 작업을 병렬로 처리하는 데 최적화되어 있다. 최상위 모델인 Jetson AGX Orin은 최대 12개의 Arm Cortex-A78AE CPU 코어, 2048개의 CUDA 코어와 64개의 텐서 코어를 갖춘 NVIDIA Ampere 아키텍처 GPU, 그리고 향상된 심층 학습 가속기(DLA)를 탑재하여 이전 세대인 Jetson AGX Xavier 대비 8배에 달하는 성능을 제공한다.1 Jetson Orin Nano와 같은 하위 모델들도 동일한 Ampere 아키텍처를 공유하며, 가격 대비 뛰어난 AI 성능을 제공하도록 설계되었다.2

이러한 고밀도 컴퓨팅 능력은 단일 Jetson Orin 디바이스가 여러 복잡한 워크로드를 동시에 처리할 수 있음을 의미한다. 예를 들어, 자율주행 로봇은 하나의 Orin 플랫폼 위에서 실시간 인식 파이프라인(카메라, LiDAR 데이터 처리), 경로 계획 및 내비게이션 스택, 그리고 자연어 처리 기반의 사용자 인터페이스를 동시에 구동할 수 있다. 이처럼 다수의 애플리케이션을 하나의 하드웨어에 통합하는 것은 효율성을 극대화하지만, 동시에 각 애플리케이션 간의 잠재적 간섭 문제를 야기한다. 한 애플리케이션의 오류가 다른 애플리케이션의 성능 저하를 유발하거나, 시스템 전체의 안정성을 해칠 수 있다. 또한, 서로 다른 보안 등급을 가진 애플리케이션을 함께 실행할 경우, 하나의 취약점이 시스템 전체로 확산될 위험도 존재한다.

따라서 Jetson Orin의 강력한 하드웨어 통합 능력은 필연적으로 강력하고 예측 가능한 런타임 격리 기술의 필요성을 동반한다. 각 애플리케이션을 독립된 샌드박스 환경에서 실행함으로써 리소스 경합을 방지하고, 보안 경계를 설정하며, 배포 및 관리의 복잡성을 줄일 수 있다. Jetson Orin 제품군은 Nano부터 AGX까지 모두 단일 SoC 아키텍처를 공유하므로, 개발 키트에서 검증된 격리 전략은 모든 타겟 모듈에 일관되게 적용될 수 있다는 장점이 있다.3


Jetson Orin 플랫폼은 단순한 고성능 컴퓨팅 유닛이 아니라, 강력한 보안 기능을 내장한 신뢰할 수 있는 엣지 디바이스다. NVIDIA는 Jetson Linux 보드 지원 패키지(BSP)를 통해 하드웨어 수준에서 시작되는 포괄적인 보안 스위트를 제공한다. 이는 보안 부팅(Secure Boot), OP-TEE(Open Portable Trusted Execution Environment), 디스크 암호화(Disk Encryption), 보안 스토리지(Secure Storage), 메모리 암호화(Memory Encryption) 등을 포함한다.4

보안 부팅은 시스템의 가장 근본적인 신뢰점을 형성한다. 이 기능은 퓨즈(fuse)에 저장된 암호화 키(PKC, SBK 등)를 사용하여 부트체인의 모든 구성 요소(부트로더, 커널 등)가 신뢰할 수 있는 소스에 의해 서명되었는지 검증한다.4 이 과정을 통해 악의적인 펌웨어나 변조된 운영체제가 로드되는 것을 원천적으로 차단하여, 시스템이 항상 신뢰할 수 있는 상태에서 시작됨을 보장한다.5

OP-TEE는 Arm TrustZone 기술을 기반으로 하는 신뢰 실행 환경(TEE)을 제공한다. 최신 Jetson Linux(L4T 36.x 이상)는 OP-TEE를 통합하여, 일반 운영체제가 실행되는 '일반 세계(Normal World)'와 완전히 격리된 '보안 세계(Secure World)'를 구축한다.4 이 보안 세계는 암호화 키 관리, 하드웨어 고유 키(HUK)로부터 새로운 키를 파생하는 등의 매우 민감한 작업을 수행하는 데 사용된다.8 일반 세계에서 실행되는 애플리케이션은 보안 세계의 데이터에 직접 접근할 수 없으므로, 핵심적인 보안 자산을 안전하게 보호할 수 있다.5

이러한 하드웨어 기반 보안 기능들은 "이 디바이스와 그 위에서 실행되는 저수준 소프트웨어를 신뢰할 수 있는가?"라는 질문에 대한 답을 제공한다. 즉, 물리적 탈취, 영속적인 악성코드 감염, 부팅 과정의 하이재킹과 같은 위협으로부터 시스템의 근본적인 무결성을 보호하는 데 그 목적이 있다.


Jetson Orin에 내장된 보안 부팅, OP-TEE, 디스크 암호화와 같은 기능들은 시스템의 무결성과 저장된 데이터를 보호하는 데 매우 효과적이다. 이들은 신뢰할 수 있는 컴퓨팅 기반을 구축하는 핵심 요소다. 하지만 이 기능들의 역할과 본 보고서에서 다루는 '샌드박스 수준 격리' 사이에는 명확한 "격리 격차(Isolation Gap)"가 존재한다.

하드웨어 보안 기능들의 작동 원리를 살펴보면 이 격차는 명확해진다. 보안 부팅은 운영체제 커널이 완전히 로드되기 전에 부트체인을 검증하는 역할을 수행하며, 시스템이 부팅된 후에는 그 역할이 끝난다.6 디스크 암호화는 디바이스의 전원이 꺼져 있을 때 데이터를 보호하는 데 중점을 두며, 실행 중인 두 프로세스 간의 상호작용에는 관여하지 않는다.4

가장 중요한 것은 OP-TEE의 역할이다. OP-TEE는 암호화 키 처리와 같은 매우 민감하고 작은 코드 조각인 "신뢰된 애플리케이션(Trusted Application)"을 위한 강력한 격리 환경, 즉 '보안 세계'를 제공한다.5 그러나 로보틱스 스택, 비디오 분석 파이프라인, AI 추론 서버와 같은 대부분의 일반적인 대규모 애플리케이션들은 '일반 세계'에서 실행된다. OP-TEE는 이 '일반 세계'에서 실행되는 애플리케이션들 사이의 상호작용을 격리하는 메커니즘을 제공하지 않는다. 예를 들어, 동일한 리눅스 사용자 권한으로 실행되는 두 개의 애플리케이션이 있다면, 하드웨어 보안 기능들은 한 애플리케이션이 다른 애플리케이션의 파일 시스템에 접근하거나, 메모리를 과도하게 사용하여 다른 애플리케이션을 불안정하게 만드는 것을 막지 못한다.

이것이 바로 '격리 격차'이다. Jetson Orin의 하드웨어 보안은 시스템 자체의 신뢰성을 보장하지만, 일반 운영체제 내에서 동시에 실행되는 여러 애플리케이션 간의 런타임 격리를 제공하지는 않는다. 이 격차를 메우는 것이 바로 Docker, Podman, LXC와 같은 샌드박스 수준의 격리 기술이다. 따라서 본 보고서에서 분석할 기술들은 하드웨어 보안의 대체재가 아니라, 그 위에 반드시 추가되어야 하는 필수적인 보완 계층이다. 이 기술들은 '일반 세계' 내에서 각 애플리케이션에 독립적인 네임스페이스(파일 시스템, 네트워크 등)와 제어된 리소스(CPU, 메모리)를 할당함으로써, 다중 테넌트, 다중 애플리케이션 환경을 안전하고 안정적으로 운영할 수 있게 해준다.


컨테이너화된 애플리케이션이 NVIDIA의 강력한 GPU 및 기타 하드웨어 가속기를 활용하기 위해서는 호스트 시스템과 컨테이너 사이를 연결하는 특수한 소프트웨어 스택이 필요하다. NVIDIA 컨테이너 스택은 바로 이 역할을 수행하는 핵심 구성 요소다. 본 섹션에서는 이 스택의 아키텍처를 심층적으로 분석하고, 과거의 복잡한 구조에서 업계 표준으로 진화하는 과정을 추적하며, 실제 발견된 보안 취약점을 통해 그 구조적 한계와 미래 방향성을 비판적으로 검토한다.


NVIDIA 컨테이너 툴킷은 컨테이너에 GPU 접근 권한을 부여하는 데 필요한 소프트웨어 라이브러리와 도구들의 집합이다.9 이 툴킷은 여러 핵심 구성 요소로 이루어져 있으며, 각 요소는 GPU 가속을 가능하게 하기 위해 유기적으로 상호작용한다. 주요 구성 요소는 다음과 같다 11:

- `libnvidia-container`: GPU 장치와 드라이버 파일을 컨테이너 네임스페이스에 주입하는 로직을 포함하는 핵심 라이브러리.
- `nvidia-container-cli`: `libnvidia-container`의 기능을 명령줄 인터페이스(CLI)로 제공하는 도구.
- `nvidia-container-runtime-hook`: 컨테이너가 시작되기 직전에 실행되는 'prestart hook' 스크립트.
- `nvidia-container-runtime`: `runc`와 같은 표준 OCI(Open Container Initiative) 런타임을 감싸는 래퍼(wrapper).

Docker와 같은 컨테이너 런타임에서 이 스택의 표준적인 실행 흐름은 다음과 같다. 사용자가 GPU를 사용하는 컨테이너 실행을 요청하면, Docker 데몬은 `nvidia-container-runtime`을 호출한다. 이 래퍼 런타임은 컨테이너의 OCI 사양(config.json 파일)에 `nvidia-container-runtime-hook`을 'prestart' 후크로 주입한 뒤, 표준 `runc`를 호출한다. `runc`는 컨테이너의 네임스페이스와 cgroup을 생성한 후, 컨테이너의 주 프로세스를 시작하기 직전에 등록된 prestart 후크를 실행한다. 이때 실행된 `nvidia-container-runtime-hook`은 `nvidia-container-cli`를 호출하여 호스트 시스템의 GPU 장치 파일(예: `/dev/nvidia0`)과 필요한 드라이버 라이브러리들을 컨테이너 내부로 마운트한다.11

이 아키텍처는 기능적으로는 동작하지만, 구조적으로는 여러 단계의 실행 파일과 런타임 사양 변경에 의존하는 복잡하고 명령형(imperative)인 프로세스다. 이는 컨테이너 런타임과 호스트 드라이버 사이를 잇는 "부서지기 쉬운 다리(Brittle Bridge)"와 같다. 이 다단계 프로세스는 각 연결 지점에서 잠재적인 실패 지점을 내포하며, 전체 시스템을 외부 변화에 취약하게 만든다.

이러한 구조적 취약성은 실제로 여러 문제점을 드러냈다. 첫째, 복잡성은 보안 공격 표면을 넓힌다. 이어지는 2.3절에서 분석할 CVE-2024-0132 취약점은 바로 이 복잡한 후크 실행 과정의 허점을 파고든 사례다.9 둘째, 생태계의 변화에 취약하다. Jetson 플랫폼에서 Docker의 특정 버전 업데이트가 NVIDIA 컨테이너 런타임과의 호환성 문제를 일으켜 전체 컨테이너 시스템을 마비시킨 사례가 이를 증명한다.13 당시 해결책은 문제가 되는 Docker 버전을 다운그레이드하고 업데이트를 막는 것이었는데, 이는 장기적으로 지속 가능한 해결책이 아니다. 따라서 이 후크 기반 아키텍처는 작동은 하지만, 설계상 견고하지 않으며 보안 및 안정성 측면에서 본질적인 한계를 가지고 있다. 이러한 한계는 생태계가 왜 더 현대적이고 표준화된 방식으로 나아가야 하는지에 대한 강력한 논거를 제공한다.


기존 NVIDIA 컨테이너 툴킷의 복잡하고 취약한 아키텍처를 해결하기 위한 대안으로 컨테이너 장치 인터페이스(CDI, Container Device Interface)가 등장했다. CDI는 컨테이너 런타임이 외부 장치에 접근하는 방식을 표준화하기 위한 개방형 사양이다.14 이 모델은 장치 공급업체(예: NVIDIA)가 장치를 컨테이너에서 사용 가능하게 만드는 데 필요한 모든 정보(마운트할 장치 파일, 필요한 라이브러리, 환경 변수 등)를 간단한 명세 파일(일반적으로 YAML 또는 JSON 형식)로 제공하도록 한다.15

NVIDIA는 `nvidia-ctk`라는 도구를 통해 이 CDI 명세 파일을 생성하는 기능을 제공한다. 다음 명령어를 사용하면 시스템에 설치된 GPU 드라이버와 장치를 분석하여 필요한 CDI 명세 파일을 생성할 수 있다 14:

```sh
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
```

단, Jetson Orin 플랫폼에서는 `nvidia-ctk`의 모드 감지 로직에 버그가 있어, 올바른 명세 생성을 위해 `--mode=csv` 플래그를 명시적으로 추가해야 한다.18 이는 CDI 모델 자체의 우수성에도 불구하고 Jetson에서의 구현은 아직 성숙 중이며 플랫폼별 특수 지식이 필요함을 보여준다.

CDI의 가장 큰 장점은 패러다임의 전환에 있다. 기존의 명령형 후크 기반 모델에서 선언형 명세 기반 모델로의 전환은 보안과 견고성을 극적으로 향상시킨다. 복잡하고 높은 권한으로 실행되는 런타임 후크 프로그램을 정적이고 선언적인 텍스트 파일로 대체함으로써, CVE-2024-0132와 같은 공격 표면을 원천적으로 제거한다. 신뢰할 수 있는 컨테이너 런타임(예: Podman)이 이 정적 명세 파일을 파싱하여 필요한 구성을 컨테이너에 직접 적용하므로, 후크 실행 과정에서 발생할 수 있는 경쟁 조건(race condition)과 같은 취약점이 사라진다.

또한, CDI는 컨테이너 런타임과 NVIDIA의 도구를 분리(decouple)하여 더 안정적이고 표준화된 인터페이스를 만든다. 이는 Docker 버전 업데이트로 인해 런타임이 깨지는 것과 같은 호환성 문제를 줄여준다. 즉, CDI는 기존 아키텍처의 "부서지기 쉬운 다리" 문제를 근본적으로 해결하는 현대적인 접근 방식이다. 따라서 보안과 안정성이 중요한 프로덕션 환경에서는 CDI를 지원하는 기술 스택을 채택하는 것이 강력히 권장된다.


NVIDIA 컨테이너 툴킷의 아키텍처가 가진 이론적 위험성은 CVE-2024-0132라는 실제적이고 심각한 보안 취약점을 통해 명백히 드러났다. 이 취약점은 `libnvidia-container` 라이브러리 내에 존재했던 TOCTOU(Time-of-Check to Time-of-Use) 버그로, 공격자가 컨테이너 격리를 탈출하여 호스트 시스템을 완전히 장악할 수 있게 했다.9

이 취약점의 공격 원리는 `nvidia-container-runtime-hook`이 작동하는 방식의 허점을 이용하는 것이다. 후크는 컨테이너가 시작되기 전, 호스트의 권한으로 실행되면서 컨테이너 내부에 필요한 드라이버 라이브러리가 있는지 확인(Check)하고, 이후 해당 라이브러리 파일을 컨테이너 내부의 특정 경로에 마운트(Use)한다. TOCTOU 공격은 이 '확인' 시점과 '사용' 시점 사이의 시간차를 이용한다. 악의적으로 조작된 컨테이너 이미지는 후크가 특정 경로에 파일이 존재함을 확인한 직후, 해당 파일을 호스트의 루트 디렉토리를 가리키는 심볼릭 링크로 교체한다. 그 결과, 후크는 원래 확인했던 안전한 파일 대신, 공격자가 심어놓은 심볼릭 링크를 따라가 호스트의 전체 루트 파일 시스템(`/`)을 컨테이너 내부에 마운트하게 된다.9

이러한 컨테이너 탈출은 치명적인 결과를 초래한다. 공격자는 호스트의 파일 시스템에 접근할 수 있게 되며, 특히 호스트의 Docker 소켓 파일(`docker.sock`)에 접근하여 새로운 권한 상승 컨테이너를 실행함으로써 호스트에 대한 완전한 제어권을 획득할 수 있다.

이 사례 연구는 두 가지 중요한 교훈을 준다. 첫째, 복잡한 후크 기반 아키텍처는 단순한 이론적 불안정성을 넘어 실제적인 공격 표면을 제공한다는 점이다. 이는 2.1절에서 언급한 "부서지기 쉬운 다리"의 위험성을 명확히 입증한다. 둘째, 컨테이너 보안은 절대적이지 않으며, 하드웨어 가속을 활성화하는 메커니즘 자체가 전체 보안 태세의 중요한 일부임을 보여준다. 따라서 이 사례는 보안에 민감한 모든 Jetson Orin 배포 환경에서 기존의 후크 기반 시스템을 지양하고, 이러한 공격 표면을 근본적으로 제거하는 선언적인 CDI 모델로 전환해야 하는 가장 강력한 이유를 제공한다.


Jetson Orin 플랫폼에서 애플리케이션을 격리하는 가장 현실적인 두 가지 기술은 기존의 강자인 Docker와 현대적인 대안인 Podman이다. 본 섹션에서는 이 두 기술을 직접적으로 비교 분석하여, 각각의 아키텍처, 보안 모델, 하드웨어 통합 방식, 그리고 Jetson 생태계 내에서의 안정성을 평가한다.


Docker는 컨테이너 기술의 사실상 표준으로, Jetson 생태계에서도 오랫동안 널리 사용되어 왔다.13 NVIDIA가 제공하는 JetPack SD 카드 이미지에는 Docker가 기본적으로 포함되어 있으며(JetPack 6의 호스트 기반 설치 시에는 제외될 수 있음), 수많은 커뮤니티 프로젝트와 튜토리얼이 Docker를 기반으로 작성되어 있다.13

Docker의 아키텍처는 중앙 집중형 데몬(`dockerd`)이 모든 컨테이너의 생성, 실행, 관리를 총괄하는 클라이언트-서버 모델을 기반으로 한다.23 Jetson에서 GPU를 사용하기 위해 Docker는 `--runtime nvidia` 옵션을 통해 NVIDIA 컨테이너 런타임을 호출하며, 이는 2.1절에서 분석한 후크 기반 메커니즘을 사용한다.11

Docker의 가장 큰 장점은 방대한 생태계와 높은 인지도, 그리고 풍부한 문서이다.25 개발자들은 기존의 지식과 도구를 활용하여 비교적 쉽게 Jetson 프로젝트를 시작할 수 있다. 그러나 Jetson 플랫폼에서의 Docker 사용은 몇 가지 중요한 단점을 안고 있다. 첫째, 데몬 기반 아키텍처는 보안적으로 취약하다. 

`dockerd`는 일반적으로 루트 권한으로 실행되므로, 데몬 자체의 취약점은 호스트 시스템 전체의 보안 위협으로 이어질 수 있다. 둘째, Jetson 생태계 내에서의 안정성 문제가 입증되었다. 앞서 언급했듯이, Docker 28.0.0 버전이 Jetson Linux 커널과의 비호환성으로 인해 작동하지 않아 사용자들이 패키지 버전을 다운그레이드하고 고정해야 하는 불편을 겪었다.13 이 문제는 이후 JetPack 6.2.1에서 해결되었지만 7, 이는 Docker와 Jetson 소프트웨어 스택 간의 결합이 얼마나 취약할 수 있는지를 보여주는 사례다.

결론적으로 Docker는 빠른 시작과 생태계 활용 측면에서 유리하지만, 아키텍처적으로 덜 안전하며 Jetson 환경에서의 소프트웨어 업데이트에 따른 안정성 리스크를 내포하고 있다.


Podman은 Docker의 대안으로 부상하고 있는 데몬리스(daemonless) 컨테이너 엔진이다. 가장 큰 특징은 Docker와 같이 중앙에서 모든 것을 관리하는 데몬 프로세스가 없다는 점이다.23 대신, 사용자가 명령을 내릴 때마다 직접 OCI 런타임(`runc` 등)을 호출하여 컨테이너를 생성하고 관리한다. 이 아키텍처는 엣지 디바이스 환경에 여러 중요한 이점을 제공한다.

첫째, 보안성이 뛰어나다. 상시 실행되는 루트 권한의 데몬이 없기 때문에 공격 표면이 근본적으로 줄어든다. 또한, Podman은 추가 설정 없이 기본적으로 진정한 루트리스(rootless) 컨테이너 실행을 지원하여 보안 태세를 크게 강화한다.24 둘째, 현대적인 하드웨어 통합 방식을 채택했다. Podman은 2.2절에서 설명한 CDI(Container Device Interface)를 공식적으로 지원하는 대표적인 런타임이다.27 사용자는 `podman run --device nvidia.com/gpu=all...`과 같은 간단한 명령으로 CDI 명세를 직접 사용하여 GPU에 접근할 수 있다.14 이는 복잡하고 취약한 레거시 후크 시스템을 완전히 배제하는, 더 안전하고 견고한 방식이다. 마이그레이션을 돕기 위해 Docker의 `--gpus` 플래그와의 호환성도 추가되었다.28

이러한 장점들을 바탕으로, Jetson Orin에서 Docker와 Podman 중 하나를 선택하는 것은 단순한 도구 선호도를 넘어선 전략적인 아키텍처 결정이 된다. Docker는 최대의 생태계 호환성과 익숙함이라는 경로를 제공하지만, 데몬 기반 모델의 구조적 책임과 레거시 NVIDIA 런타임의 "부서지기 쉬운 다리"에 의존하는 비용을 감수해야 한다. 반면, Podman은 데몬리스 아키텍처와 CDI를 통한 네이티브 GPU 통합이라는, 엣지 디바이스에 더 적합하고 안전하며 미래 지향적인 경로를 제시한다.

물론 Podman을 선택할 경우 약간의 생태계 마찰이 발생할 수 있다. 예를 들어, `docker-compose` 대신 `podman-compose`라는 별도의 유틸리티를 사용해야 할 수 있다.23 그러나 장기적인 안정성과 보안이 중요한 새로운 프로덕션 시스템을 설계하는 기술 리더에게 Podman의 아키텍처적 우위는 매우 매력적이다. 반면, 기존 Docker 기반 프로젝트를 신속하게 배포해야 하는 팀은 마이그레이션 비용 때문에 결함에도 불구하고 Docker를 고수할 수 있다.

**표 3.1: Jetson Orin에서의 Docker와 Podman 비교 매트릭스**

| 기능                    | Docker                             | Podman                         | 분석 및 시사점                                               |
| ----------------------- | ---------------------------------- | ------------------------------ | ------------------------------------------------------------ |
| **아키텍처**            | 데몬 기반 (클라이언트-서버)        | 데몬리스                       | Podman의 데몬리스 구조는 단일 장애점(SPOF)과 상시 실행되는 루트 프로세스를 제거하여 엣지 환경에 더 적합하다. |
| **기본 보안**           | 루트풀(Rootful) 기본               | 루트리스(Rootless) 기본        | Podman은 기본적으로 루트리스 모드로 실행되어 최소 권한 원칙을 더 잘 준수하며, 보안 설정이 더 간편하다.26 |
| **GPU 통합 방식**       | NVIDIA 컨테이너 런타임 (후크 기반) | 컨테이너 장치 인터페이스 (CDI) | Docker는 복잡하고 취약한 레거시 후크 시스템에 의존한다. Podman은 현대적이고 안전하며 견고한 CDI 표준을 사용한다.27 |
| **공격 표면**           | 큼 (데몬 소켓, 복잡한 후크)        | 작음 (데몬 없음)               | Podman은 중앙 데몬이 없어 공격 표면이 현저히 작다.24         |
| **생태계 성숙도**       | 매우 높음                          | 성장 중 (CLI 호환)             | Docker는 압도적인 생태계와 커뮤니티 지원을 보유하고 있다. Podman은 Docker CLI와 호환되어 마이그레이션이 용이하다.23 |
| **Jetson에서의 안정성** | 보통 (버전 호환성 문제 발생 이력)  | 높음 (CDI로 인한 디커플링)     | Docker는 Jetson Linux 업데이트와 충돌한 이력이 있다.13 Podman의 CDI 기반 접근은 런타임과 드라이버를 분리하여 안정성을 높인다. |


애플리케이션 컨테이너 외에도 시스템을 격리하는 다른 접근 방식들이 존재한다. 본 섹션에서는 경량 OS 가상화 기술인 LXC와 데스크톱 애플리케이션 배포에 널리 사용되는 Flatpak 및 Snap을 Jetson Orin 플랫폼의 관점에서 평가하고, 그 실현 가능성과 한계를 분석한다.


LXC(Linux Containers)는 단일 애플리케이션이 아닌 전체 운영체제 사용자 공간을 격리하는 OS 수준 가상화 기술이다. 각 LXC 컨테이너는 호스트와 커널을 공유하지만, 마치 가벼운 가상 머신처럼 독립적인 파일 시스템, 프로세스 공간, 네트워크 스택을 가진다.29

LXC가 제공하는 강력한 격리 모델은 매력적이지만, Jetson Orin과 같은 특수 하드웨어 환경에서는 심각한 도전에 직면한다. Docker나 Podman과 달리, LXC에서 GPU와 같은 하드웨어 가속기에 접근하는 표준화된 방법이 없다. 대신, 사용자는 LXC 설정 파일을 수동으로 편집하여 매우 낮은 수준의 구성을 직접 수행해야 한다. 이 과정은 일반적으로 다음 단계를 포함한다 31:

1. `lxc.cgroup2.devices.allow`: cgroup 규칙을 사용하여 컨테이너가 호스트의 특정 장치 노드(예: `/dev/nvidia0`)에 접근할 수 있도록 허용한다.
2. `lxc.mount.entry`: 허용된 장치 노드를 컨테이너 내부의 파일 시스템으로 마운트한다.
3. **UID/GID 매핑**: 권한 없는(unprivileged) 컨테이너의 경우, 컨테이너 내부의 사용자/그룹 ID(UID/GID)와 호스트의 UID/GID를 정확하게 매핑해야 한다. 이 과정은 매우 복잡하고 오류가 발생하기 쉽다.29

이러한 수동적이고 복잡한 설정 과정은 LXC를 "높은 마찰, 높은 제어(High-Friction, High-Control)" 옵션으로 만든다. 개발자는 시스템에 대한 깊은 이해를 바탕으로 모든 것을 직접 제어할 수 있지만, 그 과정에서 사소한 실수 하나가 전체 시스템의 불안정으로 이어질 수 있다.

더욱 우려스러운 점은, 이러한 복잡한 설정을 성공적으로 마쳤다 하더라도 하드웨어 가속 작업의 안정성을 보장할 수 없다는 것이다. NVIDIA 개발자 포럼의 한 사용자는 Jetson Orin에서 LXC 컨테이너 내에서 NVIDIA의 핵심 도구인 `trtexec`(TensorRT 실행기)를 실행했을 때 메모리 관련 오류가 발생했다고 보고했다. 동일한 명령이 호스트에서는 문제없이 작동했음에도 불구하고 컨테이너 내에서는 실패한 것이다.33 이는 겉보기에는 완벽해 보이는 하드웨어 패스스루 설정이라도, NVIDIA 소프트웨어 스택이 요구하는 미묘한 환경을 완벽하게 복제하지 못할 수 있음을 시사한다. 이러한 미세한 차이는 디버깅하기 매우 어려운 런타임 오류로 이어질 수 있다.

결론적으로, LXC는 완전한 OS 환경 격리가 반드시 필요한 특정 시나리오(예: 레거시 소프트웨어 스택 실행)를 위한 전문가용 도구다. 그러나 하드웨어 가속 AI 워크로드를 안정적으로 실행해야 하는 대부분의 Jetson Orin 사용 사례에는 부적합하며, 일반적인 애플리케이션 격리 솔루션으로 권장되지 않는다.


Flatpak과 Snap은 데스크톱 리눅스 환경에서 애플리케이션을 안전하게 배포하고 실행하기 위해 널리 사용되는 샌드박싱 기술이다. 이들은 애플리케이션과 그 의존성을 함께 패키징하여 시스템의 나머지 부분과 격리한다. 그러나 Jetson Orin 플랫폼에서 이 기술들의 현주소는 매우 불안정하며, 프로덕션 환경에 사용하기에는 부적합한 것으로 평가된다.

Snap의 경우, 최근 `snapd` 서비스가 2.70 이상 버전으로 자동 업데이트되면서 Jetson 시스템에서 Chromium과 같은 Snap 애플리케이션들이 실행되지 않는 심각한 문제가 발생했다.34 이 문제의 근본 원인은 새로운 `snapd`가 의존하는 파일 기능(file capabilities)을 지원하기 위해 필요한 커널 설정(`CONFIG_SQUASHFS_XATTR` 등)이 기본 Jetson Linux 커널에 활성화되어 있지 않기 때문이다.34 커뮤니티에서 제시된 유일한 해결책은 `snapd` 패키지를 이전 버전으로 다운그레이드하고, `apt`를 통해 업데이트되지 않도록 '홀드(hold)' 상태로 두는 것이다.34 이는 보안 업데이트를 포함한 모든 향후 업데이트를 차단하는 행위이므로, 프로덕션 시스템에서는 절대 용납될 수 없는 임시방편이다.

Flatpak의 상황도 크게 다르지 않다. Flatpak이 사용하는 샌드박싱 기술인 `bubblewrap`은 `CONFIG_NAMESPACES`와 `CONFIG_USER_NS`와 같은 특정 커널 옵션을 필요로 한다.37 이 옵션들 역시 기본 Jetson Linux 커널에서 비활성화되어 있을 수 있다. 사용자가 이 옵션들을 활성화해달라고 요청했을 때, NVIDIA의 공식적인 답변은 "개발자 가이드를 참조하여 직접 커널을 빌드하라"는 것이었다.37 이는 플랫폼 공급업체가 Flatpak을 일급 시민으로 취급하지 않으며, 지원의 책임을 사용자에게 전가하고 있음을 보여준다.

설상가상으로, Flatpak에서 GPU 하드웨어 가속을 활성화하는 과정은 매우 복잡하다. 사용자는 직접 커스텀 Flatpak 런타임을 빌드하거나 다운로드하고, 환경 변수를 설정하며, 여러 개의 `flatpak override` 명령을 통해 수동으로 장치 접근 권한을 부여해야 한다.38 이는 Docker나 Podman의 간편한 GPU 활성화 방식과 극명한 대조를 이룬다.

이러한 증거들을 종합해 볼 때, Snap과 Flatpak은 현재 Jetson Orin 플랫폼에서 성숙하거나 안정적인 기술이라고 보기 어렵다. 두 기술 모두 기본 커널에 없는 특정 설정에 근본적으로 의존하고 있어 본질적으로 취약하다. 이를 해결하기 위한 방법(중요 시스템 패키지 동결, 커널 재컴파일, 복잡한 수동 설정)은 모두 안정적이고 확장 가능하며 안전한 배포에 필요한 요구사항을 위배한다. 따라서 NVIDIA가 공식적인 지원과 함께 필요한 기능이 활성화된 기본 커널을 제공하기 전까지, 이 기술들은 실험적인 용도로만 간주해야 하며, 모든 중요한 프로덕션 배포에서는 사용을 피해야 한다.


컨테이너 기술을 도입할 때 가장 중요한 고려사항 중 하나는 성능이다. 이론적으로 컨테이너는 가상 머신에 비해 오버헤드가 거의 없다고 알려져 있지만, 실제 엣지 AI 워크로드 환경에서는 어떠한 "컨테이너 세금(Container Tax)"이 발생하는지 정량적으로 파악하는 것이 중요하다. 본 섹션에서는 Jetson Orin에서의 컨테이너 성능 오버헤드에 대한 증거를 수집하고, 이를 최소화하기 위한 실질적인 모범 사례를 제시한다.


"컨테이너는 거의 네이티브 성능을 제공한다"는 일반적인 주장에도 불구하고, Jetson Orin에서 컨테이너화된 AI 워크로드의 성능을 체계적으로 분석한 공개 벤치마크 데이터는 부족한 실정이다. 현재 가용한 정보는 대부분 개별 사용자의 경험에 기반한 일화적 증거이며, 그 결과는 상충되거나 때로는 우려스러운 수준의 오버헤드를 시사한다.

한 NVIDIA 개발자 포럼 사용자는 동일한 작업을 호스트에서 실행했을 때 26ms가 걸렸지만, Docker 컨테이너 내에서는 47ms가 소요되어 약 80%의 지연 시간 증가를 경험했다고 보고했다.39 이는 무시할 수 없는 수준의 성능 저하다. 반면, YOLOv5를 테스트한 다른 사용자는 네이티브 환경(130ms)과 Docker 컨테이너(약 170ms) 사이에서 상대적으로 작은 차이를 보고했지만, 이 경우 주된 병목 현상이 컨테이너 자체가 아닌 디스크 I/O였다고 언급했다.40

이러한 상반된 결과는 컨테이너 오버헤드의 크기가 애플리케이션의 특성에 따라 크게 달라질 수 있음을 암시한다. 일반적인 컨테이너 성능 연구에 따르면, CPU 및 메모리 집약적인 작업의 오버헤드는 낮은 반면, 네트워크 및 I/O 집약적인 워크로드에서는 성능 저하가 관찰될 수 있다.41 AI 추론 워크로드는 모델 파일 로딩(스토리지 I/O), 데이터 스트림 처리(네트워크 또는 카메라 I/O) 등 I/O 작업이 빈번하게 발생하므로, 이러한 오버헤드에 민감할 수밖에 없다. 또한, Jetson Orin 플랫폼 자체가 네이티브 환경에서도 높은 커널 실행 오버헤드를 보일 수 있다는 보고는 42, 컨테이너화 계층에 의해 이러한 지연 시간이 더욱 악화될 수 있는 잠재적 민감성을 시사한다.

결론적으로, 상당한 "컨테이너 세금"이 존재할 수 있으며, 그 규모는 정량화되지 않았고 매우 가변적이다. 오버헤드의 원인은 네임스페이스나 cgroup과 같은 핵심 격리 기술 자체보다는, Docker의 브리지 네트워킹, 스토리지 볼륨 드라이버, 그리고 NVIDIA 컨테이너 런타임의 간접 호출과 같은 주변 인프라에서 비롯될 가능성이 높다. 따라서 개발자는 "거의 네이티브 성능"을 가정해서는 안 되며, 반드시 자신의 특정 애플리케이션에 대한 벤치마킹을 수행하여 실제 성능 영향을 평가해야 한다.


컨테이너화로 인한 성능 오버헤드는 불가피할 수 있지만, 적절한 구성과 최적화 전략을 통해 그 영향을 최소화할 수 있다. Jetson Orin에서 컨테이너 성능을 극대화하기 위한 몇 가지 핵심적인 모범 사례는 다음과 같다.

네트워킹 최적화:

컨테이너의 기본 네트워킹 모드인 bridge 모드는 가상 브리지와 NAT(Network Address Translation) 계층을 거치기 때문에 지연 시간을 유발하고 처리량을 감소시킬 수 있다.43 실시간 비디오 스트리밍이나 빠른 센서 데이터 처리가 필요한 지연 시간에 민감한 애플리케이션의 경우, 

`host` 네트워킹 모드(`--net=host`)를 사용하는 것이 강력히 권장된다. 이 모드는 컨테이너가 호스트의 네트워크 스택을 직접 사용하게 하여 네트워킹 오버헤드를 완전히 제거한다.20

스토리지 최적화:

AI 워크로드는 수 기가바이트에 달하는 대용량 모델 파일과 데이터셋을 사용하므로 스토리지 성능이 전체 시스템 성능에 큰 영향을 미친다. Jetson Orin Nano 개발 키트의 SD 카드나 내장 eMMC는 성능이 제한적일 수 있다. 따라서 고성능 NVMe SSD를 추가하고, 컨테이너 관련 데이터를 모두 SSD로 이전하는 것이 중요하다. 구체적인 절차는 다음과 같다 43:

1. Docker 데이터 루트 디렉토리(`/var/lib/docker`)를 NVMe SSD로 옮긴다. 이는 Docker 데몬 설정 파일(`/etc/docker/daemon.json`)의 `"data-root"` 항목을 SSD의 마운트 경로로 지정하여 수행할 수 있다.
2. 대용량 데이터셋이나 모델 파일은 Docker가 관리하는 볼륨 대신, 호스트의 SSD 경로를 컨테이너에 직접 마운트(bind mount)하는 방식이 더 빠를 수 있다.

컴퓨팅 성능 설정:

어떤 성능 테스트를 수행하든, 먼저 Jetson 디바이스가 최대 성능 모드로 설정되어 있는지 확인해야 한다. 다음 명령어를 사용하여 전력 모델을 MAXN으로 설정하고 모든 클럭을 최대로 고정할 수 있다. 이는 네이티브 환경과 컨테이너 환경 모두에 적용되는 필수 전제 조건이다.45

```
sudo nvpmodel -m 0
```

```
sudo jetson_clocks
```

이미지 최적화:

컨테이너 이미지의 크기는 배포 속도와 스토리지 사용량에 직접적인 영향을 미친다. 다단계 빌드(multi-stage builds)를 사용하여 최종 이미지에는 빌드에만 필요했던 도구나 라이브러리가 포함되지 않도록 하고, Alpine Linux와 같이 가벼운 베이스 이미지를 사용하면 이미지 크기를 크게 줄일 수 있다.43

이러한 모범 사례들을 적용함으로써 개발자는 "컨테이너 세금"을 상당 부분 완화하고, 컨테이너 환경에서도 네이티브에 가까운 성능을 달성할 수 있다.


이론적인 분석을 넘어, 실제 Jetson Orin 환경에서 컨테이너 기술을 구현할 때 개발자들은 다양한 실용적인 문제에 직면하게 된다. 본 섹션에서는 CSI 카메라, DLA, NVENC/NVDEC와 같은 특수 하드웨어 주변 장치를 컨테이너 내부에서 접근하는 방법과, 스토리지 및 네트워킹 병목 현상을 해결하는 구체적인 절차를 안내하여 개발자들에게 유용한 실무 가이드를 제공한다.


CSI 카메라:

MIPI CSI 인터페이스를 통해 연결된 카메라는 Jetson 플랫폼에서 고대역폭, 저지연 비디오 스트리밍을 위해 널리 사용된다. 컨테이너 내부에서 CSI 카메라에 접근하려면, 호스트의 해당 장치 노드(일반적으로 /dev/video* 형태)를 컨테이너에 마운트해야 한다. 이는 Docker 또는 Podman 실행 시 --device /dev/video0:/dev/video0와 같은 플래그를 사용하여 수행할 수 있다.47 Jetson Orin Nano는 22핀 CSI 커넥터를 사용하므로, Raspberry Pi 카메라 모듈과 같은 일반적인 15핀 카메라를 연결하기 위해서는 어댑터 케이블이 필요할 수 있다.48 카메라가 올바르게 인식되고 작동하는지 확인하는 가장 좋은 방법은 NVIDIA가 제공하는 `nvgstcapture-1.0` 유틸리티를 호스트와 컨테이너 양쪽에서 실행해보는 것이다.47

심층 학습 가속기(DLA):

Jetson Orin에 탑재된 DLA는 GPU와 독립적으로 동작하는 전용 CNN 가속 하드웨어다. DLA를 사용하면 GPU의 부하를 줄이거나, GPU와 DLA에서 서로 다른 네트워크를 병렬로 실행하여 전체 시스템 처리량을 높일 수 있다. 컨테이너 내부에서 DLA를 사용하려면, 먼저 컨테이너가 NVIDIA 런타임으로 실행되어야 한다. 그 다음, 애플리케이션 코드 내에서 TensorRT API를 사용하여 DLA를 타겟으로 지정해야 한다. 예를 들어, TensorRT 빌더 설정을 config.default_device_type = trt.DeviceType.DLA로 지정하면 TensorRT가 해당 네트워크를 DLA에서 실행하도록 컴파일한다.50 Savant와 같은 일부 프레임워크는 이 과정을 추상화하여 YAML 설정 파일에 `enable_dla: true`와 같은 간단한 플래그를 추가하는 것만으로 DLA를 활성화할 수 있게 해준다.51

`jetson_dla_tutorial` GitHub 저장소는 이 과정을 보여주는 좋은 예제 코드를 제공한다.52

NVENC/NVDEC:

하드웨어 비디오 인코더(NVENC)와 디코더(NVDEC)는 비디오 처리 파이프라인에서 CPU 부하를 크게 줄여주는 필수적인 구성 요소다. 컨테이너가 NVIDIA 런타임으로 실행되면, NVENC/NVDEC에 접근하는 데 필요한 드라이버 라이브러리와 장치 노드들이 자동으로 매핑된다. 따라서 컨테이너 내부의 GStreamer나 FFmpeg와 같은 멀티미디어 프레임워크는 추가 설정 없이 nvv4l2decoder, nvv4l2h264enc와 같은 NVIDIA 가속 플러그인을 사용할 수 있다.53 Frigate과 같은 애플리케이션은 Jetson용 Docker 실행 명령에 이러한 장치들이 올바르게 전달되도록 사전 구성된 옵션을 제공하기도 한다.54 만약 LXC를 사용한다면, 이 모든 장치 노드들을 수동으로 패스스루 설정해야 한다.29

**표 6.1: 컨테이너 하드웨어 접근 구성 치트 시트**

| 하드웨어        | Docker/Podman (NVIDIA 런타임/CDI)               | LXC (수동 구성)                                              |
| --------------- | ----------------------------------------------- | ------------------------------------------------------------ |
| **GPU (CUDA)**  | `--gpus all` 또는 `--device nvidia.com/gpu=all` | `lxc.cgroup2.devices.allow: c 195:* rwm` 및 `lxc.mount.entry: /dev/nvidia0...` |
| **DLA**         | 컨테이너 내 TensorRT API로 활성화               | `lxc.mount.entry: /dev/nvhost-dla-0...` (가상, 연구 필요)    |
| **NVENC/NVDEC** | FFmpeg/GStreamer 플러그인으로 자동 활성화       | `lxc.mount.entry: /dev/nvhost-nvdec...` (가상, 연구 필요)    |
| **CSI 카메라**  | `--device /dev/videoX`                          | `lxc.mount.entry: /dev/videoX...`                            |
| **호스트 SSD**  | `-v /host/path:/container/path` (바인드 마운트) | `mp0: /host/path,mp=/container/path`                         |


Jetson Orin에서 컨테이너를 운영할 때 개발자들이 가장 흔하게 겪는 문제는 스토리지 용량 부족과 네트워킹 성능 저하이다. 이러한 병목 현상은 애플리케이션의 안정성과 성능에 직접적인 영향을 미치므로, 반드시 해결해야 한다.

스토리지 병목 해결:

대용량 AI 모델과 컨테이너 이미지는 수십 기가바이트의 공간을 차지할 수 있다. Jetson Orin Nano 개발 키트의 기본 스토리지인 SD 카드나 저용량 eMMC는 금방 가득 차게 되며, 이는 컨테이너가 시작되지 않거나 실행 중 충돌하는 주된 원인이 된다.56

이 문제에 대한 가장 효과적인 해결책은 고속 NVMe SSD를 주 스토리지로 활용하는 것이다. 다음은 `jetson-ai-lab`에서 제안하는 구체적인 절차를 요약한 것이다 44:

1. **SSD 포맷 및 마운트:** NVMe SSD를 `ext4` 파일 시스템으로 포맷하고, `/ssd`와 같은 마운트 포인트를 생성한 뒤 마운트한다.
2. **자동 마운트 설정:** 시스템이 재부팅될 때마다 SSD가 자동으로 마운트되도록 `/etc/fstab` 파일에 SSD의 UUID를 사용하여 항목을 추가한다.
3. **Docker 데이터 루트 변경:** Docker 서비스를 중지한 후, Docker 데몬 설정 파일인 `/etc/docker/daemon.json`을 열어 `"data-root"` 속성값을 SSD의 마운트 경로(예: `"/ssd/docker"`)로 지정한다.
4. **기존 데이터 이전 및 서비스 재시작:** 기존의 `/var/lib/docker` 디렉토리 내용을 새로운 데이터 루트로 복사하고, Docker 서비스를 재시작하면 모든 컨테이너 이미지와 볼륨이 SSD에 저장되어 스토리지 병목 현상을 해결하고 I/O 성능을 크게 향상시킬 수 있다.

네트워킹 병목 해결:

컨테이너의 기본 bridge 네트워크 모드는 사용 편의성을 제공하지만, 내부적으로 가상 스위치와 NAT를 거치기 때문에 성능 저하를 유발한다.43 특히 실시간 비디오 스트리밍이나 로보틱스 제어와 같이 낮은 지연 시간이 매우 중요한 애플리케이션에서는 이 오버헤드가 문제가 될 수 있다.

이 문제에 대한 해결책은 5.2절에서 언급했듯이 `--net=host` 옵션을 사용하여 호스트 네트워킹을 사용하는 것이다.20 이 옵션을 사용하면 컨테이너가 호스트의 네트워크 인터페이스에 직접 바인딩되어, 가상화 계층을 거치지 않고 네이티브와 동일한 네트워크 성능을 얻을 수 있다. 이는 설정의 복잡성을 약간 증가시킬 수 있지만(예: 포트 충돌 가능성), 성능이 최우선인 애플리케이션에는 필수적인 최적화다.


지금까지 Jetson Orin 플랫폼의 하드웨어 기반, NVIDIA 컨테이너 스택의 아키텍처, 그리고 다양한 격리 기술들의 특성과 성능, 실제 구현상의 난제들을 심층적으로 분석했다. 본 마지막 섹션에서는 이 모든 분석을 종합하여, 기술 리더와 시스템 아키텍트가 정보에 입각한 전략적 결정을 내릴 수 있도록 명확하고 실행 가능한 권고안을 제시한다.


다음 표는 본 보고서에서 분석한 핵심 기준에 따라 각 격리 기술을 평가하고 요약한 것이다. 이는 프로젝트의 특정 요구사항(예: 최고 수준의 보안, 가장 빠른 개발 속도)과 각 기술의 강점 및 약점을 연계하여 최적의 선택을 할 수 있도록 돕는 전략적 프레임워크를 제공한다.

**표 7.1: Jetson Orin 격리 기술 결정 매트릭스**

| 기술             | 보안 태세                                                    | 성능 오버헤드                                                | 하드웨어 접근성                                              | 생태계 및 안정성                                             | 권장 사용 사례                                               |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Docker**       | **보통**. 데몬 기반 아키텍처와 루트풀 기본 설정은 잠재적 위험을 내포함. | **가변적**. I/O 및 네트워크 집약적 워크로드에서 상당한 오버헤드 발생 가능.39 | **높음 (간편성)**. NVIDIA 런타임 후크를 통해 `--runtime nvidia`로 간편하게 활성화. | **매우 높음**. 방대한 커뮤니티, 문서, 사전 빌드된 이미지. 단, Jetson Linux 버전과 호환성 문제 이력 있음.13 | **신속한 프로토타이핑, 기존 Docker 생태계 활용**.            |
| **Podman**       | **높음**. 데몬리스, 루트리스 기본 아키텍처가 공격 표면 최소화. 네이티브 CDI 지원으로 레거시 후크의 보안 위험 회피.26 | **가변적 (이론적 우위)**. 데몬 오버헤드가 없으나, 핵심 성능은 Docker와 유사할 것으로 예상됨. | **높음 (견고성)**. 표준 CDI 명세를 통해 `--device` 플래그로 안정적이고 견고하게 활성화.27 | **성장 중**. Docker CLI와 호환. CDI 표준을 통해 장기적 안정성 확보. | **최고 수준의 보안 및 장기 유지보수가 필요한 프로덕션 환경**. |
| **LXC**          | **매우 높음**. 완전한 OS 사용자 공간 격리 제공.              | **알 수 없음**. 체계적 벤치마크 부재. 잘못된 설정 시 심각한 성능 저하 가능. | **낮음 (높은 복잡성)**. 전문가 수준의 수동 cgroup/마운트 설정 필요. 하드웨어 가속 라이브러리와의 불안정성 보고됨.33 | **낮음**. Jetson 관련 자료가 부족하고, 하드웨어 가속 설정이 비표준적이며 불안정함. | **전체 시스템 가상화가 반드시 필요한 전문가용 특수 사례**.   |
| **Flatpak/Snap** | **보통 (이론적)**. 데스크톱용으로 설계된 샌드박스.           | **알 수 없음**. Jetson에서의 성능 데이터 부재.               | **매우 낮음**. GPU 가속을 위한 설정이 매우 복잡하고 비공식적임.38 | **매우 낮음 (불안정)**. 기본 Jetson 커널 설정과 비호환 문제로 현재 프로덕션 사용 불가.34 | **권장하지 않음**.                                           |


위의 분석과 결정 매트릭스를 바탕으로, 특정 목표를 가진 프로덕션 환경에 대한 구체적인 권고안은 다음과 같다.

- 최고 수준의 보안과 미래 대비를 위한 선택 (예: 의료 기기, 중요 인프라, 국방)

  권장 기술: Podman (CDI 사용)

  Podman의 데몬리스 아키텍처, 기본 루트리스 실행 모델, 그리고 안전하고 표준화된 CDI 메커니즘을 통한 GPU 통합은 보안과 장기적인 유지보수성이 최우선 순위인 프로젝트에 가장 적합한 선택이다. 이는 현재 Jetson Orin에서 가장 견고하고 안전하며 미래 지향적인 컨테이너화 방식이다.

- 신속한 프로토타이핑과 기존 생태계 활용을 위한 선택 (예: 학술 연구, 스타트업 초기 개발)

  권장 기술: Docker (NVIDIA 컨테이너 런타임 사용)

  Docker의 방대한 생태계(사전 빌드된 AI 컨테이너, 튜토리얼 등)는 개발을 빠르게 시작할 수 있는 가장 쉬운 경로를 제공한다.21 아키텍처적으로 Podman보다 열등하지만, 즉각적인 생산성이 중요할 때 합리적인 선택이 될 수 있다. 단, 이 선택은 보안 및 안정성 측면의 트레이드오프를 수반하므로, Docker 버전 관리와 호스트 보안 강화 등 완화 전략을 반드시 병행해야 한다.

- 전체 시스템 가상화가 필요한 선택 (예: 레거시 소프트웨어 스택 이식, 다중 OS 환경)

  권장 기술: LXC (강력한 주의 요망)

  이 선택은 애플리케이션 컨테이너화가 불충분하고, 팀이 복잡하고 취약한 하드웨어 패스스루 설정을 직접 관리할 수 있는 심도 있는 리눅스 시스템 관리 전문 지식을 보유한 경우에만 고려해야 한다. Jetson 하드웨어 가속 기능과의 완전한 호환성을 보장하기 위해 광범위한 검증과 테스트가 필수적이다.

- 데스크톱 스타일 애플리케이션 배포

  권장 기술: 없음

  Flatpak과 Snap은 현재 Jetson 플랫폼에서 안정적으로 지원되지 않으므로, 신뢰성이 요구되는 어떠한 환경에서도 사용을 권장하지 않는다.


Jetson 생태계의 컨테이너 기술은 명확하게 CDI를 통한 표준화 방향으로 나아가고 있다. 앞으로 Docker와 같은 주요 컨테이너 런타임들이 네이티브 CDI 지원을 강화함에 따라, 복잡하고 취약했던 레거시 후크 기반 시스템은 점차 사라지고 생태계 전반의 보안과 안정성이 향상될 것이다. 엣지 디바이스의 보안 중요성이 부각되면서, 아키텍처적으로 우월한 Podman의 채택률은 점차 증가할 것으로 예상된다.

Flatpak과 Snap의 미래는 전적으로 NVIDIA가 향후 Jetson Linux 릴리스에서 필요한 커널 기능들을 기본적으로 활성화하고 공식적인 지원을 제공할 의지가 있는지에 달려있다.

결론적으로, Jetson Orin에서의 안전하고 고성능인 컨테이너화의 미래는 밝지만, 그 중심에는 Podman과 CDI가 제시하는 현대적이고 표준화된 모델이 자리 잡고 있다. 기술 리더와 아키텍트는 이러한 흐름을 이해하고, 현재의 기술적 제약과 미래의 방향성을 모두 고려하여 자신의 프로젝트에 가장 적합한 격리 전략을 수립해야 한다.


1. NVIDIA Jetson AGX Orin Development Guide | RidgeRun, accessed July 16, 2025, https://developer.ridgerun.com/wiki/index.php/NVIDIA_Jetson_Orin
2. Everything You Need to Know about Jetson Orin Nano - Elecrow, accessed July 16, 2025, https://www.elecrow.com/blog/everything-you-need-to-know-about-jetson-orin-nano.html
3. NVIDIA Jetson AGX Orin Developer Kit User Guide, accessed July 16, 2025, https://developer.nvidia.com/embedded/learn/jetson-agx-orin-devkit-user-guide/index.html
4. Security - Jetson Linux Developer Guide documentation, accessed July 16, 2025, https://docs.nvidia.com/jetson/archives/r35.4.1/DeveloperGuide/text/SD/Security.html
5. Enable security features on NVIDIA Jetson Devices - RidgeRun, accessed July 16, 2025, https://www.ridgerun.com/post/securing-jetson-devices-enable-security-features-on-nvidia-jetson-devices
6. Secure Storage - Jetson AGX Orin - NVIDIA Developer Forums, accessed July 16, 2025, https://forums.developer.nvidia.com/t/secure-storage/279809
7. Jetson Linux - NVIDIA Developer, accessed July 16, 2025, https://developer.nvidia.com/embedded/jetson-linux
8. Secure Storage - NVIDIA Jetson Linux Developer Guide 1 documentation, accessed July 16, 2025, https://docs.nvidia.com/jetson/archives/r35.5.0/DeveloperGuide/SD/Security/SecureStorage.html
9. How Wiz found a Critical NVIDIA AI vulnerability: Deep Dive into a ..., accessed July 16, 2025, https://www.wiz.io/blog/nvidia-ai-vulnerability-deep-dive-cve-2024-0132
10. A Beginner's Guide to NVIDIA Container Toolkit on Docker | by Umberto Junior Mele, accessed July 16, 2025, https://medium.com/@u.mele.coding/a-beginners-guide-to-nvidia-container-toolkit-on-docker-92b645f92006
11. Architecture Overview - NVIDIA Container Toolkit - NVIDIA Docs, accessed July 16, 2025, https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/arch-overview.html
12. Demystifying NVIDIA Docker Containers: How GPUs Power Your AI Applications - Medium, accessed July 16, 2025, https://medium.com/@skalaliya/demystifying-nvidia-docker-containers-how-gpus-power-your-ai-applications-5e66f27f99e8
13. Docker Setup on JetPack 6 - Jetson Orin - JetsonHacks, accessed July 16, 2025, https://jetsonhacks.com/2025/02/24/docker-setup-on-jetpack-6-jetson-orin/
14. Support for Container Device Interface - NVIDIA Container Toolkit, accessed July 16, 2025, https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/cdi-support.html
15. RenaudWasTaken/cdi: Container Device Interface - Devices for Linux containers - GitHub, accessed July 16, 2025, https://github.com/RenaudWasTaken/cdi
16. Container Device Interface Support in the GPU Operator - NVIDIA Docs, accessed July 16, 2025, https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/cdi.html
17. GPU container access - Podman Desktop, accessed July 16, 2025, https://podman-desktop.io/docs/podman/gpu
18. Cannot passthrough GPU to Kubernetes pod on the Jetson AGX Orin dev kit - #3 by AastaLLL - NVIDIA Developer Forums, accessed July 16, 2025, https://forums.developer.nvidia.com/t/cannot-passthrough-gpu-to-kubernetes-pod-on-the-jetson-agx-orin-dev-kit/328539/3
19. Podman + GPU on Jetson AGX Orin - NVIDIA Developer Forums, accessed July 16, 2025, https://forums.developer.nvidia.com/t/podman-gpu-on-jetson-agx-orin/297734
20. Your First Jetson Container | NVIDIA Developer, accessed July 16, 2025, https://developer.nvidia.com/embedded/learn/tutorials/jetson-container
21. The jetson-examples repository by Seeed Studio offers a seamless, one-line command deployment to run vision AI and Generative AI models on the NVIDIA Jetson platform. - GitHub, accessed July 16, 2025, https://github.com/Seeed-Projects/jetson-examples
22. balena-io-experimental/jetson-sample-new: Jetson samples for CUDA and OpenCV on Nano, TX2, Xavier and Orin - GitHub, accessed July 16, 2025, https://github.com/balena-io-experimental/jetson-sample-new
23. Podman vs. Docker: Containerization Tools Comparison - Spacelift, accessed July 16, 2025, https://spacelift.io/blog/podman-vs-docker
24. Podman vs Docker: Key Differences for Local Dev 2025 - CyberPanel, accessed July 16, 2025, https://cyberpanel.net/blog/podman-vs-docker
25. NVIDIA Docker: GPU Server Application Deployment Made Easy, accessed July 16, 2025, https://developer.nvidia.com/blog/nvidia-docker-gpu-server-application-deployment-made-easy/
26. Docker vs Podman: An In-Depth Comparison (2025) - DEV Community, accessed July 16, 2025, https://dev.to/mechcloud_academy/docker-vs-podman-an-in-depth-comparison-2025-2eia
27. Using CDI (Container Device Interface) for NVIDIA GPUs with Podman - Christian Taillon, accessed July 16, 2025, https://christiant.io/cdi
28. Podman --gpus all support instead of --devices nvidia.com/gpu=all for compability / Issue #21156 / containers/podman - GitHub, accessed July 16, 2025, https://github.com/containers/podman/issues/21156
29. GPU passthrought to LXC containers and one VM at the same time : r/Proxmox - Reddit, accessed July 16, 2025, https://www.reddit.com/r/Proxmox/comments/1iq6dr5/gpu_passthrought_to_lxc_containers_and_one_vm_at/
30. Install Ubuntu in an LXC/LXD container on an NVIDIA Jetson Nano, accessed July 16, 2025, https://forums.developer.nvidia.com/t/install-ubuntu-in-an-lxc-lxd-container-on-an-nvidia-jetson-nano/238421
31. GPU Passthrough to LXC : r/Proxmox - Reddit, accessed July 16, 2025, https://www.reddit.com/r/Proxmox/comments/1auspst/gpu_passthrough_to_lxc/
32. [GUIDE] GPU passthrough on Unprivileged LXC with Jellyfin on Rootless Docker - Reddit, accessed July 16, 2025, https://www.reddit.com/r/Proxmox/comments/1gh6pje/guide_gpu_passthrough_on_unprivileged_lxc_with/
33. Jetson Orin 32G. trtexec onnx->trt inside container - NVIDIA Developer Forums, accessed July 16, 2025, https://forums.developer.nvidia.com/t/jetson-orin-32g-trtexec-onnx-trt-inside-container/335641
34. Chromium, other browsers not working after flashing or updating - Here's WHY and QUICK FIX - Jetson Orin Nano - NVIDIA Developer Forums, accessed July 16, 2025, https://forums.developer.nvidia.com/t/chromium-other-browsers-not-working-after-flashing-or-updating-heres-why-and-quick-fix/338891
35. Jetson orin nano - Browser issue - NVIDIA Developer Forums, accessed July 16, 2025, https://forums.developer.nvidia.com/t/jetson-orin-nano-browser-issue/338580
36. Fresh JetPack Flash on AGX Orin - SELinux error "matchpathcon not found" prevents Snap apps from running - NVIDIA Developer Forums, accessed July 16, 2025, https://forums.developer.nvidia.com/t/fresh-jetpack-flash-on-agx-orin-selinux-error-matchpathcon-not-found-prevents-snap-apps-from-running/339022
37. Enable `CONFIG_NAMESPACES` and `CONFIG_USER_NS` in final L4T 32.X release, accessed July 16, 2025, https://forums.developer.nvidia.com/t/enable-config-namespaces-and-config-user-ns-in-final-l4t-32-x-release/268031
38. GPU Hardware Accelerated Flatpaks for Jetson (t210, t186, t194 ..., accessed July 16, 2025, https://forums.developer.nvidia.com/t/gpu-hardware-accelerated-flatpaks-for-jetson-t210-t186-t194-and-t234/240787
39. Performance difference of the same task between docker and host on AGX orin, accessed July 16, 2025, https://forums.developer.nvidia.com/t/performance-difference-of-the-same-task-between-docker-and-host-on-agx-orin/223540
40. Jetson nano poor performance with yolov5n and cuda activated - Stack Overflow, accessed July 16, 2025, https://stackoverflow.com/questions/75607688/jetson-nano-poor-performance-with-yolov5n-and-cuda-activated
41. Performance Characterization of Containers in Edge Computing - arXiv, accessed July 16, 2025, https://arxiv.org/html/2505.02082v2
42. Very long kernel launch overhead on Jetson Orin NX - NVIDIA Developer Forums, accessed July 16, 2025, https://forums.developer.nvidia.com/t/very-long-kernel-launch-overhead-on-jetson-orin-nx/307370
43. Common Issues in Docker Containerization and How to Resolve Them, accessed July 16, 2025, https://www.xavor.com/blog/common-issues-in-docker-containerization/
44. SSD + Docker - NVIDIA Jetson AI Lab, accessed July 16, 2025, https://www.jetson-ai-lab.com/tips_ssd-docker.html
45. Jetson-storage error? - NVIDIA Developer Forums, accessed July 16, 2025, https://forums.developer.nvidia.com/t/jetson-storage-error/322772
46. MLPerf Inference v2.1 NVIDIA-Optimized Inference on Jetson Systems - GitHub, accessed July 16, 2025, https://github.com/mlcommons/inference_results_v2.1/blob/master/closed/NVIDIA/README_Jetson.md
47. Taking Your First Picture with CSI or USB Camera - NVIDIA Developer, accessed July 16, 2025, https://developer.nvidia.com/embedded/learn/tutorials/first-picture-csi-usb-camera
48. Using the Jetson Orin Nano with CSI Cameras - JetsonHacks, accessed July 16, 2025, https://jetsonhacks.com/2023/04/05/using-the-jetson-orin-nano-with-csi-cameras/
49. CSI Camera setup - Holybro Docs, accessed July 16, 2025, https://docs.holybro.com/autopilot/pixhawk-baseboards/pixhawk-jetson-baseboard/csi-camera-setup
50. Run a part of DNN on DLA and part of DNN on GPU - NVIDIA Developer Forums, accessed July 16, 2025, https://forums.developer.nvidia.com/t/run-a-part-of-dnn-on-dla-and-part-of-dnn-on-gpu/230169
51. Using DLA on Nvidia Jetson - Savant 0.4.3 documentation, accessed July 16, 2025, https://docs.savant-ai.io/v0.4.3/advanced_topics/14_jetson_dla.html
52. NVIDIA-AI-IOT/jetson_dla_tutorial: A tutorial for getting started with the Deep Learning Accelerator (DLA) on NVIDIA Jetson - GitHub, accessed July 16, 2025, https://github.com/NVIDIA-AI-IOT/jetson_dla_tutorial
53. Jetson Software Architecture - NVIDIA Jetson Linux Developer Guide - NVIDIA Docs, accessed July 16, 2025, https://docs.nvidia.com/jetson/archives/r36.4.3/DeveloperGuide/AR/JetsonSoftwareArchitecture.html
54. Hardware Acceleration | Frigate, accessed July 16, 2025, https://docs.frigate.video/configuration/hardware_acceleration/
55. Get NVENC/Cuda working in Debian 10 LXC with e.g. ffmpeg : r/homelab - Reddit, accessed July 16, 2025, https://www.reddit.com/r/homelab/comments/jbq9h6/get_nvenccuda_working_in_debian_10_lxc_with_eg/
56. Troubleshooting - Jetson Platform Services documentation - NVIDIA Docs, accessed July 16, 2025, https://docs.nvidia.com/jetson/jps/JPS_Troubleshooting.html

