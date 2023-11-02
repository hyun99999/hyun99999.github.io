 ---
title:  "iOS) UIBulrEffect 를 사용하기 + UIVisualEffectView 에 둥글기 적용하기"
categories:
- iOS

date:   2023-11-02  12:01:00 +0900
author_profile: false
---
### 내용

- 명함 뒷면의 선택되지 않은 취향을 블러처리 해보려 했습니다.
- 둥글기를 가진 bulr effect 구현하였습니다.

<img src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/3d145d8c-81c3-48de-bf0b-fb6f76df1ee4" width ="400">

위의 blur 효과를 적용해보겠습니다.

### 결과

<img width="600" alt="2" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/bd7571c2-5dfa-45f0-b033-d951d4719221">

둥글기가….?! 이상하지만 주석에서 설명드리겠습니다.
**해당 글은 트러블 슈팅 과정을 담았기 때문에 최종 구현을 원하시면 아래 트러블 슈팅을 읽어주시기 바랍니다.**

(https://ikyle.me/blog/2022/uiblureffectstyle) 참고해서 에서 일반 배경일때와 검정 배경일때 적절히 비슷한 blur 효과를 찾아보았습니다.

```swift
let blurEffect = UIBlurEffect(style: .systemMaterialLight)
// 많은 예제들이 style 로 초기화하던데 frame 으로 해보았습니다.
let visualEffectView = UIVisualEffectView(frame: tasteView.frame)
visualEffectView.effect = blurEffect

// ✅ tasteView 에 visualEffectView 를 추가해서 둥글기를 잡아보려했습니다.
// -> 이러면 tasteView 색상에 영향을 받음
// bgView 에 effectView 를 추가해야 blur 효과가 적용되었습니다. -> 이러면 둥글기가 잡히지 않음
tasteView.backgroundColor = .clear

// FIXME: - 둥글기 설정을 위해서 view 에 추가. 그러면 backgroundImageView blur 안됨.
// tasteView.addSubview(visualEffectView)
// tasteView.layer.masksToBounds = true
            
// FIXME: - 둥글기 적용 안됨.
if index % 2 == 0 {
// 좌측
    visualEffectView.layer.maskedCorners = [.layerMinXMinYCorner, .layerMinXMaxYCorner]
} else {
// 우측
    visualEffectView.layer.maskedCorners = [.layerMaxXMinYCorner, .layerMaxXMaxYCorner]
}
visualEffectView.layer.cornerRadius = 35 / 2

bgView.addSubview(visualEffectView)
```

bgView 의 뷰 계층은 다음과 같습니다.

<img width="400" alt="3" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/ba3839d1-69f7-46db-b453-c001cf71ec0c">

**결과적으로,** blur 를 bgView(image view 위 alpha 값을 가진 view)에 추가해야하는데 그렇게 되면 corner radius 가 조절되지 않았습니다.(조절할 수 있다구! 아래에서 확인해보겠습니다.)

maskToBounds 를 사용해서 둥글기를 조절하기 위해 UIView 를 만들어서 얹으면 해당 뷰의 색상 조절에 영향을 받아서 안됐습니다.

blur effect 를 적용해보았지만, 둥글기를 적용하기에 애를 먹었습니다 ㅜ 

## 🚨 트러블 슈팅
- visualEffectView 도 UIView 이기 때문에 tasteView 에 뷰를 추가해서 masksToBounds 를 true 로 만드는 것이 아닌!
visualEffectView 의 masksToBounds 를 true 로 설정해보기로 했습니다.

뷰가 많으니 너무 어렵게 생각했던 것 같습니다...

```swift
let blurEffect = UIBlurEffect(style: .systemMaterialLight)
let visualEffectView = UIVisualEffectView(frame: tasteView.frame)

tasteView.backgroundColor = .clear
visualEffectView.effect = blurEffect

if index % 2 == 0 {
// 좌측
    visualEffectView.layer.maskedCorners = [.layerMinXMinYCorner, .layerMinXMaxYCorner]
} else {
// 우측
    visualEffectView.layer.maskedCorners = [.layerMaxXMinYCorner, .layerMaxXMaxYCorner]
}
visualEffectView.layer.cornerRadius = 35 / 2
// ✅
visualEffectView.layer.masksToBounds = true

bgView.addSubview(visualEffectView)
```

성공적으로 적용되었습니다!

<img src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/f0b0a3ea-07b8-4e47-a11f-ac255b6753e8" width ="250">

이 외에도 둥글기를 그리는 방법 중 하나인 UIBezierPath 를 사용하는 방법으로도 구현할 수 있었습니다.

```swift
let shapeLayer = CAShapeLayer()
            
shapeLayer.path = UIBezierPath(roundedRect: visualEffectView.bounds,
                             byRoundingCorners: [.topLeft, .bottomLeft],
                             cornerRadii: CGSize(width: 17.5, height: 17.5)).cgPath

visualEffectView.layer.mask = shapeLayer
```
