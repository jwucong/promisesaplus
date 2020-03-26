
# Promises/A+规范
> [Promises/A+规范原文](https://promisesaplus.com/)  

**一个由开发者制定的开放、健全、可交互的 JavaScript Promise 标准，供开发者使用。**  

一个 promise 表示一个异步操作的最终结果。与 promise 交互的方式主要是通过它的 then 方法，
该方法定义了两个回调函数，用于接收该 promise 完成时的最终值或者无法完成的原因。

本规范详细说明了 then 方法的行为，所有遵循 Promises/A+ 规范的 promise 均可以基于本规范去实现 then 方法。因此，本规范是非常稳定的。
尽管 Promises/A+ 组织有时会通过向后兼容的小改动来修订本规范，以解决一些新发现的极端情况，
只有经过谨慎的考虑，讨论和测试之后，我们才会进行不向后兼容的大改动。

从历史上看，Promises/A+ 规范进一步明确了早期 Promises/A 提案中的行为条款，扩展了原有提案中的一些约定俗成的行为并删减了一些定义含糊不清和有问题的部分。

最后，Promises/A+ 规范的核心不涉及如何创建、完成或拒绝 promise，而是专注于提供一个可交互的 then 方法，上述提到的一些方法可能会在今后的配套规范中涉及。


## 1. 术语

1. "promise" 是一个具有 then 方法的对象或函数，其行为符合此规范。
2. "thenable" 是一个定义了 then 方法的对象或函数。
3. "value" 是一个任意的 JavaScript 合法值(包括undefined, thenable, promise)。
4. "exception" 是一个是使用 throw 语句抛出的值。
5. "reason" 是一个表示 promise 被拒绝的原因。


## 2. 要求

### 2.1 Promise的状态

一个promise必须处于以下三种状态之一: 等待中(pending), 已完成(fulfilled), 已拒绝(rejected)。

1. 处于等待中(pending)的 promise

   1. 可以转变为已完成(fulfilled)或已拒绝(rejected)状态。
   
2. 处于已完成(fulfilled)的 promise

   1. 不能转变为其他状态。
   2. 必须拥有一个无法改变的最终值。
   
3. 处于已拒绝(rejected)的 promise

   1. 不能转变为其他状态。
   2. 必须拥有一个无法改变的被拒绝的原因。
   
### 2.2 then 方法
   
一个 promise 必须提供一个 then 方法来访问其当前值、最终值或者被拒绝的原因。

promise 的 then 方法接收两个参数:  

```javascript
promise.then(onFulfilled, onRejected)
```

1. onFulfilled 和 onRejected 都是可选的参数:  

   1. 如果 onFulfilled 不是一个函数，其必须被忽略。
   2. 如果 onRejected 不是一个函数，其必须被忽略。
   
2. 如果 onFulfilled 是一个函数，则: 

   1. 该函数必须在 promise 完成时被调用，以 promise 的最终值作为第一个参数传入。
   2. 该函数不能在 promise 完成前被调用。
   3. 该函数不能被多次调用。
   
3. 如果 onRejected 是一个函数，则:

   1. 该函数必须在 promise 被拒绝时调用，以 promise 的拒绝原因作为第一个参数传入。
   2. 该函数不能在 promise 被拒绝前调用。
   3. 该函数不能被多次调用。
   
4. onFulfilled 或 onRejected 只有在执行上下文堆栈仅包含平台代码时才能被调用。[[3.1](#notes1)]

5. onFulfilled 和 onRejected 必须被作为函数调用(即没有 this 值)。[[3.2](#3.-注释)]

6. then 方法可以被同一个 promise 多次调用。

   1. 当 promise 完成时，所有的 onFulfilled 回调都必须按照其原有的顺序执行。
   2. 当 promise 被拒绝时，所有的 onRejected 回调都必须按照其原有的顺序执行。
   
7. then 方法必须返回一个 promise [[3.3](#3.-注释)]

   ```javascript
   promise2 = promise1.then(onFulfilled, onRejected);
   ```
   
   1. 如果 onFulfilled 或 onRejected 返回一个值 x, 则执行下面的Promise解决过程: \[\[Resolve\]\]\(promise2, x\)
   
   2. 如果 onFulfilled 或 onRejected 使用 throw 语句抛出异常 e，则 promise2 必须以 e 作为拒绝的原因进入已拒绝状态。
   
   3. 如果 onFulfilled 不是一个函数且 promise1 处于已完成状态，
   则 promise2 也必须处于已完成状态，且必须拥有与 promise1 一样的最终值。
   
   4. 如果 onRejected 不是一个函数且 promise1 处于已拒绝状态，
   则 promise2 也必须处于已拒绝状态，且必须拥有与 promise1 一样的拒绝的原因。 
   

### 2.3 Promise 解决过程
 
此解决过程是一个抽象的操作，它需要输入一个 promise 和一个值，我们把它表示为 \[\[Resolve\]\]\(promise, x\) 。
如果 x 是一个定义了 then 方法的对象或函数，且其行为看起来像是一个 promise, 那么此解决过程将尝试使 promise 接受 x 的状态。
否则，将使用 x 作为最终值使 promise 进入已完成状态。

这种对 thenables(指那些定义了then方法的对象或函数)的处理方式使得 promise 的实现更具通用性，
只要其暴露出一个遵循 Promises/A+ 规范的 then 方法即可。
这同时也使遵循 Promises/A+ 规范的实现可以兼容那些不太规范但可用的实现。

执行 \[\[Resolve\]\]\(promise, x\), 需要遵循以下步骤: 

1. 如果 promise 和 x 指向同一对象, 则以 TypeError 作为拒绝的原因使 promise 进入已拒绝状态。

2. 如果 x 是一个 promise，则使 promise 接受 x 的状态: [[3.4](#3.-注释)]

   1. 如果 x 的状态处于等待中，则 promise 的状态也必须处于等待中，直到 x 的状态变为已完成或已拒绝。
   2. 如果 x 的状态为已完成，则使用相同的值使得 promise 进入已完成状态。
   3. 如果 x 的状态为已拒绝，则使用相同的原因使得 promise 进入已拒绝状态。

3. 否则， 如果 x 是一个对象或者函数: 

   1. 定义一个临时变量 then = x.then [[3.5](#3.-注释)]
   2. 如果读取属性 x.then 时抛出了一个异常 e, 则以 e 作为拒绝的原因使 promise 进入已拒绝状态。
   3. 如果 then 是一个函数，则以 x 作为该函数执行上下文的 this 去调用该函数，并传入两个回调函数作为参数，
   第一个参数叫做 resolvePromise， 第二个参数叫做 rejectPromise
   
      1. 如果 resolvePromise 以值 y 为参数被调用，则执行 \[\[Resolve\]\]\(promise, y\)
      2. 如果 rejectPromise 以原因 r 为参数被调用，则以原因 r 使 promise 进入已拒绝状态。
      3. 如果 resolvePromise 和 rejectPromise 均被调用，或者被以同一参数的方式调用了多次，则采用首次调用，忽略其余的调用。
      4. 如果调用 then 方法时抛出了异常 e，则: 
      
         1. 如果 resolvePromise 或 rejectPromise 已经被调用，则忽略该异常。
         2. 否则以 e 为原因使 promise 进入已拒绝状态。
      
   4. 如果 then 不是一个函数, 则以 x 作为最终值使 promise 进入完成状态。
   
4. 如果 x 不是一个对象或者不是一个函数，则以 x 作为最终值使 promise 进入完成状态。
      
如果一个 promise 被一个循环的 thenable 链中的对象置为已完成状态，
而 \[\[Resolve\]\]\(promise, thenable\) 的递归性质又使得其被再次调用，上述的算法将会陷入无限递归之中。
算法虽不强制要求，但鼓励实现者检测这样的递归是否存在，若检测到存在则以一个可识别的 TypeError 作为原因来把 promise 置为已拒绝状态。[[3.6](#3.-注释)]


## 3. 注释

1. <span id="notes1"></span>这里的平台代码指的是引擎、环境以及 promise 的具体实现代码。
在实践中要确保 onFulfilled 和 onRejected 方法异步执行，且应该在 then 方法被调用的那一轮事件循环之后的新执行栈中执行。
这个事件队列可以采用类似 setTimeout 或 setImmediate 的"宏任务"机制
或者类似 MutationObserver 或 process.nextTick 的"微任务"机制来实现。
由于 promise 的具体实现代码本身就是平台代码，所以代码自身在处理任务时很可能已经包含了一个任务调度队列。

2. 意思是：在严格模式中，函数执行上下文的 this 的值为 undefined, 在非严格模式中其为全局对象。

3. 具体实现在满足所有要求的情况下可以允许 promise2 === promise1 。
但要求每个实现都要有文档说明其是否允许以及在何种条件下允许 promise2 === promise1。

4. 通常情况下，如果 x 符合当前实现，我们才认为它是真正的 promise。这一规则允许那些特例实现接受符合已知要求的 promises 状态。

5. 这里我们先是存储了一个指向 x.then 的引用，然后测试并调用该引用，以避免多次访问 x.then 属性。这种预防措施确保了该属性的一致性，因为其值可能在读取时被改变。

6. 具体实现不应该对 thenable 链的深度进行任何的限制，假设因此导致了无限递归，只有真正的循环递归才应该导致 TypeError 异常；
如果一条无限长的 thenable 链上的每个 thenable 都不相同，那么永远递归下去也是正确的。
