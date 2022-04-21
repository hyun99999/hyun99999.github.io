---
title:  "SwiftUI) ForEach 에서 index 도 사용하기"
categories:
- iOS

date:   2022-04-21  23:50:00 +0900
author_profile: false
---
### 내용

- ForEach 로 리스트를 구성하던 중 랭킹이 필요했다.
- 랭킹은 해당 topShows(array 데이터) 의 index 로 다루면 되기 때문에 데이터에 포함시키지 않았다.
- 그런데 다음과 같은 경고가 등장했다. 무엇일까?

🔥 
- Non-constant range: argument must be an interger literal
- Non-constant range: not an integer range


- 둘다 일정하지 않는 범위(상수 범위가 아니다.)라면서 경고를 던집니다.
    - topShows 현재 이니셜라이저를 통해서 초기화 받는 var 로 선언되어 있습니다.

<img src="https://user-images.githubusercontent.com/69136340/164482428-081d5a7d-653e-4ecf-b9c0-67c1f292f7e9.png" width ="800">

- `.indices` : 오름차순의 컬렉션을 subscribe 하는데 유효한 인덱스

<img src="https://user-images.githubusercontent.com/69136340/164482490-be5ac360-34e4-469a-b46c-cecc8d83eb0c.png" width ="800">

### 해결?

- id 파라미터를 등록해주었더니 경고가 사라졌습니다.

```swift
struct TopEpisode: Identifiable {
    let id = UUID()
    let thumbnail: String
    let title: String
    let date: String
    let time: String
}

// ...

// Int 값을 식별자로 사용하게 된다.
ForEach(0..<topShows.count, id: \.self) { index in
    TopShowsItem(topShow: topShows[index], index: index + 1)
}

// Int 타입의 해당 인덱스 값을 식별자로 사용하게 된다.
ForEach(topShows.indices, id: \.self) { index in
    TopShowsItem(topShow: topShows[index], index: index + 1)
}
```

그렇다면.. 어째서..?

### Why?

<img width="700" alt="3" src="https://user-images.githubusercontent.com/69136340/164482576-9914027b-dd53-470d-90b3-3158fa6648e8.png">

`id` 파라미터가 있는 이니셜라이저를 사용하니 `data` 파라미터는 `Range<Int>` 가 아닌 `Data` 로 여기기 때문입니다.

<img width="750" alt="4" src="https://user-images.githubusercontent.com/69136340/164482582-6857190e-a834-4a8f-996c-8bae2d33548d.png">

### 위의 방법보다 더 나은 방법은!

위의 방법의 단점은 해당 배열의 중간을 수정하게 되면 항목은 업데이트되지만 인덱스 자체는 변경되지 않아서 이상하게 보인다고 합니다.

*출처:* 

[ForEach With Index in SwiftUI](https://jierong.dev/2020/05/31/foreach-with-index-in-swiftui.html)

### enumerated() 를 사용해서  index 와 item 모두 사용할 수는 없을까요?

<img width="450" alt="5" src="https://user-images.githubusercontent.com/69136340/164482977-e15825d9-9928-4eb7-8c68-ebdba2c0a102.png">


`EnumeratedSequence` 는 위의 에러처럼 `RandomAccessCollection` 과 `Hashable` 을 채택하지 않기 때문에 저 위치에는 들어갈 수 없습니다.

그래서 1️⃣ `Array` 를 통해서 래핑해서 두가지 모두를 채택할 수 있도록 했습니다. 

그래서 2️⃣ `zip` 을 사용할 겁니다.

🔥 zip 이란? 두개의 시퀀스로 이루어진 시퀀스 쌍을 만듭니다.


그리고 3️⃣ `id` 파라미터를 사용해서 위에서 언급한것처럼 `Range<Int>` 가 아닌 `Data` 타입으로 유추하도록 할거에요.

그러면 다음과 같이 `Array` 로 래핑되어있던 `zip` 의 `element` 들이 (index, item) 로 전달되게 됩니다.

```swift
// topEpisodes.indices
ForEach(Array(zip(topEpisodes.indices, topEpisodes)), id: \.0) { index, item in
    TopEpisodesItem(topEpisode: item, index: index + 1)
}

//topEpisodes.count
ForEach(Array(zip(0..<topEpisodes.count, topEpisodes)), id: \.0) { index, item in
    TopEpisodesItem(topEpisode: item, index: index + 1)
}
```

**\.0 의 타입은** `Range<Array<TopEpisode>.Index>.Element` 

- `zip(topEpisodes.indeices, topEpisodes)` 의 첫번째 인덱스 0 의 자료형은 `Range<Array<TopEpisode>.Index>.Element`

<img src="https://user-images.githubusercontent.com/69136340/164483026-9b9d33c1-1784-4057-840c-4f44a5bd39a6.png" width ="500">

**\.0 의 타입은 `Range<Int>.Element`**

- `zip(0..<topEpisodes.count, topEpisodes)` 의 첫번째 인덱스 0 의 자료형은 `Range<Int>.Element`

<img src="https://user-images.githubusercontent.com/69136340/164483035-5e115afa-de31-4fea-a0ac-6e855b58e7e0.png" width = "300">

### `\.0` ? `\.self, \.1` 을 사용하면 안되나요?

**\.self**

- (Int, TopEpisode) 튜플은 `Hashable` 을 채택하지 않기 때문에 사용할 수 없다고 한다.

<img src="https://user-images.githubusercontent.com/69136340/164483480-3f2b9dd0-629d-4c5f-ab46-f067af524384.png" width ="300"


- ID 는 `Hashable` 해야한다고 정의되어 있다.

<img src="https://user-images.githubusercontent.com/69136340/164483581-2286dc68-0a28-42dc-8e19-1dec8ac5401a.png" width ="500">


**\.1 은 `TopEpisode` **

- `zip(topEpisodes.indeices, topEpisodes)` 의 두번째 인덱스 1의 자료형은 `TopEpisode`

<img src="https://user-images.githubusercontent.com/69136340/164483607-16f3b36e-f94d-400e-8525-69d4ba392666.png" width = "300">

<img src="https://user-images.githubusercontent.com/69136340/164483962-79c9b364-e06f-46f8-bdc7-86c75d3ece66.png" width ="300">

- 식별자로 사용될 `TopEpisode` 타입이 `Hashable` 하지 않다고 한다.

<img src="https://user-images.githubusercontent.com/69136340/164484137-b3b8482a-ab47-4c35-bbce-f022a5c60dde.png" width ="500">

### 그런데요~ `zip` 대신 `.eumerated()` 를 `Array` 로 래핑해주면 되지 않나요?

맞아요! 그런데 우리가 아는 .eumerated() 에는 오해가 있답니다.

enumerated 는 우리가 index 와 item 을 반환한다고 생각하지만 아래의 글을 보게되면 그렇지 않은 것을 알 수 있어요.

[[번역]여러분은 아마 enumerated 하고 싶지 않을 것이다](https://blog.canapio.com/109)

```swift
let array = ["a", "b", "c", "d", "e"]
let arraySlice = array[2..<5]

print(arraySlice[2]) // => "c"

for (index, item) in arraySlice.enumerated() {
    print("index: \(index), item: \(item)")
}
// index: 0, item: c
// index: 1, item: d
// index: 2, item: e

print(arraySlice[0]) // fatalError
```

글에서 나왔듯이 index 가 아닌 오프셋의 개념으로 봐야할거 같아요. 그래서 이러한 경우에는 의도하는대로 되지 않을거에요! 주의해서 사용해야겠네요.

출처:

[ForEach With Index in SwiftUI](https://jierong.dev/2020/05/31/foreach-with-index-in-swiftui.html)

[How for `SwiftUI.ForEach.init(_ data: Range , @ViewBuilder content: @escaping (Int) -> Content)` compiler is able to warn if `Range` is not constant?](https://forums.swift.org/t/how-for-swiftui-foreach-init-data-range-int-viewbuilder-content-escaping-int-content-compiler-is-able-to-warn-if-range-is-not-constant/55233/4)
