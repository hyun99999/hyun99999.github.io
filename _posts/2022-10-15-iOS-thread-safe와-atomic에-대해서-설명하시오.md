---
title:  "iOS) thread-safe 와 atomic 에 대해서 설명하시오."
categories:
- iOS

date:   2022-10-10  01:11:00 +0900
author_profile: false
---
thread-safe : 멀티스레드 프로그래밍에서 자원에 스레드가 동시에 접근해도 문제가 생기지 않는 것을 말한다. 즉, 여러 곳에서 접근하더라도 올바른 결과를 얻게된다.
atomic : 멀티스레드 프로그래밍에서 데이터의 변경 전과 후에만 자원에 접근할 수 있음을 보장하는 것이다. 즉, 데이터가 변경되고 있는 중에는 접근이 불가능하다.

Swift 는 멀티스레딩(Multi-Threading) 방식입니다.
멀티스레드는 stack 을 제외한 heap, data, code 영역을 공유합니다. 그래서 한 스레드에서 영역을 사용할 때 다른 스레드에서 접근하게 되면 동일한 자원에 두 개 이상의 스레드가 접근하는 경우가 생깁니다. Swift 는 thread-safe 를 보장하는 언어가 아니기 때문에 이때 문제가 생길 수 있습니다. 그래서 프로퍼티가 non-atomic 입니다.
Obejctive-C 에서는 non-atomic 과 atomic 을 설정할 수 있습니다.(non-atomic 기본) atomic 으로 설정한 프로퍼티에 접근할 때면 getter 와 setter 에 lock 기능을 제공하게 됩니다.
멀티스레드에서 접근할 이유가 없는 property(view property 같은)를 nonatomic 키워드로 설정하는 것이 좋다.

atomic 이 안전하지만 그만큼 성능 저하가 발생한다. 그래서 꼭 필요한 경우가 아니라면 non-atomic 으로 설정해두는 것이 좋다.
Swift 에서는 별도의 atomic 을 지정할 수 없고, GCD 로 구현할 수 있다.
