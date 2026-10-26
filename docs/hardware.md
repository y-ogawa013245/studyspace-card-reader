# ハードウェア仕様 (Hardware)

本書は、`studyspace-card-reader` で使用するハードウェア構成、各コンポーネントの仕様、ピンアサイン、および配線について記述します。

---

## 1. コンポーネント一覧

| コンポーネント | 型番 / SKU | 概要 | 役割 |
| :--- | :--- | :--- | :--- |
| **マイコン** | M5Stack ATOMS3 Lite (C124) | ESP32-S3FN8 (デュアルコア 240MHz, 8MB Flash), 2.4GHz Wi-Fi, BLE | メイン制御、Wi-Fi通信、API送信 |
| **NFCユニット** | M5Stack NFC Universal Unit (U216) | STMicroelectronics ST25R3916チップ搭載, 13.56MHz RFID/NFC | FeliCa (NFC Type-F) IDm読み取り |
| **接続ケーブル** | Grove ケーブル (4ピン) | I2C通信および電源供給 | ATOMS3 Lite と NFCユニットの接続 |
| **電源** | USB Type-C ケーブル & 5V給電アダプタ | 5V / 1A 以上推奨 | 端末への給電（単体稼働用） |

---

## 2. マイコン仕様: M5Stack ATOMS3 Lite

- **SoC**: ESP32-S3FN8 (Xtensa® 32-bit LX7 デュアルコア、最大240MHz)
- **メモリ**: 384KB ROM, 512KB SRAM, 8MB Quad-SPI Flash (内蔵)
- **Wi-Fi**: 2.4 GHz Wi-Fi (802.11 b/g/n)
- **内蔵ペリフェラル**:
  - **RGB LED**: WS2812B 互換 (GPIO 35 に接続)
  - **リセットボタン / プログラマブルボタン**: GPIO 41 (本体前面プッシュスイッチ)
  - **Grove ポート**: HY2.0-4P (I2C / GPIO)
  - **電源入力**: USB Type-C (5V)

---

## 3. NFCユニット仕様: M5Stack NFC Universal Unit (U216)

- **コントローラIC**: STMicroelectronics ST25R3916
- **対応規格**:
  - ISO/IEC 14443 Type A / B
  - **ISO/IEC 18092 (NFC-IP1 / FeliCa / NFC Type-F)** ← 本プロジェクト対象
  - ISO/IEC 15693 (RFID)
- **通信インターフェース**: I2C (Grove接続)
- **デフォルトI2Cアドレス**: `0x50` (※ST25R3916チップ内部アドレス設定による、ライブラリで自動制御)
- **推奨ライブラリ**: M5Stack公式 `M5Unit-NFC`

---

## 4. ピン配置・配線仕様 (Grove / I2C)

ATOMS3 Lite の Grove ポートと NFC Universal Unit を付属の Grove ケーブルで1対1接続します。

| Pin 番号 | 線色 (Grove標準) | ATOMS3 Lite ピン | 信号名 | NFC Unit 側接続 |
| :---: | :---: | :---: | :---: | :---: |
| **1** | 黄 (Yellow) | **GPIO 2** | I2C SDA (データ) | SDA |
| **2** | 白 (White) | **GPIO 1** | I2C SCL (クロック) | SCL |
| **3** | 赤 (Red) | **5V_IN / VBUS** | 電源 5V (VCC) | 5V |
| **4** | 黒 (Black) | **GND** | グランド (GND) | GND |

> [!IMPORTANT]
> - ATOMS3 Lite の Grove コネクタは、標準で `GPIO 2 (SDA)` / `GPIO 1 (SCL)` に割り当てられています。
> - `Wire.begin(2, 1)` または M5Unified / M5Unit-NFC の初期化時に上記ピン番号が正しく指定されていることを確認してください。

---

## 5. 対象カード仕様

- **対象規格**: FeliCa (JIS X 6319-4 / NFC Forum Type 3 Tag)
- **主なカード種類**:
  - 交通系ICカード（Suica, PASMO, ICOCA, TOICA, manaca, Kitaca, SUGOCA, nimoca, はやかけん など）
  - 各種FeliCa対応学生証・社員証
  - モバイルFeliCa（Apple Pay / Google Pay の Suica・PASMO等）
- **取得データ**:
  - **IDm (製造ID)**: 8バイト (16進数表記 16文字文字列, 例: `012E456789ABCDEF`)
  - ※個人情報や履歴・残高エリア（暗号化領域および非暗号化ブロック）の読み取りは行いません。

---

## 6. 未確定事項・要検証項目

- [ ] **実機I2C通信安定性検証**: Groveケーブル長（20cm程度推奨）による波形鈍りやプルアップ抵抗の確認。
- [ ] **モバイル端末読み取り感度**: スマートフォン（Apple Pay Suica等）をかざした際の検出距離・指向性の実機テスト。
- [ ] **筐体・設置検証**: アクリル板や樹脂ケースに組み込んだ際のアンテナ通信距離（1〜3cm程度を想定）。
