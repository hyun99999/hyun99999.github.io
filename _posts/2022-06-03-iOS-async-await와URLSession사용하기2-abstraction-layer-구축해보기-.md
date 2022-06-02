---
title:  "iOS) async/await 와 URLSession 사용하기(2) - abstraction layer 구축해보기"
categories:
- iOS

date:   2022-06-03  02:01:00 +0900
author_profile: false
---
### 내용

- Error Handling.
- extension 을 활용한 protocol method 의 기본 구현 설정.
- request query 를 사용한 GET 서버통신을 구현
- existential metatype 활용해서 디코딩 에러일 때 해당 자료형 다루기.
- 전반적으로 Moya 의 구조를 공부하면서 URLSession 을 접목시켜서 진행.

# 구현 부분

- HTTP Method: HTTP 메서드를 가지는 구조체. Moya 차용
- TargetType: 해당 프로토콜을 채택해서 Service 파일을 구현하기 수월하도록 했습니다. Moya 차용
- Network Task: request 를 작업할 방법을 가지는 enum. Moya 차용
- Parameter Encoding: 파라미터를 인코딩하는 방법을 가지는 enum. Moya 차용
- NetworkProvider: TargetType 을 채택한 enum 을 제네릭 형태로 받아서 서버통신을 하기위한 `URLRequest` 를 반환하는 메서드를 가진 구조체.
- ImageFetchProvider: URL 을 가지고 데이터를 다운받아 UIImage 로 반환하는 작업을 하는 Provider. 싱글톤 패턴을 사용.

### HTTP Method

- Moya 의 코드를 그대로 가져왔습니다. 구조체를 사용해서 enum 과 같이 그대로 구현해 놓은 모습입니다. enum 보다 구조체가 확장성이 좋기 때문에 그렇지 않았나 조심스레 생각해봅니다.

```swift
import Foundation

public struct HTTPMethod: RawRepresentable, Equatable, Hashable {
    /// `CONNECT` method.
    public static let connect = HTTPMethod(rawValue: "CONNECT")
    /// `DELETE` method.
    public static let delete = HTTPMethod(rawValue: "DELETE")
    /// `GET` method.
    public static let get = HTTPMethod(rawValue: "GET")
    /// `HEAD` method.
    public static let head = HTTPMethod(rawValue: "HEAD")
    /// `OPTIONS` method.
    public static let options = HTTPMethod(rawValue: "OPTIONS")
    /// `PATCH` method.
    public static let patch = HTTPMethod(rawValue: "PATCH")
    /// `POST` method.
    public static let post = HTTPMethod(rawValue: "POST")
    /// `PUT` method.
    public static let put = HTTPMethod(rawValue: "PUT")
    /// `QUERY` method.
    public static let query = HTTPMethod(rawValue: "QUERY")
    /// `TRACE` method.
    public static let trace = HTTPMethod(rawValue: "TRACE")

    public let rawValue: String

    public init(rawValue: String) {
        self.rawValue = rawValue
    }
}
```

### TargetType

- Moya 의 코드에서 조금 삭제, 수정해서 가져왔습니다. extension 을 활용하여 protocol method 의 기본값을 설정했습니다.

```swift
import Foundation

public protocol TargetType {
    /// The target's base `URL`.
    var baseURLPath: String { get }

    /// The path to be appended to `baseURL` to form the full `URL`.
    var path: String { get }

    /// The HTTP method used in the request.
    var method: HTTPMethod { get }

    /// ✅ 실제로는 이번 프로젝트에서 사용되지 않으나 extension 을 활용하여 protocol method 의 기본값을 설정할 수 있음을 확인.
    /// Provides stub data for use in testing. Default is `Data()`.
    var sampleData: Data { get }

    /// The type of HTTP task to be performed.
    /// ✅ NetworkTask: 서버통신의 작업을 위해서 종류에 따라서 일관된 작업을 하기 위한 목적으로 만듦.
    var task: NetworkTask { get }

    /// The headers to be used in the request.
    var headers: [String: String]? { get }
}

public extension TargetType {

    /// Provides stub data for use in testing. Default is `Data()`.
    var sampleData: Data { Data() }
}
```

### NetworkTask

- Moya 의 구조를 가져와서 그 중 이미지 통신을 할 때 사용할 `requestPlain` 와 영화 목록을 가져올 때 사용할 `requestParameters` 만 구현했습니다.

```swift
import Foundation

public enum NetworkTask {
    
    /// 추가적인 데이터를 더하지 않는다.
    case requestPlain
    
    /// 파라미터를 인코딩해서 request 에 더합니다.
    case requestParameters(parameters: [String : Any], encoding: ParameterEncoding)
}
```

### ParameterEncoding

- Moya 의 구조를 가져와서 그 중 request query 을 사용하는 인코딩 방법만 구현했습니다.

```swift
import Foundation

public enum ParameterEncoding {

    /// ✅ request query 방법으로 URL 에 파라미터를 추가합니다.
    case queryString
}
```

### NetworkProvider

- TargetType 을 채택한 enum 을 제네릭 형태로 받아서 `target` 파라미터를 통해서 일괄적으로 `URLRequest` 를 반환.

```swift
import Foundation

struct NetworkProvider<Target: TargetType> {
    func request(_ target: Target) throws -> URLRequest {
        
        // url path
        let path = target.baseURLPath + target.path
        guard var urlComponents = URLComponents(string: path) else {
            throw DataDownloadError.invalidURLComponents
        }
        
        // ✅ task
        var url: URL?
        let task = target.task
        switch task {
        case .reqiestPlan:
            url = urlComponents.url
        case .requestParameters(let parameters, let encoding):
            switch encoding {
            case .queryString:
                // parameter query
                let queryItemArray = parameters.map {
                    URLQueryItem(name: $0.key, value: $0.value as? String)
                }
                urlComponents.queryItems = queryItemArray
                url = urlComponents.url
            }
        }
        guard let url = url else {
            throw DataDownloadError.invalidURLString
        }
        
        // ✅ method
        var request = URLRequest(url: url)
        request.httpMethod = target.method.rawValue
        
        // ✅ header
        if let headerField = target.headers {
            _ = headerField.map { (key, value) in
                request.addValue(value, forHTTPHeaderField: key)
            }
        }
        
        return request
    }
}
```

### ImageFetchProvider

- 이미지를 다운받아서 UIImage 로 반환. 싱글톤 패턴을 사용.

```swift
import UIKit

struct ImageFetchProvider {
    static let shared = ImageFetchProvider()
    private init() { }
    
    /// ✅ URL 을 가지고 data 를 다운받아서 UIImage 로 변환하는 메서드.
    /// - Parameter urlString: URL 가 될 String 자료형의 값.
    /// - Returns: 다운 받은 data 를 UIImage 로 변환해서 리턴. 변환되지 않는 경우 에러를 던집니다.
    public func fetchImage(with urlString: String) async throws -> UIImage {
        guard let url = URL(string: Const.Path.imageURLPath + urlString) else {
            throw ImageDownloadError.invalidURLString
        }

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

# 적용

### NetworkService

- `TargetType` 프로토콜을  채택해서 서버통신을 하기위한 정보들을 세팅합니다.

```swift
import Foundation

enum NetworkService {
    
    /// ✅ 인기있는 영화 목록을 가져오는 서버통신.
    /// - Parameter page : pagination 을 지원하는 파라미터. Default is nil.
    case popular(page: Int? = nil)
    
    /// ✅ 이미지를 가져오는 서버통신.
    /// - Parameter option : request parameter 로 이미지 사이즈 옵션을 전달하는 파라미터. Default is original.
    /// - Parameter url : option 뒤에 붙을 URL.
    case fetchImage(option: ImageSizeOptions = .original, url: String)
}

/// ✅ 해당 open API  에서 지원하는 이미지 사이즈 옵션.
enum ImageSizeOptions: String {
    case original = "original"
    case w500 = "w500"
}

extension NetworkService: TargetType {
    var baseURLPath: String {
        switch self {
        case .popular(_):
            return Const.Path.baseURLPath
        case .fetchImage(_, _):
            return Const.Path.imageURLPath
        }
    }
    
    var path: String {
        switch self {
        case .popular(let page):
            if let page = page {
                return "/movie/popular/\(page)"
            } else {
                return "/movie/popular"
            }
        case .fetchImage(let option, let url):
            return "/\(option.rawValue)/\(url)"
        }
    }
    
    var method: HTTPMethod {
        switch self {
        case .popular(_):
            return .get
        case .fetchImage(_, _):
            return .get
        }
    }
    
    var task: NetworkTask {
        switch self {
        case .popular(let page):
            let parameters: [String : Any]
            if let page = page {
                parameters = ["api_key" : Const.Key.apiKey,
                              "page" : page]
            } else {
                parameters = ["api_key" : Const.Key.apiKey]
            }
            return .requestParameters(parameters: parameters, encoding: .queryString)
        case .fetchImage(_, _):
            return .reqiestPlan
        }
    }
    
    var headers: [String : String]? {
        switch self {
        case .popular(_):
            return nil
        case .fetchImage(_, _):
            return nil
        }
    }
}
```

### NetworkAPI

- NetworkProvider 를 가지고 해당 case 를 파라미터로 구체화함. 상태 코드와 디코딩 결과를 가지고 에러를 던짐. 싱글톤 패턴 사용.
    - ImageFetchProvider 는 역할이 분명하기 때문에 NetworkProvider 와 NetworkAPI 를 합쳐둔 모양새이다.

```swift
import Foundation

public struct NetworkAPI {
    static let shared = NetworkAPI()
    
    private let provider = NetworkProvider<NetworkService>()
    private init() { }
    
    /// ✅ 인기있는 영화 목록을 가져오는 서버통신 메서드.
    /// - Parameter page : pagination 할 수 있는 매개변수. default value is nil.
    func fetchPopularMovies(page: Int? = nil) async throws -> PopularMovie {
        let request = try provider.request(.popular(page: page))
        
        // MARK: - 통신
        
        let (data, response) = try await URLSession.shared.data(for: request)
        guard let httpResponse = response as? HTTPURLResponse else {
            throw DataDownloadError.invalidServerResponse
        }
        
        let networkResult = try self.judgeStatus(by: httpResponse.statusCode, data, type: PopularMovie.self)

        return networkResult
    }

    /// ✅ 상태코드를 가지고 에러 핸들링하는 메서드.
    /// - Parameter statusCode : 상태 코드.
    /// - Parameter data : 디코딩 할 JSON 객체.
    /// - Parameter type : JSON 객체로 부터 디코딩 당할 값의 자료형.
    private func judgeStatus<T: Codable>(by statusCode: Int, _ data: Data, type: T.Type) throws -> T {
        switch statusCode {
        case 200:
            return try decodeData(from: data, to: type)
        case 400..<500:
            throw NetworkError.requestError(statusCode)
        case 500:
            throw NetworkError.serverError(statusCode)
        default:
            throw NetworkError.networkFailError(statusCode)
        }
    }
    
    /// ✅ 디코딩하고, 에러를 핸들링하는 메서드.
    /// - Parameter data : 디코딩 할 JSON 객체.
    /// - Parameter type : JSON 객체로 부터 디코딩 당할 값의 자료형.
    private func decodeData<T: Codable>(from data: Data, to type: T.Type) throws -> T {        
        guard let decodedData = try? JSONDecoder().decode(T.self, from: data) else {
            throw NetworkError.decodError(toType: T.self)
        }
    
        return decodedData
    }
}
```

# Error Handling

### NetworkError

```swift
import Foundation

/// ✅ 서버통신 시 발생하는 에러.
enum NetworkError: Error {
    
    /// 디코딩 에러.
    /// - Parameter toType : Deciadable 을 채택하는 디코딩 가능한 자료형. existential metatype 이다.
    case decodeError(toType: Decodable.Type)
    
    /// 서버 요청 에러.
    case requestError(_ statusCode: Int)
    
    /// 서버 내부 에러.
    case serverError(_ statusCode: Int)
    
    /// 네트워크 연결 실패 에러.
    case networkFailError(_ statusCode: Int)
}
```

# 사용하기

```swift
override func viewDidLoad() {
        super.viewDidLoad()

        /// ...

        Task {
            do {
                movies = try await getMovie()
                movieCollectionView.reloadData()
            } catch DataDownloadError.invalidURLString {
                print("movie error - invalidURLString")
            } catch DataDownloadError.invalidServerResponse {
                print("movie error - invalidServerResponse")
            } catch NetworkError.decodeError(let type) {
                print("network error - decodeError : \(type)")
            } catch NetworkError.requestError(let statusCode) {
                print("network error - requestError : \(statusCode)")
            } catch NetworkError.serverError(let statusCode) {
                print("network error - serverError : \(statusCode)")
            } catch NetworkError.networkFailError(let statusCode) {
                print("network error - networkFailError : \(statusCode)")
            }
        }
}

private func getMovie() async throws -> [Result] {
    let popularMovie = try await NetworkAPI.shared.fetchPopularMovies()
    return popularMovie.results
}
```
