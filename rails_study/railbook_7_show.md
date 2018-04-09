# 詳細画面の作成(show アクション)

上記のリポジトリのbooks_controller.rb を参照してください。

## show アクションメソッド

詳細画面について進めてゆきます。

詳細画面は、一覧画面で各行の [Show] リンクをクリックした時に、表示される画面のことです。

### 01.showアクションメソッド

`before_action`メソッドを利用することで、特定のメソッドをフィルターとして動作させることが可能です。

この場合のフィルタというのは、アクションメソッドの前に実行すべきメソッドのことです。複数のアクションで共通する処理は、フィルタとして切り出して`before_action`で指定することで、アクションメソッドの記述をシンプルにできます。それはつまりソースコードをDRYにすることです。

以下は例示です。

```ruby
before_action :hoge, only: [:piyo, :fuga]
# hoge に フィルタとして指定するメソッド名
# piyo, fuga に フィルタを適用するメソッド名を指定する.
# hoge, piyo, fugaの前にコロンを記載しているが、このように、シンボルで指定する。
# また、only ではなく except を使うことで、フィルタを適用しないものを指定することも可能です。
```

そして今回は以下のようなソースコードになっています。フィルタとして指定されている`set_book`メソッドの処理についても読み解きます。解説については、ソースコード内にコメントとして追加します。

```ruby
class BookController < ApplicationController
  before_action :set_book, only: [:show, :edit, :update, :destroy]
  # 中略 #
  private # 単体で呼ばれたくない処理なのでprivate
    # Use callbacks to share common setup or constraints between actions.
    def set_book
      # たとえばshow アクションを呼び出す際のURI は以下のようになります。
      # /books/:id(.:format)
      # この :id の部分を params[:id] で取得しています。
      # そして、Bookモデルのfindメソッドが id をもとに
      # テーブルから検索して @book へと Book のインスタンス を代入するという流れです。
      @book = Book.find(params[:id])
    end
  # 中略 #
end
```

## 所感

ざっくりとした解説になりましたが、このくらいのレベル感で理解にちょうどよくなってきた感じがあります。

サービスを作りたくてこの勉強をしているのですが、ちょっと完全に理解してから開始というのは現実的ではないので、何かしらでとっかかってみようと思います。

文章の創作をしなくなって久しいのですが、たまには書いてみようと最近思います。