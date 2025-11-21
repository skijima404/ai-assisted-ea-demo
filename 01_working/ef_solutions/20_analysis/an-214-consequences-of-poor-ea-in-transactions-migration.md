
---
id: an-214
date: 2025-11-07
title: consequences-of-poor-ea-in-transactions-migration
status: draft
tags: [analysis, risk, migration, transactions]
---

# Consequences of Poor EA in Transactions Migration

## Incident (Observed Behavior)
- Transactions移行でEA判断が不十分な場合、以下のような **重大インシデント** が発生し得ることが明確になった。
- 特に、イベント駆動移行後の到達保証・冪等性不足、Billsとの検算ロジック不備、監査証跡の欠落が事業継続を脅かす。

## Conclusion / Assumptions
- **結論:** EAが移行時の“データ構造の断面”と“仕様上の境界線”を十分に定義しない場合、給与天引き領域で致命的事故を誘発し、信頼・監査の両面で回復不能なダメージとなる。
- **Assumptions:**
  - BaselineではCSVが「鏡」として機能しデータロスが起きにくかった。
  - Targetではイベント駆動化・分散化のため、**小さな設計ミスが大きな不整合につながる**。
  - Billsはイベントドリブン化により、Cutoffや集計粒度の厳密化が不可欠になる。

## Reasoning
1. **BaselineとTargetのTransactionsは構造が異なる**  
   - Baseline:  
     - 社食レシート（`receipts` / `receipt_items`）とコンビニ明細（`convini_receipt_items`）は別管理。  
     - BillsはCSV経由で正しい。  
   - Target:  
     - 社食＋コンビニの合算Transactionsが新規構築され、Billsはこれに依存する。  
   - よって「計算の基準」が変わるため、EAが粒度の統一と整合性ルールを定義しなければ事故が起きる。

2. **イベントドリブン移行では“到達保証”が本質になる**  
   - CSVはデータロスの蓋然性が極めて低い。  
   - しかしイベントでは、Retry・DLQ・冪等性が弱いと **サイレントデータロス** が起きる。  
   - 給与天引きが絡む領域では致命傷。

3. **BillsはTransactionsへの依存度が上がり、Cutoff問題が顕在化する**  
   - BaselineではCSVが“月ごとの正”だった。  
   - Targetではイベント時刻か処理時刻かのどちらを基準に集計するか定義が必要。  
   - EAが定義しなければ月次締めの数字が永遠に揃わない。

4. **監査証跡は正規化の過程で失われやすい**  
   - `original_receipt_id` を保持しない限り、問合せ対応不可となる。  
   - 分散アーキテクチャでは「どこで失われたか」すら追えなくなる。

## Implication
- EAの判断が弱いと、以下の“事業継続を脅かす”事故が現実化する：
  - サイレントデータロスによる給与天引き漏れ  
  - 冪等性欠如による二重請求  
  - Cutoffのズレにより新旧数値が永遠に一致しない  
  - 監査証跡喪失によるコンプライアンス違反  
- そのため、**移行前にEAが「データ粒度」「整合条件」「到達保証」「イベント順序」「Bills集計ルール」など、移行の“見えない前提”を全て定義する必要がある。**

## Observations
- BaselineとTargetでTransactionsの構造が異なる  
- BillsがTargetではEvents依存となり難易度が上昇  
- 冪等性・到達保証が欠けるとCSV運用時には現れなかった事故が出る  
- 支払い関連データは監査・説明責任の要求が強い

## Evidence
- `2025-11-06-baseline-transactions-analysis.md`
- `2025-11-07-transactions-deep-dive.md`
- `2025-11-07-functional-extraction-difficulty.md`
