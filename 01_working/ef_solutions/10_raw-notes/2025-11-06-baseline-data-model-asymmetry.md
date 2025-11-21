---
date: YYYY-MM-DD
tags: [raw]
---

# Raw Note

## 気づき (Observation)

社食システムと社内コンビニシステムのデータモデルを比較したところ、Baseline の段階で両者の構造が根本的に非対称であることが確認された。

### 社食側（2層構造：Target の payments の原型）
- 社食決済データは `receipts`（レシート単位）と `receipt_items`（明細単位）の2層構造で保持されている。
- 1回の決済を「レシート」として保持し、その内訳となる line items が `receipt_items` に格納される。
- 税額・支払方法・購入日時など、決済としての一貫した情報が揃っており、Target Architecture の `payments` ドメインの要件を満たしている。

### コンビニ側（1層構造：Target の transactions の原型だが不完全）
- `convini_receipt_items` は line item のみを表現するテーブルであり、レシート単位の情報（小計、割引、税額）は保持されていない。
- 割引がレシート全体に適用される場合でも、その情報は Baseline データには一切含まれない。
- そのため、履歴画面では line item を単純加算した“小計（割引前）”しか再現できず、実際の決済金額（割引後）との照合が不可能。

### 結論としての非対称性
- Baseline には「transactions」に該当する正式なドメインは存在せず、
  - 社食：`receipts` / `receipt_items`（決済としての整った2層構造）
  - コンビニ：`convini_receipt_items`（line-itemのみ）
  という **不均一なデータモデルが並列に存在しているだけ** である。
- Target Architecture で定義される `transactions`（利用明細）は、Baseline にはまだ存在しない“これから整えるべきドメイン”である。

## 疑問 (Question)
- Baseline の非対称構造を前提として、移行時にどの時点で対称化（レシート→利用明細→請求台帳）を行うべきか。
- コンビニ側のレシート構造を Target に合わせて再構成する方法はどれが現実的か。

## 次アクション (Next Step)
- フェーズFで、この非対称構造が Transition Architecture に与える影響を整理する。
- payments → transactions → bills への責務分離と対称化のパスを検討する。
- 他の raw-notes（決済不整合・割引ロジックなど）からの参照リンクを追加する。