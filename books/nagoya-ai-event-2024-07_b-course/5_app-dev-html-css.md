---
title: 【開発】骨組み・外観の作成
---

## 骨組みを HTML で作成

まずは、HTML でアプリケーションの骨組みを作成していきましょう。

今回のアプリケーションで必ず必要になるのは以下の要素です。

1. タイトルの設定
2. CSS, JavaScript の読み込み
3. レシピの情報を入力するフォームの作成
4. レシピ一覧を表示する領域の確保
5. レシピの詳細を表示するモーダルの確保

これらをふまえて HTML でフォームを構成します。\
以下のコードを `index.html` に記述してください。

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

記述が完了したら、ファイルを保存し Live Server で http://localhost:5500 にアクセスして、フォームなどが実際に配置されていることを確認してみましょう！

## フォームの見た目を CSS で作成

次に、 CSS でフォームの見た目を作成します。

今回のアプリケーションで見た目を設定するべき箇所は以下の要素です。

1. アプリケーションの全体で適応される・適応できるスタイル
2. レシピの情報を入力するフォーム
3. レシピ一覧とレシピ
4. レシピの詳細を表示するモーダル

これらをふまえて CSS で見た目を整えていきます。\
以下のコードを `style.css` に記述してください。

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

記述が完了したら、ファイルを保存し Live Server で http://localhost:5500 にアクセスして、アプリケーションの見た目がかっこよくなっていることを確認しましょう！

## このチャプターの最終コード

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

```

:::
