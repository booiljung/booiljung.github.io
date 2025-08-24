# ROS 2 고성능 포인트 클라우드 처리 아키텍처
[Point Cloud Library, PCL](./index.md)


로봇 인식 시스템, 특히 3D 데이터를 다루는 시스템의 핵심에는 포인트 클라우드 데이터를 표현하고 처리하는 방식이 있다. ROS 2 생태계에서는 이 작업을 위해 두 가지 주요 데이터 구조가 지배적으로 사용된다: `sensor_msgs::msg::PointCloud2`와 Point Cloud Library (PCL)의 `pcl::PointCloud<PointT>`이다. 이 두 구조는 서로 다른 설계 철학을 가지고 있으며, 이들의 근본적인 차이점은 고성능 인식 파이프라인에서 발생하는 성능 문제의 근원이 된다. 이 섹션은 각 데이터 구조를 심층적으로 분석하여 그들의 설계 원칙, 장단점, 그리고 이 둘 사이의 근본적인 긴장 관계를 탐구한다.


`sensor_msgs::msg::PointCloud2`는 ROS 2에서 3D 포인트 클라우드 데이터를 교환하기 위한 표준 메시지 형식이다. 그러나 이 메시지는 계산적 의미의 포인트 클라우드라기보다는, 직렬화되고 자체 설명적인 바이너리 데이터 블롭(binary data blob)에 가깝다.1 이러한 설계는 특정 계산 효율성보다는 유연성과 상호 운용성을 최우선으로 고려한 결과다.


`PointCloud2` 메시지는 다음과 같은 필드로 구성된다:

- `header`: 타임스탬프와 좌표 프레임 ID(`frame_id`)를 포함하는 표준 ROS 메시지 헤더다.3

- `height`, `width`: 포인트 클라우드의 차원을 정의한다. 정렬되지 않은(unorganized) 포인트 클라우드의 경우 `height`는 1이며, `width`는 전체 포인트의 수가 된다.5 "정렬된(organized)" 클라우드와 "정렬되지 않은" 클라우드의 구분은 특정 효율적인 알고리즘에 매우 중요하다.7

- `fields`: `PointField` 메시지의 배열로, 데이터 블롭 내 각 포인트의 메모리 레이아웃을 설명한다. 예를 들어, `{name: 'x', offset: 0, datatype: FLOAT32, count: 1}`와 같은 형식이다.3 이 필드는 

  `PointCloud2`가 XYZ 좌표뿐만 아니라 강도(intensity), 법선(normal), 또는 사용자 정의 필드와 같은 임의의 데이터를 담을 수 있게 하는 유연성의 핵심이다.9

- `is_bigendian`, `point_step`, `row_step`: 데이터 블롭의 바이트 순서(endianness)와 메모리 레이아웃을 정의하는 메타데이터다.8

  `point_step`은 단일 포인트가 차지하는 바이트 크기이며, `row_step`은 `width * point_step`으로 계산된다.8

- `data`: 모든 포인트 데이터를 담고 있는 원시 `uint8` 배열이다. 데이터는 `fields`와 `point_step`에 정의된 레이아웃에 따라 순차적으로 저장된다 (예: x1, y1, z1, i1, x2, y2, z2, i2,...).6

- `is_dense`: 유효하지 않은(NaN/Inf) 포인트의 존재 여부를 나타내는 불리언 플래그다. `true`는 유효하지 않은 포인트가 없음을 의미한다.13


`PointCloud2` 메시지의 설계 목표는 직렬화된 데이터 형식이 다양한 센서나 사용자가 정의한 인메모리(in-memory) 구조체 형식과 동일하게 만들어, 이상적인 경우 간단한 `memcpy`만으로 (역)직렬화가 가능하도록 하는 것이었다.1 이는 비효율적인 데이터 재배열을 피하고, 메시지를 "네트워크 상에서(over-the-wire)" 매우 효율적인 전송 포맷으로 만든다.

그러나 데이터에 직접 접근하는 것은 복잡하고 오류가 발생하기 쉽다. 표준적인 C++ 접근 방식은 `sensor_msgs::PointCloud2Iterator`와 `PointCloud2Modifier`를 사용하여 포인터 연산을 추상화하고, 필드별로 데이터를 안전하게 읽고 쓰는 것이다.9


Point Cloud Library(PCL)의 핵심 데이터 구조인 `pcl::PointCloud<PointT>`는 ROS 메시지와는 대조적으로, 직접적이고 고성능의 계산을 위해 설계된 C++ 템플릿 클래스다.16


- `pcl::PointCloud<PointT>`는 근본적으로 `std::vector<PointT, Eigen::aligned_allocator<PointT>>`를 감싸는 래퍼(wrapper)다.16 여기서 

  `Eigen::aligned_allocator`의 사용은 매우 중요하다.

- `PointT`: 포인트의 타입을 지정하는 템플릿 매개변수로, 이는 `pcl::PointXYZ`, `pcl::PointXYZRGB`, `pcl::PointNormal`과 같은 구조체다. PCL은 사전에 정의된 방대한 종류의 포인트 타입을 제공하며 17, 사용자가 직접 타입을 정의하는 것도 허용한다.20

- **메모리 레이아웃과 SSE 최적화**: PCL의 기본 포인트 타입 중 다수는 16바이트 정렬을 위해 추가적인 패딩(padding) 바이트를 포함한다 (예: `PointXYZ`는 4번째 `float`을 패딩으로 사용).20 이는 PCL이 의존하는 Eigen 라이브러리의 핵심 기능인 SSE(Streaming SIMD Extensions)를 활용하여 벡터화된 고성능 수학 연산을 수행하기 위함이다.17 이 설계는 계산 속도를 위한 결정적인 선택이다.

- **직접 접근**: 데이터는 `cloud.points[i].x`와 같이 타입에 안전하고 직접적으로 접근할 수 있어, 알고리즘 개발에 매우 효율적이고 직관적이다.5


ROS/PCL 파이프라인의 핵심적인 성능 문제는 데이터 구조 설계에 있어 근본적이고 철학적인 충돌에서 비롯된다. `sensor_msgs::msg::PointCloud2`는 **최대의 상호 운용성과 전송 효율성**을 위해 설계된 반면 (일반적이고 유연하며 직렬화 가능한 블롭), `pcl::PointCloud<T>`는 **최대의 계산 효율성**을 위해 설계되었다 (타입에 안전하고 메모리가 정렬된 전문화된 C++ 객체). 이러한 분열로 인해 직접적이고 비용 없는 매핑이 불가능해지며, 이 간극을 메우는 "변환" 계층이 필요하게 된다. 그리고 이 변환 계층이 바로 성능 병목의 주된 원인이 된다.

이러한 상황이 발생하는 과정을 살펴보면, 먼저 `PointCloud2`의 설계 목표는 `memcpy`를 통한 빠른 직렬화를 가능하게 하여 이상적인 *전송* 포맷을 만드는 것이었다.1 `fields` 배열과 `data` 블롭 같은 구조는 이 메시지가 범용 컨테이너로서의 역할을 수행함을 확인시켜 준다.2 반면, `pcl::PointCloud<T>`는 Eigen/SSE 최적화를 위한 특정 메모리 정렬을 갖춘 템플릿화된 `std::vector`로, 빠른 계산을 위한 전문화된 *인메모리* 포맷이다.20

결과적으로, `PointCloud2` 메시지를 수신한 ROS 노드는 데이터가 불투명한 바이트 배열이기 때문에 PCL 기반 계산을 직접 수행할 수 없다. 데이터는 반드시 파싱되어 구조화된 `pcl::PointCloud<T>`로 재구성되어야 한다. 역으로, `pcl::PointCloud<T>` 객체는 ROS 토픽에 직접 게시될 수 없으며, `PointCloud2` 블롭 형식으로 직렬화되어야 한다. 이 필수적인 양방향 변환은 두 구조의 설계 차이(범용 블롭 대 특정 구조체, 비정렬 대 정렬)로 인해 간단하지 않으며, 역사적으로 상당한 데이터 복사를 수반했다. 이 근본적인 설계 불일치가 모든 후속 성능 최적화가 해결하고자 했던 "원죄"인 셈이다.


| 기능                | `sensor_msgs::msg::PointCloud2`           | `pcl::PointCloud<T>`                                  |
| ------------------- | ----------------------------------------- | ----------------------------------------------------- |
| **주요 목표**       | 상호 운용성 및 전송                       | 계산 및 알고리즘 개발                                 |
| **타입 안전성**     | 낮음 (`fields` 배열을 통한 런타임 검증)   | 높음 (C++ 템플릿을 통한 컴파일 타임 검증)             |
| **유연성**          | 높음 (임의의 N-D 포인트 데이터 표현 가능) | 중간 (컴파일 타임에 특정 `PointT` 구조체로 고정)      |
| **메모리 레이아웃** | 연속적인 바이너리 블롭 (`uint8`)          | 구조체의 `std::vector`, 종종 SSE를 위해 16바이트 정렬 |
| **직렬화**          | 네이티브 포맷 (별도 작업 불필요)          | 블롭으로의 명시적 직렬화 필요                         |
| **직접 접근**       | 간접적 (이터레이터 또는 수동 포인터 연산) | 직접적이고 효율적 (예: `cloud.points[i].x`)           |
| **주요 라이브러리** | `sensor_msgs`, `rclcpp`                   | PCL, Eigen, Boost, FLANN, VTK 16                      |

------


이 섹션은 `pcl_conversions` 라이브러리를 심층적으로 분석하여, 높은 처리량의 인식 시스템을 괴롭혀 온 역사적인 성능 병목 현상을 드러낸다.


이 패키지는 ROS 메시지 타입과 PCL 데이터 타입 간의 필수적인 변환 함수를 제공한다.21 이는 `perception_pcl` 스택의 초석이다.22 주요 관심 함수는 `pcl_conversions::toPCL`(과거 `pcl::toROSMsg`)과 `pcl::fromROSMsg`이다.9 이 패키지는 `rclcpp`, `sensor_msgs`, `pcl_msgs` 및 PCL 라이브러리 자체(`libpcl-all-dev`)에 의존한다.21


`fromROSMsg`의 표준 구현은 역사적으로 `sensor_msgs::msg::PointCloud2`에서 `pcl::PointCloud<T>`로 직접 변환하지 않았다. 대신, 다음과 같은 비효율적인 2단계 프로세스를 수행했다 28:

1. **`sensor_msgs::msg::PointCloud2` -> `pcl::PCLPointCloud2`**: `pcl_conversions::toPCL`에 의해 처리되는 이 첫 번째 변환은 모든 메타데이터를 복사하고, 결정적으로 수 메가바이트에 달하는 전체 `data` 블롭의 깊은 복사(deep copy)를 수행한다.28
2. **`pcl::PCLPointCloud2` -> `pcl::PointCloud<T>`**: `pcl::fromPCLPointCloud2`에 의해 처리되는 이 두 번째 변환은 *복사된* 데이터 블롭을 순회하며 포인트를 하나씩 최종적인 구조화된 `pcl::PointCloud<T>` 객체로 역직렬화한다.

이러한 "이중 복사"는 상당한 CPU 오버헤드와 메모리 압박을 초래하며, 이는 개발자들 사이에서 빈번한 불만이었다.28 고주파, 고밀도 포인트 클라우드의 경우, 이 변환 단계만으로도 유용한 작업을 시작하기도 전에 인식 파이프라인의 주요 병목이 될 수 있다.


앞서 언급했듯이, PCL 포인트 타입(`PointT`)은 SSE 최적화를 위해 종종 16바이트 메모리 정렬을 위한 추가 바이트로 패딩된다.20

`pcl::PointCloud<T>`를 `sensor_msgs::msg::PointCloud2`로 변환할 때, `toROSMsg`(및 `toPCLPointCloud2`) 함수는 전통적으로 패딩 바이트를 포함한 전체 인메모리 데이터 구조의 `memcpy`를 수행했다.30

이러한 동작은 변환 자체에는 계산적으로 저렴하지만, 네트워크를 통해 전송되는 ROS 메시지의 크기를 팽창시키는 심각한 부작용을 낳는다. 실제 사례로, 한 GitHub 이슈에서는 29바이트의 유용한 데이터를 가진 Ouster 라이다 포인트 구조체가 정렬 패딩으로 인해 최종 ROS 메시지에서 `point_step`이 48바이트가 되는 현상을 지적했다. 이 팽창은 네트워크 트래픽을 이론적인 300 Mbps에서 실제 500 Mbps로 증가시켜, 기가비트 링크를 포화시키고 메시지 손실을 유발할 수 있었다.30


`pcl_conversions` 내의 성능 문제는 서로 다른 시스템 리소스 최적화 간의 더 깊은, 2차적인 충돌을 드러낸다. PCL의 설계는 메모리 정렬을 통해 **CPU 바운드(CPU-bound) 계산 성능**을 우선시하는 반면, ROS의 설계는 유연하고 컴팩트한 메시지를 통해 **I/O 바운드(I/O-bound) 전송 성능**을 우선시한다. `pcl_conversions` 라이브러리는 이 충돌의 중심에 있으며, 그 역사적인 구현은 두 가지 모두에 부정적인 영향을 미치는 절충안을 만들었다.

PCL의 Eigen 및 SSE 정렬 사용은 필터링, 정합, 특징 추출과 같은 알고리즘을 더 빠르게 만들기 위한 의도적인 선택이다. 이는 CPU를 위한 최적화다.20

`toROSMsg` 함수가 패딩된 전체 구조를 `memcpy`하기로 한 선택은 변환 프로세스 자체를 위한 국지적 최적화였다. 이는 PCL 형식에서 데이터를 가장 빠르게 꺼내는 방법이다.30 그러나 이 국지적 최적화는 전역적인 비관적 결과를 낳았다. 결과적으로 팽창된 메시지는 과도한 네트워크 대역폭을 소비하여 I/O 하위 시스템에 부담을 주고 잠재적으로 패킷 손실과 같은 시스템 전체의 장애를 유발했다.30 동시에, 

`fromROSMsg` 함수의 이중 복사는 모듈성을 위한 소프트웨어 공학적 선택이었지만, CPU와 메모리에 큰 부담을 주어 PCL의 나머지 부분이 효율적으로 사용하려는 리소스를 직접적으로 해쳤다.28 따라서 이 라이브러리는 "두 세계의 최악" 시나리오에 갇혔다: ROS 메시지로의 변환은 네트워크를 해치고, ROS 메시지로부터의 변환은 CPU를 해쳤다. 이는 성능 최적화가 단일 차원의 문제가 아니라 복잡한 시스템의 여러 부분 간의 절충안 시리즈임을 보여준다.

------


이 섹션은 "업데이트"의 핵심으로, 섹션 2에서 확인된 병목 현상을 직접 해결하는 PCL 및 `pcl_conversions`의 구체적인 최신 변경 사항을 자세히 설명한다. 개발자를 위한 실행 가능한 지침을 제공한다.


PCL 버전 1.13.1은 중간 데이터 복사를 피하는 `fromPCLPointCloud2`의 중요한 오버로드를 도입했다.31 공개적으로 사용되는 `pcl_conversions::fromROSMsg`는 여전히 존재하지만, 그 내부 구현 경로는 이제 훨씬 더 효율적일 수 있다.

이 최적화의 메커니즘은 `sensor_msgs::msg::PointCloud2::data`를 임시 `pcl::PCLPointCloud2::data`로 복사하는 대신, 최종 변환 단계가 원본 ROS 메시지의 데이터 버퍼에서 직접 작동하도록 허용하는 것이다. 이는 사실상 두 번의 깊은 복사 중 하나를 제거한다. 이 변경의 중요성은 커뮤니티의 노력으로 강조된다. 한 개발자는 이 로직을 정확히 구현한 `fromROSMsg`의 대체 함수를 만들어 거의 2배의 성능 향상을 달성했으며, 이 단일 복사 제거의 실제 영향을 입증했다.28

이 최적화는 PCL 1.13.1 이상을 사용할 때 기본적으로 활성화된다. 개발자는 이 혜택을 받기 위해 자신의 환경이 충분히 최신 버전의 PCL을 사용하고 있는지 확인해야 한다. `perception_pcl` 변경 로그는 PCL 1.13.0이 관련 변경 사항을 가져왔음을 확인시켜 주며, 이 시기가 전환점이었음을 알린다.33


PCL 버전 1.14.1은 `toPCLPointCloud2`(`toROSMsg`가 호출하는 함수)에 직렬화 중 메모리 정렬 패딩을 선택적으로 제거하는 향상된 기능을 도입했다.31

이 함수의 메커니즘은 이제 선택적 불리언 매개변수인 `remove_padding`을 허용하는 것이다. 이 값이 `true`로 설정되면 변환 프로세스는 더 이상 간단한 `memcpy`를 수행하지 않는다. 대신, 각 포인트의 실제 데이터 필드만 지능적으로 복사하여 `data` 블롭에 조밀하게 채운다. 이는 더 작고 네트워크 효율적인 메시지를 생성한다. 메시지의 `point_step`은 새로운 컴팩트한 크기를 반영하여 다시 계산된다.

이러한 패딩 제거 작업은 포인트별 복사 루프가 필요하기 때문에 단일 `memcpy`보다 계산 비용이 더 많이 든다. 그러나 이 CPU 비용은 특히 제한된 링크(예: Wi-Fi)나 매우 높은 주파수의 데이터에서 네트워크 대역폭을 크게 줄일 수 있으므로 종종 가치 있는 절충안이다.30 이를 활용하려면 개발자는 PCL 1.14.1 이상을 사용하고, 이 옵션을 노출하는 `toROSMsg` 또는 `toPCLPointCloud2` 버전을 호출하도록 코드를 수정해야 할 수 있다. `ros-perception/perception_pcl` 저장소는 이러한 변경 사항을 통합했지만, 소스에서 빌드하거나 사용자 정의 환경에 있는 개발자는 기본 PCL 버전을 인지해야 한다.



`pcl_conversions`를 사용하여 `sensor_msgs::msg::PointCloud2`와 `pcl::PointCloud<T>` 간에 변환하는 최신의 표준 C++ 코드는 다음과 같다. 9

```C++
#include <rclcpp/rclcpp.hpp>
#include <sensor_msgs/msg/point_cloud2.hpp>
#include <pcl_conversions/pcl_conversions.h>
#include <pcl/point_cloud.h>
#include <pcl/point_types.h>

// ROS 메시지에서 PCL로 변환
void cloud_callback(const sensor_msgs::msg::PointCloud2::SharedPtr msg)
{
  // PCL 포인트 클라우드 객체 생성
  pcl::PointCloud<pcl::PointXYZ>::Ptr cloud(new pcl::PointCloud<pcl::PointXYZ>);

  // ROS 메시지를 PCL 포인트 클라우드로 변환
  pcl::fromROSMsg(*msg, *cloud);

  // 클라우드 처리...
}

// PCL에서 ROS 메시지로 변환
void publish_pcl_cloud(rclcpp::Publisher<sensor_msgs::msg::PointCloud2>::SharedPtr publisher)
{
  pcl::PointCloud<pcl::PointXYZ>::Ptr cloud(new pcl::PointCloud<pcl::PointXYZ>);
  // 클라우드 데이터 채우기...

  // PCL 포인트 클라우드를 ROS 메시지로 변환
  sensor_msgs::msg::PointCloud2 output_msg;
  pcl::toROSMsg(*cloud, output_msg);

  // 메시지 게시
  publisher->publish(output_msg);
}
```


이러한 새로운 최적화에 접근하기 위해서는 PCL 버전을 확인하는 것이 중요하다. 이는 이식성 있고 성능을 고려한 코드를 작성하는 데 중요한 단계다. CMake에서는 `PCL_VERSION` 변수를 사용하거나, C++ 코드 내에서는 `PCL_VERSION_PRE(major, minor, patch)` 매크로를 사용하여 확인할 수 있다.


올바른 `CMakeLists.txt` 및 `package.xml` 구성이 필수적이다. `pcl_conversions`에 대해 링크하고 PCL 의존성이 올바르게 찾아지는지 확인해야 한다.39 일반적인 오류는 종종 잘못된 빌드 구성이나 충돌하는 설치에서 비롯된다.41


| PCL 버전      | `fromROSMsg` 방식                        | `toROSMsg` 방식  | 주요 이점           | 성능 영향                                                    |
| ------------- | ---------------------------------------- | ---------------- | ------------------- | ------------------------------------------------------------ |
| **< 1.13.1**  | 이중 복사: `ROS Msg -> PCLPC2 -> PCL<T>` | 패딩된 `memcpy`  | 단순성, 코드 재사용 | **높은 CPU 사용량** (복사로 인해), **높은 네트워크 사용량** (패딩으로 인해) |
| **>= 1.13.1** | 단일 복사: `ROS Msg -> PCL<T>`           | 패딩된 `memcpy`  | `fromROSMsg` 최적화 | **감소된 CPU 사용량**, 높은 네트워크 사용량                  |
| **>= 1.14.1** | 단일 복사: `ROS Msg -> PCL<T>`           | 선택적 패딩 제거 | `toROSMsg` 최적화   | 감소된 CPU 사용량, **감소된 네트워크 사용량** (일부 CPU 비용 발생) |

------


이 섹션은 고급 ROS 2 전송 기능을 활용하여 복사를 완전히 제거하는 미래 지향적인 접근 방식을 탐구한다. 이는 복사를 최적화하는 것에서 벗어나 복사 자체를 피하는 것으로 초점을 전환한다.


제로 카피(Zero-Copy) 데이터 전송은 동일한 머신 내의 퍼블리셔와 서브스크라이버가 통신 스택을 통한 직렬화 및 데이터 복사 없이 정확히 동일한 메모리 버퍼에 접근할 수 있도록 하는 원리다.43 이는 일반적으로 공유 메모리(shared memory)를 통해 달성된다.

ROS 2는 `rclcpp::Publisher::borrow_loaned_message()` API를 통해 이를 지원한다. 이 API를 사용하면 노드가 미들웨어의 사전 할당된 메모리 풀에서 직접 메시지 인스턴스를 "대여(loan)"할 수 있다. 노드는 이 대여한 메시지에 데이터를 채운 후 `publish()`를 호출하여 메모리의 소유권을 미들웨어에 반환한다.43 이 기능은 CycloneDDS나 FastDDS와 같은 기본 RMW(ROS Middleware) 구현이 이를 지원하는지에 따라 달라진다.44


대여 메시지를 통한 제로 카피 메커니즘은 컴파일 타임에 크기가 고정되어 알려진 POD(Plain Old Data) 메시지에서 가장 효과적으로 작동한다. 이를 통해 미들웨어는 메모리 풀을 효율적으로 사전 할당할 수 있다.47

그러나 `sensor_msgs::msg::Image` 및 `sensor_msgs::msg::PointCloud2`와 같은 표준 메시지는 동적 크기의 벡터(`uint8 data`)를 포함하기 때문에 POD가 아니다. 이들의 크기는 런타임에 결정되므로 46, 간단한 사전 할당 모델을 깨뜨리고 이러한 중요한 인식 메시지 유형에 대해 진정한 제로 카피 전송을 적용하기 어렵게 만든다. 이는 제로 카피 이상과 실제 적용 사이의 주요 격차다.46


이 문제를 해결하기 위해 `ros2_shm_msgs`와 같은 커뮤니티 패키지가 등장했다.46 이 접근 방식은 POD이며 따라서 제로 카피를 활용할 수 있는 새로운 고정 크기 메시지 유형(예: `Image1m`, `Image4m`)을 정의한다. 개발자는 자신의 센서에 충분히 큰 메시지를 선택해야 한다. 이는 실용적이지만 표준 메시지의 일반성을 희생하는 유연하지 못한 해결책이다.47

이는 개발자에게 마찰을 일으킨다. 한 사용자는 카메라 드라이버로 제로 카피를 구현하려 할 때, 대여한 버퍼에 대한 포인터를 얻어 V4L2 드라이버에 직접 전달할 수 없었고, 여전히 중간 복사가 필요하다는 것을 발견했다.48 이는 제로 카피를 디바이스 드라이버에 진정으로 원활하게 만드는 데 남아있는 과제를 보여준다.

장기적으로는 타입 적응(type adaptation) 및 협상(negotiation)과 같은 향후 ROS 2 기능이 이 문제에 대한 잠재적인 해결책이 될 수 있으며, 전송 계층에서 다양한 메모리 표현을 보다 유연하게 처리할 수 있게 해줄 것이다.46


ROS 2에서 인식 데이터에 대한 진정한 제로 카피 성능을 달성하는 것은 단순히 새로운 API 호출을 사용하는 문제가 아니다. 이는 근본적인 아키텍처 전환을 요구한다. 개발자는 "메시지 전달"에서 "공유 메모리 관리"로 사고방식을 전환해야 한다. 여기에는 고정 크기 메모리 풀 처리, 잠재적으로 사용자 정의 메시지 유형 사용, 드라이버에서 소비자까지의 데이터 흐름 재고가 포함된다.

표준 `pcl_conversions` 워크플로우는 `sensor_msgs::msg::PointCloud2` 객체를 값(또는 공유 포인터)으로 전달하고 명시적인 복사를 수행하는 메시지 전달 패러다임에 기반한다. 대여 메시지 API는 미들웨어에서 메모리를 빌리는 개념을 도입하여 공유 메모리 패러다임으로의 첫걸음을 내딛는다.43 "비정형 메시지 문제"는 표준 메시지가 이 새로운 패러다임에 부적합함을 보여준다.46 커뮤니티가 고정 크기 메시지(

`ros2_shm_msgs`)를 만든 것은 아키텍처 자체가 변경되어야 한다는 분명한 신호다.46 이제 노드들은 이러한 새로운 비표준 메시지 유형에 동의해야 한다. 하드웨어 드라이버를 대여 메시지 API와 직접 통합하려는 사용자의 어려움은 데이터의 가장 원천부터 전체 파이프라인이 공유 메모리를 염두에 두고 설계되어야 함을 보여준다.48 따라서 제로 카피로의 전환은 단순한 최적화가 아니라 다른 프로그래밍 모델로의 이동이다. 성능 향상은 상당하지만, 시스템 설계자에게 더 정교하고 메모리를 인지하는 접근 방식을 요구한다.

------


이 마지막 섹션은 보고서의 결과를 종합하여 완전한 로봇 시스템의 광범위한 맥락에 배치하고, 생태계의 현재 상태와 미래 궤도에 대한 업데이트를 제공한다.


- **포인트 밀도**: 포인트 클라우드 밀도는 성능에 직접적인 영향을 미친다. 밀도가 높을수록 알고리즘 정확도는 향상될 수 있지만 49, 더 큰 메시지를 생성하여 네트워크 대역폭 30 및 CPU 변환 비용 28 문제를 모두 악화시킨다. 센서 선택 및 구성(예: 항공 라이다의 비행 속도, 스캔 속도)은 시스템의 데이터 부하를 직접 결정한다.50
- **발행 빈도**: 고주파수 센서(예: 30Hz L515 카메라 51)는 전체 파이프라인에 엄청나고 지속적인 스트레스를 가한다. 1Hz에서는 무시할 수 있는 병목 현상이 30Hz에서는 시스템을 중단시킬 수 있다.
- **데이터 품질 및 전처리**: 실제 데이터는 노이즈가 많다. VoxelGrid, StatisticalOutlierRemoval, PassThrough와 같은 전처리 필터의 필요성은 파이프라인 초기에 계산 부하를 추가한다.17 VoxelGrid 필터로 다운샘플링하면 데이터 양을 줄일 수 있지만, 필터링 자체는 CPU와 메모리를 소비한다.17




`fromROSMsg`와 같은 단일 함수를 최적화하는 것만으로는 충분하지 않다. 시스템 성능은 복잡한 노드 그래프 전반에 걸쳐 종단 간(end-to-end) 지연 시간, CPU 사용량, 메모리 사용량 및 신뢰도를 추적하여 전체적으로 측정해야 한다.54

`ros2_benchmark` 55 및 `ros2-performance` 56와 같은 오픈 소스 프레임워크를 사용하면 테스트 대상 노드를 수정하지 않고도 rosbag의 실제 데이터를 사용하여 복잡한 토폴로지를 시뮬레이션하고 부하 상태에서 현실적인 성능 평가를 할 수 있다.55 이러한 도구는 직관적이지 않은 병목 현상을 드러낼 수 있다. 예를 들어, 고주파수 발행은 사용자 코드가 아닌 ROS 2 실행기나 미들웨어 계층 깊은 곳에서 메모리 압박과 지터(jitter)를 유발할 수 있다.57 벤치마크는 속도 저하가 포인트 클라우드 변환 자체가 아니라 사용 가능한 모든 CPU를 소비하는 압축 노드렛 때문임을 보여줄 수 있다.51


`ros-perception/perception_pcl` 저장소는 최신 LTS인 Jazzy Jalisco를 포함하여 최신 ROS 2 배포판에 대해 활발하게 유지 관리되고 있다.23 Jazzy, Humble, Rolling을 위한 최신 릴리스가 있다.27 커밋 기록을 보면 2023년, 2024년, 2025년에 걸쳐 섹션 3에서 논의된 중요한 성능 최적화를 포함하여 개선, 버그 수정 및 ROS 1 기능의 ROS 2로의 포팅이 꾸준히 이루어지고 있음을 알 수 있다.31

그러나 활발한 개발에도 불구하고 사용자는 복잡한 의존성 체인(ROS 배포판, PCL 버전, Eigen, Boost)과 환경 특성(예: ARM 아키텍처, Docker)으로 인해 빌드 및 런타임 문제를 자주 보고한다.41 이러한 문제를 해결하는 것은 생태계에 진입하는 개발자에게 일반적인 장애물이다.


본 보고서는 ROS와 PCL 데이터 구조 간의 근본적인 충돌, `pcl_conversions`의 역사적인 성능 병목 현상, 이러한 문제를 완화하는 중요한 최신 최적화, 그리고 진정한 제로 카피 전송을 수용하기 위해 필요한 아키텍처 전환에 대해 논의했다.

**시스템 설계자를 위한 권장 사항:**

1. **최신 버전 우선 사용**: 모든 변환 최적화에 접근하기 위해 PCL >= 1.14.1을 적극적으로 목표로 설정할 것.
2. **전체 파이프라인 프로파일링**: 개별 구성 요소를 최적화하기 전에 `ros2_benchmark`와 같은 도구를 사용하여 실제 시스템 병목 현상을 식별할 것.
3. **제로 카피 전환 수용**: 새로운 고성능 시스템의 경우, 처음부터 공유 메모리를 염두에 두고 설계할 것. 사용자 정의 고정 크기 메시지를 조사하거나 차세대 타입 적응 솔루션에 기여할 것.
4. **의존성 격리**: 복잡한 PCL 의존성 체인을 관리하고 재현 가능한 빌드를 보장하기 위해 Docker와 같은 컨테이너화를 활용할 것.41

ROS 2에서 포인트 클라우드 처리의 진화는 단순하고 복사가 많은 메시지 전달 모델에서 정교하고 메모리를 인지하는 아키텍처로의 명확한 궤적을 보여준다. 고성능 로보틱스의 미래는 데이터 이동을 최소화하고 알고리즘이 하드웨어에 최대한 가깝게 작동하도록 하는 데 있다.


1. PCL Tutorial: - The Point Cloud Library By Example - Jeff Delmerico, accessed July 24, 2025, http://www.jeffdelmerico.com/wp-content/uploads/2014/03/pcl_tutorial.pdf
2. pcl/Handbook - ROS Wiki, accessed July 24, 2025, http://wiki.ros.org/pcl/Handbook
3. Point Cloud Library - Wikipedia, accessed July 24, 2025, https://en.wikipedia.org/wiki/Point_Cloud_Library
4. 3D is here: Point cloud library (PCL) - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/221068443_3D_is_here_Point_cloud_library_PCL
5. Navigating Common Pitfalls in Point Cloud Analysis - Mindkosh, accessed July 24, 2025, https://mindkosh.com/blog/navigating-common-pitfalls-in-point-cloud-analysis/
6. 11x Performance Gain: Latest Connext DDS, ROS 2 Performance Benchmarks - RTI, accessed July 24, 2025, https://www.rti.com/blog/latest-connext-dds-ros-2-performance-benchmarks
7. sensor_msgs/Reviews/2010-03-01 PointCloud2_API_Review - ROS Wiki, accessed July 24, 2025, [http://wiki.ros.org/sensor_msgs/Reviews/2010-03-01%20PointCloud2_API_Review](http://wiki.ros.org/sensor_msgs/Reviews/2010-03-01 PointCloud2_API_Review)
8. Adding your own custom PointT type - Point Cloud Library 0.0 documentation, accessed July 24, 2025, https://pcl.readthedocs.io/projects/tutorials/en/latest/adding_custom_ptype.html
9. Case study: convert ROS message to PCL - CPP Optimizations diary, accessed July 24, 2025, https://cpp-optimizations.netlify.app/pcl_fromros/
10. Optimized ROS->PCL conversion - Open Robotics Discourse, accessed July 24, 2025, https://discourse.ros.org/t/optimized-ros-pcl-conversion/25833
11. pcl::PointCloud< PointT > Class Template Reference - the official ROS docs, accessed July 24, 2025, https://docs.ros.org/en/diamondback/api/pcl/html/classpcl_1_1PointCloud.html
12. pcl::PointCloud< PointT > Class Template Reference - Point Cloud Library, accessed July 24, 2025, http://pointclouds.org/documentation/classpcl_1_1_point_cloud.html
13. Module common - Point Cloud Library (PCL), accessed July 24, 2025, http://pointclouds.org/documentation/group__common.html
14. [conversions] toPCLPointCloud2 or toROSMsg
15. ros2_cookbook/rclcpp/pcl.md at main - GitHub, accessed July 24, 2025, https://github.com/mikeferguson/ros2_cookbook/blob/main/rclcpp/pcl.md
16. sensor_msgs/msg/PointCloud2 Message - ROS Documentation, accessed July 24, 2025, https://docs.ros2.org/foxy/api/sensor_msgs/msg/PointCloud2.html
17. PointCloud2 - sensor_msgs: Humble 4.9.0 documentation - the official ROS docs, accessed July 24, 2025, https://docs.ros.org/en/ros2_packages/humble/api/sensor_msgs/msg/PointCloud2.html
18. Built-in Message Type | Introduction to ROS2 and Robotics, accessed July 24, 2025, https://www.learnros2.com/ros/ros2-building-blocks/built-in-types/built-in-message-type
19. PointCloud2 and PointField - pcl - Robotics Stack Exchange, accessed July 24, 2025, https://robotics.stackexchange.com/questions/73981/pointcloud2-and-pointfield
20. Using sensor_msgs/PointCloud2 natively - Robotics Stack Exchange, accessed July 24, 2025, https://robotics.stackexchange.com/questions/70072/using-sensor-msgs-pointcloud2-natively
21. In-Place Modification of PointCloud2 message? - ROS Answers archive, accessed July 24, 2025, https://answers.ros.org/question/324666/
22. ROS2 pointcloud : r/ROS - Reddit, accessed July 24, 2025, https://www.reddit.com/r/ROS/comments/gkxyqk/ros2_pointcloud/
23. GitRepJo/pcl_example: A template ROS2 C++ node to test a PCL Pointcloud2 processing, accessed July 24, 2025, https://github.com/GitRepJo/pcl_example
24. Conversion from sensor_msgs::PointCloud2 to pcl::PointCloud
25. home/travis/build/PointCloudLibrary/pcl/common/include/pcl/ros/conversions.h Source File, accessed July 24, 2025, https://pointclouds.org/documentation/ros_2conversions_8h_source.html
26. ROS 2 Performance Benchmarking - ROS General - Open Robotics Discourse, accessed July 24, 2025, https://discourse.ros.org/t/ros-2-performance-benchmarking/44382
27. NVIDIA-ISAAC-ROS/ros2_benchmark: Benchmark the performance of your ROS 2 graphs, accessed July 24, 2025, https://github.com/NVIDIA-ISAAC-ROS/ros2_benchmark
28. ROS 2 benchmark open source release, accessed July 24, 2025, https://discourse.ros.org/t/ros-2-benchmark-open-source-release/30753
29. irobot-ros/ros2-performance: Framework to evaluate peformance of ROS 2 - GitHub, accessed July 24, 2025, https://github.com/irobot-ros/ros2-performance
30. PillarGen: Enhancing Radar Point Cloud Density and Quality via Pillar-based Point Generation Network - arXiv, accessed July 24, 2025, https://arxiv.org/html/2403.01663v2
31. How Can I Estimate the LiDAR Point Density for My Flight? - Knowledge Base - Inertial Labs, accessed July 24, 2025, https://resources.inertiallabs.com/en-us/knowledge-base/how-can-i-estimate-the-point-density-for-my-flight
32. Influence of point cloud density on the results of automated Object-Based building extraction from ALS data - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/262673446_Influence_of_point_cloud_density_on_the_results_of_automated_Object-Based_building_extraction_from_ALS_data
33. Frequency of depth and pointclouds got much slower when we subscribed compressedDepth / Issue #1672 / IntelRealSense/realsense-ros - GitHub, accessed July 24, 2025, https://github.com/IntelRealSense/realsense-ros/issues/1672

