---
date: YYYY-MM-DD
tags: [raw]
---

# Raw Note

## 気づき (Observation)  
社内コンビニから受け取ったレシートと、履歴画面の金額がずれている件の原因調査 (`2025-11-06-other-complaints.md` より) 
- 社内コンビニシステムから受け取ったデータが、社食システム内ではレシートのラインアイテムレベルのデータしかない
  - たとえば、「プリン 150円」「おにぎり 100円」相当のデータしかない
  - SQL的にラインアイテムをレシート単位で計算して算出して画面に表現している
  - 合計金額算出後に適用される割引金額が反映されない
  - 結果として社食アプリ上ではレシートの小計 (割引前) が表示され、実際の決済金額 (割引後) と不一致が生じている


### テーブル設計

#### 社内コンビニのレシート連携テーブル
- table: `convini_receipt_items`
- columns:
  - `id` (PK)
  - `receipt_no` (string) — レシート単位でのグルーピングキー
  - `item_name` (string)
  - `price` (int)
  - `qty` (int)
  - `amount` (int) — `price * qty`
  - `discount_applied` (boolean, default false)
  - `created_at` (timestamp)

#### 問題点
- 割引やクーポンはレシート全体に対して適用されるが、`convini_receipt_items` は明細単位でしか記録できない。
- そのため、割引を持つレシートでもテーブル上は小計（合計前）しか保持できず、**決済金額との差分が再現不可能**。
- 本来は `receipts` テーブルと `receipt_items` テーブルの2層構造で表現すべき。
  - 同期元のコンビニシステム側は2層構造のテーブル構成になっているため、社食システム側の実装の問題



## 疑問 (Question)
- 特になし

## 次アクション (Next Step)
- 特になし