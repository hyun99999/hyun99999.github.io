---
title:  "iOS) YPImagePicker 사용기"
categories:
- iOS

date:   2023-04-17  13:30:00 +0900
author_profile: false
---
### 내용

- 다음과 같이 YPImagePicker 를 통해 구현해보자.
    - 사진 라이브러리에서만 이미지를 가져온다.
    - 하나의 사진만 선택할 수 있도록 한다.
    - 이전에 고른 사진이 선택된다.
    - crop 할 수 있는 비율을 커스텀한다.
    - 필터는 사용하지 않는다.
    - 이미지를 따로 저장하지 않는다.
    - 이미지 선택 시와 취소 시에 notification 을 post 한다.

<img src="https://user-images.githubusercontent.com/69136340/232383595-0f987e83-6f0d-4bde-af80-9cb22a026251.mp4" width ="250">

### 우선, YPImagePicker 를 사용하는 이유

이전에는 crop 에 대한 기능을 기대하며 UIImagePicker 를 사용하였는데 크기를 커스텀할 수 없었습니다.

많은 개발자들의 아쉬움을 여러 글들에서 읽을 수 있었고 다음의 이유로 YPImagePicker 를 선택하였습니다.

원하는 사이즈로 crop 할 수 있는 이미지 라이브러리는 많이 없었고, 그 중 많은 사람들의 사용기가 있으며 가장 최근까지 관리되고 있는 라이브러리를 선택하였습니다.

이외에도 image picker 에 전반적인 text, font, color 등 프로젝트의 톤과 맞는 UI 로 구현할 수 있을 것이라고 기대하였습니다.

사용 후 느낀점까지 작성해보았습니다.

## 1️⃣ YPImagePicker

[https://github.com/Yummypets/YPImagePicker](https://github.com/Yummypets/YPImagePicker)

깃허브 리드미를 참고하여 간단하게 YPImagePicker 의 사용법을 정리하였습니다.

## intallation

- CocoaPods

```
pod 'YPImagePicker'
```

## 사용 방법

```swift
// 
var config = YPImagePickerConfiguration()

// configuration 을 만들고 picker 들에 일괄적으로 적용할 수 있는 장점으로 연결된다.
YPImagePickerConfiguration.shared = config

// And then use the default configuration like so:
let picker = YPImagePicker()

// confuration 으로 image picker 를 만들 수 있다.
let imagePicker = YPImagePicker(configuration: config)
```

### plist 수정

카메라와 사진 라이브러리를 사용하기 위해서 다음과 같은 권한을 허용받아야 합니다.

- Privacy - Camera Usage Description (photo/videos)
- Privacy - Photo Library Usage Description (library)
- Privacy - Microphone Usage Description (videos)

### Single Photo

리드미에서 다음과 같이 코드를 제공하고 있었습니다. 이 중에 사용할 것들을 적용해보았습니다.

```swift
let picker = YPImagePicker()
picker.didFinishPicking { [unowned picker] items, _ in
    if let photo = items.singlePhoto {
        print(photo.fromCamera) // Image source (camera or library)
        print(photo.image) // ✅ Final image selected by the user
        print(photo.originalImage) // original image selected by the user, unfiltered
        print(photo.modifiedImage) // Transformed image, can be nil
        print(photo.exifMeta) // Print exif meta data of original image.
    }
    picker.dismiss(animated: true, completion: nil)
}
present(picker, animated: true, completion: nil)
```

## 2️⃣ 구현

목표한 구현 내용을 목표로해서 구현해보겠습니다.

default 값들이 잘 들어가있어서 구현하고자하는 부분만 설정해주는 느낌이었습니다.

```swift
private lazy var selectedImage: [YPMediaItem] = []

    private func presentToImagePicker() {
        var config = YPImagePickerConfiguration()
        
//        주요 설정해야하는 default 값.
//        config.library.mediaType = .photo
//        config.library.defaultMultipleSelection = false
//        config.library.maxNumberOfItems = 1
        
        config.screens = [.library]
        config.startOnScreen = .library
        
        // cropping style 을 square or not 으로 지정.
        config.library.isSquareByDefault = false
        
        // 필터 단계 스킵.
        config.showsPhotoFilters = false
        
        // crop overlay 의 default 색상.
//      config.colors.cropOverlayColor = .ypSystemBackground.withAlphaComponent(0.4)
        // 327 * 540 비율로 crop 희망.
        config.showsCrop = .rectangle(ratio: 0.6)
        
        // 이전에 선택한 이미지가 pre-selected 되도록 함.
//       config.library.preselectedItems = selectedImage

        // 새 이미지를 사진 라이브러리에 저장하지 않음.
        // 👉 저장하지 않으면 selectedImage 에 담긴 이미지가 사진 라이브러리에서 찾을 수가 없어서 가장 앞에 이미지를 선택함.
        config.shouldSaveNewPicturesToAlbum = false
        
        let imagePicker = YPImagePicker(configuration: config)
        imagePicker.imagePickerDelegate = self
        
//      imagePicker.didFinishPicking(completion: YPImagePicker.DidFinishPickingCompletion)
//      public typealias DidFinishPickingCompletion = (_ items: [YPMediaItem], _ cancelled: Bool) -> Void
        imagePicker.didFinishPicking { [weak self] items, cancelled in
            guard let self = self else { return }
            
            if cancelled {
                NotificationCenter.default.post(name: .cancelImagePicker, object: nil)
            }
            
//          selectedImage = items
            if let photo = items.singlePhoto {
                backgroundImage = photo.image
                NotificationCenter.default.post(name: .sendNewImage, object: backgroundImage)
            }
            imagePicker.dismiss(animated: true)
        }
        
        imagePicker.modalPresentationStyle = .overFullScreen
        present(imagePicker, animated: true, completion: nil)
    }

// MARK: - YPImagePickerDelegate
extension CardCreationViewController: YPImagePickerDelegate {
    func imagePickerHasNoItemsInLibrary(_ picker: YPImagePicker) {
        self.makeOKAlert(title: "", message: "가져올 수 있는 사진이 없습니다.")
    }
    
    func shouldAddToSelection(indexPath: IndexPath, numSelections: Int) -> Bool {
        // false 로 설정하면 선택해도 다음으로 갈 수 없다. 즉, 추가할 수 없음.
        return true
    }
}
```

**목표했던 내용들을 체크해보겠습니다.**

- 사진 라이브러리에서만 이미지를 가져온다.
- 하나의 사진만 선택할 수 있도록 한다.
- 이전에 고른 사진이 선택된다. **→ 이미지를 사진 라이브러리에 저장하지 않기 때문에 이전 사진은 고를 수 없음.**
- crop 할 수 있는 비율을 커스텀한다.
- 필터는 사용하지 않는다.
- 이미지를 사진 라이브러리에 저장하지 않는다.
- 이미지 선택 시와 취소 시에 notification 을 post 한다.

## 느낀 점

YPImagePicker 를 처음 접했던 때는 iOS 에서 이미지의 다중 선택을 지원하지 않는 것 때문이었습니다.

그러던 중 iOS 14 부터 사용할 수 있는 PHPicker 가 생겨서 이후에는 많은 사용자들이 옮겨갈 수 있었습니다.

하지만, 아직까지 crop 에 대한 기능은 제공하지 않고 있기 때문에 해당 라이브러리를 사용하게 되었습니다.

사용해보니 리드미와 example 파일의 주석이 친절하였습니다. 또한, 프로퍼티의 기본값이 잘 설정되어 있어서 적은 줄의 코드로 범용성있는 기능을 구현할 수 있었습니다.

- 기존의 UIImagePicker 와 달리 선택 후의 동작을 delegate method 에서 구현하는 것이 아니라 `didFinishPicking` 로 따로 구현할 수 있어서 코드의 흐름을 읽는데 같은 문맥에 둘 수 있었습니다.
