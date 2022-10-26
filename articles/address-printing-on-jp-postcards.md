---
title: "年賀状の宛名印刷難民の救済"
emoji: "📮"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [docker, tex, python]
published: true
---

# はじめに

毎年年末に悩まされる年賀状、近年の傾向のとおり私も基本的には止めてしまったのですが、親戚だけには出すことにしています。

Windowsであれば手頃な宛名印刷ソフトがいくつかあったと思うのですが、Macに乗り換えてからは定番のものがみつからず、困っていたものです。

それであれば、自分でつくろうと考えました。

Macだけでなく、DockerさえインストールできればWindowsでもLinuxでもラズパイでもうごくはずです。たぶん。

# つかいかた

## Dockerのインストール

Dockerをインストールしたことがない場合は、[公式サイト](https://docs.docker.com/engine/install/#desktop)などを参考にまずインストールしてください。

## ツールのインストール例

```shell
git clone https://github.com/mcre/address-printing-on-jp-postcards.git
cd address-printing-on-jp-postcards
docker build -t texlive-python .
```

## サンプルデータでの実行

```shell
docker run -it --rm -v $(pwd):/workdir --rm texlive-python sh -c "python3 main.py config/sample01.json"
docker run -it --rm -v $(pwd):/workdir --rm texlive-python sh -c "python3 main.py config/sample02.json"
```

`./output/sample01.pdf`, `./output/sample02.pdf`に 宛名PDFが出力されます。

![宛名PDFサンプル](/images/address-printing-on-jp-postcards/pdf-sample.png)
*./output/sample01.pdf*

## 入力データの説明

まずはサンプルデータの入力を確認してみましょう。

https://github.com/mcre/address-printing-on-jp-postcards/blob/master/input/sample01.csv

1行目がヘッダーです。これまで私が宛名印刷ソフトを使用してきた経験上、csvフォーマットはソフトごとにあらかじめ決まっており、それにあわせたcsvに整形するのが億劫なことのひとつだったため、このツールではcsvフォーマットごと設定できるようにしました。

https://github.com/mcre/address-printing-on-jp-postcards/blob/master/config/sample01.json

サンプルの場合、`address_1`にcsvの`住所1`が対応します。`address_1`は下記のTeXテンプレートに挿入されます。

https://github.com/mcre/address-printing-on-jp-postcards/blob/master/templates/page/default.tex

また、jsonファイル内の`processing`を調整すると、文字列の置換や「喪中」チェックが付いている行を出力しないなど、いろいろな操作を行うことができます。詳しくは[README.md](https://github.com/mcre/address-printing-on-jp-postcards/blob/master/README.md)をご確認ください。

## 使用方法

次の手順で、自分の家計簿csvファイルをPDFに変換し、印刷することができます。

1. input/ に住所録のcsvファイルを配置します。
2. サンプルを参考にconfigファイルを作成してください。
   * `input` にはcsvファイルのパスを記入します
3. 下記のコードで実行することで、`output`に指定したパスにPDFファイルが出力されます。
4. PDFビューアーを使って郵便はがきにプリントアウトしてください。
   * 位置がずれる場合は[テンプレートファイル](https://github.com/mcre/address-printing-on-jp-postcards/blob/master/templates/page/default.tex)の数値を調整してみてください。

```
docker run -it --rm -v $(pwd):/workdir --rm texlive-python sh -c "python3 main.py config/{configファイル名}.json"
```

# ソースコードや構成の解説

https://github.com/mcre/address-printing-on-jp-postcards

Docker イメージは日本語対応のTex Live入りのものにPythonをインストールして使用しています。

https://github.com/mcre/address-printing-on-jp-postcards/blob/master/dockerfile

ソースコードはこれだけです。テンプレートにCSVを差し込んでTeXを実行しているだけです。

https://github.com/mcre/address-printing-on-jp-postcards/blob/master/main.py

# プルリクエスト大歓迎

もしこのツールをお使いいただいた場合は、ぜひお作りいただいたconfigファイルとサンプルのcsvをご提供ください！

あとは、宛名面への差出人名の追加機能や、`processing`の機能の追加などなどお待ちしてます！
