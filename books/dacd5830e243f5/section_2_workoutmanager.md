---
title: "WorkoutManagerの設定"
---
:::details このChapterの目次
* 2.WorkoutManagerの作成
  * 2-1. Projectの設定
  * 2-2. HelthKitの使用権限を取得する
  * 2-3. Workout開始時に記録を開始するメソッド
  * 2-4. 
:::
# 2.WorkoutManagerの作成
WorkoutManagerのクラスでは,HKWorkoutSessionを利用して,運動中のデータを保存,更新という処理を行います. 
これらのデータを利用するまには,利用の許可やBackgroundでの挙動についても設定を行う必要があります. 
そのため,実装の前に,Projectについて行います. 

## 2-1.Projectの設定
* Capabilityの追加
Targetから,HelthKitとBackGroundModeの設定を下記のように設定します. 
今回のアプリでは,BackGround時でも運動を記録するするため,BackGroundModeを設定します. 
![](/images/article8/capasettingsample.png)

* Info.plistの設定
  HelthKit ShareとUpdateの2つを追加します. 
![](/images/article8/infoSettingUsage.png)


## 2-2.HelthKitの使用権限を取得する
今回は,WorkoutManagerから値をPublishする形式とするため,ObservableObjectを批准します. 
HelthKitのInstace methodとしてrequestAuthorization(toShare:read:completion:)を呼び出して,許諾のダイアログを表示させます. 
引数として設定するパラメータの詳細は下記のようになっています. 

| 引数        | 定義                                                                          |
| ----------- | ----------------------------------------------------------------------------- |
| typeToShare | HelthKitにデータと作成を行う内容を指定する HKSampleType  の子クラスが指定可能 |
| typesToRead | HealthKitのStoreデータから,呼び出すことができるデータを指定する.              |
| completion  | ユーザーの許諾を得たあとの処理を指定する                                      |


```swift
import Foundation
import HealthKit

class WorkoutManager: NSObject, ObservableObject {
    var selectedWorkout: HKWorkoutActivityType?

    let healthStore = HKHealthStore()
    var session: HKWorkoutSession?
    var builder: HKLiveWorkoutBuilder?

    func starWorkout(workoutType: HKWorkoutActivityType) {
        let configuration = HKWorkoutConfiguration()
        configuration.activityType = workoutType
        configuration.locationType = .outdoor

        do {
            session = try HKWorkoutSession(healthStore: healthStore, configuration: configuration)
            builder = session?.associatedWorkoutBuilder()
        } catch {
            return
        }

        builder?.dataSource = HKLiveWorkoutDataSource(healthStore: healthStore, workoutConfiguration: configuration)
        let startDate = Date()
        session?.startActivity(with: startDate)
        builder?.beginCollection(withStart: startDate, completion: { (sucess, error) in
            if let error = error {
                print(error.localizedDescription)
                return
            }
            print("Work out \(sucess ? "Sucess" : "Failer")")
        })
    }
}
```

## 3.Workout開始時に記録を開始する
指定した,Workoutの種類に応じて,データの取得を開始するメソッドを作成します. 
下記のメソッドでは,HKWorkoutSessionとHKLiveWorkoutBuilderを使用して,運動開始後から自動的にデータがUpdateされるようにしています. 
```swift
func starWorkout(workoutType: HKWorkoutActivityType) {
    /* ① 運動のタイプなど,測定時のロジックに影響する値を設定する.
        屋外と屋内で消費カロリーなどが変化する
    */ 
    let configuration = HKWorkoutConfiguration()
    configuration.activityType = workoutType
    configuration.locationType = .outdoor

    // ② HKWorkoutConfigurationをもとに,workout sessionを初期化する
    //    また,HKLiveWorkoutBuilderに参照を渡す. 
    do {
        session = try HKWorkoutSession(healthStore: healthStore, configuration: configuration)
        builder = session?.associatedWorkoutBuilder()
    } catch {
        return
    }

    /* ③ 保存先のHelthKitStoreを渡す.sessionと同じ,configurationを設定する. 
        Sessionが開始されるとデータが自動で保存されるようになる. 
    */
    builder?.dataSource = HKLiveWorkoutDataSource(healthStore: healthStore, workoutConfiguration: configuration)

    /* ④ DelegateObjectを設定して,sessionとbuilderの状態を監視する
    */
    session?.delegate = self
    builder?.delegate = self

    /* ⑤ メソッド実行のタイミングの時間でsessionとデータの保存を開始する
    */
    let startDate = Date()
    session?.startActivity(with: startDate)
    builder?.beginCollection(withStart: startDate, completion: { (sucess, error) in
        if let error = error {
            print(error.localizedDescription)
            return
        }
        print("Work out \(sucess ? "Sucess" : "Failer")")
    })
}
```


## Sessionの状態を管理するメソッドとPropertyの作成
ControlsViewの終了中断のアクションに応じて,Sessionの状態を管理するようにします. 

```swift
    // MARK: -Workout state control
    @Published var running = false

    func pause() {
        session?.pause()
    }

    func resume() {
        session?.resume()
    }

    func togglePause() {
        if running == true {
            pause()
        } else {
            resume()
        }
    }

    func endWorkout() {
        session?.end()
        showingSummaryView = true
    }
```

## SummaryViewに表示するMethodの表示

Extenstionの実装
```swift
extension WorkoutManager: HKWorkoutSessionDelegate {
    // Tells the delegate that the session’s state has changed.
    func workoutSession(_ workoutSession: HKWorkoutSession, didChangeTo toState: HKWorkoutSessionState, from fromState: HKWorkoutSessionState, date: Date) {
        DispatchQueue.main.async {
            self.running = toState == .running
        }

        if toState == .ended {
            builder?.endCollection(withEnd: date, completion: { sucess, error in
                self.builder?.finishWorkout(completion: { workout, error in

                })
            })
        }
    }
    // Tells the delegate that the session has failed with an error.
    func workoutSession(_ workoutSession: HKWorkoutSession, didFailWithError error: Error) { }
}

// MARK: -HKLiveWorkoutBuilderDelegate
extension WorkoutManager: HKLiveWorkoutBuilderDelegate {
    // Tells the delegate that new data has been added to the builder.
    func workoutBuilder(_ workoutBuilder: HKLiveWorkoutBuilder, didCollectDataOf collectedTypes: Set<HKSampleType>) {
        for type in collectedTypes {
            // Make sure the type is HKQUantityType
            guard let quantityType = type as? HKQuantityType else { return }
            let statistics = workoutBuilder.statistics(for: quantityType)

            // update the publisher
            updateForStatistics(statistics)
        }
    }
    // Tells the delegate that a new event has been added to the builder.
    func workoutBuilderDidCollectEvent(_ workoutBuilder: HKLiveWorkoutBuilder) { }
}
```