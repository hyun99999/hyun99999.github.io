---
title:  "iOS) Dispatch(6) - 성능 향상을 위한 static dispatch"
categories:
- iOS

date:   2022-08-18  02:03:00 +0900
author_profile: false
---
Swift 의 dynamic dispatch 는 method table 에서 함수를 찾은 다음 indirect call 을 수행하여 구현되기 때문에 direct call 보다 수행이 느립니다. 또는 indirect call 은 많은 컴파일러 최적화를 방지하여 호출 비용이 비쌉니다.

***그렇다면 서브클래싱과 오버라이딩이 필요없는 경우는  `static dispatch` 로 동작하는 것이 좋지 않을까?***

맞습니다, 성능이 중요한 코드에서 성능 향상에 필요하지 않을 때 이 dynamic dispatch 동작을 제한할 수 있는 방법이 필요합니다.

# 성능 향상을 위한 Static Dispatch

메서드 뿐만 아니라 프로퍼티 역시 오버라이딩의 가능성이 있기 때문에 `dynamic dispatch` 로 동작합니다. 그래서 상속 가능성이 없는 클래스나 오버라이딩 되지 않는 프로퍼티와 메서드에 대해서는 `static dispatch` 로 바꿔줄 필요가 있습니다.

그 세 가지 방법에 대해서 알아보겠습니다.

### 1. 상속이나 오버라이딩 될 필요가 없는 클래스와 메서드, 프로퍼티에 final 키워드 사용.

위에서 알아봤듯이 `final` 키워드를 통해서 static dispatch 를 수행할 수 있다.

### 2. private 키워드

`private` 키워드를 사용하여 컴파일러가 오버라이딩이 되는지 안되는지를 알아서 판단하도록 할 수 있습니다.

그리고 오버라이딩 되는 곳이 없다면 스스로 final 키워드를 추론해서 static dispatch 를 수행할 수 있다.

### 3. WMO(Whole Module Optimization) 사용

모듈 전체를 같이 컴파일하여 internal 선언에 대해서 오버라이딩 여부를 판단하여 final 을 추론해서 static dispatch 를 수행할 수 있다.

예시를 살펴봅시다. 이외의 클래스는 없다고 가정하겠습니다.

```swift
class ParticleModel {
    var point = ( 0.0, 0.0 )
    var velocity = 100.0

    func updatePoint(newPoint: (Double, Double), newVelocity: Double) {
        point = newPoint
        velocity = newVelocity
    }

    func update(newP: (Double, Double), newV: Double) {
        updatePoint(newP, newVelocity: newV)
    }
}

var p = ParticleModel()
for i in stride(from: 0.0, through: 360, by: 1.0) {
    p.update((i * sin(i), i), newV:i*1000)
}
```

컴파일러가 dynamic dispatch 를 수행하는 것은 다음과 같다.

1. p 변수의 `update` 메소드 호출
2. p 변수의 `updatePoint` 메소드 호출
3. p 변수의 튜플 `point` 프로퍼티 접근
4. p 변수의 `velocity` 프로퍼티 접근

### final

`final` 키워드는 클래스와 메소드, 프로퍼티가 오버라이드 되지 않음을 나타냅니다. 이를 통해 `dynamic dispatch` 를 안전하게 피할 수 있습니다.

```swift
class ParticleModel {
  // ✅ static dispatch.
    final var point = ( x: 0.0, y: 0.0 )
    final var velocity = 100.0

  // ✅ static dispatch.
    final func updatePoint(newPoint: (Double, Double), newVelocity: Double) {
        point = newPoint
        velocity = newVelocity
    }

// ✅ 서브클래스가 오버라이드 가능하도록 하기 위해서 dynamic dispatch 통해 호출.
    func update(newP: (Double, Double), newV: Double) {
        updatePoint(newP, newVelocity: newV)
    }
}
```

또는 `final` 을 클래스 자체에 추가할 수 있습니다. 이는 서브클래싱하는 것을 금지하고, 모든 메소드와 프로퍼티가 `final` 임을 의미합니다.

```swift
final class ParticleModel {
    var point = ( x: 0.0, y: 0.0 )
    var velocity = 100.0
    // ...
}
```

### private

`private` 키워드를 적용하여 선언의 가시성을 현재 파일로 제한합니다. 이를 통해 컴파일러는 잠재적으로 오버라이딩 할 수 있는 모든 선언을 찾을 수 있습니다. 오버라이딩 선언이 없다면 컴파일러에서 `final` 키워드를 자동으로 추론하여 indirect call 을 제거할 수 있습니다.

```swift
class ParticleModel {
  // ✅ 현재 파일로 제한되기 때문에 final 키워드 자동으로 추론.
    private var point = ( x: 0.0, y: 0.0 )
    private var velocity = 100.0

  // ✅ 현재 파일로 제한되기 때문에 final 키워드 자동으로 추론.
    private func updatePoint(newPoint: (Double, Double), newVelocity: Double) {
        point = newPoint
        velocity = newVelocity
    }

  // ✅ private 이 아니기 때문에 dynamic dispatch 통해 호출.
    func update(newP: (Double, Double), newV: Double) {
        updatePoint(newP, newVelocity: newV)
    }
}
```

`final` 과 마찬가지로 `private` 속성을 클래스 자체에 적용하여 `static dispatch` 를 기대할 수 있습니다.

```swift
private class ParticleModel {
    var point = ( x: 0.0, y: 0.0 )
    var velocity = 100.0
    // ...
}
```

### WMO(Whole Module Optimization)

Swift 는 일반적으로 모듈을 구성하는 **파일들을 별도로 컴파일**하기 때문에 컴파일러는 internal 선언이 다른 파일에서 재정의되는지 여부를 확인할 수 없습니다.(그렇죠! 그래서 dynamic dispatch 를 수행합니다.)

- internal(접근 제한자를 설정하지 않았을 때 기본값)은 선언된 모듈 내에서만 볼 수 있습니다. 같은 모듈 내에서 상속, 오버라이딩이 가능하지만 모듈 외부에서 접근은 불가능합니다.

하지만, WMO 가 활성화된 경우 모든 모듈이 동시에 컴파일됩니다. 이를 통해 컴파일러는 전체 모듈에 대한 추론을 할 수 있고 재정의가 없는 경우 `final` 을 추론할 수 있습니다. 

WMO 는 Xcode 8 부터 릴리즈일 때 자동으로 활성화 되어 있습니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f09c7bc4-74ac-4877-b1a6-055f57129984/Untitled.png)

```swift
public class ParticleModel {
  // ✅ final 을 추론.
    var point = ( x: 0.0, y: 0.0 )
    var velocity = 100.0

  // ✅ final 을 추론.
    func updatePoint(newPoint: (Double, Double), newVelocity: Double) {
        point = newPoint
        velocity = newVelocity
    }

  // ✅ open 이기 때문에 final 이라고 추론할 수 없음.
  // 외부 모듈에서 오버라이딩 할 가능성 존재.
    open func update(newP: (Double, Double), newV: Double) {
        updatePoint(newP, newVelocity: newV)
    }
}

var p = ParticleModel()
for i in stride(from: 0.0, through: times, by: 1.0) {
    p.update((i * sin(i), i), newV:i*1000)
}
```

### 잠깐!

[Increasing Performance by Reducing Dynamic Dispatch - Swift Blog](https://developer.apple.com/swift/blog/?id=27)

위의 글을 참고해서 작성하였는데 해당 글은 Swift 2 일 때 작성이 되었고, 이때는 public 이 지금의 open 과 같았다고 한다. Swift 3 부터 public 과 open 으로 구분되었다.

그렇기 때문에 위의 글은 public 접근제한자를 사용했지만, 현재 public 은 외부 상속이 불가능하고, 오버라이딩할 수 없다. 정의된 모듈 내에서만 서브 클래싱이 가능하다.

open 은 모듈 밖에서 상속과 오버라이딩 할 수 있기 때문에 예제 코드를 open 으로 변경하였다.

이 외에도 가능하다면 서브클래싱이 불가능한 Value Type 인 `Struct` 와 `enum` 을 사용하는 것도 방법이 될 수 있겠다.

### 출처

[Increasing Performance by Reducing Dynamic Dispatch - Swift Blog](https://developer.apple.com/swift/blog/?id=27)

[Swift) Dispatch를 이용한 성능 최적화](https://babbab2.tistory.com/145?category=828998)

[[iOS - swift] Static Dispatch, Dynamic Dispatch 성능 최적화 방법, Witness Table, (final, private을 사용하는 이유)](https://ios-development.tistory.com/816)

[[Swift] Dynamic Dispatch / Static Dispatch](https://dev-in-gym.tistory.com/161)
