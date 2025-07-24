# Swift Composable Architecture (TCA) 使用指南

## 项目概览

Swift Composable Architecture 是由 Point-Free 开发的 Swift 状态管理架构库，专门用于构建一致性、可测试的应用程序。

### 核心特性
- 🏗️ **单向数据流**: 可预测的状态管理
- 🧪 **高度可测试**: 内置测试工具和模式
- 🔧 **模块化设计**: 易于组合和重用的组件
- 📱 **跨平台支持**: iOS、macOS、tvOS、watchOS
- ⚡ **性能优化**: 高效的状态更新机制

## 环境要求

### 必需工具
- **Xcode 15+** (完整版，不是命令行工具)
- **Swift 5.9+**
- **平台支持**:
  - iOS 13+
  - macOS 10.15+
  - tvOS 13+
  - watchOS 6+

### 验证环境
```bash
# 检查 Swift 版本
swift --version

# 检查 Xcode 版本
xcodebuild -version
```

## 项目结构

```
swift-composable-architecture/
├── Sources/ComposableArchitecture/     # 核心库代码
├── Examples/                           # 示例应用
│   ├── Todos/                         # 简单待办事项应用
│   ├── SyncUps/                       # 会议管理应用
│   ├── TicTacToe/                     # 井字游戏
│   ├── VoiceMemos/                    # 语音备忘录
│   ├── Search/                        # 搜索功能演示
│   └── CaseStudies/                   # 各种用例研究
├── Tests/                             # 测试套件
├── ComposableArchitecture.xcworkspace # 主工作空间
└── Package.swift                      # Swift Package 配置
```

## 快速开始

### 1. 打开示例项目

#### 方法一：打开单个示例（推荐新手）
```bash
# 打开 Todos 示例
open Examples/Todos/Todos.xcodeproj
```

#### 方法二：打开完整工作空间
```bash
# 打开主工作空间（包含所有示例）
open ComposableArchitecture.xcworkspace
```

### 2. 运行示例应用

1. **在 Xcode 中**：
   - 选择目标 scheme（如 "Todos"）
   - 选择模拟器（如 iPhone 15）
   - 点击运行按钮（▶️）或按 `Cmd + R`

2. **验证功能**：
   - ✅ 添加新的 todo 项目
   - ✅ 编辑 todo 内容
   - ✅ 标记完成/未完成状态
   - ✅ 删除 todo 项目
   - ✅ 使用过滤器（All/Active/Completed）
   - ✅ 自动排序完成的项目

### 3. 运行测试

```bash
# 在 Xcode 中运行测试
# 按 Cmd + U 或选择 Product → Test
```

## TCA 核心概念

### 1. 应用入口点
```swift
@main
struct TodosApp: App {
  static let store = Store(initialState: Todos.State()) {
    Todos()
      ._printChanges()
  }

  var body: some Scene {
    WindowGroup {
      AppView(store: Self.store)
    }
  }
}
```

### 2. Reducer（状态管理器）
```swift
@Reducer
struct Todos {
  @ObservableState
  struct State: Equatable {
    var editMode: EditMode = .inactive
    var filter: Filter = .all
    var todos: IdentifiedArrayOf<Todo.State> = []
  }

  enum Action: BindableAction, Sendable {
    case addTodoButtonTapped
    case binding(BindingAction<State>)
    case clearCompletedButtonTapped
    // ... 其他 actions
  }

  var body: some Reducer<State, Action> {
    BindingReducer()
    Reduce { state, action in
      switch action {
      case .addTodoButtonTapped:
        state.todos.insert(Todo.State(id: self.uuid()), at: 0)
        return .none
      // ... 处理其他 actions
      }
    }
  }
}
```

### 3. 测试
```swift
@Test
func add() async {
  let store = TestStore(initialState: Todos.State()) {
    Todos()
  } withDependencies: {
    $0.uuid = .incrementing
  }

  await store.send(.addTodoButtonTapped) {
    $0.todos.insert(
      Todo.State(description: "", id: UUID(0), isComplete: false),
      at: 0
    )
  }
}
```

## 创建新项目

### 方法一：基于示例修改（推荐）

1. **选择合适的示例**：
   - **Todos**: 简单 CRUD 应用
   - **SyncUps**: 复杂业务应用
   - **Search**: 搜索功能应用
   - **VoiceMemos**: 媒体处理应用

2. **复制并修改**：
```bash
# 复制示例项目
cp -r Examples/Todos MyNewApp
cd MyNewApp

# 重命名项目文件
# 修改 Bundle Identifier
# 更新项目名称
```

### 方法二：从零开始

1. **创建新 Xcode 项目**
2. **添加 TCA 依赖**：
```swift
// Package.swift
dependencies: [
    .package(
        url: "https://github.com/pointfreeco/swift-composable-architecture", 
        from: "1.0.0"
    )
],
targets: [
    .target(
        name: "YourApp",
        dependencies: [
            .product(name: "ComposableArchitecture", package: "swift-composable-architecture")
        ]
    )
]
```

## 开发最佳实践

### 1. 项目结构建议
```
YourApp/
├── App/
│   ├── AppDelegate.swift
│   └── YourApp.swift
├── Features/
│   ├── FeatureA/
│   │   ├── FeatureACore.swift      # Reducer
│   │   ├── FeatureAView.swift      # SwiftUI View
│   │   └── FeatureATests.swift     # 测试
│   └── FeatureB/
├── Shared/
│   ├── Models/
│   ├── Services/
│   └── Extensions/
└── Resources/
```

### 2. 开发流程
1. **定义 State**: 描述功能的数据结构
2. **定义 Action**: 列出所有可能的用户操作
3. **实现 Reducer**: 处理状态变化逻辑
4. **创建 View**: 构建用户界面
5. **编写测试**: 验证功能正确性

### 3. 测试策略
- **单元测试**: 使用 `TestStore` 测试状态变化
- **集成测试**: 测试模块间协作
- **UI 测试**: 验证用户交互流程
- **性能测试**: 监控状态更新性能

## 常见问题解决

### 构建问题
```bash
# 清理构建缓存
# Product → Clean Build Folder (Cmd + Shift + K)

# 重置 Package 缓存
# File → Packages → Reset Package Caches
```

### 依赖问题
- 确保网络连接正常
- 等待 Swift Package 依赖下载完成
- 检查 Package.swift 中的版本约束

### 模拟器问题
- 选择合适的 iOS 模拟器版本
- 重启 Xcode 和模拟器
- 检查模拟器设置

## 学习资源

### 官方资源
- 📚 [官方文档](Sources/ComposableArchitecture/Documentation.docc/)
- 🎥 [Point-Free 视频教程](https://www.pointfree.co/collections/composable-architecture)
- 💻 [GitHub 仓库](https://github.com/pointfreeco/swift-composable-architecture)

### 学习路径
1. **基础概念**: 理解 State、Action、Reducer
2. **简单示例**: 从 Todos 开始学习
3. **复杂应用**: 研究 SyncUps 和其他示例
4. **实践项目**: 创建自己的应用
5. **高级特性**: 学习 Effects、Navigation 等

### 推荐示例学习顺序
1. **Todos** - 基础 CRUD 操作
2. **Search** - 异步操作和防抖
3. **SyncUps** - 复杂业务逻辑
4. **VoiceMemos** - 媒体处理
5. **CaseStudies** - 各种设计模式

## 总结

TCA 提供了一个强大而灵活的架构模式，特别适合构建复杂的、可测试的 iOS 应用。通过学习示例项目和遵循最佳实践，你可以快速掌握这个架构并应用到自己的项目中。

记住：从简单开始，逐步掌握复杂特性，始终保持代码的可测试性和模块化。
