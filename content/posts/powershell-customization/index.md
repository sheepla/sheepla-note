---
title: "PowerShellのプロファイルをカスタマイズしてCLIの操作を快適にする"
date: 2021-09-05
categories: ["powershell"]
tags: ["powershell"]
---


この記事では、PowerShellのプロファイルを編集してPowerShellのCLI操作を快適にする方法を紹介します。

## プロファイルとは

PowerShellにおけるプロファイルとは、PowerShellを起動したときに自動で読み込まれるスクリプトのことです。

> PowerShell プロファイルを作成して、環境をカスタマイズし、開始するすべての PowerShell セッションにセッション固有の要素を追加できます。
> PowerShell プロファイルは、PowerShell の開始時に実行されるスクリプトです。 プロファイルをログオン スクリプトとして使用して、環境をカスタマイズできます。 コマンド、エイリアス、関数、変数、スナップイン、モジュール、PowerShell ドライブを追加できます。 また、プロファイルに他のセッション固有の要素を追加して、インポートまたは再作成することなく、すべてのセッションで使用することもできます。
> 
> [about Profiles - Microsoft Docs](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_profiles?view=powershell-7.1)

## $PROFILEを作成

プロファイルのパスは、変数 `$PROFILE` に格納されています。(OSやPowerShellのバージョンによって異なります) 新規作成するには以下のコマンドレットを実行してください。 `notepad $PROFILE` またはお好きなエディタで編集します。

```powershell
New-Item -Type File -Path $PROFILE -Force # 新規作成
notepad $PROFILE                          # 編集
```

編集したプロファイルを再読込するには、`. $PROFILE` を実行してください。

### プロファイルを読み込まずにPowerShellを起動

プロファイルを読み込まずにPowerShellを起動するには `-NoProfile` スイッチを有効にします。

```powershell
powershell -NoProfile # Windows PowerShellの場合
pwsh -NoProfile       # PowerShell Coreの場合
```

### 実行ポリシーについて

Windowsのデフォルト設定では実行ポリシーが「`Restricted`」になっており、PowerShellスクリプト(`*.ps1`, `*.ps1xml`, `*.psm1`)を実行できません。実行ポリシーを変更する必要があります。以下を実行すると、ローカルコンピューター上にあるスクリプトおよびインターネット上の電子署名済みのスクリプトを実行できるようになります。`-Scope CurrentUser` を指定することで、影響範囲を現在のユーザーのみに限定することができます。

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

実行ポリシーについては以下のドキュメントを参照してください。

> [About Execution Policies - MS Docs](https://docs.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.1#execution-policy-scope)

## 基本編(PSReadlineモジュールの設定)

PowerShellには `PSReadline` というモジュールがデフォルトでインストールされています。
`Set-PSReadlineOption`, `Set-PSReadlineKeyHandler` コマンドレットを使うことでキーバインドやシンタックスハイライトの色などをカスタマイズすることができます。詳しくは以下のドキュメントを参照してください。

> [About PSReadline - Microsoft Docs](https://docs.microsoft.com/en-us/powershell/module/psreadline/?view=powershell-7.1)

### 予測インテリセンスを有効化

PSReadlineモジュールには、予測インテリセンス機能が装備されています。これは、fishのデフォルト機能やzshで言う `zsh-autosuggestions` のように、コマンドのタイプ中に履歴から予測候補を出してくれる機能です。

> [zsh-users/zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)

この機能はデフォルトでは無効になっています。有効化するには以下を実行してください。

```powershell
Set-PSReadLineOption -PredictionSource History
```

### 重複した履歴を保存しないようにする

履歴の行が重複すると、検索して再実行する際に少し不便です。重複した履歴を保存しないようにするには以下を実行します。

```powershell
Set-PSReadlineOption -HistoryNoDuplicates
```

### ベル音の無効化

PowerShellを操作しているとベル音が頻繁に鳴り煩わしいと感じるかもしれません。ベル音を無効にするには以下を実行します。

```powershell
Set-PSReadlineOption -BellStyle = "None"
```


### キーバインドの設定

#### キーバインド一覧の取得

設定されているキーバインドの一覧は `Get-PSReadlineKeyHandler` コマンドレットで取得できます。

```powershell
PS> Get-PSReadlineKeyHandler

Basic editing functions
=======================

Key              Function              Description
---              --------              -----------
Ctrl+g           Abort                 Abort the current operation, e.g. incremental history search
Enter            AcceptLine            Accept the input or move to the next line if input is missing a closing token.
Shift+Enter      AddLine               Move the cursor to the next line without attempting to execute the input
Backspace        BackwardDeleteChar    Delete the character before the cursor
...以下略...

```

#### キーバインド一覧のチートシートを作る

キーバインド一覧を `ConvertTo-Html` コマンドレットを使ってHTMLに変換し、チートシートを作っておくと便利です。

```powershell
Get-PSReadlineKeyHandler | ConvertTo-Html | Out-File -Encoding Default ~/Documents/ps-keybindings.html
```

#### 行編集のキーバインドをEmacs/Vi風に

行編集のキーバインドをEmacs/Vi風に変更するには以下を実行してください。

```powershell
Set-PSReadlineOption -EditMode "Emacs" # Emacs風キーバインド
Set-PSReadlineOption -EditMode "Vi"    # Vi風キーバインド
```

Emacs風キーバインドにを有効にすると、カーソル移動やキルリングの他に `Ctrl+R`, `Ctrl+s` での履歴のインクリメンタルサーチが使えるようになります。bashとほぼ同様の感覚で使えるので快適です。

```
# 「Ctrl+R cd」とタイプしたときの例
PS /home/sheepla> cd ~/Documents
bck-i-search: cd_
```


#### 個別にキーを割り当てる

キーを押下したときの動作を個別に設定するには `Set-PSReadlineKeyHandler` コマンドレットを使用します。例えば、`Ctrl+o` を押下したときに補完メニューを表示するように設定するには以下を実行します。

```powershell
Set-PSReadlineKeyHandler -Key "Ctrl+o" -Function "MenuComplete"
```

#### さらに高度なキーバインド設定

`Set-PSReadlineKeyHandler` は `-ScriptBlock` パラメータを指定することでさらに高度なカスタマイズが可能です。例えば、カッコや引用符を自動で閉じるようにすることができます。こちらの記事が参考になります。

> [【PowerShell】PsReadLine 設定のススメ - Qiita](https://qiita.com/AWtnb/items/5551fcc762ed2ad92a81)

### ハイライトカラーの設定

PowerShellのコンソールではデフォルトでシンタックスハイライトが有効になっています。それぞれの色を変更するには、`Set-PSReadLineOption` の `-Colors` パラメータに値を設定します。

```powershell
Set-PSReadLineOption -Colors @{
    Command            = "Magenta"
    Number             = "DarkGray"
    Member             = "DarkGray"
    Operator           = "DarkGray"
    Type               = "DarkGray"
    Variable           = "DarkGreen"
    Parameter          = "DarkGreen"
    ContinuationPrompt = "DarkGray"
    Default            = "DarkGray"
}
```

ちなみにこの変数の値は、`ConsoleColor` 型のenum値・24bitカラーエスケープシーケンス・RGB値
の中から好きな形式で設定可能です。

```powershell
Set-PSReadLineOption -Colors @{
     # ConsoleColor型のenum値を使う
     "Error" = [ConsoleColor]::DarkRed

     # 24 bitカラーエスケープシーケンス
     "String" = "$([char]0x1b)[38;5;100m"

     # RGB値
     "Command" = "#8181f7"
}
```

### 各プロパティをまとめて設定

以下のようにハッシュテーブルを作って各プロパティをまとめて設定することもできます。ファイルに記述する際に少し見通しが良くなると思います。

```powershell
$PSReadlineOptions = @{
    BellStyle = "None"
	EditMode = "Emacs"
	HistoryNoDuplicates = $true
	HistorySearchCursorMovesToEnd = $true
	Colors = @{
		Command = "White"
	}
}
Set-PSReadlineOption @PSReadlineOptions
```

## 外部ツール編

標準機能のみを使うのも良いですが、外部のツールを導入することでPowerShellをさらに使いやすくすることができます。

### scoopでツールのインストールを自動化する

Windowsにツールを導入する場合、**scoop** や **Chocolatey** を使ってインストールを自動化すると便利です。今回はscoopをインストールします。

- [scoop -- a CLI installer for Windows](https://scoop.sh)

- [Chocolatey -- THE PACKAGE MANAGER FOR WINDOWS](https://chocolatey.org)

**メモ**: scoopとchocolateyの違いについては以下の記事が参考になります。

> [Windows開発環境の構築をChocolateyからscoopに切り替える - tech.guitarrapc.com](https://tech.guitarrapc.com/entry/2019/12/01/233522)

#### インストール

scoopをインストールするには以下を実行します。

```powershell
iwr -useb get.scoop.sh | iex
```

[lukesampson/scoop/master/bin/install.ps1 - GitHub](https://raw.githubusercontent.com/lukesampson/scoop/master/bin/install.ps1) をダウンロードして実行する方法と等価です。実行する前に目を通しておくと安心かもしれません。


### 素早くcdする(zoxide)

**zoxide** を使うと、カレントディレクトリディレクトリ素早く変更することができます。
パスを記憶しておいたり毎回タイプしたりする必要がないので、ディレクトリの階層が深くなりがちなWindowsで使うと特に有効です。

> [ajeetdsouza/zoxide - GitHub](https://github.com/ajeetdsouza/zoxide)

#### 使い方

- `z`コマンド: 基本的にはcdコマンドと同様ですが、フルパスや相対パスではなく、パスの一部だけを指定することができます。
- `zi`コマンド: 移動履歴をfzfで絞り込み、選択したディレクトリに移動します。

```powershell
z <フルパス>               # そのディレクトリに移動(cdと同じ)
z <相対パス>               # そのディレクトリに移動(cdと同じ)
z <一度移動したパスの一部> # 移動履歴を検索してパスの一部にマッチするディレクトリに移動
z -                        # 直前にいたディレクトリに戻る(cdと同じ)
z ..                       # 親ディレクトリに移動(cdと同じ)
zi <検索ワード>            # fzfを使って移動履歴を絞り込んで選択したディレクトリに移動
```

#### インストール

zoxideとfzfをインストールします。

```powershell
scoop install zoxide fzf
```

プロファイルに以下を追記してください。

```powershell
Invoke-Expression (& {
    $hook = if ($PSVersionTable.PSVersion.Major -lt 6) { 'prompt' } else { 'pwd' }
    (zoxide init --hook $hook powershell) -join "`n"
})
```

### プロンプトをカスタマイズする(starship)

外部のプロンプトツールを導入することで、PowerShellのプロンプトを見やすく・かつオシャレにすることができます。PowerShellをサポートしているものは以下の2つがあります。

- [justjanne/powerline-go - GitHub](https://github.com/justjanne/powerline-go)

- [starship/starship - GitHub](https://github.com/starship/starship)

今回はstarshipをインストールします。

> [starship - cross shell prompt](https://starship.rs)

#### starshipの導入

scoopでインストールします。

```powershell
scoop install starship
```

#### セットアップ

インストールが完了したらプロファイルに以下の行を追加してください。

```powershell
Invoke-Expression (&starship init powershell)
```

`. $PROFILE` でプロファイルを再読込してプロンプトが変化すればOKです。

#### カスタマイズ

starshipをカスタマイズするには `~/.config/starship.toml` を編集します。具体的なカスタマイズ方法については公式ドキュメントを参照してください。

> [Starship Configuration](https://starship.rs/config)

> [Starship Configuration 日本語版](https://starship.rs/ja-JP/config/)



### sudo相当のコマンドを導入する

通常、Windows上で管理者権限でコマンドを実行する場合、管理者権限でPowerShellやコマンドプロンプトを新しく起動し、UAC(ユーザーアカウント制御)ポップアップをクリックする必要があります。しかし、毎回この作業を行うのは面倒です。

回避策として、Linuxの `sudo` に相当するツールを導入すると便利です。

- [mattn/sudo - GitHub](https://github.com/mattn/sudo)
- [gerardog/gsudo - GitHub](https://github.com/gerardog/gsudo) ※ `scoop install gsudo`
- [lukesampson/psutils/sudo.ps1 - GitHub](https://github.com/lukesampson/psutils/blob/master/sudo.ps1) ※ `scoop install sudo`

**注意**: インストールと実行は自己責任で行ってください。

psutilsのsudo.ps1はPowerShellのコマンドレットをそのまま管理者権限で実行できます。今回はこれをインストールします。

#### インストール

```powershell
scoop install sudo
```

#### 使い方

```powershell
sudo <PowerShellのコマンドレットなど>
```

### fzfとの統合(PSFzfモジュール)

fzfのラッパーモジュールである **PSFzf** を導入することで、コマンドの履歴やパス絞り込み検索等の操作をfzfで簡単に行えるようになります。

> [PSFzf - PowerShell Gallery](https://www.powershellgallery.com/packages/PSFzf/2.2.8-alpha)
> [kellyma49/PSFfzf](https://github.com/kelleyma49/PSFzf)

#### 使い方

PSFzfにはディレクトリの移動・ファイルの編集・履歴の検索・プロセスのkill等をfzfで行える便利なコマンドレットが含まれています。詳しい使い方はリポジトリのREADMEを参照してください。

> [README.md on kelleyma49/PSfzf - GitHub](https://github.com/kelleyma49/PSFzf/blob/master/README.md)

#### インストール

fzfをインストールします。

```powershell
scoop install fzf
```

PSFzfをインストールします。`Get-Module -All PSFzf` を実行してPSFzfモジュールの情報が表示されればOKです。

```powershell
Install-Module -Scope CurrentUser PSFzf
```

#### 設定

ファイルパスを絞り込み入力に `Ctrl+t` 、履歴の絞り込み検索に `Ctrl+r` を割り当てるには以下の行をプロファイルに追記してください。

```powershell
Set-PsFzfOption -PSReadlineChordProvider 'Ctrl+t' -PSReadlineChordReverseHistory 'Ctrl+r'
```

<!-- 
### シンタックスハイライト+lessページャでファイルを閲覧する(bat+less)

batはシンタックスハイライト付きのcatの代替コマンドです。less-WindowsはGNU lessのWindows用バイナリです。

> [sharkdp/bat - GitHub](https://github.com/sharkdp/bat)

> [jftuga/less-Windows - GitHub](https://github.com/jftuga/less-Windows)

### 使い方

`bat <ファイル>` でファイルを閲覧できます。

#### インストール

scoopでインストール可能です。

```powershell
scoop install bat less
```
-->

