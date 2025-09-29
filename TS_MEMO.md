# TS MEMO

## 类型 Type

### 1. 类型别名唯一

当你使用类型别名的时候, 它就跟你编写的类型是一样的

### 2. 类型别名和接口的不同

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

### 3. 类型别名/接口类型合并

- 接口重复声明会与之前的接口自动合并
- 类型别名更类似与**静态**声明 重复声明会直接报错

### 4. 类型断言

```ts
const myCanvas = document.getElementById('canvas') as HTMLCanvasElement

// 等价

const myCanvas = <HTMLCanvasElement>document.getElementById('canvas')

```

### 5. 字面量推断

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

- 由于 [字面量推断](#5-字面量推断), `reqObj` 的 `method` 会被推断为 `string` 类型, 而`string` 类型和联合类型 `'GET' | 'POST'`没有交集 所以会报类型错误

### 6. 非空断言操作符*后缀(!)*

```ts
function test(x?: number | null) {
    console.log(x!.toFixed()) // 此断言后 尽管编译时不会报错 但运行时可能会产生错误
}
```

断言后x的值永远不会是 `null` 或者是 `undefined`
只有当你明确知道本值不可能是 `null` 或者 `undefined` 时才使用 `!`

## 类型收窄 Narrowing

收窄是将类型推导为更精确类型的过程，在TypeScript中通过类型守卫、真值检查、等值比较、赋值语句和控制流分析等方式协助TypeScript进行类型推断，进而对类型进行收窄

### 1. 类型守卫

类型守卫是 TypeScript 类型系统的核心特性，通过运行时检查实现类型收窄，提供类型安全和智能提示 有如下几种情况进行类型收窄:

#### 1.1 typeof

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

#### 1.2 in操作符

这个例子展示了in操作符的类型收窄
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

#### 1.3 instaceof

类似于上述in操作符 都是通过类型保护来收窄类型

```ts
function getValue(x: Date | string) {
  if (x instaceof Date) {
    console.log(x) // x: Date
  } else {
    console.log(x) // x: string
  }
}
```

#### 1.4. 真值收窄

通过判断值的真值性（truthy/falsy）来收窄类型，将联合类型中的 `null`、`undefined`、`0`、`""`、`false` 等假值排除，只保留*真值*类型

#### 1.5 等值收窄

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

### 2.控制流分析
TODO 