---
title: private extensionでクラスや構造体を分割する(Swift)
tags:
  - Swift
  - SwiftUI
private: false
updated_at: '2023-05-20T12:46:40+09:00'
id: 6a64e4c1dc6eb0cb8ece
organization_url_name: dena_coltd
---
## はじめに

private extensionでクラスや構造体を分割するパターンを紹介します。

## 環境

- OS：macOS Ventura 13.3.1 (a)
- Xcode：14.3 (14E222b)
- Swift：5.8

## 分割前

以下のようなSwiftUIのViewがあります。

```swift:SakatsuInputView.swift
struct SakatsuInputView: View {
    var body: some View {
        Form {
            generalSection
            forewordSection
            temperaturesSection
        }
    }

    private var generalSection: some View {
        // ...
    }

    private var forewordSection: some View {
        // ...
    }

    private var temperaturesSection: some View {
        // ...
    }
}
```

各セクションは他のクラスや構造体から呼ばれないので `private` を付けています。

このままでも問題ないですが、Viewのプロパティやメソッドが他から呼ばれることは基本的にないので、わざわざ `private` を付けるとかえってノイズになるという意見もあります。

でも私は他から呼ばれないことを保証したいです。

そんなときにprivate extensionが使えます。

## 分割後

private extensionを作成し、各セクションを移動しました。

private extension内のプロパティやメソッドは、他のファイルから呼べません。
そのためプロパティやメソッドから `private` を外すことができます。

```swift:SakatsuInputView.swift
struct SakatsuInputView: View {
    var body: some View {
        Form {
            generalSection
            forewordSection
            temperaturesSection
        }
    }
}

// MARK: - Privates

private extension SakatsuInputView {
    var generalSection: some View {
        // ...
    }

    var forewordSection: some View {
        // ...
    }

    var temperaturesSection: some View {
        // ...
    }
}
```

ノイズが排除され、さらにextensionでグルーピングすることで元の構造体がスッキリしました。

## 注意: private extensionはfileprivate

private extensionはfileprivateなので、同一ファイルからだと呼び出せてしまいます。

```swift
struct Foo {}
private extension Foo {
    func foo() {
        print("Foo")
    }
}

struct Bar {
    func bar() {
        Foo().foo() // !!!: 他の構造体からprivate extension内のメソッドが呼び出せる
    }
}

Bar().bar() // "Foo"
```

そのため、できる限り1ファイル1クラス・構造体にするのが望ましいです。

## おわりに

好みもありますが、私はSwiftUIのViewでは積極的にprivate extensionで処理を分割しようと思いました。

分割しない場合は、中途半端に `private` を付けると混乱するので、 `private` を付けるなら他から呼ばれないすべての処理に付ける（もしくはすべてに付けない）よう統一するのが望ましいです。

## 参考リンク

- https://twitter.com/the_uhooi/status/1641597080753025024
- [Refactor privates by uhooi · Pull Request #155 · uhooi/Loki](https://github.com/uhooi/Loki/pull/155)
