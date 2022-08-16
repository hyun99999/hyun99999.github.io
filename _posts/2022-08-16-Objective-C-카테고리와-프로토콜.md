---
title:  "Objective-C) 카테고리와 프로토콜"
categories:
- iOS

date:   2022-08-16  23:13:00 +0900
author_profile: false
---
 **_본 포스팅은 ‘프로그래밍 오브젝티브-C 2.0’ 을 읽으며 실습한 코드와 내용, 추가적으로 궁금한 내용을 정리한 글입니다._**

### 내용

- **카테고리**를 사용하여 모듈 방식으로 클래스에 메서드를 추가하는 방법
- 익명 카테고리를 통해 클래스 확장(Extension) 추가하는 방법
- 메서드의 표준화된 목록 만드는 방법

# 카테고리

**카테고리는 클래스 정의를 그룹짓거나, 연관된 메서드를 카테고리로 쉽게 모듈러화 할 수 있게 해준다. 또한, 원본 소스 코드에 접근하거나 서브클래스를 생성하지 않고도 기존의 클래스 정의를 확장하는 방법도 제공한다.**

그렇기 때문에 클래스 정의에 새 메소드를 추가하고 싶을 때나 프로젝트에서 서브클래스를 작성하고 새 메소드를 구현할 수 있지만 더 쉽고 강력한 방법이 바로 카테고리이다.

Fraction 클래스에 사칙 연산을 처리하는 카테고리를 추가하는 방법을 알아보자.

```swift
// Fraction.h

#import <Foundation/Foundation.h>

// MARK: -  Fraction Class

@interface Fraction : NSObject

@property int numerator, denominator;

- (void)print;
- (double)convertToNum;
- (void)setTo:(int)n over:(int)d;
- (void)reduce;

@end
```

## 카테고리 파일 생성

1) Object-C File 을 선택한 후, 아래와 같이 **File Type** 에서 Category 를 선택해주면 된다.

<img width="500" alt="스크린샷 2022-08-11 오후 7 55 36" src="https://user-images.githubusercontent.com/69136340/184900199-7428f932-4720-4874-aa10-468107d2d5db.png">

2) **Class** 에서는 확장할 클래스인 Fraction 을 입력한다. **File** 에서는 카테고리 이름을 입력한다.

<img width="500" alt="스크린샷 2022-08-11 오후 8 04 45" src="https://user-images.githubusercontent.com/69136340/184900678-a26b04f2-7ba0-4dd3-9239-e82c24370a5a.png">


3) 이렇게 자동으로 `+` 가 붙어서 필요한 파일을 만들어준다.(Xcode 5.0에서부터 카테고리 파일을 생성하면 자동적으로 `+` 를 포함한 파일명이 생성된다.)

<img width="350" alt="스크린샷 2022-08-11 오후 8 06 53" src="https://user-images.githubusercontent.com/69136340/184900689-1daa69be-487f-4973-a053-77bea2f4dff1.png">

```swift
// Fraction+MathOps.h

#import "Fraction.h"

NS_ASSUME_NONNULL_BEGIN

@interface Fraction (MathOps)

@end
NS_ASSUME_NONNULL_END
```

```swift
// Fraction+MathOps.m

#import "Fraction+MathOps.h"

@implementation Fraction (MathOps)

@end
```

혹은, Fraction+MathOps.h 파일을 생성해도 된다.

<img width="350" alt="스크린샷 2022-08-11 오후 7 29 24" src="https://user-images.githubusercontent.com/69136340/184901031-7a980282-e0ea-419f-ab7b-9af57bd147b9.png">

```swift
@interface 클래스 (카테고리)
// 카테고리를 클래스에 추가하여 확장.
```

**이미 컴파일러는 기존의 헤더파일을 통해서 클래스에 대해 알고 있기 때문에 부모 클래스를 언급하지 않는다**

```swift
// Fraction+MathOps.h

#import "Fraction.h"

// 이미 존재하는 Fraction interface 를 확장하는 것이기 때문에 (Fraction.h 헤더파일에 카테고리를 추가하지 않는 한) 원래 인터페이스를 추가해야 컴파일러가 Fraction 클래스를 이해할 수 있다.
// ✅ Fraction 클래스의 새 카테고리로 MathOps 를 정의.
@interface Fraction (MathOps)

- (Fraction *)add:(Fraction *)f;
- (Fraction *)mul:(Fraction *)f;
- (Fraction *)sub:(Fraction *)f;
- (Fraction *)div:(Fraction *)f;

@end
```

***이를 통해 Fraction.h 인터페이스 부분에 있는 메소드와 MathOps 카테고리의 메소드를 모두 단일한 구현 부분에서 정의할 수 있게 된다.***

***혹은, 카테고리의 메소드를 별도로 정의해도 된다. 이런 경우, 메소드가 속한 카테고리가 어디인지 언급해 주어야 한다.***

(우리는 카테고리의 인터페이스와 구현 부분을 별개로 작성할 것이다.)

<img width="400" alt="스크린샷 2022-08-11 오후 7 33 59" src="https://user-images.githubusercontent.com/69136340/184901075-855a82c7-13bb-4c21-a710-fe39fdbf609a.png">

```swift
// ✅ 마찬가지로 클래스 이름 뒤에 카테고리 이름을 괄호로 감싼다.
@implementation Fraction (MathOps)

  // ...

@end
```

카테고리 MathOps 에서 사칙연산 메소드를 구현해보겠다.

```swift
// Fraction+MathOps.m

#import "Fraction+MathOps.h"

@implementation Fraction (MathOps)

- (Fraction *)add:(Fraction *)f {
    Fraction *result = [[Fraction init] alloc];
    
    result.numerator = (self.numerator * f.denominator) + (self.denominator * f.numerator) ;
    result.denominator = self.denominator * f.denominator;
    [result reduce];
    
    return result;
}

- (Fraction *)mul:(Fraction *)f {
    Fraction *result = [[Fraction init] alloc];
    
    result.numerator = self.numerator * f.numerator;
    result.denominator = self.denominator * f.denominator;
    [result reduce];
    
    return result;
}

- (Fraction *)sub:(Fraction *)f {
    Fraction *result = [[Fraction init] alloc];
    
    result.numerator = (self.numerator * f.denominator) - (self.denominator * f.numerator) ;
    result.denominator = self.denominator * f.denominator;
    [result reduce];
    
    return result;
}

- (Fraction *)div:(Fraction *)f {
    Fraction *result = [[Fraction init] alloc];
    
    result.numerator = self.numerator * f.denominator;
    result.denominator = self.denominator * f.numerator;
    [result reduce];
    
    return result;
}

@end
```

main 에서 사용해보겠습니다.

```swift
#import "Fraction.h"
// ✅ 카테고리도 import.
#import "Fraction+MathOps.h"

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Fraction *a = [[Fraction alloc] init];
        Fraction *b = [[Fraction alloc] init];
        Fraction *result;
        
        [a setTo:1 over:3];
        [b setTo:2 over:5];
        
        [a print]; NSLog(@"   +"); [b print]; NSLog(@"------");
        result = [a add:b];
        [result print];
        NSLog(@"\n");
        
        [a print]; NSLog(@"   -"); [b print]; NSLog(@"------");
        result = [a sub:b];
        [result print];
        NSLog(@"\n");
        
        [a print]; NSLog(@"   *"); [b print]; NSLog(@"------");
        result = [a mul:b];
        [result print];
        NSLog(@"\n");
        
        [a print]; NSLog(@"   /"); [b print]; NSLog(@"------");
        result = [a div:b];
        [result print];
        NSLog(@"\n");
    }
    return 0;
}

/*
1/3
  +
2/5
------
11/15 

1/3
  -
2/5
------
1/-15

1/3
  *
2/5
------
2/15

1/3
  /
2/5
------
5/6
*/
```

## 클래스 확장(Extension)

카테고리를 () 에 아무 이름 없이 사용하면 **클래스 확장**을 정의하게 된다.

이름 없는 카테고리로 선언된 메소드는 분리된 implementation 이 아니라 메인 implementation 부분에 구현된다.

클래스 확장에서 선언된 메소드는 private 하기 때문에 특정 클래스에서만 사용할 데이터나 메소드를 선언하고 싶다면 **클래스 확장**이 유용하다.

(메소드가 인터페이스 부분에서 선언되지 않으면 private 하다. 하지만, private 메소드 명을 안다면 호출할 수 있다.

아래의 코드를 살펴보자.

reduce 메소드는 구현 부분에서만 접근할 것으로 추측된다. 그렇기 때문에 이 메소드는 다른 클래스에서 접근하길 원치 않는다. 이때 Fraction.h 헤더 파일에서 선언을 지우고 Fraction.m 구현 파일에서 선언한다.

```swift
// Fraction.m

#import "Fraction.h"

// MARK: -  Fraction Class

// ✅ 이름없는 카테고리로 선언된 메소드는 메인 구현 부분에서 선언.
// private 으로 정의.
@interface Fraction ()

- (void)reduce;

@end

// public 메서드로 정의.
@implementation Fraction

// ...

@end
```

## 카테고리 주의

카테고리는 클래스에 있는 메소드를 재정의할 수 있는데 이때 메소드를 재정의 후 원래 메소드에 접근할 방법이 없어지기 때문에 즉, 서브클래싱이 아니기 때문에 super 를 호출할 수도 없다. 원래 기능을 모두 추가해줘야 한다.

그래서 **메소드를 재정의해야한다면 서브클래스를 만드는 것이 올바른 프로그래밍일 것이다.**

**또한, 객체에 대한 카테고리 이름은 유일무이해야 한다.**

# 프로토콜과 델리게이션

**프로토콜은 클래스 사이에서 공유되는 메소드 목록이다. 프로토콜에 나열된 메소드들은 구현 부분이 없다.**

일부는 선택적으로 구현할 수도 있고, 구현을 해야만 하는 경우도 있다. 즉, 선택적인 프로토콜을 정의할 수도 필수적인 프로토콜을 정의할 수도 있다.

프로토콜 이름앞에 `@protocol` 지시어를 붙이면 된다. 그리고 인터페이스 부분과 동일하게 메소드를 선언하면 된다.

```swift
// 예시
@protocol NSCopying
- (id)copyWithZone:(NSZone *)zone;
@end
```

`NSCopying` 프로토콜을 채택하려면 `copyWithZone:` 메소드를 구현해야 합니다.

## 프로토콜

프로토콜을 채택하기 위해서는 `@interface` 줄에 프로토콜의 이름을 꺾쇠로 감싸야 한다.

```swift
// 부모 클래스가 NSObject 이고, NSCopying 프로토콜을 채택한다.
@interface AddressBook: NSObject <NSCopying>

// 채택할 프로토콜이 두 개 이상인 경우 쉼표로 나열한다.
@interface AddressBook: NSObject <NSCopying, NSCoding>
```

**프로토콜에 정의된 메소드에 대해 시스템이 해당 헤더 파일을 통해 알아낸다. 그렇기 때문에 인터페이스 부분에서 메소드를 선언하지 않고, 구현 부분에서 메소드를 정의해야 한다.**

프로토콜을 사용하면 내가 만든 클래스의 서브클래스를 만드는 사람들이 구현하고자 하는 메소드를 정의할 수 있습니다. 

다음은 Drawing 프로토콜의 예시 코드입니다. 해당 프로토콜을 채택하는 클래스는 다음의 메소드들을 정의해야만 합니다.

```swift
@protocol Drawing
- (void)paint;
- (void)erase;
// ✅ @optional 다음의 메소드들은 선택 사항이다.
// 즉, outline 메소드를 구현하지 않아도 프로토콜을 준수할 수 있다.
@optional
- (void)outline;
// ✅ @required 다음의 메소드들은 필수 사항이다.
@required
- (void)stroke;
@end
```

### 프로토콜을 채택하는지 확인해보자

객체가 프로토콜을 따르는지 알아보려면 `conformsToProtocol:` 메소드를 사용한다.

```swift
id currentObject;

// ...

// ✅ @protocol 지시어에 프로토콜 이름을 받아서 conformsToProtocol 메소드가 받을 Protocol 객체를 생성.
if ([currentObject conformsToProtocol:@protocol(Drawing)] == YES) {
  // currentObject 에 paint, erase, outline, stroke 메시지를 보낼 수 있다.
}
```

@optional 메소드를 구현했는지 `respondsToSelector:` 메소드를 통해 알 수 있다.

```swift
if ([currentObject respondsToSelector:@selector(outline)] == YES) {
    [currentObject outline];
}
```

변수를 선언할 때, 해당 변수가 프로토콜을 준수하는지 여부를 컴파일러의 도움으로 알 수 있는 방법도 있다.

```swift
id <Drawing> currentObject;

// ✅Drawing 프로토콜을 따르지 않는 객체를 할당하게되면 컴파일러는 경고합니다.
// warning: class `...` does not implement the `Drawing` protocol

// ✅ 하나 이상의 프로토콜을 따르는 경우 다음과 같이 나열할 수 있다.
id <Drawing, NSCoding> myDocument;
```

프로토콜이 다른 프로토콜을 채택하는 경우도 있습니다.

```swift
@protocol Drawing3D <Drawing>
```

Drawing3D 프로토콜이 Drawing 프로토콜을 채택하고 있습니다. 따라서 Drawing3D 프로토콜을 따르는 클래스는 Drawing3D 와 Drawing 프로토콜의 메소드를 모두 구현해야 합니다.

## 델리게이션

**프로토콜을 두 클래스간의 인터페이스라고 생각해도 된다. 프로토콜을 정의하는 클래스는 정의된 처리를 구현하는 클래스에게 권한을 위임하는 것으로 볼 수 있다.**

이런 식으로 특정 이벤트의 응답으로 취하는 동작이나 특정 속성을 정의하는 등의 일을 델리게이트 클래스가 처리하면 클래스를 더 일반적으로 정의할 수 있게 된다. iOS 에서는 이런 개념에 많이 의존한다.

예를 들어, `UITableView` 클래스를 사용할 때 제목, 섹션이 포함하는 행의 수, 각 행에 무엇을 넣어야할지 전혀 모른다. 그래서 `UITableViewDataSource` 라는 프로토콜을 정의하여 책임을 사용하는 개발자에게 위임한다. `UITableViewDelegate` 프로토콜 역시 마찬가지이다.

### 비공식 프로토콜

**비공식 프로토콜은 메소드를 나열하지만 구현하지 않는 카테고리다. 추상 프로토콜이라고 부르기도 한다.**

비공식 프로토콜은 한 이름 안에 묶인 메소드 목록에 불과하고, 이를 통해 메소드를 문서화하고 모듈화하는데 도움이 된다.

비공식  프로토콜을 선언하는 클래스는 프로토콜의 메소드를 직접 구현하지 않고, 서브클래스가 필요에 따라 구현할 메소드들을선택하여 인터페이스 부분에서 다시 선언하고 구현한다. **공식 프로토콜과 달리 컴파일러는 아무런 조언도 하지 않는다.** 

사용하는 예시를 살펴보자.

```swift
#import <Foundation/Foundation.h>

// ✅ 비공식 프로토콜 선언(구현하지 않는 카테고리)
@interface NSObject (MyInformalProtocol)
- (void)processInformal;
@end

// ✅ NSObject 를 상속받았기 때문에 비공식 프로토콜 구현이 가능.
@interface Delegation : NSObject
- (void)processInformal;
@end

@implementation Delegation
- (void)processInformal {
    printf("delegation\n");
}
@end
//비공식 프로토콜 구현 끝.

@interface Processor : NSObject
// delegate 역할 객체 포인터
@property id delegate;
- (void)process;
@end

@implementation Processor
@synthesize delegate;
- (void)process {
  NSLog(@"process \n");
    
    // ✅ 델리게이트 객체가 설정되어 있고, 비공식 프로토콜에 응답 가능할 때 ,
    // delegate 업무 위임하여 수행
    if ([delegate respondsToSelector:@selector(processInformal)]) {
        [delegate processInformal];
    }
}
@end

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        
    // ✅ delegate의 초기값은 nil 로 processInfomal 메소드가 수행되지 않는다.
        id p = [[Processor alloc] init];
        [p process];
        
    // ✅ delegate 설정 후 process 수행.
        id d = [[Delegation alloc] init];
        [p setDelegate:d];
        [p process];
    }
    return 0;
}

/*
process
process
delegation
*/
```

출처: 

[[objective c 프로토콜 3 - 비공식 프로토콜](https://qteveryday.tistory.com/153)](https://qteveryday.tistory.com/153)

### 공식 / 비공식 프로토콜

- 공식 : 객체는 반드시 프로토콜이 요구하는 필수 메시지들을 모두 받아들여야 한다. 런타임뿐만 아니라 컴파일 시에도 이 규칙을 따라야 한다.
- 비공식 : 객체는 프로토콜의 모든 메소드를 받아들이지 않아도 된다. 비공식 프로토콜의 적합성은 런타임 시에 강제 할 수 있지만(`respondsToSelector:` 를 사용하여) 컴파일할 때는 강제할 수 없다.

***(위에서 배운 @optional 지시어는 비공식 프로토콜을 대체하고자 Objective-C 2.0 에 추가되었다.)***

## 복합 객체

지금까지 `서브클래스`, `카테고리`, `프로토콜` 등으로 클래스 정의를 확장하는 방법들을 배웠다. **또 다른 기법으로 다른 클래스에서 객체를 하나 이상 받아 와서 클래스를 정의하는 것이 있다. 이 새 클래스의 객체는 다른 객체들로 구성되어 있어 복합 객체라고 부른다.** 

예를 들어 정사각형에 대해 정의한 Square 와 그 부모 클래스 Rectangle  클래스가 있다고 해보자. 서브클래스로 정의되면 부모 클래스에서 메소드와 인스턴스 변수를 모두 상속받는다. **어떤 경우에는 이렇게 상속받는 것을 원치 않을 수도 있다.**

Rectangle 의 `setWidth:andHeight:` 메소드가 상속된다고 해보자. 모든 변이 동일한 정사각형에는 적합하지 않다.(그렇다고 동작하지 않는다는 것은 아니다.)

또한, 서브클래스는 부모 클래스에 의존한다. 부모 클래스를 수정하면 의도치않게 서브클래스 메소드의 동작을 망칠 수도 있다.

**그래서 복합객체는 확장하고 싶은 클래스의 객체 하나를 인스턴스 변수로 포함하는 새 클래스를 정의하는 것이다.**

복합객체 Square 를 정의해보자.

```swift
// Square.h

#import <Foundation/Foundation.h>

@interface Square : NSObject {
    Rectangle *rect;
}
- (void)setSide:(int)s;
- (int)side;
- (int)area;
- (int)perimeter;
@end
```

이때 변수 rect 에 대해서 메모리 할당을 책임져야 한다.

```swift
Square *mySquare = [[Square alloc] init];
```

위의 코드는 새  Square 객체를 생성하지만 그 안에 담긴 인스턴스 변수 rect 에 저장될 Rectangle 객체는 생성하지 않는다.

**이를 위해 `init` 을 재정의하거나 `initWithSides:` 와 같은 메소드를 추가하여 메모리를 할당한다.**

```swift
// Square.h

#import "Rectangle.h"

@interface Square : NSObject {
    Rectangle *rect;
}
- (Square *)initWithSide:(int)s;
- (Square *)init;
- (void)setSide:(int)s;
- (int)side;
- (int)area;
- (int)perimeter;
@end
```

```swift
// Square.m

#import "Square.h"

@implementation Square

// Square 를 인스턴스화하면 변수 rect 의 메모리 할당을 책임지지 않기 때문에 이니셜라이저를 재정의.
- (Square *)initWithSide:(int)s {
    self = [super init];
    
    if (rect)
        [self setSide:s];
    
    return self;
}
- (Square *)init {
    return [self initWithSide:-1];
}
// side 에 대한 게터, 세터 메소드.
- (void)setSide:(int)s {
    [rect setWidth:s andHeight:s];
}
- (int)side {
    // .연산자는 프로퍼티에 사용하도록 만들어졌고, 작업은 디괄호를 사용한 메세지 표현식을 쓰는 것이 바람직하다.
    return rect.width;
}
- (int)area {
    return [rect area];
}
- (int)perimeter {
    return [rect perimeter];
}

@end
```

### GitHub

https://github.com/hyun99999/Objective-C-Practice

### 참고

[[Objective-C) Category와 Extension](https://babbab2.tistory.com/73)](https://babbab2.tistory.com/73)
