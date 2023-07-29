---
title: FeliCaの読み取りに対応しているiPhone一覧
tags:
  - iPhone
  - NFC
  - FeliCa
  - CoreNFC
private: false
updated_at: '2020-11-20T13:35:34+09:00'
id: 2ebf4cee7d7bdc2cdf10
organization_url_name: null
slide: false
---
## 注意

私が非公式にまとめたので、正式な情報はAppleによる一次ソースをご確認ください。

## FeliCaの読み取りに対応しているiPhone一覧

Core NFC自体はiOS 11.0から使えますが、FeliCaの読み取りに対応したのはiOS 13.0からです。

そのため「iOS 13.0以上をサポートしている、かつNFCが搭載されているiPhone一覧」とほぼ同義です。
（「ほぼ」と言っている理由は追記をご参照ください）

```
iPhone 12 mini
iPhone 12
iPhone 12 Pro
iPhone 12 Pro Max
iPhone 11
iPhone 11 Pro
iPhone 11 Pro Max
iPhone XR
iPhone XS
iPhone XS Max
iPhone X
iPhone SE（第2世代）
iPhone 8
iPhone 8 Plus
iPhone 7
iPhone 7 Plus
```

## 追記： NFC-A/Bのみ搭載されている端末もある

[@treastrain](https://twitter.com/treastrain) さんからTwitterで以下を教えていただきました。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">NFC は iPhone 6s から搭載されるようになりました。しかし、6s では NFC-A/B のみで FeliCa は使えません。<br><br>iPhone SE (1st) でも NFC-A/B のみ搭載されています。<br><br>FeliCa まで対応したのは iPhone 7 以降です。また、7 以降から Core NFC によるサードパーティー利用が可能となりました。</p>&mdash; treastrain / Tanaka.R (@treastrain) <a href="https://twitter.com/treastrain/status/1329638383702851586?ref_src=twsrc%5Etfw">November 20, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">Apple 公式の1次ソースとして使えるのはこのページかと思います。<br><br>「nfc」にチェックが入っている端末で Core NFC によるサードパーティー利用ができます。<br><br>iPhone 6s 系、iPhone SE (1st) は Apple Pay 限定のためチェックがありません。<br><br>必要なデバイス機能 - サポート <a href="https://t.co/D4NWrf5SMR">https://t.co/D4NWrf5SMR</a></p>&mdash; treastrain / Tanaka.R (@treastrain) <a href="https://twitter.com/treastrain/status/1329638804571910145?ref_src=twsrc%5Etfw">November 20, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

以下のiPhoneはNFC-A/Bのみ搭載されており、かつApple Payの利用に限定されているため、サードパーティーによる読み取りを行うことはできません。

```
iPhone 6s
iPhone 6s Plus
iPhone SE（第1世代）
```

## 参考リンク

- [サポートされるiPhoneのモデル - Apple サポート](https://support.apple.com/ja-jp/guide/iphone/iphe3fa5df43/13.0/ios/13.0)
- [サポートされるiPhoneのモデル - Apple サポート](https://support.apple.com/ja-jp/guide/iphone/iphe3fa5df43/14.0/ios/14.0)
- [必要なデバイス機能 - サポート - Apple Developer](https://developer.apple.com/jp/support/required-device-capabilities/)
- [Core NFC | Apple Developer Documentation](https://developer.apple.com/documentation/corenfc)
- [NFCFeliCaTag | Apple Developer Documentation](https://developer.apple.com/documentation/corenfc/nfcfelicatag)
