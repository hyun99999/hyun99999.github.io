---
title:  "BDD(Behavior-driven development)"
categories:
- iOS

date:   2023-03-13  13:30:00 +0900
author_profile: false
---

[kotest가 있다면 TDD 묻고 BDD로 가!](https://tv.kakao.com/channel/3693125/cliplink/414004682)

Quick 이 BDD 를 지원하는 프레임워크이기 때문에 위의 if Kakao 세션을 시청하며 TDD 와 BDD 에 대해서 요약해보았습니다.

### TDD?

- Test-driven development

개발이 테스트 주도로 진행됨을 의미.

<img width="400" alt="1" src="https://user-images.githubusercontent.com/69136340/224607642-7fe5755b-c06a-451c-b188-9c55cd20204e.png">


테스트 케이스를 작성하고, 테스트를 돌려 실패한 코드를 수정하는 작업을 반복하는 것을 의미한다.

**테스트 케이스를 만들면 TDD 인가요?**

<img width="400" alt="2" src="https://user-images.githubusercontent.com/69136340/224607651-3b4d57cb-0a0b-4ef8-9e9f-b21c81685115.png">

- 전자를 TDD 라고 볼 수 있다. 테스트 케이스를 먼저 작성하고 설계하면 코드 설계가 테스트 가능하게 작성이 된다.

이렇게 되면 이후에도 테스트 가능한 코드가 유지될 수 있다.

**Testable 한 코드의 장점은?**

- 테스트 하기 쉽게 만들어진 코드다!

테스트할 모듈의 역할이 명확해야 함. 이를 위해 모듈을 단순화하게 유도하고, 모듈 또는 계층간의 coupling 도 적게 만들어 유지보수와 확장에 장점이 생김.

**TDD 가 잘이루어지지 않는 이유는?**

- 지속적인 테스트 케이스 유지보수가 필요한데 일정과 리소스 관리의 측면에서 부담이 크다.
- 필요성에 비해서 비용이 크다.
    - 변수명을 정하고, 알맞는 테스트 케이스를 설계하는 것 또한 힘듦.

**이 때문에 테스트 케이스를 상상해서 만드는 것 보다 이미 작성된 요구 사항과 기획서가 테스트 케이스가 된다면?**

- 이것이 바로 BDD 입니다.

<img width="350" alt="3" src="https://user-images.githubusercontent.com/69136340/224607713-06917e19-0583-4a31-bdf5-c2b7dcfc55ca.png">

테스트 케이스의 작성의 어려움을 이미 정의된 사용자 행위로 작성하면 됩니다.

### BDD?

- Behavior-driven development

<img width="450" alt="4" src="https://user-images.githubusercontent.com/69136340/224607726-475a5eef-ca92-40d7-a643-1a1bf4326523.png">

자연스레 BDD로 서비스에 대한 이해도도 높아짐.

**이는 TDD 로도 가능할거 같은데 굳이 BDD 인 이유는? 그렇다면 둘은 뭐가 다를까?**

- 테스트 케이스 작성 중  목적에 대한 관점이 차이점이 있다.

<img width="400" alt="5" src="https://user-images.githubusercontent.com/69136340/224607952-4195e29e-abcc-4230-9a8b-14b643e83933.png">

- 테스트할 모듈의 기능을 확인할 목적
- 시나리오 주체를 기준으로 행위를 확인하기 위한 목적

<img width="500" alt="6" src="https://user-images.githubusercontent.com/69136340/224607974-523fbc59-6256-438c-8522-5cf55342f809.png">

BDD 는 TDD 관점에서 보기 어려운 유저 시나리오의 흐름을 확인할 수 있다.

BDD 의 테스트 케이스로 시나리오 검증을 하고, 시나리오에서 사용되는 각 모듈들은 TDD 의 테스트 케이스로 검증해야 기대하는 테스트 커버리지를 가질 수 있습니다.

서로가 상호보완적인 관계이다.

예를 들어, 계산기에서 add 기능에 대해서는 문제가 없을 수 있지만, BDD 의 테스트 케이스인 결과값을 화면에 표시해주는데는 실패할 수 있다.

**BDD 의 또 다른 장점은?**

- 테스트 케이스를 만들면서 예외적인 입력값에 대한 테스트도 추가하며 테스트 커버리지를 높여간 TDD 의 경험에 대해서 생각해보자.

이처럼 설계에서 입력값의 예외 사항을 놓치는 것을 이후에 발견할 수 있다. 그렇게 되면 기획 설계 단계를 거쳐 코드를 다시 작성하게 됩니다. 

혹은, 코드 작성 중에 기획이 변경되는 경험도 있다.

<img width="500" alt="7" src="https://user-images.githubusercontent.com/69136340/224608016-5eb69f57-73b1-4aed-a959-c5567a902256.png">

TDD 의 이런 리스크를 줄이기 위해서는 기획의 검토가 충분히 되어야 한다.

반면, BDD 에서는 입력값의 예외사항을 추가하듯이 기획 시나리오의 빈틈들을 테스트 케이스 작성시에 확인할 수 있는 장점이있다.

<img width="500" alt="8" src="https://user-images.githubusercontent.com/69136340/224608137-c3c77f2c-6164-4723-979a-7fcb4b622839.png">

BDD 에서 다음과 같이 주어진 조건이 누락된 것을 테스트 케이스에 서술된 스크립트로 발견할 수 있다.

<img width="600" alt="9" src="https://user-images.githubusercontent.com/69136340/224608064-475cbcd9-671b-435d-a8f8-1182422cf5a2.png">

- 인증 방식 이슈가 추가적으로 기획이 필요하다는 것을 알 수 있다.

이런 시나리오의 빈틈을 채우기 위해 일정 연기없이 추가하려다 누더기 코드를 만들어내는 원인이 될 수 있다.

**즉, 실제 설계가 이루어지기 전에 BDD 의 테스트 케이스 작성만으로도 시나리오의 검증과 그로인한 리스크 감소의 결과도 얻을 수 있다.**

이를 통해 더 충실한 시나리오가 만들어지고, 더 탄탄한 서비스 설계가 되는 선순환이 생긴다.

<img width="600" alt="10" src="https://user-images.githubusercontent.com/69136340/224608107-2f96b28e-f51e-4c75-8a6c-cd9ccf42f9ea.png">
