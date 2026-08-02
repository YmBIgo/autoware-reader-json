# autoware_traffic_light_category_merger 概要

## 概要

`autoware_traffic_light_category_merger` は、**車両用信号機と歩行者用信号機の認識結果を1つに統合する Perception モジュール**です。

Autoware では、車両用信号と歩行者用信号を別々の分類器で認識する構成があります。このパッケージは、それぞれの分類結果をまとめて、後続のモジュールが扱いやすい1つの信号情報として出力します。なお、分類結果が存在しない信号については **Unknown** として補完されます。 :contentReference[oaicite:0]{index=0}

---

# 主な役割

このパッケージの役割は以下のとおりです。

- 車両用信号の認識結果を受け取る
- 歩行者用信号の認識結果を受け取る
- 両方の結果を1つの信号配列へ統合する
- 未分類の信号を Unknown として補完する
- 後続の信号認識・Planning モジュールへ渡す :contentReference[oaicite:1]{index=1}

---

# 処理の流れ

```text
車両用信号分類器
        │
        ▼
TrafficLightArray
        │
        │
歩行者用信号分類器
        │
        ▼
TrafficLightArray
        │
        ▼
Traffic Light Category Merger
        │
        ├─ 統合
        ├─ Unknown補完
        │
        ▼
TrafficLightArray
```

---

# 入力

主な入力は2種類の信号分類結果です。

| 入力 | 内容 |
|------|------|
| `input/car_signals` | 車両用信号の分類結果 |
| `input/pedestrian_signals` | 歩行者用信号の分類結果 |

どちらも `TrafficLightArray` メッセージとして入力されます。 :contentReference[oaicite:2]{index=2}

---

# 出力

主な出力は

- `output/traffic_signals`

です。

車両用信号と歩行者用信号が統合された `TrafficLightArray` が出力されます。 :contentReference[oaicite:3]{index=3}

---

# 主な処理

## 信号カテゴリの統合

車両用信号と歩行者用信号を1つの配列へまとめます。

これにより、後続のモジュールは複数の分類結果を個別に扱う必要がなくなります。 :contentReference[oaicite:4]{index=4}

---

## Unknown の補完

ROI（信号領域）は検出されていても分類結果が存在しない場合、その信号は **Unknown** として出力されます。

これにより、すべての信号に対して状態が設定され、一貫したデータを提供できます。 :contentReference[oaicite:5]{index=5}

---

# 特徴

- 車両用・歩行者用信号を統合
- シンプルな統合処理
- 未分類の信号を Unknown として補完
- 後続モジュールが扱いやすい形式で出力
- ROS 2 ノードとして動作 :contentReference[oaicite:6]{index=6}

---

# 利用される場面

`autoware_traffic_light_category_merger` は、

- 車両用信号と歩行者用信号を別々に認識する構成
- 複数の信号分類器を利用するシステム
- 信号情報を統一した形式で扱いたい場合

などで利用されます。 :contentReference[oaicite:7]{index=7}

---

# 他モジュールとの関係

```text
Traffic Light Detector
          │
          ▼
Traffic Light Classifier
     ├─────────────┐
     │             │
車両用分類器   歩行者用分類器
     │             │
     └──────┬──────┘
            ▼
autoware_traffic_light_category_merger
            │
            ▼
TrafficLightArray
            │
            ▼
Traffic Light Arbiter
            │
            ▼
Behavior Planning
```

このモジュールは、分類結果をまとめる役割を担い、その後の `autoware_traffic_light_arbiter` などで最終的な信号情報の決定に利用されます。 :contentReference[oaicite:8]{index=8}

---

# 他の信号認識モジュールとの違い

| モジュール | 役割 |
|------------|------|
| `autoware_traffic_light_classifier` | 信号画像から色や形状を分類する |
| `autoware_traffic_light_category_merger` | 車両用・歩行者用の分類結果を統合する |
| `autoware_traffic_light_arbiter` | 認識結果や外部（V2X）情報を統合して最終的な信号状態を決定する |

つまり、`autoware_traffic_light_category_merger` は**信号を認識するモジュールではなく、複数の分類結果を整理・統合するモジュール**です。 :contentReference[oaicite:9]{index=9}

---

# パラメータ

このパッケージには**特別なノードパラメータは用意されていません**。

入力された信号分類結果をそのまま統合するシンプルな構成となっています。 :contentReference[oaicite:10]{index=10}

---

# まとめ

`autoware_traffic_light_category_merger` は、**車両用信号と歩行者用信号の分類結果を統合する Perception モジュール**です。

主な役割は、**複数の信号分類結果を1つの `TrafficLightArray` にまとめ、未分類の信号を Unknown として補完すること**です。これにより、後続の信号認識や Planning モジュールは、統一された形式の信号情報を利用できるようになります。 :contentReference[oaicite:11]{index=11}