# autoware_detection_by_tracker

## 概要

`autoware_detection_by_tracker` は、物体追跡（Tracker）の結果を物体検出（Detection）へフィードバックし、検出結果を安定化させるためのパッケージです。

通常の物体検出では、LiDAR の点群からクラスタリングを行って物体の形状を推定します。しかし、点群の欠損やノイズの影響により、物体が正しく分割できないことがあります。

このパッケージでは、追跡中の物体情報を利用して検出結果を補正し、より安定した物体検出を実現します。 :contentReference[oaicite:0]{index=0}

---

## 主な役割

物体検出では、次のような問題が発生します。

- 1台の車両が複数のクラスタに分割される（Over Segmentation）
- 複数台の車両が1つのクラスタとして認識される（Under Segmentation）
- フレームごとに物体の形状が大きく変化する

`autoware_detection_by_tracker` は、Tracker が保持している

- 位置
- 向き
- サイズ
- 過去フレームの情報

を利用して、検出結果を補正します。

その結果、

- 検出が途切れにくい
- 形状が安定する
- Tracking の品質も向上する

という効果があります。 :contentReference[oaicite:1]{index=1}

---

## どのような処理を行うのか

このパッケージでは、

- 新しく検出されたクラスタ
- Tracker が保持している物体

を比較します。

一致する Tracker が見つかった場合は、その Tracker の情報を参考にして検出物体の形状を調整します。

つまり、

```
LiDARで検出
        │
        ▼
Trackerと照合
        │
        ▼
Tracker情報を利用して形状を補正
        │
        ▼
より安定したDetectedObjectsを出力
```

という流れになります。 :contentReference[oaicite:2]{index=2}

---

## Over Segmentationへの対応

Over Segmentation は、

「1つの物体が複数のクラスタに分割される」

現象です。

例えば、

```
本来

□□□□

↓

検出結果

□□
  □□
```

のようになることがあります。

この場合、

- Tracker のサイズ
- Tracker の向き

を利用して、

複数のクラスタを1つの物体として扱うよう補正します。 :contentReference[oaicite:3]{index=3}

---

## Under Segmentationへの対応

Under Segmentation は、

「複数の物体が1つのクラスタになる」

現象です。

例えば、

```
車A 車B

□□□□ □□□□

↓

□□□□□□□□
```

のようになります。

この場合は、

- Tracker と検出クラスタを比較
- クラスタリング条件を変更
- 複数回試行
- IoU が最も高い結果を採用

という方法で、物体を適切に分割します。 :contentReference[oaicite:4]{index=4}

---

## 入出力

### 入力

- `~/input/initial_objects`
  - DetectedObjectsWithFeature
  - 点群クラスタから生成された検出物体

- `~/input/tracked_objects`
  - TrackedObjects
  - Multi Object Tracker が保持している追跡物体

### 出力

- `~/output`
  - DetectedObjects
  - Tracker情報で補正された物体情報 :contentReference[oaicite:5]{index=5}

---

## Autoware内での位置づけ

このパッケージは、

「Tracker の結果を Detection に戻す」

という少し特殊な役割を持っています。

```
LiDAR
   │
   ▼
物体検出
   │
   ▼
Multi Object Tracker
   │
   ▼
Detection By Tracker
   │
   ▼
補正されたDetectedObjects
```

一般的なパイプラインでは検出→追跡の一方向ですが、このパッケージは追跡結果を検出へフィードバックすることで、検出の安定性を高めます。 :contentReference[oaicite:6]{index=6}

---

## 利用するメリット

このパッケージを利用することで、

- 検出結果が安定する
- 一時的な点群欠損に強くなる
- 物体サイズが急激に変化しにくい
- Over Segmentation を抑制できる
- Under Segmentation を改善できる
- Tracking の品質向上につながる

といった効果が期待できます。 :contentReference[oaicite:7]{index=7}

---

## まとめ

`autoware_detection_by_tracker` は、Tracker が保持している情報を利用して、物体検出結果を補正するパッケージです。

通常の物体検出だけでは発生しやすい、

- Over Segmentation
- Under Segmentation
- 検出の不安定さ

を改善し、より信頼性の高い `DetectedObjects` を生成します。

そのため、Autoware の認識パイプラインにおいて、検出と追跡を連携させる重要な役割を担っています。

---

## 参考

- Autoware Universe Documentation
  - https://autowarefoundation.github.io/autoware_universe/main/perception/autoware_detection_by_tracker/
- Autoware Universe
  - https://github.com/autowarefoundation/autoware_universe/tree/main/perception/autoware_detection_by_tracker