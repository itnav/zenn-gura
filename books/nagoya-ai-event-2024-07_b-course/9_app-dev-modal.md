---
title: 【開発】モーダルの実装
---

モーダルとは、画面上に重ねて表示されるウィンドウのことです。\
主に、詳細情報を表示する際に利用されます。

```javascript:./script.js
// フォームの要素を取得
const recipeFormElement = document.getElementById('recipe-form');

// レシピのリストを取得
const recipeListElement = document.getElementById('recipe-list');

// モーダルのリストを取得
const recipeModalElement = document.getElementById('recipe-modal');

// Recipe Form が Submit された時に第２引数の関数を実行するように設定
recipeFormElement.addEventListener('submit', async (event) => {
    // リロードされるブラウザの仕様を防ぐ
    event.preventDefault();

    ...省略

    // エラーをハンドリングする
    try {
        /** @see {@link https://platform.openai.com/docs/api-reference/chat/create} API Reference */
        const recipeResponse = await fetch(...)

        ...省略

        // レシピをループしてリストに表示する HTML を生成する
        for (let i = 0; i < recipes.length; i++) {
            const recipe = recipes[i];

            ...省略

            // prettier-ignore
            // レシピ単体の中に子要素を追加する
            recipeItemElement.innerHTML = /*html*/ `...省略`;

            // ボタンクリック時にモーダルを表示するイベントハンドラを追加する
            recipeItemElement.addEventListener('click', () => {
                // prettier-ignore
                // モーダルの中身を置き換える
                recipeModalElement.innerHTML = /*html*/ `
                    <div class="recipe-modal-content">
                        <img class="recipe-modal-img" src="https://flagcdn.com/${recipe.country}.svg" alt="" onerror="this.outerHTML = '<div class=\'recipe-modal-img\'></div>'" />
                        <h2 class="recipe-modal-title">${recipe.title}</h2>
                        <div class="recipe-modal-tags">
                            <span class="recipe-modal-tag recipe-modal-type">${recipe.type}</span>
                            <span class="recipe-modal-tag">約${recipe.time}</span>
                            <span class="recipe-modal-tag">${recipe.servings}</span>
                            <span class="recipe-modal-tag">${recipe.difficulty}</span>
                        </div>
                        <p class="recipe-modal-description">${recipe.description}</p>
                        <section class="recipe-modal-section">
                            <h3 class="recipe-modal-subtitle">材料</h3>
                            <ul class="recipe-modal-list">${recipe.ingredients.map((ingredient) => /*html*/`<li>${ingredient}</li>`).join('')}</ul>
                        </section>
                        <section class="recipe-modal-section">
                            <h3 class="recipe-modal-subtitle">材料</h3>
                            <ul class="recipe-modal-list">${recipe.steps.map((step) => /*html*/`<li>${step}</li>`).join('')}</ul>
                        </section>
                    </div>
                `;

                // モーダルを表示する
                recipeModalElement.classList.add('open');

                // モーダルがクリックされた時に第２引数の関数を実行するように設定
                recipeModalElement.addEventListener(
                    'click',
                    (event) => {
                        // モーダル本体がクリックされたとき、つまり、モーダルの子要素がクリックされなかったとき、モーダルを閉じる
                        if (recipeModalElement === event.target) {
                            recipeModalElement.classList.remove('open');
                        }
                    },
                    { once: true }
                );
            });

            // recipe list の子要素に要素を追加
            recipeListElement.appendChild(recipeItemElement);
        }

        // ↓ エラー時は catch の中の処理が実行される
    } catch (error) {
      ...省略

       // ↓ finally は try または catch が実行された後に実行される
    } finally {
      ...省略
    }
});
```
