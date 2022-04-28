---
title:  "iOS) Instagram 스토리, 피드 공유기능 구현해보자"
categories:
- iOS

date:   2022-04-28  14:37:00 +0900
author_profile: false
---
### 내용
- instagram story,feed 공유기능 구현을 목표

### Story 공유
>- Android 암시적 인텐트 및 iOS 맞춤 URL 스키마를 사용하면 앱에서 사진, 동영상과 스티커를 Instagram 앱으로 보낼 수 있습니다.
>- Instagram 앱이 해당 콘텐츠를 받아서 스토리 작성기에 읽어들이면 사용자는 콘텐츠를 Instagram 스토리에 게시할 수 있습니다.
- Instagram 앱의 스토리 작성기는 배경 레이어와 스티커 레이어로 구성됩니다.
<img src="https://user-images.githubusercontent.com/69136340/106865995-4386a500-670f-11eb-851b-8d200ae48aab.png" width="300">

- 배경 레이어
  > 배경 레이어는 화면을 채우며 사진, 동영상, 단색 또는 컬러 그래디언트로 맞춤 설정할 수 있습니다.
- 스티커 레이어
  > 스티커 레이어는 이미지를 포함할 수 있으며, 사용자가 스토리 작성기에서 레이어를 추가로 맞춤 설정할 수 있습니다.

### Instagram의 맞춤 URL 스키마를 허용 리스트에 등록
- 앱에서 Instagram의 맞춤 URL 스키마를 사용하려면 허용 리스트에 추가해야 합니다. 이를 위해서는 앱의 Info.plist에 있는 LSApplicationQueriesSchemes 키에 instagram-stories를 추가합니다.

### Info.plist
<img src ="https://user-images.githubusercontent.com/69136340/106881464-ed226200-6720-11eb-8126-d381773aa0e1.png" width="700">

## 완성
<p>
<img src = "https://user-images.githubusercontent.com/69136340/106991726-2b1f9480-67ba-11eb-896f-dc54529b9504.png" width="250">
<img src = "https://user-images.githubusercontent.com/69136340/106991729-2bb82b00-67ba-11eb-953d-88f4826eb001.png" width="250">
<img src = "https://user-images.githubusercontent.com/69136340/106991732-2ce95800-67ba-11eb-9c31-f6e4590be40d.png" width="250">
</p>

### 출처
- instagram story share : https://developers.facebook.com/docs/instagram/sharing-to-stories

---
### Feed 공유
>- Android 암시적 인텐트와 iOS 맞춤 URL 스키마 또는 문서 상호작용을 사용하면 앱에서 사진과 동영상을 Instagram 앱으로 보낼 수 있습니다.
>- Instagram 앱이 이 콘텐츠를 받아서 피드 작성기에 읽어들이면 사용자가 콘텐츠를 Instagram 피드에 게시할 수 있습니다.

### Info.plist
<img src ="https://user-images.githubusercontent.com/69136340/107011060-7a75bd00-67da-11eb-82e3-ff1778b37917.png" width="700">

## 완성
<p>
<img src="https://user-images.githubusercontent.com/69136340/107010528-cc6a1300-67d9-11eb-86fd-4cfdc2063024.jpeg" width="250">
<img src="https://user-images.githubusercontent.com/69136340/107010438-ad6b8100-67d9-11eb-8a10-568041fb0acb.png" width="250">
<img src="https://user-images.githubusercontent.com/69136340/107010416-a775a000-67d9-11eb-9226-c36c85e0c3f3.png" width="250">
<img src="https://user-images.githubusercontent.com/69136340/107010436-ac3a5400-67d9-11eb-9dfe-d173b79d66f4.png" width="250">
</p>

### 출처
- 171.0 버전 이후로 UIActivityViewController 를 사용하는 코드를 추가해야 실행된다고 한다.
- stackoverflow : https://stackoverflow.com/questions/65003868/instagram-sharing-to-feed-ios-not-working-uidocumentinteractioncontroller

### github

[https://github.com/hyun99999/instagramStorySharePractice-iOS](https://github.com/hyun99999/instagramStorySharePractice-iOS)
