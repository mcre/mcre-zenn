---
title: "Webアプリのための都道府県地図データの加工"
emoji: "🗾"
type: "tech"
topics: ["地図", "GIS"]
published: true
---

## はじめに

「けんみんラボ」というWebアプリをつくったのですが、そのために実施した地図の加工についてまとめます。

https://note.com/m_cre/n/n5b2cceae4b91

## 加工用ツールのダウンロード

[mapshaper](https://github.com/mbloch/mapshaper)を利用します。
npmの場合は以下のコマンドでインストールできます。

```sh
npm install -g mapshaper
```

## 地図データの入手

[国土数値情報ダウンロードサイト](https://nlftp.mlit.go.jp/ksj/)から、
「行政区域データ」の「全国」データ(N03-20250101_GML.zip)をダウンロードします。

このデータのライセンスは [CC_BY_4.0](https://creativecommons.org/licenses/by/4.0/deed.ja) のため、使用時には適切な出典を記載する必要があります。

:::message
この記事で使用している地図データは [国土交通省 国土数値情報ダウンロードサイト](https://nlftp.mlit.go.jp/ksj/)の行政区域データ([CC_BY_4.0](https://creativecommons.org/licenses/by/4.0/deed.ja))を筆者が加工して作成したものです。
:::

## 地図画像のシンプル化

ダウンロードしたzipの中に入っている `N03-20250101_prefecture.geojson` というファイルを使います。

今回作成するファイルもgeojsonファイルなので、じつはこのファイルを直接利用することもできます。
ただしファイルサイズが376.4MBなので、このままではとてもつかえません。

試しに下のコマンドでSVG画像に変換して確認してみます。

``` sh
mapshaper N03-20250101_prefecture.geojson \
  -proj epsg:3857 \
  -style fill=#fff stroke=#000 \
  -o format=svg map-original.svg
```

![map-original.svg](/images/jp-prefectures-map/map-original.png)
*map-original.svg*

小さな島がたくさん表示されますし、海岸線も複雑です。
今回のようなWebアプリでは都道府県がわかれば厳密さはそこまで要らないので、よりシンプルにしてファイルサイズを小さくします。

シンプルにするには下のようなコマンドを使います。

``` sh
mapshaper N03-20250101_prefecture.geojson \
  -simplify 0.1% \
  -clean \
  -o map-simple-0.1percent.geojson
```

`-simplify` の値を下げていくと地図がシンプルになり、geojsonファイルのサイズが小さくなります。
`-clean` をつけると、シンプル化の結果描ける線がなくなってしまった島などの要素をまるごと削除してくれます。

ファイルサイズの関係を表にしてみました。

| simplify | geojsonファイルサイズ |
| -- | -- |
| 1%       | 3.9MB |
| 0.1%     | 455KB |
| 0.05%    | 240KB |
| 0.04%    | 195KB |
| 0.03%    | 150KB |
| 0.02%    | 105KB |
| 0.01%    |  61KB |
| 0.001%   |  20KB |
| 0.0001%  |  16KB |
| 0.00001% |  16KB |

![1%](/images/jp-prefectures-map/map-simple-1percent.png =460x)
*1% (3.9MB)*

![0.1%](/images/jp-prefectures-map/map-simple-0.1percent.png =460x)
*0.1% (455KB)*

![0.05%](/images/jp-prefectures-map/map-simple-0.05percent.png =460x)
*0.05% (240KB)*

![0.04%](/images/jp-prefectures-map/map-simple-0.04percent.png =460x)
*0.04% (195KB)*

![0.03%](/images/jp-prefectures-map/map-simple-0.03percent.png =460x)
*0.03% (150KB)*

![0.02%](/images/jp-prefectures-map/map-simple-0.02percent.png =460x)
*0.02% (105KB)*

![0.01%](/images/jp-prefectures-map/map-simple-0.01percent.png =460x)
*0.01% (61KB)*

![0.001%](/images/jp-prefectures-map/map-simple-0.001percent.png =460x)
*0.001% (20KB)*

![0.0001%](/images/jp-prefectures-map/map-simple-0.0001percent.png =460x)
*0.0001% (16KB)*

![0.00001%](/images/jp-prefectures-map/map-simple-0.00001percent.png =460x)
*0.00001% (16KB)*

この中からファイルサイズとシンプルさのバランスを見てどれを採用するか決めることになります。今回は0.02を選びました。
また、最終的に `-each` と `-filter-fields` でファイル内に記載されたフィールドを整理し、さらに97KBまで削減できました。

``` sh
mapshaper N03-20250101_prefecture.geojson \
  -each 'code=N03_007.slice(0,2)' \
  -filter-fields code \
  -simplify 0.02% \
  -clean \
  -o map-simple.geojson
```

![0.02%](/images/jp-prefectures-map/map-simple-0.02percent.png)
*0.02% + フィールド整理 (97KB) を採用*

### (余談)投影方法

画像変換時に指定した `-proj` は画像へ変換する際の投影方法です。メルカトル図法とかいうあれです。
EPSG:3857 はGoogle Mapなどで採用されている [Webメルカトル図法](https://en.wikipedia.org/wiki/Web_Mercator_projection) のことのようです。
この変換をしないと見慣れない縦に潰れた日本地図が出てきてしまいます。

![つぶれた日本地図](/images/jp-prefectures-map/map-original-non-proj.png)
*つぶれた日本地図*

## 沖縄と奄美群島を右下へ

このままでも十分なのですが、沖縄と奄美群島を地図の余白に移動させることによって、スマートフォンで横幅いっぱいに表示した場合に、より拡大して表示できるようになります。

試行錯誤の結果、以下のようなコマンドで実現できました。

``` sh
mapshaper map-simple.geojson -clip bbox=-180,-90,180,30 -o south.geojson
mapshaper map-simple.geojson -erase south.geojson -o north.geojson
mapshaper south.geojson -proj epsg:3857 -affine shift=2060000,760000 -proj wgs84 -o south_moved.geojson
mapshaper -i north.geojson name=north -i south_moved.geojson name=south -merge-layers target=north,south -o merged.geojson
jq -c . merged.geojson > jp-prefectures.geojson  # (おまけ)改行やスペースを消す
```

地図を北緯30度で南北に分け、東に2,060km・北に760km移動したうえで再度重ねています。

![最終結果](/images/jp-prefectures-map/map-jp-prefectures.png)
*最終結果 (98KB)*

沖縄奄美を移動させたことがわかるように区切り線を追加する必要がありますが、今回は後続の色塗りの処理で実施しています。

## TopoJSONについて

さらにファイルを軽くしたい場合はTopoJSONという形式があるようです。

``` sh
mapshaper jp-prefectures.geojson -o format=topojson jp-prefectures.topojson
```

29KBまで削減できました。

フロント側で地図を描画する場合はTopoJSONを使うことがあるようです。
今回はサーバ側のPythonで描画しているため、geojsonを採用しました。
