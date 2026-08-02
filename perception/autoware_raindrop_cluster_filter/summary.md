# autoware_raindrop_cluster_filter

## 概要

`autoware_raindrop_cluster_filter` は、LiDARで検出された物体クラスタのうち、**雨粒や水しぶきなどによる誤検出を除去するためのフィルタ**です。

雨天時には、LiDARのレーザーが雨粒や路面から跳ね上がった水滴に反射し、小さな物体として誤って検出されることがあります。このモジュールは、**点群の反射強度（Intensity）** を利用して、そのようなノイズを除去します。

物体を新たに検出するモジュールではなく、検出済みオブジェクトの品質を向上させる後処理モジュールとして利用されます。 :contentReference[oaicite:0]{index=0}

---

# 主な役割

このモジュールでは主に以下の処理を行います。

- 点群クラスタの平均Intensityを計算
- Intensityが極端に低いクラスタを除外
- 雨粒・水しぶきによる誤検出を削減
- 必要に応じて対象ラベル（UNKNOWNなど）のみフィルタリング

これにより、悪天候時でも不要な物体検出を減らし、後続の認識精度向上に貢献します。 :contentReference[oaicite:1]{index=1}

---

# 入力

主な入力は以下です。

| Topic | 型 | 内容 |
|-------|----|------|
| `input/object` | `DetectedObjectsWithFeature` | 検出済み物体（点群特徴付き） |

各物体には対応する点群クラスタが含まれており、そのIntensity情報を利用して判定を行います。 :contentReference[oaicite:2]{index=2}

---

# 出力

| Topic | 型 | 内容 |
|-------|----|------|
| `output/object` | `DetectedObjectsWithFeature` | フィルタ後の物体一覧 |

低Intensityと判断されたクラスタは除外され、それ以外の物体のみが後続モジュールへ送られます。 :contentReference[oaicite:3]{index=3}

---

# 処理の流れ

```text
LiDAR PointCloud
       │
       ▼
物体検出
       │
       ▼
DetectedObjectsWithFeature
       │
       ▼
autoware_raindrop_cluster_filter
       │
       ▼
不要クラスタ除去
       │
       ▼
Filtered DetectedObjectsWithFeature
```

---

# 主な設定項目

代表的なパラメータは以下です。

| パラメータ | 内容 |
|------------|------|
| `intensity_threshold` | 平均Intensityの閾値 |
| `existence_probability_threshold` | フィルタを適用する存在確率の閾値 |
| `max_x` / `min_x` | フィルタを適用するX方向範囲 |
| `max_y` / `min_y` | フィルタを適用するY方向範囲 |
| `filter_target_label.*` | フィルタ対象のラベル（UNKNOWN、CARなど） |

通常は `UNKNOWN` ラベルのみを対象にする設定が推奨されており、車両や歩行者など既知クラスへの適用は必要に応じて変更できます。 :contentReference[oaicite:4]{index=4}

---

# 他モジュールとの関係

このモジュールは物体検出の後段に配置されます。

```text
PointCloud
      │
      ▼
Ground Segmentation
      │
      ▼
Object Detection
      │
      ▼
autoware_raindrop_cluster_filter
      │
      ▼
Object Validation
      │
      ▼
Object Tracker
```

検出器が出力したオブジェクトから、雨や水しぶきによるノイズを取り除いてから、追跡やセンサフュージョンへ渡します。 :contentReference[oaicite:5]{index=5}

---

# まとめ

- 雨粒や水しぶきによるLiDARの誤検出を除去するフィルタ
- 点群の **Intensity（反射強度）** を利用して判定
- 検出済みオブジェクトの後処理として動作
- 特に `UNKNOWN` オブジェクトのノイズ低減に効果がある
- 雨天時のPerceptionの安定性向上を目的とした補助モジュール