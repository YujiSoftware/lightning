---
marp: true
author: "@YujiSoftware"
title: "Javaで絵文字を正しく扱おう🥲"
description: "JJUG CCC 2026 Spring のセッション資料です。"
image: "https://yuji.software/emoji/img/card.png"
theme: gaia
style: |
  :root {
    --color-background: #fff;
    --color-foreground: #000;
    --color-highlight: #1a0dab;
    --color-dimmed: #888;
    font-size: 40px;
    line-height: 1.2;
  }

  /** 詰め気味に微調整 **/
  section > ul, marp-pre {
    margin: 0.5em 0;
  }
  h1 {
    margin-bottom: 0.5em;
  }

  section.lead img {
    border-radius: 50%;
    vertical-align: middle;
  }
  
  img[alt~="center"] {
    display: block;
    margin: 0 auto;
  }

  .info {
    border: 2px solid #dee2e6;
    background-color: #cff4fc;
    color: #052c65;
    padding: 0.5em;
    border-radius: 10px;
    margin: 10px 0;
  }
  .info::before {
    content: "📝";
  }

  span.emphasis {
    color: red;
    font-weight: bold;
  }
  span.underline {
    text-decoration: underline;
  }
  span.supplement {
    font-size: 36px;
  }

  div.balloon {
    display: flex;
    align-items: center;
  }

  div.balloon-right {
    justify-content: end;
  }
  div.balloon-right img {  
    border-radius: 50%;
  }
  
  div.balloon div {
    position: relative;
    display: inline-block;
    margin: 0.5em;
    padding: 1em;
    border-radius: 8px;
    text-align: left;
    line-height: 1.5;
    color: #000000;
  }
  div.balloon-right div {
    background-color: #A5FBFF;
  }
  div.balloon-left div {
    background-color: #FFDFDF;
  }

  div.balloon div::after {
    content: "";
    position: absolute;
    top: 50%;
    border-style: solid;
  }
  div.balloon-right div::after {
    right: 0;
    border-width: 10px 0 10px 20px;
    border-color: transparent transparent transparent #A5FBFF;
    translate: 100% -50%;
  }
  div.balloon-left div::after {
    left: 0;
    border-width: 10px 20px 10px 0;
    border-color: transparent #FFDFDF transparent transparent;
    translate: -100% -50%;
  }

  img.icon {
    border-radius: 50%;
    vertical-align: middle;
  }
  code {
    font-size: 1em !important;
  }
  .bold {
    font-weight: bold;
  }
  .yake {
    font-size: 90px;
    color: red;
  }

---

<!-- _class: lead -->

# Javaで絵文字を正しく扱おう🥲
## LT バージョン
<br>

![width:150px height:150px](../photo.jpg) @YujiSoftware

---

# このコードを実行した結果は？

- Java 21 で **isEmoji メソッド**が追加された
    - 実際に Java 21 で動かすとなにが返ってくる？
```java
jshell> Character.isEmoji('🤧');
```

---

# このコードを実行した結果は？

- Java 21 で **isEmoji メソッド**が追加された
    - 実際に Java 21 で動かすとなにが返ってくる？
```java
jshell> Character.isEmoji('🤧');

$1 ==> false
```

---

# AI に聞きました（擬人化）

<div class="balloon balloon-right">
  <div>
    なんで false になるの？
  </div>
  <img src="../photo.jpg" width="150px" height="150px">
</div>

---

# AI に聞きました（擬人化）

<div class="balloon balloon-right">
  <div>
    なんで false になるの？
  </div>
  <img src="../photo.jpg" width="150px" height="150px">
</div>

<div class="balloon balloon-left">
  <img src="duke.png" width="150px" height="150px">
  <div>
    それバグです！<br>Java 25 で修正しました！
  </div>
</div>

---

# 確認

- 改めて Java 25 で実行

```java
jshell> Character.isEmoji('🤧');
```

---

# 確認

- 改めて Java 25 で実行

```java
jshell> Character.isEmoji('🤧');

|  エラー:
|  文字リテラルに複数のUTF-16コード・ユニットが含まれています
|  Character.isEmoji('🤧');
|                    ^
```

---

# AI に聞きました（擬人化）

<div class="balloon balloon-right">
  <div>
    バグってるのですが…
  </div>
  <img src="../photo.jpg" width="150px" height="150px">
</div>

---

# AI に聞きました（擬人化）

<div class="balloon balloon-right">
  <div>
    バグってるのですが…
  </div>
  <img src="../photo.jpg" width="150px" height="150px">
</div>

<div class="balloon balloon-left">
  <img src="duke.png" width="150px" height="150px">
  <div>
    そのとおり
  </div>
</div>

---

# AI に聞きました（擬人化）

<div class="balloon balloon-right">
  <div>
    バグってるのですが…
  </div>
  <img src="../photo.jpg" width="150px" height="150px">
</div>

<div class="balloon balloon-left">
  <img src="duke.png" width="150px" height="150px">
  <div>
    そのとおり<br>
    <span class="bold">あなたのコードがバグっています！</span>
  </div>
</div>

---

# 解説

- **エラーになるのが正解 ⭕**
  - 文字リテラルに、サロゲートペアを入れてはいけない
- Java 24 までのコンパイラの処理が間違い
  - 上位サロゲートのみに切り捨てていた

```java
Character.isEmoji('🤧');
                   ↓
Character.isEmoji(\uD83E\uDD27);
                   ↓
Character.isEmoji(\uD83E); ←不正な文字 = 絵文字じゃない
```

---

# 正しいコード

- 文字列から**コードポイント**を取り出しましょう
```java
jshell> Character.isEmoji("🤧".codePointAt(0));

$1 ==> true
```

---

# このコードを実行した結果は？

- Java 21 で **isEmoji メソッド**が追加された
    - '0' を渡すとなにが返ってくる？
```java
jshell> Character.isEmoji('0');
```

---

# このコードを実行した結果は？

- Java 21 で **isEmoji メソッド**が追加された
    - '0' を渡すとなにが返ってくる？
```java
jshell> Character.isEmoji('0');

$1 ==> true
```

---

# AI に聞きました（擬人化）

<div class="balloon balloon-right">
  <div>
    なんで true になるの？
  </div>
  <img src="../photo.jpg" width="150px" height="150px">
</div>

---

# AI に聞きました（擬人化）

<div class="balloon balloon-right">
  <div>
    なんで true になるの？
  </div>
  <img src="../photo.jpg" width="150px" height="150px">
</div>

<div class="balloon balloon-left">
  <img src="duke.png" width="150px" height="150px">
  <div>
    Unicode が絵文字だって言ってる！<br>（しらんがな）
  </div>
</div>

---

# Unicode の定義を確認

```
0023        ; Emoji  # E0.0   [1] (#️)     hash sign
002A        ; Emoji  # E0.0   [1] (*️)     asterisk
0030..0039  ; Emoji  # E0.0  [10] (0️..9️)  digit zero..digit nine
00A9        ; Emoji  # E0.6   [1] (©️)     copyright
00AE        ; Emoji  # E0.6   [1] (®️)     registered
```

- 数字は絵文字、だと？？？

---

# なぜ？

- **絵文字シーケンス**により、絵文字となるから
  - ↑ 複数のコードポイントを組み合わせて、1つの絵文字とする仕組み

---

# 具体例

- 0️⃣ （キーキャップ 0）という絵文字
  - **U+0030**：数字の 0
  - U+FE0F：異体字セレクター16（絵文字表示の指定）
  - U+20E3：キーキャップを表す合成文字（□） 

- 1つの絵文字に3つのコードポイント

---

# 絵文字シーケンス

- 新しい絵文字はだいたいこれ
  - ❤️‍🩹: U+2764, U+FE0F, U+200D, U+1FA79
  - 🫱🏻‍🫲🏼: U+1FAF1, U+1F3FB, U+200D, U+1FAF2, U+1F3FC
  - 🧎🏻‍♀️‍➡️: U+1F9CE, U+1F3FB, U+200D, U+2640, U+FE0F, U+200D, U+27A1, U+FE0F 
  - 🧑🏻‍❤️‍💋‍🧑🏼：U+1F9D1, U+1F3FB, U+200D, U+2764, U+FE0F, U+200D, U+1F48B, U+200D, U+1F9D1, U+1F3FC
- どう扱えばいいの！？？？

---

# 詳しく知りたい方は

- 本日のセッションで！
  - ROOM F にて 10:25〜

---

# 詳しく知りたい方は

- 本日のセッションで！
  - ROOM F にて 10:25〜

<div class="balloon balloon-left">
  <img src="duke.png" width="150px" height="150px">
  <div>
    もう終わってるよ！
  </div>
</div>

---

# まとめ


---

# まとめ

* コンパイラですら間違える
  - 私達が正しく扱えるわけない

* エディタですら間違える
  - この資料を作る際にバグを踏みました

* <span class="yake">絵文字はめんどくさい！</span>

---

<!-- _class: lead -->

# Javaで絵文字を正しく扱おう🥲
## LTバージョン
<br>

![width:150px height:150px](../photo.jpg) @YujiSoftware
