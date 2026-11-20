# moNa2 v2 (DYA Studio 対応版)

このリポジトリは [moNa2 v2](https://github.com/sayu-hub/zmk-config-moNa2) 用 ZMK Config を、[cormoran](https://github.com/cormoran) さんが公開している **[DYA Studio](https://studio.dya.cormoran.works/)** に対応させたものです。

DYA Studio は ZMK Studio をベースに、

- キーマップ／レイヤーの GUI 編集
- トラックボール (PAW3222) のスクロール量・軸反転・自動マウスレイヤーなど **Runtime Input Processor 経由での動的設定**
- バッテリ履歴の閲覧
- BLE プロファイル管理 / Settings RPC

を Web から行えるようにした拡張版です。

> [!NOTE]
> このフォーク（hirunefu/zmk-config-moNa2-v2）は、右手のトラックボールを **PAW3222 専用**にしています。PMW3610 用のビルドはありません。右手には `mona2_r-paw3222`、左手には `mona2_l` で始まるファイルを書き込みます。

## キーマップ

![moNa2 keymap](keymap-drawer/mona2.svg)

[keymap-drawer YAML](keymap-drawer/mona2.yaml) / [SVGを開く](keymap-drawer/mona2.svg)

このフォークのキーマップは roBa（[hirunefu/zmk-config-roBa](https://github.com/hirunefu/zmk-config-roBa)）に合わせています。レイヤーは `0 BASE`〜`7 OS` の 8 枚で、トラックボールの動き（オートマウスでレイヤー 4、LANG1 長押しでスクロール、SPACE 長押しで 1/3 の速さ）も roBa に合わせています。roBa との違いは次のとおりです。

- moNa2 は 42 キーで、roBa の左手内側 2 段目のキー（ESC）の位置にノブがあります。ESC は LANG1 長押し＋内側の TAB キーに移し、NUM レイヤーの同じ位置にあった Ctrl+Alt+0 は外しました。
- ノブは、ふだんは画面の拡大・縮小（Ctrl＋ホイール）、LANG1 長押し中はタブ切り替えです。割り当ては DYA Studio でレイヤーごとに変えられます。
- roBa にある慣性スクロール・ポインタ加速・OS 別の既定レイヤー・実行時マクロは入れていません（ZMK とモジュールの版が古く、使えないものもあります）。

## DYA Studio の使い方

1. 本リポジトリをビルドし、生成された `mona2_r-...uf2`（中央側＝右手）と `mona2_l-...uf2`（周辺側＝左手）をそれぞれの XIAO BLE に書き込みます。
2. 中央側（右手）を **USB ケーブル** で PC に接続します。
3. Chrome / Edge などの WebUSB 対応ブラウザで **[https://studio.dya.cormoran.works/](https://studio.dya.cormoran.works/)** を開きます。
4. LANG2 を長押しして `6 SYSTEM` レイヤーに入り、H の位置の **`&studio_unlock`** キーを押してアンロック。
5. DYA Studio 側で「Connect」を押し、USB デバイスとして mona2 を選択。
6. キーマップ／トラックボール設定／バッテリ履歴を編集できます。

### このリポジトリで対応済みの内容

| 対応項目 | ファイル / 箇所 |
| --- | --- |
| ZMK 本体を cormoran さんの fork (`v0.3-branch+dya`) に切替 | `config/west.yml` |
| DYA Studio モジュール 4 本を west.yml に追加 (ble-management / battery-history / settings-rpc / runtime-input-processor) | `config/west.yml` |
| Studio 系 CONFIG (`CONFIG_ZMK_STUDIO`, `CONFIG_ZMK_BLE_MANAGEMENT`, `CONFIG_ZMK_BATTERY_HISTORY`, `CONFIG_ZMK_SETTINGS_RPC`, `CONFIG_ZMK_RUNTIME_INPUT_PROCESSOR`, `CONFIG_ZMK_RUNTIME_INPUT_PROCESSOR_STUDIO_RPC`) | `config/mona2_r.conf` |
| トラックボール処理を Runtime Input Processor に置換 | `boards/shields/mona2/mona2.dtsi`, `mona2_r.overlay` |
| キーマップを roBa に合わせる（`&studio_unlock` は `6 SYSTEM` レイヤーの H の位置、レイヤー名は `0 BASE`〜`7 OS`） | `config/mona2.keymap` |
| オートマウス・LANG1 長押しのスクロール・SPACE 長押しの低速を roBa に合わせる | `boards/shields/mona2/mona2_r.overlay` |
| `studio-rpc-usb-uart` snippet (中央側) | `build.yaml` |

> **Note:**
> - DYA Studio で行った変更は **中央側 (右手) の Flash** に保存されます。初期化したい場合は `settings_reset` ファームウェアを書き込んでください。
> - **`v0.3-branch+dya` / 各 DYA module の revision・CONFIG 名は調査ベースの値**です。ビルドが通らない場合は、おぐ さんの記事 [DYA Studioを導入してみるぞ！moNa2編](https://note.com/heace/n/nf06b797ffa79) を参照して、各 revision/CONFIG 名を最新版に合わせてください。

---

## トラックボールの向き（PAW3222）

PAW3222 のドライバには、軸の反転・入れ替えの設定がありません。向きが合わないときは、DYA Studio のトラックボール設定で X/Y の反転・入れ替え・回転を変えるか、`boards/shields/mona2/mona2_r.overlay` の `&mouse_runtime_input_processor` に初期値として `x-invert;`、`y-invert;`、`xy-swap-enabled;`、`rotation-degrees = <角度>;` を書いてください。CPI を変える場合は、`trackball_central` に `res-cpi = <値>;`（608〜4826、38 刻み）を書きます。
