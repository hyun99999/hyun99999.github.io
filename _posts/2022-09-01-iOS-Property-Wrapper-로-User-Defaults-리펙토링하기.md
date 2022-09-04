---
title:  "iOS) Property Wrapper 로 User Defaults 리펙토링하기"
categories:
- iOS

date:   2022-09-01  15:36:00 +0900
author_profile: false
---
### 내용

- Property Wrapper 를 사용하여 UserDefaults 읽고 쓰고 삭제하는 매커니즘을 캡슐화 해보자.

## Swift-Evolution

swift-evolution 에서 property wrappers 를 소개하면서 예시로 UserDefaults 의 매커니즘을 캡슐화하여 사용하는 예시 코드를 소개했다. 살펴보자!

[swift-evolution/0258-property-wrappers.md at main · apple/swift-evolution](https://github.com/apple/swift-evolution/blob/main/proposals/0258-property-wrappers.md#user-defaults)

프로퍼티 래퍼는 user defaults 와 같은 string-keyed 데이터에 대한 typed property 들을 제공하는데 사용할 수 있습니다. wrapper type 에서 데이터를 추출하는 매커니즘을 아래처럼 캡슐화할 수 있습니다.

```swift
@propertyWrapper
struct UserDefault<T> {
  let key: String
  let defaultValue: T
  
  // ✅ Userdefaults 로 값을 찾을 때 없다면, defaultValue 를 반환.
  // 그렇기 때문에 wrappedValue 는 옵셔널이 아님.
  var wrappedValue: T {
    get {
      return UserDefaults.standard.object(forKey: key) as? T ?? defaultValue
    }
    set {
      UserDefaults.standard.set(newValue, forKey: key)
    }
  }
}

enum GlobalSettings {
  @UserDefault(key: "FOO_FEATURE_ENABLED", defaultValue: false)
  static var isFooFeatureEnabled: Bool
  
  @UserDefault<Bool>(key: "BAR_FEATURE_ENABLED", defaultValue: false)
  static var isBarFeatureEnabled
}
```

다음과 같은 방법으로도 사용이 가능하다. 캡슐화 방식은 결국 얼마나 비슷한 기능을 묶고, 외부에서 내부의 연산과정을 얼마나 잘 숨겼냐에 따라 구현방식이 다를 수 있다.

```swift
@propertyWrapper
struct UserDefault<T> {
  // ✅ backtick 을 사용하여 예약어를 변수로 사용.
    private var `default` = UserDefaults.standard
    private let key: String

  // ✅ 생성자를 통해 key 초기화.
    init(key: String) {
        self.key = key
    }

  // ✅ UserDefaults 로 값을 찾을 때 없다면 nil 반환.
  // 호출 부분에서 nil 에 대한 대응을 해주어야 함.
  var wrappedValue: T? {
    get {
            self.default.object(forKey: key) as? T
        }
        set {
            self.default.setValue(newValue, forKey: key)
        }
    }
}

/// 선언한 propertyWrapper를 사용하면 아래와 같다. 
final class UserManager {
    @UserDefault<String>(key: "usesTouchID") 
    private var usesTouchID

  // ✅ usesTouchID 는 String? 타입입니다.
  // @UserDefault(key: "usesTouchID") 
    // private var usesTouchID: String?
}

// ✅ 다음과 같이 호출하는 곳에서 nil-Coalescing Operator(nil 병합 연산자)이나 옵셔널 바인딩 등을 활용하여 대응할 수 있다.
let usesTouchID: String = UserManager().usesTouchID ?? ""
```

### 참고

[후로훠티 래퍼 · Discussion #5 · 28th-SOPT-iOS-CloneCoding/weakselfWang](https://github.com/28th-SOPT-iOS-CloneCoding/weakselfWang/discussions/5)

[swift-evolution/0258-property-wrappers.md at main · apple/swift-evolution](https://github.com/apple/swift-evolution/blob/main/proposals/0258-property-wrappers.md#user-defaults)
