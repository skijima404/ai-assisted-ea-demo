# Step 1 Testing and Release Policy

## Overview
This document defines the testing and release policy for **Transition Architecture Step 1** (利用明細・CSV→Kafka移行).  
The focus is on ensuring data integrity, operational safety, and progressive verification through staging and parallel operation.

---

## 1. Data Validation Strategy

### 利用明細 (Transactions)
- 現行利用明細 (社食システム) とはデータモデルが異なるため、既存のテストケースは流用不可。
- 新利用明細は単純な CRUD 構造。データは決済データ (Value Object) のため、**投入後の更新不可**。
- データソースは 2 系統 (社食・コンビニ)、いずれも長年運用実績のある **CSV ベース** で信頼性高い。
- CSVは過去6ヶ月分保管されているため、ステージング環境で過去6ヶ月分を投入して検証。
- 検証観点：
  - CSV構造の整合性
  - データマッピングの妥当性（旧→新属性対応）
  - CRUD操作の正確性

---

## 2. Load & Stress Testing
- ステージング環境で過去6ヶ月分のデータを投入し、リグレッションテストを実施。
- トランザクション集中時のネットワーク負荷を再現し、Kafka連携の安定性を確認。
- 想定リスク：
  - トラフィック集中による一時的なメッセージ遅延
  - ネットワーク断による再送キュー滞留
- 検証項目：
  - Kafkaコネクタ（Camelアプリケーション）のメッセージリトライ動作
  - 再送後の重複排除処理
  - 可観測性 (Observability) ダッシュボードでのエラー検知精度

---

## 3. Parallel Operation Testing
- 本番環境で 3 ヶ月の並行稼働を実施。
- 定期的に CSV を用いて新旧データの突合を行い、抜け漏れを検出。
- 差分検出時の対応：
  - 原因分析（データソース or Kafka連携）
  - Camel再処理 or 一時的CSV再投入による補正
- 並行稼働期間はユーザー非公開のため、修正対応に十分な余裕を確保。

---

## 4. HR / Canteen / Connector Test Scope

### コネクタ (Camel)
- Camelコネクタは本番デプロイ後、**即稼働環境**として運用を開始する。
- ステージング環境での単体・統合テストを完了後、本番デプロイ。
- 本番稼働開始後も監視を強化し、ログ・メトリクスを分析して安定性を評価。
- テスト項目：
  - 再送ポリシー（Retry）とエラーハンドリング
  - Kafkaへのメッセージ送信整合性（重複・欠損）
  - 障害発生時の自己復旧挙動

### 人事: 請求台帳
- ステージング環境で過去6ヶ月分のCSVを使用して動作検証。
- 本番デプロイ後、3ヶ月間CSVを使用して計算結果の突合を継続。
- 並行稼働期間中は、旧CSV連携とKafka連携結果を比較し、整合性を確認。

### 社食: ユーザープロファイル
- ステージング環境でダミーデータを使用して検証（過去1年分程度のデータ量を想定）。
- 本番デプロイ後は通常運用を継続し、抜け漏れ発生時はユーザー報告をトリガーに調査・修正。
- 利用明細はDark Launchとして本番環境にデプロイ後も非公開稼働し、安定性評価期間を設ける。

---

## 5. Release and Monitoring

### リリース方針
- ユーザーリリースは Step 5 で実施。本ステップでは内部的稼働のみ。
- Kafka連携安定化とチームの習熟を目的とした「内部リリース」扱いとする。

### 監視・運用
- OpenShift メトリクス、Kafka モニタリング、Service Mesh トレースで異常検知。
- 定期的なエラー率・スループットレポートを作成し、運用レビューで共有。
- 抜け漏れ・再送・重複を監査用にログ保存。

---

## 6. Constraints and Dependencies
- PIIを含むデータはData Classification Policyに従い、Kafka連携時にセキュリティ対策を検討。

---

## 7. References
- Transition Architecture Pattern-1 Step 1: CSV→Kafka
- adr-0012 (テストと品質保証の再構築)
