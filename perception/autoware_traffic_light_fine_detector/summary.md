# autoware_traffic_light_fine_detector 概要

## 概要

`autoware_traffic_light_fine_detector` は、**カメラ画像内の信号機を高精度に検出し、信号機の位置（ROI: Region of Interest）を細かく補正するためのパッケージ**です。

Autowareでは、HDマップを利用して信号機のおおよその位置を取得する「Map Based Detector」と、このパッケージによる画像認識を組み合わせることで、より正確な信号機検出を実現しています。

本パッケージでは **YOLOX-s** をベースとした深層学習モデルを利用して、信号機の位置を高精度に検出します。 :contentReference[oaicite:0]{index=0}

---

## 主な役割

このモジュールは以下の役割を担当します。

- カメラ画像を入力
- Map Based Detectorが生成した大まかなROIを取得
- CNN（YOLOX-s）を用いて信号機を再検出
- より正確なROIへ補正
- 後続の Traffic Light Classifier へ渡す

単に信号機の有無を判定するだけでなく、**分類しやすい正確な位置へROIを補正すること**が目的です。 :contentReference[oaicite:1]{index=1}

---

## 処理の流れ

```text
HD Map
   │
   ▼
Map Based Detector
（大まかなROI）
   │
   ▼
Camera Image
   │
   ▼
autoware_traffic_light_fine_detector
（YOLOXによるROI補正）
   │
   ▼
Traffic Light Classifier
（色・形状の認識）
```

---

## 使用される技術

本パッケージでは、YOLOX-s をベースとした物体検出モデルを使用しています。

特徴

- CNNによる高精度な信号機検出
- ONNXモデルを利用
- TensorRTによる高速推論に対応
- 日本の信号機画像約17,000枚でファインチューニングされた学習済みモデルを利用可能 :contentReference[oaicite:2]{index=2}

---

## 入力

主な入力は以下です。

- カメラ画像
- Map Based Detector が生成した信号機ROI
- ROI選択用の期待ROI（Expected ROI）

これらを利用して、画像内で最も適切な信号機領域を検出します。 :contentReference[oaicite:3]{index=3}

---

## 出力

主な出力は

- 補正された `TrafficLightRoiArray`

です。

後続の `autoware_traffic_light_classifier` が、このROIを利用して

- 赤
- 黄
- 青
- 矢印信号
- 歩行者信号

などを認識します。

---

## 他モジュールとの関係

交通信号認識では以下のような構成になります。

```text
HD Map
   │
   ▼
Traffic Light Map Based Detector
   │
   ▼
autoware_traffic_light_fine_detector
   │
   ▼
autoware_traffic_light_classifier
   │
   ▼
Traffic Light Arbiter
   │
   ▼
Planning
```

このモジュールは、**マップ情報による粗い検出結果を、画像認識によってより正確な検出結果へ補正する役割**を担います。 :contentReference[oaicite:4]{index=4}

---

## 利用される場面

例えば交差点では、

- 遠方にある小さな信号機
- カメラ画像内でわずかに位置がずれた信号機
- Map Based Detectorだけでは位置誤差がある信号機

などに対して、画像認識を用いて正確な位置を求めます。

その結果、後続の分類器がより高い精度で信号色や矢印方向を認識できるようになります。

---

## まとめ

`autoware_traffic_light_fine_detector` は、**YOLOXを利用して信号機のROIを高精度に補正する画像認識モジュール**です。

主な特徴は次のとおりです。

- 信号機の位置を高精度に再検出
- YOLOX-sベースの深層学習モデルを使用
- ONNX・TensorRTによる高速推論に対応
- Map Based DetectorとTraffic Light Classifierの橋渡しを担当
- 交通信号認識の精度向上に重要な役割を果たす :contentReference[oaicite:5]{index=5}