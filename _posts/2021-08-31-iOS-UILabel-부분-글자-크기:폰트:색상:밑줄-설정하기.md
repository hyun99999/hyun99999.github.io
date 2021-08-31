---
title:  "iOS) UILabel 부분 글자 크기/폰트/색상/밑줄  설정하기"
categories:
- iOS

date:   2021-08-31  11:53:00 +0900
author_profile: false
---
### UILabel 부분 글자 크기/폰트/색상/밑줄  설정하기

<img src ="https://user-images.githubusercontent.com/69136340/131525013-5be69cc1-56e1-4511-be43-9b3244ee6497.png" width ="250">

하단의 " QR 체크인 쉐이크 기능 끄기 " 버튼의 부분 글자에 밑줄을 만들어보자.

```swift
let text = "QR 체크인 쉐이크 기능 끄기"
self.switchShakeButton.setTitle(text, for: .normal)

let attributeString = NSMutableAttributedString(string: text)

// ✅ 굵기 1의 언더라인과 함께 처음부터 끝까지 밑줄 설정.
attributeString.addAttribute(.underlineStyle , value: 1, range: NSRange.init(location: 0, length: text.count))
self.switchShakeButton.titleLabel?.attributedText = attributeString
```

UILabel 에서도 당연히 가능하다. 왜냐하면 UILabel 에서 가능하기 때문에 UIButton 의 UILabel 에서 적용할 수 있는 것이기 때문이다.

그렇다면 addAttribute(_:value:range:) 메서드를 뜯어보자.

### 🌈 [addAttribute(_:value:range:)](https://developer.apple.com/documentation/foundation/nsmutableattributedstring/1417080-addattribute)

 지정된 범위의 문자에 주어진 이름과 값을 가진 속성을 추가한다.

### Declartion

```swift
func addAttribute(_ name: NSAttributedString.Key, 
            value: Any, 
            range: NSRange)
```

### Parameters

- name

속성 이름을 지정하는 문자열. 

[NSAttributedString.Key](https://developer.apple.com/documentation/foundation/nsattributedstring/key) 키값을 자료형으로 요구하기 때문에 여기 있는 다양한 키들을 사용해서 우리는 속성을 줄 수 있다.

- value

이름과 관련된 속성 값이다. 예로들어 색이라면 UIColor, 밑줄이라면 Int 값의 굵기 등이 있다.

- aRange

지정된 속성/값 쌍이 적용되는 문자 범위이다.

### Discussion

문자 범위에 원하는 이름/값 쌍을 할당할 수 있습니다. 이름이나 값이 nil이면 `invalidArgumentException` 을 발생시키고 aRange의 일부가 수신자의 문자 끝을 넘어서 있으면 `rangeException` 을 발생시킵니다.

### 🌈 사용해보기

간단하게 몇개를 사용해보겠다. 더 많은 키값들이 존재하니까 [NSAttributedString.Key](https://developer.apple.com/documentation/foundation/nsattributedstring/key) 필요에 따라 개발자문서를 참고해보자.

- 크기와 폰트

```swift
let font = UIFont(name: "Helvetica", size: 12)
// ✅ UIFont 개체를 지정하지 않으면 12-point 의 Helvetica 폰트가 디폴트값이다.
attributeString.addAttribute(.font , value: font, range: NSRange.init(location: 0, length: 1))
```

- 색상

```swift
// ✅ value 로 UIColor 인스턴스를 가진다. 지정하지 않으면 검은색으로 랜더링된다.
attributeString.addAttribute(.foregroundColor , value: UIColor.black, range: NSRange.init(location: 0, length: 1))
```

- 밑줄

```swift
// ✅ value 로 정수를 포함하는 NSNumber 개체를 가진다.
attributeString.addAttribute(.underlineStyle , value: 1, range: NSRange.init(location: 0, length: 1))
// ✅ 그리고 NSUnderlineStyle 의 상수 중 하나에 해당한다. 이 속성의 기본값은 styleNone 이다.
attributeString.addAttribute(.underlineStyle , value: NSUnderlineStyle.single , range: NSRange.init(location: 0, length: text.count))
// ✅ 위의 두가지 모두 가능하다.
```

당연히 여러개의 속성과 값을 동시에 줄 수 있는 메서드도 존재한다.

- [addAttributes(_:range:)](https://developer.apple.com/documentation/foundation/nsmutableattributedstring/1414304-addattributes)

### 참고 :

[iOS ) Label의 부분 글자 크기/폰트/색상 변경방법](https://zeddios.tistory.com/300)
