---
title:  "Commit,Issue,PR"
date:   2021-03-30 01:30:06 +0900
categories: projectsetting
---
# Commit,Issue,PR
## Commit Type

- feat: 새로운 기능 추가 (new feature)
- fix: 버그 수정 (bug fix)
- docs: 문서 작성, 수정 (documentation)
- style: 코드 포맷팅, 세미콜론 누락 등 코드 변경이 없는 경우
- refactor: 코드 리팩토링 (refactoring)
- test: 테스트 코드, 리팩토링 테스트 코드 추가
- chore: 빌드 업무 수정, 패키지 매니저 수정 등 (production code 변경이 없는 경우)
- perf: 성능 개선

## issue title
`[Commit Type] 이슈 제목`

Commit type label 과 개발자 각자의 label 추가하기로 함.

issue body 는 checklist type 으로 작성하기로 함.

## commit message Description
```swift
[#이슈번호] 해당 커밋 요약

## Description
- 커밋 상세내용 1
- 커밋 상세내용 2

close #이슈번호
```

## Pull Request
- title
`[#이슈번호] 커밋 내용 요약`

- body
```swift
(커밋이 한개일 때는 그냥 description 내용 그대로 놔둬도 됨)

## Description
- 커밋 상세내용 1
- 커밋 상세내용 2
```

### 하나의 이슈에 하나의 브랜치를 만들고 하나의 커밋으로 하나의 풀리퀘스트...

### 출처
출처ㅣhttps://udacity.github.io/git-styleguide/
