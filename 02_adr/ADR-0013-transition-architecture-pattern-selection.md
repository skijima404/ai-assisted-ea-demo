# ADR-0013: Transition Architecture Pattern Selection

## Status
Accepted

## Context

Target Architectureへの移行方法として、3つのTransition Architectureパターンを評価した。

### 評価基準
1. **リスク**: 移行失敗のリスク、ユーザー影響、データ損失の可能性
2. **期間**: 移行完了までの想定期間
3. **Learning Curve**: チームの技術習得難易度
4. **検証可能性**: 各段階での品質保証の実現性
5. **依存関係管理**: システム間の依存関係による制約

### Pattern 1: 段階的マイクロサービス移行（Incremental Microservices Migration）

**アプローチ**:
```
Step 1: CSV→Kafka移行 + 利用明細独立
Step 2: 決済履歴分離 + Mirror導入
Step 3: UI分離 + API Gateway導入
Step 4: HR側API完成（ドメイン別）
Step 5: 社食側API完成 + レガシー撤去
```

**特徴**:
- 技術的難易度の低い要素から段階的に移行
- 各ステップで並行稼働による検証
- Learning Curveへの配慮（CSV→Kafkaから開始）
- Contract Testを段階的に導入
- ドメイン別に並行実施可能（Step 4/5）

**評価**:
- リスク: ★☆☆☆☆（低）- 段階的検証により早期発見
- 期間: 12-15ヶ月
- Learning Curve: ★★☆☆☆（緩やか）- 小さな変更から開始
- 検証可能性: ★★★★★（高）- 並行稼働、Dark Launch、Canary Release
- 依存関係管理: ★★★★☆（良好）- Mirrorによる二重配信で柔軟に対応

**詳細**: `01_working/ef_solutions/60_transition-draft/pattern-1/`

### Pattern 2: UI優先移行（UI-First Migration）

**アプローチ**:
```
Step 1: UI分離 + API Gateway導入
Step 2: フロントエンド/バックエンド完全分離
Step 3: バックエンドマイクロサービス化
Step 4: イベント駆動化
Step 5: レガシー撤去
```

**特徴**:
- ユーザー体験の改善を最優先
- UIの早期モダナイゼーション
- バックエンドは段階的にAPI化
- イベント駆動化は後半に実施

**評価**:
- リスク: ★★★☆☆（中）- UI/API同時変更による影響範囲の広さ
- 期間: 9-12ヶ月
- Learning Curve: ★★★☆☆（急）- 初期にUI/API両方の技術が必要
- 検証可能性: ★★★☆☆（中）- UI変更の検証が複雑
- 依存関係管理: ★★☆☆☆（課題あり）- プレゼンテーションとビジネスロジックの結合度が高い

**課題**:
- 旧システムはUI/ビジネスロジックの結合度が高く、分離が困難
- API仕様を先に確定する必要があるが、レガシーの制約で理想的なAPIを設計できない
- 初期段階でContract Testの整備が必須だが、体制が未整備

### Pattern 3: データ駆動移行（Data-Driven Migration）

**アプローチ**:
```
Step 1: イベントストリーミング基盤構築（Kafka）
Step 2: データ統合基盤構築（Single Source of Truth）
Step 3: イベント駆動でのデータ連携
Step 4: アプリケーションマイクロサービス化
Step 5: UI刷新 + レガシー撤去
```

**特徴**:
- データ基盤を最初に整備
- イベント駆動アーキテクチャを全面的に採用
- データの一元管理を優先
- アプリケーション移行は後半

**評価**:
- リスク: ★★★★☆（高）- データ移行の複雑性、影響範囲の広さ
- 期間: 15-18ヶ月
- Learning Curve: ★★★★☆（急峻）- 初期からKafka/イベント駆動の深い理解が必要
- 検証可能性: ★★☆☆☆（低）- データ移行の検証が複雑
- 依存関係管理: ★★★☆☆（課題あり）- 全システムのデータ移行を同期する必要

**課題**:
- Kafkaやイベント駆動の習熟が前提となり、Learning Curveが急
- データ移行の検証が複雑で、リグレッションテストの信頼性が低い現状では高リスク
- Quick Winが出しにくく、チームのモチベーション維持が困難

## Decision

**Pattern 1: 段階的マイクロサービス移行** を採用する。

### 採用理由

1. **リスクの最小化**:
   - 各ステップで並行稼働による検証が可能
   - 問題発生時のロールバックが容易
   - 段階的な検証により、早期に問題を発見・修正できる

2. **Learning Curveへの配慮**:
   - CSV→Kafkaという小さな変更から開始し、チームが段階的に習熟できる
   - 各ステップで新しい技術（Kafka、Contract Test、API Gateway等）を1つずつ導入
   - Quick Winを演出しやすく、チームのモチベーション維持が可能

3. **検証可能性の高さ**:
   - 並行稼働、Dark Launch、Canary Releaseを活用した段階的検証
   - リグレッションテストが不十分な現状でも、新旧比較で品質を担保
   - Contract Testを段階的に導入し、テストピラミッドを正常化

4. **柔軟性**:
   - Mirrorによる二重配信で、プレゼンテーションとビジネスロジックの結合問題に対応
   - Step 4/5はドメイン別に並行実施可能で、リードタイム短縮
   - 各ステップでの学びを次ステップに活かせる

5. **原則との整合性**:
   - principle-011 (Testability & Contract Testing): 段階的にContract Testを導入
   - principle-009 (Lifecycle-Aware Architecture): レガシー撤去プロセスまで含む
   - principle-010 (Manage Technical Debt as Portfolio): 技術的負債を段階的に解消

### Pattern 2/3を採用しない理由

**Pattern 2 (UI優先移行)**:
- プレゼンテーションとビジネスロジックの結合度が高く、UI分離が困難
- 初期段階でUI/API両方の技術が必要で、Learning Curveが急
- Contract Testの体制が未整備の状態で、API仕様を確定するリスクが高い

**Pattern 3 (データ駆動移行)**:
- 初期からKafka/イベント駆動の深い理解が必要で、Learning Curveが急峻
- データ移行の検証が複雑で、リグレッションテストが不十分な現状では高リスク
- Quick Winが出しにくく、チームのモチベーション維持が困難

## Consequences

### Positive
- 各ステップでの検証により、移行リスクを最小化できる
- チームが段階的に技術を習熟でき、品質を維持しながら移行できる
- Quick Winを演出しやすく、プロジェクトの継続性を確保できる
- ドメイン別の並行実施により、期間を短縮できる可能性がある

### Negative
- 12-15ヶ月と期間が長く、組織的な忍耐が必要
- 段階的な移行のため、一時的に新旧システムの二重運用コストが発生
- 各ステップでの並行稼働期間（最大3ヶ月）により、運用負荷が増加

### Mitigation
- 各ステップの並行稼働期間を明示的に設定（最大3ヶ月）し、突合プロセスを標準化
- Step 4/5のドメイン別並行実施により、期間を短縮
- Dark Launch/Canary Releaseにより、ユーザー影響を最小化
- Exit Criteria（SLO 30日連続達成）を明確化し、完了条件を定義

## Implementation

採用されたPattern 1の詳細は以下に格納：
- **確定版**: `03_arch-artifacts/transition/`（実装チームはこちらを参照）
- **検討履歴**: `01_working/ef_solutions/60_transition-draft/pattern-1/`（参考用）

変更管理:
- 確定後の変更: `03_arch-artifacts/transition/` で直接実施
- 変更理由: ADR Amendment で記録
- Git履歴で追跡

## Reference
- Architecture Vision: `03_arch-artifacts/arch-vision.md`
- Architecture Principles: `03_arch-artifacts/principles/`
- Target Architecture: `03_arch-artifacts/target/`
- Analysis: `01_working/ef_solutions/20_analysis/`
- Transition Architecture (Adopted): `03_arch-artifacts/transition/`（コピー後に作成予定）
- Alternative Patterns: `01_working/ef_solutions/60_transition-draft/`
- ADR-0011: Testability & Contract Testing
- ADR-0009: Lifecycle-Aware Architecture
- ADR-0010: Manage Technical Debt as a Portfolio

