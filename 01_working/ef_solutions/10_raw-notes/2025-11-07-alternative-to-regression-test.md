---
date: 2025-11-07
tags: [raw]
---

# Raw Note

## 気づき (Observation)  
- リグレッションテストの代替案として、新旧の並行稼働を使ったテストの実施が考えられる。
  - 参考) STAR by Mike Amundsen (https://mamund.site44.com/talks/2021-05-orm-migration/2021-05-orm-migration.pdf)
  - 旧環境の処理結果が割引部分を除いて正しいのであれば、新旧の並行稼働を実施し、パケットミラーリングやそれに類する方法で同じデータを新旧に流す。処理結果をDBから抜いてきて比較して突き合わせ、格納された処理結果ベースで比較。
- OpenShiftや監視ツール、API Gateway、Service Meshのメトリックを使い、デプロイメント直後は密な監視を実施
  - 基本ダークローンチ、ユーザー公開とはタイミングをずらす
  - Canary Releaseも活用する


## 疑問 (Question)
- 特になし

## 次アクション (Next Step)
- 移行アプローチの各断面でこのアプローチが成立するかを確認しながらTransition Architectureを作ってみる。
  - 成立しなさそうなら他の方法を考える