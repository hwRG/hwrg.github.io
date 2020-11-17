---
title: 딥러닝 기반의 음성합성 Vocoder - Wavenet
author: HW
date: 2020-09-20 17:00:00 +0800
categories: [Speech Synthesis]
tags: [Wavenet]
math: true
image: /assets/img/insert/wavenet/main.png
---



# **들어가면서**

앞서 설명한 Tacotron에서는 Text Preprocessing부터 Encoder와 Decoder과정을 거쳐 Mel-Spectrogram을 예측해 생성하는 역할을 했다.<br/>

그 Mel-Spectrogram을 그대로 활용해도 되지만, 발화자의 특징을 더 부각시키기 위해 Vocoder를 뒤에 추가하여 Post Processing을 진행한다.<br/><br/>

Tacotron에서는 Griffin-Lim을 사용했는데, 구글에서 자체적으로 더 성능이 뛰어난 Wavenet을 만들어서 정확도를 상당히 높였다.<br/>

그래서 이번 포스팅에서는 Wavenet에 대한 전반적인 이야기를 해보고자 한다.



# Wavenet

Wavenet은 Grffin-Lim과 다르게 속도보다 정확도에 더 우선시한 Vocoder이다. 그래서 실제로 Wavenet을 발표한 구글의 DeepMind의 홈페이지(아래 링크)에 가면 이에 대한 Sample을 들을 수 있다.

https://deepmind.com/blog/article/wavenet-generative-model-raw-audio

<br>

![exTTS](/assets/img/insert/wavenet/residual.png)

![exTTS](/assets/img/insert/wavenet/casual_layer.png)

이 모델은 실제 음성 파형을 input으로 대입해 convolution 연산을 하는 layer을 구성하여 만든 모델이다.

기존의 Concatenative TTS처럼 많은 단점(Emotionless, Not natural)을 완화할 수 있는 오디오 파형을 직접 모델링해 음성을 생성한다.

<br>

<br>

# Model Architecture

![exTTS](/assets/img/insert/wavenet/architecture.png)

위 그림은 Wavenet의 전체 모델 구조이다. input으로 들어간 파형이 Casual Conv를 지나서 겹겹이 있는 Residual Layer에 들어가 나온 결과값이 이전의 layer 값들과 함께 고려해 학습하는 Skip-Connection을 사용했다.

<br>

이후에 ReLU와 Softmax로 가장 큰 출력 값을 부여받은 클래스가 확률이 가장 높은 것을 선택하여 Output을 출력한다.

<br>

<br>

# Causal Convolution

![exTTS](/assets/img/insert/wavenet/casual_layer.png)

input으로 들어간 파형이 Casual Convolution을 거치는데 그 구조는 다음과 같다.<br>

Input으로 들어온 파형을 여러개를 선택하여 그들을 하나만 사용하는 것이 아니라 여러 개의 input을 반영하여 hidden layer을 거쳐 ouput을 출력하는 구조이다.<br><br><br>

# DILATED CAUSAL CONVOLUTIONS

![exTTS](/assets/img/insert/wavenet/dilate_casual_layer.png)

위에서 언급한 Causal Conv는 최근 input과 멀지 않은 input을 활용한다.<br>

그렇기 때문에 Receptive Field의 개수를 확장시키고자 dilated causal conv를 활용했다.<br>

이렇게 할 경우 큰 폭의 연산량 증가 없이 Receptive Field를 크게 할 수 있다.<br><br><br>



# Formula(Conditional Probability)

![exTTS](/assets/img/insert/wavenet/fomula.png)

오디오 파형 데이터를 직접 사용해 새로운 파형을 모델링 할 때 사용하는 확률 식을 활용한다.

위 식은 조건부 확률 분포 식으로, Xt는 오직 이전 샘플에만 의존한다.<br>

이 식의 결과는 T step일 때 위에서 언급했던 마지막 부분인 Softmax로 출력된다.

이는 loglikelihood로 되는데, validation set을 통해 overfitting과 underfitting을 측정할 수 있다.<br><br><br>

# Residual block

![exTTS](/assets/img/insert/wavenet/residual_block.png)

오디오 샘플16비트를 사용하여 step마다 65536개의 확률을 다뤄야 하는데, 

이를 μ-law Companding를 통해 256개 중 하나의 값으로 양자화 적용한다. (특정 상황에 강조)<br><br><br>

# Wavenet 성능

![exTTS](/assets/img/insert/wavenet/performance.png)

위 사진에서 볼 수 있듯이 이전의 모델(Concatenative TTS)과는 비교가 안될 정도로 MOS 점수가 높은 것을 알 수 있다. 심지어 편차도 줄은 것을 확인할 수 있다.

이는 사람의 말을 8비트로 스케일링 했을 때 나오는 값과 거의 차이가 없는 수준이다.

<br>

<br>

그러나 앞서 말했듯이 Wavenet은 성능은 좋지만, 제작 과정에서 굉장히 많은 시간을 요구한다.<br>

그래서 다음에 소개 할 Waveglow, MelGAN등은 다양한 Neural Network를 통해 속도와 정확도 모두 잡으려고 한다. 이는 뒤에서 포스팅 할 예정이다.