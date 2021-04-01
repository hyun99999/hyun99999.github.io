---
title:  "Swift algorithm(1)"
categories:
- swift
- algorithm
date:   2021-03-31 18:17:06 +0900
author_profile: false
---
기본 셋팅과 시작

시작 - 무엇을 어떻게 공부할 건지 정해보자.
> 공부한 내용은 github.io 에 저장하고 알고리즘 풀이는 터미널로 github 에 올리자!

## 프로젝트 생성
Swift 로 알고리즘 문제를 풀 때, 입력을 받아야할 때 `readLine()` 를 사용해야한다.

`Command Line Tool` 로 프로젝트를 생성해야 사용할 수 있다.
> `readLine()` 함수를 통해 콘솔창에 자유롭게 입력을 받을 수 있다.

## readLine 구현
<img src = "https://user-images.githubusercontent.com/69136340/113235010-549dfd80-92dd-11eb-8f2d-406c6697c3db.png" width="600">
> 리턴값이 옵셔널 타입이다. 

**1. 한줄만 입력**
```swift
//강제추출
//알고리즘을 해결 시 nil 을 입력받는 경우가 없어서 강제적 추출을 사용
let inputValues = readLine()!

//Int 형 한 개의 숫자 입력 받기
let inputValues = Int(readLine()!)!

//옵셔널 바인딩
var input = readLine()
if let input = input {
print(input)
}
```

**2. 두줄을 입력**
```swift
let inputValues1 = Int(readLine()!)!
let inputValues2 = Int(readLine()!)!
```

**3. 한줄만 입력, 2개의 수 공백 구분**

readLine 으로 읽은 값 inputValues 는 String 타입이라서 "1 2" 과 같은 형태의 문자열로 저장되어 있다.

공백을 기준으로 입력받은 값을 배열의 요소로 저장하는 방법을 사용한다.

이때 split 과 components 두가지 메서드를 사용한다.
<img src ="https://user-images.githubusercontent.com/69136340/113235610-60d68a80-92de-11eb-8ba6-76c88ac98c5b.png" width="600">

<img src = "https://user-images.githubusercontent.com/69136340/113236551-0b02e200-92e0-11eb-9f2a-02084f869421.png" width ="600">

```swift
//split
//map 해서 string 으로 사용 가능.
let inputValues = readLine()!.split(separator: " ").map { Int($0)! }

let a = inputValues[0]
let b = inputValues[1]

//components
import Foundation

let inputValues = readLine()!.components(separatedBy: " ")
```

**4. N줄만큼 입력**
```swift
var lines = [int]()
for _ in 0..<N { lines.insert(Int(readLine()!)!, at: 0) }
```

### 출처
출처ㅣhttps://babbab2.tistory.com/29

출처ㅣhttps://0urtrees.tistory.com/167
