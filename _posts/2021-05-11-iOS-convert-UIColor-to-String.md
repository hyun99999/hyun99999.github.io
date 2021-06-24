---
title:  "iOS) convert UIColor to String"
categories:
- iOS

date:   2021-05-11 12:00:00 +0900
author_profile: false
---
convert UIColor to String

- DB 에 색을 저장하려고 하니까 String 으로 변환해줄 필요가 생겼다.

```swift
import Foundation
import UIKit

extension UIColor {
    
    convenience init(hex: String, alpha: CGFloat = 1.0) {
        var hexFormatted: String = hex.trimmingCharacters(in: CharacterSet.whitespacesAndNewlines).uppercased()

        if hexFormatted.hasPrefix("#") {
            hexFormatted = String(hexFormatted.dropFirst())
        }

        assert(hexFormatted.count == 6, "Invalid hex code used.")

        var rgbValue: UInt64 = 0
        Scanner(string: hexFormatted).scanHexInt64(&rgbValue)

        self.init(red: CGFloat((rgbValue & 0xFF0000) >> 16) / 255.0,
                  green: CGFloat((rgbValue & 0x00FF00) >> 8) / 255.0,
                  blue: CGFloat(rgbValue & 0x0000FF) / 255.0,
                  alpha: alpha)
    }
    
    func toHexString() -> String {
        var r:CGFloat = 0
        var g:CGFloat = 0
        var b:CGFloat = 0
        var a:CGFloat = 0
        getRed(&r, green: &g, blue: &b, alpha: &a)
        let rgb:Int = (Int)(r*255)<<16 | (Int)(g*255)<<8 | (Int)(b*255)<<0
        
        return String(format:"#%06x", rgb)
    }
}
```

### 사용방법
```swift
//사용방법
var listBulletBtnColor = "#007aff"
listBulletBtn.backgroundColor = UIColor(hex: listBulletBtnColor)

if let color = listBulletBtn.backgroundColor {
    database.reminderColor = color.toHexString()
```
