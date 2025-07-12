---
title: '🤎 共通ファイルの実装'
---

## 共通ファイルの実装

続いて、すべての問題に共通して使うファイルの中身を実装します！

<br />

## 進め方のおさらい

Zenn の記事では、コードブロックの右上に表示されるコピーボタンを使って、コードをコピー＆ペーストで進めていきます！

![コピペガイド](/images/nagoya-ai-event-2025-programming-workshop/03_common-core-setup/02_copy-paste-guide.png)

<br />

## 各ファイルの役割

次に、各ファイルがどんな働きをするのか、簡単に見ていきましょう。

すべてのコードを理解しようとしなくても大丈夫です。「全体として何をやっているか」をつかむことを意識して、コードをコピー＆ペーストで動かしてみてください！

### 1. `ai.js`

AI との通信とシステムプロンプト生成を担当するファイルです。
システムプロンプトは非常に長いため、次のセクションで詳しく説明し、その際に `ai.js` の内容をコピー＆ペーストします。

:::details ファイルの中身（コピー&ペースト）

```javascript

```

:::

### 2. `app.js`

アプリケーション全体で使用するユーティリティ関数を定義するファイルです。\
ログ出力やエラー処理などの共通機能を提供します。

:::details ファイルの中身（コピー&ペースト）

```javascript
// ##############
// ## ロガー関数 ##
// ##############

/**
 * @typedef {object} AppLogOptions
 *
 * @property {boolean} [alert] アラートも表示するかどうか
 * @property {Function} [logger] コンソール出力メソッド（console.log, console.error, console.info等）
 */

/**
 * @typedef {object} LogParams
 *
 * @property {string}          type    ログタイプ
 * @property {string}          color   ログの色
 * @property {string|string[]} message ログメッセージ
 */

/**
 * 内部ログ処理関数。
 *
 * @param {LogParams}     params ログパラメータ
 * @param {AppLogOptions} options オプション設定
 */
function log(params, options = {}) {
  const { type, message, color } = params;
  const timestamp = new Date().toLocaleTimeString();
  const header = `[${type.toUpperCase()}]`;
  const style = `color: ${color}; font-weight: bold;`;

  // コンソール出力メソッドの選択（デフォルトはconsole.log）
  const consoleMethod = options.logger || console.log;

  // ヘッダーとメッセージを出力
  const headerWithTime = `%c${header} ${timestamp}`;
  if (Array.isArray(message)) {
    // 配列の場合、各要素間に改行を挿入した新しい配列を作成
    const messageWithNewlines = message.flatMap((item, index) => index === 0 ? [item] : [' \n', item]); // prettier-ignore
    consoleMethod(headerWithTime, style, '\n', ...messageWithNewlines);
  } else {
    consoleMethod(headerWithTime, style, '\n', message);
  }

  // アラートオプションが有効な場合
  if (options.alert) {
    let alertContent;

    /** Primitive 型のみフィルタリングする関数 */
    const filterPrimitives = (item) => {
      const type = typeof item;
      return (
        type === 'string' ||
        type === 'number' ||
        type === 'boolean' ||
        item === null ||
        item === undefined
      );
    };

    // ↓ 配列の場合 Primitive 型のみ抽出
    if (Array.isArray(message)) {
      const primitiveMessages = message.filter(filterPrimitives);
      alertContent = primitiveMessages.join('\n');

      // 単一の Primitive 型の場合
    } else if (filterPrimitives(message)) {
      alertContent = String(message);

      // ↓ Primitive 型でない場合は空文字
    } else {
      alertContent = '';
    }

    // アラートメッセージを構築（ヘッダーなし）
    if (alertContent) {
      alert(alertContent);
    }
  }
}

/**
 * アプリケーション用の共通ロガー関数。
 */
export const appLogger = {
  /**
   * 成功メッセージをログに出力する。
   *
   * @param {any|any[]} message ログメッセージ
   * @param {AppLogOptions}      options オプション設定
   */
  success(message, options = {}) {
    log({ type: 'success', message, color: '#4CAF50' }, options);
  },

  /**
   * 情報メッセージをログに出力する。
   *
   * @param {any|any[]} message ログメッセージ
   * @param {AppLogOptions}      options オプション設定
   */
  info(message, options = {}) {
    log(
      { type: 'info', message, color: '#2196F3' },
      { ...options, logger: console.info }
    );
  },

  /**
   * エラーメッセージをログに出力する。
   *
   * @param {any|any[]} message ログメッセージ
   * @param {AppLogOptions}      options オプション設定
   */
  error(message, options = {}) {
    log(
      { type: 'error', message, color: '#F44336' },
      { ...options, logger: console.error }
    );
  },

  /**
   * 警告メッセージをログに出力する。
   *
   * @param {any|any[]} message ログメッセージ
   * @param {AppLogOptions}      options オプション設定
   */
  warning(message, options = {}) {
    log(
      { type: 'warning', message, color: '#FF9800' },
      { ...options, logger: console.warn }
    );
  },

  /**
   * デバッグメッセージをログに出力する。
   *
   * @param {any|any[]} message ログメッセージ
   * @param {AppLogOptions}      options オプション設定
   */
  debug(message, options = {}) {
    log(
      { type: 'debug', message, color: '#9E9E9E' },
      { ...options, logger: console.debug }
    );
  },

  /**
   * グループを開始する。
   *
   * @param {string} label グループのラベル
   */
  group(label) {
    console.group(label);
  },

  /**
   * 折りたたみ可能なグループを開始する。
   *
   * @param {string|undefined} label グループのラベル
   */
  groupCollapsed(label) {
    const timestamp = new Date().toLocaleTimeString();
    const color = '#9C27B0'; // 紫色
    console.groupCollapsed(
      `%c${label ? `${label} ` : ''}${timestamp}`,
      `color: ${color}; font-weight: bold;`
    );
  },

  /**
   * グループを終了する。
   */
  groupEnd() {
    console.groupEnd();
  },
};

// ##############
// ## エラー定義 ##
// ##############

/**
 * @typedef {object} AppErrorOptions
 *
 * @property {boolean} [shouldAlert] アラートも表示するかどうか
 */

/**
 * アプリケーション専用のエラークラス。
 * 処理を停止する必要があるエラーに使用する。
 *
 * @extends Error
 */
export class AppError extends Error {
  /**
   * エラーの詳細情報。
   *
   * @type {string|string[]}
   */
  details;

  /**
   * アラート表示の有無。
   *
   * @type {boolean}
   */
  shouldAlert;

  /**
   * AppError のコンストラクタ。
   *
   * @param {string|string[]}  message エラーメッセージ
   * @param {AppErrorOptions} options オプション設定
   */
  constructor(message, options = {}) {
    // 配列の場合は最初の要素をエラーメッセージとする
    const errorMessage = Array.isArray(message) ? message[0] : message;
    super(errorMessage);

    this.name = 'AppError';
    this.details = message;
    this.shouldAlert = options.shouldAlert || false;
  }

  /**
   * エラーをログに出力する。
   */
  log() {
    // エラーオブジェクト自体も含めてログ出力
    const logMessage = Array.isArray(this.details)
      ? [...this.details, this]
      : [this.details, this];
    appLogger.error(logMessage, { alert: this.shouldAlert });
  }
}
```

:::

### 3. `game.js`

3D マップを画面に表示するファイルです。\
Three.js というライブラリを使っており、Web で 3D グラフィックスを表示することができます。

:::details ファイルの中身（コピー&ペースト）

```javascript
import { AppError, appLogger } from './app.js';

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
 * @throws                    Three.js のロードに失敗した場合エラーをスロー
 */
async function loadThreeJS() {
  try {
    const threeJS = await import('https://esm.sh/three@0.177.0');
    return threeJS;

    // ↓ エラーが発生した場合の処理
  } catch (error) {
    throw new AppError(['Three.js の動的インポートに失敗しました。', error], {
      shouldAlert: true,
    });
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

// #######################
// ## モデルのレンダリング ##
// ######################

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
export function renderRobotModel({ threeJS, x, y, z }) {
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
export function renderObstacleModel({ threeJS, x, z }) {
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
export function renderTrapModel({ threeJS, x, z }) {
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
export function renderGoalModel({ threeJS, x, z }) {
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
export function renderTileModel({ threeJS, x, z, color }) {
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

// ######################
// ## ゲームのセットアップ ##
// ######################

/**
 * @typedef {ReturnType<typeof renderMap>} MapContext
 */

/**
 * カメラの設定を更新する共通関数。
 *
 * @param {object} params                  パラメータ
 * @param {object} params.camera           カメラオブジェクト
 * @param {number} params.mapDisplayWidth  マップの幅
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
 * @throws                                 スタート位置またはエンド位置が見つからない場合にエラーをスロー
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
    throw new AppError('マップに開始位置が見つかりません。');
  }
  if (!position.end) {
    throw new AppError('マップに終了位置が見つかりません。');
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
          element = renderObstacleModel({
            threeJS,
            x: i,
            z: j,
          });
          // オブジェクトの位置を記録
          objectPositions.push({ x: i, y: j });
          break;
        case 'trap':
          element = renderTrapModel({ threeJS, x: i, z: j });
          // トラップの位置を記録
          trapPositions.push({ x: i, y: j });
          break;

        case 'end':
          // ゴールは宝箱で表示するが、タイルの色も表示
          let endColorValue = 0xcccccc;
          if (cellDef.color) {
            const parsedColor = parseInt(cellDef.color.replace('#', ''), 16);
            endColorValue = isNaN(parsedColor) ? endColorValue : parsedColor;
          }
          // タイルを描画
          const endTile = renderTileModel({
            threeJS,
            x: i,
            z: j,
            color: endColorValue,
          });
          scene.add(endTile);
          // 宝箱を追加
          element = renderGoalModel({ threeJS, x: i, z: j });
          goalChest = element; // 宝箱を保存
          break;

        case 'start':
          // スタートタイルは通常のタイルとして描画
          let startColorValue = 0xcccccc;
          if (cellDef.color) {
            const parsedColor = parseInt(cellDef.color.replace('#', ''), 16);
            startColorValue = isNaN(parsedColor)
              ? startColorValue
              : parsedColor;
          }
          element = renderTileModel({
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
            const parsedColor = parseInt(cellDef.color.replace('#', ''), 16);
            colorValue = isNaN(parsedColor) ? colorValue : parsedColor;
          }
          element = renderTileModel({
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
    goalChest,
    objectPositions,
    trapPositions,
  };
}

/**
 * ロボットキャラクターを作成し、シーンに追加する。
 *
 * @param   {object}     params            ロボット作成パラメータ
 * @param   {ThreeJS}    params.threeJS    Three.js インスタンス
 * @param   {MapContext} params.mapContext マップコンテキスト
 * @returns                                ロボット
 */
function createRobot({ threeJS, mapContext }) {
  // ロボットを作成
  const robot = renderRobotModel({
    threeJS,
    x: Math.round(mapContext.position.start.x),
    y: 0, // Y座標は renderRobot 内で調整される
    z: Math.round(mapContext.position.start.y),
  });

  // ロボットをシーンに追加
  mapContext.scene.add(robot);

  // ロボット表示のための初期レンダリング
  mapContext.renderer.render(mapContext.scene, mapContext.camera);

  return robot;
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
 * @throws                                 Three.js のロードまたはゲームの初期化に失敗した場合エラーをスロー
 */
export async function renderGame({ element, mapConfig }) {
  const threeJS = await loadThreeJS();

  // マップをレンダリングし、シーン、カメラ、レンダラーを取得
  const mapContext = renderMap({
    threeJS,
    element,
    mapConfig,
  });

  // ロボットを作成
  const robot = createRobot({
    threeJS,
    mapContext,
  });

  // ウィンドウリサイズ処理を更新
  window.addEventListener('resize', () => {
    if (mapContext.camera && mapContext.renderer && element) {
      // game-viewer のサイズから正方形のサイズを再計算
      const newMinSize = Math.min(element.offsetWidth, element.offsetHeight);

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
    robot,
    ...mapContext,
  };
}

// #####################
// ## 移動関連のロジック ##
// #####################

/**
 * 宝箱が回転しながら上昇するアニメーション。
 *
 * @param   {GameContext}   context ゲームコンテキスト
 * @returns {Promise<void>}         アニメーション完了後に解決されるPromise
 */
async function animateGoalChest(context) {
  const { renderer, scene, camera, goalChest } = context;

  if (!goalChest) return;

  const duration = 2000; // 2秒間のアニメーション
  let startTime = null;
  const startY = goalChest.position.y;
  const targetY = startY + 1.5; // 1.5ユニット上昇（以前の3から減らした）
  const totalRotations = 5; // 5回転

  return new Promise((resolve) => {
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

        // ↓ アニメーション中はアラートを表示しない
      } else {
        // アニメーション終了時は正面を向く
        goalChest.rotation.y = 0;
        // アニメーション完了後に成功アラートを表示
        setTimeout(() => {
          appLogger.success('お宝を獲得しました！', { alert: true });
          resolve();
        }, 100);
      }

      renderer.render(scene, camera);

      if (t < 1) {
        requestAnimationFrame(animate);
      }
    }

    requestAnimationFrame(animate);
  });
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
 * ロボットを指定された方向に1ステップ移動させ、アニメーション表示する。
 *
 * @param   {object}        params           移動パラメータ
 * @param   {GameContext}   params.context   ゲームコンテキストオブジェクト
 * @param   {string}        params.direction 移動方向
 * @returns {Promise<void>}              アニメーション完了時に解決されるPromise
 */
export async function moveRobot({ context, direction }) {
  const {
    threeJS,
    scene,
    camera,
    renderer,
    robot,
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
    appLogger.warning(['未知の方向を検出しました。', direction]);
    return; // 未知の方向の場合は早期リターン
  }

  const stepVec = { x: config.x, z: config.z };
  const targetRotationY = config.rotation;

  // 現在位置を整数に丸めてから計算（浮動小数点誤差を防ぐ）
  const currentX = Math.round(robot.position.x);
  const currentZ = Math.round(robot.position.z);
  const startPos = robot.position.clone();

  const targetX = currentX + stepVec.x;
  const targetZ = currentZ + stepVec.z;

  // 盤外への移動を防ぐ（早期リターン）
  const isOutOfBounds =
    targetX < 0 ||
    targetX >= mapDisplayWidth ||
    targetZ < 0 ||
    targetZ >= mapDisplayHeight;
  if (isOutOfBounds) {
    appLogger.debug([
      `盤外への移動をブロックしました。`,
      `(${currentX},${currentZ}) -> (${targetX},${targetZ})`,
    ]);
    return;
  }

  // オブジェクトとの衝突判定（早期リターン）
  const isObjectCollision = context.objectPositions.some(
    (pos) => pos.x === targetX && pos.y === targetZ
  );
  if (isObjectCollision) {
    appLogger.debug([
      'オブジェクトとの衝突を検出しました。',
      `(${currentX},${currentZ}) -> (${targetX},${targetZ})`,
    ]);
    return;
  }

  // トラップとの衝突判定（移動を許可し、移動後にチェック）
  const isTrapCollision = context.trapPositions.some(
    (pos) => pos.x === targetX && pos.y === targetZ
  );

  // 最終位置も整数座標にする
  const endPos = new threeJS.Vector3(targetX, robot.position.y, targetZ);

  // アニメーションの設定
  const duration = 240;
  const rotationDuration = 100; // 回転用の短い持続時間
  const startRotationY = robot.rotation.y;

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
    robot.position.lerpVectors(startPos, endPos, t);

    // 回転の補間（より早く完了）
    robot.rotation.y = startRotationY + rotationDiff * rotationT;

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
    context.isGameFinished = true;
    context.isGameError = true;
    requestAnimationFrame(() => {
      appLogger.error('トラップに接触しました！\n' + 'ゲームオーバーです。', {
        alert: true,
      });
    });
    return; // 早期リターン
  }

  // ゴールに到達したかチェック
  const isGoal =
    targetX === context.position.end.x && targetZ === context.position.end.y;
  if (isGoal) {
    context.isGameFinished = true;
    await animateGoalChest(context);
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
  }
}

/**
 * AIレスポンス表示エリアをレンダリングする。
 *
 * @param {HTMLElement} element レンダリング対象の要素
 */
function renderGameResponseViewer(element) {
  // ラベルを作成
  const label = document.createElement('div');
  label.id = 'game-response-viewer-label';
  label.textContent = 'AIからの回答';

  // レスポンス表示エリアを作成
  const response = document.createElement('div');
  response.id = 'game-response-viewer-content';
  response.innerHTML = '‎'; // ダミーテキスト

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
 * ロボット移動処理を開始するボタンのセットアップとイベントリスナーの追加。
 *
 * @param {object}            params             設定パラメータ
 * @param {HTMLButtonElement} params.element     処理を開始するボタン要素
 * @param {MapConfig}         params.mapConfig   マップ設定オブジェクト
 * @param {GameContext}       params.gameContext ゲームコンテキストオブジェクト
 * @param {function}          params.pathFetcher AIから経路を取得する関数
 */
function setupRobotMoverButton({
  element,
  mapConfig,
  gameContext,
  pathFetcher,
}) {
  const { scene, camera, renderer, robot } = gameContext;

  // 初回実行フラグ
  let hasRunOnce = false;
  element.addEventListener('click', async () => {
    appLogger.groupCollapsed();

    appLogger.info('AIによる経路指示に基づくプレイヤー移動を開始します。');
    element.disabled = true; // 処理中にボタンを無効化

    // 再実行時に宝箱をリセット
    if (hasRunOnce) {
      resetGoalChest(gameContext);

      // ゲーム終了フラグをリセット
      gameContext.isGameFinished = false;
      gameContext.isGameError = false;

      // レスポンス表示をクリアしてダミーテキストを設定
      const responseEl = document.getElementById(
        'game-response-viewer-content'
      );
      if (responseEl) {
        responseEl.innerHTML = '‎'; // ダミーテキスト
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
      appLogger.error(
        'スタート地点がマップ設定に見つかりませんでした。ロボットはリセットされません。',
        { alert: true }
      );
      return;
    }

    // ロボットをスタート地点に移動（整数座標に丸める）
    robot.position.set(Math.round(startX), 0, Math.round(startY));
    robot.rotation.y = 0; // ロボットの向きを初期化（下向き = 手前向き）
    renderer.render(scene, camera);

    // ローディング表示
    const originalHTML = element.innerHTML;

    // スピナーを追加
    element.innerHTML = '<span class="spinner"></span>';
    try {
      // AIへのリクエスト開始をログ
      appLogger.info('AIに経路を問い合わせています。');

      // 経路を取得
      const moves = await pathFetcher();

      // AIからのレスポンス完了をログ
      appLogger.info(['AIからの経路を取得しました。', moves]);

      // ChatGPTからのレスポンスが完了したら「実行中」に変更
      element.innerHTML = '実行中';

      // レスポンス表示領域の更新
      const responseEl = document.getElementById(
        'game-response-viewer-content'
      );

      if (responseEl) {
        responseEl.innerHTML = '';
        responseEl.classList.remove('visible');

        // 矢印を一つずつ表示
        const moveChars = moves[0].split('');
        moveChars.forEach((char, index) => {
          const span = document.createElement('span');
          span.textContent = char;
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
        const errorMessage = 'AIから有効な経路が返されませんでした。';
        appLogger.error(errorMessage, { alert: true });
        return;
      }

      const movingDelay = 80; // 移動間の遅延を短縮
      const moveSequence = moves[0]; // "lru" のような文字列

      // 移動をキャンセル可能にするためのフラグ
      let shouldContinue = true;

      // ゲーム終了監視用のインターバル
      const gameFinishCheckInterval = setInterval(() => {
        if (gameContext.isGameFinished) {
          shouldContinue = false;
          clearInterval(gameFinishCheckInterval);
        }
      }, 40);

      // 全ての移動を並列に開始しますが、前の移動の完了を待機
      let previousMovePromise = Promise.resolve();

      for (let i = 0; i < moveSequence.length; i++) {
        // 移動を中断すべきかチェック
        if (!shouldContinue || gameContext.isGameFinished) {
          appLogger.info('ゲーム終了のため移動を中断します。');
          break;
        }

        const moveChar = moveSequence[i];
        const currentIndex = i;

        // 前の移動完了と遅延を待ってから次の移動を開始
        previousMovePromise = previousMovePromise.then(async () => {
          // ゲーム終了チェック
          if (gameContext.isGameFinished) {
            shouldContinue = false;
            return;
          }

          // 初回以外は遅延を入れる
          if (currentIndex > 0) {
            await new Promise((resolve) => setTimeout(resolve, movingDelay));
          }

          // 再度ゲーム終了チェック
          if (gameContext.isGameFinished) {
            shouldContinue = false;
            return;
          }

          // 移動を実行
          return moveRobot({
            context: gameContext,
            direction: moveChar,
          });
        });
      }

      // 最後の移動の完了を待つ
      await previousMovePromise;
      clearInterval(gameFinishCheckInterval);

      // ↓ エラーが発生した場合の処理
    } catch (error) {
      const errors = ['予期せぬエラーが発生しました。', error];
      error instanceof AppError ? error.log() : appLogger.error(errors);
      element.innerHTML = originalHTML; // エラー時もHTMLを復元

      // ↓ 処理が全て完了した時の処理
    } finally {
      element.disabled = false; // 処理完了後またはエラー時にボタンを再度有効化
      element.innerHTML = '再実行';
      hasRunOnce = true;
      appLogger.groupEnd();
    }
  });
}

/**
 * ゲームに必要な要素を取得する。
 *
 * @param   {object}      elements                      要素のオブジェクト
 * @param   {HTMLElement} [elements.gameViewer]         ゲーム表示エリア
 * @param   {HTMLElement} [elements.gameResponseViewer] AIレスポンス表示エリア
 * @param   {HTMLElement} [elements.gameTrigger]        トリガーボタン
 * @returns {object}                                    取得した要素のオブジェクト
 * @throws                                              必要な要素が見つからない場合
 */
function getGameElement(elements = {}) {
  const viewerEl =
    elements.gameViewer || document.getElementById('game-viewer');
  if (!viewerEl) {
    throw new AppError('ゲーム表示エリアが見つかりません。');
  }

  const responseViewerEl =
    elements.gameResponseViewer ||
    document.getElementById('game-response-viewer');
  if (!responseViewerEl) {
    throw new AppError('AIレスポンス表示エリアが見つかりません。');
  }

  const triggerEl =
    elements.gameTrigger || document.getElementById('game-trigger');
  if (!triggerEl) {
    throw new AppError('トリガーボタンが見つかりません。');
  }

  return {
    viewerEl,
    responseViewerEl,
    triggerEl,
  };
}

/**
 * ゲームの統合セットアップを行う。
 * 各要素のレンダリング、ゲームの初期化、イベントハンドラの設定を行う。
 *
 * @param {object}      params                               セットアップパラメータ
 * @param {object}      params.mapConfig                    yマップ設定
 * @param {function}    params.pathFetcher                   経路取得関数
 * @param {object}      [params.element]                    要素のオブジェクト
 * @param {HTMLElement} [params.element.gameViewer]         ゲーム表示エリア
 * @param {HTMLElement} [params.element.gameResponseViewer] レスポンス表示エリア
 * @param {HTMLElement} [params.element.gameTrigger]        トリガーボタン
 */
export async function setupGame({ mapConfig, pathFetcher, element = {} }) {
  try {
    const gameElement = getGameElement(element);

    // AIレスポンス表示エリアをレンダリング
    renderGameResponseViewer(gameElement.responseViewerEl);

    // トリガーボタンをレンダリング
    renderGameTrigger(gameElement.triggerEl);

    // ゲームをレンダリング
    const gameContext = await renderGame({
      element: gameElement.viewerEl,
      mapConfig,
    });

    // ロボット移動ボタンをセットアップ
    setupRobotMoverButton({
      element: gameElement.triggerEl,
      mapConfig,
      gameContext,
      pathFetcher,
    });

    return gameContext;

    // ↓ エラーが発生した場合の処理
  } catch (error) {
    error instanceof AppError
      ? error.log()
      : appLogger.error(['ゲームのセットアップに失敗しました。', error]);
  }
}
```

:::

### 4. `secret.js`

ChatGPT を利用するための機密情報を記述するファイルです！\
後のセクションで正式な値を設定するため、ここでは仮の値を入力しておきます。

:::details ファイルの中身（コピー&ペースト）

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

後ほど、このファイルの値に API キーを設定します！

<br />

## 補足

:::details ライブラリとは？
ライブラリとは、よく使われる便利な機能（プログラムの部品）をまとめたものです。\
これを使うことで、難しい処理を自分で一から作らなくても、簡単に呼び出して利用できます。今回の `Three.js` も、3D 描画に関する複雑な処理を簡単にしてくれるライブラリです。
:::

:::details 3D グラフィックスとは？
3D グラフィックスとは、コンピューターグラフィックスで、物体の三次元的な形状データ（モデル）を作成することです。\
`game.js` の中では、`renderRobotModel` や `renderObstacleModel` といった関数が、コードでロボットや土管の形を計算して作り出しており、簡易的な 3D グラフィックスを行っています。
:::

<br />

## ファイルの中身を確認する

今回は作成するファイルの数が多いので、すべてのファイルがしっかりと作成されているか、また中身が正しくコピー＆ペーストされているかを確認してみましょう！

ファイル名を一つずつクリックして、中身が空っぽになっていないか、内容が抜けていないかを丁寧にチェックしてください。

<br />

## トラブルシューティング

何か問題が発生した場合は、以下を参考にしてください！

:::details A. コピー＆ペーストがうまくいかない

#### パターン１： コピーする範囲が違う

**原因**: コードブロックの右上にある「コピー」ボタンではなく、手動でドラッグして選択し、範囲を間違えている。

**対処法**: コードブロックの右上にマウスカーソルを合わせると表示されるコピーボタンを使うように案内し、手動での選択を避けるように伝えます。

<br />

#### パターン２： 貼り付けるファイルが違う

**原因**: ファイルを間違えてペーストしている。

**対処法**: ペーストする前に、VSCode で正しいファイル（例：`game.js`）を開いているか、タブの名前を確認させます。

:::
