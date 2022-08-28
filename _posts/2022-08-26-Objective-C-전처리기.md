---
title:  "iOS) COW(Copy-on-Write)"
categories:
- iOS

date:   2022-08-24  22:38:00 +0900
author_profile: false
---
[OptimizationTips](https://github.com/apple/swift/blob/main/docs/OptimizationTips.rst#advice-use-inplace-mutation-instead-of-object-reassignment) 문서에서 Value Type(값 타입)은 COW(Copy-on-Write) 최적화를 사용한다고 합니다.

COW 에 대해서 알아보기 전에 타입에 따라서 어떠한 식으로 복사하는지 알아봅시다.

## ✅ Swift 에서 값 타입과 참조 타입의 복사 방식은 다릅니다.

- `값 타입` : 깊은 복사(deep copy). 데이터 자체를 복사하는 방법. 독립적인 메모리를 차지하기 때문에 복사한 인스턴스의 데이터를 바꾸더라도 원본에 영향을 주지 않습니다.
- `참조 타입` : 얕은 복사(shallow copy). 최소한의 복사만 진행. 복사를 할 때 주소값을 공유한다. 원본과 복사본이 같은 주소값을 공유하므로 한 쪽의 데이터를 바꾸면 다른쪽에 영향을 줍니다.

값 타입은 수정하지 않아도 매번 데이터를 복사할 때 새로운 메모리를 차지해야한다는 것은 꽤나 불필요 해보이기도 하네요.

## ✅ Copy on Write

원복 혹은 복사본이 수정되기 전까지 복사하지 않고, 같은 주소값을 공유한다. 수정이 되면 그때 깊은 복사가 된다.

앞서 언급했듯이 Swift 에서는 value type 은 COW 를 사용한다고 한다. 이로인해 깊은 복사를 수행하는 대신 컨테이너를 유지하여 불필요한 복사본을 제거할 수 있다고 한다.

> All standard library containers in Swift are value types that use COW (copy-on-write) to perform copies instead of explicit copies. In many cases this allows the compiler to elide unnecessary copies by retaining the container instead of performing a deep copy.
> 

이처럼 COW 는 얕은 복사와 깊은 복사의 장점을 모두 가지는 기능입니다.

**좀 더 다른 역할에서 이점을 찾아볼게요.** 

함수의 매개변수로 전달되는 값은 스택에 복사되게 됩니다. 이때 기본적으로 전달된 매개변수는 상수이기 때문에 깊은 복사를 할 필요가 없습니다. 그래서 얕은 복사가 진행되어 메모리를 비효율적으로 사용하지 않아도 됩니다.

## ✅ 진짜로 COW 가 작동할까?

```swift
func address(of object: UnsafeRawPointer) -> String{
    let address = Int(bitPattern: object)
    return String(format: "%p", address)
}

var a = [1, 2, 3]
var b = a
address(of: &a) // 0x1007443b0
address(of: &b) // 0x1007443b0

b.append(4)
address(of: &a) // 0x1007443b0
address(of: &b) // 0x100744620
```

위의 코드를 살펴보게 되면 같은 메모리 주소값을 가지다가 b 변수가 수정되자 깊은 복사를 통해 다른 메모리 값을 가지는 것을 확인할 수 있습니다.

추가적으로, 같은 주소값을 가리키던 상황에서 수정이 되면 값을 깊은 복사하기 때문에 첫 수정에서 좀 더 작업 시간이 걸린다고 합니다.

### 출처

[[Swift] 깊은 복사와 얕은 복사(feat. NSCopying)](https://jeong9216.tistory.com/527)

[[Swift] Value Type의 메모리 주소](https://sujinnaljin.medium.com/swift-value-type%EC%9D%98-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EC%A3%BC%EC%86%8C-7d4c3de73b11)

[Swift) COW (Copy-on-Write)](https://babbab2.tistory.com/18)
