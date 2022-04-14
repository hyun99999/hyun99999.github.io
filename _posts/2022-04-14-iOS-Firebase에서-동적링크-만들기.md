---
title:  "iOS) Firebase 에서 동적링크 만들기"
categories:
- iOS

date:   2022-04-14  22:31:00 +0900
author_profile: false
---
### 내용

- Firebase 에서 동적링크를 만들면서 맞닥뜨린 문제와 해결을 풀어보겠습니다.

### 🔥 Firebase 동적 링크

- 동적 링크란, 접속한 플랫폼이 어디인가에 따라 적절한 반응을 하도록 하는 것이다.
- 사용자가 특정 앱 화면 링크를 공유하거나 해당 링크를 통해서 앱을 설치했을 때 추천인 코드가 입력되는 프로모션과 같은 혜택을 주는데 사용된다.

[Firebase 동적 링크 | Firebase Documentation](https://firebase.google.com/docs/dynamic-links?hl=ko)](https://firebase.google.com/docs/dynamic-links?hl=ko

### 🔥 **Firebase 및 동적 링크 SDK 설정**

- iOS에서 동적 링크를 통해서 접근한 경우를 수신해서 대응할 수 있는 방법이다.

[iOS에서 동적 링크 수신 | Firebase Documentation](https://firebase.google.com/docs/dynamic-links/ios/receive?hl=ko)

### 🔥 **동적 링크 애널리틱스 데이터 보기**

<img src="https://user-images.githubusercontent.com/69136340/163401931-4084b957-0e3d-4f77-b922-d81a7c30ce4f.png" width ="500">

[동적 링크 애널리틱스 데이터 보기 | Firebase Documentation](https://firebase.google.com/docs/dynamic-links/analytics?hl=ko)

### 🔥 동적 링크 만들기

Firebase 에서 동적 링크를 만들기 위해서 아래 포스팅을 참고해서 진행했다.

[[iOS - swift] 4. DeepLink (딥 링크) - Dynamic Link (다이나믹 링크) 사용 방법 (Firebase, 공유하기 기능)](https://ios-development.tistory.com/726)

[iOS - 앱에서 Firebase Dynamic Link 생성하고 수신하기](https://medium.com/hcleedev/ios-%EC%95%B1%EC%97%90%EC%84%9C-firebase-dynamic-link-%EC%83%9D%EC%84%B1%ED%95%98%EA%B3%A0-%EC%88%98%EC%8B%A0%ED%95%98%EA%B8%B0-e357e95343c9)

동적링크의 기능을 살려서

- 앱이 미설치되어 있다면 앱스토어로 연결.
- 데스크탑 환경에서 링크를 열게되면 미리 만들어둔 랜딩페이지로 연결.

하도록 해보겠다.

### 구상

- `isi` 파라미터를 통해서 앱이 미설치되어있다면 앱스토어로 연결하려 했다. **이는 Firebase 동적 링크 만들때 설정을 통해서 자동으로 들어간다.**

<img width="600" alt="2" src="https://user-images.githubusercontent.com/69136340/163402134-52f694ba-c100-4b9f-bf14-d6f1f594b7cc.png">

- Firebase 의 문서를 읽고 랜딩 페이지 연결은 `ofl` 을 통해서 진행하려했다.

<img src="https://user-images.githubusercontent.com/69136340/163402169-9f404644-7b68-4911-ac1c-a6f6f55d6c14.png" width ="600">

[Manually constructing a Dynamic Link URL | Firebase Documentation](https://firebase.google.com/docs/dynamic-links/create-manually#ddl_parameters)

- 동적 링크 설정

딥 링크 URL 에 ofl 파라미터를 활용해서 집어넣었다.*(이 부분은 잘못된 방법이고, 추후에 해결한 방법이 나온다.)*

`Apple용  링크 동작 정의` 에서 App 프로젝트를 등록하면 해당 프로젝트에 등록된 Apple Store ID 가 `isi` 파라미터에 추가된다.

<img src="https://user-images.githubusercontent.com/69136340/163402527-be11dd1a-0fb4-4a5c-8b8d-85e0d6943d7f.png" width ="500">

- 위와 같이 입력하면 다음과 같이 정보를 가진다. (잘못된 방법이었기 때문에 수동으로 입력한 ofl 앞뒤로 `/` 문자가 대체됨을 볼 수 있다.)

<img src="https://user-images.githubusercontent.com/69136340/163403021-aed98e99-31c4-4b1b-b541-0cafc038fdb3.png" width ="500">

### 문제 인식

- 에러가 나왔다. 웹에서 URL 을 열었을때 다음과 같은 화면이 나왔다.

<img src="https://user-images.githubusercontent.com/69136340/163403069-fc246e5d-35a0-44d0-b421-0653563f31d2.png" width ="500">

- Firebase 에서 제공하는 링크 미리보기(디버그) 에서 URL pattern 에 문제가 있음을 알게되었다.(맨위 에러)

<img src="https://user-images.githubusercontent.com/69136340/163403175-331f1ad0-ae83-4bc6-81c1-79118d08afd4.png" width ="500">

### 해결

- 앞서 읽어본 문서에서 link 부분을 좀 더 자세히 읽어보았다.

<img src="https://user-images.githubusercontent.com/69136340/163403268-c805523d-f3a3-4b00-a5f1-07d1d217df5b.png" width ="600">

[Manually constructing a Dynamic Link URL | Firebase Documentation](https://firebase.google.com/docs/dynamic-links/create-manually#ddl_parameters)


> 💡 link 에는 올바른 형식의 URL 이어야하고, 올바르게 URL 인코딩되어야 하고, HTTP 또는 HTTPS를 사용해야 하며, 다른 동적 링크가 될 수 없습니다.


라고 명시하고 있었다.

더 읽어보니

> 💡 사용자가 데스크탑 웹 브라우저에서 동적 링크를 열면 해당 URL 이 로드된다(ofl 매개변수가 지정되지 않은 경우)

라고 했다.

앞서 놓쳤는데 `동적 링크 설정` 설명에 보면 딥링크 URL 자체가 데스크톱에서는 해당 URL 이동한다고 했었다.
<img src="https://user-images.githubusercontent.com/69136340/163403403-ddbb67f1-d3b8-4475-9480-4beac357843a.png" width="500">

- 위와 같이 설정하니 다음과 같은 정보를 가지게 되었다.

<img src="https://user-images.githubusercontent.com/69136340/163403532-bb7ac087-ed7a-4883-b6a4-4f451913875b.png" width ="550">

아까 입력한 딥 링크 URL 이 `link` 파라미터에 들어가있었고, 이를 통해서 웹에서 랜딩페이지를 열 수 있도록 하고 있었다.

`ofl` 파라미터가 지정되어있지 않기때문에 앞서 문서에서 언급된 대로 `link` 파라미터의 URL 이 열리게 되었다.

(추가적으로 st, si 를 사용해서 소셜 공유 미리보기도 만들어보았다.)

[소셜 메타데이터로 링크 미리보기 생성 | Firebase Documentation](https://firebase.google.com/docs/dynamic-links/link-previews?hl=ko)

<img src="https://user-images.githubusercontent.com/69136340/163403804-12a620a8-4eb2-4c5e-b9b8-fd80d72771ba.MP4" width ="250">

옵션을 통해서 생략도 가능했다.

<img src="https://user-images.githubusercontent.com/69136340/163403867-67ccfa52-5925-4d82-860c-330d169c0068.MP4" width ="250">

소셜 공유 미리보기도 잘 나왔다.

<img src="https://user-images.githubusercontent.com/69136340/163403945-91835695-82de-4c42-a957-28c9adbac71b.png" width ="300">

> 💡 결과적으로 파이어베이스에서 동적 링크를 만드는 과정에서 옵션을 통해서 딥 링크와 여러가지 파라미터를 가진 동적링크가 자연스럽게 만들어졌다.

이러자 링크 미리보기(디버그) 맨위 오류가 사라졌다. 두번째는 안드로이드 오류이고, 세번째 오류도 해결해보자.

아래의 문서를 확인하니 보안상 `https://teamspark.page.link` 라는 URL의 패턴 개선이 필요하다고 한다.

[Allow specific URL patterns](https://support.google.com/firebase/answer/9021429)

- 하지만 이처럼 무료 도메인을 사용했기 때문에 어쩔 수 없었다.

<img src="https://user-images.githubusercontent.com/69136340/163404086-64998062-faf8-432b-968b-3b52757be2ee.png" width ="500">

### 정리

Firebase 에서 동적링크를 만들어서

- 앱이 설치되었다면 앱을 연다.
- 앱이 미설치라면 앱스토어를 연다.
- 데스크탑 환경에서는 설정해둔 웹페이지를 연다.

와 같이 설정해보았다.

앱이 설치되었다면 앱을 열기위해서는 iOS 에서는 Universal Link 를 추가해야 한다. 이 방법도 다음의 포스팅으로 알아보자.

[iOS) 유니버셜 링크 적용하기](https://gyuios.tistory.com/162)
