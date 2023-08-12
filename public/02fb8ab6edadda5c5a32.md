---
title: プロトコルで関数にデフォルト引数を指定する方法(Swift)
tags:
  - Protocol
  - Swift
private: false
updated_at: '2023-08-02T00:14:24+09:00'
id: 02fb8ab6edadda5c5a32
organization_url_name: dena_coltd
slide: false
---
## はじめに

プロトコルで関数にデフォルト引数を指定しようとすると、以下のビルドエラーが発生します。

```swift:LoggerProtocol.swift
protocol LoggerProtocol {
    // ✅
    func debug(_ message: String, file: String, function: String, line: Int, column: Int)

    // ❌: Default argument not permitted in a protocol method
    func debug(_ message: String, file: String = #file, function: String = #function, line: Int = #line, column: Int = #column)
}
```

しかしログ出力のたびに `#file` や `#function` を渡すのは非常に手間です。
実はデフォルト引数を指定する方法があるので紹介します。

## 環境

- OS：macOS Ventura 13.4.1
- Xcode：14.3.1 (14E300c)
- Swift：5.8.1

## プロトコルで関数にデフォルト引数を指定する方法

プロトコルをextensionで拡張することで、関数にデフォルト引数を指定できます。

```diff_swift:LoggerProtocol.swift
protocol LoggerProtocol {
    func debug(_ message: String, file: String, function: String, line: Int, column: Int)
}

+ extension LoggerProtocol {
+     func debug(_ message: String, file: String = #file, function: String = #function, line: Int = #line, column: Int = #column) {
+         debug(message, file: file, function: function, line: line, column: column)
+     } 
+ }
```

プロトコルに準拠したクラスを実装します。

```diff_swift:Logger.swift
+ final class Logger {
+     static let shared = Logger()
+ 
+     private init() {}
+ }
+ 
+ extension Logger: LoggerProtocol {
+     func debug(_ message: String, file: String, function: String, line: Int, column: Int) {
+         print("\(file).\(function):\(line):\(column) - \(message)")
+     }
+ }
```

実際に引数を省略して呼び出せることを確認します。

```swift
let logger: some LoggerProtocol = Logger.shared
logger.debug("Hello, Swift!")
```

無事に呼び出すことができました。

```
main.swift.Page_Contents:24:13 - Hello, Swift!
```

## おわりに

今までプロトコルの関数にデフォルト引数を指定できないと思っていたので、 [Twitterで教えていただき](https://twitter.com/omochimetaru/status/1686339551470768129) 本当に感謝です。

## 参考リンク

- https://twitter.com/the_uhooi/status/1686276951521267712
- [Update logger by uhooi · Pull Request #372 · uhooi/UhooiPicBook](https://github.com/uhooi/UhooiPicBook/pull/372)
