---
title: '🩵 ２問目のゲームを作ってみよう'
---

## ２問目のゲームを作ってみよう

では、ここからさらに一歩踏み込んで、次の課題に挑戦していきましょう！
今回の課題は、皆さんの知識と論理的思考力をさらに深めるためのものです。
焦らず、じっくりと取り組んでみてください。

準備はいいですかー？\
それでは、第２問に挑戦です！

<br />

## 新しいマップフォルダを作成する

ではさっそく `map-2` フォルダを作成します。

VSCode の下のパネルに 「ターミナル」 が表示されていることを確認してください。\
もし表示されていなければ、以下のショートカットキーでターミナルを開きます。

- **Windows の場合**: `Ctrl + j`
- **Mac の場合**: `⌘ (Command) が j`

お使いのパソコンに合わせて、以下のコマンドをコピーしてターミナルに貼り付け、Enter キーを押してください。

**PowerShell(Windows) の場合**

```powershell
New-Item -ItemType Directory -Name "map-2"
Copy-Item -Path "map-1/*" -Destination "map-2/" -Recurse
```

:::details コマンドプロンプト(Windows) の場合

```bash
mkdir map-2
copy map-1\* map-2\
```

:::

:::details シェル(Mac,Linux) の場合

```bash
mkdir map-2
cp map-1/* map-2/
```

:::

コマンドを実行すると、一気に左側のファイル一覧に新しいファイルとフォルダが表示されます。これで冒険の準備は完了です。🎉

:::details 【深く知りたい人向け】コマンドの解説

**PowerShell**

- `New-Item -ItemType Directory`: 新しいフォルダを作成
- `Copy-Item`: ファイルやフォルダをコピー
  - `-Path "map-1/*"`: map-1 フォルダ内のすべてのファイルを指定
  - `-Destination`: コピー先を指定
  - `-Recurse`: サブフォルダも含めてコピー

<br />

**コマンドプロンプト**

- `mkdir`: 新しいフォルダを作成
- `copy`: ファイルをコピー
  - `map-1\*`: map-1 フォルダ内のすべてのファイル（\はパス区切り文字）
  - `map-2\`: コピー先のフォルダ

<br />

**シェル（Mac/Linux）**

- `mkdir`: 新しいフォルダを作成
- `cp`: ファイルをコピー（copy）
  - `map-1/*`: map-1 フォルダ内のすべてのファイル（\*はワイルドカード）
  - `map-2/`: コピー先のフォルダ

:::

<br />

## マップを変更する

`map-2` フォルダの `map.js` ファイルを開いて、以下のマップに変更してください。

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
    ['s', 'n', 'o', 't', 'n', 'n', 'n', 'n'],
    ['n', 'n', 'o', 'n', 'n', 'n', 'n', 'n'],
    ['n', 'n', 'o', 'n', 'n', 'o', 'n', 'n'],
    ['n', 'n', 'o', 'n', 'n', 'o', 'n', 'n'],
    ['n', 'n', 'o', 'n', 'n', 'o', 't', 'n'],
    ['n', 'n', 'o', 'n', 'n', 'o', 'n', 'n'],
    ['n', 'n', 'n', 'n', 'n', 'o', 'n', 'n'],
    ['t', 'n', 'n', 'n', 'n', 'o', 'n', 'e'],
  ],
};
```

これで２問目用のマップの準備が完了しました！

<br />

## 動作確認してみる

では、動作確認をしてみましょう！

一度、VSCode の右下にある **「Port: 5500」** と書かれたボタンをクリックして、サーバーを停止します。

![Go Live を停止する](/images/nagoya-ai-event-2025-programming-workshop/09_2nd-game-setup/01_go-live-button-restart-guide.png)

その後、もう一度「Go Live」ボタンをクリックして、サーバーを再起動してください。

ブラウザが自動で立ち上がり、以下のような画面が表示されたら、 **「map-2」** フォルダをクリックしてください。

![map-2 を選択する](/images/nagoya-ai-event-2025-programming-workshop/09_2nd-game-setup/02_go-live-browser-file-list.png)

そうすると、先ほど作成した新しいマップが表示されます。

<br />

## トラブルシューティング

何か問題が発生した場合は、以下を参考にしてください！

:::details A. フォルダやファイルの作成でエラーが出る

**原因**: コマンドの入力ミスや、実行している場所（ディレクトリ）が違う可能性があります。

**対処法**: 前の章（02 章）と同じように、コマンドが正しいか、そして VSCode で正しいフォルダ（`ai-app`）が開かれているか確認してください。`map-2` フォルダが `map-1` と同じ階層に作られているか、VSCode のエクスプローラーで確認してみましょう。

:::
