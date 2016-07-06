---
layout: post
title:  "グラフベースな文と単語への教師なし重み付けアルゴリズム"
date: 2016-07-03 13:02:53 +0900
categories: jekyll update
---

# はじめに
PageRankを応用して，グラフベースな教師なし文と単語への重み付けアルゴリズムを実装してみました．
コードは，gistに置いてあります（３点セットです）．

- <a href="https://gist.github.com/nkt1546789/f5a8f3c5bb4445d141fe7dd03a84bcd1" target="_blank">graphranker.py</a>
- <a href="https://gist.github.com/nkt1546789/7562a66aa7157377fa531f164495a3be" target="_blank">textrank.py</a>
- <a href="https://gist.github.com/nkt1546789/a53a145282fd3befb89e5618408f4cff" target="_blank">tokenrank.py</a>

# アルゴリズム
アルゴリズムはTextRankと同じです．TextRankはPageRankと同じなので，PageRankですね．
$$ N $$個のデータ点$$V = \{1, \ldots, N\}$$に対する，隣接行列を$$ \boldsymbol{A} \in \mathbb{G}^{n \times n} $$とします．
$$V$$に対するPageRankを$$ \boldsymbol{f} = \boldsymbol{f}(V) = [f(1) \cdots f(n)]^T$$とおくと，$$ \boldsymbol{f} $$は以下で定義されます．

$$
\begin{align*}
\boldsymbol{f} \leftarrow (1-d) + d \boldsymbol{A}^T \boldsymbol{f}
\end{align*}
$$

ちなみに，
# 実験

以下，ウェブページに適用して，定性評価をしてみます．
とりあえず対象は，

1. メインコンテンツ内の文章が多いウェブページ
<a href="http://www.lifehacker.jp/2016/07/160702_grilledapple.html" target="_blank">http://www.lifehacker.jp/2016/07/160702_grilledapple.html</a>
2. メインコンテンツ内の文章が少ないページ
<a href="http://navitokyo.com/0422-88-6480/" target="_blank">http://navitokyo.com/0422-88-6480/</a>

にしようと思います．予想としては1はうまく行くけど2はダメといったところでしょうか．


1. 構造を持っている (DOM)
2. 内容のほとんどがノイズ

1について。ウェブページ、つまりHTMLで書かれた文書は、DOMと呼ばれる構造を持っています。重み付けを行う際に、この構造を利用したほうが、より良い結果が得られると予想できます。

2について。ウェブページはサイドバー、ヘッダーフッター、広告など、自身の内容と関係のないコンテンツを多く含んでいます。ウェブページによりますが、ひどいと80%がノイズなんてケースもあります。このような状況だと、言語処理でよく使われる、**頻度**が意味をなさなくなります。

1,2を緩和するために、ウェブページから本文を抽出する、本文抽出 (Main contents extraction)が盛んに研究されています。本文抽出の多くの手法はDOMの構造を利用しています。教師なしと教師あり両方提案されています。本文抽出を使って、単語への重み付けを行う場合は、本文抽出をして、本文部分に強い重みをつける、という処理が考えられます。

本文抽出とWeightressはここに違いがあります。Weightressは本文抽出を経ずに直接単語へ重み付けを行います。

# アルゴリズム

$$
WS(v_i) = (1-d) + d \sum_{v_j \in In(v_i)} \frac{w_{ji}}{\sum_{v_k \in Out(v_j)} w_{jk}} WS(v_j)
$$

$$
w_{ij} = sim(b_i, b_j)
$$

# 実験
Weightressをデータに適用し、その評価を行う。単語重み付けの正解データとはなにかを定義するのは難しいが、「ページに対して、高い重みを持つべき単語集合」を用意すれば、それなりの評価はできそうだ。そのような(ページ、単語集合)ペアを、\\( \\{(x_i, Y_i)\\}_{i=1}^n \\) とする。
