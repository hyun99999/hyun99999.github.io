---
title:  "iOS) UIView 에 transition animation 적용해보기"
categories:
- iOS

date:   2021-08-20  13:23:00 +0900
author_profile: false
---
### UIView transition animation options

먼저 개발자문서를 살펴보자
[transition(with:duration:options:animations:completion:)](https://developer.apple.com/documentation/uikit/uiview/1622574-transition)

- 특정 container 뷰의 transition animation 을 만드는 함수이다.

animation 효과를 주기 위해서 파라미터 options 에 해당하는 `UIView.AnimationOptions` 옵션을 설정해주면 된다.

### 준비

- Main.storyboard

<img src ="https://user-images.githubusercontent.com/69136340/130177903-9132cd8e-1533-48d0-a838-48d2509b8e60.png" width ="500">


- ViewController.swift

```swift
import UIKit

class ViewController: UIViewController {

    private var isInitialImage = true
    
    @IBOutlet weak var initialImageView: UIImageView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        initialImageView.image = UIImage(named: "initialImg")
        initialImageView.contentMode = .scaleAspectFill
    }

    @IBAction func touchTransitionButton(_ sender: Any) {
        print("transitionButton touched.")
        
        // ✅ 앞면의 경우
        if isInitialImage {
            self.isInitialImage = false
            initialImageView.image = UIImage(named: "transitionImg")
            
            // ✅ options 파라미터에 원하는 효과에 해당하는 옵션을 넣어주며 된다.
            UIView.transition(with: initialImageView, duration: 1, options:.transitionFlipFromLeft, animations: nil, completion: nil)
        }
        // ✅ 뒷면의 경우
        else {
            self.isInitialImage = true
            initialImageView.image = UIImage(named: "initialImg")
            UIView.transition(with: initialImageView, duration: 1, options: .transitionFlipFromLeft, animations: nil, completion: nil)
        }
    }
}
```

### UIView.AnimationOptions constants. 
transition animation 형태에 직접적인 옵션들을 실습해보았다.

- [transitionCrossDissolve](https://developer.apple.com/documentation/uikit/uiview/animationoptions/1622499-transitioncrossdissolve)

<img src ="https://user-images.githubusercontent.com/69136340/130177363-8bd4c50c-41e2-47e8-a3ec-5c63f2dd7a08.gif" width ="250">

- [transitionCurlDown](https://developer.apple.com/documentation/uikit/uiview/animationoptions/1622455-transitioncurldown)

<img src ="https://user-images.githubusercontent.com/69136340/130177368-3e779e27-ce9c-48ba-8cc6-e91b9f4180e7.gif" width ="250">


- [transitionCurlUp](https://developer.apple.com/documentation/uikit/uiview/animationoptions/1622637-transitioncurlup)

<img src ="https://user-images.githubusercontent.com/69136340/130177370-c425beea-0a9b-41c4-84db-36fba0fadd6e.gif" width ="250">

- [transitionFlipFromBottom](https://developer.apple.com/documentation/uikit/uiview/animationoptions/1622632-transitionflipfrombottom)

<img src ="https://user-images.githubusercontent.com/69136340/130177451-7bc98464-7404-4f41-b396-5d99983e4423.gif" width ="250">

- [transitionFlipFromLeft]()

<img src ="https://user-images.githubusercontent.com/69136340/130177490-2215b1c7-1e85-48a2-94db-14361e3eeb79.gif" width ="250">

- [transitionFlipFromRight](https://developer.apple.com/documentation/uikit/uiview/animationoptions/1622573-transitionflipfromright)

<img src ="https://user-images.githubusercontent.com/69136340/130177508-d14058bc-5d4a-4113-8df0-7be61bcfb51d.gif" width ="250">

- [transitionFlipFromTop](https://developer.apple.com/documentation/uikit/uiview/animationoptions/1622548-transitionflipfromtop)

<img src ="https://user-images.githubusercontent.com/69136340/130177539-547c8875-094f-41d8-b215-ef9d9e561310.gif" width ="250">
