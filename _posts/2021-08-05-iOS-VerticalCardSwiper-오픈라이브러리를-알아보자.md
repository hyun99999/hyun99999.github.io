---
title:  "iOS) VerticalCardSwiper 오픈라이브러리를 알아보자
categories:
- iOS

date:   2021-08-05  16:54:00 +0900
author_profile: false
---
📇 VerticalcardswiperTutorial

최근에 현대카드 DIVE 라는 앱의 레이아웃을 보고 다음과 같은 layout 를 가지는 오픈 라이브러리가 없나찾아보았다.

[2021.07.29 (목) - 현대카드 DIVE iOS](https://www.youtube.com/watch?v=B8-cTdaUuRQ)

아래의 사이트가 아주 유용하다 추천한다

[iOS Example](https://iosexample.com)

VerticalCardSwiper 라는 오픈 라이브러리를 찾았다.

[GitHub - JoniVR/VerticalCardSwiper: A marriage between the Shazam Discover UI and Tinder, built with UICollectionView in Swift.](https://github.com/JoniVR/VerticalCardSwiper)

### Introduce

이 프로젝트의 목표는 카드를 왼쪽/오른쪽으로 스와이프하는 Tinder 스타일과 결합하여 Shazam 의 Discover UI 를 재현하는 것이라고 한다. 이것은 UICollectionView 및 custom flowLayout 으로 빌드됩니다.

<img src ="https://user-images.githubusercontent.com/69136340/128309164-2d7dff31-57df-4907-b185-b7cffb5137c6.gif" width ="250">

### Installation

CocoaPods 

```swift
pod 'VerticalCardSwiper'
```

### Usage

우리가 사용할 주요한 클래스들이 무엇을 상속받는지 먼저 확인해보자

```swift
public class VerticalCardSwiper: UIView {

    /// The collectionView where all the magic happens.
    public var verticalCardSwiperView: VerticalCardSwiperView!
// ...
```

```swift
public class VerticalCardSwiperView: UICollectionView {
// ...
```

```swift
@objc open class CardCell: UICollectionViewCell {
```

우리는 뷰컨트롤러에서 VerticalCardSwiper 를 만들고 원하는 크기대로 설정을 할 것이다. 

깃허브에서 제공하는 코드를 보자

- ViewController.swift

```swift
import VerticalCardSwiper

class ExampleViewController: UIViewController, VerticalCardSwiperDatasource {
    
    private var cardSwiper: VerticalCardSwiper!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        cardSwiper = VerticalCardSwiper(frame: self.view.bounds)
        view.addSubview(cardSwiper)
        
        cardSwiper.datasource = self
        
        // register cardcell for storyboard use
        cardSwiper.register(nib: UINib(nibName: "ExampleCell", bundle: nil), forCellWithReuseIdentifier: "ExampleCell")
    }
    
    func cardForItemAt(verticalCardSwiperView: VerticalCardSwiperView, cardForItemAt index: Int) -> CardCell {
        
        if let cardCell = verticalCardSwiperView.dequeueReusableCell(withReuseIdentifier: "ExampleCell", for: index) as? ExampleCardCell {
            return cardCell
        }
        return CardCell()
    }
    
    func numberOfCards(verticalCardSwiperView: VerticalCardSwiperView) -> Int {
        return 100
    }
}
```

CardCell 를 상속받아서 ExampleCardCell 를 커스텀할 수 있다.

나는 좀 더 편하게 사용해보고 싶었다.

<img src ="https://user-images.githubusercontent.com/69136340/128309224-cb2d666d-09d3-4167-b76e-e631febf19a9.png" width ="600">

다음과 같이 대체했다

- ViewController.swift

```swift
@IBOutlet weak var cardSwiper: VerticalCardSwiper!
// private var cardSwiper: VerticalCardSwiper!

    override func viewDidLoad() {
        super.viewDidLoad()
                // cardSwiper = VerticalCardSwiper(frame: self.view.bounds)
                // view.addSubview(cardSwiper)
                cardSwiper.datasource = self
        cardSwiper.register(nib: UINib(nibName: "VerticalCardSwiperCell", bundle: nil), forCellWithReuseIdentifier: "VerticalCardSwiperCell")
    }
```

- VerticalCardSwiperCell.swift

```swift
import UIKit
import VerticalCardSwiper

class VerticalCardSwiperCell: CardCell {

    override func awakeFromNib() {
        super.awakeFromNib()
        // Initialization code
    }
}
```

- VerticalCardSwiperCell.xib

<img src ="https://user-images.githubusercontent.com/69136340/128309292-78abcc00-f8b7-43c0-ad65-3d4d6cdcdba7.png" width ="400">


다음과 같이 꽉차도록 uiview 를 만들었고 card 를 구분할 수 있도록 background color 를 설정해보았다.

### Result

<img src ="https://user-images.githubusercontent.com/69136340/128309359-ffd35d66-133d-42e2-9b76-74922ecb66b8.gif" width="250">

잘 적용이 되었다.

### 최대한 현대카드 DIVE 앱을 클론해보자

<img src ="https://user-images.githubusercontent.com/69136340/128309652-9b4a60f5-503a-4122-ac03-5913fa33cc85.gif" width ="250">
