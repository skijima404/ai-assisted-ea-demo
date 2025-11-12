# ADR-0004: Microservices Architecture (マイクロサービスアーキテクチャ)

## Status
Accepted

## Context

現状のモノリス構造は、技術的負債とEOSL（End of Service Life）リスクの増大、責務の曖昧さ、組織境界を越えた機能実装などの課題を抱えています。これにより、システムの保守性・拡張性が低下し、柔軟な対応が困難となっています。

これらの問題は、アーキテクチャビジョン「3. 柔軟で持続可能なシステム基盤の実現 - 役割ごとの明確な分担、変化に強い仕組み」を阻害しているため、改善が必要です。

## Decision

ビジネスドメインと運用負荷に基づき適切な粒度でシステムを分割し、マイクロサービスアーキテクチャを採用する。各サービスは独立して開発・デプロイ・スケーリング可能とし、過度な細分化は避ける。

このアプローチにより、サービス間の疎結合を保ちつつ、段階的なモダナイゼーションを実現し、運用面でのスケーラビリティとレジリエンスの向上を図る。

## Consequences

### Positive
- ビジネスドメインに沿った責務分離により、変更影響範囲を限定可能
- 各サービスの独立したスケーリングによりリソース効率が向上
- 技術的負債やEOSLリスクをサービス単位で段階的に解消可能
- システム全体の柔軟性と持続可能性が向上

### Negative
- システム全体の複雑性が増し、サービス間の調整や運用負荷が増加
- 分散トレーシングやログ集約など運用基盤の整備が必要
- 過度な細分化は管理コストの増大を招くリスクがある

### Mitigation
- ドメイン知識と運用負荷を踏まえた適切なサービス境界の設定
- 共通機能はプラットフォーム層で提供し重複を避ける
- 観測可能性の確保と段階的な移行計画によるリスク軽減

## Reference
- Architecture Vision: `03_arch-artifacts/arch-vision.md`
- Analysis: `01_working/bd_arch-design/20_analysis/2025-11-05-eosl-technical-debt.md`
- Analysis: `01_working/bd_arch-design/20_analysis/2025-11-06-system-integration-issues.md`
- Principle: `03_arch-artifacts/principles/principle-004-microservices.md`
