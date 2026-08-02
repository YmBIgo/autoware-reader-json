# autoware_tensorrt_plugins 概要

## 概要

`autoware_tensorrt_plugins` は、**TensorRT が標準ではサポートしていない演算を追加するためのプラグインライブラリ**です。

Autoware の Perception モジュールでは、BEVFusion や BEVFormer などの深層学習モデルを TensorRT 上で高速に実行しています。しかし、これらのモデルには TensorRT が標準では実装していない演算が含まれています。

このパッケージは、それらの演算を **TensorRT Plugin** として実装し、GPU 上で高速に実行できるようにする役割を担っています。 :contentReference[oaicite:0]{index=0}

---

# 主な役割

このパッケージの役割は以下のとおりです。

- TensorRT に独自演算を追加する
- CUDA を利用して高速に処理する
- 深層学習モデルを TensorRT で実行可能にする
- Autoware の各認識モジュールから利用される共通ライブラリを提供する

このパッケージ自体は認識を行うノードではなく、**他の TensorRT ベースのモジュールを支える基盤ライブラリ**です。 :contentReference[oaicite:1]{index=1}

---

# 主なプラグイン

代表的なプラグインには次のようなものがあります。

| プラグイン | 概要 |
|------------|------|
| Sparse Convolution | スパース畳み込みを実行 |
| Argsort | TensorRT 標準では扱いにくい並び替え処理 |
| BEV Pool | BEVFusion などで利用される BEV Pool 演算 |
| Scatter | インデックスに従ってデータを配置する演算 |
| Unique | 重複要素を除去する演算 |
| Multi-Scale Deformable Attention | Deformable Attention の実装 |
| Rotate | 画像や特徴量の回転処理 |
| Select and Pad | データ選択とパディング処理 | :contentReference[oaicite:2]{index=2}

---

# 処理の流れ

```text
ONNXモデル
      │
      ▼
TensorRT Engine生成
      │
      ▼
標準演算
      │
      ├─────────────┐
      │             │
      ▼             ▼
TensorRT Plugin   CUDA Kernel
      │             │
      └──────┬──────┘
             ▼
      推論結果
```

TensorRT が対応していない演算だけを、このプラグインが補完します。 :contentReference[oaicite:3]{index=3}

---

# 利用されるモジュール

このパッケージは、TensorRT を利用する Perception モジュールから利用されます。

例えば

- `autoware_tensorrt_bevformer`
- `autoware_bevfusion`
- その他 TensorRT ベースの認識モジュール

などで共通ライブラリとして使用されています。 :contentReference[oaicite:4]{index=4}

---

# 特徴

- TensorRT に独自演算を追加
- CUDA により高速実装
- FP16 に対応した演算が多数
- GPU 向けに最適化
- 複数の認識モデルで共通利用可能 :contentReference[oaicite:5]{index=5}

---

# このパッケージが必要な理由

TensorRT は多くの ONNX 演算をサポートしていますが、最新の深層学習モデルでは未対応の演算も少なくありません。

例えば

- Deformable Attention
- BEV Pool
- Scatter
- Unique

などは TensorRT 標準だけでは実行できない、または制約があります。

そのため `autoware_tensorrt_plugins` がこれらの演算を実装し、TensorRT エンジンから利用できるようにしています。 :contentReference[oaicite:6]{index=6}

---

# ノードではない

このパッケージには ROS2 ノードは含まれていません。

代わりに

- TensorRT Plugin
- CUDA カーネル
- Plugin Creator
- Plugin Registry

などが実装されており、TensorRT エンジン生成時に自動的に読み込まれます。 :contentReference[oaicite:7]{index=7}

---

# 他モジュールとの関係

```text
Camera / LiDAR
        │
        ▼
BEVFormer / BEVFusion
        │
        ▼
autoware_tensorrt_plugins
        │
        ▼
TensorRT
        │
        ▼
CUDA
        │
        ▼
推論結果
```

TensorRT ベースの認識モジュールの内部で利用される、共通の演算ライブラリとして機能します。

---

# まとめ

`autoware_tensorrt_plugins` は、**TensorRT に独自演算を追加するための共通ライブラリ**です。

主な役割は、TensorRT が標準では対応していない演算を CUDA で実装し、BEVFormer や BEVFusion などの深層学習モデルを高速に実行できるようにすることです。

Autoware の TensorRT ベースの Perception モジュールを支える重要な基盤コンポーネントとなっています。 :contentReference[oaicite:8]{index=8}