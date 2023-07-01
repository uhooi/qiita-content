---
title: iTunesの曲をiOSの着信音(アラーム音)に変換するスクリプトを作った
tags:
  - ShellScript
  - Bash
  - Mac
  - iOS
  - iTunes
private: false
updated_at: '2018-05-31T22:37:06+09:00'
id: dacd8cb6eede97b0d1b0
organization_url_name: null
---
## はじめに

私は毎朝iPhoneのアラーム機能を使って起きています:alarm_clock:

そこで長年思っているのですが、 __用意されている音が弱すぎて全然起きれません__ 。
デフォルトの「レーダー」や「さざなみ」「シルク」など、全体的にパワーが足りな過ぎます。

__遅刻する学生やサラリーマンが減らないのはこのせいに違いありません__ 。

そこで私はiTunesに入っている曲から着信音を作り、それをアラーム音に設定するようにしました。
今までは手動で作成していたのですが、手間になってきてスクリプトを実装したので公開します。

## 注意

__作成した着信音は著作権の範囲内でご使用ください。__

## 環境

- OS：macOS High Sierra 10.13.1
- FFmpeg：4.0
- 音声データ：Apple Losslessオーディオファイル(\*.m4a)  
自分用に作ったということもあり、mp3など他の形式には対応させていません。

## FFmpegのインストール

「FFmpeg」とは、音声や動画データをコマンドラインで編集できるツールです。
今回は音声データのトリミングを自動化するために使います。

Macの場合、Homebrewからインストールします。

```bash
$ brew install ffmpeg
```

類似のツールに「SoX」というのがあるのですが、こちらは.m4aファイルに非対応のようなので今回は使いませんでした。

## 着信音変換スクリプトの実装

必要なツールのインストールが完了したら、スクリプトを実装します。

やっている処理は大きく分けて2つだけなので単純です。
①音声データ(\*.m4a)を指定した時間から着信音の最大時間(39秒)までトリミング
②それを着信音の拡張子(\*.m4r)に変更

```bash:convert_m4a_to_m4r.sh
#!/bin/bash

# ----------------------------------------
# iOSの着信音を作成する
# 引数：$1 Appleの音声データパス(/path/to/*.m4a)
# 　　：$2 開始時間(秒)
# 出力：iOSの着信音(/path/to/*.m4r)
# 備考：音声データパスにスペースが存在するとエラーになる
# ----------------------------------------

# 定数定義
readonly RINGTONE_MAX_SEC=39

# 引数の保持
readonly SOUND_PATH=$1
readonly START_DATE_SEC=$2

function main () {
  checkArgument
  convertM4aToM4r

  exit
}

# Appleの音声データを着信音に変換する
function convertM4aToM4r () {
  # 音声データを開始時間から着信音の最大秒数までトリミングする
  local readonly TMP_PATH=/tmp/tmp.m4a
  ffmpeg -i ${SOUND_PATH} -t ${RINGTONE_MAX_SEC} -ss ${START_DATE_SEC} ${TMP_PATH}

  # トリミングした音声データのファイル名を「{元のファイル名}.m4r」に変更する
  mv ${TMP_PATH} ${SOUND_PATH%.m4a}.m4r
}

# 引数チェック
function checkArgument () {
  # 音声データチェック
  # 音声データが存在しない、またはファイルでない場合、
  # または拡張子が「.m4a」でない場合はエラー
  if [ ! -e ${SOUND_PATH} ] || [ ! -f ${SOUND_PATH} ] ||
     [ "${SOUND_PATH##*.}" != "m4a" ]; then
    echo "音声データが不正です。"
    exit
  fi

  # 時間チェック
  # 開始時間が0秒未満の場合はエラー
  if [ "$(echo "${START_DATE_SEC} < 0" | bc)" -eq 1 ]; then
    echo "開始時間は0秒以上の時間を指定してください。"
    exit
  fi
}

main
```

## 着信音に変換

スクリプトの実装が完了したら、いよいよ着信音に変換します。

スクリプトと音声データはどこに置いてもいいですが、両方ともデスクトップに置くとやりやすいです。

音声データはiTunesからデスクトップにドラッグ&ドロップするだけでコピーできます。
<img width="950" alt="スクリーンショット_2018-05-27_4_28_40.jpg" src="https://qiita-image-store.s3.amazonaws.com/0/138245/184f8864-d009-1588-e0bd-6a64a721a207.jpeg">

ファイル名にスペースが含まれているとエラーになるので、含まれている場合は削除します。

今回はコレサワさんの「SSW」という曲を変換します。(最近のお気に入り:notes:)
こちらの曲は72.5秒くらいからサビなので、そこから始まるようにします。

```bash
# sh convert_m4a_to_m4r.sh {音声データパス} {開始時間}
$ sh convert_m4a_to_m4r.sh ~/Desktop/SSW.m4a 72.5
```

処理が始まると以下のように出力されます。
![スクリーンショット_2018-05-27_4_01_12.jpg](https://qiita-image-store.s3.amazonaws.com/0/138245/81ee828d-ad40-4ce1-a429-4a2191b13775.jpeg)

処理が正常に完了すると、音声データと同じ場所に着信音のデータが同名で出力されます。
<img width="80" alt="スクリーンショット 2018-05-27 4.09.13.png" src="https://qiita-image-store.s3.amazonaws.com/0/138245/e511c659-a7ec-e1ce-6cf6-ac28fcbbff45.png">

アイコンの中心をクリックすると再生できるので、変な位置から始まっていないか確認します。
<img width="90" alt="スクリーンショット 2018-05-27 11.37.39.png" src="https://qiita-image-store.s3.amazonaws.com/0/138245/e99d21cd-04ce-1e64-e049-20254016296b.png">

おかしかった場合、自分の好きな位置から始まるまで秒数を変えてスクリプトを実行し直します。
出力されるファイルは上書きされるので、削除せずにそのまま実行できます。

問題なく着信音が作成できたら、コピーした音声データを削除します。
コピー元はiTunesの方に残っているので削除して問題ありません。

```bash
$ rm ~/Desktop/SSW.m4a
```

## 着信音に登録

PCにiPhoneを接続してiTunesを開き、[着信音]に作成したファイルをドラッグ&ドロップします。
<img width="957" alt="スクリーンショット_2018-05-27_4_36_25.jpg" src="https://qiita-image-store.s3.amazonaws.com/0/138245/de06b409-3da4-7e0f-6838-3f2c13ef6e8c.jpeg">

これでiPhoneに着信音として登録されたので、あとはアラームで選択すれば完了です。
<img width="320" alt="BA6A8A33-8CAB-46DC-B1D4-8FBFBF10BEBE.png" src="https://qiita-image-store.s3.amazonaws.com/0/138245/068079b1-e23b-e6e5-5f84-f899211c4c0b.png">

## おわりに

これでみなさんも毎朝寝坊せずに起きれるようになります！:sunrise_over_mountains:

## 参考リンク

- [iOS 8でもOK！ iTunesに入っている好きな曲をiPhoneの着信音にする方法 - 週刊アスキー](http://weekly.ascii.jp/elem/000/000/259/259687/)
