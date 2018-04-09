# Rails の 設定情報

## 主な設定ファイルの配置

Rails の 動作は、config フォルダ配下の 各種 の ruby ファイル によって設定できます。基本的な構成をざっと俯瞰します。

- application.rb                : すべての環境で共通の設定ファイル
- routes.rb                     : ルート定義ファイル
- /environment                  : 環境ごとの設定ファイル
  - development.rb              : 開発環境での設定
  - test.rb                     : テスト環境での設定
  - production.rb               : 本番環境での設定
- /initializers                 : その他の初期化処理＆設定情報
  - assets.rb                   : コンパイル対象のアセットを宣言
  - backtrace_silencers.rb      : 例外バックトレースをフィルタ
  - cookies_serializer.rb       : 署名付き・暗号化クッキーに利用するシリアライザー
  - filter_parameter_logging.rb : ロギングから場外するパラメーター情報の条件
  - inflections.rb              : 単数形/複数形の変換ルール
  - mime_types.rb               : アプリで利用できるコンテンツタイプ
  - new_framework_defaults.rb   : Rails5 で デフォルト値が変更になったパラメータ
  - session_store.rb            : セッション保存のための設定情報
- /locales                      : 国際化対応のためのリソースファイル

environment ディレクトリ配下のファイルは、各環境固有の設定情報をそれぞれ表します。この中では、まず`environment/development.rb` を中心に編集を進めてゆきます。

initialize ディレクトリ配下のファイルは、アプリ起動時にまとめてロードされます。デフォルトで用意されている主なファイルは、上のツリー図のとおりですが、必要に応じて自分でファイルを追加することもできます。

設定ファイル・初期化ファイルともに、アプリの起動時に読み込まるので、編集した場合にはWebサーバを再起動するのを忘れないようにしてください。

## 利用可能な主な設定パラメータ 

設定ファイルでは、 `config.パラメータ名 = 値` の形式で、様々な項目を設定することができます。

## アプリ固有の設定を定義する

アプリ固有の設定情報は、 config フォルダ配下に `my_config.yml` のようなファイルを用意してまとめておく方法が推奨されます。
このように定義した設定ファイルは、初期化ファイルで起動時に明示的に読み込んでおく必要があります。

`config/initializers`ディレクトリ配下に、 `my_config.rb` のようなファイルを用意します。

いかがその設定例です。

```ruby
MY_APP = YAML.load(File.read("#{Rails.root}/config/my_config.yml"))[Rails.env]
```

この変数`MY_APP` は、コントローラー、ビューなどすべてのファイルから参照できます。 この変数 `MY_APP` を呼び出す例を記述してみます。

```ruby
class HelloController < ApplicationController
  # 中略 #
  def app_var
    render plain: MY_APP['logo']['source']
  end
end
```

アプリ共通で利用する情報はこのように単一のグローバル変数にまとめておくことで、名前空間を無駄に汚染することなく、また、簡単にアクセスが可能です。

## 所感
今日はscaffold やるやる詐欺マンでしたね。流石に次はやります。
