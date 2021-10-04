---
title:  "iOS) Moya 에서 DELETE, PATCH 통신 사용해보기"
categories:
- iOS

date:   2021-10-04  16:18:00 +0900
author_profile: false
---
지금까지 GET, POST 통신만 사용해보았다. DELETE 와 PATCH 도 구현해보자.

먼저 DELETE 와 PATCH 통신에 대해서 알아보자.

### 🍕 DELETE?

특정 리소스를 삭제.

DELETE 는 request body 가 없지만 response body 가 존재한다.

### 🍕 PATCH?

특정 리소스의 부분만을 수정.

PATCH 는 request body 와 response body 가 존재한다.

<img width="600" alt="스크린샷 2021-10-04 오전 11 21 24" src="https://user-images.githubusercontent.com/69136340/135808898-e249db71-1d4b-44e1-8f76-9b0c1f031e70.png">


사진 출처 : 

[HTTP - 위키백과, 우리 모두의 백과사전](https://ko.wikipedia.org/wiki/HTTP)

## 🍕 Moya 에서 DELETE, PATCH 구현하기

`path` 와 `method` 를 설정하는 과정까지는 순조롭다고 생각한다. 다음의 세가지 통신을 구현해보자.

- fetchPopoList : GET 통신.
- deletePopo : URL path 에 정수형을 가진 DELETE 통신.
- changeBackground :  URL path 에 정수형과 request body 에 이미지를 담은 PATCH 통신.

```swift
import Foundation
import Moya

enum PopoService {
// ✅ GET
    case fetchPopoList
// ✅ DELETE
    case deletePopo(popoID: Int)
// ✅ PATCH
    case changeBackground(popoID: Int, backgroundImage: UIImage)
}

extension PopoService: TargetType {
    var baseURL: URL {
        return URL(string: Const.URL.baseURL)!
    }
    
    var path: String {
        switch self {
        case .fetchPopoList:
            return ""
        case .deletePopo(let popoID):
            return "/\(popoID)"
        case .changeBackground(let popoID, _):
            return "/popo/\(popoID)/background"
        }
    }
    
    var method: Moya.Method {
        switch self {
        case .fetchPopoList:
            return .get
        case .deletePopo:
            return .delete
        case .changeBackground:
            return .patch
        }
    }
    
    var task: Task {
        switch self {
        case .fetchPopoList:
            return .requestPlain
        case .deletePopo:
            return .requestPlain
        case .changeBackground(_ , let backgroundImage):
            if let backgroundImage = backgroundImage.jpegData(compressionQuality: 1.0) {
                return .uploadMultipart([MultipartFormData(provider: .data(backgroundImage), name: "image", fileName: "background.jpg", mimeType: "image/jpg")])
            }
            return .requestPlain
        }
    }
    
    var headers: [String: String]? {
        switch self {
        case .fetchPopoList:
            return ["Content-Type": "application/json"]
        case .deletePopo:
            return .none
        case .changeBackground:
            return ["Content-Type" : "multipart/form-data"]
        }
    }
}
```

GET 은 쉽게 구현했는데 DELETE, PATCH 를 구현하려 했는데 task 에서 막혔다

 먼저 Moya 가 제공하는 열거형인 Task 에 대해서 알아보자. 

```swift
import Foundation

/// Represents an HTTP task.
public enum Task {

    /// A request with no additional data.
    case requestPlain

    /// A requests body set with data.
    case requestData(Data)

    /// A request body set with `Encodable` type
    case requestJSONEncodable(Encodable)

    /// A request body set with `Encodable` type and custom encoder
    case requestCustomJSONEncodable(Encodable, encoder: JSONEncoder)

    /// A requests body set with encoded parameters.
    case requestParameters(parameters: [String: Any], encoding: ParameterEncoding)

    /// A requests body set with data, combined with url parameters.
    case requestCompositeData(bodyData: Data, urlParameters: [String: Any])

    /// A requests body set with encoded parameters combined with url parameters.
    case requestCompositeParameters(bodyParameters: [String: Any], bodyEncoding: ParameterEncoding, urlParameters: [String: Any])

    /// A file upload task.
    case uploadFile(URL)

    /// A "multipart/form-data" upload task.
    case uploadMultipart([MultipartFormData])

    /// A "multipart/form-data" upload task  combined with url parameters.
    case uploadCompositeMultipart([MultipartFormData], urlParameters: [String: Any])

    /// A file download task to a destination.
    case downloadDestination(DownloadDestination)

    /// A file download task to a destination with extra parameters using the given encoding.
    case downloadParameters(parameters: [String: Any], encoding: ParameterEncoding, destination: DownloadDestination)
}
```

내가 통신하는 목적에 대해서 이해를 하고 동작해줄 task 에 대해서 고민해본다면 쉽게 코드를 작성할 수 있었다. 겁먹지 말자!

- DELETE 는 URL 를 제외하고 본문(request body)에 추가할 데이터가 없다.(애초에 request body 자체가 없음.) 그래서 `.requestPlain` 를 사용한다. request body 가 없기 때문에 헤더의 Content-Type 이 사용되지 않는다. 그래서 `headers` 가 없어도 된다.
- PATCH 는 request body 가 존재하기 때문에 JSON 혹은 DATA 를 request body 에 담아  전송이 가능하다. 그렇기 때문에 `headers` 도 필요하다.
