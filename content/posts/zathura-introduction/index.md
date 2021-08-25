---
title: "Linuxで高速なPDFビューワZathuraを使う"
date: 2021-08-25
categories: ["zathura"]
tags: ["zathura", "pdf", "linux"]
---

Zathuraはキーボードで軽快に操作できるドキュメントビューワです。この記事では、Zathuraのインストール方法・基本的な使い方やカスタマイズ方法を紹介していきます。

## 特徴


- Vim, lessライクなキーバインド
- 高速・軽快な動作
- ファイルの自動再読込
- GUI要素で画面を占領しないシンプルな見た目
- 背景・前景色を読みやすい色に変更(recolor)
- 設定ファイル `~/.config/zathura/zathurarc` で簡単にカスタマイズ可能
- 様々なファイル形式に対応
    - PDF
    - DJVU
    - PostScript
    - CB

## こんな人におすすめ

- とにかく軽いビューワを求めている人
- 重い・UIが煩雑・画面を占領する一般的なPDFビューワに疲れている人
- タイル型ウインドウマネージャを使っているのでキーボードから手を離さずにドキュメントを読みたい人

## インストール

### Arch Linuxの場合

Arch Linuxの場合は `zathura` パッケージとファイル形式に対応させるプラグインをインストールします。

- PDF: `zathura-pdf-poppler` または `zathura-pdf-mupdf`
- DJVU: `zathura-djvu`
- PS(PostScript): `zathura-ps`

```bash
sudo pacman -S zathura その他...
```

### Ubuntuの場合

Ubuntuの場合もaptでインストール可能です。

```bash
sudo apt install zathura その他...
```

- [zathura - Ubuntu packages](https://packages.ubuntu.com/search?keywords=zathura&searchon=all&suite=all&section=all)

## ヘルプメッセージ

```
$ zathura --help

Usage:
  zathura [OPTION…]  [file1] [file2] [...]

Help Options:
  -h, --help                           Show help options

Application Options:
  -e, --reparent=xid                   Reparents to window specified by xid (X11)
  -c, --config-dir=path                Path to the config directory
  -d, --data-dir=path                  Path to the data directory
  --cache-dir=path                     Path to the cache directory
  -p, --plugins-dir=path               Path to the directories containing plugins
  --fork                               Fork into the background
  -w, --password=password              Document password
  -P, --page=number                    Page number to go to
  -l, --log-level=level                Log level (debug, info, warning, error)
  -v, --version                        Print version information
  -x, --synctex-editor-command=cmd     Synctex editor (forwarded to the synctex command)
  --synctex-forward=position           Move to given synctex position
  --synctex-pid=pid                    Highlight given position in the given process
  --mode=mode                          Start in a non-default mode
  -b, --bookmark=bookmark              Bookmark to go to
  -f, --find=string                    Search for the given phrase and display results
```

## 使い方

操作方法はVimやlessに似ており、キーボードのみで操作を行うことができます。もちろんマウスでの操作もサポートしています。


### ページ閲覧時の操作(Vimのノーマルモードに相当)

| キー | 機能 |
|------|------|
| `j`, `k` | 上下スクロール |
| `<C-f>`, `<C-b>` | 1ページスクロール |
| `q` | Zathuraを閉じる |
| `a` | 1ページが収まるように調整 |
| `s` | 横幅にページを合わせる |
| `r` | ページを回転 |
| `F11` | フルスクリーン切替 |
| `<C-r>` | ページの色を変更(recolor) |
| `d` | 1ページ/見開き2ページ切替 |
| `f` | リンクの番号を辿る |
| `F` | リンク先を表示 |
| `o` | ファイルを開く |
| `/text` | テキストを前方に検索 |
| `?text` | テキストを後方に検索 |
| `n`, `N` | マッチしたテキストのフォーカス移動 |
| `:` | コマンドを入力するバーを表示 |

### コマンドバーでの操作(Vimのコマンドモードに相当)

| キー | 機能 |
|------|------|
| `<Tab>`, `<S-Tab>` | コマンドの候補を表示し選択 |
| `:open` | ファイルを開く |
| `:ページ番号` | 指定した番号のページにジャンプ |
| `:! コマンド...` | シェルコマンドを実行 |
| `:info` | ファイルのメタデータを表示 |
| `:prinf` | ファイルを印刷 |


## カスタマイズ

Zathuraは設定ファイルを書くことで簡単にカスタマイズすることができます。

- スクロール・ズームの段階
- キーバインド変更
- GUI要素の表示・非表示切替
- ドキュメントやUI表示色(すべてカスタマイズ可)
- 外部コマンドの実行

設定は `~/zathura/zathurarc` に記述します。また、`-c` オプションの引数にファイルのパスを指定して設定を読み込ませることでもできます。

個人的な設定例をメモしておきます。配色は [gkeep/iceberg-dark](https://github.com/gkeep/iceberg-dark) からお借りしました。

```
# ズームイン・スームアウトやスクロールの段階
set zoom-step 20
set scroll-step 80

# クリップボードを有効にする
set selection-clipboard clipboard

# インクリメンタル検索を有効にする
set incremental-search true

# キーバインド
map u scroll half-up
set d scroll half-down
map D toggle_page_mode
map K zoom in
map J zoom out

# ページの色設定
set default-fg              "#bcc0d1"
set default-bg              "#161821"
set recolor true #  デフォルトでドキュメントの色を変更する
set recolor-darkcolor       "#bcc0d1"
set recolor-lightcolor      "#161821"
set highlight-color         "#84a0c6"
set highlight-active-color  "#a093c7"

#  UIの色設定
set inputbar-fg             "#bcc0d1"
set inputbar-bg             "#161821"
set notification-fg         "#bcc0d1"
set notification-bg         "#353a50"
set notification-error-fg   "#e27878"
set notification-error-fg   "#161821"
set notification-warning-fg "#e2a478"
set notification-warning-fg "#161821"
set statusbar-fg            "#bcc0d1"
set statusbar-bg            "#444b71"
set completion-fg           "#bcc0d1"
set completion-bg           "#161821"
set completion-highlight-fg "#161821"
set completion-highlight-bg "#84a0c6"
set completion-group-fg     "#bcc0d1"
set completion-group-bg     "#353a50"

# ステータスバーに表示されるファイルパスのホームディレクトリを ~ に変更
set statusbar-home-tilde true

# ウインドウタイトルをファイルのbasenameにする
set window-title-basename true

# UI要素の表示・非表示
set guioptions cshv

# UIのフォント
set font monospace 12
```

詳しい設定ファイルの書き方は公式ドキュメントやmanpageを参照してください。

- [Zathura - Documentation](https://pwmt.org/projects/zathura/documentation/)
- `man zathurarc`

## 感想

機能の多さを売りにするビューワが多い中、Zathuraは快適にドキュメントを閲覧することに特化している印象です。キーボードでサクサク操作できる点とカスタマイズが簡単な点がとても気に入っています。個人的に一押しのビューワなのでぜひ使ってみてください！

## リンク

- [Zathura - Arch Wiki](https://wiki.archlinux.jp/index.php/Zathura)
- [Zathura (document viewer) - Wikipedia](https://en.wikipedia.org/wiki/Zathura_(document_viewer))

- [zathura: Vim-based Minimalist PDF/djvu/ps/epub/comic book reader - YouTube](https://youtu.be/V_Iz4zdyRM4)
