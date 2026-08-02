# autoware_traffic_light_visualization 概要

## 概要

`autoware_traffic_light_visualization` は、**交通信号認識の結果をRVizや画像上に可視化するためのデバッグ・確認用パッケージ**です。

交通信号認識そのものを行うモジュールではなく、認識結果やROI（Region of Interest）を分かりやすく表示することで、開発や動作確認、チューニングを支援します。

このパッケージには、以下の2つの可視化ノードが含まれています。 :contentReference[oaicite:0]{index=0}

- `traffic_light_map_visualizer`
- `traffic_light_roi_visualizer`

---

## 主な役割

このモジュールは以下の役割を担当します。

- 信号機認識結果をRViz上に表示
- 信号機の位置や色をマーカーで表示
- カメラ画像上にROIを描画
- 信号色・形状・信頼度を画像へ重ねて表示
- 交通信号認識のデバッグを支援

認識アルゴリズムには影響せず、**開発者向けの可視化ツール**として利用されます。 :contentReference[oaicite:1]{index=1}

---

## 構成

本パッケージは2つのノードで構成されています。

### 1. traffic_light_map_visualizer

認識された交通信号を**RViz上にマーカーとして表示**します。

表示される情報の例

- 信号機の位置
- 信号の色（赤・黄・青など）
- HDマップ上の信号機との対応

これにより、認識結果を3次元空間で確認できます。 :contentReference[oaicite:2]{index=2}

---

### 2. traffic_light_roi_visualizer

カメラ画像上へ認識結果を重ねて表示します。

表示される情報の例

- 信号機ROI
- Rough ROI（Map Based Detector）
- Fine ROI
- 信号色
- 信号形状
- 認識信頼度（Confidence）

これにより、「どの領域を認識したか」「どのように分類されたか」を画像上で直感的に確認できます。 :contentReference[oaicite:3]{index=3}

---

## 処理の流れ

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
Traffic Light Visualization
      ├──────────────┐
      ▼              ▼
 RViz Marker      Debug Image
```

---

## 入力

### traffic_light_map_visualizer

主な入力は以下です。

- `TrafficLightGroupArray`
- ベクターマップ（Lanelet2）

これらを利用して、信号機の状態をRVizへ表示します。 :contentReference[oaicite:4]{index=4}

---

### traffic_light_roi_visualizer

主な入力は以下です。

- カメラ画像
- `TrafficLightArray`
- `TrafficLightRoiArray`
- （任意）Map Based Detectorが出力したRough ROI

認識結果とROIを画像へ重ねて出力します。 :contentReference[oaicite:5]{index=5}

---

## 出力

### traffic_light_map_visualizer

出力

- `MarkerArray`

RViz上で

- 信号位置
- 信号色

などを表示できます。 :contentReference[oaicite:6]{index=6}

---

### traffic_light_roi_visualizer

出力

- ROIが描画された画像

画像には

- ROI
- 信号色
- 信号形状
- 認識信頼度

などが表示されます。 :contentReference[oaicite:7]{index=7}

---

## 他モジュールとの関係

交通信号認識では、以下のような位置付けになります。

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
autoware_traffic_light_visualization
              │
      （デバッグ表示）
```

このモジュールは、**交通信号認識パイプラインの結果を可視化するための補助モジュール**です。

---

## 利用される場面

例えば、

- 信号機が正しく検出されているか確認したい
- ROIの位置が適切か確認したい
- 認識結果の色や信頼度を確認したい
- RViz上で信号状態を確認したい
- 学習モデルやパラメータのチューニングを行いたい

といった、開発・検証・デバッグの場面で活用されます。 :contentReference[oaicite:8]{index=8}

---

## まとめ

`autoware_traffic_light_visualization` は、**交通信号認識結果をRVizや画像上に可視化するデバッグ支援モジュール**です。

主な特徴は次のとおりです。

- 信号認識結果をRVizへ表示
- カメラ画像上へROIや認識結果を描画
- 信号色・形状・信頼度を確認可能
- 開発・デバッグ・チューニングを支援
- 交通信号認識アルゴリズムの理解と検証に役立つ