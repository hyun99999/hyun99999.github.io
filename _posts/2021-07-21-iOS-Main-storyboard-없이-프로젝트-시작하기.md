---
title:  "iOS) Main.storyboard 없이 프로젝트 시작하기"
categories:
- iOS

date:   2021-07-21  23:48:00 +0900
author_profile: false
---
협업 시 info.plist 에서 시작 스토리보드를 설정하다보니 info.plist 파일에서 충돌이 나는 것이 위험이 있다고 생각해서 코드로 시작 스토리보드를 변경하도록 해보았다.

첫번째, 프로젝트 파일의 Main Interface 를 삭제한다.

<img width="400" alt="스크린샷 2021-07-22 오전 12 07 23" src="https://user-images.githubusercontent.com/69136340/126581749-d0602f54-32b6-48fa-b3bd-b80cd27a502e.png">

두번째, info.plist 파일의 Application Scene Manifest 를 펼쳐서 Storyboard Name 을 삭제해준다.

<img width="600" alt="스크린샷 2021-07-22 오전 12 07 59" src="https://user-images.githubusercontent.com/69136340/126581757-25931209-18a9-4df2-8652-ce637fe3899c.png">

세번째, SceneDelegate.swift 파일에 다음의 코드를 추가해준다.

```swift
func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {

    guard let windowScene = (scene as? UIWindowScene) else { return }
    
    window = UIWindow(frame: windowScene.coordinateSpace.bounds)
    window?.windowScene = windowScene
    window?.rootViewController = UIStoryboard(name: Const.Storyboard.Name.Splash, bundle: nil).instantiateViewController(withIdentifier: Const.ViewController.Name.Splash)
    window?.makeKeyAndVisible()
}
```
