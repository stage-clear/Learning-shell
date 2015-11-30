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


## 6.3 しゅるスクリプトでのオプション処理

```sh
$ command [-f] [-v value]
```

### 6.3.1 getopts コマンドを使う

```sh
FLAG=FALSE
VALUE=
OPT=
while getopts fv: OPT
do
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
