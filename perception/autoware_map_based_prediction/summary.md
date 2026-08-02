# autoware_map_based_prediction

## 概要

`autoware_map_based_prediction` は、追跡された周囲の車両や歩行者、自転車などが**今後どのような経路を進むか**を予測するパッケージです。

AutowareのPredictionモジュールに位置付けられており、Trackingモジュールで検出・追跡された物体とHDマップ（Lanelet2）を利用して、各物体の将来の走行経路とその確率を推定します。:contentReference[oaicite:0]{index=0}

---

# 主な役割

`autoware_map_based_prediction` の役割は次のとおりです。

- Trackingされた物体を受け取る
- Lanelet2マップから走行可能なレーンを検索する
- 車線変更や直進などの挙動を推定する
- 将来の走行経路（Predicted Path）を生成する
- 複数の経路候補に確率を付与する
- Planningモジュールへ予測結果を渡す

---

# Predictionモジュールとは

Autowareでは、

- **Detection**：物体を見つける
- **Tracking**：同じ物体を追跡する
- **Prediction**：この先どこへ進むか予測する

という役割分担になっています。

例えば前方車両が交差点へ近付いている場合、

- 直進する
- 左折する
- 右折する

など複数の可能性を推定し、それぞれに確率を付けて出力します。:contentReference[oaicite:1]{index=1}

---

# 処理の流れ

```text
TrackedObjects
       │
       ▼
Lanelet2マップ検索
       │
       ▼
現在走行中のレーン推定
       │
       ▼
過去の移動履歴を解析
       │
       ▼
将来の経路を生成
       │
       ▼
PredictedObjects出力
```

---

# 1. Tracking結果の入力

入力されるのはTrackingモジュールが出力した

```
TrackedObjects
```

です。

各物体には

- 位置
- 速度
- 向き
- 種類（車・歩行者など）

といった情報が含まれています。:contentReference[oaicite:2]{index=2}

---

# 2. マップとの対応付け

物体の現在位置をLanelet2マップへ対応付けます。

例えば

```
        車
──────────────
    現在位置
──────────────
```

のように、

「どの車線を走っているか」

を判定します。

車線情報が得られることで、

- この先の道路形状
- 分岐
- 合流
- 交差点

などを考慮した予測が可能になります。:contentReference[oaicite:3]{index=3}

---

# 3. 過去の移動履歴を利用

現在位置だけではなく、

過去数秒間の移動履歴も保持しています。

これにより、

- 車線変更中か
- 直進中か
- 曲がろうとしているか

などを判断します。:contentReference[oaicite:4]{index=4}

---

# 4. 将来経路の生成

推定されたレーンに沿って、

将来の走行経路を生成します。

例えば交差点では

```
        ↑
      40%

←30%     30%→
```

のように

- 左折
- 直進
- 右折

それぞれの経路候補を生成し、

各経路に確率を付与します。:contentReference[oaicite:5]{index=5}

---

# 歩行者の予測

歩行者や自転車については、

- 横断歩道
- 歩道
- 進行方向

などを利用して進路を予測します。

横断歩道付近では、

道路を横断する可能性も考慮されます。:contentReference[oaicite:6]{index=6}

---

# ROS2ノード

主要ノードは

```
map_based_prediction_node
```

です。

Trackingモジュールの後段で実行されます。:contentReference[oaicite:7]{index=7}

---

# 入力

| Topic | 型 | 内容 |
|--------|----|------|
| `~/input/objects` | `autoware_perception_msgs/msg/TrackedObjects` | Tracking済み物体 |
| `~/vector_map` | `autoware_map_msgs/msg/LaneletMapBin` | Lanelet2 HDマップ |
| `~/perception/traffic_light_recognition/traffic_signals` | `autoware_perception_msgs/msg/TrafficLightGroupArray` | 信号情報 | :contentReference[oaicite:8]{index=8}

---

# 出力

| Topic | 型 | 内容 |
|--------|----|------|
| `~/output/objects` | `autoware_perception_msgs/msg/PredictedObjects` | 将来経路付き物体 |
| `~/objects_path_markers` | `visualization_msgs/msg/MarkerArray` | RViz表示用マーカー |
| `~/debug/processing_time_ms` | `std_msgs/msg/Float64` | 処理時間 |
| `~/debug/cyclic_time_ms` | `std_msgs/msg/Float64` | 実行周期 | :contentReference[oaicite:9]{index=9}

---

# ディレクトリ構成

```text
autoware_map_based_prediction/
├── config/        # パラメータ
├── include/       # ヘッダ
├── launch/        # Launchファイル
├── schema/        # パラメータスキーマ
├── src/           # 予測アルゴリズム
├── test/          # テスト
├── CMakeLists.txt
├── package.xml
└── README.md
```

---

# Autoware内での位置付け

Perception全体では次のような流れになります。

```text
LiDAR・Camera
      │
      ▼
Object Detection
      │
      ▼
Multi Object Tracker
      │
      ▼
Map Based Prediction
      │
      ▼
Behavior Planner
      │
      ▼
Motion Planner
```

Predictionモジュールは、Planningが安全な経路を生成するために重要な役割を担っています。

---

# このモジュールの特徴

- Lanelet2マップを利用した高精度な経路予測
- 車線変更の検出
- 複数の経路候補を生成
- 経路ごとに確率を付与
- 車両だけでなく歩行者・自転車にも対応
- Planningモジュールで利用しやすい `PredictedObjects` を出力 :contentReference[oaicite:10]{index=10}

---

# まとめ

`autoware_map_based_prediction` は、追跡された物体が今後どこへ進むかを予測するPredictionモジュールです。

HDマップ（Lanelet2）と物体の移動履歴を組み合わせることで、直進・右左折・車線変更など複数の経路候補を生成し、それぞれに確率を付与します。

この予測結果はPlanningモジュールへ渡され、自車が周囲の交通参加者の動きを考慮した安全な経路を生成するために利用されます。:contentReference[oaicite:11]{index=11}

---

# 参考

- Autoware Universe Documentation: Map Based Prediction :contentReference[oaicite:12]{index=12}
- GitHub: https://github.com/autowarefoundation/autoware_universe/tree/main/perception/autoware_map_based_prediction