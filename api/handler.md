# Handler

## MineHandler&lt;T&gt;

`MineHandler` 是 Mine Motion 的核心组件，负责处理单个属性从起始值到结束值的动画过渡。它是一个抽象类，不同类型的数据需要通过不同的 Handler 实现来处理。

### 何时使用Handler

在大多数情况下，你不需要直接创建或操作 Handler 对象，因为 Mine Motion 会自动为你创建合适的Handler。但了解 Handler 的工作原理有助于：

1. 理解 Mine Motion 如何处理动画
2. 创建自定义的特殊动画效果
3. 扩展 Mine Motion 支持新的数据类型

### 核心属性

| 属性 | 类型 | 描述 |
| --- | --- | --- |
| start | any | 动画的起始值 |
| end | any | 动画的结束值 |
| duration | number | 动画的持续时间（毫秒） |
| setter | [Setter](/api/types.html#setter-lt-t-gt)&lt;T&gt; | 用于更新属性当前值的函数 |
| ease | [EaseFunc](/api/types.html#easefunc) | 控制动画变化曲线的函数 |

### 核心方法

#### seek(time: number): void

`seek` 方法是 Handler 的核心，它根据当前时间计算属性应该的值，并通过 setter 更新属性。

- **time**: 相对于动画开始时间的毫秒数（0到duration之间）
- 当 time = 0 时，属性值为start
- 当 time = duration 时，属性值为end
- 中间值由 ease 函数控制过渡曲线

### 如何工作

每个 Handler 实例负责一个具体属性的动画。当 Timeline 的时间更新时，它会调用所有 Handler 的 seek 方法，传入相对于 Handler 开始时间的当前时间。Handler 会：

1. 根据 ease 函数计算当前进度（0到1之间）
2. 根据进度插值计算当前值
3. 调用 setter 函数更新属性值

### 实现示例

下面是 `MNumberHandler`（处理数字类型）的简化实现：

```typescript
class MNumberHandler extends MineHandler<number> {
  seek(time: number) {
    // 计算进度（0-1之间）
    const progress = Math.min(Math.max(time / this.duration, 0), 1);
    
    // 应用缓动函数
    const eased = this.ease(progress);
    
    // 计算当前值并更新
    const value = this.start + (this.end - this.start) * eased;
    this.setter(value);
  }
}
```

### 自定义Handler

如果你需要创建自定义Handler，需要继承 `MineHandler` 并实现 `seek` 方法。然后通过创建相应的 Plugin 使 Mine Motion 能够自动使用你的 Handler。