---
title:  "iOS) RxSwift + Moya 를 사용하여 서버통신하기"
categories:
- iOS

date:   2023-10-12  14:53:00 +0900
author_profile: false
---
### 내용

- Moya 로 구축한 서버통신 환경에서 서버통신을 진행하고 RxSwift 를 사용해보자

## 👉 들어가기 전

**우선, Moya 깃허브에서 제공하는 RxSwift 문서를 살펴보겠습니다.**

Moya 의 MoyaProvider 는 몇 가지 선택적인 RxSwift 구현을 제공합니다. **request()** 메소드를 호출하고 요청이 완료될 때 콜백 클로저를 제공하는 대신 Observable 을 사용합니다. 이는 `success` 와 `error` 를 방출하는 **trait** 의 한 종류인 **Single** 에 해당합니다.

```swift
provider.rx.request(.zen).subscribe { event in
    switch event {
    case .success(let response):
        // do something with the data
    case .failure(let error):
        // handle the error
    }
}

// (수정)case .error(let error):
// 공식문서에서는 .error인데 코드를 실제로 작성해보면 event는 Result이기 때문에 .error가 아닌 .failure를 분기처리 한다.
```

**Single 이 subscribe 되기 전까지는 network 요청이 시작되지 않는 것을 기억해야 합니다.** request 가 complete 되기 전에 구독이 삭제되면 요청도 취소됩니다.

request 가 정상적으로 complete 되면 다음 두 가지 일이 발생합니다.

- Observable 이 Moya.Response 인스턴스 값을 보냅니다.
- Observable 이 complete 됩니다.

만약 에러가 발생하면 error 의 **code** 는 실패한 요청의 상태코드(있다면)와 response data(있다면) 입니다.

**Moya.Response** 클래스에는 `**statusCode**`, `**data**`, `**HTTPURLResponse(optional)**` 가 포함됩니다. **subscribe** 혹은 **map** 을 호출하여 값을 사용할 수 있습니다.

Moya.Reasponse 를 쉽게 처리할 수 있도록 Single, Observable 에 대한 extension 들을 제공합니다.

<img width="600" alt="1" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/e0b7006f-c6c9-4cd6-bf23-f56aea62f15c">

error 의 경우 Moya.MoyaError 입니다. code 는 MoyaErrorCode 의 rawValue 중 하나입니다.

(출처: https://github.com/Moya/Moya/blob/master/docs/RxSwift.md)

MoyaProvider 가 제공하는 `request()` 를 살펴보겠습니다. 반환형은 **Single<Response>**입니다.

<img width="350" alt="2" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/6dc3d080-17e7-4cec-9d23-85bf21843a25">

Single 에 대해서 알아보겠습니다.

## 👉 Single

`Single` 은 `**success(value)**` 또는 **`failure(error)`** 이벤트를 방출합니다. `success(value)` 는 실제로 `next` 와 `completed` 이벤트의 조합입니다.

- 다음은 Single 을 만드는 create() 메서드입니다. **success** 에서 **next, completed** 이벤트를 방출하고 error 에서 error 이벤트를 방출합니다.

<img width="600" alt="3" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/de5d213c-83f4-48e2-a4b3-d957094358ac">

이는 데이터를 다운로드하거나 디스크에서 로드할 때와 같이 성공하여 값을 생성하거나 실패하는 일회성 프로세스에 유용합니다.

(출처: https://www.kodeco.com/books/rxswift-reactive-programming-with-swift/v4.0/chapters/1-hello-rxswift)

RxSwift 깃허브의 **Single** 문서도 살펴보겠습니다.

**Single 은** 일련의 element 를 방출하는 것이 아닌 **단일 element 또는 error 를 방출하도록 보장합니다.**

그렇기 때문에 Single 을 사용하는 예시로 response 또는 error 만 반환할 수 있는 HTTP 요청이 있습니다.

*(여기에서도 서버 통신과 관련해서 이점이 있음을 언급해주네요!)*

```swift
func getRepo(_ repo: String) -> Single<[String: Any]> {
    // ✅ Single 을 만들어서 반환. 
    return Single<[String: Any]>.create { single in
        let task = URLSession.shared.dataTask(with: URL(string: "https://api.github.com/repos/\(repo)")!) { data, _, error in
            if let error = error {
                // 방출
                single(.failure(error))
                return
            }

            guard let data = data,
                  let json = try? JSONSerialization.jsonObject(with: data, options: .mutableLeaves),
                  let result = json as? [String: Any] else {
                // 방출
                single(.failure(DataError.cantParseJSON))
                return
            }
            // 방출
            single(.success(result))
        }

        task.resume()

        return Disposables.create { task.cancel() }
    }
}
```

그 후에는 다음과 같이 사용할 수 있습니다.

```swift
getRepo("ReactiveX/RxSwift")
    .subscribe { event in
        switch event {
            case .success(let json):
                print("JSON: ", json)
            case .failure(let error):
                print("Error: ", error)
        }
    }
    .disposed(by: disposeBag)
```

또는 **subscribe(onSuccess:onError:)** 메소드를 사용할 수도 있습니다.

```swift
getRepo("ReactiveX/RxSwift")
    .subscribe(onSuccess: { json in
                   print("JSON: ", json)
               },
               onFailure: { error in
                   print("Error: ", error)
               })
    .disposed(by: disposeBag)
```

subscribe 은 **.success** 또는 **.failure** 일 수 있는 **Result** enum 을 사용합니다. 첫 번째 이벤트 이후에는 더이상 이벤트는 없습니다.

(출처: https://github.com/ReactiveX/RxSwift/blob/main/Documentation/Traits.md#single)

## 👉 적용해보자

- Moya 를 사용하여 **Service** 를 구성하고, 뷰 컨트롤러에서 map 을 통해 간단하게 결과물을 사용하고자 합니다.

<img width="400" alt="4" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/525f6adc-cdf6-464e-9bb5-7a1aa0d88aba">

→ 프로토콜을 채택한 타입을 얻기위해서  .Protocol 사용합니다. 즉, Decodable 을 채택한 타입을 얻기위한 자료형 표기입니다.

- 손쉽게 서버 통신의 결과물을 사용해 보겠습니다.

```swift
private func fetchWithAPI() {
    var tagProvider = MoyaProvider<TagService>()
        
    tagProvider.rx.request(.tagFetch)
        // filters status codes that are in the 200-range.
        .filterSuccessfulStatusCodes()
        .map([Tag].self)
        .subscribe(with: self) { owner, data in
            print(data)
            owner.tagList = data
        }
        .disposed(by: disposeBag)
}
```

- 다음은 map 대신 **subscribe** 을 사용해서 status code 에 따라서 분기처리를 다음과 같이 해볼 수도 있었습니다.

```swift
    private func fetchWithAPI() {
        var tagProvider = MoyaProvider<TagService>()
        
        tagProvider.rx.request(.tagFetch)
            .subscribe { event in
                switch event {
                case .success(let response):
                    let decoder = JSONDecoder()
                    guard let decodedData = try? decoder.decode(GenericResponse<[Tag]>.self, from: response.data) else { print("디코딩 실패") }
                    
                    switch decodedData.status {
                    case 200..<300:
                        print("성공")
                        self.tagList = decodedData.data
                    case 500:
                        print("서버 에러")
                    default:
                        print("통신 실패")
                    }
                case .failure(let error):
                    print(error)
                }
            }
            .disposed(by: disposeBag)
```

- request() 메소드를 통해 Single 을 subscribe 했을 때 다음과 같이 **SingleEvent** 파라미터를 전달 받습니다.

<img width="350" alt="5" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/a35b2321-4fe6-45df-b470-90eb2d1ad055">

- 코드를 살펴보게되면 next 와 error 가 success 와 failure 로 이어집니다.

<img width="550" alt="6" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/9fa45b1e-7dbb-4e9a-95a2-2ab4e397daaf">

- **subscribe(_: )** 메소드에서 전달되는 **SingleEvent** 를 살펴보면 **Result** enum 형입니다.

```swift
public typealias SingleEvent<Element> = Result<Element, Swift.Error>
```

<img width="550" alt="7" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/6605858e-168d-483c-a6c8-367b6fd471ae">

이후 switch 문을 통해서 Result enum 형의 분기처리를 진행할 수 있었습니다.

### 👉 서버통신을 위한 API 클래스를 만들어보겠습니다.

- API 클래스를 만들어 서버통신의 디코딩 처리, 상태 코드에 대한 분기처리를 해보자.
- 뷰 컨트롤러에서 구독. 즉, 스트림이 API 클래스에서 구독되어 끊기지 않도록 유의해보자.
    - 그렇기 위해서는 API 클래스의 메소드에서 Observable 형태를 반환해야 합니다.

**MoyaProvider 에서 request() 메소드를 통해 Single 을 반환하는 역할**의 API 클래스의 메소드를 만들어 주었습니다.

```swift
// TagAPI.swift
import Foundation

import Moya
import RxSwift

class TagAPI {
    static let shared = TagAPI()
    var tagProvider = MoyaProvider<TagService>()
    private init() { }

    // 받은 태그를 조회하는 api.
    func receivedTagFetch(cardUUID: String) -> Single<NetworkResult2<GenericResponse<[ReceivedTag]>>> {
        // ✅ 서버통신의 결과를 Single로 만들어서 반환하였습니다.
        return Single<NetworkResult2<GenericResponse<[ReceivedTag]>>>.create { single in
            self.tagProvider.request(.receivedTagFetch(cardUUID: cardUUID)) { result in
                switch result {
                case .success(let response):
                    let networkResult = self.judgeStatus(response: response, type: [ReceivedTag].self)
                    single(.success(networkResult))
                    return
                case .failure(let error):
                    single(.failure(error))
                    return
                }
            }
            return Disposables.create()
        }
    }

    func judgeStatus<T: Codable>(response: Response, type: T.Type) -> NetworkResult2<GenericResponse<T>> {
        let decoder = JSONDecoder()
        guard let decodedData = try? decoder.decode(GenericResponse<T>.self, from: response.data) else { return .pathErr }
        
        switch response.statusCode {
        case 200..<300:
            if decodedData.status >= 400 {
                // 서버 통신은 정상적이지만, 원치 않는 결과로 인한 에러 대응 시 사용.
                return .success(decodedData)
            } else {
                return .success(decodedData)
            }
        case 400..<500:
            return .requestErr
        case 500:
            return .serverErr
        default:
            return .networkFail
        }
    }
}

// GnericResponse.swift
struct GenericResponse<T: Codable>: Codable {
    let code: String?
    let message: String?
    let status: Int
    let data: T?
}

// NetworkResult2.swift
public enum NetworkResult2<T> {
    case success(T)
    case requestErr
    case pathErr
    case serverErr
    case networkFail
}
```

다음과 같이 뷰 컨트롤러에서 사용하였습니다.

```swift
    private func receivedTagFetchWithAPI() {
        TagAPI.shared.receivedTagFetch().subscribe(with: self, onSuccess: { owner, networkResult in
            switch networkResult {
            case .success(let response):
                // response: GenericResponse<[ReceivedTag]>
                print("receivedTagFetchWithAPI - success")
                
                // ✅ 서버통신 성공 후의 동작.
                if let data = response.data {
                    // ...
                    
                    DispatchQueue.main.async {
                        owner.collectionView.reloadData()
                    }
                }
            case .requestErr:
                print("receivedTagFetchWithAPI - requestErr")
            case .pathErr:
                print("receivedTagFetchWithAPI - pathErr")
            case .serverErr:
                print("receivedTagFetchWithAPI - serverErr")
            case .networkFail:
                print("receivedTagFetchWithAPI - networkFail")
            }
        }, onFailure: { owner, error in
            print("tagCreationWithAPI - error")
        }).disposed(by: disposeBag)
    }
```

- subscribe(with:onSuccess:onFailure…) 메소드를 사용하여 전달된 NetworkResult2<GenericResponse<T>> 를 사용하게됩니다.
- Single 을 만들 때 error 이벤트 방출이 subscribe() 을 통해 SingleEvent.failure 로 분기처리되기 때문에 핸들링 해주었습니다.

<img width="350" alt="9" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/3c7c6e18-ed3a-4f87-aa42-3dc592ca3d52">
