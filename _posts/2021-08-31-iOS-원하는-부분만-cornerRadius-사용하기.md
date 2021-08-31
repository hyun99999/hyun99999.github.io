---
title:  "iOS) 원하는 부분만 cornerRadius 사용하기"
categories:
- iOS

date:   2021-08-31  14:11:00 +0900
author_profile: false
---
아래와 같이 상단만 둥근 모서리를 가진 뷰를(파란뷰) 만들고 싶었다. (하얀배경의 뷰와 파란배경의 뷰 두개로 구성)

<img src ="https://user-images.githubusercontent.com/69136340/131445643-309e759e-bc2d-453a-bba9-d50b86b954c8.png" width = "250">

평소 같으면 `masksToBounds` 프로퍼티에 true 를 줘서 해결하려했는데 뒤에 그림자가있어서 상단좌우측 둥근 모서리와 그림자를 동시에 줄 수 없었다.(서로가 반대의 프로퍼티 값을 요구하기 때문.)

그래서 `maskedCorners` 프로퍼티를 설정하기로 했다.

```swift
qrcodeTopView.backgroundColor = .blue
qrcodeTopView.layer.cornerRadius = 10
// 좌측상단, 우측상단
qrcodeTopView.layer.maskedCorners = [.layerMinXMinYCorner, .layerMaxXMinYCorner]
```

- 다음의 옵션을 통해서 설정 가능하다.

```swift
public struct CACornerMask : OptionSet {

    public init(rawValue: UInt)

    
    public static var layerMinXMinYCorner: CACornerMask { get }

    public static var layerMaxXMinYCorner: CACornerMask { get }

    public static var layerMinXMaxYCorner: CACornerMask { get }

    public static var layerMaxXMaxYCorner: CACornerMask { get }
}
```
