---
title:  "Objective-C) 전처리기"
categories:
- iOS

date:   2022-08-26  00:00:00 +0900
author_profile: false
---
**** 본 포스팅은 ‘프로그래밍 오브젝티브-C 2.0’ 을 읽으며 실습한 코드와 내용, 추가적으로 궁금한 내용을 정리한 글입니다.***

### 내용

- 전처리기를 알아보자.
- #define 명령문
- #import 명령문
- 조건 컴파일(#ifdef, #if, #elif, #else, #endif, #ifndef, #undef)

***전처리기는 컴파일 과정에서 프로그램 코드에 산재한 특별한 명령문을 인식합니다. 이 전처리 명령문을 만들기 위해서는 샵(#)을 줄의 맨 앞에 붙여야 한다.***

# ✅ #define 명령문

#define 문의 주 용도는 상수에 심벌명을 부여하는 것이다. 다음의 전처리 명령문은 TRUE 라는 이름이 값 1과 동일하도록 정의하고, FALSE 라는 이름이 값 0과 동일하도록 정의한다.

```swift
#define TRUE 1
#define FALSE 0
```

이제 상수 1과 0을 사용할 자리에 대신 TRUE 과 FALSE 를 사용할 수 있다.

- 기존에 우리가 알던 문법과는 다릅니다. 대입할 때 등호가 사용되지도 않고, 명령어의 끝에 세미콜론이 등장하지도 않는다.

이렇게 선언된 전처리문은 아래와 같이 사용됩니다.

```swift
gameOver = TRUE;
if (gameOver == FALSE) { }
```

- `#define 문` 은 보통 `#import 문` 이나 `#include 문` 다음에 보통 정의된다. 이 순서가 반드시 지켜져야하는 것은 아니지만 심벌명이 사용되기 전까지는 정의되어 있어야 한다.

그렇기 때문에 헤더 파일에 넣어 하나 이상의 소스 파일에서 사용할 수도 있는 방법도 있다.

❗️ **장점은** 프로그램 전체에서 상수가 사용된 곳의 값을 변경하는 대신 #define 문에서만 값을 변경해주면 되는 용이함이 있다.

- 또한, 대문자로 쓰여 변수와 시각적으로 구분하는 것이 좋다.
- 자주 사용되는 네이밍으로는 k 로 시작하는 것이 있다. (ex. `kMaximumValues`)

### 고급 형태

간단한 상수 값 외에도 표현식 등을 포함할 수 있다.

```swift
#define TWO_PI 2.0 * 3.141592654

// return TWO_PI
// 와 같이 반환에서도 당연히 사용 가능하다.
```

전처리기가 오른쪽에 있는 글자 그대로 대치해주기 때문에 `;` 세미콜론 같은 경우를 넣게되면 그대로 대치된다. 이때문에 정말 사용해야할 경우가 아니라면 추가하지 말자.

```swift
#define TWO_PI 2.0 * 3.141592654

return TWO_PI 2.0 * r;
// 는 아래와 같이 대치된다.
return 2.0 * 3.141592654; * r;
```

다음과 같은 디파인 문은 어떨까?

```swift
#define AND &&
#define OR ||
#define EQUALS ==
// 당연히 가능하다 하지만 이러한 재정의는 좋지 않다.
// 다른 사람들이 당신의 코드를 이해하기 더 어려워질 것이다.
```

디파인에서 다른 디파인을 참조할 수 있을까?

```swift
// 그렇다. 다음과 같이 사용 가능하다.

#define TWO_PI 2.0 * PI
#define PI 3.141592654
```

#define 을 잘 사용하면 주석을 달 필요가 줄어든다.

```swift
#define IS_LEAP_YEAR year % 4 == 0 && year % 100 != 0 \
                   || year % 400 == 0
if (IS_LEAP_YEAR) { ... }
// 전처리기는 디파인 정의가 한 줄이라고 정의한다. 두 번째 줄이 필요하다면 맨 마지막을 \(역슬래시) 문자로 끝내야 한다.
// 변수 year 뿐만 아니라 어느 해든 윤년을 확인할 수 있도록 만든다면 좋을 것 같다.

#define IS_LEAP_YEAR(y) y % 4 == 9 && y % 100 != 0 \
                     || y %400 == 0
if (IS_LEAP_YEAR (year)) { ... }
// y 는 글자를대치하는 일을 할 뿐이기 때문에 위와 같이 사용할 수 있다.
// 그리고 이런 정의를 매크로 라고 부른다.
```

- 매크로의 함정

```swift
#define SQUARE(x) x * x

SQUARE (v);
// v * v

SQUARE (v + 1);
// (v + 1) * (v + 1) 를 기대하고 사용했을 것이다. 하지만 디파인은 텍스트를 대치해준다.
// v + 1 * v + 1 로 대치되고, 우리가 예상했던 결과 값을 나타내주지 않는다.
// 이를 해결하기 위해서는 디파인 문을 다음과 같이 수정해야 한다.
#define SQUARE(x) (x) * (x)

// 이 외에도 다음과 같이도 사용할 수 있다.
#define MAX(a, b) ( ((a) > (b)) ? (a) : (b) )
#define IS_LOWER_CASE(x) ( ((x) >= 'a') && ((x) <= 'z') )
#define TO_UPPER(x) ( IS_LOWER_CASE (x) ? (x) - 'a' + 'A' : (x) )
```

# ✅ #import 명령문

전처리기는 다른 파일에 있는 정의를 #import 문을 사용하여 프로그램에 포함시킨다. 이 파일들은 .h 로 끝나고 헤더 혹은 인클루드 파일이라고 부른다.

```swift
#define INCHES_PER_CENTIMETER 0.394
#define CENTIMETERS_PERINCH (1 / INCHES_PER_CENTIMETER)

#define QUARTS_PER_LITER 1.057
#define LITERS_PER_QUART (1 / QUARTS_PER_LITER)

#define OUNCES_PER_GRAM 0.035
#define GRAMS_PER_OUNCE (1 / OUNCES_PER_GRAM)
```

이 코드를 metric.h 라는 파일에 따로 입력해 두었다고 해보자. 이 정의가 필요하다면 간단하게 다음의 지시어를 사용하면 된다.

**그리고 `#import 명령문`이 있는 곳에 직접 입력해 넣는 것과 동일하게 취급된다.**

```swift
#import "metric.h"
// 이는 metric.h 에 정의된 #define 문을 참조하기 전에 선언되야 한다.
```

- 헤어 파일 이름 양쪽의 따옴표는 전처리기에게 파일을 하나 혹은 그 이상인 파일 디렉터리에서 찾으라는 지시다.

다음과 같이 파일명을 <> 로 감싸도 된다.

- 전처리기가 지정된 파일을 특별한 ‘시스템' 헤더 파일 디렉터리에서 찾고, 현재 소스의 디렉터리는 검색하지 않는다.

```swift
#import <Foundation/Foundation.h>
```

# ✅ 조건 컴파일

Objective-C 에서는 조건 컴파일이라는 기능을 제공한다.

조건 컴파일을 통해 다른 컴퓨터 시스템 환경에서 돌아가도록 컴파일하는 하나의 프로그램을 만들 수 있고, 다양한 명령문을 껐다 켜는데도 사용된다. 예를 들어 프로그램 실행의 흐름을 추적하는 디버깅 명령문이 있다.

## 1️⃣ #ifdef, #endif, #else, #ifndef 문

프로그램이 시스템에 따른 특정 매개변수에 의존해야 하는 때가 있다. iPhone 과 iPad 와 같은 다른 기기에 대해서나 특정 다른 운영체제에 따라 다르게 지정되야 한다.

전처리기의 조건 컴파일 기능을 사용하면 프로그램이 다른 시스템에서 실행될때마다 디파인 값을 변경해줘야하는 문제를 해결할 수 있다.

- ifndef 문 : 메크로가 정의되어 있지 않다면 수행됩니다.(#ifdef 반대)

아래는 IPAD 심벌이 정의된 경우와 그 외의 경우에 대한 조건 컴파일 문이다.

```swift
#ifdef IPAD
# define kImageFile @"barnHD.png"
#else
# define kImageFile @"barn.png"
// #ifdef 의 짝꿍
#endif
// 전처리기 명령문을 시작하는 # 다음에는 원하는 만큼 빈칸을 넣을 수 있다.
```

- **#ifdef 의 심벌명이 정의되어 있다면 #else, #elif, #endif 가 나올 때까지만 코드를 처리한다.**

위의 심벌은 단순하게 다음과 같이 선언된도 된다.

```swift
#define IPAD 1

#define IPAD
// 뒤에 아무런 텍스트가 없어도 가능하다.
```

예를 들어, 아래는 DEBUG 라는 이름이 정의되어 있을 때만 특정 변수의 값이 출력되는 코드이다.

```swift
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSString *userName = @"user1";
        int userId = 11;
#ifdef DEBUG
        NSLog(@"User name = %@, id = %i", userName, userId);
#endif
        
    }
    return 0;
}

// DEBUG 가 정의된 경우.
// User name = user1, id = 11
```

- 이를 통해 프로그램을 디버그할 때만 DEBUG 를 정의하여 모든 디버깅 명령문을 컴파일 할 수 있다. 이렇게 되면 정상 작동 시에는 디버깅 명령문들이 컴파일되지 않으므로 프로그램 크기를 줄이는 효과가 있다.
- Xcode 의 Build Setting 에서 디버깅할 때 정의되는 매크로를 작성할 수 있다.

![스크린샷 2022-08-25 오후 1.10.57.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/eb00d657-abe7-4e6f-9ced-661db381c943/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-08-25_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_1.10.57.png)

## 2️⃣ #if 와 #elif 전처리 명령문

**#if 전처리 명령문은 조건 컴파일을 조작하는 일반적인 방법을 제공한다. #if 문은 상수 표현식이 0이 아닌지를 테스트한다.** 0이 아니라면 #else, #elif, #endif 가 나올 때까지 코드를 처리하고 0이라면 무시된다.

- 다음 defined 연산자를 #if 에서 사용할 수 있다. #ifdef 와 동일한 기능을 한다.

```swift
#if defined (DEBUG)
...
#endif

// 아래와 동일하다.
#ifdef DEBUG
...
#endif
```

다음의 코드 시퀀스로도 사용 가능하다.

```swift
// DEBUG 가 정의되어 있고, 그 값이 0이 아닐 때만 실행된다.
#if defined (DEBUG) && DEBUG
...
#endif
```

## 3️⃣ #undef 명령문

**정의한 이름을 취소할 필요가 있다. 이때는 #undef 명령문을 사용하여 특정 이름에 대한 정의를 제거할 수 있다.**

다음의 명령문은 IPAD 정의를 제거한다.

```swift
#undef IPAD
```

- 이후에 나오는 #ifdef IPAD 혹은 #if defined (IPAD) 명령문은 FALSE 가 된다.

---

### GitHub

https://github.com/hyun99999/Objective-C-Practice
