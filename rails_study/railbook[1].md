Ruby On Rails 学習 []
Ruby On Rails 5 アプリケーション・プログラミング を もとにして、 Rails の学習を すすめていく記録です。  

ruby on rails は フルスタックの ruby による web アプリケーションのフレームワークです。  
開発が行われているのは以下のリポジトリです。  
[github.com/rails/rails](https://github.com/rails/rails)

# rails 環境構築
ローカル環境への構築 を やっていきます。下はあくまで手順なんで各位ググってください。

1. rbenv 導入(パスとおっているかどうかに気をつけて)
1. ruby の バージョンの決定(今回は2.4 でやっていく)
1. rbenv で ローカルディレクトリの ruby の バージョンを指定
1. gem install rails で rails を 導入する
1. rails -v で バージョン確認

# 新規アプリ作成
railbook という 名前の サンプルウェブアプリケーションを作成していきます。
カレントディレクトリをアプリケーションを作成したいディレクトリへと移動して以下のコマンドを入力します。  

```sh
$ rails new railsbook
```

構文としては以下になります。  
```sh
rails new appName [options]
```
appName: アプリ名, options: 動作オプション
ちなみに過去のrails の バージョンを使用してアプリを作成したい場合は、以下のようにしてバージョンを指定して、実行可能です。  

```sh
rails _4.2.0_ new appName
```

# お試し起動
標準の HTTP サーバは `Puma` です。本番環境では、用途に応じて様々な選択肢があると思いますが、開発時点では、まずは、標準の `Puma` を利用してゆきます。  
まずは、起動をしてその確認を行ってみます。  
以下のコマンドで起動します。  
```sh
rails server
```

以下のような起動メッセージが確認できれば、 `Puma` は正しく起動できています。  
```sh
> rails server
=> Booting Puma
=> Rails 5.1.5 application starting in development
=> Run `rails server -h` for more startup options
Puma starting in single mode...
* Version 3.11.3 (ruby 2.4.3-p205), codename: Love Song
* Min threads: 5, max threads: 5
* Environment: development
* Listening on tcp://0.0.0.0:3000
Use Ctrl-C to stop
```

サーバー終了用の専用のコマンドはないので、停止する際は `Ctrl + c` で終了してください。
終了の前に実際にページにアクセスしてみましょう。  
ブラウザで `http://localhost:3000/` へアクセスしてみてください。  
`localhost` とは現在アプリが動作しているコンピュータ自身を意味する特別なホスト名です。  
`My Machine` のようなコンピュータ名、または，`127.0.0.1` のような IP アドレスで指定しても構いません。  
`3000` は `Puma` 標準のポート番号です。

# コントローラ
コントローラは MVC のうち、 C の Controller を担うコンポーネントで、個々のリクエストに応じた処理を行います。  

## コントローラクラスの作成
まずは、 `Hello World` とだけ表示させるサンプルを作成し、コントローラークラスの基本を理解します。  
コントローラークラスを作成するためのコマンドは以下です。  
こちらでも、ターミナルから コマンドを入力します。

```sh
rails generatecontroller controllerName [options]
```
controllerName: コントローラー名, options: 動作オプション

### Hello コントローラー
```sh
❯ rails generate controller hello
Running via Spring preloader in process 22626
      create  app/controllers/hello_controller.rb
      invoke  erb
      create    app/views/hello
      invoke  test_unit
      create    test/controllers/hello_controller_test.rb
      invoke  helper
      create    app/helpers/hello_helper.rb
      invoke    test_unit
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/hello.coffee
      invoke    scss
      create      app/assets/stylesheets/hello.scss
```

上記のコマンドで、コントローラークラスが生成されます。また、コントローラーに関係する複数のFileも生成されました。  
具体的には以下のファイル群です。(正確には一つだけ、フォルダが生成されます。)
```sh
app/assets/javascripts/hello.coffee # hello コントローラー固有のCoffeeScripot
app/assets/stylesheets/hello.scss # helloコントローラー固有のSCSS
app/controllers/hello_controller.rb # コントローラークラス本体
app/views/hello # テンプレートの保存フォルダ
app/helpers/hello_helper.rb # コントローラー固有のview ヘルパー
test/controllers/hello_controller_test.rb # コントローラークラスのテストスクリプト
```

#### rails destroy
`rails generate` コマンドを利用して自動生成したFileは、 `rails destroy` コマンドでまとめて削除することもできます。

```sh
❯ rails destroy controller hello
Running via Spring preloader in process 23331
      remove  app/controllers/hello_controller.rb
      invoke  erb
      remove    app/views/hello
      invoke  test_unit
      remove    test/controllers/hello_controller_test.rb
      invoke  helper
      remove    app/helpers/hello_helper.rb
      invoke    test_unit
      invoke  assets
      invoke    coffee
      remove      app/assets/javascripts/hello.coffee
      invoke    scss
      remove      app/assets/stylesheets/hello.scss
```

### コントローラーの 基本構文
実際のコードは以下のようになっています。
```ruby
# hello_controller.rb
class HelloController < ApplicationController
  def index
    render plain: 'Hello World'
  end
end
```

#### Application Controllerクラスを継承している。
自動生成を利用した場合は、そんなに意識せずとも初期状態から継承されているのですが、コントローラークラスは、Application Contorller クラスを継承している必要があります。  

#### 具体的な処理を実装するのはアクションメソッド
1つ以上のの関連するアクションメソッドをまとめたものが、コントローラーです。  
アクションメソッドであることの条件は一つで、public なメソッドであることだけです。

#### アクションメソッドの役割
- リクエスト情報の処理
- モデル呼び出し
- ビューに埋め込むテンプレート変数の設定

今回は、試しに、 アクションから文字列を出力するだけのコードを実装してみます。本来は、コントローラーから出力を直接生成するのは不適切です。あくまでこの方法はデバッグ用途などの例外的な書き方であると理解してください。  
以下、構文です。 
```ruby
render plain: value
```
value: 出力する文字列

## ルーティングの基礎
`ルーティング` とはリクエストURL に 応じて処理の受け渡し先を決定することです。  またその仕組みのことも同様に言及します。  
Rails では、クライアントからの要求を受け取ると、まずはルーティングを利用して呼び出すべきコントローラー／アクションを決定します。  
ルーティング設定は `/config/routes.rb` に定義します。デフォルトでは定義の外枠だけが記述されているので、ここでは、以下のようにルートを追加します。  
```ruby
Rails.application.routes.draw do
  get 'hello/index', to: 'hello#index'
end
```
ルート定義には複数の方法があるが、その中でも、get メソッドはもっともシンプルな方法です。
上記の定義で、 `http://localhost:3000/hello/index` という URL が要求されたら、 hello#index アクションを呼びだしなさいという意味になります。  

```ruby
get 'hoge/piyo', to: 'hello#index'
```
ただし、両者が一致している場合は、 `to` オプションを省略できます。  
```ruby
get 'hello/index'
```
上記はこれまたサンプル動作のためのルート定義になっているため、実アプリではRESTful に実装することに留意しましょう。  

## サンプルの実行
```sh
> rails server
```

`http://localhost:3000/hello/index` にアクセスしてみてください。  
controller の index メソッドが呼び出されて、指定した文字列が画面に出力されたはずです。  

## コントローラーの命名規則
`Rails` の基本的な思想は、 `設定よりも規約` です。(Convention over Configuration).  
Rails 習得の 最初の一歩は、 関連するFileやクラスの 名前付けルールを理解することです。  
- コントローラークラス --> 先頭大文字、接尾辞に `Controller`  
- コントローラークラスのファイル名 --> すべて小文字、単語区切りは、アンダースコア
- ヘルパーファイル名 --> コントローラー名に接尾辞 `_helper.rb`
- テストscript名 --> コントローラー名に接尾辞 `_controller_test.rb`
  
## コントローラの命名
コントローラ名は可能な限りリソースの名前に沿って命名するのが望ましいとされています。  
たとえば、 membersテーブルを操作するコントローラであれば、members コントローラ( `members_controller.rb` ) とするのが望ましい。

# 所感
今日はここまで。  
コントローラークラスまで進んだ。あしたは、 View と Model について一通り触って ActiveRecord の使い方くらいまで行けたらいいなと思います。  
学ぶことはまだまだ多そうです。規約覚えるのしんどい〜。。。
