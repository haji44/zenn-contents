---
title: "SwiftUI Timerをミリ秒まで表示する!!"
emoji: "📌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["SwiftUI","Swift","Xcode","iOS"]
published: false
---

# SwiftUI
SwiftUIのチュートリアルでよくあるタイマーが秒単位でカウントダウンしている物が多く,もっと危機迫る感じを演出するために,
ミリ秒まで表示させるしかないだろ!!!ということで作ってみました

いい感じのGIFをここにはる

## 実行環境
下記の環境で動作した内容となっております
* Mac mini: M1, 2020
* macOS: Monterey: 12.0.1
* Xcode: Version 13.2.1 (13C100)


# 実装の手順
実装の流れは大きくわけるとこんな感じになります
1. 現在時刻とターゲットになる時刻の差分を求める
2. Timerが減っていく時間を設定する
3. 表示するデータを切り分ける


## Viewの実装
今回は残時間によって表示する色を変えてみる
    var body: some View {
        Text(countDownString(from: referenceDate))
            .font(.largeTitle)
            .onAppear(perform: {
                _ = self.timer
            })
    }



## Timer Propertyの定義
```swift
    @State var nowDate: Date = Date()
    let referenceDate: Date
    var timer: Timer {
        Timer.scheduledTimer(withTimeInterval: 0.1, repeats: true) {_ in
            self.nowDate = Date()
        }
    }
```

## Timer CountDownのLogicの定義


# まとめ
