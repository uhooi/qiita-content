---
title: Kotlinの静的解析ツール「detekt」のセットアップ&操作方法
tags:
  - Android
  - Kotlin
  - detekt
private: false
updated_at: '2021-05-17T22:59:34+09:00'
id: cd1250c6f6de7d4abfd6
organization_url_name: null
slide: false
---
## 「detekt」とは？

Kotlin用の静的解析ツールです。

## 環境

- OS：macOS Big Sur 11.1
- Kotlin：1.5.0
- Gradle：6.8
- Gradle plugin：4.2.1
- detekt：1.17.0

## セットアップ

### インストール

ルート直下の「build.gradle」に `mavenCentral()` を追加します。

```diff_groovy:/build.gradle
buildscript {
    repositories {
+         mavenCentral()
    }
}

allprojects {
    repositories {
+         mavenCentral()
    }
}
```

appフォルダ配下の「build.gradle」にプラグインと設定を追加します。

```diff_groovy:/app/build.gradle
plugins {
+     id 'io.gitlab.arturbosch.detekt' version '1.17.0'
}

+ detekt {
+     buildUponDefaultConfig = true
+     allRules = false
+     config = files("$projectDir/config/detekt/detekt.yml")
+     baseline = file("$projectDir/config/detekt/baseline.xml")
+ 
+     reports {
+         xml {
+             enabled = true
+             destination = file("$buildDir/reports/detekt/detekt.xml")
+         }
+         html {
+             enabled = true
+             destination = file("$buildDir/reports/detekt/detekt.html")
+         }
+         txt {
+             enabled = true
+             destination = file("$buildDir/reports/detekt/detekt.txt")
+         }
+         sarif {
+             enabled = true
+             destination = file("$buildDir/reports/detekt/detekt.sarif")
+         }
+     }
+ }
```

`failFast` は非推奨になったので、代わりに `buildUponDefaultConfig` と `allRules` を使ってください。

```
'failFast' is deprecated. Please use 'buildUponDefaultConfig' together with 'allRules'.
```

### 設定ファイル

ルールや出力フォーマットを設定するファイルです。

デフォルトの設定ファイルを参考に編集するといいです。
https://github.com/arturbosch/detekt/blob/master/detekt-cli/src/main/resources/default-detekt-config.yml

2020/05/11現在、私は最小限の設定のみ行っています。
最初からルールをすべて読み込んで設定すると学習コストがかかるので、育てていく方針にします。

私が設定している内容をひとつずつ紹介します。

#### build

ビルドエラーの設定です。
__デフォルトでは1つでも引っかかるとビルドエラーになる__ ので、適切に設定すべきです。

```yaml:detekt.yml
build:
  maxIssues: 3
  weights:
    ForbiddenComment: 0
    MatchingDeclarationName: 1
    ReturnCount: 0
    TooGenericExceptionCaught: 0
    TooGenericExceptionThrown: 1
```

「引っかかったルール×そのルールのウェイト」の合計が `maxIssues:` で指定した数値を超えるとビルドエラーになります。

`weights` で各ルールのウェイトを設定できます。
私は `ForbiddenComment` がいくつ引っかかってもビルドエラーにしたくないため、 `0` を指定しています。
`MatchingDeclarationName` と `TooGenericExceptionThrown` はそれぞれ1つずつビルドエラーにしたくない箇所があるため `1` を指定し、 `maxIssues` に `3` を指定しています。そうすることで、他に1つでもウェイトが `1` 以上のルールに引っかかることでビルドエラーにできます。

#### rulesets

各ルールの設定です。
私はルールを無効化するときに使っています。

例えば `MagicNumber` ルールは `style` ルールセット内のルールなので、以下のように書きます。
https://detekt.github.io/detekt/style.html#magicnumber

```yaml:detekt.yml
style:
  MagicNumber:
    active: false
```

##### active

`active:` を `false` にすることでルールを無効化できます。

##### severity

`severity:` でルールの重大度を設定できるとのことですが、以下のエラーが発生するので注意です。
ドキュメントに残っているだけで、廃止された設定なのかもしれません。

```shell-session
Property 'style>ForbiddenComment>severity' is misspelled or does not exist.
```

#### console-reports

コンソールにレポートを出力するかの設定です。

```yaml:detekt.yml
console-reports:
  active: true
  # exclude:
  # - 'ProjectStatisticsReport'
  # - 'ComplexityReport'
  # - 'NotificationReport'
  # - 'FindingsReport'
  # - 'FileBasedFindingsReport'
  # - 'BuildFailureReport'
```

`active:` を `true` にすると、コンソールにレポートが出力されます。
`exclude:` で出力したくないレポートを指定できますが、私は特に指定していません。

#### processors

すみません、こちらの設定は理解していません。
とりあえず `active:` を `true` にし、 `exclude:` には何も指定していません。

```yaml:detekt.yml
processors:
  active: true
  # exclude:
  # - 'FunctionCountProcessor'
  # - 'PropertyCountProcessor'
  # - 'ClassCountProcessor'
  # - 'PackageCountProcessor'
  # - 'KtFileCountProcessor'
```

#### output-reports

`output-reports:` を指定すると、以下のエラーが発生するので注意です。
ドキュメントに残っているだけで、廃止された設定なのかもしれません。

```shell-session
$ ./gradlew detekt

> Task :app:detekt FAILED
Property 'output-reports' is misspelled or does not exist.

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':app:detekt'.
> Run failed with 1 invalid config property.

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output. Run with --scan to get full insights.

* Get more help at https://help.gradle.org
```

#### 全体図

最後に私の設定ファイルの全体図を載せます。

```yaml:detekt.yml
build:
  maxIssues: 3
  weights:
    ForbiddenComment: 0
    MatchingDeclarationName: 1
    ReturnCount: 0
    TooGenericExceptionCaught: 0
    TooGenericExceptionThrown: 1

style:
  MagicNumber:
    active: false

console-reports:
  active: true
  # exclude:
  # - 'ProjectStatisticsReport'
  # - 'ComplexityReport'
  # - 'NotificationReport'
  # - 'FindingsReport'
  # - 'FileBasedFindingsReport'
  # - 'BuildFailureReport'

processors:
  active: true
  # exclude:
  # - 'FunctionCountProcessor'
  # - 'PropertyCountProcessor'
  # - 'ClassCountProcessor'
  # - 'PackageCountProcessor'
  # - 'KtFileCountProcessor'
```

### ベースラインファイル

ホワイトリストとブラックリストを設定するファイルです。

ブラックリストは、誤検知されるファイルの指定に使います。
ホワイトリストは、私にはまだわかりません。

2020/05/11現在、私はブラックリストとホワイトリストの両方に何も指定していません。
有用な使い方があれば教えていただきたいです。

```xml:baseline.xml
<SmellBaseline>
  <Blacklist>
  </Blacklist>
  <Whitelist>
  </Whitelist>
</SmellBaseline>
```

## 静的解析

Android Studioのターミナルで `./gradlew detekt` を実行すると、 `destination` で指定したフォルダに静的解析結果が出力されます。

HTMLの出力例を載せます。
<img width="1680" alt="スクリーンショット_2020-04-14_11_32_21.jpg" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/138245/06c9a602-0571-055f-885c-27bc27af798c.jpeg">

`TODO:` コメントが残っていることが検知されています。

## 特定の箇所のみルールを無効化

`@Suppress("{ルール名}")` を使うことで、特定の箇所のみ特定のルールを無効化することができます。
使い過ぎるとルールを設定した意味がなくなるので、意図しない指摘のみに適用するのがいいと思います。

```kotlin
@Suppress("UnusedPrivateMember")
@BindingAdapter("observedList")
fun observeList(recyclerView: RecyclerView, observedList: List<Any>?) {
    recyclerView.adapter?.notifyDataSetChanged()
```

## おわりに

Kotlinのコードを静的解析できるようになりました！
これでKotlinの知識不足を多少は補ってくれるでしょう😅

## 参考リンク

- [arturbosch/detekt: Static code analysis for Kotlin](https://github.com/arturbosch/detekt)
- [detekt | A static code analyzer for Kotlin](https://arturbosch.github.io/detekt/)
- [Detekt Configuration File | A static code analyzer for Kotlin](https://arturbosch.github.io/detekt/configurations.html)
- [Configure Build Failure Thresholds | A static code analyzer for Kotlin](https://detekt.github.io/detekt/failonbuild.html)
- [Reporting | A static code analyzer for Kotlin](https://detekt.github.io/detekt/reporting.html#severity)
- [Code Smell Baseline | A static code analyzer for Kotlin](https://arturbosch.github.io/detekt/baseline.html)
