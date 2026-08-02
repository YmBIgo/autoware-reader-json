# autoware_traffic_light_map_based_detector 概要

## 概要

`autoware_traffic_light_map_based_detector` は、**HDマップと車両・カメラの位置姿勢情報を利用して、カメラ画像内のどこに信号機が映るかを推定し、ROI（Region of Interest）を生成するパッケージ**です。

画像認識によって信号機を探すのではなく、**HDマップに登録された信号機の位置をもとに、画像上の予測位置を計算する**ことが特徴です。これにより、後続の画像認識モジュールは対象領域だけを効率よく解析できます。 :contentReference[oaicite:0]{index=0}

---

## 主な役割

このモジュールは以下の役割を担当します。

- HDマップから信号機の位置を取得
- 車両位置・姿勢（TF）を取得
- カメラパラメータを利用して画像上へ投影
- 信号機が映ると予想されるROIを生成
- 後続の Fine Detector へROIを渡す

画像認識を行う前段階として、**「どこを見れば信号機があるか」を計算する役割**を担います。 :contentReference[oaicite:1]{index=1}

---

## 処理の流れ

```text
HD Map
    │
    ▼
車両位置・姿勢（TF）
    │
    ▼
Camera Info
    │
    ▼
autoware_traffic_light_map_based_detector
（ROIの予測）
    │
    ▼
TrafficLightRoiArray
    │
    ▼
autoware_traffic_light_fine_detector
```

---

## どのようにROIを求めるのか

このパッケージでは、

1. HDマップ上の信号機位置
2. 車両の現在位置
3. カメラの内部・外部パラメータ

を利用して、**3次元座標を画像座標へ投影**します。

さらに、カメラの取り付け誤差や車両の振動による位置ずれを考慮し、ROIに余裕を持たせることができます。これにより、後続の検出器が信号機を見失いにくくなります。 :contentReference[oaicite:2]{index=2}

---

## 入力

主な入力は以下です。

- ベクターマップ（Lanelet2）
- カメラパラメータ（CameraInfo）
- 車両のTF（座標変換）
- （任意）走行ルート情報

走行ルートが与えられている場合は、**そのルート上の信号機のみ**を対象とします。

ルート情報がない場合は、一定距離・一定角度以内にある信号機を対象としてROIを生成します。 :contentReference[oaicite:3]{index=3}

---

## 出力

主な出力は

- `TrafficLightRoiArray`

です。

このROIは

- 信号機のおおよその位置
- 後続の Fine Detector が探索する領域

として利用されます。

また、デバッグ用に予測位置を示すマーカーも出力できます。 :contentReference[oaicite:4]{index=4}

---

## 他モジュールとの関係

交通信号認識では、以下のような流れになります。

```text
HD Map
    │
    ▼
autoware_traffic_light_map_based_detector
    │
    ▼
autoware_traffic_light_fine_detector
    │
    ▼
autoware_traffic_light_classifier
    │
    ▼
autoware_traffic_light_arbiter
    │
    ▼
Planning
```

このモジュールは、**交通信号認識パイプラインの最初のステップ**として、画像内の信号機候補領域を生成します。

---

## 利用される場面

例えば、

- 数十メートル先の信号機
- 複数の信号機が並ぶ交差点
- 遠距離で小さく映る信号機

などでも、HDマップを利用することで画像中のおおよその位置を事前に把握できます。

その結果、後続の画像認識モジュールは必要な領域だけを解析できるため、処理効率と認識精度の向上が期待できます。 :contentReference[oaicite:5]{index=5}

---

## まとめ

`autoware_traffic_light_map_based_detector` は、**HDマップを利用してカメラ画像内の信号機位置を予測し、ROIを生成するモジュール**です。

主な特徴は次のとおりです。

- HDマップから信号機位置を取得
- カメラ画像上へ信号機位置を投影
- 信号機ROIを生成
- カメラの誤差や車両振動を考慮可能
- Traffic Light Fine Detector の前段として動作し、交通信号認識の効率と精度を向上させる :contentReference[oaicite:6]{index=6}