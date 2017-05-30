---
layout: post
title:  "劣微分についてのメモ"
date: 2017-05-31 05:48:59 +0900
categories: ML CVXOPT
---

最急降下法 (Gradient descent)を使えば，任意の関数を最小化することができる．もし対称の関数が凸関数 (Convex function)ならば，グローバルな最適解を得ることができる．まずこのことを確認しよう．

凸関数であることと，次の不等式が成り立つことは等価である．これはFirst-order Conditionと呼ばれる．

$$
\begin{align*}
f(y) \geq f(x) + \langle f'(x), (y-x) \rangle \ \forall x, y
\end{align*}
$$

このことから直ちに次のことが言える．

$$
\begin{align*}
f(x) - f(x - \alpha f'(x)) \geq \langle f'(x), (x - (x-\alpha f'(x))) \rangle = \alpha \|f'(x)\|^2 \geq 0
\end{align*}
$$

最後は$$\alpha > 0$$を仮定．よって，勾配法によって解が悪くなることは無い．このことから，勾配法によって解が悪くならないことを保証するには，最初の不等式が成立していれば十分であり，別に$$f'(x)$$じゃなくてもいいことがわかる．なので，次の集合を用意する．

$$
\begin{align*}
\partial f(x) = \{g | f(y) \geq f(x) + \langle g, y-x \rangle \ \forall y\}
\end{align*}
$$

これは劣微分と呼ばれ，この元を劣勾配という．この定義から，劣勾配を使えば，勾配法により，目的関数は単調非増加となる．言い換えれば解が悪くなることは無い．
