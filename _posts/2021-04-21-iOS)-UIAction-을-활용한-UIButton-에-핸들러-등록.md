---
title:  "iOS) UIAction 을 활용한 UIButton 에 핸들러 등록"
categories:
- iOS

date:   2021-04-21 21:33:00 +0900
author_profile: false
---
### UIAction closure based UIControl
출처ㅣ 
[iOS 14 + ) UIAction closure based UIControl](https://zeddios.tistory.com/1093#recentEntries)

 `@objc`  와 `addTarget` 말고도 UIControl 생성자를 통해서 액션함수를 등록하는 방법을 소개하겠습니다!

- UIButton 을 만들고 터치할 때 마다 특정한 action 을 하는 코드

```swift
override func viewDidLoad() {
        super.viewDidLoad()
        let button = UIButton(type: .custom)
        button.addTarget(self,
                                        action: #selector(buttonDidTap),
                                        for: .touchUpInside)
}

@objc func buttonDidTap() {
        print("touch")
}
```

addTarget 으로 연결해주었다. iOS14 에서는 다른 방법으로 UIButton 에 action 을 줄 수 있다.

```swift
override func viewDidLoad() {
        super.viewDidLoad()
        let button = UIButton(type: .custom,
                                                    primaryAction: UIAction(handler: { _ in
                print("touch")
        }))
}
```

파라미터 primaryAction 이 눈에 띄는데 `UIAction` 타입을 받는다.

- iOS 14 에서 이 `UIAction` 을 사용해서 `UIControl` 인스턴스를 만들 수 있도록 `UIControl` 의 새로운 생성자를 만들어주었다.

```swift
convenience init(frame: CGRect,
        primaryAction: UIAction?)
```

- convenience init 이란?

→ 편의 이니셜라이저. 지정 이니셜라이저인 `Designated init` 은 정의된 클래스의 모든 프로퍼티를 초기화해야 하는 임무를 갖고 있지만 편의 이니셜라이저는 Designated init 의 파라미터 중 일부를 기본값으로 설정해서, 이 convenience init안에서 Designated init을 호출 하여 초기화를 진행할 수 있다. 

→ 즉, 클래스의 모든 프로퍼티를 초기화히지 않아도 괜찮다는 것이다.

- 블로그 중 "UIButton의 생성자는 `convenience init` 입니다. 궁극적으로 `UIControl` 의 생성자를 호출하겠죠?" 무슨 의미인가?

→ 자식 클래스의 지정 이니셜라이저는 부모 클래스의 지정 이니셜라이저를 반드시 호출해야 한다.

→ 편의 이니셜라이저는 자신을 정의한 클래스의 다른 이니셜라이저를 반드시 호출해야 한다.

→ 편의 이니셜라이저는 "궁극적으로" 지정 이니셜라이저를 반드시 호출해야 한다.

왜냐면 편의 이니셜라이저는 자신을 정의한 클래스의 다른 이니셜라이저를 반드시 호출하기 때문!

라는 규칙이 적용될 수 있다. ( 출처: 스위프트 프로그래밍/야곰 )

즉, UIButton 의 편의 이니셜라이저는 지정 이니셜라이저를 호출하고 지정 이니셜라이저는 부모 클래스인 UIControl 의 지정 이니셜라이저도 호출 한다. 

<img src = "https://user-images.githubusercontent.com/69136340/115554990-fe274c00-a2e9-11eb-8e02-02d880699343.png" width ="400">

확인해보니까 UIControl 에  init(type:primaryAction:) 생성자가 있다.

이것은 편의 이니셜라이저이다. 그렇다면.... 저 언급은 UIControl 의 지정 이니셜라이저를 호출한다는 말인 것 같다.

UIControl 의 편의 이니셜라이저 `convenience init(frame: CGRect, primaryAction: UIAction?)`가 UIButton 에도 편의지정자로 되어 있는데 이런 상속의 이유는 UIButton 가 부모 클래스의 지정 이니셜라이저를 상속 혹은 오버라이드를 통해서 구현했다면 부모 클래스의 편의 이니셜라이저가 자동을 상속되기 때문이다.

출처ㅣ [오늘의 Swift 상식 (Initializer 2편. 클래스의 Initializer)](https://medium.com/@jgj455/오늘의-swift-상식-initializer-2편-클래스의-initializer-7141cda4ecf2)

---

### UIAction

UIAction 은 iOS13 에서 새로나온 클래스이다. 특징은 action 에 closure 를 바로 넣을 수 있다.

- 다음은 apple 의 네비게이션 바 커스터미아징 프로젝트에서 `UIAction` 으로 `UIMenu` 의 children 을 구성한 코드이다.

```swift
let barButtonMenu = UIMenu(title: "", children: [
            UIAction(title: NSLocalizedString("Copy", comment: ""), image: UIImage(systemName: "doc.on.doc"), handler: menuHandler),
            UIAction(title: NSLocalizedString("Rename", comment: ""), image: UIImage(systemName: "pencil"), handler: menuHandler),
            UIAction(title: NSLocalizedString("Duplicate", comment: ""), image: UIImage(systemName: "plus.square.on.square"), handler: menuHandler),
            UIAction(title: NSLocalizedString("Move", comment: ""), image: UIImage(systemName: "folder"), handler: menuHandler)
        ])
        optionsBarItem.menu = barButtonMenu
```

이렇게도 사용 가능하다.

```swift
// Create a closure-based action to use as a menu element.
let refreshAction = UIAction(title: "Refresh") { (action) in
    print("Refresh the data.")
}

// Use the .displayInline option to avoid displaying the menu as a submenu,
// and to separate it from the other menu elements using a line separator.
let refreshMenuItem = UIMenu(title: "", options: .displayInline, children: [refreshAction])

// Insert the menu into the File menu before the Close menu.
builder.insertSibling(refreshMenuItem, beforeMenu: .close)
```

출처ㅣ[Apple Developer Documentation](https://developer.apple.com/documentation/uikit/uiaction)
