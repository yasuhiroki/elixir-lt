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

## Streamを調べる

- Stream.filter_map なるものがあるらしい
  - filterしつつmapする
  - map内でIO.Putsすれば行ごとに出力できそう

いけそう


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

- せっかくなら、並列処理してみたい

---

## Flow を試す

```elixir
mix run -e '
  "./file"
    |> File.stream!
    |> Flow.from_enumerable
    |> Flow.filter_map(&(&1 != "\n"), &(String.trim(&1, "\n") |> IO.puts))
    |> Enum.to_list
  '
```

---

## パフォーマンスを比較してみると

空行を含むファイルを270万行にして試す

- `File.read` 版
  - elixir -e   11.95s user 4.71s system 42% cpu 39.058 total
- `Stream.filter` 版
  - elixir -e   11.29s user 1.19s system 35% cpu 34.669 total
- `Stream.filter_map` 版
  - elixir -e   31.92s user 4.71s system 89% cpu 40.943 total
- `Flow` 版
  - mix run -e   46.20s user 14.16s system 121% cpu 49.482 total

- String.trim や IO.puts が重い

---

## IOについて調べてみる

- IO.binstream なるものがあるらしい
  - writeのstream化ができるらしい

いけそう

---

## IO.binstream を試す

```elixir
elixir -e '
  "./file"
    |> File.stream!
    |> Stream.filter(&(&1 != "\n"))
    |> Stream.into(IO.binstream(:stdio, :line))
    |> Stream.run
  '
```

elixir -e   17.37s user 4.29s system 61% cpu 35.068 total

---

## さらにFlowで試...したい

```elixir
mix run -e '
  "./file"
    |> File.stream!
    |> Flow.from_enumerable
    |> Flow.filter(&(&1 != "\n"))
    |> ??????
    |> Flow.run
  '
```

---

## 課題

- Flowでwrite streamingしたかった

---

## Fin

