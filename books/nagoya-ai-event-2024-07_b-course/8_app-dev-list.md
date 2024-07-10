---
title: 【開発】ChatGPT API でレシピをより詳細に表示する
---

前回のチャプターでは、単純な回答を ChatGPT から取得し、表示するまで実装することができました。\
ですが、これだけだとアプリケーションとして物足りないですよね。

そこで、今回は ChatGPT から、より詳細なレシピの情報を取得し、それを表示するまでの実装を行っていきます。

## より詳細なレシピの情報を取得するためには

まずは、ChatGPT API からレシピの詳細情報を取得する処理を実装していきます。

ChatGPT API から、より詳細な情報を取得するためには、以下のアプローチを取ることができます。

1. ChatGPT に対して、回答の例を提示する
2. ChatGPT に対して、回答の例に対する説明を提示する
3. ChatGPT に対して、どういった形式で回答を返してほしいかを提示する（詳細なデータをプログラム上で理解できるようにするため）

## プロンプトを改善するコードを書く

```javascript:./script.js
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
        const recipes = JSON.parse(recipeAnswer.choices[0].message.content);

        // レシピリストの要素を空にする
        recipeListElement.innerHTML = '';

        // レシピをループしてリストに表示する HTML を生成する
        for (let i = 0; i < recipes.length; i++) {
            const recipe = recipes[i];

            // レシピ単体の要素を作成する
            const recipeItemElement = document.createElement('button');

            // <button class="recipe-item"></button> となる
            recipeItemElement.classList.add('recipe-item');

            // prettier-ignore
            // レシピ単体の中に子要素を追加する
            recipeItemElement.innerHTML = /*html*/ `
                <img class="recipe-item-img" src="https://flagcdn.com/${recipe.country}.svg" alt="" onerror="this.outerHTML = '<div class=\'recipe-item-img\'></div>'" />
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
