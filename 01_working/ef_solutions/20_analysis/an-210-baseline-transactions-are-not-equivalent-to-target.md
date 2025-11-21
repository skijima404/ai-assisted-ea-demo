---
id: an-210
date: 2025-11-07
title: baseline-transactions-are-not-equivalent-to-target
status: draft
tags: [analysis, transactions, migration]
---

# Baseline transactions are not equivalent to Target transactions

## Incident (Observed Behavior)
- Baseline の社食システムには厳密な `transactions` ドメインは存在せず、  
  - 社食側の `receipts` / `receipt_items`  
  - コンビニ側の `convini_receipt_items`  
 という **非対称な 2 系統のデータソース**を寄せ集めて利用明細相当の役割を実現している。
- Target Architecture では「社食＋コンビニの統合利用明細」を提供する必要があり、Baseline の構造とは一致しない。  
- そのため “同じ処理結果を再現できるか” を単純比較で検証することができず、移行検証の難易度が上がっている。

## Conclusion / Assumptions
- **結論:** Baseline の「利用明細に相当するデータ構造」は Target の transactions とは **意味的に互換性がない**。そのため、Target での正当性検証は「構造の一致」ではなく「データロスがないこと」「同期が十分であること」など別軸で行う必要がある。
- **Assumptions:**  
  - Baseline の `receipts` / `receipt_items` は Target の `payments` に対応する。  
  - コンビニ側 `convini_receipt_items` は Target の transactions の一部要素に対応するが、レシート構造が欠落しており完全な再現は不可能。  
  - Target の transactions は「社食＋コンビニの決済記録を統合」するため、新しい責務を持つ。

## Reasoning
1. **Baseline のデータ構造が非対称であるため、統一的な “transactions” が存在しない。**  
   - 社食側は 2 段構造（レシート + 明細）。  
   - コンビニ側は明細のみ（割引・合計計算不可）。  
   - → 同じ構造に正規化できない。

2. **Target Architecture では “統合利用明細” を提供する必要がある。**  
   - Baseline の時点で社食側・コンビニ側で役割が分断されている。  
   - → アプリケーション責務の再設計が必要。

3. **同じ処理結果を比較するリグレッションが成立しない。**  
   - Baseline と Target が「同じ仕様ではない」ため、アウトプットの一致は評価軸として不適切。  
   - → データロス検知、同期エラー検証、到達保証検証の方が重要。

4. **検証の基準を “構造一致” から “安全な移行の保証” に切り替える必要がある。**  
   - CSV や現行請求台帳など、Baseline の鏡として扱えるデータを中心に検証する方が合理的。

## Implication
- “同じ transactions を作り直す” という前提は誤りであり、  
  Target は **新しい役割と統合モデルを持つ別システム**として扱うべき。  
- 検証設計では、  
  - データロスなし  
  - リトライ保証  
  - 段階移行（まずコンビニ → 次に社食）  
  の観点が必要となる。  
- ADR では「Baseline-equivalence を求めない」という前提を明記すべき。

## Observations
- Baseline transactions = receipts / receipt_items（社食） + convini_receipt_items（コンビニ）
- Target transactions = 統合利用明細（社食＋コンビニ）
- Baseline と Target は構造も責務も一致しない

## Evidence
- `01_working/ef_solutions/10_raw-notes/2025-11-07-transactions-deep-dive.md`
- `01_working/bd_arch-design/15_raw-notes/2025-11-06-receipts-table-design.md`
