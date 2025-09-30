# TypeScript 备忘录

## 目录

- [1. 基础类型](#1-基础类型)
  - [1.1 类型别名唯一性](#11-类型别名唯一性)
  - [1.2 类型别名和接口的不同](#12-类型别名和接口的不同)
  - [1.3 类型别名/接口类型合并](#13-类型别名接口类型合并)
  - [1.4 类型断言](#14-类型断言)
  - [1.5 字面量推断](#15-字面量推断)
  - [1.6 非空断言操作符*后缀(!)*](#16-非空断言操作符后缀)
- [2. 类型收窄 (Narrowing)](#2-类型收窄-narrowing)
  - [2.1 类型守卫](#21-类型守卫-type-guards)
    - [2.1.1 typeof 操作符](#211-typeof-操作符)
    - [2.1.2 in 操作符](#212-in-操作符)
    - [2.1.3 instanceof 操作符](#213-instanceof-操作符)
    - [2.1.4 真值收窄](#214-真值收窄-truthiness-narrowing)
    - [2.1.5 等值收窄](#215-等值收窄-equality-narrowing)
  - [2.2 控制流分析](#22-控制流分析-control-flow-analysis)
  - [2.3 类型谓词](#23-类型谓词-type-predicates)
  - [2.4 可辨别联合](#24-可辨别联合-discriminated-unions)
  - [2.5 穷尽检查](#25-穷尽检查-exhaustiveness-checking)

---

## 1. 基础类型

### 1.1 类型别名唯一性

当你使用类型别名的时候, 它就跟你编写的类型是一样的

### 1.2 类型别名和接口的不同

- 类型别名和接口非常相似，大部分时候，你可以任意选择使用。接口的几乎所有特性都可以在 type 中使用，两者最关键的差别在于类型别名本身无法添加新的属性，而接口是可以扩展的
- 类型别名通过交集来扩展类型

```ts
type Animal = {
    name: string
}

type Rabbit = Animal & {
    further: boolean
}
```

### 1.3 类型别名/接口类型合并

- 接口重复声明会与之前的接口自动合并
- 类型别名更类似与**静态**声明 重复声明会直接报错

### 1.4 类型断言

```ts
const myCanvas = document.getElementById('canvas') as HTMLCanvasElement

// 等价

const myCanvas = <HTMLCanvasElement>document.getElementById('canvas')

```

### 1.5 字面量推断

当定义对象类型时, ts会默认其属性会被修改, 例如

```ts
const game = {
    score: 0
}

game.score = 1 // 根据推断 score是number类型 并不是只能为0 所以不会报错
```

```ts
declare function handleRequest(url: string, method: 'GET' | 'POST')

const reqObj = { url: 'http://xxxx', method: 'GET'}

handleRequest(reqObj) // Error 
```

- 由于 [字面量推断](#15-字面量推断), `reqObj` 的 `method` 会被推断为 `string` 类型, 而 `string` 类型和联合类型 `'GET' | 'POST'` 没有交集 所以会报类型错误

### 1.6 非空断言操作符*后缀(!)*

```ts
function test(x?: number | null) {
    console.log(x!.toFixed()) // 此断言后 尽管编译时不会报错 但运行时可能会产生错误
}
```

断言后x的值永远不会是 `null` 或者是 `undefined`
只有当你明确知道本值不可能是 `null` 或者 `undefined` 时才使用 `!`

## 2. 类型收窄 (Narrowing)

收窄是将类型推导为更精确类型的过程，在TypeScript中通过类型守卫、真值检查、等值比较、赋值语句和控制流分析等方式协助TypeScript进行类型推断，进而对类型进行收窄

### 2.1 类型守卫 (type guards)

类型守卫是 TypeScript 类型系统的核心特性，通过运行时检查实现类型收窄，提供类型安全和智能提示 有如下几种情况进行类型收窄:

#### 2.1.1 typeof 操作符

```ts
function printAll(strs: string | string[] | null) {
  if (typeof strs === "object") {
    for (const s of strs) {
      // typeof null === 'object' 导致ts会报错 可能为null
      console.log(s);
    }
  } else if (typeof strs === "string") {
    console.log(strs);
  } else {
    // 由于第一个条件存在 这儿永远不会触发
  }
}
```

在上述例子中，TypeScript 提示 `typeof strs` 会进行收窄，但是由于 `typeof null === 'object'` 这个 JavaScript 的"bug"，会被收窄为 `string[] | null`，所以会报可能为 `null` 的错误。这个错误在 JavaScript 运行时不会出现，但 TypeScript 的静态类型检查帮助我们提前发现了这个潜在的逻辑问题。

#### 2.1.2 in 操作符

这个例子展示了 `in` 操作符的类型收窄

```ts
type Fish = { speed: number, swim: () => string } as const
type Bird = { speed: number, fly: () => string }

interface ChinaBird extends Bird {
  name: string
}

interface ChinaFish extends Fish {
  name: string
}

function move(animalType: ChinaBird | ChinaFish) {
  if ('fly' in animalType) {
    // 通过 'fly' 属性存在性检查，TypeScript 将 animalType 收窄为 ChinaBird 类型
    return animalType.fly()
  } else {
    return animalType.swim()
  }
}

const CrestedIbis:ChinaBird = { speed: 100, name: '朱鹮', fly: function() {
  return  `${this.name} ${this.speed}m/s`
} }
move(CrestedIbis) // >>> 朱鹮 100m/s
```

#### 2.1.3 instanceof 操作符

类似于上述 `in` 操作符 都是通过类型保护来收窄类型

```ts
function getValue(x: Date | string) {
  if (x instanceof Date) {
    console.log(x) // x: Date
  } else {
    console.log(x) // x: string
  }
}
```

#### 2.1.4 真值收窄 (Truthiness narrowing)

通过判断值的真值性（`truthy/falsy`）来收窄类型，将联合类型中的 `null`、`undefined`、`0`、`""`、`false` 等假值排除，只保留*真值*类型

通过真值收窄完善上述的 [typeof](#211-typeof-操作符) 的例子

```ts
function printAll(strs: string | string[] | null) {
  if (strs && typeof strs === "object") {
    // strs的真值收窄 加上typeof的收窄 就只剩下string[]类型了
    for (const s of strs) {
      console.log(s);
    }
  } else if (typeof strs === "string") {
    console.log(strs);
  }
}
```

#### 2.1.5 等值收窄 (Equality narrowing)

通过 `===`、`!==`、`==`、`!=` 等比较操作符来收窄类型，当比较两个值相等时，TypeScript 会推断出更具体的类型

```ts
function getPosition(x: string | number, y: string | bigInt) {
  if (x === y) {
    // 由于 === 的前提是类型也需要相等 所以将类型收窄为string类型
    console.log(x) // string 
    console.log(y) // string
  } else {
    console.log(x) // string | number
    console.log(y) // string | bigInt
  }
}
```

### 2.2 控制流分析 (Control flow analysis)

基于可达性 `reachability` 的代码分析就叫做控制流分析 `control flow analysis`

```ts
function getSomeData(suffix: string | number, input: string) {
  if (typeof suffix === 'number') {
    return new Array(suffix + 1).fill('').join(" ") + input;
  }
  // 如果代码执行到这里类型就会被收窄 剔除number类型
  return suffix + input // 这里会变成string 类型
}
```

对于类型是 `number` 的 `suffix` 后半部分的代码是不可到达的, 所以后半部分的 `suffix` 就会收窄

### 2.3 类型谓词 (type predicates)

类型谓词是用户定义的类型守卫，通过返回 `parameterName is Type` 的布尔值来收窄类型, 即 `(value) => boolean`
来判断是否是对应的类型:

`function isString(value): value is string { return typeof value === 'string' }`

```ts
interface Fish {
  swim(): void;
  name: string;
}

interface Bird {
  fly(): void;
  name: string;
}

function isFish(pet: Fish | Bird): pet is Fish {
  return "swim" in pet;
}

// 使用示例
function move(pet: Fish | Bird) {
  if (isFish(pet)) {
    // pet 被收窄为 Fish
    pet.swim();
  } else {
    // pet 被收窄为 Bird
    pet.fly();
  }
}
```

### 2.4 可辨别联合 (Discriminated unions)

现在有这样定义的例子:

```ts
type Shape = {
  type: 'circle' | 'square'
  radius?: number
  sideLength?: number
}

// 获取圆面积
function getArea(shape: Shape): number {
  if (shape.type === 'square')
    return Math.PI * shape.radius ** 2 // Error
  else
    return shape.sideLength ** 2 // Error
}
```

上述代码都会报错, 即使我们通过[控制流分析](#22-控制流分析-control-flow-analysis)收窄了类型为 `circle`, 但因为 `radius` 属性是可选值, 仍然可能为 `undefined` 导致运行错误

在这种情况下 可辨别联合才是最佳实践, 下面改写一下上述例子

```ts
interface Circle {
  type: 'circle'
  radius: number
}

interface Square {
  type: 'square'
  sideLength: number
}
type Shape = Circle | Square

// 获取圆面积
function getArea(shape: Shape): number {
  if (shape.type === 'circle')
    return Math.PI * shape.radius ** 2
  else
    return shape.sideLength ** 2 // 现在类型就被正确收窄且均可访问
}
```

当联合类型中的每个类型，都包含了一个共同的字面量类型的属性，TypeScript 就会认为这是一个可辨别联合（discriminated union），然后可以将具体成员的类型进行收窄

可辨别联合在服务端交互, 状态管理中都是非常实用的内容

### 2.5 穷尽检查 (Exhaustiveness checking)

穷尽检查同样是一种类型收窄, 只是将类型收窄为了一个特殊的 `never` 类型, 表示所有类型都已经被处理到了 只剩下 `never` 的状态了

还是刚刚的形状例子, 假设在后续又增加了一个 `rect` 类型

```ts
// 已经定义好了 Circle | Square 新增一个Rect类型
interface Rect {
  type: 'rect'
  longSideLength: number
  shortSideLength: number
}
type Shape = Circle | Square | Rect

function getArea(shape: Shape): number {
  if (shape.type === 'circle')
    return Math.PI * shape.radius ** 2
  else
    return shape.sideLength ** 2 
  // 使用 if-else 结构时，TypeScript 不会进行穷尽检查，所以确实不会有编译错误。
}
```

但是如果一开始就加上了穷尽检查 像这样（由于多状态 用 `switch` 改写一下）

```ts
function getArea(shape: Shape): number {
  switch (shape.type) {
    case 'circle':
      return Math.PI * shape.radius ** 2
    case 'square':
      return shape.sideLength ** 2
    default:
      const exhaustiveCheck: never = shape
      return exhaustiveCheck
  }
}
```

因为 TypeScript 的收窄特性，执行到 `default` 的时候，类型被收窄为 `Rect`, 但因为任何类型都不能赋值给 `never` 类型（除了 `never` 本身）, 这就会产生一个编译错误。

通过这种方式，你就可以确保 `getArea` 函数总是穷尽了所有 `shape` 的可能性。
