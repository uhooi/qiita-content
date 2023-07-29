---
title: ADBでよく使うコマンド一覧
tags:
  - Android
  - adb
  - AndroidStudio
private: false
updated_at: '2021-02-08T09:06:54+09:00'
id: fb103ad3e263b0bb0abf
organization_url_name: null
slide: false
---
## はじめに

ADBでよく使うコマンドを紹介します。

## 「ADB」とは？

「Android Debug Bridge」の略で、Android端末とやりとりするための多用途なコマンドです。
アプリのインストールやアンインストール、クラッシュログの出力など、さまざまな操作が行えます。

## 環境

- Android Studio：4.0
- adb：1.0.41

## ADBのパスを通す

ADBを使うには、Android Studioをインストールし、パスを通す必要があります。

以下の記事をご参考に行ってください。
[Android SDK Platform-Toolsのパスを通す方法 - Qiita](https://qiita.com/uhooi/items/a3dcc15f7e15ae11d1d6)

## ADBでよく使うコマンド

### グローバルオプション

|オプション|説明|
|:--|:--|
|`-s {シリアル}`|対象シリアルの端末を使う|

`-s` オプションを使うと、対象にしたい端末を確実に指定できます。
後述する `adb devices` で対象端末のシリアルを調べて指定するのがオススメです。

### adb version

|コマンド|説明|
|:--|:--|
|`adb version`|ADBのバージョンを出力する|

### adb help

|コマンド|説明|
|:--|:--|
|`adb help`|ヘルプを出力する|

### adb devices

|コマンド|説明|
|:--|:--|
|`adb devices`|接続されている端末の一覧を出力する|
|`adb devices -l`|接続されている端末の一覧を長い形式で出力する|

### adb install

|コマンド|説明|
|:--|:--|
|`adb install {パッケージ}`|対象パッケージ（APKファイルなど）をインストールする|
|`adb install -r {パッケージ}`|インストール済の対象パッケージを置き換えてインストールする|
|`adb install -d {パッケージ}`|対象パッケージのダウングレードを許可してインストールする（デバッグできるパッケージのみ）|

### adb uninstall

|コマンド|説明|
|:--|:--|
|`adb uninstall {パッケージ}`|対象パッケージをアンインストールする|

### adb push

|コマンド|説明|
|:--|:--|
|`adb push {ローカルのファイルパス} {端末のファイルパス}`|ローカルの対象ファイル（またはフォルダ）を端末にコピーする|

### adb pull

|コマンド|説明|
|:--|:--|
|`adb pull {端末のファイルパス} {ローカルのファイルパス}`|端末の対象ファイル（またはフォルダ）をローカルにコピーする|

### adb logcat

|コマンド|説明|
|:--|:--|
|`adb logcat`|デバイスログを出力する|
|`adb logcat --help`|`adb logcat` のヘルプを出力する|
|`adb logcat -b {バッファ}`|別のリングバッファを要求してデバイスログを出力する|
|`adb logcat -b crash`|クラッシュログを出力する|

`^C` を入力しないと出力が終わらないので注意です。

### adb shell

|コマンド|説明|
|:--|:--|
|`adb shell`|シェルを起動する|
|`adb shell screencap -p {端末の画像パス}`|対象パスにスクリーンショットを保存する|
|`adb shell screenrecord {端末の動画パス}`|対象パスにスクリーンレコードを保存する（ `^C` で撮影終了する）|

## ADBでよく使うテクニック

### アプリの外部ストレージのデータをデスクトップにコピーする

以下を実行すると、端末内の `files` フォルダがデスクトップにコピーされます。

```shell-session
$ adb -s {シリアル} pull /storage/emulated/0/Android/data/{パッケージ}/files ~/Desktop/
```

上記のパスは `getExternalFilesDir()` メソッドの戻り値です。
環境によって異なるため、適宜置き換えてください。

### アプリの内部ストレージにアクセスする

`adb shell` でシェルを起動し、 `run-as {パッケージ}` でアプリの内部ストレージにアクセスできます。

```shell-session
$ adb -s {シリアル} shell
sargo:/ $ run-as {パッケージ}
sargo:/data/user/0/{パッケージ} $
```

あとは `ls` や `cd` コマンドなどでどのようなファイルが存在するか確認できます。

### スクリーンショットを撮ってデスクトップにコピーする

`adb shell screencap` でスクリーンショットを撮って端末内に保存し、 `adb pull` でデスクトップにコピーします。

```shell-session
$ adb -s {シリアル} shell screencap -p /sdcard/screenshot.png
$ adb -s {シリアル} pull /sdcard/screenshot.png ~/Desktop/
/sdcard/screenshot.png: 1 file pulled, 0 skipped. 23.0 MB/s (3150517 bytes in 0.130s)
```

### スクリーンレコードを撮ってデスクトップにコピーする

スクリーンショットとほぼ同様です。
`adb shell screenrecord` でスクリーンレコードを撮って端末内に保存し、 `adb pull` でデスクトップにコピーします。

```shell-session
$ adb -s {シリアル} shell screenrecord /sdcard/screen.mp4
$ adb -s {シリアル} pull /sdcard/screen.mp4 ~/Desktop/
/sdcard/screen.mp4: 1 file pulled, 0 skipped. 26.7 MB/s (22255200 bytes in 0.795s)
```

### クラッシュログをテキストファイルに出力する

`adb logcat` は標準出力されるため、テキストファイルに出力して __Vim__ などのテキストエディタで開くと見やすいです。
「Caused by:」 にクラッシュの原因が書かれています。

```shell-session
$ adb -s {シリアル} logcat -b crash > ~/Desktop/logcat_crash.log
```

## おわりに

これでADBのコマンドを忘れたときは、本記事を見れば解決できるかもしれません！

他にもよく使うコマンドがありましたら、コメントなどで教えていただけると嬉しいです :relaxed: 

## 参考リンク

- [Android Debug Bridge（adb）  |  Android デベロッパー  |  Android Developers](https://developer.android.com/studio/command-line/adb?hl=ja)
- [複数ユーザーのテスト  |  Android オープンソース プロジェクト  |  Android Open Source Project](https://source.android.com/devices/tech/admin/multi-user-testing)
- [AndroidでPCからadbコマンドで動画・静止画キャプチャする - littlewing](https://littlewing.hatenablog.com/entry/2016/12/20/133901)
- [[Android] アプリの内部メモリを覗くとパーミッションでブロックされる](https://akira-watson.com/android/adb_run-as.html)
