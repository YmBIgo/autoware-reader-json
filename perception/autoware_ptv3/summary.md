# autoware_ptv3

## 概要

`autoware_ptv3` は、**Point Transformer V3（PTv3）を用いて LiDAR 点群から3次元物体検出を行うパッケージ**です。

Point Transformer V3 は、Transformer ベースの3次元点群認識モデルであり、点群を直接入力として扱うことで、高精度な物体検出を実現します。本パッケージでは、TensorRT を利用して学習済みモデルを高速に推論し、車両・歩行者・自転車などの物体を検出します。:contentReference[oaicite:0]{index=0}

---

## 主な役割

このパッケージでは、以下の処理を担当します。

- LiDAR点群を入力として受け取る
- Point Transformer V3 による物体検出
- 検出した物体の位置・サイズ・向きを推定
- Autoware の標準メッセージ形式で出力

---

## Point Transformer V3とは

Point Transformer V3 は、Transformer を利用した3次元点群認識モデルです。

従来の CNN ベースや Voxel ベースの手法とは異なり、

- 点群を直接扱える
- 離れた点同士の関係を学習できる
- 複雑な形状の物体も認識しやすい

といった特徴があります。

これにより、都市環境のような複雑なシーンでも高い認識性能を発揮します。:contentReference[oaicite:1]{index=1}

---

## 処理の流れ

大まかな処理は以下のようになります。

```text
LiDAR PointCloud
        │
        ▼
前処理
（座標変換・特徴生成）
        │
        ▼
Point Transformer V3
        │
        ▼
物体検出
        │
        ▼
後処理
（NMSなど）
        │
        ▼
Detected Objects
```

推論後には、重複した検出結果の除去（NMS）などの後処理を行い、最終的な検出結果を生成します。:contentReference[oaicite:2]{index=2}

---

## 入力

### PointCloud

LiDARから取得した点群

型

- `sensor_msgs/msg/PointCloud2`

---

## 出力

### Detected Objects

検出した物体情報

型

- `autoware_perception_msgs/msg/DetectedObjects`

各物体には、以下のような情報が含まれます。

- クラス（車両・歩行者・自転車など）
- 位置
- 向き
- 大きさ
- 信頼度（Confidence）

---

## 主な特徴

このパッケージには次のような特徴があります。

- Point Transformer V3 を利用した最新の点群認識
- TensorRT による高速推論
- LiDAR点群のみで3次元物体を検出
- Autoware の Perception パイプラインへ容易に組み込み可能

---

## 利用される場面

このノードは以下のような用途で利用されます。

- 車両検出
- 歩行者検出
- 自転車検出
- 障害物認識
- Planning や Tracking への入力生成

検出結果は後続の **Multi Object Tracker** や **Prediction** モジュールへ渡され、自動運転システム全体で利用されます。

---

## 関連パッケージ

PTv3 の検出結果は、以下のようなモジュールで利用されます。

- `autoware_multi_object_tracker`
- `autoware_predicted_path_postprocessor`
- `autoware_object_merger`
- Planning モジュール

これらと組み合わせることで、検出から追跡・経路予測・経路計画まで一連の Perception パイプラインを構成できます。:contentReference[oaicite:3]{index=3}

---

## まとめ

`autoware_ptv3` は、Point Transformer V3 を用いて LiDAR 点群から3次元物体を検出するパッケージです。

Transformer ベースの点群認識モデルを TensorRT により高速に実行し、車両・歩行者・自転車などの物体を高精度に検出します。検出結果は Tracking や Prediction、Planning などの後続モジュールで利用され、自動運転における Perception の重要な役割を担っています。:contentReference[oaicite:4]{index=4}