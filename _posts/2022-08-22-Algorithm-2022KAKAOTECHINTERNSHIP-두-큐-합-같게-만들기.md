---
title:  "Algorithm) 2022 KAKAO TECH INTERNSHIP - 두 큐 합 같게 만들기"
categories:
- iOS

date:   2022-08-22  20:52:00 +0900
author_profile: false
---
2022 KAKAO TECH INTERNSHIP 의 두 번째 문제 "두 큐 합 같게 만들기”

[코딩테스트 연습 - 두 큐 합 같게 만들기](https://school.programmers.co.kr/learn/courses/30/lessons/118667)

최근에 진행된 코딩테스트 문제가 공개가 되었다. 그래서인지 swift 풀이가 많이 없어서 내가 해결한 방법을 올려본다.

## 실패 - 정확성(56.7/100)

```swift
import Foundation

// 아이디어 : 
// L > R이라면, queue1의 원소를 queue2로 넘겨줍니다.
// L < R이라면, queue2의 원소를 queue1로 넘겨줍니다.
// Swift 에서는 queue 가 없기 때문에 배열을 사용하여 진행.
// 정확성(56.7/100)
func solution(_ queue1:[Int], _ queue2:[Int]) -> Int {
    var answer: Int = 0
    var queue1 = queue1
    var queue2 = queue2
    
    let queue1Sum: Int = queue1.reduce(0, +)
    let queue2Sum: Int = queue2.reduce(0, +)
    let goal: Int = (queue1Sum + queue2Sum) / 2
    
    if (queue1Sum + queue2Sum) % 2 != 0 {
        return -1
    } else if goal < queue1.max()! || goal < queue2.max()! {
        // 목표값보다 큐의 최대값이 크면 달성할 수 없음.
        return -1
    }
    
    while goal != queue1.reduce(0, +) {
        if queue1.reduce(0, +) > queue2.reduce(0, +) {
            let first = queue1.removeFirst()
            queue2.append(first)
        } else {
            let first = queue2.removeFirst()
            queue1.append(first)
        }
        answer += 1
    }
    
    return answer
}

```

## 실패 - 정확성(83.3 / 100)

```swift
import Foundation

// 어렴풋이 reduce 를 통한 합을 구하는 방법에 대한 개선이 필요하다고 느꼈고, 최소한으로 횟수를 가져갔다.
// 하지만, 시간 초과에 부딪혔다.
// 정확성(83.3 / 100)
func solution(_ queue1:[Int], _ queue2:[Int]) -> Int {
    var answer: Int = 0
    var queue1 = queue1
    var queue2 = queue2
    
    var queue1Sum: Int = queue1.reduce(0, +)
    var queue2Sum: Int = queue2.reduce(0, +)
    let goal: Int = (queue1Sum + queue2Sum) / 2
    
    if (queue1Sum + queue2Sum) % 2 != 0 {
        return -1
    } else if goal < queue1.max()! || goal < queue2.max()! {
        // 목표값보다 큐의 최대값이 크면 달성할 수 없음.
        return -1
    }
    
    while goal != queue1Sum {
        if queue1Sum > queue2Sum {
            let first = queue1.removeFirst()
            queue2.append(first)
            
            // 아이디어: reduce 과정을 통한 시간을 줄이기 위한 과정.
            queue1Sum -= first
            queue2Sum += first
        } else {
            let first = queue2.removeFirst()
            queue1.append(first)
            
            queue2Sum -= first
            queue1Sum += first
        }
        answer += 1
    }
    
    return answer
}
```

## 성공 (정확성, 효율성)

```swift
import Foundation

// 아이디어 : 투포인터
// 큐에서 다른 큐로 insert 할 때 뒤에 붙는 것을 고려.
// queue1 과 queue2 를 이어서 한 개의 배열에서 두 개의 포인터를 사용.
func solution(_ queue1:[Int], _ queue2:[Int]) -> Int {
    let array: [Int] = queue1 + queue2
    // queue1 의 좌우 포인터.
    var left: Int = 0
    var right: Int = queue1.count
    var answer: Int = 0
     
    var queue1Sum: Int = queue1.reduce(0, +)
    let queue2Sum: Int = queue2.reduce(0, +)
    let goal: Int = (queue1Sum + queue2Sum) / 2
    
    if (queue1Sum + queue2Sum) % 2 != 0 {
        return -1
    }
    
    if goal < queue1.max()! || goal < queue2.max()! {
        return -1
    }

    while right < array.count && left <= right {
        if queue1Sum < goal {
            // queue1 이 목표값보다 작으면 queue2 에서 이동.
            queue1Sum += array[right]
            right += 1
        } else if queue1Sum > goal {
            // queue1 이 목표값보다 크면 queueu2 로 이동.
            queue1Sum -= array[left]
            left += 1
        } else {
            // goal 과 같은 경우.
            break
        }
        answer += 1
    }
    
    // 이동이 마친 후에도 goal 에 도달하지 않는 경우.
    if queue1Sum != goal {
        return -1
    }
    
    return answer
}
```

풀이 코드에서 아래의 코드가 있는데 큐 사이에 이동 없이 이루어질 수 없는 조건에 대해서 구현해보았다. 이 코드를 통해 풀이 시간을 단축할 수 있겠다는 예상이 있었다.

```swift
    if goal < queue1.max()! || goal < queue2.max()! {
        return -1
    }
```

### 결과

주석처리한 코드보다 오래 걸렸다. max() 메소드를 다루는 것이 큐 사이 이동없이 초기에 불가능한 문제에 대해 체크하는 것보다 오버헤드가 컸던 것 같다.

### 참고

[2022 테크 여름인턴십 코딩테스트 해설](https://tech.kakao.com/2022/07/13/2022-coding-test-summer-internship/)
