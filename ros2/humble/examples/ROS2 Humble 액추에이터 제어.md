# ROS2 Humble 액추에이터 제어
[ROS2 Humble 예제](./index.md)



로봇의 '근육'에 해당하는 액추에이터를 제어하는 것은 로보틱스 시스템 개발의 가장 근본적인 과제 중 하나다. ROS2 Humble 환경에서 이 과제에 접근하는 방식은 크게 두 가지로 나눌 수 있다. 각 방식은 서로 다른 설계 철학을 기반으로 하며, 프로젝트의 성격과 목표에 따라 적합성이 달라진다.1

첫 번째는 **직접 통신 방식(Direct Communication)**이다. 이 접근법은 ROS2의 가장 기본적이고 핵심적인 통신 메커니즘인 Publisher/Subscriber 모델을 활용한다.3 제어 로직을 담은 노드(Node)가 액추에이터에 전달할 명령(예: 속도, 위치)을 특정 토픽(Topic)으로 발행(Publish)하면, 하드웨어와 직접 연결된 다른 노드가 이 토픽을 구독(Subscribe)하여 수신된 명령에 따라 모터 드라이버나 GPIO 핀을 직접 구동한다.5 이 방식의 가장 큰 장점은 개념이 매우 단순하고 직관적이어서 빠른 프로토타이핑에 유리하다는 점이다. ROS2를 처음 접하거나 간단한 기능 검증이 목표일 때 신속하게 결과를 확인할 수 있다.7

두 번째는 **`ros2_control` 프레임워크 활용 방식**이다. 이는 단순한 통신을 넘어, 로봇 제어를 위한 포괄적인 표준 프레임워크를 제공한다.8 `ros2_control`은 하드웨어 추상화, 실시간 제어 루프, 컨트롤러 관리, 시뮬레이션과의 원활한 연동 등 산업 현장에서 요구되는 견고하고 확장 가능한 기능을 체계적으로 지원한다.1 제어 알고리즘(Controller)과 하드웨어 통신 로직(Hardware Interface)을 명확하게 분리함으로써, 코드의 모듈성과 재사용성을 극대화하는 것이 이 프레임워크의 핵심 철학이다.2


두 패러다임의 기저에는 서로 다른 설계 철학이 깔려 있다. 직접 통신 방식은 "일단 빠르게 움직여보자"는 실용주의적 접근에 가깝다. 개발자는 특정 하드웨어(예: 특정 모터 드라이버)를 제어하기 위한 코드를 ROS 노드 내에 직접 작성한다.6 이는 단기적으로는 개발 속도를 높일 수 있지만, 장기적으로는 여러 문제점을 야기한다. 하드웨어 드라이버 로직이 제어 노드에 강하게 결합되어 있어 코드의 재사용이 어렵고, 다른 종류의 로봇이나 시뮬레이터에 적용하려면 대대적인 코드 수정이 불가피하다. 또한, ROS2의 표준 통신은 실시간성을 보장하지 않기 때문에, 운영체제의 스케줄링 정책에 따라 제어 주기가 흔들리는 '지터(jitter)'가 발생하여 정밀 제어에 한계를 보인다.6

반면, `ros2_control` 프레임워크는 "견고하고 확장 가능한 표준 시스템을 만들자"는 시스템 엔지니어링 관점의 접근법을 취한다. 이 프레임워크는 제어 알고리즘을 담은 '컨트롤러'와 하드웨어와의 통신을 담당하는 '하드웨어 인터페이스'라는 두 가지 핵심 추상화 계층을 도입한다.12 이 분리 덕분에 개발자는 하드웨어의 종류에 상관없이 표준화된 인터페이스를 통해 제어 로직을 작성할 수 있다. 예를 들어, 차동 구동 로봇을 제어하던 `diff_drive_controller`를 로봇 팔을 제어하는 `joint_trajectory_controller`로 교체하는 작업이 단지 설정 파일(YAML)을 수정하는 것만으로 가능해진다.8 물론, 이러한 구조적 이점을 얻기 위해 초기 설정 과정이 직접 통신 방식보다 복잡하게 느껴질 수 있다. 하지만 이는 장기적인 개발 효율성, 시스템 안정성, 그리고 유지보수성을 위한 필수적인 투자로 볼 수 있다.2

이러한 선택은 단순한 코딩 스타일의 차이가 아니라, 프로젝트의 확장성, 유지보수성, 그리고 미래의 개발 속도를 결정하는 전략적 결정이다. '단순한' Publisher/Subscriber 방식의 선택은 단기적인 코딩 편의성을 얻는 대신, 장기적인 '아키텍처 부채(Architectural Debt)'를 쌓는 행위일 수 있다. 프로젝트가 복잡해질수록, 예를 들어 PID 제어를 추가하거나, 다른 로봇으로 이식하거나, 정밀한 시뮬레이션 연동이 필요해질 때, 직접 작성한 코드는 `ros2_control`이 이미 표준화된 방식으로 해결한 문제들을 다시 풀어야 하는 짐이 된다. 초기 단계에서는 `motor.set_speed()`와 같은 직접적인 코드가 매력적으로 보일 수 있다.6 하지만 여기에 PID 제어를 추가하려면 타이밍 문제, 상태 피드백 처리 등을 모두 직접 구현해야 한다. 로봇 팔을 차동 구동 로봇으로 바꾸려면 제어 로직 전체를 거의 새로 작성해야 할 수도 있다. 

`ros2_control`은 이러한 문제들을 '하드웨어 인터페이스'와 '컨트롤러'라는 추상화 계층으로 분리하여 해결했다.1 따라서 `ros2_control`의 초기 학습 곡선은 이러한 아키텍처 부채를 선제적으로 상환하고, 장기적으로 더 빠르고 안정적인 개발을 가능하게 하는 과정으로 이해해야 한다.

다음 표는 두 제어 패러다임의 주요 특징을 여러 평가 기준에 따라 비교 분석한 것이다.

**Table 1: 제어 패러다임 비교 분석**

| 평가 기준 (Criteria)                  | 직접 통신 (Pub/Sub)                                          | `ros2_control` 프레임워크                                    |
| ------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **초기 개발 속도**                    | 매우 빠름. 간단한 노드 2개만으로 구동 가능.5                 | 상대적으로 느림. URDF, YAML, 하드웨어 인터페이스 등 여러 구성요소에 대한 이해 필요.2 |
| **코드 재사용성**                     | 낮음. 하드웨어 제어 로직이 ROS 노드에 강하게 결합됨.6        | 매우 높음. 컨트롤러와 하드웨어 인터페이스가 분리되어 독립적으로 재사용 및 교체 가능.1 |
| **실시간 제어 보장**                  | 어려움. ROS 통신 지연 및 OS 스케줄링에 영향을 받음.11        | 용이함. 실시간 제어 루프를 위한 별도 스레드와 RT-safe 메커니즘 제공.13 |
| **컨트롤러 교체 용이성**              | 어려움. 제어 알고리즘 변경 시 코드 수정 필요.                | 매우 쉬움. YAML 설정 파일 변경만으로 다양한 표준 컨트롤러(PID, 궤적 추종 등)로 교체 가능.8 |
| **시뮬레이션-실물 전환(Sim-to-Real)** | 어려움. 시뮬레이션용 코드와 실제 하드웨어용 코드가 크게 다를 수 있음. | 매우 용이함. 동일한 컨트롤러 코드를 사용하고 하드웨어 인터페이스 플러그인만 교체하면 됨.5 |
| **표준 준수 및 생태계 호환성**        | 낮음. 비표준적인 토픽과 메시지 타입을 사용할 가능성 높음.    | 높음. MoveIt2, Navigation2 등 주요 ROS2 프레임워크와 표준화된 방식으로 연동.18 |
| **디버깅 및 시스템 분석**             | 제한적. `ros2 topic echo` 등으로 통신만 확인 가능.           | 강력함. `ros2 control` CLI, `rqt_controller_manager` 등 다양한 표준 도구로 시스템 상태 진단 가능.16 |



Publisher/Subscriber를 이용한 직접 구동 방식은 ROS2의 가장 기본적인 통신 모델을 액추에이터 제어에 그대로 적용한 것이다.3 이 구조는 매우 단순하다. 시스템은 최소 두 개의 노드로 구성된다.

1. **제어 노드 (Publisher):** 사용자의 입력(예: 조이스틱)이나 상위 레벨의 계획(예: 내비게이션 경로)에 따라 로봇이 어떻게 움직여야 하는지에 대한 명령을 생성한다. 이 명령은 표준화된 메시지 타입, 예를 들어 선속도와 각속도를 포함하는 `geometry_msgs/msg/Twist` 또는 각 조인트의 목표 위치나 속도를 배열로 담는 `std_msgs/msg/Float64MultiArray` 형태로 특정 토픽에 주기적으로 발행된다.4
2. **구동 노드 (Subscriber):** 실제 하드웨어와 연결된 노드로, 제어 노드가 발행하는 토픽을 구독한다. 메시지가 수신될 때마다 등록된 콜백(callback) 함수가 호출되며, 이 함수 안에서 수신된 데이터를 해석하여 실제 모터를 구동하기 위한 저수준 신호(예: PWM 듀티 사이클, 시리얼 통신 명령어)로 변환하여 하드웨어에 전송한다.5

이 방식의 핵심은 제어 로직과 하드웨어 구동 로직이 ROS 토픽이라는 비동기식 메시지 큐를 통해 느슨하게 연결된다는 점이다. 이는 노드 간의 독립성을 보장하지만, 통신 지연이나 메시지 손실 가능성을 내포하며, 이것이 정밀 제어에서 문제가 될 수 있다.11


가상의 단일 모터 속도를 제어하는 간단한 예제를 통해 직접 구동 방식을 구현해 본다. 제어 노드는 `-1.0`에서 `1.0` 사이의 속도 명령을 `std_msgs/msg/Float64` 메시지로 발행하고, 구동 노드는 이를 받아 콘솔에 출력하는 시나리오다.


먼저, 예제 코드를 담을 ROS2 패키지를 생성한다. Python과 C++ 버전을 모두 만들어 본다. 워크스페이스(`ros2_ws`)의 `src` 디렉토리에서 다음 명령을 실행한다.

```Bash
# Python 패키지 생성
ros2 pkg create --build-type ament_python simple_motor_control_py

# C++ 패키지 생성
ros2 pkg create --build-type ament_cmake --dependencies rclcpp std_msgs simple_motor_control_cpp
```

5


**Publisher (제어기: `speed_publisher.py`)**

```Python
import rclpy
from rclpy.node import Node
from std_msgs.msg import Float64
import math

class SpeedPublisherNode(Node):
    def __init__(self):
        super().__init__('speed_publisher_node')
        self.publisher_ = self.create_publisher(Float64, 'motor_speed_cmd', 10)
        timer_period = 0.1  # 10Hz
        self.timer = self.create_timer(timer_period, self.timer_callback)
        self.get_logger().info('Speed Publisher Node has been started.')
        self.time = 0.0

    def timer_callback(self):
        msg = Float64()
        # 시간에 따라 -1.0 ~ 1.0 사이를 움직이는 사인파 속도 생성
        msg.data = math.sin(self.time)
        self.publisher_.publish(msg)
        self.get_logger().info(f'Publishing speed command: {msg.data:.2f}')
        self.time += 0.1

def main(args=None):
    rclpy.init(args=args)
    node = SpeedPublisherNode()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

4

**Subscriber (구동기: `motor_driver.py`)**

```Python
import rclpy
from rclpy.node import Node
from std_msgs.msg import Float64

class MotorDriverNode(Node):
    def __init__(self):
        super().__init__('motor_driver_node')
        self.subscription = self.create_subscription(
            Float64,
            'motor_speed_cmd',
            self.listener_callback,
            10)
        self.get_logger().info('Motor Driver Node has been started.')

    def listener_callback(self, msg):
        speed_command = msg.data
        # 실제 하드웨어라면 여기서 모터 제어 함수를 호출
        # 예: set_motor_pwm(speed_command)
        self.get_logger().info(f'Received speed command: {speed_command:.2f}, Driving motor...')

def main(args=None):
    rclpy.init(args=args)
    node = MotorDriverNode()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

5

**설정 및 실행**

1. `package.xml`에 `<exec_depend>rclpy</exec_depend>`, `<exec_depend>std_msgs</exec_depend>`를 추가한다.
2. `setup.py`의 `entry_points`에 `'talker = simple_motor_control_py.speed_publisher:main'`, `'listener = simple_motor_control_py.motor_driver:main'`을 추가한다.
3. 워크스페이스 루트에서 `colcon build --packages-select simple_motor_control_py`로 빌드한다.
4. 두 개의 터미널을 열고 각각 `ros2 run simple_motor_control_py talker`와 `ros2 run simple_motor_control_py listener`를 실행하여 통신을 확인한다.


**Publisher (제어기: `speed_publisher.cpp`)**

```C++
#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/float64.hpp"
#include <chrono>
#include <cmath>

using namespace std::chrono_literals;

class SpeedPublisherNode : public rclcpp::Node {
public:
    SpeedPublisherNode() : Node("speed_publisher_node"), time_(0.0) {
        publisher_ = this->create_publisher<std_msgs::msg::Float64>("motor_speed_cmd", 10);
        timer_ = this->create_wall_timer(
            100ms, std::bind(&SpeedPublisherNode::timer_callback, this));
        RCLCPP_INFO(this->get_logger(), "Speed Publisher Node has been started.");
    }

private:
    void timer_callback() {
        auto msg = std_msgs::msg::Float64();
        msg.data = std::sin(time_);
        publisher_->publish(msg);
        RCLCPP_INFO(this->get_logger(), "Publishing speed command: %.2f", msg.data);
        time_ += 0.1;
    }

    rclcpp::Publisher<std_msgs::msg::Float64>::SharedPtr publisher_;
    rclcpp::TimerBase::SharedPtr timer_;
    double time_;
};

int main(int argc, char * argv) {
    rclcpp::init(argc, argv);
    rclcpp::spin(std::make_shared<SpeedPublisherNode>());
    rclcpp::shutdown();
    return 0;
}
```

20

**Subscriber (구동기: `motor_driver.cpp`)**

```C++
#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/float64.hpp"

class MotorDriverNode : public rclcpp::Node {
public:
    MotorDriverNode() : Node("motor_driver_node") {
        subscription_ = this->create_subscription<std_msgs::msg::Float64>(
            "motor_speed_cmd", 10, std::bind(&MotorDriverNode::listener_callback, this, std::placeholders::_1));
        RCLCPP_INFO(this->get_logger(), "Motor Driver Node has been started.");
    }

private:
    void listener_callback(const std_msgs::msg::Float64::SharedPtr msg) const {
        double speed_command = msg->data;
        // 실제 하드웨어라면 여기서 모터 제어 함수를 호출
        RCLCPP_INFO(this->get_logger(), "Received speed command: %.2f, Driving motor...", speed_command);
    }

    rclcpp::Subscription<std_msgs::msg::Float64>::SharedPtr subscription_;
};

int main(int argc, char * argv) {
    rclcpp::init(argc, argv);
    rclcpp::spin(std::make_shared<MotorDriverNode>());
    rclcpp::shutdown();
    return 0;
}
```

20

**설정 및 실행**

1. `CMakeLists.txt`에 각 소스 파일에 대한 `add_executable`과 `ament_target_dependencies`, `install` 규칙을 추가한다.
2. 워크스페이스 루트에서 `colcon build --packages-select simple_motor_control_cpp`로 빌드한다.
3. 두 개의 터미널에서 `ros2 run simple_motor_control_cpp speed_publisher_node`와 `ros2 run simple_motor_control_cpp motor_driver_node`를 실행하여 통신을 확인한다.


이론적인 예제를 넘어, 실제 하드웨어인 라즈베리파이에서 직접 구동 방식을 적용하는 사례를 살펴보자. 라즈베리파이는 저렴하고 접근성이 좋아 프로토타이핑에 널리 사용되며, 내장된 GPIO(General-Purpose Input/Output) 핀을 통해 모터 드라이버와 같은 외부 회로를 직접 제어할 수 있다.24


이 시나리오에서 구동 노드(`motor_driver_node`)는 더 이상 가상의 함수를 호출하는 대신, `RPi.GPIO`나 `pigpio`와 같은 Python 라이브러리를 사용하여 라즈베리파이의 GPIO 핀 상태를 직접 변경한다.6 예를 들어, 수신된 속도 명령(`Twist` 메시지의 `linear.x`)의 부호에 따라 두 개의 방향 핀(예: IN1, IN2)을 HIGH/LOW로 설정하여 모터의 회전 방향을 제어하고, 속도 값의 크기에 따라 PWM(Pulse Width Modulation) 핀의 듀티 사이클을 조절하여 모터의 회전 속도를 제어한다.6


다음은 `Twist` 메시지를 받아 라즈베리파이 GPIO를 직접 제어하는 `MotorSubscriber` 노드의 핵심 콜백 함수 예제다.6

```Python
# 이 코드는 개념 설명을 위한 의사코드에 가까움
# 실제 사용 시에는 핀 번호, PWM 주파수 등을 정확히 설정해야 함
import RPi.GPIO as GPIO

#... (노드 초기화 및 GPIO 설정 부분 생략)...

def listener_callback(self, msg):
    linear_vel = msg.linear.x
    angular_vel = msg.angular.z

    # 1. 방향 제어 (예: L298N 모터 드라이버)
    if linear_vel > 0: # 전진
        GPIO.output(self.RIGHT_MOTOR_FORWARD_PIN, GPIO.HIGH)
        GPIO.output(self.RIGHT_MOTOR_BACKWARD_PIN, GPIO.LOW)
    elif linear_vel < 0: # 후진
        GPIO.output(self.RIGHT_MOTOR_FORWARD_PIN, GPIO.LOW)
        GPIO.output(self.RIGHT_MOTOR_BACKWARD_PIN, GPIO.HIGH)
    else: # 정지
        GPIO.output(self.RIGHT_MOTOR_FORWARD_PIN, GPIO.LOW)
        GPIO.output(self.RIGHT_MOTOR_BACKWARD_PIN, GPIO.LOW)
    
    #... (왼쪽 모터 방향 제어 로직)...

    # 2. 속도 제어 (PWM)
    # 선형 속도를 PWM 듀티 사이클(0-100)로 변환
    duty_cycle = self.map_velocity_to_pwm(abs(linear_vel))

    # 각속도를 이용해 좌우 바퀴 속도 차등 조절 (조향)
    right_pwm = duty_cycle - angular_vel * self.TURN_COEFFICIENT
    left_pwm = duty_cycle + angular_vel * self.TURN_COEFFICIENT

    # PWM 듀티 사이클 설정
    self.pwm_right.ChangeDutyCycle(max(0, min(100, right_pwm)))
    self.pwm_left.ChangeDutyCycle(max(0, min(100, left_pwm)))
```


이 방식은 간단한 로봇을 빠르게 구동시키는 데는 효과적이지만, 명확한 한계를 가진다.

1. **실시간성 부재:** ROS2 노드는 리눅스 같은 범용 운영체제 위에서 일반적인 프로세스로 실행된다. `listener_callback`이 언제 호출될지는 전적으로 OS 스케줄러에 달려있다.11 네트워크 상태나 다른 프로세스의 부하에 따라 메시지 수신이 지연되거나 콜백 실행 주기가 불규칙해질 수 있다. 이는 고속으로 움직이거나 정밀한 궤적 추종이 필요한 로봇에서는 치명적인 문제가 된다.
2. **하드웨어 종속성:** 모터 드라이버의 종류, 연결된 GPIO 핀 번호 등 하드웨어에 대한 모든 정보가 구동 노드 코드 안에 하드코딩된다. 만약 모터 드라이버를 다른 모델로 교체하거나, 라즈베리파이가 아닌 다른 보드를 사용하게 되면 코드의 상당 부분을 수정해야 한다.
3. **피드백 처리의 어려움:** 실제 로봇 제어는 단순히 명령을 보내는 것(Open-loop)으로 끝나지 않는다. 엔코더 등을 통해 실제 모터의 회전 속도나 위치를 피드백 받아 목표값과의 오차를 보정하는 폐쇄 루프 제어(Closed-loop, 예: PID 제어)가 필수적이다. 이 방식에서는 피드백 값을 읽어오는 로직, PID 연산 로직, 타이밍 관리 등을 모두 개발자가 직접 구현해야 하는 부담이 있다.1

이러한 한계점들은 왜 `ros2_control`과 같은 표준화되고 실시간성을 고려한 제어 프레임워크가 필요한지에 대한 강력한 동기를 제공한다. `ros2_control`은 바로 이 문제들을 체계적으로 해결하기 위해 설계되었다.



`ros2_control`은 ROS1 시절의 `ros_control`을 계승하고 그 한계를 극복하기 위해 ROS2 환경에 맞춰 완전히 재설계된 차세대 제어 프레임워크다.8 그 핵심 철학은 세 가지 키워드로 요약할 수 있다: 

**추상화(Abstraction), 실시간성(Real-time), 모듈화(Modularity)**.

- **추상화:** `ros2_control`의 가장 중요한 특징은 제어 알고리즘과 실제 하드웨어 간의 **하드웨어 추상화 계층(Hardware Abstraction Layer, HAL)**을 제공하는 것이다.1 이를 통해 컨트롤러 개발자는 특정 모터 드라이버나 통신 프로토콜에 대해 전혀 알 필요 없이, '위치', '속도', '힘'과 같은 표준화된 인터페이스를 통해 로봇을 제어할 수 있다. 이는 소프트웨어(컨트롤러)를 하드웨어(액추에이터, 센서)로부터 완전히 분리(decoupling)시킨다.2

- **실시간성:** `ros2_control`은 정밀하고 안정적인 제어를 위해 실시간(real-time) 제어 루프를 지원하도록 설계되었다.13

  `Controller Manager`는 별도의 실시간 스레드에서 `read-update-write` 사이클을 엄격한 주기로 실행한다. 이를 통해 ROS2 통신 계층의 비결정성(non-determinism)으로부터 제어 루프를 격리하고, 예측 가능한 성능을 보장한다.13

- **모듈화:** 프레임워크의 모든 구성 요소(컨트롤러, 하드웨어 인터페이스)는 `pluginlib`를 통해 동적으로 로드되는 플러그인 형태로 구현된다.1 이는 시스템의 유연성을 극대화한다. 새로운 하드웨어를 지원하기 위해 새 하드웨어 인터페이스 플러그인을 추가하거나, 다른 제어 전략을 시도하기 위해 컨트롤러 플러그인을 교체하는 작업이 전체 시스템의 재컴파일 없이 가능하다.2

이러한 철학 덕분에 `ros2_control`은 단순한 라이브러리를 넘어, MoveIt2(모션 플래닝)나 Navigation2(자율주행)와 같은 상위 레벨의 프레임워크들이 하드웨어에 구애받지 않고 동작할 수 있도록 하는 ROS2 로봇 시스템의 핵심 '커널'과 같은 역할을 수행한다.18


`ros2_control` 프레임워크는 크게 네 가지 핵심 구성 요소가 유기적으로 상호작용하며 동작한다. 이들의 역할과 관계를 이해하는 것이 프레임워크를 마스터하는 첫걸음이다.12

1. **Controller Manager (CM):**

   - **역할:** 이름 그대로 컨트롤러와 하드웨어 자원을 총괄하는 **중앙 지휘소**다. 컨트롤러의 생명주기(로딩, 설정, 활성화, 비활성화, 언로딩)를 관리하고, 컨트롤러가 요구하는 하드웨어 인터페이스와 실제 하드웨어가 제공하는 인터페이스를 매칭시켜주는 역할을 한다.26 또한, 

     `ros2 control` CLI나 `rqt_controller_manager`와 같은 사용자 인터페이스를 위한 ROS 서비스 서버 역할을 수행한다.19

   - **핵심 동작:** CM의 가장 중요한 임무는 실시간 `update()` 루프를 실행하는 것이다. 이 루프는 (1) `Resource Manager`를 통해 하드웨어로부터 최신 상태를 읽고(`read`), (2) 활성화된 모든 컨트롤러의 `update()` 함수를 호출하여 제어 명령을 계산하고, (3) 계산된 명령을 다시 `Resource Manager`를 통해 하드웨어에 쓰는(`write`) 과정으로 이루어진다.13

2. **Resource Manager (RM):**

   - **역할:** 물리적 하드웨어와 그 드라이버를 추상화하여 CM에게 제공하는 **자원 관리자**다.12 RM은 URDF에 명시된 하드웨어 컴포넌트 플러그인을 

     `pluginlib`를 사용해 동적으로 로드한다. 로드된 하드웨어는 '상태 인터페이스(State Interface, 예: `joint/position`, `joint/velocity`)'와 '명령 인터페이스(Command Interface, 예: `joint/position`, `joint/velocity`)'라는 자원을 RM에 등록한다.

   - **상호작용:** CM은 RM을 통해서만 하드웨어에 접근할 수 있다. 컨트롤러가 활성화될 때, CM은 해당 컨트롤러가 사용하려는 명령 인터페이스에 대한 '소유권(claim)'을 부여하여 다른 컨트롤러가 동시에 접근하는 것을 막는다. 이는 자원 충돌을 방지하고 시스템의 결정성을 보장하는 중요한 메커니즘이다.11

3. **Hardware Interface (Component):**

   - **역할:** 실제 물리적 하드웨어(모터, 엔코더, 센서 등)와의 통신을 담당하는 **드라이버 플러그인**이다.2 개발자는 자신의 하드웨어에 맞는 통신 프로토콜(예: 시리얼, CAN, EtherCAT)을 사용하여 이 컴포넌트를 C++ 클래스로 구현해야 한다.
   - **종류:** 하드웨어의 특성에 따라 세 가지 기본 타입으로 나뉜다.12
     - `SystemInterface`: 로봇 팔처럼 여러 조인트와 센서를 포함하는 복합 시스템. 읽기와 쓰기가 모두 가능하다.
     - `ActuatorInterface`: 단일 모터나 밸브처럼 1-DOF 액추에이터. 읽기와 쓰기가 가능하다.
     - `SensorInterface`: 힘/토크 센서나 IMU처럼 상태만 읽어오는 센서. 읽기만 가능하다.

4. **Controller:**

   - **역할:** 제어 이론을 실제 코드로 구현한 **알고리즘 플러그인**이다.12 컨트롤러는 

     `Hardware Interface`가 제공하는 상태 인터페이스를 통해 현재 로봇의 상태를 읽고, 제어 법칙(예: PID 제어, 궤적 보간)에 따라 계산된 명령을 명령 인터페이스에 쓴다.13

   - **표준 컨트롤러:** `ros2_controllers` 패키지에는 `joint_trajectory_controller`(궤적 추종), `diff_drive_controller`(차동 구동), `forward_command_controller`(직접 명령 전달) 등 널리 사용되는 다양한 표준 컨트롤러가 미리 구현되어 있어, 대부분의 경우 직접 컨트롤러를 작성할 필요 없이 가져다 쓸 수 있다.8


`ros2_control`의 또 다른 강력함은 로봇의 제어 시스템을 코드가 아닌, 선언적인 설정 파일을 통해 정의한다는 점이다. 이는 시스템의 구성을 명확하게 하고, 재설정을 용이하게 만든다.15

- URDF (<ros2_control> 태그):

  URDF(Unified Robot Description Format)는 본래 로봇의 물리적 형상(링크, 조인트, 시각/충돌 모델)을 기술하기 위한 XML 형식이다. ros2_control은 여기에 <ros2_control>이라는 커스텀 태그를 추가하여 제어 관점에서의 하드웨어 구성을 정의한다.2 이 태그 안에는 다음 정보가 포함된다.

  - `<plugin>`: 사용할 `Hardware Interface` 플러그인의 이름 (예: `ros2_control_demo_example_1/RRBotSystemPositionOnlyHardware`).
  - `<joint>`: 제어할 각 조인트의 이름과, 해당 조인트가 제공하는 `command_interface` 및 `state_interface`의 종류 (예: `position`, `velocity`, `effort`).
  - `<param>`: 하드웨어 인터페이스에 전달할 파라미터 (예: 시리얼 포트 경로, 통신 속도).

- YAML (컨트롤러 설정 파일):

  YAML 파일은 Controller Manager가 어떤 컨트롤러들을 어떻게 설정할지를 정의한다.14

  - `controller_manager`: CM 자체의 파라미터(예: `update_rate`)를 설정한다.
  - **컨트롤러 목록:** 로드할 각 컨트롤러의 이름을 키로 하여 설정을 정의한다.
    - `type`: 사용할 컨트롤러 플러그인의 타입 (예: `joint_state_broadcaster/JointStateBroadcaster`).
    - `joints`: 해당 컨트롤러가 제어할 조인트들의 목록.
    - `interface_name`: 컨트롤러가 사용할 명령 인터페이스의 이름.
    - 기타 컨트롤러별 파라미터 (예: PID 게인).

이 두 파일의 조합은 `ros2_control` 시스템의 전체 구성을 결정한다. URDF가 하드웨어의 '능력'을 정의한다면, YAML은 그 능력을 활용할 '전략'을 정의하는 셈이다.

이러한 구조에서 URDF의 `<ros2_control>` 태그는 단순한 설정 정보 이상의 의미를 갖는다. 이것은 하드웨어 계층과 제어 계층 사이의 **API 계약(Contract)** 과 같다. `Controller Manager`는 이 "계약서"를 파싱하여 하드웨어가 어떤 '엔드포인트'(command/state interfaces)를 노출하는지 파악하고, 컨트롤러의 요구사항과 매칭시킨다. 이로 인해 `Controller Manager`는 하드웨어의 실제 C++ 구현을 전혀 몰라도 시스템을 구성하고 운영할 수 있다.

이 과정은 다음과 같이 진행된다.

1. 사용자는 URDF에 `<plugin>my_robot_driver/MyRobotHardware</plugin>`와 `<joint name="wheel_joint"><command_interface name="velocity"/></joint>`를 명시한다.2
2. `ros2_control_node`가 시작될 때, `Controller Manager`는 `robot_description` 파라미터로 이 URDF를 읽는다.6
3. 내부적으로 `Component Parser`를 사용해 `<ros2_control>` 태그를 분석하여, 'MyRobotHardware' 플러그인을 로드해야 하며, 이 플러그인이 'wheel_joint'에 대한 'velocity' 명령 인터페이스를 제공해야 한다는 사실을 인지한다.30
4. 사용자가 YAML 파일에 정의된 `diff_drive_controller`를 로드하면, 이 컨트롤러는 'velocity' 명령 인터페이스를 요구한다.
5. `Controller Manager`는 URDF 계약서에 명시된 자원과 컨트롤러의 요구사항이 일치함을 확인하고 둘을 연결한다.

결론적으로, 하드웨어 개발자는 URDF에 명시된 '계약'만 준수하여 `Hardware Interface`를 구현하면 되고, 제어 시스템 사용자는 이 계약서를 보고 어떤 컨트롤러를 사용할지 결정할 수 있다. 이것이 `ros2_control`의 강력한 모듈성과 Sim-to-Real 전환 능력의 핵심 비결이다.



`ros2_control`의 개념과 작동 방식을 가장 효과적으로 학습하는 방법은 공식 데모 예제를 분석하고 실행해보는 것이다. `ros-controls` 그룹은 `ros2_control_demos`라는 GitHub 저장소를 통해 다양한 사용 사례를 보여주는 예제 패키지들을 제공한다.8 이 저장소에는 주로 두 종류의 가상 로봇이 사용된다.

- **RRBot:** "Revolute-Revolute Manipulator Robot"의 약자로, 2개의 회전 조인트(Revolute Joint)로 구성된 간단한 2축 로봇 팔이다. 조인트 제어와 관련된 다양한 개념을 시연하는 데 사용된다.10
- **DiffBot:** "Differential Mobile Robot"의 약자로, 차동 구동 방식의 이동 로봇이다. 이동 로봇 제어와 관련된 개념을 보여주는 데 사용된다.31

이 보고서에서는 `ros2_control`의 가장 기본적인 구조를 이해하기 위해 `ros2_control_demos`의 `example_1`, 즉 가장 기본적인 RRBot 예제를 집중적으로 분석한다.31 이 예제는 하나의 하드웨어 인터페이스를 통해 위치 제어를 수행하고, 컨트롤러를 동적으로 전환하는 방법을 보여준다.31


RRBot 시뮬레이션을 구동하는 데 필요한 파일들은 크게 URDF, Controller YAML, Launch 파일 세 가지로 나뉜다. 이 파일들이 어떻게 유기적으로 연결되어 하나의 제어 시스템을 구성하는지 단계별로 분석한다.


URDF 파일은 로봇의 구조를 정의한다. XACRO(`XML Macros`)를 사용하여 재사용성과 가독성을 높이는 것이 일반적이다.28

- **`rrbot.urdf.xacro` (메인 URDF 파일):**
  - 이 파일은 RRBot의 전체적인 구조를 정의한다. `link` 태그를 사용하여 로봇의 각 부분(base_link, link1, link2)의 물리적 속성(형상, 질량 등)을 정의한다.13
  - `joint` 태그를 사용하여 링크들을 어떻게 연결할지(기구학적 구조) 정의한다. RRBot은 `joint1`과 `joint2`라는 두 개의 `revolute`(회전) 조인트를 가진다.13
  - 가장 중요한 부분은 `<xacro:include filename="$(find ros2_control_demo_example_1)/description/ros2_control/rrbot.ros2_control.xacro" />`와 `<xacro:rrbot_ros2_control name="RRBotSystemPositionOnly" prefix=""/>` 라인이다. 이는 별도의 xacro 파일에 정의된 `ros2_control` 설정을 현재 URDF에 포함시키고, `rrbot_ros2_control` 매크로를 호출하여 실제 설정을 생성하라는 의미다.17
- **`rrbot.ros2_control.xacro` (`ros2_control` 설정 파일):**
  - 이 파일은 RRBot의 제어 시스템 구성을 집중적으로 정의한다. `<ros2_control>` 태그가 그 핵심이다.29
  - `<ros2_control name="RRBotSystemPositionOnly" type="system">`: 제어 시스템의 이름과 타입을 `system`으로 지정한다.
  - `<hardware>`: 하드웨어 인터페이스 플러그인을 정의하는 부분이다.
    - `<plugin>ros2_control_demo_example_1/RRBotSystemPositionOnlyHardware</plugin>`: 이 예제에서는 `RRBotSystemPositionOnlyHardware`라는 커스텀 하드웨어 인터페이스를 사용하도록 명시되어 있다. 이 플러그인은 실제 하드웨어 없이 조인트의 상태를 시뮬레이션하는 역할을 한다. 만약 Gazebo와 같은 시뮬레이터를 사용한다면 이 부분은 `gazebo_ros2_control/GazeboSystem`으로 변경될 것이다.29
  - `<joint name="joint1">` 및 `<joint name="joint2">`: 각 조인트가 제공하는 인터페이스를 정의한다.
    - `<command_interface name="position">`: `joint1`과 `joint2` 모두 `position` 명령을 받을 수 있음을 의미한다. 컨트롤러는 이 인터페이스를 통해 목표 위치를 전달할 수 있다.
    - `<state_interface name="position"/>`: `joint1`과 `joint2` 모두 현재 `position` 상태를 제공할 수 있음을 의미한다. 컨트롤러는 이 인터페이스를 통해 현재 조인트 각도를 읽을 수 있다.29


이 YAML 파일은 `Controller Manager`가 어떤 컨트롤러를 로드하고 어떻게 설정할지를 정의한다.17

```YAML
controller_manager:
  ros__parameters:
    update_rate: 100  # Hz

joint_state_broadcaster:
  type: joint_state_broadcaster/JointStateBroadcaster

forward_position_controller:
  type: forward_command_controller/ForwardCommandController
  joints:
    - joint1
    - joint2
  interface_name: position
```

33

- `controller_manager`:
  - `update_rate: 100`: 컨트롤러 매니저의 `read-update-write` 루프가 초당 100번 실행되도록 설정한다. 제어 주기가 10ms임을 의미한다.33
- `joint_state_broadcaster`:
  - `type: joint_state_broadcaster/JointStateBroadcaster`: 이 컨트롤러는 `ros2_control`의 필수 요소 중 하나다. 하드웨어 인터페이스로부터 모든 조인트의 상태(URDF에 정의된 `state_interface`)를 읽어와 `/joint_states`와 `/dynamic_joint_states`라는 표준 ROS 토픽으로 발행한다. `robot_state_publisher` 노드는 이 정보를 사용하여 로봇의 TF를 계산하고, RViz는 이를 이용해 로봇 모델을 시각화한다.17
- `forward_position_controller`:
  - `type: forward_command_controller/ForwardCommandController`: 이 컨트롤러는 수신한 명령을 아무런 처리 없이 그대로 지정된 명령 인터페이스로 전달(forwarding)하는 단순한 컨트롤러다.17
  - `joints`: 이 컨트롤러가 `joint1`과 `joint2`를 제어할 것임을 명시한다.
  - `interface_name: position`: 제어 명령을 `position` 명령 인터페이스로 전달하도록 설정한다. 즉, 이 컨트롤러는 `/forward_position_controller/commands` 토픽으로 `Float64MultiArray` 메시지를 받으면, 그 값을 `joint1`과 `joint2`의 목표 위치로 설정한다.33


Python으로 작성된 이 런치 파일은 위에서 분석한 URDF와 YAML 파일을 조합하여 실제 ROS2 노드들을 실행시키는 역할을 한다.31

```Python
# rrbot.launch.py의 핵심 구조
import os
from launch import LaunchDescription
from launch_ros.actions import Node
from launch_ros.substitutions import FindPackageShare
from launch.substitutions import Command
import xacro

def generate_launch_description():
    # 1. URDF 로드 및 파싱
    pkg_share = FindPackageShare(package='ros2_control_demo_example_1').find('ros2_control_demo_example_1')
    urdf_file_path = os.path.join(pkg_share, 'description/urdf/rrbot.urdf.xacro')
    robot_description_config = xacro.process_file(urdf_file_path)
    robot_description = {'robot_description': robot_description_config.toxml()}

    # 2. robot_state_publisher 노드 생성
    robot_state_publisher_node = Node(
        package='robot_state_publisher',
        executable='robot_state_publisher',
        output='screen',
        parameters=[robot_description]
    )

    # 3. ros2_control_node (Controller Manager) 생성
    controllers_yaml_path = os.path.join(pkg_share, 'bringup/config/rrbot_controllers.yaml')
    ros2_control_node = Node(
        package='controller_manager',
        executable='ros2_control_node',
        parameters=[robot_description, controllers_yaml_path],
        output='screen',
    )

    # 4. Controller Spawner (joint_state_broadcaster)
    joint_state_broadcaster_spawner = Node(
        package="controller_manager",
        executable="spawner",
        arguments=["joint_state_broadcaster", "--controller-manager", "/controller_manager"],
    )

    # 5. Controller Spawner (forward_position_controller)
    forward_position_controller_spawner = Node(
        package="controller_manager",
        executable="spawner",
        arguments=["forward_position_controller", "--controller-manager", "/controller_manager"],
    )

    return LaunchDescription([
        robot_state_publisher_node,
        ros2_control_node,
        joint_state_broadcaster_spawner,
        forward_position_controller_spawner
    ])
```

1. **URDF 로드 및 파싱:** `xacro` 라이브러리를 사용해 `rrbot.urdf.xacro` 파일을 처리하고, 그 결과를 XML 문자열로 변환하여 `robot_description`이라는 파라미터로 준비한다.18

2. **`robot_state_publisher` 실행:** 이 노드는 `robot_description` 파라미터와 `/joint_states` 토픽을 구독하여 로봇의 각 링크에 대한 TF(Transform) 정보를 계산하고 `/tf` 토픽으로 발행한다. RViz가 로봇 모델을 올바른 자세로 표시하기 위해 필수적이다.6

3. **`ros2_control_node` 실행:** 이것이 바로 `Controller Manager`의 실행 파일이다. `robot_description` 파라미터를 통해 로봇의 하드웨어 구성을 파악하고, 컨트롤러 YAML 파일 경로를 파라미터로 받아 어떤 컨트롤러를 관리할지 결정한다.6

4. **Controller Spawner 실행:** `spawner`는 `controller_manager`가 제공하는 유틸리티 스크립트다.16 지정된 컨트롤러(여기서는 

   `joint_state_broadcaster`와 `forward_position_controller`)를 `controller_manager`에 로드하고 활성화(configure and activate)시키는 역할을 한다. `--controller-manager` 인자를 통해 어떤 `controller_manager`에 명령을 내릴지 지정한다.18


이제 모든 구성 요소가 준비되었으므로, 시스템을 실행하고 정상적으로 동작하는지 확인한다.

1. **데모 실행:** `ros2_control_demos` 패키지를 빌드하고 워크스페이스를 소싱한 후, 다음 명령으로 런치 파일을 실행한다. RViz 창이 함께 실행되어 RRBot 모델이 표시될 것이다.

   ```Bash
   ros2 launch ros2_control_demo_example_1 rrbot.launch.py
   ```

   32

2. **시스템 상태 확인:** 새 터미널을 열고 `ros2 control` CLI를 사용하여 시스템의 내부 상태를 확인한다.

   - **컨트롤러 목록 확인:**

     ```Bash
     ros2 control list_controllers
     ```

     출력으로 `joint_state_broadcaster`와 `forward_position_controller`가 `active` 상태로 표시되어야 한다.38

   - **하드웨어 인터페이스 확인:**

     ```Bash
     ros2 control list_hardware_interfaces
     ```

     출력으로 `joint1/position`과 `joint2/position`에 대한 `command` 및 `state` 인터페이스가 표시된다. `command` 인터페이스는 `[available][claimed]` 상태여야 하는데, 이는 인터페이스가 사용 가능하며 `forward_position_controller`에 의해 소유권이 주장되었음을 의미한다.37

3. **로봇 구동:** `ros2 topic pub` 명령을 사용하여 `forward_position_controller`에 직접 목표 위치 명령을 발행한다.

   ```Bash
   ros2 topic pub /forward_position_controller/commands std_msgs/msg/Float64MultiArray "data: [1.0, 0.5]"
   ```

   이 명령을 실행하면 RViz에서 RRBot의 `joint1`이 1.0 라디안, `joint2`가 0.5 라디안 위치로 움직이는 것을 확인할 수 있다.32

4. **컨트롤러 동적 전환:** `ros2_control`의 강력한 기능 중 하나는 실행 중에 컨트롤러를 교체하는 것이다. `forward_position_controller`를 비활성화하고, 궤적 추종 컨트롤러인 `joint_trajectory_controller`를 활성화해보자.

   ```Bash
   # 1. JTC 로드 (비활성 상태로)
   ros2 control load_controller joint_trajectory_controller --set-state inactive
   
   # 2. 컨트롤러 전환 (JTC 활성화, FPC 비활성화)
   ros2 control switch_controllers --activate joint_trajectory_controller --deactivate forward_position_controller
   ```

   17

   이제 joint_trajectory_controller가 활성화되었으므로, /joint_trajectory_controller/joint_trajectory 토픽에 trajectory_msgs/msg/JointTrajectory 메시지를 보내 더 복잡한 궤적 추종 제어를 수행할 수 있다.



시뮬레이션에서 벗어나 실제 로봇을 `ros2_control`로 제어하기 위한 가장 중요하고 도전적인 단계는 바로 커스텀 **하드웨어 인터페이스(Hardware Interface)** C++ 클래스를 작성하는 것이다.13 이 클래스는 `ros2_control` 프레임워크와 실제 물리적 하드웨어(모터 드라이버, 엔코더 등) 사이의 다리 역할을 한다.


하드웨어 인터페이스는 `pluginlib`를 통해 동적으로 로드되는 라이브러리이므로, 특정 인터페이스 클래스를 상속받아 구현해야 한다. 가장 일반적으로 사용되는 것은 여러 조인트와 센서를 포함하는 시스템을 위한 `hardware_interface::SystemInterface`이다.13

```C++
// MyRobotHardware.hpp
#include "hardware_interface/system_interface.hpp"
#include "rclcpp_lifecycle/node_interfaces/lifecycle_node_interface.hpp"
//... 기타 필요한 헤더...

namespace my_robot_driver
{
class MyRobotHardware : public hardware_interface::SystemInterface
{
public:
  // Lifecycle methods
  rclcpp_lifecycle::node_interfaces::LifecycleNodeInterface::CallbackReturn on_init(
    const hardware_interface::HardwareInfo & info) override;
  rclcpp_lifecycle::node_interfaces::LifecycleNodeInterface::CallbackReturn on_configure(
    const rclcpp_lifecycle::State & previous_state) override;
  rclcpp_lifecycle::node_interfaces::LifecycleNodeInterface::CallbackReturn on_cleanup(
    const rclcpp_lifecycle::State & previous_state) override;
  rclcpp_lifecycle::node_interfaces::LifecycleNodeInterface::CallbackReturn on_activate(
    const rclcpp_lifecycle::State & previous_state) override;
  rclcpp_lifecycle::node_interfaces::LifecycleNodeInterface::CallbackReturn on_deactivate(
    const rclcpp_lifecycle::State & previous_state) override;

  // Data I/O methods
  std::vector<hardware_interface::StateInterface> export_state_interfaces() override;
  std::vector<hardware_interface::CommandInterface> export_command_interfaces() override;
  hardware_interface::return_type read(
    const rclcpp::Time & time, const rclcpp::Duration & period) override;
  hardware_interface::return_type write(
    const rclcpp::Time & time, const rclcpp::Duration & period) override;

private:
  // Store commands and states from the hardware
  std::vector<double> hw_commands_;
  std::vector<double> hw_positions_;
  std::vector<double> hw_velocities_;
  //... 하드웨어 통신을 위한 변수들 (예: 시리얼 포트 객체)...
};
}  // namespace my_robot_driver
```

39


하드웨어 인터페이스는 `LifecycleNode`와 유사한 생명주기 상태(Unconfigured, Inactive, Active)를 가지며, 각 상태 전환 시 특정 콜백 함수가 호출된다.13

- `on_init()`: 플러그인이 로드될 때 한 번 호출된다. `HardwareInfo` 객체를 통해 URDF에 기술된 정보를 파싱하고, 필요한 메모리를 할당하며, 하드웨어 파라미터(예: 조인트 이름, 핀 번호)를 읽어온다.13

- `on_configure()`: 하드웨어를 사용할 준비를 한다. 예를 들어, 시리얼 포트를 열고 통신 설정을 하거나, GPIO 핀을 초기화하는 작업을 수행한다.41 성공하면 

  `Inactive` 상태가 된다.

- `on_activate()`: 실제 하드웨어 구동을 시작한다. 예를 들어, 모터 드라이버의 전원을 인가하거나 브레이크를 해제한다. 성공하면 `Active` 상태가 되어 `read()`와 `write()`가 주기적으로 호출되기 시작한다.41

- `on_deactivate()`: 하드웨어 구동을 안전하게 중지한다. `Active` 상태에서 `Inactive` 상태로 돌아간다.

- `on_cleanup()`: `on_configure`에서 설정한 자원들을 해제한다.

- `on_error()`: `read` 또는 `write` 중 에러 발생 시 호출되어 시스템을 안전한 상태로 전환한다.


`Active` 상태에서 `Controller Manager`의 실시간 루프는 `read()`와 `write()` 함수를 주기적으로 호출한다.

- `export_state_interfaces()`: 이 하드웨어 인터페이스가 프레임워크에 제공할 수 있는 모든 **상태 인터페이스**를 정의하고 반환한다. 예를 들어, `joint1`의 위치와 속도 상태를 제공하고 싶다면, `hw_positions_`와 `hw_velocities_` 변수를 각각 `hardware_interface::HW_IF_POSITION`과 `hardware_interface::HW_IF_VELOCITY` 타입의 `StateInterface`로 묶어 반환한다. `ros2_control`은 이 정보를 통해 컨트롤러가 어떤 상태를 읽을 수 있는지 알게 된다.13
- `export_command_interfaces()`: 이 하드웨어 인터페이스가 외부(컨트롤러)로부터 받을 수 있는 모든 **명령 인터페이스**를 정의하고 반환한다. 예를 들어, `joint1`에 대한 속도 명령을 받고 싶다면, `hw_commands_` 변수를 `hardware_interface::HW_IF_VELOCITY` 타입의 `CommandInterface`로 묶어 반환한다. 컨트롤러는 이 인터페이스에 목표 속도 값을 쓰게 된다.13
- `read()`: `Controller Manager`의 업데이트 루프 초반에 호출된다. 이 함수 안에서는 실제 물리적 하드웨어(예: 엔코더)로부터 현재 상태 값을 읽어와 `export_state_interfaces`에서 등록했던 멤버 변수들(`hw_positions_`, `hw_velocities_` 등)을 최신 값으로 업데이트해야 한다.13
- `write()`: `Controller Manager`의 업데이트 루프 후반에 호출된다. 이 함수 안에서는 `export_command_interfaces`에서 등록했던 멤버 변수(`hw_commands_`)에 컨트롤러가 써 놓은 최신 명령 값을 읽어, 이를 실제 하드웨어에 전달하는 코드(예: 모터 드라이버에 PWM 신호 전송)를 구현해야 한다.13

다음 표는 하드웨어 인터페이스 구현 시 반드시 재정의해야 하는 핵심 함수들의 역할을 요약한 것이다.

**Table 2: Hardware Interface 필수 구현 함수 요약 및 역할**

| 함수명 (Function)           | 반환 타입 (Return Type)    | 호출 시점 (Timing)                   | 주요 역할 (Primary Role)                                     |
| --------------------------- | -------------------------- | ------------------------------------ | ------------------------------------------------------------ |
| `on_init`                   | `CallbackReturn`           | 플러그인 로드 시 1회                 | URDF 정보 파싱, 멤버 변수 초기화, 메모리 할당.13             |
| `on_configure`              | `CallbackReturn`           | `Unconfigured` -> `Inactive` 전환 시 | 하드웨어 통신 초기화 (시리얼 포트 열기, GPIO 설정 등).41     |
| `on_cleanup`                | `CallbackReturn`           | `Inactive` -> `Unconfigured` 전환 시 | `on_configure`에서 할당한 자원 해제.                         |
| `on_activate`               | `CallbackReturn`           | `Inactive` -> `Active` 전환 시       | 하드웨어 구동 시작 (모터 전원 인가, 브레이크 해제 등).41     |
| `on_deactivate`             | `CallbackReturn`           | `Active` -> `Inactive` 전환 시       | 하드웨어 구동 안전 정지.                                     |
| `on_error`                  | `CallbackReturn`           | `read`/`write` 에러 발생 시          | 시스템을 안전 상태(주로 `Unconfigured`)로 전환.41            |
| `export_state_interfaces`   | `vector<StateInterface>`   | `on_configure` 직후                  | 제공 가능한 상태 인터페이스 목록(예: 현재 위치, 속도)을 프레임워크에 등록.13 |
| `export_command_interfaces` | `vector<CommandInterface>` | `on_configure` 직후                  | 수신 가능한 명령 인터페이스 목록(예: 목표 위치, 속도)을 프레임워크에 등록.13 |
| `read`                      | `return_type`              | `Active` 상태에서 매 업데이트 루프   | 실제 하드웨어에서 상태를 읽어와 상태 변수(`hw_positions_` 등) 업데이트.13 |
| `write`                     | `return_type`              | `Active` 상태에서 매 업데이트 루프   | 명령 변수(`hw_commands_`) 값을 읽어 실제 하드웨어에 명령 전달.13 |


이제 라즈베리파이 4, L298N 모터 드라이버, 그리고 엔코더가 부착된 2개의 DC 모터를 사용하여 차동 구동 로봇을 `ros2_control`로 제어하는 시나리오를 구체화해본다.42


- **제어 보드:** 라즈베리파이 4 (Ubuntu 22.04 + ROS2 Humble).24
- **모터 드라이버:** L298N. 각 모터당 2개의 방향 제어 핀과 1개의 PWM 속도 제어 핀이 필요하다.
- **액추에이터:** 엔코더가 부착된 DC 모터 2개. 엔코더는 A, B상 출력을 가진다.


이 시나리오에 맞는 C++ 하드웨어 인터페이스 클래스 `DiffDrivePiHardware`의 핵심 구현부를 개념적으로 살펴보자. 실제 구현에는 `pigpio` C 라이브러리 등을 링크하여 사용해야 한다.

```C++
// DiffDrivePiHardware.cpp (핵심 부분)

#include "pigpio.h" // pigpio 라이브러리 사용 가정

//... (클래스 선언 및 기타 생명주기 함수)...

CallbackReturn DiffDrivePiHardware::on_init(const hardware_interface::HardwareInfo & info)
{
  // URDF로부터 조인트 이름, 핀 번호 등 파라미터 읽기
  // 예: info.hardware_parameters["left_wheel_pwm_pin"]
  //...
  // hw_commands_, hw_positions_, hw_velocities_ 벡터 크기 초기화
  //...
  return CallbackReturn::SUCCESS;
}

CallbackReturn DiffDrivePiHardware::on_configure(const rclcpp_lifecycle::State &)
{
  // pigpio 라이브러리 초기화
  if (gpioInitialise() < 0) { return CallbackReturn::ERROR; }

  // PWM, 방향, 엔코더 핀 모드 설정
  // 예: gpioSetMode(left_pwm_pin_, PI_OUTPUT);
  // 예: gpioSetPWMfrequency(left_pwm_pin_, 1000); // 1kHz
  // 예: gpioSetMode(left_encoder_a_pin_, PI_INPUT);
  //...
  return CallbackReturn::SUCCESS;
}

CallbackReturn DiffDrivePiHardware::on_activate(const rclcpp_lifecycle::State &)
{
  // 모터 정지 상태로 시작
  // 예: gpioPWM(left_pwm_pin_, 0);
  //...
  // 엔코더 값 초기화
  //...
  return CallbackReturn::SUCCESS;
}

hardware_interface::return_type DiffDrivePiHardware::read(const rclcpp::Time &, const rclcpp::Duration &)
{
  // 엔코더 핀 상태를 읽어 틱(tick) 수 계산 (실제로는 인터럽트 기반이 더 효율적)
  //...
  
  // 틱 수를 라디안 단위의 위치(position)와 속도(velocity)로 변환
  // double delta_ticks = current_ticks - last_ticks;
  // hw_positions_[i] += delta_ticks_to_rad(delta_ticks);
  // hw_velocities_[i] = delta_ticks_to_rad(delta_ticks) / period.seconds();
  //...
  
  return hardware_interface::return_type::OK;
}

hardware_interface::return_type DiffDrivePiHardware::write(const rclcpp::Time &, const rclcpp::Duration &)
{
  // 컨트롤러가 써 놓은 목표 속도(hw_commands_)를 읽음
  double left_target_vel = hw_commands_;
  double right_target_vel = hw_commands_;

  // 목표 속도를 PWM 듀티 사이클과 방향 신호로 변환
  // 방향 설정
  // gpioWrite(left_dir_pin1, left_target_vel > 0? 1 : 0);
  // gpioWrite(left_dir_pin2, left_target_vel > 0? 0 : 1);
  
  // 속도 설정 (0-255 범위의 PWM 값으로 변환)
  // int left_pwm_value = static_cast<int>(std::abs(left_target_vel) * pwm_scale_factor);
  // gpioPWM(left_pwm_pin_, left_pwm_value);
  //...

  return hardware_interface::return_type::OK;
}
```

42


1. **URDF (`\*.ros2_control.xacro`):**
   - `<plugin>` 태그를 새로 작성한 하드웨어 인터페이스 플러그인 이름으로 변경한다 (예: `diff_drive_pi_driver/DiffDrivePiHardware`).
   - `<param>` 태그를 사용하여 라즈베리파이의 GPIO 핀 번호, 엔코더의 PPR(Pulses Per Revolution) 등 하드웨어 관련 정보를 플러그인에 전달한다.27
2. **YAML (`controllers.yaml`):**
   - `forward_command_controller` 대신 `diff_drive_controller/DiffDriveController`를 사용하도록 변경한다.
   - `diff_drive_controller`에 필요한 파라미터(예: `left_wheel_names`, `right_wheel_names`, `wheel_separation`, `wheel_radius`)를 설정한다.2

이 과정을 통해, 상위 레벨에서는 `diff_drive_controller`가 발행하는 `/cmd_vel_unstamped` 토픽에 `Twist` 메시지를 보내는 것만으로 로봇을 제어할 수 있게 된다. 실제 하드웨어와의 복잡한 통신은 `DiffDrivePiHardware` 인터페이스가 모두 처리하므로, 제어 로직과 하드웨어 구현이 완벽하게 분리된다.



컨트롤러 체이닝은 `ros2_control`의 고급 기능으로, 한 컨트롤러의 출력을 다른 컨트롤러의 입력으로 연결하여 더 복잡하고 계층적인 제어 구조를 만드는 기법이다.31 이는 마치 유닉스 파이프라인처럼 컨트롤러들을 연결하는 것과 같다.

예를 들어, `ros2_control_demos/example_12`에서는 `passthrough_controller`를 사용하여 컨트롤러 체인을 시연한다.38 이 컨트롤러는 자신의 입력으로 받은 값을 그대로 출력으로 내보내는 동시에, 자신의 명령 인터페이스를 다른 컨트롤러가 사용할 수 있도록 '참조 인터페이스(reference interface)'로 노출한다.

가상 시나리오를 생각해보자.

1. **PID 컨트롤러:** 목표 위치와 현재 위치(상태 인터페이스)의 오차를 기반으로 필요한 제어 노력(effort)을 계산한다.
2. **Effort-to-PWM 컨트롤러 (가상):** PID 컨트롤러가 계산한 노력을 입력으로 받아, 이를 모터 드라이버가 이해할 수 있는 PWM 듀티 사이클 값으로 변환한다.
3. **하드웨어 인터페이스:** Effort-to-PWM 컨트롤러가 계산한 PWM 값을 받아 실제 하드웨어에 쓴다.

이 구조에서 PID 컨트롤러는 실제 하드웨어가 PWM으로 제어된다는 사실을 전혀 알 필요가 없다. 이는 제어 로직을 더욱 세분화하고 재사용성을 높이는 강력한 방법이다.


`ros2_control`은 실시간 제어를 '지원'하지만, 최상의 성능을 내기 위해서는 운영체제 수준의 설정이 동반되어야 한다.

- **Real-time Kernel:** 표준 리눅스 커널은 실시간성을 보장하지 않는다. 예측 가능한 스케줄링을 위해 `PREEMPT_RT` 패치가 적용된 실시간 커널을 사용하는 것이 권장된다. Ubuntu에서는 `linux-image-rt-amd64`와 같은 패키지를 설치하여 적용할 수 있다.16
- **메모리 락 및 스레드 우선순위:** `Controller Manager`의 메인 스레드는 `SCHED_FIFO`라는 실시간 스케줄링 정책과 높은 우선순위(기본값 50)를 사용하려고 시도한다. 이를 허용하려면 시스템에 적절한 권한 설정이 필요하다. 또한, 실시간 스레드가 페이지 폴트(page fault)로 인해 지연되는 것을 막기 위해 `mlockall`을 사용하여 프로세스의 메모리를 물리 RAM에 고정(lock)해야 한다.16

이러한 설정들은 제어 루프의 지터(jitter, 실행 주기의 변동성)를 최소화하여, 제어 시스템의 안정성과 성능을 극대화하는 데 필수적이다.

`ros2_control` 컨트롤러 내부에는 사실상 두 개의 '세계'가 공존한다. 하나는 ROS 이벤트(예: 토픽 수신)에 의해 비결정적으로 호출되는 **비실시간(Non-Real-time) 콜백 컨텍스트**이고, 다른 하나는 `Controller Manager`에 의해 고정 주기로 엄격하게 실행되는 **실시간(Real-time) `update()` 컨텍스트**다.13 이 두 세계 간에 데이터를 안전하게 주고받는 것은 매우 중요한 문제다.

만약 토픽 구독 콜백에서 받은 목표 속도 값을 멤버 변수에 쓰고, `update()` 함수에서 이 값을 읽는다고 가정해보자. 두 스레드가 동시에 이 변수에 접근하면 데이터가 깨지는 레이스 컨디션이 발생할 수 있다. 이를 막기 위해 뮤텍스(mutex)를 사용하면, 실시간 `update()` 스레드가 비실시간 콜백 스레드를 기다리다 정해진 데드라인을 놓치는 '우선순위 역전(priority inversion)' 현상이 발생할 수 있다.45

이 문제를 해결하기 위해 `ros2_control` 생태계에서는 `realtime_tools::RealtimeBuffer`라는 특수한 도구를 제공한다.45

`RealtimeBuffer`는 잠금 없는(lock-free) 또는 최소한의 잠금을 사용하는 데이터 구조로, 두 세계를 잇는 안전한 '다리' 역할을 한다. 비실시간 콜백에서는 `writeFromNonRT()`를 호출하여 데이터를 버퍼에 밀어 넣고, 실시간 `update()`에서는 `readFromRT()`를 호출하여 가장 최신의 데이터를 안전하게 가져온다. 따라서, 안정적인 `ros2_control` 컨트롤러를 작성하는 능력은 이 '두 세계'의 존재를 인지하고, `RealtimeBuffer`를 통해 안전하게 데이터를 교환하는 패턴을 이해하는 데 달려있다. 이는 단순한 코딩 기법이 아니라, 실시간 시스템 설계의 핵심 원칙을 반영하는 것이다.


`ros2_control`은 제어 이론을 적용하기 위한 훌륭한 플랫폼이다.

- **DC 모터 모델링:** 액추에이터를 더 정밀하게 제어하려면 그 물리적 특성을 이해해야 한다. DC 모터의 정상 상태(steady-state) 전기적 모델은 다음과 같은 간단한 방정식으로 표현할 수 있다.
  $$
  V = I R + K_e \omega
  $$
  여기서 $`V`$는 모터에 가해지는 전압, $`I`$는 전류, $`R`$은 모터 내부 저항, $`K_e`$는 역기전력 상수, 그리고 $`\omega`$는 모터의 각속도이다. 이 모델은 전압 입력에 따른 모터의 속도 변화를 예측하는 데 사용될 수 있다.

- **PID 제어:** PID(Proportional-Integral-Derivative) 제어는 산업계에서 가장 널리 사용되는 피드백 제어 알고리즘이다. 목표값(setpoint)과 현재 측정값 사이의 오차(`e(t)`)를 기반으로 제어 입력을 계산한다. `ros2_control`에서는 `control_toolbox` 패키지를 통해 표준 PID 컨트롤러 구현을 제공한다.8 PID 제어 법칙은 다음과 같다.
  $$
  u(t) = K_p e(t) + K_i \int_0^t e(\tau)d\tau + K_d \frac{de(t)}{dt}
  $$
  여기서 $`u(t)`$는 제어 입력(예: 목표 전압), $`e(t)`$는 오차, ‘Kp‘, ‘Ki‘, $`K_d`$는 각각 비례, 적분, 미분 이득(gain) 파라미터다. 이 파라미터들을 적절히 튜닝하여 로봇의 응답 특성(빠르기, 안정성, 정상상태 오차)을 조절할 수 있다. `ros2_control` 환경에서는 이 PID 게인들을 YAML 파일을 통해 쉽게 설정하고 변경할 수 있다.


이 보고서는 ROS2 Humble에서 액추에이터를 제어하는 두 가지 주요 패러다임, 즉 **직접 통신 방식**과 **`ros2_control` 프레임워크**를 심층적으로 분석했다. 어떤 방식을 선택할지는 프로젝트의 목표와 상황에 따라 달라지며, 다음 가이드라인을 통해 현명한 결정을 내릴 수 있다.

- **빠른 프로토타이핑 및 학습 목적이라면 `직접 통신 방식`을 선택하라:**

  - **상황:** ROS2를 처음 배우거나, 아이디어를 빠르게 검증하고 싶을 때. 하드웨어가 단순하고 정밀한 실시간 제어가 중요하지 않을 때.

  - **이유:** 개념이 단순하고 최소한의 코드로 즉각적인 결과를 볼 수 있다.7

    `ros2_control`의 복잡한 설정 없이도 로봇을 움직여보는 경험을 할 수 있다.

  - **주의사항:** 이 방식은 '아키텍처 부채'를 쌓을 수 있음을 인지해야 한다. 프로젝트가 복잡해지면 결국 `ros2_control`로 전환해야 할 가능성이 높다.

- **재사용성, 확장성, 실시간 성능이 중요하다면 `ros2_control`을 선택하라:**

  - **상황:** 상용 제품 개발, 장기 연구 프로젝트, 여러 종류의 로봇을 다루거나, 시뮬레이션과 실제 로봇 간의 원활한 전환이 필요할 때. 정밀한 궤적 추종이나 힘 제어가 요구될 때.
  - **이유:** 하드웨어와 소프트웨어의 분리를 통해 코드의 재사용성과 모듈성을 극대화한다.1 표준화된 인터페이스는 MoveIt2, Nav2 등 ROS2 생태계의 다른 주요 패키지와의 통합을 용이하게 한다.18 실시간 제어 루프는 예측 가능하고 안정적인 성능을 보장한다.13
  - **접근법:** 초기 학습 곡선이 존재하지만, `ros2_control_demos` 예제를 통해 단계적으로 학습하는 것이 효과적이다. 먼저 시뮬레이션(RRBot, Gazebo)으로 프레임워크의 구조를 완전히 이해한 후, 자신의 하드웨어에 맞는 커스텀 하드웨어 인터페이스를 작성하는 순서로 접근하는 것이 좋다.10

궁극적으로, `ros2_control`은 현대 로보틱스 시스템이 요구하는 견고함과 유연성을 제공하는 산업 표준 프레임워크로 자리매김하고 있다. 단기적인 편의성보다는 장기적인 시스템의 건강성과 확장성을 고려한다면, `ros2_control`을 채택하고 그 구조를 깊이 이해하는 것이 모든 ROS2 개발자에게 필수적인 역량이 될 것이다.


1. ros2_control_explained - GitHub Pages, accessed July 27, 2025, https://masum919.github.io/ros2_control_explained/
2. ros2_control Concepts & Simulation - Articulated Robotics, accessed July 27, 2025, https://articulatedrobotics.xyz/tutorials/mobile-robot/applications/ros2_control-concepts/
3. ROS 2 Humble Installation - JARVIS DOCS - Goat Robotics, accessed July 27, 2025, [https://jarvis.goat-robotics.com/docs/Installation/ROS2%20installation/](https://jarvis.goat-robotics.com/docs/Installation/ROS2 installation/)
4. Understanding ROS 2 nodes with a simple Publisher - Subscriber pair, accessed July 27, 2025, https://ros2-industrial-workshop.readthedocs.io/en/latest/_source/basics/ROS2-Simple-Publisher-Subscriber.html
5. Writing a simple publisher and subscriber (Python) - ROS 2 ..., accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Py-Publisher-And-Subscriber.html
6. ROS 2 Control, Robot Control the Right Way | by Jiayi Hoffman ..., accessed July 27, 2025, https://medium.com/@jiayi.hoffman/ros-2-control-robot-control-the-right-way-d0e72e7f1b6c
7. Ros2 python or c++? : r/ROS - Reddit, accessed July 27, 2025, https://www.reddit.com/r/ROS/comments/1fvnioh/ros2_python_or_c/
8. Welcome to the ros2_control documentation! - ROS2_Control: Rolling Jul 2025 documentation, accessed July 27, 2025, https://control.ros.org/
9. Welcome to the ros2_control documentation - Humble!, accessed July 27, 2025, https://control.ros.org/humble/index.html
10. ROS2 Control Framework - Online Course | The Construct, accessed July 27, 2025, https://www.theconstruct.ai/robotigniteacademy_learnros/ros-courses-library/ros2-control/
11. Question Regarding ros2_control Design - ROS General - Open Robotics Discourse, accessed July 27, 2025, https://discourse.openrobotics.org/t/question-regarding-ros2-control-design/38751
12. Getting Started - ROS2_Control: Rolling Jul 2025 documentation, accessed July 27, 2025, https://control.ros.org/rolling/doc/getting_started/getting_started.html
13. Example 7: Full tutorial with a 6DOF robot - ROS2_Control, accessed July 27, 2025, https://control.ros.org/humble/doc/ros2_control_demos/example_7/doc/userdoc.html
14. Controllers - Universal Robots ROS 2 Driver Documentation 0.1 documentation, accessed July 27, 2025, https://docs.universal-robots.com/Universal_Robots_ROS2_Documentation/doc/ur_robot_driver/ur_robot_driver/doc/usage/controllers.html
15. ROS2 Jazzy Tutorial: Basics of ros2_control library - Aleksandar Haber, accessed July 27, 2025, https://aleksandarhaber.com/ros2-jazzy-tutorial-basics-of-ros2_control-library/
16. Controller Manager - ROS2_Control: Rolling Jul 2025 documentation, accessed July 27, 2025, https://control.ros.org/rolling/doc/ros2_control/controller_manager/doc/userdoc.html
17. Example 1: RRBot - ROS2_Control: Humble Jul 2025 documentation, accessed July 27, 2025, https://control.ros.org/humble/doc/ros2_control_demos/example_1/doc/userdoc.html
18. ros-controls/roscon2022_workshop - GitHub, accessed July 27, 2025, https://github.com/ros-controls/roscon2022_workshop
19. Getting Started - ROS2_Control: Galactic Jul 2025 documentation, accessed July 27, 2025, https://control.ros.org/galactic/doc/getting_started/getting_started.html
20. ROS 2 Humble subscriptions/std::bind function doesnt work [closed] - Stack Overflow, accessed July 27, 2025, https://stackoverflow.com/questions/78287351/ros-2-humble-subscriptions-stdbind-function-doesnt-work
21. Query: Is there a Feature in ROS2-Humble that enables publisher to publish message only when at least one subscriber connected at the other end - Robotics Stack Exchange, accessed July 27, 2025, https://robotics.stackexchange.com/questions/112381/query-is-there-a-feature-in-ros2-humble-that-enables-publisher-to-publish-messa
22. Writing a simple publisher and subscriber (C++) - ROS 2 Documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Cpp-Publisher-And-Subscriber.html
23. ROS 2 C++ Tutorial: Simple Publisher & Subscriber Explained! - YouTube, accessed July 27, 2025, https://www.youtube.com/watch?v=ce2L4Y4a4zQ
24. How to Install ROS 2 Humble on Raspberry Pi - YouTube, accessed July 27, 2025, https://www.youtube.com/watch?v=QBa-nTRWl7o
25. Raspberry Pi ROS Python Publisher: Publish a GPIO State - The ..., accessed July 27, 2025, https://roboticsbackend.com/raspberry-pi-ros-python-publisher-publish-a-gpio-state/
26. Getting Started - ROS2_Control: Humble Jul 2025 documentation, accessed July 27, 2025, https://control.ros.org/humble/doc/getting_started/getting_started.html
27. ros2_control hardware interface types, accessed July 27, 2025, https://control.ros.org/rolling/doc/ros2_control/hardware_interface/doc/hardware_interface_types_userdoc.html
28. Example 7: Full tutorial with a 6DOF robot - ROS2_Control, accessed July 27, 2025, https://control.ros.org/rolling/doc/ros2_control_demos/example_7/doc/userdoc.html
29. ros2_control_demos/example_1/description/ros2_control ... - GitHub, accessed July 27, 2025, https://github.com/ros-controls/ros2_control_demos/blob/master/example_1/description/ros2_control/rrbot.ros2_control.xacro
30. Reading a ros2_control xacro file in a regular node - Robotics Stack Exchange, accessed July 27, 2025, https://robotics.stackexchange.com/questions/108225/reading-a-ros2-control-xacro-file-in-a-regular-node
31. ros-controls/ros2_control_demos: This repository aims at providing examples to illustrate ros2_control and ros2_controllers - GitHub, accessed July 27, 2025, https://github.com/ros-controls/ros2_control_demos
32. Example 9: Simulation with RRBot - ROS2_Control: Humble Jul 2025 documentation, accessed July 27, 2025, https://control.ros.org/humble/doc/ros2_control_demos/example_9/doc/userdoc.html
33. ros2_control_demos/example_1/bringup/config/rrbot_controllers ..., accessed July 27, 2025, https://github.com/ros-controls/ros2_control_demos/blob/master/example_1/bringup/config/rrbot_controllers.yaml
34. Launch Files in MoveIt, accessed July 27, 2025, https://moveit.picknik.ai/main/doc/how_to_guides/moveit_launch_files/moveit_launch_files_tutorial.html
35. ros2_control_demos/example_1/bringup/launch/rrbot.launch.py at ..., accessed July 27, 2025, https://github.com/ros-controls/ros2_control_demos/blob/master/example_1/bringup/launch/rrbot.launch.py
36. ros2_control extra bits - Articulated Robotics, accessed July 27, 2025, https://articulatedrobotics.xyz/tutorials/mobile-robot/applications/ros2_control-extra/
37. Example 1: RRBot - ROS2_Control: Rolling Jul 2025 documentation, accessed July 27, 2025, https://control.ros.org/rolling/doc/ros2_control_demos/example_1/doc/userdoc.html
38. Example 12: Controller chaining with RRBot - ROS2_Control, accessed July 27, 2025, https://control.ros.org/humble/doc/ros2_control_demos/example_12/doc/userdoc.html
39. Writing a Hardware Component - ROS2_Control: Rolling Jul 2025 documentation, accessed July 27, 2025, https://control.ros.org/rolling/doc/ros2_control/hardware_interface/doc/writing_new_hardware_component.html
40. How to configure ROS2 Control Hardware interface for differential drive motors for AMR with Roboteq FAMILY controller? - Robotics Stack Exchange, accessed July 27, 2025, https://robotics.stackexchange.com/questions/107173/how-to-configure-ros2-control-hardware-interface-for-differential-drive-motors-f
41. Hardware Components - ROS2_Control: Rolling Jul 2025 documentation, accessed July 27, 2025, https://control.ros.org/rolling/doc/ros2_control/hardware_interface/doc/hardware_components_userdoc.html
42. Hardware interface for Raspberry pi 4 GPio - The Construct ROS Community, accessed July 27, 2025, https://get-help.theconstruct.ai/t/hardware-interface-for-raspberry-pi-4-gpio/25394
43. atticusrussell/ros2_rpi_pwm_hardware_interface: hardware interface for PWM servos through Raspberry Pi's GPIO pins compatible with ros2_control framework - GitHub, accessed July 27, 2025, https://github.com/atticusrussell/ros2_rpi_pwm_hardware_interface
44. ros2_control on the real robot - Articulated Robotics, accessed July 27, 2025, https://articulatedrobotics.xyz/tutorials/mobile-robot/applications/ros2_control-real/
45. ROS2 Control: How do controllers subscribe to topics? - Robotics Stack Exchange, accessed July 27, 2025, https://robotics.stackexchange.com/questions/115545/ros2-control-how-do-controllers-subscribe-to-topics
46. ROS2 Control: Custom GPIO controller's subscriber callbacks aren't triggering, accessed July 27, 2025, https://robotics.stackexchange.com/questions/115082/ros2-control-custom-gpio-controllers-subscriber-callbacks-arent-triggering

