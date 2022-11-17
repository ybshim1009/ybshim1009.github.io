---
layout: post
title: Improved Variational Inference with Inverse Autoregressive Flow
---

***
blog link : https://ybshim1009.github.io/

2016년도에 발표된 Improved Variational Inference with Inverse Autoregressive Flow 논문에 대해 소개드리겠습니다..

***

본 내용은 Improved Variational Inference with Inverse Autoregressive Flow을 읽고 정리한 내용입니다. 아래의 저자들이 작성한 논문으로 2016년에 발표되었습니다.
Diederik P. Kingma (OpenAI), Tim Salimans (OpenAI), Rafal Jozefowics (OpenAI), Xi Chen (OpenAI), Ilya Sutskever (OpenAI), Max Welling (University of Amsterdam)


논문의 순서와 동일하게 아래와 같이 설명하겠습니다.
1. Introduction
2. Variational Inference and Learning
3. Inverse Autoregressive Transformation
4. Inverse Autoregressive Flow(IAF)
5. Experiments
6. Conclusion
7. Q&A



1. Introduction
본 논문의 주요 contribution은 새로운 타입의 NF(IAF: Inverse Autoregressive Flow)를 제안하였으며, 이는 고차원 latent space로 확장이 용이하다는 특징이 있습니다. 이를 입증하기 위해 VAE에 적용하였습니다.

![image](https://user-images.githubusercontent.com/117826908/202465512-ffc0a748-7ce8-499a-b79b-1fa97f33b294.png)
 
위 그림은 논문 서두에 있는 그림입니다. prior 분포를 기존의 가우시안 포스테리어로 피팅을 한 경우, 빈공간이 많이 보이는 것처럼 완전히 피팅을 못하였는데요, IAF로 한경우에는 prior와 거의 유사하게 피팅되었습니다. 이는 IAF가 그만큼 포스테리어의 flexibility를 향상시킨 결과라고 할 수 있습니다.



2. Variational Inference and Learning
다음은 variational inference에 대한 일반적인 내용을 다루고 있습니다. 생성 모델의 목표는 아래식과 같이 marginal likelihood를 최대화 하는 것입니다.
 
![image](https://user-images.githubusercontent.com/117826908/202465581-81f4187d-79e9-4509-85ea-9af6594f3bd7.png)

하지만, 이 것을 계산하는 것은 intractable하여, 아래와 같이 근사 분포인 q를 도입하여 ELBO를 최적화하는 것으로 문제를 푸는 방식도 제안되었습니다. 
 
![image](https://user-images.githubusercontent.com/117826908/202465633-367d537f-cf70-44c8-a9a6-406b9607eab7.png)

![image](https://user-images.githubusercontent.com/117826908/202465654-513e9b09-a151-4358-b594-6495a080ed31.png)

그러기 위해서는 q가 p와 유사한 분포가 될 수 있을 만큼 flexible하다면, KL Divergence는 작아질 것이고, p와 유사한 분포를 얻을 수 있습니다.

다음으로 논문에서는 효율적으로 ELBO를 최적화 하기 위한 inference 모델의 요구사항으로 아래 두가지를 언급하고 있습니다. 하나는 q를 계산하고 미분하기 쉬워야 하며 다른 하나는 그 모델에서 샘플링하기 쉬워야 한다고 하였습니다. 왜냐하면 최적화 과정에서 자주 사용되기 때문에 효율적으로 계산할 수 있어야 합니다.
1) Compute and differentiate q(z|x)
2) Sample from it

즉, q를 계산하고 미분하기 쉬워야하며, 그 모델에서 샘플링하기 쉬워야 합니다.
왜냐하면 최적화 과정에서 자주 사용되기 때문에 효율적으로 계산할 수 있어야 되기 때문입니다.

![image](https://user-images.githubusercontent.com/117826908/202465717-ff2284f5-b53c-4bee-a1cb-892f1e34e940.png)
 
출처 : https://lilianweng.github.io/posts/2018-10-13-flow-models/

다음으로는 NF에 대해서 이야기 하고 있습니다. 위 그림에서 보이듯이, 쉬운 분포로부터 역변환이 가능한 여러 연산을 수행하여 유연한 posterior를 얻게 됩니다.

![image](https://user-images.githubusercontent.com/117826908/202465757-c734f714-e9bb-47c3-a109-5a9c46e36611.png)

예를 들어, 가우시안 등 간단한 분포로부터 z0을 선택하여 여러 번의 변환 과정을 거칩니다.
그러면 마지막 iteration T에서는 원하는 분포를 얻을 수 있게 됩니다.
이 분포는 다음과 같으며, T번의 변환 과정동안 자코비안 디터미넌트를 구할 수 있다면, 
마지막이 분포도 계산할 수 있게 됩니다.
 
![image](https://user-images.githubusercontent.com/117826908/202465798-8324e80e-a0cd-4860-b0a7-52a90f5544a3.png)

중요한 점은 이 자코비안 디터미넌트를 얼마나 쉽게 구할 수 있느냐 입니다.



3. Inverse Autoregressive Transformation
이제부터 본 논문에서 제안하고 있는 inverse autoregressive 변환에 대한 이야기 입니다.
 
![image](https://user-images.githubusercontent.com/117826908/202465843-db4cb455-a832-4a22-b7c6-b60d43731e46.png)


앞에서 이야기한 대로 자코비안 디터미넌트를 구해야 합니다. 각 element는 위와 같습니다. 이는 현재 정보가 과거 정보에 의해 만들어지고 있는 autoregressive 구조이며, 이런 경우, 자코비안은 대각선이 0인 하삼각행렬이 됩니다. 

좀더 자세히 설명 드리면, yi는 뮤와 시그마로 만들어지는데, 미래의 yi로 과거의 뮤와 시그마를 미분하면 이는 yi에 관계가 없는 상수로 취급되어 0이 됩니다.
뮤i, 시그마i 도 yi 보다 과거의 값입니다. 뮤i는 y1부터 i-1까지로 만들어지는 것으로 yi는 뮤i 관점에서 미래에 있는, 즉 관계가 없는 변수입니다. 즉, 이것으로 미분을 하면, 상수이므로 0이 됩니다. 그래서 자코비안 행렬은 하삼각행렬이 됩니다.
또한, 엡실론0을 선택한 다음, 순차 변환을 통해서 yi를 구해갑니다. 즉, yi를 계산할 때 yi-1까지의 값이 필요하며 차원 D 가 커지면 계산은 복잡하게 됩니다.

이 식을 엡실론 기준으로 다시 정리하면 아래와 같이 됩니다. 

![image](https://user-images.githubusercontent.com/117826908/202466042-733aa8f4-c7e6-451a-8520-b23fe3573e6e.png)

이렇게 바꾸면 다음의 두가지 장점이 생깁니다. 1) 자코비안을 계산하기 쉬우면, 2) 병렬화가 가능합니다.

1) 먼저, 자코비안 계산입니다.
앞에서 이야기한데로, 뮤와 시그마를 y로 미분하면 대각선이 0이 되고 자코비안이 0이 됩니다.
하지만, 식을 엡실론으로 정리한 뒤에 y로 미분하게 되면, 같은 인덱스에 대해서도 1/시그마가 나오게 됩니다.
 
![image](https://user-images.githubusercontent.com/117826908/202466083-6aa6d2ea-8c61-404a-9834-7d349723b169.png)

![image](https://user-images.githubusercontent.com/117826908/202466102-b9492a12-0622-40c6-8014-ccece858e6f1.png)

![image](https://user-images.githubusercontent.com/117826908/202466126-1f1d50fe-02d7-41db-af33-975ab0857ed1.png)


 
 
디터미넌트는 대각선의 곱이므로 아래와 같이 계산할 수 있게 됩니다. 엡실론에 대한 자코비안 행렬과 디터미넌트 계산입니다.

![image](https://user-images.githubusercontent.com/117826908/202466195-5a30d5eb-4087-4de4-a20f-8baddf63a406.png)

![image](https://user-images.githubusercontent.com/117826908/202466222-e7dbb2b1-8f1d-4052-815e-ec87be8a4f26.png)


2) 다음은 병렬화입니다. 특징을 보다 잘 살펴보기 위해 Autoregressive 변환과 Inverse Autoregressive 변환을 비교해서 살펴보겠습니다
- Autoregressive 변환식은 다음과 같습니다.
z1 = fθ(x1)	z2 = fθ(x2;x1)	z3 = fθ(x3;x1,x2)
x1 = fθ-1(z1)	x2 = fθ-1(z2;x1)	x3 = fθ-1(z3;x1,x2)
 
![image](https://user-images.githubusercontent.com/117826908/202466376-e8b55b6b-3a53-4b0b-9576-ce6f26c577b8.png)

출처 : https://blog.evjang.com/2018/01/nf2.html
즉, 샘플링하는 과정에서 기존에 샘플링된 결과를 이용하기 때문에 시간이 오래걸릴 수 밖에 없었습니다. 대신 z를 만들어내는 학습과정은 병렬화가 가능합니다. 

반면, Inverse Autoregressive 변환에서는 그 반대입니다.
- Inverse Autoregressive 변환식입니다.
z1 = fθ-1(x1)	z2 = fθ-1(x2;z1)	z3 = fθ-1(x3;z1,z2)
x1 = fθ(z1)	x2 = fθ(z2;z1)	x3 = fθ(z3;z2,z1)

![image](https://user-images.githubusercontent.com/117826908/202466402-1ecdaf47-fb00-49ca-9a2e-90bf39b10419.png)

출처 : https://blog.evjang.com/2018/01/nf2.html
위 그림에서 보다시피 샘플링하는 과정에서 과거의 샘플링 값을 사용하기 않기 때문에 병렬화가 같능합니다.



4. Inverse Autoregressive Flow(IAF)
 
![image](https://user-images.githubusercontent.com/117826908/202466453-8112d522-7442-45ef-97be-31cf5c53da5a.png)

본 논문에서는 이 inverse autoregressive 변환을 이용하여 위 그림과 같이, IAF를 제안하였습니다.
앞에서 언급된 T번 변환된 q 값은 그림과 같이 초기값과 디터미넌트를 입력하여 풀어주면
아래와 같이 최종 형태를 얻게 됩니다.
 
![image](https://user-images.githubusercontent.com/117826908/202466500-1a3e2306-b54b-415e-9d27-6d142e55ba1d.png)

즉 D차원에 대해 계산을 반복하는데, IAF의 스탭 T만큼 디터미넌트를 더해주는 모양입니다.
간단한 모양으로 표현되었습니다.

처음에 입력된 x와 별도로 엡실론을 선택하여 z초기 값을 만든 후, IAF 스탭을 거치게 됩니다.
여기서 추가적으로 h 값이 만들어지고 각IAF 스텝에 입력됩니다. 
이는 LSTM에서 과거 정보를 어느 정도 유지하기 위해 cell state를 가지는 것과 유사한 이유 같습니다.
하나의 IAF 스탭은 위 그림의 오른쪽과 같이 생겼습니다.

이렇게 만들어진 z, h는 Autoregressive NN에 입력됩니다. 이는 Autoregressive특성을 같는 NN이면 어떤 것이든 사용할 수 있을 것 같습니다만, 여기서는 MADE를 사용하였다고 합니다. 
거기서 m, s를 만들고, 새로운 Zt를 만들때, 과거 z와 m에 비율을 적용하여 더하였습니다. 
이는 LSTM에서 영감을 받았다고 합니다. 그리고 forget gate bais로 알려진 바와 같이, St값이 1, 2면 값일 때 좋은 성능을 내었다고 합니다. 각 IAF 스탭에서는 이런 연산을 수행합니다.

![image](https://user-images.githubusercontent.com/117826908/202466559-61217889-6940-498d-a555-c88fb5b77f33.png)
 
위의 알고리즘은 현재까지 이야기한 내용을 수도코드로 표현한 것입니다.
중간에 보면 앞에서 보았던 Zt식이 보이며, 매 스탭마다 디터미넌트 값을 더하는 것을 확인할 수 있습니다.



5. Experiments
다음은 실험입니다.

![image](https://user-images.githubusercontent.com/117826908/202466616-f51e5cef-7b82-4c1a-9ab1-bc613a720c20.png)
 
resnet블럭을 사용하였으며, autoregressiveNN는 2레이어 짜리 MADE 1개로 구성하였다고 합니다.

- MNIST데이터의 경우,

![image](https://user-images.githubusercontent.com/117826908/202466647-b3239345-3294-4b13-be6d-4519cfb81f0d.png)

여러 포스테리어의 성능을 비교해본 결과입니다. IAF의 VLB가 depth와 width가 커질수록 커지며, 최대 -79.1까지 도달했습니다.


- CIFAR-10 의 경우입니다.

![image](https://user-images.githubusercontent.com/117826908/202466680-0198c38f-15da-4063-a52f-2a25a413ac07.png)

3.11의 bit per dim을 달성하여 다른 latent variable 모델보다 성능이 좋았으며
기존 Likelihood를 사용하는 모델의 최고 성능과 비슷하게 나왔다고 평가하고 있습니다. 이는 IAF의 스텝을 더 쌓으면 개선될 것 같다고 하였습니다.

게다가, 샘플링 속도에서는 0.05초로 pixelCNN의 52초에 비해 압도적으로 빨랐다고 합니다.



6. Conclusion
본 논문은 Flow의 change of variable과 Autoregressive를 조합하여 만든 논문입니다. 샘플링하는 과정에서 속도는 높이기 위해(병렬화가 가능하도록) 역변환하는 방법을 제안하였습니다. 이를 통해 빠른 샘플링과 준수한 성능의 생성 모델을 만들 수 있었습니다.



7. Q&A
Q1: 좀 기본적인 질문이 될 것 같은데, normalizing flow 자체적으로 이미 autoregressive하지 않나요? 
A1: 네 Zt-1로 Zt값을 만들기 때문에 autoregressive하다고 생각합니다.

Q2: dut/dzt-1 와 d시그마t/dzt-1 이 triangular with zeros on the diagonal 이라고 나오는데 왜 이렇게 되는지 궁금합니다. (4. Inverse Autoregressive Flow(IAF)) 
A2: 3장에서 설명하였듯이, 미래의 정보, 즉 의존관계가 없는 변수로 미분을 하게 되여 0이 된다고 이해하고 있습니다. 

Q3: Inverse Autoregressive Transformations의 8번 식 전에 나오는 lower triangular jacobian 부분이 무엇인지 잘 모르겠습니다. 
A3: A2와 같이 3장을 참고하시면 될 것 같습니다.

Q4: EncoderNN에서는 mu와 sigma가 epsilon과 independent하게 추출되지만 IAF Step의 AutoregressiveNN에서는 mu와 sigma가 Zt에 dependent하게 추출되는 것 같은데, log-determinant를 단순히 –sum(log(sigma))로 구할 수 있는 것인가요? 
A4: 네. Change of variable과 역변환을 이용해, epsilon 기준으로 식을 정리하면, 자코비안 디터미넌트를 이야기하신 식과 같이 간단히 정리할 수 있게 됩니다.

Q5: determinant를 단순하게 구할 수 있기에 inverse autoregressive flow를 사용하나요? 
A5: Det를 단순화 시킬수 있는 방법중 하나가 IAF라고 생각됩니다.

Q6: 방정식 9. 와 Figure 2를 보면 이 모델은 x간의 autoregressive구조가 아닌 z간의 autoregressive구조를 가정하는 것 같은데 섹션 3.에서 설명한 방정식 7. 의 경우 처럼 엡실론을 기준으로 뒤집으면 역연산이 병렬적으로 이루어 질 수 있다는 점 때문에 앱실론을 활용하기 위해서라고 봐도 되나요? 
A6: 네. 그렇다고 이해하고 있습니다.

Q7: 엡실론을 기준으로 inverse한다고 알고 있는데 이는 기존의 normalizing flow를 inverse한 것으로 이해하면 될까요? 
A7: 네 맞습니다.

Q8: z에서 대해서 autoregressive property가 생겨 parallel한 계산이 가능해 샘플링을 훨씬 빠르게 할 수 있다는 것으로 이해했는데, 이 parallel한 계산이 가능하다는 부분이 잘 이해가지 않습니다. 자세한 설명이 가능할까요?  
A8: 3장의 설명을 보시면, 입력되는 값이 앞에서 만들어지는 값에 의존하고 있지 않음을 알 수 있습니다. 즉, 병렬적으로 처리할 수 있다는 의미입니다.

Q9: IAF는 x간의 autoregressive가 아닌 z간의 autoregressive 구조인게 맞나요? 
A9: 네 맞습니다.

Q10: 병렬연산이 가능하다고 하는데, Autoregressive가 들어가면 병렬계산이 어렵지 않나요? 또 LSTM과 동일하게forget gate를 구현한 것 또한 병렬계산 이 불가능하게 하는 요소라고 생각했습니다 
A10: 기본적으로 autoregressive는 병렬화가 어렵지만, 이를 역으로 변환하여 샘플링 과정이 병렬화가 가능하도록 알고리즘을 제안한 것이 본 논문의 contribution중 하나라고 생각합니다.

Q11: (IAF)를 사용할 때 다른 모델보다 빠른 샘플링 속도를 가진다고 하는데 이러한 빠른 샘플링 속도를 가지게 되는 이유가뭘까요? 
A11: 병렬화가 가능하기 때문입니다.

Q12: Flow 모델에 대해서 설명을 들었을때, VAE에 빠른 sampling과 Autoregressive Model의 Tractable함을 장점으로 가져서 좋다고 배웠었는데, 해당 모델은 VAE에서 Flow개념을 적용하여 성능을 높입니다. 그렇다면 다시 intractable해지게되는데 그럼에도 불구하고 해당 생성 모델이 다른 모델에 비해 어떤 Contribution이 두각이 될 만한 것인지 정리해주시면 감사하겠습니다. 
A12: Det를 구할수 있어서 tractable 하며, 샘플링을 병렬로 진행할 수 있는 것이 장점이라고 생각합니다. 

Q13: inverse transformation을 통한 샘플링 과정에서 엡실론은 서로 독립적이므로 병렬처리가 가능하다고 생각하는데, y를 연산하는 과정에서는 Y1:i-1 까지의 데이터를 이용하여 순차적으로 계산을 하는 것으로 보입니다. 이 경우 y는 연산이 병렬적으로 처리하기 어렵다고 생각하는데, 이에 따른 학습시 간의 증가는 영향이 없는지 궁금합니다. 
A13: 네 맞습니다. 학습시간은 시퀀셜하게 진행되어 느립니다.

Q14: Encoder NN에서 나오는 h가 매 flow마다 additional input으로 제공하는 이유가 무엇인가요? Autoregressive Model의 경우, conditional pixelCNN과 같은 형태로 입력을 받는다고 보면 될까요? 
A14: 기본적으로 LSTM에서 영감을 받았다고 합니다. 즉, LSTM의 hidden과 유사한 기능을 하도록 구현한 것이 아닌가 합니다.

Q15: IAF에서는 h로 표기되는 additional input이 있는데 이 h의 역할이 무엇인가요? 
A15: A14와 같이 LSTM의 hidden과 유사한 기능을 하도록 구현한 것이 아닌가 합니다.

Q16: Fig 2.에서 h가 어떻게 나오는 것인지 잘 이해하지 못했습니다. 모델의 흐름에 대해 좀 더 자세히 설명해주실 수 있을까요? 
A16: LSTM의 long term dependency를 해결하기 위한 Cell state와 비슷하게 과거 정보(x)를 전달하는 역할일 듯 합니다.

Q17: What does it refer with 'We found that results improved when reversing the ordering of the variables after each step in the IAF chain". How are they reversed?  
Q18: volume-preserving transformation을 했을 때 더 결과가 좋게 나왔다고 하는데 해당 transformation 과정이 잘 이해가 되지 않아서 어떻게 하는 것인지 추가 설명해주시면 감사하겠습니다. 
A17-18: 이는 자코비안 연산과 관련있는 것으로 이해하고 있습니다. 변환이 일어날 때마다 특정 부분은 확장, 다른 부분은 축소가 일어나는데, 이를 적절히 유지하는 것이 학습에 도움이 된다고 이해 했습니다. 이와 관련해서 아래의 논문에서 언급하고 있는 내용을 참고하면 좋을 거 같습니다.
 
![image](https://user-images.githubusercontent.com/117826908/202466745-3c8eb770-3658-4438-ac1b-3c3573c74d7c.png)

Q19: What is the motivation behind having two outputs (m and s) from the autoregressive network in equation 12?
A19: 기본적으로 가우시안에 평균과 분산을 조정하여 분포를 변환시키는 과정을 만들기 때문에, 평균과 분산에 해당하는 출력을 만든다고 생각합니다.

Q20: z를 생성하는데 autoregressive 구조를 사용했는데 그렇다면 저희가 여태까지 배운 autoregressive 방법들을 적용하는 다른 flow 모델도 있을까요?
A20: MADE를 이용한 MAF(Masked Autoregressive Flow)가 있었으며 다른 것도 있을 듯합니다.

Q21: 해당 모델은 일반적인 normalizing flow(앞선 두 논문(vi with normalizing flow, NIce)들에 비해 어떠한 부분이 강점이라고 생각하시는지 여쭙고 싶습니다. 
A21: 샘플링 속도가 빠르며, 전체 변수를 사용하여 변환의 체인 길이가 짧아질 수 있을 것 같습니다. 

Q22: realNVP가 IAF보다 훨씬 복잡한 분포를 표현할 수 있을 것 같은데 IAF가 realNVP보다 좋은 성능이 나올 수 있었던 이유가 무엇이라고 생각하시나요? 
A22: realNVP는 변수의 일부만 사용해서 그렇지 않을까 합니다.

감사합니다.





