# new Vue

`src/core/instance/index.js`

```js
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}
```

`Vue` 通过 `new` 关键字初始化，然后调用 `this.init` 方法



`/Users/takki/vue/src/core/instance/init.js`

``` js
  Vue.prototype._init = function (options?: Object) {
  	...
    initLifecycle(vm) 
    initEvents(vm) 
    initRender(vm) 
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')
    ...
  }
```



Vue 初始化过程：

1. 初始化生命周期（挂载属性）

   `/Users/takki/vue/src/core/instance/lifecycle.js`

   ```js
   export function initLifecycle (vm: Component) {
     ...
     // 为组件实例挂载相应属性，并初始化
     vm.$parent = parent
     vm.$root = parent ? parent.$root : vm
   
     vm.$children = []
     vm.$refs = {}
   
     vm._watcher = null
     vm._inactive = null
     vm._directInactive = false
     vm._isMounted = false
     vm._isDestroyed = false
     vm._isBeingDestroyed = false
   }
   ```

2. 初始化事件中心

   `/Users/takki/vue/src/core/instance/events.js`

   ```js 
   export function initEvents (vm: Component) {
     // 创建事件对象，存储事件
     vm._events = Object.create(null)
     // 系统事件标识位
     vm._hasHookEvent = false
     // init parent attached events
     const listeners = vm.$options._parentListeners
     if (listeners) {
       updateComponentListeners(vm, listeners)
     }
   }
   ```

3. 初始化渲染（初始化$slot 、 $createElement、$attrs、$listeners 等渲染属性）

   `/Users/takki/vue/src/core/instance/render.js`

   ```js
   export function initRender (vm: Component) {
     ...
     // 处理组件slot，返回slot插槽对象
     vm.$slots = resolveSlots(options._renderChildren, renderContext)
     vm.$scopedSlots = emptyObject
     // bind the createElement fn to this instance
     // so that we get proper render context inside it.
     // args order: tag, data, children, normalizationType, alwaysNormalize
     // internal version is used by render functions compiled from templates
     vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false)
     // normalization is always applied for the public version, used in
     // user-written render functions.
     vm.$createElement = (a, b, c, d) => createElement(vm, a, b, c, d, true)
     
     // $attrs & $listeners are exposed for easier HOC creation.
     // they need to be reactive so that HOCs using them are always updated
     const parentData = parentVnode && parentVnode.data
   
     /* istanbul ignore else */
     if (process.env.NODE_ENV !== 'production') {
       defineReactive(vm, '$attrs', parentData && parentData.attrs || emptyObject, () => {
         !isUpdatingChildComponent && warn(`$attrs is readonly.`, vm)
       }, true)
       defineReactive(vm, '$listeners', options._parentListeners || emptyObject, () => {
         !isUpdatingChildComponent && warn(`$listeners is readonly.`, vm)
       }, true)
     } else {
       defineReactive(vm, '$attrs', parentData && parentData.attrs || emptyObject, null, true)
       defineReactive(vm, '$listeners', options._parentListeners || emptyObject, null, true)
     }
   }
   ```

4. 调用了`beforeCreate`钩子函数

5. 初始化 inject （状态初始化之前）

   `/Users/takki/vue/src/core/instance/inject.js`

   ```js
   export function initInjections (vm: Component) {
     const result = resolveInject(vm.$options.inject, vm)
     if (result) {
       toggleObserving(false)
       Object.keys(result).forEach(key => {
         /* istanbul ignore else */
         if (process.env.NODE_ENV !== 'production') {
           defineReactive(vm, key, result[key], () => {
             warn(
               `Avoid mutating an injected value directly since the changes will be ` +
               `overwritten whenever the provided component re-renders. ` +
               `injection being mutated: "${key}"`,
               vm
             )
           })
         } else {
           defineReactive(vm, key, result[key])
         }
       })
       toggleObserving(true)
     }
   }
   ```

   

6. **初始化状态**

   `/Users/takki/vue/src/core/instance/state.js`

   ```js
   export function initState (vm: Component) {
     vm._watchers = []
     const opts = vm.$options
     if (opts.props) initProps(vm, opts.props)
     if (opts.methods) initMethods(vm, opts.methods)
     if (opts.data) {
       initData(vm)
     } else {
       observe(vm._data = {}, true /* asRootData */)
     }
     if (opts.computed) initComputed(vm, opts.computed)
     if (opts.watch && opts.watch !== nativeWatch) {
       initWatch(vm, opts.watch)
     }
   }
   ```

   

7. 初始化 provide（状态初始化之后）

   `/Users/takki/vue/src/core/instance/inject.js`

   ```js
   export function initProvide (vm: Component) {
     const provide = vm.$options.provide
     if (provide) {
       vm._provided = typeof provide === 'function'
         ? provide.call(vm)
         : provide
     }
   }
   ```

8. 调用了`created`钩子函数

