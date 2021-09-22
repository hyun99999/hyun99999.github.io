---
title:  "iOS) URLSession 에 대해서 알아보자(1/2) - 원리"
categories:
- iOS

date:   2021-09-15  22:26:00 +0900
author_profile: false
---
많이들 알고있는 네트워크 오픈 라이브러리인 Alamofire 는 URLSession 으로 만들었다. URLSession 에 대해서도 알아보고 URLSession 사용법도 알아보자.

## 🛸 URLSession 의 Request Response

다른 HTTP 통신과 마찬가지로 Request 와 Response 를 기본구조를 가진다. 

### Request

- URL 객체를 통해 직접 통신하는 형태
- URLRequest 객체를 만들어서 옵션을 설정하여 통신하는 형태(HTTP 메서드와 HTTP 헤더가 포함)

### Response

- Task 의 completion handler 형태로 response 를 받는 형태
- URLSessionDelegate 를 통해서 지정된 메서드를 호출하는 형태로 response 를 받는 형태

## 🛸 URLSession 서버통신 과정

1. URLSession configuration 을 결정하고 생성.
2. 통신할 URL 과 URLRequest 객체를 설정.
3. 사용할 Task 를 설정하고 completion handler, delegate 메서드 작성.
4. 해당 Task 실행.
5. 완료 후 completion handler 샐행.

개발자 문서 URLSession 에 대해서 간단하게 요약해서 정리해보았다.

## 🛸 [URLSession](https://developer.apple.com/documentation/foundation/urlsession)

네트워크 데이터 전송 작업과 관련된 그룹을 조정하는 객체.

### Overview

**URLSession 클래스 및 관련 클래스는 URL 로 표시된 엔드포인트에서 데이터를 다운로드하고 데이터를 업로드하기 위한 API 를 제공한다.**

이 API 를 사용해서 앱이 실행되지 않을 때 iOS 에서 앱이 suspended 된 동안 백그라운드에서 다운로드를 수행할수도 있다. 관련 [URLSessionDelegate](https://developer.apple.com/documentation/foundation/urlsessiondelegate) 와 [URLSessionTaskDelegate](https://developer.apple.com/documentation/foundation/urlsessiontaskdelegate) 를 사용해서 인증을 지원하고 전송 또는 작업 완료 이벤트를 수신한다.

(위의 delegate 들은 필수는 아니다. 지정하지 않으면 system-provided delegate 를 사용한다. 두 경우 다 completion callboack 을 제공하거나 (URLSessionTaskDelegate 경우)asyn-await 메서드를 사용해서 데이터를 가져와야 한다.)

**Note**

API 를 사용하기 전에 [URL Loding System](https://developer.apple.com/documentation/foundation/url_loading_system#2878017) 주제의 개요를 읽어봐라.

→ URL 로딩 시스템은 https 와 같은 표준 프로토콜이나 사용자 지정 프로토콜을 사용하여 URL 로 식별되는 리소스에 대한 엑세스를 제공하는 것이다.

→ URLSession 인스턴스를 사용하여 하나 이상의 URLSessionTask 인스턴스를 생성한다. 이 인스턴스는 데이터를 가져와서 앱에 리턴하거나, 파일을 다운로드하거나, 원격 위치에 데이터와 파일을 업로드할 수 있다. 

---

앱은 관련 data-transfer 작업 그룹을 조정하는 각가의 URLSession 인스턴스를 하나 이상 만든다. 예를 들어 웹 브라우저를 생성하는 경우 탭 또는 창 하나당 하나의 세션을 생성하거나 interactive 사용을 위한 세션이나, 백그라운드 다운로드를 위한 세션을 생성할 수 있다.

### Types of URL Sessions

URLSession 은 기본 요청에 대한 싱글턴 [shared](https://developer.apple.com/documentation/foundation/urlsession/1409000-shared) 세션을 가진다. shared 클래스 메서드를 호출해서 세션에 액세스한다.

(shared 는 사용자 정의할 수 없고 인증을 수행하는 기능이 제한되거나 앱이 실행되지 않을 때 백그라운드 다운로드 또는 업로드 할 수 없는 등 제한적이다.)

다른 종류 세션의 경우 다음 세가지 configuration 중 하나로 URLSession 을 만든다.

- default session(기본 세션) : shared session 과 비슷하지만 구성할 수 있다는 점이 다르다. 데이터를 점진적으로 가져오기 위해서 delegate 를 할당할 수 있다. (전역 캐시, credential, 쿠키 저장소 객체를 사용하는 구성. 즉, 디스크 기반.)
- Ephemeral session(임시 세션) : 캐시, 쿠키 또는 credential 을 디스크에 쓰지 않는다.
- Background session(백그라운드 세션) : 앱이 running 이지 않을 동안에 콘텐츠를 업로드 및 다운로드를 수행할 수 있다.

[Apple Developer Documentation](https://developer.apple.com/documentation/foundation/urlsession)

```swift
let defaultSession = URLSession(configuration: .default)
```

다음과 같이 configuration 을 설정해서 URLSession 의 종류를 결정할 수 있다.

## 🛸 Request

### URL객체

```swift
let url = URL(string: "https://test.com")
```

### URLRequest 객체

```swift
let url = URL(string: "https://test.com")
let request: URLRequest = URLRequest(url: url)
request.httpMethod = "GET"
request.setValue("application/json", forHTTPHeaderField: "Content-Type")
```

## 🛸 [URLSessionTask](https://developer.apple.com/documentation/foundation/urlsessiontask)

URL session 에서 수행되는 특정 리소스 다운로드와 같은 작업.

### Overview

task 은 항상 세션의 일부이다. 즉, URLSession 인스턴스에서 task 생성 메서드 중 하나를 호출해서 task 를 생성한다. 호출하는 메서드는 task 의 유형에 따라 결정된다.

- `URLSessionDataTask` : Data task 는 리소스를 요청하고 응답받아 NSData 객체로 반환. default, ephemeral, shared session 에서는 지원되지만 background sessions 에서는 지원되지 않는다.
- `URLSessionUploadTask` : Upload task 는 데이터를 업로드할 수 있도록 request body 를 제공한다.  background session 도 지원된다.
- `URLSessionDownloadTask` : Download task 는 리소스를 직접 디스크의 파일로 다운로드한다. 모든 유형의 세션에서 지원된다.
- `URLSessionStreamTask` : Stream task 는 host name 과 port 또는 net service 객체에서 TCP/IP 연결을 설정한다.

task 를 만든 후 `**resume()**` 메서드를 호출해서 작업을 시작한다. 요청이 완료되거나 실패할 때까지 세션은 강력한 참조를 유지한다. 

### Controlling the Task State

- [func cancel()](https://developer.apple.com/documentation/foundation/urlsessiontask/1411591-cancel)

Cancels the task.

- [func resume()](https://developer.apple.com/documentation/foundation/urlsessiontask/1411121-resume)

Resumes the task, if it is suspended.

- [func suspend()](https://developer.apple.com/documentation/foundation/urlsessiontask/1411565-suspend)

Temporarily suspends a task.

- [var state: URLSessionTask.State](https://developer.apple.com/documentation/foundation/urlsessiontask/1409888-state)

The current state of the task—active, suspended, in the process of being canceled, or completed.

- [var priority: Float](https://developer.apple.com/documentation/foundation/urlsessiontask/1410569-priority)

The relative priority at which you’d like a host to handle the task, specified as a floating point value between `0.0` (lowest priority) and `1.0` (highest priority).

[Apple Developer Documentation](https://developer.apple.com/documentation/foundation/urlsessiontask)

### URLSessionDataTask 생성

GET 요청에 해당.

```swift
// 세션의 delegate 의 메서드를 호출해서 response 를 제공.
func dataTask(with url: URL) -> URLSessionDataTask

func dataTask(with request: URLRequest) -> URLSessionDataTask
```

요청에 대한 응답을 처리할 completion handler 가 필요한 경우 다음으로 구현.

```swift
// delegate 의 메서드 호출이 아닌 completion handler 의 내부에 response 를 제공.
// (인증 문제에 대한 delegate 메서드는 여전히 호출된다.)
func dataTask(with url: URL, 
completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void) -> URLSessionDataTask

func dataTask(with request: URLRequest, 
completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void) -> URLSessionDataTask
```

- url : 검색할 URL.
- request : URL, 캐시 정책, 요청 유형, body data 또는 body stream 등을 제공하는 URL 요청 객체입니다.

**completionHandler**

- data : 서버로부터 전달받은 data.
- response : **HTTP 헤더 및 status code 같은 response metadata 를 제공하는 객체이다.** HTTP 또는 HTTPS 요청을 수행한 경우 실제로 [HTTPURLResponse](https://developer.apple.com/documentation/foundation/httpurlresponse) 객체가 반환된다.
- error : **요청이 실패한 이유를 의미하는 error 객체. 요청이 성공적이면 nil.**

**요청 성공** : data 파라미터에 리소스 데이터. error 파라미터는 nil 이다.

**요청 실패** : data 파라미터는 nil. error 파라미터는 실패에 대한 정보를 포함한다.

서버의 response 를 수신하면 요청의 성공여부와 관계없이 response 에 해당 정보가 포함된다.(response 의 status code 로 요청의 성공을 판별할 수 있다.)

### URLSessionUploadTask 생성

POST, PUT 요청같은 request body 를 가지는 모든 요청에 해당. 업로드 할 Data 또는 file URL 형태로 업로드 요청.

```swift
// taks 는 세션의 delegate 의 메서드를 호출해서 upload's progress 와 response metatdata, response data 등을 제공.
func uploadTask(with request: URLRequest, 
       fromFile fileURL: URL) -> URLSessionUploadTask

func uploadTask(with request: URLRequest, 
           from bodyData: Data) -> URLSessionUploadTask

```

요청에 대한 응답을 처리할 completion handler 가 필요한 경우 다음으로 구현.

```swift
// delegate 의 메서드 호출이 아닌 completion handler 의 내부에 response 를 제공.
// (인증 문제에 대한 delegate 메서드는 여전히 호출된다.)
func uploadTask(with request: URLRequest, 
       fromFile fileURL: URL, 
completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void) -> URLSessionUploadTask

func uploadTask(with request: URLRequest, 
           from bodyData: Data?, 
completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void) -> URLSessionUploadTask
```

- requset : URL, 캐시 정책, 요청 유형 등을 제공하는 URL 요청 객체입니다. 이 요청 개체의 본문 스트림 및 본문 데이터는 무시됩니다
- fileURL : 업로드 할 파일의 URL.
- bodyData : 요청을 위한 body data.

**completionHandler**

- data : 서버로부터 전달받은 data.
- response : **HTTP 헤더 및 status code 같은 response metadata 를 제공하는 객체이다.** HTTP 또는 HTTPS 요청을 수행한 경우 실제로 [HTTPURLResponse](https://developer.apple.com/documentation/foundation/httpurlresponse) 객체가 반환된다.
- error : **요청이 실패한 이유를 의미하는 error 객체. 요청이 성공적이면 nil.**

**요청 성공** : data 파라미터에 리소스 데이터. error 파라미터는 nil 이다.

**요청 실패** : data 파라미터는 nil. error 파라미터는 실패에 대한 정보를 포함한다.

서버의 response 를 수신하면 요청의 성공여부와 관계없이 response 에 해당 정보가 포함된다.(response 의 status code 로 요청의 성공을 판별할 수 있다.)

### URLSessionDownTask 생성

```swift
// taks 는 세션의 delegate 의 메서드를 호출해서 progress notifications, 결과 임시파일의 위치 등을 제공.
func downloadTask(with url: URL) -> URLSessionDownloadTask

func downloadTask(with request: URLRequest) -> URLSessionDownloadTask
```

요청에 대한 응답을 처리할 completion handler 가 필요한 경우 다음으로 구현.

```swift
// delegate 의 메서드 호출이 아닌 completion handler 의 내부에 response 를 제공.
// (인증 문제에 대한 delegate 메서드는 여전히 호출된다.)
func downloadTask(with url: URL, 
completionHandler: @escaping (URL?, URLResponse?, Error?) -> Void) -> URLSessionDownloadTask

func downloadTask(with request: URLRequest, 
completionHandler: @escaping (URL?, URLResponse?, Error?) -> Void) -> URLSessionDownloadTask
**// completion handler 에서 location 에 해당하는 것이 URL 자료형을 가진 첫번째 파라미터이다.**
```

- url : 다운로드 할 URL.
- request : URL, 캐시 정책, 요청 유형, body data 또는 body stream 등을 제공하는 URL 요청 객체입니다.

**completionHandler**

- **location : 서버의 response 가 저장되는 임시파일의 위치.** completion handler 가 리턴되기 전까지 읽기 위해서 열어야 한다. 그렇지 않으면 파일이 삭제되고 손상된다.
- error : **요청이 실패한 이유를 의미하는 error 객체. 요청이 성공적이면 nil.**

**요청 성공** : location 파라미터에 임시 파일의 위치. error 파라미터는 nil 이다.

**요청 실패** : location 파라미터는 nil. error 파라미터는 실패에 대한 정보를 포함한다.

서버의 response 를 수신하면 요청의 성공여부와 관계없이 response 에 해당 정보가 포함된다.(response 의 status code 로 요청의 성공을 판별할 수 있다.)

간단히 사용하는 예제를 살펴보자.

```swift
        let defaultSession = URLSession(configuration: .default)

        guard let url = URL(string: "https://test.com") else { return }
    
        // Request
        let request = URLRequest(url: url)

        // Task
        let dataTask = defaultSession.dataTask(with: request) { (data: Data?, response: URLResponse?, error: Error?) in
            guard error == nil else {
                print("Error occur: \(String(describing: error))")
                return
            }

            guard let data = data, let response = response as? HTTPURLResponse, response.statusCode == 200 else {
                return
            }

            // data 를 처리하는 코드...

        }.resume()
```

---

### 참조

[URLSession Tutorial: Getting Started](https://www.raywenderlich.com/3244963-urlsession-tutorial-getting-started)

[URLSession Tutorial: Getting Started 번역본](https://o-o-wl.tistory.com/50)

[[iOS/Swift] HTTP/HTTPS 통신의 기본, URLSession](https://tngusmiso.tistory.com/50)
