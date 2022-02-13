# 第1章

```sh
rmdir {{ directory-name }}

# 所有者 所有グループ 所有グループ以外
chmod -r {{ file-name }}
chmod -w {{ file-name }}
chmod +r {{ file-name }}
chmod +w {{ file-name }}
chmod 640 {{ file-name }}

whoami
sudo whoami

$ man ls | grep -A 1 '^ *-a'
       -a, --all
              do not ignore entries starting with .

echo 'クロロエチルエーテル' | sed 's/エチル/&&/'
echo 'クロロメチルエチルエーテル' | sed -E 's/(メチル)(エチル)/\2\1/'

seq 100 | grep '[02468]$' | xargs
seq 100 | grep '[^02468]$' | xargs
seq 100 | grep -E '^(.)\1$' | xargs

# grepの-oオプションはマッチそのものを、別行に分けて出力する
echo 中村 山田 田代 上田 | grep -o '[^ ]田'
# -oオプションを使わない別海
# trはアルファベットに対してなら使いやすい原始的な置換
echo 中村 山田 田代 上田 | tr ' ' '\n' | grep '田$'

# awkは「grepにプログラム機能をつけたもの」と理解すると良い
seq 5 | awk '/[24]/'

seq 5 | awk '$1%2 == 0'
seq 5 | awk '$1%2 == 0 { printf("%s 偶数\n", $1) }'
seq 5 | awk '$1%2 == 0 { print($1, "偶数") }'
seq 5 | awk '$1%2 == 0 { print $1, "偶数" }'

seq 5 | awk '$1%2 == 0 { print $1, "偶数" } $1%2 == 1 { print $1, "奇数" }'
seq 5 | awk 'BEGIN { a = 0 } $1%2 == 0 { print $1, "偶数" } $1%2 == 1 { print $1, "奇数" } { a += $1 } END { print "合計", a }'

# sort, uniq -cの典型パターン
seq 5 | awk '{ print $1%2 ? "奇数" : "偶数" }' | sort | uniq -c | awk '{ print $2, $1 }'
# sortの-n, -kオプション
# -kオプションはキー列の指定、-nオプションは数値としての解釈を指定する
# nに与える数値は、基本は -kx,xn として、1列分だけを覚えるのがまずは良さそう
seq 5 | awk '{ print $1%2 ? "奇数" : "偶数" }' | sort | uniq -c | awk '{ print $2, $1 }' | sort -k2,2n
# awkだけでやる
seq 5 | awk '{ print $1%2 ? "奇数" : "偶数" }' | awk '{ a[$1]++ } END { for(k in a) print k, a[k] }'

# xargsは「コマンドに引数を渡して実行してもらう」もの
# 左から渡ってくるものは、「入力ではなく、引数として」であることに注意する
seq 4 | xargs mkdir
ls -d ? # 1文字のファイルやディレクトリを指す
seq 4 | xargs rmdir
# -nオプション、指定した数値の個数分ごとに次のコマンドの引数にする
mkdir 1 3
seq 4 | xargs -n2 mv # mv 1 2, mv 3 4 というコマンドが実行される
# -Iオプション、次に与えた文字をプレースホルダとする
seq 4 | xargs -I@ echo @

# bashによるメタプログラミング
# パイプでワンライナーを渡してもよいし、シェルスクリプトファイルを作って bash ./a.sh という実行の仕方でも良い
seq 4 | awk '{ print "mkdir " ($1%2 == 1 ? "odd_" : "even_") $1 }' | bash
seq 4 | awk '{ if ($1%2) { a = "odd_" } else { a = "even_" }; print "mkdir " a $1 }'

# findをパイプすることでgrepでファイル名を検索するテクニック
find . | grep {{ 検索パターン }}
```

## 問題1

```sh
# ドットをエスケープする
cat files.txt | grep '\.exe$'
# 手グセでawkを使うのも良さそう
cat files.txt | awk '/\.exe$/'
```

## 問題2

ImageMagickというソフトが画像変換に便利なコマンドを提供する。

```sh
sudo apt install imagemagick

# lsの結果はパイプすると行ごとに入力される
ls *.png

# fileコマンドはファイルの種類を特定するコマンド
file 10_black.png

# xargsの-I@はイディオムと考える（@を変数と考えるととりあえずつけるでも困らないかも）
# sedで一致部分を消すって発想が中々出てこないので、なじませたい
ls *.png | sed 's/\.png$//' | xargs -I@ convert @.png @.jpg

# timeコマンドで時間を計測する
# xargsの -P2 オプションでコアを2つ使って並列処理する
time ls *.png | sed 's/\.png$//' | xargs -P2 -I@ convert @.png @.jpg
# nprocコマンドはコア数を出してくれるので、以下のようにすると最適かも
time ls *.png | sed 's/\.png$//' | xargs -P$(nproc) -I@ convert @.png @.jpg

# parallelというコマンドを使っても行けるっぽい
```

## 問題3

- `ls -U` はソートしないためはやい

```sh
# renameコマンドはsedのような表記（perlのような表記）でファイル名を変更するコマンド
sudo apt install rename

# 最初の置換で0を7個つける
# 次の正規表現で後ろの7桁の数字をキャプチャする
ls -U | xargs -P2 rename 's/^/0000000/;s/0*([0-9]{7})/$1/'

# 以下はよくわからなかったが、やはりsedで余分な部分を削除する、というのは典型パターンっぽい
find . | sed 's;^\./;;' | grep -v ^\\.$ | grep -v ^0 | awk '{ print $1, sprintf("%07d", $1) }'
find . | sed 's;^\./;;' | awk '{ print $1, sprintf("%07d", $1) }'

# seqのwオプションは単体で便利そう
# sedパートで実行コマンド文字列を作っている
# sh -c, bash -cの後に渡した文字列をシェルスクリプトとして実行させられる
seq -w 100 | awk '{ print $1, $1 }' | sed 's/^0*/mv /' | xargs -P2 -I@ sh -c @
```

## 問題4

`grep -l, rg -l` はファイル名だけ出す典型オプション。

```sh
# コマンド文字列 | bashでのコマンド実行も典型、ご安全に
# $RANDOMは2^15までの乱数を手軽に生成できる環境変数、zshにもあるっぽい
seq 10000 | sed 's/^/echo $RANDOM > /' | bash

# rgはデフォルトで並列実行してくれるから早いらしい
rg -l '^10$' | xargs rm
```

## 問題5

```sh
# awkは空白区切りで考えると柔軟な条件によって行抽出が可能になる
cat ./qdata/5/ntp.conf | awk '$1 == "pool"'

# パイプしてもいいし、そのままアクションに書いても良い
cat ./qdata/5/ntp.conf | awk '$1 == "pool"' | awk '{ print $2 }'
cat ./qdata/5/ntp.conf | awk '$1 == "pool" { print $2 }'
```

## 問題6

awkのfor文が基本。
seqの特殊なオプションとか、printfまで使いこなせると尚良。

awkはコマンド部分でセミコロンをつなげれば、1行あたりに対して複数の命令が実行できる。

```sh
# tacもたまには役立つかも
seq 5 | awk '{ for (i=1; i<$1; i++) { printf " " }; print "x" }' | tac
# tacを使わずに、seqの逆順オプションを使ってみる
seq 5 -1 1 | awk '{ for (i=$1; i>=1; i--) { printf " " }; print "x" }'
# tacも逆順seqも使わずに
seq 5 | awk '{ a++; for(i=5; i>a; i--) { printf " " }; print "x" }'

# printfコマンドと初めて見る書式、幅を固定する
printf "%*s\n" 5 x 4 x 3 x 2 x 1 x

# trもシンプルながら便利ではある、-dオプションは除去
seq 4 -1 0 | awk '{ print 10^$1"x" }' | tr -d 1 | tr 0 ' '
```

## 問題7

必ずしも一息ですべてを済ませようとしなくても良い。
あと、awkの正規表現の条件は覚えておこう。

```sh
# 1回でやろうとするとうまく行かない
cat qdata/7/kakeibo.txt | awk 'BEGIN { sum = 0 } { sum += $3*1.1 } END { print sum }'

# 1列追加してみる
cat qdata/7/kakeibo.txt | awk '{ tax=1.1; print $0, tax }'
# 条件をつけて消費税を調整する、チルダでawkの正規表現の条件、3項演算子も使える
cat qdata/7/kakeibo.txt | awk '{ tax = ($1<"20191001" || $2 ~ "^*") ? 1.08 : 1.1; print $0, tax }'
# 正規表現はスラッシュで挟むほうが馴染むかも
cat qdata/7/kakeibo.txt | awk '{ tax = ($1<"20191001" || $2 ~ /^*/) ? 1.08 : 1.1; print $0, tax }'
# 1列にしてしまう
cat qdata/7/kakeibo.txt | awk '{ tax = ($1<"20191001" || $2 ~ /^*/) ? 1.08 : 1.1; print $0, tax }' | awk '{ print int($3*$4) }'
# 完成、zshではnumsumというコマンドはないっぽい？
cat qdata/7/kakeibo.txt | awk '{ tax = ($1<"20191001" || $2 ~ /^*/) ? 1.08 : 1.1; print $0, tax }' | awk '{ print int($3*$4) }' | awk '{ a+=$1 } END { print a }'
```

## 問題8

```sh
# awkの-Fオプションで区切り文字を変更する
# NFはNumber of Fieldsで列数を表す、これを利用して後ろからの列数を指定できる
cat qdata/8/access.log | awk -F: '{ print $(NF-2) }' | awk '$1 < "12" { print "午前" } $1 >= "12" { print "午後" }' | sort | uniq -c
```
