---
title:  "iOS) @available obsoleted 와 deprecated"
categories:
- iOS

date:   2022-09-03  00:01:00 +0900
author_profile: false
---
## obsoleted

```swift
@available(macOS, obsoleted: 12.0)
```

<img width="400" alt="스크린샷 2022-08-09 오후 11 13 32" src="https://user-images.githubusercontent.com/69136340/188300861-50d0a17f-e51d-4176-8665-ec94d00fe633.png">

## deprecated

```swift
@available(macOS, deprecated: 12.0)
```

<img width="400" alt="1" src="https://user-images.githubusercontent.com/69136340/188300862-7ba27eae-3dfc-4508-8eee-36ebab1bbe7c.png">

## 동시 사용

```swift
import Foundation

enum School: String{
    case elementary = "초등학교"
    case middle = "중학교"
    case high = "고등학교"
    case college = "대학교"
    // deprecated : 더이상 사용되지 않는(경고) 첫번째 버전.
    // obsoleted : 폐기되어 사용할 수 없는(에러) 첫번째 버전.
    @available(iOS, deprecated: 13.0, obsoleted: 14.0)
    // -> iOS 13 부터는 사용되지 않는 deprecated 경고.
    // -> iOS 14 부터는 사용할 수 없는 에러.
    case graduate
}
```

## unavailable 과 obsoleted

- unavailable 는 지정된 플랫폼에서 선언을 사용할 수 없음
- obsoleted 는 지정된 플랫폼의 버전에 따라 사용할 수 없음

### 출처

[https://zeddios.tistory.com/647](https://zeddios.tistory.com/647)
