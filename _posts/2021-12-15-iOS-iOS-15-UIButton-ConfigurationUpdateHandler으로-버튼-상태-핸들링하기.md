---
title:  "iOS) iOS 15 + UIButton.ConfigurationUpdateHandler 으로 버튼 상태 핸들링하기"
categories:
- iOS

date:   2021-12-15  10:58:00 +0900
author_profile: false
---
iOS 15 에서 사용하는 UIButton.ConfigurationUpdateHandler 를 가지고 버튼의 상태에 따라서 대응해보도록 했다.

```swift
enum ButtonState {
        case enable
        case disable
    }
    
    var completeButtonIsEnabled: ButtonState = .disable {
        didSet {
            if completeButtonIsEnabled == .disable {
                completeButton.isEnabled = false
                if #available(iOS 15.0, *) {
                    completeButton.setNeedsUpdateConfiguration()
                }
            } else {
                completeButton.isEnabled = true
                if #available(iOS 15.0, *) {
                    completeButton.setNeedsUpdateConfiguration()
                }
            }
        }
    }

// MARK: - #available(iOS 15.0, *)
        if #available(iOS 15.0, *) {
            let config = UIButton.Configuration.filled()
            completeButton.configuration = config
            
            let configHandler: UIButton.ConfigurationUpdateHandler = { button in
                switch button.state {
                case .disabled:
                    button.configuration?.title = "완료"
                    // button.configuration?.baseBackgroundColor = .inputBlack2
                    // button.configuration?.baseForegroundColor = .hintGray1
                default:
                    button.configuration?.title = "완료"
                    // button.configuration?.baseBackgroundColor = .mainBlue
                    // button.configuration?.baseForegroundColor = .white1
                }
            }
            completeButton.configurationUpdateHandler = configHandler
        } else {
            completeButton.setTitle("완료", for: .normal)
            // completeButton.setTitleColor(.white1, for: .normal)
        //        completeButton.setBackgroundImage(, for: .normal)
        
        // completeButton.setTitleColor(.hintGray1, for: .disabled)
        //        completeButton.setBackgroundImage(, for: .disabled)
        }
```
