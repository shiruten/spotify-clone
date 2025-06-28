# Spotify Clone アーキテクチャ図

## コンポーネント構造

```mermaid
graph TB
    subgraph "Next.js アプリケーション"
        subgraph "レイアウト層"
            RootLayout[RootLayout<br/>app/layout.tsx]
            Figtree[Figtree Font]
        end
        
        subgraph "ページ層"
            HomePage[Home Page<br/>app/site/page.tsx]
        end
        
        subgraph "コンポーネント層"
            Sidebar[Sidebar<br/>components/Sidebar.tsx]
            Header[Header<br/>components/Header.tsx]
            Box[Box<br/>components/Box.tsx]
            SidebarItem[SidebarItem<br/>components/SidebarItem.tsx]
            Library[Library<br/>components/Library.tsx]
        end
        
        subgraph "アイコン・スタイリング"
            ReactIcons[React Icons<br/>HiHome, BiSearch, TbPlaylist, AiOutlinePlus]
            TailwindCSS[Tailwind CSS]
            TailwindMerge[tailwind-merge]
        end
    end
    
    RootLayout --> Figtree
    RootLayout --> Sidebar
    Sidebar --> Box
    Sidebar --> SidebarItem
    Sidebar --> Library
    Sidebar --> HomePage
    HomePage --> Header
    
    SidebarItem --> ReactIcons
    Library --> ReactIcons
    Box --> TailwindMerge
    SidebarItem --> TailwindMerge
    
    style RootLayout fill:#e3f2fd
    style Sidebar fill:#f3e5f5
    style Header fill:#e8f5e8
    style Box fill:#fff3e0
    style SidebarItem fill:#fff3e0
    style Library fill:#fff3e0
```

## データフロー

```mermaid
sequenceDiagram
    participant User as ユーザー
    participant RootLayout as RootLayout
    participant Sidebar as Sidebar
    participant SidebarItem as SidebarItem
    participant Library as Library
    participant Header as Header
    
    User->>RootLayout: アプリケーション起動
    RootLayout->>Sidebar: children props 渡し
    Sidebar->>SidebarItem: ルート情報 (Home/Search) 渡し
    Sidebar->>Library: レンダリング
    Sidebar->>Header: children として HomePage 渡し
    
    Note over Sidebar: usePathname() でアクティブ状態を管理
    Note over SidebarItem: アクティブ状態に基づいてスタイル適用
    Note over Library: クリックハンドラー (未実装)
    Note over Header: ログアウトハンドラー (未実装)
    
    User->>SidebarItem: ナビゲーションクリック
    SidebarItem->>User: ページ遷移
```

## コンポーネント関係図

```mermaid
classDiagram
    class RootLayout {
        +children: ReactNode
        +font: Figtree
        +metadata: Metadata
    }
    
    class Sidebar {
        +children: ReactNode
        +pathname: string
        +routes: Route[]
        +useMemo()
        +usePathname()
    }
    
    class SidebarItem {
        +icon: IconType
        +label: string
        +active: boolean
        +href: string
    }
    
    class Box {
        +children: ReactNode
        +className: string
        +twMerge()
    }
    
    class Library {
        +onClick()
    }
    
    class Header {
        +children: ReactNode
        +className: string
        +handleLogout()
    }
    
    class HomePage {
        +Header
    }
    
    RootLayout ||--|| Sidebar : contains
    Sidebar ||--o{ SidebarItem : renders
    Sidebar ||--|| Library : contains
    Sidebar ||--|| Box : wraps content
    HomePage ||--|| Header : contains
    
    Box <|-- SidebarItem : styled by
    Box <|-- Library : wrapped by
```

## ファイル構造

```mermaid
graph TD
    Root[spotify-clone/] --> App[app/]
    Root --> Components[components/]
    Root --> Config[設定ファイル群]
    
    App --> Layout[layout.tsx]
    App --> Site[site/]
    Site --> PageTsx[page.tsx]
    
    Components --> BoxTsx[Box.tsx]
    Components --> HeaderTsx[Header.tsx]  
    Components --> LibraryTsx[Library.tsx]
    Components --> SidebarTsx[Sidebar.tsx]
    Components --> SidebarItemTsx[SidebarItem.tsx]
    
    Config --> PackageJson[package.json]
    Config --> TsConfig[tsconfig.json]
    Config --> TailwindConfig[tailwind.config.ts]
    Config --> NextConfig[next.config.js]
    
    style Root fill:#e3f2fd
    style App fill:#f3e5f5
    style Components fill:#e8f5e8
    style Config fill:#fff3e0
```

## 技術スタック

```mermaid
mindmap
  root((Spotify Clone))
    Frontend Framework
      Next.js 14.0.3
      React 18
      TypeScript 5
    スタイリング
      Tailwind CSS 3.3.0
      tailwind-merge 2.1.0
      Figtree Font
    アイコン
      react-icons 4.12.0
        HiHome
        BiSearch  
        TbPlaylist
        AiOutlinePlus
    開発ツール
      ESLint
      PostCSS
      Autoprefixer
```

## 現在の実装状況

### ✅ 実装済み
- レスポンシブサイドバー (モバイルでは非表示)
- ナビゲーション (Home/Search)
- アクティブ状態の管理
- 基本的なUIコンポーネント
- Tailwind CSSによるスタイリング

### 🔄 実装中
- Header コンポーネント (基本構造のみ)
- Library コンポーネント (UI のみ)

### ⏳ 未実装 (TODO)
- ユーザー認証・ログアウト機能
- 音楽アップロード機能
- プレイリスト管理
- 音楽再生機能
- 検索機能
- ユーザープロフィール

## 設計パターン

### コンポーネント設計
- **Container/Presentational パターン**: Sidebar が状態管理、SidebarItem が表示のみ
- **Compound Component パターン**: Box コンポーネントが他のコンポーネントをラップ
- **Props Drilling**: 必要最小限のプロップス渡し

### 状態管理
- **Next.js Hooks**: `usePathname()` でルーティング状態管理
- **React Hooks**: `useMemo()` でパフォーマンス最適化
- **Client Component**: "use client" ディレクティブで適切に分離

### スタイリング戦略
- **Utility-First**: Tailwind CSS によるユーティリティクラス
- **Dynamic Styling**: `twMerge()` による条件付きスタイリング
- **Responsive Design**: モバイルファーストのレスポンシブ設計