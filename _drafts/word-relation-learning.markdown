---
layout: post
title:  "単語間関係分類"
date: 2016-02-21 04:06:36 +0900
categories: jekyll update
---

## 定式化

定義：  
　$$\mathcal{X}$$: 単語空間  
　$$\mathcal{Z} = \{(x, y)|x, y \in \mathcal{X}\}$$: 単語間関係空間  
　$$\mathcal{S} = \{-1, 1\}$$: 関係候補成立のラベル  
　$$\mathcal{T} = \{-1, 1\}$$: 関係成立のラベル

目的:
$$p(s,t|x,y), \ ((x,y) \in \mathcal{Z}, s \in \mathcal{S}, t \in \mathcal{T})$$
の推定  
仮定: $$p(s,t|x,y) = p(s|x,y)p(t|x,y)$$  つまり，$$x,y$$が関係分類の候補であることと，$$x,y$$の関係が成立していることは独立である．  
制約: $$\forall x,y \in \mathcal{X}, \ p(t=1|x,y) > p(t=-1|x,y) \Leftrightarrow p(t=1|y,x) < p(t=-1|y,x)$$  
この制約は，$$p(t=1|y,x) = p(t=-1|x,y)$$ならば満たされるので，これを制約とする．  
設定: トレーニングデータとして，$$\{(x_i, y_i, t_i)\}_{i=1}^n$$が与えられる  
補足: $$s$$を考えるのは，$$t$$だけだと，関係が成立するかしないかしか表現できないので，
どっちでもない場合というか2語が全く関係ない場合にめちゃくちゃな予測をしてしまう．
これは，分類問題一般に言えることだが，定義したクラスに対して，「それ以外」な入力が必ずと言っていいほど存在するので，
本来はこのような設定で問題を考えるべきだと僕は思う．
このような設定の場合は，One Class SVMなどのOutlier Detectionの手法が用いられる．
予測:  
$$(\boldsymbol{x}, \boldsymbol{y})$$の関係が成り立っているかどうかは以下のように予測する．
まず，以下で定義する$$(\hat{s}, \hat{t})$$を求める．

$$
\begin{align*}
(\hat{s}, \hat{t}) = \text{argmax}_{(s,t) \in \mathcal{S} \times \mathcal{T}} \ p(s, t|\boldsymbol{x},\boldsymbol{y})
\end{align*}
$$

そして，$$\hat{s}=1, \hat{t}=1$$の時に限り，$$(\boldsymbol{x}, \boldsymbol{y})$$の関係が成り立っているとする．

$$p(s|\boldsymbol{x},\boldsymbol{y})$$について．
今回は，関係候補となる単語同士は互いに似ているとして，類似度$$s: \mathcal{X} \times \mathcal{X} \rightarrow [0,1]$$を用いて，
$$p(s=1|x,y) = s(x,y)$$
と定義する．
さらに，簡単のため，$$\mathcal{X}=\mathbb{R}^d$$とする．

$$p(t|\boldsymbol{x},\boldsymbol{y})$$
の推定には，ロジスティック回帰を用いる．
$$p(t|\boldsymbol{x},\boldsymbol{y})$$
を以下で定義される，対数線形モデル$$q(t|\boldsymbol{x},\boldsymbol{y};\boldsymbol{\theta}), \boldsymbol{\theta} \in \mathbb{R}^d$$でモデル化する．

$$
\begin{align*}
q(t|\boldsymbol{x},\boldsymbol{y};\boldsymbol{\theta}) = \frac{1}{1 + \exp\left(t \cdot \boldsymbol{\theta}^T(\boldsymbol{x}-\boldsymbol{y})\right)}
\end{align*}
$$

このモデルでは，$$ (\boldsymbol{x}, \boldsymbol{y})$$を$$ \boldsymbol{x}-\boldsymbol{y}$$として扱っている．
このようにモデルを定義すると，$$q(t=1|\boldsymbol{y},\boldsymbol{x}) = q(t=-1|\boldsymbol{x},\boldsymbol{y})$$となり，自動的に制約が満たされる．

以上より，解くべき問題は，

$$
\begin{align*}
\max_{\boldsymbol{\theta}} &\ \sum_{i=1}^n \log q(t_i|\boldsymbol{x}_i,\boldsymbol{y}_i;\boldsymbol{\theta}) \\
\end{align*}
$$

となり，通常のロジスティック回帰に一致する．

## 実験

### Word2Vec

gensim.Word2Vec, d=100, 他default

### 関係学習

### 結果
