+++
title = "Amazonはいわゆる研究開発費を公表していない"
date = "2020-10-21"
description = "Amazonの「研究開発費」として報じられる金額の出典は、“technology and content” というAmazon独自の項目である。しかし、この項目は、AWSの運用コストといった、研究開発費には通常含めないものを含んでいるため、他の会社との比較に使うことができない。"
+++

## Amazon の研究開発費

Amazonの研究開発費について、例えば下記のように、ニュースや日本経済再生本部(?)の資料などで言及されている。

- [アマゾン、世界研究開発費ランキングで2年連続1位 米企業が上位を占めるも、中国が台頭 \| JBpress（Japan Business Press）](https://jbpress.ismedia.jp/articles/-/54559)
  - 「（略）米アマゾン・ドットコムの年間研究開発（R＆D）費は、226億ドル（2兆5600億円）となり、世界上場企業1000社の研究開発費ランキングで、2年連続トップとなった。」とある。
- [未来投資会議（第３１回）配布資料](https://www.kantei.go.jp/jp/singi/keizaisaisei/miraitoshikaigi/dai31/index.html)
  - 資料2のp.7のグラフで、2018年のAmazonの研究開発費が3.2兆円と表示されている
- [Global Innovation 1000 \| Most Innovative Companies \| Strategy&](https://www.strategyand.pwc.com/gx/en/insights/innovation1000.html)
  - 2018年のAmazonの R&D expenditures が $22.6B と書いてある

ここで一応、研究開発費の定義を確認しておく。
日本で使われている「[研究開発費等に係る会計基準](https://www.fsa.go.jp/p_mof/singikai/kaikei/tosin/1a909e2.htm)」において、研究および開発は

>研究とは、新しい知識の発見を目的とした計画的な調査及び探究をいう。開発とは、新しい製品・サービス・生産方法（以下、「製品等」という。）についての計画若しくは設計又は既存の製品等を著しく改良するための計画若しくは設計として、研究の成果その他の知識を具体化することをいう。

と定義されており、研究開発費には「研究開発のために費消されたすべての原価が含まれる」とされている。

通常「研究開発費」というときに、この定義を念頭に置いている人はそこまで多くないとは思うが、
「2018年のAmazonの研究開発費が3.2兆円」というときは、Amazonはこの意味での研究開発費に3.2兆円使っている、という意味に解釈していいだろう[^sfas2-8ab]。

[^sfas2-8ab]: Amazonの決算報告書はアメリカの会計基準に従っている。
    そのため一応参照しておくと、アメリカの会計基準における "research" と "development" の定義は「研究開発費等に係る会計基準」と大きくは変わらないように見える（「[Statement of Financial Accounting Standards No. 2](https://www.fasb.org/resources/ccurl/286/565/fas2.pdf)」の p.5）。
    ただ、Amazonの決算報告書において "research and development" の項目はないため、アメリカの会計基準における定義をあえて参考にする必要もないだろう。

## Amazon の営業費用の内訳

Amazonの決算報告書は下記のページにある。

- [Amazon\.com, Inc\. \- Annual reports, proxies and shareholder letters](https://ir.aboutamazon.com/annual-reports-proxies-and-shareholder-letters/default.aspx)
- [Amazon 2018 Annual Report (Form 10-K)](https://s2.q4cdn.com/299287126/files/doc_financials/annual/2018-Annual-Report.pdf)[^why-2018]
  - 以後断りなくページ数を書いた場合は、このPDFの当該ページを指すこととする

[^why-2018]: 本当は最新の2019年の報告書を使いたかったが、なぜか文字のコピペに失敗するので2018年の資料を使うことにした。チラッと見た感じでは大きく内容が変わっているということはなさそうだった。

前述の2018年の決算報告書のp.25の表によれば、2016~2018年の営業費用 (operating expenses) の内訳は下記のようになっている（金額の単位は $1M、12月締め）：

 / | 2016 | 2017 | 2018
:--- | ---: | ---: | ---:
cost of sales (売上原価) | 88,265 | 111,934 | 139,156
fulfillment | 17,619 | 25,249 | 34,027
marketing (販売費) | 7,233 | 10,069 | 13,814
technology and content | 16,085 | 22,620 | 28,837
general and administrative (一般管理費) | 2,432 | 3,674 | 4,336
Other operating expense, net (その他) | 167 | 214 | 296
|
合計 | 131,801 | 173,760 | 220,466

営業費用の一覧で訳せなかった項目が2つある。

1つ目の "fulfillment" は、どうやら「[商品の受注から決済に至るまでの業務全般](https://kotobank.jp/word/%E3%83%95%E3%83%AB%E3%83%95%E3%82%A3%E3%83%AB%E3%83%A1%E3%83%B3%E3%83%88-687398)」という意味で使われているEC業界の用語らしい。
Amazonの決算報告書においては、商品の配送に関わる費用（フルフィルメントセンター、カスタマーサービスセンター、physical store）や決済関係の費用が該当する[^fulfillment]。

[^fulfillment]: p.26 の1段落目: "Fulfillment costs primarily consist of those costs incurred in operating and staffing our North America and International fulfillment centers, customer service centers, and physical stores and payment processing costs."

2つ目の "technology and content" を研究開発費と解釈すれば、例えば、「2017年にAmazonが研究開発費に$22,620M (≒2.5兆円) 計上している」と言える。
実際、Amazonの研究開発費に言及している記事の金額と "technology and content" の金額は一致しているので、一般的にはこの金額を「Amazonの研究開発費」として扱っていると考えられる。

では、"technology and content" の定義はなにで、この項目には具体的にどういうものが含まれるのだろうか。

## "technology and content" とはなにか

Amazonの決算書において、"technology and content" は、

>Technology and content costs include payroll and related expenses for employees involved in the research and development of new and existing products and services, development, design, and maintenance of our stores, curation and display of products and services made available in our online stores, and infrastructure costs. Infrastructure costs include servers, networking equipment, and data center related depreciation, rent, utilities, and other expenses necessary to support AWS and other Amazon businesses.

と定義されている (p.26)。
この定義によると、 "Technology and content costs" は、

1. payroll and related expenses for employees involved in the research and development of new and existing products and services
2. development, design, and maintenance of our stores
3. curation and display of products and services made available in our online stores
4. infrastructure costs

を含んでおり[^stanford-parser]、
この4番目の "infrastructure costs" は、AWSなどの事業に必要な、①サーバー、②ネットワーク機器、③データセンター関連の減価償却費・賃貸料・光熱費などを含んでいる、とされている。

いや普通に考えて、AWSの運用コストは売上原価として計上されているのでは？とも思ったが、
売上原価の項目においても、AWS セグメント[^segments]の費用は売上原価ではなく "technology and content" に計上されている、と明記されている[^cost-of-aws]。
どうやら、暗黙のうちに "cost of sales" を「（AWS以外の）EC事業と映像音楽事業の売上原価」という意味で使っているフシがある[^cost-of-sales]。

[^cost-of-aws]: p.25 の一番下の段落: "Costs to operate our AWS segment are primarily classified as “Technology and content” as we leverage a shared infrastructure that supports both our internal technology requirements and external sales to AWS customers."

[^cost-of-sales]: p.25の "Cost of Sales" の説明において、"Cost of sales primarily consists of the purchase price of consumer products, digital media content costs where we record revenue gross, including video and music, packaging supplies, sortation and delivery centers and related equipment costs, and inbound and outbound shipping costs, including where we are the transportation service provider." と書かれており、"primarily" がついているので断言できないが、AWSは含まれていなさそうに見える。

[^stanford-parser]: 1文目が複雑すぎて何回読み直しても解読できなかったので、[Stanford Parser](http://nlp.stanford.edu:8080/parser/index.jsp) を参考にした。

さて "Technology and content costs" について、
1,2,3は（判断がつかないものも紛れているが一旦）いいとして、少なくとも4についてはAWS事業の売上原価に該当するため、「研究開発費」に含めるべきではないだろう。
つまり、この "technology and content" を「研究開発費」として取り上げて、他の会社の研究開発費と比較するのは、不適切だろう。

## Amazon 以外の会社ではどうか？

では、Amazon以外の会社はどうしているか？
ここでは、Alphabet とトヨタ自動車の決算報告書を見てみる。

### Alphabet

Alphabet[^alphabet-ir]（Google の持株会社）
の売上原価と研究開発費は、
[FORM 10-K (2018)](https://www.sec.gov/Archives/edgar/data/1652044/000165204419000004/goog10-kq42018.htm) の p.47
にある（単位は$1M、12月締め）：

[^alphabet-ir]: [Alphabet Investor Relations \- Investor Relations \- Alphabet](https://abc.xyz/investor/)

 / | 2016 | 2017 | 2018
 :--- | ---: | ---: | ---:
Cost of revenues | 35,138 | 45,583 | 59,549
Research and development | 13,948 | 16,625 | 21,419

この研究開発費の項目は、以下の項目から構成されている（Alphabet 10-K の p.33）：

> - Compensation expenses (including SBC) and facilities-related costs for engineering and technical employees responsible for R&D of our existing and new products and services;
> - Depreciation expenses;
> - Equipment-related expenses; and
> - Professional services fees primarily related to consulting and outsourcing services.

ちなみに、データセンターの費用は "Cost of Revenues" に含まれている[^google-data-center]。

つまり、Amazon の "technology and content" と Alphabet の研究開発費を比較する場合、前者にはデータセンターの費用が含まれていて後者には含まれていない、ということになるため、両者を比較する場合には注意が要る。

[^google-data-center]: Google Form 10-K の p.51 に、"costs of revenues" は "Expenses associated with our data centers and other operations (including bandwidth, compensation expense (including SBC), depreciation, energy, and other equipment costs)" を含む、と書かれている。


### トヨタ自動車

トヨタ自動車[^toyota-ir]は、Amazon・Alphabet とは業種が違う会社ではあるが、日本で最も研究開発費に投資している会社ということでよく挙げられる[^toyota-r&d]ので見てみよう。

連結決算（米国会計基準）におけるトヨタの研究開発費・減価償却費・設備投資額は、[2020年3月期 決算要旨](https://global.toyota/pages/global_toyota/ir/financial-results/2020_4q_summary_jp.pdf) のp.30にある（単位は億円、3月締め）：

/ | 2019 | 2020
:--- | ---: | ---:
研究開発費 | 10,488 | 11,103
減価償却費 | 9,848 |  8,128
設備投資額 | 14,658 | 13,930

この数字を見る限り、もし仮に Amazon の "technology and content" を「研究開発費」として扱うならば、トヨタの「研究開発費」この3つの項目を足したものとすべきなので、「トヨタも Amazon と同じくらい "研究開発" に投資している」と言えなくもない。

[^toyota-ir]: [アーカイブズ \| 決算報告 \| 投資家情報 \| トヨタ自動車株式会社 公式企業サイト](https://global.toyota/jp/ir/financial-results/archives/)
[^toyota-r&d]: 例えば、[研究開発費11年連続増　１位トヨタ、1兆1000億円 \| 日刊工業新聞 電子版](https://www.nikkan.co.jp/articles/view/00567679)


## まとめ

Amazon の決算報告書には研究開発費の金額が書かれていない。
そのため Amazon の研究開発費に言及したい人たちは、代わりに "technology and content” という項目の金額を研究開発費として扱っている。
しかし、この “technology and content” は、AWSの運用コストといった、通常の意味での「研究開発費」ではないものを含んでいるため、他の会社との比較に使うことができない。
逆に、Amazon の "technology and content" を「研究開発費」として採用するのであれば、他の会社と比較するときに他の会社の数字を修正する必要があるだろう。

この文章を途中まで書いたところで気づいたが、この記事と同じことを主張している人がすでにいた。
まあ当たり前なんだけど。

- [How Much Does Amazon Spend on R&D? Less Than You Think\. \| The Motley Fool](https://www.fool.com/investing/2018/06/13/how-much-does-amazon-spend-on-rd-less-than-you-thi.aspx)
- [Amazon's Technology and Content Spending a Huge Gift to Economy \- Bloomberg](https://www.bloomberg.com/opinion/articles/2018-04-05/amazon-s-technology-and-content-spending-a-huge-gift-to-economy)


[^segments]: Amazon は自身の事業を、North America, International, AWSの3つのセグメントに分けている (p.66)。
    2016~2018年のセグメント別の営業費用は下記のようになっている (p.66, 単位は $1M)。

    セグメント | 2016 | 2017 | 2018
    :-- | --: | --: | --:
    North America | 77,424 | 103,273 | 134,099
    International | 45,266 | 57,359 | 68,008
    AWS | 9,111 | 13,128 | 18,359
    |
    合計 | 131,801 | 173,760 | 220,466

## 付録. なんとかして研究開発費の金額を見積もれないか？

Amazonの決算報告書に研究開発費の項目がないのは仕方ないとして、決算報告書にある他の数字からなんとかして研究開発費を見積もることはできないだろうか？

"Technology and content costs" において、1~3のコストの大半は North America セグメントに、4 の大半は AWS セグメントに配分されている[^infrastructure-costs]。
そこで仮に、2番目の「大半」（原文では "the majority"）を「ほぼ全部」という意味だと仮定してみる。
さらに、1~3が研究開発費に該当すると仮定してみる。
すると、`"technology and content" ≒ "研究開発費" + "AWS の infrastructure costs"` と書けて、さらに
`"AWS の infrastructure costs" < "AWSセグメントの営業費用"` だから、
例えば2018年の数字を使うと、

```
"研究開発費" ≒ "technology and content" - "AWS の infrastructure costs"
             > "technology and content" - "AWSセグメントの営業費用"
             = $28,837M - $18,359M
             = $10,478M
```

となり、研究開発費は$10,478M〜$28,837Mと見積もれなくもない。
しかし、これだけ幅があると実質なにもわかっていないのと同じだろう。

[^infrastructure-costs]: p.66 の1段落目: "The majority of technology infrastructure costs are allocated to the AWS segment based on usage. The majority of the remaining non-infrastructure technology costs are incurred in the U.S. and are allocated to our North America segment."
