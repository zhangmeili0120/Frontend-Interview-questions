#### 16. 你知道nextTick的原理吗？

官网解释的是这样的：

    可能你还没有注意到，Vue 在更新 DOM 时是异步执行的。只要侦听到数据变化，Vue 将开启一个队列，并缓冲在同一事件循环中发生的所有数据变更。如果同一个 watcher 被多次触发，只会被推入到队列中一次。这种在缓冲时去除重复数据对于避免不必要的计算和 DOM 操作是非常重要的。然后，在下一个的事件循环“tick”中，Vue 刷新队列并执行实际 (已去重的) 工作。Vue 在内部对异步队列尝试使用原生的 Promise.then、MutationObserver 和 setImmediate，如果执行环境不支持，则会采用 setTimeout(fn, 0) 代替。

    例如，当你设置 vm.someData = 'new value'，该组件不会立即重新渲染。当刷新队列时，组件会在下一个事件循环“tick”中更新。多数情况我们不需要关心这个过程，但是如果你想基于更新后的 DOM 状态来做点什么，这就可能会有些棘手。虽然 Vue.js 通常鼓励开发人员使用“数据驱动”的方式思考，避免直接接触 DOM，但是有时我们必须要这么做。为了在数据变化之后等待 Vue 完成更新 DOM，可以在数据变化之后立即使用 Vue.nextTick(callback)。这样回调函数将在 DOM 更新完成后被调用。

简单理解下就是：当数据变化后，dom的更新操作被当作异步任务推入异步执行队列，只有所有的同步任务完成之后，才会执行异步队列里的任务。

我们看下源代码里是如何实现的：

    export const nextTick = (function () {
        const callbacks = []
        let pending = false
        let timerFunc

        function nextTickHandler () {
            pending = false
            const copies = callbacks.slice(0)
            callbacks.length = 0
            for (let i = 0; i < copies.length; i++) {
                copies[i]()
            }
        }

        if (typeof Promise !== 'undefined' && isNative(Promise)) {
            var p = Promise.resolve()
            var logError = err => { console.error(err) }
            timerFunc = () => {
                p.then(nextTickHandler).catch(logError)
                
                if (isIOS) setTimeout(noop)
            }
        } else if (!isIE && typeof MutationObserver !== 'undefined' && (
            isNative(MutationObserver) ||
            // PhantomJS and iOS 7.x
            MutationObserver.toString() === '[object MutationObserverConstructor]'
        )) {
            // use MutationObserver where native Promise is not available,
            // e.g. PhantomJS, iOS7, Android 4.4
            var counter = 1
            var observer = new MutationObserver(nextTickHandler)
            var textNode = document.createTextNode(String(counter))
            observer.observe(textNode, {
                characterData: true
            })
            timerFunc = () => {
                counter = (counter + 1) % 2
                textNode.data = String(counter)
            }
        } else {
            // fallback to setTimeout
            /* istanbul ignore next */
            timerFunc = () => {
                setTimeout(nextTickHandler, 0)
            }
        }

        return function queueNextTick (cb?: Function, ctx?: Object) {
            let _resolve
            callbacks.push(() => {
            if (cb) {
                try {
                    cb.call(ctx)
                } catch (e) {
                    handleError(e, ctx, 'nextTick')
                }
            } else if (_resolve) {
                _resolve(ctx)
            }
            })
            if (!pending) {
                pending = true
                timerFunc()
            }
            if (!cb && typeof Promise !== 'undefined') {
                return new Promise((resolve, reject) => {
                    _resolve = resolve
                })
            }
        }
    })()

callbacks // 用来存储所有需要执行的回调函数
pending // 用来标识是否正在执行回调函数
timerFunc  // 用来触发执行回调函数
queueNextTick // 因为nextTick是一个即时函数，所以queueNextTick函数是返回的函数，接受用户传入的参数，用来往callbacks里存入回调函数
整个执行流程，关键在于timeFunc()，该函数起到延迟执行的作用。