---
title:  "iOS) FlexLayout 과 PinLayout 을 사용하여 UI 구현"
categories:
- iOS

date:   2023-03-25  15:43:00 +0900
author_profile: false
---
### 내용

- FlexLayout 과 PinLayout 을 알아보고, 사용하여 UI를 구현해보자.
- **목표** : 아래의 [업데이트를 위한 팝업 뷰]를 구현해보겠습니다.

<img width="300" alt="1" src="https://user-images.githubusercontent.com/69136340/227701015-6ab6e0e1-7cf4-4473-9e14-3c8d51749392.png">

📌 **FlexLayout 과 PinLayout 의 사용방법에 대해서는 게시글이 많고, 깃허브 리드미를 읽으면 쉽게 알 수 있을 만큼 문법이 간편합니다.**

📌 **이 글에서는 예시 코드에서 확인할 수 있는 사용방법에 대해서 간단하게 깃허브 리드미를 참고하여 주석을 작성하고, 실제로 특정 뷰를 만드는데 적용한 코드를 공유하는데 초점을 맞춰서 작성하였습니다.**

## 1️⃣ why? FlexLayout?

FlexLayout 은 카카오뱅크, 당근마켓 등에서 사용되고 있다. 그렇다면 그 이유는 무엇인지 알아보자.

<img src="https://user-images.githubusercontent.com/69136340/227701030-bc725275-c9ed-441b-b2f1-d4090edd83db.png" width ="600">

출처: [https://github.com/layoutBox/FlexLayout](https://github.com/layoutBox/FlexLayout)

UIStackView 를 개선한 프레임워크로써 AutoLayout 과 UIStackView 보다 훨씬 빠르다.

PinLayout 과 주로 함께 사용된다.

- (FlexLayout 컨테이너 안에 PinLayout을 사용하거나 FlexLayout 컨테이너에 PinLayout을 적용할 수 있습니다.)
- 많은 뷰를 배치해야하고 복잡한 애니메이션이 필요하지 않은 경우는 FlexLayout이 PinLayout보다 적합합니다.

## 2️⃣ FlexLayout install

- CocoaPods

```swift
pod 'FlexLayout'
```

## 3️⃣ FlexLayout 적용

flexbox container 를 사용하는 두 가지 방법이 있습니다.

- Setup the container
- Layout the container

### 1. Setup the container

```swift
import FlexLayout

class UpdateViewController: UIViewController {

    // MARK: - Components
    
    private let rootFlexContainer = UIView()
    // ...

}

// MARK: - Layout

extension UpdateViewController {
    private func setLayout() {
        view.addSubview(rootFlexContainer)

        rootFlexContainer.flex.define { flex in
            // ...
        }
    }
}
```

- FlexLayout 예시 코드를 살펴보겠습니다.

<img src="https://user-images.githubusercontent.com/69136340/227701099-e7b9f1d8-4eca-4d93-a77c-048778fb5f94.png" width ="600">

```swift
fileprivate let rootFlexContainer = UIView()

init() {
   super.init(frame: .zero)
   
   addSubview(rootFlexContainer)
   ...

   // Column container
   // direction : default value is 'column'.
   rootFlexContainer.flex.direction(.column).padding(12).define { (flex) in
        // Row container
        flex.addItem().direction(.row).define { (flex) in
            flex.addItem(imageView).width(100).aspectRatio(of: imageView)
            
            // Column container
            flex.addItem().direction(.column).paddingLeft(12).grow(1).define { (flex) in
                flex.addItem(segmentedControl).marginBottom(12).grow(1)
                flex.addItem(label)
            }
        }
        
        flex.addItem().height(1).marginTop(12).backgroundColor(.lightGray)
        flex.addItem(bottomLabel).marginTop(12)
    }
}
```

### 2. **Layout the container**

컨테이너의 레이아웃은 `layoutSubviews()` (또는, `willTransition(to: UITraitCollection, ...)` 및 `viewWillTransition(to: CGSize, ...)`)에서 수행해야 합니다.

1. flexbox container 의 레이아웃을 지정해야함. 즉, 위치를 지정하고 선택적으로 크기를 설정해야 합니다.
2. Flex 메서드 `layout()` 을 사용하여 flexbox 의 자식들을 레이아웃합니다.
- FlexLayout 예시 코드를 살펴보겠습니다.

<img src="https://user-images.githubusercontent.com/69136340/227701111-00630f0d-15f5-4943-93ae-7af04ee94c2b.png" width ="600">

```swift

override func layoutSubviews() {
    super.layoutSubviews() 

    // 1) Layout the flex container. This example use PinLayout for that purpose, but it could be done 
    //    also by setting the rootFlexContainer's frame:
    //       rootFlexContainer.frame = CGRect(x: 0, y: 0, 
    //                                        width: frame.width, height: rootFlexContainer.height)
    rootFlexContainer.pin.top().left().width(100%).marginTop(topLayoutGuide)

    // 2) Then let the flexbox container layout itself. Here the container's height will be adjusted automatically.
    rootFlexContainer.flex.layout(mode: .adjustHeight)
}
```

`pin` 이라는 메서드를 체이닝해서 사용하는 것을 볼 수 있습니다. 이는 **PinLayout** 을 **FlexLayout** 과 함께 사용한 것입니다. 그렇다면 **FlexLayout** 의 깃허브 리드미에서 왜 **PinLayout** 을 사용했는지 살펴보겠습니다.

### 👉 PinLayout

FlexLayout 에서 말하는 PinLayout 과 함께 사용하는 것에 대해서 정리한 것을 알아보겠습니다.

<img src ="https://user-images.githubusercontent.com/69136340/227701123-3fd20b4e-5a7f-449e-8df9-70a6db704986.png" width ="300">

FlexLayout 은 PinLayout 의 동반자입니다. 비슷한 구문과 메서드 이름을 공유합니다. PinLayout 은 CSS 의 absolute positioning 에서 영감을 받은 레이아웃 프레임워크로, 세밀한 제어와 애니메이션에 특히 유용합니다. 

한 번에 하나의 뷰를 레이아웃하여 전체 제어를 제공합니다.(코딩 및 디버깅이 간담함)

- FlexLayout, PinLayout 또는 둘다를 사용하여 뷰를 레이아웃할 수 있습니다.
- PinLayout 은 무엇이든 배치할 수 있지만, 많은 뷰를 레이아웃해야만 하거나 PinLayout 의 정밀한 제어나 복잡한 애니메이션을 필요로하지 않다면 FlexLayout 이 가장 적합합니다.
- PinLayout 을 사용하여 레이아웃된 뷰는 FlexLayout 컨테이너 내부에 포함되거나 반대로 포함할 수 있습니다. 상황에 적합한 프레임워크를 선택해야 합니다.

출처: [https://github.com/layoutBox/FlexLayout#flexlayout--pinlayout](https://github.com/layoutBox/FlexLayout#flexlayout--pinlayout)

즉, FlexLayout 은 UIStackView 를 개선한 것이고, PinLayout 은 auto layout 이 아닌 방법으로 UIView 및 CALayer 를 레이아웃할 수 있습니다.

그래서 UIStackView 처럼 안에 담을 아이템들을 FlexLayout 으로 설정하고 root container 가 되는 뷰를 PinLayout 으로 배치하는 것으로 함께 사용할 수 있습니다.

이 container 를 상위의 UIStackView 의 아이템으로 사용하게 되면 반대로 포함할 수 있는 개념이 됩니다.

**이제, PinLayout 을 사용하여 flexbox container 의 레이아웃을 잡아보겠습니다.**

- **PinLayout** install - **CocoaPods** 를 사용하여 설치하겠습니다.

```swift
pod 'PinLayout'
```

예를 들어, 다음과 같이 사용해볼 수 있습니다.

<img src="https://user-images.githubusercontent.com/69136340/227701154-e4fe05aa-90fd-4d2b-a644-f83de935e548.png" width ="600">

```swift
override func layoutSubviews() {
   super.layoutSubviews() 
   let padding: CGFloat = 10
   
   // superview 기준으로 top, bottom, left, right 등 사용 가능. 이는 superview 를 기준으로 함. 
   // pin.safeArea : UIView.safeAreaInsets 를 사용 가능.
   // aspectRatio : 너비/높이 비율. 이미지에 사용하는 경우에만 이미지 dimension 을 사용하여 설정.
   logo.pin.top(pin.safeArea).left(pin.safeArea).width(100).aspectRatio().margin(padding)
   // after
   // right
   // marginHorizontal 
   segmented.pin.after(of: logo, aligned: .top).right(pin.safeArea).marginHorizontal(padding)
   // below
   // width
   // pinEdges
   // sizeToFit
   textLabel.pin.below(of: segmented, aligned: .left).width(of: segmented).pinEdges().marginTop(10).sizeToFit(.width)
   separatorView.pin.below(of: [logo, textLabel], aligned: .left).right(to: segmented.edge.right).marginTop(10)
}
```

너무 코드가 복잡해 보이나요?

화면에 꽉차는 rootFlexContainer 를 만들기 위해서 다음과 같이도 작성해 볼 수 있습니다.

```swift
override func layoutSubviews() {
    super.layoutSubviews()

    // rootFlexContainer.pin.top().bottom().left().right() 와 동일.
    self.rootFlexContainer.pin.all() // 1

    // default is .fitContainer.
    self.rootFlexContainer.flex.layout() // 2
}
```

직접 사용하기 전에 [PinLayout](https://github.com/layoutBox/PinLayout) 깃허브에서 더 알아보겠습니다.

### PinLayout’s Performance

Layout Framework Benchmark 의 표를 살펴보면 PinLayout 은 auto layout 보다 8배에서 12배 더 빠릅니다.

<img src="https://user-images.githubusercontent.com/69136340/227701167-d0fe0e7e-3893-4ccf-8eeb-94633e4cd8b0.png" width ="600">

### UIKit safeAreaInsets support

`UIView.pin.safeArea` 를 통해 `UIView.safeAreaInsets` 을 쉽게 처리할 수 있습니다.

### RTL support

Right to left languages(RTL) 을 지원합니다. [RTL support](https://github.com/layoutBox/PinLayout/blob/master/docs/rtl_support.md) 에서 더 자세히 확인할 수 있습니다.

### PinLayout + Autolayout

**PinLayout** 을 사용하여 일부 뷰의 레이아웃을 지정할 수 있고, 다른 일부는 autolayout 을 사용할 수 있습니다. 뷰가 단지 autolayout `intrinsicContentSize` 속성을 구현하기만 하면 됩니다.

### PinLayout style guide

항상 같은 순서로 메서드를 지정해야 layout lines 을 쉽게 이해할 수 있습니다. 선호하는 순서는 다음과 같습니다.

`view.pin.[EDGE|ANCHOR|RELATIVE].[WIDTH|HEIGHT|SIZE].[pinEdges()].[MARGINS].[sizeToFit()]`

이 순서는 **PinLayout** 내부 로직를 반영합니다. `pinEdges()` 는 `margins` 전에 적용되고, `margins` 는 `sizeToFit()` 전에 적용됩니다.

```swift
// edges layout 을 설정할 때 퍼센트로도 가능.
// margin([top], [left], [bottom], [right])
view.pin.top().left(10%).margin(10, 12, 10, 12)
view.pin.left().width(100%).pinEdges().marginHorizontal(12)
// sizeToFit : 
view.pin.horizontally().margin(0, 12).sizeToFit(.width)
view.pin.width(100).height(100%)
```

항상 같은 순서로 edges 를 지정해야 합니다. `TOP, BOTTOM, LEFT, RIGHT`

```swift
view.pin.top().bottom().left(10%).right(10%)
```

layout line 이 너무 길면 여러 줄로 나눌 수 있습니다.

```swift
textLabel.pin.below(of: titleLabel)
    .before(of: statusIcon).after(of: accessoryView)
    .above(of: button).marginHorizontal(10)
```

**📌 PinLayout의 메서드 호출 순서는 무관하며 레이아웃 결과는 항상 동일합니다.**

## 4️⃣ 설계와 구현

원하는 레이아웃을 구현해보겠습니다. 아래는 최종적으로 설계한 뷰의 계층을 색깔별로 표현한 것입니다.

<img src="https://user-images.githubusercontent.com/69136340/227701240-ee691e77-139b-4b28-b74b-026358de6d19.png" width ="250">

- 우선, container 로 삼을 영역을 생각해야 합니다. rootFlexContainer 역할 뷰를 설정하고 이는 superview 에 대해서 **PinLayout** 으로 설정하겠습니다.

```swift
import UIKit

import FlexLayout
import PinLayout

class UpdateViewController: UIViewController {

    // MARK: - Components
    
    private let rootFlexContainer = UIView()
    private let updateCardImageView = UIImageView()
    private let cancelButton = UIButton()
    private let updateContentLabel = UILabel()
    private let checkBoxButton = UIButton()
    private let checkLabel = UILabel()
    private let updateButton = UIButton()
    
    // MARK: - View Life Cycle
    
    override func viewDidLoad() {
        super.viewDidLoad()

        // ...
    }
    
    override func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews()
        

        // ✅ PinLayout 으로 flexbox container 의 레이아웃을 지정.
        rootFlexContainer.pin
            .vCenter()  // vertical 중심 정렬
            .hCenter()  // horizontal 중심 정렬
            .height(487).width(327)
/*
        // 너비 대비 비율로 높이를 지정할 수도 있습니다.
        rootFlexContainer.pin.horizontally(24)  // left(24).right(24) 과 동일
            .vCenter()  
            .height(rootFlexContainer.frame.width / 327 * 487) 
*/

        // ✅ Flex 메서드 layout() 을 사용하여 flexbox 의 자식 레이아웃을 지정.
        rootFlexContainer.flex.layout()
    }
}
```

`viewDidLayoutSubviews` 메서드를 재정의해서 컨테이너의 레이아웃을 설정하였습니다.

잠시 **ViewController 에서 레이아웃이 결정되는 과정을 살펴보겠습니다.**

1. `viewWillLayoutSubviews()` 메서드  호출
2. ViewController 의 content view 가 `layoutSubviews()` 메서드 호출
    - **layoutSubviews()**: 현재 레이아웃 정보들을 바탕으로 새로운 레이아웃 정보를 계산
    - 이후 뷰 계층구조를 순회하면서 모든 하위 뷰들이 동일한 메서드를 호출
3. `viewDidLayoutSubviews()` 메서드 호출

`viewDidLayoutSubviews()` 를 재정의하여 사용하는 이유는 일일히 뷰에 대해서 **FlexLayout** 을 적용하여 커스텀 뷰로 만들지 않고  ViewController 에서 컨테이너의 레이아웃을 설정하기 위해서입니다. (`layoutSubviews()` 이후에 `viewDidLayoutSubviews()` 가 호출되기 때문)

출처 : [https://ios-development.tistory.com/195](https://ios-development.tistory.com/195)

혹은, ViewController 의 view 에 **FlexLayout** 을 적용한 UIView 를 대입해줄 때는 `loadView()` 를 재정의해주면 됩니다.

참고 : [https://dokit.tistory.com/20](https://dokit.tistory.com/20)

저는 재사용되지 않을 뷰이기 때문에 따로 커스텀 뷰를 만들어주지 않기로 했습니다.

- rootFlexLayout 의 레이아웃을 지정해보았습니다.
<img src="https://user-images.githubusercontent.com/69136340/227701276-730e27ce-6882-4beb-aee2-b9b90d56b05e.png" width ="250">

- 수직으로 item 들을 배치시키고 그 안에서 세부적으로 **FlexLayout** 으로 레이아웃을 지정하기 위해서 노란색으로 rootFlexContainer 안의 item 들을 표현해보았습니다.

<img src="https://user-images.githubusercontent.com/69136340/227701301-86104b95-ddfb-49ab-acfd-c70e064ff759.png" width ="250">

FelxLayout 의 장점 중 하나로 선언형 프로그래밍인 SwiftUI 와 유사하게 코드를 통해 UI 확인할 수 있습니다. 아래의 코드에서는 item 끼리 어떻게 묶었는지 계층을 확인해보면 되겠습니다.

```swift
// MARK: - Layout

extension UpdateViewController {
    private func setLayout() {
        view.addSubview(rootFlexContainer)
        
        rootFlexContainer.flex.direction(.column).define { flex in
            // 📌 item1
            flex.addItem().direction(.row).define { flex in
                flex.addItem(updateCardImageView)
                flex.addItem(cancelButton)
            }
            // 📌 item2
            flex.addItem(updateContentLabel)
            // 📌 item3
            flex.addItem().direction(.row).define { flex in
                flex.addItem(checkBoxButton)
                flex.addItem(checkLabel)
            }
            // 📌 item4
            flex.addItem(updateButton)
        }
    }
}
```

실행해보면 다음과 같이 뷰 계층을 확인할 수 있고, 의도한 대로 배치되었음을 확인할 수 있습니다.

(’업데이트 하러가기’ 버튼은 가운데 정렬된 것이 아니라 이미지를 추가했기 때문에  좌우의 여백 모두 UIButton 영역입니다.)

<img width="500" alt="11" src="https://user-images.githubusercontent.com/69136340/227701323-c916a212-1015-4410-a1ad-5471d1f63231.png">

- 두 번째로, 안에 item 들을 어떻게 둘지를 설계해야 합니다. 초록색으로 item 안에 item 들을 표시해보았습니다.

<img src="https://user-images.githubusercontent.com/69136340/227701361-97cc7cc2-478d-4706-993c-6ce61ce0cdaa.png" width ="250">
    
- item 1~4 에 대해서 다음과 같은 설정이 필요할 것입니다.
    - item1 에서 중앙에 위치하는 이미지와 우측 상단에 붙어있는 버튼
    - item2 의 마진
    - itme3 에서 버튼과 라벨의 레이아웃 설정. item3 는 item4 을 기준으로 위치.
    - item4 rootFlexContainer 의 bottom 을 기준으로 위치.

### 🚨 트러블 슈팅 : 어떻게 item 1 은 위에 붙어있고, item 4 는 아래에 붙어있을 수 있을까요?

- **해결** : 카드 이미지는 위에서 시작하고, 업데이트 버튼은 아래에서 시작하기 때문에 item1~4 를 감싸는 item 층이 하나 더 필요하다고 판단했습니다!
- item 을 두 개 추가하여 계층을 추가해보겠습니다.

다음과 같이 표시할 수 있습니다. 차이를 느낄 수 있도록 라벨의 내용을 줄여보겠습니다.

- 보라색으로 item 을 추가하고, **[justifyContent(.spaceBetween)](https://github.com/layoutBox/FlexLayout#justifycontent)** 메서드를 사용하여 레이아웃을 구현해보겠습니다.

<img src="https://user-images.githubusercontent.com/69136340/227701384-4751664f-7810-4a3b-8154-551722241c38.png" width = "250">

```swift
// MARK: - Layout

extension UpdateViewController {
    private func setLayout() {
        view.addSubview(rootFlexContainer)
        
        // ✅ 목표: updateCardImageView 는 위에, updateButton 는 맨 아래에서 시작.
        rootFlexContainer.flex.direction(.column)
            // 첫 번째 아이템은 시작, 마지막 아이템은 끝에 위치하도록 item1, item2 를 감싸고, item3, item4 를 감싸는 아이템 계층 두 개를 추가함.
            .justifyContent(.spaceBetween)
            .define { flex in
            
            // 1
            flex.addItem().direction(.column).define { flex in
                // 📌 item1
                // ✅ 목표: updateCardImageView 를 중앙정렬.
                flex.addItem().direction(.column).alignItems(.center).define { flex in
                        flex.addItem(updateCardImageView).marginTop(20)
                        // 아이템의 크기를 유지(기본값)
                        // .grow(0)
                        
                        // ✅ 목표: rootFlexContainer 에 대해서 우측상단에 위치.
                        // .absolute 을 사용하여(position(.relative)가 기본값) parent 와 관련하여 절대적으로 위치함.
                        flex.addItem(cancelButton).position(.absolute).top(20).right(20)
                    }
                // 📌 item2
                flex.addItem(updateContentLabel).marginTop(16).marginBottom(18).marginHorizontal(16)
            }
                
            // 2
            // 🚨 아래의 트러블 슈팅2 를 거쳐서 수정됩니다.
            flex.addItem().direction(.column).define { flex in
                // 📌 item3
                // ✅ 목표: updateButton 을 기준으로 bottom 간격을 가짐. checkBoxButton 와 cehckLabel 이 수평 중앙 정렬.
                flex.addItem().direction(.row).alignItems(.center).marginBottom(20).define { flex in
                    flex.addItem(checkBoxButton).marginLeft(16)
                    flex.addItem(checkLabel).marginLeft(2)
                }
                // 📌 item4
                flex.addItem(updateButton).marginBottom(16).marginHorizontal(17)
            }
        }
    }
}
```

- 뷰 계층을 살펴보면 의도한대로 구현한 것을 확인할 수 있습니다.

<img width="500" alt="14" src="https://user-images.githubusercontent.com/69136340/227701440-bdddfa19-8d73-493a-b0f8-c589baec35f8.png">

**..! 그런데 미세하게 ‘업데이트 하러가기’ 버튼 아래 마진이 좁은거 같지 않나요?**

<img width="500" alt="15" src="https://user-images.githubusercontent.com/69136340/227701449-5b9f6b28-de85-456b-be26-dc8dd2b52aec.png">

### 🚨 트러블 슈팅2 : 다음과 같이 뷰가 튀어 나왔습니다.

- 체크박스 버튼의 image view 위아래로 공간이 있기 때문에 의도하지 않은 공간이 생겼고, 그만큼 item 이 밀리게 되었습니다.

<img width="600" alt="16" src="https://user-images.githubusercontent.com/69136340/227701464-d207c7c9-9e4d-40ca-a189-809da64eb086.png">

- **해결** : 체크박스 버튼의 사이즈를 정해주지 않고, 16*16 크기의 image 를 설정해주었는데 버튼 자체를 16*16으로 설정.

```swift
// 2
flex.addItem().direction(.column).define { flex in
    // 📌 item3
    flex.addItem().direction(.row).alignItems(.center).marginBottom(20).define { flex in
        // 🚨 사이즈 설정. .width(16).height(16) 과 동일.
        flex.addItem(checkBoxButton).marginLeft(16).size(16)
        flex.addItem(checkLabel).marginLeft(2)
    }
}
```

<img width="500" alt="17" src="https://user-images.githubusercontent.com/69136340/227701496-9354a146-14b4-461f-bc52-7a551a8df5a0.png">

## ✅ 결과

- 왼(mini 13) / 오(14 Pro Max)

<img width="500" alt="18" src="https://user-images.githubusercontent.com/69136340/227701519-0c3a0746-deea-4398-a410-fcebf1bf10e7.png">

### 👉 느낀 점

**장점**

장점으로 알려진 점들이 확실하게 와닿았습니다.

익숙치 않았기 때문에 머리속으로만 하던 설계 단계를 가시화해야 했습니다. 하지만, 익숙해지면 코드로써도 바로 가시화할 수 있겠다고 느꼈습니다.

**단점**

여러가지 이슈를 만나게 되면 자료와 경험이 부족할 수 있겠다라고 느꼈습니다.

UIStackView 로 레이아웃을 작성하는 것만큼 고려해야할 사항도 많았고 많이 해봐야 여러가지 레이아웃에 대응할 수 있다라고 느꼈습니다.

Storboard 와 AutoLayout 을 사용했다면 손쉽게했을 관계에 대한 위치 지정과 화면 대비 크기 대응 설정이 어렵게 느껴졌습니다.

**오픈소스 라이브러리 선정과정**

당근 마켓의 경우는 레이아웃 프레임워크인 Texture 에서 변경하였다고 합니다.
그 이유로 Texture 의 life cycle 의 이해 등이 신규 멤버들에게 어려움이 되었다고 합니다. 또한, Texture 는 활발하게 유지보수 되지 못하는 오픈소스 라이브러리라는 것이 근거 중에 하나였습니다. iOS 업데이트에 대응하며 꾸준히 유지보수가 필요하기 때문에 오픈소스 라이브러리를 선정할때는 활성화 정도도 보는 것이 중요하다고 느꼈습니다.

---

**참고:**

[https://github.com/layoutBox/FlexLayout](https://github.com/layoutBox/FlexLayout)

[FlexLayout (1)](https://green1229.tistory.com/242)

[FlexLayout 사용해보기](https://zeddios.tistory.com/1251)

[새로운 iOS UI 개발 feat. FlexLayout](https://okanghoon.medium.com/%EC%83%88%EB%A1%9C%EC%9A%B4-ios-ui-%EA%B0%9C%EB%B0%9C-feat-flexlayout-64d0ca8091a7)

[[iOS - swift] 1. FlexLayout과 PinLayout 사용 방법 - UIStackView 개선, 속도 향상, 기능 추가, 선언형](https://ios-development.tistory.com/1275)
