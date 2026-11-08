# JIS-Deskflow

日本語JISキーボードのかな入力と、PC間のクリップボード共有を改善したDeskflowの派生版です。**Junichi Ogura** が改変・保守しています。Deskflow 1.27.0を基にした独立した配布で、本家の公式リリースではありません。

**[Windows版](https://github.com/jogu6/deskflow-jis-updates/releases/latest/download/JIS-Deskflow-Windows-x64-Setup.exe) ／ [Linux版](https://github.com/jogu6/deskflow-jis-updates/releases/latest/download/JIS-Deskflow-Linux-x86_64.run)**

| 配布物 | 用途 |
| --- | --- |
| `JIS-Deskflow-Windows-x64-Setup.exe` | Windows 11用インストーラー（最新版への固定リンク） |
| `JIS-Deskflow-Linux-x86_64.run` | Xubuntu / UbuntuのX11環境用インストーラー（最新版への固定リンク） |
| `JIS-Deskflow-1.27.2-Linux-x86_64.flatpak` | 同じLinux版のFlatpak本体。単体導入は常駐登録を行いません |
| `JIS-Deskflow-1.27.2-source.zip` | 対応ソース・ビルド手順 |
| `SHA256SUMS` | 配布ファイルのSHA256 |

ダウンロードにGitHubへのログインは不要です。

## できること

- 1組のキーボード・マウスで複数PCを操作し、画面端で切り替える。
- X11からWindowsへ送るJISかな入力で、キー位置が失われる問題を改善。
- テキスト、書式付きテキスト、矩形キャプチャを含む画像を共有。
- 複数ファイルをコピーし、相手のファイラーで貼り付けて複製。
- ファイルはコピー時にパス参照だけを保持し、貼り付け時に既存TLS接続で分割転送。ファイル全体の先読みや専用ディスクキャッシュを作らない。
- 時間のかかるファイル転送に進捗・速度・残り時間・キャンセルを表示。
- Windowsサービスとして動作し、サインイン画面での入力にも対応。

Linuxサーバーでは古いクリップボード容量設定を転送上限として使いません。画像の寸法・通信形式・サイズ計算・メモリ余裕を検査し、処理できない転送は中止します。旧3MiB設定を残した4K画像の双方向TLS転送を検証しています。PC全体のメモリ不足まで完全に防ぐ保証ではありません。

## 使い方

### 初回設定

1. 両PCへJIS-Deskflowを導入します。ファイル共有とJIS拡張は両端が対応版である必要があります。
2. Linuxは `.run` を実行し、メニューからJIS-Deskflowを起動します。キーボード側をサーバーにし、接続するPC名と画面配置を設定します。ユーザー用Flatpakとして導入し、ランタイムの取得にネット接続が必要な場合があります。
3. WindowsはSetup.exeを実行します。管理者承認が必要です。スタートメニューの **JIS-Deskflow → 接続設定（管理者）** を開き、サーバーのIPアドレスとPC名を設定してください。初期状態の接続先は空です。
4. TLS証明書のフィンガープリントを両側で確認し、接続先を信頼して接続します。IMEやキーボード配列は各OSで設定済みのものを使用します。

Linuxで `.run` に実行許可を付けて導入する場合は、ダウンロードしたフォルダーで次を実行します。

```sh
chmod +x JIS-Deskflow-Linux-x86_64.run
./JIS-Deskflow-Linux-x86_64.run
```

Windowsはサービスが自動起動する構成です。通常のGUIを閉じても共有サービスは継続します。トレイアイコンはGUI起動時に表示されます。設定は保護されているため、変更には上記の管理者用ショートカットを使います。

Linuxの `.run` は、ユーザー用systemdサービスとログイン時の自動起動を登録します。異常終了時は3秒後に再起動します。既に動作中のアプリを停止せず、次回ログインからサービス管理になります。sudoは不要です。

### コピーと貼り付け

- テキストや画像は、通常どおりコピーして相手PCの対応アプリへ貼り付けます。
- ファイルはエクスプローラー／Thunarで選択して `Ctrl+C`、相手のフォルダーで `Ctrl+V`。
- 双方向の複数ファイルに対応します。元のファイルは削除しません。
- コピー後に元ファイルを削除・移動すると貼り付けできません。キャンセル後は再コピーしてやり直してください。

## 確認した範囲と制約

| 組み合わせ | 確認状況 |
| --- | --- |
| Xubuntu 24.04 / X11 ⇄ Windows 11 | テキスト・矩形画像・複数ファイルの実機成功を確認。公開版の表記・更新先変更前の同機能版での確認を含む |
| X11 ⇄ X11 | 独立した仮想画面と実TLSで、テキスト・HTML・4K画像・複数ファイルを双方向検証 |
| Windows ⇄ Windows | 対応設計。個別機能のCIテストあり。Windows 2台を接続した通し試験は未実施 |
| Wayland / macOS | 今回の配布対象外 |

通常ファイルは一度に最大1024件です。フォルダー一式の再帰コピー、切断後の途中再開には対応していません。Windowsで使用できないファイル名も対象外です。貼り付け先アプリの対応形式によって結果は異なります。

Linux43スイート・1149ケース、Windows41スイート・1156ケースを実行し、失敗0件です（Windowsの日本語実配列限定1件はCIでスキップ）。Windowsのサービス起動・停止・削除、初期接続先が空である新規導入、自動起動の登録も使い捨てCIで検証しています。最新版の再起動後の実機試験、全機種・全IMEの動作保証はしていません。

## 更新と削除

起動時の更新確認を有効にすると、この派生版の最新版を確認します。アプリの更新ボタンから、使用OSに対応したインストーラーを直接取得できます。Assets一覧からファイルを探す必要はありません。適用は利用者の操作で開始し、接続設定を引き継ぎます。使用中に勝手に停止・更新する処理はありません。

Windowsは「インストールされているアプリ」のJIS-Deskflow、または同梱Uninstallから削除できます。Linuxは先に `python3 ~/.local/share/jis-deskflow/Manage-Linux-Service.py remove` で常駐登録を削除し、`flatpak uninstall --user --delete-data org.deskflow.DeskflowJIS` で専用アプリとデータを削除します。

Windowsの配布EXEはコード署名なしです。配布元とSHA256を確認してください。IME・配列・OSの入力保護設定を変更する回避策は含めません。

## ライセンスとソース

GPL-2.0-only WITH LicenseRef-OpenSSL-Exceptionです。上流の著作権・ライセンスを保持しています。[LICENSE](LICENSE)、[LICENSE_EXCEPTION](LICENSE_EXCEPTION)、[配布と利用について](TERMS-JIS.txt)を参照してください。依存ライブラリには各ライセンスが適用されます。

各Releaseにインストーラーに対応するソースZIPを添付します。ソースとビルド手順は誰でも取得できます。利用者の個人設定・接続用の証明書・秘密鍵・実機の検証ログは公開アセットへ含めません。

同梱ライブラリのライセンスと、Qt・OpenSSL・libei・libportalの対応ソースもReleaseに添付しています。詳しくは [第三者ソース一覧](https://github.com/jogu6/deskflow-jis-updates/releases/latest/download/THIRD-PARTY-SOURCES.txt) を参照してください。
