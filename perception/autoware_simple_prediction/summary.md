# autoware_simpl_prediction

## 概要

`autoware_simpl_prediction` は、**機械学習（Deep Learning）を用いて周囲車両や歩行者などの将来の移動経路を予測するモジュール**です。

Autowareでは物体検出・物体追跡の後に「この物体が今後どのように動くか」を予測する必要があります。本モジュールでは、**SIMPL（Simple and Efficient Multi-agent Motion Prediction）** と呼ばれるAIモデルを利用し、周囲環境や地図情報を考慮した複数の将来軌道を推定します。

従来のルールベース予測よりも複雑な交通状況へ対応できることが特徴です。 :contentReference[oaicite:0]{index=0}

---

# 主な役割

このモジュールでは主に以下の処理を行います。

- 周囲車両・歩行者の過去の移動履歴を取得
- Lanelet2地図から道路形状を取得
- 自車位置との相対関係を入力データとして生成
- AIモデル（SIMPL）による将来軌道予測
- 複数の候補経路とその信頼度を出力

物体を検出するモジュールではなく、**追跡済み物体の将来の動きを予測する**ことが目的です。 :contentReference[oaicite:1]{index=1}

---

# 入力

主な入力は以下です。

| Topic | 型 | 内容 |
|-------|----|------|
| `~/input/objects` | `TrackedObjects` | 追跡済み物体 |
| `~/input/vector_map` | `LaneletMapBin` | HDマップ（Lanelet2） |
| `/localization/kinematic_state` | `nav_msgs/msg/Odometry` | 自車の位置・速度 |

過去の移動履歴と道路形状を組み合わせることで、より現実的な経路を予測します。 :contentReference[oaicite:2]{index=2}

---

# 出力

| Topic | 型 | 内容 |
|-------|----|------|
| `~/output/objects` | `PredictedObjects` | 将来軌道付き物体情報 |

各物体には複数の予測経路（Predicted Path）が付与され、それぞれに発生確率（信頼度）が含まれます。 :contentReference[oaicite:3]{index=3}

---

# 処理の流れ

```text
TrackedObjects
       │
       │
Lanelet2 Map
       │
       │
Ego Odometry
       │
       ▼
autoware_simpl_prediction
       │
       ▼
SIMPL AI Model
       │
       ▼
PredictedObjects
```

---

# AIモデル（SIMPL）

本モジュールでは **SIMPL（Simple and Efficient Multi-agent Motion Prediction）** を利用しています。

SIMPLは以下の情報を入力として使用します。

- 過去の移動履歴
- 周囲車両との位置関係
- 道路形状
- 自車との相対位置

これらをニューラルネットワークへ入力し、複数の将来軌道とその確率を推定します。

TensorRTによる高速推論に対応しており、自動運転で求められるリアルタイム性能を実現しています。 :contentReference[oaicite:4]{index=4}

---

# 主な設定項目

代表的なパラメータは以下です。

| パラメータ | 内容 |
|------------|------|
| `detector.onnx_path` | ONNXモデルのパス |
| `detector.engine_path` | TensorRT Engineのパス |
| `detector.precision` | 推論精度（FP32 / FP16 / INT8） |
| `preprocess.max_num_agent` | 最大予測対象数 |
| `preprocess.num_past` | 使用する過去フレーム数 |
| `postprocess.num_future` | 予測する未来フレーム数 |
| `postprocess.num_mode` | 出力する予測経路数 |
| `postprocess.score_threshold` | 信頼度の閾値 |

推論モデルや予測対象数、予測時間などを用途に応じて調整できます。 :contentReference[oaicite:5]{index=5}

---

# 他モジュールとの関係

`autoware_simpl_prediction` は Prediction パイプラインの中心となるモジュールです。

```text
Object Detection
        │
        ▼
Multi Object Tracker
        │
        ▼
TrackedObjects
        │
        ▼
autoware_simpl_prediction
        │
        ▼
PredictedObjects
        │
        ▼
Behavior Planner
        │
        ▼
Motion Planner
```

予測された軌道は、行動計画（Behavior Planning）や経路生成（Motion Planning）で利用され、自車が安全な経路を選択するための重要な情報となります。 :contentReference[oaicite:6]{index=6}

---

# 特徴

- AI（SIMPL）による高精度な移動予測
- TensorRTによる高速推論
- Lanelet2地図を利用した道路に沿った予測
- 複数の将来軌道と信頼度を同時に出力
- 車両・歩行者・自転車など複数の対象に対応

---

# まとめ

- AIモデル **SIMPL** を利用した将来軌道予測モジュール
- 追跡済み物体・HDマップ・自車情報を入力として利用
- TensorRTによるリアルタイム推論に対応
- 複数の将来経路とその信頼度を出力
- Behavior Planner や Motion Plannerへ将来軌道を提供する Prediction モジュール