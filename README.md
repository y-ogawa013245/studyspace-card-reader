# StudySpace Card Reader

自習室「StudySpace」の入退室管理システムで使用する、単体稼働型のICカードリーダー端末です。
M5Stack ATOMS3 Lite と NFC Universal Unit を使用し、FeliCaカード（交通系ICカード等）から取得したIDmをWi-Fi経由で自作REST APIへHTTPS POST送信します。
PCへの常時接続を必要とせず、電源供給のみで自律稼働します。

## セットアップ

開発環境の構築および依存ライブラリのインストールには PlatformIO を使用します。
詳細な手順や設定方法については [セットアップガイド](docs/setup.md) を参照してください。

```bash
# リポジトリのクローン
git clone https://github.com/y-ogawa013245/studyspace-card-reader.git
cd studyspace-card-reader

# ビルドの実行
pio run
```

## 使い方

端末をPCにUSB接続してファームウェアを書き込み、シリアルモニターで動作ログを確認します。
テスト実行や実機検証の詳細は [テスト・検証手順](docs/testing.md) を参照してください。

```bash
# ファームウェアの書き込み
pio run --target upload

# シリアルモニターの起動
pio device monitor -b 115200
```

## 関連資料

本プロジェクトの詳細仕様および開発規約は、以下のドキュメントを参照してください。

- [要件定義書](docs/requirements.md) : 機能要件・非機能要件・制約事項
- [アーキテクチャ設計書](docs/architecture.md) : モジュール構成・責務分離・処理シーケンス
- [ハードウェア仕様書](docs/hardware.md) : ピンアサイン・配線仕様・対応ICカード
- [API送信仕様書](docs/api-spec.md) : HTTPS POST仕様・ペイロード形式・エラー処理
- [セットアップガイド](docs/setup.md) : 開発環境構築・設定値の管理・書き込み手順
- [テスト・検証手順](docs/testing.md) : Native単体テスト・モック検証・実機テスト項目
- [変更履歴](CHANGELOG.md) : 主な更新履歴 (Keep a Changelog)
- [AI協調開発ルール](AGENTS.md) : AIエージェント向けの開発ガイドライン
