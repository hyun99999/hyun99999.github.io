---
title:  "iOS) Gesture Recognizer 를 활용한 화면전환"
categories:
- iOS

date:   2021-04-19 18:29:00 +0900
author_profile: false
---
modal 창으로 뷰컨트롤러를 present 한 경우 아래로 쓸어내리면 창이 닫힌다 하지만 .fullScreen 속성을 주게되면 dismiss 로 쓸어내려도 창이 닫히지 않는다.

### fullScreen 형태의 화면을 제스쳐를 활용해서 dismiss 기능을 구현해보자.

### 1) Add addGestureRecognizer to View

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    view.addGestureRecognizer(UIPanGestureRecognizer(target: self, action: #selector(handleDismiss)))
}
```

### 2) Create HandleDismiss Function

```swift
@objc func handleDismiss(sender: UIPanGestureRecognizer) {
        //write here
    }
```

### 3) Handling Sender Using Switch Cases

```swift
var viewTranslation = CGPoint(x: 0, y: 0)
@objc func handleDismiss(sender: UIPanGestureRecognizer) {
    switch sender.state {
    case .changed:
        viewTranslation = sender.translation(in: view)
        UIView.animate(withDuration: 0.5, delay: 0, usingSpringWithDamping: 0.7, initialSpringVelocity: 1, options: .curveEaseOut, animations: {
            self.view.transform = CGAffineTransform(translationX: 0, y: self.viewTranslation.y)
        })
    case .ended:
        if viewTranslation.y < 200 {
            UIView.animate(withDuration: 0.5, delay: 0, usingSpringWithDamping: 0.7, initialSpringVelocity: 1, options: .curveEaseOut, animations: {
                self.view.transform = .identity
            })
        } else {
            dismiss(animated: true, completion: nil)
        }
    default:
        break
    }
}
```

팬제스처의 현재 위치를 추적할 변수 `viewTranslation` 을 만든다.

**.changed** - panning 을 시작할때(최소한으로 설정한 손가락의 갯수가 터치하면 인식) .changed 케이스의 조건을 활용할 수 있다.

- `viewTranslation` 을 사용자 팬의 현재위치로 설정.
- UIView.animate() 를 활용해서 팬의 위치까지 이동시키는 클로저를 추가해서 panning 동안 뷰가 이동하도록 하였다.

**.ended** - panning 을 완료하면 .ended 케이스의 조건을 활용할 수 있다.

- 사용자의 팬의 위치를 담고있는 `viewTeranslation` 의 y 좌표가 200 보다 작게되면 `.identity` 를 사용해서 뷰를 원래의 위치로 변환시킨다.
- 그렇지않으면 dismiss 함수로 뷰를 닫는다.

---
### 참조

[Drag down to dismiss tutorial](https://betterprogramming.pub/simple-drag-dismiss-on-presented-view-controller-tutorial-5f2f44f86f7b)

[Gesture recognizer](https://zeddios.tistory.com/356?category=682195)

[스토리보드, 타켓액션, 델리게이트로 구현해보기](https://etst.tistory.com/96)
