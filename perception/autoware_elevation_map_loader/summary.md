# autoware_elevation_map_loader

## 概要

`autoware_elevation_map_loader` は、点群地図（PointCloud Map）から高さ情報を持つ **Elevation Map（標高マップ）** を生成するためのパッケージです。

生成した Elevation Map は、主に `autoware_compare_map_segmentation` などの認識モジュールで利用され、地面や地図上の高さ情報を参照しながら物体認識の精度向上に役立てられます。

一度生成した Elevation Map はファイルとして保存できるため、次回以降は再計算せずに読み込むこともできます。 :contentReference[oaicite:0]{index=0}

---

## 主な役割

このパッケージでは、HDマップに含まれる点群地図から地面の高さを推定し、格子状（Grid Map）の高さマップを作成します。

主な役割は次のとおりです。

- 点群地図から高さマップを生成
- 地面の高さを格子単位で管理
- Elevation Map を保存・再利用
- 後続の認識モジュールへ高さ情報を提供

---

## どのような処理を行うのか

処理の流れは非常にシンプルです。

1. PointCloud Map を読み込む
2. 地図を一定サイズのセル（Grid）へ分割
3. 各セル内の地面の高さを計算
4. Elevation Map を生成
5. GridMap として配信・保存

概略図は次のようになります。

```text
PointCloud Map
        │
        ▼
セルへ分割
        │
        ▼
各セルの高さを計算
        │
        ▼
Elevation Map生成
        │
        ├── GridMapとしてPublish
        └── ファイルへ保存
```

---

## 高さの計算方法

各セルには複数の点が含まれる場合があります。

`autoware_elevation_map_loader` では、セル内の点群を解析し、**最も低いクラスタの平均高さ**をそのセルの標高として採用します。

これにより、

- 建物
- 樹木
- 車両

などではなく、できるだけ地面の高さを取得できるようになっています。 :contentReference[oaicite:1]{index=1}

---

## 欠損セルへの対応

点群が存在しないセルでは、高さを求めることができません。

そのようなセルに対しては、周囲のセルを利用して高さを補間（Inpaint）する機能が用意されています。

この機能を利用することで、

- 小さな穴
- 点群の欠損

による高さマップの不連続を減らすことができます。 :contentReference[oaicite:2]{index=2}

---

## 入出力

### 入力

- `input/pointcloud_map`
  - PointCloud Map

- `input/vector_map`（任意）
  - Lanelet2 地図

- `input/pointcloud_map_metadata`（任意）
  - 点群地図のメタデータ

### 出力

- `output/elevation_map`
  - GridMap形式の Elevation Map

- `output/elevation_map_cloud`
  - Elevation Map を点群化したデータ（任意） :contentReference[oaicite:3]{index=3}

---

## 保存機能

生成した Elevation Map はローカルへ保存できます。

そのため、

```text
初回起動
    │
PointCloud Mapから生成
    │
保存
```

次回以降は、

```text
保存済みElevation Map
        │
        ▼
そのまま読み込み
```

となり、起動時間を短縮できます。 :contentReference[oaicite:4]{index=4}

---

## Autoware内での位置づけ

このパッケージは、認識処理そのものを行うわけではありません。

認識モジュールが利用する「高さ情報」を事前に作成する役割を持っています。

```text
PointCloud Map
        │
        ▼
Elevation Map Loader
        │
        ▼
Elevation Map
        │
        ▼
Compare Map Segmentation
        │
        ▼
物体認識
```

特に `autoware_compare_map_segmentation` では、この高さマップを利用して静的な地図と現在の点群を比較し、障害物を抽出します。 :contentReference[oaicite:5]{index=5}

---

## 利用するメリット

このパッケージを利用することで、

- 地面の高さを高速に取得できる
- 認識処理で高さ情報を利用できる
- 地図との差分抽出の精度向上
- Elevation Map を再利用できるため起動が高速

といったメリットがあります。

---

## まとめ

`autoware_elevation_map_loader` は、PointCloud Map から **Elevation Map（標高マップ）** を生成するパッケージです。

生成した高さマップは、認識モジュールが地面の高さを参照するための基礎データとして利用されます。

また、一度作成した Elevation Map は保存・再利用できるため、大規模な地図でも効率よく運用できます。

---

## 参考

- Autoware Universe Documentation
  - https://autowarefoundation.github.io/autoware_universe/main/perception/autoware_elevation_map_loader/
- Autoware Universe
  - https://github.com/autowarefoundation/autoware_universe/tree/main/perception/autoware_elevation_map_loader