# autoware_image_projection_based_fusion

## 概要

`autoware_image_projection_based_fusion` は、**カメラとLiDARの認識結果を統合（センサフュージョン）するためのPerceptionモジュール**です。

Autowareでは、

- カメラ：物体の種類（車・歩行者・信号など）の認識が得意
- LiDAR：物体の位置や形状を高精度に取得するのが得意

という特徴があります。

このパッケージでは、それぞれの長所を組み合わせることで、**より高精度な物体認識**を実現します。:contentReference[oaicite:0]{index=0}

---

# 役割

主な役割は、

> **画像上の認識結果（ROIやセグメンテーション）を3D情報へ反映すること**

です。

例えば、

```
Camera
   │
   ├── 車を検出
   ├── 歩行者を検出
   └── 信号を検出
          │
          ▼
Image Projection Based Fusion
          │
          ▼
LiDARで検出した3D物体へ反映
          │
          ▼
高精度なDetectedObjects
```

という流れになります。:contentReference[oaicite:1]{index=1}

---

# なぜ必要？

LiDARだけの場合は

- 位置は正確
- 物体の種類が分かりにくい

という弱点があります。

逆にカメラだけでは

- 車か人かは分かる
- 距離精度はLiDARほど高くない

という特徴があります。

そこで、

- LiDARの位置情報
- Cameraの認識結果

を組み合わせることで、

- より正確な物体分類
- 未知物体の検出
- 誤検出の削減

が可能になります。:contentReference[oaicite:2]{index=2}

---

# 主なFusionアルゴリズム

このパッケージには複数のFusion方式が含まれています。

| モジュール | 概要 |
|------------|------|
| roi_cluster_fusion | LiDARクラスタへ画像のクラス情報を付与 |
| roi_detected_object_fusion | 既存DetectedObjectのラベルを更新 |
| pointpainting_fusion | 点群へ画像特徴を付与して3D検出器へ入力 |
| roi_pointcloud_fusion | ROIと点群を対応付けて未知物体を検出 |
| segmentation_pointcloud_fusion | セグメンテーション結果を利用して不要な点群を除去 | :contentReference[oaicite:3]{index=3}

このように、「画像情報を3D認識へ活用する」ための複数の手法がまとめられています。

---

# 処理の流れ

基本的な処理は次のようになります。

```
Camera
      │
      ▼
2D Object Detection
      │
      ▼
ROI
      │
      │
      ├────────────┐
      │            │
      ▼            ▼
                 LiDAR
                   │
             PointCloud
                   │
                   ▼
Image Projection Based Fusion
                   │
                   ▼
Detected Objects
```

処理の流れは大きく以下の4段階です。

1. カメラとLiDARのデータを時刻で対応付ける
2. ROIと3Dデータを対応付ける
3. Fusionアルゴリズムを実行する
4. 融合後の認識結果を出力する

センサは異なる周期で動作するため、タイムスタンプを用いた同期処理が重要な役割を担っています。:contentReference[oaicite:4]{index=4}

---

# 入力

代表的な入力は次のとおりです。

- PointCloud
- DetectedObjects
- Cluster
- ROI（2D検出結果）
- CameraInfo
- Image

Fusion方式によって利用する入力は異なります。:contentReference[oaicite:5]{index=5}

---

# 出力

Fusion後には、

- DetectedObjects
- PointCloud
- Cluster

などが出力されます。

どのデータが出力されるかは、使用するFusionアルゴリズムによって異なります。:contentReference[oaicite:6]{index=6}

---

# Perception内での位置付け

```
Camera
      │
      ▼
2D Detector
      │
      ▼
ROI
            ┐
            │
LiDAR ──────┤
            │
            ▼
Image Projection Based Fusion
            │
            ▼
Detected Objects
            │
            ▼
Tracking
            │
            ▼
Prediction
```

物体検出とTrackingの間に位置し、複数センサの情報を統合する役割を担います。:contentReference[oaicite:7]{index=7}

---

# まとめ

- CameraとLiDARを統合するPerceptionモジュール
- 画像の認識結果を3D認識へ反映する
- 複数のFusionアルゴリズムを提供
- タイムスタンプ同期を行いながらセンサフュージョンを実施
- より高精度な物体認識を実現するための重要なモジュール