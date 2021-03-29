---
title:  "Code Convention"
date:   2021-03-30 01:35:06 +0900
categories: projectsetting
---
# Code Convention

style share(https://github.com/StyleShare/swift-style-guide) 를 기초로 필요한 부분만 수정.

function naming 만 수정이 필요하다고 판단함.

### **서버통신**

서비스함수명 + WithAPI

### **IBAction**

동사원형 + 목적어
ex) touchBackButton

### **뷰 전환**

pop, push, present, dismiss

동사 + To + 목적지 뷰 (다음에 보일 뷰)

( dismiss는 dismiss + 현재 뷰 )

### **데이터 다루기?**

- 데이터 파싱 - parse + 모델 + 결과물

    parseDiaryUserID

### **초기세팅**

- init + 목적어

ex) initPickerView

### **hidden unhidden**

- show + 목적어
- hide + 목적어

### **뷰 UI 관련**

- 동사원형 + 목적어

### **뷰에 데이터 뿌리기**

- update + 목적어

ex) updatePickerView

### **네비게이션 바 관련**

- init, update

### **애니메이션**

- 동사원형 + 목적어 + WithAnimation
- showButtonsWithAnimation

### **register**

- register + 목적어
- registerXib

### **권한 위임**

- assign + Delegate / DataSource

### **subview로 붙이기**

- attatch

### **프로토콜**

- 뷰 이름 + View + Delegate

## MARK 주석

**// MARK: - Properties**

**// MARK: - @IBOutlet Properties**

**// MARK: - @IBAction Properties** 

**// MARK: - View Life Cycle** viewDidLoad(), viewWillAppear(_:) …

**// MARK: - Extensions**

**// MARK: - UITableViewDataSource** 

**// MARK: - UITableViewDelegate** 프로토콜들 Extension 으로 빼기

// TODO: - 

// FIXME: -
