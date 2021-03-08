---
title: "ぼくがかんがえたさいきょうのターミナル環境を作る"
emoji: "⌨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CLI", "zsh", "Powerline", "環境構築"]
published: false
---

## こんなのをめざすよ

![](https://storage.googleapis.com/zenn-user-upload/jw6xfde83kaa668koawzhtt9zh1z)


## 前提とか

:::message alert
- **この手順は主に自分用のメモなので大事なところの説明が抜けてるかもしれないよ**
- Windows Terminalから `wsl2(Ubuntu-20.04-LTS)` をつかうよ
:::


## Powerline対応フォントインストール

`zprezto`の`PowerLevel10k`テーマを使うので、それに対応したやつを入れる

[romkatv/powerlevel10k](https://github.com/romkatv/powerlevel10k#fonts) から以下をお好みで。

- [MesloLGS NF Regular](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf)
- [MesloLGS NF Bold](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold.ttf)
- [MesloLGS NF Italic](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Italic.ttf)
- [MesloLGS NF Bold Italic](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold%20Italic.ttf)


## Windows Terminalの設定いじる

`Ctrl+,`して`settings.json`を編集。
各パラメータ値はお好みでいじってください

### 直下のとこ

```json:settings.json
"defaultProfile": "Ubuntu-20.04",

"initialRows": 50,
"initialCols": 200,
```

### profiles/defaultsの中

```json:settings.json
"startingDirectory": ".",

"fontFace": "MesloLGS NF",
"fontSize": 9,
"antialiasingMode": "cleartype",

"useAcrylic": true,
"acrylicOpacity": 0.95
```

### profiles/list/Ubuntuの中

開始位置をWSL側のホームディレクトリにする

```json:settings.json
"startingDirectory": "//wsl$/Ubuntu-20.04/home/{user-name}",
```

:::message
`{user-name}`はWSLのユーザー名で。
:::


## いざWSLへ

### sudoをパスワードなしでできるように

WSLだと面倒なだけだし、ね

```bash
sudo visudo
```

して

```diff
- %sudo   ALL=(ALL:ALL) ALL
+ %sudo   ALL=(ALL:ALL) NOPASSWD:ALL
```

### とりあえず更新

```bash
sudo apt update
sudo apt upgrade
```


## Homebrewインストール

[公式](https://brew.sh/)通りに入れて

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

インストール後の指示に従う。

```bash
sudo apt install build-essential
```
```bash
echo 'eval $(/home/linuxbrew/.linuxbrew/bin/brew shellenv)' >> ~/.profile
eval $(/home/linuxbrew/.linuxbrew/bin/brew shellenv)
```
```bash
brew install gcc
```

### doctorしておく

```bash
brew doctor
```

適宜対応。


## zshインストール

```bash
brew install zsh
```

### デフォルトシェルを切り替える

```bash
which zsh | sudo tee -a /etc/shells
chsh -s `which zsh`
```

したあと、Windows Terminal立ち上げ直す

### zshの初期設定

preztoを入れるので、ここでは即`Quit`


## preztoインストール

```shell
git clone --recursive https://github.com/sorin-ionescu/prezto.git "${ZDOTDIR:-$HOME}/.zprezto"
```

おわったら

```shell
setopt EXTENDED_GLOB
for rcfile in "${ZDOTDIR:-$HOME}"/.zprezto/runcoms/^README.md(.N); do
  ln -s "$rcfile" "${ZDOTDIR:-$HOME}/.${rcfile:t}"
done
```

### zshにもHomebrewのパスとおす

```shell
echo 'eval $(/home/linuxbrew/.linuxbrew/bin/brew shellenv)' >> ~/.zprofile
eval $(/home/linuxbrew/.linuxbrew/bin/brew shellenv)
```

ここで一旦シェル再ログイン（Windows Terminalを再起動）


## Powerline導入

preztoのテーマをPowerLevel10kに変更

```shell
prompt -s powerlevel10k
```

Powerlevel10kのウィザードが出るのでお好みで。

:::message
再設定したいときは `p10k configure` を叩くとまたウィザードが出てくる
:::


終わったら、`~/.zshrc`に以下追記

```shell:~/.zshrc
autoload -Uz promptinit
promptinit
prompt powerlevel10k
```


## 素敵コマンド類をいれる

### exa

`ll`をいい感じにしてくれるやーつ。

```shell
brew install exa
```

インストールしたら

```shell
exa -l -aa -h -@ -m --icons --git --time-style=long-iso --color=automatic --group-directories-first
```

とかするとちょっと感動する。

### bat

シンタックスハイライトができる`cat`。

```shell
brew install bat
```

コマンドもそこまで変わらないので、こっちは特にaliasとかはいいかな。

### zsh-syntax-highlighting

zshのコマンドをいい感じにハイライトするやーつ。

```shell
brew install zsh-syntax-highlighting
```

インストールしたら `~/.zshrc` に

```shell:~/.zshrc
source /home/linuxbrew/.linuxbrew/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
```

を追記して `source ~/.zshrc`


## gitの設定

### GitをaptではなくHomebrewで入れる

`apt`だとバージョン上がらないみたいなので……

```shell
brew install git
```

:::message
`sudo apt purge git` はしちゃダメ。
`brew upgrade git`の中で `git config --replace-all homebrew.private true` をしようとするときにコケる
:::

### git-completionを有効化

`~/.zshrc` に追記

```shell:~/.zshrc
fpath=($(brew --prefix)/share/zsh/site-functions $fpath)

autoload -U compinit
compinit -u
```

したら `source ~/.zshrc`

### diff-highlightをパス通す

```shell
sudo chmod +x /home/linuxbrew/.linuxbrew/share/git-core/contrib/diff-highlight/diff-highlight
sudo ln -s /home/linuxbrew/.linuxbrew/share/git-core/contrib/diff-highlight/diff-highlight /usr/local/bin/diff-highlight
```


## その他zsh設定

### 使い慣れてたaliases

とりあえず`~/.zshrc`へ。

```shell:~/.zshrc
alias g=git
alias ll="exa -l -h -@ -mU --icons --git --time-style=long-iso --color=automatic --group-directories-first"
alias l="ll -aa"
```

ぶちこんだら、

```shell
source ~/.zshrc
```

### `sudoedit`がnanoなのは嫌なの

コマンドで設定変える

```shell
sudo update-alternatives --config editor
```

`~/.zprofile`に環境変数ぶちこむ

```shell:~/.zprofile
export EDITOR=vi
export SUDO_EDITOR=vi
```

### タブ補完くっそ重いので、Windows側のパスを排除する

```ini:/etc/wsl.conf
[interop]
appendWindowsPath = false
```

\# 要WSL再起動


---

以上です。お疲れさまでした！
