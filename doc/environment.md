# 環境構築

## アジェンダ

0. [はじめに](#0-はじめに)
1. [Rubyのインストール](#1-Rubyのインストール)
2. [プロジェクト作成](#2-プロジェクト作成)
3. [Docker化](#3-Docker化)
4. [DBおよびRDBMSの設定](#4-DBおよびRDBMSの設定)
5. [補足](#5-補足)
6. [参考](#6-参考)

## 0. はじめに

当ドキュメントは1からRuby on Railsの環境構築を行う方法を記載したものであり、リポジトリ内に存在するソースコード等を使用して何かを行うものではないため注意。  
当リポジトリの構築に関しては、[README.md](../README.md)を参照すること。

当ドキュメント手順ではホストマシンにRubyおよびRailsを導入する説明があるが、最終的にDocker化するため、ホストマシンを汚したくない場合はスキップしても良い。  

また、Rubyのインストールに関してはOSによって差異が出るため、適宜読み替える必要がある。  
当ドキュメントではWindows(GUI, WSL2)を使用した一連の操作手順を記載する。  
WindowsではWSL2、MacOSではhomebrewを強く推奨する。（GUIでも問題はないが、コマンド操作の練習にもなり、かつ慣れるとCLIの方が早くて確実なため。）  
適宜[こちら](#6-参考)も参考にすること。　

## 1. Rubyのインストール

<details>

<summary>GUI</summary>

### Ruby(RubyInstallers)のダウンロード

https://rubyinstaller.org/downloads/

![rubyinstaller org_downloads_](https://user-images.githubusercontent.com/85621608/125203782-f954b180-e2b4-11eb-8db4-de5a326d55c7.png)

Windows向けのインストーラーがある。  
当ドキュメントではバージョン3.0.1を使用するため、それに合わせてインストーラーを選択する。

インストーラーを手順通りに進める。  

![rubyinstaller-devkit-3 0 1-1-x64_1](https://user-images.githubusercontent.com/85621608/125203804-0ec9db80-e2b5-11eb-8989-a0dea3a28c40.png)

![rubyinstaller-devkit-3 0 1-1-x64_2](https://user-images.githubusercontent.com/85621608/125203809-1ee1bb00-e2b5-11eb-9aa4-2c6e43f3d7fd.png)


![rubyinstaller-devkit-3 0 1-1-x64_3](https://user-images.githubusercontent.com/85621608/125203820-2b661380-e2b5-11eb-96fa-9f9e7ee91f75.png)

![rubyinstaller-devkit-3 0 1-1-x64_4](https://user-images.githubusercontent.com/85621608/125203826-3a4cc600-e2b5-11eb-8f28-666c9c059c87.png)

チェックを入れたまま終了すると、以下のウィンドウが起動する。

![cmd](https://user-images.githubusercontent.com/85621608/125203835-40db3d80-e2b5-11eb-84cb-d3eb9675b0c6.png)

MSYS2を使用する場合はEnterを押してインストールするが、今回は特に必要ないため閉じて良い。  

コマンドプロンプトにて下記コマンドを実行して、それぞれのバージョンが確認できればインストール完了。  

```cmd
ruby -v
ruby 3.0.1p64 (2021-04-05 revision 0fb782ee38) [x64-mingw32]
```

```cmd
gem -v
3.2.15
```

</details>

<details>

<summary>CLI (WSL2)</summary>

### 各種ライブラリのインストール

```bash
sudo apt install -y make gcc libssl-dev libreadline-dev zlib1g-dev npm sqlite libsqlite3-dev
```

### node.js および n

```bash
sudo npm install -g n
sudo n stable
```

### yarn

```bash
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt update && sudo apt install -y yarn
```

### rbenv

```bash
git clone https://github.com/sstephenson/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
exec $SHELL -l
```

### ruby-build

```bash
git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
```

### ruby

2021/06/20現在、安定版の`3.0.1`をインストール。

```bash
rbenv install 3.0.1
```

### 確認

RubyとGemのバージョンが確認できれば完了。

```bash
ruby -v
ruby 3.0.1p64 (2021-04-05 revision 0fb782ee38) [x86_64-linux]
```

```bash
gem -v
3.2.15
```

</details>

## 2. プロジェクト作成

Ruby on Railsのプロジェクトを作成する。

### Railsのインストール

```bash
gem install rails
```

### アプリケーションの作成と移動

```bash
rails new app
```

### webpackerのインストール

*作成したアプリケーションのディレクトリで実行

```bash
rails webpacker:install
```

### 動作確認

下記コマンドを実行。

```bash
rails s
```

`http://127.0.0.1:3000`へアクセスし、以下のページが表示されれば完了。

![127 0 0 1_3000_](https://user-images.githubusercontent.com/85621608/125203681-6a479980-e2b4-11eb-8e14-771ee1420822.png)

## 3. Docker化

### フォルダ構成

```
.
├── app
│   │── source
|   |   └── ... アプリケーションのソースコード
│   └── Dockerfile
├── db
│   └── Dockerfile
├── docker-compose.yml
└── README.md
```

<details>

<summary>CLI (WSL2)を使用したファイル移動</summary>

#### ソースコード移動

```bash
rsync --remove-source-files -R $(find . -type d -name ".git" -prune -o -type f -print) ./source
```

#### 移動時に残ったディレクトリの削除

```bash
find * -type d -name ".git" -prune -o -name "source" -prune -o -type d -print | xargs rm -rf
```

#### フォルダ作成および移動

```bash
mkdir -p app db; mv source/ app/source
```

</details>

### docker関連のファイル作成

#### app/Dockerfile

```Dockerfile
FROM ruby

RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
  && echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
RUN apt-get update && apt-get install -y nodejs yarn

WORKDIR /app
ADD ./source/Gemfile Gemfile
RUN bundle install
```

#### db/Dockerfile

```Dockerfile
FROM postgres
ENV POSTGRES_USER root
ENV POSTGRES_PASSWORD password
```

#### docker-compose.yml

```yml
version: '3'

services: 
    db:
        build: ./db/
        ports: 
            - "5432:5432"
    
    app:
        build: ./app/
        tty: true
        depends_on: 
            - db
        volumes: 
            - ./app/source/:/app
        ports:
            - "3000:3000"
    
    pgadmin4:
        image: dpage/pgadmin4
        depends_on:
            - db
        environment: 
            PGADMIN_DEFAULT_EMAIL: root@root.com
            PGADMIN_DEFAULT_PASSWORD: password
        ports:
            - "8000:80"
```

pgadmin4は動作に必須ではないが、開発するうえであると便利なため導入する。

### 動作確認

全コンテナ起動

```bash
docker-compose up -d --build
```

#### Rails

サーバーの立ち上げ

```bash
docker-compose exec app rails s -b 0.0.0.0
```

`http://127.0.0.1:3000`へアクセスし、以下のページが表示されれば完了。

![127 0 0 1_3000_](https://user-images.githubusercontent.com/85621608/125203681-6a479980-e2b4-11eb-8e14-771ee1420822.png)

#### pgAdmin4

`http://127.0.0.1:8000`へアクセスし、以下のページが表示される。

![127 0 0 1_8000_](https://user-images.githubusercontent.com/85621608/125203706-906d3980-e2b4-11eb-9bba-8898fc472d8d.png)

`docker-compose.yml`で設定した`PGADMIN_DEFAULT_EMAIL`, `PGADMIN_DEFAULT_PASSWORD`を使用してログインする。  
以下のページが表示されれば完了。

![127 0 0 1_8000_browser_](https://user-images.githubusercontent.com/85621608/125203749-d1fde480-e2b4-11eb-910d-62a27ffede9a.png)

## 4. DBおよびRDBMSの設定

DBとRDBMSのコンテナを立ててはいるが、Railsと連携されていないため、RailsのDB設定を変更し、pgAdmin4にて疎通確認を行う。

### Rails

#### app/source/config/database.yml

```yml
default: &default
  adapter: postgresql
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  timeout: 5000
  encoding: unicode
  host: db
  username: root
  password: password

development:
  <<: *default
  database: app_development

test:
  <<: *default
  database: app_test

production:
  <<: *default
  database: app_production
  username: app
  password: <%= ENV['APP_DATABASE_PASSWORD'] %>
```

hostは`docker-compose.yml`で指定したサービス名。  
username, passwordは`db/Dockerfile`にて指定したもの。

#### app/source/Gemfile

```Gemfile
gem 'sqlite3', '~> 1.4'
```
↓
```Gemfile
gem 'pg', '~> 1.2.3'
```

`sqlite3`から`pg`へ変更。  
Gemfileの変更を行ったら、ソースコードがあるディレクトリにて下記コマンドを実行。

```bash
bundle install
```

gemのインストールが完了したら、下記コマンドでDBを作成する。

```bash
rails db:create
```

### pgAdmin4

「クリックリンク」 -> 「新しいサーバを追加」

![127 0 0 1_8000_browser_(2)](https://user-images.githubusercontent.com/85621608/125203715-9a8f3800-e2b4-11eb-86b0-0110dce70dbb.png)


以下を入力し、「保存」を押す。 

|タブ|項目|値|備考|
|-|-|-|-|
|一般|名称|任意|pdAdmin4の左側のメニューに表示される|
|接続|ホスト名/アドレス|db|`docker-compose.yml`で設定したDBのサービス名|
||パスワード|password|`db/Dockerfile`で設定したDBのユーザーパスワード|

![127 0 0 1_8000_browser_(3)](https://user-images.githubusercontent.com/85621608/125203734-aed33500-e2b4-11eb-8bc3-60e4e80386cf.png)

ページ左側のメニューに上記のDB情報が表示されていれば疎通確認完了。

## 5. 補足

### Dockerfileに関して

```Dockerfile
FROM ruby

RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
  && echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
RUN apt-get update && apt-get install -y nodejs yarn
```

|記述|説明|
|-|-|
|FROM|使用するイメージの指定|
|RUN|コマンドの実行|

rubyのイメージを取得し、railsを動かすのに必要なパッケージのインストールを行う。  
ここではデフォルトパッケージのアップデート、node.js, yarnのインストールを行っている。  

```Dockerfile
WORKDIR /app
ADD ./source/Gemfile Gemfile
RUN bundle install
```

|記述|説明|
|-|-|
|WORKDIR|作業フォルダの移動（なければ作成）|
|ADD|ファイルの追加<br>`追加対象ファイル・フォルダのパス 配置先パス`|

コンテナにソースコードが存在しておらずgemのインストールができないため、先にGemfileのみをコンテナに追加。  
カレントディレクトリは`/app`なので、`/app`配下に`Gemfile`が配置されている状態となる。  
配置したGemfileを参照してgemのインストールを行う。

```Dockerfile
FROM postgres
ENV POSTGRES_USER root
ENV POSTGRES_PASSWORD password
```

|記述|説明|
|-|-|
|ENV|環境変数の設定|

postgresのイメージを指定し、ユーザー名・パスワードの環境変数を設定。

### docker-compose.ymlに関して

```yml
version : '3'
```

|記述|説明|
|-|-|
|version|[DockerComposeのバージョン](https://matsuand.github.io/docs.docker.jp.onthefly/compose/compose-file/)を指定|

```yml
db:
    build: ./db/
    ports: 
        - "5432:5432"
```

|記述|説明|
|-|-|
|build|docker-compose.ymlからDockerfileへの相対パスを指定|
|ports|`Dockerホストのポート:コンテナのポート`<br>ポートの話が分からない場合は[ネットワーク関連](#ネットワーク関連)の閲覧を推奨|

```yml
app:
    build: ./app/
    tty: true
    depends_on: 
        - db
    volumes: 
        - ./app/source/:/app
    ports:
        - "3000:3000"
```

|記述|説明|
|-|-|
|build , ports|割愛|
|tty|dockerコンテナを常に起動しておく設定|
|depends_on|コンテナ間の依存関係の設定|
|volumes|ボリュームへ配置するファイルの相対パス、およびボリュームのパス|

**tty**  

現在のappコンテナの設定上、起動時はポート待ち受けをしていない状態。  
ポート待ち受けをしていない状態ではコンテナが落ちるようになっているため、ttyを設定して落とされないようにする。  
開発中、サーバーの再起動をしたい場合、その都度コンテナを再起動する必要があるため一旦このようにしている。  
appコンテナの起動と同時にrailsでサーバーを起動する設定にした場合は、起動後すぐにポート待ち受けが開始されるためttyは不要。  

**depends_on**

起動順番の制御や、サービス名（コンテナ名）をホストとして設定に記述することができる。  
[app/source/config/database.yml](#appsourceconfigdatabaseyml)の`host`が例。  

**volumes**

データの永続化を行う領域（ボリューム）に特定のファイルを配置する。  
この場合`./app/source/`配下にあるソースコードを、ボリュームの`/app`に配置している。  
そしてこのボリュームの設定をappコンテナに記載しているため、ボリュームはappコンテナにマウントされ、`/app`にソースコードが配置されている状態になる。

```yml
pgadmin4:
    image: dpage/pgadmin4
    depends_on:
        - db
    environment: 
        PGADMIN_DEFAULT_EMAIL: root@root.com
        PGADMIN_DEFAULT_PASSWORD: password
    ports:
        - "8000:80"
```

|記述|説明|
|-|-|
|depends_on , ports|割愛|
|image|Dockerイメージの指定|
|environment|環境変数の設定|

pgadmin4のイメージを指定し、ログインに必要なユーザー名およびパスワードを`environment`にて指定する。    
余談だが、個人的には必須ではないコンテナのDockerfileは作りたくないため、docker-compose.yml内で完結させたいというこだわりがある・・。

## 6. 参考

### WSL2

[Windows 10でLinuxを使う(WSL2)](https://qiita.com/whim0321/items/ed76b490daaec152dc69)

[Win10+WSL2+rbenv+Ruby3.0+Rails6.1導入](https://qiita.com/Hirororocky/items/d5a035a2a1a7961cf12a)

[[Rails] Windows10 で WSL を使って Rails 環境を構築したときのメモ](https://qiita.com/ksh-fthr/items/64a4e86c8bad08322c94)

### MacOS

[Mac M1 Big Sur にRuby / Railsをインストール 2021-01](https://qiita.com/kazutosato/items/6dea35e97f39d8d13e83)

### Docker関連

[Dockerfile リファレンス](https://docs.docker.jp/engine/reference/builder.html)

[docker-compose リファレンス](https://docs.docker.jp/compose/toc.html)

[docker-compose up したコンテナを起動させ続ける方法](https://qiita.com/sekitaka_1214/items/2af73d5dc56c6af8a167)

[Docker、ボリューム(Volume)について真面目に調べた](https://qiita.com/gounx2/items/23b0dc8b8b95cc629f32)

### ネットワーク関連

[用語集　「ポート(PORT)とは？」](https://www.cman.jp/network/term/port/)

[SSHポートフォワーディングを知った話](https://qiita.com/Ayaka14/items/449e2236af4b8c2beb81)