---
title: SwiftLintの設定ファイルをサーバーに配置して共有する方法
tags:
  - iOS
  - Swift
  - SwiftLint
private: false
updated_at: '2021-04-12T14:11:29+09:00'
id: b5b26caeeefd8dbe1afd
organization_url_name: null
---
## はじめに

本記事は [Swift/Kotlin愛好会 Advent Calendar 2020](https://qiita.com/advent-calendar/2020/love_swift_kotlin) の18日目の記事です。
SwiftLint 0.42.0から設定ファイルをリモートで取得できるようになったので、その方法を紹介します。

## 環境

- OS：macOS Big Sur 11.0.1
- Swift：5.3.2
- Xcode：12.3 (12C33)
- SwiftLint：0.42.0

## 本記事で説明しないこと

- SwiftLintの概要やセットアップ方法  
私が以前書いた記事を参考にしてください。  
[Swiftの静的解析ツール「SwiftLint」のセットアップ方法 - Qiita](https://qiita.com/uhooi/items/bf888b53b4b8210108aa)

## 設定ファイルをサーバーに配置して共有する方法

設定ファイルを作成してサーバーに配置し、ローカルで取得するまでの方法を紹介します。

### 共有する設定ファイルの作成

任意の名前でSwiftLintの設定ファイルを作成します。
通常の設定ファイルと書き方は変わりません。共有したい設定のみ抽出してください。

私は以下を抽出しました。

- disabled_rules
- opt_in_rules
- analyzer_rules
- ルール個別の設定

```yaml:uhooi-base-swiftlint-config.yml
disabled_rules:
  #- block_based_kvo
  # ...

opt_in_rules:
  - anyobject_protocol
  # ...

analyzer_rules:
  - unused_declaration
  # ...

line_length:
  warning: 300
  error: 500

identifier_name:
  min_length:
    warning: 1
```

以下はプロジェクトごとに変わるため、抽出しませんでした。

- included
- excluded

### 共有する設定ファイルをサーバーに配置

作成した設定ファイルをサーバーに配置します。

私はSwiftLintの設定ファイルを配置するためだけのGitHubリポジトリを作成し、そこに配置しました。
https://github.com/uhooi/SwiftLint-Config/blob/main/uhooi-base-swiftlint-config.yml
![スクリーンショット 2020-12-17 22.12.32.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/138245/70fba007-9228-b99a-55f6-2881f18dae1a.png)

SwiftLintの対応バージョンごとにタグを付ければ、SwiftLintのバージョンが異なるプロジェクトでもタグを指定して設定ファイルを取得して使い回せます。
私はGitHubのリリース機能を使っています。
https://github.com/uhooi/SwiftLint-Config/releases
![スクリーンショット 2020-12-17 22.19.03.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/138245/6fbdab88-4949-2e91-3716-110f5a725f46.png)

READMEにSwiftLintのバージョンと設定ファイルのリリースの対応表を載せるとわかりやすいです。
（SwiftLintのREADMEに記載されている、SwiftとSwiftLintのバージョン対応表を参考にしました）
![スクリーンショット 2020-12-17 22.21.20.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/138245/f9e2efd0-55d5-8452-4687-3c266a5f56e0.png)

### 取得先の設定ファイルの設定

サーバーに配置した設定ファイルを、他の設定ファイルから読み込みます。
`parent_config` を追記するのみです。

```yaml:.swiftLint.yml
parent_config: {共有する設定ファイルのURL}
```

GitHubリポジトリに配置している場合、 `https://raw.githubusercontent.com/{ユーザー名}/{リポジトリ名}/{タグまたはブランチ名}/{設定ファイル名}` で設定ファイルを取得できます。

```yaml:.swiftLint.yml
parent_config: https://raw.githubusercontent.com/uhooi/SwiftLint-Config/v1.0.0/uhooi-base-swiftlint-config.yml
```

`parent` の名前通り、サーバーに配置した設定ファイルが親の設定になります。
子の設定が優先されて上書きできます。

私は自分が作成した設定ファイルなのでルールは一切上書きせず、 `included` と `excluded` のみ設定しています。

```yaml:.swiftlint.yml
parent_config: https://raw.githubusercontent.com/uhooi/SwiftLint-Config/v1.0.0/uhooi-base-swiftlint-config.yml

included:
  - Shared
  - UhooiPicBook
  - UhooiPicBookStickers
  - UhooiPicBookWidgets
  - UhooiPicBookWidgetsConfigurableIntent
  #- UhooiPicBookTests
  #- UhooiPicBookUITests

excluded:
  - UhooiPicBook/Generated
```

設定ファイルがだいぶスッキリしました！

### サーバーに配置した設定ファイルの取得

`parent_config` に設定ファイルのURLを指定してSwiftLintを実行すると、 `.swiftlint/RemoteConfigCache` フォルダにリモートの設定ファイルがダウンロードされます。

上記のフォルダは名前の通りキャッシュの役目を果たすため、すでに設定ファイルが存在する場合はそのまま使われます。

ダウンロード時に `.gitignore` へキャッシュフォルダが自動で書き込まれるので、そのままコミットしてください。

```diff_yaml:.gitignore
+ 
+ 
+ # SwiftLint Remote Config Cache
+ .swiftlint/RemoteConfigCache 
```

リモートのタイムアウトはデフォルト2秒、キャッシュが存在する場合はデフォルト1秒です。
変更する場合は `remote_timeout` または `remote_timeout_if_cached` を指定します。

## おわりに

これでSwiftLintの設定を複数のプロジェクトでかんたんに共有できます！
私の設定ファイルを親として使ってもいいですよ :relaxed: 

以上、 [Swift/Kotlin愛好会 Advent Calendar 2020](https://qiita.com/advent-calendar/2020/love_swift_kotlin) の18日目の記事でした。
明日はまだ埋まっていません。ぜひ [こちら](https://qiita.com/advent-calendar/2020/love_swift_kotlin/day/new?day=19) から参加しましょう！

## 参考リンク

- [Release 0.42.0: He Chutes, He Scores · realm/SwiftLint](https://github.com/realm/SwiftLint/releases/tag/0.42.0)
- [realm/SwiftLint at 0.42.0](https://github.com/realm/SwiftLint/tree/0.42.0)
- [Feature/bump swiftlint to 0 42 0 by uhooi · Pull Request #152 · uhooi/UhooiPicBook](https://github.com/uhooi/UhooiPicBook/pull/152)
