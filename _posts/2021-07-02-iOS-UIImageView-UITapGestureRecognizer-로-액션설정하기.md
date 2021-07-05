---
title:  "iOS) UIImageView UITapGestureRecognizer 로 액션설정하기"
categories:
- iOS

date:   2021-07-02 19:12:00 +0900
author_profile: false
---
UIImageView UITapGestureRecognizer 로 액션설정하기

```swift
// MARK: - @IBOutlet Properties
@IBOutlet var loginImageView: UIImageView!

//...

// MARK: - Methods
func loginTapAction() {
    let login = UITapGestureRecognizer(target: self, action: #selector(login))
    loginImageView.isUserInteractionEnabled = true
    loginImageView.addGestureRecognizer(login)
}

// MARK: - @objc Methods
@objc
func login() {
    guard let nextVC = UIStoryboard(name: Const.Storyboard.Name.FirstOnboarding, bundle: nil).instantiateViewController(withIdentifier: Const.ViewController.Name.FirstOnboarding) as? FirstOnboardingVC else {
        return
    }
    
    self.navigationController?.pushViewController(nextVC, animated: true)
}
```
