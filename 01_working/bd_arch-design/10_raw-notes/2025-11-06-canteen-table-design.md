---
date: 2025-11-06
tags: [raw]
---

# Raw Note


## 気づき (Observation)  
社食システム側では、コンビニ側で発生した金額不整合（割引・レシート構造の欠落）が再現しない理由を確認するため、テーブル構造を調査した。

### 社食側テーブル設計（確認結果）

社食システムではレシート情報を2層構造で保持している。

#### テーブル構成
- table: `receipts`
  - `id` (PK)
  - `receipt_no` (string) — レシート番号
  - `employee_code` (string)
  - `purchased_at` (timestamp)
  - `total_amount` (int) — 支払額（税計算はレシート単位で一括処理）
  - `tax_rate` (decimal) — 一律税率（例: 0.10）
  - `tax_amount` (int)
  - `payment_method` (string) — "salary_deduction" | "IC"
  - `created_at` (timestamp)

- table: `receipt_items`
  - `id` (PK)
  - `receipt_id` (FK → receipts.id)
  - `item_name` (string)
  - `unit_price` (int)
  - `qty` (int)
  - `amount` (int)
  - `tax_included` (boolean)
  - `created_at` (timestamp)

#### コメント
- 社食システムでは、明細（`receipt_items`）とレシート全体（`receipts`）を分離する2層構造。
- 社食システムでは、コンビニでの利用金額のズレと同じような事象は発生しない
- システム初期構築時、割引は想定されずに作られており、テーブル設計にも反映されている。

## 疑問 (Question)
- 特になし

## 次アクション (Next Step)
- 特になし