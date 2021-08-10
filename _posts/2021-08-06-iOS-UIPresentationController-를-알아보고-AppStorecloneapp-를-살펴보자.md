---
title:  "iOS) UIPresentationController 를 알아보고 App Store clone app 를 살펴보자"
categories:
- iOS

date:   2021-08-06  18:35:00 +0900
author_profile: false
---
App Store clone 프로젝트를 뜯어보다가 다음과 같은 뷰의 presenting animation 에 대해서 알고 싶어졌다. 이것말고도 다른 애니메이션 효과들도 한번 알아보자.

<img src ="https://user-images.githubusercontent.com/69136340/128520943-18a337f8-65d6-461a-aafd-49764efa8aad.gif" width = "250">

# 체크해볼 애니메이션

-   상태바 숨기기
-   셀 선택 시 축소 및 되돌아오기
-   화면전환 애니메이션
    -   탭바 숨기기
    -   이미지 확대 및 축소

---

### 🎬 셀 선택시 상태바 숨기기

-   셀 선택 시 메서드 호출

```
// TodayViewController.swift(presenting view controller 역할.)
override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        guard let cell = tableView.cellForRow(at: indexPath) as? TodayTableViewCell else { return }
        selectedCell = cell
        let detailVC = CardDetailViewController(cell: cell)

                // CardDetailViewController 에 빈 클로저를 만들고 넘겨주었다. 그리고 dismiss() 할 때, 상태바가 다시 보이게 하는데 사용했다. 
                detailVC.dismissClosure = { [weak self] in
            guard let StrongSelf = self else { return }
                        // 상태바 unhidden
            StrongSelf.updateStatusBarAndTabbarFrame(visible: true)
        }
                // 상태바 hidden
        updateStatusBarAndTabbarFrame(visible: false)

        present(detailVC, animated: true, completion: nil)        
    }
```

-   func updateStatusBarAndTabbarFrame(visible:)

```
// TodayViewController.swift(presenting view controller 역할.)
private func updateStatusBarAndTabbarFrame(visible: Bool) {
        // 파라미터에 따라서 hidden/unhidden 이 조절 가능한 메서드
        statusBarShouldBeHidden = !visible

    UIView.animate(withDuration: 0.25) {

// status bar 속성이 변경되었음을 시스템에 알린다.
    self.setNeedsStatusBarAppearanceUpdate()
        }
}
```

셀 선택 시 탭바가 사라지는 것은 제일 마지막에 알아보자!

### 🎬 셀 선택 시 축소 및 되돌아오기

```
override func tableView(_ tableView: UITableView, didHighlightRowAt indexPath: IndexPath) {
        guard let row = tableView.cellForRow(at: indexPath) as? TodayTableViewCell else { return }
        UIView.animate(withDuration: 0.1) {
            row.bgBackView.transform = CGAffineTransform(scaleX: 0.95, y: 0.95)
        }
    }

    override func tableView(_ tableView: UITableView, didUnhighlightRowAt indexPath: IndexPath) {
        guard let row = tableView.cellForRow(at: indexPath) as? TodayTableViewCell else { return }
        UIView.animate(withDuration: 0.3) {
            row.bgBackView.transform = CGAffineTransform(scaleX: 1.0, y: 1.0)
        }
    }
```

### 🎬 화면전환 시 애니메이션

이제 `UIPresentationController` 을 알아볼건데 대략적으로 미리 언급하자면 View Controller 가 화면을 표시하는 기능은 presentation controller 가 담당한다.

기본 presentation(full screen, current context 등) 을 사용한다면 presentation controller 가 자동으로 생성된다. 하지만 presentation 을 custom 하여 만든다면 presentation controller 를 직접 설정해주어야 한다. 그리고 이 presentation controller 가 transition animation 을 실행한다.

---

먼저 개발자 문서를 요약해보자

## UIPresentationController

view controller 의 presentation 과 transition animations 을 관리하는 개체

### **init(presentedViewController:presenting:)**

지정된 뷰 컨트롤러 간 전환을 위해 presentation controller 를 초기화하고 반환합니다.

### **Parameters**

`presentedViewController` : modaly 하게 present 되는 view controller(즉, present 하고싶은 뷰컨)

`presentingViewController` : 컨텐츠가 transition 의 시작점을 나타내는 view controller(즉, 이전의 뷰컨)

### **Return Value**

An initialized presentation controller object or `nil` if the presentation controller could not be initialized.

초기화된 presentation controller 객체 또는 초기화 되지 않았을 경우 `nil` .

---

### Overview

뷰컨트롤러가 보여질때부터 해제될 때까지 UIKit 은 뷰컨트롤러에 대한 presentation process 의 다양한 측면들을 presentation controller 로 관리한다. presentation controller 는 animator 개체가 제공하는 animations 위에 자체 animations 를 추가할 수 있다. 그리고 사이즈 변경에 반응하거나 onscreen 에 보여지는 뷰컨트롤러의 다양한 측면을 관리할 수 있다.

`present(_:animated:completion:)` 메서드를 사용해서 뷰컨트롤러를 보여줄 때 UIKit 은 항상 presentation process 를 관리한다. process 의 일부는 주어진 presntation style 에 적합한 presentation controller 를 만드는 것을 포함한다. 기본제공 스타일(UIModalPresentationStyle.pageSheet 스타일 같은) 의 경우 UIKit 필요한 presentation controller 개체를 정의하고 생성한다. **앱이 custom presentation controller 를 제공 할 수 있을 때는 뷰컨트롤러의 `modalPresentationStyle` 속성을 custom 으로 설정했을 때이다. 너는** custom presentation controller 를 제공해서 present 중인 뷰컨트롤러 아래에 shadow view 또는 decoration views 를 추가할 수 있다. 또는 presentation 동작을 다른 방식으로 수정할 수 있다.

**너는 custom presentation controller 개체를 너의 view controller 의 [transitioning delegate](https://developer.apple.com/documentation/uikit/uiviewcontrollertransitioningdelegate) 를 통해서 제공가능하다.** UIKit 은 present 된 view contorller 가 onscreen 되어 있는 동안 presentation controller 개체를 유지한다.

### The Presentation Process

presentation controller 가 관리하는 presentation pocess 는 3단계로 나뉜다.

-   presentation phase 는 일련의 transition animations 를 통해서 새로운 뷰컨트롤러를 onsreen 으로 이동하는 작업을 포함한다.
-   management phase 는 새로운 뷰컨트롤러가 onscreen 인 동안 environment changes(device 의 회전 같은) 에 대한 응답도 포함한다.
-   dismissal phase 는 일련의 transition animations 를 통해서 새로운 뷰컨트롤러를 스크린 밖으로 이동하는 작업을 포함한다.

### Adding Custom Views to a Presentation

뷰컨트롤러가 보여질 때 UIKit 은 presentation controller 의 [`presentationTransitionWillBegin()`](%5Bhttps://developer.apple.com/documentation/uikit/uipresentationcontroller/1618330-presentationtransitionwillbegin%5D(https://developer.apple.com/documentation/uikit/uipresentationcontroller/1618330-presentationtransitionwillbegin)) 메서드를 호출한다. 이 메서드를 사용해서 view hierarchy 에 뷰를 더하고 뷰와 관련된 animations 를 설정할 수 있다. **presentation phase** 가 끝나면 UIKit 은 [`presentationTransitionDidEnd(_:)`](%5Bhttps://developer.apple.com/documentation/uikit/uipresentationcontroller/1618327-presentationtransitiondidend%5D(https://developer.apple.com/documentation/uikit/uipresentationcontroller/1618327-presentationtransitiondidend)) 메서드를 호출해서 transition 이 완료되었음을 알린다.

뷰컨트롤러가 dismiss 될 때 어떤 animations 을 구성하기 위해서 `dismissalTransitionWillBegin()` 메서드를 사용한다. view hierarchy 로부터 custom view 를 없애기 위해서 `dismissalTransitionDidEnd(_:)` 메서드를 사용한다.

**이후의 내용도 있지만 여기까지가 우리가 이해할 custom presentation contorller 을 사용하기 위한 큰 틀이다.**

---

### 📌 Presentation의 실행 단계 정리

-   presentation

1.  View Controller에서 새로운 화면을 표시하면 Transitioning Delegate에게 custom presentation controller를 요청
2.  Delegate가 controller를 리턴하면 custom presentation이 실행됨
3.  presentation controller에서 presentationTansitionWillBegin() 메소드를 실행 -> Transition에서 사용되는 CustomView를 추가하고 Animation에 필요한 속성을 설정한다.
4.  Transition이 실행되면서 화면이 실제로 전환됨. Transition(Animation)이 실행되는 동안 Presentation Controller에서 containerViewWillLayoutSubviews(), containerViewDidLayourSubviews() 메소드를 호출. 이 두개의 메소드를 통해서 커스텀 뷰의 배치를 구현한다.
5.  Transition이 완료되면 Presentation Controller에서 presentationTransitionDidEnd() 메소드가 실행되며, 여기까지 실행하면 preseneted VC가 화면에 표시되며 Management 단계로 넘어간다.

-   Management

1.  화면 회전 등 처리
2.  viewWillTransition(to:with:)메소드 호출
3.  AutoLayout을 사용한다면 따로 해줄 일은 없다.

-   Dismissal

1.  Presented View Controller를 닫으면dismissalTransitionWillBegin()메소드 호출과 함께 Dismissal 단계 시작
2.  Transition animation이 실행되는 동안 containerViewWillLayoutSubviews()와 containerViewDidLayoutSubviews()\` 메소드 실행
3.  Transition이 완료되면 Presentation Controller에서dismissalTransitionDidEnd()메소드를 마지막으로 Presentation 마무리

출처 :

[iOS) Presentation 과 Transition 그리고 Animation... (2)](https://koggy.tistory.com/4?category=922104)

---

아래의 App Store clone 앱을 살펴보자!

[GitHub - DragonTnT/appstore-clone: a clone of iOS appstore, to be continued](https://github.com/DragonTnT/appstore-clone)

앞으로 나올 코드에 있는 용어 정리 부터 분명히 해보자!

-   presentingViewController - 원래의 view controller
-   presentedViewcontroller - 새롭게 표시될 view controller
-   containerView - presentation은 containerView 안에서 일어난다. 즉 presentedViewcontroller 나 presentingViewController 모두 containerView Frame 안에서 동작하는 것. transition, presentation 이 일어나는 동안에 superView 역할을 한다.
-   CardPresentationController

: UIPresentationController 를 상속 받아 presentation controller 을 custom 하였다.

```
import UIKit

class CardPresentationController: UIPresentationController {

    private lazy var blurView = UIVisualEffectView(effect: nil)

    // 1️⃣
    override var shouldRemovePresentersView: Bool {
        return false
    }

    // 2️⃣
    override func presentationTransitionWillBegin() {
        let container = containerView!
        blurView.translatesAutoresizingMaskIntoConstraints = false
        container.addSubview(blurView)
        blurView.edges(to: container)
        blurView.alpha = 0.0

        presentingViewController.beginAppearanceTransition(false, animated: false)
        presentedViewController.transitionCoordinator!.animate(alongsideTransition: { (ctx) in
            self.blurView.effect = UIBlurEffect(style: .light)
            self.blurView.alpha = 1
        }) { (ctx) in }
    }

        // 3️⃣
    override func presentationTransitionDidEnd(_ completed: Bool) {
        presentingViewController.endAppearanceTransition()
    }

        // 4️⃣
    override func dismissalTransitionWillBegin() {
        presentingViewController.beginAppearanceTransition(true, animated: true)
        presentedViewController.transitionCoordinator!.animate(alongsideTransition: { (ctx) in
            self.blurView.alpha = 0.0
        }, completion: nil)
    }

        // 5️⃣
    override func dismissalTransitionDidEnd(_ completed: Bool) {
        presentingViewController.endAppearanceTransition()
        if completed {
            blurView.removeFromSuperview()
        }
    }
}
```

주석 1️⃣

-   presentation animations 이 끝날 때 presenting view controller 를 삭제하는지 여부. 기본값은 false 이다. 즉, present 을 명령한 뷰컨의 삭제 여부
-   override 할 때 super 를 호출하지 않는다.

주석 2️⃣ `presentationTransitionWillBegin()`

-   presentation controller 에게 presentation animations 가 시작될 것을 알리는 메서드이다.
-   이 메서드의 기본 구현은 아무 작업도 수행하지 않는다. 서브클래스가 이것을 override 하고 view hierarchy 에 커스텀뷰를 추가하기 위해서 사용하고 해당 뷰와 관련된 애니메이션을 만드는데 사용할 수 있다.
-   애니메이션을 수행하기 위해서 presented view controller 의 transition coordinator 를 가져오고 `animate(alongsideTransition:completion:)` 또는 `animateAlongsideTransition(in:animation:completion:)` 메서드를 호출한다.
-   이러한 메서드를 호출하면 다른 transition animations 와 동시에 실행된다.

주석 3️⃣ `presentationTransitionDidEnd(_:)`

-   **presentation phase** 가 끝나면 이 메서드를 호출해서 transition 이 완료되었음을 알린다.
-   정상적으로 animations 가 완료되면 completed 파라미터는 true 값을 가진다.
-   이 메서드의 기본 구현은 아무 작업도 수행하지 않는다. 서브클래스가 이것을 override 하고 필요한 정리를 수행하는데 사용 가능하다. 예를 들어, completed 파마리터가 false 면 view hierarchy 에서 custom views 를 삭제해준다.

주석 4️⃣ `dismissalTransitionWillBegin()`

-   뷰컨트롤러가 dismiss 될 때 호출된다.
-   이 메서드의 기본구현은 아무 작업도 수행하지 않는다. 서브클래스는 이것을 override 하고 presentation 의 custom views 와 관련된 모든 animations 를 구성하는데 사용 가능하다.
-   애니메이션을 수행하기 위해서 presented view controller 의 transition coordinator 를 가져오고 `animate(alongsideTransition:completion:)` 또는 `animateAlongsideTransition(in:animation:completion:)` 메서드를 호출한다.
-   이러한 메서드를 호출하면 다른 transition animations 와 동시에 실행된다.
-   이 메서드를 view hierarchy 로부터 views 를 제거하기 위해서 사용하지 마라. 대신 `dismissalTransitionDidEnd(_:)` 메서드에서 제거해라.

주석 5️⃣ `dismissalTransitionDidEnd(_:)`

-   view hierarchy 로부터 custom view 를 없애기 위해서 이 메서드를 호출한다.

---

-   CardDetailViewController (present 될 뷰컨)

: custom 한 presentation controller 를 presented view controller 에서 직접 설정해주어야 custom presentation controller 가 transition animation 을 실행한다.

```
class CardDetailViewController: UIViewController {

// ...

// 1️⃣
        private func setupTranstion() {
        modalPresentationStyle = .custom
        transitioningDelegate = self
    }
}

// ...

// 2️⃣
extension CardDetailViewController: UIViewControllerTransitioningDelegate { 
        // ...

    func presentationController(forPresented presented: UIViewController, presenting: UIViewController?, source: UIViewController) -> UIPresentationController? {
        return CardPresentationController(presentedViewController: presented, presenting: presenting)
    }
}
```

주석 1️⃣

-   `moadalPresentationStyle` : custom presentation controller 를 앱으로 제공받기 위해서 `.custom` 으로 설정해준다.
-   `transitioningDelegate` : custom presentation controller 개체를 trainsitioning delegate 를 통해서 제공할 수 있다.

주석 2️⃣ `UIViewControllerTransitioningDelegate`

-   view controller 간의 fixed-length 또는 interactive transition 을 관리하는데 사용되는 개체를 가진 메서드의 집합.
-   `presentationController(forPresented:presenting:source:)` : 뷰컨트롤러를 보여줄 때 view hierarchy 를 관리하는데 custom presentation controller 를 사용하기 위한 요청.
-   즉, custom presentation controller 개체를 view controller 에 사용할 수 있도록 전달.

---

맨앞에서 미뤘던 탭바 숨기기에 대해서 알아보자!

### 🎬 셀 선택시 탭바 숨기기

-   메서드를 호출하는 뷰컨에서의 상태바를 설정하면 되었던 상태바 숨기기와 다르게 탭바를 숨기기 위해서는 탭바컨트롤러에 알려야 한다.
-   그래서 전환 animator 개체를 가져오기 위해서 다음의 두 메서드를 사용했다.

```
extension CardDetailViewController: UIViewControllerTransitioningDelegate {
    // 1️⃣
    func animationController(forPresented presented: UIViewController, presenting: UIViewController, source: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        return TodayAnimationTransition(animationType: .present)
    }

    // 2️⃣
    func animationController(forDismissed dismissed: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        return TodayAnimationTransition(animationType: .dismiss)
    }

        // ...
}
```

주석 1️⃣ `animationController(forPresented:presenting:source:)`

-   뷰컨트롤러를 presenting 할 때 사용할 animator object 를 요청.

주석 2️⃣ `animationController(forDismissed:)`

-   뷰컨트롤러를 dismissing 할 때 사용할 animator object 를 요청.

**present 와 dismiss 경우의 애니메이션을 정의한 TodayAnimationTransition.swift 를 살펴보자**

-   1️⃣ 탭바 숨기기
-   2️⃣ 이미지 확대 및 축소

```
import UIKit

fileprivate let transitonDuration: TimeInterval = 1.0

enum AnimationType {
    case present
    case dismiss
}

class TodayAnimationTransition: NSObject {
    let animationType: AnimationType!

    init(animationType: AnimationType) {
        self.animationType = animationType
        super.init()
    }
}

extension TodayAnimationTransition: UIViewControllerAnimatedTransitioning {
    func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval {
        return transitonDuration
    }

    func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
        if animationType == .present {
            animationForPresent(using: transitionContext)
        } else {
            animationForDismiss(using: transitionContext)
        }
    }

        // present 시 함수
    func animationForPresent(using transitionContext: UIViewControllerContextTransitioning) {
        let containerView = transitionContext.containerView
        //1.Get fromVC and toVC
        guard let fromVC = transitionContext.viewController(forKey: .from) as? UITabBarController else { return }
        guard let tableViewController = fromVC.viewControllers?.first as? TodayViewController else { return }
        guard let toVC = transitionContext.viewController(forKey: .to) as? CardDetailViewController else { return }
        guard let selectedCell = tableViewController.selectedCell else { return }

        let frame = selectedCell.convert(selectedCell.bgBackView.frame, to: fromVC.view)        
        //2.Set presentation original size.
        toVC.view.frame = frame

                // 2️⃣ todayCardSize 로 imageView 의 크기 설정. (기본크기)
        toVC.scrollView.imageView.frame.size.width = GlobalConstants.todayCardSize.width
        toVC.scrollView.imageView.frame.size.height = GlobalConstants.todayCardSize.height

        containerView.addSubview(toVC.view)

        //3.Change original size to final size with animation.
        UIView.animate(withDuration: transitonDuration, delay: 0, usingSpringWithDamping: 0.5, initialSpringVelocity: 0, options: [], animations: {
            toVC.view.frame = UIScreen.main.bounds

                        // 2️⃣ width 는 UIScreen 의 width, height 는 cardDetailTopImageH 로 크기 설정. (이미지 확대)
            toVC.scrollView.imageView.frame.size.width = kScreenW
            toVC.scrollView.imageView.frame.size.height = GlobalConstants.cardDetailTopImageH
            toVC.closeBtn.alpha = 1

                        // 1️⃣ tabBar 의 y 좌료를 screen 의 높이 만큼 설정해서 화면 아래로 내려버렸다.
            fromVC.tabBar.frame.origin.y = kScreenH
        }) { (completed) in
            transitionContext.completeTransition(completed)
        }
    }

        // dismiss 시 함수
    func animationForDismiss(using transitionContext: UIViewControllerContextTransitioning) {
        guard let fromVC = transitionContext.viewController(forKey: .from) as? CardDetailViewController else { return }
        guard let toVC = transitionContext.viewController(forKey: .to) as? UITabBarController else { return }
        guard let tableViewController = toVC.viewControllers?.first as? TodayViewController else { return }
        guard let selectedCell = tableViewController.selectedCell else { return }

        UIView.animate(withDuration: transitonDuration - 0.3, delay: 0, usingSpringWithDamping: 0.8, initialSpringVelocity: 0, options: [], animations: {
            let frame = selectedCell.convert(selectedCell.bgBackView.frame, to: toVC.view)
            fromVC.view.frame = frame
                // 2️⃣ 이전 크기로 되돌아가기. (이미지 축소)
            fromVC.view.layer.cornerRadius = GlobalConstants.toDayCardCornerRadius
            fromVC.scrollView.imageView.frame.size.width = GlobalConstants.todayCardSize.width
            fromVC.scrollView.imageView.frame.size.height = GlobalConstants.todayCardSize.height
            fromVC.closeBtn.alpha = 0

                        // 1️⃣ tabBar 의 y 좌표를 원래대로 돌려놓기
            toVC.tabBar.frame.origin.y = kScreenH - toVC.tabBar.frame.height
        }) { (completed) in
            transitionContext.completeTransition(completed)
            toVC.view.addSubview(toVC.tabBar)
        }
    }

}
```

아래에서 모든 코드를 확인할 수 있다.

github - appstore clone:

[GitHub - DragonTnT/appstore-clone: a clone of iOS appstore, to be continued](https://github.com/DragonTnT/appstore-clone)

---

참고

[iOS) Presentation 과 Transition 그리고 Animation... (2)](https://koggy.tistory.com/4?category=922104)

[Apple Developer Documentation](https://developer.apple.com/documentation/uikit/uipresentationcontroller#1653379)
