# Tomato Squadron
ブラウザで動くシンプル縦スクロールシューティング。1 ファイル (`public/index.html`) にほぼ全ロジックをまとめ、Phaser 3 を CDN で読み込みます。

## 遊び方 / 操作
- 移動: 矢印キー または WASD
- 通常ショット: 自動 (レートは状態で変化)
- HYPER (高速連射一時化): SPACE （クールダウンあり）
- SUPER (広角多連射 & 取得時全方位バースト): 黄色スター状アイテム取得
- FEVER (3WAY & 移動微強化): 連続撃破数閾値到達 / トマトパワーアップ取得
- リトライ: Game Over 後に R

## 主なゲーム要素
- オブジェクトプーリング (敵 / 自弾 / 敵弾 / パワーアップ)
- FEVER / HYPER / SUPER の 3 段階火力変化
- 2 体のボス (HP バー付き / パターン弾幕)
- 背景: 多層スクロール + 星 + スピード線 + FEVER 時エフェクト増幅
- スタック/非アクティブ弾のクリーンアップ & 自動リカバリ
- 各種寿命 (弾 / パワーアップ) によるリソースリーク防止

## 主要パラメーター (index.html 冒頭付近)
| 変数                    | 役割                     | デフォルト | 調整の目安             |
| ----------------------- | ------------------------ | ---------- | ---------------------- |
| `SPAWN_INTERVAL`        | 敵出現の基礎間隔(ms)     | 550        | 小さくすると全体密度増 |
| `DIFFICULTY_STEP_MS`    | 難易度倍率更新間隔(ms)   | 8000       | 大きいほど成長ゆっくり |
| `DIFFICULTY_GROWTH`     | 更新ごとの難易度乗数     | 1.12       | 1.05～1.15 で調整      |
| `ENEMY_BASE_SPEED`      | 敵の基本落下速度         | 120        | スピード感全体に影響   |
| `ENEMY_BULLET_LIFETIME` | 敵弾寿命(ms)             | 8000       | 長くすると画面残留増   |
| `BULLET_LIFETIME`       | プレイヤー弾寿命(ms)     | 1600       | 連射可視時間を調整     |
| `POWERUP_LIFETIME`      | パワーアップ表示寿命(ms) | 8000       | 長くすると拾いやすい   |
| `FEVER_THRESHOLD`       | FEVER 必要連続撃破数     | 8          | 5～12 で頻度調整       |
| `FEVER_DURATION`        | FEVER 継続時間(ms)       | 3000       | 体感 2～5 秒推奨       |
| `HYPER_RATE`            | HYPER 時の発射間隔(ms)   | 40         | 乱れ過ぎるなら 55～65  |
| `HYPER_COOLDOWN`        | HYPER 再使用待ち(ms)     | 4500       | 連発抑制用             |
| `SUPER_DURATION`        | SUPER 継続時間(ms)       | 4200       | 4～6 秒程度            |
| `SUPER_BURST_COUNT`     | SUPER 取得直後の放射弾数 | 36         | 視認性と負荷で調整     |
| `SUPER_BULLET_SPEED`    | バースト弾速度           | 620        | 画面全域到達速度       |
| `CLEANUP_INTERVAL_MS`   | 無効オブジェクト掃除間隔 | 3000       | 頻繁すぎると負荷増     |

### 出現数ロジック
1 回スポーンでの敵数 `n = 1 + floor(min(4, difficulty - 1))` → difficulty が 5 以上で最大 5 体。ここを変えたい場合は式・上限を編集。

## 調整ガイド
- 敵が多すぎる: `SPAWN_INTERVAL` を 600 以上、`DIFFICULTY_GROWTH` を 1.08 など低めへ。
- 中盤以降が急にきつい: `DIFFICULTY_STEP_MS` を 10000 へ、または上限同時出現数の式を緩和。
- 弾幕持続を長く: `ENEMY_BULLET_LIFETIME` を 10000～12000 に。
- 画面が重い: STAR/LINE 生成数を減らす (create 時の 100 / 20 を調整)。

## 背景カスタム
`makeBackground` と `create()` の星/線初期化部分、および `update()` の星/線更新ロジックを編集。星の数・速度・色 (fillStyle / lineStyle) で雰囲気を変更可能。FEVER 時増加量はフィーバー開始判定内の push 回数を変更。

## 開発 / 実行
最小構成: 任意の静的サーバで `public/` を配信 (キャッシュ無効推奨)。

例 (Python):
```bash
python3 -m http.server 8000 --directory public
```
ブラウザで http://localhost:8000/ を開く。

## 構造
```
public/
  index.html   # 1 ファイルにゲーム本体 (Phaser CDN)
  boss1.png
  boss2.webp
  music.mp3
  favicon.ico
```

## パフォーマンス注意
- オブジェクトプール最大数 (`bullets/enemies/enemyBullets/powerups`) を小さくし過ぎると弾が出ないフレームが発生。
- 追加エフェクト (星/線) を増やしすぎるとモバイルでフレーム落ち。
- `console.log` は開発中のみ有効化推奨 (大量発火箇所あり)。

## 今後の拡張アイデア
- 難易度カーブをテーブル化 (経過時間→spawn/速度倍率)
- スコア倍率 / コンボ UI
- 設定メニュー (SFX / BGM / 視差強度 / 背景エフェクト ON/OFF)
- モバイルタッチ操作 (仮想スティック / ドラッグ移動)

## ライセンス / 素材
未設定。外部再配布する場合は各画像・音源のライセンスを明確化してください。現状の図形/パーティクルはコード生成。

---
調整や追加希望があれば遠慮なくどうぞ。
