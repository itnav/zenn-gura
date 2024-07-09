---
title: 【開発】ChatGPT API を使ってみる
---

いよいよ、ChatGPT API を使って ChatGPT に料理レシピを生成してもらう機能を実装していきます。

## 1. そもそも API とは？

## 2. ChatGPT API の用意

## 3. ChatGPT API を JavaScript から呼び出してみる

さっそく、ChatGPT API を JavaScript から呼び出してみましょう。

API は、基本的に `fetch` 関数を使用して呼び出します。\
まずは `fetch` 関数の形式を確認してみましょう。

```javascript
fetch('API の URL', { ...認証情報や質問内容など });
```

第１引数には API の URL を指定し、第２引数にはオプション情報を指定します。\
以外とシンプルです。

それをもとに、ChatGPT API を呼び出す テスト用 JavaScript を `./script.js` に書いてみましょう。

```javascript:./script.js
/**
 * 自分専用の ChatGPT API の API Key
 *
 * 注意: この API KEY は公開してはいけません！！
 * ローカルで起動して使用する場合は問題ないですが、Web サイトとして公開する場合などは、この処理を削除し、代わりにサーバーサイAPI を呼び出す必要があります。
 */
const CHAT_GPT_API_KEY = 'sk-xxxx-xxxxxx...';

// ログを表示
console.log('Chat GPT にリクエスト中...');

// ChatGPT に質問（リクエスト）
const chatGPTResponse = await fetch(
    'https://api.openai.com/v1/chat/completions',
    {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            Authorization: `Bearer ${CHAT_GPT_API_KEY}`,
        },
        body: JSON.stringify({
            model: 'gpt-3.5-turbo',
            messages: [
                // ChatGPT にメッセージを送信
                {
                    role: 'user',
                    content: 'こんにちは！',
                },
            ],
        }),
    }
);

// ChatGPT からの回答（レスポンス）を取得
const chatGPTAnswer = await chatGPTResponse.json();

// ログに ChatGPT API の回答（レスポンス）を表示
console.log('\n--------');
console.log('ChatGPT からの回答の全データ');
console.log(chatGPTAnswer);

// ログに ChatGPT API の回答を表示
console.log('\n--------');
console.log('ChatGPT からの回答');
console.log(chatGPTAnswer.choices[0].message.content);
```

記述が終ったら、Live Server で http://localhost:5500 にアクセスして、コンソールを確認してみましょう。\
コンソールは http://localhost:5500 を開いた状態で `F12` もしくは `右クリック > 検証 > Console タブ` で確認できます。

うまくいくと、以下のようなログが表示されているはずです！
![ChatGPT API のレスポンスのログ](/images/nagoya-ai-event-2024-07_b-course/6_app-dev-chatgpt-api/test-request-log.png)

実際に "ChatGPT からの回答" は、ChatGPT が返してきた回答が表示されているはずです。\
なんと、この記述だけで ChatGPT API を JavaScript から呼び出すことができました！

前のセクションで解説した通り、機能を扱うための複雑な処理は API を作成した側が実装してくれるので、API を使用する側は少ない記述量で素晴らしい機能を実装できるのです。

### さらに試してみる

ChatGPT API は普段 Web アプリケーションとして使っている ChatGPT と同じように使うことができるので、試しに「こんにちは！」というメッセージを好きなメッセージに変更し、もう一度ページをリロードしてみると、異なる回答が表示されるはずです。試して見ましょう！

### 後始末

今回は動作の確認のためのコードであるため、実際のアプリケーションでは使いませんので、次のセクションに移る際はこのコードを削除してください。\
ただし、`const CHAT_GPT_API_KEY` の部分は共通して使用するため、削除せずに残しておいてください。

## このセクションの最終ファイル

:::details HTML

```html:./index.html
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>お料理レシピ作成アプリ</title>
    <link rel="stylesheet" href="./style.css" />
    <script type="module" src="./script.js"></script>
  </head>

  <body>
    <main class="container">
      <form id="recipe-form">
        <div class="recipe-form-content">
          <h1 class="recipe-form-heading">お料理レシピ生成アプリ</h1>

          <span class="recipe-form-label">料理名・食材（必須）</span>
          <input
            class="recipe-form-input"
            name="ingredients"
            placeholder="例： パスタ、トマト"
            required
          />

          <span class="recipe-form-label">提供人数</span>
          <input
            class="recipe-form-input"
            name="servings"
            placeholder="例： 2人前"
          />

          <span class="recipe-form-label">調理時間</span>
          <input
            class="recipe-form-input"
            name="time"
            placeholder="例： 30分、サクッと"
          />

          <span class="recipe-form-label">調理難易度</span>
          <input
            class="recipe-form-input"
            name="difficulty"
            placeholder="例： かんたん、初級者向け、上級者向け"
          />

          <span class="recipe-form-label">アレルギー情報</span>
          <input
            class="recipe-form-input"
            name="allergies"
            placeholder="例： 卵、乳製品、小麦粉"
          />

          <span class="recipe-form-label">自由欄</span>
          <textarea
            class="recipe-form-textarea"
            name="notes"
            rows="3"
          ></textarea>

          <button class="recipe-form-submitter" type="submit">
            レシピ生成
          </button>
        </div>
      </form>
    </main>
  </body>
</html>
```

:::

:::details CSS

```css:./style.css
/**
* ====
* Core
* ====
*/

body {
  display: flex;
  flex-direction: column;
  padding: 0;
  margin: 0;
  overflow-y: scroll;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto,
      'Helvetica Neue', Arial, sans-serif;
  line-height: 1.6;
  background: #e0efff;
}

:where(:focus-visible) {
  outline: 1px solid #3ea8ff;
  box-shadow: 0 0 0 2.4px #e4edf4;
}

.container {
  box-sizing: border-box;
  width: 100%;
}

/**
* ======
* Common
* ======
*/

#recipe-form,
#recipe-list {
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  justify-content: center;
  padding: 24px;
}

.recipe-form-content,
.recipe-list-content {
  box-sizing: border-box;
  width: 100%;
  max-width: 800px;
  margin: auto;
}

.recipe-item-tags,
.recipe-modal-tags {
  margin-top: auto;
  overflow: auto hidden;
  white-space: nowrap;
}

.recipe-item-tag,
.recipe-modal-tag {
  box-sizing: border-box;
  display: inline-block;
  height: 24px;
  padding: 0 8px;
  font-size: 12px;
  font-weight: 600;
  line-height: 26px;
  color: white;
  background-color: #3ea8ff;
  border-radius: 4px;
}

/**
* ===========
* Recipe Form
* ===========
*/

#recipe-form {
  min-height: 96dvh;
  background-color: #fff;
  border-bottom-right-radius: 8vw;
  border-bottom-left-radius: 8vw;
}

.recipe-form-content {
  padding: 24px;
}

.recipe-form-heading {
  font-size: 24px;
  text-align: center;
}

.recipe-form-label {
  display: block;
  margin-top: 24px;
  font-weight: 600;
  color: #444;
}

.recipe-form-input,
.recipe-form-textarea,
.recipe-form-submitter {
  box-sizing: border-box;
  width: 100%;
  padding: 10px 12px;
  margin-top: 8px;
  font-size: 16px;
  background: #f5f9fc;
  border: 1px solid #d6e3ed;
  border-radius: 8px;
}

.recipe-form-input::placeholder,
.recipe-form-textarea::placeholder {
  color: #a9a9a9;
}

.recipe-form-textarea {
  resize: vertical;
}

.recipe-form-submitter {
  margin-top: 24px;
  font-size: 16px;
  font-weight: 600;
  color: white;
  cursor: pointer;
  background-color: #3ea8ff;
  border: none;
  outline-offset: 2px;
  transition: background-color 80ms ease-in-out;
}

.recipe-form-submitter:hover {
  background-color: #0f83fd;
}

.recipe-form-submitter:active {
  background-color: #0868ce;
}

.loading .recipe-form-submitter {
  pointer-events: none;
  cursor: not-allowed;
  background-color: #d6e3ed;
}
```

:::

:::details JavaScript

```js:./script.js
/**
 * 自分専用の ChatGPT API の API Key
 *
 * 注意: この API KEY は公開してはいけません！！
 * ローカルで起動して使用する場合は問題ないですが、Web サイトとして公開する場合などは、この処理を削除し、代わりにサーバーサイAPI を呼び出す必要があります。
 */
const CHAT_GPT_API_KEY = 'sk-xxxx-xxxxxx...';
```

:::
