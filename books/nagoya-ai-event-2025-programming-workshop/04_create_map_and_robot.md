---
title: "第3章：マップとロボットの作成"
---

# マップとロボットの作成

いよいよ、3D 迷路のマップとロボットを作成します。実際にコードを書いて、ゲームを動かしてみましょう。

<br />

## HTML ファイルを作成する

まず、ゲームの土台となる HTML ファイルを作成します。`map-1/index.html` を開いて、以下のコードをコピーして貼り付けましょう。

### 📝 コードを記述する

:::details 【コピペ用】map-1/index.html の完全なコード
```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI ロボット宝探しゲーム - マップ 1</title>
    <link rel="stylesheet" href="./style.css">
</head>
<body>
    <div id="container">
        <h1>🤖 AI ロボット宝探しゲーム - マップ 1 🗺️</h1>
        <div id="controls">
            <button id="startButton">スタート</button>
            <button id="resetButton">リセット</button>
        </div>
        <div id="canvas-container"></div>
        <div id="status"></div>
    </div>

    <!-- Three.js （3D ライブラリ） -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    
    <!-- ゲームのスクリプト -->
    <script type="module" src="../game.js"></script>
</body>
</html>
```
:::

---
> HTML ファイルの各部分 （head、body、script） を説明する図解
---

### HTML の説明

-   **head 部分**。ページの設定 （タイトル、文字コードなど）
-   **body 部分**。実際に表示される内容
-   **script 部分**。JavaScript ファイルの読み込み
    -   Three.js。3D グラフィックスライブラリ
    -   game.js。ゲームのメインプログラム

:::message
注意。style.css のリンクは `./style.css` （同じフォルダ内） になっています。
:::

<br />

## マップデータを作成する

次に、迷路のマップデータを作成します。`map-1/map.js` を開いて編集しましょう。

### 📝 コードを記述する

:::details 【コピペ用】map-1/map.js の完全なコード
```javascript
export const mapConfig = {
    // マップの配置（上から見た図）
    layout: [
        ['#', '#', '#', '#', '#', '#'],
        ['#', 'S', '.', '.', '.', '#'],
        ['#', '.', '#', 'T', '.', '#'],
        ['#', '.', '.', '.', '.', '#'],
        ['#', '#', '#', '#', 'G', '#'],
        ['#', '#', '#', '#', '#', '#'],
    ],
    
    // 各記号の意味と色
    cell: {
        '#': { type: 'wall', color: '#808080' },      // 壁 （グレー）
        'S': { type: 'start', color: '#00ff00' },     // スタート （緑）
        'G': { type: 'end', color: '#ff0000' },       // ゴール （赤）
        'T': { type: 'trap', color: '#000000' },      // トラップ （黒）
        '.': { type: 'normal', color: '#e0e0e0' },    // 通路 （薄いグレー）
    },
};
```
:::

---
> マップの配列がどのように 3D 迷路に変換されるかを示すビフォーアフター図
---

### マップの読み方

配列の見方を理解しましょう。

```
行番号
0: ['#', '#', '#', '#', '#', '#']    ← 上の壁
1: ['#', 'S', '.', '.', '.', '#']    ← スタート地点のある行
2: ['#', '.', '#', 'T', '.', '#']    ← トラップのある行
3: ['#', '.', '.', '.', '.', '#']    ← 通路
4: ['#', '#', '#', '#', 'G', '#']    ← ゴールのある行
5: ['#', '#', '#', '#', '#', '#']    ← 下の壁
    列: 0    1    2    3    4    5
```

---
> 2D 配列の座標システムを説明する図 （行と列の関係）
---

<br />

## AI への指示ファイルを準備する

最初は空の指示を用意しておきます。`map-1/ai-route-prompt.js` を開いて編集しましょう。

### 📝 コードを記述する

:::details 【コピペ用】map-1/ai-route-prompt.js の初期コード
```javascript
export const routePrompt = `
ここに AI ロボットへの指示を書きます
`;
```
:::

まだ指示は書かなくて大丈夫です。次の章で詳しく説明します。

<br />

## ゲームを確認する

ここまでできたら、一度ゲームを確認してみましょう。

1.  すべてのファイルを保存 （Ctrl+S または Cmd+S）
2.  VSCode で「Go Live」ボタンをクリック
3.  ブラウザで「map-1」フォルダを選択

---
> Go Live ボタンの位置を示す VSCode のスクリーンショット
---

:::message
「Go Live」ボタンが見つからない場合は、拡張機能「Live Server」がインストールされていない可能性があります。スタッフに声をかけてください。
:::

---
> ブラウザでファイル一覧が表示されている画面のスクリーンショット
---

map-1 フォルダをクリックすると...

---
> 3D 迷路が表示されたゲーム画面のスクリーンショット
---

3D 迷路が表示されたら成功です。🎉

### 画面の説明

---
> ゲーム画面の各要素 （ロボット、スタート、ゴール、トラップ、通路） を矢印で示した注釈付きスクリーンショット
---

-   **青い箱**。これがあなたのロボットです
-   **緑のマス**。スタート地点
-   **赤のマス**。ゴール地点 （宝物がある場所）
-   **黒いマス**。トラップ （通ると失敗）
-   **グレーのマス**。壁 （通れない）
-   **薄いグレーのマス**。通路 （通れる）

<br />

## map-2 も同じように準備する

map-1 と同じ内容で、map-2 のファイルも準備しておきましょう。

### 1. map-2/index.html を作成

:::details 【コピペ用】map-2/index.html の完全なコード
```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI ロボット宝探しゲーム - マップ 2</title>
    <link rel="stylesheet" href="./style.css">
</head>
<body>
    <div id="container">
        <h1>🤖 AI ロボット宝探しゲーム - マップ 2 🗺️</h1>
        <div id="controls">
            <button id="startButton">スタート</button>
            <button id="resetButton">リセット</button>
        </div>
        <div id="canvas-container"></div>
        <div id="status"></div>
    </div>

    <!-- Three.js （3D ライブラリ） -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    
    <!-- ゲームのスクリプト -->
    <script type="module" src="../game.js"></script>
</body>
</html>
```
:::

### 2. map-2/map.js を作成

map-1 と同じマップデータをコピーします （後で変更します） 。

:::details 【コピペ用】map-2/map.js の初期コード
```javascript
export const mapConfig = {
    // マップの配置（上から見た図）
    layout: [
        ['#', '#', '#', '#', '#', '#'],
        ['#', 'S', '.', '.', '.', '#'],
        ['#', '.', '#', 'T', '.', '#'],
        ['#', '.', '.', '.', '.', '#'],
        ['#', '#', '#', '#', 'G', '#'],
        ['#', '#', '#', '#', '#', '#'],
    ],
    
    // 各記号の意味と色
    cell: {
        '#': { type: 'wall', color: '#808080' },      // 壁 （グレー）
        'S': { type: 'start', color: '#00ff00' },     // スタート （緑）
        'G': { type: 'end', color: '#ff0000' },       // ゴール （赤）
        'T': { type: 'trap', color: '#000000' },      // トラップ （黒）
        '.': { type: 'normal', color: '#e0e0e0' },    // 通路 （薄いグレー）
    },
};
```
:::

### 3. map-2/ai-route-prompt.js を作成

:::details 【コピペ用】map-2/ai-route-prompt.js の初期コード
```javascript
export const routePrompt = `
ここに AI ロボットへの指示を書きます
`;
```
:::

<br />

## よくあるエラーと対処法

もしエラーが出た場合は、以下を確認してください。

### エラー 1。画面が真っ白

---
> 真っ白な画面のスクリーンショット
---

**原因**。ファイルパスが間違っている
**対処法**。
- style.css のパスが `./style.css` になっているか確認
- game.js のパスが `../game.js` になっているか確認

### エラー 2。3D 表示されない

---
> エラーコンソールのスクリーンショット
---

**原因**。JavaScript エラー
**対処法**。
1. F12 キーでコンソールを開く
2. 赤いエラーメッセージを確認
3. スペルミスがないか確認

### エラー 3。Go Live が動かない

**原因**。ポート番号の競合
**対処法**。VSCode を再起動

---

<br />

## 本チャプター終了時点のゲーム状態

このチャプターを終えると、以下のような状態になっているはずです。

-   3D 迷路が表示される
-   スタートボタンとリセットボタンがある
-   ロボット （青い箱） がスタート地点にいる
-   まだロボットは動かない （指示がないため）

次の章では、API キーを設定して、AI と通信できるようにします。
