# autoware_tracking_object_merger 概要

## 概要

`autoware_tracking_object_merger` は、**異なるセンサで追跡（Tracking）された物体情報を統合するための Perception モジュール**です。

LiDAR・Radar・Camera など複数のセンサで追跡された物体を対応付け（データアソシエーション）し、それぞれの長所を活かしながら1つの追跡結果へ統合します。これにより、より安定した物体追跡を実現します。 :contentReference[oaicite:0]{index=0}

---

# 主な役割

このパッケージの役割は以下のとおりです。

- 複数センサの追跡結果を受け取る
- 同じ物体同士を対応付ける
- 各センサの情報を統合する
- より信頼性の高い追跡結果を生成する
- 後続の Prediction モジュールへ渡す :contentReference[oaicite:1]{index=1}

---

# 処理の流れ

```text
LiDAR Tracker
        │
Radar Tracker
        │
Camera Tracker
        │
        ▼
Tracking Object Merger
        │
        ├─ 時刻同期
        ├─ データ対応付け
        └─ 状態統合
        │
        ▼
Merged TrackedObjects
```

---

# 入力

主な入力は追跡済みの物体情報です。

| 入力 | 型 |
|------|----|
| Main TrackedObjects | `autoware_perception_msgs/msg/TrackedObjects` |
| Sub TrackedObjects | `autoware_perception_msgs/msg/TrackedObjects` |

通常は

- Main：LiDAR Tracker
- Sub：Radar または Camera Tracker

という構成で利用されます。 :contentReference[oaicite:2]{index=2}

---

# 出力

主な出力は

- `TrackedObjects`

です。

各センサの情報を統合した追跡結果が出力され、後続の Prediction や Planning モジュールで利用されます。 :contentReference[oaicite:3]{index=3}

---

# 主な処理

このモジュールでは、大きく次の処理を行います。

### 時刻同期

センサごとに更新周期が異なるため、追跡結果のタイムスタンプを同期します。

---

### データ対応付け

異なるセンサで検出された物体が同じ対象かどうかを判定します。

判定には次のような情報が利用されます。

- 距離
- 向き
- IoU（重なり具合）
- マハラノビス距離
- 速度差

これらを用いて、同一物体である可能性を評価します。 :contentReference[oaicite:4]{index=4}

---

### 状態統合

対応付けられた物体について、各センサの得意な情報を組み合わせます。

例えば、

- LiDAR：位置や形状
- Radar：速度
- Camera：物体分類

など、それぞれの特徴を活かした統合が行われます。 :contentReference[oaicite:5]{index=5}

---

# マージ方式

現在は主に次のようなマージ方式が用意されています。

## decorative_tracker_merger

メインとなる追跡結果に対して、別のセンサの情報を補完する方式です。

例

```text
LiDAR
   │
   ▼
Main Tracker
   ▲
   │
Radar
```

LiDAR を基準とし、不足する情報を Radar や Camera が補います。 :contentReference[oaicite:6]{index=6}

---

## equivalent_tracker_merger

複数センサを同等に扱い、追跡結果を統合する方式です。

用途に応じて適切なマージポリシーを選択できます。 :contentReference[oaicite:7]{index=7}

---

# 特徴

- 複数センサの追跡結果を統合
- データアソシエーションを実施
- センサごとの長所を活用
- 時刻同期に対応
- より安定した Tracking を実現 :contentReference[oaicite:8]{index=8}

---

# 利用される場面

`autoware_tracking_object_merger` は、

- LiDAR と Radar の融合
- LiDAR と Camera の融合
- マルチセンサ追跡
- 高精度な物体追跡

などで利用されます。 :contentReference[oaicite:9]{index=9}

---

# 他モジュールとの関係

```text
LiDAR Tracker
        │
Radar Tracker
        │
Camera Tracker
        ▼
autoware_tracking_object_merger
        │
        ▼
TrackedObjects
        │
        ▼
Prediction
        │
        ▼
Planning
```

追跡結果を統合することで、後続モジュールがより信頼性の高い物体情報を利用できるようになります。

---

# object_merger との違い

`autoware_object_merger` は**検出（DetectedObjects）**を統合するモジュールであるのに対し、`autoware_tracking_object_merger` は**追跡済みの物体（TrackedObjects）**を統合するモジュールです。

| モジュール | 対象 |
|------------|------|
| `autoware_object_merger` | DetectedObjects（検出結果） |
| `autoware_tracking_object_merger` | TrackedObjects（追跡結果） |

そのため、`autoware_tracking_object_merger` は Tracking の後段で利用されます。 :contentReference[oaicite:10]{index=10}

---

# まとめ

`autoware_tracking_object_merger` は、**複数センサで追跡された物体情報を統合するための Perception モジュール**です。

主な役割は、**LiDAR・Radar・Camera などから得られた追跡結果を対応付けて統合し、より正確で安定した TrackedObjects を生成すること**です。これにより、Prediction や Planning モジュールは、信頼性の高い追跡結果を利用して後続の処理を行うことができます。 :contentReference[oaicite:11]{index=11}