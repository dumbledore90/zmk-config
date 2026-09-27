# XIAO nRF52840 Plus  GPIO対応表（v2 確定版）

出典: Seeed Wiki ピンマップ**表** <https://wiki.seeedstudio.com/ja/XIAO_BLE/>

> 公式の**ピンアウト図**は D17 / D19 の物理パッド位置が入れ替わっている疑いがある
> （Seeedフォーラムで報告済み・Seeedも認知）。**表の D名↔チップピン対応は正しい**。
> ⚠️ 印のピンは、組み立て後に実機で挙動を確認すること。

## 右手 oyayubi_01_right

| 論理名 | XIAOパッド | XIAO名 | チップピン | devicetree | 備考 |
|---|---:|---|---|---|---|
| Row0 | 15 | D11 | P0.15 | `&gpio0 15` | |
| Row1 | 3 | D2 | P0.28 | `&gpio0 28` | |
| Row2 | 4 | D3 | P0.29 | `&gpio0 29` | |
| Row3 | 5 | D4 | P0.04 | `&gpio0 4` | |
| Row4 | 6 | D5 | P0.05 | `&gpio0 5` | |
| Col0 | 7 | D6 | P1.11 | `&gpio1 11` | |
| Col1 | 8 | D7 | P1.12 | `&gpio1 12` | |
| Col2 | 21 | D17 | P1.03 | `&gpio1 3` | ⚠️ 要実機検証 |
| Col3 | 9 | D8 | P1.13 | `&gpio1 13` | |
| Col4 | 22 | D18 | P1.05 | `&gpio1 5` | |
| Col5 | 10 | D9 | P1.14 | `&gpio1 14` | |
| Col6 | 23 | D19 | P1.07 | `&gpio1 7` | ⚠️ 要実機検証 |
| Col7 | 11 | D10 | P1.15 | `&gpio1 15` | |
| ENC_A | 2 | D1 | P0.03 | `&gpio0 3` | |
| ENC_B | 1 | D0 | P0.02 | `&gpio0 2` | |

```devicetree
row-gpios =
    <&gpio0 15 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>,   // Row0
    <&gpio0 28 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>,   // Row1
    <&gpio0 29 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>,   // Row2
    <&gpio0 4 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>,   // Row3
    <&gpio0 5 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>;   // Row4
```

```devicetree
col-gpios =
    <&gpio1 11 GPIO_ACTIVE_HIGH>,   // Col0
    <&gpio1 12 GPIO_ACTIVE_HIGH>,   // Col1
    <&gpio1 3 GPIO_ACTIVE_HIGH>,   // Col2
    <&gpio1 13 GPIO_ACTIVE_HIGH>,   // Col3
    <&gpio1 5 GPIO_ACTIVE_HIGH>,   // Col4
    <&gpio1 14 GPIO_ACTIVE_HIGH>,   // Col5
    <&gpio1 7 GPIO_ACTIVE_HIGH>,   // Col6
    <&gpio1 15 GPIO_ACTIVE_HIGH>;   // Col7
```

## 左手 oyayubi_01_left

| 論理名 | XIAOパッド | XIAO名 | チップピン | devicetree | 備考 |
|---|---:|---|---|---|---|
| Row0 | 1 | D0 | P0.02 | `&gpio0 2` | |
| Row1 | 11 | D10 | P1.15 | `&gpio1 15` | |
| Row2 | 10 | D9 | P1.14 | `&gpio1 14` | |
| Row3 | 9 | D8 | P1.13 | `&gpio1 13` | |
| Row4 | 8 | D7 | P1.12 | `&gpio1 12` | |
| Col0 | 15 | D11 | P0.15 | `&gpio0 15` | |
| Col1 | 2 | D1 | P0.03 | `&gpio0 3` | |
| Col2 | 3 | D2 | P0.28 | `&gpio0 28` | |
| Col3 | 4 | D3 | P0.29 | `&gpio0 29` | |
| Col4 | 5 | D4 | P0.04 | `&gpio0 4` | |
| Col5 | 6 | D5 | P0.05 | `&gpio0 5` | |
| Col6 | 7 | D6 | P1.11 | `&gpio1 11` | |
| TP_SCL | 16 | D12 | P0.19 | `&gpio0 19` | |
| TP_SDA | 17 | D13 | P1.01 | `&gpio1 1` | |
| TP_RDY | 21 | D17 | P1.03 | `&gpio1 3` | ⚠️ 要実機検証 |

```devicetree
row-gpios =
    <&gpio0 2 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>,   // Row0
    <&gpio1 15 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>,   // Row1
    <&gpio1 14 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>,   // Row2
    <&gpio1 13 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>,   // Row3
    <&gpio1 12 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>;   // Row4
```

```devicetree
col-gpios =
    <&gpio0 15 GPIO_ACTIVE_HIGH>,   // Col0
    <&gpio0 3 GPIO_ACTIVE_HIGH>,   // Col1
    <&gpio0 28 GPIO_ACTIVE_HIGH>,   // Col2
    <&gpio0 29 GPIO_ACTIVE_HIGH>,   // Col3
    <&gpio0 4 GPIO_ACTIVE_HIGH>,   // Col4
    <&gpio0 5 GPIO_ACTIVE_HIGH>,   // Col5
    <&gpio1 11 GPIO_ACTIVE_HIGH>;   // Col6
```

> **入出力の向きとプルの指定は暫定。** col2row のとき ZMK が row/col の
> どちらを駆動側にするかは `zmk,kscan-gpio-matrix` のドキュメントで確認して確定させること。
> ピン番号そのものは確定値なので、フラグだけの問題。

## エンコーダ（右手）

```devicetree
encoder: encoder {
    compatible = "alps,ec11";
    a-gpios = <&gpio0 3 (GPIO_ACTIVE_HIGH | GPIO_PULL_UP)>;   // ENC_A / D1
    b-gpios = <&gpio0 2 (GPIO_ACTIVE_HIGH | GPIO_PULL_UP)>;   // ENC_B / D0
    steps = <80>;
};
```

エンコーダのプッシュスイッチはマトリクスの **Row2 × Col0** に入る（D38経由）。

## トラックパッド（左手 / IQS7211E）

```devicetree
&i2c0 {
    // pinctrl で SCL=P0.19 (D12) / SDA=P1.01 (D13) に割り当てること
    iqs7211e: iqs7211e@56 {
        compatible = "azoteq,iqs7211e";
        reg = <0x56>;
        irq-gpios = <&gpio1 3 (GPIO_ACTIVE_LOW | GPIO_PULL_UP)>;   // TP_RDY / D17 ⚠️
        single-tap = <0>;
        double-tap = <0>;
        triple-tap = <0>;
    };
};
```

標準XIAOのI2C(D4/D5)ではなく Plus 固有ピンを使うため、**pinctrl の記述が必要**。

## 未使用・使用禁止

| ピン | チップピン | 理由 |
|---|---|---|
| D14 | P0.09 | NFC兼用 |
| D15 | P0.10 | NFC兼用 |
| **D16** | **P0.31** | **バッテリー電圧ADC直結。使用禁止** |

## 衝突チェック（実施済み）

- 使用: `P0.02 03 04 05 15 19 28 29` / `P1.01 03 05 07 11 12 13 14 15`
- システム占有: `P0.06`(LED_B) `P0.09 P0.10`(NFC) `P0.14`(BAT_EN) `P0.17`(CHG_LED)
  `P0.18`(Reset) `P0.26`(LED_R) `P0.30`(LED_G) `P0.31`(BAT_ADC) `P2.03 P2.05`(RF切替)

→ **重複ゼロ** ✅

## 実機で直す手順（テスター不要）

| 症状 | 原因 | 直し方 |
|---|---|---|
| 右手で Col2 のキーを押すと Col6 の文字が出る | D17/D19 の物理位置が逆 | overlay の Col2 と Col6 の行を入れ替える |
| 左手のトラックパッドが無反応 | RDY が P1.07 側だった | `irq-gpios` を `&gpio1 7` に変える |

どちらも1〜2行の修正 → push → 5分で再ビルド。
