---
title: 'マップとロボットの作成'
---

## マップとロボットの作成

いよいよ、3D マップのマップとロボットを作成します！\
コードを書いて、ゲームを表示してみましょう！

<br />

## 必要なフォルダとファイルを作成する

VSCode の下のパネルに「ターミナル」が表示されていることを確認してください！\
もし表示されていなければ、以下のショートカットキーでターミナルを開きます！

- **Windows の場合**: `Ctrl + j`
- **Mac 場合**: `⌘ (Command) + j`

![ターミナルが表示されている VSCode のサイドバー](/images/nagoya-ai-event-2025-programming-workshop/04_map-and-robot-setup/01_opened-terminal-panel.png)

お使いのパソコンに合わせて、以下のコマンドをコピーしてターミナルに貼り付け、Enter キーを押してください！

**Windows の場合**

```bash
mkdir map-1
type nul > map-1/ai-route-prompt.js
type nul > map-1/index.html
type nul > map-1/map.js
type nul > map-1/style.css
```

**Mac の場合**

```bash
mkdir map-1
touch map-1/ai-route-prompt.js
touch map-1/index.html
touch map-1/map.js
touch map-1/style.css
```

コマンドを実行すると、一気に左側のファイル一覧に新しいファイルとフォルダが表示されます。これで冒険の準備は完了です！

![作成されたフォルダとファイルが表示されている VSCode のサイドバー](/images/nagoya-ai-event-2025-programming-workshop/04_map-and-robot-setup/02_created-files-and-folders.png)

<br />

## コードを記述する

それでは、前回の章と同じ要領でコードを記述していきましょう！

### 1. `index.html`

:::details ファイルの中身（コピー&ペースト）

```html
<!DOCTYPE html>
<html lang="ja">
    <head>
        <meta charset="UTF-8" />
        <title>Map 1</title>
        <link rel="stylesheet" href="./style.css" />
        <script type="module" defer>
            import { OPENAI_API_KEY } from '../../secret.js';
            import { fetchRoutePathWithOpenAI } from '../../ai-fetch.js';
            import { generateSystemPrompt } from '../../ai-system-prompt.js';
            import { setupGame } from '../../game.js';
            import { routePrompt } from './ai-route-prompt.js';
            import { mapConfig } from './map.js';

            // DOM 要素の取得
            const gameViewerEl = document.getElementById('game-viewer');
            const responseViewerEl = document.getElementById(
                'game-ai-response-viewer'
            );
            const triggerEl = document.getElementById('game-trigger');

            // ゲームをセットアップ
            await setupGame({
                gameViewerEl,
                responseViewerEl,
                triggerEl,
                mapConfig,
                pathFetcher: () =>
                    fetchRoutePathWithOpenAI({
                        apiKey: OPENAI_API_KEY,
                        systemPrompt: generateSystemPrompt(mapConfig),
                        routePrompt,
                    }),
            });
        </script>
    </head>

    <body>
        <div id="game-viewer"></div>
        <div id="game-ai-response-viewer"></div>
        <button id="game-trigger"></button>
    </body>
</html>
```

:::

### 2. `style.css`

:::details ファイルの中身（コピー&ペースト）

```css
html,
body {
    margin: 0;
}

body {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    min-height: 100svh;
}

#game-viewer {
    width: 80vw;
    height: 80vh;
    overflow: hidden;
    display: flex;
    justify-content: center;
    align-items: center;
    margin-bottom: 24px;
}

#game-trigger {
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 10px 16px;
    font-size: 14px;
    border-radius: 6px;
    background-color: #10b981;
    border: 1px solid rgba(16, 185, 129, 0.1);
    color: #fff;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.2s ease-in-out;
    transform: scale(1);
}

#game-trigger:hover:not(:disabled) {
    background-color: #059669;
    transform: scale(1.05);
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

#game-trigger:active:not(:disabled) {
    background-color: #047857;
    transform: scale(0.98);
}

#game-trigger:disabled {
    background-color: #9ca3af;
    border-color: #d1d5db;
    cursor: not-allowed;
    opacity: 0.6;
    transform: scale(1);
}

#game-ai-response-viewer {
    background-color: #f3f4f6;
    border: 1px solid #e5e7eb;
    border-radius: 8px;
    padding: 16px;
    margin-bottom: 16px;
    max-width: 400px;
    min-height: 60px;
    display: block;
}

#ai-response-label {
    font-size: 12px;
    color: #6b7280;
    font-weight: 600;
    margin-bottom: 8px;
}

#ai-response {
    font-size: 24px;
    color: #1f2937;
    font-weight: 500;
    letter-spacing: 4px;
    opacity: 0;
    transition: opacity 0.3s ease-in-out;
    min-height: 32px;
    display: flex;
    align-items: center;
}

#ai-response.visible {
    opacity: 1;
}

#ai-response span {
    display: inline-block;
    opacity: 0;
    animation: fadeInArrow 0.4s ease-in-out forwards;
}

@keyframes fadeInArrow {
    from {
        opacity: 0;
        transform: translateY(-10px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

.spinner {
    display: inline-block;
    width: 16px;
    height: 16px;
    border: 2px solid rgba(255, 255, 255, 0.3);
    border-top-color: #fff;
    border-radius: 50%;
    animation: spin 0.8s linear infinite;
    margin-right: 8px;
}

@keyframes spin {
    to {
        transform: rotate(360deg);
    }
}
```

:::

### 3. `map.js`

:::details ファイルの中身（コピー&ペースト）

```javascript
/**
 * 現在のマップの設定オブジェクトです。
 *
 * このオブジェクトのプロパティ値を変更することで、\
 * マップのレイアウト、スタート/エンド位置、罠の位置を変更できます。
 *
 * @type {import("../game").MapConfig}
 */
export const mapConfig = {
    layout: [
        ['s', 'n', 'n', 'n', 'n', 't'],
        ['n', 'n', 'o', 'n', 'n', 'n'],
        ['n', 'n', 'n', 'n', 'n', 'n'],
        ['n', 'n', 'n', 't', 'n', 'n'],
        ['n', 'n', 'n', 'n', 'n', 'n'],
        ['o', 'n', 'n', 'n', 'n', 'e'],
    ],
    cell: {
        s: {
            type: 'start',
            color: '#6B7ADB',
        },
        e: {
            type: 'end',
            color: '#F59E0B',
        },
        t: {
            type: 'trap',
            color: '#DC2626',
        },
        o: {
            type: 'object',
            color: '#92400E',
        },
        n: {
            type: 'normal',
            color: '#D4A574',
        },
    },
};
```

:::

### 4. `ai-route-prompt.js`

:::details ファイルの中身（コピー&ペースト）

```javascript
/**
 * ロボットに指示を出すためのプロンプトです。
 */
export const routePrompt = `
    好きなように動いてください！
`;
```

:::

<br />

## ゲームを起動してみる

ここまでできたら、一度ゲームを確認してみましょう！

### 1. すべてのファイルを保存する

まずは全てのファイルが保存されているか確認しましょう！\
上のタブのファイル名の右端に ⚪️ がついている場合はセーブされていないことを示しています。

![セーブできていないファイルのガイド](/images/nagoya-ai-event-2025-programming-workshop/04_map-and-robot-setup/03_unsaved-files.png)

ファイルのセーブは、

- **Windows の場合**: `Ctrl + s`
- **Mac の場合**: `⌘ (Command) + s`

で行えます！

### 2. VSCode の「Go Live」ボタンをクリック

続いて、VSCode の右下の「Go Live」ボタンをクリックします。

![Go Live ボタンの位置](/images/nagoya-ai-event-2025-programming-workshop/04_map-and-robot-setup/04_go-live-button-guide.png)

:::message
「Go Live」ボタンが見つからない場合は、拡張機能「Live Server」がインストールされていない可能性があります。スタッフに声をかけてください。
:::

### 3. ブラウザで「map-1」フォルダを選択

「Go Live」ボタンをクリックすると、ブラウザが起動し以下のような画面が表示されます！

![ブラウザでファイル一覧が表示されている画面](/images/nagoya-ai-event-2025-programming-workshop/04_map-and-robot-setup/05_go-live-browser-file-list.png)

この画面で map-1 フォルダをクリックすると...？

---

> 3D マップが表示されたゲーム画面のスクリーンショット

---

3D マップとロボットが表示されたら成功です！🎉

<br />

## よくあるエラーと対処法

もしエラーが出た場合は、以下を確認してください。

### パターン１： 画面が真っ白

#### 原因

- ファイルパスが間違っている
- JavaScript でエラーが起きている可能性

#### 対処法

1. F12 キーでコンソールを開く
2. 赤いエラーメッセージを確認
3. スペルミスがないか確認

### パターン２： Go Live が動かない

#### 原因

- ポート番号の競合

#### 対処法

1. VSCode を再起動

<br />

## 本チャプター終了時点のコード

### 基盤ファイル

:::details ./ai-fetch.js

```javascript

```

:::

:::details ./ai-system-prompt.js

```javascript

```

:::

:::details ./game.js

```javascript

```

:::

:::details ./game-model.js

```javascript

```

:::

:::details ./secret.js

```javascript
/**
 * OpenAI(ChatGPT) API の Key。
 *
 * 注意： この API Key は公開してはいけません！！
 *       ローカルで起動して使用する場合は問題ないのですが、Web サイトとして公開する場合などは、API Key を必要としている処理をサーバーサイドで記述するなど、API Key は隠す必要があります。
 */
export const OPENAI_API_KEY = 'sk-ここに配布されたAPIキーを入力';
```

:::

### マップファイル

:::details ./map-1/ai-route-prompt.js

```javascript

```

:::

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
