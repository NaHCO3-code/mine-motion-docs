# Plugin 插件系统

Mine Motion 的插件系统允许库轻松地扩展以支持各种数据类型的动画。插件是连接 Timeline 和 Handler 的桥梁，负责为不同类型的数据创建合适的 Handler。

## MinePlugin

`MinePlugin` 是所有插件的基类，用于创建能处理特定数据类型动画的组件。

### 核心概念

每个 `MinePlugin` 都实现了一个 `handle` 方法，该方法尝试根据提供的配置创建适当的 `MineHandler`。插件系统遵循"责任链"模式 - 当需要创建动画时，Mine Motion会依次尝试已注册的插件，直到找到能处理该数据类型的插件。

### 属性

| 属性 | 类型 | 描述 |
| --- | --- | --- |
| weight | number | 插件的优先级，数值越高越先被尝试 |

### 方法

#### handle(config: [MineHandlerConfig](/api/types.html#minehandlerconfig-lt-t-gt)&lt;any&gt;): [MineHandler](/api/handler) | [CanNotAnimateErr](/api/types.html#cannotanimateerr)

尝试创建一个Handler来处理指定的动画配置。

- **参数**: 包含动画起始值、结束值、持续时间等信息的配置对象
- **返回值**: 成功时返回一个Handler实例，失败时返回 `CanNotAnimateErr`（即`false`）

### 插件实现示例

下面是处理数字类型动画的 `MNumberPlugin` 的简化实现：

```typescript
class MNumberPlugin extends MinePlugin<number> {
  // 设置较高优先级
  weight = 10;
  
  handle(config) {
    // 检查是否能处理这种类型
    if (typeof config.start !== 'number' || typeof config.end !== 'number') {
      return false; // CanNotAnimateErr
    }
    
    // 创建并返回适当的Handler
    return new MNumberHandler(config);
  }
}
```

## MinePluginManager

`MinePluginManager` 是一个单例，负责管理所有注册的插件并为动画请求找到合适的Handler。

### 方法

#### register(plugin: [MinePlugin](/api/plugin#mineplugin))

注册一个新插件到管理器。

- **参数**: 要注册的插件实例

#### getHandler(config: [MineHandlerConfig](/api/types.html#minehandlerconfig-lt-t-gt)&lt;any&gt;): [MineHandler](/api/handler) | [CanNotAnimateErr](/api/types.html#cannotanimateerr)

尝试为指定的动画配置找到合适的Handler。

- **参数**: 包含动画配置信息的对象
- **返回值**: 成功时返回一个Handler实例，失败时返回 `CanNotAnimateErr`

### 使用示例

```typescript
// 注册自定义插件
class MyColorPlugin extends MinePlugin<Color> {
  handle(config) {
    if (config.start instanceof Color && config.end instanceof Color) {
      return new ColorHandler(config);
    }
    return false;
  }
}

// 注册到管理器
MinePluginManager.register(new MyColorPlugin());

// 现在Mine Motion可以为Color类型创建动画了
timeline.animate(myObject, [
  { value: { color: new Color(255, 0, 0) } },
  { value: { color: new Color(0, 0, 255) } }
]);
```

### 内置插件

Mine Motion内置了以下插件：

- `MNumberPlugin`: 处理数字类型动画（如位置、不透明度、大小等）

你可以根据需要创建自定义插件来扩展Mine Motion支持更多的数据类型，如颜色、路径、复杂对象等。