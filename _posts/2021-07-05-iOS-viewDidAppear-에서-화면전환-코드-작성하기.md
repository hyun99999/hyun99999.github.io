---
title:  "iOS) viewDidAppear() 에서 화면전환 코드 작성하기"
categories:
- iOS

date:   2021-07-05 11:14:00 +0900
author_profile: false
---
viewDidLoad() 에서 화면전환 하면 원하는 뷰 전환이 이루어지지 않았다.

<img width="800" alt="스크린샷 2021-07-04 오전 12 06 32" src="https://user-images.githubusercontent.com/69136340/124410612-d2aeec00-dd85-11eb-8d36-f4b2e97dd420.png">

-  view 가 다 보여지기 전에 화면전환을 하면 위와 같이 콘솔창에 찍힌다.

```swift
class SplashVC: UIViewController {
    
    // MARK: - View Life Cycle
    override func viewDidLoad() {
        super.viewDidLoad()
        // view 가 다 보여지기 전에 화면전환을 하면 위와 같이 콘솔창에 찍힌다.
        // 자연스럽게 뷰가 전환되지도 않는다.
        presentToLogin()
    }
    
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        // 다음과 같이 뷰가 다 나타난 후에 화면전화를 진행해야한다.
        presentToLogin()
    }
    
    // MARK: - Methods
    func presentToLogin() {
        guard let loginVC = UIStoryboard(name: Const.Storyboard.Name.Login, bundle: nil).instantiateViewController(withIdentifier: Const.ViewController.Name.Login) as? UINavigationController else {
            return
        }
        loginVC.modalPresentationStyle = .fullScreen
        loginVC.modalTransitionStyle = .crossDissolve
        self.present(loginVC, animated: true, completion: nil)

    }
}
```
