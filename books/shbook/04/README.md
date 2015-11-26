# リダイレクションによるファイル操作

## 4.1 ファイルディスクリプタ

- プロセスとそれが使用するファイルとを結びつけています

|ファイルディスクリプタ|意味|
|:--:|:--|
|`0`|標準入力（普通はキーボードからの入力）|
|`1`|標準出力（普通は端末への出力）|
|`2`|標準エラー（普通は端末への出力）|


## 4.2 リダイレクション

- 普通の入力元や出力先を変えることをリダイレクションといいます
- リダイレクションとは、方向を変える、宛先を変えるという意味を持つ
- リダイレクト記号としては、`>` と `<` を基本的に用います

|リダイレクト記号|意味|
|:-:|:--|
|`>`|出力先をリダイレクト（左辺のデフォルトは `1>` ）|
|`<`|入力元をリダイレクト（右辺のデフォルトは `<0` ）|
|`>>`|出力先をリダイレクトし追記する|
|`a>&b`|`a` と　`b` を一緒にまとめる（ `2>&1` ）|

|記述法|説明|
|:--|:--|
|`>file`|標準出力を `file` というファイルに書く|
|`>>file`|標準出力を `file` というファイルに追加書きする|
|`>&m`|標準出力を `m` 番のファイルディスクリプタに書く|
|`>&-`|標準出力をクローズする|
|`<file`|標準入力を `file` というファイルから読み込む|
|`<&m`|標準入力を `m` というファイルディスクリプタから読み込む|
|`<&-`|標準入力をクローズする|
|`<<word`|標準入力をヒア・ドキュメントから読み込む|

（デフォルトのファイルディスクリプタ（`1` `0`）は省略）

__注意点__

- 指定できるファイルディスクリプタの番号は `0` から `9` まで
- リダイレクト記号とファイルディスクリプタ番号との間にスペースを空けない


__例) リダイレクトする__

（現在の作業ディレクトリには何もファイルがないものとする）

```sh
$ echo abcdefg        # echo は「標準出力」に書く
abcdefg               # 結果は画面に表示される
$ echo abcdef > abc   # 出力をファイル `abc` に向ける
$                     # 画面には表示されない
$ cat abc             # ファイル `abc` に `echo` した内容が書き込まれた
abcdef
$
```

__例) リダイレクトした出力先に追加書きする__

```sh
$ echo  123456 > abc  # 別の出力を書き込んでみる
$ cat abc             # abc の内容をみる
123456                # 結果は新しく書き換わった
$
$ echo xyz >> abc     # >> というリダイレクト記号を使うと
$ cat abc             # 結果は追記されている
123456
xyz
```

__例) 標準出力の向きだけを変える__

```sh
$ cat abc xxx         # `abc` と `xxx` というファイルを cat する
123456                # `abc` の中身は表示されるが、
xyz                   # `xxx` とういファイルは存在しないのでエラー
cat: xxx: No such file or directory
$ cat abc xxx > nnn   # 同じことをリダイレクトすると
cat: xxx: No such file or directory # エラーだけが画面に出た
$                     # `nnn` というファイルには abc の中身が書かれました
```

__例) エラーだけをリダイレクトする__

```sh
$ cat abc xxx 2> nnn    # 「標準エラー」の2番をファイル `nnn` に向ける
abcdef                  # `abc` の内容は標準出力に書かれる
xyz                     # このように画面に表示される
$
$                       # `nnn` というファイルにはエラーが書かれました
```

__例) 標準出力と標準エラーをひとまとめにリダイレクトする__

```sh
$ cat abc xxx > nnn 2>&1
```

複数のリダイレクト記号が1行にあるときは「左から右」に判定される


__例) 標準入力をリダイレクトする__

キーボードからの入力するのではなくファイルを読ませてあたかもキーボードから入力したかのよ
うに見せかける。
`$ ls -l abc nnn` と書かれたファイル `xyz` を用意する。

```sh
$ cat xyz
ls -l abc nnn
$ sh < xyz              # `xyz` ファイルを「sh の標準入力」としてリダイレクトする
-rw-r--r--  1 nepdigital  staff   7 11 13 18:44 abc
-rw-r--r--  1 nepdigital  staff  43 11 13 18:51 nnn
$                       # 結果は `ls -l abc nnn` を入力したときと同じ
```

__例) あるコマンドの実行結果やエラーをすべて捨てる__

```sh
$ command > /dev/null 2>&1
$
# 誤った書き方
$ command 2>&1 > /dev/null
```

__例) パイプでの標準出力の受け渡し__

```sh
$ command1 | command2
# command1 の標準出力を command2 の標準出力に渡す
# 次のように書いたものと同等
$ command1 > file
$ command2 < file
```


## 4.3 リダイレクトを使った書き込み `>` `>>`

- コマンドの出力結果は普通端末画面に出るようになっている
- リダイレクトすることでファイルに結果を書くことができる
- ファイルが存在していなかった場合は新規作成される


__例) 標準出力をファイルにリダイレクト__

```sh
$ command > file      # ファイルを上書き
$ command >> file     # ファイルに追記
```

__例) 標準エラーをファイルにリダイレクト__

```sh
$ command 2> file     # 上書き
$ command 2>> file    # 追記
```

__例) ファイルディスクリプタを番号を明示的に指定__

```sh
$ command n>&m
```

__例) `echo` の出力先を標準エラーに向ける__

`echo` は標準出力にメッセージを表示します。`echo` を使ってエラーメッセージを表示する場合
標準エラーにリダイレクトする

```sh
$ echo "Error messages." 1>&2
```

__例) ファイルにファイルディスクリプタ番号を割り当てて書き込む__

```sh
$ echo abcdef 1>&8              # 適当に番号 8 を指定してみる
-bash: 8: Bad file descriptor   # 使用されていないのでエラー
$ exec 8> hhh                   # 8番を使って hhh というファイルへ書き込む
$ echo abcdef 1>&8              # これで hhh というファイルに結果が書かれる
```

__例) 実行するたびに初期化して書き込む__

```sh
$ command > file      # 最初に上書きモードで書き込めば必ず初期化される
$ command >> file     # 以降は追記する
$ command >> file
```


## 4.4 リダイレクトを使った読み込み `<` `<<`

- 標準入力もリダイレクトできます
- キーボードからではなく、ファイルから読み込むことができる
- 指定したファイルが存在しなかった場合はエラーが出力される


__例) ファイルから入力__

```sh
$ command < file
```

__例) そこに書いてあるテキストをそのまま入力__

- これは「[ヒア・ドキュメント](#here_documents)」という

```sh
$ command << word
# ...
# ...
word
```

__例) 任意のファイルディスクリプタの番号を別のもに変える__

```sh
$ command n<&m
```


## 4.5 制御構造文とリダイレクション

- リダイレクション機能は__サブシェル__上で動作する
- `if` `while` で制御構造文全部をリダイレクトすろときは注意が必要

__グルーピングしたコマンドに対して:__

```sh
{
  command
  #......
} > stdout
```

__`if` 文:__

```sh
if command-list
then
  command
  #....
fi > stdout
```

__`for` 文:__

```sh
for variable [ in word-list ]
do
  command
  #.....
done > stdout
```

__`while` 文:__

```sh
while command-list
do
  command
  #.....
done > stdout
```

__`case` 文:__

```sh
case string in
  pattern) command-list ;;
  #.....
esac > stdout
```

- `stdout` は任意のファイル名に置き換える
- 標準エラーをリダイレクトする場合は `2> stderr`
- 標準入力から読み込む場合は `< stdin`


__例) コマンドの結果を `while` に渡し `while` の結果をコマンドに渡す__

```sh
command |
while command-list |
do
  command
  #....
done |
command
```


## 4.6 `exec` コマンドとリダイレクション

- `exec` は現在のシェルのプロセスをそのまま置き換えて新しいコマンドを実行させるものです
- `exec` に引数を与えず、リダイレクト記号だけを書くことで現行のシェルに対してリダイレク
ト処理を行わせることができます


__例) 基本的な書き方__

```sh
$ echo abc              # 普通の状態
abc
$ exec > /dev/null      # シェルの標準出力を `/dev/null` にリダイレクトする
$ echo abc              # 出力がリダイレクト先に行った
$ cat noneist           # 存在しないファイルをオープンしてみる
cat: nonexist: No such file or directory # 標準エラーは出力される
$
```

__例) `exec` を使ったシェルの入出力のリダイレクションの用法__

```sh
$ exec < file           # file から入力する
$ exec 1>&2             # 標準出力を標準エラーに向ける
$ exec 2> /dev/null     # 標準エラーを捨てて画面に出さないようにする
$ exec > /dev/null 2>&1 # 標準出力と標準エラーをすべて捨てる
$ exec >> file          # 標準出力を file に追加書きする
```

__例) `exec` を使ってファイルをオープンする__

```sh
$ exec n> file          # 上書きモード
$ exec n>> file         # 追記モード
```

`file` を入力用にオープンし、ファイルディスクリプタ `n` 番から読み取る場合

```sh
$ exec n< file
```

ヒアドキュメントをオープンし、`n` 番で読み込む場合

```sh
$ exec n<< word
#.....
#.....
word
```


## 4.7 ファイルからの読み込み

- `read` は入力を読み込んで変数にセットする
- `read` は改行コードまでを読み取るので複数行あっても意味がない
- `read` を `while` で使用すると、ファイルから1行ずつ読み取り処理する

```sh
#!/bin/sh

while read LINE
do
  echo $LINE
done < file
```

__例) while 内での変更を現行シェルに渡す__

- `while` はサブシェル上で動作する
- `exec` を使って現在のシェルの入力ファイルからリダイレクトする

```sh
exec < file
while read LINE
do
  #.....
done
```

これでは、キーボードからの操作が一切できなくなるのでそれを対処する

```sh
exec 3<&0 < file
while read LINE
do
  echo $LINE
done
exec 0<&3 3<&-

# 説明:
# 一時的にファイルディスクリプタ3番を標準入力からにし標準入力をファイルからの入力にする
# while 処理後、標準入力を3番に（キーボード入力に戻）し、3番を閉じる
```

もしくは、ファイルからの読み込みを一時的な番号を使って行う方法も考えられる

```sh
exec 3< file
while read LINE 0<&3
do
  echo $LINE
done
exec 3<&-
```


__例) IFSを一旦クリアして戻す__

```sh
OLDIFS=$IFS
IFS=
while read LINE
do
  echo "$LINE"
done
```


## 4.8 リダイレクションのクローズ

```sh
$ exec >&-
$ exec <&-

# ファイルディスクリプタ番号を指定する場合
$ exec n>&-
$ exec n<&-

# コマンドの出力を捨ててしまいたい場合は `/dev/null` に向ける
$ command > /dev/null
```

## 4.9 ファイルのゼロリセット

- ファイルの大きさを `0` にしたいときや、なにも書かれていないファイルを作成したい場合

```sh
$ > file            # 何もしなかった結果を file に書く
$ : > file          # なにもしないコマンドの結果を file に書く
```

`/dev/null` を使う方法もよく使われます

```sh
$ cat /dev/null > file
$ cp /dev/null > file
```


## 4.10 ヒア・ドキュメント (Here Documents)

- あるコマンドに渡すキーボードからの入力文字列を、そのコマンドの直後に指定することができます


__基本的な書き方__

```sh
command << END
> ....
> ....
> END

# ヒアドキュメント内の変数をクォートする場合
command << \END     # 'END' でも可
> ....
> ....
> END

# 行頭のタブを無視する
command <<- END
> <tab>....
> <space><space>....
> END
....
  ....
```

`.....` の部分が `command` の入力として扱われます

__例) `cat` コマンド__

```sh
$ cat
This is a
This is a
input data.
input data.
$
```

これをヒアドキュメントで書き換える

```sh
$ cat << END
> This is a
> input data.
> END
This is a
input data.
```

入力のデータの中に `$` が入ったらどうなるか

```sh
$ cat << END
> This is a
> here $DOC.
> END
This is a
here .            # <- DOCに値がない
```

```sh
$ DOC=document    # DOCに値を入れる
$ cat << END
> This is a
> here $DOC.
> END
This is a
here document.     # <-
$
```

`$DOC` という文字列を表示したい場合 `\` でクォートする

```sh
$ DOC=document
$ cat << \END      # <-
> This is a
> here $DOC
> END
This is a
here $DOC.
```

- `\$DOC` と変数だけをクォートしてもかまわないが、変数が多いとわかりにくくなる
- `\` でなく、シングルクォーテーションでも同じことになる。 `'END'`


行頭のタブ（スペースではなく）を無視する

```sh
$ cat <<- END
> <tab>This is a
> <space><space>here documents.
> END
This is a             # <- 行頭のタブは無視されます
  here documents.
```

インデントを使って書くときに `-` を用いなければ、ヒアドキュメントの部分は必ず左寄せしな
ければならないため、少し見づらくなってしまいます
`-` を用いると以下のように書けます

```sh
if .....
then
  command <<- END
    This is a here documents.     # This の前にタブがあります
  END                             # END の前にはタブがあります
fi
```


__例) 関数 Usage 内の `cat` の出力を標準エラーに向ける__

```sh
Usage() {
  cat 1>&2 <<- EOF
    Usage: $0 [-options] [etc...]
    ...
    ...
  EOF
}
```

__例) 入力はヒアドキュメントから読み取り、標準出力やエラー出力をファイルにリダイレクト__

```sh
command << EOF >stdout 2>stderr
....
EOF
```

パイプに渡すことも可能

```sh
command << EOF | command2
...
...
EOF
```


__例) `ed` コマンドを使った例__

- `ed` は、`-` オプションを指定すると画面にメッセージを出力しません

```sh
# 読み込んだファイルを逆さまに並べ替える
ed - file <<- !
g/^/m0
w
q
!
# 説明:
# - ed で file というファイルをオープンします
# - ここから ! の行までがヒアドキュメントとなります
# - `g/^/m0`
#   - g でこのファイルの中で一致するパターンをさがします
#   - `^` は行頭を意味します。すべての行頭で処理を行います
#   - `m0` は m は move で、0行目に移動します
#   - 結果全部逆さまになります
# - w コマンドで上書き保存します
# - q コマンドで ed コマンドを終了します
```


## 4.11 リダイレクションの使用に関する注意と例

__コマンドにリダイレクションを使用する場合、それぞれのコマンドごとに指定しなくてはなりません__
あるコマンドに対するリダイレクトは他のコマンドには影響を与えません

```sh
command1 2>ErrorFIle1 | command2 2>ErrorFile2
# command1 のエラーと command2 のエラーは別物であり、
# コマンドごとにリダイレクトする必要があります
```

__標準入力と標準出力を、同じファイルにリダイレクトしてはいけません__

```sh
$ echo "foo" > file
$ sed 's/foo/bar/' < file > file  # 入力と出力が同じファイルではいけない
$ cat file                        # file には何も入らない
```

__例) パイプを使った使用例__

```sh
DEVICE=lp1
# DEVICE=lp2
# DEVICE=lp3
pr $* |
  case $DEVICE in
    lp1) rsh host1 lp -dlp1 ;;
    lp2) rsh host2 lp -dlp2 ;;
    lp3) rsh host3 lp -dlp3 ;;
  esac
```


__例) while のネスト__

```sh
while read FILENAME
do
  while read LINE
  do
    ....
  done < $FILENAME
done < file
```

__例)__

```sh
TTY=`tty`         # TTY には `/dev/ttyp3` といった端末デバイスが入る
while read FILENAME
do
  echo "Do you want to purge $FILENAME ?"
  read ANSWER < $TTY
  .....
done < file
```
