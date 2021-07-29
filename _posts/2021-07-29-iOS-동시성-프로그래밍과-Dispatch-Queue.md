---
title:  "iOS) 동시성 프로그래밍과 Dispatch Queue"
categories:
- iOS

date:   2021-07-29  11:34:00 +0900
author_profile: false
---
# iOS 환경에서의 동시성 프로그래밍 지원 종류

-   GCD (Grand Central Dispatch) : 멀티 코어와 멀티 프로세싱 환경에서 최적화된 프로그래밍을 할 수 있도록 애플이 개발한 기술입니다.
-   Operation Queue : 비동기적으로 실행되어야 하는 작업을 객체 지향적인 방법으로 사용합니다.

# 동시성 프로그래밍과 병렬성 프로그래밍

-   동시성 프로그래밍
-   -   동싱 실행되는 것처럼 보이는 방식. 싱그코어에서 멀티스레드를 동작시키 위한 방식. 멀티태스킹을 위해서 여러개의 스레드가 번갈아가면서 실행.
-   병렬성 프로그래밍
-   -   물리적으로 정확히 동시에 실행되는 것을 말한다. 멀티 코어에서 멀티 스레드를 동작시키는 방식으로 **데이터 병렬성(Data Parallelism)과 작업 병렬성(Task Parallelism)** 으로 구분됩니다.
    -   데이터 병렬성 : 전체 데이터를 서브데이터로 나누고 병렬 처리해서 빠륵 수행.
    -   작업 병렬성 : 서로 다른 작업을 병렬 처리하는 것.

# DispatchQueue

-   장점 : 일반 스레드 코드보다 쉽고 효율적으로 코드를 작성할 수 있다. 주로 서버에서 데이터를 내려받거나 이미지, 동영상 등 멀티미디어 처리와 같은 별도의 스레드에서 처리가 필요한 작업 후 결과를 전달.
-   DispatchQueue 를 생성 시 기본은 Serial 이다. Concurrent 유형으로 바꾸려면 별도의 명시 필요.

### 대기열(queue) 유형

-   Serial
    -   대기열에 등록한 순서대로 작업을 실행. 직렬이라서 하나의 작업을 실행하고 끝날때까지 대기열에 있는 다른 작업은 대기.
-   Concurrent
    -   실행중인 작업이 끝나기를 기다리지 않고 대기열에 있는 작업을 동시에 별도의 스레드를 사용하여 실행. 즉, 병렬 처리.

# DispatchQueue VS OperationQueue

-   OperationQueue : 기본적으로 FIFO 지만 조건에 따라 뒤의 작업 먼저 실행가능. 비동기적 실행작업을 객체 지향적인 방법으로 사용하는데 적합.
-   -   Concurrent Operation객체를 구현할 필요없이 Operation을 Operation Queue에 제출하기만하면 Concurrent Operation 객체를 만들어 줍니다.
    -   **동시에 실행할 수 있는 연산의 최대수 지정가능.**(maxConcurrentOperationCount) Key Value Observing(KVO)를 사용해 작업 진행상황 모니터링 가능. Pause, Cancle, Resume 가능.
-   DispatchQueue : 작업이 복잡하지않고 간단하게 처리하거나 특정유형의 이벤트 비동기처리 시 적합(ex: 타이머, 프로세스 )
-   Operation을 하기에는 단순한 코드들 구현할 때 사용. DispatchQueue() 처럼 객체로 만드는 큐는 명시해주어야 Concurrent로 생성 가능.

### GCD(Grand Central Dispatch)

-   GCD 는 애플에서 만든 멀티 코어환경에서의 application 을 개발할때 도움을주는 라이브러리.Task 라는 작업을 대기열을 주어 순서대로 실행 혹은 동시에 실행시켜주는 등 많은 작업을 한다.
-   GCD 를 쓰면 쓰레드를 다루는데 개발자가 직접 관리해주어야할 일이 많이 준다. 모든 Task 를 FIFO 로 관리.

### Main Queue 와 Global Queue

-   엡 실행시 시스템에서 기본적으로 2개의 Queue 만들어준다.
    
-   Main Queue
    
-   메인 쓰레드에서 사용되는 Serial Queue.  
    UI관련 task는 이곳에서 실행. 그렇지않으면 에러발생.
    
-   Global Queue
    
    ```
    // Main Queue
    let mainQueue = DispatchQueue.main
    //Global Queue
    let globalQueue = DispatchQueue.global(qos: .background)
    ```
    
-   -   편의상 사용할 수 있게 만든 Concurrent Queue.
    -   Global Queue 는 qos 파라미터를 지원.
    -   병렬적으로 동시에 처리하기 때문에 완료의 순서는 정할 수 없지만 시작은 할 수 있다.

### QoS. queue 의 우선순위(Quality of Service)

-   .userInteractive  
    : 제일 높은 우선순위. 유저에게 직접적을 반응을 보여줘야 할 때 사용. UI 업데이트나 애니메이션작업에 사용.
-   .userInitiated  
    : 유저의 UI 입력을 시작되며, 거의 바로 결과를 보여줘야 할떄 사용. 하지만 작업이 비동기적으로 끝 마칠 수 있다. 로컬에 저장된 문서를 불러올 때와 같이.
-   .utility  
    : 네트워크나 긴 실행시간을 가졌을때 사용되는 우선순위.
-   .background  
    : 직접적을 보여지지 않아도 될때 사용. 속도보다는 에너지 효율에 맞춰 작업 진행.
-   .default  
    : default 는 userInitiated 와 utility 중간 정도. QoS정보가 할당되지 않은 작업은 Default로 처리되며 GCD global queue는 이 level(default)에서 실행됩니다.
-   .unspecified  
    : 이는 Qos 에 정보가 없음을 의미.

### Sync / Async

-   sync(동기) 와 async(비동기) 라는 메소드를 가지고 있다.
-   Serial 과 Concurrent 와는 별개다. (직렬/병렬의 문제.)
-   Serial, Concurrent 는 스레드의 수와 관련이 있다.
-   -   Sync : sync의 경우 하나의 작업이 Queue에서 빠져나갈 때까지 기다리기 때문에, Serial 이냐 Concurrent냐의 차이는 없습니다. sync로 모든 작업을 Queue에 넣을 경우 그 순서가 보장됩니다.
    -   Async : 큐에 작업을 추가하고, 이 작업의 완료 여부와 관계없이 다음 명령을 실행합니다. 두 개의 Serial Queue 가 존재해도 직렬 큐이기 때문에 순서는 보장된다.
    -   Concurrent Async 방식은 병렬큐에 비동기로 동작되기 때문에 작업 처리 순서가 보장되지 않는다.
-   sync와는 다르게 async로 추가된 작업들을 기다리지 않고 바로 다음 코드가 실행되는 것을 알 수 있습니다. 여기서 정확하게 짚고 넘어가야할 부분이 있습니다. sync와 async 메소드는 DispatchQueue에 작업을 추가하는 메소드이지 작업을 실행하는 메소드가 아닙니다. 실제로 작업을 실행하는 것은 Dispatch Queue입니다.
-   sync와 async, 두 메소드가 Dispatch Queue의 동작 방식에는 영향을 주지 않습니다. Serial 큐에 작업을 추가할 때는 어떤 메소드를 사용하던지 항상 추가된 순서대로 실행됩니다. 반면에 Concurrent 큐에 추가할 때 sync 메소드를 사용하면 작업이 추가된 순서대로 하나씩 실행되는 것 같지만 Concurrent 큐가 작업을 동시에 실행할 수 있다는 점은 달라지지 않습니다. 때문에 다른 곳에서 Concurrent 큐에 작업을 추가하면 현재 실행중인 작업과 동시에 실행되게 됩니다.

---

> -   출처: [https://medium.com/nbt-tech/dispatchqueue는-어떻게-사용할까-44f22f08d62](https://medium.com/nbt-tech/dispatchqueue%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%82%AC%EC%9A%A9%ED%95%A0%EA%B9%8C-44f22f08d62)
> -   출처: [https://magi82.github.io/gcd-01/](https://magi82.github.io/gcd-01/)
> -   출처: [https://nsios.tistory.com/31](https://nsios.tistory.com/31) \[NamS의 iOS일기\]
> -   출처: [\[iOS\] GCD 활용하기 1편 (DispatchQueue)](https://onelife2live.tistory.com/4)
