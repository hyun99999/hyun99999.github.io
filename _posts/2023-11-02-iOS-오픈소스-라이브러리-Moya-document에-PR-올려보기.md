---
title:  "iOS) 오픈 소스 라이브러리(Moya) document 에 PR 올려보기"
categories:
- iOS

date:   2023-11-02  12:01:00 +0900
author_profile: false
---
### 내용

- Moya 리드미를 수정하여 기여해보자
- Moya contributing guideline 을 살펴보자
- github contribution guidelines 만드는 방법에 대해서 알아보자

## 👉 들어가기 전

오픈소스 라이브러리의 컨트리뷰터가 되는 것은 어려운 이야기처럼 들렸습니다.(성공한 후기가 아니기에 아직도 어렵습니다 ㅎㅎ)

또한, 꾸준히 해당 관련 깃허브 활동이나 컨트리뷰터들의 커뮤니티 활동에 관심을 가져야하는 것을 느껴졌습니다. 지금까지의 이슈나 PR 이 어떻게 적혀왔고, 어떤 부분들이 리뷰를 받아왔는지 저는 거꾸로 읽어갔던 것 같습니다.

PR을 올리게 되어 많은 사람들이 올바르지 않을 수 있는 나의 의견을 조회할 수 있다는 것이니까 좀 무섭기도 했지만, 마음 한켠으로는 별 상관이 없기도 했습니다.

그러다가 우연한 기회로 PR을 올릴 기회가 생겼고 그 과정을 공유해볼까 합니다.

## 👉 시작

해당 리드미를 edit 하게되면 자동으로 fork 되면서 마크업으로 구성된 글을 볼 수 있습니다.

해당 리드미는 6년 전에 마지막으로 커밋되었네요 ㄷㄷ

<img width="200" alt="1" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/916acaf5-1a14-4be1-a04a-d51bb1719fe4">

저는 리드미 한 부분만 수정하면 되서 깃허브 웹에서 진행하였고, 만약에 코드나 여러부분을 수정해야한다면 fork 를 해서 해당 레포지토리에서 작업 후에 PR 을 생성하면 되겠습니다.

## 👉 Issue, Pull Request 생성

Issue 을 만들자 다음과 같은 템플릿이 작성되어 있었습니다.

```swift
<!--
Please let us know what version of Moya you are using, so we can better pinpoint and/or solve your issue.

Please wrap code blocks in backticks, like so:

```swift
*your code goes here*
```

The code will automatically get its syntax highlighted, and doesn't need to be indented 4 spaces to be shown as code.

When referencing a dependency manager-related issue (think CocoaPods, Carthage, SwiftPM), please add its configuration file and version to the issue.
It would be helpful to put the contents in a code block too, using ```ruby for CocoaPods and ```swift for SwiftPM.

Also please make sure your title describes your problem well. Questions end with a question mark.
-->
```

- 이슈를 작성하자마자 bot 이 label 을 두 개 붙여주었습니다..!

<img width="400" alt="2" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/2d9deb41-661d-4e86-98a9-d27d075d50f3">

PR 을 만들자 다음과 같이 템플릿이 있었습니다.

```swift
// commit 세부사항을 적은 것이 그대로 나오고, 이하에 주석 처리된 안내사항이 적혀있습니다.
- fix example code.

<!--
Thank you for contributing to Moya! 🙌

Choosing a base branch:

  master: bug fixes, non breaking API changes, documentation fixes
  development: breaking changes, features for the next version

If your pull request fixes an issue, please reference the issue.
For example, when your pull request fixes issue 10, add the following line:

Fixes #10

This will make sure that when the pull request is merged, the issue will automatically be closed.

-->
```

간략하게 요약해서 어떻게 PR 을 작성할 수 있는지 정리해보겠습니다.

- 리드미에 적힌 코드를 수정하는 것이기 때문에 **`master`** 브랜치를 base branch 로 설정하였습니다.
- 또한, 이슈가 있다면 `**Fixes #10**` 와 같이 줄을 추가해서 pr 이 merge 된 후에 자동으로 닫히도록 유도하였습니다.

또한, 다음과 같이 풀리퀘스트를 작성하는데 contributing guidelines 를 확인할 수 있도록 안내하고 있었습니다. (첨보네요!)

<img width="400" alt="3" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/216c4479-bc73-4ffa-9a3d-b90a71c57536">

[Moya Contributing](https://github.com/Moya/Moya/blob/963654cd4f82d17d7546b9d6127c29f6df091717/Contributing.md)

**간략하게 요약해보겠습니다.**

`**master**` 와 `**development**` 브랜치를 유지보수 하고 있습니다.

- master 브랜치는 현재 release 작업을 위한 것으로 버그 수정이나 문서 철자 수정은 이 브랜치에서 머지되어야 합니다.
- development 브랜치는 다음 release 작업을 위한 것으로 API 변경과 관련 문서 업데이트에 해당합니다.

pull request 가 문제에 대한 잠재적인 수정이 되는 것이 일반적이고, 거기에서 프로젝트를 돕는 것은 어려울 수 있습니다. 그래서 모든 프로젝트의 토론을 깃허브 이슈에서 하는 것을 목표로 합니다.

그리고 contributor 로서 우리에게 기대하는 바가 적혀있습니다.

“오픈 소스에 기여하지 못한다고 해서 나쁘게 생각하지 마세요.” 기여하는 것은 의무사항이 아니라고 언급합니다.

이 외에도 공개적인 이슈에서 말할 수 없는 문제를 컨텍할 수 있도록 이메일 등을 알려주면서 문서는 마무리 됩니다.

## ✅ contribution guidelines

이 기능은 다음 문서를 참조해서 만들 수 있습니다.

[Setting guidelines for repository contributors - GitHub Docs](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/setting-guidelines-for-repository-contributors)

**간략하게 요약해보겠습니다.**

project contributors 가 좋은 작업을 할 수 있도록 **project repository’s root, docs, or .github folder** 에 contribution guidelines 를 추가할 수 있습니다.

누군가 pull request 또는 issue 를 만들 때, 해당 파일에 대한 링크를 볼 수 있고, repository’s `contribute` page 에서도 볼 수 있습니다.(https://github.com/Moya/moya/contribute) contribute 페이지 예시는 [github/docs/contribute](https://github.com/github/docs/contribute) 에서 볼 수 있습니다.

이런 가이드를 통해서 어떻게 contribute 해야하는지 전달할 수 있습니다.

- 이슈를 만들 때 우측하단에 **Contributing** 으로 확인할 수 있었습니다.

<img width="300" alt="4" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/dbe7d68b-e7d0-4517-9751-7724b0802b6f">

**Adding a CONTRIBUTING file**

- 깃허브에서 다음과 같이 파일을 추가할 수 있습니다.

<img src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/7cb87b87-d1df-4025-bb4f-e0544a34885a" width ="700">

- 이때 파일명은 대소문자를 구분하지 않습니다.
- project repository’s root, docs, or .github 디렉토리에 추가하면 됩니다. 이를 위해서 다음과 같이 할 수 있습니다.
    - root 에 두기 위해 **CONTRIBUTING**, docs 디렉토리에 두기 위해 **docs/CONTRIBUTING**
    - 두 개 이상의 CONTRIBUTING 파일이 있다면 링크에 표시되는 파일은 .github, root, docs directory 순으로 선택

---

## 👉 생성한 PR 살펴보자

제가 수정한 변경사항은 다음과 같습니다.

<img width="600" alt="6" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/a52618f3-c838-4f74-aaaa-955fa8bf5b69">

예시를 보고 적용하던 중에 해당 코드를 따라가다 보니 error 대신 failure 가 만들어졌습니다. 이는 둘째치고, 왜 이 코드 예시가 잘못되었는지 알아보겠습니다.

- Moya 에서 RxSwift 를 사용하여 손쉽게 서버통신의 결과물을 사용할 수 있습니다.
- 이때, **provider.rx.request()** 메소드를 통해 MoyaProvider 가 제공하는 `**request()**` 의 반환형은 다음과 같이 trait 의 하나인 **Single<Response> 입니다.**

<img width="400" alt="7" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/841f46ed-1f2c-4340-abc5-3763b52583a0">

- 이를 구독하기 위한 **subscribe(_:)** 구현부를 살펴보게되면 Single 에서 방출하는 next, error 이벤트가  success 와 failure 로 이어집니다.

<img width="500" alt="8" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/9d98f3ee-92d0-4384-b3f7-470f571a0d99">

- 이 외에도 Single.swift 파일에서 deprecated 된 메소드를 확인해보면 onError 를 통해 다룬 흔적을 확인할 수 있었습니다. 이를 onFailure 로 바꾼 것도 보이는데 그래서 6년 전의 커밋이 마지막이었던 리드미의 예시 코드가 .error 를 다룬 것이 아닌가 생각했습니다.

<img width="700" alt="9" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/1a6aed30-702c-4687-adfb-82d6593c3057">

### 그래서 다음과 같이 PR 을 올렸습니다.

2023-10-09 일자로 PR 을 올렸습니다.

https://github.com/Moya/Moya/pull/2322

<img src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/1d173975-ad84-4516-837a-c939ff9420b1" width="600">

## 👉 궁금한 점

- PR 을 리뷰해주는 시간이 얼마나 걸릴지 궁금했습니다.
- 리드미이기 때문에 다소 중요성이 갈릴 수도 있다고 생각했습니다. 이를 어떻게 Moya가 다뤄줄지 궁금했습니다.
(리드미의 양식이라던지 업데이트의 경우도 빠르게 PR 확인해주었는데 document 관련해서 머지된 PR이 2020년이 마지막이더라구요.. ㅎㅎ)
- bot 을 사용해서 답을 해주거나 빌드 확인을 해주는 오픈소스 라이브러리도 본 경험이 있는데 어떻게 대응해줄지 궁금했습니다.

현재, 이슈와 PR만 올려뒀기 때문에 다음 정도만 알 수 있었습니다.

- 이슈에서는 bot 이 label 을 붙여주기도 하였습니다.
- PR에서는 **circleci** 툴을 사용해서 유지관리하고 있었고, 가장 최근의 approved 된 코드리뷰는 2023년 4월이었습니다.
- 한 달 정도가 지난 지금까지 별도의 리뷰는 받지 못했습니다.

## 👉 느낀 점

다음에는 리드미가 아닌 코드적인 부분에서도 수정을 하는데 기여하면 좋겠다고 생각했습니다.(사실 기여만해도 좋겠다고 생각했습니다 ㅜㅜ 처음이 어려울 것 같았거든요.)

**contributing guidelines** 를 설정할 수 있는 기능은 오픈소스 라이브러리를 만들게되면 유용한 기능이라고 느꼈습니다.

대형 오픈소스 라이브러리의 guildelines 을 읽어볼 수 있는 기회가 되서 어떻게 유지 보수하는지 엿볼 수 있었습니다.
