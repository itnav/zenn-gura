---
title: '💛 マップとロボットの作成'
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

**PowerShell(Windows) の場合**

```powershell
New-Item -ItemType Directory -Name "map-1"
New-Item -ItemType File -Path "map-1/ai-route-prompt.js"
New-Item -ItemType File -Path "map-1/index.html"
New-Item -ItemType File -Path "map-1/map.js"
New-Item -ItemType File -Path "map-1/style.css"
```

:::details コマンドプロンプト(Windows) の場合

```bash
mkdir map-1
type nul > map-1/ai-route-prompt.js
type nul > map-1/index.html
type nul > map-1/map.js
type nul > map-1/style.css
```

:::

:::details シェル(Mac,Linux) の場合

```bash
mkdir map-1
touch map-1/ai-route-prompt.js
touch map-1/index.html
touch map-1/map.js
touch map-1/style.css
```

:::

コマンドを実行すると、一気に左側のファイル一覧に新しいファイルとフォルダが表示されます。これで冒険の準備は完了です！

![作成されたフォルダとファイルが表示されている VSCode のサイドバー](/images/nagoya-ai-event-2025-programming-workshop/04_map-and-robot-setup/02_created-files-and-folders.png)

:::details 【深く知りたい人向け】コマンドの解説

**PowerShell**

- `New-Item -ItemType Directory`: 新しいフォルダを作成
- `New-Item -ItemType File`: 新しいファイルを作成
- `-Path`: ファイルのパスを指定（フォルダ名/ファイル名）

<br />

**コマンドプロンプト**

- `mkdir`: 新しいフォルダを作成（make directory）
- `type nul >`: 空のファイルを作成
- `map-1/`: フォルダの中にファイルを作成

<br />

**シェル（Mac/Linux）**

- `mkdir`: 新しいフォルダを作成（make directory）
- `touch`: 空のファイルを作成
- `map-1/`: フォルダの中にファイルを作成

:::

<br />

## コードを記述する

それでは、前回の章と同じ要領でコードを記述していきましょう！

### 1. `index.html`

ゲーム画面（3D マップとロボット）をブラウザ上に表示するためのファイルです！

:::details ファイルの中身（コピー&ペースト）

```html
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
    <title>🤖 お宝探しゲーム</title>
    <link rel="stylesheet" href="./style.css" />
    <script type="module" defer>
      import {
        fetchRoutePathWithOpenAI,
        generateSystemPrompt,
      } from '../../ai.js';
      import { setupGame } from '../../game.js';
      import { OPENAI_API_KEY } from '../../secret.js';
      import { routePrompt } from './ai-route-prompt.js';
      import { mapConfig } from './map.js';

      // ゲームをセットアップ
      await setupGame({
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
    <div id="game-response-viewer"></div>
    <button id="game-trigger"></button>
  </body>
</html>
```

:::

### 2. `style.css`

ゲーム自体の体裁を整えるためのファイルです！

:::details ファイルの中身（コピー&ペースト）

```css
/**
 * ##########
 * ## Core ##
 * ##########
 */

html,
body {
  margin: 0;
}

body {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  overflow: hidden;
  height: 100svh;
  padding: 24px;
  box-sizing: border-box;
}

/**
 * ##########################
 * ## Game Response Viewer ##
 * ##########################
 */

#game-viewer {
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
  flex: 1;
  width: 100%;
}

/**
 * ##########################
 * ## Game Response Viewer ##
 * ##########################
 */

#game-response-viewer {
  display: block;
  width: 100%;
  max-width: 480px;
  margin-bottom: 24px;
  padding: 16px;
  box-sizing: border-box;
  background-color: #f3f4f6;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  overflow-y: auto;
}

#game-response-viewer-label {
  color: #6b7280;
  font-size: 12px;
  font-weight: 600;
  margin-bottom: 8px;
}

#game-response-viewer-content {
  display: flex;
  align-items: center;
  color: #1f2937;
  font-size: 24px;
  font-weight: 500;
  letter-spacing: 4px;
  transition: opacity 0.3s ease-in-out;
  opacity: 0;
}

#game-response-viewer-content.visible {
  opacity: 1;
}

#game-response-viewer-content span {
  display: inline-block;
  opacity: 0;
  animation: gameResponseViewerContentFadeInArrow 0.4s ease-in-out forwards;
}

@keyframes gameResponseViewerContentFadeInArrow {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/**
 * ##################
 * ## Game Trigger ##
 * ##################
 */

#game-trigger {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  max-width: 480px;
  min-height: 48px;
  padding: 12px 16px;
  box-sizing: border-box;
  color: #fff;
  font-size: 14px;
  font-weight: 500;
  border-radius: 6px;
  border: 1px solid rgba(62, 159, 120, 0.1);
  background-color: #3e9f78;
  cursor: pointer;
  transform: scale(1);
  transition: all 0.2s ease-in-out;
}

#game-trigger:hover:not(:disabled) {
  background-color: #358862;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transform: scale(1.02);
}

#game-trigger:active:not(:disabled) {
  background-color: #2c7150;
  transform: scale(0.98);
}

#game-trigger:disabled {
  background-color: #9ca3af;
  border-color: #d1d5db;
  cursor: not-allowed;
  transform: scale(1);
  opacity: 0.6;
}

#game-trigger .spinner {
  display: inline-block;
  width: 16px;
  height: 16px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-top-color: #fff;
  border-radius: 50%;
  animation: gameTriggerSpinner 0.8s linear infinite;
}

@keyframes gameTriggerSpinner {
  to {
    transform: rotate(360deg);
  }
}
```

:::

### 3. `map.js`

マップの設定を行うファイルです！

:::details ファイルの中身（コピー&ペースト）

```javascript
/**
 * マップの設定オブジェクトです。
 *
 * このオブジェクトのプロパティ値を変更することで、
 * マップのレイアウト、スタートやエンド、トラップやオブジェクトの位置を変更できます。
 *
 * @type {import("../game").MapConfig}
 */
export const mapConfig = {
  cell: {
    s: {
      type: 'start',
      color: '#6B7ADB',
      description: 'スタート地点',
    },
    e: {
      type: 'end',
      color: '#F59E0B',
      description: 'お宝の場所',
    },
    t: {
      type: 'trap',
      color: '#DC2626',
      description: 'トラップ',
    },
    o: {
      type: 'object',
      color: '#92400E',
      description: 'オブジェクト（土管）',
    },
    n: {
      type: 'normal',
      color: '#D4A574',
    },
  },
  layout: [
    ['s', 'n', 'n', 'n', 'n', 't'],
    ['n', 'n', 'o', 'n', 'n', 'n'],
    ['n', 'n', 'n', 'n', 'n', 'n'],
    ['n', 'n', 'n', 't', 'n', 'n'],
    ['n', 'n', 'n', 'n', 'n', 'n'],
    ['o', 'n', 'n', 'n', 'n', 'e'],
  ],
};
```

:::

### 4. `ai-route-prompt.js`

ロボットに指示を出すためのプロンプト（指示）を記述するファイルです！

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

## トラブルシューティング

何か問題が発生した場合は、以下を参考にしてください！

:::details A. ファイルやフォルダの構成が違う

**原因**: ファイルやフォルダの作成コマンドを間違えたり、保存場所を間違えたりした可能性があります。

**対処法**: VSCode の左側のエクスプローラーで、マニュアルの「本チャプター終了時点のファイル構成」と見比べて、ファイルやフォルダが正しい場所にあるか確認してください。もし違っていたら、ドラッグ＆ドロップでファイルを正しいフォルダに移動させるか、一度削除して再度正しい手順でファイルを作成し直してみてください。

:::

:::details B. 「Go Live」ボタンがない、またはクリックしても動かない

#### パターン１：Live Server 拡張機能の問題

**原因**: 拡張機能「Live Server」がインストールされていない、または無効になっている可能性があります。

**対処法**: VSCode の左側のアクティビティバーから拡張機能ビュー（四角が 4 つ並んだアイコン）を開き、「Live Server」と検索してインストールされているか確認してください。インストールされているのにボタンがない場合は、VSCode を一度再起動すると表示されることが多いです。

#### パターン２：ポートの競合

**原因**: パソコンのポートが競合している可能性があります。

**対処法**: VSCode の右下に「Port 5500 is already in use.」のようなメッセージが表示されていないか確認してください。もし表示されていたら、それが原因です。VSCode を完全に終了し、再度起動してから「Go Live」を試してみてください。これにより、別のポートでサーバーが起動するはずです。

:::

:::details C. ブラウザにファイル一覧が表示されるが、`map-1` をクリックしても何も起きない、またはエラーになる

**原因**: `map-1` フォルダの中の `index.html` が正しく読み込めていない可能性があります。

**対処法**: `map-1` フォルダ内に `index.html` という名前のファイルが存在するか確認してください。`index.html` の中身が空、またはコピー＆ペーストが不完全でないか確認してください。

:::

:::details D. 画面が真っ白で何も表示されない

#### パターン１：JavaScript のエラー

**原因**: JavaScript でエラーが発生している可能性が高いです。

**対処法**: まず、ブラウザの「開発者ツール」を開いて「コンソール」タブを確認しましょう！Windows/Mac ともに `F12` キーを押すか、画面上で右クリックして「検証」または「開発者ツール」を選択し、「コンソール」タブを開いてください。赤いエラーメッセージが表示されていないか確認します。特に `Failed to load module script:` や `Uncaught SyntaxError` などのエラーが出ていないか見てみましょう。他の JavaScript ファイル（`game.js`など）のコピー＆ペーストが間違っている可能性があるので、再度コピーし直してみてください。

#### パターン２：ブラウザのキャッシュ

**原因**: ブラウザのキャッシュが古いファイルを保持している可能性があります。

**対処法**: `Ctrl + F5`（Windows）または `Cmd + Shift + R`（Mac）で「ハードリフレッシュ」を試してみてください。これは、ブラウザに新しいファイルを強制的に読み込ませる方法です。または、ブラウザの開発者ツールを開いた状態で、更新ボタンを右クリックして「キャッシュの消去とハード再読み込み」を選択する方法も有効です。

:::
