---
title:  "iOS) Info.plist 경로 변경 시 에러 해결"
categories:
- iOS

date:   2021-08-26  19:32:00 +0900
author_profile: false
---
프로젝트 초기 설정 때 Info.plist 를 이동하고자했는데 이때 에러가 발생했다.

다른 파일은 문제가 없었는데 Info.plist 를 폴더링을 적용하기 위해서 경로를 변경했더니 문제가 발생했던 것이다.

<img src ="https://user-images.githubusercontent.com/69136340/130947667-647fbb27-a876-44bf-abed-72f3dcdef614.png" width ="700">

다음과 같이 설정된 경로를 바꿔주면된다. 나같은 경우는 Resource 폴더 하위에 위치하기 때문에

`KakaoQRcode-iOS-CloneCoding/Resource/Info.plist` 라고 설정해주었다.

<img src ="https://user-images.githubusercontent.com/69136340/130947672-827df969-9f92-4428-bd5a-c654f4944051.png" width ="700">
