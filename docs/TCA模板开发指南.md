# TCA 模板开发指南 - 基于 Todos 项目分析

## 项目概述

本文档基于 Swift Composable Architecture (TCA) 的 Todos 示例项目，详细分析了如何使用 TCA 构建应用程序，并提供了基于现有模板开发新项目的完整指南。

## Todos 项目架构分析

### 项目结构
```
Examples/Todos/
├── Todos/                    # 主应用目录
│   ├── TodosApp.swift       # 应用入口点
│   ├── Todo.swift           # 单个 Todo 项的 Reducer 和 View
│   ├── Todos.swift          # 主 Todos 功能的 Reducer 和 View
│   └── Assets.xcassets      # 资源文件
├── TodosTests/              # 测试目录
│   └── TodosTests.swift     # 完整的测试套件
└── Todos.xcodeproj          # Xcode 项目文件
```

### 依赖关系分析

#### 1. 项目依赖配置
Todos 项目通过 Xcode 的 Swift Package Manager 集成 TCA：

```swift
// 在 Xcode 项目中配置
dependencies: [
  .package(url: "https://github.com/pointfreeco/swift-composable-architecture", from: "1.17.0")
]

// 目标依赖
targets: [
  .target(
    name: "Todos",
    dependencies: [
      .product(name: "ComposableArchitecture", package: "swift-composable-architecture")
    ]
  )
]
```

#### 2. 核心导入
每个 Swift 文件都导入必要的框架：
```swift
import ComposableArchitecture  // TCA 核心框架
import SwiftUI                 // UI 框架
```

### 核心架构组件分析

#### 1. 应用入口点 (TodosApp.swift)
```swift
@main
struct TodosApp: App {
  static let store = Store(initialState: Todos.State()) {
    Todos()
      ._printChanges()  // 开发时打印状态变化
  }

  var body: some Scene {
    WindowGroup {
      AppView(store: Self.store)
    }
  }
}
```

**关键特性：**
- 使用静态 Store 避免与 Xcode 预览冲突
- `_printChanges()` 用于开发调试
- 简洁的应用启动配置

#### 2. 单个 Todo 项 (Todo.swift)
```swift
@Reducer
struct Todo {
  @ObservableState
  struct State: Equatable, Identifiable {
    var description = ""
    let id: UUID
    var isComplete = false
  }

  enum Action: BindableAction, Sendable {
    case binding(BindingAction<State>)
  }

  var body: some Reducer<State, Action> {
    BindingReducer()  // 自动处理双向绑定
  }
}
```

**设计亮点：**
- 使用 `@ObservableState` 启用观察机制
- 实现 `BindableAction` 支持 SwiftUI 双向绑定
- `BindingReducer()` 自动处理绑定逻辑
- 遵循 `Identifiable` 协议支持列表操作

#### 3. 主功能模块 (Todos.swift)
```swift
@Reducer
struct Todos {
  @ObservableState
  struct State: Equatable {
    var editMode: EditMode = .inactive
    var filter: Filter = .all
    var todos: IdentifiedArrayOf<Todo.State> = []

    var filteredTodos: IdentifiedArrayOf<Todo.State> {
      // 计算属性实现过滤逻辑
    }
  }

  enum Action: BindableAction, Sendable {
    case addTodoButtonTapped
    case binding(BindingAction<State>)
    case clearCompletedButtonTapped
    case delete(IndexSet)
    case move(IndexSet, Int)
    case sortCompletedTodos
    case todos(IdentifiedActionOf<Todo>)
  }

  @Dependency(\.continuousClock) var clock
  @Dependency(\.uuid) var uuid
  private enum CancelID { case todoCompletion }

  var body: some Reducer<State, Action> {
    BindingReducer()
    Reduce { state, action in
      // 处理各种 Action
    }
    .forEach(\.todos, action: \.todos) {
      Todo()  // 组合子 Reducer
    }
  }
}
```

**核心特性：**
- 使用 `IdentifiedArrayOf` 管理列表数据
- 依赖注入系统 (`@Dependency`)
- 副作用处理 (时钟、UUID 生成)
- 防抖动机制 (debouncing)
- 子 Reducer 组合 (`forEach`)

## 基于模板开发新项目的步骤

### 步骤 1: 项目初始化

#### 1.1 创建新的 Xcode 项目
```bash
# 使用 Xcode 创建新项目
# 选择 iOS App 模板
# 选择 SwiftUI 界面
# 选择 Swift 语言
```

#### 1.2 添加 TCA 依赖
在 Xcode 中：
1. 选择项目 → Package Dependencies
2. 点击 "+" 添加包
3. 输入 URL: `https://github.com/pointfreeco/swift-composable-architecture`
4. 选择版本并添加到目标

#### 1.3 项目结构规划
```
YourApp/
├── App/                     # 应用层
│   ├── YourApp.swift       # 应用入口
│   └── AppView.swift       # 根视图
├── Features/               # 功能模块
│   ├── FeatureA/
│   │   ├── FeatureACore.swift
│   │   └── FeatureAView.swift
│   └── FeatureB/
├── Shared/                 # 共享组件
│   ├── Models/
│   ├── Dependencies/
│   └── Extensions/
└── Tests/                  # 测试
```

### 步骤 2: 基于 Todos 模板创建功能

#### 2.1 复制并修改基础结构
从 Todos 项目复制以下模板：

**应用入口模板 (App.swift):**
```swift
import ComposableArchitecture
import SwiftUI

@main
struct YourApp: App {
  static let store = Store(initialState: AppFeature.State()) {
    AppFeature()
      ._printChanges()
  }

  var body: some Scene {
    WindowGroup {
      AppView(store: Self.store)
    }
  }
}
```

**单项模板 (Item.swift):**
```swift
import ComposableArchitecture
import SwiftUI

@Reducer
struct Item {
  @ObservableState
  struct State: Equatable, Identifiable {
    var title = ""
    let id: UUID
    var isSelected = false
    // 根据需求添加其他属性
  }

  enum Action: BindableAction, Sendable {
    case binding(BindingAction<State>)
    // 添加自定义 Action
  }

  var body: some Reducer<State, Action> {
    BindingReducer()
    Reduce { state, action in
      switch action {
      case .binding:
        return .none
      // 处理自定义 Action
      }
    }
  }
}

struct ItemView: View {
  @Bindable var store: StoreOf<Item>

  var body: some View {
    HStack {
      // 基于 TodoView 修改 UI
      TextField("Title", text: $store.title)
    }
  }
}
```

**列表管理模板 (ItemList.swift):**
```swift
import ComposableArchitecture
import SwiftUI

@Reducer
struct ItemList {
  @ObservableState
  struct State: Equatable {
    var items: IdentifiedArrayOf<Item.State> = []
    var filter: Filter = .all
    // 添加其他状态
  }

  enum Action: BindableAction, Sendable {
    case addItemButtonTapped
    case binding(BindingAction<State>)
    case delete(IndexSet)
    case items(IdentifiedActionOf<Item>)
    // 添加其他 Action
  }

  @Dependency(\.uuid) var uuid

  var body: some Reducer<State, Action> {
    BindingReducer()
    Reduce { state, action in
      switch action {
      case .addItemButtonTapped:
        state.items.insert(Item.State(id: uuid()), at: 0)
        return .none
      // 实现其他逻辑
      }
    }
    .forEach(\.items, action: \.items) {
      Item()
    }
  }
}
```

### 步骤 3: 自定义业务逻辑

#### 3.1 定义领域模型
```swift
// Models/YourModel.swift
struct YourModel: Equatable, Identifiable, Codable {
  let id: UUID
  var title: String
  var description: String
  var createdAt: Date
  var status: Status
  
  enum Status: String, CaseIterable, Codable {
    case active, completed, archived
  }
}
```

#### 3.2 添加依赖服务
```swift
// Dependencies/YourService.swift
import Dependencies

struct YourService {
  var fetch: () async throws -> [YourModel]
  var save: (YourModel) async throws -> Void
  var delete: (UUID) async throws -> Void
}

extension YourService: DependencyKey {
  static let liveValue = YourService(
    fetch: {
      // 实际 API 调用
    },
    save: { model in
      // 保存逻辑
    },
    delete: { id in
      // 删除逻辑
    }
  )
}

extension DependencyValues {
  var yourService: YourService {
    get { self[YourService.self] }
    set { self[YourService.self] = newValue }
  }
}
```

#### 3.3 集成异步操作
```swift
@Reducer
struct YourFeature {
  @Dependency(\.yourService) var service
  
  var body: some Reducer<State, Action> {
    Reduce { state, action in
      switch action {
      case .loadData:
        return .run { send in
          do {
            let data = try await service.fetch()
            await send(.dataLoaded(data))
          } catch {
            await send(.loadFailed(error))
          }
        }
      }
    }
  }
}
```

### 步骤 4: 测试实现

#### 4.1 基于 Todos 测试模板
```swift
import ComposableArchitecture
import Testing

@testable import YourApp

@MainActor
struct YourFeatureTests {
  @Test
  func addItem() async {
    let store = TestStore(initialState: YourFeature.State()) {
      YourFeature()
    } withDependencies: {
      $0.uuid = .incrementing
      $0.yourService.save = { _ in }
    }

    await store.send(.addItemButtonTapped) {
      $0.items.insert(
        Item.State(id: UUID(0), title: ""),
        at: 0
      )
    }
  }
}
```

### 步骤 5: 模块化架构 (高级)

#### 5.1 参考 TicTacToe 的模块化结构
```swift
// Package.swift
let package = Package(
  name: "your-app-modules",
  products: [
    .library(name: "AppCore", targets: ["AppCore"]),
    .library(name: "FeatureA", targets: ["FeatureA"]),
    .library(name: "FeatureB", targets: ["FeatureB"]),
  ],
  dependencies: [
    .package(url: "https://github.com/pointfreeco/swift-composable-architecture", from: "1.17.0")
  ],
  targets: [
    .target(
      name: "AppCore",
      dependencies: [
        "FeatureA",
        "FeatureB",
        .product(name: "ComposableArchitecture", package: "swift-composable-architecture")
      ]
    ),
    .target(
      name: "FeatureA",
      dependencies: [
        .product(name: "ComposableArchitecture", package: "swift-composable-architecture")
      ]
    )
  ]
)
```

## 最佳实践总结

### 1. 从简单开始
- 先实现基本的 CRUD 操作
- 逐步添加复杂的业务逻辑
- 参考 Todos 的简洁设计

### 2. 合理使用 TCA 特性
- `@ObservableState` 用于状态观察
- `BindingReducer` 处理表单绑定
- `@Dependency` 管理外部依赖
- `forEach` 组合子 Reducer

### 3. 测试驱动开发
- 为每个功能编写测试
- 使用 `TestStore` 验证状态变化
- 模拟依赖进行单元测试

### 4. 渐进式架构
- 单体应用 → 功能模块 → 独立包
- 根据项目规模选择合适的架构
- 保持代码的可测试性和可维护性

## 实际开发示例

### 示例：构建一个笔记应用

基于 Todos 模板，我们来构建一个笔记管理应用：

#### 1. 定义笔记模型
```swift
// Models/Note.swift
import Foundation

struct Note: Equatable, Identifiable, Codable {
  let id: UUID
  var title: String
  var content: String
  var createdAt: Date
  var updatedAt: Date
  var category: Category

  enum Category: String, CaseIterable, Codable {
    case personal = "Personal"
    case work = "Work"
    case ideas = "Ideas"
  }
}
```

#### 2. 单个笔记 Reducer
```swift
// Features/Note/NoteCore.swift
import ComposableArchitecture

@Reducer
struct NoteFeature {
  @ObservableState
  struct State: Equatable, Identifiable {
    var note: Note
    var isEditing = false

    var id: UUID { note.id }
  }

  enum Action: BindableAction, Sendable {
    case binding(BindingAction<State>)
    case editButtonTapped
    case saveButtonTapped
    case cancelButtonTapped
  }

  var body: some Reducer<State, Action> {
    BindingReducer()
    Reduce { state, action in
      switch action {
      case .binding:
        if state.isEditing {
          state.note.updatedAt = Date()
        }
        return .none

      case .editButtonTapped:
        state.isEditing = true
        return .none

      case .saveButtonTapped:
        state.isEditing = false
        state.note.updatedAt = Date()
        return .none

      case .cancelButtonTapped:
        state.isEditing = false
        return .none
      }
    }
  }
}
```

#### 3. 笔记列表 Reducer
```swift
// Features/Notes/NotesCore.swift
import ComposableArchitecture

@Reducer
struct NotesFeature {
  @ObservableState
  struct State: Equatable {
    var notes: IdentifiedArrayOf<NoteFeature.State> = []
    var filter: Note.Category? = nil
    var searchText = ""
    var isLoading = false

    var filteredNotes: IdentifiedArrayOf<NoteFeature.State> {
      var filtered = notes

      if let filter = filter {
        filtered = IdentifiedArrayOf(filtered.filter { $0.note.category == filter })
      }

      if !searchText.isEmpty {
        filtered = IdentifiedArrayOf(filtered.filter {
          $0.note.title.localizedCaseInsensitiveContains(searchText) ||
          $0.note.content.localizedCaseInsensitiveContains(searchText)
        })
      }

      return filtered.sorted { $0.note.updatedAt > $1.note.updatedAt }
    }
  }

  enum Action: BindableAction, Sendable {
    case addNoteButtonTapped
    case binding(BindingAction<State>)
    case delete(IndexSet)
    case loadNotes
    case notesLoaded([Note])
    case notes(IdentifiedActionOf<NoteFeature>)
  }

  @Dependency(\.uuid) var uuid
  @Dependency(\.date) var date
  @Dependency(\.notesService) var notesService

  var body: some Reducer<State, Action> {
    BindingReducer()
    Reduce { state, action in
      switch action {
      case .addNoteButtonTapped:
        let newNote = Note(
          id: uuid(),
          title: "New Note",
          content: "",
          createdAt: date(),
          updatedAt: date(),
          category: .personal
        )
        state.notes.insert(
          NoteFeature.State(note: newNote, isEditing: true),
          at: 0
        )
        return .none

      case .binding:
        return .none

      case let .delete(indexSet):
        let filteredNotes = state.filteredNotes
        for index in indexSet {
          state.notes.remove(id: filteredNotes[index].id)
        }
        return .none

      case .loadNotes:
        state.isLoading = true
        return .run { send in
          let notes = try await notesService.fetchNotes()
          await send(.notesLoaded(notes))
        }

      case let .notesLoaded(notes):
        state.isLoading = false
        state.notes = IdentifiedArrayOf(
          notes.map { NoteFeature.State(note: $0) }
        )
        return .none

      case .notes:
        return .none
      }
    }
    .forEach(\.notes, action: \.notes) {
      NoteFeature()
    }
  }
}
```

#### 4. SwiftUI 视图实现
```swift
// Features/Notes/NotesView.swift
import SwiftUI
import ComposableArchitecture

struct NotesView: View {
  @Bindable var store: StoreOf<NotesFeature>

  var body: some View {
    NavigationStack {
      VStack {
        // 搜索栏
        SearchBar(text: $store.searchText)

        // 分类过滤器
        CategoryFilter(selection: $store.filter)

        // 笔记列表
        if store.isLoading {
          ProgressView("Loading notes...")
        } else {
          List {
            ForEach(store.scope(state: \.filteredNotes, action: \.notes)) { noteStore in
              NoteRowView(store: noteStore)
            }
            .onDelete { store.send(.delete($0)) }
          }
        }
      }
      .navigationTitle("Notes")
      .toolbar {
        ToolbarItem(placement: .navigationBarTrailing) {
          Button("Add Note") {
            store.send(.addNoteButtonTapped)
          }
        }
      }
      .onAppear {
        store.send(.loadNotes)
      }
    }
  }
}

struct NoteRowView: View {
  @Bindable var store: StoreOf<NoteFeature>

  var body: some View {
    VStack(alignment: .leading, spacing: 4) {
      if store.isEditing {
        TextField("Title", text: $store.note.title)
          .font(.headline)
        TextEditor(text: $store.note.content)
          .frame(minHeight: 60)

        HStack {
          Button("Save") { store.send(.saveButtonTapped) }
          Button("Cancel") { store.send(.cancelButtonTapped) }
        }
      } else {
        Text(store.note.title)
          .font(.headline)
        Text(store.note.content)
          .lineLimit(3)
          .foregroundColor(.secondary)

        HStack {
          Text(store.note.category.rawValue)
            .font(.caption)
            .padding(.horizontal, 8)
            .padding(.vertical, 2)
            .background(Color.blue.opacity(0.2))
            .cornerRadius(4)

          Spacer()

          Text(store.note.updatedAt, style: .relative)
            .font(.caption)
            .foregroundColor(.secondary)
        }
      }
    }
    .contentShape(Rectangle())
    .onTapGesture {
      if !store.isEditing {
        store.send(.editButtonTapped)
      }
    }
  }
}
```

### 常见问题和解决方案

#### 1. 状态同步问题
**问题**: 子 Reducer 状态变化不能及时反映到父 Reducer

**解决方案**: 使用 `forEach` 正确组合 Reducer
```swift
.forEach(\.items, action: \.items) {
  ItemFeature()
}
```

#### 2. 内存泄漏
**问题**: 长时间运行的 Effect 导致内存泄漏

**解决方案**: 使用 `cancellable` 管理 Effect 生命周期
```swift
return .run { send in
  // 长时间运行的任务
}
.cancellable(id: CancelID.longRunningTask, cancelInFlight: true)
```

#### 3. 测试异步操作
**问题**: 异步操作难以测试

**解决方案**: 使用 `TestClock` 控制时间
```swift
let clock = TestClock()
let store = TestStore(initialState: Feature.State()) {
  Feature()
} withDependencies: {
  $0.continuousClock = clock
}

await store.send(.startTimer)
await clock.advance(by: .seconds(1))
await store.receive(.timerTicked)
```

## 部署和发布

### 1. 性能优化
- 使用 `@ObservableState` 减少不必要的重绘
- 合理使用 `Equatable` 优化状态比较
- 避免在 Reducer 中进行重计算

### 2. 生产环境配置
```swift
@main
struct YourApp: App {
  static let store = Store(initialState: AppFeature.State()) {
    AppFeature()
    #if DEBUG
      ._printChanges()
    #endif
  } withDependencies: {
    #if DEBUG
      // 开发环境依赖
    #else
      // 生产环境依赖
    #endif
  }
}
```

### 3. 错误处理和日志
```swift
@Reducer
struct AppFeature {
  var body: some Reducer<State, Action> {
    Reduce { state, action in
      // 全局错误处理
    }
    .ifLet(\.errorState, action: \.error) {
      ErrorFeature()
    }
  }
}
```

## 总结

通过遵循这个详细指南，您可以：

1. **快速上手**: 基于成熟的 Todos 模板开始开发
2. **渐进式开发**: 从简单功能逐步扩展到复杂应用
3. **最佳实践**: 遵循 TCA 的设计原则和模式
4. **测试保障**: 构建可靠的测试套件
5. **生产就绪**: 处理实际部署中的各种问题

TCA 提供了一个强大而灵活的架构框架，通过合理使用其特性，可以构建出结构清晰、易于测试和维护的 Swift 应用程序。
