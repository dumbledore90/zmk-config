---
project: oyayubi_01
type: XIAO ピンアサイン（v2 / 裏面ダイオード版）
updated: 2026-09-25
source: 621f348c-oya01_Assemble_R_v2.kicad_pcb / 762d451e-oya01_Assemble_L_v2.kicad_pcb を実測
status: XIAOパッド↔Dxx は確定 / Dxx↔nRF52840 GPIO は要確認（後述）
---

# oyayubi_01 ピンアサイン v2

> **この表が正典。** 旧 `pin_assignment.md` は裏面ダイオード化の前の値なので使わないこと。
> v2で Row=B.Cu / Col=F.Cu に入れ替えた結果、XIAOのピン割り当ても全面的に変わっている。

## マトリクス構成（左右共通）

| 項目 | 値 |
|---|---|
| 方式 | **col2row** |
| ダイオード | 1N4148W (SOD-123)、**全数 裏面 (B.Cu)** |
| カソード(pad1) | 部品中心から **X −1.65mm**、**Row系ネットに接続** |
| アノード(pad2) | 部品中心から X +1.65mm、スイッチに接続 |
| 行の向き | **Row0 = 最上段 / Row4 = 最下段** |

---

## 右手 (oyayubi_01_right)

37キー + ロータリーエンコーダ / Row5本 × Col8本

### Row（ダイオードのカソード側）

| 論理名 | XIAOパッド | XIAO表記 | nRF52840 GPIO |
|---|---:|---|---|
| Row0 | 15 | D11 | TBD |
| Row1 | 3  | D2  | TBD |
| Row2 | 4  | D3  | TBD |
| Row3 | 5  | D4  | TBD |
| Row4 | 6  | D5  | TBD |

### Col（スイッチ側）

| 論理名 | XIAOパッド | XIAO表記 | nRF52840 GPIO |
|---|---:|---|---|
| Col0 | 7  | D6  | TBD |
| Col1 | 8  | D7  | TBD |
| Col2 | 21 | D17 | TBD |
| Col3 | 9  | D8  | TBD |
| Col4 | 22 | D18 | TBD |
| Col5 | 10 | D9  | TBD |
| Col6 | 23 | D19 | TBD |
| Col7 | 11 | D10 | TBD |

### その他

| 機能 | XIAOパッド | XIAO表記 |
|---|---:|---|
| エンコーダ A相 | 2  | D1 |
| エンコーダ B相 | 1  | D0 |
| リセットSW     | 26 | EN |
| 電池スライドSW | 28 | VBAT |
| GND            | 13 / 27 / 29 | — |
| VBUS           | 14 | — |

### マトリクスの埋まり方（右手）

- Row0 × Col1〜Col7 = 7キー
- Row1 × Col1〜Col7 = 7キー
- Row2 × Col0〜Col7 = 8キー（うち1つはエンコーダのプッシュ = D38経由）
- Row3 × Col0〜Col7 = 8キー
- Row4 × Col0〜Col7 = 8キー
- 合計 37キー + エンコーダ1 = ダイオード38個

**エンコーダのプッシュスイッチ**: `SW39.S1 → Col0` / `SW39.S2 → Net-(D38-A) → D38アノード` / `D38カソード → Row2`
つまりエンコーダのクリックは **Row2 × Col0** のマトリクス位置に入る。

---

## 左手 (oyayubi_01_left)

27キー + トラックパッド / Row5本 × Col7本

### Row（ダイオードのカソード側）

| 論理名 | XIAOパッド | XIAO表記 | nRF52840 GPIO |
|---|---:|---|---|
| Row0 | 1  | D0  | TBD |
| Row1 | 11 | D10 | TBD |
| Row2 | 10 | D9  | TBD |
| Row3 | 9  | D8  | TBD |
| Row4 | 8  | D7  | TBD |

### Col（スイッチ側）

| 論理名 | XIAOパッド | XIAO表記 | nRF52840 GPIO |
|---|---:|---|---|
| Col0 | 15 | D11 | TBD |
| Col1 | 2  | D1  | TBD |
| Col2 | 3  | D2  | TBD |
| Col3 | 4  | D3  | TBD |
| Col4 | 5  | D4  | TBD |
| Col5 | 6  | D5  | TBD |
| Col6 | 7  | D6  | TBD |

### トラックパッド (IQS7211E / I2C)

| 機能 | XIAOパッド | XIAO表記 |
|---|---:|---|
| SCL   | 16 | D12 |
| SDA   | 17 | D13 |
| RDY   | 21 | D17 |
| +3V3  | 12 | 3V3_OUT |

### その他

| 機能 | XIAOパッド | XIAO表記 |
|---|---:|---|
| リセットSW     | 26 | EN |
| 電池スライドSW | 28 | VBAT |
| GND            | 13 / 27 / 29 | — |
| VBUS           | 14 | — |

### マトリクスの埋まり方（左手）

- Row0〜Row3 × Col0〜Col4 = 20キー
- Row4 × Col0〜Col6 = 7キー
- 合計 27キー

---

## 使用していないピン（重要）

| XIAOパッド | XIAO表記 | 理由 |
|---|---|---|
| **20** | **D16 / P0.31** | **充電電圧モニタに直結。絶対に使わない** |
| 18 | D14 (NFC) | NFC兼用ピン。未使用 |
| 19 | D15 (NFC) | NFC兼用ピン。未使用 |
| 24 / 25 | SWD | デバッグ用。未使用 |

左右とも pad20 は未使用であることを実測で確認済み。

---

## ⚠️ ZMKを書く前にやること：GPIO番号の確定

上の表の `nRF52840 GPIO` 列が **TBD** になっている。ZMKのoverlayは
XIAOの「D11」みたいな呼び名ではなく、**nRF52840の `P0.xx` / `P1.xx`** か、
**`&xiao_d <n>` というnexusノード**で書く必要がある。

### 問題

ZMK標準の `seeeduino_xiao_ble` ボード定義が持つ `xiao_d` nexus は
**D0〜D10 の11本しか定義されていない**。

今回使っているのは **XIAO nRF52840 "Plus"** で、D11〜D19 という
**標準XIAOには存在しない追加ピン**を使っている：

- 右手: Row0=D11, Col2=D17, Col4=D18, Col6=D19, Col7=D10
- 左手: Col0=D11, TP_SCL=D12, TP_SDA=D13, TP_RDY=D17

### やること

1. **XIAO nRF52840 Plus のピンアウト資料**（Seeed Wiki / 回路図）から
   D11〜D19 が nRF52840 のどのGPIOポートに繋がっているかを調べる
2. この表の TBD を `P0.xx` / `P1.xx` で埋める
3. ZMKのoverlayでは D0〜D10 は `&xiao_d n`、D11以降は `&gpio0 xx` / `&gpio1 xx`
   で直接書く（または Plus用のボード定義を自作する）

**判明している手がかり**: pad20 = D16 = **P0.31**（充電電圧モニタ）

---

## 変更履歴

- **v2 (2026-09-25)**: 裏面ダイオード化に伴いピン割り当てを全面刷新。
  Row系をB.Cu、Col系をF.Cuに入れ替えたことで、旧版とは全く別の対応になっている。
  旧版 `pin_assignment.md` のピン表は**無効**。
