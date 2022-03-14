---
title: "SwiftUIでCouldKitを使ってみたシリーズ2!"
emoji: "😸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["iOS","Swift","Cloud","Backend"]
published: true
---
# CloudKitを使いたい
前回はCloudKitの簡単なセットアップとしてプロジェクトからContainerの作成まで行いました.
今回は,Dashboard上でデータベースを作成して,アプリ側でデータを受け取るところまでを取り上げます.
前回のリンクはこちらから
https://zenn.dev/jime/articles/article5_cloudkit1

# 今回の完成イメージ
画面右上のツールバーのボタンを押すとCloudKitからデータを取得します表示したリストからは,お店のHPに遷移するLinkを設定します.
![](/images/article7/samplescreen.gif)

# CloudKit Dashboardでデータの登録
## Modelの定義
今回は下記の2店舗の情報を直接CloudKitのデータベースに書き込みます

| name   | street                     | website                                                                                   | phone\_number | latitude    | longitude   |
| ------ | -------------------------- | ----------------------------------------------------------------------------------------- | ------------- | ----------- | ----------- |
| とみ田    | 千葉県松戸市松戸1339　高橋ビル1F        | [https://www.tomita-cocoro.jp/about\_us.html](https://www.tomita-cocoro.jp/about_us.html) | 047-368-8860  | 35.78200511 | 139.9009887 |
| 麺屋翔みなと | 東京都新宿区西新宿1-26-2 新宿野村ビル地下2階 | [https://menya-sho.co.jp/shop/minato.html](https://menya-sho.co.jp/shop/minato.html)      | 03-6304-5242  | 35.69302059 | 139.6954082 |

:::message
CloudKitの弱点がこの他のデータを取り込むことがやりづらいという点です
テストデータなどの注入はCLIツールなどからできる様になっているようですが,ダッシュボードからCSVなどのファイル形式を読み込むことは現状できていないようです.
:::


## Dashboard上でSchemeを定義する
アプリの全ユーザーがアクセス可能なようにPublicのデータベースを作成します.

![](/images/article7/newrecord.png )


### 1. Recordの定義
レコードのPropertyの定義を行います.
各PropertyのNameとTypeを指定します. 今回は画像を,Assetとして指定をします.AssetはCKAsset型として扱われ,アプリ側でAsseet型をImageに変換する処理を行う形になります. 
![](/images/article7/setting.png )

### 2. Indexの作成
レコードのIndexを指定します.
クエリが実行可能なように,record_nameに対して,Querableを指定します. また,今回は店舗名でソートができるようにしたいので,nameをSortableとしてIndexを指定します.
![](/images/article7/setupindex.png )

:::message
Querableを設定しなかった場合には,検索ができなくなるため,注意しましょう!
:::
### 3. データの挿入
先程の表にまとめた情報をモデルの定義に沿って,入力していきます. 画像は,ファイルフォルダーから選択する形になります.LocationはLat/Lonを一括で登録する事ができます.
![](/images/article7/insertdata.png =240x)

:::message
ここで登録する写真の縮尺は,統一したほうが後のViewの表示を調整しやすくなります．
:::

## 4. データの確認
登録が完了したらクエリを実行して登録したデータを確認してみましょう. 無事に2件分のデータが保存されてQueryの実行結果として表示されることが確認できました.
![](/images/article7/checkrecord.png)
# CloudKitのデータを利用するための実装

## 定義したModelの定義
先程の定義したRamenShopを扱うためのModelを定義します
下記の3つの手順を行います.
```swift
import Foundation
import CloudKit

struct RamenShop {
    // ①Propertyの定義
    // MARK: - Property
    let ckRecordId: CKRecord.ID
    let name: String
    let street: String
    let website: String
    let phoneNumber: String
    let location: CLLocation
    let photo: CKAsset! // イメージではなくAsset

    // ②Keyの定義
    // MARK: - KeyValue
    static let kName = "name"
    static let kStreet = "street"
    static let kWebsite = "website"
    static let kPhoneNumber = "phoneNumber"
    static let kLocation = "location"
    static let kPhoto = "photo"

    // ③Initialzierの定義
    // MARK: - Init
    init(record: CKRecord) {
        ckRecordId = record.recordID
        name = record[RamenShop.kName] as? String ?? "N/A"
        street = record[RamenShop.kStreet] as? String ?? "N/A"
        website = record[RamenShop.kWebsite] as? String ?? "N/A"
        phoneNumber = record[RamenShop.kPhoneNumber] as? String ?? "N/A"
        location = record[RamenShop.kLocation] as? CLLocation ?? CLLocation(latitude: 0, longitude: 0)
        photo = record[RamenShop.kPhoto] as? CKAsset
    }
}
```
### 1. Propertyを定義する
Dashboardで登録した内容と一致するように,各Propertyを定義します.
注意点としては,レコードのIDとしてCKRecord.IDを定義しています.これは,各Objectの持つユニークなIDで,CloudKitがデータの生成時に自動的に付与するものです.

### 2. Keyの定義
CKRecordはDictionaryなので,Keyを指定して値を取り出します.Keyは,Dashboardで設定したPropertyのNameと完全に一致するようにしなくてはなりません.
今回は,keyをStaticな定数として定義して扱うことにしました.

:::message
Keyの入力内容はPropertyのNameと完全に一致するようにしなくてはなりません.大事なことなので2回書きました.
:::

### 3. Initialzierによる初期化処理
レコードの初期化を行います.この処理はJSONのパースを行うのとどのようの手順になるかと思います.recordを値をキャストして各Propertyに割り当てています.



## CloudKitManagerの作成
CloudKitに対してデータの取得処理を担うクラスを作成します.
Async/ AwaitでCloudKitに対して,先程の登録したPublickDatabaseから全件データを取得してみます.

```swift
import CloudKit

enum RecordName {
    static let ramenShop = "RamenShop"
}

struct CloudKitManager {
    static let shared = CloudKitManager()

    private init() { }
    static let container = CKContainer.default()

    static func getShopInfo() async throws -> [RamenShop] {
        // ① クエリの生成
        let sortDescriptor = NSSortDescriptor(key: RamenShop.kName, ascending: true)
        let query = CKQuery(recordType: RecordName.ramenShop, predicate: NSPredicate(value: true))
        query.sortDescriptors = [sortDescriptor]
        // ② データベースに対してqaueryの実行
        let (matchResults, errorn) = try await container.publicCloudDatabase.records(matching: query)
        // ③ 取得結果のパース処理
        let records = matchResults.compactMap { _, result in try? result.get() }
        return records.map(RamenShop.init)
    }
}
```

**1. クエリの生成**
下記のコードではCloudKitのデータベースに対して,検索クエリを生成しています.NSSortDescriptorは,レコードのkeyに対して,降順または昇順で並べ替えを行います.
CKQueryの概要はこんな感じになります↓ 今回はnameを昇順で表示するためにsortDescriptorsを指定します.

| プロパティー: 型                                 | 概要                    |
| -------------------------------------- | --------------------- |
| recordType: CKRecord.RecordType        | 検索対象のレコードタイプの名前       |
| predicate: NSPredicate                 | 検索条件 SQLでいうWHERE句にあたる |
| sortDescriptors: \[NSSortDescriptor\]? | 並べ替えの条件を指定する          |
```swift

let sortDescriptor = NSSortDescriptor(key: RamenShop.kName, ascending: true)
let query = CKQuery(recordType: RecordName.ramenShop, predicate: NSPredicate(value: true))
query.sortDescriptors = [sortDescriptor]
```

**2. データベースに対してqaueryの実行**
container内のPublicデータベースに対して,クエリを実行します.実行結果の返り値として,検索結果とErrorが返されるためTupleで結果を受け取ります.
```swift

let (matchResults, error) = try await container.publicCloudDatabase.records(matching: query)
```
**3. 取得結果のパース処理**
取得結果のmatchResultsは,レコードのIDと取得したRecordの結果をResult型で保持しています.
![](/images/article7/typeinfo.png )
今回はIDは必要ないので,resultのみを取得したいと思います.resultの値はgetメソッドを実行し,失敗した場合にはtry?でnilを返すようにします.
```swift
// MARK: getの定義
func get() throws -> Success
```
compactMapはnilを除外するため,getメソッドが失敗した場合にはrecordは空になります.最後にRamenShopの初期化を実行して,配列として結果を返します.
```swift
let records = matchResults.compactMap { _, result in try? result.get() }
return records.map(RamenShop.init)
```


## Viewの実装
画面に取得結果を表示するまでを確認したいので,今回は画面左上のツールバーのボタンを押すとCloudKitと通信を行うことにします.
重要な点としては,Async/Awaitで非同期処理を実行するため,Task内で今回の処理を実行する必要があります.取得結果は,shopInfoに渡すことで,リスト内にショップ名とリンクのボタンが表示されます.
```swift
import SwiftUI

struct ContentView: View {
    @State private var shopInfo: [RamenShop] = []

    var body: some View {
        NavigationView {
            List {
                ForEach(shopInfo, id: \.self) { shopInfo in
                    HStack {
                        Text(shopInfo.name)
                        Spacer()
                        Link(destination: URL(string: shopInfo.website)!) {
                            Image(systemName: "map.fill")
                        }//: LINK
                    }//: HSTACK
                }//: FROEATCH
            }//: LIST
            .navigationTitle("ラーメン情報")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button {
                        // CloudKitのクエリの実行を行う
                        // Asycに対応するためにTask内で実行する
                        Task {
                            do {
                                shopInfo = try await CloudKitManager.getLocation()
                            } catch let error {
                                print(error)
                            }
                        }
                    } label: {
                        Image(systemName: "globe.badge.chevron.backward")
                            .imageScale(.large)
                    }

                }
            }
        }//: NAVIGATION
    }//: BODY
}

```

# まとめ
いかがでしたでしょうか? 取得処理でしたが比較的シンプルに取得ができたと思います!
次回は,取得したCKAssetのImageへの変換処理と位置情報をMapViewへの表示処理について取り上げたいと思います.
