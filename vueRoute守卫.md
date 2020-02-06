#### 18. vue-router导航钩子有哪些？

全局前置守卫

​	router.beforeEach(to, form, next) {}

全局解析守卫 2.5.0+

​	router.beforeResolve(to, from, next) {}

​	会在所有组件内守卫和异步路由组件被解析之后调用

全局后置钩子

​	router.afterEach(to, from)

​	后置钩子没有next参数，不会改变导航

路由独享守卫

​	router.beforeEnter(to, from, next) {}

```
const route = [
	{
        path: '/foo',
        component: Foo,
        beforeEnter(to, from, next) {
            ...
        }
	}
]
```

组件内守卫

```
beforeRouteEnter(to, from, next) {
	// 在渲染该组件的对应路由被confirm前调用，无法访问this实例，因为守卫执行前，组件实例还未被创建。
}

beforeRouteUpdate(to, from, next) {
	// 当复用一个组件的情况下调用，可以访问实例this
}

beforeRouteLeave(to, from, next) {
	// 导航离开该组件的路由时调用，可以访问实例this
}
```

完整的导航解析流程：

​	1.导航被触发 =》 2.失活的组件里调用beforeRouteLeave 守卫 =》 3.调用全局beforEach守卫 =》 4.重用的组件调用beforeRouteUpdate守卫 （2.2.0+） =》5.路由配置里调用beforeEnter	=》 6.解析异步路由组件。 =》7.被激活的组件里调用beforeRouteEnter  =》8.调用全局beforeResolve守卫 （2.5+） =》9.导航被确认。=》10.调用全局afterEach守卫。=》11.触发DOM更新。=》 12.创建好的实例调用beforeRouteEnter守卫中传给next的回调函数。