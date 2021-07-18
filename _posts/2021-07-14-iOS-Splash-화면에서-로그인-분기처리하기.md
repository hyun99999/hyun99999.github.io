---
title:  "iOS)  Splash 화면에서 로그인 분기처리하기"
categories:
- iOS

date:   2021-07-14  19:42:00 +0900
author_profile: false
---
Splash 화면 만들기

<p align = "center"><img width="800" alt="스크린샷 2021-07-14 오후 7 31 48" src="https://user-images.githubusercontent.com/69136340/125607890-4f28bb40-9704-490c-afd9-4cab33ed6d4a.png"></p>

하단 출처의 개발자문서에서 애플은 첫번째 화면을 launch screen 과 흡사하게 만들라고 하고 있다.

- SplashViewController 에서 로그인 분기처리를 하기 위해서 Launch Screen 과 동일한 Splash 스토리보드를 만들었다.

<p align = "center">
<img width="400" alt="스크린샷 2021-07-14 오후 7 35 09" src="https://user-images.githubusercontent.com/69136340/125608340-a8313a1f-6661-422b-a98f-13a7e1d31bdb.png">

<img width="400" alt="스크린샷 2021-07-14 오후 7 35 19" src="https://user-images.githubusercontent.com/69136340/125608356-4d844b3d-127f-4934-a0c3-a2534bc88f07.png">
</p>

### 로그인 분기처리
- AppDelegate.swift 에서 로그인 유무를 판단할 수 있는 변수를 생성해두고 앱이 실행되면 application(_:didFinishLaunchingWithOptions:) 메서드 안에서 분기처리해서 변수를 바꾸어주었다.

- SplashVC

```swift
// MARK: - Properties
// AppDelegate 에서 생성한 로그인 유무 변수
private let appDelegate = UIApplication.shared.delegate as? AppDelegate

// viewDidAppear() 에서 화면전환을 해주어야한다.
override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
    
    // 딜레이 후 화면 전환
    DispatchQueue.main.asyncAfter(deadline: DispatchTime.now() + 0.5) {
        self.setIsLogin()
    }
    
}

// MARK: - Methods
private func setIsLogin() {
    if appDelegate?.isLogin == true {
        presentToHome()
    } else {
        presentToLogin()
    }
}

private func presentToHome(){
    //뷰전환 코드
}

private func presentToLogin() {
    //뷰전환 코드
}
```

### 출처
[Human Interface Guidelines-Launch Screen](https://developer.apple.com/design/human-interface-guidelines/ios/visual-design/launch-screen/)
