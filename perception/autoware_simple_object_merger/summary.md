# autoware_simple_object_merger

## 概要

`autoware_simple_object_merger` は、**複数の `DetectedObjects` トピックを1つに統合するためのシンプルなマージモジュール**です。

複数のレーダーや複数の物体検出器から得られた検出結果を、**低い計算コスト**で1つの `DetectedObjects` としてまとめることを目的としています。

`autoware_object_merger` のようなデータ対応付け（Data Association）は行わず、入力された物体をそのまま結合するため、高速に動作する点が特徴です。:contentReference[oaicite:0]{index=0}

---

# 主な役割

このモジュールでは主に以下の処理を行います。

- 複数の `DetectedObjects` トピックを受信
- 各トピックの物体一覧を1つにまとめる
- タイムアウトした入力を除外
- 出力フレーム（frame_id）を統一

物体同士の対応付けや重複除去は行わず、**シンプルな統合処理**のみを担当します。:contentReference[oaicite:1]{index=1}

---

# 入力

入力するトピック数はパラメータで自由に設定できます。

| Topic | 型 | 内容 |
|-------|----|------|
| `input_topics` | `DetectedObjects` | 複数の検出結果 |

例えば、

- Radar Front
- Radar Left
- Radar Right
- Radar Rear

など複数のセンサーからの検出結果を同時に入力できます。:contentReference[oaicite:2]{index=2}

---

# 出力

| Topic | 型 | 内容 |
|-------|----|------|
| `~/output/objects` | `DetectedObjects` | 統合後の検出結果 |

すべての入力トピックに含まれる物体をまとめた一覧を出力します。:contentReference[oaicite:3]{index=3}

---

# 処理の流れ

```text
Radar Front
        │
Radar Left
        │
Radar Right
        │
Radar Rear
        │
        ▼
autoware_simple_object_merger
        │
        ▼
Merged DetectedObjects
```

---

# 主な設定項目

代表的なパラメータは以下です。

| パラメータ | 内容 |
|------------|------|
| `input_topics` | 入力するトピック一覧 |
| `update_rate_hz` | 出力周期 |
| `new_frame_id` | 出力フレームID |
| `timeout_threshold` | タイムアウト判定時間 |

タイムアウトした入力トピックはマージ対象から除外されるため、一部のセンサーが停止しても処理を継続できます。:contentReference[oaicite:4]{index=4}

---

# 他モジュールとの違い

`autoware_simple_object_merger` と `autoware_object_merger` の違いは以下の通りです。

| モジュール | 特徴 |
|-----------|------|
| **autoware_simple_object_merger** | 物体をそのまま結合するだけ。高速で複数トピックに対応。 |
| **autoware_object_merger** | 物体同士を対応付け（Data Association）して重複を除去・統合する。計算コストは高い。 |

そのため、複数レーダーの検出結果をまず1つのトピックへまとめたい場合などに `autoware_simple_object_merger` が適しています。:contentReference[oaicite:5]{index=5}

---

# 利用例

代表的な利用例は、複数レーダーの検出結果を統合するケースです。

```text
Front Radar
        │
Left Radar
        │
Right Radar
        │
Rear Radar
        │
        ▼
autoware_simple_object_merger
        │
        ▼
Merged Radar Objects
        │
        ▼
Radar Fusion
        │
        ▼
Multi Object Tracker
```

複数のレーダーを1つの入力として扱えるため、その後の認識パイプラインをシンプルに構成できます。:contentReference[oaicite:6]{index=6}

---

# 制限事項

このモジュールには以下の制限があります。

- 重複する物体は除去されない
- データ対応付け（Data Association）は行わない
- 同じ物体が複数回出力される可能性がある
- 必要に応じて後段で重複除去やフュージョンを行うことが前提

そのため、**高速な統合処理を優先する場面**で利用されます。:contentReference[oaicite:7]{index=7}

---

# まとめ

- 複数の `DetectedObjects` を高速に統合するモジュール
- データ対応付けは行わず、そのまま結合する
- 複数レーダーなど多数の入力トピックを扱える
- 計算コストが低く、リアルタイム処理に適している
- 重複除去は行わないため、必要に応じて後続モジュールで処理する