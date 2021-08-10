---
title:  "iOS) Custom UIPresentationController 와 UIViewControllerAnimatedTransitioning 를 사용해서 화면전환 해보자"
categories:
- iOS

date:   2021-08-10  15:48:00 +0900
author_profile: false
---
[iOS) VerticalCardSwiper 오픈라이브러리를 알아보자](https://gyuios.tistory.com/73)

[iOS) UIPresentationController 를 알아보고 App Store clone app 을 살펴보자](https://gyuios.tistory.com/74)

위의 두가지 공부를 하고 `UIPresentationController` 를 custom 해서 적용해보고 싶어졌고 VerticalCardSwiper 를 사용한 프로젝트와 접목시켜 보았다. 

- dismiss 시 모습은 CardCell 의 도넛 이미지와 DetailCardCollectionViewCell 의 도넛이미지의 레이아웃 크기의 차이와 불투명 뷰가 있어서 좀 어색하지만 잘 적용되었다.

<img src ="https://user-images.githubusercontent.com/69136340/128821558-2c130093-c2a6-417e-adbe-2a3aeaae4843.gif" width ="250">

클론앱을 분석하고 난 후)

`Uipresentation` 을 사용하는 예제는 블러처리를 함으로써 확인을 할 수 있었다.

<img src = "https://user-images.githubusercontent.com/69136340/128819537-a507e413-2915-4bfe-8f9b-dc45cc8fba8c.gif" width ="250">

블러처리가 된 뷰

<img src ="https://user-images.githubusercontent.com/69136340/128821544-dc03817c-7891-4bd5-a3a3-a3dac975ae07.png" width ="250">


<img src ="https://user-images.githubusercontent.com/69136340/128821552-3decdd63-ed42-4bde-867c-df75e834f1ad.png" width ="400">

- 사실 이 클론앱을 분석하면서 `UIPresentationController` 의 사용은 굳이 필요는 없었다는 것을 알았다. `UIPresentationContorller` 를 `nil` 로 주면 앱이 자동으로 설정하는데 보이는 사진처럼 블러처리 없이 자동으로 presentationStyle 이 적용되었다.

<img src ="https://user-images.githubusercontent.com/69136340/128821715-e4a4a1db-8f7c-4ebf-8cc6-e8539e65c722.png" width ="400">

- 정작 내가 원했던 화면이 확대되고 축소되는 애니메이션은 `UIViewControllerAnimatedTransitioning` 를 활용해서 구현할 수 있었다.

---

- `UIViewControllerTransitioningDelegate` 채택해서 present, dismiss 시 custom 한 `UIViewControllerAnimatedTransitioning` 를 리턴해주었다.

```swift
// MARK: - UIViewControllerTransitioningDelegate

extension DetailCardViewController: UIViewControllerTransitioningDelegate {
    func animationController(forPresented presented: UIViewController, presenting: UIViewController, source: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        return PopAnimator(animationType: .present)
    }
    
    func animationController(forDismissed dismissed: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        return PopAnimator(animationType: .dismiss)
    }

    // custom presentation controller 설정.
    func presentationController(forPresented presented: UIViewController, presenting: UIViewController?, source: UIViewController) -> UIPresentationController? {
        return CustomPresentationController(presentedViewController: presented, presenting: presenting)
    }
}
```

### PopAnimator

: 카드셀 선택 시 이미지가 확대되면서 present, dismiss 시 이미지가 셀의 위치로 축소되는 커스텀 화면전환 애니메이션 구현.

- ViewController : UITabBarController 의 첫번째 뷰컨트롤러.
- DetailCardViewController : ViewController 에서 카드셀 선택 시 확대되면서 present 시킬 뷰.

```swift
import UIKit

fileprivate let transitonDuration: TimeInterval = 1.0

// ✅ DetailCardViewController 에 대한 present 와 dismiss 애니메이션을 동시에 관리하는 하기 위한 열거형. 
enum AnimationType {
    case present
    case dismiss
}

class PopAnimator: NSObject {
    let animationType: AnimationType
    
    init(animationType: AnimationType) {
        self.animationType = animationType
        super.init()
    }
}

// MARK: - UIViewControllerAnimatedTransitioning
// ✅ 뷰컨트롤러를 전환할 때 커스텀으로 애니메이션을 구현하기 위한 메소드 세트.
extension PopAnimator: UIViewControllerAnimatedTransitioning {
        // ✅ animator 개체에게 transition animation 의 duration 을 묻는다.(초단위)
    func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval {
        return transitonDuration
    }
        // ✅ transition animations 수행하도록 animator 개체에게 지시한다.
    func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
                // ✅ transitionContext : transition 에 대한 정보를 포함한 context 개체.
                if animationType == .present {
            animationForPresent(using: transitionContext)
        } else {
            animationForDismiss(using: transitionContext)
        }
    }
}

// MARK: - Extensions

extension PopAnimator {

        **//** ✅ present 시 사용할 transition animations.
    func animationForPresent(using transitionContext: UIViewControllerContextTransitioning) {
        
                // ✅ 애니메이션이 발생할 containerView 를 가져온다.
                let containerView = transitionContext.containerView
                
                // ✅ transition 과 관련된 뷰를 키값을 통해 가져온다.(.form, .to)
                //1.Get fromVC and toVC
                // ✅ .from : transition 이 시작할때 혹은 취소된 전환이 끝날 때 표시되는 뷰컨트롤러를 식별하는 키.(=가려질 뷰컨트롤러)
        guard let fromVC = transitionContext.viewController(forKey: .from) as? UITabBarController else { return }
        guard let mainVC = fromVC.viewControllers?.first as? ViewController else { return }
                
                // ✅ .to : 완료된 전환이 끝날 때 표시되는 뷰컨트롤러를 식별하는 키.(=보여질 뷰컨트롤러)
                guard let toVC = transitionContext.viewController(forKey: .to) as? DetailCardViewController else { return }
        
                // ✅ selectedCell : ViewController 에서 선택된 card cell 개체를 저장한 변수.     
                guard let selectedCell = mainVC.selectedCell else { return }
        
                // ✅ 첫번째 파라미터에서 to(=UIView) 로 CGRect 을 변환. 즉, 좌표계 설정을 to 뷰로 변환한다.
                // ✅ 이 부분이 핵심. 카드셀에서 present 하는 것이 아닌 fromVC(=UITabBarViewController)의 좌표계 중 카드셀의 좌표에 present 되야하기 때문에 convert 가 필요하다.
                // ✅ 즉, 선택된 카드셀의 bgview(=이미지) frame 과 같은 곳에서 present 가 시작되게 하기 위한 convert.
                let frame = selectedCell.convert(selectedCell.bgView.frame, to: fromVC.view)
        
                //2.Set presentation origin al size.
                toVC.view.frame = frame
                toVC.view.layer.cornerRadius = 10
        
                toVC.scrollView.imageView.frame.size.width = UIScreen.main.bounds.width - 40
                toVC.scrollView.imageView.frame.size.height = 500
        
                containerView.addSubview(toVC.view)
        
                //3.Change original size to final size with animation.
                UIView.animate(withDuration: transitonDuration, delay: 0, usingSpringWithDamping: 0.5, initialSpringVelocity: 0, options: [], animations: {
                        // ✅ toVC 의 frame(기존에는 선택된 card cell 의 bgView frame 이었다.)를 화면전체로 설정. 자연스럽게 확대되는 애니메이션을 구현.
                        toVC.view.frame = UIScreen.main.bounds
                        toVC.view.layer.cornerRadius = 10
                        toVC.scrollView.imageView.frame.size.width = UIScreen.main.bounds.width
                        toVC.scrollView.imageView.frame.size.height = 500
                        
                        // ✅ dismiss 역할을 하는 closeBtn 을 보이게 한다.
                        toVC.closeBtn.alpha = 1

                        // ✅ 하단 탭바의 y좌표를 화면 아래로 설정해서 자연스럽게 아래로 사라지도록 함.
                        fromVC.tabBar.frame.origin.y = UIScreen.main.bounds.height
                }) { completed in
                        // ✅ transition 이 끝났음을 알린다.
                        transitionContext.completeTransition(completed)
                }
        }
    
        **//** ✅ dimiss 시 사용할 transition animations.
        func animationForDismiss(using transitionContext: UIViewControllerContextTransitioning) {
                guard let fromVC = transitionContext.viewController(forKey: .from) as? DetailCardViewController else { return }
                guard let toVC = transitionContext.viewController(forKey: .to) as? UITabBarController else { return }
                guard let mainVC = toVC.viewControllers?.first as? ViewController else { return }
                guard let selectedCell = mainVC.selectedCell else { return }
        
                UIView.animate(withDuration: transitonDuration / 2, delay: 0, usingSpringWithDamping: 1, initialSpringVelocity: 0, options: [], animations: {
                        let frame = selectedCell.convert(selectedCell.bgView.frame, to: fromVC.view)

                        fromVC.view.frame = frame
                        fromVC.view.layer.cornerRadius = 10
                        fromVC.scrollView.imageView.frame.size.width = UIScreen.main.bounds.width - 40
                        fromVC.scrollView.imageView.frame.size.height = 500
                        fromVC.closeBtn.alpha = 0
                        // ✅ 화면 아래로 설정되어 있던 하단 탭바의 y좌표를 화면 위로 올려 자연스럽게 나타나도록 함.
                        toVC.tabBar.frame.origin.y = UIScreen.main.bounds.height - toVC.tabBar.frame.height
                }) { completed in
                        transitionContext.completeTransition(completed)
                }
        }
}
```

### 생각해볼 점

view controller 에서 셀 선택 하고 presnet 시

```swift
detailVC.modalPresentationStyle = .custom
detailVC.transitioningDelegate = **self**
```

와 같이 설정해주면 **스토리보드로 DetailCardViewController 를 만들어도 되었긴 했었다.** 

내가 참고했던 예제에서는 mvc 패턴느낌으로 진행했었기 때문에 나도 한번해보았다. 참고한 예제도 링크를 달아두겠다.

[iOS Animation Tutorial: Custom View Controller Presentation Transitions](https://www.raywenderlich.com/2925473-ios-animation-tutorial-custom-view-controller-presentation-transitions)

---

**전체 코드:**

[GitHub - hyun99999/VerticalcardswiperTutorial-iOS: 📇 VerticalcardswiperTutorial](https://github.com/hyun99999/VerticalcardswiperTutorial-iOS)
