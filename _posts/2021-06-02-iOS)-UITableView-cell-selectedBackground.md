---
title:  "iOS) UITableView cell selectedBackground"
categories:
- iOS

date:   2021-06-02 13:47:00 +0900
author_profile: false
---
### selectedBackgroundView 와 selectionStyle 을 활용해서 cell 이 선택되었을때 불필요하게 변하는 회색 배경을 없앨 수 있다.

```swift
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    guard let cell = tableView.dequeueReusableCell(withIdentifier: "StoryTVC") as? StoryTVC else {
        return UITableViewCell()
    }
// 방법 1
//        let backgroundCell = UIView()
//        backgroundCell.backgroundColor = .white
//        cell.selectedBackgroundView = backgroundCell
// 방법2
    cell.selectionStyle = .none
    
    return cell
}
```
### 추가적으로 cell 을 터치할 때 cell 의 label 의 속성을 바꾸고 싶다면 cell 의 setHighlighted 에서 설정하면 된다.

- selected 상태는 cell 이 선택된 상태. 다른 셀을 선택하기전까지는 바뀌지 않음.
- highlighted 상태는 cell 을 선택할 때 터치할 때 상태.

```siwft
import UIKit

class StoryTVC: UITableViewCell {

    // MARK: - @IBOutlet Properties
    @IBOutlet weak var titleLabel: UILabel!
    @IBOutlet weak var dateLabel: UILabel!
    
    override func awakeFromNib() {
        super.awakeFromNib()
        
        //highlight된 상태에서의 글자 color설정
        //selected되었을 때도 계속 적용됩니다.
        textLabel?.highlightedTextColor = .red
    }
    
    /*cell이 재사용될 때도 호출됩니다. 
    tableView가 가지고 있는 selected indexPath와
    cell이 표시될 indexPath를 비교하고 선택상태롤 자동으로 설정하기 때문에 
    재사용 여부와 관계없이 항상 정확한 상태를 표시합니다.*/
    override func setSelected(_ selected: Bool, animated: Bool) {
        super.setSelected(selected, animated: animated)
        //선택상태가 바뀔 때마다 호출됩니다.
        /*selected 파라미터로 조건문을 걸어 
        didSelected 혹은 didDeseleted되었을 때 
        실행될 코드를 작성할 수 있습니다.*/
        accessoryType = selected ? .checkmark : .none
    }
    
    override func setHighlighted(_ highlighted: Bool, animated: Bool) {
        super.setHighlighted(highlighted, animated: animated)
        //highlighted 상태가 바뀔 때마다 호출됩니다.
        /*highlighted 파라미터로 조건문을 만들어
        didHighlighted 혹은 didUnhighlighted되었을 때 
        실행될 코드를 작성할 수 있습니다.*/
    }
}
```
