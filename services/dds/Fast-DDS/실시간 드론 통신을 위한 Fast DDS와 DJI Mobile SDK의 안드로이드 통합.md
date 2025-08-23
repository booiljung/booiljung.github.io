# 실시간 드론 통신을 위한 Fast DDS와 DJI Mobile SDK의 안드로이드 통합


본 섹션에서는 제안된 시스템의 고수준 설계를 확립하고, 구성 요소와 데이터 흐름을 상세히 설명합니다. 이는 각 기술의 강점과 제약에 기반하여 아키텍처 선택을 정당화하며, 구현에 앞서 견고한 개념적 토대를 제공합니다.


eProsima Fast DDS는 OMG(Object Management Group)의 DDS(Data Distribution Service) 표준을 구현한 C++ 라이브러리로, 데이터 중심 퍼블리셔-서브스크라이버(Data-Centric Publish-Subscribe, DCPS) 통신 모델을 제공합니다.1 이 모델은 메시지 중심 또는 브로커 기반 시스템과 근본적으로 다릅니다. DCPS의 핵심은 애플리케이션들이 강력하게 타입이 정의된 데이터 토픽(Topic)을 발행(publish)하고 구독(subscribe)하는 공유된 "글로벌 데이터 공간(global data space)" 개념에 있습니다.2 이 모델은 생산자와 소비자를 공간(위치), 시간, 흐름 측면에서 분리(decouple)시키며, 이는 구성 요소가 예측 불가능하게 네트워크에 참여하거나 이탈할 수 있는 동적인 로봇 시스템에 이상적입니다.

이러한 특성은 드론 애플리케이션에 있어 중요한 의미를 가집니다. 지상 관제 시스템(Ground Control Station, GCS), 임무 계획기, 그리고 본 보고서에서 개발할 DJI 조종기 앱은 자동 발견(automatic discovery) 메커니즘 덕분에 서로의 IP 주소를 알 필요 없이 'DroneTelemetry'와 같은 토픽을 통해 드론 데이터와 상호작용할 수 있습니다.1 이는 중앙 브로커나 사전 구성된 연결을 요구하는 MQTT 또는 ZeroMQ와 같은 시스템에 비해 핵심적인 장점입니다.3

Fast DDS 아키텍처의 핵심 구성 요소는 다음과 같습니다 1:

- `DomainParticipant`: DDS 도메인으로의 진입점 역할을 합니다.
- `Publisher` 및 `Subscriber`: 데이터를 생성하고 소비하는 주체입니다.
- `Topic`: 이름과 데이터 타입으로 정의되며, 발행자와 구독자를 연결하는 매개체입니다.
- `DataWriter` 및 `DataReader`: 데이터를 송수신하는 엔드포인트입니다.

데이터 구조는 인터페이스 정의 언어(Interface Definition Language, IDL)를 사용하여 엄격하게 정의됩니다. 그 후 자바 애플리케이션인 `fastddsgen` 도구가 직렬화 및 역직렬화를 위한 필수 C++ 타입 지원 코드를 생성합니다.1 이는 DDS 도메인에 참여하는 모든 주체 간의 강력한 데이터 계약을 강제합니다.


DJI Mobile SDK(MSDK) v5는 통일된 키(Key) 기반 아키텍처로 전환되었습니다. 이전 버전처럼 모든 기능에 대해 특정 메서드를 가진 개별적인 관리자 클래스를 사용하는 대신, 대부분의 상호작용이 이제 `IKeyManager` 인터페이스를 통해 이루어집니다.6

핵심 구성 요소 및 메서드는 다음과 같습니다:

- `ISDKManager`: 앱을 초기화하고 등록하기 위한 주 진입점입니다.7

- `IKeyManager`: 모든 데이터 및 제어 작업의 중심 허브입니다. 세 가지 주요 메서드를 사용합니다: 데이터를 읽기 위한 `getValue`, 설정을 변경하기 위한 `setValue`, 그리고 사진 촬영과 같은 명령을 실행하기 위한 `performAction`입니다.6 지속적인 업데이트를 수신하기 위한 

  `listen` 메서드도 존재합니다.

- `IVirtualStickManager`: 자동 비행에 필수적인 고주파 비행 제어 데이터를 전송하기 위한 특수 관리자입니다.7

개발될 안드로이드 애플리케이션은 `FlightControllerKey.KeyAircraftLocation`이나 `BatteryKey.KeyChargeRemainingInPercent`와 같은 키와 함께 `IKeyManager.listen`을 사용하여 텔레메트리를 수집할 것입니다. 또한 `CameraKey.KeyStartShootPhoto`와 같은 키로 `IKeyManager.performAction`을 사용하여 특정 동작을 트리거합니다.6 DDS 네트워크로부터 수신된 비행 제어 명령어는 `VirtualStickFlightControlParam` 객체로 변환되어 `IVirtualStickManager.sendVirtualStickAdvancedParam`을 통해 전송됩니다.11


JNI(Java Native Interface)는 안드로이드에서 관리 코드(Kotlin/Java)와 네이티브 코드(C/C++) 간의 상호작용을 가능하게 하는 표준 프레임워크입니다.14 JNI는 강력하지만, 신중하게 관리하지 않으면 성능 병목 현상을 일으키기 쉽습니다. JNI 경계를 넘나드는 모든 호출에는 상당한 오버헤드가 발생하며, JVM 힙과 네이티브 메모리 간의 데이터 마샬링(복사)은 비용이 많이 듭니다.15

본 애플리케이션의 핵심 기능은 자바 기반의 MSDK에서 C++ 기반의 Fast DDS 라이브러리로 고주파 텔레메트리와 고대역폭 비디오 데이터를 전송하는 것입니다. 모든 업데이트마다 데이터를 복사하는 순진한 구현 방식은 허용할 수 없는 지연 시간과 가비지 컬렉션(GC) 압박을 유발할 것입니다. 따라서, 설계 시 JNI 호출을 최소화하고 데이터 복사를 제거하는 것을 최우선으로 해야 합니다. 이는 C++와 자바 간에 네이티브 메모리 영역을 공유하여 제로-카피(zero-copy) 데이터 전송을 가능하게 하는 `DirectByteBuffer`를 주요 데이터 교환 메커니즘으로 선택하는 이유가 됩니다.17


이 하위 섹션에서는 상세한 아키텍처 다이어그램을 통해 시스템의 데이터 흐름을 설명합니다.

- **데이터 흐름 (업링크: 드론에서 네트워크로)**
  1. DJI 드론이 H.264 비디오와 텔레메트리를 DJI RC(조종기)로 스트리밍합니다.
  2. 안드로이드 앱의 `Foreground Service`가 실행됩니다.
  3. DJI MSDK(자바)가 콜백을 통해 데이터를 수신합니다 (비디오는 `ICameraStreamManager.addReceiveStreamListener`, 텔레메트리는 `IKeyManager.listen`).10
  4. **JNI 브리지**: 자바 계층이 이 데이터를 사전에 할당된 `DirectByteBuffer`에 씁니다.
  5. 네이티브 C++ 계층이 `DirectByteBuffer`의 메모리 주소에서 직접 데이터를 읽습니다.
  6. Fast DDS `DataWriter`가 데이터를 직렬화(Fast CDR 사용)하여 UDP 또는 TCP를 통해 DDS 글로벌 데이터 공간에 발행합니다.1
- **데이터 흐름 (다운링크: 네트워크에서 드론으로)**
  1. 원격 DDS 애플리케이션(예: GCS)이 `Command` 토픽을 발행합니다.
  2. 네이티브 C++ 계층의 `DataReader`가 DDS 샘플을 수신합니다.
  3. **JNI 브리지**: C++ 계층이 네이티브-투-자바 콜백을 호출합니다.
  4. 자바 계층이 명령어를 수신하고 적절한 MSDK 함수(예: `IVirtualStickManager.sendVirtualStickAdvancedParam`)를 호출합니다.11
  5. MSDK가 명령어를 드론으로 전송합니다.

이러한 아키텍처에서 안드로이드 `Foreground Service`의 사용은 단순히 좋은 관행을 넘어, 애플리케이션의 생존을 위한 필수 요건입니다. 사용자가 앱의 UI에서 벗어나는 즉시 표준 안드로이드 백그라운드 실행 제한이 DDS 통신 소켓이나 MSDK 연결을 종료시킬 수 있습니다. `Foreground Service`는 운영체제(OS)가 해당 프로세스를 우선적으로 처리하도록 보장하여, 장시간 실행되는 네트워크 작업에 적합하게 만듭니다.21 본 애플리케이션은 DJI 드론과 DDS 네트워크 양쪽에 지속적인 연결을 유지해야 하며, 이는 장시간 실행되는 I/O 작업입니다. 최신 안드로이드 버전은 배터리 절약을 위해 백그라운드 프로세스를 매우 공격적으로 종료합니다. 일반적인 `Activity`나 `Service`는 앱이 백그라운드로 전환된 후 수 분 내에 종료될 수 있습니다. 반면, `Foreground Service`는 사용자에게 지속적인 알림을 표시함으로써 OS에 사용자가 백그라운드 활동을 인지하고 동의했음을 알립니다.22 따라서 MSDK 리스너와 Fast DDS 참가자를 포함한 전체 통신 로직은 `Foreground Service` 내에 캡슐화되어야만 지속적인 작동을 보장할 수 있습니다. 이는 안드로이드 플랫폼 자체에서 파생된 핵심적인 설계 제약 조건입니다.


본 섹션은 전체 개발 및 빌드 툴체인을 설정하기 위한 규범적인 단계별 가이드를 제공합니다. 복잡한 NDK 프로젝트에서 흔히 발생하는 실패의 주된 원인인 이 과정을 명확하게 정의하여 성공적인 빌드 경로를 제시하는 것을 목표로 합니다.


개발 환경 설정의 첫 단계는 안드로이드 스튜디오와 필수 도구를 설치하는 것입니다. SDK 매니저를 통해 안드로이드 스튜디오, NDK(Side by side), 그리고 CMake를 설치해야 합니다.25

프로젝트 구성은 주로 모듈 수준의 `build.gradle(.kts)` 파일에서 이루어집니다. 먼저, DJI의 Maven 저장소에서 MSDK 종속성을 추가해야 합니다. 문서는 기체, 네트워크 등 다양한 패키지를 명시하고 있으며, 이들을 모두 포함시켜야 합니다.27 다음으로, `android.defaultConfig` 블록에 타겟 SDK, 최소 SDK, NDK 버전을 구성합니다. 가장 중요한 단계는 `externalNativeBuild` 블록을 추가하고 이를 프로젝트의 `CMakeLists.txt` 파일에 연결하는 것입니다.25 빌드 시간을 줄이기 위해 `ndk` 블록에서 빌드할 ABI(예: `arm64-v8a`)를 명시하는 것이 좋습니다.31 프로젝트 수준의 `build.gradle(.kts)`과 `settings.gradle(.kts)` 파일에서는 `maven.dji.com`을 저장소로 포함하도록 설정해야 합니다.

마지막으로, `AndroidManifest.xml` 파일에 필수 권한(인터넷, 블루투스, USB, 위치 등)을 선언하고 DJI API 키를 메타데이터로 추가해야 합니다.27 또한, `connectedDevice`나 `dataSync`와 같은 적절한 `foregroundServiceType`과 함께 `Foreground Service`를 선언해야 합니다.33


Fast DDS는 C++ CMake 프로젝트이며, 안드로이드용으로 사전 컴파일된 AAR 형태로 공식 제공되지 않습니다. 따라서 소스에서 직접 크로스 컴파일해야 합니다. 공식 로드맵에서 향후 "안드로이드 플랫폼에 대한 Tier 1 지원"을 언급하고 있지만 35, 현재로서는 수동 크로스 컴파일이 필요합니다. 이 과정은 안드로이드 NDK에서 제공하는 특별한 "툴체인 파일(toolchain file)"을 사용하여 CMake를 호출하는 것을 포함합니다. 이 파일은 컴파일러(Clang), 시스템 루트, 타겟 ABI 및 API 레벨을 구성합니다.

단계별 가이드는 다음과 같습니다:

1. `Fast-CDR` 저장소를 복제합니다.36
2. 빌드 디렉터리를 생성하고, `CMAKE_TOOLCHAIN_FILE`을 `$NDK_PATH/build/cmake/android.toolchain.cmake`로 지정하고 `-DANDROID_ABI=arm64-v8a`를 설정하여 CMake를 실행합니다.37 빌드 후 로컬 스테이징 디렉터리에 설치합니다.
3. `Fast-DDS` 저장소를 복제합니다.40
4. `CMAKE_PREFIX_PATH`를 스테이징 디렉터리로 설정하여 방금 빌드한 Fast CDR 라이브러리를 찾을 수 있도록 하고, 동일한 CMake 과정을 반복합니다. 이때 `-DBUILD_SHARED_LIBS=ON`과 같은 주요 CMake 플래그가 매우 중요합니다.38

이 과정은 또한 Asio 및 TinyXML2와 같은 Fast DDS 자체의 종속성을 고려해야 하며, 이는 `THIRDPARTY=ON` CMake 플래그로 처리할 수 있습니다.41


애플리케이션 자체의 `CMakeLists.txt` 파일은 JNI 래퍼 코드와 크로스 컴파일된 Fast DDS 라이브러리를 연결하는 접착제 역할을 합니다. CMake 구성은 다음과 같이 진행됩니다:

1. `add_library(native-bridge SHARED...)`를 사용하여 JNI 래퍼 라이브러리를 정의합니다.43
2. 디버깅을 위해 NDK의 로그 라이브러리(`log-lib`)를 찾기 위해 `find_library(...)`를 사용합니다.44
3. `add_library(... SHARED IMPORTED)`와 `set_target_properties(... IMPORTED_LOCATION...)`를 사용하여 스테이징 디렉터리에 있는 사전 컴파일된 `libfastcdr.so` 및 `libfastdds.so`에 대한 타겟을 정의합니다.44
4. `target_link_libraries(native-bridge...)`를 사용하여 JNI 라이브러리를 `log`, `fastdds`, `fastcdr`에 연결합니다.44 이 명령은 링커를 위한 종속성 그래프를 설정합니다.

공유 라이브러리를 빌드하고 로드하는 순서는 매우 중요하며 직관적이지 않을 수 있습니다. 종속성은 그것에 의존하는 구성 요소가 처리되기 전에 사용 가능해야 합니다. 이는 컴파일 타임 링크(`target_link_libraries`)와 런타임 로드(`System.loadLibrary`) 모두에 적용됩니다. `native-bridge` 코드는 `fastdds`의 헤더를 포함하므로, 컴파일 시 `libfastdds.so`가 링크되어야 합니다. 마찬가지로, Fast DDS 코드는 `fastcdr`의 헤더를 포함하므로, `libfastdds.so`를 빌드할 때 `libfastcdr.so`가 링크되어야 합니다. 이는 `Fast-CDR` -->> `Fast-DDS` -->> `native-bridge`라는 명확한 빌드 타임 종속성 체인을 형성합니다.

런타임에 안드로이드 `PackageManager`가 네이티브 라이브러리를 로드할 때, 심볼을 해석해야 합니다. 만약 `libnative-bridge.so`를 로드하려 할 때 `libfastdds.so`의 심볼을 찾지 못하면(아직 로드되지 않았기 때문에), 애플리케이션은 `UnsatisfiedLinkError`와 함께 충돌할 것입니다. 따라서 자바 코드는 종속성의 역순으로 라이브러리를 명시적으로 로드해야 합니다: `System.loadLibrary("fastcdr")`, 그 다음 `System.loadLibrary("fastdds")`, 그리고 마지막으로 `System.loadLibrary("native-bridge")` 순서입니다. 더 나아가, 78 및 79에서 언급된 바와 같이 공유 C++ 런타임을 사용하는 경우, `libc++_shared.so`는 `libfastcdr.so`보다 먼저 로드되어야 합니다. 이는 애플리케이션의 시작 코드(예: `Application` 클래스 또는 `Foreground Service`의 `onCreate` 메서드)에서 구현되어야 하는 엄격한 4단계 로딩 순서를 만듭니다. 이 순서를 간과하는 것은 흔하고 치명적인 런타임 오류의 원인이 됩니다.


본 섹션은 애플리케이션의 안드로이드 특정(Kotlin/Java) 부분에 대한 구현 세부 사항을 제공합니다.


SDK 통합의 첫 단계는 애플리케이션의 생명주기에 맞춰 SDK를 올바르게 초기화하는 것입니다. 이를 위해 사용자 정의 `Application` 클래스를 생성하여 `attachBaseContext`에서 일회성 SDK 설치(`com.secneo.sdk.Helper.install`)를 처리해야 합니다.11 그 후, `Application.onCreate` 또는 중앙 싱글턴에서 `SDKManager`를 초기화하고 앱을 등록합니다. 이 과정에는 컨텍스트와 비동기 등록 프로세스의 결과를 처리할 `SDKManagerCallback`을 제공하는 것이 포함됩니다.7 이 콜백은 `onRegister`, `onProductConnect`, `onProductDisconnect` 등의 이벤트를 수신합니다. 코드는 연결된 제품이 MSDK v5에서 지원되는 모델인지 확인해야 하며, 지원 목록은 광범위하지만 특정 모델로 제한됩니다.47


애플리케이션은 `IKeyManager.listen`을 사용하여 키-값 업데이트를 구독함으로써 텔레메트리 데이터를 수집합니다. 예를 들어, `FlightControllerKey.KeyAircraftLocation`에 대한 리스너를 생성하면 최대 10Hz로 호출되며, 수신된 데이터(예: `LocationCoordinate3D`)는 JNI 계층으로 전달됩니다.48

비디오 데이터의 경우, `ICameraStreamManager`가 진입점입니다. `addReceiveStreamListener`를 사용하여 원시 H.264 비디오 프레임을 `byte` 형태로 수신하는 리스너를 등록합니다.20 이 바이트 배열이 네이티브 계층으로 전달될 데이터입니다. 이전의 `DJICodecManager`도 `sendDataToDecoder` 메서드를 제공하지만, 우리의 목적에는 리스너 방식이 더 직접적입니다.49

가상 스틱 제어는 다음 단계를 따릅니다:

1. `IVirtualStickManager.getInstance().enableVirtualStick(...)`을 호출하여 가상 스틱 모드를 활성화합니다.11

2. `setVirtualStickAdvancedModeEnabled(true)`를 통해 고급 모드를 활성화합니다.11 이는 스틱 각도를 시뮬레이션하는 대신 속도 및 위치 제어를 가능하게 하고, 드론이 바람에 대해 자동으로 안정화되도록 하므로 매우 중요합니다.48

3. JNI 계층에서 수신한 명령에 따라 `pitch`, `roll`, `yaw`, `verticalThrottle` 필드를 채운 `VirtualStickFlightControlParam` 객체를 생성합니다.11

4. `sendVirtualStickAdvancedParam(...)`을 사용하여 5-25 Hz의 높은 빈도로 명령을 전송합니다.11 이 작업은 `Foreground Service`에서 관리하는 전용 스레드나 루프에서 수행될 가능성이 높습니다.


안드로이드에서 장시간 실행되는 네트워크 작업을 안정적으로 유지하기 위해 `Foreground Service` 구현이 필수적입니다. 이를 위해 `android.app.Service`를 상속하는 클래스를 생성합니다. `onCreate`에서는 JNI 브리지와 네이티브 Fast DDS 참가자를 초기화하고, `onDestroy`에서는 리소스를 해제하기 위해 네이티브 참가자가 적절히 종료되도록 보장해야 합니다.

서비스가 시작될 때 `onStartCommand` 메서드 내에서 `Notification`을 생성하고 `startForeground()`를 호출해야 합니다. 이는 최신 안드로이드 버전을 타겟으로 하는 앱이 네트워크 I/O와 같은 장시간 백그라운드 작업을 수행하기 위한 엄격한 요구사항입니다.21 이 알림은 사용자에게 드론 통신 링크가 활성 상태임을 알려줍니다. 대안으로, 네트워크 가용성과 같은 제약 조건이 있는 더 복잡한 백그라운드 작업을 위해서는 `WorkManager`를 사용할 수 있습니다. `WorkManager`는 앱을 대신하여 `Foreground Service`를 관리할 수 있기 때문입니다.24 그러나 실시간 소켓 연결과 같이 지속적인 작업의 경우, 직접 `Foreground Service`를 구현하는 것이 더 간단하고 직접적인 제어를 제공합니다.

DJI MSDK v5의 `IKeyManager.listen` 및 `IVirtualStickManager.send...` API는 비동기적이고 콜백 기반인 반면, Fast DDS의 `DataWriter.write`는 동기 또는 비동기 방식일 수 있습니다.1 이 두 패러다임을 연결하여 지연 시간을 최소화하고 중요한 스레드를 차단하지 않도록 하는 것은 중요한 설계 결정입니다. MSDK는 SDK에서 제공하는 임의의 백그라운드 스레드에서 텔레메트리 데이터를 전달합니다. 만약 MSDK의 콜백 스레드에서 동기식 `write`를 직접 호출하면 해당 스레드가 차단될 위험이 있으며, 이는 MSDK가 후속 데이터 패킷을 놓치거나 불안정해지는 원인이 될 수 있습니다. 이는 전형적인 동시성 문제입니다. 반대로 비동기식 DDS 쓰기를 사용하면 데이터 생명주기를 신중하게 관리해야 합니다. MSDK에서 전달된 `byte` 또는 `ByteBuffer`는 DDS 비동기 스레드가 처리를 완료할 때까지 수정되거나 가비지 컬렉션되어서는 안 됩니다.

견고한 해결책은 스레드 안전하고 비차단적인 큐(예: `java.util.concurrent.ConcurrentLinkedQueue`)를 중개자로 사용하는 것입니다. MSDK 콜백 스레드는 수신된 데이터(텔레메트리 또는 비디오 프레임)를 큐에 넣는 단순하고 빠른 작업만 수행합니다. `Foreground Service`에 의해 관리되는 별도의 전용 "DDS Publisher" 스레드가 이 큐를 지속적으로 폴링하고, 데이터를 검색한 후 JNI 함수를 호출하여 Fast DDS를 통해 발행합니다. 이 설계는 MSDK의 타이밍과 DDS의 타이밍을 분리하여 차단을 방지하고, 필요한 경우 흐름 제어 및 일괄 처리를 위한 명확한 지점을 제공하여 더 안정적이고 성능이 뛰어난 시스템을 만듭니다.


본 섹션은 시스템의 성능을 결정하는 저수준 메모리 관리 및 데이터 전송에 대한 기술적 핵심을 상세히 다룹니다.


JNI 브리지의 첫 단계는 자바/코틀린과 C++ 간의 계약을 정의하는 것입니다. 코틀린/자바 측에서는 `external` 함수를 포함하는 `object` 또는 `class`를 정의합니다. 예를 들어, `external fun initializeDds(): Long`, `external fun publishTelemetry(buffer: ByteBuffer)`, `external fun shutdownDds(participantPtr: Long)`와 같이 선언할 수 있습니다.14

C++ 측에서는 `Java_com_package_ClassName_methodName`이라는 이름 규칙을 따르는 해당 JNI 함수를 구현해야 합니다. C++ 이름 맹글링(name mangling)을 방지하기 위해 `extern "C"`를 사용해야 합니다.14 이 구현은 `JNIEnv*` 포인터와 `ByteBuffer`에 대한 `jobject`와 같은 자바 인자를 받게 됩니다.


핵심 원칙은 제로-카피(zero-copy)입니다. 즉, JVM이 힙에 `byte`를 할당하고 데이터를 복사한 후, JNI 계층이 이를 다시 네이티브 메모리로 복사하는 과정을 피하는 것입니다.

구현 과정은 다음과 같습니다:

1. **자바**: `Foreground Service`에서 `ByteBuffer.allocateDirect(size)`를 사용하여 한 번만 `ByteBuffer`를 할당합니다. 이 버퍼의 메모리는 JVM 힙 외부에 존재하며 안정적인 메모리 주소를 가집니다.17
2. **자바**: 이 `ByteBuffer` 객체를 네이티브 `publishTelemetry` 함수에 전달합니다.
3. **C++**: JNI 함수 내에서 `env->GetDirectBufferAddress(buffer)`를 사용하여 버퍼의 원시 메모리 주소를 가져옵니다. 이 함수는 `void*`를 반환하며, 이를 텔레메트리 `struct`에 대한 포인터로 캐스팅할 수 있습니다.19
4. **C++**: C++ 스택에 텔레메트리 `struct`를 생성합니다.
5. **C++**: `memcpy(buffer_address, &my_struct, sizeof(my_struct))`를 사용하여 구조체의 바이트 표현을 공유 버퍼에 직접 복사합니다.54
6. **C++**: Fast DDS `DataWriter`는 이 메모리 위치에서 직접 데이터를 발행할 수 있습니다.

이 방법은 여러 번의 메모리 복사 대신 단 한 번의 `memcpy`만 포함하므로 지연 시간을 극적으로 줄이고 GC 압박을 완화합니다.


`memcpy`를 사용할 때, 우리는 C++ `struct`의 원시 바이트 이미지를 생성합니다. 이 이미지는 읽는 프로세스(다른 아키텍처의 C++ 애플리케이션 또는 디버깅을 위한 자바 코드)가 정확히 동일한 메모리 레이아웃으로 해석할 때만 유효합니다. 두 가지 요인, 즉 패딩(padding)과 엔디언(endianness)이 이를 방해할 수 있습니다.57

패딩은 C++ 컴파일러가 멤버 변수들이 워드 경계에 정렬되도록(예: 4바이트 `int`는 4로 나누어지는 주소에서 시작) 구조체 멤버 사이에 삽입하는 바이트를 의미합니다. 이 패딩은 암묵적이며 컴파일러와 아키텍처에 따라 다를 수 있습니다.59 엔디언은 `int`, `double`과 같은 멀티바이트 타입의 바이트 순서로, 리틀 엔디언(인텔) 또는 빅 엔디언(일부 ARM, 네트워크 순서)일 수 있습니다.

이 문제를 해결하기 위한 최선의 방법은 다음과 같습니다:

1. **패딩 제어**: 플랫폼 간 데이터 구조의 경우, 구조체 정의 전후에 `#pragma pack(push, 1)`과 `#pragma pack(pop)`을 사용하여 특정 패킹 정렬을 강제하는 것이 가장 안전합니다. 이는 모든 패딩을 제거하여 동일한 메모리 레이아웃을 보장하지만 일부 아키텍처에서는 약간의 성능 저하를 유발할 수 있습니다.61

2. **엔디언 제어**: Fast DDS의 기본 직렬화 프로토콜인 CDR은 엔디언을 명시합니다. 발행 시 Fast CDR이 이 변환을 자동으로 처리하므로 64, C++ 구조체를 Fast DDS 

   `DataWriter`에 전달하는 한 수동으로 바이트를 교환할 필요가 없습니다. 이 문제는 DDS `DataReader`를 사용하지 않고 다른 엔디언 시스템에서 `memcpy`된 버퍼를 해석하려 할 때만 발생합니다.

개발자가 메모리 레이아웃을 시각화하고 검증할 수 있도록 다음 표를 제공하는 것이 필수적입니다. 이 표는 C++ 텔레메트리 구조체의 각 필드, C++ 타입, 크기, 계산된 바이트 오프셋(패딩 또는 `#pragma pack` 고려), 그리고 자바에서 이를 읽고 쓰기 위한 해당 `ByteBuffer` 메서드(예: `buffer.getFloat(offset)`)를 상세히 기술합니다. 고성능 브리지의 핵심은 C++ `struct`를 `DirectByteBuffer`에 `memcpy`하는 것입니다. 이는 타입 안전성을 우회하는 저수준 메모리 작업이므로, 메모리 레이아웃에 대한 오해는 조용한 데이터 손상으로 이어질 수 있습니다. 컴파일러가 삽입하는 패딩이나 데이터 타입 크기(`long`은 4 또는 8바이트일 수 있음)와 같은 요소는 즉각적으로 명확하지 않으며 가변적일 수 있습니다. 표는 이 암묵적인 메모리 계약을 명시적으로 만듭니다. 예를 들어, `struct Telemetry { uint8_t status; float altitude; };`에 대해 64비트 컴파일러는 `altitude`를 4바이트 경계에 정렬하기 위해 `status` 뒤에 3바이트 패딩을 추가할 수 있습니다. 표는 이를 명확히 보여줌으로써(status: 오프셋 0, 패딩: 오프셋 1, 크기 3, altitude: 오프셋 4), 개발자가 자바 측에서 `ByteBuffer`를 읽을 때 패딩을 고려하도록 강제하여 데이터 손상을 방지합니다.

| 필드명       | C++ 타입 | 크기 (바이트) | 오프셋 (`#pragma pack(1)`) | Java `ByteBuffer` 메서드 |
| ------------ | -------- | ------------- | -------------------------- | ------------------------ |
| `latitude`   | `double` | 8             | 0                          | `getDouble(0)`           |
| `longitude`  | `double` | 8             | 8                          | `getDouble(8)`           |
| `altitude`   | `float`  | 4             | 16                         | `getFloat(16)`           |
| `velocity_x` | `float`  | 4             | 20                         | `getFloat(20)`           |
| `...`        | `...`    | `...`         | `...`                      | `...`                    |


`memcpy`는 POD(Plain Old Data) 구조체에 대해 빠르지만 취약합니다. 대안으로 FlatBuffers와 같은 제로-카피 전용 직렬화 라이브러리를 사용할 수 있습니다. FlatBuffers는 파싱/압축 해제 단계 없이 버퍼 내에서 직접 데이터를 탐색하여 직렬화된 데이터에 접근할 수 있게 해줍니다.66

구현 개요는 다음과 같습니다:

1. `.fbs` 파일에서 스키마를 정의합니다.
2. 스키마로부터 C++ 및 자바 코드를 생성합니다.
3. **C++**: FlatBuffers 빌더를 사용하여 버퍼에 데이터 페이로드를 생성합니다.
4. **JNI**: C++ 빌더의 기본 `uint8_t*`를 `NewDirectByteBuffer`로 생성된 `DirectByteBuffer`를 통해 자바로 전달합니다.
5. **자바**: 생성된 자바 FlatBuffers 클래스를 사용하여 `ByteBuffer`에서 직접 데이터 필드에 접근합니다.

이 방식은 빌드 단계에 종속성을 추가하지만, 스키마 진화(하위/상위 호환성)를 제공하며 원시 `memcpy`보다 안전합니다. 복잡하거나 진화하는 데이터 구조의 경우, 이는 더 우수한 선택입니다.17 원시 `memcpy`와 FlatBuffers와 같은 라이브러리 사이의 선택은 POD 타입에 대한 순수한 성능/단순성과 복잡하거나 진화하는 데이터 타입에 대한 유지보수성/안전성 간의 근본적인 트레이드오프를 나타냅니다. 프로젝트가 간단한 텔레메트리 구조체로 시작할 때 `memcpy`는 가장 빠르고 직접적인 방법입니다.54 그러나 요구사항이 변경되어 새로운 센서가 추가되거나 데이터 필드 타입이 변경되면(`int32_t`에서 `int64_t`로), `memcpy` 방식은 C++ 구조체 변경 시 자바 측 파싱 로직과 오프셋 테이블의 수동 변경을 요구합니다. 분산 시스템의 모든 참여자(예: GCS)가 새로운 구조체 레이아웃으로 업데이트되지 않으면 데이터 파싱에 실패하여 시스템이 취약해집니다. 반면, FlatBuffers는 스키마를 통해 데이터 계약을 강제하고, 이전 클라이언트를 손상시키지 않으면서 새로운 필드를 추가할 수 있는 호환성을 제공합니다.66 순수 `memcpy`에 비해 약간의 오버헤드가 있지만, 중요하거나 진화하는 시스템에서의 견고성, 유지보수성, 안전성 측면에서의 이득은 막대합니다. 따라서 전문가의 권장 사항은 다음과 같습니다: 초기 프로토타이핑이나 안정적이고 단순한 데이터 구조(예: 비디오 프레임 헤더)에는 `memcpy`를 사용하고, 진화할 가능성이 있는 주요 텔레메트리 및 명령 토픽에는 처음부터 FlatBuffers와 같은 스키마 기반 직렬화 라이브러리에 투자하는 것이 장기적으로 이익이 되는 전략적 아키텍처 결정입니다.


본 섹션은 네이티브 라이브러리로 실행되며 모든 DDS 통신을 처리하는 C++ 코드에 대해 상세히 설명합니다.


먼저 `DroneData.idl` 파일을 생성하고 데이터 토픽을 위한 `struct`를 정의해야 합니다.

```
// DroneData.idl
struct DroneTelemetry {
    double latitude;
    double longitude;
    float altitude;
    float velocity_x;
    //... 기타 필드
};

struct H264VideoFrame {
    unsigned long long timestamp;
    sequence<octet> frame_data;
};

struct FlightCommand {
    float pitch;
    float roll;
    float yaw_velocity;
    float vertical_velocity;
};
```

그 후 `fastddsgen -replace DroneData.idl`을 실행하여 `DroneData.h`, `DroneData.cxx` 및 `DroneDataTypeSupport.h`와 같은 타입 지원 파일을 생성합니다.2


`initializeDds` JNI 함수는 DDS 엔티티를 설정하는 역할을 합니다. 구현 단계는 다음과 같습니다:

1. `DomainParticipantFactory::get_instance()->create_participant(...)`를 사용하여 `DomainParticipant`를 생성합니다.
2. `fastddsgen`으로 생성된 데이터 타입(예: `DroneTelemetryTypeSupport`)을 등록합니다.
3. 텔레메트리와 비디오를 위한 `Topic`을 생성합니다.
4. `Publisher`를 생성합니다.
5. 각 토픽에 대한 `DataWriter`를 생성합니다.
6. 이러한 엔티티, 특히 `DomainParticipant`와 `DataWriter`에 대한 포인터를 전역 또는 정적 구조체에 저장하여 후속 JNI 호출에서 접근할 수 있도록 합니다. `DomainParticipant` 포인터는 종료 시 사용할 핸들로 사용하기 위해 `long`으로 캐스팅하여 자바로 반환합니다.

`publishTelemetry` JNI 함수는 `DirectByteBuffer`를 받아 `memcpy`를 통해 데이터를 C++ `DroneTelemetry` 구조체 인스턴스에 복사한 후, `telemetry_writer->write(&telemetry_sample)`를 호출하여 데이터를 발행합니다.


명령 수신을 위해 `dds::DataReaderListener`를 상속하는 C++ 클래스를 생성하고, `on_data_available` 메서드를 재정의해야 합니다. 이 메서드 내에서 `reader->take_next_sample(...)`을 호출하여 `FlightCommand` 데이터를 가져옵니다.

JNI 콜백 메커니즘은 다음과 같이 구현됩니다:

1. 리스너는 `JavaVM*`과 명령을 처리할 자바 객체에 대한 전역 참조(`jobject`)에 접근해야 합니다.
2. 이들은 초기화 시(예: `initializeDds` JNI 호출 중) 캐시되어야 합니다.
3. `on_data_available`에서 캐시된 `JavaVM*`을 사용하여 현재 스레드의 `JNIEnv*`를 얻고, 핸들러 함수의 자바 메서드 ID를 찾아 명령 데이터를 전달하며 호출합니다.

`initializeDds` 함수 내에서 `Subscriber`, `DataReader`를 생성하고 이 리스너 인스턴스를 연결하여 구독자 설정을 완료합니다.

JNI 객체와 네이티브 스레드의 생명주기 관리는 복잡하고 오류가 발생하기 쉽습니다. DDS 리스너는 주 애플리케이션 스레드가 아닌 Fast DDS 미들웨어가 생성한 스레드에서 호출됩니다. 적절한 설정 없이 이 스레드에서 자바로 콜백을 시도하면 실패합니다. DDS `DataReaderListener`의 `on_data_available` 콜백은 내부 Fast DDS 스레드에서 실행됩니다. 초기 JNI 호출(예: `initializeDds`)에서 얻은 `JNIEnv*`는 해당 스레드에만 유효하므로 15, 리스너 스레드에서 캐시하여 사용할 수 없습니다. 네이티브 스레드에서 JVM으로 콜백을 하려면 해당 스레드가 먼저 JVM에 연결(attach)되어야 합니다. 올바른 절차는 다음과 같습니다: (a) 초기 JNI 호출에서 `env->GetJavaVM(&g_jvm)`을 사용하여 `JavaVM*` 포인터를 캐시합니다. (b) `env->NewGlobalRef(callback_object)`를 사용하여 콜백을 처리할 자바 객체에 대한 전역 참조를 생성합니다. 지역 참조는 JNI 메서드가 반환되면 무효화됩니다.15 (c) 리스너 함수(`on_data_available`) 내에서 `g_jvm->AttachCurrentThread(&env, NULL)`를 호출하여 현재 스레드에 유효한 `JNIEnv*`를 얻습니다. (d) 이 새로운 `env`를 사용하여 캐시된 전역 참조에 대해 자바 메서드를 호출합니다. (e) 작업이 끝나면 `g_jvm->DetachCurrentThread()`를 호출합니다. 이 연결/분리 생명주기는 JVM이 생성하지 않은 스레드에서 발생하는 모든 네이티브-투-자바 콜백에 필수적이며, 이를 지키지 않으면 복잡한 JNI 애플리케이션에서 흔히 충돌이 발생합니다.


본 마지막 섹션에서는 전문가의 논평을 제공하고, 기술적 선택의 맥락을 설명하며, 프로토타입에서 생산 시스템으로 나아가기 위한 지침을 제공합니다.


주요 성능 지표는 종단 간 지연 시간(비디오의 경우 glass-to-glass, 텔레메트리의 경우 sensor-to-GCS)과 처리량입니다. `eProsima Fast DDS Monitor`를 사용하여 DDS 네트워크 트래픽을 검사하고, 지연 시간을 측정하며, 패킷 손실을 식별할 수 있습니다.70 안드로이드 측에서는 안드로이드 스튜디오의 내장 프로파일러를 사용하여 CPU 사용량과 메모리 할당(GC 이벤트)을 확인해야 합니다.

최적화 전략은 다음과 같습니다:

- **DDS QoS**: 서비스 품질(QoS) 정책을 미세 조정합니다. 비디오의 경우 낮은 지연 시간을 우선시하기 위해 `BEST_EFFORT` 신뢰성이 허용될 수 있지만, 명령어의 경우 `RELIABLE`이 필수적입니다. `HISTORY` 깊이를 조절하여 메모리를 절약할 수 있습니다.
- **JNI 일괄 처리**: 텔레메트리 업데이트마다 JNI 호출을 하는 대신, 자바 측 큐에서 여러 업데이트를 일괄 처리하여 단일 호출로 JNI 브리지를 통해 전송함으로써 오버헤드를 줄일 수 있습니다.
- **비디오 키프레임**: DJI MSDK의 원시 H.264 스트림에는 I-프레임(키프레임)이 자주 없을 수 있습니다. 스트림 중간에 참여하는 DDS 구독자는 I-프레임을 수신할 때까지 디코딩할 수 없습니다. 따라서 애플리케이션은 MSDK에 키프레임을 요청하거나, 일부 DJI 샘플에서 이미 종속성으로 사용되는 FFmpeg과 같은 도구를 사용하여 스트림을 더 지능적으로 관리해야 할 수 있습니다.50


- **DDS (Data Distribution Service)**:
  - **강점**: 브로커가 없는 P2P 아키텍처와 자동 발견 기능.1 풍부하고 세분화된 QoS 제어(신뢰성, 내구성, 데드라인 등).1 표준화되어 다른 벤더 구현과 상호 운용 가능. ROS 2의 기본 미들웨어.73 동적인 로컬 네트워크에서 실시간 다대다 통신에 이상적입니다.
- **MQTT (Message Queuing Telemetry Transport)**:
  - **강점**: 매우 가벼운 클라이언트 프로토콜. 저대역폭, 고지연 또는 불안정한 네트워크(주로 셀룰러를 통한 IoT)에 탁월함.3 브로커 기반 아키텍처는 방화벽 통과를 단순화하고 중앙 관리 지점을 제공합니다.
  - **본 사용 사례의 약점**: 중앙 브로커는 단일 장애점이며 실시간 로컬 네트워크 제어에 잠재적인 지연 병목 현상을 유발할 수 있습니다.4 QoS가 DDS보다 덜 세분화되어 있습니다.
- **ZeroMQ**:
  - **강점**: 다양한 패턴(pub/sub, req/rep 등)을 위한 소켓과 유사한 추상화를 제공하는 고성능 메시징 *라이브러리*.3 극도로 낮은 지연 시간과 높은 처리량. 브로커가 없습니다.
  - **본 사용 사례의 약점**: 완전한 미들웨어가 아닌 라이브러리입니다. 내장된 발견 기능, 타입 안전성/IDL, 풍부한 QoS 정책이 부족하여, 이러한 기능들을 ZeroMQ 위에 직접 구축해야 하며, 이는 사실상 DDS의 일부를 재구현하는 것과 같습니다.3

결론적으로, 낮은 지연 시간, 실시간 데이터 교환, 동적 발견 및 강력한 데이터 타이핑이 요구되는 로컬 네트워크의 로봇 구성 요소에는 Fast DDS가 우월한 아키텍처 선택입니다.


- **보안**: DDS-Security를 구현해야 합니다. Fast DDS는 인증, 접근 제어, 암호화 플러그인을 제공하여 드론 명령 및 텔레메트리 스트림을 무단 접근이나 변조로부터 보호하는 데 필수적입니다.1 이를 위해서는 `SECURITY=ON` 플래그로 Fast DDS를 빌드하고 필요한 인증서를 관리해야 합니다.

- **DDS 라우터를 통한 네트워크 브리징**: WAN이나 인터넷을 통해 드론을 제어하기 위해(예: 원격 클라우드 서버에서) `eProsima DDS Router`를 사용해야 합니다. 이 도구는 드론의 로컬 Wi-Fi/OcuSync 네트워크와 클라우드 기반 DDS 도메인과 같은 두 개의 분리된 DDS 도메인을 연결하고, TCP를 통해 투명하게 토픽을 전달할 수 있습니다.71

- **기록 및 재생**: 테스트, 시뮬레이션, 사후 임무 분석을 위해 `eProsima DDS Record & Replay`를 사용하여 모든 DDS 트래픽을 MCAP 파일에 캡처하고 정확한 타이밍으로 재생할 수 있습니다.71 이는 물리적 드론 없이 복잡한 상호작용을 디버깅하는 데 매우 유용합니다.

- **ROS 2 통합**: Fast DDS는 ROS 2의 기본 미들웨어이므로, 이 아키텍처는 DJI 드론을 더 넓은 ROS 2 생태계와 직접적이고 고성능으로 통합할 수 있는 경로를 제공합니다. 안드로이드 앱에서 발행된 토픽은 동일 네트워크상의 모든 ROS 2 노드(예: SLAM, 내비게이션, 인식)에서 직접 소비될 수 있습니다.73 이는 이 설계의 중요한 전략적 이점입니다.


1. Fast DDS - eProsima, accessed July 15, 2025, https://fast-dds.docs.eprosima.com/
2. Fast DDS Documentation, accessed July 15, 2025, https://media.readthedocs.org/pdf/eprosima-fast-rtps/latest/eprosima-fast-rtps.pdf
3. MQTT Vs ZeroMQ for IoT - HiveMQ, accessed July 15, 2025, https://www.hivemq.com/blog/mqtt-vs-zeromq-for-iot/
4. MQTT vs ZeroMQ | Svix Resources, accessed July 15, 2025, https://www.svix.com/resources/faq/mqtt-vs-zeromq/
5. Fast DDS Documentation, accessed July 15, 2025, https://fast-dds.docs.eprosima.com/_/downloads/en/v2.1.0/pdf/
6. Differences between 4.X and 5.X - Mobile SDK, accessed July 15, 2025, https://developer.dji.com/doc/mobile-sdk-tutorial/en/quick-start/version-differences.html
7. DJI Mobile SDK Documentation - DJI Developer, accessed July 15, 2025, https://developer.dji.com/api-reference-v5/android-api/index.html
8. class DJISDKManager - DJI Mobile SDK Documentation, accessed July 15, 2025, https://developer.dji.com/api-reference/android-api/index.html
9. class KeyManager - Mobile SDK - DJI Developer, accessed July 15, 2025, https://developer.dji.com/api-reference/android-api/Components/KeyManager/DJIKeyManager.html
10. class IKeyManager - DJI Mobile SDK Documentation, accessed July 15, 2025, https://developer.dji.com/api-reference-v5/android-api/Components/IKeyManager/IKeyManager.html
11. Tips for setting up an Android application with DJI Mini 3 Pro & DJI MSDK v5.7.0 - Medium, accessed July 15, 2025, https://medium.com/@cropcopter42/tips-for-setting-up-an-android-application-with-dji-mini-3-pro-dji-msdk-v5-7-0-ec0007f71b3b
12. Virtual Stick Sample - Mobile SDK - DJI Developer, accessed July 15, 2025, https://developer.dji.com/doc/mobile-sdk-tutorial/en/tutorials/virtual-stick.html
13. Overview of MSDK Architecture - Mobile SDK, accessed July 15, 2025, https://developer.dji.com/doc/mobile-sdk-tutorial/en/basic-introduction/overview.html
14. Android NDK & JNI Guide: Integrating C/C++ with Kotlin (Step-by-Step) | Medium, accessed July 15, 2025, https://medium.com/@dev.rutwijb/android-ndk-overview-509e198645c3
15. JNI tips - NDK - Android Developers, accessed July 15, 2025, https://developer.android.com/training/articles/perf-jni
16. Is rewriting some Java code to C++ using JNI to improve performance a good idea? [closed], accessed July 15, 2025, https://softwareengineering.stackexchange.com/questions/210278/is-rewriting-some-java-code-to-c-using-jni-to-improve-performance-a-good-idea
17. jni-benchmarks/DataBenchmarks.md at main - GitHub, accessed July 15, 2025, https://github.com/evolvedbinary/jni-benchmarks/blob/main/DataBenchmarks.md
18. Java API Performance Improvements - RocksDB, accessed July 15, 2025, https://rocksdb.org/blog/2023/11/06/java-jni-benchmarks.html
19. how to write and read from bytebuffer passing from java to jni - Stack Overflow, accessed July 15, 2025, https://stackoverflow.com/questions/13386176/how-to-write-and-read-from-bytebuffer-passing-from-java-to-jni
20. class ICameraStreamManager - DJI Mobile SDK Documentation, accessed July 15, 2025, https://developer.dji.com/api-reference-v5/android-api/Components/IMediaDataCenter/ICameraStreamManager.html
21. Services overview | Background work - Android Developers, accessed July 15, 2025, https://developer.android.com/develop/background-work/services
22. Mastering Android Components: Services - Started, Bound, Foreground, Background and IntentService | by Manish Kumar | Medium, accessed July 15, 2025, https://medium.com/@manishkumar_75473/mastering-android-components-services-started-bound-foreground-background-and-intentservice-bca5fd982710
23. Foreground services overview | Background work - Android Developers, accessed July 15, 2025, https://developer.android.com/develop/background-work/services/fgs
24. Background tasks overview | Background work - Android Developers, accessed July 15, 2025, https://developer.android.com/develop/background-work/background-tasks
25. Add C and C++ code to your project | Android Studio, accessed July 15, 2025, https://developer.android.com/studio/projects/add-native-code
26. Install and configure the NDK and CMake | Android Studio, accessed July 15, 2025, https://developer.android.com/studio/projects/install-ndk
27. Integrate SDK into Application - DJI Mobile SDK Documentation - DJI Developer, accessed July 15, 2025, https://developer.dji.com/mobile-sdk/documentation/application-development-workflow/workflow-integrate.html
28. dji-sdk/Mobile-SDK-Android-V5: MSDK V5 Sample - GitHub, accessed July 15, 2025, https://github.com/dji-sdk/Mobile-SDK-Android-V5
29. Link Gradle to your native library | Android Studio, accessed July 15, 2025, https://developer.android.com/studio/projects/gradle-external-native-builds
30. CMake | Android NDK, accessed July 15, 2025, https://developer.android.com/ndk/guides/cmake
31. Integrate a C++ library with your Android build system - Slions, accessed July 15, 2025, https://slions.net/threads/integrate-a-c-library-with-your-android-build-system.45/
32. Run Sample Application - DJI Mobile SDK Documentation - DJI Developer, accessed July 15, 2025, https://developer.dji.com/mobile-sdk/documentation/quick-start/index.html
33. Guide to Foreground Services on Android 14 - Medium, accessed July 15, 2025, https://medium.com/@domen.lanisnik/guide-to-foreground-services-on-android-9d0127dc8f9a
34. Support for long-running workers | Background work - Android Developers, accessed July 15, 2025, https://developer.android.com/develop/background-work/background-tasks/persistent/how-to/long-running
35. roadmap.md - eProsima/Fast-DDS - GitHub, accessed July 15, 2025, https://github.com/eProsima/Fast-RTPS/blob/master/roadmap.md
36. eProsima/Fast-CDR: eProsima FastCDR library provides two serialization mechanisms. One is the standard CDR serialization mechanism, while the other is a faster implementation of it. Looking for commercial support? Contact info@eprosima.com - GitHub, accessed July 15, 2025, https://github.com/eProsima/Fast-CDR
37. New cmake modules commit breaks android build / Issue #80 / eProsima/Fast-DDS - GitHub, accessed July 15, 2025, https://github.com/eProsima/Fast-RTPS/issues/80
38. Problem cross compiling for Android. [13697] / eProsima Fast-DDS / Discussion #2545, accessed July 15, 2025, https://github.com/eProsima/Fast-DDS/discussions/2545
39. Fast DDS Shapes - Maruniak, Lukáš - GitLab, accessed July 15, 2025, https://gitlab.fel.cvut.cz/marunluk/fastdds-shapes
40. eProsima/Fast-DDS: The most complete DDS - Proven: Plenty of success cases. Looking for commercial support? Contact info@eprosima.com - GitHub, accessed July 15, 2025, https://github.com/eProsima/Fast-DDS
41. 7. CMake options - Fast DDS 2.14.4 documentation, accessed July 15, 2025, https://fast-dds.docs.eprosima.com/en/2.14.x/installation/configuration/cmake_options.html
42. 7. CMake options - 3.3.0 - Fast DDS - eProsima, accessed July 15, 2025, https://fast-dds.docs.eprosima.com/en/stable/installation/configuration/cmake_options.html
43. Add JNI(C/C++) into your existing Android app | - erev0s.com, accessed July 15, 2025, https://erev0s.com/blog/add-jnicc-your-existing-android-app/
44. Configure CMake | Android Studio, accessed July 15, 2025, https://developer.android.com/studio/projects/configure-cmake
45. How to link C/C++ library to your Android project with CMake - Torcheux Frédéric - Medium, accessed July 15, 2025, https://bowser-f.medium.com/link-c-c-library-dependencies-to-your-own-c-c-code-in-an-android-application-using-cmake-79a165202ff9
46. Android JNI/C++/CMake: how to add and build interdependent C++ libraries?, accessed July 15, 2025, https://stackoverflow.com/questions/68210898/android-jni-c-cmake-how-to-add-and-build-interdependent-c-libraries
47. Android MSDK v5.13.0 Version Release Notes - Mobile SDK, accessed July 15, 2025, https://developer.dji.com/doc/mobile-sdk-tutorial/en/
48. Flight Controller - Mobile SDK - DJI Developer, accessed July 15, 2025, https://developer.dji.com/doc/mobile-sdk-tutorial/en/basic-introduction/basic-concepts/flight-controller.html
49. class DJICodecManager - DJI Mobile SDK Documentation, accessed July 15, 2025, https://developer.dji.com/api-reference/android-api/Components/CodecManager/DJICodecManager.html
50. Android Video Stream Decoding Sample - DJI Mobile SDK Documentation, accessed July 15, 2025, https://developer.dji.com/mobile-sdk/documentation/sample-code/index.html
51. DJI virtual stick-tutorial, waypoint- and hotpoint-mission for DJI Mini- and Air-series | Page 4 | B4X Programming Forum, accessed July 15, 2025, https://www.b4x.com/android/forum/threads/dji-virtual-stick-tutorial-waypoint-and-hotpoint-mission-for-dji-mini-and-air-series.140089/page-4
52. Wrapping a returned C struct from inside JNI (and other JNI stuff), accessed July 15, 2025, https://bedroomcoders.co.uk/posts/48
53. JNI: Direct buffer reading & writing - java - Stack Overflow, accessed July 15, 2025, https://stackoverflow.com/questions/44197468/jni-direct-buffer-reading-writing
54. Pass a C++ struct to Java - Oracle Forums, accessed July 15, 2025, https://forums.oracle.com/ords/apexds/post/pass-a-c-struct-to-java-8980
55. Copy struct into char array - Stack Overflow, accessed July 15, 2025, https://stackoverflow.com/questions/26430446/copy-struct-into-char-array
56. I need help copying struct to char array - C++ Forum, accessed July 15, 2025, https://cplusplus.com/forum/general/192568/
57. why when i want copy (with memcpy)struct array to char array , character array is null??, accessed July 15, 2025, https://forum.qt.io/topic/80200/why-when-i-want-copy-with-memcpy-struct-array-to-char-array-character-array-is-null
58. Change endianness of entire struct in C++ - Stack Overflow, accessed July 15, 2025, https://stackoverflow.com/questions/52062584/change-endianness-of-entire-struct-in-c
59. Converting a struct to char array using memcpy - c++ - Stack Overflow, accessed July 15, 2025, https://stackoverflow.com/questions/33229336/converting-a-struct-to-char-array-using-memcpy
60. The Lost Art of C Structure Packing : r/cpp - Reddit, accessed July 15, 2025, https://www.reddit.com/r/cpp/comments/1u8a9w/the_lost_art_of_c_structure_packing/
61. GNU Compiler Collection (GCC) Internals, accessed July 15, 2025, https://gcc.gnu.org/onlinedocs/gcc-3.4.0/gccint/Misc.html
62. pack pragma | Microsoft Learn, accessed July 15, 2025, https://learn.microsoft.com/en-us/cpp/preprocessor/pack?view=msvc-170
63. JNI Structure Alignment - java - Stack Overflow, accessed July 15, 2025, https://stackoverflow.com/questions/19856048/jni-structure-alignment
64. fastcdr - ROS Index, accessed July 15, 2025, https://index.ros.org/r/fastcdr/
65. fastcdr - OpenEmbedded Layer Index, accessed July 15, 2025, https://layers.openembedded.org/layerindex/recipe/406923/
66. Java produce a huge and different byte array result from C++ / Issue #3809 / google/flatbuffers - GitHub, accessed July 15, 2025, https://github.com/google/flatbuffers/issues/3809
67. Open-Source fastFFI: An Efficient Java Cross-Language Communication Frame, accessed July 15, 2025, https://www.alibabacloud.com/blog/599128
68. Efficiently passing GPB serialized data from Java to C++ using JNI - Stack Overflow, accessed July 15, 2025, https://stackoverflow.com/q/22069443
69. MapBuffer: data structure optimized for JNI access / reactwg react-native-new-architecture / Discussion #32 - GitHub, accessed July 15, 2025, https://github.com/reactwg/react-native-new-architecture/discussions/32
70. eProsima Fast DDS Monitor Documentation - Read the Docs, accessed July 15, 2025, https://fast-dds-monitor.readthedocs.io/
71. eProsima documentation index - all-docs 1.0 documentation, accessed July 15, 2025, https://docs.eprosima.com/
72. Fast DDS - The middleware powering the ESO ELT, accessed July 15, 2025, https://www.eso.org/sci/meetings/2023/RTC4AO/01_11_martin_losa.pdf
73. Fast DDS Installation | PX4 User Guide (v1.12), accessed July 15, 2025, https://docs.px4.io/v1.12/en/dev_setup/fast-dds-installation.html
74. eProsima Fast DDS - ROS 2 Documentation: Iron documentation, accessed July 15, 2025, https://docs.ros.org/en/iron/Installation/DDS-Implementations/Working-with-eProsima-Fast-DDS.html
75. Is MQTT the fastest protocol for sending messages/communicating with robots remotely? Are there other protocols I can explore that are faster/as fast and more secure? - Reddit, accessed July 15, 2025, https://www.reddit.com/r/robotics/comments/1i89n04/is_mqtt_the_fastest_protocol_for_sending/
76. ROS2 node for DJI Tello and Visual SLAM for mapping of indoor environments. - GitHub, accessed July 15, 2025, https://github.com/tentone/tello-ros2
77. Introduction - psdk_ros2 wrapper documentation - GitHub Pages, accessed July 15, 2025, https://umdlife.github.io/psdk_ros2/documentation/Introduction.html
78. 6.4 Using DDS C++ Libraries for Android Applications - RTI Community, accessed July 15, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/getting_started_platforms/android_systems_addendum/android_systems_addendum/Using_DDS_Cpp_Libs.htm
79. C++ library support - NDK - Android Developers, accessed July 15, 2025, https://developer.android.com/ndk/guides/cpp-support

