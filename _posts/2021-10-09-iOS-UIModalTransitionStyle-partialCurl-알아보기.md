---
title:  "iOS) UIModalTransitionStyle.partialCurl 알아보기"
categories:
- iOS

date:   2021-10-09  16:08:00 +0900
author_profile: false
---
transition style 을 골라서 해보다가 `.partialCurl` 를 해보았다. 그런데 다음과 같이 오류가 발생했다.

![스크린샷 2021-10-05 오후 4.52.43.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/de3c28d0-092a-46d8-b5b5-da050fd9ac39/스크린샷_2021-10-05_오후_4.52.43.png)

fullscreen 이 아닌 곳에서 UIModalTransitionStylePartialCurl 을 present 하려한다는 에러 메시지였다.

그렇다면 `.partialCurl` 는 조건이 필요하다는 소리다. 시도해보니 에러 그대로 정말`UIModalPresentationStyle.fullscreen` 을 프로퍼티로 가지지 않는 모든 경우에 에러를 일으켰다.

이제는 진짜 `.partialCurl` 을 사용해보자.

<img src ="https://user-images.githubusercontent.com/69136340/136648058-4dfcf750-ab55-49e3-9fd6-2e296a97bd1f.gif" width ="250">

개발자 문서를 들여다 보았다.

## [UIModalTransitionStyle.partialCurl](https://developer.apple.com/documentation/uikit/uimodaltransitionstyle/partialcurl)

이 transition style 은 parent 뷰컨트롤러가 full-screen 를 표시하고 개발자가 `UIModalPresentationStyle.fullScreen` 모달 표시 스타일을 사용하는 경우에만 지원된다.

parent view(보여지는 뷰) 또는 다른 presentation style 에 대해서 다른 form factor 를 사용하려고 시도하면 예외가 트리거 된다.

라고 개발자 문서에 명시되어 있었다!
