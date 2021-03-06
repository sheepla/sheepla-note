---
title: "pacman(+fzf)ワンライナーで快適パッケージ管理"
date: 2021-08-07
categories: ["pacman", "fzf", "oneliner"]
tags: ["pacman", "fzf", "oneliner"]
---

この記事では、Arch Linuxのパッケージマネージャー `pacman` について、ワンライナーや通常よりも一歩進んだ便利な使い方を紹介していきます。`pacman` の基本的な使い方は解説していません。以下の記事が参考になります。

- [Arch Wiki - Pacman](https://wiki.archlinux.jp/index.php/Pacman)

- [Arch Linux の pacman コマンドを使うことのメモ](http://malkalech.com/arch_linux_pacman)



## 基本編: パッケージのリストアップ

- インストール済みのパッケージについて、名前のみをすべて表示: `pacman -Qq`
- 明示的にインストールしたもののみ表示: `pacman -Qeq`
- インストール済みのパッケージのうち、アップグレード可能なものを検索: `pacman -Quq`


## インストール容量を一覧表示

インストール済みのパッケージすべての容量をリストアップし、インストールサイズの昇順でソートします。

```bash
LANG=C pacman -Qi [キーワード...] | 
    awk '/^Name/{name=$3} /^Installed Size/{print $4$5, name}' | 
    sort -h
```

### 解説

- `LANG=C` 言語設定が日本語になっている場合、キーの名前が変わってしまうので強制的にデフォルトのロケールで実行します。
- `pacman -Qi` : インストール済みのパッケージをすべて検索し、そのパッケージの詳細情報を表示します。
- `awk ...`: パッケージの名前とインストールサイズの値を取得し、`<インストールサイズ><単位> <パッケージ名>` の形式で出力します。
- `sort -h` : インストールサイズの昇順でソートします。`-h`, `--human-numeric-sort` オプションを付けることで、 `K` (キロ) `G` (ギガ) などの単位を解釈してソートすることができます。



## fzfを活用する

*fzf* は汎用のあいまい検索インタフェース(Fuzzy Finder)です。fzfを使うことでインタラクティブに項目を絞り込み、素早くパッケージを見つけることができます。

> [junegunn/fzf - GitHub](https://github.com/junegunn/fzf)

まずは `fzf` をインストールしましょう。公式のcommunityリポジトリにあります。


```bash
sudo pacman -S fzf
```


### パッケージの情報をリアルタイムプレビュー

fzfはプレビュー機能を備えておりとても便利です。以下のワンライナーを実行すると、パッケージの情報をリアルタイムプレビューで閲覧することができます。

```bash
# インストール済みのパッケージを検索する場合
pacman -Qq [キーワード...] | 
    fzf --preview 'LANG=C pacman -Qi {}' \
    --bind 'enter:execute(pacman -Qi {} | less)'

# リモートのパッケージデータベースからパッケージを検索する場合
pacman -Ssq [キーワード...] | 
    fzf --preview 'LANG=C pacman -Si {}' \
    --bind 'enter:execute(LANG=C pacman -Si {} | less)'
```

#### 使い方

1. 上記のワンライナーを実行する
2. `Ctrl+j` `Ctrl+k` キーまたは `Ctrl+n` `Ctrl+p` でカーソルを移動し`Enter`でパッケージを選択する
3. `less`ページャが起動しパッケージの詳細情報を閲覧できる
4. `q` キーを押下すると `fzf` の画面に戻る

#### 解説

- `pacman -Qq` : インストール済みのパッケージをすべて検索し、そのパッケージの名前のみ出力します。

- `pacman -Ssq キーワード...` : インストール済みのパッケージをすべて検索し、そのパッケージの名前のみ出力します。

- `pacman -Qi パッケージ名`, `pacman -Si パッケージ名` : パッケージの詳細情報を出力します。

- `fzf --preview="...{}..."`: `...` に記述したコマンドを実行し、プレビューウインドウに表示します。`{}` が現在フォーカスしている項目に置換されてから評価が行われます。

- `fzf --bind 'キー:execute(コマンド...)'`: `fzf` が起動した状態で `キー` を押下したときの挙動を設定します。

- `LANG=C` 言語設定が日本語になっている場合、プレビューウインドウの表示が崩れてしまう場合があるので強制的にデフォルトのロケールで実行します。

  

### パッケージを探して即座にインストール・アンインストール

```bash
# インストール
pacman -Ssq [キーワード...] | 
    fzf --multi --preview 'LANG=C pacman -Si {}' | 
    sudo pacman -S -

# アンインストール
pacman -Qq [キーワード...] | 
    fzf --multi --preview 'LANG=C pacman -Qi {}' |
    sudo pacman -Rn -
```

#### 使い方

基本的には上記と同様です。`Tab` キーを押下することで複数のパッケージを選択することができます。 `Enter` で選択を確定すると、インストール・アンインストールの内容を確認するプロンプトが表示されます。

#### 解説

- ` sudo pacman -S パッケージ名` : パッケージをインストールします。 引数にパッケージ名ではなく`-` を与えると、標準入力から受け取ったテキストをパッケージ名と解釈してそのパッケージをインストールします。
- `sudo pacman -Rn パッケージ名`: パッケージをアンインストールします。`-S` と同様、引数に `-` を与えると、標準入力から受け取ったテキストをパッケージ名と解釈してそのパッケージをアンインストールします。
    - 任意のパッケージを再帰的に削除するには `sudo pacman -Rns パッケージ名` を使います。

- `fzf --multi`: fzfが起動した状態で `Tab` キーを押下することで、複数の項目を選択することができます。

### パッケージのURLをブラウザで開く

fzfでパッケージを絞り込み、選択したパッケージのURLをデフォルトのブラウザで開きます。

```bash
LANG=C pacman -Ssq [キーワード...] |
    fzf --multi --preview 'LANG=C pacman -Si {}' |
    pacman -Si - |
    awk '/^URL/{print $3}' |
    xargs -I@ -- xdg-open @
```

#### 解説

- `awk...`: URLの値を取り出します。
- `xdg-open`: xdg-openは引数に与えられたファイルパスやURLをデフォルトのアプリケーションで開くコマンドです。Windowsの `start` やmacOSの `open` に相当します。

    **メモ**

    xdg-openは `xdg-utils` パッケージに含まれています。詳しくは以下の記事が参考になります。

    > [ターミナルからファイルを開く - sekikka.github.io](https://sekika.github.io/2015/10/27/open-command/)

    > [xdg-open - Arch Linux Manual](https://man.archlinux.org/man/xdg-open.1)

- それ以外は上記と同様です。


## 参考

- [Arch Wiki - Pacmanヒント](https://wiki.archlinux.jp/index.php/Pacman_%E3%83%92%E3%83%B3%E3%83%88)

- [Arch Wiki -Fzf](https://wiki.archlinux.jp/index.php/Fzf#Arch_.E3.81.A7_fzf_.E3.82.92.E4.BD.BF.E7.94.A8.E3.81.99.E3.82.8B)

