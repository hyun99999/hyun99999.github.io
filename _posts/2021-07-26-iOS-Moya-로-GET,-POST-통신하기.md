---
title:  "iOS) Moya 로 GET, POST 통신하기"
categories:
- iOS

date:   2021-07-26  02:38:00 +0900
author_profile: false
---
Moya 의 기본을 다루는 글은 많은데 막상 다양한 경우의 get 과 post 요청에 대해서는 정보가 많지 않아서 이번 프로젝트에서 사용한 방법을 공유하고 한다. 그리고 내가 가진 궁금증에 대해서 알아가면서 진행될 포스팅이다.

## 🤔 궁금증 1

**omoolent 프로젝트 중)**

이번 프로젝트에서 서버에서 get 의 request body 에 데이터를 담아달라고 부탁했었는데(결국 존재하지 않기때문에 오류만 잔뜩 얻었다) 그 이유는 get 의 response body 로 정보를 보내주기 위함이었다. 하지만 post 통신에서도 response body 가 존재하기 때문에 post 의 request body 를 통해서 정보를 서버로 보내고 post 의 response body 로 정보를 얻기로 하였다.

위처럼 GET, POST 에 대해서 정확히 공부할 필요성을 느꼈다.


먼저 아래의 글을 읽고 get, post 통신에 대해서 이해해보자


참고 : [Get과 Post의 차이를 아시나요?](https://velog.io/@songyouhyun/Get과-Post의-차이를-아시나요)

아래는 위키백과의 HTTP 요약표이다.

<img src ="https://user-images.githubusercontent.com/69136340/126907990-aa2911ca-7333-48f0-9ca0-5a70284bedf4.png" width ="600">

### **GET vs POST**

<img src ="https://user-images.githubusercontent.com/69136340/126907992-da312bdd-204e-4585-8d75-567410c44b7f.png" width ="500">

HTTP POST 요청은 클라이언트에서 서버로 전송할 때 추가적인 데이터를 body에 포함할 수 있다. 반면에 GET 요청은 모든 필요한 데이터를 URL에 포함하여 요청한다. 만약에 GET 메소드를 사용하면 모든 data는 URL로 인코딩되어 URL에 query string parameters로 전달된다. POST 메소드를 사용하면 data는 HTTP request의 message body에 나타날 것이다.

출처 : [[HTTP] HTTP Method 정리 / GET vs POST 차이점](https://im-developer.tistory.com/166)

---

그렇다면 이제 Moya 에서의 get, post 에 대해서 알아보자.

**예시)**

잘 알 수 있도록 열거형의 이름을 정해보았다.

justGet : 특별한 것 없이 가장 무난한 get 통신

queryStringGet : URL query string parameters get 통신

requestBodyPost : `request body` 에 data 와 `header` 에 accesstoken 을 담은 post 통신

```swift
import Foundation
import Moya

enum SearchService {
    case justGet
    
    case queryStringGet(keyword: String)

    case requestBodyPost(param: RequestDataModel, accesstoken: String)
}

extension SearchService: TargetType {
    var baseURL: URL {
        return URL(string: "baseURL")!
    }
    
    var path: String {
        switch self {
        case .justGet:
            return "/api/justGet"
        case .queryStringGet:
            return "/api/queryStringGet"
        case .requestBodyPost:
            return "/api/requestBodyPost"
        }
    }
    
    var method: Moya.Method {
        switch self {
        case .justGet:
            return .get
        case .queryStringGet:
            return .get
        case .requestBodyPost:
            return .post
        }
    }
    
    var sampleData: Data {
        return  "sampleData".data(using: .utf8)! 
    }
    
    var task: Task {
        switch self {
        case .searchWindow:
            return .requestPlain
                case .queryStringGet(let keyword):
// get 통신은 request body 가 없다고 했는데 moya 에서 이와 같은 방법으로 query string 을 서버로 보낸다.
            return .requestParameters(parameters: ["keyword" : keyword], encoding: URLEncoding.queryString)
        case .requestBodyPost(let requestDataModel, _):
            return .requestJSONEncodable(requestDataModel)
        }
    }
    
    var headers: [String : String]? {
        switch self {
        case .searchWindow(_, let accesstoken):
            return ["Content-Type": "application/json", "accesstoken" : accesstoken]
        default:
            return ["Content-Type": "application/json"]
        }
    }
}
```

## 🤔 궁금증 2

**`task` 에 대해서 좀 더 알아보자**

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

// 출처 : https://github.com/Moya/Moya/blob/master/Sources/Moya/Task.swift
```

## 🤔 궁금증 3

**get 통신에서 서버로 추가적인 정보를 보내기 위해서 `ParameterEncoding` 타입에 해당하는 `URLEncoding` 을 사용하면서 많이들 봤을 것이다! 어떤것에 해당하는지 알아보자.**

<img width="500" alt="스크린샷 2021-07-26 오전 1 32 18" src="https://user-images.githubusercontent.com/69136340/126908092-c2850a95-15ee-42a1-845d-f390d08d67c4.png">

- default : Returns a default URLEncoding instance with a .methodDependent destination.
- httpBody : Returns a URLEncoding instance with a .httpBody destination.
- queryString : Returns a URLEncoding instance with a .queryString destination.

Moya 프로젝트에서 확인해보니 Alamofire 의 URLEncoding 을 그대로 사용했기 때문에 위의 내용들은 Alamofire 에서 찾을 수 있었다.

```swift
/// Choice of parameter encoding.
public typealias ParameterEncoding = Alamofire.ParameterEncoding
public typealias JSONEncoding = Alamofire.JSONEncoding
public typealias URLEncoding = Alamofire.URLEncoding

// 출처 : https://github.com/Moya/Moya/blob/master/Sources/Moya/Moya%2BAlamofire.swift
```

---

### Alamofire - `URLEncodedFormParameterEncoder`

`URLEncodedFormParameterEncoder` 는 기존 URL query stirng 또는 요청의 HTTP body 로 설정될 URL 인코딩 문자열로 값을 인코딩. destination of the encoding 을 설정하여 인코딩 된 문자열이 설정된 위치를 제어 가능. `URLEncodedFormParameterEncoder.Destination` 에는 세가지 경우가 있다.

- `.methodDependent` - 인코딩 된 query 문자열 결과를 .get, .head, .delete 요청에 대한 기존 query 문자열에 적용하고 다른 HTTP 메소드의 요청에 대한 HTTP body 로 설정.
- `.queryString` - request `URL` query 에 인코딩 된 문자열을 설정하거나 추가.
- `.httpBody` - 인코딩된 문자열을 `URLRequest` 의 HTTP body 로 설정.

**GET Request With URL-Encoded Parameters**

```swift
let parameters = ["foo": "bar"]

// All three of these calls are equivalent
AF.request("https://httpbin.org/get", parameters: parameters)// encoding defaults to `URLEncoding.default`
AF.request("https://httpbin.org/get", parameters: parameters, encoder: URLEncodedFormParameterEncoder.default)
AF.request("https://httpbin.org/get", parameters: parameters, encoder: URLEncodedFormParameterEncoder(destination: .methodDependent))

// https://httpbin.org/get?foo=bar
```

**POST Request With URL-Encoded Parameters**

```swift
let parameters: [String: [String]] = [
    "foo": ["bar"],
    "baz": ["a", "b"],
    "qux": ["x", "y", "z"]
]

// All three of these calls are equivalent
AF.request("https://httpbin.org/post", method: .post, parameters: parameters)
AF.request("https://httpbin.org/post", method: .post, parameters: parameters, encoder: URLEncodedFormParameterEncoder.default)
AF.request("https://httpbin.org/post", method: .post, parameters: parameters, encoder: URLEncodedFormParameterEncoder(destination: .httpBody))

// HTTP body: "qux[]=x&qux[]=y&qux[]=z&baz[]=a&baz[]=b&foo[]=bar"
```

출처 : [Alamofire(알라모파이어) 깃허브 문서를 요약해보자](https://gyuios.tistory.com/64)

---

## 🤔 궁금증 4

**그렇다면 request body, http body, http header 등 이것들은 대체 무엇일까?**

http 요청(request)과 응답(response)의 헤더(header)와 본문(body) 이라고 생각하면 된다.

---

### 출처

[HTTP - 위키백과, 우리 모두의 백과사전](https://ko.wikipedia.org/wiki/HTTP)

[Moya/docs/Examples at master · Moya/Moya](https://github.com/Moya/Moya/tree/master/docs/Examples)
