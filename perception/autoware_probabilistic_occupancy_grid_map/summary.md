# autoware_probabilistic_occupancy_grid_map

## 概要

`autoware_probabilistic_occupancy_grid_map` は、**LiDARなどのセンサから取得したデータをもとに、周囲環境を Occupancy Grid Map（占有格子地図）として生成するパッケージ**です。

Occupancy Grid Map は、空間を一定サイズの格子（セル）に分割し、それぞれのセルに**「障害物が存在する確率」**を持たせた2次元マップです。このパッケージでは、センサから得られる点群情報を利用し、各セルの占有確率を継続的に更新します。さらに、**Binary Bayes Filter** を用いて複数フレームの観測結果を統合することで、一時的なノイズを抑えた安定したマップを生成します。 :contentReference[oaicite:0]{index=0}

---

## 主な役割

このパッケージでは、以下のような処理を行います。

- LiDAR点群からOccupancy Grid Mapを生成
- 障害物・自由空間・未知領域を推定
- Binary Bayes Filterによる占有確率の更新
- 後続の経路計画や障害物回避で利用できるマップを提供

---

## Occupancy Grid Mapとは

Occupancy Grid Mapでは、地図を細かなセルに分割し、それぞれのセルに障害物の存在確率を持たせます。

例：

```text
■■■■■■■■■■
■□□□□□□□■
■□□障害物□■
■□□□□□□□■
■■■■■■■■■■
```

セルごとに例えば次のような状態を保持します。

|状態|意味|
|----|----|
|Occupied|障害物が存在する可能性が高い|
|Free|障害物が存在しない|
|Unknown|まだ観測されていない|

このような確率マップを利用することで、単一フレームだけでは判断しにくい環境でも、より安定した障害物認識が可能になります。 :contentReference[oaicite:1]{index=1}

---

## 処理の流れ

大まかな処理は以下のようになります。

```text
Raw PointCloud
        │
        ▼
Obstacle PointCloud
        │
        ▼
Ray Tracing
        │
        ▼
Occupancy Grid生成
        │
        ▼
Binary Bayes Filter
        │
        ▼
Occupancy Grid Map
```

Ray Tracing により、LiDARから各点までの経路をたどって「自由空間」と「障害物セル」を推定し、その結果をBinary Bayes Filterで時系列的に統合します。 :contentReference[oaicite:2]{index=2}

---

## 入力

主な入力は以下のとおりです。

### Raw PointCloud

LiDARから取得した生の点群

型

- `sensor_msgs/msg/PointCloud2`

### Obstacle PointCloud

地面除去後などの障害物点群

型

- `sensor_msgs/msg/PointCloud2`

※ 構成によっては Raw PointCloud のみ、または複数LiDARからの点群を利用することもできます。 :contentReference[oaicite:3]{index=3}

---

## 出力

### Occupancy Grid

占有確率マップ

型

- `nav_msgs/msg/OccupancyGrid`

各セルには、

- 障害物
- 自由空間
- 未知領域

の情報が格納されます。

---

## Binary Bayes Filter

このパッケージの特徴の一つが **Binary Bayes Filter** による更新です。

毎フレーム新しいセンサ情報だけでマップを書き換えるのではなく、

```text
前回のOccupancy
        │
        ▼
新しい観測
        │
        ▼
確率更新
        │
        ▼
新しいOccupancy
```

という流れで占有確率を更新します。

これにより、

- 一瞬だけ現れたノイズ
- 一時的な観測漏れ
- センサ誤差

などの影響を軽減できます。 :contentReference[oaicite:4]{index=4}

---

## 主なパラメータ

代表的な設定項目です。

|パラメータ|内容|
|-----------|----|
|`fusion_map_length_x`|マップのX方向サイズ|
|`fusion_map_length_y`|マップのY方向サイズ|
|`fusion_map_resolution`|セルの解像度|
|`probability_matrix.*`|Bayes Filterの状態遷移確率|
|`downsample_input_pointcloud`|入力点群をダウンサンプリングするか|

これらを調整することで、マップの精度や計算負荷を用途に応じて変更できます。 :contentReference[oaicite:5]{index=5}

---

## 利用される場面

このノードは以下のような用途で利用されます。

- 障害物マップの生成
- 経路計画（Planning）
- 障害物回避
- 自由空間の推定
- センサ情報の統合

Occupancy Grid Map は、Autoware の Perception と Planning をつなぐ重要な地図情報として利用されます。 :contentReference[oaicite:6]{index=6}

---

## まとめ

`autoware_probabilistic_occupancy_grid_map` は、LiDAR点群をもとに Occupancy Grid Map を生成し、各セルに障害物の存在確率を保持するパッケージです。

Ray Tracing によって自由空間と障害物を推定し、Binary Bayes Filter によって複数フレームの観測を統合することで、ノイズに強く安定したマップを構築します。このマップは、Planning や障害物回避など、自動運転システムのさまざまなモジュールで活用される重要な基盤情報となっています。 :contentReference[oaicite:7]{index=7}