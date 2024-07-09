---
title: 【開発】フォームの実装
---

さて、まず手始めに、ユーザーが作りたい料理の情報を入力するためのフォームを作成していきましょう。

## 1. フォームの構成・配置を設定

フォームは、ユーザーが入力するための入力欄やボタンなどをまとめたものです。\
今回は、以下の６つの入力欄を持つフォームを作成します。

1. 料理名・食材（必須）
2. 提供人数
3. 調理時間
4. 調理難易度
5. アレルギー情報
6. 自由欄

それを踏まえて HTML でフォームを構成します。\
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
    </main>
  </body>
</html>
```

以上で、フォームの構成・配置が完了しました。\
ファイルを保存した後 Live Server で http://localhost:5500 にアクセスしてフォームが表示されることを確認してみましょう！

## 2. フォームの見た目を設定

次に、 CSS でフォームの見た目を設定します。\
以下のコードを `style.css` に記述してください。

```css:./style.css



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

以上で、フォームのスタイルの実装が完了しました。\
ファイルを保存した後 Live Server で http://localhost:5500 にアクセスしてフォームがかっこよくなっていることを確認しましょう！

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

```

:::
