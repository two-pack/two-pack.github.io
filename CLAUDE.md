# ブログ記事の追加

記事ファイルは `docs/_posts/YYYY-MM-DD-<slug>.md` に作成する。

## パターン1: 外部記事へのリダイレクト（Qiitaなど）

```
---
layout: redirect
title: <記事タイトル>
redirect_to: <元記事URL>
---
```

## パターン2: このブログに直接書く記事

```
---
layout: page
title: <記事タイトル>
---

Markdown本文...
```

- 画像は `docs/assets/YYYY-MM-DD-<説明>.png` に置き、`/assets/...` で参照する

## ローカル確認

```
cd docs && bundle install && bundle exec jekyll serve --livereload
```

→ http://127.0.0.1:4000/

# 依存関係の更新

- ビルドは自前の workflow で行う。`ruby/setup-ruby` と `bundle exec jekyll build` で、`docs/Gemfile.lock` にある gem をそのまま使う
  - `actions/jekyll-build-pages` は、コンテナ画像の gem が2024年8月のまま更新されておらず、ロックした gem が使われないため、やめた
  - Jekyll は 3.10.0 のまま。`github-pages` gem が固定している
- `docs/Gemfile.lock` はコミットする。本番ビルドで使う gem の版を決め、Dependabot が脆弱性を検出するためにも必要
- 記事の URL は日付を含む。ビルドのタイムゾーンで日付がずれるため、ローカルで出力を比べるときは `TZ=UTC` を付ける
- workflow の `uses:` を変えたら `gh actions-lock` を実行し、`.github/workflows/actions.lock` を更新してコミットする
  - 拡張機能は `gh extension install github/gh-actions-lock` で入れる
  - Dependabot が出すアクション更新のプルリクエストでは、ロックファイルも一緒に更新される
- `verify-lockfile` workflow が、ロックファイルと workflow のずれをプルリクエストで検出する
