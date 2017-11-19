---
layout: post
title: TensorFlowLite 맛보기 - MoblieNet demo 빌드 + 인스톨 + 실행
description: Tensorflow Lite 프리뷰 버전에서 공개된 데모앱의 버그를 잡고 빌드해서 테스트 해봅니다.
author: jwkang
subdir: tensorflowlite-mobilenet-demo
comments: true
---
아기다리 고기다리던 TensorFlow Lite preview 버전이 릴리즈되었습니다.
([[Link]](https://www.tensorflow.org/mobile/tflite/))
안타깝게도 [pre-build되어 공개된 apk](https://storage.googleapis.com/download.tensorflow.org/deps/tflite/TfLiteCameraDemo.apk) 는 인스톨 에러가 나는 관계로.... (구글 디스아님) 
제가 Github에 올라와 있는 데모를 빌드해서 삼성의 명품 갤럭시 S7에서 동작시켜봤습니다.
Demo는 구글이 공개한 컴퓨터 비전 모델인 MobileNet을 가지고 사물인식 (Object classification)을 하는 앱입니다.

## 동작 결과 
갤럭시 S7으로 동작시킨 결과입니다. 맥주병과 머그컵 그리고 마우스를 인식시켜 보았습니다. 
테스트 상으로는 맥주병과 마우스를 잘인식하는 반면 머그컵은 잘 인식하지 못하는 군요.
그리고 멀리서 찍으면 비전상에 다양한 사물이 들어와서 그런지 인식률이 떨어지는 것을 확인하였습니다.


## TensorFlow Lite 개요
TensorFlow Lite는 기존의 TensorFlow으로 학습된 모델을 TensorFlow Lite 모델로 (tflite)로 변환하여
Andoird /iOS 플랫폼에 집어넣어 모바일에서 Inference를 용이하게 만든 프래임워크 입니다. 
따라서 TensorFlow Lite의 목적은 모델의 훈련에 있는 것이 아니고 저복잡도+작은바이너리로 모델을 모바일 환경에서 구동하는 것에 있습니다.

{% include image.html subdir=page.subdir name='tensorflowlite.png' caption='구글문서 중 Tensorflow Lite 아키텍쳐 그림' %}

## 준비하기 
- 먼저 Android Studio 3.0 가 필요합니다. 인스톨 해주세요. ([[여기서 가능]](https://developer.android.com/studio/index.html))

{% include image.html subdir=page.subdir name='android_studio_3.png' caption='구글본가 Android Studio 3.0' %}

- TensorFlow repository를 클론 안하신 분들은 적당한 장소에 클론합니다. 참고로 demo는 클론한 repo의 "tensorflow/contrib/lite/java/demo"에 위치해 있습니다.
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
- Android Studio 3.0 인스톨 후에  Android SDK Tools version을 최신버전으로 업데이트 해주세요. 현재 26.1.1이 최신이네요.
- 다음으로 C++ Native 코드를 빌드하기 위한 NDK를 업데이트 합니다. 구글문서에는 14버전 이상이면 된다고 했으니 기왕하는거 최신버전인 16으로 합니다.
- 위에서 언급되지 않은 라이브러리나 빌드툴은 gradle sync과정에서 인스톨 할 수 있습니다. IDE가 시키는데로 따라하면 됩니다.

{% include image.html subdir=page.subdir name='android_sdk_setup.png' caption='Android SDK tool + NDK 업데이트' %}

## APK 빌드하고 실행 후 1차 실패

{% include image.html subdir=page.subdir name='failure_trial.png' caption='1차 시도 실패! 모델이 없으므로 Uninitialized Classifier or invalid context라고 나온다.' %}

## MobileNet 모델을 다운로드해서 다시 빌드 후 성

{% include image.html subdir=page.subdir name='beer_bottle.png' caption='모델 넣고 다시 빌드 해서 성공!' %}

## 모델사이즈 비교

## 메모리 사용량 체크

## 개인적인 감상
- 모델사이즈은 이전 TensoFlow모델에 비해서 상당이 줄어든 모양! 거의 1/50 수준입니다. 
- 하지만 모델사이즈가 줄어든만큼 정확도에 loss가 있을텐데 이건 정확하게 측정안해봐서 잘 모르겠네요.
- apk 바이너리가 디버그 빌드이긴 하지만 7.8MB 정도 입니다. 여기서 MobileNet 모델사이즈만 4.3MB입니다. 국민앱 카카오톡이 37MB정도 인데 테스트앱이 7.8MB이면 좀 큰편이군요. 
- 메모리도 상당이 머
- 딥러닝이 모바일에 가볍게 적용되기 위해서는 아직 바이너리사이즈 + 메모리 사용량 부분에서 상당한 개선이 필요한듯 합니다.
- 사운들리 코어에 현재 버전의 TensorFlow Lite를 적용할 수 있을지는 다소 흐림이네요 ㅠㅠ. 커스터마이즈가 필요한듯!

## 참고자료 및 출처
- [Introduction to TensorFlow Lite 구글 문서](https://www.tensorflow.org/mobile/tflite/)
- [TensorFlow Lite Preview Github](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/contrib/lite)
