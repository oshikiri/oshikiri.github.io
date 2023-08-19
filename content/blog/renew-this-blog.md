+++
title = "oshikiri.org の構成をzola+GitHub Pagesに変更した"
date = 2023-08-19
description = "2023年にこのサイトの構成を更新したときの備忘録として、それぞれの理由などの記録を残しておく。"
+++

書いている途中

## 静的サイトジェネレーターをHugoからzolaに変えた {#SSG}

[zola]はRustで書かれた静的サイトジェネレーター（static site generator; SSG）である。
zolaは[2016-12から開発されている](https://github.com/getzola/zola/graphs/contributors)。
私が前に使っていたJekyll [(2008-10〜)](https://github.com/jekyll/jekyll/graphs/contributors)やHugo [(2013-06〜)](https://github.com/gohugoio/hugo/graphs/contributors)と比較するとかなり新しい部類に入る。

[zola]: https://github.com/getzola/zola

今回Hugoからzolaに変えたわけだが、他にもあるSSGの中からzolaを選んだ強い理由はない。
Hugoに不満があったわけではないが、せっかく作り直すなら別の似たようなツールを試しておこうかな、と思ってzolaを選んでみた[^zola-vs-hugo]。

他のSSGと同じくzolaにもカスタムテーマが用意されているが、CSSの勉強のために最初はこれを一切使わないことにしてみた。
以前Hugoを使っていたときは既存テーマを少し調整して使っていたが、結局自分でCSSの調整をやりたくなってしまうので、これを気にテーマを使わずに0から作ってみる。
とりあえずダークモード前提で色を選択した。

[^zola-vs-hugo]:
ただ、zolaを選んでおいてこんなことを言うのも変だが、速さと機能の多さなどを考慮に入れると、2023年現在ではHugoのほうが優れているような気はする。
zolaのREADMEには[Hugoのテンプレートエンジンに不満があった]と書かれているので、そこは改善されていると期待はできるかもしれない。
あとzolaには[Hugoの `.GitInfo`](https://gohugo.io/variables/git/) に相当する機能がない。[一応リクエストは上がっている](https://zola.discourse.group/t/git-info-variables-like-hugo/1448)が、対応される見込みは今のところなさそう。

[Hugoのテンプレートエンジンに不満があった]: https://github.com/getzola/zola/blob/74056d15ab6ee46a336463c9098745f14564aae1/README.md?plain=1#L11-L12


## ローカルビルド&手動デプロイから、GitHub Actionsでの自動ビルド&デプロイに変えた {#deployment}
せっかくGitHub Pagesを使うのであれば(?)、GitHub Actionsでデプロイまで自動化しておきたい。
このブログでは、ビルド+デプロイの処理は[こちらのワークフロー]で行っている。注意すべきところは、

[こちらのワークフロー]: https://github.com/oshikiri/oshikiri.github.io/blob/master/.github/workflows/build-deploy.yaml

1. zolaを使ったビルドの処理は[shalzz/zola-deploy-action](https://github.com/shalzz/zola-deploy-action)で行っている。
本来このactionは、ビルド結果を gh-pages ブランチにコミットしてデプロイするところまでを担当するのだが、今回は後述する通りデプロイについては別の方法を使うので、`BUILD_ONLY: true` を設定してビルドまでで止めている。
zolaはデフォルトで `public/` にビルド結果を出力する。

    ```yaml
    - name: Install zola
      uses: taiki-e/install-action@v2.16.0
      with:
        tool: zola@0.17.2
    - name: Build
      run: |
        zola build
    ```

2. [actions/upload-pages-artifact](https://github.com/actions/upload-pages-artifact) の `path` に `public` を指定する。
[Publishing with a custom GitHub Actions workflow - GitHub Docs](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site#publishing-with-a-custom-github-actions-workflow)

    ```yaml
        - name: Upload artifact
          uses: actions/upload-pages-artifact@v2.0.0
          with:
            path: ./public
    ```

    そのあとは他と同様に actions/deploy-pages でデプロイする処理を発火させればよい。

    ```yaml
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

    ただし、この機能は一応[まだベータ版ということになっている](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site#publishing-with-a-custom-github-actions-workflow)のでそこは少し気になる。


## ホスティング先をAmazon S3からGitHub Pagesに変えた {#hosting}
ホスティング先については、なにも噛ませずにS3に置いていた状態のままになっていた。
まあアクセス数が多いわけではないのでそのままでもいいといえばいいのだが、さすがに改善したほうがいいかなと思って、GitHub Pagesに変えた。
そこまでヘビーに使うわけでもないし[^heavy]、GitHub Pagesで十分かなと判断した。

GitHub Pagesのままでもいいのだが、せっかくドメインを持っているのでカスタムドメインを設定しておく。
簡単にできるようだった。

- [GitHub Pages サイトのカスタムドメインを管理する - GitHub Docs](https://docs.github.com/ja/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site)
- [Github Pages 独自ドメインの証明書エラー - 行けたら行く](https://www.ted027.com/post/domain-certificate/)

[^heavy]: どういった状況になるとGitHub Pagesで足りなくなるか？→
  [GitHubのドキュメント](https://docs.github.com/ja/pages/getting-started-with-github-pages/about-github-pages#usage-limits)によればサイズ制限、デプロイのタイムアウトの制限、帯域幅制限など、いくつか制限はあるのだが、個人のブログであれば通常は引っかかることはないと思われる。
  「月当たり100GBのソフトな帯域幅制限」については、仮に1PVで100KBやりとりするとして、100GB / 100KB ≒ 1,000,000 なので100万PV/月になるまでは問題ないはず。


## その他 {#others}

- デフォルトでは目次につく id がダサい。これを避けるために、`## その他 {#others}` のように明示的に指定しておく必要がある。
- コメントシステムは giscus が面白そうだったので、必要になったら検討したい。
