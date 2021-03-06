---
layout: post
title: 사운들리 백엔드 이야기
subdir: soundlly-backend-story
thumbnail: soundlly-aws-blueprint.png
comments: true
---

사운들리는 **'귀에 들리지 않는 소리'** 를 이용해서 컨텐츠를 전달할 수 있는 SaaS 플랫폼을 서비스하고 있습니다.
제품의 구성요소는,

- 음파를 송신할 수 있는 송신단
- 음파를 모바일에서 수신할 수 있는 Android, iOS SDK
- 그리고 컨텐츠를 제공하고 데이터를 수집, 분석하는 백엔드로 구성되어 있습니다.

오늘은 구성 요소중 백엔드에 대해서 이야기 해보도록 하겠습니다.

{% include image.html subdir=page.subdir name='soundlly-solution-blueprint.png' caption='그림 1. 사운들리 솔루션 구성도'%}

사운들리의 인프라는 모두가 잘 아시는 [아마존 웹 서비스](http://aws.amazon.com/ko/)를 이용하고 있으며, 크게 컨텐츠를 제공하는 API서버 부분, 로그를 수집, 분석하는 부분, 그리고 컨텐츠를 관리하는 CMS 부분으로 이루어져 있습니다.

## 소프트웨어 스택

- **Java** : 현재 사운들리의 일부 시스템을 제외하고는 전부 자바로 작성되어 있습니다. Node.js로 시작하여 PHP를 거쳐 지금의 자바 기반의 시스템으로 구성하게 되었습니다. 다양한 사람들이 개발을 해오면서 각자 가장 잘할 수 있고, 빠르게 구현할 수 있는 언어로 개발되어 가다 현재의 자바로 통일되어 구성되게 되었습니다.
- **Spring** : API서버는 HTTP 기반의 REST API를 이용해 컨텐츠를 전달하고 있으며 스프링 프레임워크를 이용해 개발되었습니다. 이외에도 일부 분석에 스프링 배치를 사용하고 스프링을 편리하게 사용할 수 있게해주는 스프링 부트도 이용하고 있습니다.
- **gRPC** : 분산되어있는 서버들끼리 이기종 언어간 통신을 하기 위해서 Protocol Buffers 기반의 gRPC를 이용하고 있으며 서버들의 모니터링하는 서버와 에이전트들 사이의 통신 목적으로 사용합니다.
- **Flume** : 분산된 서버들에서 로그를 수집하는 역할을 합니다. 수집된 로그는 파일로 저장하며 실시간으로 볼수 있도록 엘라스틱서치에 같이 저장하고 있습니다. SDK에서 전송되는 로그 또한 웹서버의 엑세스 로그를 플럼 에이전트가 수집하는 방식으로 비동기로 처리하고 있습니다.
- **ElasticSearch** : 수집된 로그들을 실시간으로 확인하기 위해서 사용되며 Kibana를 이용해 시각화하고 있습니다.
- [**Angular.js**](https://www.angularjs.org/) : CMS의 프론트엔드는 [Angular.js](https://www.angularjs.org/) + [Bootstrap](http://getbootstrap.com/)을 이용해 개발되었으며,[Bower](http://bower.io/)를 이용한 라이브러리 관리, [Grunt](http://gruntjs.com/)를 이용한 빌드 관리를 하고 있습니다.

## 소프트웨어 개발/운영

- **Git** : 소스코드는 git로 관리하며 Git-Flow를 이용한 브랜치 정책을 수립하여 가져가고 있고 저장소로는 [깃허브](https://github.com/)를 이용합니다.

- **Quality Practice** : QA단계에서 제품을 테스트하기 전 개발자들은 QA 프로세스에 맞게 다음 3가지 기준으로 소스 코드의 품질을 관리합니다.

  - **코딩 컨벤션** : 사운들리 내부 코딩 컨벤션에 맞게 개발되었는지 확인합니다. [Checkstyle](http://checkstyle.sourceforge.net/)의 규칙을 정의 및 자동화합니다.
  - **테스트 코드** : 단위 테스트 코드를 작성하며 테스트 결과는 모두 통과되어야 합니다.
  - **테스트 커버리지** : 단위 테스트 코드가 작성된 커버리지를 계산하며 현재 60%를 목표로 진행하고 있습니다.

- **젠킨스** : 소스코드 저장소에 변동이 일어나면 젠킨스가 소스코드를 빌드하고 위에서 언급한 세가지에 대한 리포트를 작성합니다.

- **소나큐브** : 무료 오픈소스로 코드 정적 분석을 해주며 및 QA 리포트를 같이 볼 수 있습니다.

- **슬랙** : 인력이 적은 저희 팀도 슬랙을 적극적으로 개발/운영에서 사용하고 있습니다.

  - **팀 커뮤니케이션** : 팀원들 간의 의사사통을 위한 주요 수단으로 모든 팀원이 함께 사용하고 있습니다.
  - **분석 리포트** : 젠킨스나 배치를 통해 분석된 데이터들은 분석이 끝난 지표들은 슬랙으로 결과를 전송하여 모든 팀원이 볼 수 있도록 공유하고 있습니다.
  - **서버 모니터링** : 서버들의 이상 징후 감지나 배치 오류등을 슬랙을 통해 담당자에게 전송하여 조치할 수 있도록 합니다.

- **애플리케이션 및 서버 모니터링** : 애플리케이션의 모니터링은 Naver에서 오픈소스로 공개한 [핀포인트](https://github.com/naver/pinpoint)를 사용하고 있고, 서버 상태 모니터링을 위해 자체 개발한 모니터링 시스템을 사용하고 있습니다. 모니터링 데이터 수집을 하는 에이전트와 전체 시스템의 데이터를 관장 하는 서버간에는 gRPC를 이용하여 상태 체크를 합니다. 서버의 상태에 문제가 있을 때에는 slack을 통해 담당자들에게 알람을 주도록 시스템 설계를 하였습니다.

## 개발 문화

- 개발자들은 각각 개발을 할때 정해진 정책에 맞춰 브랜치를 만들어 개발합니다.
- 각각 개발된 소스들은 저장소인 깃허브에 푸시된 후 깃허브의 댓글 기능을 이용하거나 오프라인을 통해 코드 리뷰를 진행합니다.
- 리뷰가 끝난 후 합쳐진 소스는 QP 활동을 통해 분석이 됩니다.
- **빌드가 실패할 경우 커피를 사야합니다 ^^ (커피를 얻어 먹으려는 것이 아닌 소스코드를 푸시하기 전 잘 확인하자는 취지입니다)**

## AWS

- **EC2** : 사운들리의 대부분의 구성 요소인 API서버와 로그 수집, 분석 서버, 엘라스틱서치, 플럼, CMS등이 모두 EC2에 구축되어 있습니다.
- **RDS** : 컨텐츠의 주 저장소로 데이터베이스 관리의 용이성을 고려하여 RDS의 Multi-AZ에 배포하여 Active-Standby로 구성되어 있으며 이 데이터들은 레디스와 로컬 캐시를 이용하여 API서버에서 활용하고 있습니다.
- **S3** : 컨텐츠에 포함된 각종 정적 데이터들이 저장되며 수집된 로그들도 저장하여 보관됩니다.
- **EMR** : 로그 수집서버를 통해 S3에 저장된 로그들은 EMR을 이용해서 분석됩니다.
- **Beanstalk** : 개발 서버의 배포에 사용됩니다. 최근 [IntelliJ](https://plugins.jetbrains.com/plugin/7274?pr=idea)의 플러그인이 업데이트 되면서 IntelliJ 15버전을 지원하게 되므로써 로컬에서 개발하고 개발 서버에 배포까지 편리하게 하고 있습니다.
- **VPC** : 인터넷이 필요 없는 서버들은 VPC 내부 private-zone에 배포 및 ELB를 통해 외부에서 접근하도록 구성되어 있습니다.

{% include image.html subdir=page.subdir name='soundlly-aws-blueprint.png' caption='그림 2. AWS 배포 구성도' %}

이상으로 사운들리에서 사용하고 있는 백엔드 소프트웨어들을 소개해 보았습니다. 적은 인력으로 빠르게 사업을 진행하는 스타트업에서는 비즈니스에 집중할 수 있도록 도와주는 다양한 툴이나 오픈소스를 이용하여 많은 도움을 받을 수 있는 것 같습니다. 또한 코드를 잘 작성하여 에러를 줄이는 것도 필요하지만 여유가 많지 않으면 최소한 제품의 에러에 빠르게 대응할 수 있도록 하는 방법도 필요한 것 같습니다.
