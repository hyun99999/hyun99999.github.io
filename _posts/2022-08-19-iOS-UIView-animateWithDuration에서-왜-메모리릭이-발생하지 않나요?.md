---
title:  "iOS) UIView.animateWithDuration(…) 에서 왜 메모리릭이 발생하지 않나요?"
categories:
- iOS

date:   2022-08-19  20:49:00 +0900
author_profile: false
---
[Is it necessary to use [unowned self] in closures of UIView.animateWithDuration(...)?](https://stackoverflow.com/questions/27019676/is-it-necessary-to-use-unowned-self-in-closures-of-uiview-animatewithduration)

***이 글은 위의 stackoverflow 질문을 읽고 정리해본 글입니다.***

작성자님의 질문을 아래와 같았다.

```swift
UIView.animateWithDuration(1,
        animations: { [unowned self] in
            self.box.center = self.boxTopRightPosition
        },
        completion: { [unowned self] completed in
            self.box.hidden = true
    })
```

위으 코드는 메모리 릭을 피할 수 있나요?

답변부터 살펴보자면, **네! 피할 수 있습니다.**

animations 와 completion unowned 캡쳐리스트를 통해 self 로 부터 retain 하지 않으므로 강력한 참조 사이클의(strong retain cycle) 위험이 없습니다.

좀 더 자세히 살펴보겠습니다.

escaping closure 로 구성된 animations 와 completion handler 를 주목해야 합니다. non-escaping closure 는 해당 함수의 scope 내에서 실행되기 때문에 `self.` 혹은 `캡쳐리스트`를 사용하여 참조를 가져갈 필요가 없습니다.

대신, escaping closure 는 해당 함수 scope 밖에서 실행되기 때문에 위의 방법을 통해 참조를 가져야 합니다.

예를 들어, 뷰가 해제된 경우에도 뷰컨트롤러는 애니메이션 블록을 유지하고 있기 때문에 completion handler 가 완료될 때까지 해제되지 않습니다. 그런데 이때 없는 뷰에 대한 애니메이션을 유지하는 것이 합리적이지 않을 수 있습니다.

뷰가 없는 애니메이션은 의미가 없기 때문에 뷰가 해제된 후에는 animations 와 completion 블럭 안의 뷰에 대해서 캡쳐리스트 [weak self] 를 사용하는 것은 좋은 옵션이 됩니다.

***단, [unowned self] 를 사용하는 경우 completion handler 가 호출되기 전에 할당 해제되는 경우 충돌이 발생할 수 있습니다.***

<img src="https://user-images.githubusercontent.com/69136340/185614380-ccf349e8-f310-4a6f-81cf-8b30a00162af.gif" width ="400">


### 왜 [unowned self] 만 충돌이 일어나나요?

 unowned(미소유 참조)는 약한참조처럼 인스턴스의 reference count(참조 횟수)를 증가시키지 않습니다.

대신 참조하는 인스턴스가 항상 메모리에 존재할 것이라는 전제를 기반으로 동작합니다. 즉, 인스턴스가 메모리에서 해제되더라도 스스로 nil 을 할당해주지 않습니다.

그렇기 때문에 옵셔널 혹은 변수가 아니어도 되지만, **사실상 nil 이 되어버린 인스턴스를 completion handler 에서 참조하려고 하기 때문에 위의 예시처럼 화면전환을 통해 인스턴스가 해제될 때 충돌이 발생합니다.**

### strong 은요?

그렇다면 강한 참조에 대해서는 왜 충돌은 일어나지 않나요?

위의 예시에서는 로그를 확인하면 yellow completion 까지 호출이 된것을 보아 정상적으로 애니메이션이 끝난 뒤에 화면전환이 이루어진 것으로 보여요.

예를 들어 완료되지 않고 화면이 전환되었다는 가정하에 생각을 해봅시다.

할당해제가 된 인스턴스에 접근하는 것이 아니라 반대로 메모리에 더 이상 사용하지 않는 인스턴스가 남아있게 됩니다.(강한 참조 순환 발생) 즉, **충돌은 일어나지 않지만 메모리 릭이 발생하게 됩니다.**

### 참고

[unowned self vs weak self (캡쳐 리스트)-(1)](https://jinswift.tistory.com/6)](https://jinswift.tistory.com/6)

[Is it necessary to use [unowned self] in closures of UIView.animateWithDuration(...)?](https://stackoverflow.com/questions/27019676/is-it-necessary-to-use-unowned-self-in-closures-of-uiview-animatewithduration)
