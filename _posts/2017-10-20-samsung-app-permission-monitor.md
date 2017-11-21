---
layout: post
title: 삼성 '앱 권한 모니터' 파헤치기
description: 정식 출시된 삼성 갤럭시 노트8부터 탑재되는 삼성 안드로이드 7.1.1 OS의 신규 기능인 '앱 권한 모니터'의 기능과 대응 방안을 살펴 봅니다.
author: pjhjohn
subdir: samsung-app-permission-monitor
thumbnail: thumbnail.png
comments: true
---

2017년 9월 15일 삼성의 갤럭시 노트8이 공식 출시되었습니다. 사전 예약 판매량만 **약 85만 대**로 집계될 정도로 큰 인기를 얻어 시장 점유율이 가파르게 상승하고 있는데요, 삼성 스마트폰으로는 최초로 Android 7.1.1 OS를 탑재한 기기이기도 합니다(~~보온병 7~~ 갤럭시 노트 FE도  7.1.1 으로 업데이트 되었네요). 정식 출시된 삼성 7.1.1 OS의 신규 기능인 **앱 권한 모니터**는 뭘까요?

## 앱 권한 모니터란?

안드로이드 개발자라면 악명높은 삼성의 [**스마트매니저**](https://news.samsung.com/kr/%ED%98%84%EC%9E%AC-%EB%82%B4-%EC%8A%A4%EB%A7%88%ED%8A%B8%ED%8F%B0%EC%9D%98-%EC%83%81%ED%83%9C%EB%8A%94-%EC%8A%A4%EB%A7%88%ED%8A%B8%ED%8F%B0%EC%9D%84-%EB%8A%98-%EC%83%88%EA%B2%83)를 잘 아실 텐데요, 새로 탑재된 삼성의 Android 7.1.1 OS에는 **앱 권한 모니터**라는 스마트매니저의 악명을 나눠 가지게 될 존재가 나타났습니다. **앱 권한 모니터**는 앱 별로 사용되는 권한을 시스템 레벨에서 추적하고 사용자에게 주기적으로 알려줌으로써 스마트폰 사용자의 개인정보 및 사생활 보호를 더욱 강화해주는 기능입니다.

최근 Android Oreo가 출시됨과 동시에 API26 이상의 기기에서는 OS레벨의 [백그라운드 실행 제한](https://developer.android.com/about/versions/oreo/background.html), [백그라운드 위치 제한](https://developer.android.com/about/versions/oreo/background-location-limits.html) 등 사용자가 알지 못 하는 사이에 실행되는 백그라운드 서비스를 제한함으로써 배터리 소모 감소와 더불어 사용자가 스마트폰을 통제하고 있다는 경험을 강화하려 합니다. 무거운 서비스를 제한 없이 돌리려면 Foreground Service로 돌려서 사용자가 알게끔 하거나 사용자에게 백그라운드 기능의 필요성을 어필하도록 유도하는 것 처럼요. 따라서 삼성의 **앱 권한 모니터**는 이러한 맥락을 공유하고 있다고 볼 수 있습니다.

{% include image.html subdir=page.subdir name='app-permission-monitor.jpg' caption='삼성 앱 권한 모니터의 각 화면들' %}

Galaxy Note8 기기에서 **설정 → 잠금화면 및 보안**에 들어가면 **앱 권한 모니터**가 추가된 것을 볼 수 있습니다. 항목 자체의 스위치를 이용해 해당 기능을 ON/OFF할 수 있으며, 상세 기능과 조건은 다음과 같습니다:

- **앱 권한 모니터 ON/OFF** : 앱을 사용하지 않을 때, 특정 권한이 사용되면 1분 간격으로 사용자에게 알림
- **앱 별 권한 모니터 ON/OFF** : **앱 권한 모니터 세부 설정**에 들어가면 앱 별로 권한 모니터링 ON/OFF 가능
- **앱의 권한 별 모니터 ON/OFF** : **앱 권한 모니터 → 앱 선택** 설정에서는 보유 권한 별로 모니터링 ON/OFF 가능
- **권한 사용 기록** : 위의 동작 기록을 1개월간 날짜별 - 앱별 - 시간별 권한 사용 내역을 보여줌

기능만 보면 사용자가 앱을 사용하지 않는 상태에서 특정 권한이 사용되는 것을 막는 좋은 기능으로 보입니다. 안드로이드 앱 시장에는 다양한 권한 만큼 그 권한을 악용하려는 앱들도 많기 때문에 이러한 기능이 있다면 사용자가 스마트폰을 더 안전하게 통제할 수 있겠지요. 예를 들어 앱이 **몰래 SMS를 읽는 것 / 내 위치를 추적하는 것 / 동영상을 촬영하는 것 등을 방지**하진 못하더라도 사용하고 있다는 사실을 사용자에게 알려주어 확인해볼 수 있도록요. 그런데, 문제는 스마트매니저의 경우와 마찬가지로 권한을 악용하지 않는 일반 개발자가 대응할 수 없도록 만들었고, **기본 옵션이 활성화**된 것으로 보인다는 점입니다. 철저히 사용자 편의만을 고려한 시스템 제약사항이죠. 그래서 사용자가 편하게 될 지도 사실 모르겠네요. 현재 왕좌를 차지하고 있는 스마트매니저의 **앱 절전 기능**과 비교해보면 다음과 같습니다:

#### 스마트매니저의 앱 절전

- 앱을 3일이상 사용하지 않으면 앱으로부터 **Push & Notification 이 전혀 오지 않도록 패키지 비활성화**
- **기본적으로 활성화**되어 있음
- **자사 및 협력사의 편익을 기준으로 화이트리스트를 선정**하여 운영함
- 일부 화이트리스트된 앱은 사용자가 원해도 앱 절전을 끌 수 없음 ([링크](http://gun0912.tistory.com/64))

#### 앱 권한 모니터

- 앱을 사용하지 않고 있을 때 권한이 사용되는 경우 1분 간격으로 사용자에게 Notification으로 알리고 1개월간 기록 저장
- **SMS / 마이크 / 카메라** 권한은 설치되는 순간부터 활성화되어 있음
- **Android Marshmallow 이상에서 앱이 사용자로부터 권한을 명시적으로 획득함과 무관하게 동작함**
- **자사 / 선탑재 앱은 전부 화이트리스트**되어 있음
- 일부 화이트리스트된 앱은 사용자가 원해도 앱 절전을 끌 수 없음 → **아예 리스트에서 보이지 않음**
- Android Oreo의 변화에서 구글이 수 개월간 개발자들에게 변화를 유도한 것과 달리 **갑툭튀**
- Galaxy S8 / Galaxy S8+ 등의 기종도 조만간 **업데이트를 통해 이슈가 더욱 확대될 것으로 예상**됨

이정도면 ~~왕위 계승 아니냐~~ 감이 오실거라 생각합니다. 사용자에게 열심히 안내해서 겨우 권한 얻었더니 이건 또 뭐...

{% include image.html subdir=page.subdir name='succeeding-you-father.jpeg' caption='왕위를 계승하는 중입니다, 아버지' %}

## 앱 권한 모니터 대응하기

그래도 뭐 어쩌겠습니까...전 삼성 직원이 아니니까요.  
스타트업 다니는 불효자가 할 수 있는 유일한 스킬인 ~~바둥바둥~~**대응**을 열심히 해 보기로 합니다.

{% include image.html subdir=page.subdir name='magikarp-gyarados.jpeg' caption='바둥바둥 하다보면 뭐라도 되겠죠' %}

### 시도 1 : "거 화이트리스트에 우리도 좀 넣어주쇼"

분명 화이트리스트는 있습니다. 왜냐면 **우리 삼성의 귀염둥이 빅스비**가 안보이거든요. 눈을 씻고 찾아봐도 없습니다. 스마트매니저 때를 회상하면 보이지만 않을 뿐 있을 겁니다. 시스템 앱이나, 제조사 앱이나, 선 탑재 앱이나... 이게 된다면 HAPPY END!! 아무것도 바꾸지 않아도 됩니다.

하지만 다시 생각해 봅시다. **우린 스타트업 다니는 불효자**입니다.  
곱게 문의 하나 밀어넣고 다음 방안을 찾아 봅니다.

#### 곱게 문의 하나 밀어넣을 곳
- [Google Play :: Device Mainternance](https://play.google.com/store/apps/details?id=com.samsung.android.lool) → Developer Email : [smart.mngr@samsung.com](mailto:smart.mngr@samsung.com)
- [삼성 계정 :: 개인정보 처리방침](https://account.samsung.com/membership/pp) → 10. 개인정보 보호책임자 및 고객서비스 담당부서 등 안내
  - 고객서비스 담당부서: **무선사업부 서비스PM그룹**
  - 전화 : **1588-4730**
  - 문의처 : **[https://help.content.samsung.com](https://help.content.samsung.com)**

### 시도 2 : "일단 적을 알아보자"

> 知己知彼 百戰百勝(지피지기 백전백승) : 나를 알고 적을 알면 백 번 싸워도 백 번 이긴다

**대응 1**에서 나(=스타트업 불효자)를 알았으니 **이제 적만 알면 다 이길 수 있다는 마음가짐이라도..** 가져 봅니다. 정 안되면 우리는 **앱 권한 모니터**가 깔려있는 사용자들에게 안내 혹은 고객 대응이라도 해야 합니다. **사용자가 공지사항, 팝업, 푸시, 알림, 튜토리얼을 싫어한다**는건 (슬프지만)상식 중의 상식이므로, 클릭 수를 어디까지 줄여줄 수 있는지도 파악해야 합니다. 우리가 가진 단서는 어쩌다 노출되는 로그 뿐이므로, AndroidStudio의 LogCat창을 열고 우측 드롭다운에서 **No Filter**를 누릅니다. LogCat 창을 최대화하면서 동공도 같이 최대화됩니다. 다행인 점은 1번에 한 번씩 Notification을 날려주니 몇 분만 집중하면 됩니다. 이렇게 알아낸 조건들은 다음과 같습니다:

- 알람을 보내는 앱 : `SecurityLogAgent` `(com.samsung.android.securitylogagent)`

- `PermissionMonitor`가 권한을 조사하고, `AppPermissionMonitor`가 알람을 보냄

- 모든 권한 사용이 발생할 때 마다 권한을 조사함

- 권한 사용이 발생할 때, 해당 앱이 숨겨진(백그라운드) 상태가 아니라면 무시함
```
I/PermissionMonitor: App is not in hidden status (background/etc). Ignore
```

- 권한 사용이 발생할 때, 해당 앱이 `숨겨진(백그라운드)` 상태라면 푸시를 보냄
```
D/AppPermissionMonitor:APMApplist: BCreceiver, package_name: io.bitsound.apitestapp, status: 2, cur_time: 2017-10-11 14:38:00
D/AppPermissionMonitor:APMApplist: BCreceiver, detectedPermission: MICROPHONE
D/AppPermissionMonitor:APMApplist: sendToHistory, pkg: io.bitsound.apitestapp, perm: MICROPHONE 2017-10-11 14:38:00
D/AppPermissionMonitor:APMApplist: BCreceiver, status_of_detected_app: 66
D/AppPermissionMonitor:APMApplist: getnotimessage, send noti Notimessage: APITEST 앱에서 마이크 권한을 사용 중입니다.
```

- 이미 푸시를 보냈다면, 이후 1분간 무시함
```
I/PermissionMonitor: isCalledPkg is in effect. Ignore.
```

- 화이트리스트 존재 확인
```
I/PermissionMonitor: App is whitelisted. Ignore
```

- 포그라운드 조건
  - 앱 실행 중
  - 락스크린 앱의 경우, 화면 켜서 락스크린이 보이는 상태
- 백그라운드 조건
  - 기기 화면 꺼짐
  - 앱 미실행, 포그라운드 서비스 실행 중
  - 기타 포그라운드 조건이 아닌 경우

### 시도 3 : "찾아서 지워라"

우선, 알람을 보내는 `SecurityLogAgent` 앱을 비활성화하도록 유도할 수 있다면 가장 좋습니다. 안내 페이지 혹은 앱 내 설정 창에서 곧바로 해당 앱의 설정 페이지를 열어 비활성화를 유도할 수 있겠네요.

{% include image.html subdir=page.subdir name='i-will-find-you.jpg' caption='그리고 지울 것이다' %}

앱 목록에서 보이지 않는 앱이지만, 이미 패키지명을 알아 냈으므로 다음 예제와 같이 간단한 Intent를 통해 설정 화면으로 이동할 수 있습니다.

#### Java
```java
Uri destination = Uri.parse("package:com.samsung.android.securitylogagent");
Intent intent = new Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS, destination);
context.startActivity(intent);
```

#### Kotlin
```kotlin
val destination = Uri.parse("package:com.samsung.android.securitylogagent")
val intent = Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS, destination)
context.startActivity(intent)
```

하지만 역시 들어갈 수 있어도 `사용 해제`나 `삭제`는 할 수 없도록 되어 있습니다. 다만 `강제 중지`를 누르면 Notification이 클리어되고 수분동안 기능이 정지되기는 합니다. 이는 단기적 해결책도, 장기적 해결책도 될 수 없기 때문에 채택하긴 어렵습니다.

### 시도 4 : "고객 서-비스"

지울 수 없다면 역시...**고객 서-비스** 뿐이겠죠 ㅠㅠ
사용자에게 가장 적은 클릭수로 **앱 권한 모니터** 설정창을 열어주는게 가장 좋으므로 눈을 로그에 밀착시킨 채로 앱 권한 모니터 Notification을 눌러 설정 창으로 이동해 보면 다음과 같은 로그를 확인할 수 있습니다

```
I/ActivityManager: START u0 {act=null typ=null flg=0x20008000 cmp=ComponentInfo{com.samsung.android.securitylogagent/com.samsung.android.apppermissionmonitor.APMActivity}} from uid 1000 on display 0
```

앱 권한 모니터의 패키지명과 액티비티 명을 알았으니, `ComponentName`을 이용해 수동으로 접근해 봅니다.

#### Java
```java
final String pkg = "com.samsung.android.securitylogagent";
final String cls = "com.samsung.android.apppermissionmonitor.APMActivity";
final Intent intent = new Intent();
intent.setComponent(new ComponentName(pkg, cls));
context.startActivity(intent);
```

#### Kotlin
```kotlin
val pkg = "com.samsung.android.securitylogagent"
val cls = "com.samsung.android.apppermissionmonitor.APMActivity"
val intent = Intent()
intent.component = ComponentName(pkg, cls)
context.startActivity(intent)
```

그리고 물론 보안 예외가 발생합니다. 위의 ActivityManager 로그를 보면 `uid 1000`으로부터 앱 실행이 이루어졌고, 아래의 SecurityException 로그를 보면 `uid 1000`으로부터 발생하지 않아 예외가 발생합니다.

```
java.lang.SecurityException: Permission Denial: starting Intent { cmp=com.samsung.android.securitylogagent/com.samsung.android.apppermissionmonitor.APMActivity launchParam=MultiScreenLaunchParams { mDisplayId=0 mBaseDisplayId=0 mFlags=0 } } from ProcessRecord{e6ea6a8 17962:io.pjhjohn.advertisementids/u0a214} (pid=17962, uid=10214) not exported from uid 1000
```

[찾아보니](https://stackoverflow.com/a/8685751) `uid 1000`은 **안드로이드 시스템**이라고 하니 이것도 일반적인 해결책이 될 수는 없을 것 같네요.  
이쯤되니 **대응 3**의 [android.provider.Settings.ACTION_APPLICATION_DETAILS_SETTINGS](https://developer.android.com/reference/android/provider/Settings.html#ACTION_APPLICATION_DETAILS_SETTINGS) 인텐트 액션에서 힌트를 얻어 고객 서비스(CS) 대응 시 **잠금화면 및 보안** 페이지까지라도 바로 진입할 수 있도록 하는 것이 최선이 아닐까 하는 생각이 듭니다.

#### Java
```java
context.startActivity(new Intent(Settings.ACTION_SECURITY_SETTINGS));
```

#### Kotlin
```kotlin
context.startActivity(Intent(Settings.ACTION_SECURITY_SETTINGS))
```

## 마치며

- Android Marshmallow의 명시적 앱 권한 요청과 삼성 스마트매니저
- Android Oreo의 백그라운드 실행 제한과 삼성 앱 권한 모니터

이번 이슈를 분석하고 대응책을 찾아 보면서 느낀 점은 **둘 모두 비슷한 시기에 발생한 OS 및 시스템 자체에서 사용자에게 권한을 선택할 수 있도록 발생한 변경사항이지만, 문제를 해결하고 개발자들과의 공감대와 해결 및 우회 방안을 제시하는 방법은 너무나도 다르다**는 점입니다.

Google은 충분한 기간동안 API를 선 공개하고 개발자들과의 소통을 지속적으로 개진하였지만, 삼성의 시스템 앱은 몇몇 이해 관계자를 화이트리스트에 포함시키고 일반 개발자들과의 소통 창구를 막아버려 굉장히 아쉽습니다. 최소한 **Google Play Services API처럼 삼성의 시스템 앱과 소통하여 현재 감시당하는 중인지 여부를 앱에서 알 수 있게 하여 사용자들에게 불편한 UX를 제공하지 않도록 앱 쪽에서 대응할 수 있게는 해주어야 하는게 아닌가** 싶은데요, 앞으로 일반 개발자들과의 커뮤니케이션 방법이 더 발전되었으면 좋겠습니다.

개인적으로는 안드로이드 내부 동작 및 사소한 로그도 찾아보게 되었던 터라 나름 즐겁게 분석을 해보는 신선한 경험이었습니다. 본 포스트를 토대로 더 좋은 해결 방안이 있다면 꼭 공유해주시길 부탁합니다! 긴 글 읽어주셔서 감사합니다.

## 참고자료 및 출처

- [박상권의 삽질 블로그 :: 삼성 '스마트매니저' 푸시알림 차단이슈 대응하기](http://gun0912.tistory.com/64)
- [Android Oreo :: 백그라운드 실행 제한](https://developer.android.com/about/versions/oreo/background.html)
- [Android Oreo :: 백그라운드 위치 제한](https://developer.android.com/about/versions/oreo/background-location-limits.html)
- [Android Developers :: android.provider.Settings](https://developer.android.com/reference/android/provider/Settings.html)
- [StackOverflow :: How many UIDs are possible in Android?](https://stackoverflow.com/a/8685751)
- [StackOverflow :: Go to My app's App Permission screen](https://stackoverflow.com/a/32899317)
- [Image :: 아서스 메네실](http://bbs.ruliweb.com/family/211/board/300075/read/22958983)
- [Image :: 잉어킹 & 갸라도스](http://game.dailyesports.com/view.php?ud=2017021001585932300)
- [Image :: 테이큰](https://twitter.com/byuljiggy/status/470793101993189377)
