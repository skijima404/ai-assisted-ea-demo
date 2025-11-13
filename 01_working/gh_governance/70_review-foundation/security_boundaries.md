# Security Boundaries (Draft)

このドキュメントは、アーキテクチャレビューおよび CI/GenAI チェックで参照する  
**「壊してはいけないセキュリティ境界（Security Boundaries）」** を定義します。  
デモ用として、一般的なクラウドネイティブ構成に基づく軽量なインバリアントセットです。

---

## 1. Purpose

- “どこからどこまでが安全か” を明確にし、境界を越える際のリスクを可視化する
- Public / Private / Data の各境界に対して、**破ってはいけない最小限のルール** を定義する
- AIレビュー時に、今回の差分が境界を侵食していないかを判断する前提コンテキストとなる

---

## 2. Boundary Model Overview

本プロジェクトでは、以下の3つの境界を基本とする：

1. **PUBLIC**  
   - インターネット等から直接到達可能な領域  
   - Gateway / API Edge などが該当

2. **PRIVATE-APP**  
   - サービスメッシュ／VPC内のアプリケーション間通信  
   - 認証済みのアプリ間トラフィックを想定

3. **DATA**  
   - 永続化層（DB/KVS/オブジェクトストア）  
   - Public からの直接アクセスは禁止

---

## 3. Invariants (Security Invariants)

### **INV-SEC-001: PUBLIC → DATA の直アクセス禁止**

- Public から Data ストアへの直接接続は禁止  
- 経路は必ず Application 層（PRIVATE-APP）を経由すること  
-  
> Check例  
> - Public Gateway から DB 接続設定がないか  
> - Public API から Data 層フィールドが漏れていないか

---

### **INV-SEC-002: PUBLIC 境界は必ず認証・レートリミットを通過する**

- PUBLIC にある API / Endpoint は必ず AuthN 経由で保護される  
- レートリミットなしの公開不可  
-  
> Check例  
> - 新規 Public API の AuthN 設定の有無  
> - レートリミットをまたぐ bypass 経路がないか

---

### **INV-SEC-003: PRIVATE 層は最小権限（Least Privilege）でアクセスする**

- PRIVATE-APP 層の各サービスは、必要最小限のアクセス権で動作する  
- 「なんでもできる ServiceAccount」の禁止  
-  
> Check例  
> - RBAC が broad 権限になっていないか  
> - Sidecar/Proxy 経由で権限昇格する経路がないか

---

### **INV-SEC-004: Sensitive / PII は PUBLIC から直接見えないこと**

- PII/Sensitive は API の Response に直接出さない  
- 内部IDと外部IDを分離する  
-  
> Check例  
> - OpenAPI の Response に PII が含まれていないか  
> - Entity モデルがそのまま外部公開されていないか

---

### **INV-SEC-005: Service-to-Service 呼び出しは信頼境界を越える場合に理由を要する**

- PRIVATE-APP を跨ぐ依存は、コンテキスト境界に明確な理由が必要  
- “便利だから直接呼ぶ” は不可逆リスク（境界侵食）の温床  
-  
> Check例  
> - 新規依存の追加が Domain Boundary を破っていないか  
> - ms-design-canvas と照合して想定外の依存がないか

---

## 4. How Used in Review

- GenAIレビュー時に  
  - 差分コード／OpenAPI／Manifest がどの境界に影響するか  
  - どれかの SECURITY INVARIANT を破っているか  
  を確認させるためのコンテキストとして使用

- CIでは  
  - Public API のスキーマ変更  
  - RBAC / Deployment 設定  
  - Data アクセス  
  を対象に静的チェック可能な項目を拾う

---

## 5. Notes

- 本ドキュメントはデモ用の軽量版。  
- 実案件ではシステム固有の境界（例：社内VPN/取引先API/IDP）を追加する。