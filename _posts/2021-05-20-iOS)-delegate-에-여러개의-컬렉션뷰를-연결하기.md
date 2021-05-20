---
title:  "iOS) delegate 에 여러개의 컬렉션뷰를 연결하기"
categories:
- iOS

date:   2021-05-20 20:24:00 +0900
author_profile: false
---
### delegate 에 여러개의 컬렉션뷰를 연결하기

컬렉션뷰를 구분해서 각각 다르게 설정해주고 싶었다. 총 두가지 방법이 있다.

1. `collectionView == ...` 과 같이 내가 원하는 뷰일때 라는 조건문을 사용한다.

```swift
func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
    if collectionView == bandCollectionView {
        guard let bandCell = bandCollectionView.dequeueReusableCell(withReuseIdentifier: "BandCollectionViewCell", for: indexPath) as? BandCollectionViewCell else {
            return UICollectionViewCell()
        }
        
        return bandCell
    } else if collectionView == missionCollectionView {
        guard let missionCell = missionCollectionView.dequeueReusableCell(withReuseIdentifier: "MissionCollectionViewCell", for: indexPath) as? MissionCollectionViewCell else {
            return UICollectionViewCell()
        }
        
        return missionCell
    } else if collectionView == pageCollectionView {
        guard let pageCell = pageCollectionView.dequeueReusableCell(withReuseIdentifier: "PageCollectionViewCell", for: indexPath) as? PageCollectionViewCell else {
            return UICollectionViewCell()
        }
        pageCell.initializeData(pageList[indexPath.row].pageImage, pageList[indexPath.row].title, pageList[indexPath.row].detail, pageList[indexPath.row].subscribe)
        
        return pageCell
    } else {
        guard let topicCell = topicCollectionView.dequeueReusableCell(withReuseIdentifier: "TopicCollectionViewCell", for: indexPath) as? TopicCollectionViewCell else {
            return UICollectionViewCell()
        }
        topicCell.initializeData(topicList[indexPath.row])
        
        return topicCell
    }
}
```

2. `tag` 를 사용한다.

```swift
override func viewDedLoad() {
    super.viewDidLoad()
    //...
    bandCollectionView.tag = 1
    bandCollectionView.tag = 2
    bandCollectionView.tag = 3
    bandCollectionView.tag = 4
}
```
```swift
func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
     if collectionView.tag = 1 {
     //...
     }else  if collectionView.tag = 2 {
     //...
     }else  if collectionView.tag = 3 {
     //...
     } else {
     //...
     }
}
```


### 총 4개의 커스텀 컬렉션 뷰를 설정
<p>
<img src = "https://user-images.githubusercontent.com/69136340/118971788-ca077f80-b9aa-11eb-8c06-48c9313d2fce.png" width = "250">

<img src = "https://user-images.githubusercontent.com/69136340/118971209-18684e80-b9aa-11eb-8ff4-e55064902d17.png" width ="250">
</p>

### 출처
출처ㅣ[https://zeddios.tistory.com/169](https://zeddios.tistory.com/169)
