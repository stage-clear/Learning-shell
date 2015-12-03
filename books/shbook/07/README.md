# フィルタの使用法

## 7.1 フィルタとは

`cat` コマンドで標準出力にファイルの内容を書き出しフィルタに渡すことでデータに手を加えます
その結果を `while` ループの標準入力としています。


```sh
$ cat file      |
      filter_1  |
      filter_2  |
      while read LINE
      do
        # do something...
      done
```

ファイルをまとめて処理したければ、`cat` の引数に書き並べればいいだけなのでとても便利です

```sh
$ cat file file2 file3... | filter_1 | ...
```

1つのファイルだけの処理なら、リダイレクション機能を使うこともできます

```sh
$ filter_1 < file | filter_2 | ...
```


## 7.2 `sed` コマンド

- `sed` コマンドはフィルタを作るためにはとても有用なコマンドです
- 標準入力からデータを受け取って編集後の結果を標準出力に書き出します


### 7.2.1 `sed` コマンドの基本的な使い方

```sh
$ sed -e "s/OldText/NewText/" samplefile
```
