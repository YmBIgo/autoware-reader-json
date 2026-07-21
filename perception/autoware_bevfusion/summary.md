# autoware_bevfusion

## 概要

`autoware_bevfusion` は、Autoware Universe の **3次元物体検出（3D Object
Detection）** パッケージです。

LiDAR点群とカメラ画像を **Bird's-Eye View（BEV：鳥瞰図）**
空間で統合し、自動車・歩行者・自転車などの物体を検出します。

内部では NVIDIA TensorRT
を利用してニューラルネットワークを高速に実行し、検出結果を Autoware の
Perception パイプラインへ渡します。

------------------------------------------------------------------------

## 主な役割

このパッケージは、センサデータから周囲の物体を認識する役割を担います。

``` text
LiDAR
   \
    +--> autoware_bevfusion --> DetectedObjects
   /
Camera
```

出力された物体情報は、その後の

-   Tracking
-   Prediction
-   Planning

などで利用されます。

------------------------------------------------------------------------

## BEVFusionとは

BEVFusion は、LiDARとカメラを **BEV（Bird's-Eye View）**
に変換してから融合する認識手法です。

-   LiDAR：距離や形状を正確に取得できる
-   Camera：色や模様などの情報を取得できる
-   BEVFusion：両者の特徴を組み合わせて認識精度を向上させる

------------------------------------------------------------------------

## 主な入力

-   LiDAR点群 (`PointCloud2`)
-   カメラ画像
-   CameraInfo
-   TF（LiDARとカメラの位置関係）

------------------------------------------------------------------------

## 主な出力

-   `DetectedObjects`

検出結果には次のような情報が含まれます。

-   物体種類
-   位置
-   向き
-   大きさ
-   信頼度

------------------------------------------------------------------------

## 処理の流れ

概略は次のようになります。

``` text
LiDAR / Camera
        │
        ▼
前処理
        │
        ▼
BEVFusion (TensorRT)
        │
        ▼
後処理
        │
        ▼
DetectedObjects
```

------------------------------------------------------------------------

## パッケージ構成

``` text
src/
  ROS2ノード

lib/
  推論・前処理・後処理

include/
  ヘッダファイル

config/
  パラメータ

launch/
  起動ファイル
```

------------------------------------------------------------------------

## 特徴

-   LiDAR単独でも利用可能
-   LiDAR + Camera の融合にも対応
-   TensorRT による高速推論
-   CUDA を利用したGPU処理
-   Autoware の標準 Perception パイプラインに統合

------------------------------------------------------------------------

## 他モジュールとの関係

``` text
Sensors
    │
    ▼
PointCloud Preprocessor
    │
    ▼
autoware_bevfusion
    │
    ▼
Object Tracker
    │
    ▼
Prediction
    │
    ▼
Planning
```

------------------------------------------------------------------------

## まとめ

`autoware_bevfusion` は、Autoware における3次元物体検出モジュールです。

LiDARやカメラから取得したセンサ情報を統合し、周囲の物体を認識して
`DetectedObjects` として出力します。GPU（CUDA）と TensorRT
を活用することで、高速かつ高精度な推論を実現しており、自動運転システムの
Perception の中核となるパッケージの一つです。