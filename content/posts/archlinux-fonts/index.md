---
title: "Arch Linuxにフォントをインストールする"
date: 2021-08-28
categories: ["archlinux", "fonts"]
tags: ["archlinux", "fonts"]
---

# 

この記事では、Arch Linuxにフォントをインストールする方法や、フォントに関連した便利なツールの使い方をメモしています。

## パッケージを探す

公式リポジトリやAURには豊富なフォントのパッケージがあり、Arch Wikiにはその一覧が纏まっています。


> [フォント - フォントパッケージ - ArchWiki](https://wiki.archlinux.jp/index.php/フォント?rdfrom=https%3A%2F%2Fwiki.archlinux.org%2Findex.php%3Ftitle%3D%25E3%2583%2595%25E3%2582%25A9%25E3%2583%25B3%25E3%2583%2588%26redirect%3Dno#.E3.83.95.E3.82.A9.E3.83.B3.E3.83.88.E3.83.91.E3.83.83.E3.82.B1.E3.83.BC.E3.82.B8)

CLIからは以下の方法で探すことができます。`yay`, `paru` 等のAURヘルパーを使っている場合は読み換えて実行してください。


```bash
pacman -Ss font\|ttf\|otf
```

## 手動でインストール

欲しいフォントが公式パッケージやAURに無い場合は、以下の方法で手動インストールします。

1. フォントをダウンロードする
1. 展開する
1. 所定のディレクトリにインストールする

    - 1つのユーザーのみにインストールする場合: `~/.local/share/fonts/`
    - システムにインストールする場合: `/usr/local/share/fonts/`

**メモ**: TrueTypeフォントは `...略.../fonts/TTF/`に、OpenTypeフォントは `...略.../fonts/OTF/` にインストールする慣習があるようです。（要調査）

**メモ**: `~/.fonts/` は非推奨となっているようです。[フォント - 手動インストール - Arch Wiki](https://wiki.archlinux.jp/index.php/%E3%83%95%E3%82%A9%E3%83%B3%E3%83%88?rdfrom=https%3A%2F%2Fwiki.archlinux.org%2Findex.php%3Ftitle%3D%25E3%2583%2595%25E3%2582%25A9%25E3%2583%25B3%25E3%2583%2588%26redirect%3Dno#.E6.89.8B.E5.8B.95.E3.82.A4.E3.83.B3.E3.82.B9.E3.83.88.E3.83.BC.E3.83.AB)

**メモ**: アプリケーション側でフォントが更新されない場合はfontconfigのキャッシュを強制更新すると治る場合があります。: `fc-cache -f`


## フォントファイルを自動でパッケージ化してインストール(makefontpkg)

makefontpkgを使うと、TTF, OTFフォントファイルをパッケージ化し、pacmanで管理できるようにすることができます。自分で `PKGBUILD` を書く手間が省けるので便利です。

> [misterdanb/makefontpkg - GitHub](https://github.com/misterdanb/makefontpkg)

### インストール

AURからインストールすることができます。

> [makefontpkg - AUR](https://aur.archlinux.org/packages/makefontpkg/) 

```bash
yay -S makefontpkg
```

### 使い方

引数にフォントファイルを指定します。インストールを行う場合は `-i` オプションを付けます。

```
makefontpkg [-h] [-i | -S] [-n NAME] [--ver VER[-REL]] [--desc DESC]
            FILE [FILE ...]
```

パッケージの名前・説明・バージョンは自動で設定されますが、オプションの引数により変更可能です。

- `-n パッケージ名`
- `-d 説明`
- `-v バージョン`

## インストール済みのフォントを探す(fc-list)

`fc-list` コマンドを実行すると、インストール済みのフォント一覧を表示することができます。

**メモ**: 

- フォントのパス一覧は `fc-list | cut -d : -f1 | sort | uniq` で取得できます。
- 名前一覧は `fc-list | cut -d : -f2 | sort | uniq` で取得できます。

**メモ**: 日本語のフォントのみに絞り込みたい場合は `fc-list :lang=ja` を実行してください。


## フォントファイルを解析する(fc-scan)

`*.ttf`、 `*.otf` 形式のフォントファイルを解析して、メタデータを取得するには`fc-scan <フォントファイルへのパス>` を実行します。

**例**:

```bash
$ fc-scan /usr/share/fonts/Inconsolata-Regular.ttf | head
attern has 27 elts (size 32)
	family: "Inconsolata"(s)
	familylang: "en"(s)
	style: "Regular"(s)
	stylelang: "en"(s)
	fullname: "Inconsolata Regular"(s)
	fullnamelang: "en"(s)
	slant: 0(i)(s)
	weight: 80(f)(s)
	width: 100(f)(s)
```



## ターミナル上でフォントをプレビューする(fontpreview-ueberzug)

インストール済みのフォントをプレビューして確認するツールとして、[gnome-font-viewer](https://gitlab.gnome.org/GNOME/gnome-font-viewer)  や [font-manager](https://github.com/FontManager/font-manager) 等がありますが、個人的におすすめなのはfontpreview-ueberzugです。

> [OliverLew/fontpreview-ueberzug - GitHub](https://github.com/OliverLew/fontpreview-ueberzug) 

<img src="https://github.com/OliverLew/fontpreview-ueberzug/blob/master/demo.gif?raw=true" width="80%" />

### インストール

AURからインストール可能です。

> [fontpreview-ueberzug-git - AUR](https://aur.archlinux.org/packages/fontpreview-ueberzug-git/)

```bash
yay -S fontpreview-ueberzug-git
```

### 使い方

`fontpreview-ueberzug`を起動すると、fzfのプレビューウインドウ上にフォントのプレビューが表示されます。名前の一部を入力すると選択肢が絞り込まれます。`<C-j>`, `<C-k>` または `<C-n>`, `<C-p>` を押下してフォーカスを移動すると、リアルタイムにプレビューが更新されます。

## 参考

- [フォント - Arch Wiki](https://wiki.archlinux.jp/index.php/フォント?rdfrom=https%3A%2F%2Fwiki.archlinux.org%2Findex.php%3Ftitle%3D%25E3%2583%2595%25E3%2582%25A9%25E3%2583%25B3%25E3%2583%2588%26redirect%3Dno#.E3.83.95.E3.82.A9.E3.83.B3.E3.83.88.E3.83.91.E3.83.83.E3.82.B1.E3.83.BC.E3.82.B8)
- [フォント設定 - Arch Wiki](https://wiki.archlinux.jp/index.php/%E3%83%95%E3%82%A9%E3%83%B3%E3%83%88%E8%A8%AD%E5%AE%9A)
