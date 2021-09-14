---
title:  "iOS) Kakao QRcode Widget 클론코딩 - FaceID 적용"
categories:
- iOS

date:   2021-09-14  10:00:00 +0900
author_profile: false
---
애플의 Face ID 를 활용한 샘플 프로젝트와 개발자 문서를 정리해봤다. 

[iOS) Face ID & Touch ID - Biometrics Authentication(생체인식 인증)](https://gyuios.tistory.com/104)

자, 이제 카카오톡 QR코드 위젯에 적용해보자.

### 내용

- 앱 접근 시 Face ID 를 통해서 인증을 필요로 한다.

<img width="600" alt="스크린샷 2021-09-14 오전 9 57 10" src="https://user-images.githubusercontent.com/69136340/133176483-007203a4-ba71-403a-8d26-0c98c1974212.png">

# 시작하기

## 🔓 프로젝트 설정

<img width="888" alt="4" src="https://user-images.githubusercontent.com/69136340/133176388-64977ee6-067e-46cb-b15f-b4d3f6c5f81f.png">

## 🔓 UI 구성

- 기기에서 Face ID 를 지원하면 Face ID 버튼을 보여줌.

## 🔓 Face ID 적용

mvvm 패턴에서 Face ID 인증 절차를 가지는 Service 클래스를 만들고 로직을 구성하였다.

- FaceIDAuthenticationViewController

```swift
private func setFaceIDAuthentication() {
        faceIDButton.isHidden = service.checkBiometryTypeFaceID()
        service.loginWithFaceID()
}
```

- FaceIDAuthenticationService

```swift
import Foundation
import LocalAuthentication

class FaceIDAuthenticationService {
    
    var context = LAContext()
    
    enum AuthenticationState {
        case loggedin, loggedout
    }
    
    var state = AuthenticationState.loggedout
    
    // ✅ 장비가 Face ID 가능한지 묻는 것
    func checkBiometryTypeFaceID() -> Bool {
        return context.biometryType == .faceID
    }
    
    func loginWithFaceID() {
        context.localizedCancelTitle = "Enter Username/Password"

        var error: NSError?
        if context.canEvaluatePolicy(.deviceOwnerAuthentication, error: &error) {

            let reason = "Log in to your account"
            context.evaluatePolicy(.deviceOwnerAuthentication, localizedReason: reason ) { success, error in

                if success {

                    // Move to the main thread because a state update triggers UI changes.
                    DispatchQueue.main.async { [unowned self] in
                        self.state = .loggedin
                    }
                } else {
                    print(error?.localizedDescription ?? "Failed to authenticate")
                }
            }
        } else {
            print(error?.localizedDescription ?? "Can't evaluate policy")
        }
    }
}
```

### 결과

<img src ="https://user-images.githubusercontent.com/69136340/133176342-2b3df54c-1937-47b1-9e4f-1d3a3f300b33.gif" width = "250">

### 깃허브

[GitHub - 28th-SOPT-iOS-CloneCoding/MiraClone-KimHyunGyu: 🧚 아요 미라클론코딩 김현규](https://github.com/28th-SOPT-iOS-CloneCoding/MiraClone-KimHyunGyu)
