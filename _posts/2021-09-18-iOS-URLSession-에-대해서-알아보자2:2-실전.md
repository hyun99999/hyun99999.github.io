---
title:  "iOS) URLSession 에 대해서 알아보자(2/2) - 실전"
categories:
- iOS

date:   2021-09-18  19:23:00 +0900
author_profile: false
---
서버통신에서 가장 많이 사용하는 GET, POST 통신의 다양한 경우에 대한 코드를 작성해보겠다.

- GET 통신
    - 기본적인 URL 만 가진 GET 통신
    - query parameter 를 가지는 GET 통신
    - http header 를 가지는 GET 통신
    - file 을 download 해서 임시저장하는 GET 통신(URLSessionDownloadTask 사용)
- Post 통신
    - request body 를 가지는 POST 통신(URLSessionDataTask 사용)
    - request body 를 가지는 POST 통신(URLSessionUploadTask 사용)
    - request body 에 image 를 포함한 multipart/form-data POST 통신

## ⛑ 프로젝트 설정

https 가 아닌 http 로 진행할 경우 다음과 같이 info.plist 를 설정해주어야 한다.

<img width="600" alt="스크린샷 2021-09-18 오후 5 19 46" src="https://user-images.githubusercontent.com/69136340/133885595-a285ac96-8756-4a80-8e1c-19487a1a9d4f.png">

## ⛑ Model 설정

다음은 임시로 만든 model 이다. 서버에서 넘겨주는 JSON 에 맞춰서 바꿔주면 된다.

```swift
// 📌 Model.swift
import Foundation

// MARK: - Model
struct Model: Codable {
    let code: Int
    let data: DataClass
    let message: String
}

// MARK: - DataClass
struct DataClass: Codable {
}
```

## ⛑ GET 통신

- 기본적인 URL 만 가진 GET 통신

```swift
// GET
    func requestGET(url: String, completionHandler: @escaping (Bool, Any) -> Void) {
        guard let url = URL(string: url) else {
            print("Error: cannot create URL")
            return
        }

        // Request
        var request = URLRequest(url: url)
        request.httpMethod = "GET"

        // Task
        let defaultSession = URLSession(configuration: .default)
        
        defaultSession.dataTask(with: request) { (data: Data?, response: URLResponse?, error: Error?) in
            guard error == nil else {
                print("Error occur: error calling GET - \(String(describing: error))")
                return
            }

            guard let data = data, let response = response as? HTTPURLResponse, (200..<300) ~= response.statusCode else {
                print("Error: HTTP request failed")
                return
            }

            guard let output = try? JSONDecoder().decode(Model.self, from: data) else {
                print("Error: JSON data parsing failed")
                return
            }
            
            completionHandler(true, output.data)
        }.resume()
    }
```

- query parameter 를 가지는 GET 통신

```swift
// GET query parameter
    func requestGETWithQuery(url: String, query: [String : String], completionHandler: @escaping (Bool, Any) -> Void) {
        // URLComponents 를 사용하면 qery parameter 부분을 URLQueryItem 객체로 쉽게 추가할 수 있다.
        guard var urlComponents = URLComponents(string: url) else {
            print("Error: cannot create URLComponents")
            return
        }
        
        // ✅ query 추가
        let queryItemArray = query.map {
            URLQueryItem(name: $0.key, value: $0.value)
        }
        urlComponents.queryItems = queryItemArray
        
        // URL
        guard let requestURL = urlComponents.url else { return }
        
        // request method 는 기본적으로 GET 이다. 혹시나 POST 등을 사용하고 싶다면 URLRequest 을 만들어야한다.
//        var request = URLRequest(url: requestURL)
//        request.httpMethod = "GET"
        
        // Task
        let defaultSession = URLSession(configuration: .default)
        
        defaultSession.dataTask(with: requestURL) { (data: Data?, response: URLResponse?, error: Error?) in
            // ...
        }.resume()
    }
```

- http header 를 가지는 GET 통신

```swift
// GET header
    func requestGETWithHeader(url: String, headerField: [String : String], completionHandler: @escaping (Bool, Any) -> Void) {
        guard let url = URL(string: url) else {
            print("Error: cannot create URL")
            return
        }

        // Request
        var request = URLRequest(url: url)
        request.httpMethod = "GET"
        
        // ✅ header 추가
        _ = headerField.map { (key, value) in
            request.addValue(key, forHTTPHeaderField: value)
        }

        // Task
        let defaultSession = URLSession(configuration: .default)
        
        defaultSession.dataTask(with: request) { (data: Data?, response: URLResponse?, error: Error?) in
            // ...
        }.resume()
    }
```

- file 을 download 해서 임시저장하는 GET 통신(URLSessionDownloadTask 사용)

```swift
// GET - Image 수신
    func requestPOSTWithURLSessionDownloadTask(url: String, completionHandler: @escaping (Bool, Any) -> Void) {
        guard let url = URL(string: url) else {
            print("Error: cannot create URL")
            return
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = "GET"
        
        let defaultSession = URLSession(configuration: .default)
        
        defaultSession.downloadTask(with: request) { (location: URL?, response: URLResponse?, error: Error?) in
            guard error == nil else {
                print("Error occur: error calling POST - \(String(describing: error))")
                return
            }

            guard let location = location, let response = response as? HTTPURLResponse, (200..<300) ~= response.statusCode else {
                print("Error: HTTP request failed")
                return
            }
            
            completionHandler(true, location)
        }.resume()
    }
```

## ⛑ POST 통신

URLSessionDataTask 로 URLRequest 에 httpBody 를 설정해서 POST 통신을 할 수 있다. 

또한, POST 통신을 목적으로 한 URLSessionUploadTask 를 사용할 수도 있다.

- `URLSessionDataTask` : Data task 는 리소스를 요청하고 응답받아 NSData 객체로 반환. default, ephemeral, shared session 에서는 지원되지만 background sessions 에서는 지원되지 않는다.
- `URLSessionUploadTask` : Upload task 는 데이터를 업로드할 수 있도록 request body 를 제공한다. background session 도 지원된다.

Alamofire 는 어떤 task 를 사용하는지 해체쇼를 해봤다.. URLSessionUploadTask 를 사용하고 있었다.

```swift
public class UploadRequest: DataRequest {
    //...
    }

    // ...

    override func task(for request: URLRequest, using session: URLSession) -> URLSessionTask {
        // ...
        switch uploadable {
        case let .data(data): return session.uploadTask(with: request, from: data)
        case let .file(url, _): return session.uploadTask(with: request, fromFile: url)
        case .stream: return session.uploadTask(withStreamedRequest: request)
        }

    // ...
}
```

두가지 모두 가능하니 해보겠다.

- request body 를 가지는 POST 통신(URLSessionDataTask 사용)

```swift
// POST - URLSessionDataTask
    func requestPOSTWithURLSessionDataTask(url: String, parameters: [String : Any], completionHandler: @escaping (Bool, Any) -> Void) {
        // ✅ paramters 를 JSON 으로 encode.
        let requestBody = try! JSONSerialization.data(withJSONObject: parameters, options: [])
        
        guard let url = URL(string: url) else {
            print("Error: cannot create URL")
            return
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.addValue("application/json", forHTTPHeaderField: "Content-Type")
        // ✅ request body 추가
        request.httpBody = requestBody
        
        let defaultSession = URLSession(configuration: .default)
        
        defaultSession.dataTask(with: request) { (data: Data?, response: URLResponse?, error: Error?) in
            guard error == nil else {
                print("Error occur: error calling POST - \(String(describing: error))")
                return
            }

            guard let data = data, let response = response as? HTTPURLResponse, (200..<300) ~= response.statusCode else {
                print("Error: HTTP request failed")
                return
            }

            guard let output = try? JSONDecoder().decode(Model.self, from: data) else {
                print("Error: JSON data parsing failed")
                return
            }
            
            completionHandler(true, output.data)
        }.resume()
    }
```

- URLSessionUploadTask 사용

```swift
// POST - URLSessionUploadTask
    func requestPOSTWithURLSessionUploadTask(url: String, parameters: [String : Any], completionHandler: @escaping (Bool, Any) -> Void) {
        // ✅ parameters 를 JSON 으로 encode.
        let uploadData = try! JSONSerialization.data(withJSONObject: parameters, options: [])
        
        guard let url = URL(string: url) else {
            print("Error: cannot create URL")
            return
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.addValue("application/json", forHTTPHeaderField: "Content-Type")
        // request.httpBody = uploadData
        
        let defaultSession = URLSession(configuration: .default)

        // ✅ uploadTask(with:from:) 메서드 사용해서 reqeust body 에 추가.
        defaultSession.uploadTask(with: request, from: uploadData) { (data: Data?, response: URLResponse?, error: Error?) in
            // ...
        }.resume()
    }
```

- request body 에 image 를 포함한 Multipart/form-data POST 통신

multipart/form-data POST 통신에 대해서 코드를 살펴보기 전에 multipart/form-data 에 대해서 알아보자.

따로 글로 정리해보았다. 

[iOS) HTTP Multipart/form-data 이해하기](https://gyuios.tistory.com/107)

본격적으로 코드를 짜보자.

```swift
// POST - Image 송신
    // ✅ data : UIImage 를 pngData() 혹은 jpegData() 사용해서 Data 로 변환한 것.
    // ✅ filename : 파일이름(img.jpg 과 같은 이름)
    // ✅ mimeType :  타입에 맞게 png면 image/png, text text/plain 등 타입.
    func requestPOSTWithMultipartform(url: String,
                                      parameters: [String : String],
                                      data: Data,
                                      filename: String,
                                      mimeType: String,
                                      completionHandler: @escaping (Bool, Any) -> Void) {
        
        guard let url = URL(string: url) else {
            print("Error: cannot create URL")
            return
        }
        
        // ✅ boundary 설정
        let boundary = "Boundary-\(UUID().uuidString)"
        
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("multipart/form-data; boundary=\(boundary)", forHTTPHeaderField: "Content-Type")
        // request.httpBody = uploadData
        
        // ✅ data
        var uploadData = Data()
        let imgDataKey = "img"
        let boundaryPrefix = "--\(boundary)\r\n"
        
        for (key, value) in parameters {
            uploadData.append(boundaryPrefix.data(using: .utf8)!)
            uploadData.append("Content-Disposition: form-data; name=\"\(key)\"\r\n\r\n".data(using: .utf8)!)
            uploadData.append("\(value)\r\n".data(using: .utf8)!)
        }
        
        uploadData.append(boundaryPrefix.data(using: .utf8)!)
        uploadData.append("Content-Disposition: form-data; name=\"\(imgDataKey)\"; filename=\"\(filename)\"\r\n".data(using: .utf8)!)
        uploadData.append("Content-Type: \(mimeType)\r\n\r\n".data(using: .utf8)!)
        uploadData.append(data)
        uploadData.append("\r\n".data(using: .utf8)!)
        uploadData.append("--\(boundary)--".data(using: .utf8)!)
        
        let defaultSession = URLSession(configuration: .default)
        // ✅ uploadTask(with:from:) 메서드 사용해서 reqeust body 에 data 추가.
        defaultSession.uploadTask(with: request, from: uploadData) { (data: Data?, response: URLResponse?, error: Error?) in
            // ...
        }.resume()
    }
```

### 느낀점

URLSession 을 사용해보니까 통신의 경우의 수는 많은데 URLSession 을 통해서 매번 그에 맞는 최적의 인터페이스를 짜주는 기분이었다. 

Alamofire 의 경우에는 `AF.request()` 를 통해서 어떤 통신의 종류든 파라미터가 있든 없든 쉽게 작성가능했다. 또한 `validate()` 를 통해서 유효성 검사도 간편했다. 다양한 response handler 를 만들어 두었기 때문에 이 역시 편리하게 느껴졌다.

URLSession 에 대해서 공부해봤으니 다음에는 Alamofire 를 해체해보려고 한다..🔪슉..슈슉....슈..슈슉..

### 참고

[[iOS - swift] URLSession 네트워크 통신 기본 (URLSessionConfiguration, URLSession, URLComponents, URLSessionTask)](https://ios-development.tistory.com/651)

[POST, 파일 업로드 요청을 URLSession 사용하여 구현해보자.](https://hyunsikwon.github.io/ios/iOS-Networking-01/)

[URLSession을 통한 간단한 업로드 - Swift · Wireframe](https://soooprmx.com/urlsession을-통한-간단한-업로드-swift/)

[[Swift] - MultiPart통신 (멀티파트 이미지업로드)](https://nsios.tistory.com/39)

### 깃허브

[GitHub - hyun99999/URLSessionTutorial-iOS: 🛸 URLSession 서버통신 튜토리얼](https://github.com/hyun99999/URLSessionTutorial-iOS)
