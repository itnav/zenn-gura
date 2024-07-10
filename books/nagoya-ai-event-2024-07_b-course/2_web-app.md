---
title: Web アプリケーション
---

## Web アプリケーションの構成

![Web アプリケーションの３要素](https://github.com/itnav/zenn-gura/blob/main/books/nagoya-ai-event-2024-07_b-course/assets/2_web-app/three-elements.jpg?raw=true)

Web アプリケーションは、主に HTML、CSS、JavaScript という３つの技術で構成されています。

### HTML

HTML（HyperText Markup Language）は、Web ページの構造を定義するために使用されるマークアップ言語です。\
HTML を使用して、見出し、段落、リンク、画像などの要素をページに追加します。

```html
<div class="container">
  <h1>タイトル</h1>
  <p>本文</p>
</div>
```

### CSS

Cascading Style Sheets の略で、Web ページのスタイルを定義するために使用されます。\
CSS を使用して、レイアウト、色、フォント、間隔などの視覚的なスタイルを指定します。

```css
.container {
  width: 100%;
  margin: 0 auto;
}

h1 {
  color: red;
}
```

### JavaScript

JavaScript は、Web ページの動的な振る舞いを定義するために使用されるプログラミング言語です。\
JavaScript を使用して、ユーザーとのインタラクション、アニメーション、データの操作などを行います。

```javascript
const message = 'Hello, World!';

function logMessage() {
  console.log(message);
}

logMessage();
```

### 建物の例え

![建物で例えた３要素](https://github.com/itnav/zenn-gura/blob/main/books/nagoya-ai-event-2024-07_b-course/assets/2_web-app/three-building-elements.png?raw=true)

これらの３要素は、建物に例えると覚えやすいかもしれません。

- **HTML**

  - 建物の骨組みや構造に相当します。壁や床、天井などの基本的な構造を定義します。

- **CSS**

  - 建物の外観や装飾に相当します。壁の色や床の素材、窓のデザインなどを指定します。

- **JavaScript**

  - 建物の動きや機能に相当します。ドアの開閉や照明の点灯、ガスコンロの点火などの動的な振る舞いを定義します。

## Web サイトとの違い

「Web サイト」と「Web アプリケーション」の違いは何でしょうか？

と聞かれたとき、技術的な違いとかかな？と思い浮かぶ方も少なくないと思います。
ですが、どちらも HTML、CSS、JavaScript で構成する必要があるため、基本的な技術的構成は似ています。

実は、一番の違いは「目的」にあります。

「Web サイト」は情報を提供することが主な目的です。例えば、ニュースサイトや企業の紹介ページがこれに該当します。

一方、「Web アプリケーション」は、ユーザーとの対話を通じて特定のタスクを実行することを目的としています。例えば、オンラインバンキングシステムやメールサービスなどがこれに該当します。

このように、「目的」によって呼び方が変わってしまうため、こんがらがりやすいので注意が必要です。
