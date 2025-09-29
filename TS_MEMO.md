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
- 由于上述的字面量推断 `reqObj` 的 `method` 会被推断为 `string` 类型
- `string` 类型和联合类型 `'GET' | 'POST'`没有交集 所以报类型错误

### 6. 非空断言操作符*后缀(!)*

```ts
function test(x?: number | null) {
    console.log(x!.toFixed()) // 此断言后 运行时可能会产生错误
}
```

断言后x的值永远不会是 `null` 或者是 `undefined`
只有当你明确知道本值不可能是 `null` 或者 `undefined` 时才使用 `!`