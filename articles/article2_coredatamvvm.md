---
title: "Core Date+ SwiftUIをMVVMを実装する"
emoji: "😺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["SwiftUI","Swift","iOS"]
published: false
---




# 実装までのフロー

NSManagedObjectのコード生成
ジェネレイトしたので完了


Model作成
要するにトランザクションの処理
Controllerで定義する自動で作成される
過去のライフサイクルとは異なる

Modelの実装要件
* シングルトン
* Class内でデータのCRUDを行う
* ExtenstionでCloudKitContainerの保存処理,rollback処理を実装する


ViewModel
一覧画面と登録画面でそれぞれ1つずつ作る

* 一覧画面
@Published でListViewに渡したいモデルを定義する
```swift
//
//  UserListViewModel.swift
//

import SwiftUI

class UserListViewModel: ObservableObject {
    
    /// ユーザー情報一覧
    @Published var users: [User]
    
    /// 初期化
    init() {
        self.users = []
    }
    
    /// データ全件取得
    func fetchAll() {
        users = CoreDataModel.allUsers()
    }
    
    /// 表示イベント
    func onAppear() {
        fetchAll()
    }
    
    /// 行削除イベント
    func onDeleteRows(offsets: IndexSet) {
        
        offsets.forEach { index in
            CoreDataModel.delete(users[index])
        }
        
        CoreDataModel.save()
        
        users.remove(atOffsets: offsets)
    }
}

```

* 登録画面
  Inputのvalidationを行う
  1. @ObservedObjectで登録するデータを受け取る
  2. Alertの表示判定を返す
  3. データの登録を行う

## 参考にした資料
https://www.2nd-walker.com/2020/03/24/
how-to-bind-coredata-to-swiftui-in-mvvm/



責務の分解をシてみる
VMにはEntityに対して実施する更新処理等は書かない
