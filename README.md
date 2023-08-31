# About Parameters in Text Generative AI

<br>

### 자연어 처리(Natural Language Processing) 모델의 디코딩 방식
- Greedy Search
- Beam Search
- Sampling
  - 확률 분포를 기반으로 다음 토큰을 결정하는 과정
  - temperature를 이용하여 확률 분포를 조정
- Top-K Sampling
  - 확률 값이 높은 순서대로 토큰을 나열한 후 순서대로 k개만큼 후보에 추가
  - 위 k개의 후보 중에서 다음 토큰을 결정
  - 이때 사용하는 k값이 top_k
- Top-P Sampling
  - 확률 값이 높은 순서대로 토큰을 나열한 후 누적 확률이 p에 최초로 도달할 때까지 순서대로 후보에 추가
  - 위 후보들 중에서 다음 토큰을 결정
  - 이때 사용하는 p값이 top_p
- top_p, top_k, temperature의 동시 사용
  - 일반적인 우선 순위는 temperature → top_p → top_k
  - 모델 별로 우선 순위가 다를 수 있음
    - Ex) Aleph Alpha에서는 temperature → top_k → top_p 순으로 적용
  - 모델에 따라 3가지를 모두 동시에 사용하는 것이 권장되지 않을 수 있음
    - EX)
      - Aleph Alpha(temperature, top_k 또는 top_p 중 하나를 사용하는 것이 좋지만 세 가지를 동시에 사용하지 않는 것이 좋습니다.)
      - Kakao KoGPT에서는 top_k 파라미터가 없음
      - Kakao KoGPT(temperature, top_p 두 항목의 값을 동시에 변경하는 것은 권장하지 않습니다. 동시에 변경하면 결과에 영향을 주지 않는 항목이 발생해 사용자의 의도에 맞는 KoGPT의 답변을 유도하기 어려울 가능성이 있습니다.)

<br>

### Temperature
  - 일반적으로 0 < temperature ≤ 1.0
  - temperature ↓ → 다양성 ↓
  - 원리
    - 아래 수식은 단순 원리 참고용 수식
    - P: 각 토큰이 선택될 확률, a: 각 토큰의 벡터값, T: temperature
    - temperature 적용 전
      <p align="leading">
        <img width="15%" src="https://github.com/cyeond/AI-Study-Text/assets/139483587/26a5825e-25ca-4c62-a4ce-4e258ae066fe">
      </p>
    - temperature 적용 후
      <p align="leading">
        <img width="15%" src="https://github.com/cyeond/AI-Study-Text/assets/139483587/ceaa7c1a-2226-464c-9e9c-5024b3aa697d">
      </p>
  - Ex)
    - a = [0.25, 0.5, 1]
      - T = 1.0
        - exp(a/T) = [1.28, 1.65, 2.72]
        - P = [0.23, 0.29, 0.48]
      - T = 0.5
        - exp(a/T) = [1.65, 2.72, 7.39]
        - P = [0.14, 0.23, 0.63]
      - T = 0.25
        - exp(a/T) = [2.72, 7.39, 54.60]
        - P = [0.04, 0.11, 0.84]
       
<br>

### top_k
- 자연수 값을 사용
- top_k ↑ → 다양성 ↑
- Ex) <p align="leading">
    <img width="40%" src="https://github.com/cyeond/AI-Study-Text/assets/139483587/e08697cd-4a77-490c-9ab9-644c94948106">
  </p>
  
  - “너” 다음에 올 토큰을 선택하는 과정
  - top_k = 2
    - “너가”, “너는“ 중 하나로 결정됨
  - top_k = 5
    - “너가“, “너는“, “너에게“, “너를“, “너을“ 중 하나로 결정됨
    - 다양성이 증가했지만 “너을“과 같이 적절하지 않은 후보가 선택될 수 있음
   
<br>

### top_p
- 0 ≤ top_p ≤ 1.0
- top_p ↑ → 다양성 ↑
- Ex)
  - top_k와 동일한 예시
  - top_p = 0.7
    - “너가”, “너는“ 중 하나로 결정됨
  - top_p = 1.0
    - “너가“, “너는“, “너에게“, “너를“, “너을“ 중 하나로 결정됨
    - 다양성이 증가했지만 “너을“과 같이 적절하지 않은 후보가 선택될 수 있음
   
<br>

### Penalties
- logits[i] -> logits[i] - c[i] * frequency_penalty - float(c[i] > 0) * presence_penalty
  - logits[i]: i번째 토큰의 로짓
    - 로짓
      - 확률로 변환되기 전의 최종 결과값
      - Softmax 함수를 거쳐 확률로 변환
    - c[i]: i번째 이전까지 해당 토큰이 몇 번이나 나왔었는지(횟수)
    - float(c[i] > 0): i번째 이전까지 해당 토큰이 한 번이라도 나왔었는지(1 또는 0)
- presence_penalty
  - presence_penalty ↑ → 반복 가능성 ↓
  - 이미 나왔던 토큰이 또 나올 가능성을 낮춤
  - 토큰의 등장 횟수에 영향을 받지는 않음
  - 일반적으로 양수를 사용하며 0 이상 1.0 이하 값을 주로 사용함
  - 음수를 사용하여 반복 가능성을 높일 수 있도록 하는 모델도 존재
- frequency_penalty
  - frequency_penalty ↑ → 반복 가능성 ↓
  - 이미 나왔던 토큰이 또 나올 가능성을 낮춤
  - 토큰이 이전에 여러 번 나왔을 수록 다시 반복될 가능성이 낮아짐
  - 일반적으로 양수를 사용하며 0 이상 1.0 이하 값을 주로 사용함
  - 음수를 사용하여 반복 가능성을 높일 수 있도록 하는 모델도 존재
 
<br>
<br>
