---
title:  "iOS) Design pattern MVVM(2/2) - MVVM 실습해보기"
categories:
- iOS

date:   2021-08-24  00:07:00 +0900
author_profile: false
---
이전 글을 읽고 오면 이해가 더 잘 될 것이다.

[iOS) Design pattern MVVM(1/2) - MVC, MVVM 알아보기](https://gyuios.tistory.com/87)

# 시작 전

디자인 패턴에 대해서는 무엇이 정답이라는 것이 없다고 한다. 그만큼 맹신하면 안된다고 한다. 무엇이 장점이고 단점인지에 대해서 알고 사용해봤는지가 중요하다고 한다. 또한 현업에서도 같은 프로젝트 내에서 한가지 디자인 패턴만 사용하지 않는다고 한다. 그래서 어떤 상황에 어떤 패턴이 유리한지에 대해서 아는 것이 중요하다고 생각 했다.

다음 소개하는 mvvm 패턴은 기본적인 mvvm 의 구성요소에 충실하게 진행했다. 이것보다 더 구체적인 구조도 있고 같은 역할을 다르게 구현한 코드도 많다. 즉, 같은 mvvm 패턴내에서도 행동패턴을 어떻게 가져가냐에 따라 다양하다. 하나의 포스팅으로 배울 것이 아닌 여러 포스팅을 보고 충분히 이해한 후 진행하면 좋을 것 같다.

### 적용할 프로젝트

- 프로필사진 옆의 버튼을 누르면 이미지 변경
- 텍스트필드에 텍스트를 입력 후 버튼을 누르면 라벨의 텍스트 변경

<img src ="https://user-images.githubusercontent.com/69136340/130476086-a75ca439-6bf4-410e-bd94-e351375e17df.gif" width ="250">

## 🌂 Data Binding

우선 data binding 에 대해서 알아보자. 가장 쉽고 널리 알려진 Observable 테크닉을 사용할 것이다. Observable 이라는 클래스를 생성해서 바인딩 역할을 수행해주면 된다.

- Observable

이 클래스를 통해서 원하는 값으로 초기화해주고, binding 역할을 해주고, 값을 가져오는 함수를 제공할 것이다.

```swift
import Foundation

class Observable<T> {
    
    typealias Listener = (T) -> Void
    
    var listener: Listener?
    
// ✅ 값이 변할 때마다 클로저 listener 를 호출한다.(View 의 액션에 따라서 자동으로 View Model 의 값이 최신화.)
    var value: T {
        didSet {
            listener?(value)
        }
    }
    
    init(_ value: T) {
        self.value = value
    }
    
    func bind(_ closure: @escaping (T) -> Void) {
        self.listener = closure
        listener?(value)
    }
}
```

## MVVM 패턴 적용

간단하게 인적사항을 다루는 프로젝트에 적용해보겠다.

- Person.swift

Model 역할

```swift
import Foundation
import UIKit

struct Person {
    var name: String
    var age: String
    var image: UIImage
}
```

- ViewController.swift & Main.storyboard

View 역할

<img src ="https://user-images.githubusercontent.com/69136340/130476156-eb034448-7235-4930-bf14-fea07d47f22c.png" width ="400">

```swift
import UIKit

class ViewController: UIViewController {
    
    // MARK: - Properties
    // ✅ ViewModel 인스턴스화해서 사용
    private var personViewModel = ViewModel()
    
    // MARK: - @IBOutlet Properties
    
    @IBOutlet weak var profileImg: UIImageView!
    @IBOutlet weak var setProfileButton: UIButton!
    @IBOutlet weak var nameLabel: UILabel!
    @IBOutlet weak var nameTextfield: UITextField!
    @IBOutlet weak var ageLabel: UILabel!
    @IBOutlet weak var ageTextfield: UITextField!
    
    // MARK: - View Life Cycle
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // ✅ View Model 이 업데이트 되면 view 를 바꿀 수 있도록 data binding 을 한다.
        personViewModel.image.bind { image in
            self.profileImg.image = image
        }
        
        personViewModel.name.bind { name in
            self.nameLabel.text = name
        }
        
        personViewModel.age.bind { age in
            self.ageLabel.text = age
        }
    }
    
    // MARK: - @IBAction Properties
    
    // ✅ @IBAction 으로 View Model을 업데이트한다.
    @IBAction func touchSetProfile(_ sender: UIButton) {
        personViewModel.setImg(to: "person.fill")
    }
    
    @IBAction func touchSetNameButton(_ sender: UIButton) {
        if let name = nameTextfield.text {
            personViewModel.setName(to: name)
        }
    }
    
    @IBAction func touchSetAgeButton(_ sender: UIButton) {
        if let age = ageTextfield.text {
            personViewModel.setAge(to: age)
        }
    }
}
```

- View Model 역할

```swift
import Foundation
import UIKit

public class ViewModel {
    
    // ✅ 아무 설정 없는 초기의 View 를 보여주기위한 목적의 변수 초기화
    let defaultName = "홍길동"
    let defaultAge = "25"
    let defaultImage = "person"
    
    // ✅ 변수 초기화
    let name = Observable("")
    let age = Observable("")
    // ✅ 변수의 자료형 명시
    let image: Observable<UIImage?> = Observable(nil)
    
    init() {
        setName(to: defaultName)
        setAge(to: defaultAge)
        setImg(to: defaultImage)
    }
    
    // ✅ value 를 바꾸어서 didSet 이 실행되도록 함.
    func setName(to name: String) {
        self.name.value = name
    }
    
    func setAge(to age: String) {
        self.age.value = age
    }
    
    func setImg(to image: String) {
        if let image = UIImage(systemName: image) {
            self.image.value = image
        }
    }
}
```

## 🌂 끝

실습이 끝나고 mvvm 패턴 이미지를 살펴보았다. 아래는 mvvm 을 구글링하면 많이 볼 수 있는 이미지이다. 조금 더 잘 이해할 수 있게 된 것 같다.

<img width="600" alt="스크린샷 2021-08-23 오후 2 33 35" src="https://user-images.githubusercontent.com/69136340/130476212-3d0a59a6-0101-412e-968a-61e08289a359.png">

출처 : [https://lsh424.tistory.com/68](https://lsh424.tistory.com/68)


### 전체코드:

[hyun99999
/
MVVMPatternTutorial-iOS](https://github.com/hyun99999/MVVMPatternTutorial-iOS)

### 출처:

[Data Binding in MVVM on iOS](https://beenii.tistory.com/124)

[[iOS] MVVM 패턴? 어떤 장점이 있을까!](https://velog.io/@sso0022/iOS-MVC-와-MVVM)

[간단한 예제로 살펴보는 iOS Design/Architecture Pattern: MVVM](https://lena-chamna.netlify.app/post/ios_design_pattern_mvvm/#MVVM의-규칙들)

[[iOS] MVVM 디자인 패턴 정리 및 예제코드](https://lsh424.tistory.com/68)
