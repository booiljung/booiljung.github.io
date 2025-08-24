# Ubuntu 22.04 호스트 기반 ROS2 Humble 애플리케이션의 OTA 배포 전략
[우분투 리눅스 (Ubuntu Linux)](./index.md)


현대 로보틱스 시스템의 배포 및 유지보수 패러다임이 근본적으로 변화하고 있습니다. 과거의 정적인 임베디드 시스템과 달리, 오늘날의 로봇은 끊임없이 진화하는 소프트웨어를 통해 그 가치가 결정됩니다. 이러한 환경에서 무선 업데이트(Over-the-Air, OTA)는 단순한 기술적 편의를 넘어, 로봇 플릿(fleet)의 운영, 보안, 그리고 비즈니스 연속성을 위한 핵심적인 전략적 자산으로 자리매김하고 있습니다.


초기 프로토타입 단계를 넘어 수십, 수백 대의 로봇을 현장에 배포하는 순간, 전통적인 수동 업데이트 방식은 심각한 한계에 부딪힙니다. OTA는 이러한 한계를 극복하고 현대적인 로봇 운영을 가능하게 하는 필수 요소입니다.

- **변화의 필연성**: 로봇 애플리케이션은 한번 배포되면 끝나는 것이 아닙니다. 버그 수정, 새로운 기능 추가, 알고리즘 개선, 그리고 보안 취약점 패치를 위해 지속적인 소프트웨어 업데이트가 필수적입니다.1 OTA는 이러한 변화를 현장의 모든 장치에 신속하고 일관되게 적용할 수 있는 유일한 실질적인 방법입니다.
- **수동 개입의 막대한 비용**: 분산된 로봇 플릿의 모든 장치에 기술자가 물리적으로 접근하여 USB나 유선 연결을 통해 업데이트를 수행하는 시나리오는 상상만으로도 물류 및 재정적 악몽입니다. OTA 시스템은 이러한 직접적인 개입의 필요성을 제거함으로써, 막대한 유지보수 비용과 기술 인력의 시간을 절감하고, 결과적으로 높은 투자수익률(ROI)을 제공합니다.2
- **로보틱스를 위한 현대적 DevOps 구현**: OTA는 로보틱스 분야에서 CI/CD(지속적 통합/지속적 배포) 파이프라인의 'CD' 부분을 담당하는 핵심 기술입니다. 이를 통해 개발팀은 새로운 기능을 더 빠르게 시장에 출시하고, 고객의 피드백을 신속하게 제품에 반영하며, 전반적으로 민첩한 제품 개발 주기를 확립할 수 있습니다.5
- **보안의 핵심 동인**: 상호 연결된 세상에서 보안은 선택이 아닌 필수입니다. 새로운 보안 취약점이 발견되었을 때, OTA는 전 세계에 흩어져 있는 플릿 전체에 긴급 보안 패치를 실시간으로 배포하여 위협을 완화할 수 있는 유일하고 실용적인 메커니즘입니다. 이는 장치의 무결성을 보호하고 기업의 자산과 명성을 지키는 데 결정적인 역할을 합니다.2


단순히 파일을 무선으로 전송하는 것을 넘어, 상용 등급의 OTA 시스템은 예측 불가능한 현장 상황에서도 안정성을 보장하는 세 가지 핵심 원칙 위에 구축되어야 합니다.

- **원자성 (Atomicity)**: 업데이트는 '전부 아니면 전무(all or nothing)' 원칙을 따라야 합니다. 업데이트 도중 전원 손실이나 네트워크 중단과 같은 예기치 않은 사건이 발생하더라도, 장치가 부팅 불능 상태("벽돌")가 되거나 불완전하게 업데이트된 불안정한 상태에 빠져서는 안 됩니다.2 이 원자성은 견고한 OTA 시스템의 가장 기본적인 초석입니다.
- **롤백 (Rollback)**: 새로운 업데이트가 성공적으로 설치되었더라도, 예상치 못한 심각한 버그를 유발할 수 있습니다. 이 경우, 시스템은 이전에 안정적으로 작동하던 '알려진 양호한(known-good)' 버전으로 안전하게 되돌아갈 수 있는 신뢰할 수 있는 메커니즘을 갖추어야 합니다. 롤백 기능은 최후의 안전망 역할을 하며, 실패한 배포로 인한 서비스 중단을 최소화합니다.2
- **보안 (Chain of Trust)**: 다층적인 보안 접근 방식이 필수적입니다. 이는 신뢰의 사슬을 구축하여 업데이트 프로세스의 모든 단계에서 무결성을 보장합니다.
  - **장치 신원 확인**: 서버는 합법적인 장치와 통신하고 있음을 확인해야 합니다.
  - **서버 신원 확인**: 장치는 합법적인 서버로부터 업데이트를 수신하고 있음을 확인해야 합니다(주로 TLS 인증을 통해).
  - **업데이트 무결성 및 신뢰성**: 업데이트 페이로드 자체는 암호학적으로 서명되어야 합니다. 이를 통해 업데이트가 신뢰할 수 있는 제조업체로부터 왔으며 전송 중에 변조되지 않았음을 증명할 수 있습니다(코드 서명).4


OTA 업데이트는 그 범위와 방식에 따라 여러 유형으로 나눌 수 있으며, 각 방식은 장단점을 가집니다.

- **전체 시스템 / 루트 파일시스템 업데이트**: 운영 체제 파티션 전체를 교체하는 방식입니다. 이는 커널부터 시스템 라이브러리, 애플리케이션에 이르기까지 소프트웨어 스택의 모든 것을 업데이트할 수 있는 가장 견고한 방법입니다. 일반적으로 A/B 파티션 스킴을 통해 구현되어 원자성과 롤백을 보장합니다.4
- **애플리케이션 수준 업데이트**: 기반 호스트 OS는 그대로 두고 특정 애플리케이션 파일, 패키지 또는 컨테이너만 업데이트하는 방식입니다. 이 방식은 더 빠르고 대역폭 효율적이지만, 커널이나 핵심 시스템 라이브러리와 같은 하위 시스템은 업데이트할 수 없다는 한계가 있습니다.16
- **델타 업데이트 (Delta Updates)**: 이전 버전과 새 버전 간의 바이너리 차이점("델타")만 전송하는 중요한 최적화 기술입니다. 이는 대역폭 소비와 다운로드 시간을 획기적으로 줄여주며, 전체 시스템 이미지와 애플리케이션 업데이트 모두에 적용될 수 있습니다.2

이러한 전략적 고려사항들은 단순히 '어떻게 파일을 무선으로 보낼 것인가'라는 기술적 질문을 넘어섭니다. 이는 '어떻게 수명 주기 동안 로봇 플릿의 가치를 안정적이고 안전하게 유지하고 향상시킬 것인가'라는 근본적인 아키텍처 질문에 대한 답을 요구합니다. 특히, 부트로더 선택이나 파티션 레이아웃과 같은 초기 설계 단계의 결정은 한번 장치가 현장에 배포된 후에는 되돌리기 어렵거나 불가능한 결과를 초래할 수 있으므로 13, 신중한 사전 계획이 무엇보다 중요합니다.


로보틱스 OTA 업데이트 전략을 수립할 때 가장 먼저 마주하게 되는 근본적인 선택지는 '어떻게' 업데이트를 구성하고 전달할 것인가에 대한 철학의 차이입니다. 이는 크게 두 가지 모델, 즉 '요새 모델(Fortress Model)'로 비유되는 이미지 기반 업데이트와 '마이크로서비스 모델(Microservices Model)'로 비유되는 컨테이너 기반 업데이트로 나뉩니다. 이 선택은 단순히 기술 스택의 차이를 넘어, 개발 워크플로우, 시스템의 견고성, 그리고 장기적인 유지보수 전략 전반에 깊은 영향을 미칩니다.


이 모델은 전통적인 임베디드 시스템의 안정성을 최우선으로 하는 접근 방식입니다. 시스템의 핵심을 견고하게 보호하는 요새와 같이, 업데이트 실패로 인한 시스템 불능 상태를 원천적으로 방지하는 데 초점을 맞춥니다.

- **개념**: 저장 장치를 루트 파일시스템을 위한 두 개의 동일한 파티션(A와 B)으로 나눕니다. 시스템은 현재 '활성(active)' 파티션(예: A)으로 부팅하여 실행되며, 새로운 업데이트는 '비활성(inactive)' 파티션(B)에 기록됩니다. 업데이트 파일의 무결성 검증이 완료되면, 부트로더에게 다음 부팅 시 B 파티션으로 부팅하도록 지시합니다. 만약 B 파티션으로의 부팅이 어떤 이유로든 실패하면(예: 커널 패닉, 핵심 서비스 실패), 부트로더는 자동으로 다시 A 파티션으로 부팅하여 시스템을 이전의 안정적인 상태로 복구합니다.4
- **장점**:
  - **최고 수준의 견고성**: 전원 손실이나 네트워크 중단과 같은 극한 상황에서도 업데이트 실패로 인해 장치가 '벽돌'이 되는 것을 거의 완벽하게 방지합니다. 이는 OTA 안정성의 황금 표준으로 여겨집니다.
  - **완벽한 시스템 제어**: 리눅스 커널, 하드웨어 드라이버, 핵심 시스템 라이브러리 등 소프트웨어 스택의 모든 구성 요소를 업데이트할 수 있습니다. 이는 장기적인 시스템 발전에 필수적입니다.
- **단점**:
  - **저장 공간 오버헤드**: 루트 파일시스템을 위해 최소 두 배의 저장 공간이 필요합니다.23
  - **대역폭 집약적**: 전체 이미지 업데이트는 파일 크기가 클 수 있습니다. 다만, 이 문제는 델타 업데이트 기술을 통해 상당 부분 완화될 수 있습니다.
  - **느린 업데이트 주기**: 업데이트 적용을 위해 전체 시스템 재부팅이 필요하므로, 약간의 서비스 중단 시간이 발생합니다.23
- **주요 도구**: Mender 16, RAUC 24, SWUpdate 11와 같은 프레임워크들이 이 철학을 기반으로 설계되었습니다.


이 모델은 클라우드 및 웹 개발에서 발전한 현대적인 소프트웨어 개발 철학을 로보틱스에 적용한 것입니다. 애플리케이션을 독립적인 마이크로서비스로 간주하고, 이를 컨테이너 기술을 통해 빠르고 유연하게 배포하는 데 중점을 둡니다.

- **개념**: 호스트 운영 체제는 안정적이고 최소한의 상태로 유지됩니다. ROS2 애플리케이션과 그 의존성들은 하나 이상의 도커(Docker) 컨테이너로 패키징됩니다. 업데이트는 새로운 버전의 컨테이너 이미지를 원격 레지스트리에서 가져와 기존 컨테이너를 교체하는 방식으로 이루어집니다.25
- **장점**:
  - **개발 속도 향상**: 개발자는 자신의 로컬 머신에서 로봇에서 실행될 환경과 동일한 컨테이너를 빌드하고 테스트할 수 있어, 개발과 배포 사이의 간극을 줄입니다.23
  - **민첩성과 속도**: 전체 시스템 재부팅이 필요 없으므로 업데이트가 매우 빠릅니다. 컨테이너 레이어 수준에서 작동하는 델타 업데이트는 대역폭 효율성이 매우 높습니다.28
  - **격리**: 애플리케이션의 의존성을 호스트 OS로부터 완벽하게 격리하여, 라이브러리 충돌과 같은 문제를 방지합니다.
- **단점**:
  - **제한된 업데이트 범위**: 호스트 OS의 커널이나 드라이버를 업데이트할 수 없습니다. 이는 미래에 애플리케이션이 새로운 커널 기능을 요구하게 될 경우 심각한 제약이 될 수 있습니다.23
  - **오케스트레이션의 복잡성**: 다중 컨테이너 애플리케이션, 특히 ROS2의 DDS(Data Distribution Service) 통신을 위한 네트워킹, 그리고 영구 데이터 관리는 Docker Compose와 같은 도구를 사용한 신중한 오케스트레이션이 필요합니다.23
- **주요 도구**: Balena는 이 모델을 위한 선도적인 통합 플랫폼입니다.20 표준 도커와 사용자 정의 스크립트를 조합하여 직접 구현할 수도 있습니다.


이 접근법은 위 두 모델의 장점을 결합하려는 시도입니다. Balena 20 및 Mender의 고급 구성 29에서 볼 수 있듯이, A/B 파티션 스킴을 사용하여 기반 호스트 OS를 견고하게 업데이트하면서, 동시에 컨테이너를 사용하여 빠르게 변화하는 사용자 애플리케이션(ROS2 노드)을 신속하게 관리하고 업데이트합니다. 이는 로보틱스 시스템이 가진 이중적 특성, 즉 안정적인 하드웨어 제어 계층과 빠르게 진화하는 상위 애플리케이션 계층을 모두 만족시키는 효과적인 전략이 될 수 있습니다.


올바른 모델을 선택하는 것은 단순히 기술적 선호의 문제가 아니라, 프로젝트의 전체 수명 주기를 고려한 전략적 결정입니다.

- 만약 로봇의 하드웨어와 핵심 OS가 제품 수명 동안 안정적으로 유지될 것으로 예상되고, 주로 애플리케이션 로직만 변경된다면, 순수 컨테이너 모델이 개발 속도 면에서 유리할 수 있습니다.
- 반면, 제품이 장기적으로 진화하면서 새로운 센서 지원을 위한 드라이버 추가나 실시간 성능 향상을 위한 커널 패치 등이 필요할 가능성이 있다면, 커널까지 업데이트할 수 있는 이미지 기반 모델이나 하이브리드 모델이 훨씬 안전한 장기적 선택입니다.13

이 결정은 로보틱스가 가진 독특한 위치, 즉 하드웨어와 밀접하게 결합된 임베디드 시스템의 특성과 복잡한 상위 로직이 빠르게 반복 개발되어야 하는 소프트웨어 서비스의 특성 사이의 긴장을 어떻게 해결할 것인가에 대한 답을 찾는 과정입니다. 특히 컨테이너 기반 접근법을 채택할 경우, 전통적인 `ros2 launch` 시스템의 역할이 Docker Compose와 같은 컨테이너 오케스트레이터로 상당 부분 이전되는 패러다임의 전환이 일어난다는 점을 인지해야 합니다.23 이는 ROS 개발팀의 기존 워크플로우와 기술 스택에 상당한 변화를 요구할 수 있습니다.


올바른 OTA 아키텍처 모델을 선택했다면, 다음 단계는 해당 모델을 구현할 최적의 도구를 선택하는 것입니다. 임베디드 리눅스 생태계에는 각각 뚜렷한 철학과 장단점을 가진 여러 강력한 OTA 프레임워크가 존재합니다. 본 섹션에서는 Mender, Balena, RAUC, SWUpdate 네 가지 주요 프레임워크를 심층적으로 비교 분석하여, 프로젝트의 기술적 요구사항, 팀의 역량, 그리고 비즈니스 전략에 가장 부합하는 솔루션을 선택할 수 있도록 돕습니다.


- **핵심 철학**: Mender는 견고하고 안전한 이미지 기반 A/B 업데이트를 위한 오픈소스 엔드투엔드(end-to-end) 솔루션을 지향합니다. '그냥 작동하는' 안정적인 업데이트 시스템을 제공하는 데 중점을 둡니다.16
- **아키텍처**: 장치에서 실행되는 클라이언트와 배포를 관리하는 중앙 관리 서버로 구성됩니다. 서버는 웹 UI를 통해 플릿 관리, 배포 스케줄링, 롤아웃 제어 등의 기능을 제공하며, 클라이언트는 서버의 지시에 따라 실제 업데이트를 수행합니다.16
- **주요 특징**: 원자성 업데이트와 자동 롤백을 보장하는 A/B 파티셔닝이 핵심입니다. 또한, 업데이트 전후에 특정 작업을 수행하기 위한 상태 스크립트, 위험을 최소화하기 위한 단계적 롤아웃, 그리고 대역폭 절약을 위한 델타 업데이트 기능을 지원합니다.16
- **업데이트 모듈**: Mender는 '업데이트 모듈'이라는 확장 기능을 통해 애플리케이션 수준의 업데이트(예: `.deb` 패키지, 스크립트, 도커 컨테이너)도 지원합니다. 하지만 이는 Mender의 주력 기능이 아니며, 예를 들어 `deb` 모듈은 참조 구현 수준으로 롤백을 지원하지 않는 등 기능적 한계가 있습니다.18
- **통합**: Mender의 모든 기능을 최대한 활용하는 가장 권장되는 방법은 Yocto Project와의 깊은 통합입니다. `meta-mender` 레이어를 사용하면 A/B 파티션 레이아웃부터 부트로더 통합까지 복잡한 과정이 자동화됩니다.30 Debian/Ubuntu와 같은 기성 OS에 적용하는 것도 가능하지만, Yocto 기반 접근법에 비해 견고성이 떨어지고 통합이 더 복잡합니다.


- **핵심 철학**: Balena는 컨테이너화된 애플리케이션 배포를 중심으로 구축된, 고도로 통합되고 독자적인(opinionated) 플랫폼입니다. 개발자가 저수준의 OS 관리에 신경 쓰지 않고 애플리케이션 개발에만 집중할 수 있도록 하는 것을 목표로 합니다.20
- **아키텍처**: 최소한의 호스트 OS인 `balenaOS`, 컨테이너 엔진인 `balenaEngine`, 장치 에이전트인 `supervisor`, 그리고 빌드, 배포, 플릿 관리를 위한 백엔드인 `balenaCloud`로 구성된 완전한 생태계입니다.20
- **주요 특징**: 컨테이너 레이어에 최적화된 매우 효율적인 델타 업데이트가 가장 큰 장점입니다. `balena push` 명령어를 통한 원활한 개발-배포 워크플로우, 통합 빌드 서비스, 직관적인 웹 대시보드 및 강력한 CLI를 제공합니다.20 호스트 OS 자체의 업데이트는 내부적으로 A/B 메커니즘을 통해 안전하게 처리됩니다.20
- **통합**: Balena는 호스트 OS를 추상화하여 사용자가 쉽게 시작할 수 있도록 합니다. 이는 완전한 생태계로 설계되었기 때문에 강력한 편의성을 제공하지만, `balenaOS`와 `balenaCloud`에 대한 벤더 종속성(vendor lock-in)이 발생할 수 있습니다. `openBalena`라는 자체 호스팅 가능한 대안이 존재하지만, 상용 클라우드 버전에 비해 기능이 제한적입니다.21


- **핵심 철학**: RAUC는 완전한 서버 솔루션이 아닌, 임베디드 리눅스 장치를 위한 가볍고, 유연하며, 안전한 업데이트 *클라이언트*입니다. 특정 배포 서버에 종속되지 않는 독립적인 구성 요소를 지향합니다.9
- **아키텍처**: 장치에서 실행되는 클라이언트 도구 및 서비스로, D-Bus API나 커맨드 라인 인터페이스를 통해 제어됩니다. RAUC의 핵심 역할은 '번들(bundle)'이라고 불리는 업데이트 패키지를 안전하게 설치하는 것입니다.24
- **주요 특징**: A/B, 복구 파티션 등 매우 유연한 파티션/슬롯 설정을 지원합니다. X.509 인증서를 사용한 강력한 암호화 서명 및 검증, 부트로더와의 긴밀한 통합, 그리고 중간 저장 공간 없이 업데이트를 설치할 수 있는 스트리밍 기능을 제공합니다.9
- **통합**: 의도적으로 특정 배포 서버와 분리되어 설계되었습니다. 따라서 자체적으로 구축한 대규모 시스템에 통합하기 위한 구성 요소로 사용됩니다. Yocto, Buildroot, PTXdist와 같은 빌드 시스템을 완벽하게 지원하며 36, hawkBit과 같은 배포 서버와 연동하여 사용할 수 있습니다.


- **핵심 철학**: SWUpdate는 정형화된 제품이 아니라, 프로젝트의 요구사항에 맞춰 변형될 수 있는 매우 유연하고 사용자 정의 가능한 프레임워크입니다. "SWUpdate를 위해 프로젝트를 바꾸는 것이 아니라, 프로젝트에 맞게 SWUpdate를 조정한다"는 것이 핵심 철학입니다.11
- **아키텍처**: 이미지, 파일, 스크립트, UBI 볼륨 등 다양한 업데이트 유형을 처리하는 풍부한 '핸들러(handler)' 집합을 가진 단일 바이너리로 구성됩니다. 독립 실행형 도구, 내장 웹 서버를 통한 제어, 또는 hawkBit과 같은 백엔드 서버와의 연동 등 다양한 모드로 실행될 수 있습니다.12
- **주요 특징**: 단일 파티션(복구 시스템과 함께 사용) 및 이중 파티션(A/B) 방식을 모두 지원합니다. 하드웨어 호환성 검사, 설치 전/후 스크립트 실행, 그리고 사용자 정의 로직 구현을 위한 강력한 Lua 스크립팅 엔진을 제공합니다.11
- **통합**: 사용자 정의 시스템에 깊숙이 통합되도록 설계되었습니다. RAUC와 마찬가지로 Mender나 Balena보다 훨씬 많은 통합 노력이 필요하지만, 그 대가로 비할 데 없는 유연성을 제공합니다.


아래 표는 네 가지 프레임워크의 핵심적인 특징을 한눈에 비교하여, 각 프로젝트의 상황에 맞는 최적의 도구를 선택하는 데 도움을 줍니다.

| 기능                | Mender                                       | Balena                                                   | RAUC                                                  | SWUpdate                                       |
| ------------------- | -------------------------------------------- | -------------------------------------------------------- | ----------------------------------------------------- | ---------------------------------------------- |
| **주요 모델**       | 이미지 기반 (A/B)                            | 컨테이너 기반 (하이브리드 OS)                            | 이미지 기반 (유연함)                                  | 프레임워크 (유연함)                            |
| **솔루션 유형**     | 엔드투엔드 플랫폼 (클라이언트 + 서버)        | 통합 플랫폼 (OS + 클라우드)                              | 클라이언트 측 도구                                    | 클라이언트 측 프레임워크                       |
| **원자성 업데이트** | 예 (A/B 루트FS)                              | 예 (호스트 OS용 A/B)                                     | 예 (A/B 또는 기타 스킴)                               | 예 (A/B 또는 단일 파티션)                      |
| **롤백**            | 예 (부트로더 통합)                           | 예 (호스트 OS); 애플리케이션은 재배포                    | 예 (부트로더 통합)                                    | 예 (부트로더 통합)                             |
| **업데이트 단위**   | 전체 OS, 애플리케이션 (모듈), 델타           | 호스트 OS, 컨테이너 (효율적 델타)                        | 전체 OS, 파일/아카이브, 델타 (casync)                 | 전체 OS, 파일, 스크립트, 델타 (zchunk)         |
| **보안 모델**       | 서명된 Mender 아티팩트                       | 암호화 통신, balenaCloud 보안                            | 서명된 번들 (x.509 인증서)                            | 서명된 업데이트 (PKI 또는 GPG)                 |
| **호스트 OS 지원**  | **Yocto (권장)**; Debian/Ubuntu 지원         | **balenaOS (필수)**                                      | **Yocto/Buildroot (권장)**; Debian/Ubuntu 패키지 존재 | **Yocto/Buildroot (권장)**; Debian 패키지 존재 |
| **ROS2 통합**       | Yocto `meta-ros` 또는 ROS2를 `.deb`로 패키징 | 도커 컨테이너                                            | Yocto `meta-ros`                                      | Yocto `meta-ros` 또는 커스텀 핸들러            |
| **생태계/라이선스** | 오픈소스 (Apache 2.0) 및 상용 버전           | 오픈코어 (`openBalena`) 및 상용 클라우드 (`balenaCloud`) | 오픈소스 (LGPL-2.1)                                   | 오픈소스 (GPLv2, 라이브러리는 LGPLv2)          |
| **사용 편의성**     | 중간 (Yocto 사용 시); 높음 (서버 UI)         | 높음 (관리형 플랫폼)                                     | 낮음 (높은 통합 노력 필요)                            | 낮음 (높은 설정 노력 필요)                     |


사용자의 요구사항인 'Ubuntu 22.04 호스트'는 현대 로보틱스 개발에서 매우 일반적인 환경입니다. 하지만 이 요구사항은 임베디드 OTA 프레임워크들이 추구하는 최고 수준의 견고성 철학과 본질적인 충돌을 일으킬 수 있습니다. 이 섹션에서는 이 충돌의 본질을 분석하고, 이를 해결하기 위한 두 가지 구체적이고 실용적인 아키텍처 경로를 제시합니다.


- **핵심 과제**: Mender, RAUC와 같은 프레임워크가 최고의 견고성을 달성하는 비결은 시스템 이미지 빌드 프로세스 전체와 부트로더 통합을 완벽하게 제어하는 데 있습니다. 이는 Yocto와 같은 임베디드 리눅스 빌드 시스템의 영역입니다.33 반면, 사용자가 `apt`를 통해 설치한 표준 Ubuntu 22.04 시스템은 이미 모든 것이 빌드되어 제공되는 '바이너리 배포판'입니다. 이러한 '브라운필드(brownfield)' 시나리오에서는 OTA 프레임워크가 요구하는 파티션 레이아웃 변경이나 부트로더 제어와 같은 저수준 제어권을 확보하기가 매우 어렵습니다.
- **'충분히 좋은' 것과 '진정으로 견고한' 것의 트레이드오프**: 표준 Ubuntu 설치에 OTA 기능을 추가하는 것은 불가능하지 않습니다. 예를 들어, Mender의 애플리케이션 업데이트 모듈을 사용하여 특정 패키지나 파일을 업데이트할 수 있습니다. 하지만 이는 전체 시스템의 원자성을 보장하지 못하며, 커널이나 핵심 드라이버를 업데이트할 수 없습니다. 결국, Yocto를 통해 처음부터 시스템을 설계하는 '그린필드(greenfield)' 접근법에 비해 견고성 측면에서 타협이 발생하게 됩니다. 따라서 사용자의 질문에 담긴 숨겨진 모순, 즉 '표준 Ubuntu의 편리함'과 '임베디드급 OTA의 견고성'을 동시에 달성하려는 욕구는 직접적으로 해결되기 어렵습니다. 이 모순을 해결하기 위해서는 둘 중 하나의 축을 중심으로 아키텍처를 재구성해야 합니다.


이 접근법은 대부분의 로보틱스 팀에게 가장 실용적이고 균형 잡힌 해결책이 될 수 있습니다. 이는 사용자의 핵심 요구사항인 '안정적인 애플리케이션 환경'의 정신은 유지하면서, '견고한 OS 업데이트' 문제를 우아하게 해결하기 때문입니다.

- **선택 근거**: Balena는 호스트 OS를 추상화합니다. 개발팀은 복잡한 저수준 OS 관리에 대한 걱정 없이, A/B 업데이트가 내장된 견고하고 관리되는 `balenaOS` 위에서 표준 컨테이너 워크플로우를 통해 ROS2 애플리케이션 개발에만 집중할 수 있습니다.20 이 경로는 견고성, 개발 속도, 사용 편의성 사이에서 최적의 균형을 제공합니다.

- **워크플로우**:

  1. 대상 하드웨어(예: Raspberry Pi 4, Intel NUC, NVIDIA Jetson)에 호환되는 `balenaOS`를 설치합니다.43 이 

     `balenaOS`는 내부적으로 Yocto로 빌드되었지만, 사용자는 Yocto를 직접 다룰 필요가 없습니다.

  2. ROS2 Humble 애플리케이션을 도커를 사용하여 컨테이너화합니다 (세부 사항은 파트 5에서 설명). 애플리케이션의 기반이 되는 컨테이너 이미지는 `ubuntu:22.04`를 사용할 수 있으므로, 개발 환경의 일관성을 유지할 수 있습니다.

  3. 개발자는 `balena push` 명령어를 사용하여 로컬에서 작업한 코드를 Balena 클라우드로 푸시합니다.20

  4. `balenaCloud`는 컨테이너 빌드, 이미지 저장, 그리고 플릿 전체에 대한 효율적인 델타 업데이트 오케스트레이션을 자동으로 처리합니다.20

  5. `balenaOS` 자체에 대한 업데이트가 필요한 경우, 이는 Balena 대시보드를 통해 별도로 관리되며, 내장된 A/B 메커니즘 덕분에 안전하게 수행됩니다.


이 경로는 호스트 OS에 대한 깊은 저수준 제어가 반드시 필요하거나, 특정 벤더에 대한 종속성을 완전히 배제해야 하거나, Balena가 지원하지 않는 독특한 하드웨어를 사용하는 팀을 위한 것입니다. 이 접근법은 '표준 Ubuntu 22.04 설치'라는 개념을 포기하고, 대신 Yocto를 사용하여 Ubuntu와 유사한 기능을 가진 커스텀 OS를 *직접 빌드*하는 것을 전제로 합니다.

- **선택 근거**: 이 방식은 견고성과 사용자 정의 가능성을 극대화합니다. 팀이 시스템의 모든 측면을 완벽하게 제어할 수 있지만, 이를 위해서는 Yocto에 대한 상당한 전문 지식과 학습 곡선을 감수해야 합니다.
- **워크플로우**:
  1. Yocto Project를 설정하고, `meta-openembedded`, `meta-ubuntu`, 그리고 `meta-ros`와 같은 레이어를 사용하여 ROS2 Humble 패키지를 포함하는 커스텀 리눅스 배포판을 빌드합니다.29
  2. 빌드 과정에 `meta-mender` 레이어를 통합하여 A/B 파티션 레이아웃을 생성하고 Mender 클라이언트를 이미지에 설치합니다.31
  3. 빌드의 최종 결과물은 초기 프로비저닝을 위해 장치에 플래시할 수 있는 완전한 디스크 이미지(`.sdimg`)입니다.
  4. 이후의 모든 업데이트는 Mender 아티팩트(`.mender` 파일)로 생성되며, 자체 호스팅하거나 Mender가 호스팅하는 서버를 통해 플릿에 배포됩니다.
  5. 이 접근법은 최고의 견고성과 유연성을 제공하지만, Yocto 빌드 시스템을 유지하고 관리하는 데 상당한 엔지니어링 리소스가 지속적으로 투입되어야 합니다.

결론적으로, 사용자의 초기 질문에 담긴 제약 조건을 해결하는 가장 효과적인 방법은 아키텍처의 관점을 전환하는 것입니다. Balena는 OS를 추상화하여 문제를 해결하고, Yocto+Mender는 OS 자체를 재구성하여 문제를 해결합니다. 이 두 가지 경로 중 하나를 선택하는 것이, 어설프게 표준 Ubuntu에 견고한 OTA를 억지로 끼워 맞추려는 시도보다 훨씬 더 성공적인 결과를 가져올 것입니다.


이 섹션에서는 파트 4에서 권장된 아키텍처, 즉 Balena를 활용한 컨테이너 중심 워크플로우를 구현하기 위한 실용적인 단계별 가이드를 제공합니다. 또한, 여기서 다루는 원칙들은 모든 OTA 배포에 적용될 수 있는 보편적인 모범 사례를 포함합니다.


견고하고 재현 가능한 배포의 첫걸음은 애플리케이션을 올바르게 컨테이너화하는 것입니다. 이는 단순히 코드를 박스에 넣는 행위를 넘어, 애플리케이션의 구조, 의존성, 설정, 그리고 상태를 명확하게 정의하는 설계 과정입니다.

- **`Dockerfile` 작성**:
  - **기반 이미지 선택**: 공식 ROS2 Humble 기반 이미지(`ros:humble-ros-base` 또는 `ros:humble-ros-core`)에서 시작하여 불필요한 도구를 최소화합니다.27
  - **다단계 빌드(Multi-stage builds) 활용**: 빌드 환경과 최종 런타임 환경을 분리하기 위해 다단계 빌드를 사용하는 것이 좋습니다. 이는 최종 이미지 크기를 크게 줄이고 공격 표면을 감소시키는 효과를 가져옵니다.46
  - **의존성 설치**: 빌드 단계에서 ROS2 패키지 소스 코드를 복사하고, `rosdep install`을 사용하여 필요한 모든 시스템 의존성을 설치합니다. `apt-get`을 사용하여 추가적인 라이브러리를 설치할 수도 있습니다.
  - **워크스페이스 빌드**: `colcon build` 명령어를 사용하여 ROS2 워크스페이스를 빌드합니다.
  - **최종 이미지 구성**: 최종 런타임 단계에서는 빌드 단계에서 생성된 `install` 디렉토리와 실행에 필요한 최소한의 런타임 의존성만을 복사합니다.
  - **진입점(Entrypoint) 설정**: ROS2 환경을 설정(`source /opt/ros/humble/setup.bash` 및 `source /ros2_ws/install/setup.bash`)하고 애플리케이션의 주 실행 파일(예: `ros2 launch...`)을 실행하는 `ENTRYPOINT` 스크립트를 작성합니다.47
- **`docker-compose.yml`로 구성하기**:
  - **서비스 정의**: 각 ROS2 노드 또는 논리적 노드 그룹을 별도의 서비스로 정의합니다. 이는 마이크로서비스 아키텍처의 원칙을 따르며, 각 구성 요소의 독립적인 업데이트와 관리를 용이하게 합니다.23
  - **네트워킹**: 동일 장치 내 컨테이너 간의 DDS(Data Distribution Service) 검색을 단순화하기 위해 `network_mode: host`를 사용하는 것이 가장 일반적이고 쉬운 방법입니다. 이는 컨테이너가 호스트의 네트워크 스택을 직접 사용하게 하여 노드 간 통신 문제를 최소화합니다.27
  - **영구 데이터 관리**: SLAM으로 생성된 지도, 로그, 장치별 보정 값 등 컨테이너가 업데이트되거나 재시작되어도 유지되어야 하는 데이터는 명명된 볼륨(named volumes)을 사용하여 관리합니다.27
  - **하드웨어 접근**: 라이다를 위한 `/dev/ttyUSB0`나 카메라를 위한 `/dev/video0`와 같은 하드웨어 장치는 `devices` 키를 사용하여 컨테이너에 직접 전달합니다.27
  - **GPU 가속**: GPU를 사용하는 노드의 경우, Balena의 `docker-compose.yml` 확장 기능을 사용하여 `deploy.resources.reservations.devices` 항목에 GPU 접근 권한을 명시적으로 설정해야 합니다.27


컨테이너화된 애플리케이션이 준비되면, Balena와 같은 OTA 플랫폼을 통해 배포 및 업데이트 수명 주기를 관리합니다.

- **초기 프로비저닝**: balenaCloud에서 플릿(fleet)을 생성하고, 대상 하드웨어에 맞는 `balenaOS` 이미지를 다운로드하여 SD 카드나 eMMC에 플래시합니다. 장치가 부팅되면 자동으로 Balena 클라우드에 등록되고 대시보드에 온라인 상태로 표시됩니다.20
- **최초 배포 (`balena push <fleet-name>`)**: 로컬 개발 환경의 프로젝트 폴더에서 이 명령을 실행하면, 코드가 Balena의 빌드 서버로 전송됩니다. 빌드 서버는 `Dockerfile`에 따라 컨테이너를 빌드하고 플릿을 위한 첫 번째 릴리스를 생성합니다. 이후 플릿의 모든 장치가 이 릴리스를 다운로드하여 실행합니다.20
- **반복적 업데이트**: 소스 코드를 수정한 후 다시 `balena push`를 실행하면 새로운 빌드가 트리거됩니다. 장치의 `supervisor`는 이전 릴리스와의 차이점(델타)만을 효율적으로 다운로드하고, 기존 컨테이너를 중지한 후 새 컨테이너를 시작합니다. 이 과정은 매우 빠르고 효율적인 업데이트 주기를 가능하게 합니다.20
- **개발을 위한 로컬 모드**: `balena push <device-ip>` 워크플로우를 사용하면 클라우드 빌더를 거치지 않고 특정 개발용 장치에 직접 코드를 푸시하고 실시간으로 변경 사항을 확인할 수 있어 개발 속도를 크게 향상시킬 수 있습니다.44


실제 상용 환경에서는 기능 구현을 넘어 보안과 안정성을 보장하기 위한 추가적인 조치가 필수적입니다.

- **안전한 키 관리 및 서명**:
  - Balena가 전송 계층의 보안을 상당 부분 처리하지만, ROS2 노드 간의 DDS 통신 자체를 보호하는 것은 여전히 중요합니다. 이를 위해 SROS2(Secure ROS2)를 사용해야 합니다.50
  - SROS2의 키와 인증서는 로컬에서 안전하게 생성하고, 각 컨테이너에는 해당 노드가 실행되는 데 필요한 최소한의 공개 키와 권한 파일만 주입해야 합니다. 인증 기관(CA)의 개인 키와 같은 민감한 정보는 절대로 장치에 배포해서는 안 됩니다.50
- **안전한 단계적 롤아웃 실행**:
  - 새로운 업데이트를 한 번에 전체 플릿의 100%에 배포하는 것은 매우 위험한 관행입니다. 예상치 못한 문제가 발생할 경우 전체 플릿이 마비될 수 있습니다.51
  - 일반적인 전략은 먼저 내부 테스트 장치나 극소수(1-5%)의 현장 장치에 배포하여 안정성을 검증하는 것입니다. 문제가 없다고 판단되면 점진적으로 롤아웃 대상을 10%, 50%, 100%로 확대해 나갑니다.
  - Mender와 같은 플랫폼은 이 기능을 내장하고 있으며 21, Balena에서는 장치를 다른 플릿으로 이동하거나 릴리스 고정(pinning) 기능을 사용하여 유사한 전략을 구현할 수 있습니다.
- **영구 데이터 및 장치별 설정 관리**:
  - 도커의 명명된 볼륨을 사용하여 업데이트 후에도 보존되어야 할 데이터를 관리하는 것의 중요성을 다시 한번 강조합니다 (예: 지도, 보정 값).
  - 로봇의 이름, 특정 센서의 고유 ID와 같은 장치별 설정은 컨테이너 이미지를 재빌드하지 않고도 관리할 수 있도록 Balena의 장치 환경 변수(Device Environment Variables)를 활용하는 것이 효율적입니다.

이러한 체계적인 접근법을 통해, 컨테이너화는 단순한 패키징 기술을 넘어 애플리케이션의 아키텍처를 명확히 하고, 재현 가능하며 신뢰할 수 있는 배포 파이프라인의 기반을 마련하는 핵심적인 역할을 수행하게 됩니다.


본 보고서는 Ubuntu 22.04 호스트 환경에서 ROS2 Humble 애플리케이션을 OTA로 배포하는 복잡한 과제를 다각도에서 분석했습니다. 최종적으로, 기술적 세부 사항을 넘어 프로젝트의 성공을 좌우할 수 있는 전략적 결정을 내릴 수 있는 프레임워크를 제공하고, 로보틱스 배포의 미래를 조망하며 마무리하고자 합니다.


'최고의' OTA 솔루션은 존재하지 않으며, '가장 적합한' 솔루션만이 존재합니다. 최적의 선택은 기술적 우수성뿐만 아니라 팀의 문화, 비즈니스 목표, 그리고 프로젝트의 맥락에 따라 달라집니다. 따라서 단일 솔루션을 추천하기보다는, 다음의 네 가지 핵심 질문을 통해 각 프로젝트가 스스로 최적의 경로를 찾을 수 있는 의사결정 프레임워크를 제시합니다.

- **질문 1: 당신의 팀은 어떤 핵심 역량을 가지고 있는가?**
  - **웹/클라우드 DevOps 배경**: 만약 팀이 Docker, CI/CD 파이프라인, 클라우드 서비스에 익숙하다면, Balena와 같은 컨테이너 중심의 관리형 플랫폼은 학습 곡선을 최소화하고 개발 속도를 극대화할 수 있는 최상의 선택입니다.
  - **임베디드 리눅스/Yocto 배경**: 만약 팀이 저수준 임베디드 리눅스, 커널 빌드, 그리고 Yocto Project에 대한 깊은 전문성을 보유하고 있다면, Mender, RAUC, 또는 SWUpdate를 사용하여 시스템을 완벽하게 제어하고 맞춤화하는 것이 더 큰 가치를 제공할 수 있습니다.
- **질문 2: 당신의 제품은 앞으로 어떻게 진화할 것으로 예상되는가?**
  - **안정적인 하드웨어 및 OS**: 만약 제품의 수명 주기 동안 하드웨어 구성이 고정되고, 애플리케이션 로직 업데이트가 주를 이룰 것이 확실하다면, 순수 컨테이너 기반 업데이트 모델로도 충분할 수 있습니다.
  - **예측 불가능한 진화**: 만약 미래에 새로운 센서 추가로 인한 드라이버 업데이트, 실시간 성능 개선을 위한 커널 패치, 또는 근본적인 시스템 라이브러리 변경이 필요할 가능성이 있다면, 호스트 OS까지 안정적으로 업데이트할 수 있는 하이브리드 모델(Balena)이나 전체 이미지 기반 모델(Mender+Yocto)이 장기적으로 훨씬 안전한 선택입니다.
- **질문 3: 당신의 비즈니스 모델과 벤더 종속성에 대한 허용 범위는 어떠한가?**
  - **SaaS 모델 선호**: 예측 가능한 월별 비용(SaaS 요금)을 지불하고 인프라 관리 및 유지보수에 대한 부담을 최소화하고 싶다면, BalenaCloud나 Mender의 상용 호스팅 서비스가 매력적일 수 있습니다.
  - **자체 구축 및 소유 선호**: 초기 엔지니어링 투자를 감수하더라도 장기적인 운영 비용을 절감하고 특정 벤더에 대한 종속성을 피하고 싶다면, openBalena, Mender(자체 호스팅), RAUC, SWUpdate와 같은 완전한 오픈소스 솔루션을 기반으로 자체 시스템을 구축하는 것이 적합합니다.
- **질문 4: 당신의 프로젝트 타임라인은 어떠한가?**
  - **빠른 시장 출시(Time-to-Market) 중시**: 가능한 한 빨리 제품을 시장에 출시하고 OTA 기능을 구현해야 한다면, Balena와 같이 즉시 사용 가능한 통합 플랫폼이 가장 빠른 경로를 제공합니다.
  - **장기적인 맞춤화 및 제어 중시**: 초기 개발 기간이 더 걸리더라도 장기적으로 시스템을 완벽하게 제어하고 최적화할 수 있는 유연성을 확보하는 것이 더 중요하다면, Mender/RAUC/SWUpdate와 Yocto를 결합하는 접근법이 더 나은 선택일 수 있습니다.

이 질문들에 대한 답을 바탕으로, 대부분의 현대 로보틱스 팀에게는 **Balena를 사용한 하이브리드 아키텍처**가 가장 균형 잡힌 출발점이 될 것입니다. 이는 ROS 개발자에게 친숙한 컨테이너 워크플로우를 제공하면서도, 저수준 OS 업데이트의 견고성 문제를 해결해주기 때문입니다. 반면, 고도의 제어와 맞춤화가 필수적인 특정 산업(예: 국방, 항공우주, 고도로 규제된 의료기기)에서는 **Yocto와 Mender/RAUC를 결합한 아키텍처**가 더 적합할 수 있습니다.


본 보고서에서 논의된 주제들은 로보틱스 산업이 중요한 변곡점에 있음을 시사합니다.

- **성숙하는 ROS2 플랫폼**: ROS2는 실시간 성능, 보안, 모듈성 측면에서 지속적으로 개선되고 있으며, 이는 상용 제품 환경에 점점 더 적합해지고 있음을 의미합니다.54 안정적인 플랫폼의 존재는 그 위에 견고한 배포 시스템을 구축할 수 있는 기반이 됩니다.
- **IoT와 로보틱스의 융합**: Mender, Balena와 같은 도구들은 본래 더 넓은 IoT 시장에서 출발했지만, 이제는 로보틱스가 가진 독특한 과제(예: 복잡한 소프트웨어 스택, 하드웨어와의 긴밀한 상호작용)에 맞춰 조정되고 있습니다. 이는 두 분야의 기술과 철학이 서로 융합하며 발전하고 있음을 보여줍니다.
- **"RobOps"의 부상**: 이 모든 논의는 결국 "RobOps"라는 새로운 분야의 등장을 가리킵니다. RobOps는 DevOps의 원칙과 관행(CI/CD, 자동화, 모니터링)을 물리적 세계와 상호작용하는 로봇의 개발 및 운영에 적용하는 학문입니다. 이 새로운 패러다임에서, 본 보고서가 심층적으로 다룬 견고하고 안전하며 확장 가능한 OTA 전략은 성공적인 RobOps 이니셔티브를 위한 가장 중요한 초석이 될 것입니다. 미래의 로봇 가치는 하드웨어가 아닌, OTA를 통해 지속적으로 전달되는 소프트웨어에 의해 정의될 것이기 때문입니다.


1. Why do OTA updates matter in IoT? - Arduino Blog, accessed July 24, 2025, https://blog.arduino.cc/2024/06/13/why-do-ota-updates-matter-in-iot/
2. What does OTA mean? [Part I] - Ubuntu, accessed July 24, 2025, https://ubuntu.com/blog/what-does-ota-mean
3. OTA IoT Breakdown: What OTA Is and How It Works in IoT - Memfault, accessed July 24, 2025, https://memfault.com/blog/ota-for-iot/
4. What is OTA Update? - Mender, accessed July 24, 2025, https://mender.io/blog/what-is-ota-update
5. 3 reasons why OTA updates are important [Part II] - Ubuntu, accessed July 24, 2025, https://ubuntu.com/blog/3-reasons-why-ota-updates-are-important-part-ii
6. How to update and manage Linux & ROS2 software on robot fleets - qbee.io, accessed July 24, 2025, https://qbee.io/how-to-update-linux-software-on-robot-fleets-and-ros2/
7. OTA (Over-The-Air) Updates For IoT Devices - Ezurio, accessed July 24, 2025, https://www.ezurio.com/resources/blog/ota-over-the-air-updates-for-iot-devices
8. IoT Firmware Security and Update Mechanisms: A Deep Dive | Encryption Consulting, accessed July 24, 2025, https://www.encryptionconsulting.com/iot-firmware-security-and-update-mechanisms-a-deep-dive/
9. rauc/rauc: Safe and secure software updates for embedded Linux - GitHub, accessed July 24, 2025, https://github.com/rauc/rauc
10. Update Factory - Android OTA and Fleet Management - Kynetics, accessed July 24, 2025, https://www.kynetics.com/android-ota-mdm
11. Main Features – SWUpdate, accessed July 24, 2025, https://swupdate.org/features
12. SWUpdate – Your OTA for Embedded Linux and IOT, accessed July 24, 2025, https://swupdate.org/
13. Challenges With Device OTA Updates and Their Solutions - SoftServe, accessed July 24, 2025, https://www.softserveinc.com/en-us/blog/challenges-with-device-ota-updates
14. A more secure and reliable OTA update architecture for IoT devices - Texas Instruments, accessed July 24, 2025, https://www.ti.com/lit/pdf/sway021
15. Designing for OTA: How to Future-Proof Your Embedded Product from Day One - Promwad, accessed July 24, 2025, https://promwad.com/news/designing-for-ota-future-proofing
16. Introduction | Mender documentation, accessed July 24, 2025, https://docs.mender.io/overview/introduction
17. OTA update: Definition & partitioning best practices - The Embedded Kit, accessed July 24, 2025, https://theembeddedkit.io/blog/ota-update-definition-best-practices/
18. Deb packages - Update Modules - Mender Hub, accessed July 24, 2025, https://hub.mender.io/t/deb-packages/326
19. Over-the-air update - Wikipedia, accessed July 24, 2025, https://en.wikipedia.org/wiki/Over-the-air_update
20. Core Concepts | balena, accessed July 24, 2025, https://docs.balena.io/learn/welcome/primer/
21. Choosing the Right IoT Fleet Management System: A Look at Torizon, Balena and Mender, accessed July 24, 2025, https://www.ics.com/blog/iot-fleet-management-system-torizon-balena-mender
22. Updating Embedded Linux Devices: Update strategies, accessed July 24, 2025, https://mkrak.org/2018/01/10/updating-embedded-linux-devices-part1/
23. Automatic Deployment of ROS2 Based System to remote devices ..., accessed July 24, 2025, https://discourse.ros.org/t/automatic-deployment-of-ros2-based-system-to-remote-devices-dual-copy-or-containers/33884
24. Welcome to the RAUC documentation! - RAUC 1.14.127-e6b61 ..., accessed July 24, 2025, https://rauc.readthedocs.io/en/latest/
25. Architecture Overview | balenaOS, accessed July 24, 2025, https://balena-os.pages.dev/architecture/
26. Define a container - balena docs, accessed July 24, 2025, https://docs.balena.io/learn/develop/dockerfile/
27. The Complete Beginner's Guide to Docker for ROS 2 Deployment (2025) - Robotair, accessed July 24, 2025, https://blog.robotair.io/the-complete-beginners-guide-to-using-docker-for-ros-2-deployment-2025-edition-0f259ca8b378
28. Comprehensive Embedded OTA Guide for IoT Solutions - Witekio, accessed July 24, 2025, https://witekio.com/blog/ota-update-solutions-the-ultimate-guide/
29. Help Automating OS Image Creation with Mkosi for Mender-Compatible Deployment?, accessed July 24, 2025, https://hub.mender.io/t/help-automating-os-image-creation-with-mkosi-for-mender-compatible-deployment/7337
30. README.md - mendersoftware/mender - GitHub, accessed July 24, 2025, https://github.com/mendersoftware/mender/blob/master/README.md
31. Updating Embedded Linux Devices: Mender, accessed July 24, 2025, https://mkrak.org/2018/02/09/updating-embedded-linux-devices-mender/
32. Upgrading | Mender documentation, accessed July 24, 2025, https://docs.mender.io/client-installation/install-with-debian-package/upgrading
33. Yocto and OTA software updates in an IoT project - Mender, accessed July 24, 2025, https://mender.io/blog/yocto-and-ota-software-updates
34. balena docs, accessed July 24, 2025, https://docs.balena.io/learn/welcome/introduction/
35. balena CLI Documentation, accessed July 24, 2025, https://docs.balena.io/reference/balena-cli/latest/
36. RAUC: Safe and Secure OTA Updates for Embedded Linux, accessed July 24, 2025, https://rauc.io/
37. Welcome to the RAUC documentation! - RAUC 1.14.127-e6b61 documentation, accessed July 24, 2025, https://rauc.readthedocs.io/
38. SWUpdate OTA i.MX8MM EVK / i.MX8QXP MEK - NXP Community, accessed July 24, 2025, https://community.nxp.com/t5/i-MX-Processors-Knowledge-Base/SWUpdate-OTA-i-MX8MM-EVK-i-MX8QXP-MEK/ta-p/1307416
39. SWUpdate: software update for embedded system, accessed July 24, 2025, https://sbabic.github.io/swupdate/swupdate.html
40. SWUpdate Best Practice - Embedded Software Update Documentation 2025.05 documentation, accessed July 24, 2025, https://sbabic.github.io/swupdate/swupdate-best-practise.html
41. Updating an embedded system - Embedded Software Update Documentation 2025.05 documentation, accessed July 24, 2025, https://sbabic.github.io/swupdate/
42. ros/meta-ros: OpenEmbedded Layers for ROS 1 and ROS 2 - GitHub, accessed July 24, 2025, https://github.com/ros/meta-ros
43. Machine names and architectures - balena docs, accessed July 24, 2025, https://docs.balena.io/reference/base-images/devicetypes/
44. balena ROS2 Foxy Base container - GitHub, accessed July 24, 2025, https://github.com/balena-io-examples/balena-ros2-foxy-base
45. ROS 2 Humble Hawksbill with Yocto and PetaLinux - Hardware Acceleration in Robotics, accessed July 24, 2025, https://news.accelerationrobotics.com/ros2-humble-yocto-petalinux/
46. Best Practices for Deploying a Production-Ready ROS2 Robot : r/ROS - Reddit, accessed July 24, 2025, https://www.reddit.com/r/ROS/comments/1idvrr4/best_practices_for_deploying_a_productionready/
47. Install ROS2 Humble on Ubuntu 22.04 - YouTube, accessed July 24, 2025, https://www.youtube.com/watch?v=flT3LIIR5qo
48. Intro: Install and Setup ROS2 Humble - ROS2 Tutorial 1 - YouTube, accessed July 24, 2025, https://www.youtube.com/watch?v=0aPbWsyENA8
49. Example projects - balena docs, accessed July 24, 2025, https://docs.balena.io/learn/more/examples/example-projects/
50. Deployment Guidelines - ROS 2 Documentation: Humble documentation, accessed July 24, 2025, https://docs.ros.org/en/humble/Tutorials/Advanced/Security/Deployment-Guidelines.html
51. OTA Best Practices - Getting Started, accessed July 24, 2025, https://docs.aylanetworks.com/docs/command-center-ota-best-practices
52. Beware the OTA: The Dangers of Over the Air Updates - ByteSnap, accessed July 24, 2025, https://www.bytesnap.com/news-blog/beware-ota-dangers-over-the-air-updates/
53. Implementing OTA Updates: Best Practices for Manufacturers, accessed July 24, 2025, https://www.regamiota.com/post/implementing-ota-updates-best-practices-for-manufacturers
54. Building the future of robotics with open-source and ROS 2, accessed July 24, 2025, https://pal-robotics.com/blog/future-of-robotics-open-source-ros-2/
55. ROS 2 (Robot Operating System): overview and key points for robotics software | Robotnik ®, accessed July 24, 2025, https://robotnik.eu/ros-2-robot-operating-system-overview-and-key-points-for-robotics-software/
56. ROS 2 Key Challenges and Advances: A Survey of ROS 2 Research, Libraries, and Applications - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/384942758_ROS_2_Key_Challenges_and_Advances_A_Survey_of_ROS_2_Research_Libraries_and_Applications
57. ROS 2 Key Challenges and Advances: A Survey of ROS 2 Research, Libraries, and Applications - Preprints.org, accessed July 24, 2025, https://www.preprints.org/manuscript/202410.1204/v1

