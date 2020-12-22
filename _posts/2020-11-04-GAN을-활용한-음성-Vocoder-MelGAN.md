---
title: GAN(적대적 생성 모델)을 활용한 Vocoder - MelGAN
author: HW
date: 2020-11-04 17:00:00 +0800
categories: [Speech Synthesis]
tags: [MelGAN]
math: true
image: /assets/img/insert/melgan/melgan_performance.png

---



# **들어가면서**

이전까지는 음성합성을 위해 GAN(Generative Adversarial Network)을 활용한 모델이 없었다.<br/>

그러나 2019년 10월 이 논문이 발표되면서 최근 관련된 연구가 시작되고 있다.<br/>

최근에 국내 게임 기업인 NC소프트 음성 개발 연구 팀에서 VocGAN을 발표하였는데, 이 것에 대한 내용 정리도 다음에 다루고자 한다.<br/>

<br/>

GAN을 활용함으로써 얻는 이득은 단순함과 직관성이다. 자세한 내용은 뒤에서 다루고자 한다.



# 요약

이 모델의 경우에는, Generator 모델이 Conv1d가 여러 층으로 이루어져 있다. 

그래서 구글에서 개발한 Wavenet, Nvidia에서 개발한 Waveglow보다 단순하고 직관적이다.





![exTTS](/assets/img/insert/melgan/generator.png)

위 그림에서 볼 수 있듯이 Conv와 Residual로 되어 있는데, Residual은 Stack으로 이루어져 있으며, 그 스택은 오른쪽 그림과 같다. 

![exTTS](/assets/img/insert/melgan/residual_stack.png)

Stack에서도 Conv1d가 있으며 이런 connection을 3개를 거쳐서 Output을 낸다.







### 차후 디테일 내용을 채워나갈 예정입니다.

