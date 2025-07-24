# Swift Composable Architecture (TCA) 项目分析

## 项目概述

Swift Composable Architecture (TCA) 是由 Point-Free 团队开发的一个用于构建 Swift 应用程序的架构库。它提供了一致且可理解的方式来构建应用程序，注重组合性、测试性和人体工程学。该库可以在 SwiftUI、UIKit 以及所有 Apple 平台（iOS、macOS、iPadOS、visionOS、tvOS 和 watchOS）上使用。

## 依赖管理

### 主要依赖项

项目通过 Swift Package Manager 管理依赖，主要依赖包括：

#### 核心依赖
- **swift-collections** (v1.2.0) - Apple 官方集合类型扩展
- **combine-schedulers** (v1.0.3) - Combine 调度器工具
- **swift-case-paths** (v1.7.1) - 枚举路径操作工具
- **swift-concurrency-extras** (v1.3.1) - Swift 并发扩展
- **swift-custom-dump** (v1.3.3) - 自定义调试输出
- **swift-dependencies** (v1.9.2) - 依赖注入系统
- **swift-identified-collections** (v1.1.1) - 标识集合类型
- **swift-navigation** (v2.3.1) - 导航工具（包含 SwiftUI 和 UIKit 导航）
- **swift-perception** (v1.6.0) - 观察和响应式编程
- **swift-sharing** (v2.5.2) - 状态共享工具
- **xctest-dynamic-overlay** (v1.5.2) - 测试工具

#### 开发和构建依赖
- **swift-macro-testing** (v0.6.3) - 宏测试工具
- **swift-docc-plugin** (v1.4.4) - 文档生成插件
- **swift-syntax** (v601.0.1) - Swift 语法分析（用于宏）
- **swift-snapshot-testing** (v1.18.4) - 快照测试

### 平台支持
- iOS 13.0+
- macOS 10.15+
- tvOS 13.0+
- watchOS 6.0+

## 目录结构详解

### 根目录结构
```
swift-composable-architecture/
├── Sources/                    # 源代码目录
│   ├── ComposableArchitecture/ # 主库源码
│   └── ComposableArchitectureMacros/ # 宏实现
├── Examples/                   # 示例项目
├── Tests/                      # 测试代码
├── Benchmarks/                 # 性能基准测试
├── Package.swift              # Swift Package 配置
├── Package.resolved           # 依赖锁定文件
└── README.md                  # 项目说明
```

### Sources 目录详解

#### ComposableArchitecture 核心模块
```
Sources/ComposableArchitecture/
├── Core.swift                 # 核心架构定义
├── Store.swift               # Store 实现
├── Reducer.swift             # Reducer 基础
├── Effect.swift              # 副作用处理
├── TestStore.swift           # 测试 Store
├── ViewStore.swift           # 视图 Store（已弃用）
├── Macros.swift              # 宏定义
├── SwiftUI/                  # SwiftUI 集成
│   ├── Binding.swift         # 数据绑定
│   ├── ForEachStore.swift    # 列表 Store
│   ├── IfLetStore.swift      # 条件 Store
│   ├── NavigationStackStore.swift # 导航栈
│   ├── Alert.swift           # 警告框
│   ├── Sheet.swift           # 模态视图
│   └── ...                   # 其他 SwiftUI 组件
├── UIKit/                    # UIKit 集成
├── Dependencies/             # 依赖管理
├── Effects/                  # 副作用工具
├── Observation/              # 观察机制
├── Reducer/                  # Reducer 工具
└── Sharing/                  # 状态共享
```

#### ComposableArchitectureMacros 宏模块
```
Sources/ComposableArchitectureMacros/
├── ReducerMacro.swift        # @Reducer 宏
├── ObservableStateMacro.swift # @ObservableState 宏
├── PresentsMacro.swift       # @Presents 宏
└── ViewActionMacro.swift     # @ViewAction 宏
```

### Examples 示例项目

项目包含丰富的示例，展示不同场景的使用方法：

```
Examples/
├── CaseStudies/              # 案例研究（SwiftUI + UIKit + tvOS）
├── Search/                   # 搜索功能示例
├── SpeechRecognition/        # 语音识别示例
├── SyncUps/                  # 会议管理应用（完整应用示例）
├── TicTacToe/               # 井字游戏（模块化示例）
├── Todos/                   # 待办事项应用
└── VoiceMemos/              # 语音备忘录应用
```

## SwiftUI 集成

### 是否使用 SwiftUI？
**是的**，TCA 对 SwiftUI 有深度集成和原生支持：

1. **专门的 SwiftUI 模块**：`Sources/ComposableArchitecture/SwiftUI/` 包含了完整的 SwiftUI 集成
2. **SwiftUI 特定组件**：
   - `ForEachStore` - 处理列表数据
   - `IfLetStore` - 条件渲染
   - `NavigationStackStore` - 导航管理
   - `SwitchStore` - 状态切换
   - 各种模态视图支持（Alert、Sheet、FullScreenCover 等）

3. **示例项目广泛使用 SwiftUI**：所有现代示例都基于 SwiftUI 构建

### SwiftUI 使用示例
```swift
import ComposableArchitecture
import SwiftUI

struct FeatureView: View {
  let store: StoreOf<Feature>

  var body: some View {
    Form {
      Section {
        Text("\(store.count)")
        Button("Increment") { store.send(.incrementButtonTapped) }
      }
    }
  }
}
```

## 核心架构概念

### 四大核心组件

1. **State（状态）**：描述功能所需的数据
2. **Action（动作）**：表示功能中可能发生的所有操作
3. **Reducer（归约器）**：描述如何根据动作更新状态
4. **Store（存储）**：驱动功能的运行时

### 关键特性

- **@Reducer 宏**：简化 Reducer 定义
- **@ObservableState 宏**：启用观察机制
- **@Presents 宏**：处理模态展示
- **依赖注入系统**：管理外部依赖
- **副作用处理**：统一的异步操作管理
- **测试支持**：完整的测试工具链

## 如何使用 TCA 开发新项目

### 步骤 1：项目设置

#### 通过 Xcode 添加依赖
1. 在 Xcode 中打开项目
2. 选择 **File** → **Add Package Dependencies...**
3. 输入 URL：`https://github.com/pointfreeco/swift-composable-architecture`
4. 选择版本并添加 **ComposableArchitecture** 到目标

#### 通过 Swift Package Manager
在 `Package.swift` 中添加：
```swift
dependencies: [
  .package(url: "https://github.com/pointfreeco/swift-composable-architecture", from: "1.17.0")
],
targets: [
  .target(
    name: "YourTarget",
    dependencies: [
      .product(name: "ComposableArchitecture", package: "swift-composable-architecture")
    ]
  )
]
```

### 步骤 2：创建基础功能

#### 定义 Reducer
```swift
import ComposableArchitecture

@Reducer
struct CounterFeature {
  @ObservableState
  struct State: Equatable {
    var count = 0
  }
  
  enum Action {
    case decrementButtonTapped
    case incrementButtonTapped
  }
  
  var body: some Reducer<State, Action> {
    Reduce { state, action in
      switch action {
      case .decrementButtonTapped:
        state.count -= 1
        return .none
      case .incrementButtonTapped:
        state.count += 1
        return .none
      }
    }
  }
}
```

#### 创建 SwiftUI 视图
```swift
import SwiftUI

struct CounterView: View {
  let store: StoreOf<CounterFeature>
  
  var body: some View {
    VStack {
      Text("\(store.count)")
      HStack {
        Button("-") { store.send(.decrementButtonTapped) }
        Button("+") { store.send(.incrementButtonTapped) }
      }
    }
  }
}
```

#### 设置应用入口
```swift
@main
struct MyApp: App {
  var body: some Scene {
    WindowGroup {
      CounterView(
        store: Store(initialState: CounterFeature.State()) {
          CounterFeature()
        }
      )
    }
  }
}
```

### 步骤 3：添加副作用和依赖

#### 定义依赖
```swift
struct APIClient {
  var fetchData: () async throws -> String
}

extension APIClient: DependencyKey {
  static let liveValue = APIClient(
    fetchData: {
      // 实际 API 调用
      return "Live data"
    }
  )
}

extension DependencyValues {
  var apiClient: APIClient {
    get { self[APIClient.self] }
    set { self[APIClient.self] = newValue }
  }
}
```

#### 在 Reducer 中使用依赖
```swift
@Reducer
struct DataFeature {
  @Dependency(\.apiClient) var apiClient
  
  // ... State 和 Action 定义
  
  var body: some Reducer<State, Action> {
    Reduce { state, action in
      switch action {
      case .loadButtonTapped:
        return .run { send in
          let data = try await apiClient.fetchData()
          await send(.dataLoaded(data))
        }
      // ...
      }
    }
  }
}
```

### 步骤 4：编写测试

```swift
import ComposableArchitecture
import XCTest

@Test
func counterFeature() async {
  let store = TestStore(initialState: CounterFeature.State()) {
    CounterFeature()
  }
  
  await store.send(.incrementButtonTapped) {
    $0.count = 1
  }
  
  await store.send(.decrementButtonTapped) {
    $0.count = 0
  }
}
```

### 步骤 5：模块化和组合

#### 创建子功能
```swift
@Reducer
struct AppFeature {
  @ObservableState
  struct State: Equatable {
    var counter = CounterFeature.State()
    var settings = SettingsFeature.State()
  }
  
  enum Action {
    case counter(CounterFeature.Action)
    case settings(SettingsFeature.Action)
  }
  
  var body: some Reducer<State, Action> {
    Scope(state: \.counter, action: \.counter) {
      CounterFeature()
    }
    Scope(state: \.settings, action: \.settings) {
      SettingsFeature()
    }
  }
}
```

## 最佳实践建议

1. **从简单开始**：先实现基本的 State-Action-Reducer 模式
2. **逐步添加复杂性**：根据需要添加副作用、导航、依赖注入
3. **充分利用示例**：参考 Examples 目录中的实际项目
4. **编写测试**：使用 TestStore 确保功能正确性
5. **模块化设计**：将大功能拆分为小的、可组合的模块
6. **依赖管理**：使用 TCA 的依赖注入系统管理外部依赖

## 学习资源

- **官方文档**：https://pointfreeco.github.io/swift-composable-architecture/
- **Point-Free 视频教程**：https://www.pointfree.co/collections/composable-architecture
- **示例项目**：项目 Examples 目录
- **社区讨论**：GitHub Discussions 和 Point-Free Slack

TCA 是一个功能强大且设计精良的架构框架，特别适合构建复杂的、可测试的 Swift 应用程序。通过遵循其核心原则和最佳实践，可以构建出结构清晰、易于维护的应用程序。
