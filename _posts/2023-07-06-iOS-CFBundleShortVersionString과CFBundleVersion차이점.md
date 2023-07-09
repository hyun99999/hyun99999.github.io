 ---
title:  "iOS) CFBundleShortVersionString 과 CFBundleVersion 차이점"
categories:
- iOS

date:   2023-07-06  14:30:00 +0900
author_profile: false
---
상황

fastlane 으로 다음의 Info 파일이 수정되었습니다.
스크린샷 2023-07-06 오후 1 04 19

MARKETING_VERSION 환경변수에 영향을 받는 것이 상수로 수정되었습니다. 그리고 이 환경변수는 프로젝트의 버전으로 설정되고 있습니다.

프로젝트의 버전은 다음의 Info 파일에 영향을 주고 있었습니다.
스크린샷 2023-07-06 오후 1 05 43

그래서 Bundle version string 과 Bundle version 의 차이에 대해서 찾아보았습니다.
공식문서에서 찾아보니 다음과 같았습니다.

스크린샷 2023-07-06 오후 1 12 59스크린샷 2023-07-06 오후 1 12 42

https://developer.apple.com/documentation/bundleresources/information_property_list/cfbundleshortversionstring
https://developer.apple.com/documentation/bundleresources/information_property_list/cfbundleversion
CFBundleVersion 은 모든 반복되는 번들의 버전에 대해서 사용되고, CFBundleShortVersionString 은 릴리즈를 포함하여 버전을 나타냅니다.

릴리즈는 하지 않지만, 번들의 버전을 변경할때는 CFBundleVersion 을 사용하여 앱스토어에 올릴때 영향을 주지 않을 수 있는 것입니다.

결론

fastlane 으로 버전을 업뎃할 때는 MARKETING_VERSION 이라는 환경변수에 영향을 받다가 CFBundleShortVersionString 이 지정한 버전으로 수정됩니다.

이 MARKETING_VERSION 은 프로젝트의 버전에 영향을 받고 있기 때문에 기존에는 프로젝트의 버전만 수정해주어도 릴리즈를 위한 버전 역시 함께 적용되었던 것입니다.

결론적으로, 릴리즈 버전이 아닌 프로젝트 버전도 관리하기 위해서 CFBundleShortVersionString 즉, info 에서 Bundle version string 도 수정해주었습니다!
