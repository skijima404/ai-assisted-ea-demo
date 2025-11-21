---
id: an-211
date: 2025-11-07
title: transactions-should-adopt-two-phase-migration
status: draft
tags: [analysis, transactions, migration, testing]
---

# Transactions should adopt two-phase migration

## Incident (Observed Behavior)
- Baseline における "利用明細相当" の構造は、
  - コンビニ側の `convini_receipt_items`（line-item のみ）、
  - 社食側の `receipts` / `receipt_items`（2層構造）、
 という非対称な 2 系統のデータソースとして存在しており、統一された `transactions` ドメインはまだ存在しない。
- Target Architecture では、社食＋コンビニ双方の利用履歴を統合した `transactions` を新規に定義し、そこから `bills`（請求台帳）をロールアップする責務を持たせる必要がある。
- 移行時に "旧と新で同じ transactions を比較する" という単純なリグレッションテストは成立せず、**いきなりフルスコープで新 transactions に切り替えると検証困難性とリスクが高い** ことが分かった。

## Conclusion / Assumptions
- **結論:** 新しい `transactions` ドメインは、一度にフルスコープで移行するのではなく、
  1. コンビニ由来の line-item 部分（`convini_receipt_items` 相当）の移行・検証、
  2. 社食側 `payments` からの肩代わり部分の移行・検証、
  という **二段階の移行パス** で段階的に構築・検証するべきである。
- **Assumptions:**
  - コンビニ側 line-item は Baseline と Target の構造差分が比較的小さく、CSV を "鏡" として利用した Apple-to-Apple 検証が可能である。
  - 社食側 `payments` から引き継ぐ情報は、line-item 構造の検証が完了した後であれば、単純な同期ロジックで移行できる。
  - データロス検証には、既存 CSV と人事側の現行請求台帳を Oracle（鏡データ）として活用できる。

## Reasoning
1. **Baseline には統一的な transactions が存在せず、構造的互換性がない。**  
   - `convini_receipt_items` は line-item のみで割引やレシートヘッダを保持しておらず、  
     `receipts` / `receipt_items` は社食側の完全な決済構造を持つ。  
   - Target で定義される `transactions` は、これらを統合する新しいドメインであり、Baseline と 1:1 で比較できない。

2. **検証の観点から、もっとも単純な部分（コンビニ line-item）を先に移行するのが妥当。**  
   - コンビニ側は現在も CSV を起点とした運用であり、長期運用によるデータロスが確認されていない。  
   - この CSV を Oracle として利用することで、"新 transactions のコンビニ部分" を Baseline と比較的ストレートに検証できる。

3. **社食側 payments 由来の情報は、line-item 部分が安定した後に追随させる方が安全。**  
   - 現状、社食側 `payments` は "人事側で持つべき天引き情報" を肩代わりしており、その責務をどこまで `transactions`／`bills` に分配するかは設計上の論点である。  
   - 先にコンビニ側 line-item → 新 transactions の経路を固定し、その後に payments → transactions の単純コピー／同期ロジックを設計した方が、論点を切り分けやすい。

4. **二段階移行にすることで、テスト設計と監視のスコープも段階的に増やせる。**  
   - 第1段階：コンビニ側 line-item 部分のみを新 transactions に流し、CSV／人事請求台帳との整合性を重点的に監視。  
   - 第2段階：社食側 `payments` 由来のデータを新 transactions に統合し、最終的な `bills` ロールアップとの一貫性を検証する。

## Implication
- `transactions` を一度に "作り直す" のではなく、**構成要素ごとに移行と検証を分離する設計** が必要となる。  
- 移行計画（Transition Architecture）では、
  - コンビニ line-item 移行フェーズ  
  - payments 連携統合フェーズ  
  を明示的に分けて定義し、それぞれに専用の検証指標（データロス率、同期遅延、請求台帳との差分など）を設定するべきである。  
- ADR では「transactions は二段階で成立させる」「検証はコンポーネントごとに行う」という前提を記録し、単一ステップ移行の期待値を早期に潰しておく必要がある。

## Observations
- Baseline の利用明細相当:
  - コンビニ: `convini_receipt_items`（line-item のみ）
  - 社食: `receipts` / `receipt_items`（2層構造）
- Target の transactions:
  - 社食＋コンビニを統合した canonical 利用明細ドメイン
- 段階移行:
  - 先にコンビニ側 line-item → 新 transactions  
  - 後から社食側 `payments` → 新 transactions へ役割移管

## Evidence
- `01_working/ef_solutions/10_raw-notes/2025-11-07-transactions-deep-dive.md`
- `01_working/ef_solutions/10_raw-notes/2025-11-07-service-dependencies.md`
- `01_working/bd_arch-design/15_raw-notes/2025-11-06-convini-receipt-items.md`
