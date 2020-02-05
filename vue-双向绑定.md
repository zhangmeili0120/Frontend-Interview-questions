

#### 17. vue数据双向绑定原理

vue使用Object.defineProperty()对传入的对象处理为get和set方法的对象



我们从源码来看，new Vue()接收了参数后如何做的响应式处理：



// src\core\instance\index.js

```
function Vue (options) {
  // 初始化
  this._init(options)
}
// 通过混入定义了_init方法
initMixin(Vue)
```



// src\core\instance\init.js

```


export function initMixin (Vue: Class<Component>) {
	...
    initState(vm) // 数据响应式
	...
}
```



// src\core\instance\state.js

```

export function initState (vm: Component) {
	...
	initData(vm) // 对data响应式处理
	...
}

// 对data响应式处理
function initData (vm: Component) {
  let data = vm.$options.data
  data = vm._data = typeof data === 'function'
    ? getData(data, vm)
    : data || {}
  
  ...
  // 响应式方法
  observe(data, true /* asRootData */)
  ...
}
```



// src\core\observer\index.js

```
export function observe (value: any, asRootData: ?boolean): Observer | void {
  
  let ob: Observer | void
  // ob就是观察者，存在的直接返回，不存在创建一个并返回
  
  ...
  ob = new Observer(value)
  ...

  // 返回一个Observer 观察者实例
  return ob
}


// Observer类
export class Observer {
  value: any;
  dep: Dep;
  vmCount: number; // number of vms that have this object as root $data

  constructor (value: any) {
    this.value = value
    this.dep = new Dep()
    this.vmCount = 0
    def(value, '__ob__', this)
    if (Array.isArray(value)) {
      if (hasProto) {
        protoAugment(value, arrayMethods)
      } else {
        copyAugment(value, arrayMethods, arrayKeys)
      }
      this.observeArray(value)
    } else {
      this.walk(value)
    }
  }

  /**
   * Walk through all properties and convert them into
   * getter/setters. This method should only be called when
   * value type is Object.
   */
  walk (obj: Object) {
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i])
    }
  }

  /**
   * Observe a list of Array items.
   */
  observeArray (items: Array<any>) {
    for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i])
    }
  }
}



// 数据响应化处理
export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function,
  shallow?: boolean
) {
  // 为什么要创建一个dep?每一个对象里边都要有一个dep
  // object增删属性时候
  // array数组发生修改时（7个方法）都可以通知界面更新

  // dep和key一一对应
  const dep = new Dep()

  const property = Object.getOwnPropertyDescriptor(obj, key)
  if (property && property.configurable === false) {
    return
  }

  // cater for pre-defined getter/setters
  const getter = property && property.get
  const setter = property && property.set
  if ((!getter || setter) && arguments.length === 2) {
    val = obj[key]
  }

  // 只要是对象类型，均会返回childOb
  let childOb = !shallow && observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      // 获取key对应的值
      const value = getter ? getter.call(obj) : val
      // 如果存在依赖
      if (Dep.target) {
        // 则进行依赖收集
        dep.depend()
        // 如果存在childOb，则子ob也收集这个依赖
        if (childOb) {
          childOb.dep.depend()
          if (Array.isArray(value)) {
            dependArray(value)
          }
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val
      ...
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      // 如果新值是对象，也要响应式处理
      childOb = !shallow && observe(newVal)
      // 通知更新
      dep.notify()
    }
  })
}


```

data对象创建了一个Observer观察者，data对象中的数组和对象要分别进行处理。

data下的所有对象执行defineReactive方法，创建消息订阅器dep实例，通过Object.defineProperty方法把每个对象转换成get和set方法。当获取数据时执行get，如果存在依赖，则进行依赖收集，如果子子观察者存在也要进行依赖收集；当修改数据后执行set方法，执行通知更新dep.notify()被调用。