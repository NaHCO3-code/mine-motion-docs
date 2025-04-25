# Timeline 时间轴

时间轴是 Mine Motion 中最重要的组件，它负责管理和协调多个动画的播放。`MineTimeline` 类提供了丰富的 API 来创建、控制和管理动画序列。

## 创建时间轴

### 构造函数

```typescript
new MineTimeline(config?: MTimelineConfig)
```

通过创建 `MineTimeline` 实例来开始使用 Mine Motion：

```typescript
// 创建默认时间轴
const timeline = new MineTimeline();

// 创建带配置的时间轴
const timeline = new MineTimeline({
  autoStop: true,
  onFinish: () => console.log('动画完成！'),
  onStart: () => console.log('动画开始！')
});
```

### 配置选项

| 配置项 | 类型 | 默认值 | 描述 |
| ----- | ---- | ----- | ---- |
| autoStop | boolean | 基于时间驱动时为true | 所有动画完成后是否自动停止 |
| onFinish | () => void | undefined | 动画完成时的回调函数 |
| onStart | () => void | undefined | 动画开始时的回调函数 |
| driver | MotionDriver | MDefaultDriver | 时间轴的驱动器，默认基于时间 |

## 添加动画

### animate&lt;T&gt;

`animate` 方法是创建动画的主要方式，它通过关键帧定义目标对象属性的变化：

```typescript
timeline.animate(object, [
  { value: { x: 0, opacity: 0 }, duration: 1000, easing: MineEases.easeOut },
  { value: { x: 100, opacity: 1 }, duration: 500, easing: MineEases.easeIn }
], { offset: 0 });
```

#### 参数

| 参数 | 类型 | 描述 |
| ---- | ---- | ---- |
| obj | T | 要动画的对象 |
| keyframes | MKeyframe&lt;T&gt;[] | 关键帧数组 |
| config | MAnimationConfig | 动画配置 |

#### 关键帧属性

| 属性 | 类型 | 默认值 | 描述 |
| ---- | ---- | ----- | ---- |
| value | Partial&lt;T&gt; | (必填) | 关键帧的目标值 |
| duration | number | 1000 | 从上一帧到当前帧的持续时间(毫秒) |
| easing | EaseFunc | MineEases.linear | 缓动函数 |

### to&lt;T, K&gt;

`to` 方法为对象的单个属性创建动画，从当前值到目标值：

```typescript
// 将myObject.x从当前值动画到100，持续1秒
timeline.to(myObject, 'x', {
  end: 100,
  duration: 1000,
  ease: MineEases.easeInOut
});
```

### fromTo&lt;T, K&gt;

`fromTo` 方法为对象的单个属性创建动画，从指定的起始值到目标值：

```typescript
// 将myObject.opacity从0动画到1，持续500毫秒
timeline.fromTo(myObject, 'opacity', {
  start: 0,
  end: 1,
  duration: 500,
  ease: MineEases.easeOut
});
```

### applyHandler

如果你需要更精细的控制，可以通过 `applyHandler` 方法直接将一个 `MineHandler` 添加到时间轴：

```typescript
const handler = new MyCustomHandler({...});
timeline.applyHandler(handler, 200); // 在200ms位置添加handler
```

## 控制动画

### run(reset?: boolean)

启动时间轴动画：

```typescript
// 从当前位置开始播放
timeline.run(false);

// 从开头重新播放
timeline.run(true);
```

### pause()

暂停时间轴动画：

```typescript
timeline.pause();
```

### seek(time: number)

跳转到时间轴的特定时间点：

```typescript
// 跳转到500ms位置
timeline.seek(500);
```

## 属性

### speed

控制动画播放速度的倍数，默认为1：

```typescript
// 设置为两倍速播放
timeline.speed = 2;

// 设置为半速播放
timeline.speed = 0.5;
```

### now

获取当前时间轴的时间位置（只读）：

```typescript
console.log(timeline.now); // 当前时间点（毫秒）
```

### duration

获取时间轴的总持续时间（只读）：

```typescript
console.log(timeline.duration); // 所有动画的总持续时间
```

### running

检查时间轴是否正在运行（只读）：

```typescript
if (timeline.running) {
  console.log('动画正在播放');
}
```

## 最佳实践

1. **预先创建时间轴**：在性能敏感的应用中，提前创建和配置时间轴。
2. **组合多个动画**：使用一个时间轴控制多个相关动画，而不是创建多个时间轴。
3. **重用时间轴**：调用`seek(0)`和`run()`可以重播动画，而不需要重新创建时间轴。