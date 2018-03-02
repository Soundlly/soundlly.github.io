---
layout: post
title: AWS 컴퓨팅 서비스 타입 비교
description: AWS 에서 제공하는 컴퓨팅 서비스 중에 어떤걸 사용하는게 좋을지 고민해봤습니다.
author: humbledude
subdir: aws-service-types
thumbnail: thumbnail.png
comments: true
---

안녕하세요 사운들리 박근희 입니다.



최근에 사내 서버팀 스터디를 통해 AWS 에서 제공하는 컴퓨팅 서비스 타입에 대해 정리해 보았습니다.

보통 AWS 로 처음 웹 서비스를 시작하게 되면, EC2 와 S3 조합을 가장 기본적으로 사용하게 되는데요,

저는 AWS 에서 제공하는 컴퓨팅 서비스 중, 사운들리에서 사용해본 서비스인 EC2, ECS, Lambda 를 비교해 보았습니다.

이 외에도 Beanstalk, Lightsail 등이 있지만, 사운들리에서는 위 세가지만 쓰기에... ㅎㅎㅎㅎ

그리고 과금 방식과 운영 방식에서 오는 주의점에 대해서도 알아보겠습니다.



### EC2

#### 개요

AWS 에서 제공하는 가장 기본적인 컴퓨팅 서비스로서, 운영체제를 선택해서 server instance (가상머신)를 띄워 그 위에서 원하는 작업을 진행하게 됩니다.

#### 개발자 할일

개발자는 server instance 위에 원하는 application 들을 배치하며 instance 자체를 운영하게 됩니다.

어찌보면 물리 서버 한대를 쉽게 임대해서 쓰는 느낌 입니다.

#### 과금 방식

instance 의 분단위 사용량으로 계산하며, instance 의 성능에 따라 기준 비용이 달라집니다.

#### 운영 포인트

AWS 입문자에게 가장 진입이 쉬운 서비스이지만, instance 를 운영하게 되면서 발생하는 각종 운영체제 셋팅에 대한 압박이 있습니다.

이것저것 다양한것을 해보고 싶을때는 그냥 편하게 EC2 하나 띄워서 씁니다.

하지만 서비스가 진화해 감에 따라 범용으로 셋팅했던 EC2 를 점점 전문화 해 가면서 답답한 부분이 생길 수 있습니다.

#### 주의 사항

맨 처음 편하게 EC2 를 띄워서 아무렇게나 설정하고 application 을 배포 해서 쓸때는 몰랐지만, Load Balancer + Auto Scaling 을 적용하게 되면 AMI 관리가 번거로워지는 편.

Auto Scaling 설정을 해도 instance 올라가는 속도때문에 약 5분이 걸린다는 점..

EBS (Storage) 를 생각없이 빵빵하게 쓰다보면, 늘리기는 쉽지만 줄이기는 어려운 EBS 의 특성 덕에 요금에 적잖은 데미지를 받음.

비용 절감을 위해 안쓰는 (덜쓰는) instance 를 찾아내야 하고, 트래픽 패턴과 자원 사용량을 주시해야 함.



### ECS

#### 개요

AWS 에서 제공하는 기본적인 Docker 서비스로서, 짝꿍으로 제공되는 ECR 을 사용해서 Docker 이미지를 업로드 해둔 뒤, EC2 로 구성된 클러스터에 container 들을 운용하는 Docker Ochestration 서비스 입니다.

#### 개발자 할일

일단 EC2 cluster 셋업이 필요합니다. EC2 cluster 의 베이스가 되는 AMI 를 고르고, 필요시 ECS agent 도 설치해 둡니다.

이후 개발자가 application 을 docker image 로 만들어 repository 에 업로드 하게 되면, ECS 설정에 따라 EC2 cluster 어딘가에서 container 가 운영되게 됩니다.

#### 과금방식

구성된 EC2 cluster 에 대해서 EC2 과금방식으로 과금 하게 됩니다.

#### 운영 포인트

EC2 cluster 를 구성하기 위해 베이스가 되는 EC2 이미지를 잘 셋업해야 합니다. Docker 의 베이스 운영체제 셋업에 대해서는 다양한 글들을 참고하게 되겠네요. 이 베이스 EC2 cluster 를 셋업하는것과 ECS 개념을 이해하는데에 약간의 시간이 소요 됩니다.

한번 EC2 cluster 를 구성하고 ECS 설정을 마치면, 그 위에서 돌아가는 container 관리는 의외로 쉽습니다.

#### 주의 사항

ECS 에도 Auto Scaling 이 있어서 EC2 와 다르게 1초만에 Auto Scaling 이 될줄 알았는데, EC2 cluster 에 남는 자리가 없으면 결국 EC2 한대 더 띄워야 되서 똑같다는 점.. (오히려 EC2 cluster 에 scaling 설정을 추가 하기 복잡해짐)

요금 걱정때문에 EC2 cluster 에 빈자리 없이 꽉꽉 채워서 써야 한다면, 그냥 EC2 에 docker 셋업해서 쓰는게 더 간편함. (저희같은 소규모 클러스터에는 안맞나 봅니다)

(Fargate 라는 서비스가 등장하면서, 기존 EC2 Cluster 운영과 다른 포인트가 생겼습니다만, 아직 많은 Region 에서 지원하지 않고, 제가 아직 써보지 못해서 본문에서 제외 합니다. 개인적으로 좀 기대하고 있습니다 ㅎㅎ)



### Lambda

#### 개요

요새? 뜨고있는 serverless 방식의 컴퓨팅 서비스 입니다. 개발자는 운영체제 셋팅 등의 기반 기능에 대해서는 신경을 끄고, 처리해야되는 로직만 구현해서 돌릴 수 있는 서비스 입니다.

#### 개발자 할일

개발자는 Lambda 서비스가 호출할 수 있는 형식에 맞추어 application 을 개발해 업로드 하게 되고, 지정된 조건에 따라 업로드된 코드가 호출되게 됩니다.

#### 과금 방식

Lambda 가 호출된 횟수와 호출되었을때 사용한 메모리 + 컴퓨팅 시간에 따른 과금 정책을 더해서 과금하게 됩니다.

보통 웹서비스를 만들때 API Gateway 와 함께 사용하게 됩니다.

#### 운영 포인트

인프라 관리 걱정이 없기 때문에, 편하게 만들어서 편하게 호출해 봅니다. Load Balancing 도 딱히 신경 안써도 됩니다. 

Lambda 자체는 기한없이 월 100만건 Free Tier 가 있으므로, 쓸수 있다면 팍팍 써보는게 좋습니다.

또한 서비스 극초기에 web service 를 API Gateway + Lambda 로 구성해서 Free Tier 혜택을 받는다면, EC2 Free Tier 한대가 남기 때문에 다른 용도로 쓰기 좋습니다.

한마디로, 쓸수 있는 환경이라면 무조건 써보는게 좋습니다.

#### 주의 사항

API Gateway + Lambda 를 기본적인 serveless web application 으로 구성하는데, 호출 수가 많아질수록 비례하는 요금때문에, heart beat 성 트래픽이나, 요청이 잦은 서비스에는 쓰면 안됨 (정확히는 요금이 너무 많이 청구됨)

API Gateway + Lambda 의 초기 설정이 별로 친숙하게 다가오지 않음. (그래서 Serverless 등의 Framework 를 써서 극복해 봅니다.)



##### 참고
[EC2 Pricing](https://aws.amazon.com/ko/ec2/pricing/)

[ECS Pricing](https://aws.amazon.com/ko/ecs/pricing/)

[Lambda Pricing](https://aws.amazon.com/ko/lambda/pricing/)

[API Gateway Pricing](https://aws.amazon.com/ko/api-gateway/pricing/)

[AWS 요금 계산기](https://aws.amazon.com/ko/api-gateway/pricing/)

[Lambda 요금 계산기](https://s3.amazonaws.com/lambda-tools/pricing-calculator.html)

