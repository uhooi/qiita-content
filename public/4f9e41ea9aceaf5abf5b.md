---
title: ロガーライブラリ「Timber」のセットアップ&操作方法(Kotlin)
tags:
  - Android
  - Kotlin
  - log
  - logger
  - Timber
private: false
updated_at: '2021-04-12T12:58:01+09:00'
id: 4f9e41ea9aceaf5abf5b
organization_url_name: null
slide: false
---
## はじめに

Androidアプリ開発で使えるロガーライブラリを教えてもらったので、導入することにしました。

## 「Timber」とは？

Android向けのロガーライブラリです。

## 環境

- OS：macOS Big Sur 11.1
- Android Studio：4.1.2
- Kotlin：1.4.20
- Gradle：6.8
- Gradle plugin：4.1.2
- Timber：4.7.1

## セットアップ

### Timberのインストール

appフォルダ配下の「build.gradle」に依存関係を追加します。

```diff_groovy:/app/build.gradle
dependencies {
+     implementation 'com.jakewharton.timber:timber:4.7.1'
}
```

### アプリケーションクラスでの初期化

アプリケーションクラスの `onCreate()` で `Tree` インスタンスをインストールします。

```diff_kotlin
+ import timber.log.Timber

class UhooiPicBookApp : Application() {

    override fun onCreate() {
        super.onCreate()

+         configureTimber()
    }

+     private fun configureTimber() {
+         if (BuildConfig.DEBUG) {
+             Timber.plant(Timber.DebugTree())
+         }
+     }
}
```

私はデバッグ時のみTimberを使うようにしています。

これでTimberのセットアップは完了です。

## 使い方

Timberの各メソッドを呼び出すことで、Logcatにログを出力します。

```kotlin:MainActivity.kt
import timber.log.Timber

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContentView(R.layout.activity_main)

        Timber.v("Verbose")
        Timber.d("Debug")
        Timber.i("Info")
        Timber.e("Error")
        Timber.w("Warning")
    }
}
```

```:Logcatの出力結果
2021-03-02 21:58:46.736 29451-29451/com.theuhooi.uhooipicbook V/MainActivity: Verbose
2021-03-02 21:58:46.736 29451-29451/com.theuhooi.uhooipicbook D/MainActivity: Debug
2021-03-02 21:58:46.736 29451-29451/com.theuhooi.uhooipicbook I/MainActivity: Info
2021-03-02 21:58:46.736 29451-29451/com.theuhooi.uhooipicbook E/MainActivity: Error
2021-03-02 21:58:46.737 29451-29451/com.theuhooi.uhooipicbook W/MainActivity: Warning
```

インスタンスの出力は、String templatesを使うとかんたんです。

```kotlin:MainActivity.kt
import timber.log.Timber

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContentView(R.layout.activity_main)

        Timber.d("$this")
        Timber.d("${this.actionBar}") // プロパティやメソッドを呼ぶ場合は波括弧 `{ }` で括る必要がある
    }
}
```

```:Logcatの出力結果
2021-03-06 18:52:58.661 24680-24680/com.theuhooi.uhooipicbook D/MainActivity: com.theuhooi.uhooipicbook.MainActivity@95e20bb
2021-03-06 18:52:58.661 24680-24680/com.theuhooi.uhooipicbook D/MainActivity: null
```

## おわりに

かんたんにログ出力を実現できました！
どのようなアプリの開発でも有用だと思います。

## 参考リンク

- [JakeWharton/timber: A logger with a small, extensible API which provides utility on top of Android's normal Log class.](https://github.com/JakeWharton/timber)
- [timber/ExampleApp.java at master · JakeWharton/timber](https://github.com/JakeWharton/timber/blob/master/timber-sample/src/main/java/com/example/timber/ExampleApp.java)
- [Timber (Timber 3.0.1 API)](http://jakewharton.github.io/timber/)
- [Setup Timber · Issue #80 · uhooi/UhooiPicBook-Android](https://github.com/uhooi/UhooiPicBook-Android/issues/80)
- [Setup Timber #80 by omooooori · Pull Request #85 · uhooi/UhooiPicBook-Android](https://github.com/uhooi/UhooiPicBook-Android/pull/85)
- [Refactor Timber by uhooi · Pull Request #104 · uhooi/UhooiPicBook-Android](https://github.com/uhooi/UhooiPicBook-Android/pull/104)
- [Basic types—Kotlin](https://kotlinlang.org/docs/basic-types.html#string-templates)
- https://twitter.com/Susan_jacko/status/1368133369251307525?s=20
