---
title:  "iOS) UICollectionView 스크롤에 따라 같이 움직이는 상단 바 구현하기"
categories:
- iOS

date:   2021-10-06  16:23:00 +0900
author_profile: false
---
**내용**

- UICollectionView 의 스크롤을 함에 따라서 같이 움직이는 상단 바를 구현해보자

### 완성

<img src ="https://user-images.githubusercontent.com/69136340/136189706-eebf6111-f1e3-472f-a539-75802f53e610.gif" width ="250">

### 🌊 애니메이션 넣기

```swift
// ✅ "뒷면" 라벨의 하단으로 이동시킬 때
UIView.animate(withDuration: 0.3) {
// 상단 바의 위치를 ("뒷면" 라벨의 x 좌표) - (상단 바의 x 좌표) 만큼 이동.
        self.statusMovedView.transform = CGAffineTransform(translationX: self.backTextLabel.frame.origin.x - self.statusMovedView.frame.origin.x + 5, y: 0)
}

// ✅ 다시 "앞면" 라벨의 하단으로 돌아올 때
UIView.animate(withDuration: 0.3) {
// 원래 위치로 원상복구
        self.statusMovedView.transform = .identity
}

```

### 🌊 스크롤하는 index 의 위치에 따른 분기 처리

```swift
func scrollViewDidEndDragging(_ scrollView: UIScrollView, willDecelerate decelerate: Bool) {
        // ✅ 스크롤뷰의 x 좌표를 가로길이로 나눠서 index 를 설정.(0(앞면), 1(뒷면))   
        let index = Int(round(scrollView.contentOffset.x / scrollView.frame.size.width))
        // "뒷면" 라벨 하단으로 이동
        if index == 1 {
            UIView.animate(withDuration: 0.3) {
                self.statusMovedView.transform = CGAffineTransform(translationX: self.backTextLabel.frame.origin.x - self.statusMovedView.frame.origin.x + 5, y: 0)
            }
        }
        // "앞면" 라벨 하단으로 이동.
// else 를 사용하지 않는 이유는 "뒷면" 에서 
        else if index == 0 {
            UIView.animate(withDuration: 0.3) {
                self.statusMovedView.transform = .identity
            }
        }
    }
```

위와 같이 구현하게 되었는데 마지막 셀에서 왼쪽으로 스와이프 할 때 다시 "앞면" 라벨 하단으로 animate 되는 경우가 생겨서 방지하는 코드도 추가했다. 

그리고 공부하면서 몇가지 delegate 메서드가 있었는데 적당한 메서드를 골라보았다.

UIScrollViewDelegate 에 대해 알아보자.

[iOS) UIScrollViewDelegate 에 대해 알아보자(Scrolling and Dragging)](https://gyuios.tistory.com/116)

### 🌊 scrollViewWillEndDragging(_:withVelocity:targetContentOffset:) 메서드 사용

```swift
private var currentIndex = 0
// ✅ 사용자가 스크롤을 멈출 때 호출.
func scrollViewWillEndDragging(_ scrollView: UIScrollView, withVelocity velocity: CGPoint, targetContentOffset: UnsafeMutablePointer<CGPoint>) {
        // ✅ targetContentOffset : 스크롤 동작이 정지할 때 예상되는 offset. (0.0(앞면으로 갈 때) 또는 375.0(뒷면으로 갈 때. iPhone12 기준.))
        let targetIndex = targetContentOffset.pointee.x / scrollView.frame.size.width
        if targetIndex == 1 && currentIndex == 0 {
            UIView.animate(withDuration: 0.3) {
                self.statusMovedView.transform = CGAffineTransform(translationX: self.backTextLabel.frame.origin.x - self.statusMovedView.frame.origin.x + 5, y: 0)
            }
            currentIndex = 1
        } else if targetIndex == 0 && currentIndex == 1 {
            UIView.animate(withDuration: 0.3) {
                self.statusMovedView.transform = .identity
            }
            currentIndex = 0
        }
    }
```
