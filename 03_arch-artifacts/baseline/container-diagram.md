---
title: Baseline Architecture - Container Diagram
last_updated: 2025-11-06
---

```mermaid
flowchart TB

  %% Scope markers
  noteOverStart[/"Scope-In Start:\n決済取込完了後"/]

  User[社員 (End User)]
  Browser[ブラウザ]

  subgraph Cafeteria["社食システム (Monolithic Application)"]
    App[Monolith App\n(決済履歴/ユーザープロファイル/メニュー&栄養素/利用明細/請求台帳 ほか)]
    DB[(社食DB)]
    Ingest[(決済取込\n(外部からの投入点))]:::ing
  end

  subgraph HR["人事システム (External Monolith)"]
    HRApp[HR App]
    HRDB[(人事DB)]
  end

  External[コンビニシステム (External)]

  %% interactions (minimal)
  User --> Browser --> App
  Ingest --> App
  App --> DB

  %% External batch (CSV)
  App -- "月次CSV（締め後）\n清算用: 社食→人事（給与天引き予定）" --> HRApp
  External -- "月次CSV（締め後）\n清算用: コンビニ→人事（給与天引き予定）" --> HRApp
  External -- "日時CSV（夜間）\nレシート明細: コンビニ→社食App（表示用）" --> App
  HRApp -- "日時CSV（夜間）\n社員マスタ/所属 更新: 人事→社食App（ID同期）" --> App

  %% cosmetics
  classDef ing fill:#eef7ff,stroke:#1b6dd1,rx:6,ry:6;
  ```
