---
title: モバイルアプリでアクセシビリティ対応をする前に知るべきこと
tags:
  - Mobile
  - MobileApp
  - アクセシビリティ
  - a11y
private: false
updated_at: '2022-03-09T17:53:27+09:00'
id: f566dbcfec9fdb69e993
organization_url_name: null
---
## はじめに

アクセシビリティについて、ただ闇雲に対応するだけでは向上しません。
アクセシビリティに関する機能で何ができて、誰がどのように使うか知ることで、初めて適切な対応方針を決められると考えます。

本記事ではアクセシビリティの対応前に知るべきことを紹介します。

## 注意

もし差別に当たるような発言などがありましたら、意図的でなく私の勉強不足によるものです。
気をつけて執筆していますが、まだ勉強中なので至らない可能性が十分にあります。
何かしらの問題を見つけたら、コメントなどでご指摘いただけると助かります。

## 「アクセシビリティ」とは？

英語で「Accessibility（A11y）」といい、「製品やサービスの利用しやすさ」を指します。
「ユーザビリティ」と似ていますが、より多様なユーザーや幅広い利用状況を前提とします。

本記事では目に障害を持つ人（視覚障害者）のみを対象としたアクセシビリティを考えます。

## アクセシビリティに関する機能

各OSに用意されているアクセシビリティに関する機能を紹介します。

### VoiceOver（iOS）

iOSには「VoiceOver」という機能が用意されています。
画面を見なくても操作できるようにするためのツール（いわゆる「スクリーンリーダー」）です。

主な機能は以下の通りです。

- 要素の読み上げ
- 要素間の遷移
- 要素の操作（タップやスワイプなど）

詳細は公式ドキュメントをご参照ください。

https://support.apple.com/ja-jp/guide/iphone/iphe4ee74be8/ios

### TalkBack（Android）

Androidには「TalkBack」という機能が用意されています。
iOSのVoiceOverと同様、画面を見なくても操作できるようにするためのツールです。

詳細は公式ドキュメントをご参照ください。

https://support.google.com/accessibility/android/answer/6283677?hl=ja

## アクセシビリティに関する機能が必要な人

以下のような人がアクセシビリティに関する機能を必要としています。

### 弱視（ロービジョン）

弱視には様々な種類があります。
本記事では「眼鏡などで視力を矯正しても、スマートフォンの画面で文字や画像が見えづらい」ことを弱視と呼ぶことにします。

視覚障害者の8〜9割が弱視といわれています。

最近だと [恋です！〜ヤンキー君と白杖ガール](https://www.ntv.co.jp/yangaru/) というドラマのヒロインが弱視で、看板の文字を読むためにスマホで撮影して画像を拡大して確認する場面など、弱視の人がどのように生活しているかわかります。
ドラマなのですべてが正しいとは限りませんが、純粋に面白いのでオススメです。
（ヒロインの杉咲 花ちゃんが可愛過ぎますよ！）

#### 主な対応

アプリ内の文字サイズを大きくできるようにすべきです。
Androidではあまり意識しなくても文字サイズにOSの設定が反映されますが、iOSはDynamic Typeに対応しないと文字サイズにOSの設定が反映されないので注意です。

### 全盲

まったく目が見えない人です。

#### 主な対応

先ほど紹介したスクリーンリーダーを使い、画面間の各要素に過不足なく適切な順序で遷移でき、適切に読み上げられ、操作できるようにすべきです。

### 色盲

視力には異常がないですが、色の識別が困難な人です。

男性に多く、20人に1人は色盲だといわれています。
私も何人か会ったことがあり、特に赤と緑の識別が難しい人が多い印象でした。

#### 主な対応

色以外で区別できる手段を設けることと、コントラスト比を上げることの主に2つです。

パッと見の色で識別するのは便利なので残しつつ、テキストを添えたりして色以外で区別する手段を併せて用意するのがいいと思います。
コントラスト比は背景色と前景色（文字やアイコンなど）が4.5:1以上だと識別しやすいといわれています。

## 視覚障害者の人数

2006年に厚労省が行った調査によると、約31万人とのことです。
これより新しいデータはわかりませんでした。

参考: [身体障害児・者等実態調査｜厚生労働省](https://www.mhlw.go.jp/toukei/list/108-1.html)

## 視覚障害者のスマホ利用状況

2021年に日本視覚障害者ICTネットワークが行った調査によると、スマホを使っている視覚障害者の方は約9割のようです。
その中でiOSのみを使っている人は約76%、Androidのみを使っている人は約2%、両OSを使っている人は約18%と、約95%の人がiOSを使っており、逆にAndroidのみ使っている人はほとんどいないことがわかります。

アプリの規模や種類、実際のユーザーセグメントなどによりますが、両OS対応のアプリはiOSを優先して対応するのがいいかもしれません。

参考: [第1回支援技術利用状況調査報告書 | 日本視覚障害者ICTネットワーク](https://jbict.net/survey/at-survey-01)

## アクセシビリティに関する公式ドキュメント

AppleやGoogleなどはアクセシビリティに関するガイドラインなどのドキュメントを公開しています。
こちらを一読してから対応すべきです。

### Apple

- [Introduction - Accessibility - Human Interface Guidelines - Apple Developer](https://developer.apple.com/design/human-interface-guidelines/accessibility/overview/introduction/)
- [Accessibility - Apple](https://www.apple.com/accessibility/)
- [iPhoneでVoiceOverをオンにして練習する - Apple サポート (日本)](https://support.apple.com/ja-jp/guide/iphone/iph3e2e415f/ios)
- [Accessibility for UIKit | Apple Developer Documentation](https://developer.apple.com/documentation/uikit/accessibility_for_uikit)

### Google

- [Accessibility - Material Design](https://material.io/design/usability/accessibility.html)
- [Google ユーザー補助機能](https://www.google.com/accessibility/)
- [TalkBack - Android のユーザー補助機能 ヘルプ](https://support.google.com/accessibility/android/topic/3529932?hl=ja&ref_topic=9078845)

### WAIC

- https://waic.jp/
- https://waic.jp/docs/jis2016/understanding/201604/

「WAIC」は「Web Accessibility Infrastructure Committee」の略で、日本語で「ウェブアクセシビリティ基盤委員会」といいます。
「JIS X 8341-3」という指針が公開されており、こちらに準拠するのが望ましいです。
A、AA、AAAと3レベルの達成基準が定められています。

例えば厚労省が提供しているCOCOAはレベルAAに準拠することを目標としています。
「OSの言語設定を日本語にする > COCOA > メニュー > ウェブアクセシビリティ方針」から確認できます。

Webアプリになりますが、母子モというサービスでは2017年度に一部のページについて目標を達成し、試験結果も掲載しています。
参考になるのでリンクを載せます。

https://www.mchh.jp/WebAccessibility

## おわりに

本記事を読んで、少しでもアクセシビリティに関する機能やそれを使う人のことがわかったら嬉しいです。
この後に具体的な対応方法を検討するのがいいと思います。

## 参考リンク

- [アクセシビリティ - Wikipedia](https://ja.wikipedia.org/wiki/アクセシビリティ)
- [アクセシビリティとは | ウェブアクセシビリティ基盤委員会（WAIC）](https://waic.jp/knowledge/accessibility/)
- [弱視 - Wikipedia](https://ja.wikipedia.org/wiki/弱視)
- [弱視 | 日本弱視斜視学会](https://www.jasa-web.jp/general/medical-list/amblyopia)
- [ロービジョン（弱視）の方向けリーフレット「見えにくいことは はずかしいことではありません！」 | 社会福祉法人 日本視覚障害者団体連合](http://nichimou.org/notice/200304-jimu/)
- https://twitter.com/the_uhooi/status/1456428043900096519?s=20
