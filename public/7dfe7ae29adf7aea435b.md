---
title: 文字列を左寄せにする方法の考察(SwiftUI)
tags:
  - iOS
  - Swift
  - SwiftUI
private: false
updated_at: '2022-07-13T12:02:05+09:00'
id: 7dfe7ae29adf7aea435b
organization_url_name: dena_coltd
slide: false
---
## はじめに

SwiftUIで `Image` と `Text` を左寄せにしつつ、文字列を幅いっぱいに表示したかったのですが、うまくいきませんでした。

https://twitter.com/the_uhooi/status/1546519865544306688

実際のコードです。

```swift:MonsterCellView.swift
import SwiftUI

struct MonsterCellView: View {
    var iconName: String
    var name: String
    
    var body: some View {
        HStack(spacing: 32) {
            Image(iconName)
                .resizable()
                .scaledToFit()
                .frame(width: 68, height: 68)
            Text(name)
                .font(.title)
            Spacer()
        }
        .padding(16)
        .frame(maxWidth: .infinity)
        .background(
            Color(.systemBackground)
                .cornerRadius(3)
                .elevate(elevation: 1)
        )
    }
}

private extension View {
    func elevate(elevation: CGFloat) -> some View {
        self.shadow(
            color: .init(white: 0, opacity: 0.24),
            radius: elevation,
            x: 0,
            y: elevation
        )
    }
}

struct MonsterCellView_Previews: PreviewProvider {
    static var previews: some View {
        VStack(spacing: 28) {
            MonsterCellView(iconName: "uhooi", name: "uhooi")
            MonsterCellView(iconName: "uhooi", name: "とてつもなく長い名前のモンスター")
        }
        .padding(.horizontal, 24)
    }
}
```

プレビューです。
24行目で `.padding(_:)` に `16` を指定していますが、明らかに `Text` の右側は `16` ピクセル以上空いています。
![before.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/138245/7edea48c-4315-3451-c1ab-018c2a015d34.png)

この原因と解決策を考察したので紹介します。

## 環境

- OS：macOS Monterey 12.4
- Xcode：13.4.1 (13F100)
- Swift：5.6.1

## 原因

「 `Text` と `Spacer` の間にも `HStack` の `spacing` が適用されるため」です。

言葉だとわかりづらいので図解します。
![MonsterCellView_before.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/138245/6b49d4b2-63f0-6c72-aedb-325646d616ec.png)

これで少しはわかりやすくなったはずです。

`Spacer` 自体のサイズは（おそらく） `0` ですが、 `HStack` の `spacing: 32` が `Image` と `Text` の間のみでなく、 `Text` と `Spacer` の間にも適用されてしまいます。

```swift:MonsterCellView.swift
HStack(spacing: 32) {
    Image(iconName)
        .resizable()
        .scaledToFit()
        .frame(width: 68, height: 68)
    // ここに余白が `32` 入る
    Text(name)
        .font(.title)
    // !!!: ここにも余白が `32` 入る
    Spacer()
}
```

いわゆる「 `Spacer` の罠」です。
`Spacer` はViewを詰めるのに便利ですが、仕様を理解していないとつまづきます。

## 解決策

解決策をTwitterでいろいろ教えていただいたので紹介します。

### 1. HStackのspacingを0にして、幅固定のSpacerを入れる

おそらく最もシンプルな解決策です。

`HStack` の `spacing` を `32` から `0` にして、 `Image` と `Text` の間に幅 `32` の `Spacer` を入れます。

```diff_swift:MonsterCellView.swift
import SwiftUI

struct MonsterCellView: View {
    var iconName: String
    var name: String
    
    var body: some View {
-       HStack(spacing: 32) {
+       HStack(spacing: 0) {
            Image(iconName)
                .resizable()
                .scaledToFit()
                .frame(width: 68, height: 68)
+           Spacer().frame(width: 32)
            Text(name)
                .font(.title)
            Spacer()
        }
        .padding(16)
        .frame(maxWidth: .infinity)
        .background(
            Color(.systemBackground)
                .cornerRadius(3)
                .elevate(elevation: 1)
        )
    }
}

// ...
```

先ほどより `Text` の右側の余白が小さくなったことがわかります。
![after_幅固定spacer.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/138245/6a35cdb9-afb6-2d79-1c86-133658caffe8.png)

ただ `HStack` の `spacing` は本来 `0` にしたくないので、可読性は落ちてしまいます。

あと `Spacer` に `.frame(width:)` で幅を指定しても、必ずそのサイズに固定されるわけではないかもしれません。
`Spacer` の仕様に詳しい方がいたら教えていただきたいです :pray:

### 2. HStackのspacingを0にして、Imageの右側に余白を付ける

解決策1とほぼ同じです。
幅固定の `Spacer` を入れる代わりに、 `Image` の右側に `.padding(_:)` を付けます。

```diff_swift:MonsterCellView.swift
import SwiftUI

struct MonsterCellView: View {
    var iconName: String
    var name: String
    
    var body: some View {
-       HStack(spacing: 32) {
+       HStack(spacing: 0) {
            Image(iconName)
                .resizable()
                .scaledToFit()
                .frame(width: 68, height: 68)
+               .padding(.trailing, 32)
            Text(name)
                .font(.title)
            Spacer()
        }
        .padding(16)
        .frame(maxWidth: .infinity)
        .background(
            Color(.systemBackground)
                .cornerRadius(3)
                .elevate(elevation: 1)
        )
    }
}

// ...
```

パッと見の結果は同じです。
![after_padding.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/138245/bf3fa555-9546-d4f4-088a-5bed33b8f402.png)

しかし `Image` の右側に余白が付くのが異なります。
![after_padding_2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/138245/319dd9fd-2044-378c-89ba-09bf2abe4f43.png)

好みの問題かもしれませんが、「View間の余白」というより「Image右の余白」という意図が強くなってしまいます。

### 3. HStackを入れ子にする

`spacing` が `0` と `32` の `HStack` を用意して、入れ子にします。

```diff_swift:MonsterCellView.swift
import SwiftUI

struct MonsterCellView: View {
    var iconName: String
    var name: String
    
    var body: some View {
-       HStack(spacing: 32) {
-           Image(iconName)
-               .resizable()
-               .scaledToFit()
-               .frame(width: 68, height: 68)
-           Text(name)
-               .font(.title)
+       HStack(spacing: 0) {
+           HStack(spacing: 32) {
+               Image(iconName)
+                   .resizable()
+                   .scaledToFit()
+                   .frame(width: 68, height: 68)
+               Text(name)
+                   .font(.title)
            }
            Spacer()
        }
        .padding(16)
        .frame(maxWidth: .infinity)
        .background(
            Color(.systemBackground)
                .cornerRadius(3)
                .elevate(elevation: 1)
        )
    }
}

// ...
```

解決策1と同様の結果になりました。
![after_HStack入れ子.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/138245/65b275b5-54c4-9d4e-805f-b37efea8f6a8.png)

解決策1や2より意図が明確になり、可読性は高いと思います。
しかし `HStack` が入れ子になるのが気になります。

### 4. Spacerを消してframeにalignmentを付ける

個人的に最も好みな解決策です。

`Spacer()` を削除し、 `.frame()` に `alignment: .leading` を付けます。

```diff_swift:MonsterCellView.swift
import SwiftUI

struct MonsterCellView: View {
    var iconName: String
    var name: String
    
    var body: some View {
        HStack(spacing: 32) {
            Image(iconName)
                .resizable()
                .scaledToFit()
                .frame(width: 68, height: 68)
            Text(name)
                .font(.title)
-           Spacer()
        }
        .padding(16)
-       .frame(maxWidth: .infinity)
+       .frame(maxWidth: .infinity, alignment: .leading)
        .background(
            Color(.systemBackground)
                .cornerRadius(3)
                .elevate(elevation: 1)
        )
    }
}

// ...
```

解決策1や3と同様の結果になりました。
![after_frameにalignment.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/138245/45d2f196-718e-0f9e-39b0-24fda674a740.png)

`.frame()` に `alignment` を指定すると、内部のViewを寄せられることを初めて知りました。
特にデメリットがなく、 `.leading` を指定して左寄せなことを明示的にしているので、スマートな解決策だと思います。

ちなみに `.frame()` を `HStack` でなく `Text` に付けてもほぼ同じ結果が得られます。


```diff_swift:MonsterCellView.swift
import SwiftUI

struct MonsterCellView: View {
    var iconName: String
    var name: String
    
    var body: some View {
        HStack(spacing: 32) {
            Image(iconName)
                .resizable()
                .scaledToFit()
                .frame(width: 68, height: 68)
            Text(name) 
               .font(.title)
+              .frame(maxWidth: .infinity, alignment: .leading)
        }
        .padding(16)
-       .frame(maxWidth: .infinity, alignment: .leading)
        .background(
            Color(.systemBackground)
                .cornerRadius(3)
                .elevate(elevation: 1)
        )
    }
}

// ...
```

どちらでもよさそうですが、 `.frame()` を `Text` に付けるほうが「文字列が幅いっぱいに膨らむ」ことがわかるのでいいかもしれません。
![after_frameにalignment_2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/138245/ae5e010f-2c6c-f87d-e6f2-76bb413da81d.png)

私は `Text` に付けるほうが好みです。

### 5. LabelとLabelStyleを使ってスタイルを外出しする

@noppefoxwolf さんに本記事のアンサー記事を書いてもらい、そこで紹介されていた方法です。

https://qiita.com/noppefoxwolf/items/45343a05ab4f21a6e059

`Image` と `Text` に関連性がある場合、 `Label` としてまとめることで可読性を上げられます。
`LabelStyle` を使ってスタイルを外出しすることで、HTMLとCSSのように構造と装飾を分離することもできます。

解決策1〜4のどれかと組み合わせる必要があるので、私は4の後者と組み合わせました。
4の後者との差分を載せます。

```diff_swift:MonsterCellView.swift
import SwiftUI

struct MonsterCellView: View {
    var iconName: String
    var name: String
    
    var body: some View {
-       HStack(spacing: 32) {
-           Image(iconName)
-               .resizable()
-               .scaledToFit()
-               .frame(width: 68, height: 68)
-           Text(name) 
-              .font(.title)
-              .frame(maxWidth: .infinity, alignment: .leading)
-       }
+       Label {
+           Text(name)
+       } icon: {
+           Image(iconName)
+               .resizable()
+       .labelStyle(MonsterCellLabelStyle())
        .padding(16)
        .background(
            Color(.systemBackground)
                .cornerRadius(3)
                .elevate(elevation: 1)
        )
    }
}

+ private struct MonsterCellLabelStyle: LabelStyle {
+     func makeBody(configuration: Configuration) -> some View {
+         HStack(spacing: 32) {
+             configuration.icon
+                 .scaledToFit()
+                 .frame(width: 68, height: 68)
+             configuration.title
+                 .font(.title)
+                 .frame(maxWidth: .infinity, alignment: .leading)
+         }
+     }
+ }
+ 
// ...
```

解決策4の後者と同様の結果になりました。
![after_label.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/138245/96e6c632-7e6b-d5e9-4fe3-4b9d5865bc05.png)

`Image` と `Text` に関連性があることがひと目でわかり、さらに可読性が上がったと思います。

LabelStyleの詳細は以下の記事をご参照ください。

https://qiita.com/uhooi/items/6aa53a3f07b867a8977a

## おまけ

おまけです。

### 1. SpacerにminLengthを指定できる

`Spacer` には `minLength` を指定できます。

https://twitter.com/mtj_j/status/1546672782410657792

公式ドキュメントにも書いてある通り、 `minLength` のデフォルト値は `nil` で、 `nil` の場合はシステムのデフォルト値が入ります。（それがいくつなのかはわかりませんｗ）

なので `Spacer` の長さを確実に `0` にしたいときは、 `Spacer(minLength: 0)` と書くべきです。

### 2. Spacerは使わないほうがいい？

`Spacer()` はできる限り使わないほうが実装ミスが減り、可読性も高くなると思いました。

使いどころとしては、単に全Viewを左（右）寄せにするときではなく、「画像は左寄せ、文字列は右寄せ」など、他の方法だと複雑になるパターンのときに使うとよさそうです。

```swift:MonsterCellView.swift
HStack {
    Image(iconName)
        .resizable()
        .scaledToFit()
        .frame(width: 68, height: 68)
    Spacer() // !!!: このような場合に `Spacer()` を使うとよさそう
    Text(name)
        .font(.title)
}
```


`Spacer` の使いどころに詳しい方がいたら教えていただきたいです :pray:

2022/07/13, 追記
Twitterで「Spacerはその名の通り、何かの間のスペースを埋めるために使う」と教えていただきました。

https://twitter.com/macneko_ayu/status/1546832967011487744

なので私が紹介したようなパターンのときに使うので合っていそうです。

## おわりに

文字列を左寄せにする方法を考察しました。
みなさんがオススメの解決策はどれでしょうか？
また他に解決策はあるでしょうか？
ぜひコメントなどで教えてください。

今回の考察を通して、比較的シンプルなViewを作るのにも様々なパターンがあることを知り、SwiftUIは可読性を高めるのが難しいと実感しました。
読み手が意図を汲みやすい実装ができるよう、SwiftUI力を高めていきたいです。

## 参考リンク

- [SwiftUIでの文字寄せとか - MOL](https://t32k.me/mol/log/text-align-swiftui/)
- [minLength | Apple Developer Documentation](https://developer.apple.com/documentation/swiftui/spacer/minlength)
