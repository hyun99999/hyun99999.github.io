---
title:  "iOS) UIButton programmatically"
categories:
- iOS

date:   2021-05-19 01:24:00 +0900
author_profile: false
---
UIButton 코드로 짜기

```swift
let bookBtn = UIButton()
//title
bookBtn.setTitle("예매율순", for: .normal)

//title color
bookBtn.setTitleColor(.black, for: .normal)

//title fontsize
bookBtn.titleLabel?.font = UIFont.systemFont(ofSize: 12)

//imageview image
bookBtn.setImage(UIImage(systemName: "circle.fill"), for: .normal)

//imageview image size
bookBtn.setPreferredSymbolConfiguration(.init(pointSize: 3, weight: .regular, scale: .default), forImageIn: .normal)

//imageview image color
bookBtn.tintColor = .black

//UIButton insets
bookBtn.titleEdgeInsets = .init(top: .zero, left: 8, bottom: .zero, right: .zero)

//UIButton add action
bookBtn.addTarget(self, action: #selector(touchBookBtn), for: .touchUpInside)
```
