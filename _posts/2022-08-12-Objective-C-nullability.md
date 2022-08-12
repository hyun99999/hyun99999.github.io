---
title:  "Objective-C) nullability"
categories:
- iOS

date:   2022-08-12  22:13:00 +0900
author_profile: false
---
카테고리를 생성했는데 다음과 같이 인터페이스 위아래로 매크로가 있었다. 무엇일까?

```swift
// Fraction+MathOps.h

#import "Fraction.h"

NS_ASSUME_NONNULL_BEGIN

@interface Fraction (MathOps)

@end
NS_ASSUME_NONNULL_END
```

Swift 가 등장하면서 Swift 의 null 의 호환성을 위해서(Objective-C 에서는 nil 이라고 합니다.) Xcode 6.3 부터 Objective-C 에 **nullability** 라는 것이 추가되었습니다.

Swift 에서는 `?` 를 통해 옵셔널을 표시하는데 컴파일러는 Objective-C 에서 해당 변수가 옵셔널인지 논옵셔널인지 알 수 있는 방법이 없기 때문이죠.

예를 들어, `NSView` 와 `NSView?` 를 Objective-C 는 `NSView *` 로 나타냅니다. 또한, Swift 의 컴파일러는 `NSView *` 가 옵셔널인지 아닌지 모르기 때문에 암시적 추출 옵셔널인 `NSView!` 로 가져오게 됩니다.

그래서 다음과 같이 annotation을 사용해서 표시해줍니다. nullability 는 포인터형만 지정하면 됩니다.

### nullability annotation

- `nullable` , `_Nullable`

null 일 수 있다. 즉, Swift 의 옵셔널입니다.

- `nonnull` , `_Nonnull`

null 일 수 없다. 즉, Swift 의 논옵셔널입니다.

- `null_unspecified`, `_Null_unspecified`

null 여부가 지정되어 있지 않다. nullability 관련 키워드를 설정해주지 않은 경우 null_specified 가 기본값으로 설정. 암시적 추출 옵셔널 형식(implicitly unwrapped optional)입니다.(=!)

- `null_resettable`

기본값을 가지고 있어 nill 일 수 없는 경우입니다. 암시적 추출 옵셔널 형식(implicitly unwrapped optional)입니다.(=!)

아래의 nullability annotation 이 없는 경우 즉, `null_unspecified`, `_Null_unspecified` 를 Swift 에서 어떻게 가져오는지 살펴봅시다.

```swift
@interface MyList : NSObject
- (MyListItem *)itemWithName:(NSString *)name;
- (NSString *)nameForItem:(MyListItem *)item;
@property (copy) NSArray<MyListItem *> *allItems;
@end
```

Swift 는 각 객체 인스턴스 매개변수, 반환 값 및 속성을 암시적 추출 옵셔널로 가져옵니다.

```swift
class MyList: NSObject {
    func item(withName name: String!) -> MyListItem!
    func name(for item: MyListItem!) -> String!
    var allItems: [MyListItem]!
}
```

annotation 을 사용한 예제도 살펴볼까요?

```swift
@interface MyList : NSObject
- (nullable MyListItem *)itemWithName:(nonnull NSString *)name;
- (nullable NSString *)nameForItem:(nonnull MyListItem *)item;
@property (copy, nonnull) NSArray<MyListItem *> *allItems;
@end
```

Swift 에서는 nualbility annotation 을 통해 다음과 같이 가져옵니다.

```swift
class MyList: NSObject {
    func item(withName name: String) -> MyListItem?
    func name(for item: MyListItem) -> String?
    var allItems: [MyListItem]
}
```

### underscored 와 non-underscored form

`_Nonnull`, `_Nullable` annotation 을 포인터 타입에 적용해야 하지만 일반 C const 키워드에도 사용할 수 있습니다. 

`nullable`, `nonnull` 처럼 소문자로 사용할 수도 있는데 이때는 단순 객체 또는 블록 포인터인 경우와 괄호 바로 뒤에 사용할 수 있습니다. 프로퍼티의 경우는 annotation 을 프로퍼티 목록으로 이동하여 non-undersocred form 으로 사용할 수 있습니다.

```swift
@property Fraction * _Nonnull a;
@property (nullable)Fraction *b;

- (Fraction * _Nullable)f;
- (nonnull Fraction *)with:(nonnull NSString *)f;
```

underscored 보다 non-undersocred form 이 보기 낫지만 여전히 헤더의 모든 타입에 적용해야 합니다. audited regions 를 사용하여 범위적으로 적용해 봅시다!

### audited regions

`NS_ASSUME_NONNULL_BEGIN` 과 `NS_ASSUME_NONULL_END` 매크로를 사용하여 범위적으로 nonnull 옵션을 지정할 수 있습니다. 이 범위에서 nollable annotation 을 사용하지 않은 경우 nonnull 로 인식되고, 필요에 따라 nullable annotation 을 사용하면 됩니다.

```swift
NS_ASSUME_NONNULL_BEGIN
@interface AAPLList : NSObject <NSCoding, NSCopying>
// ...
// ✅
- (nullable AAPLListItem *)itemWithName:(NSString *)name;
- (NSInteger)indexOfItem:(AAPLListItem *)item;

// ✅
@property (copy, nullable) NSString *name;
@property (copy, readonly) NSArray *allItems;
// ...
@end
NS_ASSUME_NONNULL_END

// --------------

self.list.name = nil;   // okay

AAPLListItem *matchingItem = [self.list itemWithName:nil];  // warning!
```

안전을 위해 이 규칙에 몇 가지 예외가 있습니다.

- typedef 타입은 본질적으로 nullability 아니기 때문에 audited regions 안에서도 nonnull 하지 않습니다.
- `id *` 와 같은 복잡한 포인터 타입은 명시적으로 주석을 달아야 합니다. 예를 들어 nullable 객체 참조에 대한 non-nullable 포인터를 지정하려면 `_Nullable id * _Nonnull` 을 사용합니다.
- `NSError **` 는 매개변수를 통해 오류를 반환하는데 자주 사용되므로 항상 nullable NSError 참조에 대한 nullable 포인터로 간주됩니다.

**출처:** 

[Nullability and Objective-C - Swift Blog](https://developer.apple.com/swift/blog/?id=25)

[Apple Developer Documentation](https://developer.apple.com/documentation/swift/designating-nullability-in-objective-c-apis)

[Difference between nullable, __nullable and _Nullable in Objective-C](https://stackoverflow.com/questions/32452889/difference-between-nullable-nullable-and-nullable-in-objective-c)

[[Apple Dev Reference] Nullability and Objective-C](https://rhammer.tistory.com/145)

[Swift에 따른 Objective-C의 새로운 기능들 - yagom's blog](https://blog.yagom.net/519/)
