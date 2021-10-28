---
title:  "iOS) iOS) UICollectionViewCell 내용에 대해 동적인 사이즈 적용시키기"
categories:
- iOS

date:   2021-10-28  22:55:00 +0900
author_profile: false
---
동적으로 셀의 너비와 높이를 설정하는 방법에 대해서 알아보자구요!

셀의 사이즈를 정하는 것이니 **collectionView(_:layout:sizeForItemAt:)** 메서드 안에서 이루어지는 것이라고 생각할 수 있을거에요! 자 그럼 어떻게 코드를 짜야하냐..! 

이제부터 소개하는 내용에 정답은 없어요! 상황과 여건에 따라서 사용하시면되고 더 좋은 코드가 있다면 알려주세요!

- 방법1 : collection view cell 객체 생성해서 데이터를 넣고 frame 의 size 를 가져오기.
- 방법2 : UILabel 사용하기
- 방법3 : NSString 사용하기
- 방법4 : systemLayoutSizeFitting 사용하기
- 방법5 : boundingRect(with:options:attributes:context:) 를 사용하기

# Dynamic Cell Width

핵심은 `intrinsicContentSize` 를 기반으로 cell 의 사이즈를 정하는 것입니다!

📌 **intrinsicContentSize**

> 내용을 기반으로 하는 고유 크기라고 볼 수 있어요! 그래서 sizeToFit() 함수 대신 아래와 같이 사용할 수 있지요!

```swift
// ✅ sizeToFit() 과 동일한 코드.
// ✅ 즉, sizeToFit() 를 호출하면 사이즈가 intrinsicContentSize 가 된다.
label.frame.size = label.intrinsicContentSize 
```

그래서 셀의 내용에 따라서 달라지는 고유크기를 기반으로 셀의 사이즈를 설정해볼거에요

## 🛼 방법1 : collection view cell 객체 생성해서 데이터를 넣고 frame 의 size 를 가져오기

```swift
func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAt indexPath: IndexPath) -> CGSize {
        guard let cell = topicCollectionView.dequeueReusableCell(withReuseIdentifier: "TopicCollectionViewCell", for: indexPath) as? TopicCollectionViewCell else {
            return .zero
        }
        cell.topicLabel.text = topicList[indexPath.row].name
        // ✅ sizeToFit() : 텍스트에 맞게 사이즈가 조절
        cell.topicLabel.sizeToFit()

        // ✅ cellWidth = 글자수에 맞는 UILabel 의 width + 20(여백)
        let cellWidth = cell.topicLabel.frame.width + 20

        return CGSize(width: cellWidth, height: 30)
    }
```

cell 이 가진 label 의 `intrinsicContentSize` 크기를 알아내서! 여백이 될 값을 더해서! 셀의 너비를 설정해주는 방법입니다! 😎 

이 방법은 셀의 너비를 (텍스트라벨의 너비 + 내가 정한 여백) 으로 구성하기 때문에 xib 에서 라벨의 constraints 를 어떤 방법(수직&수평정렬, leading&trailing 등)으로 잡아도 괜찮아요!

## 🛼 방법2 : UILabel 사용하기

UILabel 을 만들고 셀의 텍스트를 넣어서 크기를 구해주면 됩니다! 이 역시 방법1과 동일하게 label 의 intrinsicContentSize 를 알아내서 여백을 더해주는 방법입니다.

방법 1과의 차이점은 속성을 그대로 다시금 부여해줘야한다는 점과 cell 객체를 가져오지 않아도 되는 점이 있겠네요!

```swift
func calculateCellWidth(index: Int) -> CGFloat {
        let label = UILabel()
        label.text = topicList[index]
        label.font = .systemFont(ofSize: 14)
        label.sizeToFit()
        // ✅ 20(여백)
        return label.frame.width + 20
}
```

## 🛼 방법3 :  NSString.**size(withAttributes:)** 사용하기

### **[size(withAttributes:)](https://developer.apple.com/documentation/foundation/nsstring/1531844-size)**

> 주어진 속성으로 그릴 때 bounding box size 를 반환한다.

```swift
func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAt indexPath: IndexPath) -> CGSize {
        // ✅ 20(여백)
        let cellSize = CGSize(width: topicList[indexPath.item].size(withAttributes: [NSAttributedString.Key.font : UIFont.systemFont(ofSize: 14)]).width + 20, height: 30)
        return cellSize
}
```

텍스트에 속성(font, size)을 부여하고 그릴 때 bounding box size 의 width 에 여백을 더해서 cell 사이즈를 설정했어요!

## 🛼  방법4 : **systemLayoutSizeFitting 사용하기**

먼저 systemLayoutSizeFitting 에 대해서 알아볼까요?

### **[systemLayoutSizeFitting(_:withHorizontalFittingPriority:verticalFittingPriority:)](https://developer.apple.com/documentation/uikit/uiview/1622623-systemlayoutsizefitting)**

> constraints 와 지정된 priorities 를 기반으로 뷰의 최적 크기를 반환한다.

이 메서드를 사용해서 이번에는 cell 에서 지정한 constraints 를 기반으로 한 contentView 의 최적 크기로 cell 을 설정해보자구요!
❗️단, 방법1과 다른 점은 cell 의 contentView 의 너비를 구하기 위해서 Xib 파일에서 라벨에 leading, trailing 의 constraints 를 필수로 잡아주어야 한답니다!

**Declaration**

```swift
func systemLayoutSizeFitting(_ targetSize: CGSize, 
withHorizontalFittingPriority horizontalFittingPriority: UILayoutPriority, 
     verticalFittingPriority: UILayoutPriority) -> CGSize
```

**Parameters**

`targetSize`

view 에 선호하는 size 이다. 가능한 작은 view 를 얻으려면 [layoutFittingCompressedSize](https://developer.apple.com/documentation/uikit/uiview/1622568-layoutfittingcompressedsize) 를 지정하고, 가능한 큰 view 를 얻으려면 [layoutFittingExpandedSize](https://developer.apple.com/documentation/uikit/uiview/1622532-layoutfittingexpandedsize) 를 지정하면 된다.

`horizontalFittingPriority`

horizontal constraints 의 우선 순위이다. 파라미터 targetSize 의 너비값에 최대한 가까운 값을 얻으려면 [fittingSizeLevel](https://developer.apple.com/documentation/uikit/uilayoutpriority/1622248-fittingsizelevel) 를 지정하면 된다. 

`verticalFittingPriority`

vertical constraints 의 우선 순위이다. 파라미터 targetSize 의 너비값에 최대한 가까운 값을 얻으려면 [fittingSizeLevel](https://developer.apple.com/documentation/uikit/uilayoutpriority/1622248-fittingsizelevel) 를 지정하면 된다.

---

`horizontalFittingPriority` 와 `verticalFittingPriority` 모두 `UILayoutPriority` 라는 자료형을 요구해요.

### [UILayoutPriority](https://developer.apple.com/documentation/uikit/uilayoutpriority)

> layout priority 는 constraint-based(제약조건 기반) 레이아웃에서 어떤 제약 조건이 더 중요한지를 나타내는데 사용된다.

1-1000 까지의 값을 가질 수 있지만 1000(required), 750(defaultHigh), 250(defaultLow) 으로 권장된다.

- required : 가장 강한 우선순위. 필수 제약 조건이다.
- defaultHigh
- defaultLow
- fittingSizeLevel : 뷰가 target size 를 준수하려는 우선 순위. systemLayoutSizeFitting(_:) 메서드를 호출하면 target size 에 가장 가까운 크기가 계산된다.

즉, 실제 레이아웃을 고정적으로 변경하는 것이 아니라 상황에 따라서 어떤 제약 조건을 적용시킬지 요구될때 우선순위를 부여하는 것이지요! 그러면 시스템은 적절하게 적용을 할 수 있겠죠?

무..서워하지마세요..! 우리는 이걸 이미 사용해봤답니다?!

스크롤뷰를 만들 때 priority 를 주잖아요! 그때 250(Low)을 준 이유는 내용에 따라서 유동적으로 높이를 조절하기 위함이었어요! 그래서 제약조건을 낮게 주어야 제한되지 않도록 했지요! 이해가 조금 되시나요?

<img width="300" alt="스크린샷 2021-10-28 오후 2 27 47" src="https://user-images.githubusercontent.com/69136340/139270118-c056aa77-0074-482e-83cf-7398860b842a.png">

먼저, 셀에서 systemLayoutSizeFitting() 메서드를 호출해서 최적의 사이즈를 반환하는 메서드를 만들어봅시다!

```swift
// TopicCollectionViewCell.swift

class TopicCollectionViewCell: UICollectionViewCell {
    @IBOutlet weak var topicLabel: UILabel!

    // ...

    override func awakeFromNib() {
        super.awakeFromNib()
    }

    // ...

    // ✅ collectionView(_:layout:sizeForItemAt:) 에서 아래의 메서드를 호출해서 셀의 사이즈를 설정해줄거에요.
    // ✅ cellHeight : 고정적인 높이값, text: 너비를 결정할 문자열.
    func sizeFittingWith(cellHeight: CGFloat, text: String) -> CGSize {
        topicLabel.text = text

        // ✅ systemLayoutSizeFitting 메서드 파라미터에 필요한 targetSize(선호하는 사이즈)를 만들어보자.
        // ✅ 너비의 경우 intrincsicContentSize 에 딱 맞도록 최소 크기를 위해서 layoutFittingCompressedSize 를 사용해서 설정.
        // ✅ 높이의 경우 파라미터로 받아온 고정 값 설정.
        let targetSize = CGSize(width: UIView.layoutFittingCompressedSize.width, height: cellHeight)

        // ✅ targetSize 를 만들어주었으니 선호하는 사이즈를 명시할 수 있겠죠!
        // ✅ horizontal constraints(너비)는 contentView 에 따라서 늘어나야하기 때문에 우선순위가 낮고 최대한 targetSize 와 가까운 값을 얻는 fittingSizeLevel 로 설정.
        // ✅ vertical constraints(높이)는 targetSize 에 맞춰야하니 우선순위가 가장 높은 required 로 설정.
        return self.contentView.systemLayoutSizeFitting(targetSize, withHorizontalFittingPriority: .fittingSizeLevel, verticalFittingPriority: .required)
    } 
}
```

마지막으로 UICollectionViewDelegateFlowLayout 프로토콜을 채택해서 셀의 사이즈를 설정해봅시다! 

```swift
extension mainVC: UICollectionViewDelegateFlowLayout {
    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAt indexPath: IndexPath) -> CGSize {
        guard let cell = topicCollectionView.dequeueReusableCell(withReuseIdentifier: "TopicCollectionViewCell", for: indexPath) as? TopicCollectionViewCell else {
            return .zero
        }
        // ✅ 고정적인 셀의 높이 값.
        let cellHeight: CGFloat = 30

        let cellSize = cell.sizeFittingWith(cellHeight: cellHeight, text: topicList[indexPath.row].name)
        return cellSize
    }
}
```

# Dynamic Cell Height

이번에는 높이를 조절해볼까요? 

## 🛼  방법5:  NSString.**boundingRect(with:options:attributes:context:) 를 사용하기**

[boundingRect(with:options:attributes:context:)](https://developer.apple.com/documentation/foundation/nsstring/1524729-boundingrect)

> 특정한 사각형 내부에서 옵션과 속성들을 사용해서 그릴 때 bounding rect 을 반환한다.
> 

**Parameter**

**`size`**

그릴 사각형의 사이즈.

**`options`**

문자열 그리기 옵션. [NSStringDrawingOptions](https://developer.apple.com/documentation/foundation/nsstring/nsstringdrawingoptions) 자료형.

**`attributes`**

문자열에 적용할 텍스트 속성의 dictionary. 

 NSAttributedString 객체에 적용할 수 있는 것과 동일한 속성이다. 하지만, NSString 객체의 경우 문자열 내의 범위가 아닌 전체 문자열에 적용된다. 

**`context`**

receiver 가 사용할 문자열 drawing context. 

**Discussion**

여러 줄 텍스트를 올바르게 그리고 크기를 조정하려면 options 매개변수에 `usesLineFragmentOrigin` 을 전달하세요.

이제는 코드를 짜볼까요!

```swift
func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAt indexPath: IndexPath) -> CGSize {
        let cellSize = NSString(string: topicList[indexPath.row]).boundingRect(
            with: CGSize(width: 100, height: CGFloat.greatestFiniteMagnitude),
            options: .usesLineFragmentOrigin,
            attributes: [
                    NSAttributedString.Key.font: UIFont.systemFont(ofSize: 14)
            ],
            context: nil)
                
        return CGSize(width: 100, height: cellSize.height + 110)
}
```

파라미터가 너무 많아서 여기에 나눠적어볼게요!

`CGFloat.greatestFiniteMagnitude` : CGFloat 의 가장 큰 유한 수. 동적 높이를 계산하기 때문에 그릴 사각형의 높이를 가장 큰 유한수로 설정하면 맘이 편하겠죠!

`usesLineFragmentOrigin` : 문자열이 사각형에 담기에 너무 크다면 주어진 사각형에 맞도록 문자열의 간격을 조절 하는 옵션. 여러 줄 텍스트를 올바르게 그리고 크기를 조정하기 위해서 사용.

방법 3에서 사용했던 NSString.size(withAttributes:) 메서드가 생각나시지 않으신가요! 줄 수가 하나라면 size 를! 여러줄이라면 boundingRect 이 적합한 것 같아요!

---

위의 코드들은 셀의 높이도 너비도 모두 적용가능해요! 예시로 든것일뿐!

어떤 방법이 정답이라고 할 수 없고 상황에 따라서 필요에 따라서 선택하면 될 것 같아요. 이외에도 좋은 방법이 있다면 알려주세요!

### 출처 :

[[UIKit] Dynamic cell sizing - systemLayoutSizeFitting](https://ntomios.tistory.com/15)

[Text Size 구하기](https://onemoonstudio.tistory.com/2)
