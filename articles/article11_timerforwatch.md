---
title: "SwiftUI mm秒単位で時間を表示させる方法"
emoji: "👌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Swfit","Xcode","SwiftUI"]
published: true
---

# 100分の1秒単位で更新されるTimerを作成する
Timerを作成する際に,Defaultでは秒までの時間が扱われることになります. 
運動中のタイマーなどは,より細かい秒以下の単位を表示させることが必要な場面があったので,Formatterをうまく使って表示させる方法を紹介します. 

## 今回の目標物
今回は,SwiftUIを使ってタイマーを作成します. 
以下 実装の手順になります.

1. DateComponentsFormatterの設定
2. 経過時間を指定のFormatに変換するメソッドの設定
3. 表示させるView側の設定


# 1.DateComponentsFormatterの設定
時間の表示形式を設定するCustomeのFormatterClassを作成します.
まず,DateComponentsFormatterを設定します`allowedUnits`は`yaer` や`day`などで構成されており,指定されている以外の単位を指定したときに例外を返します.
https://developer.apple.com/documentation/foundation/datecomponentsformatter/1410216-allowedunits

`zeroFormattingBehavior`は値が0の場合の挙動を決定します. `pad`を指定すると0のときであっても値が表示されます. 


```swift
import Foundation

class ElapesedTImeFormatter: Formatter {
    let componentFormatter: DateComponentsFormatter = {
        let formatter = DateComponentsFormatter()
        formatter.allowedUnits = [.minute, .second]
        formatter.zeroFormattingBehavior = .pad
        return formatter
    }()
}
```

# 2. 経過時間を指定のFormatに変換するメソッドの設定
Formatterクラスの`String`への変換メソッドをoverrideします. 
今回は,TimeIntervalを文字列に変換して,1/100秒まで表示をさせます.
%は文字列として出力するという意味を持ちます. 
%@はObjective-C objectを出力します
%0.2dは,dが代入する値がIntであることを表し,0.2は.のあとの数値が表示する小数点の位置を示します. 
```swift
import Foundation

class ElapesedTImeFormatter: Formatter {
    let componentFormatter: DateComponentsFormatter = {
        let formatter = DateComponentsFormatter()
        formatter.allowedUnits = [.minute, .second]
        formatter.zeroFormattingBehavior = .pad
        return formatter
    }()

    override func string(for obj: Any?) -> String? {
        // TimeInterval(Double型)にCast
        guard let time = obj as? TimeInterval else { return nil }
        // 先程作成したFormatterからcastしたTimeIntervalを文字列に変換する
        guard let formattedString = componentFormatter.string(from: time) else {
            return nil
        }
        // 1/100秒を算出して,
        let hunredths = Int((time.truncatingRemainder(dividingBy: 1)) * 100)
        let decimalSperator = Locale.current.decimalSeparator ?? "."
        return String(format: "%@%@%0.2d", formattedString, decimalSperator, hunredths)
    }
}
```

# 3. 表示させるView側の設定
経過時間をTimeIntervalとして,@Stateで宣言します. 
@Stateがない場合には,下記のErrorが出ます. 
`Left side of mutating operator isn't mutable: 'self' is immutable`

Textで表示をする場合には,Formatterのかけ方とFontの2つに注意をする必要があります.
```swift
Text(NSNumber(value: timeInterval), formatter: formatter)
                .font(Font(UIFont.monospacedDigitSystemFont(ofSize: 50, weight: .light)))
                .onReceive(timer) { _ in
                    timeInterval += 0.01
}
```
* Formatterのかけ方
valueにはNSObjectを継承したObjectである必要があります. formatterはFormatterクラスを継承したTimerFormatterを渡します．
https://swiftontap.com/text/init(_:formatter:)-54e07

* Fontの指定
Fontの指定もしくは,Textの位置を左寄せにするなどしない場合には,下のように時間がずれる表示なります. 
![](/images/article11/timer.gif)

これを防ぐために,monospacedDigitSystemFontを指定します. 
https://omochiblog.com/2021/01/31/swiftui-sf-monospace/

Fontの指定でこのように表示位置が固定されます
![](https://github.com/haji44/TimerViewSampel/blob/main/Simulator%20Screen%20Recording%20-%20iPhone%2012%20-%202022-04-02%20at%2021.09.09.gif)


全体のコード
```swift
import SwiftUI

struct ContentView: View {
    @State private var timeInterval: TimeInterval = 0

    private let formatter = TimerFormatter()
    private let timer = Timer.publish(every: 0.01, on: .main, in: .common).autoconnect()

    var body: some View {
            Text(NSNumber(value: timeInterval), formatter: formatter)
                .font(Font(UIFont.monospacedDigitSystemFont(ofSize: 50, weight: .light)))
                .onReceive(timer) { _ in
                    timeInterval += 0.01
            }
                .padding()
    }
}
```

# まとめ
自分は,初めてTimerを実装する際に,
* 1秒より細かい単位を表示したい
* 時間表示の文字ズレを無くしたい
という2点でつまずいてしまいました.

この記事が,SwiftUIでの実装例として参考になれば幸いです!
サンプルのレポジトリです!
https://github.com/haji44/TimerViewSampel
