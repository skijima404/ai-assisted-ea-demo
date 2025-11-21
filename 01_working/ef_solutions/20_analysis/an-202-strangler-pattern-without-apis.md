---
id: an-202
date: 2025-11-06
title: strangler-pattern-without-apis
status: draft
tags: [analysis, migration, strangler, integration]
---

# Strangler Pattern in a Non-API Environment

## Incident (Observed Behavior)
- 現行システムは Web API を提供せず、すべてのシステム間連携はファイル（CSV）で構成されている。
- UI とビジネスロジックが密結合しているため、UI 分離や API 化を初手で行うことは高いリスクを伴う。
- Strangler Fig Pattern を用いた段階的移行が望ましいが、一般的に想定される「API ルーティング」「パケットミラーリング」が利用できない環境である。

## Conclusion / Assumptions
- **結論:** API を使わずとも、Strangler Fig Pattern の本質（既存システムをつぶさず、段階的にトラフィックを新旧で切り替える）は「データ流通点（CSV、テーブル、トピック）」をインターフェースとして扱うことで実現可能である。
- **Assumptions:**
  - CSV／テーブル／Kafka トピックなど「データの発生点」は既存システムとの境界として十分機能する。
  - 「UI/API のルーティングを使わない Strangler」という設計は、データ指向のアプローチとして妥当。
  - ルーティングの切り替えに相当する操作は、データ取り込み元の切り替え（CSV→Kafka）や、読み取り先の切り替えで代替可能。

## Reasoning
1. **Strangler Pattern の本質は「段階的置き換え」と「境界インターフェースの二重化」にある。**  
   API が前提と思われがちだが、本質的には「新旧を並行させながら徐々にトラフィックを移すこと」であり、API は単にそのための手段に過ぎない。

2. **CSV／テーブルは既に「境界インターフェース」として機能している。**  
   多くの処理は「(旧) CSV → (旧) DB」に依存しており、これを「(新) Kafka → (新) microservice」へ段階的にリプレースできる構造がある。

3. **データ流通点を Strangler の「チョークポイント」として扱える。**  
   - CSV/ファイル出力  
   - DBテーブル  
   - Kafkaトピック  
   これらは API と同等に「新旧でデータを重ねて流す」「ミラーリングする」ための観測点となる。

4. **UI/BL の結合度が高いため、アプリケーション境界での Strangler は困難。**  
   データ指向の Strangler を採用する方が成功確率が高い。

## Implication
- Strangler Pattern を **API ルーティング前提で理解していた場合に比べ、段階的移行が可能であることが明確になる。**
- UI やアプリケーションロジックに手を入れる前に、データフローの二重化・ミラーリング・バイパス経路の準備を進められる。
- 結果として、**リスクの高い UI / BL の再構築を後ろ倒しにしながら、安全に移行を開始できる。**

## Observations
- raw-notes で以下の観察があった：  
  - API が存在しないことが Strangler を阻害している  
  - CSV／DB／トピックをインターフェースと見立てれば Strangler 思想が適用可能という示唆  
  - UI 切り出しのリスクが大きい

## Evidence
- `01_working/ef_solutions/10_raw-notes/2025-11-06-strangler-pattern.md`
