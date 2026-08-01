# autoware_lidar_frnet

## 概要

`autoware_lidar_frnet` は、LiDARから取得した点群データに対して、点ごとのクラス分類を行うための3Dセマンティックセグメンテーションパッケージです。

入力された各点の位置情報や反射強度をニューラルネットワークで処理し、それぞれの点が道路、車両、歩行者、植生などのどのクラスに属するかを推定します。

Autowareの認識処理において、LiDAR点群の意味を理解し、後段の物体認識や点群フィルタリングで利用しやすい形式に変換する役割を持ちます。

---

## FRNetとは

このパッケージは、LiDARセマンティックセグメンテーション手法である **FRNet（Frustum-Range Network）** をベースにしています。

FRNetでは、LiDAR点群をレンジ画像に近い形式で扱いながら、点群が持つ3次元的な情報も利用してクラス分類を行います。

`autoware_lidar_frnet` では、学習済みモデルの推論に **TensorRT** を使用し、GPU上で高速に処理できるように実装されています。

---

## 主な役割

`autoware_lidar_frnet` の主な役割は、次のとおりです。

- LiDAR点群をニューラルネットワークへ入力できる形式に変換する
- TensorRTを使ってFRNetモデルの推論を実行する
- 各点についてクラスIDと推定確率を出力する
- 推論結果を色付き点群として可視化する
- 指定されたクラスの点群を抽出または除外する
- 前処理、推論、後処理にかかった時間を出力する

---

## 処理の流れ

処理の流れを簡単に表すと、次のようになります。

```text
LiDAR点群
    ↓
点群フォーマットの確認
    ↓
前処理・レンジ画像への変換
    ↓
FRNetによる推論
    ↓
各点へのクラスID・確率の付与
    ↓
セグメンテーション点群・可視化点群・フィルタ済み点群の出力
```

### 1. 点群の受信

ROS 2の `sensor_msgs/msg/PointCloud2` 形式でLiDAR点群を受信します。

点群には、主に以下の情報が含まれます。

- X座標
- Y座標
- Z座標
- 反射強度

センサーによっては、リング番号、チャンネル番号、距離、タイムスタンプなどの追加情報も利用できます。

### 2. 前処理

受信した点群をFRNetモデルへ入力できる形式に変換します。

FRNetはLiDARの走査構造を利用するため、点群をセンサー原点を基準としたレンジ画像形式へ投影して処理します。

### 3. TensorRTによる推論

学習済みのFRNetモデルをTensorRTで実行します。

各点について、どの意味クラスに属する可能性が高いかを推定します。

### 4. 後処理

推論結果を元の点群へ対応付け、各点に以下の情報を追加します。

- クラスID
- 推定確率

また、クラスごとに色を付けた可視化用点群や、条件に応じて点を抽出したフィルタ済み点群も生成します。

---

## ROS 2ノード

このパッケージの主要なノードは、`lidar_frnet_node` です。

主な実装ファイルは次のとおりです。

```text
src/lidar_frnet_node.cpp
```

起動には、次のLaunchファイルが用意されています。

```text
launch/lidar_frnet.launch.xml
```

---

## 入力

| トピック | メッセージ型 | 内容 |
|---|---|---|
| `~/input/pointcloud` | `sensor_msgs/msg/PointCloud2` | セマンティックセグメンテーション対象のLiDAR点群 |

---

## 出力

| トピック | メッセージ型 | 内容 |
|---|---|---|
| `~/output/pointcloud/segmentation` | `sensor_msgs/msg/PointCloud2` | 各点にクラスIDと推定確率を付加した点群 |
| `~/output/pointcloud/visualization` | `sensor_msgs/msg/PointCloud2` | クラスごとにRGB色を付けた可視化用点群 |
| `~/output/pointcloud/filtered` | `sensor_msgs/msg/PointCloud2` | 指定条件で抽出・除外された点群 |
| `debug/cyclic_time_ms` | `autoware_internal_debug_msgs/msg/Float64Stamped` | ノードの処理周期 |
| `debug/pipeline_latency_ms` | `autoware_internal_debug_msgs/msg/Float64Stamped` | パイプライン全体の遅延 |
| `debug/processing_time/preprocess_ms` | `autoware_internal_debug_msgs/msg/Float64Stamped` | 前処理時間 |
| `debug/processing_time/inference_ms` | `autoware_internal_debug_msgs/msg/Float64Stamped` | 推論時間 |
| `debug/processing_time/postprocess_ms` | `autoware_internal_debug_msgs/msg/Float64Stamped` | 後処理時間 |
| `debug/processing_time/total_ms` | `autoware_internal_debug_msgs/msg/Float64Stamped` | 合計処理時間 |
| `/diagnostics` | `diagnostic_msgs/msg/DiagnosticArray` | 処理時間などに関する診断情報 |

---

## 対応する点群形式

このパッケージは、複数の `PointCloud2` フィールド構成に対応しています。

主な対応形式は次のとおりです。

- `XYZI`
- `XYZIRC`
- `XYZIRADRT`
- `XYZIRCAEDT`

最初に受信した点群のフィールド構成から、入力フォーマットを自動的に判定します。

---

## TensorRTエンジンの生成

`build_only` オプションを使用すると、ONNXモデルからTensorRTエンジンを生成できます。

```bash
ros2 launch autoware_lidar_frnet lidar_frnet.launch.xml build_only:=true
```

通常の実行時には、生成されたTensorRTエンジンを使って推論を行います。

---

## 学習済みモデル

公式の学習済みモデルは、T4 Datasetを使用して学習されています。

対応する主なLiDARセンサーは次のとおりです。

- Hesai OT128
- Hesai QT128

FRNetはレンジ画像を利用するため、LiDARの視野角や水平・垂直解像度が学習時と異なる場合、認識性能が低下する可能性があります。

また、入力点群はLiDARセンサーの原点を基準とした座標である必要があります。

---

## ディレクトリ構成

主なディレクトリと役割は次のとおりです。

```text
autoware_lidar_frnet/
├── config/                         # ノードやモデルの設定
├── include/autoware/lidar_frnet/  # ノード・推論処理のヘッダ
├── launch/                         # ROS 2 Launchファイル
├── lib/                            # CUDA・TensorRT関連の実装
├── schema/                         # パラメータスキーマ
├── src/                            # ROS 2ノードの実装
├── CMakeLists.txt
├── package.xml
└── README.md
```

---

## Autoware内での位置づけ

`autoware_lidar_frnet` は、LiDAR点群から物体の意味情報を抽出する認識モジュールです。

一般的な物体検出が車両や歩行者を矩形として検出するのに対し、セマンティックセグメンテーションでは点群の各点を直接分類します。

そのため、次のような処理で利用できます。

- 地面や道路領域の識別
- 車両や歩行者に属する点の識別
- 植生や構造物などの除外
- 後段の物体検出やクラスタリングの補助
- 認識結果の可視化

---

## まとめ

`autoware_lidar_frnet` は、FRNetとTensorRTを利用して、LiDAR点群の各点を意味クラスへ分類するパッケージです。

入力点群に対して前処理、ニューラルネットワーク推論、後処理を行い、クラス情報付き点群、可視化用点群、フィルタ済み点群を出力します。

Autowareにおいては、LiDAR点群を単なる座標の集合として扱うのではなく、それぞれの点が何を表しているかを推定するためのモジュールとして位置づけられます。

---

## 参考

- Autoware Universe: `autoware_lidar_frnet`
  - https://github.com/autowarefoundation/autoware_universe/tree/main/perception/autoware_lidar_frnet
- FRNet: Frustum-Range Networks for Scalable LiDAR Segmentation
  - https://xiangxu-0103.github.io/FRNet
- AWML
  - https://github.com/tier4/AWML