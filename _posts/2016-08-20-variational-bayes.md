---
layout: post
title:  "変分ベイズ法 (Variational Bayes)"
date: 2016-08-20 16:01:12 +0900
categories: jekyll update
---

事後確率の近似を求めたり，事前確率のパラメータ（ハイパーパラメータ）を求めたりする時に使う変分ベイズを勉強してみた．
とりあえず，混合ユニグラムモデルの事後確率の導出に使うので，事後確率の近似を求めよう，というモチベーションで書いてみる．
基本的には，「対数尤度を最大化したい」という立場は変わらない．
入力変数が潜在変数やパラメータに依存しているため，対数尤度というのをこれらの依存関係を考慮して表現しなければならないため，こういう考えにいたったんだと思っている．

i.i.d.データ$$\mathcal{X} = \{\boldsymbol{x}_i\}_{i=1}^n$$が与えられているとする．
$$\boldsymbol{x} \sim p(\boldsymbol{x}|\boldsymbol{\theta})$$とし，パラメータ$$\boldsymbol{\theta}$$の事後確率の計算を試みる．
ここで，データ$$\mathcal{X}$$はパラメータ$$\boldsymbol{\theta}$$に依存しているため，対数尤度を表現するには，

$$
\begin{align*}
\log p(\mathcal{X}) = \log \int p(\mathcal{X}|\boldsymbol{\theta})p(\boldsymbol{\theta}) d \boldsymbol{\theta}
\end{align*}
$$

と書かなければならない．前述の依存関係の考慮というのはこういうことを指します．
もはや対数尤度がすんなり書けないのです．
ちなみにこれはパラメータで対数尤度を周辺化したので，周辺対数尤度，あるいは単に周辺尤度と呼ばれるみたいです．

さて，対数尤度の下界は，潜在変数$$\boldsymbol{\eta}$$を導入して，以下のように導出できる．

$$
\begin{align*}
\log p(\mathcal{X})
&= \log \int \int p(\mathcal{X}, \boldsymbol{\eta}, \boldsymbol{\theta}) d \boldsymbol{\eta} d \boldsymbol{\theta} \\
&= \log \int \int p'(\boldsymbol{\eta}) p''(\boldsymbol{\theta}) \frac{p(\mathcal{X}, \boldsymbol{\eta}, \boldsymbol{\theta})}{p'(\boldsymbol{\eta}) p''(\boldsymbol{\theta})}  d \boldsymbol{\eta} d \boldsymbol{\theta}\\
&\geq \int \int p'(\boldsymbol{\eta}) p''(\boldsymbol{\theta}) \log \frac{p(\mathcal{X}, \boldsymbol{\eta}, \boldsymbol{\theta}) }{p'(\boldsymbol{\eta}) p''(\boldsymbol{\theta})} d \boldsymbol{\eta} d \boldsymbol{\theta} \\
&=: F(p', p')
\end{align*}
$$

この下界$$F(p',p'')$$は変分自由エネルギー (Variational free energy)と呼ばれるらしい．
この$$F(p',p'')$$を$$p',p''$$に関して最大化することで，対数尤度の近似を得ることができるが，
今回は，対数尤度とこの下界の差に注目してみる．
対数尤度とこの下界の差は，以下のように書くことができる．

$$
\begin{align*}
\log p(\mathcal{X}) - F(p', p')
&= \log p(\mathcal{X}) - \int \int p'(\boldsymbol{\eta}) p''(\boldsymbol{\theta}) \log \frac{p(\mathcal{X}, \boldsymbol{\eta}, \boldsymbol{\theta}) }{p'(\boldsymbol{\eta}) p''(\boldsymbol{\theta})} d \boldsymbol{\eta} d \boldsymbol{\theta} \\
&= \int \int p'(\boldsymbol{\eta})p''(\boldsymbol{\theta}) \log p(\mathcal{X}) d \boldsymbol{\eta} d \boldsymbol{\theta} - \int \int p'(\boldsymbol{\eta}) p''(\boldsymbol{\theta}) \log \frac{p(\mathcal{X}, \boldsymbol{\eta}, \boldsymbol{\theta}) }{p'(\boldsymbol{\eta}) p''(\boldsymbol{\theta})} d \boldsymbol{\eta} d \boldsymbol{\theta} \\
&= \int \int p'(\boldsymbol{\eta})p''(\boldsymbol{\theta}) \left( \log p(\mathcal{X}) - \log \frac{p(\mathcal{X}, \boldsymbol{\eta}, \boldsymbol{\theta}) }{p'(\boldsymbol{\eta}) p''(\boldsymbol{\theta})} \right) d \boldsymbol{\eta} d \boldsymbol{\theta} \\
&= \int \int p'(\boldsymbol{\eta})p''(\boldsymbol{\theta}) \log \frac{p'(\boldsymbol{\eta}) p''(\boldsymbol{\theta})}{p(\boldsymbol{\eta}, \boldsymbol{\theta} | \mathcal{X})} d \boldsymbol{\eta} d \boldsymbol{\theta} \\
&= KL(p'(\boldsymbol{\eta}) p''(\boldsymbol{\theta}) || p(\boldsymbol{\eta}, \boldsymbol{\theta}|\mathcal{X}))
\end{align*}
$$

ただし，KLはカルバックライブラー情報量である．
つまり，対数尤度の近似を求めることは，$$p(\boldsymbol{\eta}, \boldsymbol{\theta}|\mathcal{X})$$を，$$p'(\boldsymbol{\eta}) p''(\boldsymbol{\theta})$$で近似することに相当する．従って，対数尤度の下界を最大化する$$p''(\boldsymbol{\theta})$$が，計算したかった事後確率$$p(\boldsymbol{\theta}|\mathcal{X})$$の良い近似になっていることが期待できる．これがからくり．

では，実際に$$p''(\boldsymbol{\theta})$$を求めてみる．
しかし，$$F$$は関数の関数という，いわゆる汎関数で，この最大化は汎関数の極値問題で少しややこしい．
求め方だけ勉強してみたので，それを書いてみる．

$$
\begin{align*}
f(x,y,z) = \int y p''(\boldsymbol{\theta}) \log \frac{p(\mathcal{X}, \boldsymbol{\eta}, \boldsymbol{\theta}) }{y p''(\boldsymbol{\theta})} d \boldsymbol{\theta} \\
\end{align*}
$$

とすると，

$$
\begin{align*}
F(p', p'') = \int f(x, p'(\boldsymbol{\eta}), z) d \boldsymbol{\eta}
\end{align*}
$$

と書ける．この$$f$$は被積分関数と呼ばれているらしい，積分される関数だから当たり前なのだけれど...．
上記の形をした汎関数の極値は，以下で定義されるオイラーの方程式を満たす．

$$
\begin{align*}
\frac{d}{dx} \left(\left.\frac{\partial f}{\partial z} \right|_{y=p'(\boldsymbol{\eta})} \right) = \left.\frac{\partial f}{\partial y}\right|_{y=p'(\boldsymbol{\eta})}
\end{align*}
$$

今考えている$$f$$は$$x$$も$$z$$も出てこないので，上記の条件は，

$$
\begin{align*}
\left.\frac{\partial f}{\partial y}\right|_{y=p'(\boldsymbol{\eta})} = 0
\end{align*}
$$

となる．この条件を使うと，

$$
\begin{align*}
\left.\frac{\partial f}{\partial y}\right|_{y=p'(\boldsymbol{\eta})} = 0
&\Rightarrow \int p''(\boldsymbol{\theta}) \left( \log \frac{p(\mathcal{X}, \boldsymbol{\eta}, \boldsymbol{\theta}) }{p'(\boldsymbol{\eta}) p''(\boldsymbol{\theta})} - 1 \right)d \boldsymbol{\theta} = 0 \\

&\Rightarrow \int p''(\boldsymbol{\theta}) \log p(\mathcal{X}, \boldsymbol{\eta} , \boldsymbol{\theta}) d \boldsymbol{\theta}
- \int p''(\boldsymbol{\theta}) \log p'(\boldsymbol{\eta}) d \boldsymbol{\theta}
- \int p''(\boldsymbol{\theta}) \log p''(\boldsymbol{\theta}) d \boldsymbol{\theta}
- \int p''(\boldsymbol{\theta}) d \boldsymbol{\theta} = 0 \\

&\Rightarrow \int p''(\boldsymbol{\theta}) \log p(\mathcal{X}, \boldsymbol{\eta} , \boldsymbol{\theta}) d \boldsymbol{\theta}
- \log p'(\boldsymbol{\eta})
- \int p''(\boldsymbol{\theta}) \log p''(\boldsymbol{\theta}) d \boldsymbol{\theta}
- 1 = 0 \\

&\Rightarrow
\log p'(\boldsymbol{\eta}) =
\int p''(\boldsymbol{\theta}) \log p(\mathcal{X}, \boldsymbol{\eta} , \boldsymbol{\theta}) d \boldsymbol{\theta}
- \int p''(\boldsymbol{\theta}) \log p''(\boldsymbol{\theta}) d \boldsymbol{\theta}
- 1\\

&\Rightarrow
p'(\boldsymbol{\eta}) =
\exp\left(\int p''(\boldsymbol{\theta}) \log p(\mathcal{X}, \boldsymbol{\eta} , \boldsymbol{\theta}) d \boldsymbol{\theta}\right)
\exp\left(- \int p''(\boldsymbol{\theta}) \log p''(\boldsymbol{\theta}) d \boldsymbol{\theta}
- 1 \right)\\

&\Rightarrow
p'(\boldsymbol{\eta}) \propto
\exp\left(\int p''(\boldsymbol{\theta}) \log p(\mathcal{X}, \boldsymbol{\eta} , \boldsymbol{\theta}) d \boldsymbol{\theta}\right)
\end{align*}
$$

となり，まず，$$p'(\boldsymbol{\eta})$$を求めることができた．

次に，$$p''(\boldsymbol{\theta})$$を求める．これが本題．

$$
\begin{align*}
g(x,y,z) = \int p'(\boldsymbol{\eta}) y \log \frac{p(\mathcal{X}, \boldsymbol{\eta}, \boldsymbol{\theta}) }{p'(\boldsymbol{\eta}) y} d \boldsymbol{\eta} \\
\end{align*}
$$

とすると，

$$
\begin{align*}
F(p', p'') = \int g(x, p''(\boldsymbol{\theta}), z) d \boldsymbol{\theta}
\end{align*}
$$

となる．同様に，$$g$$には，$$x,z$$が出てこないので，最適解は以下のように導出できる．

$$
\begin{align*}
\left.\frac{\partial g}{\partial y}\right|_{y=p''(\boldsymbol{\theta})} = 0

&\Rightarrow \int p'(\boldsymbol{\eta}) \left(\log \frac{p(\mathcal{X}, \boldsymbol{\eta}, \boldsymbol{\theta}) }{p'(\boldsymbol{\eta}) p''(\boldsymbol{\theta})} - 1 \right)  d \boldsymbol{\eta} = 0\\

&\Rightarrow \int p'(\boldsymbol{\eta}) \left(\log p(\mathcal{X}, \boldsymbol{\eta}, \boldsymbol{\theta}) - \log {p'(\boldsymbol{\eta}) - \log p''(\boldsymbol{\theta})} - 1 \right)  d \boldsymbol{\eta} = 0 \\

&\Rightarrow \int p'(\boldsymbol{\eta}) \log p(\mathcal{X}, \boldsymbol{\eta}, \boldsymbol{\theta}) d \boldsymbol{\eta}
- \int p'(\boldsymbol{\eta}) \log p'(\boldsymbol{\eta}) d \boldsymbol{\eta}
- \int p'(\boldsymbol{\eta}) \log p''(\boldsymbol{\theta})  d \boldsymbol{\eta}
- \int p'(\boldsymbol{\eta}) d \boldsymbol{\eta} = 0\\

&\Rightarrow \int p'(\boldsymbol{\eta}) \log p(\mathcal{X}, \boldsymbol{\eta}, \boldsymbol{\theta}) d \boldsymbol{\eta}
- \int p'(\boldsymbol{\eta}) \log p'(\boldsymbol{\eta}) d \boldsymbol{\eta}
- \log p''(\boldsymbol{\theta})
- 1 = 0\\

&\Rightarrow
\log p''(\boldsymbol{\theta}) =
\int p'(\boldsymbol{\eta}) \log p(\mathcal{X}, \boldsymbol{\eta}, \boldsymbol{\theta}) d \boldsymbol{\eta}
- \int p'(\boldsymbol{\eta}) \log p'(\boldsymbol{\eta}) d \boldsymbol{\eta}
- 1\\

&\Rightarrow
p''(\boldsymbol{\theta}) =
\exp\left(\int p'(\boldsymbol{\eta}) \log p(\mathcal{X}, \boldsymbol{\eta}, \boldsymbol{\theta}) d \boldsymbol{\eta}\right)
\exp\left(- \int p'(\boldsymbol{\eta}) \log p'(\boldsymbol{\eta}) d \boldsymbol{\eta}
- 1\right)\\

&\Rightarrow
p''(\boldsymbol{\theta}) \propto
\exp\left(\int p'(\boldsymbol{\eta}) \log p(\mathcal{X}, \boldsymbol{\eta}, \boldsymbol{\theta}) d \boldsymbol{\eta}\right)
\end{align*}
$$

まとめると，

$$
\begin{align*}
p'(\boldsymbol{\eta}) &\propto
\exp\left(\int p''(\boldsymbol{\theta}) \log p(\mathcal{X}, \boldsymbol{\eta} , \boldsymbol{\theta}) d \boldsymbol{\theta}\right) \\
p''(\boldsymbol{\theta}) &\propto
\exp\left(\int p'(\boldsymbol{\eta}) \log p(\mathcal{X}, \boldsymbol{\eta}, \boldsymbol{\theta}) d \boldsymbol{\eta}\right)
\end{align*}
$$

となり，これを交互に計算するアルゴリズムは，変分ベイズEMアルゴリズム (Variational Bayes EM Algorithm)と呼ばれる．

次回やる混合ユニグラムモデルのベイズ推定では，上の式を使う予定．
