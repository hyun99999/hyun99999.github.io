---
title:  "WWDC21) Build a research and care app, part 2: Schedule tasks"
categories:
- iOS

date:   2022-06-07  22:28:00 +0900
author_profile: false
---
### WWDC21) ****Build a research and care app, part 2: Schedule tasks****

[Build a research and care app, part 2: Schedule tasks - WWDC21 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2021/10069/)

****본 글은 WWDC 를 보고, 번역 및 요약 그리고 실행해보는 스터디 프로젝트의 일환입니다.***

# 들어가기전에

ResearchKit 과 CareKit 에 대해서 더 많은 정보를 얻고 싶다면 아래의 소개글도 도움이 될 것입니다.

[ResearchKit과 CareKit](https://www.apple.com/kr/researchkit/)

part1 에서는 onboarding 과 consent 에 대해서 마쳤습니다.

🤦🏻‍♂️ Erick: oh, hang on. Jamie 로부터 메시지를 받은 것 같습니다.

“앱에 대한 새로운 아이디어를 얻었어요.”

“내 마지막 text 봤어요?”

와 같이 메시지를 받고, mail 과 Notes 의 알림을 Erick 이 받게됩니다. 

*발표가 참 기가 막히군요 크..*

<img width="700" alt="1" src="https://user-images.githubusercontent.com/69136340/172381595-aedaed42-e154-43c3-a3c8-10fdae282c9a.png">

자! 그럼 이번에는 무엇을 해야할지 봅시다.

<img width="700" alt="2" src="https://user-images.githubusercontent.com/69136340/172381647-6a8e964c-6c87-4834-9425-83f9208ccd25.png">

*Display forms, persisting some data, dynamic schedules, range of motion..*

- Ok. Jamie 가 참가자들에게 수면 시간과 고통에 대해 물어보는 daily check-in survey 를 원하는 것 같습니다.
- ResearchKit 의 form items 를 사용하여 한 페이지에 여러개의 질문을 넣는 방법을 보여드리겠습니다.
    
<img width="700" alt="3" src="https://user-images.githubusercontent.com/69136340/172381738-b2f56361-5774-4424-9655-58607f4ceec0.png">

- 그리고나서 ResearchKit survey 로부터 결과를 분석하여 CareKit 에 유지할 것입니다. 이것은 completion ring 을 채우고, card UI 가 업데이트되도록 합니다.
    
<img width="700" alt="4" src="https://user-images.githubusercontent.com/69136340/172381841-21f4dcf2-d1d4-4bc7-a693-df8412cf7d5a.png">
    
- 또한, CareKit 으로 고급 일정을 만드는 방법에 대해 살펴본 다음 ResearchKit 과 함께 일정 중 하나를 사용해서 참가자들에게 무릎의 range of motion(움직임 범위) 을 측정하도록 할 것입니다.
    
<img width="713" alt="5" src="https://user-images.githubusercontent.com/69136340/172381913-8126fe5b-b536-4348-96cd-a0e4d460b9a0.png">

<img width="700" alt="6" src="https://user-images.githubusercontent.com/69136340/172381896-ce360cc0-82d4-4b21-9fa5-aaa4a321ee62.png">

## daily check-in survey

**아래의 part1 에서 작성한 코드를 그대로 사용하겠습니다.*

- AppDelegate

```swift
import CareKit
import CareKitStore
import UIKit
import os.log

@main
class AppDelegate: UIResponder, UIApplicationDelegate {

    let storeManager = OCKSynchronizedStoreManager(
        wrapping: OCKStore(
            name: "com.apple.wwdc.carekitstore",
            type: .inMemory
        )
    )

    func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions
            launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

        seedTasks()

        return true
    }

    // MARK: UISceneSession Life Cycle

    func application(
        _ application: UIApplication,
        configurationForConnecting connectingSceneSession: UISceneSession,
        options: UIScene.ConnectionOptions) -> UISceneConfiguration {

        UISceneConfiguration(
            name: "Default Configuration",
            sessionRole: connectingSceneSession.role
        )
    }

    // MARK: Seeding the Store

    private func seedTasks() {

        let onboardSchedule = OCKSchedule.dailyAtTime(
            hour: 0, minutes: 0,
            start: Date(), end: nil,
            text: "Task Due!",
            duration: .allDay
        )

        var onboardTask = OCKTask(
            id: TaskIDs.onboarding,
            title: "Onboard",
            carePlanUUID: nil,
            schedule: onboardSchedule
        )
        onboardTask.instructions = "You'll need to agree to some terms and conditions before we get started!"
        onboardTask.impactsAdherence = false

        // 2.1 Add a check-in task

        // 2.6 Add a range of motion task

        storeManager.store.addAnyTasks(
            [onboardTask],
            callbackQueue: .main) { result in

            switch result {

            case let .success(tasks):
                Logger.store.info("Seeded \(tasks.count) tasks")
                
            case let .failure(error):
                Logger.store.warning("Failed to seed tasks: \(error as NSError)")
            }
        }
    }
}
```

- onboarding 작업에서 했던 것처럼 schedule 과 CareKit task 를 정의하는 것으로 시작합니다. 매일 check-in 해야하니까 아침 8시에 하도록 하겠습니다.

```swift
// 2.1 Add a check-in task
        
        let checkInSchedule = OCKSchedule.dailyAtTime(
            hour: 8, minutes: 0,
            start: Date(), end: nil,
            text: nil
        )

        // 🔥 고유한 식별자와 방금 정의한 일정으로 check-in task 를 만듭니다.
        let checkInTask = OCKTask(
            id: TaskIDs.checkIn,
            title: "Check In",
            carePlanUUID: nil,
            schedule: checkInSchedule
        )

        // 2.6 Add a range of motion task

        storeManager.store.addAnyTasks(
            // 🔥 유지하기위해서 여기에 task 를 넣습니다.
            [onboardTask, checkInTask],
            callbackQueue: .main) { result in
```

온보딩 작업과 마찬가지로 다음 단계는 CareFeedViewController 로 이동해서 CareKit 에 task 를 표시하는 방법을 알려주는 것입니다.

- CardFeedViewContorller

```swift
import CareKit
import CareKitStore
import CareKitUI
import ResearchKit
import UIKit
import os.log

final class CareFeedViewController: OCKDailyPageViewController,
                                    OCKSurveyTaskViewControllerDelegate {

    override func dailyPageViewController(
        _ dailyPageViewController: OCKDailyPageViewController,
        prepare listViewController: OCKListViewController,
        for date: Date) {

        checkIfOnboardingIsComplete { isOnboarded in

            guard isOnboarded else {

                let onboardCard = OCKSurveyTaskViewController(
                    taskID: TaskIDs.onboarding,
                    eventQuery: OCKEventQuery(for: date),
                    storeManager: self.storeManager,
                    survey: Surveys.onboardingSurvey(),
                    extractOutcome: { _ in [OCKOutcomeValue(Date())] }
                )

                onboardCard.surveyDelegate = self

                listViewController.appendViewController(
                    onboardCard,
                    animated: false
                )

                return
            }

            // 2.2 Query and display a card for each task.
        }
    }

    private func checkIfOnboardingIsComplete(_ completion: @escaping (Bool) -> Void) {

        var query = OCKOutcomeQuery()
        query.taskIDs = [TaskIDs.onboarding]

        storeManager.store.fetchAnyOutcomes(
            query: query,
            callbackQueue: .main) { result in

            switch result {

            case .failure:
                Logger.feed.error("Failed to fetch onboarding outcomes!")
                completion(false)

            case let .success(outcomes):
                completion(!outcomes.isEmpty)
            }
        }
    }

    // 2.3 Query all the tasks to be displayed on a given date

    // 2.4 Create a card for a given task

    // MARK: SurveyTaskViewControllerDelegate

    func surveyTask(
        viewController: OCKSurveyTaskViewController,
        for task: OCKAnyTask,
        didFinish result: Result<ORKTaskViewControllerFinishReason, Error>) {

        if case let .success(reason) = result, reason == .completed {
            reload()
        }
    }
}
```

이번에는 solution 을 좀 더 generic 하게 만들어보겠습니다. 현재 날짜의 모든 task 를 가져온 다음 각 작업에 대해서 뷰 컨트롤러를 생성하고, 해당 뷰 컨트롤러를 list 에 추가합니다. task 를 추가할수록 크기가 조정됩니다. 

```swift
// 2.2 Query and display a card for each task.
            
            self.fetchTasks(on: date) { tasks in
                tasks.compactMap {
                    self.taskViewController(for: $0, on: date)
                }.forEach {
                    listViewController.appendViewController($0, animated: false)
                }
            }

// ...

// 2.3 Query all the tasks to be displayed on a given date
    
    private func fetchTasks(
        on date: Date,
        completion: @escaping([OCKAnyTask]) -> Void) {

        var query = OCKTaskQuery(for: date)
        // ✅ Determines if tasks with no events should be included in the query results or not. False be default.
        // 🔥 예약된 이벤트가 없는 작업을 제외하도록 지정합니다.
        query.excludesTasksWithNoEvents = true

        storeManager.store.fetchAnyTasks(
            query: query,
            callbackQueue: .main) { result in

            switch result {

            case .failure:
                Logger.feed.error("Failed to fetch tasks for date \(date)")
                completion([])
            // 🔥 query 가 반환되면 가져온 tasks 를 completion handler 로 전달합니다.
            case let .success(tasks):
                completion(tasks)
            }
        }
    }
```

이것은 매일 일어나지 않은 일이 있을 때 실행됩니다. 예를 들어, 매주 월요일마다 약을 복용하는 처방을 받았다고 했을 때 화,수요일은 여전히 약이 있습니다. 아래의 속성은 그러한 작업이 qeury 에서 반환되지 않도록 합니다.

```swift
query.excludesTasksWithNoEvents = true
```

또한, 우리는 task 를 수행하고 뷰 컨트롤러를 반환하는 메서드를 작성해야합니다.

task 의 id 를 확인하고, check-in task 의 경우는 part1 에서 소개한 것처럼 SurveyTaskViewController 를 사용합니다.

```swift
// 2.4 Create a card for a given task

    private func taskViewController(
        for task: OCKAnyTask,
        on date: Date) -> UIViewController? {

        switch task.id {

        case TaskIDs.checkIn:
            // 🔥 part1 과 마찬가지로 task, event query, store manager 에 대한 참조를 제공합니다.
            let survey = OCKSurveyTaskViewController(
                task: task,
                eventQuery: OCKEventQuery(for: date),
                storeManager: storeManager,
                survey: Surveys.checkInSurvey(),
                extractOutcome: Surveys.extractAnswersFromCheckInSurvey
            )

            return survey

        case TaskIDs.rangeOfMotionCheck:
            let survey = OCKSurveyTaskViewController(
                task: task,
                eventQuery: OCKEventQuery(for: date),
                storeManager: storeManager,
                survey: Surveys.rangeOfMotionCheck(),
                extractOutcome: Surveys.extractRangeOfMotionOutcome
            )

            return survey

        default:
            return nil
        }
    }
```

ResearchKit 의 survey 와 결과를 CareKit 결과 값으로 변환하는 기능도 만들어야 합니다.

먼저, ResearchKit 과 CareKit 에 대해 살펴보겠습니다.

<img width="700" alt="7" src="https://user-images.githubusercontent.com/69136340/172382137-41c03704-1239-464f-aa2c-ae46de138710.png">

우리 Recover 앱이 ResearchKit survey 를 만들 것이고, ResearchKit 은 survey flow 를 진행하면서 참가자를 안내합니다.

그러면 ORKTaskResult 가 생성되어 우리의 앱으로 반환됩니다.

그리고나서 우리의 앱은 ResearchKit 의 결과를 CareKit 의 스토어에 유지되도록 CareKit 결과 값으로 변환합니다.

새로운 결과를 저장하면 completion ring 이 채워지고, card UI 가 업데이트 됩니다.

Survey.swift 에서 메서드를 정의하겠습니다.

```swift
import CareKitStore
import ResearchKit

struct Surveys {

    private init() {}

    // MARK: Onboarding

    static func onboardingSurvey() -> ORKTask {
        
        // The Welcome Instruction step.
        let welcomeInstructionStep = ORKInstructionStep(
            identifier: "onboarding.welcome"
        )

        welcomeInstructionStep.title = "Welcome!"
        welcomeInstructionStep.detailText = "Thank you for joining our study. Tap Next to learn more before signing up."
        welcomeInstructionStep.image = UIImage(named: "welcome-image")
        welcomeInstructionStep.imageContentMode = .scaleAspectFill
        
        // The Informed Consent Instruction step.
        let studyOverviewInstructionStep = ORKInstructionStep(
            identifier: "onboarding.overview"
        )

        studyOverviewInstructionStep.title = "Before You Join"
        studyOverviewInstructionStep.iconImage = UIImage(systemName: "checkmark.seal.fill")
        
        let heartBodyItem = ORKBodyItem(
            text: "The study will ask you to share some of your health data.",
            detailText: nil,
            image: UIImage(systemName: "heart.fill"),
            learnMoreItem: nil,
            bodyItemStyle: .image
        )

        let completeTasksBodyItem = ORKBodyItem(
            text: "You will be asked to complete various tasks over the duration of the study.",
            detailText: nil,
            image: UIImage(systemName: "checkmark.circle.fill"),
            learnMoreItem: nil,
            bodyItemStyle: .image
        )

        let signatureBodyItem = ORKBodyItem(
            text: "Before joining, we will ask you to sign an informed consent document.",
            detailText: nil,
            image: UIImage(systemName: "signature"),
            learnMoreItem: nil,
            bodyItemStyle: .image
        )

        let secureDataBodyItem = ORKBodyItem(
            text: "Your data is kept private and secure.",
            detailText: nil,
            image: UIImage(systemName: "lock.fill"),
            learnMoreItem: nil,
            bodyItemStyle: .image
        )
        
        studyOverviewInstructionStep.bodyItems = [
            heartBodyItem,
            completeTasksBodyItem,
            signatureBodyItem,
            secureDataBodyItem
        ]

        // The Signature step (using WebView).
        let webViewStep = ORKWebViewStep(
            identifier: "onboarding.signatureCapture",
            html: informedConsentHTML
        )

        webViewStep.showSignatureAfterContent = true
        
        // The Request Permissions step.
        let healthKitTypesToWrite: Set<HKSampleType> = [
            HKObjectType.quantityType(forIdentifier: .bodyMassIndex)!,
            HKObjectType.quantityType(forIdentifier: .activeEnergyBurned)!,
            HKObjectType.workoutType()
        ]

        let healthKitTypesToRead: Set<HKObjectType> = [
            HKObjectType.characteristicType(forIdentifier: .dateOfBirth)!,
            HKObjectType.workoutType(),
            HKObjectType.quantityType(forIdentifier: .appleStandTime)!,
            HKObjectType.quantityType(forIdentifier: .appleExerciseTime)!
        ]

        let healthKitPermissionType = ORKHealthKitPermissionType(
            sampleTypesToWrite: healthKitTypesToWrite,
            objectTypesToRead: healthKitTypesToRead
        )

        let notificationsPermissionType = ORKNotificationPermissionType(
            authorizationOptions: [.alert, .badge, .sound]
        )

        let motionPermissionType = ORKMotionActivityPermissionType()

        let requestPermissionsStep = ORKRequestPermissionsStep(
            identifier: "onboarding.requestPermissionsStep",
            permissionTypes: [
                healthKitPermissionType,
                notificationsPermissionType,
                motionPermissionType
            ]
        )

        requestPermissionsStep.title = "Health Data Request"
        requestPermissionsStep.text = "Please review the health data types below and enable sharing to contribute to the study."

        // Completion Step
        let completionStep = ORKCompletionStep(
            identifier: "onboarding.completionStep"
        )

        completionStep.title = "Enrollment Complete"
        completionStep.text = "Thank you for enrolling in this study. Your participation will contribute to meaningful research!"

        let surveyTask = ORKOrderedTask(
            identifier: "onboard",
            steps: [
                welcomeInstructionStep,
                studyOverviewInstructionStep,
                webViewStep,
                requestPermissionsStep,
                completionStep
            ]
        )

        return surveyTask
    }

    // MARK: 2.5 Check In Survey

    // MARK: 2.7 Range of Motion
}
```

- ResearchKit survey 를 만드는 메서드와 결과를 CareKit 결과 값으로 변환하는 메서드가 필요합니다.

## Create Survey Method

우리는 참가자가 잡을 자는 시간과 고통을 느끼는 시간 사이에 연관성이 있는지 알아보려고 합니다. 따라서 두가지 질문을 만들겠습니다.

- pain 에 대한 질문을 할 것입니다.
    - 필수 질문으로 만들 것. form 을 건너뛸 수 없고 특정 질문에 대한 답변 없이는 제출할 수 없음.
    - answer format. ResearchKit 에 예상하는 답변의 종류와 사용자에게 입력을 요청하는 방법을 알려준다. *(가중치를 입력하거나 이미지를 선택하거나 음성을 녹음하거나.)*
    이 경우는 UISlider 를 만드는 ORKScaleAnswerFormat 을 사용할 것입니다. *(slider 는 iOS 사용자에게 친숙한 인터페이스이다. 직관적이고, 설문조사에서 원을 채우는 것 보다 훨씬 더 좋은 경험을 제공합니다.)*
- sleep 에 대한 질문을 할 것입니다.
    - 최소 수면 시간 0시간, 최대 수면 시간 12시간.

두 item 을 단일 양식에 전달한 다음 ORKOrderedTask 를 생성하면 됩니다.

- Survey

```swift
// MARK: 2.5 Check In Survey
   
    static let checkInIdentifier = "checkin"
    static let checkInFormIdentifier = "checkin.form"
    static let checkInPainItemIdentifier = "checkin.form.pain"
    static let checkInSleepItemIdentifier = "checkin.form.sleep"

    // ✅ 2.5.1 create the survey
    static func checkInSurvey() -> ORKTask {

// 고통

// 최대 통증을 10으로 지정하고, 최소값을 1로 지정하고, 단계 크기를 1로 설정하여 반올림 숫자만 허용하도록 설정하고, 최소/최대에 대한 설명을 제공하겠습니다.
        let painAnswerFormat = ORKAnswerFormat.scale(
            withMaximumValue: 10, // 최대 통증 10.
            minimumValue: 1,      // 최소값 1.
            defaultValue: 0,
            step: 1,              // 단계 크기 1.
            vertical: false,      // 반올림 숫자만 허용.
            maximumValueDescription: "Very painful", // 최댓값 설명.
            minimumValueDescription: "No pain"       // 최솟값 설명.
        )

        // 🔥 pain 에 대한 질문
        let painItem = ORKFormItem(
            identifier: checkInPainItemIdentifier,
            text: "How would you rate your pain?",
            answerFormat: painAnswerFormat
        )
        painItem.isOptional = false

// 수면
        // 🔥 최소 수면 시간 0시간, 최대 수면 시간 12시간.
        let sleepAnswerFormat = ORKAnswerFormat.scale(
            withMaximumValue: 12,
            minimumValue: 0,
            defaultValue: 0,
            step: 1,
            vertical: false,
            maximumValueDescription: nil,
            minimumValueDescription: nil
        )

        let sleepItem = ORKFormItem(
            identifier: checkInSleepItemIdentifier,
            text: "How many hours of sleep did you get last night?",
            answerFormat: sleepAnswerFormat
        )
        sleepItem.isOptional = false

// formStep
        // 🔥 formStep 에는 고유한 식별자, 제목 및 텍스트가 필요합니다.
        let formStep = ORKFormStep(
            identifier: checkInFormIdentifier,
            title: "Check In",
            text: "Please answer the following questions."
        )

        // 🔥 form 을 건너뛸 수 없도록 하려면 이 항목을 false 설정해야 합니다.
        formStep.isOptional = false
        // 🔥 위의 두 항목(painItem, sleepItem)을 formStep 에 전달.
        formStep.formItems = [
            painItem,
            sleepItem
        ]

        // 🔥 item 을 가지고 ORKOrderedTask 생성.
        let surveyTask = ORKOrderedTask(
            identifier: checkInIdentifier,
            steps: [formStep]
        )

        return surveyTask
    }
```

### Create CareKit values to persist.

두 번째 함수는 ResearchKit task 결과를 가져와서 지속할 CareKit 값을 생성해야 합니다. 그 전에 ResearchKit task 의 구조를 살펴보고 어떻게 분석하는지 방법에 대해서 알아봅시다.

ORKTaskResults 는 중첩된 타입이라는 것을 이해하는 것이 중요합니다. check-in survey 를 위한 root 결과에서 시작한 다음 checkin.form 을 dril down(아래로 내려가며 분석) 합니다.

<img width="700" alt="8" src="https://user-images.githubusercontent.com/69136340/172383148-d704253c-8c00-4f06-87b8-4f74b962e3c5.png">

checkin.form 은 두 가지 children 을 가지고 있고, 이것을 파헤쳐야 합니다.

먼저, pain item 과 sleep itme 식별자에 대해 주어진 답을 찾고 싶습니다. 예시에서는 4와 11 입니다. 

<img width="714" alt="9" src="https://user-images.githubusercontent.com/69136340/172383225-63527af3-3de2-433d-868f-7331bf783754.png">

<img width="713" alt="10" src="https://user-images.githubusercontent.com/69136340/172383250-3bb79833-01ed-4bc0-a750-23b5f3704d0b.png">

<img width="713" alt="11" src="https://user-images.githubusercontent.com/69136340/172383286-48af5cbb-68ef-4d45-8da9-17a992f01aed.png">

시각적으로 살펴봤고, 코드에서도 같은 프로세스입니다. 

```swift
// 2.5.2 Parse the results

static func extractAnswersFromCheckInSurvey(
        _ result: ORKTaskResult) -> [OCKOutcomeValue]? {

        guard
            let response = result.results?
                .compactMap({ $0 as? ORKStepResult })
                // 🔥 'checkin.form' 식별자만 선택
                .first(where: { $0.identifier == checkInFormIdentifier }),

            // 🔥 ORKScaleQuestionResult 타입을 가진 모든 children 을 추출할 수 있습니다. 
            let scaleResults = response
                .results?.compactMap({ $0 as? ORKScaleQuestionResult }),

            // 🔥 pain answer 은 해당 식별자를 가진 첫번째 것이고, sleep answer 은 해당 식별자를 가진 첫번째 것 입니다.
            let painAnswer = scaleResults
                .first(where: { $0.identifier == checkInPainItemIdentifier })?
                .scaleAnswer,

            let sleepAnswer = scaleResults
                .first(where: { $0.identifier == checkInSleepItemIdentifier })?
                .scaleAnswer
        else {
            assertionFailure("Failed to extract answers from check in survey!")
            return nil
        }

// 🔥 CareKit 결과값으로 변환해야 합니다.
// kind 프로퍼티는 선택사항이지만 나중에 값을 찾으려면 도움이 될 수 있습니다.(이 부분은 part 3 에서 알아보겠습니다!)
        var painValue = OCKOutcomeValue(Double(truncating: painAnswer))
        painValue.kind = checkInPainItemIdentifier

        var sleepValue = OCKOutcomeValue(Double(truncating: sleepAnswer))
        sleepValue.kind = checkInSleepItemIdentifier

        return [painValue, sleepValue]
    }
```

### Let’s run the app and see how we’re doing.

<img width="700" alt="12" src="https://user-images.githubusercontent.com/69136340/172385586-5746487f-8c8f-4e37-97d4-f8f4e5d96bc1.png">

part 1에서 이미 onboarding 을 완료했기 때문에 consent flow 를 다시 하지 않아도 됩니다.
card 를 탭하면 ResearchKit survey 로 이동합니다. 예를 들어 통증 4, 수면시간 8 설정하겠습니다.  

<img width="700" alt="13" src="https://user-images.githubusercontent.com/69136340/172383896-166db951-1c18-4063-82c6-5bdd8236cec9.png">

Care feed 로 돌아가면 위에 completion ring 이 채워지는 것을 볼 수 있고, ResearchKit 의 답이 CareKit 에 성공적으로 파싱되었음을 알 수 있습니다.

<img width="700" alt="14" src="https://user-images.githubusercontent.com/69136340/172383923-e7554ae6-ec00-438d-a9d5-cad384d14c36.png">

다수의 질문 폼을 가진 check-in survey 를 마쳤고, CareKit 에 우리가 원하는 대로 유지되는 것을 확인했습니다. 반쯤 왔군요!

<img width="700" alt="15" src="https://user-images.githubusercontent.com/69136340/172384023-1139215c-6b8f-4a0c-bf98-2151fe58c50b.png">

### Create Dynamic Schedules

다음은 CareKit 의 고급 일정으로 넘어가봅시다. 나중에 우리는 range of motion task 를 적용할 것입니다. check-in task 이나 이전의 onboarding task 처럼 첫 번째는 schedule 을 정의하는 것입니다. 하지만, 이 작업에는 좀 더 많은 작업이 필요합니다.

Jamie 는 시간이 지남에 따라 참가자들에게 range of motion(운동 범위)를 측정하도록 요청하는 빈도를 줄이도록 요청했습니다. 구체적으로, 우리는 참가자가 첫 주 동안 매일 range of motion 을 측정하도록 하는 schedule 을 세웠으면 합니다. 그 다음 월말까지 일주일에 한번 그리고 그 이후에는 다시 측정하지 않도록 했으면 합니다.

첫 주에 하루 한 번, 나머지 주에는 주마다 한번. 몇가지 주요 날짜를 정의하는 것으로 시작하겠습니다.(thisMorning, nextWeek, and nextMonth)

- AppDelegate

```swift
// 2.6 Add a range of motion task

        let thisMorning = Calendar.current.startOfDay(for: Date())

        let nextWeek = Calendar.current.date(
            byAdding: .weekOfYear,
            value: 1,
            to: Date()
        )!

        let nextMonth = Calendar.current.date(
            byAdding: .month,
            value: 1,
            to: thisMorning
        )

        // 🔥 CareKit 에서 미묘한 일정을 만들려면 OCKScheduleElement 를 사용해야 합니다.
        // schedule element 에는 시작 날짜, 종료 날짜가 있고, 해당 기간 동안 iinterval 을 가지고 반복됩니다.
        let dailyElement = OCKScheduleElement(
            start: thisMorning,
            end: nextWeek,
            interval: DateComponents(day: 1),
            text: nil,
            targetValues: [],
            duration: .allDay
        )

        // 🔥 다음주부터 시작해서 다음달까지 매주 반복됩니다.
        let weeklyElement = OCKScheduleElement(
            start: nextWeek,
            end: nextMonth,
            interval: DateComponents(weekOfYear: 1),
            text: nil,
            targetValues: [],
            duration: .allDay
        )

        // 🔥 두개의 element 를 가지고 compound schedule 을 만들 수 있습니다.
        let rangeOfMotionCheckSchedule = OCKSchedule(
            composing: [dailyElement, weeklyElement]
        )

        // 🔥 그리고 해당 schdule 을 사용하는 range of motion task 를 만들 수 있습니다.
        let rangeOfMotionCheckTask = OCKTask(
            id: TaskIDs.rangeOfMotionCheck,
            title: "Range Of Motion",
            carePlanUUID: nil,
            schedule: rangeOfMotionCheckSchedule
        )

        // 🔥 물론 store 에 추가해 주어야 합니다.
        storeManager.store.addAnyTasks(
            [onboardTask, checkInTask, rangeOfMotionCheckTask],
            callbackQueue: .main) { result in

            switch result {

            case let .success(tasks):
                Logger.store.info("Seeded \(tasks.count) tasks")
                
            case let .failure(error):
                Logger.store.warning("Failed to seed tasks: \(error as NSError)")
            }
        }
```

다른 task 와 마찬가지로 다음 단계는 Care feed 로 이동해서 CareKit 에 이 작업을 표시할 방법을 지정하는 것입니다.

### Range of motion

다시 SurveyTaskViewController 를 사용합니다. 답변을 얻기위해서 survey 와 function 을 제공해야 합니다. Survey.swift 로 돌아가서 작업해야 합니다. range of motion tak 는 사실상 간단합니다.

ResearchKit 에 미리 정의되어있습니다. 식별자를 지정하고, 측정할 무릎을 지정하기만 하면 됩니다.

```swift
// MARK: 2.7 Range of Motion

static func rangeOfMotionCheck() -> ORKTask {

        let rangeOfMotionOrderedTask = ORKOrderedTask.kneeRangeOfMotionTask(
            withIdentifier: "rangeOfMotionTask",
            limbOption: .left,
            intendedUseDescription: nil,
            // 🔥 해당 task 가 주는 메시지는 기본값이지만 사용자 지정 메시지를 보여드리기 위해서 다음 옵션 설정하겠습니다. 
            options: [.excludeConclusion]
        )
        // 🔥 위의 옵션으로 설정 후, 물리치료와 관련된 특별한 격려의 메시지를 사용자 지정하여 전달할 것입니다.
        let completionStep = ORKCompletionStep(identifier: "rom.completion")
        completionStep.title = "All done!"
        completionStep.detailText = "We know the road to recovery can be tough. Keep up the good work!"

        rangeOfMotionOrderedTask.appendSteps([completionStep])
        
        return rangeOfMotionOrderedTask
    }

    // 🔥 ResearchKit 결과값으로 변환하는 것입니다.
    static func extractRangeOfMotionOutcome(
        _ result: ORKTaskResult) -> [OCKOutcomeValue]? {

        guard let motionResult = result.results?
            .compactMap({ $0 as? ORKStepResult })
            .compactMap({ $0.results })
            .flatMap({ $0 })
            .compactMap({ $0 as? ORKRangeOfMotionResult })
            // 🔥 우리가 할 수 있는 방법은 많지만, 오늘은 첫 번째 범위의 움직임 결과가 나올 때만 사용하겠습니다. 그리고 오직 하나 밖에 없다는 것을 알고 있습니다.
            .first else {

            assertionFailure("Failed to parse range of motion result")
            return nil
        }

        // 🔥 range of motion 의 결과는 유용한 특성들을 가지고 있습니다. 하지만, 우리의 use case 에서 가장 관심 있는 것은 range 입니다. 이것은 참가자가 무릎을 얼마나 구부릴 수 있었는지 측정하는 범위입니다.
        var range = OCKOutcomeValue(motionResult.range)
// 🔥 이 값은 나중에 쉽게 찾을 수 있도록 keyPath 와 함께 사용할 수 있습니다. part 3 에서 좀 더 알아보겠습니다.
        range.kind = #keyPath(ORKRangeOfMotionResult.range)

        return [range]
    }
```

모든 준비를 마친 것 같습니다.

먼저 스케줄이 우리가 의도하는 대로 작동하고 있는지 확인해볼 필요가 있습니다. range of motion task 는 매일  표시되어야 합니다. 그러나 다음주로 넘어가면 월요일을 제외하고 표시되지 않습니다. 더 나아가서 다음 달에는 더 이상 나타나지 않습니다.

CareKit shcedules 는 이러한 요법을 미리 프로그래밍할 수 있는 좋은 방법입니다.

6월 7일 월요일 과 6월 9일에는 첫주이기 때문에 매일매일 check-in 나타난다.

<img width="1000" alt="스크린샷 2022-06-07 오후 10 04 22" src="https://user-images.githubusercontent.com/69136340/172386865-e454f4ce-964b-4286-b5fe-3148b00fa5da.png">

6월 14일 월요일이라 표시되고, 6월 17일은 표시되지 않습니다.

<img width="1000" alt="스크린샷 2022-06-07 오후 10 04 31" src="https://user-images.githubusercontent.com/69136340/172386906-78457eff-0da8-4b7d-9572-68cd4b8ae353.png">

다음 달에는 표시되지 않습니다.

<img width="700" alt="1" src="https://user-images.githubusercontent.com/69136340/172386956-eda3ce0a-0118-4e21-8fe7-c20bb05b9141.png">

- range of motion

<img width="700" alt="11" src="https://user-images.githubusercontent.com/69136340/172391480-e13667be-6c98-4ee7-8311-d4a98d4125c2.png">

<img width="700" alt="22" src="https://user-images.githubusercontent.com/69136340/172391515-d0e482b2-4539-401a-919a-7fbacd2ec3cc.png">

<img width="700" alt="33" src="https://user-images.githubusercontent.com/69136340/172391533-7ab3e4fa-073c-4ab5-bd48-59bc0f74f5f4.png">

<img width="700" alt="44" src="https://user-images.githubusercontent.com/69136340/172391541-ed5c2491-6303-4a20-b96f-7b7a01f1adc5.png">

