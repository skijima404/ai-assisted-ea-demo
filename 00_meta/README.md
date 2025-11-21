# 00_meta — Repository Governance Guide  
*For GenAI ingestion & reproducible EA reasoning*

このフォルダは、リポジトリ全体の **運用ルール・構造設計・命名規則** を定める「憲法」です。  
ここに書かれた内容は、GenAI と人間の両方がこのリポジトリを正しく扱うための最上位レイヤーです。

---

## 1. Repository Purpose
このリポジトリは、アーキテクトの観察・推論・意思決定を  
**「後から再現できる形」** で記録するための Knowledge Repository です。

目的は以下の3つ：

1. raw-notes → analysis → artifacts → GraphDB という  
   **思考パイプラインの標準化**
2. reasoning（因果構造）の可視化と再現性確保
3. GenAI と協働するための情報構造の最適化

---

## 2. Folder Roles

### `00_meta/`
リポジトリの運用ルール・テンプレート・命名規則。  
**このフォルダがリポジトリの憲法。**

### `01_working/`
調査・観察・試行の作業ログ（非確定情報）。  
LLM ingest を前提としない、生の思考ログ。

- `15_raw-notes/`  
  一次観察ログ。誤観測を含めて削除しない。

- `20_analysis/`  
  raw から抽出された **結論ノード**（Analysis）。  
  GraphDB に載る構造化知識になる前提で書く。

### `03_artifacts/`
顧客提供物・意思決定の最終確定版。  
**EA の SoT（Source of Truth）**。

---

## 3. raw-notes Rules
raw-notes は観察ログであり、事実の“地層”です。  
後から分析ノードを作る際の情報源になります。

**ルール：**

- 観察・事実・背景情報をそのまま書く  
- 推論・結論は書かない（analysis に持っていく）
- 誤観測があっても削除しない
- ただし事実が後で反転した場合は `## Correction` を追記する
- 個人名・機微情報は伏せる or 伏せた形で保持

**Correction の例：**
```
## Correction
- 以下の観察は後の調査で誤りと判明：
  - 「XXXX」
- 正しい内容は analysis-012 で確定（YYYY）
- 理由：ZZZZ
```

---

## 4. analysis Rules
analysis は raw-notes をもとにした **結論ノード** です。  
GraphDB のノードとして扱えるレベルに構造化します。

**1ファイル = 1結論**（One Node, One Logic）。

テンプレートは以下の順序を必須とします：

1. **Incident (Observed Behavior)**  
2. **Conclusion / Assumptions**  
3. **Reasoning**（因果関係を段階で明示）  
4. **Implication**  
5. **Improvement Directions**

追加ルール：

- `analysis-xxx-title.md` の命名規則を使う
- YAML frontmatter の `id` は固定（内容だけ更新）
- conclusion は “確定” ではなく **Provisional Conclusion**
- 最終判断は artifacts フォルダに格納される成果物が SoT
- confidence（★評価）は暫定性の指標

---

## 5. artifacts Rules
artifacts は **意思決定の最終確定版**。

- EA の正式な判断
- アーキテクチャ原則・移行計画・ゴール状態
- ステークホルダー説明資料
- システム再設計の確定版

analysis（結論候補）とは異なり、  
**ここに置かれた内容だけが公式判断**。

---

## 6. Update & Correction Rules

### raw-notes
- 誤りを発見したら削除ではなく Correction を追加
- 断片的な観察ログはそのまま残す

### analysis
- id は固定・内容だけ更新
- 修正が analysis の結論を反転させた場合  
  → raw-notes に Correction を必ず書く

### artifacts
- 稟議後に更新  
- 更新時は関連 analysis のリンクを確認して反映

---

## 7. Naming Rules
- raw-notes:  
  `YYYY-MM-DD-description.md`

- analysis:  
  `analysis-xxx-title.md`

- artifacts:  
  `artifact-architecture-vision.md`  
  `artifact-transition-architecture.md`  
  など目的別に明確化

- GraphDB ノード用 ID は YAML frontmatter に記載  
  ```
  id: an-001
  ```

---

## 8. GenAI Usage Guidelines
GenAI と協働する前提で、書き方にルールを設けています。

**analysis は“GenAI ingest 前提”で書く。**  
**raw-notes は ingest しない（または限定）**。

- 因果関係は曖昧にせず、ステップ化して書く  
- 専門用語は定義を書く or 明確に使う  
- 結論の論理構造が一目でわかるように  
- raw/notebooks はノイズが多いため、直接渡さないこと

テンプレートには必ず GenAI への注意事項を含める。

---

## 9. Governance
- PR はテンプレート使用を必須  
- analysis 追加時は issue or observation source を明記  
- artifacts 更新時は、関連 analysis を参照して反映  
- raw-notes の直接編集は Correction を除き不可  
- リポジトリ構造改変は PR + 承認を必須

---

以上が meta（ガイドライン）です。  
このフォルダは随時アップデートし、リポジトリの変化に追随させます。
