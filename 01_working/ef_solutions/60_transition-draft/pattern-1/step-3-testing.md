# Step 3 Testing and Release Policy

> 対象: **社食 Read-Only API (CamelAPI) + API Gateway + 新UI (CanteenUI/DedUI)**
> 目的: 画面分離の第1段 (参照系) を安全に提供し、**E2E とテストピラミッド正常化**を検証する。

---

## 1. Overview
Step 3 では参照系のみを公開する。UI からの書き込みユースケースは限定的（栄養士のメニュー登録のみ・本ステップでは対象外）であるため、**大規模な機能障害リスクは低い**。
DBが従来のDBと同じであるため、新旧環境の突合によるテストは不要。

テストの主眼は次の 3 点:
- **E2E 経路**の健全性 (Browser → API Gateway → CamelAPI → 旧DB Read)
- **Contract/CDC** による API 仕様の固定化と UI との合意
- **テストピラミッドの正常化** (E2E 過多から、Unit/Integration へ重心シフト)

---

## 2. Test Types & Scope

### 2.1 Contract / CDC (最優先)
- **APIスキーマ**: OpenAPI を single source とし、Gateway ルーティング／UI クライアントを同一契約で検証。
- **Consumer-Driven Contracts**: CanteenUI/DedUI からの期待を Pact などで宣言し、CamelAPI のビルドで検証。
- **後方互換性**: 破壊的変更は versioned path で提供。ヘッダーフラグの AB 切替は不可避時のみ。

### 2.2 Integration (CamelAPI)
- DB Read-Only 経路: リポジトリ層のクエリ、ページング、ソート、フィルタの境界値。
- 認可フィルタ: トークン中の claim に応じた行・列フィルタ (必要最小限)。
- エラーハンドリング: タイムアウト、部分不達 (fallback 200 + 空配列/既定メッセージ) を合意。

### 2.3 Security / AuthN & AuthZ
- **トークン伝播**: Browser → UI → GW → CamelAPI の**スコープ/オーディエンス**検証。
- **RBAC**: 一般社員と栄養士ロールでの可視範囲差を確認 (本ステップは参照のみ)。
- **PII最小化**: API レスポンスの過剰属性を削減。監査でマスキングを確認。

### 2.4 E2E (軽量・重要経路のみ)
- 代表 3 経路: 
  1) 社食メニュー参照、2) 自分の明細参照 (DedUI)、3) エンプティ状態。
- ブラウザ互換: 最新 2 メジャー (Chromium/Firefox/Safari)。
- アクセシビリティ smoke: キーボード操作、主要ARIA、コントラスト。

### 2.5 Non-Functional
- **遅延**: 95p ≤ 300ms (GW→CamelAPI→DB→GW の往復) を目安。ピーク (12:00–13:00) で計測。
- **可用性**: Gateway で 99.9%/月 (参照系)。
- **可観測性**: Correlation-ID による trace 連鎖を収集 (UI→GW→Camel)。

---

## 3. Data Validation (参照系)
- 旧画面と新UIの**表示整合性**をスナップショット比較 (主要 10 画面)。
- **境界データ**: 大量メニュー日／割引適用日／データ欠損日で比較。
- 一致条件は **業務的同値**: 並び順・文言差は許容、金額・件数・合計は厳密一致。

---

## 4. Load & Peak Monitoring (Live)
- 本番デプロイ後 **3 営業日**, 11:30–13:30 をライブ観測。
- 監視メトリクス:
  - GW: 5xx/4xx 率、p95/p99 レイテンシ、サーキットブレーカ発火。
  - CamelAPI: スループット、DB プール使用率、タイムアウト率。
  - DB: CPU、IO Wait、クエリ p95、ロック競合。
- **閾値越え時の運用手順**: ルールベースでスロットリング→キャッシュ TTL 延長→旧画面フォールバック。

---

## 5. Release Strategy
- **Dark Launch**: URL 非公開で先行稼働、社内テスターのみ利用。
- **Rollback**: GW ルールで即時旧UIへ切替。データは参照系のため**巻き戻し不要**。

---

## 6. Success Criteria
- Contract/CDC が CI で恒常緑 (破壊的変更ゼロ)。
- ピーク帯の p95 レイテンシが閾値内、重大アラート 0。
- 主要 10 画面で業務的同値が満たされる (金額/件数/合計一致)。
- セキュリティ: 未認可アクセス 0、過剰露出属性 0。

---

## 7. Constraints & Dependencies
- 本ステップは **参照系のみ**。メニュー登録等の書き込みは次フェーズ。
- 旧 DB スキーマに依存するため、**スキーマ変化は逆マイグレーション層** (DB View想定) で吸収。
- HR 側利用明細 API (Step 1/2) の健全性に依存 (DedUI 表示)。

---

## 8. Observability & Reporting
- ダッシュボード: UI→GW→Camel→DB の**縦串トレース**を 1 画面に集約。
- レポート: デプロイ翌営業日から 3 日間、昼帯の**運用サマリ**を共有 (SRE/EA/開発)。

---

## 9. References
- step-1-testing.md / step-2-testing.md
- adr-0007 Observability / adr-0012 Testing & Quality / principle-003 EDA / principle-007 Observability
