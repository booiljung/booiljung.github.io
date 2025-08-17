# OpenWRT를 활용한 고성능 UAV 데이터링크 시스템 구축에 대한 고찰

## 서론: 차세대 드론 통신을 위한 OpenWRT의 가능성

현대의 무인 항공기(Unmanned Aerial Vehicle, UAV) 시스템은 단순한 원격 조종을 넘어 자율 비행, 실시간 데이터 처리, 그리고 복잡한 임무 수행이 가능한 고도로 지능화된 플랫폼으로 발전하고 있다. 이러한 발전의 중심에는 기체와 지상 통제소(Ground Control Station, GCS)를 연결하는 데이터링크 기술이 있다. 전통적인 데이터링크는 주로 저대역폭의 직렬 통신에 의존하여 원격 측정(Telemetry) 데이터와 제어 신호를 주고받는 데 국한되었다.1 그러나 고해상도 영상 실시간 전송, 다중 센서 데이터 스트리밍, 그리고 다수의 드론이 협력하는 군집 비행(Swarm) 임무가 보편화되면서, 기존 데이터링크는 대역폭, 지연 시간, 확장성 측면에서 명백한 한계에 직면했다.

이러한 기술적 요구사항의 변화 속에서 OpenWRT가 차세대 드론 데이터링크를 위한 강력한 대안으로 부상하고 있다. OpenWRT는 상용 라우터에 탑재되는 제한적인 펌웨어가 아니라, 리눅스 커널을 기반으로 하는 완전한 기능을 갖춘 오픈소스 임베디드 운영체제이다.2 이는 사용자에게 네트워크 스택의 모든 계층에 대한 완전한 제어 권한을 부여하며, 상용 데이터링크 솔루션이 제공하지 못하는 극강의 유연성, 확장성, 그리고 투명성을 보장한다. Herelink, DJI O3 Air Unit과 같은 상용 솔루션들은 특정 생태계에 종속되어 있고 기능 확장이 제한적인 반면 4, OpenWRT는 사용자가 특정 임무 요구사항에 완벽하게 부합하는 맞춤형 데이터링크를 '설계'하고 '구축'할 수 있는 무한한 가능성을 제공한다.

본 보고서는 OpenWRT의 핵심 기술 원리부터 시작하여, 실제 드론 환경에 적합한 하드웨어 선정 기준, MAVLink 프로토콜 연동 및 저지연 영상 스트리밍과 같은 핵심 소프트웨어의 구체적인 구현 방법, 그리고 최종적으로 상용 솔루션과의 심층적인 성능 비교 분석에 이르기까지, OpenWRT 기반의 고성능 UAV 데이터링크를 구축하는 데 필요한 모든 기술적 요소를 체계적이고 심도 있게 고찰하는 것을 목표로 한다. 이를 통해 UAV 시스템 엔지니어와 연구자들에게 이론적 기반과 실용적 지침을 동시에 제공하고자 한다.

## 1.  OpenWRT 기반 데이터링크의 기술적 원리

### 1.1  OpenWRT의 핵심 아키텍처와 네트워크 스택

OpenWRT를 드론 데이터링크에 적용하기 위해서는 먼저 그 근간을 이루는 기술적 아키텍처에 대한 깊이 있는 이해가 선행되어야 한다. OpenWRT는 단순한 라우팅 기능의 집합이 아니라, 고도로 최적화된 임베디드 리눅스 배포판으로서 강력하고 유연한 기반을 제공한다.

임베디드 리눅스로서의 특징

OpenWRT의 핵심은 경량화된 리눅스 환경에 있다. 이는 제한된 CPU 성능, 메모리, 그리고 저장 공간을 가진 드론의 온보드 컴퓨터 환경에 최적화되어 있다.2 주요 구성 요소는 현대적인 리눅스 커널, 시스템 유틸리티의 집합인 BusyBox, 그리고 표준 C 라이브러리의 경량 버전인 musl이다. 이러한 구성 요소들은 시스템 리소스를 최소한으로 사용하면서도 완전한 POSIX 호환 환경을 제공하여, 복잡한 네트워크 애플리케이션을 안정적으로 구동할 수 있게 한다.

OverlayFS 파일 시스템의 중요성

OpenWRT의 가장 큰 특징 중 하나는 OverlayFS 파일 시스템을 채택했다는 점이다.2 이는 읽기 전용의 압축된 SquashFS 파티션 위에 변경 사항을 기록하는 쓰기 가능한 JFFS2 파티션을 겹쳐서 사용하는 방식이다. 이 구조 덕분에 사용자는 펌웨어 전체를 다시 컴파일하고 플래싱하는 위험하고 번거로운 과정 없이도 새로운 소프트웨어 패키지를 설치하거나 시스템 설정을 영구적으로 변경할 수 있다. 드론 데이터링크를 구축하는 과정에서 수많은 설정 변경과 소프트웨어 추가가 필요한데, OverlayFS는 이러한 개발 및 유지보수 과정을 매우 유연하고 안전하게 만들어주는 핵심적인 역할을 수행한다.

UCI (Unified Configuration Interface)

OpenWRT는 모든 시스템 설정을 /etc/config/ 디렉토리 내의 단순한 텍스트 파일들로 통합하여 관리하는데, 이를 UCI(Unified Configuration Interface)라고 한다.2 네트워크 인터페이스, 방화벽 규칙, 무선 설정 등 모든 것이 일관된 형식으로 기술된다. 사용자는 `uci`라는 강력한 명령줄 유틸리티를 통해 이러한 설정들을 스크립트 기반으로 읽고, 쓰고, 수정할 수 있다. 이는 복잡한 드론 네트워크 구성을 수동으로 편집하는 과정에서 발생할 수 있는 실수를 줄이고, 다수의 드론에 동일한 구성을 배포하는 등 자동화된 관리를 가능하게 한다. 사용이 편리한 웹 인터페이스인 LuCI 역시 내부적으로는 이 UCI 시스템을 통해 설정을 변경하므로, GUI와 CLI 간의 설정이 완벽하게 동기화된다.2

opkg 패키지 관리 시스템

OpenWRT는 opkg라는 경량 패키지 관리 시스템을 통해 수천 개에 달하는 방대한 소프트웨어 패키지 저장소에 접근할 수 있다.2 이는 OpenWRT의 확장성을 극대화하는 요소이다. 데이터링크 구축에 필수적인 `ser2net`(시리얼-네트워크 브릿지), `mavlink-router`(지능형 MAVLink 라우터), `gstreamer`(영상 스트리밍 프레임워크), `wireguard`(VPN), `kmod-batman-adv`(메시 네트워킹) 등의 핵심 소프트웨어를 단 몇 줄의 명령어로 온보드 시스템에 신속하게 설치하고 구성할 수 있다.

이러한 아키텍처적 특징들은 OpenWRT가 단순한 통신 모듈 펌웨어를 넘어, 드론 자체를 하나의 완전한 '네트워크 노드'로 변모시키는 기반이 된다. 즉, OpenWRT를 탑재한 드론은 더 이상 외부 데이터링크 모듈에 의존하는 단말기가 아니라, 스스로 라우팅하고, 데이터를 처리하며, 네트워크 토폴로지를 동적으로 구성할 수 있는 '비행 라우터(Flying Router)'로서 기능하게 된다. 이는 전통적인 드론 시스템 아키텍처의 패러다임을 전환하는 것으로, 군집 비행, 동적 네트워크 재구성, 온보드 데이터 분산 처리 등 기존에는 상상하기 어려웠던 고도의 네트워크 기반 임무 수행의 문을 여는 핵심적인 변화라 할 수 있다.

### 1.2  네트워크 토폴로지 설계

OpenWRT의 강력한 네트워크 스택을 활용하면, 임무의 특성에 따라 다양한 네트워크 토폴로지를 유연하게 설계하고 구현할 수 있다.

#### 1.2.1  기본 Point-to-Point (PtP) 링크

가장 기본적인 구성은 단일 드론과 단일 GCS 간의 1:1 통신을 위한 지점 간(Point-to-Point) 링크이다. 이는 전통적인 RC 링크를 IP 기반으로 현대화하는 첫 단계이다. 일반적으로 드론에 탑재된 OpenWRT 장치는 AP(Access Point) 모드로 동작하여 자체적인 Wi-Fi 네트워크를 생성하고, GCS 측의 노트북이나 태블릿은 이 네트워크에 클라이언트(Station)로 접속한다. `uci` 명령어를 사용하여 드론 측의 무선 인터페이스를 AP 모드로 설정하는 예시는 다음과 같다.7

```Bash
# /etc/config/wireless 설정
uci set wireless.radio0.disabled='0'
uci set wireless.@wifi-iface.device='radio0'
uci set wireless.@wifi-iface.mode='ap'
uci set wireless.@wifi-iface.ssid='Drone_AP'
uci set wireless.@wifi-iface.network='lan'
uci set wireless.@wifi-iface.encryption='psk2+ccmp'
uci set wireless.@wifi-iface.key='YourSecurePassword'
uci commit wireless
wifi reload
```

이 구성은 설정이 간단하고 직관적이지만, 통신 거리가 Wi-Fi 신호 강도에 직접적으로 의존하며, 다수의 드론을 동시에 운용하기에는 부적합하다.

#### 1.2.2  다개체 통신을 위한 메시 네트워킹

두 대 이상의 드론이 협력하는 군집 비행이나 광역 감시 임무에서는 메시 네트워킹이 필수적이다. 메시 네트워크는 중앙의 AP 없이 각 노드(드론)가 서로 직접 통신하며, 데이터 패킷을 목적지까지 여러 노드를 거쳐 전달(multi-hop)하는 탈중앙화된 네트워크 토폴로지이다.

- **Ad-hoc (IBSS) 모드:** 가장 오래된 방식의 P2P 네트워킹으로, 설정이 매우 간단하다.8 모든 노드가 동일한 SSID와 채널을 사용하여 하나의 브로드캐스트 도메인을 형성한다. 하지만 노드 수가 증가하면 성능이 급격히 저하되고, 특정 노드가 다른 두 노드의 통신 범위 밖에 있어 충돌을 감지하지 못하는 '숨은 노드 문제(Hidden Node Problem)'에 매우 취약하다는 단점이 있다.9
- **IEEE 802.11s:** Wi-Fi 표준에 공식적으로 포함된 링크 계층(Layer 2) 메시 프로토콜이다. 별도의 라우팅 프로토콜 없이 MAC 주소를 기반으로 경로를 설정하고 유지한다. OpenWRT에서 802.11s를 사용하기 위해서는 기본 `wpad-basic` 패키지를 제거하고 메시 기능을 지원하는 `wpad-mesh-openssl` 또는 `wpad-mesh-wolfssl` 패키지를 설치해야 한다.10
- **B.A.T.M.A.N.-adv (Better Approach To Mobile Adhoc Networking):** 현재 드론 메시 네트워킹에서 가장 각광받는 기술로, 링크 계층 위에서 동작하며 가상의 이더넷 스위치처럼 작동하는 고급 메시 프로토콜이다.10 B.A.T.M.A.N.-adv의 핵심은 각 노드가 주기적으로 OGM(Originator Messages)이라는 작은 패킷을 브로드캐스트하여 이웃 노드와의 링크 품질(TQ, Transmit Quality)을 지속적으로 측정하고, 이 정보를 바탕으로 전체 네트워크에서 목적지까지의 최적 경로를 동적으로 계산한다는 점이다. 이는 802.11s, 이더넷, 심지어 VPN 터널 등 다양한 종류의 하위 링크 위에서 동작할 수 있어 이종(heterogeneous) 네트워크 구성에 매우 유리하다. `kmod-batman-adv`와 `batctl` 패키지를 설치하여 활성화할 수 있다.10

B.A.T.M.A.N.-adv는 단순한 연결성을 제공하는 것을 넘어, 드론 군집을 위한 '자율 회복 네트워크(Self-Healing Network)'를 구현하는 핵심 기술로 평가받는다. 군집 비행 중 특정 드론이 장애물에 의해 다른 드론과의 직접적인 Wi-Fi 링크를 상실하거나, 임무 지역을 이탈하여 통신이 두절되는 상황을 가정해 보자. B.A.T.M.A.N.-adv는 이러한 링크의 단절을 수 초 내에 감지하고, 즉시 사용 가능한 다른 드론을 경유하는 대체 경로를 자동으로 재설정한다.10 이는 군집 네트워크가 외부의 방해나 내부의 결함에 대해 스스로를 '치유'하는 능력을 갖게 됨을 의미한다. 여기에 OpenWRT의 다중 인터페이스 관리 능력을 결합하면 더욱 강력한 시스템을 구축할 수 있다. 예를 들어, Wi-Fi 메시 링크가 끊어진 드론이 예비로 장착된 LTE 모뎀을 통해 인터넷에 접속하고, 이 LTE 링크를 통해 다른 드론과 VPN 터널을 형성할 수 있다. B.A.T.M.A.N.-adv는 이 VPN 터널 또한 메시 네트워크의 일부로 즉시 통합하여, 전체 군집의 연결성을 끊김 없이 유지하는 고도의 '하이브리드 메시 네트워크'를 실현할 수 있다. 이러한 자율 회복 능력은 군사 작전이나 재난 대응과 같이 단일 실패 지점(Single Point of Failure)이 치명적인 결과를 초래할 수 있는 고신뢰성 시나리오에서 결정적인 차이를 만들어낸다.

#### 1.2.3  BVLOS를 위한 셀룰러 네트워크 연동

가시거리 너머(Beyond Visual Line of Sight, BVLOS) 작전을 위해서는 기존의 Wi-Fi 기반 통신의 거리 한계를 극복해야 한다. OpenWRT는 USB 포트를 통해 4G/LTE 모뎀을 손쉽게 통합할 수 있는 기능을 제공한다.13 이를 통해 드론은 전 세계에 구축된 상용 이동통신망에 접속하여 이론적으로 무제한의 통신 거리를 확보할 수 있다.14 하지만 셀룰러 네트워크는 지상 사용자를 기준으로 설계되었기 때문에, 고도가 높아질수록 여러 기지국으로부터의 간섭이 증가하고 핸드오버가 빈번하게 발생하여 지연 시간(latency)이 증가하고 대역폭이 불안정해지는 특성이 있다.14 따라서 실시간 FPV 조종과 같이 수십 밀리초(ms) 수준의 초저지연이 요구되는 임무보다는, 상대적으로 지연 시간에 덜 민감한 광역 감시, 지도 제작, 물류 배송 등의 BVLOS 임무에 더 적합하다.

### 1.3  물리 계층 최적화

안정적이고 장거리 통신이 가능한 데이터링크를 구축하기 위해서는 네트워크 프로토콜뿐만 아니라 물리 계층(Physical Layer)에 대한 최적화가 필수적이다. 이는 안테나 선택과 배치, 그리고 무선 주파수(RF) 파라미터의 정밀한 제어를 포함한다.

#### 1.3.1  안테나 공학

- **이득(Gain)과 빔폭(Beamwidth)의 상충 관계:** 안테나의 이득(Gain)은 특정 방향으로 전파 에너지를 집중시키는 능력을 나타내며, dBi(decibels relative to an isotropic antenna) 단위로 측정된다.18 이득이 높을수록 신호가 더 멀리 도달하지만, 에너지가 집중되는 각도인 빔폭(Beamwidth)은 반대로 좁아진다.19 따라서 GCS와 같이 고정된 위치에서는 이득이 높은 지향성 안테나(예: 야기, 파라볼릭 그리드 안테나)를 사용하여 드론 방향으로 신호를 집중시키는 것이 유리하다.20 반면, 비행 중 자세가 계속 변하는 드론 기체에는 모든 방향으로 비교적 균일한 신호를 송수신할 수 있는 무지향성(Omnidirectional) 안테나를 사용하는 것이 일반적이다.

- **링크 버짓(Link Budget) 계산:** 데이터링크 설계 시, 통신 가능 거리를 예측하기 위해 링크 버짓을 계산하는 것이 중요하다. 이는 송신 전력, 송/수신 안테나 이득, 그리고 거리에 따른 신호 손실 등을 종합적으로 고려하는 과정이다. 주파수와 거리에 따른 신호 감쇠를 나타내는 자유 공간 경로 손실(Free Space Path Loss, FSPL)은 다음 공식으로 계산할 수 있다.
  $$
  FSPL(dB)=20 \log_{10}(d) + 20 \log _{10}(f) + 20 \log _{10} (c4π)
  $$
  여기서 $d$는 송수신기 간의 거리(m), $f$는 주파수(Hz), $c$는 빛의 속도(m/s)를 의미한다. 이 계산을 통해 특정 하드웨어 구성으로 목표하는 통신 거리를 달성할 수 있는지 사전에 검증할 수 있다.

#### 1.3.2  RF 파라미터 제어

OpenWRT는 리눅스 커널이 제공하는 강력한 무선 제어 도구를 통해 RF 파라미터를 세밀하게 조정할 수 있는 환경을 제공한다.

- **`iw` 및 `iwinfo` 명령어 활용:** `iw`는 무선 장치를 설정하고 관리하는 표준 리눅스 명령어이며, `iwinfo`는 OpenWRT에서 제공하는 보다 사용자 친화적인 정보 조회 도구이다. 이 명령어들을 사용하여 현재 연결 상태, 채널, 신호 강도(RSSI), 노이즈 레벨, 비트레이트 등 상세한 정보를 실시간으로 확인할 수 있다.22
- **송신 출력(Tx Power) 제어:** `iw dev <devname> set txpower fixed <value_in_mBm>` 또는 `uci set wireless.radio0.txpower=<value_in_dBm>` 명령어를 통해 무선 모듈의 송신 출력을 조절할 수 있다. 출력을 높이면 통신 거리가 늘어나지만, 전력 소모가 증가하고 다른 무선 장비에 대한 간섭을 유발할 수 있다. 반대로 출력을 낮추면 여러 드론이 동일 채널을 사용할 때 상호 간섭을 줄일 수 있다. 단, 실제 출력은 국가별 전파 규제 및 하드웨어 드라이버의 제약에 따라 설정값과 다를 수 있으며, 일부 하드웨어는 출력 조절 기능을 지원하지 않을 수도 있다.22
- **RSSI 기반 링크 품질 관리:** `iw dev <devname> cqm rssi <threshold_in_dBm>` 명령어를 사용하여 연결 품질 모니터링(Connection Quality Monitoring, CQM) 기능을 활성화할 수 있다.25 이를 통해 수신 신호 강도(RSSI)가 설정된 임계값 이하로 떨어지면, 커널이 상위 계층 애플리케이션에 이벤트를 통지하도록 설정할 수 있다. 이 기능은 다중 링크를 사용하는 시스템에서 신호 품질 저하를 감지하고 다른 안정적인 링크로 능동적으로 전환(roaming)하는 로직을 구현하는 데 활용될 수 있다.

## 2.  하드웨어 선정 및 시스템 구축

OpenWRT 기반 데이터링크의 성능은 그 위에서 동작하는 하드웨어의 성능과 직결된다. 특히 드론 환경에서는 크기(Size), 무게(Weight), 전력(Power), 비용(Cost)을 종합적으로 고려하는 SWaP-C 최적화가 매우 중요하다.

### 2.1  SWaP-C를 고려한 온보드 라우터 하드웨어 선정

#### 2.1.1  SBC(Single-Board Computer) 플랫폼 비교

다양한 SBC가 OpenWRT를 구동할 수 있지만, 드론 적용을 위해서는 몇 가지 핵심 요소를 고려해야 한다.

- **Raspberry Pi Compute Module 4 (CM4):** 강력한 4코어 ARM Cortex-A72 CPU, 최대 8GB RAM, 그리고 eMMC 또는 SD 카드를 통한 유연한 스토리지 옵션을 제공한다.26 CM4 자체는 작은 모듈 형태이지만, 다양한 I/O를 제공하는 캐리어 보드와 결합하여 맞춤형 시스템을 구축하는 데 매우 유리하다. 특히 듀얼 기가비트 이더넷 포트를 갖춘 캐리어 보드와 함께 사용하면, 비행 컨트롤러와의 유선 연결과 고대역폭 센서(예: LiDAR) 데이터 처리를 동시에 수행하는 강력한 온보드 라우터를 만들 수 있다.27
- **GL-iNet 제품군:** GL-iNet은 OpenWRT를 기반으로 자체적인 사용자 인터페이스를 추가한 상용 라우터 제품을 전문적으로 생산하는 기업이다. GL-MT6000 (Flint 2)이나 GL-MT300N (Mango)과 같은 제품들은 컴팩트한 크기와 안정적인 성능을 제공하며, 공식 OpenWRT 펌웨어로 쉽게 교체할 수 있어 빠른 프로토타이핑 및 개발에 매우 적합하다.29
- **기타 SBC (Banana Pi, Orange Pi 등):** 이들 플랫폼은 특정 요구사항에 맞는 독특한 하드웨어 구성을 제공하는 경우가 많다. 예를 들어, Orange Pi R1 Plus는 초소형 크기에 듀얼 이더넷 포트를 내장하고 있으며 32, Banana Pi BPI-R4는 10Gbps SFP+ 포트와 Wi-Fi 7 지원을 목표로 하는 고성능 보드이다.33 하지만 이러한 비주류 SBC를 선택할 때는 공식 OpenWRT 지원 여부, 커뮤니티의 활성도, 그리고 드라이버 안정성을 반드시 사전에 철저히 검증해야 한다.34

#### 2.1.2  OpenWRT 구동을 위한 최소/권장 사양

과거에는 4MB Flash와 32MB RAM을 가진 장치에서도 OpenWRT가 구동되었지만, 시스템의 기능이 점차 확장됨에 따라 요구 사양이 높아졌다. 최신 OpenWRT 릴리즈(18.06 이후)를 안정적으로 사용하고 LuCI 웹 인터페이스와 추가적인 패키지를 설치할 공간을 확보하기 위해서는 **최소 16MB의 Flash 저장 공간과 128MB의 RAM**이 강력하게 권장된다. 8MB Flash / 64MB RAM 장치는 더 이상 신규 구매가 권장되지 않으며, 기능 제약이 따를 수 있다.35

### 2.2  구축 사례 연구: Raspberry Pi CM4와 Dual-GbE 캐리어 보드 기반 시스템

이론적 논의를 넘어 실제 시스템을 구축하는 과정을 통해 이해를 돕고자, Raspberry Pi CM4와 Seeed Studio의 Dual Gigabit Ethernet Carrier Board를 기반으로 한 데이터링크 시스템 구축 사례를 단계별로 설명한다.

#### 2.2.1  하드웨어 조립 및 OpenWRT 펌웨어 설치

1. **하드웨어 결합:** Raspberry Pi CM4 모듈을 캐리어 보드의 지정된 소켓에 단단히 결합한다. Wi-Fi/Bluetooth가 내장된 CM4 모델의 경우, U.FL 커넥터에 외부 안테나를 연결한다.27
2. **펌웨어 다운로드:** OpenWRT 공식 웹사이트의 'Table of Hardware' 또는 'Firmware Selector' 페이지로 이동하여 Raspberry Pi 4/CM4용 최신 안정 버전 또는 스냅샷 빌드의 `sysupgrade` 이미지를 다운로드한다.26
3. **펌웨어 플래싱:** 다운로드한 이미지 파일(.img.gz)의 압축을 해제한다. BalenaEtcher나 Raspberry Pi Imager와 같은 도구를 사용하여 이미지를 MicroSD 카드에 기록한다. CM4가 eMMC 스토리지를 사용하는 경우, `rpiboot` 도구를 사용하여 CM4를 USB 저장 장치로 인식시킨 후 이미지를 플래싱한다.38

#### 2.2.2  초기 네트워크 및 시스템 설정

1. **최초 부팅 및 접속:** 펌웨어가 설치된 저장 매체를 삽입하고 CM4에 전원을 공급한다. 컴퓨터를 캐리어 보드의 LAN 포트 중 하나에 유선으로 연결한다. 컴퓨터의 네트워크 설정을 DHCP로 설정하면, OpenWRT로부터 자동으로 IP 주소를 할당받는다. 웹 브라우저에서 `http://192.168.1.1`로 접속하거나, SSH 클라이언트를 사용하여 `ssh root@192.168.1.1` 명령으로 접속한다.34
2. **기본 시스템 설정:**
   - 최초 접속 시 비밀번호가 설정되어 있지 않으므로, 즉시 `passwd` 명령어를 실행하여 루트 계정의 비밀번호를 설정한다.
   - `opkg update` 명령어를 실행하여 패키지 목록을 최신 상태로 갱신한다.
   - `opkg install luci` 명령어를 실행하여 LuCI 웹 인터페이스를 설치한다. 설치 후 웹 브라우저를 통해 다시 접속하면 그래픽 인터페이스로 시스템을 관리할 수 있다.40
   - 필요에 따라 시간대(Timezone), 시스템 이름(Hostname) 등을 설정한다.

이 과정을 통해 드론에 탑재할 수 있는 강력하고 유연한 맞춤형 라우터의 기본 골격이 완성된다. 다음 장에서는 이 하드웨어 플랫폼 위에 실제 데이터링크 기능을 구현하는 소프트웨어 설정 방법을 다룬다.

**Table 1: UAV용 SBC 하드웨어 비교**

| 특성             | Raspberry Pi CM4 (+ Carrier)                     | GL-iNet GL-MT6000                        | Orange Pi R1 Plus                        | Banana Pi BPI-R4                     |
| ---------------- | ------------------------------------------------ | ---------------------------------------- | ---------------------------------------- | ------------------------------------ |
| **CPU**          | Broadcom BCM2711 (4x Cortex-A72 @ 1.5GHz)        | MediaTek MT7986 (4x Cortex-A53 @ 2.0GHz) | Rockchip RK3328 (4x Cortex-A53 @ 1.5GHz) | MediaTek MT7988A (4x Cortex-A73)     |
| **RAM**          | 1/2/4/8 GB LPDDR4                                | 1 GB DDR4                                | 1 GB DDR4                                | 4 GB DDR4                            |
| **Flash**        | 8/16/32 GB eMMC 또는 MicroSD                     | 8 GB eMMC                                | MicroSD                                  | 8 GB eMMC + MicroSD                  |
| **이더넷 포트**  | 캐리어 보드에 따라 다름 (일반적으로 1-2x 1GbE)   | 2x 2.5GbE, 4x 1GbE                       | 2x 1GbE                                  | 2x 10GbE SFP+, 4x 1GbE               |
| **Wi-Fi**        | Cypress CYW43455 (802.11ac)                      | MediaTek MT7986 (802.11ax, Wi-Fi 6)      | 외장 USB 동글 필요                       | MediaTek MT7977B (802.11be, Wi-Fi 7) |
| **USB 포트**     | 캐리어 보드에 따라 다름 (일반적으로 USB 2.0/3.0) | 1x USB 3.0                               | 1x USB 2.0                               | 1x USB 3.2                           |
| **확장 슬롯**    | PCIe Gen 2 x1                                    | -                                        | -                                        | 2x M.2, 2x miniPCIe                  |
| **크기 (mm)**    | 55 x 40 (모듈) + 캐리어 보드                     | 233 x 137 x 53                           | 56 x 57                                  | 148 x 100.5                          |
| **무게 (g)**     | ~30 (모듈) + 캐리어 보드                         | 761                                      | ~52                                      | 미정                                 |
| **전력 소모**    | 가변적 (일반적으로 3-7W)                         | ~10W (유휴)                              | ~2.5W                                    | 미정                                 |
| **OpenWRT 지원** | 공식 지원 우수 26                                | 공식 지원 (OEM 펌웨어 기반) 29           | 공식 지원 32                             | 개발 중 33                           |
| **예상 비용**    | 중-고 (모듈 + 캐리어)                            | 중                                       | 저                                       | 고                                   |

## 3.  데이터링크 프로토콜 및 소프트웨어 구현

하드웨어 플랫폼이 준비되면, 그 위에 실제 데이터 통신을 담당하는 소프트웨어를 구현해야 한다. OpenWRT는 강력한 패키지 관리 시스템과 유연한 설정 인터페이스를 통해 MAVLink 원격 측정, 저지연 영상 스트리밍, 보안 터널링 등 복잡한 요구사항을 체계적으로 구현할 수 있는 환경을 제공한다.

### 3.1  MAVLink 연동 및 IP 기반 라우팅

드론의 상태 정보와 제어 명령을 담고 있는 MAVLink 프로토콜을 IP 네트워크를 통해 안정적으로 전송하는 것은 데이터링크의 가장 핵심적인 기능이다. 비행 컨트롤러(FC)는 대부분 UART(직렬) 포트를 통해 MAVLink 데이터를 출력하므로, 이를 IP 패킷으로 변환하고 라우팅하는 과정이 필요하다.

#### 3.1.1  직렬-네트워크 브릿징: `ser2net`

가장 간단하고 전통적인 방법은 `ser2net` 유틸리티를 사용하는 것이다. `ser2net`은 특정 직렬 포트로 들어오는 데이터를 지정된 TCP 또는 UDP 포트로 그대로 전달해주는 역할을 한다.41

`opkg install ser2net` 명령으로 설치 후, `/etc/ser2net.conf` 설정 파일을 다음과 같이 수정한다.41

```
# <TCP/UDP port>:<state>:<timeout>:<device>:<options>
5760:raw:0:/dev/ttyS1:115200 8DATABITS NONE 1STOPBIT
```

위 설정은 FC가 연결된 시리얼 포트 `/dev/ttyS1`의 데이터를 115200 보드레이트로 읽어, 네트워크의 TCP 포트 5760으로 전달하라는 의미이다. GCS(예: Mission Planner, QGroundControl)에서는 TCP 연결을 선택하고 OpenWRT 장치의 IP 주소와 포트 5760을 입력하면 MAVLink 연결이 수립된다. 이 방식은 구현이 매우 간단하지만, 하나의 GCS만 연결할 수 있고 MAVLink 메시지의 내용을 해석하지 못하는 한계가 있다.

#### 3.1.2  지능형 라우팅: `mavlink-router`

보다 복잡한 시스템, 예를 들어 GCS와 온보드 컴패니언 컴퓨터 애플리케이션이 동시에 MAVLink 데이터에 접근해야 하는 경우, `mavlink-router`가 훨씬 강력한 솔루션을 제공한다.42 C 언어로 작성되어 매우 가볍고 효율적인 `mavlink-router`는 MAVLink 메시지를 이해하고, 메시지의 발신지(System ID, Component ID)와 목적지 정보를 기반으로 지능적인 라우팅을 수행한다.44

`opkg`를 통해 설치 후, `/etc/mavlink-router/main.conf` 파일을 통해 라우팅 규칙을 정의한다.43

```Ini, TOML
[General]
MavlinkDialect = ardupilotmega
DebugLogLevel = info

[UartEndpoint]
Device = /dev/ttyS0
Baud = 921600


Mode = Normal
Address = 192.168.1.100  # GCS의 IP 주소
Port = 14550

[UdpEndpoint OnboardApp]
Mode = Normal
Address = 127.0.0.1      # 로컬호스트
Port = 14551
```

이 설정은 FC가 연결된 `/dev/ttyS0`로부터 MAVLink 데이터를 수신하여, GCS(192.168.1.100:14550)와 로컬에서 실행 중인 온보드 애플리케이션(127.0.0.1:14551) 양쪽으로 동시에 데이터를 전달한다. 또한 GCS나 온보드 앱에서 발생한 MAVLink 명령은 FC로 정확하게 전달된다. `mavp2p`는 `mavlink-router`의 대안으로 고려할 수 있는 또 다른 경량 라우터이다.48

### 3.2  저지연 영상 스트리밍 구현

고화질 영상을 최소한의 지연 시간으로 전송하는 것은 현대 드론 데이터링크의 핵심 요구사항이다. OpenWRT에서는 GStreamer라는 강력한 멀티미디어 프레임워크를 사용하여 유연하고 효율적인 영상 스트리밍 파이프라인을 구축할 수 있다.

#### 3.2.1  GStreamer 파이프라인 설계

GStreamer는 다양한 미디어 처리 요소(element)들을 파이프라인(`!`)으로 연결하여 데이터 흐름을 정의하는 방식으로 동작한다.

- **송신 측 (드론) 파이프라인 예시:**

  ```Bash
  gst-launch-1.0 libcamerasrc! video/x-raw,width=1280,height=720,framerate=30/1! v4l2h264enc extra-controls="controls,video_bitrate=4000000;"! h264parse config-interval=-1! rtph264pay! udpsink host=<GCS_IP> port=5000
  ```

  이 파이프라인은 `libcamerasrc`를 통해 카메라 입력을 받아 49, 하드웨어 가속 H.264 인코더(`v4l2h264enc`)로 압축하고, RTP(Real-time Transport Protocol) 패킷으로 변환(`rtph264pay`)한 뒤, UDP를 통해 GCS로 전송(`udpsink`)한다.49

- **수신 측 (GCS) 파이프라인 예시:**

  ```Bash
  gst-launch-1.0 udpsrc port=5000 caps="application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)H264"! rtpjitterbuffer! rtph264depay! h264parse! avdec_h264! videoconvert! autovideosink sync=false
  ```

  수신 측에서는 UDP 포트 5000으로 들어오는 RTP 패킷을 받아(`udpsrc`), 네트워크 지터를 보상하고(`rtpjitterbuffer`), H.264 데이터로 변환(`rtph264depay`, `h264parse`)한 뒤, 디코딩(`avdec_h264`)하여 화면에 출력(`autovideosink`)한다.49

#### 3.2.2  지연 시간 최소화 기법

영상 스트리밍의 엔드-투-엔드 지연 시간은 캡처, 인코딩, 전송, 디코딩, 렌더링 등 파이프라인의 모든 단계에서 누적된다. 이를 최소화하기 위한 기법은 다음과 같다.

- **하드웨어 가속 인코딩/디코딩:** CPU를 사용하는 소프트웨어 인코더(`x264enc`)는 유연성이 높지만 상당한 지연을 유발한다. SBC가 제공하는 하드웨어 가속 인코더(예: Raspberry Pi의 `v4l2h264enc`, Nvidia Jetson의 `nvv4l2h264enc`)를 사용하면 CPU 부하를 크게 줄이고 인코딩 단계의 지연 시간을 수십 밀리초 단축할 수 있다.49

- **인코더 파라미터 튜닝:** H.264 인코더 설정에서 `tune=zerolatency` 옵션을 사용하고, 예측을 위해 미래 프레임을 참조하는 B-프레임(B-frame)을 비활성화(`bframes=0`)하며, 키프레임(I-frame) 간격을 짧게 설정하는 것이 지연 시간 감소에 효과적이다.49

- **`rtpjitterbuffer` 튜닝:** `rtpjitterbuffer`는 네트워크 지터(패킷 도착 시간의 불규칙성)를 흡수하여 부드러운 재생을 돕지만, 본질적으로 버퍼링을 통해 지연 시간을 증가시킨다.52 기본 버퍼 크기(보통 200ms)는 FPV와 같은 실시간 애플리케이션에는 너무 크다. 

  `latency` 속성을 20-50ms 정도로 낮추어 지연과 안정성 사이의 균형을 맞추거나, 네트워크 상태가 매우 양호한 경우 버퍼를 완전히 비활성화하여 지연을 최소화할 수 있다.50

- **네트워크 파라미터 최적화:** 무선 링크에서는 패킷 손실이 발생하기 쉽다. UDP 패킷의 크기가 네트워크의 MTU(Maximum Transmission Unit)를 초과하면 패킷이 단편화되어 손실 가능성이 커진다. `udpsink`의 MTU 설정을 1400바이트 이하로 낮추어 단편화를 방지하는 것이 영상 품질 저하를 막는 데 도움이 될 수 있다.50

### 3.3  보안 및 신뢰성 강화

임무 데이터의 기밀성 보장과 통신 두절 방지는 전문적인 드론 운용의 필수 조건이다. OpenWRT는 VPN과 다중 WAN 관리 기능을 통해 이러한 요구사항을 충족시킬 수 있는 강력한 도구를 제공한다.

#### 3.3.1  WireGuard VPN 터널링

WireGuard는 최신 암호화 기술을 사용하면서도 코드베이스가 매우 작고 성능이 뛰어나 임베디드 장치에 적합한 VPN 프로토콜이다.55 OpenWRT에 WireGuard를 설정하면 드론과 GCS 간의 모든 데이터(MAVLink, 영상, 제어 신호)를 강력하게 암호화하여 중간자 공격(Man-in-the-middle attack)이나 데이터 도청으로부터 완벽하게 보호할 수 있다. 

`uci`를 사용한 서버 측 설정 단계는 다음과 같다.56

```Bash
# 1. 패키지 설치
opkg update
opkg install wireguard-tools kmod-wireguard

# 2. 키 생성
wg genkey | tee privatekey | wg pubkey > publickey

# 3. 네트워크 인터페이스 설정
uci set network.wg0='interface'
uci set network.wg0.proto='wireguard'
uci set network.wg0.private_key='<서버의_비밀키>'
uci set network.wg0.listen_port='51820'
uci add_list network.wg0.addresses='10.0.10.1/24'

# 4. 피어(클라이언트) 설정
uci set network.peer_gcs='wireguard_wg0'
uci set network.peer_gcs.public_key='<클라이언트의_공개키>'
uci add_list network.peer_gcs.allowed_ips='10.0.10.2/32'
uci set network.peer_gcs.persistent_keepalive='25'

# 5. 방화벽 설정
uci set firewall.wg_zone='zone'
uci set firewall.wg_zone.name='wg'
uci set firewall.wg_zone.input='ACCEPT'
uci set firewall.wg_zone.output='ACCEPT'
uci set firewall.wg_zone.forward='ACCEPT'
uci set firewall.wg_zone.network='wg0'
uci set firewall.wg_fwd='forwarding'
uci set firewall.wg_fwd.src='wg'
uci set firewall.wg_fwd.dest='lan'
uci commit
/etc/init.d/network restart
/etc/init.d/firewall restart
```

#### 3.3.2  `mwan3`를 이용한 이중화 링크 페일오버

BVLOS 임무 중 주 통신 링크(예: 장거리 Wi-Fi)가 장애물이나 전파 간섭으로 인해 끊기는 상황은 매우 치명적이다. `mwan3` 패키지는 이러한 상황에 대비하여 여러 개의 WAN 인터페이스를 관리하고, 주 링크에 장애가 발생했을 때 자동으로 예비 링크(예: LTE)로 모든 트래픽을 전환하는 페일오버(Failover) 기능을 제공한다.57

설정의 핵심은 `/etc/config/network`에서 각 인터페이스의 우선순위를 `metric` 값으로 지정하고(낮을수록 우선순위 높음), `/etc/config/mwan3`에서 각 링크의 상태를 주기적으로 확인할 방법(ping 대상)과 전환 정책을 정의하는 것이다.57 이를 통해 드론은 통신 환경의 변화에 능동적으로 대응하여 임무 지속성을 획기적으로 향상시킬 수 있다.

이처럼 OpenWRT는 MAVLink 라우팅, 영상 스트리밍, VPN, 링크 이중화와 같은 개별 기술들을 단순하게 나열하는 것이 아니라, UCI라는 일관된 설정 프레임워크와 강력한 리눅스 네트워크 스택을 통해 이들을 하나의 '통합된 고신뢰성 통신 플랫폼'으로 융합할 수 있는 기반을 제공한다. 사용자는 마치 레고 블록을 조립하듯 필요한 기능들을 선택하고 조합하여, 자신의 특정 임무 요구사항에 완벽하게 부합하는 맞춤형 통신 시스템을 설계하고 구현할 수 있다. 이는 기성품 솔루션에서는 결코 얻을 수 없는 OpenWRT만의 독보적인 가치이다.

## 4.  성능 비교 분석 및 종합 고찰

OpenWRT 기반의 DIY 데이터링크는 최고의 유연성과 확장성을 제공하지만, 이것이 모든 시나리오에서 최적의 선택임을 의미하지는 않는다. 임무의 목표, 예산, 개발 기간, 운용 인력의 기술 수준 등 다양한 요소를 고려하여 상용 솔루션과의 장단점을 객관적으로 비교하고 전략적인 선택을 내리는 것이 중요하다.

### 4.1  OpenWRT DIY 데이터링크 vs. 상용 솔루션

**성능, 비용, 개발 난이도, 확장성 분석**

- **OpenWRT DIY:** 이 방식의 가장 큰 장점은 '완전한 제어'이다. 사용자는 하드웨어부터 네트워크 토폴로지, 보안 프로토콜에 이르기까지 모든 요소를 직접 선택하고 최적화할 수 있다.59 이는 특정 임무에 완벽하게 맞는 시스템을 구축할 수 있게 해주며, 장기적으로는 하드웨어 비용을 절감하고 기술 내재화를 이룰 수 있다는 장점이 있다. 하지만 이를 위해서는 네트워킹, 임베디드 리눅스, RF 공학에 대한 깊이 있는 지식이 요구되며, 안정적인 시스템을 구축하기까지 상당한 개발 시간과 노력이 필요하다는 명백한 단점이 존재한다.
- **Herelink / SIYI:** 이들 제품은 RC 조종, 원격 측정, 영상 전송 기능을 하나의 패키지로 통합하여 제공한다.61 MAVLink와 호환되며 상대적으로 긴 통신 거리를 제공하여 준전문가급 사용자들에게 인기가 있다. 개발 과정이 매우 단순화된다는 장점이 있지만, 폐쇄적인 하드웨어 및 소프트웨어 생태계에 종속되어 기능 확장이 어렵고, 문제가 발생했을 때 제조사의 기술 지원에 의존해야 하는 한계가 있다.4
- **DJI O3 Air Unit:** 현존하는 FPV(First Person View) 시스템 중 최고의 영상 품질과 가장 낮은 지연 시간을 제공한다.6 이는 DJI의 독자적인 O3+ 전송 기술에 기반하며, 특히 고속으로 비행하는 레이싱 드론이나 시네마틱 FPV 분야에서는 거의 대체 불가능한 성능을 보여준다. 그러나 DJI 생태계에 완전히 종속되어 있으며, MAVLink와 같은 표준 프로토콜과의 호환성이 제한적이고, 사용자 정의 기능 추가는 거의 불가능하다.65
- **4G/LTE:** 통신사 인프라를 활용하여 거리 제한을 사실상 없애는 BVLOS 임무의 핵심 기술이다.15 하지만 통신망의 상태에 따라 지연 시간과 대역폭이 크게 변동하며, 일반적으로 100ms를 상회하는 지연 시간은 실시간 FPV 조종에는 부적합하다.14 자율 비행 기반의 광역 감시나 물류 배송에 더 적합한 솔루션이다.

데이터링크 솔루션을 선택하는 것은 단순히 기술적 스펙을 비교하는 것을 넘어, 프로젝트의 '철학'과 '목표'에 부합하는 도구를 선택하는 과정이다. 만약 목표가 최고의 영상 품질과 반응성으로 FPV 비행을 즐기는 것이라면 DJI O3가 정답에 가깝다. 반면, 다수의 드론이 상호작용하며 복잡한 임무를 수행하는 연구 개발 프로젝트라면, OpenWRT가 제공하는 무한한 확장성과 제어 능력이 그 어떤 상용 솔루션보다 더 큰 가치를 제공할 것이다.

**Table 2: 데이터링크 솔루션 비교 분석**

| 구분                  | OpenWRT DIY (Wi-Fi)                     | OpenWRT DIY (LTE)                   | Herelink V1.1                 | DJI O3 Air Unit              | SiK Radio (915MHz)   |
| --------------------- | --------------------------------------- | ----------------------------------- | ----------------------------- | ---------------------------- | -------------------- |
| **주파수 대역**       | 2.4 / 5 GHz                             | 셀룰러 (Bands)                      | 2.4 GHz                       | 5.8 GHz (TX)                 | 915 MHz (ISM)        |
| **최대 통신 거리**    | 1-10+ km (안테나/출력 의존)             | 통신망 커버리지 내 무제한           | 12-20 km 61                   | 2-10 km 64                   | 1-5+ km              |
| **비디오 해상도/FPS** | 사용자 정의 (e.g., 1080p@30fps)         | 사용자 정의 (대역폭 의존)           | 1080p@30/60fps 66             | 1080p@100fps 67              | 지원 안함            |
| **엔드-투-엔드 지연** | 50-150 ms (최적화 시)                   | 100-300+ ms 14                      | ~110 ms 66                    | < 30 ms 64                   | N/A                  |
| **최대 대역폭**       | 수십-수백 Mbps                          | 수십 Mbps (가변적)                  | 10/20 MHz 채널 61             | 50 Mbps 64                   | ~250 kbps            |
| **주요 장점**         | 최고의 유연성, 확장성, 비용 효율성      | 무제한 통신 거리, BVLOS 가능        | 통합 솔루션, 긴 통신 거리     | 최고의 영상 품질, 초저지연   | 저렴, 간단, 안정적   |
| **주요 단점**         | 높은 개발 난이도, 안정성 확보 노력 필요 | 높은 지연, 가변적 성능, 통신비 발생 | 폐쇄적 생태계, 기능 확장 제한 | DJI 생태계 종속, 확장성 부재 | 저대역폭 (영상 불가) |
| **예상 비용**         | 저-중                                   | 중 (모뎀+통신비)                    | 고                            | 중-고                        | 저                   |
| **개발 난이도**       | 높음                                    | 높음                                | 낮음                          | 낮음                         | 낮음                 |
| **확장성/유연성**     | 매우 높음                               | 매우 높음                           | 낮음                          | 매우 낮음                    | 매우 낮음            |

### 4.2  통신 방식에 따른 성능 트레이드오프

각 통신 방식은 고유한 물리적 특성으로 인해 명확한 장단점을 가진다. Wi-Fi는 높은 대역폭을 제공하여 고화질 영상 전송에 유리하지만, 주파수가 높아 직진성이 강하고 장애물 투과력이 약해 LOS(Line of Sight) 환경이 중요하다. 4G/LTE는 기존 인프라를 활용하여 비가시 환경에서도 통신이 가능하지만, 네트워크 혼잡도와 기지국과의 거리에 따라 성능이 크게 좌우된다. B.A.T.M.A.N.-adv와 같은 메시 네트워킹은 노드 간 협력을 통해 커버리지를 확장하고 링크 안정성을 높이지만, 홉(hop)이 늘어날수록 지연 시간이 누적되고 전체 가용 대역폭이 감소하는 단점이 있다.9 최적의 데이터링크는 단일 통신 방식에 의존하기보다, 이들을 융합한 하이브리드 아키텍처를 통해 달성될 수 있다.

### 4.3  최종 제언: 시나리오별 최적의 데이터링크 아키텍처

이상의 분석을 바탕으로, 대표적인 드론 운용 시나리오에 따른 최적의 OpenWRT 기반 데이터링크 아키텍처를 다음과 같이 제안한다.

- **단일 드론, 단거리 고화질 영상 전송 (예: 시네마틱 FPV, 시설 점검):**
  - **아키텍처:** 고출력 5GHz Wi-Fi 모듈과 고이득 안테나를 사용한 기본 Point-to-Point 링크.
  - **핵심 기술:** 하드웨어 가속 H.264 인코딩, 저지연 GStreamer 파이프라인 튜닝.
  - **기대 효과:** 100ms 이하의 낮은 지연 시간으로 1080p급 영상을 안정적으로 전송.
- **드론 군집, 중거리 협력 임무 (예: 정밀 농업, 환경 모니터링):**
  - **아키텍처:** 5GHz 대역을 사용한 802.11s + B.A.T.M.A.N.-adv 기반의 자율 회복 메시 네트워크.
  - **핵심 기술:** `kmod-batman-adv`, `batctl`을 이용한 동적 라우팅, `mavlink-router`를 이용한 군집 내 MAVLink 데이터 브로드캐스팅.
  - **기대 효과:** 일부 드론의 링크가 단절되어도 전체 군집의 통신망은 유지되며, 모든 드론이 서로의 상태 정보를 실시간으로 공유.
- **장거리 감시 및 정찰 (BVLOS) (예: 광역 실종자 수색, 국경 감시):**
  - **아키텍처:** 주 링크로 장거리 Wi-Fi(예: 900MHz 또는 저주파 2.4GHz), 예비 링크로 4G/LTE 모뎀을 `mwan3`로 결합한 이중화 링크 구성.
  - **핵심 기술:** `mwan3`를 이용한 자동 페일오버, WireGuard VPN을 통한 종단 간 데이터 암호화.
  - **기대 효과:** 통신 두절 위험을 최소화하고, 공용망을 사용하더라도 임무 데이터의 보안을 완벽하게 보장하는 최고 수준의 신뢰성 확보.

## 5. 결론: OpenWRT 기반 UAV 데이터링크의 현재와 미래

본 보고서는 OpenWRT를 드론 데이터링크에 적용하는 기술적 원리부터 실제 구축 방법, 그리고 성능 분석에 이르기까지 다각적인 고찰을 수행했다. 결론적으로 OpenWRT는 기존의 상용 데이터링크 솔루션이 제공할 수 없는 압도적인 유연성, 제어 능력, 그리고 비용 효율성을 제공함으로써, 차세대 UAV 통신 시스템을 위한 가장 강력한 플랫폼 중 하나임이 명백하다. 사용자는 리눅스 네트워크 스택의 모든 기능을 활용하여 메시 네트워킹, 셀룰러 연동, VPN 보안, 링크 이중화 등 고도의 기능을 자신의 임무에 맞게 '설계'하고 '통합'할 수 있다.

물론 이러한 강력한 기능에는 상응하는 책임이 따른다. 높은 초기 학습 곡선, 수많은 하드웨어 옵션 간의 호환성 문제, 그리고 직접 시스템의 안정성을 확보해야 하는 지속적인 노력은 OpenWRT 기반 개발의 주요 기술적 과제이다. 그럼에도 불구하고, 오픈소스 커뮤니티의 집단 지성과 방대한 소프트웨어 생태계는 이러한 어려움을 극복할 수 있는 훌륭한 자산을 제공한다.

OpenWRT 기반 UAV 데이터링크의 미래는 더욱 밝다. 5G 모뎀이 점차 소형화되고 저렴해짐에 따라, 이를 OpenWRT 시스템에 통합하여 수십 밀리초 수준의 초저지연과 기가비트급의 초고대역폭을 BVLOS 환경에서 실현하는 것이 가능해질 것이다. 더 나아가, 온보드 컴퓨터의 성능 향상과 함께 AI/ML 기술을 접목하여, 실시간으로 전파 환경을 분석하고 링크 품질을 예측하여 최적의 통신 채널과 라우팅 경로를 스스로 찾아내는 '지능형 데이터링크'의 등장을 기대할 수 있다. OpenWRT는 이러한 미래 기술들이 구현될 가장 유연하고 개방적인 실험장이자 플랫폼으로서, 완전 분산형 자율 군집 통신 프로토콜 개발과 같은 첨단 연구를 선도하며 UAV 기술의 발전에 지속적으로 기여할 것이다.

#### 참고 자료

1. Flight Computer with Built-in OpenWRT WiFi Router! - Blogs ..., accessed August 11, 2025, https://diydrones.com/profiles/blogs/flight-computer-with-built-in-openwrt-wifi-router
2. OpenWrt - Wikipedia, accessed August 11, 2025, https://en.wikipedia.org/wiki/OpenWrt
3. [OpenWrt Wiki] Documentation, accessed August 11, 2025, https://openwrt.org/docs/start
4. Stable, high quality telemetry and RC link options for proffesional aplications - Radios, accessed August 11, 2025, https://discuss.ardupilot.org/t/stable-high-quality-telemetry-and-rc-link-options-for-proffesional-aplications/66237
5. Herelink or siyi unirc 7 pro? - Radios - ArduPilot Discourse, accessed August 11, 2025, https://discuss.ardupilot.org/t/herelink-or-siyi-unirc-7-pro/126975
6. Is there anything comparable to the 03 air unit that's not DJI? : r/fpv - Reddit, accessed August 11, 2025, https://www.reddit.com/r/fpv/comments/16n9d8k/is_there_anything_comparable_to_the_03_air_unit/
7. [OpenWrt Wiki] Wi-Fi /etc/config/wireless, accessed August 11, 2025, https://openwrt.org/docs/guide-user/network/wifi/basic
8. Topic: adhoc mode guide - OpenWrt Forum Archive, accessed August 11, 2025, https://forum.archive.openwrt.org/viewtopic.php?id=56181
9. BattleMeshV4/MeshGuide - Wireless Battle of the Mesh, accessed August 11, 2025, https://battlemesh.org/BattleMeshV4/MeshGuide
10. [OpenWrt Wiki] B.A.T.M.A.N. / batman-adv - Network, accessed August 11, 2025, https://openwrt.org/docs/guide-user/network/wifi/mesh/batman
11. Mesh networking: A guide to using free and open-source software with common hardware, accessed August 11, 2025, https://cgomesu.com/blog/Mesh-networking-openwrt-batman/
12. B.A.T.M.A.N. OpenWrt configuration - open-mesh.org latest documentation, accessed August 11, 2025, https://www.open-mesh.org/doc/batman-adv/Batman-adv-openwrt-config.html
13. I've been working on an OpenWrt Router in a 30.5mm package : r/drones - Reddit, accessed August 11, 2025, https://www.reddit.com/r/drones/comments/1e7icjr/ive_been_working_on_an_openwrt_router_in_a_305mm/
14. Mobile Network Performance and Technical Feasibility of LTE-Powered Unmanned Aerial Vehicle - PMC - PubMed Central, accessed August 11, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC8073316/
15. Comparing Datalinks: Cellular Bonding vs. RF Drone - Elsight, accessed August 11, 2025, https://www.elsight.com/blog/comparing-cellular-bonding-with-rf-drone-datalinks/
16. New Product Sky Drone FPV 2 - low latency & long range HD video over 4G/LTE, accessed August 11, 2025, https://www.rcgroups.com/forums/showthread.php?2796345-Sky-Drone-FPV-2-low-latency-long-range-HD-video-over-4G-LTE
17. Is there a good way to do ultra-long-range FPV using phone signal? : r/drones - Reddit, accessed August 11, 2025, https://www.reddit.com/r/drones/comments/16yp6jp/is_there_a_good_way_to_do_ultralongrange_fpv/
18. High Gain WiFi Antennas: Maximize Signal Strength - Tesswave, accessed August 11, 2025, https://www.tesswave.com/guide-to-high-gain-wifi-antenna/
19. What is the commercially available Wi-Fi Antenna with the highest dbi? : r/wifi - Reddit, accessed August 11, 2025, https://www.reddit.com/r/wifi/comments/19d0ea8/what_is_the_commercially_available_wifi_antenna/
20. 26 dBi Grid Parabolic Wifi Antenna 2.4 GHz Ultra High Gain and Hi, accessed August 11, 2025, https://simplewifi.com/products/parabolic-grid
21. Creating 1.5 km Wi-Fi Link for UAV - Antenna Polarizations | Ubiquiti Community, accessed August 11, 2025, https://community.ui.com/questions/Creating-1-5-km-Wi-Fi-Link-for-UAV-Antenna-Polarizations/073f0136-1246-4756-81c0-b57c1f399b69
22. Topic: davidc502 1900ac 3200acm builds - OpenWrt Forum Archive, accessed August 11, 2025, https://forum.archive.openwrt.org/viewtopic.php?id=64949&p=119
23. Topic: Cannot set more than 15 dBm Tx power on Bufallo WZR-HP-AG300H - OpenWrt Forum Archive, accessed August 11, 2025, https://forum.archive.openwrt.org/viewtopic.php?id=45260
24. Adjusting the TX power level with the iw command no longer works in the latest SNAPSHOT / Issue #15968 - GitHub, accessed August 11, 2025, https://github.com/openwrt/openwrt/issues/15968
25. Configuring and Compiling the Latest Supplicant for Roaming - Developer Docs - Silicon Labs, accessed August 11, 2025, https://docs.silabs.com/wifi91xrcp/latest/wifi91xrcp-developers-guide-wifi-configuration/roaming-configuration
26. Raspberry Pi Foundation Raspberry Pi Compute Module 4 - [OpenWrt Wiki] Techdata, accessed August 11, 2025, https://openwrt.org/toh/hwdata/raspberry_pi_foundation/raspberry_pi_foundation_raspberry_pi_cm4
27. OpenWRT Router built with Raspberry Pi Compute Module 4, Dual ..., accessed August 11, 2025, https://www.seeedstudio.com/Dual-GbE-Carrier-Board-with-4GB-RAM-32GB-eMMC-RPi-CM4-Case-p-5029.html
28. Compute Module 4 | Hackaday, accessed August 11, 2025, https://hackaday.com/tag/compute-module-4/
29. [OpenWrt Wiki] GL.iNet GL-MT6000, accessed August 11, 2025, https://openwrt.org/toh/gl.inet/gl-mt6000
30. GL.iNet GL-MT300N v2 - [OpenWrt Wiki] Techdata, accessed August 11, 2025, https://openwrt.org/toh/hwdata/gl.inet/gl.inet_gl-mt300n_v2
31. GL.iNet GL-MT6000 - Questions from a noob : r/openwrt - Reddit, accessed August 11, 2025, https://www.reddit.com/r/openwrt/comments/1e9dw9w/glinet_glmt6000_questions_from_a_noob/
32. Business card-sized dual GbE SBC runs OpenWrt on Rockchip RK3328 SoC, accessed August 11, 2025, https://www.cnx-software.com/2021/04/21/business-card-sized-dual-gbe-sbc-runs-openwrt-on-rockchip-rk3328-soc/
33. Best high-performance hardware (router, SBC, x86 build) with full OpenWrt support?, accessed August 11, 2025, https://www.reddit.com/r/openwrt/comments/1ksjnzb/best_highperformance_hardware_router_sbc_x86/
34. Turn SBC Into OpenWRT Router - Catzy's Blog, accessed August 11, 2025, https://catzy007.github.io/loader.html?post=2020-09-06-Turn-SBC-Into-OpenWRT-Router
35. [OpenWrt Wiki] Supported devices, accessed August 11, 2025, https://openwrt.org/supported_devices
36. [OpenWrt Wiki] Table of Hardware, accessed August 11, 2025, https://openwrt.org/toh/start
37. [OpenWrt Wiki] Table of Hardware: Firmware downloads, accessed August 11, 2025, https://openwrt.org/toh/views/toh_fwdownload
38. How to Build Your Own OpenWRT Router with an SBC - Latest News from Seeed Studio, accessed August 11, 2025, https://www.seeedstudio.com/blog/2020/02/24/how-to-build-your-own-openwrt-router-with-an-sbc/
39. CM4 Openwrt - Waveshare Wiki, accessed August 11, 2025, https://www.waveshare.com/wiki/CM4_Openwrt
40. Guide to installing OpenWRT on the $20 Linksys LN1301/MX4300 - Reddit, accessed August 11, 2025, https://www.reddit.com/r/openwrt/comments/1fgta78/guide_to_installing_openwrt_on_the_20_linksys/
41. Mavlink Through Serial | Wiki, accessed August 11, 2025, https://wiki.cbunmanned.com/wiki/products/openwrt-uav-router/mavlink-through-serial
42. MAVLink routing with a Router software - Blog - ArduPilot Discourse, accessed August 11, 2025, https://discuss.ardupilot.org/t/mavlink-routing-with-a-router-software/82138
43. mavlink-router/mavlink-router: Route mavlink packets ... - GitHub, accessed August 11, 2025, https://github.com/mavlink-router/mavlink-router
44. MAVLink Routing in ArduPilot - Dev documentation, accessed August 11, 2025, https://ardupilot.org/dev/docs/mavlink-routing-in-ardupilot.html
45. Routing - MAVLink Guide, accessed August 11, 2025, https://mavlink.io/en/guide/routing.html
46. Communicating with Raspberry Pi via MAVLink - Dev documentation - ArduPilot, accessed August 11, 2025, https://ardupilot.org/dev/docs/raspberry-pi-via-mavlink.html
47. companion/Common/Ubuntu/mavlink-router/mavlink-router.conf at master - GitHub, accessed August 11, 2025, https://github.com/ArduPilot/companion/blob/master/Common/Ubuntu/mavlink-router/mavlink-router.conf
48. bluenviron/mavp2p: flexible and efficient Mavlink router - GitHub, accessed August 11, 2025, https://github.com/bluenviron/mavp2p
49. Low latency h264 rtp video using gstreamer - Raspberry Pi Forums, accessed August 11, 2025, https://forums.raspberrypi.com/viewtopic.php?t=349669
50. [Solved] Stream H264 over RTP using nvv4l2h264enc using Gstreamer (Getting corrupted frames) - Jetson Nano - NVIDIA Developer Forums, accessed August 11, 2025, https://forums.developer.nvidia.com/t/solved-stream-h264-over-rtp-using-nvv4l2h264enc-using-gstreamer-getting-corrupted-frames/159319
51. Any tips for low latency for video streaming using v4l2h264enc? : r/gstreamer - Reddit, accessed August 11, 2025, https://www.reddit.com/r/gstreamer/comments/1geq2e1/any_tips_for_low_latency_for_video_streaming/
52. RTP-Stream receiving with Gstreamer: How to reduce High Latency? - Super User, accessed August 11, 2025, https://superuser.com/questions/1912033/rtp-stream-receiving-with-gstreamer-how-to-reduce-high-latency
53. Quality-of-Service - GStreamer, accessed August 11, 2025, https://gstreamer.freedesktop.org/documentation/additional/design/qos.html
54. Rtpjitterbuffer in gstreamer pipelines - Jetson TX2 - NVIDIA Developer Forums, accessed August 11, 2025, https://forums.developer.nvidia.com/t/rtpjitterbuffer-in-gstreamer-pipelines/161146
55. [OpenWrt Wiki] WireGuard basics, accessed August 11, 2025, https://openwrt.org/docs/guide-user/services/vpn/wireguard/basics
56. [OpenWrt Wiki] WireGuard server, accessed August 11, 2025, https://openwrt.org/docs/guide-user/services/vpn/wireguard/server
57. Failover to LTE using mwan3 [Turris wiki], accessed August 11, 2025, https://wiki.turris.cz/en/howto/multiwan
58. Configure Multiwan Failover with mwan3 - OpenWRT - YouTube, accessed August 11, 2025, https://www.youtube.com/watch?v=vHWYH_5ooEY
59. Most reliable OpenWRT powered router you've used (or are using)? - Reddit, accessed August 11, 2025, https://www.reddit.com/r/openwrt/comments/zqtotp/most_reliable_openwrt_powered_router_youve_used/
60. Are travel routers workable with Openwrt? And if so, which one would you recommend?, accessed August 11, 2025, https://www.reddit.com/r/openwrt/comments/1jq6syw/are_travel_routers_workable_with_openwrt_and_if/
61. CubePilot Herelink 2.4GHz Long Range HD Video Transmission System - V1.1 - GetFPV, accessed August 11, 2025, https://www.getfpv.com/cubepilot-herelink-2-4ghz-long-range-hd-video-transmission-system-v1-1.html
62. Herelink V1.1 Manual - World Drone Market, accessed August 11, 2025, https://www.worldronemarket.com/herelink-v1-1-manual/
63. Review: DJI O3 Air Unit and DJI Goggles 2 | How to Setup and Best Settings - Oscar Liang, accessed August 11, 2025, https://oscarliang.com/dji-o3-air-unit-fpv-goggles-2/
64. DJI O3 Air Unit - Specs, accessed August 11, 2025, https://www.dji.com/o3-air-unit/specs
65. DJI O3 Air Unit - X-flight Fpv, accessed August 11, 2025, https://xflight-fpv.com/product/dji-o3-air-unit/
66. CubePilot Herelink HD Air Unit V1.1 - Lumenier, accessed August 11, 2025, https://www.lumenier.com/products/herelink-hd-air-unit-v1-1?Title=Default+Title
67. DJI O3 - Rotorama, accessed August 11, 2025, https://www.rotorama.com/video-vysilace/dji-o3-air-unit

