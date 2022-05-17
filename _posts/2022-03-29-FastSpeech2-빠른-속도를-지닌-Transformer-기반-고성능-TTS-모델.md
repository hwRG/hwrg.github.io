---
title: FastSpeech2 - 빠른 속도를 지닌 Transformer 기반 고성능 TTS 모델
author: HW
date: 2022-03-29 21:56:05 +0800
categories: [Speech Synthesis]
tags: [FastSpeech2]
math: true
---



## **Abstract**

![](https://velog.velcdn.com/images/hws0120/post/21f8203b-d5a0-4ac5-87c1-8ceba0417400/image.png)

- **FastSpeech2는 Fastspeech을 계승한 Non-Autoregressive 모델이며, Fastspeech의 단점을 개선**했다. FastSpeech의 단점은 1) Teacher-student distillation이 너무 복잡하며, 2) teacher 모델로부터 duration 추출이 확실하지 않다.

- 먼저 Teacher 모델에 의존하는 것을 극복하기 위해 Kaldi로 만들어진 MFA(Montreal Forced Aligner)를 이용해 duration을 추출했다. MFA를 사용하면 duration을 훨씬 정확한 단위로 추출해내어 결과가 훨씬 안정적이다.

- 여러 방법으로 추출한 음성의 정보들인 **Variation information을 input에 추가하여 확장성을 높였다**. 같은 구조를 가진 Variance Adaptor는 Duration, Pitch, Energy 3가지 항목을 각기 다른 블록으로 구성하여 각각에 대해 더 치밀하게 예측하도록 구성했다. 여기서 **Variance Adaptor는 Length regulator에서 추가 보완한 구조이며, pitch와 energy에 대해서도 추정**하도록 했다.

  <br>

## **Introduction**

최근 제안되는 논문들은 FastSpeech와 마찬가지로 Non-Autoregressive 기반으로 설계하여, 생성 속도가 빠르면서 안정적인 성능을 보이는 모델을 제시하고 있다.

이전 모델인 FastSpeech는 2가지 기법을 통해 one-to-many mapping 문제를 해결했다.

>1) Autoregressive teacher 모델에서 생성된 spectrogram을 학습 대상으로 하여 타겟의 데이터 분산을 최소화함<br>
2) Mel-spectrogram 길이와 일치하도록 텍스트 시퀀스를 확장하기 위해 duration 정보 추가<br>

이에 따라 FastSpeech는 3가지 단점이 존재했다.

>1) Teacher model에서 생성한 spectrogram은 실제 spectrogram보다 품질이 떨어짐<br>
2) Teacher-Student 모델이라 학습 파이프라인이 굉장히 복잡함<br>
3) Teacher 모델에서의 duration 추출이 충분하지 않음

FastSpeech2는 위 문제를 **Input과 target output 사이에 정보의 격차를 줄이는 방법**으로 one-to-many mapping을 완벽하게 해결했다. 이 정보의 격차를 줄이기 위해 FastSpeech 구조에 **pitch와 energy, 훨씬 정확한 duration 정보를 갖는 variation information을 conditional input**으로 넣는다. <br>

Pitch가 시간에 따라 변동이 커서 예측이 어려운 점을 극복하기 위해, Continuous wavelet transform(CWT) 알고리즘을 활용했다. 즉, FastSpeech2의 가장 핵심은 variational information을 사전에 명확하게 추출하여 그대로 사용해 안정적인 학습이 가능하도록 만들었다.
<br>

## **Model Architecture**

TTS는 텍스트와 목소리가 매핑되는 one-to-many 테스크다. 이에 따라 단순 파형 뿐 아니라, pitch와 duration 등 수많은 정보를 담고 있다. Non Autoregressive TTS는 text만을 input으로 하면 학습이 충분하지 않아 variation information에 overfitting이 발생하기도 한다. 따라서 FastSpeech2는 이 문제를 해결하고자 Variation information을 직접 활용한다.

>1) Phoneme Duration: 음성의 길이가 얼마나 되는지 확인할 수 있는 요소<br>
2) Pitch: 음성에 담긴 감정과 운율의 강도를 알 수 있는 핵심 요소<br>
3) Energy: Mel-spectrogram의 프레임 수준의 크기로, 음성의 양과 운율에 핵심 요소<br>



### **Variance Adapter**

3가지 variation information을 추출하고자 Variance Adapter에 다음 모듈을 추가한다. Variance Adapter의 모듈에서는 모두 MSE Loss를 사용한다.
![](https://velog.velcdn.com/images/hws0120/post/6e28b1d2-c58d-491f-890d-72da771dc56e/image.png)

**1) Duration Predictor:** Input으로 들어온 text에 phoneme의 각각의 hidden 시퀀스에서 얼만큼의 mel frame이 존재하는지 예측하고 비교한다. FastSpeech는 Autoregressive 모델에서 duration을 추출해 활용한 것과 다르게, MFA를 사용해 뽑은 duration을 target으로 사용해 예측하고 평가한다.



**2) Pitch Predictor:** pitch의 윤곽을 확실하게 예측하고자 CWT(Continuous Wavelet Transform) 알고리즘을 사용해 pitch spectrogram으로 분해하고, 이를 target으로 하여 학습한다. 그리고 Inference 과정에선 invert CWE로 pitch spectrogram을 예측한다. 일반적으로 f0는 pitch를 의미하고, log scale에서 256개로 양자화하고 pitch embedding p로 변환한다.



**3) Energy Predictor:** STFT를 수행한 프레임의 amplitude을 L2-Norm으로 계산해 에너지 값을 구한다. 마찬가지로 256개 값으로 양자화하고, 임베딩 e로 인코딩 후 hidden 시퀀스에 추가한다. 




### **Discussion**

>1)	Non-autoregressive 모델로 병렬 생성 가능<br>
2)	다른 Non-autoregressive와 다르게 duration 뿐만 아니라 pitch, energy 등을 고려하여 information gap을 축소<br>
3)	Prosody 향상을 위해 Continuous wavelet transform을 추가로 적용함<br>
4)	FastSpeech2s는 Autoregressive를 아예 사용하지 않아 속도가 굉장히 빠름(inference)

<br>

## **Experiments**

### **Dataset**

- FastSpeech와 마찬가지로 LJSpeech를 사용했다. (13100개 문장, 약 24시간 분량)
  12228개 데이터는 학습에 사용하고, 검증과 테스트 각각 349개, 523개를 활용했다.
- MFA로 grapheme-to-phoneme을 수행하여 phoneme sequence를 추출한다. 그리고 Waveform 데이터는 mel-spectrogram으로 변환한다. (Sampling rate: 22050, Window 1024, Hop: 256)
  <br>

## **Result**

### **Audio Quality**

![](https://velog.velcdn.com/images/hws0120/post/65deaec0-9900-4296-a567-98d7706a4e69/image.png)

- 20명의 테스터에게 위에 해당되는 음성을 들려주어 MOS와 CMOS로 결과를 보여주었다. FastSpeech에서 Waveglow를 사용한 것과 다르게 **Paralle WaveGAN**을 Vocoder로 활용했다. 

- FastSpeech2는 FastSpeech와 Autoregressive 모델의 성능을 약 0.1정도 더 높으며, 더 자세한 비교를 위해 CMOS로 명확한 차이를 비교한 결과를 보면 FastSpeech보다 0.8 더 높다. FastSpeech보다 성능이 더 향상된 이유는 variance information을 사용했다는 점이 있으며, Teacher-student distillation 파이프라인을 사용하지 않고 MFA로 duration을 학습에 사용한 것이 영향을 주었음을 알 수 있다.<br>

### **Speedup**

![](https://velog.velcdn.com/images/hws0120/post/3930dea2-58fb-4354-b499-eb558e4a7b8f/image.png)

- FastSpeech는 추론(예측)단계 에서 Transformer TTS와 비교해 높은 속도를 자랑했지만, 학습 단계에서 속도는 느리다. 그런데 FastSpeech2는 학습 속도가 개선되어 Transformer TTS에 비해 2.27배 빠르고, FastSpeech보다 3.12배 빠르다. <br>

### **Analyses On Variance Information**

 ![](https://velog.velcdn.com/images/hws0120/post/0a381a19-2c34-4648-b176-3efd1ab12fd8/image.png)

- Ground truth와 합성된 결과물의 pitch 값이 얼마나 차이 나는지 FastSpeech와 함께 비교했다. MAE로 오차를 계산했을 때 0.1만큼 줄어든 것을 알 수 있다. 즉, Variance Adapter의 영향이 크게 작용했음을 확인할 수 있다.
  ![](https://velog.velcdn.com/images/hws0120/post/f2824047-32de-41f9-91a5-2105114498d5/image.png)

- MFA를 사용해 duration을 추출한 결과물이 teacher model에서 뽑은 것과 길이 오차를 비교해 보았다. 오차도 teacher model에 비해 36.6% 줄어들었으며, FastSpeech에 MFA를 사용해 학습한 결과에서도 CMOS가 0.195 더 크다.
  <br>

## **Conclusion**

- FastSpeech2는 FastSpeech에서 teacher model 활용하는 것을 개선하고, Variance Adapter를 활용해 직접 variation information을 활용하여 견고한 TTS 결과물을 생성할 수 있게 되었다.

- 물론 duration을 추출하기 위해 MFA, pitch를 추출하기 위한 CWT 등 외부적인 요소를 사용해 복잡한 구조로 보일 수 있으나 현재로선 빠른 합성에 유용하다. 추후에 제안할 모델에선 더 간단한 솔루션을 찾아내어 end-to-end를 확실하게 만들어내고자 한다.