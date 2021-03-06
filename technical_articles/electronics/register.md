# 저항

## 색상띠

|  색상  |  숫자  | 허용오차  |
| :----: | :----: | :-------: |
|  검정  |   0    | $\pm 1$%  |
|  갈색  |   1    |     -     |
| 빨간색 |   2    |     -     |
| 주황색 |   3    |     -     |
| 노랑색 |   4    |     -     |
| 초록색 |   5    |     -     |
| 파랑색 |   6    |     -     |
| 보라색 |   7    |     -     |
|  회색  |   8    |     -     |
|  흰색  |   9    |     -     |
|  금색  | 0.1배  | $\pm 5$%  |
|  은색  | 0.01배 | $\pm 10$% |



## 읽는 법

### 허용오차

우측 하나는 허용 오차를 나타낸다.

### 저항값

좌측 2~3개는 저항값을 나타내는데, 마지막 하나는 10의 지수부를, 앞의 나머지는 실수부를 나타낸다.

![img](https://upload.wikimedia.org/wikipedia/commons/thumb/f/fc/Tiru%C5%9F_%285%29.jpg/300px-Tiru%C5%9F_%285%29.jpg)

예를 들어 위 사진에서,

첫번째 큰 저항은 $51 \times 10^0 \Omega$ 이다.

두번재 저항은 $22 \times 0.01 \Omega$이다.

세번째 저항은 $10 \times 10^2 \Omega$이다.

네번째 저항도 $10 \times 10^2 \Omega$이다.

다섯번째 저항은 $20 \times 10^2 \Omega$이다.

## 규격

### E24 제품군

3개의 띠로 표현.

허용오차: 5%

실수부: 10, 11, 12, 13, 15, 16, 18, 20, 22, 24, 27, 30, 33, 36, 39, 43, 47, 51, 56, 62, 68, 72, 82, 91 이 있다.

### E96 제품군

4개의 띠로 표현 (E192, E96, E48에 해당)

허용오차: 1%

자세한 내용은 [이곳](https://m.cafe.daum.net/funny-circuit/BgLH/8)을 참조.

## 직렬로 연결

저항을 직렬로 연결하면
$$
R_\text{total} = R_1 + R_2
$$
과 같다.

저항을 2개 이상 직렬로 연결하면 전력을 높일 수 있다.

가변저항(포텐셔미터)를 고정저항과 함께 사용하면 두 저항의 저항값이 더해지기 때문에 저항값이 고정저항의 값보다 낮아지지 않는다.

직렬로 연결한 저항은 분압기를 구성하는데 사용되기도 한다.

## 병렬로 연결

저항을 병렬로 연결하면
$$
R_\text{total} = \frac{1}{\frac{1}{R_1} + \frac{1}{R_2}}
$$

## 분압기

분압기는 전압을 나누는 일을 합니다. 5V를 입력하여 3V를 얻을 수 있습니다.

 <img src="register.assets/220px-Resistive_divider2.svg.png" alt="img" style="zoom:67%;" />
$$
V_\text{out} = \frac{R_2}{R_1 + R_2} \cdot V_\text{in}
$$
분압기로 전력을 낮추어 사용하는 경우는 출력이 높아야 하는 경우 무용지물이므로 다른 방법을 사용해야 한다.

## 참조

- 위키백과: [저항기](https://ko.wikipedia.org/wiki/%EC%A0%80%ED%95%AD%EA%B8%B0)

- 참조: [저항이란?, 2013, 덤보](https://m.cafe.daum.net/funny-circuit/BgLH/8)

- 위키백과: [Voltage Driver](https://en.wikipedia.org/wiki/Voltage_divider)