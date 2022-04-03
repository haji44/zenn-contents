---
title: "はじめに"
---

# HelthKitを使ってWatchOSで動かす!
この記事では,運動時のデータをリアルタイムに記録するアプリをWatchOSで作成します. 
リアルタイムなデータの更新,取得はHKWorkoutSessionを利用することができます. 

:::message
実装の内容は,WWDCのWorkoutアプリを開発するセッションをもとにしています.
:::
## 完成のイメージ
![](/images/article8/finish.gif)

## 画面の遷移のイメージ
![](/images/article8/apparchitecture.png)

* 使用するライブラリ
1. SwiftUI→ Viewの作成
2. HelthKit→ 運動時のデータ取得
3. WatchKit→ MediaPlayerの設定に使用

# 実装手順
今回は下記の手順に沿って,実装を進めます. 

1. Viewの作成(Viewを作っているだけなのでSkipしても問題ありません)
2. WorkoutManagerの設定
3. WorkoutManageからViewへのDataの受け渡し