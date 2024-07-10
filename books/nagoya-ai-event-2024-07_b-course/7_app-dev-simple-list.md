---
title: 【開発】ChatGPT API でレシピを生成し表示する
---

## 1. レシピを生成し表示するコードを書く

さて、次は ChatGPT API にお料理レシピを生成してもらうい、さらに結果を表示する機能を実装していきましょう。

お料理レシピを生成してもらうために、以下のような処理を実装する必要があります。

1. ユーザーが入力した情報を受け取るために、フォーム要素を取得し、フォーム送信時のイベントを監視
2. ユーザーが入力した情報を添えて、ChatGPT に命令（リクエスト）
3. ChatGPT から返ってきた回答（レスポンス）を受け取る
4. リスト要素を取得し、ChatGPT から返ってきた回答を画面に表示

それを踏まえ、`./script.js` に以下のようなスクリプトを記述してみましょう。

```javascript:./script.js
/**
 * 自分専用の ChatGPT API の API Key
 *
 * 注意: この API KEY は公開してはいけません！！
 * ローカルで起動して使用する場合は問題ないですが、Web サイトとして公開する場合などは、この処理を削除し、代わりにサーバーサイAPI を呼び出す必要があります。
 */
const CHAT_GPT_API_KEY = 'sk-xxxx-xxxxxx...';

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

    // レシピリストの要素を空にする
    recipeListElement.innerHTML = '';

    // レシピリストの要素に回答を表示する
    recipeListElement.innerText = recipes;

    // ログを表示
    console.log('レシピを表示しました。', recipes);
});
```

記述が終ったら、Live Server で http://localhost:5500 にアクセスして、実際にフォームに文字を入力し、「レシピ生成」ボタンをクリックし、ChatGPT に命令して見ましょう。

このように、ChatGPT API の回答が画面に表示されていれば成功です！

![デモ](https://github.com/itnav/zenn-gura/blob/main/books/nagoya-ai-event-2024-07_b-course/assets/7_app-dev-simple-list/demo.mov?raw=true)

## 2. 今の実装の課題

現在の実装は、問題なく動いていはいますが、アプリケーションとして動かすにはまだ改善の余地があります。

アプリケーションを作成する際には、ユーザーが混乱しないように、アクションに応じて UI に変化を与え、ユーザーが使いやすいものにすることが重要です。\
また、予期しない事態が発生した場合には、ユーザーに対して何が起こったのかをわかりやすく伝えることも大切です。

### 課題

現在の実装には、以下のような課題があります。

A. エラーが発生しても UI が変わらないのでユーザーは何もわからない
B. 「レシピ生成」ボタンが連打できてしまう
C. レシピが表示されたのに、画面が変わらないのでユーザーが気づきにくい
D. ユーザーが入力した情報を ChatGPT に伝える際、人間にとってわかりやすい形になっているため、ChatGPT が誤解することがある

### 解決後のコード

最初に、これらの課題をすべてのコードを解決したコードを記述して見ましょう。

```javascript:./script.js
/**
 * 自分専用の ChatGPT API の API Key
 *
 * 注意: この API KEY は公開してはいけません！！
 * ローカルで起動して使用する場合は問題ないですが、Web サイトとして公開する場合などは、この処理を削除し、代わりにサーバーサイAPI を呼び出す必要があります。
 */
const CHAT_GPT_API_KEY = 'sk-xxxx-xxxxxx...';

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

さて、どのように課題を解決したのか、それぞれの課題について詳しく見ていきましょう。

### A. エラーが発生しても UI が変わらないのでユーザーは何もわからない

### B.「レシピ生成」ボタンが連打できてしまう問題

「レシピ生成」ボタンが連打できてしまう問題は、フォームが送信された後にボタンを無効化することで解決できます。

無効化の方法はいくつかありますが、今回はボタンをクリックできなくする CSS のクラスを定義しておき、フォームが送信された後にそのクラスを追加する方法を採用します。

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

また、もしスタイルが反映されずボタンを押された時などにも処理が走らないようにするために、以下の条件式も含めています。

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

### C. レシピが表示されたのに、画面が変わらないのでユーザーが気づきにくい

この問題は、ChatGPT API への命令が完了した際に処理画面をスクロールすることで解決できます。

```javascript
// リスト要素にスクロール
setTimeout(() => recipeListElement.scrollIntoView({ behavior: 'smooth' }));
```

setTimeout で囲っている理由としては、新しく追加した要素が表示された後にスクロールしてほしいからです。\
なぜ、setTimeout か処理を囲むと新しく追加した要素が表示されるのかを理解するためにはというのは DOM の描画処理実行タイミングとイベントループに関する JavaScript の知識が必要が必要になります。
