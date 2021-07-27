---
title:  "iOS) Alamofire 깃허브 내용을 요약해보자.
categories:
- iOS

date:   2021-07-25  22:43:00 +0900
author_profile: false
---
-   Alamofire 깃허브 내용을 요약해보자.

# **Using Alamofire**

## **Introduce**

alamofire 는 HTTP network requests 의 인터페이스를 제공. Foundation 프레임워크에서 제공하는 Apple 의 URL 로딩 시스템을 기반으로 구축. 즉, URLSession 과 URLSessionTask 하위클래스가 핵심이다. Alamofire 는 이러한 API 와 기타여러 API 를 상요학 쉬운 인터페이스로 래핑해서 제공.

### **The AF Namespace and Reference**

이전버전의 Alamofire 문서에서는 `Alamofire.request()` 와 같은 예제를 사용했다. `Alamofire` 접두사가 필요해보였지만 없이도 작동하고 `import Alamofire`로 전역적으로 `request` 메소드와 다른 함수 사용이 가능했다. Alamofire 5 부터는 `AF` 전역이 사용된다.

## **Making Requests**

가장 간단하게 `URL`로 전환되는 `String` 을 제공.

```
AF.request("https://httpbin.org/get").response { response in
    debugPrint(response)
}
```

> 모든 예제는 import Alamofire 가 필요. 이것은 Alamofire의 Session타입 의 최상위 API 중 한개다.(보통 GET 요청 시 사용) 전체정의는 다음과 같다,

```
open func request<Parameters: Encodable>(_ convertible: URLConvertible,
                                         method: HTTPMethod = .get,
                                         parameters: Parameters? = nil,
                                         encoder: ParameterEncoder = URLEncodedFormParameterEncoder.default,
                                         headers: HTTPHeaders? = nil,
                                         interceptor: RequestInterceptor? = nil) -> DataRequest
```

이 메서드는 `DataRequest` 를 생성하는 동시에 메서드와 헤더와 같은 개별적인 components 의 요청 구성을 허용. 또한 요청 별 `RequestInterceptor` 와 `Encodable` 매개변수도 허용.

> Parameters 와 ParameterEncoding 타입을 활용한 추가 메소드는 더 이상 권장되지 않으며 제거되었다. API 의 두번째 버전은 더 간단하다.

```
open func request(_ urlRequest: URLRequestConvertible, 
                  interceptor: RequestInterceptor? = nil) -> DataRequest
```

이 메서드는 `URLRequestConvertible` 프로토콜을 준수하는 모든 타입의 `DataRequest` 를 생성한다.

### **HTTP Methods**

`HTTPMethods` 는 RFC 7231 §4.3 에서 정의된 HTTP methods 이다.

```
public struct HTTPMethod: RawRepresentable, Equatable, Hashable {
    public static let connect = HTTPMethod(rawValue: "CONNECT")
    public static let delete = HTTPMethod(rawValue: "DELETE")
    public static let get = HTTPMethod(rawValue: "GET")
    public static let head = HTTPMethod(rawValue: "HEAD")
    public static let options = HTTPMethod(rawValue: "OPTIONS")
    public static let patch = HTTPMethod(rawValue: "PATCH")
    public static let post = HTTPMethod(rawValue: "POST")
    public static let put = HTTPMethod(rawValue: "PUT")
    public static let trace = HTTPMethod(rawValue: "TRACE")

    public let rawValue: String

    public init(rawValue: String) {
        self.rawValue = rawValue
    }
}
```

이런 값들은 `AF.request` 의 `method` argument 로 전달 가능.

```
AF.request("https://httpbin.org/get")
AF.request("https://httpbin.org/post", method: .post)
AF.request("https://httpbin.org/put", method: .put)
AF.request("https://httpbin.org/delete", method: .delete)
```

중요한 점은 HTTP methods 가 다름에 따라 다른 의미를 가질 수 있으며 서버가 기대하는 것에 따라 다른 매개변수 인코딩이 필요하는 것.

Alamofire 의 `HTTPMethod` 타입이 지원하지 않는 http methods 를 사용해야하는 경우 사용자 정의 타입도 확장가능하다.

```
extension HTTPMethod {
    static let custom = HTTPMethod(rawValue: "CUSTOM")
}
```

### **Setting Other `URLRequest` Properties**

Alamofire request 생성 메서드는 가장 일반적인 매개변수를 제공하지만 충분하지 않다. `RequestModifer` 클로저를 사용하여 수정가능. 예를들어 `timeoutInterval` 을 5초로 설정하려면 클로저에서 요청을 수정한다.

```
AF.request("https://httpbin.org/get", requestModifier: { $0.timeoutInterval = 5 }).response(...)
```

`RequestModifier` 는 trailing closure 구문에서도 작동 가능.

```
AF.request("https://httpbin.org/get") { urlRequest in
    urlRequest.timeoutInterval = 5
    urlRequest.allowsConstrainedNetworkAccess = false
}
.response(...)
```

`RequestModifier` 는 URL 과 개별적인 components 를 사용하여 생성된 요청에만 적용되며, `URLRequestConvertible` 값에서 생성된 값에는 모든 매개변수를 직접 설정할 수 있어야 한다. 또한 대부분의 요청을 생성하는 동안 수정해야하기 시작하면 `URLRequestConvertible` 을 채택하는 것이 좋다.

### **Request Parameters and Parameter Encoders**

Alamofire 는 request 의 매개변수로 모든 `Encodable` 타입 지원한다.

> Encodable : 자신을 외부표현(JSON 이라고 생각하면 쉽다.)으로 인코딩 할 수 있는 타입.

이러한 매개변수는 `ParameterEncoder` 프로토콜을 준수하는 유형을 통해서 전달되고 `URLRequest` 에 추가되어 네트워크를 통해 전송. Alamofire 에는 `JSONParameterEncoder` 와 `URLEncoderFormParameterEncoder` 의 두가지가 포함된다.

```
struct Login: Encodable {
    let email: String
    let password: String
}

let login = Login(email: "test@test.test", password: "testPassword")

AF.request("https://httpbin.org/post",
           method: .post,
           parameters: login,
           encoder: JSONParameterEncoder.default).response { response in
    debugPrint(response)
}
```

### **`URLEncodedFormParameterEncoder`**

`URLEncodedFormParameterEncoder` 는 기존 URL query stirng 또는 요청의 HTTP body 로 설정될 URL 인코딩 문자열로 값을 인코딩. destination of the encoding 을 설정하여 인코딩 된 문자열이 설정된 위치를 제어 가능. `URLEncodedFormParameterEncoder.Destination` 에는 세가지 경우가 있다.

-   `.methodDependent` - 인코딩 된 query 문자열 결과를 .get, .head, .delete 요청에 대한 기존 query 문자열에 적용하고 다른 HTTP 메소드의 요청에 대한 HTTP body 로 설정.
-   `.queryString` - request `URL` query 에 인코딩 된 문자열을 설정하거나 추가.
-   `.httpBody` - 인코딩된 문자열을 `URLRequest` 의 HTTP body 로 설정.

HTTP body 와 함께 encoded 된 request 의 `Content-Type` HTTP header 는 기본값이 `application/x-www-form-urlencoded; charset=utf-8`로 설정된다.

**GET Request With URL-Encoded Parameters**

```
let parameters = ["foo": "bar"]

// All three of these calls are equivalent
AF.request("https://httpbin.org/get", parameters: parameters) // encoding defaults to `URLEncoding.default`
AF.request("https://httpbin.org/get", parameters: parameters, encoder: URLEncodedFormParameterEncoder.default)
AF.request("https://httpbin.org/get", parameters: parameters, encoder: URLEncodedFormParameterEncoder(destination: .methodDependent))

// https://httpbin.org/get?foo=bar
```

**POST Request With URL-Encoded Parameters**

```
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

### **Configuring the Sorting of Encoded Values**

Swift 4.2 이후로 딕셔너리 타입에서 사용하는 해싱 알고리즘은 런타임에 앱 실행마다 다른 임의의 내부 순서를 생성. 이로인해 인코딩된 매개변수의 순서 달라져 기타동작에 영향을 미칠 수 있다.`URLEncodedFormEncoder` 는 기본적으로 인코딩된 key-value 쌍을 정렬한다. `Encodable` 타입에 대해서 상수출력을 제공하지만 실제 인코딩 순서와 일치하지 않을 수 있다. `alphabetizeKeyValuePairs` 를 `false` 로 설정하여 구현순서로 돌아갈 수 있지만, 이 경우에도 `Dictionary` 순서가 무작위로 지정된다.

사용자의 `URLEncodedFormParameterEncorder` 를 만들고 `URLEncodedFormEncoder` 의 이니셜라이저에서 요구하는 `alphabetizeKeyValuePairs`를 지정할 수 있다.

`let encoder = URLEncodedFormParameterEncoder(encoder: URLEncodedFormEncoder(alphabetizeKeyValuePairs: false))`

### **Configuring the Encoding of `Array` Parameters**

collection 타입을 인코딩하는 방법에 대해 제시된 특별한 사양이 없기 때문에 기본적으로 Alamofire 는 key 에 \[\] 를 추가한다. `foo [] = & foo [] = 2` 형식으로 인코딩합니다.

`URLEncodedFormEncoder.ArrayEncoding` 열거형은 `Array` 매개변수의 인코딩을 위해서 다음의 메서드를 제공합니다.

-   `.brackets` - 모든 값에 대해 key에 \[\] 가 추가된다. 이것이 기본이다.
-   `.noBrackets` - \[\] 가 추가되지 않는다. 키는 있는 그대로 인코딩된다.

기본적으로 Alamofire 는 `.brackets` 인코딩을 사용한다. `foo = [1,2]` 는 `foo[]=1&foo[]=2` 으로 인코딩된다.

`.noBrackets` 을 사용하면 `foo = [1,2]` 는 `foo=1&foo=2` 으로 인코딩된다.

고유한 `URLEncodedFormParameterEncoder` 를 만들고 전달된 `URLEncodedFormEncoder` 의 이니셜라이저에서 요구하는 `ArrayEncoding` 을 지정할 수 있다.

```
let parameters: [String: [String]] = [
    "foo": ["1", "2"]
]
let encoder = URLEncodedFormParameterEncoder(encoder: URLEncodedFormEncoder(arrayEncoding: .noBrackets))

AF.request("https://httpbin.org/post", method: .post, prameters, encoder: encoder)

//출처ㅣ https://velog.io/@wimes/Alamofire-%EB%B6%84%EC%84%9D
```

### **Configuring the Encoding of Bool Parameters**

`URLEncodedFormEncoder.BoolEncoding` 열거형은 `Bool` 매개변수를 인코딩하기위해서 다음 메서드를 제공합니다.

-   `.numeric` - `true` 를 1 로 `false` 를 0 으로 인코딩. 기본케이스이다.
-   `.literal` - `true`, `false` 를 문자열 리터럴로 인코딩.

고유한 `URLEncodedFormParameterEncoder` 를 만들고 전달된 `URLEncodedFormEncoder` 의 이니셜라이저에서 요구하는 `BoolEncoding` 을 지정할 수 있다.

```
let encoder = URLEncodedFormParameterEncoder(encoder: URLEncodedFormEncoder(boolEncoding: .numeric))
```

### **Configuring the Encoding of `Data` Parameters**

`DataEncoding` 은 `Data` 인코딩을 위해서 다음의 메서드를 포함합니다.

-   `.deffedToData` - 데이터의 기본 `Encodable` 지원을 사용.
-   `.base64` - `Data` 를 Base64 로 인코딩된 문자열로 인코딩. 기본 케이스이다.
-   `.custom((Data) -> throws -> String)` - 주어진 클로저를 사용하여 `Data` 를 인코딩.

고유한 `URLEncodedFormParameterEncoder` 를 만들고 전달된 `URLEncodedFormEncoder` 의 이니셜라이저에서 요구하는 `DataEncoding` 을 지정할 수 있다.

```
let encoder = URLEncodedFormParameterEncoder(encoder: URLEncodedFormEncoder(dataEncoding: .base64))
```

### **Configuring the Encoding of `Date` Parameters**

`Date` 를 `String` 으로 인코딩하려는 수많은 방법을 고려할 때 `DateEncoding` 에는 `Date` 매개변수를 인코딩하는 다음 메서드를 포함한다.

-   `.deferredToDate` - `Date` 의 `Encodable` 지원을 사용. 기본 케이스이다.
-   `.secondsSince1970` - 1970년 1월 1일 자정 UTC 이후 날짜를 초로 인코딩.
-   `.millisecondsSince1970` - 1970년 1월 1일 자정 UTC 이후 날짜를 milliseconds 로 인코딩.
-   `.iso8601` - ISO 8601 과 RFC3339 표준에 따라 날짜를 인코딩.
-   `.formatted(DateFormatter)` - 주어진 `DataFormatter` 를 사용해서 날짜를 인코딩.
-   `.custom((Date) throws -> String)` - 주어진 클로저를 사용햇 날짜를 인코딩.

고유한 `URLEncodedFormParameterEncoder` 를 만들고 전달된 `URLEncodedFormEncoder` 의 이니셜라이저에서 요구하는 `DateEncoding` 을 지정할 수 있다.

```
let encoder = URLEncodedFormParameterEncoder(encoder: URLEncodedFormEncoder(dateEncoding: .iso8601))
```

### **Configuring the Encoding of Coding Keys**

다양한 매개변수 key 스타일 때문에 `KeyEncoding` 은 `lowerCamelCase` 의 키에서 키 인코딩을 커스터마이즈 할 수 있는 메서드를 제공한다.

-   `.useDefaultKeys` - 각 타입에 지정된 키를 사용. 기본 케이스이다.
-   `.convertToSnakeCase` - `oneTwoThree` becomes `one_two_three`.
-   `.conVertToKebabCase` - `oneTwoThree` becomes `one-two-three`.
-   `.capitalized` - a.k.a `UpperCamelCase`: `oneTwoThree` becomes `OneTwoThree`.
-   `.uppercased` - `oneTwoThree` becomes `ONETWOTHREE`
-   `.lowecased` - `oneTwoThree` becomes `onetwothree`.
-   `.custom((String) -> String)` - 주어진 클로저를 사용해서 인코딩.

고유한 `URLEncodedFormParameterEncoder` 를 만들고 전달된 `URLEncodedFormEncoder` 의 이니셜라이저에서 요구하는 `KeyEncoding` 을 지정할 수 있다.

```
let encoder = URLEncodedFormParameterEncoder(encoder: URLEncodedFormEncoder(keyEncoding: .convertToSnakeCase))
```

### **Configuring the Encoding of Spaces**

이전 형식의 인코더는 `+` 를 사용해서 공백을 인코딩하고 일부 서버는 여전히 최신 백분율 인코딩 대신에 이 인코딩을 기대하므로 alamofire 는 공백코딩을 위해서 다음 메서드를 포함한다.

-   `.percentEscaped` - 표준 퍼센트 이스케이핑을 적용해서 공백문자를 인코딩한다. `" "` 는 `%20` 으로 인코딩. 이것이 기본 케이스이다.
-   `.plusReplaced` - `" "` 를 `+` 로 대체해서 인코딩.

고유한 `URLEncodedFormParameterEncoder` 를 만들고 전달된 `URLEncodedFormEncoder` 의 이니셜라이저에서 요구하는 `SpaceEncoding` 을 지정할 수 있다.

```
let encoder = URLEncodedFormParameterEncoder(encoder: URLEncodedFormEncoder(spaceEncoding: .plusReplaced))
```

### **`JSONParameterEncoder`**

`JSONParameterEncoder` 은 Swift 의 `JSONEncoder` 를 사용해서 `Encodable` 값을 인코딩하고 결과를 URLRequest 의 httpBody 로 설정한다. `Content-Type` HTTP header 필드의 값은 `application/json` 으로 기본값을 가진다.

-   \*POST Request with JSON-Encoder Parameters

```
let parameters: [String: [String]] = [
    "foo": ["bar"],
    "baz": ["a", "b"],
    "qux": ["x", "y", "z"]
]

AF.request("https://httpbin.org/post", method: .post, parameters: parameters, encoder: JSONParameterEncoder.default)
AF.request("https://httpbin.org/post", method: .post, parameters: parameters, encoder: JSONParameterEncoder.prettyPrinted)
AF.request("https://httpbin.org/post", method: .post, parameters: parameters, encoder: JSONParameterEncoder.sortedKeys)

// HTTP body: {"baz":["a","b"],"foo":["bar"],"qux":["x","y","z"]}
```

### **HTTP Headers**

```
let headers: HTTPHeaders = [
    "Authorization": "Basic VXNlcm5hbWU6UGFzc3dvcmQ=",
    "Accept": "application/json"
]

AF.request("https://httpbin.org/headers", headers: headers).responseJSON { response in
    debugPrint(response)
}
```

위의 headers 를 아래와 같은 형태로도 사용 가능.

```
let headers: HTTPHeaders = [
    .authorization(username: "Username", password: "Password"),
    .accept("application/json")
]

AF.request("https://httpbin.org/headers", headers: headers).responseJSON { response in
    debugPrint(response)
}
```

> 변경되지 않는 HTTP headers 경우, URLSession 을 URLSessionConfiguration 에서 설정하여 생성 된 모든 URLSessionTask에 자동으로 적용되도록하는 것이 좋습니다

URLSessionConfiguration.af.default를 사용하여 Alamofire의 기본 헤더를 유지하는 구성을 사용자정의할 수 있다.

### **Response Validation**

기본적으로 Alamofire 는 request 의 내용에 관계 없이 완료된 요청은 성공을 처리한다. 응답 처리 전에 `validate()` 를 호출하면 응답에 허용되지 않는 상태코드 또는 MIME 유형이 있는 경우 오류가 생성된다.

**Automatic Validation**

`validate()` 는 자동으로 200..<300 범위 내 상태코드가 있고 `Content-Type` 헤더가 request 의 `Accept` 헤더와 일치하는지 유효성을 검사한다.

```
AF.request("https://httpbin.org/get").validate().responseJSON { response in
    debugPrint(response)
}
//Manual Validation
AF.request("https://httpbin.org/get")
    .validate(statusCode: 200..<300)
    .validate(contentType: ["application/json"])
    .responseData { response in
        switch response.result {
        case .success:
            print("Validation Successful")
        case let .failure(error):
            print(error)
        }
    }
```

### **Response Handling**

Alamofire 의 `DataRequest` 와 `DownloadRequest` 에 대한 response type : `DataResponse<Success, Failure: Error>` 와 `<DownloadResponse<Success, Failure: Error>`

`DataRequest` 를 처리하려면 responseJSON 과 같은 response handler 를 `DataRequest` 에 연결해야한다.

```
AF.request("https://httpbin.org/get").responseJSON { response in
    debugPrint(response)
}
```

서버의 응답을 기다리기위해서 실행을 차단하는 대신 closure 가 응답을 수신 한 후 처리하기 위해 콜백으로 추가됩니다. request 의 결과는 오직 response closure 에서 사용 가능하고 응답 혹은 데이터에 따른 실행은 response closure 내에서 수행되어야 한다.

> Alamofire 는 asynchronously 비동기적으로 수행됨.

Alamofire 는 6개의 response handlers 를 기본으로 가지고 있다.

```
// Response Handler - Unserialized Response
func response(queue: DispatchQueue = .main, 
              completionHandler: @escaping (AFDataResponse<Data?>) -> Void) -> Self

// Response Serializer Handler - Serialize using the passed Serializer
func response<Serializer: DataResponseSerializerProtocol>(queue: DispatchQueue = .main,
                                                          responseSerializer: Serializer,
                                                          completionHandler: @escaping (AFDataResponse<Serializer.SerializedObject>) -> Void) -> Self

// Response Data Handler - Serialized into Data
func responseData(queue: DispatchQueue = .main,
                  dataPreprocessor: DataPreprocessor = DataResponseSerializer.defaultDataPreprocessor,
                  emptyResponseCodes: Set<Int> = DataResponseSerializer.defaultEmptyResponseCodes,
                  emptyRequestMethods: Set<HTTPMethod> = DataResponseSerializer.defaultEmptyRequestMethods,
                  completionHandler: @escaping (AFDataResponse<Data>) -> Void) -> Self

// Response String Handler - Serialized into String
func responseString(queue: DispatchQueue = .main,
                    dataPreprocessor: DataPreprocessor = StringResponseSerializer.defaultDataPreprocessor,
                    encoding: String.Encoding? = nil,
                    emptyResponseCodes: Set<Int> = StringResponseSerializer.defaultEmptyResponseCodes,
                    emptyRequestMethods: Set<HTTPMethod> = StringResponseSerializer.defaultEmptyRequestMethods,
                    completionHandler: @escaping (AFDataResponse<String>) -> Void) -> Self

// Response JSON Handler - Serialized into Any Using JSONSerialization
func responseJSON(queue: DispatchQueue = .main,
                  dataPreprocessor: DataPreprocessor = JSONResponseSerializer.defaultDataPreprocessor,
                  emptyResponseCodes: Set<Int> = JSONResponseSerializer.defaultEmptyResponseCodes,
                  emptyRequestMethods: Set<HTTPMethod> = JSONResponseSerializer.defaultEmptyRequestMethods,
                  options: JSONSerialization.ReadingOptions = .allowFragments,
                  completionHandler: @escaping (AFDataResponse<Any>) -> Void) -> Self

// Response Decodable Handler - Serialized into Decodable Type
func responseDecodable<T: Decodable>(of type: T.Type = T.self,
                                     queue: DispatchQueue = .main,
                                     dataPreprocessor: DataPreprocessor = DecodableResponseSerializer<T>.defaultDataPreprocessor,
                                     decoder: DataDecoder = JSONDecoder(),
                                     emptyResponseCodes: Set<Int> = DecodableResponseSerializer<T>.defaultEmptyResponseCodes,
                                     emptyRequestMethods: Set<HTTPMethod> = DecodableResponseSerializer<T>.defaultEmptyRequestMethods,
                                     completionHandler: @escaping (AFDataResponse<T>) -> Void) -> Self
```

response handlers 는 서버에서 반환되는 `HTTPURLResponse` 의 유효성 검사를 하지 않는다.

> 즉 400..<500와 500..<600 범위 내의 status codes 에 대해서 error 를 트리거하지 않는다. validate() 를 통해서 진행.

_serialize : 직렬화. oop언어에서 특수하게 가공하는 것을 의미. encode 와 같다고 보면 된다._

**Response Handler**

`response` handler 는 response data 에 대해서 검증하지 않는다.(`.success` , `.failure` 없다.) `URLSessionDelegate` 로 직접 모든 정보를 전달한다.

```
AF.request("https://httpbin.org/get").response { response in
    debugPrint("Response: \(response)")
}

```

> Response 와 Result type 을 활용할 수 있는 다른 response serializers 를 사용하는 것을 권장한다.

**Response Data Handler**

`responseData` handler 는 `DataResponseSerializer` 사용해서 서버에서 반환된 `Data` 를 추출하고 유효성 검사한다. 오류가 발생하지 않고 서버로 부터 `Data` 가 반환되었으면 `Result` 는 `.success` 이다. 그리고 `value` 는 서버에서 반환받은 `Data` 가 된다.

```
AF.request("https://httpbin.org/get").responseData { response in
    debugPrint("Response: \(response)")
}
```

**Response String Handler**

`responseString` handler 는 `StringResponseSerializer` 를 사용해서 서버에서 반환된 `Data` 를 지정된 인코딩을 사용하는 `String` 으로 변환해준다. 오류가 발생하지 않고 서버 데이터가 성공적으로 `String` 으로 직렬화되면 `Result` 는 `.success` 이고 `value` 는 문자열이 된다.

```
AF.request("https://httpbin.org/get").responseString { response in
    debugPrint("Response: \(response)")
}
```

**Response JSON Hanlder**

`responseJSON` handler 는 `JSONResponseSerializer` 를 사용해서 서버에서 반환된 `Data` 를 지정된 `JSONSerializer.ReadingOptions` 를 사용해서 `Any` 타입으로 변환한다. 오류가 발생하지 않고 서버 데이터가 성공적으로 JSON object 로 직렬화 되면 `AFResult` 는 `.success` 가 되고 `value` 는 `Any` 타입이 된다.

```
AF.request("https://httpbin.org/get").responseJSON { response in
    debugPrint("Response: \(response)")
}
```

> responseJSON 의 JSON 직렬화는 Foundation 프레임워크의 JSONSerilaization API 에 의해 처리된다.

**Response `Decodable` Handler**

`responseDecodable` handler 는 `DecodableResponseSerializer` 를 사용해서 서버로부터 반환된 `Data` 를 지정된 `DataDecoder` 를 사용해서 전달된 `Decodable` 타입으로 변환한다. 오류가 발생하지 않고 서버 데이터가 성공적으로 `Decodable` 타입으로 디코딩되면 `Result` 는 `.success` 가 되고 `value` 는 전달된 타입이 된다.

```
struct HTTPBinResponse: Decodable { let url: String }

AF.request("https://httpbin.org/get").responseDecodable(of: HTTPBinResponse.self) { response in
    debugPrint("Response: \(response)")
}
```

### Downloading data to a file

data 를 메모리로 가져오는 것 외에도 `Alamofire` 는 disk 로의 다운로딩을 위해서 `Session.download`, `DownloadRequest`, `DownloadResponse<Success, Failure: Error>` API 를 제공한다.

```
AF.download("https://httpbin.org/image/png").responseURL { response in
    // Read file from provided URL.
}
```

`responseURL` 은 다른 응답 핸들러들과 달리 다운로드 된 데이터의 위치가 포함 된 `URL` 만 반환하고 disk 에서 `Data` 를 읽지 않는다.

`responseDecodable` 과 같은 other response hanlders 는 disk 에서 `Data` 읽기가 가능하다. 이 경우 큰 데이터를 메모리로 읽는 것이 포함 될 수 있으므로 이런 핸들러를 사용할 때 염두해두어야 한다.

### Download File Destination

다운로드 된 모든 데이터는 system temporary directory 에 저장된다. 결국 언젠가는 삭제가 되므로 더 오래 유지해야할 경우 다른 곳으로 이동시켜야 한다.

final destination 으로 옮기기 위해서 `Destinaion` closure 를 제공할 수 있다. `destinationURL` 로 이동하기 전에 closure 에 지정된 옵션이 실행된다. 두가지 옵션:

-   `.createIntermediateDirectories` - 지정된 경우 destination URL 에 대한 intermediate directories 를 만든다.
-   `.removePreviousFile` - 지정된 경우 destionation URL 의 이전 파일을 삭제한다.

```
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

```
let destination = DownloadRequest.suggestedDownloadDestination(for: .documentDirectory)

AF.download("https://httpbin.org/image/png", to: destination)
```

### **출처**

[Alamofire/Alamofire](https://github.com/Alamofire/Alamofire/blob/master/Documentation/Usage.md#response-handling)
