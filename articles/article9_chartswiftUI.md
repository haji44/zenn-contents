---
title: "MotionをつけたグラフをSwiftUIで実装する"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Swift","SwiftUI"]
published: false
---
# SwiftUIでChartに動きをつける
個人開発のアプリでChartを実装したかったのでついでに,それっぽい動きをつけてみた

# 実装のイメージ
こんな感じで円グラフと,バーグラフをそれぞれ作ります.
Fitnessアプリに近付けるために,ページネーションを追加します.

# 表示するデータのModelを定義する
まずは,データを表示する際の具体的なデータを作成します.
今回は, RingとBarでそれぞれMockも含めてデータを作成するようにします.






