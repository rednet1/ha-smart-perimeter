[English](../entities.md) | [Italiano](entities.it.md) | [Español](entities.es.md) | [日本語](entities.ja.md)

# セットアップ用ワークシート

ブループリントをインポートする前に、自分の環境のエンティティをここに記入しておきましょう。各ブループリントの入力フォームをクリックしながら1つずつ調べるより、あらかじめ一箇所にまとめておく方が楽です。

| 役割 | ドメインの例 | あなたのエンティティ |
|---|---|---|
| アーム状態ヘルパー | `input_boolean.*` | |
| パーシャルモードフラグ | `input_boolean.*` または余っている `automation.*` | |
| サイレン | `switch.*` | |
| LED/視覚インジケーター(任意) | `light.*` | |
| 認証IDセンサー(指紋/キーパッド/NFC) | `sensor.*` | |
| モーションセンサー | `binary_sensor.*` (device_class: motion) | |
| ドア/窓センサー | `binary_sensor.*` (device_class: door/window/opening) | |
| ペット在室センサー(カメラごと) | `binary_sensor.*` | |
| 在宅トラッカー(自動アーム用) | `person.*` / `device_tracker.*` | |
| 音声通知先(任意) | `assist_satellite.*` | |

人物ごとの表(1行 → `person_trigger.yaml`オートメーションのインスタンス1つ):

| 人物 | センサー上のID | メモ |
|---|---|---|
| | | |
| | | |
