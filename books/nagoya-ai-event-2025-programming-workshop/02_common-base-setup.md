---
title: '🤎 下準備'
---

## 下準備

まずは、冒険を出かける準備をしましょう！

<br />

## Visual Studio Code (VSCode) のセットアップ

### 1. Visual Studio Code を起動

VSCode のアイコンをクリックして起動します。

![VSCodeのアイコン](/images/nagoya-ai-event-2025-programming-workshop/02_base-setup/01_open-vscode.png)
<br />

### 2. フォルダーを開く

VSCode の左サイドバーから「フォルダーを開く」をクリックします。

![VSCodeのフォルダーを開く](/images/nagoya-ai-event-2025-programming-workshop/02_base-setup/02_open-vscode-folder.png)
<br />

### 3. 新しいフォルダを作成して開く

1. フォルダ選択ダイアログが開いたら、新しいフォルダを作成します
2. 「新規フォルダ」ボタンをクリックします
3. フォルダ名を「ai-app」と入力します
4. 「フォルダーの選択」をクリックします

![プロジェクトのフォルダーを開く](/images/nagoya-ai-event-2025-programming-workshop/02_base-setup/03_open-project-folder.png)

左側にファイルの一覧が表示されたら準備完了です！

![プロジェクトのフォルダーが開かれた状態](/images/nagoya-ai-event-2025-programming-workshop/02_base-setup/04_opened-project-folder.png)

<br />

## 必要なファイルを作成する

続いて、必要なファイルを作成していきます！\
ファイルは VSCode のボタンやマウス操作でも作成できますが、今回は数が多いので、コマンドを使って**カッコよく**一括で作成しましょう！

VSCode の下のパネルに、コマンドを入力するための「ターミナル」を開きます！\
以下のショートカットキーを入力してみてください！

- **Windows の場合**: `Ctrl + j`
- **Mac 場合**: `⌘ (Command) + j`

![ターミナルが表示されている VSCode のサイドバー](/images/nagoya-ai-event-2025-programming-workshop/02_base-setup/05_opened-terminal-panel.png)

お使いのパソコンに合わせて、以下のコマンドをコピーしてターミナルに貼り付け、Enter キーを押してください！

**Windows の場合**

```bash
type nul > game.js
type nul > game-model.js
type nul > ai-fetch.js
type nul > ai-system-prompt.js
type nul > secret.js
```

**Mac の場合**

```bash
touch game.js
touch game-model.js
touch ai-fetch.js
touch ai-system-prompt.js
touch secret.js
```

コマンドを実行すると、一気に左側のファイル一覧に新しいファイルとフォルダが表示されます。これで冒険の準備は完了です！

![作成されたフォルダとファイルが表示されている VSCode のサイドバー](/images/nagoya-ai-event-2025-programming-workshop/02_base-setup/06_created-files-and-folders.png)

:::details 【深く知りたい人向け】コマンドの意味

- `type nul`（Windows）: 空のファイルを作成するコマンド
- `touch`（Mac）: 空のファイルを作成するコマンド

:::

<br />

## 準備完了！

これで基本的なファイル構成が整いました！！！🚀\
次の章では、ゲームを動かすための基盤を作成していきます！

<br />

## 本チャプター終了時点のファイル構成

以下のようなファイル構成になっていれば正解です。\
講座に追いつけなくなった場合は、この構成を参考にファイルを作成してください。

```
.
├── ai-fetch.js
├── ai-system-prompt.js
├── game.js
├── game-model.js
└── secret.js
```
