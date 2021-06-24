---
title:  "iOS) UICollectionView cell 왼쪽정렬"
categories:
- iOS

date:   2021-05-20 20:24:00 +0900
author_profile: false
---
### UICollectionView cell 왼쪽정렬

1.다음과 같은 스위프트 파일을 만듭니다.

```swift
import Foundation
import UIKit

class LeftAlignedCollectionViewFlowLayout: UICollectionViewFlowLayout {

    override func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
        let attributes = super.layoutAttributesForElements(in: rect)

        var leftMargin = sectionInset.left
        var maxY: CGFloat = -1.0
        attributes?.forEach { layoutAttribute in
            if layoutAttribute.frame.origin.y >= maxY {
                leftMargin = sectionInset.left
            }

            layoutAttribute.frame.origin.x = leftMargin

            leftMargin += layoutAttribute.frame.width + minimumInteritemSpacing
            maxY = max(layoutAttribute.frame.maxY , maxY)
        }

        return attributes
    }
}
```

2.storyboard 에서 Collection View Flow Layout 의 클래스로 위의 클래스를 지정해준다.

<img src = "https://user-images.githubusercontent.com/69136340/118971197-14d4c780-b9aa-11eb-95e7-6d39b62aaa87.png" width="1000">

### 완성

적용하게되면 오른쪽 뷰처럼 왼쪽 정렬이 된다.
<p>
<img src = "https://user-images.githubusercontent.com/69136340/118971212-1900e500-b9aa-11eb-8b5c-2ecb3689dae4.png" width="250">

<img src = "https://user-images.githubusercontent.com/69136340/118971209-18684e80-b9aa-11eb-8ff4-e55064902d17.png" width="250">
</p>
