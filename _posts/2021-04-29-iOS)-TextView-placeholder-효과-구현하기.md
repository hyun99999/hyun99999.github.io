---
title:  "iOS) TextView placeholder 효과 구현하기"
categories:
- iOS

date:   2021-04-29 13:12:00 +0900
author_profile: false
---
TextView placeholder 효과 구현하기

### UITextFieldDelegate

TextField 는 placehoder 기능을 지원하지만 여러줄을 입력받는 TextView 는 그렇지 않다. 그래서 TextViewDelegate 를 통해서 커스텀해주어야 한다.


```swift
private func setTextViewPlaceholder() {
    if textView.text == "" {
        textView.text = "메모"
        textView.textColor = UIColor.lightGray
    } else if textView.text == "메모"{
        textView.text = ""
        textView.textColor = UIColor.black
    }
}

...

extension ScheduleListTopCell: UITextViewDelegate {

    //textView 편집 시작
    
    func textViewDidBeginEditing(_ textView: UITextView) {
        setTextViewPlaceholder()
    }
    
    //textView 편집 끝
    
    func textViewDidEndEditing(_ textView: UITextView) {
        if textView.text == "" {
            setTextViewPlaceholder()
        }
    }
    
    //textView 특정 text 가 대체될 때 호출
    //개행문자 시 textView 의 활성화를 포기하는 요청을 보내서 키보드를 내림
    
    func textView(_ textView: UITextView, shouldChangeTextIn range: NSRange, replacementText text: String) -> Bool {
        if text == "\n" {
            textView.resignFirstResponder()
        }
        return true
    }
}
```
<img src ="https://user-images.githubusercontent.com/69136340/116504226-cb391580-a8f2-11eb-9b45-ce5725cd6b40.png" width ="250">

하지만 TextView 의 기본여백때문에 높이와 시작점이 다른 것을 볼 수 있다.

### 원인

UITextView 에는 textContainerInset 속성이 있어서 여백을 관리할 수 있다. 개발자문서에서 확인해보니 상단과 왼쪽에 8 이 주어져있다.

<img src = "https://user-images.githubusercontent.com/69136340/116504650-c759c300-a8f3-11eb-8331-f60a2195eaf8.png" widht ="500">

### 해결

아래의 코드로 여백을 0으로 설정해주었다.

```swift
textView.textContainerInset = .zero
//textView.textContainerInset = .init(top: 8, left: -5, bottom: 0, right: 0)
```

<img src ="https://user-images.githubusercontent.com/69136340/116505642-5bc52500-a8f6-11eb-811f-e05837d70072.png" width ="250">

### 출처
출처ㅣhttps://gigas-blog.tistory.com/7

