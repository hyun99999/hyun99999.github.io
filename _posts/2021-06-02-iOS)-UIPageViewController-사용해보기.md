---
title:  "iOS) UIPageViewController 사용해보기"
categories:
- iOS

date:   2021-05-27 23:44:00 +0900
author_profile: false
---
### UIPageViewController

```swift
import UIKit

class MainPageVC: UIPageViewController {

    // MARK: - Properties
    private var currentIndex = 0
    
    lazy var vcArray: [UIViewController] = {
        return [self.vcInstance(name: "ViewController"),
                self.vcInstance(name: "AddVC")]
    }()
    
    private func vcInstance(name: String) -> UIViewController {
        return UIStoryboard(name: "Main", bundle: nil).instantiateViewController(withIdentifier: name)
    }
    
    // MARK: - @IBOutlet Properties
    
    //MARK: - View Life Cycle
    override func viewDidLoad() {
        super.viewDidLoad()

//        self.transitionStyle = .scroll
        
        self.delegate = self
        self.dataSource = self
        if let firstVC = vcArray.first {
            setViewControllers([firstVC], direction: .forward, animated: true, completion: nil)
        }
    }
}

// MARK: - UIPageViewControllerDataSource
extension MainPageVC: UIPageViewControllerDataSource {
    func pageViewController(_ pageViewController: UIPageViewController, viewControllerBefore viewController: UIViewController) -> UIViewController? {
        guard let viewControllerIndex = vcArray.firstIndex(of: viewController) else {
            return nil
        }
        let previousIndex = viewControllerIndex - 1
        
        guard previousIndex >= 0 else {
            return nil
        }
        
        return vcArray[previousIndex]
    }
    
    func pageViewController(_ pageViewController: UIPageViewController, viewControllerAfter viewController: UIViewController) -> UIViewController? {
        guard let viewControllerIndex = vcArray.firstIndex(of: viewController) else {
            return nil
        }
        let nextIndex = viewControllerIndex + 1
        
        guard nextIndex < vcArray.count else {
            return nil
        }
        
        return vcArray[nextIndex]
    }
}

// MARK: - UIPageViewControllerDelegate
extension MainPageVC: UIPageViewControllerDelegate {
    
}
```
