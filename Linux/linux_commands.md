今日使用して便利に感じたコマンド

# sort

## 説明

並び替えに使うコマンド。正直あまり使うことは多くなかったですが、今日使い始めてみることにしました。ログファイルの並べ替えなど、便利ですね。

以下、`man` というか `sort --help`
```sh
$ sort --help
使用法: sort [OPTION]... [FILE]...
または: sort [OPTION]... --files0-from=F
Write sorted concatenation of all FILE(s) to standard output.

Mandatory arguments to long options are mandatory for short options too. 並び替えオプション:

  -b, --ignore-leading-blanks  先頭の空白を無視する
  -d, --dictionary-order      空白および英数字のみ含まれていると仮定する
  -f, --ignore-case           大文字・小文字を区別しない
  -g, --general-numeric-sort  compare according to general numerical value
  -i, --ignore-nonprinting    consider only printable characters
  -M, --month-sort            compare (unknown) < 'JAN' < ... < 'DEC'
  -h, --human-numeric-sort    人間が読むことができる形式の数値を比較する (例: 2K 1G)
  -n, --numeric-sort          文字列を数値とみなして比較する
  -R, --random-sort           キーのランダムハッシュ順にソートする
      --random-source=FILE    ランダムソースを FILE に設定する
  -r, --reverse               逆順にソートを行う
      --sort=WORD             WORD に応じてソートする。WORD の候補は次の通り:
                                general-numeric -g, human-numeric -h, month -M,
                                numeric -n, random -R, version -V
  -V, --version-sort          自然な (バージョン) 数字順でソートする

そのほかのオプション:

      --batch-size=NMERGE   一度に最大 NMERGE 行、併合を行う。それ以上の場合
                            は一時ファイルが使用される
  -c, --check, --check=diagnose-first  入力がソートされているかを確認する。ソート
                                       は行わない
  -C, --check=quiet, --check=silent  -c と同様だが、正しくソートされていない最初
                                     の行を出力しない
      --compress-program=PROG  PROG を使用して一時ファイルを圧縮し、PROG -d を
                              使用して展開する
      --debug               ソートに使用されている行の一部に注釈をつけて、不確かな
                              使用方法について標準エラー出力に警告を表示する
      --files0-from=F       ファイル F に含まれた NULL 文字で区切られた文字列を
                            ファイル名として扱い、それらのファイルの中身を入力行
                            として読み込む。ファイル F に - を指定した時は、ファ
                            イル名を標準入力から読み込む
  -k, --key=KEYDEF          sort via a key; KEYDEF gives location and type
  -m, --merge               merge already sorted files; do not sort
  -o, --output=FILE         結果を標準出力の代わりに FILE に書き込む
  -s, --stable              前の比較結果に頼らない安定的な並び替えを行う
  -S, --buffer-size=SIZE    主記憶のバッファの大きさとして SIZE を使用する
  -t, --field-separator=SEP フィールド区切り文字として空白の代わりに SEP を使用する
  -T, --temporary-directory=DIR  一時ディレクトリとして $TMPDIR または /tmp ではなく
                              DIR を使用する。オプションを複数指定すると、複数のディ
                              レクトリを指定できる
      --parallel=N          同時に実行するソートの数を N に変更する
  -u, --unique              -c と併せて使用した場合、厳密に順序を確認する。-c を付け
                              ずに使用した場合、最初の同一行のみ出力する
  -z, --zero-terminated     文字列の最後に改行でなく NULL 文字を付加する
      --help     この使い方を表示して終了する
      --version  バージョン情報を表示して終了する

KEYDEF is F[.C][OPTS][,F[.C][OPTS]] for start and stop position, where F is a
field number and C a character position in the field; both are origin 1, and
the stop position defaults to the line's end.  If neither -t nor -b is in
effect, characters in a field are counted from the beginning of the preceding
whitespace.  OPTS is one or more single-letter ordering options [bdfgiMhnRrV],
which override global ordering options for that key.  If no key is given, use
the entire line as the key.

SIZE may be followed by the following multiplicative suffixes:
% はメモリの何 % を使用するか。b は 1倍、K は 1024倍 (標準)。同様に M, G, T, P, E, Z, T
なども指定できます。

*** 警告 ***
環境変数によって指定されたロケールで並び替え順序が変わります。
本来のバイト単位の値を使用した伝統的な並び替え順にしたい場合、
LC_ALL=C を指定してください。

GNU coreutils online help: <http://www.gnu.org/software/coreutils/>
sort の翻訳に関するバグは <http://translationproject.org/team/ja.html> に連絡してください。
完全な文書を参照する場合は info coreutils 'sort invocation' を実行してください。
```

まぁ読んだらオッケーって感じなんですが、かんたんな例を。たとえば、以下のようなファイルが有ったとします.

```sh
> cat sample.txt
12 2 35
22 02 14
22 03 14
12 2 356
```

`-k` オプションのあとに数字をつけると左から数えて何番目かの値を基準に昇順に並べ替えてくれます。

この出力をまず左から一番目の数字で昇順に並べて、次は二番目の数字で昇順に並べて、最後に三番目の数字で昇順にならべて、ということが簡単にできるわけです。

```sh
> cat sample.txt |sort -k123
12 2 35
12 2 356
22 02 14
22 03 14
```

今日はこの使い方が、

```sh
> ls -l |sort -k678
```

という感じで役に立ちました。タイムスタンプによる並べ替えです。

調べてみると

```sh
> ls -lt
```

とか

```sh
> ls -lrt
```

でもできたみたいですね。多分こちらのほうが確実だと思うのでこちらを推奨します。

次は、 `awk` あたりについて勉強してみようと思います。