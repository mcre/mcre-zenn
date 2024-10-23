---
title: "Vue3 + AWSでフルスタック開発 - コード全公開"
emoji: "🧑‍💻"
type: "tech"
topics: ["vue3", "AWS", "個人開発", "ogp", "cdk"]
published: true
---

# はじめに

Vue3でWebアプリを開発しました。
SPAながら、OGPやSSGに対応し、SEO的にも悪くないはずです。バックエンドはAWS CDKで構築しています。

ソースコードを全部公開しているので、すこしでも読者のかたの参考になるところがあれば嬉しいです。

https://github.com/mcre/mcre-tools

**用語**

- SPA (Single Page Application): VueのデフォルトだけどSEO的にちょっと使いづらいやつ。
- OGP (Open Graph Protocol): XなどでURLを貼ると画像が表示されるやつ
- SSG (Static Site Generation): ビルド時にHTMLが生成されるやつ。素のVueだとコンテンツを埋め込んでくれないので、これをやるとSEO的に良いはず。
- AWS CDK: AWSインフラをコードで管理するやつ

# どんなWebアプリか

本質的なところでないのでさらっと流しますが、いまのところこんなサイトです。

https://tools.mcre.info/ja

![](/images/mcre-tools-summary/website01.webp)
*トップページ*

![](/images/mcre-tools-summary/website02.webp)
*熟語パズル*

今あるツールは「熟語パズル」のみです。よくある「和同開珎」パズルを解いてくれるアプリです。今後ツールは追加していく予定です。

https://tools.mcre.info/ja/jukugo?t=%E5%92%8C&r=%E9%96%8B&b=%E7%8F%8E&l=%E9%8A%85&ab=0&a=%E7%94%B0

OGPもこんな感じで自動生成されます。

# 技術スタック

## フロントエンド

主な使用パッケージなど

- [Vue3](https://ja.vuejs.org/)
  - フロントエンドフレームワーク
- [Vuetify](https://vuetifyjs.com/ja/)
  - マテリアルデザインに対応したUIライブラリ
- [Vite SSG](https://github.com/antfu-collective/vite-ssg)
  - Vue3をSSGに対応させるライブラリ
- [aspida](https://github.com/aspida/aspida) / [openapi2aspida](https://github.com/aspida/openapi2aspida)
  - OpenAPI定義からAPI呼び出しのためのTypeScriptの型を作ってくれるライブラリ

## バックエンド

![](/images/mcre-tools-summary/architecture.webp)
*[cdk-dia](https://github.com/pistazie/cdk-dia)で自動生成したアーキテクチャ図*

AWSを使用しています。自動生成したものをそのまま貼っているのであまり見やすくないですが図のような感じです。

- CloudFront
  - コンテンツ配信サービス(CDN)
- Lambda@Edge
  - CloudFrontに来たリクエストごとに特定の処理を行う。たとえばOGP画像取得の場合に`<head>`タグを動的に生成するなど。
- API Gateway
  - APIリクエストをLambdaに渡す
- Lambda
  - 内部で使用するAPIの作成(DBにアクセスして熟語リストを返すなど)
  - OGP画像を生成し返却するAPIの作成
- Route53 / ACM
  - ドメインと証明書の取得・管理
- DynamoDB
  - データベース
- S3
  - ビルドしたVueプロジェクトのデプロイ先
  - 動的に生成したOGP画像置き場

図の左が東京リージョンで右はバージニア北部リージョン(us-east-1)です。
Lambda@EdgeとCloudFrontの証明書をバージニア北部リージョンに置かなければいけないためリージョンを分けましたが、全部をバージニア北部においたほうが楽だったかもしれません。CDKでリージョンをまたいだリソース名を相互に参照するのは出来なくはないのですが結構手間でした。

## CI/CD

- AWS CDK
  - 上記バックエンドのほぼすべてをコードで管理し、デプロイできます。
  - TypeScript/Pythonなど使用言語を選べますが、ここではPythonを使っています。一番得意なので。
- GitHub Actions
  - 単体テスト、e2eテスト、CDKのデプロイ、Lambda関数のアップデート、Vue3プロジェクトのビルド・デプロイを実行します。
  - mainブランチが更新されるごとに自動実行されます。

# 雑感

開発時に迷ったこととして大きいところでいえば、なぜ[Nuxt](https://nuxt.com/)を使わなかったかということです。
VueでSSG/SSRを実現するのであればまず選択肢に上がるのがNuxtです。

最終的にNuxtを使用しないと決めた理由としてはこのような感じです。

1. 追加したい機能はSSG/SSRだけなのに学習コストや不要な追加ライブラリなどが大きすぎるのでは
2. サーバレス志向が強い。サーバレス構成であれば圧倒的に安い
   - [Vercel](https://vercel.com/)とかを使えばいいかもだけど、無料プランだと広告載せられないし・・・
3. [Vite SSG](https://github.com/antfu-collective/vite-ssg)がちょうどニーズに合った

私がAWSやVueには慣れているがNuxtを使ったことが無いというのも大きいかもしれません。触ってみたら考えが変わるかもしれません。

なお、AWSコストは実際にかなり安いです。**約1ヵ月あたりなんと0.05ドル！(8円くらい)**

![](/images/mcre-tools-summary/aws-cost.webp)
*10月1日〜23日*

検討時に参考にしたサイトなど

https://ja.vuejs.org/guide/scaling-up/ssr
https://www.reddit.com/r/vuejs/comments/1e4gwwf/vue3_vs_nuxt3/?rdt=45028


# 今後の記事

このWebアプリをつくるにあたって試行錯誤したことがいろいろあってまだまだ書き足りないので、今後も別の記事としてしていろいろ書いていきたいと思います。

たとえばこんなネタがあります。

- 熟語パズルの作り方やライセンスについて
- CDKやGithub Actionsについて
- DynamoDBの使い方について
- Vite SSGの所感
- i18n対応について
- OGPの実現について
- APIやaspidaについて
- テスト(Vitest, Playwright)

いずれにしてもソースコードにはすでに試行錯誤の結果が載っているので、興味あるかたは見てみてください。

https://github.com/mcre/mcre-tools