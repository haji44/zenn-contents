---
title: "WathcOS 入門 HealthKitを使う"
emoji: "⌚️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["WatchOS","HealthKit","Swift","Xcode"]
published: false
---

# HelthKitを使ってWatchOSで動かす!
この記事では,運動時のデータをリアルタイムに記録するアプリをWatchOSで作成します. HelthKitは運動時のデータを管理するDBの役割を果たします.
実装の内容は,WWDCのWorkoutアプリを開発するセッションをもとにしています.

## 完成のイメージ
![](/images/article8/finish.gif)

## 画面の遷移のイメージ
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

# StartView (運動のタイプを選択する画面)
測定するWorkoutのタイプをリスト表示をします.
HKWorkoutActivityTypeは取得可能な運動のタイプを設定することが可能です.
```swift
import SwiftUI
import HealthKit

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
今回は,サンプルのためにextensionで実装しますが,実際のプロジェクトでは影響範囲を鑑みてインスタンスに切り出すなどをしたほうが良いでしょう.
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

# PagingView (アクティビティー中の表示画面)
WatchOSではアクティビティー中に特定操作をページごとに分けて実装を行うようにします.
以下実装手順
1. 切り替えるPageをEnumのCaseとして定義する
2. @Stateで変更があるたびにViewが変更されるようにする
3. TabViewでselectionをBindingする

WatchOSでは,TabViewを宣言しておくとWatchOS側でPageとして表示処理をしてくれます.

```swift
import SwiftUI
import WatchKit

struct SessionPagingView: View {
    @State private var selection: Tab = .metrics

    enum Tab {
        case control, metrics, nowPlaying
    }

    var body: some View {
        TabView(selection: $selection) {
            ControlsView().tag(Tab.control) // 運動の終了および
            MetricsView().tag(Tab.metrics) // 運動中の心拍数などの情報を表示する
            NowPlayingView().tag(Tab.nowPlaying) // 音楽Playerを表示する(WatchKitKitが必須となる)
        }
    }
}
```

## 1.MetricsView(運動中の消費カロリー等の詳細情報の画面の作成)
今回の肝であるリアルタイムにデータを取得して表示する画面を実装します. 
表示するデータの項目は, ①心拍数,②距離,③運動時間,④消費カロリーとします.

MesurementViewの構成要素は,Timerとそれぞれの指標を表示するTextで構成します.
Timerの実装は別Viewで切り出します. 

1. 各指標の実装
距離や消費カロリーは,それぞれをMesurementで,適した単位などで実装するようにします. 
MesurementクラスからFormatを指定する際には,実装時には下記の記事などが参考になりました.
https://cocoacasts.com/swiftui-essentials-working-with-units-and-measurements-in-swiftui

```swift
import SwiftUI

struct MetricsView: View {
    @EnvironmentObject var workoutManager: WorkoutManager

    var body: some View {
        VStack(alignment: .leading) {
            // mm秒まで表示する運動時間を表示するView
            ElapsedTimeView(elapsedTime: workoutManager.builder?.elapsedTime ?? 0, showSubseconds: context.cadence == .live)
                    .foregroundStyle(.yellow)
            // 消費カロリー
            Text(Measurement(value: 47, unit: UnitEnergy.kilocalories)
                        .formatted(.measurement(width: .abbreviated, usage: .workout, numberFormatStyle: .number.precision(.fractionLength(0)))))
            // 心拍数                        
            Text( 153.formatted(.number.precision(.fractionLength(0))) + " bpm")
            // 移動距離
            Text(Measurement(value: 800, unit: UnitLength.meters).formatted(.measurement(width: .abbreviated, usage: .road)))
        }
    }
}
```
経過時間を表示するViewは,Formatterを利用して「分:秒:mm秒」の形式で画面に表示をさせます.
こちらの実装は,こちらに記載しております.

https://zenn.dev/jime/articles/article11_timerforwatch

経過時間を表示するViewの実装内容
```swift
import SwiftUI

struct ElapesdTimeView: View {
    var elapedTime: TimeInterval = 0
    var showSubseconds: Bool = true
    @State private var timeFormatter = ElapesedTImeFormatter()

    var body: some View {
        Text(NSNumber(value: elapedTime), formatter: timeFormatter)
            .fontWeight(.semibold)
            .onChange(of: showSubseconds) {
                timeFormatter.showSubseconds = $0
            }
    }
}

class ElapesedTImeFormatter: Formatter {
    let componentFormatter: DateComponentsFormatter = {
        let formatter = DateComponentsFormatter()
        formatter.allowedUnits = [.minute, .second]
        formatter.zeroFormattingBehavior = .pad
        return formatter
    }()

    var showSubseconds = true

    override func string(for obj: Any?) -> String? {

        guard let time = obj as? TimeInterval else { return nil }

        guard let formattedString = componentFormatter.string(from: time) else {
            return nil
        }

        if showSubseconds {
            let hunredths = Int((time.truncatingRemainder(dividingBy: 1)) * 100)
            let decimalSperator = Locale.current.decimalSeparator ?? "."
            return String(format: "%@%@%0.2d", formattedString, decimalSperator, hunredths)
        }
        return formattedString
    }

}
```

## 2. 運動の中断,終了を選択するView





# 運動終了後のSummaryを表示するViewの作成
運動終了後に表示するSummaryViewを作成します. 
MetricsViewと同様にMesurementでデータの単位を指定します. 

```swift
import SwiftUI
import HealthKit

struct SummaryView: View {
    @Environment(\.dismiss) var dismiss
    @State private var durationFormatter: DateComponentsFormatter = {
        let formatter = DateComponentsFormatter()
        formatter.allowedUnits = [.hour, .minute, .second]
        formatter.zeroFormattingBehavior = .pad
        return formatter
    }()

    var body: some View {
        ScrollView(.vertical, showsIndicators: true) {
            VStack(alignment: .leading) {
                SummaryMetricView(title: "Total Time", value: durationFormatter.string(from: 30 * 60 ) ?? "")
                    .accentColor(.yellow)
                SummaryMetricView(title: "Total Distance",
                                  value: Measurement(value: 1624, unit: UnitLength.meters).formatted(
                                    .measurement(width: .abbreviated, usage: .road)
                                  ))
                    .accentColor(.yellow)
                SummaryMetricView(title: "Total Calories",
                                  value: Measurement(value: 332, unit: UnitEnergy.kilocalories).formatted(
                                    .measurement(width: .abbreviated, usage: .workout, numberFormatStyle: .number.precision(.fractionLength(0)))
                                  ))
                    .accentColor(.pink)
                SummaryMetricView(title: "Avg Heart Rate",
                                  value: 154.formatted() + "bpm")
                .accentColor(Color.red)
                Text("Acitivity Rings")
                ActivityRingsView(healthStore: HKHealthStore())
                    .frame(width: 50, height: 50)
                Button {
                    dismiss()
                } label: {
                    Text("Done")
                }
            }
            .scenePadding()
        }
        .navigationTitle("Summary")
        .navigationBarTitleDisplayMode(.inline)
    }
}

// 各指標のViewComponents
struct SummaryMetricView: View {
    var title: String
    var value: String

    var body: some View {
        Text(title)
            .foregroundStyle(.foreground)
        Text(value)
            .font(.system(.title2, design: .rounded).lowercaseSmallCaps())
        Divider()
    }
}

```

# AcitivityRingsを表示してみる

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

Acitivityringは,WKInterfaceObjectRepresentableを使ってSwiftUIで使用する事ができます.
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
HealthKitで取得したデータをViewに反映するまでは下記のような流れでデータのやり取りをします. 

## WorkoutManagerを作成
このクラスでは,Workoutのタイプに応じて記録開始時点のデータを特定することができます. 

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
    func workoutBuilderDidCollectEvent(_ workoutBuilder: HKLiveWorkoutBuilder) { }
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