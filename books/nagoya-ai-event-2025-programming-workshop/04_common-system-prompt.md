---
title: '🤎 AI ロボットの実装'
---

## AI ロボットの実装

続いて、AI ロボットの頭の中を作っていきましょう！\
今回は AI ロボットを「システムプロンプト」を用いて作っていきます！

<br />

## システムプロンプトってなんだろう？

システムプロンプトとは、AI に「こんな役割で、こんな風に動いてね！」と指示するための、AI へのお願いのようなものです。\
AI はこのお願いを読んで、どのように応答すべきかを理解します！

このゲームでは、AI ロボットがマップを動くためのシステムプロンプトを `ai.js` で作っています。プロンプトにはマップや移動ルールなど、AI が動くための情報が入っています・

<br />

## `ai.js` の中身を実装しよう

それでは、`ai.js` の中身をコピー＆ペーストしてみましょう。このファイルには、AI のシステムプロンプトを動的に生成するロジックと、OpenAI API を呼び出して AI から経路を取得するロジックが含まれています。

:::details ファイルの中身（コピー&ペースト）

```javascript
import { AppError } from './app.js';
import { movementKey } from './game.js';

// #########################
// ## システムプロンプト生成 ##
// #########################

/**
 * AI のためのシステムプロンプトを動的に生成する。\
 * マップ設定のセル定義をプロンプトに含めることで、AI が各セルの意味を理解できるようにする。
 *
 * @param   {import("./game").MapConfig} mapConfig マップ設定のセル定義オブジェクト
 * @returns {string}                               システムプロンプト文字列
 */
export function generateSystemPrompt(mapConfig) {
  const result = [
    '# システムプロンプト\n',
    // 1. 基本設定（役割、ユーザー理解、座標系）
    generateRoleSection(),
    generateUserRoleSection(),
    generateCoordinateSection(mapConfig),
    // 2. マップ理解（セルタイプ、記号定義、レイアウト）
    generateCellTypeSection(),
    generateMapSymbolSection(mapConfig),
    generateMapLayoutSection(mapConfig),
    // 3. 移動ルール（基本ルール、方向解釈、特殊な指示）
    generateMovementRulesSection(mapConfig),
    generateNumericalUnderstandingSection(),
    generateAmbiguousInstructionSection(),
    generateStepByStepSection(),
    generateInterpretationPrinciplesSection(),
    // 4. 出力形式（使用可能コマンド、フォーマット）
    generateMovementInstructionSection(),
    // 5. 重要な注意事項
    generateImportantWarningSection(),
    // 6. 具体例（map-1の例を先に示す）
    generateMap1SuccessExample(),
    // 7. 具体例（map-2の例）
    generateMap2SuccessExample(),
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
    '\n' +
    `あなたはグリッドベースのマップをナビゲートするロボットです。\n` +
    `あなたの目的は、ユーザーからの指示に従って、移動の指示を出力することです。\n` +
    `あなたの初期位置は、'start' の位置です。\n`
  );
}

/**
 * ユーザーの役割理解セクションを生成。
 *
 * @returns {string}
 */
function generateUserRoleSection() {
  return (
    `## ユーザーの役割の理解（重要）\n` +
    '\n' +
    `- ユーザーはマップ全体（トラップやゴールの位置を含む）を見ることができます。\n` +
    `- ユーザーの指示は、あなたには見えない危険を回避し、ゴールへ導くための信頼できるガイドです。\n` +
    `- 指示が一見遠回りに見えても、それはトラップを避けるためかもしれません。\n` +
    `- ユーザーの指示に忠実に従ってください。\n` +
    `- 指示された移動以外の追加の移動は行わないでください。\n`
  );
}

/**
 * 移動ルールと指示解釈セクションを生成。
 *
 * @param   {import("./game").MapConfig} mapConfig
 * @returns {string}
 */
function generateMovementRulesSection(mapConfig) {
  const maxX = mapConfig.layout[0].length - 1;
  const maxY = mapConfig.layout.length - 1;
  return (
    `## 移動ルールと指示の解釈\n` +
    '\n' +
    `### オブジェクトへの移動\n` +
    '\n' +
    `- オブジェクト（土管）があるマスには進入できません\n` +
    `- **「オブジェクトまで」という指示は、オブジェクトの1マス手前で停止することを意味します**\n` +
    '\n' +
    `#### 例：土管が(2,0)にある場合\n` +
    `- 「右の土管まで」で(0,0)から移動 → (1,0)で停止（土管の左側）\n` +
    `- 「下の土管まで」で(2,-1)から移動 → 不可（Y=-1は存在しない）\n` +
    `- 「上の土管まで」で(2,1)から移動 → (2,0)は土管なので停止できない\n` +
    `- 「左の土管まで」で(3,0)から移動 → (2,0)は土管なので停止できない\n` +
    '\n' +
    `### 方向のオブジェクト検索\n` +
    '\n' +
    `**「下のオブジェクト」「右のオブジェクト」などの指示の解釈：**\n` +
    `- 「下のオブジェクト」= 現在位置よりY座標が大きい（下にある）オブジェクトを探す\n` +
    `- 「右のオブジェクト」= 現在位置よりX座標が大きい（右にある）オブジェクトを探す\n` +
    `- 「上のオブジェクト」= 現在位置よりY座標が小さい（上にある）オブジェクトを探す\n` +
    `- 「左のオブジェクト」= 現在位置よりX座標が小さい（左にある）オブジェクトを探す\n` +
    '\n' +
    `**重要：同じX座標やY座標上にある必要はありません。指定された方向にあるオブジェクトを見つけてください。**\n` +
    '\n' +
    `### 「一番○○まで」の解釈\n` +
    '\n' +
    `**「端」「一番○○の端」「一番○○」などの表現は、すべて座標を維持して移動します。**\n` +
    '\n' +
    `- 「一番上の端まで」「一番上まで」= 現在のX座標を維持してY=0まで移動\n` +
    `- 「一番下の端まで」「一番下まで」= 現在のX座標を維持してY=${maxY}まで移動\n` +
    `- 「一番右の端まで」「一番右まで」= 現在のY座標を維持してX=${maxX}まで移動\n` +
    `- 「一番左の端まで」「一番左まで」= 現在のY座標を維持してX=0まで移動\n` +
    '\n' +
    `### 番号付き指示の処理\n` +
    '\n' +
    `- 1から順番にすべてのステップを実行してください\n` +
    `- 各ステップの終了位置が次のステップの開始位置になります\n` +
    `- 途中で止まらず、最後のステップまで必ず実行してください\n` +
    '\n' +
    `### 注意事項\n` +
    '\n' +
    `- あなたが認識できる情報（オブジェクトの位置など）を手がかりに指示を解釈してください\n` +
    `- 指示の順序には意味があります。直線的でない経路は、見えない障害物を避けている可能性があります\n`
  );
}

/**
 * 座標系と方向の理解セクションを生成。
 *
 * @param   {import("./game").MapConfig} mapConfig
 * @returns {string}
 */
function generateCoordinateSection(mapConfig) {
  const maxX = mapConfig.layout[0].length - 1;
  const maxY = mapConfig.layout.length - 1;
  return (
    `## 座標系と方向の理解\n` +
    '\n' +
    `### 座標系の定義\n` +
    '\n' +
    `- 原点: 左上 (0,0)\n` +
    `- X軸： 左から右へ（0 → ${maxX}）\n` +
    `- Y軸： 上から下へ（0 → ${maxY}）\n` +
    `- マップサイズ: ${maxX + 1} × ${maxY + 1}\n` +
    `\n` +
    '### 方向の理解\n' +
    '\n' +
    `- 上 (UP) [${movementKey.UP}]： Y座標を減らす（現在位置から上の行へ）\n` +
    `- 下 (DOWN) [${movementKey.DOWN}]： Y座標を増やす（現在位置から下の行へ）\n` +
    `- 左 (LEFT) [${movementKey.LEFT}]： X座標を減らす（現在位置から左の列へ）\n` +
    `- 右 (RIGHT) [${movementKey.RIGHT}]： X座標を増やす（現在位置から右の列へ）\n`
  );
}

/**
 * 移動指示と出力形式の統合セクションを生成。
 *
 * @returns {string}
 */
function generateMovementInstructionSection() {
  return (
    `## 移動指示と出力形式\n` +
    '\n' +
    `### 使用可能な移動コマンド\n` +
    `移動の指示は以下の4つの文字だけを使用してください。他の文字は一切使用しないでください。\n` +
    //
    `- ${movementKey.UP} : 上へ移動（Y座標 -1）\n` +
    `- ${movementKey.DOWN} : 下へ移動（Y座標 +1）\n` +
    `- ${movementKey.LEFT} : 左へ移動（X座標 -1）\n` +
    `- ${movementKey.RIGHT} : 右へ移動（X座標 +1）\n` +
    '\n' +
    `### 出力ルール\n` +
    `- 応答は移動文字列のみとしてください\n` +
    `- 説明や解説は一切含めないでください\n` +
    `- 文字を連続して出力し、スペースや他の文字を挟まないでください\n` +
    '\n' +
    `### 正しい出力例\n` +
    `\`\`\`\n` +
    `${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.RIGHT}${movementKey.DOWN}${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.RIGHT}\n` +
    `${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.LEFT}${movementKey.LEFT}${movementKey.UP}\n` +
    `\`\`\`\n` +
    '\n' +
    `### 間違った出力例\n` +
    `- \"下に4マス移動してから...\" （説明を含む）\n` +
    `- \"↓4→1↓1→5\" （数字を含む）\n` +
    `- \"down down down...\" （単語を使用）\n` +
    `- \"↓ ↓ ↓ ↓\" （スペースを含む）\n`
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
    '\n' +
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

  // X座標のヘッダーを生成
  const xHeader = Array.from(
    { length: maskedLayout[0].length },
    (_, i) => i
  ).join(' ');

  // マップの各行を文字列に変換
  const layoutRows = maskedLayout
    .map((row, y) => `${y} ${row.join(' ')}`)
    .join('\n');

  // オブジェクトの位置を列挙
  const objectPositions = [];
  mapConfig.layout.forEach((row, y) => {
    row.forEach((cell, x) => {
      const cellConfig = mapConfig.cell[cell];
      if (cellConfig?.type === 'object') {
        objectPositions.push(`(${x},${y})`);
      }
    });
  });

  return (
    `## この固有マップのレイアウトの説明\n` +
    '\n' +
    `### マップレイアウト（${maskedLayout[0].length}×${maskedLayout.length}グリッド）\n` +
    `\`\`\`\n` +
    `  ${xHeader}\n` +
    `${layoutRows}\n` +
    `\`\`\`\n` +
    '\n' +
    `### 座標の読み方\n` +
    '\n' +
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
  let section = `## この固有マップの記号とタイプ定義\n` + '\n';
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
 * 数値理解セクションを生成。
 *
 * @returns {string}
 */
function generateNumericalUnderstandingSection() {
  return (
    `## 数値の理解と移動の実行\n` +
    '\n' +
    `「Nマス進む」という指示を受けた場合。\n` +
    `- 「4マス進む」→ その方向に4回移動（例：下なら\"${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}\"）\n` +
    `- 「1マス進む」→ その方向に1回移動（例：右なら\"${movementKey.RIGHT}\"）\n` +
    `- 数値は必ず移動回数として解釈してください\n` +
    `\n` +
    `### 具体例\n` +
    '\n' +
    `- 「下に4マス」→ \`${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}\`\n` +
    `- 「右に1マス」→ \`${movementKey.RIGHT}\`\n` +
    `- 「右に4マス」→ \`${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.RIGHT}\`\n`
  );
}

/**
 * 曖昧な指示への対処セクションを生成。
 *
 * @returns {string}
 */
function generateAmbiguousInstructionSection() {
  return (
    `## 曖昧な指示への対処\n` +
    `\n` +
    `以下のような曖昧な指示を受けた場合の対処法。\n` +
    `- 「好きなように動いて」「自由に動いて」「適当に動いて」など\n` +
    `    - ランダムな方向に多めに8つほど移動する（例：${movementKey.UP}${movementKey.RIGHT}${movementKey.DOWN}）\n` +
    `- 「探索して」「見て回って」など\n` +
    `    - 現在位置の周辺を小さく移動する（例：${movementKey.RIGHT}${movementKey.DOWN}${movementKey.LEFT}${movementKey.UP}）\n` +
    `\n` +
    `### 重要\n` +
    '\n' +
    `ゴールの位置は不明なので、特定の方向に向かって進むべきではありません。\n` +
    `曖昧な指示の場合は、移動を最小限に留め、より具体的な指示を待つような動きをしてください。\n`
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
    '\n' +
    `番号付きの指示がある場合。\n` +
    `1. 各ステップは前のステップの終了位置から開始\n` +
    `2. 指示の順序は重要 - 変更してはいけません\n` +
    `3. 「進めます」「進んでください」などの表現の違いは無視し、移動指示として統一的に解釈\n` +
    `4. 各ステップの数値指示を正確に実行する\n` +
    `5. 指示されていない追加の移動は絶対に行わない\n` +
    '\n' +
    `### 重要事項\n` +
    '\n' +
    `ユーザーの指示通りの移動のみを出力してください。\n` +
    '\n' +
    `### 例\n` +
    '\n' +
    `指示「1. 下に4マス 2. 右に1マス 3. 下に1マス 4. 右に4マス」の場合。\n` +
    `- 正しい出力: \`${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.RIGHT}${movementKey.DOWN}${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.RIGHT}\`\n` +
    `- 間違った出力: \`${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.RIGHT}${movementKey.DOWN}${movementKey.RIGHT}${movementKey.DOWN}${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.RIGHT}\` （余計な下移動を追加）\n`
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
    '\n' +
    `1. ユーザーの指示の順序を守る（障害物回避のため）\n` +
    `2. 「〜まで」という表現は、認識可能な位置への移動を意味する\n` +
    `3. 一見非効率な経路も、見えない危険を避けるための最適解\n` +
    `4. 指示を「最適化」せず、忠実に実行する\n`
  );
}

/**
 * 重要な注意事項セクションを生成。
 *
 * @returns {string}
 */
function generateImportantWarningSection() {
  return (
    `## ⚠️ 重要な注意事項 ⚠️\n` +
    '\n' +
    `**以下は参考例ですが、絶対にこの例に囚われないでください！**\n` +
    '\n' +
    `- **ユーザーの指示が例と異なっても、必ずユーザーの指示を優先してください**\n` +
    `- **同じマップでも、ユーザーは違う経路や解釈を求めているかもしれません**\n` +
    `- **例は理解を助けるためだけのもので、ユーザーの意図を曲げる理由にはなりません**\n` +
    `- **ユーザーの指示を勝手に「修正」したり「最適化」したりしないでください**\n`
  );
}

/**
 * map-1（6x6）の成功例セクションを生成。
 *
 * @returns {string}
 */
function generateMap1SuccessExample() {
  return (
    `## 完全な成功例（6×6グリッド）\n` +
    '\n' +
    `### マップレイアウト\n` +
    `\`\`\`\n` +
    `  0 1 2 3 4 5\n` +
    `0 s . . . . t\n` +
    `1 . . o . . .\n` +
    `2 . . . . . .\n` +
    `3 . . . t . .\n` +
    `4 . . . . . .\n` +
    `5 o . . . . e\n` +
    `\n` +
    `記号説明:\n` +
    `s = スタート位置 (0,0)\n` +
    `e = ゴール位置 (5,5)\n` +
    `o = 土管（オブジェクト）- 通過不可\n` +
    `. = 通行可能なセル\n` +
    `t = トラップ（あなたには見えない）\n` +
    '\n' +
    `### 指示と正しい解釈\n` +
    `\`\`\`\n` +
    `#### 指示\n` +
    `1. 下の土管まで進んでください。\n` +
    `2. 一番右まで進んでください。\n` +
    `3. 一番下まで進んでください。\n` +
    `\n` +
    `#### 正しい解釈と移動\n` +
    `ステップ1: 下の土管まで\n` +
    `- 開始: (0,0)\n` +
    `- 下の土管: (0,5)を発見（X=0でY>0のオブジェクト）\n` +
    `- 土管の手前で停止: (0,4)\n` +
    `- 移動: ${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}\n` +
    `\n` +
    `ステップ2: 一番右まで\n` +
    `- 開始: (0,4)\n` +
    `- Y座標4を維持、X=5へ\n` +
    `- 終了: (5,4)\n` +
    `- 移動: ${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.RIGHT}\n` +
    `\n` +
    `ステップ3: 一番下まで\n` +
    `- 開始: (5,4)\n` +
    `- X座標5を維持、Y=5へ\n` +
    `- 終了: (5,5) [ゴール]\n` +
    `- 移動: ${movementKey.DOWN}\n` +
    `\n` +
    `#### 最終出力\n` +
    `${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.DOWN}\n` +
    '\n' +
    `### 重要なポイント\n` +
    `1. **「下の土管」= 現在位置より下（Y座標が大きい）にある土管**\n` +
    `2. **土管は通過不可** - 「土管まで」は手前で停止\n` +
    `3. **「一番○○まで」は座標を維持して移動**\n` +
    `4. **各ステップの終了位置が次の開始位置**\n`
  );
}

/**
 * map-2（8x8）の成功例セクションを生成。
 *
 * @returns {string}
 */
function generateMap2SuccessExample() {
  return (
    `## 完全な成功例（8×8グリッド）\n` +
    '\n' +
    `### マップレイアウト\n` +
    `\`\`\`\n` +
    `  0 1 2 3 4 5 6 7\n` +
    `0 s . o t . . . .\n` +
    `1 . . o . . . . .\n` +
    `2 . . o . . o . .\n` +
    `3 . . o . . o . .\n` +
    `4 . . o . . o t .\n` +
    `5 . . o . . o . .\n` +
    `6 . . . . . o . .\n` +
    `7 t . . . . o . e\n` +
    `\n` +
    `記号説明:\n` +
    `s = スタート位置 (0,0)\n` +
    `e = ゴール位置 (7,7)\n` +
    `o = 土管（オブジェクト）- 通過不可\n` +
    `. = 通行可能なセル\n` +
    `t = トラップ（あなたには見えない）\n` +
    '\n' +
    `### 指示と正しい解釈\n` +
    `\`\`\`\n` +
    `#### 指示\n` +
    `1. 右の土管まで進んでください\n` +
    `2. 一番下の端までまっすぐ進んでください\n` +
    `3. 右の土管まで進んでください\n` +
    `4. 一番上の端まで進んでください\n` +
    `5. 一番右の端まで進んでください\n` +
    `6. 一番下の端まで進んでください\n` +
    `\n` +
    `#### 正しい解釈と移動\n` +
    `ステップ1: 右の土管まで\n` +
    `- 開始: (0,0)\n` +
    `- 右の土管: (2,0)を発見\n` +
    `- 土管の手前で停止: (1,0)\n` +
    `- 移動: ${movementKey.RIGHT}\n` +
    `\n` +
    `ステップ2: 一番下の端までまっすぐ\n` +
    `- 開始: (1,0)\n` +
    `- X座標1を維持、Y=7へ\n` +
    `- 終了: (1,7)\n` +
    `- 移動: ${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}\n` +
    `\n` +
    `ステップ3: 右の土管まで\n` +
    `- 開始: (1,7)\n` +
    `- 右の土管: (5,7)を発見\n` +
    `- 土管の手前で停止: (4,7)\n` +
    `- 移動: ${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.RIGHT}\n` +
    `\n` +
    `ステップ4: 一番上の端まで（「まっすぐ」なしでも同じ解釈）\n` +
    `- 開始: (4,7)\n` +
    `- X座標4を維持、Y=0へ\n` +
    `- 終了: (4,0)\n` +
    `- 移動: ${movementKey.UP}${movementKey.UP}${movementKey.UP}${movementKey.UP}${movementKey.UP}${movementKey.UP}${movementKey.UP}\n` +
    `\n` +
    `ステップ5: 一番右の端まで（「まっすぐ」なしでも同じ解釈）\n` +
    `- 開始: (4,0)\n` +
    `- Y座標0を維持、X=7へ\n` +
    `- 終了: (7,0)\n` +
    `- 移動: ${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.RIGHT}\n` +
    `\n` +
    `ステップ6: 一番下の端まで（「まっすぐ」なしでも同じ解釈）\n` +
    `- 開始: (7,0)\n` +
    `- X座標7を維持、Y=7へ\n` +
    `- 終了: (7,7) [ゴール]\n` +
    `- 移動: ${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}\n` +
    `\n` +
    `#### 最終出力\n` +
    `${movementKey.RIGHT}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.UP}${movementKey.UP}${movementKey.UP}${movementKey.UP}${movementKey.UP}${movementKey.UP}${movementKey.UP}${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.RIGHT}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}${movementKey.DOWN}\n` +
    '\n' +
    `### 重要なポイント\n` +
    `1. **土管は通過不可** - 「土管まで」は手前で停止\n` +
    `2. **「まっすぐ」の有無に関わらず、端への移動は座標を維持**\n` +
    `3. **各ステップの終了位置が次の開始位置**\n` +
    `4. **すべてのステップを最後まで処理**\n`
  );
}

// ###########################
// ## OpenAI API 経路取得処理 ##
// ###########################

const allowedCharsList = Object.values(movementKey);
const allowedCharsPattern = allowedCharsList.join('');
const filterRegex = new RegExp(`[^${allowedCharsPattern}]`, 'g');
const extractRegex = new RegExp(`[${allowedCharsPattern}]+`, 'g');

/**
 * OpenAI API を使用して経路パスを取得する。
 *
 * @param {object} params                 AIパス取得のためのパラメータ
 * @param {string} params.apiKey          OpenAI API キー
 * @param {string} params.routePrompt     ユーザー提供の経路プロンプト（指示）
 * @param {string} params.systemPrompt    AIへのシステムプロンプト
 * @returns {Promise<Array<string>|null>} 移動指示の配列（例: ['l', 'r', 'u', 'd']）
 */
export async function fetchRoutePathWithOpenAI({
  apiKey,
  routePrompt,
  systemPrompt,
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
    throw new AppError(
      [
        'OpenAI API のエラーが発生しました。',
        errorData?.message || response.statusText,
        errorData,
      ],
      { shouldAlert: true }
    );
  }

  // レスポンスを JSON として解析する
  const data = await response.json();

  // API からの応答に choices が含まれているか、またその配列が空でないかを確認する
  if (!data.choices || data.choices.length === 0) {
    throw new AppError(
      'OpenAI APIの応答に選択肢 (choices) が含まれていませんでした。',
      { shouldAlert: true }
    );
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
    throw new AppError('AIの応答から有効な移動指示を抽出できませんでした。', {
      shouldAlert: true,
    });
  }

  // 抽出された移動指示の配列を返す
  return moves;
}
```

<br />

## ファイルの中身を確認しよう

先ほどと同じように、中身が正しくコピー＆ペーストされているかを確認してみましょう！

<br />

## トラブルシューティング

何か問題が発生した場合は、以下を参考にしてください！

:::details A. コピー＆ペーストがうまくいかない

**原因**: コードブロックの右上にある「コピー」ボタンではなく、手動でドラッグして選択し、範囲を間違えている。

**対処法**: コードブロックの右上にマウスカーソルを合わせると表示されるコピーボタンを使うように案内し、手動での選択を避けるように伝えます。

:::
