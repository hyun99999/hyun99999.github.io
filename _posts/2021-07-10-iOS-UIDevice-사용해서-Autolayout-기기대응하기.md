---
title:  "iOS) viewDidAppear() 에서 화면전환 코드 작성하기"
categories:
- iOS

date:   2021-07-10  00 :04:00 +0900
author_profile: false
---
UIDevice 를  extension 으로 아이폰의 기종유무를 저장하는 변수를 만든다.

```swift
import UIKit

extension UIDevice {
    public var isiPhoneSE: Bool {
        if UIDevice.current.userInterfaceIdiom == UIUserInterfaceIdiom.phone && (UIScreen.main.bounds.size.height == 568 && UIScreen.main.bounds.size.width == 320) {
            return true
        }
        return false
    }
    
    public var isiPhoneSE2: Bool {
        if UIDevice.current.userInterfaceIdiom == UIUserInterfaceIdiom.phone && (UIScreen.main.bounds.size.height == 667 && UIScreen.main.bounds.size.width == 375) {
            return true
        }
        return false
    }
    
    public var isiPhone8Plus: Bool {
        if UIDevice.current.userInterfaceIdiom == UIUserInterfaceIdiom.phone && (UIScreen.main.bounds.size.height == 736 || UIScreen.main.bounds.size.width == 414) {
            return true
        }
        return false
    }
    
    public var isiPhone12mini: Bool {
        if UIDevice.current.userInterfaceIdiom == UIUserInterfaceIdiom.phone && (UIScreen.main.bounds.size.height == 812 && UIScreen.main.bounds.size.width == 375) {
            return true
        }
        return false
    }
    
    public var isiPone12Pro: Bool {
        if UIDevice.current.userInterfaceIdiom == UIUserInterfaceIdiom.pad && (UIScreen.main.bounds.size.height == 844 && UIScreen.main.bounds.size.width == 390) {
            return true
        }
        return false
    }
}
```

```swift
// MARK: - UIComponents
    @IBOutlet weak var customNavigationBarViewHeight: NSLayoutConstraint!
    @IBOutlet weak var progressBarLeftAnchor: NSLayoutConstraint!
    @IBOutlet weak var indexLeftAnchor: NSLayoutConstraint!
    @IBOutlet weak var indexRightAnchor: NSLayoutConstraint!
    @IBOutlet weak var guideLabel1LeftAnchor: NSLayoutConstraint!
    @IBOutlet weak var guideLabel1BottomAnchor: NSLayoutConstraint!
    @IBOutlet weak var guideLabel1TopAnchor: NSLayoutConstraint!
    @IBOutlet weak var genderListCollectionViewTopAnchor: NSLayoutConstraint!
    @IBOutlet weak var genderListCollectionViewLeftAnchor: NSLayoutConstraint!
    @IBOutlet weak var genderListCollectionViewRightAnchor: NSLayoutConstraint!
    @IBOutlet weak var guideLable3TopAnchor: NSLayoutConstraint!
    @IBOutlet weak var nextButtonBottomAnchor: NSLayoutConstraint!
    @IBOutlet weak var genderListCollectionViewRatio: NSLayoutConstraint!

override func viewDidLoad() {
        super.viewDidLoad()

        setPhoneResolution()
    }

func setPhoneResolution(){
        if UIDevice.current.isiPhoneSE2 {
            customNavigationBarViewHeight.constant = 50
            
            progressBarLeftAnchor.constant = 16
            indexLeftAnchor.constant = 9
            indexRightAnchor.constant = 18
            
            guideLabel1TopAnchor.constant = 29
            guideLabel1LeftAnchor.constant = 16
            guideLabel1BottomAnchor.constant = 6
            
            genderListCollectionViewTopAnchor.constant = 15
            genderListCollectionViewLeftAnchor.constant = 11
            genderListCollectionViewRightAnchor.constant = 11
            
            guideLable3TopAnchor.constant = 31

            nextButtonBottomAnchor.constant = 28
        }
```
