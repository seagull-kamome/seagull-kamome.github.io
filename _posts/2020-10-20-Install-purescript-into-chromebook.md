---
title: "Install purescript environment on Chromebook."
author: "Hattori, Hiroki"
layout: "post"
tag: ["Chromebook", "Purescript"]
categories: ["blog"]
---

　Crostini入れたはいいけど、apt-getでnodejsとnpm入れると、バージョンが
不整合になってる上に、そこからpurescriptをインストールするとコケてしまっ
て入らない。 

- バージョン不整合をなんとかしたい 
- Purescriptをちゃんといれたい 


　回避方法としては、nvmをつかってnode.jsとnpmを入れるしかないらしい。

　以下の手順でHP 14aには入りました。が、CT100PAには無理でした。
purescriptはaarch64をサポートしていません。正確にはstackagega
サポートしていないのでpurescript側もどうにもならん。

<!-- more -->

### 手順 

#### Nvmをhomeに(ここ大事)インストール 

````
git clone https://github.com/creationix/nvm.git ~/.nvm 
cd ~/.nvm 
git checkout `git describe --abbrev=0 --tags` 
````

　最後の一文は要するに「最新のリリースタグが付いたバージョンを使え」
という事。 

````
. ~/.nvm/nvm.sh 
nvm install v12 
````

　nvm ls-remoteでインストール可能なバージョンの一覧が出るので、最新の
LTSを入れる。現時点ではv12でした。 Nvm.shが環境変数をセットアップして
パスを通してくれるので、~/.profileの最後に追加しておくこと。 

 

#### 依存パッケージを入れる 

　Purescriptがlibtinfo5を要求してくる。多分REPLで使うんだろうけど、
こいつはもうレガシーになっていて標準では入っていないので手動で入れる。 

````
apt-get install libtinfo5 
````

#### Node.jsとnpmをインストール 

````
npm install –g purescript 
npm install –g spago 
````

　-gは”グローバルにインストールする”という意味だが、npm自身をhome以下
に入れているので、ここいう”グローバル”とは"プロジェクトローカルではなく、
homeの今使ってるnodeのディレクトリに入れる”という意味らしい。

~/.nvm/versions/node/v12.19.0/bin/pursにインストールできました。
やったー 


#### aarch64対応とか 

　Ghcはaarch64のバイナリを出しているが、stackageがいつからかaarch64の
バイナリ提供をやめてx86_64一本になってしまったせいで、haskellで書かれた
プロジェクトはcabalに戻るかaarch64を捨てるかの選択に迫られるっぽい。
まぁ、いまさらcabalに戻りたい人なんていないのでaarch64が捨てられると...

以上

