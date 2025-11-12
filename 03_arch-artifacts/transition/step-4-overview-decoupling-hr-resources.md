# Step 4: HRアプリケーションの疎結合化とAPI公開

## 目的 (Purpose)
本ステップは旧HRアプリケーションのリタイアに向けて残りの機能をマイクロサービスとし、移行を完成させることを目的とする。
社食システムとの連携が必要な箇所は全て終了しているため、以降は人事システム内のリファクタリングとなる。

## 範囲 (Scope)
- **人事側マイクロサービス独立**
  - 請求台帳
  - 社員の公開プロファイル
  - 個人情報

## 制約 (Constraints)
- 個人情報はAPI化するが、利用はHR所属メンバーのみとする。
- 個人情報を扱うAPIは、利用前にHR部門がデータ精査・マスキングルールを定義する。

## Consequences

### Positive
- 社員の公開プロファイルをAPI化することで、社内でデータの利活用を促進できる
- 責務の分離により、セキュリティ境界が明確化される

### Negative
- 認可設計が複雑化する
- データのクレンジングや分類に要するコストが増加する

### Mitigations
- Data Classification Policy 準拠のアクセススコープ管理を実施
- API Gateway および IDP と連携した認可統制を導入

## Reference
- principle-001: Single Source of Truth  
- principle-002: リアルタイムデータ処理  
- principle-003: イベント駆動アーキテクチャ  
- principle-006: Security First
