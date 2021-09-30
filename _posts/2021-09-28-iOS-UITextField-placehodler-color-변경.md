---
title:  "iOS) UITextField placehodler color 변경"
categories:
- iOS

date:   2021-09-28  21:12:00 +0900
author_profile: false
---
**내용**

- UITextField placehodler color 변경

### 결과

```swift
textField.attributedPlaceholder = NSAttributedString(string: "명함이름", attributes: [NSAttributedString.Key.foregroundColor: UIColor.system6])
```

다음과 같이 textField 가 어두운 경우 placeholder 의 색을 밝게 변경할 수 있다.

<img src ="https://user-images.githubusercontent.com/69136340/135084193-378330af-4f19-4677-80ce-5c4832e03922.png" width ="350">

