# Backcasting Map Cross-Check Report

**Date**: 2025-11-13  
**Reviewer**: AI-Assisted EA System  
**Scope**: Target Architecture & Transition Architecture (Step 1-5)

---

## Executive Summary

Target ArchitectureとTransition Architectureを、Backcasting Map（7 Success Criteria, 12 Symptoms, 41 Root Causes）を用いてクロスチェックしました。

### ✅ 優れている点
- Transition Architectureは非常に詳細で、段階的移行戦略が明確
- テスト戦略（Contract Test、E2E Test）が各ステップに組み込まれている
- セキュリティ考慮（Data Classification、認可統制）が明示
- 運用アーキテクチャ（Observability、SLO/SLI）がStep 3以降で充実

### ⚠️ 改善が必要な点
1. **Target Architectureで外部システム連携が不明瞭** (rc-012)
2. **データ検証方法が「検討」レベル** (rc-027)
3. **DR戦略が欠落** (rc-019, rc-030, rc-018)
4. **Target Architectureでキャパシティ計画が不明** (rc-017, rc-011)
5. **サブシステム間の契約（API仕様、データフォーマット）が不明瞭** (rc-028)

---

## Phase D: Target Architecture レビュー

### ✅ 適切に対応されているRoot Cause

#### rc-013: Baseline理解不足
- **Status**: ✅ 問題なし
- **Evidence**: Baseline文書（container-diagram.md, functional-map.md）が存在し、既存システムの構造が明示されている

#### rc-034: 非現実的なTarget
- **Status**: ✅ 問題なし
- **Evidence**: 
  - マイクロサービス化、イベント駆動は実現可能な設計
  - 技術スタック（OpenShift, Kafka, PostgreSQL, Keycloak）は標準的で実績あり

#### rc-023: 非標準構成
- **Status**: ✅ 問題なし
- **Evidence**: OpenShift、AMQ Streams/Kafka、PostgreSQLなど、エンタープライズで実績のある技術を採用

---

### ⚠️ 改善が必要なRoot Cause

#### ❌ rc-012: 外部システム依存関係の欠落
- **Status**: **問題発見**
- **Description**: 現行システムが他の社内外システムとどのように連携しているかがベースラインアーキテクチャで明示されておらず、全体構造の前提が欠落している。
- **Evidence in Baseline**: 
  - 人事システム（月次CSV連携）
  - 社内コンビニシステム（月次CSV連携）
- **Evidence in Target**: 
  - **外部システムとの連携が図示されていない**
  - 8つのマイクロサービス（Bill, Employee, History, Menu, Payment, Private, Transaction, UserProfile）は内部システムのみ
- **Impact**: 
  - 人事システムへのデータ連携方法（Kafka? REST?）が不明
  - 社内コンビニシステムからのデータ取込方法が不明
  - 統合点の欠落により、移行時に障害が発生するリスク
- **Triggered Symptom**: rf-003 (Capacity shortage)
- **Threatened Success Criteria**: sc-005 (Day 1 Operation Success)
- **Recommendation**:
  - Target Architectureに外部システム（人事、コンビニ）を明示
  - データ連携方式（Kafka Topic、REST API、バッチ）を図示
  - 統合点のインターフェース定義を追加

---

#### ⚠️ rc-020: テスト容易性戦略不足
- **Status**: 改善余地あり
- **Description**: テスト容易性を意識した設計やアーキテクチャ戦略が初期段階から不在である。
- **Evidence in Target**: 
  - Target Architecture自体にはテスト戦略が明示されていない
  - Principle-011 (Testability and Contract Testing) は存在
- **Evidence in Transition**: 
  - 各ステップにtesting.mdが存在
  - Step 3で「Contract Test、E2Eテスト」が明記
- **Impact**: 
  - Target Architectureレベルでテスタビリティが考慮されていないと、実装時に「テスト困難な設計」が混入するリスク
- **Triggered Symptom**: rf-012 (Numerous bugs occurred)
- **Recommendation**:
  - Target Architectureにテスト戦略セクションを追加
  - マイクロサービス間の契約テスト方針を明示
  - テストダブル（モック、スタブ）の利用方針を明示

---

#### ⚠️ rc-024: 運用アーキテクチャ未考慮
- **Status**: 改善余地あり
- **Description**: モニタリング、アラート設計、運用フロー、障害対応手順といった「運用アーキテクチャ」の検討が不足している。
- **Evidence in Target**: 
  - Observabilityに関する記述がない
  - 監視基盤が図示されていない
- **Evidence in Transition**: 
  - Step 3: 「API Gateway と Observability 基盤でメトリクスを収集・監視」
  - Step 5: SLO/SLI、アラート、オンコール、ダッシュボードが詳細
- **Impact**: 
  - Target Architectureに運用基盤が含まれていないと、実装時に「運用困難な構成」が採用されるリスク
- **Triggered Symptom**: rf-004 (Design-induced operational risk), rf-007 (Extended time required for troubleshooting)
- **Threatened Success Criteria**: sc-006 (Day 2 Operation Success)
- **Recommendation**:
  - Target Architectureに以下を追加：
    - Observability基盤（Prometheus、Grafana、Jaeger等）
    - ログ集約基盤（ELK Stack、Loki等）
    - アラート・通知基盤
  - SLO/SLIの初期値を定義

---

#### ⚠️ rc-017, rc-011: キャパシティ計画不足
- **Status**: 改善余地あり
- **rc-017**: 新システム側での必要な性能要件（リクエスト数、スループット、同時接続など）に対する容量見積もりが不十分。
- **rc-011**: 現行システムのアーキテクチャにおける処理能力・リソース使用状況が十分に分析されておらず、新アーキテクチャ設計時に必要な性能要件が漏れている。
- **Evidence in Baseline**: 
  - キャパシティに関する記述がない（現行システムの負荷特性が不明）
- **Evidence in Target**: 
  - キャパシティ要件が明示されていない
- **Evidence in Transition**: 
  - Step 3: レイテンシ監視、スループット・レスポンスタイムの閾値
  - Step 5: HPA/クォータ/PodAutoscaler
- **Impact**: 
  - 昼ピーク時の処理能力不足
  - データベース接続数の不足
  - Kafka throughputの不足
- **Triggered Symptom**: rf-003 (Capacity shortage)
- **Threatened Success Criteria**: sc-005 (Day 1 Operation Success)
- **Recommendation**:
  - Baselineに現行システムの性能特性を追加：
    - 昼ピーク時のトランザクション数（TPS）
    - DB同時接続数
    - レスポンスタイム（95パーセンタイル）
  - Target Architectureに性能要件を追加：
    - 各マイクロサービスのTPS要件
    - データベース接続プール設定
    - Kafka partition数、replication factor
    - Auto-scaling設定（HPA閾値）

---

## Phase F: Transition Architecture レビュー

### ✅ 適切に対応されているRoot Cause

#### rc-006: 非現実的なTransition
- **Status**: ✅ 問題なし
- **Evidence**: 
  - 5ステップの段階的移行は現実的
  - 各ステップで並行稼働期間（最大3ヶ月）を明示
  - Mitigationsが各ステップで詳細に記載

#### rc-016: 拡張性未考慮
- **Status**: ✅ 問題なし
- **Evidence**: 
  - マイクロサービス化により責務が分離
  - Kafkaによるイベント駆動で疎結合
  - API Gatewayによりルーティングが柔軟

#### rc-025: セキュリティ考慮不足
- **Status**: ✅ 適切に対応（Step 4）
- **Evidence**: 
  - Step 1: 「Data Classification Policy を遵守」
  - Step 4: 「Data Classification Policy 準拠のアクセススコープ管理」
  - Step 4: 「API Gateway および IDP と連携した認可統制」

#### rc-024: 運用アーキテクチャ（Transition内）
- **Status**: ✅ 適切に対応（Step 3, 5）
- **Evidence**: 
  - Step 3: API Gateway、Observability基盤、メトリクス収集、監視
  - Step 5: SLO/SLI、アラート、オンコール、運用移管、Runbook

---

### ⚠️ 改善が必要なRoot Cause

#### ⚠️ rc-027: データ検証方法未定義
- **Status**: 改善余地あり
- **Description**: 移行・結合・本番反映におけるデータ整合性の検証方法（例：差分チェック、クロスチェック、期待値定義など）が設計段階で明確に定義されていない。
- **Evidence**: 
  - Step 1: 「突合プロセスを標準化」「過去のCSVを鏡データ（Baseline）としてリプレイテスト」
  - Step 2: 「突合頻度（例：日次）を明示し、スクリプトまたはCamel等で自動化を**検討**」
- **Issue**: 
  - 「検討」レベルで、具体的な検証方法が確定していない
  - 突合の具体的な方法（フィールド単位の比較、集計値の比較、サンプリング等）が不明
- **Impact**: 
  - 不整合やデータ破壊の兆候を早期に発見できず、汚染状態でシステムが稼働してしまうリスク
- **Triggered Symptom**: rf-002 (Production data contamination)
- **Threatened Success Criteria**: sc-004 (Successful Migration)
- **Recommendation**:
  - Step 1, 2に具体的なデータ検証方法を追加：
    ```
    ## Data Verification Strategy
    
    ### 検証対象
    - 決済履歴: 全件突合（トランザクションID、金額、タイムスタンプ）
    - 集計値: 日次・ユーザー別合計金額
    
    ### 検証方法
    1. **全件突合**: 新旧システムのトランザクションをIDで突合し、差分を検出
    2. **集計値突合**: 日次バッチで集計値を比較
    3. **サンプリング検証**: ランダムサンプリング（1%）で詳細検証
    
    ### 検証ツール
    - diff script (Python/bash)
    - Apache Camel の DataSet component
    
    ### 不整合発生時の対応
    1. アラート発報
    2. トリアージ会議（24時間以内）
    3. 根本原因分析
    4. 修正・再検証
    ```

---

#### ⚠️ rc-028: サブシステム間の前提不一致
- **Status**: 改善余地あり
- **Description**: アーキテクチャ設計段階で定義された全体構成図（箱ダイアグラム）は存在するものの、実装フェーズにおいて各サブシステム間で前提や制約が共有されず、実装の整合性が取れていない。
- **Evidence**: 
  - Step 2: Mirror、PayApp、Kafkaの連携が図示
  - しかし、以下が不明：
    - Kafka Topic名、スキーマ定義
    - メッセージフォーマット（JSON? Avro?）
    - データ契約（必須フィールド、データ型）
    - 性能要件（throughput、latency）
- **Impact**: 
  - 実装フェーズで「仕様のズレ」が発覚
  - 統合テストで大量の不具合
- **Triggered Symptom**: rf-003 (Capacity shortage), rf-012 (Numerous bugs occurred)
- **Recommendation**:
  - 各ステップに「Data Contract」セクションを追加：
    ```
    ## Data Contract (Step 2)
    
    ### Kafka Topics
    - `canteen.payment.events`: 決済イベント
    - `hr.employee.events`: 社員マスタ変更イベント
    
    ### Message Schema (Avro)
    ```avro
    {
      "type": "record",
      "name": "PaymentEvent",
      "fields": [
        {"name": "transaction_id", "type": "string"},
        {"name": "employee_id", "type": "string"},
        {"name": "amount", "type": "int"},
        {"name": "timestamp", "type": "long"}
      ]
    }
    ```
    
    ### Performance Requirements
    - Throughput: 100 TPS (peak: 500 TPS)
    - Latency: p95 < 200ms
    - Retention: 30 days
    ```

---

#### ❌ rc-019, rc-030, rc-018: DR戦略が欠落
- **Status**: **問題発見**
- **rc-019**: DR切替やバックアップ復元に関するテストが実施されていない、または頻度・網羅性が不足している。
- **rc-030**: （未読、DR戦略関連と推測）
- **rc-018**: （未読、DR戦略関連と推測）
- **Evidence**: 
  - Target Architecture、Transition Architectureのいずれにも、DR戦略、フェイルオーバー、レプリケーションに関する記述がない
- **Impact**: 
  - 災害時にシステム復旧ができない
  - データロストのリスク
  - 事業継続性が損なわれる
- **Triggered Symptom**: rf-006 (DR switchover failure)
- **Threatened Success Criteria**: sc-006 (Day 2 Operation Success)
- **Recommendation**:
  - Target Architectureに以下を追加：
    ```
    ## Disaster Recovery Strategy
    
    ### RTO/RPO
    - RTO (Recovery Time Objective): 4 hours
    - RPO (Recovery Point Objective): 1 hour
    
    ### Backup Strategy
    - PostgreSQL: Continuous WAL archiving to Object Storage
    - Kafka: Multi-region replication (MirrorMaker 2)
    
    ### Failover Strategy
    - Active-Standby configuration
    - DNS-based failover
    
    ### DR Testing
    - Quarterly DR switchover test
    - Annual full disaster recovery drill
    ```
  - Transition Architecture Step 5に「DR Testing」を追加

---

## Phase E: 技術選定 レビュー

### ✅ 適切に対応されているRoot Cause

#### rc-002: アップグレード容易性未考慮
- **Status**: ✅ おおむね適切
- **Evidence**: 
  - コンテナ化（OpenShift）により、バージョン管理が容易
  - Step 3でCamel APIは「移行限定」と明示し、Step 5で本格APIに移行
- **Minor Issue**: 
  - 各マイクロサービスのアップグレード戦略（Blue-Green? Canary?）が不明
- **Recommendation**: 
  - Target Architectureに「Deployment Strategy」セクションを追加

#### rc-023: 非標準構成
- **Status**: ✅ 問題なし
- **Evidence**: OpenShift、Kafka、PostgreSQL、Keycloak は標準的

---

## Symptoms & Success Criteria の観点でのレビュー

### 脅威にさらされているSuccess Criteria

#### sc-004: Successful Migration
- **Threatened by**: 
  - rf-002 (Production data contamination) ← rc-027 (データ検証方法未定義)
- **Current Status**: ⚠️ rc-027の対応が「検討」レベル

#### sc-005: Day 1 Operation Success
- **Threatened by**: 
  - rf-003 (Capacity shortage) ← rc-012 (外部システム依存関係欠落), rc-017 (キャパシティ計画不足), rc-011 (Baseline capacity未分析)
- **Current Status**: ⚠️ 外部システム連携とキャパシティ計画が不明瞭

#### sc-006: Day 2 Operation Success
- **Threatened by**: 
  - rf-004 (Design-induced operational risk) ← rc-024 (運用アーキテクチャ未考慮)
  - rf-006 (DR switchover failure) ← rc-019 (DR testing不足)
  - rf-007 (Extended troubleshooting) ← rc-024 (運用アーキテクチャ未考慮)
- **Current Status**: 
  - ⚠️ Target ArchitectureでObservabilityが不明瞭
  - ❌ DR戦略が欠落

---

## 優先度別推奨事項

### 🔴 Critical (即座に対応すべき)

1. **DR戦略の策定** (rc-019, rc-030, rc-018)
   - Target ArchitectureにDR戦略を追加
   - RTO/RPO、バックアップ戦略、フェイルオーバー戦略を明示
   - Transition Architecture Step 5にDR Testingを追加

2. **外部システム連携の明示** (rc-012)
   - Target Architectureに人事システム、コンビニシステムとの連携を図示
   - データ連携方式（Kafka Topic、REST API）を明示
   - インターフェース定義を追加

3. **データ検証方法の確定** (rc-027)
   - Step 1, 2の「検討」を「確定」に変更
   - 具体的な検証方法（全件突合、集計値突合、サンプリング）を明示
   - 検証ツールと不整合発生時の対応手順を追加

---

### 🟡 High (Phase G開始前に対応すべき)

4. **キャパシティ計画の策定** (rc-017, rc-011)
   - Baselineに現行システムの性能特性を追加
   - Target Architectureに性能要件を追加
   - 各マイクロサービスのTPS要件、DB接続プール、Kafka設定を明示

5. **運用アーキテクチャの明示** (rc-024)
   - Target ArchitectureにObservability基盤を追加
   - SLO/SLIの初期値を定義
   - ログ集約、メトリクス収集、分散トレーシングの構成を明示

---

### 🟢 Medium (Phase G中に対応可能)

6. **サブシステム間のData Contractの明示** (rc-028)
   - 各ステップにData Contractセクションを追加
   - Kafka Topic名、スキーマ定義、性能要件を明示

7. **テスト戦略の明示** (rc-020)
   - Target Architectureにテスト戦略セクションを追加
   - Contract Test、モック/スタブの利用方針を明示

8. **Deployment Strategyの明示** (rc-002)
   - Blue-Green / Canary Deployment戦略を明示
   - ロールバック手順を明示

---

## 結論

**Overall Assessment**: ⭐⭐⭐⭐☆ (4/5)

### Strengths
- Transition Architectureは非常に詳細で、段階的移行戦略が優れている
- テスト戦略、セキュリティ、運用アーキテクチャ（Step 3以降）が充実
- リスク軽減策が各ステップで明示されている

### Areas for Improvement
- Target Architectureが抽象的で、外部システム連携、DR戦略、キャパシティ計画が不足
- データ検証方法が「検討」レベルで、具体性が不足
- サブシステム間のData Contractが不明瞭

### Next Steps
1. Critical項目（DR戦略、外部システム連携、データ検証）を優先的に対応
2. High項目（キャパシティ計画、運用アーキテクチャ）をPhase G開始前に完了
3. Medium項目はPhase G中に段階的に対応

---

**Review Completed**: 2025-11-13

