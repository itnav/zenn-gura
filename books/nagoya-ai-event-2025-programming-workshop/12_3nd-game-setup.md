---
title: '💜 ３問目のゲームを作ってみよう'
---

## ３問目のゲームを作ってみよう

２問目も見事クリア出来ましたでしょうか！

いよいよ最後の問題です。
これまでの経験を活かして、さらに複雑なマップに挑戦してみましょう！
AI ロボットがトラップにかからないよう、いろいろ試行錯誤してみてください 🔥

<br />

## 新しいマップフォルダを作成する

ではさっそく `map-3` フォルダを作成します！

VSCode の下のパネルに「ターミナル」が表示されていることを確認してください！\
もし表示されていなければ、以下のショートカットキーでターミナルを開きます！

- **Windows の場合**: `Ctrl + j`
- **Mac の場合**: `⌘ (Command) + j`

お使いのパソコンに合わせて、以下のコマンドをコピーしてターミナルに貼り付け、Enter キーを押してください！

**PowerShell(Windows) の場合**

```powershell
New-Item -ItemType Directory -Name "map-3"
Copy-Item -Path "map-1/*" -Destination "map-3/" -Recurse
```

:::details コマンドプロンプト(Windows) の場合

```bash
mkdir map-3
copy map-1\* map-3\
```

:::

:::details シェル(Mac,Linux) の場合

```bash
mkdir map-3
cp map-1/* map-3/
```

:::

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
  - `map-3\`: コピー先のフォルダ

<br />

**シェル（Mac/Linux）**

- `mkdir`: 新しいフォルダを作成
- `cp`: ファイルをコピー（copy）
  - `map-1/*`: map-1 フォルダ内のすべてのファイル（\*はワイルドカード）
  - `map-3/`: コピー先のフォルダ

:::

コマンドを実行すると、一気に左側のファイル一覧に新しいファイルとフォルダが表示されます。これで冒険の準備は完了です！🎉

<br />

## マップを変更する

`map-3` フォルダの `map.js` ファイルを開いて、以下のマップに変更してください。

```javascript
export const mapConfig = {
  layout: [
    ['o', 'o', 'o', 'o', 'o', 'o', 'o'],
    ['o', 's', '.', '.', 't', '.', 'o'],
    ['o', 'o', 'o', '.', 'o', '.', 'o'],
    ['o', '.', '.', '.', '.', '.', 'o'],
    ['o', '.', 'o', 't', 'o', 't', 'o'],
    ['o', '.', '.', '.', '.', 'e', 'o'],
    ['o', 'o', 'o', 'o', 'o', 'o', 'o'],
  ],
  cell: {
    o: {
      type: 'object',
      color: '#333333',
    },
    '.': {
      type: 'normal',
      color: '#ffffff',
    },
    s: { type: 'start', color: '#00ff00' },
    e: { type: 'end', color: '#ff0000' },
    t: { type: 'trap', color: '#000000' },
  },
};
```
