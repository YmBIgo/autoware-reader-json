# autoware_occupancy_grid_map_outlier_filter

## 概要

`autoware_occupancy_grid_map_outlier_filter` は、**Occupancy Grid Map（占有確率マップ）を利用して、LiDAR点群から外れ値（ノイズ）を除去するパッケージ**です。

通常の外れ値除去では点群の密度だけを利用しますが、このパッケージでは **Occupancy Grid Map が持つ「その場所に障害物が存在する確率」** を利用することで、一時的なノイズや誤検出をより高精度に取り除くことができます。:contentReference[oaicite:0]{index=0}

---

## 主な役割

このパッケージの役割は以下のとおりです。

- Occupancy Grid Mapを利用して点群の信頼性を判定
- 占有確率が低い点を候補として抽出
- ノイズと判断された点群を除去
- 後続の物体検出・追跡の精度向上

---

## 処理の流れ

大まかな処理は以下のようになります。

```text
Ground Segmentation済み点群
           │
           ▼
 Occupancy Grid Map
           │
           ▼
各点の占有確率を取得
           │
           ▼
占有確率が高い点
        │
        ├──そのまま保持
        │
        ▼
占有確率が低い点
        │
        ▼
（必要に応じて）
Radius Searchによる密度判定
        │
        ▼
外れ値を除去
        │
        ▼
フィルタ済みPointCloud
```

Occupancy Grid Mapだけではなく、必要に応じて**Radius Searchによる密度ベースの外れ値除去**を組み合わせることもできます。これにより、移動物体の一部など、占有確率が低くても本来残すべき点群を誤って除去しにくくなっています。:contentReference[oaicite:1]{index=1}

---

## 入力

### PointCloud

- 地面除去後の障害物点群

型

- `sensor_msgs/PointCloud2`

### Occupancy Grid Map

障害物の存在確率を保持するマップ

型

- `nav_msgs/OccupancyGrid`

---

## 出力

### フィルタ済みPointCloud

ノイズを除去した点群

### デバッグ用PointCloud

以下の情報も出力できます。

- 外れ値として除去した点群
- 占有確率が低い点群
- 占有確率が高い点群

デバッグ時にフィルタの挙動を確認できます。:contentReference[oaicite:2]{index=2}

---

## 主なパラメータ

代表的な設定項目です。

|パラメータ|内容|
|-----------|----|
|`cost_threshold`|障害物と判定する占有確率の閾値|
|`use_radius_search_2d_filter`|Radius Searchによる追加フィルタを有効化|
|`search_radius`|近傍探索半径|
|`min_points`|最低近傍点数|
|`enable_debugger`|デバッグ出力を有効化|

---

## 利用される場面

このノードは次のようなケースで役立ちます。

- LiDARの瞬間的なノイズ除去
- 鳥や雨などによる一時的な反射の除去
- 誤検出された障害物の削減
- 物体検出前の前処理

Occupancy Grid Mapには時間方向の情報が蓄積されるため、一瞬だけ現れたノイズを除去しやすいという特徴があります。:contentReference[oaicite:3]{index=3}

---

## まとめ

`autoware_occupancy_grid_map_outlier_filter` は、Occupancy Grid Map を利用して点群の信頼性を評価し、ノイズを除去するパッケージです。

通常の密度ベースフィルタよりも環境情報を活用できるため、一時的なノイズや誤検出を抑えつつ、後続の物体検出や追跡処理の精度向上に貢献します。