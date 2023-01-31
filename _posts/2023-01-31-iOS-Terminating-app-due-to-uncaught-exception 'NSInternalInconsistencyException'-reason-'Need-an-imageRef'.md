---
title:  "iOS) Terminating app due to uncaught exception 'NSInternalInconsistencyException', reason: 'Need an imageRef'"
categories:
- iOS

date:   2023-01-31  16:48:00 +0900
author_profile: false
---
### 💫 상황

- 프로젝트를 진행하면서 전혀 작업하지 않은 부분에서 앱이 종료되는 것을 경험하였습니다.
- `** Terminating app due to uncaught exception 'NSInternalInconsistencyException', reason: 'Need an imageRef'` 에러와 함께 앱이 종료되었습니다.

<img width="329" alt="1" src="https://user-images.githubusercontent.com/69136340/215697821-a9ae7da5-cb40-44b8-9053-12d301d43b1e.png">

- 해당 뷰에서는 작업조차하지 않았기 때문에 확인을 못했었고 어디서부터 잘못되었는지 커밋 히스토리를 따라가보았습니다.
    - **Card 라는 Color 에셋을 추가한 커밋부터 해당 에러가 발생하는 것을 확인하였습니다.**

### 🚨 해결

아래를 참고하여 해결하였습니다.

[Loading image from xcassets causes assertion failure](https://stackoverflow.com/questions/60317240/loading-image-from-xcassets-causes-assertion-failure)

- 강제종료가 되는 해당 뷰의 컬렉션 뷰 셀에서 프로젝트에서는 삭제된 **Card** 라는 이미지 에셋을 사용하고 있었고, 이것이 **컬러 에셋과 중복**되면서 발생한 문제였습니다.
- **이미지 뷰에서 Card 라는 이미지 에셋을 삭제하여 해결하였습니다.**
    
<img width="700" alt="2" src="https://user-images.githubusercontent.com/69136340/215697916-948db7dd-2f2c-4e2d-ab64-ba92abeb4e04.png">
