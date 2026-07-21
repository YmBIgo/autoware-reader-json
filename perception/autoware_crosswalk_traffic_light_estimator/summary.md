# autoware_crosswalk_traffic_light_estimator

## 概要

`autoware_crosswalk_traffic_light_estimator` は、Autoware Universe の **Perception** モジュールに含まれる **横断歩道用信号機（歩行者信号）の状態を推定する** パッケージです。

カメラによる信号認識結果とHDマップ（Lanelet2）を組み合わせ、**歩行者信号が検出できなかった場合でも、その状態を推定**します。また、歩行者信号の**点滅状態**も推定し、より安定した信号情報を後続モジュールへ提供します。 :contentReference[oaicite:0]{index=0}

---

# 役割

自動運転では、歩行者用信号機が

- 遠くにある
- 車両などに隠れている
- 太陽光や逆光で認識できない

といった状況が発生します。

そのような場合でも、車両用信号機とHDマップの情報を利用して、

- 横断歩道は赤なのか
- 青なのか
- 点滅しているのか

を推定し、安全な走行判断に利用できるようにします。 :contentReference[oaicite:1]{index=1}

---

# Perceptionパイプラインでの位置付け

おおよそ次のような流れになります。

```text
カメラ画像
      │
      ▼
交通信号認識
      │
      ▼
Vehicle Traffic Lights
      │
      ▼
autoware_crosswalk_traffic_light_estimator
      │
      ├── HDマップ
      │
      ▼
推定済み歩行者信号
      │
      ▼
Planning
```

このモジュールは、新しく信号を検出するのではなく、**既存の認識結果を補完・補正する役割**を担っています。

---

# 主な特徴

- 車両用信号から歩行者信号を推定
- HDマップを利用した信号関係の判定
- 歩行者信号の点滅状態を推定
- 認識できなかった信号を補完
- ROS 2ノードとして利用可能

これにより、信号認識が一時的に失敗しても、安定した歩行者信号情報を提供できます。 :contentReference[oaicite:2]{index=2}

---

# 入力

主な入力は以下です。

- ベクターマップ（Lanelet2）
  - `LaneletMapBin`
- 車両・歩行者信号の認識結果
  - `TrafficLightGroupArray`
- （任意）走行ルート
  - `LaneletRoute`

HDマップには、車両信号と横断歩道の対応関係が記録されています。 :contentReference[oaicite:3]{index=3}

---

# 出力

出力は

```text
TrafficLightGroupArray
```

です。

出力には、

- 認識された信号
- 推定された歩行者信号
- 必要に応じて補正された点滅状態

が含まれます。 :contentReference[oaicite:4]{index=4}

---

# 処理の流れ

内部では、おおよそ次のような処理を行います。

```text
交通信号認識
      │
      ▼
歩行者信号が認識できる？
      │
      ├── Yes
      │      │
      │      ▼
      │ 点滅状態を補正
      │
      └── No
             │
             ▼
車両信号とHDマップから推定
             │
             ▼
歩行者信号を出力
```

認識結果が利用できる場合はそれを優先し、利用できない場合はHDマップと車両信号の情報から推定を行います。 :contentReference[oaicite:5]{index=5}

---

# 推定の考え方

例えば、

```text
車両信号：青
        │
        ▼
交差する横断歩道
        │
        ▼
歩行者信号：赤（推定）
```

のように、車両が進行可能であれば、交差する横断歩道では歩行者は停止している可能性が高いと判断します。

逆に、

```text
車両信号：赤
        │
        ▼
歩行者信号：青
```

となるケースも、HDマップに登録された信号の対応関係を利用して推定できます。なお、特殊な交差点ではHDマップに上書きルールを定義することも可能です。 :contentReference[oaicite:6]{index=6}

---

# 主なパラメータ

代表的なパラメータには次のようなものがあります。

| パラメータ | 説明 |
|------------|------|
| `use_last_detect_color` | 一時的に認識できなくなった場合に直前の色を保持する |
| `use_pedestrian_signal_detect` | 認識結果を優先するか、推定結果で上書きするかを設定 |
| `last_detect_color_hold_time` | 最後に認識した色を保持する時間 |
| `last_colors_hold_time` | 推定履歴を保持する時間 |

これらの設定により、一時的な認識失敗でも信号状態が頻繁に切り替わらないようになっています。 :contentReference[oaicite:7]{index=7}

---

# 利用シーン

`autoware_crosswalk_traffic_light_estimator` は、

- 歩行者信号が隠れている
- カメラで認識できない
- 一時的に検出が失敗した
- 点滅状態を推定したい

といった場面で利用されます。

推定された信号情報は、その後の

- Behavior Planning
- 交差点通過判断
- 横断歩道での停止判断

などに利用されます。 :contentReference[oaicite:8]{index=8}

---

# traffic_light_classifier との違い

| パッケージ | 主な役割 |
|------------|----------|
| **autoware_traffic_light_classifier** | カメラ画像から交通信号の色を認識する |
| **autoware_crosswalk_traffic_light_estimator** | 認識結果とHDマップを利用して歩行者信号を推定・補正する |

つまり、

- **Traffic Light Classifier** は「認識するモジュール」
- **Crosswalk Traffic Light Estimator** は「認識結果を補完・推定するモジュール」

という違いがあります。

---

# まとめ

- `autoware_crosswalk_traffic_light_estimator` は **歩行者用信号機の状態を推定する** Perceptionパッケージ
- 車両用信号とHDマップを利用して歩行者信号を補完する
- 一時的に認識できない場合でも信号状態を推定できる
- 点滅状態の推定や認識結果の補正も行う
- より安定した信号情報をPlanningモジュールへ提供する :contentReference[oaicite:9]{index=9}