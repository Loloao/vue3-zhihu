# vue3-zhihu

## 新特性
- `setUp`
  相当于一个环境，里面写一些`hooks`，接受两个参数`props`和`context`，第二个参数用于获取`attrs`，`slot`，`emit`
- 由于 vue3 使用的是`proxy`，所以它能够对数组和对象新增的属性进行拦截，这和 vue2 是不同的，vue2 需要使用`this.$set()`将没有响应式的属性进行包装才能获得响应式
  ```javascript
  // vue2
  // 它只能对单个属性进行监听，而未来增加的属性不能监听
  Object.defineProperty(obj, prop, {
    get() {}
    set() {}
   })

  // vue3
  new Proxy(data, {
    get() {}
    set() {}
  })
  ```
  
- 生命周期
  
  在`setUp`函数中，可以使用以下生命周期函数来代替对应生命周期
  - `beforeCreate` => `use setup()`
  - `created` => `use setup()`
  - `beforeMount` => `onBeforeMount`
  - `mounted` => `onMounted`
  - `beforeUpdate` => `onBeforeUpdate`
  - `updated` => `onUpdated`
  - `beforeDestroy` => `onBeforeUnmount`
  - `destroyed` => `onUnmounted`
  - `activated` => `onActivated`
  - `deactivated` => `onDeactivated`
  - `errorCaptured` => `onErrorCaptured`
  新增两个生命周期函数，用于调试，跟踪数据变化，通过监听它提供的`event`参数
  - `onRenderTracked`
  - `onRenderTriggered`

- 监听器
  由于`setUp`只在初始化的时候执行一次，所以如果后面需要监听某个变量并处理副作用的话可以使用`watch`
  ```javascript
  setUp() {
    const greeting = ref('')
    // 接受两个参数，新值和旧值
    // 当监控多个值时，第一个值可以为数组
    // 如果是 ref，直接填入即可，而如果是 reactive 包裹的对象属性，则需要使用函数返回需要监控的属性
    watch(greeting, (newValue, oldValue) => {
      // 副作用
      document.title = '改标题'
    })
  }
  ```

- 为 Typescript 服务
  vue3 中为了让 Typescript 能够适应组件的定义，有一个`defineComponent`用于定义组件，它的功能只是返回`Component`
  ```javascript
  const comp = defineComponent({
    // 此时下列属性都能有自动补全
    name: 'hello-world',
    props: { msg: string }
  })
  export default comp
  ```
- `Teleport`其实就是 react 中的`Portals`
  ```javascript
  // 通过 teleport 包裹，to 属性接受一个 css 选择器表示选择到那个 dom 上
  <teleport to='#modal'>
  <div>dialog</div>
  </teleport>
  ```
- `Suspense`
  ```javascript
  defineComponent({
    setup() {
      return new Promise((resolve) => {
        resolve({
          result: 42
        })
      })
    }
  })
  <Suspense>
    <template #default>
      <async-show>
    </template>
    <template #fallback>
      <h1>loading</h1>
    </template>
  </Suspense>
  ```
- 全局 API修改
  因为 vue2 的全局 api 会直接更改 vue实例，所以会导致以下情况
  - 在单元测试中，全局配置非常容易污染全局环境
  - 不同 app 中，共享一份不同配置的 vue 对象，会变得非常困难
  ```javascript
  import {createApp} from 'vue'
  import App from './App'

  const app = createApp(App)
  app.config.isCustomElem = tag => {
    tag.startsWith('app-')
  }
  app.use(...)
  app.mixins(...)
  app.mount('#app')
  ```
  全局配置更改
  - `config.productioinTip`被删除
  - `config.ignoreElements`改为`config.isCustomElement`
  - `config.keyCodes`被删除
  其他没有修改vue全局对象的api改为直接引入，为了实现 api 的`tree shaking`
  `import {nextTick, observable} from 'vue'`
