# LiDAR SLAM을 위한 후처리 기반 동적 객체 제거 기법, Removert에 대한 심층 고찰



동시적 위치 추정 및 지도 작성(Simultaneous Localization and Mapping, SLAM) 기술, 특히 3D LiDAR(Light Detection And Ranging) 센서를 기반으로 하는 SLAM은 자율주행, 모바일 로봇, 도시 측량 등 다양한 분야에서 핵심적인 역할을 수행한다.1 이러한 LiDAR SLAM 알고리즘의 대다수는 '정적 세계 가정(Static World Assumption)'이라는 근본적인 전제 위에 구축되어 있다.3 이 가정은 센서가 관측하는 환경 내의 모든 객체가 움직이지 않고 고정되어 있다는 것을 의미하며, 알고리즘이 안정적으로 자신의 위치를 추정하고 일관된 지도를 생성하기 위한 수학적 기반이 된다.

그러나 이 가정은 현실 세계, 특히 차량과 보행자가 끊임없이 움직이는 도심 환경에서는 쉽게 위배된다.5 동적 객체(dynamic objects)의 존재는 SLAM 시스템에 심각한 문제를 야기한다. 움직이는 객체에서 반사된 LiDAR 포인트들은 시간의 흐름에 따라 다른 위치에 기록되면서, 최종적으로 구축된 지도에 마치 유령과 같은 잔상(ghost trail)이나 길게 늘어진 흔적을 남기게 된다.7 이러한 아티팩트는 지도의 품질을 심각하게 저하시킬 뿐만 아니라, 이를 기반으로 수행되는 후속 작업, 즉 로봇의 정확한 위치를 찾는 위치 추정(localization)이나 안전한 경로를 계획하는 경로 계획(path planning)의 성능에 치명적인 오류를 유발할 수 있다.7 따라서 신뢰성 높은 자율 시스템을 구현하기 위해서는 동적 객체를 효과적으로 식별하고 제거하여 순수한 정적(static) 정보만을 담은 지도를 구축하는 기술이 필수적이다.


이러한 문제를 해결하기 위한 동적 객체 제거(Dynamic Object Removal, DOR) 기법은 처리 시점에 따라 크게 두 가지 범주로 나눌 수 있다: **온라인(Online) 방식**과 **오프라인/후처리(Offline/Post-processing) 방식**이다.7

**온라인 방식**은 LiDAR 데이터를 수신하는 즉시, 즉 SLAM 과정과 동시에 동적 객체를 탐지하고 제거하는 접근법이다. 이 방식은 실시간으로 깨끗한 지도를 생성해야 하는 자율주행차나 모바일 로봇의 내비게이션 시스템에 필수적이다. 온라인 방식은 다시 현재 입력된 스캔에서 동적 포인트를 걸러내는 `scan-removal`과, 이미 구축된 지도에서 동적 포인트를 직접 수정하거나 제거하는 `map-removal`로 세분화될 수 있다.7 대표적인 온라인 기법으로는 RF-LIO 3, FreeDOM 7 등이 있다.

반면, **후처리 방식**은 SLAM 알고리즘을 우선 실행하여 전체 시퀀스에 대한 로봇의 궤적(trajectory)과 초기 지도(raw map)를 먼저 생성한다. 그 후, 누적된 모든 스캔 데이터와 포즈 정보를 종합적으로 활용하여 지도에 포함된 동적 객체들을 일괄적으로 제거하고 정제하는 과정을 거친다.5 이 방식은 실시간성은 보장하지 못하지만, 전체 데이터를 활용할 수 있다는 장점 때문에 더 정교하고 정확한 동적 객체 제거가 가능하다.

본 보고서에서 심층적으로 고찰할 **Removert**는 Kim and Kim (2020)에 의해 제안된 대표적인 후처리 방식의 정적 지도 구축 알고리즘이다.5 Removert가 후처리 방식을 채택한 것은 단순히 구현의 편의성 때문이 아니라, 알고리즘의 핵심 철학인 '제거 후 복원(Remove, then Revert)'을 실현하기 위한 필연적이고 전략적인 선택이었다. 온라인 방식은 각 시점의 데이터만으로 즉각적인 판단을 내려야 하므로, 미래에 들어올 정보를 활용할 수 없다.10 따라서 한번 동적으로 분류된 포인트를 나중에 다시 정적으로 되돌리는 '복원' 과정을 수행하기가 원천적으로 어렵다. 하지만 후처리 방식은 전체 시퀀스의 데이터를 하나의 배치(batch)로 처리할 수 있는 '전지적 시점'을 제공한다.5 이러한 포괄적인 정보 접근 능력 덕분에, Removert는 초기에는 과감하게 동적 객체 후보군을 제거하고(Remove), 이후 SLAM 포즈 오차 등으로 인해 잘못 제거되었을 수 있는 정적 포인트를 점진적으로 복원(Revert)하는 독특한 2단계 접근법을 구현할 수 있었다. 이는 실시간성을 희생하는 대신, 포즈 추정 오차에 대한 강건함과 최종 지도의 품질을 극대화하려는 의도적인 설계적 트레이드오프(trade-off)이며, Removert의 가장 근본적인 설계 사상을 이해하는 핵심 열쇠가 된다.



정적 지도를 구축하는 작업과 동적 객체를 제거하는 작업은 본질적으로 서로 깊이 맞물려 있는 '닭과 달걀(chicken-and-egg)'의 관계에 놓여 있다.5 만약 완벽하게 정적인, 즉 동적 객체가 전혀 없는 기준 지도가 존재한다면, 새로운 LiDAR 스캔과 비교하여 동적 객체를 찾아내는 것은 매우 간단한 문제가 된다. 반대로, 만약 모든 동적 객체를 완벽하게 식별하고 제거할 수 있다면, 남은 포인트들을 모아 정적 지도를 구축하는 것 또한 쉬운 일이다. 현실에서는 두 가지 모두 불완전한 상태에서 시작해야 하므로 문제가 복잡해진다.

Removert는 이 상호의존적인 문제를 정면으로 돌파하기 위해 '제거 후 복원(Remove, then Revert)'이라는 독창적인 2단계 전략을 제안한다.5 이는 먼저 잠재적으로 동적인 포인트를 포함하여 의심스러운 모든 것을 보수적으로(conservatively) 제거한 뒤, 그 과정에서 발생했을 수 있는 오류, 특히 SLAM의 포즈 오차로 인해 정적인 포인트가 동적인 것으로 잘못 분류된 경우를 신중하게 되돌리는(revert) 방식으로 문제를 해결하고자 하는 철학을 담고 있다.


Removert 알고리즘의 효율성과 성능의 중심에는 **거리 이미지(Range Image)**라는 데이터 표현 방식이 있다.5 거리 이미지는 3차원 공간에 흩어져 있는 포인트 클라우드를 LiDAR 센서의 고유한 구면 좌표계(spherical coordinate system)를 이용하여 2D 이미지 형태로 투영한 것이다. 이미지의 각 픽셀 

`$(u, v)$`는 특정 방위각과 고각에 해당하며, 픽셀 값은 해당 방향으로 측정된 거리(range)를 저장한다. 이 변환을 통해 3D 공간에서의 복잡하고 비용이 많이 드는 최근접 이웃 탐색이나 공간 분할 문제를 2D 이미지 평면에서의 빠르고 효율적인 픽셀 단위 연산으로 대체할 수 있다. 이는 전통적인 방식인 레이캐스팅(ray-casting)을 통해 3차원 복셀(voxel)의 점유 여부를 일일이 확인하는 방법에 비해 계산 비용을 획기적으로 절감하는 효과를 가져온다.4

Removert의 독창성은 여기서 한 걸음 더 나아가 **다중 해상도(multiresolution)** 개념을 도입한 데 있다.5 알고리즘은 단일 해상도가 아닌, 서로 다른 해상도의 거리 이미지를 목적에 맞게 활용한다.

- **고해상도(Fine Resolution) 이미지**: 동적 포인트를 정밀하게 식별하고 제거하는 '제거(Remove)' 단계에서 사용된다. 높은 해상도는 객체의 경계를 명확하게 구분하고 미세한 변화를 감지하는 데 유리하다.
- **저해상도(Coarse Resolution) 이미지**: 잘못 제거된 정적 포인트를 되살리는 '복원(Revert)' 단계에서 사용된다. 낮은 해상도는 미세한 위치 변화에 둔감하므로, SLAM의 포즈 오차로 인해 약간의 위치 이동이 발생한 정적 포인트를 주변 영역과 함께 고려하여 복원하는 데 효과적이다.5


Removert의 핵심 알고리즘은 '제거'와 '복원'이라는 명확히 구분된 두 단계로 구성된다.


알고리즘의 근간은 가시성(visibility)에 기반한 불일치(discrepancy) 계산이다. 먼저, 후처리 대상이 되는 기준 지도(`$M$`)와 특정 시점 `$k$`의 LiDAR 스캔(`$S_k$`)을 각각 거리 이미지 `$d_{map}$`과 `$d_{scan}$`으로 변환한다. 두 거리 이미지의 동일한 픽셀 위치 `$(u, v)$`에 대해 저장된 거리 값을 비교한다.

만약 지도상의 거리 값이 현재 스캔의 거리 값보다 유의미하게 크다면, 즉 다음 조건을 만족한다면,
$$
d_{map}(u, v) > d_{scan}(u, v) + \delta
$$
여기서 `$\delta$`는 센서 노이즈 등을 고려한 작은 마진 값이다. 이 조건은 지도상의 포인트가 현재 스캔의 포인트보다 뒤에 위치함을 의미한다. 이는 곧, 과거에는 존재했던 지도상의 포인트가 현재 시점에서는 새로운 객체(현재 스캔의 포인트)에 의해 가려졌다는 뜻이다. 정적 환경이라면 이런 일은 발생할 수 없으므로, 이전에 존재했던 지도상의 포인트는 동적 객체의 일부였고 지금은 그 자리를 떠났다고 강하게 추론할 수 있다. Removert는 이 원리를 이용하여 잠재적 동적 포인트를 1차적으로 식별한다.5


알고리즘의 첫 단계에서는 앞서 계산된 불일치를 바탕으로 잠재적 동적 포인트를 제거한다. 이때 Removert는 매우 보수적인 기준을 적용한다. 즉, '확실하게' 정적이라고 판단되는 포인트(solidly static points)만을 남기고, 조금이라도 동적일 가능성이 있거나 불일치가 관찰된 포인트는 일단 지도에서 제거 대상으로 분류한다.5 이 전략은 동적 포인트를 정적으로 잘못 판단하는 오류(False Negative)를 최소화하는 데 초점을 맞춘다. 그 대가로 일부 정적 포인트가 동적으로 잘못 판단되어 제거되는 오류(False Positive)가 증가할 수 있지만, 이는 다음 단계인 '복원' 과정에서 해결될 문제로 남겨둔다.


이 두 번째 단계야말로 Removert를 다른 기법들과 차별화하는 가장 독창적이고 핵심적인 부분이다. 이 단계는 SLAM 과정에서 필연적으로 발생하는 부정확한 포즈 추정(`$T_k$`)으로 인해, 실제로는 움직이지 않은 정적 포인트가 마치 움직인 것처럼 보여 동적으로 오인되는 문제를 해결하기 위해 고안되었다.5

SLAM의 포즈 추정 결과에는 항상 약간의 오차가 포함되어 있다. 이 오차 때문에 완벽히 고정된 벽의 포인트라 할지라도, 지도에 등록된 위치와 현재 스캔에서 관측된 위치 사이에 미세한 불일치가 발생할 수 있다. 엄격한 1:1 픽셀 비교 방식은 이러한 미세한 불일치를 모두 '동적 변화'로 오인하여 수많은 정적 구조물을 불필요하게 제거하는 심각한 False Positive 문제를 야기한다.

Removert는 이 문제를 '연관 영역 확대(enlarging the association window)'라는 아이디어로 해결한다. 알고리즘은 복원 과정을 여러 번 반복(iteration)하며, 매 반복마다 비교를 위한 **연관 영역(association window)**의 크기를 점진적으로 늘려나간다.

1. **초기 반복 (e.g., 1x1 윈도우)**: 가장 작은 윈도우를 사용하여 1:1 비교를 수행한다. 이 단계에서는 포즈 오차의 영향을 거의 받지 않는, 가장 확실한 정적 포인트들만이 복원된다.
2. **후속 반복 (e.g., 3x3, 5x5 윈도우)**: 윈도우 크기를 점차 확대한다. 1x1 비교에서는 동적으로 판단되었던 포인트라도, 더 넓은 3x3 또는 5x5 주변 영역 내에서 일관되게 정적인 특성이 관찰된다면, 알고리즘은 "이 포인트의 불일치는 객체가 실제로 움직여서 발생한 것이 아니라, 포즈 오차 때문에 픽셀 위치가 약간 밀려난 결과일 것이다"라고 판단한다. 그리고 해당 포인트를 다시 정적 지도에 포함시키는 '복원'을 수행한다.5

이 과정은 저해상도 거리 이미지를 사용하여 비교하는 것과 유사한 효과를 내며, SLAM의 포즈 오차를 명시적으로 모델링하지 않고도 암묵적으로 보상하는 정교한 메커니즘으로 작동한다.5 즉, '복원' 단계는 단순한 재분류 과정이 아니라, 불확실한 포즈 정보를 가진 데이터로부터 강건하게 정적 구조를 추출하기 위한 일종의 확률적 필터링 과정에 가깝다. 이 반복적인 복원 과정은 True Positive (TP), False Positive (FP), False Negative (FN) 포인트 수의 변화를 추적하며, 일반적으로 1~3회 반복 후 변화가 수렴하면 종료된다.5



Removert는 연구 재현성과 활용성을 높이기 위해 오픈소스로 공개되었으며, 그 구조를 통해 알고리즘의 구체적인 동작 방식을 파악할 수 있다.13

- **입력 데이터**: Removert를 실행하기 위해서는 두 종류의 데이터가 필요하다. 첫째는 순차적인 LiDAR 스캔 데이터로, KITTI 데이터셋과 동일한 바이너리 형식(`.bin`)으로 저장되어야 한다. 둘째는 각 스캔이 취득된 시점의 센서 포즈로, SLAM 알고리즘을 통해 추정된 `$SE(3)$` 변환 행렬(12개의 숫자)이 텍스트 파일에 순서대로 기록되어 있어야 한다.13
- **전처리 및 설정**: 실행에 앞서 사용자는 `config/params.yaml` 파일을 수정하여 데이터가 저장된 경로, 센서의 사양(예: LiDAR 채널 수, 최대/최소 측정 거리), 그리고 알고리즘의 주요 파라미터(예: 복원 단계의 반복 횟수, 거리 이미지 해상도 등)를 자신의 환경에 맞게 설정해야 한다.13
- **실행 과정**: 설정이 완료되면 `roslaunch` 명령어를 통해 시스템을 구동한다. 흥미로운 점은 Removert가 내부적으로 ROS의 메시지 통신(토픽)을 전혀 사용하지 않는다는 것이다. ROS는 단지 파라미터 파일을 읽어오는 파서(parser)와 처리 결과를 시각화하기 위한 도구(jsk-visualization)로만 편의상 활용된다.13 이는 시스템이 ROS 환경에 강하게 종속되지 않음을 의미한다.
- **출력**: 알고리즘 실행이 완료되면, 입력된 지도에서 동적 포인트가 제거된 최종 정적 포인트 클라우드 맵이 PCD(`.pcd`) 파일 형식으로 저장된다. 또한, 제거된 동적 포인트들만 따로 모아 파싱할 수도 있어, 제거 성능을 분석하거나 다른 용도로 활용할 수 있다.13


공개된 소스 코드는 Removert의 기술적 특성과 개발 방향성을 명확히 보여준다.13

- **언어 및 의존성**: 코드는 현대적인 C++17 표준을 기반으로 작성되었으며, 로봇 개발에 널리 사용되는 라이브러리인 ROS (Melodic 버전에서 테스트됨), Point Cloud Library (PCL), Eigen, 그리고 병렬 처리를 통한 속도 향상을 위한 OpenMP에 의존성을 가진다.13

- **실행 방식**: 현재 공개된 버전은 하드디스크나 SSD에 미리 저장된 데이터를 순차적으로 읽어와 일괄 처리하는 **오프라인 배치(offline batch) 처리** 방식으로 동작한다. 개별 스캔 한 장을 처리하여 동적 포인트를 제거하는 속도는 10Hz 이상으로 매우 빠르지만, 시스템 전체가 실시간 SLAM 파이프라인과 직접적으로 연동되어 동작하지는 않는다.13

- **향후 개선 방향**: 개발자들은 소스 코드 저장소에서 향후 개선 방향을 명확히 제시하고 있다. 첫째, 현재의 오프라인 방식을 넘어 **실시간 SLAM 시스템에 통합**하는 것을 장기적인 목표로 삼고 있다.13 둘째, 매우 실용적인 

  **하이브리드 접근법**을 제안한다. 이는 딥러닝 기반의 동적 객체 분할(segmentation) 기법(예: LiDAR-MOS)을 먼저 적용하여 차량, 보행자 등 명백한 동적 객체 덩어리를 1차적으로 제거한 후, 그 결과물에 Removert를 적용하여 남아있을 수 있는 미세한 동적 포인트나 아티팩트를 정교하게 다듬는(finer cleaning) 방식이다.13

이러한 구현상의 특징들은 Removert가 실시간 상용 제품을 지향하기보다는, **연구 및 기존 지도의 품질 향상(map refinement)을 위한 실용적인 도구**로서의 성격이 강함을 보여준다. SLAM 시스템에 강하게 결합된(tightly-coupled) 모듈이 아닌, 독립적인 후처리 도구로 제공됨으로써 연구자들은 자신의 SLAM 알고리즘 결과물(포즈와 원시 지도)에 Removert를 손쉽게 적용하여 고품질의 정적 지도를 생성하고 그 성능을 평가할 수 있다. 딥러닝 기법과의 연계를 직접 제안한 점은, Removert 단독으로 모든 동적 환경 문제를 해결하기보다는 다른 기법의 단점을 보완하고 시너지를 내는 '정밀 청소' 도구로서의 역할에 더 큰 강점이 있음을 스스로 인정한 것으로 해석할 수 있다. 이는 Removert가 특정 SLAM 파이프라인에 종속되지 않고 다양한 환경에서 유연하게 활용될 수 있는 모듈형 정적 지도 생성 툴킷으로 자리매김하려는 개발 철학을 반영한다.



Removert의 성능을 평가하기 위해 원본 논문과 후속 연구들에서 제시된 결과들을 종합적으로 분석할 필요가 있다.

- **정량적 지표**: Removert 원본 논문에서는 동적 객체 제거 성능을 평가하는 표준 지표인 정밀도(Precision), 재현율(Recall), F1-점수(F1-score)를 수치로 명확하게 제시하지는 않았다.5 대신, '복원(Revert)' 과정이 반복됨에 따라 True Positive (올바르게 제거된 동적 포인트), False Positive (잘못 제거된 정적 포인트), False Negative (제거되지 못한 동적 포인트)의 개수가 어떻게 변화하는지를 보여주는 그래프를 통해 알고리즘이 의도한 대로 동작함을 간접적으로 보인다.5 이는 알고리즘의 내부 동작 원리를 설명하는 데는 효과적이지만, 다른 기법과의 객관적인 성능 비교에는 한계가 있다.
- **정성적 평가**: KITTI 데이터셋을 이용한 정성적 평가에서, Removert는 SemanticKITTI 데이터셋의 인간이 직접 레이블링한 ground truth와 비교했을 때, 특히 동적인지 정적인지 구분이 애매한 영역에서 경쟁적이거나 더 나은 제거 성능을 보인다고 주장한다.5
- **한계 상황에서의 성능**: 하지만 모든 상황에서 우수한 성능을 보이는 것은 아니다. 특히 보행자 밀도가 매우 높은 혼잡한 시나리오(challenging scenarios)에서의 성능 비교 실험은 Removert의 약점을 명확히 보여준다.10 이 실험에서 Removert는 일부 동적 객체만을 제거하고 지도에 여전히 많은 잔상(traces)을 남겨 제한적인 성능을 보였다. 같은 실험에서 경쟁 기법인 ERASOR는 Removert보다 훨씬 더 많은 동적 객체를 제거하는 데 성공했지만, 벽이나 나무 기둥과 같은 명백한 정적 구조물까지 동적으로 오인하여 제거하는 심각한 False Positive 문제를 드러냈다.10 이는 각 기법이 가진 장단점이 특정 상황에서 어떻게 발현되는지를 잘 보여주는 사례이다.


Removert의 성능과 특징을 더 깊이 이해하기 위해서는 동시대의 주요 경쟁 기법들과의 다각적인 비교가 필수적이다. 아래 표는 Removert를 포함한 주요 동적 객체 제거 기법들의 방법론적 특징을 요약하고 비교 분석한 것이다.

| 기법 (Method)   | 핵심 아이디어 (Core Idea)                                    | 처리 방식 (Processing Type) | 데이터 표현 (Data Representation)        | 장점 (Advantages)                                | 단점 (Disadvantages)                                         |
| --------------- | ------------------------------------------------------------ | --------------------------- | ---------------------------------------- | ------------------------------------------------ | ------------------------------------------------------------ |
| **Removert**    | 다중 해상도 거리 이미지 기반 '제거 후 복원' 5                | 후처리 (Offline) 5          | 거리 이미지 (Range Image) 5              | SLAM 포즈 오차에 강건함 5                        | 정적 객체에 의한 가려짐(Occlusion)에 취약 5                  |
| **ERASOR**      | 유사 점유 기술자(Pseudo Occupancy) 기반 높이 비교 8          | 후처리 (Offline) 12         | 복셀/수직 기둥 (Voxel/Vertical Column) 8 | 가시성에 자유로움(Visibility-free) 15            | 지면 추정 정확도에 의존, 단순한 기술자 표현 8                |
| **ERASOR++**    | ERASOR에 높이 코딩 또는 인스턴스 분할 정보 추가 16           | 후처리 (Offline)            | 복셀 + 시맨틱 정보                       | 인스턴스 단위의 정교한 제거 가능                 | ERASOR의 단점 일부 계승, 딥러닝 모델 의존성 추가             |
| **FreeDOM**     | 보수적 자유 공간 추정(Conservative Free Space Estimation) 7  | 온라인 (Online) 7           | 다중 해상도 복셀 맵                      | 사전 학습 불필요, 가시성 기반 방법의 한계 극복 7 | 단기 정보에 의존, 후처리 방식보다 정확도 낮을 수 있음        |
| **딥러닝 기반** | 시맨틱 분할(Semantic Segmentation)을 통한 동적 클래스 식별 18 | 온라인/오프라인             | 포인트 클라우드, 이미지                  | 복잡한 패턴 인식, 잠재적 동적 객체 사전 식별 19  | 학습 데이터에 없는 객체에 취약, 정적인 동적 클래스 오탐지 11 |

- **Removert vs. ERASOR**: 두 기법은 후처리 방식이라는 공통점을 가지지만, 문제에 접근하는 방식이 근본적으로 다르다. Removert가 2D 거리 이미지 위에서 '가시성 불일치'를 찾는 반면, ERASOR는 3D 공간을 수직 기둥(bin)으로 나누고 각 기둥의 '높이 정보 불일치'를 통해 동적 여부를 판단한다.5 이로 인해 Removert는 포즈 오차에 강건한 복원 메커니즘을 갖추었지만 가려짐 문제에 취약하고, ERASOR는 가시성에 자유로운 대신 지면 추정의 정확도에 크게 의존하는 상반된 장단점을 갖는다.5
- **Removert vs. 딥러닝/시맨틱 분할 기반 기법**: 딥러닝 기반 기법들은 '자동차', '보행자'와 같이 의미론적으로(semantically) 움직일 가능성이 있는 객체 클래스를 사전 학습된 모델로 식별하여 제거한다.18 이는 복잡한 패턴을 가진 동적 객체를 효과적으로 제거할 수 있다는 장점이 있다. 하지만 주차된 차와 같이 '움직일 수 있는 클래스'에 속하지만 '현재는 정적인' 객체까지 무분별하게 제거할 위험이 있으며, 학습 데이터에 존재하지 않았던 새로운 형태의 동적 객체(예: 공사장 장비)에는 대처하기 어렵다.11 반면, Removert는 순수하게 기하학적 변화만을 관찰하므로 이러한 사전 지식 없이도 동작할 수 있는 일반성(generality)을 가진다.

이러한 비교 분석은 동적 객체 제거 분야에 "공짜 점심은 없다(No-Free-Lunch Theorem)"는 원리가 적용됨을 시사한다. 각 방법론은 특정 문제(예: 포즈 오차, 가려짐, 계산 복잡도, 일반성)를 해결하기 위해 다른 측면에서의 성능을 희생하는 명백한 트레이드오프 관계에 있다. Removert는 포즈 오차 문제를 해결하기 위해 복원 메커니즘을 도입했지만, 그 대가로 가려짐 문제에 취약한 거리 이미지를 채택했다. ERASOR는 가려짐 문제를 피하고자 가시성-무관 기술자를 사용했지만, 지면 추정의 정확도라는 새로운 의존성을 갖게 되었다. 딥러닝 기법은 강력한 패턴 인식 능력을 얻었지만, 학습 데이터에 대한 의존성과 일반화 성능이라는 대가를 치른다. 따라서 어떤 단일 기법도 모든 시나리오에서 완벽할 수는 없다. 사용자는 자신의 애플리케이션 환경(예: 실내/실외, 혼잡도, 실시간성 요구)과 기반 SLAM 시스템의 특성(예: 포즈 정확도)을 면밀히 분석하여 가장 적합한 DOR 기법을 선택하거나, 여러 기법을 결합하는 하이브리드 접근법을 고려해야 한다.


Removert는 독창적인 접근법에도 불구하고 몇 가지 명확한 한계점을 가지고 있다.

- **정적 객체에 의한 가려짐 (Occlusion by Static Objects)**: 이는 Removert의 가장 근본적이고 치명적인 약점이다.5 거리 이미지는 센서의 시점에서 볼 때 가장 가까운 표면의 정보만을 담고 있다. 따라서 만약 정적인 벽이나 큰 트럭 뒤에서 보행자가 움직이고 있다면, 그 보행자는 센서에 의해 관측되지 않으므로 거리 이미지에 표현되지 않는다. 당연히 Removert는 이러한 객체의 존재 자체를 인지할 수 없으며, 따라서 제거도 불가능하다. 이는 가시성에 기반한 모든 접근법이 공유하는 내재적 한계이다.
- **혼잡한 환경에서의 성능 저하**: 앞서 언급된 바와 같이, 보행자나 차량이 매우 많고 복잡하게 얽혀 움직이는 밀집 환경에서는 제거 성능이 크게 저하되는 경향을 보인다.10 이는 개별 포인트의 단순한 거리 값 불일치만으로는 객체 간의 복잡한 상호작용과 가려짐을 모두 해석하기 어렵기 때문으로 분석된다. 다수의 동적 객체가 서로를 가리거나 배경을 가리는 상황이 빈번하게 발생하면, 가시성 기반의 판단 규칙이 모호해지고 결과적으로 많은 동적 포인트를 놓치게 된다.



Removert는 동적 환경에서의 정적 지도 구축이라는 난제, 특히 SLAM 과정에서 발생하는 필연적인 포즈 오차라는 현실적인 문제를 '제거 후 복원'이라는 독창적인 2단계 철학으로 해결하고자 한 중요한 학술적 시도이다. 3D 포인트 클라우드를 다중 해상도 거리 이미지로 변환하여 계산 효율성을 확보하고, 점진적인 복원 과정을 통해 포즈 오차에 대한 강건성을 동시에 추구한 점은 Removert의 핵심적인 기여로 평가할 수 있다.5 이 접근법은 동적 객체 제거와 SLAM 오차 보정을 분리된 문제가 아닌, 상호 연관된 문제로 보고 통합적으로 해결하려는 새로운 시각을 제시했다.

그러나 동시에 후처리 방식이 갖는 실시간성의 한계, 가시성 기반 접근법의 고질적인 문제인 가려짐(occlusion) 현상에 대한 취약성, 그리고 객체가 매우 밀집된 환경에서의 성능 저하와 같은 명확한 약점 또한 존재한다.5 이는 Removert가 만능 해결책이 아닌, 특정 장단점을 가진 하나의 방법론임을 의미하며, 그 한계를 명확히 인지하고 활용해야 함을 시사한다.


Removert의 기본 철학과 한계점은 다음과 같은 흥미로운 향후 연구 방향을 제시한다.

- **장기 매핑으로의 확장 (Extension to Long-term Mapping)**: 원본 논문에서 직접 제안한 바와 같이, Removert의 적용 범위를 단일 세션(single-session)에서 다중 세션(multi-session) 데이터로 확장하는 연구가 가능하다.5 이는 단순히 실시간으로 움직이는 객체를 제거하는 것을 넘어, 며칠, 몇 주, 혹은 몇 달에 걸쳐 발생한 환경의 변화(예: 어제는 주차되어 있었지만 오늘은 사라진 자동차, 계절에 따라 변하는 식생 등)를 탐지하고 지도를 지속적으로 업데이트하는 'Lifelong SLAM' 또는 'Lifelong Mapping' 20의 영역으로 나아가는 길이다. Removert의 불일치 탐지 메커니즘은 이러한 장기적인 변화를 감지하는 데 효과적으로 사용될 수 있다.
- **하이브리드 접근법의 심화 (Deepening the Hybrid Approach)**: 개발자들이 제안한 딥러닝과의 결합 아이디어를 더욱 구체화하고 발전시킬 수 있다.13 예를 들어, 시맨틱 분할(semantic segmentation) 네트워크를 이용해 포인트 클라우드에서 '자동차', '보행자' 등 '움직일 가능성이 있는(potentially movable)' 객체 영역을 1차적으로 식별한다. 그 후, 해당 영역 내에서만 Removert의 정밀한 기하학적 불일치 검사를 수행하고, 나머지 영역(건물, 지면 등)은 정적이라고 가정하여 계산에서 제외하는 방식이다. 이는 딥러닝의 높은 식별률과 Removert의 기하학적 정밀성을 결합하여 전체적인 계산 효율성과 제거 정확도를 동시에 높일 수 있는 유망한 전략이다.
- **온라인화 연구 (Research into Online Adaptation)**: Removert의 가장 큰 약점인 후처리 방식을 극복하고, 그 핵심 철학인 '복원' 개념을 온라인 시나리오에 적용하기 위한 연구가 필요하다. 예를 들어, 전체 지도가 아닌 제한된 시간의 과거 데이터만을 유지하는 슬라이딩 윈도우(sliding-window) 방식의 맵을 구축하고, 이 윈도우 내에서 '단기적인 복원'을 수행하는 방식을 생각해 볼 수 있다. 또는, 각 맵 포인트를 단순히 동적/정적으로 이분법적으로 분류하는 대신, 동적일 확률과 정적일 확률을 함께 관리하는 확률적 맵 업데이트 방식을 도입하여, 새로운 관측이 들어올 때마다 이 신뢰도를 점진적으로 갱신하는 방식으로 '복원'의 개념을 구현할 수도 있다.

궁극적으로 Removert의 미래는 단순히 '동적 객체'를 제거하는 것을 넘어, '의미 있는 변화(semantic change)'를 탐지하고 관리하는 더 넓은 문제로 확장될 잠재력을 가지고 있다. 현재 Removert의 목표는 '움직이는 것'을 제거하여 '영원히 정적인 것'을 남기는 것이다.5 그러나 장기적인 관점에서 보면, 어제 주차되어 있던 차가 오늘 사라진 것 역시 로봇이 인지하고 대응해야 할 중요한 '변화'이다. 이 차는 어제는 '정적'이었지만 오늘은 '제거 대상'이 된다. 이는 동적/정적의 구분이 절대적인 것이 아니라 관측 시점에 따라 달라지는 상대적인 개념임을 보여준다. 따라서 Removert의 기하학적 불일치 탐지 메커니즘은 단기적인 움직임뿐만 아니라, 장기적인 '존재/부재의 변화'를 감지하는 강력한 도구로 일반화될 수 있다. 이러한 관점에서 Removert의 핵심 기술은 로봇이 시간이 지남에 따라 변화하는 환경을 스스로 학습하고 지도에 반영하여 지속적으로 임무를 수행하는 **Lifelong SLAM** 20이라는 거대한 연구 패러다임의 핵심 구성 요소로 발전할 수 있는 중요한 가능성을 내포하고 있다.


1. LiDAR-based SLAM for robotic mapping: state of the art and new frontiers - arXiv, 8월 14, 2025에 액세스, https://arxiv.org/pdf/2311.00276
2. A Review of Dynamic Object Filtering in SLAM Based on 3D LiDAR - MDPI, 8월 14, 2025에 액세스, https://www.mdpi.com/1424-8220/24/2/645
3. [2206.09463] RF-LIO: Removal-First Tightly-coupled Lidar Inertial Odometry in High Dynamic Environments - arXiv, 8월 14, 2025에 액세스, https://arxiv.org/abs/2206.09463
4. A Coarse-to-Fine LiDAR-Based SLAM with Dynamic Object Removal in Dense Urban Areas | PolyU, 8월 14, 2025에 액세스, [https://www.polyu.edu.hk/aae/ipn-lab/us/publications/Conference/A%20Coarse-to-Fine%20LiDAR-Based%20SLAM%20with%20Dynamic%20Object%20Removal%20in%20Dense%20Urban%20Areas.pdf](https://www.polyu.edu.hk/aae/ipn-lab/us/publications/Conference/A Coarse-to-Fine LiDAR-Based SLAM with Dynamic Object Removal in Dense Urban Areas.pdf)
5. Remove, then Revert: Static Point cloud Map ... - Giseop Kim, 8월 14, 2025에 액세스, https://gisbi-kim.github.io/publications/gkim-2020-iros.pdf
6. No More Potentially Dynamic Objects: Static Point Cloud Map Generation based on 3D Object Detection and Ground Projection - arXiv, 8월 14, 2025에 액세스, https://arxiv.org/html/2407.01073v1
7. FreeDOM: Online Dynamic Object Removal Framework for Static Map Construction Based on Conservative Free Space Estimation - arXiv, 8월 14, 2025에 액세스, https://arxiv.org/html/2504.11073v1
8. ERASOR++: Height Coding Plus Egocentric Ratio Based Dynamic Object Removal for Static Point Cloud Mapping - arXiv, 8월 14, 2025에 액세스, https://arxiv.org/html/2403.05019v1
9. Moving target removal for lidar SLAM - SPIE Digital Library, 8월 14, 2025에 액세스, https://www.spiedigitallibrary.org/conference-proceedings-of-spie/11763/117637B/Moving-target-removal-for-lidar-SLAM/10.1117/12.2587440.pdf
10. Observation Time Difference: an Online Dynamic Objects Removal Method for Ground Vehicles - arXiv, 8월 14, 2025에 액세스, https://arxiv.org/html/2406.15774v1
11. Mapping the Static Parts of Dynamic Scenes from 3D LiDAR Point Clouds Exploiting Ground Segmentation, 8월 14, 2025에 액세스, https://www.ipb.uni-bonn.de/wp-content/papercite-data/pdf/arora2021ecmr.pdf
12. Efficient Dynamic LiDAR Odometry for Mobile Robots with Structured Point Clouds - arXiv, 8월 14, 2025에 액세스, https://arxiv.org/html/2411.18443v1
13. gisbi-kim/removert: Remove then revert (IROS 2020) - GitHub, 8월 14, 2025에 액세스, https://github.com/gisbi-kim/removert
14. Remove, then Revert: Static Point cloud Map Construction using Multiresolution Range Images | Request PDF - ResearchGate, 8월 14, 2025에 액세스, https://www.researchgate.net/publication/350079625_Remove_then_Revert_Static_Point_cloud_Map_Construction_using_Multiresolution_Range_Images
15. ERASOR: Egocentric Ratio of Pseudo ... - KOASAS - KAIST, 8월 14, 2025에 액세스, [https://koasas.kaist.ac.kr/bitstream/10203/282327/1/074_ERASOR%20Egocentric%20Ratio%20of%20Pseudo%20Occupancy-Based%20Dynamic%20Object%20Removal%20for%20Static%203D%20Point%20Cloud%20Map%20Building.pdf](https://koasas.kaist.ac.kr/bitstream/10203/282327/1/074_ERASOR Egocentric Ratio of Pseudo Occupancy-Based Dynamic Object Removal for Static 3D Point Cloud Map Building.pdf)
16. [2403.05019] ERASOR++: Height Coding Plus Egocentric Ratio Based Dynamic Object Removal for Static Point Cloud Mapping - arXiv, 8월 14, 2025에 액세스, https://arxiv.org/abs/2403.05019
17. ERASOR2: Instance-Aware Robust 3D Mapping of the Static World in Dynamic Scenes, 8월 14, 2025에 액세스, https://www.ipb.uni-bonn.de/wp-content/papercite-data/pdf/lim2023rss.pdf
18. A Semantic Segmentation Based Lidar SLAM System Towards Dynamic Environments | Request PDF - ResearchGate, 8월 14, 2025에 액세스, https://www.researchgate.net/publication/334854295_A_Semantic_Segmentation_Based_Lidar_SLAM_System_Towards_Dynamic_Environments
19. LiDAR-Based SLAM under Semantic Constraints in Dynamic ... - MDPI, 8월 14, 2025에 액세스, https://www.mdpi.com/2072-4292/13/18/3651
20. [2501.18110] Lifelong 3D Mapping Framework for Hand-held & Robot-mounted LiDAR Mapping Systems - arXiv, 8월 14, 2025에 액세스, https://arxiv.org/abs/2501.18110