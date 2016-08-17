---
layout: post
title:  "EMアルゴリズムによる混合ユニグラムモデルのパラメータ推定の割と丁寧な導出"
date: 2016-08-17 12:54:02 +0900
categories: MachineLearining NLP
---

$$\boldsymbol{w}_d=\left(w_{d1}, \ldots, w_{dN_d}\right)$$を単語列（文書）とし，i.i.d.標本$$\boldsymbol{W} = \{\boldsymbol{w}_d\}_{d=1}^D$$が与えられているとする．ただし，$$w_{dn}$$は単語を表し，$$w_{dn} \in \{1, \ldots, V\}$$である．ここで，トピックという概念を導入する．トピックは，例えば「政治」とか「経済」みたいなもの．$$K$$個のトピックを考え，これを$$z \in \{1, \ldots, K\}$$で表す．

混合ユニグラムモデルでは，文書という名の単語列はただ一つのトピックに基づいて生成されると仮定する．
つまり，文書がトピックをもち，トピックが単語分布を持つ，というイメージ．
トピック$$z$$と単語$$v$$が従う確率分布が持つ確率質量関数$$p(z;\boldsymbol{\theta})$$と$$p(v|z;\boldsymbol{\Phi})$$をそれぞれ以下のように定義する．

$$
\begin{align*}
p(z=k)=\theta_k, &\ \sum_{k=1}^K \theta_k = 1 \\
p(v|z=k)=\Phi_{kv}, &\ \sum_{v=1}^V \Phi_{kv} \ \forall k = 1,\ldots, K
\end{align*}
$$

これはカテゴリ分布と呼ばれているらしい．

さて，パラメータ$$\boldsymbol{\theta}, \boldsymbol{\Phi}$$をデータ
$$\boldsymbol{W} = \{\boldsymbol{w}_d\}_{d=1}^D$$から推定することを考える．
とりあえず最尤推定してみよう．
最適化問題は以下のようになる．

$$
\begin{align*}
\max_{\boldsymbol{\theta}, \boldsymbol{\Phi}} &\ J(\boldsymbol{\theta}, \mathbf{\Phi}) \\
s.t. &\ \sum_{k=1}^K \theta_k = 1 \\
&\ \sum_{v=1}^V \Phi_{kv} = 1 \ \forall k=1,\ldots,K
\end{align*}
$$

ただし，対数尤度$$J(\boldsymbol{\theta}, \boldsymbol{\Phi})$$は，

$$
\begin{align*}
J(\boldsymbol{\theta}, \mathbf{\Phi}) = \sum_{d=1}^D \log p(\boldsymbol{w}_d) = \sum_{d=1}^D \log \sum_{k=1}^K \theta_k \prod_{v=1}^{V} \Phi_{kv}^{N_{dv}}
\end{align*}.
$$

ちなみに，

$$
\begin{align*}
p(\boldsymbol{w}_d)
&= \sum_{k=1}^K p(\boldsymbol{w}_d|z=k)p(z=k) \\
&= \sum_{k=1}^K p(z=k) \prod_{n=1}^{N_d} p(w_{dn}|z=k) \\
&= \sum_{k=1}^K p(z=k) \prod_{v=1}^{V} p(v|z=k)^{N_{dv}} \\
&= \sum_{k=1}^K \theta_k \prod_{v=1}^{V} \Phi_{kv}^{N_{dv}}
\end{align*}
$$

である．ここのきもは，文書はトピックに基づいて生成されると仮定したので，$$p(\boldsymbol{w})$$を周辺化してトピックの条件付き確率で記述している点．
僕なりにいうと，文書がトピックとは独立に生成されるなんてことはありえないというか今そんなことは考えていないのでこうやるんだと思っている．

さて，解いていきたいところだけど，実はこの問題，ラグランジュの未定乗数法で条件は出せるが，解析的に解が計算できない．
これは，logの中にsumがいて邪魔だから．
この問題を解決するために，以下のようにジェンセンの不等式（Jensen's inequality）を用いて対数尤度の下界を導出する．

$$
\begin{align*}
J(\boldsymbol{\theta}, \mathbf{\Phi})
&= \sum_{d=1}^D \log \sum_{k=1}^K \theta_k \prod_{v=1}^{V} \Phi_{kv}^{N_{dv}}  \\
&= \sum_{d=1}^D \log \sum_{k=1}^K Q_{dk} \frac{\theta_k \prod_{v=1}^{V} \Phi_{kv}^{N_{dv}} }{Q_{dk}} \\
&\geq \sum_{d=1}^D \sum_{k=1}^K Q_{dk} \log \frac{\theta_k \prod_{v=1}^{V} \Phi_{kv}^{N_{dv}} }{Q_{dk}} \\
&= \sum_{d=1}^D \sum_{k=1}^K Q_{dk} \left(\log \theta_k \prod_{v=1}^V \Phi_{kv}^{N_{dv}} - \log Q_{dk}\right) := F_{\boldsymbol{Q}}(\boldsymbol{\boldsymbol{\theta}, \boldsymbol{\Phi}}) \\
\end{align*}
$$

ただし，$$\sum_{k=1}^K Q_{dk} = 1$$.
なんか僕は最初，一行目ですでに$$\sum_{k=1}^K \theta_k = 1$$という制約があるんだからジェンセンの不等式適用できるじゃないか！って思っていました．
あと，多分$$Q_{dk}$$じゃなくて，$$q_{k}$$とかにしてもいいと思いますが，こうすると後々楽なんだと理解しています．
さて，Jの代わりにFを最大化するのですが，これって何をやっているのでしょうか．
一つの解釈は，$$\boldsymbol{Q}$$は下界Fが持つパラメータで，Fを$$\boldsymbol{Q}$$に関して最大化して，Jの近似を得ているのでは，と思います．
では，以下の問題を解いて，Jの近似を得ましょう．

$$
\begin{align*}
\max_{\boldsymbol{Q}} &\ F_{\boldsymbol{Q}}(\boldsymbol{\theta}, \boldsymbol{\Phi}) \\
s.t. &\ \sum_{k=1}^K Q_{dk} = 1
\end{align*}
$$

Lagrangianは以下のようになる．

$$
\begin{align*}
L(\boldsymbol{Q}, \lambda) = F(\boldsymbol{Q}) + \lambda \left(\sum_{k=1}^K Q_{dk} - 1\right)
\end{align*}
$$

以下途中計算．

$$
\begin{align*}
\frac{\partial L}{\partial Q_{dk}} = 0
&\Rightarrow  \log \theta_k \prod_{v=1}^V \Phi_{kv}^{N_{dv}} - \log Q_{dk} -1 + \lambda = 0\\
&\Rightarrow \log Q_{dk} = \log \theta_k \prod_{v=1}^V \Phi_{kv}^{N_{dv}} + \lambda - 1 \\
&\Rightarrow Q_{dk} = \exp\left(\lambda - 1\right) \theta_k \prod_{v=1}^{V} \Phi_{kv}^{N_{dv}} \\
\end{align*}
$$

$$
\begin{align*}
\frac{\partial L}{\partial \lambda} = 0
&\Rightarrow \sum_{k=1}^K Q_{dk} = 1 \\
&\Rightarrow \exp\left(\lambda - 1\right) \sum_{k=1}^K \theta_k \prod_{v=1}^{V} \Phi_{kv}^{N_{dv}}  = 1 \\
&\Rightarrow \exp\left(\lambda - 1\right) = \frac{1}{\sum_{k=1}^K \theta_k \prod_{v=1}^{V} \Phi_{kv}^{N_{dv}} } \\
\end{align*}
$$

これらから，

$$
\begin{align*}
Q_{dk} = \frac{\theta_k \prod_{v=1}^{V} \Phi_{kv}^{N_{dv}} }{\sum_{k=1}^K \theta_k \prod_{v=1}^{V} \Phi_{kv}^{N_{dv}} }
\end{align*}
$$

を得る．
さて，Jの近似が得られました．
今度は，Jの近似$$F_{\hat{\boldsymbol{Q}}}(\boldsymbol{\theta}, \boldsymbol{\Phi})$$を$$\boldsymbol{\theta}, \boldsymbol{\Phi}$$に関して最大化します．

Lagrangianは以下のようになります．

$$
\begin{align*}
L(\boldsymbol{\theta}, \boldsymbol{\Phi}, \alpha, \boldsymbol{\beta})
&= F_{\hat{\boldsymbol{Q}}}(\boldsymbol{\theta}, \boldsymbol{\Phi}) + \alpha \left(\sum_{k=1}^K \theta_k - 1 \right) + \sum_{k=1}^K \beta_k \left(\sum_{v=1}^V \Phi_{kv} - 1\right)
\end{align*}
$$

以下，途中計算．
まずは$$\boldsymbol{\theta}$$．

$$
\begin{align*}
\frac{\partial L}{\partial \theta_k} = 0
&\Rightarrow \sum_{d=1}^D \frac{\hat{Q}_{dk}}{\theta_k} + \alpha = 0 \\
&\Rightarrow \theta_k = - \sum_{d=1}^D \frac{\hat{Q}_{dk}}{\alpha}
\end{align*}
$$

$$
\begin{align*}
\frac{\partial L}{\partial \alpha} = 0
&\Rightarrow \sum_{k=1}^K \theta_k - 1 = 0 \\
&\Rightarrow - \frac{1}{\alpha} \sum_{k=1}^K \sum_{d=1}^D \hat{Q}_{dk} - 1 = 0 \\
&\Rightarrow \alpha = - \sum_{k=1}^K \sum_{d=1}^D \hat{Q}_{dk}
\end{align*}
$$

よって，

$$
\begin{align*}
\theta_k = \frac{\sum_{d=1}^D \hat{Q}_{dk}}{\sum_{k=1}^K \sum_{d=1}^D \hat{Q}_{dk}}
\end{align*}.
$$

次は$$\boldsymbol{\Phi}$$.

$$
\begin{align*}
\frac{\partial L}{\partial \Phi_{kv}} = 0
&\Rightarrow \frac{\sum_{d=1}^D \hat{Q}_{dk}N_{dv}}{\Phi_{kv}} + \beta_k = 0 \\
&\Rightarrow \Phi_{kv} = - \frac{\sum_{d=1}^D \hat{Q}_{dk}N_{dv}}{\beta_k}
\end{align*}
$$

$$
\begin{align*}
\frac{\partial L}{\partial \beta_k} = 0
&\Rightarrow - \frac{1}{\beta_k} \sum_{v=1}^V \sum_{d=1}^D \hat{Q}_{dk}N_{dv} - 1 = 0 \\
&\Rightarrow \beta_k = - \sum_{v=1}^V \sum_{d=1}^D \hat{Q}_{dk}N_{dv}
\end{align*}
$$

よって，

$$
\begin{align*}
\Phi_{kv} = \frac{\sum_{d=1}^D \hat{Q}_{dk}N_{dv}}{\sum_{v=1}^V \sum_{d=1}^D \hat{Q}_{dk}N_{dv}} .
\end{align*}
$$

以上をまとめると，適当に初期値$$\boldsymbol{\theta}, \boldsymbol{\Phi}$$を決めて，

$$
\begin{align*}
Q_{dk} &\leftarrow \frac{\theta_k \prod_{v=1}^{V} \Phi_{kv}^{N_{dv}}}{\sum_{k=1}^K \theta_k \prod_{v=1}^{V} \Phi_{kv}^{N_{dv}}} \\
\theta_k &\leftarrow \frac{\sum_{d=1}^D \hat{Q}_{dk}}{\sum_{k=1}^K \sum_{d=1}^D \hat{Q}_{dk}} \\
\Phi_{kv} &\leftarrow \frac{\sum_{d=1}^D \hat{Q}_{dk}N_{dv}}{\sum_{v=1}^V \sum_{d=1}^D \hat{Q}_{dk}N_{dv}} .
\end{align*}
$$

を繰り返す．

さて，こういうアルゴリズムを作ったけど，対数尤度が減少しないことの証明が最低限必要だ．
これは現在の解において，対数尤度と下界が接していれば良い．
つまり，$$\hat{\boldsymbol{\theta}}, \hat{\boldsymbol{\Phi}}$$において，$$J(\hat{\boldsymbol{\theta}}, \hat{\boldsymbol{\Phi}}) = F_{\hat{\boldsymbol{Q}}}(\hat{\boldsymbol{\theta}}, \hat{\boldsymbol{\Phi}})$$となっていれば良い．
そうすると，下界の最大値は少なくとも今の対数尤度の値となる．
以下，これを証明する．

$$
\begin{align*}
F_{\hat{\boldsymbol{Q}}}(\hat{\boldsymbol{\theta}}, \hat{\boldsymbol{\Phi}})
&= \sum_{d=1}^D \sum_{k=1}^K \hat{Q}_{dk} \log \frac{\hat{\theta}_k \prod_{v=1}^V \hat{\Phi}_{kv}^{N_{dv}}}{\hat{Q}_{dk}} \\
&= \sum_{d=1}^D \sum_{k=1}^K \hat{Q}_{dk} \log \sum_{k'=1}^K \hat{\theta}_{k'} \prod_{v=1}^{V} \hat{\Phi}_{k'v}^{N_{dv}} \\
&= \sum_{d=1}^D \left(\sum_{k=1}^K \hat{Q}_{dk}\right) \log \sum_{k'=1}^K \hat{\theta}_{k'} \prod_{v=1}^{V} \hat{\Phi}_{k'v}^{N_{dv}} \\
&= \sum_{d=1}^D  \log \sum_{k'=1}^K \hat{\theta}_{k'} \prod_{v=1}^{V} \hat{\Phi}_{k'v}^{N_{dv}} \\
&= J(\hat{\boldsymbol{\theta}}, \hat{\boldsymbol{\Phi}})
\end{align*}
$$

これで，EMアルゴリズムによって，対数尤度は減少しないことがわかった．
