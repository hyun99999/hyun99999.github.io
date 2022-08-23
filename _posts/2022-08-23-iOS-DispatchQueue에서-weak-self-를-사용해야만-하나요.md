---
title:  "iOS) DispatchQueue에서 [weak self] 를 사용해야만 하나요?"
categories:
- iOS

date:   2022-08-23  22:36:00 +0900
author_profile: false
---
***좀 더 근본적인 질문을 던져보자.***

## ✅ DispatchQueue **에서** strong reference cycle(강한 참조 순환) **은 발생할 수 있는가?**

라는 질문에 대해서 결론적으로는 변수로 저장하지 않으면 강한 참조 순환이 발생하지 않는다.

즉, DispatchQueue 를 이니셜라이저를 통해 만들어서 변수에 저장하지 사용하지 않는 이상 강한 참조 순환은 발생하지 않는다.

DispatchQueue 의 `main` 과 `global` 은 각각 타입 프로퍼티와 타입 메소드이기 때문에 호출하는 인스턴스가 레퍼런스 카운트를 올리지 않고, 강한 참조 순환에서 자유로울 수 있다.

## ✅ [weak self] 를 사용하지 않아도 될까?

***그렇다면 [weak self] 와 같은 캡처리스트를 사용하지 않아도 될까? 그럼에도 불구하고 [weak self] 를 사용해야 할 이유가 존재한다.*** 

다시 말해 강한 참조 순환은 발생하지 않지만 **캡처 현상**으로 인해 의도치 않게 생명주기를 길게 가져갈 수도 있는 경우를 해결하기 위해서 캡처리스트를 사용하는 것이다.

❗️ **중요한 것은 이 시점에서 강한 순환 참조와 캡처리스트는 더이상 관계가 없다는 것이다. 우리는 이제 생명주기에 대해 신경쓰면서 캡처리스트를 다루면 된다.**

예를 들어 버튼을 누르면 배열에 접근해 값을 추가하고 10초를 스레드를 지연시키고 main 스레드로 넘어와 UI 업데이트를 하는 코드가 있다고 가정해보자. 버튼을 누르고 화면전화를 통해 뒤로가게 되면 어떤 결과를 가질까?

이는 어떤 참조의 캡처리스트를 사용하느냐에 따라 수명주기가 결정되므로 결과가 달라지고, 목적에 따라 다르게 쓰일 수 있다.

## strong

```swift
DispatchQueue.global(qos: .userInitiated).async {
    // ✅ 아이템 추가.
    self.array.append("2")
    self.array.append("3")
    // ✅ 스레드 지연
    Thread.sleep(forTimeInterval: 10)
    
    // ✅ main 스레드로 작업을 넘김.
    DispatchQueue.main.async {
        self.array.removeLast()
        print(self.array)
        self.indicator.stopAnimating()
    }
}
```

10초가 지난 후 main 스레드에서 클로저가 실행된다. 화면 전환 후에도 클로저가 완료된 후, 뷰 컨트롤러가 deinit 된다.

### [weak self]

- `self?` 를 사용하게 되면 화면전환 즉시 메모리에서 해제되고 10초가 지나도 아무일도 일어나지 않는다. self 가 nil 이 되었기 때문이다.

```swift
DispatchQueue.global(qos: .userInitiated).async { [weak self] in
    self.array.append("2")
    self.array.append("3")
    Thread.sleep(forTimeInterval: 10)
    
    DispatchQueue.main.async {
        self?.array.removeLast()
        print(self?.array ?? "")
        self?.indicator.stopAnimating()
    }
}
```

- `guard let self = self else { return }` 을 사용하게 되면 self 의 참조를 가지기 때문에 배열에 값은 추가되지만 10초 뒤에 뷰컨트롤러가 해제되어 main 스레드의 클로저에서는 self 가 nil 이기 때문에 실행되지 않는다.

```swift
DispatchQueue.global(qos: .userInitiated).async { [weak self] in
    guard let self = self else { return }
    self.array.append("2")
    self.array.append("3")
    Thread.sleep(forTimeInterval: 10)
    
    DispatchQueue.main.async {
        guard let self = self else { return }
        self.array.removeLast()
        print(self.array)
        self.indicator.stopAnimating()
    }
}
```

### [unowned self]

```swift
DispatchQueue.global(qos: .userInitiated).async { [unowned self] in
    self.proArray.append("2")
    self.proArray.append("3")
    Thread.sleep(forTimeInterval: 10)
    
    DispatchQueue.main.async {
        self.proArray.removeLast()
        print(self.proArray)
        self.indicator.stopAnimating()
    }
}
```

unowned 은 메모리에 참조하는 인스턴스가 항상 존재할 것이라는 전제로 동작하지만, weak 와 동일하게 레퍼런스 카운트는 증가시키지 않습니다.

화면전환으로 인해 nil 이 된 self 에 대해서 참조하려고 하기 때문에 충돌이 발생하게 됩니다.

### 주의 사항

다른 스레드로 작업을 보낸다는 것은 클로저를 보내는 것이므로 **캡처 현상**이 발생할 수 있다.

**따라서 객체 내부에서 비동기(async) global queue를 사용할 경우, [weak self] 사용하지 않으면 strong 참조를 하게 된다.**

그렇기 때문에 작업 중에 참조하는 객체가 메모리에서 해제되더라도 레퍼런스 카운트가 남아있어 메모리 해지를 지연하고 작동하게 된다. (strong capture현상)

따라서 필요에 따라 [weak self] 을 통해 객체가 사라질 때 작업도 종료되도록 설정하는 것도 고려해야 한다.

### 참조

[[iOS] 디스패치큐(GCD)의 종류와 특징, 그리고 주의사항](https://tngusmiso.tistory.com/49)

[iOS - GCD what the __weak is going on?](https://medium.com/rocknnull/ios-gcd-what-the-weak-is-going-on-d5a10fc682a)

[Should we use [weak self] in GCD](https://zeushin.github.io/2017/09/26/should-we-use-weak-self-in-gcd/)

[[Swift] weak self를 사용할 때 guard let self vs self? - 일부 번역](https://baechukim.tistory.com/95)
