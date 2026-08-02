# autoware_predicted_path_postprocessor

## 概要

`autoware_predicted_path_postprocessor` は、**物体予測（Prediction）によって生成された予測経路（Predicted Path）を後処理し、より現実的で扱いやすい軌跡へ補正するパッケージ**です。

物体の将来経路は、予測アルゴリズムによって生成されますが、そのままでは現在の速度や周囲の静止物体を十分に考慮できていない場合があります。このパッケージでは、予測経路に対して追加の補正を行い、後続のプランニングや衝突判定で利用しやすい経路へ改善します。:contentReference[oaicite:0]{index=0}

---

## 主な役割

このパッケージでは、予測経路に対して以下のような後処理を行います。

- 現在の速度を考慮して経路を補正
- 静止物体を貫通する予測経路を修正
- 不自然な予測結果を減らす
- 後続モジュールが利用しやすい予測経路を生成

---

## 処理の流れ

大まかな処理の流れは以下のようになります。

```text
Predicted Objects
        │
        ▼
Predicted Paths
        │
        ▼
現在速度による補正
(RefineBySpeed)
        │
        ▼
静止物体との衝突補正
(RefinePenetrationByStaticObjects)
        │
        ▼
補正済みPredicted Objects
```

処理内容は設定ファイルで自由に組み合わせることができ、複数のプロセッサを順番に適用できます。:contentReference[oaicite:1]{index=1}

---

## 現在実装されている主なプロセッサ

### RefineBySpeed

現在の物体速度を考慮して予測経路を補正します。

例えば、

- 停止している物体
- 非常に低速な物体

などに対して、不自然に長い予測経路を生成しないよう調整します。:contentReference[oaicite:2]{index=2}

---

### RefinePenetrationByStaticObjects

予測経路が静止障害物を突き抜けてしまう場合に、それを修正します。

例えば、

- ガードレール
- 壁
- 建物
- 停車車両

などを貫通するような予測経路を改善し、より現実的な軌跡に補正します。:contentReference[oaicite:3]{index=3}

---

## 入力

### Predicted Objects

物体検出・追跡・予測によって生成された予測付き物体

型

- `autoware_perception_msgs/msg/PredictedObjects`

### Lanelet Map（任意）

道路情報を保持するHDマップ

型

- `autoware_map_msgs/msg/LaneletMapBin`

Lanelet Map は一部のプロセッサで利用されます。:contentReference[oaicite:4]{index=4}

---

## 出力

### Processed Predicted Objects

後処理された予測経路を持つ物体情報

型

- `autoware_perception_msgs/msg/PredictedObjects`

出力フォーマットは入力と同じですが、各物体が保持する予測経路が補正されています。:contentReference[oaicite:5]{index=5}

---

## 特徴

このパッケージには次のような特徴があります。

- プロセッサを自由に追加できる構成
- 複数の後処理を順番に適用可能
- 新しい補正アルゴリズムを追加しやすい設計
- Prediction モジュールの後段で利用可能

新しい処理は **RefineByXXX** や **FilterByXXX** といった形式で追加できるため、拡張性の高い設計となっています。:contentReference[oaicite:6]{index=6}

---

## 利用される場面

このノードは以下のような用途で利用されます。

- Prediction結果の品質向上
- 衝突判定の精度向上
- Planningモジュールへの入力改善
- 不自然な予測経路の修正

予測経路をそのまま利用するのではなく、一度後処理を行うことで、より現実的な挙動を前提としたプランニングが可能になります。:contentReference[oaicite:7]{index=7}

---

## まとめ

`autoware_predicted_path_postprocessor` は、Prediction モジュールが生成した予測経路を後処理し、現在の速度や周囲の静止物体を考慮したより自然な経路へ補正するパッケージです。

処理内容はプロセッサとしてモジュール化されており、必要に応じて複数の補正を組み合わせることができます。そのため、Prediction の精度向上だけでなく、後続の Planning や衝突判定の信頼性向上にも貢献する重要なパッケージです。:contentReference[oaicite:8]{index=8}