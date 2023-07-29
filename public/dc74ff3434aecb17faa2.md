---
title: 改行コード一覧
tags:
  - Vim
private: false
updated_at: '2021-04-12T17:07:47+09:00'
id: dc74ff3434aecb17faa2
organization_url_name: null
slide: false
---
私はiOSとWindows、たまにLinuxでアプリを開発しています。どのOSでどの改行コードを使うかわからなくなるときがあるため、一覧にしてみました。

|改行コード名|意味|改行コード|OS|fileformat(Vim)|
|:--|:--|:--|:--|:--|
|CR(Carriage Return)|復帰。キャレットを左端の位置に戻すこと|\r|Mac OS(9以前)|mac|
|LF(Line Feed)|改行。キャレットを新しい行に移動すること|\n|UNIX系全般|unix|
|CRLF|復帰+改行。キャレットを左端の位置に戻して改行すること|\r\n|Windows系全般|dos|

## 参考リンク
- [備忘録：改行コード「LF」と「CR」と「CRLF」の違い - marusunrise2〜Hello from Rwanda〜](http://marusunrise2.blogspot.jp/2014/06/lfcrcrlf.html)
- [テキストファイル - 改行の、\nと\r\nの違いは何ですか？ - スタック・オーバーフロー](https://ja.stackoverflow.com/questions/12897/%E6%94%B9%E8%A1%8C%E3%81%AE-n%E3%81%A8-r-n%E3%81%AE%E9%81%95%E3%81%84%E3%81%AF%E4%BD%95%E3%81%A7%E3%81%99%E3%81%8B)
- [Vimで改行コードを変更する - Qiita](https://qiita.com/bezeklik/items/aca37ffb127821311d6b)
