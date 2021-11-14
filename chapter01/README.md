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


