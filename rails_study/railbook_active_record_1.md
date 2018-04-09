# ActiveRecord の基本

## データ取得の基本

ActiveRecord を 利用する準備ができたところで動作確認も兼ねて簡単なサンプルを作成します。

ここで作成するのは、 books テーブルから すべてのデータを 取得し、一覧表として整形するサンプルです。

### 01. list アクションを追加する

hello コントローラーに対して、 list アクションを追加します。

```ruby:hello_controller.rb
class HelloController < ApplicationController
  # 中略 #
  def list
    @books = Book.all
  end
end
```

ちなみにこの時点で `Book` が指しているのは、 `models/book.rb` のことです。

同ファイルの中身を除くと以下のようになっていることかと思います。

```ruby:models/book.rb
class Book < ApplicationRecord
end
```

殆どからなのですが、このクラスの基底クラスである `ApplicationRecord` （正確にはその基底クラスである`ActiveRecord::Base`）がデータベース・アクセスのための基本機能を提供しているため、、このままでも検索や登録などの操作が可能なのです。当面は自動生成されたモデルに対しては手を加えず、そのまま利用していくことにします。モデルに対して、入力値検証などの独自の機能を実装する方法については別途、他記事で話題にします。

### 02.テンプレートファイルを作成

hello#list アクションに対応するテンプレートFile `list.html.erb` を作成します。テンプレートファイルは、コントローラクラスに対応するように `app/view/hello` ディレクトリに配置するんでしたね。

```html:views/hello/list.html.erg
<table>
    <tr>
        <th>ISBNコード</th><th>署名</th><th>価格</th>
        <th>出版社</th><th>刊行日</th><th>ダウンロード</th>
    </tr>
    <% @books.each do |book| %>
        <tr>
            <td><%= book.isbn %></td>
            <td><%= book.title %></td>
            <td><%= book.price %></td>
            <td><%= book.publish %></td>
            <td><%= book.published %></td>
            <td><%= book.dl %></td>
        </tr>
    <% end %>
</table>
```

オブジェクト配列の内容を準に取り出すのは、each メソッドの役割です。eachメソッドで順番にBookオブジェクトを取り出し、その内容を出力しています。

eachブロックの中では、`book.isbn` という形で オブジェクトの各プロパティにアクセスできます。

### 03.ルート定義を追加する

以上を理解できたら、あとは、ルート定義を追加します。

`config/routes.rb` に `get 'hello/list'` を追加します。

```ruby:config/routes.rb
Rails.application.routes.draw do
  # 中略 #
  get 'hello/list'
end
```

以上の作業が完了したら、 `http://localhost:3000/hello/list` へ アクセスしてみます。

ページ画像

ページソースgist

## 補足：アプリの実行環境を指定する

Rails には development, test, production という３つの実行環境があります。これらを変更する方法はいくつか存在します。最も手軽なのはweb サーバ（今回は Puma) の起動時に指定するというものでしょう。

例えば、以下のように -eオプションを指定することで本番環境でPumaを起動することができます。

```sh
> rails server -e production
```

デフォルトは開発環境になっているので普段は殆ど意識する必要はないと思いますが、本番環境のデータで動作を確認したいなどの用途で利用してください。

ちなみに `rails server` コマンドでは、 `-e` 以外にも以下のようなオプションを指定できます。

```sh
> rails server [name] [options]
# name: 起動するHTTPサーバー
# options: 動作オプション
```

その他のオプションは、 `--help` オプションで見てみましょう。

```sh
❯ rails server --help
Usage:
  rails server [puma, thin etc] [options]

Options:
  -p, [--port=port]                        # Runs Rails on the specified port - defaults to 3000.どのポートで動作させるか
  -b, [--binding=IP]                       # Binds Rails to the specified IP - defaults to 'localhost' in development and '0.0.0.0' in other environments'.特定のIP で動作させる
  -c, [--config=file]                      # Uses a custom rackup configuration.これはわからなかったので調べてみます。。。
                                           # Default: config.ru  デフォルトではこれを見に行っているみたい.
  -d, [--daemon], [--no-daemon]            # Runs server as a Daemon. サーバーをデーモンとして起動する.
  -e, [--environment=name]                 # Specifies the environment to run this server under (development/test/production). 動作環境を指定する.
  -P, [--pid=PID]                          # Specifies the PID file. プロセスID を収納するファイルを指定する.
                                           # Default: tmp/pids/server.pid
  -C, [--dev-caching], [--no-dev-caching]  # Specifies whether to perform caching in development. dev環境でcacheを行うかどうかを指定できる.
```

この中では `-p` コマンドをよく使うことが多そうです。何らかの理由でPumaのデフォルトポート3000 が使用済みであると、 Puma の起動に失敗します。その時は、 -pオプションで以下のように指定してみてください。

```sh
❯ rails server -p 81
```

これで `http://localhost:81/~` ようなアドレスでアプリにアクセスできる様になります。

## 所感

以上です。とりあえず MVC の基本は今日でおしまいです。

次回は `scaffold` を扱ってゆきます。

細かい内容に入っていきますよ〜。がんばるぞ！！