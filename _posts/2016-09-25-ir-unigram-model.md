---
layout: post
title:  "Term Frequencyの確率論的解釈"
date: 2016-09-25 20:50:13 +0900
categories: jekyll update
---

文書を$$d \in \{1, \ldots, D\}$$, 単語を$$w \in \{1, \ldots, W\}$$とすると，
文書検索におけるゴールは$$p(w|d)$$を推定することになる．
一度，$$p(w|d)$$が推定できれば，単語$$w$$に対して，$$p(w|d)$$の大きい順に文書を並べれば，検索っぽいことができる．

さて，文書検索において古くからよく使われる関数に，TF-IDF (Term frequency - inverse document frequency)がある．
今回は，このTFの部分を確率論的に解釈したいと思う．

さて，データ$$\{(w_i, d_i)\}_{i=1}^m$$が与えられたとする．
この時，対数尤度は以下のようになる．

$$
\begin{align*}
\sum_{i=1}^m \log p(w_i, d_i) 
&= \sum_{d=1}^D \sum_{n=1}^{N_{d}} \log p(w^{(d)}_n, d) \\
&= \sum_{d=1}^D \sum_{w=1}^{W} N_{dw} \log p(w, d) \\
&= \sum_{d=1}^D \sum_{w=1}^{W} N_{dw} \left(\log p(w|d) + \log p(d)\right) \\
\end{align*}
$$

ただし，$$N_d$$は文書$$d$$が含む総単語数，$$w^{d}_n$$は文書$$d$$の$$n$$番目に出現する単語，$$N_{dw}$$は文書$$d$$での単語$$w$$の出現回数である．
今回は，$$p(w|d), p(d)$$をそれぞれ以下のようにモデル化する．

$$
\begin{align*}
p(w|d) = \phi_{dw}, \ \ p(d) =  \theta_d
\end{align*}
$$

制約条件を加えて最適化問題を書くと以下のような問題を解くことで，パラメータ$$\boldsymbol{\Phi}, \boldsymbol{\theta}$$を求める．

$$
\begin{align*}
\max_{\boldsymbol{\Phi}, \boldsymbol{\theta}} &\ \sum_{d=1}^D \sum_{w=1}^{W} N_{dw} \left(\log \phi_{dw} + \log \theta_d\right) \\
s.t. &\ \sum_{w=1}^W \phi_{dw} = 1 \ \forall d=1,\ldots,D \\
&\ \sum_{d=1}^D \theta_d = 1
\end{align*}
$$

この時ラグランジアンは以下で与えられる．

$$
\begin{align*}
L(\boldsymbol{\Phi}, \boldsymbol{\theta}, \boldsymbol{\alpha}, \beta)
= \sum_{d=1}^D \sum_{w=1}^{W} N_{dw} \left(\log \phi_{dw} + \log \theta_d\right) 
+ \sum_{d=1}^D \alpha_d \left(\sum_{w=1}^W \phi_{dw} - 1\right)
+ \beta \left(\sum_{d=1}^D \theta_d - 1\right)
\end{align*}
$$

最適解は以下の条件を満たす．
$$
\begin{align}
\frac{\partial L}{\partial \phi_{dw}} = 0 &\Rightarrow \phi_{dw} = - \frac{N_{dw}}{\alpha_d} \\
\frac{\partial L}{\partial \alpha_d} = 0 &\Rightarrow \sum_{w=1}^W \phi_{dw} = 1 \\
(1), (2) &\Rightarrow \alpha_d = - N_d \\
(1), (3) &\Rightarrow \phi_{dw} = \frac{N_{dw}}{N_d}
\end{align}
$$

よって，
$$
\begin{align*}
p(w|d) = \frac{N_{dw}}{N_d}
\end{align*}
$$
となり，これはTerm Frequency (TF)に一致する．
なお，今回は，$$p(w|d)$$のみに関心があったので，$$\boldsymbol{\theta}$$については省略する．

こうして見てみると，文書ごとにユニグラムモデルを考えているように見える．
次は，これを混合ユニグラムモデルに拡張してみようと思う．
