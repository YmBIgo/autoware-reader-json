# autoware_detected_object_feature_remover

## 概要

`autoware_detected_object_feature_remover` は、Autoware Universe の **Perception** モジュールに含まれる **検出物体から Feature 情報を取り除く** パッケージです。

役割は非常にシンプルで、`DetectedObjectWithFeatureArray` を **`DetectedObjects` に変換**します。物体の位置やサイズ、分類情報はそのまま保持し、追加の Feature データのみを削除します。 :contentReference[oaicite:0]{index=0}

---

# 役割

Perceptionでは、物体検出の結果として

- 物体情報
- 点群や画像などの Feature 情報

をまとめて持つメッセージが利用されることがあります。

しかし、後続のモジュールでは **Feature 情報が不要** な場合もあります。

`autoware_detected_object_feature_remover` は、そのような場合に不要な Feature を取り除き、よりシンプルなメッセージへ変換します。 :contentReference[oaicite:1]{index=1}

---

# Perceptionパイプラインでの位置付け

おおよそ次のような流れになります。

```text
物体検出
      │
      ▼
DetectedObjectWithFeatureArray
      │
      ▼
autoware_detected_object_feature_remover
      │
      ▼
DetectedObjects
      │
      ▼
Tracking
Prediction
Planning
```

このパッケージは、物体検出や追跡を行うものではなく、**メッセージ形式を変換するためのユーティリティノード**です。

---

# 主な特徴

- Feature情報のみを削除
- 物体情報はそのまま保持
- メッセージ形式を変換
- 高速・軽量
- ROS 2ノードとして利用可能

処理内容は単純な変換のみのため、計算負荷はほとんどありません。 :contentReference[oaicite:2]{index=2}

---

# 入力

入力は

```text
tier4_perception_msgs/msg/DetectedObjectWithFeatureArray
```

です。

このメッセージには、

- 物体位置
- 姿勢
- サイズ
- クラス情報
- Feature情報（点群・画像特徴など）

が含まれています。 :contentReference[oaicite:3]{index=3}

---

# 出力

出力は

```text
autoware_perception_msgs/msg/DetectedObjects
```

です。

Feature情報を除いた、

- 物体位置
- 向き
- サイズ
- クラス情報

などの基本情報のみが出力されます。 :contentReference[oaicite:4]{index=4}

---

# 処理の流れ

内部処理は非常にシンプルです。

```text
DetectedObjectWithFeatureArray
            │
            ▼
Feature情報を削除
            │
            ▼
DetectedObjects
```

物体の認識結果自体は変更せず、Featureフィールドだけを取り除きます。 :contentReference[oaicite:5]{index=5}

---

# パラメータ

このノードには**設定パラメータはありません**。

入力を受け取ると、そのまま `DetectedObjects` へ変換して出力します。 :contentReference[oaicite:6]{index=6}

---

# 利用シーン

`autoware_detected_object_feature_remover` は、次のような場面で利用されます。

- Feature情報が不要な後続ノードへ接続する
- メッセージサイズを小さくしたい
- 異なるPerceptionモジュール間でメッセージ形式を統一したい

特に、TrackingやPlanningなど、物体の位置や分類情報だけが必要なモジュールとの接続に適しています。

---

# 他のPerceptionモジュールとの違い

| パッケージ | 主な役割 |
|------------|----------|
| **autoware_detected_object_feature_remover** | Feature情報を削除し、メッセージ形式を変換する |
| **autoware_detected_object_validation** | 検出物体の妥当性を確認する |
| **autoware_object_merger** | 複数の物体検出結果を統合する |
| **autoware_bytetrack** | 検出物体を時間方向に追跡する |

このように、`autoware_detected_object_feature_remover` は認識アルゴリズムではなく、**データ形式を整理するための補助パッケージ**という位置付けです。

---

# まとめ

- `autoware_detected_object_feature_remover` は **Feature情報を削除してメッセージ形式を変換する** パッケージ
- `DetectedObjectWithFeatureArray` を `DetectedObjects` へ変換する
- 物体の位置・姿勢・サイズ・クラス情報は保持される
- Feature情報だけを除去するため、軽量かつ高速に処理できる
- TrackingやPlanningなど、Feature情報を必要としないモジュールへの橋渡しとして利用される。 :contentReference[oaicite:7]{index=7}