바이트 페어 인코딩(BPE) 토크나이저

## 1. I. 자연어 처리와 토큰화의 패러다임

### 1.1  언어와 기계 사이의 간극: 토큰화의 근본적 필요성

자연어 처리(Natural Language Processing, NLP)의 궁극적 목표는 인간의 언어를 기계가 이해하고 처리할 수 있도록 하는 것이다. 그러나 컴퓨터는 텍스트를 인간처럼 직관적으로 인식하지 못하며, 오직 숫자, 특히 벡터(vector)와 행렬(matrix)의 형태로만 데이터를 처리할 수 있다. 이 근본적인 간극을 메우는 첫 번째이자 가장 중요한 단계가 바로 **토큰화(Tokenization)**이다.1 토큰화는 연속적인 텍스트 스트림을 의미 있는 최소 단위, 즉 **토큰(token)**으로 분할하는 과정을 의미한다.3 이 토큰들은 이후 숫자 ID로 변환되고, 임베딩(embedding)이라는 과정을 통해 고차원 벡터 공간의 한 점으로 표현되어 모델의 입력값으로 사용된다.

토큰화의 목표는 텍스트를 분석 가능한 단위로 잘게 쪼개어 기계가 언어의 구조와 의미를 학습할 수 있는 기반을 마련하는 것이다.1 예를 들어, "오늘 날씨가 좋다"라는 문장은 ["오늘", "날씨가", "좋다"]와 같은 토큰의 연속으로 변환될 수 있다.1 이렇게 분할된 토큰들은 감정 분석, 기계 번역, 챗봇, 정보 검색 등 거의 모든 NLP 응용 분야에서 핵심적인 전처리 단계로 기능한다.4 토큰화의 품질은 후속 단계인 모델의 학습 효율과 최종 성능에 직접적인 영향을 미치므로, 어떤 단위를 토큰으로 삼을 것인지는 NLP 시스템 설계의 핵심적인 결정 사항이라 할 수 있다.

### 1.2  단어, 형태소, 그리고 서브워드: 토큰화 단위의 진화

토큰화의 단위는 시대와 기술의 발전에 따라 진화해왔다. 초기의 가장 직관적인 접근법은 **단어 토큰화(Word Tokenization)**였다. 이는 공백이나 구두점을 기준으로 텍스트를 분할하는 방식으로, "Don't you love Transformers?"라는 문장을와 같이 분리한다.5 그러나 이 방식은 두 가지 치명적인 한계를 내포한다. 첫째, 훈련 데이터에 등장하는 모든 고유 단어를 어휘집(vocabulary)에 포함시켜야 하므로, 대용량 코퍼스를 다룰 때 어휘집의 크기가 기하급수적으로 커지는 **'어휘 폭발(Vocabulary Explosion)'** 문제가 발생한다.5 둘째, 훈련 데이터에 등장하지 않은 새로운 단어, 즉 **미등록 단어(Out-of-Vocabulary, OOV)**가 입력으로 들어왔을 때 이를 처리할 수 없다는 점이다.6 이러한 단어들은 보통 `<UNK>`(unknown)라는 단일 특수 토큰으로 처리되는데, 이는 단어가 가진 고유한 의미 정보를 완전히 소실시키는 결과를 초래한다.

이러한 OOV 문제에 대한 극단적인 해결책으로 **문자 토큰화(Character Tokenization)**가 등장했다. 이 방식은 텍스트를 개별 문자 단위로 분할한다.4 예를 들어 'helpful'은 ['h', 'e', 'l', 'p', 'f', 'u', 'l']로 분해된다. 문자 단위의 어휘집은 알파벳, 숫자, 기호 등을 모두 포함해도 그 크기가 매우 작기 때문에 OOV 문제가 원천적으로 발생하지 않는다.7 하지만 이 방식은 두 가지 새로운 문제를 야기한다. 첫째, 하나의 단어가 여러 개의 토큰으로 표현되므로 모델이 처리해야 할 시퀀스의 길이가 매우 길어져 계산 비용이 급증한다. 둘째, '단어'라는 기본적인 의미 단위가 파괴되어 모델이 언어의 의미론적, 구조적 정보를 학습하기가 훨씬 더 어려워진다.7

이러한 단어와 문자 토큰화의 딜레마, 즉 '표현력(expressiveness)'과 '효율성(efficiency)' 사이의 상충 관계를 해결하기 위한 절충안으로 **서브워드 토큰화(Subword Tokenization)**가 등장했다. 서브워드 토큰화는 "자주 사용되는 단어는 하나의 토큰으로 유지하되, 희귀한 단어는 의미 있는 더 작은 단위(subword)로 분해하자"는 실용적인 원칙에 기반한다.5 예를 들어, 'annoyingly'라는 비교적 희귀한 단어는 더 빈번하게 등장하는 'annoying'과 'ly'로 분리될 수 있다.5 이 접근법은 OOV 문제에 강건하면서도, 단어의 형태론적 구조를 보존하여 모델이 단어 간의 의미적 관계를 학습하는 데 도움을 준다. 이처럼 서브워드 토큰화는 고정된 어휘 크기 내에서 사실상 무한한 단어를 표현해야 하는 현대 신경망 언어 모델, 특히 트랜스포머(Transformer) 아키텍처의 성공에 필수적인 전제 조건이 되었다.

### 1.3  한국어 토큰화의 특수성과 서브워드 분절의 효용성

한국어의 토큰화는 영어와 같은 언어에 비해 훨씬 더 복잡한 과제이다. 가장 큰 이유는 한국어가 어근(root)에 조사(particle), 어미(ending) 등 다양한 문법적 기능의 형태소들이 결합하여 단어를 형성하는 **교착어(Agglutinative Language)**의 특성을 갖기 때문이다.1 예를 들어, '학교'라는 명사 어근은 '학교에', '학교가', '학교를' 등 다양한 조사가 띄어쓰기 없이 결합하여 문법적 역할을 수행한다. 따라서 영어처럼 단순히 공백을 기준으로 단어를 분리하는 것은 한국어의 구조를 제대로 반영하지 못한다.

이러한 언어적 특성 때문에 전통적인 한국어 자연어 처리에서는 의미를 가지는 최소 단위인 **형태소(morpheme)**를 분석하는 작업이 필수적이었다.1 KoNLPy와 같은 라이브러리에 포함된 형태소 분석기들은 단어를 자립 형태소(e.g., 명사, 동사)와 의존 형태소(e.g., 조사, 어미)로 분리하고 품사를 태깅하는 역할을 수행한다.1 하지만 형태소 분석기는 잘 구축된 사전에 의존하기 때문에 신조어나 오타, 비정형 텍스트에 취약하다는 한계가 있다.

바로 이 지점에서 BPE와 같은 데이터 기반의 서브워드 토큰화가 강력한 대안으로 부상한다. BPE는 언어학적 지식이나 사전에 의존하지 않고, 오직 대규모 텍스트 코퍼스에 나타나는 문자의 통계적 패턴만을 학습하여 단어를 분절한다.11 예를 들어, 코퍼스에 '먹었다', '보았다', '갔다'와 같은 단어가 빈번하게 나타난다면, BPE는 '었', '다'와 같은 문자열이 반복되는 패턴을 발견하고 이를 하나의 서브워드 단위로 병합할 가능성이 높다. 이는 형태소 분석과 유사한 결과를 데이터 기반 방식으로 달성하는 것으로, 신조어나 비정형 텍스트에 대해서도 유연하게 대처할 수 있는 잠재력을 제공한다. 이로 인해 BPE는 현대 한국어 대규모 언어 모델의 표준적인 토큰화 기법 중 하나로 자리 잡게 되었다.

## 2. II. 바이트 페어 인코딩의 기원과 원리

### 2.1  데이터 압축 알고리즘에서 자연어 처리 토크나이저로

바이트 페어 인코딩(Byte-Pair Encoding, BPE)은 본래 자연어 처리를 위해 고안된 기술이 아니었다. 이 알고리즘은 1994년 Philip Gage에 의해 발표된 논문 "A New Algorithm for Data Compression"에서 처음으로 제안된 데이터 압축 기법이었다.8 BPE 압축의 핵심 원리는 매우 간단하다. 주어진 데이터에서 가장 빈번하게 연속적으로 나타나는 바이트 쌍(pair of bytes)을 찾아, 해당 데이터에 존재하지 않는 새로운 바이트로 치환하는 과정을 반복하는 것이다.

예를 들어, `aaabdaaabac`라는 텍스트를 압축하는 과정을 살펴보자.9

1. 가장 빈번하게 등장하는 바이트 쌍은 `aa`이다 (3회). 이를 데이터에 없는 새로운 바이트 `Z`로 치환한다.
   - 데이터: `ZabdZabac`
   - 치환 테이블: `Z = aa`
2. 이제 데이터에서 가장 빈번한 쌍은 `ab`이다 (2회). 이를 `Y`로 치환한다.
   - 데이터: `ZYdZYac`
   - 치환 테이블: `Z = aa`, `Y = ab`
3. 이 과정을 더 이상 반복되는 쌍이 없을 때까지 계속할 수 있다. 예를 들어 `ZY`를 `X`로 치환하면 최종적으로 `XdXac`가 된다.

이 과정에서 원본 데이터의 길이는 11에서 5로 크게 줄어들지만, 데이터를 복원하기 위한 치환 테이블(사전)의 크기는 `a, b, c, d`에서 `a, b, c, d, Z, Y, X`로 증가하는 트레이드오프 관계가 존재한다.12

이러한 압축 알고리즘이 자연어 처리 분야에 도입된 것은 Sennrich et al. (2015, 2016)의 연구를 통해서였다.13 당시 신경망 기계 번역(Neural Machine Translation, NMT) 분야에서는 희귀 단어나 미등록 단어(OOV)를 처리하는 것이 주요 난제였다. Sennrich 연구팀은 BPE의 아이디어를 차용하여, 단어를 고정된 단위가 아닌, 데이터의 통계적 빈도에 기반한 가변적인 크기의 서브워드(subword) 단위로 분절하는 방법을 제안했다. NLP에서의 BPE는 최대 압축을 목표로 하는 것이 아니라, 사용자가 미리 지정한 어휘 크기(vocabulary size) 내에서 텍스트를 가장 효율적으로 표현하는 서브워드의 집합을 찾는 것을 목표로 한다는 점에서 원본 알고리즘과 목적이 다르다.14 이 아이디어는 OOV 문제를 효과적으로 해결하며 현대 대규모 언어 모델 토큰화의 표준으로 자리 잡는 계기가 되었다.

### 2.2  핵심 메커니즘: 빈도 기반의 반복적 쌍 병합

BPE 토크나이저의 작동 원리는 한 문장으로 요약할 수 있다: **"코퍼스에서 가장 빈번하게 연속으로 등장하는 토큰 쌍을 찾아 하나의 새로운 토큰으로 병합하는 과정을 반복하는 것"**이다.11 이 메커니즘은 몇 가지 중요한 특징을 가진다.

첫째, BPE는 **가산적(additive)** 접근 방식을 취한다.19 학습 초기에는 어휘집이 텍스트에 등장하는 모든 개별 문자(character)로만 구성된다. 이후 병합 과정을 거치면서 두 문자가 합쳐진 서브워드, 더 나아가 여러 서브워드가 합쳐진 완전한 단어에 이르는 더 큰 단위의 토큰들이 점진적으로 어휘집에 추가된다. 이는 매우 작은 단위에서 시작하여 점차 복잡한 단위를 구축해나가는 상향식(bottom-up) 방식이다.

둘째, BPE는 **탐욕적(greedy)**이고 **결정론적인(deterministic)** 알고리즘이다. 각 병합 단계에서 알고리즘은 오직 현재 상태에서 가장 빈도가 높은 쌍만을 고려하여 병합을 결정한다. 이 결정이 미래에 더 나은 분절 결과를 가져올지에 대해서는 고려하지 않는다. 일단 병합 규칙이 학습되고 나면, 새로운 텍스트에 대한 토큰화는 항상 동일한 규칙에 따라 유일한 결과물을 생성한다. 이러한 단순성과 결정성은 BPE를 빠르고 구현하기 쉽게 만들지만, 동시에 의미론적으로 최적이 아닌 분절을 만들어내는 원인이 되기도 한다.

이러한 BPE의 원리는 정보 이론의 압축 개념과 언어의 통계적 특성 사이의 깊은 연관성을 시사한다. 언어는 'Zipf의 법칙'에서 알 수 있듯이 소수의 단어와 구문이 매우 빈번하게 사용되는 통계적 중복성을 내포하고 있다. BPE는 바로 이 언어의 통계적 중복성을 활용하여 가장 빈번한 문자열 패턴(형태소, 단어의 일부 등)을 하나의 단위로 '압축'(토큰화)함으로써, 언어학적 지식 없이도 효율적인 언어 표현을 데이터로부터 직접 학습하는 강력한 메커니즘을 제공한다.

## 3. III. BPE 알고리즘 심층 분석

### 3.1  어휘 집합 구축 과정: 단계별 상세 해설

BPE 토크나이저의 학습, 즉 어휘 집합을 구축하는 과정은 크게 네 단계로 구성된다. 여기서는 `hug pug pun bun hugs`라는 단어 집합과 각 단어의 빈도 `("hug", 10), ("pug", 5), ("pun", 12), ("bun", 4), ("hugs", 5)`를 예시로 사용하여 각 단계를 상세히 설명한다.18

#### 3.1.1 단계: 전처리(Pre-tokenization) 및 단어 빈도 계산

본격적인 BPE 학습에 앞서, 입력 코퍼스를 단어와 유사한 단위로 미리 분할하는 **전처리(Pre-tokenization)** 과정이 선행된다.12 이 과정은 주로 공백이나 구두점을 기준으로 텍스트를 분리하며, BPE의 병합 연산이 이 전처리된 단어의 경계를 넘어가지 않도록 하는 역할을 한다. 예시에서는 이미 단어 단위로 분리되고 빈도가 계산된 상태를 가정한다.

#### 3.1.2 단계: 초기 어휘 집합(Vocabulary) 구성

전처리된 단어들을 기반으로 초기 어휘 집합을 구성한다.

1. **초기 어휘 생성:** 코퍼스에 등장하는 모든 고유 문자(character)를 수집하여 초기 어휘 집합을 만든다. 예시의 경우, `b, g, h, n, p, s, u` 7개의 문자가 초기 어휘가 된다.12

2. **단어 분할 및 특수 기호 추가:** 각 단어를 문자 시퀀스로 분해한다. 이때, 단어의 경계를 명확히 하기 위해 단어의 끝을 나타내는 특수 기호(예: `</w>` 또는 `_`)를 추가하는 경우가 많다.7 이는 'est' (e.g., in 'estimate')와 'est

   ' (e.g., in 'highest')를 구분하여, 동일한 서브워드라도 단어 내 위치에 따라 다른 의미를 가질 수 있음을 모델이 학습하도록 돕는다.9

   예시에서는 편의상 이 기호를 생략하고 진행하지만, 실제 구현에서는 매우 중요한 단계이다. 분할 결과는 다음과 같다.

   - `("h", "u", "g")`: 10회
   - `("p", "u", "g")`: 5회
   - `("p", "u", "n")`: 12회
   - `("b", "u", "n")`: 4회
   - `("h", "u", "g", "s")`: 5회

#### 3.1.3 단계: 반복적인 쌍 병합(Iterative Merging)

이 단계는 BPE 알고리즘의 핵심으로, 사용자가 지정한 병합 횟수(num_merges) 또는 목표 어휘 크기(vocab_size)에 도달할 때까지 다음 과정을 반복한다.11

- **반복 1:**

  1. **가장 빈번한 쌍 찾기:** 현재 분할 상태에서 인접한 모든 토큰 쌍(바이그램)의 빈도를 계산한다.

     - `(u, g)`: 10 (from "hug") + 5 (from "pug") + 5 (from "hugs") = **20회**

     - `(p, u)`: 5 (from "pug") + 12 (from "pun") = 17회

     - `(u, n)`: 12 (from "pun") + 4 (from "bun") = 16회

     - `(h, u)`: 10 (from "hug") + 5 (from "hugs") = 15회

     - ... (기타)

       가장 빈번한 쌍은 (u, g)이다.18

  2. **병합 및 어휘 업데이트:** `(u, g)`를 새로운 토큰 `ug`로 병합하고, 이를 어휘 집합에 추가한다.

     - 어휘 집합: `{b, g, h, n, p, s, u, ug}`

  3. **단어 분할 상태 업데이트:** 코퍼스 내 모든 `(u, g)`를 `ug`로 치환한다.

     - `("h", "ug")`: 10회
     - `("p", "ug")`: 5회
     - `("p", "u", "n")`: 12회
     - `("b", "u", "n")`: 4회
     - `("h", "ug", "s")`: 5회

- **반복 2:**

  1. **가장 빈번한 쌍 찾기:** 업데이트된 상태에서 다시 쌍의 빈도를 계산한다.

     - `(u, n)`: 12 (from "pun") + 4 (from "bun") = **16회**

     - `(h, ug)`: 10 (from "hug") + 5 (from "hugs") = 15회

     - ... (기타)

       이제 가장 빈번한 쌍은 (u, n)이다.

  2. **병합 및 어휘 업데이트:** `(u, n)`을 `un`으로 병합하고 어휘 집합에 추가한다.

     - 어휘 집합: `{b, g, h, n, p, s, u, ug, un}`

  3. **단어 분할 상태 업데이트:**

     - `("h", "ug")`: 10회
     - `("p", "ug")`: 5회
     - `("p", "un")`: 12회
     - `("b", "un")`: 4회
     - `("h", "ug", "s")`: 5회

이러한 병합 과정은 목표 어휘 크기에 도달할 때까지 계속된다.

#### 3.1.4 단계: 최종 어휘 집합 및 병합 규칙(Merge Rules) 저장

학습이 완료되면, 최종적으로 생성된 어휘 집합과 함께 병합이 일어난 순서를 기록한 **병합 규칙(merge rules)** 리스트가 저장된다.12 이 병합 규칙은 우선순위를 나타내며, 새로운 텍스트를 토큰화할 때 낮은 순위(먼저 학습된)의 병합부터 순차적으로 적용된다. 예를 들어, 병합 규칙이 `[(u, g), (u, n), (h, ug),...]` 순서로 저장되었다면, 새로운 텍스트를 토큰화할 때도 이 순서대로 병합을 시도하게 된다.

### 3.2  의사 코드 및 수학적 공식화

BPE 학습 알고리즘의 절차는 의사 코드(pseudo-code)를 통해 보다 명확하고 구조적으로 표현할 수 있다. 의사 코드는 특정 프로그래밍 언어에 종속되지 않으면서 알고리즘의 논리적 흐름을 기술하는 데 효과적이다.23

#### 3.2.1 BPE 알고리즘 의사 코드 (학습 단계)

```
Algorithm: BPE_Train(corpus, vocab_size)
Input: 텍스트 코퍼스 C, 목표 어휘 크기 V_size
Output: 어휘 집합 V, 병합 규칙 M

1.  // 1단계 & 2단계: 초기화
2.  word_counts ← Pre-tokenize(C)를 통해 단어별 빈도 계산
3.  splits ← 각 단어를 문자 시퀀스로 분할 (e.g., "hug" -> ["h", "u", "g"])
4.  V ← corpus에 있는 모든 고유 문자의 집합
5.  M ← // 비어있는 병합 규칙 리스트
6.
7.  // 3단계: 반복적 병합
8.  num_merges = vocab_size - |V|
9.  FOR i FROM 1 TO num_merges:
10.     // 모든 인접 토큰 쌍의 빈도 계산
11.     pair_counts ← Compute_Pair_Frequencies(splits, word_counts)
12.
13.     IF pair_counts is empty THEN BREAK // 더 이상 병합할 쌍이 없음
14.
15.     // 가장 빈번한 쌍 선택
16.     best_pair ← argmax_{p}(pair_counts[p])
17.     new_token ← merge(best_pair.first, best_pair.second)
18.
19.     // 어휘 집합 및 병합 규칙 업데이트
20.     V.add(new_token)
21.     M.append(best_pair)
22.
23.     // 코퍼스 내 모든 best_pair를 new_token으로 치환
24.     splits ← Merge_In_Corpus(splits, best_pair, new_token)
25.
26. // 4단계: 결과 반환
27. RETURN V, M
```

이러한 알고리즘적 접근을 넘어, BPE는 **조합 최적화(combinatorial optimization)** 문제로도 공식화될 수 있다.27 이 관점에서 BPE의 목표는 주어진 코퍼스를 토큰화했을 때 최종 토큰 시퀀스의 총 길이를 최소화하는 최적의 병합 순서(optimal merge sequence)를 찾는 것이다. 텍스트를 바이트 시퀀스 $𝐛 = b₁b₂...bₙ$로, 병합 규칙 적용을 연산자 $τ$로 표기하여 과정을 수학적으로 모델링할 수 있다.29

그러나 BPE의 탐욕적(greedy) 접근 방식은 각 단계에서 국소 최적(locally optimal)의 해, 즉 가장 빈번한 쌍을 선택할 뿐, 전체 과정에 걸친 전역 최적(globally optimal)의 해를 보장하지는 않는다. 이는 BPE가 근본적으로 근사 알고리즘(approximation algorithm)임을 의미한다. 최근 연구에서는 이 문제를 **준모듈 함수(submodular function)** 최적화의 관점에서 분석하여, BPE의 탐욕적 버전이 특정 조건 하에서 이론적인 근사 비율(approximation ratio)을 보장함을 증명하기도 했다.27 이 근사 비율은 최적 병합 순서에 대한 총 후방 곡률(total backward curvature) 

$σ(μ^⋆)$에 의존하며, 수식으로는 $ \frac{1}{{\sigma(\boldsymbol{\mu}^\star)}}(1-e^{-{\sigma(\boldsymbol{\mu}^\star)}}) $로 표현된다. 이는 BPE의 성능이 이론적으로 어느 정도 보장됨을 시사하며, 알고리즘의 경험적 성공에 대한 이론적 기반을 제공한다.

### 3.3  구현상 고려사항: 전처리(Pre-tokenization)와 특수 토큰의 역할

BPE 알고리즘의 단순성 이면에는 실제 언어의 복잡성을 다루기 위한 여러 구현상의 고려사항이 존재한다.

- **전처리(Pre-tokenization):** BPE는 기본적으로 전처리 단계에서 분리된 단어의 경계를 넘어서는 병합을 수행하지 않는다.12 따라서 이 초기 분할 방식이 최종 토큰화 결과에 지대한 영향을 미친다. 예를 들어, "New York"을 하나의 단위로 전처리하면 "NewYork"이라는 토큰이 학습될 수 있지만, "New"와 "York"으로 분리하면 "New"와 "York"이 각각 또는 그 일부가 토큰으로 학습될 것이다. 특히 GPT-2와 같은 모델에서는 공백을 

  `Ġ` (U+0120)라는 특수 문자로 치환하는 방식을 사용한다.17 이로써 " a"와 "a"가 다른 토큰으로 취급되어 단어의 시작 정보를 명시적으로 보존할 수 있게 된다.

- **바이트 수준 BPE(Byte-level BPE):** 다국어 텍스트를 처리할 때, 모든 유니코드 문자를 초기 어휘집에 포함하면 그 크기가 매우 커질 수 있다. 이 문제를 해결하기 위해 텍스트를 유니코드 문자가 아닌 UTF-8 바이트 스트림으로 간주하고, 256개의 모든 가능한 바이트 값을 초기 어휘로 사용하는 방식이 제안되었다.14 이 방식은 어휘 크기를 256으로 고정하면서도 어떠한 언어나 기호도 처리할 수 있게 해주며, 미등록 단어(

  `UNK`) 토큰을 원천적으로 방지하는 매우 강력하고 우아한 해결책이다.14

- **특수 토큰(Special Tokens):** 언어 모델은 텍스트 내용 외에도 문장의 시작(`), 끝(`), 분류(`), 분리(`), 마스킹(``) 등 다양한 제어 신호를 필요로 한다. 이러한 특수 토큰들은 BPE 학습 과정과는 별개로 어휘집에 명시적으로 추가되어야 하며, 모델이 특정 작업을 수행하도록 지시하는 데 사용된다.35

이처럼 BPE는 이론적으로는 단순한 탐욕적 알고리즘이지만, 전처리, 바이트 처리, 특수 토큰과 같은 실용적인 장치들과 결합하여 실제 언어 데이터에 효과적으로 작동하는 강력한 공학적 산물로 완성된다.

## 4. IV. BPE 토크나이저의 실제: 장점, 단점, 그리고 인공물(Artifacts)

### 4.1  장점: OOV 문제 해결 및 가변적 어휘 크기

BPE 토크나이저가 현대 자연어 처리 모델의 표준으로 자리 잡은 데에는 몇 가지 명확한 장점이 있다.

첫째, BPE는 **미등록 단어(OOV) 문제를 효과적으로 해결한다**. 훈련 데이터에 등장하지 않은 새로운 단어나 희귀한 단어가 입력되더라도, BPE는 이를 학습된 서브워드 단위로 분해하고, 최악의 경우 개별 문자 단위로까지 분할하여 표현할 수 있다.10 이로 인해 이론적으로 OOV 문제가 발생하지 않으며, 모델은 어떠한 입력에 대해서도 의미 있는 표현을 생성할 수 있는 **강건성(robustness)**을 확보하게 된다.29 이는 신조어, 오타, 전문 용어가 빈번하게 등장하는 실제 환경에서 모델의 안정적인 성능을 보장하는 핵심적인 특징이다.

둘째, BPE는 **어휘 크기를 유연하게 제어할 수 있다**. 어휘집의 크기는 병합 횟수(number of merges)라는 하이퍼파라미터를 통해 사용자가 직접 설정할 수 있다.11 작은 어휘 크기는 모델의 파라미터 수를 줄여 경량화에 유리하지만, 텍스트를 더 잘게 분할하여 시퀀스 길이를 늘릴 수 있다. 반면, 큰 어휘 크기는 더 많은 단어를 통째로 표현하여 시퀀스 길이를 줄이고 표현력을 높이지만, 모델의 크기와 메모리 사용량을 증가시킨다. 이처럼 BPE는 모델의 크기와 성능, 그리고 계산 비용 사이의 균형을 특정 과업의 요구사항에 맞게 조절할 수 있는 유연성을 제공한다.

셋째, BPE는 텍스트를 효율적으로 **압축**하는 효과가 있다. 자주 사용되는 긴 단어나 구문은 더 적은 수의 토큰으로 표현된다.32 예를 들어 "tokenization"이라는 단어는 문자 토큰화 시 12개의 토큰이 필요하지만, BPE를 통해 학습된 토크나이저에서는 "token"과 "ization" 두 개의 토큰으로 표현될 수 있다. 이렇게 전체 시퀀스 길이가 줄어들면 트랜스포머와 같이 시퀀스 길이에 따라 계산량이 이차적으로 증가하는 모델의 계산 효율성을 크게 향상시킬 수 있다.13

### 4.2  단점과 한계: 의미론적 모호성과 비직관적 분절 사례 분석

BPE의 장점은 명확하지만, 그 이면에는 통계적 빈도에만 의존하는 탐욕적 알고리즘의 본질적인 한계가 존재한다.

가장 근본적인 단점은 **의미론적 불일치**이다. BPE는 오직 문자의 등장 빈도만을 기준으로 병합을 결정하기 때문에, 생성된 서브워드가 언어학적인 형태소(morpheme)나 의미 단위와 일치하지 않는 경우가 빈번하게 발생한다.38 이는 BPE가 언어의 구조나 의미를 전혀 이해하지 못하고 기계적으로 작동하기 때문에 발생하는 필연적인 결과이다.

이러한 한계는 다음과 같은 **비직관적 분절(Unintuitive Segmentation)** 사례에서 명확하게 드러난다.

- `alphabet` → `alph` + `abet` 39: 이 경우, 

  `alphabet`이라는 단어는 원래 의미와 전혀 관련이 없는 두 개의 서브워드로 분해된다. 모델은 `alph`와 `abet`라는 토큰으로부터 `alphabet`의 의미를 재구성해야 하는 어려운 과제에 직면하게 된다.

- `GPU!` → `gp` + `##u` + `!` (BERT의 WordPiece 예시, BPE에서도 유사 현상 발생) 5: 'Graphics Processing Unit'이라는 명확한 의미를 가진 기술 용어가 의미 없는 단위로 분해된다.

- `Don't` → `Don` + `'` + `t` 5: 'do not'의 축약형이라는 문법적 구조가 무시되고, 단순히 문자열 패턴에 따라 분리된다. 'Do'와 'n't'라는 의미 단위가 보존되지 않는다.

또 다른 한계는 **결정론적 분절**이다. 학습된 BPE 토크나이저는 주어진 텍스트에 대해 항상 유일하고 동일한 토큰 시퀀스를 생성한다. 이는 일관성을 보장하지만, 모델이 하나의 단어가 가질 수 있는 다양한 형태론적 해석이나 구성을 학습할 기회를 제한할 수 있다.40 예를 들어, 'undoing'이라는 단어는 'un-doing'으로도, 'undo-ing'으로도 해석될 수 있지만, BPE는 이 중 단 하나의 분절 방식만을 강제한다.

### 4.3  훈련 데이터 편향이 토큰화에 미치는 영향

BPE 어휘집은 전적으로 훈련 코퍼스의 통계적 특성을 반영하는 거울과 같다. 따라서 훈련 데이터에 내재된 편향은 토크나이저에 그대로 각인되며, 이는 모델의 성능과 공정성에 심각한 영향을 미칠 수 있다.

- **언어 불균형과 공정성 문제:** 현대의 대규모 언어 모델은 주로 인터넷에서 수집한 방대한 텍스트로 훈련되는데, 이 데이터는 압도적으로 영어가 많다. 이러한 데이터로 학습된 BPE 토크나이저는 영어를 매우 효율적으로, 즉 적은 수의 토큰으로 표현한다. 반면, 데이터에 적게 포함된 다른 언어들은 제대로 된 서브워드를 학습하지 못해 매우 긴 문자 단위 토큰 시퀀스로 분해되는 경향이 있다.41 연구에 따르면 동일한 내용의 텍스트라도 이탈리아어는 영어보다 1.6배, 아랍어는 3배 더 많은 토큰을 필요로 할 수 있다.41 이는 비영어권 사용자들이 토큰당 과금되는 API를 사용할 때 더 많은 비용을 지불하게 만들고, 모델의 고정된 컨텍스트 길이 내에서 더 적은 정보만을 처리할 수 있게 하는 등 심각한 기술적, 경제적 불평등을 야기한다.30
- **글리치 토큰(Glitch Tokens):** 훈련 데이터에 위키피디아, 레딧, 게임 포럼 등 특정 커뮤니티의 텍스트가 다수 포함될 경우, 언어적으로는 의미가 없지만 해당 커뮤니티에서 자주 사용되는 사용자 ID, 게임 아이템 이름, 인터넷 밈 등이 하나의 완전한 토큰으로 학습될 수 있다.41 예를 들어, 'SolidGoldMagikarp'와 같은 문자열이 반복적으로 나타나면 BPE는 이를 하나의 토큰으로 병합할 수 있다. 이러한 '글리치 토큰'은 어휘집의 공간을 비효율적으로 차지하며, 모델의 예측을 예기치 않은 방향으로 왜곡할 수 있는 잠재적 위험을 내포한다.

결론적으로, BPE 토크나이저의 기술적 특성은 훈련 데이터의 사회적, 문화적 맥락과 분리될 수 없다. 데이터의 편향은 토큰화의 비효율과 불균형을 낳고, 이는 다시 모델 성능의 불평등으로 이어지는 연쇄 효과를 낳는다. 따라서 더 나은 토크나이저를 개발하는 것은 단순한 기술 최적화를 넘어, 공정하고 포용적인 AI를 구축하기 위한 핵심적인 과제라 할 수 있다.

## 5. V. 주요 서브워드 토크나이저 비교 분석

BPE는 서브워드 토큰화의 시대를 열었지만, 유일한 해결책은 아니다. BPE의 한계를 극복하기 위해 다양한 대안 알고리즘이 제안되었으며, 그중 WordPiece와 Unigram 언어 모델이 가장 대표적이다. 이들과의 비교를 통해 BPE의 특징을 더욱 명확히 이해할 수 있다.

### 5.1  BPE 대 WordPiece: 병합 기준의 차이 - '가장 흔한 쌍'과 '가장 가치 있는 쌍'

WordPiece는 BERT를 위해 구글에서 개발한 토큰화 알고리즘으로, BPE와 매우 유사한 가산적(additive) 방식을 사용하지만 핵심적인 차이점을 가진다.10

- **병합 기준:** BPE가 단순히 가장 **빈도(frequency)**가 높은 인접 쌍을 병합하는 반면, WordPiece는 병합되었을 때 전체 훈련 코퍼스의 **우도(Likelihood)**를 가장 크게 증가시키는 쌍을 병합한다.43

- **수식적 표현:** WordPiece가 사용하는 점수(score)는 다음과 같이 계산될 수 있다.42
  $$
  \mathrm{score} = \frac{\mathrm{freq}(\mathrm{pair})}{\mathrm{freq}(\mathrm{first}) \times \mathrm{freq}(\mathrm{second})}
  $$
  이 수식은 두 토큰이 독립적으로 나타날 확률 대비 함께 나타날 확률, 즉 점별 상호 정보량(Pointwise Mutual Information)과 유사한 개념을 측정한다. 점수가 높다는 것은 두 토큰이 우연히 함께 나타난 것이 아니라 강한 연관성을 가지고 있음을 의미한다.

- **직관적 해석:** BPE가 '가장 흔하게 만나는 커플'을 맺어준다면, WordPiece는 '결합했을 때 시너지가 가장 큰 커플'을 맺어주는 것과 같다. 즉, WordPiece는 두 기호를 병합함으로써 잃게 되는 정보(각 기호가 독립적으로 나타날 수 있는 가능성)를 평가하여, 그 손실을 감수할 만큼 병합의 가치가 있는지를 따진다.44 이로 인해 WordPiece는 BPE보다 언어학적으로 더 의미 있는(예: 형태소 경계와 일치하는) 분절을 생성하는 경향이 있다.46

- **구현상 차이:** WordPiece는 단어의 시작 부분이 아닌 서브워드 앞에 `##`와 같은 특별한 접두사를 붙여 단어 내부의 토큰임을 명시적으로 표시한다. 예를 들어, `playing`은 `play`와 `##ing`로 토큰화된다.33

### 5.2  BPE 대 Unigram 언어 모델: 결정론적 접근과 확률론적 접근

Unigram 언어 모델 토크나이저는 BPE, WordPiece와는 근본적으로 다른 철학을 가진다.

- **접근 방식:** BPE가 작은 단위(문자)에서 시작하여 어휘를 구축하는 **'가산적(additive)'** 방식인 반면, Unigram은 가능한 모든 서브워드를 포함한 거대한 초기 어휘집에서 시작하여 점차 불필요한 토큰을 제거해 나가는 **'감산적(subtractive)'** 방식을 사용한다.19
- **토큰화 방식:** BPE는 학습된 병합 규칙에 따라 항상 유일하고 **결정론적인(deterministic)** 토큰 시퀀스를 생성한다. 그러나 Unigram은 각 서브워드가 생성될 확률을 언어 모델을 통해 학습한다. 새로운 문장을 토큰화할 때는, 가능한 모든 분절 조합을 고려하여 그중 전체 확률(likelihood)이 가장 높은 시퀀스를 비터비(Viterbi) 알고리즘과 같은 탐색 기법을 통해 찾아낸다. 이는 하나의 단어가 문맥에 따라 여러 방식으로 토큰화될 수 있는 **확률론적(probabilistic)** 토큰화를 가능하게 한다.19 이 특징은 모델 학습 시 매번 다른 분절을 샘플링하여 주입하는 '서브워드 정규화(Subword Regularization)' 기법의 기반이 되어 모델의 강건성을 높이는 데 사용된다.
- **최적화 기준:** Unigram의 어휘 축소 과정 역시 코퍼스의 전체 우도를 최대화하는 방향으로 진행된다. 각 단계에서 현재 어휘집에 있는 토큰을 하나씩 제거해 보았을 때, 전체 우도의 손실(loss)이 가장 적게 발생하는 토큰부터 순차적으로 제거하여 최종 목표 어휘 크기에 도달한다.19

### 5.3  SentencePiece: 통합 프레임워크로서의 역할과 공백 처리 방식

SentencePiece는 특정 알고리즘의 이름이라기보다는, 구글에서 개발한 오픈소스 토크나이저 라이브러리의 이름이다. 이 라이브러리는 BPE와 Unigram 알고리즘을 모두 구현하고 있으며, T5와 같은 여러 모델에서 표준적으로 사용된다.10

SentencePiece의 가장 혁신적인 특징은 **전처리(pre-tokenization) 과정 없이** 원시 텍스트(raw text)에서 직접 어휘를 학습하고 토큰화를 수행한다는 점이다.51 기존의 BPE나 WordPiece 구현체들은 학습 전에 텍스트를 공백이나 특정 규칙에 따라 단어 단위로 분리해야 하는 언어 종속적인 전처리 과정을 요구했다.

SentencePiece는 이 문제를 해결하기 위해 **공백(whitespace)을 일반 문자처럼 취급**한다. 공백 문자를 ` ` (U+2581)라는 특수한 메타 심볼로 치환한 뒤, 전체 텍스트를 하나의 거대한 문자 시퀀스로 보고 서브워드 분절을 수행한다.51 예를 들어, "Hello world."는 "Hello world."로 변환된 후 토큰화된다. 이렇게 하면 토큰화된 시퀀스 `` 안에 공백 정보가 그대로 보존되므로, 단순히 토큰들을 이어 붙이고 

` `를 실제 공백으로 되돌리기만 하면 원본 문장이 손실 없이 완벽하게 복원(detokenization)된다. 이 방식은 중국어나 일본어처럼 단어 간 공백이 없는 언어를 처리할 때 매우 효과적이며, 토큰화 파이프라인에서 언어 종속적인 전처리기를 제거하여 시스템을 단순화하고 다국어 처리를 용이하게 만든다.

### 5.4 [표 1] 주요 서브워드 토크나이저 비교

| 구분            | **Byte-Pair Encoding (BPE)**                         | **WordPiece**                                                | **Unigram Language Model**                                   |
| --------------- | ---------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **접근 방식**   | 가산적 (Additive): 문자에서 시작해 병합              | 가산적 (Additive): 문자에서 시작해 병합                      | 감산적 (Subtractive): 큰 어휘에서 시작해 제거                |
| **핵심 기준**   | **빈도 (Frequency):** 가장 빈번한 쌍을 병합          | **우도 (Likelihood):** 병합 시 코퍼스 우도를 최대화하는 쌍을 선택 | **우도 (Likelihood):** 제거 시 우도 감소가 가장 적은 토큰을 제거 |
| **토큰화 방식** | 결정론적 (Deterministic): 학습된 병합 규칙 순차 적용 | 결정론적 (Deterministic): 어휘 내 가장 긴 서브워드 우선 매칭 | 확률론적 (Probabilistic): Viterbi 알고리즘으로 최적 분할 탐색 |
| **주요 특징**   | 알고리즘이 간단하고 빠름                             | `##` 접두사로 단어 내부 토큰 표시                            | 다중 분할 생성 가능 (Subword Regularization)                 |
| **대표 모델**   | GPT-2, RoBERTa, XLM                                  | BERT, DistilBERT, Electra                                    | T5, ALBERT, XLNet (SentencePiece 통해 사용)                  |

## 6. VI. BPE의 진화와 미래: 개선 및 대안

BPE는 그 자체로 강력한 도구이지만, 내재된 한계를 극복하기 위한 다양한 개선 연구와 심지어 토크나이저 자체를 없애려는 근본적인 대안들이 활발히 연구되고 있다.

### 6.1  BPE 개선 연구: BPE-Dropout, AG-BPE를 통한 한계 극복 시도

BPE의 본질적인 한계, 즉 결정론적 분절과 의미론적 무지를 해결하기 위한 연구들이 진행되어 왔다.

- **BPE-Dropout:** 이 기법은 BPE의 **결정론적 분절** 문제를 완화하기 위해 제안되었다. 일반적인 BPE는 학습된 병합 규칙에 따라 항상 동일한 결과를 내놓는다. BPE-Dropout은 모델 학습(training) 중에 토큰화를 수행할 때, 각 병합 단계를 정해진 확률(dropout rate)에 따라 무작위로 건너뛰는 방식을 도입한다.37 이로 인해 동일한 단어라도 학습 시마다 

  `[play, ing]` 또는 `[p, l, a, y, ing]`과 같이 다양한 방식으로 분절될 수 있다. 이는 모델이 단어의 다양한 구성 방식을 학습하게 하여 강건성을 높이는 일종의 데이터 증강(data augmentation) 및 정규화(regularization) 효과를 가져온다.

- **Attention-Guided BPE (AG-BPE):** 이 연구는 BPE의 **의미론적 무지(semantic unawareness)**를 해결하려는 시도이다.38 기존 BPE는 오직 통계적 빈도에만 의존하지만, AG-BPE는 여기에 의미론적 정보를 추가한다. 구체적으로, 트랜스포머 모델을 학습시키면서 얻어지는 어텐션(attention) 점수를 병합 기준에 통합한다. 어텐션 점수는 단어 또는 토큰 간의 의미적 연관성을 나타내므로, 어텐션 점수가 높은 쌍, 즉 의미적으로 강하게 연결된 쌍을 우선적으로 병합하도록 유도한다. 예를 들어, 

  `(do, ing)` 쌍은 빈도도 높고 의미적으로도 강하게 연결되어 있으므로 높은 병합 점수를 받게 된다. 이 방식을 통해 순수 빈도 기반 BPE보다 더 형태론적으로, 그리고 의미론적으로 일관된 어휘집을 구축하고자 한다.

### 6.2  토크나이저-프리 모델: ByT5와 바이트 수준 접근법의 부상

BPE를 개선하는 것을 넘어, 토크나이저 자체를 완전히 제거하려는 근본적인 패러다임 전환도 시도되고 있다. 이러한 **토크나이저-프리(Tokenizer-Free)** 모델들은 서브워드 토크나이저가 야기하는 기술적 부채(복잡한 전처리 파이프라인), 다국어 처리의 불공정성, 노이즈에 대한 취약성 등의 문제를 근본적으로 해결하는 것을 목표로 한다.49

대표적인 모델인 **ByT5**는 T5 모델을 기반으로, 토크나이저를 거치지 않고 원시 UTF-8 바이트 스트림을 직접 모델의 입력으로 받는다.34

- **장점:**

  - **완벽한 보편성:** 어떠한 언어나 기호도 UTF-8 바이트로 표현 가능하므로, OOV 문제가 원천적으로 사라지고 진정한 의미의 다국어 처리가 가능하다.34
  - **강건성:** 오타나 노이즈가 포함된 텍스트도 별도의 처리 없이 그대로 입력받아 처리할 수 있어 매우 강건하다.54
  - **단순성 및 효율성:** 복잡한 토큰화 파이프라인이 필요 없으며, 어휘집의 크기가 256개의 바이트와 몇 개의 특수 토큰으로 고정되므로, 거대한 어휘 임베딩 행렬에 필요한 파라미터가 극적으로 감소한다.55

- **단점:**

  - **계산 비용:** 가장 치명적인 단점은 입력 시퀀스의 길이가 폭발적으로 증가한다는 것이다. 바이트 시퀀스는 서브워드 시퀀스보다 평균적으로 4배 이상 길다.55 트랜스포머 아키텍처의 어텐션 메커니즘은 시퀀스 길이에 대해 이차적(

    $O(n^2)$)인 계산 복잡도를 가지므로, 시퀀스 길이가 길어지면 훈련 및 추론에 필요한 계산 비용과 메모리가 기하급수적으로 증가한다.56

  - 이러한 계산 비효율 문제를 완화하기 위해 ByT5는 인코더의 깊이를 디코더보다 3배 더 깊게 설계하는 비대칭 구조를 채택했으며 55, MrT5와 같은 후속 연구에서는 학습된 게이트를 통해 불필요한 바이트 토큰을 동적으로 삭제하여 시퀀스 길이를 줄이는 메커니즘을 도입하는 등 효율성 개선을 위한 연구가 계속되고 있다.56

### 6.3  최신 언어 모델의 토크나이저 동향: GPT-4o의 'o200k_base'를 중심으로

토크나이저-프리 모델이 장기적인 연구 방향을 제시하는 한편, 현재의 주류 대규모 언어 모델(LLM)들은 BPE의 기본 틀을 유지하면서도 그 한계를 보완하는 실용적인 방향으로 진화하고 있다. 가장 두드러진 경향은 **어휘 크기의 대폭적인 증가**이다.

GPT-2의 어휘 크기는 약 5만 개(50,257)였고, GPT-3/4에서 사용된 `cl100k_base`는 약 10만 개(100,256)였다.14 그리고 2024년에 공개된 최신 모델인 GPT-4o는 **`o200k_base`**라는 이름에서 유추할 수 있듯이 약 20만 개(200,019)에 달하는 어휘를 사용한다.38

어휘 크기를 이렇게 대폭적으로 늘리는 이유는 다음과 같다.

1. **다국어 및 코드 처리 효율 향상:** 더 큰 어휘집은 더 많은 언어의 자주 사용되는 단어나 형태소, 그리고 C++, Python과 같은 프로그래밍 언어의 예약어, 함수명, 구문 요소들을 하나의 완전한 토큰으로 포함할 수 있게 한다.41 이는 비영어 텍스트와 소스 코드가 불필요하게 긴 문자 단위 시퀀스로 분해되는 것을 막아 토큰화 효율을 높이고, 결과적으로 앞서 언급된 언어 불균형 및 공정성 문제를 완화하는 효과를 가져온다.
2. **텍스트 압축률 증가:** 어휘집에 더 길고 복잡한 단위가 포함될수록, 동일한 텍스트를 더 적은 수의 토큰으로 표현할 수 있게 된다. 즉, 텍스트 압축률이 높아진다.38 이는 모델이 고정된 컨텍스트 길이(context window) 내에서 더 많은 실제 정보를 처리할 수 있게 함을 의미하며, 이는 장문 이해나 복잡한 추론 작업에서 모델의 성능을 향상시키는 데 직접적으로 기여한다.

이러한 경향은 BPE의 근본적인 한계를 인정하면서도, 계산 효율성이라는 현실적인 제약 속에서 어휘 크기 확장을 통해 그 단점을 보완하려는 지극히 실용적인 공학적 절충안이라 할 수 있다.

### 6.4 [표 2] 토큰 기반 모델과 바이트 수준 모델 비교

| 구분               | **토큰 기반 모델 (e.g., GPT, BERT)**           | **바이트 수준 모델 (e.g., ByT5)**            |
| ------------------ | ---------------------------------------------- | -------------------------------------------- |
| **기본 처리 단위** | 서브워드 (Subword)                             | 바이트 (Byte)                                |
| **어휘 크기**      | 큼, 가변적 (e.g., 30k ~ 200k)                  | 작음, 고정 (256 + 특수 토큰)                 |
| **시퀀스 길이**    | 상대적으로 짧음 (압축 효과)                    | 매우 김 (평균 4배 이상)                      |
| **정보 밀도**      | 토큰 당 정보 밀도 높음                         | 바이트 당 정보 밀도 낮음                     |
| **OOV 처리**       | `UNK` 토큰 또는 서브워드로 분해                | 원천적으로 발생하지 않음                     |
| **계산 비용**      | 토큰화 오버헤드, 트랜스포머 계산량 상대적 적음 | 토큰화 불필요, 트랜스포머 계산량 매우 큼     |
| **주요 장점**      | 계산 효율성, 높은 정보 밀도                    | 언어 독립성, 노이즈 강건성, OOV 없음         |
| **주요 단점**      | 훈련 데이터 편향, 다국어 불균형, OOV 가능성    | 긴 시퀀스로 인한 높은 계산 비용 및 느린 속도 |

## 7. VII. 결론: BPE 토크나이저에 대한 종합적 평가

### 7.1  BPE의 유산: 현대 NLP 모델의 초석

바이트 페어 인코딩(BPE)은 데이터 압축이라는 공학적 영역에서 탄생하여, 현대 자연어 처리, 특히 대규모 언어 모델의 패러다임을 형성한 핵심 기술로 자리매김했다. BPE의 가장 큰 유산은 언어학적 지식이나 복잡한 규칙 없이도, 대규모 데이터의 통계적 패턴만으로 언어의 구조적 단위(서브워드)를 효과적으로 학습할 수 있음을 증명한 데 있다. 이 단순하면서도 강력한 아이디어는 미등록 단어(OOV)라는 고질적인 문제를 해결하고, 가변적인 어휘 크기를 통해 모델의 효율성과 표현력 사이의 균형을 맞출 수 있는 실용적인 방법을 제공했다. GPT, BERT와 같은 혁신적인 언어 모델들의 성공은 BPE가 마련한 견고한 토대 위에서 가능했다고 해도 과언이 아니다.

그러나 BPE의 성공은 그 본질적인 한계와 분리하여 생각할 수 없다. 순수한 빈도에만 의존하는 탐욕적 알고리즘은 의미론적으로 비직관적인 분절을 낳고, 훈련 데이터의 편향을 그대로 학습하여 다국어 처리의 불균형과 같은 공정성 문제를 야기한다. BPE는 효율적인 '압축기'일지는 몰라도, 현명한 '언어학자'는 아니었다.

### 7.2  미래 전망: 토큰화 패러다임의 변화 속 BPE의 위치

BPE가 드러낸 명백한 한계들은 토큰화 기술의 다음 단계를 모색하는 원동력이 되고 있다. 미래의 토큰화 패러다임은 두 가지 방향으로 전개될 것으로 전망된다.

**단기적으로는 '개선된 BPE'의 시대가 지속될 것이다.** GPT-4o가 20만 개에 달하는 거대한 어휘 크기를 채택한 사례에서 볼 수 있듯이, BPE의 기본 틀은 유지하되 어휘 크기를 대폭 확장하여 다국어, 다중 모달리티(코드, 이미지 등) 데이터를 더 효율적으로 처리하는 실용적인 접근법이 주류를 이룰 것이다. 여기에 AG-BPE와 같이 의미론적 정보를 결합하거나 BPE-Dropout처럼 확률론적 요소를 가미하여 BPE의 단점을 보완하려는 연구가 계속될 것이다.

**장기적으로는 '토크나이저-프리' 모델이 궁극적인 지향점이 될 가능성이 높다.** ByT5와 같은 바이트 수준 모델은 토크나이저가 야기하는 모든 복잡성, 편향성, 기술적 부채에서 자유로운 진정한 엔드-투-엔드(end-to-end) 학습의 비전을 제시한다. 현재는 긴 시퀀스로 인한 막대한 계산 비용이 상용화를 가로막는 가장 큰 장벽이지만, 트랜스포머를 대체할 새로운 효율적인 아키텍처(e.g., 상태 공간 모델)의 발전과 하드웨어의 성장이 이 문제를 해결한다면, 토큰화라는 전처리 단계 자체가 과거의 유산이 될 수도 있다.

결론적으로, BPE는 현대 NLP의 발전에 지대한 공헌을 한 기념비적인 기술이다. 비록 그 한계로 인해 끊임없이 도전받고 있지만, 앞으로도 한동안은 실용적인 선택지로 남을 것이며, 미래의 새로운 토큰화 패러다임을 평가하고 비교하는 중요한 기준점(baseline)으로서 그 역할을 계속 수행할 것이다. BPE에 대한 고찰은 단순히 하나의 알고리즘을 이해하는 것을 넘어, 기계가 언어를 어떻게 표현하고 학습해야 하는지에 대한 근본적인 질문으로 우리를 이끈다.

#### **참고 자료**

1. [pytorch] 토큰화 | 토크나이저(Tokenization) - resultofeffort - 티스토리, 8월 16, 2025에 액세스, https://resultofeffort.tistory.com/72
2. 토큰화란 무엇인가요? - IBM, 8월 16, 2025에 액세스, https://www.ibm.com/kr-ko/think/topics/tokenization
3. Python(38)- 자연어처리(NLP) 프로젝트 순서 - 많이 금 간 개발자 - 티스토리, 8월 16, 2025에 액세스, https://young0378.tistory.com/145
4. What is Tokenization? Types, Use Cases, Implementation - DataCamp, 8월 16, 2025에 액세스, https://www.datacamp.com/blog/what-is-tokenization
5. Summary of the tokenizers - Hugging Face, 8월 16, 2025에 액세스, https://huggingface.co/docs/transformers/tokenizer_summary
6. Tokenizers in Language Models - MachineLearningMastery.com, 8월 16, 2025에 액세스, https://machinelearningmastery.com/tokenizers-in-language-models/
7. BPE Tokenization Demystified: Implementation and Examples - MartinLwx's Blog, 8월 16, 2025에 액세스, https://martinlwx.github.io/en/the-bpe-tokenizer/
8. Understanding Byte-Pair Encoding - Medium, 8월 16, 2025에 액세스, https://medium.com/@hsinhungw/understanding-byte-pair-encoding-fd196ebfe93f
9. Byte-Pair Encoding: Subword-based tokenization algorithm - Medium, 8월 16, 2025에 액세스, https://medium.com/data-science/byte-pair-encoding-subword-based-tokenization-algorithm-77828a70bee0
10. 토크나이저(Tokenizer) 종류 - 아는 것의 미학, 8월 16, 2025에 액세스, https://applepy.tistory.com/162
11. 한국어 기계 독해를 위한 언어 모델의 효과적 토큰화 방법 탐구, 8월 16, 2025에 액세스, https://koreascience.kr/article/CFKO201930060625815.pdf
12. Byte Pair Encoding - ratsgo's NLPBOOK, 8월 16, 2025에 액세스, https://ratsgo.github.io/nlpbook/docs/preprocess/bpe/
13. Tokenization Is More Than Compression - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/html/2402.18376v1
14. Byte-pair encoding - Wikipedia, 8월 16, 2025에 액세스, https://en.wikipedia.org/wiki/Byte-pair_encoding
15. minibpe: 작고, 깔끔하고, 학습용으로 적합한 BPE(Byte Pair Encoding) 코드 (feat. Andrej Karpathy) - 파이토치 한국 사용자 모임, 8월 16, 2025에 액세스, https://discuss.pytorch.kr/t/minibpe-bpe-byte-pair-encoding-feat-andrej-karpathy/3527
16. Tokenization Is More Than Compression - ACL Anthology, 8월 16, 2025에 액세스, https://aclanthology.org/2024.emnlp-main.40.pdf
17. [Hands-On] BPE(Byte Pair Encoding)를 활용한 토크나이저 구현 - Medium, 8월 16, 2025에 액세스, [https://medium.com/@hugmanskj/hands-on-bpe-byte-pair-encoding-%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-%ED%86%A0%ED%81%AC%EB%82%98%EC%9D%B4%EC%A0%80-%EA%B5%AC%ED%98%84-6bfef6f80f3b](https://medium.com/@hugmanskj/hands-on-bpe-byte-pair-encoding-를-활용한-토크나이저-구현-6bfef6f80f3b)
18. Byte pair encoding 설명 (BPE tokenizer, BPE 설명, BPE 예시) - 유니 ..., 8월 16, 2025에 액세스, https://process-mining.tistory.com/189
19. [Hands-On] Unigram을 이용한 토크나이저 구현. 파이썬과 허깅 ..., 8월 16, 2025에 액세스, [https://medium.com/@hugmanskj/hands-on-unigram%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-%ED%86%A0%ED%81%AC%EB%82%98%EC%9D%B4%EC%A0%80-%EA%B5%AC%ED%98%84-b65b48025821](https://medium.com/@hugmanskj/hands-on-unigram을-이용한-토크나이저-구현-b65b48025821)
20. HuggingFace 내 토크나이저 종류 살펴보기 - Programador - Huffon Blog, 8월 16, 2025에 액세스, https://huffon.github.io/2020/07/05/tokenizers/
21. Explain bpe (Byte Pair Encoding) with examples? - Stack Overflow, 8월 16, 2025에 액세스, https://stackoverflow.com/questions/50583254/explain-bpe-byte-pair-encoding-with-examples
22. Byte-Pair Encoding (BPE) in NLP - GeeksforGeeks, 8월 16, 2025에 액세스, https://www.geeksforgeeks.org/nlp/byte-pair-encoding-bpe-in-nlp/
23. 의사코드(Pseudo-code) 작성법, 8월 16, 2025에 액세스, https://42kchoi.tistory.com/114
24. 슈도코드(의사코드, pseudocode) 가이드라인 - 공부하는 스누피, 8월 16, 2025에 액세스, https://snoop-study.tistory.com/58
25. 의사코드 - 나무위키, 8월 16, 2025에 액세스, [https://namu.wiki/w/%EC%9D%98%EC%82%AC%EC%BD%94%EB%93%9C](https://namu.wiki/w/의사코드)
26. 1. 알고리즘 분석과 의사코드, Big-O 표시법 - 컴퓨터와 수학, 몽상 조금, 8월 16, 2025에 액세스, https://skyil.tistory.com/21
27. A Formal Perspective on Byte-Pair Encoding - ACL Anthology, 8월 16, 2025에 액세스, https://aclanthology.org/2023.findings-acl.38/
28. A Formal Perspective on Byte-Pair Encoding - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/html/2306.16837v2
29. Formalizing BPE Tokenization - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/pdf/2309.08715
30. Parity-Aware Byte-Pair Encoding: Improving Cross-lingual Fairness in Tokenization - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/html/2508.04796v1
31. [2306.16837] A Formal Perspective on Byte-Pair Encoding - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/abs/2306.16837
32. Implementing A Byte Pair Encoding (BPE) Tokenizer From Scratch, 8월 16, 2025에 액세스, https://sebastianraschka.com/blog/2025/bpe-from-scratch.html
33. Difficulty in understanding the tokenizer used in Roberta model - Stack Overflow, 8월 16, 2025에 액세스, https://stackoverflow.com/questions/61134275/difficulty-in-understanding-the-tokenizer-used-in-roberta-model
34. ByT5 - Hugging Face, 8월 16, 2025에 액세스, https://huggingface.co/docs/transformers/model_doc/byt5
35. [Huggingface] PreTrainedTokenizer class - 티스토리, 8월 16, 2025에 액세스, https://misconstructed.tistory.com/80
36. BPE(Byte Pair Encoding) - velog, 8월 16, 2025에 액세스, https://velog.io/@gwkoo/BPEByte-Pair-Encoding
37. Byte Pair Encoding: Cracking the Subword Code, 8월 16, 2025에 액세스, https://www.atharvyeoelekar.blog/post/byte-pair-encoding-cracking-the-subword-code
38. AG-BPE: Attention-Guided Byte-Pair Encoding for Semantic-Aware Tokenization, 8월 16, 2025에 액세스, https://huggingface.co/blog/RDTvlokip/ag-bpe-attention-guided-byte-pair-encoding
39. Tokens are chunks of characters. For example, the word “alphabet” gets broken - Hacker News, 8월 16, 2025에 액세스, https://news.ycombinator.com/item?id=31402640
40. Daily Papers - Hugging Face, 8월 16, 2025에 액세스, [https://huggingface.co/papers?q=subword%20BPE%20tokens](https://huggingface.co/papers?q=subword+BPE+tokens)
41. Language Model Tokenizers Introduce Unfairness Between Languages - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/pdf/2305.15425
42. WordPiece tokenization - Hugging Face LLM Course, 8월 16, 2025에 액세스, https://huggingface.co/learn/llm-course/chapter6/6
43. [NLP] Tokenizer - velog, 8월 16, 2025에 액세스, https://velog.io/@khs0415p/Tokenizer
44. 토크나이저 요약 - Hugging Face, 8월 16, 2025에 액세스, https://huggingface.co/docs/transformers/main/ko/tokenizer_summary
45. BPE vs WordPiece Tokenization - when to use / which? - Data Science Stack Exchange, 8월 16, 2025에 액세스, https://datascience.stackexchange.com/questions/75304/bpe-vs-wordpiece-tokenization-when-to-use-which
46. WordPiece Tokenization: A BPE Variant | by Atharv Yeolekar | Medium, 8월 16, 2025에 액세스, https://medium.com/@atharv6f_47401/wordpiece-tokenization-a-bpe-variant-73cc48865cbf
47. Tokenization for language modeling: Byte Pair Encoding vs Unigram Language Modeling, 8월 16, 2025에 액세스, https://ndingwall.github.io/blog/tokenization
48. [NLP-2] 텍스트 전처리하기 - 토크나이저 - 의미있는블로그, 8월 16, 2025에 액세스, https://all-the-meaning.tistory.com/44
49. T-FREE: Tokenizer-Free Generative LLMs via Sparse Representations for Memory-Efficient Embeddings - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/html/2406.19223v1
50. Byte Pair Encoding vs. Unigram Tokenization: A Deep Dive into Subword Models - Medium, 8월 16, 2025에 액세스, https://medium.com/@hexiangnan/byte-pair-encoding-vs-unigram-tokenization-a-deep-dive-into-subword-models-4963246e9a34
51. google/sentencepiece: Unsupervised text tokenizer for Neural Network-based text generation. - GitHub, 8월 16, 2025에 액세스, https://github.com/google/sentencepiece
52. Supporting SentencePiece tokenization algorithms other than BPE · ggml-org llama.cpp · Discussion #7732 - GitHub, 8월 16, 2025에 액세스, https://github.com/ggerganov/llama.cpp/discussions/7732
53. [1908.10322] Bridging the Gap for Tokenizer-Free Language Models - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/abs/1908.10322
54. [2105.13626] ByT5: Towards a token-free future with pre-trained byte-to-byte models - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/abs/2105.13626
55. ByT5: Towards a Token-Free Future with Pre-trained Byte-to-Byte Models - MIT Press Direct, 8월 16, 2025에 액세스, https://direct.mit.edu/tacl/article/doi/10.1162/tacl_a_00461/110049/ByT5-Towards-a-Token-Free-Future-with-Pre-trained
56. MrT5: Dynamic Token Merging for Efficient Byte-level Language Models - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/html/2410.20771v3
57. ByT5: Towards a Token-Free Future with Pre ... - ACL Anthology, 8월 16, 2025에 액세스, https://aclanthology.org/2022.tacl-1.17.pdf
58. A Comparative Analysis of Byte-Level and Token-Level Transformer Models in Natural Language Processing - Greg Robison, 8월 16, 2025에 액세스, https://gregrobison.medium.com/a-comparative-analysis-of-byte-level-and-token-level-transformer-models-in-natural-language-9fb4331b6acc
59. What's the new tokenization algorithm for gpt-4o? - API - OpenAI ..., 8월 16, 2025에 액세스, https://community.openai.com/t/whats-the-new-tokenization-algorithm-for-gpt-4o/746708
60. Understanding GPT tokenizers - Hacker News, 8월 16, 2025에 액세스, https://news.ycombinator.com/item?id=36248633

