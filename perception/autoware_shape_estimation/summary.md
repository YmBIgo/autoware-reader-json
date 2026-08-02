# autoware_shape_estimation

## 概要

`autoware_shape_estimation` は、物体検出によって得られた点群クラスタから、**より正確な3D形状（Shape）を推定するモジュール**です。

物体検出器は「この位置に車両がある」「歩行者がいる」といった情報を出力しますが、そのままでは物体の大きさや向きが十分に正確でない場合があります。

このモジュールでは、**点群クラスタと物体ラベル**を利用して、物体に最適な形状（バウンディングボックス・円柱・凸包など）を推定し、後続のトラッキングや経路計画で利用しやすい形へ補正します。:contentReference[oaicite:0]{index=0}

---

# 主な役割

このモジュールでは主に以下の処理を行います。

- 点群クラスタから物体形状を推定
- 物体ラベルに応じて最適な形状を選択
- 車両の向き（Yaw）の推定
- 推定結果の補正・フィルタリング
- 必要に応じて機械学習モデル（PointNet）による形状推定

物体を検出するモジュールではなく、**検出済み物体の形状をより正確にする**ことが目的です。:contentReference[oaicite:1]{index=1}

---

# 入力

主な入力は以下です。

| Topic | 型 | 内容 |
|-------|----|------|
| `input` | `DetectedObjectsWithFeature` | 点群クラスタ付き検出物体 |

入力には物体ラベル（CAR、TRUCK、PEDESTRIANなど）と、その物体に対応する点群クラスタが含まれています。:contentReference[oaicite:2]{index=2}

---

# 出力

| Topic | 型 | 内容 |
|-------|----|------|
| `output/objects` | `DetectedObjects` | 形状推定後の物体情報 |

出力される物体は、位置だけでなく形状・サイズ・向きが補正された状態になります。:contentReference[oaicite:3]{index=3}

---

# 推定方法

物体の種類によって推定方法が異なります。

| 物体 | 推定方法 |
|------|---------|
| 車両（CAR・TRUCK・BUSなど） | L-Shape Fitting または ML（PointNet） |
| 歩行者 | 円柱（Cylinder） |
| その他・UNKNOWN | 凸包（Convex Hull） |

車両では長方形の形状を推定し、歩行者は円柱、未知物体は点群全体を囲む凸包として推定されます。:contentReference[oaicite:4]{index=4}

---

# 処理の流れ

```text
PointCloud
      │
      ▼
Object Detection
      │
      ▼
DetectedObjectsWithFeature
      │
      ▼
autoware_shape_estimation
      │
      ▼
Shape Refinement
      │
      ▼
DetectedObjects
```

---

# 主な設定項目

代表的なパラメータは以下です。

| パラメータ | 内容 |
|------------|------|
| `use_corrector` | 推定結果を補正する |
| `use_filter` | 不適切な形状を除外する |
| `use_vehicle_reference_yaw` | 車両の向きを補正する |
| `use_vehicle_reference_shape_size` | 車両サイズを補正する |
| `use_boost_bbox_optimizer` | Bounding Box最適化を有効化 |
| `model_params.use_ml_shape_estimator` | PointNetによる形状推定を使用 |
| `model_params.minimum_points` | 推定に必要な最小点数 |

通常はルールベースの形状推定を利用し、必要に応じてMLベースの推定へ切り替えることができます。:contentReference[oaicite:5]{index=5}

---

# 他モジュールとの関係

このモジュールは物体検出の直後に配置されることが多く、形状を補正した後に追跡やフュージョンへ渡されます。

```text
LiDAR / Camera
        │
        ▼
Object Detection
        │
        ▼
autoware_shape_estimation
        │
        ▼
DetectedObjects
        │
        ├────────► Multi Object Tracker
        │
        ├────────► Radar Fusion
        │
        └────────► Prediction
```

より正確な形状を推定することで、後続のトラッキングや将来軌道予測の精度向上につながります。:contentReference[oaicite:6]{index=6}

---

# 特徴

- **ルールベース**と**機械学習（PointNet）**の両方に対応
- 車両・歩行者・未知物体で最適な推定アルゴリズムを自動選択
- 推定後にフィルタ・補正処理を実施
- 推定に失敗した場合は、安全なデフォルト形状へフォールバック

これにより、さまざまな環境でも安定した形状推定を実現します。:contentReference[oaicite:7]{index=7}

---

# まとめ

- 点群クラスタから3D物体形状を推定するモジュール
- 車両はBounding Box、歩行者はCylinder、未知物体はConvex Hullを推定
- L-Shape FittingやPointNetによる高精度な形状推定に対応
- 物体のサイズ・向き・形状を補正し、後続モジュールへ提供
- Perceptionパイプラインにおける形状推定・補正を担当する重要なモジュール