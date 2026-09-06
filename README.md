# Corcell ZMK ファームウェア

Corcell は、PAW3222 トラックボールと乾電池駆動に対応した ZMK ファームウェアです。
キー配線、matrix transform、physical layout、charlieplex kscan は Corchibi 互換です。

**このブランチ（`lenotp`）は、FPC スロット 1 のセンサーを PAW3222 から
lenoTP に置き換えた実験中の版です。一般利用には `main` または `dya-studio` を選んでください。**
通常版は `main`、DYA Studio 対応版は `dya-studio` ブランチです。

lenoTP モジュールは PAW3222 とピン配置互換なので、スロット 1 のネットを
そのまま流用します。

| FPC | lenoTP | PAW3222 での役割 | XIAO |
|---|---|---|---|
| 1 | NC | NCS | P0.05（RE_B）・未使用 |
| 2 | SDA | SDIO | `P0.09` |
| 3 | SCL | SCLK | `P0.10` |
| 4 | INT | MOTION | `P1.12` |
| 5 | VCC | VCC | 3V3 |
| 6 | GND | GND | GND |

- ドライバは [`zmk-driver-lenotp`](https://github.com/yuchamichami/zmk-driver-lenotp) です。
  I2C アドレスは `0x15`、`INT` は `GPIO_ACTIVE_LOW | GPIO_PULL_UP` で受けます。
- nRF52840 の `i2c0` と `spi0` は同一インスタンス（どちらも `0x40003000`）なので、
  snippet 側で `spi0` を無効化しています。PAW3222 との併用はできません。
- `P0.09` / `P0.10` は NFC ピンですが、`corcell.dtsi` の `nfct-pins-as-gpios` で
  GPIO として使える状態にしてあります。
- 電池駆動のため `sleep-on-suspend` / `wakeup-on-resume` を有効にしています。
- **向きと速度は未調整です。** 実機で確認したうえで、ドライバの
  `invert-x` / `invert-y` / `swap-xy` / `x-divisor` / `y-divisor` で合わせてください。

## ブランチの選び方

利用者向けは **`main` と `dya-studio`** です。接続ガイドのための専用ブランチは不要で、各 README から参照できます。

| ブランチ | 用途 |
| --- | --- |
| [`main`](https://github.com/yuchamichami/zmk-config-Corcell/tree/main) | 通常版。ブラウザでの設定編集を使わない方向け |
| [`dya-studio`](https://github.com/yuchamichami/zmk-config-Corcell/tree/dya-studio) | DYA Studio 対応版。ブラウザでキーマップやポインタ倍率を調整する方向け |
| [`lenotp`](https://github.com/yuchamichami/zmk-config-Corcell/tree/lenotp) | lenoTP の実験用。通常の利用者向け配布版ではありません |

## 接続・使い方

- **USB は右手側に挿します。** 右が親機で、左の入力も右から PC へ送ります。USB 使用時も左右間は Bluetooth 通信なので、左にも電源が必要です。
- 初回 Bluetooth 接続は、USB を外し、左右の電源を入れます。初期配列では右親指のレイヤー 3 キー（Space と短押し A のキーの間）を押しながら右上端の Backspace 位置を押して接続先 0 を選び、OS の Bluetooth 設定で Corcell を追加します。
- **USB の文字入力と Studio 接続は別機能です。** `main` と `lenotp` は Studio 非対応です。DYA Studio を使う場合は `dya-studio` のファームを選んでください。
- DYA 接続改善版の初期配列では、レイヤー 3 ＋ U 位置で USB 優先、＋ I 位置で Bluetooth 優先、＋右親指の A 位置でブラウザの Bluetooth 検出を開始します。通常版・旧 DYA 版・保存済みの独自配列では同じ操作が使えるとは限りません。
- `The port is already open` はまず Studio のタブを閉じ、USB を抜き差しして 1 タブだけで再接続します。全設定初期化を最初に行う必要はありません。
- UF2 は ZIP を展開し、右用を右、左用を左へ **1 ファイルずつ**コピーします。Windows の `0x80070022` だけでは書き込みの成否を判断せず、通常起動と更新版を確認してください。
- `settings_reset` は保存済み配列・ペアリングなどを消す最終手段です。通常の更新では不要です。

**詳しい接続・更新・復旧手順： [接続・使い方ガイド](https://github.com/yuchamichami/zmk-config-Corcell/blob/dya-studio/docs/connection-guide.md)**

[Discord 向け案内文](https://github.com/yuchamichami/zmk-config-Corcell/blob/dya-studio/docs/discord-connection-announcement.md) ／ [点検結果と実機確認項目](https://github.com/yuchamichami/zmk-config-Corcell/blob/dya-studio/docs/firmware-audit-2026-09-06.md)

ガイド内の Studio 操作は DYA 版向けです。`lenotp` のセンサー調整や動作確認は実験用ブランチの説明を参照してください。

## セッティングガイド

ボトムケースの開けかたと、チルトスタンドのサポート材の除去は
動画つきの別ページにまとめています。

**→ [セッティングガイド](docs/setup-guide.md)**

## ハードウェア構成

- `corcell_l` は split peripheral です。左側のモジュール入力を右側へ転送します。
- `corcell_r` は split central です。右側のモジュール入力と、左側から転送された入力を扱います。
- キー配線と kscan ピンは Corchibi と同じです。
- 6 ピン FPC スロットには、左右それぞれ 1 つずつ任意の入力モジュールを接続できます。
- デフォルトの FPC モジュールは PAW3222 トラックボールです。
- PAW3222 は `SCLK=P0.10`、`SDIO=P0.09`、`MOTION=P1.12` を使います。
- PAW3222 の NCS はデフォルトで GND 固定です。そのため、ファームウェア側では SPI chip-select GPIO を設定していません。
- PAW3222 で chip-select GPIO 制御が必要になった場合のみ、NCS を `RE_B` 側へジャンパして `&spi0` 配下に `cs-gpios = <&gpio0 5 GPIO_ACTIVE_LOW>;` を追加します。
- PAW3222 の CPI はファームウェア側で上書きせず、カーソル移動量は固定の input processor 倍率
  （`zip_xy_scaler 2 5`、スクロールは `zip_scroll_scaler 1 10`）で調整します。
- 基板上のロータリーエンコーダーは `RE_A=P0.04`、`RE_B=P0.05` で、デフォルトで有効です。
- FPC エンコーダーモジュールでは、PAW3222 の `NCS` 位置を A 相、`MOTION` 位置を B 相として使います。
- 現行回路では、エンコーダーモジュール使用時に `NCS` を `RE_B` 側へジャンパしてください。このときファームウェアは `A=P0.05`、`B=P1.12` として読みます。
- 乾電池の入力電圧は ADC0 / `P0.02` で読みます。

## FPC モジュールの切り替え

FPC モジュールは Zephyr/ZMK のスニペットで切り替えます。
通常の `build.yaml` では PAW3222 snippet だけを指定しているため、生成される UF2 の数は増えません。

- 右手 PAW3222: `corcell-right-slot1-paw3222`
- 左手 PAW3222: `corcell-left-slot1-paw3222`
- 右手エンコーダー: `corcell-right-slot1-encoder`
- 左手エンコーダー: `corcell-left-slot1-encoder`

たとえば右手スロットをエンコーダーにする場合は、`build.yaml` の
`corcell-right-slot1-paw3222` を `corcell-right-slot1-encoder` に変更します。

ユーザー目線では次の流れです。

1. 左右それぞれ、FPC スロットに取り付けるモジュールを決めます。
2. `build.yaml` の `corcell_r` と `corcell_l` に、取り付けたモジュールのスニペットを 1 つだけ指定します。
3. 変更を push します。
4. GitHub Actions の `Build` が完了したら、Artifacts から UF2 をダウンロードします。
5. `Corcell_R-...uf2` を右手、`Corcell_L-...uf2` を左手に書き込みます。

たとえば右手をエンコーダー、左手を PAW3222 にする場合は次のようにします。

```yaml
include:
  - board: xiao_ble/nrf52840/zmk
    shield: corcell_r
    artifact-name: Corcell_R-xiao_ble_zmk
    snippet: corcell-right-slot1-encoder
  - board: xiao_ble/nrf52840/zmk
    shield: corcell_l
    artifact-name: Corcell_L-xiao_ble_zmk
    snippet: corcell-left-slot1-paw3222
```

キーは `snippet:`（単数・文字列）です。`snippets:` のようにリストで書くと、
`zmkfirmware/zmk` の `build-user-config.yml` は `matrix.snippet` を空として扱い、
`west build` に `-S` が渡りません。その場合でもビルドは成功しますが、
スニペットの内容が丸ごと無視された UF2 が出力されます。

新しい FPC モジュールを増やす場合は、`snippets/` に右手用と左手用のスニペットを追加します。
`build.yaml` には実際に取り付けたモジュールのスニペットだけを書くため、モジュール候補が増えても UF2 の出力数は増えません。

## 電源設定

- ZMK sleep を有効にしています。
- BLE TX power は Corchibi の +8 dBm ではなく、0 dBm にしています。
- BLE preferred connection interval は `6-12`、latency は `0` にして、ポインタ操作の遅延を抑えています。
- PAW3222 の `force-awake` は有効にしていません。
- smooth scrolling は無効にしています。
- logging、shell、SPI shell は無効にしています。
- insomnia behavior module は含めていません。
- 通常版 `main` では DYA Studio 用の runtime input processor を含めていません。DYA Studio で保存したポインタ設定が通常版に影響しない構成です。
- 電池残量は乾電池向け voltage divider 構成で Bluetooth の battery level として報告します。
- 分圧抵抗は `output-ohms = 470k`、`full-ohms = 1M + 470k` です。
- 1 セル Ni-MH 向けの millivolt-to-percent thresholds で Bluetooth の battery level として報告します。

## 電源投入 LED

電池を入れると、XIAO の緑 LED が 2 秒だけ点灯して消えます。
組み立て時に、ペアリングしなくても電池と昇圧回路が生きているか確認できます。

- 点灯後は GPIO を切り離すので、消えたあとの消費電流はありません。
- 点灯時間は `CONFIG_CORCELL_POWER_ON_LED_MS`（既定 2000、100〜10000 ms）で変えられます。
- 不要なら `CONFIG_CORCELL_POWER_ON_LED=n` を conf に書けば丸ごと無効化できます。
- 左右どちらの半身でも点灯します。USB 給電でも同じく点灯します。

## 更新履歴

書き込みが必要な側を「対象」に書いています。記載がない項目は左右とも書き換えてください。

### 2026-09-05

- 無線接続時にカーソルがカクつく問題を修正しました。BLE の送信出力を 0 dBm から
  +8 dBm へ引き上げ、接続間隔を 15 ms 固定にしています。（対象: 左右）
- スリープから復帰したあとトラックボールが反応しなくなる問題を修正しました。
  復帰時にセンサーの電源管理が復旧しないため、電池を抜くまで復帰しませんでした。
  ドライバ側で対処しています。（対象: 右手）
- スリープ後にキーを押しても復帰しない問題を修正しました。右手側のキー読み取りに
  割り込み設定が抜けており、スリープ中にキー入力を検知できませんでした。（対象: 右手）

### 2026-09-03

- セッティングガイドを追加しました。ボトムケースの取り外し、マグネットの取り付け、
  チルトスタンドのサポート材除去を、動画つきで別ページにまとめています。

### 2026-08-31

- 電池を入れると XIAO の緑 LED が 2 秒点灯するようにしました。組み立て時に、
  ペアリングせずに電池と昇圧回路の動作を確認できます。（対象: 左右）
- トラックボールがまったく動作しない問題を修正しました。`build.yaml` の記述が
  ビルド側の想定と食い違っており、センサーの設定が丸ごと無視された UF2 が
  出力されていました。（対象: 左右）
- カーソル速度を調整し、基板上のロータリーエンコーダーを既定で有効にしました。
- 通常版から DYA Studio 用の設定を分離しました。DYA Studio 対応版は
  `dya-studio` ブランチで管理します。

## ライセンス

このリポジトリ内のファームウェアソースコード、ZMK 設定ファイル、ドキュメントは MIT License です。
詳しくは `LICENSE` を確認してください。
