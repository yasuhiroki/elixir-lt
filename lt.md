name: inverse
layout: true
class: center, middle, inverse
---

# Elixir LT

Yasuhiro Kiyota (@yasuhiroki, @duck_yasuhiroki)

---
layout:false

## 自己紹介

- @yasuhiroki
- 最近の業務は Node.js が中心
  - AWS で Serverless 的なあれこれ
- Rubyが好き、vimが好き、シェルが好き
  - シェル芸はもっと好き
- 関数型言語は未経験

.right[![Right-aligned image](https://avatars2.githubusercontent.com/u/3108110?v=3&s=200)]

---

## 話すこと

- Elixirを楽しいと思った理由

---

template: inverse

# その前に...

---

layout: false

## 自己紹介

- @yasuhiroki
- 最近の業務は Node.js が中心
  - AWS で Serverless 的なあれこれ
- Rubyが好き、vimが好き、シェルが好き
  - **シェル芸はもっと好き** ←
- 関数型言語は未経験

.right[![Right-aligned image](https://avatars2.githubusercontent.com/u/3108110?v=3&s=200)]

---

## シェル芸

> マウスも使わず、  
> ソースコードも残さず、  
> GUIツールを立ち上げる間もなく、  
> あらゆる調査・計算・テキスト処理を  
> CLI端末へのコマンド入力一撃で終わらすこと。  
> あるいはそのときのコマンド入力のこと。  
>> https://blog.ueda.tech/?page_id=1434

---

## シェル芸人の思考回路

課題: 素数を見つけよ

1. 素数かどうか判別するコマンドはあるか
1. 素数の条件を満たす出力を得られるコマンドはあるか

---

## シェル芸人の思考回路

課題: 素数を見つけよ

**素数かどうか判別するコマンドはあるか**

`openssl prime`

```sh
$ openssl prime  2
2 is prime
$ openssl prime  4
4 is not prime

$ seq 10 | xargs -I@ openssl prime @ | grep -v not
2 is prime
3 is prime
5 is prime
7 is prime
```

---

## シェル芸人の思考回路

課題: 素数を見つけよ

**素数の条件を満たす出力を得られるコマンドはあるか**

`factor` (gfactor)

```sh
$ gfactor 2
2: 2
$ gfactor 4
4: 2 2

$ seq 10 | gfactor | awk 'NF==2'
2: 2
3: 3
5: 5
7: 7
```

---

template: inverse

## Elixirを触った時の衝撃

---

## Programming Elixir (p.3) より

> Elixirでの問題の解き方は、Unixシェルと同様だ。  
> コマンドラインユーティリティの代わりに、関数がある。

---

template: inverse

## Functinal |> Concurrent |> Pragmatic |> Fun

---

template: inverse

## !?

---

template: inverse

## |>

---

template: inverse

## |

---

template: inverse

## |>
## |

---

template: inverse

## 完全に一致

---

template: inverse

## 惚れた

---

template: inverse

## シェル芸的な視点で触れる

---

## ファイル空白行を消す

↓これを

```sh
hoge

fuga
piyo


moko
```

↓こうする

```sh
hoge
fuga
piyo
moko
```

---

## シェル芸的発想

### 一撃で条件を満たすコマンドはないか

例) `sed`

```sh
$ cat file | sed '/^$/d'
hoge
fuga
piyo
moko
```

---

## Elixirではどうする?

最低限必要な要素

- ファイルを読み込み
  - 出来れば行思考で読み込む
- 処理
  - 今回は改行 (`\n`) しか無い行を消す
- 出力

---

## ベタに

```elixir
elixir -e '
  # ファイルを読み込む
  "./file"
    |> File.read
    |> elem(1)
  # 改行文字で区切って配列に
    |> String.split("\n")
  # 空文字しか無い要素を取り除く
    |> Enum.filter(&(&1 != ""))
  # 改行で結合
    |> Enum.join("\n")
  # 出力
    |> IO.puts'
```

---

## いまいち

- split して join するのがいまいち
- ファイルを全部読み込んでから処理するのもいまいち

---

## Streamを調べる

- Stream なるものがあるらしい
  - 遅延列挙と言うらしい
  - File.Stream でファイルからStreamを作れるらしい
  - Stream.filter という関数があるらしい

いけそう

---

## Streamで試す

```elixir
elixir -e '
  # ファイルを読み込みつつ空行を取り除く
  "./file"
    |> File.stream!
    |> Stream.filter(&(&1 != "\n"))
    |> Enum.to_list
    |> IO.puts
  '
```

が、まだいまいち。  

- 最終行に空行ができてしまう。
- せっかくなら、読み込めた行から随時出力したい。
  - 全行の読み込みが完了するまで待ちたくない。

---

## Stream.filter_map で試す

```elixir
elixir -e '
  "./file"
    |> File.stream!
    |> Stream.filter_map(&(&1 != "\n"), &(String.trim(&1, "\n") |> IO.puts))
    |> Enum.to_list
  '
```

---

## パフォーマンスを比較してみる

空行を含むファイルを10万行にして試す

- `File.read` 版
  - elixir -e   0.35s user 0.23s system 41% cpu 1.392 total
- `Stream.filter` 版
  - elixir -e   0.59s user 0.20s system 49% cpu 1.601 total
- `Stream.filter_map` 版
  - elixir -e   1.57s user 0.38s system 92% cpu 2.095 total

遅くなっている...

---

## 課題

- マルチプロセスで実行すれば
- パターンマッチを活用した記法も取り入れたい

---

## Fin

