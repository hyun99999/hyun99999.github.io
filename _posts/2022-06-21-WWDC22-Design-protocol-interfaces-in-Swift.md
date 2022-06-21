---
title:  "WWDC22) Design protocol interfaces in Swift"
categories:
- iOS

date:   2022-06-21  21:26:00 +0900
author_profile: false
---
****본 글은 WWDC 를 보고, 번역 및 요약 그리고 실행해보는 스터디 프로젝트의 일환입니다.***

해당 세션에는 세 가지 주요 주제가 있습니다.

<img width="500" alt="1" src="https://user-images.githubusercontent.com/69136340/174794409-c448869f-3566-4f50-a229-71ee542e234f.png">

- 어떻게 `result type erasure` 가 작동하는지 설명하여 associated type 이 있는 프로토콜이 어떻게 existential `any` type 과 상호작용하는 방법을 보여드리겠습니다.
- 구현과 인터페이스를 분리하여 캡술화를 개선하기 위해 opaque result types 를 사용하는 방법을 설명하겠습니다.
- 프로토콜의 same-type requirements 가 여러 다른 concrete types 의 집합간의 관계를 모델링할 수 있는 방법을 볼 수 있습니다.

## Undestand type erasure

---
<img width="600" alt="2" src="https://user-images.githubusercontent.com/69136340/174794461-24b6c6e6-d36e-4d76-91b7-2e477e9e7347.png">

`Embrace swift generics` 세션에서 Cow 와 Chicken 에서 서로 다른 return type 의 produce() 를 추상화하는 가장 좋은 방법은 associated type 을 사용하는 것이 었습니다. 

associated type 을 사용하면 특정 타입의 Animal 의 경우 produce() 메서드를 호출하면 특정 Animal 에 대한 특정 Food 가 반환됩니다.(다이어그램 참고)

`Self` 은 프로토콜을 준수하는 실제 concrete type 입니다. Self 타입에는 Food 를 준수하는 associated type `Commodity` 가 있습니다. concrete Chicken 과 Cow 타입간의 관계와 assocated type 다이어그램에 대해서 살펴보겠습니다.

Chicken, Cow 타입은 CommodityType(각각 Egg 와 Milk) 을 가진 Animal 프로토콜을 준수하고 있습니다.

<img width="500" alt="3" src="https://user-images.githubusercontent.com/69136340/174794472-bb5e09ef-3141-4cb2-976e-359cbb4af511.png">

이제 `any Animal` array 가 저장되어있는 Farm 을 살펴봅시다.

<img width="400" alt="4" src="https://user-images.githubusercontent.com/69136340/174794605-c422ef96-3bb9-4ab2-bc17-c55c7eef1106.png">

`Embrace Swift generics` 에서 알아보았듯이 다른 concrete types 에 대해 동일한 표현을 사용하는 이러한 전략을 `type erasure` 라고 했습니다. `produceCommodities()` 메서드는 간단해보이지만 type erasure 가 underlying type 에 대한 static type relationships 를 제거하는 것을 알고 있습니다.

<img width="600" alt="5" src="https://user-images.githubusercontent.com/69136340/174794669-f44a9552-e08e-4950-9168-1c397ce00101.png">

existential type 에서 associated type 을 리턴하는 메서드를 호출할 때(`produce()` 호출) 아래와 같이 컴파일러는 type erasure 를 사용하여 호출의 result type 을 결정합니다.

<img width="600" alt="6" src="https://user-images.githubusercontent.com/69136340/174794703-979dddd6-ce16-4ed6-94e9-85ae166208da.png">

concret Animal 타입과 associated CommodityType 을 any Animal 과 any Food 로 대체하여 관계를 지웠습니다. any Food 타입은 associated CommodityType 의 `upper bound` 라고 부릅니다. any Animal 에 대해서 produce() 메서드가 호출되기 때문에 반환값이 type erase 되어서 any Food 타입 값을 제공합니다. 

Swift 5.7 의 새로운 기능인 `associated-type erasure` 가 어떻게 작동하는지 살펴보겠습니다.

<img width="600" alt="7" src="https://user-images.githubusercontent.com/69136340/174794738-6de87837-a4ed-4f0f-a924-e9a0a4639b7b.png">

프로토콜 메서드의 result type 에 나타나는 associated type 은 `producing position` 에 있다고 합니다. 메서드를 호출하면 이 타입의 값이 생성되기 때문입니다. 

<img width="600" alt="8" src="https://user-images.githubusercontent.com/69136340/174794771-8d688d8b-7181-4e39-a74a-9e6b6eb9f13b.png">

produce() 메서드를 any Animal 에서 호출할 때 우리는 컴파일 타임에는 concrete result type 을 알지 못합니다. 하지만, upper bound 의 subtype 은 알 수 있습니다.(즉, any Food 인 것은 알 수 있습니다.)

위의 예제에서는 런타임에서는 any Animal 이 Cow 인 것을 알 수 있습니다. produce() 메서드는 Milk 를 반환합니다. Milk 는 associated CommodityType 의 upper bound 인 any Food 에 저장될 수 있습니다.(아까 `produceCommodities()` 메서드의 리턴 값이 any Food 였다.)

이것은 Animal 프로토콜을 준수하는 모든 concrete types 에게 항상 안전하게 적용됩니다.

다른쪽으로 associated type 이 메서드나 생성자의 파라미터에 등장하는 것을 생각해봅시다.

<img width="600" alt="9" src="https://user-images.githubusercontent.com/69136340/174794835-43fb3bbb-742a-424a-acaa-f7e45720dcd4.png">

`eat()` 메서드는 associated FeedType 을 `consuming postion` 에 가지고 있습니다.

<img width="600" alt="10" src="https://user-images.githubusercontent.com/69136340/174794899-50b90ea3-ed97-4b8c-9940-03a94d30bf70.png">

eat 메서드를 호출하려면 값을 파라미터로 넘겨주어야 합니다. 변환이 반대방향으로 진행되기 때문에 type erasure 를 수행할 수 없습니다. associated type 의 upper bound 는 concrete type 을 알 수 없기 때문에 안전하게 변환되지 않습니다.

Animal 프로토콜과 관련된 FeedType 의 upper bound 는 any AnimalFeed 입니다. 그러나 위의 상황처럼 any AnimalFeed 가 주어지면 Hay concrete type 을 정적으로 저장한다는 것을 보장할 방법이 없습니다. 다음 처럼요!

<img width="600" alt="11" src="https://user-images.githubusercontent.com/69136340/174795332-c6c419f1-3b33-479d-8617-18d16c6520bf.png">

type erasure 는 consuming position 에 있는 associated types 으로 작업하는 것을 허용하지 않습니다. 대신 opaque `some` type 을 사용하는 함수에 전달해서 existential `any` type 을 언박싱해야 합니다.

associated types 의 type erasure 동작은 실제로 Swift 5.6 에서 볼 수 있는 기존 기능과 유사합니다. cloning reference type 을 위한 프로토콜을 고려해보자.

아래의 프로토콜은 Self 를 반환하는 clone() 메서드를 정의합니다. any Cloneable 타입의 값에 대해 clone() 메서드를 호출하면 리턴 타입 Self 가 upper bound 까지 type erase 됩니다. Self 의 upper bound 는 항상 프로토콜 자체이므로 any Cloneable 타입의 새 타입을 반환합니다.

<img width="600" alt="12" src="https://user-images.githubusercontent.com/69136340/174795385-10d5df8a-9be2-45ae-a25a-9e4716d469db.png">

요약하자면 `any` 를 사용하여 값 타입이 프로토콜을 준수하는 concrete type 을 저장하는 existential type 을 선언할 수 있습니다.

<img width="500" alt="13" src="https://user-images.githubusercontent.com/69136340/174795418-49f6bcee-b1eb-4089-a41e-7e647f2c6ec5.png">

이것은 associated types 가 있는 프로토콜에서도 동작합니다. producing postion 에서 associated type 이 있는 프로토콜 메서드를 호출할 때 associated type 은 upper bound 까지 type erase 됩니다. 

## Hide implementation details

---

concrete types 에 대한 추상화는 함수 input 뿐만 아니라 oupt 에서도 유용하므로 concrete types 는 구현에서만 볼 수 있습니다. 

concrete result types 를 추상화하여 구현 세부 정보에서 코드의 필수 인터페이스를 분리하여 더 모듈화 되고, 강력하게 만드는 방법을 살펴보겠습니다.

동물에게 먹이를 줄 수 있도록 Animal 프로토콜을 일반화해보겟습니다. 동물은 배고프면 먹어야 합니다.

<img width="600" alt="14" src="https://user-images.githubusercontent.com/69136340/174795461-1269ee25-8c39-4420-8032-e8d589fb1a01.png">

hungryAnimals 의 반복을 통해서 먹이를 줄 것 입니다. 이때 feedAnimals() 메서드가 한번만 반복하기 때문에 hungryAnimals 의 수가 많은 경우 비효율적입니다. 그래서 `lazy.filter` 로 대체함으로써 우리는 임시 할당을 방지할 수 있습니다.

이제, hungryAnimals 타입의 프로퍼티는 concrete type 보다 복잡한 `LazyFilterSequence of Array of any Animal` 로 선언되어야 합니다.

<img width=" 500" alt="15" src="https://user-images.githubusercontent.com/69136340/174796504-a9ef5e40-3919-41b1-a436-6aab0f6d0798.png">

이것은 불필요한 구현 세부사항을 노출합니다. 클라이언트인 feedAnimals() 는 `hungryAnimals` 의 구현에서 `lazy.filter` 를 사용한 것을 신경 쓰지 않습니다. 이때 opaque result type 을 사용해서 추상적인 인터페이스 뒤에 concrete type 을 숨길 수 있습니다. 

<img width="400" alt="16" src="https://user-images.githubusercontent.com/69136340/174796555-60f3d9ed-b11e-4814-9a58-7969dd1db8e2.png">

이제 클라이언트는 collection 프로토콜을 준수하는 구체적인 유형을 얻고 있다는 것만 알지 concrete type 을 알지 못합니다.

<img width="400" alt="17" src="https://user-images.githubusercontent.com/69136340/174796682-4200107b-2035-40e2-9370-f50f8284e8c0.png">

그러나 작성된 대로 이것은 실제로 클라이언트에서 너무 많은 정적인 정보를 숨깁니다. hungryAnimals 가 collection 을 준수하는 concrete type 을 출력한다고 선언하고 있지만, 이 collection 의 element 타입에 대해서는 아무 것도 모릅니다. element type 이 any Animal 이라는 지식 없이는 할 수 있는 것은 전달하는 것 뿐입니다. 즉, Animal 프로토콜 의 메서드를 호출할 수 없습니다.

opaque result type `some Collection` 에 집중해 보겠습니다. 구현 세부 정보를 숨기는 것과 풍부한 인터페이스를 노출하는 것 사이에서 올바른 균형을 잡을 수 있습니다.

`constrained opaqeue result types` 는 Swift 5.7 부터 적용되는 것입니다.

<img width="400" alt="18" src="https://user-images.githubusercontent.com/69136340/174796737-ea90db77-cc83-49eb-9cd5-6dbce274832c.png">

이것은 프로토콜 이름 뒤에 꺾쇠 괄호 안에 type arguments 를 적용하여 작성하면 됩니다. Collection 프로토콜에는 단일 type argument 인 Element type 이 있습니다. 이제 hungryAnimals 가 constrained opaque result type 으로 선언되면 `LazyFilterSequence of Array of any Animal` 라는 사실이 클라이언트에서 숨겨집니다. 그러나 클라이언트는 여전히 `any Animal` 과 같은 Element associated type 을 가진 Collection 프로토콜을 준수하는 concrete type 이라는 것을 여전히 알고 있습니다.

이것이 우리가 원하는 인터페이스입니다.

<img width="500" alt="19" src="https://user-images.githubusercontent.com/69136340/174796801-ddeaedae-1eea-4cab-8b75-1e633f4f3975.png">

`feedAnimals()` 의 for 루프에서 animal 변수는 `any Animal` 타입을 가지므로 Animal 프로토콜의 메서드를 각 배고픈 동물들에 대해 호출할 수 있습니다. 

<img width="400" alt="20" src="https://user-images.githubusercontent.com/69136340/174796854-12ee93d5-d2b4-47bf-9114-ccac41ec083e.png">

`Collection<Element>` 로 정의되고, associated type 으로 Element 이 primary associated type 라고 선언하기 때문에 작동이 가능합니다.

<img width="700" alt="21" src="https://user-images.githubusercontent.com/69136340/174796895-74104687-9bdd-4224-ab56-668e52c626b2.png">

hungryAnimals 을 lazily 또는 eagerly 하게 계산할지 옵션을 갖기를 원하는 경우 opaque Collection 을 사용하면 두 가지 다른 underlying types 를 반환한다는 오류가 발생합니다. 

대신 `any Collection of any Animal` 을 반환하여 API 가 호출간에 다른 타입을 반환할 수 있음을 알려줌으로써 이 문제를 해결할 수 있습니다.

opaque types 를 사용하여 일반적인 code 를 작성하려면 abstract type relationships 에 의존해야 합니다. 관련 프로토콜을 사용하여 여러  abstract types 간에 필요한 타입 관계를 식별하고 보장하는 방법에 대해 논의해 보겠습니다.

## Identify type relationships

---

앞서 만든 Animal 프로토콜에 AniamlFeed 타입의 새로운 associatedtype 과 동물에게 FeedType 을 먹도록 지시하는 eat() 메서드를 추가할 것입니다. 흥미롭게 하기 위해서 동물에게 먹이를 주기 전에 적절한 타입의 작물을 재배하고 수확하여 사료를 생산하는 복잡성을 추가해 보겠습니다.

<img width="400" alt="22" src="https://user-images.githubusercontent.com/69136340/174797074-87972536-44fb-46fe-b72c-3591f5720ac4.png">

다음은 두 가지 concrete type 입니다.

<img width="600" alt="23" src="https://user-images.githubusercontent.com/69136340/174797085-76fb8fd7-6f34-48f5-afba-ae1b6e7f98fa.png">

위의 두가지 concrete types 에 대해 추상화해보겠습니다. 이를 통해 feedAnimal() 메서드를 한 번만 구현하면 cow, chicken 및 다른 새로운 타입의 동물을 먹일 수 있습니다.

<img width="500" alt="24" src="https://user-images.githubusercontent.com/69136340/174797270-f00ca198-831c-4dd6-b011-e7c3a89ad579.png">

feedAnimal() 메서드는 Animal 프로토콜의 associated type 이 consuming postion 에 있는 eat() 메서드와 함께 작동하기 때문에 파라미터 타입으로 `some Animal` 을 가지도록 선언해서 언박싱하겠습니다.

시작하기 위해서 AnimalFeed 와 Crop 프로토콜을 선언하겠습니다.

<img width="600" alt="25" src="https://user-images.githubusercontent.com/69136340/174797333-70a9ece9-96be-4d70-8a56-758a0cccbdb5.png">

다이어그램을 통해 보듯이 AnimalFeed 와 Crop 사이에서 무한 중첩과 함께 영원히 계속 됩니다.

<img width="600" alt="26" src="https://user-images.githubusercontent.com/69136340/174797419-18d7b148-aaf4-470c-8c93-bcd4e810402a.png">

 AnimalFeed 프로토콜에서 시작했지만, Crop 프로토콜을 사용해도 비슷한 상황이 발생합니다. 단지 하나만 이동했을 뿐입니다.

이러한 프로토콜이 concrete types 간의 관계를 올바르게 모딜링하는지 살펴봅시다. 

동물에게 먹이를 주기(eat)전 작물(crop)을 재배(harvest)해야 사료(animal feed)로 가공할 수 있습니다. 

<img width="600" alt="27" src="https://user-images.githubusercontent.com/69136340/174797453-3bacbfad-672d-47cc-a6fa-9ad8621e5fcf.png">

Animal 을 준수하는 타입을 얻을 수 있고, Animal 은 associated type 으로 AnimalFeed 를 준수하는 FeedType 을 가지고 있습니다.

<img width="600" alt="28" src="https://user-images.githubusercontent.com/69136340/174797511-2b70ef15-d9f8-48e6-b0e2-557aa3fe949e.png">

이 타입은 메서드 grow() 를 호출하는 기반으로 사용할 수 있습니다. grow() 메서드는 AnimalFeed 의 CropType 을 리턴합니다.

CropType 은 Crop 을 준수한다는 것을 알고 있으므로 harvest() 메서드를 호출할 수 있습니다.

이를 통해 우리는 무엇을 돌려받을 수 있을까요? 바로 FeedType 입니다.

<img width="600" alt="29" src="https://user-images.githubusercontent.com/69136340/174797568-2b58dafc-6a7a-4fd4-97e1-b6ca4b69306b.png">

불행하게도 이것은 잘못된 타입입니다.

eat() 메서드는 `(some Animal).FeedType` 예상하지 `(some Animal).FeedType.CropType.FeedType`  예상하지 않습니다. 

<img width="600" alt="30" src="https://user-images.githubusercontent.com/69136340/174797648-53fd471c-4d6c-4585-ac1a-ce32a4fd6216.png">

concrete types 간의 관계를 정의하기에 프로토콜의 선언이 너무 일반적이었습니다. 즉, 원하는 관계를 정확하게 모델링하지 못하고 있습니다.

<img width="400" alt="31" src="https://user-images.githubusercontent.com/69136340/174797691-ba9d7b04-8c50-4c83-ae92-40f8438d28d2.png">

그 이유를 이해하기 위해서 Hay 와 Alfalfa 타입을 살펴보겠습니다. hay 를 기르면 alfalfa 를, alfalfa 를 수확하면 hay 를 얻는 식입니다. 코드를 리펙토링하고 있는데 실수로 Alfalfa 에서 harvest() 메서드의 리턴값을 변경하여 hay 대신 Scratch 를 반환한다고 상상해 볼까요?

이런 유연한 변경 후에도 concrete type 은 여전히 AnimalFeed 및 Crop 프로토콜의 요구사항을 충족합니다. 작물을 재배하고, 수확하면 우리가 시작한 것과 동일한 animal feed 가 생상된다는 우리가 원하는 규칙을 위반하더라도 말입니다.

<img width="600" alt="32" src="https://user-images.githubusercontent.com/69136340/174797759-9b59c30f-ceb5-4fc7-8e28-3ce1e4e6b0df.png">

AnimalFeed 프로토콜을 다시 살펴보겠습니다. 여기서 진짜 문제는 너무 많은 별개의 associated type 가 있다는 것입니다. 이러한 associated Types 중 두 개가 실제로 동일한 concrete type 이라는 것을 기록해야 합니다.(즉, crop 을 재배하고, 수확해서 feed 가 되는데 crop 과 feed 는 결국 같다. 이것을 별개의 associated type 로 가지고 있다.)

## where

---

`where` 절로 작성된 same-type requirement 를 사용하여 이러한 associated type 간의 관계를 표현할 수 있습니다. 

<img width="500" alt="33" src="https://user-images.githubusercontent.com/69136340/174797797-5ad83a3e-8a4b-45f8-ab57-b6173ee5dcd6.png">

same-type requirement 은 중첩될 수 있는 associated types 이 실제로 동일한 concrete type 이어야 한다는 정적 보장을 나타냅니다. `Self.CropType.FeedType` 이 `Self` 와 동일한 타입임을 선언합니다. 이것을 다이어그램으로 보게되면 아래와 같습니다.

<img width="500" alt="34" src="https://user-images.githubusercontent.com/69136340/174797847-532155b5-9114-43f1-9f8f-7a3eaadfc729.png">

AnimalFeed 를 준수하는 concrete type 에 CropType 이 있고, 이것의 FeedType 은 original AnimalFeed 타입의 concrete type 입니다. 중첩된 구조 대신 단일 쌍으로 축소했습니다. Crop 프로토콜은 어떨까요?

<img width="500" alt="35" src="https://user-images.githubusercontent.com/69136340/174797913-831f88b9-d068-40f4-bbfc-0938a78471be.png">

(위와 같이 where 절을 사용해서 아까 우려되었던 harvest 를 통해서 기르는 작물과 먹이가 동일하지 않은 경우를 방지할 수 있습니다.)

이 두 프로토콜이 same-type requirements 를 갖추었으므로 `feedAnimal()` 메서드를 다시 살펴볼 수 있습니다.

<img width="600" alt="36" src="https://user-images.githubusercontent.com/69136340/174797992-00c10fb9-084e-4830-acf7-e37ef34b71e3.png">

이제는 또 다른 associated type 을 얻는 대신 animal 이 기대하는 정확한 animal feed 타입을 eat() 할 수 있습니다. 마지막으로, 지금까지 본 모든 것을 하나로 묶는 Animal 프로토콜의 associated type 다이어그램을 살펴보겠습니다.

<img width="300" alt="37" src="https://user-images.githubusercontent.com/69136340/174798023-4d376dea-2968-4551-86f7-71207604aff3.png">

Cow 와 chicken 이 있습니다. 각 관계를 어떻게 모델링하는지 주목해서 살펴보겠습니다.

<img width="600" alt="38" src="https://user-images.githubusercontent.com/69136340/174798054-8e0a49f6-b8e0-4ffa-9ee0-a1abf2c6df48.png">

다른 중첩된 associated types 의 동등성을 정의할 수 있었습니다. 그러면 일반 코드는 프로토콜 요구사항에 대해 여러 호출을 함께 연결할 때 이러한 관게에 의존할 수 있습니다.

<img width="600" alt="39" src="https://user-images.githubusercontent.com/69136340/174798138-7de473f0-c027-4f8b-b03c-0679db38a3bb.png">

이번 세션에서 우리는 type erasure 가 언제 안전한지, type relationships 가 보장되는 context 가 있어야 할 때를 탐구했습니다.

그런 다음 opaque result types 와 existential types 모두에 사용할 수 있는 primary associated types 를 사용해서 풍부한 타입의 정보를 보존과 구현 상세 정보를 숨기는 것 사이의 균형을 유지하는 방법에 대해 논의했습니다.

마지막으로 관련 타입의 집합을 나타내는 프로토콜 전체에서 same-type relationships 를 사용하여 concrete types 간의 관계를 식별하고 보장하는 방법을 보았습니다. 

<img width="600" alt="40" src="https://user-images.githubusercontent.com/69136340/174798170-50172547-f3bd-4be4-b631-8d37ef674ada.png">
