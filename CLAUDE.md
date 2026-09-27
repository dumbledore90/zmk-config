# oyayubi_01 — ZMK firmware

自作ワイヤレス分割キーボード **oyayubi_01** の ZMK 設定リポジトリ。

## ハードウェア

| | |
|---|---|
| MCU | **Seeed XIAO nRF52840 Plus** ×2（標準XIAOと別物。D11〜D19がある） |
| 右手 | 37キー + EC11互換ロータリーエンコーダ（プッシュスイッチ付き） |
| 左手 | 27キー + 円形トラックパッド φ30（Azoteq **IQS7211E**, I2C） |
| スイッチ | Kailh Choc V1 ホットスワップ（ソケットは裏面実装） |
| ピッチ | **18 × 17 mm**（標準19.05mmではない）、列スタガーあり |
| マトリクス | **col2row**、1N4148W、**ダイオードは全数裏面 (B.Cu)** |
| 電源 | Li-po 820mAh + スライドスイッチ + リセットスイッチ |
| ケース | サンドイッチ構造（トッププレート1.2mm / 基板1.6mm / ボトム1.6mm） |

## 正典（必ずここを参照する）

| ファイル | 内容 |
|---|---|
| `docs/xiao_gpio_map_v2.md` | **★最重要。** Row/Col ↔ XIAOピン ↔ nRF52840 GPIO の確定対応表 |
| `docs/pin_assignment_v2.md` | ピンアサインの全体像・マトリクスの埋まり方 |
| `docs/右手_配置座標表.md` | 右手38セルの Row/Col/座標/スタガー |
| `docs/左手_配置座標表.md` | 左手27セルの Row/Col/座標/スタガー |
| `docs/*_v2.kicad_pcb` | 基板の実物。疑わしいときはネット接続を直接読んで検証する |
| `docs/*_v2.kicad_sch` | 回路図。XIAOのピン番号↔D名の定義が入っている |

- **旧 `pin_assignment.md` は裏面ダイオード化前の値なので無効。絶対に参照しない。**
- 座標表の「指」の行は**推定値**。座標・Row/Col は基板からの実測値で確実。

## マトリクス構成（基板から実測・重複ゼロ）

### 左手 27セル

| 行 | 列 | セル数 |
|---|---|---:|
| Row0 | Col0〜Col4 | 5 |
| Row1 | Col0〜Col4 | 5 |
| Row2 | Col0〜Col4 | 5 |
| Row3 | Col0〜Col4 | 5 |
| Row4 | Col0〜**Col6** | 7 |

### 右手 38セル

| 行 | 列 | セル数 |
|---|---|---:|
| Row0 | **Col1**〜Col7 | 7 |
| Row1 | **Col1**〜Col7 | 7 |
| Row2 | Col0〜Col7 | 8 |
| Row3 | Col0〜Col7 | 8 |
| Row4 | Col0〜Col7 | 8 |

- **Row0 と Row1 に Col0 は存在しない。** 右手 Col0 は Row2/Row3/Row4 の3セルのみ
- **Row2 × Col0 = SW39 = ロータリーエンコーダのプッシュスイッチ**（D38経由）。通常のキーではない

### 統合レイアウト（ZMK）

- `columns = <15>` （左7列 = 0〜6、右8列 = 7〜14）
- `rows = <5>` （Row0 = 最上段 〜 Row4 = 最下段）
- 右の `col-offset = <7>`（雛形の初期値 2 から変更すること）
- **キー位置の総数 65**

行ごとの並び（左を左から右へ → 右を左から右へ）：

| 行 | 左 | 右 | その行のキー数 |
|---|---|---|---:|
| Row0 | Col0〜4 | Col1〜7 | 12 |
| Row1 | Col0〜4 | Col1〜7 | 12 |
| Row2 | Col0〜4 | Col0〜7 | 13 |
| Row3 | Col0〜4 | Col0〜7 | 13 |
| Row4 | Col0〜6 | Col0〜7 | 15 |
| | | **合計** | **65** |

**`map` の要素数・`keys` の要素数・keymap の `bindings` の要素数は、すべて 65 で一致させること。**
ここがズレるとビルドが落ちる。最初に転ぶならたいていここ。

## 設計方針

### 1. ZMK Studio / DYA Studio 対応を維持する

- `chosen` は **`zmk,physical-layout`**。`zmk,matrix-transform` を chosen にしない
- ただし **`default_transform`（compatible = "zmk,matrix-transform"）ノード自体は必須**。消さない
  （物理レイアウトがこのノードを `transform = <&default_transform>` で参照している）
- `oyayubi_01-layouts.dtsi` の `keys` プロパティに全65キーの座標を入れる
- `oyayubi_01.zmk.yml` の `features: - studio` を維持する
- キーマップに `&studio_unlock` を1つ置く
- `build.yaml` で左（central）に `snippet: studio-rpc-usb-uart` と
  `cmake-args: -DCONFIG_ZMK_STUDIO=y` を指定する

### 2. GPIO参照は overlay 2ファイルだけに隔離する

- `oyayubi_01_left.overlay` / `oyayubi_01_right.overlay` **以外に GPIO を書かない**
- **`row-gpios` は雛形だと `.dtsi` に書かれているが、左右で Row のピンが違うので overlay に移すこと**
- 実機検証でピンを直すとき、この2ファイルだけ触れば済むようにする

### 3. `&xiao_d` は使わない

- ZMK標準の `xiao_d` nexus は **D0〜D10 しか定義されていない**
- 本機は D11/D12/D13/D17/D18/D19 を使うため、**全ピンを `&gpio0 xx` / `&gpio1 xx` で直接指定**する
- ボードは `seeeduino_xiao_ble` を使う（チップは同じ nRF52840 なので直接指定なら問題ない）

### 4. kscan の設定

- `diode-direction = "col2row"` は**雛形の初期値で正しい**。変更しない
- col2row では **row が入力（`GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN`）、col が出力（`GPIO_ACTIVE_HIGH`）**

### 5. 左手を central にする

- `Kconfig.defconfig` の初期値（`SHIELD_OYAYUBI_01_LEFT` → `ZMK_SPLIT_ROLE_CENTRAL`）のまま
- トラックパッドが central 側にあると `zmk,input-split` が不要になり構成が単純になる

## ⚠️ 未確定事項：D17 / D19 の物理パッド位置

Seeed公式のピンアウト**図**に誤りが報告されている（**表**のD名↔チップピン対応は正しい）。
D17 と D19 の物理パッド位置が入れ替わっている可能性がある。

影響するのは以下の2信号のみ：

| 箇所 | 暫定値 | 症状が出たら |
|---|---|---|
| 右手 Col2 / Col6 | `&gpio1 3` / `&gpio1 7` | Col2を押すとCol6の文字が出る → overlayの2行を入れ替える |
| 左手 TP_RDY | `&gpio1 3` | トラックパッド無反応 → `&gpio1 7` に変更 |

**該当行には必ず `// TODO: 要実機検証` のコメントを付けること。**

これ以外のピン（D0〜D10, D11, D12, D13, D18）は標準XIAOと共通か、
図の誤りの影響を受けない位置にあるため確度が高い。

## トラックパッド（後回し）

**まずキーとエンコーダだけで動くものを完成させる。トラックパッドは最後。**

- ドライバは外部モジュール: <https://github.com/amgskobo/zmk-iqs7211e-driver>
  （`zmk module add` または `config/west.yml` に追加）
- I2Cアドレス `0x56`、`irq-gpios` が RDY ピン
- 必要な Kconfig: `CONFIG_I2C=y` `CONFIG_GPIO=y` `CONFIG_INPUT=y`
  `CONFIG_ZMK_POINTING=y` `CONFIG_IQS7211E=y`
- **I2Cは標準の D4/D5 ではなく D12/D13 に割り当てているため pinctrl の記述が必要**
- センサー表面に1〜2mmの素材（アクリル）を載せる前提。裸だと感度が出ない

## 禁止事項

- **`docs/` 配下のファイルを書き換えない**（ハードの実測値なので）
- **ピン番号を推測で埋めない。** `docs/xiao_gpio_map_v2.md` にない値が必要になったら
  手を止めて確認する
- 座標を勝手に丸めたり整数化したりしない。スタガーが崩れる

## 検証記録

**2026-09-27**: 基板ファイル（`*_v2.kicad_pcb`）と回路図（`*_v2.kicad_sch`）から
ピンアサインとマトリクスをゼロベースで再検証。

- 回路図のシンボル定義（pin15=D11 / pin21=D17 / pin22=D18 / pin23=D19）が
  `pin_assignment_v2.md` の pad↔D名 対応と一致
- 右手 XIAO 15本・左手 16本のネット接続がすべて表どおり
- pad20 (D16 / P0.31 = バッテリー電圧ADC) は左右とも未接続
- マトリクス 右38セル・左27セル、いずれも**重複ゼロ**
- 使用GPIOとXIAOのシステム占有ピン（LED・リセット・電池監視・アンテナ切替）に**重複なし**
- 4本指のスタガー値は左右で完全一致（人差4.25 / 中指12.75 / 薬指8.50）
