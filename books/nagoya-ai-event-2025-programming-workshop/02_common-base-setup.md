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

**PowerShell(Windows) の場合**

```powershell
New-Item -ItemType File -Name "ai.js"
New-Item -ItemType File -Name "app.js"
New-Item -ItemType File -Name "game.js"
New-Item -ItemType File -Name "secret.js"
```

:::details コマンドプロンプト(Windows) の場合

```bash
type nul > ai.js
type nul > app.js
type nul > game.js
type nul > secret.js
```

:::

:::details シェル(Mac,Linux) の場合

```bash
touch ai.js
touch app.js
touch game.js
touch secret.js
```

:::

コマンドを実行すると、一気に左側のファイル一覧に新しいファイルとフォルダが表示されます。これで冒険の準備は完了です！

![作成されたフォルダとファイルが表示されている VSCode のサイドバー](/images/nagoya-ai-event-2025-programming-workshop/02_base-setup/06_created-files-and-folders.png)

:::details 【深く知りたい人向け】コマンドの解説

**PowerShell**

- `New-Item -ItemType File`: 新しいファイルを作成するコマンド
  - `-ItemType File`: ファイルを作成することを指定
  - `-Name`: 作成するファイル名を指定

<br />

**コマンドプロンプト**

- `type nul >`: 空のファイルを作成するコマンド
  - `type nul`: 何も出力しない
  - `>`: 出力をファイルにリダイレクト（空のファイルが作成される）

<br />

**シェル（Mac/Linux）**

- `touch`: 空のファイルを作成するコマンド
  - ファイルが存在しない場合は新規作成
  - ファイルが存在する場合はタイムスタンプを更新

:::

<br />

## 準備完了！

これで基本的なファイル構成が整いました 🚀\
次の章では、ゲームを動かすための基盤を作成していきます！

<br />

## 本チャプター終了時点のファイル構成

以下のようなファイル構成になっていれば正解です。\
講座に追いつけなくなった場合は、この構成を参考にファイルを作成してください。

```
.
├── ai.js
├── app.js
├── game.js
└── secret.js
```

<br />

## トラブルシューティング

何か問題が発生した場合は、以下を参考にしてください！

:::details A. 「フォルダーを開く」が見つからない

**原因**: VSCode のバージョンや表示言語によっては、UI が若干異なる場合があります。

**対処法**: 左上の「ファイル」メニューから「フォルダーを開く...」を選択するように案内してください。サイドバーのアイコンの意味を簡単に説明し、一番上のエクスプローラーアイコンをクリックするように伝えます。

:::

:::details B. ターミナルが開かない

**原因**: ショートカットキーが他のアプリケーションと競合している、あるいはキーボードの設定が特殊である可能性があります。

**対処法**: VSCode の上部メニューから「表示」 > 「ターミナル」を選択する方法を案内してください。または、「表示」 > 「コマンドパレット...」から `>Terminal: Create New Terminal` を検索して実行する方法も有効です。

:::

:::details C. コマンドを実行してもファイルが作成されない

#### パターン１：コマンドが違う

**原因**: Windows と Mac のコマンドを間違えている。

**対処法**: 生徒さんが使用している OS を確認し、正しい方のコマンドをコピー＆ペーストするように再度案内してください。ターミナルに表示されたエラーメッセージ（`'touch' は、内部コマンドまたは外部コマンド、操作可能なプログラムまたはバッチ ファイルとして認識されていません。` など）を確認させ、コマンドが違う可能性を指摘します。

<br />

#### パターン２：場所が違う

**原因**: コマンドを実行するディレクトリが違う。

**対処法**: VSCode のエクスプローラーで `ai-app` フォルダが開かれているか確認させてください。ターミナルに表示されているパス（プロンプト）が、`ai-app` になっているか確認させます。もし違う場所であれば、`cd` コマンドで移動するか、VSCode で再度正しいフォルダを開き直すように指示します。

<br />

#### パターン３：どうしてもうまくいかない

**原因**: `touch` や `type nul` が使えない環境。

**対処法**: もしコマンドでうまくいかない場合は、無理にコマンドに固執せず、GUI でファイルを作成する方法を案内してください。VSCode のエクスプローラーで、`ai-app` の何もない領域を右クリックし、「新しいファイル」を選択して、`game.js` などのファイル名を一つずつ入力して作成する方法を教えます。

:::
