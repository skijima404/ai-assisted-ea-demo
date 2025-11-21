---
id: an-001
date: 2025-11-06
title: upgrade-path-is-blocked-by-mw-compatibility
status: draft
tags: [analysis]
---

# Incident (Observed Behavior)
- OS / MW / DB いずれもサポート終了バージョンで稼働しており、保守ベンダーから複数回リスク通知が上がっている
- OS・DB のアップグレード計画が立案されたが、MW の互換性制約により実施不可と判断された
- アプリ改修なしに MW を更新できず、全レイヤーの更新がブロッケージされている状態が継続している
- 結果として、セキュリティパッチの欠如・障害発生時のベンダーサポート不可など、運用リスクが累積している

# Conclusion
OS/MW/DB のバージョンがすべて EOSL であり、特に **MW のバージョン互換性がボトルネックとなって全レイヤーのアップグレードパスが詰んでいる**。

# Assumptions / Hypotheses
- MW のバージョンは OS と DB の上位互換が存在しない（＝アプリ修正なしでのアップグレードは不可）
- MW がアプリコードと密結合しており、コード複雑性のため改修コストが高い
- EOL になった OS/MW/DB を同時にアップグレードするためには段階的移行が必要だが、その第一段階が MW に阻まれている
- セキュリティ対応は現行構成では限界がある（例：Backport がない、もしくは限定的）

# Reasoning
- OS/MW/DB がすべて EOSL → セキュリティ対応・サポートリスクが顕在化
- MW の互換性が「OS→MW→DB」の依存リンクを塞いでいる  
  → MW を更新できないために OS/DB も更新できない
- コード複雑性が MW のアップグレード難易度を押し上げている  
  → MW更新に伴うアプリ改修のリスクが高い
- その結果、**“どのレイヤーからも手をつけられない状態”** が発生している

# Implications
- レイヤー単体のアップグレードでは解決しない「構造的負債」になっている  
- Transition Architecture では  
  ① MW の置き換え/抽象化  
  ② アプリコード整理（デカ複雑性の削減）  
  を前段に置く必要がある  
- Baseline Architecture における Technical Debt の中心ノードとして扱うべき
- Security レベルの観点でも「現状維持」は最もリスクが高い判断になる

# Observations
- OS, MW, DB がすべて EOSL  
- アプリコードの複雑性が高い  
- MW の互換性が OS/DB の更新をブロックしている

# Evidence
- raw-notes/2025-11-05-eosl.md

