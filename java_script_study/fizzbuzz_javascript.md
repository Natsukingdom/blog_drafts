久しぶりにfizzbuzz 書いてみました。
# 今回勉強したこと
- for (var i = 0; i < hoge; i++) {...} 構文(まぁ普通のfor 文) 
- for (hoge of piyo) {...} 構文
- JSDoc コメント

# fizzbuzz用のファイルについて
同じ階層に２つのファイルは配置しています。

## fizbuzz.js
```javascript
// fizzbuzz.js
/**
 * 1からcount までの fizzbuzz を 実行するメソッド
 * @param {number} count いくつまでfizzbuzz を行うかを指定する数値.
 * @return {Array} fizzBuzzResult fizzbuzz の結果を 詰めた配列.
 */
function fizzBuzz(count) {
    'use strict';
    let fizzBuzzResult = []; // []で宣言してあげないとコケる.
    // new Array(hoge) という 宣言も可能。その場合、 hoge は int で、 長さが hoge の array を宣言したことになる。
    for (var i = 1; i <= count; i++) {
        if (i % 15 == 0) {
            fizzBuzzResult.push('FizzBuzz');
        } else if (i % 5 == 0) {
            fizzBuzzResult.push('Buzz');
        } else if (i % 3 == 0) {
            fizzBuzzResult.push('Fizz');
        } else {
            fizzBuzzResult.push(i);
        }
    }
    return fizzBuzzResult; // ruby ばっかり書いていて、return を 書くのを 最初は忘れてた。
}

let result = fizzBuzz(100);

for (value of result) {
    console.log(value);
}

for (value of result) {
    document.write(value);
    document.write('<br />')
}

```

## fizzbuzz.html
```html
<!-- fizzbuzz.html-->
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>fizzbuzz</title>
    </head>
    <body>
        <script type="text/javascript" src="./fizzbuzz.js"></script>
        <noscript>JavaScriptが利用できません。</noscript>
    </body>
</html>
```

## 出力
改行入れたせいでとても長くなってしまった。
```
1
2
Fizz
4
Buzz
Fizz
7
8
Fizz
Buzz
11
Fizz
13
14
FizzBuzz
16
17
Fizz
19
Buzz
Fizz
22
23
Fizz
Buzz
26
Fizz
28
29
FizzBuzz
31
32
Fizz
34
Buzz
Fizz
37
38
Fizz
Buzz
41
Fizz
43
44
FizzBuzz
46
47
Fizz
49
Buzz
Fizz
52
53
Fizz
Buzz
56
Fizz
58
59
FizzBuzz
61
62
Fizz
64
Buzz
Fizz
67
68
Fizz
Buzz
71
Fizz
73
74
FizzBuzz
76
77
Fizz
79
Buzz
Fizz
82
83
Fizz
Buzz
86
Fizz
88
89
FizzBuzz
91
92
Fizz
94
Buzz
Fizz
97
98
Fizz
Buzz
```

# 動かし方
clone して chrome で HTML Fileの方を オープンしてみてね.

↓ リポジトリ  
https://github.com/Natsukingdom/JS

# 所感
GitHubにソース上げながらブログ更新するのいいですね。 実際に手を動かしているんで、妄想垂れ流しにならずに済みます。  
あとは書いてないと、記法のルールとかすぐ忘れますね。まぁ、何種類もないので平気です。  
次はクラスとかについてやってい来ます。