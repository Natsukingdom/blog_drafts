# 一覧画面の作成

以降では、`Scaffold`で自動生成された個々の画面について、そのコードを読み解いていきます。

## index アクションメソッド

`books#index`アクションは、`/books`で呼び出すことができるいわゆるトップページを生成するためのアクションです。

```ruby
def index
  @books = Book.all
end
```

テンプレートファイルは、`/app/views`ディレクトリの配下に、`controller名/action名.html.erb`という命名形式で配置するのでした。

しかし、`Scaffold`での自動生成時には、`index.html.erb`とはまた別に、`index.json.jbuilder`というFileが存在します。このFileはindexアクションの結果を`JSON`形式で出力するためのテンプレートです。

テンプレートとして、`index.json.jbuilder`を利用するためには、URLを以下のように変えてアクセスします。

```html
http://localhost:3000/books.json
```

表示結果は以下です。

-- 画像貼る

books テーブルの内容がJSON形式で出力されています。このように、Railsでは、与えられた拡張子に応じて、利用するテンプレートを選択して、出力を変更できます。もっといえば`.html.erb`は、 __ERBを使って、HTML形式の出力を生成するためのテンプレート__ を意味していたわけです。

同様に、`.json.jbuilder`は、 __JBuilderを利用して、JSON形式の出力を生成するためのテンプレート__ を表します.JBuilderテンプレートの記法については、以後取り上げます。まず、拡張子.jsonによって、テンプレートファイル`.json.jbuilder`が呼び出されることを理解しておいてください。

### ルート定義

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

ルートを確認すると、このように、`hoge/piyo(.:format)`のようなルートが定義されています。このようにアクションメソッドでは、省略可能な`:format`パラメータを受け取ることで
出力形式を決めていたわけです。

これまで、`http://localhost:3000/books`のように指定できたのは、`:format`パラメータのデフォルト値が`html`だったためです。明示的に`http://localhost:3000/books.html`としても同じ結果を得ることができます。

## index.html.erbテンプレート

books#indexアクションに対応するbooks/index.html.erbのコードです。
<script src="https://gist.github.com/Natsukingdom/937c85cd41a3c9e0cd02beb6965ea95b.js"></script>

今回は、コメントの箇所に注目してみましょう.

### 01.ビューヘルパーを活用する

ビューヘルパーとは、テンプレートファイルを記述する際に役立つメソッドの総称です。ビューヘルパーを利用することで以下の様なことがよりシンプルに記述できます。

- フォーム要素の生成
- 文字列や数値の整形
- エンコード処理

たとえば、ここで利用している`link_to`メソッドは、与えられた引数をもとにハイパーリンクを生成するためのメソッドです。

`<a href="<%= url%><%= text %></a>`のように記述することでも可能ですが、HTMLとスクリプトブロックが混在するのは、コードが読みにくくなる一因となります。避けましょう。

また、ビューヘルパーにはモデルやルートと連携できるなど、Railsとの親和性も高いという特徴もあるので、特別な理由がないならば、出来るだけビューヘルパーを利用するのが望ましいでしょう。

```ruby
# link_to メソッドの記法
link_to(body, url [,html_options])
# body:リンクテキスト, url: リンク先のパス(またはパラメータ情報)
# html_options: <a>要素に付与する属性([属性名： 値]のハッシュ形式)
```

```html
<% = link_to('サポートサイト', 'http://www.wings.msn.to/', class: 'outer', title: '困ったときはこちらへ!') %>
```

としたときは、以下のようなアンカータグが出力されます.

```html
<a class="outer" title="困ったときはこちらへ!" href="http://www.wings.msn.to/">サポートサイト</a>
```

Railsでは`link_to`メソッドの他にも様々なビューヘルパーが用意されています。後日掲載します。

### 02.link_toメソッドでの特殊なパス表記(オブジェクト)

先のindex.html.erbのFileの中に`<%= link_to 'Show', book %>`という記述が存在します。この中の`book`はテンプレート変数`@books`から`each`メソッドによって取り出された1つの`book`のモデルのインスタンスです。

`link_to`メソッドにモデルオブジェクトが渡された場合、Railsはそのインスタンスを一意に表す値、つまり、`book.id`を取得しようとします。

そして、`id`には、`1,2,...`といったような連番がセットされているはずなので、ここでは、リンク先のパスも1,2...となるわけです。そして、現在のパスは、`/books`なので、最終的には`/books/1`といったようなパスが生成されることになります。

### 03.link_toメソッドでの特殊なパス表記（ビューヘルパー)

同じく引数urlに相当する部分で不思議な表記があります。以下のコードに注目してみましょう。

```html
<%= link_to 'Edit', edit_book_path(book) %>
<!-- 中略 -->
<%= link_to 'New Book', new_book_path %>
```

new_book_path, edit_book_path は、 routres.rb で resources メソッドを呼び出したときに自動的に用意されるビューヘルパーです。例えば、現在のルート定義では、以下のようなビューヘルパーが自動的に用意されたことになります。

- books_path : /books
- book_path(id) : /books/id
- new_book_path : /books/new
- edit_book_path(id) : /books/edit

これらのビューヘルパーを利用することで、それぞれ対応するパスを得られるわけです。可読性の観点からもRailsでパス指定する場合には、まずはこれらのビューヘルパーを利用してみるといいでしょう。

### 04.リンククリック時に確認ダイアログを表示する

html_options (<a>要素に付与する属性)には、アンカータグの属性だけでなく、特殊な動作オプションを指定することもできます。以下のコードに注目してみましょう。

```html
<%= link_to 'Destroy', book, method: :delete, data: {confirm: 'Are you sure?' } %>
```

data-confirmオプションを指定すると、link_toメソッドはリンククリック時に確認ダイアログを表示します。不可逆な処理を行うケースでは、確認ダイアログを表示することで、エンドユーザによる誤操作の防止につながります。

#### Unobtrusive JavaScript

控えめな JavaScript

Railsのポリシー。要は、HTMLのあちこちにでしゃばらない --> HTMLからJavaScriptが完全に分離されているという意味です。

### 05.リンク先へのアクセスをHTTP GET 以外で行う

method オプションにも注目してみてください。

```html
<%= link_to 'Destroy', book, mehtod: :delete, data: {confirm: 'Are you sure?' } %>
```

本来、ハイパーリンクによるページ移動では、最も基本的なHTTP GETメソッドを利用します。しかし、methodオプションを利用することでHTTP GET以外でのリクエストを生成できます。

resources メソッドで作成されるルートには、HTTP GET以外に関連づいたものが幾つかあります。そのようなルートにハイパーリンク経由でアクセスするには、methodオプションで明示的に使用するHTTPメソッドを指定する必用があります。

#### 内部的には、HTTP POSTメソッド

もっとも、一般的なブラウザは、DELETE や PUT、 PATCHなどのHTTPメソッドに対応していないため、Railsも内部的には、HTTP POSTで通信を行います。その上で、'HTTP DELETEを利用しようとした'という情報だけを一緒に送信しています。

これによって、Railsは本来あるべきブラウザの姿、RESTfulな思想を緊急避難的に体現しています。

#### 出力されるHTML
GIST

