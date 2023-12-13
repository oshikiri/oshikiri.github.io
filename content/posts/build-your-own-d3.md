+++
title = "D3を理解するために機能の一部を再実装してみる"
date = "2023-09-30"
description = "D3を使おうと思ったが、使用例やドキュメントを読むだけではどうしても理解できなかった。そこで、理解するために機能の一部を自作してみることにする。"
+++

{% warning() %}
  完成まで時間がかかりそうだったので、この記事は途中で公開している。
{% end %}

## TL;DR
D3の難しさは、

1. D3が出力するSVGを意識する必要がある。一方で、通常のチャートライブラリでは出力した画像やHTMLの中身を意識することはない
2. 使うために必要になる事前知識が多い（JavaScript、HTML、SVG、…）
2. D3特有のAPI（特にd3-selectionあたり）になれる必要がある

から来ている（と思う）ので、挙動を理解しつつ使うためにはそこを事前に抑えておく必要がある。


## はじめに {#introduction}
### この記事の背景 {#background}

[D3](https://github.com/d3/d3)（D3.js）の使い方を調べたが、難しくていまいち理解できなかった[^d3-is-difficult]。
そこでこの記事では、D3の挙動を理解することを目的として、D3と同じインターフェースで同じグラフを描画できるようなライブラリを作ってみる[^build-your-own-x]。
イメージとしては、D3のサンプルコードのうち、 `<script src="https://d3js.org/d3.v7.min.js"></script>` の部分を自作ライブラリに差し替えても動くようにしたい。
もちろん、D3の機能すべてを再実装するのは現実的ではないので「主なサンプルコードをある程度動かすことができる」くらいの目標にしておく。


[^d3-is-difficult]: 念のため調べてみたが、よく言われているらしい: [The Trouble with D3](https://medium.com/dailyjs/the-trouble-with-d3-4a84f7de011f)、[Is it just me, or is D3.js too hard?](https://www.quora.com/Is-it-just-me-or-is-D3-js-too-hard)、[Could The Developer Experience For D3.js Be Improved](https://javascript.plainenglish.io/how-the-developer-experience-for-d3-js-could-be-improved-31b372ab3119)

[^build-your-own-x]: 有名なツールやパッケージの中身を理解するためにはそれを再実装すればいい、と一部では言われていて、これを実践する記事は [build-your-own-x](https://github.com/codecrafters-io/build-your-own-x) という一大ジャンル？になっている。

### この記事の扱っている範囲
この記事を書き始めてから気づいたが、D3の記事をゼロから書くといつまで経っても書き終わらないので、取り扱う項目を絞っておく必要がある。

**この記事で扱っていること**
- D3の基本的なAPIの挙動・実装方法
- D3っぽい挙動をするライブラリの作り方
- 私がD3を解読していたときのメモ

**この記事で扱っていないこと**
- D3の使い方[^d3-introduction]
- D3のデータハンドリング（d3-array など）
- d3-geoなどの発展的な可視化
- JavaScript（特にDOM操作周り）、CSS、SVGなどに関する知識

こう書いてみると、いったい想定読者は誰なんだろう？という疑問が浮かんでくる。
この文はこの記事をある程度は書き終えたときに書いているが、まだよくわかっていない。
まあいいや。


[^d3-introduction]: [D3 Tips and Tricks v7.x by Malcolm Maclean](https://leanpub.com/d3-t-and-t-v7) あたりを読んでからこの記事を読むのが良いと思う

### 実行環境

D3のバージョンについては、現在の最新版 ([v7.8.5](https://github.com/d3/d3/blob/v7.8.5)) をもとに記事を書く。
ただし、この記事では基本的なAPIしか扱わないので、数年経ったくらいではそう影響を受けないと思われる。
まあ数年前のアップデートで `attr` の使い方が大きく変わったという例はあるが、最近はD3もあまり更新されていないようだし多分大丈夫でしょう[^d3-selection-last-update]。

また、このページにあるコードの動作確認は基本的にChrome v116で行った。

[^d3-selection-last-update]: ちなみに後で出てくる d3-selection は[最終更新が2年前](https://github.com/d3/d3-selection/commit/91245ee124ec4dd491e498ecbdc9679d75332b49)だった

### この記事について

この記事自体は[ここ](https://github.com/oshikiri/oshikiri.github.io/blob/master/content/posts/build-your-own-d3.md)にあるマークダウンなどから生成されている。
また、最終的な実装と、この記事中にある `examples/~` はすべて [build-your-own-d3](https://github.com/oshikiri/build-your-own-d3) リポジトリに入っている。


## 簡単な図形を描画する {#simple-diagrams}
グラフの描画処理を実装する前に、簡単な図形（文字列、長方形、折れ線など）を描画できるようにしておく。
これらの図形は、棒グラフや折れ線グラフなどの基本的なグラフを描画する際のパーツとして使われる。

### 文字列を描画する {#hello-world-text}
まずはじめに、`hello world` という文字列を画面上に描画するだけの処理を実装してみる。

この処理をD3で実装しようとすると、以下のようなコードになる。
これ以降、D3を使ってグラフを描画するコードを「D3バージョン」と呼ぶことにする。

一応あとのためにコメントで一行ずつ説明しておく。

{% with_caption(title="examples/hello-world/d3.html (D3バージョン)") %}
```html
<!doctype html>
<meta charset="utf-8" />
<body>
  <div id="chart"></div>

  <script src="https://d3js.org/d3.v7.min.js"></script>

  <script>
    d3.select("#chart")     // body 内にある id=chart の要素を探す
      .append("div")        // それに div 要素を追加する
      .text("hello world"); // さらにその div 要素に "hello world" という文字列を追加する
  </script>
</body>
```
{% end %}

最後の `<script>` 内にあるJSが実行されると、`#chart` というIDがついている div 要素が更新され、次の図のように `chart` 内に `<div>hello world</div>` が追加される。

{{ image(
  title="hello-world (D3バージョン) の実行結果",
  path="/images/build-your-own-d3/hello-world-d3.png"
) }}

これをD3を使わずに再実装してみる。

まずコメントに書かれている内容を読んでみると、そのままJSで書けそうなことに気づく。
試しに書いてみる。

{% with_caption(title="examples/hello-world/vanilla-js.html (Web APIを使うバージョン)") %}
```html
<!doctype html>
<meta charset="utf-8" />
<body>
  <div id="chart"></div>

  <script>
    const chart = document.querySelector("#chart");
    const div = document.createElement("div");
    chart.appendChild(div);
    const text = document.createTextNode("hello world");
    div.appendChild(text);
  </script>
</body>
```
{% end %}

上記のコードをHTMLファイルとして保存してブラウザで開くと、D3バージョンと同じ挙動になっているのを確認できる。

{{ image(
  title="hello-world (Web APIを使うバージョン) の実行結果",
  path="/images/build-your-own-d3/hello-world-vanillajs.png"
) }}

さて、この記事の目的は「D3と同じインターフェースで同じような挙動をするライブラリを実装する」ということだった。
そこで次にインターフェースを合わせてみる。

D3バージョンのグラフ描画処理のコードを再度眺めてみると、
`d3.select("#chart").append("div").text("hello world");` という、D3に特徴的なメソッドチェインを基本としたインターフェースになっていることがわかる。
また `d3.select` の返り値は [@types/d3-selection によれば Selection らしい](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/61748a217350dddbef9842803adf8533a8b1e8b9/types/d3-selection/index.d.ts#L107-L121)。
以上の情報をもとに、ひとまず次のように Selection クラスを追加してみる。

{% with_caption(title="examples/hello-world/myd3.html (自作バージョン)") %}
```html
<!doctype html>
<meta charset="utf-8" />
<body>
  <div id="chart"></div>

  <script>
    const d3 = {
      select: function (selector) {
        const el = document.querySelector(selector);
        return new Selection(el);
      },
    };

    class Selection {
      element;
      constructor(element) {
        this.element = element;
      }
      append(name) {
        const child = document.createElement(name);
        this.element.append(child);
        return new Selection(child);
      }
      text(content) {
        const txt = document.createTextNode(content);
        this.element.append(txt);
        return this;
      }
    }
  </script>

  <script>
    d3.select("#chart").append("div").text("hello world");
  </script>
</body>
```
{% end %}

これをHTMLとして保存しブラウザで開くと、D3バージョンと同じ挙動になっているのを確認できる。

これ以降では、この実装を徐々に拡張していって、描ける図形やグラフを増やしていくことにする。


### 長方形を描画する {#svg-rect}

D3で長方形を書くコードは以下の通りである。

{% with_caption(title="examples/rect/d3.html") %}
```html
<!doctype html>
<meta charset="utf-8" />
<body>
  <div id="chart"></div>

  <script src="https://d3js.org/d3.v7.min.js"></script>

  <script>
    const svg = d3
      .select("#chart")
      .append("svg")
      .attr("width", 500)
      .attr("height", 500)
      .append("g");
    svg
      .append("rect")
      .attr("x", 200)
      .attr("y", 200)
      .attr("width", 50)
      .attr("height", 20)
      .attr("fill", "blue");
  </script>
</body>

```
{% end %}

これを実行すると、`#chart` に以下のようなSVG要素が追加される。

```html
<div id="chart">
  <svg width="500" height="500">
    <g>
      <rect x="200" y="200" width="50" height="20" fill="blue"></rect>
    </g>
  </svg>
</div>
```

一応解説しておくと、500x500のSVG要素を作成してその中に `g` 要素を作成、さらにその中に (200, 200) の位置に 50x20 で青で塗りつぶされた長方形を描画している。

ここでは新しく `attr` というメソッドを使っているため、これを実装する必要がある。
Selection クラスに以下のような `attr` を追加してみる。

```js
attr(key, value) {
  this.element.setAttribute(key, value);
  return this;
}
```

しかし、`attr` を追加しても、なぜか長方形は表示されない。

{{ image (
  title = "長方形は表示されない",
  path = "/images/build-your-own-d3/rect-did-not-appear.png"
) }}

これはSVGを扱うときのハマりポイントなのだが、いったん理由は置いておくとして、

```js
const child = document.createElement(name);
```

としていたところを

```js
const child = document.createElementNS("http://www.w3.org/2000/svg", name);
```

にすれば解決する。
実際、この変更を加えてみると、期待通り下図のように長方形が表示される。

{{ image (
  title = "長方形が表示された",
  path = "/images/build-your-own-d3/rect-appeared.png"
) }}

なぜ `createElementNS` を使う必要があるか？
`document.createElement("svg")` は `svg` という名前のHTML要素を作る[^createelement]のだが、SVGはHTML要素ではない（HTMLとは別の名前空間に属する）ので、`createElementNS` の第一引数で名前空間を明記する必要があるからだ。

- [svg - SVG: スケーラブルベクターグラフィック | MDN](https://developer.mozilla.org/ja/docs/Web/SVG/Element/svg)
- [名前空間の速修講座 - SVG: スケーラブルベクターグラフィック | MDN](https://developer.mozilla.org/ja/docs/Web/SVG/Namespaces_Crash_Course)
- [No `createElement` with SVG | webhint documentation](https://webhint.io/docs/user-guide/hints/hint-create-element-svg/)

ちなみに、以下のコードではすべての要素にSVGの名前空間を指定しているが、本当は一番外側の `<svg>` のみに名前空間を指定すればいいらしい。

[^createelement]: Document.createElementのドキュメントを注意深く読んでみると、たしかに「[HTML 文書において、 document.createElement() メソッドは tagName で指定された HTML 要素を生成し](https://developer.mozilla.org/ja/docs/Web/API/Document/createElement)」と書かれている。

一応ソースコード全体をまとめておく。

{% with_caption(title="examples/rect/myd3-createelementns.html") %}
```html
<!doctype html>
<meta charset="utf-8" />
<body>
  <div id="chart"></div>

  <script>
    const d3 = {
      select: function (selector) {
        const el = document.querySelector(selector);
        return new Selection(el);
      },
    };

    class Selection {
      element;
      constructor(element) {
        this.element = element;
      }
      append(name) {
        const child = document.createElementNS(
          "http://www.w3.org/2000/svg",
          name,
        );
        this.element.append(child);
        return new Selection(child);
      }
      text(content) {
        const txt = document.createTextNode(content);
        this.element.append(txt);
        return this;
      }
      attr(key, value) {
        this.element.setAttribute(key, value);
        return this;
      }
    }
  </script>

  <script>
    const svg = d3
      .select("#chart")
      .append("svg")
      .attr("width", 500)
      .attr("height", 500)
      .append("g");
    svg
      .append("rect")
      .attr("x", 200)
      .attr("y", 200)
      .attr("width", 50)
      .attr("height", 20)
      .attr("fill", "blue");
  </script>
</body>
```
{% end %}


### 折れ線を描画する {#svg-path}

{{ image (
  title = "SVGのパスで描画したうずまき",
  path = "/images/build-your-own-d3/svg-path.png"
) }}

{% with_caption(title="examples/svg-path/d3.html") %}
```html
<!doctype html>
<meta charset="utf-8" />
<style>
  .line {
    fill: none;
    stroke: steelblue;
    stroke-width: 1px;
  }
</style>
<body>
  <div id="chart"></div>

  <script src="https://d3js.org/d3.v7.min.js"></script>

  <script>
    const pathDefinition = "M10,10 V50 H50 V10 H20 V40 H40 V20 H30 V30";
    const svg = d3
      .select("#chart")
      .append("svg")
      .attr("width", 500)
      .attr("height", 500)
      .append("g");
    svg.append("path").attr("d", pathDefinition).attr("class", "line");
  </script>
</body>
```
{% end %}

実はパスを追加する処理については、これまでの実装のままで動く。
SVG のパス `<path>` については、参考になるリンクだけ貼っておく。

- [`<path>` - SVG: スケーラブルベクターグラフィック | MDN](https://developer.mozilla.org/ja/docs/Web/SVG/Element/path)
- [パス - SVG: スケーラブルベクターグラフィック | MDN](https://developer.mozilla.org/ja/docs/Web/SVG/Tutorial/Paths)


### 外部のJSONからデータを読み込む {#load-json}

{% with_caption(title="examples/load-json/data.json") %}
```json
[
  { "x": 0, "y": 0, "width": 50, "height": 50 },
  { "x": 100, "y": 100, "width": 50, "height": 100 },
  { "x": 0, "y": 200, "width": 100, "height": 50 }
]
```
{% end %}

`d3.json` を使って上記のようなJSONファイルをロードし、それをもとに下図のように複数の長方形を描画してみる。

{{ image (
  title = "JSONの中身をもとに長方形を描画する",
  path = "/images/build-your-own-d3/load-json.png"
) }}

D3バージョンは以下の通り。

{% with_caption(title="examples/load-json/d3.html") %}
```html
<!doctype html>
<meta charset="utf-8" />
<body>
  <div id="chart"></div>

  <script src="https://d3js.org/d3.v7.min.js"></script>
  <script>
    const svg = d3
      .select("#chart")
      .append("svg")
      .attr("width", 500)
      .attr("height", 500)
      .append("g");

    d3.json("./data.json").then((data) => {
      for (const d of data) {
        svg
          .append("rect")
          .attr("class", "bar")
          .attr("x", d.x)
          .attr("y", d.y)
          .attr("width", d.width)
          .attr("height", d.height);
      }
    });
  </script>
</body>
```
{% end %}

D3の例を見たことがあると、`data` や `enter` を使っていない上記のコードは違和感があるかもしれない[^d3-without-selection]。
こういう書き方になっているのは、`data` や `enter` はややこしくてすぐに実装できないためである。

[^d3-without-selection]: ただし、d3-selection のAPIに慣れるまでは、（邪道ではあるものの）こういう書き方を使い続けるのもひょっとするとアリなんじゃないかな〜とは思っている。
でも慣れたら d3-selection のAPIを使ったほうが便利ではある。

`d3.json` は単に Promise を返しているだけなので、以下のように実装しておけば良い。

{% with_caption(title="examples/load-json/myd3.html の d3 定義の部分") %}
```js
const d3 = {
  select: function (selector) {
    const el = document.querySelector(selector);
    return new Selection(el);
  },
  async json(path) {
    return fetch(path)
      .then((response) => response.text())
      .then((data) => JSON.parse(data));
  },
};
```
{% end %}

ちなみに実際の[d3-fetchのソースコード](https://github.com/d3/d3-fetch/blob/v3.0.1/src/json.js)もほぼ同じような実装になっている。



{% warning() %}
  ここから先はまだ書けていない。
  実装は一応できているのだが、リファクタリングが必要なのと、説明の形にまとめるのが難しかったので書く作業が止まっている。
{% end %}


## パッケージとして整理する {#setup-package}
コード量が増えてきたので、今のうちにパッケージの形に整えておく。

[実際のD3の構成](https://github.com/d3/d3/blob/v7.8.5/src/index.js)を見てみると、おおまかな機能ごとにリポジトリが別れていて、 d3/d3リポジトリですべてを読み込む形になっている。
今回の実装ではリポジトリを分けるほどでもないので、ディレクトリに分けた上で 一番上の index.ts で `export * from "./selection";` のように export する形にしておく。


## Selection を実装する {#selection-class}

これまでの例でも使われていた[`d3.select`](https://github.com/d3/d3-selection/blob/v3.0.0/src/select.js) （document.body から対象の要素を拾ってくる）は、[`Selection`](https://github.com/d3/d3-selection/blob/v3.0.0/src/selection/index.js) を生成して返すだけの関数になっている。
`@types/d3` を見てみると、`d3.select` や `d3.selectionAll` 以降につながっているメソッドはすべて Selection から生えていることがわかる。
つまり、これまで使われてきたようなD3特有の操作（append, attr, text) は Selection のメソッドとして実装されているので、D3を理解するためには Selection を理解する必要がある。


### d3-selection とは {#d3-selection}
Selection の実装は [d3-selection](https://github.com/d3/d3-selection) リポジトリにある。


## 棒グラフを描画する {#bar-chart}

{{ image (
  title = "自作バージョンのD3で描画した棒グラフ",
  path = "/images/build-your-own-d3/bar-chart.png"
) }}

### 棒グラフの棒を描画する {#bar-of-bar-chart}
### スケーリングの処理を実装する {#d3-scale}
### 軸を描画する {#bar-chart-axis}
### 目盛りを追加する (1) {#ticks-1}

## 線グラフを描画する {#line-chart}

{{ image (
  title = "自作バージョンのD3で描画した折れ線グラフ",
  path = "/images/build-your-own-d3/line-chart.png"
) }}

### 目盛りを追加する (2) {#ticks-2}



## 最後に

### D3とはなにか

そもそも「D3が難しい」といったとき、「D3はグラフ描画ライブラリとして難しい」というふうに解釈されるが、
D3のことを「グラフ描画ライブラリ」「チャートライブラリ」というとミスリーディングなのかなと思う。
いやもちろん、D3はグラフを描画するために使うライブラリではあるのだが、ブラウザがSVGの描画を行っているためユーザーはSVGを意識する必要がある（一方で、通常のチャートライブラリの場合は最終的なグラフしか意識しない）という点でミスリーディングになる。

実際、[公式ドキュメント](https://d3js.org/what-is-d3#d3-is-a-low-level-toolbox)でも、*"D3 is a low-level toolbox", "D3 is not a charting library in the traditional sense."* と書かれている[^d3-is-not-chart-library]。

[^d3-is-not-chart-library]:
え？でも普通にD3はグラフ描画ライブラリ/チャートライブラリって紹介されてない？と疑問に思ったので調べてみた。
D3の紹介文を見てみると、["D3 (or D3.js) is a free, open-source JavaScript library for visualizing data."](https://github.com/d3/d3)だったり["The JavaScript library for bespoke data visualization"](https://d3js.org/)（bespokeに強調）だったりと明言を避けた書き方をしていて、chart library などといい切っていないことに気づく。
なるほどねぇ。


### あらためて、D3はなぜ難しいのか？

D3をチャートライブラリではなくSVGを生成するライブラリと認識すれば、D3の難しさはある程度説明がつくような気もするが、その他にも難しさの原因はあるように感じる。
この記事を書いているときに思いついた理由を列挙してみる。

- 命名や定義が難しい
  - ギリギリまで文字数を削るような命名をしている (ex. `attr`)[^d3-naming]
  - 1つの関数でいろいろな機能（例えば getter/setter）をまとめて表現していたりする
    - 例えば、[selection.datum](https://github.com/d3/d3-selection#selection_datum) は引数の型（undefined, value, null, function）によってそれぞれ挙動が変わるのだが、それを文章だけで説明されるのはしんどい。
- D3はJavaScriptで書かれているので、定義ジャンプが使えない。`@types/d3` を導入すれば一応型と関数の説明は読めるが、実装に飛ぶことはできない[^rewrite-in-typescript]。
- d3-selection が難しい
  - ドキュメントと実装とテストを読んで、ようやく理解できた関数がいくつか
- ~~コピペしてそのまま動くコードがほしいのに、Observable のコード（そのままでは流用しづらい）しか出てこない~~

[^d3-naming]: メソッド名を短くしたいというのも理解できなくはない。
しかし、細かい部分を調整したり、複雑な可視化を行えたりする、というD3の特性上、命名に関しては短くすることより、どちらかといえば冗長気味にしたほうがいいケースが多いと思う。
例えば、PythonやRの日々のデータハンドリングの途中で行う可視化（読み返されることを考慮する必要がほぼない）と、ウェブサイトで大きく表示される一点ものの複雑な可視化（書かれたあとにメンテナンスのために読み返されることが多い）だと、前者であれば簡潔なほうがよさそうだが、後者であれば冗長気味になっても可読性が高いほうがいいだろう。

[^rewrite-in-typescript]: [TypeScriptで型を付けようという提案は上がっていたが、2018年に一旦却下されている](https://github.com/d3/d3/issues/3284)。
却下されるのは意外だなと最初は思ったが、d3配下の一つ一つのリポジトリはわりと小さいということもあって、最悪型なしでもなんとか理解できる、というのはあるかもしれない。
あとはD3はTSより前からあるし（？）。
コード量少ないから書き換えるPRを投げてみようかなとも思ったが、数年前に却下されている時点でマージされなさそう。
ただし、却下した本人が直近で作っているライブラリは[一部TypeScriptが導入されている](https://github.com/observablehq/plot/issues/401)ようなので、今再度提案したら通る可能性はあるのかもしれない。
