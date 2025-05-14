---
marp: true
theme: gaia
_class: lead
paginate: true
backgroundColor: 
backgroundImage: url('https://marp.app/assets/hero-background.svg')
style: |-
  // 全体のcssスタイル
  .hoge {
    border: solid 1px;
    text-align: right;
  }
---
<style scoped>
  .hoge2 {
    //border: solid 1px;
    text-align: right;
  }
</style>

# 🧠 AIチャット × Markdown  
## 最強の相性！

<div class="hoge2">2025/4/16 梅本</div>

---

## 📌 なぜMarkdown？

| 特徴 | AIとの相性が良い理由 |
|------|------------------------|
| **シンプルな記法** | 読みやすく、AIが正確に解釈しやすい |
| **構造化された表現** | 見出し・リスト・表などで論理構造が明確になる |
| **コードブロック対応** | プログラムや数式をそのまま記述でき、コピペも楽 |
| **互換性の高さ** | GitHubやNotionなど多くのツールでそのまま使える |
| **AI出力との親和性** | AIがMarkdown形式で返すことで読みやすさUP |

---

## 📊 Markdown出力が役立つシーン

- 📝 資料作成
  * そのままMarpスライドに
  * Latexへの変換もスムーズ！
- 📚 ノート整理（Obsidianに貼るだけ）
- 🔢 数式を使った説明（LaTeX数式）

---

## 💡 Markdown出力をAIに頼むコツ

- 「**Markdown形式で**」「**コードブロックで**」など明示する
- 数式は `$...$` や `$$...$$` で囲もう
- Marp形式ならスライド作成も楽々！

---
<style scoped>
  .columns {
    display: flex;
  }
  .column1 {
    width: 40%;
    padding-right:50px;
  }
  </style>

## ✍️ 具体例：制御工学の安定性評価

<div class="columns">
<div class="column1">

### ユーザーの質問

`$\dot{x} = Ax + Bu$`の安定性を評価する方法を教えてください。 LaTeX数式つきで説明して。

</div>
<div class="column2">

### 🤖 AIの返答（例）

1. システム：$\dot{x} = Ax$

2. Lyapunov候補関数：$V(x) = x^T P x$
ただし、$P > 0$

このとき、
$$
\dot{V}(x) = x^T (A^T P + P A) x
$$
が負定であれば、原点は漸近安定。
</div>
</div>

---
<style scoped>
  .columns {
    display: flex;
  }
  .column1 {
    width: 60%;
    padding-right:50px;
  }
  .column2 {
    width: 40%;
    padding-right:50px;
    margin-top: 10px;
  }
  </style>

## 🧩 Mermaid.js で図を描こう！

<div class="columns">
<div class="column1">

**図・フロー**

```mermaid
graph TD
  A[開始] --> B{条件分岐}
  B -->|Yes| C[処理1]
  B -->|No| D[処理2]
  C --> E[終了]
  D --> E
```

</div>
<div class="column2">

![生成画像](https://kroki.io/mermaid/svg/eNpLL0osyFAIceFSUHCMfjmt--ny7lgFXV07BafqZ3MXPtm97WlH29NNE2qB8k4g8ZrI1OIaBefop-3Lnk9oM4yFi_vl1yi4QIWNQMLOYGNco59vbXqyqw0k4gIRAQCTrixB)

</div>
<div class="column3">


- Markdownに直接書ける
- Obsidian・Marp・GitHubでも使える
- フローチャート、シーケンス図など対応

</div>
</div>

---
<style scoped>
  .columns {
    display: flex;
  }
  .column1 {
    width: 60%;
    padding-right:50px;
  }
  .column2 {
    width: 40%;
    padding-right:50px;
    margin-top: 50px;
  }
  </style>

<div class="columns">
<div class="column1">

## 🛠 工程表も！


```mermaid
gantt
  title プロジェクト工程表
  dateFormat  YYYY-MM-DD
  section 準備
  要件定義     :a1, 2025-04-15, 3d
  設計         :after a1, 4d
  section 実装
  開発         :2025-04-20, 5d
  テスト       :2025-04-25, 3d
```


</div>
<div class="column2">

- `gantt` ブロックでガントチャートも簡単作成！
- スケジュール管理にも最適

![生成画像](https://kroki.io/mermaid/svg/eNpLT8wrKeFSUCjJLMlJVXjcPP1x89rHTTseNy1_3LT-cXPH0-1Ln6_ofrFwBVBNSmJJqlt-UW5iiYJCJBDo-vrqurgAJYpTk0sy8_MUnu2a9rRpJlDgxbLGJ7u3PV036_m-lQogYJVoqKNgZGBkqmtgomtoqqNgnAJStmLtixUdCjBglZhWklqkAFJqkoJk7NN1818sbgUKvJzW_XzmLoR6mIFGBjoKpiAdj5vbHjftBDobQwXYSgDhiVhO)

</div>

</div>





---

## 🚀 まとめ

### ✅ AIチャットとMarkdownは…

- 情報整理に最適
- コーディングとの相性抜群
- あらゆる場面で応用できる！

---
<style scoped>
  .hoge3 {
    //border: solid 1px;
    text-align: center;
  }
</style>
## 🎁 おまけ：このスライドもMarkdownだけで作られています！

> Marpを使えば、**AIの出力をそのままプレゼン資料に！**

つまり・・・
<div class="hoge3">

## **Markdownは使えるとすごく便利！**

そしてすぐ使えるようになる！
</div>

---

## Marpに限り注意あり

HTMLとCSSがわからないと、うまく調整ができません。Markdownだけでは厳しいときもある。

HTML、CSSはシンプルなのでAI Chatに聞きながら書けばよい。

---
