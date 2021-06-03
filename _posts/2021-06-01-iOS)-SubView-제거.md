---
title:  "iOS) SubView 제거"
categories:
- iOS

date:   2021-06-01 23:44:00 +0900
author_profile: false
---
### SubView 제거

```swift
// 추가할 서브뷰를 생성
var imageView:UIImageView?

// 서브 view 를 식별할 태그를 지정
imageView?.tag=100

// 서브 뷰를 불러오고
let viewWithTag = self.view.viewWithTag(100)

// 해당 뷰를 제거한다
viewWithTag.removeFromSuperview()
```

### 출처
출처ㅣhttps://devureak.tistory.com/13
