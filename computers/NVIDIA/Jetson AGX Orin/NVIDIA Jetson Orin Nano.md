# NVIDIA Jetson Orin Nano
[NVIDIA Jetson AGX Orin](./index.md)



현대 인공지능(AI) 기술은 클라우드 중심의 대규모 연산에서 벗어나, 데이터가 생성되는 지점인 '엣지(Edge)'에서 실시간으로 정보를 처리하는 방향으로 진화하고 있다. 엣지 AI 컴퓨팅은 데이터 전송 지연 시간을 최소화하고, 네트워크 대역폭 의존도를 낮추며, 개인정보 보호를 강화하는 등 다양한 이점을 제공한다.1 이러한 패러다임 전환의 중심에는 고성능, 저전력 임베디드 컴퓨팅 플랫폼이 있으며, NVIDIA Jetson은 이 분야에서 사실상의 산업 표준으로 자리매김해왔다.2 Jetson 플랫폼은 컴팩트한 폼팩터에 강력한 GPU 가속 성능을 집약하여 로보틱스, 지능형 영상 분석(IVA), 자율주행차 등 복잡한 AI 워크로드를 엣지 디바이스에서 구현하는 것을 가능하게 했다.4


2019년, NVIDIA는 합리적인 가격대의 Jetson Nano를 출시하며 메이커, 학생, 개발자 커뮤니티에 AI 기술의 대중화를 이끌었다.4 이 성공을 기반으로, 시장은 더욱 복잡하고 정교한 차세대 AI 모델, 특히 트랜스포머(Transformer) 아키텍처에 기반한 거대 언어 모델(LLM)과 비전 모델을 엣지 환경에서 구동할 수 있는 강력한 성능을 요구하기 시작했다. 이러한 시장의 요구에 부응하여 등장한 것이 바로 Jetson Orin Nano이다.6 Jetson Orin Nano의 출시는 단순히 이전 세대 대비 성능을 향상시킨 것을 넘어, 엣지 디바이스에서 구현 가능한 애플리케이션의 범주를 생성형 AI 영역까지 근본적으로 확장시켰다는 점에서 중대한 의의를 가진다.


Jetson Orin Nano의 가치를 극적으로 끌어올린 결정적 계기는 '슈퍼(Super)' 소프트웨어 업데이트였다. NVIDIA는 JetPack SDK 업데이트를 통해 기존 하드웨어의 AI 연산 성능을 40 TOPS(Trillions of Operations Per Second)에서 67 TOPS로 약 1.7배 향상시켰다.8 이 업데이트는 새로운 하드웨어 구매 없이 기존 사용자들에게도 동일하게 적용되어, 제품의 수명 가치를 높이고 고객 충성도를 강화하는 효과를 낳았다.8

이러한 접근 방식은 하드웨어의 잠재 성능이 출시 초기에는 소프트웨어적으로 제한되었을 가능성을 시사한다. 초기 40 TOPS 성능은 상위 모델인 Jetson Orin NX와의 시장 구분을 명확히 하고, 낮은 전력 범위 내에서 안정적인 발열 관리를 보장하기 위한 전략적 선택이었을 수 있다. 이후 소프트웨어 업데이트를 통해 25W 전력 모드를 활성화하고 잠재된 성능을 개방함으로써 11, NVIDIA는 경쟁 제품 대비 압도적인 가격 대비 성능 우위를 확보했다.13 이는 하드웨어를 정적인 제품이 아닌, 소프트웨어를 통해 지속적으로 가치가 향상되는 '플랫폼'으로 정의하려는 NVIDIA의 전략을 명확히 보여주는 사례이다. 즉, '소프트웨어 정의 성능(Software-Defined Performance)'은 Jetson 생태계의 핵심적인 경쟁력으로 작용한다.



Jetson Orin Nano의 핵심은 신용카드보다 작은 크기(69.6mm x 45mm)의 시스템 온 모듈(SoM)에 집약된 고성능 컴퓨팅 유닛이다.5


GPU는 NVIDIA의 최신 Ampere 아키텍처를 기반으로 설계되었다. 8GB 모델은 1024개의 CUDA 코어와 32개의 3세대 Tensor 코어를 탑재하고 있으며, 4GB 모델은 512개의 CUDA 코어와 16개의 Tensor 코어를 갖추고 있다.6 이는 128개의 CUDA 코어를 탑재했던 이전 세대 Maxwell 아키텍처 기반 Jetson Nano와 비교했을 때, 아키텍처의 혁신과 코어 수의 대폭적인 증가를 통해 비약적인 성능 향상을 이루었음을 의미한다.4 CUDA 코어는 범용 병렬 연산을, Tensor 코어는 AI 추론의 핵심인 행렬 연산을 하드웨어적으로 가속하여 딥러닝 워크로드에 최적화된 성능을 제공한다.13


6코어 Arm Cortex-A78AE v8.2 64비트 CPU가 탑재되어 있으며, 1.5MB L2 캐시와 4MB L3 캐시를 공유한다.6 이 CPU는 AI 모델의 추론 전후 데이터 처리, 운영체제 및 복잡한 애플리케이션 파이프라인의 제어 등 순차적인 작업을 효율적으로 수행하며 GPU와 협력하여 전체 시스템 성능을 극대화한다.


메모리 시스템은 8GB 128-bit LPDDR5를 채택하여 기본 68 GB/s의 높은 대역폭을 제공한다.6 특히 '슈퍼' 모드에서는 이 대역폭이 102 GB/s까지 확장되어 8, LLM과 같은 대규모 모델의 가중치를 메모리에 빠르게 로드하고, 고해상도 센서 데이터를 지연 없이 처리하는 데 있어 병목 현상을 최소화한다. 이는 이전 세대 LPDDR4 대비 두 배 이상의 대역폭 향상이다.16


Jetson Orin Nano 모듈은 애플리케이션의 요구사항에 따라 전력 소비를 유연하게 조절할 수 있다. 기본적으로 7W에서 15W 사이의 전력 프로파일을 제공하며, '슈퍼' 모드를 통해 최대 성능을 요구하는 경우 최대 25W까지 전력을 사용하도록 구성할 수 있다.11 이러한 가변적인 전력 설계는 배터리로 구동되는 로봇이나 드론과 같이 전력 제약이 심한 환경부터, 상시 전원이 공급되는 고정형 비전 시스템에 이르기까지 광범위한 엣지 AI 애플리케이션에 적용될 수 있는 유연성을 부여한다.


NVIDIA Jetson Orin Nano 개발자 키트는 SoM과 함께 다양한 I/O 인터페이스를 제공하는 레퍼런스 캐리어 보드를 포함하여 즉각적인 프로토타이핑을 지원한다.6


개발자 키트는 풍부한 최신 인터페이스를 제공한다. 4개의 USB 3.2 Gen2 Type-A 포트는 최대 10Gbps의 빠른 속도로 외부 장치와 데이터를 교환할 수 있게 하며, Gigabit Ethernet 포트는 안정적인 유선 네트워크 연결을 보장한다.9 USB Type-C 포트는 데이터 전송 및 디바이스 모드, 복구 모드를 지원하지만, 디스플레이 출력 기능은 제공하지 않는다.2 유일한 비디오 출력은 DisplayPort 1.2 커넥터를 통해 이루어지므로, HDMI 모니터 연결 시에는 별도의 어댑터가 필요하다.2


기본적인 운영체제 부팅과 데이터 저장을 위해 microSD 카드 슬롯이 제공된다.18 하지만 이는 초기 설정 및 가벼운 작업을 위한 옵션이며, Jetson Orin Nano의 진정한 성능을 활용하기 위해서는 NVMe SSD 사용이 필수적이다. 캐리어 보드에는 2개의 M.2 Key M 슬롯(각각 PCIe 3.0 x4, PCIe 3.0 x2 레인 지원)이 탑재되어 있어 고속 NVMe SSD를 장착할 수 있다.6

이러한 스토리지 구성은 NVIDIA의 의도적인 설계 철학을 반영한다. microSD 카드는 라즈베리 파이와 같이 손쉬운 초기 설정을 제공하여 진입 장벽을 낮추는 역할을 한다.20 그러나 LLM과 같은 수십 기가바이트에 달하는 AI 모델과 대용량 데이터셋을 다루는 실제 개발 환경에서 microSD 카드는 심각한 성능 병목을 유발한다.21 따라서 NVMe SSD는 선택이 아닌 사실상의 필수 요건이다. 이는 개발자 키트의 초기 구매 비용($249) 외에 추가적인 지출이 필요함을 의미한다. NVIDIA는 고성능 스토리지를 사용자 선택과 시장의 범용 부품에 맡김으로써, 모듈 자체의 가격 경쟁력을 유지하고 사용자에게 스토리지 용량과 성능 선택의 유연성을 제공하는 전략을 취하고 있다. 이는 eMMC를 내장하는 방식과 대조적으로, 향후 급증할 스토리지 요구에 효과적으로 대응하고, SSD를 고속 스왑 메모리처럼 활용하는 고급 기법까지 가능하게 한다.21


2개의 22핀 MIPI CSI-2 카메라 커넥터를 통해 고해상도 카메라 모듈을 직접 연결할 수 있으며, 4레인까지 지원하여 다중 카메라 시스템 구축이 용이하다.6 또한, 라즈베리 파이와 유사한 핀 배열을 가진 40핀 확장 헤더는 GPIO, I2C, SPI, UART 등의 프로토콜을 지원하여 다양한 센서, 모터 컨트롤러, 디스플레이 등 외부 하드웨어와의 손쉬운 통합을 가능하게 한다.9

| 기능 (Feature)        | Jetson Orin Nano 8GB 모듈 (Module)                     | Jetson Orin Nano 개발자 키트 (Developer Kit)                 |
| --------------------- | ------------------------------------------------------ | ------------------------------------------------------------ |
| **GPU**               | NVIDIA Ampere 아키텍처, 1024 CUDA 코어, 32 Tensor 코어 | 모듈 사양과 동일                                             |
| **CPU**               | 6코어 Arm Cortex-A78AE v8.2 (1.5MB L2 + 4MB L3)        | 모듈 사양과 동일                                             |
| **AI 성능**           | 최대 67 TOPS (INT8, 슈퍼 모드)                         | 모듈 사양과 동일                                             |
| **메모리**            | 8GB 128-bit LPDDR5, 102 GB/s (슈퍼 모드)               | 모듈 사양과 동일                                             |
| **스토리지**          | 외부 NVMe 지원 (모듈 자체 저장 공간 없음)              | microSD 카드 슬롯, 2x M.2 Key M (NVMe용)                     |
| **비디오 인코딩**     | 1080p30 (CPU 코어 1-2개 사용)                          | 모듈 사양과 동일                                             |
| **비디오 디코딩**     | 1x 4K60, 2x 4K30, 5x 1080p60 (H.265)                   | 모듈 사양과 동일                                             |
| **카메라 인터페이스** | 8 MIPI CSI-2 레인                                      | 2x 22핀 MIPI CSI-2 커넥터                                    |
| **PCIe**              | 1x4 + 3x1 (PCIe Gen3)                                  | 1x M.2 Key M (PCIe 3.0 x4), 1x M.2 Key M (PCIe 3.0 x2), 1x M.2 Key E |
| **USB**               | 3x USB 3.2 Gen2, 3x USB 2.0                            | 4x USB 3.2 Gen2 Type-A, 1x USB Type-C (데이터 전용)          |
| **네트워킹**          | 1x GbE                                                 | 1x GbE 커넥터, Wi-Fi/BT 모듈 (M.2 Key E 슬롯에 사전 설치)    |
| **디스플레이**        | 1x 4K30 multi-mode DP 1.2/eDP 1.4/HDMI 1.4             | 1x DisplayPort 1.2 (+MST) 커넥터                             |
| **기타 I/O**          | UART, SPI, I2S, I2C, CAN, GPIOs                        | 40핀 확장 헤더, 12핀 버튼 헤더, 4핀 팬 헤더                  |
| **전력**              | 7W – 25W (모듈 전력)                                   | 19V DC 전원 잭 (45W 어댑터 포함)                             |
| **크기**              | 69.6mm x 45mm (SoM)                                    | 100mm x 79mm x 21mm (전체 키트)                              |



Jetson Orin Nano의 AI 성능은 최대 67 TOPS로 명시되어 있다.8 이 수치는 1초에 67조 번의 연산을 수행할 수 있음을 의미하며, 주로 8비트 정수(INT8) 연산 정밀도와 가중치 희소성(Sparsity)을 활용했을 때의 이론적 최대 성능을 나타낸다.15 AI 모델 추론 과정에서 부동소수점(Floating-Point) 가중치를 정수(Integer)로 양자화(Quantization)하면 연산 속도를 크게 높이고 메모리 사용량을 줄일 수 있다. Tensor 코어는 이러한 INT8 행렬 연산을 하드웨어적으로 가속하여 높은 TOPS를 달성하는 데 핵심적인 역할을 한다. 하지만 모든 모델이나 연산이 INT8 정밀도에 적합한 것은 아니며, 32비트 부동소수점(FP32) 정밀도가 요구되는 과학 계산이나 특정 AI 모델의 초기 레이어에서는 성능 이득이 달라질 수 있음을 이해해야 한다.16


Jetson Orin Nano의 가장 주목할 만한 성능 향상은 생성형 AI 분야에서 나타난다. '슈퍼' 모드를 활성화하고 25W 전력 모드를 사용했을 때, 다양한 최신 생성형 AI 모델에서 괄목할 만한 성능을 보인다. NVIDIA가 공개한 벤치마크에 따르면, 주요 거대 언어 모델(LLM)인 Llama 3.1 8B 모델은 초당 14 토큰에서 19.14 토큰으로, 시각 언어 모델(VLM)인 VILA 1.5 3B는 초당 0.7 이미지에서 1.06 이미지로 처리량이 증가했다.25 이는 각각 37%, 51%의 성능 향상에 해당한다.

이러한 성능은 엣지 디바이스에서 실시간 대화형 AI 챗봇, 이미지에 대한 자연어 질의응답, 실시간 이미지 캡셔닝과 같은 이전에는 불가능했던 고급 애플리케이션을 구현할 수 있는 수준이다.8 독립적인 성능 측정에서도 15W 모드 대비 25W '슈퍼' 모드에서 텍스트 생성 속도(tokens/s)는 약 31%, 이미지 생성 속도(iterations/s)는 약 34% 향상되는 결과가 관찰되었다.11 이는 단순히 이론적 수치를 넘어 실제 애플리케이션 구동 시 체감할 수 있는 상당한 성능 개선임을 입증한다.


전통적인 컴퓨터 비전 분야에서도 Jetson Orin Nano는 압도적인 성능을 자랑한다. NVIDIA의 TAO Toolkit으로 최적화된 PeopleNet(사람 탐지), DashCamNet(차량 및 표지판 탐지)과 같은 모델은 물론, ResNet-50과 같은 표준 이미지 분류 벤치마크에서 이전 세대 Jetson Nano 대비 최대 80배의 성능 향상을 기록했다.6

특히 최신 객체 탐지 모델인 Ultralytics YOLOv11을 이용한 벤치마크 결과는 매우 인상적이다. Jetson Orin Nano Super는 상위 제품군인 Jetson Orin NX 16GB(슈퍼 모드 미적용)와 필적하거나 때로는 능가하는 추론 속도를 보여주었다.13 이는 Orin Nano Super가 절반 이하의 가격으로 매우 높은 수준의 실시간 객체 탐지 성능을 제공함을 의미하며, 가격 대비 성능 측면에서 탁월한 가치를 지니고 있음을 증명한다.

모델의 정밀도에 따른 성능 변화를 분석해 보면, FP32에서 FP16(16비트 부동소수점)으로 정밀도를 낮추었을 때 약 30%에서 50% 사이의 뚜렷한 성능 향상이 관찰된다. 반면, FP16에서 INT8로 양자화했을 때의 성능 향상은 5%에서 15% 수준으로 상대적으로 낮게 나타났다. 이러한 현상은 TensorRT가 모델을 변환하는 과정에서 일부 레이어가 INT8을 완벽하게 지원하지 않아 FP16으로 유지되었을 가능성을 시사한다.4 그럼에도 불구하고, 대부분의 비전 AI 애플리케이션에서 FP16 정밀도는 정확도 손실 없이 상당한 성능 이점을 제공하므로 매우 효과적인 최적화 전략이다.

| 모델 유형              | 모델 이름      | Orin Nano (15W) | Orin Nano Super (25W) | 성능 향상률 (%)  |
| ---------------------- | -------------- | --------------- | --------------------- | ---------------- |
| **LLM**                | Llama 3.1 8B   | 14.0 tokens/s   | 19.14 tokens/s        | 36.7%            |
|                        | Phi 3.5 3B     | 24.7 tokens/s   | 38.1 tokens/s         | 54.3%            |
| **VLM**                | LLAVA 1.6 7B   | 0.412 images/s  | 0.57 images/s         | 38.3%            |
|                        | InternVL2.5 4B | 2.5 images/s    | 5.1 images/s          | 104.0%           |
| **Vision Transformer** | NanoOWL        | -               | -                     | 상당한 성능 향상 |
| **Object Detection**   | YOLOv11        | -               | Orin NX 16GB와 필적   | -                |


Jetson Orin Nano의 강력한 하드웨어 성능은 NVIDIA가 제공하는 포괄적인 소프트웨어 개발 키트(SDK)인 JetPack을 통해 온전히 발휘된다. JetPack은 단순한 운영체제를 넘어, 엣지 AI 애플리케이션 개발에 필요한 모든 요소를 통합한 솔루션이다.26


JetPack SDK는 크게 세 가지 핵심 요소로 구성된다.27

- **Jetson Linux:** 이는 Jetson 하드웨어를 위한 보드 지원 패키지(BSP)로, 부트로더, 최적화된 Linux 커널, Ubuntu 데스크톱 환경, 그리고 하드웨어 드라이버와 툴체인을 포함한다. 안정적인 시스템 운영의 기반이 된다.
- **Jetson AI Stack:** Jetson 플랫폼의 핵심 경쟁력으로, 하드웨어 가속을 위한 라이브러리 집합이다.
  - **CUDA Toolkit:** NVIDIA GPU의 대규모 병렬 처리 능력을 활용하여 범용 연산을 가속하기 위한 C/C++ 기반 개발 플랫폼이다. AI뿐만 아니라 다양한 과학 및 공학 연산의 기반이 된다.3
  - **cuDNN (CUDA Deep Neural Network library):** 합성곱(Convolution), 풀링(Pooling), 정규화(Normalization) 등 딥러닝에 필수적인 기본 연산들을 고도로 최적화하여 제공하는 라이브러리다. TensorFlow, PyTorch와 같은 딥러닝 프레임워크들은 내부적으로 cuDNN을 호출하여 GPU 성능을 극대화한다.28
  - **TensorRT:** 훈련된 AI 모델을 실제 배포 환경에 맞게 최적화하는 고성능 추론 SDK다. 레이어 융합(Layer Fusion), 정밀도 보정(Precision Calibration), 커널 자동 튜닝(Kernel Auto-Tuning) 등의 기술을 통해 모델의 크기를 줄이고, 추론 지연 시간(Latency)을 최소화하며, 처리량(Throughput)을 극대화한다. 이는 엣지 디바이스의 제한된 자원으로 실시간 추론을 가능하게 하는 핵심 기술이다.28
  - **VPI (Vision Programming Interface), OpenCV:** 컴퓨터 비전 및 이미지 처리를 위한 가속 라이브러리들이다.
- **Jetson Platform Services:** JetPack 6부터 도입된 새로운 구성 요소로, AI 애플리케이션 개발을 가속화하기 위한 즉시 사용 가능한 서비스 모음이다. VLM 추론 서비스, DeepStream 기반 인식 서비스 등이 포함된다.28


JetPack은 지속적으로 발전하며 새로운 기능과 향상된 성능을 제공한다.

- **JetPack 5.x (L4T 35.x):** Ubuntu 20.04를 기반으로 하며, Jetson Orin 제품군을 위한 안정적인 프로덕션 릴리스로 자리 잡았다. 보안 부팅, 디스크 암호화, OTA(Over-The-Air) 업데이트 등 상용 제품에 필수적인 기능들을 지원한다.26
- **JetPack 6.x (L4T 36.x):** Ubuntu 22.04와 최신 Linux 커널(5.15)을 도입한 대규모 업그레이드다. 가장 큰 변화는 소프트웨어 스택의 모듈화이다.28

JetPack 6의 모듈화는 단순한 운영체제 업데이트를 넘어선 전략적 전환을 의미한다. 이전의 모놀리식(Monolithic) 구조에서는 최신 TensorRT나 CUDA를 사용하기 위해 전체 Jetson Linux(L4T) 버전을 업그레이드해야만 했다. 이는 이미 안정성이 검증된 운영 환경을 변경해야 하는 부담을 개발자에게 안겨주었고, 새로운 AI 기술 채택의 장벽으로 작용했다.35 JetPack 6는 'Jetson AI Stack'을 기본 'Jetson Linux' BSP와 분리함으로써 이러한 문제를 해결했다.27 이제 개발자들은 안정적인 기본 OS를 유지하면서도 최신 AI 라이브러리만 독립적으로 업데이트할 수 있게 되었다. 이는 개발 편의성을 높이고, 새로운 AI 최적화 기술을 더 빠르게 적용할 수 있게 하여 프로젝트의 장기적인 유지보수성을 크게 향상시킨다. 더 나아가, 이러한 모듈화는 Canonical의 Ubuntu나 Red Hat과 같은 제3자 Linux 배포판을 공식적으로 지원할 수 있는 길을 열었다.34 이는 특정 OS 환경을 요구하는 기업 고객들에게 Jetson 플랫폼의 매력을 높이고, NVIDIA가 단독 OS 제공자로서의 부담을 덜게 하는 생태계 확장 전략의 일환이다.


NVIDIA는 특정 도메인에 최적화된 고수준 프레임워크를 제공하여 개발 생산성을 극대화한다.

- **NVIDIA Isaac:** 로보틱스 애플리케이션 개발을 위한 종합 플랫폼으로, 시뮬레이션 환경(Isaac Sim)과 로봇 운영 체제(ROS)에 최적화된 가속 라이브러리(Isaac ROS)를 제공하여 로봇의 인식, 탐색, 조작 기능 개발을 가속화한다.6
- **NVIDIA DeepStream:** 다중 비디오 및 센서 스트림을 실시간으로 분석하기 위한 지능형 영상 분석(IVA) 툴킷이다. 최적화된 GStreamer 플러그인 파이프라인을 통해 최소한의 코드로 고성능 영상 분석 애플리케이션을 구축할 수 있다.6
- **NVIDIA Riva:** 음성 인식(ASR), 텍스트 음성 변환(TTS) 등 대화형 AI 애플리케이션을 엣지에서 구현하기 위한 프레임워크다.6
- **TAO Toolkit & Omniverse Replicator:** TAO Toolkit을 사용하면 적은 양의 데이터로도 사전 훈련된 고성능 AI 모델을 특정 작업에 맞게 미세 조정(fine-tuning)할 수 있다. Omniverse Replicator는 물리적으로 정확한 합성 데이터를 생성하여 실제 데이터 수집의 어려움을 해결하고 모델의 강건성을 높인다.6


Jetson Orin Nano는 PyTorch, TensorFlow와 같은 업계 표준 딥러닝 프레임워크를 완벽하게 지원한다.3 NVIDIA는 Jetson에 최적화된 프레임워크 빌드를 제공하며, 개발자들은 익숙한 환경에서 모델을 개발하고 TensorRT를 통해 손쉽게 배포용으로 최적화할 수 있다. 또한, 

`l4t-text-generation`과 같은 사전 구성된 Docker 컨테이너를 통해 HuggingFace Transformers와 같은 최신 라이브러리 생태계를 복잡한 설정 없이 즉시 활용할 수 있다.22


Jetson Orin Nano의 강력한 성능과 컴팩트한 폼팩터는 다양한 산업 분야에서 혁신적인 AI 애플리케이션을 가능하게 한다.


로보틱스는 Jetson Orin Nano의 가장 핵심적인 응용 분야 중 하나다. 로봇이 자율적으로 주변 환경을 인식하고, 동적으로 경로를 계획하며, 인간 또는 물체와 정밀하게 상호작용하기 위해서는 막대한 양의 센서 데이터를 실시간으로 처리하는 능력이 필수적이다. Orin Nano는 카메라, LiDAR, IMU 등 다양한 센서로부터 입력되는 데이터를 실시간으로 융합하고, 이를 기반으로 복잡한 AI 모델을 실행하여 로봇의 '지능'을 구현한다.8

실제 적용 사례로는 공장이나 물류 창고에서 자율적으로 물품을 운반하는 자율 이동 로봇(AMR), 인간과 같은 공간에서 안전하게 협업하는 협동 로봇, 그리고 가정 환경에서 사용자를 돕는 개인용 서비스 로봇 등이 있다.8 커뮤니티 프로젝트 중 하나인, 단종된 Anki Vector 로봇의 두뇌를 Jetson Orin Nano로 교체하여 클라우드 서버 없이 로컬에서 모든 기능을 복원하고 Ollama LLM을 통합하여 대화 능력을 향상시킨 사례는 Orin Nano의 잠재력을 잘 보여준다.39


Jetson Orin Nano는 여러 개의 고해상도(1080p) 비디오 스트림을 동시에 처리하고 분석할 수 있는 능력을 갖추고 있어, 지능형 영상 분석(IVA) 애플리케이션에 이상적인 플랫폼이다.36 스마트 시티 분야에서는 교차로의 카메라 영상을 분석하여 실시간으로 교통 흐름을 최적화하고, 사고나 돌발 상황을 즉시 감지하여 대응 시간을 단축할 수 있다.8 스마트 리테일 환경에서는 매장 내 고객의 동선을 추적하고, 상품 진열 상태를 분석하여 재고를 관리하며, 맞춤형 쇼핑 경험을 제공하는 데 활용된다.38

이러한 애플리케이션들은 NVIDIA Metropolis 플랫폼과 DeepStream SDK를 통해 효율적으로 개발될 수 있다. DeepStream은 복잡한 영상 처리 파이프라인을 모듈화된 플러그인 형태로 구성하여, 개발자가 하드웨어 가속의 세부 사항을 신경 쓰지 않고도 고성능 IVA 솔루션을 신속하게 구축할 수 있도록 지원한다.8


드론은 크기, 무게, 전력 소비(SWaP: Size, Weight, and Power)에 대한 제약이 매우 큰 플랫폼이다. Jetson Orin Nano는 소형, 저전력 폼팩터에 강력한 AI 연산 능력을 집약하여 스마트 드론의 두뇌 역할을 수행하기에 최적이다.18 드론에 Orin Nano를 탑재하면, 촬영된 영상을 지상으로 전송하여 분석하는 대신 기체 내에서 실시간으로 객체를 탐지하고 추적하며, 이를 기반으로 자율적으로 비행 경로를 수정하거나 특정 임무를 수행할 수 있다.

응용 분야는 매우 다양하다. 농업에서는 드론이 넓은 경작지를 비행하며 작물의 생육 상태나 병충해를 실시간으로 분석하고, 필요한 곳에만 정밀하게 방제 작업을 수행할 수 있다.38 재난 현장에서는 실종자를 수색하거나 위험 지역의 상황을 파악하는 데 활용될 수 있다.41 이미 시장에는 Orin Nano를 기반으로 한 드론 개발 키트와 Pixhawk 비행 컨트롤러가 통합된 번들 제품들이 출시되어 있어, 개발자들이 보다 쉽게 AI 드론을 구축할 수 있는 환경이 마련되어 있다.42


Jetson Orin Nano는 엣지 AI 시장에서 독특한 포지션을 차지하고 있으며, 이는 이전 세대 제품 및 주요 경쟁 플랫폼과의 비교를 통해 명확해진다.


Jetson Orin Nano는 1세대 Jetson Nano의 직접적인 후속 제품으로, 모든 측면에서 세대교체를 이루었다. GPU는 Maxwell에서 Ampere 아키텍처로, CPU는 4코어 Cortex-A57에서 6코어 Cortex-A78AE로, 메모리는 LPDDR4에서 LPDDR5로 업그레이드되었다.4 이러한 하드웨어의 근본적인 변화는 압도적인 성능 차이로 이어진다. AI 추론 성능은 특정 모델에서 최대 80배, 일반적인 CPU 연산 성능은 6.6배 향상되었으며, 전력 효율성을 나타내는 와트당 성능은 50배 개선되었다.6 이는 단순히 더 빠른 것이 아니라, 동일한 전력으로 훨씬 더 복잡하고 무거운 AI 작업을 수행할 수 있게 되었음을 의미한다.


엣지 AI 시장에는 각기 다른 설계 철학과 강점을 가진 플랫폼들이 경쟁하고 있다. Jetson Orin Nano는 이들 사이에서 '유연한 고성능'이라는 독자적인 영역을 구축하고 있다.

- **NVIDIA Jetson Orin Nano (유연한 강자):** CUDA 기반의 범용 GPU를 핵심으로 하는 Orin Nano는 최고의 유연성을 제공한다. 개발자는 PyTorch, TensorFlow 등 원하는 프레임워크로 개발된 거의 모든 종류의 AI 모델을 실행할 수 있으며, 전체 파이프라인에 대한 세밀한 제어가 가능하다. 특히 트랜스포머와 같이 새롭게 등장하는 모델 아키텍처에 신속하게 대응할 수 있는 것은 프로그래밍 가능한 GPU 덕분이다. 이러한 유연성은 상대적으로 높은 전력 소비와 가격이라는 트레이드오프를 가진다.37
- **Raspberry Pi 5 (만능 재주꾼):** 라즈베리 파이의 강점은 강력한 범용 CPU 성능과 방대한 커뮤니티 생태계에 있다. AI는 이 플랫폼의 핵심 기능이 아니며, USB 가속기(AI Kit)를 추가하거나 CPU 기반의 라이브러리를 통해 구현되는 '부가 기능'에 가깝다. AI 작업 성능은 Jetson에 비해 현저히 낮지만, 가격이 저렴하고 AI가 프로젝트의 여러 구성 요소 중 하나일 뿐인 범용 컴퓨팅 작업에 더 적합하다.45
- **Google Coral (특화된 전문가):** Coral 플랫폼은 Edge TPU라는 주문형 반도체(ASIC)를 사용한다. 이 하드웨어는 8비트 정수로 양자화된 특정 유형의 신경망 레이어를 실행하는 단 하나의 목적에 극도로 최적화되어 있다.47 그 결과 와트당 성능은 매우 뛰어나지만, 지원하는 프레임워크가 TensorFlow Lite로 제한되고, 특정 컴파일 과정을 거쳐야 하며, 지원되지 않는 모델 레이어가 있을 경우 성능이 급격히 저하되는 등 유연성이 극히 낮다는 단점이 있다.37

결론적으로, Jetson Orin Nano는 라즈베리 파이가 제공하는 수준 이상의 AI 성능과 유연성을 필요로 하면서도, Google Coral의 경직된 아키텍처에는 맞지 않는 복잡하고 새로운, 혹은 맞춤형 AI 모델을 개발하려는 사용자에게 최적의 솔루션이다. 이는 범용 컴퓨팅과 특화된 ASIC 사이의 중요한 교량 역할을 한다.

| 기능 (Feature)         | NVIDIA Jetson Orin Nano Super           | Raspberry Pi 5 (+ AI Kit)              | Google Coral Dev Board            |
| ---------------------- | --------------------------------------- | -------------------------------------- | --------------------------------- |
| **주요 응용 분야**     | 고성능 엣지 AI, 로보틱스, 생성형 AI     | 범용 컴퓨팅, 교육, IoT, 입문용 AI      | 고효율 AI 추론 (특정 모델)        |
| **AI 아키텍처**        | Ampere GPU (CUDA + Tensor 코어)         | CPU + 외부 가속기 (TPU)                | Edge TPU (ASIC)                   |
| **AI 성능 (TOPS)**     | 최대 67 TOPS (INT8)                     | 최대 13 TOPS (AI Kit 사용 시)          | 4 TOPS                            |
| **프레임워크 유연성**  | 매우 높음 (PyTorch, TF, ONNX 등)        | 중간 (CPU 기반, TFLite 등)             | 낮음 (TensorFlow Lite 전용)       |
| **범용 CPU 성능**      | 우수 (6-core Cortex-A78AE)              | 매우 우수 (4-core Cortex-A76 @ 2.4GHz) | 보통 (Quad-core Cortex-A53)       |
| **내장 연결성**        | Wi-Fi/BT (모듈 포함)                    | Wi-Fi 6, Bluetooth 5.0 내장            | Wi-Fi 5, Bluetooth 5.0 내장       |
| **전력 소비**          | 7W – 25W                                | 3W – 5W (유휴/부하)                    | 2.5W – 5W                         |
| **가격 (개발자 키트)** | $249                                    | ~$80 (8GB) + AI Kit                    | ~$150                             |
| **생태계 및 지원**     | NVIDIA JetPack, CUDA, Isaac, DeepStream | 방대한 커뮤니티, 다양한 OS 지원        | Google Coral SDK, TensorFlow Lite |


NVIDIA Jetson 플랫폼을 기반으로 상용 제품을 개발하는 기업은 개발자 키트와 양산형 모듈의 차이점을 명확히 이해해야 한다. 이 둘은 용도, 하드웨어 사양, 보증 정책 등에서 근본적인 차이를 가진다.


- **용도 (Intended Use):** 개발자 키트는 이름 그대로 개발 및 테스트 단계, 즉 양산 전 환경에서 소프트웨어를 검증하고 프로토타입을 제작하기 위한 도구다. 반면, 양산형 모듈은 최종 상용 제품에 탑재되어 긴 수명주기 동안 안정적으로 작동하도록 설계된 부품이다. 개발자 키트를 최종 제품에 사용하는 것은 보증 및 신뢰성 문제로 인해 권장되지 않는다.35
- **하드웨어 (Hardware):** 개발자 키트에 포함된 SoM은 비양산 사양의 부품을 일부 포함할 수 있으며, 부품 변경 사항이 사전에 공지되지 않을 수 있다. 가장 큰 물리적 차이는 microSD 카드 슬롯의 유무로, 개발자 키트 모듈에는 편의성을 위해 슬롯이 있지만 양산형 모듈에는 없다.35 양산형 모듈은 더 엄격한 환경(온도, 진동 등)에서 작동하도록 검증된 산업 등급의 부품으로 구성된다.35
- **소프트웨어 (Software):** 양산형 모듈은 아무런 소프트웨어도 설치되지 않은 '공장 초기화' 상태로 출하된다. 개발자는 개발자 키트에서 충분히 테스트하고 검증한 최종 소프트웨어 이미지를 양산형 모듈에 직접 플래싱하여 사용해야 한다.35
- **보증 및 수명 (Warranty and Lifetime):** 개발자 키트는 일반적으로 1년의 제한된 보증을 제공한다. 반면, 양산형 모듈은 통상 3년 이상의 보증 기간과 함께 5년에서 10년에 이르는 장기 운영 수명을 보장한다. 또한, 단종(EOL) 전 최종 구매(Last Time Buy) 공지를 통해 기업이 안정적으로 부품을 수급할 수 있도록 지원한다.35

| 구분 (Aspect)     | 개발자 키트 (Developer Kit)             | 양산형 모듈 (Production Module)             |
| ----------------- | --------------------------------------- | ------------------------------------------- |
| **주요 용도**     | 프로토타이핑, 소프트웨어 개발 및 테스트 | 최종 상용 제품에 통합 및 배포               |
| **하드웨어 사양** | 비양산 사양 부품 포함 가능              | 산업 등급 부품, 엄격한 환경 기준 충족       |
| **스토리지**      | microSD 카드 슬롯 탑재                  | microSD 카드 슬롯 없음, 외부 NVMe/eMMC 사용 |
| **소프트웨어**    | JetPack 사전 설치 또는 사용자가 설치    | 소프트웨어 미설치 상태로 제공               |
| **보증 기간**     | 1년 (개발용)                            | 3년 이상 (상용 제품용)                      |
| **운영 수명**     | 명시되지 않음                           | 5~10년 보장                                 |
| **공급 보장**     | 보장되지 않음, 예고 없이 단종 가능      | 장기 공급 보장, 단종 전 LTB 공지            |
| **검증 수준**     | 제한된 환경에서의 기본 기능 검증        | 전체 기능 및 신뢰성 검증 (데이터시트 명시)  |



NVIDIA Jetson Orin Nano는 엣지 AI 컴퓨팅 분야에서 새로운 기준점을 제시하는 혁신적인 플랫폼이다. 이전 세대와는 차원을 달리하는 비약적인 성능 향상을 이루었으며, 특히 트랜스포머 기반의 생성형 AI 모델을 소형 엣지 디바이스에서 실시간으로 구동할 수 있는 능력을 $249라는 합리적인 가격에 제공함으로써 엣지 AI 기술의 접근성을 획기적으로 개선했다.8

강력한 Ampere 아키텍처 기반 하드웨어, JetPack SDK를 중심으로 한 성숙하고 포괄적인 소프트웨어 생태계, 그리고 전 세계의 방대한 개발자 커뮤니티 지원이 결합되어, 개발자들이 혁신적인 아이디어를 신속하게 프로토타입으로 구현하고, 나아가 안정적인 상용 제품으로 출시하는 전 과정을 가속화하는 최적의 환경을 제공한다.6


Jetson Orin Nano의 등장은 로보틱스, 지능형 공장 자동화, 스마트 시티, 헬스케어 등 다양한 산업 분야에서 AI 기술 도입을 촉진하는 강력한 기폭제가 될 것이다. 특히, 로컬 디바이스에서 실행되는 생성형 AI는 사용자 경험을 근본적으로 바꾸고 새로운 서비스 모델을 창출할 잠재력을 지니고 있다.

이 플랫폼을 성공적으로 활용하기 위해 개발자와 기업은 다음 사항을 고려해야 한다.

1. **개발자를 위한 제언:** 초기 설정 과정에서 JetPack 6.x 호환성을 위한 펌웨어 업데이트가 필수적임을 인지해야 한다.18 또한, 가벼운 테스트를 넘어선 본격적인 AI 개발 및 성능 평가를 위해서는 microSD 카드가 아닌 NVMe SSD를 필수 스토리지로 계획하고 예산에 반영해야 한다.20
2. **기업을 위한 제언:** 프로토타이핑 단계에서는 개발자 키트의 편의성과 접근성을 최대한 활용하되, 제품의 상용화를 계획하는 시점에서는 반드시 양산형 모듈로의 전환을 고려해야 한다. 이는 제품의 장기적인 신뢰성, 안정적인 부품 수급, 그리고 보증 정책을 확보하기 위한 필수적인 과정이다. 개발 초기부터 양산형 모듈의 하드웨어 제약(예: microSD 슬롯 부재)을 염두에 두고 소프트웨어를 설계하는 것이 바람직하다.

Jetson Orin Nano는 기술적 가능성과 경제적 합리성의 균형을 맞춘 플랫폼으로서, 향후 수년간 엣지 AI 혁신을 주도할 핵심 동력으로 기능할 것으로 전망된다.


1. Edge AI: 7 Incredible Reasons the Jetson Orin Nano is a Game-Changer - DEV Community, accessed August 19, 2025, https://dev.to/zulnourain_muhammad_4026c/edge-ai-7-incredible-reasons-the-jetson-orin-nano-is-a-game-changer-46lm
2. NVIDIA Jetson Orin Nano Super Developers Kit - Getting Started | DroneBot Workshop, accessed August 19, 2025, https://dronebotworkshop.com/jetson-orin-nano/
3. Which Jetson is best for AI application build? - WizzDev, accessed August 19, 2025, https://wizzdev.com/blog/which-jetson-is-best-for-ai-application-build/
4. NVIDIA Jetson Orin Nano Review: A New Fantastic Beast, accessed August 19, 2025, https://nuadev.com/nvidia-jetson-orin-nano-review-new-fantastic-beast/
5. Jetson Nano Brings the Power of Modern AI to Edge Devices - NVIDIA, accessed August 19, 2025, https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/jetson-nano/product-development/
6. NVIDIA Jetson Orin Nano Developer Kit - Seeed Studio, accessed August 19, 2025, https://files.seeedstudio.com/wiki/Jetson-Orin-Nano-DevKit/jetson-orin-nano-developer-kit-datasheet.pdf
7. NVIDIA Jetson Orin Nano Developer Kit Offers to Ease Creation of Robotics and Edge AI Applications, accessed August 19, 2025, https://www.robotics247.com/article/nvidia_jetson_orin_nano_developer_kit_offers_ease_creation_robotics_edge_ai_applications
8. Jetson Orin Nano Super Developer Kit - NVIDIA, accessed August 19, 2025, https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/jetson-orin/nano-super-developer-kit/
9. NVIDIA Jetson Orin™ Nano Super Developer Kit - SparkFun Electronics, accessed August 19, 2025, https://www.sparkfun.com/nvidia-jetson-orin-nano-developer-kit.html
10. NVIDIA Jetson Orin Nano Super Developer Kit (67 TOPS, 8GB, 25W) - DFRobot, accessed August 19, 2025, https://www.dfrobot.com/product-2900.html
11. Exploring NVIDIA Jetson Orin Nano Super Mode performance using Generative AI, accessed August 19, 2025, https://www.ridgerun.com/post/exploring-nvidia-jetson-orin-nano-super-mode-performance-using-generative-ai
12. NVIDIA Jetson Orin Nano Super Developer Kit/Module, accessed August 19, 2025, https://www.macnica.co.jp/en/business/semiconductor/manufacturers/nvidia/products/141900/
13. YOLO11 Jetson Orin Nano: Super Fast Edge AI | Ultralytics, accessed August 19, 2025, https://www.ultralytics.com/blog/ultralytics-yolo11-on-nvidia-jetson-orin-nano-super-fast-and-efficient
14. jetson-orin-nano-datasheet-r4-web.pdf - Open Zeka, accessed August 19, 2025, https://openzeka.com/wp-content/uploads/2023/03/jetson-orin-nano-datasheet-r4-web.pdf
15. NVIDIA's Jetson Orin Nano SuperComputer: The Next Big Thing in EdgeAI - Medium, accessed August 19, 2025, https://medium.com/data-science-in-your-pocket/nvidias-jetson-orin-nano-supercomputer-the-next-big-thing-in-edgeai-e9eff687ae62
16. Everything You Need to Know about Jetson Orin Nano - Elecrow, accessed August 19, 2025, https://www.elecrow.com/blog/everything-you-need-to-know-about-jetson-orin-nano.html
17. NVIDIA Jetson Orin Nano 8 GB Specs | TechPowerUp GPU Database, accessed August 19, 2025, https://www.techpowerup.com/gpu-specs/jetson-orin-nano-8-gb.c4082
18. Jetson Orin Nano Developer Kit Getting Started Guide, accessed August 19, 2025, https://developer.nvidia.com/embedded/learn/get-started-jetson-orin-nano-devkit
19. Jetson Orin Nano Developer Kit User Guide - Hardware Specs, accessed August 19, 2025, https://developer.nvidia.com/embedded/learn/jetson-orin-nano-devkit-user-guide/hardware_spec.html
20. Getting Started with NVIDIA Jetson Orin Nano - Shawn Hymel, accessed August 19, 2025, https://shawnhymel.com/2255/getting-started-with-nvidia-jetson-orin-nano/
21. NVIDIA Jetson Orin Nano Super: Powering DeepSeek R1 70B Inference at the Edge!, accessed August 19, 2025, https://www.storagereview.com/review/nvidia-jetson-orin-nano-super-powering-deepseek-r1-70b-inference-at-the-edge
22. Tutorial - API Examples - NVIDIA Jetson AI Lab, accessed August 19, 2025, https://www.jetson-ai-lab.com/tutorial_api-examples.html
23. Jetson Orin Nano Tutorial: SSD Install, Boot, and JetPack Setup - JetsonHacks, accessed August 19, 2025, https://jetsonhacks.com/2023/05/30/jetson-orin-nano-tutorial-ssd-install-boot-and-jetpack-setup/
24. Benchmarking the Brand New nVidia Jetson Nano: 4GB, USB 3, $99!, accessed August 19, 2025, https://www.sevarg.net/2019/04/07/benchmarking-nvidia-jetson-nano/
25. Benchmarks - NVIDIA Jetson AI Lab, accessed August 19, 2025, https://www.jetson-ai-lab.com/benchmarks.html
26. JetPack SDK 5.1.2 - NVIDIA Developer, accessed August 19, 2025, https://developer.nvidia.com/embedded/jetpack-sdk-512
27. JetPack SDK - NVIDIA Developer, accessed August 19, 2025, https://developer.nvidia.com/embedded/jetpack
28. JetPack SDK 6.0 | NVIDIA Developer, accessed August 19, 2025, https://developer.nvidia.com/embedded/jetpack-sdk-60
29. dev.to, accessed August 19, 2025, [https://dev.to/zulnourain_muhammad_4026c/edge-ai-7-incredible-reasons-the-jetson-orin-nano-is-a-game-changer-46lm#:~:text=For%20developers%20and%20businesses%2C%20the,deployment%20of%20deep%20learning%20models.](https://dev.to/zulnourain_muhammad_4026c/edge-ai-7-incredible-reasons-the-jetson-orin-nano-is-a-game-changer-46lm#:~:text=For developers and businesses%2C the,deployment of deep learning models.)
30. NVIDIA® TensorRT™ is an SDK for high-performance deep learning inference on NVIDIA GPUs. This repository contains the open source components of TensorRT. - GitHub, accessed August 19, 2025, https://github.com/NVIDIA/TensorRT
31. NVIDIA Xavier - JetPack 5.0.2 - Components - TensorRT - RidgeRun Developer Wiki, accessed August 19, 2025, https://developer.ridgerun.com/wiki/index.php/Xavier/JetPack_5.0.2/Components/TensorRT
32. JetPack SDK - NVIDIA Developer, accessed August 19, 2025, https://developer.nvidia.com/embedded/jetpack-sdk-513
33. JetPack SDK 5.0.2 - NVIDIA Developer, accessed August 19, 2025, https://developer.nvidia.com/embedded/jetpack-sdk-502
34. JetPack SDK 6.0 DP - NVIDIA Developer, accessed August 19, 2025, https://developer.nvidia.com/embedded/jetpack-sdk-60dp
35. Jetson FAQ | NVIDIA Developer, accessed August 19, 2025, https://developer.nvidia.com/embedded/faq
36. Accelerate Video Analytics Development with NVIDIA Jetson Orin - Seeed Studio, accessed August 19, 2025, https://www.seeedstudio.com/blog/2024/06/06/mastering-video-analytics-on-nvidia-jetson-your-best-bet-for-concurrent-and-high-throughput-solution/
37. Top 5 Edge AI Devices Like NVIDIA Jetson Nano (2025 Edition) - GenAI Protos, accessed August 19, 2025, https://www.genaiprotos.com/project/edge-ai-devices/
38. Case Studies: Real-World Applications of NVIDIA® Jetson Orin™ Nano - ProX PC, accessed August 19, 2025, https://www.proxpc.com/blogs/case-studies-real-world-applications-of-nvidia-jetson-orin-nano
39. Turn NVIDIA Jetson Orin Nano Super into an AI Brain for Anki Vector Robot - YouTube, accessed August 19, 2025, https://www.youtube.com/watch?v=qG0q6FLFAg8
40. NVIDIA INTELLIGENT VIDEO ANALYTICS (IVA), accessed August 19, 2025, https://www.nvidia.com/content/dam/en-zz/Solutions/intelligent-machines/documents/NVIDIA_Intelligent_Video_Analytics_Platform_Infographic_Poster.pdf
41. 25 Best Jetson Nano Projects - All3DP, accessed August 19, 2025, https://all3dp.com/2/best-jetson-nano-projects/
42. DAMIAO ORIN Nano UAV Drone Development Kit for NVIDIA Jetson ORIN Nano | eBay, accessed August 19, 2025, https://www.ebay.com/itm/297382305436
43. ARK Jetson PAB Orin Nano 4GB Bundle, accessed August 19, 2025, https://arkelectron.com/product/ark-jetson-orin-nano-bundle/
44. Jetson Nano vs Raspberry Pi AI: The Ultimate Performance Comparison fo, accessed August 19, 2025, https://thinkrobotics.com/blogs/learn/jetson-nano-vs-raspberry-pi-ai-the-ultimate-performance-comparison-for-edge-computing
45. Jetson Nano vs Raspberry Pi 5 for AI: The Ultimate Performance and Val, accessed August 19, 2025, https://thinkrobotics.com/blogs/learn/jetson-nano-vs-raspberry-pi-5-for-ai-the-ultimate-performance-and-value-comparison
46. Raspberry Pi 5 vs. Jetson Nano: General purpose or AI-focused - XDA Developers, accessed August 19, 2025, https://www.xda-developers.com/raspberry-pi-5-vs-jetson-nano/
47. Google Coral Edge TPU vs NVIDIA Jetson Nano: A quick deep dive into EdgeAI performance | by Sam Sterckval | Medium, accessed August 19, 2025, https://medium.com/@samsterckval/google-coral-edge-tpu-vs-nvidia-jetson-nano-a-quick-deep-dive-into-edgeai-performance-bc7860b8d87a
48. NVIDIA Jetson Nano and Google Coral Edge TPU — a comparison - Medium, accessed August 19, 2025, https://medium.com/3dvisionlabs/nvidia-jetson-nano-and-google-coral-edge-tpu-a-comparison-b244a176d0e0
49. Google Coral Edge TPU vs NVIDIA Jetson Nano: A quick deep dive into EdgeAI performance : r/hardware - Reddit, accessed August 19, 2025, https://www.reddit.com/r/hardware/comments/bqrrro/google_coral_edge_tpu_vs_nvidia_jetson_nano_a/
50. Jetson AGX Orin for Next-Gen Robotics - NVIDIA, accessed August 19, 2025, https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/jetson-orin/
51. Module vs developer kit - Jetson Xavier NX, accessed August 19, 2025, https://forums.developer.nvidia.com/t/module-vs-developer-kit/164040
52. Any difference between orin module and develop kit？ - Jetson Orin ..., accessed August 19, 2025, https://forums.developer.nvidia.com/t/any-difference-between-orin-module-and-develop-kit/317576
53. NVIDIA's $249 Secret Weapon for Edge AI - Jetson Orin Nano Super: Driveway Monitor, accessed August 19, 2025, https://www.youtube.com/watch?v=QHBr8hekCzg