---
title:  "iOS) 인앱에서 mail 보내기 - MFMailComposeViewController"
categories:
- iOS

date:   2022-03-13  13:37:00 +0900
author_profile: false
---
```swift
if MFMailComposeViewController.canSendMail() {
    let mailComposeVC = MFMailComposeViewController()
    mailComposeVC.mailComposeDelegate = self
            
    mailComposeVC.setToRecipients(["teamsparker66@gmail.com"])
    mailComposeVC.setSubject("스파크 문의 사항")
    mailComposeVC.setMessageBody("문의 사항을 상세히 입력해주세요.",
                                             isHTML: false)
    mailComposeVC.modalPresentationStyle = .overFullScreen
    present(mailComposeVC, animated: true, completion: nil)
} else {
    // ✅ 메일이 계정과 연동되지 않은 경우.
}

// MARK: - MFMailComposeViewControllerDelegate
extension MypageVC: MFMailComposeViewControllerDelegate {
    func mailComposeController(_ controller: MFMailComposeViewController, didFinishWith result: MFMailComposeResult, error: Error?) {
        // ✅ MFMailComposeResult 가지고 메일 작성 인터페이스가 닫힐때의 결과에 대응할 수있다.
        switch result {
        case .cancelled:
// ✅ The user canceled the operation.
            print("cancelled")
        case .saved:
// ✅ The email message was saved in the user’s drafts folder.
            print("saved")
        case .sent:
// ✅ The email message was queued in the user’s outbox.
            print("sent")
        case .failed:
// ✅ The email message was not saved or queued, possibly due to an error.
            print("failed")
        }
// ✅ Dismiss the mail compose view controller.
        controller.dismiss(animated: true, completion: nil)
    }
}
```

출처:

[[iOS] Swift를 이용해서 iOS 디바이스에서 Email 보내기 📧](https://borabong.tistory.com/6)

[Apple Developer Documentation](https://developer.apple.com/documentation/messageui/mfmailcomposeviewcontroller)
