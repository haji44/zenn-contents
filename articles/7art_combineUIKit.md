---
title: "Swift CombineCocoa入門 UIKit編"
emoji: "🦁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["iOS","Combine","Swift","Reactive Programming"]
published: false
---


# Combineに入門しつつUIKitをいじる!
Combineを使って,非同期処理を色々試してみる
今回は,Switchでフラグを変換できる足し算アプリを作ります
題材としてはこちらの動画を参考にしました
<!-- TODO: アプリ道場の課題の動画を書く -->
https://www.youtube.com/watch?v=dqYijANnyLc
# CombineCocoa
UIKitのComponentsに対してCombineが簡単にかける,CombineCocoaというフレームワークを活用していきます!


# 実装前の準備
## SPMから導入

下記のリンクからパッケージを導入します
https://github.com/CombineCommunity/CombineCocoa


## データのやり取りの確認
今回は,入力した内容がラベルに反映されること
ラベルの内容が,Switchの状態によって正負が反転するようになるということを条件にします
<!-- TODO: ここにシークエンスダイアグラムを作る -->

1. 数値の入力をPublishする
2. Switchの状態を確認する
3. combineLatestでどちらかの入力があった際にそれぞれの値を確認してPropertyとLabelを更新する


下記は,ちょっと応用的なないようなのでスキップしても大丈夫です.
より実践的なテクニックとして挑戦することにしました.
* その他考慮しないと行けない点
  * データのバリデーション
    入力制限として下記の3つを確認します.該当した場合には,Alertを表示します
    1. 入力が空
    2. 入力がテキスト
    3. 先頭に0を入れる
    4. 入力が7桁未満

  * ボタンのDisable
    今回は,データのバリデーションを満たさない場合にはボタンが有効にならないように設定を行います


# 実装!!!

1. Proprtyの定義
今回は,左右の数値をそれぞれ@Publsihed Propertyとして保持します
@Published を使うと,指定したプロパティーの値がせってされたときにイベントを発行されることになります



2. subscriptionsの定義
3. Combine Publisherの定義

4. ButtonにオブジェクトをAssing
```swift
buttonSubscription
.recive(on: RunLoop.main)
.assign(to: \.isEnabled, on: signupButton)

```
* RuLoop.mainは何をしているか
SwiftLeeの記事内では,下記のように示されています
> What is RunLoop.main?
> A RunLoop is a programmatic interface to objects that manage input sources, such as touches for an application. A RunLoop is created and managed by the system, who’s also responsible for creating a RunLoop object for each thread object. The system is also responsible for creating the main run loop representing the main thread.

要するにRubLoopがタップなどのインプットをプログラマティックに処理するインターフェイスということでした.
まぁよくわからないですね...



* assignは何をしているのか
サブスクリプションの結果に変更があった際に,onで渡したObject内のプロパティーをto:で指定する
指定はKeyPathという方法を使っている
keypathについてはこちらにちょろっとまとめましたTweet






# 参考資料
CombineCococaの解説記事
https://dev.classmethod.jp/articles/combinecocoa/

Combineの解説記事
https://www.youtube.com/watch?v=UvXvFa-k6dw
https://www.avanderlee.com/combine/runloop-main-vs-dispatchqueue-main/

RunLoopの解説記事
https://prafullkumar77.medium.com/ios-run-loop-what-why-when-7febead400b7