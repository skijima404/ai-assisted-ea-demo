---
id: an-213
date: 2025-11-07
title: bills-require-billing-view-in-event-driven-architecture
status: draft
tags: [analysis, bills, migration, event-driven]
---

# Bills require a redesigned billing view in an event-driven architecture

## Incident (Observed Behavior)
- 現行の bills（請求台帳）は **月次 CSV を単純に集計するだけ** の仕組みとして成立している。  
  - コンビニ CSV と社食 CSV をロールアップするだけで整合性を保てる。
  - 遅延や重複、補正イベントなどは存在しない前提で作られている。  
- Target Architecture では **決済イベント (transactions)** をリアルタイムに取り込み、そこから請求台帳を生成する必要がある。  
  - 遅延イベント、重複、再送、取消など “イベント処理の難しさ” が発生する。
- よって、Baseline の bills は Target のイベントドリブンモデルでは成立せず、**責務を再設計する必要がある**。

## Conclusion / Assumptions
- **結論:** bills は CSV 集計モデルではなく、**決済イベントから導出される billing view** として再構築しなければならない。
- **Assumptions:**
  - Target の transactions は社食・コンビニ双方の canonical な決済ログ。
  - イベントの順序保証や到達保証を前提としないと整合性を維持できない。
  - CSV と現行請求台帳は移行初期における **Oracle（正解データ源）** として使える。

## Reasoning
1. **Baseline の bills は “月次 CSV 集計” という静的モデルであり、イベントの概念を持っていない。**
   - 遅延データも、取消も、再送もない前提で成り立つ。
   - Target ではこの前提が破綻する。

2. **イベントドリブン化により、時間軸と順序の問題が発生する。**
   - 給与締め日までに “すべてのイベントが届いている” 保証がない。
   - duplicate, retry, late arrival を処理する必要がある。

3. **単純集計ではなく、決済ログ（event journal）から billing view を計算する必要がある。**
   - Event Sourcing 的に “決済ジャーナル” を保持するモデル。
   - または Kafka Streams / Camel / バッチ再演算でビューを形成する方式が必要。

4. **移行フェーズでは CSV が Oracle として利用できる。**
   - 長年運用されている CSV → 人事請求台帳は実績があるため、差分検証が容易。

## Implication
- bills の移行は transactions より “イベント特有の難易度” が高い領域になる。
- 移行計画では以下を明確に分離しないと破綻する：
  1. 新 transactions（社食＋コンビニ統合）の構築  
  2. 決済イベントジャーナルの設計  
  3. billing view の導出パイプラインの構築  
  4. CSV / 現行請求台帳との突き合わせによる整合性確認  
- ADR では、bills が **派生ビューであり、単なるロールアップではない** ことを明記すべき。

## Observations
- Baseline bills: 月次 CSV の単純集計。イベントなし前提。
- Target bills: unified transactions からの派生ビューであり、イベント到達性を考慮する必要がある。
- CSV と現行請求台帳は移行初期の Oracle として重要。

## Evidence
- `01_working/ef_solutions/10_raw-notes/2025-11-07-transactions-deep-dive.md`
- `01_working/ef_solutions/10_raw-notes/2025-11-07-service-dependencies.md`
- Baseline CSV 運用実績（社内ヒアリング）