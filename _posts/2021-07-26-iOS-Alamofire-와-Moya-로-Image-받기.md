---
title:  "iOS) Alamofire 와 Moya 에서 Image 받기"
categories:
- iOS

date:   2021-07-26  00:37:00 +0900
author_profile: false
---
# Almofire

### Downloading data to a file

data 를 메모리로 가져오는 것 외에도 `Alamofire` 는 disk 로의 다운로딩을 위해서 `Session.download`, `DownloadRequest`, `DownloadResponse<Success, Failure: Error>` API 를 제공한다.

```swift
AF.download("https://httpbin.org/image/png").responseURL { response in
    // Read file from provided URL.
}
```

`responseURL` 은 다른 응답 핸들러들과 달리 다운로드 된 데이터의 위치가 포함 된 `URL` 만 반환하고 disk 에서 `Data` 를 읽지 않는다.

`responseDecodable` 과 같은 other response hanlders 는 disk 에서 `Data` 읽기가 가능하다. 이 경우 큰 데이터를 메모리로 읽는 것이 포함 될 수 있으므로 이런 핸들러를 사용할 때 염두해두어야 한다.

### Download File Destination

다운로드 된 모든 데이터는 system temporary directory 에 저장된다. 결국 언젠가는 삭제가 되므로 더 오래 유지해야할 경우 다른 곳으로 이동시켜야 한다.

final destination 으로 옮기기 위해서 `Destinaion` closure 를 제공할 수 있다. `destinationURL` 로 이동하기 전에 closure 에 지정된 옵션이 실행된다. 두가지 옵션:

- `.createIntermediateDirectories` - 지정된 경우 destination URL 에 대한 intermediate directories 를 만든다.
- `.removePreviousFile` - 지정된 경우 destionation URL 의 이전 파일을 삭제한다.

```swift
let destination: DownloadRequest.Destination = { _, _ in
    let documentsURL = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
    let fileURL = documentsURL.appendingPathComponent("image.png")

    return (fileURL, [.removePreviousFile, .createIntermediateDirectories])
}

AF.download("https://httpbin.org/image/png", to: destination).response { response in
    debugPrint(response)

    if response.error == nil, let imagePath = response.fileURL?.path {
        let image = UIImage(contentsOfFile: imagePath)
    }
}
```

제시된 download destination API 를 사용할 수 있다.

```swift
let destination = DownloadRequest.suggestedDownloadDestination(for: .documentDirectory)

AF.download("https://httpbin.org/image/png", to: destination)
```

자, 이제 그럼 Moya 에서는 어떻게 download data 하는지 알아보자. 

# Moya

## Reactive Extensions

RxSwift 와 ReactiveSwift 에 대한 extensions 를 제공한다.

### ReactiveSwift

사용자가 start, bind, map 등 수행할 수 있는 `SignalProducer` 를 즉시 반환하는 `reactive.request(:callbackQueue:)`,`reactive.requestWithProgress(:callbackQueue:)` 메서드를 제공한다.

```swift
provider = MoyaProvider<GitHub>()
provider.reactive.request(.userProfile("ashfurrow")).start { event in
    switch event {
    case let .value(response):
        image = UIImage(data: response.data)
    case let .failed(error):
        print(error)
    default:
        break
    }
}
```

### RxSwift

`rx.request(:callbackQueue:)` 와 `rx.requestWithProgress(:callbackQueue:)` 메서드를 제공하지만 리턴 타입이 둘다 다르다.

- `rx.request(:callbackQueue:)` : `Single<Response>`
- `rx.requestWithProgress(:callbackQueue:)` : `Observable<ProgressResponse>` 이유는 progress 과정에서 여러개의 이벤트와 response 에서 한개의 이벤트를 받기 때문.

```swift
provider = MoyaProvider<GitHub>()
provider.rx.request(.userProfile("ashfurrow")).subscribe { event in
    switch event {
    case let .success(response):
        image = UIImage(data: response.data)
    case let .error(error)d
        print(error)
    }
}
```

callback blocks 대신에 signals 사용하는 옵션 외에도 RxSwift, ReactiveSwift 에 대한 signal operators 가 존재한다. 네트워크에서 받은 응답의 데이터를 image, JSON, string 에 매핑을 하는 `mapImage()`, `mapJSON()`, `mapString()`. 매핑을 실패하면 error 발생. invalid responses 를 핸들링하는 코드에서 400가 같은 API 오류를 처리하는 코드도 동일하게 배치 가능.

---

[Moya/Moya](https://github.com/Moya/Moya#reactive-extensions)
