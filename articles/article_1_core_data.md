---
title: "[Xcode] NSCFSet objectAtIndex unrecognized selector の解決方法"
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Swift,CoreData,XCode]
published: false
---

## Errorの内容
CoreDataでリレーションを設定しているときに出現したError
下記のようなリレーションをCoreDataで再現しようとした
![](\/images/article_1/coredata_setting.png =250x)

データを保存するCoreDataでよくに遭遇してしまうので,状況別の対処方法をまとめてみた
あたりをつければ,どこに問題があるかすぐに見つけられるかも?

## 発生した状況
Listに紐づく,Reminderを設定しようとした際にクラッシュした


## 発生した原因
1. Errorのメッセージそのままで調査してみる
   Errorの原因は,引数がNSArrayとして設定しているのにも関わらずSetを返しているというのが問題だった
   https://stackoverflow.com/questions/1814217/nscfset-objectatindex-unrecognized-selector-iterating-works
2. そういえば...
   CoreDataのEditorメニューで生成したときにNSSetで自動的にCode生成されていたということを思い出し,DataModelの設定を確認しました
3. Arrangementお前だったのか
   Relationの設定を注意深く見ていくと下記の設定を発見しました.


### Arrangementってなんぞや
このArrangementはDocumentではこのように書かれていました.
つまり
NSSetを返すときにはこれをOnにしないとハンドリングをされないためErrorになるということでした.
https://developer.apple.com/documentation/foundation/nsorderedset

### 解決方法
DataModelの設定から変更


buildして無事に動きました!!

gti