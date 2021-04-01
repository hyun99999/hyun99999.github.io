---
title:  "Swift alogorithm(2)"
categories:
- swift
- algorithm
date:   2021-04-01 14:24:06 +0900
author_profile: false
---
배열(Array) 다루기

1.빈 배열 만들기
```swift
var empty: [Int] = []
var empty: [Int]()
var empty: Array<Int> = []
```

2.임의의 Data 넣어서 만들기
```swift
var array = Array(1...5) //[1,2,3,4,5]
var array = [1, 2, 3, 4, 5]
```

3.크기와 Data 가 정해진 배열
```swift
var = Arrat(repeating: 0, count: 3) //[0,0,0]
```

4.2차원 배열 만들기
```swift
let matrix = [[Int]]()
let arr: [[Int]] = Array(repeating: Array(repeating: 1, count: 5),
    count: 3)
//1 1 1 1 1
//1 1 1 1 1
//1 1 1 1 1
//arr[i][j] 이런식으로 사용
```

5.배열 거꾸로 출력
```swift
array.reversed()
```

6.배열 정리하기

sort : 배열의 값들을 변경하여 정렬한다.
```swift
array.sort() //default 는 오름차순(1,2,3...)
array.sort(by: >) //내림차순을 원할 때
```
sorted : 정렬된 배열을 반환한다.
```swift
let arr1 = array.sorted()
//내림차순
let arr2 = array.sorted(by: >)
```

7.map, filter, reduce

- map
```swift
let string = ["1", "2", "3"]
let a = string.map {Int($0)!}
```

- filter
```swift
let arr = [1,2,3,4]
let a = arr.filter { $0 % 2 == 0 }
```

- reduce
```swift
let arr = [1, 2, 3, 4]
let a = arr.reduce(0, +)
```

### 출처
출처ㅣhttps://twih1203.medium.com/swift-알고리즘에-필요한-swift-basic-총정리-d86453bbeaa5

출처ㅣhttps://0urtrees.tistory.com/119


