---
layout: post
title:  "多層パーセプトロンでデータの可視化"
date: 2016-12-19 01:20:51 +0900
categories: ml
---

前回，<a href="http://nktmemoja.github.io/ml/2016/12/18/chainer_mlp.html" target="_blank">ChainerでDeep learning入門 多層パーセプトロンで分類問題を解く</a>という記事を書きました．
今回は，多層パーセプトロンを使った，データの可視化に挑戦したいと思います．
アイデアは非常にシンプルです．まず，前回構築した多層パーセプトロンは，式で書くと以下のような形をしています．

$$
\begin{align*}
\boldsymbol{u}^{(1)} &= \boldsymbol{W}^{(1)} \boldsymbol{x} + \boldsymbol{b}^{(1)}\\
\boldsymbol{z}^{(1)}_j &= relu_j(\boldsymbol{u}^{(1)}) = \max\{0, u^{(1)}_j\} \\
\boldsymbol{u}^{(2)} &= \boldsymbol{W}^{(2)} \boldsymbol{z}^{(1)}  + \boldsymbol{b}^{(2)} \\
\boldsymbol{z}^{(2)}_j &= relu_j(\boldsymbol{u}^{(2)}) = \max\{0, u^{(2)}_j\} \\
\boldsymbol{u}^{(3)} &= \boldsymbol{W}^{(3)} \boldsymbol{z}^{(2)}  + \boldsymbol{b}^{(3)}\\
\boldsymbol{y} &= softmax(\boldsymbol{u}^{(3)})
\end{align*}
$$

これを見ると，
入力$$ \boldsymbol{x}$$は，非線形変換を経て，$$ \boldsymbol{u}^{(3)} $$になり，これに対して線形のロジスティック回帰を適用しているようにも見えます．
そうすると，ネットワークの中で，特徴量を学習し，その結果が$$ \boldsymbol{u}^{(3)}$$という特徴ベクトルであると解釈できます．
これは教師ありの次元削減とも考えられるのではないでしょうか．
というわけで，今回はこの$$ \boldsymbol{u}^{(3)}$$をプロットしてみようと思います．
線形分離可能な状態になっているのでしょうか？
可視化のために，隠れユニット数は2にしておきます．

結果は以下のようになりました．左が元データで，右が，$$ \boldsymbol{u}^{(3)}$$をプロットしたものです．
うまく線形分離可能な状態になっていますね．
これを使えば，いろんなデータを2次元，あるいは3次元に落として可視化できるのではないでしょうか．

![mlp_xor_visualization]({{nktmemo.github.io}}/assets/mlp_xor_visualization.png)
