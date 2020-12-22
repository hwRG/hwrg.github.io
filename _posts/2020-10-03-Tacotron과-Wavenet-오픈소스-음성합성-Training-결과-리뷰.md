---
title: Tacotron과 Wavenet 오픈소스 음성 합성 Training 결과 리뷰
author: HW
date: 2020-10-03 17:00:00 +0800
categories: [Speech Synthesis]
tags: [Training]
math: true
image: /assets/img/sample/
---



# **들어가면서**

앞서 설명한 Tacotron과 Wavenet 두 모델을 기반으로 한 오픈소스로 Training을 해보았다.<br/>

코드는 깃허브에 올라온 오픈소스를 활용하여 진행했으며, 데이터는 한국어 데이터는 KSS, 영어 데이터는 LJSpeech를 활용했다.<br/>

<br/>

Training은 데스크탑에 있는 RTX2060 Super을 사용했고, Training 시간만 따지면 약 3일 정도 소요된 것 같다.



# 트레이닝 환경 세팅

리눅스를 사용하긴 하나, 비중이 굉장히 낮아서 처음 트레이닝을 할 때는 윈도우에 CUDA와 CUDNN을 설치하고 진행했다.

GPU는 실제 트레이닝하는데 기존의 GTX1060 3GB의 공간이 많이 부족하다 생각하여 큰 맘먹고 RTX2060 SUPER 8GB로 새로 구입하였다. 

<br><br>

# 코드 요약

음성 합성을 위한 데이터를 활용하고자 필요한 것은 다름 아닌, 데이터를 numpy로 변환해 주는 것이다.

우리가 갖고 있는 음성 파일을 npy로 변환해 줌으로써 처리를 쉽게 할 수 있도록 한다.

![exTTS](/assets/img/insert/trainset/npy_transform.png)

<br>

그 이후에 총 13100개의 데이터를 갖고 트레이닝을 진행한다.

![exTTS](/assets/img/insert/trainset/count.png)

<br><br>

# 실행 과정

![exTTS](/assets/img/insert/taco_wave_train/first_ckpt.png)

위와 같이 체크 포인트를 2000 간격을 두어 저장했다.

<br><br>

# 실행 결과

실행 결과를 따로 음성 파일로 업로드 하고 싶었지만, 복잡하여 일단 사진으로 대체했다.

![exTTS](/assets/img/insert/taco_wave_train/20000_1.png)

먼저 20000 step을 진행했을 때의 결과물로, 15000 step 쯤 진행되었을 때 어느정도 윤곽은 드러나면서 비슷한 발음을 해낼 수 있게 되었다.<br><br>

![exTTS](/assets/img/insert/taco_wave_train/40000_1.png)

그 이후로 40000 step이며, 20000 step과 별 다른 것이 없는 것을 알 수 있다.<br><br>

![exTTS](/assets/img/insert/taco_wave_train/120000_1.png)<br><br>

마지막으로 약 20시간동안 돌린 결과물인 120000 step이다.

크진 않지만 확실히 loss 값이 줄어든 것을 확인할 수 있었다.

<br><br>

# 결과물 확인

아직, 해결하지 못한 부분이 있는데, loss에 대한 제대로 된 지식을 갖고 있지 않아 약 15000 step 이후로 loss의 변화가 적은 이유와, 더 좋은 성능을 뽑아내는 것에 대한 한계가 존재한다는 점이 있다.

<br>

이 부분은 추가로 스터디를 진행해야 하지만, 학기 중 시간 관계상 제대로 진행하지 못했다.

겨울 방학 시즌에 딥러닝에 대한 기본적인 이해를 갖고, 실제로 완벽히 알아보고자 한다.



