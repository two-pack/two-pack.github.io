---
layout: page
title: OpenAI CUAモデルのTesting Agent Demo（を見たメモ）
---

# はじめに
[Testing Agent Demo](https://github.com/openai/openai-testing-agent-demo)
Open AIの [Computer-Using Agent (CUA) モデル](https://platform.openai.com/docs/models/computer-use-preview) を使ったデモ。
触ってみようと思った結果、Tier3以上でないとそもそもモデルが使えないことが分かって終了！でしたが、ちょっと見たことのメモ。

# とりあえずREADME通りやってみる

サクッと行くと思いましたが、
* Windows 11
* Ubuntu 24.04.1 on WSL2

という環境のせいもあってか、以下のあたりがうまくいきませんでした。（深追いせず）

* sample-test-appがそもそも動かずエラーになる。
* cua-serverでPlaywrightを動かす際に、ディスプレイ無いよエラーになる。
    * これはヘッドレスにするようにコードを変えれば動いた。
* cua-serverは.envを見てくれない。

とはいえ、起動すると以下の感じでテストケースを自然言語で入力する画面が表示されます。
![テストケースの入力画面](/assets/2025-06-15-demo-screenshot-01.png)

**Variables** は、URLなどを設定する画面です。
![設定画面](/assets/2025-06-15-demo-screenshot-02.png)

**Submib** すると、テストケースを解釈してテスト手順を作成、それを実行していきます。
![テスト実行画面](/assets/2025-06-15-demo-screenshot-03.png)

前述の通り、サンプルアプリが動かなかったこともあり、テストは動いていません。
別のサンプルアプリでもやってみましたが、今度はモデルのエラーになっていて、これはOpenAIの課金が足りずTier3になっていないので、そもそもCUAのモデルが使える状態じゃなかったという落ちでした。
![モデルエラー](/assets/2025-06-15-model-error.png)

# 実装について

動かないのでちょっと実装を見ていた際のメモ。

## プロンプト

[https://github.com/openai/openai-testing-agent-demo/blob/main/cua-server/src/lib/constants.ts](https://github.com/openai/openai-testing-agent-demo/blob/main/cua-server/src/lib/constants.ts)

ここにプロンプトがあります。
テストケースをテストステップに解釈しなおすプロンプトと、テストステップをCUAに食わせてテストを実行するプロンプトです。
なるほど、こんな感じでステップに落とすといいのかと思って、Geminiに同じ間でスクリーンショットと合わせて投げてみたらいい感じでした。

## Playwrightの操作

[https://github.com/openai/openai-testing-agent-demo/blob/main/cua-server/src/handlers/cua-loop-handler.ts](https://github.com/openai/openai-testing-agent-demo/blob/main/cua-server/src/handlers/cua-loop-handler.ts)
[https://github.com/openai/openai-testing-agent-demo/blob/main/cua-server/src/handlers/action-handler.ts](https://github.com/openai/openai-testing-agent-demo/blob/main/cua-server/src/handlers/action-handler.ts
)

この辺で行ってるようで、スクリーンショットを取ってテストステップを繰り返し確認しながら動かしているようです。

# というわけで

追加課金したらCUA modelを試してみたいですが、参考にしたプロンプトと [Aether](https://github.com/autifyhq/aethr) との組み合わせもやってみたいところ。