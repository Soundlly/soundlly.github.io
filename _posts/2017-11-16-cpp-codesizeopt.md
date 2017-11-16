---
layout: post
title: C++ On Mobile - 코드 사이즈 최적화
description: 모바일 SDK에 적용되는 C++ Native 코드 사이즈 최적화에 대해서 이야기 합니다.
author: jwkang
subdir: cpp-codesizeopt
comment: true
---
모바일앱 SDK에 적용되는 C++ Native 코드 사이즈 최적화에 대해서 이야기 합니다.

그 어떤 사용자도 무거운 (사이즈가 큰) 모바일 앱을 좋아하지 않습니다. 
앱 다운로드에 데이터가 많이 소모되고, 폰안에서 많은 용량을 차지하기 때문입니다. 같은 맥락이 SDK에도 적용됩니다. 앱도 무거운 SDK 탑재를 선호 하지 않습니다.
따라서 SDK를 항상 날씬하게 유지시키는 것이 ~~코드트레이너~~개발자들의 하나의 목표가 됩니다. 

본 포스트에서는 사운들리  [**BitSound SDK**](http://bitsound.io/)에 들어가는 날씬한 C++ 모듈개발을 위한 저의 코딩 노하우를 공유합니다. 
슬림한 모바일 SDK 유지하기 위한 노력을 매 릴리즈에서 하고 있습니다.
제 몸무게도 릴리즈마다 같이 빠지는거 같아요.~~(대표님 보세요)~~ 
임베디드 C++ 과 시스템 프로그래밍의 기본적인 내용이지만 대학원에서 MATLAB/파이썬만 사용하다가
product레벨의 C++코딩을 처음으로 하게된 신입 개발자들에게는 크게 도움이 될것이라고 믿습니다.

### 일단 컴파일러 옵션를 이용하자 이건 공짜다!
- 일단 컴파일러 최적화 옵션을 이용하는 것은 무족건 해야합니다. 이건 **공짜**이기 때문이죠. :-D 
- 경우에 따라서 다르지만 컴파일러 옵션으로 대략 30% 정도의 사이즈 절감이 가능합니다. 
- 안드로이드 NDK 컴파일러의 경우 **릴리즈 빌드**를 적용해야 겠죠! ([안드로이드 ndk 컴파일 옵션 참조](https://developer.android.com/ndk/guides/ndk-build.html?hl=ko))
- 모바일은 아니지만 g++ 컴파일러인 경우 아래와 같이 최적화 옵션이 있습니다. **-O2를 사용하는게 보통**이고 임베디드 시스템에서는 -O5를 사용하기도 하나 요즘 임베디드 시스템도 메모리 사이즈가 많이 개선되어 많이 안쓰는거 같습니다.
   - 1) -O0 옵션 : 최적화를 수행하지 않는다.
   - 2) -O1 옵션 : -O0보다는 조금 낫다. 
   - 3) -O2 옵션 : 가장 많이 사용하는 옵션! 일반 응용 프로그램이나 커널을 컴파일 할 때 사용 (거의 대부분의 최적화를 수행한다.)
   - 4) -O3 옵션 : 가장 높은 레벨의 최적화 모든 함수를 인라인 함수와 같이 취급 (너무나 많은 소스의 변경이 가해지기 때문에 왜곡이 발생할 위험이 있음)        
   - 5) -O5 옵션 : 사이즈 최적화를 실행 (공간이 협소한 곳에서 사용 - 임베디드 시스템)
- 저의 회사의 경우 사실 SDK개발자이신 pjhjohn님께서 알아서 해주시기 때문에 제가 실제로 신경쓰는 부분은 아닙니다. :-) (올웨이즈 감솨 pjhjohn!)

### 외부 라이브러리는 네버!
- 코드를 개발하다보면 외부 라이브러리를 사용해야하는 경우가 있습니다. 하지만 외부 라이브러리를 그대로 가져와서 #include "xxxx.h"하고 사용하면  빌드하신 후에 SDK가 **뚱땡이**가 되어 있는 것을 확인이 가능하십니다. 
- 따라서 모바일 또는 임베디드 플랫폼에서 외부 라이브러리나 표준 라이브러리를 그대로 include해서 사용하는 것이 권장되지 않습니다. 최대한 **지원하는 표준 라이브러리만 사용해서 직접 구현**을 하는 것이 바람직합니다.
- 안드로이드 NDK 빌드시에 android.mk파일에서 지원하는 표준 라이브러리를 정의합니다. ([Android NDL C++ library support 참조](https://developer.android.com/ndk/guides/cpp-support.html))
- 위 옵션에서 아래로 갈수록 빌드시 코드가 무거우지게 되는 것이죠.  슬림한 사이즈를 유지하기 위해서 가급적으로 libstdc++라이브러리을 선택하고 필요한 기능은 직접 구현해서 사용하는 것이 좋습니다. 

### #ifdef/#ifndef문 통한 불필요 코드는 빌드 피하기
- 실력있는 개발자들은 코딩 과정에서 디버깅을 위한 로그를 중간중간에 삽입합니다. 자신이 구현한 코드가 정상동작을 하는지 확인하고 또는 이레귤러가 발생했을때 상황파악이 빠르게 하기 위함입니다.
- 하지만 이러한 코드들은 디버깅/테스트에서만 사용되는 redundant 코드입니다. 그대로 방치하면 괜히 사이즈만 차지하게 되죠.
- 이런 경우 #ifdef/#ifndef 전처리문을 사용하면 릴리즈 시에 필요한 코드와 테스트시에 필요한 코드를 구분해서 빌드할 수 있습니다.
- 여기서 주의 할 점은 #ifdef/#ifndef문을 너무 남발하는 경우 코드 유연성과 가독성이 떨어지는 코드를 결과한다는 것입니다.
- 저는 **플랫폼별 빌드 + 테스트 빌드 (예를들면 iOS / Android /Debug)  정도만 구분해서 사용하는 것이 현명**하다는 생각합니다.


### 변수 선언시 워드 사이즈를 유지 해야
- 임베디드/모바일 플랫폼을 위한 C++개발에서는 적용될 시스템 환경에 대한 고려를 해야합니다. 먼저 CPU 워드 사이즈에 관련된 얘기를 해보죠.
- CPU는 연산처리을 위한 일정한 워드 사이즈를 가집니다. 여기서 워드란 CPU가 한번에 처리할수 있는 비트 수를 말합니다. 
- 효율적인 연산을 위해서는 변수크기가 워드사이즈 보다 크 경우를 피해야합니다. 그 이유는 그런 경우 연산처리를 위해서 부가적인 일을 CPU가 해야하기 때문입니다.
- 예를 들면 CPU 워드사이즈가 32비트인 경우 (즉 4byte) 8byte인 long이나 double형 연산을 하는 경우 32비트씩 두 번 연산을 하고 그 두 연산결과를 머지하는 또 다른 연산이 부가적으로 수행됩니다.
- **아직까지는 ARMv7-A를 지원해야하기 때문 하위호환성을 위해서 워드사이즈를 32비트보고 코딩을 하는 것이 좋은 선택**입니다. 따라서 double 보다는 float, long보다는 int를 사용하는 코딩이 바람직합니다.
- ARMv8-A에서는 64비트 워드사이즈를 지원합니다. 갤럭시 S6쯤음 부 ARMv8 아키텍쳐가 적용되는 스마트폰은 점점 많아지기 시작됐습니다. 앞으로의 변화가 예상됩니다. 저는 pjhjohn님이 신호를 주시면 그때부터는 64비트 워드사이즈 코딩을 하려고 하고 있어요!


### 인라인 함수의 사용을 줄여야 한다
- 항상 우리가 염두에 두어야 하는 것은 **코드 사이즈와 프로그램 실행시간은 trade-off 관계**에 있다는 것입니다. 이것은 인라인함수의 사용은 이러한 사실을 잘 뒷받침합니다.
- main함수에서 local함수를 호출하면 프로그램 제어포인터가 해당 코드가 있는 메모리영역로 점프합니다. 그리고 코드를 실행하고 main함수로 다시 리턴되는 것이죠. 이 "메모리 점프"는 사실상 프로그램 동작 측면에서만 보면 부수적인 동작입니다.
- 함수를 "inline" 키워드로 선언을 하면 local함수 코드가 main함수에 그대로 삽입되기 때문에 "메모리 점프"를 할 필요가 없습니다. 따라서 자주 사용되는 local 함수의 경우 "inline"으로 선언을 하면 효율적일 수 있습니다.
- 하지만 세상의 공짜는 없듯이 인라인함수의 사용은 함수코드를 main 함수에 직접 삽입하기 때문에 함수 호출횟수에 비례하는 코드사이즈의 
증가를 가져옵니다. 예를 들어서 같은 인라인 local 함수를 5번 호출하면  main함수는 같은 코드를 5번 삽입합니다.
- 코드 사이즈를 줄이기 위해서라면 인라인 함수의 사용은 다시한번 고려해야 할 것입니다.

### #define / const변수 사용 줄이자
- 프로그램이 실행되면 코드가 RAM에 올라갑니다. 그리고 프로그램 코드 수행을 위해서 메모리가 데이터/힙/스택 영역으로 나뉘어서 각 변수를 위한 공간이 할당됩니다.
( [[Beom's Blog](http://myblog.opendocs.co.kr/archives/1301
)] 그림 참조) 
- 여기서 
  1) 데이터 영역은 전역변수 정적변수가 올라가는 곳. 프로그램 시작시 할당 되고 끝날때 해제됩니다. 
  2) 힙 영역은 동적변수가 올라가는 곳. 메모리 주소값에 의해서 만 참조되고 사용됩니다. 
  3) 스택 영역은 지역변수와 함수매개변수가 주로 올라가는 임시적으로 머무는 메모리 공간입니다.
- 전처리기 **#define /const으로 선언된 변수들은 코드 영역**으로 들어갑니다. 자연스럽게 코드사이즈의 증가를 가져옵니다.

### 최적화를 위해서 일반화을 잊어라
- 일반화와 최적화는 정반대의 개념이다. 특정한 플랫폼 또는 환경에 최적화 시키기 위해서는 일반화 개념을 어느정도 포기해야합니다.
- 일반화 프로그래밍을 위한 대표적 개념이 "템플릿"입니다. 
- **템플릿의 사용은 코드 사이즈 증가를 비용으로** 합니다. 그 이유는 컴파일러가 obj파일 생성시에 템플릿이 사용된 종류의 수만큼 코드를 찍어내기 때문입니다.
- 이것은 "[코드 부풀림 현상](http://yesarang.tistory.com/268)" 이라고 합니다.
- 날씬한 코드를 위해서라면 장인정신을 가지고 하나하나 커스터마이즈 된 클래스 또는 함수를 설계하는 것이 좋습니다. (sometime 노가다 ㅠ)

### 굿바이! 예외처리
- Try, Throw, Catch문으로 구현되는 예외처리입니다. 무엇보다 로직에 구멍이나서 메모리 누수을 쉽게 야기 할수 있습니다.

### 어셈...블리언어를 사용할수 있다면야...
- 이건 저에게도 해당되지 않은 사항이지만ㅠ 들은 바에 의하면 숙련된 개발자가 직접 어셈블리 언어로 코딩하면 보다 효율적인 코드를 작성하는 것이 가능하다고 합니다. 
- 하지만 C/C++ 컴파일러가 매우 발전한 현재시점에서 어셈블리 코딩에 드는 시간과 노력에 견주어 봤을때 생산적인 방법인지는 잘 생각해 봐야 할 부분인거 같습니다.

{% include image.html subdir=page.subdir name='myster.png' caption='임베디드 C++는 ~~노가다~~ 장인정신 이다. by jwkang' %}


## 마치며
- 제가 정리하고 보니 결국 모바일 / 임베디드 C++ 최적화는 결국 ~~노가다~~ 장인정신인거 같네요 ㅠㅠ.
- 다음 C++ on Mobile 2탄 에서는 실행속도 최적화에 대해서 얘기합니다! 기대해주세요!

기계들이 스스로 노래하고 흥얼거리는 세상을 위해!

## 참고 링크 
- [C언어의 메모리 구조] [http://dsnight.tistory.com/50](http://dsnight.tistory.com/50)
- [Beom's Blog] [http://myblog.opendocs.co.kr/archives/1301](http://myblog.opendocs.co.kr/archives/1301)
- [Block Busting Blog] [http://sfixer.tistory.com/entry/%EB%A9%94%EB%AA%A8%EB%A6%AC-%EC%98%81%EC%97%ADcode-data-stack-heap](http://sfixer.tistory.com/entry/%EB%A9%94%EB%AA%A8%EB%A6%AC-%EC%98%81%EC%97%ADcode-data-stack-heap)
- [템플릿 코드부풀림 현상] [http://yesarang.tistory.com/268](http://yesarang.tistory.com/268)
- [C++ 예외처리] [http://yesarang.tistory.com/372](http://yesarang.tistory.com/372) 