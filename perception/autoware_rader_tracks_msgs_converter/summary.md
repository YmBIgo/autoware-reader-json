# autoware_radar_tracks_msgs_converter

## 概要

`autoware_radar_tracks_msgs_converter` は、レーダーが出力する `radar_msgs/msg/RadarTracks` メッセージを、Autoware の Perception モジュールで利用できるメッセージ形式へ変換するパッケージです。

レーダードライバはメーカーごとに異なるメッセージ形式や分類ラベルを使用することがありますが、このモジュールを利用することで、Autoware が共通で扱う `DetectedObject` および `TrackedObject` に変換できます。

そのため、レーダーを利用した物体検出・物体追跡・センサフュージョンの入口となる重要な変換モジュールです。:contentReference[oaicite:0]{index=0}

---

# 主な役割

このモジュールは主に以下の処理を行います。

- RadarTracks を Autoware のメッセージへ変換
- レーダーの物体ラベルを Autoware の分類へ変換
- 必要に応じて座標系（frame_id）の変換
- 自車速度を利用した速度補正（Twist Compensation）
- 静止物体・移動物体の判定

高度な認識アルゴリズムを行うのではなく、「レーダーデータをAutoware標準形式へ統一する」ことが目的です。:contentReference[oaicite:1]{index=1}

---

# 入力

主な入力は以下です。

| Topic | 型 | 内容 |
|-------|----|------|
| `~/input/radar_objects` | `radar_msgs/msg/RadarTracks` | レーダーで検出した物体一覧 |
| `~/input/odometry` | `nav_msgs/msg/Odometry` | 自車速度・姿勢情報 |

Odometry は速度補正や座標変換に利用されます。:contentReference[oaicite:2]{index=2}

---

# 出力

変換後は以下のトピックを出力します。

| Topic | 型 | 用途 |
|-------|----|------|
| `~/output/radar_detected_objects` | `DetectedObjects` | レーダー物体検出結果 |
| `~/output/radar_tracked_objects` | `TrackedObjects` | レーダー追跡結果 |

この出力は後続のセンサフュージョンやトラッキングモジュールで利用されます。:contentReference[oaicite:3]{index=3}

---

# 処理の流れ

```text
Radar Driver
      │
      ▼
RadarTracks
      │
      ▼
autoware_radar_tracks_msgs_converter
      │
      ├──────────────► DetectedObjects
      │
      └──────────────► TrackedObjects
                              │
                              ▼
      Radar Fusion / Object Tracker
```

---

# 主な設定項目

代表的なパラメータは以下です。

| パラメータ | 内容 |
|------------|------|
| `update_rate_hz` | 出力周期 |
| `new_frame_id` | 出力座標系（通常は `base_link`） |
| `use_twist_compensation` | 自車速度補正を有効化 |
| `use_twist_yaw_compensation` | ヨー方向の補正を有効化 |
| `static_object_speed_threshold` | 静止物体と判断する速度閾値 |

環境やレーダー性能に応じて調整できます。:contentReference[oaicite:4]{index=4}

---

# 他モジュールとの関係

このモジュールは Perception パイプラインの前段に位置します。

```text
Radar Driver
      │
      ▼
autoware_radar_tracks_msgs_converter
      │
      ▼
DetectedObjects / TrackedObjects
      │
      ├────────► autoware_radar_fusion_to_detected_object
      │
      ├────────► autoware_multi_object_tracker
      │
      └────────► autoware_radar_object_tracker
```

レーダーデータをAutoware標準形式へ統一することで、後続モジュールがセンサー依存を意識せず利用できるようになります。:contentReference[oaicite:5]{index=5}

---

# まとめ

- RadarTracks を Autoware 標準メッセージへ変換するモジュール
- DetectedObjects と TrackedObjects を生成
- ラベル変換や座標変換、自車速度補正を担当
- レーダー認識アルゴリズムではなく「データ変換・標準化」が主目的
- レーダーを利用する Perception モジュールの入口となる重要なパッケージ