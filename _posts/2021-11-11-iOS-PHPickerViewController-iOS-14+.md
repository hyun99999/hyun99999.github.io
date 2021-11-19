---
title:  "iOS) PHPickerViewController iOS 14+"
categories:
- iOS

date:   2021-11-15  19:37:00 +0900
author_profile: false
---
기존에 지원하는 UIPickerViewController 가 있었는데요! iOS 14 에서 새로운 Photo Picker 가 나왔슴다!

[Apple Developer Documentation](https://developer.apple.com/documentation/photokit/phpickerviewcontroller)

자자 개발자 문서를 살펴보자구요!

### Overview

`PHPickerViewController` 클래스는 `UIImagePickerController` 의 대안입니다. `PHPickerViewController` 는 안정성과 신뢰성을 개선하고 다음과 같은 개발자와 사용자에게 이점을 제공합니다.

- Deferred image loading and recovery UI

지연된 이미지 로딩과 복구 UI

- Reliable handling of large and complex assets, like RAW and panoramic images

RAW 및 파노라마 이미지와 같은 크고 복잡한 에셋의 안정적인 처

(RAW 는 이미지 파일 포맷이다. 다른 확장자들보다 크기가 매우 크지만 사진편집에 유용하다고 한다. iPhone 12 프로 및 프로맥스에서부터 촬영가능.)

- User-selectable assets that aren’t available for `UIImagePickerController`

UIImagePickerController 에 사용할 수 없는 사용자선택 가능한 에셋

- Configuration of the picker to display only Live Photos

Live Photos 만 표시하는 picker 구성

- Availability of [PHLivePhoto](https://developer.apple.com/documentation/photokit/phlivephoto) objects without library access

라이브러리 액세스 없이 PHLivePhoto(Live Photo) 객체 사용가능

- Stricter validations against invalid inputs

유효하지 않는 입력에 대한 더 엄격한 검증

**이렇게 보니까 기능면에서 무엇이 확실히 다른지 모르겠는데요! 만들어보기전에 먼저 소개해보겠습니다!**

- multiple select
- zoom in or out
- search
- 권한 요청 팝업이 뜨지 않는다.

엄청..나지 않나요..? 휘둥그레

# 출발

### 1. import PhotosUI

```swift
import PhotosUI
```

### 2. Create PHPickerConfiguration

: picker 뷰컨트롤러를 구성하는 정보를 포함한 객체입니다.

이 객체로 제어 할 수 있는데 그 종류를 알아보자!

```swift
var configuration = PHPickerConfiguration()
// 🎆 selectionLimit
// 🎆 유저가 선택할 수 있는 에셋의 최대 갯수. 기본값 1. 0 설정시 제한은 시스템이 지원하는 최대값으로 설정.
configuration.selectionLimit = 1

// 🎆 filter
// 🎆 picker 가 표시하는 에셋 타입 제한을 적용. 기본적으로 모든 에셋 유형을 표시(이미지, 라이브포토, 비디오)
configuration.filter = .images
// configuration.filter = .any(of: [.images, .livePhotos, .videos])
```

### 3. Initialize PHPicker

만든 PHPickerConfiguration 을 가지고 PHPickerViewController 를 만들어야 합니다.

```swift
let picker = PHPickerViewController(configuration: configuration)
```

### 4. PHPickerViewControllerDelegate

`UIImagePickerController` 도 선택하고 `didFinishPickingMediaWithInfo` 메서드로 선택한 미디어의 정보를 가져왔는데 PHPicker 도 동일한 역할이 있다.

```swift
// 🎆 유저가 선택을 완료했거나 취소 버튼으로 닫았을 때 알려주는 delegate
extension ViewController: PHPickerViewControllerDelegate {
    // 🎆 required method
    func picker(_ picker: PHPickerViewController, didFinishPicking results: [PHPickerResult]) {
        // ...
    }
}
```

`PHPickerViewControllerDelegate` 을 채택해주고 역시나 `picker.delegate = self` 설정해주어야 한다.

### 5. Present Picker

이제 피커를 뷰에 띄어봅시다! 아래는 전체 코드입니다.

```swift
class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        
        var configuration = PHPickerConfiguration()
        // 🎆 The maximum number of selections the user can make.
        // 🎆 기본값 1. 0 설정시 제한은 시스템이 지원하는 최대값으로 설정.
        configuration.selectionLimit = 2
        
        // 🎆 The filter you apply to restrict the asset types the picker displays.
        // 🎆 기본적으로 이미지(라이브포토 포함), 라이브포토, 비디오와 같은 모든 에셋타입 표시.
        configuration.filter = .images
//        configuration.filter = .any(of: [.images, .livePhotos, .videos])
        
        let picker = PHPickerViewController(configuration: configuration)
        picker.delegate = self
        
        self.present(picker, animated: true, completion: nil)
    }
}

// 🎆 유저가 선택을 완료했거나 취소 버튼으로 닫았을 때 알려주는 delegate
extension ViewController: PHPickerViewControllerDelegate {
    // 🎆 required method
    func picker(_ picker: PHPickerViewController, didFinishPicking results: [PHPickerResult]) {
        // ...
    }
}
```

### 6. Handling assets

이제 선택한 에셋들을 다루어보겠습니다.

delegate 에서 파라미터 results 를 사용하면된답니다!

- PHPickerResult : 선택된 에셋을 나타내는 타입.
- itemProvider: **[NSItemProvider](https://developer.apple.com/documentation/foundation/nsitemprovider).** 선택된 에셋의 respresentation.

```swift

extension ViewController: PHPickerViewControllerDelegate {
    func picker(_ picker: PHPickerViewController, didFinishPicking results: [PHPickerResult]) {
        // 🎆 선택완료 혹은 취소하면 뷰 dismiss.
        picker.dismiss(animated: true, completion: nil)
        
        // 🎆 itemProvider 를 가져온다.
        let itemProvider = results.first?.itemProvider
        if let itemProvider = itemProvider,
           // 🎆 itemProvider 에서 지정한 타입으로 로드할 수 있는지 체크
           itemProvider.canLoadObject(ofClass: UIImage.self) {
            // 🎆 loadObject() 메서드는 completionHandler 로 NSItemProviderReading 과 error 를 준다.
            itemProvider.loadObject(ofClass: UIImage.self) { image, error in
                // 🎆 itemProvider 는 background asnyc 작업이기 때문에 UI 와 관련된 업데이트는 꼭 main 쓰레드에서 실행해줘야 합니다.
                DispatchQueue.main.sync {
                    self.myImageView.image = image as? UIImage
                }
            }
        }
    }
}
```

### 7. Multiple selection Handling

다음과 같이 `PHPickerResult` 배열에서 꺼내주면 된다.

```swift
    func picker(_ picker: PHPickerViewController, didFinishPicking results: [PHPickerResult]) {
        picker.dismiss(animated: true, completion: nil)   
    
        let itemFirstProvider = results[0].itemProvider
        let itemSecondProvider = results[1].itemProvider
        
        if itemFirstProvider.canLoadObject(ofClass: UIImage.self) {
            itemFirstProvider.loadObject(ofClass: UIImage.self) { image, error in
                DispatchQueue.main.sync {
                    self.myFirstImageView.image = image as? UIImage
                }
            }
        }
        
        if itemSecondProvider.canLoadObject(ofClass: UIImage.self) {
            itemSecondProvider.loadObject(ofClass: UIImage.self) { image, error in
                DispatchQueue.main.sync {
                    self.mySecondImageView.image = image as? UIImage
                }
            }
        }
    }
```

### 결과

<img src ="https://user-images.githubusercontent.com/69136340/141780108-70dddc9e-6fda-4b16-9e16-45c9769edfab.gif" width ="250">

출처 : 

[iOS 14+ ) PHPicker](https://zeddios.tistory.com/1052)
