---
title: 【開発】下準備
---

さっそく、アプリケーションを開発するための下準備を行いましょう。

## 1. 空ディレクトリの作成

まずは、アプリケーションの開発に使用するディレクトリ（フォルダ）を作成します。\
ディレクトリの名前はお好みで構いませんが、ここでは `recipe-ai-app` という名前で作成します。

## 2. VSCode で作成したディレクトリを開く

作成したディレクトリを VSCode で開きます。\
VSCode でディレクトリを開くには、以下の手順を参考にしてください。

1. VSCode を起動
2. 左上メニューの「ファイル」 > 「フォルダーを開く...」をクリック
3. 作成したディレクトリを選択して「開く」をクリック

## 3. ファイルの作成

次に、アプリケーションの開発に必要なファイルを作成します。\
作成するファイルは以下の通りです。

- `index.html`
- `style.css`
- `script.js`

ファイルの中身は空でも問題ありません。後ほど、ファイルの中身を追記していきます。

Material Icons Theme 拡張機能をインストールしている場合、ファイルアイコンがカラフルに表示されます。

## 4. VSCode 設定ファイルの作成

最後に、VSCode の設定ファイルを作成します。

1. `.vscode` ディレクトリを作成
2. `.vscode` ディレクトリの中に `settings.json` ファイルを作成

`settings.json` は、以下の内容をコピペします。

```json:./.vscode/settings.json
{
  "editor.tabSize": 2,
  "editor.insertSpaces": true,
  "editor.detectIndentation": false,

  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,

  "prettier.printWidth": 80,
  "prettier.tabWidth": 2,
  "prettier.useTabs": false,
  "prettier.semi": true,
  "prettier.singleQuote": true
}
```

記述後は、忘れずに保存してください！
