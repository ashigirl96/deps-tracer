# JetBrains MCP Server Pluginのビルド方法

JetBrains MCP Server Plugin（vs-deps-tracer mcp-server-plugin）をビルドするための手順を説明します。

## 前提条件

以下の開発環境が必要です：

1. **JDK (Java Development Kit) 17**
   - プラグインは JDK 17 を使用してビルドされます
   - 現在のシステムには JDK 23 がインストールされていますが、プラグインは JDK 17 を要求しています

2. **Gradle**
   - ビルドツールとして Gradle を使用しています
   - プロジェクトには Gradle Wrapper（`gradlew`）が含まれているため、Gradle をインストールする必要はありません

3. **IntelliJ IDEA**
   - プラグイン開発には IntelliJ IDEA が推奨されます
   - Community Edition でも開発可能です

## ビルド方法

### 方法1: IntelliJ IDEAでのビルド（推奨）

1. IntelliJ IDEAでプロジェクトを開く：
   ```
   /Users/nishimura/.ghq/src/github.com/ashigirl96/vs-deps-tracer/mcp-server-plugin
   ```

2. プロジェクトが読み込まれたら、右上の「Run Plugin」ボタンをクリックする
   - これにより、プラグインがビルドされ、テスト用のIDEインスタンスが起動します
   - このIDEインスタンスにはプラグインがすでにインストールされています

3. テスト用のIDEインスタンスでプラグインの動作を確認できます

### 方法2: コマンドラインでのビルド

#### 1. JDK 17 のインストール

macOS では Homebrew を使用して JDK 17 をインストールできます：

```bash
brew install openjdk@17
```

インストール後、環境変数を設定するか、ビルド時に JDK 17 を明示的に指定します。

#### 2. プラグインのビルド

プロジェクトディレクトリに移動し、Gradle Wrapper を使ってビルドを実行します：

```bash
cd /Users/nishimura/.ghq/src/github.com/ashigirl96/vs-deps-tracer/mcp-server-plugin
./gradlew build
```

ビルドが成功すると、プラグインの JAR ファイルが `build/libs` ディレクトリに生成されます。

#### 3. プラグインの配布可能なファイルを作成

以下のコマンドを実行して、IntelliJ プラットフォーム用の配布可能なプラグインファイル（zip）を作成します：

```bash
./gradlew buildPlugin
```

このコマンドは `build/distributions` ディレクトリに `.zip` 形式のプラグインパッケージを作成します。

## バージョン管理

プラグインのバージョンは `settings.gradle.kts` ファイルで定義されています：

```kotlin
extra["pluginVersion"] = "1.0.16"
```

新しいバージョンをリリースする場合は、このバージョン番号を更新します。

## プラグインのインストール

ビルドされたプラグインは以下の方法でインストールできます：

1. JetBrains IDE（IntelliJ IDEA、WebStorm など）を開く
2. 「Preferences」（または「Settings」）→「Plugins」→「⚙️」→「Install Plugin from Disk...」を選択
3. `build/distributions` ディレクトリにあるプラグインの zip ファイルを選択
4. IDE を再起動してプラグインを有効にする

## 注意事項

- MCP サーバープラグインを使用するには、[JetBrains MCP Proxy](https://github.com/JetBrains/mcpProxy) が必要です。
- このプラグインは IntelliJ ベースの IDE 向けに設計されており、オープンソースのコミュニティエディションを使用しています。
- プラグインは IDE のバージョン 242（2024.2）から 251 までをサポートしています。

## トラブルシューティング

ビルド時に問題が発生した場合は、以下を確認してください：

- 正しいバージョンの JDK がインストールされていること
- Gradle のキャッシュをクリアする（`./gradlew clean`）
- より詳細なエラーメッセージを取得する（`./gradlew build --stacktrace` または `--info`）