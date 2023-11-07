---
title:  "iOS) YPImagePicker error :  Stored properties cannot be marked unavailable with `@available`"
categories:
- iOS

date:   2023-09-28  23:43:00 +0900
author_profile: false
---
collection view 에서 선택된 셀에 대해서 알아보고자 할 때 **indexPathsForSelectedItems** 를 사용하였습니다.

개발자 문서를 살펴보겠습니다.

[indexPathsForSelectedItems | Apple Developer Documentation](https://developer.apple.com/documentation/uikit/uicollectionview/1618099-indexpathsforselecteditems)

The index paths for the selected items.

선택한 항목이 없다면 nil 입니다.

```swift
var indexPathsForSelectedItems: [IndexPath]? { get }

// Optional([[0, 0], [0, 1]]) 와 같이 출력.
```

다음과 같은 기능을 구현하고자 하였습니다.
- cell 이 하나라도 선택될 때 cancelButton 은 등장하고, deleteButton 은 활성화
- cell 이 모두 선택 해제될 때 cancelButton 은 사라지고, deleteButton 은 비활성화

```swift
xtension FetchTagSheetVC: UICollectionViewDelegate {
    func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {
        if collectionView.indexPathsForSelectedItems == nil {
            // 선택된 셀이 없다면(선택 해제 시도를 고려하였다.) cancelButton 은 사라지고, deleteButton 은 비활성화 시킴.
            cancelButton.isHidden = true
            deleteButton.isEnabled = false
        } else {
            // 선택된 셀이 있다면 cancelButton 은 등장하고, deleteButton 은 활성화 시킴.
            cancelButton.isHidden = false
            deleteButton.isEnabled = true
        }
    }
}
```

### 트러블 슈팅

기대하던 결과와는 다르게 cell 을 모두 선택 해제했을 때 UI가 변하지 않았습니다. 뭐가 잘못되었는지 살펴보겠습니다.

<img src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/044532c5-5a0f-4041-b0a0-f63c6d40ee1a" width ="250">

**우선, 두 가지가 적절하지 않게 고려되었습니다.**

- 선택할 때 호출되는 delegate method 에 선택이 해제되는 경우는 적용되지 않습니다.
- 선택한 셀을 해제하여 **indexPathsForSelectedItems** 출력해보니 nil 이 아닌 빈 배열이 출력되었습니다.

**❗️ indexPathsForSelectedItems 프로퍼티는 선택한 항목이 없다면 nil 이지만, 한 번이라도 선택후에 해제한다면 빈 배열을 가지게 되는 프로퍼티였습니다!**

- 그래서 다음과 같이 구현하였습니다.

```swift
xtension FetchTagSheetVC: UICollectionViewDelegate {
    func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {
        // ✅ 셀이 선택될 때 호출
        cancelButton.isHidden = false
        deleteButton.isEnabled = true
    }

    func collectionView(_ collectionView: UICollectionView, didDeselectItemAt indexPath: IndexPath) {
        // ✅ 셀이 선택 해제될 때 호출
        // ❗️빈 배열에서 UI 변화
        if collectionView.indexPathsForSelectedItems?.isEmpty == true {
            cancelButton.isHidden = true
            deleteButton.isEnabled = false
        }
    }
}    
```

### 결과

<img src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/853bcc37-0b15-4c05-9a30-68d1a627d17e" width ="250">
