# 開発環境構築・セットアップ (Setup)

本書は、`studyspace-card-reader` の開発環境構築、ビルド、初期設定、および実機書き込み手順を解説します。

---

## 1. 前提ツール・動作要件

本プロジェクトは無料・オープンソースのツールをベースに構成されています。

| ツール | 推奨バージョン / 概要 | 役割 |
| :--- | :--- | :--- |
| **IDE** | Google Antigravity / VS Code | 開発統合環境・AI協調開発 |
| **ビルド基盤** | PlatformIO (Core 6.x 以上) | コンパイル・ライブラリ管理・ファームウェア書込 |
| **実行環境** | Python 3.10 以上 | PlatformIO Core の動作要件 |
| **Git** | Git 2.x 以上 | バージョン管理 |
| **USBドライバ** | CH9102 / CP210x ドライバ | ATOMS3 Lite とのUSBシリアル通信（OSによる） |

---

## 2. 開発環境のセットアップ

### 2.1 リポジトリのクローン
```bash
git clone https://github.com/y-ogawa013245/studyspace-card-reader.git
cd studyspace-card-reader
```

### 2.2 PlatformIO のインストール
PlatformIO Core (CLI) をインストールしていない場合、pip またはインストーラーを用いてセットアップします。

```bash
# pipでインストールする場合
pip install -U platformio

# インストール確認
pio --version
```

---

## 3. 初期設定（Wi-Fi・APIトークン）

認証情報（Wi-Fi SSID/パスワード、APIトークン）はソースコード本体にコミットしません。
設定ファイル用テンプレート（例: `include/config.example.h`）からコピーして個別設定を作成します。

```bash
# 設定テンプレートをコピー（※将来の実装に合わせて実施）
cp include/config.example.h include/config.h
```

### 設定項目例 (`include/config.h`)
```cpp
#pragma once

// Wi-Fi 接続設定
#define WIFI_SSID     "YOUR_WIFI_SSID"
#define WIFI_PASSWORD "YOUR_WIFI_PASSWORD"

// API サーバー設定
#define API_BASE_URL  "https://api.studyspace.example.com"
#define API_TOKEN     "YOUR_DEVICE_SECRET_TOKEN"
#define DEVICE_ID     "reader-atom-01"
```

> [!CAUTION]
> `include/config.h` は `.gitignore` に登録し、絶対にGitリポジトリへコミット・プッシュしないでください。

---

## 4. ビルドと実機書き込み

### 4.1 ビルド確認
```bash
# PlatformIO を用いたファームウェアビルド
pio run
```

### 4.2 実機書き込み (Flash)
ATOMS3 Lite を PC の USB Type-C ポートに接続し、以下のコマンドで書き込みを行います。

```bash
# 書き込み
pio run --target upload

# シリアルモニターの起動（ボーレート 115200bps）
pio device monitor -b 115200
```

> [!TIP]
> ATOMS3 Lite（ESP32-S3）への書き込みに失敗する場合（ポート認識エラー等）、本体前面ボタンを押しながらPCにUSB接続して「ダウンロードモード」に入れてから再度 `pio run -t upload` を実行してください。

---

## 5. 未確定事項・要検証項目

- [ ] **PlatformIO依存設定 (`platformio.ini`) の最終構成**: `m5stack-atoms3` ボード定義の最適設定およびライブラリバージョンの固定。
- [ ] **設定管理手法の拡張**: ヘッダーファイル方式に加え、USBシリアルコンソール経由の設定やWebポータル（WiFiManager）方式の導入要否。
