---
title: NC소프트에서 개발한 GAN기반 음성 Vocoder VocGAN
author: HW
date: 2020-11-10 17:00:00 +0800
categories: [Speech Synthesis]
tags: [VocGAN]
math: true
image: /assets/img/insert/vocgan/vocgan_model_architecture.png

---



# **들어가면서**

저번주에 GAN 기반의 음성합성 Vocoder인 MelGAN을 소개해 보았다. 

그런데 최근에 NC소프트에서  GAN을 활용한 음성합성 Vocoder을 발표하여 9월 22일에 국제학회에 게재 승인되었다.<br/>

내용을 찾기 어려운 음성 합성 시장에서 이런 소식은 나에게 굉장한 희소식이었기에 바로 찾아 보았다.

GAN을 활용함으로써 얻는 이득은 MelGAN에서 말했듯이 단순함과 직관성이다. 이를 어떻게 NC소프트에서 활용했는지 뒤에서 확인해 보도록 하겠다.

NC소프트는 모바일 RPG 게임을 많이 출시해 운영하는 만큼 캐릭터 성우에 대한 필요성이 높으며, 그만큼 음성 데이터가 많다. 

개인적인 추측이지만 그래서인지 음성 관련 연구팀도 존재하고, 지속적인 연구로 독자적인 모델도 발표할 수 있었던 것이라 생각이 든다.

<br>

<br>

참고로 VocGAN은 발표된지 얼마 되지 않아 관련된 내용이 거의 없다. 

그렇기 때문에 아래 링크의 NC소프트 공식 블로그에서 내용을 전적으로 참고하여 작성했다.

https://blog.ncsoft.com/vocgan-ai-20200922-02/ <br>

<br>

# 요약

이전에 개발된 MelGAN은 품질이 부족하거나, Mel-Spectrogram의 음향 특성이 제대로 일치하지 않는 경우가 많았다.

VocGAN은 이를 해결하고자, 속도를 조금 낮추는 대신 더 안정적이고 일관된 Mel-Spectrogram을 추출할 수 있게 했다.

이런 접근이 가능하게한 알고리즘은, multi-scale waveform generator을 적용했으며, hierarchically-nested discriminator 이다.



VocGAN의 핵심은 새로운 화자에 대한 새로운 Vocoder 모델을 갖고 Training을 해야 한다는 점을 해결하고자 재적되었다.

이를 통해 연구개발 속도가 빨라졌으며, 추가적으로 품질도 향상되었다고 한다. 



# Introduction

이 논문에서는 VocGAN을 WaveGAN을 주로 비교 대상으로 하였으며, WaveGAN의 경우 훌륭한 정확도로 인한 높은 계산성으로 GPU에서 밖에 학습이 불가능하다.

그 이후에 나온 MelGAN도 훌륭했지만, MelSpectrogram의 특징이 제대로 일치하지 않는 파형을 종종 생성하여 실제로 활용하기에는 무리가 있었다.

그래서 이 VocGAN에서는 영상 합성에서 좋은 결과를 보여준 알고리즘인,  joint conditional and unconditional  JCU을 활용하였으며, 이를 통해 파형 합성에 좋은 결과를 보인 STFT 손실을 결합하여 출력 품질을 개선했다고 한다.



# 셋





# 넷





# 다섯





# 마지막?

