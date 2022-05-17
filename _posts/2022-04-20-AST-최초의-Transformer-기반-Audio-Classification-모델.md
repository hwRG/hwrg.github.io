---
title: AST - 최초의 Transformer 기반 Audio Classification 모델
author: HW
date: 2022-04-20 09:30:17 +0800
categories: [Audio Classification]
tags: [AST]
math: true
---



## **Abstract**
- 지난 10년간 CNN이 Audio Classification Task에서 오디오를 관련 라벨에 바로 매핑할 수 있다는 장점 덕분에 필수적으로 활용되어왔다. 그런데 최근 long range context를 포착하기 위해 self-attention을 적용한 여러 CNN 모델이 제안되고 있다.

- 그러나 필자는 CNN이 Audio Classification에 꼭 필요하지 않다는 것을 증명하고, 오직 attention만을 사용하면 더 좋은 성능을 낼 것이란 가정을 증명하고자 제안하게 되었다. 즉, 본 논문은 CNN을 사용하지 않고 attention 구조만을 활용한 Audio Classification의 최초의 Transformer 모델이다. 

- Audioset 데이터셋에서 0.485 mAP를 기록하고, ESC-50 데이터셋에서 95.6% 정확도를 기록하며 SOTA(State-Of-The-Art) 이상의 성능을 보이는 것을 확인할 수 있다.
  <br><br>

  
## **Introduction**
- Attention을 활용한 감정 인식, 명령 인식, 이벤트 분류 등이 있었지만, 가장 성능이 좋은 것은 Vision에서 활용한 Vision Transformer(ViT)였다. 특히 Pretrained ImageNet 정보를 AST로 전송하는 방식을 제안하여 성능을 크게 향상시켰다.

AST의 핵심 3가지 요소
1. 우수한 성능
2. 다양한 길이에 input을 받을 수 있고, 구조 변경 없이 다른 task에도 적용 가능
   → AudioSet 데이터셋이 1초부터 10초까지 다양한데 fixed 없이 그대로 적용 가능
3. 최신 CNN 구조보다 더 간단한 구조와 파라미터를 가지며, 학습 수렴이 빠름

### **Related Work**

- 이전에 CNN이 결합된 Transformer 모델은 두개를 합치거나, 아니면 Transformer 대신 Self-Attention만을 결합하는 방식을 사용했다. ViT는 이 모델의 기반이지만 고정된 차원의 input만 가능하다는 점에서 차별된 점이 있다. 
  <br><br>

  
## **AST Architecture**



![](https://velog.velcdn.com/images/hws0120/post/2c62e69a-26c1-4a38-b083-14cd2112bfd9/image.png)

- 구조는 요약하면 다음과 같다. 

1. Input 2D Spectrogram을 overlap하여 16 by 16 patch로 split
2. 생성한 1D patch에 linear projected
3. Projected patch에 1D position embedding 추가
4. Classification token이 sequence 앞에 추가
5. Transformer를 지나 classification token의 output이 linear을 거쳐 최종 classification

- Input data는 128 dimension의 log Mel filter-bank이다. 10ms 간격으로 Hamming window를 25ms 만큼 적용한 데이터를 활용한다. → 128 x 100t spectrogram

- Spectrogram을 6만큼 overlap하여 N 16 x 16 patch sequence로 나누고, N = 12((100t-16)/10)에서 가장 Transformer에서 효율적이다.

- 이후 768 크기의 Embedding으로 flatten하여 1D patch를 linear projection에서 활용한다.

- Transformer에서 순서에 대한 정보를 필요로 하여 patch sequence의 순서를 구하기 위해 각 patch마다 positional embedding으로 capture 가능하도록 한다. (2D 오디오에서도 가능하게)

- Classification Task로, Transformer 구조에서 Encoder 구조만을 그대로 활용했다. 여기서 [CLS]를 사용했는데, 이 token output은 spectrogram의 표현으로 사용된다.

- 마지막으로 Sigmoid를 갖는 Linear Layer는 spectrogram 표현 label에 대해 분류가 수행된다.
  <br>

  
### **ImageNet Pretrain**



![](https://velog.velcdn.com/images/hws0120/post/7acb79ba-a5e6-45d7-865b-ee80164d2040/image.png)

- Transformer는 CNN에 비해 더 많은 데이터가 요구되는 단점이 있다. ViT 논문에 따르면, Transformer는 이미지 분류 작업에 1400만 이상의 이미지가 있어야 CNN의 성능을 능가하는데, 오디오는 이미지보다 훨씬 데이터 확보가 어렵다는 문제가 존재한다.
- 문제를 해결하기 위해 AST는 cross-modality transfer learning를 활용했다. 이미 Vision에서 Audio로의 transfer learning 연구는 여럿 존재하지만, CNN으로 한정되어 있어 본 모델은 AST 모델과 구조가 거의 동일한 ViT를 Pre-train 모델로 채택했다.

- ViT와 embedding size나 patch size등은 모두 동일하나 완전 같은 건 아니라 수정이 필요하다. 
1) 임베딩 레이어의 RGB 3채널의 가중치를 평균 내어 1채널로 변경하고 Spectrogram을 mean 0, std 0.5로 normalize 수행
2) ViT의 input shape은 224^2 또는 384^2으로 매번 길이가 변하는 spectrogram과 일치하지 않아 positional embedding을 유의하여 ImageNet이 학습할 수 있어야 함. 따라서 cut(), bi-linear interpolate() → (추가) 함수를 positional embedding adaptation에 추가
ex) ViT에 input 384^2에 patch 16^2일 경우 384/16=24 PE는 24^2를 AST의 input 12x100에 적용하고, 이를 AST의 positional embedding으로 사용할 수 있게 된다.

- 위 과정을 통해 input 크기가 다름에도 ViT의 2D spatial knowledge를 활용할 수 있다.
  마지막으로, 본질적으로 vision과 audio의 classification이 다르기 때문에 ViT의 마지막 레이어를 버리고 AST에 맞게 초기화한다.

- 그리고 다른 하나는 전통적인 ImageNet을 사용하는데 384 X 384로 학습되며 8700만개의 파라미터를 가지는데, 이에 대한 학습 결과의 가중치를 사용하게 된다. ImageNet에서도 DeiT는 두 개의 CLS 토큰을 가지는데 이 또한 평균내서 하나의 CLS 토큰으로 바꾼다.
  <br><br>

  
## **Experiments**
### **Parameter**
- Label_dim – 클래스의 개수 (내 데이터셋 label 18개) – 0420 기준
fstride – frequency 차원에서 patch 분할 크기: 16*16은 겹침X / 10*10은 6만큼 겹침
tstride – time 차원에서 patch 분할 크기: 16*16은 겹침X / 10*10은 6만큼 겹침
input_fdim – input spectrogram의 frequency bin의 개수 (기본 128)
input_tdim – input spectrogram의 time을 의미 (1024일 경우 10.24초)
imagenet_pretrain – ViT 모델로 ImageNet 데이터셋을 학습한 가중치를 사용 (성능이 뛰어남)

### **Dataset**



![](https://velog.velcdn.com/images/hws0120/post/31663caf-aa46-45fc-9d7b-5dcbbf650f6a/image.png)

평가를 위해 Audioset과 ESC-50 데이터셋을 사용했다. 
- Audioset은 유튜브 영상의 특정 부분을 지정하여 레이블을 부여해 학습 데이터로 설정한다. 규모는 10초 단위로 약 200만개 데이터가 존재하며 527개의 레이블이 있다. 

- ESC-50 데이터셋은 Environmental Sound Classification를 위한 데이터셋으로, 50개 레이블(자연, 동물, 사람 행동, 실내, 실외 등)을 가진 평균 5초 데이터를 2000개 갖고 있다.
  <br><br>

  
## **Result**
- Audioset 데이터셋의 성능 평가는 Mean Average Precision(mAP)을 사용했다. Precision-recall 그래프에서 정량적인 비교가 가능한 average precision를 구해 0~1 사이 값으로 나타낸다. 하나의 데이터에 여러 레이블을 담고 있기에 물체 검출 분야 처럼 mAP를 사용한다.

![](https://velog.velcdn.com/images/hws0120/post/1f739faa-3a87-487b-a005-e003b4b31b06/image.png)

- CNN과 attention을 결합한 이전 모델과 성능을 비교했을 때, 앙상블을 적용하지 않았을 경우 PSLA에 비해 Full mAP 기준 0.015 증가한 것을 알 수 있다. 이때 ViT의 pretrain을 적용하지 않으면 앞서 언급했듯이 데이터가 적어 성능이 Full Set 기준 0.366이 나오면서 CNN과 MLP를 사용한 모델과 비슷한 성능을 보임을 알 수 있다.

![](https://velog.velcdn.com/images/hws0120/post/f717e9b8-00cb-4267-a431-2f38bed4ae2c/image.png)

- 좀더 직관적으로 보고자 ESC-50 데이터셋으로 accuracy를 측정했을 때, 이전의 SOTA 모델의 경우 86.5%를 보이지만, AST 모델은 88.7%을 달성한 것을 확인할 수 있다.

- 이때 Audioset pretrain을 추가로 적용하면 SOTA 모델은 94.7%, AST 모델은 95.6%의 결과를 보인다.
 <br><br>


## **Conclusion**
- Audio classification task에서 지난 10년간 CNN이 필수였으나, AST를 시작으로 더 이상 CNN가 주류가 되지 않고 오직 Attention만을 사용하는 Transformer로도 의미 있는 분류가 가능함을 보여준다.