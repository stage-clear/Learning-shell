# シェル変数

## 2.1 シェル変数とは

- 変数にセットされる値は、すべて文字列として処理される

```sh
$ variable=value

# = の両側にスペースを入れてはいけない
$ variable = value
```

以下のように、複数の変数を1行で設定できる。この場合セパレータとしてスペースを使っている。

```sh
$ variable1=value1 variable2=value2 ...
```

## 2.2 シェル変数を使ってみる

- シェル変数を参照する場合は変数名の先頭に `$` をつける
- `{}` で囲んでも同じ
- `$` や `{}` はシェル変数としては取り扱われず、これが変数であることを示すことに使われる


```sh
$variable
${variable}
```

`DIR` という変数に、`/tmp` という実際のディレクトリの値をセットし、`cd` コマンドでそれをつかってみます

```sh
$ pwd
/usr/local/home
$ DIR=/tmp
$ cd $DIR
$ pwd
/tmp
$
```

変数の前後に文字を付け加えたいは、`{}` を使う

```sh
$ FOO=abc
$ echo $FOOBAR

$ echo ${FOO}BAR    # <-
abcBAR
$
```

シェル変数として利用できる文字は、アルファベット、数字、`_` の3種類だけなので、これ以外
の文字が前後にある場合は、シェル変数を区切るものとして判断される

```sh
# sample 1
$ pwd
/usr/local/home
$ DIR=/tmp
$ cd $DIR/work       # <-
$ pwd
/tmp/work
$

# sample 2
$ ROOT=main
$ CFILE=$ROOT.c      # <-
$ echo $CFILE
main.c
$
```

`sed` コマンドを使うときなどは、クォーテーションの中に変数を入れる必要がでてきます。
そのときは、普通 `""` で囲みます。`''` は展開しないからです

```sh
$ OLD_TEXT=abc
$ NEW_TEXT=def
$ sed -e "s/$OLD_TEXT/$NEW_TEXT/g" < file > newfile
```

コマンドに変数を展開した値で渡したいなら `''` を使ってはいけない


## 2.3 値がヌルである状態のシェル変数

- シェル変数は、値を何かセットすると同時に定義します


__値は何もない変数の定義__

```sh
VARIABLE=
VARIABLE=""
```


## 2.4 シェル変数の初期設定

- 変数名と代入する値を `=` で結ぶことでセットします

__基本的な代入の形__

```sh
$ VARIABLE=value
$ echo $VARIABLE
```


__柔軟な代入の方法__

```sh
$ echo ${variable:=value}
# `:` は省略可能
```

- `:` がある場合、`Null` か未定義のとき代入される（値が入っていたら代入されない）

このように変数を設定する方法は4通りあります

```sh
${variable:=value}
${variable:-value}
${variable:?message}
${variable:+value}
```

### 2.4.1 `=` によるシェル変数の設定

- 値を代入する

__基本構文__

```sh
${variable:=value}
${variable=value}
```



```sh
$ echo ${ABC:=xyz}
# 変数が未定義か、Null のときに xyz を代入する

$ echo ${ABC=xyz}
# 変数が未定義のときのみ xyz を代入する
```

```sh
                      # 変数 ABC をまだ 1回も使っていないとき
$ echo ${ABC:=xyz}    # この書き方でセットすると
xyz                   # ABC には xyz が代入される
$ echo $ABC           #
xyz
$ echo ${ABC:=abc}    # すでに値がセットされていると
xyz                   # ここで指定したものはセットされない
$ ABC=""              # ABC に Null をセットする
$ echo ${ABC=123}     # コロンを省略する
$                     # Null が入っていたのでセットしない
$ echo ${ABC:=123}    # コロンをつける
123                   # 代入される
$
```


### 2.4.2 `-` によるシェル変数の設定

__基本構文__

```sh
${variable:-value}
${variable-value}
```

- 変数に値を代入せず、指定した値をそのまま返します
- 位置パラメタが使える


例)

```sh
$ echo ${ABC:-xyz}      #
xyz                     # 結果として = を使ったときと同じに見える
$ echo $ABC
$                       # しかし ABC には何も代入されていない
$ echo ${ABC:=abc}      # = を使うと
abc
$ echo $ABC
abc                     # ABC に値が代入される
$
```

```sh
$ echo "The variable ${1:-abc} will be used."
# $1 になんらかの値がセットされていればそれを使い、未定義かNullなら abc を返す
```

### 2.4.3 `?` によるシェル変数の設定

__基本構文__

```sh
${variable:?message}
${variable?message}
```

- 未定義 `Null` を確認するときに使う
- 未定義 `Null` のときに `message` の部分が表示され、さらにシェルスクリプトの場合は
そこで処理を終了します


例)

```sh
$ echo ${ABC:?"ABC is not set"}
```


### 2.4.4 `+` によるシェル変数の設定

__基本構文__

```sh
${varialbe:+value}
${variable+value}
```

- 変数に何らかの値が設定されているときに、値を取り替えて表示します
- 代入せず、元の変数の値は変更しません
- 変数の値を変更せずにそのときだけ結果を変えたい場合に利用します

例)

```sh
$ echo ${ABC:+zzz}
$                     # ABC が未設定なら何もしない
$ ABC=www             # ABC に www という値をセット
$ echo $ABC
www                   # たしかにセットされている
$ echo ${ABC:+zzz}    # + による記述
zzz                   # 値がセットされていたから、指定した zzz を表示
$ echo $ABC
www                   # 実際の変数の値は変わらず
$
```

実際にシェルスクリプトを書く場合には、もっとわかりやすい記述をするのが普通です。

```sh
if [ "$VAR" = "" ]; then
  echo "VAR is not set."
fi

# さらに略すと
if [ ! "$VAR" ]; then
  echo "VAR is not set."
fi
```


## 2.5 位置パラメタ
