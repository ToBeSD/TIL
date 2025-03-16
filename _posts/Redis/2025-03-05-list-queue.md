## はじめに

バックエンドエンジニア歴2年、Redisに初めて触れたので記念するため記事として記録する！

## どんなサービスに導入した？

ディスプレイ広告のImpression集計サービスに導入した。

## Impression集計フローを変えた。
### 理由
サーバのディスク使用率が１００％になったことがあり、Impressionの書き込みができず、損失に繋がったことがある。
ディスク使用率が１００％に絶対ならないような対策と共に、リアルタイムでDBに保存するよう変更した。既存と同じくcsvファイルとしてもバックアップするので、どっちか一箇所が壊れてもImpressionを担保できる。

**既存の仕様**
1. ImpressionサーバでImpressionをcsvファイルにテキスト形式で保存
2. Batchサーバでcsvファイルを読みDBに保存

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3256444/43ae65f4-b633-434f-8dae-519b7a91d5e5.png)


**新しい仕様**
1. ImpressionサーバでDBに直保存する
2. バックアップとしてcsvファイルも取っておく

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3256444/f577b706-3a61-4aa4-9a3e-7ead868705a4.png)

## Redisで実現できたこと

1. リアルタイムでImpressionを集計（Redisのシングルスレッドの特徴、ListをQueueのように使うことでデータ損失、マルチスレッドの同時生問題をふs）
2. ImressionデータをRedisに臨時保存する→BatchInsertすることでRDSとのI/Oコスト削減
