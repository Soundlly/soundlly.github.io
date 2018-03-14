---
layout: post
title: AWS 컴퓨팅 서비스 타입 비교
description: AWS 에서 제공하는 컴퓨팅 서비스 중에 어떤걸 사용하는게 좋을지 고민해봤습니다.
author: humbledude
subdir: comparing-aws-service-types
thumbnail: thumbnail.png
comments: true
---

안녕하세요 사운들리 박근희 입니다.

최근에 사내 서버팀 스터디를 통해 AWS 에서 제공하는 컴퓨팅 서비스 타입에 대해 정리해 보았습니다.  
보통 AWS 로 처음 웹 서비스를 시작하게 되면, EC2 와 S3 조합을 가장 기본적으로 사용하게 되는데요,  
저는 AWS 에서 제공하는 컴퓨팅 서비스 중, 사운들리에서 사용해본 서비스인 **EC2**, **ECS**, **Lambda** 를 비교해 보았습니다.  
저도 처음 사운들리에서 EC2 를 기반으로 개발을 시작 했는데요,  
점점 AWS 를 배워감에 따라 EC2 가 오히려 다양한 제품중 마지막 선택지라는 생각이 들게 되었습니다. (switch.. case 중 default..)  
처음 AWS 를 접하시는 분과 어느정도 AWS 에서 서비스를 개발 해 보셨지만, EC2 만 쓰고 계시는 중이라면 한번 읽어 보시면서 다른 제품에 대해서 도전 의욕을 채워 보시길 바랍니다.



### [EC2](https://aws.amazon.com/ko/ec2)

#### 개요

AWS 에서 제공하는 가장 기본적인 컴퓨팅 서비스로서, 운영체제를 선택해서 server instance (가상머신)를 띄워 그 위에서 원하는 작업을 진행하게 됩니다.

#### 개발자 할 일

개발자는 server instance 위에 원하는 application 들을 배치하며 instance 자체를 운영하게 됩니다.
어찌 보면 물리 서버 한대를 쉽게 임대해서 쓰는 느낌 입니다.

#### 과금 방식

instance 의 **분 단위** 사용량으로 계산하며, instance 의 성능에 따라 기준 비용이 달라집니다.

#### 운영 포인트

- AWS 입문자에게 가장 진입이 쉬운 서비스이지만, instance 를 운영하게 되면서 발생하는 각종 운영체제 셋팅에 대한 압박이 있습니다.  
- 이것저것 다양한 것을 해보고 싶을 때는 그냥 편하게 EC2 하나 띄워서 씁니다.  
- 하지만 서비스가 진화해 감에 따라 범용으로 셋팅했던 EC2 를 점점 전문화 해 가면서 답답한 부분이 생길 수 있습니다.

#### 주의 사항

- 맨 처음 편하게 EC2 를 띄워서 아무렇게나 설정하고 application 을 배포해서 쓸 때는 몰랐지만, Load Balancer + Auto Scaling 을 적용하게 되면 **AMI 관리**가 번거로워지는 편.  
- Auto Scaling 설정을 해도 instance 올라가는 속도 때문에 scale out에 약 5분이 걸린다는 점..  
- EBS (Storage) 를 생각 없이 빵빵하게 쓰다 보면, **늘리기는 쉽지만 줄이기는 어려운 EBS 의 특성** 덕에 요금에 적잖은 데미지를 받음.  
- 비용 절감을 위해 안쓰는 (덜쓰는) instance 를 찾아내야 하고, 트래픽 패턴과 자원 사용량을 주시해야 함.



### [ECS](https://aws.amazon.com/ko/ec2)

#### 개요

AWS 에서 제공하는 기본적인 Docker 서비스로서, 짝꿍으로 제공되는 ECR 을 사용해서 Docker 이미지를 업로드 해둔 뒤, EC2 로 구성된 클러스터에 container 들을 운용하는 Docker Ochestration 서비스 입니다.

#### 개발자 할 일

일단 **EC2 cluster 셋업**이 필요합니다. EC2 cluster 의 베이스가 되는 AMI 를 고르고, 필요시 ECS agent 도 설치해 둡니다.  
ECS agent 는 EC2 cluster 의 각 instance 위에서 돌면서 개발자의 구령에 따라 docker 를 제어하는 역할을 수행합니다.
이후 개발자가 application 을 docker image 로 만들어 repository 에 업로드 하게 되면, ECS 설정에 따라 EC2 cluster 어딘가에서 container 가 운영되게 됩니다.

#### 과금방식

구성된 EC2 cluster 에 대해서 **EC2 과금방식 - 분 단위**으로 과금 하게 됩니다.

#### 운영 포인트

- EC2 cluster 를 구성하기 위해 베이스가 되는 EC2 이미지를 잘 셋업해야 합니다. Docker 의 베이스 운영체제 셋업에 대해서는 다양한 글들을 참고하게 되겠네요. 이 베이스 EC2 cluster 를 셋업하는것과 ECS 개념을 이해하는데에 약간의 시간이 소요 됩니다.  
- 한번 EC2 cluster 를 구성하고 ECS 설정을 마치면, 그 위에서 돌아가는 container 관리는 쉽습니다. (ECS cluster 구성을 하면서 ECS 를 공부하다보면 운영에 대한것 까지 다 알게 됩니다ㅎㅎ )

#### 주의 사항

- ECS 에도 Auto Scaling 이 있어서 EC2 와 다르게 1초만에 Auto Scaling 이 될줄 알았는데, EC2 cluster 에 남는 자리가 없으면 결국 **EC2 한대 더 띄워야 되서** 똑같다는 점.. (오히려 EC2 cluster 에 scaling 설정을 추가 하기 복잡해짐)  
- 요금 걱정때문에 EC2 cluster 에 빈자리 없이 꽉꽉 채워서 써야 한다면, 그냥 EC2 에 docker 셋업해서 쓰는게 더 간편함. (저희같은 소규모 클러스터에는 안맞나 봅니다)  
(**Fargate** 라는 서비스가 등장하면서, 기존 EC2 Cluster 운영과 다른 포인트가 생겼습니다만, 아직 많은 Region 에서 지원하지 않고, 제가 아직 써보지 못해서 본문에서 제외 합니다. 개인적으로 좀 기대하고 있습니다 ㅎㅎ)



### [Lambda](https://aws.amazon.com/ko/lambda)

#### 개요

요새(?) 뜨고있는 **serverless** 방식의 컴퓨팅 서비스 입니다. 개발자는 운영체제 셋팅 등의 기반 기능에 대해서는 신경을 끄고, 처리해야되는 로직만 구현해서 돌릴 수 있는 서비스 입니다.

#### 개발자 할 일

개발자는 Lambda 서비스가 호출할 수 있는 형식에 맞추어 application 을 개발해 업로드 하게 되고, 미리 지정된 이벤트 트리거에 따라 업로드된 코드가 호출되게 됩니다.

#### 과금 방식

Lambda 가 호출된 횟수와 호출되었을때 **사용한 메모리 + 컴퓨팅 시간**에 따른 과금 정책을 더해서 과금하게 됩니다.  
보통 웹서비스를 만들때 API Gateway 와 함께 사용하게 되는데, API Gateway 는 **호출된 횟수**에 따라 과금합니다.

#### 운영 포인트

- 인프라 관리 걱정이 없기 때문에, 편하게 만들어서 편하게 호출해 봅니다. Load Balancing 도 딱히 신경 안써도 됩니다.   
- Lambda 자체는 **기한없이 월 100만건 Free Tier** 가 있으므로, 쓸수 있다면 팍팍 써보는게 좋습니다.  
- 또한 서비스 극초기에 web service 를 API Gateway + Lambda 로 구성해서 Free Tier 혜택을 받는다면, **EC2 Free Tier 한대가 남기 때문에** 다른 용도로 쓰기 좋습니다.  
- 한마디로, 쓸수 있는 환경이라면 무조건 써보는게 좋습니다.

#### 주의 사항

- API Gateway + Lambda 를 기본적인 serveless web application 으로 구성하는데, 호출 수가 많아질수록 비례하는 요금때문에, heart beat 성 트래픽이나, 요청이 잦은 서비스에는 쓰면 안됨 (정확히는 요금이 너무 많이 청구됨)  
- API Gateway + Lambda 의 초기 설정이 별로 친숙하게 다가오지 않음. (그래서 Serverless 등의 Framework 를 써서 극복해 봅니다.)


EC2 는 AWS 관련 지식의 깊이에 상관없이 사용할 수 있어서 쉽게 접근 가능하지만,   
결국 클라우드 를 잘 사용하는건 클라우드에서 제공하는 다양한 서비스를 잘 이해하고, 문제에 맞는 솔루션을 적용하는 것이라고 생각 됩니다. *(물론 비용도 맞춰서....)*  
이 외에도 AWS 에서 제공하는 컴퓨팅 서비스에는 Beanstalk, Lightsail 등이 있는데요,  
각 상황에 맞는 솔루션 적용으로 즐겁고 저렴한 개발 되시길 바라겠습니다. ㅎㅎㅎ



##### 참고
- [EC2 Pricing](https://aws.amazon.com/ko/ec2/pricing/)
- [ECS Pricing](https://aws.amazon.com/ko/ecs/pricing/)
- [Lambda Pricing](https://aws.amazon.com/ko/lambda/pricing/)
- [API Gateway Pricing](https://aws.amazon.com/ko/api-gateway/pricing/)
- [AWS 요금 계산기](https://aws.amazon.com/ko/api-gateway/pricing/)
- [Lambda 요금 계산기](https://s3.amazonaws.com/lambda-tools/pricing-calculator.html)

