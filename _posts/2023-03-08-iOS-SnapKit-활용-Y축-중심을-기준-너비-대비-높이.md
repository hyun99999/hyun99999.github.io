---
title:  "iOS) SnapKit 활용(Y축 중심을 기준 / 너비 대비 높이)"
categories:
- iOS

date:   2023-03-08  14:13:00 +0900
author_profile: false
---
아래의 뷰를 만들기 위한 과정 중에서 알게된 SnapKit 사용법을 적어보자.

- 화면의 y축 중심으로부터 constraint 를 잡고싶다.
- 너비 대비 높이를 비율로 잡고싶다.

우선 아래와 같은 레이아웃을 잡고싶었다.

<img src="https://user-images.githubusercontent.com/69136340/223625106-a2444010-8335-411c-8e74-b335da3b5f87.png" width ="200">

그래서 다음과 같이 코드를 작성하였다. 

```swift
// "기본" 이라고 적힌 UIView.
basicBackgroundView.snp.makeConstraints { make in
    // 🚨 inset 이 적용되지 않아서 y축 중심에 위치.
    make.bottom.equalTo(view.snp.centerY).inset(-63)
    make.leading.trailing.equalToSuperview().inset(24)
    // 🚨 multipliedBy 가 적용되지 않아서 뷰에서 보이지 않음.
    make.height.equalTo(basicBackgroundView.snp.width).multipliedBy(117 / 327)
}

// 나머지 "직장", "덕질" 이라고 적힌 UIView 는 높이를 정적이게 설정.
```

결과는 아래와 같았다.

<img src="https://user-images.githubusercontent.com/69136340/223625089-2b92bfa0-f1d4-4dcb-bf98-75af6c2f72e8.png" width ="200">

이런 경우에 다음과 같이 코드를 작성하면 된다.

```swift
basicBackgroundView.snp.makeConstraints { make in
    // ✅ inset 이 아닌 offset 으로 다룸.
    make.bottom.equalTo(view.snp.centerY).offset(-63)
    make.leading.trailing.equalToSuperview().inset(24)
    // ✅ 소수점을 표시해줌.
    make.height.equalTo(basicBackgroundView.snp.width).multipliedBy(117.0 / 327.0)
}
```
