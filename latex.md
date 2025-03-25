# latexdiff

latexdiffとは，latexのコードの変更点を分かりやすく表示するためのツール（を使用するためのコマンド）である．

以下にlatexdiffを適用したファイルを示す（赤文字は削除した文字，青文字は追加した文字，黒文字は変更がない文字）．

![latexdiffを適用した結果](./image/latexdiff.png)


このページで表示するツールは研究室のlatex導入方法をしていれば，そのまま実行できるはずである．

日本語を含むlatexコードの差分を表示するには，以下のコマンドのようなオプションを付ける必要がある．

```shell:bash
latexdiff -e utf8 -t CFONT v4.tex v5.tex > v4-5_diff.tex
```

出力される`v4-5_diff.tex`をコンパイルすると上の画像のようなファイルになる．

## 参考
- https://qiita.com/S_Kagami/items/9f5afcb76d4dfc2971fc