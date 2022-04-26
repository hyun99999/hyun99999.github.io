---
title:  "swiftui) 이미지에 border 추가하기"
categories:
- iOS

date:   2022-04-21  23:51:00 +0900
author_profile: false
---
```swift
var body: some View {
    Image("featuredImage1")
        .resizable()
        .scaledToFill()
        .cornerRadius(5)
// overlay 를 통해서 border 를 추가할 수 있다.
        .overlay(RoundedRectangle(cornerRadius: 5)
            .stroke(Color.secondary, lineWidth: 0.2))
    }
}
```
