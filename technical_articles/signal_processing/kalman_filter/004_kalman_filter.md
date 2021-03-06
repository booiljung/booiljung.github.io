[Up][index.md]

# 칼만필터 (Kalman filters)

## 칼만필터 처리 과정 (Process of Kalman filter)

칼만필터는 아래와 같은 과정을 거칩니다. 이 식을 지금 당장 이해하려고 하지 않아도 되는데, 다음에서 자세히 설명하도록 하겠습니다.

**0. 추정값 $\hat x$ 와 오차공분산 $P$ 에 대해 초기값 $_0$을 선정 합니다.**
$$
\begin{align}
\hat x_0, P_0
\end{align}
$$

**1. $k$번째 추정값과 오차공분산을 '예측'합니다.**
$$
\begin{align}
\breve x_k &= A \hat x_{k-1} \\\\
\breve P_k &= AP_{k-1} A^\top + Q \\
\end{align}
$$

**2. $k$번째 칼만이득(Kalman gain) $K$ 을 계산합니다.**
$$
\begin{align}
K_k = \breve P_k H ^\top (H \breve P_k H^\top + R)^{-1}
\end{align}
$$

**3. $k$번째 추정값 $\hat x$을 계산합니다.**
$$
\begin{aligned}
\hat x_k = \breve x_k + K_k (z_k - H \breve x_k)
\end{aligned}
$$

**4. $k$번째 오차 공분산 $P$를 계산합니다.**
$$
\begin{aligned}
P_k = \breve P_k - K_k H \breve P_k
\end{aligned}
$$

**5. 다시 1번으로 반복 합니다.**

## 자세한 설명

### 0. 준비

먼저 추정값과 오차공분산을 초기화 합니다.
$$
\begin{align}
\hat x_0, P_0
\end{align}
$$

### 1. 예측

#### 1.1 추정값 예측

$k-1$번째 추정값에 상수 $A$를 곱하여 $k$번째 추정값 $\breve x_k$를 '예측' 합니다.
$$
\begin{align}
\breve x_k &= A \hat x_{k-1}
\end{align}
$$

#### 1.2 오차공분산 예측

$k-1$번째 오차공분산 $P_{k-1}$으로 $k$번째 오차공분산 $\breve P_k$를 '예측'합니다.
$$
\begin{align}
\breve P_k &= A P_{k-1} A^\top + Q
\end{align}
$$
예측에서 사용되는 시스템 모델 상수는 $A$와 $Q$이며 이 상수의 값을 기초로 오차공분산이 어떤 값이 될지 예측 합니다.

### 2. 칼만이득

앞에서 예측한 오차공분산 예측값 $P_\bar k$로  $k$번째 칼만이득 $K_k$ 를 계산합니다.
$$
\begin{align}
K_k = \breve P_k H^\top (H \breve P_k H^\top + R)^{-1}
\end{align}
$$
칼만이득에서 사용되는 시스템 모델 상수는 $H$와 $R$입니다.

### 3. 추정

측정값과 예측값의 차이를 보정하여 새로운 추정값을 계산 합니다. 이 추정값이 칼만필터의 목적 결과물입니다.

앞에서 예측한 추정 예측값 $\breve x_k$에 입력된 $k$번째 측정값 $z_k$ 을 적용하여 $k$번째 추정값 $\hat x_k$를 계산합니다.
$$
\hat x_k = \breve x_k + K_k (z_k - H \breve x_k)
$$

### 4. 오차공분산

앞에서 예측한 오차공분산 $\breve P_k$에 칼만이득 $K_k$를 적용하여 $k$번째 오차 공분산 $P_k$ 계산합니다.
$$
\begin{aligned}
P_k = \breve P_k - K_k H \breve P_k
\end{aligned}
$$
오차공분산은 추정값이 얼마나 정확한지를 알려주는 척도로 사용됩니다. 보통 오차공분산을 검토해서 계산한 추정값을 믿고 쓸지 아니면 버릴지를 판단합니다.

여기서 설계자가 정하는 변상는 $A$, $H$, $Q$, $R$ 로 이것은 시스템모델과 관련된 상수이며 시스템의 성능에 영향을 줍니다.

