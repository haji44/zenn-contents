---
title: "HealthKit入門"
emoji: "🍣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---
# HealthKit とは?
Healthはいくつかのデータで構成されている
![](2022-03-24-01-13-00.png)
![](2022-03-24-01-12-07.png)



# Set Up HealthKit
1. パーミッション
2. プラットフォーム別の対応
3. HKHealthStoreの対応

![](/images/article8/capability.png)



![](/images/article8/HealthData.png)

## パーミッションに必要なinfo.plistのkey
![](/images/article8/infokey.png)
アプリが,いつそしてどのようにデータを共有するのかを明示する必要がある


# 取得したデータの保存
データの取得から保存の流れ3ステップ

1. 書き込みの許可を取る
書き込み限定であることに注意.アクセスが失敗した場合には再度アクセスを試みるなど

![](2022-03-24-01-03-32.png)

2. HKQuanity型でデータを作成する
値とUnitを指定する事ができる. 
![](2022-03-24-01-05-00.png)

3. HealthKitに保存する
データベースにHKQuantityを保存することができる
![](2022-03-24-01-05-31.png)
  
# View 取得したデータの表示
データの読み取りをする必要がある
よく使うクエリを呼び出してみる

Queryはデータのタイプ

Queryの統計値はカテゴリに分けられている.

WatchとiPhoneで同時間帯に取得した結果は,それぞれを保管する形で利用する事ができる
![](2022-03-24-01-18-05.png)



# HKQueryの種類は豊富にある
![](2022-03-24-09-48-52.png)

# HelathKitのデータをシンクする
* HKAncoredObjectQuery
Updateを検知することができるようになる

MetadataはVersioningとIDを判別して重複を起こさないようにするために必要
