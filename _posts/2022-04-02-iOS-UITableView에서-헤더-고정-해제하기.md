---
title:  "iOS) UITableView 에서 헤더 고정 해제하기"
categories:
- iOS

date:   2022-04-02  14:09:00 +0900
author_profile: false
---
### 내용

- UITableView 에서 헤더를 사용하니까 스크롤 시 고정되어 원하는 UI 가 구현되지 않았다.

### 문제
<img src="https://user-images.githubusercontent.com/69136340/161309706-182ae3e7-82f7-4432-b1cc-47a43ac8cac5.mp4" width = "250">


### UITableView.Style 이 grouped 가 아닌 plain 일때 스크롤하면 헤더가 고정된다고 해요.

- 다음과 같이 변경해주면됩니다.

```swift
// 최초 선언시 이니셜라이저를 통해서 접근 가능.
private let tableView = UITableView(frame: .zero, style: .grouped)
```

### 라고할뻔.. ㅋㅋ

`plain` 과 `grouped` 의 차이점이 들어나게되요!

개발자문서를 잠깐 볼까요

<img width="700" alt="111" src="https://user-images.githubusercontent.com/69136340/161309776-3a372b7f-bf4d-448c-9219-2b3e98ae5d79.png">



`grouped` 스타일 속성은 고유한 행 그룹이 있는 섹션을 가집니다.

그래서 아래와 같이 적용됩니다.

<img src="https://user-images.githubusercontent.com/69136340/161309895-1b0e7822-a1c1-4a76-8f48-dbacfa00d123.mp4" width ="250">

UITableView.Style 은 다음과 같다고 합니다.

<img width="700" alt="222" src="https://user-images.githubusercontent.com/69136340/161309971-bde43221-9517-404a-baba-7a1211bad468.png">

사진 출처: 

[[iOS - swift] tableView 스타일) Plain vs Grouped vs Insert Grouped](https://ios-development.tistory.com/538)

그렇다면 우리가 위에서 본 섹션영역을 좀 더 자세히 봐볼게요.

view hierachy 를 살펴보면 헤더위쪽으로 또다른 영역이 존재해요! 그리고 이것은 footer 입니다!

<img width="250" alt="스크린샷 2022-03-31 오후 2 53 25" src="https://user-images.githubusercontent.com/69136340/161309925-f6dce8d3-6741-4967-be9d-4bf67f73f95c.png">

### 해결

footer의 높이를 줄이면 되겠죠?

```swift
tableView.sectionFooterHeight = 0
```
<img width="250" alt="444" src="https://user-images.githubusercontent.com/69136340/161310145-ed070b76-4c56-4618-a525-4f0a23a23247.png">

추가적으로 색상도 변경해서 완성해볼게요

```swift
tableView.backgroundColor = .sparkWhite
```

<img src="https://user-images.githubusercontent.com/69136340/161310150-7aa90bc2-9637-40f3-8820-f18182a02462.mp4" width ="250">

**참고:**

[UIKit - UITableView 를 Grouped Style 로 지정했을 때 생기는 Footer 없에기](https://kasroid.github.io/posts/ios/20200905-uikit-deleting-unwanted-spaces-in-grouped-style-uitableview/)
