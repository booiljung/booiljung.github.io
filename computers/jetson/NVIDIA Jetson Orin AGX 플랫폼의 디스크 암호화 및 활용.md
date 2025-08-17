# NVIDIA Jetson AGX Orin 플랫폼의 디스크 암호화 및 활용

## 요약 (Excutive Summary)

 본 보고서는 NVIDIA Jetson AGX Orin 플랫폼의 디스크 암호화 기능에 대한 포괄적이고 전문적인 분석을 제공합니다. 자율 머신이 물리적으로 안전하지 않은 환경에서 작동하는 엣지에서는, 저장 데이터(data-at-rest) 보호가 단순한 기능이 아니라 지적 재산 보호, 데이터 프라이버시, 운영 무결성을 위한 기본 요건입니다. Jetson AGX Orin은 Jetson Linux(L4T) 운영 체제를 통해 Linux 표준인 LUKS(Linux Unified Key Setup)에 기반한 강력한 하드웨어 가속 디스크 암호화 솔루션을 제공합니다. 이 구현은 하드웨어 신뢰점(Hardware Root of Trust)과 신뢰 실행 환경(Trusted Execution Environment, TEE)을 포함한 SoC의 하드웨어 보안 기능과 깊이 통합되어 자동화되고 안전하며 "사용자 개입 없는(zero-touch)" 잠금 해제 메커니즘을 제공합니다. 본 분석은 암호화가 측정 가능한 성능 오버헤드를 유발하지만, Orin의 Cortex-A78AE CPU 코어에 내장된 강력한 Armv8 암호화 확장(Cryptography Extensions)이 이 영향을 완화하여, 까다로운 실시간 AI 파이프라인에서도 안전한 배포가 가능함을 입증할 것입니다. 기술적 구현을 탐색하고, 성능 상충 관계를 정량화하며, 자율 주행 차량, 스마트 제조, 헬스케어와 같은 핵심 분야에서 이러한 보안 태세의 전략적 활용(활용) 방안을 상세히 설명합니다. 마지막으로, 진화하는 위협 환경을 조망하며 고급 공격에 대한 완화책과 포스트 퀀텀(Post-Quantum) 암호화 시대에 대한 플랫폼의 준비 상태를 고찰합니다.

## 1.  Jetson Orin AGX의 신뢰 기반 구축

이 파트에서는 Jetson 플랫폼의 기본 보안 아키텍처를 설명합니다. 디스크 암호화는 그것이 실행되는 플랫폼만큼만 안전하므로, "신뢰 체인(Chain of Trust)"을 이해하는 것이 전제 조건입니다.

### 1.1  Jetson Orin 시스템-온-칩(SoC) 아키텍처: 보안 관점

Jetson AGX Orin은 강력한 컴퓨팅 모듈일 뿐만 아니라 보안을 염두에 두고 설계된 정교한 시스템-온-칩(SoC)입니다.1 핵심 구성 요소에는 Armv8.2 아키텍처와 암호화 확장을 특징으로 하는 12코어 Arm Cortex-A78AE CPU 4, 2048개의 CUDA 코어와 64개의 텐서 코어를 갖춘 NVIDIA Ampere 아키텍처 GPU 2, 그리고 이 논의에서 가장 중요한 전용 플랫폼 보안 컨트롤러(Platform Security Controller, PSC) 및 보안 엔진(Security Engine, SE)이 포함됩니다.7 PSC는 보안 기능을 관리하고, 방화벽을 설정하며, 보안 감사를 수행하는 RISC-V 서브시스템입니다.7 SE는 하드웨어 가속 암호화 연산을 처리합니다.10 이러한 이기종 아키텍처는 보안에 중요한 작업을 주 애플리케이션 프로세서에서 오프로드할 수 있게 합니다.

**표 1: NVIDIA Jetson AGX Orin 64GB 모듈의 주요 사양**

| 구성 요소 | 사양                                                    |      |
| --------- | ------------------------------------------------------- | ---- |
| AI 성능   | 275 TOPS (INT8)                                         |      |
| GPU       | NVIDIA Ampere 아키텍처, 2048 CUDA 코어, 64 텐서 코어    |      |
| CPU       | 12코어 Arm Cortex-A78AE (Armv8.2)                       |      |
| 메모리    | 64GB LPDDR5                                             |      |
| 스토리지  | 64GB eMMC 5.1                                           |      |
| 보안      | 플랫폼 보안 컨트롤러(PSC), 보안 엔진(SE), Arm TrustZone |      |

데이터 소스: 1

### 1.2  신뢰 체인 구축: 실리콘에서 소프트웨어까지

신뢰 체인은 전원이 공급되는 순간부터 소프트웨어 스택의 무결성을 보장하는 순차적 검증 프로세스입니다.

#### 1.2.1  하드웨어 신뢰점 및 보안 부팅

신뢰 체인은 변경 불가능한 하드웨어 신뢰점(Hardware Root of Trust, HRoT)에서 시작됩니다. 이는 암호화 키를 SoC의 온칩 퓨즈에 한 번만 프로그래밍 가능하게 기록함으로써 설정됩니다.11 기본 키인 

`OEM_K1`은 256비트 AES 키로, 장치의 궁극적인 신뢰점이 됩니다.13 보안 부팅(Secure Boot) 프로세스는 이 HRoT를 사용하여 초기 부트로더(MB1, MB2)부터 UEFI 펌웨어, 그리고 최종적으로 Linux 커널에 이르기까지 각 후속 부팅 단계의 서명을 암호학적으로 검증합니다.11 만약 어떤 구성 요소의 서명이 유효하지 않으면 부팅 프로세스가 중단되어, 변조되거나 승인되지 않은 코드의 실행을 방지합니다.16 이 프로세스는 운영 체제 커널과 디스크 암호 해제가 이루어질 환경이 신뢰할 수 있고 수정되지 않았음을 보장합니다.

#### 1.2.2  신뢰 실행 환경(TEE): OP-TEE

Arm TrustZone 기술은 프로세서를 두 개의 독립된 환경, 즉 주 OS(Jetson Linux)와 애플리케이션이 실행되는 "Normal World"(또는 Rich Execution Environment - REE)와 신뢰 실행 환경(TEE)을 호스팅하는 "Secure World"로 분할합니다.13 NVIDIA의 L4T는 OP-TEE(Open Portable Trusted Execution Environment)를 TEE OS로 사용합니다.11 TEE는 Normal World 커널보다 높은 권한 수준(secure EL1)에서 실행되며, 민감한 코드(Trusted Applications 또는 TA)를 실행하고 비밀 데이터를 처리하기 위한 보호된 공간을 제공하며, 하드웨어에 의해 주 OS로부터 격리됩니다.19 이 격리는 주 OS 커널이 손상되더라도 디스크 암호화 키 파생과 같은 암호화 작업을 위한 안전한 컨텍스트를 제공하므로 매우 중요합니다.

이러한 계층적 보안 아키텍처의 핵심은 디스크 암호화의 보안이 독립적인 기능이 아니라 세심하게 설계된 결과물이라는 점에 있습니다. "사용자 개입 없는" 자동 잠금 해제 메커니즘은 TEE가 기밀 컴퓨팅 환경을 제공하기 때문에 가능하며, TEE 자체는 보안 부팅이 하드웨어 퓨즈에 기록된 신뢰점을 기준으로 그 무결성을 검증했기 때문에 신뢰할 수 있습니다. 이는 강력한 시너지를 창출합니다. 보안 부팅은 암호 해제 환경의 *무결성*을 보장하고, TEE는 암호 해제 프로세스의 *기밀성*을 보장합니다. 디스크 암호화 키의 보안은 SoC의 물리적 실리콘까지 직접 추적할 수 있습니다. 이 끊어지지 않는 "신뢰 체인"은 전체 보안 모델을 뒷받침하는 핵심 원리이며, 다양한 공격에 취약하고 자율 머신에는 비현실적인 사용자 입력 암호에 의존하는 기존 모델보다 훨씬 강력한 접근 방식입니다.

### 1.3  전체적인 보안 모델

Jetson Orin 플랫폼은 단순한 보안 부팅 및 TEE를 넘어 상호 작용하는 보안 기능 모음을 제공합니다. 여기에는 하드웨어 가속 암호화, 키를 위한 보안 스토리지(예: RPMB), 취약한 소프트웨어 버전으로의 다운그레이드를 방지하는 롤백 보호, 메모리 암호화 등이 포함됩니다.11 이 기능들이 결합될 때, 심층 방어(defense-in-depth) 전략을 형성합니다. 디스크 암호화는 저장 데이터를 보호하고, 보안 부팅은 부팅 체인의 무결성을 보호하며, TEE는 사용 중인 키와 민감한 작업을 보호합니다.

**표 2: Jetson Orin 플랫폼의 주요 보안 기능 개요**

| 기능                    | 설명                                                         | 주요 목표                                  |      |
| ----------------------- | ------------------------------------------------------------ | ------------------------------------------ | ---- |
| 보안 부팅 (Secure Boot) | 하드웨어 신뢰점을 사용하여 부팅 체인의 각 구성 요소를 암호학적으로 검증합니다. | 부팅 시 소프트웨어 무결성 보장             |      |
| OP-TEE                  | Arm TrustZone을 활용하여 주 OS와 격리된 안전한 실행 환경을 제공합니다. | 사용 중인 키 및 민감한 연산의 기밀성 보호  |      |
| 디스크 암호화 (LUKS)    | LUKS 표준을 사용하여 전체 디스크 또는 파티션을 암호화합니다. | 저장 데이터(Data-at-rest)의 기밀성 보호    |      |
| 보안 스토리지 (RPMB)    | 재생 보호 메모리 블록(RPMB)을 사용하여 키와 같은 데이터를 안전하게 저장합니다. | 키 및 중요 데이터의 영구적이고 안전한 저장 |      |
| 롤백 보호               | 퓨즈에 버전 번호를 기록하여 이전 버전의 펌웨어로 다운그레이드하는 것을 방지합니다. | 알려진 취약점을 이용한 공격 방지           |      |
| 하드웨어 가속 암호화    | 전용 보안 엔진(SE)을 사용하여 AES와 같은 암호화 알고리즘을 가속합니다. | 성능 저하 최소화 및 보안 강화              |      |

데이터 소스: 11

## 2.  기술 심층 분석: Jetson Linux의 LUKS 디스크 암호화

이 섹션에서는 기본적인 "왜"에서 기술적인 "어떻게"로 전환하여 암호화 메커니즘을 세분화하여 분석합니다.

### 2.1  핵심 기술: LUKS 및 dm-crypt 소개

NVIDIA Jetson Linux의 디스크 암호화는 업계 표준인 Linux Unified Key Setup(LUKS)을 기반으로 합니다.10 LUKS는 블록 장치 암호화 형식을 위한 사양입니다.24 자체적으로 암호화를 수행하지는 않지만, 암호화된 키 슬롯을 포함한 메타데이터를 저장하는 표준 온디스크 헤더를 제공합니다.24 실제 암호화 작업은 장치 매퍼(device-mapper) 대상인 

`dm-crypt` 커널 모듈에 의해 수행됩니다.28 파티션 포맷이나 잠금 해제와 같은 사용자 공간 관리는 

`cryptsetup` 유틸리티에 의해 처리됩니다.10 핵심 개념은 2계층 키 시스템입니다. 대용량 데이터를 암호화하는 데는 높은 엔트로피의 마스터 키(디스크 암호화 키 또는 DEK라고도 함)가 사용되며, 이 DEK 자체는 사용자 키(암호에서 파생됨)로 암호화되어 LUKS 헤더의 키 슬롯 중 하나에 저장됩니다.21 이를 통해 전체 디스크를 다시 암호화하지 않고도 암호를 변경할 수 있습니다.

### 2.2  NVIDIA의 하드웨어 가속 구현

#### 2.2.1  암호화를 위한 파티션 아키텍처

중요한 구현 세부 사항은 부트로더가 암호화된 파일을 읽을 수 없다는 것입니다. 따라서 디스크 암호화를 활성화하려면 주 애플리케이션 파티션(`APP`)을 두 개의 새로운 파티션으로 분할해야 합니다.10 첫 번째는 암호화되지 않은 `APP` 파티션으로, Linux 커널(`Image`), 장치 트리 블롭(DTB), 초기 RAM 디스크(`initrd`)를 포함하는 `/boot` 디렉토리를 포함합니다. 두 번째는 새로운 더 큰 `APP_ENC` 파티션으로, 나머지 루트 파일 시스템(`/`)을 포함하며 LUKS로 완전히 암호화됩니다.21 이 레이아웃은 보드의 XML 파티션 구성 파일(예: `flash_t234_qspi_sdmmc_enc_rfs.xml`)에 정의되어 있습니다.21

#### 2.2.2  자동 잠금 해제 프로세스

부팅 시, 검증된 커널과 `initrd`가 암호화되지 않은 `APP` 파티션에서 로드됩니다. `initrd`에는 필요한 스크립트와 경량 클라이언트 애플리케이션인 `nvluks-srv-app`이 포함되어 있습니다.10 이 애플리케이션은 OP-TEE Secure World에서 실행되는 `luks-srv` Trusted Application(TA)과 통신하여 잠금 해제 시퀀스를 시작합니다.10 TA는 고유한 암호를 파생시켜(아래에서 자세히 설명) `nvluks-srv-app`에 안전하게 다시 전달합니다. 이 암호는 `initrd` 내의 `cryptsetup` 유틸리티에 공급되어 마스터 키(DEK)를 해독하여 `APP_ENC` 파티션을 잠금 해제합니다.10 그런 다음 `dm-crypt` 커널 모듈에 DEK가 로드되고, 암호 해독된 파티션이 최종 루트 파일 시스템으로 마운트됩니다. 이후의 디스크 I/O는 AES 작업을 위해 보안 엔진(SE) 하드웨어를 활용하는 `dm-crypt`에 의해 투명하게 암호화 및 해독됩니다.10

#### 2.2.3  TEE 내 키 관리: "비밀 소스"

이것이 NVIDIA의 보안 구현의 핵심입니다. 암호는 장치의 어느 곳에도 저장되지 않습니다. 부팅할 때마다 즉석에서 생성됩니다.21 `nvluks-srv-app`은 `luks-srv` TA에 장치별 고유 암호를 요청합니다.22 TA는 차례로 다른 의사 TA인 `jetson_user_key_pta`에 LUKS 키를 요청합니다.22 이 pTA는 NIST-SP-800-108 키 파생 함수(Key Derivation Function, KDF)를 사용하여 암호화된 키 블롭(Encrypted Key Blob, EKB) 파티션(`eks.img`)에 저장된 루트 키에서 키를 파생합니다.13 이 EKB 자체는 하드웨어 퓨즈에 기록된 `OEM_K1` 키에서 파생된 키를 사용하여 암호화되고 인증됩니다.13 KDF는 장치의 고유 칩 ID(ECID)를 컨텍스트 문자열로 사용하여 파생된 LUKS 키를 해당 특정 SoC에 고유하게 만듭니다.22`luks-srv` TA는 이 고유한 LUKS 키를 사용하여 최종 암호를 파생하며, 이때 다시 KDF를 사용하지만 파티션의 UUID를 컨텍스트로 사용하여 암호를 스토리지 매체에 바인딩합니다.22 마지막으로, 그리고 결정적으로, `initrd` 스크립트는 `LUKS_SRV_TA_CMD_SRV_DOWN` 명령을 TA에 보내어 다음 재부팅까지 암호를 다시 발급하지 못하도록 하여, 시스템이 실행된 후 이를 쿼리하려는 공격을 막습니다.21

**표 3: LUKS 암호 생성을 위한 키 파생 함수(KDF) 매개변수**

| 파생 결과물         | KDF 알고리즘    | 입력 키 소스                    | 레이블 문자열 (KDF 매개변수)   | 컨텍스트 문자열 (KDF 매개변수) |      |
| ------------------- | --------------- | ------------------------------- | ------------------------------ | ------------------------------ | ---- |
| 장치별 고유 LUKS 키 | NIST-SP-800-108 | EKB에서 파생된 키 (OEM_K1 기반) | `"luks-srv-ecid"`              | 장치 ECID                      |      |
| LUKS 암호           | NIST-SP-800-108 | 장치별 고유 LUKS 키             | `"luks-srv-passphrase-unique"` | 파티션 UUID                    |      |

데이터 소스: 10

### 2.3  실용적인 구현 가이드

#### 2.3.1  호스트 머신 준비 및 툴체인 설정

`cryptsetup` 및 Python 암호화 라이브러리(`python3-cryptography`, `python3-pycryptodome` 등)를 포함한 특정 패키지가 설치된 Linux 호스트 머신(Ubuntu 22.04 권장)이 필요합니다.21 NVIDIA JetPack SDK 및 L4T 드라이버 패키지를 다운로드하여 압축을 풀어야 합니다.

#### 2.3.2  암호화된 RootFS 이미지 구성 및 생성

이 프로세스는 파티션 레이아웃 XML 파일(예: `flash_t234_qspi_sdmmc_enc_rfs.xml`)을 수정하여 `APP_ENC` 파티션에 대해 `encrypted="true"` 속성을 설정하는 것을 포함합니다.21 암호화 키 파일(`disk_enc.key` 또는 `ekb.key`)을 생성해야 합니다. 이 키는 TEE용 키를 포함하는 `eks.img` 블롭을 생성하는 데 사용된 키와 일치해야 합니다.21

#### 2.3.3  암호화된 이미지 플래싱

`flash.sh` 스크립트가 기본 도구입니다. 핵심 단계는 환경 변수 `ROOTFS_ENC=1` 또는 `sudo ROOTFS_ENC=1`을 설정하는 것입니다. 그런 다음 스크립트는 적절한 보드 구성과 암호화 키 파일의 경로와 함께 호출됩니다. 예를 들면 다음과 같습니다: `sudo ROOTFS_ENC=1./flash.sh -i "./disk_enc.key" jetson-agx-orin-devkit internal`.21 이 스크립트는 암호화된 rootfs 이미지 생성, 수정된 `initrd` 생성 및 필요한 모든 파티션을 장치에 플래싱하는 작업을 자동으로 처리합니다.

이 플래싱 프로세스는 겉보기에는 단일 명령으로 간소화되었지만, 그 이면에는 중요한 종속성이 숨어 있습니다. `flash.sh`에 제공된 키와 `eks.img`에 내장된 키가 일치해야 한다는 점입니다. 불일치는 TEE가 올바른 암호를 파생할 수 없어 부팅 실패로 이어집니다. 호스트에서 `flash.sh`의 헬퍼 스크립트가 LUKS 헤더용 암호를 미리 계산하는 데 사용하는 키는, 장치에서 TEE가 `eks.img`로부터 파생할 키와 동일해야 합니다. 따라서 `eks.img`는 `flash.sh`에 전달되는 동일한 루트 키(예: `sym_key2` 21)를 사용하여 생성되어야 합니다. 이는 맞춤형 보안 장치를 만드는 개발자에게 중요하고 명확하지 않을 수 있는 단계이며, 일반적인 구현 오류를 방지하기 위해 명시적으로 강조되어야 합니다.

## 3.  성능, 분석 및 고급 구성

이 파트에서는 디스크 암호화 활성화의 실제 영향을 정량화하고 대체 보안 태세와 비교합니다.

### 3.1  성능 분석: 암호화의 오버헤드

#### 3.1.1  I/O 처리량 벤치마킹 (NVMe SSD)

스토리지 I/O에 미치는 영향을 평가하기 위해 `fio`(Flexible I/O Tester)를 사용한 벤치마크 결과를 제시합니다. NVMe SSD의 표준 `ext4` 파일 시스템에서 기준선을 설정한 후 35, 동일한 하드웨어의 LUKS 암호화 볼륨에서 동일한 테스트를 실행합니다. 주요 지표는 순차 읽기/쓰기 대역폭(MB/s)과 랜덤 4k 읽기/쓰기 IOPS입니다. 일반적인 Linux `fio` 결과는 때때로 50% 이상의 상당한 성능 저하를 보여주며 38, 이는 예상할 수 있는 성능 저하의 기준점으로 삼을 수 있습니다. 이 분석은 이러한 성능 저하가 대용량 모델 로딩이나 고대역폭 센서 데이터 로깅과 같은 AI 워크로드에 어떤 영향을 미치는지 논의할 것입니다.

**표 4: FIO 벤치마크 결과: 암호화(LUKS) vs. 비암호화 NVMe 스토리지**

| 테스트 케이스    | 블록 크기 | 비암호화 (기준) | LUKS 암호화  | 성능 변화 (%) |      |
| ---------------- | --------- | --------------- | ------------ | ------------- | ---- |
| 순차 읽기 (MB/s) | 1M        | ~3500 MB/s      | ~1500 MB/s   | ~ -57%        |      |
| 순차 쓰기 (MB/s) | 1M        | ~3000 MB/s      | ~1200 MB/s   | ~ -60%        |      |
| 랜덤 읽기 (IOPS) | 4K        | ~70,000 IOPS    | ~30,000 IOPS | ~ -57%        |      |
| 랜덤 쓰기 (IOPS) | 4K        | ~60,000 IOPS    | ~25,000 IOPS | ~ -58%        |      |

참고: 이 값들은 일반적인 NVMe SSD 및 LUKS 성능 저하율 35을 기반으로 한 대표적인 예상치이며, 실제 Jetson Orin 시스템에서는 다를 수 있습니다.

#### 3.1.2  CPU 오버헤드 및 하드웨어 가속

암호화의 주된 오버헤드는 CPU 연산입니다. Jetson Orin AGX에서 `cryptsetup benchmark` 결과를 분석할 것입니다. 이 도구는 다른 암호에 대한 CPU의 원시 암호화 처리량을 측정합니다. Jetson Orin AGX는 Armv8 암호화 확장(CE)을 포함하는 Arm Cortex-A78AE CPU를 사용합니다.39 이 확장은 AES 연산을 위한 전용 명령어를 제공하여 프로세스를 크게 가속합니다. L4T에서 사용되는 AES-XTS-256 암호 22의 벤치마크 처리량을 NVMe 드라이브의 이론적 I/O 대역폭과 비교할 것입니다. 이는 하드웨어 가속 덕분에 CPU의 암호화 성능이 종종 주된 병목 현상이 아니며, 오히려 전체 I/O 하위 시스템 지연 시간이 병목임을 보여줄 것입니다.

**표 5: Cryptsetup 벤치마크: Jetson Orin CPU의 AES-XTS-256 성능**

| 암호            | 키 크기  | 암호화 방향 (처리량) | 복호화 방향 (처리량) |      |
| --------------- | -------- | -------------------- | -------------------- | ---- |
| aes-xts-plain64 | 256 비트 | ~2500 MiB/s          | ~2500 MiB/s          |      |

참고: 이 값들은 Armv8 CE를 갖춘 고성능 ARM 코어의 일반적인 `cryptsetup benchmark` 결과 38를 기반으로 한 예상치입니다.

#### 3.1.3  부팅 시간 영향

암호화를 활성화하면 부팅 프로세스에 단계가 추가됩니다: 수정된 `initrd` 로드, TEE와의 통신, LUKS 볼륨 잠금 해제. 이는 측정 가능한 지연을 유발합니다. 부팅 로그42를 분석하여 이 영향을 정량화할 것입니다. 프로세스는 자동화되어 사용자 입력이 필요 없지만 32, TEE 통신 및 `cryptsetup`이 장치를 잠금 해제하는 데 걸리는 시간은 총 부팅 시간에 몇 초를 추가할 수 있습니다. 엄격한 부팅 시간 요구 사항이 있는 애플리케이션(예: 자동차)의 경우 이는 중요한 설계 고려 사항입니다.41

Jetson Orin에서 디스크 암호화의 성능 오버헤드는 상당하지만 관리 가능하며, 병목 현상은 종종 오해됩니다. 원시 I/O 처리량(MB/s)은 눈에 띄는 비율로 감소하지만, CPU의 하드웨어 AES 가속은 매우 효율적이어서 특히 랜덤 I/O의 경우 스토리지 장치의 성능을 능가할 수 있습니다. 따라서 실제 영향은 워크로드에 따라 크게 달라집니다. AI 모델 로딩(대규모 순차 읽기)의 경우 오버헤드가 존재하지만 수용 가능할 수 있습니다. 고주파 데이터 로깅(많은 작은 쓰기)의 경우 영향이 더 심각할 수 있습니다. 성능 손실은 단순히 "CPU가 암호화로 바쁘다"는 것이 아니라, 전체 I/O 스택(`dm-crypt`, 장치 매퍼, NVMe 드라이버) 내의 추상화 및 데이터 처리 계층 추가로 인한 것입니다. 이는 더 빠른 CPU가 암호화된 I/O 성능을 생각만큼 향상시키지 않을 수 있음을 의미합니다.

### 3.2  기밀성 대 무결성: LUKS와 dm-verity 비교 분석

#### 3.2.1  dm-verity 이해

`dm-verity`는 블록 장치의 투명한 무결성 검사를 제공하는 또 다른 장치 매퍼 대상입니다.43 읽기 전용 파일 시스템 전체에 해시 트리를 생성합니다. 블록을 읽을 때 해시가 계산되고 트리에 대해 검증되어 데이터가 변조되지 않았음을 보장합니다.43 이는 Android의 검증된 부팅(Verified Boot)의 핵심 구성 요소입니다.46 공격자에 의한 루트 파일 시스템의 영구적인 수정을 방지합니다.

#### 3.2.2  설계 상충 관계 및 시너지

근본적인 상충 관계는 **기밀성(LUKS) 대 무결성(dm-verity)**입니다.15 LUKS는 데이터를 암호화하여 키 없이는 읽을 수 없게 만들고 읽기/쓰기 파일 시스템을 지원합니다. 주된 목표는 오프라인 공격(장치 도난 시)으로부터 데이터를 보호하는 것입니다. 

`dm-verity`는 데이터가 신뢰할 수 있고 수정되지 않았음을 보장하지만, 데이터 자체는 평문이며 읽을 수 있습니다. 읽기 전용 파일 시스템용으로 설계되었으며 시스템 소프트웨어를 수정하려는 온라인 공격으로부터 보호합니다. 이 둘은 상호 배타적이지 않습니다. 일반적인 임베디드 시스템 패턴은 루트 파일 시스템(불변이어야 함)에 `dm-verity`를 사용하고, 사용자 데이터, 로그 및 기타 민감한 정보가 저장되는 별도의 암호화된 데이터 파티션에 LUKS를 사용하는 것입니다.15

**표 6: 보안 태세 비교: LUKS vs. dm-verity**

| 기능             | LUKS (dm-crypt)               | dm-verity                               |      |
| ---------------- | ----------------------------- | --------------------------------------- | ---- |
| 주요 목표        | 기밀성 (Confidentiality)      | 무결성 (Integrity)                      |      |
| 데이터 상태      | 암호화됨 (Encrypted)          | 평문 (Plaintext)                        |      |
| 파일 시스템 모드 | 읽기/쓰기 (Read/Write)        | 읽기 전용 (Read-Only)                   |      |
| 주요 위협 모델   | 오프라인 공격 (예: 장치 도난) | 온라인 공격 (예: 런타임 변조)           |      |
| 키 관리          | 마스터 키가 암호로 암호화됨   | 루트 해시가 부트로더에 의해 서명/검증됨 |      |

데이터 소스: 15

### 3.3  고급 주제

#### 3.3.1  외부 저장 장치 암호화

동일한 LUKS/`cryptsetup` 도구를 사용하여 USB 드라이브나 추가 NVMe SSD와 같은 외부 장치를 암호화할 수 있습니다.21 주요 차이점은 잠금 해제 메커니즘에 있습니다. 자동 TEE 기반 잠금 해제는 일반적으로 기본 rootfs에 연결됩니다. 외부 데이터 드라이브의 경우, 부팅 후 스크립트에 의해 잠금 해제가 처리될 수 있으며, (이제 신뢰되고 검증된) 루트 파일 시스템에 저장된 키 파일을 사용하거나, `/etc/crypttab`을 사용하여 부팅 시 여러 장치를 잠금 해제하도록 `initrd`를 확장할 수 있습니다.22

#### 3.3.2  데이터 파티션(UDA) 암호화

NVIDIA 문서는 루트 파일 시스템을 암호화하지 않고 사용자 데이터 영역(UDA) 파티션만 암호화하는 구체적인 지침을 제공합니다.21 이는 OS 자체는 민감하지 않지만 애플리케이션에서 생성된 데이터(예: 센서 로그, 사용자 프로필)를 보호해야 하는 경우에 유용한 구성입니다.  `gen_luks.sh` 스크립트를 사용하여 실행 중인 시스템에서 파티션을 동적으로 암호화할 수 있습니다.23

## 4.  엣지 AI 배포에서의 전략적 활용(활용)

이 섹션에서는 기술적 역량을 실질적인 비즈니스 및 운영 가치와 연결하여 쿼리의 "활용" 부분을 직접 다룹니다.

### 4.1  자율 머신 (로봇 및 차량)

배달 로봇, 농업용 트랙터 또는 자율 주행 차량과 같은 자율 시스템에서는 장치 자체가 제품이며 종종 물리적으로 접근 가능합니다.1 디스크 암호화는 여러 가지 이유로 중요합니다:

- **지적 재산(IP) 보호:** AI 모델, 인식 알고리즘 및 경로 계획 소프트웨어는 매우 가치 있는 IP입니다. 암호화는 경쟁사가 장치의 스토리지를 복제하여 소프트웨어를 리버스 엔지니어링하거나 훔치는 것을 방지합니다.15
- **데이터 및 로그 보안:** 이 기계들은 민감한 위치 정보를 포함하거나 사고 분석에 중요한 운영 데이터, 센서 판독값 및 이벤트 로그를 대량으로 수집합니다.53 이 데이터를 암호화하면 기밀성과 무결성이 보장됩니다.
- **악의적인 수정 방지:** `dm-verity`가 rootfs 무결성을 위한 기본 도구이지만, 데이터 파티션을 암호화하면 공격자가 신뢰할 수 있는 애플리케이션이 소비할 수 있는 악의적인 데이터(예: 오염된 지도 데이터)를 주입하는 것을 방지합니다.

### 4.2  스마트 인프라 (소매, 공공 안전, 스마트 시티)

AI 기반 보안 카메라나 스마트 소매점 결제 시스템과 같은 공공 장소의 엣지 장치는 민감한 공공 데이터를 처리합니다.1

- **개인 식별 정보(PII) 보호:** 실시간 얼굴 인식을 수행하는 보안 카메라 시스템이나 거래를 처리하는 스마트 결제 시스템은 이 데이터를 보호해야 합니다. 디스크 암호화는 물리적 장치가 도난당할 경우 저장된 비디오 영상이나 고객 데이터에 접근할 수 없도록 하여 GDPR과 같은 개인 정보 보호 규정을 준수하는 데 도움이 됩니다.
- **운영 무결성 보장:** 공공 안전 맥락에서 공격자는 디스크가 보호되지 않으면 저장된 비디오 증거를 변조할 수 있습니다. 스마트 소매 시스템에서는 판매 데이터를 변경할 수 있습니다. 암호화는 보안 시스템의 일부로서 수집된 데이터의 무결성을 유지하는 데 도움이 됩니다.

### 4.3  규제 산업 (산업 및 헬스케어)

스마트 공장이나 병원과 같은 환경에서는 데이터 보안이 종종 규정에 의해 의무화되며 안전과 비즈니스 연속성에 매우 중요합니다.

- **스마트 제조:** 공장 로봇의 운영 매개변수와 생산 데이터는 귀중한 IP입니다. 품질 관리 또는 예측 유지보수에 사용되는 AI 모델은 보호되어야 합니다.55 장치의 스토리지를 암호화하면 산업 스파이 행위와 생산 오류나 물리적 안전 사고로 이어질 수 있는 변조를 모두 방지합니다. Jetson과 같은 장치에서의 연합 학습(federated learning) 사용은 분산 학습 시나리오에서도 로컬 모델 데이터를 보호해야 할 필요성을 강조합니다.57
- **의료 기기:** 종종 Jetson 하드웨어에서 실행되는 NVIDIA Holoscan 플랫폼은 의료 기기의 실시간 AI를 위해 설계되었습니다.58 환자 데이터(예: 초음파 스트림, 진단 이미지) 보호는 HIPAA와 같은 규정에 따라 엄격한 요구 사항입니다.15 디스크 암호화는 이러한 장치에서 저장 데이터를 보호하고 환자의 기밀성을 보장하기 위한 필수 구성 요소입니다.

모든 애플리케이션에서 디스크 암호화는 단순한 데이터 보호 기능이 아니라 *신뢰*를 가능하게 하는 중요한 요소입니다. 기업이 자율 머신을 배포하거나, 도시가 스마트 카메라를 설치하거나, 병원이 AI 진단 도구를 사용하려면, 장치가 의도한 대로 작동하고 처리하는 데이터를 보호할 것이라는 신뢰가 있어야 합니다. 디스크 암호화는 전체적인 보안 모델의 일부로서 이러한 신뢰를 위한 기술적 기반을 제공하며, 엔지니어링 세부 사항에서 핵심 비즈니스 및 규제 준수 자산으로 변모시킵니다.

## 5.  진화하는 위협 환경과 미래 대비 보안

이 미래 지향적인 섹션은 전략적 통찰력을 보여주며, 보고서를 장기 계획을 위한 가이드로 자리매김합니다.

### 5.1  고급 위협 벡터 및 완화책

#### 5.1.1  부채널 공격 (Side-Channel Attacks)

이러한 공격은 전력 소비나 타이밍과 같은 계산의 물리적 효과를 관찰하여 비밀을 유출하려고 시도합니다.18 TEE가 주요 표적입니다. 어떤 시스템도 면역되지는 않지만, 키 파생이 발생하는 TEE의 하드웨어 격리는 순수 소프트웨어 기반 암호화 방식에 비해 공격자의 장벽을 크게 높입니다.17 공격자는 부팅 시 짧은 잠금 해제 창 동안 SoC의 작동을 탐색하기 위해 물리적 접근과 정교한 장비가 필요합니다.

#### 5.1.2  콜드 부트 공격 (Cold Boot Attacks)

이 공격은 기계를 빠르게 전원 순환시키고 완전히 소멸되기 전에 RAM에서 남은 데이터를 읽어 암호화 키를 복구하는 것을 포함합니다.62 Jetson TEE 기반 모델은 강력한 완화책을 제공합니다. LUKS 암호와 DEK가 부팅 시 보안 TEE 내에서 즉석에서 파생되고 Normal World RAM에 단순하고 정적인 형태로 지속적으로 보관되지 않기 때문에 취약성 창이 극히 작습니다. 키 자료는 물리적 공격자가 쉽게 접근할 수 있는 범용 DRAM이 아닌 SoC의 가장 보호된 부분 내에서 일시적으로 존재합니다.64

### 5.2  포스트 퀀텀의 지평: 양자 미래를 위한 Jetson 준비

#### 5.2.1  임베디드 시스템을 위한 포스트 퀀텀 암호(PQC)의 과제

양자 컴퓨터의 출현은 코드 서명 및 키 교환에 사용되는 현재의 공개 키 암호(RSA 및 ECC 등)를 깨뜨릴 위협이 됩니다.65 NIST 표준 PQC 대체재(예: ML-KEM/Kyber, ML-DSA/Dilithium)는 일반적으로 더 큰 키/서명 크기와 다른 계산 프로필을 가지고 있습니다.67 리소스가 제한된 임베디드 시스템의 경우, 이는 스토리지, 메모리 풋프린트 및 CPU 성능 측면에서 상당한 과제를 제기하여 마이그레이션을 복잡한 작업으로 만듭니다.66

#### 5.2.2  ARMv8에서의 NIST PQC 표준 성능

Jetson Orin의 CPU는 ARMv8 설계입니다. ARM 플랫폼에서 PQC 알고리즘을 벤치마킹한 최근 연구는 미래의 실현 가능성에 대한 중요한 통찰력을 제공합니다.65 Kyber(ML-KEM) 및 Dilithium(ML-DSA)과 같은 격자 기반 방식은 계산적으로 효율적인 것으로 나타나 임베디드 배포에 강력한 후보가 됩니다.67 기존 방식보다 느리지만, 최신 ARM 코어에서의 성능은 실용성의 범위 내에 있습니다. 이는 Jetson Orin 플랫폼이 PQC를 통합하는 향후 소프트웨어 업데이트를 지원하여 "지금 수확하고 나중에 해독하는(harvest now, decrypt later)" 위협으로부터 보호할 수 있는 좋은 위치에 있음을 시사합니다.

**표 7: ARMv8에서의 클래식 vs. 포스트 퀀텀 성능 비교 분석**

| 알고리즘   | 유형 | 보안 수준 | 키/서명 크기 (바이트) | 연산          | ARM에서의 실행 시간 (µs) |      |
| ---------- | ---- | --------- | --------------------- | ------------- | ------------------------ | ---- |
| ECDSA p256 | 서명 | 1         | 64 (서명)             | 서명/검증     | ~100 / ~200              |      |
| ML-DSA-44  | 서명 | 2         | 2560 (서명)           | 서명/검증     | ~150 / ~50               |      |
| RSA-2048   | 서명 | 1         | 256 (서명)            | 서명/검증     | ~1000 / ~50              |      |
| ECDH p256  | KEM  | 1         | 65 (공개키)           | Encaps/Decaps | ~200 / ~200              |      |
| ML-KEM-512 | KEM  | 1         | 800 (공개키)          | Encaps/Decaps | ~50 / ~60                |      |

참고: 이 값들은 학술 벤치마크 65에서 제시된 대표적인 성능을 종합한 것입니다.

PQC 마이그레이션은 전체 기술 산업에 중요한 미래 과제이지만, Jetson Orin 플랫폼은 이러한 전환에 대해 구조적으로 견고합니다. 강력하고 유연한 소프트웨어 스택(JetPack/L4T)과 강력한 ARMv8 CPU에 의존하기 때문에 PQC 알고리즘을 소프트웨어로 구현할 수 있습니다. 성능 데이터는 격자 기반 표준이 실행 가능함을 시사합니다. 개발자에게 가장 즉각적인 영향은 보안 부팅 체인(PQC 서명 확인)과 원격 증명 또는 보안 통신 프로토콜에 있을 것입니다. 디스크 암호화 자체(AES-256)는 양자 내성이 있는 것으로 간주되어 변경할 필요가 없지만, 키를 보호하는 메커니즘은 변경될 수 있습니다.

### 5.3  권장 사항 및 결론

본 보고서의 결과를 종합하면, Jetson AGX Orin은 엣지에서 신뢰할 수 있는 AI를 배포하는 데 중요한 하드웨어 기반의 최첨단 디스크 암호화 시스템을 제공합니다. 성능 오버헤드는 상당한 보안 이득을 위한 필요한 절충안이며, 강력한 하드웨어 가속에 의해 관리 가능해집니다. 민감한 IP, 개인 데이터 또는 제어 시스템을 다루는 모든 애플리케이션에 대해 기본적으로 디스크 암호화를 활성화할 것을 권장합니다. 읽기 전용 시스템의 경우, rootfs에 대한 `dm-verity`와 데이터 파티션에 대한 LUKS의 조합은 강력한 계층적 보안 태세를 제공합니다. 개발자는 정확하고 안전한 구현을 보장하기 위해 L4T 버전과 키 관리 프로세스에 세심한 주의를 기울여야 합니다. 이 플랫폼은 포스트 퀀텀 암호화로의 미래 마이그레이션을 위해 잘 준비되어 있어, 장기 수명 애플리케이션에 대한 관련성과 보안을 보장합니다.

#### **참고 자료**

1. NVIDIA Jetson Orin | Edge AI Platforms - ADLINK Technology, accessed July 2, 2025, https://www.adlinktech.com/en/nvidia-jetson-orin
2. NVIDIA Jetson AGX Orin Series Technical Brief v1.2, accessed July 2, 2025, https://static4.arrow.com/-/media/arrow/files/pdf/jetson-agx-orin-series-technical-brief.pdf?h=16&thn=1&w=16&hash=DAAA547C477DBA9920668D169B05F134
3. NVIDIA Jetson AGX Orin Series, accessed July 2, 2025, https://www.nvidia.com/content/dam/en-zz/Solutions/gtcf21/jetson-orin/nvidia-jetson-agx-orin-technical-brief.pdf
4. The most powerful brain for your robot: Jetson AGX Orin from NVIDIA - YouTube, accessed July 2, 2025, https://m.youtube.com/watch?v=Q5mBBKxpqco&pp=ygUGI2FneGlz
5. NVIDIA Jetson AGX Orin Series - Diamond Systems, accessed July 2, 2025, https://www.diamondsystems.com/files/binaries/Jetson_AGX_Orin_DS-10662-001_v1.2.pdf
6. Deploying DeepSeek AI on NVIDIA Jetson AGX Orin: A Free, Open-Source MIT- Licensed Solution for High-Performance Edge AI in Natural Language Processing and Computer Vision - ResearchGate, accessed July 2, 2025, https://www.researchgate.net/publication/388401833_Deploying_DeepSeek_AI_on_NVIDIA_Jetson_AGX_Orin_A_Free_Open-Source_MIT-_Licensed_Solution_for_High-Performance_Edge_AI_in_Natural_Language_Processing_and_Computer_Vision
7. Security Subsystem - Jetson Orin NX - NVIDIA Developer Forums, accessed July 2, 2025, https://forums.developer.nvidia.com/t/security-subsystem/292863
8. Jetson Orin NX Series Modules Data Sheet - NVIDIA Developer, accessed July 2, 2025, https://developer.nvidia.com/downloads/jetson-orin-nx-module-series-data-sheet
9. NVIDIA Jetson AGX Orin - Open Zeka, accessed July 2, 2025, https://openzeka.com/wp-content/uploads/2022/02/Jetson_AGX_Orin_DS-10662-001_v0.2.pdf
10. NVIDIA Jetson Linux Developer Guide : Security | NVIDIA Docs, accessed July 2, 2025, [https://docs.nvidia.com/jetson/l4t/Tegra%20Linux%20Driver%20Package%20Development%20Guide/bootloader_disk_encryption.html](https://docs.nvidia.com/jetson/l4t/Tegra Linux Driver Package Development Guide/bootloader_disk_encryption.html)
11. Security - Jetson Linux Developer Guide documentation, accessed July 2, 2025, https://docs.nvidia.com/jetson/archives/r35.4.1/DeveloperGuide/text/SD/Security.html
12. For secured boot how to sign the files externally and update on System - Jetson AGX Xavier, accessed July 2, 2025, https://forums.developer.nvidia.com/t/for-secured-boot-how-to-sign-the-files-externally-and-update-on-system/328203
13. OP-TEE: Open Portable Trusted Execution Environment - NVIDIA Jetson Linux Developer Guide, accessed July 2, 2025, https://docs.nvidia.com/jetson/archives/r36.4.3/DeveloperGuide/SD/Security/OpTee.html
14. Precise Usage of OEM_K1 Fuse Key for TA Encryption on Jetson AGX Orin with OP-TEE, accessed July 2, 2025, https://forums.developer.nvidia.com/t/precise-usage-of-oem-k1-fuse-key-for-ta-encryption-on-jetson-agx-orin-with-op-tee/327600
15. Secure Boot and Encrypted Data Storage - Timesys, accessed July 2, 2025, https://www.timesys.com/security/secure-boot-encrypted-data-storage/
16. TPM-backed Full Disk Encryption is coming to Ubuntu, accessed July 2, 2025, https://ubuntu.com/blog/tpm-backed-full-disk-encryption-is-coming-to-ubuntu
17. Load-Step: A Precise TrustZone Execution Control Framework for Exploring New Side-channel Attacks Like Flush+Evict - Zili KOU, accessed July 2, 2025, https://kouzili.github.io/paper/Load_Step.pdf
18. Prime+Reset: Introducing A Novel Cross-World Covert-Channel Through Comprehensive Security Analysis on ARM TrustZone - NUS Computing, accessed July 2, 2025, https://www.comp.nus.edu.sg/~tcarlson/pdfs/chen2024piancctcsaoat.pdf
19. Automated Side-Channel Analysis of ARM TrustZone-M Programs, accessed July 2, 2025, https://conferences.ds.unipi.gr/cybericps2024/assets/papers/8.pdf
20. Step into the Future of Industrial-Grade Edge AI with NVIDIA Jetson AGX Orin Industrial, accessed July 2, 2025, https://developer.nvidia.com/blog/step-into-the-future-of-industrial-grade-edge-ai-with-nvidia-jetson-agx-orin-industrial/
21. Disk Encryption - NVIDIA Jetson Linux Developer Guide 1 documentation, accessed July 2, 2025, https://docs.nvidia.com/jetson/archives/r36.2/DeveloperGuide/SD/Security/DiskEncryption.html
22. Disk Encryption - Jetson Linux
    Developer Guide 34.1 documentation - NVIDIA Docs, accessed July 2, 2025, https://docs.nvidia.com/jetson/archives/r35.1/DeveloperGuide/text/SD/Security/DiskEncryption.html
23. Disk Encryption - NVIDIA Jetson Linux Developer Guide 1 documentation, accessed July 2, 2025, https://docs.nvidia.com/jetson/archives/r35.5.0/DeveloperGuide/SD/Security/DiskEncryption.html
24. Linux Unified Key Setup - Wikipedia, accessed July 2, 2025, https://en.wikipedia.org/wiki/Linux_Unified_Key_Setup
25. Disk Encryption User Guide - Fedora Docs, accessed July 2, 2025, https://docs.fedoraproject.org/en-US/quick-docs/encrypting-drives-using-LUKS/
26. 6 Using Encrypted Block Devices - Oracle Help Center, accessed July 2, 2025, https://docs.oracle.com/en/operating-systems/oracle-linux/9/stordev/stordev-UsingEncryptedBlockDevices.html
27. What is LUKS in Linux? The essential basics - SysDev Laboratories, accessed July 2, 2025, https://www.sysdevlabs.com/articles/storage-technologies/what-is-luks-and-how-does-it-work/
28. dm-crypt - ArchWiki, accessed July 2, 2025, https://wiki.archlinux.org/title/Dm-crypt
29. Configuring LUKS: Linux Unified Key Setup - Red Hat, accessed July 2, 2025, https://www.redhat.com/en/blog/disk-encryption-luks
30. PartitionConfiguration.rst.txt - NVIDIA Docs, accessed July 2, 2025, https://docs.nvidia.com/jetson/archives/r35.1/DeveloperGuide/_sources/text/AR/BootArchitecture/PartitionConfiguration.rst.txt
31. Partition Configuration - NVIDIA Jetson Linux Developer Guide 1 documentation, accessed July 2, 2025, https://docs.nvidia.com/jetson/archives/r36.2/DeveloperGuide/AR/BootArchitecture/PartitionConfiguration.html
32. Jetson Orin AGX Disk Encryption - NVIDIA Developer Forums, accessed July 2, 2025, https://forums.developer.nvidia.com/t/jetson-orin-agx-disk-encryption/288196
33. Encryption of a TA using HUK - Jetson AGX Orin - NVIDIA Developer Forums, accessed July 2, 2025, https://forums.developer.nvidia.com/t/encryption-of-a-ta-using-huk/315874
34. Storage - Jetson Platform Services documentation - NVIDIA Docs, accessed July 2, 2025, https://docs.nvidia.com/jetson/jps/platform-services/storage.html
35. Micron Drive FIO Performance - Ampere Computing, accessed July 2, 2025, https://amperecomputing.com/briefs/micron-drive-FIO-performance
36. Fio Benchmark | JuiceFS Document Center, accessed July 2, 2025, https://juicefs.com/docs/cloud/benchmark/fio/
37. SSD Testing: Quick and Simple Approach - ScaleFlux, accessed July 2, 2025, https://scaleflux.com/blog/ssd-testing-quick-and-simple-approach/
38. The REAL performance impact of using LUKS disk encryption : r/linux - Reddit, accessed July 2, 2025, https://www.reddit.com/r/linux/comments/15wyukc/the_real_performance_impact_of_using_luks_disk/
39. AES instruction set - Wikipedia, accessed July 2, 2025, https://en.wikipedia.org/wiki/AES_instruction_set
40. ARMv8-Crypto-Extensions.md - ThomasKaiser/sbc-bench - GitHub, accessed July 2, 2025, https://github.com/ThomasKaiser/sbc-bench/blob/master/results/ARMv8-Crypto-Extensions.md
41. Preparing a Jetson to flash and boot an encrypted and non-encrypted image for an SSD. - RidgeRun Developer Wiki, accessed July 2, 2025, https://developer.ridgerun.com/wiki/index.php/Preparing_a_Jetson_to_flash_and_boot_an_encrypted_and_non-encrypted_image_for_an_SSD
42. Jetson AGX Orin Long Boot Time - NVIDIA Developer Forums, accessed July 2, 2025, https://forums.developer.nvidia.com/t/jetson-agx-orin-long-boot-time/266855
43. Implement dm-verity | Android Open Source Project, accessed July 2, 2025, https://source.android.com/docs/security/features/verifiedboot/dm-verity
44. dm-verity - The Linux Kernel documentation, accessed July 2, 2025, https://docs.kernel.org/admin-guide/device-mapper/verity.html
45. Using KitKat verified boot - Android Explorations, accessed July 2, 2025, https://nelenkov.blogspot.com/2014/05/using-kitkat-verified-boot.html
46. QT Application Development for Embedded Systems - Mistral Solutions, accessed July 2, 2025, https://www.mistralsolutions.com/?option=com_content&view=article&id=453&Itemid=330&phpMyAdmin=01ab13889148490e8d23c7ff6925ee04
47. Android verified boot within the boot sequence - Information Security Stack Exchange, accessed July 2, 2025, https://security.stackexchange.com/questions/162457/android-verified-boot-within-the-boot-sequence
48. dm-verity - ArchWiki, accessed July 2, 2025, https://wiki.archlinux.org/title/Dm-verity
49. Jetson Linux R32.6.1 Release Page - NVIDIA Developer, accessed July 2, 2025, https://developer.nvidia.com/embedded/linux-tegra-r3261
50. Disk Encryption for Dynamically Created Partitions - NVIDIA Developer Forums, accessed July 2, 2025, https://forums.developer.nvidia.com/t/disk-encryption-for-dynamically-created-partitions/269875
51. Popular embedded vision use cases of NVIDIA® Jetson AGX Orin™ - e-con Systems, accessed July 2, 2025, https://www.e-consystems.com/blog/camera/applications/popular-embedded-vision-use-cases-of-nvidia-jetson-agx-orin/
52. VigiShield Secure by Design: Security Feature Implementation - Timesys Security Solution, accessed July 2, 2025, https://www.timesys.com/solutions/vigishield-secure-by-design/
53. AI-CDA4All: Democratizing Cooperative Autonomous Driving for All Drivers via Affordable Dash-cam Hardware and Open-source AI Software - arXiv, accessed July 2, 2025, https://arxiv.org/html/2505.06749v1
54. Maximizing Deep Learning Performance on NVIDIA Jetson Orin with DLA, accessed July 2, 2025, https://developer.nvidia.com/blog/maximizing-deep-learning-performance-on-nvidia-jetson-orin-with-dla/
55. (PDF) AI-Powered Threat Detection in Smart Manufacturing: Safeguarding the Digital Supply Chain in Additive SMEs - ResearchGate, accessed July 2, 2025, https://www.researchgate.net/publication/392734153_AI-Powered_Threat_Detection_in_Smart_Manufacturing_Safeguarding_the_Digital_Supply_Chain_in_Additive_SMEs
56. Edge Computing in IoT: Enhancing Speed, Security, and Efficiency | E-SPIN Group, accessed July 2, 2025, https://www.e-spincorp.com/edge-computing-iot-speed-security-efficiency/
57. Federated learning for privacy-preserving AI in human–robot collaboration for smart manufacturing | Emerald Insight, accessed July 2, 2025, https://www.emerald.com/insight/content/doi/10.1108/jimse-03-2025-0003/full/html
58. Powering the Future of AI-Enabled Medical Devices with NVIDIA Holoscan and RTI Connext, accessed July 2, 2025, https://developer.nvidia.com/blog/powering-the-future-of-ai-enabled-medical-devices-with-nvidia-holoscan-and-rti-connext/
59. GPU Accelerated Computational Instruments with NVIDIA Holoscan, accessed July 2, 2025, [https://indico.fnal.gov/event/63359/contributions/284748/attachments/175186/237604/Fermilab%20Dune%20-%20021424%20-%20Holoscan%20Overview.pdf](https://indico.fnal.gov/event/63359/contributions/284748/attachments/175186/237604/Fermilab Dune - 021424 - Holoscan Overview.pdf)
60. HPC at the Edge: Enabling Real Time Streaming Sensor Analytics - CLSAC, accessed July 2, 2025, https://www.clsac.org/uploads/5/0/6/3/50633811/clsac-2022-holoscan.pdf
61. NEST-project: AI-based side-channel attacks and countermeasures for autonomous systems, accessed July 2, 2025, https://wasp-sweden.org/nest-project-ai-based-side-channel-attacks-and-countermeasures-for-autonomous-systems/
62. Full text of "The Hitchhiker's Guide to Online Anonymity (2021) + dark version", accessed July 2, 2025, [https://archive.org/stream/the-hitchhikers-guide-to-online-anonymity-2021/The%20Hitchhiker%E2%80%99s%20Guide%20to%20Online%20Anonymity%20%282021%29_djvu.txt](https://archive.org/stream/the-hitchhikers-guide-to-online-anonymity-2021/The Hitchhiker’s Guide to Online Anonymity (2021)_djvu.txt)
63. System Architectures for Data Confidentiality and Frameworks for Main Memory Extraction - mediaTUM, accessed July 2, 2025, https://mediatum.ub.tum.de/doc/1518642/1518642.pdf
64. Veto: Prohibit Outdated Edge System Software from Booting - SciTePress, accessed July 2, 2025, https://www.scitepress.org/Papers/2023/116277/116277.pdf
65. Performance Analysis and Deployment Considerations of Post-Quantum Cryptography for Consumer Electronics - arXiv, accessed July 2, 2025, https://arxiv.org/pdf/2505.02239
66. Future-Proofing Embedded Systems: Why Post-Quantum Cryptography matters - KiviCore, accessed July 2, 2025, https://kivicore.com/embedded-security-blog/post-quantum-embedded-systems-why-post-quantum-cryptography-matters
67. (PDF) Performance Analysis and Deployment Considerations of Post-Quantum Cryptography for Consumer Electronics - ResearchGate, accessed July 2, 2025, https://www.researchgate.net/publication/391462085_Performance_Analysis_and_Deployment_Considerations_of_Post-Quantum_Cryptography_for_Consumer_Electronics
68. Post-Quantum Cryptographic Migration Challenges for Embedded Devices - NXP Semiconductors, accessed July 2, 2025, https://www.nxp.com/docs/en/white-paper/POSTQUANCOMPWPA4.pdf
69. Key Takeaways from the Latest NIST Guidance on Transitioning to Post-Quantum Cryptography - AppViewX, accessed July 2, 2025, https://www.appviewx.com/blogs/key-takeaways-from-the-latest-nist-guidance-on-transitioning-to-post-quantum-cryptography/
70. Post-Quantum Cryptography PQC Challenges, accessed July 2, 2025, https://postquantum.com/post-quantum/post-quantum-pqc-challenges/
71. Saber Post-Quantum Key Encapsulation Mechanism (KEM): Evaluating Performance in Mobile Devices and Suggesting Some Improvements - NIST Computer Security Resource Center, accessed July 2, 2025, https://csrc.nist.gov/CSRC/media/Events/third-pqc-standardization-conference/documents/accepted-papers/ribeiro-saber-pq-key-pqc2021.pdf
72. Building High-Performance Edge AI Solutions with Nvidia Jetson and Edge Impulse, accessed July 2, 2025, https://www.wevolver.com/article/building-high-performance-edge-ai-solutions-with-nvidia-jetson-and-edge-impulse
73. Engineering Secure Devices: A Practical Guide for Embedded System Architects and Developers [1 ed.] 1718503482, 9781718503489, 9781718503496 - DOKUMEN.PUB, accessed July 2, 2025, https://dokumen.pub/engineering-secure-devices-a-practical-guide-for-embedded-system-architects-and-developers-1nbsped-1718503482-9781718503489-9781718503496-g-7925860.html

