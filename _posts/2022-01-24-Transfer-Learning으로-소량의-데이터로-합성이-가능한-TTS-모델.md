---
title: 소량의 데이터로 음성을 합성하는 Transfer Learning 활용 TTS 모델 
author: HW
date: 2022-01-24 16:33:47 +0800
categories: [Speech Synthesis]
tags: [Transfer Learning]
math: true
---



본 포스팅은 [Adapting TTS Models For New Speakers Using Transfer Learning] 논문에 대한 이해를 목적으로 작성

### **Abstract**

- 새로운 speaker를 추가할 때 보통 몇시간 단위의 좋은 퀄리티의 음성 데이터를 필요로 한다. 

- 보통 음성을 복제하기 위해, 적은 양의 새로운 목소리 데이터로 목소리를 효율적으로 활용하기 위해 pre-training에서 멀티 스피커 TTS을 사용해 만드는 시도가 있었다.<BR>
  그러나 큰 데이터셋은 대체로 노이즈가 많았고, 일정하지 못하다는 단점이 있다.

- 이 논문에서는 **transfer-learning** 가이드라인을 통해 몇 분의 데이터로 새로운 speaker(사람) 목소리를 생성하는 품질 좋은 싱글 스피커 TTS를 제안한다.

- 새로운 speaker의 데이터 양이 제각기 다른 상황을 감안하며, 타겟 음성과 자연스러운지와 유사한지 정도를 판단한다.

- 30분의 데이터로 27시간이 넘는 데이터로 학습한 각각 남녀의 데이터와 비슷한 수준의 결과를 확인할 수 있다. 

  <br>

### **Background and Related work**

- Tacotron 또는 FastPitch의 경우, 새로운 한정된 speaker의 데이터를 통해 adapt를 진행하는 것이 어려운 과제였다. 그래서 최근에 적은 speaker의 데이터 샘플로 사람의 목소리를 잘 합성해내는 모델을 만들기 위해 많은 논문들이 제안되고 있다.

- 대부분은 multi-speaker TTS를 사용하여 zero-shot voice cloning이 가능하도록 했다.
   그러나 그 모델은 많은 데이터셋이 필수적으로 요구된다는 한계가 존재한다.
   문제는 거대한 multi speaker dataset에는 대체로 노이즈가 끼며, 이에 따라 결과가 제대로 균일하지 못하다는 단점이 있다.

- 학습은 end-to-end로 이루어져 있으며, 데이터셋은 labeled TTS 데이터를 사용했다.

  <br>

### **Methodology**

##### **Spectrogram Synthesizer** 

- **FastPitch**를 선택했다. **FastPitch**는 **2개의 feed forward transformer(FFTr)** 스택으로 구성되어 있으며, 첫 번째 스택의 input은 phonemes token이다. Output은 hidden representation으로, 이는 모든 token의 duration과 평균 pitch를 예측하는데 활용된다. (각각 생성)

  ![predictor](/assets/img/adpat/predictor.png)

- 다음은 pitch가 프로젝션되어 hidden representation의 차원과 매치되어 추가된다.
   모두 합한 g는 따로 업샘플링되고 두 번째 FFTr을 통과한다. 이 때 output mel-spectrogram sequence를 생성하게 된다.

- Duration을 예측하기 위해 learnable alignment 모듈과 loss를 적용했다. (One TTS alignment to rule them all) 

- Alignment 모듈의 output d는 duration 예측 모듈을 위한 수도 타겟으로 제공된다.
   또한 pitch 예측 모델을 안내하기 위해 ground truth p을 활용하여, PYIN을 통해 도출된 d로 input token에 평균을 낸다.

- 학습할 때 end to end로 텍스트와 음성을 묶어내고, mel-spec y와 pitch p는 data-loading 파이프라인에서 계산된다.
   그리고 추론할 때 텍스트를 바로 음성으로 생성하기 위해 p와 d를 예측한다. 

  

##### **Vocoder**

- HiFi-GAN을 사용하는데, 이는 mel-spec to audio 과정을 더 업샘플링할 수 있도록 conv 구조를 변환한 generator 네트워크로 구성되어 있다.

- HiFi-GAN은 unseen speaker에 대한 mel-spec을 변환할 수 있다는 이점을 갖고 있으며, 실제로 HiFi-GAN으로 unseen data에 finetune했을 때 음성 품질이 향상된 것을 확인했다.

  <br>

### **Experiment**

##### **Dataset**

- Hi-Fi TTS 데이터셋(292시간, 10명이 최소 17시간, 44.1kHz)을 사용했으며, 8051 여자의 single speaker로 학습하고, finetuning은 6097 남자 데이터와 92 여자 데이터를 사용했다.

- 각 3명의 데이터는 최소 27시간을 넘으며, 텍스트와 음성과 묶어서 50개를 valid sample로 보관한다.

##### **Train setting**

- 92와 6097의 데이터를 1, 5, 30, 60분으로 나눈 subset을 두어 각각 적용했다. 
   Mixed finetuning을 위해 8051의 음성 데이터를 5000개 샘플(5시간)과 혼합한다.

- 미니 배치 gradient descent는 200*데이터크기(분)으로 하여 바로 finetuning을 진행하고, 1000*데이터크기로 반복하여 mixed finetuning method를 수행한다.

- 직접 finetuning method를 할 때는 적은 iteration으로 overfitting을 막는다.
   한 개의 V100 GPU로 5분의 데이터로 직접 finetuning을 진행하면 1시간보다 적게 걸린다.

 <br>

### **Finetuning Method**

- Mel-spectrogram만 생성하는 finetuning을 진행했을 때, pre-trained multi speaker에서 HiFiGAN을 사용하여 unseen speaker의 실제 mel-spectrogram을 변환했을 때 품질이 저하되는 것을 확인했다. 그래서 Finetuning 방법을 2가지로 나누어 진행했다.

   **Direct Finetuning**

- Pretrained TTS 모델 전체에 직접적으로 파라미터에 finetuning을 진행하는 방식이다.
   spectrogram 생성 모델은 speaker의 텍스트와 음성을 묶고, vocoder 모델에선 음성 샘플만 두어 spectrogram과 waveform을 묶는다. 미니배치와 adam, fixed learning rate를 사용한다.

   **Mixed Finetuning**
   
- Direct는 새로운 speaker의 학습 데이터에 의해 overfitting과 forgetting이 발생할 수 있다.
   이를 해결하기 위해 기존 speaker 데이터와 새로운 speaker 데이터를 finetuning 중 섞는 transfer learning method를 고안해냈다. 이를 위해 data-loading 파이프라인에서 샘플에서 기존, 새로운 speaker의 샘플 수를 미니 배치마다 일치시키도록 구성했다.

- FastPitch 모델을 2명의 speaker를 받아들일 수 있도록 임베딩 레이어를 추가하여 구조를 변경했다. 그리고 트레이닝 중 speaker의 id를 찾기 위한 임베딩을 추가하고, time step마다 첫 번째 FFTr에 넣기 전에 text embedding 또한 추가했다.

  ![FFTr_formula](/assets/img/adpat/FFTr_formula.png)

- Vocoder를 mixed data로 finetuning하는건 fastpitch보다 쉽다. Spectrogram representation에 이미 speaker 식별 요소가 있으면, 굳이 speaker conditioning을 추가할 필요 없다.

- 그럼 간단히 두 speaker의 spectrogram과 waveform만 묶어주면 된다. 마찬가지로 finetuning 중에 두 speaker에 대한 balanced data와 함께 미니배치를 사용한다.

<br>

### **Three Aspects**<br>

각 valid set에 대한 합성을 평가하기 위한 3가지 종목으로, naturalness, voice similarity, style similarity가 존재한다. 이는 target speaker의 valid set의 unseen 텍스트 샘플을 위해 결정했다.<br>
오디오 샘플과 pretrained TTS 모델에 대한 소량의 데이터셋으로 finetuning한 결과를 제시한다.<br>

#### **Naturalness**
- 전통적으로 음성 합성 결과를 위해 사용한 MOS를 통해 성능을 분석한다.
  

#### **Voice Similarity**
- Verification을 평가하기 위해 pre-trained speaker verification (SpeakerNet)을 사용했다. <br>
  실제와 만들어진 데이터의 embedding을 확인하기 위해 256 dim 말뭉치로 임베딩하여 2 dim의 t-SNE를 사용하여 나타냈다.<br>

![figure1](/assets/img/adpat/figure1.png)<br>

- 기본적으로 Equal Error Rate를 사용하여 평가를 진행하며, 위 t-SNE 그래프를 보면 5분마다 Direct와 Mixed Finetuning으로 합성된 모델이 비슷하게 분포해 있음을 확인할 수 있다. 
  

#### **Style similarity**

- 추가 예정