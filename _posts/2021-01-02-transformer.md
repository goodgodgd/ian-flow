---
layout: post
title:  "Transformer"
date:   2021-01-02 09:00:03
categories: research
---



## Introduction

본 포스트는 철저히 내가 공부한 내용을 쉽게 리마인드 하기위해 작성하였다. 앞뒤 내용 빼고 몸통만 설명하므로 자세한 설명이 필요한 분들은 다른 곳을 찾아보시길 바란다.  



## 1. Attention Is All You Need

<table>
<colgroup>
<col width="10%" />
<col width="90%" />
</colgroup>
<thead>
<tr class="header">
<th>제목</th>
<th>Attention Is All You Need</th>
</tr>
</thead>
<tbody>
<tr>
<td markdown="span">저자</td>
<td markdown="span">Ashish Vaswani외 6명 (모두 Google Brain)</td>
</tr>
<tr>
<td markdown="span">출판</td>
<td markdown="span">NIPS, 2017</td>
</tr>
</tbody>
</table>
> 주요 참고 자료: 허민석 <https://www.youtube.com/watch?v=mxGCEWOxfe8&list=PLVNY1HnUlO26qqZznHVWAqjS1fWw0zqnT>



### 1.1. Model Architecture

<img src="../assets/transformer/transformer-overall-architecture.png" alt="transformer-overall-architecture" width="500"/>

<img src="../assets/transformer/transformer-self-attention.png" alt="transformer-self-attention" width="300"/>  <img src="../assets/transformer/transformer-multi-head.jpg" alt="transformer-multi-head" width="300"/>

예시를 들 때 편의상 번역기를 가정하고 예시를 만든다. Transformer가 번역기에서 좋은 결과가 나왔을 뿐만 아니라 가장 전형적인 sequence-to-sequence 모델이기 때문이다. 여기서는 "GNU is Not Unix"를 "그누는 유닉스가 아니다"로 번역하는 경우를 생각해본다.



#### A. Encoder

##### A.1 Inputs

'GNU', 'is', 'Not', 'Unix' 각각을 word2vec 같은 워드 임베딩을 통해 벡터로 만든다. 전체 입력의 길이를 $$L_{in}(=4)$$이라 한다. 입력을 하나씩 순서대로 임베딩 하는게 아니라 한번에 모두 임베딩을 계산해야 한다. 임베딩의 결과는 $$M (L_{in},d_{model})$$ 이다.  

Input Embedding에는 순서를 알려주기 위해 다음과 같은 Positional Encoding을 더해준다.  

![transformer-position-encoding](../assets/transformer/transformer-position-encoding.png)



##### A.2. Linear

인코더에는 모든 입력이 한번에 들어간다. $$(L_{in},d_{model})$$ 모양의 입력이 Query, Key, Value 세 갈래로 들어간다. 각각은 linear 레이어를 통과하여 서로 다른 특성을 학습하게 한다. Multi-Head Attention에서는 여러개의 attention을 위한 다른 입력을 만들어야 하기 때문에 $$h(=8)$$개의 Head가 있다면 사실상 linear 레이어의 출력 차원이 $$h$$배가 되는 셈이다.

![transformer-multi-head-eq1](../assets/transformer/transformer-multi-head-eq1.png)

![transformer-weight-dimensions](../assets/transformer/transformer-weight-dimensions.png)

Query, Key, Value 모두 처음에는 $$(L_{in},d_{model})$$ 이었는데 각 head의 linear 레이어를 통과하면 Query, Key는 $$(L_{in},d_k)$$가 되고 Value는 $$(L_{in},d_v)$$ 가 된다. Value의 dimension을 다르게 할 수도 있지만 논문에서는 $$d_k=d_v=d_{model}/h=64$$ 로 모두 같게 한다.  

실제 구현을 한다면 $$h$$개의 head를 위해 따로 linear 레이어를 구성하기 보다는 출력 차원이 $$h * d_k$$인 하나의 linear 레이어로 최적화 할 것 같다.  

##### A.3. Self-attention

**Self-attention 내부에는 학습 파라미터가 없다.** "Scaled Dot-Product Attention"그림에 나온대로 단순히 아래 식을 계산할 뿐이다. $$\sqrt{d_k}$$ 로 나눠주는 이유는 학습 효율을 개선하기 위함이다. Dimension이 늘어날수록 전반적으로 dot product($$QK^T$$)의 크기 편차가 커지고 그에 따라 softmax결과로 0에 가까운 값이 많이 나와서 gradient도 0에 가깝게 나온다. Dimension에 비례하여 dot product의 스케일을 줄이면 이러한 현상을 완화할 수 있다고 한다.

![transformer-attention-eq](../assets/transformer/transformer-attention-eq.png)

$$QK^TV$$의 차원은 $$(L_{in},d_k) \times (d_k,L_{in}) \times (L_{in},d_v) = (L_{in},d_v)$$ 이다. Self-attention 식의 $$Q,K,V$$는 사실 Multi-head attention 식의 $$QW^Q, KW^K, VW^V$$ 이다. 논문에서는 self-attention 식을 먼저 설명해서 편의상 새로운 기호를 쓰지 않고 $$Q,K,V$$를 쓴 것 같다.  

**Self-attention의 의미**는 무엇일까? 첫 번째 dot product $$QK^T$$ 에서 $$(L_{in},L_{in})$$의 행렬이 나오는데 무슨 의미일까? $$QK^T$$의 (i,k) 원소의 의미는 i번째 query와 k번째 key 사의의 연관성이라고 볼 수 있다. "GNU is Not Unix" 예시에서 (0, 2) 원소는 'GNU'와 'Not' 사이의 연관성이다. 이 연관성을 합이 1이 되도록 normalize 하기위해 softmax를 한다.  

단어 사이의 연관성을 계산한 후 Value를 곱하면 $$QK^TV$$의 i번째 행(row)는 i번째 단어에 대한 새로운 임베딩이 된다. 이것은 i번째 단어의 정보 뿐만 아니라, i번째 단어와의 연관성에 따라 모든 단어의 linear combination이기 때문에 전체 문맥에서 가지는 i번째 단어의 의미 같은 것을 알 수 있게 된다. 이점이 RNN과의 가장 큰 차이점이다. 순서대로 입력하지 않고 처음부터 전체를 다 보기 때문에 sequence가 길어져도 멀리 떨어진 단어들 사이의 관계를 파악할 수 있다. 언어 사이의 어순이 달라도 이를 처리할 수 있다.



##### A.4. Multi-head attention

Multi-head attention에서는 self-attention을 여러개 사용하는데 이건 마치 CNN의 convolution에서 채널 마다 다른 특징을 추출하는 것과 비슷한 것 같다. Self-attention도 하나만 있으면 정확한 문맥 파악을 잘 못 할수 있지만 여러가지 self-attention을 쓰면 다양한 문맥적 의미를 탐색하여 최종적으로는 더욱 정확한 의미를 알 수 있다.  

Multi-head attention에서는 $$d_v$$ 차원의 출력이 $$h$$개 나온다. 최종 출력은 입력과 같은 $$d_{model}$$ 이 되어야 하기 때문에 Multi-head의 결과물들을 concat하여 $$h * d_v$$ 차원의 행렬을 만들고 여기에 $$W^O (h*d_v,d_{model})$$을 곱하여 $$d_{model}$$ 차원의 벡터를 출력한다.  

$$(L_{in}, h*d_v) \times (h*d_v, d_{model}) = (L_{in}, d_{model})$$



##### A.5. Feed forward

Multi-head attention 이후 Add&Norm 레이어가 있다. ResNet 처럼 입력을 출력에 더해주는데 이것은 벡터(단어의 임베딩)의 순서 정보를 잊지 않기 위함이다. [Layer Normalization](https://arxiv.org/abs/1607.06450) 까지 적용후 feed forward net(FFN)을 통과하는데 이것은 단순히 Linear-ReLU-Linear 조합이다. 

![transformer-ffn](../assets/transformer/transformer-ffn.png)

FFN은 벡터(단어)별로 따로 적용된다. 같은 레이어에서 벡터 별 적용되는 $$W_1, W_2$$는 같지만 레이어 별로 다른 weight를 가진다. 이후 한 번 더 Add&Norm 레이어를 통해 입력과 출력을 더하고 정규화한다.  



##### A.6. Stack

인코더는 self-attention과 FFN이 결합된 하나의 인코더 레이어를 6번 쌓아 만든다. 인코더의 최종 출력만 디코더에 입력으로 들어간다. 인코더는 auto-regressive 하지 않기 때문에 출력이 입력으로 다시 들어가지 않고 병렬로 모든 입력 데이터를 처리한다.



#### C. Decoder

디코더도 인코더와 비슷한데 세 가지 차이가 있다.

##### C.1. Input

인코더와 달리 디코더는 auto-regressive 하다. 이전의 출력이 현재의 입력으로 들어간다. 디코더의 첫 입력은 SoS (Start of Sentence)이고 이후 출력이 하나씩 붙어서 점점 입력 단어 수가 늘어난다. 계속 출력이 입력에 추가되다가 EoS (End of Sentence) 가 출력되면 중단된다.  



##### C.2. First multi-head attention

디코더에서는 multi-head attention이 두 개 들어간다. 첫 번째는 **Masked** mutli-head attention 인데 'mask'는 "Scaled Dot-Product Attention" 그림에서 "Mask (opt.)"라고 된 블럭을 말한다. 디코더의 첫 번째 multi-head attention에서만 이 masking이 적용된다.  

논문에서는 auto-regressive한 성질을 유지하기 위해, 즉 미래의 입력을 사용하는 "leftward information"을 막기 위해, softmax 전에 미래 입력에 대한 값들을 $$-\infty$$로 바꿔서 softmax에서 0이 나오도록 했다.  

여기서 의문이었던 점은 원래 입력에 과거부터 현재까지의 단어만 들어오는데 masking 해야 할 미래의 입력이 무엇인지 이해할 수 없었다. 하지만 코드를 보니 학습할 때는 target sentence (GT output)을 디코더에 바로 넣어서 한번에 문장이 나오도록 했다. 즉 "\<SoS\> 그누는 유닉스가 아니다"를 바로 디코더에 입력해서 "그누는 리눅스가 아니다 \<EoS\>"가 나오도록 학습하는 것 같다. 그래서 각 단어별로 그 다음 단어에 대한 입력을 masking 하게 한 것이다. 예를 들면 '그누'라는 단어를 처리 할 때는 '유닉스가', '아니다'를 masking 하는 것이다.



##### C.3. Second multi-head attention

두 번째 multi-head attention은 입력 구성이 다르다. 기존에는 하나의 입력이 세 갈래로 갈라지면서 Query, Key, Value가 됐다. 여기서는 인코더의 최종 출력이 Value, Key로 들어오고 첫 번째 attention의 출력은 Query로만 들어간다.  

다음 레이어로 넘어가는 Value(단어 임베딩?)은 인코더에서 가져오고 인코더의 Key와 디코더의 Query를 dot product 한다. 인코더는 입력 언어(영어)에 대한 정보를 담고 있고 디코더는 새로운 언어(한글)에 대한 정보를 담고 있다. 디코더의 Query를 이용해 인코더의 Value 중에서 이번에 출력될 다음 단어(한글)와 관련된 입력 언어(영어)의 단어 정보를 조합하는 것이다.



#### D. Final Output

논문에서도 유툽, 블로그 등에서도 간단히 최종 출력은 Linear - Softmax로 다음 단어가 하나 나오고 끝나는 듯 말하지만 세부적인 내용을 숨기고 있다.  

인코더에서는 입력과 출력의 dimension이 똑같다고 했다. 즉 입력 단어가 5개면 출력되는 임베딩도 5개다. 디코더도 거의 비슷한 구조인데 auto-regressive하게 출력을 내면 출력되는 임베딩의 개수가 늘어날 수 밖에 없다. 처음엔 1개, 다음엔 2개 이런식으로 디코더 자체의 출력 임베딩 수가 늘어나게 된다. 출력 데이터의 차원이 일정하지 않은데 어떻게 Linear 레이어를 적용할까?  

[번역하는 코드](https://github.com/jadore801120/attention-is-all-you-need-pytorch/blob/132907dd272e2cc92e3c10e6c4e783a87ff8893d/transformer/Translator.py#L86)를 보니 for문을 돌면서 auto-regressive 하게 하는 것 같지만 `_get_the_best_score_and_idx` 이 함수가 끝내 잘 이해되진 않았다. 대략 추측하기로는 마지막 Linear - Softmax 레이어도 출력 임베딩 별로 따로 적용하고 그 결과로 나온 단어 별 벡터들을 스텝별로 쌓아두고 그 중에서 가장 좋은 (score가 높은?) 벡터를 선택하는 것 같다. 즉 단어를 순서대로 입력하면 '그누는'이라는 단어가 디코더에 세 번 들어가는데 그 결과로 나온 세 개의 벡터 중에 가장 좋은 것을 선택하는 것 같다.  



### 1.2. Loss

딥러닝 논문이라면 자로고 Loss를 잘 설명해야 하거늘 이 논문에는 loss라는 단어가 아예 없다!  

결과가 softmax로 나오므로 당연히 cross entropy loss를 쓸거라고 생각은 하지만... Loss와 관련된 내용은 label smoothing을 쓴다는 것 뿐이다.  



