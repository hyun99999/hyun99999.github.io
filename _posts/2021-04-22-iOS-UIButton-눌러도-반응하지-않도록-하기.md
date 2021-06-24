---
title:  "iOS) UIButton 눌러도 반응하지 않도록 하기"
categories:
- iOS

date:   2021-04-22  16:07:00 +0900
author_profile: false
---
### UIButton 눌러도 반응하지 않도록 하기

- 미리 알림 클론코딩을 하면서 중앙에 정렬된 둥근 이미지뷰는 버튼으로 쉽게 만들어주고 있다. 버튼으로 만드는 것이 좀 더 편했다!
- 그런데 버튼은 눌리기 때문에 비활성화해줄 필요가 있었다.

```swift
@IBOutlet weak var listBulletBtn: UIButton!
override func viewDidLoad() {
    super.viewDidLoad()
    listBulletBtn.isEnabled = false
}
```
> 위의 코드를 통해서 비활성화해주었는데 버튼의 색이 회색으로 변해서 내가 설정한 색이 나타나지 않았다.

```swift
@IBOutlet weak var listBulletBtn: UIButton!
override func viewDidLoad() {
    super.viewDidLoad()
    listBulletBtn.isUserInteractionEnabled = false
}
```
그래서 위의 상호작용을 할 수 없는 코드로 설정해주었다.

<img src ="https://user-images.githubusercontent.com/69136340/115670665-a25dd100-a384-11eb-9e38-6907c10652b3.png" width ="250">
