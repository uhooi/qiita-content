---
title: FragmentにおけるActivityとContextの使い分け
tags:
  - Android
  - Kotlin
private: false
updated_at: '2021-07-09T09:05:19+09:00'
id: 471b14cb74adadebc33c
organization_url_name: null
---
## はじめに

Fragmentにおける `Activity` と `Context` の意味や使い分けがわからなかったので、調べてみました。

## 結論

- `Context` で済む
  - nullを許容する → `this.context`
  - nullを許容しない → `requireContext()`
- `Activity` を渡す必要がある
  - nullを許容する → `this.activity`
  - nullを許容しない → `requireActivity()`

## 環境

- OS：macOS Big Sur 11.1
- Android Studio：4.1.2
- Kotlin：1.4.20
- Gradle：6.8
- Gradle plugin：4.1.2
- Timber：4.7.1

## 調査

### 各処理の戻り値

Fragmentで以下を実行しました。

```kotlin:MonsterListFragment.kt
Timber.d("${this.activity}")
Timber.d("${requireActivity()}")
Timber.d("${this.context}")
Timber.d("${requireContext()}")
```

```:Logcatの出力結果
2021-03-06 19:07:40.162 24680-24680/com.theuhooi.uhooipicbook D/MonsterListFragment: com.theuhooi.uhooipicbook.MainActivity@95e20bb
2021-03-06 19:07:40.162 24680-24680/com.theuhooi.uhooipicbook D/MonsterListFragment: com.theuhooi.uhooipicbook.MainActivity@95e20bb
2021-03-06 19:07:40.163 24680-24680/com.theuhooi.uhooipicbook D/MonsterListFragment: dagger.hilt.android.internal.managers.ViewComponentManager$FragmentContextWrapper@e47f888
2021-03-06 19:07:40.166 24680-24680/com.theuhooi.uhooipicbook D/MonsterListFragment: dagger.hilt.android.internal.managers.ViewComponentManager$FragmentContextWrapper@e47f888
```

`this.activity` と `requireActivity()` 、 `this.context` と `requireContext()` の出力結果は同じです。
つまり同じインスタンスを返しています。

### require○○() vs this.○○

`requireActivity()` と `requireContext()` の実装を見てみます。

```java:Fragment.java
/**
 * Return the {@link FragmentActivity} this fragment is currently associated with.
 *
 * @throws IllegalStateException if not currently associated with an activity or if associated
 * only with a context.
 * @see #getActivity()
 */
@NonNull
public final FragmentActivity requireActivity() {
    FragmentActivity activity = getActivity();
    if (activity == null) {
        throw new IllegalStateException("Fragment " + this + " not attached to an activity.");
    }
    return activity;
}
```

```java:Fragment.java
/**
 * Return the {@link Context} this fragment is currently associated with.
 *
 * @throws IllegalStateException if not currently associated with a context.
 * @see #getContext()
 */
@NonNull
public final Context requireContext() {
    Context context = getContext();
    if (context == null) {
        throw new IllegalStateException("Fragment " + this + " not attached to a context.");
    }
    return context;
}
```

見てわかる通り、 `activity (context)` が `null` だったら例外が発生し、 `null` でなかったらインスタンスを返す、というシンプルな実装になっています。

渡す先が `null` を許容する場合、 `null` のエラーハンドリングを任せられるため、 `require○○()` を呼ばずに `this.○○` で渡すべきだと考えます。
`require○○()` を呼ぶと例外が発生し、他でハンドリングできなくなるためです。

逆に `null` でない `context` や `activity` が必要なのに `require○○()` を使わないのは冗長なので避けるべきです。

```kotlin:MonsterListFragment.kt
class MonsterListFragment : Fragment() {
    override fun onOptionsItemSelected(item: MenuItem): Boolean {
        // ×冗長
        this.context?.let {
            MaterialAlertDialogBuilder(it)
        }

        // ○簡潔
        MaterialAlertDialogBuilder(requireContext())
    }
}
```

### Activity vs Context

出力結果と `require○○()` メソッドの実装より、Fragmentが持つ `activity` の型は `FragmentActivity` 、 `context` の型は `Context` ということがわかりました。

`FragmentActivity` の継承を追っていくと、 `Context` に辿り着きます。

```
Context
↑
ContextWrapper
↑
ContextThemeWrapper
↑
Activity
↑
androidx.core.app.ComponentActivity
↑
ComponentActivity
↑
FragmentActivity
```

できる限り小さく済ませたいので、 `Context` で済むところに `Activity` を渡す必要はないと考えます。

### Contextの種類

`FragmentActivity` の継承図より `Activity` が `Context` の子孫であることがわかりました。

同様に `Application` も `Context` の子孫であることがわかります。

```
Context
↑
ContextWrapper
↑
Application
```

Contextを区別するため、Applicationのことを「Application Context」、Activityのことを「Activity Context」と呼ぶことがあります。

Fragmentで取得できるContextは「Activity Context」であり、「Application Context」はActivityを介さないと取得できません（間違っていたらご指摘ください）。
`Activity` からは `this.applicationContext` でApplication Contextを取得できます。

Application Contextのほうが生存期間が長いので安全です。
しかし基本的にはActivityのライフサイクルに従ってActivity Contextを使うのがよさそうです。

未検証ですが、Fragmentの `this.activity` と `this.context` は同じインスタンス（今回は `MainActivity` ）を参照していると思われます。

つまり __基本的にはどちらを使っても変わりません__ 。
`this.activity` と `this.context` の中身が異なったり、片方のみ `null` になるケースがあるかどうかが気になります。

## おわりに

本記事の内容はあくまで私が出した結論です。
他にご意見やご指摘などありましたら、お気軽にコメントなどお願いします :pray: 

## 参考リンク

- [Androidの基礎知識 - mixi-inc/AndroidTraining](http://mixi-inc.github.io/AndroidTraining/introductions/1.04.basic-knowledge.html#context)
- [Androidで使われているApplication ContextとActivity Contextの使い所まとめ。 - GA technologies GROUP Tech Blog](https://tech.ga-tech.co.jp/entry/2020/06/android-contexts)
- https://twitter.com/the_uhooi/status/1368107727961026563?s=20
- https://twitter.com/yt8492/status/1368113367412473860?s=20
- https://twitter.com/the_uhooi/status/1368208335703449603?s=20
