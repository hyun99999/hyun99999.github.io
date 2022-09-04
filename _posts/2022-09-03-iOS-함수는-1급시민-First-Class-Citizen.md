---
title:  "iOS) 함수는 1급시민(First Class Citizen)"
categories:
- iOS

date:   2022-09-03  00:01:00 +0900
author_profile: false
---
함수는 1급시민이기 때문에 상수나 변수에 참조를 할당할 수 있습니다.

**1급시민(1급객체)란?**

- 변수나 데이터 구조 안에 담을 수 있음
- 파라미터로 전달 가능
- 반환값으로 사용 가능

위의 특징을 가지면 1급시민으로 취급한다. swift 에서 함수는 1급 시민이다. 그래서 다음과 같이 사용이 가능하다.

```swift
func someFunc(paramA: Any, paramB: Any) {
    print("someFunc called")
}
// ✅ 변수에 함수의 참조를 할당할 수 있습니다.
var function = someFunc(paramA:paramB:)

// ✅ 다음과같이 할당된 변수에 매개변수를 전달하여 사용할 수 있습니다.
function(1, 2)
```

출처:

[Swift 일급객체(First-class object) 일급시민(First-class citizen) / iOS프로그래밍](https://admd13.tistory.com/44)
