# 바이트 페어 인코딩(BPE) 토크나이저
[토큰화 (Tokenizations)](./index.md)


바이트 페어 인코딩(Byte Pair Encoding, BPE)은 현대 자연어 처리(Natural Language Processing, NLP), 특히 대규모 언어 모델(Large Language Models, LLMs)의 근간을 이루는 핵심 기술 중 하나다. 그러나 그 기원은 언어 처리와는 무관한 데이터 압축의 영역에 있다. BPE의 발전사는 단순한 데이터 압축 기법이 어떻게 시대적 요구에 따라 그 목적과 철학을 달리하며 인공지능 분야의 혁신을 이끄는 중추적 기술로 변모했는지를 명확히 보여준다. 이 장에서는 BPE의 탄생 배경과 NLP 분야로의 도입 과정을 추적하며, 이 기술이 지닌 역사적 의의와 현대적 위상을 조명한다.


BPE 알고리즘은 1994년 Philip Gage가 "The C User Journal"에 기고한 'A New Algorithm for Data Compression'이라는 논문을 통해 처음 세상에 알려졌다.1 당시 Gage의 목표는 순수하게 데이터를 더 작은 크기로 저장하고 전송하는 것이었다. BPE의 핵심 원리는 지극히 단순하고 직관적이다. 주어진 데이터 스트림에서 가장 빈번하게 연속적으로 나타나는 바이트 쌍(digram)을 찾아, 해당 데이터에 존재하지 않는 새로운 '대체(placeholder)' 바이트로 치환하는 과정을 반복하는 것이다.1

예를 들어, `aaabdaaabac`라는 데이터가 있다면, 가장 빈번한 쌍은 `aa`이다. 이를 사용되지 않은 바이트인 `Z`로 치환하면 데이터는 `ZabdZabac`가 되고, 치환 테이블에는 `Z=aa`가 기록된다. 다음으로 빈번한 `ab` 쌍을 `Y`로 치환하면 `ZYdZYac`가 되고, 테이블에는 `Y=ab`가 추가된다. 이 과정은 더 이상 빈번하게 나타나는 쌍이 없거나 사용할 대체 바이트가 없을 때까지 계속된다. 원본 데이터는 압축된 데이터와 치환 테이블만 있으면 손실 없이 완벽하게 복원할 수 있으므로, 이는 무손실 압축(lossless compression) 기법에 해당한다.4

Gage가 BPE를 제안했을 당시, 데이터 압축 분야에서는 Lempel-Ziv-Welch(LZW)와 같은 알고리즘이 널리 사용되고 있었다.3 BPE는 LZW에 비해 압축 속도는 다소 느렸지만, 압축 해제 루틴이 매우 작고 빨라 메모리가 제한적인 환경에서 특히 유용했다.3 이처럼 BPE는 본래 NLP와는 전혀 무관하게, 순수하게 정보 이론적 관점에서 데이터의 중복성을 제거하여 저장 효율을 높이는 것을 유일한 목적으로 설계된 실용적인 압축 도구였다.


BPE가 데이터 압축의 영역을 넘어 NLP 분야의 핵심 기술로 도약하게 된 결정적인 전환점은 2016년, Rico Sennrich와 그의 동료들이 발표한 기념비적인 논문 "Neural Machine Translation of Rare Words with Subword Units"에서 비롯되었다.6 당시 신경망 기계 번역(Neural Machine Translation, NMT) 모델들은 고정된 크기의 어휘집(vocabulary)을 기반으로 작동했다. 이는 모델이 훈련 데이터에 등장한 단어들만 이해하고 생성할 수 있다는 근본적인 한계를 의미했다. 따라서 훈련 데이터에 없었던 단어, 즉 미등록 단어(Out-of-Vocabulary, OOV)가 입력으로 들어오면 모델은 이를 처리하지 못하고 성능이 급격히 저하되는 문제가 발생했다.6

Sennrich 연구팀은 이 OOV 문제를 해결하기 위한 혁신적인 방법으로 BPE 알고리즘을 재해석하여 도입했다. 그들은 Gage의 알고리즘을 변형하여, 단어를 고정된 단위가 아닌, 더 작은 의미 단위인 '서브워드(subword)'의 연속으로 표현하는 방법을 제안했다.6 이 변형된 BPE의 목표는 최대 압축이 아니었다. 대신, 사용자가 미리 지정한 크기의 어휘집을 구축하는 것을 목표로 삼았다. 알고리즘은 텍스트 코퍼스에 등장하는 모든 개별 문자로 초기 어휘집을 구성한 뒤, 가장 빈번하게 등장하는 인접한 토큰(초기에는 문자) 쌍을 병합하여 새로운 토큰(서브워드)을 만들고 이를 어휘집에 추가하는 과정을 반복한다.1 이 과정은 어휘집 크기가 목표치에 도달할 때까지 계속된다.

이러한 접근법을 통해, "unbelievable"과 같은 단어는 `["un", "believ", "able"]`과 같이 의미 있는 단위로 분절될 수 있다.2 설령 "hyperparameter"라는 단어가 훈련 데이터에 없었더라도, 

`["hyper", "parameter"]`와 같이 알려진 서브워드들의 조합으로 표현될 수 있다면 모델은 그 의미를 유추할 수 있게 된다. 이는 NMT 모델이 고정된 어휘의 한계를 넘어, 사실상 무한한 단어를 처리할 수 있는 '개방 어휘(open-vocabulary)' 능력을 갖추게 됨을 의미했다.6


Sennrich의 선구적인 연구 이후, BPE는 자연어 처리 분야, 특히 트랜스포머(Transformer) 아키텍처를 기반으로 하는 대규모 언어 모델의 표준적인 토큰화 기법으로 빠르게 자리 잡았다. OpenAI의 GPT 시리즈(GPT, GPT-2, GPT-3), Meta의 Llama, Google의 BART 등 오늘날의 대표적인 LLM 다수가 BPE 또는 그 변형을 채택하고 있다.2

BPE가 이토록 널리 채택된 이유는 서브워드 토큰화가 제공하는 본질적인 효용성 때문이다. 기존의 토큰화 방식은 양극단의 문제를 안고 있었다. '단어(word)' 수준 토큰화는 어휘집의 크기가 수십만 개에 달할 정도로 거대해져 메모리 및 계산 비용이 막대했고, OOV 문제에 여전히 취약했다.8 반대로 '문자(character)' 수준 토큰화는 어휘집 크기는 작지만, 텍스트를 너무 긴 토큰 시퀀스로 변환하여 모델의 처리 효율을 떨어뜨리고, 문자 하나하나가 갖는 의미 정보가 부족하여 모델이 언어의 고차원적 구조를 학습하기 어렵게 만들었다.10

BPE는 이 두 극단 사이에서 절묘한 균형점을 찾았다. 자주 사용되는 단어나 의미 있는 형태소는 하나의 토큰으로 유지하여 시퀀스 길이를 효율적으로 관리하고, 희귀하거나 처음 보는 단어는 여러 개의 알려진 서브워드로 분해하여 어휘집의 무한한 확장을 막는다.4

결론적으로 BPE의 역사는 기술의 목적이 근본적으로 재정의되는 과정을 보여준다. Gage에게 BPE는 '정보량의 최소화'라는 압축의 관점에서 최적화된 도구였다. 이는 정보 이론에 기반한 '효율적 저장'에 초점을 맞춘다. 반면 Sennrich에게 BPE는 '의미의 분해와 재조합'이라는 표현의 관점에서 재해석된 도구였다. 이는 표현 학습(representation learning)에 기반한 '의미적 일반화'에 초점을 맞춘다. 이 패러다임의 전환은 NLP 분야의 핵심 과제가 단순한 기호 처리를 넘어, 기호의 의미적 구조를 어떻게 효율적으로 학습하고 표현할 것인가로 이동했음을 명확히 보여준다. BPE의 성공은 복잡하고 정교한 이론이 아닌, 단순하고 견고한 고전적 아이디어를 창의적으로 재활용함으로써 최첨단 AI 시스템을 구축할 수 있다는 사실을 증명하는 강력한 사례다.4


BPE 토크나이저의 작동 원리는 크게 두 단계로 나뉜다. 첫 번째는 대규모 텍스트 코퍼스로부터 서브워드 어휘집과 병합 규칙을 학습하는 '어휘집 구축(학습) 단계'이며, 두 번째는 학습된 규칙을 이용해 새로운 텍스트를 토큰 시퀀스로 변환하는 '토큰화(추론) 단계'이다. 이 장에서는 각 단계의 메커니즘을 구체적인 예시와 형식적 표현을 통해 심도 있게 분석한다.


어휘집 구축 단계의 목표는 주어진 텍스트 코퍼스의 통계적 특성을 반영하여, 가장 효율적인 서브워드 집합을 생성하는 것이다. 이 과정은 초기 어휘집 구성에서 시작하여, 통계에 기반한 반복적인 병합을 통해 최종 어휘집을 완성한다.


BPE 학습 과정의 첫걸음은 가장 기본적인 단위로 구성된 '초기 어휘집'을 정의하는 것이다. 전통적으로 이는 훈련 코퍼스에 존재하는 모든 고유한 문자(character)들의 집합으로 구성된다.5 예를 들어, 다음과 같은 단어와 빈도로 구성된 코퍼스가 주어졌다고 가정하자.

- `("low", 5), ("lower", 2), ("newest", 6), ("widest", 3)`

이 코퍼스에서 고유 문자를 추출하면 초기 어휘집 Vinitial은 `{'l', 'o', 'w', 'e', 'r', 'n', 's', 't', 'i', 'd'}`가 된다. 여기에 단어의 경계를 명확히 하기 위해 단어의 끝을 나타내는 특수 기호(예: `</w>` 또는 `_`)를 추가하는 것이 일반적이다.14 따라서 최종적인 초기 어휘집은 `{'l', 'o', 'w', 'e', 'r', 'n', 's', 't', 'i', 'd', '_'}`가 된다.

그러나 이러한 문자 기반 접근법은 심각한 한계를 가진다. 세상에 존재하는 모든 유니코드 문자를 초기 어휘집에 포함시키려면 그 크기가 매우 커지며, 그럼에도 불구하고 훈련 데이터에 없었던 새로운 문자(예: 이모지, 특수 기호)가 등장하면 여전히 OOV 문제가 발생한다.

이 문제를 근본적으로 해결하기 위해 GPT-2와 같은 현대 언어 모델들은 '바이트 수준 BPE(Byte-Level BPE, BBPE)'를 채택했다.1 BBPE는 텍스트를 유니코드 문자가 아닌, UTF-8 인코딩을 통해 변환된 바이트(byte) 시퀀스로 취급한다. 바이트는 8비트로 표현되며, 0부터 255까지 총 256개의 고유한 값을 가질 수 있다. 따라서 BBPE의 초기 어휘집은 이 256개의 모든 바이트 값으로 고정된다.9 이 접근법의 가장 큰 장점은 보편성이다. 지구상의 어떤 텍스트라도 UTF-8 바이트 시퀀스로 표현될 수 있으므로, BBPE는 이론적으로 어떠한 문자열도 OOV 없이 처리할 수 있다. 이는 어휘집에 없는 문자를 `[UNK]`(unknown) 토큰으로 대체해야 했던 기존 방식의 한계를 원천적으로 극복한다.1


초기 어휘집이 구성되면, 알고리즘은 반복적인 병합 과정을 통해 어휘집을 확장해 나간다. 각 반복 단계에서 BPE는 현재 코퍼스에 존재하는 모든 인접한 토큰 쌍의 빈도를 계산한다.15 그리고 그중 가장 높은 빈도를 가진 쌍을 선택하여 하나의 새로운 토큰으로 병합한다. 이 병합된 새 토큰은 어휘집에 추가되며, 이 병합 과정 자체가 하나의 '병합 규칙(merge rule)'으로 기록된다.14 코퍼스 내에서는 해당 쌍이 나타나는 모든 위치가 새로 생성된 토큰으로 대체된다.

이 과정은 사용자가 사전에 정의한 총 병합 횟수, 또는 목표 어휘집 크기에 도달할 때까지 계속된다.5 예를 들어, 목표 어휘집 크기가 50,000이고 초기 어휘집 크기가 256(바이트 수준)이라면, 총 49,744번의 병합이 수행된다.

앞서 제시된 예시 `("low_", 5), ("lower_", 2), ("newest_", 6), ("widest_", 3)`를 통해 병합 과정을 살펴보자. (단어 끝 기호 `_` 추가)

1. **초기 상태:** 코퍼스는 문자 단위로 분리되어 있다. `(l o w _, 5), (l o w e r _, 2), (n e w e s t _, 6), (w i d e s t _, 3)`
2. **병합 1:** 인접 쌍의 빈도를 계산한다. `(e, s)` 쌍은 `newest_`에서 6번, `widest_`에서 3번, 총 9번 등장하여 가장 빈번하다.
   - **병합 규칙 생성:** `(e, s) → es`
   - **어휘집 추가:** `V`에 `es` 추가
   - **코퍼스 업데이트:** `(l o w _, 5), (l o w e r _, 2), (n e w es t _, 6), (w i d es t _, 3)`
3. **병합 2:** 업데이트된 코퍼스에서 다시 빈도를 계산한다. 이제 `(es, t)` 쌍이 총 9번으로 가장 빈번하다.
   - **병합 규칙 생성:** `(es, t) → est`
   - **어휘집 추가:** `V`에 `est` 추가
   - **코퍼스 업데이트:** `(l o w _, 5), (l o w e r _, 2), (n e w est _, 6), (w i d est _, 3)`
4. **병합 3:** 다음으로 빈번한 쌍은 `(l, o)`로, 총 7번 등장한다.
   - **병합 규칙 생성:** `(l, o) → lo`
   - **어휘집 추가:** `V`에 `lo` 추가
   - **코퍼스 업데이트:** `(lo w _, 5), (lo w e r _, 2), (n e w est _, 6), (w i d est _, 3)`

이러한 과정은 목표 병합 횟수에 도달할 때까지 계속된다. BPE의 이러한 접근 방식은 '탐욕적(greedy)' 알고리즘에 해당한다. 각 단계에서 국소적으로 최적의 선택(가장 빈번한 쌍을 병합)을 할 뿐, 모든 병합 과정을 마쳤을 때 전역적으로 최적의 어휘집이 생성된다는 보장은 없다.4


BPE 어휘집 학습 알고리즘의 절차적 본질은 다음 의사 코드를 통해 명확하게 표현할 수 있다.16

```
\begin{algorithm}
\caption{BPE Vocabulary Learning Algorithm}\label{alg:bpe_learn}
\begin{algorithmic}
\Procedure{LearnBPE}{$corpus, V_{size}$}
    \State $V \gets \text{Initial vocabulary from unique characters/bytes in } corpus$
    \State $M \gets \text{empty list of merge rules}$
    \State $num\_merges \gets V_{size} - \rvert V \rvert$
    \For{$i = 1 \to num\_merges$}
        \State $pairs \gets \text{get\_stats}(corpus, V)$ \Comment{Count all adjacent pair frequencies}
        \State $best\_pair \gets \text{argmax}_{p \in pairs} (\text{frequency}(p))$
        \State $V \gets V \cup \{ \text{merge}(best\_pair) \}$
        \State $M.\text{append}(best\_pair)$
        \State $corpus \gets \text{replace\_all}(corpus, best\_pair, \text{merge}(best\_pair))$
    \EndFor
    \State \textbf{return} $V, M$
\EndProcedure
\end{algorithmic}
\end{algorithm}
```

수학적 관점에서 BPE는 조합 최적화(combinatorial optimization) 문제로 형식화될 수 있다. 각 병합 단계는 전체 텍스트의 압축률, 즉 최종 토큰 시퀀스의 길이를 최소화하는 쌍을 선택하는 과정으로 볼 수 있다.22 이 문제에 대한 최적해를 찾는 것은 계산적으로 매우 어렵기 때문에, BPE는 탐욕적 접근법을 통해 근사해를 구한다. 최근 연구에서는 서브모듈러 함수(submodular functions) 이론을 통해 BPE의 탐욕적 알고리즘이 제공하는 근사해의 품질 보증(approximation guarantee)을 분석하기도 했다.22 또한, 알고리즘의 시간 복잡도는 순진한 구현의 경우 $O(NM)$ (여기서 $N$은 코퍼스 길이, $M$ 병합 횟수)이지만, 우선순위 큐와 같은 자료 구조를 활용하여 $O(N \log M)$으로 개선할 수 있다.22


학습 단계가 완료되면, 최종 어휘집(V)과 병합 규칙의 순서화된 목록(M)이 생성된다. 토큰화 단계에서는 이 결과물을 사용하여 새로운, 즉 이전에 보지 못한 텍스트를 토큰 시퀀스로 변환한다.


새로운 텍스트를 토큰화하는 과정은 학습된 병합 규칙 목록(M)을 순서대로 적용하는 간단하고 결정론적인 절차다.17

1. **초기 분할:** 입력 텍스트를 초기 어휘집의 단위(문자 또는 바이트) 시퀀스로 분할한다.
2. **반복적 병합:** 학습된 병합 규칙 목록(M)의 첫 번째 규칙부터 마지막 규칙까지 순서대로 훑어가며, 현재 텍스트 시퀀스에 적용할 수 있는 규칙이 있다면 즉시 병합을 수행한다. 이 과정을 모든 규칙에 대해 반복한다.9

이 과정은 탐욕적이며, 규칙의 적용 순서가 매우 중요하다. 즉, 항상 학습 시 먼저 생성된 규칙(더 빈번했던 쌍)이 나중에 생성된 규칙보다 우선적으로 적용된다.


학습을 통해 다음과 같은 순서의 병합 규칙 목록 M이 생성되었다고 가정하자.

- M=[(e,s)→es,(es,t)→est,(l,o)→lo,(lo,w)→low]

이제 새로운 단어 "lowest"를 토큰화해 보자.

1. **초기 분할:** 단어를 문자 시퀀스로 분할하고 단어 끝 기호를 추가한다.
   - `['l', 'o', 'w', 'e', 's', 't', '_']`
2. **병합 규칙 적용 (순서대로):**
   - **규칙 1 `(e, s) → es`:** 시퀀스 내에 `(e, s)` 쌍이 존재하므로 병합한다.
     - `['l', 'o', 'w', 'es', 't', '_']`
   - **규칙 2 `(es, t) → est`:** 시퀀스 내에 `(es, t)` 쌍이 존재하므로 병합한다.
     - `['l', 'o', 'w', 'est', '_']`
   - **규칙 3 `(l, o) → lo`:** 시퀀스 내에 `(l, o)` 쌍이 존재하므로 병합한다.
     - `['lo', 'w', 'est', '_']`
   - **규칙 4 `(lo, w) → low`:** 시퀀스 내에 `(lo, w)` 쌍이 존재하므로 병합한다.
     - `['low', 'est', '_']`
3. **최종 토큰 시퀀스:** 더 이상 적용할 수 있는 병합 규칙이 없으므로 과정을 종료한다. 만약 학습 과정에서 `(est, _)`가 `est_`로 병합되는 규칙이 있었다면, 최종 결과는 `["low", "est_"]`가 될 것이다.14

이처럼 BPE의 학습 및 토큰화 과정은 경로 의존적(path-dependent) 특성을 가진다. 학습 단계에서 어떤 쌍이 먼저 병합되느냐가 그 이후의 모든 병합 가능성과 최종 어휘집의 구성을 결정한다. 예를 들어, `(e, s)`가 먼저 병합되면 `(s, t)`의 빈도는 0이 되고, 대신 `(es, t)`라는 새로운 병합 후보가 등장한다. 이렇게 결정된 고정된 병합 규칙 리스트는 토큰화 단계에서 엄격한 순서로 적용되므로, 동일한 문자열은 항상 동일한 토큰 시퀀스로 변환된다. 이러한 결정론적(deterministic) 특성은 모델 입력의 일관성을 보장하는 중요한 장점이지만, 동시에 언어가 가진 문맥적 다의성이나 유연성을 포착하지 못하는 근본적인 한계의 원인이 되기도 한다.


BPE는 현대 NLP, 특히 대규모 언어 모델의 발전에 지대한 공헌을 했지만, 그 설계에 내재된 한계 또한 명확하다. 이 장에서는 BPE가 제공하는 핵심적인 이점과 그 이면에 존재하는 단점들을 심층적으로 분석하고, 특히 OOV 문제에 대한 궁극적인 해결책으로 제시된 Byte-Level BPE의 효용성과 그에 따르는 대가를 고찰한다.


BPE가 널리 채택된 가장 큰 이유는 미등록 단어(OOV) 문제를 효과적으로 해결하면서도, 어휘집 크기와 처리 효율성 사이의 실용적인 균형을 제공하기 때문이다.


전통적인 단어 기반 토크나이저의 가장 큰 골칫거리는 OOV 문제였다. 훈련 코퍼스에 등장하지 않은 단어는 `[UNK]`(unknown) 토큰으로 처리되어 정보 손실을 야기했다.8 BPE는 이 문제를 근본적으로 완화한다.4 훈련 시 보지 못한 단어라 할지라도, 그 단어를 구성하는 더 작은 단위의 서브워드들이 어휘집에 존재한다면, 해당 단어를 이 서브워드들의 시퀀스로 표현할 수 있다.25

예를 들어, 'tokenization'이라는 단어가 OOV일지라도, 어휘집에 `["token", "iz", "ation"]`과 같은 서브워드가 있다면 모델은 이들의 조합을 통해 단어의 의미를 추론할 수 있다. 이러한 능력은 특히 다음과 같은 경우에 강력한 강건성(robustness)을 부여한다.

- **희귀 단어 및 신조어:** 자주 사용되지 않는 단어나 새롭게 생겨난 단어(예: 'deepfake')도 기존 서브워드로 분해하여 처리할 수 있다.5
- **형태론적 변형:** 'run', 'running', 'ran'과 같이 형태가 변하는 단어들도 공통된 어근 'run'을 공유하는 토큰으로 표현되어 모델이 이들의 관계를 학습하기 용이해진다.26
- **오탈자 및 노이즈:** 'tokenizatoin'과 같은 오탈자도 `["token", "iz", "atoin"]`과 같이 최대한 알려진 서브워드로 분해하여 의미 손실을 최소화할 수 있다.5


토큰화 방식은 어휘집 크기와 생성되는 토큰 시퀀스 길이 사이에 본질적인 상충관계(trade-off)를 가진다.

- **단어 수준 토큰화:** 각 단어를 고유 토큰으로 취급하므로 시퀀스 길이는 짧지만, 언어에 존재하는 모든 단어를 포함하려면 어휘집 크기가 수십만에서 수백만에 달할 수 있다. 이는 모델의 임베딩 레이어 크기를 비대하게 만들어 막대한 메모리와 계산 자원을 요구한다.8
- **문자 수준 토큰화:** 어휘집 크기는 알파벳, 숫자, 기호를 포함해도 수백 개 수준으로 매우 작다. 하지만 'unbelievable'이라는 단어 하나가 13개의 토큰으로 변환되는 등, 생성되는 시퀀스 길이가 매우 길어져 모델이 장거리 의존성을 학습하기 어려워지고 계산 효율성이 저하된다.12

BPE는 이 두 극단 사이의 실용적인 절충안을 제공한다.10 자주 등장하는 단어(예: 'the', 'is')나 의미 있는 서브워드(예: 'tion', 'ing')는 하나의 토큰으로 병합하여 시퀀스 길이를 줄인다. 반면, 희귀한 단어는 여러 개의 서브워드로 분해하여 어휘집이 불필요하게 커지는 것을 방지한다. 이처럼 BPE는 어휘집 크기를 수만 개 수준(예: GPT-3의 경우 약 5만 개)으로 제어하면서도, 대부분의 텍스트를 효율적인 길이의 시퀀스로 변환할 수 있다.4


BPE의 단순함과 효율성은 탐욕적(greedy)이고 순수하게 통계에만 의존하는 설계에서 비롯된다. 이는 동시에 여러 가지 내재적 한계를 낳는다.


BPE는 오직 인접 쌍의 빈도만을 기준으로 병합을 결정한다. 이 과정에는 언어학적 지식이나 단어의 의미 구조에 대한 고려가 전혀 없다.4 그 결과, 생성된 서브워드가 인간의 직관이나 언어의 형태론적 단위(morpheme)와 일치하지 않는 경우가 빈번하게 발생한다.

가장 널리 알려진 예시는 "therapist"라는 단어가 `["the", "rap", "ist"]`와 같이 의미적으로 완전히 무관하고 심지어 부정적인 함의를 가질 수 있는 토큰들로 분할되는 경우다.2 이러한 비의미론적 분할은 모델이 단어의 진정한 구성 요소를 학습하는 데 혼란을 줄 수 있으며, 모델의 해석 가능성을 저해하는 요인이 된다.


BPE 어휘집은 훈련 코퍼스를 기반으로 한번 생성되고 나면 완전히 고정된다. 이는 모델이 배포된 이후에 등장하는 새로운 언어 패턴이나 어휘 변화에 적응할 수 없음을 의미한다.4

더욱 근본적인 문제는 BPE의 토큰화 과정이 전적으로 문맥 독립적(context-independent)이라는 점이다. 예를 들어, "The band is playing rock music"에서의 'rock'과 "He threw a rock into the river"에서의 'rock'은 의미가 전혀 다르지만, BPE는 두 경우 모두 동일한 토큰 ID로 변환한다. 이러한 동음이의어(homonym)의 모호성을 해결하는 부담은 전적으로 모델의 후속 레이어에 전가된다.4


Byte-Level BPE(BBPE)는 문자 수준에서 발생할 수 있는 OOV 문제까지 해결하기 위해 제안된 BPE의 확장판이다. 이는 완벽한 강건성을 제공하지만, 새로운 형태의 비효율성을 야기한다.


BBPE는 초기 어휘집을 256개의 모든 가능한 바이트 값으로 고정한다.1 모든 텍스트는 표준 UTF-8 인코딩을 통해 바이트 시퀀스로 변환될 수 있으므로, BBPE는 이론적으로 어떠한 언어, 이모지, 특수 기호도 `[UNK]` 토큰 없이 표현할 수 있는 완벽한 보편성을 확보한다.1 이는 다국어 환경이나 예측 불가능한 사용자 입력이 많은 실제 서비스 환경에서 모델의 안정성을 크게 높여준다.


BBPE가 제공하는 완벽한 강건성은 상당한 비용을 수반한다. 가장 큰 문제는 토큰 시퀀스의 길이가 급격히 증가한다는 것이다. 예를 들어, 영어 알파벳과 같은 ASCII 문자는 1바이트로 표현되지만, 한국어, 중국어, 일본어(CJK)의 많은 문자는 UTF-8에서 3바이트로 인코딩된다. 따라서 BBPE를 사용하면 이러한 언어의 텍스트는 문자 수준 토큰화에 비해 시퀀스 길이가 3배로 늘어난다.29

이렇게 길어진 시퀀스는 여러 문제를 야기한다.

- **계산 비용 증가:** 트랜스포머 모델의 계산 복잡도는 시퀀스 길이에 대해 제곱에 비례(O(n2))하므로, 시퀀스 길이가 길어지면 훈련 및 추론에 필요한 계산량이 기하급수적으로 증가한다.13
- **유효 컨텍스트 길이 감소:** 모델이 한 번에 처리할 수 있는 최대 시퀀스 길이(context window)는 제한되어 있다. BBPE는 동일한 양의 정보를 더 긴 토큰 시퀀스로 표현하므로, 모델이 참조할 수 있는 실제 텍스트의 양, 즉 유효 컨텍스트 길이가 줄어든다.13
- **학습 부담 가중:** 모델은 이제 개별 바이트들의 조합으로부터 의미 있는 문자나 단어를 재구성하는 법을 학습해야 한다. 예를 들어, 모델이 유효한 UTF-8 바이트 시퀀스를 생성하도록 학습하는 것은 문자 단위로 작업할 때보다 더 어려운 과제가 될 수 있다.29

이러한 특성은 토크나이저의 설계 철학과 모델의 역할 분담에 대한 중요한 시사점을 던진다. BPE와 같은 서브워드 토크나이저는 '자주 함께 나타나는 문자열은 의미 단위일 것이다'라는 강한 귀납적 편향(inductive bias)을 모델에 주입한다. 이 편향 덕분에 모델은 `["un", "believ", "able"]`과 같이 어느 정도 정제된 입력을 받아 학습 부담을 덜 수 있다. 반면, BBPE는 이러한 언어학적 편향을 최소화하고 '모든 텍스트는 바이트의 연속일 뿐'이라는 훨씬 약한 편향을 선택한다. 그 결과, 언어의 구조적, 통계적 패턴을 파악해야 하는 부담이 토크나이저에서 모델 자체로 상당 부분 전가된다. BBPE의 채택은 토크나이저의 역할을 '의미 단위 추출기'에서 '보편적 인코더'로 전환시키고, 그만큼 모델의 학습 능력을 더 많이 요구하는 설계 결정이라고 볼 수 있다. 이는 대규모 모델이 더 원시적인(raw) 데이터로부터 더 복잡한 패턴을 학습할 수 있다는 믿음에 기반한다.


BPE는 서브워드 토큰화의 시대를 열었지만, 유일한 해결책은 아니다. BPE의 한계를 개선하거나 다른 철학적 접근을 시도하는 여러 알고리즘이 개발되었다. 이 장에서는 BPE를 WordPiece, SentencePiece, Unigram LM과 같은 주요 경쟁 알고리즘과 비교하여, 각 방식의 핵심적인 차이점과 장단점을 분석한다.


BPE와 WordPiece는 모두 문자 단위에서 시작하여 반복적으로 토큰을 병합하는 상향식(bottom-up) 접근법을 공유하지만, 병합 대상을 선택하는 기준에서 근본적인 차이를 보인다.

- **BPE의 병합 기준:** BPE는 순수하게 통계적 **빈도(frequency)**에 의존한다. 각 단계에서 코퍼스 내에서 가장 자주 등장하는 인접 토큰 쌍을 탐욕적으로 선택하여 병합한다.30

- **WordPiece의 병합 기준:** WordPiece는 훈련 데이터의 **우도(likelihood)**를 최대화하는 방향으로 병합을 수행한다.2 즉, 수많은 가능한 병합 후보 중에서, 해당 쌍을 새로운 하나의 토큰으로 병합했을 때 언어 모델의 관점에서 코퍼스의 생성 확률을 가장 많이 높이는 쌍을 선택한다. 이 기준은 수학적으로 두 토큰의 점별 상호 정보량(Pointwise Mutual Information, PMI)과 유사하게 공식화될 수 있다: 

  ```
  Score(A,B)=freq(A)×freq(B)freq(AB).32
  ```

이러한 차이는 토큰화 결과에 미묘하지만 중요한 영향을 미친다. BPE가 단순히 자주 나타나는 문자열(예: `es`)을 병합하는 경향이 있다면, WordPiece는 개별적으로 나타나는 빈도에 비해 함께 나타날 확률이 유의미하게 높은 쌍(예: `jet` 다음에 `##son`이 오는 경우)을 선호한다. 이로 인해 WordPiece는 BPE보다 언어학적으로 더 타당하고 의미 있는 단위로 단어를 분절하는 경향이 있다.33 Google이 개발한 BERT, RoBERTa, DistilBERT와 같은 모델들은 주로 WordPiece를 토크나이저로 사용한다.2


BPE와 WordPiece는 일반적으로 텍스트를 공백이나 구두점을 기준으로 단어로 먼저 나누는 '사전 토큰화(pre-tokenization)' 과정에 의존한다.21 이 방식은 영어와 같이 단어 경계가 명확한 언어에서는 효과적이지만, 여러 단점이 존재한다. SentencePiece는 이러한 전처리 의존성을 제거하기 위해 설계된 통합 프레임워크다.

- **BPE/WordPiece의 한계:** 사전 토큰화는 언어별 규칙에 의존해야 하며, 일본어, 중국어, 태국어와 같이 단어 간 공백이 없는 언어에는 적용하기 어렵다. 또한, 토큰화와 역토큰화(detokenization) 과정이 비가역적이 되어, 토큰화된 시퀀스를 다시 원본 텍스트로 완벽하게 복원하기 어려운 경우가 발생한다.
- **SentencePiece의 접근:** SentencePiece는 사전 토큰화 없이 원시 텍스트(raw text)를 직접 입력으로 받는다.2 이 알고리즘은 공백(whitespace)을 버리는 대신, 이를 ` ` (U+2581)와 같은 특수한 메타 기호로 취급하여 다른 문자와 동등하게 처리한다.18 즉, 모든 텍스트를 유니코드 문자 시퀀스로 간주하고 토큰화를 수행한다.

이러한 접근법 덕분에 SentencePiece는 언어에 독립적인(language-agnostic) 토큰화 시스템을 구축할 수 있으며, 특히 다국어 모델에 매우 적합하다.30 SentencePiece는 특정 알고리즘을 지칭하기보다는, BPE 또는 Unigram LM 알고리즘을 내부 구현으로 사용하여 원시 텍스트를 처리할 수 있는 강력한 프레임워크라고 할 수 있다.36 Google의 T5, ALBERT, XLNet과 같은 다국어 및 다중 과제 모델들이 SentencePiece를 채택하고 있다.2


BPE가 작은 단위에서 시작해 어휘집을 키워나가는 방식이라면, Unigram Language Model(Unigram LM)은 정반대의 접근법을 취한다.

- **BPE의 접근 방식:** 문자 단위에서 시작하여 어휘집을 점진적으로 구축하는 '생성적(generative)' 또는 '상향식(bottom-up)' 접근법이다.21
- **Unigram LM의 접근 방식:** 모든 단어와 빈번한 부분 문자열 등을 포함하는 매우 큰 초기 어휘집으로 시작한다. 그 후, 각 토큰을 어휘집에서 제거했을 때 전체 코퍼스의 우도(likelihood) 손실이 얼마나 발생하는지를 계산한다. 손실이 가장 적은 토큰부터 순차적으로 제거해 나가며 목표 어휘집 크기에 도달할 때까지 이 과정을 반복한다. 이는 '제거적(pruning)' 또는 '하향식(top-down)' 접근법에 해당한다.5

Unigram LM의 가장 큰 특징은 토큰화 과정이 확률적이라는 점이다. 학습된 Unigram 모델은 주어진 단어에 대해 여러 가능한 분절 경로와 각 경로의 확률을 계산할 수 있다. 예를 들어, "hugging"이라는 단어는 `["hug", "ging"]`, `["h", "ugg", "ing"]`, `["h", "u", "gging"]` 등 여러 방식으로 분절될 수 있으며, Unigram LM은 이 중 가장 확률이 높은 경로를 선택하거나, 훈련 시에는 확률 분포에 따라 여러 경로를 샘플링할 수 있다. 이러한 '서브워드 정규화(subword regularization)' 기법은 모델이 토큰화의 작은 변동에 대해 강건해지도록 돕는 일종의 데이터 증강 효과를 낳는다.37


각 토크나이저의 철학, 메커니즘, 장단점을 종합하면 다음과 같이 정리할 수 있다. 이 표는 각 알고리즘의 핵심적인 차이점을 한눈에 파악할 수 있도록 돕는다.

| 특징 (Feature)     | Byte Pair Encoding (BPE)                   | WordPiece                                   | SentencePiece                          | Unigram LM                                |
| ------------------ | ------------------------------------------ | ------------------------------------------- | -------------------------------------- | ----------------------------------------- |
| **핵심 아이디어**  | 데이터 압축 기반 서브워드 생성             | 언어 모델 우도 기반 서브워드 생성           | 전처리 독립적인 통합 토큰화 프레임워크 | 확률 기반 서브워드 제거                   |
| **병합/제거 기준** | 인접 쌍의 **빈도(Frequency)** 최대화       | 인접 쌍 병합 시 **우도(Likelihood)** 최대화 | 내부적으로 BPE 또는 Unigram LM 사용    | 전체 우도 손실을 최소화하는 토큰 **제거** |
| **접근 방식**      | 상향식 (Bottom-up)                         | 상향식 (Bottom-up)                          | 프레임워크 (BPE/Unigram 선택)          | 하향식 (Top-down)                         |
| **공백 처리**      | 사전 토큰화에 의존 (단어 경계)             | 사전 토큰화에 의존 (단어 경계)              | 공백을 일반 기호(`_`)로 취급           | 사전 토큰화에 의존 (단어 경계)            |
| **주요 사용 모델** | GPT, GPT-2, Llama, RoBERTa                 | BERT, DistilBERT                            | T5, ALBERT, XLNet, mT5                 | ALBERT, T5 (SentencePiece 내부 옵션)      |
| **장점**           | 구현이 간단하고 빠름. OOV 처리에 효과적.   | 언어 모델링에 최적화된 분할 생성.           | 언어 독립적, 다국어 처리에 강함.       | 여러 토큰화 경로 샘플링 가능 (정규화).    |
| **단점**           | 탐욕적 병합으로 인한 비최적/비의미적 분할. | BPE보다 학습이 복잡함.                      | 분할 결과가 직관적이지 않을 수 있음.   | 학습 과정이 상대적으로 느림.              |

결론적으로, 어떤 토크나이저가 절대적으로 우월하다고 말하기는 어렵다. 선택은 모델의 아키텍처, 훈련 데이터의 언어적 특성, 그리고 해결하고자 하는 특정 과제에 따라 달라진다. BPE는 그 단순성과 효율성 덕분에 여전히 많은 모델에서 강력한 기준으로 사용되고 있으며, 다른 알고리즘들은 BPE의 한계를 특정 방향으로 개선하려는 시도로 이해할 수 있다.


BPE 알고리즘을 적용할 때 가장 중요하게 결정해야 할 하이퍼파라미터는 단연 '어휘집 크기(vocabulary size)'이다. 어휘집 크기는 BPE 학습 과정에서 수행될 총 병합 횟수를 결정하며 17, 이 결정은 모델의 압축률, 성능, 메모리 사용량, 심지어 학습 동역학에까지 광범위하고 다층적인 영향을 미친다. 이 장에서는 어휘집 크기가 시스템 전반에 미치는 영향을 심층적으로 분석한다.


어휘집 크기와 텍스트의 압축률, 즉 토큰 시퀀스의 길이는 직접적인 반비례 관계에 있다.

- **어휘집 크기가 클 경우:** 더 많은 병합이 수행되므로, 더 길고 복잡한 서브워드나 심지어 자주 사용되는 온전한 단어들이 하나의 토큰으로 어휘집에 포함된다.20 예를 들어, 어휘집이 충분히 크다면 "tokenization"이라는 단어가 단일 토큰으로 존재할 수 있다. 이는 텍스트를 더 적은 수의 토큰으로 표현하게 만들어 '압축률'을 높이고, 결과적으로 토큰 시퀀스의 길이를 단축시킨다.42
- **어휘집 크기가 작을 경우:** 병합 횟수가 제한되므로, 어휘집은 주로 짧은 서브워드들로 구성된다. 따라서 긴 단어는 여러 개의 짧은 서브워드로 분해되어야 하므로, 생성되는 토큰 시퀀스의 길이는 길어진다.13

이러한 관계는 다음과 같은 중요한 상충관계를 낳는다.

- **장점 (큰 어휘집, 짧은 시퀀스):** 시퀀스 길이가 짧아지면 모델의 유효 컨텍스트 길이가 늘어난다. 즉, 동일한 컨텍스트 윈도우 내에 더 많은 의미 정보를 담을 수 있다. 또한, 추론 시 생성해야 할 토큰 수가 줄어들어 속도가 향상되는 효과가 있다.42
- **단점 (큰 어휘집):** 어휘집 크기가 커질수록 모델의 입력 임베딩 행렬과 최종 출력 소프트맥스 레이어의 크기가 비례하여 증가한다. 이는 모델의 전체 파라미터 수를 늘리고, 특히 메모리 사용량을 크게 증가시키는 원인이 된다.11


최근 연구는 BPE 어휘집 크기가 모델의 '암기(memorization)' 능력에 직접적인 영향을 미친다는 흥미로운 사실을 밝혀냈다. 다른 모든 조건을 동일하게 통제했을 때, 어휘집 크기가 클수록 트랜스포머 모델이 훈련 데이터를 더 잘, 그리고 더 많이 기억하는 경향이 나타났다.44

이 현상의 주된 원인으로는 어휘집 크기 증가에 따른 '시퀀스 길이의 단축'이 지목된다. 트랜스포머의 핵심인 셀프 어텐션 메커니즘은 시퀀스 내 모든 토큰 쌍 간의 관계를 계산한다. 시퀀스가 짧아지면 모델이 특정 패턴이나 데이터 샘플을 학습하고 기억하는 것이 더 용이해질 수 있다.44

이러한 발견은 중요한 실용적 함의를 가진다.

- **개인정보 보호:** 개인 식별 정보나 민감한 데이터가 포함된 코퍼스로 모델을 훈련할 경우, 큰 어휘집을 사용하는 것은 훈련 데이터를 그대로 생성할 위험을 높일 수 있다.
- **보안:** 큰 어휘집을 가진 모델은 특정 데이터가 훈련에 사용되었는지를 추론하는 멤버십 추론 공격(membership inference attacks)에 더 취약해질 수 있다.44

따라서 모델의 사용 목적에 따라 어휘집 크기를 신중하게 선택해야 한다. 데이터 암기가 중요한 과제(예: 특정 형식의 코드 생성)에서는 큰 어휘집이 유리할 수 있지만, 일반적인 대화형 AI나 개인정보 보호가 중요한 응용에서는 오히려 작은 어휘집이 더 안전한 선택일 수 있다.


어휘집 크기가 특정 다운스트림 태스크(예: 번역, 분류, 질의응답)의 최종 성능에 미치는 영향은 단순하지 않으며, 여러 요인이 복합적으로 작용한다.

- **성능과의 관계:** 일반적으로 어휘집 크기를 늘리면 특정 지점까지는 성능이 향상될 수 있지만, 그 관계가 항상 선형적이지는 않다. 오히려 어휘집이 지나치게 커지면, 빈도가 낮은 희귀 토큰들은 훈련 데이터에서 충분한 예시를 통해 학습되지 못하는 '데이터 희소성(data sparsity)' 문제가 발생할 수 있다.13 이러한 '덜 학습된(undertrained)' 토큰들은 모델 성능에 부정적인 영향을 미칠 수 있다.
- **언어 및 도메인 의존성:** 최적의 어휘집 크기는 처리하고자 하는 언어의 특성과 데이터의 도메인에 따라 크게 달라진다. 독일어나 터키어처럼 단어의 결합과 변형이 많은 교착어(agglutinative language)는 더 많은 형태소를 포착하기 위해 상대적으로 큰 어휘집이 유리할 수 있다.5 마찬가지로, 의학, 법률, 금융 등 특정 전문 용어가 빈번하게 등장하는 도메인에서는 해당 용어들을 토큰으로 포함시키는 것이 모델 성능에 도움이 될 수 있다.46
- **다국어 환경의 비효율성:** 어휘집 크기의 문제는 다국어 환경에서 더욱 두드러진다. 예를 들어, 주로 영어 데이터로 학습된 토크나이저를 어휘집 확장 없이 중국어 텍스트에 적용하면, 중국어 한 글자가 여러 개의 개별 바이트 토큰으로 분해되어 극심한 비효율을 초래한다. 이는 동일한 의미를 전달하는 데 영어보다 훨씬 긴 토큰 시퀀스를 필요로 하게 만들어, 사실상 중국어에 대한 모델의 유효 컨텍스트 길이를 크게 줄이는 결과를 낳는다.43

결론적으로, 어휘집 크기는 단순히 기술적인 파라미터가 아니라, 모델 시스템 내에서 '언어적 지식'이 어디에, 그리고 어떻게 저장될지를 결정하는 근본적인 조절 장치 역할을 한다. 어휘집이 크다는 것은 'unbelievable'이라는 단어의 구조적 지식(`un-` + `believe` + `-able`)이 토크나이저의 어휘집 자체에 명시적으로 '인코딩'됨을 의미한다. 즉, 지식의 일부가 규칙 기반의 형태로 토크나이저에 저장되는 것이다. 반면, 어휘집이 작다면 이러한 구조적 지식은 토크나이저가 아닌, 모델의 수많은 파라미터(가중치) 내에 암묵적으로 '학습'되어야 한다.

따라서 어휘집 크기를 조절하는 행위는 토크나이저(명시적, 규칙 기반 지식)와 모델(암묵적, 파라미터 기반 지식) 사이의 '지식 분배'를 설계하는 것과 같다. 토크나이저에 지식을 많이 위임하면(큰 어휘집), 모델의 학습 부담이 줄고 시퀀스가 짧아져 효율적이지만, 암기 위험이 커지고 새로운 단어 조합에 대한 유연성이 저하될 수 있다. 반대로 모델에 지식을 위임하면(작은 어휘집), 유연성은 높아지지만 더 긴 시퀀스를 처리해야 하고 더 많은 파라미터를 통해 언어 규칙을 스스로 학습해야 하는 부담이 커진다. 최적의 어휘집 크기를 찾는 것은 이러한 복잡한 상충관계를 이해하고, 주어진 과제와 자원 제약 하에서 최적의 균형점을 찾는 과정이라 할 수 있다.


바이트 페어 인코딩(BPE)은 데이터 압축이라는 실용적인 문제에서 출발하여, 현대 자연어 처리의 지형을 근본적으로 바꾼 핵심 기술로 자리매김했다. 그 여정은 단순한 아이디어가 어떻게 시대적 과제와 만나 혁신적인 해결책으로 재탄생할 수 있는지를 보여주는 대표적인 사례다. 이 장에서는 BPE가 남긴 유산을 종합적으로 평가하고, 그 내재적 한계를 극복하기 위한 미래 연구 방향을 조망하며 보고서를 마무리한다.


BPE의 가장 큰 유산은 그 '단순함'과 '실용성'에 있다. 복잡한 언어학적 규칙이나 정교한 통계 모델 없이, 오직 가장 빈번한 쌍을 반복적으로 병합한다는 직관적인 원리 하나만으로 NLP 분야의 고질적인 문제였던 미등록 단어(OOV) 문제를 효과적으로 해결했다. 이는 단어와 문자 수준 토큰화의 장점을 절묘하게 결합하여, 어휘집 크기와 시퀀스 길이 사이의 균형을 맞추는 실용적인 해법을 제시했다.

이러한 BPE의 기여는 대규모 언어 모델(LLM)의 등장을 가능하게 한 기술적 초석이 되었다. 만약 BPE와 같은 효율적인 서브워드 토큰화 기법이 없었다면, 수십억, 수조 개의 파라미터를 가진 오늘날의 LLM들은 폭발하는 어휘집 크기나 비효율적인 시퀀스 길이 문제에 부딪혀 실현 불가능했을지도 모른다. BPE는 언어라는 복잡하고 비정형적인 데이터를, 신경망이 효율적으로 학습할 수 있는 고정된 차원의 벡터 시퀀스로 변환하는 가장 중요한 다리 역할을 수행했다.

결론적으로 BPE는 '충분히 좋은(good enough)' 단순한 아이디어가 복잡한 문제를 해결하는 데 얼마나 강력할 수 있는지를 증명했다. 이는 최첨단 기술이 반드시 난해한 이론에서만 비롯되는 것이 아니라, 기존의 검증된 아이디어를 새로운 관점에서 재해석하고 적용하는 과정에서도 탄생할 수 있음을 보여주는 중요한 교훈을 남긴다.


BPE는 눈부신 성공을 거두었지만, 그 한계 또한 명확하다. 탐욕적 병합으로 인한 비최적, 비의미론적 분할 문제, 문맥을 고려하지 못하는 한계, 그리고 고정된 어휘집의 비유연성은 차세대 언어 모델이 극복해야 할 과제로 남아있다. 이에 따라 BPE의 한계를 넘어서려는 다양한 연구가 활발히 진행되고 있다.

- **언어학적 지식의 결합:** 순수한 통계에만 의존하는 BPE의 단점을 보완하기 위해, 형태소 분석과 같은 언어학적 지식을 토큰화 과정에 결합하려는 시도가 이루어지고 있다. 이를 통해 'therapist'와 같은 단어를 의미론적으로 더 타당한 단위로 분절하여 모델의 학습 효율과 해석 가능성을 높일 수 있다.
- **동적 및 적응형 토큰화:** 고정된 어휘집의 한계를 극복하기 위해, 입력 문맥이나 특정 도메인에 따라 토큰화를 동적으로 조정하는 적응형(adaptive) 토크나이저에 대한 연구가 진행 중이다. 이는 모델이 새로운 어휘나 변화하는 언어 패턴에 더 유연하게 대처할 수 있도록 도울 것이다.
- **토크나이저 없는 모델(Tokenizer-free Models):** 가장 급진적인 방향은 토큰화라는 전처리 단계 자체를 제거하려는 시도다. 이러한 모델들은 텍스트를 토큰 시퀀스가 아닌, 원시 UTF-8 바이트 스트림이나 심지어 비트(bit) 스트림으로 직접 처리한다.29 이 접근법은 토크나이저가 유발하는 모든 종류의 편향과 정보 손실을 원천적으로 제거할 수 있다는 장점이 있다. 하지만 이는 언어의 구조를 파악해야 하는 모든 부담을 모델 아키텍처 자체에 전가하는 것이므로, 훨씬 더 강력하고 효율적인 모델 구조의 개발을 요구한다.

결국 토크나이저의 발전은 모델이 언어를 얼마나 더 근본적이고, 세밀하며, 효율적으로 이해할 수 있는가에 대한 질문과 직결된다. BPE는 그 질문에 대한 첫 번째 성공적인 대답이었으며, 앞으로 등장할 새로운 기술들은 BPE가 닦아놓은 길 위에서 언어 이해의 새로운 지평을 열어갈 것이다. BPE의 시대가 저물지라도, 그것이 남긴 유산과 교훈은 오랫동안 인공지능 연구의 중요한 이정표로 기억될 것이다.


1. Byte-pair encoding - Wikipedia, 8월 17, 2025에 액세스, https://en.wikipedia.org/wiki/Byte-pair_encoding
2. Byte Pair Encoding - by Akash Chandrasekar - Medium, 8월 17, 2025에 액세스, https://medium.com/@csakash03/byte-pair-encoding-c4ae347ecdb6
3. FEB94 A New Algorithm for Data Compression, 8월 17, 2025에 액세스, http://www.pennelynn.com/Documents/CUJ/HTML/94HTML/19940045.HTM
4. Byte Pair Encoding: How a 90s Compression Trick Became the Secret Sauce of LLMs | by Apurva Bhargava | Medium, 8월 17, 2025에 액세스, https://medium.com/@apurva_bhargava/byte-pair-encoding-how-a-90s-compression-trick-became-the-secret-sauce-of-llms-a2be5ee5336e
5. Mastering Byte Pair Encoding - Number Analytics, 8월 17, 2025에 액세스, https://www.numberanalytics.com/blog/mastering-byte-pair-encoding
6. Neural Machine Translation of Rare Words with Subword Units - ResearchGate, 8월 17, 2025에 액세스, https://www.researchgate.net/publication/281427452_Neural_Machine_Translation_of_Rare_Words_with_Subword_Units
7. Neural Machine Translation of Rare Words with Subword Units, 8월 17, 2025에 액세스, https://arxiv.org/abs/1508.07909
8. Subword Tokenization in NLP - GeeksforGeeks, 8월 17, 2025에 액세스, https://www.geeksforgeeks.org/nlp/subword-tokenization-in-nlp/
9. Byte-Pair Encoding tokenization - Hugging Face LLM Course, 8월 17, 2025에 액세스, https://huggingface.co/learn/llm-course/chapter6/5
10. Breaking Down Words: Byte Pair Encoding | by Yugen.ai - Medium, 8월 17, 2025에 액세스, https://medium.com/yugen-ai-technology-blog/breaking-down-words-byte-pair-encoding-a6a5c6fa2137
11. Summary of the tokenizers - Hugging Face, 8월 17, 2025에 액세스, https://huggingface.co/docs/transformers/tokenizer_summary
12. BPE Tokenization Demystified: Implementation and Examples - MartinLwx's Blog, 8월 17, 2025에 액세스, https://martinlwx.github.io/en/the-bpe-tokenizer/
13. [D] Why are Byte Pair Encoding tokenizers preferred over character level ones in LLMs?, 8월 17, 2025에 액세스, https://www.reddit.com/r/MachineLearning/comments/1ax6xuh/d_why_are_byte_pair_encoding_tokenizers_preferred/
14. Understanding Byte-Pair Encoding - Medium, 8월 17, 2025에 액세스, https://medium.com/@hsinhungw/understanding-byte-pair-encoding-fd196ebfe93f
15. Byte-Pair Encoding (BPE) in NLP - GeeksforGeeks, 8월 17, 2025에 액세스, https://www.geeksforgeeks.org/nlp/byte-pair-encoding-bpe-in-nlp/
16. Byte-Pair Encoding For Beginners | Towards Data Science, 8월 17, 2025에 액세스, https://towardsdatascience.com/byte-pair-encoding-for-beginners-708d4472c0c7/
17. Explain bpe (Byte Pair Encoding) with examples? - Stack Overflow, 8월 17, 2025에 액세스, https://stackoverflow.com/questions/50583254/explain-bpe-byte-pair-encoding-with-examples
18. A Comparative Analysis of Byte-Level and Token-Level Transformer Models in Natural Language Processing - Greg Robison, 8월 17, 2025에 액세스, https://gregrobison.medium.com/a-comparative-analysis-of-byte-level-and-token-level-transformer-models-in-natural-language-9fb4331b6acc
19. Byte-Level BPE, 8월 17, 2025에 액세스, https://www.atharvyeoelekar.blog/post/byte-level-bpe
20. Implementing A Byte Pair Encoding (BPE) Tokenizer From Scratch - Sebastian Raschka, 8월 17, 2025에 액세스, https://sebastianraschka.com/blog/2025/bpe-from-scratch.html
21. Byte Pair Encoding and Data Structures | Rust NLP tales, 8월 17, 2025에 액세스, https://guillaume-be.github.io/2021-09-16/byte_pair_encoding
22. A Formal Perspective on Byte-Pair Encoding - ACL Anthology, 8월 17, 2025에 액세스, https://aclanthology.org/2023.findings-acl.38/
23. A Formal Perspective on Byte-Pair Encoding - arXiv, 8월 17, 2025에 액세스, https://arxiv.org/html/2306.16837v2
24. (PDF) A Formal Perspective on Byte-Pair Encoding - ResearchGate, 8월 17, 2025에 액세스, https://www.researchgate.net/publication/371954434_A_Formal_Perspective_on_Byte-Pair_Encoding
25. Byte-Pair Encoding in NLP - Number Analytics, 8월 17, 2025에 액세스, https://www.numberanalytics.com/blog/byte-pair-encoding-nlp-guide
26. Byte Pair Encoding - Lei Mao's Log Book, 8월 17, 2025에 액세스, https://leimao.github.io/blog/Byte-Pair-Encoding/
27. What are the advantages and disadvantages of using subword tokenization algorithms like WordPiece and BPE? - Massed Compute, 8월 17, 2025에 액세스, [https://massedcompute.com/faq-answers/?question=What%20are%20the%20advantages%20and%20disadvantages%20of%20using%20subword%20tokenization%20algorithms%20like%20WordPiece%20and%20BPE?](https://massedcompute.com/faq-answers/?question=What+are+the+advantages+and+disadvantages+of+using+subword+tokenization+algorithms+like+WordPiece+and+BPE?)
28. What are the differences between BPE and byte-level BPE? - Data Science Stack Exchange, 8월 17, 2025에 액세스, https://datascience.stackexchange.com/questions/126715/what-are-the-differences-between-bpe-and-byte-level-bpe
29. Bit-level BPE: Below the byte boundary - arXiv, 8월 17, 2025에 액세스, https://arxiv.org/html/2506.07541v1
30. Natural Language Processing • Tokenizer - aman.ai, 8월 17, 2025에 액세스, https://aman.ai/primers/ai/tokenizer/
31. How is WordPiece tokenization helpful to effectively deal with rare words problem in NLP?, 8월 17, 2025에 액세스, https://stackoverflow.com/questions/55382596/how-is-wordpiece-tokenization-helpful-to-effectively-deal-with-rare-words-proble
32. Tokenization Is More Than Compression - arXiv, 8월 17, 2025에 액세스, https://arxiv.org/html/2402.18376v1
33. WordPiece Tokenization: A BPE Variant | by Atharv Yeolekar | Medium, 8월 17, 2025에 액세스, https://medium.com/@atharv6f_47401/wordpiece-tokenization-a-bpe-variant-73cc48865cbf
34. Vinija's Notes • Natural Language Processing • Tokenizer, 8월 17, 2025에 액세스, https://vinija.ai/nlp/tokenizer/
35. WordPiece Tokenization: What is it & how does it work? - BotPenguin, 8월 17, 2025에 액세스, https://botpenguin.com/glossary/wordpiece-tokenization
36. Linguistic Laws Meet Protein Sequences: A Comparative Analysis of Subword Tokenization Methods - arXiv, 8월 17, 2025에 액세스, https://arxiv.org/html/2411.17669v1
37. google/sentencepiece: Unsupervised text tokenizer for Neural Network-based text generation. - GitHub, 8월 17, 2025에 액세스, https://github.com/google/sentencepiece
38. Unigram tokenizer: how does it work? - Data Science Stack Exchange, 8월 17, 2025에 액세스, https://datascience.stackexchange.com/questions/88824/unigram-tokenizer-how-does-it-work
39. Unigram tokenization - Hugging Face LLM Course, 8월 17, 2025에 액세스, https://huggingface.co/learn/llm-course/chapter6/7
40. [D] SentencePiece, WordPiece, BPE... Which tokenizer is the best one? : r/MachineLearning - Reddit, 8월 17, 2025에 액세스, https://www.reddit.com/r/MachineLearning/comments/rprmq3/d_sentencepiece_wordpiece_bpe_which_tokenizer_is/
41. What changes when you randomly choose BPE merge operations? Not much. - Brandeis ScholarWorks, 8월 17, 2025에 액세스, https://scholarworks.brandeis.edu/view/pdfCoverPage?instCode=01BRAND_INST&filePid=13509587380001921&download=true
42. Getting the most out of your tokenizer for pre-training and domain adaptation - arXiv, 8월 17, 2025에 액세스, https://arxiv.org/html/2402.01035v2
43. [D] Does the vocabulary size really affect the size of textual LLMs? - Reddit, 8월 17, 2025에 액세스, https://www.reddit.com/r/MachineLearning/comments/198xx6o/d_does_the_vocabulary_size_really_affect_the_size/
44. How BPE Affects Memorization in Transformers - OpenReview, 8월 17, 2025에 액세스, https://openreview.net/forum?id=3pZTPQjeQDR
45. Training your Tokenizer Right - TU Delft Repository, 8월 17, 2025에 액세스, https://repository.tudelft.nl/file/File_f9a328ac-c0d1-45cd-99c3-e718e1f5e53c
46. Subword Secrets: The Intricacies and Impact of BPE Tokenization | by Sudhendra Kambhamettu | The Deep Hub | Medium, 8월 17, 2025에 액세스, https://medium.com/thedeephub/subword-secrets-the-intricacies-and-impact-of-bpe-tokenization-6e3e27207ff6
47. [Literature Review] Bit-level BPE: Below the byte boundary - Moonlight, 8월 17, 2025에 액세스, https://www.themoonlight.io/en/review/bit-level-bpe-below-the-byte-boundary

