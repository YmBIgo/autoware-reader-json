# autoware_traffic_light_classifier 概要

## 概要

`autoware_traffic_light_classifier` は、**カメラ画像から切り出された信号機領域（ROI）を解析し、信号機の色や形状を分類するパッケージ**です。

Autowareでは、信号機を検出するモジュールと、その検出結果を認識するモジュールが分かれており、本パッケージは**「この信号は赤・黄・青のどれか」「車両用か歩行者用か」**といった判定を担当します。

---

## 主な役割

このモジュールの役割は以下のとおりです。

- 信号機ROI（切り出し画像）の入力
- 信号灯火の色を判定
- 信号の形状（丸・矢印など）を判定
- 認識結果を `TrafficSignalArray` として出力

認識結果は、その後の経路計画（Planning）や交差点での停止・発進判断などに利用されます。 :contentReference[oaicite:0]{index=0}

---

## 処理の流れ

```text
カメラ画像
      │
      ▼
Traffic Light Detector
（信号機ROI検出）
      │
      ▼
autoware_traffic_light_classifier
（色・形状の分類）
      │
      ▼
TrafficSignalArray
      │
      ▼
Planning / Behavior
```

---

## 主な認識方式

本パッケージでは、用途に応じて2種類の分類方式を利用できます。

### 1. CNN Classifier（推奨）

深層学習モデルを用いて信号機を分類します。

特徴

- EfficientNet や MobileNet を利用
- 赤・黄・青だけでなく形状も判定可能
- 高い認識精度
- TensorRTによる高速推論にも対応

通常のAutowareではこちらが利用されます。 :contentReference[oaicite:1]{index=1}

---

### 2. HSV Classifier

画像のHSV色空間を利用して色を判定する方式です。

特徴

- 軽量
- GPU不要
- 色のみを比較的簡単に判定

深層学習を使用しない簡易的な分類器として利用できます。 :contentReference[oaicite:2]{index=2}

---

## 入力

主な入力は以下です。

- カメラ画像
- 信号機ROI（TrafficLightRoiArray）

ROIは通常、

- map_based_detector
- traffic_light_fine_detector

などの検出ノードから送られてきます。 :contentReference[oaicite:3]{index=3}

---

## 出力

主な出力は

- `TrafficSignalArray`

です。

各信号について

- 色（Red / Yellow / Green）
- 形状（Circle / Arrow など）
- 信頼度

が格納されます。 :contentReference[oaicite:4]{index=4}

---

## 他モジュールとの関係

交通信号認識パイプラインでは、以下のような位置付けになります。

```text
Map Based Detector
        │
        ▼
Traffic Light Fine Detector
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

本パッケージは、**検出された信号機を最終的な認識結果へ変換する認識モジュール**として機能します。

---

## 利用される場面

例えば交差点では

- 赤信号
- 青信号
- 黄信号
- 右折矢印
- 左折矢印
- 歩行者信号

などを認識し、自動運転システムへ通知します。

---

## まとめ

`autoware_traffic_light_classifier` は、**信号機ROIから信号の色・形状を分類する画像認識モジュール**です。

主な特徴は次のとおりです。

- 信号機画像を分類する
- CNN方式とHSV方式の2種類をサポート
- 色・形状・信頼度を出力
- TrafficSignalArrayを生成して後続モジュールへ渡す
- 自動運転における信号認識の中核となるパッケージ
```