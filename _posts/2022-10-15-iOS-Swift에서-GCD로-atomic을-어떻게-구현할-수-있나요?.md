---
title:  "iOS) Swift 에서 GCD 로 atomic 을 어떻게 구현할 수 있나요?"
categories:
- iOS

date:   2022-10-10  01:11:00 +0900
author_profile: false
---
### 1️⃣ 하나의 serial queue 가 테스크를 담당하도록 함
- concurrent queue 를 사용하거나 여러 개의 serial queue 를 사용하면 여러 스레드가 동시에 접근하는 경우 발생. 이를 막기 위해 근본적으로 하나의 serial queue 가 테스크를 담당. 
- 이는 동시성을 포기하게 됨.

### 2️⃣ Dispatch Barrier
- Dispatch Barrier 사용하여 쓰기 작업은 thread-safe 하게, 읽기 작업은 동시에 작업할 수 있도록 만들 수 있다.(읽기 작업은)
- custom concurrent queue 에서만 사용할 수 있고, 이 queue 의 특정 task 를 수행할 때 다른 스레드가 일을 하지 않도록 막는 방법이다. 막는 스레드는 custom concurrent queue 가 담당하는 스레드들이다.
<img src="https://velog.velcdn.com/images%2Fyohanblessyou%2Fpost%2Fb6705c9f-8ee1-43cb-9d09-971ace1b2a4a%2Fimage.png" width ="500">
- 다른 스레드의 기존작업이 마무리되는 것을 기다리고 -> barrier task 수행 -> 완료되면 다른 스레드의 작업 실행

```swift
DispatchQueue(label: "barrierTestQueue", attributes: .concurrent).async(flags: .barrier) { }
```

### 3️⃣ Dispatch Semaphore
Semaphore 의 value 를 1로 설정하게 되면 barrier 처럼 사용할 수 있다.

- Semaphore 는 공유 자원에 동시 접근 가능한 작업 수를 제한할 때 사용됩니다.(Semaphore 객체 생성 시점에 value 값을 접근 가능한 작업 수만큼으로 초기화한다.)

```swift
// 💫 value 를 1로 설정하여 한번에 하나의 작업만 접근할 수 있도록 구현할 수 있다.
let semaphore = DispatchSemaphore(value: 1)

// 💫 value 가 0이면 기다리고, 0이 아니면 차감하고 다음 코드를 진행.
semaphore.wait()

DispatchQueue.global().async {
    print("task")
    
    // 💫 value 를 증감하고 다음 코드를 진행.
    semphore.signal()
}
```
