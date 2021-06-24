---
title:  "iOS) 스와이프해서 pop 하기"
categories:
- iOS

date:   2021-06-03 22:16:00 +0900
author_profile: false
---
## interactivePopGestureRecognizer

- navigationBar 를 숨기지 않은 상태라면 좌측에서 우측으로 스와이프하면 push 된 뷰컨이 pop 된다. 하지만 다음과 같이 숨김 상태라면

```swift
navigationController?.navigationBar.isHidden = true
```
스와이프해도 pop 되지 않는다.

이때 interactivePopGestureRecognizer 를 사용하면 pop 할 수 있다,

```swift
override func viewDidLoad() {
    super.viewDidLoad()

    self.delegate = self
    self.dataSource = self
    if let firstVC = vcArray.first {
        setViewControllers([firstVC], direction: .forward, animated: true, completion: nil)
    }
    navigationController?.interactivePopGestureRecognizer?.isEnabled = true
    navigationController?.interactivePopGestureRecognizer?.delegate = self
}

//...

extension ViewController: UIGestureRecognizerDelegate {
    func gestureRecognizer(_ gestureRecognizer: UIGestureRecognizer, shouldRequireFailureOf otherGestureRecognizer: UIGestureRecognizer) -> Bool {
        return true
    }
}
```

- `gestureRecognizer(_:shouldBeRequiredToFailBy:)` 는 `gestureRecognizer` 가 `otherGestureRecognizer` 에 실패해야하는지 묻는 delegate 함수이다.

**Parameters**
- gestureRecognizer : 추상 베이스 클래스 UIGestureRecognizer 의 subclass 인스턴스입니다. delegate 에 메시지를 보내는 객체

- otherGestureRecognizer : 추상 베이스 클래스 UIGestureRecognizer 의 subclass 인스턴스입니다. 

**Return Value**

`true` 는 제스쳐인식기간의 동적 실패 요구사항을 설정한다. `false` 는 기본값이다. `gestureRecognizer` 가 `otherGestureRecognizer` 에 의해서 실패함이 요구되지 않는다. 

- `true` 반환하면 실패 요구사항이 설정된다. `false` 를 반환하면 `otherGestureRecognizer` 가 자체 하위 클래스 또는 델리게이트 메서드를 사용해서 자체적 실패 요구사항을 만들 수 있기 때문에 실패요구사항을 방지하거나 제거를 보장할 수 없다.

즉, 두개의 gestureRecognizer 가 인식되었을 때 위의 메서드를 재정의해서 원하는 제스처를 인식할 수 있도록 한다. otherGestureRecognizer 가 인식된다.

### 출처
출처: https://shoveller.tistory.com/116
