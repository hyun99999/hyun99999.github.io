---
title:  "iOS) KakaoQRcode 클론코딩 - 화면캡처 block"
categories:
- iOS

date:   2021-09-01  13:54:00 +0900
author_profile: false
---
### 내용

- 카카오톡 QR코드 위젯을 통해서 접근하거나 카카오톡에서 쉐이크 모션을 통해 접근할 수 있는 QR코드 뷰의 화면캡처와 기록, 미러링, AirPlay 를 막고있었다. 구현해보자.

먼저 카카오톡 QR코드 뷰가 어떻게 작동하는지 살펴보자.

<img src ="https://user-images.githubusercontent.com/69136340/131613048-c674e0dc-dd2c-4aa9-a543-e410261b0e54.png" width ="250">

QR코드 뷰 화면캡처를 시도하자 빈화면이 캡쳐되었다.

<img src ="https://user-images.githubusercontent.com/69136340/131613034-e21f18ad-3dfe-4f8d-8976-15f635eb4a4b.png" width ="250">

그리고 경고메시지가 등장했다. 이것은 화면캡처가 되서 첨부해본다. 물론 기기에서는 경고 메시지 뒤에 QR 코드 뷰가 있다.

### 목표

- 화면캡쳐 시 alert 창 등장
- 화면캡쳐 결과에 관여해서 빈 화면이 캡쳐되도록하기

# 시작 전

## 📸 원리

## 📌 [UIApplication.userDidTakeScreenshotNotification](https://developer.apple.com/documentation/uikit/uiapplication/1622966-userdidtakescreenshotnotificatio)

screenshot 할 때 notification 이 post.

### Discussion

notification 에 userInfo dictionary 가 없다. (즉, notification 를 받는 추가 object 가 없다는 뜻.)

notificaation 이 screenshot 을 찍은 후에 post 된다.

## 📌 [UIScreen.capturedDidChangeNotification](https://developer.apple.com/documentation/uikit/uiscreen/2921652-captureddidchangenotification)

캡처된 screen 상태가 변경될 때 sent 되는 notification.

### Discussion

화면의 내용을 기록, 미러링, AirPlay 를 통해 전송하거나 다른 곳으로 복제할 수 있다. UIKit 은 이러한 화면 캡쳐 상태가 변경될 때 이 알림을 보낸다.

notification 의 object 는 [isCaptured](https://developer.apple.com/documentation/uikit/uiscreen/2921651-iscaptured) 프로퍼티가(YES 값은 상태가 변경됨을 의미) 변경된 UIScreen object 이다. userInfo dictionary 가 없다.

---

## 🤨 흠..

screenshot 과 관련된 프로퍼티를 알아보았다. 찾아보니 아이폰자체에서 스크린샷을 막을 수 없었다. 그리고 이름에서부터 알겠지만  이미 캡쳐가 시작되고 끝날 때 notification 을 보내고 있다.

**위의 프로퍼티를 사용해서 screenshot 시 동작을 처리해보자.**

# 시작하기

## 📸 화면캡쳐 시 alert 창 등장

- MainViewController.swift

```swift
class QRCodeViewController: UIViewController {

// ...
    
    // MARK: - View Life Cycle
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // ...

        setNotification()
    }
}

extension QRCodeViewController {

// ...

    private func setNotification() {
        // ✅ UIApplication.userDidTakeScreenshotNotification : 유저가 스크린샷을 찍을 때 notification 이 post 한다.
        NotificationCenter.default.addObserver(self, selector: #selector(blockScreenShot), name: UIApplication.userDidTakeScreenshotNotification, object: nil)

        // ✅ UIScreen.capturedDidChangeNotification
        NotificationCenter.default.addObserver(self, selector: #selector(blockScreenShot), name: UIScreen.capturedDidChangeNotification, object: nil)    
    }
    
    @objc
    func blockScreenShot() {

        // ✅ alert 창 생성
        let alert = UIAlertController(title: "캡쳐는 안돼요!", message: "보안 정책에 따라 스크린샷을 캡쳐할 수 없습니다.", preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "확인", style: .default, handler: nil))
        self.present(alert, animated: true, completion: nil)
    }
}
```

- 화면캡처를 시도하거나 레코딩하려는 시도를 하면 alert 창이 뜨도록 하였다.

<img src ="https://user-images.githubusercontent.com/69136340/131613025-4160f910-40f6-4685-8a78-085d96552ad7.png" width ="250">

## 📸 화면캡쳐 결과에 관여해서 빈 화면이 캡쳐되도록하기

흠.. 좀 더 찾아봐야할 것 같다.

### 출처 :

### 깃허브

[GitHub - 28th-SOPT-iOS-CloneCoding/MiraClone-KimHyunGyu: 🧚 아요 미라클론코딩 김현규](https://github.com/28th-SOPT-iOS-CloneCoding/MiraClone-KimHyunGyu)
