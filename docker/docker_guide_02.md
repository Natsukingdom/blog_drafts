# Part2. コンテナ

## 前提

- docker の ver1.13 以降がインストールされていること.
- Part1 を読んでいること
- 以下のコマンドが動くこと.

   ```sh
   docker run hello-world
   ```

## はじめに、

このパートでは、アプリのレイヤーの最下層である、コンテナを扱います。

## 新しい 'かいはつかんきょう'

以前までは、いざ、Pythonの アプリを書き始めようということがあれば、最初にするべきは、パイソンの実行環境をあなたのローカルマシーンに導入することでした。しかし、それは同時に、期待どおりにアプリが動作する環境をあなたのローカルマシン上に完璧に構築される必要があると言う状況を作り出していました。また、本番環境と適合している必要さえありました。

Dockerを利用することで、イメージとして、ポータブルなPythonのランタイム環境を手に入れることができます。インストールは必要ありません。そしてビルドでは、基本的なPythonのイメージとソースコードを含めて、アプリケーションとその依存先、実行環境すべてを一緒に移動させることができるようになります。

これらのポータブルなイメージは`Dockerfile`によって定義されます。

## `Dockerfile`でコンテナを定義する

`Dockerfile`は、あなたのコンテナの中の環境でなにが起こるかを定義します。ネットワーク・インターフェースやディスクドライブなどのリソースへのアクセスは、システムの他の部分から独立したこの環境の中で仮想化されているため、ポートを外部にマップする必要と、その環境の中にコピーしたいファイルを明確にしておく必要があります。

しかしながら、これらのことをおこなえば、`Dockerfile`の中で定義されたアプリのビルドはどこで動かそうとも、全く同じ振る舞いをすることを期待できます。

### `Dockerfile` ハンズオン

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

#### プロキシサーバを利用していますか？

プロキシサーバが起動後に、ウェブアプリケーションへの接続をブロックする可能性があります。もし、プロキシサーバを経由した通信を行っているのなら、以下の行を`Dockerfile`に追加してください。`ENV`コマンドを使って、プロキシサーバーように、hostとportを指定しています。

```docker
# Set proxy server, replace host:port with values for your servers
ENV http_proxy host:port
ENV https_proxy host:port
```

インストールを成功させるために、`pip` コマンドの前に上記の行は追加するようにしてください。

この`Dockerfile`は私達がまだ作成していない、２つのFile、`app.py` と `requirements.txt` について言及しています。これらは次章で作ってみましょう。

## アプリそのもの

以下の２つのファイルを`Dockerfile`を作成したディレクトリに作成しましょう。これで私たちのアプリは完成しました。見ることができるそのアプリは非常にシンプルです。(うまく訳せなかった。原文： 'This completes our app, which as you can see is quite simple. '）上記の`Dockerfile`がイメージにビルドされる時、`app.py`と`requirements.txt`は（イメージの中に）含まれます。これは、`Dockerfile`に記述された`ADD`コマンドによるものです。そして、`app.py`による出力は`EXPOSE`コマンドのおかげでHTTPを利用してアクセス可能です。

### `requirements.txt`

```txt
Flask
Redis
```

### `app.py`

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

今度は、 `pip install -r requirements.txt` が PythonのFlaskとRedisライブラリをインストールし、アプリケーションが`socket.gethostname()`への呼び出しの出力と同様の環境変数`NAME`をプリントするのがわかります。結局、Redisが動作しない（Pythonのライブラリをinstall しただけであって、Redis自身をinstall していないから）ため、このアプリのRedis呼び出しの動作が失敗し、エラーメッセージを吐くことが予想されます。

>留意事項:コンテナ内でホスト名にアクセスするとコンテナIDが取得されます。これは、実行中のプログラムのプロセスIDと似ています。

以上です。システム内に、Pythonや`requirements.txt`の中に記述されていたものは必要ありません。このイメージをシステム上でビルドしたり、実行したりする必要もありません。まるで、本当はPythonとFlaskの環境構築をしていないようですが、しかし実際にあなたはその環境を手にしています。

### アプリのビルド

アプリをビルドする準備はできています。先ほど作成した新しいディレクトリのトップレベルにカレントディレクトリがあることを確認してください。ここで`ls`を入力した際に、表示されれるべき画面をいかに記述します。

```sh
$ ls
Dockerfile		app.py			requirements.txt
```

今度は、ビルドコマンドを実行しましょう。これはDockerイメージを作成するコマンドです。`-t`オプションを利用することでタグを付けています。こうすることでよりフレンドリーな名前をつけることができます。

```sh
docker build -t friendlyhello .
```

ビルドされたイメージはどこにあるでしょう？それはコンピュータの中のローカルなDocker専用のレジストリの中に保存されています。

```sh
docker image ls
```

#### Linuxユーザのためのトラブルシューティング

##### DNS設定

プロキシサーバが起動した後に、ウェブアプリへの通信を遮断する可能性があります。もしあなたが、プロキシサーバごしに通信を行っているのであれば、以下の行をDockerFileに追加してください。`ENV`コマンドを使用することで、ホストを指定、また、プロキシサーバへの接続ポートを指定することが可能です。

```docker
# Set proxy server, replace host:port with values for your servers
ENV http_proxy host:port
ENV https_proxy host:port
```

#### プロキシサーバ設定

DNSの誤った設定は`pip`に関する問題を引き起こす事があります。`pip`を適切に動作させるために自身のDNSサーバのアドレスを設定する必要があります。DockerデーモンのDNS設定を変更したいと思うかもしれません。`/etc/docker/daemon.json`で`dns`キーを用いて設定ファイルを記述することができます。以下のように書くことができます。

```json
{
  "dns": ["your_dns_address", "8.8.8.8"]
}
```

上の例では、配列の最初の要素がDNSサーバーのアドレスです。二つ目の要素は最初の要素が使えなかったときのためのGoogleのDNSです。

手順を続ける前に、daemon.jsonを保存して、dockerサービスを再起動してください。

```sh
sudo service docker restart
```

いったん修正したら、`build`コマンドを再度動かしてみてください。

### アプリを動かす

アプリを動かします。ローカルホストの4000番ポートをコンテナが公開している80番ポートを`-p`オプションを使用して接続してみましょう。

```sh
docker run -p 4000:80 friendlyhello
```

あなたは、`http://0.0.0.0:80`でPythonがアプリをサービスインさせています、といったメッセージを目にするはずです。しかし、このメッセージはコンテナの内部からやってきたものです。コンテナはあなたが80番ポートを4000番ポートへ接続したことを認識していません。URLを`http://localhost:4000`と修正しましょう。

そのURLへアクセスしてウェブページにアップされているコンテンツを確認してみてください。

!!画像を貼る

> 留意事項:もしあなたがWindows7上でDockerToolBoxを使っているのであれば、`localhost`の代わりに、DockerMachineのIPアドレスを使用してください。例えば、`http://192.168.99.100:4000/`といったアドレスです。これを確認するためには、`docker-machine ip`コマンドを利用します。

`curl`コマンドをshellから利用しても同じ内容を見ることができます。

```sh
$ curl http://localhost:4000
<h3>Hello World!</h3><b>Hostname:</b> 88f12064cd94<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>
```

この`4000:80`のポートリマッピングは`Dockerfile`内部で`EXPOSE`(さら)したものと`docker run -p`を利用して`publish`(公開)したものの違いを実証しています。(うまく訳せず。)後述する手順では、ホストコンピューターの80番ポートをコンテナの80番ポートにマッピングして、`http:localhost`を利用します。

終了するために、`CTRL+C`をターミナルで入力してください。

> __Windows上での明示的コンテナの停止__
>
> WIndowsのシステム上では、`CTRL+C`はコンテナを停止させません。したがって、最初の`CTRL+C`はプロンプトを戻すためのものです(あるいは、別のシェルをオープンする）。それから`docker container ls`を入力して稼働中のコンテナをリストアップしいます。そして、`docker container stop <Container NAME or ID>` をコンテナを停止するために入力します。そうしなければ、次の手順で再度コンテナを動かそうとしたときに、デーモンからエラーのレスポンスが返ってくることでしょう。

今度は、バックグラウンドでアプリを動かしてみましょう。detached mode で動かします。

```sh
docker run -d -p 4000:80 friendlyhello
```

アプリの長いコンテナIDがターミナルに吐き出されるのが見られるでしょう。コンテナは今、バックグラウンドで稼働しています。`docker container ls`をすることで、省略されたコンテナIDを確認することもできます。（コマンドの実行時にはどちらも互換的に動作します。）

```sh
❯ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                  NAMES
d34700db24bb        friendlyhello       "python app.py"     13 seconds ago      Up 8 seconds        0.0.0.0:4000->80/tcp   determined_clarke
```

`CONTAINER ID`が`http://localhost:4000`のものと一致することに留意してください。

今度は、`docker container stop`をプロセスを終了するために使用します。`CONTAINER ID`を使用します。以下のようにです。

```sh
docker container stop d34700db24bb
```

### Imageを共有する

