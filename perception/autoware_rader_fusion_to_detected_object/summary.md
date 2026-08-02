# autoware_radar_fusion_to_detected_object

## 概要

`autoware_radar_fusion_to_detected_object` は、**LiDARなどによる3次元物体検出結果とレーダーの検出結果を融合（Sensor Fusion）するパッケージ**です。

LiDAR は物体の位置や大きさを高精度に推定できますが、速度の推定は得意ではありません。一方、レーダーはドップラー効果を利用して物体の速度を高精度に測定できますが、位置や形状の精度は LiDAR より劣ります。

このパッケージでは、それぞれの長所を組み合わせることで、**位置・大きさ・速度をより高精度に推定した物体情報**を生成します。 :contentReference[oaicite:0]{index=0}

---

## 主な役割

このパッケージでは、以下のような処理を行います。

- LiDAR検出物体とレーダー検出物体を対応付ける
- レーダーから取得した速度情報を付与
- 信頼度の低い検出結果を補強
- Tracking や Planning が利用しやすい物体情報を生成する :contentReference[oaicite:1]{index=1}

---

## なぜセンサ融合が必要か

各センサには得意・不得意があります。

|センサ|得意なこと|苦手なこと|
|-------|----------|-----------|
|LiDAR|位置・形状・サイズの推定|速度推定|
|Radar|速度推定（ドップラー速度）|位置・形状の推定|

そこで両者を組み合わせることで、

- 正確な位置
- 正確なサイズ
- 正確な速度

を同時に取得できるようになります。 :contentReference[oaicite:2]{index=2}

---

## 処理の流れ

大まかな処理は以下のようになります。

```text
LiDAR Detected Objects
            │
            │
Radar Detected Objects
            │
            ▼
    物体同士を対応付け
(Data Association)
            │
            ▼
速度情報を付与
            │
            ▼
信頼度を補正
            │
            ▼
Fusion済みDetected Objects
```

対応付けでは、LiDAR のバウンディングボックスとレーダーで検出された物体の位置関係を利用して、同じ物体かどうかを判定します。 :contentReference[oaicite:3]{index=3}

---

## 入力

### 3D Detected Objects

LiDARなどで検出された物体

型

- `autoware_perception_msgs/msg/DetectedObjects`

---

### Radar Objects

レーダーによる検出物体

型

- `autoware_perception_msgs/msg/DetectedObjects`

※ 両方とも同じ座標系（frame_id）である必要があります。 :contentReference[oaicite:4]{index=4}

---

## 出力

### Fusion済みDetected Objects

型

- `autoware_perception_msgs/msg/DetectedObjects`

出力される物体には、

- 位置
- 大きさ
- 向き
- クラス
- **速度（Twist）**

などの情報が含まれます。

また、信頼度が低く出力対象にならなかった物体は、デバッグ用トピックとして出力できます。 :contentReference[oaicite:5]{index=5}

---

## 主なパラメータ

代表的な設定項目です。

|パラメータ|内容|
|-----------|----|
|`bounding_box_margin`|対応付け時にバウンディングボックスへ追加する余白|
|`threshold_probability`|出力する物体の最低信頼度|
|`compensate_probability`|レーダー情報を利用して信頼度を補正するか|
|`update_rate_hz`|ノードの更新周期|

これらのパラメータを調整することで、センサ融合の精度や挙動を変更できます。 :contentReference[oaicite:6]{index=6}

---

## 利用される場面

このノードは以下のような用途で利用されます。

- 移動物体の速度推定
- Adaptive Cruise Control（ACC）
- Multi Object Tracker の入力
- Prediction モジュールの入力
- Planning モジュールへの物体情報提供

特に、速度情報が重要となる追跡や経路計画で大きな役割を果たします。 :contentReference[oaicite:7]{index=7}

---

## 関連パッケージ

このパッケージの出力は、以下のようなモジュールで利用されます。

- `autoware_multi_object_tracker`
- `autoware_predicted_path_postprocessor`
- `autoware_object_merger`
- Planning モジュール

センサ融合によって得られた高品質な物体情報は、後続の認識・予測・経路計画全体の精度向上に貢献します。

---

## まとめ

`autoware_radar_fusion_to_detected_object` は、LiDAR による3次元物体検出とレーダー検出結果を融合し、**位置・大きさ・速度を組み合わせた高精度な物体情報**を生成するパッケージです。

LiDAR の高い位置推定能力と、レーダーの高精度な速度推定能力を組み合わせることで、Tracking や Prediction、Planning で利用しやすい認識結果を提供し、自動運転システム全体の認識性能向上に貢献します。 :contentReference[oaicite:8]{index=8}