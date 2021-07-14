---
title:  "iOS) PanModal 오픈 라이브러리를 사용해서 모달창 만들기"
categories:
- iOS

date:   2021-07-12  11:19:00 +0900
author_profile: false
---
- 모달창을 띄우고 싶은 뷰컨

```swift
import PanModal

//...

// MARK: - @IBAction Properties
    @IBAction func sortButton(_ sender: Any) {
                let storyboard = UIStoryboard(name: Const.Storyboard.Name.SortPanModal, bundle: nil)
        let vc = storyboard.instantiateViewController(withIdentifier: Const.ViewController.Name.SortPanModal) as! SortPanModalVC
        self.presentPanModal(vc)
    }
```

- 모달창 뷰컨

```swift
import PanModal

class SortPanModalVC: UIViewController {
    
    // MARK: - View Life Cycle
    override func viewDidLoad() {
        super.viewDidLoad()
// ...
    }
    
// ...

}
// MARK: - PanModalPresentable
extension SortPanModalVC: PanModalPresentable {

    // 스크롤되는 tableview 나 collectionview 가 있다면 여기에 넣어주면 PanModal 이 모달과 스크롤 뷰 사이에서 팬 제스처를 원활하게 전환합니다.
    var panScrollable: UIScrollView? {
        return nil
    }
    
    var shortFormHeight: PanModalHeight {
        return .contentHeight(280)
    }

    var longFormHeight: PanModalHeight {
        return .maxHeightWithTopInset(0)
    }
}
```

### 완성
<img src = "https://user-images.githubusercontent.com/69136340/125229580-0eabf900-e312-11eb-85b1-ffe4726d7168.png" width = "250"

### 모달창 높이 속성값
- contentHeight

<img width="375" alt="2" src="https://user-images.githubusercontent.com/69136340/125230008-ea045100-e312-11eb-8a3e-7f9a707abbce.png">

- contentHeightIgnoringSafeArea

<img width="376" alt="3" src="https://user-images.githubusercontent.com/69136340/125230003-e7a1f700-e312-11eb-8978-c282548bfa39.png">

- intrinsicHeight

<img width="374" alt="4" src="https://user-images.githubusercontent.com/69136340/125230018-ebce1480-e312-11eb-999a-d73d17bef04d.png">

- maxHeight

<img width="376" alt="5" src="https://user-images.githubusercontent.com/69136340/125230014-eb357e00-e312-11eb-8f2d-a1abed987b79.png">

- maxHeightWithTopInset

<img width="375" alt="6" src="https://user-images.githubusercontent.com/69136340/125230007-e96bba80-e312-11eb-8d71-db9d236cfb93.png">

### 출처

[slackhq/PanModal](https://github.com/slackhq/PanModal)
