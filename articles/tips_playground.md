---
title: "Playground MarkdownのCheet sheetを作ってみた"
emoji: "🤖"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["Swift","Playground","Xcode"]
published: true
---


# Playground Markdown
TutrialでよくPlaygroundがきれいにまとまっていて便利
Combineの文法をメモするのに利用したいと思ったのでまとめる


サンプルのプロジェクトはこちらから
https://github.com/haji44/PlaygroundCheet
## 有効にする方法
Editro -> Show Renderd Markupをチェックする
MarkDownが認識される
![](/images/tips1/renderMark.png)



# Cheet Sheet
基本的な文法
```swift
//:# Header
//:## Header2
//:### Header3

/*: Bullet points
+ list item 1
+ list item 2
+ list item 3
*/

/*: Numbered lists:
1. list item
1. list item
1. list item
*/

/*:
- *Italic fonts* with a single _leading_ and _trailing_ * or _
- **Bold fonts** with a **double leading and trailing** ** or __
*/

/*:
 \````
 let indices1 = Array(0..<10)
 let indices2 = Array(0..<5)

 for i in indices1 {
 for j in indices2 {
 print (i*j)
 }
 }
 \````
*/


/*:
There are just a few link types:

 - [external playground link with a link description](http://www.allaboutswift.com)
 - [internal playground link pointing to a page named "page"](page) **or**
 - internal playground link pointing to the **[previous page](@previous)** or **[next page](@next)**
 */


/*:
 - callout(Custom callout): FYI only
 - experiment: Let's see ...
 - note:What's that?
 - important: Don't forget this !
 - example: Something like this
*/
```

* 新規ページの追加方法

![](/images/tips1/2022-03-13-12-05-19.png =10px)

* ページの順序
File Inspectorで並んでいる順で使う


# Tips紹介のコーナー
## 使うと便利な機能 その1 ショートカット
Previewなどがないので,MarkDownのレンダリングを切り替えたいときが発生する
Show Rendered Markupにキーボードショートカットを割り当てると切り替えが簡単!
![](/images/tips1/shortcut.png)

## 使うと便利な機能その2 Codeスニペット
各ページ共通のコード,例えばPlaygroundページ間でのNavigationなどをなんども書くのは面倒なので登録する
例として,Header部分に実装する内容をスニペットに登録する

* プレビューのイメージ
![](/images/tips1/imageHeader.png)
* コード
```swift
/*:
 [<- Previeous](@previous)                [Home](Home)                       [Next ->](@next)
 # PageTitle
*/
```
### 作り方の手順
1. 登録したいコードを選択した状態でCreate Code Snipetをチェックする
![](/images/tips1/createsnipet.png =250x)
:::message alert
コードを選択していないと新しく作成する画面に遷移しません!!
必ず選択してからチェックしましょう!
:::

2. 編集画面が表示されるので,項目ごとの設定
![](/images/tips1/editSnipet.png)

詳細な作り方はこちらが参考になりました
https://dev.classmethod.jp/articles/create-code-snippet/


# 参考資料
https://www.youtube.com/watch?v=iy-sG8OGT9M&t=1040s