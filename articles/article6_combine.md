---
title: "Swift Combine初学者が躓いたポイントまとめ "
emoji: "😽"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# Combine入門
Combineに入門したけど,Tutrialで知らないことがいっぱい出てきたので,備忘録的に概念をサマっていく


## What is Stream?



Streamとはユーザーによって生じたUI上のイベントや,ネットワークコールなどいろんな種類の非同期なデータのやり取りという感じ

* 例
1. UIButtonのタップの一例
2. 

# What is the difference between Upstream and Downstream?



## What is Operation?
* Seaquenceっていう

Combine のOperationがどのような順番でStreamを処理するのかというのをまとめているもの

Rx版
https://rxmarbles.com/

Swift版
https://github.com/robertpalmer/CombineMarbles

## Why does end of method call .eraseToAnyPublisher()?

* つまづきポイント
型消去毎回してるけど,そもそもメリットがわからなかった

* 調査結果
タイプをAnyPublisherにすることで他のPublisherとのやり取りが楽になります
//ここに例を
1. 型を消去しない場合のMapの処理
2. 型を消去している場合のMapの処理

めっちゃ便利やんか

## What is the reason to store subscription into a subscriptions set?
* つまずきポイント
Tutrialでよく.store(in:) というメソッドが出てくるけど,なんで?
そしてSetを保存しておく必要って何なの?となったので調べた

* 調査結果
  こちらのStackOverFlowには,サブスクリプションを保存しておきそれがオブジェクトが破棄されるまでの間で保持するためとうことでした
  多分,これがないとイベントが発生後に継続してPubslishができないからそれを防ぐ的な感じだと思っています.
  
sink の戻り値の subscription に対して何もしないで破棄してしまうと、subscribe で指定した受信処理が破棄されます。store メソッドを使うと、subscription を保持できます。これによって、受信処理が保持されます。store メソッドを使っている リスト1.1 では受信処理が期待どおりに実行されます。
ちなみに下記の処理で,.storeをしないとそもそも動かないです.
```swift
private var subscriptions = Set<AnyCancellable>()

future
  .sink(receiveCompletion: { print($0) }, receiveValue: { print($0) }) 
  .store(in: &subscriptions)

```
https://stackoverflow.com/questions/63543450/what-is-the-reason-to-store-subscription-into-a-subscriptions-set

# まとめ
他にも随時参考になる記事などをまとめていきたいと思います!