---
title: 【開発】ChatGPT API でレシピを生成し表示する
---

さて、次は ChatGPT API にお料理レシピを生成してもらう機能を実装していきます。\（そのため、前のセクションで記述したスクリプトはすべて削除してください。）
お料理レシピを生成してもらうために、以下のような処理を実装する必要があります。

1. ユーザーが入力した情報を受け取るために、フォーム要素を取得し、フォーム送信時のイベントを監視
2. ユーザーが入力した情報を添えて、ChatGPT に質問（リクエスト）
3. ChatGPT から返ってきた回答（レスポンス）を受け取る
4. リスト要素を取得し、ChatGPT から返ってきた回答を画面に表示

それをふまえ、`./script.js` に以下のようなスクリプトを記述してみましょう。\
（`CHAT_GPT_API_KEY` は削除しないでください）

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

    /** レシピを出力させるためのプロンプト */
    const recipePromptMessage = `
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
                messages: [{ role: 'user', content: recipePromptMessage }],
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
