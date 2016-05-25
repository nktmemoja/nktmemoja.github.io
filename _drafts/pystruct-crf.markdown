---
layout: post
title:  "pystructのChainCRF + FrankWolfSVMで各系列への予測値を取得する方法"
date: 2016-02-29 17:39:04 +0900
categories: jekyll update
---
pystructは構造化学習のいくつかの手法を，Python上に実装したライブラリである．
その中に系列ラベリングの手法である，条件付き確率場も実装されている．
使い方は簡単で，例えば[OCR Letter sequence recognition][pystruct-ocr-demo]にデモコードがある．

{% highlight python %}
ypred = clf.predict(X)
f = clf.model.batch_joint_feature([X], [ypred])
print clf.w.dot(f)
{% endhighlight %}


[pystruct-ocr-demo]: https://pystruct.github.io/auto_examples/plot_letters.html#sphx-glr-auto-examples-plot-letters-py
