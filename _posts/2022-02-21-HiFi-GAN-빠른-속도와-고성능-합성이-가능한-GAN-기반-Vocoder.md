---
title: HiFi-GAN - 빠른 속도와 고성능 합성이 가능한 GAN 기반 Vocoder
author: HW
date: 2022-02-21 11:18:03 +0800
categories: [Speech Synthesis]
tags: [Vocoder]
math: true
---



## **Abstract**

- **HiFi-GAN**은 오디오가 sin파로 이루어진 점을 고려하여 **주기적인 패턴을 모델링 하는 것에 집중하여** 품질을 향상시켰다.

- 이 모델의 구조에서 특이한 점은 1개의 Generator와 2개의 Discriminator로 이루어진 것이다.

- 이 결과로 사람과 거의 유사한 수준의 음성을 생성하여 

- 성능도 뛰어나고 파라미터도 적어 시간이 많이 단축되었다.
   CPU는 13.4배, GPU는 167.9배 더 빠르고, MelGAN보다 14.63 빠른 속도를 기록했다. 

 <br>

## **Model Architecture**

- HiFi-GAN은 1개의 Generator와 2개의 Discriminator(multi-scale, multi-period)로 구성되었다.

- 전통적인 GAN과 동일하게 Generator와 Discriminator 사이에 adversarial로 학습을 진행하며, 학습 안정성과 성능을 위해 2개의 loss를 추가했다.

### **Generator**

![generator](/assets/img/hifigan/generator.png)

- **Generator**의 베이스는 fully convolutional NN이다. 

- Mel-spectrogram을 input으로 받아 output(waveform) sequence와 시간 길이가 일치해 지도록 transposed convolution을 통해 up-sampling을 진행한다. 

- **Transposed convolution**
   MRF(multi-receptive field fusion) 모듈 구성을 따르며, 다음 단락을 찾는다.
   이 때, generator에 노이즈를 추가적으로 input으로 넣어주지 않는다.

- **MRF**
  Generator에 사용한 MRF는 다양한 길이의 패턴을 병렬적으로 관찰하도록 모듈을 구성했다.<br>
  여러 개의 residual block에서 각각 kernel size와 dilation rate를 조절하여 다양한 receptive field를 형성한다. 그리고 여기서 나온 output을 return 하는 구조를 가진다.

- 위 그림에서 보다시피 $$k_{u}, h_{u}, k_{r}, D_{r}$$등 파라미터를 조절하여 합성 효율과 퀄리티 사이에 균형을 임의로 변경하여 조절할 수 있다.

### **Discriminator**

![discriminator](/assets/img/hifigan/discriminator.png)

- 앞서 언급했듯이 위 그림과 같이 2개의 Discriminator(MSD, MPD)가 존재한다. 

- **MSD**는 Mel-GAN의 구조로, 다른 레벨에서 오디오에 대한 연속된 평가를 수행한다. 
   MSD는 연속적인 패턴을 찾아내는 것에 용이하고, 장기 의존성에 유리하다.

- Mel-GAN과 다르게 새로 추가된 **MPD**는 음성 오디오가 여러 시간동안 sin파로 이루어진 점을 고려하여, **오디오 속에 다양한 주기적인 패턴을 파악하기 위해 사용**되었다. 

- MPD는 각각의 input audio의 signal에 대해 sub-discriminator가 각각 적용되어 조절한다.

#### **MPD(Multi-Period Discriminator)**

![mpd](/assets/img/hifigan/mpd.png)

- MPD는 sub-discriminator가 뭉친 것으로, 각각 동일한 간격의 input audio만 허용한다.
   Sub-discriminator는 input의 각 부분에서 나오는 암묵적으로 다른 구조를 파악하는데 사용된다. 중복을 무시하고, 간격을 [2, 3, 5, 7, 11]로 설정했다. 

- 특이한 점으로, 2D Stride Conv를 수행하기 때문에 기존 1D 구조인 audio를 2D로 변환하는 reshape를 진행한다.  

- 또한 모든 Conv layer의 axis(너비)축의 kernel size를 1로 제한하여 periodic 샘플을 독립적으로 처리한다. (ReLU + weight norm)

- 장점: period audio를 샘플링 하는 대신 2D 데이터로 구성하여 MPD의 gradient를 모든 time-step에 전달할 수 있다.

#### **MSD(Multi-Scale Discriminator)**

![msd](/assets/img/hifigan/msd.png)

- MPD는 분리된 샘플만 받아낼 수 있어 MSD를 추가하여 audio의 연속적인 평가가 가능하도록 했다.

- 3개의 sub-discriminator가 섞인 구조로 각각 다른 input scale을 갖는다. (raw audio, X2 average-pooled audio, X4 average-pooled audio)

- Stride, grouped Conv 레이어로 이루어져 있다. (leaky ReLU, weight norm)

### **Continually**

- MSD와 MPD는 mixture 측면에서 유사하다고 볼 수 있는데, 이는 Markovian window-based fully unconditional discriminator로 output을 평균화하고 조건부 discriminator를 갖는다.

- 또한 MPD는 prime number를 사용하여 가능한 많은 period 데이터를 구별하는 장점이 있다.

 <br>

## **Experiments**

- Unseen data에 대한 평가를 위해 VCTK multi-speaker 데이터셋을 사용했다. VCTK는 44200개 발화로 이루어져 있으며, 109명이 참여한 44시간 데이터셋이다. 메모리 효율을 위해 44 kHz를 22kHz로 변환했으며, 무작위로 9명의 화자를 선택해 사용했다.

- GPU와 CPU 환경 모두에서 측정하기 위해 GPU는 V100 1개를 사용했고, CPU는 인텔 모바일 i7으로 수행했다.

- **Version** – 3가지 버전으로 실험을 진행했다. 
   V1은 $$h_{u}=512, k_{u}=[16,16,4,4], k_{r}=[3,7,11], D_{r}=[[1,1],[3,1],[5,1]*3]$$이다. 
   V2는 V1의 hidden dim만 ¼한 $$h_{u}=128$$이고 그 외에는 V1과 동일하다.
   V3는 receptive field를 넓히면서 layer 수를 줄이고자 kernel size와 dilation rate를 선택했다.

<br>

## **Result**

### **Audio Quality and Synthesis Speed**

- 성능 측정을 위해 전통적인 MOS 방식을 채택했고, 속도 측정의 경우 mel-spectrogram가 audio로 변환되는 순간의 기간을 측정하여 기록했다. 

- MOS 테스트를 위해 활용하지 않은 랜덤한 50개 발화를 선택했으며, 이들 모두 예측된 spectrogram이 아닌 Ground Truth spectrogram을 활용했다.

![result](/assets/img/hifigan/result.png)

- V1은 1392만개의 파라미터를 가지면서, **Ground Truth와 0.09 차이**로 사람과 구분이 불가능할 정도의 수준이라고 한다. 그리고 V1이 속도 측면에서 많은 파라미터를 가진 만큼 MelGAN보단 느리지만 Waveglow보다 7배 빠른 모습을 확인할 수 있다.

- V2는 파라미터가 많이 줄어든 만큼 속도 측면에서 V1보다 약 5배 더 빠른 것을 확인할 수 있다. MOS도 4.23으로 GT와 0.22 차이를 확인할 수 있다.

- V3는 receptive field를 늘려 수용할 수 있는 데이터가 많아지며 속도 측면에서 V1보다 약 7배 더 빠른 속도를 갖고, MOS는 WaveNet(MoL)과 비슷한 4.05를 기록함을 확인했다.