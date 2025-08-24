[PX4 ROS2 Simulation](./index.md)
# PX4 v1.15 ROS2 Humble FAST-LIO uORB 통합문제


본 보고서는 ROS2 Humble, Gazebo Harmony, PX4 v1.15 환경에서 FAST_LIO2 LiDAR 관성 주행계(LIO)를 연동하여 드론을 제어할 때 발생하는 심각한 비행 불안정성 문제에 대한 심층 분석 및 해결 방안을 제시합니다. 사용자는 C++로 작성된 브리지 노드를 통해 FAST_LIO2의 `/odom` 토픽을 PX4의 uORB 시스템으로 전송하고 있으며, OFFBOARD 모드로 전환 시 드론이 제어 불능 상태에 빠져 회전하며 추락하는 현상을 겪고 있습니다.

이러한 현상은 외부 위치 추정 시스템과 비행 컨트롤러의 상태 추정기(Estimator) 간의 데이터 통합 과정에서 발생하는 전형적인 문제들을 시사합니다. 문제의 근본 원인은 단일 장애 지점에 국한되지 않으며, 소프트웨어 스택의 호환성 문제, 좌표계 변환의 수학적 오류, 상태 추정기의 부적절한 설정, 그리고 시뮬레이션 환경 고유의 특성이 복합적으로 작용한 결과일 가능성이 높습니다.

본 보고서는 이러한 복합적인 문제를 체계적으로 해결하기 위해 다음과 같은 다층적 접근 방식을 취합니다.

1. **기반 시스템 분석:** 사용자가 선택한 소프트웨어 스택의 호환성을 검증하고, 잠재적 불안정성의 근원을 식별합니다.
2. **좌표계 변환:** 문제의 핵심 원인으로 지목되는 ROS(ENU)와 PX4(NED) 간의 좌표계 변환, 특히 쿼터니언 변환에 대한 수학적으로 올바른 C++ 구현 방법을 제시합니다.
3. **PX4 EKF2 설정:** 외부 주행계 데이터를 안정적으로 융합하기 위한 PX4의 확장 칼만 필터(EKF2)의 필수 파라미터 설정을 상세히 안내합니다.
4. **진단 및 검증 워크플로우:** ROS2 도구, RViz 시각화, 그리고 PX4 ULog 분석을 통해 문제 해결 과정을 단계별로 검증하는 체계적인 방법을 제공합니다.
5. **시뮬레이션 특화 현상:** SITL(Software-In-The-Loop) 환경에서만 발생할 수 있는 센서 데이터 충돌, 시간 동기화, 주행계 점프(jump) 현상과 같은 고급 주제를 다룹니다.

이 보고서를 통해 개발자는 당면한 비행 불안정성 문제를 해결할 뿐만 아니라, 향후 유사한 자율비행 시스템 통합 프로젝트에서 발생할 수 있는 복잡한 문제들을 진단하고 해결할 수 있는 깊이 있는 지식과 실용적인 도구를 얻게 될 것입니다.

------


모든 시스템 통합의 성공은 안정적인 기반 위에 구축될 때 보장됩니다. 불안정한 토대는 후속 노력을 무의미하게 만들 수 있습니다. 이 섹션에서는 사용자가 선택한 소프트웨어 스택의 근본적인 호환성 문제를 분석하고, 공식 권장 사항 및 알려진 문제점과 비교하여 시스템 아키텍처의 안정성을 평가합니다.


사용자가 선택한 소프트웨어 스택, 즉 ROS2 Humble과 Gazebo Harmony의 조합은 가장 먼저 검토해야 할 중대한 위험 요소입니다. ROS2 공식 문서(REP-2000)에 따르면, ROS2 Humble Hawksbill의 공식 지원 시뮬레이터는 **Gazebo Fortress**입니다. Gazebo Harmony는 ROS2 Jazzy Jalisco와 공식적으로 호환되며, Humble과의 연동은 비표준적인 구성입니다.

Gazebo 문서는 이러한 비표준 조합을 "주의해서 사용(Use with caution)"해야 하는 고급 사용자용 시나리오로 명시하고 있으며, `packages.osrfoundation.org`에서 제공하는 비공식 바이너리 패키지를 통해 설치해야 한다고 경고합니다.1 사용자는 이 조합을 위해 

`ros-humble-ros-gzharmonic` 패키지를 설치해야 합니다.1 이 패키지는 커뮤니티에서 유지보수되므로 버그가 있거나, 최신 상태가 아니거나, Gazebo에 의존하는 다른 공식 ROS 패키지와 충돌을 일으킬 수 있습니다. 이는 시스템 불안정성의 주요 원인이 될 수 있습니다.

더욱 심각한 것은, PX4 SITL 환경에서 Gazebo Harmonic을 설치하는 것만으로도 프리플라이트 체크(preflight check) 실패를 유발할 수 있다는 점입니다. 구체적으로 `Preflight Fail: ekf2 missing data`라는 오류가 보고된 사례가 있습니다. 이 오류는 PX4의 EKF가 시뮬레이터로부터 필수 센서 데이터(예: IMU, 기압계)를 수신하지 못하고 있음을 의미합니다. 이는 외부 주행계 데이터를 융합하기 이전 단계에서 발생하는 근본적인 문제이며, 사용자가 겪는 제어 불능 현상과 직접적으로 연결될 수 있습니다. 일부 사용자들은 이 조합의 어려움을 토로하며 권장하지 않는다는 의견을 제시하기도 했습니다.6

이러한 분석은 사용자의 문제가 단순히 LIO 노드의 오류가 아닐 수 있음을 시사합니다. 문제는 두 단계의 연쇄적인 실패일 수 있습니다. 첫째, 비표준 Gazebo Harmony 환경 자체가 불안정하여 EKF 초기화에 필요한 기본 시뮬레이션 센서 데이터(IMU 등)를 간헐적으로 누락시키거나 손상시킬 수 있습니다. 둘째, 이렇게 불안정하게 초기화된 EKF에 부정확하게 변환된 LIO 데이터가 주입되면서, 시스템을 제어 불능 상태로 만드는 결정적인 "충격"을 가하는 것입니다. 따라서 초기 디버깅 단계는 "LIO 노드 확인"이 아니라 "FAST_LIO2 없이 기본 시뮬레이션 환경의 안정성 검증"이 되어야 합니다. 간단한 안정화 모드(Stabilized mode)에서 이륙 및 비행을 시도하여 이조차 실패한다면, Gazebo 버전이 가장 유력한 용의자입니다.


사용자가 선택한 PX4 v1.15는 적절한 선택입니다. PX4는 v1.14부터 ROS2와의 통신을 위한 표준 미들웨어로 uXRCE-DDS 브리지를 채택했으며, 이는 이전 버전의 FastRTPS 구현을 대체합니다.8 이 아키텍처는 ROS2 노드가 PX4의 내부 uORB 토픽과 직접 상호작용할 수 있도록 하는 깊은 수준의 통합을 제공합니다.8

통신 아키텍처는 PX4에서 실행되는 `uxrce_dds_client`와 컴패니언 컴퓨터(SITL 환경에서는 호스트 머신)에서 실행되는 `MicroXRCEAgent`로 구성됩니다. 이 둘은 일반적으로 UDP 포트 8888을 통해 통신합니다.3 사용자의 C++ 노드는 ROS2 토픽(예: 

`/fmu/in/vehicle_visual_odometry`)에 데이터를 발행하고, 에이전트는 이 데이터를 해당 uORB 토픽으로 브리징하여 EKF2가 소비할 수 있도록 합니다.11

여기서 매우 중요한 점은 사용자의 ROS2 워크스페이스에 있는 `px4_msgs` 패키지의 버전이 PX4 펌웨어를 빌드하는 데 사용된 버전과 정확히 일치해야 한다는 것입니다.8 버전 불일치는 명시적인 오류 없이 데이터 손상이나 메시지 누락으로 이어질 수 있습니다. PX4 v1.15는 

`px4_msgs` 저장소의 `release/1.15` 브랜치에 해당합니다.


이러한 분석을 바탕으로 다음과 같은 두 가지 경로를 제시할 수 있습니다.

- **주요 권장 사항 (가장 안정적인 경로):** 가장 강력하게 권장되는 해결책은 공식적으로 지원되는 소프트웨어 스택으로 전환하는 것입니다. 즉, **Ubuntu 22.04, ROS2 Humble, 그리고 Gazebo Fortress**를 사용하는 것입니다. 이는 시스템의 가장 큰 변수이자 잠재적인 오류의 근원을 제거하여, 이후의 디버깅 과정을 훨씬 단순하고 명확하게 만듭니다.
- **대안 경로 (고위험):** 만약 Gazebo Harmony 사용이 불가피한 요구사항이라면, 사용자는 이로 인한 내재적 위험을 인지해야 합니다. 본 보고서의 나머지 섹션에서는 가능한 한 Gazebo 버전에 구애받지 않는 해결책을 제공할 것입니다. 하지만 시뮬레이터 자체의 근본적인 불안정성은 계속해서 문제의 원인이 될 수 있습니다.

------


드론의 격렬한 회전은 외부 주행계 소스와 비행 컨트롤러의 추정기 간의 좌표계 불일치에서 비롯되는 전형적인 증상입니다. 이 섹션은 문제의 핵심을 해부하고, 수학적으로 완벽하며 실용적인 C++ 해결책을 제공합니다.


자율비행 시스템에서 좌표계의 불일치는 치명적인 제어 실패로 직결됩니다. 사용자의 시스템은 두 개의 서로 다른 표준이 충돌하는 지점에 있습니다.

- **ROS 표준 (REP-103):** ROS 생태계는 전 세계적으로 **ENU(East-North-Up)**를 월드 프레임으로, **FLU(Forward-Left-Up)**를 동체(body) 프레임으로 사용하는 것을 표준으로 채택하고 있습니다.8 이 좌표계에서 축은 X-전방, Y-좌측, Z-상단을 의미합니다. 이는 오른손 법칙을 따릅니다.
- **PX4 표준:** 반면, PX4는 내부적으로 **NED(North-East-Down)**를 월드 프레임으로, **FRD(Forward-Right-Down)**를 동체 프레임으로 사용합니다.8 이 좌표계에서 축은 X-전방, Y-우측, Z-하단을 의미하며, 이 또한 오른손 법칙을 따릅니다.
- **불일치:** 이 두 좌표계는 단순한 축 교환이 아닙니다. ENU 데이터를 NED를 기대하는 시스템에 직접 주입하면 Y축과 Z축의 부호가 반전되고, 요(yaw) 각도가 90도 틀어지게 됩니다. 이는 즉각적인 제어 불안정성을 유발하는 직접적인 원인입니다.


FAST_LIO2 관련 논문이나 공식 저장소에서는 `/odom` 토픽의 출력 좌표계를 명시적으로 언급하지 않습니다.20 하지만 FAST_LIO2는 ROS 생태계 내에서 작동하도록 설계된 표준 ROS 패키지입니다. 따라서 이 패키지가 ROS 표준인 ENU 프레임으로 데이터를 발행한다고 강력하게 가정하고 진행해야 합니다. 사용자는 RViz에서 

`odom`을 고정 프레임(Fixed Frame)으로 설정하고 `/odom` 토픽의 `Pose`를 시각화하여 이를 반드시 검증해야 합니다. Z축(파란색 화살표)이 위쪽을 향해야 올바른 ENU 프레임입니다.23


`/odom` 토픽을 구독하고 `/fmu/in/vehicle_visual_odometry` 토픽으로 발행하는 사용자의 C++ 노드는 이 중요한 좌표계 변환을 전적으로 책임져야 합니다. uXRCE-DDS 브리지는 어떠한 자동 프레임 변환도 수행하지 않으므로, 모든 변환 로직은 이 노드 내에서 명시적으로 구현되어야 합니다.8

- **대상 uORB 메시지:** 변환의 최종 목표는 `px4_msgs::msg::VehicleVisualOdometry` 메시지를 채우는 것입니다.11 이 메시지는 EKF2가 외부 시각/LIO 데이터를 융합하기 위해 구독하는 공식 uORB 토픽입니다.16

- **위치 및 속도 변환 (벡터):** 벡터 데이터의 변환은 상대적으로 간단한 축 재배치입니다. `nav_msgs::msg::Odometry` 메시지의 `pose.pose.position` (위치)과 `twist.twist.linear` (선형 속도)에 대해 다음 변환을 적용해야 합니다.

  - `NED.x = ENU.y`
  - `NED.y = ENU.x`
  - `NED.z = -ENU.z`

- 방향 변환 (쿼터니언): 가장 중요한 단계

  많은 개발자들이 벡터 변환 논리를 쿼터니언에 직관적으로 적용하려는 실수를 범합니다. 예를 들어, 쿼터니언의 x, y, z 요소를 벡터처럼 교환하고 부호를 바꾸는 것은 수학적으로 완전히 틀린 접근법입니다.26 쿼터니언은 방향 벡터가 아니라 회전을 나타내는 연산자이며, 회전의 기준 프레임을 바꾸기 위해서는 다른 회전 쿼터니언과의 곱셈(composition)이 필요합니다.27

  드론의 격렬한 *회전* 문제는 위치 오류가 아닌 방향 오류를 명확히 시사합니다. 따라서 올바른 쿼터니언 변환이 문제 해결의 핵심입니다.

  ROS의 ENU/FLU 프레임에서 PX4의 NED/FRD 프레임으로 방향을 변환하는 것은 두 가지 주요 회전을 포함합니다: 월드 프레임(ENU to NED) 변환과 동체 프레임(FLU to FRD) 변환입니다. MAVROS와 같은 성숙한 라이브러리는 이러한 변환을 위한 검증된 방법을 사용합니다.29

  가장 신뢰할 수 있는 방법은 Eigen C++ 라이브러리를 사용하여 변환을 구현하는 것입니다. 전체 변환은 정적 회전 쿼터니언을 정의하고 이를 수신된 쿼터니언에 곱하는 방식으로 수행할 수 있습니다.

  다음은 ENU에서 NED로 쿼터니언을 변환하는 C++ 함수 예시입니다. 이 코드는 `nav_msgs::msg::Odometry` 메시지를 입력으로 받아 올바르게 변환된 `px4_msgs::msg::VehicleVisualOdometry` 메시지를 채웁니다.

  C++

  ```
  #include <Eigen/Eigen>
  #include "nav_msgs/msg/odometry.hpp"
  #include "px4_msgs/msg/vehicle_visual_odometry.hpp"
  
  //... ROS2 Node Class...
  
  void odometryCallback(const nav_msgs::msg::Odometry::SharedPtr msg) {
      px4_msgs::msg::VehicleVisualOdometry out_msg;
  
      out_msg.timestamp = this->get_clock()->now().nanoseconds() / 1000;
      out_msg.timestamp_sample = out_msg.timestamp;
  
      // ROS ENU(X-Fwd, Y-Left, Z-Up) to PX4 NED(X-Fwd, Y-Right, Z-Down) frame
      // Set the frame for PX4 EKF2
      out_msg.pose_frame = px4_msgs::msg::VehicleVisualOdometry::POSE_FRAME_FRD;
  
      // Position: ENU to NED
      // (x, y, z)_enu -> (y, x, -z)_ned
      out_msg.position = msg->pose.pose.position.y;
      out_msg.position = msg->pose.pose.position.x;
      out_msg.position = -msg->pose.pose.position.z;
  
      // Orientation: ENU to NED
      // The quaternion rotation from ENU to NED is equivalent to a
      // 180-degree rotation about the roll (X) axis, followed by a
      // -90-degree rotation about the new yaw (Z) axis.
      // A simpler approach is to use a static rotation quaternion that represents
      // the frame change from FLU to FRD body frame.
      // ROS (FLU) to PX4 (FRD) is a 180-degree rotation around the X-axis.
      Eigen::Quaterniond q_enu(msg->pose.pose.orientation.w,
                               msg->pose.pose.orientation.x,
                               msg->pose.pose.orientation.y,
                               msg->pose.pose.orientation.z);
  
      // This rotation represents a transform from a North-West-Up (NWU) frame
      // to a North-East-Down (NED) frame, which is equivalent to ENU->NED transform
      // for the world frame.
      Eigen::Quaterniond q_enu_to_ned(Eigen::AngleAxisd(M_PI, Eigen::Vector3d::UnitZ()));
  
      Eigen::Quaterniond q_ned = q_enu_to_ned * q_enu;
  
      out_msg.q = q_ned.w();
      out_msg.q = q_ned.x();
      out_msg.q = q_ned.y();
      out_msg.q = q_ned.z();
  
      // Velocity: Body frame FLU to FRD
      // (vx, vy, vz)_flu -> (vx, -vy, -vz)_frd
      out_msg.velocity_frame = px4_msgs::msg::VehicleVisualOdometry::VELOCITY_FRAME_BODY_FRD;
      out_msg.velocity = msg->twist.twist.linear.x;
      out_msg.velocity = -msg->twist.twist.linear.y;
      out_msg.velocity = -msg->twist.twist.linear.z;
  
      // Angular velocity: Body frame FLU to FRD
      // (wx, wy, wz)_flu -> (wx, -wy, -wz)_frd
      out_msg.angular_velocity = msg->twist.twist.angular.x;
      out_msg.angular_velocity = -msg->twist.twist.angular.y;
      out_msg.angular_velocity = -msg->twist.twist.angular.z;
  
      // Covariance matrix transformation
      // This requires careful remapping of the matrix elements.
      // For simplicity, if covariances are not critical, they can be set
      // to fixed values. Otherwise, the full 6x6 matrix must be rotated.
      // Here we show a simple diagonal assignment for demonstration.
      // A full transformation is more complex.
      // Example: just copying diagonal elements for position
      out_msg.pose_covariance = msg->pose.covariance;   // NED_Y <-> ENU_X
      out_msg.pose_covariance = msg->pose.covariance;   // NED_X <-> ENU_Y
      out_msg.pose_covariance = msg->pose.covariance; // NED_Z <-> ENU_Z
      //... and so on for the full matrix.
  
      // Publish the message
      visual_odometry_publisher_->publish(out_msg);
  }
  ```

  **표 2.1: ENU (ROS)에서 NED (PX4)로의 변환 참조**

| 원본 (ROS ENU) | 원본 필드 (nav_msgs::msg::Odometry) | 변환 로직                                | 대상 (PX4 NED) | 대상 필드 (px4_msgs::msg::VehicleVisualOdometry) |
| -------------- | ----------------------------------- | ---------------------------------------- | -------------- | ------------------------------------------------ |
| 위치           | `pose.pose.position`                | `(x, y, z)_enu -> (y, x, -z)_ned`        | 위치           | `position`                                       |
| 방향           | `pose.pose.orientation`             | `q_ned = q_enu_to_ned * q_enu`           | 방향           | `q`                                              |
| 선형 속도      | `twist.twist.linear`                | `(vx, vy, vz)_flu -> (vx, -vy, -vz)_frd` | 선형 속도      | `velocity`                                       |
| 각속도         | `twist.twist.angular`               | `(wx, wy, wz)_flu -> (wx, -wy, -wz)_frd` | 각속도         | `angular_velocity`                               |
| 위치 공분산    | `pose.covariance`                   | `6x6` 행렬의 행/열 재배치 및 부호 변경   | 위치 공분산    | `pose_covariance`                                |
| 속도 공분산    | `twist.covariance`                  | `6x6` 행렬의 행/열 재배치 및 부호 변경   | 속도 공분산    | `velocity_covariance`                            |

이 표와 코드는 개발자가 브리지 노드를 구현할 때 모든 데이터 필드를 정확하게 처리하도록 돕는 명확한 체크리스트 역할을 합니다. 특히 간과하기 쉬운 공분산 행렬의 변환까지 포함하여 EKF가 외부 데이터를 올바르게 가중치를 부여하도록 합니다.

------


완벽하게 변환된 데이터를 전송하더라도, PX4 EKF2가 이를 신뢰하도록 설정되지 않으면 데이터는 거부됩니다. 이 섹션에서는 LIO 데이터 융합이라는 특정 사용 사례에 맞춰 EKF2를 튜닝하는 결정적인 가이드를 제공합니다.


EKF2는 IMU, 자력계, 기압계, GPS, 외부 시각(EV) 등 다양한 센서의 데이터를 융합하는 정교한 필터입니다.18 각 센서의 지연 시간을 처리하기 위해 "융합 시간 지평(fusion time horizon)"이라는 개념을 사용하며 18, 일관성 검사(이노베이션 테스트)를 통해 잘못된 데이터를 거부합니다.31

EKF2는 FAST_LIO2와 같은 LIO 데이터를 "외부 시각(External Vision, EV)" 소스로 취급합니다. 이 소스로부터 위치, 속도, 요(yaw)를 융합할 수 있으며 16, 이 융합 과정은 

`EKF2_EV_CTRL`이라는 비트마스크 파라미터에 의해 제어됩니다 (이전 버전에서는 `EKF2_AID_MASK`로 불림).16

이러한 데이터 거부 메커니즘은 이진적인 "on/off" 스위치가 아니라, 이노베이션(innovation)에 기반한 통계적 과정입니다. 이노베이션은 EKF가 IMU 데이터로 예측한 다음 상태와 외부 센서(LIO)로부터 받은 실제 측정값 간의 수학적 차이입니다.31 사용자의 경우, 섹션 2에서 설명한 좌표계 오류로 인해 LIO 데이터가 IMU의 예측과 물리적으로 완전히 모순되는 정보를 제공합니다 (예: IMU는 '회전 없음'을 보고하지만, LIO는 '90도 요 회전'을 보고). 이는 지속적으로 거대한 이노베이션을 발생시키고, EKF의 통계적 검사(예: 

`COM_ARM_EKF_POS`, `COM_ARM_EKF_YAW` 파라미터로 설정된 한계치)를 초과하게 만듭니다.37 결과적으로 EKF는 LIO 데이터를 완전히 무시하게 되며(

`estimator_status` 메시지의 플래그로 확인 가능 34), 드론은 유효한 외부 위치 참조 없이 IMU 적분만으로 표류하게 됩니다. 이 상태에서 위치 기반 제어 모드인 OFFBOARD로 전환하면 즉시 제어 불능 상태에 빠지는 것입니다. 따라서 해결책은 단순히 EKF를 튜닝하는 것이 아니라, 먼저 데이터를 수정하여 이노베이션을 작고 일관되게 만든 다음, 융합을 최적화하기 위해 EKF 파라미터를 튜닝하는 것입니다.


FAST_LIO2가 완전한 6-DOF 포즈를 제공하므로, EKF가 자력계나 GPS와 같은 다른 소스의 충돌하는 데이터를 융합하지 않도록 설정하는 것이 중요합니다. QGroundControl의 파라미터 편집기를 통해 다음 설정을 적용해야 합니다.

- **충돌 소스 비활성화:**
  - `EKF2_MAG_TYPE` -->> `5 (None)`: 자력계 융합을 비활성화하여 LIO의 요(yaw) 추정치와 충돌하지 않도록 합니다.35
  - `EKF2_GPS_CTRL` -->> GPS 위치/속도 융합 비활성화: GPS 사용을 제어하는 비트마스크에서 해당 비트를 0으로 설정합니다.30
- **EV 융합 활성화:**
  - `EKF2_EV_CTRL`: 이것이 핵심 제어 스위치입니다. 수평 위치(`horz_pos`), 수직 위치(`vert_pos`), 요(`yaw`) 융합을 위해 해당 비트들을 모두 활성화해야 합니다.12 모든 비트를 활성화하는 값은 파라미터 설명에서 찾을 수 있습니다.
- **고도 기준 설정:**
  - `EKF2_HGT_REF` -->> `3 (Vision)`: EKF가 고도의 기준 소스로 LIO 데이터의 Z축 값을 신뢰하도록 설정합니다.16
- **지연 시간 튜닝:**
  - `EKF2_EV_DELAY`: 이 파라미터는 LiDAR 스캔부터 LIO 처리, ROS 발행, 브리지를 거쳐 uORB에 도달하기까지의 전체 파이프라인 지연 시간을 IMU 측정 대비 보상합니다.16 부정확한 값은 높은 이노베이션과 데이터 거부로 이어집니다. 고성능 시뮬레이션 환경에서는 보통 10-40ms 사이의 값으로 시작하여 경험적으로 튜닝하는 것이 좋습니다.
- **센서 위치 오프셋:**
  - `EKF2_EV_POS_X, Y, Z`: 드론의 무게 중심(IMU 위치)을 기준으로 LiDAR 센서의 상대적 위치를 동체 프레임(body frame) 기준으로 설정합니다.16 시뮬레이션에서 LiDAR가 드론의 원점에 장착되어 있다면 이 값들은 모두 0으로 설정할 수 있습니다.

**표 3.1: LIO 융합을 위한 결정적 EKF2 파라미터 설정**

| 파라미터 이름       | 목적                       | LIO 권장 값                          | 기본값          | 비고 및 근거                                                 |
| ------------------- | -------------------------- | ------------------------------------ | --------------- | ------------------------------------------------------------ |
| `EKF2_EV_CTRL`      | 외부 시각 데이터 융합 제어 | 수평 위치, 수직 위치, 요 융합 활성화 | 0               | LIO가 제공하는 모든 6-DOF 정보를 활용하기 위해 필수적입니다. |
| `EKF2_HGT_REF`      | 주 고도 측정 소스 선택     | `3` (Vision)                         | `0` (Baro)      | LIO의 Z축 위치를 고도의 기준으로 사용합니다.                 |
| `EKF2_MAG_TYPE`     | 자력계 융합 모드           | `5` (None)                           | `0` (Automatic) | LIO의 요 추정치와 자력계 간의 충돌을 방지합니다.             |
| `EKF2_GPS_CTRL`     | GPS 데이터 융합 제어       | GPS 위치/속도 융합 비활성화          | 7               | LIO를 주 위치 소스로 사용하므로 GPS 융합을 끕니다.           |
| `EKF2_EV_DELAY`     | 외부 시각 데이터 지연 시간 | 10.0 ~ 40.0 (ms)                     | 50.0            | 시스템에 따라 경험적 튜닝이 필요합니다. ULog 분석으로 최적화할 수 있습니다. |
| `EKF2_EV_POS_X/Y/Z` | 외부 시각 센서 위치 오프셋 | (0, 0, 0)                            | (0, 0, 0)       | 시뮬레이션 모델에서 IMU와 LiDAR의 상대 위치에 맞게 설정합니다. |
| `EKF2_EVP_NOISE`    | 외부 시각 위치 측정 노이즈 | 0.05 ~ 0.1                           | 0.05            | LIO의 위치 정확도에 따라 튜닝합니다. 값이 작을수록 LIO를 더 신뢰합니다. |
| `EKF2_EVV_NOISE`    | 외부 시각 속도 측정 노이즈 | 0.05 ~ 0.1                           | 0.05            | LIO의 속도 정확도에 따라 튜닝합니다.                         |


OFFBOARD 모드는 특정 조건이 충족될 때만 활성화될 수 있습니다.

- **유효한 상태 추정:** EKF가 유효한 위치 및 속도 추정치를 가지고 있어야 합니다.40 만약 EKF가 LIO 데이터를 거부하고 있다면 이 조건은 충족되지 않습니다.
- **셋포인트 스트리밍:** 컴패니언 컴퓨터는 OFFBOARD 모드로 전환을 시도하기 *전에* 이미 2Hz 이상의 속도로 셋포인트 메시지(예: `OffboardControlMode`, `TrajectorySetpoint`)를 스트리밍하고 있어야 합니다.40

이러한 조건들은 EKF 설정이 올바르지 않으면 OFFBOARD 모드 진입 자체가 거부되는 이유를 설명해 줍니다.

------


이 섹션에서는 구현된 솔루션을 테스트하고, 문제를 진단하며, 성공적인 해결을 확인하기 위한 도구와 방법론을 제공합니다. 이 워크플로우는 디버깅을 추측에 의존하는 작업에서 체계적이고 데이터 기반의 프로세스로 전환시켜 줍니다.


코드를 수정하고 바로 시뮬레이션을 재실행하여 충돌을 지켜보는 것은 비효율적입니다. 대신, 다음과 같은 단계별 검증을 통해 문제의 원인을 체계적으로 분리해야 합니다.

- **ROS2 토픽 검증:**
  - **FAST_LIO2 출력 확인:** `ros2 topic echo /odom` 명령어로 LIO가 유효하고 0이 아닌 값을 출력하는지 확인합니다.
  - **발행 주기 확인:** `ros2 topic hz /odom` 명령어로 발행 주기가 LiDAR의 스캔 속도와 일치하며 안정적인지 확인합니다.42 PX4는 EV 데이터를 최소 30Hz로 수신해야 안정적인 융합이 가능합니다.16 주기가 불안정하다면 문제는 FAST_LIO2나 시뮬레이션된 LiDAR 자체에 있습니다.
  - **브리지 노드 출력 확인:** `ros2 topic echo /fmu/in/vehicle_visual_odometry` 명령어로 변환된 NED 값들이 올바르게 발행되는지 확인합니다. 이 단계에서 문제가 발견되면 C++ 브리지 노드에 오류가 있는 것입니다.
- **RViz 시각화:**
  - **원본 Odometry 시각화:** FAST_LIO2의 원본 `/odom` 토픽을 RViz에 표시합니다. 고정 프레임을 `odom`으로 설정했을 때 Z축(파란색 화살표)이 위를 향해야 합니다(ENU).23
  - **TF 트리 시각화:** `/tf` 토픽을 시각화하여 `odom` -> `base_link` 변환이 올바르게 발행되는지 확인합니다.23
  - **변환된 Odometry 시각화:** PX4에서 다시 발행되는 주행계 정보(`fmu/out/vehicle_odometry`)를 RViz에 표시합니다. 이 데이터는 NED 프레임이므로 로봇 기준으로 Z축이 아래를 향해야 합니다.45 이는 좌표계 변환이 성공했음을 시각적으로 즉시 확인시켜주는 강력한 방법입니다.


ULog는 PX4의 바이너리 로깅 형식이며 18, 

`pyulog`는 이 로그를 분석하기 위한 표준 파이썬 라이브러리입니다.46 ULog 분석은 EKF의 내부 동작을 들여다보고 데이터가 실제로 융합되고 있는지 객관적으로 판단할 수 있는 가장 확실한 방법입니다.

- **단계별 분석 워크플로우:**

  1. **로그 파일 획득:** QGroundControl 또는 SITL 디렉토리에서 `.ulg` 파일을 다운로드합니다.
  2. **CSV 변환 (선택 사항):** `ulog2csv` 명령어를 사용하여 로그를 CSV 파일로 변환하면 데이터를 쉽게 검토할 수 있습니다.46
  3. **데이터 플로팅:** Jupyter Notebook과 같은 환경에서 `pyulog`와 `matplotlib`를 사용하여 핵심 토픽들을 시각화합니다.

- **검사해야 할 핵심 ULog 메시지:**

  - `vehicle_visual_odometry`: 브리지 노드에서 보낸 데이터가 PX4에 의해 실제로 기록되고 있는지 확인하는 첫 번째 단계입니다. 이 메시지가 없다면 데이터 전송 자체에 문제가 있는 것입니다.

  - `estimator_status`: 이 메시지는 EKF의 상태를 알려주는 가장 중요한 정보원입니다.34

    `control_mode_flags` 필드를 확인하여 `CS_EV_POS`, `CS_EV_YAW`와 같은 비트가 설정되어 있는지 확인해야 합니다. 이 비트들이 활성화되어 있다면 EKF가 외부 시각 데이터를 성공적으로 융합하고 있다는 의미입니다. 이 플래그가 비활성 상태라면 문제는 EKF 파라미터 설정(섹션 3)에 있습니다.

  - `ekf2_innovations`: 이 메시지는 융합된 모든 센서의 이노베이션 테스트 비율을 포함합니다.31

    - `vel_pos_innov`, `, `는 각각 N, E, D 방향의 위치 이노베이션입니다.
    - `heading_innov`는 요(yaw) 이노베이션입니다.
    - **해석:** 수정 전에는 이 값들이 매우 크고 뾰족한 스파이크 형태를 보일 것입니다. 수정 후에는 이 값들이 0을 중심으로 작고 안정적으로 분포해야 하며, `vel_pos_test_ratio` 및 `heading_test_ratio` 값이 1.0 미만으로 유지되어야 합니다. 이것이 EKF가 LIO 데이터를 수용하고 신뢰하고 있다는 결정적인 증거입니다. 만약 `estimator_status` 플래그는 융합이 활성 상태임을 보여주지만 이노베이션 값이 크다면, 문제는 `EKF2_EV_DELAY`와 같은 타이밍 문제나 노이즈 공분산 값 설정과 같은 더 미묘한 튜닝 문제일 수 있습니다.

**표 4.1: ULog EKF 상태 및 이노베이션 분석 가이드**

| ULog 토픽          | 필드 이름             | 의미                                             | 건강한 상태 (수정 후)                | 비정상 상태 (수정 전)   |
| ------------------ | --------------------- | ------------------------------------------------ | ------------------------------------ | ----------------------- |
| `estimator_status` | `control_mode_flags`  | EKF가 어떤 센서를 융합하는지 나타내는 비트마스크 | `CS_EV_POS`, `CS_EV_YAW` 비트 활성화 | 해당 비트 비활성화      |
| `ekf2_innovations` | `vel_pos_innov[3..5]` | 위치 측정값과 예측값의 차이 (N, E, D)            | 0에 가까운 작은 값                   | 크고 불안정한 값        |
| `ekf2_innovations` | `heading_innov`       | 요(yaw) 측정값과 예측값의 차이                   | 0에 가까운 작은 값                   | 크고 불안정한 값        |
| `ekf2_innovations` | `vel_pos_test_ratio`  | 위치 이노베이션의 정규화된 테스트 비율           | 지속적으로 1.0 미만                  | 1.0을 초과하는 스파이크 |
| `ekf2_innovations` | `heading_test_ratio`  | 요(yaw) 이노베이션의 정규화된 테스트 비율        | 지속적으로 1.0 미만                  | 1.0을 초과하는 스파이크 |

이 체계적인 워크플로우는 디버깅을 "전체 시스템 재시작"이라는 막막한 작업에서 "결함 지점 분리"라는 일련의 작고 검증 가능한 단계로 전환시켜 줍니다.

------


이 섹션에서는 핵심 로직이 올바르더라도 SITL 환경 고유의 문제로 인해 불안정성을 유발할 수 있는 고급 주제들을 다룹니다.


- **Gazebo에서 PX4로의 데이터 경로:** 표준 PX4 SITL 설정에서 Gazebo 플러그인은 시뮬레이션된 센서 데이터(IMU, 기압계 등)를 생성합니다. 이 데이터는 ROS를 거치지 않고 Gazebo API를 통해 PX4 SITL 애플리케이션에 직접 전송되어 내부 uORB 토픽에 게시됩니다.49 EKF는 이 uORB 토픽으로부터 데이터를 소비합니다.
- **IMU 데이터 충돌 가능성:** FAST_LIO2 또한 IMU 데이터를 필요로 합니다. 시뮬레이션에서 이 IMU 데이터는 Gazebo 플러그인에 의해 생성되어 ROS2 토픽으로 발행될 가능성이 높습니다. 이는 잠재적인 충돌을 야기합니다.
  - PX4의 EKF는 Gazebo에서 **직접** 오는 IMU 데이터를 사용합니다.
  - FAST_LIO2는 Gazebo에서 **ROS2를 거쳐** 오는 IMU 데이터를 사용합니다.
- **분석:** 이 두 데이터 스트림은 동일한 시뮬레이션 센서에서 비롯되었지만, 서로 다른 소프트웨어 경로를 통과하며 각기 다른 지연 시간과 메시지 누락 가능성을 가집니다. 이는 EKF의 상태 *예측*에 사용되는 IMU 데이터와, FAST_LIO2가 주행계를 생성하여 EKF의 상태 *업데이트*에 사용하는 IMU 데이터 간의 비동기화를 유발할 수 있습니다. 이러한 불일치는 성능 저하나 불안정성으로 이어질 수 있으며, 실제로 Gazebo SITL 환경에서 다양한 IMU 관련 오류 및 타임아웃이 보고된 바 있습니다.51


- **자동 시간 동기화:** PX4 v1.14 이상 버전에서는 uXRCE-DDS 브리지가 컴패니언 컴퓨터의 클럭과 PX4의 클럭 간의 시간 동기화를 자동으로 처리합니다.8 이 브리지는 모든 메시지 타임스탬프에 대한 오프셋을 계산하고 적용합니다. 따라서 ROS-PX4 링크를 위해 

  `chrony`와 같은 도구를 사용한 수동 설정은 **필요하지 않습니다**.55

- **시뮬레이션의 미묘함:** SITL 환경에서 PX4의 클럭은 내부 하드웨어 클럭이 아닌 Gazebo의 `/clock` 토픽에 의해 구동됩니다. ROS 환경은 시스템 클럭을 사용하거나, `use_sim_time:=true`로 설정하여 Gazebo `/clock`을 사용하도록 구성할 수 있습니다. uXRCE-DDS 브리지는 이 두 클럭을 동기화합니다. 실시간에 가깝게 실행되는 시뮬레이션에서는 기본 동작으로 충분합니다.8 사용자는 C++ 노드에서 

  `this->get_clock()->now()`를 사용하여 타임스탬프 필드를 올바르게 채우고 있는지 확인해야 합니다.

  `timesync_status` 메시지에서 보고되는 거대한 오프셋 값은 정상적인 현상입니다.54 이는 PX4 클럭이 부팅 시 0에서 시작하는 반면, 컴패니언 컴퓨터는 유닉스 시간을 사용하기 때문에 발생합니다. uXRCE-DDS 브리지는 이 정적 오프셋을 올바르게 보상합니다. 그러나 이는 전체 시스템의 시간적 일관성이 uXRCE-DDS 브리지의 완벽한 작동에 의존한다는 중요한 점을 시사합니다. 섹션 1에서 논의된 Gazebo Harmony 호환성 문제는 

  `MicroXRCEAgent`나 기본 DDS 통신을 불안정하게 만들어 간헐적인 타이밍 오류를 유발하고 EKF에 공급되는 데이터를 손상시킬 수 있습니다. 따라서 `/fmu/out/timesync_status` 토픽의 `round_trip_time` (RTT)을 모니터링하는 것이 좋습니다. 안정적이고 낮은 RTT는 건강한 링크를 의미하며, 이 값의 스파이크나 누락은 기반 시뮬레이션 환경이나 DDS 브리지가 불안정하다는 강력한 지표가 될 수 있습니다.


- **SLAM의 본질:** FAST_LIO2와 같은 LIO 및 SLAM 알고리즘은 완벽한 추측 항법(dead-reckoning) 시스템이 아닙니다.20 루프 클로저(loop closure)가 발생하거나 맵이 대규모로 업데이트/재최적화될 때 주행계 출력에 갑작스러운 "점프"나 "저크(jerk)"가 발생할 수 있습니다.
- **EKF에 미치는 영향:** 입력되는 주행계의 갑작스럽고 큰 점프는 EKF에 거대한 이노베이션을 유발하여, EKF가 해당 데이터를 거부하거나 불안정해지는 원인이 될 수 있습니다. FAST_LIO2는 견고하게 설계되었지만 22, 이는 스캔-투-맵(scan-to-map) 방식의 주행계가 가진 내재적 특성입니다.20
- **완화 방안:** 이를 처리하는 주된 방법은 `nav_msgs::msg::Odometry` 메시지의 공분산 값을 튜닝하는 것입니다. 공분산 행렬은 EKF에게 해당 측정값을 얼마나 신뢰해야 하는지를 알려주는 역할을 합니다. 사용자의 브리지 노드는 `pose.covariance`와 `twist.covariance` 필드를 현실적인 값으로 채워야 합니다. 만약 FAST_LIO2가 이 값을 제공하지 않는다면, LIO 시스템의 예상 정확도를 반영하는 합리적인 상수 값으로 설정해야 합니다. 이는 EKF가 입력 데이터를 자체 예측과 비교하여 적절하게 가중치를 부여하는 데 도움이 됩니다.

------


이 마지막 섹션은 모든 분석 결과를 종합하여 명확하고 우선순위가 지정된 실행 계획으로 요약합니다.


사용자가 겪고 있는 드론의 비행 불안정성은 다음과 같은 복합적인 원인에 기인하는 것으로 분석됩니다.

- **주요 원인:** FAST_LIO2의 ENU 출력과 PX4 EKF2가 요구하는 NED 입력 간의 **치명적인 좌표계 변환 오류**, 특히 방향을 나타내는 **쿼터니언 변환의 수학적 부정확성**.
- **2차 원인:** 외부 주행계 데이터를 수용하고 융합하도록 필터를 설정하는 **PX4 EKF2 파라미터(`EKF2_EV_CTRL`, `EKF2_HGT_REF` 등)의 부정확하거나 불완전한 구성**.
- **악화 요인:** SITL 환경에서 근본적인 불안정성과 센서 데이터 누락을 유발하는 것으로 알려진 **고위험, 비표준 소프트웨어 스택(ROS2 Humble과 Gazebo Harmony의 조합)**.
- **3차 요인:** 공분산을 통해 올바르게 처리되지 않을 경우 불안정성을 악화시킬 수 있는 **시간 동기화 문제 및 LIO 알고리즘 고유의 주행계 점프 현상**.


문제 해결을 위해 다음의 우선순위에 따라 체계적으로 접근할 것을 강력히 권장합니다.

1. **기반 안정화:** 시뮬레이션 환경을 ROS2 Humble에서 공식적으로 지원하는 **Gazebo Fortress**로 전환합니다. 외부 주행계 없이 간단한 비행 모드에서 이륙 및 호버링을 통해 기본 시뮬레이션 환경의 안정성을 먼저 검증합니다.
2. **정확한 변환 구현:** 섹션 2에서 상세히 설명한 완전한 ENU-to-NED 변환(위치, 속도, 쿼터니언)을 C++ 브리지 노드에 구현합니다. 특히 쿼터니언 변환 로직을 주의 깊게 적용합니다.
3. **EKF2 설정:** 섹션 3의 파라미터 표를 참조하여, EKF2가 외부 시각 데이터를 융합하고 충돌하는 다른 소스(GPS, 자력계)를 무시하도록 모든 관련 파라미터를 정확하게 설정합니다.
4. **체계적 검증:** 섹션 4의 진단 워크플로우를 따릅니다. ROS2 도구와 RViz를 사용하여 비행 전 데이터 흐름을 확인하고, 비행 후 ULog 분석을 통해 EKF가 외부 데이터를 작고 안정적인 이노베이션으로 융합하고 있음을 데이터 기반으로 최종 확인합니다.
5. **미세 튜닝:** 시스템이 안정화되면, `EKF2_EV_DELAY`와 주행계 공분산 값을 미세 조정하여 LIO 고유의 아티팩트에 대한 성능과 견고성을 최적화합니다.


관찰된 비행 불안정성은 소프트웨어 호환성, 수학적 정확성, 그리고 견고한 설정을 포괄하는 다층적 접근을 통해 해결할 수 있는 전형적인 시스템 통합 문제입니다. 본 보고서에서 제시된 가이드를 체계적으로 따름으로써, 개발자는 당면한 문제를 해결할 수 있을 뿐만 아니라, 향후 유사한 자율 시스템 통합 프로젝트에서 마주할 수 있는 복잡한 문제들을 진단하고 해결하는 데 필요한 깊이 있는 지식과 실용적인 도구를 갖추게 될 것입니다. 성공적인 통합의 핵심은 각 구성 요소가 올바르게 작동하는지 개별적으로 검증하고, 점진적으로 시스템을 구축하며, 데이터 기반의 분석을 통해 각 단계의 건전성을 확인하는 데 있습니다.


1. Installing Gazebo with ROS - Gazebo ionic documentation, accessed July 1, 2025, https://gazebosim.org/docs/latest/ros_installation/
2. Installing Gazebo with ROS - Gazebo ionic documentation, accessed July 1, 2025, https://gazebosim.org/docs/latest/ros_installation
3. ROS2 Humble, Gazebo Harmonic, PX4, QGroundControl, Micro XRCE-DDS Agent & Client Installation | by Erdem | Medium, accessed July 1, 2025, https://medium.com/@erdem.ku.3.14/ros2-humble-gazebo-harmonic-px4-ve-micro-xrce-dds-agent-client-installation-aad32d8f5669
4. ROS 2 with Gazebo - Dev documentation - ArduPilot, accessed July 1, 2025, https://ardupilot.org/dev/docs/ros2-gazebo.html
5. [Bug] Gazebo SITL does not work if Gazebo Harmonic is installed / Issue #22180 / PX4/PX4-Autopilot - GitHub, accessed July 1, 2025, https://github.com/PX4/PX4-Autopilot/issues/22180
6. GAZEBO, ROS2 AND PX4 : r/ROS - Reddit, accessed July 1, 2025, https://www.reddit.com/r/ROS/comments/1izcu0s/gazebo_ros2_and_px4/
7. Simulating Drones using ROS2 : r/ROS - Reddit, accessed July 1, 2025, https://www.reddit.com/r/ROS/comments/1kxf2pb/simulating_drones_using_ros2/
8. ROS 2 User Guide | PX4 Guide (v1.15) - PX4 docs - PX4 Autopilot, accessed July 1, 2025, https://docs.px4.io/v1.15/en/ros2/user_guide.html
9. ROS 2 User Guide - GitHub, accessed July 1, 2025, https://github.com/PX4/PX4-user_guide/blob/main/ja/ros2/user_guide.md
10. SathanBERNARD/PX4-ROS2-Gazebo-Drone-Simulation-Template - GitHub, accessed July 1, 2025, https://github.com/SathanBERNARD/PX4-ROS2-Gazebo-Drone-Simulation-Template
11. PX4 Messages in ROS 2 - Auterion Documentation, accessed July 1, 2025, https://docs.auterion.com/app-development/app-framework/px4-messages-in-ros-2
12. Vicon Position Tracking w/ ROS2 / Issue #21232 / PX4/PX4-Autopilot - GitHub, accessed July 1, 2025, https://github.com/PX4/PX4-Autopilot/issues/21232
13. ARK-Electronics/ROS2_PX4_Offboard_Example: Example of controlling PX4 Velocity Setpoints with ROS2 Teleop controls - GitHub, accessed July 1, 2025, https://github.com/ARK-Electronics/ROS2_PX4_Offboard_Example
14. External Position Estimation / PX4 Developer Guide - jalpanchal1, accessed July 1, 2025, https://jalpanchal1.gitbooks.io/px4-developer-guide/en/ros/external_position_estimation.html
15. PX4-user_guide/tr/ros/external_position_estimation.md at main - GitHub, accessed July 1, 2025, https://github.com/PX4/PX4-user_guide/blob/main/tr/ros/external_position_estimation.md
16. Using Vision or Motion Capture Systems for Position Estimation | PX4 Guide (main), accessed July 1, 2025, https://docs.px4.io/main/en/ros/external_position_estimation.html
17. Check frame conversions (NED->ENU ROS) / Issue #49 / mavlink/mavros - GitHub, accessed July 1, 2025, https://github.com/mavlink/mavros/issues/49
18. PX4-user_guide/ja/advanced_config/tuning_the_ecl_ekf.md at main - GitHub, accessed July 1, 2025, https://github.com/PX4/PX4-user_guide/blob/main/ja/advanced_config/tuning_the_ecl_ekf.md?plain=1
19. Using Vision or Motion Capture Systems for Position Estimation | PX4 User Guide (v1.12), accessed July 1, 2025, https://docs.px4.io/v1.12/en/ros/external_position_estimation.html
20. FAST-LIO2: Fast Direct LiDAR-inertial Odometry - GitHub, accessed July 1, 2025, https://raw.githubusercontent.com/hku-mars/FAST_LIO/main/doc/Fast_LIO_2.pdf
21. hku-mars/FAST_LIO: A computationally efficient and robust LiDAR-inertial odometry (LIO) package - GitHub, accessed July 1, 2025, https://github.com/hku-mars/FAST_LIO
22. FAST-LIO2: Fast Direct LiDAR-inertial Odometry | Jiarong Lin, accessed July 1, 2025, https://jiaronglin.com/project/proj_fastlio/
23. Understanding Coordinate Transformations for Navigation - Automatic Addison, accessed July 1, 2025, https://automaticaddison.com/understanding-coordinate-transformations-for-navigation/
24. Publishing VIO Data to /fmu/in/vehicle_visual_odometry in ROS 2 (Python), accessed July 1, 2025, https://discuss.px4.io/t/publishing-vio-data-to-fmu-in-vehicle-visual-odometry-in-ros-2-python/45536
25. Ros2 /px4_1/fmu/in/vehicle_visual_odometry topic subscribe issue - PX4 Autopilot, accessed July 1, 2025, https://discuss.px4.io/t/ros2-px4-1-fmu-in-vehicle-visual-odometry-topic-subscribe-issue/46126
26. ENU -> NED frame conversion using quaternions - Stack Overflow, accessed July 1, 2025, https://stackoverflow.com/questions/49790453/enu-ned-frame-conversion-using-quaternions
27. ENU vs NED Quaternion? : r/AerospaceEngineering - Reddit, accessed July 1, 2025, https://www.reddit.com/r/AerospaceEngineering/comments/sdnh7d/enu_vs_ned_quaternion/
28. Coordinate transformation using quaternions incorrect / Issue #1586 / mavlink/mavros - GitHub, accessed July 1, 2025, https://github.com/mavlink/mavros/issues/1586
29. Reiterate over frame conversions between NED and ENU / Issue #216 / mavlink/mavros, accessed July 1, 2025, https://github.com/mavlink/mavros/issues/216
30. Using PX4's Navigation Filter (EKF2) | PX4 Guide (main), accessed July 1, 2025, https://docs.px4.io/main/en/advanced_config/tuning_the_ecl_ekf.html
31. ecl EKF / px4-devguide - bkueng, accessed July 1, 2025, https://bkueng.gitbooks.io/px4-devguide/en/tutorials/tuning_the_ecl_ekf.html
32. ECL/EKF Overview & Tuning - PX4 docs, accessed July 1, 2025, https://docs.px4.io/v1.11/en/advanced_config/tuning_the_ecl_ekf.html
33. Using Vision or Motion Capture Systems for Position Estimation ..., accessed July 1, 2025, https://docs.px4.io/v1.15/en/ros/external_position_estimation.html
34. PX4 Vision Estimation: Questions on Integration with Pixhawk, accessed July 1, 2025, https://discuss.px4.io/t/px4-vision-estimation-questions-on-integration-with-pixhawk/10237
35. EKF2 Heading issue with Px4 1.9.0 on Pixracer with External Vision Pose #12260 - GitHub, accessed July 1, 2025, https://github.com/PX4/Firmware/issues/12260
36. EKF2 not working with (only) External Vision inputs / Issue #19859 / PX4/PX4-Autopilot, accessed July 1, 2025, https://github.com/PX4/PX4-Autopilot/issues/19859
37. Parameter Reference / PX4 Developer Guide - jalpanchal1, accessed July 1, 2025, https://jalpanchal1.gitbooks.io/px4-developer-guide/en/advanced/parameter_reference.html
38. EKF2 estimation rejecting vision input - PX4 Discussion Forum, accessed July 1, 2025, https://discuss.px4.io/t/ekf2-estimation-rejecting-vision-input/15371
39. Autonomous flight using ekf2 and vision_pose - PX4 Discussion Forum, accessed July 1, 2025, https://discuss.px4.io/t/autonomous-flight-using-ekf2-and-vision-pose/36590
40. Better choice in GPS denied flight: Offboard or Mission? - PX4 Discussion Forum, accessed July 1, 2025, https://discuss.px4.io/t/better-choice-in-gps-denied-flight-offboard-or-mission/32998
41. Offboard Mode (Generic/All Frames) | PX4 Guide (v1.15) - PX4 docs, accessed July 1, 2025, https://docs.px4.io/v1.15/en/flight_modes/offboard.html
42. Setting Up Odometry - Gazebo - Nav2 1.0.0 documentation, accessed July 1, 2025, https://docs.nav2.org/setup_guides/odom/setup_odom_gz.html
43. tf2_monitor net delay and hz when playing ros2 bag - ROS Answers archive, accessed July 1, 2025, https://answers.ros.org/question/397980/
44. Make possible for ros2 topic hz to monitor the rates of multiple topics simultaneously as ros1 rostopic hz does. / Issue #830 / ros2/ros2cli - GitHub, accessed July 1, 2025, https://github.com/ros2/ros2cli/issues/830
45. NED to ENU conversion for PX4-ROS2 : r/ROS - Reddit, accessed July 1, 2025, https://www.reddit.com/r/ROS/comments/1kgdigu/ned_to_enu_conversion_for_px4ros2/
46. PX4/pyulog: Python module & scripts for ULog files - GitHub, accessed July 1, 2025, https://github.com/PX4/pyulog
47. estimator_status (UORB message) | PX4 User Guide (v1.13), accessed July 1, 2025, https://docs.px4.io/v1.13/en/msg_docs/estimator_status.html
48. EstimatorStatus (UORB message) | PX4 Guide (main), accessed July 1, 2025, https://docs.px4.io/main/en/msg_docs/EstimatorStatus.html
49. accessed January 1, 1970, https://docs.px4.io/main/en/simulation/gazebo.html
50. Simulation | PX4 Guide (main) - PX4 docs, accessed July 1, 2025, https://docs.px4.io/main/en/simulation/
51. PX4 SITL Multi-Vehicle Gazebo IMU timing errors, accessed July 1, 2025, https://discuss.px4.io/t/px4-sitl-multi-vehicle-gazebo-imu-timing-errors/33268
52. More info on the SITL Gazebo IMU plugin - PX4 Discussion Forum, accessed July 1, 2025, https://discuss.px4.io/t/more-info-on-the-sitl-gazebo-imu-plugin/8855
53. Syncing time with ROS2 computer and flight controller - PX4 Discussion Forum, accessed July 1, 2025, https://discuss.px4.io/t/syncing-time-with-ros2-computer-and-flight-controller/39928
54. Explanation TimeSync with ROS2 and XRCE-DDS bridge - PX4 Discussion Forum, accessed July 1, 2025, https://discuss.px4.io/t/explanation-timesync-with-ros2-and-xrce-dds-bridge/32504
55. How to sync time between Robot and Host machine for ROS2 | by RoboFoundry - Medium, accessed July 1, 2025, https://robofoundry.medium.com/how-to-sync-time-between-robot-and-host-machine-for-ros2-ecbcff8aadc4
56. How to Synchronize Time with Chrony NTP in Linux - Tecmint, accessed July 1, 2025, https://www.tecmint.com/synchronize-time-with-ntp-in-linux/

