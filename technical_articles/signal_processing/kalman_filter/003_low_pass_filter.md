[Up](index.md)

# 칼만필터 (Kalman filters)

## 저주파 통과 필터 (Low pass filter)

저주파는 통과하고 고주파는 걸러내는 필터이며 주로 잡음제거에 사용 됩니다. 이동평균은 오래된 데이터나 최근의 데이터에 동일한 가중치를 부여하여 최근의 데이터를 반영하기 어렵습니다. 저주파 통과 필터는 이동평균과 달리 최근의 데이터의 가중치를 높게 부여 합니다. 저주파 통과 필터는 잡음제거와 변화 민감성을 동시에 만족 시킬 수 있습니다.

### 1차저주파 통과 필터 (1st order low pass filter)

저주파 통과 필터 식은 아래와 같습니다.
$$
\bar x_{k} = a \bar x_{k-1} + (1-a) x_k
$$
이며, 여기서 $a$는 $0 < a <1$ 인 임의의 상수입니다. 식을 보면 $a$가 클수록 새로 입력되는 데이터의 가중치가 낮아 집니다.

다시 평균필터를 보자면
$$
\bar x_k = a \bar x_{k-1} + (1-a)x_k
$$
매우 유사합니다. 평균필터의 $a$는 $a \equiv \frac{k-1}{k} = 1-\frac{1}{k}$ 로 데이터 갯수 $k$에 따라 정해진 값이었습니다.

1차 저주파 통과 필터를 지수 가중 이동 평균 필터(exponentially weighted moving average filter)라고도 부릅니다.

