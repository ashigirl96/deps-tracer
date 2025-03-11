# JetBrains MCP 通信ログ分析

## 概要

このドキュメントは、JetBrains MCP (Model Context Protocol) サーバープラグインとの通信内容を記録したものです。AIアシスタント（Goose）がユーザーの指示に従い、JetBrains IDEの機能にアクセスするために行った通信内容とその結果を時系列で記録しています。

## セッション情報

- **日時**: 2025-03-11
- **使用ツール**: Goose AIアシスタント
- **IDE**: JetBrains IDE

## 通信内容

### 1. アクション一覧の取得（失敗）

**リクエスト**:
```json
{
  "name": "list_available_actions"
}
```

**レスポンス**:
```
Failed to execute tool list_available_actions, message Access is allowed from Event Dispatch Thread (EDT) only; 
If you access or modify model on EDT consider wrapping your code in WriteIntentReadAction or ReadAction; 
see https://jb.gg/ij-platform-threading for details
Current thread: Thread[#90,Netty Builtin Server 2,5,main] 23644472 (EventQueue.isDispatchThread()=false)
SystemEventQueueThread: Thread[#63,AWT-EventQueue-0,6,main] 772917906
```

**分析**:
このエラーは、JetBrains IDEの重要な設計原則であるスレッディングモデルに関連しています。UI関連の操作は「イベントディスパッチスレッド（EDT）」から実行する必要があり、MCP経由のこのリクエストは別スレッドから実行されたため失敗しました。これはIDEの内部アーキテクチャに起因する制約です。

### 2. シンボル検索アクションの実行（成功）

**リクエスト**:
```json
{
  "name": "execute_action_by_id",
  "args": {
    "actionId": "GotoSymbol"
  }
}
```

**レスポンス**:
```
Tool execute_action_by_id executed successfully
ok
```

**分析**:
アクション実行のリクエストは成功し、IDEでシンボル検索ダイアログが表示されました。このリクエストでは、特定のアクションID（GotoSymbol）を指定して実行しています。アクション実行はEDTの制約を回避できる形で実装されているようです。

### 3. テキスト検索の実行（成功）

**リクエスト**:
```json
{
  "name": "search_in_files_content",
  "args": {
    "searchText": "setIsModalVisible"
  }
}
```

**レスポンス**:
```json
[
  {"path": "apps/[app1]/src/pages/[Feature]/[Subfeature]/Show.tsx", "name": "Show.tsx"},
  {"path": "apps/[app2]/src/screens/[Feature]/index.tsx", "name": "index.tsx"},
  {"path": "packages/[package1]/src/components/Shared/[Component1].tsx", "name": "[Component1].tsx"},
  {"path": "apps/[app1]/src/components/Shared/Navigation/[Component2].tsx", "name": "[Component2].tsx"},
  {"path": "packages/[package2]/src/components/[Feature]/[Feature]Content/[Component3].tsx", "name": "[Component3].tsx"},
  {"path": "packages/[package1]/src/components/Shared/[Component4].tsx", "name": "[Component4].tsx"},
  {"path": "packages/[package2]/src/components/[Feature2]/[Component5].tsx", "name": "[Component5].tsx"},
  {"path": "packages/[package1]/src/components/[Feature3]/[Feature4]/Form.tsx", "name": "Form.tsx"},
  {"path": "packages/[package1]/src/components/Shared/[Component1].tsx", "name": "[Component1].tsx"},
  {"path": "apps/[app2]/src/modules/[Feature5]/[Feature6]/[Component6].tsx", "name": "[Component6].tsx"},
  {"path": "apps/[app2]/src/components/[Feature]/[Component7].tsx", "name": "[Component7].tsx"},
  {"path": "apps/[app2]/src/components/[Feature]/[Feature7]/Modal.tsx", "name": "Modal.tsx"}
]
```

**分析**:
プロジェクト全体で「setIsModalVisible」という文字列を検索するリクエストが成功し、12のファイルがマッチしました。この機能はIDEの全文検索機能にアクセスし、結果をJSON形式で返しています。検索は正規表現を使用せず、単純な文字列マッチで行われています。

### 4. ファイル内容の取得（成功）

**リクエスト**:
```json
{
  "name": "get_file_text_by_path",
  "args": {
    "pathInProject": "apps/[app1]/src/pages/[Feature]/[Subfeature]/Show.tsx"
  }
}
```

**レスポンス**:
```
// 長いファイル内容（省略）
import React, { useState } from 'react'
// ...その他のインポートとコード
function RequestReservationShowPage() {
  // ...コンポーネントの実装
  const [isModalVisible, setIsModalVisible] = useState<boolean>(false)
  // ...その他のコード
}
export default RequestReservationShowPage
```

**分析**:
特定のファイルの内容を取得するリクエストが成功しました。レスポンスには完全なファイル内容が含まれており、これにより「setIsModalVisible」の使用コンテキストを分析できました。このファイルでは、Reactの`useState`フックを使用してモーダルの表示状態を管理していることがわかります。

### 5. 別のファイル内容の取得（成功）

**リクエスト**:
```json
{
  "name": "get_file_text_by_path",
  "args": {
    "pathInProject": "packages/[package1]/src/components/Shared/[Component1].tsx"
  }
}
```

**レスポンス**:
```
// 長いファイル内容（省略）
export function UnpaidBillNotificationModal() {
  const token = useToken()
  const [isModalVisible, setIsModalVisible] = useState(false)
  // ...その他のコード
  
  useEffect(() => {
    if (unpaidBillNotification?.notified === false) {
      setIsModalVisible(true)
    }
  }, [isModalVisible, unpaidBillNotification?.notified])
  
  const handleCloseModal = async () => {
    // ...モーダルを閉じる処理
    .finally(() => setIsModalVisible(false))
  }
  
  // ...レンダリングコード
}
```

**分析**:
別のファイルの内容も取得に成功し、こちらでも「setIsModalVisible」の使い方を確認できました。このファイルでは、モーダルの表示状態を管理するだけでなく、`useEffect`フックを使用して条件付きで自動表示する機能や、非同期処理と連携した状態更新の例も確認できます。

### 6. 特定コンポーネント定義の調査

**リクエスト**:
```json
{
  "name": "get_file_text_by_path",
  "args": {
    "pathInProject": "apps/[app2]/src/screens/[Feature]/Reservations/Request.tsx"
  }
}
```

**レスポンス**:
```
// ファイル内容（省略）
import { DetailModalContent } from '@[company]/[package1]/src/components/[Feature]/RequestView/DetailModalContent'
// ...その他のインポートとコード
```

**分析**:
このリクエストでは、特定のファイル内で使用されている`DetailModalContent`コンポーネントのインポート元を特定しました。インポート文からコンポーネントの実際の定義場所を特定できました。

### 7. コンポーネント定義ファイルの取得（成功）

**リクエスト**:
```json
{
  "name": "get_file_text_by_path",
  "args": {
    "pathInProject": "packages/[package1]/src/components/[Feature]/RequestView/DetailModalContent.tsx"
  }
}
```

**レスポンス**:
```
// 長いファイル内容（省略）
export function DetailModalContent({
  restaurantId,
  data,
  isInPossibleRange,
  onPressApproval,
  onPressRejection,
}: DetailModalContentProps) {
  // ...コンポーネントの実装
}
```

**分析**:
`DetailModalContent`コンポーネントの実際の定義を含むファイルの取得に成功しました。このコンポーネントのプロパティや実装詳細を確認することができました。

## 技術的考察

### MCPプロトコルの特徴

1. **シンプルなJSON形式**:
   - リクエストは「name」と必要に応じて「args」を含むシンプルなJSON
   - レスポンスは成功/失敗情報と、必要に応じて結果データを含む

2. **IDE機能へのアクセス**:
   - ファイル操作（内容取得、検索）
   - アクション実行（GotoSymbolなど）
   - UI要素の操作

3. **IDEのアーキテクチャによる制約**:
   - EDTに関連する制約が存在
   - 一部機能はスレッディングモデルの影響を受ける

### 実用的な応用

- **コード探索**: 特定のシンボルやパターンを持つファイルの特定
- **コンテキスト理解**: ファイル内容の取得によるコードの詳細分析
- **IDE機能の活用**: アクション実行によるIDE機能のプログラム的利用

## まとめ

JetBrains MCPプロトコルは、AIアシスタントがIDE機能にプログラム的にアクセスするための強力な仕組みを提供しています。今回のセッションでは、コード検索、ファイル内容取得、アクション実行などの基本的な機能を使用しましたが、一部のリクエストはIDEのアーキテクチャ上の制約により制限されることも明らかになりました。

このプロトコルを活用することで、AIアシスタントはユーザーの作業コンテキストを深く理解し、よりインテリジェントな支援を提供することが可能となります。