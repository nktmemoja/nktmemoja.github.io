---
layout: post
title:  "tensorflow.contrib.crfで固有表現抽出"
date: 2016-12-29 04:16:51 +0900
categories: jekyll update
---

↓にあるデータを使わせていただきました．

- <a href="http://qiita.com/Hironsan/items/326b66711eb4196aa9d4" target="_blank">【チュートリアル】機械学習を使って30分で固有表現抽出器を作る</a>

500件中，450件を学習に，50件をテストに用いました．なお，辞書も学習データのみで構築しています．つまりテストデータには未知語が含まれています．
結果を以下に示します．やはり未知語が多いと予測が辛そうです．ただ，未知語であっても予測できているケースもあります．
今回はデータ数が少ないので，仕方ないかもしれません．次回はもっと大規模なデータでやりたいですね．
コーパスを作らないといけないわけですが...．
コードのアルゴリズム部分は最後に載せておきます．
今回も短文だ...．長文を書く元気が欲しいです．

{% highlight bash %}
元の文:  昨年10月には、34人が、今回の現場に近いエジプトのタバで爆発事件のため死亡している。
正解の固有表現: 昨年10月, エジプト, タバ
抽出した固有表現: 昨年10月, エジプト
未知語: 34, 近い, タバ
====================
元の文:  タバはイスラエルからの観光客に人気のあるリゾート地である。
正解の固有表現: タバ, イスラエル
抽出した固有表現: イスラエル
未知語: タバ
====================
元の文:  このため、当局は今回の爆発はイスラエル・パレスチナ間の緊張にかかわるものとして非難している。
正解の固有表現: イスラエル, パレスチナ
抽出した固有表現: イスラエル, パレスチナ
未知語:
====================
元の文:  地元の警察によれば少なくとも30人が死亡、100人以上が負傷した。
正解の固有表現:
抽出した固有表現:
未知語:
====================
元の文:  BBCの報道によれば、少なくとも49人が死亡した。
正解の固有表現: BBC
抽出した固有表現: BBC
未知語:
====================
元の文:  ロイター通信社報道局は、現場にいた複数の医師が、負傷者の多くは危険な状態にあると語っていると伝えた。
正解の固有表現: ロイター通信社報道局
抽出した固有表現: ロイター通信社
未知語: ロイター通信, 報道局, 危険
====================
元の文:  またロイターでは爆弾はおそらく車に仕掛けられていたと報じた。
正解の固有表現: ロイター
抽出した固有表現:
未知語: ロイター
====================
元の文:  死亡者の中には、イギリス人、フランス人、スペイン人、オランダ人、カタール人、クウェート人、エジプト人が含まれているとAP通信が伝えている。
正解の固有表現: イギリス, フランス, スペイン, オランダ, カタール, クウェート, エジプト, AP通信
抽出した固有表現: イギリス, フランス, スペイン, オランダ, エジプト, AP通信
未知語: カタール, クウェート
====================
元の文:  アメリカのマイクロソフトは現地時間2005年7月22日、同社のオペレーティングシステム（OS）、WindowsXPの次にリリースされるOSの正式名称を「WindowsVista」（ウィンドウズ・ビスタ）にすると発表した。
正解の固有表現: アメリカ, マイクロソフト, 2005年7月22日, WindowsXP, WindowsVista, ウィンドウズ・ビスタ
抽出した固有表現: アメリカ, マイクロソフト, 2005年7月22日
未知語: オペレーティングシステム, OS, Windows, XP, 次に, OS, Windows, Vista, ウィンドウズ, ビスタ
====================
元の文:  それと同時に、WindowsVistaの公式サイトが開設され、追って日本語版のサイトも開設された。
正解の固有表現: WindowsVista
抽出した固有表現:
未知語: Windows, Vista, 開設, 追って, 開設
====================
元の文:  Vistaとは、展望、眺望、見通し、予想、美しい景色などを意味する単語である。
正解の固有表現:
抽出した固有表現:
未知語: Vista, 展望, 眺望, 見通し, 美しい, 景色, 意味, 単語
====================
元の文:  同OSの開発コードネームは「Longhorn」（ロングホーン）で、正式名称が発表されるまではそのコードネームで呼ばれていた。
正解の固有表現: Longhorn, ロングホーン
抽出した固有表現:
未知語: OS, Longhorn, ロング, ホーン
====================
元の文:  同社は、開発者や技術者向けのベータ版を2005年8月3日（日本時間:8月4日）頃にリリースする予定と発表している。
正解の固有表現: 2005年8月3日, 日本, 8月4日
抽出した固有表現: 2005年8月3日, 日本時間:8月4日
未知語: ベータ
====================
元の文:  気象庁によると、23日午後4時35分（以下本文はすべて日本時間、UTC+9)ごろ、関東地方・甲信越など広い地域で最大震度5強の地震があった。
正解の固有表現: 気象庁, 午後4時35分, 日本, 関東, 甲信越
抽出した固有表現: 気象庁, 午後4時35分, 日本, 関東
未知語: 本文, 甲信越, 広い, 震度, 強, 地震
====================
元の文:  震源は千葉県北西部、震源の深さは約73km、地震の規模を示すマグニチュードは6.0と推定している。
正解の固有表現: 千葉県
抽出した固有表現: 千葉県
未知語: 震源, 北西, 震源, 深, 73, 地震, 規模, 示す, マグニチュード, 推定
====================
元の文:  この地震による津波の心配はない。
正解の固有表現:
抽出した固有表現:
未知語: 地震, 津波, 心配
====================
元の文:  気象庁が発表した各地の主な震度は次のとおり。
正解の固有表現: 気象庁
抽出した固有表現: 気象庁
未知語: 震度, とおり
====================
元の文:  Yahoo!路線情報等による、記事発行時（23時）の状況は次の通り。
正解の固有表現: Yahoo!路線情報
抽出した固有表現: Yahoo!, 23時
未知語: Yahoo, 路線, 記事
====================
元の文:  東京・埼玉・千葉・神奈川で2時間あまり通話が規制された。
正解の固有表現: 東京, 埼玉, 千葉, 神奈川
抽出した固有表現: 東京, 埼玉, 千葉, 神奈川
未知語: あまり, 通話
====================
元の文:  エレベーターの中に人が閉じ込められる被害が4都県で46件。
正解の固有表現:
抽出した固有表現:
未知語: エレベーター, 閉じ込め, 都県
====================
元の文:  すべて救出済み。
正解の固有表現:
抽出した固有表現:
未知語: 救出, 済み
====================
元の文:  （22時現在）
正解の固有表現: 22時
抽出した固有表現: 22時
未知語:
====================
元の文:  三郷市のカラオケ店では、ガラスの破片が刺さり女性がけがをしたとの情報。
正解の固有表現: 三郷市
抽出した固有表現: 三郷市
未知語: 三郷, カラオケ, 破片, 刺さり, けが
====================
元の文:  鴻巣市のホームセンターでは落ちた案内板で客5人が軽傷を負った。
正解の固有表現: 鴻巣市
抽出した固有表現: 鴻巣市
未知語: 鴻巣, ホームセンター, 案内, 板, 軽傷, 負っ
====================
元の文:  新宿区・品川区・足立区・豊島区では、6箇所でエレベーターが停止し人が閉じ込められているとの情報。
正解の固有表現: 新宿区, 品川区, 足立区, 豊島区
抽出した固有表現: 品川区, 足立区, 豊島区
未知語: 品川, 足立, 豊島, 箇所, エレベーター, 閉じ込め
====================
元の文:  足立区では熱湯がかかりやけどをしたという通報があったとの情報。
正解の固有表現: 足立区
抽出した固有表現: 足立区
未知語: 足立, 熱湯, かかり, やけど
====================
元の文:  目黒区の民家では火災が発生。
正解の固有表現: 目黒区
抽出した固有表現: 目黒区
未知語: 目黒, 民家
====================
元の文:  けが人なし。
正解の固有表現:
抽出した固有表現:
未知語: けが人, なし
====================
元の文:  江戸川区では鉄塔が倒れ高圧線が落下。
正解の固有表現: 江戸川区
抽出した固有表現: 江戸川区
未知語: 江戸川, 鉄塔, 高圧線
====================
元の文:  隣家の屋根などが焦げたがけが人はなし。
正解の固有表現:
抽出した固有表現:
未知語: 隣家, 屋根, 焦げ, けが人, なし
====================
元の文:  江東区の立体駐車場で2階から乗用車が地上に落ちた。
正解の固有表現: 江東区
抽出した固有表現: 江東区
未知語: 江東, 立体, 駐車, 階
====================
元の文:  けが人なし。
正解の固有表現:
抽出した固有表現:
未知語: けが人, なし
====================
元の文:  毎日新聞によると、23日21時2分現在で、東京・埼玉・千葉・神奈川の4都県で計29人が重軽傷を負った。
正解の固有表現: 毎日新聞, 21時2分, 東京, 埼玉, 千葉, 神奈川
抽出した固有表現: 23日21時2分, 東京, 埼玉, 千葉, 神奈川
未知語: 毎日新聞, 都県, 計, 重軽傷, 負っ
====================
元の文:  22日英語版ウィキニュースによると、
正解の固有表現: 22日, ウィキニュース
抽出した固有表現: 22日, ウィキニュース
未知語:
====================
元の文:  イギリスのロンドン首都警察は南ロンドンのストックウェルにある住宅で容疑者を逮捕しようとしたと発表し、「これは21日の爆破事件の捜査の一環である」と語った。
正解の固有表現: イギリス, ロンドン, 南ロンドン, ストックウェル, 21日
抽出した固有表現: イギリス, ロンドン首都警察, ロンドン, 21日
未知語: ストックウェル, 住宅, 一環
====================
元の文:  爆発現場付近で監視カメラ網（CCTV)のカメラが捉えた男性の写真4枚と、この容疑者が同一人物かどうかについては、当局はコメントしない意向である。
正解の固有表現: CCTV
抽出した固有表現:
未知語: 監視, カメラ, 網, CCTV, カメラ, 捉え, 写真, 枚, 一人物, どう
====================
元の文:  また同日現地時間午後6時すぎ、バーミンガム市中心部の鉄道駅スノウ・ヒル駅でテロ法44項により警察が男性1名を逮捕した。
正解の固有表現: 午後6時, バーミンガム市, スノウ・ヒル駅, テロ法44項
抽出した固有表現: 午後6時, バーミンガム市, 44
未知語: バーミンガム, 鉄道, スノウ・ヒル, 項
====================
元の文:  駅構内で2個の疑わしいスーツケースが発見されたため、駅は非常線が張られ人は退避させられていた。
正解の固有表現:
抽出した固有表現:
未知語: 構内, 疑わしい, スーツケース, 張ら, 退避
====================
元の文:  スーツケースは調査され、爆発物は入っていなかったことが判明した。
正解の固有表現:
抽出した固有表現:
未知語: スーツケース, 判明
====================
元の文:  逮捕された男性は不起訴になるだろうとBBCは報道した。
正解の固有表現: BBC
抽出した固有表現: BBC
未知語:
====================
元の文:  スノウ・ヒル駅は現時時間午後8時に再び開放された。
正解の固有表現: スノウ・ヒル駅, 午後8時
抽出した固有表現: 午後8時
未知語: スノウ・ヒル, 現時, 再び, 開放
====================
元の文:  レバノンの位置
正解の固有表現: レバノン
抽出した固有表現:
未知語: レバノン
====================
元の文:  23日、レバノンの首都ベイルートのレストラン街で爆発があり、少なくとも12人が負傷した。
正解の固有表現: 23日, レバノン, ベイルート
抽出した固有表現: 23日
未知語: レバノン, ベイルート, レストラン, 街
====================
元の文:  22日よりベイルートを訪問していたライス米国務長官が出国してから2時間後だった。
正解の固有表現: 22日, ベイルート, 米, 2時間後
抽出した固有表現: 22日, 米
未知語: ベイ, ルート, 訪問, ライス, 国務, 長官, 出国
====================
元の文:  22日、イスラエルより専用機でレバノン入りしたライス長官の電撃訪問は、今年始まったシリアのレバノン撤兵とレバノン新政権への支援を確実なものにする目的で行われた。
正解の固有表現: 22日, イスラエル, レバノン, ライス, 今年, シリア, レバノン, レバノン
抽出した固有表現: 22日, イスラエル, 今年
未知語: レバノン, ライス, 長官, 電撃, 訪問, シリア, レバノン, 撤兵, レバノン, 確実, 目的
====================
元の文:  爆発は午後10時頃レストランの前で起こり、3台の乗用車を破壊した、と治安当局者はAP通信に語った。
正解の固有表現: 午後10時, AP通信
抽出した固有表現: 午後10時, AP通信
未知語: レストラン, 台
====================
元の文:  爆弾は路上に駐車した乗用車の下または中に仕掛けられたと推測されている。
正解の固有表現:
抽出した固有表現:
未知語: 路上, 駐車, 下, または, 推測
====================
元の文:  場所は、東ベイルートのキリスト教徒居住地区の境界部にあたる私立・聖ヨハネ大学の付近である。
正解の固有表現: 東ベイルート, 私立・聖ヨハネ大学
抽出した固有表現: ヨハネ大学
未知語: 東, ベイルート, キリスト教徒, 境界, にあたる, 私立, 聖, ヨハネ
====================
元の文:  地元のLBCテレビは当初、主に飛散したガラスの破片により6名が負傷し、乗用車の所有者は地元の住民ヨセフ・ナディムたと報じた。
正解の固有表現: LBCテレビ, ヨセフ・ナディム
抽出した固有表現:
未知語: LBC, 飛散, 破片, 所有, 住民, ヨセフ・ナディム
====================
{% endhighlight %}



{% highlight python %}
# n: 学習データ数
# d: 次元数
# m: 最大系列長（適当に決めてやる必要がある．実際の系列長はsで与える）
# c: クラス数
# n_dim: 隠れ層のユニット数

n_dim = 100
sp_indices = tf.placeholder(tf.int64)
sp_values = tf.placeholder(tf.float32)
sp_shape = tf.placeholder(tf.int64)
X = tf.SparseTensor(sp_indices, sp_values, sp_shape)
y = tf.placeholder(tf.int32, [n, m])
s = tf.placeholder(tf.int32, [n])

W1 = tf.Variable(tf.random_normal([d, n_dim]))
b1 = tf.Variable(tf.random_normal([n_dim]))
W2 = tf.Variable(tf.random_normal([n_dim, n_dim]))
b2 = tf.Variable(tf.random_normal([n_dim]))
w = tf.Variable(tf.random_normal([n_dim, c]))

mX = tf.sparse_reshape(X, [-1, d])
z1 = tf.nn.relu(tf.sparse_tensor_dense_matmul(mX, W1) + b1)
mZ = tf.matmul(z1, W2) + b2

sp_indices_test = tf.placeholder(tf.int64)
sp_values_test = tf.placeholder(tf.float32)
sp_shape_test = tf.placeholder(tf.int64)
X_test = tf.SparseTensor(sp_indices_test, sp_values_test, sp_shape_test)
mX_test = tf.sparse_reshape(X_test, [-1, d])
z1_test = tf.nn.relu(tf.sparse_tensor_dense_matmul(mX_test, W1) + b1)
mZ_test = tf.matmul(z1_test, W2) + b2

unary_scores_test = tf.reshape(tf.matmul(mZ_test, w), [n_test, m, c])

unary_scores = tf.reshape(tf.matmul(mZ, w), [n, m, c])
log_likelihood, transition_params = tf.contrib.crf.crf_log_likelihood(unary_scores, y, s)
loss = tf.reduce_mean(-log_likelihood)
train_op = tf.train.AdamOptimizer(0.1).minimize(loss)

init = tf.global_variables_initializer()
with tf.Session() as sess:
    sess.run(init)
    for i in range(1000):
        tf_unary_scores, tf_transition_params, _ = sess.run([unary_scores, transition_params, train_op],
                                                            feed_dict={
                                                                sp_indices: indices_train,
                                                                sp_values: values_train,
                                                                sp_shape: (n, m, d),
                                                                y: y_train_,
                                                                s: s_train_,
                                                                keep_prob: 0.5
                                                    }
        )
        if i % 20 == 0:
            correct_labels = 0
            total_labels = 0
            for tf_unary_score, yy, ss in zip(tf_unary_scores, y_train_, s_train_):
                tf_unary_score = tf_unary_score[:ss]
                yy = yy[:ss]

                viterbi_sequence, _ = tf.contrib.crf.viterbi_decode(tf_unary_score, tf_transition_params)

                correct_labels += np.sum(np.equal(viterbi_sequence, yy))
                total_labels += ss
            accuracy = 100.0 * correct_labels / float(total_labels)
            print("Accuracy: %.2f%%" % accuracy)
    print("training done")

    tf_unary_scores_test = sess.run(unary_scores_test,
                               feed_dict={
                                   sp_indices_test: indices_test,
                                   sp_values_test: values_test,
                                   sp_shape_test: (n_test, m, d),
                                   keep_prob: 1.0
                               }
    )
    for tf_unary_score_test, yy, ss, ws in zip(tf_unary_scores_test, y_test_, s_test_, word_lists_test):
        tf_unary_score_test = tf_unary_score_test[:ss]
        yy = yy[:ss]

        viterbi_sequence, _ = tf.contrib.crf.viterbi_decode(tf_unary_score_test, tf_transition_params)
        print("元の文: ", "".join(ws))
        entities = []
        entity = ""
        for word, label in zip(ws, yy):
            if label == 1:
                entity = word
            elif label == 2:
                if entity:
                    entity += word
            else:
                if entity:
                    entities.append(entity)
                    entity = ""
        print("正解の固有表現:", ", ".join(entities))
        entities = []
        entity = ""
        for word, label in zip(ws, viterbi_sequence):
            if label == 1:
                entity = word
            elif label == 2:
                if entity:
                    entity += word
            else:
                if entity:
                    entities.append(entity)
                    entity = ""
        print("抽出した固有表現:", ", ".join(entities))
        print("未知語:", ", ".join(word for word in ws if word not in word2id))
        print("="*20)
{% endhighlight %}