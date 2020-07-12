### 介绍

`async` 函数 `ES2017` 标准，是目前前端最优雅的处理异步的方式（谁知道会不会被后来者干掉，狗头保命。:)

*一句话介绍：`async` 函数使得异步操作变得更加方便，更像“同步”代码。*

`async` 函数的返回值总是一个 `Promise`，分为以下几种情况：

1. 没有显式 `return`，相当于 `return Promise.resolve(undefined)`
2. `return` 非 `Promise` 的数据 `data`，相当于 `return Promise.resolve(data)`
3. `return Promise`, 会得到 `Promise` 对象本身

多个 `await` 命令后面的异步操作，如果不存在继发关系，最好让它们同时触发。

```javascript
let foo = await getFoo()
let bar = await getBar()
```

上面代码中，`getFoo` 和 `getBar` 是两个独立的异步操作（即互不依赖），被写成继发关系，这样比较耗时，因为只有 `getFoo` 完成以后，才会执行 `getBar`，完全可以让它们同时触发。

```javascript
// 写法一
let [foo, bar] = await Promise.all([getFoo(), getBar()])
// 写法二
let fooPromise = getFoo()
let barPromise = getBar()
let foo = await fooPromise
let bar = await barPromise
```

上面两种写法，`getFoo` 和 `getBar` 都是同时触发，这样就会缩短程序的执行时间。

### 错误处理

先看下面代码：

```javascript
let num
async function func() {
    await Promise.reject('Error')
    await Promise.resolve('Success')
    num = 9 // 函数执行不到这里来
}
```

上面代码表明当 `async` 函数中只要有一个 `await` 出现异常，则后面的 `await` 和其余代码都不会被执行，而会直接跳出函数，而解决这种问题最优的方法就是使用 `try/catch` 的逻辑。

```javascript
let num
async function func() {
    try {
        await Promise.reject('Error')
    } catch (error) {
        console.log(error)
    }
    num = 9
    return num
}
func().then(res => console.log(num)) // 9
// 亦或是
async function resultsCaptured(cb) {
    try {
        let res = await cb()
        return [null, res]
    } catch (err) {
        return [err, null]
    }
}
async function func() {
    let [err, res] = await resultsCaptured(cb)
    if (err) {
        // 处理错误逻辑
    } else {
        // 处理成功逻辑
    }
}
func()
```

### 在循环中使用 `await`

#### forEach 循环

```javascript
const loop = () => {
    const arr = [1,2,3]
    const printResult = (res) => {
        return new Promise(resolve => setTimeout(resolve(res),1000))
    }
    console.log('start')
    arr.forEach(async (item) => {
        const res = await printResult(item)
        console.log(res)
    })
    console.log('end')
}
loop()
// 期望结果：
'start'
1
2
3
'end'
// 实际结果：
'start'
'end'
1
2
3
// 这时三个 printResult 操作是并发执行，也就是同时执行，会被推入微任务队列。
```

#### for 循环

```javascript
const loop = async () => {
    const arr = [1,2,3]
    const printResult = (res) => {
        return new Promise(resolve => setTimeout(resolve(res),1000))
    }
    console.log('start')
    for (let i = 0; i < arr.length; i++) {
        const res = await printResult(arr[i])
        console.log(res)
    }
    console.log('end')
}
loop()
// 期望结果：
'start'
1
2
3
'end'
// 实际结果：
'start'
1
2
3
'end'
// 这时三个 printResult 操作是继发执行。
// for...of 与 for 同样效果。
```

如果希望多个请求**并发执行**，也可以使用 `Promise.all` 方法。

```javascript
async function func(db) {
  const arr = [{}, {}, {}]
  const promises = arr.map((item) => db.post(item))
  const results = await Promise.all(promises)
  console.log(results)
}
```

想要**并发执行**异步请求，可以用 `forEach`、`map`、`filter` 等有回调的循环，也可以用 `Promise.all`。

想要**继发执行**异步请求，就用 `for` 或 `for...of` 循环。这一般适用于那种后一个异步依赖于前一个异步的结果的场景。

实际开发场景中，我们尽量多用**并发执行**去实现代码。
