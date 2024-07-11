---
title: 【開発】ChatGPT API でレシピを生成し表示する機能
---

次は ChatGPT API にお料理レシピを生成してもらい、回答を表示する機能を実装していきましょう。

:::message
このチャプターから、かなり難易度が上がっていきます！

そのため、「追いつくのが厳しい」「疲れてきた」など感じましたら、[**📝 コードを記述する**](#📝-コードを記述する) セクションのコードや、[**本チャプター終了時点のコード**](#本チャプター終了時点のコード) などをコピペして、アプリケーションがどのように動作するのかを確認するだけでも問題ないです！無理せず講習を進めていきましょう 👍
:::

## レシピを生成し表示する機能の実装

### 🧐 実装する機能の確認

お料理レシピを生成してもらうために、以下のような処理を実装する必要があります。

1. ユーザーが入力した情報を受け取るために、フォーム要素を取得し、フォーム送信時のイベントを監視
2. ユーザーが入力した情報を添えて、ChatGPT に命令（リクエスト）
3. ChatGPT から返ってきた回答（レスポンス）を受け取る
4. リスト要素を取得し、ChatGPT から返ってきた回答を画面に表示

それぞれ、どのように実装するかを見ていきましょう。

#### 1. ユーザーが入力した情報を受け取るために、フォーム要素を取得し、フォーム送信時のイベントを監視

フォーム要素とは、HTML の `<form id="recipe-form">` 要素のことです。\
フォーム要素の取得は `document.getElementById` メソッドを使用します。

```javascript
// HTML の id="recipe-form" の要素を取得
const recipeFormElement = document.getElementById('recipe-form');
```

そして、フォーム送信時のイベントを監視するために、`addEventListener` メソッドを使用します。

```javascript
// フォーム の入力が確定した時に第２引数の関数を実行するように設定
recipeFormElement.addEventListener('submit', (event) => {
  // フォームが送信された時の処理

  // 入力されている情報は、recipeFormElement.name属性で指定した名前.value で取得できる。
  console.log(recipeFormElement.ingredients.value);
});
```

#### 2. ユーザーが入力した情報を添えて、ChatGPT に命令（リクエスト）

今回は、ChatGPT に送るメッセージが大きくなってしまうので、あらかじめ変数に格納しておきます。

どういう AI かの説明をする `recipeAiPrompt` 変数と、レシピを出力させるための命令をする `recipeRequestPrompt` 変数の２つを用意します。

```javascript
/** どういう AI かを設定するための説明 */
const recipeAIPrompt = `
    あなたはユーザーの要望を元に、複数の料理のレシピを提案するアシスタントです。
`;

/** レシピを出力させるための命令（リクエスト） */
const recipeRequestPrompt = `
    以下の条件から、料理のレシピを提案してください。
    食材: ${recipeFormElement.ingredients.value},
    提供人数: ${recipeFormElement.servings.value},
    調理時間: ${recipeFormElement.time.value},
    調理難易度: ${recipeFormElement.difficulty.value},
    アレルギー情報: ${recipeFormElement.allergies.value},
    そのほかの要望: ${recipeFormElement.notes.value},
`;
```

そして、そのプロンプトをもとに、ChatGPT にリクエストを送信します。

```javascript
/** @see {@link https://platform.openai.com/docs/api-reference/chat/create} API Reference */
const recipeResponse = await fetch(
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
        // ChatGPT がどういった AI かを説明
        { role: 'system', content: recipeAIPrompt },

        // ChatGPT に命令する
        { role: 'user', content: recipeRequestPrompt },
      ],
    }),
  }
);
```

#### 3. リスト要素を取得し、ChatGPT から返ってきた回答を画面に表示

`fetch()` 関数実行後の結果は HTTP 通信 の情報であるため、そのままでは ChatGPT の回答が取得できません。\
そのため、`response.json()` メソッドを使用して、JSON 形式のデータを取得します。

```javascript
const recipeAnswer = await recipeResponse.json();
```

さらに、`response.json()` で取得した値は ID や質問者の情報など、回答の以外の情報も含まれているため、抽出する必要があります。

```javascript
const recipes = recipeAnswer.choices[0].message.content;
```

#### 4. リスト要素を取得し、ChatGPT から返ってきた回答を画面に表示

リスト要素とは、HTML の `<form id="recipe-list">` 要素のことです。\
リスト要素の取得は `document.getElementById` メソッドを使用します。

```javascript
// HTML の id="recipe-list" の要素を取得
const recipeListElement = document.getElementById('recipe-list');
```

そして、取得したリスト要素に、ChatGPT から返ってきた回答を表示します。

```javascript
// レシピリストの要素に回答を表示する
recipeListElement.innerText = recipes;
```

### 📝 コードを記述する

では、レシピを生成して表示する機能を実際したコードを `./script.js` に記述してみましょう。

```javascript:./script.js
/**
 * 自分専用の ChatGPT API の API Key
 *
 * 注意: この API KEY は公開してはいけません！！
 * ローカルで起動して使用する場合は問題ないですが、Web サイトとして公開する場合などは、この処理を削除し、代わりにサーバーサイAPI を呼び出す必要があります。
 */
const CHAT_GPT_API_KEY =
    'sk-xxxx-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx';

// フォームの要素を取得
const recipeFormElement = document.getElementById('recipe-form');

// レシピのリストを取得
const recipeListElement = document.getElementById('recipe-list');

// Recipe Form が Submit された時に第２引数の関数を実行するように設定
recipeFormElement.addEventListener('submit', async (event) => {
    // リロードされるブラウザの仕様を防ぐ
    event.preventDefault();

    /** どういう AI かを設定する */
    const recipeAIPrompt = `
        あなたはユーザーの要望を元に、複数の料理のレシピを提案するアシスタントです。
    `;

    /** レシピを出力させるための命令（リクエスト） */
    const recipeRequestPrompt = `
        以下の条件から、料理のレシピを提案してください。
        食材: ${recipeFormElement.ingredients.value},
        提供人数: ${recipeFormElement.servings.value},
        調理時間: ${recipeFormElement.time.value},
        調理難易度: ${recipeFormElement.difficulty.value},
        アレルギー情報: ${recipeFormElement.allergies.value},
        そのほかの要望: ${recipeFormElement.notes.value},
    `;

    /** @see {@link https://platform.openai.com/docs/api-reference/chat/create} API Reference */
    const recipeResponse = await fetch(
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
                  // ChatGPT がどういった AI かを説明
                  { role: 'system', content: recipeAIPrompt },

                  // ChatGPT に命令する
                  { role: 'user', content: recipeRequestPrompt }
                ],
            }),
        }
    );

    // ChatGPT からの回答に関する情報を取得
    const recipeAnswer = await recipeResponse.json();

    // ChatGPT からの回答をもとに、レシピのリストを取得
    const recipes = recipeAnswer.choices[0].message.content;

    // レシピリストの要素に回答を表示する
    recipeListElement.innerText = recipes;

    // ログを表示
    console.log('レシピを表示しました。', recipes);
});
```

記述が完了したら、ファイルを保存し Live Server で http://localhost:5500 にアクセスして、実際にフォームに文字を入力し、「レシピ生成」ボタンをクリックし、ChatGPT に命令して見ましょう。

このように、ChatGPT API の回答が画面に表示されていれば成功です！

![デモ](https://github.com/itnav/zenn-gura/blob/main/books/nagoya-ai-event-2024-07_b-course/assets/7_app-dev-simple-list/demo.gif?raw=true)

## もっとアプリケーションっぽくする

現在の実装は、問題なく動いていはいますが、アプリケーションとして動かすにはまだ改善の余地があります。

アプリケーションを作成する際には、ユーザーが混乱しないように、アクションに応じて UI に変化を与え、ユーザーが使いやすいものにすることが重要です。\
また、予期しない事態が発生した場合には、ユーザーに対して何が起こったのかをわかりやすく伝えることも大切です。

### 🤔 課題

現在の実装には、以下のような課題があります。

1. エラーが発生しても UI が変わらないのでユーザーは何もわからない
2. 「レシピ生成」ボタンが連打できてしまう
3. レシピが表示されたのに、表示している画面に変化がないのでユーザーが気づきにくい
4. ユーザーが入力した情報を ChatGPT に伝える際、人間にとってわかりやすい形になっているため、ChatGPT が誤解することがある

### 🔅 課題の解決方法

#### 1. エラーが発生しても UI が変わらないのでユーザーは何もわからない

エラーが発生した時に UI を表示するために、まずは「エラーが発生した」といった現象を捉える必要があります。

エラーが発生した時の条件分岐は、`try` `catch` `finally` 構文を使用することで実装できます。

```javascript
try {
  // エラーしたことを捉えたい処理を try の中に記述する
  const recipeResponse = await fetch('...');

  // リクエストが失敗していた場合、throw を実行することで、エラーを発生させる
  if (!recipeResponse.ok) {
    // `recipeResponse.json()` で取得できるエラーの詳細を error 変数にセットする
    throw await recipeResponse.json();
  }

  // ...省略
} catch (error) {
  // エラーが発生した場合の処理を catch で囲む

  // エラーを表示
  console.error(error);

  // エラーを文字列化
  const strError =
    error instanceof Error ? error.message : JSON.stringify(error, null, 2);

  // エラーを表示
  recipeListElement.innerHTML = /* html */ `
      <div>
          <h2 class="recipe-error-title">エラーが発生しました</h2>
          <pre class="recipe-error-message"><code>${strError}</code></pre>
      </div>
  `;
} finally {
  // 括弧内の処理（try, catch）が完了したタイミングで実行される
}
```

エラー時の処理（`catch`）には、リスト要素の中にエラーが発生したということを表示する HTML を追加しています。

#### 2.「レシピ生成」ボタンが連打できてしまう問題

「レシピ生成」ボタンが連打できてしまう問題は、フォームが送信された後にボタンを無効化することで解決できます。

ボタン無効化の方法はいくつかありますが、今回はボタンをクリックできなくする CSS のクラスを定義しておき、フォームが送信された後にそのクラスを追加する方法を採用します。

```javascript
// 処理中ということを示すために loading クラスを追加
recipeFormElement.classList.add('loading');
```

```css
/** loading がついているときだけ、このクラスが適応される */
.loading .recipe-form-submitter {
  pointer-events: none; /** pointer-events: none; が付与されると、クリックできなくなる */
  cursor: not-allowed;
  background-color: #d6e3ed;
}
```

また、もしスタイルが反映されずボタンを押された時などにも処理が走らないようにするために、以下のコードを最初の処理に追記しています。

```javascript
// 処理中は、重複して処理が実行されないように loading クラスがついていたら処理を中断する
if (recipeFormElement.classList.contains('loading')) {
  return;
}
```

最後に、処理が終わった後に loading クラスを削除することで、ボタンが再度クリックできるようになります。\
これは、処理の成功時・エラー時、どちらでも発火してほしいので `finally` の中に記述します。

```javascript
try {
  ...省略

} catch {
  ...省略

} finally {
  // loading クラスを削除
  recipeFormElement.classList.remove('loading');
}
```

#### 3. レシピが表示されたのに、表示している画面に変化がないのでユーザーが気づきにくい

この問題は、ChatGPT API への命令が完了した際に処理画面をスクロールすることで解決できます。

```javascript
// リスト要素にスクロール
setTimeout(() => recipeListElement.scrollIntoView({ behavior: 'smooth' }));
```

setTimeout で囲っている理由としては、新しく追加した要素が表示された後にスクロールしてほしいからです。

#### 4. ユーザーが入力した情報を ChatGPT に伝える際、人間にとってわかりやすい形になっているため、ChatGPT が誤解することがある

この問題は、広く使用されている形式でフォーマットした文字列を ChatGPT に伝えることで解決できます。

今回は `JSON.stringify()` メソッドを使用して、オブジェクトを JSON 形式の文字列に変換しています。

```javascript
/** ユーザーの入力情報をまとめたオブジェクト */
const recipeUserInput = {
  食材: recipeFormElement.ingredients.value,
  提供人数: recipeFormElement.servings.value,
  調理時間: recipeFormElement.time.value,
  調理難易度: recipeFormElement.difficulty.value,
  アレルギー情報: recipeFormElement.allergies.value,
  そのほかの要望: recipeFormElement.notes.value,
};

/** レシピを出力させるためのプロンプト */
const recipeRequestPrompt = `
    以下の条件から、複数の料理のレシピをなるべく多く提案してください。
    ${JSON.stringify(recipeUserInput)}
`;
```

`JSON` について詳しく知りたい方は、以下のリンクを参考にしてください。

https://reffect.co.jp/html/what_is_json

### 📝 解決後のコードを記述する

さあ、それらの課題の解決策を実装したコードを、`./script.js` に記述してみましょう。

```javascript:./script.js
/**
 * 自分専用の ChatGPT API の API Key
 *
 * 注意: この API KEY は公開してはいけません！！
 * ローカルで起動して使用する場合は問題ないですが、Web サイトとして公開する場合などは、この処理を削除し、代わりにサーバーサイAPI を呼び出す必要があります。
 */
const CHAT_GPT_API_KEY =
    'sk-xxxx-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx';

// フォームの要素を取得
const recipeFormElement = document.getElementById('recipe-form');

// レシピのリストを取得
const recipeListElement = document.getElementById('recipe-list');

// Recipe Form が Submit された時に第２引数の関数を実行するように設定
recipeFormElement.addEventListener('submit', async (event) => {
    // リロードされるブラウザの仕様を防ぐ
    event.preventDefault();

    // 処理中は、重複して処理が実行されないように loading クラスがついていたら処理を中断する
    if (recipeFormElement.classList.contains('loading')) {
        return;
    }

    // 処理中ということを示すために loading クラスを追加
    recipeFormElement.classList.add('loading');

    /** ユーザーの入力情報 */
    const recipeUserInput = {
        食材: recipeFormElement.ingredients.value,
        提供人数: recipeFormElement.servings.value,
        調理時間: recipeFormElement.time.value,
        調理難易度: recipeFormElement.difficulty.value,
        アレルギー情報: recipeFormElement.allergies.value,
        そのほかの要望: recipeFormElement.notes.value,
    };

    /** レシピを出力させるためのプロンプト */
    const recipeRequestPrompt = `
        以下の条件から、複数の料理のレシピをなるべく多く提案してください。
        ${JSON.stringify(recipeUserInput)}
    `;

    // エラーをハンドリングする
    try {
        /** @see {@link https://platform.openai.com/docs/api-reference/chat/create} API Reference */
        const recipeResponse = await fetch(
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
                        { role: 'user', content: recipeRequestPrompt },
                    ],
                }),
            }
        );

        // エラーが発生した場合はエラーメッセージを表示
        if (!recipeResponse.ok) {
            throw await recipeResponse.json();
        }

        // ChatGPT からの回答に関する情報を取得
        const recipeAnswer = await recipeResponse.json();

        // ChatGPT からの回答をもとに、レシピのリストを取得
        const recipes = JSON.parse(recipeAnswer.choices[0].message.content);

        // レシピリストの要素を空にする
        recipeListElement.innerHTML = '';

        // レシピリストの要素に回答を表示する
        recipeListElement.innerText = recipes;

        // ログを表示
        console.log('レシピを表示しました。', recipes);

        // ↓ エラー時は catch の中の処理が実行される
    } catch (error) {
        // エラーを表示
        console.error(error);

        // エラーを文字列化
        const strError =
            error instanceof Error
                ? error.message
                : JSON.stringify(error, null, 2);

        // エラーを表示
        recipeListElement.innerHTML = /* html */ `
            <div>
                <h2 class="recipe-error-title">エラーが発生しました</h2>
                <pre class="recipe-error-message"><code>${strError}</code></pre>
            </div>
        `;

        // ↓ finally は try または catch が実行された後に実行される
    } finally {
        // loading クラスを削除
        recipeFormElement.classList.remove('loading');

        // リスト要素にスクロール
        setTimeout(() =>
            recipeListElement.scrollIntoView({ behavior: 'smooth' })
        );
    }
});
```

記述が終ったら、Live Server で http://localhost:5500 にアクセスして、どこが変わったか、確認して見ましょう。

どうですか？ UI に変化を加えるだけでも、一気に「アプリケーション感」が増しましたね！\
さらに、エラー時の対応も含んでいるので、もしエラーが発生してもユーザーにわかりやすく表示されるようになりました。

<br />

---

## 本チャプター終了時点のコード

以下のコードは、このチャプターの終了時点のものです。
講座に追いつけなくなった場合は、これらのコードをコピーして貼り付けることで、進行状況に追いつくことができます。

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

            <div id="recipe-list"></div>

            <div id="recipe-modal"></div>
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

/**
  * ===========
  * Recipe List
  * ===========
  */

#recipe-list {
    display: grid;
    grid-template-columns: repeat(auto-fit, 240px);
    gap: 24px 80px;
    place-content: center space-evenly;
    min-height: 100dvh;
}

#recipe-list:empty {
    display: none;
}

.recipe-item {
    position: relative;
    box-sizing: border-box;
    display: flex;
    flex-direction: column;
    padding: 0;
    overflow: hidden;
    cursor: pointer;
    background: #fafafa;
    border: none;
    border-radius: 8px;
    border-top-left-radius: 120px;
    border-top-right-radius: 120px;
    outline: 0;
    box-shadow: 0 3px 1px -2px rgb(0 0 0 / 20%), 0 2px 2px 0 rgb(0 0 0 / 14%),
        0 1px 5px 0 rgb(0 0 0 / 12%);
    transition: box-shadow 80ms ease-in-out;
}

.recipe-item:hover {
    box-shadow: 0 3px 5px -1px rgb(0 0 0 / 20%), 0 6px 10px 0 rgb(0 0 0 / 14%),
        0 1px 18px 0 rgb(0 0 0 / 12%);
}

.recipe-item-img {
    box-sizing: border-box;
    flex-shrink: 0;
    width: 240px;
    height: 240px;
    background: #e4edf4;
    border-bottom-right-radius: 120px;
    border-bottom-left-radius: 120px;
    box-shadow: 0 3px 1px -2px rgb(0 0 0 / 20%), 0 2px 2px 0 rgb(0 0 0 / 14%),
        0 1px 5px 0 rgb(0 0 0 / 12%);
    object-fit: cover;
}

.recipe-item-content {
    box-sizing: border-box;
    display: flex;
    flex-direction: column;
    width: 100%;
    height: 100%;
    padding: 8px;
    margin-top: 8px;
}

.recipe-item-title {
    display: -webkit-box;
    margin-top: 0;
    margin-bottom: 8px;
    overflow: hidden;
    font-size: 20px;
    font-weight: 600;
    text-overflow: ellipsis;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
}

.recipe-item-type {
    display: -webkit-box;
    margin-bottom: 24px;
    overflow: hidden;
    font-size: 14px;
    color: #a9a9a9;
    text-overflow: ellipsis;
    -webkit-line-clamp: 1;
    -webkit-box-orient: vertical;
}

.recipe-item-tags {
    text-align: right;
}

.recipe-error-title {
    text-align: center;
}

.recipe-error-message {
    font: inherit;
    font-size: 16px;
    white-space: pre-wrap;
}

/**
  * ============
  * Recipe Modal
  * ============
  */

#recipe-modal {
    position: fixed;
    top: 0;
    left: 0;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    width: 100%;
    height: 100%;
    pointer-events: none;
    background-color: rgb(0 0 0 / 60%);
    opacity: 0;
    transition: opacity 160ms ease-in-out;
}

#recipe-modal.open {
    pointer-events: auto;
    opacity: 1;
}

.recipe-modal-content {
    box-sizing: border-box;
    width: min(800px, 88%);
    height: 88%;
    padding: min(40px, 4vw);
    margin: auto;
    overflow-y: scroll;
    overscroll-behavior: contain;
    background-color: #fafafa;
    border-radius: 16px;
}

.recipe-modal-content::-webkit-scrollbar {
    display: none;
}

.recipe-modal-img {
    width: 100%;
    aspect-ratio: 3 / 2;
    background-color: #e4edf4;
    border-radius: 24px;
    object-fit: cover;
}

.recipe-modal-title {
    margin-top: 32px;
    margin-bottom: 8px;
}

.recipe-modal-subtitle {
    margin-top: 32px;
    margin-bottom: 8px;
}

.recipe-modal-tags {
    overflow: auto hidden;
    white-space: nowrap;
}

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

.recipe-modal-type {
    background-color: #ffd100;
}

.recipe-modal-description {
    margin-top: 1.25em;
    font-size: 18px;
}

.recipe-modal-list {
    padding-left: 24px;
    margin-top: 0;
    margin-bottom: 16px;
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
const CHAT_GPT_API_KEY =
    'sk-xxxx-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx';

// フォームの要素を取得
const recipeFormElement = document.getElementById('recipe-form');

// レシピのリストを取得
const recipeListElement = document.getElementById('recipe-list');

// Recipe Form が Submit された時に第２引数の関数を実行するように設定
recipeFormElement.addEventListener('submit', async (event) => {
    // リロードされるブラウザの仕様を防ぐ
    event.preventDefault();

    // 処理中は、重複して処理が実行されないように loading クラスがついていたら処理を中断する
    if (recipeFormElement.classList.contains('loading')) {
        return;
    }

    // 処理中ということを示すために loading クラスを追加
    recipeFormElement.classList.add('loading');

    /** ユーザーの入力情報 */
    const recipeUserInput = {
        食材: recipeFormElement.ingredients.value,
        提供人数: recipeFormElement.servings.value,
        調理時間: recipeFormElement.time.value,
        調理難易度: recipeFormElement.difficulty.value,
        アレルギー情報: recipeFormElement.allergies.value,
        そのほかの要望: recipeFormElement.notes.value,
    };

    /** レシピを出力させるためのプロンプト */
    const recipeRequestPrompt = `
        以下の条件から、複数の料理のレシピをなるべく多く提案してください。
        ${JSON.stringify(recipeUserInput)}
    `;

    // エラーをハンドリングする
    try {
        /** @see {@link https://platform.openai.com/docs/api-reference/chat/create} API Reference */
        const recipeResponse = await fetch(
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
                        { role: 'user', content: recipeRequestPrompt },
                    ],
                }),
            }
        );

        // エラーが発生した場合はエラーメッセージを表示
        if (!recipeResponse.ok) {
            throw await recipeResponse.json();
        }

        // ChatGPT からの回答に関する情報を取得
        const recipeAnswer = await recipeResponse.json();

        // ChatGPT からの回答をもとに、レシピのリストを取得
        const recipes = JSON.parse(recipeAnswer.choices[0].message.content);

        // レシピリストの要素を空にする
        recipeListElement.innerHTML = '';

        // レシピリストの要素に回答を表示する
        recipeListElement.innerText = recipes;

        // ログを表示
        console.log('レシピを表示しました。', recipes);

        // ↓ エラー時は catch の中の処理が実行される
    } catch (error) {
        // エラーを表示
        console.error(error);

        // エラーを文字列化
        const strError =
            error instanceof Error
                ? error.message
                : JSON.stringify(error, null, 2);

        // エラーを表示
        recipeListElement.innerHTML = /* html */ `
            <div>
                <h2 class="recipe-error-title">エラーが発生しました</h2>
                <pre class="recipe-error-message"><code>${strError}</code></pre>
            </div>
        `;

        // ↓ finally は try または catch が実行された後に実行される
    } finally {
        // loading クラスを削除
        recipeFormElement.classList.remove('loading');

        // リスト要素にスクロール
        setTimeout(() =>
            recipeListElement.scrollIntoView({ behavior: 'smooth' })
        );
    }
});
```

:::
