---
layout: post
title:  "TensorFlowで疎行列（Sparse Matrix）を扱う"
date: 2016-12-29 03:00:38 +0900
categories: ml
---

行列のほとんどの要素が0である時，その行列は疎 (Sparse) であるという．
ほとんどが0なのだから，値のあるとこだけ保持しておく，と言うのが疎行列 (Sparse Matrix) だ．
scipy.sparseには，csr_matrixとかが実装されていて，僕は普段それをよく使う．
今回TensorFlowを勉強していて疎行列を使いたくなったので調べてみた．
ただし，あまり資料がなかった．確かGithubのissueが一番参考になったと思う．

さて，tensorFlowにはtf.SparseTensorというのがあって，これで疎なテンソルを表現できる．
与える引数は，indices (何行何列)とvalues（そこの値）とshapeの３つ．
これらを↓みたいにplaceholderとして定義しておくといい感じ．

{% highlight python %}
sp_indices = tf.placeholder(tf.int64)
sp_values = tf.placeholder(tf.float32)
sp_shape = tf.placeholder(tf.int64)
X = tf.SparseTensor(sp_indices, sp_values, sp_shape)
{% endhighlight %}

あとは使う時に，feed_dict内でそれぞれに値を入れてやる．これで疎行列の完成．いやあ，簡単だ．

{% highlight python %}
indices = ...
values = ...
shape = ...
sess.run(..., feed_dict={sp_indices: indices, sp_values: values, sp_shape: shape}
{% endhighlight %}

indices, values, shapeの作り方は，↓みたいにすれば良い．

{% highlight python %}
dictionary = {}
sp_dict = {}
for row, document in enumerate(documents):
    for word in document:
        col = dictionary.setdefault(word, len(dictionary))
        sp_dict.setdefault((row, col), 0.0)
        sp_dict[(row, col)] += 1.0
indices = list(sp_dict.keys())
values = list(sp_dict.values())
shape = (row, len(dictionary))
{% endhighlight %}

以上TensorFlowでの疎行列の作り方でした．
