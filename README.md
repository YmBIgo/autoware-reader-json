# Autoware Reader

Autoware Reader は、Autoware Universe のソースコードを **Linux Reader (https://linux.tokyo)** のように、ブラウザ上でインタラクティブに読み進められるようにするためのオープンソースプロジェクトです。

Autoware は数百の ROS 2 パッケージと多数のノードから構成されており、初学者だけでなく経験者でも全体像を把握することは容易ではありません。本プロジェクトでは、ソースコードの構造を解析し、ノード・クラス・関数・ROS 2 の通信関係を可視化することで、「Autoware を読むための地図」を提供することを目的としています。

このレポジトリでは、その中のJSONの管理をします。

---

## 注意点

下記モジュールは、clangdが上手くインデックスできていないので、いったん飛ばしています。

```
perception/autoware_ground_segmentation_cuda
```

---

## Demo

（公開後に URL を記載）

```
https://autoware.jp (まだ未公開)
```

---

## Features

### ノード単位でコードを探索

Autoware Universe に存在する各ノードを一覧表示し、エントリーポイントから順番にコードを追うことができます。

例

```
compare_map_segmentation
 ├── DistanceBasedCompareMapFilterNode
 │    ├── onPointCloud()
 │    ├── filter()
 │    └── is_close_to_map()
 └── ...
```

---

### 関数呼び出しツリー

各関数から呼び出される関数を木構造で表示します。

```
pointcloud_callback()

 ├── convertPointCloud()
 ├── voxelFilter()
 ├── segmentGround()
 ├── removeStaticObjects()
 └── publish()
```

クリックするだけで次の関数へ移動できます。

---

### コードジャンプ

クリックすると GitHub 上の該当行へジャンプできます。

```
filter()

↓

node.cpp:L203
```

---

## 対象

現在は

* Autoware Universe

を対象としています。

将来的には

* Autoware Core
* Autoware.Auto
* 独自パッケージ

にも対応予定です。

---

## アーキテクチャ

```
Autoware Source

        │

        ▼

clang AST

        │

        ▼

JSON Generator

        │

        ▼

reader.json

        │

        ▼

Web Viewer
```

解析フェーズでは C++ の AST を利用し、

* 関数呼び出し
* クラス
* 継承
* include
* 名前空間
* ROS 2 ノード
* Publisher
* Subscriber

などを JSON に変換します。

ビューアは JSON のみを読み込み、ブラウザ上で高速に描画します。

---

## reader.json

解析結果は JSON として保存されます。

例

```json
{
  "nodes": [
    {
      "name": "DistanceBasedCompareMapFilterNode",
      "functions": [
        "onPointCloud",
        "filter",
        "publish"
      ]
    }
  ]
}
```

JSON を差し替えるだけで、別バージョンの Autoware に対応できます。

---

## ディレクトリ構成

```
autoware-reader/

├── perception/
│   ├── autoware_bevfusion
│   ├── autoware_camera_streampetr
│   └── ...
│
├── plannig/
│   ├── TBD...
│   ├── ...
│   └── ...
│
├── sensing/
│   └── TBD...
│
└── README.md
```

---

## Roadmap

TBD

---

## Motivation

Autoware は世界でも最大級のオープンソース自動運転ソフトウェアですが、ソースコードを理解するための入り口は十分とは言えません。

Linux カーネルには Linux Reader のような優れた学習ツールがあります。一方で、Autoware には「どこから読めばよいか」「この関数はどこから呼ばれるのか」「このノードは何をしているのか」を直感的に理解できるツールがほとんどありません。

Autoware Reader は、そうした課題を解決し、初学者から研究者・開発者までが効率よくコードを探索できる環境を提供することを目指しています。

---

## License

MIT
