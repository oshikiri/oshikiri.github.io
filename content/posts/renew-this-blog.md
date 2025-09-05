+++
title = "このサイトの構成をzola+GitHub Pagesに変更した"
date = 2023-08-24
description = "2023年にこのサイトの構成を更新したときの備忘録として、それぞれの理由などの記録を残しておく。"
+++


## 静的サイトジェネレーターをHugoからzolaに変えた

[zola]はRustで書かれた静的サイトジェネレーター（static site generator; SSG）である。
zolaは[2016-12から開発されている](https://github.com/getzola/zola/graphs/contributors)。
私が前に使っていたJekyll [(2008-10〜)](https://github.com/jekyll/jekyll/graphs/contributors)やHugo [(2013-06〜)](https://github.com/gohugoio/hugo/graphs/contributors)と比較するとかなり新しい部類に入る。

[zola]: https://github.com/getzola/zola

今回Hugoからzolaに変えたわけだが、他にもあるSSGの中からzolaを選んだ強い理由はない。
Hugoに不満があったわけではないが、せっかく作り直すなら別の似たようなツールを試しておこうかな、と思ってzolaを選んでみた[^zola-vs-hugo]。
今のところ、使い勝手はそこまで変わらない。


[^zola-vs-hugo]:
ただ、zolaを選んでおいてこんなことを言うのも変だが、速さと機能の多さなどを考慮に入れると、2023年現在ではHugoのほうが優れているような気はする。
zolaのREADMEには[Hugoのテンプレートエンジンに不満があった]と書かれているので、そこは改善されていると期待はできるかもしれない。
あとzolaには[Hugoの `.GitInfo`](https://gohugo.io/variables/git/) に相当する機能がない。[一応リクエストは上がっている](https://zola.discourse.group/t/git-info-variables-like-hugo/1448)が、対応される見込みは今のところなさそう。

[Hugoのテンプレートエンジンに不満があった]: https://github.com/getzola/zola/blob/74056d15ab6ee46a336463c9098745f14564aae1/README.md?plain=1#L11-L12


## ローカルでの手動ビルド&デプロイから、GitHub Actionsでの自動ビルド&デプロイに変えた

せっかくGitHub Pagesを使うのであれば(?)、GitHub Actionsを使って、masterブランチにpushされるたびにデプロイされるようにしておきたい。
このブログでは、ビルド+デプロイの処理は [build-deploy.yaml] （以下のyaml）で行っている。

[build-deploy.yaml]: https://github.com/oshikiri/oshikiri.github.io/blob/14d59eb0729e306a80736cef4e92e8e260b0f82a/.github/workflows/build-deploy.yaml

```yaml
name: Zola on GitHub Pages

on:
 push:
  branches:
   - master

jobs:
  build:
    name: Build and upload
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@v3.5.3
    - name: Install zola
      uses: taiki-e/install-action@v2.16.0
      with:
        tool: zola@0.17.2
    - name: Build
      run: |
        zola build
    - name: Upload artifact
      uses: actions/upload-pages-artifact@v2.0.0
      with:
        path: ./public

  deploy:
    name: Deploy
    needs: build
    permissions:
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2.0.4
```

後半はほとんど [Publishing with a custom GitHub Actions workflow - GitHub Docs](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site#publishing-with-a-custom-github-actions-workflow) をそのまま実行している。
ただし、この機能は一応[まだベータ版ということになっている](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site#publishing-with-a-custom-github-actions-workflow)のでそこは少し気にしておく必要がありそう。


## ホスティング先をAmazon S3からGitHub Pagesに変えた
ホスティング先については、これまではなにも噛ませずにS3に置いている状態になっていた。
まあアクセス数が多いわけではないのでそのままでもいいといえばいいのだが、さすがに改善したほうがいいかなと思って、GitHub Pagesに変えた。
そこまでヘビーに使うわけでもないし[^heavy]、GitHub Pagesで十分だと思われる。

GitHub Pagesのドメイン (`oshikiri.github.io`) のままでもいいのだが、せっかくドメインを持っているのでカスタムドメインを設定しておく。
カスタムドメインの設定についても調べてみたが、どうやらかなり簡単にできるようだった。

- [GitHub Pages サイトのカスタムドメインを管理する - GitHub Docs](https://docs.github.com/ja/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site)
- [Github Pages 独自ドメインの証明書エラー - 行けたら行く](https://www.ted027.com/post/domain-certificate/)

[^heavy]: どういった状況になるとGitHub Pagesで足りなくなるか？→
  [GitHubのドキュメント（2023年8月現在）](https://docs.github.com/ja/pages/getting-started-with-github-pages/about-github-pages#usage-limits)によればサイズ制限（サイトが1GB未満）、デプロイのタイムアウトの制限（10分以内）、帯域幅制限（100GB/月まで）など、いくつか制限はあるのだが、個人のブログの通常の使い方であればほぼ引っかかることはないと思われる。
  帯域幅制限については、仮に1PVで100KBやりとりするとして、100GB / 100KB ≒ 1,000,000 なので100万PV/月になると問題になるかもしれないが、普通は気にしなくて良さそう。


## その他

- zolaのデフォルトの設定では、目次に付与される id がダサい（`静的サイトジェネレーターをHugoからzolaに変えた` が `#jing-de-saitozieneretawohugokarazolanibian-eta` になってしまう）。初めて知ったがこれは slugify と呼ばれる処理らしい。slugify を避ける方法はいくつかあるが、id に ascii 文字以外が入るのが許容できるのであれば、config.toml の `[slugify]` セクションに `anchors = "off"` を指定すればいい。
- 他のSSGと同じくzolaにもカスタムテーマが用意されているが、CSSの勉強のために最初はこれを一切使わないことにしてみた。
以前Hugoを使っていたときは既存のテーマを少し調整して使っていたが、結局は自分でCSSの調整をやりたくなってしまうし、どうせなら0から作ってみたほうがいいかなと思った。
- コメントシステムは [giscus](https://github.com/giscus/giscus) という GitHub Discussions を流用するライブラリが面白そうだった。必要になったら検討したい。
