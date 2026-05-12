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
cd docs && bundle exec jekyll serve --livereload
```

→ http://127.0.0.1:4000/
