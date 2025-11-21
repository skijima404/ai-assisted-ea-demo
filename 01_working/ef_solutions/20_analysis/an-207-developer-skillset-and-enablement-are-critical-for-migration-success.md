---
id: an-207
date: 2025-11-07
title: developer-skillset-and-enablement-are-critical-for-migration-success
status: draft
tags: [analysis, enablement, skills, pfe]
---

# Developer skillset and enablement are critical for migration success

## Incident (Observed Behavior)
- ef_solutions の Target Architecture（イベント駆動・GitOps・マイクロサービス・コントラクトテスト等）は、現行の一般的な開発案件と比較して、かなり高いスキルセットを前提としている。
- raw-notes では、必要とされるスキルとして以下が挙げられている：
  - GitOps / CI/CD パイプライン運用
  - IaC（Infrastructure as Code）
  - コントラクトテスト
  - マイクロサービス設計原則
- 組織として調達可能な開発者のスキルセットはまだ不明であり、そのギャップをどう埋めるかが課題となっている。
- Enablement（教育・トレーニング）と、Platform Engineering による認知負荷の軽減（テンプレート化・標準化）が必要との示唆が raw-notes にある。
- 負荷の高い前提を鑑み、`CSV → Kafka` のようなスコープの小さい変更を初手とすることが Learning Curve の観点からも望ましいとされている。

## Conclusion / Assumptions
- **結論:** ef_solutions のようなモダンアーキテクチャへの移行は、開発者スキルセットと Enablement なしには成立せず、同時に Platform Engineering による認知負荷軽減（テンプレート・標準化）が必須となる。これらを前提条件として明示しない移行計画は、実行フェーズで失速するリスクが高い。
- **Assumptions:**
  - 現行組織の平均的な開発スキルセットは、Kafka・GitOps・マイクロサービス・コントラクトテストなどを前提とした継続運用にはまだ十分ではない。
  - 外部パートナーだけでなく、内製側にも一定レベルのスキル蓄積が必要である。
  - PFE（Platform Engineering）により、標準的なパイプライン・Observability・デプロイ方法をテンプレート化することで、必要なスキルの一部は「プラットフォーム側に吸収」できる。

## Reasoning
1. **モダンアーキテクチャの運用には、高スキルセットが“常態として”求められる。**  
   Kafka・イベント駆動・マイクロサービス・GitOps などは、一度導入して終わりではなく、継続的に運用するための知識・スキルが不可欠である。

2. **スキルギャップを放置したまま移行を進めると、「新アーキテクチャを使いこなせない」状態に陥る。**  
   - 形だけ Kafka や GitOps を導入しても、問題発生時に原因究明や改善ができない。
   - テンプレートや自動化がなければ、開発者一人ひとりの負担が大きくなり、品質・スピードともに不安定になる。

3. **Enablement により、最低限必要なスキルセットを“意図的に引き上げる”必要がある。**  
   - GitOps, IaC, コントラクトテスト, マイクロサービス設計原則などは、ワークショップやハンズオンを通じて実案件に紐づけて学ぶ方が効果的。
   - 単発トレーニングではなく、移行ステップと連動した Enablement 計画が必要。

4. **Platform Engineering（PFE）は、開発者が学ばなくてもよい領域を隠蔽する役割を担う。**  
   - CI/CD テンプレート、Observability 標準、デプロイ方法などをプラットフォーム側で統一することで、個々の開発者が覚えるべきことを減らせる。
   - これにより、高度なアーキテクチャでも「現場の認知負荷」を抑えた運用が可能になる。

5. **初手を `CSV → Kafka` のような小さな変更にするのは、Learning Curve を考慮した現実的な戦略である。**  
   - Kafka の基本（トピック設計・スキーマ管理・監視）を小さなスコープで学べる。
   - 開発チームは、いきなりフルマイクロサービスではなく、既存との橋渡しから始められる。

## Implication
- 移行計画・Architecture Vision には、以下を明記すべきである：
  - 「必要な開発者スキルセット」と「現状のギャップ」の認識。
  - Enablement（トレーニング・ハンズオン）の具体的なプラン。
  - PFE チーム（またはそれに相当する機能）が提供するテンプレート／標準の範囲。
- 技術的な To-Be だけでなく、「人と組織がどこまでついてこられるか」を設計に織り込む必要がある。
- 初期ステップとしては、
  - `CSV → Kafka` のような小さな変更、
  - それを通じた Kafka / GitOps 実践、
  をセットで計画するのが望ましい。

## Observations
- ef_solutions の Target Architecture は、GitOps, IaC, コントラクトテスト, マイクロサービス設計原則など、比較的高度なスキルセットを前提としている。
- raw-notes では、
  - 「開発者のスキルセットを事前に確認すべき」、
  - 「必要に応じて Enablement とアプリケーションプラットフォーム整備（PFE）を行うべき」、
  - 「要求が高いからこそ、CSV → Kafka のような小さな変更から始めるのが優しい」、
  といった示唆が記録されている。

## Evidence
- `01_working/ef_solutions/10_raw-notes/2025-11-07-agile-development.md`
