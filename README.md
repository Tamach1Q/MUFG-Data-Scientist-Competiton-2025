# MUFG Data Science Challenge 2025 Basic Camp

このリポジトリは、**MUFG Data Science Challenge 2025 / Basic Camp** に取り組んだ際の実験コードをまとめたものです。  
メイン実装は `MUFGCompetition.ipynb` にあります。

本取り組みでは、**Basic Camp 部門で 18 位** でした。

## コンペ概要

MUFG Data Science Challenge 2025 は、三菱UFJフィナンシャル・グループが開催した学生向けデータサイエンスコンペティションです。  
公開情報では、今からデータ分析を学びたい層から既に知識を持つ層まで幅広く参加できる形式で、**Basic Camp** と **Champion Ship** の 2 部門が用意されていました。

Basic Camp については、公開ページで次のように案内されています。

- 対象: 初中級者
- テーマ: **中小企業の債務返済の判定**
- 形式: オンライン
- 実施期間: **2025年8月1日 から 2025年9月2日**
- 学習コンテンツ: あり
- データ特性: テーブルデータ、不均衡データ

参考リンク:

- コンペページ: <https://user.competition.signate.jp/ja/competition/detail/?competition=693aed0f148a4bcdb2cfffa85d1588b2>
- MUFG公式案内: <https://www.saiyo.bk.mufg.jp/datascience/>

## リポジトリの概要

このリポジトリでは、主に以下を管理しています。

- `MUFGCompetition.ipynb`: 前処理、特徴量生成、学習、評価、提出ファイル生成までをまとめたメインノートブック
- `submissions/`: 提出用CSVの保存先
- `submit_probs.csv` / `submit_1.csv`: 生成済み提出ファイルの一部

`train.csv` と `test.csv` は GitHub には含めていません。

## アプローチ

この notebook では、既存の前処理と学習フローをベースに、複数モデルを比較しながら最終提出を作る構成にしています。

### 1. 前処理

- 目的変数列と ID 列を推定
- 日付らしい列を自動検出して、年・月・日・曜日・時間に展開
- 高 cardinality なカテゴリ列を除外
- 残った文字列列を `category` 型へ変換

### 2. 特徴量生成

- `BusinessAge` の順序特徴量化
- `CollateralInd`、`FixedOrVariableInterestInd` の二値化
- 比率・派生特徴量の追加
  - `SBA_Portion`
  - `Approval_per_Job`
  - `Years_to_Pay`
- 集約特徴量の追加
  - キー: `NaicsSector`, `BusinessAge`, `Subprogram`, `BusinessType`
  - 対象: `InitialInterestRate`, `GrossApproval`, `SBA_Portion`, `TermInMonths`
  - 平均、標準偏差、最大、最小、および平均との差分・比率

### 3. 学習

主に以下のモデルを利用しています。

- **LightGBM**
  - Optuna によるハイパーパラメータ探索
  - 一部主要カテゴリ列に対して平滑化付き Target Encoding
- **CatBoost**
  - カテゴリ変数をそのまま扱う構成
  - Optuna によるハイパーパラメータ探索
- **XGBoost**
  - One-Hot Encoding 後に学習
  - Optuna によるハイパーパラメータ探索

### 4. アンサンブル

ノートブック内では、単体モデルだけでなく次の構成も試しています。

- LightGBM + CatBoost の加重平均
- LightGBM / CatBoost / XGBoost のスタッキング
- 高信頼予測を使った疑似ラベル付け

### 5. 評価と提出

- OOF 予測を用いて評価
- しきい値は `F1` ベースで探索
- 確率提出用ファイルと、ラベル提出用ファイルを出力

## 実行環境

ノートブックは **Google Colab** を前提に作成しています。  
冒頭で `optuna`, `catboost`, `xgboost` をインストールし、Google Drive 上の CSV を読む構成です。

必要に応じて、以下のパスを自分の環境に合わせて変更してください。

- `TRAIN_PATH`
- `TEST_PATH`
- `SUB_PATH`
- `LABEL_SUB_PATH`

## メモ

- このリポジトリは、コンペ参加時の試行錯誤を残すことを目的にしています。
- notebook には単体モデル、アンサンブル、スタッキング、疑似ラベル付けなど複数の実験系パイプラインが含まれています。
- 公開時点では、データ本体は含めず、コードと提出物の一部のみを管理しています。
