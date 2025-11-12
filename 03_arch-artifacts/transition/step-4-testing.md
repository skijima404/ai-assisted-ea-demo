# Step 4 Testing and Release Policy

## Overview
This document defines the testing and release policy for **Transition Architecture Step 4** (HRアプリ疎結合およびAPI公開).
The focus is on validating BillAPI’s business logic, confirming API CRUD reliability, and ensuring test pyramid practices are effective without excessive parallel execution.

---

## 1. Testing Scope

### BillAPI
- ビジネスロジックを含むが、ロジック自体は簡潔であり、重大な副作用のリスクは低い。
- 並行稼働によるテストは原則不要。ただし以下の条件に該当する場合は実施を検討する：
  - OCM（Operational Change Management）の観点で必要とされる場合
  - テストピラミッドによるテスト品質が期待値を下回る場合
- 上記の場合は **CDC（Change Data Capture）によるリアルタイム突合** または **月次CSVによるデータ突合** を実施。
- 重点検証項目：
  - 請求ロジック（合算・端数処理・社員単位集計）
  - 書き込み競合（複数同時更新）時の整合性
  - HRDBとのトランザクション境界の整合

---

### 個人情報API / 社員プロファイルAPI
- 単純なCRUD操作のみを提供するAPI。
- 並行稼働によるテストは不要。
- 検証項目：
  - 権限管理（認可スコープ）とData Classification Policyとの整合
  - CRUD操作の正確性（Create, Read, Update, Delete）
  - APIスキーマの後方互換性維持

---

## 2. Test Pyramid Validation
- テストピラミッドの正常化が成立しているかを本ステップで確認。
- 単体テスト／統合テスト／E2Eテストがそれぞれ明確に分離され、E2Eへの過剰依存が解消されているかを確認。
- 自動テストの結果をCI/CDパイプラインで集約・可視化し、テストカバレッジを測定。

---

## 3. Monitoring & Release
- 本ステップでのAPI追加は**内部リリース扱い**とし、HRドメイン内での安定稼働を確認。
- OpenShiftメトリクスとログでAPIレイテンシおよびスループットを監視。
- 3営業日程度のライブ監視期間を設定し、異常がない場合に安定稼働とみなす。

---

## 4. Constraints and Dependencies
- PIIを含むAPIは、利用前にHR部門がデータ精査・マスキングルールを定義。
- Data Classification Policy準拠のアクセス制御を適用。
- テストピラミッドによる品質保証体制が整備されていることを前提とする。

---

## 5. References
- Step 3 Testing Policy (API Gateway Integration)
- principle-006: Security First
- adr-0011: テストと品質保証の再構築