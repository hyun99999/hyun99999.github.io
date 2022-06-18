---
title:  "WWDC22) Embrace Swift generics"
categories:
- iOS

date:   2022-06-18  17:22:00 +0900
author_profile: false
---
<img width="500" src= "https://user-images.githubusercontent.com/69136340/174425733-058c4774-9160-4b67-a9fe-35bdd693c998.png">

## Abstraction separated ideas from specific details

function  에서 기능성을 추출할 때, details 은 abstracted 로 부터 멀어집니다. 그리고 abstraction 은 details 을 반복없이 무슨 일이 일어나고 있는지 아이디어를 표현할 수 있습니다.

<img width="400" alt="스크린샷 2022-06-18 오후 3 23 51" src="https://user-images.githubusercontent.com/69136340/174425746-37773903-4d8a-4bde-9045-dcd2900188a3.png">

Swift 에서는 concrete type 을 abstract 할 수 있습니다.

<img width="300" alt="2" src="https://user-images.githubusercontent.com/69136340/174425769-c6010c22-4ba8-4b62-94c2-b8d97d986d2e.png">

_바로 이것..! 처럼요_

서로 다른 details 를 가진 동일한 아이디어의 set of types 가 있는 경우 abstract code 를 작성하여 concrete types 으로 작업할 수 있습니다.

오늘은 다음에 대해서 알아보겠습니다.


farm 시뮬레이션을 위한 코드를 만들어봅시다!

## 👉 Model with concrete types
<img src="https://user-images.githubusercontent.com/69136340/174425783-7d952ac9-02e6-4887-9fb5-9dca70209054.png" width ="400">

아래는 Cow 라는 구조체가 있습니다.

- Cow 는 hay(건초)를 먹는 eat 메서드를 가지고,
- Hay 구조체는 건초를 생산하는 작물인 Alfalfa 를 기르기 위한 grow 라는 static 메서드를 가집니다.
- Alfalfa 구조체는 Hay 를 수확하는 harvest 메서드를 가집니다.
- 마지막으로, 우리는 소에게 먹이를 주는 feed 메서드가 있는 구조체를 추가할 것입니다.

<img src="https://user-images.githubusercontent.com/69136340/174425793-a3562593-df37-4be8-8fec-de15c2ab03e6.png" width ="600">

짠, 이제 우리는 cow 들을 farm 에서 기를 수 있어요.

<img src="https://user-images.githubusercontent.com/69136340/174425815-daa31cd5-79b7-4304-9009-4e0df4849a07.png" width ="400">

하지만 좀 더 많은 동물들을 원하는 경우에는 어떻게 할까요?

feed 메서드를 overload 할 수 있겠지만, 각 구현은 매우 흡사하고, 반복되는 코드가 되어버렸습니다.

<img src="https://user-images.githubusercontent.com/69136340/174428576-0860eca8-e930-44bb-b2dd-105e81e5422a.png" width ="600">

## 👉 Identify common capabilities

다음은 동물 타입들 사이의 common capabilities 를 파악하는 것입니다.

<img src="https://user-images.githubusercontent.com/69136340/174428650-bdbff39c-de9f-40e7-9def-2c1f353bdb60.png" width ="600">

동물들은 각기 다른 방법으로 먹이를 먹고, eat 메서드는 다르게 구현되야 합니다.

우리가 원하는 것은 **abstract code 가 eat 메서드를 호출하도록 하고, concrete type 에 따라 abstract code 가 다르게 동작**하는 것입니다.

이처럼 다른 concrete types 에 대해 다르게 동작하는 abstract code 를 `polymorphism` 즉, `다형성` 이라고 합니다. 다형성을 사용하면 코드 사용 방식에 따라서 하나의 코드 조각이 여러 동작을 할 수 있습니다.

다향성은 다양한 형태가 있습니다.

<img src="https://user-images.githubusercontent.com/69136340/174428668-04ba4d26-5ffd-42e2-be12-13a27b79e8f6.png" width ="600">


첫 번째는 overloading 으로, 동일한 함수 호출이 argument 의 타입에 따라 다른 의미를 가질 수 있습니다. 오버로딩은 일반적인 솔루션이 아니기 때문에 `ad-hoc polymorphism(임시 다향성)`이라고 불립니다.

우리는 앞서 오버로딩으로 반복적인 코드로 이어지는 것을 확인했습니다.

두 번째는 subtype 으로 구현해보겠습니다.

마지막으로, generic 을 사용하는 parametric polymorphism 이 있습니다. type parameters 를 사용하여 다른 타입과 작동하는 코드를 작성할 수 있도록 하며 구체적인 타입이 arguments 로 사용됩니다.

우리는 이미 overloads 를 배제했으므로 subtype polymorphism 을 사용해 보겠습니다. subtype 관계의 하나의 방법은 class hierarchy 입니다.

<img src="https://user-images.githubusercontent.com/69136340/174428677-5ce2795a-3f93-4f72-8166-8dd603a34499.png" width ="600">

Animal 이라는 클래스를 만들고 각 동물 클래스에서 Animal 을 상속받아서 eat 메서드를 오버라이딩하는 것입니다. 즉, subtype polymorphism 을 사용해서 하위 클래스에서의 구현을 호출합니다.

Animal 의 eat 메서드의 매개변수 타입을 아직 채우지 못했으며, 이 코드에는 몇가지 red flags! 가 있습니다. 위험 신호가 있다는 소리죠..

- 다른 Animal 인스턴스간에 상태를 공유할 필요가 없고, 원하지 않더라도 클래스를 사용하면 reference semanitcs 를 강제합니다. (클래스는 참조 타입이기 때문입니다.)
- subclasses 가 base class 의 메서드를 override 하는 것이 요구되는데 이것을 잊게되더라도 런타임까지 확인되지 않습니다.
- model of abstraction 의 가장 큰 문제는 각 Animal subtype 이 다른 음식 타입을 먹고 이러한 dependency 을 표현하는 것이 class hierarchy 에서는 매우 어렵다는 것입니다.

우리가 취할 수 있는 한 가지 방식은 `Any` 와 같이 덜 구체적인 유형을 허용하도록 하는 것입니다.

<img src="https://user-images.githubusercontent.com/69136340/174428730-bc0173ea-8735-4ac2-8946-56156fb0d525.png" width ="600">

하지만, 이 방법은 위와 같이 올바른 타입이 전달되었는지 as? 를 사용해서 다운캐스팅을 진행해야 합니다. 따라서 추가적인 boilerplate code 를 부과했지만 실수로 잘못된 타입을 집어넣었을 때는 런타임에만 잡을 수 있는 또 다른 버그를 남길 수 있다는 것입니다.

자! 다른 방법을 시도해 봅시다.

type parameter(타입 인자. ex. T) 를 도입해서 type-safe 방법으로 동물의 food 의 타입을 표현할 수 있습니다.

이 접근 방식은 Food 타입 인자가 Animal 클래스의 선언으로 승격되는 것을 보여줍니다. 이것은 부자연스럽습니다. 동물이 동작하려면 음식이 필요하지만 음식을 먹는 것이 동물의 핵심 목적이 아니며 동물에서 작동하는 많은 코드들은 음식에 대해서 전혀 신경쓰지 않을 것이기 때문이죠. 끄덕끄덕..

<img src="https://user-images.githubusercontent.com/69136340/174428753-0d0c727c-107b-41e3-922a-4d051e549ffe.png" width = "600">

위의 코드를 보게 되면 Animal 클래스에 대한 모든 참조는 food 타입을 지정해야만하게 됩니다.

<img src="https://user-images.githubusercontent.com/69136340/174428774-09d505fa-8586-4128-898d-8e2c8ec06118.png" width ="600">

또한, 필요하다면 각 Animal 에 타입을 추가해야 하는 경우도 있습니다. 이처럼 위의 방법 중 어느 것도 좋은 방식은 없었습니다.

### **우리는 작동 방식에 대한 세부정보 없이 capabilites 의 타입을 보여줄 수 있는 구조를 원합니다!**

동물에게는 두 가지 capabilites 가 있습니다.

- A specific type of food.
- An operation for consuming some of its food.

이제 위의 기능을 하는 인터페이스를 만들어 봅시다.

## 👉 Build an interface

Swift 에서는 protocol 을 사용하여 수행할 수 있습니다.

프로토콜은 준수하는 타입의 기능을 설명하는 추상화 도구입니다.

### A protocol separated ideas from implementation details.

<img width="400" alt="15" src="https://user-images.githubusercontent.com/69136340/174428851-5b381473-ff97-4805-9c16-e6b15e34c056.png">

type parameter 처럼 associated type 은 구체적인 타입에 대한 placeholder 역할을 합니다.

associated type 을 사용하므로써 프로토콜을 준수하는 특정 타입에 의존합니다. 이것은 특정 타입의 Animal 의 각 인스턴스가 같은 food 타입을 가지는 것을 보장합니다.

<img width="400" alt="16" src="https://user-images.githubusercontent.com/69136340/174428900-b3e77261-e41d-467b-9588-bec0243109e4.png">


다음으로 음식을 소비하는 작업은 eat 메서드에 매핑됩니다. Animal 의 Feed 타입을 매개변수로 받습니다.

프로토콜에는 메서드의 구현이 되어있지 않으며, 구현하려면 concrete Animal 타입이 필요합니다.

<img width="300" alt="17" src="https://user-images.githubusercontent.com/69136340/174428928-7468c32b-e306-4c7b-a01e-b8b2574dfac1.png">

프로토콜을 준수하기 위해서 각 Animal 타입은 eat 메서드를 구현해야하며 컴파일러는 Feed 타입을 유추할 수 있습니다.

### 이렇게 우리는 동물의 common capabilites 를 성공적으로 식별하고, 프로토콜 인터페이스를 사용하여 능력을 표현했습니다.

다음은 generic code 를 사용해봅시다.

## 👉 Write generic code

우리는 모든 concrete Animal 타입에 대해 작동하는 하나의 구현을 만들고 싶습니다! parametric polymorphism 을 사용하고 메서드가 호출될 때 type parameter 를 도입할 것입니다.

<img width="400" alt="18" src="https://user-images.githubusercontent.com/69136340/174428941-02070f8f-3e22-425f-b986-b4d258edad60.png">

그리고 항상 concrete Animal(구현된 Animal. 즉, Animal 을 준수한 것.)가 Animal 프로토콜을 준수하기를 원하므로 `<A: Animal>` 로 표시합니다.

프로토콜을 준수하는 것은 꺽쇠 괄호로 작성하거나 trailing `where` 절에 작성할 수 있습니다. 여기서 다른 type parameters 와의 관계도 지정할 수 있습니다.

<img width="400" alt="19" src="https://user-images.githubusercontent.com/69136340/174428947-0de8f081-bada-41fa-87e5-0f50e544ccdf.png">

```swift
func feed<A>(_ animal: A) where A: Animal
```

위의 제네릭 패턴은 실제보다 훨씬 복잡해 보입니다. 우리는 더 간단하게 표현할 수 있는 방법이 있습니다.

```swift
func feed(_ animal: some Animal)
```

타입 인자를 명시적으로 작성하는 대신 `some Animal` 을 작성하여 protocol conformance 를 표현할 수 있습니다. 이전 선언과 동일하지만 불필요한 타입 인자 목록과 `where` 절이 사라졌습니다.

### some

`some Animal` 에서 some 은 작업 중인 특정 타입이 있음을 나타냅니다. some 키워드 뒤에는 항상 conformance 요구사항이 따릅니다. 이 경우에는 특정 타입은 매개변수에 대해 Animal 프로토콜을 준수해야함을 의미합니다.

some 키워드는 파라미터 및 result 타입에도 사용됩니다.

```swift
var body: some View { ... }
var body: modifiercontent<text, _backgroundmodifier<color>> {

}
```

 SwiftUI 코드에서도 확인할 수 있습니다. body 프로퍼티는 특정 View 타입을 반환하지만, 코드는 특정 타입이 무엇인지 알 필요가 없습니다. 이처럼 concrete type 에 대한 placeholder 를 나타내는 abstract type 이 바로 `opaque type, 불투명 타입` 이라고 합니다.

예를 들어, 뷰를 추가하거나 변경할 때마다 새로운 타입으로 body 를 정의해주어야 한다면 어떻게 될까요? 이것이 바로 opaque type 을 사용하는 이유입니다. 이런 불투명 타입은 protocol type 에 비해서 강력한 보장을 제공합니다. 왜냐면 protocol type 은 특정 프로토콜을 준수하고 있기만하다면 mulitple type 에 대해서 리턴할 수 있지만, opaque type 은 single type 이어야만 합니다.

**참고:** 

[[Swift] Opaque Type vs Protocol Type](https://eunjin3786.tistory.com/490)

---

실제 사용되는 타입을 `underlying type` 이라고 부릅니다.

이런 식으로 값에 접근할 때마다 동일한 underlying type 을 가져오도록 보장됩니다. 

<img width="400" alt="20" src="https://user-images.githubusercontent.com/69136340/174428991-16216b11-f044-44ab-b823-e73e980b8170.png">

→ (function arrow) 는 parameter position 과 result position 을 나눕니다.

opaque type 의 위치로 프로그램의 어느 부분이 abstract  trype 을 보고 concrete type 을 정하는지 결정합니다.

<img width="300" alt="21" src="https://user-images.githubusercontent.com/69136340/174429004-99a4e809-5af4-43d8-908a-1078a93d1800.png">

명시되는 타입 인자는 input 측에서 선언되고, 호출자가 underlying type 을 결정하고 구현은 abstract type 을 사용합니다. 일반적으로 opaque type 또는 result type 대한 값을 제공하는 프로그램 부분은 underlying type 을 결정하고, 사용하는 프로그램 부분은 abstract type 을 봅니다.

<img width="400" alt="22" src="https://user-images.githubusercontent.com/69136340/174429014-5d0efdaf-cad2-43b3-9bdf-338358bc6f95.png">

**local variable 에 대해서 underlying type 은 할당의 right-hand side 에서 유추됩니다.** 이것은 opaque type 의 지역변수가 항상 초기값을 가져야함을 의미합니다. 제공하지 않으면, 컴파일러는 error 을 보고합니다.

또한, underlying type 은 해당 변수의 scope 에 대해서는 고정되어야 하므로 underlying type 을 변경하려면 error 가 발생합니다.

<img width="500" alt="23" src="https://user-images.githubusercontent.com/69136340/174429033-a06c9e45-d8f6-40d7-af78-036c2ce58b7a.png">

**opaque type 의 파라미터의 경우, underlying type 은 호출 부분의 argument 로 부터 유추됩니다.**

some 을 파라미터 포지션에서 사용하는 것이 Swift 5.7 의 새로운 기능입니다.

underlying type 은 파라미터 범위 내에서만 수정되므로 각 호출은 다른 argument type 을 제공할 수 있습니다.

<img width="500" alt="24" src="https://user-images.githubusercontent.com/69136340/174429043-1ccc0719-0592-4cc4-87e6-068a5b643710.png">

**opaque type 의 result type 의 경우, underlying type 이 구현의 반환값에서 유추됩니다.**

opaque result 를 가진 메서드 혹은 computed property 는 프로그램 어디에서나 호출되기 때문에 명명된 값의 범위는 전역입니다. 이는 underlying return type 이 모든 return 문에서 동일해야 함을 의미합니다. 

<img width="500" alt="25" src="https://user-images.githubusercontent.com/69136340/174429059-1d194612-242c-4bb4-804c-14b58caae6fa.png">

위의 코드처럼 그렇지 않은 경우, 컴파일러는 underlying 값이 미스매칭되었다고 오류를 보고합니다.

<img width="500" alt="26" src="https://user-images.githubusercontent.com/69136340/174429083-cad6554b-cb7d-47fa-a8c6-c57cb7146cd8.png">

opaque SwiftUI view 의 경우, ViewBuilder 은 각 분기에 대해 동일한 underlying return type 을 갖도록 control-flow 문을 변환할 수 있습니다. 따라서 ViewBuilder 로 문제를 해결할 수 있습니다.

드디어.. feed  메서드로 돌아가 봅시다.

<img width="400" alt="27" src="https://user-images.githubusercontent.com/69136340/174429100-6588ef35-4a11-4bff-9a88-1c26c8ecc4d7.png">

다른 곳에서 opaque type 을 참조할 필요가 없기 때문에 파라미터 리스트에서 some 을 사용할 수 있습니다. 함수에서 opaque type 을 여러번 참조해야할 때가 타입 인자가 유용할 때입니다.

<img width="500" alt="28" src="https://user-images.githubusercontent.com/69136340/174429102-d68b7b0d-7a67-488e-a1fa-3f3a4ee0c98d.png">

예를 들어, Habitat 라는 다른 associatedtype 을 추가하는 경우 농장에 지정된 동물의 서식지를 구축하기를 원할 수 있습니다. 이 경우, result type 은 특정 동물 타입에 따라 달라지므로 파라미터와 리턴 타입에 타입 인자 A 를 사용해야 합니다.

<img width="400" alt="29" src="https://user-images.githubusercontent.com/69136340/174429127-da609da3-a132-4ee1-84be-778bdef87141.png">

opaque tpye 을 여러번 참조해야하는 또 다른 상황은 generic type 입니다. 코드는 종종 타입인자를 선언하고, 타입인자를 저장 프로퍼티에 사용하고, 다시 memberwise initializer 에 사용합니다. 다른 context 에서 제네릭 타입을 참조하려면 `< >` 안에 타입 인자를 명시적으로 지정해야 합니다. 이것은 사용 방법을 명확히 하는데 도움이 될 수 있습니다.

이제 feed 메서드를 구현해봅시다.

<img width="400" alt="30" src="https://user-images.githubusercontent.com/69136340/174429142-39dcb30c-7b94-43fa-b91d-50ebdc505d33.png">

animal 매개변수를 사용하여 grow 하는 작물 타입에 접근할 수 있습니다. 다음으로, harvest 메서드를 호출해서 작물을 수확해야 합니다. 그리고 animal 에게 먹일 수 있습니다.

underlying animal type 이 고정되어 있기 때문에 컴파일러는 다양한 메서드 호출에서 타입의 관계에 대해서 알고 있습니다.

<img width="500" alt="31" src="https://user-images.githubusercontent.com/69136340/174429149-6e0af2c6-a8e3-4e5f-b9f8-4ab204d9fe35.png">

이러한 정적 관계는 동물에게 잘못된 타입의 먹이를 먹이는 실수를 방지합니다.

배열을 사용하는 feedAll 메서드를 추가해보겠습니다. element 타입이 Animal 프로토콜을 준수해야하는 것은 알고있지만 배열이 다양한 유형의 동물을 저장할 수 있기를 원합니다.

<img width="500" alt="32" src="https://user-images.githubusercontent.com/69136340/174429159-e6c465f1-2b72-43bf-bf48-402bc7caabd0.png">

`some` 에는 변경할 수 없는 underlying type 이 있습니다. underlying type 은 고정되어 있기 때문에 배열의 모든 요소는 동일한 타입을 가져야 합니다.

따라서 `some Animal` 은 원하는 것을 표현하지 못합니다. 여기서 우리는 모든 유형의 Animal 을 나타낼 수 있는 supertype 이 필요합니다.

`any Animal` 사용하면 arbitrary type 의 Animal 을 표현할 수 있습니다.

이러한 storage flexibility 을 허락하기 위해서 any Animal 타입은 메모리에 특별한 표현이 있습니다.

<img width="400" alt="33" src="https://user-images.githubusercontent.com/69136340/174429194-2f409091-9f7b-42ef-b427-3da695c9c522.png">

값이 상자 안에 직접 들어갈 만큼 충분히 작기도 하지만 때론 상자에 비해 너무 커서 다른 곳에 할당하고 해당 값에 대한 포인터를 저장하기도 합니다.

any Animal 타입은 동적으로 모든 concrete Animal type 을 저장할 수 있고, existential type 이라고 불립니다.

그리고 다른 concrete types 에 대해 동일한 표현을 사용하는 방법을 `type erasure` 라고 부릅니다. concrete type 은 컴파일 타임에 erase 되고 런타임에만 알려집니다.

extential type any Animal 의 두 인스턴스는 정적 타입은 동일하지만 동적 타입은 다릅니다. type erasure 는 다른 동물 값 사이의 type-level distinction 을 제거하여 다른 동적 유형의 값을 동일한 정적 유형으로 교환 가능하게 사용할 수 있도록 합니다.

type erasure 를 사용하여 우리가 원하는 배열을 작성할 수 있습니다.

<img width="400" alt="34" src="https://user-images.githubusercontent.com/69136340/174429205-02b8a8aa-21ec-4dc7-8c5e-65f7b23b6110.png">

**다음과 같이 associated types 가 있는 프로토콜에 any 키워드를 사용하는 것이 Swift 5.7 의 새로운 기능입니다.**

<img width="600" alt="35" src="https://user-images.githubusercontent.com/69136340/174429216-5716d6bc-55ef-4b88-9d1a-44062ace30d2.png">

feedAll 메서드를 구현하기 위해서 각 anmial 에 대해서 eat 메서드를 호출하겠습니다. 이때 반복에서 underlying animal 에 대한 Feed 타입을 가져와야 합니다. 그러나 Animal 에 대해 eat 메서드를 호출하려고 하자 컴파일 오류가 발생합니다.

<img width="400" alt="36" src="https://user-images.githubusercontent.com/69136340/174429230-eb098d60-8285-4c2c-a8ec-bf678a4d145e.png">

type-level distinction 을 제거했기 때문에 특정 animal types 에 대한 의존하는 associated types 도 제거했습니다. 그래서 우리는 이 동물이 어떤 먹이를 예상하는지 알 수 없습니다.

그래서! 우리는 type relationships 를 의존하려면 특정 타입의 동물이 고정된 context 로 돌아가야 합니다. Animal 에서 직접 eat 메서드를 호출하는 대신 some Animal 을 사용하는 feed 메서드를 호출해야 합니다.

<img width="400" alt="37" src="https://user-images.githubusercontent.com/69136340/174429242-b6d51d0c-f8ce-464f-8dd2-25b9871c24ba.png">

any Animal 과 some Animal 은 다른 타입이지만, 컴파일러는 underlying value 를 unboxing 하고, some Animal 파라미터로 직접 전달함으로써 any Animal 의 인스턴스를 some Animal 로 변환할 수 있습니다.

**arguments 를 unboxing 하는 것이 Swift 5.7 의 새로운 기능입니다.**

some Animal 파라미터 범위의 경우, 값이 underlying type 으로 고정되있으므로 underlying type 에 대한 operations 를 포함하여 associated types 에도 액세스 가능합니다.

<img width="400" alt="38" src="https://user-images.githubusercontent.com/69136340/174429251-3a158f05-3c6d-427f-a514-9b18f9b9c269.png">

이것은 정적 타입의 표현성을 갖는 컨텍스트로 돌아갈 수 있도록 해줍니다. 위의 some 을 사용한 파라미터를 통해서 각 동물을 feed 메서드로 전달할 수 있습니다. 이를 통해 각 반복에서 특정 동물에게 먹일 적절한 작물을 재배하고 수확하여 먹일 수 있습니다.

이 과정에서 some 과 any 가 서로 다른 기능을 하는 것을 확인했습니다.

<img width="500" alt="39" src="https://user-images.githubusercontent.com/69136340/174429278-253c6162-f80f-4052-b887-c155a3b1e04b.png">

### some

some 을 사용하면 underlying type 이 고정됩니다. 이를 통해 generic code 의 underlying type 에 대한 type relationships 를 의존할 수 있으므로 작업중인 프로토콜의 API 및 associated types 에 대한 전체 액세스 권한을 갖게 됩니다.

### any

arbitrary concrete type 을 저장하는 경우 any 를 사용하면 됩니다. any 는 type erasure 을 제공하여 서로 다른 종류로 이루어진 collections 을 표시하고 optional 을 사용하여 underlying type 의 부재를 표현합니다. 또한 세부적인 구현사항을 추상화할 수 있습니다.

일반적으로 some 을 기본으로 사용하고, 임의의 값을 저장해야 한다면 any 로 변경하면 됩니다.

<img width="300" alt="40" src="https://user-images.githubusercontent.com/69136340/174429288-deede377-75d7-4a3d-a4cb-33c2e7493280.png">

이 접근 방식을 사용하면 storage flexibility 가 필요할 때 type erasure 와 semantic limitations 만 지불하면 됩니다. 이 워크플로우는 마치 mutation 이 필요하다는 것을 알 때까지 기본적으로 let-constants 를 작성하는 것과 유사합니다.

<img width="500" alt="41" src="https://user-images.githubusercontent.com/69136340/174429304-77b1218f-9857-4880-b6c8-cdf0794c5161.png">

<img width="500" alt="42" src="https://user-images.githubusercontent.com/69136340/174429306-40053779-a21b-4623-93ef-deacca0bdacb.png">
