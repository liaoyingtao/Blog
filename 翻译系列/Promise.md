原文：[Promises/A+](https://promisesaplus.com/)

# Promises/A+
一个可靠、通用的 **Javascript Promises** 公开标准——由开发者制定，供开发者参考。

Promise 表示一个异步操作的最终结果，与 promise 进行交互的主要方式是通过 `then` 方法，该方法可以接受两个回调函数作为参数，用于接收最终值或不能转换到 `fulfilled` 状态的原因。

本规范详细列出了 `then` 方法的行为，提供通用的基础，所有遵循 Promises/A+ 标准实现的 promise 均可以本规范作为标准。本规范十分稳定，尽管 Promises/A+ 组织会偶尔修订它，去处理一些新发现的边界问题，但都是一些向后兼容的小改动。只有经过谨慎的考虑、讨论和测试后，我们才会合并大规模改动或不向后兼容的改动。

Promises/A+ 阐述了早期 Promises/A 协议的行为，扩展已有的行为，删减有问题的部分。

最后，Promises/A+ 规范的核心不规定如何去创建、成功或失败，而是专注于提供一个通用的 `then` 方法。未来有关配套规范的工作可能涉及到这些主题。

## 1. 术语
**promise**

表示拥有 `then` 方法的对象或函数，并且它的行为遵循本规范。

**thenable**

表示定义了 `then` 方法的对象或函数。

**value**

表示任何一个合法的 JavaScript 值（包含 `undefined`，thenable 和 promise）。

**exception**

表示 `throw` 语句抛出的一个值。

**reason**

表示 promise 被失败的原因。

## 2. 要求

### 2.1. Promise 状态
一个 promise 必须在这三种状态的其中一种：pending（进行中）、fulfilled（已成功）和 rejected（已失败）。

2.1.1 当处于 pending 状态时，promise 需满足：
- 可以改变为 fulfilled 或 rejected 状态

2.1.2 当处于 fulfilled 状态时，promise 需满足：
- 状态不会再变
- 必须拥有一个不会变的值

2.1.3 当处于 rejected 状态时，promise 需满足：
- 状态不会再变
- 必须拥有一个不会变的失败原因

这里的 "不会变" 指的是不变恒等式（即可用 === 判断相等），而不是意味着更深层次的不变性（译者注：引用类型）。

### 2.2. `then` 方法
一个 promise 必须提供一个 `then` 方法去接收当前的最终值或失败原因。

promise 的 `then` 方法接收两个参数：
```js
promise.then(onFulfilled, onRejected);
```

2.2.1. `onFulfilled` 和 `onRejected` 都是可选的参数：
- 如果 `onFulfilled` 不是一个函数，那么它将被忽略
- 如果 `onRejected` 不是一个函数，那么它将被忽略

2.2.2. 如果 `onFulfilled` 是一个函数：
- 在 `promise` 状态变为 fulfilled 后必须被调用，用 `promise` 的返回值作为第一个参数。
- 在 `promise` 状态变为 fulfilled 之前不能被调用
- 调用次数不超过一次

2.2.3. 如果 `onRejected` 是一个函数：
- 在 `promise` 状态变为 fulfilled 后必须被调用，用 `promise` 失败的原因作为第一个参数。
- 在 `promise` 状态变为 rejected 之前不能被调用
- 调用次数不超过一次

2.2.4. `onFulfilled` 和 `onRejected` 只有在 [执行环境](https://es5.github.io/#x10.3) 堆栈仅包含 "platform code" 时才可被调用 <sup>注1</sup>。

2.2.5. `onFulfilled` 和 `onRejected` 必须作为函数被调用（即没有 `this` 值） <sup>注2</sup>。

2.2.6. 在同一个 promise 中，`then` 方法可以被多次调用：
- 当 `promise` 完成时，所有 `onFulfilled` 回调必须按照 `then` 起始的顺序依次调用。
- 当 `promise` 失败时，所有 `onRejected` 回调必须按照 `then` 起始的顺序依次调用。

2.2.7. `then` 方法必须返回一个 promise <sup>注3</sup>。
```js
promise2 = promise1.then(onFulfilled, onRejected);
```
- 如果 `onFulfilled` 或 `onRejected` 返回了值 x，则运行 `[[Resolve]](promise2, x)` 处理 Promise。（译者注：意思是 promise1 的返回值将作为 promise2 的参数）
- 如果 `onFulfilled` 或 `onRejected` 抛出异常 `e`，则 `promise2` 将以 `e` 为作为参数调用 reject 方法。
- 如果 `onFulfilled` 不是函数，且 `promise1` 已成功，则 `promise2` 将以同一值作为参数调用 resolve 方法。（译者注：第三和第四条的意思是，将参数一层一层传递下去，如下面代码）
- 如果 `onRejected` 不是函数，且 `promise1` 已失败，则 `promise2` 将以同一失败原因作为参数调用 reject 方法。

```js
const promise2 = new Promise((resolve) => {
  resolve('666');
}).then(); // onFulfilled 不是函数，且 `promise1` 已成功

promise2.then(val => {
  console.log(val); // 666
});
```

### 2.3. Promise 处理
**promise 处理程序** 是一个抽象的操作，它接收一个 promise 和一个值作为参数，伪代码表示为 `[[Resolve]](promise, x)`。如果 `x` 是一个定义了 `then` 方法的对象或函数，此操作将尝试让 `promise` 采用 `x` 的状态，前提是 `x` 的行为跟 promise 相似。否则，`promise` 将携带 `x` 转换为"成功"。

这种 thenables 的处理使得 promise 的实现更具通用性，只需要暴露一个符合 Promise/A+ 规范的 `then` 方法。在有符合规范的 `then` 方法前提下，允许 Promise/A+ 实现与不符合规范的实现共存。

为了实现 `[[Resolve]](promise, x)`，需遵循以下内容：

2.3.1. 如果 `promise` 和 `x` 指向同一个对象，reject 这个 `promise`，抛出一个 `TypeError` 异常。

2.3.2. 如果 `x` 是一个 promise，则采用它的状态：<sup>注4</sup>
- 如果 `x` 处于 pending，`promise` 必须停留在 pending 状态，直到 x 转换为 fulfilled 或 rejected。
- 当 `x` 成功时，需要以相同值执行 `promise`。
- 当 `x` 成功时，需要以相同失败原因拒绝 `promise`。

2.3.3. 如果 `x` 是一个对象或函数时：
- 把 `x.then` 赋值给 `then`。<sup>注5</sup>
- 如果取 `x.then` 的值时抛出错误 `e`，则以 `e` 为据因拒绝 `promise`。
- 如果 `then` 是一个函数，则将 `x` 作为 `this` 调用它，第一个参数为 `resolvePromise`，第二个参数为 `rejectPromise`：
  - 如果 `resolvePromise` 被调用后返回值 `y`，则继续运行 `[[Resolve]](promise, y)`。
  - 如果 `rejectPromise` 被调用后返回据因 `r`，则以据因为 `r` 拒绝 promise。
  - 如果 `resolvePromise` 和 `rejectPromise` 均被调用，或以同一参数多次调用，第一次调用将有优先权，并忽略其他的调用。
  - 如果调用 `then` 方法抛出异常 `e`：
    - 如果 `resolvePromise` 和 `rejectPromise` 已经被调用，那么忽略此操作。
    - 其他情况，以 `e` 为据因拒绝 `promise`。
- 如果 `then` 不是一个函数，则以 `x` 为参数执行 `promise`。

2.3.4. 如果 `x` 不是一个对象或函数，则以 `x` 为参数执行 `promise`。

如果一个 promise 执行后返回一个 thenable，且做为循环链中的一部分，像这样递归形状的 `[[Resolve]](promise, thenable)` 最终都会导致 `[[Resolve]](promise, thenable)` 再次被调用，遵循这样的算法将会导致无限递归。规范鼓励但不要求去检测这样的递归，它将抛出一个 `TypeError` 据因拒绝 `promise`。

## 3. 注解
注1 这里的 "platform code" 表示引擎、环境和实现 promise 的代码。在实践中需要确保 `onFulfilled` 和 `onRejected` 是异步执行，且应该在 `then` 方法被调用的那一轮事件循环之后的新执行栈中执行。可以利用像 `setTimeout` 或 `setImmediate` 这样的 "宏任务" 机制，或 `MutationObserver` 或 `process.nextTick` 这样的 "微任务" 实现。由于 promise 的实现本来就是 platform code，被调用后，它自身可能就包含了一个任务调度队列。

注2 这里的意思是，在严格模式下 `this` 将会是 `undefined`，在松散模式下，它将是全局对象。

注3 在满足所有规范的情况下，可以允许 `promise2 === promise1` 。每个实现都需要文档说明，在什么情况下，是否支持 `promise2 === promise1`。

注4 一般情况下，只有当 `x` 是一个满足规范的实现才是一个真正的 promise。这些条例允许一些特殊实现去兼容已知的符合规范的 promises。

注5 这步我们先是存储了一个指向 x.then 的引用，然后测试并调用该引用，以避免多次访问 x.then 属性。这种预防措施确保了该属性的一致性，因为其值可能在检索调用时被改变。

注6 实现不应该对 thenable 链的深度设限，并假定超出本限制的递归就是无限循环。只有真正的循环递归才应能导致 TypeError 异常；如果一条无限长的链上 thenable 均不相同，那么递归下去永远是正确的行为。
