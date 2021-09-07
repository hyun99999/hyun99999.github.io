---
title:  "iOS) UIImage 를 Data 로 변환해서 UserDefaults 에 저장하는 것은 부적합하다?"
categories:
- iOS

date:   2021-09-06  17:30:00 +0900
author_profile: false
---
## 📌 UIImage 를 Data 로 변환하는 것은 부적합하다?

UIImage 를 Data 로 변환하는 것에 의문을 가진 게시글이 있었다. 참고해보자.

[How to Save an Image in User Defaults in Swift](https://cocoacasts.com/ud-9-how-to-save-an-image-in-user-defaults-in-swift)

그 이유는 UserDefaults 를 이미지 데이터와 같이 대량의 데이터를 저장하는데 사용하기에는 부적합하다는 것이다.

그리고 나는 이러한 문제를 바로 최근에 직면했었다.

[iOS) UserDefaults 에 image 저장하기](https://gyuios.tistory.com/101)

## 📌 그렇다면?

위의 글에서는 이렇게 제안한다.

application's sandbox 에 이미지를 저장하고 UserDefaults 에 이미지의 위치를 저장하는 것이다. 즉, FileManager 를 사용해서 이미지를 저장하고 그 경로를 UserDefaults 에 저장하는 것이다.

```swift
import UIKit

// Load Image
let image = UIImage(named: "image_name")

// Convert to Data
if let data = image?.pngData() {
    // Create URL
    let documents = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
    let url = documents.appendingPathComponent("image_name.png")

    do {
        // Write to Disk
        try data.write(to: url)

        // Store URL in User Defaults
        UserDefaults.standard.set(url, forKey: "image")

    } catch {
        print("Unable to Write Data to Disk (\(error))")
    }
}
```
