---
title:  "iOS)  Alamofire  해체쇼"
categories:
- iOS

date:   2021-09-19  11:55:00 +0900
author_profile: false
---
따란, URLSession 을 공부하면서 Alamofire 에서는 어떻게 구현했을까? 라는 궁금증에서 시작된 라이브러리 해체쇼 

### 내용

- upload task 와 data task 으로 POST 통신을 구현할 수 있다. Alamofire 에서는 어떻게 구현하고 있을까?
- Multipart/form-data POST 통신을 Alamofire 에서는 어떻게 구현하고 있을까?

## upload task 와 data task 으로 POST 통신을 구현할 수 있다. Alamofire 에서는 어떻게 구현하고 있을까?

URLSessionDataTask 로 URLRequest 에 httpBody 를 설정해서 POST 통신을 할 수 있다. 

또한, POST 통신을 목적으로 한 URLSessionUploadTask 를 사용할 수도 있다.

- `URLSessionDataTask` : Data task 는 리소스를 요청하고 응답받아 NSData 객체로 반환. default, ephemeral, shared session 에서는 지원되지만 background sessions 에서는 지원되지 않는다.
- `URLSessionUploadTask` : Upload task 는 데이터를 업로드할 수 있도록 request body 를 제공한다. background session 도 지원된다.

Alamofire 는 어떤 task 를 사용하는지 해체쇼를 해봤다. URLSessionUploadTask 를 사용하고 있었다.

```swift
// 📌 Request.swift
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

## Multipart/form-data POST 통신을 Alamofire 에서는 어떻게 구현하고 있을까?

- 먼저, URLSession 을 활용해서 직접 구현해보자.

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

### 그렇다면 Alamofire 는 위와 같은 폼을 어떻게 구현하고 있을까?

먼저 간단하게 Alamofire 로 구현해보자

```swift
AF.upload(multipartFormData: { multipartFormData in
    multipartFormData.append(images, withName: "img", fileName: "image1", mimeType: "image/jpeg")

    // ...
}
```

- upload() 메서드는 다음과 같이 구현되어 있다.

```swift
// 📌 Session.swift
open func upload(multipartFormData: @escaping (MultipartFormData) -> Void,
                     to url: URLConvertible,
                     usingThreshold encodingMemoryThreshold: UInt64 = MultipartFormData.encodingMemoryThreshold,
                     method: HTTPMethod = .post,
                     headers: HTTPHeaders? = nil,
                     interceptor: RequestInterceptor? = nil,
                     fileManager: FileManager = .default,
                     requestModifier: RequestModifier? = nil) -> UploadRequest {
// ...
}
```

위의 multipartFormData 파라미터의 append() 메서드로 parameter 들을 추가하고 있다. 알아보자.

```swift
// 📌 MultipartFormData.swift

// 1️⃣
    /// Creates a body part from the data and appends it to the instance.
    ///
    /// The body part data will be encoded using the following format:
    ///
    /// - `Content-Disposition: form-data; name=#{name}; filename=#{filename}` (HTTP Header)
    /// - `Content-Type: #{mimeType}` (HTTP Header)
    /// - Encoded file data
    /// - Multipart form boundary
    ///
    /// - Parameters:
    ///   - data:     `Data` to encoding into the instance.
    ///   - name:     Name to associate with the `Data` in the `Content-Disposition` HTTP header.
    ///   - fileName: Filename to associate with the `Data` in the `Content-Disposition` HTTP header.
    ///   - mimeType: MIME type to associate with the data in the `Content-Type` HTTP header.
    public func append(_ data: Data, withName name: String, fileName: String? = nil, mimeType: String? = nil) {
        let headers = contentHeaders(withName: name, fileName: fileName, mimeType: mimeType)
        let stream = InputStream(data: data)
        let length = UInt64(data.count)

        append(stream, withLength: length, headers: headers)
    }

// ...

// 2️⃣
// MARK: - Private - Content Headers

    private func contentHeaders(withName name: String, fileName: String? = nil, mimeType: String? = nil) -> HTTPHeaders {
        var disposition = "form-data; name=\"\(name)\""
        if let fileName = fileName { disposition += "; filename=\"\(fileName)\"" }

        var headers: HTTPHeaders = [.contentDisposition(disposition)]
        if let mimeType = mimeType { headers.add(.contentType(mimeType)) }

        return headers
    }

// 3️⃣
// ✅ 1️⃣ 에서 마지막 append(_:withLength:headers:) 함수.
    /// - Parameters:
    ///   - stream:  `InputStream` to encode into the instance.
    ///   - length:  Length, in bytes, of the stream.
    ///   - headers: `HTTPHeaders` for the body part.
    public func append(_ stream: InputStream, withLength length: UInt64, headers: HTTPHeaders) {
        let bodyPart = BodyPart(headers: headers, bodyStream: stream, bodyContentLength: length)
        bodyParts.append(bodyPart)
    }

// 4️⃣
class BodyPart {
        let headers: HTTPHeaders
        let bodyStream: InputStream
        let bodyContentLength: UInt64
        var hasInitialBoundary = false
        var hasFinalBoundary = false

        init(headers: HTTPHeaders, bodyStream: InputStream, bodyContentLength: UInt64) {
            self.headers = headers
            self.bodyStream = bodyStream
            self.bodyContentLength = bodyContentLength
        }
    }

// ... 
```

위 부분을 보면 얼추 비슷하게 꾸리고 있다는게 보여진다. 조금 더 해체해 보자.. 슥삭🔪.. 가만히 있거라 요놈아..

```swift
// 📌 HTTPHeaders.swift
// 2️⃣ 와 연결된다.
public static func contentDisposition(_ value: String) -> HTTPHeader {
        HTTPHeader(name: "Content-Disposition", value: value)
    }

// ... 

public static func contentType(_ value: String) -> HTTPHeader {
        HTTPHeader(name: "Content-Type", value: value)
    }
```

그리고 아래의 boundaryDaya() 메서드를 보게되면 폼을 만들수 있는 로직이 있다.

```swift
// 📌 MultipartFormData.swift

open class MultipartFormData {
    // MARK: - Helper Types

    enum EncodingCharacters {
        static let crlf = "\r\n"
    }

    enum BoundaryGenerator {
        enum BoundaryType {
            case initial, encapsulated, final
        }

        static func randomBoundary() -> String {
            let first = UInt32.random(in: UInt32.min...UInt32.max)
            let second = UInt32.random(in: UInt32.min...UInt32.max)

            return String(format: "alamofire.boundary.%08x%08x", first, second)
        }

        static func boundaryData(forBoundaryType boundaryType: BoundaryType, boundary: String) -> Data {
            let boundaryText: String

            switch boundaryType {
            case .initial:
                boundaryText = "--\(boundary)\(EncodingCharacters.crlf)"
            case .encapsulated:
                boundaryText = "\(EncodingCharacters.crlf)--\(boundary)\(EncodingCharacters.crlf)"
            case .final:
                boundaryText = "\(EncodingCharacters.crlf)--\(boundary)--\(EncodingCharacters.crlf)"
            }

            return Data(boundaryText.utf8)
        }
    }
```
