## 1.Vue3.0 响应式方面做了哪些改变

*   vue2在初始化的时候，对data中的每个属性使用definepropery调用getter和setter使之变为响应式对象。如果属性值为对象，还会递归调用defineproperty使之变为响应式对象。
*   vue3使用proxy对象重写响应式。proxy的性能本来比defineproperty好，proxy可以拦截属性的访问、赋值、删除等操作，不需要初始化的时候遍历所有属性，另外有多层属性嵌套的话，只有访问某个属性的时候，才会递归处理下一级的属性。

> 优势：  
> 可以监听动态新增的属性；  
> 可以监听删除的属性 ；  
> 可以监听数组的索引和 length 属性；



2.Vue 3.0 所采用的 Composition Api 与 Vue 2.x使用的Options Api 有什么区别？
--------------------------------------------------------------

### Options Api

> 包含一个描述组件选项（data、methods、props等）的对象 options；  
> API开发复杂组件，同一个功能逻辑的代码被拆分到不同选项 ；  
> 使用mixin重用公用代码，也有问题：命名冲突，数据来源不清晰；

### composition Api

> vue3 新增的一组 api，它是基于函数的 api，可以更灵活的组织组件的逻辑。  
> 解决options api在大型项目中，options api不好拆分和重用的问题。

3.Proxy 相对于 Object.defineProperty 有哪些优点？
-----------------------------------------

proxy的性能本来比defineproperty好，proxy可以拦截属性的访问、赋值、删除等操作，不需要初始化的时候遍历所有属性，另外有多层属性嵌套的话，只有访问某个属性的时候，才会递归处理下一级的属性。

> 可以监听数组变化  
> 可以劫持整个对象  
> 操作时不是对原对象操作,是 new Proxy 返回的一个新对象  
> 可以劫持的操作有 13 种

4.Vue3.0 在编译方面有哪些优化？
----------------------

*   vue.js 3.x中标记和提升所有的静态节点，diff的时候只需要对比动态节点内容；
*   Fragments（升级vetur插件): template中不需要唯一根节点，可以直接放文本或者同级标签
*   静态提升(hoistStatic),当使用 hoistStatic 时,所有静态的节点都被提升到 render 方法之外.只会在应用启动的时候被创建一次,之后使用只需要应用提取的静态节点，随着每次的渲染被不停的复用。
*   patch flag, 在动态标签末尾加上相应的标记,只能带 patchFlag 的节点才被认为是动态的元素,会被追踪属性的修改,能快速的找到动态节点,而不用逐个逐层遍历，提高了虚拟dom diff的性能。
*   缓存事件处理函数cacheHandler,避免每次触发都要重新生成全新的function去更新之前的函数
*   tree shaking 通过摇树优化核心库体积,减少不必要的代码量

5.Vue3.0 响应式api如何实现的
-------------------------

### 1. reactive

设置对象为响应式对象。接收一个参数，判断这参数是否是对象。不是对象则直接返回这个参数，不做响应式处理。  
创建拦截器handerler，设置get/set/deleteproperty。  
**get**  
收集依赖（track）；  
如果当前 key 的值是对象，则为当前 key 的对象创建拦截器 handler, 设置 get/set/deleteProperty；  
如果当前的 key 的值不是对象，则返回当前 key 的值。  
**set**  
设置的新值和老值不相等时，更新为新值，并触发更新（trigger）。  
**deleteProperty**  
当前对象有这个 key 的时候，删除这个 key 并触发更新（trigger）。

### 2. effect

接收一个函数作为参数。作用是：访问响应式对象属性时去收集依赖

### 3. track

接收两个参数：target 和 key  
－如果没有 activeEffect，则说明没有创建 effect 依赖  
－如果有 activeEffect，则去判断 WeakMap 集合中是否有 target 属性  
－WeakMap 集合中没有 target 属性，则 set(target, (depsMap = new Map()))  
－WeakMap 集合中有 target 属性，则判断 target 属性的 map 值的 depsMap 中是否有 key 属性  
－depsMap 中没有 key 属性，则 set(key, (dep = new Set()))  
－depsMap 中有 key 属性，则添加这个 activeEffect

### ４.trigger

判断 WeakMap 中是否有 target 属性，WeakMap 中有 target 属性，则判断 target 属性的 map 值中是否有 key 属性，有的话循环触发收集的 effect()。

## 6.Vue2.0 和 Vue3.0 有什么区别？

1.  响应式系统的重新配置，使用代理替换对象.define属性，使用代理优势：

*   可直接监控阵列类型的数据变化
*   监听的目标是对象本身，不需要像Object.defineProperty那样遍历每个属性，有一定的性能提升
*   可拦截应用、拥有密钥、有等13种方法，以及Object.define属性没有办法
*   直接添加对象属性/删除

1.  新增组合API，更好的逻辑重用和代码组织
2.  重构虚拟 DOM

*   模板编译时间优化，将一些静态节点编译成常量
*   slot优化，采取槽编译成懒人功能，拿槽渲染的决定留给子组件
*   在模板中提取和重用内联事件（最初，每次渲染时都会重新生成内联函数）

1.  代码结构调整，更方便树摇动，使其更小
2.  使用打字脚本替换流

7.Vue3带来了什么改变？
--------------

### 1.性能的提升

* 打包大小减少41%

* 初次渲染快55%, 更新渲染快133%

* 内存减少54%

  ......

### 2.源码的升级

* 使用Proxy代替[defineProperty](https://so.csdn.net/so/search?q=defineProperty&spm=1001.2101.3001.7020)实现响应式

* 重写[虚拟DOM](https://so.csdn.net/so/search?q=%E8%99%9A%E6%8B%9FDOM&spm=1001.2101.3001.7020)的实现和Tree-Shaking

  ......

### 3.拥抱TypeScript

*   Vue3可以更好的支持TypeScript

### 4.新的特性

1.  Composition API（组合API）

    *   setup配置

    *   ref与reactive

    *   watch与watchEffect

    *   provide与inject

    *   ......

2.  新的内置组件

    *   Fragment

    *   Teleport

    *   Suspense

3.  其他改变

    *   新的生命周期钩子

    *   data 选项应始终被声明为一个函数

    *   移除keyCode支持作为 v-on 的修饰符

    *   ......

4.vue3还有哪些其他改变？

* data选项应始终被声明为一个函数。

* 过度类名的更改：

  * Vue2.x写法

    ```
    .v-enter,
    .v-leave-to {
      opacity: 0;
    }
    .v-leave,
    .v-enter-to {
      opacity: 1;
    }
    ```

  * Vue3.x写法

    ```
    .v-enter-from,
    .v-leave-to {
      opacity: 0;
    }
    
    .v-leave-from,
    .v-enter-to {
      opacity: 1;
    }
    ```

* **移除**keyCode作为 v-on 的修饰符，同时也不再支持`config.keyCodes`

* **移除**`v-on.native`修饰符

  * 父组件中绑定事件

    ```
    <my-component
      v-on:close="handleComponentEvent"
      v-on:click="handleNativeClickEvent"
    />
    ```

  * 子组件中声明自定义事件

    ```
    <script>
      export default {
        emits: ['close']
      }
    </script>
    ```

* **移除**过滤器（filter）

  > 过滤器虽然这看起来很方便，但它需要一个自定义语法，打破大括号内表达式是 “只是 JavaScript” 的假设，这不仅有学习成本，而且有实现成本！建议用方法调用或计算属性去替换过滤器。

* ......

8.Vue 生命周期
----------------------------



**vue2.x的生命周期**

![](https://v2.cn.vuejs.org/images/lifecycle.png)

**vue3.0的生命周期**

![](https://cn.vuejs.org/assets/lifecycle.16e4c08e.png)

*   Vue3.0中可以继续使用Vue2.x中的生命周期钩子，但有有两个被更名：

    *   `beforeDestroy`改名为 `beforeUnmount`

    *   `destroyed`改名为 `unmounted`

*   Vue3.0也提供了 Composition API 形式的生命周期钩子，与Vue2.x中钩子对应关系如下：

    *   `beforeCreate`===>`setup()`

    *   `created`=======>`setup()`

    *   `beforeMount` ===>`onBeforeMount`

    *   `mounted`=======>`onMounted`

    *   `beforeUpdate`===>`onBeforeUpdate`

    *   `updated` =======>`onUpdated`

    *   `beforeUnmount` ==>`onBeforeUnmount`

    *   `unmounted` =====>`onUnmounted`

9.Vue3.0中的响应式原理是什么？vue2的响应式原理是什么？
------------------------------------------------------------------------------------------------------------------------

### vue2.x的响应式

* 实现原理：

  * 对象类型：通过`Object.defineProperty()`对属性的读取、修改进行拦截（数据劫持）。

  * 数组类型：通过重写更新数组的一系列方法来实现拦截。（对数组的变更方法进行了包裹）。

    ```
    Object.defineProperty(data, 'count', {
        get () {}, 
        set () {}
    })
    ```

* 存在问题：

  *   新增属性、删除属性, 界面不会更新。

  *   直接通过下标修改数组, 界面不会自动更新。

### Vue3.0的响应式

* 实现原理:

  * 通过Proxy（代理）: 拦截对象中任意属性的变化, 包括：属性值的读写、属性的添加、属性的删除等。

  * 通过Reflect（反射）: 对源对象的属性进行操作。

  * MDN文档中描述的Proxy与Reflect：

    * Proxy：[Proxy - JavaScript | MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy "Proxy - JavaScript | MDN")

    * Reflect：[Reflect - JavaScript | MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect "Reflect - JavaScript | MDN")

      ```
      new Proxy(data, {
          // 拦截读取属性值
          get (target, prop) {
              return Reflect.get(target, prop)
          },
          // 拦截设置属性值或添加新属性
          set (target, prop, value) {
              return Reflect.set(target, prop, value)
          },
          // 拦截删除属性
          deleteProperty (target, prop) {
              return Reflect.deleteProperty(target, prop)
          }
      })
      
      proxy.name = 'tom'   ![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)
      ```

10.vue3响应式数据的判断？
---------------

*   isRef: 检查一个值是否为一个 ref 对象

*   isReactive: 检查一个对象是否是由 `reactive` 创建的响应式代理

*   isReadonly: 检查一个对象是否是由 `readonly` 创建的只读代理

*   isProxy: 检查一个对象是否是由 `reactive` 或者 `readonly` 方法创建的代理

## 11.vue3的常用 Composition API有哪些？

官方文档: [介绍 | Vue.js](https://v3.cn.vuejs.org/guide/composition-api-introduction.html "介绍 | Vue.js")

1.拉开序幕的setup
------------

1.  理解：Vue3.0中一个新的配置项，值为一个函数。

2.  setup是所有**Composition API（组合API）**_“ 表演的舞台 ”_。

3.  组件中所用到的：数据、方法等等，均要配置在setup中。

4.  setup函数的两种返回值：

    1.  若返回一个对象，则对象中的属性、方法, 在模板中均可以直接使用。（重点关注！）

    2.  若返回一个渲染函数：则可以自定义渲染内容。（了解）

5.setup的几个注意点

*   setup执行的时机

    *   在beforeCreate之前执行一次，this是undefined。

*   setup的参数

    *   props：值为对象，包含：组件外部传递过来，且组件内部声明接收了的属性。

    *   context：上下文对象

        *   attrs: 值为对象，包含：组件外部传递过来，但没有在props配置中声明的属性, 相当于 `this.$attrs`。

        *   slots: 收到的插槽内容, 相当于 `this.$slots`。

        *   emit: 分发自定义事件的函数, 相当于 `this.$emit`。

*   尽量不要与Vue2.x配置混用

    *   Vue2.x配置（data、methos、computed...）中**可以访问到**setup中的属性、方法。

    *   但在setup中**不能访问到**Vue2.x配置（data、methos、computed...）。

    *   如果有重名, setup优先。

*   setup不能是一个async函数，因为返回值不再是return的对象, 而是promise, 模板看不到return对象中的属性。（后期也可以返回一个Promise实例，但需要Suspense和异步组件的配合）

2.ref函数
-------

*   作用: 定义一个响应式的数据

*   语法: `const xxx = ref(initValue)`

    *   创建一个包含响应式数据的**引用对象（reference对象，简称ref对象）**。

    *   JS中操作数据： `xxx.value`

    *   模板中读取数据: 不需要.value，直接：`<div>{{xxx}}</div>`

*   备注：

    *   接收的数据可以是：基本类型、也可以是对象类型。

    *   基本类型的数据：响应式依然是靠`Object.defineProperty()`的`get`与`set`完成的。

    *   对象类型的数据：内部 _“ 求助 ”_ 了Vue3.0中的一个新函数—— `reactive`函数。

3.reactive函数
------------

*   作用: 定义一个**对象类型**的响应式数据（基本类型不要用它，要用`ref`函数）

*   语法：`const 代理对象= reactive(源对象)`接收一个对象（或数组），返回一个**代理对象（Proxy的实例对象，简称proxy对象）**

*   reactive定义的响应式数据是“深层次的”。

*   内部基于 ES6 的 Proxy 实现，通过代理对象操作源对象内部数据进行操作。

4.reactive对比ref
---------------

*   从定义数据角度对比：

    *   ref用来定义：**基本类型数据**。

    *   reactive用来定义：**对象（或数组）类型数据**。

    *   备注：ref也可以用来定义**对象（或数组）类型数据**, 它内部会自动通过`reactive`转为**代理对象**。

*   从原理角度对比：

    *   ref通过`Object.defineProperty()`的`get`与`set`来实现响应式（数据劫持）。

    *   reactive通过使用**Proxy**来实现响应式（数据劫持）, 并通过**Reflect**操作**源对象**内部的数据。

*   从使用角度对比：

    *   ref定义的数据：操作数据**需要**`.value`，读取数据时模板中直接读取**不需要**`.value`。

    *   reactive定义的数据：操作数据与读取数据：**均不需要**`.value`。

5.计算属性与监视
---------

### 1.computed函数

* 与Vue2.x中computed配置功能一致

* 写法

  ```
  import {computed} from 'vue'
  
  setup(){
      ...
      //计算属性——简写
      let fullName = computed(()=>{
          return person.firstName + '-' + person.lastName
      })
      //计算属性——完整
      let fullName = computed({
          get(){
              return person.firstName + '-' + person.lastName
          },
          set(value){
              const nameArr = value.split('-')
              person.firstName = nameArr[0]
              person.lastName = nameArr[1]
          }
      })
  }![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)
  ```

### 2.watch函数

* 与Vue2.x中watch配置功能一致

* 两个小“坑”：

  *   监视reactive定义的响应式数据时：oldValue无法正确获取、强制开启了深度监视（deep配置失效）。

  *   监视reactive定义的响应式数据中某个属性时：deep配置有效。

  ```
  //情况一：监视ref定义的响应式数据
  watch(sum,(newValue,oldValue)=>{
      console.log('sum变化了',newValue,oldValue)
  },{immediate:true})
  
  //情况二：监视多个ref定义的响应式数据
  watch([sum,msg],(newValue,oldValue)=>{
      console.log('sum或msg变化了',newValue,oldValue)
  }) 
  
  /* 情况三：监视reactive定义的响应式数据
              若watch监视的是reactive定义的响应式数据，则无法正确获得oldValue！！
              若watch监视的是reactive定义的响应式数据，则强制开启了深度监视 
  */
  watch(person,(newValue,oldValue)=>{
      console.log('person变化了',newValue,oldValue)
  },{immediate:true,deep:false}) //此处的deep配置不再奏效
  
  //情况四：监视reactive定义的响应式数据中的某个属性
  watch(()=>person.job,(newValue,oldValue)=>{
      console.log('person的job变化了',newValue,oldValue)
  },{immediate:true,deep:true}) 
  
  //情况五：监视reactive定义的响应式数据中的某些属性
  watch([()=>person.job,()=>person.name],(newValue,oldValue)=>{
      console.log('person的job变化了',newValue,oldValue)
  },{immediate:true,deep:true})
  
  //特殊情况
  watch(()=>person.job,(newValue,oldValue)=>{
      console.log('person的job变化了',newValue,oldValue)
  },{deep:true}) //此处由于监视的是reactive素定义的对象中的某个属性，所以deep配置有效![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)
  ```

### 3.watchEffect函数

* watch的套路是：既要指明监视的属性，也要指明监视的回调。

* watchEffect的套路是：不用指明监视哪个属性，监视的回调中用到哪个属性，那就监视哪个属性。

* watchEffect有点像computed：

  *   但computed注重的计算出来的值（回调函数的返回值），所以必须要写返回值。

  *   而watchEffect更注重的是过程（回调函数的函数体），所以不用写返回值。

  ```
  //watchEffect所指定的回调中用到的数据只要发生变化，则直接重新执行回调。
  watchEffect(()=>{
      const x1 = sum.value
      const x2 = person.age
      console.log('watchEffect配置的回调执行了')
  })
  ```

## 12.toRef 代表什么意思

*   作用：创建一个 ref 对象，其value值指向另一个对象中的某个属性。

*   语法：`const name = toRef(person,'name')`

*   应用: 要将响应式对象中的某个属性单独提供给外部使用时。

*   扩展：`toRefs` 与`toRef`功能一致，但可以批量创建多个 ref 对象，语法：`toRefs(person)`

## 13.toRaw 代表什么意思

*   作用：将一个由`reactive`生成的**响应式对象**转为**普通对象**。

*   使用场景：用于读取响应式对象对应的普通对象，对这个普通对象的所有操作，不会引起页面更新。

## 14.provide 与 inject

![](https://img-blog.csdnimg.cn/img_convert/b40f40b6d0ea19db114649b400d8e29f.png)

* 作用：实现**祖与后代组件间**通信

* 套路：父组件有一个 `provide` 选项来提供数据，后代组件有一个 `inject` 选项来开始使用这些数据

* 具体写法：

  1. 祖组件中：

     ```
     setup(){
         ......
         let car = reactive({name:'奔驰',price:'40万'})
         provide('car',car)
         ......
     }
     ```

  2. 后代组件中：

     ```
     setup(props,context){
         ......
         const car = inject('car')
         return {car}
         ......
     }
     ```

15、vue3为什么要添加新的Composition API，它可以解决哪些问题
-----------------------------

在 Vue2.0 中，随着功能的增加，组件越来越复杂，维护起来也越来越难，而难以维护的根本原因是 Vue 的 API 设计迫使开发者使用监视、计算、方法 Option 组织代码，而不是实际的业务逻辑。

另外 Vue2.0 缺乏一个简单而低成本的机制来完成逻辑重用，虽然你可以 minxis 完全重用逻辑，但是当 mixin 更多的时候，就使得很难找到相应的数据，计算出来也许是从中 mixin 的方法，使得类型推断变得困难。

因此组合API外观，主要是解决选项API带来的问题，首先是代码组织，组合API开发者可以根据业务逻辑组织自己的代码，让代码更具可读性和可扩展性，也就是说，当下一个开发者接触到这段不是自己写的代码， 他可以更好地利用代码的组织来反转实际的业务逻辑，或者根据业务逻辑更好地理解代码。

二是实现代码的逻辑提取和重用，当然mixin逻辑提取和重用也可以实现，但就像我之前说的，多个mixin在作用于同一个组件时，很难看出mixin的属性，来源不明确，另外，多个mixin的属性存在变量命名冲突的风险。而 Composition API 恰恰解决了这两个问题。

16、什么是hook？什么是自定义hook函数？
-----------------------

*   什么是hook？—— 本质是一个函数，把setup函数中使用的Composition API进行了封装。

*   类似于vue2.x中的mixin。

*   自定义hook的优势: 复用代码, 让setup中的逻辑更清楚易懂。

17、都说 Composition API 和 React Hook 很像，请问他们的区别是什么？
------------------------------------------------

从 React Hook 从实现的角度来看，React Hook 是基于 useState 的调用顺序来确定下一个 re 渲染时间状态从哪个 useState 开始，所以有以下几个限制

*   不在循环中、条件、调用嵌套函数 Hook
*   你必须确保它总是在你这边 React Top level 调用函数 Hook
*   使用效果、使用备忘录 依赖关系必须手动确定

和 Composition API 是基于 Vue 的响应系统，和 React Hook 相比

*   在设置函数中，一个组件实例只调用一次设置，而 React Hook 每次重新渲染时，都需要调用 Hook，给 React 带来的 GC 比 Vue 更大的压力，性能也相对 Vue 对我来说也比较慢
*   Compositon API 你不必担心调用的顺序，它也可以在循环中、条件、在嵌套函数中使用
*   响应式系统自动实现依赖关系收集，而且组件的性能优化是由 Vue 内部完成的，而 React Hook 的依赖关系需要手动传递，并且依赖关系的顺序必须得到保证，让路 useEffect、useMemo 等等，否则组件性能会因为依赖关系不正确而下降。

虽然Compoliton API看起来像React Hook来使用，但它的设计思路也是React Hook的参考。

## 18、Options API 存在的问题是什么？Composition API 的优势有哪些？

### 1.Options API 存在的问题

使用传统OptionsAPI中，新增或者修改一个需求，就需要分别在data，methods，computed里修改 。



![](https://img-blog.csdnimg.cn/img_convert/1c05fac96e45cc122ab3206fcdbc4aa8.gif)

![](https://img-blog.csdnimg.cn/img_convert/74b4733cb731be4bec2c3afa812fdbe4.gif)

### 2.Composition API 的优势

我们可以更加优雅的组织我们的代码，函数。让相关功能的代码更加有序的组织在一起。



![](https://img-blog.csdnimg.cn/img_convert/8f74725c96df6d4ce5e98c8525ad0a75.gif)

![](https://img-blog.csdnimg.cn/img_convert/f2f235b92bce00d7e8459b1900d3ee7e.gif)

## 19、vue3有哪些新的组件？

### 1.Fragment

*   在Vue2中: 组件必须有一个根标签

*   在Vue3中: 组件可以没有根标签, 内部会将多个标签包含在一个Fragment虚拟元素中

*   好处: 减少标签层级, 减小内存占用

### 2.Teleport

* 什么是Teleport？—— `Teleport` 是一种能够将我们的**组件html结构**移动到指定位置的技术。

  ```
  <teleport to="移动位置">
      <div v-if="isShow" class="mask">
          <div class="dialog">
              <h3>我是一个弹窗</h3>
              <button @click="isShow = false">关闭弹窗</button>
          </div>
      </div>
  </teleport>
  ```

### 3.Suspense

* 等待异步组件时渲染一些额外内容，让应用有更好的用户体验

* 使用步骤：

  * 异步引入组件

    ```
    import {defineAsyncComponent} from 'vue'
    const Child = defineAsyncComponent(()=>import('./components/Child.vue'))
    ```

  * 使用`Suspense`包裹组件，并配置好`default` 与 `fallback`

    ```
    <template>
        <div class="app">
            <h3>我是App组件</h3>
            <Suspense>
                <template v-slot:default>
                    <Child/>
                </template>
                <template v-slot:fallback>
                    <h3>加载中.....</h3>
                </template>
            </Suspense>
        </div>
    </template>
    ```

20.vue2和vue3的全局 API 和配置区别？
--------------------------

* Vue 2.x 有许多全局 API 和配置。

  * 例如：注册全局组件、注册全局指令等。

    ```
    //注册全局组件
    Vue.component('MyButton', {
      data: () => ({
        count: 0
      }),
      template: '<button @click="count++">Clicked {{ count }} times.</button>'
    })
    
    //注册全局指令
    Vue.directive('focus', {
      inserted: el => el.focus()
    }
    ```

* Vue3.0中对这些API做出了调整：全局API的转移

  * 将全局的API，即：`Vue.xxx`调整到应用实例（`app`）上

    <table><thead><tr><th>2.x 全局 API（<code>Vue</code>）</th><th>3.x 实例 API (<code>app</code>)</th></tr></thead><tbody><tr><td>Vue.config.xxxx</td><td>app.config.xxxx</td></tr><tr><td>Vue.config.productionTip</td><td><strong>移除</strong></td></tr><tr><td>Vue.component</td><td>app.component</td></tr><tr><td>Vue.directive</td><td>app.directive</td></tr><tr><td>Vue.mixin</td><td>app.mixin</td></tr><tr><td>Vue.use</td><td>app.use</td></tr><tr><td>Vue.prototype</td><td>app.config.globalProperties</td></tr></tbody></table>

