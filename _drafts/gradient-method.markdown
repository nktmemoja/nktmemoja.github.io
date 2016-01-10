---
layout: post
title:  "勾配法で目的関数値は単調に減少していくのか？"
date: 2016-01-10 04:57:55 +0900
categories: jekyll update
---
目的関数$$f: \mathbb{R}^d \rightarrow \mathbb{R}$$の最小化を考えましょう．
勾配法では，以下のようにパラメータ$$ \boldsymbol{x} $$を更新していきます．

$$
\begin{align*}
\boldsymbol{x}_{k+1} = x_k - h_k \nabla f(\boldsymbol{x}_k), \ k=0,1,\ldots
\end{align*}
$$

$$h_k > 0$$はステップサイズ．

1ステップ分の更新を考えましょう．
$$y = \boldsymbol{x} - h_k \nabla f(\boldsymbol{x})$$とすると，我々が示したいのは，

$$
\begin{align*}
f(\boldsymbol{y}) \leq f(\boldsymbol{x})
\end{align*}.
$$

これだけではなにも議論できないので，これからは目的関数はリプシッツ連続 (Lipschitz continuous) だとします．
ではまずリプシッツ連続の定義から始めましょう．

$$Q \subseteq \mathbb{R}^d, L>0$$に対して，集合$$C^{k,p}_L(Q)$$を以下の性質を有する集合とします．

- $$f \in C^{k,p}_L(Q) $$ はk回微分可能で連続
- $$f^{(p)}$$を$$f$$のp階微分とすると，
$$ \|f^{(p)}(\boldsymbol{x}) - f^{(p)}(\boldsymbol{y})\|_2 \leq L\|\boldsymbol{x}-\boldsymbol{y}\|_2 \ \forall \boldsymbol{x},\boldsymbol{y} \in Q$$.

この時$$f\in C^{k,p}_L(Q)$$はリプシッツ連続であるという．

今，目的関数が$$f \in C^{1,1}_L(\mathbb{R}^d)$$だとしましょう．この時，以下が成立します．

$$
\begin{align*}
|f(\boldsymbol{y})-f(\boldsymbol{x})-\langle f'(\boldsymbol{x}),\boldsymbol{y}-\boldsymbol{x} \rangle| \leq \frac{L}{2}\|\boldsymbol{y}-\boldsymbol{x}\|_2^2 \ \cdots (1)
\end{align*}
$$

さて，再び勾配法による一回分の更新を考えましょう．
$$ \boldsymbol{y}=\boldsymbol{x}-h\nabla f(\boldsymbol{x}) $$とすると，(1)より，

$$
\begin{align*}
f(\boldsymbol{y}) &\leq f(\boldsymbol{x}) + \langle \nabla f(\boldsymbol{x}), \boldsymbol{y}-\boldsymbol{x} \rangle + \frac{L}{2}\|\boldsymbol{y}-\boldsymbol{x}\|^2_2 \\
&= f(\boldsymbol{x}) - h\left(1 - \frac{h}{2}L\right)\|\nabla f(\boldsymbol{x})\|^2_2
\end{align*}
$$

したがって，$$ h\left(1 - \frac{h}{2}L\right) \geq 0$$，つまり，$$ h \leq \frac{2}{L} $$であれば，$$ f(\boldsymbol{y}) \leq f(\boldsymbol{x}) $$が言えます．
つまり，当たり前かもしれませんが，勾配法によって関数が単調に減少するかはステップサイズの決め方に依存するということです．
ここから，ステップサイズの決め方がいかに重要かがわかりますね．

問題なのは$$L$$がわからないということです．
もっともナイーブなステップサイズの決定方法は$$ h_k = h \ \forall k=0,1,\ldots$$として，適当に$$h>0$$を決めてやることですが，これだと必ずしも$$ h \leq \frac{2}{L} $$となっている保証はありません．そこで，なんらかの工夫が必要です．

今回はGoldstein-Armijo Ruleというステップサイズの決め方を紹介します．

