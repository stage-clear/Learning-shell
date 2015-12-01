# コマンド行の解析、処理

## 6.1 コマンドラインについて

- ここからここまでが1つのコマンドであると区切るのは、改行であったり `;` であったりします
（「[コマンドセパレータ](../01/README.md)」を参照）
- その1つのコマンドを「コマンドライン」と呼びます

```sh
$ mv abc def                # mv がコマンド名。abc と def が引数
$
$
$ command "Hello, world"    # 空白を含む場合はクォーティングする
$
$ echo "part one;" \        # 改行コードを `\` でエスケープする（無意味にする）
$      "part tow."          # ことでコマンドを複数行に分けて書くことができます
```

## 6.2 コマンドラインの書き方

- コマンドはどんなものであれ、コマンドラインにある引数を処理します
- きちんと処理するためにも、次のように並べる規則になっています

```sh
$ command [options] [parameters]
```

- そのコマンドにどのように振舞わせるか、どのように処理させるのかといった、動作に関する
指定を、`option` （オプション）という形で指定します

```sh
$ command -a                  # オプションを指定する
$ command -aOption Value      # オプションに値を指定する
$ command -a Option Value     # オプションに値を指定する
$ command -a -b               # 複数のオプションを指定する
$ command -ab                 # 複数のオプションを指定する（値がなければまとめられる）
$ command -b -a               # 複数のオプション（順序を変えても大抵は問題ない）
$ command -ba                 # 複数のオプション（順序を変えても大抵は問題ない）
$ command -a Option Value -b  # 複数のオプション（値がある場合）
$ command -ba Option Value    # 複数のオプション（値があるものは後ろ）
```

引数にハイフンが含まれる場合は `--` で区切ります。
`--` の後ろは、たとえ `-` がついていてもコマンドに渡す引数となります

```sh
$ command -- -parameter       # `--` の右側は 先頭に `-` があっても引数
$ command -a -- -parameter    # `--` の左側がオプションで右側が引数
```

- __引数は__ 、ほとんどの場合並べる順序が重要になります


```sh
$ cp file1 file2
```

```sh
$ cp file1 [ file2 ... ] directory
```


## 6.3 シェルスクリプトでのオプション処理

```sh
$ command [-f] [-v value]
```

### 6.3.1 getopts コマンドを使う

```sh
FLAG=FALSE              # <- 変数の初期化
VALUE=                  # <- 変数の初期化
OPT=                    # <- 変数の初期化
while getopts fv: OPT   # getopts の次には引数を書く `:` 付きの引数には値が必要となる
do                      #
  case $OPT in
    f)  FLAG=TRUE
        ;;
    v)  VALUE=$OPTARG
        ;;
    \?) echo "Usage: $0 [-f] [-v value]" 1>&2
        exit 1
        ;;
  esac
done
shift 'expr $OPTIND - 1'
```

- `getopts fv: OPT`: `getopts` の次には引数を書く `:` 付きの引数には値が必要となります。
この場合の `fv:` は `command [-f] [-v value]` という意味になります。

```sh
$ command [-abc] [-v value] [-x value]
```

これを処理する場合、次のように書きます。

```sh
getopts abcv:x: OPT
```

`while` 分を抜けた後の `expr $OPTIND - 1` で、位置パラメタ `$1` に引数の最初を指すことになります


### 6.3.2 `getopt` コマンドを使う

- UNIXによっては `getopts` を使えない場合があります。そのときは、`getopt` を使います
- `getopt` を拡張したものが `getopts` なので、両方使える場合は `getopts` を使いましょう

__例) `getopt` を使った例__

```sh
FLAG=
VALUE=
OPT=
set -- `getopt fv: $*`      # `getopt` が受け取れるように位置パラメタ`$*`を展開します
if [ $? != 0 ]; then
  echo "Usage: $0 [-f] [-v value]" 1>&2
  exit 1
fi
for OPT in $*
do
  case $OPT in
    -f) FLAG=TRUE
        shift
        ;;
    -v) VALUE=$2
        shift 2
        ;;
    --) shift
        break
        ;;
  esac
done
```


### 6.3.3 `getopts` コマンドがないとき

- `getopts` コマンドもなく `getopt` コマンドもなかった場合にはどのようにオプションを
処理させればいいでしょうか。正直言ってかなり難しいものです。

1つだけ目をつぶってしまうことにしましょう

```sh
$ command -ab
# という書き方はできません。かならず、
$ command -a -b
```

```sh
while [ $# -gt 0 ]
do
  case $1 in
    -f) FLAG=TRUE
        shift
        ;;
    -v) VALUE=$2
        shift 2
    -v*) VALUE=`echo "$1" | sed 's/^..//'` # `-v value` と `-vvalue` から `value` を抜き取ります
        shift
        ;;
    --) shift # 処理は終わりなので位置パラメタを1つ進めて while を終了します
        break
        ;;
    -*) echo "Usage: $0 [-f] [-v value]" 1>&2
        exit 1
        ;;
    *) break
        ;;
  esac
done
```


### 6.3.4 その他のオプション処理方法

__例) 1つしかオプションを取らない場合__

わざわざ `getopts` を使うほどではない場合

```sh
$ command -v parameter ...
```

```sh
if [ "$1" = "-v" ]; then
  VERBOSE=TRUE
  shift
fi
for param in "$@"
do

done
```

__例) オプションとして1文字ではなく文字列を指定する場合__

```sh
while [ $# -gt 0 ]; then
do
  case "$1" in
    -display) if [ $# -lt 2 ]; then
                echo "Missing parameter."
                exit 1
              fi
              DISPLAY=$2
              shift         # shift 2 という書き方は古いシステムでは使えない場合もある
              shift         #
    -*)       echo "Usage: $0 ....." 1>&2
              exit 1
              ;;
    *)        break
              ;;
  esac
done
```


## 6.4 シェルスクリプトでの引数処理

- オプションが終わったら引数を処理します
- 引数の個数をチェックするのに便利なのは、 `$#` 変数です

```sh
$ copy source target    # 引数がちょうど2つなければならない
```

```sh
if [ $# -ne 2 ]; then   # not equal
  echo "Usage: copy source target" 1>&2
  exit 1
fi
SOURCE=$1
TARGET=$2
```


個数の定まらない引数を処理する場合、それぞれに同様の処理を行えばいいのなら、次のような
`while` 文を利用できます。位置パラメタを `shift` で減らしていっているので、引数がなく
なれば `while` 文も終了します

```sh
while [ $# -gt 0 ]
do
  FILE=$1
  shift
done
```

`for` 分を使った例

```sh
for FILE
do
  #....
  #....
done

# for 文の構文は
for variable in word-list
do
  #....
  #....
done
```

変数に与えるリストを書かなかった場合、for文は位置パラメタの値を順に代入します。
`FILE` という変数に順に代入されていき、なくなればそこで `for` ループを終了します

`while` 文との違いは、`shift` を使っていないことで、そのため、位置パラメタにはそのまま
引数の値が残っています。
位置パラメタを再利用する場合は、`for` 文を使う

また、以下のようにすれば、`LAST` 変数に最後の引数が入ります。

```sh
for LAST
do
  :
done
```
