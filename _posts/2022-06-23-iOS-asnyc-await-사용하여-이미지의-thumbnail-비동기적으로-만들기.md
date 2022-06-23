---
title:  "iOS) asnyc/await 사용하여 이미지의 thumbnail 비동기적으로 만들기"
categories:
- iOS

date:   2022-06-23  21:51:00 +0900
author_profile: false
---
- iOS 15 부터 적용이 가능한 `prepareThumbnail(of:completionHandler:)`  메서드를 사용해서 동기적 코드에서 background 스레드에서 비동기적으로 thumbnail image 를 만드는 것을 해보자!
- asnyc 로 선언된 비동기적 메서드인 `byPreparingThumbnail(ofSize:)` 를 사용해보자!
- debug navigator 로 CPU, Memory 에 실제로 유효한지 확인해보자!

[Meet async/await in Swift - WWDC21 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2021/10132/)

WWDC 21 세션을 보다가 비동기적으로 thumbnail image 를 만드는 메서드가 보여서 적용해보기로 하였다.

먼저 개발자 문서를 확인해보자.

## [prepareThumbnail(of:completionHandler:)](https://developer.apple.com/documentation/uikit/uiimage/3750845-preparethumbnail)

Creates a thumbnail image at the specified size asynchronously on a background thread.

<img width="500" alt="1" src="https://user-images.githubusercontent.com/69136340/175293864-69f4e7c2-fa6a-4e72-8752-0fe2f05c637a.png">

### Discussion

**Concurrency Note**

completion handler 를 사용하여 동기 코드에서 이 메서드를 호출하거나 다음 선언이 있는 비동기 메서드로 호출할 수 있습니다.

```swift
func byPreparingThumbnail(ofSize size: CGSize) async -> UIImage?
```

UIImageView 에 이미지를 표시할 때, view 의 contentMode 프로퍼티를 사용하여 이미지를 자동으로 자르거나 크기를 조정할 수 있습니다. 그러나 기본 이미지 크기가 뷰의 bounds 보다 훨씬 큰 경우 전체 크기 이미지를 디코딩하면 불필요한 메모리 오버헤드가 생성됩니다. 이 방법을 사용하여 지정된 크기로 thumbnail 이미지를 생성하면 전체 크기로 이미지를 디코딩하는 오버헤드를 피할 수 있습니다.

이 메서드는 백그라운드 스레드에서 thumbnail 이미지를 비동기적으로 생성하고 해당 스레드에서 completion handler 를 호출합니다. 앱이 completion handler 에서 UI 를 업데이트하는 경우에는 main thread 에서 UI 업데이트를 예약해야 합니다.

```swift
func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
    guard let cell = collectionView.dequeueReusableCell(withReuseIdentifier: cellIdentifier, for: indexPath) as? ItemCell else {
        fatalError("Unexpected type for cell. Check configuration.")
    }
        
    let item = items[indexPath.item]
    cell.nameLabel?.text = item.name
    item.image.prepareThumbnail(of: thumbnailSize) { thumbnail in
        DispatchQueue.main.async {
            cell.thumbnailImageView?.image = thumbnail
        }
    }
    return cell
}
```



## byPreparingThumbnail(ofSize:)

---

해당 메서드는 개발자 문서에 별도로 페이지가 존재하지 않고, 위의 개발자 문서에서만 확인할 수 있었습니다. 동일한 기능이지만 asnyc context 에서 사용할 수 있는 점이 달랐습니다.

async 메서드인 `byPreparingThumbnail(ofSize:)` 를 사용해 보겠습니다.

[Meet async/await in Swift - WWDC21 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2021/10132/)

위의 세션의 코드를 참고하여 진행하겠습니다.

```swift
import UIKit

 extension UIImage {
     var thumbnail: UIImage? {
         get async {
             let size = CGSize(width: 100, height: 140)
             return await self.byPreparingThumbnail(ofSize: size)
         }
     }
 }

// 사용
guard let thumbnailImage = await cache[url]?.thumbnail else { throw ImageDownloadError.unsupportImage }

```

read-only properties 는 asnyc 가 가능합니다.

```swift
import UIKit

actor ImageDownloader {
    static let shared = ImageDownloader()
    private init() { }
    
    private var cache: [URL: UIImage] = [:]
    
    func image(from urlPath: String) async throws -> UIImage? {
        guard let url = URL(string: Const.Path.imageURLPath + urlPath) else {
            throw ImageDownloadError.invalidURLString(Const.Path.imageURLPath + urlPath)
        }
        
        if let cached = cache[url] {
            return cached
        }
        
        let image = try await downloadImage(from: url)
        
        cache[url] = cache[url, default: image]
        
        // 🔥 캐싱된 이미지의 thumbnail 을 만들어서 반환.
        guard let thumbnailImage = await cache[url]?.thumbnail else { throw ImageDownloadError.unsupportImage }
        
        return thumbnailImage
    }

    private func downloadImage(from url: URL) async throws -> UIImage {
        let imageFetchProvider = ImageFetchProvider.shared
        return try await imageFetchProvider.fetchImage(with: url)
    }
}
```

## 적용 결과

개선 되었는지 확인해보기 위해서 debug navigator 를 확인해보았다.

### 조건)
<img src="https://user-images.githubusercontent.com/69136340/175294075-1e358a0c-430d-43d6-8072-f0fa6b7e984b.gif" width ="250">

- 아래로 끝까지 스크롤 한 후, 위로 다시 한번 스크롤하였다.
- 캐싱된 이미지와 thumbnail 을 사용하여 CPU와 Memory 를 절약해보자.

**thumbnail 사용 전)**
<img width="700" alt="2" src="https://user-images.githubusercontent.com/69136340/175294268-5f26059d-4447-43a3-85c2-aa881365ad99.png">

<img width="700" alt="3" src="https://user-images.githubusercontent.com/69136340/175294278-d28bf473-9100-4470-a410-7b73420fe6e1.png">

**thumbnail 사용 후)**

<img width="700" alt="4" src="https://user-images.githubusercontent.com/69136340/175294292-02261ef8-6864-40b3-8cc4-59c6cd6d97a5.png">

<img width="700" alt="55" src="https://user-images.githubusercontent.com/69136340/175294299-8d32570c-5902-489e-b6ff-7887abf884c7.png">

…? thumbnail 을 사용한 후 CPU, Memory 둘 다 소폭 줄었지만.. 후반부에 CPU 를 더 많이 사용하는 것을 확인 할 수 있었다.(밑으로 내린 스크롤을 위로 올릴 때)

## 개선

```swift
func image(from urlPath: String) async throws -> UIImage? {
        guard let url = URL(string: Const.Path.imageURLPath + urlPath) else {
            throw ImageDownloadError.invalidURLString(Const.Path.imageURLPath + urlPath)
        }
        
        if let cached = cache[url] {
            return cached
        }
        
        let image = try await downloadImage(from: url)
        
        // 🔥 thumbnail 이미지를 캐싱해주도록 변경.
        // 🔥 이전에는 캐싱된 이미지는 그대로. 캐싱되지 않은 이미지는 thumbnail 로 변경해서 반환했다.(즉, 캐싱되지 않은 경우만 thumbnail 로 변경)
        guard let thumbnailImage = await image.thumbnail else { throw ImageDownloadError.unsupportImage }

        cache[url] = cache[url, default: thumbnailImage]
        
        return cache[url]!
    }
```
<img width="700" alt="6" src="https://user-images.githubusercontent.com/69136340/175294463-db55f886-3590-46c1-8a3a-cf06860b82d6.png">

<img width="700" alt="7" src="https://user-images.githubusercontent.com/69136340/175294473-28174a20-d22b-4780-b5c9-6c868e788d1e.png">

***매번 결과가 동일하지는 않겠지만 전체적으로 보았을 때 CPU 와 Memory 가 확실히 줄었습니다.***

### 느낀점

평소라면 그냥 예상되는 결과로써 넘어갈 수 있을 부분들인데 실제로 확인을 해보니까 간단하게 내가 작성한 코드가 얼마나 유효한지 알 수 있었다. 또한,  이를 통해서 원인을 찾고 코드도 개선할 수 있었다.

코드를 보고 결과를 예측하고, 시뮬레이터에서 느낌적으로 알고 넘어가던 것들을 실제로 수치적으로 확인할 수 있어서 코드로 결과를 기대하는 것이 아닌 실질적인 결과를 얻을 수 있어서 뿌듯했다.

**출처:** 

[Apple Developer Documentation - preparethumbnail](https://developer.apple.com/documentation/uikit/uiimage/3750845-preparethumbnail)
