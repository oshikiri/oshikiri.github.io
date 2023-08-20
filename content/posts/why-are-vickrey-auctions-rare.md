+++
title = "なぜセカンドプライスオークションはあまり使われないのか？"
date = "2018-03-17"
description = "1990年に発表された、“Why Are Vickrey Auctions Rare?” というタイトルの論文の紹介。セカンドプライスオークション（Vickrey auction）が**当時**あまり使われていなかった理由を検討している。"
+++

>Rothkopf, Michael H., Thomas J. Teisberg, and Edward P. Kahn. **"Why Are Vickrey Auctions Rare?"** Journal of Political Economy 98, no. 1 (1990): 94-109. <http://www.jstor.org/stable/2937643>.


## 内容

（説明が面倒なので、Vickrey auction の定義と性質については割愛する）

[Vickrey auction](https://en.wikipedia.org/wiki/Vickrey_auction) （sealed-bid second-price auction、あるいは単に[セカンドプライスオークション](https://ja.wikipedia.org/wiki/%E7%AB%B6%E5%A3%B2#%E5%B0%81%E5%8D%B0%E5%85%A5%E6%9C%AD%E6%96%B9%E5%BC%8F_(sealed_bid_auction))とも呼ばれる） は、ある仮定のもとで

* sealed-bid first-price auction
* English auction
* Dutch auction

の3つの形式のオークションと同じ収益が上げられること（収入等価定理）が1961年に示されていて、
また、ある仮定のもとで、入札者がそれぞれ感じた評価値を正直に入札する戦略（truth-revealing strategy）が最適な戦略であることが示されている。

これら著しい性質があるにも関わらず、Vickrey auction はこれら3つのオークションに比べて、（Vickrey の論文から29年経った）1990年当時あまり使われていなかった。

これは不思議だということで、著者たちが「Vickrey auction が使われない理由の候補」を7個挙げて検討している。

### Nonreasons

まず、著者たちが「説得力がない」と考えた理由（nonreasons）を5つ挙げて検討している。

1. **Multiple Objects for Sales** : Vickrey auction には望ましい性質がいくつかあるが、これは single item （オークションにかけられる品が単一の場合）の話で、multiple items の場合に一般に成り立たない。これが使われない理由ではないだろうか？しかし、 single item の場合も使われてないので、それは理由ではないだろう。
2. **Bidder Risk Aversion** : "risk-averse bidder" がいる場合、Vickrey auction より first-price auction の方が収益が高くなる（[参考](https://en.wikipedia.org/wiki/Auction_theory#Benchmark_model)）。これが理由ではないか？
3. **Bidder Asymmetry** : Vickrey の収入等価定理では、入札額の分布が *i.i.d.* であることを仮定していた。これが成り立たないとどうか？
4. **Nonindependent values** : 同様に、入札者それぞれの評価値が独立でない場合はどうか？後にもう少し緩和した仮定のもとで同じ結果が示されている。この場合、得られる収益が SP > FP になる[^notations]ため、セカンドプライスオークションがあまり使われないことの説明にはならない。
5. **Inertia** : 制度や法令などは一旦決まると変わりにくい。昔からよく使われていたオークション形式が、慣習としてそのまま残っているだけではないか？しかし、また別の新しいオークション方式が使われたりするので、これも説明になっていない。

[^notations]: FP = ファーストプライスオークションで得られる収益、 SP = セカンドプライスオークションで得られる収益

### Reasons

そして、著者たちが「これが理由だろう」と考えた理由は以下の2つである。

#### 6. Bidder Fear of Bid Taker[^bid-taker] Cheating

[^bid-taker]: bid taker = auctioneer

実際に使うオークション方式の選定においては、「最適性」より、不正への耐性が重要となる。ここで言う不正としては、1)入札者が共謀する、2)オークショニアがサクラの入札者を用意する、などの行為が挙げられる。セカンドプライスオークションがこの 2) の不正に弱い、というのが理由ではないか？

2) の具体例としては、次のような不正が挙げられる： 予約価格より高い入札が2件以上あったとして、入札終了後にオークショニアがファーストプライス b<sub>1</sub> とセカンドプライス b<sub>2</sub> を確認して、サクラに中間 (b<sub>2</sub> ≦ x ≦ b<sub>1</sub>) の金額 x で入札するように指示する。こうすることで、落札額を x - b<sub>2</sub> だけ増やすことができる。

このような不正が想定される場合（≒ オークショニアが信頼できない場合）、入札者にはそれぞれの評価値より安い額で入札する動機が発生する[^reasons]。
ちなみに Vickrey 自身もこのような問題点は予期していて、信頼できる第三者にオークションを開催してもらうことで解決できると言っている(出典不明)。

[^reasons]: 明示的には書かれていないが、おそらく最後に以下のような主張が隠れている： *これらの理由によって、truth-revealing strategy が入札者にとって最適ではなく、少し安めに入札する戦略（すなわち、ファーストプライスオークションで使われるような戦略）をわざわざ考える必要が生じるのであれば、最初からファーストプライスオークションを実施すれば良い。*


#### 7. Bidder Resistance to Truth-revealing Strategies

オークショニアが不正をしないとわかっている場合であっても、truth-revealing strategy に基づいて入札を行うと、機密情報である評価値がオークショニアにバレてしまう、という問題がある。

例えば、企業の代表としてオークションに参加するような状況を考えると、商品に対する評価値は機密情報であり、これを外部に公開する（つまり、正直に入札する）ことは強く禁止される。

では仮に、この「機密情報を外部に公開できない」という制限がなかった場合はどうか？実はこの場合でも問題は起こる。オークション理論では、当然オークションの開始から終了までしか考慮しないが、現実にはオークション後に後処理がある場合（詳細は不明だが、電力のオークションなどでこういうケースがあるらしい）があって、このような場合に truth-revealing strategy を採用すると後処理で不利になる可能性がある。

これを聞くと、「いや、オークショニアが winning bid を秘密に（しているので｜すれば）問題ないのでは？」という疑問が浮かぶが、

1. 公共事業などの場合は透明性の面で秘密にするのは難しいし、
2. オークショニアが秘密情報を持つことで権力を持ってしまったりして面倒だし[^power]、その秘密が漏れる可能性がある、という可能性があるだけで正直に入札しない動機にはなる

といった背景があるため、結局正直に入札しない動機が発生する[^reasons]。

[^power]: "Secret information tends to give power to its holder" とのことだった


## 感想

RTBの分野では、セカンドプライスオークションは（今のところは）デファクトスタンダードとなっている。「使われるようになったということは、先ほどの理由6,7を解決できたということか？」と思ってしまうが、別にそういうわけでもなさそう。

例えば、2017年に出た記事に、理由6に該当する話が書かれている。

>"Rather than setting price floors as a flat fee upfront, some SSPs are setting high price floors after their bids come in as a way to squeeze out more money from ad buyers who believe they are bidding into a second-price auction ..."
>
><https://digiday.com/marketing/ssps-use-deceptive-price-floors-squeeze-ad-buyers/>

また、理由7についても（当時の想定からややズレるが）、正直に入札すると reserve price optimization の材料として使われてしまうという問題点が残っており、こちらも解決できていないと思う。

1990年の論文で指摘されている問題点が放置されたまま(?)で、2017年の記事で再度指摘されている、というのは少し面白かった。
