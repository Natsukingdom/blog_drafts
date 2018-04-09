scaffolding 機能によるアプリ開発

# Scaffolding機能によるRails開発の基礎

## Scaffolding機能とは

定型的なCRUD(Create-Read-Update-Delete)機能を持ったアプリを構築するための機能です.

`Scaffolding`とは、`足場`という意味です。言葉通り、基本機能を予め実装したアプリの足場を自動作成します.

ただし、現実的には、自動生成したコード群は、随所を修正する必要があるので結局省力化にならない点もあるので一長一短であることを覚えておきましょう。

以下のような場合には、開発生産性や学習効率の向上が望めるかもしれません。

- とりあえず動作するアプリを作れる.
- マスターメンテナンスなど凝ったレイアウトを要求されないページを大量に制作したい.
- Railsによる基本的なCRUD実装を理解したい.

### 01.Scaffolding開発の手順

[1] 一部のファイルを削除する.
前回までに作成したファイルを削除します.前回までの作業で`book`モデル関連のファイルを作成してしまっています。このままでは正しくScaffolding機能を動作させることができません。

そこで今回は作成したmodelのファイルとその関連ファイル,そしてデータベースを削除しておきます.

以下、削除コマンドと標準出力です。

```sh
# Fileの自動削除を行います
❯ rails destroy model book
Running via Spring preloader in process 63447
      invoke  active_record
      remove    db/migrate/20180327151318_create_books.rb
      remove    app/models/book.rb
      invoke    test_unit
      remove      test/models/book_test.rb
      remove      test/fixtures/books.yml
# databaseの削除を行います
❯ rails db:drop DISABLE_DATABASE_ENVIRONMENT_CHECK=1
Dropped database 'db/development.sqlite3'
Database 'db/test.sqlite3' does not exist
```

[2] booksテーブルを操作する関連ファイルをまとめて作成する.
こちらの操作はコマンド一発です.

以下はサンプルです.

```sh
rails generate scaffold name field:type [...] [options]
# name: モデル名, field: フィールド名, type: データ型, options: 動作オプション
```

以下に実行結果を貼ります.

```sh
❯ rails generate scaffold book isbn:string title:string price:integer publish:string published:date dl:booklean
Running via Spring preloader in process 63670
      invoke  active_record
      create    db/migrate/20180331150543_create_books.rb
      create    app/models/book.rb
      invoke    test_unit
      create      test/models/book_test.rb
      create      test/fixtures/books.yml
      invoke  resource_route
       route    resources :books
      invoke  scaffold_controller
      create    app/controllers/books_controller.rb
      invoke    erb
      create      app/views/books
      create      app/views/books/index.html.erb
      create      app/views/books/edit.html.erb
      create      app/views/books/show.html.erb
      create      app/views/books/new.html.erb
      create      app/views/books/_form.html.erb
      invoke    test_unit
      create      test/controllers/books_controller_test.rb
      invoke    helper
      create      app/helpers/books_helper.rb
      invoke      test_unit
      invoke    jbuilder
      create      app/views/books/index.json.jbuilder
      create      app/views/books/show.json.jbuilder
      create      app/views/books/_book.json.jbuilder
      invoke  test_unit
      create    test/system/books_test.rb
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/books.coffee
      invoke    scss
      create      app/assets/stylesheets/books.scss
      invoke  scss
      create    app/assets/stylesheets/scaffolds.scss
```

ほとんどmodel生成のときと同じ構文です.が、ログを見てみると、作成されたFileの数は随分多いです。ただしこれは基本的に今まで随時作成してきたFileをまとめて作成したに過ぎません.

ちなみに上で`boolean`を`booklean`とタイポしていたのでそれを修正しました。`--force`オプションを付けることで

```sh
❯ rails generate scaffold book isbn:string title:string price:integer publish:string published:date dl:boolean --force
Running via Spring preloader in process 64144
      invoke  active_record
      remove    db/migrate/20180331150543_create_books.rb
      create    db/migrate/20180331152934_create_books.rb
   identical    app/models/book.rb
      invoke    test_unit
   identical      test/models/book_test.rb
       force      test/fixtures/books.yml
      invoke  resource_route
       route    resources :books
      invoke  scaffold_controller
   identical    app/controllers/books_controller.rb
      invoke    erb
       exist      app/views/books
   identical      app/views/books/index.html.erb
   identical      app/views/books/edit.html.erb
   identical      app/views/books/show.html.erb
   identical      app/views/books/new.html.erb
       force      app/views/books/_form.html.erb
      invoke    test_unit
   identical      test/controllers/books_controller_test.rb
      invoke    helper
   identical      app/helpers/books_helper.rb
      invoke      test_unit
      invoke    jbuilder
   identical      app/views/books/index.json.jbuilder
   identical      app/views/books/show.json.jbuilder
   identical      app/views/books/_book.json.jbuilder
      invoke  test_unit
   identical    test/system/books_test.rb
      invoke  assets
      invoke    coffee
   identical      app/assets/javascripts/books.coffee
      invoke    scss
   identical      app/assets/stylesheets/books.scss
      invoke  scss
   identical    app/assets/stylesheets/scaffolds.scss
```

上記のようにすることで

[3] マイグレーションファイルを実行する

必要なことの殆どを自動で実行してくれるScaffold機能ですが、マイグレーションによるテーブルの作成だけは`rails db:migrate`コマンドで実行する必要があります.

以下、実行コマンドと出力です.

```sh
❯ rails db:migrate
== 20180331152934 CreateBooks: migrating ======================================
-- create_table(:books)
   -> 0.0012s
== 20180331152934 CreateBooks: migrated (0.0013s) =============================
```

## 02.自動生成されたルートを確認する

Scaffolding機能でアプリを作成すると、`config/routes.rb`にリストのようなコードが追加されます。`rails routes`は`routes.rb`を解析し、現在の有効なルートをリスト表示してくれるコマンドです。以降も頻繁に利用するので覚えておきましょう。

```ruby
# routes.rb
Rails.application.routes.draw do
  resources :books
  # 中略 #
end
```

試しに、`rails routes` コマンドを実行してみます.以下、実行コマンドと出力です.

```sh
❯ rails routes
       Prefix Verb   URI Pattern               Controller#Action
        books GET    /books(.:format)          books#index
              POST   /books(.:format)          books#create
     new_book GET    /books/new(.:format)      books#new
    edit_book GET    /books/:id/edit(.:format) books#edit
         book GET    /books/:id(.:format)      books#show
              PATCH  /books/:id(.:format)      books#update
              PUT    /books/:id(.:format)      books#update
              DELETE /books/:id(.:format)      books#destroy
  hello_index GET    /hello/index(.:format)    hello#index
   hello_view GET    /hello/view(.:format)     hello#view
   hello_list GET    /hello/list(.:format)     hello#list
hello_app_var GET    /hello/app_var(.:format)  hello#app_var
```

`resources`メソッドによって、books(書籍情報)というリソースに対する標準的な操作がまとめてルート定義されるわけです.

### URLパターンに含まれる[:id], [:format]の意味

`resource`メソッドで定義されたURLパターンの`/books/:id/edit(.:format)`に注目して見てゆきます.

この中に登場する`:id`と`:format`のような`:hoge`の部分は変数のプレースホルダになっていて、アクションメソッドに渡される任意のパラメータを指します.また丸括弧で括られている部分は、省略可能であることを示しています.

ココで定義されたURLパターンは以下のようなリクエストURLにマッチして、`books#edit`アクションで処理されるということです.

- `http://localhost:3000/books/108/edit`
- `http://localhost:3000/books/8/edit`
- `http://localhost:3000/books/8/edit.json`

ルートパラメータを取得する方法については後の記事で解説します.

ここではまずルーティングを利用することでURL経由で任意の値を受け渡しできるということを覚えておきます。

## 所感

とりあえず今日はココまで。あしたは、一覧画面の作成とかからやっていきます。くそう！！わからんことたくさん！！！ある！！！！


