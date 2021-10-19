---
title:  "iOS) 여러개의 view controller pop 하기"
categories:
- iOS

date:   2021-10-20  05:22:00 +0900
author_profile: false
---
navigation view controller 에서 pop 하는 방법은 총 세가지다.

- [popToRootViewController(animated:)](https://developer.apple.com/documentation/uikit/uinavigationcontroller/1621855-poptorootviewcontroller) : rootViewController 를 제외한 모든 view controller 를 pop 한다.
- [popToViewController(_:animated:)](https://developer.apple.com/documentation/uikit/uinavigationcontroller/1621871-poptoviewcontroller) : 특정 viewController 가 navigation stack 의 맨 위에 올때까지 pop 한다.
- [popViewController(animated:)](https://developer.apple.com/documentation/uikit/uinavigationcontroller/1621886-popviewcontroller) : navigation stack 에서 view controller 를 pop 한다.

마지막 `popViewController(animated:)` 메서드는 우리가 자주 사용해봤으니 앞에 2개 메서드에 대해서 알아보자.

다음의 코드는 네비게이션컨트롤러에서 뷰컨트롤러를 present 했고 **버튼을 통해서 dismiss 와 pop 을 동시에 구현하는 코드**이다.

### popToRootViewController(animated:)

```swift
// ✅ popToRootViewController 메서드 사용
// ✅ 현재 뷰컨트롤러를 present 한 presentingViewController(이전 뷰) 를 반환.
        guard let presentingVC = self.presentingViewController as? UINavigationController else { return }
        self.dismiss(animated: true) {
            presentingVC.popToRootViewController(animated: true)
        }
```

### popToViewController(_:animated:)

- rootViewController 뿐 아니라 특정 view controller 로 pop 하고 싶을 때 사용 가능하다.

```swift
// ✅ popToViewController 메서드 사용
// ✅ 현재 뷰컨트롤러를 present 한 presentingViewController(이전 뷰) 를 반환.
        guard let presentingVC = self.presentingViewController as? UINavigationController else { return }
        let viewControllerStack = presentingVC.viewControllers
        // 출력해보자
        print("✅ viewControllerStack: \(viewControllerStack)")
        self.dismiss(animated: true) {
            for viewController in viewControllerStack {
// ✅ navigation stack 에서 LoginViewController(돌아가고싶은 뷰)가 있다면 거기까지 pop. 
                if let rootVC = viewController as? LoginViewController {
                    // 출력해보자
                    print("✅ rootVC: \(rootVC)")
                    presentingVC.popToViewController(rootVC, animated: true)
                }
            }
        }
```

- LoginViewController → (push)SignupViewController → (present)마지막 뷰 → (dismiss) → (pop)

<img width="600" alt="스크린샷 2021-10-19 오후 4 57 30" src="https://user-images.githubusercontent.com/69136340/137982214-b3eb7316-39e2-4f4e-b973-e6b17ccae08b.png">

- LoginViewController → (present)마지막 뷰 → (dismiss) → (pop)

<img width="600" alt="스크린샷 2021-10-19 오후 4 57 25" src="https://user-images.githubusercontent.com/69136340/137982230-3f0eddeb-6cf8-49bf-b1da-5d5faed838ae.png">

## 결과

<img src ="https://user-images.githubusercontent.com/69136340/137982292-26470b99-ac4b-4656-885c-c1dc531cc451.gif" width ="250">

### ⁉️ 궁금증 1

루트뷰로 돌아가기 위해서 `popToRootViewController(animated:)` 메서드를 시도했다. 

```swift
self.dismiss(animated: true) {
        self.navigationController?.popToRootViewController(animated: true)
}

```

하지만 적용되지 않았다! 그 이유는 dismiss 하게되면 현재 뷰가 메모리 상에서 사라지기 때문이었다. 하지만 애초에 presentedViewController(present 되서 보여진 뷰) 의 navigationController 에 접근하는 것도 문제였다.

어쨋든 위의 코드를 사용하면 이미 사라진 현재뷰의 navigationController 를 참조하게 되는 것이 된다.

그래서 `guard let presentingVC = self.presentingViewController else { return }` 이렇게 현재 뷰컨트롤러를 제공한 뷰컨트롤러 객체를(즉, 이전 뷰) 가져와서 presentingVC 를 대상으로 pop 시켜줘야 한다.

```swift
guard let presentingVC = self.presentingViewController as? UINavigationController else { return }
        dismiss(animated: true) {
                presentingVC.popToRootViewController(animated: true)
        }

```

### ⁉️ 궁금증 2

- 처음에는 다음과 같이 작성했었다. 그런데 dismiss 가 된 후 pop 이 되지 않았다.

<img width="600" alt="스크린샷 2021-10-19 오전 10 25 35" src="https://user-images.githubusercontent.com/69136340/137982556-b7c39b2b-33f9-4d86-aaf4-e283739a46e9.png">

- 그래서 다음과 같이 작성했고 구현이 되었다. 왜 그럴까? 두 코드의 차이점은 다운 캐스팅이다.

<img src ="https://user-images.githubusercontent.com/69136340/137982351-41cc8e4d-863f-4cd4-aa56-91764af4ec6e.png" widht ="600">

우선 presentingViewController 의 반환형이 UIViewController 이니까 변수 presentingVC 도 UIViewController 라고 생각했던 것이 잘못이었다. 그래서 navigationController 프로퍼티의 `popToRootViewController(animated:)` 메서드를 호출했다. 

<img width="400" alt="스크린샷 2021-10-19 오전 11 10 27" src="https://user-images.githubusercontent.com/69136340/137982808-2ba43ecd-ff47-43a4-93a1-29646818bef4.png">

하지만 presentingViewController 의 반환값을 출력해보니 UINavigationController 객체를 가지고 있었다. 

```swift
guard let presentingVC = self.presentingViewController else { return }
print(presentingVC)
```

<img width="250" alt="스크린샷 2021-10-19 오전 11 08 45" src="https://user-images.githubusercontent.com/69136340/137982869-c0d23ebc-a3cd-4c70-8293-04bdbba062ff.png">


그렇다면 실행되지 않던 코드의 presentingVC.navigationController 의 값을 출력해보자.

<img width="550" alt="스크린샷 2021-10-19 오전 11 18 25" src="https://user-images.githubusercontent.com/69136340/137982944-bc1fb270-d19e-460e-9228-7e093851e093.png">

어라? **nil 을 출력한다.**

위의 출력 결과에서 보듯이 presentingViewController 는 UINavigationController 를 객체로 가진다. 그런데 navigationController 프로퍼티를 참조하니 값이 비어있을 수밖에 없는 것이다.

(presentingController 의 반환형이 UIController? 이기 때문에 다운캐스팅을 하지 않으면 navigationController 프로퍼티를 참조할 수 있게 코드는 작성되지만 navigationController 값은 가지지 않기때문에  nil 값을 출력.)
