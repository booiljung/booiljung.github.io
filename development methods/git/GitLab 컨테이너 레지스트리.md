[git](./index.md)
# GitLab 컨테이너 레지스트리


현대의 소프트웨어 개발에서 컨테이너 기술은 애플리케이션의 개발, 배포, 실행 방식을 근본적으로 변화시켰습니다. 이러한 컨테이너 기반 워크플로우의 중심에는 컨테이너 이미지를 체계적으로 저장, 관리, 배포하는 '컨테이너 레지스트리'가 자리 잡고 있습니다. 컨테이너 레지스트리는 단순히 불변의 이미지 파일을 보관하는 저장소를 넘어, 애플리케이션과 그 모든 종속성을 하나의 패키지로 묶고 1, 시스템 간에 이를 원활하게 공유하는 핵심 중개자 역할을 수행합니다.2 이는 특히 마이크로서비스 아키텍처로 구성된 클라우드 네이티브 애플리케이션 개발에서 필수적인 인프라로 간주됩니다.2

이러한 맥락에서 GitLab 컨테이너 레지스트리는 단순한 기능 중 하나가 아닌, GitLab이 지향하는 '완전한 단일 DevOps 플랫폼' 전략의 핵심 축을 담당합니다.4 GitLab은 소스 코드 관리(SCM), 지속적 통합 및 배포(CI/CD), 패키지 관리, 보안 등 소프트웨어 개발 수명주기(SDLC)에 필요한 모든 도구를 하나의 통합된 인터페이스와 단일 권한 모델 아래에서 제공하는 것을 목표로 합니다.4 컨테이너 레지스트리는 이 비전의 중심에서, 코드 작성부터 최종 배포에 이르기까지의 전 과정을 매끄럽게 연결하는 중추적인 역할을 합니다.

GitLab 컨테이너 레지스트리의 진정한 가치는 독립적인 기능으로서가 아니라, GitLab 플랫폼의 다른 핵심 기능들과의 유기적인 결합에서 발현됩니다. 소스 코드 관리, CI/CD 파이프라인, 그리고 내장된 보안 스캐닝 기능과의 긴밀한 통합은 GitLab 컨테이너 레지스트리를 단순한 '이미지 저장소'를 넘어, 코드 커밋부터 빌드, 취약점 분석, 그리고 최종 배포 가능한 아티팩트에 이르기까지 모든 산출물과 그 과정을 추적하고 관리하는 **'DevSecOps 아티팩트 허브'**로 기능하게 만듭니다.

이러한 통합은 다음과 같은 흐름을 통해 이루어집니다. 첫째, GitLab 컨테이너 레지스트리는 플랫폼에 내장되어 있어, 별도의 설치나 복잡한 구성 없이 프로젝트를 생성하는 것만으로도 자동으로 활성화되고 사용할 수 있습니다.1 둘째, CI/CD 파이프라인은 

`$CI_REGISTRY_USER`, `$CI_REGISTRY_IMAGE`와 같은 사전 정의된 변수들을 통해 레지스트리와 원활하게 상호작용하며, 인증, 이미지 빌드, 푸시, 배포의 전 과정을 완벽하게 자동화합니다.7 셋째, 내장된 보안 스캐닝 기능은 CI/CD 파이프라인의 일부로 실행되어 레지스트리에 푸시된 이미지를 자동으로 스캔하고, 발견된 취약점 결과를 병합 요청(Merge Request)이나 파이프라인 결과 페이지에 직접 표시하여 개발자가 컨텍스트 전환 없이 즉각적으로 문제를 인지하고 조치할 수 있도록 돕습니다.8 이처럼 모든 과정이 GitLab이라는 단일 플랫폼 내에서 투명하게 연결되고 관리되므로, 컨테이너 레지스트리는 더 이상 고립된 아티팩트 저장소가 아닌, 전체 개발 프로세스의 맥락과 히스토리를 담고 있는 살아있는 중심 허브가 되는 것입니다. 본 보고서는 GitLab 컨테이너 레지스트리의 기술적 구조부터 전략적 활용 방안, 시장 내 경쟁 구도와 미래 방향성까지 다각도로 심층 분석하여, 기술 리더와 엔지니어들이 정보에 기반한 최적의 의사결정을 내릴 수 있도록 지원하는 것을 목표로 합니다.


GitLab 컨테이너 레지스트리의 아키텍처와 관리 모델을 이해하는 것은 그 기능과 한계를 파악하고, 조직의 요구사항에 맞게 최적으로 구성 및 운영하기 위한 첫걸음입니다. 기반 기술부터 스토리지 백엔드, 그리고 차세대 아키텍처의 진화 과정까지 심층적으로 분석합니다.


GitLab 컨테이너 레지스트리의 근간은 널리 사용되는 오픈소스 프로젝트인 **Docker Distribution Registry**입니다.10 GitLab은 이 견고한 기반 위에 자체적인 고유의 인증 및 권한 관리 레이어를 추가하여 플랫폼과의 완벽한 통합을 구현했습니다.

인증 흐름은 다음과 같이 동작합니다. 개발자가 로컬 머신에서 `docker login registry.example.com` 명령을 실행하면, 이 요청은 먼저 레지스트리 백엔드로 전달됩니다. 레지스트리는 유효한 인증 토큰이 없음을 확인하고, HTTP `401 Unauthorized` 응답과 함께 토큰을 발급받을 수 있는 주소(`token_realm`)를 클라이언트에게 반환합니다.10 이 주소는 GitLab의 내부 API 엔드포인트(예: `https://gitlab.example.com/jwt/auth`)를 가리키도록 설정되어 있습니다.10 Docker 클라이언트는 이 주소로 다시 요청을 보내 사용자 자격 증명을 전달하고, GitLab API는 이를 검증한 후 해당 사용자의 권한이 담긴 JWT(JSON Web Token)를 생성하여 클라이언트에게 반환합니다. 이 JWT를 획득한 클라이언트는 비로소 레지스트리에 이미지를 푸시하거나 풀할 수 있는 권한을 얻게 됩니다.

이 모든 통신 과정의 보안은 TLS(Transport Layer Security) 암호화에 의해 보장됩니다. 특히 GitLab 애플리케이션과 레지스트리 서비스 간의 통신은 GitLab이 개인 키로 JWT에 서명하고, 레지스트리가 공개 키로 서명을 검증하는 방식으로 안전하게 이루어집니다.10 이 키 쌍은 기본적으로 각 설치 환경마다 자동으로 생성되어 구성됩니다.


GitLab 컨테이너 레지스트리는 이미지 데이터를 저장하기 위한 백엔드로 다양한 스토리지 옵션을 지원하며, 이는 운영 규모와 환경에 따라 유연하게 선택할 수 있습니다.

- **로컬 파일시스템:** 가장 기본적인 방식으로, 별도의 외부 서비스 없이 GitLab 서버의 로컬 디스크에 이미지를 저장합니다. Omnibus 패키지로 설치한 경우, 기본 저장 경로는 `/var/opt/gitlab/gitlab-rails/shared/registry`입니다.4 이 방식은 설정이 간단하여 소규모 테스트나 개발 환경에 적합하지만, 서버 디스크 용량에 직접적인 제약을 받고, 백업 및 확장이 복잡하다는 단점이 있습니다.

- **클라우드 객체 스토리지:** 대규모 프로덕션 환경에서는 Amazon S3, Google Cloud Storage (GCS), Microsoft Azure Blob Storage와 같은 클라우드 객체 스토리지를 백엔드로 사용하는 것이 강력히 권장됩니다.4 객체 스토리지는 사실상 무한한 확장성, 뛰어난 데이터 내구성, 그리고 간편한 관리 기능을 제공하여 레지스트리 운영의 복잡성을 크게 낮춰줍니다.10 스토리지 백엔드 설정은 `gitlab.rb`(Omnibus) 또는 `gitlab.yml`(소스 설치) 파일에서 원하는 스토리지 드라이버(예: `s3`, `gcs`, `azure`)와 접근에 필요한 자격 증명(access key, secret key 등)을 지정하여 구성할 수 있습니다.10


기존 레거시 레지스트리는 특히 대규모 환경에서 심각한 성능 문제와 운영상의 비효율성을 드러냈습니다. 가장 큰 문제점은 가비지 컬렉션(Garbage Collection, GC) 과정이 매우 느리고, 실행 중 서비스 중단이 필요하다는 점이었습니다.17 이러한 문제를 근본적으로 해결하기 위해 GitLab은 아키텍처를 재설계한 

**차세대 컨테이너 레지스트리**를 선보였습니다.

- 핵심 개선점: 메타데이터 데이터베이스 도입

  차세대 레지스트리의 가장 큰 구조적 변화는 메타데이터 데이터베이스의 도입입니다.10 기존 방식에서는 이미지 레이어, 태그, 매니페스트와 같은 모든 메타데이터 정보가 파일시스템 구조 자체에 의존했습니다. 이로 인해 특정 태그를 찾거나 참조되지 않는 레이어를 식별하기 위해서는 전체 파일시스템을 탐색해야 했고, 이는 데이터 양이 증가할수록 기하급수적으로 느려지는 원인이었습니다.18 차세대 아키텍처에서는 이러한 메타데이터를 별도의 PostgreSQL 데이터베이스에 저장하고 관리합니다. 따라서 정보가 필요할 때 파일시스템을 스캔하는 대신 데이터베이스에 효율적인 쿼리를 실행할 수 있게 되어, 태그 목록 조회, 스토리지 사용량 계산, 가비지 컬렉션 대상 식별과 같은 작업의 성능이 극적으로 향상되었습니다.10

- 비즈니스 가치: 온라인(Zero-Downtime) 가비지 컬렉션

  메타데이터 데이터베이스 도입이 가져온 가장 실질적인 비즈니스 가치는 온라인 가비지 컬렉션의 지원입니다.10 레거시 레지스트리에서는 사용하지 않는 이미지 레이어를 삭제하여 디스크 공간을 확보하기 위해, 관리자가 레지스트리 서비스를 읽기 전용 모드로 전환하거나 일시적으로 중단해야만 했습니다.15 이는 24/7 운영이 필수적인 프로덕션 환경에서는 상당한 부담이었습니다. 차세대 레지스트리는 메타데이터를 통해 참조 관계를 신속하게 파악할 수 있으므로, 서비스 중단 없이 백그라운드에서 안전하게 불필요한 데이터를 정리할 수 있습니다. 이를 통해 조직은 스토리지 비용을 지속적으로 최적화하고, 중요한 유지보수 작업으로 인한 서비스 중단을 피할 수 있습니다.20

차세대 컨테이너 레지스트리는 GitLab.com에서 먼저 성공적으로 적용되었으며, 셀프 호스팅 사용자를 위해 GitLab 17.3 버전부터 정식 지원(GA, General Availability)이 시작되었습니다. 기존 레거시 레지스트리는 향후 GitLab 19.0 버전에서 지원이 중단될 예정이므로, 셀프 호스팅 관리자는 마이그레이션을 계획해야 합니다.20


셀프 호스팅 환경에서 GitLab 컨테이너 레지스트리를 구성하고 운영할 때는 몇 가지 핵심 요소를 고려해야 합니다.

- **네트워크 구성:** 레지스트리를 외부에 노출하는 방식은 크게 두 가지입니다. 첫째는 기존 GitLab 도메인에 별도의 포트를 할당하는 방식(예: `gitlab.example.com:5050`)이고, 둘째는 레지스트리 전용 도메인(예: `registry.example.com`)을 사용하는 것입니다.4 포트를 사용하는 방식은 기존 GitLab의 TLS 인증서를 재사용할 수 있어 설정이 비교적 간단하다는 장점이 있습니다.10
- **TLS 인증서:** 컨테이너 레지스트리는 보안 통신을 위해 기본적으로 HTTPS를 요구합니다.10 이는 민감한 이미지 데이터와 인증 토큰을 보호하기 위한 필수적인 조치입니다. GitLab에 내장된 Let's Encrypt 통합 기능을 사용하면 TLS 인증서가 자동으로 발급 및 갱신될 수 있지만, 그렇지 않은 환경에서는 관리자가 직접 유효한 TLS 인증서를 발급받아 구성해야 합니다.10
- **대규모 환경 참조 아키텍처:** 사용자가 수천 명에 달하는 대규모 환경을 위해서는 단일 서버 구성으로는 한계가 있습니다. GitLab은 사용자 수(예: 2,000명, 3,000명)에 따른 **참조 아키텍처(Reference Architectures)**를 공식 문서로 제공합니다.14 이 아키텍처에는 HAProxy를 이용한 로드 밸런싱, 고가용성(HA)을 위한 외부 PostgreSQL 및 Redis 클러스터 구성, 그리고 Git 데이터 관리를 위한 Gitaly 클러스터 설정 등, 대규모 트래픽을 안정적으로 처리하기 위한 구체적이고 검증된 구성 가이드가 포함되어 있습니다.

이처럼 GitLab 컨테이너 레지스트리의 아키텍처는 단순한 파일 저장소에서 시작하여, 성능과 확장성 문제를 해결하기 위해 메타데이터 데이터베이스를 도입하는 방향으로 진화해왔습니다. 이 진화는 단순한 기술 개선을 넘어, 셀프 호스팅 환경의 운영 패러다임에 중요한 변화를 가져옵니다. 차세대 레지스트리는 사용자에게 온라인 가비지 컬렉션과 같은 강력한 기능을 제공하는 대신, 관리자에게는 레지스트리 전용 데이터베이스라는 새로운 구성 요소를 프로비저닝하고, 백업하며, 안정적으로 운영해야 하는 추가적인 책임을 부여합니다.20 이는 '단순성'을 일부 희생하여 '성능과 확장성'을 얻는 전략적 트레이드오프로 볼 수 있습니다. 따라서 기술 리더는 이러한 변화를 명확히 인지하고, 차세대 레지스트리 도입 시 데이터베이스 관리 역량 확보 및 체계적인 마이그레이션 계획 수립에 대한 투자를 반드시 고려해야 합니다


GitLab 컨테이너 레지스트리의 가장 강력한 특징 중 하나는 GitLab CI/CD와의 완벽하고 매끄러운 통합입니다. 이 통합을 통해 개발팀은 소스 코드 변경부터 컨테이너 이미지의 빌드, 테스트, 저장, 배포에 이르는 전체 워크플로우를 단일 플랫폼 내에서 완벽하게 자동화할 수 있습니다.


모든 자동화의 중심에는 프로젝트 루트 디렉토리에 위치한 `.gitlab-ci.yml` 파일이 있습니다. 이 YAML 파일을 통해 개발자는 파이프라인의 각 단계(stage)와 작업(job)을 정의하고, 컨테이너 레지스트리와 상호작용하는 모든 명령을 스크립트로 작성할 수 있습니다.4 GitLab은 이 파일을 해석하여 파이프라인을 실행하며, 이를 통해 이미지 빌드, 푸시, 테스트, 배포의 전 과정을 자동화합니다.


GitLab은 CI/CD 파이프라인 내에서 컨테이너 레지스트리 관련 작업을 매우 편리하게 수행할 수 있도록 다양한 **사전 정의된 변수(Predefined Variables)**를 제공합니다.7 이 변수들은 파이프라인 실행 시점에 GitLab에 의해 동적으로 값이 할당되므로, 스크립트의 재사용성과 이식성을 높여줍니다.

- `$CI_REGISTRY`: 현재 GitLab 인스턴스의 컨테이너 레지스트리 주소를 나타냅니다. (예: `registry.gitlab.com`)
- `$CI_REGISTRY_IMAGE`: 현재 파이프라인이 실행되는 프로젝트에 대한 고유한 이미지 기본 경로를 제공합니다. (예: `registry.gitlab.com/my-group/my-project`)
- `$CI_REGISTRY_USER` 및 `$CI_REGISTRY_PASSWORD`: 각 CI/CD 작업(job)마다 동적으로 생성되는 임시 사용자 이름과 비밀번호입니다. 이 자격 증명은 해당 작업이 실행되는 동안에만 유효하며, 레지스트리에 대한 읽기/쓰기 권한을 가집니다. `docker login` 명령에 안전하게 사용될 수 있습니다.
- `$CI_JOB_TOKEN`: 작업(job)별로 생성되는 단기 인증 토큰입니다. 주로 다른 프로젝트의 리소스를 읽어올 때 사용되며, 컨테이너 레지스트리에 대해서는 기본적으로 읽기(`read_registry`) 권한만 가집니다. 이미지를 푸시(push)하기 위해서는 `$CI_REGISTRY_PASSWORD`를 사용해야 합니다.22
- `$CI_COMMIT_REF_SLUG`: 현재 브랜치나 태그의 이름을 URL에 사용하기 안전한 형태로 변환한 값입니다. 예를 들어, `feature/new-login`이라는 브랜치 이름은 슬래시(`/`)를 포함하고 있어 Docker 태그로 사용할 수 없지만, `$CI_COMMIT_REF_SLUG`는 이를 `feature-new-login`과 같이 하이픈으로 변환해줍니다. 따라서 이미지 태그를 동적으로 생성할 때 매우 유용합니다.7


이러한 변수들을 활용하여 다양한 고급 워크플로우를 구성할 수 있습니다.

- **Docker-in-Docker(DinD) 활용:** CI/CD 작업 환경 내에서 `docker build`, `docker run`과 같은 Docker 명령을 직접 실행하기 위해서는 **Docker-in-Docker (DinD)** 기법이 널리 사용됩니다. 이는 `services` 키워드를 통해 `docker:dind` 이미지를 서비스 컨테이너로 실행하는 방식입니다. 이때, CI 작업이 서비스 컨테이너와 통신할 수 있도록 `alias`를 `docker`로 설정하는 것이 매우 중요합니다. 이를 생략하면 `docker: no such host`와 같은 오류가 발생할 수 있습니다.7

  다음은 DinD를 사용하여 이미지를 빌드하고 레지스트리에 푸시하는 기본적인 `.gitlab-ci.yml` 예시입니다.

  ```YAML
  build_image:
    stage: build
    image: docker:24.0.5
    services:
      - name: docker:24.0.5-dind
        alias: docker  # 'docker'라는 호스트 이름으로 dind 서비스에 접근 가능하게 함
    variables:
      # CI_COMMIT_REF_SLUG를 사용하여 동적이고 고유한 이미지 태그 생성
      IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG
    before_script:
      # CI/CD 변수를 사용하여 안전하게 레지스트리에 로그인
      - echo "$CI_REGISTRY_PASSWORD" | docker login -u "$CI_REGISTRY_USER" --password-stdin $CI_REGISTRY
    script:
      - docker build -t $IMAGE_TAG.
      - docker push $IMAGE_TAG
  ```
  
- **Dependency Proxy 활용:** Docker Hub와 같은 공용 레지스트리에서 자주 사용되는 베이스 이미지를 프록시처럼 캐싱하는 기능입니다. 이를 통해 외부 네트워크 의존도를 줄이고, Docker Hub의 이미지 pull 속도 제한(rate limit)을 피하며, 파이프라인 실행 속도를 향상시킬 수 있습니다.6

  `.gitlab-ci.yml`에서 이미지 경로를 지정할 때 `${CI_DEPENDENCY_PROXY_GROUP_IMAGE_PREFIX}` 변수를 사용하면 이 기능을 활성화할 수 있습니다.7

- **다단계 파이프라인(Multi-stage Pipeline):** 일관성 있고 신뢰할 수 있는 배포를 위해 다단계 파이프라인을 구성하는 것이 일반적입니다.

  1. **build 단계:** 소스 코드를 컴파일하고 모든 종속성을 포함한 Docker 이미지를 빌드한 후, 고유한 태그(예: commit SHA)를 붙여 컨테이너 레지스트리에 푸시합니다.
  2. **test 단계:** `build` 단계에서 생성된 바로 그 이미지를 레지스트리에서 풀(pull)하여 단위 테스트, 통합 테스트 등을 실행합니다.
  3. **deploy 단계:** 모든 테스트를 통과한 동일한 이미지를 스테이징 또는 프로덕션 환경에 배포합니다.

  이러한 방식은 각 단계가 정확히 동일한 아티팩트를 기반으로 실행되도록 보장하여, "내 로컬 환경에서는 잘 됐는데..."와 같은 환경 불일치 문제를 원천적으로 차단합니다.7

CI/CD 파이프라인의 각 작업(job)은 기본적으로 격리된 환경에서 실행되어, 한 작업의 결과물이 다른 작업으로 직접 전달되지 않습니다. GitLab은 이 문제를 해결하기 위해 `artifacts`나 `cache`와 같은 메커니즘을 제공하지만, 컨테이너 기반의 현대적인 워크플로우에서는 컨테이너 레지스트리가 훨씬 더 근본적이고 강력한 해결책을 제시합니다. `build` 작업에서 소스 코드와 모든 종속성을 포함하여 생성된 Docker 이미지는 해당 시점의 '완벽한 실행 가능 스냅샷'이 됩니다.7 이후의 모든 

`test`, `security_scan`, `deploy` 작업들은 레지스트리에서 이 특정 이미지를 가져와 사용함으로써, 모든 과정이 완벽하게 동일한 환경에서 실행됨을 보장받습니다. 이처럼 GitLab 컨테이너 레지스트리는 단순한 아티팩트 저장소의 역할을 넘어, 파이프라인의 각 단계를 논리적으로 연결하고 상태(state)를 전달하는 **신뢰할 수 있는 상태 저장소(state repository)**로 기능합니다. 이는 전체 CI/CD 프로세스의 일관성, 재현성, 그리고 신뢰성을 보장하는 핵심적인 메커니즘입니다.


GitLab 컨테이너 레지스트리의 보안과 접근 제어는 GitLab 플랫폼의 통합된 인증 및 권한 모델에 깊이 뿌리내리고 있습니다. 다양한 인증 토큰과 역할 기반 접근 제어(RBAC)를 통해, 조직은 누가, 무엇을, 어떻게 레지스트리 자원에 접근할 수 있는지를 세밀하게 통제할 수 있습니다.


GitLab은 다양한 사용 시나리오에 맞춰 여러 종류의 토큰 기반 인증 방법을 제공합니다. 특히, 2단계 인증(2FA)이 활성화된 계정은 보안상의 이유로 일반 비밀번호를 사용한 `docker login`이 불가능하며, 반드시 토큰 기반 인증을 사용해야 합니다.22

- **개인 액세스 토큰 (Personal Access Token, PAT):**

  - **특징:** 특정 사용자 계정에 직접 귀속되는 토큰으로, 해당 사용자가 가진 모든 프로젝트와 그룹에 대한 권한을 잠재적으로 상속받을 수 있습니다. API 호출, Git over HTTPS, 컨테이너 레지스트리 인증 등 광범위한 용도로 사용됩니다.25
  - **사용 시나리오:** 주로 개발자가 자신의 로컬 환경에서 레지스트리에 접근하거나, 외부 스크립트에서 사용자 권한으로 작업을 수행해야 할 때 사용됩니다.
  - **보안 고려사항:** 강력한 만큼 유출 시 위험이 매우 큽니다. 따라서 CI/CD 자동화와 같이 시스템적으로 사용되는 경우에는 PAT 사용을 최소화하고, 불가피하게 사용하더라도 반드시 필요한 최소한의 권한(scope)만 부여하고 짧은 만료일을 설정하는 것이 보안 모범 사례입니다.26

- **배포 토큰 (Deploy Token):**

  - **특징:** 특정 프로젝트나 그룹 단위로 생성되며, 사용자 계정과 독립적입니다. 주로 읽기(`read_registry`, `read_package`) 또는 쓰기(`write_registry`, `write_package`) 권한을 부여하는 데 특화되어 있습니다.22
  - **사용 시나리오:** Kubernetes 클러스터나 외부 CI/CD 시스템이 GitLab 레지스트리에서 이미지를 풀(pull)하거나, 외부 시스템이 패키지를 업로드하는 등 자동화된 시스템 간의 인증에 이상적입니다.
  - **보안 고려사항:** 사용자 계정과 분리되어 있어 특정 시스템의 접근 권한만 취소하기 용이하며, PAT보다 관리 및 보안 측면에서 더 안전한 선택입니다.

- **프로젝트/그룹 액세스 토큰 (Project/Group Access Token):**

  - **특징:** PAT와 기능적으로 유사하지만, 그 권한이 생성된 특정 프로젝트나 그룹으로 엄격하게 제한됩니다. GitLab Ultimate 등급에서는 '봇(bot)' 사용자를 위해 이 토큰을 생성할 수 있어, 특정 자동화 작업을 위한 서비스 계정처럼 활용할 수 있습니다.22
  - **사용 시나리오:** 특정 프로젝트나 그룹 내에서만 작동하는 자동화 스크립트나 애플리케이션의 인증에 적합합니다.
  - **보안 고려사항:** 권한 범위가 명확하게 한정되어 있어 PAT보다 안전합니다.

- **CI/CD 작업 토큰 (Job Token, `CI_JOB_TOKEN`):**

  - **특징:** GitLab CI/CD 파이프라인의 각 작업(job)이 실행될 때마다 동적으로 생성되고 작업 종료 시 만료되는 매우 짧은 수명의 토큰입니다.22

  - **사용 시나리오:** 파이프라인 내에서 현재 프로젝트의 레지스트리나 다른 리소스에 접근할 때 가장 기본적으로 사용되는 인증 방식입니다.

  - **보안 고려사항:** 수명이 매우 짧고 권한 범위가 작업이 실행되는 프로젝트로 제한되어 있어 가장 안전한 옵션입니다. 기본적으로 레지스트리에 대한 읽기(`read_registry`) 권한만 가지므로, 이미지를 푸시(push)할 때는 쓰기 권한이 있는 `$CI_REGISTRY_PASSWORD` 변수를 사용해야 합니다.22 다른 프로젝트의 레지스트리에 접근해야 하는 경우, 대상 프로젝트의 

    `Settings > CI/CD > Token Access`에서 명시적으로 접근을 허용해 주어야 합니다.30


어떤 토큰을 사용하든, '최소 권한의 원칙'에 따라 필요한 최소한의 범위(scope)만 부여하는 것이 중요합니다.

- **이미지 풀 (읽기 접근):** `read_registry` 권한이 반드시 필요합니다.22
- **이미지 푸시 (쓰기 접근):** `write_registry` 권한이 필요하며, 이 권한은 `read_registry` 권한을 포함합니다.22


레지스트리에 대한 접근 권한은 GitLab 프로젝트 멤버에게 부여된 역할(Role)과 긴밀하게 연동됩니다. 각 역할(Guest, Reporter, Developer, Maintainer, Owner)에 따라 레지스트리에 대한 권한이 차등적으로 부여됩니다.24

여기에 더해, 프로젝트 관리자는 **레지스트리 가시성(Visibility)** 설정을 통해 접근 제어를 한 단계 더 강화할 수 있습니다. 이 설정은 프로젝트의 `Settings > General > Visibility, project features, permissions`에서 찾을 수 있습니다.24

- **`Everyone With Access` (기본값):** 프로젝트에 접근 권한이 있는 모든 사용자가 컨테이너 레지스트리를 보고 이미지를 풀(pull)할 수 있습니다. 예를 들어, 프로젝트가 'Public'이면 인터넷상의 모든 익명 사용자도 이미지를 풀할 수 있습니다.
- **`Only Project Members`:** 'Reporter' 역할 이상을 가진 프로젝트 멤버만이 레지스트리를 보거나 이미지를 풀할 수 있습니다. Public 프로젝트라도 이 설정을 사용하면 사실상 Private 레지스트리처럼 동작하게 됩니다.

이 가시성 설정은 주로 **읽기(view, pull) 권한**에 영향을 미치며, 이미지 푸시나 삭제와 같은 쓰기 권한은 사용자의 역할(Developer 이상)에 따라 결정됩니다.24

다양한 인증 토큰 옵션은 개발자와 관리자에게 유연성을 제공하지만, 동시에 어떤 상황에서 어떤 토큰을 사용해야 할지에 대한 혼란을 야기할 수 있습니다. 아래 표는 각 토큰의 핵심적인 차이점을 명확히 하여, 보안과 편의성 사이에서 최적의 결정을 내리는 데 도움을 줄 수 있도록 구성되었습니다.

| **토큰 유형**                        | **주요 용도**                                          | **권한 범위**                  | **수명**                                 | **보안 고려사항 및 권장 시나리오**                           |
| ------------------------------------ | ------------------------------------------------------ | ------------------------------ | ---------------------------------------- | ------------------------------------------------------------ |
| **개인 액세스 토큰 (PAT)**           | 사용자 대리 인증, 로컬 개발 환경, 개인 스크립트        | 사용자 계정 전체               | 사용자가 지정 (최대 365일, 장기 가능) 25 | **고위험.** 유출 시 피해 범위가 넓음. CI/CD 자동화에는 사용을 지양하고, 불가피할 경우 최소 권한과 짧은 만료일 설정 필수. |
| **배포 토큰 (Deploy Token)**         | 외부 시스템 연동 (예: Kubernetes 클러스터의 이미지 풀) | 그룹 또는 프로젝트 한정        | 만료일 없이 영구적으로 사용 가능         | **중간 위험.** 사용자 계정과 분리되어 관리가 용이. 시스템 간의 자동화된 읽기/쓰기 작업에 권장됨. |
| **프로젝트/그룹 액세스 토큰**        | 봇(Bot) 계정, 특정 프로젝트/그룹 내 자동화             | 프로젝트 또는 그룹 한정        | 사용자가 지정                            | **낮은 위험.** 권한 범위가 명확히 제한되어 PAT보다 안전. 특정 범위 내에서만 작동하는 자동화에 이상적. |
| **CI/CD 작업 토큰 (`CI_JOB_TOKEN`)** | 파이프라인 내에서 리소스 접근                          | 작업(Job)이 속한 프로젝트 한정 | 작업 실행 기간 동안만 유효 (단기)        | **최저 위험.** 가장 안전한 옵션. 파이프라인 내에서 현재 프로젝트의 리소스를 읽는 데 기본적으로 사용. |


컨테이너 기반의 개발 워크플로우가 활발해질수록, 생성되는 이미지의 수와 버전은 기하급수적으로 증가합니다. 효과적인 버전 관리 전략과 체계적인 정리 정책이 없다면, 컨테이너 레지스트리는 금세 관리 불가능한 상태에 빠지고 막대한 스토리지 비용을 유발할 수 있습니다.


컨테이너 이미지의 태그는 가변적(mutable)이라는 특성을 가집니다. 즉, `my-app:latest`라는 동일한 태그가 어제와 오늘 서로 다른 이미지 버전을 가리킬 수 있습니다. `latest`와 같은 롤링 태그는 개발 환경에서 최신 버전을 쉽게 가져올 수 있어 편리하지만, 특정 버전의 배포 일관성과 재현성이 중요한 프로덕션 환경에서는 예측 불가능성을 야기하여 심각한 문제를 일으킬 수 있습니다.31 따라서, 목적에 맞는 명확하고 일관된 태깅 전략을 수립하는 것이 매우 중요합니다.

- **Git Commit SHA:**

  - **방법:** 각 이미지를 빌드할 때, 해당 소스 코드의 Git 커밋 해시(hash)를 태그로 사용합니다. GitLab CI/CD에서는 `$CI_COMMIT_SHA` 또는 짧은 버전인 `$CI_COMMIT_SHORT_SHA` 변수를 활용할 수 있습니다.32
  - **장점:** 모든 이미지 빌드가 소스 코드의 특정 시점과 1:1로 완벽하게 연결되므로, 최고의 추적성과 불변성을 보장합니다. 어떤 이미지로부터 문제가 발생했는지 즉시 역추적할 수 있습니다.
  - **단점:** `a1b2c3d4`와 같은 해시값은 사람이 직관적으로 이해하기 어렵습니다.

- **시맨틱 버저닝 (Semantic Versioning, SemVer):**

  - **방법:** `MAJOR.MINOR.PATCH` (예: `v1.2.3`) 형식의 버전 번호를 태그로 사용합니다. 이는 일반적으로 소스 코드 저장소의 Git 태그와 연동하여 자동화됩니다.31
  - **장점:** 사람이 버전을 쉽게 이해할 수 있으며, 버전 번호의 규칙을 통해 하위 호환성 여부를 명확하게 전달할 수 있습니다. 프로덕션 릴리스나 공식 배포 버전을 관리하는 데 매우 적합합니다.
  - **단점:** 버전 번호를 관리하고 증가시키는 별도의 프로세스가 필요합니다.

- 하이브리드 전략 (권장):

  실제 개발 환경에서는 위 두 가지 전략을 조합한 하이브리드 방식이 가장 효과적입니다. 브랜치와 빌드 목적에 따라 다른 태깅 규칙을 적용하는 것입니다.

  - **개발/피처 브랜치:** 개발 과정에서 수시로 생성되는 임시 빌드에는 브랜치 이름과 커밋 해시를 조합한 태그를 사용합니다. (예: `feature-login-a1b2c3d4`)
  - **메인/개발 브랜치:** 주요 개발 브랜치에 통합될 때마다 생성되는 이미지는 `dev-latest` 또는 `dev-` 접두사와 타임스탬프/커밋 해시를 사용합니다.
  - **릴리스/프로덕션:** 실제 배포를 위한 릴리스에는 Git 태그를 기반으로 한 명확한 시맨틱 버저닝 태그(예: `v1.2.3`)를 부여합니다. 추가로, 이 버전에 `stable`이나 `latest` 태그를 함께 붙여 최신 안정 버전을 쉽게 식별할 수 있도록 할 수 있습니다.31


자동화된 CI/CD 파이프라인은 하루에도 수십, 수백 개의 임시 이미지 태그를 생성할 수 있습니다. 이러한 태그들이 정리되지 않고 방치되면 레지스트리 스토리지 용량을 빠르게 소모하고, UI 성능 저하를 유발합니다.18 GitLab은 이를 자동화된 방식으로 관리하기 위해 

**정리 정책(Cleanup Policy)** 기능을 제공합니다.

- **정책 설정:** 관리자는 프로젝트의 `Settings > Packages & registries` 메뉴에서 UI를 통해 정리 정책을 설정할 수 있습니다. API를 통한 자동화 설정도 가능합니다.15 정책은 다음 규칙들의 조합으로 구성됩니다:
  - `Keep the most recent`: 이미지 이름별로 항상 유지할 최신 태그의 개수를 지정합니다.
  - `Keep tags matching`: 정규식을 사용하여 특정 패턴의 태그(예: `.*-production`)를 영구 보존하도록 설정합니다.
  - `Remove tags older than`: 태그가 생성된 후 특정 기간(예: `30 days`)이 지나면 삭제 대상으로 지정합니다.
  - `Remove tags matching`: 정규식을 사용하여 특정 패턴의 태그(예: `feature-.*`, `.*-rc.*`)를 삭제 대상으로 지정합니다.
- **정책 실행:** 설정된 정책은 GitLab에 의해 주기적으로(예: 매일) 실행되며, 정의된 규칙에 부합하는 태그들을 레지스트리에서 **'제거(untag)'**합니다.36


- **필요성:** 여기서 중요한 점은, 정리 정책이 태그를 제거하더라도 이미지 데이터를 구성하는 기본 레이어(layer)들은 레지스트리 스토리지에 그대로 남아있다는 것입니다. 태그는 단지 이미지 레이어에 대한 포인터일 뿐이기 때문입니다. 이렇게 어떤 태그에도 연결되지 않은 '주인 없는(unreferenced)' 레이어들을 물리적으로 삭제하여 실제 디스크 공간을 확보하는 과정이 바로 **가비지 컬렉션(Garbage Collection, GC)**입니다.10
- **실행 절차:** 가비지 컬렉션은 자동이 아니며, 시스템 관리자가 GitLab 서버에 직접 접근하여 CLI 명령(`gitlab-ctl registry-garbage-collect`)을 실행해야 합니다. `-m` 옵션을 추가하면 참조되지 않는 매니페스트까지 함께 정리하여 더 많은 공간을 확보할 수 있습니다.15 레거시 레지스트리에서는 이 과정 동안 레지스트리를 읽기 전용 모드로 전환해야 하는 등 서비스 영향이 있었지만, 앞서 설명한 차세대 레지스트리에서는 이 과정이 서비스 중단 없이 온라인으로 수행될 수 있습니다.10

효과적인 컨테이너 레지스트리 관리는 단순히 기술적인 도구를 설정하는 것만으로 완성되지 않습니다. 무분별하게 생성되고 방치되는 이미지 태그가 문제의 근원이며 35, GitLab의 '정리 정책'은 이를 해결하기 위한 기술적 수단입니다.35 하지만 이 도구를 효과적으로 사용하기 위해서는 "어떤 태그를 얼마나 오래 보존하고, 어떤 태그를 언제 삭제할 것인가?"라는 근본적인 정책적 질문에 먼저 답해야 합니다. 이 질문에 대한 답은 조직의 브랜치 전략, 릴리스 주기, 감사 및 규정 준수 요구사항(예: 특정 버전의 이미지를 법규에 따라 1년간 보관)에 따라 결정됩니다. 예를 들어, 프로덕션에 배포된 이미지는 영구 보존(

`Keep tags matching: v.*`)하고, 개발 브랜치에서 생성된 임시 이미지는 7일 후에 삭제(`Remove tags older than: 7 days`, `Remove tags matching: dev-.*`)하는 명확한 정책을 수립할 수 있습니다. 이처럼, 기술적 도구인 '정리 정책'은 조직의 '이미지 수명 주기 및 보관 정책'을 시스템적으로 구현하는 수단입니다. 따라서 성공적인 스토리지 관리는 기술팀과 거버넌스팀이 협력하여 명확한 정책을 먼저 정의하고, 그 다음에 이를 GitLab의 기능으로 구현하는 순서로 접근해야 합니다.


GitLab은 'DevSecOps'라는 개념을 플랫폼의 핵심 가치로 삼고 있으며, 컨테이너 레지스트리는 이러한 철학을 구현하는 중요한 무대입니다. 이미지를 저장하는 단계를 넘어, 저장된 이미지의 보안을 자동으로 검증하고 관리하는 강력한 기능을 내장하여 개발 파이프라인 초기에 보안을 통합하는 'Shift Left'를 실현합니다.


GitLab은 **컨테이너 스캐닝(Container Scanning)** 기능을 통해 레지스트리에 저장되거나 CI/CD 파이프라인에서 빌드되는 이미지의 보안 취약점을 자동으로 탐지합니다.8

- 핵심 스캐너: Trivy

  이 기능의 핵심에는 널리 사용되는 오픈소스 취약점 스캐너인 Trivy가 통합되어 있습니다. Trivy는 컨테이너 이미지의 각 레이어를 정적으로 분석하여 알려진 보안 취약점을 식별합니다. 과거에는 Grype 스캐너도 지원되었으나, 현재는 Trivy가 주력 스캐너로 사용되고 있습니다.8

- 스캔 대상 및 범위

  컨테이너 스캐닝은 다양한 유형의 보안 위협을 탐지합니다.

  - **운영 체제 패키지 취약점:** 이미지의 기반이 되는 OS(예: Debian, Ubuntu, Alpine)에 포함된 시스템 패키지(예: `apt`, `yum`으로 설치된 라이브러리)에서 발견된 알려진 취약점(CVE)을 탐지합니다.8
  - **언어별 종속성 취약점:** 애플리케이션이 사용하는 프로그래밍 언어의 패키지 매니저(예: `npm`, `pip`, `Maven`)를 통해 설치된 라이브러리의 취약점도 스캔할 수 있습니다.8
  - **지원 종료(EOL) OS 탐지:** 더 이상 보안 업데이트가 제공되지 않아 위험에 노출된 지원 종료(End-of-Life) 운영 체제를 기반으로 이미지가 만들어졌는지 탐지하고 경고합니다.8


스캔 결과는 단순히 로그 파일로 남는 것이 아니라, GitLab의 UI와 워크플로우에 완벽하게 통합되어 개발자가 즉각적으로 문제를 인지하고 조치할 수 있도록 돕습니다.

- **보고 및 시각화:**

  - **파이프라인 보안 탭:** CI/CD 파이프라인이 완료되면, 결과 페이지의 `Security` 탭에서 해당 빌드에서 발견된 모든 취약점 목록을 심각도(Critical, High, Medium, Low 등), 취약점 설명, CVSS 점수 등과 함께 상세하게 확인할 수 있습니다.8
  - **병합 요청(Merge Request) 위젯:** 개발자가 새로운 코드를 푸시하여 병합 요청을 생성하면, 파이프라인이 실행되고 스캔 결과가 MR 위젯에 요약되어 표시됩니다. 이를 통해 코드 리뷰어는 새로운 변경 사항이 어떤 보안 취약점을 유발했는지 코드 변경점과 함께 직접 확인할 수 있습니다.
  - **보안 대시보드 (Ultimate 등급):** 프로젝트 또는 그룹 레벨의 보안 대시보드를 통해 여러 프로젝트의 보안 상태를 중앙에서 모니터링하고 관리할 수 있습니다.

- 조치 및 관리:

  개발자는 GitLab UI 내에서 발견된 취약점에 대해 다양한 조치를 취할 수 있습니다.

  - **이슈 생성:** 클릭 한 번으로 특정 취약점에 대한 기밀 이슈(confidential issue)를 생성하여 담당자에게 할당하고 해결 과정을 추적할 수 있습니다.
  - **취약점 해제(Dismiss):** 분석 결과 해당 취약점이 프로젝트에 영향을 미치지 않는 오탐(false positive)이라고 판단되면, 사유를 기재하고 목록에서 해제할 수 있습니다.
  - **자동 교정 및 허용 목록 (Ultimate 등급):** 일부 취약점에 대해서는 GitLab이 자동으로 수정 방안(예: 패키지 버전 업그레이드)을 제안하는 MR을 생성해 주기도 합니다. 또한, 조직의 정책에 따라 특정 취약점을 의도적으로 무시해야 할 경우, 이를 허용 목록(allowlist)에 추가하여 보고서에서 제외할 수 있습니다.8


소프트웨어 공급망 보안의 중요성이 커짐에 따라, 사용 중인 컨테이너 이미지가 변조되지 않았으며 신뢰할 수 있는 출처에서 왔음을 보증하는 것이 필수적입니다. GitLab은 이를 위해 오픈소스 도구인 **Cosign**을 이용한 컨테이너 이미지 서명을 지원합니다.24 개발자는 CI/CD 파이프라인에서 이미지를 빌드한 후 Cosign을 사용하여 디지털 서명을 추가할 수 있습니다. 서명된 이미지는 레지스트리 UI에서 별도의 아이콘으로 표시되며, 사용자는 서명의 유효성과 서명자 정보를 확인할 수 있습니다. 이는 악의적으로 변조된 이미지가 프로덕션 환경에 배포되는 것을 방지하는 강력한 보안 장치입니다.24


GitLab의 내장 기능을 최대한 활용함과 동시에, 다음과 같은 일반적인 컨테이너 보안 모범 사례를 함께 적용하는 것이 중요합니다.

- **최소 기반 이미지 사용:** 공격 표면을 줄이기 위해 `alpine` 리눅스나 `distroless` 이미지와 같이 불필요한 도구나 라이브러리가 포함되지 않은 최소한의 베이스 이미지를 사용합니다.40
- **루트 권한 실행 금지:** 컨테이너 내부의 프로세스는 가능한 한 non-root 사용자로 실행하여, 컨테이너 탈출(escape)과 같은 공격이 발생하더라도 호스트 시스템에 미치는 영향을 최소화합니다.40
- **시크릿 관리:** API 키, 데이터베이스 비밀번호와 같은 민감 정보는 절대로 이미지 내부에 하드코딩하지 않습니다. 대신, GitLab CI/CD의 보호된 변수(protected variables)나 HashiCorp Vault와 같은 외부 시크릿 관리 도구를 사용해야 합니다.40
- **지속적인 스캔과 'Shift Left':** 컨테이너 스캐닝을 CI/CD 파이프라인에 통합하여, 모든 코드 커밋과 빌드 시점에 자동으로 보안 검사를 수행하는 'Shift Left' 전략을 채택하여 개발 초기 단계에서 문제를 발견하고 해결합니다.43

GitLab 컨테이너 레지스트리는 시간이 지남에 따라 단순한 취약점 '탐지 및 보고' 도구를 넘어, 조직의 보안 정책을 시스템적으로 '강제'하는 **정책 실행 지점(Policy Enforcement Point)**으로 진화하고 있습니다. 초기에는 빌드된 이미지를 스캔하여 결과를 보고하는 사후 대응적 기능에 중점을 두었습니다.8 이후 병합 요청 위젯에 결과를 통합하여, 취약점이 있는 코드가 메인 브랜치에 병합되는 것을 사전에 '차단'하는 기능으로 발전했습니다. 이미지 서명 기능 24은 한 걸음 더 나아가, 서명되지 않은 신뢰할 수 없는 이미지의 배포 자체를 원천적으로 막는 강력한 통제 수단입니다. 향후 로드맵에 포함된 '보호된 저장소' 및 '불변 태그' 20와 같은 기능들은 특정 버전의 이미지가 임의로 변경되거나 삭제되는 것을 방지하여 규정 준수와 배포 안정성을 시스템적으로 '강제'하게 될 것입니다. 이러한 발전 방향은 "특정 보안 기준을 충족하지 않는 아티팩트는 파이프라인을 통과하거나 배포될 수 없다"는 DevSecOps의 핵심 원칙인 '정책 기반 자동화(Policy-as-Code)'를 GitLab 플랫폼 내에서 직접 구현하고 있음을 보여줍니다.


GitLab 컨테이너 레지스트리는 수많은 경쟁자들이 존재하는 성숙한 시장에 속해 있습니다. 각 솔루션은 고유한 강점과 특징을 가지고 있으며, 조직의 요구사항, 기술 스택, 운영 모델에 따라 최적의 선택은 달라질 수 있습니다.


- **GitLab Container Registry:**

  - **핵심 차별점:** DevOps 플랫폼과의 **완벽한 통합**.4 소스 코드 관리, CI/CD, 이슈 트래킹, 보안 스캐닝이 모두 하나의 워크플로우 안에서 유기적으로 연결됩니다.
  - **이상적인 사용 사례:** 이미 GitLab을 SCM 및 CI/CD의 중심으로 사용하고 있는 조직. 단일 플랫폼의 일관된 경험과 관리 효율성을 최우선으로 생각하는 팀에 최적입니다.45
  - **한계:** 독립적인 전문 레지스트리 솔루션에 비해 특정 기능(예: 고급 복제, 다양한 아티팩트 지원)의 깊이가 부족할 수 있습니다.

- **Docker Hub:**

  - **핵심 차별점:** 컨테이너 기술의 **사실상 표준이자 최대 규모의 퍼블릭 이미지 라이브러리**.46 수많은 공식 이미지와 커뮤니티 이미지를 제공하며, 

    `docker` CLI와의 완벽한 통합으로 뛰어난 개발자 경험을 자랑합니다.

  - **이상적인 사용 사례:** 오픈소스 프로젝트, 개인 개발자, 또는 개발 초기 단계에서 다양한 베이스 이미지를 쉽게 활용하고자 하는 팀.

  - **한계:** 무료 플랜에서는 비공개(private) 저장소가 1개로 제한되며, 익명 사용자의 이미지 풀(pull) 횟수에 엄격한 제한(rate limit)이 있어 대규모 CI/CD 환경에서는 유료 플랜이 필수적입니다.45

- **Amazon ECR (Elastic Container Registry):**

  - **핵심 차별점:** AWS 클라우드 생태계와의 **완벽한 통합**.48 AWS IAM(Identity and Access Management)을 통한 세분화되고 강력한 접근 제어, Amazon EKS(Kubernetes) 및 ECS(Container Service)와의 원활한 연동이 최대 강점입니다.
  - **이상적인 사용 사례:** 인프라의 대부분을 AWS에서 운영하고 있는 조직. AWS의 보안 및 관리 도구를 활용하여 컨테이너 워크로드를 안정적으로 운영하고자 하는 경우에 최적의 선택입니다.45
  - **한계:** AWS 생태계에 강하게 종속되며, 멀티 클라우드나 온프레미스 환경과의 연동이 상대적으로 복잡할 수 있습니다.

- **Google Artifact Registry:**

  - **핵심 차별점:** 컨테이너 이미지뿐만 아니라 Maven(Java), npm(Node.js), Python(PyPI) 등 **다양한 종류의 아티팩트를 하나의 저장소에서 통합 관리**하는 것을 목표로 합니다.50
  - **이상적인 사용 사례:** 여러 프로그래밍 언어와 프레임워크를 사용하는 복잡한 프로젝트에서 모든 아티팩트를 중앙에서 관리하고자 하는 조직. GKE, Cloud Run 등 GCP 서비스와의 긴밀한 통합이 필요할 때 유리합니다.45
  - **한계:** 기존 GCR(Google Container Registry)에 비해 가격이 비싸다는 평가가 있으며, GCP 생태계 외부에서의 활용성은 제한적입니다.52

- **Harbor:**

  - **핵심 차별점:** **엔터프라이즈급 보안 및 관리 기능에 특화된 오픈소스** 솔루션입니다. 강력한 역할 기반 접근 제어(RBAC), 다중 레지스트리 간 이미지 복제, 고급 취약점 스캐닝(Trivy, Clair 등 연동), 이미지 서명(Notary), 감사 로깅 등 온프레미스(self-hosted) 환경에서 강력한 통제와 규정 준수가 필요할 때 요구되는 대부분의 기능을 제공합니다.53
  - **이상적인 사용 사례:** 보안 및 규정 준수 요구사항이 매우 엄격한 금융, 공공, 의료 분야의 조직. 데이터 주권을 완벽하게 통제해야 하는 온프레미스 또는 에어갭(air-gapped) 환경에 최적입니다.45
  - **한계:** 자체적으로 설치하고 운영해야 하므로 인프라 관리 부담이 따릅니다.

기술 리더가 여러 옵션 중에서 전략적 결정을 내릴 때, 각 솔루션의 특징을 한눈에 비교하는 것은 매우 중요합니다. 아래 표는 주요 레지스트리의 핵심 기능을 구조화하여 각 솔루션의 강점과 약점을 명확히 보여줍니다.

| **기능**                | **GitLab**                | **Docker Hub**              | **Amazon ECR**             | **Google Artifact Registry** | **Harbor**                  |
| ----------------------- | ------------------------- | --------------------------- | -------------------------- | ---------------------------- | --------------------------- |
| **핵심 차별점**         | DevSecOps 플랫폼 통합     | 최대 규모 퍼블릭 라이브러리 | AWS 생태계 통합            | 다중 아티팩트 통합 관리      | 엔터프라이즈급 보안/관리    |
| **CI/CD 통합 수준**     | 최상 (내장)               | 높음 (웹훅)                 | 높음 (AWS CodeBuild 등)    | 높음 (Google Cloud Build 등) | 중간 (웹훅, API)            |
| **지원 아티팩트**       | 컨테이너, 패키지, Helm 등 | 컨테이너                    | 컨테이너                   | 컨테이너, 언어 패키지 등     | 컨테이너, Helm, OCI         |
| **취약점 스캐닝**       | 내장 (Trivy) 8            | 내장 (Docker Scout) 47      | 내장 48                    | 내장 (Artifact Analysis) 50  | 내장 (Trivy, Clair 등) 53   |
| **이미지 서명**         | 지원 (Cosign) 24          | 지원 (Docker Content Trust) | 미지원                     | 미지원                       | 지원 (Notary) 54            |
| **역할 기반 접근 제어** | 프로젝트/그룹 역할 기반   | 조직/팀 역할 기반           | AWS IAM 기반 (매우 세분화) | Google Cloud IAM 기반        | 프로젝트 역할 기반 (세분화) |
| **이미지 복제**         | 미지원 (Geo는 DR 목적)    | 미지원                      | 지원 (리전 간)             | 지원 (리전 간)               | 지원 (다중 레지스트리 간)   |
| **호스팅 모델**         | SaaS / Self-hosted        | SaaS                        | SaaS                       | SaaS                         | Self-hosted                 |

클라우드 서비스 도입에서 비용은 핵심적인 고려사항입니다. 아래 표는 각 서비스의 가격 모델을 표준화된 기준으로 정리하여 총 소유 비용(TCO) 예측에 도움을 줍니다.

| **서비스**                   | **스토리지 요금 (GB/월)**       | **데이터 전송(Out) 요금**       | **무료 티어 제공량**                       | **가격 모델 특징**                                           |
| ---------------------------- | ------------------------------- | ------------------------------- | ------------------------------------------ | ------------------------------------------------------------ |
| **GitLab (SaaS)**            | 플랜에 포함된 용량 초과 시 부과 | 플랜에 포함된 용량 초과 시 부과 | Free 플랜: 5GB 스토리지, 10GB 전송 57      | 사용자 기반 구독료에 기능과 용량이 포함되는 모델.            |
| **Docker Hub**               | 플랜에 따라 다름                | 플랜에 따라 다름                | 1 Private Repo, 100 pulls/hr 58            | 사용자 및 기능 기반의 월간/연간 구독 모델.                   |
| **Amazon ECR**               | $0.10 45                        | $0.09/GB 부터 (구간별 할인)     | Private: 500MB/월(1년), Public: 50GB/월 60 | 사용한 만큼 지불하는(Pay-as-you-go) 종량제.                  |
| **Google Artifact Registry** | $0.10 61                        | $0.08/GB 부터 (대륙 간)         | 0.5GB/월                                   | 사용한 만큼 지불하는 종량제. 리전 간 전송 비용이 복잡함.     |
| **Harbor**                   | 자체 인프라 비용                | 자체 인프라 비용                | 오픈소스 (무료)                            | 소프트웨어 자체는 무료이나, 운영을 위한 인프라 및 관리 비용 발생. |

GitLab 컨테이너 레지스트리의 경쟁 전략은 개별 기능의 '최고(Best-in-Breed)'를 지향하기보다는, GitLab이라는 '통합 제품군(Suite)' 내에서의 **'최적의 경험(Best-in-Suite)'**을 제공하는 데 집중하고 있습니다. 예를 들어, Harbor는 엔터프라이즈 보안 기능에서, ECR은 AWS와의 통합 깊이에서, Artifact Registry는 다양한 아티팩트 지원 측면에서 GitLab보다 더 전문화된 기능을 제공할 수 있습니다.45 만약 조직이 각 영역에서 최고의 기능을 원한다면, 여러 전문 도구를 조합하는 'Best-in-Breed' 전략을 선택할 수 있습니다. 하지만 이 경우, 각 도구를 별도로 구매, 구성, 인증 연동, CI/CD 파이프라인 통합에 상당한 엔지니어링 비용과 운영 복잡성이 발생합니다.

GitLab은 바로 이 지점을 공략합니다. 컨테이너 레지스트리가 SCM, CI/CD, 보안 스캐너와 처음부터 하나의 제품처럼 완벽하게 설계되었기 때문에 4, 사용자는 별도의 통합 작업 없이도 매끄러운 DevSecOps 워크플로우를 즉시 경험할 수 있습니다. 따라서 고객의 선택은 "어떤 레지스트리가 개별적으로 가장 뛰어난가?"라는 질문을 넘어, "통합의 편리함과 단일 플랫폼의 관리 효율성이 개별 전문 도구의 장점을 능가하는가?"라는 전략적 질문에 대한 답에 따라 달라집니다. GitLab은 후자를 선택하는 고객에게 가장 강력하고 차별화된 가치를 제공합니다.


GitLab 컨테이너 레지스트리는 강력한 통합 기능을 제공하지만, 특히 대규모 환경에서 운영될 때 몇 가지 알려진 문제점과 기술적 한계를 드러냈습니다. 이러한 과제들은 GitLab 이슈 트래커와 커뮤니티를 통해 지속적으로 논의되고 개선되어 왔으며, 이를 이해하는 것은 안정적인 레지스트리 운영을 위해 필수적입니다.


- **태그 목록 조회 지연:** 프로젝트 내에 저장된 이미지 태그의 수가 수백, 수천 개로 증가함에 따라 컨테이너 레지스트리 UI 페이지를 로딩하는 데 수 분이 소요되는 심각한 성능 저하 문제가 오랫동안 보고되었습니다. 근본적인 원인은 레거시 아키텍처에서 UI가 각 태그의 상세 메타데이터(생성일, 크기 등)를 얻기 위해 레지스트리 API를 반복적으로 호출했기 때문입니다.18 이 문제는 메타데이터 데이터베이스를 도입한 차세대 레지스트리에서 새로운 API를 사용함으로써 극적으로 개선되었습니다.19
- **대용량 이미지 푸시 속도 저하:** 일부 사용자들은 자신의 로컬 네트워크 대역폭이 충분함에도 불구하고 GitLab.com 레지스트리로 대용량 이미지를 푸시하는 속도가 현저히 느리다는 문제를 제기했습니다. 이 문제는 간헐적으로 발생했으며, 특히 CI/CD 파이프라인에서 Kaniko와 같은 도구를 사용하여 이미지를 빌드하고 푸시할 때 두드러지는 경향이 관찰되었습니다.62
- **느리고 비효율적인 가비지 컬렉션(GC):** 레거시 레지스트리의 가장 큰 골칫거리 중 하나는 비효율적인 GC 프로세스였습니다. 수 테라바이트(TB)급의 데이터를 저장하는 대규모 인스턴스에서는 GC의 '표시(mark)' 단계에만 수 시간이 소요되어, 정해진 유지보수 시간 내에 작업을 완료하는 것이 사실상 불가능했습니다.17 이로 인해 사용하지 않는 레이어가 계속 쌓여 스토리지 비용이 통제 불가능하게 증가하는 악순환이 발생했습니다. 이 문제는 차세대 레지스트리의 온라인 GC 기능 도입으로 해결되었습니다.37


- **대규모 네임스페이스 사용량 계산 문제:** 수만 개 이상의 이미지를 보유한 극소수의 매우 큰 네임스페이스(전체의 약 1%)에서는, 중복 제거된 실제 스토리지 사용량을 실시간으로 정확하게 계산하는 데 심각한 성능 부하가 발생합니다.35 이 문제를 해결하기 위해 GitLab은 이러한 네임스페이스에 한해, 중복 제거를 고려하지 않고 모든 고유 이미지 레이어의 크기를 합산하는 **'추정치(estimated usage)'**를 표시하는 임시방편을 사용합니다. 이 추정치는 실제 사용량보다 클 수 있어 사용자에게 혼란을 줄 수 있으며, 스토리지 용량 제한 정책 적용 시 불이익을 받을 수 있다는 우려가 제기되었습니다.63
- **정리 정책의 한계:**
  - GitLab.com과 같이 공유된 환경에서는 서버 자원의 과도한 사용을 막기 위해 정리 정책의 단일 실행 시간이 제한됩니다. 이로 인해 삭제할 태그가 매우 많을 경우, 한 번의 실행으로 모든 대상 태그가 정리되지 않고 일부가 남을 수 있습니다. 모든 태그가 완전히 정리되려면 정책이 여러 번 반복 실행되어야 합니다.35
  - 정리 정책 실행 중 내부적으로 사용하는 인증 토큰(기본 만료 시간 5분)이 만료되어 작업이 중간에 실패하는 문제도 보고되었습니다. 이에 대한 해결책으로 관리자가 토큰 만료 시간을 늘리거나, 정책의 실행 단위를 더 작게 제한하는 설정이 제공됩니다.35


셀프 호스팅 GitLab 인스턴스에서 가장 큰 운영상의 제약 중 하나는, **컨테이너 레지스트리에 이미지가 하나라도 존재하는 프로젝트는 경로(path)나 이름을 변경하거나 다른 그룹으로 이전할 수 없다**는 점입니다.24 만약 프로젝트 구조 변경이 필요할 경우, 관리자는 다음과 같은 매우 복잡하고 번거로운 수동 작업을 거쳐야 합니다:

1. 프로젝트의 모든 이미지를 로컬 환경으로 `docker pull`하여 백업합니다.
2. UI나 API를 통해 기존 프로젝트의 레지스트리에 있는 모든 이미지를 삭제합니다.
3. 프로젝트의 경로를 변경하거나 다른 그룹으로 이전합니다.
4. 백업해 둔 이미지를 새로운 경로에 맞게 다시 태그(`docker tag`)한 후, 새로운 레지스트리로 다시 `docker push`합니다.64

이 과정은 이미지 수가 많을수록 엄청난 시간과 노력을 요구하며, 사실상 대규모 프로젝트의 유연한 관리를 가로막는 심각한 한계점으로 지적되어 왔습니다.

GitLab의 핵심 철학인 'All-in-One' 통합 플랫폼은 사용자에게 매끄럽고 일관된 경험을 제공한다는 강력한 장점을 가집니다.4 하지만 이러한 긴밀한 통합은 역설적으로 대규모 운영 환경에서 특정 구성 요소가 성능이나 기능적 한계에 부딪혔을 때, 해당 부분만 독립적으로 교체하거나 확장하기 어렵게 만드는 

**구조적 경직성**으로 작용할 수 있습니다. 예를 들어, 사용자가 레거시 레지스트리의 성능 문제에 직면했을 때 17, 만약 레지스트리가 독립적인 서비스였다면 고성능의 Harbor와 같은 외부 솔루션으로 비교적 쉽게 교체할 수 있었을 것입니다. 그러나 GitLab 컨테이너 레지스트리는 CI/CD 변수, 내장 보안 스캐닝, UI 등 플랫폼의 다른 기능들과 너무나도 깊게 결합되어 있어 7, 이를 외부 제품으로 대체하는 순간 GitLab 플랫폼의 핵심 가치인 '완벽한 통합'이 깨지게 됩니다.65 결과적으로 사용자는 성능 문제를 겪으면서도, GitLab 자체의 다음 버전 업데이트나 차세대 레지스트리와 같은 근본적인 아키텍처 개선을 기다려야 하는 상황에 놓일 수 있습니다. 이는 통합 플랫폼이 특정 영역에서 병목 현상을 겪을 때, 전체 시스템의 유연성과 발전 속도의 발목을 잡을 수 있는 잠재적 위험이 있음을 시사합니다.


GitLab 컨테이너 레지스트리는 단순한 이미지 저장소에서 출발하여, GitLab의 DevSecOps 플랫폼 전략의 핵심 구성 요소로 진화해왔습니다. 아키텍처 개선, CI/CD 및 보안 기능과의 심층적인 통합을 통해 강력한 기능을 제공하지만, 동시에 대규모 운영 환경에서의 도전 과제들도 명확히 드러났습니다. 본 결론에서는 향후 로드맵을 분석하고, 이를 바탕으로 기술 리더와 DevOps 엔지니어를 위한 전략적 제언을 제시합니다.


GitLab의 향후 개발 방향은 현재의 한계를 극복하고, 시장의 요구에 부응하여 경쟁력을 강화하는 데 초점이 맞춰져 있습니다.

- **차세대 레지스트리의 완전한 전환 및 확산:** 현재 가장 중요한 과제는 GitLab 17.3부터 GA(정식 지원)가 시작된 차세대 레지스트리를 모든 셀프 호스팅 환경에 안정적으로 보급하는 것입니다. 2025년 11월 18.6 버전에서 셀프 호스팅 롤아웃 지원이 예정되어 있으며, 2026년 5월 19.0 버전에서 레거시 레지스트리 지원을 완전히 중단하는 것을 목표로 하고 있습니다.20 이는 성능과 안정성을 근본적으로 개선하기 위한 최우선 과제입니다.
- **보안 및 규정 준수 기능 강화:** 소프트웨어 공급망 보안의 중요성이 대두됨에 따라, 이와 관련된 기능 강화에 집중하고 있습니다. 대표적으로 **태그 불변성(Tag Immutability)** 기능이 2025년 7월 18.2 버전(Ultimate 등급)에 출시될 예정입니다.66 이 기능은 특정 태그가 덮어쓰이는 것을 방지하여 배포의 안정성과 감사 추적성을 보장합니다. 이와 더불어 보호된 저장소(Protected Repositories), 이미지 서명 및 증명(Attestations) 지원 강화를 통해 신뢰할 수 있는 아티팩트 관리 체계를 구축해 나갈 것입니다.20
- **범용 아티팩트 관리자로의 확장:** GitLab은 컨테이너 이미지를 넘어, Maven, npm, PyPI, NuGet 등 다양한 언어의 패키지 포맷을 통합 관리하는 방향으로 나아가고 있습니다. **가상 레지스트리(Virtual Registry)** 및 **Dependency Proxy** 기능을 지속적으로 확장하여, 여러 업스트림 저장소를 단일 엔드포인트로 관리하는 기능을 제공할 계획입니다.66 이는 JFrog Artifactory나 Sonatype Nexus와 같은 전통적인 범용 아티팩트 관리 도구 시장에서 본격적인 경쟁을 시작하겠다는 의지를 보여줍니다.


- **강점 (Strengths):** DevSecOps 플랫폼과의 완벽한 통합, 소스 코드부터 배포까지의 단일 워크플로우, 뛰어난 CI/CD 연동성 및 일관된 UI/UX.
- **약점 (Weaknesses):** 레거시 버전의 대규모 환경 성능 문제, 복잡한 스토리지 관리 및 가비지 컬렉션, 전문 레지스트리 대비 특정 기능(예: 고급 복제)의 깊이 부족.
- **기회 (Opportunities):** DevSecOps 및 소프트웨어 공급망 보안 시장의 지속적인 성장, GitLab Duo와 같은 AI 기능과의 결합을 통한 새로운 가치 창출, 통합 아티팩트 관리 시장으로의 성공적인 확장 가능성.
- **위협 (Threats):** 각 영역에 특화된 전문 경쟁사(Harbor, 클라우드 제공업체)의 지속적인 기능 강화, 오픈소스 기반 기술의 내재적 복잡성, 대규모 사용자의 운영 부담 증가로 인한 이탈 가능성.


GitLab 컨테이너 레지스트리를 성공적으로 도입하고 운영하기 위해, 조직의 역할에 따라 다음과 같은 전략적 접근을 권장합니다.

- **기술 리더를 위한 제언:**
  1. **통합의 가치 극대화:** 조직이 이미 GitLab 생태계에 깊이 투자하고 있다면, 외부 레지스트리를 도입하기보다 내장 컨테이너 레지스트리를 적극 활용하여 개발 생산성과 관리 효율성이라는 통합의 이점을 극대화하는 것이 현명한 선택입니다.
  2. **선제적 정책 수립:** 기술 도입에 앞서, 조직의 릴리스 주기와 규정 준수 요구사항을 반영한 **명확한 이미지 태깅 및 보관 정책**을 먼저 수립하십시오. 이는 기술적 설정의 기반이 되며, 장기적인 스토리지 비용과 관리 복잡성을 결정하는 가장 중요한 요소입니다.
  3. **차세대 아키텍처 조기 도입 계획:** 셀프 호스팅 환경을 운영 중이라면, 성능과 안정성 확보를 위해 **차세대 레지스트리로의 마이그레이션을 조기에 계획**하고, 이를 위한 데이터베이스 프로비저닝 및 관리 역량을 사전에 확보해야 합니다.
- **DevOps 엔지니어를 위한 제언:**
  1. **파이프라인 자동화 및 최적화:** `$CI_COMMIT_REF_SLUG`와 같은 CI/CD 변수를 적극 활용하여 이미지 태깅을 완벽하게 자동화하고, 다단계 파이프라인(Build -->> Test -->> Deploy)을 구성하여 각 단계의 일관성과 신뢰성을 확보하십시오.
  2. **스토리지 관리 내재화:** 모든 신규 프로젝트 템플릿에 합리적인 **기본 정리 정책을 포함**시키고, 관리자와 협력하여 **정기적인 가비지 컬렉션 실행을 자동화(cron job 등)**하여 스토리지 비용을 선제적으로 통제하십시오.37
  3. **보안을 기본으로 (Security by Default):** 모든 프로젝트의 CI/CD 파이프라인에 **컨테이너 스캐닝을 기본으로 활성화**하고, 병합 요청 승인 규칙과 연동하여 특정 심각도 이상의 취약점이 발견되면 병합을 차단하는 정책을 적용하십시오. 이를 통해 안전하지 않은 이미지가 프로덕션에 배포되는 것을 원천적으로 방지할 수 있습니다.


1. Introduction to Registries - GitLab University, accessed July 8, 2025, https://university.gitlab.com/courses/introduction-to-registries
2. What is a container registry? - IBM, accessed July 8, 2025, https://www.ibm.com/think/topics/container-registry
3. 컨테이너 레지스트리란? - Red Hat, accessed July 8, 2025, https://www.redhat.com/ko/topics/cloud-native-apps/what-is-a-container-registry
4. Enable and Use GitLab Container Registry | Baeldung on Ops, accessed July 8, 2025, https://www.baeldung.com/ops/gitlab-container-registry
5. Amazon Elastic Container Registry (ECR) vs. GitLab Comparison - SourceForge, accessed July 8, 2025, https://sourceforge.net/software/compare/Amazon-Elastic-Container-Registry-ECR-vs-GitLab/
6. Packages and Registries - GitLab Docs, accessed July 8, 2025, https://docs.gitlab.com/user/packages/
7. Build and push container images to the container registry | GitLab ..., accessed July 8, 2025, https://docs.gitlab.com/user/packages/container_registry/build_and_push_images/
8. Container Scanning | GitLab Docs, accessed July 8, 2025, https://docs.gitlab.com/user/application_security/container_scanning/
9. Security scanner integration contribute - GitLab Docs, accessed July 8, 2025, https://docs.gitlab.com/development/integrations/secure/
10. GitLab container registry administration, accessed July 8, 2025, https://docs.gitlab.com/administration/packages/container_registry/
11. Setup GitLab Container Registry - GitLab Managed | by Lal Zada - Medium, accessed July 8, 2025, https://medium.com/@BuildWithLal/setup-gitlab-container-registry-gitlab-managed-bbda71b3aef4
12. GitLab 컨테이너 레지스트리 관리, accessed July 8, 2025, https://gitlab-docs.infograb.net/ee/administration/packages/container_registry.html
13. Container registry / Packages / Administration / Help / GitLab, accessed July 8, 2025, https://microfluidics.utoronto.ca/gitlab/help/administration/packages/container_registry.md
14. Reference architecture: Up to 40 RPS or 2000 users - GitLab Docs, accessed July 8, 2025, https://docs.gitlab.com/administration/reference_architectures/2k_users/
15. How to Garbage Collect the GitLab Container Registry to Free Up Storage - How-To Geek, accessed July 8, 2025, https://www.howtogeek.com/devops/how-to-garbage-collect-the-gitlab-container-registry-to-free-up-storage/
16. Container registry / Packages / Administration / Help / GitLab - ETSI Labs, accessed July 8, 2025, https://labs.etsi.org/rep/help/administration/packages/container_registry.md
17. Improved performance for the GitLab Container Registry garbage collection algorithm for S3, accessed July 8, 2025, https://gitlab.com/gitlab-org/gitlab/-/issues/31071
18. container_registry view performance gets slower with more tags (#17371) / Issue - GitLab, accessed July 8, 2025, https://gitlab.com/gitlab-org/gitlab/-/issues/17371
19. Listing tags for a container registry image is taking ~10 seconds (#207368) / Issue - GitLab, accessed July 8, 2025, https://gitlab.com/gitlab-org/gitlab/-/issues/207368
20. Next-generation GitLab container registry goes GA, accessed July 8, 2025, https://about.gitlab.com/blog/next-generation-gitlab-container-registry-goes-ga/
21. Reference architecture: Up to 60 RPS or 3,000 users | GitLab Docs, accessed July 8, 2025, https://docs.gitlab.com/administration/reference_architectures/3k_users/
22. Authenticate with the container registry | GitLab Docs, accessed July 8, 2025, https://docs.gitlab.com/user/packages/container_registry/authenticate_with_container_registry/
23. Run your CI/CD jobs in Docker containers - GitLab Docs, accessed July 8, 2025, https://docs.gitlab.com/ci/docker/using_docker_images/
24. GitLab container registry, accessed July 8, 2025, https://docs.gitlab.com/user/packages/container_registry/
25. Personal access tokens - GitLab Docs, accessed July 8, 2025, https://docs.gitlab.com/user/profile/personal_access_tokens/
26. GitLab token overview, accessed July 8, 2025, https://docs.gitlab.com/security/tokens/
27. gitlab_container_registry.pptx - BioHPC, accessed July 8, 2025, https://portal.biohpc.swmed.edu/media/filer_public/0f/e8/0fe8fd68-3f55-4d46-8ac9-1dc17cee06f5/gitlab_container_registry.pptx
28. Authenticate with container registry - GitLab - ETSI Labs, accessed July 8, 2025, https://labs.etsi.org/rep/help/user/packages/container_registry/authenticate_with_container_registry.md
29. Authenticate with container registry - GitLab - Genboree, accessed July 8, 2025, https://genboree.org/gitlab/help/user/packages/container_registry/authenticate_with_container_registry.md
30. docker - use image from gitlab Registry in CI - Stack Overflow, accessed July 8, 2025, https://stackoverflow.com/questions/53088837/use-image-from-gitlab-registry-in-ci
31. Container Image Versioning, accessed July 8, 2025, https://container-registry.com/posts/container-image-versioning/
32. github - Docker tagging strategy in CI systems (GitLab) - Stack ..., accessed July 8, 2025, https://stackoverflow.com/questions/70054762/docker-tagging-strategy-in-ci-systems-gitlab
33. Best Practice for Container Image Versioning - DevOps Stack Exchange, accessed July 8, 2025, https://devops.stackexchange.com/questions/14362/best-practice-for-container-image-versioning
34. A little help with CI/CD and Semantic Versioning : r/gitlab - Reddit, accessed July 8, 2025, https://www.reddit.com/r/gitlab/comments/k6hre1/a_little_help_with_cicd_and_semantic_versioning/
35. Reduce container registry storage - GitLab Docs, accessed July 8, 2025, https://docs.gitlab.com/user/packages/container_registry/reduce_container_registry_storage/
36. Cleanup policies - Container registry - GitLab Docs, accessed July 8, 2025, https://docs.gitlab.com/development/packages/cleanup_policies/
37. Cleaning-up 7TB of data from our on-prem GitLab Container Registry - Reddit, accessed July 8, 2025, https://www.reddit.com/r/gitlab/comments/15iowcp/cleaningup_7tb_of_data_from_our_onprem_gitlab/
38. Cleaning-up 7TB of data from our on-prem GitLab Container Registry, accessed July 8, 2025, https://blog.crafteo.io/2023/08/04/efficient-gitlab-container-registry-cleanup/
39. Index / Container scanning / Application security / User / 帮助 / GitLab, accessed July 8, 2025, https://code.kingdee.com/gitlab/help/user/application_security/container_scanning/index.md
40. 7 Best Practices for Container Security - Jit.io, accessed July 8, 2025, https://www.jit.io/resources/devsecops/7-best-practices-for-container-security
41. A beginner's guide to container security - GitLab, accessed July 8, 2025, https://about.gitlab.com/topics/devsecops/beginners-guide-to-container-security/
42. Best Practices for Managing and Securing Container Images : r/docker - Reddit, accessed July 8, 2025, https://www.reddit.com/r/docker/comments/1i2it13/best_practices_for_managing_and_securing/
43. What is GitLab Container Scanning? - SentinelOne, accessed July 8, 2025, https://www.sentinelone.com/cybersecurity-101/cloud-security/gitlab-container-scanning/
44. GitLab Security Best Practices: Safeguarding Your Development Pipeline - VivaOps, accessed July 8, 2025, https://www.vivaops.ai/post/gitlab-security-best-practices
45. Choosing the Right Container Registry | Glasskube, accessed July 8, 2025, https://glasskube.dev/blog/container-image-registry-comparison/
46. Choosing a Container Registry in 2024 - Shipyard.build, accessed July 8, 2025, https://shipyard.build/blog/container-registries/
47. Comparing Docker Hub and GitHub Container Registry - JFrog, accessed July 8, 2025, https://jfrog.com/devops-tools/article/comparing-docker-hub-and-github-container-registry/
48. Amazon ECR vs. Docker Hub vs. GitHub Container Registry | cloudonaut, accessed July 8, 2025, https://cloudonaut.io/amazon-ecr-vs-docker-hub-vs-github-container-registry/
49. Container registry preference : r/AskTechnology - Reddit, accessed July 8, 2025, https://www.reddit.com/r/AskTechnology/comments/zl8zig/container_registry_preference/
50. Transition from Container Registry | Artifact Registry documentation - Google Cloud, accessed July 8, 2025, https://cloud.google.com/artifact-registry/docs/transition/transition-from-gcr
51. Google Cloud: Artifact Registry vs Container Registry - Stack Overflow, accessed July 8, 2025, https://stackoverflow.com/questions/65725870/google-cloud-artifact-registry-vs-container-registry
52. Google Cloud Artifact Registry vs GCR Pricing : r/googlecloud - Reddit, accessed July 8, 2025, https://www.reddit.com/r/googlecloud/comments/murvaa/google_cloud_artifact_registry_vs_gcr_pricing/
53. Compare GitLab vs. Harbor in 2025 - Slashdot, accessed July 8, 2025, https://slashdot.org/software/comparison/GitLab-vs-Harbor/
54. Harbor vs Quay Container Registry - Wallarm, accessed July 8, 2025, https://www.wallarm.com/cloud-native-products-101/harbor-vs-quay-container-registry
55. Harbor Container Registry - the StarlingX Documentation, accessed July 8, 2025, https://docs.starlingx.io/admintasks/kubernetes/harbor-as-system-app-1d1e3ec59823.html
56. Harbor, accessed July 8, 2025, https://goharbor.io/
57. Clearing Gitlab's Package Registry automatically from the CI | by Cédric Dué | Medium, accessed July 8, 2025, https://medium.com/@cedricdue/clearing-gitlabs-package-registry-automatically-from-the-ci-4fab353340c
58. Pricing | Docker, accessed July 8, 2025, https://www.docker.com/pricing/
59. Fully Managed Container Registry – Amazon Elastic Container ..., accessed July 8, 2025, https://aws.amazon.com/ecr/pricing/
60. Getting Started with Amazon Elastic Container Registry - DEV Community, accessed July 8, 2025, https://dev.to/deserie_murembeni/getting-started-with-aws-elastic-container-registry-767
61. Pricing | Artifact Registry | Google Cloud, accessed July 8, 2025, https://cloud.google.com/artifact-registry/pricing
62. Investigate: Gitlab registry push extremely slow with Kaniko (#241996) / Issue, accessed July 8, 2025, https://gitlab.com/gitlab-org/gitlab/-/issues/241996
63. Estimated registry usage for very large namespaces (#9413) / Epic ..., accessed July 8, 2025, https://gitlab.com/groups/gitlab-org/-/epics/9413
64. Troubleshooting the GitLab container registry, accessed July 8, 2025, https://docs.gitlab.com/user/packages/container_registry/troubleshoot_container_registry/
65. Integrate Harbor as a proxy cache for GitLab Container Registry (#421471) / Issue, accessed July 8, 2025, https://gitlab.com/gitlab-org/gitlab/-/issues/421471
66. GitLab upcoming releases | GitLab, accessed July 8, 2025, https://about.gitlab.com/upcoming-releases/
67. GitLab Package roadmap for 2024, accessed July 8, 2025, https://about.gitlab.com/blog/gitlab-package-roadmap-for-2024/