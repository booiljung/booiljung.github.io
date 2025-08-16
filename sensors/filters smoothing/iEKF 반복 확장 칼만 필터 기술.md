# 반복 확장 칼만 필터(iEKF) 기술

## 1. 서론: 비선형 상태 추정의 과제

### 0.1  선형 시스템의 최적 추정기, 칼만 필터

칼만 필터(Kalman Filter, KF)는 잡음이 포함된 일련의 측정치를 사용하여 동적 시스템의 상태를 추정하는 매우 강력한 재귀적 알고리즘이다.1 루돌프 칼만에 의해 개발된 이 필터는 시스템의 동적 모델과 측정 모델이 모두 선형(linear)이고, 시스템 프로세스 잡음과 측정 잡음이 모두 평균이 0인 가우시안(Gaussian) 분포를 따른다고 가정할 때, 최소 평균 제곱 오차(Minimum Mean Square Error, MMSE) 관점에서 최적의 상태 추정치를 제공한다.1

알고리즘은 예측(Prediction)과 업데이트(Update)라는 두 가지 핵심 단계를 재귀적으로 반복하는 구조를 가진다.1 예측 단계에서는 이전 시간의 상태 추정치와 시스템의 동적 모델을 사용하여 현재 시간의 상태를 예측한다. 업데이트 단계에서는 새로 들어온 측정값을 사용하여 예측된 상태를 보정하고, 더 정확한 현재 상태 추정치를 계산한다. 이러한 구조 덕분에 칼만 필터는 레이다 추적, 컴퓨터 비전, 로봇 공학 등 다양한 분야에서 성공적으로 사용되어 왔다.1 하지만 칼만 필터의 최적성은 선형성과 가우시안성이라는 엄격한 가정에 의존하며, 이 가정이 깨지는 순간 그 성능은 보장되지 않는다.5

### 0.2  비선형 세계로의 첫걸음, 확장 칼만 필터 (EKF)

대부분의 현실 세계 시스템, 특히 자율 항법, 로보틱스, 항공 우주 분야의 시스템들은 비선형적인 동특성을 가진다.5 예를 들어, 로봇의 움직임이나 센서 측정 과정은 삼각함수나 복잡한 기하학적 관계로 표현되는 경우가 많다. 이러한 비선형 시스템에 선형 칼만 필터를 직접 적용하는 것은 불가능하다.

이 문제를 해결하기 위한 가장 고전적이고 널리 알려진 방법이 바로 확장 칼만 필터(Extended Kalman Filter, EKF)이다.1 EKF의 핵심 아이디어는 비선형 함수를 특정 지점 주변에서 선형으로 근사하여 칼만 필터를 적용하는 것이다.8 구체적으로, 비선형 상태 전이 함수 $f(x)$와 측정 함수 $h(x)$를 현재 상태 추정치 주변에서 1차 테일러 급수 전개(Taylor series expansion)를 통해 선형화한다.7 이 과정에서 함수의 편미분으로 구성된 자코비안 행렬(Jacobian matrix)이 선형 칼만 필터의 상태 전이 행렬($A$)과 측정 행렬($H$)의 역할을 대신하게 된다.7 이처럼 EKF는 매 순간 시스템을 강제로 선형화하여 칼만 필터의 프레임워크를 적용하는 방식으로 비선형 문제를 다룬다.

### 0.3  EKF의 본질적 한계와 iEKF의 필요성

EKF는 비선형 문제에 대한 간단하고 효과적인 해결책을 제시하지만, 근본적인 한계를 내포하고 있다.

첫째, **선형화 오차(Linearization Error)**이다. 1차 테일러 급수 전개는 비선형 함수의 곡률을 무시하고 접선으로 근사하는 방식이다. 만약 시스템의 비선형성이 강하다면, 이 선형 근사는 실제 비선형 함수와 상당한 오차를 가지게 되며, 이 오차는 필터의 성능을 직접적으로 저하시키는 원인이 된다.10 EKF는 이 선형화로 인해 발생하는 오차를 보상할 어떠한 내부 메커니즘도 가지고 있지 않다.

둘째, **발산 위험(Risk of Divergence)**이다. 부정확한 선형화는 필터의 핵심 요소인 칼만 이득(Kalman Gain)을 잘못 계산하게 만든다. 칼만 이득은 예측값과 측정값 사이의 신뢰도를 조절하여 상태를 보정하는 가중치 역할을 하는데, 이 값이 부정확하면 필터는 오차를 줄이는 대신 오히려 증폭시키는 양의 피드백 루프에 빠질 수 있다. 최악의 경우, 추정 오차가 무한대로 커지는 발산 현상이 발생할 수 있다.11

셋째, **비최적성(Sub-optimality)**이다. 이러한 선형화 오차 때문에 EKF는 더 이상 베이즈 필터 이론에 입각한 최적의 추정기가 아니다.7

이러한 문제의 근원은 EKF가 사전 추정치($\hat{x}_k^-$)라는 *단 하나의 지점*에서만 선형화를 수행한다는 점에 있다. 만약 이 사전 추정치가 실제 상태와 멀리 떨어져 있다면, 그 지점에서의 접선(자코비안)은 실제 측정 함수 $h(x)$의 형태를 제대로 대표하지 못한다. 이는 마치 산 정상의 위치를 추정하는데, 산기슭의 경사 정보만을 이용하는 것과 같다. 이로 인해 계산된 칼만 이득은 엉뚱한 방향으로 상태를 보정하게 되고, 이는 필터의 성능 저하를 넘어 발산으로까지 이어질 수 있다.12 이러한 EKF의 본질적인 한계를 극복하고, 특히 측정 모델의 비선형성이 강한 경우에 더 정확한 상태 추정을 달성하기 위해 반복 확장 칼만 필터(Iterated Extended Kalman Filter, iEKF)가 제안되었다. iEKF는 바로 이 '선형화 지점'을 새로 들어온 측정값($z_k$)을 이용해 더 나은 곳으로 반복적으로 옮겨가려는 시도이다.

## 1.  반복 확장 칼만 필터(iEKF)의 원리

### 1.1  핵심 아이디어: 더 나은 선형화 지점을 찾기 위한 반복

반복 확장 칼만 필터(iEKF)는 EKF의 선형화 오차를 줄이기 위해 측정치 업데이트 단계를 여러 번 반복하는 방식으로 EKF를 개선한 알고리즘이다.7 iEKF의 철학은 간단하다. EKF가 사전 추정치($\hat{x}_k^-$)라는 단 한 번의 기회만을 가지고 선형화를 수행하는 반면, iEKF는 새로 들어온 측정치($z_k$)라는 귀중한 정보를 활용하여 더 나은 선형화 지점을 찾을 때까지 상태 추정치를 반복적으로 정제한다.

핵심적인 과정은 다음과 같다. 먼저 EKF와 동일하게 사전 추정치를 계산한다. 그 후, 이 사전 추정치를 초기값으로 하여 반복 루프를 시작한다. 각 반복 단계에서는 현재의 상태 추정치를 기준으로 측정 함수를 다시 선형화(즉, 자코비안을 재계산)하고, 이를 바탕으로 칼만 이득을 구해 상태 추정치를 업데이트한다. 이렇게 업데이트된 새로운 상태 추정치는 다음 반복에서 더 나은 선형화의 기준점(center point of Taylor expansion)이 된다.2 이 과정은 상태 추정치가 더 이상 크게 변하지 않거나 미리 정해진 횟수만큼 반복될 때까지 계속된다.

이러한 반복 과정을 통해 iEKF는 측정치 $z_k$가 가리키는, 확률적으로 가장 가능성이 높은 상태 값에 더 가깝게 선형화 지점을 이동시킬 수 있다. 이는 특히 측정 모델 $h(x)$의 비선형성이 강하여 EKF의 단일 선형화가 큰 오차를 유발하는 경우에 매우 효과적이다.13

### 1.2  가우스-뉴턴 최적화 관점에서의 해석

iEKF의 동작 원리는 비선형 최적화 기법인 가우스-뉴턴(Gauss-Newton, GN) 방법의 관점에서 더 깊이 이해할 수 있다. 칼만 필터의 업데이트 단계는 베이즈 정리(Bayes' rule)에 기반하여 사후 확률(a posteriori probability)을 최대화하는 상태를 찾는 과정으로 볼 수 있다. 가우시안 가정 하에서 이는 다음과 같은 비선형 최소제곱(Nonlinear Least Squares, NLS) 비용 함수 $J(x_k)$를 최소화하는 문제와 등가이다.13
$$
J(x_k) = (x_k - \hat{x}_k^-)^T (P_k^-)^{-1} (x_k - \hat{x}_k^-) + (z_k - h(x_k))^T R_k^{-1} (z_k - h(x_k))
$$
여기서 첫 번째 항은 사전 추정치($\hat{x}_k^-$)로부터의 편차를, 두 번째 항은 측정 잔차(innovation)를 나타낸다. EKF의 업데이트 단계는 이 비용 함수 $J(x_k)$를 최소화하는 해를 찾기 위해 가우스-뉴턴 방법을 *단 한 번의 스텝*만으로 근사하는 것과 수학적으로 동일하다.2

반면, iEKF는 이 가우스-뉴턴 방법을 여러 번 반복 적용하여 비용 함수를 최소화하는 해, 즉 최대 사후 확률(Maximum a Posteriori, MAP) 추정치에 점근적으로 수렴하려는 시도이다.13 매 반복마다 상태 추정치를 업데이트하고 그 지점에서 다시 선형화하는 과정은, 가우스-뉴턴 알고리즘이 현재 해에서 함수의 기울기(자코비안)를 구해 다음 해를 찾는 과정과 정확히 일치한다. 따라서 iEKF는 EKF를 비선형 최소제곱 문제의 해법으로 확장한 자연스러운 형태로 볼 수 있으며, 이 때문에 최적점에 수렴하기 위해 일반적으로 반복이 필요하다.13

이러한 관점은 iEKF가 필터의 전체 구조를 바꾸는 것이 아님을 명확히 한다. 예측(Prediction) 단계는 EKF와 완전히 동일하며, iEKF의 혁신은 오직 측정치 업데이트(Update) 단계에 국한된다. 이는 iEKF가 '전역적' 최적해를 찾는 것이 아니라, 주어진 사전 정보($\hat{x}_k^-$, $P_k^-$)와 새로운 측정값($z_k$) 하에서 *국소적으로* 더 나은 사후 추정치($\hat{x}_k^+$)를 찾는 데 집중하는 '국소적 최적화기'임을 의미한다.14 이 국소적 최적화는 계산 비용 증가를 동반하지만 7, 예측 단계의 모델링 오차나 과거의 누적된 오차를 수정하는 것이 아니라, 현재 시점의 측정 정보를 최대한 활용하여 선형화 오차를 줄이는 데 그 목적이 있다. 따라서 iEKF는 시스템 모델($f(x)$)의 비선형성보다 측정 모델($h(x)$)의 비선형성이 강할 때 더 큰 효과를 발휘한다. 또한, 높은 신호 대 잡음비(SNR), 즉 측정 잡음 공분산($R_k$)이 작을수록 iEKF의 성능 향상 효과가 더욱 커진다.13 이는 잡음이 적은 측정값을 더 신뢰할 수 있으므로, 이 측정값에 부합하도록 상태를 정밀하게 조정하는 반복 과정이 더 큰 의미를 갖기 때문이다.

## 2.  iEKF 알고리즘의 수학적 유도 및 분석

iEKF 알고리즘을 이해하기 위해 먼저 이산 시간 비선형 시스템 모델을 정의하고, 예측 단계와 반복적인 업데이트 단계를 수학적으로 기술한다.

### 2.1  시스템 모델

iEKF가 다루는 시스템은 다음과 같은 일반적인 비선형 상태 공간 모델로 표현된다.

- **상태 방정식 (State Equation):**
  $$
  x_k = f(x_{k-1}, u_{k-1}) + w_{k-1}
  $$
  **측정 방정식 (Measurement Equation):**
  $$
  z_k = h(x_k) + v_k
  $$
  여기서 각 변수는 다음과 같이 정의된다.15

- $x_k \in \mathbb{R}^n$: 시간 $k$에서의 상태 벡터
- $u_k \in \mathbb{R}^p$: 시간 $k$에서의 제어 입력 벡터
- $z_k \in \mathbb{R}^m$: 시간 $k$에서의 측정 벡터
- $f(\cdot)$: 상태 전이(state transition)를 나타내는 비선형 함수
- $h(\cdot)$: 상태를 측정값으로 변환하는 비선형 함수
- $w_{k-1}$: 평균이 0이고 공분산이 $Q_k$인 프로세스 잡음, 즉 $w_{k-1} \sim N(0, Q_k)$
- $v_k$: 평균이 0이고 공분산이 $R_k$인 측정 잡음, 즉 $v_k \sim N(0, R_k)$

### 2.2  예측 단계

예측 단계는 EKF와 완전히 동일하다. 시간 $k-1$에서의 사후 추정치($\hat{x}_{k-1}^+$, $P_{k-1}^+$)를 사용하여 시간 $k$에서의 사전 추정치($\hat{x}_k^-$, $P_k^-$)를 계산한다.

- **사전 상태 추정 (A Priori State Estimate):**
  $$
  \hat{x}_k^- = f(\hat{x}_{k-1}^+, u_{k-1})
  $$
  비선형 상태 전이 함수 $f$에 이전 상태 추정치를 직접 대입하여 다음 상태를 예측한다.

- **사전 오차 공분산 (A Priori Error Covariance):**
  $$
  P_k^- = F_{k-1} P_{k-1}^+ F_{k-1}^T + Q_{k-1}
  $$
  오차 공분산은 직접 전파할 수 없으므로, 상태 전이 함수를 선형화한 자코비안 행렬 $F_{k-1}$을 사용하여 전파한다. $F_{k-1}$은 다음과 같이 정의된다.7
  $$
  F_{k-1} = \frac{\partial f}{\partial x} \bigg|_{\hat{x}_{k-1}^+, u_{k-1}}
  $$

### 2.3  측정치 업데이트 단계 (Measurement Update Step) - 반복 루프

이 단계가 iEKF의 핵심이며, EKF와 차별화되는 지점이다. 새로운 측정치 $z_k$를 사용하여 사전 추정치를 보정하는 과정을 최적의 선형화 지점을 찾기 위해 $N$회 반복한다.

- 반복 초기화 (Initialization):

  반복 루프를 시작하기 위해, 첫 번째 반복($i=0$)의 상태 추정치를 사전 추정치로 설정한다.2
  $$
  \hat{x}_{k,0} = \hat{x}_k^-
  $$

- 반복 루프 (for $i=0$ to $N-1$):

  미리 설정된 최대 반복 횟수 $N$ 또는 수렴 조건이 만족될 때까지 다음 과정을 반복한다.

  1. 측정 자코비안 재계산 (Re-calculate Measurement Jacobian):

     매 반복마다 가장 최근의 상태 추정치 $\hat{x}_{k,i}$를 기준으로 측정 함수 $h(x)$를 다시 선형화한다. 이는 더 나은 선형화 지점을 찾으려는 iEKF의 핵심 철학을 반영한다.2
     $$
     H_{k,i} = \frac{\partial h}{\partial x} \bigg|_{x=\hat{x}_{k,i}}
     $$

  2. 칼만 이득 계산 (Calculate Kalman Gain):

     새롭게 계산된 자코비안 $H_{k,i}$를 사용하여 칼만 이득 $K_{k,i}$를 계산한다. 여기서 매우 중요한 점은 오차 공분산으로 사전 공분산 $P_k^-$를 계속 사용한다는 것이다. 만약 이득 계산 시 공분산까지 매번 업데이트하면, 필터는 동일한 측정치를 여러 번 독립적으로 받은 것으로 착각하여 공분산을 과도하게 줄이는(overly confident) 결과를 초래한다. 이는 필터의 성능을 저하시키는 심각한 오류이다.2
     $$
     K_{k,i} = P_k^- H_{k,i}^T (H_{k,i} P_k^- H_{k,i}^T + R_k)^{-1}
     $$

  3. 상태 벡터 업데이트 (Update State Vector):

     계산된 칼만 이득을 사용하여 다음 반복을 위한 상태 추정치 $\hat{x}_{k,i+1}$을 계산한다. 가장 일반적이고 올바른 상태 업데이트 수식은 다음과 같다.2
     $$
     \hat{x}_{k,i+1} = \hat{x}_k^- + K_{k,i} (z_k - h(\hat{x}_{k,i}) - H_{k,i}(\hat{x}_k^- - \hat{x}_{k,i}))
     $$
     이 수식은 사전 추정치 $\hat{x}_k^-$를 기준으로, 현재 추정치 $\hat{x}_{k,i}$에서의 예측 측정치 $h(\hat{x}_{k,i})$와 실제 측정치 $z_k$ 사이의 차이를 보정한다. 마지막 항 $H_{k,i}(\hat{x}_k^- - \hat{x}_{k,i})$는 선형화 지점이 $\hat{x}_k^-$에서 $\hat{x}_{k,i}$로 변경됨에 따른 보상 항으로, 업데이트의 정확도를 높이는 역할을 한다.

- 반복 종료 조건 (Termination Condition):

  다음 두 조건 중 하나가 만족되면 반복 루프를 종료한다.

  - 미리 정해진 최대 반복 횟수 $N$에 도달한 경우.
  - 연속된 상태 추정치의 변화량이 특정 임계값 $\epsilon$보다 작아진 경우: $\|\hat{x}_{k,i+1} - \hat{x}_{k,i}\| < \epsilon$.14

### 2.4  최종 업데이트

반복 루프가 종료된 후, 최종 상태 추정치와 공분산을 확정한다.

- 사후 상태 추정 (A Posteriori State Estimate):

  반복 루프의 마지막 결과가 최종 사후 상태 추정치가 된다.
  $$
  \hat{x}_k^+ = \hat{x}_{k,N}
  $$
  (만약 임계값 조건으로 조기 종료되었다면, 그 시점의 마지막 추정치가 된다.)

- 사후 오차 공분산 (A Posteriori Error Covariance):

  공분산은 반복 루프가 모두 끝난 후, 마지막으로 계산된 칼만 이득 $K_{k,N-1}$과 자코비안 $H_{k,N-1}$을 사용하여 단 한 번만 업데이트한다.2
  $$
  P_k^+ = (I - K_{k,N-1} H_{k,N-1}) P_k^-
  $$

#### 2.4.1 표 1: iEKF 알고리즘 의사코드

이론적 수식을 실제 구현 가능한 알고리즘으로 명확하게 변환하면 다음과 같다. 이 의사코드는 상태 추정치는 반복적으로 업데이트되지만 공분산은 사전 값을 계속 사용하다가 마지막에 한 번만 업데이트된다는 핵심적인 구현 디테일을 명시하여, 흔히 발생하는 구현 오류를 방지하는 데 도움을 준다.2

| 단계                 | 연산                                                         |
| -------------------- | ------------------------------------------------------------ |
| **입력**             | $\hat{x}_{k-1}^+, P_{k-1}^+, u_{k-1}, z_k, Q_k, R_k, N, \epsilon$ |
| **출력**             | $\hat{x}_k^+, P_k^+$                                         |
| **1. 예측**          | $\hat{x}_k^- = f(\hat{x}_{k-1}^+, u_{k-1})$                  |
|                      | 마크다운 표 형식과 충돌 위 수식 참조                         |
|                      | $P_k^- = F_{k-1} P_{k-1}^+ F_{k-1}^T + Q_{k-1}$              |
| **2. 반복 업데이트** | $\hat{x}_{k,0} = \hat{x}_k^-$                                |
|                      | **for** $i = 0$ **to** $N-1$:                                |
|                      | 마크다운 표 형식과 충돌 위 수식 참조                         |
|                      | $K_{k,i} = P_k^- H_{k,i}^T (H_{k,i} P_k^- H_{k,i}^T + R_k)^{-1}$ |
|                      | $\hat{x}_{k,i+1} = \hat{x}_k^- + K_{k,i} (z_k - h(\hat{x}_{k,i}) - H_{k,i}(\hat{x}_k^- - \hat{x}_{k,i}))$ |
|                      | **if** $|\hat{x}_{k,i+1} - \hat{x}_{k,i}| < \epsilon$ **then break** |
|                      | **end for**                                                  |
| **3. 최종 확정**     | $\hat{x}_k^+ = \hat{x}_{k,i+1}$                              |
|                      | $P_k^+ = (I - K_{k,i} H_{k,i}) P_k^-$                        |

## 3.  성능 및 특성 분석

iEKF는 EKF의 한계를 극복하기 위해 제안되었지만, 그 자체로도 고려해야 할 성능적 특성과 트레이드오프가 존재한다.

### 3.1  정확도

iEKF의 가장 큰 장점은 EKF 대비 향상된 정확도이다. 반복적인 선형화 과정을 통해 측정치에 더 부합하는 상태 추정치를 찾기 때문에, 특히 측정 모델의 비선형성이 강한 시스템에서 EKF보다 훨씬 낮은 평균 제곱근 오차(RMSE)를 보인다.9 이러한 성능 향상은 신호 대 잡음비(SNR)가 높을수록, 즉 측정 잡음(

$R_k$)이 작고 측정치를 더 신뢰할 수 있는 상황에서 더욱 두드러진다.13 측정치가 더 많은 정보를 담고 있을 때, 그 정보를 최대한 활용하여 상태를 정밀하게 조정하는 반복 과정이 더 큰 효과를 발휘하기 때문이다.

### 3.2  계산 복잡도

정확도 향상은 계산 비용 증가라는 대가를 치른다. iEKF는 EKF의 업데이트 단계를 $N$번 반복하므로, EKF에 비해 약 $N$배의 연산량이 추가로 필요하다.7 각 반복 과정에는 자코비안 행렬 계산, 여러 번의 행렬 곱셈, 그리고 

$(m \times m)$ 크기 행렬의 역행렬 계산이 포함되어 있어 상당한 계산 부하를 유발한다. 따라서 실시간성이 매우 중요한 시스템(예: 고속으로 움직이는 로봇 제어)에 iEKF를 적용할 때는 반복 횟수 $N$을 신중하게 선택해야 한다. $N$은 정확도와 계산 시간 사이의 트레이드오프 관계를 결정하는 핵심적인 튜닝 파라미터이며, 일반적으로 2~3회 정도의 적은 반복만으로도 상당한 성능 개선을 얻을 수 있는 것으로 알려져 있다.2

### 3.3  수렴성 및 안정성

iEKF는 가우스-뉴턴 최적화에 기반하기 때문에 항상 최적의 해로 수렴한다고 보장할 수 없다. 만약 초기 추정치(사전 추정치 $\hat{x}_k^-$)가 실제 값에서 너무 멀리 떨어져 있거나, 측정 함수의 비선형성이 매우 심한 경우, 반복 과정이 수렴하지 않거나 잘못된 지역 최적해(local minimum)로 수렴할 수 있다.2 또한, EKF와 마찬가지로 시스템 모델링 오차가 크거나 잡음 특성이 가정과 다를 경우 필터가 발산할 위험을 여전히 내포하고 있다.11

이러한 수렴성 문제는 iEKF가 만병통치약이 아님을 시사한다. EKF가 대략적인 방향을 제시한다면, iEKF는 그 방향에서 더 정밀하게 조준하는 '정밀 조준경'과 같다. 하지만 EKF가 초기에 완전히 엉뚱한 방향을 가리키고 있다면(즉, 사전 추정치가 너무 나쁘면), iEKF가 아무리 정밀하게 조준을 반복해도 올바른 목표물을 맞출 수 없다. 오히려 잘못된 목표에 더 강한 확신을 갖고 수렴할 위험도 존재한다.19 이는 iEKF가 EKF의 '대체재'라기보다는 '개선판' 혹은 '보완재'임을 의미한다. 필터의 전반적인 성능은 여전히 양질의 동적 모델, 적절한 잡음 공분산($Q, R$) 설정, 그리고 합리적인 초기 추정치에 크게 의존한다. 따라서 iEKF를 성공적으로 적용하기 위해서는 반복 횟수($N$)라는 새로운 튜닝 파라미터를 조절하는 것 외에도, 시스템에 대한 깊은 이해가 반드시 선행되어야 한다.

## 4.  주요 비선형 필터와의 비교 분석

iEKF의 실용적 가치를 평가하기 위해서는 다른 주요 비선형 필터들과의 비교 분석이 필수적이다.

### 4.1  iEKF vs. EKF

iEKF와 EKF의 관계는 명확하다. iEKF는 EKF의 직접적인 확장판이다.

- **핵심 차이:** 가장 큰 차이는 측정치 업데이트 단계의 반복 수행 여부이다. EKF는 단 한 번의 업데이트를 수행하는 반면, iEKF는 더 나은 선형화 지점을 찾기 위해 이 과정을 여러 번 반복한다.
- **장점 (iEKF > EKF):** 강한 측정 비선형성에 대한 강건성과 이로 인한 높은 추정 정확도를 제공한다.13
- **단점 (iEKF > EKF):** 반복으로 인한 계산 비용 증가와, 반복 횟수($N$)라는 추가적인 튜닝 파라미터가 생긴다는 점이다.7
- **선택 가이드:** 실시간 제약이 매우 엄격하고 시스템의 비선형성이 약하다면 EKF가 합리적인 선택이다. 반면, 계산 자원이 충분하고 문제의 핵심이 측정 모델의 비선형성에서 비롯된다면 iEKF가 훨씬 우수한 성능을 제공할 것이다.

### 4.2  iEKF vs. UKF (Unscented Kalman Filter)

무향 칼만 필터(Unscented Kalman Filter, UKF)는 EKF와는 근본적으로 다른 방식으로 비선형성에 접근하며, iEKF의 강력한 경쟁자이다.

- **접근 방식의 근본적 차이:**
  - **iEKF:** 비선형 *함수*를 반복적으로 선형 근사(자코비안)하여 가우시안 분포를 전파한다.
  - **UKF:** 비선형 함수 자체는 그대로 두고, 가우시안 *분포*를 대표하는 소수의 결정론적 샘플(시그마 포인트, sigma points)을 추출한다. 이 시그마 포인트들을 실제 비선형 함수에 직접 통과시킨 후, 변환된 포인트들의 통계(가중 평균, 가중 공분산)를 계산하여 새로운 가우시안 분포를 근사한다.7 즉, '함수를 근사하는 것보다 분포를 근사하는 것이 더 쉽다'는 철학에 기반한다.20
- **자코비안 계산:** iEKF는 매 반복마다 자코비안을 계산해야 하지만, UKF는 자코비안 계산이 전혀 필요 없다. 이는 UKF를 미분 불가능한 함수에도 적용할 수 있게 만드는 큰 장점이다.22
- **성능 비교:** 일반적으로 UKF는 2차 테일러 급수까지의 정확도를 보장하므로, 1차 근사인 EKF보다 강한 비선형성에서 더 정확하고 강건한 성능을 보인다.7 iEKF는 측정 모델의 비선형성이 특히 강한 경우 UKF와 필적하거나 더 나은 성능을 보일 수 있지만, 이는 문제에 따라 다르다. 계산 복잡도는 상태 벡터의 차원($n$)에 따라 달라진다. EKF/iEKF의 계산량은 자코비안 계산 비용에 크게 좌우되는 반면, UKF의 계산량은 시그마 포인트의 개수($2n+1$)와 이들을 전파하는 비용에 지배된다. 따라서 상태 차원이 매우 높은 시스템에서는 UKF의 계산량이 급격히 증가하여 실시간 적용이 어려울 수 있다.24
- **실용적 관점:** UKF는 자코비안이 필요 없어 구현이 비교적 간단하고, 튜닝 파라미터가 적고 직관적이라는 장점이 있다. 반면, iEKF는 기존에 구현된 EKF 코드베이스가 있다면 측정치 업데이트 부분만 수정하여 비교적 쉽게 구현할 수 있다는 장점이 있다.23

### 4.3  iEKF vs. IUKF (Iterated Unscented Kalman Filter)

반복 무향 칼만 필터(Iterated Unscented Kalman Filter, IUKF)는 UKF의 업데이트 단계에 iEKF와 유사한 반복 최적화 개념을 적용한 필터이다.13 이는 '반복'이라는 개선 전략이 EKF에만 국한된 것이 아니라, 비선형 필터를 개선하는 일반적인 원리임을 보여준다. IUKF는 자코비안 계산 없이 반복 최적화의 이점을 얻으려는 시도로, iEKF의 미분-불필요(derivative-free) 대안으로 볼 수 있다.13 연구에 따르면 IUKF는 UKF나 iEKF보다 더 나은 성능을 보이는 경우도 있으며, 잡음에 더 강건한 특성을 나타낸다.16

#### 4.3.1 표 2: 비선형 칼만 필터 비교

각 필터의 핵심적인 차이점과 장단점을 요약하면 다음과 같다. 이 표는 사용자가 자신의 문제 특성에 가장 적합한 필터를 선택하는 데 유용한 가이드라인을 제공한다.

| 특징 (Feature)             | EKF                    | iEKF                        | UKF                           |
| -------------------------- | ---------------------- | --------------------------- | ----------------------------- |
| **기본 원리**              | 1차 테일러 급수 선형화 | 반복적 선형화 (가우스-뉴턴) | 무향 변환 (결정론적 샘플링)   |
| **선형화 대상**            | 비선형 함수            | 비선형 함수 (반복)          | 확률 분포                     |
| **자코비안 필요**          | 예                     | 예 (반복 계산)              | 아니오                        |
| **정확도 (강한 비선형성)** | 낮음                   | 중간-높음                   | 높음                          |
| **계산 복잡도**            | 낮음                   | 중간-높음 ($N$에 비례)      | 중간-높음 (차원 $n$에 비례)   |
| **주요 장점**              | 빠르고 간단함          | 측정 비선형성에 강함        | 자코비안 불필요, 고차 근사    |
| **주요 단점**              | 선형화 오차, 발산 위험 | 계산량, 수렴성 문제         | 고차원 시스템에서 계산량 증가 |

## 5.  응용 분야 및 확장

iEKF는 EKF의 한계가 드러나는 다양한 실제 비선형 시스템에서 그 가치를 입증하고 있다.

### 5.1  항법 시스템

자율주행차, 드론, 로봇 등의 항법 시스템은 iEKF의 주요 응용 분야이다.

- **관성항법/GPS 통합 (INS/GPS Integration):** 저가형 MEMS 관성 측정 장치(IMU)와 GPS를 융합하는 시스템에서 센서의 비선형적 오차 특성과 외부 환경의 잡음을 효과적으로 처리하기 위해 iEKF가 EKF의 대안으로 활발히 연구되고 있다. 특히 측정 모델의 비선형성이 강한 경우, iEKF는 EKF보다 월등한 위치 추정 정확도를 제공한다.18
- **시각-관성 주행 거리 측정 (Visual-Inertial Odometry, VIO):** 카메라와 IMU 센서를 융합하여 이동체의 위치와 자세를 추정하는 VIO 기술은 자율주행 및 증강현실의 핵심 기술이다. 카메라의 투영 모델은 강한 비선형성을 가지므로, iEKF의 반복적인 업데이트 과정이 선형화 오차를 크게 줄여 추정 정확도를 향상시키는 데 기여한다.27
- **적응형 iEKF (Adaptive iEKF, AIEKF):** 실제 환경에서는 잡음의 통계적 특성이 시간에 따라 변하는 경우가 많다. AIEKF는 잡음 공분산 행렬($Q, R$)을 실시간으로 추정하는 기법을 iEKF에 결합하여, 동적으로 변하는 환경에 더 강건하게 대응하려는 시도이다. 연구 결과에 따르면 AIEKF는 표준 iEKF보다도 더 높은 정확도를 달성할 수 있다.18

### 5.2  SLAM (Simultaneous Localization and Mapping)

SLAM은 로봇이 미지의 환경을 탐험하면서 자신의 위치를 추정하고 동시에 주변 환경의 지도를 작성하는 문제이다. EKF-SLAM은 이 문제에 대한 고전적인 해법이지만, EKF의 내재적인 문제로 인해 일관성(consistency)과 안정성에 취약하다.28 iEKF는 EKF-SLAM의 측정치 업데이트 단계를 개선하여 더 정확한 랜드마크 위치 추정과 로봇 자세 추정을 가능하게 함으로써 SLAM 성능을 향상시키는 데 사용될 수 있다.30

하지만 iEKF가 SLAM의 근본적인 문제를 해결하는 것은 아니다. SLAM에서 EKF의 핵심적인 실패 원인 중 하나는 관측되지 않는 방향(unobservable directions, 예: 전체 맵의 절대 위치 및 방향)에 대해 필터가 잘못된 확신을 갖게 되는 '일관성' 문제이다.29 이는 선형화 지점이 로봇의 추정된 궤적에 따라 계속 바뀌면서 발생하는 구조적인 문제이다. iEKF는 주어진 측정값에 대해 선형화 지점을 '개선'할 뿐, 이 구조적인 문제를 근본적으로 해결하지는 못한다. SLAM 문제에 대한 더 근본적인 해결책은 필터의 구조 자체를 시스템의 기하학적 대칭성(symmetry)에 맞게 설계하는 **불변 확장 칼만 필터(Invariant EKF, IEKF)**에서 찾아볼 수 있다. 불변 EKF는 상태 오차를 리 군(Lie Group) 상에서 정의하여 선형화 지점 의존성을 줄임으로써 일관성 문제를 해결하고, EKF보다 훨씬 강건한 수렴성을 보장한다.28 이는 iEKF가 EKF의 점진적 개선이라면, 불변 EKF는 패러다임의 전환에 가깝다는 것을 시사한다.

### 5.3  위성 자세 결정

인공위성의 자세(attitude)는 3차원 공간에서의 방향을 의미하며, 쿼터니언(quaternion)이나 회전 행렬 등으로 표현된다. 위성에 장착된 센서(태양 센서, 자력계, 자이로스코프 등)의 측정 모델은 종종 비선형적이다. 이 분야에서는 쿼터니언의 단위 길이 제약을 효과적으로 다루기 위해 EKF를 변형한 곱셈 확장 칼만 필터(Multiplicative EKF, MEKF)가 널리 사용된다. 여기에 iEKF의 개념을 적용한 **반복 곱셈 확장 칼만 필터(Iterated MEKF, IMEKF)**가 제안되었으며, 적은 추가 계산량으로 MEKF보다 훨씬 향상된 자세 추정 정확도를 달성함이 입증되었다.33 이는 iEKF의 '반복을 통한 개선'이라는 핵심 철학이 다양한 형태의 칼만 필터에 성공적으로 적용될 수 있는 일반적인 원리임을 보여준다.

### 5.4  고급 변형 필터

iEKF의 개념은 더 정교하고 강력한 필터의 개발로 이어지고 있다.

- **불변 EKF (Invariant EKF, IEKF):** 앞서 언급했듯이, 시스템의 기하학적 구조(리 군 대칭성)를 보존하도록 설계된 필터이다. 이 필터는 추정 오차 방정식이 실제 상태 궤적에 의존하지 않게 만들어, EKF가 발산하는 도전적인 상황에서도 안정적인 수렴을 보장한다. SLAM과 관성 항법 문제에서 EKF보다 이론적으로 우월한 특성을 가진다.12
- **반복 불변 EKF (Iterated IEKF, IterIEKF):** 불변 EKF의 업데이트 단계에 iEKF와 같은 가우스-뉴턴 반복을 적용한 최신 연구이다. 이는 불변 EKF의 강건한 수렴성과 iEKF의 높은 정확도를 결합하려는 시도로, 잡음이 적은 환경에서 선형 칼만 필터와 유사한 강력한 이론적 특성을 보이는 것으로 나타났다.10 이는 비선형 필터링 분야의 최전선에 있는 연구 방향 중 하나이다.

## 6.  결론: iEKF의 실용적 가치와 전망

### 7.1. iEKF의 핵심 가치 요약

반복 확장 칼만 필터(iEKF)는 EKF의 가장 큰 약점인 선형화 오차를 보완하기 위한 강력하고 직관적인 개선책이다. EKF가 단 한 번의 선형화로 상태를 추정하는 반면, iEKF는 새로 들어온 측정 정보를 바탕으로 최적의 선형화 지점을 반복적으로 탐색함으로써 추정의 정확도를 크게 향상시킨다. 이는 EKF와, 자코비안 계산이 필요 없는 UKF와 같은 고성능 필터 사이에서 정확도와 계산량의 균형을 맞추는 유용한 선택지를 제공한다. 특히 측정 모델의 비선형성이 강하고 측정 잡음이 비교적 적은 문제에서 iEKF의 가치는 극대화된다.

### 6.1  적용 시 고려사항 및 필터 선택 가이드라인

iEKF를 실제 문제에 적용하거나 다른 필터와 비교하여 선택할 때는 다음 사항들을 종합적으로 고려해야 한다.

- **문제의 비선형성:** 시스템의 비선형성이 주로 시스템 모델($f(x)$)에서 기인하는가, 아니면 측정 모델($h(x)$)에서 기인하는가? iEKF는 특히 $h(x)$의 비선형성이 강할 때 효과적이다. 양쪽 모두 비선형성이 강하다면 UKF가 더 나은 선택일 수 있다.
- **자코비안 계산 용이성:** 비선형 함수들의 자코비안을 해석적으로 쉽게 구할 수 있는가? 만약 자코비안 계산이 매우 복잡하거나 불가능하다면, 미분이 필요 없는 UKF가 훨씬 매력적인 대안이 될 것이다.
- **계산 자원 및 실시간 요구사항:** 임베디드 시스템과 같이 계산 자원이 제한적인 환경에서는 EKF나 반복 횟수를 1~2회로 최소화한 iEKF가 현실적인 해법일 수 있다.23 계산 자원이 풍부하다면 더 많은 반복을 통해 정확도를 높이거나 UKF, IUKF 등을 고려할 수 있다.
- **상태 공간의 기하학적 구조:** 추정하려는 상태가 회전 행렬($SO(3)$)이나 쿼터니언과 같이 특수한 기하학적 구조(리 군, Lie Group)를 가지는가? 그렇다면 iEKF보다 이러한 구조를 명시적으로 다루는 불변 EKF(Invariant EKF)가 이론적으로 더 강건하고 일관된 해법을 제공할 가능성이 높다.

### 6.2  향후 연구 방향

iEKF와 관련된 연구는 여전히 활발히 진행 중이며, 다음과 같은 방향으로 발전할 것으로 전망된다.

- **이론적 분석 강화:** iEKF의 수렴 조건과 안정성에 대한 더 엄밀한 이론적 분석을 통해 필터의 행동을 예측하고 보장하는 연구가 필요하다.
- **적응형 및 하이브리드 필터:** 반복 횟수나 잡음 공분산과 같은 파라미터를 실시간으로 조절하는 적응형 iEKF 연구가 계속될 것이다. 또한, 불변 EKF의 강건함과 iEKF의 정확도를 결합한 반복 불변 EKF(IterIEKF)와 같은 하이브리드 필터의 개발은 비선형 추정 이론의 중요한 흐름이 될 것이다.10
- **계산 효율성 증대:** SLAM과 같이 상태 벡터의 차원이 매우 커지는 문제에서 iEKF의 계산 효율성을 높이기 위한 희소 행렬(sparse matrix) 기법 활용이나 알고리즘 변형에 대한 연구가 요구된다.

결론적으로, iEKF는 비선형 상태 추정 문제에 대한 실용적이고 효과적인 도구이다. EKF의 한계를 명확히 인식하고, UKF 및 불변 EKF와 같은 다른 대안들과의 장단점을 정확히 이해하며, 주어진 문제의 특성에 맞게 필터를 신중하게 선택하고 튜닝하는 것이 성공적인 적용의 핵심이라 할 수 있다.

#### **참고 자료**

1. 칼만 필터 - 위키백과, 우리 모두의 백과사전, accessed July 23, 2025, [https://ko.wikipedia.org/wiki/%EC%B9%BC%EB%A7%8C_%ED%95%84%ED%84%B0](https://ko.wikipedia.org/wiki/칼만_필터)
2. AAS 14-345 ADAPTABLE ITERATIVE AND RECURSIVE KALMAN ..., accessed July 23, 2025, https://sites.utexas.edu/near/files/2022/05/recursiveUpdate.pdf
3. Kalman and Extended Kalman Filters: Concept, Derivation and Properties - CiteSeerX, accessed July 23, 2025, https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=b689ad56d935ceb053e24bf4de26f1466b911263
4. [SLAM] 칼만 필터(Kalman Filter)란? - Jbground - 티스토리, accessed July 23, 2025, https://jbground.tistory.com/82
5. [SLAM] Kalman filter and EKF(Extended Kalman Filter) - Jinyong, accessed July 23, 2025, http://jinyongjeong.github.io/2017/02/14/lec03_kalman_filter_and_EKF/
6. Slam 7강 (Kalman Filter & EKF) 요약 - GitHub, accessed July 23, 2025, https://taeyoung96.github.io/slam/SLAM_07/
7. Extended Kalman filter - Wikipedia, accessed July 23, 2025, https://en.wikipedia.org/wiki/Extended_Kalman_filter
8. [제어시스템공학-10] Extended Kalman Filter(확장칼만필터) 개념 - Engineering insight, accessed July 23, 2025, https://limitsinx.tistory.com/78
9. [논문 리뷰] Notes on Kalman Filter (KF, EKF, ESKF, IEKF, IESKF) - Moonlight, accessed July 23, 2025, https://www.themoonlight.io/ko/review/notes-on-kalman-filter-kf-ekf-eskf-iekf-ieskf
10. Iterated Invariant Extended Kalman Filter (IterIEKF) - arXiv, accessed July 23, 2025, https://arxiv.org/pdf/2404.10665
11. [칼만 필터는 어렵지 않아] 12 확장 칼만 필터 - velog, accessed July 23, 2025, [https://velog.io/@zajangbumbuck/%EC%B9%BC%EB%A7%8C-%ED%95%84%ED%84%B0%EB%8A%94-%EC%96%B4%EB%A0%B5%EC%A7%80-%EC%95%8A%EC%95%84-12-%ED%99%95%EC%9E%A5-%EC%B9%BC%EB%A7%8C-%ED%95%84%ED%84%B0](https://velog.io/@zajangbumbuck/칼만-필터는-어렵지-않아-12-확장-칼만-필터)
12. The Invariant Extended Kalman filter as a stable observer arXiv:1410.1465v4 [cs.SY] 19 Oct 2015, accessed July 23, 2025, https://arxiv.org/pdf/1410.1465
13. On Iterative Unscented Kalman Filter using Optimization - DiVA portal, accessed July 23, 2025, https://www.diva-portal.org/smash/get/diva2:1335682/FULLTEXT01.pdf
14. Iterated Extended Kalman Filter, accessed July 23, 2025, http://www-sop.inria.fr/odyssee/software/old_robotvis/Tutorial-Estim/node18.html
15. (PDF) A Quick Guide for the Iterated Extended Kalman Filter on ..., accessed July 23, 2025, https://www.researchgate.net/publication/372445074_A_Quick_Guide_for_the_Iterated_Extended_Kalman_Filter_on_Manifolds
16. application of kalman filtering methods to online real-time structural identification: a comparison study, accessed July 23, 2025, https://opus.lib.uts.edu.au/rest/bitstreams/212d20eb-1c75-4ea1-b4e7-de4646df9e9c/retrieve
17. 칼만 필터(Kalman Filter) 개념 정리 (+ KF, EKF, ESKF, IEKF, IESKF) Part 1 - A L I D A, accessed July 23, 2025, https://alida.tistory.com/54
18. Adaptive Iterated Extended Kalman Filter and Its Application to Autonomous Integrated Navigation for Indoor Robot - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/261295145_Adaptive_Iterated_Extended_Kalman_Filter_and_Its_Application_to_Autonomous_Integrated_Navigation_for_Indoor_Robot/download
19. Extended Kalman filter converging to incorrect value, is this just the nature of the beast? (pics), accessed July 23, 2025, https://stackoverflow.com/questions/15360248/extended-kalman-filter-converging-to-incorrect-value-is-this-just-the-nature-of
20. Unscented Kalman Filter Tutorial - ResearchGate, accessed July 23, 2025, [https://www.researchgate.net/profile/Mohamed-Mourad-Lafifi/post/Unscented-Kalman-Filter/attachment/5a3401d34cde266d587b4efd/AS%3A571878130622464%401513357779148/download/Tutorial+UKF.pdf](https://www.researchgate.net/profile/Mohamed-Mourad-Lafifi/post/Unscented-Kalman-Filter/attachment/5a3401d34cde266d587b4efd/AS%3A571878130622464@1513357779148/download/Tutorial+UKF.pdf)
21. A Comparison of Unscented and Extended Kalman Filtering for Nonlinear System Identification - UBC Library Open Collections, accessed July 23, 2025, https://open.library.ubc.ca/media/download/pdf/53032/1.0076081/1
22. What is the difference between EKF and UKF? - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/post/What_is_the_difference_between_EKF_and_UKF
23. Why should I still use EKF instead of UKF? - Robotics Stack Exchange, accessed July 23, 2025, https://robotics.stackexchange.com/questions/3063/why-should-i-still-use-ekf-instead-of-ukf
24. Unscented KF vs EKF - General - ArduPilot Discourse, accessed July 23, 2025, https://discuss.ardupilot.org/t/unscented-kf-vs-ekf/27052
25. On the Complexity and Consistency of UKF-based SLAM - People, accessed July 23, 2025, https://people.csail.mit.edu/ghuang/paper/Huang2009ICRA.pdf
26. US20100036613A1 - Methods and systems for implementing an iterated extended kalman filter within a navigation system - Google Patents, accessed July 23, 2025, https://patents.google.com/patent/US20100036613A1/en
27. Schur Complement Optimized Iterative EKF for Visual–Inertial Odometry in Autonomous Vehicles - MDPI, accessed July 23, 2025, https://www.mdpi.com/2075-1702/13/7/582
28. Fixing the consistency issues of the extended Kalman filter for simultaneous localization and mapping (SLAM), accessed July 23, 2025, http://www.silvere-bonnabel.com/pdf/CONSISTENT_EKF_SLAM.pdf
29. Decoupled Right Invariant Error States for Consistent Visual-Inertial Navigation, accessed July 23, 2025, https://par.nsf.gov/servlets/purl/10376061
30. Application of a modified IEKF algorithm in mobile robot localization - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/370149632_Application_of_a_modified_IEKF_algorithm_in_mobile_robot_localization
31. [2406.06427] Notes on Kalman Filter (KF, EKF, ESKF, IEKF, IESKF) - arXiv, accessed July 23, 2025, https://arxiv.org/abs/2406.06427
32. Invariant extended Kalman filter - Wikipedia, accessed July 23, 2025, https://en.wikipedia.org/wiki/Invariant_extended_Kalman_filter
33. Iterated multiplicative extended Kalman filter for attitude estimation using vector observations | Request PDF - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/310467318_Iterated_multiplicative_extended_Kalman_filter_for_attitude_estimation_using_vector_observations

