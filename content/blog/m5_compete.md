---
title: "前提知識ゼロで理解する M5コンペ"
author: "chizuchizu"
date: 2020-07-06T06:12:01+09:00
draft: false
description: "KaggleのM5コンペを誰でもわかるように"
categories: ["Kaggle"]
tags: ["Kaggle"]
---



深夜テンションで思いついたものです。この記事が公開されるかどうかもわかりませんが、とりあえず書きます。

# 目的や対象など

## 対象者

誰でもです。前提知識ゼロでも理解できるように書きました。

## 目的

意外と発想が面白かったり感覚的なところがあるということを知ってもらいたい。

自分用のまとめも兼ねてます。

# ゼロから理解するM5

## Kaggleとは

世界最大のデータサイエンスプラットフォームです。

このスライドの開設がわかりやすいです。細かいところは省きますが、「目的の何か」の達成率等を数値化して競うものです。

<iframe src="//www.slideshare.net/slideshow/embed_code/key/r5UBmaPuheL2pF?startSlide=5" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/HaradaKei/devsumi-2018summer" title="Devsumi 2018summer" target="_blank">Devsumi 2018summer</a> </strong> from <strong><a href="//www.slideshare.net/HaradaKei" target="_blank">Harada Kei</a></strong> </div>



## M5とは何なのか

コンペのURLは貼っておきますが、英語だけなので読むのも辛いかも。

https://www.kaggle.com/c/m5-forecasting-accuracy/overview

何でM5なんでしょうね。（関係ないですけど留年した修士課程の人ですか？と言われたことがある）

このコンペは日本で言う西友の親会社「ウォルマート」が開催しています。今回は5回目なのでM5です。Mはウォルマートと手を組んだ研究機関MOFCの頭文字からとったらしい。

## 目的

ウォルマートのすべての店のすべてのアイテムの売上を過去5年分あげるから1ヶ月予測してね。

### 謎の評価関数「WMRSSE」

評価関数っていうのは「精度」の決め方です。RMSEやMAEは聞いたことがある人もいるかも？

**Weighted Root Mean Squared Scaled Error**らしいんですけど、日本語に直すと「重み付き平方根平均正規化誤差」

要するに売上が大きく店の収益にどれだけ影響されるかによってその重みが違っていて、毎日買うような食べ物はおももが高くおもちゃなどは重みが低いです。この指標、ややこしくて計算するだけでもかなり時間がかかります。（0.2秒くらい）

0が完璧です。大体0.5~0.6が普通です。（小さいほど良い）

### 配られるデータ

- price
- event(祝日のデータやNFL開催日など)
- 日付
- 売上
- id(店、アイテムのカテゴリ、アイテム)

これだけ！？！？！？

### 期間の話

1~1913｜1914~1941｜1942~1969　（日）

学習｜public｜private

#### public privateとは

public：自分を含めみんなが見られる公開されたスコア

private：コンペが終わるまでわからないスコア（これで最終的な順位が決まる）

今回はpublicとprivateがそれぞれ28日あります。いくらpublicがよくてもprivateでスコアが超悪い可能性だってあります。



## アプローチ

3つ紹介します。どれも個性的です。ベースとしては1913日のデータを使って次の日を予測するというモデルを使います。時系列モデルというとARIMAといったモデルもありますが、適当に汎用的なモデルに突っ込めば上手くいってます。（何でこんなにうまく行くか不思議でしょうがない）

### single model

![](/img/main/Screenshotfrom-2020-07-06-18-07-10.png)

万物の根源とも言えるようなシンプルさ。

### day by day

![](/img/main/Screenshotfrom-2020-07-06-18-07-10.png)

デイ・バーイ・デイ　ARASHIですかね。いいえ、違います。

図だけで説明できるかわからないのですが、簡単に言うとsingle modelの強化版です。

基本的に学習に使うデータは最近のもののほうが良いです。single modelだと28日を予測するのでどうしても使うデータは28日より前のデータになってしまいます。ただ、day by dayだと1日目は昨日のデータを使うことができますし（後半になるほどsingle modelの性能に近づく）良いモデルっぽいです。ただ、28日間を予測しなきゃいけないのでモデルも28個必要で学習に時間が必要です。（組んでいるチームだと18時間以上かかってた）

日ごとなのでday by dayらしいです。（知らんけど）

### recursive

![](/img/main/Screenshotfrom-2020-07-06-18-22-14.png)

急にナックルを投げ出したピッチャー並の異端児感があります。

recursive（リカーシブ）というのは「再帰」という日本語の意味になります。再帰ってのは自分で自分を呼び出すという端的にぐるぐる回るノリです。

さきほどday by dayでも言ったように使うデータは直近のほうが予測しやすいんですよね。そしたら明日を予測するモデルを作って尺取り虫みたいに28日予測すればいいんじゃね？的な発想です。

明日を予測するモデルの精度に依存すると思うんですけど、最初から誤差が多ければ後に響くので色んな意味で危ないモデルです。

結局の所publicでは超性能良かったけどprivateで堕落したモデルです😢

これ思いついた人かなり凄いと思ってます。

自分が調べた中だとこのrecursiveモデルの誤差を上手く扱うために誤差込みで学習する方法もあるらしいです。

### 結局何がいいの

結果論ですが、recursiveとday by day(single model)の混ぜ合わせ（アンサンブル）がよく効くらしいです。(1位の人がこうしてた)

publicだとrecursive最高！だったのにprivateだとday by day最高！！！という不思議なデータだったので……（うちのチームだとday by dayが最高性能出してたけど最終提出はrecursiveと組み合わせたことによって下がってた）

## データ加工など

いわゆる予測に使うデータは特徴量と呼ばれています。そのデータの特徴です。このような時系列のモデルだと"shift"や"rolling"が使われます。（なんですかそれ…）

### そもそも論

このコンペの学習データは最終的に30GB近くなってしまいました。理由は色々ありますが、主な理由は1つです。

アバウトに計算します。

2000(日) x 30000(アイテム) = 6000万（行）

6000万行のデータを扱うので実行時間もかかりますし、メモリも必要ですし色々大変です。Σ(ﾟ∀ﾟﾉ)ﾉｷｬｰ

### ラグ特徴量

ラグ、ずらす特徴量です。基本的に売上高を使うので未来の情報を使わないように気をつけないといけません。（いわゆるleak）

#### SHIFT (ずらす)

SIMPLE IS THE BEST

n日ずらすだけです。過去の売上の情報が無いと今日や未来の売上は予測できませんよね。そんな感じです。

一つずらすと　1, 2, 3, 2, 3→None, 1, 2, 3, 2　となります。（最初は何も無いので空）

#### 移動平均

グラフを滑らかにするために使います。どんな感じかというのは調べて見ると色々出てきます！

今回これがめっちゃ効いてます。過去7日間の売上の平均や標準偏差（ばらつき）といったものですね。

新たな移動平均の特徴をまとめた列をaとして添字（右下に書いてある数）を日付として過去3日間の移動平均はこのように算出されます。
$$
\frac{x_1 + x_2 + x_3}{3} = a_4
$$
やってることはかなりシンプルです。

#### 余談　recursiveモデルでshiftを使わない理由

使えるには使えるのですが、スコアがダダ下がりします。何でだと思いますか？

簡単に言うとoverfit（過剰適合）してしまうからです。というのも、recursiveは途中から予測値を頼りに次の値を予測しています。予測値というのはどうしても誤差が生じてしまうものです。shiftというのは単に一つのデータしか使わないですよね。

とりあえずshiftの話は置いておいて、shiftは使いませんが移動平均の場合も同様に考えてみましょう。誤差というのはプラスもあればマイナスもあります。例えば7日間の移動平均を取ったとしてそれらが正解の値付近で上下していたとしたら移動平均を取ったらほぼ完璧な値になりますよね。そういう意味で移動平均はshiftよりも信頼できます。（実際に実験してみてもshiftを使うと精度が悪くなってしまう）

### メモリや時間を上手く付き合うコツ

30GB近くあるので、一気に学習することは（僕のパソコンだと）無理です。30GBあったとしても実際は100GBくらいないとパソコンがフリーズしてしまいます。特にKaggle notebook上だと16GBの制限があるのでできることがかなり限られてきます。

- お店ごと(10店舗あるの）に学習させることで負担を減らす
- メモリを食わないような型を指定してあげる
- メモリを増やす



時間のほうだと

- csvを使わない
- 並列処理をする
- 特徴量を選択する

などです。（自分が気付いてないだけで色々あるのかもしれない）

csvではなくてpickleを使いましょう。（といってもほとんどの人はは？だと思いますが）

ちなみにcsvでやると読み込みだけで数十分必要になりますが、pickleだと数秒です。

マシンスペックにも依りますが16スレッドあれば同時で16プロセスを動かせるので処理が速くなります。パソコンは偉大。

## 結局M5は何だったのか

思ったことを箇条書きで書きます

- 難易度の割に参加者が多い(エントリーだけで9万人近く)
- モデルが面白い（特にrecursive）
- 大規模データの扱いに慣れた
- 謎(上から50番以内にkaggle最高位であるGMが誰一人といない)
- わからない

結局わからなかったです。シンプル・イズ・ベストのようです。



もう一つ、日本人の活躍も見てみましょう。上位300位以内のチームの約3割が日本人らしいです（うちのチームも含まれています）。Kaggle人口かなり増えてるじゃねぇか！（巷だとKaggleで勝つデータ分析の技術という本の影響もあるとか）

https://www.kaggle.com/c/m5-forecasting-accuracy/discussion/163410

![kaggle_300](/img/main/kaggle_300.png)

# おわりに

不可解なコンペでしたが、4ヶ月という長い間このコンペに尽力してきました。謎でしたが、色々とあって今思えば楽しかったです。

これがKaggleを初めて知った人や敷居が高いと思ってる人に届けば良いと思っています。興味を持ってくれたり面白いと思ってくれたら嬉しいです！

このコンペの経験も後にどこかで役に立つと思うのでこれも良い糧として吸収しておきます。