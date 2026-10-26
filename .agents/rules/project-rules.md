# Antigravity Project Rules — StudySpace Card Reader

本ファイルは、Google Antigravity の Workspace Customization Root (`.agents/rules/`) に配置されたプロジェクトルールです。
リポジトリルートの [AGENTS.md](../../AGENTS.md) に定義された共通開発ルールを継承・適用します。

## コアルール要約

1. **プロジェクト目的**:
   - M5Stack ATOMS3 Lite + NFC Universal Unit (U216) によるスタンドアロンICカードリーダー。
   - FeliCa IDm（8バイト）のみを取得し、Wi-Fi経由で自作REST APIへHTTPS POST送信。
2. **実装・設計方針**:
   - 責務分離（NFC、Wi-Fi、API通信、UI/LED、AppController）。
   - `main.cpp` への処理集中を避け、Nativeテストが可能なインターフェース設計とする。
   - 同一カードの連続送信（チャタリング）防止を実装。
   - 秘密情報（Wi-Fi、APIトークン）のハードコード禁止、TLS証明書検証の維持。
3. **テスト方針**:
   - 「実機がない状態で、ハードウェア動作を確認済みとして扱わない」。
   - PC上のPlatformIO Native単体テスト・モックテストを活用。
4. **Git・運用**:
   - `push`・PR作成・マージはユーザーの明示的な許可を得てから実行する。
   - ドキュメントの変更時は `docs/` 配下および `CHANGELOG.md` を同期更新する。

※詳細な開発規約・ハードウェア仕様・API仕様は [AGENTS.md](../../AGENTS.md) および [docs/](../../docs/) を参照してください。
