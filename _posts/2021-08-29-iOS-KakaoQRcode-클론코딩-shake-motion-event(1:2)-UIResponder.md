---
title:  "iOS) KakaoQRcode 클론코딩 - shake motion event(1/2) - UIResponder"
categories:
- iOS

date:   2021-08-29  22:32:00 +0900
author_profile: false
---
### 내용

-   카카오톡에서 두번 흔들면 qr 코드 뷰를 띄어주는 기능 클론코딩

---

두가지 방법에 대해서 알아볼 것이다.

1.  UIResponder 의 motion event 메서드를 재정의해서 사용
2.  CoreMotion Framework 사용

## 1️⃣ UIResponder 의 motion event 메서드를 재정의해서 사용

# 시작 전

## 👏 원리

-   motion capture delegate 에서 shake gesture 를 capture 해서 사용.

## 👏 [UIResponder](https://developer.apple.com/documentation/uikit/uiresponder)

이벤트에 응답하고 처리하기 위한 추상 인터페이스입니다.

**overall**

이벤트가 발생하면 UIKit 은 처리를 위해서 UIResponder 의 인스턴스에게 보내지게 된다.

touch events, motion events, remote-control events, press events 을 비롯한 여러종류의 이벤트가 있다. 특정유형의 이벤트를 처리하기 위해서는 responder 가 해당 메서드를 재정의 해야한다.

예를 들어, 터치 이벤트를 처리하기 위해 응답자는 _touchBegan(:with:), touchMoved(_:with:), _touchEnded(:with:) 및 touchCancelled(_:with:) 메소드를 구현합니다.

다음은 모션 이벤트의 경우 재정의할 메서드이다.

**motion event**

-   [func motionBegan(\_ motion: UIEvent.EventSubtype, with event: UIEvent?)](https://developer.apple.com/documentation/uikit/uiresponder/1621120-motionbegan)

: Tells the receiver that a motion event has begun.

-   [func motionEnded(\_ motion: UIEvent.EventSubtype, with event: UIEvent?)](https://developer.apple.com/documentation/uikit/uiresponder/1621090-motionended)

: Tells the receiver that a motion event has ended.

-   [func motionCancelled(\_ motion: UIEvent.EventSubtype, with event: UIEvent?)](https://developer.apple.com/documentation/uikit/uiresponder/1621087-motioncancelled)

: Tells the receiver that a motion event has been cancelled.

### motionBegan(_:with:) & motionEnded(_:with:)

UIKit은 모션 이벤트가 시작되고 끝날 때만 responder 에게 알립니다. 중간 흔들림은 보고하지 않습니다. 모션 이벤트는 처음에 첫 번째 responder 에게 전달되고 적절하게 responder chain 위로 전달됩니다.

### motionCancelled(\_:with:)

motion evnet 가 중단될 때 호출된다. 애플리케이션이 비활성화되거나 모션 이벤트를 처리하는 뷰가 window 에서 제거되도록 하는 모든 것이 이에 해당한다.

흔들림이 너무 오래 지속되면 UIKit 이 메서드를 호출할 수 있다. 모든 responder 는 이 메서드를 구현해야한다.

## 👏 UIEvent.EventSubtype

[Apple Developer Documentation](https://developer.apple.com/documentation/uikit/uievent/eventsubtype)

위의 motion event 와 관련된 일반적인 모션은 [UIEvent.EventSubtype.motionShake](https://developer.apple.com/documentation/uikit/uievent/eventsubtype/motionshake) 로 표시되는 흔들림이다.

# 시작하기

### motion event 를 인식해서 화면전환해보자

-   MainViewController.swift

쉐이크를 인식하는 메인 뷰컨트롤러에 해당

```
import UIKit
import SnapKit

class MainViewController: UIViewController {

    // ...

    // MARK: - View Life Cycle

    override func viewDidLoad() {
        super.viewDidLoad()
        configUI()
        setLayout()

        // ✅ UIKit 이 이 object 를 window 에서 first responder 로 만들도록 한다.
        becomeFirstResponder()
    }
}

// MARK: - Extensions

extension MainViewController {

    private func configUI(){
        // ...
    }

    private func setLayout() {
        // ...
    }

    // ✅ responder 는 자신이 first responder 가 될 수 있게 하기 위해서 canBecomeFirstResponder 프로퍼티를 오버라이드해서 ture 를 리턴하도록 만들어야 한다.
    // ✅ UIView 는 UIResponder 의 상속을 받기 때문에 아래와 같이 재정의할 수 있는 이유가 된다.
    override var canBecomeFirstResponder: Bool {
        return true
    }

    @objc
    private func presentToQRCodeVC() {
        let nextVC = QRCodeViewController()
        nextVC.modalPresentationStyle = .overFullScreen
        present(nextVC, animated: true, completion: nil)
    }

    // ✅ **shake motion detect with UIResponder**
    override func motionBegan(_ motion: UIEvent.EventSubtype, with event: UIEvent?) {
        if motion == .motionShake {
            print("shake began")
            presentToQRCodeVC()
        }

        // ✅ 조건문을 사용해서 motion 을 비교하지 않고 아래의 코드처럼 사용해도 괜찮다. 일반적으로 사용할 때는 motionShake 에 해당되기 때문이다.
        // print("shake began")
        // presentToQRCodeVC()
    }

    override func motionEnded(_ motion: UIEvent.EventSubtype, with event: UIEvent?) {
        if motion == .motionShake {
            print("shake ended")
        }
    }

    override func motionCancelled(_ motion: UIEvent.EventSubtype, with event: UIEvent?) {
        if motion == .motionShake {
            print("shake cancelled")
        }
    }
}
```

위의 코드를 통해서 디바이스를 흔들면 올바르게 해당 메서드가 호출되는 것을 확인해볼 수 있다.

### ❗️**motion event 가 두번 연속 발생하면 화면전환해보자**

motionEnded(\_:with) 메서드를 다음과 같이 수정하면서 구현해보았다.

```
override func motionEnded(_ motion: UIEvent.EventSubtype, with event: UIEvent?) {
        if motion == .motionShake {
            print("shake ended")

            motionNumber += 1
            print("motionNumber: \(motionNumber)")

            if motionNumber == 2 {
                motionNumber = 0
                presentToQRCodeVC()
            }
        }
    }
```

### 아쉬운 점

motion 의 횟수에 대한 변수를 설정하고 2번 흔들림이 발생하면 present 하고 싶었는데 위의 `motionBegan(_:with:)` , `motionEnded(_:with:)` 메서드를 보면 알다시피 중간 흔들림에 대해서는 보고하지 않는다.

그래서 연속으로 두번을 흔들지 않아도 횟수가 누적되면 실행되도록 구현하였다.

### 출처 :

[iOS ShakeGesture](https://blog.lenskart.com/ios-shakegesture-ecece76d88cf)

[\[iOS\] motion gesture 감지하기](https://sujinnaljin.medium.com/ios-motion-gesture-%EA%B0%90%EC%A7%80%ED%95%98%EA%B8%B0-dd94a75411b6)

[iOS의 Responder와 Responder Chain 이해하기](https://seizze.github.io/2019/11/26/iOS%EC%9D%98-Responder%EC%99%80-Responder-Chain-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0.html)

### 깃허브 :

[GitHub - 28th-SOPT-iOS-CloneCoding/MiraClone-KimHyunGyu: 🧚 아요 미라클론코딩 김현규](https://github.com/28th-SOPT-iOS-CloneCoding/MiraClone-KimHyunGyu)
