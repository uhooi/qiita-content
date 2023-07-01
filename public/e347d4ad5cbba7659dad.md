---
title: IBActionの引数にあるsenderの使いみち(Swift)
tags:
  - iOS
  - Swift
  - IBAction
private: false
updated_at: '2020-06-17T18:56:30+09:00'
id: e347d4ad5cbba7659dad
organization_url_name: null
---
## はじめに

`sender` については以下の記事にいろいろ書いているので、読んでから本記事を読んでいただけるとわかりやすいと思います。
https://qiita.com/uhooi/items/e90d06e5d5681d72cbd0

そもそも `IBAction` の `sender` を使う機会はあまりないと思います。
少なくとも私はほとんど使わず、使いみちがパッとは出てきませんでした。

Twitterで教えていただいたので紹介します。

## `sender` の使いみち

さっそく `IBAction` の引数である `sender` の使いみちを紹介します。

### 1つの `IBAction` に複数のUIを紐付けて区別する

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">例えばボタンのタップを受け取るときに、1つのFunctionに対して複数のボタンアクションを設定したときに、senderをswitchするとかはやったことある</p>&mdash; moga🍳 (@_mogaming) <a href="https://twitter.com/_mogaming/status/1272811720432480257?ref_src=twsrc%5Etfw">June 16, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

例えば電卓アプリで、1〜9のボタンを1つの `IBAction` に紐付け、 `sender` でどのボタンがタップされたか判断します。
`IBAction` を1つのみ定義すれば済むので、コードがスッキリします。
（わかりやすいように `titleLabel?.text` で判断していますが、その是非はここでは考えません）

```swift:FooViewController.swift
final class FooViewController: UIViewController {
    @IBAction private func didTapNumberButton(_ sender: UIButton) {
        switch sender.titleLabel?.text {
        case "1":
            // 「1」をタップしたときの処理
        case "2":
            // 「2」をタップしたときの処理
        // ...
        }
    }
}
```

これは好みかもしれませんが、私はUIと `IBAction` が1:1で紐付いているほうが可読性が高いと感じるので、あまり使いません。

```swift:私は1つのIBActionに1つのUIのみ紐付ける
final class FooViewController: UIViewController {
    @IBAction private func didTapOneButton(_ sender: UIButton) {
        // 「1」をタップしたときの処理
    }

    @IBAction private func didTapTwoButton(_ sender: UIButton) {
        // 「2」をタップしたときの処理
    }

    // ...
}
```

コード量は増えますが、より具体的なボタン名をメソッド名に付けられるので、わかりやすいと感じます。

また、電卓の `didTapNumberButton(_:)` のように、どのUIが紐付いているか推測しやすければいいのですが、UIによってはどれが `IBAction` に紐付いているかコードから推測しにくくなります。
1:1と決めておけばコードからかんたんに推測できます。

### ポップオーバーの表示元のビューに指定する

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">受け取ったsenderを使ってpopOverのsourceView, sourceRectを指定する時に使いますねー。iPadで指定しないとクラッシュするアレです。</p>&mdash; Atom（アトム） (@FromAtom) <a href="https://twitter.com/FromAtom/status/1272815211255181312?ref_src=twsrc%5Etfw">June 16, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

iPadではポップオーバーを吹き出しで表示するため、吹き出しの表示元（三角がどこから表示されるか）を指定する必要があります。
そのときに `sender` を使うとかんたんに指定できます。

```swift:「共有」ボタンのタップ時にアクティビティを表示する
final class FooViewController: UIViewController {
    @IBAction private func didTapShareButton(_ sender: UIBarButtonItem) {
        let activityItems: [Any?] = ["Foo", nil, nil]
        let activityVC = UIActivityViewController(activityItems: activityItems as [Any], applicationActivities: nil)
        activityVC.popoverPresentationController?.barButtonItem = sender
        present(activityVC, animated: true)
    }
}
```

表示元（ここでは `UIBarButtonItem` ）を `IBOutlet` で繋げても実現できますが、アウトレット接続するとプロパティを自由に変更できるなどスコープが広がるため、 `sender` を使って処理するのが望ましいです。

### macOSでは大活躍

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">macOSだとIBOutletCollectionないのとNSMenuItemまわりのイベントハンドリングでsenderを使ったりとでsenderは大活躍ですね。そして型は基本書いてあった方が後々便利です。</p>&mdash; Kyome (@Kyomesuke) <a href="https://twitter.com/Kyomesuke/status/1273174335558168579?ref_src=twsrc%5Etfw">June 17, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

macOSだと `sender` が大活躍するとのことです。
私はmacOSのアプリを開発したことがないため、Kyomeさんのツイートとコメントの紹介に留めます。

https://qiita.com/uhooi/items/e347d4ad5cbba7659dad#comment-07088b8bb3ef0a5a82b5
