---
title:  "Swift) inout 매개변수"
categories:
- swift
date:   2021-04-02 19:48:00 +0900
author_profile: false
---
inout 매개변수

Swift 에서 함수와 메소드의 매개변수는 기본적으로 상수이다. 

함수의 파라미터를 함수 내부에서 변경하고 함수 종류 후에도 변경한 값이 지속되려면 변수의 주소값을 넘겨 직접 접근할 수 있도록 해주는 **inout 키워드**를 사용하면 가능하다.

또, class 를 파라미터로 받았다면 인스턴스의 주소를 가지고 있기 때문에 클래스 내부의 값은 변경할 수 있다. 파라미터 상수가 참조를 하고 있기 때문이다.
```swift
calss A {}
func a(p1: A) { ... }
```

```swift
var p1 = 1
var p2 = A() 
var p3 = 2
func a(p1: Int, p2: A, p3: inout Int) {
    p1 = 3 //error
    p2.a = 5 //값 변경
    p3 = 5 //값 변경

}
a(p1: p1, p2: p2, p3: &p3)

print(p1) // 1 변경불가
print(p2.a) // 5 변경된 값 출력
print(p3) // 5 변경된 값 출력
```

그래서 파라미터로 받은 배열의 값을 수정하는 알고리즘은 버블, 선택, 삽입 정렬은 inout 을 사용한다.

### 출처
출처ㅣhttps://nsios.tistory.com/57 [NamS의 iOS일기]



