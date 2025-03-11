# JetBrains MCP Server Pluginのビルド成果物

IntelliJでビルドされたMCP Server Pluginの成果物（アーティファクト）について説明します。

## メインの成果物

### 1. JAR ファイル

プラグインのメインとなる成果物は以下のパスにあります：

```
/Users/nishimura/.ghq/src/github.com/ashigirl96/vs-deps-tracer/mcp-server-plugin/build/libs/mcp-server-plugin.jar
```

同じディレクトリには以下の関連ファイルも生成されています：
- `mcp-server-plugin-base.jar`
- `mcp-server-plugin-instrumented.jar`

これらはビルドプロセスの異なる段階で生成されたファイルで、内容は本質的に同じです。

### 2. サンドボックス環境のプラグイン

ビルドプロセスでは、IntelliJ IDEのサンドボックス環境も作成され、そこにプラグインがインストールされた状態になっています：

```
/Users/nishimura/.ghq/src/github.com/ashigirl96/vs-deps-tracer/mcp-server-plugin/build/idea-sandbox/IC-2024.3/plugins/mcp-server-plugin/lib/mcp-server-plugin.jar
```

このディレクトリには、プラグインの動作に必要な依存ライブラリも含まれています：
- `annotations-13.0.jar`
- `kotlin-stdlib-1.9.22.jar`
- `kotlinx-serialization-core-jvm-1.6.3.jar`
- `kotlinx-serialization-json-jvm-1.6.3.jar`

## 配布パッケージの不在

通常、IntelliJ Platformプラグインのビルドでは、`/build/distributions` ディレクトリに `.zip` 形式のプラグイン配布パッケージが生成されますが、現在のビルドではそのディレクトリが存在しません。これは、完全な配布パッケージが生成されていない可能性を示しています。

## 配布可能なパッケージの作成方法

完全な配布可能なプラグインパッケージを作成するには、以下のいずれかの方法を使用します：

### IntelliJ IDE内から
「Build」→「Prepare Plugin Module 'mcp-server-plugin' For Deployment」メニューオプションを選択

### コマンドラインから
以下のGradleタスクを実行：

```bash
cd /Users/nishimura/.ghq/src/github.com/ashigirl96/vs-deps-tracer/mcp-server-plugin
./gradlew buildPlugin
```

このコマンドを実行すると、`/build/distributions` ディレクトリに配布可能なzipファイルが生成されます。

## プラグインのインストール方法

### 方法1: JARファイルから直接インストール

1. JetBrains IDE（IntelliJ IDEAなど）を開く
2. 「Settings」→「Plugins」→「⚙️」→「Install Plugin from Disk...」を選択
3. `/Users/nishimura/.ghq/src/github.com/ashigirl96/vs-deps-tracer/mcp-server-plugin/build/libs/mcp-server-plugin.jar` を選択
4. IDEを再起動してプラグインを有効にする

### 方法2: 配布パッケージ（ZIP）からインストール

配布パッケージが作成されている場合：

1. JetBrains IDE（IntelliJ IDEAなど）を開く
2. 「Settings」→「Plugins」→「⚙️」→「Install Plugin from Disk...」を選択
3. `/Users/nishimura/.ghq/src/github.com/ashigirl96/vs-deps-tracer/mcp-server-plugin/build/distributions/mcp-server-plugin-x.x.x.zip` を選択
4. IDEを再起動してプラグインを有効にする

### 方法3: プロジェクトから直接実行

開発やテスト目的の場合：

1. IntelliJ IDEでプラグインプロジェクトを開く
2. 「Run」→「Run Plugin」を選択（または右上の実行ボタン）
3. 新しいIntelliJ IDEインスタンスが起動し、プラグインがインストールされた状態になる

## 注意事項

- プラグインをインストールする前に、[JetBrains MCP Proxy](https://github.com/JetBrains/mcpProxy)がインストールされていることを確認してください。
- プラグインはIDE のバージョン242（2024.2）から251までをサポートしています。
- プラグインのバージョンは`settings.gradle.kts`ファイルの`extra["pluginVersion"]`で定義されています（現在は1.0.16）。