# autoware_multi_object_tracker

## 概要

`autoware_multi_object_tracker` は、**物体検出結果を時系列で追跡（Tracking）するPerceptionモジュール**です。

物体検出器（CenterPointやBEVFusionなど）が毎フレーム出力する検出結果に対して、

- 一意のIDを付与
- 速度を推定
- 過去の状態を維持

することで、「同じ物体」を継続して追跡します。:contentReference[oaicite:0]{index=0}

---

# 役割

このモジュールの役割は、

> **検出された物体を継続的に追跡し、IDと運動状態を管理すること**

です。

例えば、車が前方を走行している場合、

```
時刻 t0
車A

時刻 t1
車A

時刻 t2
車A
```

のように、毎フレーム同じIDを維持します。

これにより、

- 同じ車なのか
- 新しく現れた車なのか
- 消えた車なのか

を判断できるようになります。:contentReference[oaicite:1]{index=1}

---

# なぜ必要？

物体検出器は各フレームを独立して処理するため、

```
Frame1
車

Frame2
車

Frame3
車
```

という検出結果だけでは、

- 同じ車なのか
- 別の車なのか

が分かりません。

そこでTrackingを行うことで、

- IDの維持
- 速度推定
- 一時的な検出漏れへの対応
- 将来位置予測のための情報生成

が可能になります。:contentReference[oaicite:2]{index=2}

---

# 特徴

このモジュールの特徴は次のとおりです。

- 複数物体を同時に追跡
- 各物体へ一意のIDを付与
- EKF（Extended Kalman Filter）による状態推定
- データアソシエーションによる対応付け
- 車・歩行者・自転車などクラスごとに異なる追跡モデルを使用

Autowareでは、物体の種類に応じて適切な運動モデルを使い分けることで、より安定した追跡を実現しています。:contentReference[oaicite:3]{index=3}

---

# 処理の流れ

基本的な処理は次のようになります。

```
Detected Objects
        │
        ▼
Data Association
（対応付け）
        │
        ▼
EKF Tracking
        │
        ▼
ID更新
速度推定
姿勢推定
        │
        ▼
Tracked Objects
```

1. 新しい検出結果を受信
2. 既存トラックとの対応付け（Data Association）
3. EKFで位置・速度を更新
4. 新しい物体は新規トラックを作成
5. TrackedObjectsとして出力

:contentReference[oaicite:4]{index=4}

---

# Data Associationとは？

Trackingで最も重要なのが、

> **今回検出された物体が、前回のどの物体と同じかを判断する処理**

です。

例えば、

```
前フレーム

ID1
ID2

↓

現在フレーム

車A
車B
```

という状況で、

```
ID1 ← 車A

ID2 ← 車B
```

のように対応付けを行います。

Autowareでは、この対応付けを**最小コスト最大フロー（Min Cost Max Flow）問題**として解き、`muSSP` ソルバーを利用しています。また、物体クラスやBEV上の面積、マハラノビス距離、最大距離などを用いたゲート条件によって、不適切な対応付けを防いでいます。:contentReference[oaicite:5]{index=5}

---

# EKF（Extended Kalman Filter）

対応付け後は、EKFを用いて物体の状態を更新します。

EKFでは、

- 位置
- 速度
- 向き

などを継続的に推定します。

これにより、

- 一時的に物体が見えなくなった場合
- センサノイズがある場合

でも、安定した追跡が可能になります。:contentReference[oaicite:6]{index=6}

---

# 入力

主な入力は次のとおりです。

| トピック | 型 | 内容 |
|-----------|----|------|
| `~/input` | `DetectedObjects` | 物体検出結果 |

入力には、物体検出モジュールが出力した `DetectedObjects` を使用します。:contentReference[oaicite:7]{index=7}

---

# 出力

主な出力は次のとおりです。

| トピック | 型 | 内容 |
|-----------|----|------|
| `~/output` | `TrackedObjects` | 追跡結果 |

出力には、

- トラックID
- 位置
- 速度
- 向き
- クラス情報

などが含まれます。:contentReference[oaicite:8]{index=8}

---

# Perception内での位置付け

```
LiDAR / Camera
        │
        ▼
Object Detection
        │
        ▼
Detected Objects
        │
        ▼
Multi Object Tracker
        │
        ▼
Tracked Objects
        │
        ▼
Prediction
        │
        ▼
Planning
```

物体検出とPredictionの間に位置し、時系列情報を付加する重要な役割を担います。:contentReference[oaicite:9]{index=9}

---

# まとめ

- 物体検出結果を時系列で追跡するTrackingモジュール
- 各物体へ一意のIDを付与
- EKFによって位置・速度・向きを推定
- Data Associationで検出結果と既存トラックを対応付け
- `TrackedObjects` を生成し、PredictionやPlanningへ受け渡す重要なモジュール