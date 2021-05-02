---
title:  "iOS) UITalbeView editing mode 에서 multiple selection 구현"
categories:
- iOS

date:   2021-05-02 00:16:00 +0900
author_profile: false
---
UITalbeView editingStyle allowsMultipleSelectionDuringEditing

# allowsMultipleSelectionDuringEditing

<img src ="https://user-images.githubusercontent.com/69136340/116802527-6f6ac880-ab4e-11eb-8dce-35de0c14a0d0.png" width ="250">

- editing mode 에서 체크마크 속서이 없어서 구현에 어려움을 느꼈다. 하지만 allowsMultipleSelectionDuringEditing 속성을 통해서 해결했다.
- 이렇게 선택된 row 들의 정보는 아래의 속성을 통해서 리턴 가능하다.
- `tableView.indexPathForSelectedRow` 현재 선택되어있는 cell의 index를 return
- `tableView.indexPathsForSelectedRows` tableView가 선택된 cell들의 index를 배열로 return합니다.(multiple selection이 가능할 때 사용)

    → 출력해보면 `[[0,0],[0,1]]` 처럼 section 과 row 를 보여준다.

그래서 나는 cell 이 선택될 때 출력해보도록했다.

```swift
func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        print("didSelectRowAt")
        print(tableView.indexPathsForSelectedRows!)
    }
```

1. 우선 테이블뷰에 설정을 해주어야합니다. 스토리보드로 진행해도 되고 코드로 작성해도 됩니다.
- storyboard 진행

<img src ="https://user-images.githubusercontent.com/69136340/116802528-709bf580-ab4e-11eb-971e-f4796387a804.png" width ="250">

- programmatically 진행

```swift
tableView.allowsMultipleSelectionDuringEditing = true
```

2. cell 의 selectionStyle 을 설정

tableView(tableView:cellForRowAt indexPath:) 메서드를 통해서 selectionStyle 을 설정해주면 된다.

```swift
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        guard  let cell = tableView.dequeueReusableCell(withIdentifier: ListCell.identifier) as? ListCell else {
            return UITableViewCell()
        }
//기본값이 .none 이 되어있기때문에 설정.
//옵션에 .blue .gray .default 가 있는데 다 같은 효과를 보여주고 있다.
        cell.selectionStyle = .blue

        return cell
    }
```

<img src ="https://user-images.githubusercontent.com/69136340/116802530-71cd2280-ab4e-11eb-829c-3330ac9b3c68.png" width ="250">

- 위의 코드를 입력하지 않으면 이런식으로 눌러도 체크가 되지 않는다. 하지만, `tableView(tableView:didSelectRowAt:)` 메서드로 로그를 찍어보니까 처음 선택했을 때 찍히고 두번째 선택하면 찍히지 않는다. 선택이 해제가 된다는 의미이다.
- 즉, 선택은 되는데 보이지만 않는 것이다. `selectionStyle = .none` 인 상태이다.

# 해결

<img src ="https://user-images.githubusercontent.com/69136340/116802531-7265b900-ab4e-11eb-8064-cdd6182091d5.png" width ="250">

---
### 출처

출처ㅣ[stackoverflow - Cells not getting selected in UITableView with allowsMultipleSelectionDuringEditing set in edit mode](https://stackoverflow.com/questions/12708186/cells-not-getting-selected-in-uitableview-with-allowsmultipleselectionduringedit/12710236)
