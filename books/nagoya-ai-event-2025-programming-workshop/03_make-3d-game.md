---
title: '第2章： ゲーム基盤作成'
---

## ゲーム基盤作成

それでは、さっそくゲーム基盤を作成していきましょう！

<br />

## ファイルを確認する

VSCode の左側のファイル一覧を見てください。以下のファイルがあることを確認しましょう。

<br />

## 各ファイルの役割

それぞれのファイルがどんな仕事をしているか、簡単に説明します。

### ai-fetch.js

Web ブラウザから ChatGPT を呼び出すための処理を記述したファイルです。

:::details ファイルの中身（コピー&ペースト）

```javascript
import { movementKeys } from './game.js'; // game.js から movementKeys をインポート

// movementKeys を使用して正規表現を動的に生成
const allowedCharsList = Object.values(movementKeys);
const allowedCharsPattern = allowedCharsList.join('');
const filterRegex = new RegExp(`[^${allowedCharsPattern}]`, 'g');
const extractRegex = new RegExp(`[${allowedCharsPattern}]+`, 'g');

/**
 * OpenAI API を使用して経路パスを取得する。
 *
 * @param {object} params                 AIパス取得のためのパラメータ
 * @param {string} params.apiKey          OpenAI API キー
 * @param {string} params.systemPrompt    AIへのシステムプロンプト
 * @param {string} params.routePrompt     ユーザー提供の経路プロンプト（指示）
 * @returns {Promise<Array<string>|null>} 移動指示の配列（例: ['l', 'r', 'u', 'd']）
 */
export async function fetchRoutePathWithOpenAI({
    apiKey,
    systemPrompt,
    routePrompt,
}) {
    // OpenAI API のエンドポイント
    const openAIEndpoint = 'https://api.openai.com/v1/chat/completions';

    // OpenAI API のためのメッセージペイロードを構築する
    const messages = [
        {
            role: 'system',
            content: systemPrompt,
        },
        {
            role: 'user',
            content: routePrompt,
        },
    ];

    // OpenAI API にリクエストを送信する
    try {
        console.debug('OpenAI API にリクエストを送信中...');

        const response = await fetch(openAIEndpoint, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                Authorization: `Bearer ${apiKey}`,
            },
            body: JSON.stringify({
                model: 'gpt-3.5-turbo',
                messages: messages,
                temperature: 0.0,
            }),
        });

        // レスポンスが正常か確認する
        if (!response.ok) {
            const errorData = await response.json().catch(() => null);
            const errorMessage = `OpenAI API エラー: ${errorData?.message}`;
            console.error(errorMessage);
            alert(errorMessage);
            return null;
        }

        // レスポンスを JSON として解析する
        const data = await response.json();

        // API からの応答に choices が含まれているか、またその配列が空でないかを確認する
        if (!data.choices || data.choices.length === 0) {
            const errorMessage =
                'OpenAI APIの応答に選択肢 (choices) が含まれていませんでした。';
            console.error(errorMessage);
            alert(errorMessage);
            return null;
        }

        // AI からの応答メッセージ（移動指示）を解析する
        const messageContent = data.choices[0].message.content;

        // 応答から許可された文字 (allowedCharsPattern で定義されたもの) のみを抽出する
        const filteredMessageContent = messageContent.replace(filterRegex, '');

        // フィルタリングされた文字列から連続した移動指示 (extractRegex で定義されたもの) を抽出する
        // これにより、"lrt" のようなシーケンスが一つの要素として抽出される
        const moves = filteredMessageContent.match(extractRegex);

        // 抽出された移動指示が有効か確認する
        // AI からの応答は movementKeys で定義された文字のみを含むことを期待している
        if (!moves || moves.length === 0 || moves[0].length === 0) {
            console.warn('AIの応答から有効な移動指示を抽出できませんでした。');
            alert('AIの応答から有効な移動指示を抽出できませんでした。');
            return null;
        }

        // 抽出された移動指示の配列を返す
        return moves;
    } catch (error) {
        // ネットワークエラーやその他の予期せぬエラーをキャッチします。
        console.error(
            'OpenAIからの経路取得中に予期せぬエラーが発生しました:',
            error
        );
        alert(
            'OpenAIからの経路取得中に予期せぬエラーが発生しました。詳細はコンソールを確認してください。'
        );
        return null;
    }
}
```

:::

### ai-system-prompt.js

AI ロボットの基本となる「性格」や「ルール」を決めるファイルです。\
このファイルを変更することで、AI ロボットの性格を変えることができます。

:::details ファイルの中身（コピー&ペースト）

```javascript
import { movementKeys } from './game.js';

/**
 * AI のためのシステムプロンプトを動的に生成する。
 * マップ設定のセル定義をプロンプトに含めることで、AI が各セルの意味を理解できるようにする。
 *
 * @param   {import("./game").MapConfig} mapConfig マップ設定のセル定義オブジェクト
 * @returns {string}                               システムプロンプト文字列
 */
export function generateSystemPrompt(mapConfig) {
    let prompt = '';

    // 1. あなたの役割
    prompt +=
        `\n【あなたの役割】\n` +
        `あなたはグリッドベースのマップをナビゲートするロボットです。\n` +
        `あなたの目的は、ユーザーからの指示に従って、移動の指示を出力することです。\n` +
        `あなたの初期位置は、'start' の位置です。\n` +
        `\n`;

    // 2. 移動の指示
    prompt +=
        `\n【移動の指示】\n` +
        `移動の指示は以下の文字のみを使用して、一連の動作として応答してください。（例: "rlu"）\n` +
        `- ${movementKeys.LEFT}: 左へ移動\n` +
        `- ${movementKeys.RIGHT}: 右へ移動\n` +
        `- ${movementKeys.UP}: 上へ移動\n` +
        `- ${movementKeys.DOWN}: 下へ移動\n` +
        `\n`;

    // 3. 一般的なセルタイプの説明
    prompt +=
        `\n【一般的なセルタイプの説明】\n` +
        `- タイプ 'start' は、プレイヤーの開始地点を示します。位置は認識できます。\n` +
        `- タイプ 'end' は、プレイヤーの目標地点を示します。位置は認識できません。\n` +
        `- タイプ 'trap' は、避けるべきトラップセルを示します。位置は認識できません。\n` +
        `- タイプ 'object' は、マップ上の特定のオブジェクトや目印を示します。位置は認識できます。\n` +
        `- タイプ 'normal' は、通行可能な通常の経路セルを示します。位置は認識できます。\n` +
        `- タイプ 'custom' は、そのステージ特有のセルを示します。位置を認識できるかどうかも異なります。\n` +
        `\n`;

    // 4. 固有マップの説明
    prompt +=
        `\n【この固有マップのレイアウトの説明】\n` +
        `このマップのレイアウトは、以下の通りです。\n` +
        `${mapConfig.layout
            .map((row, i) => `${i + 1}行目: ${row.toString()}`)
            .join('\n')}\n` +
        `\n`;

    // 5. 固有マップの記号とタイプ
    prompt += `\n【この固有マップの記号とタイプ定義】\n`;
    for (const [char, config] of Object.entries(mapConfig.cell)) {
        if (!config || !config.type) continue;
        const type = config.type;
        const description = config.description
            ? ` (説明: ${config.description})`
            : '';
        prompt += `- マップ記号「${char}」はタイプ「${type}」に対応します。${description}\n`;
    }
    prompt += `\n`;

    // 6. 追加の指示と制約
    prompt +=
        `\n【追加の指示と制約】\n` +
        `ユーザーからの経路指示と、上記で提供されたセル情報を基に最適な経路を判断してください。\n` +
        `必ずゴール（'end'タイプのセル）に到達する経路を生成してください。\n` +
        `トラップ（'trap'タイプのセル）は必ず避けてください。\n` +
        `応答は、指示された移動文字の連続のみとしてください。他のテキストや説明は一切含めないでください。`;

    return prompt;
}
```

:::

### game.js

3D 迷路を画面に表示するファイルです。\
Three.js というライブラリを使っており、Web で 3D グラフィックスを表示することができます！

:::details ファイルの中身（コピー&ペースト）

```javascript
// ###########
// ## 型定義 ##
// ###########

/**
 * @typedef {object} ThreeJS
 */

/**
 * @typedef {object} MapConfig
 *
 * @property {string[][]}                    layout マップレイアウト
 * @property {Record<string, MapCellConfig>} cell   セル定義
 */

/**
 * @typedef {object} MapCellConfig
 *
 * @property {MapCellConfigType} type          セルタイプ
 * @property {string}            [image]       セル画像パス
 * @property {string}            [color]       セル色
 * @property {string}            [description] セルの説明
 */

/**
 * @typedef {"start"|"end"|"trap"|"object"|"normal"|"custom"} MapCellConfigType
 */

// #############
// ## 共通処理 ##
// #############

/**
 * Three.js ライブラリを動的にインポートする。
 *
 * @returns {Promise<object>} Three.js インスタンス
 * @throws  {Error}           Three.js のロードに失敗した場合エラーをスロー
 */
async function loadThreeJS() {
    try {
        const threeJS = await import('https://esm.sh/three@0.177.0');
        return threeJS;

        // ↓ エラーが発生した場合はエラーをスロー
    } catch (error) {
        console.error('Three.js の動的インポートに失敗しました:', error);
        alert('Three.js の読み込みに失敗しました。ゲームを開始できません。');
        throw error;
    }
}

// ######################
// ## ゲームのセットアップ ##
// ######################

/**
 * @typedef {ReturnType<typeof renderMap>} MapContext
 */

/**
 * マップをレンダリングし、基本的なシーンを設定する。
 *
 * @param   {object}      params           レンダリングパラメータ
 * @param   {ThreeJS}     params.threeJS   Three.js インスタンス
 * @param   {HTMLElement} params.element   レンダリング対象の要素
 * @param   {MapConfig}   params.mapConfig マップ設定オブジェクト
 * @returns                                マップコンテキスト
 * @throws  {Error}                        スタート位置またはエンド位置が見つからない場合にエラーをスロー
 */
function renderMap({ threeJS, element, mapConfig }) {
    // スタート位置とエンド位置を検索
    const layout = mapConfig.layout;

    /**
     * @type {{
     *  start: { x: number; y: number };
     *  end: { x: number; y: number };
     * }}
     */
    const position = {};

    // スタート位置とエンド位置を検索
    for (let j = 0; j < layout.length; j++) {
        for (let i = 0; i < layout[j].length; i++) {
            const cellAlias = layout[j][i];
            const cellDef = mapConfig.cell[cellAlias];

            // セル定義がない場合はスキップ
            if (!cellDef) {
                continue;
            }

            // セルタイプに応じて位置を設定
            if (cellDef.type === 'start') {
                position.start = { x: i, y: j };
            }
            if (cellDef.type === 'end') {
                position.end = { x: i, y: j };
            }
        }
    }

    // スタート位置またはエンド位置が見つからない場合はエラーをスロー
    if (!position.start) {
        throw new Error('マップに開始位置が見つかりません。');
    }
    if (!position.end) {
        throw new Error('マップに終了位置が見つかりません。');
    }

    // シーンを作成
    const scene = new threeJS.Scene();

    // カメラを作成
    const aspectRatio = element.offsetWidth / element.offsetHeight;
    const camera = new threeJS.PerspectiveCamera(45, aspectRatio, 0.1, 1000);

    // マップのレイアウトを取得
    const mapDisplayWidth = layout[0].length;
    const mapDisplayHeight = layout.length;

    // カメラの位置を設定
    camera.position.set(
        mapDisplayWidth / 2,
        mapDisplayWidth * 1.5,
        mapDisplayHeight / 2
    );

    // カメラの向きを設定
    camera.lookAt(
        new threeJS.Vector3(mapDisplayWidth / 2, 0, mapDisplayHeight / 2)
    );

    // レンダラーを作成
    const renderer = new threeJS.WebGLRenderer({ antialias: true });
    renderer.setSize(element.offsetWidth, element.offsetHeight);
    element.appendChild(renderer.domElement);

    // 環境光を作成
    const ambientLight = new threeJS.AmbientLight(0xffffff, 0.6); // 環境光
    scene.add(ambientLight);

    // 平行光源を作成
    const directionalLight = new threeJS.DirectionalLight(0xffffff, 0.8); // 平行光源
    directionalLight.position.set(0, 10, 0);
    scene.add(directionalLight);

    // マップタイルを作成
    for (let j = 0; j < mapDisplayHeight; j++) {
        for (let i = 0; i < mapDisplayWidth; i++) {
            const cellAlias = layout[j][i];
            const cellDef = mapConfig.cell[cellAlias];
            const geometry = new threeJS.PlaneGeometry(1, 1); // 1x1 サイズの平面ジオメトリ
            let colorValue = 0xcccccc; // 未定義タイル用のデフォルト色

            // セル定義がない場合はスキップ
            if (!cellDef) {
                continue;
            }

            // セルの色を設定
            if (cellDef.color) {
                const parsedColor = parseInt(
                    cellDef.color.replace('#', ''),
                    16
                );
                colorValue = isNaN(parsedColor) ? colorValue : parsedColor;
            }

            // セルのマテリアルを作成
            const tile = new threeJS.Mesh(
                geometry,
                new threeJS.MeshStandardMaterial({
                    color: colorValue,
                    side: threeJS.DoubleSide,
                })
            );

            // セルの回転と位置を設定
            tile.rotation.x = -Math.PI / 2; // X軸を中心に-90度回転して床にする
            tile.position.set(i, 0, j); // XZ平面に配置

            // セルをシーンに追加
            scene.add(tile);
        }
    }

    // 初期レンダリング
    renderer.render(scene, camera);

    // ウィンドウリサイズ時の処理
    window.addEventListener('resize', () => {
        if (camera && renderer && element) {
            camera.aspect = element.offsetWidth / element.offsetHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(element.offsetWidth, element.offsetHeight);
            renderer.render(scene, camera); // リサイズ後にもレンダリング
        }
    });

    return {
        scene,
        camera,
        renderer,
        position,
    };
}

/**
 * プレイヤーキャラクターを作成し、シーンに追加する。
 *
 * @param   {object}          params            プレイヤー作成パラメータ
 * @param   {ThreeJS}         params.threeJS    Three.js インスタンス
 * @param   {MapContext}      params.mapContext マップコンテキスト
 * @returns                                     プレイヤー
 */
function createPlayer({ threeJS, mapContext }) {
    // プレイヤーを作成
    const player = new threeJS.Mesh(
        // プレイヤーの形状
        new threeJS.BoxGeometry(0.8, 0.8, 0.8),
        // プレイヤーの色
        new threeJS.MeshStandardMaterial({
            color: 0x6b7adb,
        })
    );

    // プレイヤーの位置を設定
    player.position.set(
        mapContext.position.start.x,
        0.5,
        mapContext.position.start.y
    );

    // プレイヤーをシーンに追加
    mapContext.scene.add(player);

    // プレイヤー表示のための初期レンダリング
    mapContext.renderer.render(mapContext.scene, mapContext.camera);

    return player;
}

/**
 * @typedef {Awaited<ReturnType<typeof setupGame>>} GameContext
 */

/**
 * ゲームの主要なセットアップ処理を行う。\
 * Three.js のロード、マップレンダリング、プレイヤー作成を順番に実行する。
 *
 * @param   {object}               params           セットアップパラメータ
 * @param   {HTMLElement}          params.element   レンダリング対象の要素
 * @param   {object}               params.mapConfig マップ設定オブジェクト
 * @returns                                         コンテキストオブジェクト
 * @throws  {Error}                                 Three.js のロードまたはゲームの初期化に失敗した場合エラーをスロー
 */
export async function setupGame({ element, mapConfig }) {
    const threeJS = await loadThreeJS();

    // マップをレンダリングし、シーン、カメラ、レンダラーを取得
    const mapContext = renderMap({
        threeJS,
        element,
        mapConfig,
    });

    // プレイヤーを作成
    const player = createPlayer({
        threeJS,
        mapContext,
    });

    // ゲームコンテキストを返す
    return {
        threeJS,
        player,
        ...mapContext,
    };
}

// #####################
// ## 移動関連のロジック ##
// #####################

/**
 * 移動コマンド文字の定義。
 */
export const movementKeys = {
    LEFT: 'l',
    RIGHT: 'r',
    UP: 'u',
    DOWN: 'd',
};

/**
 * プレイヤーを指定された方向に1ステップ移動させ、アニメーション表示する。
 *
 * @param {object}      params           移動パラメータ
 * @param {GameContext} params.context   ゲームコンテキストオブジェクト
 * @param {string}      params.direction 移動方向
 */
export function movePlayer({ context, direction }) {
    const { threeJS, scene, camera, renderer, player } = context;

    const stepVec = { x: 0, z: 0 }; // 移動ベクトル
    switch (direction) {
        case movementKeys.RIGHT:
            stepVec.x = 1;
            break; // 右 (Right)

        case movementKeys.LEFT:
            stepVec.x = -1;
            break; // 左 (Left)

        case movementKeys.UP:
            stepVec.z = -1;
            break; // 上 (Up)

        case movementKeys.DOWN:
            stepVec.z = 1;
            break; // 下 (Down)

        default:
            return; // 未知の方向の場合は何もしない
    }

    // 現在位置と目標位置を計算
    const startPos = player.position.clone();
    const endPos = player.position
        .clone()
        .add(new threeJS.Vector3(stepVec.x, 0, stepVec.z));

    // アニメーションの設定
    const duration = 240;
    let startTime = null;

    // アニメーションループ
    function animate(currentTime) {
        if (startTime === null) startTime = currentTime;
        const elapsedTime = currentTime - startTime;
        const t = Math.min(elapsedTime / duration, 1); // 進行度 (0から1)
        player.position.lerpVectors(startPos, endPos, t); // 線形補間で中間位置を計算
        renderer.render(scene, camera); // シーンをレンダリング
        if (t < 1) requestAnimationFrame(animate);
    }

    // アニメーション開始
    requestAnimationFrame(animate);
}

/**
 * プレイヤー移動処理を開始するボタンのセットアップとイベントリスナーの追加。
 *
 * @param {object}            params             設定パラメータ
 * @param {HTMLButtonElement} params.element     処理を開始するボタン要素
 * @param {MapConfig}         params.mapConfig   マップ設定オブジェクト
 * @param {GameContext}       params.gameContext ゲームコンテキストオブジェクト
 * @param {function}          params.pathFetcher AIから経路を取得する関数
 */
export function setupPlayerMoverButton({
    element,
    mapConfig,
    gameContext,
    pathFetcher,
}) {
    const { scene, camera, renderer, player } = gameContext;

    element.addEventListener('click', async () => {
        console.log('AIによる経路指示に基づくプレイヤー移動を開始します...');
        element.disabled = true; // 処理中にボタンを無効化

        // スタート位置にプレイヤーをリセット (オプション)
        let startX, startY;
        let startPositionFound = false;
        for (let j = 0; j < mapConfig.layout.length; j++) {
            const rowIndex = mapConfig.layout[j].findIndex(
                (cellAlias) => mapConfig.cell[cellAlias]?.type === 'start'
            );
            if (rowIndex !== -1) {
                startX = rowIndex;
                startY = j;
                startPositionFound = true;
                break;
            }
        }

        // スタート地点が見つからない場合はエラーを表示して終了
        if (!startPositionFound) {
            alert(
                'スタート地点がマップ設定に見つかりませんでした。プレイヤーはリセットされません。'
            );
            return;
        }

        // プレイヤーをスタート地点に移動
        player.position.set(startX, 0.5, startY);
        renderer.render(scene, camera);

        try {
            // 経路を取得
            const moves = await pathFetcher();

            // 経路をコンソールに出力
            console.log('AIからの経路:', moves);

            // 経路がない場合はエラーを表示して終了
            if (!moves || moves.length === 0) {
                alert('AIから有効な経路が返されませんでした。');
                return;
            }

            const movingDelay = 240;
            const moveSequence = moves[0]; // "lru" のような文字列
            for (let i = 0; i < moveSequence.length; i++) {
                const moveChar = moveSequence[i];
                await new Promise((resolve) =>
                    setTimeout(resolve, i === 0 ? 0 : movingDelay)
                );
                movePlayer({
                    context: gameContext,
                    direction: moveChar,
                });
            }

            // ↓ エラーが発生した場合の処理
        } catch (error) {
            console.error(
                'AI経路取得またはプレイヤー移動の実行中にエラーが発生しました:',
                error
            );

            // ↓ 処理が全て完了した時の処理
        } finally {
            element.disabled = false; // 処理完了後またはエラー時にボタンを再度有効化
        }
    });
}
```

:::

### secret.js（機密情報の設定）

API キーを設定するためのファイルです。

:::details コードをコピペする

```javascript
/** OpenAI APIキー */
export const OPENAI_API_KEY = 'sk-ここに配布されたAPIキーを入力';
```

:::

後ほど、このファイルの値に API キーを設定します。

<br />

:::details 【深く知りたい人向け】使用技術について

- **Three.js**: WebGL を使った 3D グラフィックスライブラリ
- **ES6 モジュール**: JavaScript の最新の書き方
- **async/await**: 非同期処理の仕組み
- **Fetch API**: HTTP リクエストを送る仕組み

これらの技術に興味がある人は、ワークショップ後に調べてみてください。
:::

## まとめ

これらの基本ファイルが協力して、3D ゲームが動いています！

1.  **ai-fetch.js**: AI と通信
2.  **ai-system-prompt.js**: AI の設定
3.  **game.js**: 3D 迷路を表示、ゲーム全体をコントロール
4.  **secret.js**: API キーのひな形

次の章では、いよいよマップとロボットを作成していきます！
