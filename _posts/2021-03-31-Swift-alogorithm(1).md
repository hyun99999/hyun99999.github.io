---
title:  "Swift algorithm(1)"
categories:
- swift
- algorithm
date:   2021-03-31 18:17:06 +0900
author_profile: false
---

# Swift algorithm(1)

시작 - 무엇을 어떻게 공부할 건지 정해보자.


## 프로젝트 생성
Swift 로 알고리즘 문제를 풀 때, 입력을 받아야할 때 `readLine()` 를 사용해야한다.
`Command Line Tool` 로 프로젝트를 생성해야 사용할 수 있다.
> `readLine()` 함수를 통해 콘솔창에 자유롭게 입력을 받을 수 있다.

## readLine 구현
알고리즘을 해결 시 nil 을 입력받는 경우가 없어서 강제적 추출을 사용

**1. 한줄만 입력**
```swift
let inputValues = Int(readLine()!)!
```
**2. 한줄만 입력, 2개의 수 공백 구분**
```swift
let inputValues = readLine()!.split(separator: " ").map { Int($0)! }

let a = inputValues[0]
let b = inputValues[1]
```

**3. 두줄을 입력**
```swift
let inputValues = Int(readLine()!)!
let inputValues = Int(readLine()!)!
```
**4. N줄만큼 입력**
```swift
var lines = [int]()
for _ in 0..<N { lines.insert(Int(readLine()!)!, at: 0) }
```

## component
split 과 component 는 입력받은 문자열을 배열로 리턴해주는 공통점이 있지만, 

### 출처
출처ㅣ

출처ㅣ

출처ㅣ
