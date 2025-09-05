+++
title = "D3を理解するために機能の一部を再実装してみる"
date = "2024-01-14"
description = "D3でグラフを描こうと思ったが、ドキュメントや使用例を読むだけではどうしても理解できなかった。そこで、理解するために機能の一部を自作してみることにする。"
+++

{% warning() %}
  GPT-5に校正してもらってから英訳したバージョン:
  <https://github.com/oshikiri/build-your-own-d3>
{% end %}

## この記事の要約
D3の難しさは、

1. 使う上でD3が出力するSVGを意識する必要がある。一方で、通常のチャートライブラリでは出力した画像やHTMLの中身を意識することはない
2. JavaScript、HTML、SVG など使うために必要になる事前知識が多い
3. D3特有のAPI、特に d3-selection に慣れる必要がある

から来ている（と思う）ので、内部の挙動を理解しつつ使うためにはこれらの項目を事前に押さえておく必要がある。

これらについて理解するために、D3のAPIに沿って簡単なグラフを描画できるスクリプト [mini-d3.js](https://github.com/oshikiri/build-your-own-d3/blob/main/mini-d3.js) （[デモページ](https://oshikiri.github.io/build-your-own-d3/demo/bar_chart.html)）を作成した。


## はじめに {#introduction}
### この記事の背景 {#background}

少し複雑な可視化をしようと思って
[D3](https://github.com/d3/d3)（D3.js）の使い方を調べたが、難しくていまいち理解できなかった[^d3-is-difficult]。
そこでこの記事では、D3の挙動を理解することを目的として、D3と同じインターフェースで同じグラフを描画できるようなスクリプトを作ってみる[^build-your-own-x]。
イメージとしては、D3のサンプルコードのうち、 `<script src="https://d3js.org/d3.v7.min.js"></script>` の部分を自作スクリプト `<script src="./mini-d3.js"></script>` に差し替えても動くようにする。
もちろん、D3の機能すべてを再実装するのは現実的ではないので「主なサンプルコードをある程度動かすことができる」くらいの目標にしておく。


[^d3-is-difficult]: 念のため調べてみたが、よく言われているらしい: [The Trouble with D3](https://medium.com/dailyjs/the-trouble-with-d3-4a84f7de011f)、[Is it just me, or is D3.js too hard?](https://www.quora.com/Is-it-just-me-or-is-D3-js-too-hard)、[Could The Developer Experience For D3.js Be Improved](https://javascript.plainenglish.io/how-the-developer-experience-for-d3-js-could-be-improved-31b372ab3119)

[^build-your-own-x]: 有名なツールやパッケージの中身を理解するためにはそれを再実装すればいい、と一部では言われていて、これを実践する記事は [build-your-own-x](https://github.com/codecrafters-io/build-your-own-x) という一大ジャンル？になっている。


### この記事の扱っている範囲
この記事を書き始めてから気づいたが、D3の記事をゼロから書くといつまで経っても書き終わらないので、取り扱う項目を絞っておく必要がある。
この記事で扱うこと/扱わないことを以下のように決めておく。

**この記事で扱うこと:**
- D3の基本的なAPIの挙動・実装方法
- D3っぽい挙動をするライブラリの作り方
- 私がD3を解読していたときのメモ

**この記事で扱わないこと:**
- D3の使い方[^d3-introduction]
- D3のデータハンドリング（d3-array など）
- d3-geoなどの発展的な可視化
- JavaScript（特にDOM操作周り）、CSS、SVGなどに関する知識

こう書いてみると、想定読者はいったい誰なんだろう？という疑問が浮かんでくる。
この文はこの記事をほぼ書き終えたときに書いているが、まだよくわかっていない。
まあいいや。

[^d3-introduction]: ここが怪しい場合は、[D3 Tips and Tricks v7.x by Malcolm Maclean](https://leanpub.com/d3-t-and-t-v7) あたりを読んでからこの記事を読むのが良いと思う


### 実行環境

D3のバージョンについては、現在の最新版 ([v7.8.5](https://github.com/d3/d3/blob/v7.8.5)) をもとに記事を書く。
ただし、この記事では基本的な機能しか扱わないので、数年経ったくらいではそう影響を受けないと思われる。
まあ[2018年のアップデート](https://github.com/d3/d3/blob/main/CHANGES.md#changes-in-d3-50)で基本的なAPIの挙動が大きく変わったという例はあるが、ここ数年はD3もあまり更新されていないようだし[^d3-selection-last-update]多分大丈夫でしょう。

また、このページにあるコードの動作確認はChrome v120で行った。

[^d3-selection-last-update]: ちなみに後で出てくる d3-selection は2023-09-30時点では[最終更新が2年前](https://github.com/d3/d3-selection/commit/91245ee124ec4dd491e498ecbdc9679d75332b49)だった。

### この記事について

この記事自体は[GitHubにあるmarkdown](https://github.com/oshikiri/oshikiri.github.io/blob/master/content/posts/build-your-own-d3.md)から生成されている。
また、最終的な実装は [build-your-own-d3](https://github.com/oshikiri/build-your-own-d3) リポジトリに入っている。


## 簡単な図形を描画する {#simple-diagrams}
グラフの描画処理を実装する前に、簡単な図形（文字列、長方形、折れ線など）を描画できるようにしておく。
これらの図形は、棒グラフや折れ線グラフなどの基本的なグラフを描画する際のパーツとして使われる。

### 文字列を描画する {#hello-world-text}
まずはじめに、`hello world` という文字列を画面上に描画するだけの処理を実装してみる。

この処理をD3で実装しようとすると以下のようなコードになる。
これ以降、オリジナルのD3の `https://d3js.org/d3.v7.min.js` を使ってグラフを描画するコードを「本家D3」と呼ぶことにする。

あとのためにコメントで一行ずつ説明しておく。

{% with_caption(title="demo/insert-text/d3.html (本家D3)") %}
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
  title="hello-world (本家D3) の実行結果",
  path="/images/build-your-own-d3/hello-world-d3.png"
) }}

このサンプルで使われている `d3` を再実装してみる。

まずコメントに書かれている内容を読んでみると、そのままJavaScriptで実装できそうだということに気づく。
試しに書いてみる。

{% with_caption(title="demo/insert-text/vanilla-js.html (Web APIを使うバージョン)") %}
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

上記のコードをHTMLファイルとして保存してブラウザで開くと、本家D3バージョンと同じ挙動になっていることを確認できる。

{{ image(
  title="hello-world (Web APIを使うバージョン) の実行結果",
  path="/images/build-your-own-d3/hello-world-vanillajs.png"
) }}

さて、この記事の目的は「D3と同じインターフェースで同じような挙動をするライブラリを実装する」ということだった。
そこで次にインターフェースを合わせてみる。

本家D3バージョンのグラフ描画処理のコードを再度眺めてみると、
`d3.select("#chart").append("div").text("hello world");` という、D3に特徴的なメソッドチェインを基本としたインターフェースになっていることがわかる。
また `d3.select` の返り値は [@types/d3-selection によれば Selection らしい](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/61748a217350dddbef9842803adf8533a8b1e8b9/types/d3-selection/index.d.ts#L107-L121)。
以上の情報をもとに、ひとまず次のように Selection クラスを追加してみる[^original-d3-select]。

{% with_caption(title="demo/insert-text/myd3.html (自作バージョン)") %}
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

- [code](https://github.com/oshikiri/build-your-own-d3/blob/main/demo/insert-text/myd3.html)
- [demo](https://www.oshikiri.org/build-your-own-d3/demo/insert-text/myd3.html)

これをHTMLとして保存しブラウザで開くと、本家D3バージョンと同じ挙動になっているのを確認できる。

これ以降では、この実装を徐々に拡張して描ける図形やグラフを増やしていく。

[^original-d3-select]: ちなみに `d3.select` の[実際の実装](https://github.com/d3/d3-selection/blob/v3.0.0/src/select.js) も自作バージョンと同様に [`Selection`](https://github.com/d3/d3-selection/blob/v3.0.0/src/selection/index.js) を生成して返すだけの関数になっている。


### 長方形を描画する {#svg-rect}

本家D3で長方形を書くコードは以下の通りである。

{% with_caption(title="demo/rectangle/d3.html") %}
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

一応解説しておくと、500x500のSVG要素を作成してその中に `g` 要素を作成、さらにその中に (200, 200) の位置に 50x20 の青で塗りつぶされた長方形を描画している。

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
const child = document.createElementNS(
  "http://www.w3.org/2000/svg",
  name,
);
```

にすれば解決する。
実際、この変更を加えてみると、期待通り下図のように長方形が表示される。

{{ image (
  title = "長方形が表示された",
  path = "/images/build-your-own-d3/rect-appeared.png"
) }}

なぜ `createElement` ではなく `createElementNS` を使う必要があるか？
ここで作りたいのはSVG名前空間に属する `<svg>` なのだが、`document.createElement("svg")` だと HTML名前空間の `svg` 要素を作ってしまう[^createelement]からだ。
そのため、SVGの名前空間に属する `<svg>` を作りたい場合は、`createElementNS` の第一引数で明示的に名前空間を指定する必要がある。

- [svg - SVG: スケーラブルベクターグラフィック | MDN](https://developer.mozilla.org/ja/docs/Web/SVG/Element/svg)
- [名前空間の速修講座 - SVG: スケーラブルベクターグラフィック | MDN](https://developer.mozilla.org/ja/docs/Web/SVG/Namespaces_Crash_Course)
- [No `createElement` with SVG | webhint documentation](https://webhint.io/docs/user-guide/hints/hint-create-element-svg/)

[実際の append](https://d3js.org/d3-selection/modifying#selection_append) の実装では `append("svg")` の場合に名前空間として `http://www.w3.org/2000/svg` を使い、そうでない場合も親要素の名前空間を引き継ぐ実装になっている。
ちなみに、D3では[SVG以外の名前空間もサポートしている](https://d3js.org/d3-selection/namespaces)。

[^createelement]: Document.createElementのドキュメントを注意深く読んでみると、たしかに「[HTML 文書において、 document.createElement() メソッドは tagName で指定された HTML 要素を生成し](https://developer.mozilla.org/ja/docs/Web/API/Document/createElement)」と明記されている。

一応ソースコード全体をまとめておく。

{% with_caption(title="demo/rectangle/myd3.html") %}
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

- [code](https://github.com/oshikiri/build-your-own-d3/blob/main/demo/rectangle/myd3.html)
- [demo](https://www.oshikiri.org/build-your-own-d3/demo/rectangle/myd3.html)


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



## パッケージとして整理する？ {#setup-package}
コード量が増えてきたので今のうちにパッケージの形に整えておきたい。
しかし、ステップバイステップで説明を進める都合上、パッケージの形にまとめると、build-your-own-d3 リポジトリ内に大量の package.json を作る必要があり面倒になる。
そのためこの記事では、多少無理をしつつ、以下のように単一のJSファイルにすべて実装を詰め込む形で話を進める。

{% with_caption(title="examples/setup-package/myd3.html") %}
```js
<!doctype html>
<meta charset="utf-8" />
<body>
  <div id="chart"></div>

  <script src="./myd3.js"></script>
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

{% with_caption(title="examples/setup-package/myd3.js") %}
```js
const d3 = {
  select: function (selector) {
    const el = document.querySelector(selector);
    return new Selection(el);
  },
  async json(path) {
    return fetch(path)
      .then((response) => response.json())
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
```
{% end %}

もしパッケージとして整備したい場合は、実際のD3の構成が参考になる。
[実際のD3の構成](https://github.com/d3/d3/blob/v7.8.5/src/index.js)を見てみると、おおまかな機能ごとにリポジトリが別れていて、 d3/d3リポジトリですべてを読み込む形になっている[^d3-adopt-a-monorepo]。

私も最初に TypeScript で自作D3を実装したときは、本家D3と同様にディレクトリに分けた上で、一番上の index.ts で `export * from "./selection";` のように export する形にした。

[^d3-adopt-a-monorepo]: ただしこれについては今後変わる可能性がある:
  [Adopt a monorepo · Issue #3791 · d3/d3](https://github.com/d3/d3/issues/3791)


## 棒グラフを描画する {#bar-chart}

このセクションでは、最終的に以下のような棒グラフ ([デモページ](https://oshikiri.github.io/build-your-own-d3/demo/bar_chart.html)) が描けるようになることを目標にして実装を進めていく。

{{ image (
  title = "自作バージョンのD3で描画した棒グラフ",
  path = "/images/build-your-own-d3/bar-chart.png"
) }}

グラフは「D3 Tips and Tricks v7.x」で使われていたもので、元データは[こちらのJSON](https://github.com/oshikiri/build-your-own-d3/blob/main/demo/data/sales.json)にアップロード済み。

これ以降はかなり込み入った実装になるため、本家D3の実装で必要なものだけを抜き出す形で実装を進める。
特にこのセクションでは、 [d3-selection](https://github.com/d3/d3-selection) の実装を参考に実装を進めていく。


### 棒グラフの棒を描画する {#bar-chart-bars}

スケーリングの処理や目盛りなどは一旦無視して、下図のように棒グラフの棒の部分だけをまず描いてみる。

{{ image (
  title = "自作D3で描画した棒グラフ（スケーリングなし、heightがsalesの値になっている）",
  path = "/images/build-your-own-d3/bars-without-scaling.png"
) }}

まずは本家D3を使ってこれを描画するコードを作る。

{% with_caption(title="examples/bar-chart/bars/myd3.html") %}
```html
<!doctype html>
<meta charset="utf-8" />
<body>
  <div id="chart"></div>

  <!-- <script src="./myd3.js"></script> -->
  <script src="https://d3js.org/d3.v7.min.js"></script>

  <script>
    // スケーリングを実装していないので、とりあえず sales の最大値を埋めておく
    const height = 59;
    const width = 960;
    const barwidth = 50;

    const svg = d3
      .select("#chart")
      .append("svg")
      .attr("width", width)
      .attr("height", height)
      .append("g");

    d3.json("./../sales.json").then((data) => {
      svg
        .selectAll(".bar")
        .data(data)
        .enter()
        .append("rect")
        .attr("class", "bar")
        .attr("x", function (d) {
          return xScale(d.salesperson);
        })
        .attr("width", barwidth)
        .attr("y", function (d) {
          return yScale(d.sales);
        })
        .attr("height", function (d) {
          return height - yScale(d.sales);
        });

      svg
        .append("g")
        .attr("transform", `translate(0,${height})`)
        .call(d3.axisBottom(xScale));

      svg.append("g").call(d3.axisLeft(yScale));
    });

    // 仮実装
    let called = -1;
    function xScale(name) {
      called += 1;
      return barwidth * called;
    }
    function yScale(y) {
      return 59 - y;
    }
  </script>
</body>
```
{% end %}

自作D3でもこのサンプルコードが動くようにするためには、`d3.selectAll` `Selection.data` `Selection.enter` を実装し、さらに `Selection.attr` がバインドしたデータを使えるようにする必要がある。
本家 d3-selection の実装をもとに以下のように実装を追加してみる。

{% with_caption(title="examples/bar-chart/bars/myd3.js") %}
```js
const d3 = {
  select: function (selector) {
    const element = document.querySelector(selector);
    return new Selection([[element]], [document.documentElement]);
  },
  json: async function (path) {
    return fetch(path).then((response) => response.json());
  },
  scaleBand,
  scaleLinear,
  axisLeft,
  axisBottom,
};

class Selection {
  #groups;
  #parents;
  #enter;

  constructor(groups, parents) {
    this.#groups = groups;
    this.#parents = parents;
  }

  append(name) {
    return this.select(function () {
      const child = document.createElementNS(
        "http://www.w3.org/2000/svg",
        name,
      );
      child.__data__ = this.__data__;
      return this.appendChild(child);
    });
  }

  select(selectorOrFunction) {
    const selectFunction = this.#makeSelectFunction(selectorOrFunction);
    const subgroups = this.#groups.map((group) =>
      group.map((node, i) => {
        if (node === null) {
          return undefined;
        }
        const subnode = selectFunction.call(node, node.__data__, i, group);
        if ("__data__" in node) {
          subnode.__data__ = node.__data__;
        }
        return subnode;
      }),
    );

    return new Selection(subgroups, this.#parents);
  }

  #makeSelectFunction(selectorOrFunction) {
    if (typeof selectorOrFunction == "string") {
      return function () {
        return this.querySelector(selectorOrFunction);
      };
    } else {
      return selectorOrFunction;
    }
  }

  attr(key, valueOrFunction) {
    return this.#each(function (__data__) {
      const value = Selection.getValue(valueOrFunction, __data__);
      this.setAttribute(key, value);
      return this;
    });
  }

  static getValue(valueOrFunction, __data__) {
    if (typeof valueOrFunction == "function") {
      return valueOrFunction(__data__);
    } else if (["number", "string"].includes(typeof valueOrFunction)) {
      return String(valueOrFunction);
    } else {
      return valueOrFunction.apply(__data__);
    }
  }

  #each(callback) {
    const groups = this.#groups.map(function (group) {
      return group.map(function (node, i) {
        return callback.call(node, node.__data__, i);
      });
    });
    return new Selection(groups, this.#parents);
  }

  selectAll(selector) {
    const subgroups = [];
    const parents = [];
    this.#groups.forEach((group) => {
      group.forEach((node) => {
        subgroups.push(Array.from(node.querySelectorAll(selector)));
        parents.push(node);
      });
    });
    return new Selection(subgroups, parents);
  }

  data(__data__) {
    const groupsLength = this.#groups.length;
    const dataLength = __data__.length;
    const enter = new Array(groupsLength);
    for (let i = 0; i < groupsLength; i++) {
      enter[i] = new Array(dataLength);
      Selection.bindIndex(
        this.#parents[i],
        this.#groups[i],
        enter[i],
        __data__,
      );
    }
    this.#enter = enter;
    return this;
  }

  static bindIndex(parent, group, enter, data) {
    for (let i = 0; i < data.length; i++) {
      if (i < group.length) {
        group[i].__data__ = data[i];
      } else {
        enter[i] = new EnterNode(parent, data[i]);
      }
    }
  }

  enter() {
    return new Selection(this.#enter || this.#groups, this.#parents);
  }

  call(callback) {
    arguments[0] = this;
    callback.apply(null, arguments);
    return this;
  }
}

class EnterNode {
  constructor(parent, __data__) {
    this.parent = parent;
    this.__data__ = __data__;
  }
  appendChild(child) {
    return this.parent.appendChild(child);
  }
}

function scaleBand() {}

function scaleLinear() {}

function axisLeft() {
  return function () {};
}

function axisBottom() {
  return function () {};
}
```
{% end %}


### スケーリングの処理を実装する {#bar-chart-scale}

ここでは、D3のコードで先ほど省略していた `d3.scaleBand` などのスケーリングの処理を実装する。
D3を使ってグラフを描画するコードは以下のとおり。

```html
<!doctype html>
<meta charset="utf-8" />
<style>
  .bar {
    fill: steelblue;
  }
</style>
<body>
  <!-- <script src="https://d3js.org/d3.v7.min.js"></script> -->
  <script src="./myd3.js"></script>

  <script>
    const svgWidth = 960;
    const svgHeight = 500;
    const margin = { top: 20, right: 20, bottom: 30, left: 40 };
    const width = svgWidth - margin.left - margin.right;
    const height = svgHeight - margin.top - margin.bottom;

    const xScale = d3.scaleBand().range([0, width]).padding(0.1);
    const yScale = d3.scaleLinear().range([height, 0]);

    const svg = d3
      .select("body")
      .append("svg")
      .attr("width", svgWidth)
      .attr("height", svgHeight)
      .append("g")
      .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

    d3.json("../sales.json").then(function (data) {
      xScale.domain(
        data.map(function (d) {
          return d.salesperson;
        }),
      );
      yScale.domain([
        0,
        Math.max(
          ...data.map(function (d) {
            return d.sales;
          }),
        ),
      ]);

      svg
        .selectAll(".bar")
        .data(data)
        .enter()
        .append("rect")
        .attr("class", "bar")
        .attr("x", function (d) {
          return xScale(d.salesperson);
        })
        .attr("width", xScale.bandwidth())
        .attr("y", function (d) {
          return yScale(d.sales);
        })
        .attr("height", function (d) {
          return height - yScale(d.sales);
        });

      svg
        .append("g")
        .attr("transform", "translate(0," + height + ")")
        .call(d3.axisBottom(xScale));

      svg.append("g").call(d3.axisLeft(yScale));
    });
  </script>
</body>
```

この描画処理を実行できるようにするために、自作D3のほうに追加で `scaleBand` と `scaleLinear` を実装する必要がある。
長くなるのでここでは `scaleLinear` のみ載せておく。

{% with_caption(title="mini-d3.js から抜粋した scaleLinear の実装") %}
```js
function scaleLinear() {
  const scale = function (x) {
    const [xl, xh] = scale._domain;
    const [yl, yh] = scale._range;
    return yl + ((yh - yl) * (x - xl)) / (xh - xl);
  };
  scale._range = [];
  scale._domain = [];
  scale.range = function (_range) {
    this._range = _range;
    return this;
  };
  scale.domain = function (_domain) {
    this._domain = _domain;
    return this;
  };
  scale.getTickPoints = function () {
    return range(this._domain[0], this._domain[1], 5);
  };
  return scale;
}

function range(l, h, stepsize) {
  const values = [];
  for (let current = l; current <= h; current += stepsize) {
    values.push(current);
  }
  return values;
}
```
{% end %}

この実装を自作D3に追加したあと、HTMLをブラウザで開くと以下のようなグラフが表示される。

{{ image (
  title = "自作D3で描画した棒グラフ（スケーリングあり）",
  path = "/images/build-your-own-d3/bars-with-scaling.png"
) }}


### 軸と目盛りを描画する {#bar-chart-axis}

棒グラフの棒の部分は描画できたので、残りの軸と目盛りを描画する関数 `axisLeft` `axisBottom` を実装する。

ざっくり言ってしまうと軸の線を引いて適切な位置にテキストを配置するだけだが、雑に実装してもわりと長くなる。
そのため、ここでは `axisLeft` のみを載せておく。

{% with_caption(title="mini-d3.js から抜粋した axisLeft の実装") %}
```js
const tickLength = 6;
const tickLineWidth = 0.5;

function axisLeft(scale) {
  return function (axisRoot) {
    const mainLineLength = Math.abs(scale._range[1] - scale._range[0]);
    const valueLine =
      `M-${tickLength},${mainLineLength + tickLineWidth} ` +
      `H${tickLineWidth} V${tickLineWidth} H-${tickLength}`;

    axisRoot
      .attr("fill", "none")
      .attr("font-size", "10")
      .attr("font-family", "sans-serif")
      .attr("text-anchor", "end");
    axisRoot
      .append("path")
      .attr("class", "domain")
      .attr("stroke", "currentColor")
      .attr("d", valueLine);

    const ticks = axisRoot
      .selectAll(".tick")
      .data(scale.getTickPoints())
      .enter()
      .append("g")
      .attr("class", "tick")
      .attr("opacity", "1")
      .attr("transform", (d) => `translate(0, ${scale(d)})`);

    ticks.append("line").attr("stroke", "currentColor").attr("x2", -tickLength);
    ticks
      .append("text")
      .attr("fill", "currentColor")
      .attr("x", -9)
      .attr("dy", "0.32em")
      .text((d) => d);

    return axisRoot;
  };
}
```
{% end %}

最終的なコードとデモページはこちら:

- [mini-d3.js](https://github.com/oshikiri/build-your-own-d3/blob/main/mini-d3.js)
- [demo/bar_chart.html](https://oshikiri.github.io/build-your-own-d3/demo/bar_chart.html)

マジックナンバーが何箇所か出現していることからもわかるとおり、かなり限定的な実装にはなっているものの、たった約300行で棒グラフが描画できた。


## 折れ線グラフを描画する {#line-chart}

[D3 Tips and Tricks v7.x](https://leanpub.com/d3-t-and-t-v7) で扱われている他のグラフも描画できるようにしたい。
詳細は省略するが、例えば `d3.timeParse` や `d3.scaleTime` などを追加で実装すると、以下のような折れ線グラフが書けるようになる。

{{ image (
  title = "自作バージョンのD3で描画した折れ線グラフ",
  path = "/images/build-your-own-d3/line-chart.png"
) }}

「軸と目盛りを描画する」の実装を見ると想像がつくと思うが、これ以降は込み入ってくるので必要なコード量がかなり増えてくる。
ここまで理解できればもう本家D3のソースコードを読んだほうが理解が早いと思うので、これ以降は読者の課題ということにしておきたい。

現状の実装で足りていない部分を一応列挙しておく。

- 目盛りの実装。現状の実装ではかなり省略されているので、例えば先ほどのグラフではy軸のtickの間隔が63刻みと中途半端になっている。
- `d3.csv` や `d3.max` などデータハンドリングのよく使われる機能
- その他実装を省略してマジックナンバーでごまかしている部分


## 最後に

### D3とはなにか

そもそも「D3が難しい」といったとき、「D3は可視化ライブラリとして難しい」というふうに解釈されるが、
D3のことを「可視化ライブラリ」「チャートライブラリ」というとミスリーディングなのかなと思う。
いやもちろん、D3はグラフを描画するために使うライブラリではあるのだが、D3を扱う上ではユーザーはSVGを意識する必要があって、
その一方で通常の可視化ライブラリの場合はグラフの中身の構造を意識しなくてよい、という点でD3は他の有名な可視化ライブラリとは大きく異なる。

記事を書いているときに改めて調べて気づいたが、[公式ドキュメント](https://d3js.org/what-is-d3#d3-is-a-low-level-toolbox)でも *"D3 is a low-level toolbox", "D3 is not a charting library in the traditional sense."* と書かれている[^d3-is-not-chart-library]。

[^d3-is-not-chart-library]:
え？でも普通にD3はグラフ描画ライブラリ/チャートライブラリって紹介されてない？と疑問に思ったので調べてみた。
D3の紹介文を見てみると、["D3 (or D3.js) is a free, open-source JavaScript library for visualizing data."](https://github.com/d3/d3)だったり["The JavaScript library for bespoke data visualization"](https://d3js.org/)（bespokeに強調）だったりと明言を避けた書き方をしていて、chart library などといい切っていないことに気づく。
なるほどねぇ。


### あらためて、D3はなぜ難しいのか？

D3をチャートライブラリではなくSVGを生成するライブラリと認識すれば、D3の難しさはある程度説明がつくような気もするが、その他にも難しさの原因はあるように感じる。
この記事を書いているときに思いついた理由を列挙してみる。

- 命名や定義が難しい
  - ギリギリまで文字数を削るような命名をしている (例えば `attr` や `data`)[^d3-naming]
  - 1つの関数でいろいろな機能（例えば getter/setter）をまとめて表現していたりする。
    例えば、[selection.datum](https://github.com/d3/d3-selection#selection_datum) は引数の型（undefined, value, null, function）によってそれぞれ挙動が変わるのだが、それを文章だけで説明されるのはしんどい。
- エディタ上でドキュメントの閲覧や補完をするのが難しい。ただし、`@types/d3` を導入すれば一応型と関数の説明はエディタから簡単に開くことができる[^rewrite-in-typescript]。
- d3-selection が難しい。
  ドキュメントと実装とテストを読んで、ようやく理解できた関数がいくつかあった
- コピペしてそのまま動くコードがほしいのに、Observable のコード（そのままでは流用しづらい）しか出てこない

とはいえ、じゃあどうすればとっつきやすくなるか？と聞かれると返答に困る[^d3-improvement]ので、頑張ってD3に慣れるしかなさそうだ。

[^d3-naming]: メソッド名を短くしたいというのも理解できなくはないが、
細かい部分を調整したり、複雑な可視化を行えたりする、というD3の特性上、命名に関しては短くすることより、どちらかといえば冗長気味にしたほうがいいケースが多いと思う。
例えば、日々のデータハンドリングの途中で行う可視化（読み返されることを考慮する必要がほぼない）と、ウェブサイトで大きく表示される一点ものの複雑な可視化（書かれたあとにメンテナンスのために読み返されることが多い）だと、前者であれば簡潔なほうがよさそうだが、後者であれば冗長気味になっても可読性が高いほうがいいだろう。

[^rewrite-in-typescript]: [TypeScriptで型を付けようという提案が2018年に上がっているが却下されている](https://github.com/d3/d3/issues/3284)。
却下されるのは意外だなと最初は思ったが、d3配下の一つ一つのリポジトリはわりと小さいということもあって、最悪型なしでもなんとか理解できる、というのはあるかもしれない。

[^d3-improvement]: 関数名をもう少し長めにして、TypeScriptで型をつける、くらいでエンドユーザーの使用感はわりと改善しそうな気もするがどうだろうか？
まあそれくらいの違いだったら、D3の資産を捨ててまで別ライブラリに移行するほどでもなさそう。
