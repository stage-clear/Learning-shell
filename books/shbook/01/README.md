# 書き方にかかわる基本的な説明

## 1.1 シェルスクリプトを作る際の基本事項

`#!` に続くコマンドが宣言となる。

```sh
#!/bin/sh
```

- [サンプル) あるファイルを別のファイルとくっつける](append)

```sh
$ append abc xyz
-bash: append: command not found

$ ./append abc xyz
-bash: ./append: Permission denied

$ ls -l
-rw-r--r--  1 nepdigital  staff  117 11  2 11:49 append
```

`append` ファイルのモードが `-rw-r--r--` となっているところに注意する
これは「作った本人は読み書きができ、他の人はみることだけができるファイル」という意味で、
「実行する許可がない」というエラーです。
シェルスクリプトはコマンドと同じように「実行できる」ようにしておかなくてはなりません。

```sh
$ chmod +x append
$ ls -l append
-rwxr-xr-x  1 nepdigital  staff  117 11  2 11:49 append
```

再度、実行します。

```sh
$ append abc xyz
$
$ ls -l
-rw-r--r--  1 nepdigital  staff     4 11  2 11:44 abc
-rwxr-xr-x  1 nepdigital  staff   117 11  2 11:49 append
-rw-r--r--  1 nepdigital  staff     8 11  2 11:58 xyz
```

`xyz` のファイルの大きさが増え、`abc` ファイルの文字数を加えたものになっている。

```sh
$ append FROMFILE TOFILE
```

とタイプすればいいということになります。

### まとめると

- シェルプログラムの先頭には、`#!/bin/sh` を書く
- シェルプログラムには、実行権限 （`x` フラグ）を与える


* `append: not found` というエラーになった場合は、コマンドのサーチパス（PATH変数）がセ
ットされていません。その場合は、 `$ ./append abc xyz` と実行してください。
