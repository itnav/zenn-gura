---
title: 【開発】ChatGPT API でレシピをより詳細に表示する
---

前回のチャプターでは、単純な回答を ChatGPT から取得し、表示するまで実装することができました。\
ですが、これだけだとアプリケーションとして物足りないですよね。

そこで、今回は ChatGPT から、より詳細なレシピの情報を取得し、それを表示するまでの実装を行っていきます。

## ChatGPT API により詳細なレシピを教えてもらう

より詳細なレシピを表示するためには、詳細なレシピ情報を ChatGPT API から教えてもらう必要があります。

ChatGPT API から、詳細なレシピ情報を取得するには、以下のアプローチを取ることができます。

1. ChatGPT に対して、回答の例と、各パラメーターの説明を提示する
2. ChatGPT に対して、どういった形式で回答を返してほしいかを提示する

では、それぞれの具体的なアプローチを見ていきましょう。

### 1. ChatGPT に対して、回答の例を提示する

ChatGPT に対して、どのような回答を返してほしいかを提示することで、より詳細なレシピ情報を取得することができます。

今回は、チキンカレーのレシピを例に、回答の例を提示してみましょう。

```javascript
/** ChatGPT から受け取りたい回答（レスポンス）の例 */
const recipeAnswerExample = {
  title: 'シンプルおいしい！チキンカレーの基本レシピと作り方とコツ',
  description:
    '風味豊かでクリーミーなチキンカレーです。スパイスが効いたソースと柔らかいチキンが特徴で、白ご飯やナンとよく合います。',
  country: 'in',
  type: '主菜',
  time: '30分',
  servings: '4人前',
  difficulty: 'ふつう',
  ingredients: [
    '鶏胸肉500g、一口大に切る',
    '植物油大さじ2',
    '大きな玉ねぎ1個、みじん切り',
    'ニンニク3片、みじん切り',
    'ショウガ大さじ1、みじん切り',
    'カレーパウダー大さじ2',
    'クミンパウダー小さじ1',
    'コリアンダーパウダー小さじ1',
    'ターメリック小さじ1',
    'パプリカ小さじ1',
    'ココナッツミルク400ml（1缶）',
    'カットトマト400g（1缶）',
    'チキンブロス1カップ',
    '塩とコショウ、適量',
    '新鮮なパクチー、みじん切り（飾り用）',
  ],
  steps: [
    '中火で大きな鍋に植物油を熱する。',
    'みじん切りにした玉ねぎを加え、5分ほど黄金色になるまで炒める。',
    'みじん切りにしたニンニクとショウガを加え、さらに2分間炒める。',
    'カレーパウダー、クミンパウダー、コリアンダーパウダー、ターメリック、パプリカを加え、香りが立つまで1分ほど炒める。',
    '一口大に切った鶏胸肉を鍋に加え、ピンク色がなくなるまで約5-7分間炒める。',
    'ココナッツミルク、カットトマト、チキンブロスを加えてよく混ぜる。',
    '混ぜ合わせたら沸騰させ、その後火を弱めて20-25分間煮込み、鶏肉が完全に火が通り、ソースが濃くなるまで煮込む。',
    '塩とコショウで味を調える。',
    '新鮮なみじん切りのパクチーを飾りとして添えてから提供する。',
  ],
};
```

例を記述するだけでも、ChatGPT が回答してくれる内容が限定されてきますが、もっと思った通りに回答してもらうためには、例で与えたパラメーターに対する説明を与えるとよいでしょう。

```javascript
/** ChatGPT から受け取りたい回答（レスポンス）の例の説明 */
const recipeAnswerDescription = {
  title: '料理名を回答してください。',
  description: '料理の説明を回答してください。',
  country:
    '料理が発祥した国の IOS 3166 コードを回答してください。（例： jp, us, in ...）',
  type: '料理の種類を回答してください。',
  time: '料理の調理時間を回答してください。',
  servings: '料理の提供人数を回答してください。',
  difficulty: '料理の難易度を回答してください。',
  ingredients: '料理に使う材料をリストで回答してください。',
  steps: '料理の手順をリストで回答してください。',
};
```

そして、ChatGPT に与える命令は以下のように追記します。

```javascript
/** レシピを出力させるための命令（リクエスト） */
const recipeRequestPrompt = `
  ...省略

  回答は以下の例の形式に従ってください。
  ${JSON.stringify([recipeAnswerExample])}

  回答に関する各パラメーターの説明に関しては以下の通りです。
  ${JSON.stringify(recipeAnswerDescription)}
`;
```

これで、ChatGPT に対して、より詳細なレシピ情報を取得するための準備が整いました。

#### 2. ChatGPT に対して、どういった形式で回答を返してほしいかを提示する

ChatGPT が予期せぬ回答を返さないようにするために、回答の形式を提示することが重要です。\
加えて、間違って出力する可能性のあるデータを排除するための命令を ChatGPT に与えることも効果的です。

以下のようにプロンプトを ChatGPT に与えることで、回答の形式を指定することができます。

```javascript
/** レシピを出力させるための命令（リクエスト） */
const recipeRequestPrompt = `
  ...省略

  回答は JavaScript で JSON.parse() できるようにバックティックを含まない JSON 形式で返してください。
`;
```

このような命令を付け加えるだけで、ChatGPT がどのような形式で回答を返してほしいかを理解し、それに従って回答を返してくれるようになります。

### 📝 コードを書く

```javascript:./script.js
import { CHAT_GPT_API_KEY } from './secret.js';

// フォームの要素を取得
const recipeFormElement = document.getElementById('recipe-form');

// リストの要素を取得
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

    /** どういう AI かを設定する */
    const recipeAIPrompt = `
        あなたはユーザーの要望を元に、複数の料理のレシピを提案するアシスタントです。
    `;

    /** ユーザーの入力情報 */
    const recipeUserInput = {
        食材: recipeFormElement.ingredients.value,
        提供人数: recipeFormElement.servings.value,
        調理時間: recipeFormElement.time.value,
        調理難易度: recipeFormElement.difficulty.value,
        アレルギー情報: recipeFormElement.allergies.value,
        そのほかの要望: recipeFormElement.notes.value,
    };

    /** ChatGPT から受け取りたい回答（レスポンス）の例 */
    const recipeAnswerExample = {
        title: 'シンプルおいしい！チキンカレーの基本レシピと作り方とコツ',
        description:
            '風味豊かでクリーミーなチキンカレーです。スパイスが効いたソースと柔らかいチキンが特徴で、白ご飯やナンとよく合います。',
        country: 'in',
        type: '主菜',
        time: '30分',
        servings: '4人前',
        difficulty: 'ふつう',
        ingredients: [
            '鶏胸肉500g、一口大に切る',
            '植物油大さじ2',
            '大きな玉ねぎ1個、みじん切り',
            'ニンニク3片、みじん切り',
            'ショウガ大さじ1、みじん切り',
            'カレーパウダー大さじ2',
            'クミンパウダー小さじ1',
            'コリアンダーパウダー小さじ1',
            'ターメリック小さじ1',
            'パプリカ小さじ1',
            'ココナッツミルク400ml（1缶）',
            'カットトマト400g（1缶）',
            'チキンブロス1カップ',
            '塩とコショウ、適量',
            '新鮮なパクチー、みじん切り（飾り用）',
        ],
        steps: [
            '中火で大きな鍋に植物油を熱する。',
            'みじん切りにした玉ねぎを加え、5分ほど黄金色になるまで炒める。',
            'みじん切りにしたニンニクとショウガを加え、さらに2分間炒める。',
            'カレーパウダー、クミンパウダー、コリアンダーパウダー、ターメリック、パプリカを加え、香りが立つまで1分ほど炒める。',
            '一口大に切った鶏胸肉を鍋に加え、ピンク色がなくなるまで約5-7分間炒める。',
            'ココナッツミルク、カットトマト、チキンブロスを加えてよく混ぜる。',
            '混ぜ合わせたら沸騰させ、その後火を弱めて20-25分間煮込み、鶏肉が完全に火が通り、ソースが濃くなるまで煮込む。',
            '塩とコショウで味を調える。',
            '新鮮なみじん切りのパクチーを飾りとして添えてから提供する。',
        ],
    };

    /** ChatGPT から受け取りたい回答（レスポンス）の例の説明 */
    const recipeAnswerDescription = {
        title: '料理名を回答してください。',
        description: '料理の説明を回答してください。',
        country: '料理が発祥した国の IOS 3166 コードを回答してください。',
        type: '料理の種類を回答してください。',
        time: '料理の調理時間を回答してください。',
        servings: '料理の提供人数を回答してください。',
        difficulty: '料理の難易度を回答してください。',
        ingredients: '料理に使う材料をリストで回答してください。',
        steps: '料理の手順をリストで回答してください。',
    };

    /** レシピを出力させるための命令（リクエスト） */
    const recipeRequestPrompt = `
        以下の条件から、複数の料理のレシピをできるだけ多く提案してください。
        ${JSON.stringify(recipeUserInput)}

        回答は以下の例の形式に従い JSON 形式で返してください。
        ${JSON.stringify([recipeAnswerExample])}

        回答に関する各パラメーターの説明に関しては以下の通りです。
        ${JSON.stringify(recipeAnswerDescription)}

        回答は JavaScript で JSON.parse() できるようにバックティックや誤ったフォーマットを含まないシンプルな JSON 形式で返してください。
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
                        // ChatGPT がどういった AI かを説明
                        { role: 'system', content: recipeAIPrompt },

                        // ChatGPT に命令する
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
        const recipes = recipeAnswer.choices[0].message.content;

        // レシピリストの要素に回答を表示する
        recipeListElement.innerText = recipes;

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

記述が完了したら、ファイルを保存し Live Server で http://localhost:5500 にアクセスして、ChatGPT からの返答が変わっていることを確認しましょう！

![デモ](https://github.com/itnav/zenn-gura/blob/main/books/nagoya-ai-event-2024-07_b-course/assets/8_app-dev-list/demo.gif?raw=true)

次は、詳細になったデータを用いて UI を構築していきます！

## 詳細なレシピを表示する

さて、ChatGPT から詳細なレシピを取得することができたので、それを表示するための UI を構築していきましょう。

### 🧐 レシピを表示するまでの流れ

1. ChatGPT から取得したレシピ情報は文字列であるため、それを JavaScript のオブジェクトに変換する
2. レシピ情報をもとに、HTML を生成し追加する

### 📝 コードを書く

```javascript:./script.js
import { CHAT_GPT_API_KEY } from './secret.js';

// フォームの要素を取得
const recipeFormElement = document.getElementById('recipe-form');

// リストの要素を取得
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

    /** どういう AI かを設定する */
    const recipeAIPrompt = `
        あなたはユーザーの要望を元に、複数の料理のレシピを提案するアシスタントです。
    `;

    /** ユーザーの入力情報 */
    const recipeUserInput = {
        食材: recipeFormElement.ingredients.value,
        提供人数: recipeFormElement.servings.value,
        調理時間: recipeFormElement.time.value,
        調理難易度: recipeFormElement.difficulty.value,
        アレルギー情報: recipeFormElement.allergies.value,
        そのほかの要望: recipeFormElement.notes.value,
    };

    /** ChatGPT から受け取りたいデータの例 */
    const recipeAnswerExample = {
        title: 'シンプルおいしい！チキンカレーの基本レシピと作り方とコツ',
        description:
            '風味豊かでクリーミーなチキンカレーです。スパイスが効いたソースと柔らかいチキンが特徴で、白ご飯やナンとよく合います。',
        country: 'in',
        type: '主菜',
        time: '30分',
        servings: '4人前',
        difficulty: 'ふつう',
        ingredients: [
            '鶏胸肉500g、一口大に切る',
            '植物油大さじ2',
            '大きな玉ねぎ1個、みじん切り',
            'ニンニク3片、みじん切り',
            'ショウガ大さじ1、みじん切り',
            'カレーパウダー大さじ2',
            'クミンパウダー小さじ1',
            'コリアンダーパウダー小さじ1',
            'ターメリック小さじ1',
            'パプリカ小さじ1',
            'ココナッツミルク400ml（1缶）',
            'カットトマト400g（1缶）',
            'チキンブロス1カップ',
            '塩とコショウ、適量',
            '新鮮なパクチー、みじん切り（飾り用）',
        ],
        steps: [
            '中火で大きな鍋に植物油を熱する。',
            'みじん切りにした玉ねぎを加え、5分ほど黄金色になるまで炒める。',
            'みじん切りにしたニンニクとショウガを加え、さらに2分間炒める。',
            'カレーパウダー、クミンパウダー、コリアンダーパウダー、ターメリック、パプリカを加え、香りが立つまで1分ほど炒める。',
            '一口大に切った鶏胸肉を鍋に加え、ピンク色がなくなるまで約5-7分間炒める。',
            'ココナッツミルク、カットトマト、チキンブロスを加えてよく混ぜる。',
            '混ぜ合わせたら沸騰させ、その後火を弱めて20-25分間煮込み、鶏肉が完全に火が通り、ソースが濃くなるまで煮込む。',
            '塩とコショウで味を調える。',
            '新鮮なみじん切りのパクチーを飾りとして添えてから提供する。',
        ],
    };

    /** ChatGPT から受け取りたいデータの説明 */
    const recipeAnswerDescription = {
        title: '料理名を回答してください。',
        description: '料理の説明を回答してください。',
        country:
            '料理が発祥した国の IOS 3166 コードを回答してください。（例： jp, us, in ...）',
        type: '料理の種類を回答してください。',
        time: '料理の調理時間を回答してください。',
        servings: '料理の提供人数を回答してください。',
        difficulty: '料理の難易度を回答してください。',
        ingredients: '料理に使う材料をリストで回答してください。',
        steps: '料理の手順をリストで回答してください。',
    };

    /** レシピを出力させるためのプロンプト */
    const recipeRequestPrompt = `
        以下の条件から、複数の料理のレシピをなるべく多く提案してください。
        ${JSON.stringify(recipeUserInput)}

        回答は以下の例の形式に従ってください。
        ${JSON.stringify([recipeAnswerExample])}

        回答に関する各パラメーターの説明に関しては以下の通りです。
        ${JSON.stringify(recipeAnswerDescription)}

        回答は JavaScript で JSON.parse() できるようにバックティックを含まない JSON 形式で返してください。
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
                        // ChatGPT がどういった AI かを説明
                        { role: 'system', content: recipeAIPrompt },

                        // ChatGPT に命令する
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

        // レシピをループしてリストに表示する HTML を生成する
        for (let i = 0; i < recipes.length; i++) {
            const recipe = recipes[i];

            // レシピ単体の要素を作成する（<button></button>）
            const recipeItemElement = document.createElement('button');

            // レシピ単体の要素にクラスをつける（<button class="recipe-item"></button>）
            recipeItemElement.classList.add('recipe-item');

            // prettier-ignore
            // レシピ単体の中に子要素を追加する
            recipeItemElement.innerHTML = /*html*/ `
                <img class="recipe-item-img" src="https://flagcdn.com/${recipe.country}.svg" alt="" />
                <div class="recipe-item-content">
                    <h2 class="recipe-item-title">${recipe.title}</h2>
                    <div class="recipe-item-type">${recipe.type}</div>
                    <div class="recipe-item-tags">
                        <span class="recipe-item-tag">約${recipe.time}</span>
                        <span class="recipe-item-tag">${recipe.servings}</span>
                        <span class="recipe-item-tag">${recipe.difficulty}</span>
                    </div>
                </div>
            `;

            // recipe list の子要素に要素を追加
            recipeListElement.appendChild(recipeItemElement);
        }

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

記述が完了したら、ファイルを保存し Live Server で http://localhost:5500 にアクセスして、料理のレシピが表示されていることを確認しましょう！

![デモ２](https://github.com/itnav/zenn-gura/blob/main/books/nagoya-ai-event-2024-07_b-course/assets/8_app-dev-list/demo-2.gif?raw=true)

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

```javascript:./secret.js
/**
 * 自分専用の ChatGPT API の API Key
 *
 * 注意: この API Key は公開してはいけません！！
 * ローカルで起動して使用する場合は問題ないですが、Web サイトとして公開する場合などは、API Key を必要としている処理をサーバーサイドで記述するなど、API Key は隠す必要があります
 */
export const CHAT_GPT_API_KEY =
    'sk-xxxx-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx';
```

```javascript:./script.js
import { CHAT_GPT_API_KEY } from './secret.js';

// フォームの要素を取得
const recipeFormElement = document.getElementById('recipe-form');

// リストの要素を取得
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

    /** どういう AI かを設定する */
    const recipeAIPrompt = `
        あなたはユーザーの要望を元に、複数の料理のレシピを提案するアシスタントです。
    `;

    /** ユーザーの入力情報 */
    const recipeUserInput = {
        食材: recipeFormElement.ingredients.value,
        提供人数: recipeFormElement.servings.value,
        調理時間: recipeFormElement.time.value,
        調理難易度: recipeFormElement.difficulty.value,
        アレルギー情報: recipeFormElement.allergies.value,
        そのほかの要望: recipeFormElement.notes.value,
    };

    /** ChatGPT から受け取りたいデータの例 */
    const recipeAnswerExample = {
        title: 'シンプルおいしい！チキンカレーの基本レシピと作り方とコツ',
        description:
            '風味豊かでクリーミーなチキンカレーです。スパイスが効いたソースと柔らかいチキンが特徴で、白ご飯やナンとよく合います。',
        country: 'in',
        type: '主菜',
        time: '30分',
        servings: '4人前',
        difficulty: 'ふつう',
        ingredients: [
            '鶏胸肉500g、一口大に切る',
            '植物油大さじ2',
            '大きな玉ねぎ1個、みじん切り',
            'ニンニク3片、みじん切り',
            'ショウガ大さじ1、みじん切り',
            'カレーパウダー大さじ2',
            'クミンパウダー小さじ1',
            'コリアンダーパウダー小さじ1',
            'ターメリック小さじ1',
            'パプリカ小さじ1',
            'ココナッツミルク400ml（1缶）',
            'カットトマト400g（1缶）',
            'チキンブロス1カップ',
            '塩とコショウ、適量',
            '新鮮なパクチー、みじん切り（飾り用）',
        ],
        steps: [
            '中火で大きな鍋に植物油を熱する。',
            'みじん切りにした玉ねぎを加え、5分ほど黄金色になるまで炒める。',
            'みじん切りにしたニンニクとショウガを加え、さらに2分間炒める。',
            'カレーパウダー、クミンパウダー、コリアンダーパウダー、ターメリック、パプリカを加え、香りが立つまで1分ほど炒める。',
            '一口大に切った鶏胸肉を鍋に加え、ピンク色がなくなるまで約5-7分間炒める。',
            'ココナッツミルク、カットトマト、チキンブロスを加えてよく混ぜる。',
            '混ぜ合わせたら沸騰させ、その後火を弱めて20-25分間煮込み、鶏肉が完全に火が通り、ソースが濃くなるまで煮込む。',
            '塩とコショウで味を調える。',
            '新鮮なみじん切りのパクチーを飾りとして添えてから提供する。',
        ],
    };

    /** ChatGPT から受け取りたいデータの説明 */
    const recipeAnswerDescription = {
        title: '料理名を回答してください。',
        description: '料理の説明を回答してください。',
        country:
            '料理が発祥した国の IOS 3166 コードを回答してください。（例： jp, us, in ...）',
        type: '料理の種類を回答してください。',
        time: '料理の調理時間を回答してください。',
        servings: '料理の提供人数を回答してください。',
        difficulty: '料理の難易度を回答してください。',
        ingredients: '料理に使う材料をリストで回答してください。',
        steps: '料理の手順をリストで回答してください。',
    };

    /** レシピを出力させるためのプロンプト */
    const recipeRequestPrompt = `
        以下の条件から、複数の料理のレシピをなるべく多く提案してください。
        ${JSON.stringify(recipeUserInput)}

        回答は以下の例の形式に従ってください。
        ${JSON.stringify([recipeAnswerExample])}

        回答に関する各パラメーターの説明に関しては以下の通りです。
        ${JSON.stringify(recipeAnswerDescription)}

        回答は JavaScript で JSON.parse() できるようにバックティックを含まない JSON 形式で返してください。
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
                        // ChatGPT がどういった AI かを説明
                        { role: 'system', content: recipeAIPrompt },

                        // ChatGPT に命令する
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

        // レシピをループしてリストに表示する HTML を生成する
        for (let i = 0; i < recipes.length; i++) {
            const recipe = recipes[i];

            // レシピ単体の要素を作成する（<button></button>）
            const recipeItemElement = document.createElement('button');

            // レシピ単体の要素にクラスをつける（<button class="recipe-item"></button>）
            recipeItemElement.classList.add('recipe-item');

            // prettier-ignore
            // レシピ単体の中に子要素を追加する
            recipeItemElement.innerHTML = /*html*/ `
                <img class="recipe-item-img" src="https://flagcdn.com/${recipe.country}.svg" alt="" />
                <div class="recipe-item-content">
                    <h2 class="recipe-item-title">${recipe.title}</h2>
                    <div class="recipe-item-type">${recipe.type}</div>
                    <div class="recipe-item-tags">
                        <span class="recipe-item-tag">約${recipe.time}</span>
                        <span class="recipe-item-tag">${recipe.servings}</span>
                        <span class="recipe-item-tag">${recipe.difficulty}</span>
                    </div>
                </div>
            `;

            // recipe list の子要素に要素を追加
            recipeListElement.appendChild(recipeItemElement);
        }

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
