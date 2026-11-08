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

- ビルドは GitHub Pages の標準ビルド（`actions/jekyll-build-pages`）で、Jekyll は 3.10.0 に固定されている
- `docs/Gemfile.lock` はコミットする。Dependabot が gem の実バージョンを見て、脆弱性を検出するために必要
- workflow の `uses:` を変えたら `gh actions-lock` を実行し、`.github/workflows/actions.lock` を更新してコミットする
  - 拡張機能は `gh extension install github/gh-actions-lock` で入れる
  - Dependabot が出すアクション更新のプルリクエストでは、ロックファイルも一緒に更新される
- `verify-lockfile` workflow が、ロックファイルと workflow のずれをプルリクエストで検出する
