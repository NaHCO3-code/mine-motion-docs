# 类型和常量

Mine Motion 内部使用了多种类型和常量来保证动画系统的类型安全和功能一致性。本文档解释了这些类型及其在API中的作用。

## 核心类型

### Setter&lt;T&gt;

`Setter`是一个函数类型，用于更新动画目标的属性值：

```typescript
type Setter<T> = (v: T) => void;
```

这个类型在`MineHandler`和其他需要更新属性值的地方使用。例如，数字属性的 setter 可能是：

```typescript
const setter: Setter<number> = (value) => {
  element.style.opacity = value;
};
```

### EaseFunc

`EaseFunc`定义了缓动函数的类型，它接收一个 0 到 1 之间的进度值，返回一个调整后的进度值，通常也在 0 至 1 之间。

```typescript
type EaseFunc = (x: number) => number;
```

缓动函数用于创建自然、平滑的动画过渡效果。例如：

```typescript
// 线性缓动
const linear: EaseFunc = (x) => x;

// 二次缓入
const easeIn: EaseFunc = (x) => x * x;
```

### CanNotAnimateErr

当插件无法处理特定类型的动画时返回的标识值，它是一个简单的 `false` 值：

```typescript
type CanNotAnimateErr = false;
const CanNotAnimateErr = false;
```

在插件系统中用于表示"我不能处理这种类型"，这样系统会尝试下一个插件。

### MineAnimatable

定义了可以被 Mine Motion 动画化的对象类型：

```typescript
type MineAnimatable = { [K: string | number]: any };
```

基本上任何带有属性的对象都可以被动画化，前提是有合适的插件来处理属性值类型。

## 配置类型

### MineHandlerConfig&lt;T&gt;

`MineHandlerConfig` 定义了创建 `MineHandler` 所需的配置：

```typescript
interface MineHandlerConfig<T> {
  setter: Setter<T>;   // 更新值的函数
  start: T;            // 起始值
  end: T;              // 结束值
  duration: number;    // 持续时间(毫秒)
  ease: EaseFunc;      // 缓动函数
}
```

这个配置类型被传递给插件的 `handle` 方法，用于创建处理特定数据类型的Handler。

### MTimelineConfig

`MTimelineConfig` 用于配置 `MineTimeline` 的行为：

```typescript
interface MTimelineConfig {
  autoStop?: boolean;        // 是否自动停止
  onFinish?: () => void;     // 结束回调
  onStart?: () => void;      // 开始回调
  driver?: MotionDriver;     // 驱动器
}
```

当创建新的 `MineTimeline` 实例时使用这个配置类型：

```typescript
const timeline = new MineTimeline({
  autoStop: true,
  onFinish: () => console.log('动画完成')
});
```

### MKeyframe&lt;T&gt;

`MKeyframe` 定义了动画关键帧的结构：

```typescript
interface MKeyframe<T extends MineAnimatable> {
  value: Partial<T>;      // 关键帧的目标值
  duration?: number;      // 持续时间(默认1000ms)
  easing?: EaseFunc;      // 缓动函数(默认线性)
}
```

在 `MineTimeline.animate` 方法中使用关键帧数组定义动画序列：

```typescript
timeline.animate(element, [
  { value: { x: 0, opacity: 0 } },
  { value: { x: 100, opacity: 1 }, duration: 500, easing: MineEases.easeOut }
]);
```

### MAnimationConfig

`MAnimationConfig` 用于配置 `animate` 方法的行为：

```typescript
interface MAnimationConfig {
  offset: number;    // 动画开始的时间偏移(毫秒)
}
```

这允许你控制动画在时间轴上的位置：

```typescript
// 在时间轴的200ms位置开始这个动画
timeline.animate(element, [...], { offset: 200 });
```

## 常量

### MineEases

`MineEases` 是一个包含常用缓动函数的对象：

```typescript
const MineEases = {
  linear: (r: number) => r,                          // 线性
  sine: (r: number) => Math.sin(r*Math.PI/2),        // 正弦
  easeInOut: (r: number) => 6*r**5 - 15*r**4 + 10*r**3, // 平滑过渡
  easeIn: (r: number) => r * r,                     // 缓入
  easeOut: (r: number) => r * (2-r)                 // 缓出
};
```

这些缓动函数可以直接在动画定义中使用：

```typescript
timeline.to(element, 'opacity', {
  end: 1,
  duration: 1000,
  ease: MineEases.easeInOut  // 使用平滑的缓入缓出效果
});
```

### 自定义缓动函数

你可以创建自己的缓动函数来实现特殊的动画效果：

```typescript
// 弹跳效果
const bounce: EaseFunc = (x) => {
  const n1 = 7.5625;
  const d1 = 2.75;
  
  if (x < 1 / d1) {
    return n1 * x * x;
  } else if (x < 2 / d1) {
    return n1 * (x -= 1.5 / d1) * x + 0.75;
  } else if (x < 2.5 / d1) {
    return n1 * (x -= 2.25 / d1) * x + 0.9375;
  } else {
    return n1 * (x -= 2.625 / d1) * x + 0.984375;
  }
};
```
