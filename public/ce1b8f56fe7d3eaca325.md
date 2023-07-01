---
title: XIBファイルを使ってカスタムビューを実装する方法(Swift)
tags:
  - iOS
  - InterfaceBuilder
  - xib
  - Swift
  - Rswift
private: false
updated_at: '2020-06-17T22:39:09+09:00'
id: ce1b8f56fe7d3eaca325
organization_url_name: null
---
## はじめに

iOSアプリ開発において、UIを部品化して使い回したい場合、XIBファイルを使って実現することが多いと思います。
実装方法を忘れることが多いので、本記事でまとめることにします。

## 環境

本記事ではXIBファイルの読み込みにR.swiftを使っています。
`R.` から始まる処理はR.swiftの記述なので、適宜読み替えてください。

- OS：macOS Mojave 10.14.6
- Xcode：11.3.1 (11C504)
- Swift：5.1.3
- R.swift：5.2.2

## カスタムビューの実装

カスタムビューの実装方法を紹介します。

### カスタムビュークラスの作成

まず、 `UIView` を継承したクラスを作成します。
私は↓をテンプレートとしてコピペしています。

```swift:FooView.swift
import UIKit

@IBDesignable
final class FooView: UIView {

    // MARK: Stored Instance Properties

    // MARK: Computed Instance Properties

    // MARK: IBOutlets

    // MARK: Initializers

    override init(frame: CGRect) {
        super.init(frame: frame)

        configureView()
    }

    required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)

        configureView()
    }

    // MARK: IBActions

    // MARK: Other Internal Methods

    override func prepareForInterfaceBuilder() {
        super.prepareForInterfaceBuilder()

        configureView()
    }

    // MARK: Other Private Methods

    private func configureView() {
        loadNib()

        // TODO: 他にUIの装飾処理があれば実行する
    }

    private func loadNib() {
        guard let fooView = R.nib.fooView(owner: self) else {
            fatalError("Fail to load FooView from Nib.")
        }
        fooView.frame = self.bounds
        addSubview(fooView)
    }
}
```

XIBファイルを編集してビルドしても画面が真っ白になることがありますが、大体はXIBファイルを読み込んで `addSubview(_:)` していないのが原因です。
忘れやすいので、↑のように自分の中でテンプレート化しておくと便利です。

### XIBファイルの作成

このままでは `R.nib.fooView(owner: self)` でビルドエラーになるので、XIBファイルを作成します。

__プロジェクトツリーで右クリック > New File... > iOS - User Interface - View > [Next]ボタンをクリック__
<img width="740" alt="スクリーンショット_2020-06-17_19_40_52.jpg" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/138245/f1cefd1e-4c36-7209-5a36-d20ba9979565.jpeg">

__XIBファイル名を入力 > [Create]ボタンをクリック__
<img width="850" alt="スクリーンショット_2020-06-17_19_44_00.jpg" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/138245/5f03a0d3-5c8d-6d13-4981-49933b304562.jpeg">

XIBファイル名は任意ですが、先ほど作成したカスタムビュークラスと同じにするとわかりやすいです。

### XIBファイルのFile's Ownerを設定

XIBファイルのFile's Ownerに、先ほど作成したカスタムビュークラスを指定します。
<img width="694" alt="スクリーンショット_2020-06-17_19_49_34.jpg" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/138245/f79c6be8-2852-3864-2e30-d7bd9eed8b2e.jpeg">

忘れやすいので注意です。

### XIBファイルから不要なものを非表示

画面全体でなく一部分のみのカスタムビューを作る場合、ViewのSizeを `Freeform` 、Top BarとBottom Barを `None` にすると、不要なものが表示されなくなって実装しやすくなります。
<img width="265" alt="スクリーンショット 2020-06-17 21.55.32.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/138245/6c22ba45-02a8-e3f0-f73d-1937674e4958.png">

こちらの設定はあくまでIB上でのみ反映され、実際のアプリには無関係という認識です。
そのため、作りたいカスタムビューに応じて設定を変更するのが望ましいです。

### XIBファイルの編集

ここまで来たら好きなように `UILabel` や `UIButton` などのUIを配置し、カスタムビューを作成します。
必要に応じて `IBOutlet` や `IBAction` でカスタムビュークラスと接続してください。

## カスタムビューの使い方

カスタムビューの使い方は、通常のUIと同様です。
StoryboardやXIBファイルに `UIView` として配置し、Custom Classに指定するのみです。
<img width="264" alt="スクリーンショット_2020-06-17_19_56_45.jpg" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/138245/71207e26-4e18-2ab6-f456-01c5d832bc86.jpeg">

カスタムビュークラスに `@IBDesignable` を付けると、IB上で描画されてデザインが作りやすくなります。

## 注意

XIBファイルを使う際の注意点を紹介します。

### XIBファイルから読み込んだビューはカスタムビューに載せているだけ

例えば、カスタムビュークラスで背景色を以下のように変更しようとします。

```swift:FooView.swift
    private func configureView() {
        loadNib()

        backgroundColor = .blue
    }
```

これだけだと背景色は変わったように見えません。

理由は単純で、 `loadNib()` 内の処理を見ればわかります。
XIBファイルから読み込んだビューは、カスタムビュークラスのビューの上に追加しているので、いくら背景色を変えてもXIBのビューに隠されて見えないです。

```swift:FooView.swift
    private func loadNib() {
        // XIBファイルからビューを読み込む
        guard let fooView = R.nib.fooView(owner: self) else {
            fatalError("Fail to load FooView from Nib.")
        }

        // XIBから読み込んだビューのサイズを、カスタムビュークラスのビューに合わせる
        fooView.frame = self.bounds

        // XIBから読み込んだビューを、カスタムビュークラスのサブビューに追加する
        addSubview(fooView)
    }
```

図で見るともっとわかりやすいと思います。
先ほどのコードで変えた背景色は、図の斜線部分なので、XIBのビューに隠されていることがわかります。
![1592378872334.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/138245/46197893-72c9-c209-f142-942b25ec7590.jpeg)

解決法は大きく2つあります。

- XIBファイル上でビューの背景色を透過色にする  
最もかんたんな方法だが、透過色は計算負荷が高いのでパフォーマンスが落ちる可能性がある
- XIBファイルから読み込んだビューを `contentView` のような名前でプロパティとして保持し、こちらの背景色を変える  
プロパティとして持たせると、何でも操作できて無駄にスコープが広がる

パフォーマンスが落ちることはめったにないので、私は前者を選ぶことが多いです。

## おわりに

これでXIBファイルを使ってカスタムビューを作れるようになりました！
UIを適切な粒度で部品化し、効率よく開発しましょう:relaxed:

もし本記事の内容に誤りや過不足があれば、ご指摘いただけると嬉しいです。
例えばよく見る `awakeFromNib()` というメソッドを使わずに実現しているので、問題ないのか気になります＞＜

## 参考リンク

- https://twitter.com/the_uhooi/status/1273147820510515200?s=20
