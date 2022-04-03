---
title: "SwiftUIでViewを作成する"
---
:::details このChapterの目次
* 1.WatchOSのView作成
  * 1-1.StartView (運動のタイプを選択する画面)
  * 1-2.PagingView (アクティビティー中の表示画面)
  * 1-3.運動終了後のSummaryを表示するViewの作成
:::

# 1.WatchOSのView作成
WatchOSはSwiftUIでレイアウトが組めます．今回の記事では,SwiftUIでViewの実装を行います.

## 1-1.StartView (運動のタイプを選択する画面)
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

* 実装した画面
![](/images/article8/workoutList.png)

## 1-2.PagingView (アクティビティー中の表示画面)
WatchOSではアクティビティー中に特定操作をページごとに分けて実装を行うようにします.
以下実装手順
1. 切り替えるPageをEnumのCaseとして定義する
2. @Stateで変更があるたびにViewが変更されるようにする
3. TabViewでselectionをBindingする

WatchOSでは,TabViewを宣言しておくとWatchOS側でPageとして表示処理をしてくれます.
また,NowPlayingViewはWatchKitをimportすることで,使用できる音楽再生などを管理することができるアプリです

* サンプルコード
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

### MetricsView(運動中の消費カロリー等の詳細情報の画面の作成)
今回の肝であるリアルタイムにデータを取得して表示する画面を実装します. 
表示するデータの項目は, ①心拍数,②距離,③運動時間,④消費カロリーとします.

MesurementViewの構成要素は,Timerとそれぞれの指標を表示するTextで構成します.
経過時間を表示するViewは,ElapsedTimeViewとして子Viewに切り出します. 

* 各指標の実装
距離や消費カロリーは,それぞれをMesurementで,適した単位などで実装するようにします. 
MesurementクラスからFormatを指定する際には,実装時には下記の記事などが参考になりました.

https://cocoacasts.com/swiftui-essentials-working-with-units-and-measurements-in-swiftui

* サンプルコード
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

ElapsedTimeViewとして経過時間を表示するViewをさくせいします.
実装の詳細は, こちらに別の記事として投稿しているので気になる方はそちらをみていただければと思います.

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
* 実装した画面
![](/images/article8/metricsView.png)


### 運動の中断,終了を選択するView
運動の終了または,中断を選択する画面としてボタンをじっそうします. 
Flagは,WorkoutManagerの作成後差し替えます. また,Workout終了後にSummaryViewを表示する処理を実行します. 

* サンプルコード
```swift
import SwiftUI

struct ControlsView: View {
    @State private var flag = true
    var body: some View {
        HStack {
            VStack {
                Button {
                    // workoutを終了する
                } label: {
                    Image(systemName: "xmark")
                }
                .tint(Color.red)
                .font(.title2)
                Text("End")
            }
            VStack {
                Button {
                    // 運動の記録を中断する
                    flag.toggle()
                } label: {
                    Image(systemName: flag ? "pause" : "play")
                }
                .tint(Color.yellow)
                .font(.title2)
                Text(flag ? "pause" : "play")
            }
        }
    }
}
```
* 実装した画面
![](/images/article8/controlView.png)


## 1-3.運動終了後のSummaryを表示するViewの作成
運動終了後に表示するSummaryViewを作成します. 
今回は,各指標をSummaryMetricViewという子Viewから構成する形式としました.
表示するデータの形式は,MetricsViewと同様にMesurementでデータの単位を指定します.
下記のサンプルコードでは,ActivityRingsViewをまだ作成していないため,Errorが表示されると思います. 次にそのAcitivityRingの作成方法を紹介します.
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

### AcitivityRingsを表示してみる
AcitivityRingとは,消費カロリーや移動距離などを可視化したRingです. 
![](/images/article8/ringsample.png)

実はこのAcitivityRingは,HelthKitを利用することでApp内で表示することが可能です. 


* 詳細の実装手順
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
healthStoreに対して,作成したqueryを実行します.
```swift
healthStore.execute(query)
return activityRingsObject
```
* 公式Doc: Acitivity Summaryクエリの実行方法

https://developer.apple.com/documentation/healthkit/hkactivitysummaryquery/executing_activity_summary_queries

* 全体のサンプルコード
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


AcitivtyRingの実装が完了したら,シュミレータでこのようなSummaryViewが表示されると思います. 
![](/images/article8/summary.gif)

次のChapterでは,WorkoutManagerを