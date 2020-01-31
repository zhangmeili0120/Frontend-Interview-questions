#### 12. vue扩展

- vue.extend()

  - 使用基础 Vue 构造器，创建一个“子类”。参数是一个包含组件选项的对象。

  - 可以用组件构造器来创建组件

    ```
    // 1.创建组件构造器
    let myBaseComConstructor = Vue.extend({
    	template: `this is my base component!`
    })
    
    // 2.根据Vue.component('组件名', 组件构造器)来创建新组件
    let myFirstCom = Vue.component('my-first-com', myBaseComConstructor)
    ```

    

- mixins

  - `mixins` 选项接收一个混入对象的数组。这些混入对象可以像正常的实例对象一样包含实例选项，这些选项将会被合并到最终的选项中，使用的是和 `Vue.extend()` 一样的选项合并逻辑。也就是说，如果你的混入包含一个 created 钩子，而创建组件本身也有一个，那么两个函数都会被调用。

    

  - Mixin 钩子按照传入顺序依次调用，并在调用组件自身的钩子之前被调用。如果是全局混入Vue.mxins()，则先执行全局混入。
  - mixins可以考虑用在一些全局的生命周期里，以减少重复代码。

- extends()

  - 允许声明扩展另一个组件(可以是一个简单的选项对象或构造函数)，而无需使用 `Vue.extend`。这主要是为了便于扩展单文件组件。

- mixins和extends同时声明的情况下，会先执行extends，后执行mixins中的方法。

