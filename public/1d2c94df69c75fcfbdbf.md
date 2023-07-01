---
title: DispatchQueue→Task置き換えまとめ(Swift)
tags:
  - task
  - Swift
  - concurrency
  - DispatchQueue
  - AsyncAwait
private: false
updated_at: '2022-09-30T11:54:18+09:00'
id: 1d2c94df69c75fcfbdbf
organization_url_name: dena_coltd
---
## はじめに

Objective-C時代から私たちを支えてくれたDispatchQueueですが、Swift Concurrency（async/await）の登場によりその役目を終えようとしています。

本記事では `DispatchQueue` を使った様々なユースケースを `Task` に置き換える方法を紹介します。

## 注意

- 基本的にiOS 13.0+では `DispatchQueue` を使う必要がない
  - `DispatchQueue` でしか実現できない処理があったら教えてほしい
    - `DispatchQueue` はキューに名前を付けられるので、デバッグしやすいというメリットはある
- Before/Afterで処理が完全に同じになるかまでは確認していない

## 環境

- OS：macOS Monterey 12.5.1
- Xcode：14.0 (14A309)
- Swift：5.7

## 置き換え一覧

`DispatchQueue` → `Task` 置き換えの一覧です。

|DispatchQueue|Task|
|:--|:--|
|`DispatchQueue.main.sync(excute:)`|`MainActor.run(resultType:body:)`|
|`DispatchQueue.main.async(excute:)`|`Task.detached { @MainActor in ... }`|
|`DispatchQueue.async(execute:)` （シリアルキュー）|`actor`|
|`DispatchGroup` ・ `DispatchQueue.async(group:execute:)`|`async let`|
|`DispatchGroup.enter()` ・ `DispatchGroup.leave()`|`withCheckedThrowingContinuation(function:_:)` または `withCheckedContinuation(function:_:)`|
|`DispatchQueue.main.asyncAfter(deadline:execute:)`|iOS 13.0+: `Task.sleep(nanoseconds:)`<br>iOS 16.0+: `Task.sleep(for:)`|

置き換え方法は他にもあると思います。ここでは書きやすい方法を紹介しています。

## ユースケース

置き換えをユースケースごとに紹介します。

### 特定の処理が完了するのを待つ

特定の処理が完了するのを待つユースケースです。

今までは `DispatchQueue.main.sync(excute:)` を使っていましたが、これからは `MainActor.run(resultType:body:)` を使います。

以下のサンプルでは、BeforeもAfterも `1` → `2` の順番で出力されます。

```swift
// Before
DispatchQueue.main.sync {
  print("1")
}
print("2")

// After
Task {
  await MainActor.run {
    print("1")
  }
  print("2")
}
```

もっとも `DispatchQueue.main.sync(excute:)` はメインスレッドで実行するとデッドロックのためにクラッシュしたりと、基本的には使わないはずなので、このユースケースの置き換えはあまりないかもしれません。

### 特定の処理を非同期で実行する

特定の処理を非同期で実行するユースケースです。

今までは `DispatchQueue.main.async(excute:)` を使っていましたが、これからは　`Task.detached { @MainActor in ... }` を使います。

この置き換えはSwiftのプロポーザルに書いてありました。

https://github.com/apple/swift-evolution/blob/f64c699afeac2ed15c4a6c36a200a56cc9fb1ef5/proposals/0316-global-actors.md#closures

以下のサンプルでは、BeforeもAfterも `2` → `1` の順番で出力されます。

```swift
// Before
DispatchQueue.main.async {
  print("1")
}
print("2")

// After
Task.detached { @MainActor in
  print("1")
}
print("2")
```

タスクを切り離す必要がないときは、 `Task { @MainActor in ... }` と書けます。
クロージャ内の処理をメインスレッドで実行したいだけの場合、 `Task.detached` を使わないほうが多いと思います。

```diff_swift
// このように書くことのほうが多い
- Task.detached { @MainActor in
+ Task { @MainActor in
  print("1")
}
print("2")
```

### データ競合を防ぐ

データ競合を防ぐユースケースです。

今までは `DispatchQueue` でシリアルキューを作って実現していました。
これからは `actor` を使って実現します。

```swift
// Before
final class Counter {
  private let queue = DispatchQueue(label: "com.example.myqueue") // シリアルキュー

  private var count = 0

  func increment() {
    queue.async { [self] in
      count += 1
    }
  }

  func decrement() {
    queue.async { [self] in
      count -= 1
    }
  }
}

// After
actor Counter {
  private var count = 0

  func increment() {
    count += 1
  }

  func decrement() {
    count -= 1
  }
}
```

かなりスッキリしました。

詳細は以下の記事をご参照ください。

https://zenn.dev/koher/articles/swift-concurrency-cheatsheet#💼-case-14-(actor)%3A-共有された状態の変更

### 複数の処理を非同期で実行し、完了するのを待つ

複数の処理を非同期で実行し、完了するのを待つユースケースです。

今までは `DispatchQueue.async(group:execute:)` を使っていましたが、これからは `async let` を使います。
非同期メソッドの戻り値がなくても、 `async let` で明示的に `Void` を指定することで変数として代入できます。

```swift
// Before
let group = DispatchGroup()
let queue1 = DispatchQueue(label: "com.example.myqueue1")
let queue2 = DispatchQueue(label: "com.example.myqueue2")
let queue3 = DispatchQueue(label: "com.example.myqueue3")

queue1.async(group: group) {
  print("1")
}
queue2.async(group: group) {
  print("2")
}
queue3.async(group: group) {
  print("3")
}

group.notify(queue: .main) {
  print("4")
}

// After
async let queue1: Void = printAsync("1")
async let queue2: Void = printAsync("2")
async let queue3: Void = printAsync("3")

_ = await (queue1, queue2, queue3)

print("4")

private func printAsync(_ text: String) async {
  print(text)
}
```

### クロージャの実行を待つ

クロージャの実行を待つユースケースです。

今まではクロージャへ入る前に `DispatchGroup.enter()` を実行し、クロージャを抜けるときに `DispatchGroup.leave()` を実行して実現していました。
これからは `withCheckedThrowingContinuation(function:_:)` （または `withCheckedContinuation(function:_:)` ）を使い、非同期メソッドに変換して `await` することで実現します。

以下のサンプルでは、BeforeもAfterも `1` → `2` の順番で出力されます。

```swift
let request = URLRequest(url: URL(string: "https://example.com")!)

// Before
let group = DispatchGroup()

group.enter()
URLSession.shared.dataTask(with: request) { data, response, error in
  if let error { return }
  guard let data, let response else { return }

  print("1")

  group.leave()
}.resume()

group.notify(queue: .global()) {
  print("2")
}

// After
_ = try! await URLSession.shared.data(from: request)

print("2")

private extension URLSession {
  func data(from request: URLRequest) async throws -> (Data, URLResponse) {
    struct BadServerResponseError: Error {}
    return try await withCheckedThrowingContinuation { continuation in
      self.dataTask(with: request) { data, response, error in
        if let error {
          return continuation.resume(throwing: error)
        }
        guard let data, let response else {
          return continuation.resume(throwing: BadServerResponseError())
        }

        print("1")

        continuation.resume(returning: (data, response))
      }.resume()
    }
  }
}
```

スローしない場合は `withCheckedThrowingContinuation(function:_:)` の代わりに `withCheckedContinuation(function:_:)` を使います。

```diff_swift
- func data(from request: URLRequest) async throws -> (Data, URLResponse) {
+ func data(from request: URLRequest) async -> (Data?, URLResponse?) {
-   struct BadServerResponseError: Error {}
-   return try await withCheckedThrowingContinuation { continuation in
+   await withCheckedContinuation { continuation in
      self.dataTask(with: request) { data, response, error in
-       if let error {
+       if error != nil {
-         return continuation.resume(throwing: error)
+         return continuation.resume(returning: (nil, nil))
        }
-       guard let data, let response else {
-         return continuation.resume(throwing: BadServerResponseError())
-       }

        print("1")

        continuation.resume(returning: (data, response))
      }.resume()
    }
  }
}
```

戻り値がない場合は `continuation.resume(returning:)` の代わりに `continuation.resume()` を使います。

```diff_swift
- func data(from request: URLRequest) async throws -> (Data, URLResponse) {
+ func data(from request: URLRequest) async throws {
    // ...
    return try await withCheckedThrowingContinuation { continuation in
      self.dataTask(with: request) { data, response, error in
        // ...
-       guard let data, let response else {
+       guard data != nil, response != nil else {
          return continuation.resume(throwing: BadServerResponseError())
        }
        // ...
-       continuation.resume(returning: (data, response))
+       continuation.resume()
      }.resume()
    }
  }
```

コンプリションハンドラを非同期メソッドへ変換するのは最終手段であり、もし標準で非同期メソッドが用意されているならそちらを使うべきです。
例えば `data(from:delegate:)` はiOS 15.0+から使えるので、iOS 14以下で非同期メソッドにしたい場合のみ変換するのが望ましいです。

https://developer.apple.com/documentation/foundation/urlsession/3767353-data

### 特定の処理を遅延実行する

特定の処理を遅延実行するユースケースです。

今までは `DispatchQueue.main.asyncAfter(deadline:execute:)` を使い、クロージャとして実行していました。
これからは `await Task.sleep(nanoseconds:)` を使い、スリープさせることで遅延実行を表現します。

ただ指定する単位がナノ秒と、少し使いづらいです。
Xcode 14.1（Swift 5.7.1）かつiOS 16.0+では `Task.sleep(for:)` を使って単位を指定できるので、こちらを使うのが望ましいです。

```swift
// Before
DispatchQueue.main.asyncAfter(deadline: .now() + .milliseconds(100)) {
  // ...
}

// After
Task {
  // iOS 13.0+
  try await Task.sleep(nanoseconds: 100_000_000)

  // iOS 16.0+
  try await Task.sleep(for: .nanoseconds(100_000_000))
  try await Task.sleep(for: .microseconds(100_000))
  try await Task.sleep(for: .milliseconds(100))
  try await Task.sleep(for: .seconds(0.1))
  // ...
}
```

## 置き換えないケース

例外として、置き換えずに `DispatchQueue` を使い続けるケースを紹介します。

### Combineでスケジューラを指定して値を受け取る

Combineでスケジューラを指定して値を受け取るケースです。

`Publisher.sink(receiveValue:)` 内でゴニョゴニョしても同じことが実現できます。
しかし `Publisher.receive(on:options:)` で `DispatchQueue.main` を指定するほうがスマートなので、この場合は `DispatchQueue` を使ってもいいと考えます。

```swift
// ✅Before
pub.receive(on: DispatchQueue.main).sink { [weak self] _ in
  // ...
}

// ⚠After
pub.sink { [weak self] _ in
  Task.detached { @MainActor in
    // ...
  }
}
```

## おわりに

これで `DispatchQueue` を `Task` に置き換えられ、スマートになりました :relaxed:

`DispatchQueue` を使ったユースケースは他にもあるので、コメントを頂いたりしたら追記したいと思います。

## 参考リンク

- [sync(execute:) | Apple Developer Documentation](https://developer.apple.com/documentation/dispatch/dispatchqueue/1452870-sync)
- [async(execute:) | Apple Developer Documentation](https://developer.apple.com/documentation/dispatch/dispatchqueue/2016103-async)
- [run(resultType:body:) | Apple Developer Documentation](https://developer.apple.com/documentation/swift/mainactor/run(resulttype:body:))
- [Swift Concurrency まとめ（正式版対応済）](https://zenn.dev/akkyie/articles/swift-concurrency#mainactor)
- [SwiftでConcurrency Programming - Qiita](https://qiita.com/shoheiyokoyama/items/433ff8e6612d8d368875)
- [DispatchGroup | Apple Developer Documentation](https://developer.apple.com/documentation/dispatch/dispatchgroup)
- [init(label:qos:attributes:autoreleaseFrequency:target:) | Apple Developer Documentation](https://developer.apple.com/documentation/dispatch/dispatchqueue/2300059-init)
- [async(group:execute:) | Apple Developer Documentation](https://developer.apple.com/documentation/dispatch/dispatchqueue/2300095-async)
- [notify(queue:work:) | Apple Developer Documentation](https://developer.apple.com/documentation/dispatch/dispatchgroup/2016084-notify)
- [dataTask(with:) | Apple Developer Documentation](https://developer.apple.com/documentation/foundation/urlsession/1410592-datatask)
- [withCheckedThrowingContinuation(function:_:) | Apple Developer Documentation](https://developer.apple.com/documentation/swift/withcheckedthrowingcontinuation(function:_:))
- [Swift Concurrency チートシート](https://zenn.dev/koher/articles/swift-concurrency-cheatsheet)
- [asyncAfter(deadline:execute:) | Apple Developer Documentation](https://developer.apple.com/documentation/dispatch/dispatchqueue/2300020-asyncafter)
- [sleep(nanoseconds:) | Apple Developer Documentation](https://developer.apple.com/documentation/swift/task/sleep(nanoseconds:))
- [sleep(for:) | Apple Developer Documentation](https://developer.apple.com/documentation/swift/task/sleep(for:))
- [Duration | Apple Developer Documentation](https://developer.apple.com/documentation/swift/duration)
- https://twitter.com/the_uhooi/status/1531195527001350144
- [receive(on:options:) | Apple Developer Documentation](https://developer.apple.com/documentation/combine/publisher/receive(on:options:))
