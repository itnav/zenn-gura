---
title: はじめに
---

## 概要

この本は、2024 年の 7 月 13 日と 7 月 14 日に開催される [NAGOYA TEENS AI CAMP](https://nt-aicamp.com) の B コースの講習で使用する教材になります。

今回の講習では、ChatGPT API を組み込んだ「お料理レシピ生成アプリ」の作成を体験することで、AI を活用した Web アプリケーション開発についての知識を深めることを目的としています。

## この講習を通して学べること

1. **HTML と CSS の基本**

   - アプリケーションの骨組みの作成や見た目のスタイリングを通じて、HTML と CSS の基本的な使い方を学びます。

2. **ChatGPT API の利用方法**

   - ChatGPT API を使って、ユーザーの入力に基づいた応答を生成する方法を学びます。

3. **Web アプリケーションを作成する際の実践的な JavaScript の使い方**

   - ChatGPT API をアプリケーションに組み込むことで、JavaScript を使ったデータの取得や表示方法を学びます。

## 進め方

「お料理レシピ生成アプリ」を作成するにあたり、基本的にコードは **コピペ** で問題ありません。

これは、この講習が具体的な実装よりも概念や構造の理解を重視しているためです。
「Web アプリケーションはどうやって作るのか」や「Web アプリケーションはどう構成されているのか」にフォーカスを当て、コードを概念や構造の理解を深めるための手段として使用します。

そのため、HTML、CSS、JavaScript の書き方に関する解説は少なめなのでご了承ください。

## 開発環境

この講習では、以下の開発環境を使用します。\
あらかじめご用意いただけるとスムーズに進めることができます。

### 🍀 必須環境

#### 1. VSCode

[VSCode](https://code.visualstudio.com) は、Microsoft が提供する無料のコードエディタです。\
HTML、CSS、JavaScript などの Web 開発にも最適なエディタで、多くの拡張機能が提供されています。

#### 2. お好きな Browser の最新版

お好きなブラウザの最新版をご用意ください。

##### おすすめ

- **Window**: Microsoft Edge, Google Chrome
- **Mac**: The Browser Company Arc, Google Chrome

### 🌱 推奨環境

#### 1. VSCode の Live Server 拡張機能

ローカルサーバーを立ち上げて、Web ページの変更を確認できる拡張機能です。\
ファイルを保存すると自動でリロードされるので、開発効率が向上します。

[VSCode Live Server 拡張機能](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) から、Install ボタンをクリックするとインストールできます。

VSCode 右下の「Go Live」をクリックすることで、`http://localhost:5500` が開けるようになります。

:::message
アプリケーション自体は Live Server をインストールせずとも動作しますが、本教材では、動作確認をする際に Live Server を使用しています。
:::

#### 2. VSCode の Prettier 拡張機能

コードのフォーマットを自動で整えてくれる拡張機能です。

[VSCode Prettier 拡張機能](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) から、Install ボタンをクリックするとインストールできます。

#### 3. VSCode の Material Icon Theme 拡張機能

ファイルアイコンをカラフルに表示してくれる拡張機能です。フォルダ・ファイルがとても見やすくなります。

[VSCode Material Icon Theme 拡張機能](https://marketplace.visualstudio.com/items?itemName=pkief.material-icon-theme) から、Install ボタンをクリックするとインストールできます。

#### 4. VSCode の Japanese Language Pack 拡張機能

VSCode の日本語化拡張機能です。VSCode を日本語で使用することができます。

[VSCode Japanese Language Pack 拡張機能](https://marketplace.visualstudio.com/items?itemName=MS-CEINTL.vscode-language-pack-ja) から、Install ボタンをクリックするとインストールできます。

#### 5. VSCode の Code Spell Checker 拡張機能

VSCode のスペルチェック機能を有効にする拡張機能です。英語のスペルミスを防ぐのに役立ちます。

[VSCode Code Spell Checker 拡張機能](https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker) から、Install ボタンをクリックするとインストールできます。

#### 6. VSCode の ES6 String HTML 拡張機能

ES6 のテンプレートリテラルを使用して HTML を記述する際に、シンタックスハイライトを有効にする拡張機能です。

[VSCode ES6 String HTML 拡張機能](https://marketplace.visualstudio.com/items?itemName=Tobermory.es6-string-html) から、Install ボタンをクリックするとインストールできます。
