# autoware_radar_object_tracker

## 概要

`autoware_radar_object_tracker` は、**レーダーで検出した物体を時系列で追跡（Tracking）し、一貫したIDと速度情報を付与するパッケージ**です。

レーダーは天候や照明条件の影響を受けにくく、物体の速度を高精度に計測できるという特徴があります。しかし、単一フレームの検出だけでは、同じ物体を継続して認識することはできません。

このパッケージでは、連続するレーダー検出結果を関連付け（Data Association）し、**各物体に一意のIDを割り当てて継続的に追跡**します。また、物体の移動モデルを利用して、位置や速度を安定して推定します。 :contentReference[oaicite:0]{index=0}

---

## 主な役割

このパッケージでは、以下の処理を行います。

- レーダーで検出した物体を継続的に追跡
- 同じ物体に一貫したIDを付与
- 位置・速度を時系列で推定
- ノイズとなる検出結果を除去
- 後続の Prediction や Planning が利用しやすい追跡結果を生成 :contentReference[oaicite:1]{index=1}

---

## 処理の流れ

大まかな処理の流れは以下のようになります。

```text
Radar Detected Objects
          │
          ▼
   Data Association
（既存トラックとの対応付け）
          │
          ▼
 Motion Model
（位置・速度の更新）
          │
          ▼
 ノイズフィルタ
          │
          ▼
Tracked Objects
```

新しく検出された物体は既存の追跡対象（トラック）と対応付けられ、対応するトラックが見つからない場合は新しいトラックが作成されます。対応付けが成功したトラックは、新しい観測結果を利用して状態が更新されます。 :contentReference[oaicite:2]{index=2}

---

## Data Association

Tracking において最も重要な処理が **Data Association（データ対応付け）** です。

例えば、

```text
前フレーム

ID=1  車A
ID=2  車B

↓

現在フレーム

車A
車B
```

現在検出された物体が前フレームのどの物体と同じかを判定し、同じIDを維持します。

この処理によって、

- IDの切り替わり
- 同じ物体の重複追跡

などを防ぐことができます。 :contentReference[oaicite:3]{index=3}

---

## Tracker Model

このパッケージでは、物体の種類に応じて異なる運動モデル（Tracker Model）を利用できます。

代表的なモデルは以下の2種類です。

- **Linear Motion Tracker**
  - 直線運動を仮定したシンプルなモデル
- **Constant Turn Rate Motion Tracker**
  - 一定の旋回速度を仮定したモデル

物体の種類や用途に応じて適切なモデルを選択できるため、より現実的な追跡が可能になります。 :contentReference[oaicite:4]{index=4}

---

## 入力

### Detected Objects

レーダーによる検出物体

型

- `autoware_perception_msgs/msg/DetectedObjects`

---

### Lanelet Map

HDマップ（Lanelet2）

型

- `autoware_map_msgs/msg/LaneletMapBin`

マップ情報は、道路外に存在するノイズの除去などに利用できます。 :contentReference[oaicite:5]{index=5}

---

## 出力

### Tracked Objects

追跡済み物体

型

- `autoware_perception_msgs/msg/TrackedObjects`

各物体には以下のような情報が含まれます。

- Track ID
- 位置
- 速度
- 向き
- クラス
- 共分散（推定誤差）

---

## 主なパラメータ

代表的な設定項目です。

|パラメータ|内容|
|-----------|----|
|`publish_rate`|出力周期|
|`tracker_lifetime`|トラックを保持する時間|
|`enable_delay_compensation`|遅延補償を有効化するか|
|`use_distance_based_noise_filtering`|距離に基づくノイズ除去を有効化するか|
|`use_map_based_noise_filtering`|HDマップを利用したノイズ除去を有効化するか|
|`max_distance_from_lane`|車線から離れた物体を除去する距離の閾値|

これらを調整することで、追跡の安定性やノイズ除去の挙動を変更できます。 :contentReference[oaicite:6]{index=6}

---

## 利用される場面

このノードは以下のような用途で利用されます。

- レーダーによる物体追跡
- 悪天候時の物体認識
- Adaptive Cruise Control（ACC）
- Prediction モジュールへの入力
- Planning モジュールへの入力

レーダーは雨・霧・夜間でも性能が低下しにくいため、LiDAR やカメラを補完する重要なセンサとして活用されます。

---

## 関連パッケージ

このパッケージは、以下のようなモジュールと組み合わせて利用されます。

- `autoware_radar_fusion_to_detected_object`
- `autoware_multi_object_tracker`
- `autoware_predicted_path_postprocessor`
- Planning モジュール

センサ融合や物体予測と組み合わせることで、より高精度な環境認識を実現します。

---

## まとめ

`autoware_radar_object_tracker` は、レーダーで検出した物体を時系列で追跡し、一貫した Track ID と位置・速度を推定するパッケージです。

Data Association と運動モデルを組み合わせることで、連続フレーム間で同一物体を安定して追跡し、後続の Prediction や Planning に利用できる高品質な追跡結果を提供します。また、距離やHDマップを利用したノイズ除去機能も備えており、悪天候を含むさまざまな環境で信頼性の高い物体追跡を実現します。 :contentReference[oaicite:7]{index=7}