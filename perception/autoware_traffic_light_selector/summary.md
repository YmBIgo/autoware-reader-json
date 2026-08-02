# autoware_traffic_light_selector 概要

## 概要

`autoware_traffic_light_selector` は、**交通信号検出器が検出した複数の信号機候補の中から、HDマップ上で期待される信号機に対応するものを選択し、交通信号IDを付与するパッケージ**です。

物体検出モデルは画像内のすべての信号機を検出できますが、そのままでは「どの信号機が自車の走行に関係するのか」は分かりません。本パッケージは、マップベースで予測されたROI（関心領域）を利用して、**必要な信号機だけを選び出す役割**を担います。 :contentReference[oaicite:0]{index=0}

---

## 主な役割

このモジュールは以下の役割を担当します。

- 信号機検出器が検出した信号機候補を取得
- HDマップから予測されたROIを取得
- 各検出結果と予測ROIを対応付け
- 対応する交通信号IDを付与
- 後続モジュールへ対象信号機のみを出力

これにより、後続の信号分類モジュールや統合モジュールは、自車に関係する信号機だけを処理できます。 :contentReference[oaicite:1]{index=1}

---

## 処理の流れ

```text
Traffic Light Detector
（信号機候補）
        │
        │
        ├─────────────┐
        │             │
        ▼             ▼
Expected ROI     Rough ROI
（HDマップ）      （マップ投影）
        │             │
        └──────┬──────┘
               ▼
autoware_traffic_light_selector
（対応付け・ID付与）
               │
               ▼
TrafficLightRoiArray
               │
               ▼
Traffic Light Classifier
```

---

## どのように選択するのか

本パッケージでは、

- 検出器が出力した正確な信号機領域
- HDマップから計算された期待ROI（Expected ROI）
- マップ投影による大まかなROI（Rough ROI）

を比較し、**最も一致する信号機候補を選択**します。

また、選択した信号機にはHDマップに対応する**Traffic Light ID**が付与され、後続モジュールで一貫して利用できるようになります。 :contentReference[oaicite:2]{index=2}

---

## 入力

主な入力は以下です。

- 検出器が出力した信号機検出結果（Detected ROIs）
- Rough ROI（マップ投影によるROI）
- Expected ROI（理想的なROI）

これらの情報を組み合わせて、対象となる信号機を決定します。 :contentReference[oaicite:3]{index=3}

---

## 出力

主な出力は

- `TrafficLightRoiArray`

です。

出力されるROIには、

- 信号機ROI
- 対応する交通信号ID

が含まれます。

この情報は、後続の `autoware_traffic_light_classifier` が信号色や形状を認識する際に利用されます。 :contentReference[oaicite:4]{index=4}

---

## 他モジュールとの関係

交通信号認識では、以下のような流れになります。

```text
Traffic Light Map Based Detector
              │
              ▼
Traffic Light Detector
（YOLOなど）
              │
              ▼
autoware_traffic_light_selector
              │
              ▼
Traffic Light Classifier
              │
              ▼
Traffic Light Arbiter
              │
              ▼
Planning
```

このモジュールは、**検出器が見つけた複数の信号機候補から、自車に関係する信号機を選択する役割**を担います。 :contentReference[oaicite:5]{index=5}

---

## 利用される場面

例えば、

- 交差点に複数の信号機が並んでいる
- 対向車線用の信号機も画像に映っている
- 歩行者用信号と車両用信号が同時に検出されている

といった状況では、物体検出だけでは自車が参照すべき信号機を特定できません。

本パッケージは、HDマップから得られる位置情報を利用して、**自車に関係する信号機だけを選択**します。 :contentReference[oaicite:6]{index=6}

---

## まとめ

`autoware_traffic_light_selector` は、**交通信号検出結果とHDマップ情報を対応付け、自車が参照すべき信号機を選択するモジュール**です。

主な特徴は次のとおりです。

- 検出された複数の信号機候補を選別
- Expected ROI・Rough ROIを利用して対応付け
- 信号機IDを付与
- 後続のTraffic Light Classifierへ適切なROIを提供
- 複数の信号機が存在する交差点でも正しい対象信号機を扱えるようにする重要なモジュール :contentReference[oaicite:7]{index=7}