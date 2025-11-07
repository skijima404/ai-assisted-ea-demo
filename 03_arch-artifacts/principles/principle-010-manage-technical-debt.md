---
id: principle-010
title: Manage Technical Debt as a Portfolio（技術的負債をポートフォリオとして管理する）
tags: [principle, project]
status: approved
approver: Architecture Board
approved_on: 2025-11-06
---

# Manage Technical Debt as a Portfolio（技術的負債をポートフォリオとして管理する）

## Statement
技術的負債を「見えないコスト」ではなく、リスクを伴う投資対象としてポートフォリオ管理する。  
発生を抑制するだけでなく、適切に可視化・評価・優先順位付けを行い、組織として意思決定可能な状態を維持する。

## Rationale
多くの組織では、短期的な機能追加や納期優先の結果、技術的負債が蓄積し、システム更新やモダナイゼーションを阻害している。  
技術的負債を管理対象として明確に扱うことで、事業リスクを定量的に把握し、返済（リファクタリングや再設計）を投資判断として扱えるようになる。  
また、負債の可視化はアーキテクチャガバナンスを実効性あるものとし、継続的改善を促進する。

## Implications
- 技術的負債を定期的に棚卸しし、種類（コード、アーキテクチャ、プロセスなど）と影響範囲を明示化する  
- 複雑度メトリクスや依存関係分析などを用い、定量的にリスクを評価する  
- 新規開発と負債返済のバランスを取るため、負債返済スプリントや改善バッファを定常化する  
- 負債のリスクスコアをアーキテクチャレビューに組み込み、優先度を定期的に見直す  
- 負債の推移をポートフォリオとして追跡し、組織的な改善トレンドを評価する  
- AIや自動解析ツールを活用し、検出・分類・トレンド可視化を継続的に行う

## Related Principles
- principle-008: Data Contract & Schema Evolution  
- principle-009: Lifecycle-Aware Architecture  
- principle-007: Observability & Continuous Governance  

## Linked Vision
持続的な変革を支える仕組みの確立 — 技術的負債を見える化し、リスクを制御可能な状態で変革を継続する