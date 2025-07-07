---
title: 'ゲーム基盤作成'
---

## ゲーム基盤作成

それでは、さっそくゲーム基盤を作成していきましょう！

<br />

## ファイルを確認する

まずは VSCode の左側のファイル一覧を見てください！\
以前の章で作成したファイルがあることを確認しましょう 🔥

![VSCodeのファイル一覧](/images/nagoya-ai-event-2025-programming-workshop/03_game-base-setup/01_vscode-file-list.png)

<br />

## 各ファイルの役割

次に、各ファイルがどんな働きをするのか、簡単に見ていきましょう。
すべてのコードを理解しようとしなくても大丈夫です。「全体として何をやっているか」をつかむことを意識して、コードをコピー＆ペーストで動かしてみてください！

### 1. `ai-fetch.js`

Web ブラウザから ChatGPT を呼び出すための処理を記述したファイルです。

:::details ファイルの中身（コピー&ペースト）

```javascript
import { movementKey } from './game.js'; // game.js から movementKey をインポート

// movementKey を使用して正規表現を動的に生成
const allowedCharsList = Object.values(movementKey);
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
        // AI からの応答は movementKey で定義された文字のみを含むことを期待している
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

### 2. `ai-system-prompt.js`

AI ロボットの基本となる「性格」や「ルール」を決めるファイルです。

:::details ファイルの中身（コピー&ペースト）

```javascript
import { movementKey } from './game.js';

/**
 * AI のためのシステムプロンプトを動的に生成する。
 * マップ設定のセル定義をプロンプトに含めることで、AI が各セルの意味を理解できるようにする。
 *
 * @param   {import("./game").MapConfig} mapConfig マップ設定のセル定義オブジェクト
 * @returns {string}                               システムプロンプト文字列
 */
export function generateSystemPrompt(mapConfig) {
    const result = [
        '# システムプロンプト\n',
        generateRoleSection(),
        generateCoordinateSection(),
        generateMovementInstructionSection(),
        generateCellTypeSection(),
        generateMapSymbolSection(mapConfig),
        generateMapLayoutSection(mapConfig),
        generateUserRoleSection(),
        generateLimitedInformationSection(),
        generateNumericalUnderstandingSection(),
        generateStepByStepSection(),
        generateImplicitInformationSection(),
        generateInterpretationPrinciplesSection(),
        generateExecutionMindsetSection(),
        generateOutputFormatSection(),
    ]
        .filter(Boolean)
        .join('\n\n');

    return result;
}

/**
 * あなたの役割セクションを生成。
 *
 * @returns {string}
 */
function generateRoleSection() {
    return (
        `## あなたの役割\n` +
        `あなたはグリッドベースのマップをナビゲートするロボットです。\n` +
        `あなたの目的は、ユーザーからの指示に従って、移動の指示を出力することです。\n` +
        `あなたの初期位置は、'start' の位置です。\n`
    );
}

/**
 * 座標系と方向の理解セクションを生成。
 *
 * @returns {string}
 */
function generateCoordinateSection() {
    return (
        `## 座標系と方向の理解\n` +
        `マップは左上が原点(0,0)の座標系\n` +
        `- X軸：左から右へ（0 → 増加）\n` +
        `- Y軸：上から下へ（0 → 増加）\n` +
        `\n` +
        '### 方向の理解\n' +
        `- 上 (UP) [${movementKey.UP}]： Y座標を減らす（現在位置から上の行へ）\n` +
        `- 下 (DOWN) [${movementKey.DOWN}]： Y座標を増やす（現在位置から下の行へ）\n` +
        `- 左 (LEFT) [${movementKey.LEFT}]： X座標を減らす（現在位置から左の列へ）\n` +
        `- 右 (RIGHT) [${movementKey.RIGHT}]： X座標を増やす（現在位置から右の列へ）\n`
    );
}

/**
 * 移動の指示セクションを生成。
 *
 * @returns {string}
 */
function generateMovementInstructionSection() {
    return (
        `## 移動の指示\n` +
        `移動の指示は以下の4つの文字だけを使用してください。他の文字は一切使用しないでください。\n` +
        //
        `- ${movementKey.UP} : 上へ移動（Y座標 -1）\n` +
        `- ${movementKey.DOWN} : 下へ移動（Y座標 +1）\n` +
        `- ${movementKey.LEFT} : 左へ移動（X座標 -1）\n` +
        `- ${movementKey.RIGHT} : 右へ移動（X座標 +1）\n` +
        //
        `これらの文字を連続して出力してください。スペースや他の文字を挟まないでください。\n` +
        `例: "${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.DOWN}${movementKey.LEFT}"\n`
    );
}

/**
 * セルタイプの説明セクションを生成。
 *
 * @returns {string}
 */
function generateCellTypeSection() {
    return (
        `## セルタイプの説明\n` +
        `- タイプ 'start' は、プレイヤーの開始地点を示します。位置は認識できます。\n` +
        `- タイプ 'end' は、プレイヤーの目標地点を示します。位置は認識できません。\n` +
        `- タイプ 'trap' は、避けるべきトラップセルを示します。位置は認識できません。\n` +
        `- タイプ 'object' は、マップ上の特定のオブジェクトや目印を示します。位置は認識できます。\n` +
        `- タイプ 'normal' は、通行可能な通常の経路セルを示します。位置は認識できます。\n` +
        `- タイプ 'custom' は、そのステージ特有のセルを示します。位置を認識できるかどうかも異なります。\n`
    );
}

/**
 * マップレイアウトセクションを生成。
 *
 * @param   {import("./game").MapConfig} mapConfig マップ設定のセル定義オブジェクト
 * @returns {string}
 */
function generateMapLayoutSection(mapConfig) {
    // マップより normal タイプのセルを探す
    const normalCell = Object.entries(mapConfig.cell).find(
        ([_, cell]) => cell.type === 'normal'
    );

    // マップより normal タイプのセル
    const normalCellKey = normalCell?.[0] || '?';

    // ロボットが認識できないセルをマスク
    const maskedLayout = mapConfig.layout.map((row) =>
        row.map((cellAlias) => {
            const cellType = mapConfig.cell[cellAlias]?.type;
            return cellType === 'trap' || cellType === 'end'
                ? normalCellKey
                : cellAlias;
        })
    );

    console.log('maskedLayout');
    console.log(maskedLayout);

    return (
        `## この固有マップのレイアウトの説明\n` +
        `このマップのレイアウトは、Y座標、X座標の順で以下の通りです。\n` +
        `\`\`\`\n` +
        `${maskedLayout
            .map((row, i) => `Y=${i}(${i + 1}行目): [${row.join(',')}]`)
            .join('\n')}\n` +
        `\`\`\`\n` +
        '\n' +
        `### 座標の読み方\n` +
        `- 左上が原点(0,0)\n` +
        `- 横方向（右へ）がX座標の増加\n` +
        `- 縦方向（下へ）がY座標の増加\n`
    );
}

/**
 * マップの記号定義セクションを生成。
 *
 * @param   {import("./game").MapConfig} mapConfig
 * @returns {string}
 */
function generateMapSymbolSection(mapConfig) {
    let section = `## この固有マップの記号とタイプ定義\n`;
    for (const [char, config] of Object.entries(mapConfig.cell)) {
        if (!config || !config.type) continue;
        const overview = `- マップ記号「${char}」はタイプ「${config.type}」に対応します。`;
        const visibility =
            config.type === 'trap' || config.type === 'end'
                ? '（あなたはこのセルの位置を認識できません）'
                : '（認識可能）';
        const description = config.description
            ? ` (説明: ${config.description})`
            : '';
        section += `${overview}${visibility}${description}\n`;
    }
    return section;
}

/**
 * ユーザーの役割理解セクションを生成。
 *
 * @returns {string}
 */
function generateUserRoleSection() {
    return (
        `## ユーザーの役割の理解\n` +
        `重要：ユーザーはマップ全体（トラップやゴールの位置を含む）を見ることができます。\n` +
        `ユーザーの指示は、あなたには見えない危険を回避し、ゴールへ導くための信頼できるガイドです。\n` +
        `指示が一見遠回りに見えても、それはトラップを避けるためかもしれません。\n` +
        `ユーザーの指示に忠実に従ってください。\n` +
        `指示された移動以外の追加の移動は行わないでください。\n`
    );
}

/**
 * 限定的な情報での指示解釈セクションを生成。
 *
 * @returns {string}
 */
function generateLimitedInformationSection() {
    return (
        `## 限定的な情報での指示解釈\n` +
        `あなたが認識できる情報（オブジェクトの位置など）を手がかりに指示を解釈してください：\n` +
        `- 「下のオブジェクトまで」→ 現在位置から下方向にある、認識可能な最も近いオブジェクトを特定\n` +
        `- 「右に1マス」→ その位置から正確に1マス右へ\n` +
        `注意： 指示の順序には意味があります。直線的でない経路は、見えない障害物を避けている可能性があります。\n`
    );
}

/**
 * 数値理解セクションを生成。
 *
 * @returns {string}
 */
function generateNumericalUnderstandingSection() {
    return (
        `## 数値の理解と移動の実行\n` +
        `「Nマス進む」という指示を受けた場合：\n` +
        `- 「4マス進む」→ その方向に4回移動（例：下なら\"${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}\"）\n` +
        `- 「1マス進む」→ その方向に1回移動（例：右なら\"${movementKey.RIGHT}\"）\n` +
        `- 数値は必ず移動回数として解釈してください\n` +
        `\n` +
        `具体例：\n` +
        `- 「下に4マス」→ \`${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}\`\n` +
        `- 「右に1マス」→ \`${movementKey.RIGHT}\`\n` +
        `- 「右に4マス」→ \`${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.RIGHT}\`\n`
    );
}

/**
 * 段階的指示解釈セクションを生成。
 *
 * @returns {string}
 */
function generateStepByStepSection() {
    return (
        `## 段階的な指示の解釈\n` +
        `番号付きの指示がある場合。\n` +
        `1. 各ステップは前のステップの終了位置から開始\n` +
        `2. 指示の順序は重要 - 変更してはいけません\n` +
        `3. 「進めます」「進んでください」などの表現の違いは無視し、移動指示として統一的に解釈\n` +
        `4. 各ステップの数値指示を正確に実行する\n` +
        `5. 指示されていない追加の移動は絶対に行わない\n` +
        '\n' +
        `### 重要事項\n` +
        `ユーザーの指示通りの移動のみを出力してください。\n` +
        '\n' +
        `### 例\n` +
        `指示「1. 下に4マス 2. 右に1マス 3. 下に1マス 4. 右に4マス」の場合：\n` +
        `- 正しい出力: \`${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.RIGHT}${movementKey.DOWN}${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.RIGHT}\`\n` +
        `- 間違った出力: \`${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.RIGHT}${movementKey.DOWN}${movementKey.RIGHT}${movementKey.DOWN}${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.RIGHT}\` （余計な下移動を追加）\n`
    );
}

/**
 * 暗黙的情報セクションを生成。
 *
 * @returns {string}
 */
function generateImplicitInformationSection() {
    return (
        `## 指示に含まれる暗黙的な情報\n` +
        `ユーザーはあなたを安全にゴールまで導こうとしています。\n` +
        `ユーザーの指示から以下を推測してください。\n` +
        `- 特定の順序での移動 → その経路上に障害物がない\n` +
        `- 迂回するような指示 → 直線経路上に障害物がある可能性\n` +
        `- オブジェクトを目印にした指示 → そのオブジェクトまでは安全\n`
    );
}

/**
 * 指示解釈の原則セクションを生成。
 *
 * @returns {string}
 */
function generateInterpretationPrinciplesSection() {
    return (
        `## 指示解釈の重要な原則\n` +
        `1. ユーザーの指示の順序を守る（障害物回避のため）\n` +
        `2. 「〜まで」という表現は、認識可能な位置への移動を意味する\n` +
        `3. 一見非効率な経路も、見えない危険を避けるための最適解\n` +
        `4. 指示を「最適化」せず、忠実に実行する\n`
    );
}

/**
 * 実行時の心構えセクションを生成。
 *
 * @returns {string}
 */
function generateExecutionMindsetSection() {
    return (
        `## 実行時の心構え\n` +
        `1. ユーザーの指示を信頼し、正確に実行する\n` +
        `2. 見える情報（オブジェクトの位置）を活用して指示を解釈\n` +
        `3. 指示の背後にある意図（障害物回避）を意識する\n` +
        `4. しかし、指示されていない「最適化」は行わない\n`
    );
}

/**
 * 最終的な出力形式セクションを生成。
 *
 * @returns {string}
 */
function generateOutputFormatSection() {
    return (
        `## 最終的な出力形式\n` +
        `応答は移動文字列のみとしてください。説明や解説は一切含めないでください。\n` +
        `使用できる文字は \`${movementKey.UP}\`、\`${movementKey.DOWN}\`、\`${movementKey.LEFT}\`、\`${movementKey.RIGHT}\` の4つだけです。\n` +
        `\n` +
        `### 正しい出力例\n` +
        `\`\`\`\n` +
        `${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.RIGHT}${movementKey.DOWN}${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.RIGHT} (下4回、右1回、下1回、右4回)\n` +
        `${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.LEFT}${movementKey.LEFT}${movementKey.UP} (右2回、下3回、左2回、上1回)\n` +
        `\`\`\`\n` +
        `\n` +
        `### 間違った出力例\n` +
        `- \"下に4マス移動してから...\" （説明を含む）\n` +
        `- \"↓4→1↓1→5\" （数字を含む）\n` +
        `- \"down down down...\" （単語を使用）\n` +
        `- \"↓ ↓ ↓ ↓\" （スペースを含む）\n`
    );
}
```

:::

### 3. `game.js`

3D 迷路を画面に表示するファイルです。\
Three.js というライブラリを使っており、Web で 3D グラフィックスを表示することができます！

:::details ファイルの中身（コピー&ペースト）

```javascript
import * as GameModel from './game-model.js';

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

/**
 * requestAnimationFrameをPromiseでラップする関数。
 *
 * @returns {Promise<number>} タイムスタンプ
 */
function animateFrame() {
    return new Promise((resolve) => {
        requestAnimationFrame((timestamp) => resolve(timestamp));
    });
}

// ######################
// ## ゲームのセットアップ ##
// ######################

/**
 * @typedef {ReturnType<typeof renderMap>} MapContext
 */

/**
 * カメラの設定を更新する共通関数
 * @param {object} params パラメータ
 * @param {object} params.camera カメラオブジェクト
 * @param {number} params.mapDisplayWidth マップの幅
 * @param {number} params.mapDisplayHeight マップの高さ
 */
function updateCameraSettings({ camera, mapDisplayWidth, mapDisplayHeight }) {
    // マップの最大サイズを取得
    const mapMaxSize = Math.max(mapDisplayWidth, mapDisplayHeight);

    // FOVを計算して盤面がcanvas内に確実に収まるようにする
    // カメラ距離を増やしてより広い範囲を表示
    const cameraDistance = mapMaxSize * 2.5; // 2.2 から 2.5 に変更
    const topMargin = 3.0; // 2.5 から 3.0 に増やして余白を確保
    const halfMapSizeWithMargin = mapMaxSize / 2 + topMargin;
    const fov =
        2 * Math.atan(halfMapSizeWithMargin / cameraDistance) * (180 / Math.PI);

    // カメラのFOVを更新
    camera.fov = fov;
    camera.updateProjectionMatrix();

    // カメラの位置を設定
    const cameraHeight = cameraDistance * 0.52;
    const cameraOffsetZ = cameraDistance * 0.48;
    camera.position.set(
        (mapDisplayWidth - 1) / 2,
        cameraHeight,
        (mapDisplayHeight - 1) / 2 + cameraOffsetZ
    );

    // カメラの向きを設定
    camera.lookAt((mapDisplayWidth - 1) / 2, 0, (mapDisplayHeight - 1) / 2);
}

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

    // 宝箱を保存するための変数
    let goalChest = null;

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

    // マップのレイアウトを取得
    const mapDisplayWidth = layout[0].length;
    const mapDisplayHeight = layout.length;

    // game-viewer のサイズから正方形のサイズを計算
    const minSize = Math.min(element.offsetWidth, element.offsetHeight);

    // PerspectiveCamera を作成（初期FOVは後で設定）
    const camera = new threeJS.PerspectiveCamera(45, 1, 0.1, 1000);

    // カメラの設定を更新
    updateCameraSettings({ camera, mapDisplayWidth, mapDisplayHeight });

    // レンダラーを作成
    const renderer = new threeJS.WebGLRenderer({ antialias: true });
    renderer.setSize(minSize, minSize);
    renderer.setClearColor(0xffffff); // 背景色を白に設定
    element.appendChild(renderer.domElement);

    // 環境光を作成
    const ambientLight = new threeJS.AmbientLight(0xffffff, 0.6); // 環境光
    scene.add(ambientLight);

    // 平行光源を作成
    const directionalLight = new threeJS.DirectionalLight(0xffffff, 0.8); // 平行光源
    directionalLight.position.set(0, 10, 0);
    scene.add(directionalLight);

    // ベースとなる大きな盤面を作成
    const boardGeometry = new threeJS.BoxGeometry(
        mapDisplayWidth,
        0.3,
        mapDisplayHeight
    );
    const boardMaterial = new threeJS.MeshStandardMaterial({
        color: 0x8b6f47, // 木目調の色
        side: threeJS.DoubleSide,
    });
    const board = new threeJS.Mesh(boardGeometry, boardMaterial);
    board.position.set(
        (mapDisplayWidth - 1) / 2,
        -0.15,
        (mapDisplayHeight - 1) / 2
    );
    scene.add(board);

    // オブジェクトとトラップの位置を記録
    const objectPositions = [];
    const trapPositions = [];

    // マップタイルを作成
    for (let j = 0; j < mapDisplayHeight; j++) {
        for (let i = 0; i < mapDisplayWidth; i++) {
            const cellAlias = layout[j][i];
            const cellDef = mapConfig.cell[cellAlias];

            // セル定義がない場合はスキップ
            if (!cellDef) {
                continue;
            }

            // セルタイプに応じて適切なレンダリング関数を呼び出す
            let element = null;

            switch (cellDef.type) {
                case 'object':
                    element = GameModel.renderObstacle({
                        threeJS,
                        x: i,
                        z: j,
                    });
                    // オブジェクトの位置を記録
                    objectPositions.push({ x: i, y: j });
                    break;
                case 'trap':
                    element = GameModel.renderTrap({ threeJS, x: i, z: j });
                    // トラップの位置を記録
                    trapPositions.push({ x: i, y: j });
                    break;
                case 'end':
                    // ゴールは宝箱で表示するが、タイルの色も表示
                    let endColorValue = 0xcccccc;
                    if (cellDef.color) {
                        const parsedColor = parseInt(
                            cellDef.color.replace('#', ''),
                            16
                        );
                        endColorValue = isNaN(parsedColor)
                            ? endColorValue
                            : parsedColor;
                    }
                    // タイルを描画
                    const endTile = GameModel.renderTile({
                        threeJS,
                        x: i,
                        z: j,
                        color: endColorValue,
                    });
                    scene.add(endTile);
                    // 宝箱を追加
                    element = GameModel.renderGoal({ threeJS, x: i, z: j });
                    goalChest = element; // 宝箱を保存
                    break;
                case 'start':
                    // スタートタイルは通常のタイルとして描画
                    let startColorValue = 0xcccccc;
                    if (cellDef.color) {
                        const parsedColor = parseInt(
                            cellDef.color.replace('#', ''),
                            16
                        );
                        startColorValue = isNaN(parsedColor)
                            ? startColorValue
                            : parsedColor;
                    }
                    element = GameModel.renderTile({
                        threeJS,
                        x: i,
                        z: j,
                        color: startColorValue,
                    });
                    break;
                case 'normal':
                default:
                    // 通常のタイルを描画
                    let colorValue = 0xcccccc; // デフォルト色
                    if (cellDef.color) {
                        const parsedColor = parseInt(
                            cellDef.color.replace('#', ''),
                            16
                        );
                        colorValue = isNaN(parsedColor)
                            ? colorValue
                            : parsedColor;
                    }
                    element = GameModel.renderTile({
                        threeJS,
                        x: i,
                        z: j,
                        color: colorValue,
                    });
                    break;
            }

            // 要素をシーンに追加
            if (element) {
                scene.add(element);
            }
        }
    }

    // 初期レンダリング
    renderer.render(scene, camera);

    // リサイズハンドラーは setupGame で設定される
    return {
        scene,
        camera,
        renderer,
        position,
        mapDisplayWidth,
        mapDisplayHeight,
        goalChest, // 宝箱を返す
        objectPositions, // オブジェクトの位置を返す
        trapPositions, // トラップの位置を返す
    };
}

/**
 * プレイヤーキャラクターを作成し、シーンに追加する。
 *
 * @param   {object}     params            プレイヤー作成パラメータ
 * @param   {ThreeJS}    params.threeJS    Three.js インスタンス
 * @param   {MapContext} params.mapContext マップコンテキスト
 * @returns                                     プレイヤー
 */
function createPlayer({ threeJS, mapContext }) {
    // ロボット（プレイヤー）を作成
    const player = GameModel.renderRobot({
        threeJS,
        x: Math.round(mapContext.position.start.x),
        y: 0, // Y座標は renderRobot 内で調整される
        z: Math.round(mapContext.position.start.y),
    });

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
 * ゲームのレンダリング処理を行う。\
 * Three.js のロード、マップレンダリング、プレイヤー作成を順番に実行する。
 *
 * @param   {object}      params           セットアップパラメータ
 * @param   {HTMLElement} params.element   レンダリング対象の要素
 * @param   {object}      params.mapConfig マップ設定オブジェクト
 * @returns                                コンテキストオブジェクト
 * @throws  {Error}                        Three.js のロードまたはゲームの初期化に失敗した場合エラーをスロー
 */
export async function renderGame({ element, mapConfig }) {
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

    // ウィンドウリサイズ処理を更新
    window.addEventListener('resize', () => {
        if (mapContext.camera && mapContext.renderer && element) {
            // game-viewer のサイズから正方形のサイズを再計算
            const newMinSize = Math.min(
                element.offsetWidth,
                element.offsetHeight
            );

            // カメラの設定を更新
            updateCameraSettings({
                camera: mapContext.camera,
                mapDisplayWidth: mapContext.mapDisplayWidth,
                mapDisplayHeight: mapContext.mapDisplayHeight,
            });

            // レンダラーのサイズを正方形に更新
            mapContext.renderer.setSize(newMinSize, newMinSize);
            mapContext.renderer.render(mapContext.scene, mapContext.camera);
        }
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
 * 宝箱が回転しながら上昇するアニメーション。
 *
 * @param {GameContext} context ゲームコンテキスト
 */
function animateGoalChest(context) {
    const { renderer, scene, camera, goalChest } = context;

    if (!goalChest) return;

    const duration = 2000; // 2秒間のアニメーション
    let startTime = null;
    const startY = goalChest.position.y;
    const targetY = startY + 1.5; // 1.5ユニット上昇（以前の3から減らした）
    const totalRotations = 5; // 5回転

    function animate(currentTime) {
        if (startTime === null) startTime = currentTime;
        const elapsedTime = currentTime - startTime;
        const t = Math.min(elapsedTime / duration, 1);

        // 上昇
        goalChest.position.y = startY + (targetY - startY) * t;

        // 回転（最後に正面を向く）
        if (t < 1) {
            // アニメーション中は回転
            goalChest.rotation.y = t * Math.PI * 2 * totalRotations;
        } else {
            // アニメーション終了時は正面を向く
            goalChest.rotation.y = 0;
            // アニメーション完了後に成功アラートを表示
            setTimeout(() => {
                alert('ゴールに到達しました！ おめでとうございます！');
            }, 100);
        }

        renderer.render(scene, camera);

        if (t < 1) {
            requestAnimationFrame(animate);
        }
    }

    requestAnimationFrame(animate);
}

/**
 * 移動コマンド文字の定義。
 */
export const movementKey = {
    LEFT: '←',
    RIGHT: '→',
    UP: '↑',
    DOWN: '↓',
};

/**
 * プレイヤーを指定された方向に1ステップ移動させ、アニメーション表示する。
 *
 * @param   {object}        params           移動パラメータ
 * @param   {GameContext}   params.context   ゲームコンテキストオブジェクト
 * @param   {string}        params.direction 移動方向
 * @returns {Promise<void>}              アニメーション完了時に解決されるPromise
 */
export async function movePlayer({ context, direction }) {
    const {
        threeJS,
        scene,
        camera,
        renderer,
        player,
        mapDisplayWidth,
        mapDisplayHeight,
    } = context;

    // 方向の設定と早期リターン
    const directionConfig = {
        [movementKey.RIGHT]: { x: 1, z: 0, rotation: Math.PI / 2 },
        [movementKey.LEFT]: { x: -1, z: 0, rotation: -Math.PI / 2 },
        [movementKey.UP]: { x: 0, z: -1, rotation: Math.PI },
        [movementKey.DOWN]: { x: 0, z: 1, rotation: 0 },
    };

    const config = directionConfig[direction];
    if (!config) {
        console.log('未知の方向:', direction);
        return; // 未知の方向の場合は早期リターン
    }

    const stepVec = { x: config.x, z: config.z };
    const targetRotationY = config.rotation;

    // 現在位置を整数に丸めてから計算（浮動小数点誤差を防ぐ）
    const currentX = Math.round(player.position.x);
    const currentZ = Math.round(player.position.z);
    const startPos = player.position.clone();

    const targetX = currentX + stepVec.x;
    const targetZ = currentZ + stepVec.z;

    // 盤外への移動を防ぐ（早期リターン）
    const isOutOfBounds =
        targetX < 0 ||
        targetX >= mapDisplayWidth ||
        targetZ < 0 ||
        targetZ >= mapDisplayHeight;
    if (isOutOfBounds) {
        console.log(
            `盤外への移動をブロック: (${currentX},${currentZ}) -> (${targetX},${targetZ})`
        );
        return;
    }

    // オブジェクトとの衝突判定（早期リターン）
    const isObjectCollision = context.objectPositions.some(
        (pos) => pos.x === targetX && pos.y === targetZ
    );
    if (isObjectCollision) {
        console.log('オブジェクトとの衝突を検出しました');
        return;
    }

    // トラップとの衝突判定（移動を許可し、移動後にチェック）
    const isTrapCollision = context.trapPositions.some(
        (pos) => pos.x === targetX && pos.y === targetZ
    );

    // 最終位置も整数座標にする
    const endPos = new threeJS.Vector3(targetX, player.position.y, targetZ);

    // アニメーションの設定
    const duration = 240;
    const rotationDuration = 100; // 回転用の短い持続時間
    const startRotationY = player.rotation.y;

    // 回転角度の差を最短経路に調整
    let rotationDiff = targetRotationY - startRotationY;
    if (rotationDiff > Math.PI) rotationDiff -= 2 * Math.PI;
    if (rotationDiff < -Math.PI) rotationDiff += 2 * Math.PI;

    // アニメーション開始
    const startTime = await animateFrame();
    let currentTime = startTime;

    // アニメーションループ
    while (true) {
        const elapsedTime = currentTime - startTime;
        const t = Math.min(elapsedTime / duration, 1); // 進行度 (0から1)
        const rotationT = Math.min(elapsedTime / rotationDuration, 1); // 回転用の進行度

        // 位置の補間
        player.position.lerpVectors(startPos, endPos, t);

        // 回転の補間（より早く完了）
        player.rotation.y = startRotationY + rotationDiff * rotationT;

        renderer.render(scene, camera); // シーンをレンダリング

        // アニメーション完了判定
        if (t >= 1) {
            break;
        }

        // 次のフレームを待つ
        currentTime = await animateFrame();
    }

    // 移動完了時の処理
    // トラップに到達したかチェック
    if (isTrapCollision) {
        console.log('トラップに接触しました！');
        context.gameOver = true;
        requestAnimationFrame(() =>
            alert('トラップに接触しました！ ゲームオーバーです。')
        );
        return; // 早期リターン
    }

    // ゴールに到達したかチェック
    const isGoal =
        targetX === context.position.end.x &&
        targetZ === context.position.end.y;
    if (isGoal) {
        console.log('ゴールに到達しました！');
        animateGoalChest(context);
    }
}

/**
 * 宝箱を初期状態にリセットする関数。
 *
 * @param {GameContext} gameContext ゲームコンテキスト
 */
function resetGoalChest(gameContext) {
    const { goalChest } = gameContext;
    if (goalChest) {
        // 宝箱の回転と位置を初期化
        goalChest.rotation.y = 0;
        goalChest.position.y = 0;
        console.log('宝箱を初期状態にリセットしました');
    }
}

/**
 * AIレスポンス表示エリアをレンダリングする。
 *
 * @param {HTMLElement} element レンダリング対象の要素
 */
function renderGameAIResponseViewer(element) {
    // ラベルを作成
    const label = document.createElement('div');
    label.id = 'ai-response-label';
    label.textContent = 'AIからの指示';

    // レスポンス表示エリアを作成
    const response = document.createElement('div');
    response.id = 'ai-response';

    // 要素を追加
    element.appendChild(label);
    element.appendChild(response);
}

/**
 * ゲームトリガーボタンをレンダリングする。
 *
 * @param {HTMLElement} element レンダリング対象の要素
 */
function renderGameTrigger(element) {
    element.textContent = 'スタート';
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
function setupPlayerMoverButton({
    element,
    mapConfig,
    gameContext,
    pathFetcher,
}) {
    const { scene, camera, renderer, player } = gameContext;

    // 初回実行フラグ
    let hasRunOnce = false;

    // ボタンの初期化はrenderGameTriggerで実行済み

    // GameContextにmapDisplayWidthとmapDisplayHeightが含まれているか確認
    console.log(
        'GameContext check:',
        gameContext.mapDisplayWidth,
        'x',
        gameContext.mapDisplayHeight
    );

    element.addEventListener('click', async () => {
        console.log('AIによる経路指示に基づくプレイヤー移動を開始します...');
        element.disabled = true; // 処理中にボタンを無効化

        // 再実行時に宝箱をリセット
        if (hasRunOnce) {
            resetGoalChest(gameContext);

            // ゲームオーバーフラグをリセット
            gameContext.gameOver = false;

            // レスポンス表示をクリア
            const responseEl = document.getElementById('ai-response');
            if (responseEl) {
                responseEl.innerHTML = '';
                responseEl.classList.remove('visible');
            }
        }

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

        // プレイヤーをスタート地点に移動（整数座標に丸める）
        player.position.set(Math.round(startX), 0, Math.round(startY));
        // プレイヤーの向きを初期化（下向き = 手前向き）
        player.rotation.y = 0;
        renderer.render(scene, camera);

        // ローディング表示
        const originalText = element.textContent;
        const originalHTML = element.innerHTML;

        // スピナーを追加
        element.innerHTML = '<span class="spinner"></span>回答待ち';

        try {
            // 経路を取得
            const moves = await pathFetcher();

            // ChatGPTからのレスポンスが完了したら「実行中...」に変更
            element.innerHTML = '実行中...';

            // 経路をコンソールに出力
            console.log('AIからの経路:', moves);

            // レスポンス表示領域の更新
            const responseEl = document.getElementById('ai-response');

            if (responseEl) {
                responseEl.innerHTML = '';
                responseEl.classList.remove('visible');

                // 矢印を一つずつ表示
                const arrows = moves[0].split('');
                const directionMap = {
                    u: '↑',
                    d: '↓',
                    l: '←',
                    r: '→',
                };

                arrows.forEach((arrow, index) => {
                    const span = document.createElement('span');
                    span.textContent = directionMap[arrow] || arrow;
                    span.style.animationDelay = `${index * 100}ms`;
                    responseEl.appendChild(span);
                });

                // 少し遅れて全体を表示
                setTimeout(() => {
                    responseEl.classList.add('visible');
                }, 50);
            }

            // 経路がない場合はエラーを表示して終了
            if (!moves || moves.length === 0) {
                alert('AIから有効な経路が返されませんでした。');
                return;
            }

            const movingDelay = 80; // 移動間の遅延を短縮
            const moveSequence = moves[0]; // "lru" のような文字列

            // 移動をキャンセル可能にするためのフラグ
            let shouldContinue = true;
            const movePromises = [];

            // ゲームオーバー監視用のインターバル
            const gameOverCheckInterval = setInterval(() => {
                if (gameContext.gameOver) {
                    shouldContinue = false;
                    clearInterval(gameOverCheckInterval);
                }
            }, 40);

            // 全ての移動を並列に開始しますが、前の移動の完了を待機
            let previousMovePromise = Promise.resolve();

            for (let i = 0; i < moveSequence.length; i++) {
                // 移動を中断すべきかチェック
                if (!shouldContinue || gameContext.gameOver) {
                    console.log('ゲームオーバーのため移動を中断します');
                    break;
                }

                const moveChar = moveSequence[i];
                const currentIndex = i;

                // 前の移動完了と遅延を待ってから次の移動を開始
                previousMovePromise = previousMovePromise.then(async () => {
                    // ゲームオーバーチェック
                    if (gameContext.gameOver) {
                        shouldContinue = false;
                        return;
                    }

                    // 初回以外は遅延を入れる
                    if (currentIndex > 0) {
                        await new Promise((resolve) =>
                            setTimeout(resolve, movingDelay)
                        );
                    }

                    // 再度ゲームオーバーチェック
                    if (gameContext.gameOver) {
                        shouldContinue = false;
                        return;
                    }

                    // 移動を実行
                    return movePlayer({
                        context: gameContext,
                        direction: moveChar,
                    });
                });
            }

            // 最後の移動の完了を待つ
            await previousMovePromise;

            clearInterval(gameOverCheckInterval);

            // ↓ エラーが発生した場合の処理
        } catch (error) {
            console.error(
                'AI経路取得またはプレイヤー移動の実行中にエラーが発生しました:',
                error
            );
            // エラー時もHTMLを復元
            element.innerHTML = originalHTML;

            // ↓ 処理が全て完了した時の処理
        } finally {
            element.disabled = false; // 処理完了後またはエラー時にボタンを再度有効化

            // 初回実行後にボタンのラベルを「再実行」に変更
            if (!hasRunOnce) {
                element.innerHTML = '再実行';
                hasRunOnce = true;
            } else {
                // 2回目以降は「再実行」に戻す
                element.innerHTML = '再実行';
            }
        }
    });
}

/**
 * ゲームの統合セットアップを行う。
 * 各要素のレンダリング、ゲームの初期化、イベントハンドラの設定を行う。
 *
 * @param {object} params セットアップパラメータ
 * @param {HTMLElement} params.gameViewerEl ゲーム表示エリア
 * @param {HTMLElement} params.responseViewerEl AIレスポンス表示エリア
 * @param {HTMLElement} params.triggerEl トリガーボタン
 * @param {object} params.mapConfig マップ設定
 * @param {function} params.pathFetcher 経路取得関数
 */
export async function setupGame({
    gameViewerEl,
    responseViewerEl,
    triggerEl,
    mapConfig,
    pathFetcher,
}) {
    // AIレスポンス表示エリアをレンダリング
    renderGameAIResponseViewer(responseViewerEl);

    // トリガーボタンをレンダリング
    renderGameTrigger(triggerEl);

    // ゲームをレンダリング
    const gameContext = await renderGame({
        element: gameViewerEl,
        mapConfig,
    });

    // プレイヤー移動ボタンをセットアップ
    setupPlayerMoverButton({
        element: triggerEl,
        mapConfig,
        gameContext,
        pathFetcher,
    });

    return gameContext;
}
```

:::

### 4. `game-model.js`

AI ロボットやオブジェクトのモデリングを行うファイルです。

:::details ファイルの中身（コピー&ペースト）

```javascript
/**
 * ロボット（プレイヤー）を描画する。
 *
 * @param   {object} params         パラメータ
 * @param   {object} params.threeJS Three.js インスタンス
 * @param   {number} params.x       X座標
 * @param   {number} params.y       Y座標
 * @param   {number} params.z       Z座標
 * @returns {object}                ロボットのメッシュオブジェクト
 */
export function renderRobot({ threeJS, x, y, z }) {
    const group = new threeJS.Group();

    // メインボディ（丸みを帯びた箱型）
    const bodyGeometry = new threeJS.BoxGeometry(0.6, 0.5, 0.5);
    const bodyMaterial = new threeJS.MeshStandardMaterial({
        color: 0xf5f5dc, // ベージュ色
        metalness: 0.1,
        roughness: 0.7,
    });
    const body = new threeJS.Mesh(bodyGeometry, bodyMaterial);
    body.position.y = 0.35;

    // ボディの角を丸くするためにスケール調整
    body.scale.set(1, 1, 0.9);
    group.add(body);

    // 顔のプレート（青色のスクリーン部分）
    const faceGeometry = new threeJS.BoxGeometry(0.35, 0.25, 0.02);
    const faceMaterial = new threeJS.MeshStandardMaterial({
        color: 0x4a90e2, // 明るい青色
        metalness: 0.2,
        roughness: 0.6,
    });
    const face = new threeJS.Mesh(faceGeometry, faceMaterial);
    face.position.set(0, 0.35, 0.26); // ボディの前面に配置
    group.add(face);

    // 目（左）
    const eyeGeometry = new threeJS.BoxGeometry(0.08, 0.12, 0.02);
    const eyeMaterial = new threeJS.MeshStandardMaterial({
        color: 0x1a1a1a,
        metalness: 0.8,
        roughness: 0.2,
    });
    const leftEye = new threeJS.Mesh(eyeGeometry, eyeMaterial);
    leftEye.position.set(-0.1, 0.35, 0.27);
    group.add(leftEye);

    // 目（右）
    const rightEye = new threeJS.Mesh(eyeGeometry, eyeMaterial);
    rightEye.position.set(0.1, 0.35, 0.27);
    group.add(rightEye);

    // 車輪（左）
    const wheelGeometry = new threeJS.CylinderGeometry(0.15, 0.15, 0.1, 16);
    const wheelMaterial = new threeJS.MeshStandardMaterial({
        color: 0x2c2c2c,
        metalness: 0.3,
        roughness: 0.8,
    });
    const leftWheel = new threeJS.Mesh(wheelGeometry, wheelMaterial);
    leftWheel.rotation.z = Math.PI / 2;
    leftWheel.position.set(-0.35, 0.15, 0);
    group.add(leftWheel);

    // 車輪（右）
    const rightWheel = new threeJS.Mesh(wheelGeometry, wheelMaterial);
    rightWheel.rotation.z = Math.PI / 2;
    rightWheel.position.set(0.35, 0.15, 0);
    group.add(rightWheel);

    // 上部の三角マーカー（方向を示す）
    const markerGeometry = new threeJS.ConeGeometry(0.1, 0.15, 4);
    const markerMaterial = new threeJS.MeshStandardMaterial({
        color: 0x333333,
        metalness: 0.2,
        roughness: 0.8,
    });
    const marker = new threeJS.Mesh(markerGeometry, markerMaterial);
    marker.position.y = 0.7;
    marker.rotation.y = Math.PI / 4; // 45度回転してダイヤモンド形に
    group.add(marker);

    // 位置を設定
    group.position.set(x, y, z);

    return group;
}

/**
 * 障害物（土管）を描画する。
 *
 * @param   {object} params         パラメータ
 * @param   {object} params.threeJS Three.js インスタンス
 * @param   {number} params.x       X座標
 * @param   {number} params.z       Z座標
 * @returns {object}                土管のメッシュオブジェクト
 */
export function renderObstacle({ threeJS, x, z }) {
    const group = new threeJS.Group();

    // 土管の本体
    const pipeGeometry = new threeJS.CylinderGeometry(0.4, 0.4, 0.8, 16);
    const pipeMaterial = new threeJS.MeshStandardMaterial({
        color: 0x228822,
        metalness: 0.1,
        roughness: 0.8,
    });
    const pipe = new threeJS.Mesh(pipeGeometry, pipeMaterial);
    pipe.position.y = 0.4;
    group.add(pipe);

    // 土管の縁（上）
    const rimGeometry = new threeJS.TorusGeometry(0.4, 0.05, 8, 16);
    const rimMaterial = new threeJS.MeshStandardMaterial({
        color: 0x33aa33,
        metalness: 0.1,
        roughness: 0.8,
    });
    const topRim = new threeJS.Mesh(rimGeometry, rimMaterial);
    topRim.rotation.x = Math.PI / 2;
    topRim.position.y = 0.8;
    group.add(topRim);

    // 土管の縁（下）
    const bottomRim = new threeJS.Mesh(rimGeometry, rimMaterial);
    bottomRim.rotation.x = Math.PI / 2;
    bottomRim.position.y = 0;
    group.add(bottomRim);

    // 土管の内側（黒い空洞を表現）
    const innerGeometry = new threeJS.CylinderGeometry(
        0.35,
        0.35,
        0.79,
        16,
        1,
        true
    );
    const innerMaterial = new threeJS.MeshBasicMaterial({
        color: 0x000000,
        side: threeJS.BackSide, // 内側から見えるように
    });
    const innerPipe = new threeJS.Mesh(innerGeometry, innerMaterial);
    innerPipe.position.y = 0.395; // 上端が土管の上端とほぼ同じ高さに
    group.add(innerPipe);

    // 土管の底（内部を黒く見せるため）
    const bottomCapGeometry = new threeJS.CircleGeometry(0.35, 16);
    const bottomCapMaterial = new threeJS.MeshBasicMaterial({
        color: 0x000000,
        side: threeJS.DoubleSide,
    });
    const bottomCap = new threeJS.Mesh(bottomCapGeometry, bottomCapMaterial);
    bottomCap.rotation.x = -Math.PI / 2;
    bottomCap.position.y = 0.01; // ほぼ底の位置
    group.add(bottomCap);

    // 位置を設定
    group.position.set(x, 0, z);

    return group;
}

/**
 * トラップ（針）を描画する。
 *
 * @param   {object} params         パラメータ
 * @param   {object} params.threeJS Three.js インスタンス
 * @param   {number} params.x       X座標
 * @param   {number} params.z       Z座標
 * @returns {object}                針のメッシュオブジェクト
 */
export function renderTrap({ threeJS, x, z }) {
    const group = new threeJS.Group();

    // ベース（土台）
    const baseGeometry = new threeJS.BoxGeometry(0.9, 0.05, 0.9);
    const baseMaterial = new threeJS.MeshStandardMaterial({
        color: 0x999999,
        metalness: 0.5,
        roughness: 0.3,
    });
    const base = new threeJS.Mesh(baseGeometry, baseMaterial);
    base.position.y = 0.025;
    group.add(base);

    // 針を複数配置
    const spikeMaterial = new threeJS.MeshStandardMaterial({
        color: 0xcccccc,
        metalness: 0.8,
        roughness: 0.2,
    });

    for (let i = -1; i <= 1; i++) {
        for (let j = -1; j <= 1; j++) {
            const spikeGeometry = new threeJS.ConeGeometry(0.08, 0.3, 8);
            const spike = new threeJS.Mesh(spikeGeometry, spikeMaterial);
            spike.position.set(i * 0.25, 0.2, j * 0.25);
            group.add(spike);
        }
    }

    // 位置を設定
    group.position.set(x, 0, z);

    return group;
}

/**
 * ゴール（宝箱）を描画する。
 *
 * @param   {object} params         パラメータ
 * @param   {object} params.threeJS Three.js インスタンス
 * @param   {number} params.x       X座標
 * @param   {number} params.z       Z座標
 * @returns {object}                宝箱のメッシュオブジェクト
 */
export function renderGoal({ threeJS, x, z }) {
    const group = new threeJS.Group();

    // 宝箱の本体
    const boxGeometry = new threeJS.BoxGeometry(0.7, 0.5, 0.5);
    const boxMaterial = new threeJS.MeshStandardMaterial({
        color: 0x8b4513,
        metalness: 0.1,
        roughness: 0.9,
    });
    const box = new threeJS.Mesh(boxGeometry, boxMaterial);
    box.position.y = 0.25;
    group.add(box);

    // 宝箱の蓋
    const lidGeometry = new threeJS.BoxGeometry(0.7, 0.15, 0.5);
    const lidMaterial = new threeJS.MeshStandardMaterial({
        color: 0xa0522d,
        metalness: 0.1,
        roughness: 0.9,
    });
    const lid = new threeJS.Mesh(lidGeometry, lidMaterial);
    lid.position.set(0, 0.525, -0.1);
    lid.rotation.x = -0.3; // 少し開いた状態
    group.add(lid);

    // 金の装飾（中央）
    const decorationGeometry = new threeJS.BoxGeometry(0.1, 0.3, 0.02);
    const decorationMaterial = new threeJS.MeshStandardMaterial({
        color: 0xffd700,
        metalness: 0.7,
        roughness: 0.3,
    });
    const decoration = new threeJS.Mesh(decorationGeometry, decorationMaterial);
    decoration.position.set(0, 0.25, 0.26);
    group.add(decoration);

    // 金の装飾（横バー）
    const barGeometry = new threeJS.BoxGeometry(0.7, 0.05, 0.02);
    const bar = new threeJS.Mesh(barGeometry, decorationMaterial);
    bar.position.set(0, 0.25, 0.26);
    group.add(bar);

    // 位置を設定
    group.position.set(x, 0, z);

    return group;
}

/**
 * 通常のタイルを描画する。
 *
 * @param   {object} params         パラメータ
 * @param   {object} params.threeJS Three.js インスタンス
 * @param   {number} params.x       X座標
 * @param   {number} params.z       Z座標
 * @param   {number} params.color   タイルの色
 * @returns {object}                タイルのメッシュオブジェクト
 */
export function renderTile({ threeJS, x, z, color }) {
    const geometry = new threeJS.PlaneGeometry(0.98, 0.98);
    const material = new threeJS.MeshStandardMaterial({
        color: color,
        side: threeJS.DoubleSide,
    });

    const tile = new threeJS.Mesh(geometry, material);
    tile.rotation.x = -Math.PI / 2; // X軸を中心に-90度回転して水平に
    tile.position.set(x, 0.01, z); // 盤面の上に少し浮かせて配置

    return tile;
}
```

:::

### 5. `secret.js`

ChatGPT を利用するための機密情報を記述するファイルです！\
後のセクションで正式な値を設定するため、ここでは仮の値を入力しておきます。

:::details ファイルの中身（コピー&ペースト）

```javascript
/**
 * OpenAI(ChatGPT) API の Key。
 *
 * 注意： この API Key は公開してはいけません！！
 *       ローカルで起動して使用する場合は問題ないですが、Web サイトとして公開する場合などは、API Key を必要としている処理をサーバーサイドで記述するなど、API Key は隠す必要があります。
 */
export const OPENAI_API_KEY = 'sk-ここに配布されたAPIキーを入力';
```

:::

後ほど、このファイルの値に API キーを設定します！

<br />

## まとめ

これらの基本ファイルが協力して、3D ゲームを動かします！\
現状はまだ動作しませんが、次の章でマップとロボットを作成していきます！

1.  **ai-fetch.js**: AI と通信するためのファイル
2.  **ai-system-prompt.js**: AI の設定を記述するファイル
3.  **game.js**: 3D 迷路を表示、ゲーム全体をコントロールするファイル
4.  **game-model.js**: 3D のモデリングを行うファイル
5.  **secret.js**: ChatGPT を利用するための機密情報を記述するファイル
