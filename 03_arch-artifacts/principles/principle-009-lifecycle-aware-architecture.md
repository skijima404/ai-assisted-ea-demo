---
id: principle-009
title: Lifecycle-Aware Architecture（ライフサイクルを意識した設計）
tags: [principle, project]
status: approved
approver: Architecture Board
approved_on: 2025-11-06
---

# Lifecycle-Aware Architecture（ライフサイクルを意識した設計）

## Statement
システムおよびコンポーネントは導入時点の要件だけでなく、保守・アップグレード・廃棄を含むライフサイクル全体を見据えて設計する。  
ライフサイクルに応じた更新容易性を確保し、技術的負債やEOSLリスクを最小化する。

## Rationale
多くのシステムでは、初期導入時の要件に最適化されすぎており、時間の経過とともにメンテナンスやアップグレードが困難になる。  
特にモノリス構造では、OS・MW・DBなどのバージョン依存が強く、個別更新ができないことからEOSL（End of Service Life）リスクや技術的負債が蓄積しやすい。  
ライフサイクルを前提とした設計を行うことで、コンポーネント単位での独立した更新や段階的モダナイゼーションを可能にし、継続的な健全性を維持できる。

## Implications
- コンポーネントは独立したライフサイクルを持ち、個別にデプロイ・更新・廃止できる構造とする  
- 使用する外部製品（OS、MW、DB等）はLTS（Long Term Support）ポリシーを基準に選定する  
- 技術スタックの更新サイクルを可視化し、アーキテクチャ設計時点で更新計画を明示する  
- 年次または半期ごとに技術スタックの棚卸しを実施し、EOSLリスクを定期的に評価する  
- アップグレードや置換を容易にするため、コンポーネント間の依存関係を最小化する  
- 継続的なアーキテクチャレビューにより、ライフサイクル計画と実装の乖離を検出する

## Related Principles
- principle-002: Freshness-Driven Architecture  
- principle-004: Microservices Architecture  
- principle-007: Observability & Continuous Governance  

## Linked Vision
持続的な変革を支える仕組みの確立 — 技術の更新性と業務の継続性を両立するためのライフサイクル設計