# autoware_image_object_locator

## 概要

`autoware_image_object_locator` は、**カメラ画像上で検出された2D物体（バウンディングボックス）から、3D空間上の物体位置を推定する**ためのパッケージです。

画像認識では、物体は「画像中の矩形（2D Bounding Box）」として検出されます。しかし、自動運転では「車両から何メートル先にあるか」といった3次元の位置が必要になります。

このパッケージは、カメラの内部パラメータやTF情報を利用して、2Dの検出結果を3Dの物体情報へ変換します。主な実装として **`bbox_object_locator`** が提供されています。 :contentReference[oaicite:0]{index=0}

---

## 主な役割

画像認識AIは、

- 車
- 歩行者
- 自転車

などを画像上で検出できますが、出力されるのは

- 画像中の位置
- バウンディングボックス
- クラス
- 信頼度

だけです。

`autoware_image_object_locator` は、これらの情報から

- 車両からの距離
- 3D位置
- 物体の幅
- 物体の高さ

を推定し、Autowareで利用できる `DetectedObjects` へ変換します。 :contentReference[oaicite:1]{index=1}

---

## どのような処理を行うのか

基本的な処理の流れは次のようになります。

```text
カメラ画像
      │
      ▼
2D物体検出
      │
      ▼
Bounding Box
      │
      ▼
Image Object Locator
      │
      ▼
3D DetectedObjects
```

画像上の物体を、自動運転で利用できる3次元物体情報へ変換する役割を担います。 :contentReference[oaicite:2]{index=2}

---

## 位置推定の仕組み

`bbox_object_locator` では、カメラモデルを利用して画像上の点を3次元空間へ逆投影（Back Projection）します。

主な処理は次のとおりです。

1. バウンディングボックス下端中央を地面へ投影
2. その位置を物体の中心位置として推定
3. 上端から物体の高さを推定
4. 左右端から物体の幅を推定

これらの情報を組み合わせて、3Dの物体サイズと位置を算出します。 :contentReference[oaicite:3]{index=3}

---

## 入出力

### 入力

- 2D物体検出結果（ROI）
- カメラキャリブレーション情報（CameraInfo）
- TF（座標変換情報）

### 出力

- `DetectedObjects`

出力される物体には、

- 位置
- サイズ
- クラス
- 信頼度

などの情報が含まれます。 :contentReference[oaicite:4]{index=4}

---

## 主なパラメータ

代表的な設定項目には次のようなものがあります。

| パラメータ | 内容 |
|------------|------|
| `target_frame` | 出力する座標系 |
| `roi_confidence_threshold` | ROIを採用する最小信頼度 |
| `detection_max_range` | 最大検出距離 |
| `pseudo_height` | 高さ推定できない場合の既定値 |
| `detection_target_class` | 対象とする物体クラス |

これらを調整することで、対象物体や推定条件を変更できます。 :contentReference[oaicite:5]{index=5}

---

## Autoware内での位置づけ

このパッケージは、カメラ認識パイプラインで利用されます。

```text
カメラ画像
      │
      ▼
2D物体検出
      │
      ▼
Image Object Locator
      │
      ▼
DetectedObjects
      │
      ▼
Object Tracking
      │
      ▼
Prediction
```

2D画像認識と3D認識をつなぐ役割を持っています。 :contentReference[oaicite:6]{index=6}

---

## 利用するメリット

このパッケージを利用することで、

- 2D画像認識を3D物体情報へ変換できる
- カメラのみでも物体位置を推定できる
- TrackingやPredictionへ入力できる形式に変換できる
- カメラ認識結果をAutoware全体で利用しやすくなる

といったメリットがあります。

---

## まとめ

`autoware_image_object_locator` は、**画像上で検出された2D物体を3D空間上の物体情報へ変換する**ためのパッケージです。

主な実装である **`bbox_object_locator`** は、カメラモデルと画像上のバウンディングボックスを利用して、

- 物体の位置
- 幅
- 高さ

を推定し、`DetectedObjects` として出力します。

このパッケージにより、カメラによる2D認識結果を、追跡や経路計画など後続のAutowareモジュールで活用できるようになります。 :contentReference[oaicite:7]{index=7}

---

## 参考

- Autoware Universe Documentation
  - https://autowarefoundation.github.io/autoware_universe/main/perception/autoware_image_object_locator/
- bbox_object_locator Documentation
  - https://autowarefoundation.github.io/autoware_universe/main/perception/autoware_image_object_locator/docs/bbox-object-locator/
- Autoware Universe
  - https://github.com/autowarefoundation/autoware_universe/tree/main/perception/autoware_image_object_locator