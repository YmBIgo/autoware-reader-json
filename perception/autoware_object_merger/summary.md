# autoware_object_merger

## 概要

`autoware_object_merger` は、**複数の物体検出結果（DetectedObjects）を1つの検出結果へ統合する**ためのパッケージです。

Autowareでは、LiDAR・カメラ・クラスタリングなど複数の検出アルゴリズムが同時に物体を検出することがあります。同じ物体が複数回検出される場合があるため、このパッケージでは**データ関連付け（Data Association）**を行い、重複した検出結果を統合します。:contentReference[oaicite:0]{index=0}

---

# 主な役割

`autoware_object_merger` の役割は次のとおりです。

- 2つの `DetectedObjects` を受け取る
- 同じ物体同士を対応付ける
- 重複した検出結果を統合する
- 信頼性の高い物体情報を優先する
- PlanningやTrackingで利用しやすい検出結果を出力する

---

# なぜ必要なのか

Autowareでは複数の認識アルゴリズムが並列で動作することがあります。

例えば、

```text
LiDAR Detector
        │
        ▼
DetectedObjects A

Camera Detector
        │
        ▼
DetectedObjects B
```

どちらも同じ車を検出している場合、

```text
A: 車①
B: 車①
```

という重複が発生します。

`autoware_object_merger` は、

```text
車①
```

という1つの物体へ統合します。:contentReference[oaicite:1]{index=1}

---

# 処理の流れ

```text
DetectedObjects①
         │
         │
DetectedObjects②
         │
         ▼
データ関連付け
         │
         ▼
重複判定
         │
         ▼
統合
         │
         ▼
Merged Objects
```

---

# 1. 入力

2つの物体検出結果を受け取ります。

それぞれの物体には

- 種類
- 位置
- 大きさ
- 向き
- 信頼度

などの情報が含まれています。:contentReference[oaicite:2]{index=2}

---

# 2. データ関連付け（Data Association）

まず、

「どの物体同士が同じ対象なのか」

を判定します。

例えば

```text
LiDAR
車A

Camera
車B
```

について、

- 距離
- 大きさ
- 面積

などを比較し、

```
車A = 車B
```

と判断します。:contentReference[oaicite:3]{index=3}

---

# 3. 重複判定

関連付けが成功した物体について、

実際に重なっているかを判定します。

判定には

- 距離
- 面積
- オーバーラップ率

などが利用されます。:contentReference[oaicite:4]{index=4}

---

# 4. 統合

同じ物体と判断された場合、

1つの `DetectedObject` に統合します。

パラメータ設定に応じて、

- 優先する検出器
- クラスごとの優先順位

などを切り替えることができます。:contentReference[oaicite:5]{index=5}

---

# 使用されるアルゴリズム

物体同士の対応付けには、

**Successive Shortest Path Algorithm**

が利用されています。

これは最小コストフロー問題を解くアルゴリズムで、

「どの物体同士を対応付けるのが最も自然か」

をコスト最小化によって決定します。

コストには

- 物体間距離
- 面積
- ゲート条件（距離やサイズの閾値）

などが使用されます。:contentReference[oaicite:6]{index=6}

---

# ROS2ノード

主要ノードは

```
object_merger
```

です。

複数の物体検出ノードの後段に配置されます。

---

# 入力

| Topic | 型 | 内容 |
|--------|----|------|
| `input/object0` | `autoware_perception_msgs/msg/DetectedObjects` | 検出結果① |
| `input/object1` | `autoware_perception_msgs/msg/DetectedObjects` | 検出結果② |

---

# 出力

| Topic | 型 | 内容 |
|--------|----|------|
| `output/object` | `autoware_perception_msgs/msg/DetectedObjects` | 統合後の検出結果 |

入力された2つの検出結果を1つにまとめた `DetectedObjects` が出力されます。:contentReference[oaicite:7]{index=7}

---

# 主なパラメータ

代表的な設定項目には以下があります。

| パラメータ | 内容 |
|------------|------|
| `priority_mode` | どちらの検出器を優先するか |
| `class_based_priority_matrix` | クラスごとの優先順位 |
| `precision_threshold_to_judge_overlapped` | 重複判定のPrecision閾値 |
| `recall_threshold_to_judge_overlapped` | 重複判定のRecall閾値 |
| `remove_overlapped_unknown_objects` | Unknown物体を除去するか |
| `base_link_frame_id` | 判定を行う座標系 |

これらを調整することで、検出器ごとの特性に合わせた統合が可能です。:contentReference[oaicite:8]{index=8}

---

# ディレクトリ構成

```text
autoware_object_merger/
├── config/          # パラメータ
├── include/         # ヘッダ
├── launch/          # Launchファイル
├── schema/          # パラメータスキーマ
├── src/             # 統合アルゴリズム
├── test/            # テスト
├── CMakeLists.txt
├── package.xml
└── README.md
```

---

# Autoware内での位置付け

Perception全体では次のような流れになります。

```text
LiDAR Detector
        │
Camera Detector
        │
Cluster Detector
        │
        ▼
Object Merger
        │
        ▼
DetectedObjects
        │
        ▼
Multi Object Tracker
        │
        ▼
Prediction
        │
        ▼
Planning
```

`autoware_object_merger` は、複数の検出器の結果を整理し、後続モジュールが扱いやすい一貫した物体情報を提供する役割を担います。

---

# このモジュールの特徴

- 複数の検出器の結果を統合
- データ関連付けによる重複除去
- クラスごとの優先順位を設定可能
- Unknown物体の扱いを調整可能
- Trackingへ渡す検出結果を整理できる
- Successive Shortest Path Algorithm による対応付けを採用:contentReference[oaicite:9]{index=9}

---

# まとめ

`autoware_object_merger` は、複数の物体検出器が出力した `DetectedObjects` を統合するためのパッケージです。

物体同士のデータ関連付けを行い、重複した検出を1つにまとめることで、TrackingやPlanningが利用しやすい認識結果を生成します。

Autowareでは、複数の認識アルゴリズムを組み合わせる際の重要な橋渡しとなるモジュールです。

---

# 参考

- Autoware Universe Documentation - object_merger :contentReference[oaicite:10]{index=10}
- GitHub: https://github.com/autowarefoundation/autoware_universe/tree/main/perception/autoware_object_merger