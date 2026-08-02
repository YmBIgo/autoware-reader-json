# autoware_object_sorter

## 概要

`autoware_object_sorter` は、検出された物体 (`DetectedObjects`) や追跡された物体 (`TrackedObjects`) を**条件に応じてフィルタリングする**ためのパッケージです。

主に、**距離**や**速度**、**物体クラス**などを基準として不要な物体を除外し、後続の Tracking や Prediction、Planning が扱いやすい認識結果を出力します。:contentReference[oaicite:0]{index=0}

---

# 主な役割

`autoware_object_sorter` の役割は次のとおりです。

- 認識結果を受け取る
- 物体までの距離を計算する
- 速度が一定以下の物体を除外する
- クラスごとに異なる条件でフィルタリングする
- 条件を満たした物体のみを出力する

---

# なぜ必要なのか

物体検出器は、多数の物体を検出します。

しかし実際には、

- 非常に遠い物体
- ほとんど動いていない物体
- 利用したくないクラス

などは後続処理で不要な場合があります。

例えば、

```text
検出結果

車A（10m）
車B（250m）
歩行者（5m）
Unknown（150m）
```

という検出結果があった場合、

設定によっては

```text
車A
歩行者
```

だけを残し、

遠方の物体や不要なクラスを除外できます。:contentReference[oaicite:1]{index=1}

---

# 処理の流れ

```text
DetectedObjects
      または
TrackedObjects
        │
        ▼
距離計算
        │
        ▼
速度判定
        │
        ▼
クラス別フィルタ
        │
        ▼
Filtered Objects
```

---

# 1. 認識結果の入力

入力には

- `DetectedObjects`
- `TrackedObjects`

のどちらも利用できます。

各物体には

- 位置
- 速度
- 形状
- クラス

などの情報が含まれています。:contentReference[oaicite:2]{index=2}

---

# 2. 距離によるフィルタ

物体までの距離を計算し、

設定した範囲内にある物体だけを残します。

例えば、

```text
自車
 │
 ├── 20m ○
 ├── 50m ○
 ├──150m ×
```

のように、

一定距離より遠い物体を除外できます。:contentReference[oaicite:3]{index=3}

---

# 3. 速度によるフィルタ

各物体の速度を利用して、

設定した閾値より遅い物体を除外できます。

例えば、

```text
歩行者 0.2 m/s
```

で、

```
min_velocity = 0.5
```

の場合は出力されません。:contentReference[oaicite:4]{index=4}

---

# 4. クラスごとの設定

このパッケージでは、

物体クラスごとに

- publishするか
- 最低速度
- 距離条件

などを個別に設定できます。

対応する代表的なクラスは

- CAR
- TRUCK
- BUS
- TRAILER
- MOTORCYCLE
- BICYCLE
- PEDESTRIAN
- ANIMAL
- HAZARD
- UNKNOWN

などです。:contentReference[oaicite:5]{index=5}

---

# ROS2ノード

主要ノードは

```
object_sorter
```

です。

Perceptionパイプラインの途中で、

不要な物体を除外する目的で利用されます。

---

# 入力

| Topic | 型 | 内容 |
|--------|----|------|
| `~/input/objects` | `autoware_perception_msgs/msg/DetectedObjects` または `TrackedObjects` | 認識・追跡結果 |

---

# 出力

| Topic | 型 | 内容 |
|--------|----|------|
| `~/output/objects` | `autoware_perception_msgs/msg/DetectedObjects` または `TrackedObjects` | フィルタ後の物体 |

入力と同じ型で出力されるため、後続ノードへそのまま接続できます。:contentReference[oaicite:6]{index=6}

---

# 主なパラメータ

代表的なパラメータは次のとおりです。

| パラメータ | 内容 |
|------------|------|
| `range_calc_frame_id` | 距離計算に使用する座標系 |
| `range_calc_offset` | 距離計算時のオフセット |
| `publish` | クラスを出力するか |
| `min_velocity_threshold` | 最低速度 |
| `min_distance` | 最小距離 |
| `max_distance` | 最大距離 |
| `min_x / max_x` | X方向の範囲 |
| `min_y / max_y` | Y方向の範囲 |

これらはクラスごとに個別に設定できます。:contentReference[oaicite:7]{index=7}

---

# ディレクトリ構成

```text
autoware_object_sorter/
├── config/         # パラメータ
├── include/        # ヘッダ
├── launch/         # Launchファイル
├── schema/         # パラメータスキーマ
├── src/            # フィルタ処理
├── CMakeLists.txt
├── package.xml
└── README.md
```

---

# Autoware内での位置付け

Perception全体では次のような位置になります。

```text
Object Detection
        │
        ▼
Object Sorter
        │
        ▼
Object Merger
        │
        ▼
Multi Object Tracker
        │
        ▼
Prediction
```

物体検出直後や物体統合の前後で利用され、不要な認識結果を減らすことで後続処理の負荷を軽減します。

---

# このモジュールの特徴

- 距離によるフィルタリング
- 速度によるフィルタリング
- クラスごとの個別設定
- DetectedObjects・TrackedObjects の両方に対応
- シンプルな設定で認識結果を整理できる:contentReference[oaicite:8]{index=8}

---

# まとめ

`autoware_object_sorter` は、認識・追跡された物体を距離や速度、物体クラスなどの条件でフィルタリングするパッケージです。

不要な物体を除外することで、Tracking・Prediction・Planning が扱うデータを整理し、認識パイプライン全体の効率向上に貢献します。:contentReference[oaicite:9]{index=9}

---

# 参考

- Autoware Universe Documentation - Object Sorter
  https://autowarefoundation.github.io/autoware_universe/latest/perception/autoware_object_sorter/
- GitHub
  https://github.com/autowarefoundation/autoware_universe/tree/main/perception/autoware_object_sorter