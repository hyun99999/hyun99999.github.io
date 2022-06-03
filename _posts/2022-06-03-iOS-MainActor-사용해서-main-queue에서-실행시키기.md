---
title:  "iOS) MainActor 사용해서 main queue 에서 실행시키기"
categories:
- iOS

date:   2022-06-03  22:02:00 +0900
author_profile: false
---
### 내용

- MainActor 를 사용해서 main thread 에서의 동작을 보장해보자.

`MainActor` 는 Concurrency 의 Actors API collection 중 하나입니다.

[Apple Developer Documentation](https://developer.apple.com/documentation/swift/swift_standard_library/concurrency)

### MainActor

[Apple Developer Documentation](https://developer.apple.com/documentation/swift/mainactor)

A singleton actor whose executor is equivalent to the main dispatch queue.

즉, mian thread 에서의 동작을 보장하는 Actor 입니다.`MainActor` 를 사용하면 `DispatchQueue.main` 을 언제 사용할지 고민하지 않아도 됩니다!

```swift
Task {
    do {
        movies = try await getMovie()
        await MainActor.run {
            movieCollectionView.reloadData()
        }
    } catch {
        // error handling.
    }
}
```

DispatchQueue.main 과 동일하게 다음 run loop 를 기다리지 않고 비동기적으로 처리합니다.

**출처:** 

[Apple Developer Documentation](https://developer.apple.com/documentation/swift/swift_standard_library/concurrency)

[Protect mutable state with Swift actors](https://velog.io/@unknown_horse/Protect-mutable-state-with-Swift-actors)

[How to use @MainActor to run code on the main queue](https://www.hackingwithswift.com/quick-start/concurrency/how-to-use-mainactor-to-run-code-on-the-main-queue)
