---
title: "WathcOS 入門 HealthKitを使う"
emoji: "⌚️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["WatchOS","HealthKit","Swift","Xcode"]
published: false
---

# HelthKitを使ってWatchOSで動かす!
この記事では,運動時のデータをリアルタイムに記録するアプリをWatchOSで作成します. HelthKitは運動時のデータを管理するDBの役割を果たします.

* 完成のイメージ
![](/images/article8/finish.gif)

* 画面の遷移のイメージ
![](/images/article8/apparchitecture.png)

* 使用するライブラリ
1. SwiftUI
2. HelthKit

# 実装手順
1. SwiftUIのViewのset upとMockデータの挿入
2. Acitivity Ringの作成
3. HelthKitとのデータのやり取りを行うManagerClassの作成
4. Mockデータをリプレイス


# Viewのsetupをしてみる
WatchOSのViewをMockデータをもとにじっそうしていきます. WatchOSはSwiftUIでレイアウトが組めます．今回は,AcitivityRingを除いてSwiftUIで実装をしていきます.

## StartView 運動のタイプを選択
測定するWorkoutのタイプをリスト表示をします.
HKWorkoutActivityTypeは取得可能な運動のタイプを設定することが可能です.
```swift
struct StartView: View {
    var workoutTypes: [HKWorkoutActivityType] = [.cycling, .running, .swimming]

    var body: some View {
        List(workoutTypes) { workoutType in
            NavigationLink {
                Text(workoutType.name) // 
            } label: {
                Text(workoutType.name)
            }
            .padding(
                EdgeInsets(top: 15, leading: 5, bottom: 15, trailing: 5)
            )
        }
        .listStyle(.carousel)
        .navigationBarTitle("Workout")
    }
}
```

HKWorkoutActivityTypeのcaseをLabelに表示させるために,ComputedPropertyを定義します．また,SwiftUIのList表示を行うために,Identifiableに適合させます.
```swift
extension HKWorkoutActivityType: Identifiable {
    public var id: UInt {
        rawValue
    }

    var name: String {
        switch self {
        case .running:
            return "Run"
        case .cycling:
            return "Cycling"
        case .swimming:
            return "Swim"
        default:
            return "Nothing"
        }
    }
}
```

## アクティビティー中の表示画面の作成

WatchOSではアクティビティー中に特定操作をページごとに分けて実装を行うようにする.ページの実装はTabViewで簡単に可能!
以下実装手順
1. PageをEnumのCaseとして定義する
2. @Stateで最初に表示するTabを宣言
3. TabViewのselectionをBindingする

TabViewを宣言しておくとWatchOS側でPageとして表示をしてくれます.
```swift
struct SessionPagingView: View {

    @State private var selection: Tab = .metrics

    enum Tab {
        case control, metrics, nowPlaying
    }

    var body: some View {
        TabView(selection: $selection) {
            Text("Controls").tag(Tab.control)
            Text("Metrics").tag(Tab.metrics)
            Text("Now Plyaing").tag(Tab.nowPlaying)
        }
    }
}
```

### 運動中のMetricsViewの作成

Metricsがこの記事の肝です!
HelthKitから取得したデータを表示する際には,異なる単位の値を適切にFormatterを掛ける必要があります.
MesurementクラスからFormatを指定するのですがなかなか,直感的に理解するにはむずかしかったですね..
実装時には下記の記事などが参考になりました.

https://cocoacasts.com/swiftui-essentials-working-with-units-and-measurements-in-swiftui

まずMesurementに表示したい値と単位を指定します. 
![](2022-03-26-23-29-29.png)


##  Summaryを表示するViewの作成

* 完成のイメージ
Summaryを反映するイメージを作成する


## AcitivityRingsを表示してみる

```swift
import Foundation
import HealthKit // アクティビティー情報の取得に必要
import SwiftUI // WKInterfaceObjectRepresentableに必要

struct ActivityRingsView: WKInterfaceObjectRepresentable {
    let healthStore: HKHealthStore

    func makeWKInterfaceObject(context: Context) -> some WKInterfaceObject {
        let activityRingsObject = WKInterfaceActivityRing()

        let calendar = Calendar.current
        var components = calendar.dateComponents([.era, .year, .month, .day], from: Date())
        components.calendar = calendar

        let predicate = HKQuery.predicateForActivitySummary(with: components)

        let query = HKActivitySummaryQuery(predicate: predicate) { query, summaries, erorr in
            DispatchQueue.main.async {
                activityRingsObject.setActivitySummary(summaries?.first, animated: true)
            }
        }

        healthStore.execute(query)
        return activityRingsObject
    }

    func updateWKInterfaceObject(_ wkInterfaceObject: WKInterfaceObjectType, context: Context) {}
    
}
```
AcitivityringのViewはWKInterfaceObjectRepresentableを使ってSwiftUIで使用する事ができます.
Acitivityの情報はHelthKitから取得をする必要があるので,HelthKitのDatabaseにアクセスして,その結果をacitivityの結果に入れるようにします.
* HelthKitからデータを取得する手順
1. 取得する対象の日付を決める
```swift
let calendar = Calendar.current
var components = calendar.dateComponents([.era, .year, .month, .day], from: Date())
components.calendar = calendar
```

2. HKQueryを作成する
predicateに検索する条件としてdatecomponentsを設定し,HKActivitySummaryQueryに引数として渡します.

```swift
let predicate = HKQuery.predicateForActivitySummary(with: components)

let query = HKActivitySummaryQuery(predicate: predicate) { query, summaries, erorr in
    DispatchQueue.main.async {
        activityRingsObject.setActivitySummary(summaries?.first, animated: true)
    }
}
```

3. クエリの実行


## DataのFlow
HealthKitで取得したデータをViewに反映するまでは下記のような流れでデータのやり取りをします


## WorkoutManagerを作成
このクラスでは,Workoutのタイプに応じて記録開始時点のデータを特定することができます


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




# Projectの設定

## Capabiriltyの設定
![](/images/article8/infokey.png)

![](/images/article8/projectsetting.png)


# Workoutmanager
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
    func workoutBuilderDidCollectEvent(_ workoutBuilder: HKLiveWorkoutBuilder) {    }
}
```
HKWorkoutSessionDelegate SessionのStatusが変化したときに,呼ばれる

HKLiveWorkoutBuilderDelegate Builderに新しいデータが追加されるときに呼ばれる

この2つを使ってPublisherをアップデートするようにする
PublisherはViewでbindする.こうすることで


## Timelineを使って常にデータがアップデートされるようにする

```swift
import SwiftUI
import HealthKit

struct MetricsView: View {
    @EnvironmentObject var workoutManager: WorkoutManager
    
    var body: some View {
        TimelineView(MetricsTimelineSchedule(from: workoutManager.builder?.startDate ?? Date())) { context in
            VStack(alignment: .leading) {
                ElapsedTimeView(elapsedTime: workoutManager.builder?.elapsedTime ?? 0, showSubseconds: context.cadence == .live)
                    .foregroundStyle(.yellow)
                Text(Measurement(value: workoutManager.activeEnergy, unit: UnitEnergy.kilocalories)
                        .formatted(.measurement(width: .abbreviated, usage: .workout, numberFormatStyle: .number.precision(.fractionLength(0)))))
                Text(workoutManager.heartRate.formatted(.number.precision(.fractionLength(0))) + " bpm")
                Text(Measurement(value: workoutManager.distance, unit: UnitLength.meters).formatted(.measurement(width: .abbreviated, usage: .road)))
            }
            .font(.system(.title, design: .rounded).monospacedDigit().lowercaseSmallCaps())
            .frame(maxWidth: .infinity, alignment: .leading)
            .ignoresSafeArea(edges: .bottom)
            .scenePadding()
        }
    }
}
```


