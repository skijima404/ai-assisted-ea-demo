# Step 2 Testing and Release Policy

## Overview
This document defines the testing and release policy for **Transition Architecture Step 2** (決済履歴分離 & ミラーリング導入).  
The focus is on verifying the correctness and stability of the new **Payment History microservice (PayApp)** and **Mirror (Camel) integration**, through staging verification, contract testing, and dark launch parallel operation.

---

## 1. Data Validation Strategy

### 決済履歴 (PayApp)
- 決済履歴は **Value Object** を扱うため、ビジネスロジックは軽量。
- 既存システムと異なり、Web API を実装するため **Contract Test** の導入をこのステップで実施。
- 本番環境からクレンジングした **過去3ヶ月分の決済データ** をステージングで流し、データ整合と性能を検証。
- 検証観点：
  - データ整合性（既存CSVデータとの比較）
  - 重複・欠損・順序の異常検出
  - 冪等性 (idempotency) の確認

### ミラーリング (Mirror / Camel)
- Camel ルートを用いた標準的なメッセージ転送を実装。
- 単体テストで Happy Path / Retry / Dead Letter を網羅。
- ステージング環境でフルパスの検証（CSV→Mirror→Kafka→PayApp）。

---

## 2. Load & Stress Testing
- ステージング環境で過去3ヶ月分のデータをリプレイし、処理性能とレイテンシを測定。
- 想定負荷：ピーク7,000件/時。
- 検証項目：
  - Mirror→Kafka→PayAppの遅延・再送時の追従性
  - Kafkaラグ解消時間・スループット
  - DLQ発生時の復旧手順確認
- 目標：
  - P99 レイテンシ ≤ 1秒
  - バックログ解消 ≤ 5分

処理遅延が一時的に発生しても、ユーザーは通常決済後20〜30秒の自然な遅延を伴って残高確認を行うため、即時の影響は想定されない。

---

## 3. Parallel Operation Testing
- 本番環境で **3ヶ月間の並行稼働** を実施。
- Mirror 経由の新決済履歴と既存システムの処理結果を突合。
- 定期的に差分チェックを実施し、検出時は根因分析 → 再送 → 修正。
- 並行稼働中は Dark Launch。ユーザーリリースは実施しない。

### Reconsiliation (新旧突合: オフピークCSV方式)
- 目的
  - ピーク帯の負荷増を避けつつ、決済履歴（PayApp）と旧決済テーブルの**業務的同値**を日次で検証する。
- チェック項目
  - 金額、書き込みタイミング (60秒までのずれを許容する)

---

## 4. HR / Canteen / Mirror Test Scope

### Mirror (Camel)
- ステージングで単体・統合テストを完了後、本番デプロイ。
- 本番デプロイ後、監視を強化して安定性を評価。
- テスト項目：
  - Retry / Error Handling
  - Kafka送信整合性（重複・欠損なし）
  - 再送ポリシー動作
  - DLQ発生時の検証

### HR / Canteen連携
- Kafka経由でのデータ受信・突合を継続。
- HR側では請求台帳と新決済履歴の整合を確認。
- 社食側では既存Appとのデータ重複を防止し、Mirror経由で統一。

---

## 5. Release and Monitoring

### リリース方針
- Dark Launchを3ヶ月実施後、Kafkaへの決済データ投入ソースを新PayAppに切り替える。
- Web APIはリリース済みだが、外部クライアントがないためユーザー影響は限定的。

### 監視・運用
- OpenShift メトリクス、Kafkaモニタリング、Service Meshトレースを利用して監視。
- 主要メトリクス：
  - 取り込みTPS、Kafkaラグ、DLQ件数、再送回数
  - DB挿入失敗率、重複抑止件数
- 運用レビューを週次で実施し、問題発生時のRunbookを更新。
- PayApp DBの CPU使用率 / IO Wait / Connection Pool 使用率
- Kafka→PayApp間のメッセージレイテンシ
- Camel Mirrorのスループット（トランザクション毎秒）
- Peak時間（12:00–13:00）と非Peakの比較比率


### Decision Gate: R/W分離（Read/Write Split）の要否判定
ライブ監視の結果に基づき、**PayApp の R/W 分離**（例: 読み取り専用レプリカの導入、CQRS的分離）の要否を判定する。

**主要判定指標（いずれかが継続的に閾値超過した場合は要検討）**
- **DB CPU 使用率**: 平均 &gt; 70% がピーク時間帯に **3日連続** で発生  
- **IO Wait**: 平均 &gt; 10% を **連続3日** 観測  
- **コネクションプール使用率**: ピーク時に **80%超が5分以上継続**  
- **読み取りクエリのP99レイテンシ**: **&gt; 200ms** が **3日連続**  
- **書き込み競合（ロック待機）件数**: ピーク帯で **継続的に増加傾向**  
- **Kafka→PayApp取り込みラグ**: **5分超** が **連続2回以上**（読取り負荷が回復を阻害）

**R/W分離を行う場合の基本方針**
1. **最小実装から**:  
   - 読取系のみをレプリカに逃がす（アプリは Read-only DSN を優先）  
   - API Gateway / Service Mesh レベルで **キャッシュ（短寿命）** を併用  
2. **データ整合性**:  
   - Read, Writeは必ず別端末からとなるためWrite直後のReadの結果の整合性は意識する必要なし
     - Writeは必ず決済端末、その後ReadはスマートフォンやPCからとなる。20-30秒のラグは許容範囲
     - 将来的に同一端末での決済（Write）と照会（Read）が統合される場合は、整合性制御（read-your-write保証）の導入要否を再検討する。
3. **スキーマ/インデックス整備**:  
   - 読み取り頻度の高いクエリに合わせて **カバリングインデックス** を再設計  
   - 書込み性能に影響するインデックスは **コスト/効果評価** の上で見直し  
4. **スケーリング連携**:  
   - HPA の閾値（CPU/レイテンシ）と DB のオートスケール/レプリカ数を **連動**  
   - ピーク前の **事前スケール** をスケジュール適用

**代替/補完オプション（R/W分離の前に検討）**
- **アプリケーションレベルキャッシュ**（例: 短TTLのメニュー/サマリ系API）  
- **クエリ最適化**（N+1解消、SELECT列の最小化、バッチサイズ調整）  
- **パーティショニング/シャーディング**（時系列ベースでの書込み拡張）  
- **アウトボックスパターン**で書込みと配信を分離し、取り込み競合を緩和

**検証・リリース手順**
- ステージングで **レプリカ遅延** と **整合性要件** を確認（読み取り専用での契約テスト）。  
- Canary で **読み取り系の一部エンドポイントのみ** レプリカ参照へ切替。  
- 監視強化（レイテンシ/エラー率/ユーザー影響）し、問題なければ段階的に拡大。

**ロールバック基準**
- 読取P99レイテンシが **導入前比+50%超** を30分以上継続  
- 整合性関連のユーザー問い合わせが **閾値（例: 0.1%）** を超過  
- Kafkaラグが **10分超** に悪化し回復傾向なし（60分以内）

---

## 6. Constraints and Dependencies
- 決済データは PII を含むため、Data Classification Policy に従いセキュリティ対策を実施。
- Camel・PayApp ともに OpenAPI / Schema Registry で契約を固定し、破壊的変更を防止。
- Mirror の運用開始後、既存CSV経路をバックアップとして維持（最大3ヶ月）。

---

## 7. References
- Transition Architecture Pattern-1 Step 2: 決済履歴分離 & ミラーリング導入
- adr-0012 (テストと品質保証の再構築)
- principle-007 (Observability)
- principle-002 (リアルタイムデータ処理)
- principle-003 (イベント駆動アーキテクチャ)
