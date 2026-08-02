````markdown
# autoware_traffic_light_multi_camera_fusion 概要

## 概要

`autoware_traffic_light_multi_camera_fusion` は、**複数のカメラから得られた交通信号認識結果を統合し、より信頼性の高い信号認識結果を生成するパッケージ**です。

自動運転車には前方・側方など複数のカメラが搭載されることがありますが、各カメラは異なる角度や条件で信号機を撮影します。本パッケージは、それらの認識結果を融合することで、**一部のカメラで信号が見えない場合や誤認識が発生した場合でも、安定した信号認識を実現**します。 :contentReference[oaicite:0]{index=0}

---

## 主な役割

このモジュールは以下の役割を担当します。

- 複数カメラの信号認識結果を受け取る
- 各カメラの認識結果を比較
- 最も信頼できる認識結果を選択
- 同じ信号グループの情報を統合
- 1つの信頼性の高い信号認識結果として出力

これにより、単一カメラでは難しい状況でも、より安定した認識が可能になります。 :contentReference[oaicite:1]{index=1}

---

## 処理の流れ

```text
Camera 1
      │
Camera 2
      │
Camera 3
      │
      ▼
Traffic Light Classifier
（各カメラ）
      │
      ▼
autoware_traffic_light_multi_camera_fusion
      │
      ▼
統合されたTrafficLightGroupArray
      │
      ▼
Traffic Light Arbiter
      │
      ▼
Planning
```

---

## どのように融合するのか

本パッケージでは、主に2段階で認識結果を統合します。

### 1. Best View Selection

まず、各信号機について複数カメラの結果を比較し、最も信頼できる認識結果を選択します。

選択時には以下のような情報が考慮されます。

- 最新の認識結果
- Unknownではない認識結果
- 信号機全体が画像内に写っているか
- 認識信頼度（Confidence）

これにより、最も良い視点（Best View）が選ばれます。 :contentReference[oaicite:2]{index=2}

---

### 2. Group Fusion

次に、同じ交差点・同じ信号グループに属する信号機の情報を統合します。

単純な多数決ではなく、**Bayesian Fusion（ベイズ更新）**を利用して各認識結果の信頼度を考慮しながら最終的な信号状態を決定します。

これにより、より信頼性の高い認識結果を得ることができます。 :contentReference[oaicite:3]{index=3}

---

## 入力

各カメラについて、主に以下の情報を入力します。

- CameraInfo
- TrafficLightRoiArray（Fine DetectorのROI）
- TrafficLightArray（Classifierの認識結果）

カメラごとの名前空間を設定するだけで、必要なトピックを自動的に購読します。 :contentReference[oaicite:4]{index=4}

---

## 出力

主な出力は

- `TrafficLightGroupArray`

です。

この出力には、

- 信号色
- 信号形状
- 複数カメラを統合した最終的な認識結果

が含まれます。 :contentReference[oaicite:5]{index=5}

---

## 他モジュールとの関係

交通信号認識では以下のような構成になります。

```text
Traffic Light Map Based Detector
              │
              ▼
Traffic Light Fine Detector
              │
              ▼
Traffic Light Classifier
（各カメラ）
              │
              ▼
autoware_traffic_light_multi_camera_fusion
              │
              ▼
Traffic Light Arbiter
              │
              ▼
Planning
```

このモジュールは、**各カメラの認識結果を統合し、後続モジュールへより信頼性の高い信号情報を提供する役割**を担います。 :contentReference[oaicite:6]{index=6}

---

## 利用される場面

例えば、

- 前方カメラでは信号が街路樹に隠れている
- 左右のカメラでは信号がはっきり見えている
- 一部のカメラで逆光により誤認識した
- 複数の信号灯器がある複雑な交差点

といった状況でも、複数カメラの情報を統合することで、より安定した信号認識を行えます。 :contentReference[oaicite:7]{index=7}

---

## まとめ

`autoware_traffic_light_multi_camera_fusion` は、**複数カメラから得られた交通信号認識結果を統合し、信頼性の高い最終認識結果を生成するモジュール**です。

主な特徴は次のとおりです。

- 複数カメラの認識結果を統合
- 最も信頼できる視点（Best View）を選択
- ベイズ更新による高信頼なグループ融合
- 部分的な遮蔽や誤認識に強い
- 交通信号認識全体の安定性と精度を向上させる :contentReference[oaicite:8]{index=8}
````
