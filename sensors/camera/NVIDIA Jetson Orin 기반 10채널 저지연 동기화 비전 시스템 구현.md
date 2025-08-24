[Camera](./index.md)
# NVIDIA Jetson Orin 기반 10채널 저지연 동기화 비전 시스템 구현



10대의 카메라를 단일 임베디드 시스템에 연결하는 것은 단순히 I/O 포트의 수를 맞추는 문제를 넘어섭니다. 이는 시스템의 I/O 처리량, 데이터 처리 능력, 메모리 대역폭의 한계를 시험하는 복잡한 엔지니어링 과제입니다. 여기에 '낮은 전송 지연(low-latency)'이라는 요구사항이 더해지면, 일반적으로 사용되는 많은 접근 방식들이 즉시 배제됩니다.1 또한, 자율 주행이나 3D 재구성(3D reconstruction)과 같은 고급 애플리케이션에서는 여러 시점의 데이터를 일관성 있게 융합해야 하므로, 마이크로초 단위의 정밀한 '프레임 동기화'는 타협할 수 없는 필수 요건이 됩니다.2 이러한 다차원적인 제약 조건은 시스템 아키텍처 설계에 있어 매우 신중하고 전략적인 접근을 요구합니다.


본 보고서의 핵심 결론을 명확히 제시하면, 사용자의 요구사항을 충족시키는 유일하고 실용적이며 확장 가능한 솔루션은 **NVIDIA Jetson AGX Orin** 플랫폼을 중심으로, **다채널 디시리얼라이저 허브(Deserializer Hub)**를 사용하는 **SerDes(Serializer/Deserializer) 기반 아키텍처**입니다.

이 아키텍처는 다음과 같은 핵심 과제들을 직접적으로 해결합니다.

- **확장성**: SerDes 허브는 8개, 12개, 또는 16개와 같은 다수의 카메라 스트림을 Jetson의 MIPI CSI-2 포트로 집약(aggregate)하여, Jetson 모듈이 기본적으로 제공하는 포트 수의 한계를 극복합니다.4
- **저지연**: 이 방식은 MIPI CSI-2와 거의 동일한 수준의 데이터 경로를 유지합니다. 이는 USB나 이더넷/RTSP 방식에 비해 현저히 낮은 지연 시간을 보장하는 핵심적인 장점입니다.1
- **견고성**: SerDes 기술은 최대 15미터에 달하는 장거리 케이블 연결을 지원하고, 동일한 케이블을 통해 전력을 공급하며(PoC, Power-over-Coax), 높은 신호 무결성을 보장하여 산업 및 자동차 애플리케이션에 매우 적합합니다.9
- **동기화**: SerDes 허브는 모든 연결된 카메라에 하드웨어 동기화 신호를 분배하도록 명시적으로 설계되어, 모든 카메라가 동시에 프레임을 캡처하도록 보장합니다.2

이러한 이유로, 임시방편적인 해결책 대신 SerDes 기반 아키텍처를 선택하는 것은 단순한 옵션이 아니라, 프로젝트의 성공을 위한 전략적 필수 사항입니다. 사용자의 핵심 요구사항인 10대의 카메라 연결, 저지연, 그리고 동기화를 고려할 때, 다른 기술들은 명백한 한계를 보입니다. 예를 들어, USB 카메라 다수를 사용하는 방식은 USB 소프트웨어 스택으로 인해 상당한 지연 시간과 CPU 부하를 유발하여 저지연 요구사항을 충족시키지 못합니다.1 이더넷(RTSP) 카메라는 카메라 자체에서 H.264/HEVC 인코딩과 디코딩 과정을 거치면서 최대 수 초에 달하는 지연을 발생시켜 실시간 애플리케이션에는 부적합합니다.1 MIPI CSI-2 멀티플렉서는 카메라 간 전환에는 유용할 수 있으나, 동시 스트리밍과 SerDes가 제공하는 견고성(케이블 길이, PoC) 측면에서는 부족합니다.13 따라서, 이러한 대안들을 기술적 요구사항에 근거하여 배제하고 나면, SerDes 아키텍처는 가장 좋은 선택지를 넘어, 전문가 수준의 시스템을 구축하기 위한 유일한 선택지로 부상합니다. 본 보고서의 모든 후속 하드웨어 및 소프트웨어 결정은 이 근본적인 판단에 기반을 둡니다.



10채널 카메라 워크로드를 처리하기 위해서는 Jetson Orin 제품군 내에서도 신중한 선택이 필요합니다. 결론적으로, Jetson AGX Orin은 이 과업에 필요한 I/O, 메모리, 그리고 연산 능력을 모두 갖춘 유일한 논리적 선택입니다.

- **Jetson AGX Orin**: 이 제품은 최대 275 TOPS에 달하는 최고 수준의 AI 성능을 제공하여, 10개의 고해상도 비디오 스트림을 실시간으로 처리하는 데 필수적인 연산 능력을 갖추고 있습니다.15 가장 중요한 점은, 최대 16개의 MIPI CSI-2 레인과 C-PHY를 통해 최대 6대의 네이티브 카메라를 지원하며, 16개의 가상 채널(Virtual Channels)을 처리할 수 있는 가장 강력한 카메라 인터페이스를 탑재하고 있다는 것입니다.15 이 높은 레인 수와 가상 채널 지원은 다채널 SerDes 허브와 인터페이스하기 위한 하드웨어 전제 조건입니다. 또한, 204.8 GB/s에 달하는 방대한 메모리 대역폭은 10개의 RAW 비디오 스트림에서 발생하는 막대한 양의 데이터를 병목 현상 없이 처리하는 데 반드시 필요합니다.18
- **Jetson Orin NX/Nano**: 이 모듈들은 강력하지만, 이번 과업에는 제약이 따릅니다. 이들은 8개의 MIPI 레인과 최대 8개의 가상 채널만을 지원하여 I/O 대역폭이 제한적입니다.17 8대의 카메라 시스템은 구현 가능할 수 있으나, 10대로 확장하는 것은 여러 개의 허브를 사용하는 복잡한 솔루션 없이는 거의 불가능하며, 이는 상당한 소프트웨어 및 통합 복잡성을 야기할 것입니다.

다음 표는 Jetson Orin 제품군의 주요 다중 카메라 관련 사양을 요약한 것입니다.

**표 1: NVIDIA Jetson Orin 시리즈: 다중 카메라 관련 사양 비교**

| 기능                                                         | Jetson AGX Orin (64GB)   | Jetson Orin NX (16GB)   | Jetson Orin Nano (8GB)  |      |
| ------------------------------------------------------------ | ------------------------ | ----------------------- | ----------------------- | ---- |
| **AI 성능 (TOPS)**                                           | 최대 275                 | 최대 100                | 최대 40                 |      |
| **GPU**                                                      | 2048-core NVIDIA Ampere  | 1024-core NVIDIA Ampere | 1024-core NVIDIA Ampere |      |
| **CPU**                                                      | 12-core Arm Cortex-A78AE | 8-core Arm Cortex-A78AE | 6-core Arm Cortex-A78AE |      |
| **메모리**                                                   | 64GB 256-bit LPDDR5      | 16GB 128-bit LPDDR5     | 8GB 128-bit LPDDR5      |      |
| **메모리 대역폭**                                            | 204.8 GB/s               | 102.4 GB/s              | 68 GB/s                 |      |
| **MIPI CSI-2**                                               | 16 레인 (D-PHY/C-PHY)    | 8 레인 (D-PHY)          | 8 레인 (D-PHY)          |      |
| **최대 네이티브 카메라**                                     | 6                        | 4                       | 4                       |      |
| **최대 가상 채널**                                           | 16                       | 8*                      | 8*                      |      |
| **이더넷**                                                   | 10GbE                    | 1GbE                    | 1GbE                    |      |
| 참고: Orin NX 및 Nano의 가상 채널 지원은 변경될 수 있습니다.17 |                          |                         |                         |      |
| 데이터 출처: 15                                              |                          |                         |                         |      |


시스템의 전체 지연 시간을 최소화하기 위해서는 카메라 인터페이스의 선택이 매우 중요합니다. 다음은 주요 인터페이스 기술에 대한 지연 시간 중심의 분석입니다.

- **MIPI CSI-2 (Camera Serial Interface 2)**: 저지연 비전 시스템의 황금 표준입니다. 이는 센서와 SoC의 이미지 신호 처리 장치(ISP) 또는 메모리 간의 직접적인 포인트-투-포인트 인터페이스입니다. 높은 대역폭과 최소한의 프로토콜 오버헤드를 통해 '글래스-투-메모리(glass-to-memory)' 지연 시간을 가장 낮게 유지할 수 있습니다.1 주된 한계는 약 30cm의 짧은 케이블 길이지만, 이 문제는 SerDes 기술을 통해 완벽하게 해결됩니다.11
- **USB (Universal Serial Bus)**: 편리하고 널리 지원되는 범용 인터페이스이지만, 호스트 OS와 카메라 펌웨어 양단의 USB 프로토콜 스택은 예측 불가능하고 상당한 지연 시간을 추가합니다. 동기화된 저지연 다중 카메라 시스템에는 적합하지 않습니다.1
- **이더넷 (RTSP/GigE Vision)**: 케이블 길이 면에서 가장 큰 유연성을 제공하지만, 가장 높은 지연 시간이라는 대가를 치릅니다. 비디오 데이터는 카메라에서 압축(예: H.264/HEVC)되고, TCP/IP를 통해 전송된 후, 호스트에서 다시 디코딩되어야 합니다. 이 인코딩/디코딩 과정은 상당한 지연(최대 수 초)을 추가하며, Jetson의 귀중한 하드웨어 디코더(NVDEC) 자원을 소모합니다.1

아래 표는 각 인터페이스 기술의 주요 특성을 비교하여 MIPI/SerDes 아키텍처 선택의 타당성을 명확히 보여줍니다.

**표 2: 카메라 인터페이스 기술 비교**

| 인터페이스             | 일반적인 지연 시간    | 최대 대역폭            | 최대 케이블 길이 | 전원 공급 | 동기화          | 호스트 CPU 부하 |      |
| ---------------------- | --------------------- | ---------------------- | ---------------- | --------- | --------------- | --------------- | ---- |
| **MIPI CSI-2**         | 매우 낮음 (<1 프레임) | 높음 (레인당 2.5Gbps+) | 짧음 (~30cm)     | D-PHY     | 네이티브 지원   | 낮음            |      |
| **GMSL2/FPD-Link III** | 낮음 (MIPI + 약간)    | 매우 높음 (6Gbps+)     | 김 (15m+)        | PoC/PoL   | 하드웨어 지원   | 낮음            |      |
| **USB 3.0**            | 중간 (수십 ms)        | 중간 (5Gbps)           | 중간 (~3m)       | VBUS      | 소프트웨어 기반 | 중간            |      |
| **이더넷 (RTSP)**      | 높음 (수백 ms ~ 초)   | 가변적 (1Gbps+)        | 매우 김 (100m+)  | PoE       | PTP (복잡함)    | 높음 (디코딩)   |      |
| 데이터 출처: 1         |                       |                        |                  |           |                 |                 |      |



SerDes는 고속 데이터 전송을 위해 병렬 데이터를 직렬 데이터로 변환(Serializer)하여 전송하고, 수신 측에서 다시 병렬 데이터로 복원(Deserializer)하는 기술입니다. 이 기술은 카메라의 MIPI 데이터를 고속 직렬 스트림으로 변환하여, 단일 동축 케이블이나 STP 케이블을 통해 장거리로 안정적으로 전송할 수 있게 해줍니다.10

- **GMSL™ (Gigabit Multimedia Serial Link)**: Analog Devices(구 Maxim Integrated)가 개발한 자동차 등급 표준으로, 높은 대역폭(GMSL2는 최대 6Gbps), 견고한 EMC 성능, 양방향 제어 채널 및 PoC(Power-over-Coax)와 같은 고급 기능으로 잘 알려져 있습니다.9 특히, 링크당 최대 16개의 가상 채널을 지원하여 복잡한 시스템 설계에 유연성을 제공합니다.11
- **FPD-Link™ III**: Texas Instruments가 개발한 경쟁적인 자동차 등급 표준입니다. GMSL과 유사하게 15미터 이상의 장거리 케이블, PoC, 양방향 제어 채널을 지원합니다. FPD-Link III는 GMSL2에 비해 약간 낮은 4.16Gbps의 대역폭과 최대 4개의 가상 채널을 지원합니다.22

기술적으로 GMSL2가 대역폭과 가상 채널 지원 측면에서 약간의 우위를 가지지만 22, 두 기술 모두 매우 우수하며, 최종 선택은 특정 카메라 모듈과 디시리얼라이저 허브의 생태계 지원 및 가용성에 따라 결정되는 경우가 많습니다.

**표 3: GMSL2 vs. FPD-Link III 기술 비교**

| 기능                       | GMSL2          | FPD-Link III      |      |
| -------------------------- | -------------- | ----------------- | ---- |
| **최대 순방향 대역폭**     | 6 Gbps         | 4.16 Gbps         |      |
| **최대 역방향 대역폭**     | 187.5 Mbps     | 50 Mbps           |      |
| **최대 가상 채널**         | 16             | 4                 |      |
| **지원 순방향 인터페이스** | MIPI, Parallel | 4-lane MIPI CSI-2 |      |
| **지원 역방향 인터페이스** | I2C, UART      | I2C               |      |
| **주요 공급업체**          | Analog Devices | Texas Instruments |      |
| 데이터 출처: 11            |                |                   |      |


10개의 카메라 입력을 직접 지원하는 Jetson 모듈은 없기 때문에, 여러 카메라 스트림을 하나로 모으는 디시리얼라이저 허브(aggregator/deserializer hub)가 반드시 필요합니다. 시스템의 물리적 아키텍처는 다음과 같이 구성됩니다: 10대의 GMSL2 카메라가 각각의 동축 케이블을 통해 16채널 GMSL2 디시리얼라이저 허브에 연결되고, 이 허브는 다시 MIPI CSI-2 리본 케이블을 통해 Jetson AGX Orin에 연결됩니다. 동기화 신호는 허브에서 생성되어 각 카메라로 분배됩니다.

이 아키텍처를 구현하기 위한 상용 솔루션들이 존재하며, 이는 개념을 실제 제품으로 전환하는 데 있어 매우 중요한 정보입니다.

- **D3 Engineering**: Jetson AGX Orin용 16채널 GMSL2 및 FPD-Link III 디시리얼라이저 인터페이스 카드를 제공합니다.4 이 보드들은 이러한 고밀도 애플리케이션을 위해 특별히 설계되었으며, 하드웨어 수준의 동기화 기능을 포함하고 있습니다. 16채널 보드의 가격은 약 $799입니다.24
- **Leopard Imaging**: AGX Orin용 8채널 GMSL2 디시리얼라이저 보드(LI-JAG-ADP-GMSL2-8CH)를 약 $557에 제공합니다.27 10대의 카메라를 연결하려면 이 보드 2개가 필요하지만, 이는 두 개의 개별 I2C 버스를 관리해야 하는 등 상당한 소프트웨어 통합의 복잡성을 야기합니다.
- **e-con Systems**: 구형 Jetson 모델용 6채널 어댑터와 같은 다중 카메라 솔루션을 제공하며 6, STURDeCAM25 키트와 함께 AGX Orin용 8-커넥터 디시리얼라이저 보드를 $549(키트 가격)에 제공합니다.30

10채널 시스템의 경우, D3 Engineering과 같은 공급업체의 단일 16채널 보드를 사용하는 것이 여러 개의 8채널 보드를 사용하는 것보다 하드웨어 및 소프트웨어 통합을 단순화하므로 가장 이상적이고 견고한 솔루션입니다.

**표 4: Jetson Orin용 상용 다채널 SerDes 허브**

| 제품명                                    | 공급업체        | 채널 수 | SerDes 타입  | 동기화 지원      | 가격 (샘플) | 주요 특징                                         |      |
| ----------------------------------------- | --------------- | ------- | ------------ | ---------------- | ----------- | ------------------------------------------------- | ---- |
| **DesignCore 16-ch GMSL2 Interface Card** | D3 Engineering  | 16      | GMSL2        | 하드웨어 (FPGA)  | $799        | AGX Orin Dev Kit과 직접 인터페이스                |      |
| **DesignCore 16-ch FPD-Link III Card**    | D3 Engineering  | 16      | FPD-Link III | 하드웨어 (FPGA)  | $799        | 16개 FPD-Link III 입력 채널                       |      |
| **LI-JAG-ADP-GMSL2-8CH**                  | Leopard Imaging | 8       | GMSL2        | 지원             | $557        | AGX Orin Dev Kit용 8채널 보드                     |      |
| **STURDeCAM25_CUOAGX Kit**                | e-con Systems   | 8       | GMSL2        | 외부 트리거 지원 | $549 (키트) | 8개 카메라 커넥터가 있는 디시리얼라이저 보드 포함 |      |
| 데이터 출처: 4                            |                 |         |              |                  |             |                                                   |      |


로보틱스 애플리케이션에서 데이터의 일관성을 보장하기 위해서는 정밀한 프레임 동기화가 필수적입니다.

- **메커니즘**: 대부분의 다채널 디시리얼라이저 허브는 마스터 프레임 동기화 신호(FSYNC)를 생성하거나 분배하는 FPGA 또는 마이크로컨트롤러를 내장하고 있습니다.2 이 전기적 펄스는 디시리얼라이저의 GPO(General-Purpose Output) 핀에서 생성되어 동축 케이블의 역방향 채널을 통해 전송되고, 각 카메라의 시리얼라이저에 있는 GPI(General-Purpose Input) 핀에서 수신됩니다.2 이 신호는 모든 카메라 센서가 정확히 동일한 순간에 노출을 시작하도록 명령합니다.
- **트리거링 모드**: 시스템은 다양한 동기화 모드로 구성될 수 있습니다.
  - **프리 런(Free Run)**: 카메라가 독립적으로(비동기적으로) 스트리밍합니다. 이 애플리케이션에는 적합하지 않습니다.
  - **내부 트리거(Auto Trigger)**: 디시리얼라이저 허브의 온보드 클럭이 설정된 프레임 속도(예: 30Hz)로 주기적인 트리거 신호를 생성합니다.12
  - **외부 트리거(Manual Trigger)**: 시스템은 GPS의 PPS(Pulse-Per-Second) 신호나 IMU의 트리거 신호와 같은 외부 신호를 받아 카메라를 다른 센서와 동기화할 수 있습니다.12 이는 가장 진보되고 유연한 방식입니다.

하드웨어 동기화(동시 캡처)와 소프트웨어 동기화(캡처 후 타임스탬프 기반 정렬)를 명확히 구분하는 것이 중요합니다. 하드웨어 동기화는 선제적이고 정밀하지만, 소프트웨어 동기화는 사후 대응적이며 오류와 지터(jitter)에 취약합니다.3 따라서 이 애플리케이션에서는 하드웨어 동기화가 반드시 필요합니다.



성공적인 다중 카메라 시스템 구현은 하드웨어만큼이나 소프트웨어에 크게 의존합니다.

- **디바이스 트리(Device Tree) 구성**: 이는 가장 중요하고 첫 번째로 수행해야 할 소프트웨어 단계입니다. Jetson의 리눅스 커널은 I/O 핀에 연결된 하드웨어를 인식해야 합니다. 이를 위해 디바이스 트리 소스(`.dts` 파일)를 수정하여 디시리얼라이저 허브, 시리얼라이저, 그리고 카메라 센서 자체의 I2C 주소를 정의해야 합니다. 또한 MIPI CSI-2 포트를 구성하고, 디시리얼라이저에서 오는 가상 채널 ID를 Jetson의 비디오 입력(VI) 장치에 매핑하는 작업이 포함됩니다.20 이는 일반적으로 디시리얼라이저 허브 공급업체(예: D3, Leopard)가 제공하는 지원 및 문서에 크게 의존하는 복잡한 작업입니다.
- **V4L2 vs. libargus API**: 이는 지연 시간과 직결되는 중요한 성능 결정 사항입니다.
  - **libargus**: NVIDIA의 고수준 "카메라와 유사한" API입니다. 자동 노출, 화이트 밸런스, 노이즈 제거와 같은 ISP(Image Signal Processor) 기능을 자동 제어합니다. 하지만 이 처리 파이프라인은 상당한 지연 시간(일부 경우 40-60ms로 측정됨)을 추가하며, '블랙박스'처럼 작동하여 제어력이 떨어집니다.33
  - **V4L2 (Video4Linux2)**: 비디오 캡처를 위한 표준 리눅스 커널 API입니다. V4L2를 사용하여 RAW Bayer 데이터를 요청하면, 애플리케이션은 ISP와 그에 따른 지연 시간을 우회할 수 있습니다. RAW 프레임은 메모리로 직접 전달되어 CUDA를 사용하는 GPU의 맞춤형 알고리즘으로 처리될 수 있습니다. 가능한 가장 낮은 지연 시간을 위해서는 V4L2가 월등한 선택입니다.33 지연 시간 차이는 상당할 수 있으며, 지연 시간에 민감한 모든 애플리케이션에 V4L2가 권장됩니다.33
- **GStreamer & DeepStream SDK**: 프레임이 캡처되면(이상적으로는 V4L2를 통해), GStreamer는 처리 파이프라인을 구축하기 위한 최적의 프레임워크입니다. NVIDIA의 DeepStream SDK는 GStreamer 위에 구축되어 추론(`nvinfer`), 객체 추적(`nvtracker`) 등을 위한 하드웨어 가속 플러그인을 제공하여, 매우 효율적인 엔드-투-엔드 비전 AI 애플리케이션을 만들 수 있게 해줍니다.38


시스템 설계의 타당성을 검증하기 위해 간단한 대역폭 계산을 수행할 수 있습니다. 예를 들어, 10대의 카메라가 각각 1920x1080 해상도, 30fps, 12비트/픽셀(RAW12)로 작동한다고 가정하면, 총 필요한 대역폭은 약 7.5 Gbps (10×1920×1080×30×12)입니다. Jetson AGX Orin의 MIPI 인터페이스는 최대 40 Gbps(D-PHY) 또는 164 Gbps(C-PHY)를 지원하며 15, 메모리 대역폭은 204.8 GB/s에 달하므로 18, 하드웨어는 충분한 대역폭을 가지고 있음을 확인할 수 있습니다.

여기서 반드시 고려해야 할 중요한 소프트웨어적 제약 사항이 있습니다. 2025년 초 NVIDIA 개발자 포럼의 여러 스레드에 따르면, JetPack 6(L4T 36.x)에서는 소프트웨어 수준에서 8개 이상의 동시 NVDEC(하드웨어 비디오 디코더) 세션을 실행할 수 없는 제한이 보고되었습니다.41 이는 AGX Orin 하드웨어가 데이터시트 상 20개 이상의 FHD 스트림을 처리할 수 있음에도 불구하고 발생하는 문제입니다. 이 문제는 20개 이상의 스트림을 실행할 수 있었던 JetPack 5에 비해 성능이 저하된 것으로 보입니다.41

이 소프트웨어 제한은 10채널 카메라 시스템 아키텍처에 중대한 영향을 미칩니다. 만약 RTSP 카메라처럼 하드웨어 디코딩에 의존하는 아키텍처를 채택한다면, 9번째 스트림부터 시스템이 실패하게 될 것입니다. 따라서 이 문제는 V4L2를 통해 RAW 데이터를 캡처하는 파이프라인을 사용해야 하는 강력한 근거가 됩니다. RAW 데이터 파이프라인은 스트림 수신 단계에서 NVDEC를 완전히 우회하므로, 8개 스트림 제한의 영향을 받지 않습니다. 결과적으로, V4L2를 사용하는 것은 단순히 '저지연'을 위한 최적화를 넘어, JetPack 6 환경에서 10채널 시스템을 기능적으로 구현하기 위한 필수 조건이 됩니다. 개발자는 이 잠재적인 문제를 인지하고 시스템을 설계해야 합니다.



다음 표는 제안된 아키텍처를 기반으로 완전한 시스템을 구축하는 데 필요한 구체적인 부품 목록과 예상 비용을 제공합니다. 이는 개념적인 가이드를 실행 가능한 계획으로 전환하는 데 도움이 될 것입니다.

**표 5: 10채널 GMSL2 시스템을 위한 샘플 자재 명세서 (BOM)**

| 구성 요소                                                    | 권장 제품                                | 공급업체        | 수량 | 예상 단가 (USD) | 예상 총액 (USD) |      |
| ------------------------------------------------------------ | ---------------------------------------- | --------------- | ---- | --------------- | --------------- | ---- |
| **컴퓨트 모듈**                                              | NVIDIA Jetson AGX Orin 64GB 개발자 키트  | NVIDIA          | 1    | $1,999          | $1,999          |      |
| **디시리얼라이저 허브**                                      | DesignCore 16-ch GMSL2 Interface Card    | D3 Engineering  | 1    | $799            | $799            |      |
| **카메라**                                                   | D3CM-AR0234 GMSL2 Camera (또는 동급)     | D3 Engineering  | 10   | $599            | $5,990          |      |
| **케이블**                                                   | 15m Coaxial Cable with FAKRA connectors  | D3/Leopard      | 10   | $50             | $500            |      |
| **전원 공급 장치**                                           | 12V/5A Power Supply for Deserializer Hub | D3/Leopard      | 1    | $35             | $35             |      |
| **저장 장치**                                                | 1TB NVMe SSD                             | Samsung/Crucial | 1    | $100            | $100            |      |
| **총 예상 비용**                                             |                                          |                 |      |                 | **$9,423**      |      |
| *가격은 샘플 및 소매가를 기준으로 하며, 실제 구매 시 변동될 수 있습니다.* |                                          |                 |      |                 |                 |      |
| 데이터 출처: 16                                              |                                          |                 |      |                 |                 |      |


복잡한 시스템의 통합을 위해 단계적인 접근 방식이 권장됩니다.

1. **1단계: 하드웨어 조립**: 디시리얼라이저 허브를 Jetson AGX Orin의 카메라 커넥터에 물리적으로 연결합니다. 먼저 단일 카메라를 허브에 연결합니다.
2. **2단계: 소프트웨어 및 드라이버 통합**: Jetson에 최신 JetPack을 플래시합니다. 허브 및 카메라 공급업체에서 제공하는 드라이버와 디바이스 트리 오버레이를 설치합니다. 이 단계가 프로젝트에서 가장 중요하고 어려울 수 있는 부분입니다.
3. **3단계: 단일 카메라 검증**: V4L2 또는 GStreamer 명령어를 사용하여 단일 카메라에서 비디오 스트림을 성공적으로 캡처할 수 있는지 확인합니다.
4. **4단계: 확장 및 동기화**: 나머지 9대의 카메라를 연결합니다. 공급업체의 소프트웨어 도구를 사용하여 하드웨어 프레임 동기화를 구성하고 검증합니다.
5. **5단계: 애플리케이션 개발**: 10개의 모든 스트림을 처리하기 위한 최종 GStreamer/DeepStream 파이프라인을 구축하고 최적화합니다.


본 보고서에서 제안한 10채널 저지연 동기화 카메라 시스템은 올바른 아키텍처와 구성 요소를 선택한다면 충분히 구현 가능합니다. 프로젝트 성공의 가장 중요한 단일 요소는 D3 Engineering과 같이 우수한 소프트웨어 지원(잘 문서화된 드라이버, 디바이스 트리 오버레이, 기술 지원 포함)을 제공하는 디시리얼라이저 허브 공급업체를 선택하는 것입니다. 이 프로젝트의 복잡성은 하드웨어 조립보다는 주로 소프트웨어 통합에 있습니다. 특히, JetPack 6의 잠재적인 NVDEC 8-스트림 제한은 RAW V4L2 파이프라인을 사용하여 설계 단계에서 반드시 회피해야 할 주요 위험 요소입니다. 이러한 고려 사항들을 염두에 두고 체계적으로 접근한다면, 로보틱스, 자율 시스템, 첨단 감시 등 다양한 분야에서 혁신을 이끌어낼 강력한 비전 시스템을 구축할 수 있을 것입니다.


1. USB/CSI-2 VS RTSP Cam For The Nvidia Jetson Project | SAVANT, accessed July 3, 2025, https://b.savant-ai.io/2024/11/07/usb-csi-2-vs-rtsp-cam-for-the-nvidia-jetson-project/
2. What is frame sync, and why is it required in GMSL applications? - Documents - Interface and Isolation - EngineerZone, accessed July 3, 2025, https://ez.analog.com/interface-isolation/w/documents/27777/what-is-frame-sync-and-why-is-it-required-in-gmsl-applications
3. Keeping camera synchronization at software level - Jetson AGX Xavier, accessed July 3, 2025, https://forums.developer.nvidia.com/t/keeping-camera-synchronization-at-software-level/111355
4. 16 Synchronized Cameras On NVIDIA Jetson AGX Orin - D3 Embedded, accessed July 3, 2025, https://www.d3embedded.com/solutions/16-cameras-running-simultaneously-on-nvidia-jetson/
5. Edge AI Using Cameras, 3D Time-of-Flight, and NVIDIA Jetson | D3 Embedded, accessed July 3, 2025, https://www.d3embedded.com/solutions/artificial-intelligence-system-using-2d-3d-mipi-csi-2-cameras-and-nvidia-jetson-modules/
6. Multiple MIPI CSI-2 Cameras for NVIDIA Jetson AGX Xavier/TX2, accessed July 3, 2025, https://www.e-consystems.com/multiple-csi-cameras-for-nvidia-jetson-tx2.asp
7. Machine vision camera types overview - MRTech SK, accessed July 3, 2025, https://mr-technologies.com/iff-sdk/cameras/
8. USB, GMSL and MIPI: Choosing the Right Camera Interface | Simms International, accessed July 3, 2025, https://www.simms.co.uk/tech-talk/usb-gmsl-and-mipi-choosing-the-right-camera-interface/
9. GMSL™ Provides Robust Sensor Links For Nvidia's Jetson Platform ..., accessed July 3, 2025, https://www.analog.com/en/lp/forms/gmsl-form.html
10. Jetson Virtual Channel with GMSL Camera Framework Guide | NVIDIA Developer, accessed July 3, 2025, https://developer.nvidia.com/embedded/secure/jetson/jetson_gmsl_camera_framework_guide
11. GMSL vs. MIPI cameras: why are GMSL cameras better? - Sinoseen, accessed July 3, 2025, https://www.sinoseen.com/gmsl-vs-mipi-cameras-why-are-gmsl-cameras-better
12. Frame Synchronization - roscube-doc 0.9.1 documentation - GitHub Pages, accessed July 3, 2025, https://adlink-ros.github.io/roscube-doc/roscube-x/gmsl_camera/frame_sync.html
13. 24 MIPI CSI-2 Cameras On A Single Jetson Using Multiplexer - NVIDIA Developer Forums, accessed July 3, 2025, https://forums.developer.nvidia.com/t/24-mipi-csi-2-cameras-on-a-single-jetson-using-multiplexer/82488
14. 7 Types of MIPI CSI 2 Converters & Bridges: Turning MIPI-CSI 2 Cameras into Other Interfaces - Arducam, accessed July 3, 2025, https://archive.arducam.com/mipi-csi-2-converters-bridges-other-interfaces/
15. Our NVIDIA Jetson Orin and NVIDIA Jetson Xavier comparison ..., accessed July 3, 2025, https://www.generationrobots.com/blog/en/our-nvidia-jetson-orin-and-nvidia-jetson-xavier-comparison/
16. NVIDIA Jetson Nano, TX2, Xavier and Orin Comparison and FAQ - BVM Ltd, accessed July 3, 2025, https://www.bvm.co.uk/edge-ai-computing/nvidia-jetson-comparison-and-faq/
17. NVIDIA Jetson Orin vs. other NVIDIA Jetson modules – a detailed ..., accessed July 3, 2025, https://www.e-consystems.com/blog/camera/technology/nvidia-jetson-orin-vs-other-nvidia-jetson-modules-a-detailed-look/
18. NVIDIA® Jetson Comparison: Nano, TX2 NX, Xavier NX, AGX, Orin - Latest News from Seeed Studio, accessed July 3, 2025, https://www.seeedstudio.com/blog/nvidia-jetson-comparison-nano-tx2-nx-xavier-nx-agx-orin/
19. Jetson Orin Nano Developer Kit 4GB/8GB MIPI CSI-2 capabilities, SD card, accessed July 3, 2025, https://forums.developer.nvidia.com/t/jetson-orin-nano-developer-kit-4gb-8gb-mipi-csi-2-capabilities-sd-card/249627
20. Jetson Virtual Channel with GMSL Camera Framework - NVIDIA Docs, accessed July 3, 2025, https://docs.nvidia.com/jetson/archives/r35.4.1/DeveloperGuide/text/SD/CameraDevelopment/JetsonVirtualChannelWithGmslCameraFramework.html
21. MAX96716A Dual GMSL2 to CSI-2 Deserializer - Analog Devices, accessed July 3, 2025, https://www.analog.com/media/en/technical-documentation/data-sheets/max96716a.pdf
22. GMSL2 cameras vs. FPD-Link III cameras – a detailed study - Electronic Specifier, accessed July 3, 2025, https://www.electronicspecifier.com/products/sensors/gmsl2-cameras-vs-fpd-link-iii-cameras-a-detailed-study
23. 16 Synchronized Cameras On NVIDIA Jetson AGX Orin | D3 ..., accessed July 3, 2025, https://www.d3engineering.com/solutions/16-cameras-running-simultaneously-on-nvidia-jetson/
24. DesignCore® NVIDIA® Jetson AGX Orin GMSL2 Interface Card | D3 Embedded, accessed July 3, 2025, https://www.d3embedded.com/product/designcore-nvidia-jetson-agx-orin-gmsl2-interface-card/
25. DesignCore® NVIDIA® Jetson AGX Orin FPD-Link™ III Interface Card | D3 Embedded, accessed July 3, 2025, https://www.d3embedded.com/product/designcore-nvidia-jetson-agx-orin-fpd-link-iii-interface-card/
26. Boards & Cards | D3 Embedded, accessed July 3, 2025, https://www.d3embedded.com/product-category/boards-cards/
27. LI-AR0234CS-ST75-GM2C-120H - Leopard Imaging Inc., accessed July 3, 2025, https://leopardimaging.com/product/automotive-cameras/cameras-by-interface/maxim-gmsl-2-cameras/li-ar0234cs-stereo-gmsl2/li-ar0234cs-st75-gm2c-120h/
28. LI-JAG-ADP-GMSL2-8CH - Leopard Imaging Inc., accessed July 3, 2025, https://leopardimaging.com/product/accessories/adapters-carrier-boards/for-nvidia-jetson/li-jag-adp-gmsl2-8ch/
29. 6 MIPI CSI-2 Cameras support for NVIDIA® Jetson TX2/ TX1 - e-con Systems, accessed July 3, 2025, https://www.e-consystems.com/blog/camera/products/6-mipi-csi-2-cameras-support-for-jetson-tx1/
30. STURDeCAM25 - IP67 Full HD GMSL2™ Global Shutter Camera Module - e-con Systems, accessed July 3, 2025, https://www.e-consystems.com/camera-modules/gmsl2-ar0234-global-shutter-camera.asp
31. PCIe-GL26: NVIDIA Jetson Xavier NX AI-Enabled 6-Port GMSL2 Camera Frame Grabber Card - CoastIPC, accessed July 3, 2025, https://coastipc.com/pcie-gl26-nvidia-jetson-xavier-nx-ai-enabled-6-port-gmsl2-camera-frame-grabber-card
32. GMSL camera integration - Jetson Orin Nano - NVIDIA Developer Forums, accessed July 3, 2025, https://forums.developer.nvidia.com/t/gmsl-camera-integration/329757
33. Lib Argus Latency - Jetson Orin NX - NVIDIA Developer Forums, accessed July 3, 2025, https://forums.developer.nvidia.com/t/lib-argus-latency/334757
34. NVIDIA Jetson TX2: Argus vs V4L2 Latency Comparison - RidgeRun Developer Wiki, accessed July 3, 2025, https://developer.ridgerun.com/wiki/index.php/NVIDIA_Jetson_TX2_-_Argus_vs_V4L2_Latency_Analysis
35. Image processing on Jetson: Libargus and Fastvideo SDK - fastcompression.com, accessed July 3, 2025, https://www.fastcompression.com/blog/jetson-image-processing.htm
36. Latency Analysis with LibArgus - Jetson AGX Xavier - NVIDIA Developer Forums, accessed July 3, 2025, https://forums.developer.nvidia.com/t/latency-analysis-with-libargus/203673
37. Achieve minimum latency with CSI camera - Jetson TX2 - NVIDIA Developer Forums, accessed July 3, 2025, https://forums.developer.nvidia.com/t/achieve-minimum-latency-with-csi-camera/161122
38. NVIDIA DeepStream SDK, accessed July 3, 2025, https://developer.nvidia.com/deepstream-sdk
39. Performance - DeepStream documentation - NVIDIA Docs, accessed July 3, 2025, https://docs.nvidia.com/metropolis/deepstream/dev-guide/text/DS_Performance.html
40. Managing Video Streams in Runtime with the NVIDIA DeepStream SDK, accessed July 3, 2025, https://developer.nvidia.com/blog/managing-video-streams-in-runtime-with-the-deepstream-sdk/
41. Orin AGX Jetpack 6 NVDEC limitation of <= 8 streams - NVIDIA Developer Forums, accessed July 3, 2025, https://forums.developer.nvidia.com/t/orin-agx-jetpack-6-nvdec-limitation-of-8-streams/320865
42. Orin AGX Jetpack 6 NVDEC limitation - NVIDIA Developer Forums, accessed July 3, 2025, https://forums.developer.nvidia.com/t/orin-agx-jetpack-6-nvdec-limitation/337237

