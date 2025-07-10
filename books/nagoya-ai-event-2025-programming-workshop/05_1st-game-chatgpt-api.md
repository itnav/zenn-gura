---
title: '💛 ChatGPT API とは'
---

## ChatGPT API とは

前のセクションで無事にマップとロボットが表示されましたね！！
ですが、現状のままだとロボットが動きません...🤔

これは ChatGPT と通信するための「鍵」が設定されていないからです。\
このセクションでは、API について簡単に学びつつ、ChatGPT と通信するための「鍵」となる API キーを `secret.js` に設定していきましょう！

<br />

## API とは？

API とは "Application Programming Interface" の略で、プログラムから別のプログラムへ、仕事の **「お願い（リクエスト）」をするための窓口** です！

例えば今回であれば「お宝探しゲーム」から「ChatGPT」へ、「API」という窓口を通して「自分はどのように動けばいいか教えて！」といったお願いをします。\
すると、それが API を経由して ChatGPT に伝えられ、ChatGPT が「答え」を返してくれます。

そして、その「答え」をゲームが解析し、ロボットを動かしているのです！

![API のイラスト](/images/nagoya-ai-event-2025-programming-workshop/05_chatgpt-api/01_chatgpt-api.png)

<br />

## API でできること

- **自分のアプリから ChatGPT を使える**

  - 自分が作ったゲームやアプリに AI 機能を追加できる
  - Web ブラウザを開かなくても ChatGPT と会話できる

- **プログラムによる柔軟なカスタマイズ**
  - JavaScript のコードから ChatGPT に質問を送れる
  - AI の性格や返答の形式を自由に設定できる
  - AI の返答を受け取って、ゲームの動作に反映できる

<br />

## `secret.js` に値を設定する

では、さっそく API を使うための API キーを設定します！

本講習を受けた方はデスクトップに **`chat-gpt-api-key-day1.txt`** もしくは **`chat-gpt-api-key-day2.txt`** というファイルがあるはずなので、そのテキストファイルを開いてください。

そしたら、そこにある `sk-` から始まる文字列をコピーして、 `secret.js` の値に書き換えてください。

```javascript:./secret.js
/**
 * OpenAI(ChatGPT) API の Key。
 *
 * 注意： この API Key は公開してはいけません！！
 *       ローカルで起動して使用する場合は問題ないのですが、Web サイトとして公開する場合などは、API Key を必要としている処理をサーバーサイドで記述するなど、API Key は隠す必要があります。
 */
export const OPENAI_API_KEY = 'sk-ここに配布されたAPIキーを入力';
```

:::message
API キーは「sk-」で始まる長い文字列です。全部コピーして貼り付けてください。
:::

<br />

もし個人で API キーを用意したい場合は、以下の記事を参考にしてください。
https://zenn.dev/umi_mori/books/chatbot-chatgpt/viewer/how_to_use_openai_api#api%E3%81%AE%E7%99%BA%E8%A1%8C%E6%96%B9%E6%B3%95

<br />

## ChatGPT API を動かしてみる

これまでに作成したコードを動かしてみましょう！
正常に設定が完了すると、実行ボタンを押すことでロボットが無造作に動き始めます！🎉

---

> ロボットが動き始めた画像を表示

---

もし、うまく動かない場合は `secret.js` の API キーが正しく設定されているか、もう一度確認してみてください。

<br />

:::message
この段階では、まだロボットはお宝に向かって正しく動きません。\
次のチャプターでロボットへの指示を正しく設定すると、ロボットが動くようになります。
:::

<br />

## API キーを使う上での注意点

:::message alert
ChatGPT の API キーは、絶対に公開してはいけません。
:::

ChatGPT の API キーは絶対に公開してはいけません！\
悪意のあるユーザーによって API が利用され、大量の請求が発生してしまう可能性があります。

そのため、今回作成したものを Web サイト上に公開する場合は、ChatGPT にリクエストを送る処理をサーバーサイドで行い、そのサーバーサイドの API を呼び出すようにする必要があります！

<br />

## 本チャプター終了時点のコード

### 基盤ファイル

:::details ./ai-fetch.js

```javascript

```

:::

:::details ./game.js

```javascript

```

:::

:::details ./secret.js

```javascript

```

:::

### マップファイル

:::details ./map-1/ai-route-prompt.js

```javascript

```

:::details ./map-1/index.html

```html

```

:::

:::details ./map-1/map.js

```javascript

```

:::

:::details ./map-1/style.css

```css

```

:::
