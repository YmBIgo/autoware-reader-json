# autoware_ground_segmentation

## 概要

`autoware_ground_segmentation` は、LiDARで取得した点群から**地面（Ground）と障害物（Non-Ground）を分離**するためのパッケージです。

自動運転では、LiDARの点群には道路・歩道・建物・車両・歩行者など、周囲のすべての物体が含まれています。そのままでは物体検出が難しいため、まず地面の点群を取り除き、障害物だけを抽出します。 :contentReference[oaicite:0]{index=0}

---

## 主な役割

このパッケージの目的は、**地面を除去して障害物点群を生成すること**です。

地面を取り除くことで、

- 車両
- 歩行者
- 自転車
- コーン
- ガードレール

などの障害物だけを後続の認識処理へ渡せるようになります。

その結果、

- クラスタリング
- 物体検出
- 物体追跡

などの処理精度を向上させることができます。 :contentReference[oaicite:1]{index=1}

---

## どのような処理を行うのか

基本的な処理の流れは次のようになります。

```text
LiDAR点群
     │
     ▼
Ground Segmentation
     │
     ├── Ground Points
     └── Non-Ground Points
                 │
                 ▼
      物体検出・クラスタリング
```

入力された点群を解析し、

- 地面に属する点
- 障害物に属する点

へ分類します。 :contentReference[oaicite:2]{index=2}

---

## 主なアルゴリズム

`autoware_ground_segmentation` には、用途に応じて複数の地面除去アルゴリズムが用意されています。

### ray_ground_filter

LiDARのレーザーを放射状（Ray）に分割し、各Ray上の点の高さ変化を利用して地面を判定する方式です。

実装が比較的シンプルで、多くのLiDARで利用できます。 :contentReference[oaicite:3]{index=3}

---

### scan_ground_filter

`ray_ground_filter` を改良したアルゴリズムです。

点群をRayごとに並べ、距離や高さの変化を見ながら地面と障害物を判定します。

現在のAutowareでは、最も一般的に利用される地面除去方式の一つです。 :contentReference[oaicite:4]{index=4}

---

### ransac_ground_filter

RANSACを用いて地面を1枚の平面として近似する方式です。

比較的平坦な環境では有効ですが、大きな坂道や複雑な地形では適さない場合があります。 :contentReference[oaicite:5]{index=5}

---

## 入出力

### 入力

- `~/input/points`
  - `sensor_msgs/msg/PointCloud2`
  - LiDARから取得した点群

必要に応じて、対象点を限定するためのインデックス情報も利用できます。 :contentReference[oaicite:6]{index=6}

### 出力

- Ground PointCloud
  - 地面と判定された点群

- Non-Ground PointCloud
  - 障害物と判定された点群

後続の認識処理では、主に Non-Ground PointCloud が利用されます。 :contentReference[oaicite:7]{index=7}

---

## Autoware内での位置づけ

このパッケージは、LiDAR認識パイプラインの最初の段階で利用されます。

```text
LiDAR点群
     │
     ▼
Ground Segmentation
     │
     ▼
Euclidean Cluster
     │
     ▼
Shape Estimation
     │
     ▼
Object Tracking
```

地面を除去することで、後続のクラスタリングや物体認識が行いやすくなります。 :contentReference[oaicite:8]{index=8}

---

## 利用するメリット

このパッケージを利用することで、

- 地面と障害物を分離できる
- クラスタリングの精度が向上する
- 誤検出を減らせる
- 認識処理の計算量を削減できる
- 複数の地面除去アルゴリズムを環境に応じて選択できる

といったメリットがあります。 :contentReference[oaicite:9]{index=9}

---

## まとめ

`autoware_ground_segmentation` は、LiDAR点群から地面を除去し、障害物だけを抽出するためのパッケージです。

主なアルゴリズムとして、

- **ray_ground_filter**
- **scan_ground_filter**
- **ransac_ground_filter**

が用意されており、利用するLiDARや走行環境に応じて使い分けることができます。

Autowareの認識パイプラインでは、物体検出やクラスタリングの前処理として重要な役割を担っています。

---

## 参考

- Autoware Universe Documentation
  - https://autowarefoundation.github.io/autoware_universe/main/perception/autoware_ground_segmentation/
- Autoware Universe
  - https://github.com/autowarefoundation/autoware_universe/tree/main/perception/autoware_ground_segmentation