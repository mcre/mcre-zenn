---
title: "家庭の書類スキャン運用遍歴"
emoji: "🖨️"
type: "idea"
topics: [gcp, google cloud vision api]
published: false
---

# はじめに

10年ほどまえから、学校からの書類、郵便物、公共料金の明細など、家庭の紙ものはスキャンして捨てる運用を続けています。

徐々に改善を重ねて現在第4世代になっており、その遍歴とそれぞれどうなったかについて紹介していこうと思います。

# 遍歴

# 1. Scan Snap + フォルダごとに保存

* 開始時期
  * 2012年7月〜
* スキャナー
  * ScanSnap S1500
* PDFファイルの保存方法
  * 内容ごとにフォルダ分けしてDropboxに保存

念願のScanSnapを購入したので、・・・

# 2. Scan Snap + フォルダ保存をあきらめる

* 開始時期
  * 2015年10月〜
* スキャナー
  * ScanSnap S1500
* PDFファイルの保存方法
  * フォルダ分けせずに、NAS(Synology DiskStation DS215j)に保存する
  * DiskStationの機能でインデックスするすることで、Universal Searchでテキスト検索できる



# 3. Brother複合機

* 開始時期
  * 2020年4月〜
* スキャナー
  * Brother インクジェット複合機 DCP-J982N
* PDFファイルの保存方法
  * プリンタの機能で連携したGoogle Driveに保存する
  * NAS(Synology DiskStation DS215j)に手動で移動する
  * DiskStationの機能でインデックスするすることで、Universal Searchでテキスト検索できる

## 4. (現在)iPhoneでスキャンし複合機

* 開始時期
  * 2024年3月〜
* スキャナー
  * iPhoneの「ファイル」アプリの「書類をスキャン」機能
* PDFファイルの保存方法
  * iPhoneの「ファイル」を NAS(Synology DiskStation DS215j)に直接接続し、「書類をスキャン」機能でPDF保存する。
  * この方法で作成したPDFはサーチャブルPDFにならないので、Google Cloud Vision APIでテキスト化したものを同じフォルダにテキスト形式で保存
  * DiskStationの機能でインデックスするすることで、Universal Searchでテキスト検索できる

妻も作業できる

# 現在の方法の手順

