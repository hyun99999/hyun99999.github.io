---
title:  "iOS) masksToBounds 와 clipsToBounds 차이"
categories:
- iOS

date:   2021-06-23 16:17:00 +0900
author_profile: false
---
masksToBounds 와 clipsToBounds 차이

먼저 이 두 메서드는 같은 기능을 한다. 하지만 다른 곳에서 불러온다!
```swift
label.layer.masksToBounds
label.clipsToBounds
```

- masksToBounds 는 CALayer 의 프로퍼티입니다. (CALayer 는 이미지 기반 content 를 관리하고 content 의 애니메이션을 수행할 수 있는 object)

- clipsToBounds 는 UIView 의 프로퍼티입니다.

### masksToBounds
하위 레이어가 레이어 경계까지 잘리는 여부를 결정.
- true 로 설정 시 레이어의 경계를 일치시키고 모서리 반경(corner radius) 를 포함 하는 암시적인 clipping mask 를 만든다.(default 는 false)

`var masksToBounds: Bool { get set }`

### clipsToBounds
하위 뷰가 뷰의 경계로 제한되는지 여부를 결정. 
- true 로 설정 시 경계로 제한된다.(default 는 false)

`var clipsToBounds: Bool { get set }`

### 출처
출처ㅣ[iOS ) masksToBounds/clipsToBounds의 차이점](https://zeddios.tistory.com/37)
출처ㅣ[clipsToBounds](https://developer.apple.com/documentation/uikit/uiview/1622415-clipstobounds)
