---
title:  "Objective-C) 변수와 데이터 형에 대하여"
categories:
- iOS

date:   2022-08-02  22:04:00 +0900
author_profile: false
---
**** 본 포스팅은 ‘프로그래밍 오브젝티브-C 2.0’ 을 읽으며 실습한 코드와 내용, 추가적으로 궁금한 내용을 정리한 글입니다.***

### 내용

- 변수의 범위, 객체의 초기화 메서드, 데이터 형에 대해 상세히 알아보자
- Objective-C 컴파일러의 지시어를 사용하여 인스턴스 변수의 범위 조작
- 정적 변수, 전역 변수, 외부 변수
- 열거(enumerated) 데이터 형
- typedef

---

## 1. 객체 초기화

 객체를 초기화하고 나서 초깃값을 설정할 수 있다.

```objectivec
// Initializes a newly allocated array by placing in it the objects contained in a given array.
myArray = [[NSArray alloc] initWithArray: myOtherArray];
```

- init method 재정의.

```objectivec
- (id)init {
  // ✅ 부모 초기화 메서드 호출.
  sefl = [super.init];

  if (self) {
    // 부모 초기화 메서드가 성공적으로 동작했다는 뜻.
    // ✅ 초기화 코드
  }
  return self;
}
```

클래스가 초기화 메서드를 하나 이상 갖는다면 그 가운데 하나는 `지정된 초기화 메서드` 여야 하고 다른 메서드는 모두 이 `지정된 초기화 메서드` 를 사용해야 한다. 그래서 `지정된 초기화 메서드` 는 가장 많은 이수를 받는 메서드이고 가장 복잡한 초기화 메서드이다. 이를 통해 주요 초기화 코드를 집중시킬 수 있다. 

서브 클래스를 만들때도 `지덩된 초기화 메서드` 를 재정의하여 새 인스턴스를 초기화해야 한다.

```objectivec
// Fraction.h

#import <Foundation/Foundation.h>

// MARK: -  Fraction Class

@interface Fraction : NSObject

@property int numerator, denominator;

// ✅ 지정된 초기화 메서드. Designated Initializer.
- (Fraction *)initWith:(int)n over:(int)d;

- (void)print;

@end
```

```objectivec
// Fraction.m

#import "Fraction.h"

// MARK: -  Fraction Class

@implementation Fraction

// ✅ 지정된 초기화 메서드. Designated Initializer.
- (Fraction *) initWith:(int)n over:(int)d {
    self = [super init];
    
    if (self)
        [self setTo:n over:d];
        
    return self;
}

- (void)print {
    NSLog(@"%i/%i", self.numerator, self.denominator);
}

@end
```

```objectivec
// main.siwft

#import "Fraction.h"

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Fraction *a, *b;

        // ✅ 지정된 초기화 메서드. Designated Initializer.
        a = [[Fraction alloc] initWith:1 over:3];
        b = [[Fraction alloc] initWith:3 over:7];
        
        [a print];
        [b print];
    }
    return 0;
}

/*
1/3
3/7
*/
```

***위에서 언급했듯이 Fraction 클래스가 서브 클래스될 수 있는 경우 init 메서드의 수정도 필요합니다.***

클래스가 초기화 메서드를 하나 이상 갖는다면 지정된 초기화 메서드를 제외한 초기화 메서드는 지정된 초기화 메서드를 사용해야 하기 때문이다.

```objectivec
// Fraction.h

#import <Foundation/Foundation.h>

// MARK: -  Fraction Class

@interface Fraction : NSObject

@property int numerator, denominator;

- (id)init;
- (id)initWith:(int)n over:(int)d;
// ...

@end
```

```objectivec
// Fraction.m

//...

// ✅ 클래스의 이름을 하드코딩하지 않는다.
// ✅ 서브 클래스의 객체가 부모 클래스와 다를 수 있기 때문이다.
- (id) init {
    // 확인을 위해서 -1 로 초기화.
    return [self initWith:-1 over:-1];
}

// ✅ 일관성을 위해 지정된 초기화 메서드도 id 로 반환타입을 변경했다.
// 지정된 초기화 메서드. Designated Initializer.
- (id) initWith:(int)n over:(int)d {
    // ...
}
```

```objectivec
// main.siwft

#import "Fraction.h"

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Fraction *a, *b, *c;
        c = [[Fraction alloc] init];
        a = [[Fraction alloc] initWith:1 over:3];
        b = [[Fraction alloc] initWith:3 over:7];
        
        [a print];
        [b print];
        [c print];  // -1/-1

    }
    return 0;
}
```

### id

<img width="400" alt="스크린샷 2022-07-27 오후 9 47 00" src="https://user-images.githubusercontent.com/69136340/182381614-65c683a9-6647-4e45-aa3e-9e0db5b3e347.png">

---

실행될 때 프로그램은 모든 클래스의 initalize 메서드를 보낸다. 만일 관련된 서브 클래스가 있다면 부모 클래스가 먼저 이 메세지를 받는다. 이 메시지는 클래스마다 단 한 번 호출되고, 다른 어떤 메시지보다 먼저 보내진다.

초기화가 필요한 클래스를 해당 시점에서 처리하기 위함이다.

## 2. 범위

### 1) 네 가지 지시어

인터페이스 부분에서 인스턴스 변수를 선언할 때 네 가지 지시어를 통해 범위를 더 상세히 설정할 수 있다.

- `@protected` : 그 클래스와 서브클래스에 정의된 메서드는 바로 접근할 수 있음. 인터페이스에서 정의된 인스턴스 변수는 이것이 기본값이다.
- `@private` : 클래스에 정의된 메서드는 바로 접근할 수 있지만 서브클래스의 메서드는 바로 접근할 수 없다. 구현부분에서 정의된 인스턴스 변수는 이것이 기본 값이다.
- `@public` : 인스턴스 변수가 정의된 클래스와 그 밖의 클래스 그리고 모듈에 정의된 메서드라면 어디서든 인스턴스 변수에 접근할 수 있다.
- `@package` : 64비트 이미지의 경우, 그 클래스를 구현하는 이미지 안에서는 어디든 인스턴스 변수에 접근할 수 있다.

```objectivec
@interface Printer {
  @private
    int pageCount;
    int tonerLevel;

  @public
    // 다른 인스턴스 변수
}

...
@end
```

***해당 지시어 다음에 나오는 모든 변수는(} 가 나오기 전까지는) 이 지시어가 나타내는 범위를 갖는다.***

`@public` 지시어는 포인터 연산자(→) 를 써서 인스턴스 변수를 다른 메서드나 함수에서 접근하도록 합니다.

### 2) 자동생성 접근자 메서드

Xcode 4.5 부터는 더 이상 `@synthesize` 지시어를 사용하여 접근자를 자동생성할 필요가 없어졌다. 컴파일러가 해당 지시어가 없이 선언된 프로퍼티에 대해서 자동으로 만들어 주기 때문입니다. 

```objectivec
// @interface 부분에서 아래와 같이 선언했다면
@property BOOL isFinishedFlag;

// @implementation 에서 아래와 같은 코드가 자동으로 포함됩니다.
@synthesize isFinishedFlag = _isFinishedFlag;

// 그러므로 아래와 같이 변수를 사용하여 참조할 수 있습니다.
_isFinishedFlag = NO;

// 또한, 자동생성된 세터 메서드를 통해서 다음과 같이 사용할 수도 있습니다.
self.isFinishedFlag = NO;
```

***구현 부분에서 선언된 인스턴스 변수는 private 이므로 서브클래스에서 이름으로 접근할 수 없습니다. 따라서 서브클래스는 상속받은 접근자 메서드를 써야만 이 값에 접근할 수 있습니다.***

### 3) 전역 변수

메서드와 클래스 정의, 함수 바깥에서 작성했다면 아래의 변수는 어느 모듈에서나 참조할 수 있는 전역(global) 변수이다.

```objectivec
int gMoveNumber = 0;
// 보통 전역 변수의 첫 글자로 g 가 쓰인다고 한다.
```

`외부 변수` 는 그 값을 다른 메서드나 함수에서 접근하고 또 바꿀 수 있는 변수이다. **`외부 변수`에 접근하고 싶은 모듈에서 일반적인 선언 방식과 동일하게 변수를 선언한 다음 그 앞에 extern 키워드를 추가한다.**

시스템은 다른 파일에서 전역으로 정의된 변수에 접근해야 한다는 것을 해당 키워드를 통해 알게 된다.

```objectivec
extern int gMoveNumber;
```

`외부 변수`를 다룰 때는 변수는 소스파일의 어딘가에 `정의`되어 있어야만 하는 규칙이 있다. 즉, 위의 변수를 메서드나 함수 바깥에 extern 키워드 없이 `선언`해야 한다는 이야기이다.

```objectivec
int gMoveNumber;
// 초깃값 설정은 선택적 할당.
```

`외부 변수`를 정의하는 두 번째 방법은 아래와 같이 동시에 진행하는 것이다.

```objectivec
extern int gMoveNumber = 0;
```

 그런데 이 방식은 별로 선호되지 않고, 컴파일러 역시 extern 변수를 선언함과 동시에 값을 할당했다는 경고를 표시합니다.

extern 을 사용하면 `선언`할 뿐 `정의`하는 것이 아니기 때문이다.

***외부 변수를 다룰 때 여러 곳에서 extern 으로 변수를 선언할 수 있지만 정의는 딱 한 번만 해야 한다.***

예제를 살펴보자.

```objectivec
#import "Foo.h"

// ✅ 외부 변수 선언 및 정의.
// 정의는 딱 한 번만.
// 선언은 할당될 저장 공간을 만들지 않지만 정의에서는 공간을 만든다.
int gGlobalVar = 5;

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Foo *myFoo = [[Foo alloc] init];
        NSLog(@"%i", gGlobalVar);
        // 5
        
        [myFoo setgGlobalVar:100];
        NSLog(@"%i", gGlobalVar);
        // 100
    }
    return 0;
}

// ✅ Foo.h 에 선언된 외부 변수를 변경하는 메서드라고 가정.
// setgGlobalVar:

- (void)setgGlobalVar:(int)val {
    // 외부 변수에 접근하기 위해서 extern 키워드를 사용하여 선언.
    extern int gGlobalVar;
    gGlobalVar = val;
}
```

외부 변수에 접근해야 하는 메서드가 많은 경우, extern 선언을 파일 앞부분에 한 번만 해주면 편리하게 사용할 수 있다. 그러나 횟수가 저다면 각각 선언하여 분리해주는 효과를 가져갈 수 있다.

변수에 접근하는 코드가 담긴 파일 내에서 변수가 정의되었다면 extern 을 일일히 선언할 필요는 없다.(즉, 전역변수로써 사용한다는 의미이다.)

### 4) 정적 변수

앞서 정의한 외부 변수는 전역 변수만이 아니라 외부 변수도 된다. 그러나 전역 변수이면서도 외부 변수는 되지 않기를 원하는 경우가 많다.

**즉, 특정 모듈(파일)에서는 지역 변수이면서 전역으로 변수를 정의하고 싶을 때가 있다는 것이다.**

```objectivec
static int gGlobalVar = 0
// 이 정의가 있는 파일 안에서, 명령문 다음의 모든 지점에서 gGlobalVar 에 접근할 수 있다.
```

클래스 메서드가 변수에 접근하고 값을 설정해야 할 때가 있다. 예를 들어 생성한 객체의 개수를 세는 클래스 할당 메서드가 있다.

```objectivec
// Fraction.h

#import <Foundation/Foundation.h>

// MARK: -  Fraction Class

@interface Fraction : NSObject

@property int numerator, denominator;

// 인스턴스 메서드는 -.
- (id)init;
- (id)initWith:(int)n over:(int)d;
- (void)print;
- (double)convertToNum;
- (void)setTo:(int)n over:(int)d;
- (Fraction *)add:(Fraction *)f;
- (void)reduce;

// ✅ 클래스 메서드는 +.
+ (Fraction *)allocF;
+ (int)count;

@end
```

```objectivec
// Fraction.m

#import "Fraction.h"

// MARK: -  Fraction Class

// ✅ 정적변수
static int gCounter;

@implementation Fraction

// ✅ alloc 은 물리적인 메모리 할당을 다루는 메서드이기 때문에, 이를 재정의하는 것은 좋지 않은 프로그래밍 습관이다.
/// ✅ alloc 메서드를 재정의하는 대신 상속받아 사용.
+ (Fraction *)allocF {
    // 정적변수가 동일한 파일에 선언되어있기 때문에 extern 키워드를 사용하지 않아도 된다.
    // 선언하더라도 메서드를 읽는 사람이 메서드 바깥에서 변수가 정의되어있다는 것을 좀 더 빨리 이해할 뿐이다.
    // 변수 이름의 g 역시 읽는 사람을 배려한 네이밍이다.
    extern int gCounter;
    ++gCounter;
    
    return [Fraction alloc];
}

// ✅ 생성한 객체의 개수를 추적하는 gCounter 를 반환하는 클래스 메서드.
+ (int)count {
    extern int gCounter;
    
    return gCounter;
}

// ...

@end
```

```objectivec
// main.m

#import "Fraction.h"

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Fraction *a, *b, *c;
        
        // ✅ 다른 정적 변수와 같이 0 이외의 값을 설정하고 싶다면 init 을 재정의 하면 된다.
        NSLog(@"Fractions allocataed: %i", [Fraction count]);
        // Fractions allocataed: 0
        
        c = [[Fraction allocF] init];
        a = [[Fraction allocF] init];
        b = [[Fraction allocF] init];

        NSLog(@"Fractions allocated: %i", [Fraction count]);
        // Fractions allocataed: 3
    }
    return 0;
}
```

## 3. 열거(enumerated) 데이터 형

열거 데이터 형은 키워드 `enum` 을 붙여서 정의한다. 

```objectivec
enum flag { false, true);
```

중괄호 안에 대입이 가능한 값들을 정의한다. 안타깝게도 컴파일러는 이 규칙을 위반해도 경고하지 않는다.

```objectivec
enum flag endOfData, matchFound;

// ✅ 사용
endOfData = true;

if (matchFound == false)
    // ...

// ✅ 특정 정수 값을 연계하고 싶을 경우 해당 식별자에 정수를 할당할 수 있다.
// 특정 정수 값이 할당되었다면 다음에 나오는 열거형 식별자는 그 값에 1씩 더한 값을 할당 받는다.
enum direction { up, down, left = 10, right };
// 0, 1, 10 ,11

enum boolean { no = 0, false = 0, yes = 1, true = 1 };

enum month { january = 1, february, march, april, may, june, july, august, september, october, november, december }

// ✅ 사용
enum month thisMonth;

thisMonth = december;
// 12
```

- 예제

```objectivec
#import <Foundation/Foundation.h>

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        enum month { january = 1, february, march, april, may, june, july, august, september, october, november, december };
        enum month amonth;
        enum month lastMonth;
        int days;
        
        NSLog(@"Enter month number: ");
        scanf("%i", &amonth);
        
        switch (amonth) {
            case january:
            case march:
            case may:
            case july:
            case august:
            case october:
            case december:
                days = 31;
                break;
            case april:
            case june:
            case september:
            case november:
                days = 30;
                break;
            case february:
                days = 28;
                break;
            default:
                NSLog(@"bad month number");
                days = 0;
                break;
        }
        
        if (days != 0) {
            NSLog(@"Number of days is %i", days);
        }
        
        if (amonth == february) {
            NSLog(@"...or 29 if it's a leap year");
        }

        // ✅ 형 변환 연산자를 사용하여 정수 값을 명시적으로 대입할 수도 있다.
        lastMonth = (enum month)(amonth - 1);
        NSLog(@"lastMont: %i", lastMonth);
        
        return 0;
    }
}

/*
Enter month number:
5
Number of days is 31
*/
```

**형 변환 연산자를 위와 같이 사용하지 않더라도 컴파일러는 경고하지 않는다.**

Xcode 4.4 부터는 열거형을 정의할 때 상수 데이터 형과 열거형 이름을 연관시킬 수 있다.

```objectivec
enum iPhoneModels: unsigned short int { iPhone, iPhone3G, iPhone3GS, iPhone4, iPhone4S, iPhone5 };
```

데이터 형의 이름을 생략해도 되며, 특정 열거 데이터 형을 정의하면서 변수를 동시에 선언할 수도 있다.

```objectivec
enum { east, west, south, north } direction;
```

열거 데이터 형은 동일한 범위에서 정의된 변수 이름이나 식별자와 겹치지 않도록 정의해야 한다.

### 데이터 형 변환 규칙

1. 만일 두 항 중 하나가 long double 이라면 다른 항과 결과 값 **모두** long double 형이 된다.
2. 만일 두 항 중 하나가 double 이라면 다른 항과 결과 값 **모두** double 형이 된다.
3. 만일 두 항 중 하나가 float 라면 다른 항과 결과 값 **모두** float 형이 된다.
4. 만일 두 항 중 하나가 _Bool, char, short int, 비트필드 혹은 열거 데이터 형이라면, int 형으로 변환된다.
5. 만일 두 항 중 하나가 long long int 라면 다른 항과 결과 값 **모두** long long int 형이 된다.
6. 두 항 중 하나가 long int 라면 다른 항과 결과 값 **모두** long int 형이 된다.

## 4. typedef 명령문

***데이터 형에 다른 이름을 부여하는 기능을 지원한다.***

```objectivec
typedef int Counter;
// Counter 라는 이름을 int 형과 동일하게 만들어 준다.

// 사용
Counter j, n;
```

이를 통해 변수 선언에 가독성을 높여줄 수 있다.

```objectivec
typedef enum { east, west, south, north } Direction;

Direction step1, step2;
```

---

### GitHub

https://github.com/hyun99999/Objective-C-Practice
