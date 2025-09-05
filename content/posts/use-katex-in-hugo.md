+++
title = "Hugo でエスケープの問題を回避しつつ KaTeX を使う"
date = "2019-08-12"
description = "Hugo で構築したブログで [KaTeX](https://katex.org/) を使った数式組版を行いたいが、よく使われている auto-render extension を使う方式だとエスケープの問題がある。そこで、Hugo の shortcode を経由することでエスケープの問題を回避する。"
+++

!!! 注意: このブログは以前 Hugo でビルドしていたが、[現在は zola に移行済み](@/posts/renew-this-blog.md)のため、この記事の数式レンダリングの一部は動かない。!!!

## KaTeX の auto-render extension を使う方式

[Auto-render extension](https://katex.org/docs/autorender.html) という機能で、`$$` などで挟んだ LaTeX コードをレンダリングすることができるが、エスケープ周りにちょっと問題がある。

例えば、`$[a](b)$` の $ で挟まれた中身は通常の KaTeX でも有効な LaTeX コードだが、一旦 Markdown パーサーを通してしまうと `$<a href="b">a</a>$` と解釈されてしまい、意図しない表示になってしまう。

これは極端な例なのでは？という気もするが、Markdown パーサーに意図せず変換されてしまう例は、他にも下記のようにいくつか考えられる。

- `$a _{b_ c}$`
- `$a *b* c$`
- `$<a>$`

長めの式を書いていて気づかないうちにこの手のエラーに引っかかると、原因の特定だけでかなり時間を消費してしまうので、なんとかして防ぎたい。

* [KaTeX の数式を LaTeX の\\\[...\\\] と $...$ で記述し，それを Hugo で生成するページで表示する \- Qiita](https://qiita.com/mametank/items/fa2b8a03598c9548e461)
  - あきらめてエスケープで我慢している
* [Hugo meets kramdown \+ KaTeX \#gohugo \| takuti\.me](https://takuti.me/note/hugo-kramdown-and-katex/)
  - Markdown パーサーを差し替えて回避している


## Shortcodes を使う方式

Hugo の [shortcodes](https://gohugo.io/content-management/shortcodes/) で独自のマークアップを作ることで、エスケープの問題を回避することができる。

インラインの数式と別行立ての数式用の shortcode はそれぞれ以下の通り。

layouts/shortcodes/eq.html:
```html
<span class="tex" data-expr="{{ .Get 0 }}"></span>
```

layouts/shortcodes/eq-display.html:
```html
<div class="tex displaystyle" data-expr="\displaystyle {{ trim .Inner "\n\r" }}"></div>
```

後から CSS で参照する都合上、displaystyle をクラスに追加しておいたが、別に必須ではない。

数式を入れたページで通常通り KaTeX のファイルをロードしつつ、下記のような JS を実行すれば数式が表示される。

```js
(function() {
  var tex = document.getElementsByClassName("tex");
  Array.prototype.forEach.call(tex, function(el) {
    katex.render(el.getAttribute("data-expr"), el);
  });
})();
```

例えば下記のような感じで使う[^example]。
[^example]: 例は <https://katex.org/> より

```
See how it renders with {{<eq "\KaTeX">}}:

{{< eq-display >}}
  f(x) = \int_{-\infty}^\infty
    \hat f(\xi)\,e^{2 \pi i \xi x}
    \,d\xi
{{< /eq-display >}}
```

するとこんな感じでレンダリングされる。

>See how it renders with {{<eq "\KaTeX">}}:
>
>{{< eq-display >}}
>  f(x) = \int_{-\infty}^\infty
>    \hat f(\xi)\,e^{2 \pi i \xi x}
>    \,d\xi
>{{< /eq-display >}}

ただし、auto-render extension 方式に比べると明らかに記法が煩雑なので、こちらの方式が必ずしも良いとは言い切れない。
簡潔だがたまにエスケープで厄介なことになる記法を好むか、煩雑だがエスケープの問題がない記法を好むか、好みが分かれるかなと思う。
