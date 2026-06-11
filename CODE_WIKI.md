# EmbyTok Code Wiki

> 本文档提供 EmbyTok 项目的**完整代码架构**和**开发参考**，供开发者快速理解项目结构、模块职责和关键实现。
> 所有描述均基于实际代码分析。

---

## 1. 项目概览

### 1.1 项目简介

**EmbyTok** 是一款为 **Emby** 和 **Plex** 媒体服务器设计的**竖屏视频浏览客户端**，提供类似 TikTok 的体验。

| 项目属性 | 详情 |
| --- | --- |
| 技术栈 | **React 18** + **TypeScript** + **Vite** + **Tailwind CSS** + **Capacitor**（Android） |
| 图标库 | `lucide-react` |
| 动画库 | `framer-motion` |
| 测试 | `Vitest`（单元/集成）+ `Playwright`（E2E） |
| PWA | `vite-plugin-pwa` + Workbox |
| 核心功能 | 媒体库浏览 · 竖屏视频流 · 字幕 · 收藏 · 观看历史 · 搜索 · 自动连播 |
| 多端适配 | Web/移动端（StandardRoot）+ 智能电视端（TVRoot）+ Android App（Capacitor） |

### 1.2 项目版本与构建

- 版本号由 `package.json` 的 `version` 字段驱动，Vite 构建时通过 `define` 注入为 `import.meta.env.VITE_APP_VERSION`
- 生产构建使用 `terser` 压缩（`drop_console: true`、`drop_debugger: true`）
- **代码分割**：手动分包为 `react-vendor`（react/react-dom）、`ui-vendor`（lucide-react）、`motion-vendor`（framer-motion）
- **PWA**：Workbox 运行时缓存 + 图片缓存策略（`CacheFirst`，最多 100 条，7 天过期）
- `manifest.json`：`orientation: portrait`，`display: standalone`，`theme_color: #000000`

### 1.3 核心目录结构

```
/workspace
├── index.html                  # 应用入口 HTML（含 #root 和 meta viewport）
├── index.tsx                   # React 渲染入口（createRoot + StrictMode）
├── App.tsx                     # 根组件（设备检测 + 模式切换 + lazy 加载 Root）
├── types.ts                    # 全局类型定义（ServerConfig, EmbyItem, 收藏、历史、字幕等）
├── index.css                   # 全局样式（Tailwind @tailwind 指令 + 少量自定义）
├── components/                 # React UI 组件层（Presentation Layer）
│   ├── standard/
│   │   └── StandardRoot.tsx    # 标准/移动端根组件（TopBar + Feed/Grid + 浮层）
│   ├── tv/
│   │   ├── TVRoot.tsx          # 电视端根组件（方向键焦点 + 返回键层级）
│   │   ├── TVDashboard.tsx     # 电视端首页 Dashboard
│   │   ├── TVVideoGrid.tsx     # 电视端海报网格
│   │   ├── TVVideoPlayer.tsx   # 电视端播放器
│   │   └── TVSettings.tsx      # 电视端设置页
│   ├── Login.tsx               # 登录 / 连接服务器页面
│   ├── LibrarySelect.tsx       # 媒体库选择菜单（含设置/登出）
│   ├── VideoCard.tsx           # 单个视频播放卡片（核心播放逻辑）
│   ├── VideoFeed.tsx           # 竖屏滚动式视频流容器
│   ├── VideoGrid.tsx           # 栅格化海报视图
│   ├── VideoPlayer.tsx         # 基础 <video> 包装组件
│   ├── VideoInfo.tsx           # 视频元信息
│   ├── VideoSkeleton.tsx       # 加载占位（骨架屏）
│   ├── FavoritesManager.tsx    # 收藏合集管理 UI
│   ├── WatchHistoryView.tsx    # 观看历史视图
│   ├── SearchBar.tsx           # 搜索框 + 搜索历史
│   ├── SearchResults.tsx       # 搜索结果网格
│   ├── SubtitleControls.tsx    # 字幕控制（轨道、字体、颜色、位置）
│   ├── SubtitleRenderer.tsx    # 字幕覆盖层渲染
│   ├── UpdateNotification.tsx  # 新版本通知弹窗
│   ├── HeartAnimation.tsx      # 双击爱心动画
│   └── DeleteConfirmDialog.tsx # 删除确认
├── services/                   # 媒体服务客户端层（Data Access Layer）
│   ├── MediaClient.ts          # 抽象基类（协议规范）
│   ├── EmbyClient.ts           # Emby 服务器 REST API 实现
│   ├── PlexClient.ts           # Plex 服务器 HTTP API 实现
│   ├── clientFactory.ts        # 根据配置创建对应客户端实例（工厂模式）
│   └── embyService.ts          # 辅助服务
├── src/
│   ├── hooks/                  # 自定义 React Hooks（Business Logic Layer）
│   │   ├── index.ts            # 统一导出
│   │   ├── useLocalStorageState.ts  # 通用 localStorage hook（基础抽象）
│   │   ├── useConfig.ts        # 服务器配置管理 + MediaClient 实例化
│   │   ├── useLibraries.ts     # 媒体库列表 + 隐藏库管理
│   │   ├── useVideoList.ts     # 视频列表加载 · 分页 · 导航栈
│   │   ├── useFavorites.ts     # 本地收藏合集管理
│   │   ├── useWatchHistory.ts  # 观看历史管理
│   │   ├── useSearch.ts        # 搜索 + 搜索历史
│   │   ├── useSubtitles.ts     # 字幕解析（VTT）+ 当前 cue
│   │   ├── useUpdateChecker.ts # GitHub Release 更新检查
│   │   ├── useUIState.ts       # UI 状态（静音/全屏/自动连播/方向过滤）
│   │   ├── useDeviceDetection.ts  # 设备检测（iOS Safari/移动/横屏）
│   │   ├── useSmartVideoPreload.ts # 智能视频预加载策略
│   │   ├── useImagePreload.ts  # 图片预加载
│   │   ├── useLazyImage.ts     # 图片懒加载包装
│   │   ├── useGestureControls.ts  # 手势识别（点击/双击/滑动/长按）
│   │   ├── useVideoControls.ts    # 视频控制辅助
│   │   └── useTranslation.ts      # 多语言翻译 hook
│   └── locales/                # 多语言文件
│       ├── zh.ts
│       ├── en.ts
│       └── index.ts
├── utils/                      # 纯函数工具（无 React 依赖）
│   ├── device.ts              # 设备检测 · 设备 ID
│   ├── media.ts               # 媒体项类型判断 · 播放进度计算
│   ├── time.ts                # 时间格式化（100ns ticks → 可读字符串）
│   └── index.ts               # 统一 re-export
├── scripts/                    # 构建脚本与工具
│   ├── build.js               # 统一构建脚本
│   ├── benchmark-compare.js   # 基准测试对比
│   ├── version-manager.js     # 版本号管理（major/minor/patch）
│   ├── extract-changelog.js   # 从 git 提取 changelog
│   ├── pre-release-check.js   # 发布前自检
│   └── generate-*.js          # 图标、Banner 等资源生成
├── docs/                       # 额外文档
│   ├── API.md / ARCHITECTURE.md
│   ├── GIT_COMMIT_GUIDE.md
│   ├── RELEASE_CHECKLIST.md
│   ├── TESTING.md / TESTING_SUMMARY.md
│   └── VERSION_UPGRADE_GUIDE.md
├── e2e/                        # Playwright E2E 测试
│   ├── app.spec.ts
│   ├── login.spec.ts
│   ├── mobile.spec.ts
│   └── performance.spec.ts
├── android/                    # Android 原生工程（Capacitor）
│   ├── app/src/main/java/com/embytok/app/MainActivity.java
│   └── build.gradle
├── capacitor.config.ts         # Capacitor（Android）配置
├── vite.config.ts              # Vite + PWA 配置
├── vite.config.local.ts        # Vite 本地开发配置
├── tailwind.config.js          # Tailwind 配置（自定义 heart-float 动画）
├── tsconfig.json               # TypeScript 配置
├── postcss.config.js           # PostCSS（Tailwind + Autoprefixer）
├── manifest.json              # PWA 清单
├── sw.js                      # Service Worker
├── package.json               # 依赖 + scripts
└── nginx.conf / Dockerfile    # 容器化部署
```

---

## 2. 架构设计

### 2.1 整体架构图（分层模型）

```
┌──────────────────────────────────────────────────────┐
│                Presentation Layer (UI)                │
│                                                      │
│  [App.tsx] ──▶ [StandardRoot / TVRoot]               │
│                      │                                │
│     ┌──────────┬─────┴──────┬──────────┬────────┐   │
│     Login   VideoFeed   VideoGrid   VideoCard   …   │  React 组件
│     (components/)                                     │
└──────────────────┬───────────────────────────────────┘
                   │ props 传递 · 回调调用
┌──────────────────▼───────────────────────────────────┐
│             Business Logic Layer (Hooks)             │
│                                                      │
│   useConfig / useVideoList / useFavorites            │  自定义 Hooks
│   useSearch / useSubtitles / useWatchHistory         │  (src/hooks/)
│   useUIState / useUpdateChecker                      │
└──────────────────┬───────────────────────────────────┘
                   │ 纯数据接口调用
┌──────────────────▼───────────────────────────────────┐
│              Data Access Layer (Services)            │
│                                                      │
│   [clientFactory.ts] ──▶ [MediaClient 抽象基类]       │  services/
│          │                 ▲                         │
│    ┌─────┴─────┐           │                         │
│    ▼           ▼           │                         │
│  EmbyClient   PlexClient ───┘                         │
│  (Emby API)    (Plex API)                             │
└──────────────────┬───────────────────────────────────┘
                   │ HTTP(S)
┌──────────────────▼───────────────────────────────────┐
│                 External Services                    │
│                                                      │
│     Emby Server API    /    Plex Server API          │
│     GitHub Releases API /    ntfy.sh (TV 扫码登录)    │
└──────────────────────────────────────────────────────┘
```

### 2.2 核心数据流（启动 → 播放）

```
1. 用户打开页面
   └─▶ index.tsx → ReactDOM.createRoot → <App />

2. App.tsx 设备检测
   ├─▶ localStorage.embyForceDeviceMode ?
   ├─▶ navigator.userAgent 含 tv/googletv/smarttv ?
   └─▶ 决定 lazy 加载 <StandardRoot /> 或 <TVRoot />

3. StandardRoot 初始化
   ├─▶ localStorage.embyConfig ?
   │     ├─ YES → 构造 ClientFactory → client = EmbyClient | PlexClient
   │     │         ├─▶ client.getLibraries() → 填充顶栏菜单
   │     │         └─▶ client.getVideos(...) → 渲染 VideoFeed/VideoGrid
   │     └─ NO  → 显示 <Login />
   └─▶ Login.tsx 表单提交
         └─▶ ClientFactory.authenticate('emby'|'plex', url, u, p)
               └─▶ 返回 ServerConfig → localStorage.embyConfig = JSON.stringify(...)
                     └─▶ 页面重新进入"已登录"流程

4. VideoFeed 渲染与自动播放
   ├─▶ videos.map → 每项渲染 <VideoCard />
   ├─▶ IntersectionObserver (threshold: 0.85) 观察每张卡片
   └─▶ 进入视口 85% → setActiveIndex(i) → video.play()

5. 播放视频（VideoCard 内部）
   ├─▶ video.src = client.getVideoUrl(item, 'direct')
   ├─▶ 失败 → onError → playMode 切 'transcode' → 重载
   ├─▶ 再失败 → 切 'fallback'（更低码率/分辨率）
   ├─▶ 字幕：client.getSubtitleTracks(itemId) → useSubtitles → 渲染
   └─▶ 进度保存：每 5s 写 localStorage.embbytok_progress_{itemId}
```

### 2.3 状态管理策略

- **无 Redux / Zustand / MobX**：项目采用轻量的 **Hook 组合**方式管理状态
- **服务端数据**：通过 `MediaClient` 实例 + `useEffect/useCallback` 拉取，局部 `useState` 管理
- **持久化状态**：`useLocalStorageState` 统一抽象（用户配置、收藏、历史、搜索历史、UI 设置、播放进度）
- **组件间状态**：`StandardRoot` / `TVRoot` 作为顶层容器，通过 **props drilling** 将状态/回调下传给子组件（代码规模尚在可接受范围，未引入 context）
- **TV 端状态**：除 props 外，还有全局 `keydown` 监听处理方向键与 `backButton`

---

## 3. 关键类型定义

[types.ts](file:///workspace/types.ts) 是整个项目的"类型中枢"，定义所有跨模块共享的数据结构：

| 类型 / 接口 | 用途 |
| --- | --- |
| `ServerType = 'emby' \| 'plex'` | 区分媒体服务器类型 |
| `ServerConfig` | 登录后保存的配置：`url / username / token / userId / serverType` |
| `EmbyLibrary` | 媒体库：`Id / Name / CollectionType?` |
| `EmbyItem` | 媒体项（视频/剧集/季/文件夹），含 `ImageTags / UserData / SeriesName / _PlexKey / Width / Height / MediaSources` |
| `MediaSource / MediaStream` | 媒体源与媒体流信息（供转码/直接播放判断使用，`SupportsDirectPlay` 等） |
| `FeedType = 'latest' \| 'random' \| 'favorites' \| 'history'` | 视频排序/筛选模式 |
| `OrientationMode = 'vertical' \| 'horizontal' \| 'both'` | 按宽高比过滤视频 |
| `WatchHistoryItem / WatchHistory` | 观看历史条目和容器（`positionTicks / totalTicks / watchedAt / libraryId`） |
| `FavoriteCollection / FavoritesState` | 本地收藏合集：`id / name / createdAt / itemIds[]` + 默认合集 ID |
| `SubtitleTrack / SubtitleCue / SubtitleSettings` | 字幕轨道、时间片、用户设置（字体/颜色/位置/启用） |
| `GitHubRelease / UpdateCheckResult` | 版本检查用 GitHub Release 结构 |
| `VideoResponse` | `getVideos()` 返回值：`items / nextStartIndex / totalCount` |

### EmbyItem.UserData 字段特别说明

- `PlaybackPositionTicks`：上次播放位置（**100ns 单位**，除以 `10_000_000` 得秒）
- `Played`：是否已完整播放过
- `IsFavorite`：是否在服务端标记为收藏
- `LastPlayedDate`：最后播放时间（ISO 字符串）

---

## 4. 服务层详解（services/）

### 4.1 MediaClient（抽象基类）

[MediaClient.ts](file:///workspace/services/MediaClient.ts) 定义所有媒体客户端**必须实现的合约**：

```typescript
abstract class MediaClient {
  config: ServerConfig;
  constructor(config: ServerConfig);

  abstract authenticate(username, password): Promise<ServerConfig>;
  abstract getLibraries(): Promise<EmbyLibrary[]>;
  abstract getResumeItems(): Promise<EmbyItem[]>;          // TV 首页"继续观看"
  abstract getVideos(parentId, library, feedType, skip,
                     limit, orientationMode, includeIds?): Promise<VideoResponse>;
  abstract getVideoUrl(item, mode?: 'direct'|'transcode'|'fallback'): string;
  abstract getImageUrl(itemId, tag?, type?): string;
  abstract getFavorites(libraryName): Promise<Set<string>>;  // 服务端收藏
  abstract toggleFavorite(id, isFavorite, libraryName): Promise<void>;
  abstract deleteItem(id): Promise<void>;
  abstract searchItems(query): Promise<EmbyItem[]>;
  abstract getSubtitleTracks(itemId): Promise<SubtitleTrack[]>;
}
```

**设计要点**：所有客户端实现同一接口，上层（Hooks & 组件）**完全不感知具体服务器类型**。

### 4.2 EmbyClient（Emby 服务器实现）

[EmbyClient.ts](file:///workspace/services/EmbyClient.ts) 调用 Emby REST API：

#### 认证与请求头

- `POST /Users/AuthenticateByName` → 返回 `{ User: { Id, Name }, AccessToken, ServerId }`
- 请求头：`X-Emby-Token`、`X-MediaBrowser-Token`、自定义 `X-Emby-Authorization: MediaBrowser Client="EmbyTok Web", Device="...", DeviceId="...", Version="1.0.0", Token="..."`
- `DeviceId` 由 `utils/device.ts → getDeviceId()` 生成并持久化，保持 Emby 会话一致

#### 媒体库与内容获取

- `GET /Users/{userId}/Views` → 媒体库列表
- `GET /Users/{userId}/Items` → 通用内容列表，关键参数：
  - `ParentId`：库或文件夹 ID
  - `Recursive`：是否递归（非文件夹导航时通常 `true`）
  - `IncludeItemTypes`：`Movie / Video / Episode / Series / Season / Folder / BoxSet`
  - `SortBy`：`Random` 或 `DateCreated`
  - `Fields`：必须包含 `MediaSources,Width,Height,UserData,SeriesName`
  - `Limit` / `StartIndex`：分页
- 命名处理：`formatItemName()` 对 Episode 拼装 `Sxx.Eyy. 标题`

#### 方向过滤

`applyOrientationFilter(items, mode)` 在客户端按 `item.Width` / `item.Height` 过滤：
- `vertical`：要求 `height ≥ width × 0.8`
- `horizontal`：要求 `width > height`
- `both`：不过滤
- 注意：`FOLDER_TYPES`（Series/Season/Folder/BoxSet 等）始终通过，不做尺寸过滤

#### 视频 URL 策略

| 模式 | URL 形式 | 使用场景 |
| --- | --- | --- |
| `direct` | `/Videos/{id}/stream?Static=true&MediaSourceId={srcId}&PlaySessionId={time}&api_key={token}` | 找到 `MediaSource.SupportsDirectPlay = true` 的源 |
| `transcode` | `/Videos/{id}/master.m3u8?VideoCodec=h264&AudioCodec=aac&MaxWidth=1920&MaxHeight=1080&VideoBitrate=4000000&AudioBitrate=192000&...` | 硬编码的转码参数（HLS） |
| `fallback` | `/Videos/{id}/stream?VideoCodec=h264&AudioCodec=aac&MaxWidth=1280&MaxHeight=720&VideoBitrate=2000000&...` | 更保守的转码参数，用于兼容性兜底 |

#### 图片 URL

`/Items/{id}/Images/{Primary|Backdrop}?maxWidth=800&tag={ImageTags.Primary}&quality=90&api_key={token}`

#### 服务端收藏（Playlist 机制）

- 对每个 `libraryName` 创建/查找名为 `Tok-{libraryName}` 的 Playlist
- `POST /Playlists?Name=...&UserId=...` 创建新播放列表
- `POST /Playlists/{pid}/Items?Ids={itemId}` 添加条目
- `DELETE /Playlists/{pid}/Items?EntryIds={entryId}` 移除条目
- `GET /Playlists/{pid}/Items` 读取当前收藏 ID 列表
- 这实现了**跨设备的服务端同步收藏**

#### 字幕

- 从 `MediaSources[].MediaStreams[]` 中过滤 `Type === 'Subtitle'` 的轨道
- 对外部字幕（`IsExternal`），构造 `/Videos/{itemId}/{index}/Stream?api_key={token}` 供浏览器直接 fetch
- VTT 由 `useSubtitles.parseVTT` 解析为 cue 数组

### 4.3 PlexClient（Plex 服务器实现）

[PlexClient.ts](file:///workspace/services/PlexClient.ts) 使用 Plex HTTP API：

- **认证**：简化为 `X-Plex-Token` 方式（用户在登录页直接填 token），验证 `/identity` 端点
- **机器标识符**：从 `/identity` 响应的 `MediaContainer.machineIdentifier` 读取，作为 `userId`
- **库列表**：`GET /library/sections`
- **继续观看**：`GET /library/onDeck`
- **视频列表**：`GET /library/sections/{id}/all?type=1&sort=addedAt:desc&X-Plex-Token=...`
  - `random` 模式：shuffle 由客户端做
- **播放 URL**：
  - 直接播放：`item._PlexKey`（媒体 part 路径）+ `X-Plex-Token`
  - 转码：`/video/:/transcode/universal/start?path=...&protocol=hls&...`
- **图片 URL**：`/photo/:/transcode?url={thumb}&width=800&height=1200&X-Plex-Token={token}`（Plex 自己做尺寸转换）
- **字幕**：从 `Media[].Part[].Stream[]` 中挑 `streamType === 3` 的轨道

### 4.4 ClientFactory（工厂模式）

[clientFactory.ts](file:///workspace/services/clientFactory.ts) 只有两类方法：

- `create(config)`：根据 `config.serverType` 实例化 `EmbyClient` 或 `PlexClient`
- `authenticate(type, url, username, password)`：临时构造客户端并执行认证流程，返回填充好的 `ServerConfig`

---

## 5. Hooks 层详解（src/hooks/）

> Hooks 是项目的"业务逻辑容器"。所有数据拉取、状态组织、用户行为响应均在这里。每个 hook 一个文件，在 `index.ts` 统一导出。

### 5.1 useLocalStorageState（基础抽象）

```typescript
function useLocalStorageState<T>(key, initialValue): [T, Dispatch<SetStateAction<T>>]
```

- 初始化时从 `localStorage.getItem(key)` 读 JSON；失败 / 未找到 / 解析错误时用 `initialValue`
- 状态每次变化自动 `JSON.stringify` 后写回
- 是**所有持久化 Hook** 的基础：`useConfig / useLibraries / useFavorites / useWatchHistory / useSearch / useSubtitles / useUIState` 等都基于它

### 5.2 useConfig

```typescript
{ config, setConfig, client, logout }
```

- 读取 `embyConfig`，在变化时创建 `MediaClient` 实例（用 `useMemo` 缓存）
- `logout()` 清空配置 + `window.location.reload()`

### 5.3 useLibraries

```typescript
{ libraries, selectedLib, setSelectedLib, hiddenLibIds, hiddenLibIdsSet,
  toggleHiddenLib, fetchLibraries }
```

- 自动调用 `client.getLibraries()` 填充媒体库列表
- `hiddenLibIds` 持久化，用户可在菜单中隐藏不常用的库
- `toggleHiddenLib(id)` 在 Set 中增删

### 5.4 useVideoList

```typescript
{ videos, loading, hasMore, navStack, favoriteIds, viewMode, setViewMode,
  currentIndex, setCurrentIndex, loadVideos, toggleFavorite, deleteVideo,
  navigateTo, navigateBack, selectVideo }
```

- **导航栈** `navStack: NavItem[]` — 用于进入剧集/季等文件夹结构后可逐级返回
- **视图模式** `viewMode = 'feed' | 'grid'`；进入文件夹时自动切为 `grid`
- **收藏 Set** `favoriteIds`（服务端收藏 ID 集合），与 `client.toggleFavorite` 双向同步
- `loadVideos(true)` 触发**完全重置**（清空 + 拉取 + 根据首项类型决定 viewMode）
- `navigateTo(item)`：若 `isFolderType(item)` → 压栈并重新请求；否则 → `selectVideo(item)`

### 5.5 useFavorites（本地收藏合集）

```typescript
{ collections, defaultCollectionId, createCollection, deleteCollection,
  renameCollection, addToFavorites, removeFromFavorites, isFavorite,
  getCollections, getCollection, getItemCollections }
```

- **本地**收藏合集系统，与服务端 Playlist 收藏**独立并存**
- 数据结构：`collections: FavoriteCollection[]`，每个合集 `{ id, name, createdAt, itemIds[] }`
- 默认合集 ID `'default'`，名为 `"收藏"`
- 删除合集、重命名合集等操作都会写回 `embytok_favorites` key

### 5.6 useWatchHistory

```typescript
{ history, addToHistory, removeFromHistory, clearHistory,
  getHistoryItem, getProgress }
```

- 每 5 秒由 `VideoCard` 中的 `saveProgressIntervalRef` 触发 `addToHistory(itemInfo)`
- 记录 `itemId / positionTicks / totalTicks / watchedAt / libraryId`
- `MAX_HISTORY_ITEMS = 100`，超出自动淘汰最旧条目
- `getHistoryItem(itemId)` 可在 UI 上显示"继续从 mm:ss 播放"
- `getProgress(itemId)` 返回播放位置 ticks，用于海报底部进度条

### 5.7 useSearch

```typescript
{ query, results, loading, loadingMore, hasMore, searchHistory, setQuery,
  debouncedSearch, performSearch, addToHistory, removeFromHistory,
  clearHistory, loadMore }
```

- 调用 `client.searchItems(query)` 返回 `EmbyItem[]`
- `debouncedSearch` 做 `300ms` 防抖，避免打字过程频繁请求
- 搜索历史持久化在 `embytok_search_history`
- `loadMore()` 在用户滚动到底部时追加新结果（分页由服务端支持）

### 5.8 useSubtitles

```typescript
{ settings, cues, currentCue, updateSettings, toggleSubtitles, selectTrack,
  loadSubtitles, updateTime, parseVTT }
```

- `settings`：`{ enabled, selectedTrackId, fontSize, textColor, backgroundColor, position }`
- `parseVTT(content)`：**手写的轻量 VTT 解析器**，把 `WEBVTT` 文本切成 `{ startTime, endTime, text }[]`
  - 解析 `hh:mm:ss.mmm --> hh:mm:ss.mmm` + 后续文本行
  - 自动跳过 `NOTE`、`STYLE`、`REGION` 等元信息块
- `currentCue`：根据 `updateTime(seconds)` 在 cues 数组里做**线性搜索**当前时间片（cue 数量通常很小，无需二分）
- 组件层由 `SubtitleRenderer.tsx` 渲染字幕浮层，`SubtitleControls.tsx` 负责设置

### 5.9 useUpdateChecker

```typescript
{ currentVersion, latestVersion, hasUpdate, release, isChecking,
  lastCheckTime, checkError, checkForUpdates }
```

- 检查 `https://api.github.com/repos/1525745393/embytok/releases/latest`
- 版本号比较由 `parseVersion` 转为数字数组 `[major, minor, patch]` 后逐段比较
- `UpdateNotification.tsx` 在检测到新版本时弹出对话框，展示 Release body、tag_name、HTML URL、asset 下载链接

### 5.10 useUIState

```typescript
{ isMenuOpen, setIsMenuOpen, isMuted, setIsMuted, isFullscreen, setIsFullscreen,
  isAutoPlay, setIsAutoPlay, orientationMode, setOrientationMode,
  toggleFullscreen, toggleMute, toggleAutoPlay }
```

- 纯 UI 状态管理，**无网络请求**
- `orientationMode`（`vertical / horizontal / both`）持久化，配合 `EmbyClient.applyOrientationFilter` 使用
- 监听 `fullscreenchange` + `webkitfullscreenchange` 事件同步 `isFullscreen` 状态
- `toggleMute` 同时控制**所有** `<video>` 元素（通过 DOM 查询 `.video-card video` 批量设置 `muted`）

### 5.11 useDeviceDetection

```typescript
{ isIOSSafari, isMobile, isLandscape }
```

- 调用 `utils/device.ts` 与 `utils/media.ts` 的纯函数
- 返回布尔值，供组件做条件渲染（如给 iOS Safari 加特殊 CSS class，解决安全区与自动播放限制）

### 5.12 useSmartVideoPreload / useImagePreload / useLazyImage

- `useSmartVideoPreload`：根据 `activeIndex` 动态决定相邻视频的 `preload` 属性
  - active 项 → `'auto'`
  - 相邻 ±1 项 → `'metadata'`
  - 其他项 → `'none'`
- `useImagePreload`：新 Image() 预加载 URL，返回 `{ loaded, error }`
- `useLazyImage`：IntersectionObserver 驱动的懒加载图片包装

### 5.13 useGestureControls

```typescript
{ bindGestures(ref), state: { clicks, swipes, longPress, ... } }
```

- 统一识别单击、双击、水平/垂直滑动、长按
- 内部用 `pointerdown / pointermove / pointerup` 事件 + 时间/位移阈值
- 为 `VideoCard.tsx` 提供统一手势回调

### 5.14 useVideoControls

```typescript
{ togglePlay, seekBy, seekTo, setPlaybackRate, toggleMute, ... }
```

- 包装常见视频控制操作，供 `VideoCard` 和 `TVVideoPlayer` 复用

### 5.15 useTranslation

```typescript
{ t, language, setLanguage, availableLanguages }
```

- 从 `src/locales/{zh,en}.ts` 读取翻译字典
- `t('key.path')` 做嵌套 key 查询（`lodash.get` 风格手写实现）
- `language` 持久化到 `embytok_language`，默认 `'zh'`

---

## 6. 组件层详解（components/）

### 6.1 App.tsx（根组件）

[App.tsx](file:///workspace/App.tsx)

- **设备模式决策**：
  1. 读取 `localStorage.embyForceDeviceMode`
  2. 若无，检查 `navigator.userAgent.toLowerCase()` 是否含 `'tv' / 'googletv' / 'smarttv'`
  3. 返回 `'tv'` 或 `'standard'`
- **代码分割**：`React.lazy(() => import('./components/standard/StandardRoot'))` 等 + `<Suspense fallback={<LoadingFallback/>}>`
- **body class**：`useLayoutEffect` 在 `<body>` 上挂 `mode-tv` 或 `mode-standard`
- **模式切换按钮**：`handleToggleMode(mode)` → `localStorage.setItem('embyForceDeviceMode', mode)` + `window.location.reload()`

### 6.2 index.tsx（渲染入口）

[index.tsx](file:///workspace/index.tsx)

- `ReactDOM.createRoot(document.getElementById('root')!).render(<React.StrictMode><App/></React.StrictMode>)`
- 如 `#root` 不存在，抛出 `Error('Could not find root element to mount to')`

### 6.3 StandardRoot.tsx（标准/移动/Web 端 Root）

[StandardRoot.tsx](file:///workspace/components/standard/StandardRoot.tsx)

承载整个标准端的 UI 骨架和状态，是**最复杂的顶层组件**：

```
顶栏（Top Bar · flex 布局）
  ├─ 左：菜单按钮（展开 LibrarySelect：媒体库列表 + 设置 + 登出）
  ├─ 中：排序按钮（最新 / 随机 / 收藏 / 历史）→ 设置 feedType
  └─ 右：搜索 / 收藏合集 / 历史 / 全屏 / 静音 / 视图切换（feed ⇄ grid）

主体（Main · relative h-[calc(100dvh-56px)]）
  ├─ VideoFeed（竖屏流，全屏 snap）当 viewMode === 'feed'
  └─ VideoGrid（海报墙，支持 folder 导航）当 viewMode === 'grid'

浮层（Overlays · 独立 Portal 或 absolute）
  ├─ LibrarySelect（从顶栏左菜单展开）
  ├─ WatchHistoryView（从顶栏"历史"按钮弹出）
  ├─ FavoritesManager（从顶栏"收藏"按钮弹出）
  ├─ SearchBar + SearchResults（从顶栏"搜索"按钮弹出）
  ├─ UpdateNotification（新版本检查 → 自动弹出）
  └─ DeleteConfirmDialog（删除视频时确认）
```

**关键实现特征**：

- `StandardRoot` 使用 `useState` + `useCallback` 手动组织状态，**直接调用 services 层**（`ClientFactory.create(config)`）而非依赖 `useConfig` Hook
- 大量子组件使用 `React.lazy` 懒加载：`WatchHistoryView / SearchBar / SearchResults / FavoritesManager / UpdateNotification / Login / VideoFeed / VideoGrid / LibrarySelect`
- `subtitleTracksMap`（`itemId → SubtitleTrack[]`）存为 `useRef` 以避免滚动时重渲染
- 对 `[selectedLib, feedType, orientationMode]` 的变化触发 `loadVideos(true)` 重置

### 6.4 VideoFeed.tsx（竖屏流容器）

- 容器：`overflow-y-scroll snap-y snap-mandatory no-scrollbar` + 子项 `snap-center snap-always`
- 渲染：`videos.map((item, i) => <VideoCard key={item.Id} index={i} ... />)`
- 每项外层包 `div className="video-card-container h-[100dvh]"`
- **IntersectionObserver**：`threshold: 0.85`，进入 85% 即设为 `activeIndex`，触发该卡片播放，其他暂停
- `visibleIndices`：根据 `activeIndex` 计算相邻项（默认 ±2，慢速滚动范围更大）做**条件渲染**，避免长列表性能问题
- `feedType === 'random'` 时，底部显示 `"换一批"` 按钮，点击调用 `loadVideos(true)` 重新请求

### 6.5 VideoCard.tsx（核心播放组件）

> 业务最复杂的组件，整合播放控制、手势识别、字幕、双击点赞、进度保存。

#### 内部状态要点

| 状态 | 类型 | 说明 |
| --- | --- | --- |
| `isPlaying` | boolean | 当前是否播放，由 IntersectionObserver 自动切换 |
| `hasStarted` | boolean | 是否已播放至少一次，控制海报图淡出 |
| `videoPreload` | `'metadata' \| 'auto' \| 'none'` | active 时置 `'auto'`，相邻 `'metadata'`，其他 `'none'` |
| `playMode` | `'direct' \| 'transcode' \| 'fallback'` | 失败时自动降级 |
| `currentTime / duration` | number（秒） | 从 `onTimeUpdate / onLoadedMetadata` 同步 |
| `playbackRate` | number | 默认 1.0；长按或菜单可切换 |
| `isSeeking / seekOffset` | boolean / number | 滑动 seek 时临时暂停视频 |
| `isSpeedAdjusting` | boolean | 区分"临时加速"和"固定倍速" |
| `hearts` | array | 双击爱心动画的短暂记录（最多 5 个同时存在） |

#### 播放模式自动降级

```
playback 失败
  └─▶ playMode = 'direct' → onError 触发
       └─▶ 若 playMode === 'direct' → 切 'transcode'，重载 src
            └─▶ 若又失败 → 切 'fallback'
                 └─▶ 再失败 → 显示错误 UI + 手动"尝试转码播放"按钮
```

#### 手势识别（pointer 事件）

- **单击**（tap < 300ms 且无后续第二击）→ 显示/隐藏控制面板（底部进度条）
- **双击**（两击间隔 < 300ms，位移 < 30px）→ 在点击位置弹出爱心动画；若 item 未服务端收藏，同时 `onToggleFavorite(itemId)`
- **水平滑动**（`|dx| > 50px && |dx| > |dy|`）→ 拖动时显示 `±xx s` 预览，释放时 `video.currentTime += dx / pxPerSecond`
- **长按**（`pointerdown` 持续 > 500ms 无显著移动）→ 临时 `playbackRate = 2.0`，释放恢复；长按时若上下滑动则转为"固定倍速"
- **垂直滑动**（`|dy| > 50px && |dy| > |dx|`）→ 音量或亮度（TV 端特化逻辑）

#### 字幕渲染

- 调用 `client.getSubtitleTracks(itemId)` 拉轨道列表
- 选中轨道后 → `useSubtitles.loadSubtitles(track)` 解析 VTT 并填充 `cues`
- `SubtitleRenderer` 根据 `settings.position / bgColor / fontSize / textColor` 渲染当前时间片文本
- 字幕浮层用 `pointer-events-none` 避免拦截视频点击

#### 播放进度持久化

- `saveProgressIntervalRef`（5 秒定时器，仅当 `active && playing` 时启用）
- 保存 `localStorage.embbytok_progress_{itemId} = { time(秒), duration(秒), timestamp: Date.now() }`
- 下次进入同视频 → 在 `onLoadedMetadata` 中恢复 `video.currentTime = time`
- 配合 `useWatchHistory.addToHistory({ itemId, positionTicks, totalTicks, watchedAt, libraryId })` 做跨视频历史

### 6.6 TVRoot.tsx（电视端 Root）

电视端与标准端的核心差异：

| 方面 | Standard | TV |
| --- | --- | --- |
| 导航方式 | 点击 / 触摸滚动 | 方向键（ArrowUp/Down/Left/Right）、Enter、Back |
| 焦点管理 | 无（浏览器原生） | 全局 `keydown` 处理 + `findNearest` 根据矩形距离寻找下一焦点元素 |
| 首页布局 | 竖屏视频流或海报墙 | 左侧 Sidebar + 右侧 `TVDashboard`（"继续观看" + 各媒体库最新内容） |
| 设置入口 | 集成在 `LibrarySelect` 菜单中 | 独立 `TVSettings` 页面 |
| 登录 | 手动填账号 | 可选"扫码登录"（TV 显示二维码，手机扫描后推送配置到 ntfy.sh） |
| 返回键 | 浏览器原生 | Capacitor `backButton` 事件 + 浏览器 `Escape/Backspace` |

#### 扫码登录机制

1. TV 端调用 `getDeviceId()` 获得 `embytok-tv-xxxxxx`
2. TV 订阅 `https://ntfy.sh/embytok_sync_{deviceId}` 的 SSE JSON 流
3. TV 显示二维码，指向 `https://embytok.vercel.app/setup?id={deviceId}`
4. 手机扫描后，在 setup 页填写 `url / username / token`，提交时以 `POST https://ntfy.sh/embytok_sync_{deviceId}` 推送 JSON 消息
5. TV 收到消息后自动填入登录表单并触发 `ClientFactory.authenticate`

#### 返回键层级

- 由 `executeBackLogic(currentPage, setCurrentPage, goBackFn)` 统一处理
- 层级：`播放中 → 设置/历史/收藏 → 媒体库列表 → Discover Dashboard → 退出(closeApp)`
- TV 端在 `uiStateRef` 中维护当前页面栈快照，避免频繁 re-render

### 6.7 TVDashboard / TVVideoGrid / TVVideoPlayer

- **`TVDashboard`**：顶部"继续观看"横向滚动行（由 `client.getResumeItems()`），下方按库分行展示各库的最新 12 项
- **`TVVideoGrid`**：更大的海报网格，支持 6/8 列；focus ring 用 `focus:ring-8 focus:ring-white`，焦点缩放动画必须显著
- **`TVVideoPlayer`**：全屏播放 + 方向键控制（← → 切换 ±10s、↑ ↓ 在 `items[]` 中切换集数、Enter 播放/暂停、Back 退出）

### 6.8 其他 UI 组件

| 组件 | 职责 |
| --- | --- |
| `Login.tsx` | 登录表单 · Emby/Plex 切换 · 语言切换 · TV 扫码登录 · 自动补 `http://` 前缀 |
| `LibrarySelect.tsx` | 媒体库列表 · 隐藏库切换 · 登出 · 更新检查 · 设备模式切换 |
| `VideoGrid.tsx` | 标准端海报墙 · 文件夹导航 · 海报底部播放进度条 · hover 高亮 |
| `VideoPlayer.tsx` | 简洁 `<video>` 包装 · 错误降级 · 倍速/进度 UI |
| `SubtitleControls.tsx` | 字幕轨道切换 · 字体/大小/颜色/位置设置 |
| `SubtitleRenderer.tsx` | 根据 cues + currentTime + settings 渲染覆盖层 |
| `FavoritesManager.tsx` | 创建/重命名/删除合集 · 在合集间管理视频条目 |
| `WatchHistoryView.tsx` | 观看历史条目的卡片列表，可点击继续播放 |
| `SearchBar.tsx` | 搜索输入（带历史） + 下拉建议 |
| `SearchResults.tsx` | 结果网格（与 VideoGrid 共享样式） |
| `UpdateNotification.tsx` | 版本更新提示对话框（展示 tag、body、下载链接） |
| `HeartAnimation.tsx` | 双击爱心动画（`animate-heart-float`，关键帧：缩放下滑 + 渐隐） |
| `DeleteConfirmDialog.tsx` | 删除确认（"删除视频将影响原文件"警告） |
| `VideoSkeleton.tsx` | 加载占位（骨架屏，模拟海报+标题占位） |
| `VideoInfo.tsx` | 视频元信息卡片（标题、年份、简介、时长、评分） |

---

## 7. 工具函数与通用能力（utils/）

### 7.1 [device.ts](file:///workspace/utils/device.ts)

```typescript
isMobile(): boolean            // UA 含 'android|iphone|ipad|ipod' 即真
isLandscape(): boolean         // window.innerWidth > window.innerHeight
isTVDevice(): boolean          // UA 含 'tv|googletv|smarttv|appletv'
isIOSSafari(): boolean         // 正则判断 iPad/iPhone/iPod 上的 Safari（排除 Chrome/Firefox iOS）
getDeviceId(): string          // 首次调用生成 'embytok-web-xxxxxx' 并存 localStorage，之后复用
```

`getDeviceId()` 用于 `EmbyClient.getHeaders()` 中的 `DeviceId`，保持 Emby 会话一致。

### 7.2 [media.ts](file:///workspace/utils/media.ts)

```typescript
const FOLDER_TYPES = ['Series', 'Season', 'Folder', 'CollectionFolder', 'BoxSet', 'show', 'season']
isFolderType(item: EmbyItem): boolean        // 点击后是"进入子目录"还是"开始播放"
isVerticalVideo(item): boolean               // Height ≥ Width × 0.8
isHorizontalVideo(item): boolean             // Width > Height
calculatePlaybackProgress(positionTicks, runTimeTicks): number // 0-100，用于海报进度条
formatFileSize(bytes): string                // '1.2 GB' 等
```

### 7.3 [time.ts](file:///workspace/utils/time.ts)

```typescript
ticksToSeconds(ticks): number                // ticks / 10_000_000
secondsToTicks(sec): number                  // sec × 10_000_000
formatSeconds(sec): 'mm:ss' | 'h:mm:ss'      // 短格式（播放器进度）
formatTimeText(ticks): string                // '1小时 20分钟'（中文友好）
formatTime(ticks): string                    // '1h 20m'（紧凑）
```

**单位说明**：Emby / Plex 使用 **100 纳秒**（`1e-7 s`）作为 `ticks` 单位；`1 tick = 100 ns`。

---

## 8. 样式、动画与 PWA 能力

### 8.1 Tailwind 配置（[tailwind.config.js](file:///workspace/tailwind.config.js)）

- `content` 路径覆盖 `index.html`、`App.tsx`、`index.tsx`、`components/**/*`、`services/**/*`、`types.ts`
- 自定义动画 `heart-float`（关键帧：`scale(0) translateY(60px) opacity(0)` → `scale(1.8) translateY(30px) opacity(1)` → `scale(1.2) translateY(15px) opacity(0.8)` → `scale(0.8) translateY(0) opacity(0)`）

### 8.2 Vite + PWA（[vite.config.ts](file:///workspace/vite.config.ts)）

- **代码分割**：`react-vendor`、`ui-vendor`（lucide-react）、`motion-vendor`（framer-motion）
- **压缩**：`terser` + `drop_console` + `drop_debugger`；目标 `es2015`
- **PWA manifest**：`orientation: portrait`、`display: standalone`、`theme_color: #000000`、`background_color: #000000`
- **运行时缓存**：`CacheFirst` 策略缓存 `png/jpg/jpeg/svg/gif/webp`，最多 100 条、7 天过期
- **开发模式**：`server.host: '0.0.0.0'`，允许局域网/移动设备访问
- `vite.config.local.ts`：本地开发专用配置（与生产配置分离）

### 8.3 全局样式（[index.css](file:///workspace/index.css)）

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

/* 关键自定义 class */
.no-scrollbar           /* 隐藏原生滚动条（-ms-overflow-style: none; scrollbar-width: none; ::-webkit-scrollbar { display: none }） */
body.mode-tv .tv-only   /* TV 端专属样式切换 */
body.mode-standard .standard-only
```

### 8.4 关键 CSS class 约定

| class | 用途 |
| --- | --- |
| `h-[100dvh]` | 动态视口高度，避免 iOS 地址栏导致的高度错位 |
| `snap-y snap-mandatory` + `snap-center snap-always` | 视频流逐屏 snap |
| `no-scrollbar` | 隐藏滚动条（feed 容器使用） |
| `ios-safari` | 由 App.tsx 相关逻辑挂到 `<html>` 上，用于 iOS 特殊布局修正 |
| `mode-tv` / `mode-standard` | 由 App.tsx 挂到 `<body>` 上，用于 TV/标准端分支样式 |

---

## 9. 项目运行与开发脚本

### 9.1 常用脚本（[package.json](file:///workspace/package.json)）

```bash
npm run dev            # 启动 Vite 开发服务器（http://localhost:5173，host 0.0.0.0）
npm run build          # tsc 类型检查 + Vite 构建 → dist/
npm run preview        # 本地预览生产构建
npm test               # Vitest 单元测试（监听模式）
npm test:ui            # Vitest 浏览器 UI
npm test:coverage      # Vitest + c8 覆盖率报告
npm test:run           # Vitest 单次运行（CI）
npm test:bench         # 基准测试（vitest bench）
npm test:e2e           # Playwright 端到端测试
npm test:e2e:ui        # Playwright UI 模式
npm test:e2e:install   # 安装 Playwright 浏览器依赖
npm test:performance   # Playwright 性能测试（grep performance）
npm benchmark:save     # 保存当前基准为 baseline
npm benchmark:compare  # 与 baseline 对比
npm cap:add            # Capacitor 添加 Android 平台（首次）
npm cap:sync           # Capacitor 同步 dist/ → android/
npm build:android      # = tsc && vite build && cap sync android
npm version:current    # 打印当前版本
npm version:patch      # 升级 patch 版本
npm version:minor      # 升级 minor 版本
npm version:major      # 升级 major 版本
npm version:set 1.2.3  # 指定版本号
npm changelog:generate # 从 git 历史提取更新日志
npm release:check      # 发布前自检（版本号、格式、测试）
npm build:app          # 统一构建脚本（scripts/build.js）
npm build:app:debug    # 构建脚本（debug 模式）
```

### 9.2 本地开发最小流程

```bash
# 1. 安装依赖
npm install

# 2. 启动开发服务器（默认 http://localhost:5173，host 0.0.0.0 供局域网访问）
npm run dev

# 3. 打开页面 → 填 Emby/Plex 服务器地址、账号信息 → 进入主界面浏览

# 4. 运行测试
npm test                # 开发：持续监听模式
npm test:run            # CI：单次运行

# 5. 构建 & 预览生产包
npm run build && npm run preview
```

### 9.3 Android 构建（Capacitor）

```bash
npm run build:android       # = tsc && vite build && cap sync android
# 之后在 Android Studio 中打开 android/ 目录构建 APK / AAB
```

[capacitor.config.ts](file:///workspace/capacitor.config.ts) 关键配置：

- `server.androidScheme: 'http'`（内网 http 优先，避免证书问题）
- `server.allowNavigation: ['*']`（允许跳转到任意域名，含自建服务器）
- `plugins.CapacitorHttp.enabled: false`（禁用原生 HTTP，改用浏览器 Fetch 以提高视频流兼容性）
- Android 入口：`android/app/src/main/java/com/embytok/app/MainActivity.java`

### 9.4 容器化部署（Docker）

- `Dockerfile`：多阶段构建（Node 构建 → Nginx 提供静态文件）
- `nginx.conf`：静态文件路由 + SPA fallback + gzip 压缩 + 安全头
- `docker-compose.simple.yml`：最简版（仅 web 服务）
- `docker-compose.yml`：完整版（web + 可选反向代理 / 自动证书）
- `docker-compose.synology.yml`：群晖 (Synology) 专用 compose
- 镜像构建：`docker build -t embytok:latest .`，运行：`docker run -p 80:80 embytok:latest`

---

## 10. 多语言（src/locales/）

- `zh.ts` / `en.ts` 导出同样键名的翻译对象（扁平或嵌套对象均可）
- `type Translations = typeof zh` 保证两套语言键名完全同构（结构校验）
- `useTranslation.ts` 提供 `t(key, fallback?)` 查值，支持 `'common.cancel'` 这类嵌套点式查询
- 目前 `Login.tsx` / `StandardRoot.tsx` / `TVRoot.tsx` / `VideoCard.tsx` 等组件各有**内联双语**（简单情况下够用），`useTranslation` 提供规模化方案供未来统一

---

## 11. 关键数据流程示例

### 11.1 用户点击播放一个视频（标准端 feed 模式）

```
VideoCard 变为 active
  └─▶ useEffect([isActive])
       ├─▶ video.preload = 'auto'
       ├─▶ video.src = client.getVideoUrl(item, 'direct')
       ├─▶ video.play()
       │     ├─▶ onPlaying → setIsPlaying(true)
       │     │              + 启动 saveProgressIntervalRef（每 5s 写 localStorage）
       │     └─▶ onTimeUpdate → setCurrentTime(sec)（驱动字幕渲染）
       ├─▶ 如 MediaSources 里有字幕：client.getSubtitleTracks(itemId)
       │     └─▶ useSubtitles.loadSubtitles(selectedTrack)
       │           └─▶ parseVTT(rawText) → cues[]
       │                └─▶ SubtitleRenderer 渲染
       ├─▶ 如播放失败 → onError → playMode 切 'transcode' 并重载 src
       │     └─▶ 再失败 → 切 'fallback'
       └─▶ 用户滚动离开 → IntersectionObserver 触发 unmount
             └─▶ 暂停 + video.preload = 'none' + 保存最后进度
```

### 11.2 从媒体库切换到"随机"模式

```
StandardRoot 中 feedType state = 'random'
  └─▶ useEffect([feedType, selectedLib, orientationMode])
       └─▶ loadVideos(true)
            └─▶ client.getVideos(parentId?, library?, 'random', 0, PAGE_SIZE, orientationMode)
                 └─▶ EmbyClient.getVideos
                      └─▶ params.append('SortBy', 'Random')
                           └─▶ GET /Users/{userId}/Items?...
                                └─▶ setVideos(items)
                                     └─▶ VideoFeed 重新渲染 items
                                          └─▶ IntersectionObserver 激活第 0 项 → 自动播放
```

### 11.3 服务端收藏（Emby 版）

```
用户点击爱心图标
  └─▶ onToggleFavorite(item.Id)
       └─▶ client.toggleFavorite(item.Id, !wasFavorite, library.Name)
            └─▶ getTokPlaylistId(library.Name)
                 ├─▶ GET /Users/{userId}/Items?IncludeItemTypes=Playlist&Recursive=true
                 │      └─▶ 查找 name === 'Tok-{libraryName}'
                 ├─▶ 未找到 → POST /Playlists?Name=Tok-{libraryName}&UserId={userId}（创建）
                 ├─▶ 若为"加入收藏" → POST /Playlists/{pid}/Items?Ids={itemId}
                 └─▶ 若为"取消收藏"
                      ├─▶ GET /Playlists/{pid}/Items 找到 entryId
                      └─▶ DELETE /Playlists/{pid}/Items?EntryIds={entryId}
  同步刷新 favoriteIds Set → UI 爱心状态更新
```

### 11.4 字幕渲染管道

```
<video> onTimeUpdate(sec)
  └─▶ useSubtitles.updateTime(sec)
       └─▶ currentCue = cues.find(c => c.startTime ≤ sec < c.endTime)
            └─▶ SubtitleRenderer 渲染 currentCue.text
                 └─▶ 应用 settings（fontSize/bgColor/textColor/position）
```

---

## 12. 测试结构

项目测试分层（每个源文件旁有对应的 `*.test.tsx` / `*.test.ts`）：

| 目录/文件 | 覆盖范围 |
| --- | --- |
| `components/__tests__/*.test.tsx` | 所有 UI 组件的渲染/交互测试 |
| `components/standard/__tests__/` | StandardRoot 特定路径 |
| `components/tv/__tests__/` | TVRoot / TVDashboard / TVSettings / TVVideoGrid / TVVideoPlayer |
| `services/__tests__/EmbyClient.test.ts` | EmbyClient API 调用 mock 测试（fetch Mock） |
| `services/__tests__/PlexClient.test.ts` | PlexClient API 调用 mock 测试 |
| `src/hooks/__tests__/*.test.ts` | 所有自定义 Hook 的行为测试（`@testing-library/react` 的 `renderHook`） |
| `utils/__tests__/*.test.ts` | 纯函数工具单元测试 |
| `utils/__tests__/media.benchmark.ts` | 工具函数的性能基准（`vitest bench`） |
| `e2e/app.spec.ts` | 应用整体冒烟测试 |
| `e2e/login.spec.ts` | 登录流程 E2E |
| `e2e/mobile.spec.ts` | 移动端布局 E2E |
| `e2e/performance.spec.ts` | 性能指标 E2E |

运行：

```bash
npm test                     # Vitest 监听模式（开发时）
npm test:run                 # Vitest 单次运行
npm test:coverage            # 带覆盖率（.coverage/ 输出）
npm test:bench               # 基准测试 → 与 baseline 对比
npm test:e2e                 # Playwright（首次需 `npx playwright install`）
npm test:performance         # 仅跑性能 E2E
```

---

## 13. 开发者最佳实践与约定

### 13.1 文件与命名

- **组件文件**：`PascalCase.tsx`，文件名与默认导出同名；测试文件 `*.test.tsx`
- **Hooks 文件**：`useCamelCase.ts`，一个 hook 一个文件；在 `src/hooks/index.ts` 中统一 re-export
- **类型**：共享类型一律放入 `types.ts`；组件本地 Props/State 类型写在文件顶部
- **服务文件**：`PascalCase` 类名（`EmbyClient`、`PlexClient`）
- **工具函数**：`camelCase.ts`，纯函数无 React 依赖

### 13.2 数据访问与状态

- 始终通过 `MediaClient` 抽象（`services/`）调用 API，**不在组件里直接 `fetch`**
- 持久化 Key 前缀：`embytok_` 或 `embbytok_progress_`，避免与其他应用在 `localhost` 下冲突
- 新的持久化状态优先用 `useLocalStorageState<T>(key, initial)` 实现，不要手写 `localStorage.getItem/setItem`

### 13.3 错误处理与降级

- `try/catch` + **降级逻辑**：播放模式 `direct → transcode → fallback` 自动降级
- 网络错误 UI：显示错误卡片 + 手动"重试"按钮
- 字幕加载失败：静默降级（不显示字幕，但不阻塞播放）

### 13.4 性能优化

- 图片一律 `loading="lazy"` + `decoding="async"`（由 `useLazyImage` 统一处理）
- 长列表使用 `IntersectionObserver` 做**条件渲染**（`visibleIndices`）
- 视频 `preload` 策略：非 active 项降为 `'metadata'` 或 `'none'`，避免带宽浪费
- `React.memo` + 自定义 `arePropsEqual` 用于 `VideoCard` 等高渲染频率组件
- `useCallback` / `useMemo` 用于传给子组件的回调与派生数据
- 代码分割：所有大体积子组件用 `React.lazy` 懒加载

### 13.5 可访问性与焦点

- 所有交互元素 `tabIndex=0` 或使用原生 `<button>`
- TV 端依赖方向键导航，`focus:ring` 视觉反馈必须显著（`focus:ring-4 focus:ring-white`）
- 语义化：播放器用 `role="region" aria-label="Video player"`，按钮带 `aria-label`

### 13.6 提交与发布

- Git 提交信息规范参考 `docs/GIT_COMMIT_GUIDE.md`（Conventional Commits 风格）
- 发布前检查：`npm run release:check`（版本号、格式、测试、lint）
- 发布清单：`docs/RELEASE_CHECKLIST.md`

---

## 14. 关键扩展点（下一步可做）

1. **字幕格式扩展**：`useSubtitles.parseVTT` 可扩展为支持 **SRT**（时间戳格式不同）、**ASS/SSA**（样式复杂）、**闭字幕**（CEA-608/708）
2. **转码参数用户可配置**：当前 `EmbyClient.getVideoUrl` 的 bitrate / width 为硬编码，可改为从 `useUIState` 读取用户偏好
3. **统一 Hook 化 StandardRoot**：目前 `StandardRoot` / `TVRoot` 仍用"手搓"状态管理 + 局部 useEffect，后续可抽取成 `useStandardAppState` 等组合 Hook，提高可测试性与可读性
4. **TV 端语音搜索**：Android TV `SpeechRecognizer` + Capacitor plugin → 语音 → `useSearch.performSearch()`
5. **离线观看**：利用 PWA 缓存策略保存常看视频的转码流（`sw.js` 已提供 Workbox 基础，需新增 `runtimeCaching` 对 `/Videos/*/stream*` 的缓存策略 + 后台同步）
6. **收藏同步**：当前"本地收藏合集"（`useFavorites`）与"服务端 Playlist 收藏"（`MediaClient.toggleFavorite`）**独立并存**。可以做双向同步，让用户在多设备上看到同一个收藏合集
7. **Cast (Chromecast) 支持**：引入 `cast.framework`，将当前播放项推送到 TV

---

## 15. 项目关键文件快速索引

| 文件 | 作用 |
| --- | --- |
| [App.tsx](file:///workspace/App.tsx) | 根组件，设备模式决策 |
| [types.ts](file:///workspace/types.ts) | 全局类型定义 |
| [services/MediaClient.ts](file:///workspace/services/MediaClient.ts) | 媒体客户端抽象基类 |
| [services/EmbyClient.ts](file:///workspace/services/EmbyClient.ts) | Emby API 实现 |
| [services/PlexClient.ts](file:///workspace/services/PlexClient.ts) | Plex API 实现 |
| [services/clientFactory.ts](file:///workspace/services/clientFactory.ts) | 客户端工厂 |
| [components/standard/StandardRoot.tsx](file:///workspace/components/standard/StandardRoot.tsx) | 标准端主界面 |
| [components/tv/TVRoot.tsx](file:///workspace/components/tv/TVRoot.tsx) | 电视端主界面 |
| [components/VideoCard.tsx](file:///workspace/components/VideoCard.tsx) | 核心播放卡片 |
| [components/VideoFeed.tsx](file:///workspace/components/VideoFeed.tsx) | 竖屏流容器 |
| [src/hooks/index.ts](file:///workspace/src/hooks/index.ts) | Hooks 统一导出 |
| [src/hooks/useLocalStorageState.ts](file:///workspace/src/hooks/useLocalStorageState.ts) | 持久化基础 Hook |
| [utils/device.ts](file:///workspace/utils/device.ts) | 设备检测 |
| [utils/media.ts](file:///workspace/utils/media.ts) | 媒体项判断 |
| [utils/time.ts](file:///workspace/utils/time.ts) | 时间格式化 |
| [vite.config.ts](file:///workspace/vite.config.ts) | 构建配置 |
| [tailwind.config.js](file:///workspace/tailwind.config.js) | Tailwind 配置 |
| [capacitor.config.ts](file:///workspace/capacitor.config.ts) | Capacitor 配置 |
| [package.json](file:///workspace/package.json) | 依赖 & 脚本 |

---

_本 Wiki 基于项目当前代码自动整理，建议在架构显著变更后同步更新。_
