---
title: "Reactのチャートライブラリのアクセシビリティ調査"
emoji: "♿️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["React"]
published: true
---

## まえがき

Reactでチャート（グラフ描画）ライブラリの選定をするにあたって、アクセシビリティ（以下a11y）の対応について調査したのでまとめます。
対象とするのは以下の記事で紹介されていた11個のライブラリです。
https://zenn.dev/aldagram_tech/articles/346224ada3faa1

とりあえずザッとふるいにかけたいので、今回調べるのは以下の二点にとどめます。

- キーボードだけでマウスと同等の操作ができるか(WCAG2.1 SC2.1.1)
- スクリーンリーダーのことが考慮されているか(WCAG2.1 SC1.1.1)

色の使いかたなどはある程度実装側でカバーする予定です。

## Recharts

https://github.com/recharts/recharts

v2時点ではa11yの考慮はほとんどされていなさそう（？）
しかし、v3（記事執筆時点ではalpha版）でaccessibilityLayerというのが導入されるようです。これによってキーボード操作とスクリーンリーダー対応が入りそうです。

:::message
2025年01月15日追記
ドキュメントのAPIには記載がありませんが、v2の時点で既にaccessibilityLayerが使えそうです！
https://github.com/recharts/recharts/blob/v2.15.0/src/chart/generateCategoricalChart.tsx#L866
:::

ドキュメントサイトのStorybookの`/API/Accessibility`に詳細が記載されています。
https://recharts.org/en-US/storybook

ドキュメントではtitle要素でチャートの説明を記載していますが、チャートにマウスオーバーしたときにツールチップと表示が被ることがありました。

![ツールチップにタイトルの表示が被っている例](/images/20250111_search_react_chart_libs_a11y/title_over_tooltip.png)

少し気になってしまうので、自分が実際に使うときはtitleではなく`aria-`属性あたりを使うかもしれません。
ともかく、v3でかなりの改善がありそうなので正式リリースが楽しみです！

## Nivo

https://github.com/plouc/nivo

一部（Bar Chart）にはキーボード操作、スクリーンリーダー読み上げの対応が入っています。しかし、それ以外のチャートはまだ未対応な模様。
詳細は以下のIssueにまとまっています。
https://github.com/plouc/nivo/issues/126

Nivoはコンポーネント数も多いしデザインも好きなのでおしい

## Victory

https://github.com/FormidableLabs/victory

読み上げるラベルとか、どの要素をフォーカス可能にするかをちゃんと自分で指定していく感じ。
特定の要素をグルーピングしてキーボードフォーカスの順番を管理できるので、機能的には問題なさそう。
ドキュメント↓に実装例があります。
https://commerce.nearform.com/open-source/victory/docs/guides/accessibility

## Ant Design Charts

https://github.com/ant-design/ant-design-charts

v2はドキュメントサイトがほとんど中国語で読めませんでした、、。
v1は軽くみた感じだと特に考慮なさそう？

## billboard.js

https://github.com/naver/billboard.js

ドキュメントにa11yについての記載は見当たらず。
描画のhookにひっかけて自分で頑張って各要素を修正しろ、という方針に見える。
https://github.com/naver/billboard.js/issues/2245#issuecomment-896557994

## react-chartjs-2(Chart.js)

https://github.com/reactchartjs/react-chartjs-2

Chart.jsのラッパーなのでChart.jsのほうを調べます。

https://github.com/chartjs/Chart.js

canvasにチャートを書き込む都合上、自分で`aria-label`なりフォールバックコンテンツに適切な説明を書けとのこと。
https://www.chartjs.org/docs/latest/general/accessibility.html

方針は分からなくはないものの、データ量増えると大変そう。
上記ドキュメントで言及のある以下のサイトで紹介されているように、実際にはテーブルなどを併記してそちらで説明をする方が良いかも。
https://pauljadam.com/demos/canvas.html

なお、キーボード操作についての説明は見つけられませんでした。

## MUI X Charts

https://github.com/mui/mui-x

特にa11yについての全般的な記述は見つからず。
サンプルもキーボードフォーカス効かなかったので考慮されていない？
Issueはありました。
https://github.com/mui/mui-x/issues/13113

## unovis

https://github.com/f5/unovis

v1.2で`aria-`系がサポートされたとのこと。
https://unovis.dev/releases/1.2/#--accessibility-supporting-aria-tags
キーボード操作はまだ未対応っぽい。
なお、ロードマップにはちゃんとキーボード操作含めたa11yの記載があるので、対応は予定されていそう。
https://github.com/f5/unovis/discussions/251

## react-plotly.js(plotly.js)

https://github.com/plotly/react-plotly.js?tab=readme-ov-file

plotly.jsのラッパーなのでplotly.jsのほうを調べます。

https://github.com/plotly/plotly.js

キーボード操作未対応。2016年からIssueがあり趣を感じる。
https://github.com/plotly/plotly.js/issues/562

## AG Charts

https://github.com/ag-grid/ag-charts

今回調べた中で一番しっかりa11yが考慮されていると感じた。
キーボード操作、スクリーンリーダーでの読み上げにちゃんと対応している。

https://ag-grid.com/charts/react/accessibility/

ギャラリーに載っているチャートもキーボード操作できますし、さすがに有償版も提供されているだけのことはある。
無償版でもよく使うチャートは用意されているので、もっと流行っても良いのではという印象。

## Charts.css

https://github.com/ChartsCSS/charts.css

HTMLとCSSで構築するので実装者次第というように思う。

## おわりに

よりアクセシブルなWebを目指して
