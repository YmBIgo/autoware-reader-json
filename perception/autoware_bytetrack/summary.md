# autoware_bytetrack

## 概要

`autoware_bytetrack` は、Autoware Universe の **Perception** モジュールに含まれる **2Dマルチオブジェクトトラッキング** パッケージです。

カメラによる物体検出結果（2Dバウンディングボックス）をフレーム間で対応付けし、「同じ物体」を継続して追跡します。

Autowareでは、歩行者・自転車・車両などを毎フレーム検出しますが、そのままでは各フレームが独立しているため、「前のフレームの人」と「今見えている人」が同一人物なのか判断できません。

`autoware_bytetrack` は ByteTrack アルゴリズムを利用して、各物体に一貫した ID を付与し、時間方向に追跡できるようにします。 :contentReference[oaicite:0]{index=0}

---

# ByteTrackとは

ByteTrack は、2022 年に提案された高速・高精度なマルチオブジェクトトラッキング（MOT: Multi Object Tracking）アルゴリズムです。

従来のトラッカーは、検出スコアが高い物体だけを追跡対象としていました。

一方 ByteTrack は、

- 高スコアの検出結果
- 低スコアの検出結果

の両方を対応付けに利用します。

そのため、

- 一時的な遮蔽物
- 遠距離で検出信頼度が低い物体
- 見失いかけた物体

でも追跡を継続しやすくなっています。 :contentReference[oaicite:1]{index=1}

---

# Autowareでの役割

Perception パイプラインでは、おおよそ次の流れになります。

```text
Camera画像
      │
      ▼
2D物体検出
（YOLOなど）
      │
      ▼
autoware_bytetrack
      │
      ▼
Tracking済み2D物体
(ID付き)
```

ByteTrack は **検出器ではありません。**

検出器が出力した矩形（Bounding Box）を受け取り、

- これは前フレームの車と同じ
- この歩行者はID=12
- この自転車はID=5

のように対応付けを行います。

---

# 入力

主な入力は

- 2D Bounding Box付きの検出結果

です。

具体的には

```
DetectedObjectsWithFeature
```

が入力されます。 :contentReference[oaicite:2]{index=2}

---

# 出力

出力も同じく

```
DetectedObjectsWithFeature
```

ですが、

各物体が追跡された状態になります。

つまり、

- 継続したID
- 更新された追跡情報

が付加されたデータになります。 :contentReference[oaicite:3]{index=3}

---

# Autoware独自の工夫

元論文では Kalman Filter の状態として

- 左上座標
- アスペクト比
- サイズ

を使用しています。

しかし Autoware では、

物体が隠れる（Occlusion）とアスペクト比が大きく変化し、追跡が不安定になることがあります。

そのため、

- 左上座標
- サイズ

のみを状態量として利用するよう変更されています。

これにより、遮蔽物の多い実環境でも安定した追跡が期待できます。 :contentReference[oaicite:4]{index=4}

---

# 主なノード

## bytetrack_node

実際に ByteTrack を実行するノードです。

役割

- 検出結果を受信
- Kalman Filter による予測
- Bounding Box の対応付け
- Tracking ID の更新
- 結果を出力

Perception パイプラインで使用されるメインノードです。

---

## bytetrack_visualizer

Tracking結果を可視化するためのノードです。

主に開発・デバッグ用途で利用されます。

例えば

- ID番号表示
- Bounding Box表示
- Tracking結果確認

などを行います。 :contentReference[oaicite:5]{index=5}

---

# 利用シーン

`autoware_bytetrack` は、

- 車両
- 歩行者
- 自転車
- バイク

などの物体を継続的に追跡するために利用されます。

追跡された情報は、その後の

- 行動予測
- Planning
- 障害物回避

などのモジュールで活用されます。

---

# まとめ

- `autoware_bytetrack` は **2D物体追跡（Multi Object Tracking）** を担当するパッケージ
- ByteTrack アルゴリズムを利用して、検出された物体へ継続した ID を付与する
- 検出器ではなく、検出結果を時間方向に対応付ける役割を持つ
- 低信頼度の検出も活用するため、見失いにくい
- Autowareでは Kalman Filter を一部改良し、遮蔽物がある環境でも安定した追跡を実現している。 :contentReference[oaicite:6]{index=6}