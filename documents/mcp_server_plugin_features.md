# JetBrains MCP Server Plugin（vs-deps-tracer mcp-server-plugin）の機能

このプラグインは、大規模言語モデル（LLM）とJetBrains IDE間のシームレスな統合を可能にするModel Context Protocol（MCP）サーバーの実装です。以下の主な機能を提供します：

## 1. IDE操作のための基本機能

### ファイル操作
- 現在開いているファイルのテキスト取得（`get_open_in_editor_file_text`）
- 現在開いているファイルのパス取得（`get_open_in_editor_file_path`）
- 選択されたテキストの取得（`get_selected_in_editor_text`）
- ファイルのテキスト置換（`replace_current_file_text`、`replace_file_text_by_path`）
- 選択テキストの置換（`replace_selected_text`）
- 新規ファイルの作成（`create_new_file_with_text`）
- ファイルを開く（`open_file_in_editor`）
- すべての開いているファイルの内容とパスを取得（`get_all_open_file_texts`、`get_all_open_file_paths`）

### プロジェクト探索
- ファイル名の部分一致による検索（`find_files_by_name_substring`）
- ファイル内容の検索（`search_in_files_content`）
- フォルダ内のファイル一覧取得（`list_files_in_folder`）
- プロジェクトのモジュールとその依存関係の取得（`get_project_modules`、`get_project_dependencies`）

### デバッグ機能
- ブレークポイントの設定/解除（`toggle_debugger_breakpoint`）
- 現在設定されているブレークポイントの取得（`get_debugger_breakpoints`）

### 実行機能
- 実行構成（Run Configuration）のリスト取得（`get_run_configurations`）
- 指定した実行構成の実行（`run_configuration`）

### IDE操作
- 利用可能なアクションのリスト取得（`list_available_actions`）
- アクションの実行（`execute_action_by_id`）
- 進行中のインジケータのステータス取得（`get_progress_indicators`）
- 指定時間の待機（`wait`）

## 2. 拡張機能開発のためのフレームワーク

このプラグインは、サードパーティのプラグイン開発者が独自のMCPツールを実装するための拡張ポイントシステムを提供しています：

- `mcpTool`拡張ポイントを通じて、独自ツールをプラグインに統合可能
- 独自ツールは`AbstractMcpTool`クラスを継承し、引数データクラスを定義して実装

## 3. 必要条件

- [JetBrains MCP Proxy](https://github.com/JetBrains/mcpProxy)のインストール
- JetBrains IDE（IntelliJ IDEA、WebStormなど）

## 利用シナリオ

このプラグインを使用すると、大規模言語モデル（Claude、ChatGPTなど）から：

1. IDEで開いているコードの内容を分析
2. コードの特定部分を修正・更新
3. プロジェクト内のファイルを検索・探索
4. 新しいファイルの作成やコードの生成
5. デバッグ用のブレークポイントの設定
6. IDEの機能（実行、ビルドなど）の呼び出し

といった操作が可能になります。

このプラグインはLLMとIDE間の橋渡しをすることで、コーディング支援、コードレビュー、デバッグ支援などを自動化・効率化するための基盤技術となっています。

## カスタムツール実装例

```kotlin
class MyCustomTool : AbstractMcpTool<MyArgs>() {
    override val name: String = "myCustomTool"
    override val description: String = "Description of what your tool does"

    override fun handle(project: Project, args: MyArgs): Response {
        // ツールのロジックを実装
        return Response.ok("Result")
    }
}

// 引数データクラスの定義
@Serializable
data class MyArgs(
    val param1: String,
    val param2: Int
)
```

## プラグインXMLでのツール登録方法

```xml
<idea-plugin>
    <!-- プラグイン設定 -->
    <depends>com.intellij.mcpServer</depends>
    
    <extensions defaultExtensionNs="com.intellij.mcpServer">
        <mcpTool implementation="com.example.MyCustomTool"/>
    </extensions>
</idea-plugin>
```