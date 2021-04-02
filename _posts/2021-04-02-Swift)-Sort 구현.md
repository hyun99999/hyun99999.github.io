---
title:  "Swift) Sort 구현"
categories:
- swift
- algorithm
date:   2021-04-02 16:35:00 +0900
author_profile: false
---
Bubble / Selection / Insertion Sort

## Bubble Sort

```swift
func bubbleSort(_ array: inout [Int]) {
    for index1 in 0..<(array.count - 1) {
        var isSwap = false
        for index2 in 0..<((array.count - index1) - 1) {
            if array[index2] > array[index2 + 1] {
               array.swapAt(index2, (index2 + 1))
                isSwap = true
            }
        }
        if isSwap == false { return }
    }
}
```

## Selection Sort

```swift
func selectionSort(_ array: inout [Int]) {
    for stand in 0..<(array.count - 1) {
        var minIndex = stand
        for index in (stand + 1)..<array.count {
            if array[index] < array[minIndex] {
                minIndex = index
            }
        }
        //탐색 중 가장 작은 수를 찾는다.
        array.swapAt(stand, minIndex)
    }
}
```

## Insertion Sort

```swift
func insertionSort(_ array: inout [Int]) {
    for stand in 1..<array.count {
        for index in stride(from: stand, to: 0, by: -1) {
            if array[index] < array[index - 1] {
                array.swapAt(index, index - 1)
            } else {
                break
            }
        }
    }
}
```

### 출처
출처ㅣhttps://babbab2.tistory.com/96?category=908012

출처ㅣhttps://babbab2.tistory.com/97?category=908012

출처ㅣhttps://babbab2.tistory.com/98?category=908012



