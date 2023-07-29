---
title: GitHub ActionsでAABファイル生成のCDを構築する方法
tags:
  - Android
  - GitHub
  - gradle
  - GitHubActions
private: false
updated_at: '2021-02-07T14:48:45+09:00'
id: 14f1d8c5292e1bcbbfdd
organization_url_name: null
slide: false
---
## はじめに

GitHub Actionsを使い、AABファイルを生成するCDを構築します。

## 本記事で書かないこと

- GitHub Actionsの概要や基本的な操作方法  
[私が以前書いた記事](https://qiita.com/uhooi/items/29664ecf0254eb637951) が参考になると思います

## 事前準備: ローカルでAABファイルを生成できるようにする

私が以前書いた記事を参考にして、ローカルでAABファイルを生成できるようにします。
https://qiita.com/uhooi/items/3bee37d763642c703738

## キーストアのエンコード

2021/02/06現在、GitHubではファイルを安全に管理する方法がないため、機密ファイルは文字列に変換する必要があります。

キーストアをBase64形式でエンコードします。
ローカルで以下のコマンドを実行します。

```shell-session
$ base64 ./app/release.keystore
```

標準出力された内容から __改行を削除して__ コピーします。
Base64形式でエンコードした結果は常に同じなので、メモる必要はありません。

## Repository secretsへの追加

以下をGitHubの安全な場所（Repository secrets）で保持します。

- Base64形式でエンコードしたキーストア（ `ENCODED_RELEASE_KEYSTORE` ）
- 鍵のパスワード（ `RELEASE_KEYSTORE_KEY_PASSWORD` ）
- キーストアのパスワード（ `RELEASE_KEYSTORE_STORE_PASSWORD` ）

![スクリーンショット 2021-02-06 13.33.43.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/138245/ab503809-669b-fcca-d3e8-db2649e84a90.png)

## 設定ファイルの紹介

設定ファイルの内容を上から順に紹介します。

### name

[GitHub ActionsでAndroidアプリのCIを構築する方法](https://qiita.com/uhooi/items/70ffe67ba65d33189db2#name) と同様なので省略します。

```yaml:generate-aab.yml
name: "Generate AAB"
```

### on

[GitHub ActionsでAndroidアプリのCIを構築する方法](https://qiita.com/uhooi/items/70ffe67ba65d33189db2#name) と同様なので省略します。

私はリリースブランチのプッシュをトリガーにしています。

```yaml:generate-aab.yml
on:
  push:
    branches:
      - release/*
    paths-ignore:
      - docs/**
      - README.md
      - LICENSE
```

### jobs

今回はジョブを1つのみ用意しています。

#### generate-aab

AABファイルを生成するジョブです。

基本的にはローカルでAABファイルを生成するコマンドをそのまま実行しているだけなので、詳細は省略します。

リリース用のビルドなので、Gradleのキャッシュは使っていません。
アップロード時にAABファイルが存在しなかったらエラーにしています。

```yaml:generate-aab.yml
jobs:
  generate-aab:
    runs-on: ubuntu-latest
    steps:
    # チェックアウト
    - uses: actions/checkout@v2

    # JDKのセットアップ
    - name: set up JDK 1.8
      uses: actions/setup-java@v1
      with:
        java-version: 1.8

    # 依存関係の出力
    - name: Displays the Android dependencies of the project
      run: ./gradlew androidDependencies

    # キーストアのデコード
    - name: Decode Keystore
      run: echo ${{ secrets.ENCODED_RELEASE_KEYSTORE }} | base64 --decode > ./app/release.keystore

    # AABの生成
    - name: Generate AAB
      run: ./gradlew :app:bundleRelease
      env:
        RELEASE_KEYSTORE_STORE_PASSWORD: ${{ secrets.RELEASE_KEYSTORE_STORE_PASSWORD }}
        RELEASE_KEYSTORE_KEY_PASSWORD: ${{ secrets.RELEASE_KEYSTORE_KEY_PASSWORD }}

    # AABのアップロード
    - name: Upload AAB Artifact
      uses: actions/upload-artifact@v2
      with:
        name: aab
        path: app/build/outputs/bundle/release/app-release.aab
        if-no-files-found: error
```


## 設定ファイルの全体図

最後に設定ファイルの全体図を載せます。

```yaml:generate-aab.yml
name: "Generate AAB"

on:
  push:
    branches:
      - release/*
    paths-ignore:
      - docs/**
      - README.md
      - LICENSE

jobs:
  generate-aab:
    runs-on: ubuntu-latest
    steps:
    # チェックアウト
    - uses: actions/checkout@v2

    # JDKのセットアップ
    - name: set up JDK 1.8
      uses: actions/setup-java@v1
      with:
        java-version: 1.8

    # 依存関係の出力
    - name: Displays the Android dependencies of the project
      run: ./gradlew androidDependencies

    # キーストアのデコード
    - name: Decode Keystore
      run: echo ${{ secrets.ENCODED_RELEASE_KEYSTORE }} | base64 --decode > ./app/release.keystore

    # AABの生成
    - name: Generate AAB
      run: ./gradlew :app:bundleRelease
      env:
        RELEASE_KEYSTORE_STORE_PASSWORD: ${{ secrets.RELEASE_KEYSTORE_STORE_PASSWORD }}
        RELEASE_KEYSTORE_KEY_PASSWORD: ${{ secrets.RELEASE_KEYSTORE_KEY_PASSWORD }}

    # AABのアップロード
    - name: Upload AAB Artifact
      uses: actions/upload-artifact@v2
      with:
        name: aab
        path: app/build/outputs/bundle/release/app-release.aab
        if-no-files-found: error
```

## おわりに

GitHub ActionsでAABファイルを生成できました！

## 参考リンク

- [Add generate AAB pileline by uhooi · Pull Request #93 · uhooi/UhooiPicBook-Android](https://github.com/uhooi/UhooiPicBook-Android/pull/93)
- [Android App Bundleをコマンドラインで署名して生成する方法 - Qiita](https://qiita.com/uhooi/items/3bee37d763642c703738)
- [Base64 - Wikipedia](https://ja.wikipedia.org/wiki/Base64)
