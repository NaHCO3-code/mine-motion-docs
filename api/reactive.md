# 响应式系统

Mine Motion 提供了一个轻量级的响应式系统，用于创建数据绑定和自动更新。这个系统是 Mine Motion 动态动画的核心，使动画能够响应数据变化。

## 核心概念

响应式系统的核心概念是追踪数据依赖和自动触发更新。它基于以下主要组件：

- **响应式对象(Ref)**: 可观察的值容器
- **计算属性(Computed)**: 依赖于响应式数据的派生值
- **监听器(Watch)**: 当响应式数据变化时执行回调函数

## 创建响应式对象

### ref&lt;T&gt;

`ref`函数用于创建一个响应式对象，包装任意类型的值：

```typescript
import { ref } from 'mine-motion';

// 创建一个响应式数字
const count = ref(0);

// 读取值
console.log(count.value); // 0

// 修改值
count.value = 1; // 自动触发依赖更新
```

#### 参数

| 参数 | 类型 | 描述 |
| ---- | ---- | ---- |
| value | T | 响应式对象的初始值 |

#### 返回值

| 返回值 | 类型 | 描述 |
| ------ | ---- | ---- |
| ref | Ref&lt;T&gt; | 包含`.value`属性的响应式对象 |

## 派生计算属性

### computed&lt;T&gt;

`computed`函数创建一个只读的派生状态，其值自动根据依赖的响应式数据更新：

```typescript
import { ref, computed } from 'mine-motion';

const width = ref(100);
const height = ref(200);

// 创建计算属性
const area = computed(() => width.value * height.value);

console.log(area.value); // 20000

// 当width或height变化时，area会自动更新
width.value = 150;
console.log(area.value); // 30000
```

#### 参数

| 参数 | 类型 | 描述 |
| ---- | ---- | ---- |
| update | () => T | 计算函数，内部可使用响应式数据 |

#### 返回值

| 返回值 | 类型 | 描述 |
| ------ | ---- | ---- |
| computed | Computed&lt;T&gt; | 包含只读`.value`属性的计算属性 |

## 监听数据变化

### watch&lt;T&gt;

`watch`函数用于观察响应式数据的变化并执行回调：

```typescript
import { ref, watch } from 'mine-motion';

const position = ref(0);

// 监听position变化
watch(
  () => position.value,
  (newValue) => {
    console.log(`位置更新为: ${newValue}`);
  }
);

// 触发回调
position.value = 100; // 控制台输出: "位置更新为: 100"
```

#### 参数

| 参数 | 类型 | 描述 |
| ---- | ---- | ---- |
| update | () => T | 返回要监听的值的函数 |
| callback | (value: T) => void | 值变化时执行的回调函数 |

## 类型定义

### Ref&lt;T&gt;

响应式对象的类型定义：

```typescript
type Ref<T> = {
  value: T;
}
```

### Computed&lt;T&gt;

计算属性的类型定义：

```typescript
type Computed<T> = {
  readonly value: T;
}
```

## 实际应用

### 与动画结合

响应式系统最强大的用例是与动画结合，创建数据驱动的动画：

```typescript
import { ref, MineTimeline, MDataDriver } from 'mine-motion';

// 创建响应式数据源
const progress = ref(0);

// 创建数据驱动的时间轴
const timeline = new MineTimeline({
  driver: new MDataDriver(progress)
});

// 添加动画
timeline.animate(element, [
  { value: { opacity: 0, x: -100 } },
  { value: { opacity: 1, x: 0 } }
]);

// 通过更新数据控制动画
progress.value = 0.5; // 动画移动到50%位置
```