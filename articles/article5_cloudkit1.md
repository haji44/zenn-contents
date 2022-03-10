---
title: "SwiftUIでCouldKitを使ってみたシリーズ1!"
emoji: "💭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["iOS","Swift","Cloud","Backend"]
published: true
---


# CloudKitを使いたい
FireBaseのその前に,Apple純正のBackend as a ServiceであるCloudkitを使ってみました.
今回は概要から最初のSetUPまでをまとめます.

# CloudKitとは?
まず,CloudKitを利用する際の利点と弱点を簡単にまとめてみようと思います.

## CloudKit Pros&Cons

|  | Pros | Cons |
|:---:|:---:|:---:|
| ユーザー認証 | 自動でやってくれるログインスクリーンいらん. |  特になし |
| 他サービスとの連携 | 他のAppleデバイスとの連携も楽 ユーザー間でのシェア機能もあり | 他のアプリと連携できない. CSVなどのデータの取り込みも不可 |
| 費用 | 上限までは無料かつ上限もデカめ<br>(詳細は料金表) | Developperアカウントが必要<br>(1万2千円: 2022/3現在) |
| セキュリティー | Privacyデータベースを使えば秘匿化され開発者からもみられない | 特になし | 

* 料金表
[![料金表](/images/article5/price.png)](https://blog.back4app.com/what-is-cloudkit)

僕は実際にお試しで使う予定だったので,FireBaseなどGoogleのサービスを使うのをためらっていたので,無料で使用できるというのが結構決め手になりました.


# CloudKitを利用する際の登場人物を紹介するぜ!
[![](/images/article5/cloudkit_bigpicture.png)](https://developer.apple.com/icloud/cloudkit/designing)
## Container
Containerは1アプリひとつ存在しています. このコンテナーの中にDataBaseとして3つのタイプがあります

|データベース|できること|
|:---:|:---:|
|Public|アプリをインストールしている全ユーザーがアクセスできるデータベース|
|Private|iCloudアカウントごとにアクセスが制限される. 開発者でも中身は見れないので安全|
|Shared|Privateの中でもユーザーが許可したデータを他のユーザーに<br>共有することができるデータベース|



## CKRecord   
Data ModelとしてEntityにPropertyを定義して扱います. Dictionaryとしてkeyを設定する必要があります.
@[card](https://developer.apple.com/documentation/cloudkit/ckrecord)


## CKReference 
Core Dataで言うところのRelationに当たる機能になります.CKRecord間でPropertyが存在しているかなどをチェックする際に利用します.
  https://developer.apple.com/documentation/cloudkit/ckreference

## CKOperation
DataのFetchなど,いわゆるCRUDを処理を行うクラスです.
  https://developer.apple.com/documentation/cloudkit/ckoperation


# Projectの設定
Projectを立ち上げたら,Target → Siging & Capability → Capabilityの順に遷移して,iCloudを追加します.

詳細の設定方法はこちらのやり方を参考にしました.
https://developer.apple.com/documentation/cloudkit/enabling_cloudkit_in_your_app

最終的に完了するとこのような表示になります. 
![](/images/article5/SettingCloudKit.png)

:::message alert
Apple Developperに支払いを終えて登録が完了していないと上記のiCloudを選択することができません.また,土日は対応してもらえないのでちゃちゃと払ってしまうと良いと思います!
:::

# ダッシュボードを見てみよう
ProjectのCloudKit Dashboardをクリックするとダッシュボードに遷移します.
ホーム画面に4つメニューが表示されます.
![](/images/article5/dashboard.png)

今回はDatabaseの設定を見たいので,CloudKit Databaseの画面に遷移します.
この画面で先程追加したアプリのコンテナが表示されていると思います.この中でデータベースやRecordの定義を行っていきます
![](/images/article5/dabaseselect.png)

色々設定てんこ盛りという感じですが,ここの3つを抑えておけば最初はいいと思います
![](/images/article5/settingIntro.png)

# まとめ
初歩の初歩といった内容でしたが,これからCloudKitを使いたい方の理解の助けになれば幸いです.
次はDataの登録からFetch処理までをまとめたいと思います.


# 参考資料
https://developer.apple.com/icloud/cloudkit/designing/

https://www.youtube.com/watch?v=mXGg-iwZNUA
