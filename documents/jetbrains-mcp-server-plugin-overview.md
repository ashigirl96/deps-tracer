# JetBrains MCP Server Plugin の概要

## プロジェクト概要

JetBrains MCP (Model Context Protocol) Server Pluginは、大規模言語モデル（LLM）とJetBrains IDEの間でシームレスな統合を可能にするプラグインです。このプラグインは、MCPリクエストを処理するためのサーバーサイド実装を提供し、カスタムツールを実装するための拡張ポイントを公開しています。

## 主要機能

1. **MCPプロトコルサポート**: JetBrains IDEとLLM（Claude等）間の通信を可能にします。
2. **組み込みツール**: 多数の便利なIDE操作ツールが提供されています：
   - ファイル操作（読み取り、書き込み、検索）
   - コード編集（選択テキストの置換、ファイル全体の置換）
   - デバッガー操作（ブレークポイントの設定・取得）
   - 実行構成の管理
   - プロジェクト構造・依存関係の取得
   - ターミナル操作
   - Git統合
   - その他多数のIDE機能

3. **拡張ポイントシステム**: サードパーティプラグインが独自のMCPツールを実装できるようにする拡張ポイントを提供。

## アーキテクチャ

1. **MCPService**: HTTPリクエストハンドラーとして実装され、MCPリクエストを処理し、適切なツールに転送します。
2. **McpTool インターフェース**: すべてのツールが実装する必要があるインターフェース。
3. **AbstractMcpTool クラス**: カスタムツール実装のための抽象基底クラス。
4. **McpToolManager**: 組み込みツールと拡張ポイントで提供されるツールを管理。

## 実装詳細

- **言語**: Kotlin
- **ビルドシステム**: Gradle (Kotlin DSL)
- **ターゲットIDE**: JetBrains IDE（IntelliJ IDEA、WebStorm等）
- **互換性**: IDE build 242〜251.*
- **依存関係**: 
  - org.jetbrains.plugins.terminal (オプション)
  - Git4Idea (オプション)

## カスタムツール開発

カスタムツールを開発するには以下のステップが必要です：

1. **AbstractMcpTool を拡張**: ツールのロジックを実装するKotlinクラスを作成
2. **入力パラメータのデータクラスを定義**: ツールに渡される引数の構造を定義
3. **plugin.xml で拡張を登録**: mcpToolタイプの拡張を登録

## 主要コンポーネント

1. **MCPService**: クライアントからのHTTPリクエストを処理し、ツールを実行
2. **McpToolManager**: 利用可能なすべてのツールを管理
3. **MCPServerStartupValidator**: プラグイン起動時の検証を実行
4. **組み込みツール群**: ファイル操作、デバッグ、実行、検索など様々な機能を提供するツール群
5. **拡張ポイント**: サードパーティプラグインがカスタムツールを提供するための仕組み

## 利用要件

- JetBrains MCPプロキシのインストール（[GitHub: mcpProxy](https://github.com/JetBrains/mcpProxy)）
- JetBrains IDE（IntelliJ IDEA、WebStormなど）

## ライセンス

Apache License 2.0

## MCP経由での「Go to Definition」の実行について

MCP経由で「Go to Definition」機能を実行することは可能です。この機能を使用するには以下の手順を踏む必要があります：

1. **アクションIDの特定**：
   - IntelliJ IDEで「Go to Definition」機能は通常、「GotoDeclaration」または「GotoImplementation」などのアクションIDを持っています。
   - `list_available_actions`ツールを使用して、利用可能なすべてのアクションとそのIDのリストを取得できます。

2. **アクションの実行**：
   - 適切なアクションIDを特定したら、`execute_action_by_id`ツールを使用してアクションを実行できます。
   - このツールは`actionId`パラメータを受け取り、指定されたアクションを実行します。

### 実行例：

1. 最初に利用可能なアクションを一覧表示します：
   ```json
   {
     "name": "list_available_actions"
   }
   ```

2. 一覧からGoTo関連のアクションを探します。以下のようなアクションが含まれている可能性があります：
   - `GotoDeclaration` - 定義に移動
   - `GotoImplementation` - 実装に移動
   - `GotoSuperMethod` - スーパーメソッドに移動
   - など

3. 適切なアクションIDを見つけたら、そのアクションを実行します：
   ```json
   {
     "name": "execute_action_by_id",
     "args": {
       "actionId": "GotoDeclaration"
     }
   }
   ```

### 注意点：

- アクションIDは各IntelliJ IDEのバージョンや、インストールされているプラグインによって異なる場合があります。
- `execute_action_by_id`ツールはドキュメントによると「アクションの完了を待たない」とあるため、アクションの実行結果（定義が見つかったかどうかなど）を直接取得することはできないかもしれません。
- アクションを実行するには、エディタで適切なコンテキスト（カーソルが定義を調べたい識別子の上にあるなど）が必要です。

このアプローチを使用することで、MCPを介したLLMやその他のクライアントから「Go to Definition」機能を利用することが可能です。

## リリース・公開方法

1. settings.gradle.ktsで新しいバージョンを提供
2. Githubでリリースを作成すると、自動的に公開タスクが実行される

## 開発者の貢献方法

プロジェクトはJetBrains Open Source and Community Code of Conductに従っており、貢献は歓迎されています。プルリクエストを通じて貢献することが可能です。