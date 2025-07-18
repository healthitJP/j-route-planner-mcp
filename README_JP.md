# Japan Transfer MCP Server

**[English](README.md)** | **日本語**

[![npm version](https://badge.fury.io/js/japan-transfer-mcp.svg)](https://www.npmjs.com/package/japan-transfer-mcp)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

日本の交通機関経路検索をAIアシスタントと連携するためのModel Context Protocol（MCP）サーバーです。

## ⚠️ 重要な注意事項

**データ取得元**: このサーバーは**ジョルダン乗換案内**（[www.jorudan.co.jp](https://www.jorudan.co.jp/)）から交通情報を取得しています。

**利用制限**: このMCPサーバーをご利用の際は、[ジョルダンの利用規約](https://www.jorudan.co.jp/terms/)に従う必要があります。特に第6条「本サービスのネットワークまたはシステム等に過度な負荷をかける行為（自動処理ツール等により、短時間に大量の情報を取得しようとする行為を含みますが、自動処理ツール等の使用そのものを妨げるものではありません。）」にご注意ください。ジョルダンのサーバーに過度な負荷をかけず、現実的な使用量にとどめるようお願いいたします。

## 概要

Japan Transfer MCP Serverは、日本の交通機関経路検索サービスをClaude DesktopなどのAIアシスタントと連携するためのModel Context Protocolサーバーです。AIアシスタントが鉄道駅・バス停の検索や、日本全国の充実した交通ネットワークを活用した最適経路の計画を行えるようにします。

## 主な機能

- **駅・停留所検索**: 鉄道駅、バス停、交通施設を名前で検索
- **経路計画**: 日本国内の任意の2地点間の最適経路を検索
- **マルチモーダル交通**: 電車、バス、地下鉄、その他の交通手段に対応
- **スマート表示**: 絵文字と構造化された情報による自然言語形式の応答
- **トークン最適化**: 効率的なAPI使用のためのトークン制限による応答サイズ制御
- **包括的データ**: 時刻表、運賃、乗り換え情報を含むリアルタイム交通データ

## 対象ユーザー

### 一般ユーザー向け
Claude Desktopで日本の交通機関検索機能を使いたい方に最適です。AIアシスタントが以下をサポートします：

- 日本全国の鉄道駅・バス停の検索
- 旅行のための最適経路の計画
- リアルタイムの交通情報取得
- 複雑な日本の交通システムの理解

### AIアシスタント利用者向け
このサーバーによりAIアシスタントは以下が可能になります：

- 名前による交通施設の検索
- 詳細情報付きの地点間経路計画

## 技術仕様

### 利用可能なツール

#### 1. `search_station_by_name`
**説明**: 駅名・交通施設名による検索

**パラメータ**:
- `query` (必須, string): 検索する駅名（日本語である必要があります）
- `maxTokens` (オプション, number): 返すトークンの最大数
- `onlyName` (オプション, boolean): 駅名のみを返すかどうか。詳細情報が不要な場合は、通常trueに設定することを推奨します。

**戻り値**: 位置情報、読み、行政コードを含む詳細情報付きの該当駅リスト

#### 2. `search_route_by_name`
**説明**: 駅名による2地点間の最適経路検索

**パラメータ**:
- `from` (必須, string): 出発駅名（日本語）
- `to` (必須, string): 到着駅名（日本語）
- `datetime` (任意, string): ISO-8601形式の日時（YYYY-MM-DD HH:MM:SS）。入力しない場合は現在の日本の時刻が設定される。
- `datetime_type` (必須, string): 時刻指定タイプ:
  - `departure`: 出発時刻で検索
  - `arrival`: 到着時刻で検索
  - `first`: 始発で検索
  - `last`: 終電で検索
- `maxTokens` (オプション, number): 応答の最大トークン数

**戻り値**: 時刻表、運賃、乗り換え、所要時間を含む詳細経路情報


## セットアップ

### Claude Desktop設定

`claude_desktop_config.json`ファイルに以下の設定を追加してください：

**macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
**Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

#### 基本設定

```json
{
  "mcpServers": {
    "japan-transfer-mcp": {
      "command": "npx",
      "args": ["japan-transfer-mcp"]
    }
  }
}
```

### オンラインテスト

ローカルセットアップなしでMCPサーバーの機能をオンラインでテストできます：

- **SSE形式**: https://japan-transfer-mcp-server.ushida-yosei.workers.dev/sse
- **Streamable HTTP形式**: https://japan-transfer-mcp-server.ushida-yosei.workers.dev/mcp

ローカルセットアップの前に、[@https://playground.ai.cloudflare.com/](https://playground.ai.cloudflare.com/) でこれらのエンドポイントを試してサーバー機能をテストしてください。

## 使用例

### 駅検索
```
「東京駅」を検索 → 座標、読み、行政コードを含む東京駅の詳細情報を返します。
```

### 経路計画
```
明日の午前9時に東京駅から大阪駅への経路を計画 → 時刻表、運賃、乗り換え情報を含む複数の経路オプションを返します。
```

## トラブルシューティング

### よくある問題

#### 1. 駅が見つからない
**症状**: `search_station_by_name`が空の結果を返す
**解決方法**:
- 駅名が日本語であることを確認
- 別の表記や読みを試す
- より広い検索結果を得るため部分的な名前を使用

#### 2. 経路検索が失敗する
**症状**: `search_route_by_name`でエラーが発生
**解決方法**:
- 出発駅と到着駅の両方が存在することを確認
- 日時の形式を確認（YYYY-MM-DDTHH:MM）
- datetime_typeが有効なオプションの一つであることを確認

#### 3. トークン制限を超過
**症状**: 応答が切り詰められる
**解決方法**:
- maxTokensパラメータを削減
- 詳細情報が不要な場合は駅検索でonlyName=trueを使用
- 複雑なクエリをより小さな部分に分割

#### 4. サーバーの起動が失敗
**症状**: MCP サーバーの初期化が失敗
**解決方法**:
- Node.js バージョンの互換性を確認
- ネットワーク接続を確認
- Claude Desktop の設定が正しいことを確認

## 応答形式

サーバーは以下の豊富な形式で応答を返します：
- 🚃 交通手段のアイコン
- 📅 日付と時刻の情報
- 💰 運賃の詳細
- ⏱️ 所要時間と時刻表情報
- 🔄 乗り換え情報
- 🌱 環境への影響データ
- ⚠️ 運行情報とアラート

## 制限事項

- 日本の交通ネットワークのみサポート
- 駅名検索には日本語入力が必要
- 一部の地方や地域交通サービスは対象外の場合があります

## ライブラリとしての使用

このMCPサーバーは他のNode.jsプロジェクトでライブラリとしても使用できます。モジュールとしてインポートされた場合、自動的にstdio transportに接続されず、プログラマティックに使用することができます。

### インストール

```bash
npm install japan-transfer-mcp
```

### 使用方法

```javascript
import server from 'japan-transfer-mcp';

// サーバーインスタンスをプログラマティックに使用
// ライブラリとしてインポートされた場合は自動接続されません
// サーバーのツールやハンドラーに直接アクセスできます
```

## 依存関係

主な依存関係：
- `@modelcontextprotocol/sdk`: MCP サーバー開発キット
- `axios`: API リクエスト用 HTTP クライアント
- `cheerio`: ウェブスクレイピング用 HTML パーサー
- `gpt-tokenizer`: 応答最適化のためのトークン計算
- `zod`: スキーマ検証

## ライセンス

MIT License

## 貢献

プルリクエストやイシューの報告を歓迎します。以下を確認してください：
1. コードがTypeScriptのベストプラクティスに従っている
2. 新機能のテストが通る
3. ドキュメントが適切に更新されている

## サポート

問題が発生した場合は、以下を確認してください：
1. Claude Desktop の設定
2. ネットワーク接続
3. 入力形式の検証
4. トークン制限と応答サイズ

追加のサポートが必要な場合は、GitHub リポジトリにイシューを作成してください。

## 謝辞

このプロジェクトは日本の交通データプロバイダーと連携して、包括的な経路計画機能を提供しています。公共データアクセスを提供する交通事業者の皆様に特別な感謝を申し上げます。
