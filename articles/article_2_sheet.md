---
title: "SwiftUI modal Viewの表示方法を使い分ける!"
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [SwiftUI,Xcode]
published: false
---


## SwiftUIでmodal view表示の方法いっぱいあるけどどう使わける?
SwiftUIを本格的に書き始めて,約1ヶ月立ったので数多あるモーダルViewの表示方法に蹴りをつけたい

ここでは3つの方法を紹介して,それぞれ使っているシュチュエーションとメリデメなどをまとめる

### 方法1 親Viewで@StateでBoolを保持する
ボタンもしくは,ontapのイベントを受け取ってsheetにわたす方法

子 
```swift
struct NewReminderButtonView: View {
  @Binding var isShowingCreateModal: Bool
  let reminderList: ReminderList
  
  var body: some View {
    Button(action: { isShowingCreateModal.toggle() }) {
      Image(systemName: "plus.circle.fill")
        .foregroundColor(.red)
      Text("New Reminder")
        .font(.headline)
        .foregroundColor(.red)
    }.sheet(isPresented: $isShowingCreateModal) {
      CreateReminderView(reminderList: reminderList)
    }
  }
}
```
* いつ使う
* Pros
* Cons


### 方法2 

* いつ使う
* Pros
* Cons