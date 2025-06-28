# Spotify Clone コンポーネントフロー図

## レンダリングフロー

```mermaid
flowchart TD
    A[アプリケーション開始] --> B[RootLayout レンダリング]
    B --> C[Figtree フォント読み込み]
    B --> D[Sidebar コンポーネント初期化]
    
    D --> E[usePathname フック実行]
    E --> F[現在のパス取得]
    F --> G[routes 配列作成 useMemo]
    
    G --> H{パスが /search?}
    H -->|Yes| I[Search アクティブ状態]
    H -->|No| J[Home アクティブ状態]
    
    I --> K[SidebarItem レンダリング]
    J --> K
    
    K --> L[アイコンとラベル表示]
    L --> M[アクティブ状態でスタイル適用]
    
    D --> N[Box コンポーネント 1]
    N --> O[SidebarItem 群レンダリング]
    
    D --> P[Box コンポーネント 2]
    P --> Q[Library コンポーネント]
    Q --> R[プレイリストアイコンと追加ボタン]
    
    D --> S[メインコンテンツエリア]
    S --> T[children プロップス]
    T --> U[HomePage コンポーネント]
    U --> V[Header コンポーネント]
    
    style A fill:#e1f5fe
    style K fill:#e8f5e8
    style V fill:#fff3e0
```

## ユーザーインタラクションフロー

```mermaid
flowchart TD
    Start[ユーザーアクション] --> Check{アクションタイプ}
    
    Check -->|ナビゲーションクリック| Nav[SidebarItem クリック]
    Check -->|追加ボタンクリック| Add[Library 追加ボタン]
    Check -->|ログアウトクリック| Logout[Header ログアウト]
    
    Nav --> NavCheck{どのナビゲーション?}
    NavCheck -->|Home| HomeAction[/ パスに遷移]
    NavCheck -->|Search| SearchAction[/search パスに遷移]
    
    HomeAction --> UpdatePath[パス状態更新]
    SearchAction --> UpdatePath
    UpdatePath --> ReRender[コンポーネント再レンダリング]
    ReRender --> UpdateActive[アクティブ状態更新]
    UpdateActive --> End[完了]
    
    Add --> AddHandler[onClick ハンドラー実行]
    AddHandler --> AddTodo[TODO: アップロード機能]
    AddTodo --> End
    
    Logout --> LogoutHandler[handleLogout 実行]
    LogoutHandler --> LogoutTodo[TODO: ログアウト機能]
    LogoutTodo --> End
    
    style Start fill:#e3f2fd
    style End fill:#e8f5e8
    style AddTodo fill:#ffeb3b
    style LogoutTodo fill:#ffeb3b
```

## コンポーネント初期化シーケンス

```mermaid
sequenceDiagram
    participant App as Next.js App
    participant Layout as RootLayout
    participant SB as Sidebar
    participant SI as SidebarItem
    participant Lib as Library
    participant Home as HomePage
    participant Header as Header
    
    App->>Layout: アプリケーション起動
    Layout->>Layout: Figtree フォント設定
    Layout->>SB: children props 渡し
    
    SB->>SB: usePathname() 実行
    SB->>SB: useMemo() でルート配列作成
    
    loop routes 配列
        SB->>SI: props 渡し (icon, label, active, href)
        SI->>SI: twMerge() でスタイル計算
        SI-->>SB: レンダリング完了
    end
    
    SB->>Lib: レンダリング
    Lib->>Lib: onClick ハンドラー設定
    Lib-->>SB: レンダリング完了
    
    SB->>Home: children として渡し
    Home->>Header: レンダリング
    Header->>Header: useRouter() 初期化
    Header->>Header: handleLogout 設定
    Header-->>Home: レンダリング完了
    Home-->>SB: レンダリング完了
    SB-->>Layout: レンダリング完了
    Layout-->>App: アプリケーション準備完了
```

## 状態管理フロー

```mermaid
flowchart LR
    subgraph "Next.js Router"
        Router[Next.js Router]
        Pathname[Current Pathname]
    end
    
    subgraph "Sidebar Component"
        UsePathname[usePathname Hook]
        UseMemo[useMemo Hook]
        Routes[Routes Array]
    end
    
    subgraph "SidebarItem Components"
        Item1[Home Item]
        Item2[Search Item]
        Active1[Active State]
        Active2[Active State]
    end
    
    Router --> Pathname
    Pathname --> UsePathname
    UsePathname --> UseMemo
    UseMemo --> Routes
    
    Routes --> Item1
    Routes --> Item2
    
    Routes --> Active1
    Routes --> Active2
    
    Active1 --> Style1[Conditional Styling]
    Active2 --> Style2[Conditional Styling]
    
    style Router fill:#e3f2fd
    style Routes fill:#e8f5e8
    style Style1 fill:#fff3e0
    style Style2 fill:#fff3e0
```

## エラーハンドリングフロー

```mermaid
flowchart TD
    UserAction[ユーザーアクション] --> TryCatch{エラーチェック}
    
    TryCatch -->|成功| Success[正常処理]
    TryCatch -->|エラー| ErrorType{エラータイプ}
    
    ErrorType -->|ナビゲーションエラー| NavError[ルーティングエラー]
    ErrorType -->|レンダリングエラー| RenderError[コンポーネントエラー]
    ErrorType -->|スタイリングエラー| StyleError[CSS エラー]
    
    NavError --> Fallback1[デフォルトページ表示]
    RenderError --> Fallback2[エラーバウンダリ]
    StyleError --> Fallback3[デフォルトスタイル]
    
    Success --> Complete[処理完了]
    Fallback1 --> Complete
    Fallback2 --> Complete
    Fallback3 --> Complete
    
    style UserAction fill:#e3f2fd
    style Success fill:#e8f5e8
    style NavError fill:#ffebee
    style RenderError fill:#ffebee
    style StyleError fill:#ffebee
    style Complete fill:#e8f5e8
```

## パフォーマンス最適化フロー

```mermaid
flowchart TD
    Render[コンポーネントレンダリング] --> Memo{useMemo チェック}
    
    Memo -->|依存関係変更なし| Cache[キャッシュから取得]
    Memo -->|依存関係変更あり| Recalc[再計算実行]
    
    Cache --> FastRender[高速レンダリング]
    Recalc --> Calculate[routes 配列計算]
    Calculate --> NewCache[新しいキャッシュ作成]
    NewCache --> SlowRender[通常レンダリング]
    
    FastRender --> TailwindMerge[twMerge 実行]
    SlowRender --> TailwindMerge
    
    TailwindMerge --> OptimizedCSS[最適化された CSS]
    OptimizedCSS --> FinalRender[最終レンダリング]
    
    style Render fill:#e3f2fd
    style Cache fill:#e8f5e8
    style FastRender fill:#e8f5e8
    style OptimizedCSS fill:#fff3e0
    style FinalRender fill:#e8f5e8
```

## コンポーネント通信フロー

```mermaid
graph LR
    subgraph "Parent Components"
        RootLayout --> Sidebar
    end
    
    subgraph "Sidebar Internal"
        Sidebar --> Box1[Navigation Box]
        Sidebar --> Box2[Library Box]
        Sidebar --> Main[Main Content]
    end
    
    subgraph "Child Components"
        Box1 --> SidebarItem1[Home Item]
        Box1 --> SidebarItem2[Search Item]
        Box2 --> Library
        Main --> HomePage
        HomePage --> Header
    end
    
    subgraph "Props Flow"
        Props1[icon, label, active, href]
        Props2[children]
        Props3[className, children]
    end
    
    Sidebar -.->|props| Props1
    Props1 -.-> SidebarItem1
    Props1 -.-> SidebarItem2
    
    Sidebar -.->|children| Props2
    Props2 -.-> Main
    
    Box1 -.->|styling props| Props3
    Props3 -.-> SidebarItem1
    
    style RootLayout fill:#e3f2fd
    style Sidebar fill:#f3e5f5
    style Props1 fill:#fff3e0
    style Props2 fill:#fff3e0
    style Props3 fill:#fff3e0
```