# ROS2 Humble의 Tf2
[ROS2 Humble](./index.md)


로보틱스 시스템의 본질은 물리적 세계와의 상호작용에 있다. 로봇이 주변 환경을 인식하고, 목표를 설정하며, 정밀한 작업을 수행하기 위해서는 '공간'에 대한 일관되고 정확한 이해가 전제되어야 한다. 로봇에 부착된 수많은 센서로부터 들어오는 데이터, 모터를 움직이는 액추에이터 명령, 그리고 복잡한 경로를 계획하는 알고리즘에 이르기까지, 모든 정보는 결국 "어디에 있는가?"라는 근본적인 질문으로 귀결된다. 이 질문에 답하지 못하는 로봇은 그저 움직이는 기계 덩어리에 불과하다.

`tf2`는 바로 이 근본적인 질문에 답하기 위한 ROS 2의 표준 라이브러리이자 핵심 인프라다.1 

`tf2`를 단순히 좌표를 변환하는 유틸리티로 생각해서는 안 된다. `tf2`는 분산된 노드들이 각자의 시점에서 독립적으로 생성하는 시변(time-varying) 좌표계들의 관계를 실시간으로, 그리고 안정적으로 관리하는 정교한 시스템이다.3 로봇의 몸체(`base_link`), 바퀴, 로봇 팔의 각 관절, 카메라, 라이다 등 수많은 구성 요소들은 각자 고유의 좌표 프레임(coordinate frame)을 가지며, 이들의 상대적인 위치와 방향(pose)은 끊임없이 변한다. `tf2`는 이 복잡한 관계망을 하나의 일관된 '트리(tree)' 구조로 유지하고, 어떤 프레임에서든 다른 프레임으로 데이터를 변환할 수 있는 강력한 기능을 제공한다.

이 보고서는 ROS 2 Humble Hawksbill 버전을 기준으로, `tf2`의 가장 기본적인 개념부터 시작하여 내부 동작 메커니즘, 실전 프로그래밍, 고급 디버깅 전략, 그리고 시스템 성능 최적화에 이르기까지 모든 것을 심층적으로 파헤치는 것을 목표로 한다. 표면적인 사용법을 넘어 `tf2`가 왜 그렇게 설계되었는지, 그리고 그 설계가 실제 로보틱스 애플리케이션에 어떤 의미를 갖는지에 대한 깊이 있는 분석을 제공할 것이다.


`tf2`를 효과적으로 사용하기 위해서는 먼저 그 기반을 이루는 몇 가지 핵심 개념을 명확하게 이해해야 한다. 이 개념들은 `tf2` 시스템 전체의 동작 방식을 규정하는 규칙과도 같다.


로봇 시스템의 모든 물리적, 논리적 구성 요소는 자신만의 3차원 좌표 프레임(frame)을 가질 수 있다. 예를 들어, 로봇의 몸체 중심은 `base_link` 프레임, 왼쪽 바퀴의 중심은 `left_wheel_link` 프레임, 카메라 렌즈의 광학 중심은 `camera_optical_frame`을 가질 수 있다. `tf2`는 이러한 무수히 많은 프레임들 간의 상대적인 기하학적 관계, 즉 변환(transform) 정보를 관리한다.2

이러한 변환 관계들의 집합은 'TF 트리'라고 불리는 자료구조를 형성한다.3 TF 트리는 몇 가지 중요한 규칙을 따른다.

- **단일 부모 규칙(Single-Parent Rule)**: TF 트리의 가장 중요하고 강력한 제약 조건은 모든 프레임이 단 하나의 부모 프레임만을 가질 수 있다는 것이다.7 이는 A 프레임이 B 프레임의 자식이면서 동시에 C 프레임의 자식이 될 수 없음을 의미한다. 이 규칙 덕분에 TF 트리는 항상 순환이 없는 유향 트리(directed tree) 구조를 유지한다.
- **유일한 변환 경로**: 단일 부모 규칙은 임의의 두 프레임 간의 변환 경로가 항상 유일하게 존재함을 보장한다. 예를 들어 `camera_optical_frame`에서 `world` 프레임으로 데이터를 변환하려면, `camera_optical_frame` -> `camera_link` -> `base_link` -> `odom` -> `map` -> `world` 와 같이 트리를 따라 부모 방향으로 거슬러 올라가기만 하면 된다. 이 경로의 유일성은 변환 계산을 결정론적이고 매우 빠르게 만들어준다. 만약 `tf2`가 임의의 그래프(graph) 구조를 허용했다면, 두 프레임 간에 여러 경로가 존재할 수 있고, 최적 경로를 찾는 과정은 실시간 시스템에 큰 부담이 되었을 것이다. 트리 구조를 강제하는 것은 유연성을 일부 희생하는 대신, 실시간성과 계산 효율성을 확보하기 위한 의도적인 설계 결정이다.8

ROS 생태계에서는 일반적으로 통용되는 몇 가지 표준 프레임 이름이 있다. 이를 따르는 것은 다른 ROS 패키지와의 호환성을 위해 매우 중요하다.2

- `world` / `map`: 전역 고정 좌표계. 일반적으로 로봇이 동작하는 환경의 기준점으로, 시간이 지나도 변하지 않는다. SLAM(Simultaneous Localization and Mapping) 알고리즘이 생성하는 지도의 기준이 되는 프레임이 주로 `map`이다.
- `odom`: 주행 거리계(Odometry) 기준 좌표계. 로봇이 처음 켜진 위치를 원점으로 삼으며, 바퀴 엔코더나 IMU 같은 센서 측정치를 누적하여 계산된 로봇의 상대적인 이동을 나타낸다. 시간이 지남에 따라 오차가 누적되는 특징이 있다.
- `base_link`: 로봇의 기준 좌표계. 보통 로봇의 물리적 중심에 위치하며, 로봇의 다른 모든 부분(링크, 센서)들은 이 `base_link`에 대한 상대적인 위치로 정의된다.


`tf2`는 프레임 간의 변환을 두 가지 종류로 구분하여 처리하는데, 이는 시스템 성능에 지대한 영향을 미치는 중요한 설계 특징이다.

- **정적 변환 (Static Transform)**: 시간이 지나도 절대 변하지 않는 프레임 간의 관계를 의미한다.9 예를 들어, 로봇의 `base_link`에 단단히 고정된 라이다 센서의 `laser_frame`은 로봇이 움직이더라도 `base_link`에 대한 상대적 위치와 방향이 항상 동일하다. 이러한 변환은 시스템이 시작될 때 단 한 번만 계산하고 정의하면 충분하다.

- **동적 변환 (Dynamic Transform)**: 시간에 따라 계속해서 변하는 프레임 간의 관계를 의미한다.9 가장 대표적인 예는 `odom` 프레임에 대한 로봇 `base_link`의 변환이다. 로봇이 움직이면 `base_link`의 위치와 방향은 `odom` 프레임에 대해 계속해서 변하게 된다.

`tf2`는 이 두 가지 변환을 서로 다른 ROS 토픽을 통해 처리한다.

- `/tf_static`: 정적 변환은 이 토픽을 통해 발행된다. `/tf_static`은 'latched' 토픽으로 설정되어 있어, 한 번 발행된 메시지는 토픽에 계속 유지된다. 새로 구독을 시작하는 노드는 가장 최근에 발행된 정적 변환 메시지를 즉시 수신할 수 있다. 이 덕분에 정적 변환 브로드캐스터는 시스템 시작 시 단 한 번만 변환 정보를 발행하면 되며, 이는 불필요한 네트워크 트래픽과 모든 리스너 노드의 CPU 부하를 극적으로 줄여주는 핵심적인 최적화 기능이다.2 리스너 또한 정적 변환에 대해서는 시간 이력을 저장할 필요가 없으므로 메모리 사용량도 절약된다.4 ROS1 `tf`에서는 모든 변환이 동일한 토픽으로 계속 발행되어 발생했던 성능 병목 현상을 해결하기 위한 직접적인 개선점이다.
- `/tf`: 동적 변환은 이 토픽을 통해 지속적으로 발행된다. 로봇의 위치를 추정하는 노드(예: odometry 노드)는 계산된 최신 위치를 주기적으로 `/tf` 토픽에 발행해야 한다.


`tf2` 시스템은 정보를 생성하는 '브로드캐스터'와 정보를 소비하는 '리스너'라는 두 가지 주요 역할로 구성된다.

- **브로드캐스터 (Broadcaster)**: 특정 프레임 간의 변환 정보를 계산하여 `/tf` 또는 `/tf_static` 토픽으로 발행(broadcast)하는 노드를 말한다.2 예를 들어, 로봇의 관절 각도를 읽어 각 링크의 위치를 계산하는 노드나, SLAM 알고리즘을 실행하여 `map` 프레임과 `odom` 프레임 간의 관계를 계산하는 노드가 브로드캐스터에 해당한다.
- **리스너 (Listener)**: `/tf`와 `/tf_static` 토픽을 구독하여 내부적으로 완전한 TF 트리를 실시간으로 재구성하고 유지하는 노드를 말한다.2 리스너는 사용자의 요청이 있을 때, 내부 버퍼에 저장된 변환 정보들을 사용하여 특정 시점의 프레임 간 변환을 계산하고 제공하는 역할을 한다.

`tf2`는 분산 시스템으로 설계되었기 때문에, 시스템 내의 모든 노드가 독립적으로 리스너가 되어 TF 정보를 얻을 수 있다.4 또는, 리소스가 제한적인 노드를 위해 중앙 집중식 TF 서버(`buffer_server`)를 두고, 다른 노드들이 액션(action) 통신을 통해 원격으로 변환 정보를 쿼리하는 것도 가능하다.11

개발 및 디버깅 과정에서 `tf2` 시스템의 상태를 빠르고 직관적으로 파악하기 위한 몇 가지 필수적인 명령줄 인터페이스(CLI) 도구가 제공된다.

**표 1: 주요 `tf2` CLI 도구와 기능**

| 명령어                       | 패키지      | 설명                                                         |
| ---------------------------- | ----------- | ------------------------------------------------------------ |
| `tf2_echo`                   | `tf2_ros`   | 지정된 두 프레임 간의 최신 변환 정보를 터미널에 지속적으로 출력한다. 특정 변환이 제대로 발행되고 있는지 확인하는 가장 기본적인 방법이다.15 |
| `view_frames`                | `tf2_tools` | 현재 ROS 네트워크에서 발행되고 있는 모든 TF 정보를 수집하여 전체 TF 트리의 구조를 PDF 파일로 생성해준다. 트리가 끊어졌는지(bifurcated) 등을 시각적으로 확인하는 데 필수적이다.15 |
| `tf2_monitor`                | `tf2_ros`   | 모든 TF 체인에 대해 발행 주기(rate), 지연 시간(delay) 등 상세한 통계 정보를 실시간으로 모니터링한다. 시간 관련 문제를 디버깅할 때 매우 유용하다.15 |
| `static_transform_publisher` | `tf2_ros`   | 터미널에서 직접 정적 변환을 발행할 수 있게 해준다. 간단한 테스트나 임시 프레임을 설정할 때 편리하다.5 |


`tf2`의 개념을 이해했다면, 이제 실제 코드에서 어떻게 활용되는지 살펴볼 차례다. C++과 Python에서 정적/동적 변환을 발행하고 조회하는 핵심 패턴을 분석한다.


정적 변환은 주로 로봇의 고정된 부품 간의 관계를 정의하는 데 사용된다.

- **CLI 사용**: 가장 간단한 방법은 `static_transform_publisher` CLI 도구를 사용하는 것이다. 예를 들어, `world` 프레임에서 x축으로 2m, y축으로 1m 이동하고 z축(yaw)으로 45도(0.785 라디안) 회전한 위치에 `robot_1` 프레임을 생성하려면 다음과 같이 실행한다.17

  ```Bash
  ros2 run tf2_ros static_transform_publisher --x 2 --y 1 --z 0 --yaw 0.785 --pitch 0 --roll 0 --frame-id world --child-frame-id robot_1
  ```

  이 방식은 테스트나 간단한 프로토타이핑에 매우 유용하며, launch 파일에서도 노드로 실행할 수 있다.19

- **C++ `StaticTransformBroadcaster`**: C++ 노드 내에서 프로그래밍 방식으로 정적 변환을 발행하려면 `tf2_ros::StaticTransformBroadcaster`를 사용한다.17

  ```C++
  #include <memory>
  #include "geometry_msgs/msg/transform_stamped.hpp"
  #include "rclcpp/rclcpp.hpp"
  #include "tf2/LinearMath/Quaternion.h"
  #include "tf2_ros/static_transform_broadcaster.h"
  
  class StaticPublisherNode : public rclcpp::Node
  {
  public:
      StaticPublisherNode() : Node("static_transform_publisher_node")
      {
          // StaticTransformBroadcaster 객체 생성
          tf_static_broadcaster_ = std::make_shared<tf2_ros::StaticTransformBroadcaster>(this);
  
          // 노드 초기화 시점에 변환 정보 생성 및 발행
          this->publish_transform();
      }
  private:
      void publish_transform()
      {
          geometry_msgs::msg::TransformStamped t;
  
          t.header.stamp = this->get_clock()->now();
          t.header.frame_id = "world";
          t.child_frame_id = "mystaticframe";
          t.transform.translation.x = 1.0;
          t.transform.translation.y = 0.5;
          t.transform.translation.z = 0.0;
  
          tf2::Quaternion q;
          q.setRPY(0, 0, 0); // Roll, Pitch, Yaw
          t.transform.rotation.x = q.x();
          t.transform.rotation.y = q.y();
          t.transform.rotation.z = q.z();
          t.transform.rotation.w = q.w();
  
          // sendTransform 호출
          tf_static_broadcaster_->sendTransform(t);
      }
      std::shared_ptr<tf2_ros::StaticTransformBroadcaster> tf_static_broadcaster_;
  };
  ```

  핵심은 생성자에서 `StaticTransformBroadcaster`를 초기화하고, `sendTransform`을 한 번만 호출하여 `/tf_static` 토픽으로 변환을 발행하는 것이다.

- **Python `StaticTransformBroadcaster`**: Python에서도 C++과 거의 동일한 로직으로 `tf2_ros.StaticTransformBroadcaster`를 사용할 수 있다.19

실제 복잡한 로봇 애플리케이션에서는 개별 노드나 CLI로 정적 변환을 발행하는 경우는 드물다. 대부분의 정적 변환, 즉 로봇의 물리적 구조에 의해 결정되는 변환들은 URDF(Unified Robot Description Format) 파일에 'fixed' 타입의 관절로 정의된다. 그리고 `robot_state_publisher`라는 특수한 노드가 이 URDF 파일을 읽고, 정의된 모든 고정 관절 정보를 `/tf_static` 토픽으로 자동으로 발행해준다.20 이 방식은 로봇의 기하학적 모델과 TF 발행을 하나의 파일(URDF)로 일원화하여 관리하므로, 유지보수성이 뛰어나고 오류 발생 가능성을 줄인다. 따라서 개발자가 `StaticTransformBroadcaster`를 직접 코딩하는 경우는, URDF에 정의되지 않은 가상의 프레임(예: 외부 카메라 보정 후 결정된 마커의 위치)을 발행하는 등 비교적 제한적인 상황이다.


로봇의 움직임과 같이 시간에 따라 변하는 관계는 동적 변환으로 발행해야 한다.

- **C++ `TransformBroadcaster`**: `turtlesim` 예제에서 거북이의 `Pose` 메시지를 구독하여 TF로 변환하는 코드를 살펴보자.21

  ```C++
  #include <memory>
  #include "geometry_msgs/msg/transform_stamped.hpp"
  #include "rclcpp/rclcpp.hpp"
  #include "tf2_ros/transform_broadcaster.h"
  #include "turtlesim/msg/pose.hpp"
  //... 기타 헤더
  
  class DynamicPublisherNode : public rclcpp::Node
  {
  public:
      DynamicPublisherNode() : Node("dynamic_transform_publisher_node")
      {
          // TransformBroadcaster 객체 생성
          tf_broadcaster_ = std::make_unique<tf2_ros::TransformBroadcaster>(*this);
  
          // turtlesim의 pose 토픽 구독
          subscription_ = this->create_subscription<turtlesim::msg::Pose>(
              "/turtle1/pose", 10, 
              std::bind(&DynamicPublisherNode::pose_callback, this, std::placeholders::_1));
      }
  private:
      void pose_callback(const std::shared_ptr<turtlesim::msg::Pose> msg)
      {
          geometry_msgs::msg::TransformStamped t;
  
          // 헤더 정보 채우기
          t.header.stamp = this->get_clock()->now();
          t.header.frame_id = "world";
          t.child_frame_id = "turtle1";
  
          // Pose 메시지로부터 변환 정보 채우기
          t.transform.translation.x = msg->x;
          t.transform.translation.y = msg->y;
          t.transform.translation.z = 0.0;
  
          tf2::Quaternion q;
          q.setRPY(0, 0, msg->theta);
          t.transform.rotation.x = q.x();
          t.transform.rotation.y = q.y();
          t.transform.rotation.z = q.z();
          t.transform.rotation.w = q.w();
  
          // sendTransform 호출하여 /tf 토픽으로 발행
          tf_broadcaster_->sendTransform(t);
      }
      std::unique_ptr<tf2_ros::TransformBroadcaster> tf_broadcaster_;
      rclcpp::Subscription<turtlesim::msg::Pose>::SharedPtr subscription_;
  };
  ```

  이 코드의 핵심 패턴은 센서 데이터(여기서는 `Pose` 메시지)를 수신하는 콜백 함수나 주기적인 타이머 루프 내에서, `TransformStamped` 메시지를 최신 정보로 채우고 `TransformBroadcaster`의 `sendTransform` 메서드를 호출하는 것이다.

- **Python `TransformBroadcaster`**: Python 버전의 코드도 구조적으로 매우 유사하다.10

여기서 매우 중요한 점은 `TransformStamped` 메시지의 `header.stamp` 필드다. 이 타임스탬프는 단순히 메시지를 발행하는 현재 시간을 기록하는 것이 아니다. 이 값은 "이 변환 정보가 유효한 시점"을 명시하는, `tf2`의 시간 기반 시스템에서 가장 핵심적인 정보다.2 만약 브로드캐스터가 이 타임스탬프를 부정확하게 설정하면(예: 항상 0으로 설정하거나, 실제 데이터가 측정된 시점과 큰 차이가 나는 시간을 설정하면), 리스너는 올바른 시간 보간을 수행할 수 없게 된다. 이는 결국 

`ExtrapolationException`을 발생시키거나, 심지어 예외 없이 부정확한 변환 값을 반환하여 시스템 전체에 치명적인 오류를 유발할 수 있다. 따라서 `header.stamp`는 해당 변환 정보가 측정되거나 계산된 '실제 시간'을 최대한 정확하게 반영해야 한다. 시뮬레이션 환경에서는 `use_sim_time` 파라미터를 `true`로 설정하고, `/clock` 토픽에서 제공하는 시뮬레이션 시간을 사용해야 시간 동기화 문제를 피할 수 있다.25


발행된 변환 정보는 리스너 노드에서 조회하여 활용한다.

- **C++ `Buffer`와 `TransformListener`**: C++에서 변환을 조회하기 위해서는 `tf2_ros::Buffer`와 `tf2_ros::TransformListener`가 필요하다.14

  ```C++
  #include <memory>
  #include "rclcpp/rclcpp.hpp"
  #include "tf2_ros/buffer.h"
  #include "tf2_ros/transform_listener.h"
  #include "tf2/exceptions.h"
  //... 기타 헤더
  
  class ListenerNode : public rclcpp::Node
  {
  public:
      ListenerNode() : Node("transform_listener_node")
      {
          // 버퍼와 리스너 초기화
          tf_buffer_ = std::make_unique<tf2_ros::Buffer>(this->get_clock());
          tf_listener_ = std::make_shared<tf2_ros::TransformListener>(*tf_buffer_);
  
          // 1초마다 on_timer 콜백 호출
          timer_ = this->create_wall_timer(
              std::chrono::seconds(1),
              std::bind(&ListenerNode::on_timer, this));
      }
  private:
      void on_timer()
      {
          std::string from_frame = "turtle1";
          std::string to_frame = "world";
          geometry_msgs::msg::TransformStamped transform_stamped;
  
          try {
              // 변환 조회
              transform_stamped = tf_buffer_->lookupTransform(
                  to_frame, from_frame,
                  tf2::TimePointZero); // 가장 최신 변환 요청
          } catch (const tf2::TransformException & ex) {
              RCLCPP_INFO(this->get_logger(), "Could not transform %s to %s: %s",
                  from_frame.c_str(), to_frame.c_str(), ex.what());
              return;
          }
  
          // 조회된 변환 정보 활용
          RCLCPP_INFO(this->get_logger(), "Transform from %s to %s is available!", 
              from_frame.c_str(), to_frame.c_str());
      }
      std::unique_ptr<tf2_ros::Buffer> tf_buffer_;
      std::shared_ptr<tf2_ros::TransformListener> tf_listener_;
      rclcpp::TimerBase::SharedPtr timer_;
  };
  ```

  `TransformListener`는 생성과 동시에 `/tf`와 `/tf_static` 토픽 구독을 시작하고, 수신된 데이터를 `Buffer`에 자동으로 채워 넣는다. 개발자는 `Buffer`의 `lookupTransform()` 메서드를 호출하기만 하면 된다. 이때 `lookupTransform`은 다양한 예외(`tf2::TransformException`의 서브클래스들)를 발생시킬 수 있으므로, 반드시 `try-catch` 블록으로 감싸야 한다.

- **Python `Buffer`와 `TransformListener`**: Python에서도 동일한 개념이 적용된다. `tf2_ros.Buffer`와 `tf2_ros.TransformListener`를 생성하고, `buffer.lookup_transform()`을 호출한다.10

조회된 `TransformStamped` 메시지는 그 자체로도 유용하지만, 보통은 다른 종류의 데이터(예: 센서가 측정한 3D 포인트, 로봇 팔의 목표 자세 등)를 다른 좌표계로 변환하는 데 사용된다. `tf2_geometry_msgs`와 같은 헬퍼 패키지는 `geometry_msgs`에 정의된 다양한 타입(예: `PointStamped`, `PoseStamped`, `Vector3Stamped`)을 쉽게 변환할 수 있는 함수들을 제공하여 이 과정을 단순화해준다.27


`tf2`를 진정으로 마스터하기 위해서는 API 사용법을 넘어 그 내부에서 데이터가 어떻게 저장, 관리, 계산되는지 이해해야 한다. 이는 특히 복잡한 디버깅 상황에서 문제의 근본 원인을 파악하는 데 결정적인 역할을 한다.


`tf2`의 아키텍처에서 가장 주목할 만한 점은 핵심 로직과 ROS 의존성 부분의 분리다.

- **ROS 독립성**: `tf2`의 모든 핵심 알고리즘, 즉 TF 트리를 구성하고, 데이터를 저장하며, 시간 보간을 수행하는 로직은 `tf2::BufferCore`라는 클래스에 구현되어 있다.11 이 클래스는 ROS 메시징 시스템이나 통신 라이브러리에 대한 어떠한 의존성도 갖지 않는 순수 C++ 라이브러리다. 우리가 C++ 코드에서 사용하는 `tf2_ros::Buffer`는 이 `BufferCore`를 상속받아 ROS 통신 기능(토픽 구독 등)을 추가한 래퍼(wrapper) 클래스다.29 이러한 모듈식 설계는 ROS2 전체의 설계 철학을 반영하는 것으로, ROS1의 많은 핵심 패키지들이 ROS 시스템과 너무 강하게 결합되어 있어 재사용과 단위 테스트가 어려웠던 문제를 해결한다.30 이 덕분에 `tf2`의 핵심 변환 기능은 ROS가 아닌 다른 프로젝트에서도 쉽게 가져다 쓸 수 있으며, 테스트 용이성 또한 크게 향상된다.8

- **`TimeCache`**: `BufferCore` 내부에서는 각 프레임 간의 변환(하나의 부모-자식 관계) 데이터를 `TimeCache`라는 자료구조에 저장한다.28

  `TimeCache`는 본질적으로 타임스탬프 순서로 정렬된 변환 데이터의 연결 리스트(linked list)다. 새로운 변환 데이터가 들어오면, 타임스탬프에 맞는 위치에 삽입된다.

- **10초 버퍼**: `TimeCache`는 기본적으로 최근 10초(`DEFAULT_MAX_STORAGE_TIME`) 분량의 변환 데이터만 저장하도록 설정되어 있다.8 10초보다 오래된 데이터는 버퍼에서 자동으로 삭제된다. 이 10초라는 시간은 대부분의 실시간 로봇 제어 및 인식 작업에는 충분한 길이다. 하지만 로봇의 전체 이동 경로를 분석하는 등 더 긴 시간의 데이터가 필요하다면, `tf2` 버퍼에만 의존해서는 안 되며 rosbag과 같은 외부 저장소에 데이터를 기록해야 한다.


리스너가 특정 시점 T에 대한 변환을 요청했을 때, 버퍼에 정확히 T 시점의 데이터가 존재할 확률은 거의 없다. `tf2`의 강력함은 바로 이럴 때 발휘된다. `tf2`는 버퍼에 저장된 이산적인(discrete) 시간의 데이터들을 사용하여 요청된 시점의 변환을 '보간(interpolate)'하여 추정한다.7

- **보간 과정**:
  1. 리스너가 `lookupTransform`을 통해 시점 T의 변환을 요청한다.
  2. `BufferCore`는 해당 변환 체인을 구성하는 각 링크(부모-자식 쌍)의 `TimeCache`를 확인한다.
  3. 각 `TimeCache` 내부에서는 `findClosest`와 유사한 함수가 요청 시점 T를 시간적으로 감싸는 가장 가까운 두 개의 데이터(시점 T1 < T, 시점 T2 > T)를 찾는다.31
  4. `interpolate` 함수는 이 두 데이터를 사용하여 시점 T에서의 변환 값을 계산한다.31 이동(translation) 성분은 간단한 선형 보간(linear interpolation)을 사용한다. 회전(rotation) 성분인 쿼터니언에 대해서는 두 쿼터니언 사이의 최단 경로를 일정한 각속도로 움직이는 구면 선형 보간(SLERP, Spherical Linear Interpolation)을 사용하여 부드러운 회전 변화를 계산한다.
  5. 체인을 구성하는 모든 링크에 대해 보간된 변환을 구한 뒤, 이들을 모두 곱하여 최종적으로 요청된 두 프레임 간의 변환을 계산한다.

이 보간 메커니즘은 `tf2`가 서로 다른 주기와 지연을 갖는 비동기적인 데이터 소스들을 통합하여 일관된 시공간적 정보를 제공할 수 있게 하는 핵심 기술이다. 하지만 이 강력한 기능에는 암묵적인 가정이 내포되어 있다. 바로 '로봇의 움직임은 두 데이터 포인트 사이에서 부드럽고 연속적일 것'이라는 가정이다. 만약 로봇의 움직임에 비해 TF 발행 주기가 너무 낮다면(예: 빠르게 회전하는 로봇의 TF를 1 Hz로 발행), 보간으로 추정된 중간 시점의 자세는 실제 로봇의 자세와 큰 오차를 보일 수 있다.32 따라서 개발자는 시스템의 최대 동역학(최대 속도, 최대 각속도)을 고려하여 충분히 높은 주기로 TF를 발행해야 보간된 데이터의 정확성을 보장할 수 있다.


`tf2`를 사용하다 보면 가장 흔하게 마주치는 예외가 바로 `ExtrapolationException`이다. 이 예외는 `tf2`의 버그가 아니라, 분산 비동기 시스템의 본질적인 특성을 드러내는 중요한 신호다.

- **예외의 본질**: `ExtrapolationException`은 말 그대로 요청한 시간이 `TimeCache`에 저장된 데이터의 시간 범위를 '벗어났을(extrapolate)' 때 발생한다.34 즉, 버퍼에 있는 가장 최신 데이터보다 더 미래의 데이터를 예측해달라고 요청하거나, 버퍼에 있는 가장 오래된 데이터보다 더 과거의 데이터를 요구할 때 발생한다.

- **`lookupTransform(time=0)`의 함정**: `time=0`(또는 C++의 `tf2::TimePointZero`, Python의 `rclpy.time.Time()`)은 '가장 최신의(latest)' 변환을 요청하는 편리한 방법이지만, 역설적으로 `ExtrapolationException`의 주된 원인이 되기도 한다.36 문제는 TF 트리를 구성하는 여러 변환 데이터들이 서로 다른 노드에서, 다른 시간, 다른 주기로 발행된다는 점에 있다. 예를 들어, 

  `map` -> `base_link` 변환을 얻기 위해서는 `map` -> `odom` 변환과 `odom` -> `base_link` 변환이 모두 필요하다. 만약 `odom` -> `base_link` 변환의 최신 데이터가 T=10.1초에 도착했고, `map` -> `odom` 변환의 최신 데이터는 T=10.0초에 도착했다고 가정하자. 이 상황에서 리스너가 "가장 최신"의 `map` -> `base_link` 변환을 요청하면, `tf2`는 전체 체인이 유효한 가장 최신 시간인 T=10.1초를 기준으로 계산을 시도한다. 하지만 T=10.1초 시점의 `map` -> `odom` 데이터는 아직 버퍼에 없으므로, T=10.0초 데이터를 기반으로 미래를 '외삽(extrapolation)'해야만 한다. 이 때 `ExtrapolationException`이 발생한다.36

- **해결책: `timeout`**: 이 문제에 대한 가장 표준적이고 효과적인 해결책은 `lookupTransform` 함수에 `timeout` 인자를 제공하는 것이다.37

  ```C++
  // C++
  transform_stamped = tf_buffer_->lookupTransform(
      to_frame, from_frame, tf2::TimePointZero, std::chrono::milliseconds(50));
  ```

  ```Python
  # Python
  t = self.tf_buffer.lookup_transform(
      to_frame, from_frame, rclpy.time.Time(), timeout=rclpy.duration.Duration(seconds=0.05))
  ```

  `timeout`을 설정하면 `lookupTransform`은 즉시 예외를 발생시키는 대신, 요청한 변환에 필요한 모든 데이터가 버퍼에 도착할 때까지 지정된 시간(예: 50ms)만큼 기다린다. 대부분의 경우, 짧은 네트워크 지연 후에 필요한 데이터가 도착하므로 예외를 성공적으로 피할 수 있다. `ExtrapolationException`은 `tf2`가 "현재 내가 가진 불완전한 데이터로는 당신의 요청에 답할 수 없다"고 보내는 정직한 응답이다. 따라서 개발자는 `try-catch`와 `timeout`을 사용하여 이러한 '일시적인 데이터 불일치' 상황을 예상하고 우아하게 처리하는 방어적 프로그래밍(defensive programming)을 해야만 견고한(robust) ROS2 노드를 작성할 수 있다.


복잡한 로봇 시스템에서 `tf2` 관련 문제는 필연적으로 발생한다. `tf2`가 제공하는 강력한 디버깅 및 시각화 도구를 활용하면 문제의 원인을 체계적으로 추적할 수 있다.


터미널에서 직접 실행할 수 있는 CLI 도구들은 `tf2` 상태를 빠르게 진단하는 데 매우 효과적이다.

- **`tf2_echo`**: 두 프레임 간의 변환이 실제로 발행되고 있는지, 그 값은 올바른지 확인하는 가장 첫 번째 단계다.15 만약 `tf2_echo`에서 아무런 출력이 없다면, 브로드캐스터 노드가 실행되지 않았거나 토픽 이름이 잘못되었을 가능성이 높다.
- **`view_frames`**: TF 트리의 전체 구조를 시각적으로 확인하는 데 필수적인 도구다.15 이 도구를 실행하면 현재 네트워크의 TF 정보를 수집하여 `frames.pdf` 파일을 생성한다. 이 PDF를 열어보면 모든 프레임과 그 부모-자식 관계가 다이어그램으로 표시된다. 만약 트리가 두 개 이상의 부분으로 나뉘어 있다면(bifurcated tree), 이는 특정 변환이 발행되지 않아 트리가 끊어졌음을 의미하며, 문제 해결의 중요한 단서가 된다. 한 가지 유의할 점은, ROS1과 달리 ROS2에서는 발행 노드 이름이 `default_authority`로 표시되는 경우가 많다는 것이다.41 이는 ROS2의 분산적인 특성 때문이며, 어떤 노드가 특정 변환을 발행하는지 추적하기 어렵게 만들 수 있다.
- **`tf2_monitor`**: `ExtrapolationException`과 같은 시간 관련 문제를 진단하는 데 가장 강력한 도구다.15 이 도구는 모든 TF 체인에 대해 평균 발행 주기(rate), 데이터 도착 지연(delay), 버퍼 길이 등을 상세하게 보여준다. 만약 특정 변환 체인의 지연 시간이 비정상적으로 길다면, 해당 브로드캐스터 노드의 성능 문제나 네트워크 문제를 의심해볼 수 있다.


RViz2는 `tf2` 정보를 3D 공간에서 직관적으로 시각화하여 디버깅하는 데 없어서는 안 될 도구다.

- **TF Display Type**: RViz2의 좌측 'Displays' 패널에서 'Add' 버튼을 누르고 'TF' 디스플레이 타입을 추가하면, 현재 TF 트리에 있는 모든 프레임들이 좌표축과 함께 3D 뷰에 표시된다.18 각 프레임의 이름과 부모-자식 관계를 나타내는 화살표도 함께 표시할 수 있다.

- **`Fixed Frame`의 중요성**: RViz2에서 `tf2` 관련 문제를 겪는 가장 흔한 원인은 `Fixed Frame` 설정 오류다. 'Global Options' 패널의 `Fixed Frame`은 RViz2의 3D 뷰를 렌더링하는 기준이 되는 절대 좌표계를 의미한다.18 만약 이 `Fixed Frame`이 존재하지 않는 프레임 이름으로 설정되거나, 다른 프레임들과 연결되지 않은 고립된 프레임으로 설정되면, RViz2는 다른 프레임들의 위치를 계산할 수 없어 아무것도 표시하지 못하거나 오류를 낸다.44 디버깅의 첫 단계는 항상 `Fixed Frame`을 `world`, `map`, `odom`과 같이 TF 트리의 최상위에 존재하는 유효한 프레임으로 올바르게 설정하는 것이다. `tf2_echo`로는 변환이 잘 보이는데 RViz2에서는 보이지 않는다면 45, 십중팔구 `Fixed Frame` 설정 문제이거나, RViz2가 사용하는 시간과 TF 데이터의 타임스탬프가 동기화되지 않았기 때문이다.


`tf2` 관련 오류는 복잡해 보이지만, 근본 원인은 대개 몇 가지로 좁혀진다. 따라서 체계적인 접근법을 따르면 효율적으로 문제를 해결할 수 있다. `tf2` 디버깅은 크게 '연결성(Connectivity)'과 '시간(Time)'이라는 두 가지 축을 중심으로 이루어진다.

1. **연결성 문제 진단 (Is the path valid?)**: "요청한 프레임 간의 경로가 TF 트리에 물리적으로 존재하는가?"
   - **증상**: `tf2::LookupException`, `tf2::ConnectivityException`.
   - **진단 도구**: `ros2 run tf2_tools view_frames`를 실행하여 전체 트리를 확인한다. 요청한 프레임들이 보이는가? 트리가 끊어진 부분은 없는가? 프레임 이름에 오타는 없는가?
   - **추가 진단**: `ros2 run tf2_ros tf2_echo <parent_frame> <child_frame>`을 사용하여 문제가 의심되는 특정 링크의 변환이 실제로 발행되고 있는지 확인한다.
2. **시간 문제 진단 (Is the data available at the requested time?)**: "경로는 존재하지만, 요청한 특정 시간에 유효한 데이터가 없는가?"
   - **증상**: `tf2::ExtrapolationException`.
   - **진단 도구**: `ros2 run tf2_ros tf2_monitor <frame1> <frame2>`를 실행하여 문제가 되는 변환 체인의 발행 주기(rate)와 지연(delay)을 확인한다. 발행 주기가 너무 낮거나 지연이 크지 않은가?
   - **코드 확인**: 브로드캐스터 노드에서 `header.stamp`를 올바르게 설정하고 있는지 확인한다. 리스너 노드에서 `lookupTransform`을 호출할 때 어떤 시간을 요청하는지 확인한다.
   - **해결 시도**: 리스너의 `lookupTransform`에 적절한 `timeout`을 추가하거나, `tf2::TimePointZero`를 사용하여 가장 최신 변환을 요청하도록 수정해본다.15

이 두 가지 축을 기준으로 체계적으로 접근하면, 대부분의 `tf2` 오류를 논리적으로 분석하고 근본 원인을 찾아낼 수 있다.


`tf2`는 ROS1 시절의 `tf`를 계승하고 발전시킨 라이브러리다. ROS1 경험이 있는 개발자라면 `tf`와 `tf2`의 차이점을 이해하는 것이 중요하며, 복잡한 시스템에서는 성능 최적화 기법을 아는 것이 필수적이다.


ROS Hydro 배포판 이후로 `tf`는 공식적으로 `tf2`에 의해 대체되었다. `tf2`는 `tf`의 모든 기능을 포함하면서 여러 중요한 개선 사항을 도입했다.11

**표 2: ROS1 `tf` vs ROS2 `tf2` 핵심 비교**

| 기능            | ROS1 `tf`                                         | ROS2 `tf2`                                                   | 근거 / 영향                                                  |
| --------------- | ------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **정적 변환**   | 모든 변환을 `/tf` 토픽으로 발행                   | `/tf` (동적)와 `/tf_static` (정적) 토픽으로 분리 11          | 불필요한 데이터 전송을 제거하여 네트워크 및 CPU 부하를 크게 감소시키는 핵심적인 성능 최적화.11 |
| **코어 의존성** | ROS 시스템과 강하게 결합됨                        | ROS에 독립적인 `tf2::BufferCore` 라이브러리 분리 11          | 코어 로직의 재사용성과 테스트 용이성이 비약적으로 향상됨.11  |
| **API**         | 고정된 데이터 타입만 지원하는 비-템플릿 API       | `tf2::convert`를 사용하는 템플릿 기반 API 11                 | Eigen, KDL 등 다양한 외부 수학 라이브러리의 데이터 타입을 플러그인 형태로 쉽게 통합하고 변환할 수 있음. |
| **Python 지원** | C++ 래퍼(wrapper)를 통해 지원, 일부 불안정성 존재 | 순수 Python으로 통신 부분을 재작성하여 1급 시민(First-class)으로 지원 11 | Python으로 작성된 `tf2` 노드의 안정성과 성능이 크게 향상됨.  |
| **원격 쿼리**   | 모든 노드가 자체적으로 리스너와 버퍼를 가져야 함  | 액션(Action) 기반의 원격 쿼리(`buffer_server`) 지원 11       | 리소스가 제한적인 임베디드 노드는 TF 버퍼를 직접 유지할 필요 없이 중앙 서버에 변환을 요청할 수 있음. |
| **`tf_prefix`** | 지원했으나, 사용법이 혼란스럽고 비권장됨          | 공식적으로 지원 중단 11                                      | 다중 로봇 시나리오에서 프레임 이름 충돌 문제를 ROS2의 네임스페이스를 통해 더 명확하게 해결하도록 유도. |


수십, 수백 개의 프레임이 존재하는 복잡한 로봇 시스템에서는 `tf2`의 성능이 전체 시스템의 성능에 영향을 미칠 수 있다. 다음은 `tf2` 관련 성능을 최적화하기 위한 몇 가지 전략이다.

- **정적 변환의 적극적 활용**: 가장 기본적이고 효과적인 최적화다. 조금이라도 변하지 않는 관계는 반드시 `/tf_static`으로 발행해야 한다. 이는 시스템의 고정된 부하를 크게 줄여준다.6
- **발행 주기 최적화**: 동적 변환의 발행 주기는 시스템의 동역학을 고려하여 신중하게 결정해야 한다. 주기가 너무 높으면 불필요한 리소스를 낭비하고, 너무 낮으면 보간 오류로 인해 정확성이 떨어진다.47 로봇이 정지했을 때는 발행 주기를 낮추는 등의 동적 조절도 고려해볼 수 있다.
- **ROS2 아키텍처 활용**: `tf2` 자체의 성능을 최적화하는 것보다, `tf2`가 동작하는 ROS2 시스템 전체의 데이터 흐름을 최적화하는 것이 더 효과적인 경우가 많다.
  - **Composable Nodes**: 만약 TF 브로드캐스터와 리스너가 동일한 프로세스 내에서 실행될 수 있다면, 이들을 Composable Node로 구성하여 노드 간 통신에서 발생하는 메시지 직렬화/역직렬화 오버헤드를 완전히 제거할 수 있다. 이는 특히 높은 빈도로 발행되는 동적 변환에 효과적이다.48
  - **DDS 튜닝**: ROS2의 통신은 DDS(Data Distribution Service) 미들웨어를 기반으로 한다. 사용 중인 DDS 구현체(예: Fast DDS, Cyclone DDS)의 설정을 튜닝하거나, 네트워크 환경에 맞는 QoS(Quality of Service) 프로파일을 `tf` 및 `tf_static` 토픽에 적용하여 메시지 전송의 신뢰성과 효율을 높일 수 있다.49
  - **하드웨어 가속**: 고해상도 카메라 이미지나 포인트 클라우드와 같이 대용량 센서 데이터를 처리하고 그 결과를 TF와 연계하는 인식 파이프라인에서는 하드웨어 가속이 필수적이다. NVIDIA의 NITROS(NVIDIA Isaac Transport for ROS)와 같은 기술은 ROS2의 타입 어댑테이션(type adaptation) 기능을 활용하여 CPU와 GPU 간의 메모리 복사를 제거(zero-copy)한다. 이를 통해 `tf2`와 연계된 전체 인식 파이프라인의 성능을 극적으로 향상시킬 수 있다.50

`tf2`의 코어 로직은 이미 매우 효율적으로 작성되어 있으므로 8, 성능 병목은 `tf2` 자체보다는 `tf2`로 들어오거나 `tf2`에서 나가는 데이터의 전송 및 처리 과정에서 발생하는 경우가 많다. 따라서 시스템 전체의 관점에서 데이터 흐름을 최적화하는 것이 중요하다.


견고하고 유지보수하기 쉬운 `tf2` 코드를 작성하기 위해 다음의 모범 사례를 따르는 것이 좋다.

- **프레임 네이밍 규칙 준수**: REP-105 문서에 정의된 좌표 프레임 명명 규칙을 따른다. 예를 들어, 물리적 링크는 `_link` 접미사를, 가상의 프레임은 `_frame` 접미사를 사용하고, 모든 프레임 이름은 소문자와 밑줄(`_`)로만 구성한다.
- **견고한 리스너 작성**: 모든 `lookupTransform` 호출은 `try-catch` 블록으로 감싸고, 예외 상황에 대비하여 적절한 `timeout`을 설정한다. 이는 노드의 안정성을 보장하는 가장 기본적인 원칙이다.
- **논리적인 TF 트리 설계**: 로봇의 기구학적 구조를 명확하게 반영하는 논리적인 TF 트리를 설계한다. 불필요하게 깊거나 복잡한 변환 체인은 계산 지연을 유발할 수 있다.
- **방어적 프로그래밍**: `tf2` 관련 코드를 작성할 때는 항상 데이터가 제때 도착하지 않거나, 순서가 뒤바뀌거나, 유실될 수 있다는 분산 시스템의 특성을 염두에 두고 방어적으로 프로그래밍해야 한다.51

------


이 보고서를 통해 살펴본 바와 같이, `tf2`는 단순히 좌표를 변환하는 도구를 넘어, 로봇이 '자신'과 '세상'을 시공간적으로 이해하기 위한 핵심 데이터베이스이자 인프라다. 그 내부에는 분산 시스템, 시간 동기화, 데이터 보간 등 정교하고 복잡한 개념들이 녹아있다.

따라서 ROS2에서 `tf2`를 마스터한다는 것은 C++이나 Python API의 사용법을 익히는 것을 넘어, 분산 비동기 환경에서 발생하는 시간, 데이터 동기화, 그리고 네트워크 지연의 문제를 이해하고 이에 대처하는 능력을 갖추는 것을 의미한다. `ExtrapolationException`을 버그로 여기는 대신, 시스템의 상태를 알려주는 유용한 신호로 받아들이고, `timeout`과 `try-catch`로 이를 우아하게 처리하는 것이 바로 전문가의 접근 방식이다.

이 보고서에서 다룬 핵심 개념, 내부 동작 원리, 체계적인 디버깅 전략, 그리고 성능 최적화 기법들을 바탕으로, 개발자들은 더욱 복잡하고 견고하며 신뢰성 있는 로봇 시스템을 구축하는 데 자신감을 가질 수 있을 것이다. `tf2`의 원리를 깊이 이해하는 것은 결국 더 나은 로봇을 만드는 길로 이어진다.


1. ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/index.html
2. Tf2 - ROS 2 Documentation: Iron documentation, accessed July 26, 2025, https://docs.ros.org/en/iron/Concepts/Intermediate/About-Tf2.html
3. Tf2 - ROS 2 Documentation: Rolling documentation, accessed July 26, 2025, https://docs.ros.org/en/rolling/Concepts/Intermediate/About-Tf2.html
4. Tf2 - ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/Concepts/Intermediate/About-Tf2.html
5. Transformation | Husarion, accessed July 26, 2025, https://husarion.com/tutorials/ros2-tutorials/7-transformation/
6. About tf2 - ROS 2 Documentation: Foxy documentation, accessed July 26, 2025, https://docs.ros.org/en/foxy/Concepts/About-Tf2.html
7. tf2/CommonQuestions - ROS Wiki, accessed July 26, 2025, http://wiki.ros.org/tf2/CommonQuestions
8. tf2/Design - ROS Wiki, accessed July 26, 2025, http://wiki.ros.org/tf2/Design
9. tf2 - Steven Gong, accessed July 26, 2025, https://stevengong.co/notes/tf2
10. TF2 - The second generation of the transform library - ROS 2 workshop documentation, accessed July 26, 2025, https://ros2-industrial-workshop.readthedocs.io/en/latest/_source/navigation/ROS2-TF2.html
11. tf2/Migration - ROS Wiki, accessed July 26, 2025, http://wiki.ros.org/tf2/Migration
12. What is the time complexity tf2 core library - Robotics Stack Exchange, accessed July 26, 2025, https://robotics.stackexchange.com/questions/79226/what-is-the-time-complexity-tf2-core-library
13. Writing a listener (Python) - ROS 2 Documentation, accessed July 26, 2025, https://docs.ros.org/en/foxy/Tutorials/Intermediate/Tf2/Writing-A-Tf2-Listener-Py.html
14. Writing a listener (C++) - ROS 2 Documentation: Humble ..., accessed July 26, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/Tf2/Writing-A-Tf2-Listener-Cpp.html
15. Debugging - ROS 2 Documentation: Galactic documentation, accessed July 26, 2025, https://docs.ros.org/en/galactic/Tutorials/Intermediate/Tf2/Debugging-Tf2-Problems.html
16. Introducing tf2 - ROS 2 documentation documentation, accessed July 26, 2025, https://ros.ncnynl.com/en/humble/Tutorials/Intermediate/Tf2/Introduction-To-Tf2.html
17. Writing a static broadcaster (C++) - ROS 2 Documentation: Humble ..., accessed July 26, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/Tf2/Writing-A-Tf2-Static-Broadcaster-Cpp.html
18. The Transform System (tf2) | Articulated Robotics, accessed July 26, 2025, https://articulatedrobotics.xyz/tutorials/ready-for-ros/tf/
19. Writing a static broadcaster (Python) - ROS 2 Documentation, accessed July 26, 2025, https://docs.ros.org/en/rolling/Tutorials/Intermediate/Tf2/Writing-A-Tf2-Static-Broadcaster-Py.html
20. tf2 - ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/Tf2/Tf2-Main.html
21. Writing a broadcaster (C++) - tf2 - ROS documentation, accessed July 26, 2025, https://docs.ros.org/en/galactic/Tutorials/Intermediate/Tf2/Writing-A-Tf2-Broadcaster-Cpp.html
22. Writing a tf2 broadcaster (C++) - ROS/Tutorials, accessed July 26, 2025, [http://wiki.ros.org/tf2/Tutorials/Writing%20a%20tf2%20broadcaster%20%28C%2B%2B%29](http://wiki.ros.org/tf2/Tutorials/Writing a tf2 broadcaster (C%2B%2B))
23. Writing a broadcaster (Python) - ROS 2 Documentation: Rolling documentation, accessed July 26, 2025, https://docs.ros.org/en/rolling/Tutorials/Intermediate/Tf2/Writing-A-Tf2-Broadcaster-Py.html
24. Writing a broadcaster (Python) - ROS 2 Documentation: Humble ..., accessed July 26, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/Tf2/Writing-A-Tf2-Broadcaster-Py.html
25. Clock and Time - ROS2 Design, accessed July 26, 2025, https://design.ros2.org/articles/clock_and_time.html
26. Writing a listener (Python) - ROS 2 Documentation: Humble ..., accessed July 26, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/Tf2/Writing-A-Tf2-Listener-Py.html
27. ros2_cookbook/rclpy/tf2.md at main - GitHub, accessed July 26, 2025, https://github.com/mikeferguson/ros2_cookbook/blob/main/rclpy/tf2.md
28. tf2: tf2::BufferCore Class Reference - ROS documentation, accessed July 26, 2025, http://docs.ros.org/jade/api/tf2/html/classtf2_1_1BufferCore.html
29. tf2::BufferCoreInterface Class Reference - ROS Documentation, accessed July 26, 2025, https://docs.ros2.org/foxy/api/tf2/classtf2_1_1BufferCoreInterface.html
30. Changes between ROS 1 and ROS 2 - ROS2 Design, accessed July 26, 2025, http://design.ros2.org/articles/changes.html
31. tf2: tf2::TimeCache Class Reference - the official ROS docs, accessed July 26, 2025, http://docs.ros.org/jade/api/tf2/html/classtf2_1_1TimeCache.html
32. tf interpolation - ros - Robotics Stack Exchange, accessed July 26, 2025, https://robotics.stackexchange.com/questions/80896/tf-interpolation
33. ros - How does tf do interpolation? - Robotics Stack Exchange, accessed July 26, 2025, https://robotics.stackexchange.com/questions/32309/how-does-tf-do-interpolation
34. tf2::ExtrapolationException Class Reference - ROS Documentation, accessed July 26, 2025, https://docs.ros2.org/foxy/api/tf2/classtf2_1_1ExtrapolationException.html
35. Unexpected output of _chainAsVector() / Issue #749 / ros2/geometry2 - GitHub, accessed July 26, 2025, https://github.com/ros2/geometry2/issues/749
36. tf2::lookupTransform with time 0 throws Extrapolation / Issue #120 ..., accessed July 26, 2025, https://github.com/ros/geometry/issues/120
37. tf2_ros::Buffer Class Reference - ROS Documentation, accessed July 26, 2025, https://docs.ros2.org/foxy/api/tf2_ros/classtf2__ros_1_1Buffer.html
38. Learning about tf2 and time (Python) - ROS 2 Documentation, accessed July 26, 2025, https://ftp.udx.icscoe.jp/ros/ros_docs_mirror/en/humble/Tutorials/Tf2/Learning-About-Tf2-And-Time-Py.html
39. Learning about tf2 and time (Python) - ROS 2 Documentation: Xin ..., accessed July 26, 2025, https://daobook.github.io/ros2-docs/xin/Tutorials/Tf2/Learning-About-Tf2-And-Time-Py.html
40. ROS2 - Visualize TFs for a Robot with RViz and tf2_tools - YouTube, accessed July 26, 2025, https://www.youtube.com/watch?v=NuhSdA1G4pM
41. tf2_tools view_frames: who is "default_authority"? - Robotics Stack Exchange, accessed July 26, 2025, https://robotics.stackexchange.com/questions/114637/tf2-tools-view-frames-who-is-default-authority
42. rviz/DisplayTypes/TF - ROS Wiki, accessed July 26, 2025, http://wiki.ros.org/rviz/DisplayTypes/TF
43. ROS 2 - Data display with RViz2 - Stereolabs, accessed July 26, 2025, https://www.stereolabs.com/docs/ros2/rviz2
44. Rviz doesn't show the correct tf2 tree - ROS Answers archive, accessed July 26, 2025, https://answers.ros.org/question/418327/
45. rviz2 not showing transform properly, but tf2_echo does : r/ROS - Reddit, accessed July 26, 2025, https://www.reddit.com/r/ROS/comments/16ttj48/rviz2_not_showing_transform_properly_but_tf2_echo/
46. What is the difference between tf and tf2? - Robotics Stack Exchange, accessed July 26, 2025, https://robotics.stackexchange.com/questions/34349/what-is-the-difference-between-tf-and-tf2
47. TF2 Performance Guide 2: Is your toaster set to high? - YouTube, accessed July 26, 2025, https://m.youtube.com/watch?v=XfXeOpJ8dLU
48. Tutorials - ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/Tutorials.html
49. (PDF) ROS2 Real-time Performance Optimization and Evaluation - ResearchGate, accessed July 26, 2025, https://www.researchgate.net/publication/376215526_ROS2_Real-time_Performance_Optimization_and_Evaluation
50. Improve Perception Performance for ROS 2 Applications with NVIDIA Isaac Transport for ROS | NVIDIA Technical Blog, accessed July 26, 2025, https://developer.nvidia.com/blog/improve-perception-performance-for-ros-2-applications-with-nvidia-isaac-transport-for-ros/
51. ROS 2 developer guide - ROS 2 Documentation: Rolling documentation, accessed July 26, 2025, https://docs.ros.org/en/rolling/The-ROS2-Project/Contributing/Developer-Guide.html

