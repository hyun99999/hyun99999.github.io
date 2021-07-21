---
title:  "iOS) UICollectionView-의-vertical,horizontal-유무-programmatically-구현"
categories:
- iOS

date:   2021-07-20  22:08:00 +0900
author_profile: false
---
스토리보드에서 설정가능하지만 programmatically 하게 구현도 가능하다.

- collectionViewLayout 속성을 통해서 설정할 수 있다.

```swift
let layout = onboardingCollectionView.collectionViewLayout as? UICollectionViewFlowLayout
layout?.scrollDirection = .horizontal
//layout?.scrollDirection = .vertical
```
