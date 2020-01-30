#### vue3.0新特性

 - 更快：
	虚拟 DOM 重写，我们可以期待更多的 编译时（compile-time）提示来减少 运行时（runtime）开销。重写将包括更有效的代码来创建虚拟节点。
	Vue 的响应式系统是使用 Object.defineProperty 的 getter 和 setter。 但是，Vue 3 将使用 ES2015 Proxy 作为其观察者机制。 这消除了以前存在的警告，使速度加倍，并节省了一半的内存开销。
 - 更小：
 Vue已经非常小了，在运行时（runtime）压缩后大约 20kb 。 但我们可以期待它会变得更加小，新的核心运行时压缩后大概 10kb 。 这将在很大程度上通过消除不使用的库(也称为Tree Shaking)来实现。 
 - 更易于维护：
 源代码迁移改成 TypeScript
 - 兼容性：
 vue3.0 会兼容vue2.0  语法 和 API  帮助 2.0 用户平滑过渡到3.0 

 