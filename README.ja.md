# Typeless AutoSend（フォーク版・日本語説明）

これは [llj15/typeless-autosend](https://github.com/llj15/typeless-autosend) のフォークです。
上流の使い方・仕様は英語の [README.md](README.md) が正本で、この文書はフォーク側の差分と運用方針だけを説明します。

## このフォークの目的

「デジタル地下室」（@dgtlchika）の運用環境で、音声入力アプリ Typeless の自動送信を **Muse アプリ** で安定して使うために維持しているフォークです。
上流の更新は取り込み続け、ローカルの適応はこのフォーク側のブランチで管理します。

- フォーク: https://github.com/dgtlchika/typeless-autosend
- ローカル適応ブランチ: `kohaku-patches`
- 上流: https://github.com/llj15/typeless-autosend

## 上流との差分（パッチ2件、いずれも `InputMonitor.swift` のみ）

### パッチ1: キャレット確認のスキップ＋AXタイムアウトの延長

- Accessibility（AX）経由でキャレット／選択範囲を読めないアプリ（Electron 系など）では、上流は「検査できない」として自動送信をスキップしていました。フォークでは選択範囲を「不明（`selectionKnown = false`）」として扱い、対象アプリ・ウィンドウ・入力欄の同一性チェックは維持したまま送信を進めます。
- あわせて、AX のメッセージングタイムアウトを 0.08 秒から **0.3 秒（300ms）** に延長しました（定数 `axInspectionTimeout`）。WebKit 系アプリは応答が遅いことがあり、短すぎるタイムアウトが検査失敗の原因になっていたためです。

### パッチ2: Muse アプリ向けフォーカス確認の緩和

- 対象を `relaxedFocusBundleIDs`（現在は `com.meta.endo` のみ）に限定し、そのアプリではフォーカスの同一性を「同じアプリ（Bundle ID＋プロセスID）かつ同じウィンドウ」で判定します。
- アプリ内で入力欄が移動したり、選択範囲が読めなかったりしてもキャンセルしません。**ウィンドウの変更やアプリの切り替えでは、従来どおりキャンセルされます。**
- Muse アプリはアプリ内フォーカス移動が多く、上流の厳密な入力欄単位の確認では正当な送信までキャンセルされてしまうための適応です。

## ビルドとインストール

手順は上流と同一です（詳細は [README.md](README.md) の "Build and install from source"）。Xcode Command Line Tools が必要で、リポジトリは `~/Developer` など保護されていないディレクトリに置いてください。

```bash
git clone -b kohaku-patches https://github.com/dgtlchika/typeless-autosend.git ~/Developer/typeless-autosend
cd ~/Developer/typeless-autosend
./install.sh
```

## 運用上の注意

- **対象アプリは Muse アプリ（`com.meta.endo`）のみを有効化して運用しています。** 有効化は Bundle ID で行います:

  ```bash
  typeless-autosend apps add com.meta.endo
  typeless-autosend apps list
  ```

- 再ビルドすると macOS のアクセシビリティ許可が無効になることがあります。再インストール後は `typeless-autosend doctor` で確認し、必要なら「システム設定 → プライバシーとセキュリティ → アクセシビリティ」で現在の実行ファイルを追加し直してください。
- 個人用の設定ファイル `config.json` やログはリポジトリに含めません（上流の `.gitignore` どおり）。

## ライセンスについて

上流リポジトリにはライセンスファイルが同梱されていません。本フォークは上流プロジェクトの派生物であり、利用にあたっては上流の条件に従ってください。
