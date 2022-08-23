---
title:  "iOS)UIView.animate(withDuration:animations:completion:) 왜 메모리릭이 발생하지 않나요?"
categories:
- iOS

date:   2022-08-23  19:29:00 +0900
author_profile: false
---
[Is it necessary to use [unowned self] in closures of UIView.animateWithDuration(...)?](https://stackoverflow.com/questions/27019676/is-it-necessary-to-use-unowned-self-in-closures-of-uiview-animatewithduration)

***이 글은 위의 stackoverflow 질문을 읽고 정리해본 글입니다.***

작성자님의 질문을 아래와 같았다. 아래의 코드는 메모리 릭을 피할 수 있나요?

```swift
UIView.animateWithDuration(1,
        animations: { [unowned self] in
            self.box.center = self.boxTopRightPosition
        },
        completion: { [unowned self] completed in
            self.box.hidden = true
        })
```

**이 글에는 꽤나 많은 부분들이 연결되어 있기 때문에 샅샅히 살펴봐야 합니다. 🧐**

- 우선 위의 `animateWithDuration:animations:completion:` 메소드는 Swift 가 아닌 Objective-C 의 메소드이기 때문에 Swift 의 `animate(withDuration:animations:completion:)` 메소드와 동일하다고 보시면 됩니다.
- `memory leak` 과 앞으로 이야기할 `strong reference cycle` 에 대해서 무엇인지 정의해봅시다.

## ✅ Memory leak? Strong Reference Cycle?

- 메모리 릭: 필요하지 않은 메모리를 계속 점유하고 있는 현상.
- Strong Reference Cycle(강한 순환 참조): 강한 참조 관계를 통해 형성된 레퍼런스 카운트가 참조 해제 후에도 0이 아니기 때문에 메모리에서 해제되지 않는 현상. retain cycle 이라고도 함.

## ✅ 메모리 릭을 피할 수 있나요?

답변부터 살펴보자면, **네! 피할 수 있습니다.**

그 이유는 [unowned self] 캡쳐리스트를 통해 강한 순환 참조을 피할 수 있기 때문입니다.

그렇다면 아래의 코드는 메모리 릭을 피할 수 없을까요?

```swift
UIView.animate(withDuration: 1.0,
        animations: {
            self.box.center = self.boxTopRightPosition
        },
        completion: {
            self.box.hidden = true
        })
```

**네! 피할 수 있습니다.**

<img width="400" alt="스크린샷 2022-08-23 오후 3 20 17" src="https://user-images.githubusercontent.com/69136340/186135211-d19aa038-531e-44ad-93ee-bb782dce8233.png">

바로 `UIView.animateWithDuration:animations:completion:` 타입 메소드때문입니다.

즉, 강한 순환 참조를 우려해 캡쳐리스트를 사용하기 이전에 타입 메소드이기 때문에 강한 순환 참조에서 우리는 자유롭습니다.(참조의 기본은 강한 참조이다)

그렇다면 **왜 animations 와 completion 클로저 안에서 [unowned self] 와 self 를 통해서 참조하는지에 대해서 먼저 알아봅시다.**

## ✅ [unowned self] 은 왜 사용되나요?

***animations 와 completion 클로저에서 왜 캡쳐리스트를 사용하는 것인가에 대해서 출발해보겠습니다.***

**escaping closure 로 구성된 animations 와 completion handler 를 주목해야 합니다.** non-escaping closure 는 해당 함수의 scope 내에서 실행되기 때문에 `self.` 혹은 `캡쳐리스트`를 사용하여 참조할 필요가 없습니다.

대신, escaping closure 는 해당 함수 scope 밖에서 실행되기 때문에 위의 방법을 통해 참조를 가져야 합니다.

**그렇다면 `[weak self]` 는 무엇이고, `[unowned self]` 는 무엇인가요?**

예를 들어, 뷰가 해제된 경우에도 뷰컨트롤러는 애니메이션 블록을 유지하고 있기 때문에 completion handler 가 완료될 때까지 해제되지 않습니다. 그런데 이때 없는 뷰에 대한 애니메이션을 유지하는 것이 합리적이지 않을 수 있습니다.

뷰가 없는 애니메이션은 의미가 없기 때문에 뷰가 해제된 후에는 animations 와 completion 블럭 안의 뷰에 대해서 캡쳐리스트 [weak self] 를 사용하는 것은 좋은 옵션이 됩니다.

***단, [unowned self] 를 사용하는 경우 completion handler 가 호출되기 전에 할당 해제되는 경우 충돌이 발생할 수 있습니다.***

<img src="https://user-images.githubusercontent.com/69136340/186135380-32329012-df3c-403a-93df-eae07f73c4cb.gif" width ="400">

## ✅ 왜 [unowned self] 만 충돌이 일어나나요?

unowned(미소유 참조)는 약한 참조처럼 인스턴스의 reference count(참조 횟수)를 증가시키지 않습니다.

**대신 참조하는 인스턴스가 항상 메모리에 존재할 것이라는 전제를 기반으로 동작합니다.** 즉, 인스턴스가 메모리에서 해제되더라도 스스로 nil 을 할당해주지 않습니다.

- (약한 참조는 스스로 nil 을 할당할 수 있어야하기 때문에 상수일 수 없고, 옵셔널 변수여야만 합니다. 미소유 참조는 항상 nil 이 아닐 것이라는 전제를 기반으로 두기 때문에 상수도 가능합니다.)

[unowned self] 를 사용하게 되면 nil 에 대한 전제없이 **사실상 nil 이 되어버린 인스턴스를 completion handler 에서 참조하려고 하기 때문에 위의 예시처럼 화면전환을 통해 인스턴스가 해제될 때 충돌이 발생합니다.**

## ✅ strong 은요?

[weak self] 와 [unowned self] 의 결과에 대해서 이해해보았습니다. 그렇다면 strong 즉, 강한 참조는요?

위의 예시에서 로그를 확인하면 yellow completion 까지 호출이 된것을 보아 화면 전환 후에도 completion handler 가 실행된 것을 알 수 있습니다.

이는 강한 참조가 레퍼런스 카운트를 증가시키기 때문에 환면전환으로 인한 뷰컨트롤러가 해제되면서 레펀런스 카운트를 감소시킴에도 불구하고 0이 되지 않아 메모리 상에 남아있기 때문입니다.

그렇기때문에 completion handler 가 완료된 후에 메모리에서 해제되면서 deinit 되는 로그를 확인할 수 있습니다.

## ✅ 타입 메소드

맨 처음에 스택오버플로우 코드의 여파는 굉장했습니다…

[unowned self] 를 사용하며 강한 순환 참조를 피하려한 것처럼 보이기도 하지만… 사실은 UIView 클래스의 타입 메소드를 사용함으로써 이미 강한 순환 참조에서 자유로웠던 구조입니다.

이제 조금 이해가 되실까요?(전 굉장히 힘들었답니다)

<img src="https://user-images.githubusercontent.com/69136340/186135469-937e0ae4-759a-41c7-b296-8fcaf6c151f2.png" width ="250">

자, 이제 타입 메소드를 사용했던 점이 어떻게 강한 순환 참조에서 자유로운지 알아봅시다.

***(상수 혹은 변수에 인스턴스를 할당할 때 해당 인스턴스에 대한 참조가 생깁니다.)***

***(클로저 내부에서 self 프로퍼티를 여러 번 호출하여 접근하더라도 레퍼런스 카운트는 한 번만 증가합니다.)***

```swift
UIView.animate(withDuration: 1.0,
        animations: {
            self.box.center = self.boxTopRightPosition
        },
        completion: {
            self.box.hidden = true
        })
```

타입 메소드이기 때문에 인스턴스를 생성하지 않고 호출합니다. 이때문에 view controller 는 animate 의 파라미터에 있는 클로저를 참조하지 않습니다. 즉, 레퍼런스 카운트는 올라가지 않습니다.

클로저에서 self 를 통한 레퍼런스 카운트가 올라갑니다. 그래서 클로저가 끝나면 view controller 의 레퍼런스 카운트가 0이 되어 deinit 이 호출됩니다.

하지만, 변수를 만들고 참조가 된 클로저를 넣어 파라미터에 전달하면 강한 순환 참조가 일어나 뷰컨트롤러 해제 후에도 레퍼런스 카운트가 0이 되지 않아 메모리가 해제되지 않습니다.

***이때 강한 순환 참조과 상관없이 상황에 따라서 우리는 strong, weak, unowned 를 고려해야할 순간이 찾아옵니다.***

### ✅ weak

뷰컨트롤러가 해제되면 참조할 인스턴스가 사라지기 때문에 self 는 nil 이 되고 경우에 따라 completion handler 까지 완료되지 않게 됩니다.

옵셔널 변수로 사용되기 때문에 옵셔널 바인딩 과정을 거쳐야 하고, 이때 주로 `gaurd let self = self else { }` 와 `self?`  방식을 사용하게 됩니다.

- `guard let self = self else { }` 의 경우는 해당 escaping clousre 가 시작할 때 self 에 대한 참조를 만들고 시작하기 때문에 메모리에서 뷰컨트롤러가 해제되더라도 꼭 마무리해야 할 작업이 있다면 사용하는데 적합하다. 즉, 메모리 해제를 지연시킵니다.
- `self?` 는 사용할 때마다 매번 nil 인지 확인하고 실행하기 때문에 중간에 메모리에서 뷰컨트롤러가 해제되어 마무리되지 않아도 되는 작업이 있다면 사용하는데 적합하다. 즉, 메모리 해제를 지연시키지 않습니다.

### ✅ unowned

인스턴스가 항상 메모리에 존재할 것이라는 전제하에 사용되기 때문에 생명주기가 같거나 긴 것이 보장될 때 사용됩니다. weak 와 다르게 옵셔널 변수로 사용해야만 하는 것이 아닌 일반 변수와 상수로도 사용할 수 있기 때문에 클로저에서 다루기가 용이합니다.

하지만, 메모리가 해제되는 시점에서 nil 이 된 self 에 접근하기 때문에 충돌이 발생합니다. 그렇기 때문에 생명주기를 고려해야 합니다.

### ✅ strong

참조하는 인스턴스가 메모리에서 해제되더라도 강한 참조로 인한 레퍼런스 카운트가 올라가있기 때문에 무조건 메모리 해제를 지연시키게 됩니다.

### 참고

[Swift ) ARC / Strong Reference Cycle 해결 방법(weak, unowned)](https://zeddios.tistory.com/1213)

[unowned self vs weak self (캡쳐 리스트)-(1)](https://jinswift.tistory.com/6)

[[Swift] weak, unowned, Retain cycle 톺아보기](https://hyunndyblog.tistory.com/154)

[Is it necessary to use [unowned self] in closures of UIView.animateWithDuration(...)?](https://stackoverflow.com/questions/27019676/is-it-necessary-to-use-unowned-self-in-closures-of-uiview-animatewithduration)

[Apple Developer Documentation](https://developer.apple.com/documentation/uikit/uiview/1622515-animate)

[Apple Developer Documentation](https://developer.apple.com/documentation/uikit/uiview/1622515-animatewithduration?language=objc)
