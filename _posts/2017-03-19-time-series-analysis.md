---
layout: post
title:  "指数混合分布 (Mixture of Exponential Distributions) 推定のバースト検出への応用"
date: 2017-03-19 15:31:39 +0900
categories: jekyll update
---

## 問題設定

$$y \in \{0, \ldots, c\}$$: 状態 (0は平常状態, 1以上はバースト状態の段階を表す)  
$$c \in (0, 1)$$: 状態遷移確率 (cで状態を遷移し，(1-c)でとどまる)  
$$x_t \in \mathbb{R}$$: シグナルの到着間隔

$$\{x_t\}_{t=1}^{T}$$から，$$\{y_t\}_{t=1}^T$$を予測する問題．

## 指数混合分布推定

各状態におけるシグナルの到着間隔は，指数分布に従うとする．つまり，
$$p(x|y) = \lambda_{y} \exp\left(-\lambda_y x\right)$$．

そして，シグナルの到着間隔を指数混合モデルで表す:

$$
p(x) = \sum_{y=0}^c p(x|y)p(y) = \sum_{y=0}^c \theta_y \lambda_y \exp\left(-\lambda_y x\right)
$$

$$\boldsymbol{\lambda} = [\lambda_0 \cdots \lambda_c]^T$$を最尤推定する．

最適化問題は，以下で表される．

$$
\begin{align*}
\max_{\boldsymbol{\lambda}} &\ J(\boldsymbol{\lambda}) \\
s.t. &\ \sum_{y=0}^c \theta_y = 1
\end{align*}
$$

ただし，

$$
\begin{align*}
J(\boldsymbol{\lambda}) 
&= \sum_{t=1}^T \log p(x_t) \\
&= \sum_{t=1}^T \log \sum_{y=0}^c \theta_y \lambda_y \exp\left(-\lambda_y x\right).
\end{align*}
$$

$$\log$$の中にsumがいるよく見る形．
Jensenの不等式で下界$$b(\boldsymbol{\lambda}, \boldsymbol{\theta}, \boldsymbol{Q})$$を導出する．

$$
\begin{align*}
J(\boldsymbol{\lambda}) 
&= \sum_{t=1}^T \log \sum_{y=0}^c \theta_y \lambda_y \exp\left(-\lambda_y x\right) \\
&=  \sum_{t=1}^T \log \sum_{y=0}^c Q_{ty} \frac{\theta_y \lambda_y \exp\left(-\lambda_y x\right)}{Q_{ty}} \\
&\geq  \sum_{t=1}^T \sum_{y=0}^c Q_{ty} \left( \log \theta_y \lambda_y \exp\left(-\lambda_y x\right) - \log Q_{ty}\right) \\
&=  \sum_{t=1}^T \sum_{y=0}^c Q_{ty} \left( \log \theta_y + \log \lambda_y -\lambda_y x - \log Q_{ty}\right) =: b(\boldsymbol{\lambda}, \boldsymbol{\theta}, \boldsymbol{Q})
\end{align*}
$$

ただし，$$\sum_{y=0}^c Q_{ty} = 1 \ \forall t=1,\ldots,T$$．
下界$$b(\boldsymbol{\lambda}, \boldsymbol{\theta}, \boldsymbol{Q})$$を$$\boldsymbol{Q}$$に関して最大化し，$$J$$の近似を得る．
以下の最適化問題を解く．

$$
\begin{align*}
\max_{\boldsymbol{Q}} &\ b(\boldsymbol{\lambda}, \boldsymbol{\theta}, \boldsymbol{Q}) \\
s.t. &\ \sum_{y=0}^c Q_{ty} = 1 \ \forall t=1,\ldots,T
\end{align*}
$$

Lagrangianは以下で表される．

$$
L(\boldsymbol{Q}, \boldsymbol{\alpha}) = \sum_{t=1}^T \sum_{y=0}^c Q_{ty} \left( \log \theta_y + \log \lambda_y -\lambda_y x - \log Q_{ty}\right) + \sum_{t=1}^T \alpha_t \left(\sum_{y=0}^c Q_{ty} - 1\right)
$$

ただし，$$\boldsymbol{\alpha} = [\alpha_1 \cdots \alpha_T]^T$$．
最適解は以下を満たす．

$$
\begin{align*}
\frac{\partial L}{\partial Q_{ty}} = 0 
&\Rightarrow \log \theta_y + \log \lambda_y -\lambda_y x - \log Q_{ty} - 1 + \alpha_t = 0 \\
&\Rightarrow \log Q_{ty} = \alpha_t - 1 + \log \theta_y + \log \lambda_y -\lambda_y x \\
&\Rightarrow Q_{ty} = \exp\left(\alpha_t - 1\right) \theta_y \lambda_y \exp\left(-\lambda_y x\right) \\
\frac{\partial L}{\partial \alpha_t} = 0 
&\Rightarrow \sum_{y=0}^c Q_{ty} = 1 \\
&\Rightarrow \exp\left(\alpha_t - 1\right) \sum_{y=0}^c \theta_y \lambda_y \exp\left(-\lambda_y x\right) = 1 \\
&\Rightarrow \exp\left(\alpha_t - 1\right) = \frac{1}{\sum_{y=0}^c \theta_y \lambda_y \exp\left(-\lambda_y x\right)} 
\end{align*}
$$

よって，

$$
\begin{align*}
Q_{ty} = \frac{\theta_y \lambda_y \exp\left(-\lambda_y x\right)}{\sum_{y'=0}^c \theta_{y'} \lambda_{y'} \exp\left(-\lambda_{y'} x\right)}.
\end{align*}
$$

これで，$$J$$の近似を得た．次はこれを，$$\boldsymbol{\lambda}, \boldsymbol{\theta}$$に関して最大化する．
以下の最適化問題を解く．

$$
\begin{align*}
\max_{\boldsymbol{\lambda}, \boldsymbol{\theta}} &\ b(\boldsymbol{\lambda}, \boldsymbol{\theta}, \boldsymbol{Q}) \\
s.t. &\ \sum_{y=0}^c \theta_y = 1
\end{align*}
$$

Lagrangianは以下のようになる．

$$
\begin{align*}
L(\boldsymbol{\lambda}, \boldsymbol{\theta}) = \sum_{t=1}^T \sum_{y=0}^c Q_{ty} \left( \log \theta_y + \log \lambda_y -\lambda_y x - \log Q_{ty}\right) + \beta \left(\sum_{y=0}^c \theta_y - 1\right)
\end{align*}
$$

最適解は以下を満たす．

$$
\begin{align*}
\frac{\partial L}{\partial \lambda_y} = 0
&\Rightarrow \frac{1}{\lambda_y} \sum_{t=1}^T Q_{ty} - \sum_{t=1}^T Q_{ty} x_t = 0 \\
&\Rightarrow \lambda_y = \frac{\sum_{t=1}^T Q_{ty}}{\sum_{t'=1}^T Q_{t'y} x_{t'}} \\
\frac{\partial L}{\partial \theta_y} = 0
&\Rightarrow \frac{1}{\theta_y} \sum_{t=1}^T Q_{ty} + \beta = 0 \\
&\Rightarrow \theta_y = - \frac{1}{\beta} \sum_{t=1}^T Q_{ty} \\
\frac{\partial L}{\partial \beta} = 0
&\Rightarrow \sum_{y=0}^c \theta_y = 1 \\
&\Rightarrow - \frac{1}{\beta} \sum_{t=1}^T \sum_{y=0}^c Q_{ty} = 1 \\
&\Rightarrow \beta = - \sum_{t=1}^T \sum_{y=0}^c Q_{ty} \\
\end{align*}
$$

よって，

$$
\begin{align*}
\lambda_y &= \frac{\sum_{t=1}^T Q_{ty}}{\sum_{t'=1}^T Q_{t'y} x_{t'}} \\
\theta_y &= \frac{\sum_{t=1}^T Q_{ty}}{\sum_{t'=1}^T \sum_{y'=0}^c Q_{t'y'}}
\end{align*}
$$

以下の順番でパラメータを更新していく．
これはいわゆるEMアルゴリズムなんだと思う．

$$
\begin{align*}
Q_{ty} &\leftarrow \frac{\theta_y \lambda_y \exp\left(-\lambda_y x\right)}{\sum_{y'=0}^c \theta_{y'} \lambda_{y'} \exp\left(-\lambda_{y'} x\right)} \\
\lambda_y &\leftarrow \frac{\sum_{t=1}^T Q_{ty}}{\sum_{t'=1}^T Q_{t'y} x_{t'}} \\
\theta_y &\leftarrow \frac{\sum_{t=1}^T Q_{ty}}{\sum_{t'=1}^T \sum_{y'=0}^c Q_{t'y'}}
\end{align*}
$$

さて，こうして$$\boldsymbol{\lambda}$$が得られたわけですが，このままだとどれが平常状態でどれがバースト状態なのかわかりません．
しかし，指数分布の期待値が$$1 / \lambda$$で表されることから，この値が小さい，つまり$$\lambda$$が大きいものがより強いバースト状態を表しているとして良いでしょう．
よって，バースト検出で使う場合，バースト強さの段階を$$k_1, \ldots, k_c$$，ただし，$$\lambda_{k_1} \geq \cdots \geq \lambda_{k_c}$$とすれば良さそうです．
この場合，$$k_c$$を平常状態としています．


## バースト検出への応用
バースト検出への応用の仕方は，<a href="http://amzn.to/2n9VF0q" target="_blank">ウェブデータの機械学習 (機械学習プロフェッショナルシリーズ)</a>の2章を参照してください．

人工データで実験をしてみた．
実験結果を以下に示す．

<div style="text-align: center;">
<img src="{{nktmemo.github.io}}/assets/burst_detection_truepng.png">
</div>

<div style="text-align: center;">
<img src="{{nktmemo.github.io}}/assets/burst_detection_estimated.png">
</div>

横軸が時間で，縦軸が青い線の場合出現回数で，赤の線の場合が状態を示しています．
一つ目の図が真の値で，二つ目の図が推定結果です．
ほぼ完璧に推定できました．

今回は指数混合分布推定をバースト検出に応用してみた．
実際にバースト検出に応用する場合，状態数を2にして，推定した$$\boldsymbol{\lambda}$$の値をチェックしたほうが良いと思われます．
これは，バーストしてないデータでも，無理やりバースト状態を仮定しているためです．
例えば，$$\lambda_1 / \lambda_0 \geq e$$の時，このデータはバースト状態を持つと仮定するなどです．

