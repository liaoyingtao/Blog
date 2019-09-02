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

2.2.4. `onFulfilled` 和 `onRejected` 只有在 [执行环境](https://es5.github.io/#x10.3) 堆栈仅包含平台代码时才可被调用 <sup>注1</sup>。

2.2.5. `onFulfilled` 和 `onRejected` 必须作为函数被调用（即没有 `this` 值） <sup>注2</sup>。

2.2.6. 在同一个 promise 中，`then` 方法可以被多次调用：
- 当 `promise` 完成时，所有 `onFulfilled` 回调必须按照 `then` 起始的顺序依次调用。
- 当 `promise` 失败时，所有 `onRejected` 回调必须按照 `then` 起始的顺序依次调用。

2.2.7. `then` 方法必须返回一个 promise <sup>注3</sup>。
```js
promise2 = promise1.then(onFulfilled, onRejected);
```