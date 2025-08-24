[IMU](./index.md)
# Jetson Orin AGX 기반 드론의 센서 퓨전을 위한 IMU 선정 및 상태 추정 아키텍처 구축



이 보고서는 NVIDIA Jetson Orin AGX를 메인 컴퓨터로 사용하는 드론의 상태(위치, 속도, 자세, 각속도)를 정밀하게 추정하는 과제를 다룬다. 본 과제의 핵심적인 기술적 난제는 GPS 데이터의 수신 방식에 있다. GPS 수신기로부터 직접 데이터를 받는 것이 아니라, Flight Controller(FC)를 경유하여 수신하기 때문에 상당한 시간 지연(latency)이 발생하며, 이로 인해 데이터의 신뢰성에 문제가 발생한다. 이러한 지연된 데이터는 표준적인 센서 퓨전 알고리즘, 특히 칼만 필터 계열의 알고리즘에 적용될 경우 필터의 성능을 저하시키거나 심각한 경우 발산(divergence)시켜 시스템 전체를 불안정하게 만들 수 있다.

따라서 본 보고서는 이러한 제약 조건 하에서 드론의 상태를 강건하고(robust) 정확하게 추정하기 위한 포괄적인 해결책을 제시하는 것을 목표로 한다. 구체적으로, Xsens MTi 센서를 ROS2(Robot Operating System 2) 환경에서 활용하여 지연된 GPS 데이터를 효과적으로 융합하는 전체 아키텍처와 구현 방법을 상세히 기술한다. 요구사항은 명확하다: 최소 200Hz 이상의 데이터 출력률을 지원하고, 검증된 ROS2 드라이버를 제공하는 완제품 IMU를 특정하여 도입 비용과 함께 제시해야 한다.


이 문제에 대한 해결책으로, ROS 생태계에서 표준으로 자리 잡은 `robot_localization` 패키지를 활용한 '이중 확장 칼만 필터(Dual Extended Kalman Filter)' 아키텍처를 제안한다. 이 접근법은 로보틱스 분야에서 널리 사용되는 모범 사례(best practice)로, 두 종류의 데이터를 분리하여 처리하는 것이 핵심이다. 첫 번째 필터는 IMU와 같이 빠르고 연속적인 데이터를 처리하여 부드러운 단기적 움직임을 추정하고, 두 번째 필터는 첫 번째 필터의 결과와 GPS처럼 느리고 지연이 있지만 절대적인 기준을 제공하는 데이터를 융합하여 전역적인 오차를 보정한다.

가장 중요한 점은 `robot_localization` 패키지가 FC를 통해 들어오는 지연된 GPS 데이터 문제를 해결할 수 있는 내장 메커니즘을 이미 갖추고 있다는 것이다. 이 '지연된 측정치 처리(Delayed Measurement Handling)' 기능은 현재 시간보다 과거의 타임스탬프를 가진 데이터가 들어왔을 때, 필터의 상태를 해당 과거 시점으로 되돌린 후(revert), 지연된 측정치를 반영하고 다시 현재 시간까지 상태를 재계산(re-propagate)하는 매우 강력한 방식이다. 이 기능을 통해 우리는 FC로부터의 지연 문제를 정면으로 해결할 수 있다.

따라서 본 보고서는 Xsens MTi 센서를 활용한 이중 EKF 아키텍처 설계, 그리고 최종적으로 Jetson Orin AGX 기반의 ROS2 시스템에 통합하고 검증하는 전 과정을 포괄하는 엔지니어링 가이드를 제공할 것이다.



드론의 성공적인 상태 추정을 위해 IMU 센서는 다음 세 가지 핵심 요구사항을 반드시 만족해야 한다.

**200Hz 이상 데이터 출력률**: 드론은 매우 동적인 시스템으로, 급격한 기동과 빠른 자세 변화가 빈번하게 발생한다. 200Hz 이상의 높은 데이터 출력률은 이러한 빠른 움직임을 놓치지 않고 포착하여 칼만 필터의 예측(prediction) 단계에서 모델의 정확도를 유지하는 데 필수적이다. 특히 GPS 데이터 수신 간격이 길고 지연이 있는 상황에서, 두 GPS 보정 사이의 구간은 오롯이 IMU 데이터에 기반한 추측 항법(dead-reckoning)으로 채워진다. 이때 IMU 데이터의 주파수가 높을수록 더 정밀한 적분이 가능해져 오차 누적을 최소화할 수 있다.

**ROS2 호환성**: 프로젝트 개발 시간을 단축하고 안정성을 확보하기 위해서는 성숙한 ROS2 드라이버의 존재가 매우 중요하다. 이상적인 드라이버는 센서 제조사에 의해 공식적으로 지원되며, ROS2의 최신 배포판(예: Humble, Iron)과 호환되고, `colcon build` 명령어만으로 간단히 빌드되어야 한다. 또한, 센서의 다양한 파라미터를 launch 파일이나 YAML 파일을 통해 쉽게 설정할 수 있어야 한다.

**완제품(COTS) 및 PCB 불필요**: 이 요구사항은 연구 및 개발의 초점을 소프트웨어와 알고리즘에 맞추기 위함이다. 별도의 전원 회로나 인터페이스 회로를 설계하고 제작하는 과정 없이, USB나 RS-232와 같은 표준 인터페이스를 통해 Jetson Orin AGX에 바로 연결하여 사용할 수 있는 완제품 형태여야 한다.


**개요**: Movella사의 Xsens MTi 시리즈는 소형 폼팩터와 우수한 성능으로 로보틱스 분야에서 널리 사용된다. 요구사항에 부합하는 모델로는 MTi-1 (IMU), MTi-2 (VRU), MTi-3 (AHRS) 등이 있으며, 이 중 MTi-3는 내장 필터를 통해 안정적인 자세 정보를 제공하는 AHRS(Attitude and Heading Reference System)이다.

**성능**: MTi-3 AHRS는 동적 조건에서 롤/피치 정확도 1.0도, 헤딩 정확도 2.0도를 제공한다. 자이로스코프의 영점 편향 안정성은 6 °/h로 드리프트가 적다. 데이터 출력률은 IMU 데이터의 경우 최대 1kHz까지 가능하며, 진동 제거 필터와 같은 고급 신호 처리 기술이 내장되어 있어 드론과 같은 진동 환경에서 강점을 보인다.

**ROS2 지원**: Xsens는 공식 MT Software Suite 내에 ROS 드라이버를 포함하여 배포한다. 또한, GitHub에는 커뮤니티에서 개발하고 유지보수하는 여러 버전의 ROS2 드라이버가 존재한다. 공식 드라이버와 3rd-party 드라이버 중 프로젝트의 ROS 배포판 및 요구사항에 더 적합한 것을 선택해야 하며, 호환성 확인이 필요하다.


센서를 선택할 때, 단순히 개별 사양만 보는 것이 아니라 당면한 문제, 즉 '지연된 GPS'와의 상호작용을 고려하는 것이 매우 중요하다. 두 GPS 보정 신호가 들어오는 사이의 긴 시간 동안, 드론의 상태 추정은 전적으로 IMU 데이터에 기반한 추측 항법(dead-reckoning)에 의존하게 된다. 이 구간에서 IMU의 성능, 특히 자이로스코프의 영점 편향 안정성(Gyro Bias Instability)은 자세 오차가 누적되는 속도를 결정하는 핵심 요소다.

Xsens MTi와 같이 드리프트가 적은 고품질 IMU를 사용하면 추측 항법 구간 동안 오차 누적이 훨씬 적다. 따라서 지연된 GPS 보정치가 들어왔을 때 수정해야 할 오차의 양 자체가 작아지므로, 필터의 상태가 급격한 점프 없이 부드럽고 안정적으로 교정될 수 있다. 결론적으로, '지연된 GPS'라는 문제는 역설적으로 시스템의 안정성과 강건함(robustness)을 확보하기 위해 Xsens MTi와 같은 고품질 IMU에 투자해야 할 필요성을 더욱 부각시킨다.



본 시스템은 `robot_localization` 패키지를 이용한 '이중 EKF' 구조를 채택한다.

- **ekf_local (Odometry EKF)**: IMU와 같은 고주파 센서를 융합하여 연속적이고 부드러운 `odom` -> `base_link` 변환을 생성한다. 이 출력은 드리프트가 누적되지만 단기적으로 안정적인 로컬 오도메트리를 제공한다.
- **ekf_global (Map EKF)**: `ekf_local`의 출력, IMU, 그리고 지연된 GPS 데이터를 모두 융합한다. GPS의 절대 위치 정보를 사용하여 `ekf_local`의 누적된 드리프트를 보정하고, 전역적으로 정확한 `map` -> `odom` 변환을 생성한다.

이 구조는 ROS 표준 좌표계(REP-105)를 따르며, 내부적으로 15차원 상태 벡터 `x`를 추정한다.
$$
x =^T
$$


FC를 경유하여 발생하는 GPS 지연 문제는 `robot_localization` 패키지의 내장 기능으로 해결한다.

1. **정확한 타임스탬프**: FC로부터 GPS 데이터를 수신하여 ROS 토픽으로 발행할 때, 메시지의 `header.stamp` 필드에 반드시 GPS 데이터가 측정된 실제 시각을 기입해야 한다.
2. **지연 보상 활성화**: `robot_localization`은 과거의 타임스탬프를 가진 데이터가 들어오면, 내부적으로 상태를 과거로 되돌린 후 해당 데이터를 반영하고 다시 현재까지 재계산한다.
3. **계산 부하 고려**: 이 기능은 CPU 자원을 소모하므로, `ekf_global`의 `history_length` 파라미터를 시스템에서 발생하는 최대 지연 시간보다 약간 길게 설정해야 한다. 너무 길면 실시간성을 해치고, 너무 짧으면 지연된 데이터가 무시될 수 있다.
4. **`navsat_transform_node` 활용**: 이 노드는 `sensor_msgs/NavSatFix` (위도/경도) 메시지를 `nav_msgs/Odometry` (map 좌표계) 메시지로 변환하는 역할을 한다.


**Xsens MTi 연결**: Xsens MTi 센서를 Jetson Orin AGX의 USB 포트에 연결한다.

**udev 규칙 설정**: USB 장치 파일명이 바뀌는 것을 방지하기 위해 고정된 심볼릭 링크를 생성한다.

1. `lsusb` 명령어로 센서의 `idVendor`와 `idProduct`를 확인한다. Xsens 제품의 `idVendor`는 `2639`이다.

2. `/etc/udev/rules.d/99-xsens.rules` 파일을 생성하고 아래 내용을 작성한다.

   ```Bash
   SUBSYSTEM=="tty", ATTRS{idVendor}=="2639", MODE="0666", GROUP="dialout", SYMLINK+="xsens_imu"
   ```

3. `sudo udevadm control --reload-rules && sudo udevadm trigger` 명령어로 규칙을 적용한다. 이제 장치는 항상 `/dev/xsens_imu`로 접근할 수 있다.

**Xsens 드라이버 설치**:

```Bash
# 워크스페이스 src 폴더로 이동
cd ~/ros2_ws/src
# 소스 코드 클론
git clone https://github.com/xsens/xsens_mti_ros_node.git
# 워크스페이스 루트로 이동하여 빌드
cd ~/ros2_ws
colcon build --symlink-install
```

**PX4-ROS 2 브릿지 설정**: PX4와 ROS2의 통신을 위해 RTPS 브릿지를 설정한다. 이는 PX4 펌웨어에서 직접 지원하며, Jetson Orin AGX에서 `micrortps_agent`를 실행할 필요가 없다. PX4의 파라미터 `UXRCE_DDS_CFG`를 Jetson Orin AGX가 연결된 시리얼 포트로 설정한다.


PX4 RTPS 브릿지가 발행하는 `px4_msgs/msg/SensorGps`를 `robot_localization`이 요구하는 `sensor_msgs/msg/NavSatFix`로 변환하는 C++ 노드를 작성한다.

**`px4_gps_bridge_node.cpp`**

```C++
#include <rclcpp/rclcpp.hpp>
#include <px4_msgs/msg/sensor_gps.hpp>
#include <sensor_msgs/msg/nav_sat_fix.hpp>

class Px4GpsBridge : public rclcpp::Node
{
public:
    explicit Px4GpsBridge() : Node("px4_gps_bridge")
    {
        // NavSatFix 메시지를 발행할 퍼블리셔 생성
        nav_sat_fix_publisher_ = this->create_publisher<sensor_msgs::msg::NavSatFix>("/gps/data", 10);

        // PX4의 GPS 토픽을 구독할 서브스크라이버 생성
        sensor_gps_subscriber_ = this->create_subscription<px4_msgs::msg::SensorGps>(
            "/fmu/out/vehicle_gps_position",
            10,
            std::bind(&Px4GpsBridge::gps_callback, this, std::placeholders::_1));
        
        RCLCPP_INFO(this->get_logger(), "PX4 GPS Bridge 노드가 시작되었습니다.");
    }

private:
    void gps_callback(const px4_msgs::msg::SensorGps::SharedPtr msg)
    {
        auto nav_sat_fix_msg = std::make_unique<sensor_msgs::msg::NavSatFix>();

        // *** 가장 중요한 부분: PX4의 타임스탬프를 그대로 사용 ***
        // 이를 통해 robot_localization이 지연 보상을 정확히 수행할 수 있다.
        nav_sat_fix_msg->header.stamp = rclcpp::Time(msg->timestamp * 1000); // PX4 timestamp는 마이크로초 단위
        nav_sat_fix_msg->header.frame_id = "gps_link"; // 또는 적절한 프레임 ID

        // GPS 상태 매핑
        // PX4의 fix_type 3 이상은 3D fix로 간주
        if (msg->fix_type >= 3) {
            nav_sat_fix_msg->status.status = sensor_msgs::msg::NavSatStatus::STATUS_FIX;
        } else {
            nav_sat_fix_msg->status.status = sensor_msgs::msg::NavSatStatus::STATUS_NO_FIX;
        }
        nav_sat_fix_msg->status.service = sensor_msgs::msg::NavSatStatus::SERVICE_GPS;

        // 위도, 경도, 고도 데이터 변환 및 복사
        // PX4는 1e7을 곱한 정수형, mm 단위로 제공하므로 변환 필요
        nav_sat_fix_msg->latitude = msg->lat / 1e7;
        nav_sat_fix_msg->longitude = msg->lon / 1e7;
        nav_sat_fix_msg->altitude = msg->alt / 1e3;

        // 위치 공분산은 현재 사용하지 않으므로 0으로 설정
        nav_sat_fix_msg->position_covariance.fill(0.0);
        nav_sat_fix_msg->position_covariance_type = sensor_msgs::msg::NavSatFix::COVARIANCE_TYPE_UNKNOWN;

        nav_sat_fix_publisher_->publish(std::move(nav_sat_fix_msg));
    }

    rclcpp::Publisher<sensor_msgs::msg::NavSatFix>::SharedPtr nav_sat_fix_publisher_;
    rclcpp::Subscription<px4_msgs::msg::SensorGps>::SharedPtr sensor_gps_subscriber_;
};

int main(int argc, char * argv)
{
    rclcpp::init(argc, argv);
    rclcpp::spin(std::make_shared<Px4GpsBridge>());
    rclcpp::shutdown();
    return 0;
}
```

빌드 설정 (CMakeLists.txt 및 package.xml)

이 노드를 빌드하기 위해 CMakeLists.txt와 package.xml에 다음 내용을 추가해야 한다.

`package.xml`:

```XML
<depend>rclcpp</depend>
<depend>sensor_msgs</depend>
<depend>px4_msgs</depend>
```

`CMakeLists.txt`:

```CMake
add_executable(px4_gps_bridge_node src/px4_gps_bridge_node.cpp)
ament_target_dependencies(px4_gps_bridge_node rclcpp sensor_msgs px4_msgs)

install(TARGETS
  px4_gps_bridge_node
  DESTINATION lib/${PROJECT_NAME}
)
```


이중 EKF를 위해 `ekf_local.yaml`, `ekf_global.yaml`과 GPS 변환을 위한 `navsat.yaml`을 작성한다.


```YAML
# ekf_local.yaml
ekf_local_node:
  ros__parameters:
    frequency: 50.0
    two_d_mode: false
    publish_tf: true
    
    map_frame: map
    odom_frame: odom
    base_link_frame: base_link
    world_frame: odom

    imu0: /imu/data
    imu0_config:  # AX, AY, AZ
    imu0_differential: false
    imu0_relative: false
    
    initial_estimate_covariance: [1.0e-3, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
                                  0, 1.0e-3, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
                                  0, 0, 1.0e-3, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
                                  0, 0, 0, 1.0e-3, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
                                  0, 0, 0, 0, 1.0e-3, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
                                  0, 0, 0, 0, 0, 1.0e-3, 0, 0, 0, 0, 0, 0, 0, 0, 0,
                                  0, 0, 0, 0, 0, 0, 1.0e-3, 0, 0, 0, 0, 0, 0, 0, 0,
                                  0, 0, 0, 0, 0, 0, 0, 1.0e-3, 0, 0, 0, 0, 0, 0, 0,
                                  0, 0, 0, 0, 0, 0, 0, 0, 1.0e-3, 0, 0, 0, 0, 0, 0,
                                  0, 0, 0, 0, 0, 0, 0, 0, 0, 1.0e-3, 0, 0, 0, 0, 0,
                                  0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1.0e-3, 0, 0, 0, 0,
                                  0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1.0e-3, 0, 0, 0,
                                  0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1.0e-3, 0, 0,
                                  0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1.0e-3, 0,
                                  0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1.0e-3]
    
    process_noise_covariance: [0.05, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
                               0, 0.05, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
                               0, 0, 0.06, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
                               0, 0, 0, 0.03, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
                               0, 0, 0, 0, 0.03, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
                               0, 0, 0, 0, 0, 0.06, 0, 0, 0, 0, 0, 0, 0, 0, 0,
                               0, 0, 0, 0, 0, 0, 0.025, 0, 0, 0, 0, 0, 0, 0, 0,
                               0, 0, 0, 0, 0, 0, 0, 0.025, 0, 0, 0, 0, 0, 0, 0,
                               0, 0, 0, 0, 0, 0, 0, 0, 0.04, 0, 0, 0, 0, 0, 0,
                               0, 0, 0, 0, 0, 0, 0, 0, 0, 0.01, 0, 0, 0, 0, 0,
                               0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0.01, 0, 0, 0, 0,
                               0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0.02, 0, 0, 0,
                               0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0.01, 0, 0,
                               0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0.01, 0,
                               0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0.015]
```


```YAML
# ekf_global.yaml
ekf_global_node:
  ros__parameters:
    frequency: 30.0
    two_d_mode: false
    publish_tf: true
    
    map_frame: map
    odom_frame: odom
    base_link_frame: base_link
    world_frame: map
    
    history_length: 1.0 

    odom0: /odometry/filtered/local
    odom0_config:
       # AX, AY, AZ
    odom0_differential: false
    odom0_relative: false

    odom1: /odometry/gps
    odom1_config:
       # AX, AY, AZ
    odom1_differential: false
    odom1_relative: false

    imu0: /imu/data
    imu0_config:
       # AX, AY, AZ
    imu0_differential: false
    imu0_relative: false
```


```YAML
# navsat.yaml
navsat_transform_node:
  ros__parameters:
    frequency: 5.0
    magnetic_declination_radians: -0.1309 
    yaw_offset: 1.57079
    zero_altitude: false
    broadcast_utm_transform: true
    publish_filtered_gps: true
    use_odometry_yaw: false
    wait_for_datum: true
    datum: [35.0280, 126.7529, 0.0]
```


모든 노드를 하나의 launch 파일로 실행한다.

```Python
# in drone_bringup/launch/localization.launch.py
import os
from ament_index_python.packages import get_package_share_directory
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    pkg_share = get_package_share_directory('drone_bringup') # 자신의 패키지 이름으로 변경
    
    # 1. Xsens MTi 드라이버 노드
    xsens_mti_node = Node(
        package='xsens_mti_driver',
        executable='xsens_mti_node',
        name='xsens_mti_driver',
        output='screen',
        parameters=[{'port': '/dev/xsens_imu', 'baudrate': 115200}]
    )

    # 2. PX4 GPS 브릿지 노드
    px4_gps_bridge_node = Node(
        package='drone_bringup', # 브릿지 노드가 포함된 패키지 이름
        executable='px4_gps_bridge_node',
        name='px4_gps_bridge_node',
        output='screen'
    )

    # 3. Local EKF 노드
    ekf_local_node = Node(
        package='robot_localization',
        executable='ekf_node',
        name='ekf_local_node',
        output='screen',
        parameters=[os.path.join(pkg_share, 'config/ekf_local.yaml')],
        remappings=[('/odometry/filtered', '/odometry/filtered/local')]
    )

    # 4. Global EKF 노드
    ekf_global_node = Node(
        package='robot_localization',
        executable='ekf_node',
        name='ekf_global_node',
        output='screen',
        parameters=[os.path.join(pkg_share, 'config/ekf_global.yaml')],
        remappings=[('/odometry/filtered', '/odometry/filtered/global')]
    )

    # 5. Navsat Transform 노드
    navsat_transform_node = Node(
        package='robot_localization',
        executable='navsat_transform_node',
        name='navsat_transform_node',
        output='screen',
        parameters=[os.path.join(pkg_share, 'config/navsat.yaml')],
        remappings=[('/imu', '/imu/data'),
                    ('/gps/fix', '/gps/data'), # 브릿지 노드가 발행하는 토픽으로 변경
                    ('/odometry/filtered', '/odometry/filtered/global'),
                    ('/odometry/gps', '/odometry/gps')]
    )

    return LaunchDescription([
        xsens_mti_node,
        px4_gps_bridge_node,
        ekf_local_node,
        ekf_global_node,
        navsat_transform_node
    ])
```


**RViz2 시각화**:

1. Fixed Frame을 `map`으로 설정한다.
2. TF 디스플레이를 추가하여 `map` -> `odom` -> `base_link` transform 트리를 확인한다.
3. Path 디스플레이를 추가하여 `/odometry/filtered/local`과 `/odometry/filtered/global`의 궤적을 비교한다. `global` 경로가 GPS 보정 시 `local` 경로의 오차를 수정하는지 확인한다.

**커맨드라인 도구**:

- `ros2 topic hz /imu/data`: IMU 데이터 발행 주기를 확인한다.
- `ros2 topic echo /fmu/out/vehicle_gps_position`: PX4 브릿지에서 GPS 데이터가 들어오는지 확인한다.
- `ros2 topic echo /gps/data`: 작성한 변환 노드가 `NavSatFix` 메시지를 잘 발행하는지 확인하고, `header.stamp`가 올바른지 검증한다.
- `ros2 run tf2_ros tf2_echo map odom`: `map` -> `odom` 변환이 안정적으로 발행되는지 확인한다.
- `ros2 run tf2_tools view_frames`: 전체 TF 트리를 PDF 파일로 생성하여 연결 관계를 최종 검증한다.


- delayed fusion in robot_localization stack - Robotics Stack Exchange, accessed July 30, 2025, https://robotics.stackexchange.com/questions/114170/delayed-fusion-in-robot-localization-stack
- robot_localization wiki - ROS documentation, accessed July 30, 2025, http://docs.ros.org/en/melodic/api/robot_localization/html/index.html
- Smoothing Odometry using Robot Localization - Nav2 1.0.0 documentation, accessed July 30, 2025, https://docs.nav2.org/setup_guides/odom/setup_robot_localization.html
- Integrating GPS Data - robot_localization - ROS documentation, accessed July 30, 2025, http://docs.ros.org/en/melodic/api/robot_localization/html/integrating_gps.html
- DEMCON/ros2_xsens_mti_driver: ros2 driver for xsens mti devices - GitHub, accessed July 30, 2025, https://github.com/DEMCON/ros2_xsens_mti_driver
- MicroStrain 3DM-GX5-AR High Performance IMU/VRU | Parker NA, accessed July 30, 2025, https://ph.parker.com/us/en/product/microstrain-3dm-gx5-ar-high-performance-imu-vru/microstrain-3dm-gx5-ar
- Xsens Sensor Modules - Movella.com, accessed July 30, 2025, https://www.movella.com/sensor-modules
- Data sheet MTi 1-series, accessed July 30, 2025, https://www.mouser.com/catalog/specsheets/xsens_DatasheetMTi1series.pdf
- Xsens MTi-7 GNSS/INS | Movella.com, accessed July 30, 2025, https://www.movella.com/sensor-modules/xsens-mti-7-gnss-ins
- xsens_mti_driver - ROS Wiki, accessed July 30, 2025, http://wiki.ros.org/xsens_mti_driver
- Estimation for Quadrotors - arXiv, accessed July 30, 2025, https://arxiv.org/pdf/1809.00037



