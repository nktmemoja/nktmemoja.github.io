---
layout: post
title:  "劣モジュラ最適化による文書要約（理論編）"
date: 2016-02-09 00:09:34 +0900
categories: jekyll update
---
<a rel="nofollow" href="http://www.amazon.co.jp/gp/product/4061529099/ref=as_li_ss_tl?ie=UTF8&camp=247&creative=7399&creativeASIN=4061529099&linkCode=as2&tag=nettodesyuu00-22" target="_blank">劣モジュラ最適化と機械学習 (機械学習プロフェッショナルシリーズ)</a><img src="http://ir-jp.amazon-adsystem.com/e/ir?t=nettodesyuu00-22&l=as2&o=9&a=4061529099" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />
を読んでいます．
この中に，劣モジュラ最適化の文書要約への応用が紹介されていたので，勉強がてら書こうと思います．
やっぱりこういう理論的なものを学ぶ時は，応用を同時に学ぶといいですよね．

さて，まずは文書要約について．
文書要約手法には，抽出型と生成型があるらしいです．
抽出型は，要約対象に出てくる要素（例えば文）を組み合わせて，要約を作るアプローチです．
生成型は，要約対象の文書の内容を理解して，要約を生成するアプローチです．
直感的には，抽出型の方がシンプルで簡単そうな気がします．
というわけで，今回は抽出型について書こうと思います．

さて，問題を定式化していきましょう．
抽出対象は文にしようと思います．
文書を文の集合と考えると，ある文書は$$V=\{1,\ldots,n\}$$と表されます．
ここで，$$n$$はその文書が含む総文数です．
抽出型の文書要約はなんらかの基準で$$S \subseteq V$$を選ぶ問題だと解釈できます．

このような基準は，自然言語処理分野で数多く提案されているみたいですが，
今回は本に出てきた，「関連性」と「多様性」による基準を採用しようと思います．
ここでいう関連性とは，要約文と本文との関連度を表し，
多様性は，要約文に含まれる文の多様性を表します．
つまり，この二つを同時に最大化したいわけです．
さて，要約文$$S \subseteq V$$の関連性を$$L(S)$$，多様性を$$R(S)$$と表し，要約文の良さを表す関数$$f_{doc}$$を以下で定義します．

$$
\begin{align*}
f_{doc} = L(S) + \lambda R(S)
\end{align*}
$$

ここで，$$\lambda$$はトレードオフを調節するパラメータです．
さて，まずは全文$$V$$と要約文$$S$$の関連性を表す関数$$L(S)$$について考えましょう．

$$
\begin{align*}
L(S) = \sum_{i \in V} \min\{C_i(S), \gamma C_i(V)\}
\end{align*}
$$


$$
\begin{align*}
C_i(S) = \sum_{j \in S} s_{ij}
\end{align*}
$$

$$
\begin{align*}
R(S) = \sum_{k=1}^K \sqrt{\sum_{j \in P_k \cap S} r_j}
\end{align*}
$$
