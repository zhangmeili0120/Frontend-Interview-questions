##### 3. 你知道vue中key的作用和工作原理吗？说说你对它的理解。
vue中的key用来管理不复用元素，vue为了高效渲染，通常会复用已有节点，但是有时候不需要复用节点，这时候给每个节点加个key属性，就使得当前节点变成了唯一的，不在复用其他相同节点。

```
src/core/vdom/patch.js

function sameVnode (a, b) {
  return (
    a.key === b.key && (
      (
        a.tag === b.tag &&
        a.isComment === b.isComment &&
        isDef(a.data) === isDef(b.data) &&
        sameInputType(a, b)
      ) || (
        isTrue(a.isAsyncPlaceholder) &&
        a.asyncFactory === b.asyncFactory &&
        isUndef(b.asyncFactory.error)
      )
    )
  )
}
```
虚拟dom比较的方法里边：首先判断了两个节点是否是同一个使用到了key属性对比判断，也就是说如果给循环节点手动写了key属性，虚拟dom就不会复用啦。