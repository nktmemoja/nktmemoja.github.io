---
layout: post
title:  "ハードマージンSupport Vector Machine (SVM)の定式化"
date:  2016-01-08 00:30:29 +0900
image: 
categories: jekyll update
---
<a rel="nofollow" href="http://www.amazon.co.jp/gp/product/4061529064/ref=as_li_ss_tl?ie=UTF8&camp=247&creative=7399&creativeASIN=4061529064&linkCode=as2&tag=nettodesyuu00-22" target="_blank">サポートベクトルマシン (機械学習プロフェッショナルシリーズ)</a><img src="http://ir-jp.amazon-adsystem.com/e/ir?t=nettodesyuu00-22&l=as2&o=9&a=4061529064" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />
を読んでいます．
SVMについて，わかっている気になっていましたが，わかっていませんでした．
意外と奥が深いですね．というわけでその整理の意味も込めてのポスト．

この記事ではハードマージンSVMを考えます．
ハードマージンSVMでは，与えられたデータは線形分離可能であると仮定します．
つまり，超平面$$f(\boldsymbol{x})=\boldsymbol{w}^T \boldsymbol{x} + b$$で，与えられた訓練データがミスなく分離できることを仮定します．
このような超平面は複数存在すると考えられますが，
SVMでは，「マージンを最大化する超平面を求める」，というのはよくある説明ですね．

ではまず，マージンの定義から入りましょう．
今，訓練データ$$\{(\boldsymbol{x}_i,y_i)\}_{i=1}^n, \boldsymbol{x}_i \in \mathbb{R}^d, y_i \in \{-1,1\} \ \forall i=1, \ldots, n$$が与えられているとします．
マージンを，
超平面$$f(\boldsymbol{x})=\boldsymbol{w}^T \boldsymbol{x} + b$$からもっとも近いサンプルまでの距離と定義します．
つまり，

$$
\begin{align*}
i^{*} = \mathop{\rm arg\,min}\limits_{1 \leq i \leq n} \ \frac{|\boldsymbol{w}^T \boldsymbol{x}_i + b|}{\|\boldsymbol{w}\|}
= \mathop{\rm arg\,min}\limits_{1 \leq i \leq n} \ |\boldsymbol{w}^T \boldsymbol{x}_i + b|
\end{align*}
$$

と定義すると，マージンは

$$
\begin{align*}
\frac{|\boldsymbol{w}^T \boldsymbol{x}_{i^{*}} + b|}{\|\boldsymbol{w}\|}
\end{align*}
$$

と表すことができます．
全ての訓練データを正しく分類しつつ，このマージンを最大化したいので，以下の最適化問題を解きます．

$$
\begin{align*}
\max_{\boldsymbol{w},b} &\ \frac{|\boldsymbol{w}^T \boldsymbol{x}_{i^{*}} + b|}{\|\boldsymbol{w}\|} \\
s.t. &\ y_i(\boldsymbol{w}^T \boldsymbol{x}_i + b) > 0 \ \forall i=1,\ldots ,n
\end{align*}
$$

分子が邪魔なので，これを以下のように書き換えます．

$$
\begin{align*}
\max_{\boldsymbol{w},b} &\ \frac{1}{\|\boldsymbol{w}\|} \\
s.t. &\ y_i(\boldsymbol{w}^T \boldsymbol{x}_i + b) > 0 \ \forall i=1,\ldots ,n \\
&\ |\boldsymbol{w}^T \boldsymbol{x}_{i^{*}} + b |=1
\end{align*}
$$

二つの制約条件を一つにまとめると，以下のようになります．

$$
\begin{align*}
\max_{\boldsymbol{w},b} &\ \frac{1}{\|\boldsymbol{w}\|} \\
s.t. &\ y_i(\boldsymbol{w}^T \boldsymbol{x}_i + b) \geq 1 \ \forall i=1,\ldots ,n \\
\end{align*}
$$

個人的にはここの変形が一番きつかったですね...．
なので軽く補足しときます．
以下の制約を書き換えていきます．

$$
\begin{align}
 y_i(\boldsymbol{w}^T \boldsymbol{x}_i + b) > 0 \ \forall i=1,\ldots ,n \ \cdots (1) \\
 |\boldsymbol{w}^T \boldsymbol{x}_{i^{*}} + b |=1 \ \cdots (2)
\end{align}
$$

$$i^{*}$$の定義から，(2)は以下のように書き換えられます．

$$
\begin{align}
\ |\boldsymbol{w}^T \boldsymbol{x}_{i} + b| \geq 1  \ \forall i=1,\ldots ,n \ \cdots (2')
\end{align}
$$

ここで，(1)と(2')を合体させたいのですが，
$$y_i(\boldsymbol{w}^T \boldsymbol{x}_{i} + b) = |\boldsymbol{w}^T \boldsymbol{x}_{i} + b|$$
を示すことができれば良さそうです．
ここで，

$$
\begin{align*}
y_i(\boldsymbol{w}^T \boldsymbol{x}_i + b) = \left\{
\begin{array}
\boldsymbol{w}^T \boldsymbol{x}_i + b & (y_i=1) \\
-(\boldsymbol{w}^T \boldsymbol{x}_i + b) & (y_i=-1)
\end{array}
\right.
\end{align*},
$$

$$
\begin{align*}
|\boldsymbol{w}^T \boldsymbol{x}_i + b| = \left\{
\begin{array}
\boldsymbol{w}^T \boldsymbol{x}_i + b & (\boldsymbol{w}^T \boldsymbol{x}_i + b \geq 0) \\
-(\boldsymbol{w}^T \boldsymbol{x}_i + b) & (\boldsymbol{w}^T \boldsymbol{x}_i + b < 0)
\end{array}
\right.
\end{align*},
$$

ですが，(1)より，$$ \boldsymbol{w}^T \boldsymbol{x}_i + b \geq 0 $$ は成り立たず，この文脈では必ず
$$ \boldsymbol{w}^T \boldsymbol{x}_i + b > 0 $$となります．

なので，$$y_i=1 \Leftrightarrow \boldsymbol{w}^T \boldsymbol{x}_i + b > 0 $$と
$$y_i=-1 \Leftrightarrow \boldsymbol{w}^T \boldsymbol{x}_i + b < 0 $$を示すことにします．
すごく簡単すぎて示す必要もないかもしれませんが...．

$$y_i=1$$の時，(1)より，$$ \boldsymbol{w}^T \boldsymbol{x}_i + b > 0 $$．
$$ \boldsymbol{w}^T \boldsymbol{x}_i + b > 0 $$の時，(1)と$$y_i \in \{-1,1\}$$より，$$y_i=1$$．
$$y_i=-1 \Leftrightarrow \boldsymbol{w}^T \boldsymbol{x}_i + b < 0 $$も同様．

よって，
$$y_i(\boldsymbol{w}^T \boldsymbol{x}_{i} + b) = |\boldsymbol{w}^T \boldsymbol{x}_{i} + b| $$．

そういうわけで，(2')は，

$$
\begin{align}
\ y_i(\boldsymbol{w}^T \boldsymbol{x}_{i} + b) \geq 1  \ \forall i=1,\ldots ,n \ \cdots (2'')
\end{align}
$$

これで，(1)と(2'')を合体できますね．
最後に最大化と最小化を書き換えて，ノルムを2乗すれば，よく見るハードマージンSVMの最適化問題の完成です：

$$
\begin{align*}
\min_{\boldsymbol{w},b} &\ \|\boldsymbol{w}\|^2 \\
s.t. &\ y_i(\boldsymbol{w}^T \boldsymbol{x}_i + b) \geq 1 \ \forall i=1,\ldots ,n 
\end{align*}
$$

ここで，与えられたデータに対して，制約$$y_i(\boldsymbol{w}^T \boldsymbol{x}_i + b) \geq 1 \ \forall i=1,\ldots ,n $$を満たす$$ \boldsymbol{w}$$と$$b$$が存在しないかもしれません．
というより，現実問題ではそのような場合がほとんどだと考えられます．
そこで，ソフトマージンの考え方が出てきます．
これについてはまた後日書けたらと思います．
