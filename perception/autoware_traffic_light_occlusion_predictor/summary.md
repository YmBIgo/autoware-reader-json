# autoware_traffic_light_occlusion_predictor 概要

## 概要

`autoware_traffic_light_occlusion_predictor` は、**LiDARの点群データを利用して、カメラで認識した信号機が他の物体によって隠れている（Occlusion）かどうかを判定するパッケージ**です。

交通信号認識では、信号機が街路樹や大型車両、標識などに隠れている場合、画像認識だけでは誤った認識を行う可能性があります。本パッケージは、LiDAR点群を利用して遮蔽率（Occlusion Ratio）を計算し、**信頼できない認識結果を無効化することで、交通信号認識の信頼性を向上**させます。 :contentReference[oaicite:0]{index=0}

---

## 主な役割

このモジュールは以下の役割を担当します。

- 信号機ROIを取得
- LiDAR点群を取得
- ROIがどの程度遮蔽されているかを計算
- 遮蔽率が高い信号機を判定
- 遮蔽された信号機の認識結果を `UNKNOWN` に変更

これにより、見えていない信号機を誤って「赤」や「青」と判定してしまうことを防ぎます。 :contentReference[oaicite:1]{index=1}

---

## 処理の流れ

```text
Traffic Light Classifier
（信号認識）
        │
        ▼
TrafficLightRoiArray
        │
        ▼
LiDAR PointCloud
        │
        ▼
autoware_traffic_light_occlusion_predictor
（遮蔽判定）
        │
        ▼
TrafficLightArray
（遮蔽時はUNKNOWN）
        │
        ▼
Traffic Light Arbiter
        │
        ▼
Planning
```

---

## どのように遮蔽を判定するのか

本パッケージでは、信号機ROI内の多数の画素を選択し、それらを3次元空間へ投影します。

その後、

- LiDAR点群との位置関係
- カメラから見た視点

を利用して、各画素が物体によって遮られているかを判定します。

最後に、

> **遮蔽された画素数 ÷ 全画素数**

を遮蔽率（Occlusion Ratio）として計算します。

遮蔽率が設定した閾値を超えた場合、その信号機は「十分に見えていない」と判断されます。 :contentReference[oaicite:2]{index=2}

---

## 入力

主な入力は以下です。

- ベクターマップ（Lanelet2）
- TrafficLightRoiArray
- 車両用信号認識結果
- 歩行者用信号認識結果
- CameraInfo
- LiDAR点群（PointCloud2）

これらの情報を利用して、信号機と周囲物体の位置関係を評価します。 :contentReference[oaicite:3]{index=3}

---

## 出力

主な出力は

- `TrafficLightArray`

です。

遮蔽されていない信号機はそのまま出力されます。

一方、遮蔽率が高いと判断された信号機については、

- 色：`UNKNOWN`
- 形状：`UNKNOWN`
- 信頼度：`0.0`

へ書き換えられて出力されます。 :contentReference[oaicite:4]{index=4}

---

## 他モジュールとの関係

交通信号認識では、以下のような流れになります。

```text
Traffic Light Map Based Detector
              │
              ▼
Traffic Light Fine Detector
              │
              ▼
Traffic Light Classifier
              │
              ▼
autoware_traffic_light_occlusion_predictor
              │
              ▼
Traffic Light Arbiter
              │
              ▼
Planning
```

このモジュールは、**画像認識結果をそのまま利用するのではなく、LiDARによる遮蔽判定を加えることで認識結果の信頼性を高める役割**を担います。 :contentReference[oaicite:5]{index=5}

---

## 利用される場面

例えば、

- 大型トラックが信号機を隠している
- 樹木や街路樹が信号機の一部を覆っている
- 標識や電柱によって信号機が見えにくい
- 信号機が建物の陰に入っている

といった状況では、画像だけでは誤認識する可能性があります。

本パッケージはLiDAR点群を利用して遮蔽を検出し、誤った信号認識が後続のPlanningへ伝わることを防ぎます。 :contentReference[oaicite:6]{index=6}

---

## まとめ

`autoware_traffic_light_occlusion_predictor` は、**LiDAR点群を利用して信号機の遮蔽状況を判定し、信頼できない認識結果を除外するモジュール**です。

主な特徴は次のとおりです。

- LiDAR点群による遮蔽判定
- 信号機ROIごとの遮蔽率を計算
- 遮蔽された信号機を `UNKNOWN` として出力
- 誤認識による危険な判断を防止
- 交通信号認識全体の安全性と信頼性を向上させる :contentReference[oaicite:7]{index=7}