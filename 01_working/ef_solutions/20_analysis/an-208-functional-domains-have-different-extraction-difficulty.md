---
id: an-208
date: 2025-11-07
title: functional-domains-have-different-extraction-difficulty
status: draft
tags: [analysis, domain-extraction, migration]
---

# Functional domains have different extraction difficulty

## Incident (Observed Behavior)
- 社食システムに内包されている複数のドメイン（users, menu, payments, transactions, bills）について、切り出し難易度に差があることが判明した。  
- `transactions`（利用明細）は、現行では社食側の `payments` と社内コンビニからの CSV データという **2 つのデータソースに依存**しているが、いずれもバッチ／CSV 連携に落とし込めるため、技術的な依存関係としては比較的限定的である。  
  一方 Target Architecture では、`transactions` は `bills`（請求台帳）のロールアップ元となることが想定されており、**毎回の決済と月次ロールアップの対応が取れていること** が前提になる。現状のデータ構造ではその関係が十分に表現されていない。  
- `menu` や `users` は更新頻度が低く、データソースも限定的であり、比較的早期に切り出してスキルトレーニング（コントラクトテスト等）に使える可能性がある。

## Conclusion / Assumptions
- **結論:** ドメインごとに移行難易度が大きく異なるため、全ドメインを横並びで移行するのは不合理である。依存関係の複雑さ・更新頻度・参照される文脈の多さを基準に、段階的な移行シーケンスを設計する必要がある。  
- **Assumptions:**  
  - `menu` や `users` は比較的独立しており、早期移行が可能。  
  - `transactions` は現行では `payments` とコンビニ CSV をソースとするが、どちらも CSV／バッチに落とし込めるため、現行構造のままでも段階移行の設計は可能である。  
  - Target Architecture では、`transactions` が `bills` の明確なロールアップ元となり、**「毎回の決済」→「月次請求」という意味での対応関係** が生じる。このため、両者の整合性を担保する仕組みが必要になる。  
  - `bills` は現行では CSV をデータ源としており、transactions との関係は弱いが、鏡データとして段階移行の設計を支えつつ、将来的には transactions との対応付けを強化する役割を担う。

## Reasoning
1. **依存関係の強弱がドメインごとの移行容易性を左右する。**  
   - `users` や `menu` は参照元が限定的で、独立した CRUD として扱いやすい。  
   - `transactions` は現行では `payments` ＋ コンビニ CSV という 2 ソース依存だが、双方ともバッチ／CSV 経由で扱えるため、技術的な切り出しは比較的コントロールしやすい。  
   - Target Architecture では、`transactions` が `bills` のロールアップ元として働く必要があり、「1明細 vs 月次集計」という意味で **論理的な結合度** が高くなる。

2. **更新頻度が高いドメインは切り出しタイミングの慎重な選定が必要。**  
   - `payments` は決済端末からの書き込みが多く、切り出し時のダウンタイムやデータロスのリスクが高い。  
   - `transactions` はその上流のデータフローに依存するため、先に `payments`／コンビニ CSV 経路の安定化を行う必要がある。

3. **Target Architecture では業務責務の境界が変わるため、“正しいドメインのあり方” が現行とは異なる。**  
   - 現行の `transactions` は、主にコンビニ側の「レシートのラインアイテム」（決済金額より細かい粒度）を表現しており、人事側の請求台帳（`bills`）とのロールアップ関係は実質的に機能していない。  
   - To-Be では、`transactions` が「天引き対象となる利用履歴」の正として集約され、`bills` はそのロールアップビューとして機能することが期待される。

4. **早期に切り出せるドメインを用いたスキルトレーニングは、全体移行の成功率を高める。**  
   - `menu` などはデータ更新が少なく、移行中のリスクが低いため、開発者にコントラクトテストや GitOps を習熟させる教材として適している。  
   - より複雑な `transactions`／`bills` に着手する前に、こうしたシンプルなドメインでパイプラインやテストの型を確立できる。

## Implication
- 移行計画では、技術依存よりも **ドメイン境界・データ更新頻度・依存関係の深さ** を基準に優先順位を設計すべきである。  
- 特に `transactions` 系は後段の移行ステップにまとめ、  
  - 鏡（CSV / 現行人事請求台帳）  
  - 並行稼働（payments → 新 transactions）  
  を組み合わせた慎重な戦略が必要になる。  
- 逆に `menu` や `users` は早期切り出し → 標準 API → コントラクトテスト習熟 → 移行パイプラインの整備といった “アーリーステージ学習素材” として活用できる。

## Observations
- users: 入力頻度が少なく切り出しやすい  
- menu: 曜日単位の更新であり、リスクが低い  
- payments: 書き込みが多く、参照元も多い  
- transactions: データソースは `payments` とコンビニ CSV の 2 系統。現状はコンビニ決済のレシート行レベルのみで、人事側請求台帳（`bills`）とのロールアップ関係は弱いが、Target では per-transaction と monthly bills の対応付けが必要。  
- bills: 現行は CSV をデータ源としており、transactions との結びつきは弱いが、鏡データおよび将来のロールアップ先として重要。

## Evidence
- `01_working/ef_solutions/10_raw-notes/2025-11-07-functional-extraction-difficulty.md`