---
title:  "iOS) Actor 를 활용한 이미지 캐싱처리 - WWDC21 Protect mutable state with Swift actors"
categories:
- iOS

date:   2022-06-03  19:30:00 +0900
author_profile: false
---
### 내용

- `Actor` 을 활용해서 이미지를 캐싱하는 다운로더를 만들어 보겠습니다.
- `async/await` 을 활용해서 이미지를 다운받고, 그 이후의 캐싱 역할은 `Actor` 로 만든 ImageDownloader 에서 처리하도록 하겠습니다.

아래의 세션을 참고해서 적용해보았습니다.

[Protect mutable state with Swift actors - WWDC21 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2021/10133/)

## Create ImageDownloader with Actor

이미지 다운로더 actor 를 만들어보겠습니다.

이미지 다운로더는 다른 서비스에서 이미지를 다운받는 역할을 수행하며, 다운 받은 이미지를 cache 에 저장하여 동일한 이미지에 대한 중복 다운로드를 막습니다.

- cache 를 확인하고 cache 에 이미지가 없다면 다운로드한 뒤 cache 에 저장하고 반환합니다. 이는 actor 에서 실행되는 코드이므로 `data races` 로부터 안전합니다.

```swift
actor ImageDownloader {
    private var cache: [URL: Image] = [:]

    func image(from url: URL) async throws -> Image? {
        if let cached = cache[url] {
            return cached
        }

        let image = try await downloadImage(from: url)

        // Potential bug: `cache` may have changed.
        cache[url] = image
        return image
    }
}
```

Actor 의 동기화 메커니즘은 한 번에 하나의 작업만 cache 에 접근하도록 보장하므로 캐시가 손상될 수 있는 경우는 없다고 생각했… 습니다만. **여기서 `await` 키워드가 문제를 발생시킵니다.**

`await` 는 해당 시점에서 함수가 일시 정지될 수 있음을 의미합니다. 즉, potential suspension point 를 가질 수 있다는 의미입니다.

이것은 프로그램의 다른 코드가 실행될 수 있도록 스레드 제어권을 포기해서 전체 프로그램의 상태에 영향을 줄 수 있도록 합니다. 이렇게 되면 await 이후 함수가 다시 실행되는 시점에 전체 프로그램 상태가 변경되어서 유지되지 않을 수 있는 상태가 생길 수 있습니다. **그리고 이런 경우를 정의하지 않았는지 확인하는 것이 중요합니다.**

## actor 에서 await 가 만들어낼 수 있는 문제

동일한 URL 에 대해서 이미지를 가져오는 작업인 Task 1 과 Task2 가 있다고 가정해 보겠습니다.

- Task 1 은 캐시에 해당 URL 에 대한 이미지가 없기 때문에 😸 이미지 다운로드를 시작한 뒤 suspend 됩니다.
<img src="https://user-images.githubusercontent.com/69136340/171853343-f0331679-bdb8-4d5b-93f6-30ee4bac8ed0.png" width ="700">

***이렇게 Task1 이 suspend 된 동안 동일한 URL의 서버에 새로운 이미지가 올라올 수 있습니다…!***

- Task 2 는 동일한 URL 에 대해서 이미지를 가져오려 하는데, cache 에는 이미지가 없기 때문에(Task 1 의 다운로드가 완료되지 않았습니다.) 😿 이미지를 다운받기 시작하고, suspend 됩니다.
<img src="https://user-images.githubusercontent.com/69136340/171853417-5a8c048c-a6c1-47e4-8e17-2e35b73cdcea.png" width ="700">

잠시 후, Task 1 다운로드 작업이 끝나고 cache 에 😸 이미지를 저장합니다. Task 2 역시 다운로드 작업이 끝나고 😿 이미지를 cache 에 덮어씌우게 됩니다. 즉, 동일한 URL 에 대해서 서로 다른 이미지를 다운로드하게 됩니다.

**Actor 가 low-level 의 data races 는 없지만 await 로 인해 버그가 발생한 것이지요!**

어떻게 해야할까요?

await 이후에 잘 수행되는지 확인하면 됩니다. async 함수를 다시 실행할 때, cache 에 값이 있으면 원래 버전을 유지하고 새로운 버전을 버리도록 하거나, 동일한 URL 에 대해서는 중복 다운로드를 못하게 하면 됩니다.

```swift
actor ImageDownloader {
    private var cache: [URL: Image] = [:]

    func image(from url: URL) async throws -> Image? {
        if let cached = cache[url] {
            return cached
        }

        let image = try await downloadImage(from: url)

        // Replace the image only if it is still missing from the cache.
        // ✅ 딕셔너리의 Subscript 로 element 에 접근하면 기본 반환값이 optional type 입니다. 옵셔널이 싫다면 default 값을 직접 명시 할 수 있습니다.
        // ✅ await 이후, cache[url] 이 있다면?(Task 2 가 suspend 되었을 때, Task 1 다운로드 완료 한 경우.) Task 2 가 다운로드한 이미지 대신 기존의 이미지 유지. 없다면 다운로드한 이미지 설정.
        cache[url] = cache[url, default: image]
// ✅ 결과: 세션 속의 경우에 😸 이미지가 캐싱됨.

        return cache[url]
    }
}
```

## 적용하기

---

자, Actor 를 활용한 ImageDownloader 에 대해서 알아보았습니다. 이제는 async/await 로 만든 ImageFetchProvider 를 활용해서 이미지를 다운받고, 그 이후의 캐싱 역할은 Actor 로 만든 ImageDownloader 에서 처리해보도록 하겠습니다.

- ImageFetchProvider

```swift
import UIKit

struct ImageFetchProvider {
    static let shared = ImageFetchProvider()
    private init() { }
    
    /// URL 을 가지고 data 를 다운받아서 UIImage 로 변환하는 메서드.
    /// - Parameter url: 다운받을 URL 값.
    /// - Returns: 다운 받은 data 를 UIImage 로 변환해서 리턴. 변환되지 않는 경우 에러를 던집니다.
    public func fetchImage(with url: URL) async throws -> UIImage {
        let (data, response) = try await URLSession.shared.data(from: url)
        guard let httpResponse = response as? HTTPURLResponse, httpResponse.statusCode == 200 else {
            throw ImageDownloadError.invalidServerResponse
        }
        
        guard let image = UIImage(data: data) else {
            throw ImageDownloadError.unsupportImage
        }
        
        return image
    }
}
```

- ImageDownloader

```swift
import UIKit

actor ImageDownloader {
    // ✅ 캐싱 구현하기 위해서 싱글톤 패턴 사용.
    static let shared = ImageDownloader()
    private init() { }
// 초기에는 ImageFetchProvider 에서 에러를 핸들링해주었고, 이 과정에서 유효한 URL 에 대한 핸들링도 진행해주었다.
// 그래서 ImageFetchProvider.shared.fetchImage(with:)에 매개변수로 String 을 넘겨주어서 에러 핸들링을 하는 구조였다. 그 결과, cache 의 자료형이 [String, UIImage] 로 설정되었고, 어떤 key 로 캐싱하는지에 대한 문맥상 표현이 안되서 [URL: UIImage] 자료형을 고수하게 되었다.
// 어떻게 보면 에러를 던지는 함수를 호출하는 함수에서 다시금 에러를 던질 수 있는 구조이다. 하지만 이건 내가 역할을 나누기 위해서 나눈것이지 결과적으로는 호출하는 맨위 함수에서 에러를 핸들링하고 있고, 에러라는 것은 어느 순간에나 등장할 수 있기 때문에 에러를 핸들링하는 역할을 특정 객체에 한정짓지 않아도 생각했습니다.(마치 초기에 ImageFetchProvider 가 모든 에러를 던지도록 했던 모양새처럼 말이죵)
    private var cache: [URL: UIImage] = [:]
    
    func image(from urlPath: String) async throws -> UIImage? {
        guard let url = URL(string: Const.Path.imageURLPath + urlPath) else {
            throw ImageDownloadError.invalidURLString
        }
        
        // ✅ 이미 다운 받은 URL 에 대해서 캐싱 처리.
        if let cached = cache[url] {
            return cached
        }
        
        let image = try await downloadImage(from: url)

        cache[url] = cache[url, default: image]
        
        return cache[url]!
    }

    private func downloadImage(from url: URL) async throws -> UIImage {
        let imageFetchProvider = ImageFetchProvider.shared
        return try await imageFetchProvider.fetchImage(with: url)
    }
}
```

- 다음의 메서드를 collection view datasource 에서 호출해서 cell 을 만들어주었습니다.

```swift
// MovieCollectionViewCell.swift

func initCellWith(urlPath: String, title: String) {
        Task {
            do {
                // ✅ 싱글톤 패턴 사용
                let posterImage = try await ImageDownloader.shared.image(from: urlPath)

                posterImageView.image = posterImage
                titleLabel.text = title
            } catch ImageDownloadError.unsupportImage {
                print("image download error - unsupportImage")
            } catch ImageDownloadError.invalidServerResponse {
                print("image download error - invalidServerResponse")
            } catch ImageDownloadError.invalidURLString {
                print("image download error - invalidURLString")
            }
        }
    }
```

## 참고) Enabling Thread Sanitizer

---

 Thread Sanitizer 를 활성화해서  `data races` 코드를 확인해보겠습니다.
<img src="https://user-images.githubusercontent.com/69136340/171853545-ff21e0f1-a1f2-4616-8322-a5e0c694fedb.png" width ="400">

<img src="https://user-images.githubusercontent.com/69136340/171853555-97a65598-5b32-486e-a340-cc5b06ed07a5.png" width ="600">

- 다음과 같이 ImageDownloader 를 calss 로 선언한 경우는 data races 이 발생할 수 있습니다. actor 로 선언하니 사라졌습니당

<img src="https://user-images.githubusercontent.com/69136340/171853562-0152291b-3564-4a1a-9ff7-ec2c1a006839.png" width ="600">

**깃허브:**

[https://github.com/28th-SOPT-iOS-CloneCoding/SpectaClone-KimHyunGyu](https://github.com/28th-SOPT-iOS-CloneCoding/SpectaClone-KimHyunGyu)

**출처:**

[Protect mutable state with Swift actors - WWDC21 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2021/10133/)

[Protect mutable state with Swift actors](https://velog.io/@unknown_horse/Protect-mutable-state-with-Swift-actors)

[[WWDC 2021] Protect mutable state with Swift actors](https://icksw.tistory.com/269)

[Swift) Dictionary - 자주 사용하는 메서드](https://babbab2.tistory.com/113)
