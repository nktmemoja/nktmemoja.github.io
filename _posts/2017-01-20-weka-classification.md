---
layout: post
title:  "Wekaで機械学習 (Java)"
date: 2017-01-20 22:24:09 +0900
categories: jekyll update
---

機械学習といえばPythonな世の中．
scikit−learnみたいに，多くのアルゴリズムがすでに実装されていて，使うのも楽です．
自分でアルゴリズムを実装したければ，numpyなどの高速な数値計算ライブラリを使うこともできます．
そう，アルゴリズム部分はPythonでも十分．
しかし，前処理後処理部分は遅いと思います．
こういう部分は他のコンパイル言語に任せた方がいいような気がします．
でも，前処理後処理だけ他の言語で，機械学習部分はPythonみたいになんかごちゃごちゃしてしまう問題もあります．
今回は，一つの選択肢としてJavaで機械学習をしてみます．

Javaの機械学習系ライブラリはいくつかありますが，今回は有名なWekaを使ってみます．
本当に日本語の「使ってみた」と言うページが少なくて驚きました．
というわけで，この記事が少しでも役に立てば幸いです．
今回扱うのは分類問題で，交差検定でパラメータを決めることもします．

まず，WekaではInstancesを作成する必要があります．
これは，データセットに対応するもので，Instanceのリストみたいなものです．
これをうまく作る方法を模索したのですが，あまり良い方法は思いつきませんでした．
というのも，Instancesを作る際に，次元数などを事前に決めてやる必要があるのです．
言語処理などでは，データを読み込んでいく過程で，次元数が徐々に膨れ上がってくるものですが，こういうのが表現できません．
なので，最初はRaw dataとして，MapなりListなりで生のデータを保存していきます．
data.csvからRaw dataを生成するには，例えば以下のようになります．
ラベルはStringしか対応していないようです（多分）．

{% highlight java %}
Path path = Paths.get("/path/to/data.csv");

List<double[]> X = new ArrayList<>();
List<String> y = new ArrayList<>();
Files.lines(path).forEach(line -> {
    String[] cols = line.split(",");
    double[] xi = Arrays.stream(Arrays.copyOfRange(cols, 0, cols.length - 1))
            .mapToDouble(Double::parseDouble).toArray();
    String yi = cols[cols.length - 1];
    X.add(xi);
    y.add(yi);
});
{% endhighlight %}


こんな感じで，scikit-learnを使うときみたいに，Xとyを生成したら，以下の関数にこれを渡してInstancesを得ます．

{% highlight java %}
private Instances createDataset(List<double[]> X, List<String> y) {
    Set<String> labels = new HashSet<>();
    labels.addAll(y);

    int numberOfSamples = X.size();
    int numberOfFeatures = X.get(0).length;
    int numberOfClasses = labels.size();

    ArrayList<Attribute> attributes = new ArrayList<>();
    IntStream.range(0, numberOfFeatures).forEach(i -> {
        attributes.add(new Attribute(Integer.toString(i)));
    });
    List<String> classes = labels.stream().collect(Collectors.toList());
    attributes.add(new Attribute("class", classes));

    Instances data = new Instances("", attributes, 0);
    data.setClassIndex(data.numAttributes() - 1);
    IntStream.range(0, numberOfSamples).forEach(i -> {
        double[] xi = X.get(i);
        String yi = y.get(i);
        Instance instance = new DenseInstance(numberOfFeatures + 1);
        instance.setDataset(data);
        IntStream.range(0, xi.length).forEach(j -> {
            instance.setValue(j, xi[j]);
        });
        instance.setClassValue(yi);
        instance.setWeight(1.0);
        data.add(instance);
    });

    return data;
}
{% endhighlight %}

さて，これでInstancesができました．
あとはこれを学習アルゴリズムに渡して学習させます．
今回は<a href="http://weka.sourceforge.net/doc.dev/weka/classifiers/functions/SGD.html" target="_blank">weka.classifiers.functions.SGD</a>を使います．ただ，これだけだとマルチクラスに対応できないので，<a href="http://weka.sourceforge.net/doc.dev/weka/classifiers/meta/MultiClassClassifier.html" target="_blank">weka.classifiers.meta.MultiClassClassifier</a>で包みます．
交差検定でパラメータを調整して，最終的なモデルを作ります．この際，numpy.linspace, numpy.logspaceなどのメソッドがあると便利なので，以下のように自前実装します．

{% highlight java %}
public static double[] linspace(double from, double to, int n) {
    double step = (to - from) / (n - 1);
    return IntStream.range(0, n).boxed().mapToDouble(i -> from + i * step).toArray();
}

public static double[] logspace(double from, double to, int n) {
    return DoubleStream.of(linspace(from, to, n)).map(k -> Math.pow(10, k)).toArray();
}
{% endhighlight %}

これで準備は整ったので，以下のように交差検定によるパラメータ調整を実装します．
今回調整するのは，正則化パラメータのみです．

{% highlight java %}
int randomState = 1;
int numberOfFolds = 5;
double[] lambdas = logspace(-4, 1, 10);

Random random = new Random(randomState);
Instances randData = new Instances(data);
randData.randomize(random);
randData.stratify(numberOfFolds);

double best_lambda = 0.001;
double minimum_error = 1.0;

for (double lambda : lambdas) {
    double[] scores = new double[numberOfFolds];
    for (int k = 0; k < numberOfFolds; k++) {
        Instances train = randData.trainCV(numberOfFolds, k);
        Instances test = randData.testCV(numberOfFolds, k);

        SGD base = new SGD();
        base.setOptions(Utils.splitOptions(String.format("-F 1 -S 1 -R %f", lambda)));
        MultiClassClassifier clf = new MultiClassClassifier();
        clf.setClassifier(base);
        clf.buildClassifier(train);

        Evaluation eval = new Evaluation(train);
        eval.evaluateModel(clf, test);
        scores[k] = eval.errorRate();
    }
    double error = DoubleStream.of(scores).sum() / scores.length;
    System.out.println(String.format("%f: %f", lambda, error));
    if (error < minimum_error) {
        minimum_error = error;
        best_lambda = lambda;
    }
}
System.out.println(String.format("minimum error: %f", minimum_error));
System.out.println(String.format("best lambda: %f", best_lambda));

SGD base = new SGD();
base.setOptions(Utils.splitOptions(String.format("-F 1 -S 1 -R %f", best_lambda)));
MultiClassClassifier clf = new MultiClassClassifier();
clf.setClassifier(base);
clf.buildClassifier(data);
{% endhighlight %}

これで学習は完了です．
次に予測ですが，適当なinstanceを作って，以下のようにすればラベルと確率値を得ることができます．

{% highlight java %}
Instance instance = ...
double label = clf.classifyInstance(instance);
double[] probabilities = clf.distributionForInstance(instance);
{% endhighlight %}

もし，このモデルを保存したければ，weka.core.SerializationHelperを使って以下のようにするといいです．
また，バイナリ化してデータベースに保存することもできます．
MultiClassClassifierの所は適宜指定してください．

{% highlight java %}
SerializationHelper.write("/path/to/test.model", clf); // ファイルに保存
MultiClassClassifier model = (MultiClassClassifier) SerializationHelper.read("/path/to/test.model"); // ファイルから読み込み

// byte配列に変換
ByteArrayOutputStream stream = new ByteArrayOutputStream();
SerializationHelper.write(stream, clf);
byte[] modelBinary = stream.toByteArray();
// byte配列から読み込み
ByteArrayInputStream inputStream = new ByteArrayInputStream(modelBinary, 0, modelBinary.length);
MultiClassClassifier model = (MultiClassClassifier) SerializationHelper.read(inputStream);
{% endhighlight %}

基本的な使い方は以上になります．
Wekaを使えば意外と簡単にJavaで機械学習ができることがわかりました．
