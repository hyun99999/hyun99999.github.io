---
title:  "iOS) UIPickerView 에 고정 라벨 추가하기"
categories:
- iOS

date:   2021-12-01  16:14:00 +0900
author_profile: false
---
```swift
extension UIPickerView {
    func setPickerLabelsWith(labels: [String]) {
        let columCount = labels.count
        let fontSize: CGFloat = UIFont.textBold01.pointSize + 3
        
        var labelList: [UILabel] = []
        for index in 0..<columCount {
            let label = UILabel()
            label.text = labels[index]
            label.font = .textBold01
            label.textColor = .mainColorNadaMain
            label.sizeToFit()
            labelList.append(label)
        }
        
        let pickerWidth: CGFloat = self.frame.width
        let labelY: CGFloat = (self.frame.size.height / 2) - (fontSize / 2)
    
        for (index, label) in labelList.enumerated() {
            let labelX: CGFloat = (pickerWidth / CGFloat(columCount)) * CGFloat(index + 1) - fontSize
            label.frame = CGRect(x: labelX, y: labelY, width: fontSize, height: fontSize)
            self.addSubview(label)
        }
    }
}

// 사용
// pickerView.setPickerLabelsWith(labels: ["년","월","일"])
```
<img src="https://user-images.githubusercontent.com/69136340/144188550-98840623-9208-49e7-bfe3-fda4e01e24d5.PNG" width ="250">

