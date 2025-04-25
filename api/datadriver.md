# 数据驱动动画

Mine Motion 不仅支持基于时间的动画，还提供了强大的数据驱动功能。数据驱动允许你将动画进度与任何数值型数据源绑定，使动画能够响应用户交互、滚动位置或其他数据变化。

## MDataDriver 类

`MDataDriver` 是 Mine Motion 的数据驱动器，它接收一个响应式数值，并将其映射为动画的进度。

### 创建数据驱动器

```typescript
import { ref, MineTimeline, MDataDriver } from 'mine-motion';

// 创建响应式数据源(0到1之间)
const progress = ref(0);

// 创建数据驱动的时间轴
const timeline = new MineTimeline({
  driver: new MDataDriver(progress),
  autoStop: false // 数据驱动通常不需要自动停止
});

// 添加动画
timeline.animate(element, [
  { value: { opacity: 0, x: -100 } },
  { value: { opacity: 1, x: 0 } }
]);

// 控制动画进度
progress.value = 0.5; // 动画移动到50%位置
```

### 构造函数

```typescript
new MDataDriver(data, config?)
```

#### 参数

| 参数 | 类型 | 描述 |
| --- | --- | --- |
| data | Ref&lt;number&gt; | 响应式数值数据源 |
| config? | MDataDriverConfig | 可选的配置对象 |

### 配置选项

`MDataDriverConfig` 包含以下选项：

| 属性 | 类型 | 默认值 | 描述 |
| --- | --- | --- | --- |
| scale? | number | 1 | 数据值到时间的缩放比例 |
| damping? | MDataDriverDampingConfig | {enabled: false} | 平滑过渡的阻尼配置 |

### 阻尼配置

`MDataDriverDampingConfig` 用于平滑数据变化，防止动画过于生硬：

| 属性 | 类型 | 默认值 | 描述 |
| --- | --- | --- | --- |
| enabled | boolean | false | 是否启用阻尼效果 |
| halflife | number | 0 | 阻尼半衰期(数值越大过渡越平滑) |
| deltaMs | number | 7 | 阻尼更新间隔(毫秒) |

## 使用场景

### 数据映射

你可以使用`scale`参数调整数据与时间的映射关系：

```typescript
// 将0-100的数据映射到0-1的动画进度
const driver = new MDataDriver(largeValue, {
  scale: 0.01 // 除以100
});
```

### 平滑过渡

对于快速变化的数据，使用阻尼效果创建更平滑的动画：

```typescript
const driver = new MDataDriver(jumpyData, {
  damping: {
    enabled: true,
    halflife: 0.2, // 数值越大过渡越平滑
    deltaMs: 16    // 约60fps的更新频率
  }
});
```