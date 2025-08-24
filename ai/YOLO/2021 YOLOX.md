# YOLOX (2021)
[YOLO 객체 탐지 모델](./index.md)



2021년 당시 실시간 객체 탐지(Real-time Object Detection) 분야는 YOLOv4, YOLOv5와 같은 모델들이 속도와 정확도 측면에서 SOTA(State-of-the-Art) 성능을 달성하며 앵커 기반(Anchor-based) 파이프라인의 정점을 보여주고 있었다.1 이들 모델은 사전에 정의된 다양한 크기와 종횡비의 앵커 박스(Anchor Box)를 사용하여 객체의 위치를 예측하는 방식을 채택하였고, 이는 당시 객체 탐지의 표준적인 접근법으로 자리 잡았다.

그러나 앵커 기반 방식은 본질적인 한계를 내포하고 있었다. 첫째, 최적의 앵커 박스 설정을 위해 데이터셋에 특화된 클러스터링 분석(예: k-means)이 선행되어야 했으며, 이는 모델의 일반화 성능을 저해하는 요인이었다.3 둘째, 앵커의 수, 크기, 종횡비 등 수많은 하이퍼파라미터를 수동으로 튜닝해야 하는 부담이 있었고, 이는 훈련 과정을 복잡하게 만들었다.5 셋째, 각 그리드 위치마다 여러 개의 앵커에 대한 예측을 수행해야 하므로 예측 헤드의 복잡도가 증가하고, 이는 후처리 과정에서 비최대 억제(Non-Maximum Suppression, NMS)에 대한 과도한 의존으로 이어졌다.3

이러한 상황 속에서, 컴퓨터 비전 학계에서는 새로운 패러다임에 대한 연구가 활발히 진행되고 있었다. 앵커 박스를 사용하지 않는 앵커 프리(Anchor-free) 탐지기(예: FCOS) 1, 훈련 과정에서 보다 지능적으로 positive 샘플을 할당하는 진보된 레이블 할당(Label Assignment) 전략(예: OTA) 1, 그리고 NMS 후처리를 제거하여 완전한 End-to-End 탐지를 구현하려는 시도들이 대표적이다.1 하지만 이러한 최신 연구 성과들은 아직 YOLO 계열의 모델에 본격적으로 통합되지 않은 상태였다.1


YOLOX 개발팀은 이러한 기술적 배경 하에, 최신 학계의 성과를 YOLO 프레임워크에 통합하고자 했다. 흥미로운 점은 당시 최신 모델이었던 YOLOv4나 YOLOv5가 아닌, 상대적으로 구식인 YOLOv3-SPP를 개발의 베이스라인으로 선택했다는 점이다.1 개발팀은 YOLOv4와 YOLOv5가 기존의 앵커 기반 파이프라인에 대해 "과도하게 최적화(a little over-optimized)"되었을 수 있다고 판단했다.1 이는 이미 고도로 튜닝된 모델에 새로운 기술을 적용할 경우, 그 기술의 근본적인 효과를 순수하게 검증하기 어렵다는 전략적 고려에서 비롯된 것이다.

이러한 결정은 YOLOX가 제안하는 방법론의 보편적 가치를 증명하기 위한 탁월한 선택이었다. 만약 최신 모델인 YOLOv5에 약간의 개선을 더해 1-2%의 성능 향상을 보였다면, 그 성과가 근본적인 구조 개선 덕분인지 아니면 단순히 미세 조정의 결과인지 명확히 구분하기 어려웠을 것이다. 반면, 산업계에서 여전히 광범위하게 사용되고 있던 YOLOv3를 기반으로 삼아 Anchor-free, Decoupled Head, SimOTA와 같은 핵심 아이디어를 적용함으로써, ultralytics-YOLOv3의 COCO 데이터셋 AP(Average Precision) 44.3%를 47.3%로 3.0% AP만큼 대폭 끌어올렸다.1 이 결과는 YOLOX의 혁신이 특정 최신 아키텍처에 국한된 것이 아니라, 다양한 모델에 적용될 수 있는 일반적인 원칙임을 명확하게 입증했다.


YOLOX의 궁극적인 목표는 YOLO 계열을 한 단계 더 발전시키는 것이었다. 이를 위해 당시 학계의 최신 연구 성과들, 즉 앵커 프리 설계, 분리된 헤드(Decoupled Head), 그리고 진보된 레이블 할당 전략을 YOLO 프레임워크에 체계적으로 통합하고자 했다.1 속도와 정확도 간의 최적의 균형을 추구하는 YOLO의 전통을 계승하면서, YOLOX-Nano와 같은 초경량 모델부터 YOLOX-X와 같은 고성능 모델에 이르기까지 다양한 규모의 모델 라인업에 일관된 설계 철학을 적용하여 SOTA 성능을 달성하는 것을 목표로 설정했다.1




기존의 앵커 기반 방식은 객체 탐지 파이프라인을 복잡하게 만드는 여러 요소를 포함했다. 사전 정의된 앵커 박스들은 모델의 계산량을 증가시켰고 5, 최적의 앵커 설정을 위해 데이터셋에 대한 사전 통계 분석이 필요했기 때문에 특정 데이터셋에 과적합될 위험이 있었다.3 또한, 각 그리드 위치마다 여러 개의 앵커에 대해 예측을 수행해야 했으므로(YOLOv3의 경우 3개), 총 예측의 수가 불필요하게 많아져 NMS 후처리 과정의 부담을 가중시켰다.3


YOLOX는 이러한 문제들을 해결하기 위해 앵커 박스를 완전히 제거했다. 각 그리드 셀(또는 위치)에서 단 하나의 예측만을 수행하도록 설계를 변경하여, 위치당 예측 수를 3개에서 1개로 대폭 줄였다.3 대신, 각 위치에서 직접 4개의 값을 예측하도록 했다. 이는 그리드 좌상단 모서리로부터의 2개 오프셋(`dx`, `dy`)과 예측된 경계 상자(bounding box)의 높이 및 너비(`h`, `w`)이다.3 이 방식은 FCOS와 같은 Center-based 접근법과 유사하며 6, 다음과 같은 장점을 가진다.

- **설계 단순화**: 앵커 관련 하이퍼파라미터 튜닝의 필요성을 제거하여 모델 아키텍처를 크게 단순화했다.
- **유연성 향상**: 사전 정의된 형태에 얽매이지 않으므로, 불규칙한 모양이나 다양한 크기의 객체에 대해 더 유연하게 대응할 수 있다.6
- **효율성 증대**: 예측 헤드의 복잡도와 총 예측 수를 줄여 훈련 및 추론 과정의 효율성을 높였다.


단순한 앵커 프리 방식에서는 각 실제 객체(Ground-Truth, GT)의 중심점이 위치한 단 하나의 그리드 셀만을 positive 샘플로 간주한다. 이 경우 훈련 초기에 positive 샘플이 극도로 부족해져 학습이 불안정해질 수 있다.14 YOLOX는 이 문제를 완화하기 위해 FCOS에서 제안된 "center sampling" 전략을 채택했다.15 이 전략은 GT 박스의 중심점 주변 `3x3` 영역에 해당하는 그리드 셀들을 모두 positive 샘플로 할당하여, 훈련 초기에 충분한 수의 positive 샘플을 확보하고 학습의 안정성을 높이며 최종적으로 모델의 재현율(recall)을 향상시키는 데 기여했다.3



YOLOv3부터 YOLOv5에 이르는 모델들은 결합된 헤드(Coupled Head) 구조를 사용했다. 이는 객체의 종류를 판별하는 분류(Classification) 작업과 객체의 위치 및 크기를 예측하는 회귀(Regression) 작업을 단일 컨볼루션 레이어에서 동시에 처리하는 방식이었다.2 그러나 이 두 작업은 근본적으로 다른 목표를 가진다. 분류 작업은 객체의 외형적 특징에 집중하므로 공간적 위치 변화에 덜 민감해야 하는 반면(spatial invariance), 회귀 작업은 픽셀 단위의 정확한 위치 정보에 매우 민감하다. 이처럼 상이한 두 작업을 하나의 헤드에서 처리하려는 시도는 작업 간 상충 관계(conflict)를 유발하여 모델의 학습을 저해하고 성능을 제한하는 요인으로 작용했다.11


YOLOX는 이 문제를 해결하기 위해 대부분의 최신 1-stage 및 2-stage 탐지기에서 이미 효과가 검증된 분리된 헤드(Decoupled Head) 구조를 도입했다.2 그 구조는 다음과 같다. FPN(Feature Pyramid Network)의 각 레벨에서 나온 피처 맵에 대해, 먼저 $1 \times 1$ Conv 레이어를 적용하여 채널 차원을 256으로 축소한다. 그 후, 네트워크는 두 개의 병렬 브랜치로 분기된다. 각 브랜치는 두 개의 $3 \times 3$ Conv 레이어로 구성되며, 하나는 분류(Cls) 작업을, 다른 하나는 회귀(Reg) 및 객체 존재 여부(Objectness/IoU) 예측을 독립적으로 수행한다.1 이 구조적 차이는 YOLOX 논문의 Figure 2에 명확히 시각화되어 있다.1


Decoupled Head의 도입은 여러 측면에서 긍정적인 효과를 가져왔다. 실험적으로 모델의 수렴 속도가 크게 향상되었으며 2, Ablation Study에서는 이 구조 변경만으로 YOLOv3 베이스라인의 AP가 1.1% 상승하는 등 최종 정확도 향상에도 직접적으로 기여했다.2

더 나아가, Decoupled Head의 도입은 단순한 성능 개선을 넘어, YOLO 아키텍처를 NMS-free와 같은 보다 진보된 형태로 확장하기 위한 근본적인 구조 변경이라는 점에서 더 깊은 의미를 가진다. NMS는 추론 속도를 저하시키는 대표적인 병목 지점으로, 이를 제거하는 것은 End-to-End 탐지기의 주요 목표 중 하나이다.18 NMS를 제거하려면 모델이 스스로 중복된 예측을 억제하는 매우 섬세한 학습을 수행해야 한다. Coupled Head 구조에서는 분류와 회귀 작업이 서로를 방해하기 때문에 이러한 학습이 제대로 이루어지기 어렵다. 실제로 YOLOX의 실험에서 Coupled Head를 사용한 End-to-End 버전은 AP가 4.2%나 하락하는 심각한 성능 저하를 보였다.2 반면, Decoupled Head는 각 작업이 독립적으로 최적화될 수 있는 환경을 제공하여 성능 저하를 0.8%로 최소화했다.2 이는 Decoupled Head가 YOLO 계열의 장기적인 발전, 즉 End-to-End화를 위한 필수적인 기반을 마련했음을 시사한다.



레이블 할당은 훈련 과정에서 수많은 예측 박스 중 어떤 것을 positive로, 어떤 것을 negative로 간주할지 결정하는 핵심적인 과정이다.8 기존의 Max-IoU와 같은 정적 할당 방식은 GT 박스와 가장 IoU(Intersection over Union)가 높은 하나의 예측만을 positive로 할당하는 등 고정된 규칙에 의존했다. 이러한 방식은 여러 객체가 겹쳐 있거나 경계가 불분명한 경우, 최적이 아닌 할당을 유발하여 모델의 학습 잠재력을 충분히 이끌어내지 못하는 한계가 있었다.13


OTA는 레이블 할당 문제를 "최적 수송 문제(Optimal Transport Problem)"라는 수학적 프레임워크로 재정의하여 이 문제를 해결하고자 했다.8 이를 비유적으로 설명하면 다음과 같다. GT들을 한정된 수량($k$개)의 'positive label'이라는 상품을 가진 '공급자'로, 수많은 예측들을 '수요자'로 간주한다. 이때 '운송 비용'은 예측과 GT 간의 불일치 정도(분류 손실과 회귀 손실의 가중합)로 정의된다. OTA의 목표는 전체 '운송 비용'을 최소화하면서 모든 수요를 충족시키는 최적의 '운송 계획', 즉 최적의 레이블 할당을 찾는 것이다.8


OTA는 Sinkhorn-Knopp이라는 반복적인 최적화 알고리즘을 사용하여 수학적으로 보장된 최적의 할당을 찾는다. 이는 성능 향상에 매우 효과적이었지만, 훈련 시간을 약 25% 증가시키는 상당한 계산 부담을 동반했다.8 YOLOX는 실시간 탐지 모델의 핵심인 속도를 저해하지 않으면서 OTA의 장점을 취하기 위해 SimOTA(Simplified OTA)를 제안했다. SimOTA는 복잡한 Sinkhorn-Knopp 알고리즘 대신, 각 GT에 대해 독립적으로 비용이 가장 낮은 상위 $k$개의 예측을 positive 샘플로 선택하는 간단한 탐욕적(greedy) 방식으로 최적 할당을 근사한다.3 이 단순화된 접근법은 OTA의 성능 향상 효과를 대부분 유지하면서도 추가적인 훈련 시간 부담을 거의 없애는 데 성공했다.8


$j$번째 예측과 $i$번째 GT 간의 비용 $c_{ij}$는 다음과 같이 계산된다.3
$$
c_{ij} = L_{cls}^{ij} + \lambda L_{reg}^{ij}
$$
여기서 $L_{cls}$는 분류 손실(주로 Focal Loss 사용), $L_{reg}$는 회귀 손실(주로 IoU Loss 사용)이며, $\lambda$는 두 손실 간의 균형을 맞추는 하이퍼파라미터이다.

또한 SimOTA는 각 GT가 할당해야 할 positive 샘플의 수 $k$를 고정값으로 두지 않고, 동적으로 결정하는 **Dynamic Top-k** 방식을 사용한다. 구체적으로, 각 GT에 대해 모든 예측과의 IoU를 계산하고, 그중 IoU가 높은 상위 `q`개(예: `q=10`) 예측들의 IoU 값을 모두 합산하여 해당 GT의 $k$ 값으로 설정한다. 이 방식은 크고 명확하게 예측된 객체에는 더 많은 positive 샘플을, 작고 불분명한 객체에는 더 적은 샘플을 할당하도록 유도하여, 보다 유연하고 합리적인 학습을 가능하게 한다.3 추가적으로, GT의 중심에서 멀리 떨어진 예측에는 높은 비용을 부과하거나 후보에서 제외하는 **Center Prior** 전략을 통해 학습 초기의 불안정한 할당을 방지하고 훈련을 안정화시켰다.3



YOLOX는 모델의 강건함과 일반화 성능을 극대화하기 위해 Mosaic과 MixUp이라는 두 가지 강력한 데이터 증강(Data Augmentation) 기법을 기본 훈련 전략으로 채택했다.3 Mosaic은 4개의 이미지를 무작위로 잘라 하나로 합치는 기법이며, MixUp은 두 이미지를 픽셀 단위로 선형 보간하는 기법이다.16 이러한 강력한 데이터 증강 덕분에, 대규모 데이터셋인 ImageNet으로 사전 훈련된 가중치를 사용하는 것보다 처음부터(from scratch) 훈련하는 것이 더 나은 성능을 보였다.1

YOLOX의 데이터 증강 전략은 단순히 여러 기법을 적용하는 데 그치지 않고, 모델의 용량과 훈련 단계를 모두 고려한 정교한 미세 조정을 포함한다. 이는 'one-size-fits-all' 접근법이 아님을 보여준다. 데이터 증강은 일반적으로 성능 향상에 도움이 되지만, 모델의 표현 능력(capacity)을 초과하는 과도한 증강은 오히려 학습을 방해할 수 있다. YOLOX 연구진은 이 점을 인지하고 모델 크기별로 차등적인 전략을 적용했다.

- **대형 모델 (YOLOX-L 등)**: Mosaic과 MixUp을 모두 적극적으로 사용하여 성능을 극대화했다. 실험 결과, MixUp 적용 시 0.9% AP 향상 효과가 있었다.1
- **소형 모델 (YOLOX-S, Tiny, Nano)**: 파라미터 수가 적은 소형 모델은 강력한 증강에 과적합될 위험이 있다. 따라서 MixUp은 사용하지 않고, Mosaic의 이미지 스케일링 범위를 `[0.1, 2.0]`에서 `[0.5, 1.5]`로 줄여 증강의 강도를 약화시켰다. 이 전략을 통해 YOLOX-Nano의 AP는 24.0%에서 25.3%로 오히려 향상되었다.1

또한, 전체 300 에포크 훈련 중 마지막 15 에포크에서는 Mosaic과 MixUp을 비활성화했다.1 이는 마치 운동선수가 실전 경기를 앞두고 훈련 강도를 조절하는 것과 같다. 모델이 최종 수렴 단계에서 증강되지 않은 '깨끗한' 데이터, 즉 실제 추론 환경과 유사한 데이터 분포에 미세 조정될 시간을 줌으로써 최종 성능을 안정화시키고 향상시키는 효과를 가져온다.19 이처럼 YOLOX는 각 요소들이 최적의 시너지를 내도록 세심하게 훈련 파이프라인을 설계했음을 보여준다.


YOLOX는 산업 현장의 다양한 요구사항에 대응하기 위해 경량 모델부터 고성능 모델까지 폭넓은 라인업을 제공한다. YOLOX-Nano (0.91M 파라미터)부터 YOLOX-Tiny, YOLOX-S, YOLOX-M, YOLOX-L, 그리고 YOLOX-X (99.1M 파라미터)에 이르기까지, 사용자는 자신의 하드웨어 제약과 성능 목표에 맞춰 최적의 모델을 선택할 수 있다.1

더불어 YOLOX는 실제 배포의 중요성을 깊이 인지하고 있었다. 이를 위해 ONNX, TensorRT, NCNN, OpenVINO 등 다양한 추론 엔진 및 프레임워크를 공식적으로 지원하여, 개발자들이 훈련된 모델을 손쉽게 다양한 엣지 디바이스나 서버 환경에 적용할 수 있도록 생태계를 구축했다.1 이는 YOLOX가 학술적 성과에만 머무르지 않고, 산업적 실용성을 깊이 고려하여 개발되었음을 보여주는 중요한 부분이다.



YOLOX 논문은 제안된 각 기술 요소가 최종 성능에 얼마나 기여했는지를 정량적으로 분석하기 위해 상세한 점진적 개선 실험(Ablation Study) 결과를 제시했다. 이는 YOLOv3-SPP 베이스라인에서 시작하여 각 기술을 순차적으로 추가하며 성능 변화를 측정한 것으로, 각 혁신의 독립적인 가치를 명확히 보여준다.

Table 1: YOLOX-DarkNet53 Ablation Study (COCO val)

이 표는 YOLOX를 구성하는 각 핵심 기술 요소가 최종 성능에 미치는 영향을 정량적으로 분해하여 보여준다. 이를 통해 어떤 기술이 성능 향상에 가장 크게 기여했는지 직관적으로 파악할 수 있다.

| 구성 요소                                    | AP (%)   | AP 향상분 |
| -------------------------------------------- | -------- | --------- |
| YOLOv3-SPP Baseline (Ultralytics)            | 44.3     | -         |
| YOLOX Baseline (EMA, Cosine, IoU loss, etc.) | 38.5     | -         |
| + Decoupled Head                             | 42.7     | +4.2      |
| + Strong Augmentation (Mosaic, MixUp)        | 44.8     | +2.1      |
| + Anchor-free                                | 45.9     | +1.1      |
| + Multi-positives                            | 46.5     | +0.6      |
| + SimOTA                                     | **47.3** | **+0.8**  |

표의 데이터는 2을 기반으로 재구성됨.

분석 결과, Decoupled Head와 강력한 데이터 증강이 초기 성능 향상을 크게 이끌었으며, 이후 Anchor-free 전환과 SimOTA와 같은 진보된 전략들이 성능을 더욱 정교하게 다듬었음을 확인할 수 있다.


YOLOX의 객관적인 성능 수준과 속도-정확도 트레이드오프를 평가하기 위해, 동시대 및 후속 SOTA 모델들과 COCO test-dev 벤치마크에서 직접 비교한 결과는 다음과 같다.

Table 2: 주요 객체 탐지 모델 성능 비교 (COCO test-dev)

이 표는 YOLOX를 동시대 및 후속 SOTA 모델들과 동일한 벤치마크에서 직접 비교함으로써, 객관적인 성능 수준과 속도-정확도 트레이드오프 곡선 상에서의 위치를 평가한다.

| 모델        | 해상도  | mAP (test-dev) | 속도 (V100, FPS) | 파라미터 (M) | FLOPs (G) |
| ----------- | ------- | -------------- | ---------------- | ------------ | --------- |
| YOLOv3-SPP  | 640x640 | 44.3           | ~65              | 63.0         | ~156      |
| YOLOv4-CSP  | 640x640 | 47.5           | ~57              | 52.8         | 126.1     |
| YOLOv5-L    | 640x640 | 48.2           | 115              | 46.5         | 109.1     |
| **YOLOX-S** | 640x640 | 40.5           | 118              | 9.0          | 26.8      |
| **YOLOX-M** | 640x640 | 46.9           | 93               | 25.3         | 73.8      |
| **YOLOX-L** | 640x640 | 50.1           | 97               | 54.2         | 155.6     |
| **YOLOX-X** | 640x640 | 51.5           | 81               | 99.1         | 281.9     |
| YOLOv7      | 640x640 | 51.4           | -                | 36.9         | 104.7     |
| YOLOv8-L    | 640x640 | 52.9           | -                | 43.6         | 165.2     |

표의 데이터는 1를 기반으로 종합됨.


- **YOLOv5-L 대비 성능 우위**: YOLOX-L은 YOLOv5-L과 유사한 수준의 파라미터 및 연산량(FLOPs)을 가지면서도 COCO AP에서 1.8% 더 높은 50.1%를 달성했다.1 이는 Anchor-free, Decoupled Head, SimOTA의 조합이 당시 앵커 기반 파이프라인의 성능 한계를 넘어섰음을 명확히 보여주는 결과이다.
- **속도-정확도 트레이드오프**: YOLOX는 모든 모델 스케일에서 당시 SOTA 수준의 속도-정확도 균형을 달성했다. 특히 YOLOX-Nano와 같은 경량 모델은 경쟁 모델인 NanoDet을 큰 차이로 능가하며, 엣지 디바이스에서의 실용성을 입증했다.1
- **후속 모델에 미친 영향**: YOLOv7, YOLOv8과 같은 후속 모델들은 네트워크 아키텍처 재설계(예: E-ELAN), 재파라미터화(re-parameterization) 등의 새로운 기술을 도입하며 YOLOX의 성능을 넘어섰다.21 하지만 이들 모델 중 다수가 YOLOX가 도입한 Anchor-free 설계와 Decoupled Head 구조를 계승하거나 발전시켰다는 점에서 YOLOX의 깊은 기술적 영향력을 확인할 수 있다.21



YOLOX의 높은 실시간 성능과 정확도는 다양한 산업 분야에서 높은 활용 가치를 지닌다. 대표적으로 자율주행 시스템에서의 보행자 및 차량 탐지, 영상 감시 시스템에서의 이상 행동 감지, 로보틱스에서의 물체 조작, 의료 영상 분석에서의 병변 탐지, 그리고 소매 및 물류 자동화 등에서 핵심 기술로 활용될 수 있다.13

실제 적용 사례로는 산업 현장의 인프라 균열, 부식, 태양광 패널 핫스팟과 같은 미세 결함 탐지 24, 교통 상황 모니터링 25, 재활용 시설에서의 폐기물 자동 분류 26 등이 연구 및 보고되었다. 특히 경량 모델인 YOLOX-Nano는 컴퓨팅 자원이 제한된 드론이나 모바일 기기에서 실시간 객체 탐지를 가능하게 하여, 원격 감시를 통한 토사류 재해 감지 등 새로운 응용 분야를 개척했다.27


YOLOX는 중요한 기술적 진보를 이루었지만, 동시에 몇 가지 한계점을 가지고 있었으며 이는 후속 연구의 방향을 제시했다.

- **NMS 의존성**: SimOTA를 통해 훈련 시 레이블 할당 문제를 해결했지만, 추론 단계에서는 여전히 NMS 후처리에 의존했다. NMS는 추론 속도를 저하시키고 하이퍼파라미터에 민감하며, 완전한 End-to-End 학습을 방해하는 요소이다. 이 한계는 YOLOv10과 같은 후속 연구에서 NMS-free를 목표로 하는 직접적인 계기가 되었다.18
- **아키텍처의 한계**: YOLOX는 YOLOv3/v5의 기본 골격인 CSPNet 백본과 PAN 넥 구조를 크게 변경하지 않았다. 이후 YOLOv7의 E-ELAN, YOLOv9의 GELAN 등 더 효율적이고 강력한 특징 추출 및 융합을 위한 새로운 네트워크 아키텍처가 제안되면서, YOLOX의 기본 아키텍처는 상대적으로 구식이 되었다.21
- **Open-Vocabulary 한계**: 모든 전통적인 YOLO 계열 모델과 마찬가지로, YOLOX는 훈련 데이터셋에 사전 정의된 클래스만 탐지할 수 있다는 근본적인 한계를 가진다. 이는 YOLO-World와 같은 후속 연구에서 Vision-Language 모델링을 통해 '개방형 어휘(Open-Vocabulary)' 탐지 능력으로 확장하는 연구로 이어졌다.29


YOLOX는 단순히 YOLO 시리즈의 성능을 점진적으로 개선한 모델이 아니다. 앵커 기반 패러다임이 지배적이던 시기에, 과감하게 앵커 프리 설계를 전면에 내세우고, Decoupled Head와 동적 레이블 할당(SimOTA)과 같은 학계의 최신 연구 성과를 성공적으로 통합하여 YOLO 생태계에 새로운 방향성을 제시한 '게임 체인저(Game Changer)'였다.

YOLOX가 제시한 설계 원칙들, 특히 Anchor-free와 Decoupled Head는 이후 등장한 YOLOv6, v7, v8 등 수많은 후속 객체 탐지 모델의 사실상 표준(de facto standard)으로 자리 잡았다.21 이는 YOLOX가 YOLO의 기술적 진화 과정에서 중요한 변곡점을 만들었음을 의미한다.

결론적으로 YOLOX는 속도-정확도 곡선을 한 단계 끌어올렸을 뿐만 아니라, YOLO 프레임워크의 복잡성을 줄이고 유연성을 높이며, 후속 연구들이 나아갈 길을 제시한 중요한 기술적 이정표로 평가받아야 한다.


1. [2107.08430] YOLOX: Exceeding YOLO Series in 2021 - ar5iv - arXiv, 8월 24, 2025에 액세스, https://ar5iv.labs.arxiv.org/html/2107.08430
2. arXiv:2107.08430v2 [cs.CV] 6 Aug 2021, 8월 24, 2025에 액세스, https://arxiv.org/pdf/2107.08430
3. Brief Review — YOLOX: Exceeding YOLO Series in 2021 | by Sik-Ho Tsang | Medium, 8월 24, 2025에 액세스, https://sh-tsang.medium.com/brief-review-yolox-exceeding-yolo-series-in-2021-70182b70473f
4. What are anchor-free Object Detectors? - LearnOpenCV - Quora, 8월 24, 2025에 액세스, https://learnopencv.quora.com/What-are-anchor-free-Object-Detectors
5. Paper Review: “YOLOX: Exceeding YOLO Series in 2021” | by Anh Tuan - Medium, 8월 24, 2025에 액세스, https://medium.com/mlearning-ai/paper-review-yolox-exceeding-yolo-series-in-2021-ffc1bd94a1f3
6. Anchor-Free Object Detection - Ultralytics, 8월 24, 2025에 액세스, https://www.ultralytics.com/glossary/anchor-free-detectors
7. FCOS- Anchor Free Object Detection Explained | LearnOpenCV #, 8월 24, 2025에 액세스, https://learnopencv.com/fcos-anchor-free-object-detection-explained/
8. YOLOX Explanation — SimOTA For Dynamic Label Assignment | by ..., 8월 24, 2025에 액세스, https://gmongaras.medium.com/yolox-explanation-simota-for-dynamic-label-assignment-8fa5ae397f76
9. [PDF] YOLOX: Exceeding YOLO Series in 2021 - Semantic Scholar, 8월 24, 2025에 액세스, https://www.semanticscholar.org/paper/YOLOX%3A-Exceeding-YOLO-Series-in-2021-Ge-Liu/c01b385205e488a731c8c8c11c0c494d426beb03
10. YOLOX: Exceeding YOLO Series in 2021 - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/353343997_YOLOX_Exceeding_YOLO_Series_in_2021
11. improved YOLOX approach for low-light and small object detection: PPE on tunnel construction sites | Journal of Computational Design and Engineering | Oxford Academic, 8월 24, 2025에 액세스, https://academic.oup.com/jcde/article/10/3/1158/7177527
12. YOLOX Explanation — How Does YOLOX Work? | by Gabriel ..., 8월 24, 2025에 액세스, https://gmongaras.medium.com/yolox-explanation-how-does-yolox-work-3e5c89f2bf78
13. Understanding YOLOX: Key Features and Benefits for Object Detection - Thinking Stack, 8월 24, 2025에 액세스, https://www.thinkingstack.ai/blog/vision-ai-9/understanding-yolox-key-features-and-benefits-for-object-detection-55
14. YOLO Object Detection Explained: A Beginner's Guide - DataCamp, 8월 24, 2025에 액세스, https://www.datacamp.com/blog/yolo-object-detection-explained
15. [CVPR2021/PaperSummary]YOLOX: Exceeding YOLO Series in 2021 | by abhigoku10, 8월 24, 2025에 액세스, https://abhigoku10.medium.com/cvpr2021-papersummary-yolox-exceeding-yolo-series-in-2021-c6423199b1fc
16. YOLOX Explained: Features, Architecture and Applications - Viso Suite, 8월 24, 2025에 액세스, https://viso.ai/computer-vision/yolox/
17. YOLOCS: Object Detection based on Dense Channel Compression for Feature Spatial Solidification - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/pdf/2305.04170
18. YOLOv10: Real-Time End-to-End Object Detection - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/pdf/2405.14458
19. Data Augmentation using Ultralytics YOLO, 8월 24, 2025에 액세스, https://docs.ultralytics.com/guides/yolo-data-augmentation/
20. YOLOv9 vs. YOLOX: A Technical Comparison - Ultralytics YOLO Docs, 8월 24, 2025에 액세스, https://docs.ultralytics.com/compare/yolov9-vs-yolox/
21. YOLOX vs. YOLOv7: A Technical Comparison - Ultralytics YOLO Docs, 8월 24, 2025에 액세스, https://docs.ultralytics.com/compare/yolox-vs-yolov7/
22. (PDF) Yolov5, Yolo-x, Yolo-r, Yolov7 Performance Comparison: A Survey - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/363848609_Yolov5_Yolo-x_Yolo-r_Yolov7_Performance_Comparison_A_Survey
23. YOLOX vs. YOLOv8: A Technical Comparison - Ultralytics YOLO Docs, 8월 24, 2025에 액세스, https://docs.ultralytics.com/compare/yolox-vs-yolov8/
24. YOLOX-Ray: An Efficient Attention-Based Single-Staged Object Detector Tailored for Industrial Inspections - PubMed Central, 8월 24, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC10222537/
25. Application of i-YOLOX in multi-object scenarios. (a) is a bathroom... - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/figure/Application-of-i-YOLOX-in-multi-object-scenarios-a-is-a-bathroom-scene-to-detect_fig12_363619049
26. 16 Applications of AI-Powered Object Detection in Businesses - Techlancers, 8월 24, 2025에 액세스, https://techlancersme.com/blog/16-applications-of-ai-powered-object-detection-in-businesses/
27. Application of Enhanced YOLOX for Debris Flow Detection in Remote Sensing Images, 8월 24, 2025에 액세스, https://www.mdpi.com/2076-3417/14/5/2158
28. Sensing for Space Safety and Sustainability: A Deep Learning Approach with Vision Transformers - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2412.08913v1
29. YOLO-World: Real-Time Open-Vocabulary Object Detection - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2401.17270v2