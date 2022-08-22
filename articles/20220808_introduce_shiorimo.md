---
title: "Remixで個人開発をしたので技術スタックを紹介"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Remix", "個人開発"]
published: true
---

## 概要

初めまして。wancupです。元々フルスタックエンジニア的に色々やっていましたが、ここ一年くらいはほぼフロントエンドしか触っていません。最近会社を辞めたので、就職活動のポートフォリオ代わりに記事を書くことにしました。

今回は個人開発をしている「しおりモ！」というサービスについて、使っている技術とその所感を書いていきます。各技術の詳しい説明や導入の仕方などは、他に優れた記事が幾つもあるので、そちらに譲ります。本記事ではあくまで各技術をサービス開発のレベルで（とはいえ個人開発レベルですが）使ってみてどうだったかという点に主眼を置きます。構成の参考にでもしていただけると幸いです。

### 今回言及する技術

| 名称          | 概要                                         | リンク                                        |
|-------------|--------------------------------------------|--------------------------------------------|
| Remix       | SSRフレームワーク                                 | https://github.com/remix-run/remix         |
| Fly.io      | PssS                                       | https://fly.io                             |
| Firebase    | BaaS（Firebase AuthenticationとFirestoreを使用） | https://firebase.google.com/               |
| Mantine     | コンポーネントライブラリ                               | https://github.com/mantinedev/mantine      |
| CodeceptJS  | E2Eテストのフレームワーク                             | https://github.com/codeceptjs/CodeceptJS   |
| Meilisearch | 全文検索エンジン                                   | https://github.com/meilisearch/meilisearch |
| Upstash     | サーバーレスデータベース（セッション管理に使用）                   | https://upstash.com                        |

## しおりモ！

まずは開発しているサービスの紹介をします。

https://shiorimo.fly.dev

「しおりモ！」は本に関するメモをいい感じ（当社比）に記録するためのサービスです。元々読書中に気に入ったセリフや後から読み直したい箇所を手帳や他のサービスにメモしていました。しかし、あとで見返すときにどの本のどのページか分かりづらい/検索がしづらかったので、自分が使いやすいサービスを開発することにしました。一応本についてのメモとは謳っているものの、通常のメモ帳サービスとしても使えるはずです。

まだ初期機能（メモの追加、編集、削除と検索）の実装が終わった段階です。これからも細かい機能追加を色々したいとは思っています。ただ、とりあえず自分で使う分には最低限のところは終わったので、あとはのんびりやっていくつもりです。SEO対策とかOGPの設定はしていません。

よかったら使ってみてください。

### メモの作成

![](/images/20220808_introduce_shiorimo/shiorimo_create_memo.gif)

### メモの検索

![](/images/20220808_introduce_shiorimo/shiorimo_search_memo.gif)

### メモの編集

![](/images/20220808_introduce_shiorimo/shiorimo_edit_memo.gif)

## Remix

せっかくの個人開発なので、これまで触ったことがなかった[Remix](https://github.com/remix-run/remix)を採用しました。

まず感じたのが、SSR特化というのは分かりやすいということでした。Next.jsのようにページごとにレンダリング手法を選べるのはメリットではあるのですが、適しているレンダリング手法が何か、データはどのタイミングにどこで取得するべきなのかを考える必要が（あまり）ないのは楽でした。基本的にデータはサーバー側で処理をして、その結果をクライアント側に投げるという従来の（?）Webっぽい形式なので、人によっては結構馴染みがあるのではと感じました。

### 最初見落としていた点

Remixはリンクのプリフェッチがデフォルトで無効なので、プリフェッチしたい場合は自分で指定する必要があります。
https://remix.run/docs/en/v1/api/remix#link

### 便利なライブラリ

RemixのFormでpostしたデータはActionのrequestから取得できますが、その際にバリデーションと型変換をしたいことがよくあります。[Remix Params Helper](https://github.com/kiliman/remix-params-helper)（とzod）を使うとそこらへんをかなり簡単に記述できました。クライアントサイドでのバリデーションとかも含めてもっとリッチなライブラリが欲しい場合は[Remix Validated Form](https://github.com/airjp73/remix-validated-form)も良さそうな感じです。

### Markdown

Remixで地味に便利だと思ったのが、Markdown(.mdや.mdx)をそのままレンダリングできることです。

https://remix.run/docs/en/v1/guides/mdx

Markdownで書いたファイルがそのまま通常のルーティングに従ってレンダリングされるので、単純なテキストベースのページだとめちゃくちゃ管理が楽でした。今回で言えば[利用規約](https://shiorimo.fly.dev/docs/terms)や[プライバシーポリシー](https://shiorimo.fly.dev/docs/privacy)をMarkdownで書いています。

## Fly.io

ユーザに（物理的に）近い位置でサービスを提供するためのフルマネージドなPaaSです。Node.jsの文脈で語られることが多い気がしますが、Dockerイメージにできるなら（多分）なんでも動かせます。個人的な印象としては実態はAWSのECSとかに近い感じがしました。アプリの設定をして、動かすためのDockerfileを書いて、flyctl(Fly.ioのCLI)でデプロイコマンドを叩くだけで動かせます。

最近は当たり前のようになってきていますが、GitHub Actionsとの連携もめちゃくちゃ簡単で、CDの構築もすぐにできました。

https://fly.io/docs/app-guides/continuous-deployment-with-github-actions/#speed-run-your-way-to-continuous-deployment

ただ、仕事で使うとなると現在のところロールバックの機能が公式では提供されていないのが多少ネックになるかなと思っています。個人的にはロールバックをすることは滅多にないですし、その場合でも戻したい内容で再ビルド&デプロイを走らせれば良いだけだとは思っていますが。

https://community.fly.io/t/rollback/2082/7

かなり更新が頻繁なようで、ダッシュボードとかCLIがどんどん改善されています。触り始めたときにはCLIでしか確認できなかったような内容が、今ではダッシュボードで確認できるようになっていたりしました。変化が感じられるのは使っていて楽しかったです。

使うことは少ないですが、CLIを使ってsshでコンソール入ることも簡単にできます。CLIに色々なコマンドが用意されているのがありがたいです。

## Firebase

Firebaseで一番嬉しかったのが、ローカルの開発中に使えるエミュレータが便利だったことです。CIでテストを動かすときにもエミュレータを使えたので、環境を整えるのが楽でした。

### Firebase Authentication

認証まわりで使用しています。Google製のおかげでGoogleアカウントとの連携が楽でした。エミュレータ上でGoogleアカウントでのログインをエミュレートできるのはかなり嬉しいです。

ただ、ログイン処理の結果を取得するgetRedirectResultの処理が遅いのは難点だと感じました。数年前にそういった[issue](https://github.com/firebase/firebase-js-sdk/issues/1499)が上がっていたようですが、そういうものらしいです、、。

地味に嬉しかったのが、エミュレータで新規ユーザを作成する際に、ダミーデータの自動作成機能があることです。ローカルでの開発中に適当なアカウントを作りたいときに重宝しています。

![](/images/20220808_introduce_shiorimo/firebase_create_dummy_user.gif)

### Firestore

言わずと知れたNoSQL DBです。今までNoSQLは数年前にDynamoDBを触っただけでしたが、「ドキュメント」と「コレクション」という概念さえ理解すればすんなりと使えました。コレクションに関してはサブコレクションというかたちでネストできるのが便利でした。

## Mantine

最終的にMantineに落ち着いたのですが、それまでに[tailwindcss](https://github.com/tailwindlabs/tailwindcss) + [daisyUI](https://github.com/saadeghi/daisyui) + [Headless UI](https://github.com/tailwindlabs/headlessui)や[ChakraUI](https://github.com/chakra-ui/chakra-ui)を試したりもしました。Mantineはデザインが好みというのもありますが、かなり使い勝手が良くてカスタマイズ性も高かったのが最終的に落ち着いた理由です。用意されているコンポーネント数も多く、痒いところに手が届く感じがあります。例えばオートコンプリートやStepperなども用意されているのが嬉しいです。

コンポーネント群だけでなく、[Theme functions](https://mantine.dev/theming/functions/)で用意されている関数群も便利です。例えばレスポンシブデザインのために画面サイズでスタイルを変えたいときにはsmallerThanやlargerThanを使ったり、色のアルファ値だけを変えたいときにrgbaを使ったりしています。

Mantineのv5とRemixを組み合わせると警告が出るという[問題](https://github.com/mantinedev/mantine/issues/1895)があるので、~~まだMantineはバージョン4を使っています~~。回避できないかといくつか試してみたのですが、今のところ解決策が見つかっていません、、。情報提供お待ちしています。
とりあえず警告だけで動作には問題なさそう（[FYI](https://github.com/mantinedev/mantine/issues/1895#issuecomment-1219426360)）なので、v5にアップデートしてみました。（2022/8/22追記）

## CodeceptJS

E2Eテストのフレームワークです。実際には他のテストツールのラッパーで、テスト用の高レベルなAPIを提供してくれています。今回は内部ではPlaywrightを使いました。

書き方は少し独特なのですが、個人的にはユーザの振る舞いをそのまま記述している感じがして好みでした。これまで仕事ではCypressをよく使っていたのですが、セレクターで要素を選択するのが結構面倒に感じていました。CodeceptJSではそこらへんが抽象化されていて、テストの記述が楽です。

例えばこのようなテストを書くとします。

1. `/XXX`というパスに遷移
2. 「入力」というテキストを持つlabelに紐づいたinput要素に「テスト」と入力する
3. 「完了」というテキストを持つsubmitボタンをクリックする

CodeceptJSで書くと以下のようになります。

```javascript
I.amOnPage("/xxx");
I.fillField("入力", "テスト");
I.click("完了");
```

例えば2行目の`I.fillField("入力", "テスト")`でやっているようなことをCypressでやろうとすると、「入力」というテキストを持つラベルを探し出して、その子要素（もしくはforで指定している要素）を探し出す、あるいはclassやidなどから要素を見つけ出すことになると思います。CodeceptJSだとこの流れをかなり簡潔に記述できて、E2Eテストを書く労力が大分低減されました。

また、便利だったのが[pause](https://codecept.io/basics/#pause)というAPIです。`pause()`と記述している箇所でテストが止まり、その続きをターミナル上で実際に書いて動作を確認しながら動かすことができます。そして、試し終えたら自分が書いていたコマンドがテキストファイルに出力されるので、その内容をコピペすればテストが完成します。

書き方も独特なので好みは別れると思いますが、個人開発レベルでパパッとテストを書きたいときには重宝するフレームワークだと思いました。

## Meilisearch

Rust製の全文検索エンジンです。Cloudサービスも提供されているので、特に自分で設定等せずにも使えそうですが、今回は無料で使いたかったのでOSS版を自分でホスティングしています。結構しっかり開発されているようで、ちょくちょくバージョンが更新されます。ロードマップとかもちゃんと定義されています。ただし、まだバージョン1に達していないので、バージョンアップごとにわりと大きめな修正が入っていたりします。

DockerHubに公開されているイメージをそのままFly.io上にデプロイしています。また、Fly.ioの別のAppからmeilisearchのコンテナに[Private Networking](https://fly.io/docs/reference/private-networking/)で接続しています。Private NetworkingはIPv6で接続するので、Meilisearchのコンテナ側でIPv6もリッスンするようMEILI_HTTP_ADDRを少しいじっています。これはFly.ioの話ですが、ここら辺の構成を簡単にできたのがかなり嬉しかったです。

v0.27.0で日本語対応が入ったものの、現在（v0.28.1）漢字の検索まわりで[問題](https://github.com/meilisearch/meilisearch/issues/2403)があるようです。

## Upstash

セッション管理のためのRedis用に使っています。本当はfly.io上でRedisを使いたかったのですが、すでに無料プランの枠を使い切っているため、別サービスを使うことにしました。

ローカルでの開発をどうしようかと考えて、Upstashに接続するのはデプロイ環境のみ、ローカルやCI上でのテストでは自分で立ち上げている通常のRedisコンテナに繋ぐことにしました。ただ、この構成だとRedisクライアントを使うのため、RESTで接続できないのがデメリットです。ベストな構成が知りたいです。

## おまけ（ロゴ作成）

アプリロゴの作成には[Affinity Designer](https://affinity.serif.com/ja-jp/designer/)のiPad版を使用しました。大分前に安くなっていたタイミングで買っていたのを初めてちゃんと使いました。ベクターデザイン用のツールを触るのも初めてだったので結構戸惑うことも多かったのですが、公式で動画を用意してくれていたり、YouTubeで解説してくれている方もいるのでなんとかロゴ制作はできました（レタリング難しい、、）。

ともかく、こういったツールは滅多に触らないので、買い切りなのはありがたいです。そして、値段の割にめちゃくちゃ機能が豊富だと感じました。個人開発でちょっとだけSVGとか作ることもあるという方にかなりオススメです。
