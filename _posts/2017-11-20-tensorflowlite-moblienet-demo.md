---
layout: post
title: TensorFlow Lite 101 - MoblieNet 맛보기
description: TensorFlow Lite 프리뷰 버전에서 공개된 데모의 버그를 잡고 빌드해서 테스트 해봅니다.
author: jwkang
subdir: tensorflowlite-mobilenet-demo
thumbnail: thumbnail.png
comments: true
---

아기다리 고기다리던 TensorFlow Lite Preview 버전이 릴리즈되었습니다([Link](https://www.tensorflow.org/mobile/tflite/))!!
신나게 [pre-built binary](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/contrib/lite#downloading-the-pre-built-binary)를 인스톨
해보니 에러가 나더군요(~~구글 디스아님~~).
그래서 GitHub에 올라와 있는 데모를 직접 빌드해서 삼성의 명품 갤럭시 S7에서 동작시켜봤습니다.
데모는 구글이 공개한 컴퓨터 비전 모델인 [MobileNet](https://arxiv.org/abs/1704.04861)을 가지고 사물을 분류(Object Classification)하여 인식하는 앱입니다.



## 동작 화면

일단 보여주고 시작합니다. 갤럭시 S7으로 동작시킨 결과입니다. 맥주병과 머그컵 그리고 마우스를 인식시켜 보았습니다.
테스트 상으로는 맥주병과 마우스를 잘 인식하는 반면 머그컵은 잘 인식하지 못하는 군요.
그리고 멀리서 찍으면 비전상에 다양한 사물이 들어와서 그런지 인식률이 떨어지는 것을 확인하였습니다.

{% include image.html subdir=page.subdir name='demo.gif' caption='그림1: 갤럭시 S7 + TensorFlow Lite + MobileNet으로 마우스 + 머그컵 + 맥주병 인식' %}


## TensorFlow Lite 개요

구글은 IoT 및 모바일 디바이스에 머신러닝 엔진을 탑재하기 위해 그동안 많은 것을 준비해 왔습니다.
여기저기서 주워들은 바를 바탕으로 구글의 머신러닝 비전을 해석해 봤습니다(많은 의견 바랍니다).

1. **Pre-training Model in Cloud**: 구글은 [Cloud ML](https://cloud.google.com/ml-engine/)과 [TensorFlow](https://www.tensorflow.org/)를 개발 하였습니다. 클라우드에 있는 public 데이터로 훈련하여 어느정도 성능이 보장되는 모델을 사용자들에게 제공합니다.
2. **Local fine-training with private data**: 구글은 [Colaboratory](https://research.google.com/colaboratory/unregistered.html)을 [TensorFlow](https://www.tensorflow.org/)와 함께 지원합니다. 사용자들이 손쉽게 개인 데이터를 TensorFlow를 통해서 훈련시킬수 있도록 합니다.
3. **Low complex inference in mobile!**

**3.**을 위해서 구글이 제공하는것이 바로 [TensorFlow Lite](https://www.tensorflow.org/mobile/tflite/)입니다. 따라서 TensorFlow Lite의 목적은 모델의 훈련에 있는 것이 아니고 모바일 환경에서 낮은 복잡도와 적은 용량으로 모델를 구동하는 것에 있습니다.
TensorFlow Lite는 기존에 TensorFlow로 훈련된 모델을 "TensorFlow Lite Converter"를 통해 TensorFlow Lite 모델(tflite)로 변환하여 Andoird /iOS 환경에서 사용하기 쉬운 인터페이스를 제공합니다.

{% include image.html subdir=page.subdir name='tensorflowlite.jpg' caption='그림2: TensorFlow Lite 아키텍쳐 (https://www.tensorflow.org/mobile/tflite/)' %}

## 빌드 환경 준비하기

- 먼저 Android Studio 3.0 가 필요합니다. 인스톨 해주세요([다운로드](https://developer.android.com/studio/index.html)).
- 이전에 TensorFlow Repository를 클론 안하신 분들은 적당한 장소에 클론합니다. 참고로 데모는 클론한 Repository의 "tensorflow/contrib/lite/java/demo"에 위치해 있습니다.
```bash
$ git clone https://github.com/tensorflow/tensorflow
```
- 빌드 툴인 bazel을 최신버전으로 업데이트 해야합니다. 현재 0.7이 최신 버전입니다.
```bash
$ brew upgrade bazel
$ bazel version
Build label: 0.7.0-homebrew
Build target: bazel-out/darwin_x86_64-opt/bin/src/main/java/com/google/devtools/build/lib/bazel/BazelServer_deploy.jar
```
- Android Studio 3.0 인스톨 후에 Android SDK Tools API version을 최신버전으로 업데이트 해주세요. 현재 26.1.1이 최신이네요.
- 다음으로 C++ Native 코드를 빌드하기 위한 NDK를 업데이트 합니다. 구글문서에는 14버전 이상이면 된다고 했지만 기왕하는거 최신버전인 16으로 합니다.

{% include image.html subdir=page.subdir name='android_sdk_setup.png' caption='그림3: Android SDK Tools + NDK 업데이트' %}

### 테스트 환경 정리

- 테스트 디바이스: Samsung Galaxy S7 (SM-G930L) + Android 7.0 (Nougat)
- 빌드 환경:
  - Mac OSX 10.11.6
  - bazel Version : 0.7
  - Android Studio 3.0
  - Android Build Tools Level: 26.1.1
  - Android NDK Version: 16.04442984

## APK 빌드하고 실행 후 1차 실패

1. Android Studio를 실행해서 "tensorflow/contrib/lite/java/demo" 폴더를 선택해서 프로젝트를 open합니다.
2. Android Studio상에서 부족한 라이브러리나 빌드 툴을 IDE가 시키는데로 인스톨하고 Gradle Sync 진행시킵니다.
3. 테스트 안드로이드 디바이스를 USB로 랩탑과 연결합니다. 여기서 중요한 것은 안드로이드 디바이스의 OS버전이 API level 23이상을 지원해야 한다는 것입니다. 즉 마쉬멜로 이상이겠네요(저는 갤럭시 S7 + 누가로 진행하였습니다).
4. 준비가 다 되면 APK를 빌드하고 연결된 디바이스에서 Run합니다.
5. 빌드가 성공하고 구동이 잘되는줄 알았으나....

{% include image.html subdir=page.subdir name='beer_bottle.jpg' caption='그림4: (a) 학습모델이 없어서 "Uninitialized Classifier or invalid context."라고 나온다. (b) 모델 넣고 다시 빌드 해서 성공!' %}

## MobileNet 모델을 다운로드해서 다시 빌드 후 성공

- 성공한 줄 알았으나, **그림4-(a)** 처럼 "Uninitialized Classifier or invalid context."라는 메세지와 함께 동작을 하지 않았습니다. 여차여차 알아본 결과 모델이 없이 껍데기만 빌드 했던 것이 원인이었습니다.
- wget을 통해서 직접 MobileNet 모델을 다운로드 합니다.
```bash
$ wget https://storage.googleapis.com/download.tensorflow.org/models/tflite/mobilenet_v1_224_android_quant_2017_11_08.zip
```
- 다운로드한 파일의 압축을 풀면 아래와 같이 두 파일이 나옵니다.
  1. label.txt : 라벨 정보
  2. mobilenet_quant_v1_224.tflite: TensorFlow Lite pre-trained 모델
- 위 두 파일을 "tensorflow/contrib/lite/java/demo/app/src/main/assets/"에 위치시키고 다시 APK를 빌드합니다.
- 테스트 디바이스로 RUN을 하면 그림4-(b)와 같이 잘 동작하는 것을 확인하였습니다.

## 모델사이즈 비교

- MobileNet의 모델사이즈는 뉴럴넷안의 파라미터의 개수에 비례합니다. 구글에서는 이것을 모델안에서 사용된 곱셈기(Multiplyer)와 누적덧셈기(Accumulator)의 개수 정량화 하였습니다([링크참고](https://github.com/tensorflow/models/blob/master/research/slim/nets/mobilenet_v1.md)).
- 당연히 모델사이즈가 클수록 모델이 더 정교하기 때문에 예측정확도가 높습니다.
- 구글에서 제공하는 MobileNet중 가장 큰모델과 작은 모델 그리고 TensorFlow Lite 모델의 바이너리파일 사이즈를 비교하였습니다. 모델의 예측 정확도는 구글에서 [ILSVRC-2012-CLS](http://image-net.org/challenges/LSVRC/2012/browse-synsets) 데이터 셋을 통해서 측정한 결과입니다.
  1. [mobilenet_v1_1.0_224](http://download.tensorflow.org/models/mobilenet_v1_1.0_224_2017_06_14.tar.gz) : 67.9 MB, Top-1 Accuracy=70.7, Top-5 Accuracy=89.5
  2. [mobilenet_v1_0.25_128](http://download.tensorflow.org/models/mobilenet_v1_0.25_128_2017_06_14.tar.gz): 7.6 MB, Top-1 Accuracy=41.3, Top-5 Accuracy=66.2
  3. [mobilenet_quant_v1_224.tflite](https://storage.googleapis.com/download.tensorflow.org/models/tflite/mobilenet_v1_224_android_quant_2017_11_08.zip): 4.3 MB, Top-1 Accuracy=??, Top-5 Accuracy=??
- 확실히 TensorFlow Lite 모델의 사이즈가 작습니다.

## 개인적인 감상

- 모델 바이너리사이즈은 이전 TensoFlow모델 ([MobileNet_v1_1.0_224](http://download.tensorflow.org/models/mobilenet_v1_1.0_224_2017_06_14.tar.gz))에 비해서 상당이 줄어든 모양! 거의 1/50 수준입니다.
- 하지만 모델사이즈가 줄어든만큼 ([mobilenet_v1_0.25_128](http://download.tensorflow.org/models/mobilenet_v1_0.25_128_2017_06_14.tar.gz)에 비해서도!) 예측 정확도에서 손실이 있을텐데요. 같은 데이터 셋을 이용해서 측정을 안해봐서 잘 모르겠네요.
- 테스트한 apk가 디버그 빌드이긴 하지만 7.8MB 정도 입니다. 여기서 mobilenet_quant_v1_224.tflite 모델사이즈만 4.3MB입니다. 국민앱 카카오톡이 37MB정도 인데 테스트앱이 7.8MB이면 좀 큰 편이군요.
- 딥러닝이 모바일에 가볍게 적용되기 위해서는 아직 모델의 바이너리사이즈 부분에서 상당한 개선이 필요한듯 합니다. 당연히 정확도가 동반되어야 하겠죠.
- 사운들리 코어에 현재 버전의 TensorFlow Lite를 적용할 수 있을지는 다소 흐림이네요 ㅠㅠ. 상당한 커스터마이즈가 필요한듯!
- 다음 TensorFlow Lite 101에는 자체 모델을 가지고 포스트 하길 바라며 마침니다 :-)

## 참고자료 및 출처
- [Introduction to TensorFlow Lite 구글 문서](https://www.tensorflow.org/mobile/tflite/)
- [TensorFlow Lite Preview GitHub (TensorFlow Lite)](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/contrib/lite)
- [Google Developer Blog](https://developers.googleblog.com/2017/11/announcing-tensorflow-lite.html)
- [MobileNet GitHub (MobileNet_v1)](https://github.com/tensorflow/models/blob/master/research/slim/nets/mobilenet_v1.md)
- [TensorFlow Lite Image from CloudMile](https://www.mile.cloud/single-post/tensorflow-lite/)
