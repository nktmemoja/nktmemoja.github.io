---
layout: post
title:  "クラスタリング評価指標V-Measureについて勉強してみた"
date: 2016-06-05 15:33:17 +0900
categories: jekyll update
---

# 1. V-Measure
---

最近会社でクラスタリングを使っています．
僕は今までクラスタリングの評価指標には<a href="http://nktmemoja.github.io/jekyll/update/2016/05/24/ari.html" target="_blank">ARI</a>を使っていたのですが，もう古いみたいですね．
そこで，今回，V-Measure (Rosenberg 2007) というのを見つけたので勉強してみました．

クラスタリングの評価のポイントは，大きく分けて以下の2つになるみたいです．

- Completeness: 同じクラスタに属するべきものが，同じクラスタに属している
- Homogeneity: 異なるクラスタに属するべきものが，異なるクラスタに属している

そしてこれらは，一般に再現率と適合率のように，トレードオフの関係にあります．
例えば，全てのデータ点を1つのクラスタに入れてしまえば，Completeness=1ですが，Homogeneity=0です．
反対に，全てのデータ点を異なるクラスタに入れると，Homogeneity=1ですが，Completeness=0です．

良いクラスタリングは，応用先によって違うと思うのですが，だいたいこれらのバランスを変えるだけで表現できそうな気がします．
というわけで，これらの重み付き調和平均をとって，$$V_\beta$$-Measureは以下で定義されます．

$$
\begin{align*}
V_\beta = \frac{(1+\beta)*h*c}{(\beta*h)+c}
\end{align*}
$$

ただし，$$c$$はcompleteness, $$h$$はhomogeneity, $$\beta$$はcompletenessへの重みです．
$$\beta > 1$$の時，completenessをより重要視した評価になります．
$$\beta < 1$$の時，homogeneityをより重要視した評価になります．
とても使いやすそうですね．

ここからは，CompletenessやHomogeneityの定義を見ていくことにします．

# 2. Completeness
---
割と論文の記法に沿いたいですが，あまり好きでない部分があるので適宜変えます．
今，$$N$$個のデータ$$D=\{1, \ldots, N\}$$があるとします．
真のクラスタ集合を$$C=\{c_i|c_i \subseteq D\}_{i=1}^m$$，ただし$$c_i \cap c_j = \{\} \ \forall i,j$$とします．
同様に，あるクラスタ集合（例えばクラスタリングアルゴリズムによる出力）を$$K=\{k_i|k_i \subseteq D\}_{i=1}^n$$，ただし$$k_i \cap k_j = \{\} \ \forall i,j$$とします．

completenessは，「同じクラスタに属するべきものが，同じクラスタに属している」度合いを表すと定義しました．
そう考えると，例えば，$$c_i$$の要素全てが$$k_j$$に属していると，$$c_i$$に対するcompletenessは最大ですよね．
反対に，

# 3. Homogeneity
---
