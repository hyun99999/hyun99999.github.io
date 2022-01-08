---
title:  "git forking workflow 적용"
categories:
- iOS

date:   2022-01-08  12:16:00 +0900
author_profile: false
---
# git forking workflow 적용

1. 팀 프로젝트 레포 포크한다.(이하 팀 레포)
2. 포크한 개인 레포(이하 개인 레포)를 클론한다.
3. 개인 레포에서 작업하고 개인 레포의 원격저장소로 푸시한다.
4. 풀리퀘스트를 통해서 팀 레포로 머지한다.
5. 풀받아야 할때는 팀 레포에서 풀 받는다.

<img src="https://user-images.githubusercontent.com/69136340/148629689-08849227-c654-4c0e-bc7a-614552e89bd2.png" width ="800">

- 위와 같이 `팀 레포`에서 포크를 한 `개인 레포`에 푸시하게 되면 feature/#16 브랜치는 팀 레포에 브랜치가 남지 않아서 지저분하지 않습니당!

<img width="500" alt="2" src="https://user-images.githubusercontent.com/69136340/148629690-3494d326-2818-4800-a72a-adfb2ddffecc.png">

- 포크한 개인레포에서 풀리퀘를 만들게 되면 위 처럼 `개인:브랜치` 와 같이 표시가 됩니당!
- 다른 개발자들의 코드를 리뷰해줄때나 코드를 보고 싶을 때 팀 레포 를 포크한 개인의 레포를 원격 저장소로 등록하고 풀 받으면 됩니다.
