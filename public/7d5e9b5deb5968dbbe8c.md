---
title: SELECT文の検索結果をCSVへ出力するSQL(Oracle)
tags:
  - SQL
  - oracle
  - DB
  - CSV
  - select
private: false
updated_at: '2018-03-28T12:48:39+09:00'
id: 7d5e9b5deb5968dbbe8c
organization_url_name: null
slide: false
---
## はじめに

システムに不具合が発生し、データに問題があることまで調査できたとします。
そのとき、手っ取り早くDBの中身をCSVで出力して確認したいとき、ありませんか？
私はありました。

OracleにはSQLの実行結果をファイルに出力する `SPOOL` コマンドが用意されているので、それを応用してCSVへ出力するSQLを実装しました。
調査の際などにご活用ください。

## 環境

- DB：Oracle Database 10g Release 10.2.0.5.0 - 64bit Production

## CSV出力SQL

```sql:output_csv.sql
-- 検索結果をCSVへ出力する

-- ①SQL設定
-- コンソールメッセージを非表示にする
SET ECHO OFF
-- 1行に出力するバイト数
-- 少ないと見切れるので最大に設定する
SET LINESIZE 32767
-- 1ページの行数
-- 少ないとヘッダーが多くて見づらいので無制限に設定する
SET PAGESIZE 0
-- 「○○行が選択されました」メッセージを非表示にする
SET FEEDBACK OFF
-- 区切り文字をカンマに指定する
SET COLSEP ','
-- 各行の右端にあるスペースを削除する
SET TRIMSPOOL ON

-- ②出力開始
-- 出力パスは適宜変更する
SPOOL C:\output.csv

-- ③出力内容
-- 検索文は適宜変更する
SELECT *
  FROM {テーブル名}
;

-- ④出力終了
SPOOL OFF
```

## 追記

[コメント](https://qiita.com/uhooi/items/7d5e9b5deb5968dbbe8c#comment-9e9ccaaced6692a0618e)を頂いて気づいたのですが、この方法だと固定長となり、各列の右にスペースが含まれてしまいます。

CSV出力したデータを目視以外で使うなど、可変長で取得したい場合は、列をカンマで結合して1つの列として検索するしかなさそうです。

```sql
SELECT {列1} || ',' ||
       {列2} || ',' ||
       {列3}
  FROM {テーブル名}
;
```

## 参考リンク

- [ＯＲＡＣＬＥ／オラクルＳＱＬリファレンス（SQLPLUS）](http://oracle.se-free.com/utl/C1_csv.html)
- [SQL*Plus で実行した結果をテキストファイルに出力する - Oracle - Project Group](http://www.projectgroup.info/tips/Oracle/Oracle_000003.html)
- [SET LINESIZE - オラクル・Oracle SQL*Plus リファレンス](http://www.shift-the-oracle.com/sqlplus/system-variable/linesize.html)
- [SET PAGESIZE - オラクル・Oracle SQL*Plus リファレンス](http://www.shift-the-oracle.com/sqlplus/system-variable/pagesize.html)
