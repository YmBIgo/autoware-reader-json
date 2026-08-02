# autoware_traffic_light_arbiter 概要

## 概要

`autoware_traffic_light_arbiter` は、**複数の信号機認識結果を統合し、最終的な信号状態を決定する Perception モジュール**です。

Autoware では、カメラによる画像認識だけでなく、V2X（Vehicle-to-Everything）などの外部システムから信号情報を取得できる場合があります。このパッケージは、それら複数の情報を比較・統合し、Planning モジュールへ渡す信頼性の高い信号情報を生成します。 :contentReference[oaicite:0]{index=0}

---

# 主な役割

このパッケージの役割は以下のとおりです。

- カメラ認識による信号情報を受け取る
- 外部システム（V2Xなど）の信号情報を受け取る
- 複数の信号情報を比較・統合する
- 最終的な信号状態を決定する
- Planning モジュールへ信号情報を提供する :contentReference[oaicite:1]{index=1}

---

# 処理の流れ

```text
カメラ認識
      │
      ▼
TrafficLightGroup
      │
      │
外部システム(V2X)
      │
      ▼
TrafficLightGroup
      │
      ▼
Traffic Light Arbiter
      │
      ├─ 信号対応確認
      ├─ 優先順位判定
      └─ 信頼度判定
      │
      ▼
最終的な信号情報
```

---

# 入力

主な入力は以下のとおりです。

| 入力 | 内容 |
|------|------|
| Vector Map | 信号IDの対応付けに利用 |
| Perception Traffic Signals | カメラ認識による信号情報 |
| External Traffic Signals | V2Xなど外部システムからの信号情報 |

ベクターマップを利用して、各信号機のIDを正しく対応付けます。 :contentReference[oaicite:2]{index=2}

---

# 出力

主な出力は

- `TrafficLightGroupArray`

です。

統合された信号情報は、Planning モジュールで停止・発進などの判断に利用されます。 :contentReference[oaicite:3]{index=3}

---

# 主な処理

## 信号の対応付け

まず、カメラ認識結果と外部システムの信号が同じ信号機を表しているか確認します。

この際、ベクターマップに登録されている信号機IDを利用します。 :contentReference[oaicite:4]{index=4}

---

## 信号の一致判定

オプションとして **Signal Match Validator** を利用できます。

この機能では、

- 両方とも赤
- 両方とも青
- 両方とも黄

など、両者が一致している場合のみその信号状態を採用します。

一致しない場合は、安全側の判断として **UNKNOWN（不明）** を出力することがあります。 :contentReference[oaicite:5]{index=5}

---

## 優先順位による統合

Signal Match Validator を使用しない場合は、信号情報の優先順位を設定できます。

主なモードは次の3種類です。

- **External Priority**
  - 外部システムを優先
- **Perception Priority**
  - カメラ認識を優先
- **Confidence Priority**
  - 信頼度の高い情報を採用

利用環境に応じて適切な方式を選択できます。 :contentReference[oaicite:6]{index=6}

---

# 特徴

- カメラ認識と外部信号を統合
- V2Xとの連携に対応
- 信号の一致判定機能を搭載
- 優先順位による統合が可能
- 信頼性の高い信号状態を生成 :contentReference[oaicite:7]{index=7}

---

# 利用される場面

`autoware_traffic_light_arbiter` は、

- カメラ認識のみの環境
- V2X対応車両
- インフラ協調型自動運転
- 信号認識の冗長化

などで利用されます。 :contentReference[oaicite:8]{index=8}

---

# 他モジュールとの関係

```text
Traffic Light Classifier
          │
          ▼
TrafficLightGroup
          │
          │
External Signal (V2X)
          │
          ▼
autoware_traffic_light_arbiter
          │
          ▼
Merged TrafficLightGroup
          │
          ▼
Behavior Planning
          │
          ▼
Vehicle Control
```

複数の信号情報を統合することで、後続の Planning モジュールはより信頼性の高い信号状態を利用できます。

---

# 他の信号認識モジュールとの違い

| モジュール | 役割 |
|------------|------|
| `autoware_traffic_light_classifier` | カメラ画像から信号の色を判定する |
| `autoware_crosswalk_traffic_light_estimator` | 横断歩道用信号を推定する |
| `autoware_traffic_light_arbiter` | 複数の信号情報を統合して最終結果を決定する |

つまり、`autoware_traffic_light_arbiter` は**信号を認識するモジュールではなく、認識結果を統合・調停（Arbiter）するモジュール**です。 :contentReference[oaicite:9]{index=9}

---

# まとめ

`autoware_traffic_light_arbiter` は、**カメラ認識や外部システム（V2X）から得られた信号情報を統合し、最終的な信号状態を決定する Perception モジュール**です。

主な役割は、**複数の信号情報を比較・統合し、Planning モジュールへ信頼性の高い信号状態を提供すること**です。これにより、カメラ認識だけに依存しない、より安全で堅牢な信号認識システムを実現できます。 :contentReference[oaicite:10]{index=10}