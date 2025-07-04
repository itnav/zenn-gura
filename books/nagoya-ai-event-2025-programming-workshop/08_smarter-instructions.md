---
title: '第7章：もっと賢い指示を出そう'
---

## もっと賢い指示を出そう！

簡単な指示ができるようになったら、もっと賢い指示を考えてみましょう！\
もっと賢い指示を出すことで、ロボットがより正確に動くようになります！

<br />

## 周りを見る力を教えよう

今までは「まっすぐ進んで」「右に曲がって」という単純な指示でした。でも、もっと賢くロボットを動かすには、「周りを見る」ことを教える必要があります。

<br />

## コードを記述する

では、周りを見る指示を書いてみましょう。

```javascript:./map-1/ai-route-prompt.js
export const routePrompt = `
    緑のリンゴを見つけたら、その方向に進んでください。
    黒い罠を見つけたら、それを避けて別の道を探してください。
    赤いゴールが見えたら、そこに向かってください。
`;
```

どうですか？「見つけたら」という条件を使うことで、ロボットが自分で判断できるようになりました！\
これで、より賢い指示ができるようになりました！

## 順番を教えよう

複雑な迷路では、順番が大切です。「まず」「次に」「最後に」という言葉を使って、順番を教えてみましょう！

```javascript:./map-1/ai-route-prompt.js
export const routePrompt = `
    1. まず、右に進めるだけ進んでください
    2. 壁にぶつかったら、上に進んでください
    3. 黒い罠があったら、左に回り道をしてください
    4. 最後に、赤いゴールを目指してください
`;
```

番号を使うと、AIが順番通りに実行してくれます。どこで失敗したかも分かりやすくなりますね！\
これで、より賢い指示ができるようになりました！

<br />

## 条件を教えよう

「もし〜なら」という条件も使えます。これを使うと、さらに賢い判断ができるようになります！

---

> AIが条件分岐を判断している様子を示すフローチャート風の図解

---

```javascript:./map-1/ai-route-prompt.js
export const routePrompt = `
    もし目の前に黒い罠があったら、右か左に避けてください。
    もし行き止まりになったら、来た道を戻って別の道を探してください。
    もし赤いゴールが見えたら、最短距離で向かってください。
`;
```

これで、いろいろな状況に対応できるロボットになりました！\
これで、より賢い指示ができるようになりました！

:::details 【深く知りたい人向け】プログラミングの基本概念
今、みなさんは重要なプログラミングの考え方を学びました：

1. **観察**（見る）→ 状況を把握する
2. **手順**（順番）→ 順序立てて考える
3. **分岐**（条件）→ 状況に応じて判断する

これらは、どんなプログラミング言語でも使われる基本的な考え方です！
:::

## 組み合わせてみよう

学んだことを全部組み合わせて、最強の指示を作ってみましょう！

```javascript:./map-1/ai-route-prompt.js
export const routePrompt = `
    【目標】赤いゴールまで安全に到達する

    【ルール】
    1. 黒いトラップは絶対に踏まない
    2. できるだけ最短ルートを選ぶ

    【手順】
    1. スタート地点から右方向を確認
    2. 進める道があれば進む
    3. 黒いトラップがあれば避ける
    4. 行き止まりなら別の方向を試す
    5. 赤いゴールが見えたら直進

    【注意】
    - 慎重に進むこと
    - 周りをよく見ること
`;
```

すごい！まるでロボットの取扱説明書みたいですね。こんなに詳しく書けば、きっとゴールまでたどり着けるはずです！\
これで、より賢い指示ができるようになりました！
