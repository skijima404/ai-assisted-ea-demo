---
id: an-212
date: 2025-11-07
title: regression-testing-cannot-rely-on-baseline-equivalence
status: draft
tags: [analysis, testing, regression, migration]
---

# Regression testing cannot rely on baseline equivalence

## Incident (Observed Behavior)
- Baseline の社食システムでは「利用明細（transactions）」が統合ドメインとして存在せず、  
  コンビニ (`convini_receipt_items`) と社食 (`receipts` / `receipt_items`) が非対称な構造で分断されている。  
- Target Architecture では両者を統合した新しい `transactions` ドメインを定義する必要があり、  
  **Baseline と Target の処理結果を 1:1 で比較するリグレッションテストが原理的に成立しない** ことが判明した。
- また、Baseline には「正しい期待値（Expected Result）」が体系化されておらず、  
  現行のリグレッションテストは “画面挙動が合っていること” を指標としているだけで、  
  ビジネスロジックの正しさを担保していない。

## Conclusion / Assumptions
- **結論:** Target の transactions を検証する際、Baseline-equivalence（Baseline と同じ出力を得ること）を  
  リグレッションテストの基準にすることは不適切であり、**別の検証軸**に切り替える必要がある。
- **Assumptions:**
  - Baseline の transactions 相当処理は、構造的・責務的に Target と互換性がない。
  - Baseline の期待値は “正しい状態” を保証していないため、テストの Oracle として利用できない。
  - CSV や現行の請求台帳など、実績ベースでデータロスがないことが確認できるデータのみ Oracle として扱える。

## Reasoning
1. **Baseline と Target の transactions は構造も責務も異なるため、同一性比較は不可能。**  
   - Baseline: コンビニは line-item のみ、社食は 2 層構造で保持。  
   - Target: 両者を統合し、さらに天引きロールアップ（bills 前段）も含む。  
   - → Baseline と Target の “正解” が異なる。

2. **Baseline におけるテスト基準が曖昧であり、期待値として使用できない。**  
   - 現在のリグレッションは “画面が動くこと” が成功条件になっている。  
   - ビジネスロジック自体の要件を満たしているかどうかは保証されていない。  
   - → Target のテストオラクルとして使うと誤判定が発生する。

3. **正しい検証は「構造一致」ではなく「データロス防止 / 同期保証」に基づくべき。**  
   - Baseline に正確な transactions が存在しない以上、データ到達性（delivery guarantee）が最優先となる。  
   - そのため CSV や外部実績データを中心とした “外形的な正しさ” の検証が必要。

## Implication
- リグレッションテスト設計において「Baseline と Target を比較する」という一般的な前提は破綻している。  
- Target transactions の検証は **以下の指標に基づく新しいテスト体系** を構築する必要がある：  
  - データロス検知  
  - 到達保証（Retry / 冪等性）  
  - 並行稼働による比較（Baseline は部分的な比較対象としてのみ使用）  
  - bills ロールアップとの整合性  
- ADR では「Baseline-equivalence を指標にしない」「期待値を実績データから定義する」といった前提を明記すること。

## Observations
- Baseline transactions = 社食（2層構造）＋ コンビニ（line-item のみ）の寄せ集め  
- Target transactions = canonical model（社食＋コンビニ統合）  
- Baseline テスト基準は曖昧で、期待値として利用できない  
- CSV や請求台帳などの外部実績のみが Oracle として機能する

## Evidence
- `01_working/ef_solutions/10_raw-notes/2025-11-07-transactions-deep-dive.md`
- `01_working/ef_solutions/10_raw-notes/2025-11-07-agile-testing-issues.md`
- `01_working/bd_arch-design/15_raw-notes/2025-11-06-convini-receipt-items.md`
