---
layout: post
title: Improved Variational Inference with Inverse Autoregressive Flow
---

***
2016년도에 발표된 Improved Variational Inference with Inverse Autoregressive Flow 논문에 대해 소개드리겠습니다.

***

![_config.yml]({{ site.baseurl }}/images/01.PNG)

***

![_config.yml]({{ site.baseurl }}/images/02.PNG)

소개 순서는 위와 같습니다. 기본적으로 논문에서 서술하고 있는 순서입니다.

***

![_config.yml]({{ site.baseurl }}/images/03.PNG)

이 논문은 새로운 타입의 NF를 제안하고 있습니다. inverse autoregressive flow의 약자로 IAF라는 NF입니다.
이 것은 고차원의 latent space로 확장이 용이하다고 합니다.
그리고 이를 VAE에 적용하여 IAF의 효과를 보여주었습니다.

아래는 논문 서두에 있는 그림입니다. prior 분포를 기존의 가우시안 포스테리어로 피팅을 한 경우, 빈공간이 많이 보이는 것처럼
완전히 피팅을 못하였는데요, IAF로 한경우에는 prior와 거의 유사하게 피팅되었습니다. 
이는 IAF가 그만큼 포스테리어의 flexibility를 향상시킨 결과라고 할 수 있습니다. 

***

![_config.yml]({{ site.baseurl }}/images/04.png)

설명1

***
