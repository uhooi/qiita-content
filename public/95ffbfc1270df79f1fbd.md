---
title: NeovimのHEADをビルドしてインストールする方法(Mac)
tags:
  - Vim
  - Mac
  - neovim
private: false
updated_at: '2023-07-29T23:48:15+09:00'
id: 95ffbfc1270df79f1fbd
organization_url_name: dena_coltd
slide: false
---
:::note info
本記事は [Vim 駅伝](https://vim-jp.org/ekiden/) の6/12の記事です。
前回は6/9で [yukimemi](https://zenn.dev/yukimemi) さんの [TypeScript で Vim / Neovim の設定を書く！](https://zenn.dev/yukimemi/articles/2023-06-09-dvpm) です。
次回は6/14で [kawarimidoll](https://zenn.dev/kawarimidoll) さんの [Vim scriptで1回しか使えないコマンド・関数を定義する](https://zenn.dev/vim_jp/articles/22856ed2627056) です。
:::

## はじめに

Homebrewで普通に `brew install nvim` を実行してNeovimをインストールすると、Neovimの安定版がインストールされます。

しかしそれだと `vim.iter` [^vim.iter] や `vim.system` [^vim.system] など `master` ブランチにマージされたPRの変更をすぐに使えませんし、 [dropbar.nvim](https://github.com/Bekaboo/dropbar.nvim) など一部のエクスペリメンタルなプラグインを使えません。
すでに `vim.iter` を使いこなしている某jpの高度な会話に着いていくこともできません。

[^vim.iter]: https://github.com/neovim/neovim/pull/23029
[^vim.system]: https://github.com/neovim/neovim/pull/23827

`master` ブランチのHEADをビルドして、文字通り最先端のNeovimをインストールする方法を紹介します。

## 環境

- OS：macOS Ventura 13.3.1
- Homebrew：4.0.21
- Neovim：v0.10.0-dev-509+g6e1fa16dd

## 初回セットアップ

最初のみ必要なセットアップを紹介します。

以下のWikiの通りに行っているので、英語が読める方はWikiを見るほうがいいと思います。

https://github.com/neovim/neovim/wiki/Building-Neovim

### Xcode Command Line Toolsのインストール

Xcode Command Line Toolsをインストールします。

```shell-session
$ xcode-select --install
```

### Homebrewのインストール

以下のページを参考にHomebrewをインストールします。

https://brew.sh/

### 依存関係のインストール

Neovimのビルドに必要な依存関係をインストールします。

```shell-session
$ brew install ninja cmake gettext curl
```

### Neovimのパスを通す（任意）

必要に応じてNeovimのパスを通します。
私は `$HOME/.local/nvim` にNeovimをインストールしています。

```diff_shell:.bash_profile
+ # Neovim
+ export NEOVIM_HOME=$HOME/.local/nvim
+ if [ -d "${NEOVIM_HOME}" ]; then
+   export PATH="${NEOVIM_HOME}/bin:$PATH"
+ fi
```

### 既存のNeovimのアンインストール（任意）

必要に応じてすでにインストールされているNeovimをアンインストールします。
私はHomebrewでインストールしていたので、以下のコマンドを実行してアンインストールしました。

```shell-session
$ brew uninstall nvim
```

### リポジトリのクローン

Neovimのリポジトリをクローンします。

```shell-session
$ mkdir neovim
$ cd neovim
$ git clone https://github.com/neovim/neovim
$ cd neovim
```

## Neovimのビルドとインストール

初回セットアップが完了したら、Neovimをビルドしてインストールします。

### リモートリポジトリから最新を取得

リモートリポジトリから最新の変更を取得します。

```shell-session
$ git fetch --prune origin
$ git rebase origin/master
```

### Neovimのビルド

Neovimをビルドします。

`CMAKE_BUILD_TYPE` に `Release` を指定するとリリースビルドします。
リリースビルドするとコンパイラが最適化し、最高のパフォーマンスとなります。

`CMAKE_INSTALL_PREFIX` でインストールするパスを指定できます。
`sudo` が不要な場所にインストールするのがオススメです。

```shell-session
$ make CMAKE_BUILD_TYPE=Release CMAKE_INSTALL_PREFIX=$HOME/.local/nvim
```

### Neovimのインストール

Neovimをインストールします。

```shell-session
$ make install
```

### バージョンの確認

Neovimのバージョンを確認します。
バージョンのサフィックスに `-dev-{リビジョン番号}+g{コミットID}` が付いていたら成功です。

```shell-session
$ nvim -v
NVIM v0.10.0-dev-509+g6e1fa16dd
Build type: Release
LuaJIT 2.1.0-beta3

      システム vimrc: "$VIM/sysinit.vim"
       省略時の $VIM: "/Users/uhooi/.local/nvim/share/nvim"

Run :checkhealth for more info
```

## トラブルシューティング

トラブルシューティングです。

### Xcode.appが存在しない

Xcodeをリネームや削除するとビルドできなくなります。

```shell-session
$ make CMAKE_BUILD_TYPE=Release CMAKE_INSTALL_PREFIX=$HOME/.local/nvim
mkdir -p build
touch build/.ran-deps-cmake
ninja  -C .deps
ninja: Entering directory `.deps'
[2/2] cd /Users/uhooi/ghq/github.com/neovim/neovim/.deps && /opt/homebrew/Cellar/cmake/3.26.4/bi.../lib/lib*.dylib* -P /Users/uhooi/ghq/github.com/neovim/neovim/cmake.deps/cmake/RemoveFiles.cmake
ninja  -C build
ninja: Entering directory `build'
[0/2] Re-checking globbed directories...
ninja: error: '/Applications/Xcode-15.0.0-Beta.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX14.0.sdk/usr/lib/libiconv.tbd', needed by 'bin/nvim', missing and no known rule to make it
make: *** [nvim] Error 1
```

私は `Xcode-15.0.0-Beta.app` を削除したらビルドできなくなりました。

前回のビルドで使った `libiconv.tbd` のパスをキャッシュしているようなので、 `make disclean` を実行してキャッシュを削除すると `make` のエラーが解消されます。

### リポジトリを移動する

リポジトリのパスを移動するとビルドできなくなります。
一旦削除してクローンし直すと直ります。

## おまけ

おまけです。

### スクリプトの作成

私はNeovimのビルドとインストールを頻繁に行うので、スクリプトを作成してdotfilesリポジトリへコミットしています。

https://github.com/uhooi/dotfiles/blob/37dd3e141a2d5c23305afa5ea9888d93edb1be7d/install-nvim.sh

### プラグインと言語サーバーの更新

私は毎朝Neovimのビルドのほかに、プラグインと言語サーバー、Treesitterのパーサーを更新しています。

- プラグインマネージャ: [lazy.nvim](https://github.com/folke/lazy.nvim)
- 言語サーバーマネージャ: [mason.nvim](https://github.com/williamboman/mason.nvim)
- Treesitter: [nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter)

```vim
:Lazy update
```

```vim
:MasonUpdate
:Mason
更新する言語サーバーで「u」
```

```vim
:TSUpdate
```

### HomebrewでHEADビルドする

Homebrewには `--HEAD` フラグがあるので、これを使うだけでHEADビルドができるようです。
私は手動ビルドしか行ったことがないですが、こちらのほうが楽です。

```shell-session
$ brew install nvim --HEAD
```

## おわりに

これでNeovimのHEADをビルドしてインストールできました。
毎朝温かみのある手動ビルドをして、常に最新のNeovimを使っていきましょう :relaxed:

## 参考リンク

- https://twitter.com/the_uhooi/status/1663553425509060609
- https://vim-jp.slack.com/archives/CJ3BV8EMA/p1682644516095639
