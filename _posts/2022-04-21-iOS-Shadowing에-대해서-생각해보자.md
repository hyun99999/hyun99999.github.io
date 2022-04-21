---
title:  "iOS) Shadowing 에 대해서 생각해보자"
categories:
- iOS

date:   2022-04-21  23:36:00 +0900
author_profile: false
---
### 내용

- Swift 5.7 부터 옵셔널 변수를 언래핑하는 작업이 개선될 예정입니다. 이와 함께 Shadowing 에 대해서 알아봅시다.

### Shadowing 이란?

우리는 언래핑을 할 때 안전하게 진행하기 위해서 옵셔널 바인딩을 합니다.

```swift
var x: Int?
        
if let x = x {
   // do something with the new x
}
// or
guard let x = x { return }
```

그리고우리는 결과적으로 x 라는 이름의 새 상수를사용하게 됩니다.

이 작업을 ***Shadowing*** 이라고 합니다. (새로운 상수가 언래핑 하고자 했던 옵셔널 변수의 그림자와 같기 때문이라고 합니다.)

실제로 우리가 사용하는 변수의 이름은 더 길고 때로는 훨씬 더 깁니다. 예를들어

```swift
var lastTimeUserEnteredTheApp: Date?

if let lastTimeUserEnteredTheApp = lastTimeUserEnteredTheApp {

   // do something with the new variable
}
```

이러한 긴 이름의 옵셔널 변수는 ***확정된 언래핑 변수 코드***를 생성합니다. 즉, 어쩔 수 없이 긴 변수명을 두번이나 사용하게 된다는 것이지요.

**그러면 이름을 줄이면 되는 것 아닐까요?**

```swift
var lastTimeUserEnteredTheApp: Date?
   

if let date = lastTimeUserEnteredTheApp {

     // do something with the new variable
}
```

이로써 date 로 변수명의 길이를 줄였지만, ***date 가 무엇을 의미하는지는 불분명*** 해졌습니다.

적합한 네이밍은 가독성이 좋은 코드를 위해서는 중요한 역할을 합니다. 그래서 옵셔널 변수에 대한 새로운 상수의 변수명을 지정하는 행위 즉, ***Shadowing*** 에 대해서 위와 같이 고찰해볼 사항들이 있습니다.

이제 본론입니다! Swift 5.7 은 이러한 상황을 처리하는 흥미로운 새 접근 방식을 가집니다.

### ****Shortening Shadowing****

```swift
var x: Int?
        
if let x = x {
   // do something with the new x
}
```

- 위의 코드를 아래와 같이 짧게 작성할 수 있습니다. 와우

```swift
If let x {

    // do something with the new x
}
```

변수명이 길어질 경우, 이 방식은 훨씬 유용할거에요.

```swift
if let lastTimeUserEnteredTheApp {

  // do something
  
}
// 또다른 shadowing unwrapping 방법에도 적용 가능합니다.
guard let lastTimeUserEnteredTheApp else {return}
```

**하지만, Swif 5.7 에서 허용하는 이러한 방식이 단점도 있습니다.**

### 단점

우리는 앞서 길더라도 `= x` 를 사용해서 context 즉, 문맥을 제공할 수 있었어요. 하지만 새로운 방식은 `x` 를 새로운 변수처럼 느끼게 해줍니다.

```swift
if var x {

}
```

앞서 Shadowing 의  단점 중 중복 변수를 만드는 점도 있었습니다. 그러나 이 새로운 방식은 이러한 Shadowing 을 막지 못하고, optional 변수를 숨기는 결과를 제공하게 됩니다.

앞으로 Swift 5.7 버전이 적용되면 사용하면서 느껴봐야할 것 같다.

---

출처: 

[Swift 5.7: Unwrapping Optionals Gets an Improvement](https://betterprogramming.pub/swift-5-7-unwrapping-optionals-gets-improvement-be81c578e9fa)

[swift-evolution/0345-if-let-shorthand.md at main · apple/swift-evolution](https://github.com/apple/swift-evolution/blob/main/proposals/0345-if-let-shorthand.md)
