# テスト方針・検証手順 (Testing)

本書は、`studyspace-card-reader` のテスト戦略、単体テスト、モックテスト、実機検証、およびCI環境について記述します。

---

## 1. テスト方針・基本原則

1. **テスト可能な設計の徹底**:
   - ハードウェア（I2C, NFCチップ, Wi-Fiモジュール, HTTP通信）をインターフェース経由で抽象化し、ロジック部（状態遷移、重複検知、JSON組み立て）をPC上の単体テスト（Native環境）で検証可能にします。
2. **実機未接続時の推測禁止**:
   - **「実機がない状態で、ハードウェア動作を確認済みとして扱わない」** 原則を徹底します。モックによる検証は「ロジックの正しさ」のみを保証し、NFC検知感度やI2C通信安定性、Wi-Fi通信速度は実機接続による検証フェーズで確認します。
3. **継続的インテグレーション (CI)**:
   - コミットやプルリクエスト時に GitHub Actions で自動ビルドおよびNative単体テストを実行し、リグレッションを早期発見します。

---

## 2. テストの分類

```
[テストピラミッド]
       /  実機検証 (Manual / HIL)  \      <- NFC実カードタッチ、実API送信
      /----------------------------\
     /    統合・モックテスト (Host)    \   <- NFCモック、HTTPクライアントモック
    /--------------------------------\
   /       単体テスト (Unit Test)      \  <- カードイベント判定、クールダウンタイマー
```

| レベル | 対象環境 | テスト内容 | 実行コマンド例 |
| :--- | :--- | :--- | :--- |
| **単体テスト** | Host (Native / x86_64) | ロジック、文字列処理、JSONシリアライズ、クールダウンタイマー | `pio test -e native` |
| **モックテスト** | Host (Native) | モックインターフェースを用いた `AppController` の状態遷移検証 | `pio test -e native` |
| **ビルド検証** | ESP32-S3 (Cross-compile) | ファームウェア全体のコンパイルエラーチェック | `pio run` |
| **実機検証** | ATOMS3 Lite + NFC Unit | FeliCaカード実読み取り、LED表示、Wi-Fi/API実通信 | 実機書き込み (`pio run -t upload`) |

---

## 3. PlatformIO によるテスト実行

### 3.1 Native 単体テスト（PCローカル実行）
ESP32実機を接続せず、開発PC上でテストを実行します（`platformio.ini` に `[env:native]` を定義）。

```bash
# Native環境でのテスト実行
pio test -e native
```

### テスト実装方針
- `test/` ディレクトリ配下にUnityテストフレームワークを用いたテストコードを配置。
- ハードウェア通信クラス（`NfcReader`, `ApiClient`）のモック（ダミークラス）を作成し、`AppController` に注入して挙動をテスト。

```cpp
// モックテストの例（イメージ）
void test_cooldown_prevents_duplicate_card() {
    MockNfcReader mockNfc;
    MockApiClient mockApi;
    AppController app(&mockNfc, &mockApi);

    // 1回目の読み取り -> API送信が1回呼ばれる
    mockNfc.simulateScan("012E456789ABCDEF");
    app.update();
    TEST_ASSERT_EQUAL(1, mockApi.getSendCount());

    // クールダウン期間内の再読み取り -> 送信されない
    mockNfc.simulateScan("012E456789ABCDEF");
    app.update();
    TEST_ASSERT_EQUAL(1, mockApi.getSendCount());
}
```

---

## 4. 実機検証チェックリスト

実機検証時は以下の項目を段階的に確認します。

- [ ] **電源・起動確認**:
  - USB給電でATOMS3 Liteが正常起動し、内蔵LEDが初期化ステータスを示すか。
- [ ] **I2C / NFCユニット認識**:
  - NFC Universal Unit (ST25R3916) がI2Cバス上で認識され、初期化に成功するか。
- [ ] **Wi-Fi接続確認**:
  - 指定のアクセスポイントに接続し、IPアドレスを取得できるか。切断時に自動復帰するか。
- [ ] **カード読み取りテスト**:
  - 交通系ICカード（Suica / PASMO等）をかざした際に、正確な8バイトIDmが取得できるか。
  - スマートフォン（Apple Pay Suica等）でも安定して読み取れるか。
- [ ] **チャタリング・連続読取防止テスト**:
  - カードを端末に置いたままにした場合、複数回連続でAPI送信されないか。
- [ ] **API送信・TLS検証**:
  - 自作REST APIへHTTPS通信が通り、HTTP 200/201が返却されるか。
  - 証明書検証エラーやネットワークエラー時のLED表示（赤点滅等）が正しく動作するか。

---

## 5. CI (GitHub Actions) 方針

GitHub Actions のワークフロー（`.github/workflows/ci.yml`）を導入予定です。

1. **ファームウェアビルドチェック**: `pio run` によるビルド成功の確認。
2. **Native単体テスト実行**: `pio test -e native` によるロジックテストのパス確認。

---

## 6. 未確定事項・要検証項目

- [ ] **Unity / Native環境のビルド設定**: Windows環境におけるGCC/Clang環境の整備およびPlatformIO `native` 環境の互換性検証。
- [ ] **実機テスト用治具・手順**: 自動テスト化が困難なNFCタッチの実機受入検証フローの策定。
