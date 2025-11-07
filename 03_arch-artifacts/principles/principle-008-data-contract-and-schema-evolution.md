---
id: principle-008
title: Data Contract & Schema Evolution（データ契約とスキーマ進化）
tags: [principle, project]
status: approved
approver: Architecture Board
approved_on: 2025-11-06
---

# Data Contract & Schema Evolution（データ契約とスキーマ進化）

## Statement
サービス間のデータ契約（Data Contract）は厳格に管理し、スキーマの変更は進化的（Evolutionary）に行う。破壊的変更を避け、後方互換性を維持しながら継続的な改善を可能にする。

## Rationale
現在のシステムでは、データ構造の非互換やAPI仕様の不整合が原因で、機能追加やリファクタリング時に大きな影響範囲が発生している。  
明確なデータ契約と進化的スキーマ管理を採用することで、依存関係を最小化し、継続的な機能拡張とモダナイゼーションを両立できる。  
また、契約テストやスキーマバージョニングを導入することで、変更の安全性と透明性を担保する。

## Implications
- すべてのサービス間通信は明示的なデータ契約を持つ（例: OpenAPI, AsyncAPI, Avro Schema）
- スキーマ変更は後方互換性を維持し、破壊的変更の場合は新バージョンを明示的に発行する
- スキーマリポジトリを集中管理し、契約変更時には影響範囲を自動分析する
- コードレビューやCI/CDパイプラインに契約テストを組み込み、破壊的変更を自動検出する
- データ移行時はスキーマバージョンごとに段階的移行を行う（デュアルライティング等）

## Related Principles
- principle-001: Single Source of Truth
- principle-003: Event Driven Architecture
- principle-005: Auto Scaling Efficiency

## Linked Vision
信頼できる情報基盤の構築 — 高品質なデータ構造と拡張性の両立