---
title: "ColorLiteralを Xcode13 で使う "
emoji: "📑"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["Xcode","Swift","Playground","iOS"]
published: false
---




# 発生した問題
この記事では下記の環境で発生した事象に関しての内容になります(2022/03/11現在)

* 動作した環境
```
Xcode: 13.2.1
macOS: 12.0.1（21A559）
Machine: Mac mini(M1, 2020)
```
Xcode13.2.1を利用中にColor Literalが表示できないという問題が発生しました
本来は,下記のように入力することで,自動でColorLiteralが表示されていましたが,Xcode13では動作が異なっていました
![](/images/resolution1/error.png)
↓同じような問題
https://www.reddit.com/r/SwiftUI/comments/psbxix/color_literal_missing_on_xcode_13/

* [ColorLiteralについてはこちらの記事が参考になりました](https://medium.com/swift-column/color-literal-6223850f7e2c)

https://medium.com/swift-column/color-literal-6223850f7e2c
# 解決方法
Colorの引数に入れるときにうまく作動しないということが起きているようだった.
下記のように打ち込むと自動で表示してくれるので,Xcode13でも使えます
```swift
#colorLiteral(
```
今は暫定的に,外で定義した値を渡すというのがいいかも
![](/images/resolution1/screen.mov)

