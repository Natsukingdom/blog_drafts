# docker 公式 ガイド(英語)を読んで気になる部分の翻訳

あんまり英訳が得意じゃないので、ちゃんと誰もがわかる言葉に落とし込めていない可能性があります。あくまで個人的なメモがわりと釈明させてください。

[GetStartedDockeGet Started, Part 1: Orientation and setup | Docker Documentationr](https://docs.docker.com/get-started/)

## Part1. Orientation and setup

### コンテナの特徴

- 柔軟性: とてもふくざつなアプリケーションでもコンテナ化することができる
- 軽量性: コンテナはホストのカーネルを利用する,そしてそれを共有する.
- 交換可能性: そのばでアップデートやアップグレードを適用可能.
- 携帯性: ローカルでビルド可能、クラウドへもデプロイ可能、どこでも動かせる.
- スケーラビリティ: 稼働するアプリケーションを増やすことができ、コンテナの複製を自動的に配布可能
- スタッカビリティ: その場でサービスに対してコンポーネントを積み上げることができる

### イメージとコンテナ

コンテナは イメージによって実行されます。

イメージは、ソースコード、ランタイム、ライブラリ、環境変数、設定ファイルなど必要なものがすべて含まれた実行可能なパッケージのことです。

コンテナは、実行時インスタンスです。これは、実行されるときに、イメージがメモリの中でどのようにあるか（つまり、状態を持ったイメージ、もしくはユーザプロセス）のインスタンスということです。`docker ps` コマンドを使って、Linuxでやるように、稼働させているコンテナのリストを確認することができます。

### Docker 環境の用意

コミュニティエディション、もしくはエンタプライズエディションの保守されているバージョンをインストールしてください。

[Install Docker](https://docs.docker.com/install/)

### Docker バージョンのテスト

1. サポートされているバージョンのDocker をインストールしていることを確認してください。

   ```sh
   docker --version
   ```

1. より詳細なインストール情報を知るために以下のコマンドを実行してください。

   ```sh
   docker info
   ```

### Docker の インストールのテスト

1. hello-world イメージで docker が 動くかどうか確かめましょう.

   ```sh
   docker run hello-world
   ```

1. ダウンロードされたhello-world イメージをリスト表示させてみましょう.

   ```sh
   docker image ls
   ```

1. hello-worldのコンテナをリスト表示してみましょう

   ```sh
   docker container ls --all
   ```

### 要約とチートシート

まぁこのへんしっかり抑えたいときは、自分で help 叩いてみるのが良さげ.

```sh
## docker の cli コマンドをリスト表示する.
docker
docker container --help

## docker の バージョンと 情報を表示する.
docker --version
docker version
docker info

## docker image を 実行する.
docker run hello-world

## docker image を リスト表示する.
docker image ls

## docker conatainer をリスト表示する.
docker container ls ## 実行中のもの
docker container ls --all ## すべて
docker container ls -aq ## quiet mode 中のものすべて
```

### パート1の結論

コンテナ化は、継続的インテグレーション、継続的デプロイを継ぎ目のないものにします。

- アプリケーションは、システムに依存性を持たない.
- アップデートは分散アプリケーションの任意の部分に対してプッシュ可能.
- リソース密度を最適化可能.

Docker を利用することで、あなたのアプリケーションをスケーリングすることは、動作の重たいＶＭホストを動かすことではなく、単に新たな実行ファイルをスピンアップすることになります。

## Part2. コンテナ

### 前提

- docker の ver1.13 以降がインストールされていること.
- Part1 を読んでいること
- 以下のコマンドが動くこと.

   ```sh
   docker run hello-world
   ```

### はじめに、

このパートでは、アプリのレイヤーの最下層である、コンテナを扱います。

### 新しい 'かいはつかんきょう'

以前までは、いざ、Pythonの アプリを書き始めようということがあれば、最初にするべきは、パイソンの実行環境をあなたのローカルマシーンに導入することでした。しかし、それは同時に、期待どおりにアプリが動作する環境をあなたのローカルマシン上に完璧に構築される必要があると言う状況を作り出していました。また、本番環境と適合している必要さえありました。

Dockerを利用することで、イメージとして、ポータブルなPythonのランタイム環境を手に入れることができます。インストールは必要ありません。そしてビルドでは、基本的なPythonのイメージとソースコードを含めて、アプリケーションとその依存先、実行環境すべてを一緒に移動させることができるようになります。

これらのポータブルなイメージは`Dockerfile`によって定義されます。

### `Dockerfile`でコンテナを定義する

`Dockerfile`は、あなたのコンテナの中の環境でなにが起こるかを定義します。ネットワーク・インターフェースやディスクドライブなどのリソースへのアクセスは、システムの他の部分から独立したこの環境の中で仮想化されているため、ポートを外部にマップする必要と、その環境の中にコピーしたいファイルを明確にしておく必要があります。

しかしながら、これらのことをおこなえば、`Dockerfile`の中で定義されたアプリのビルドはどこで動かそうとも、全く同じ振る舞いをすることを期待できます。

#### `Dockerfile` ハンズオン

からのディレクトリを作って、そのディレクトリの中で`Dockerfile`というテキストファイルを作成してください。そして、以下の内容をそのFileの中にコピーアンドーペーストしてください。`Dockerfile`のなかでそれぞれの記述について説明しているコメントについて留意しておいてください。

```docker
# Use an official Python runtime as a parent image
# 親イメージとして、公式に提供されているPythonの実行環境を採用します。
FROM python:2.7-slim

# Set the working directory to /app
# ワークディレクトリを `/app` に設定します。
WORKDIR /app

# Copy the current directory contents into the container at /app
# 現在のカレントディレクトリの内容を コンテナの `/app` ディレクトリへコピーします。
ADD . /app

# Install any needed packages specified in requirements.txt
# requirements.txt で指定されている任意の必要なパッケージをインストールします。
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
# このコンテナの外部の世界に対して、80番ポートを利用可能にします。
# コンテナ側の80番ポートが外部と接続するのに使用可能になるということ.
EXPOSE 80

# Define environment variable
# 環境変数を定義（設定）します。
ENV NAME World

# Run app.py when the container launches
# app.py を コンテナが起動されたときに実行します。
CMD ["python", "app.py"]
```

##### プロキシサーバを利用していますか？

プロキシサーバが起動後に、ウェブアプリケーションへの接続をブロックする可能性があります。もし、プロキシサーバを経由した通信を行っているのなら、以下の行を`Dockerfile`に追加してください。`ENV`コマンドを使って、プロキシサーバーように、hostとportを指定しています。

```docker
# Set proxy server, replace host:port with values for your servers
ENV http_proxy host:port
ENV https_proxy host:port
```

インストールを成功させるために、`pip` コマンドの前に上記の行は追加するようにしてください。

この`Dockerfile`は私達がまだ作成していない、２つのFile、`app.py` と `requirements.txt` について言及しています。これらは次章で作ってみましょう。

### アプリそのもの

以下の２つのファイルを`Dockerfile`を作成したディレクトリに作成しましょう。これで私たちのアプリは完成しました。見ることができるそのアプリは非常にシンプルです。(ココうまく訳せなかった。原文： 'This completes our app, which as you can see is quite simple. '）上記の`Dockerfile`がイメージにビルドされる時、`app.py`と`requirements.txt`は（イメージの中に）含まれます。これは、`Dockerfile`に記述された`ADD`コマンドによるものです。そして、`app.py`は`EXPOSE`コマンドのおかげでHTTPを利用してアクセス可能です。

#### `requirements.txt`

```txt
Flask
Redis
```

#### `app.py`

```python
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```
