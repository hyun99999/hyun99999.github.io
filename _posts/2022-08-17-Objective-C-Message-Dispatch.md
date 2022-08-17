---
title:  "iOS) Dispatch(4) - Message Dispatch"
categories:
- iOS

date:   2022-08-17  23:58:00 +0900
author_profile: false
---
 # Message Dispatch

**Objective-C 는 클래스의 메소드가 프로퍼티를 호출할 때 해당 객체에 메시지를 보내는 방식으로 구현되어 있습니다. 그리고 이 과정이 런타임 시에 일어납니다. 이것이 `message dispatch` 입니다.**

즉, message dispatch 는 dynamic dispatch 의 일종입니다.

**message dispatch 는 오버라이딩하거나 새로 정의한 메소드들만 테이블에 유지합니다.**(swift 의 dynamic dispatch 는 모든 메소드에 대한 포인터를 해당 클래스가 가짐.) **대신, 부모 클래스로의 포인터를 가지고 있기 때문에 상속받은 메소드들을 찾아갈 수 있습니다.**

대신 Swift 는 이런 기능을 자체적으로 제공하지 않기때문에 message dispatch 를 Swift 에서도 사용하기 위해서는 `dynamic` 과 `@objc` 키워드가 필요합니다.

### 그렇다면 message dispatch 를 Swift 에서 왜 사용하나요?

message dispatch 방식은 굉장히 유연해서 런타임에 메소드를 수정하는 것부터 새로운 메소드나 프로퍼티를 수정하고, 클래스를 동적으로 만드는 것도 가능합니다. 또한, 코어데이터나 KVO 과 같은 동적인 기능은 Objective-C 런타임 덕분에 가능하기 때문입니다.

### dynamic

**dynamic 은 Swift 와 Objective-C 의 상호운용성 때문에 사용됩니다.** 즉, Objective-C 에서 dynamic dispatch 를 Swift 에서도 사용하기 위한 역할입니다.

그래서 `dynamic var` 혹은 `dynamic func` 이 의미하는 것은 Swift 의 런타임이 아닌 Objective-C 런타임에서 사용하겠다는 것입니다. 즉, 객체한테 메시지를 보내고, 객체가 메소드의 위치를 찾아서 실행하는 과정입니다.

참고로 오직 Class 의 멤버에만 사용할 수 있습니다. 런타임에 결정할 필요가 없는 구조체와 열거형에서는 당연히 필요가 없습니다.

### @objc

@objc 어노테이션을 앞에 붙이게 되면 Objective-C 런타임에 접근할 수 있게 합니다. 하지만, selector 등으로 접근하지 않는 경우는 원래의 dispatch 가 적용된다고 해요.

### 잠깐 !

[swift-evolution/0160-objc-inference.md at master · apple/swift-evolution](https://github.com/apple/swift-evolution/blob/master/proposals/0160-objc-inference.md#dynamic-no-longer-infers-objc)

위의 evolution 문서를 보게되면  Swift 4 이전에는 `dynamic` 어노테이션을 앞에 붙이게 되면 `@objc` 을 추론해서 추가했는데 이후부터는 dynamic 만 사용해서는 `@objc` 를 자동으로 추가해주지 않습니다.

이러한 변화는 다음 두 가지를 의미합니다.

- 미래에는 Swift 언어와 런타임이 Objective-C 런타임에 의존하지 않고 dynamic 을 지원하도록 진화할 가능성이 있고, 진화의 문을 열어두는 것이 중요하다.
- dynamic 동작이 Objective-C 런타임과연결되어 있음을 분명히 합니다.

(단, `@objc` 어노테이션이 적용된 프로토콜의 메소드를 구현하거나 클래스 전체에 `@objcMember` 어노테이션이 적용되어 있다면 자동 추론)

결과적으로 `@objc dynamic` 와 같이 어노테이션을 함께 사용해야 합니다.

## message disaptch 를 사용해야할 때

앞서 잠깐 Swift 에서 사용되는 곳을 알아보았는데 좀 더 자세히 살펴봅시다!

1. extension 에서 함수 오버라이딩

Swift 에서는 extension 에서의 함수 오버라이딩이 불가능합니다. 이를 시도하면 다음과 같은 에러가 발생합니다.

```swift
import Foundation

class someClass {
    func action() {
        print("action")
    }
}

class childClass: someClass { }

extension childClass {
    override func action() {
        print("override")
    }
}

// ERROR
// Non-@objc instance method 'action()' declared in 'someClass' cannot be overridden from extension
```

에러 메시지를 통해 @objc 어노테이션을 추가해보겠습니다.

```swift
import Foundation

class someClass {
// ✅ @objc 어노테이션 추가.
    @objc func action() {
        print("action")
    }
}

class childClass: someClass { }

extension childClass {
    override func action() {
        print("override")
    }
}

// ERROR
// Cannot override a non-dynamic class declaration from an extension
```

여전히 에러는 존재합니다. dynamic 어노테이션을  추가해서 해결하겠습니다.

```swift
import Foundation

class someClass {
// ✅ dynamic 어노테이션 추가.
    @objc dynamic func action() {
        print("action")
    }
}

class childClass: someClass { }

extension childClass {
    override func action() {
        print("override")
    }
}
```

`**@objc dynamic` 어노테이션을 통해 extension 에서 메소드를 오버라이딩 할 수 있습니다.**

1. Selector, KVO 와 같은 Objective-C 런타임 시에만 제공되는 기능

### 출처

[Swift) dynamic이란? / Realm의 dynamic var는?](https://zeddios.tistory.com/296)

[Message Dispatch](https://jcsoohwancho.github.io/2019-11-02-Message-Dispatch/)
