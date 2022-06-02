---
title:  "iOS) URLSession 과 async/await 사용하기(1) - async/await"
categories:
- iOS

date:   2022-06-03  01:59:00 +0900
author_profile: false
---
### 핵심 내용

- Movie open API 를 사용해서 URLSession 으로 서버 통신을 진행할 것이다.
- async/await 를 사용해서 비동기 처리를 동기적으로 사용해보자.

WWDC 21 에서 async/await 가 소개되었습니다.

- [Meet async/await in Swift](https://developer.apple.com/videos/play/wwdc2021/10132/)
- [Use async/await with URLSession](https://developer.apple.com/videos/play/wwdc2021/10095/)

세션들의 일부 내용을 가져와서 기존 completionHandler 의 문제가 무엇이었는지, async/await 는 무엇이고 URLSession 과 어떻게 함께 사용하는지 알아봅시다.

기존에 우리는 비동기 작업에서 completion handler 를 사용해왔어요! 아래의 코드를 async/await 를 사용해서 바꾸어 봅시다.

*(아래는 WWDC21 [Use async/await with URLSession](https://developer.apple.com/videos/play/wwdc2021/10095/) 세션의 일부입니다.)*
<img width="700" alt="1" src="https://user-images.githubusercontent.com/69136340/171682149-39bce35a-e8ed-41fb-a1dd-40212ce6dd77.png">

## 기존의 completion handler 의 문제

- 다음은 제어 흐름을 표시한 것입니다.
<img width="700" alt="2" src="https://user-images.githubusercontent.com/69136340/171682221-c5565484-d659-4393-a029-3c3e31622c3a.png">


보시면 앞뒤로 점프하고 있습니다… 이렇게 제어 흐름이 복잡합니다.

- 스레딩은 어떨까요?
<img width="700" alt="3" src="https://user-images.githubusercontent.com/69136340/171682234-cd42b32b-9176-43af-a82a-70baa598fb07.png">


총 3개의 서로 다른 실행 context 가 있을만큼 놀라울 정도로 복잡합니다.

- 가장 바깥의 레이어는 호출자의 스레드 또는 큐에서 실행되고,
- URLSessionTask completionHandler 는 Session 의 delegate queue 에서 실행되며
- 최종 completionHandler 는 main queue 에서 실행됩니다.

컴파일러는 여기서 도울 수 없기 때문에 data races 와 같은 스레딩 이슈를 피하기 위해서 극도의 주의를 기울여야된다고 합니다.

- 에러가 발생하면 completionHandler 를 두 번 호출할 수 있습니다.

<img width="700" alt="4" src="https://user-images.githubusercontent.com/69136340/171682332-8196a8b9-a299-4795-8ac6-72be398b02b9.png">

이는 caller(호출자)가 만든 가정을 위반할 수도 있습니다.

- 명확하지 않을 수 있지만 UIImage 를 만드데 실패할 수 있습니다.
<img width="700" alt="5" src="https://user-images.githubusercontent.com/69136340/171682401-65e600a6-fe93-4d78-8a4e-206f604d2b81.png">

데이터 형식이 잘못된 경우, UIImage init 은 nil 을 반환하므로 nil 이미지와  nil 에러를 가진 completionHandler 를 호출했을 것입니다.

*(이 부분은 Swift5 부터 도입된 `Result Type` 으로 대비 할 수 있지만 중첩의 문제는 남아있습니다.)*

*(아래는 WWDC21 [Meet async/await in Swift](https://developer.apple.com/videos/play/wwdc2021/10132/)  세션의 일부입니다.)*

- completion 호출이 개발자의 몫이 됩니다.
<img width="700" alt="6" src="https://user-images.githubusercontent.com/69136340/171682422-21210129-4c6f-4b2e-ad74-1ebdad0a0ff7.png">

completionHandler 는 작업 종료시 항상 호출되어야 하는데, 이것은 개발자에게 달려있고, 컴파일러가 검증해주지도 않습니다. 따라서 잠재적으로 버그가 발생할 수 있는 코드가 됩니다.

- 그래서! completionHandler 를 추가했습니다.

<img width="700" alt="7" src="https://user-images.githubusercontent.com/69136340/171682431-21ef06af-165b-4e1d-b532-8a9cad8ca870.png">

결과적으로 총 20여줄의 코드를 작성하게 되었고, 버그가 발생할 수 있는 지점은 5곳이 있습니다…

- 좀 더 안전한 방법에 대해서도 소개해드리겠습니다. 앞서 잠깐 언급한 `Result Type` 을 사용하는 것입니다.
<img width="700" alt="8" src="https://user-images.githubusercontent.com/69136340/171682510-dcb1f174-698e-4815-aaa1-61d4b7a3119f.png">


세션에서는 이것을 좀 더 안전하지만, 코드를 더 못생기고 약간 더 길게 만드는 행위라고 이야기 합니다.

### 🔥 정리하자면..!

- 복잡한 제어흐름과 스레딩.
- 오류 처리에 길어짐과 의도치 않은 결과.
- completionHandler 의 호출을 잊어버리는 점.

문제점이라고 할 수 있겠네요.

## async/await 적용


- 다음은 위의 코드에 asnyc/await 를 사용한 새 버전입니다.
<img width="700" alt="9" src="https://user-images.githubusercontent.com/69136340/171682608-2d90beaa-347e-441a-946d-ec7120280051.png">


- 위에서 아래로의 제어흐름.
    - 비동기 코드를 마치 동기 코드처럼 작성 가능하다. 프로그래머가 동기 코드에서 사용하는 코드 의미 구조를 최대한 활용 가능하다.
    - 위에서 아래로 자연스럽게 코드의 의미를 보존할 수 있다.
- 모두 동일한 concurrency context 에서 실행되기 때문에 스레딩 문제 해결.
- 오류를 throw 해서 Swift 의 기본 오류 처리를 사용해서 처리.
- optional UIImage 를 반환하려고 하면 컴파일러가 알려주기 때문에 nil 처리를 강제.
- 작업이 종료될 때 completionHandler 없이도 호출한 곳에 알려주는 것을 보장.

### iOS 13부터 async/await 를 지원하는 URLSession API 가 추가되었습니다.

Xcode 13.2 릴리즈 노트에서 iOS 15 부터 지원되었던 것이 iOS 13 으로 바뀌었습니다!(야-호)

[Apple Developer Documentation - Xcode 13.2 Release Notes](https://developer.apple.com/documentation/xcode-release-notes/xcode-13_2-release-notes)

- 네트워크로부터 data 를 fetch 하는 대표적인 메서드입니다. 해당 메서드를 사용해서 아래에서 적용해보겠습니다.
<img width="700" alt="10" src="https://user-images.githubusercontent.com/69136340/171682660-545dd301-f677-40da-a286-ef46de41a446.png">

### async

함수 이름 뒤에 `async` 키워드를 붙여서 비동기로 만듭니다.

async 함수는 호출할 때 앞에 `try await` 추가하여 호출하는 thorwing function 일 수 있습니다. `do-catch 문`으로 호출을 래핑하여 처리하면 됩니다.

```swift
func fetchPhoto(url: URL) async throws -> UIImage
```

 `async` 메서드는 concurrent context 에서만 실행할 수 있습니다. 즉, 다른 asnyc 메서드와 Task 를  통해서 수동으로 concurrent context 를 제공할 때 사용할 수 있습니다.

(Task 는 비동기 작업의 단위입니다. 비동기 컨텍스트를 생성해서 동기 컨텍스트에서도 비동기를 호출 할 수 있습니다.)

### await

비동기 함수 호출시 potential suspension point(잠재적인 일시 중단 지점)로 지정합니다.

```swift
let (data, response) = try await URLSession.shared.data(for:request)
```

비동기 처리 함수를 실행 후, 완료될 때까지 기다리기 위해서 potential suspension point 가 있어야하고 `await` 키워드가 필요합니다. 즉, `suspend`. 멈출 수 있다는 것을 의미하는 키워드입니다.

`suspend` 는 해당 스레드가 다른 동작을 수행할 수 있게 제어권을 포기한다는 뜻입니다.

*(`async` 함수가 에러를 던질 수 있다면 `await` 역시 `try` 와 함께 사용해야합니다.)*

- 평범한 함수 호출
<img width="700" alt="11" src="https://user-images.githubusercontent.com/69136340/171682716-e20ff0e1-260a-4c0d-8297-ff3b44e5c3cf.png">


평범한 함수를 호출하는 경우 작업이 끝날 때까지 스레드를 점령하고 있습니다.

- async 함수 호출

<img width="700" alt="12" src="https://user-images.githubusercontent.com/69136340/171683095-cb5cf221-cbc1-4b42-bfe7-057e0681daba.png">

그런데 async 함수는 suspend 될 수 있고, 그 동안 다른 작업이 실행될 수 있습니다.

**어떻게 가능할까요?**

async 함수는 suspend 되면 스레드 제어권을 포기하고, 시스템에게 넘깁니다. 그래서 다른 작업을 할 수 있고, 적절한 시기에 다시 async 함수를 resume 합니다.

async 함수가 끝나면 스레드의 제어권은 해당 함수로 다시 넘어옵니다.

**어떻게 멈춘 함수로 돌아가나요?**

suspension point 에서 유지되는 모든 정보는 힙에 저장되기 때문입니다.

# 적용해보자!

### 내용

- 영화 정보를 조회하는 간단한 뷰 구현.
- Movie API 를 사용해서 asnyc/await 를 적용한 서버통신 구현.

## Movie API

[API Docs](https://developers.themoviedb.org/3/getting-started/introduction)](https://developers.themoviedb.org/3/getting-started/introduction)

위의 오픈 API 를 사용했습니다. 사용 방법에 대해서는 API 문서를 확인할 수 있습니다.

제가 사용한 API 들은 다음과 같습니다.

- 인기있는 영화 목록을 가져오는 API

```swift
[GET] https://api.themoviedb.org/3/movie/popular?api_key="내 api key"

// 출처: https://developers.themoviedb.org/3/movies/get-popular-movies
```

- 영화 포스터 등 이미지를 가져오는 API

```swift
// original, w500 은 이미지 사이즈에 따라 설정하면 됩니다.
[GET] https://image.tmdb.org/t/p/original/"이미지 URL"
[GET] https://image.tmdb.org/t/p/w500/"이미지 URL"

// 출처: https://developers.themoviedb.org/3/getting-started/images
```

## async/awiat 를 활용한 서버통신과 에러 핸들링.


```swift
// ✅ async 함수는 concurrent context 에서만 사용이 가능합니다. 그래서 Task 블럭 안에서 실행.
Task {
    do {
        // ✅ getMovie() 메서드는 async 메서드이고, 에러를 던집니다.
        movies = try await getMovie()
        movieCollectionView.reloadData()
    }
    // ✅ do-catch 문을 활용해서 throw 된 에러를 핸들링.
    catch MovieDownloadError.invalidURLString {
        print("movie error - invalidURLString")
    } catch MovieDownloadError.invalidServerResponse {
        print("movie error - invalidServerResponse")
    }
}

// ✅ 비동기 함수를 구현하기 위해서 async 키워드 사용.
// ✅ 에러를 던지기 때문에 throws 사용.
private func getMovie() async throws -> [Result] {
    // Const 구조체에 상수로써 URL 을 관리.
    guard let url = URL(string: Const.URL.baseURL + Const.Endpoint.popular + Const.Key.apiKey) else {
        throw MovieDownloadError.invalidURLString
    }

    let (data, response) = try await URLSession.shared.data(from: url)
    guard let httpResponse = response as? HTTPURLResponse, httpResponse.statusCode == 200 else {
        throw MovieDownloadError.invalidServerResponse
    }
    let popularMovie = try JSONDecoder().decode(PopularMovie.self, from: data)
        
    return popularMovie.results
}
```

### Error Handling 을 위한 열거형 생성

```swift
import Foundation

enum MovieDownloadError: Error {

    /// 유효하지 않은 URL 형식 오류.
    case invalidURLString

    /// 유효하지 않은 통신 오류.
    case invalidServerResponse
}
```

### Codable 한 Model 구조체 생성

- 해당 Movie API 의 response 를 기반으로 작성.

```swift
import Foundation

// MARK: - Movie

struct PopularMovie: Codable {
    let page: Int
    let results: [Result]
    let totalPages, totalResults: Int

    enum CodingKeys: String, CodingKey {
        case page, results
        case totalPages = "total_pages"
        case totalResults = "total_results"
    }
}

// MARK: - Result

struct Result: Codable {
    let adult: Bool
    let backdropPath: String
    let genreIDS: [Int]
    let id: Int
    let originalLanguage: String
    let originalTitle, overview: String
    let popularity: Double
    let posterPath, releaseDate, title: String
    let video: Bool
    let voteAverage: Double
    let voteCount: Int

    enum CodingKeys: String, CodingKey {
        case adult
        case backdropPath = "backdrop_path"
        case genreIDS = "genre_ids"
        case id
        case originalLanguage = "original_language"
        case originalTitle = "original_title"
        case overview, popularity
        case posterPath = "poster_path"
        case releaseDate = "release_date"
        case title, video
        case voteAverage = "vote_average"
        case voteCount = "vote_count"
    }
}
```

## 결과

<img src="https://user-images.githubusercontent.com/69136340/171683607-602a6fe9-8b5e-4ed5-9dc0-0155d6c27974.mov" width ="220">


**출처:**

[Use async/await with URLSession - WWDC21 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2021/10095/)

[Meet async/await in Swift - WWDC21 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2021/10132/)

[Apple Developer Documentation](https://developer.apple.com/documentation/swift/swift_standard_library/concurrency/updating_an_app_to_use_swift_concurrency)

[Apple Developer Documentation](https://developer.apple.com/documentation/swift/task)

[번역 - Use async / await with URLSession, WWDC 2021](https://velog.io/@okstring/%EB%B2%88%EC%97%AD-Use-async-await-with-URLSession-WWDC-2021)

[[Swift] async / await & concurrency](https://sujinnaljin.medium.com/swift-async-await-concurrency-bd7bcf34e26f)

[[Swift Concurrency] Async/await](https://zeddios.tistory.com/1230)

[[Swift] async / await 등장배경](https://eunjin3786.tistory.com/381)
