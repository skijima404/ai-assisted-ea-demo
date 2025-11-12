---
id: adr-0011
title: テストと品質保証の再構築
status: proposed
date: 2025-11-07
approver: Architecture Board
tags: [ADR, testing, quality]
---

# テストと品質保証の再構築

## Context
現行システムではリグレッションテストの成功条件が曖昧で、ビジネス要件に基づく期待値が明示されていない。テストピラミッドは逆転構造となり、E2Eテストへの過剰依存が発生している。  
コード品質の低下、テストの信頼性欠如、並行稼働検証の欠如が移行プロジェクトの主要リスクとなっている。

## Decision
テストと品質保証を再構築し、**並行稼働を中心とした代替検証手法と、テストピラミッドの正常化を同時に実施する。**  
特にTransition Architectureにおける意思決定では、**Testability（検証容易性）を最重要項目として考慮する。**

## Rationale
- リグレッションテストの品質が低く、従来の方式では変更の安全性を検証できないため。
- 並行稼働・パケットミラーリングを通じた実データベース比較は、実環境での確実な検証を可能にする。
- テストピラミッドの正常化により、変更影響検知と継続的デリバリーの品質を確保できる。
- CI/CDと連携したコード品質チェックにより、技術的負債の再発を防止できる。

## Consequences

### Positive
- 並行稼働による新旧比較で、リグレッションテストの代替が可能
- 実データを用いた信頼性の高い検証が可能
- 段階的移行の各ステップで品質を確認でき、移行リスクを軽減
- コード品質メトリクス導入により、開発チーム間で品質基準が統一される

### Negative
- 並行稼働と監視に伴う運用コストが増加する
- 初期の自動テスト整備とメトリクス導入に時間がかかる
- チームのテストスキル向上が前提となり、初期トレーニングが必要

### Mitigations
- OpenShiftやService Meshの標準監視メトリクスを活用して監視コストを抑制
- Canary ReleaseやDark Launchを活用し、影響範囲を段階的に拡大
- 自動テストテンプレートや品質ゲートの標準化を推進

## Reference
- STAR by Mike Amundsen (https://mamund.site44.com/talks/2021-05-orm-migration/2021-05-orm-migration.pdf)
- ADR-0007 (Observability)
- ADR-0009 (Lifecycle-Aware Architecture)
- ADR-0010 (Manage Technical Debt as a Portfolio)
