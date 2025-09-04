# Assistant UI プロジェクト完全理解ガイド

## プロジェクト概要
assistant-uiは、ReactベースのAI チャットUIライブラリです。ChatGPTのようなUI体験を提供し、完全にカスタマイズ可能なprimitiveコンポーネントで構築されています。

## アーキテクチャ全体図

```
┌─────────────────────────────────────────────────────────────┐
│                     Application Layer                       │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐ │
│  │   examples/     │ │     apps/       │ │    python/      │ │
│  │  (使用例)        │ │  (docs, registry)│ │   (Python統合)   │ │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘ │
└─────────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────┐
│                      Library Layer                          │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐ │
│  │  @assistant-ui/ │ │  @assistant-ui/ │ │  @assistant-ui/ │ │
│  │     react       │ │  react-ai-sdk   │ │     cloud       │ │
│  │  (コアライブラリ)  │ │   (AI SDK統合)   │ │  (クラウド統合)   │ │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘ │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐ │
│  │  react-markdown │ │     styles      │ │      cli        │ │
│  │  (Markdown表示)  │ │    (スタイル)    │ │   (CLIツール)    │ │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

## メインライブラリ (@assistant-ui/react) の詳細構造

```
packages/react/src/
├── index.ts              # 外部向けAPI エクスポート
├── internal.ts           # 内部API（アドバンスユーザー向け）
├── api/                  # 公開API定義
│   ├── AssistantRuntime.ts    # メインランタイムAPI
│   ├── ThreadRuntime.ts       # スレッドランタイムAPI  
│   └── ThreadListRuntime.ts   # スレッドリストAPI
├── primitives/           # UI Primitive コンポーネント
│   ├── thread/          # スレッド関連UI
│   ├── message/         # メッセージ表示UI
│   ├── composer/        # 入力UI
│   ├── actionBar/       # アクションバーUI
│   ├── branchPicker/    # ブランチ選択UI
│   └── attachment/      # 添付ファイルUI
├── runtimes/             # バックエンド統合層
│   ├── core/            # コア実装
│   ├── external-store/  # 外部ストア連携
│   ├── local/           # ローカル実行
│   └── adapters/        # アダプターパターン
├── context/              # React Context管理
│   ├── providers/       # Context Provider
│   ├── react/           # React Context定義
│   └── stores/          # Zustand状態管理
├── types/                # TypeScript型定義
└── utils/                # ユーティリティ関数
```

## 学習の推奨パス（実装レベル対応）

### Level 1: 基本使用方法の理解
**目標**: 基本的なチャットUIが作れるようになる

1. **基本サンプルの分析**
   ```typescript
   // examples/with-ai-sdk-v5/app/page.tsx
   export default function Home() {
     const runtime = useChatRuntime(); // AI SDK統合
     return (
       <AssistantRuntimeProvider runtime={runtime}>
         <Thread />
       </AssistantRuntimeProvider>
     );
   }
   ```

2. **Thread コンポーネントの構造理解**
   ```typescript
   // examples/with-ai-sdk-v5/components/assistant-ui/thread.tsx
   <ThreadPrimitive.Root>           // スレッドのルートコンテナ
     <ThreadPrimitive.Viewport>      // スクロール可能なビューポート
       <ThreadPrimitive.Messages     // メッセージ一覧
         components={{
           UserMessage,              // ユーザーメッセージコンポーネント  
           AssistantMessage,         // AIメッセージコンポーネント
           EditComposer             // 編集用コンポーザー
         }}
       />
       <Composer />                 // 入力フィールド
     </ThreadPrimitive.Viewport>
   </ThreadPrimitive.Root>
   ```

### Level 2: Primitive システムの理解  
**目標**: カスタムコンポーネントが作成できるようになる

1. **Primitive パターンの理解**
   ```typescript
   // packages/react/src/primitives/thread/ThreadRoot.tsx
   export const ThreadPrimitiveRoot = forwardRef<
     ThreadPrimitiveRoot.Element,
     ThreadPrimitiveRoot.Props
   >((props, ref) => {
     return <Primitive.div {...props} ref={ref} />; // Radix Primitiveベース
   });
   ```

2. **主要Primitiveコンポーネント**
   - `ThreadPrimitive.*` - スレッド全体の管理
   - `MessagePrimitive.*` - 個別メッセージの表示・操作  
   - `ComposerPrimitive.*` - メッセージ入力UI
   - `ActionBarPrimitive.*` - メッセージに対するアクション
   - `BranchPickerPrimitive.*` - 分岐したメッセージの選択

3. **条件付きレンダリング**
   ```typescript
   <ThreadPrimitive.If empty={false}>          // スレッドが空でない時
   <ThreadPrimitive.If running>                // AI応答中の時  
   <MessagePrimitive.If copied>                // コピー完了時
   ```

### Level 3: Runtime システムの理解
**目標**: バックエンド統合やカスタムランタイムが作成できる

1. **Runtime階層の理解**
   ```typescript
   // packages/react/src/api/AssistantRuntime.ts
   export type AssistantRuntime = {
     readonly threads: ThreadListRuntime;  // 複数スレッド管理
     readonly thread: ThreadRuntime;       // メインスレッド
     registerModelContextProvider(provider: ModelContextProvider): Unsubscribe;
   };
   ```

2. **Core Runtime の実装**
   ```typescript
   // packages/react/src/runtimes/core/AssistantRuntimeCore.tsx
   export class AssistantRuntimeCore {
     public readonly threads: ThreadListRuntimeCore;
     
     registerModelContextProvider(provider: ModelContextProvider) {
       return this._modelContextProviders.add(provider);
     }
   }
   ```

3. **カスタムランタイムの作成パターン**
   - `external-store/` - 外部状態管理システムとの統合
   - `adapters/` - 既存APIとの連携層
   - MessageRepositoryパターンによるメッセージ管理

### Level 4: Context System の理解
**目標**: 状態管理やカスタムフックが作成できる

1. **Provider階層**
   ```typescript
   // packages/react/src/context/providers/AssistantRuntimeProvider.tsx
   <AssistantContext.Provider value={context}>
     <ThreadRuntimeProvider runtime={runtime.thread}>
       {children}
     </ThreadRuntimeProvider>
   </AssistantContext.Provider>
   ```

2. **Zustand による状態管理**
   ```typescript
   // Zustandストアベースの状態管理
   const useAssistantRuntimeStore = (runtime: AssistantRuntime) => {
     const [store] = useState(() => create(() => runtime));
     // ...
   };
   ```

3. **カスタムフック作成パターン**
   - Context から値を取得
   - 状態の購読と更新
   - エラーハンドリング

## 実装に必要な重要概念

### 1. メッセージフロー
```
User Input → Composer → Runtime → Backend → Response → MessageRepository → UI Update
```

### 2. ストリーミング処理
```typescript
// assistant-stream パッケージによるストリーミング処理
export type StreamChunk = {
  type: 'text-delta' | 'tool-call-begin' | 'tool-call-delta' | 'finish';
  // ...
};
```

### 3. 拡張ポイント
- **カスタムメッセージコンポーネント**: `MessagePrimitive.Parts` の`components`プロパティ
- **カスタムツールUI**: `tools` プロパティでツール固有のUI定義
- **カスタムランタイム**: `AssistantRuntime`インターフェースの実装
- **カスタムテーマ**: CSS変数とTailwindクラスのオーバーライド

### 4. テストとデバッグ
```bash
# テスト実行
pnpm test

# 開発サーバー（例を使用）
cd examples/with-ai-sdk-v5
pnpm dev

# 型チェック
pnpm build
```

## 機能実装のための具体的な手順

### 新しいPrimitiveコンポーネントの追加
1. `packages/react/src/primitives/[component-name]/` ディレクトリ作成
2. インターフェース定義（`[Component].tsx`）
3. `index.ts` でエクスポート
4. `packages/react/src/primitives/index.ts` に追加

### カスタムランタイムの実装
1. `AssistantRuntime` インターフェースの実装
2. `ThreadRuntime` インターフェースの実装  
3. メッセージストリーミングの処理
4. エラーハンドリングの実装

### UIカスタマイゼーション
1. 既存のcomponentsサンプルをコピー
2. 必要なPrimitiveコンポーネントを組み合わせ
3. スタイリング（TailwindCSS/CSS変数）
4. 条件付きレンダリングの調整

この詳細ガイドにより、assistant-uiプロジェクトの実装レベルでの理解と、具体的な機能改修が可能になります。

## Azure でのバックエンド・ホスティング

### デプロイメント選択肢

#### 1. Azure Static Web Apps (推奨)
**最もコスト効率の良い選択肢**

```bash
# デプロイ手順
npx assistant-ui create my-assistant-app
cd my-assistant-app
git init && git add . && git commit -m "initial"
# Azure Static Web Apps リソースを作成し、GitHub連携設定
```

**無料プラン制限 (2025年)**
- **ストレージ**: 250MB/app
- **帯域幅**: 100GB/月
- **Azure Functions**: 1M リクエスト/月無料
- **認証**: Microsoft/GitHub 無制限、カスタムロール 25ユーザー
- **SSL**: 無料
- **カスタムドメイン**: 無料
- **SLA**: なし（個人・ホビー用途）

**Standard プラン**
- **ストレージ**: 250MB/app  
- **帯域幅**: 100GB/月 + 追加分 $0.20/GB
- **Azure Functions**: 1M リクエスト/月無料 + $0.20/1M リクエスト
- **SLA**: 99.95%保証

#### 2. Azure App Service
**エンタープライズ向け本格運用**

**Basic プラン (B1)**
- **価格**: 約 $13.14/月
- **CPU**: 1コア
- **RAM**: 1.75GB
- **ストレージ**: 10GB
- **カスタムドメイン**: 対応
- **SSL**: 無料

**Standard プラン (S1)**  
- **価格**: 約 $73.00/月
- **CPU**: 1コア
- **RAM**: 1.75GB  
- **ストレージ**: 50GB
- **スケールアウト**: 10インスタンスまで
- **ステージング環境**: 5スロット

### AI バックエンド統合

#### Azure OpenAI Service 料金 (2025年)

**標準 (従量課金) モデル**
- **GPT-4o**: $2.50/1M input tokens, $10.00/1M output tokens
- **GPT-4o mini**: $0.150/1M input tokens, $0.600/1M output tokens  
- **GPT-3.5 Turbo**: $0.500/1M input tokens, $1.500/1M output tokens

**プロビジョニング済みスループット (PTU)**
- **予約容量**: 時間単位の固定料金
- **月契約**: 標準料金から約20%割引
- **年契約**: 標準料金から約40%割引

**Batch API**
- **割引率**: 標準料金から50%割引
- **処理時間**: 24時間以内

### デプロイメント手順

#### Static Web Apps へのデプロイ

1. **Azure リソース作成**
```bash
# Azure CLI でリソース作成
az staticwebapp create \
    --name my-assistant-app \
    --resource-group my-resource-group \
    --source https://github.com/[username]/[repo-name] \
    --location "East Asia" \
    --branch main \
    --app-location "/" \
    --api-location "api" \
    --output-location ".next"
```

2. **環境変数設定**
```bash
# Azure OpenAI 接続設定
az staticwebapp appsettings set \
    --name my-assistant-app \
    --setting-names AZURE_OPENAI_API_KEY=your_api_key \
                   AZURE_OPENAI_ENDPOINT=your_endpoint
```

3. **GitHub Actions 設定** (自動生成)
```yaml
# .github/workflows/azure-static-web-apps.yml
name: Azure Static Web Apps CI/CD
on:
  push:
    branches: [ main ]
jobs:
  build_and_deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build And Deploy
        uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }}
          repo_token: ${{ secrets.GITHUB_TOKEN }}
          action: "upload"
          app_location: "/"
          api_location: "api"
          output_location: ".next"
```

#### App Service へのデプロイ

1. **Azure Web App 作成**
```bash
az webapp create \
    --resource-group my-resource-group \
    --plan my-service-plan \
    --name my-assistant-app \
    --runtime "NODE:18-lts"
```

2. **環境変数設定**
```bash  
az webapp config appsettings set \
    --resource-group my-resource-group \
    --name my-assistant-app \
    --settings AZURE_OPENAI_API_KEY=your_api_key
```

3. **デプロイ設定**
```bash
# GitHub からの継続的デプロイ設定
az webapp deployment source config \
    --resource-group my-resource-group \
    --name my-assistant-app \
    --repo-url https://github.com/[username]/[repo-name] \
    --branch main \
    --manual-integration
```

### 推奨構成とコスト見積もり

#### 小規模プロジェクト (MVP/検証用)
```
Azure Static Web Apps (Free) + Azure OpenAI (従量課金)
月額コスト: $0 + AI使用料のみ
- 10K メッセージ/月: 約 $5-15
- 適用場面: プロトタイプ、個人プロジェクト
```

#### 中規模プロジェクト (スタートアップ)
```
Azure Static Web Apps (Standard) + Azure OpenAI (従量課金)
月額コスト: $9 + AI使用料
- 100K メッセージ/月: 約 $50-150  
- 適用場面: 成長中のアプリケーション
```

#### 大規模プロジェクト (エンタープライズ)
```
Azure App Service (S1) + Azure OpenAI (PTU)
月額コスト: $73 + $500-2000 (PTU)
- 1M+ メッセージ/月: $573-2073
- 適用場面: 本格運用、高トラフィック
```

### 参考リンク

**Azure サービス**
- [Azure Static Web Apps 料金](https://azure.microsoft.com/en-us/pricing/details/app-service/static/)
- [Azure App Service 料金](https://azure.microsoft.com/en-us/pricing/details/app-service/windows/)  
- [Azure OpenAI Service 料金](https://azure.microsoft.com/en-us/pricing/details/cognitive-services/openai-service/)
- [Azure 料金計算ツール](https://azure.microsoft.com/en-us/pricing/calculator/)

**デプロイメント ガイド**
- [Next.js を Azure Static Web Apps にデプロイ](https://learn.microsoft.com/en-us/azure/static-web-apps/deploy-nextjs-hybrid)
- [Azure OpenAI Service クイックスタート](https://learn.microsoft.com/en-us/azure/ai-services/openai/chatgpt-quickstart)
- [Azure CLI リファレンス](https://docs.microsoft.com/en-us/cli/azure/)

**コスト管理**
- [Azure Cost Management](https://learn.microsoft.com/en-us/azure/cost-management-billing/)
- [Azure OpenAI コスト管理](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/manage-costs)