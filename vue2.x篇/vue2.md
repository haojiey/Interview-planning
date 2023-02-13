# vue篇

一、`MVVM`原理
----------

在`Vue2`官方文档中没有找到`Vue`是`MVVM`的直接证据，但文档有提到：虽然没有完全遵循`MVVM模型`，但是 Vue 的设计也受到了它的启发，因此在文档中经常会使用`vm`(ViewModel 的缩写) 这个变量名表示 Vue 实例。

为了感受`MVVM模型`的启发，我简单列举下其概念。

MVVM是Model-View-ViewModel的简写，由三部分构成：

*   Model: 模型持有所有的数据、状态和程序逻辑
*   View: 负责界面的布局和显示
*   ViewModel：负责模型和界面之间的交互，是Model和View的桥梁

二、`SPA`单页面应用
------------

单页Web应用（single page web application，SPA），就是只有一张Web页面的应用，是加载单个HTML页面并在用户与应用程序交互时动态更新该页面的Web应用程序。我们开发的`Vue`项目大多是借助个官方的`CLI`脚手架，快速搭建项目，直接通过`new Vue`构建一个实例，并将`el:'#app'`挂载参数传入，最后通过`npm run build`的方式打包后生成一个`index.html`，称这种只有一个`HTML`的页面为单页面应用。

当然，`vue`也可以像`jq`一样引入，作为多页面应用的基础框架。

三、`Vue`的特点
----------

*   清晰的官方文档和好用的`api`，比较容易上手。
*   是一套用于构建用户界面的**渐进式框架**，将注意力集中保持在核心库，而将其他功能如路由和全局状态管理交给相关的库。
*   使用 **Virtual DOM**。
*   提供了**响应式** (Reactive) 和**组件化** (Composable) 的视图组件。

四、`Vue`的构建入口
------------

vue使用过程中可以采用以下两种方式：

*   在vue脚手架中直接使用，参考文档：`https://cn.vuejs.org/v2/guide/installation.html`
*   或者在html文件的头部通过静态文件的方式引入： `<script src="https://cdn.jsdelivr.net/npm/vue@2.6.14/dist/vue.js"></script>`

那么问题来了，使用的或者引入的到底是什么？  
答：引入的是已经打包好的vue.js文件，通过rollup构建打包所得。

构建入口在哪里？  
答：在`vue`源码的package.json文件中：

```
"scripts": {
    // ...
    "build": "node scripts/build.js",
    "build:ssr": "npm run build -- web-runtime-cjs,web-server-renderer",
    "build:weex": "npm run build -- weex",
    // ...
  },
复制代码
```

通过执行npm run build的时候，会进行scripts/build.js文件的执行，npm run build：ssr和npm run build：weex的时候，将ssr和weex作为参数传入，按照参数构建出不一样的vue.js打包文件。

所以说，`vue`中的`package.json`文件就是构建的入口，具体构建流程可以参考[vue2入口：构建入口](https://juejin.cn/post/7128225931293884452 "https://juejin.cn/post/7128225931293884452")。

五、对`import Vue from "vue"`的理解
-----------------------------

在使用脚手架开发项目时，会有一行代码`import Vue from "vue"`，那么这个`Vue`指的是什么。  
答：一个构造函数。

```
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}
initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)
复制代码
```

我们开发中引入的`Vue`其实就是这个构造函数，而且这个构造函数只能通过`new Vue`的方式进行使用，否则会在控制台打印警告信息。定义完后，还会通过`initMixin(Vue)`、`stateMixin(Vue)`、`eventsMixin(Vue)`、`lifecycleMixin(Vue)`和`renderMixin(Vue)`的方式为`Vue`原型中混入方法。我们通过`import Vue from "Vue"`引入的本质上就是一个原型上挂在了好多方法的构造函数。

六、对`new Vue`的理解
---------------

```
// main.js文件
import Vue from "vue";
var app = new Vue({
  el: '#app',
  data() {
    return {
      msg: 'hello Vue~'
    }
  },
  template: `<div>{{msg}}</div>`,
})

console.log(app);
复制代码
```

`new Vue`就是对构造函数`Vue`进行实例化，执行结果如下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b68a5e0eeda4dc28bd6ca3f2ad0c167~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

可以看出实例化后的实例中包含了很多属性，用来对当前`app`进行描述，当然复杂的`Vue`项目这个`app`将会是一个树结构，通过`$parent`和`$children`维护父子关系。

`new Vue`的过程中还会执行`this._init`方法进行初始化处理。

七、`编译`
------

虚拟`DOM`的生成必须通过`render`函数实现，`render`函数的产生是在编译阶段完成，核心代码如下：

```
export const createCompiler = createCompilerCreator(function baseCompile (
  template: string,
  options: CompilerOptions
): CompiledResult {
  const ast = parse(template.trim(), options)
  if (options.optimize !== false) {
    optimize(ast, options)
  }
  const code = generate(ast, options)
  return {
    ast,
    render: code.render,
    staticRenderFns: code.staticRenderFns
  }
})
复制代码
```

主要完成的功能是：

*   通过`const ast = parse(template.trim(), options)`将`template`转换成`ast`树
*   通过`optimize(ast, options)`对`ast`进行优化
*   通过`const code = generate(ast, options)`将优化后的`ast`转换成包含`render`字符串的`code`对象，最终`render`字符串通过`new Function`转换为可执行的`render`函数

`模板编译的真实入口`可以参考[vue2从template到render：模板编译入口](https://link.juejin.cn?target=https%3A%2F%2Fjuejin.cn%2Fpost%2F7134253213871505415 "https://juejin.cn/post/7134253213871505415")  
`parse`可以参考[vue2从template到render：AST](https://juejin.cn/post/7134722260870365198 "https://juejin.cn/post/7134722260870365198")  
`optimize`可以参考[vue2从template到render：optimize](https://link.juejin.cn?target=https%3A%2F%2Fjuejin.cn%2Fpost%2F7136206751602311175 "https://juejin.cn/post/7136206751602311175")  
`generate`可以参考[vue2从template到render：code](https://juejin.cn/post/7135783392133513252 "https://juejin.cn/post/7135783392133513252")

八、虚拟`DOM`
---------

**先看浏览器对`HTML`的理解**：

```
<div>  
    <h1>My title</h1>  
    Some text content  
    <!-- TODO: Add tagline -->  
</div>
复制代码
```

当浏览器读到这些代码时，它会建立一个DOM树来保持追踪所有内容，如同你会画一张家谱树来追踪家庭成员的发展一样。 上述 HTML 对应的 DOM 节点树如下图所示：

![544ef95bdd7c96a19d700ce613ab425a_dom-tree.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cc218300418a46119b2b26f0d9269343~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?) 每个元素都是一个节点。每段文字也是一个节点。甚至注释也都是节点。一个节点就是页面的一个部分。就像家谱树一样，每个节点都可以有孩子节点 (也就是说每个部分可以包含其它的一些部分)。

**再看`Vue`对`HTML template`的理解**

Vue 通过建立一个**虚拟 DOM** 来追踪自己要如何改变真实 DOM。因为它所包含的信息会告诉 Vue 页面上需要渲染什么样的节点，包括及其子节点的描述信息。我们把这样的节点描述为“虚拟节点 (virtual node)”，也常简写它为“**VNode**”。“虚拟 DOM”是我们对由 Vue 组件树建立起来的整个 VNode 树的称呼。

简言之，浏览器对HTML的理解是DOM树，Vue对`HTML`的理解是虚拟DOM，最后在`patch`阶段通过DOM操作的api将其渲染成真实的DOM节点。

九、模板或者组件渲染
----------

`Vue`中的编译会执行到逻辑`vm._update(vm._render(), hydrating)`，其中的`vm._render`执行会获取到`vNode`，`vm._update`就会对`vNode`进行`patch`的处理，又分为模板渲染和组件渲染。

*   模板渲染，参考[vue2从数据到视图渲染：模板渲染](https://link.juejin.cn?target=https%3A%2F%2Fjuejin.cn%2Fpost%2F7129095821261275167 "https://juejin.cn/post/7129095821261275167")
*   组件渲染，参考[vue2从数据到视图渲染：组件渲染](https://juejin.cn/post/7129095821261275167 "https://juejin.cn/post/7129095821261275167")

十、数据响应式处理
---------

`Vue`的数据响应式处理的核心是`Object.defineProperty`，在递归响应式处理对象的过程中，为每一个属性定义了一个发布者`dep`，当进行`_render`函数执行时会访问到当前值，在`get`中通过`dep.depend`进行当前`Watcher`的收集，当数据发生变化时会在`set`中通过`dep.notify`进行`Watcher`的更新。

数据响应式处理以及发布订阅者模式的关系请参考[vue2从数据变化到视图变化：发布订阅模式](https://link.juejin.cn?target=https%3A%2F%2Fjuejin.cn%2Fpost%2F7130991439965585444 "https://juejin.cn/post/7130991439965585444")

十一、`this.$set`
--------------

```
const app = new Vue({
  el: "#app",
  data() {
    return {
      obj: {
        name: "name-1"
      }
    };
  },
  template: `<div @click="change">{{obj.name}}的年龄是{{obj.age}}</div>`,
  methods: {
    change() {
      this.obj.name = 'name-2';
      this.obj.age = 30;
    }
  }
});
复制代码
```

以上例子执行的结果是：  
name-1的年龄是  
当点击后依然是：  
name-2的年龄是  
可以看出点击后，`obj`的`name`属性变化得到了视图更新，而`age`属性并未进行变化。

`name`属性响应式的过程中锁定了一个发布者`dep`，在当前视图渲染时在发布者`dep`的`subs`中做了记录，一旦其发生改变，就会触发`set`方法中的`dep.notify`，继而执行视图的重新渲染。然而，`age`属性并未进行响应式的处理，当其改变时就不能进行视图渲染。

此时就需要通过`this.$set`的方式对其进行手动响应式的处理。具体细节请参考 [手动响应式处理和数组检测变化](https://juejin.cn/post/7131280408385159175 "https://juejin.cn/post/7131280408385159175")

十二、组件注册
-------

组件的使用是先注册后使用，又分为：

*   全局注册：可以直接在页面中使用
*   局部注册：使用时需要通过`import xxx from xxx`的方式引入，并且在当前组件的选项`components`中增加局部组件的名称。

全局注册和局部注册实现原理可以参考 [vue2从数据到视图渲染：组件注册(全局组件/局部组件)](https://link.juejin.cn?target=https%3A%2F%2Fjuejin.cn%2Fpost%2F7129338204448260109 "https://juejin.cn/post/7129338204448260109")

十三、异步组件
-------

Vue单页面应用中一个页面只有一个`<div id="app"></div>`承载所有节点，因此复杂项目可能会出现首屏加载白屏等问题，Vue异步组件就很好的处理了这问题。

异步组件的分类和实现原理请参考 [vue2从数据到视图渲染：异步组件](https://juejin.cn/post/7129784993663942693 "https://juejin.cn/post/7129784993663942693")

十四、`this.$nextTick`
-------------------

因为通过`new`实例化构造函数`Vue`的时候会执行初始化方法`this._init`，其中涉及到的方法大多都是同步执行。`nextTick`在vue中是一个很重要的方法，在`new Vue`实例化的同步过程中将一些需要异步处理的函数推到异步队列中去，可以等`new Vue`所有的同步任务执行完后，再执行异步队列中的函数。

`nextTick`的实现可以参考 [vue2从数据变化到视图变化：nextTick](https://link.juejin.cn?target=https%3A%2F%2Fjuejin.cn%2Fpost%2F7129855050259628068 "https://juejin.cn/post/7129855050259628068")，

十五、`keep-alive`内置组件
-------------------

`vue`中支持组件化，并且也有用于缓存的内置组件`keep-alive`可直接使用，使用场景为`路由组件`和`动态组件`。

*   `activated`表示进入组件的生命周期，`deactivated`表示离开组件的生命周期
*   `include`表示匹配到的才缓存，`exclude`表示匹配到的都不缓存
*   `max`表示最多可以缓存多少组件

`keep-alive`的具体实现请参考 [vue中的keep-alive（源码分析）](https://juejin.cn/post/7155828702356439076/ "https://juejin.cn/post/7155828702356439076/")

十六、生命周期
-------

`vue`中的生命周期有哪些？  
答案：`11`个，分别为`beforeCreate`、`created`、`beforeMount`、`mounted`、`beforeUpdate`、`updated`、`activated`、`deactivated`、`beforeDestroy`、`destroyed`和`errorCaptured`。

具体实现请参考 [vue生命周期](https://link.juejin.cn?target=https%3A%2F%2Fjuejin.cn%2Fpost%2F7140218856152236068 "https://juejin.cn/post/7140218856152236068")

十七、`v-show`和`v-if`的区别
---------------------

先看`v-if`和`v-show`的使用场景：

（1）`v-if`更多的使用在需要考虑白屏时间或者切换次数很少的场景  
（2）`v-show`更多使用在`transition`控制的动画或者需要非常频繁地切换的场景

再从底层实现思路上分析：

（1）`v-if`条件为`false`时，会生成空的占位注释节点，那么在考虑首页白屏时间时，选用`v-if`比较合适。条件从`false`变化为`true`的话会从空的注释节点变成真实节点，条件再变为`false`时真实节点又会变成注释节点，如果切换次数比较多，那么开销会比较大，频繁切换场景不建议使用`v-if`。  
（2）`v-show`条件为`false`时，会生成真实的节点，只是为当前节点增加了`display:none`来控制其隐藏，相比`v-if`生成空的注释节点其首次渲染开销是比较大的，所以不建议用在考虑首屏白屏时间的场景。如果我们频繁切换`v-show`的值，从`display:none`到`display:block`之间的切换比起空的注释节点和真实节点的开销要小很多，这种场景就建议使用`v-show`。

可以通过[vue中v-if和v-show的区别（源码分析）](https://juejin.cn/post/7139815983979757575 "https://juejin.cn/post/7139815983979757575")了解`v-if`和`v-show`详细过程。

十八、`v-for`中`key`的作用
-------------------

在`v-for`进行循环展示过程中，当数据发生变化进行渲染的过程中，会进行新旧节点列表的比对。首先新旧`vnode`列表首先通过`首首`、`尾尾`、`首尾`和`尾首`的方式进行比对，如果`key`相同则采取原地复用的策略进行节点的移动。

如果首尾两两比对的方式找不到对应关系，继续通过`key`和`vnode`的对应关系进行寻找。

如果`key`和`vnode`对应关系中找不到，继续通过`sameVnode`的方式在未比对的节点中进行寻找。

如果都找不到，则将其按照新`vnode`进行`createElm`的方式进行创建，这种方式是比节点移动的方式计算量更大。

最后将旧的`vnode`列表中没有进行匹配的`vnode`中的`vnode.elm`在父节点中移除。

简单总结就是，新的`vnode`列表在旧的`vnode`列表中去寻找具有相同的`key`的节点进行原地复用，如果找不到则通过创建的方式`createElm`去创建一个，如果旧的`vnode`列表中没有进行匹配则在父节点中移除其`vnode.elm`。这就是原地复用逻辑的大体实现。

具体`key`和`diff`算法的关系可以参考[vue2从数据变化到视图变化：diff算法图解](https://link.juejin.cn?target=https%3A%2F%2Fjuejin.cn%2Fpost%2F7131760211609518094 "https://juejin.cn/post/7131760211609518094")

十九、`v-for`和`v-if`能同时使用吗
-----------------------

答案是：用了也能出来预期的效果，但是会有性能浪费。

同时包含`v-for`和`v-if`的`template`模板在编辑阶段会执行`v-for`比`v-if`优先级更高的编译流程；在生成`vnode`的阶段，会包含属性`isComment`为`true`的空白占位`vnode`；在`patch`阶段，会生成真实的占位节点。虽然一个空的占位节点无妨，但是如果数据量比较大的话，也是一个性能问题。

当然，可以在获取到数据(一般是在`beforeCreate`或者`created`阶段)时进行过滤处理，也可以通过计算属性对其进行处理。

可以通过[`v-for`和`v-if`可以一起使用吗？](https://juejin.cn/post/7137326610822201357 "https://juejin.cn/post/7137326610822201357")了解`v-for`和`v-if`的详细过程。

二十、`vue`中的`data`为什么是函数
----------------------

答案是：是不是一定是函数，得看场景。并且，也无需担心什么时候该将`data`写为函数还是对象，因为`vue`内部已经做了处理，并在控制台输出错误信息。

**场景一**：`new Vue({data: ...})`  
这种场景主要为项目入口或者多个`html`页面各实例化一个`Vue`时，这里的`data`即可用对象的形式，也可用工厂函数返回对象的形式。因为，这里的`data`只会出现一次，不存在重复引用而引起的数据污染问题。

**场景二**：组件场景中的选项  
在生成组件`vnode`的过程中，组件会在生成构造函数的过程中执行合并策略：

```
// data合并策略
strats.data = function (
  parentVal,
  childVal,
  vm
) {
  if (!vm) {
    if (childVal && typeof childVal !== 'function') {
      process.env.NODE_ENV !== 'production' && warn(
        'The "data" option should be a function ' +
        'that returns a per-instance value in component ' +
        'definitions.',
        vm
      );

      return parentVal
    }
    return mergeDataOrFn(parentVal, childVal)
  }

  return mergeDataOrFn(parentVal, childVal, vm)
};
复制代码
```

如果合并过程中发现子组件的数据不是函数，即`typeof childVal !== 'function'`成立，进而在开发环境会在控制台输出警告并且直接返回`parentVal`，说明这里压根就没有把`childVal`中的任何`data`信息合并到`options`中去。

可以通过[`vue`中的`data`为什么是函数？](https://link.juejin.cn?target=https%3A%2F%2Fjuejin.cn%2Fpost%2F7139925047145316360 "https://juejin.cn/post/7139925047145316360")了解详细过程。

二十一、`this.$watch`
-----------------

使用场景：用来监听数据的变化，当数据发生变化的时候，可以做一些业务逻辑的处理。

配置参数：

*   `deep`：监听数据的深层变化
*   `immediate`：立即触发回调函数

实现思路： `Vue`构造函数定义完成以后，在执行`stateMixin(Vue)`时为`Vue.prototype`上定义`$watch`。该方法通过`const watcher = new Watcher(vm, expOrFn, cb, options)`进行`Watcher`的实例化，将`options`中的`user`属性设置为`true`。并且，`$watch`逻辑结束的会返回函数`function unwatchFn () { watcher.teardown() }`，用来取消侦听的函数。

可以通过`watch`选项和`$watch`方法的区别[vue中的watch和$watch监听的事件，执行几次？](https://juejin.cn/post/7156172339414040584 "https://juejin.cn/post/7156172339414040584")来了解详细过程。

二十二、计算属性和侦听属性的区别
----------------

**相同点：** 两者都是`Watcher`实例化过程中的产物

**计算属性：**

*   使用场景：模板内的表达式主要用于简单运算，对于复杂的计算逻辑可以用计算属性
*   计算属性是基于它们的响应式依赖进行缓存的，当依赖的数据未发生变化时，多次调用无需重复执行函数
*   计算属性计算结果依赖于`data`中的值
*   同步操作，不支持异步

**侦听属性：**

*   使用场景：当需要在数据变化时执行异步或开销较大的操作时，可以用侦听属性
*   可配置参数：可以通过配置`immediate`和`deep`来控制立即执行和深度监听的行为
*   侦听属性侦听的是`data`中定义的

计算属性请参考[vue2从数据变化到视图变化：计算属性](https://link.juejin.cn?target=https%3A%2F%2Fjuejin.cn%2Fpost%2F7132100391776452615 "https://juejin.cn/post/7132100391776452615")  
侦听属性请参考[vue2从数据变化到视图变化：侦听器](https://juejin.cn/post/7132312215402577956 "https://juejin.cn/post/7132312215402577956")

二十三、`v-model`
-------------

```
// main.js
new Vue({
  el: "#app",
  data() {
    return {
      msg: ""
    };
  },
  template: `<div>
    <input v-model="msg" placeholder="edit me">
    <p>msg is: {{ msg }}</p>
  </div>`
});
复制代码
```

**普通input：**`input`中的`v-model`，最终通过`target.addEventListener`处理成在节点上监听`input`事件`function($event){msg=$event.target.value}}`的形式，当`input`值变化时`msg`也跟着改变。

```
// main.js
const inputBox = {
  template: `<input @input="$emit('input', $event.target.value)">`,
};

new Vue({
  el: "#app",
  template: `<div>
    <input-box v-model="msg"></input-box>
    <p>{{msg}}</p>
  </div>`,
  components: {
    inputBox
  },
  data() {
    return {
      msg: 'hello world!'
    };
  },
});
复制代码
```

**组件**：`v-model`在组件中则通过给点击事件绑定原生事件，当触发到`$emit`的时候，再进行回调函数`ƒunction input($$v) {msg=$$v}`的执行，进而达到子组件修改父组件中数据`msg`的目的。

二十四、`v-slot`
------------

`v-slot`产生的主要目的是，在组件的使用过程中可以让父组件有修改子组件内容的能力，就像在子组件里面放了个插槽，让父组件往插槽内塞入父组件中的楔子；并且，父组件在子组件中嵌入的楔子也可以访问子组件中的数据。`v-slot`的产生让组件的应用更加灵活。

### 1、具名插槽

```
let baseLayout = {
  template: `<div class="container">
    <header>
      <slot ></slot>
    </header>
    <main>
      <slot></slot>
    </main>
    <footer>
      <slot ></slot>
    </footer>
  </div>`,
  data() {
    return {
      url: ""
    };
  }
};

new Vue({
  el: "#app",
  template: `<base-layout>
    <template v-slot:header>
      <h1>title-txt</h1>
    </template>
    <p>paragraph-1-txt</p>
    <p>paragraph-2-txt</p>
    <template v-slot:footer>
      <p>foot-txt</p>
    </template>
  </base-layout>`,
  components: {
    baseLayout
  }
});
复制代码
```

引入的组件`baseLayout`中的`template`被添加了属性`v-slot:header`和`v-slot:footer`，子组件中定义了对应的插槽被添加了属性和，未被进行插槽标识的内容被插入到了匿名的`<slot></slot>`中。

### 2、作用域插槽

```
let currentUser = {
  template: `<span>
    <slot >{{childData.firstName}}</slot>
  </span>`,
  data() {
    return {
      childData: {
        firstName: "first",
        lastName: "last"
      }
    };
  }
};

new Vue({
  el: "#app",
  template: `<current-user>
    <template v-slot:user="slotProps">{{slotProps.userData.lastName}}</template>
  </current-user>`,
  components: {
    currentUser
  }
});
复制代码
```

当前例子中作用域插槽通过`v-bind:userData="childData"`的方式，将`childData`作为参数，父组件中通过`v-slot:user="slotProps"`的方式进行接收，为父组件使用子组件中的数据提供了可能。

`v-slot`的底层实现请参考[vue中的v-slot（源码分析）](https://link.juejin.cn?target=https%3A%2F%2Fjuejin.cn%2Fpost%2F7144188805254709256 "https://juejin.cn/post/7144188805254709256")

二十五、`Vue.filters`
-----------------

`filters`类似于管道流可以将上一个过滤函数的结果作为下一个过滤函数的第一个参数，又可以在其中传递参数让过滤器更灵活。

```
// main.js文件
import Vue from "vue";

Vue.filter("filterEmpty", function(val) {
  return val || "";
});

Vue.filter("filterA", function(val) {
  return val + "平时周末的";
});

Vue.filter("filterB", function(val, info, fn) {
  return val + info + fn;
});

new Vue({
  el: "#app",
  template: `<div>{{msg | filterEmpty | filterA | filterB('爱好是', transformHobby('chess'))}}</div>`,
  data() {
    return {
      msg: "张三"
    };
  },
  methods: {
    transformHobby(type) {
      const map = {
        bike: "骑行",
        chess: "象棋",
        game: "游戏",
        swimming: "游泳"
      };
      return map[type] || "未知";
    }
  }
});
复制代码
```

其中我们对`msg`通过`filterEmpty`、`filterA`和`filterB('爱好是', transformHobby('chess'))}`进行三层过滤。

`Vue.filters`的底层实现请查看[vue中的filters（源码分析）](https://juejin.cn/post/7146889304143298597 "https://juejin.cn/post/7146889304143298597")

二十六、`Vue.use`
-------------

*   作用：`Vue.use`被用来安装Vue.js插件，例如`vue-router`、`vuex`、`element-ui`。
*   `install`方法：如果插件是一个对象，必须提供 `install` 方法。如果插件是一个函数，它会被作为`install`方法。`install`方法调用时，会将`Vue`作为参数传入。
*   调用时机：该方法需要在调用 `new Vue()` 之前被调用。
*   特点：当 install 方法被同一个插件多次调用，插件将只会被安装一次。

二十七、`Vue.extend`和选项`extends`
----------------------------

### 1、`Vue.extend`

`Vue.extend`使用基础`Vue`构造器创建一个“子类”，参数是一个包含组件选项的对象，实例化的过程中可以修改其中的选项，为实现功能的继承提供了思路。

```
new Vue({
  el: "#app",
  template: `<div><div id="person1"></div><div id="person2"></div></div>`,
  mounted() {
    // 定义子类构造函数
    var Profile = Vue.extend({
      template: '<p @click="showInfo">{{name}} 喜欢 {{fruit}}</p>',
      data: function () {
        return {
          name: '张三',
          fruit: '苹果'
        }
      },
      methods: {
        showInfo() {
          console.log(`${this.name}喜欢${this.fruit}`)
        }
      }
    })
    // 实例化1，挂载到`#person1`上
    new Profile().$mount('#person1')
    // 实例化2，并修改其`data`选项，挂载到`#person2`上
    new Profile({
      data: function () {
        return {
          name: '李四',
          fruit: '香蕉'
        }
      },
    }).$mount('#person2')
  },
});
复制代码
```

在当前例子中，通过`Vue.extend`构建了子类构造函数`Profile`，可以通过`new Profile`的方式实例化无数个`vm`实例。我们定义初始的`template`、`data`和`methods`供`vm`进行使用，如果有变化，在实例的过程中传入新的选项参数即可，比如例子中实例化第二个`vm`的时候就对`data`进行了调整。

### 2、选项`extends`

`extends`允许声明扩展另一个组件 (可以是一个简单的选项对象或构造函数)，而无需使用 `Vue.extend`。这主要是为了便于扩展单文件组件，以实现组件继承的目的。

```
const common = {
  template: `<div>{{name}}</div>`,
  data() {
    return {
      name: '表单'
    }
  }
}

const create = {
  extends: common,
  data() {
    return {
      name: '新增表单'
    }
  }
}

const edit = {
  extends: common,
  data() {
    return {
      name: '编辑表单'
    }
  }
}

new Vue({
  el: "#app",
  template: `<div>
    <create></create>
    <edit></edit>
  </div>`,
  components: {
    create,
    edit,
  }
});
复制代码
```

当前极简demo中定义了公共的表单`common`，然后又在新增表单组件`create`和编辑表单组件`edit`中扩展了`common`。

二十八、`Vue.mixin`和选项`mixins`
--------------------------

全局混入和局部混入视情况而定，主要区别在全局混入是通过`Vue.mixin`的方式将选项混入到了`Vue.options`中，在所有获取子组件构建函数的时候都将其进行了合并，是一种影响全部组件的混入策略。

而局部混入是将选项通过配置`mixins`选项的方式合并到当前的子组件中，只有配置了`mixins`选项的组件才会受到混入影响，是一种局部的混入策略。

二十九、`Vue.directive`和`directives`
--------------------------------

### 1、使用场景

主要用于对于DOM的操作，比如：文本框聚焦，节点位置控制、防抖节流、权限管理、复制操作等功能

### 2、钩子函数

*   `bind`：只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置。
*   `inserted`：被绑定元素插入父节点时调用 (仅保证父节点存在，但不一定已被插入文档中)。
*   `update`：所在组件的 VNode 更新时调用，但是可能发生在其子 VNode 更新之前。指令的值可能发生了改变，也可能没有。但是你可以通过比较更新前后的值来忽略不必要的模板更新。
*   `componentUpdated`：指令所在组件的 VNode **及其子 VNode** 全部更新后调用。
*   `unbind`：只调用一次，指令与元素解绑时调用。

### 3、钩子函数参数

*   `el`：指令所绑定的元素，可以用来直接操作 DOM。
*   `binding`：一个对象，包含以下 property：
    *   `name`：指令名，不包括 `v-` 前缀。
    *   `value`：指令的绑定值，例如：`v-my-directive="1 + 1"` 中，绑定值为 `2`。
    *   `oldValue`：指令绑定的前一个值，仅在 `update` 和 `componentUpdated` 钩子中可用。无论值是否改变都可用。
    *   `expression`：字符串形式的指令表达式。例如 `v-my-directive="1 + 1"` 中，表达式为 `"1 + 1"`。
    *   `arg`：传给指令的参数，可选。例如 `v-my-directive:foo` 中，参数为 `"foo"`。
    *   `modifiers`：一个包含修饰符的对象。例如：`v-my-directive.foo.bar` 中，修饰符对象为 `{ foo: true, bar: true }`。
*   `vnode`：Vue 编译生成的虚拟节点。
*   `oldVnode`：上一个虚拟节点，仅在 `update` 和 `componentUpdated` 钩子中可用。

### 4、动态指令参数

指令的参数可以是动态的。例如，在 `v-mydirective:[argument]="value"` 中，`argument` 参数可以根据组件实例数据进行更新！这使得自定义指令可以在应用中被灵活使用。

三十、`vue`中的原生事件
--------------

`vue`中可以通过`@`或者`v-on`的方式绑定事件，也可为其添加修饰符。

```
new Vue({
  el: '#app',
  template: `<div @click='divClick'><a @clickt='aClick' href=''>点击</a></div>`,
  methods: {
    divClick() {
      console.log('divClick')
    },
    aClick() {
      console.log('aClick')
    },
  }
})
复制代码
```

以上例子如果点击`a`会触发其默认行为，如果`href`不为空还会进行跳转。除此之外，点击还会继续触发`div`上绑定的点击事件。

如果通过`@click.stop.prevent='aClick'`的方式为`a`标签的点击事件添加修饰符`stop`和`prevent`，那么就不会触发其`a`的默认行为，即使`href`不为空也不会进行跳转，同时，`div`上的点击事件也不会进行触发。

模板的渲染一般分为编译生成`render`函数、`render`函数执行生成`vNode`和`patch`进行渲染。下面按照这步骤进行简单分析。

### 1、`render`

通过编译生成的`render`函数：

```
with(this) {
    return _c('div', {
        on: {
            "click": divClick
        }
    }, [_c('a', {
        attrs: {
            "href": "http://www.baidu.com"
        },
        on: {
            "click": function ($event) {
                $event.stopPropagation();
                $event.preventDefault();
                return aClick($event)
            }
        }
    }, [_v("点击")])])
}
复制代码
```

其中`div`的`on`作为`div`事件描述。`a`标签的`attrs`作为属性描述，`on`作为事件描述，在描述中`.stop`被编译成了`$event.stopPropagation()`来阻止事件冒泡，`.prevent`被编译成了`$event.preventDefault()`用来阻止`a`标签的默认行为。

### 2、`vNode`

通过执行`Vue.prototype._render`将`render`函数转换成`vNode`。

### 3、`patch`

`patch`的过程中，当完成`$el`节点的渲染后会执行`invokeCreateHooks(vnode, insertedVnodeQueue)`逻辑，其中，针对`attrs`会将其设置为`$el`的真实属性，当前例子中会为`a`标签设置`herf`属性。针对`on`会通过`target.addEventListener`的方式将其处理过的事件绑定到`$el`上，当前例子中会分别对`div`和`a`中的`click`进行处理，再通过`addEventListener`的方式进行绑定。

### 小结

`vue`中的事件，从编译生成`render`再通过`Vue.prototype._render`函数执行`render`到生成`vNode`，主要是通过`on`作为描述。在`patch`渲染阶段，将`on`描述的事件进行处理再通过`addEventListener`的方式绑定到`$el`上。

三十一、常用修饰符
---------

### 1、表单修饰符

#### （1）`.lazy`

在默认情况下，`v-model` 在每次 `input` 事件触发后将输入框的值与数据进行同步 ，可以添加 `lazy` 修饰符，从而转为在 `change` 事件之后进行同步:

```
<input v-model.lazy="msg">
复制代码
```

#### （2）`.number`

如果想自动将用户的输入值转为数值类型，可以给 `v-model` 添加 `number` 修饰符：

```
<input v-model.number="age" type="number">
复制代码
```

#### （3）`.trim`

如果要自动过滤用户输入的首尾空白字符，可以给 `v-model` 添加 `trim` 修饰符：

```
<input v-model.trim="msg">
复制代码
```

### 2、事件修饰符

#### （1）`.stop`

阻止单击事件继续传播。

```
<!--这里只会触发a-->
<div @click="divClick"><a v-on:click.stop="aClick">点击</a></div>
复制代码
```

#### （2）`.prevent`

阻止标签的默认行为。

```
<a href="http://www.baidu.com" v-on:click.prevent="aClick">点击</a>
复制代码
```

#### （3）`.capture`

事件先在有`.capture`修饰符的节点上触发，然后在其包裹的内部节点中触发。

```
<!--这里先执行divClick事件，然后再执行aClick事件-->
<div @click="divClick"><a v-on:click="aClick">点击</a></div>
复制代码
```

#### （4）`.self`

只当在 event.target 是当前元素自身时触发处理函数，即事件不是从内部元素触发的。

```
<!--在a标签上点击时只会触发aClick事件，只有点击phrase的时候才会触发divClick事件-->
<div @click.self="divClick">phrase<a v-on:click="aClick">点击</a></div>
复制代码
```

#### （5）`.once`

不像其它只能对原生的 DOM 事件起作用的修饰符，`.once` 修饰符还能被用到自定义的组件事件上，表示当前事件只触发一次。

```
<a v-on:click.once="aClick">点击</a>
复制代码
```

#### （6）`.passive`

`.passive` 修饰符尤其能够提升移动端的性能

```
<!-- 滚动事件的默认行为 (即滚动行为) 将会立即触发 -->  
<!-- 而不会等待 `onScroll` 完成 -->  
<!-- 这其中包含 `event.preventDefault()` 的情况 -->  
<div v-on:scroll.passive="onScroll">...</div>
复制代码
```

### 3、其他修饰符

除了表单和事件的修饰符，`Vue`还提供了很多其他修饰符，在使用的时候可以查阅文档。

### 小结

> `Vue`中提供了很多好用的功能和`api`，那么修饰符的出现就为功能和`api`提供了更为丰富的扩展属性和更大的灵活度。

三十二、`vue-router`
----------------

`vue`路由是单页面中视图切换的方案，有三种`mode`:

*   hash，#后的仅仅作为参数，不属于url部分
*   history，路径作为请求url请求资源链接，如果找不到会出现404错误
*   abstract，服务端渲染场景 hash场景下，会出现`url`链接，再修改其view-router中对应的值。

了解`vue-router`的底层实现请参考[vue2视图切换：vue-router](https://link.juejin.cn?target=https%3A%2F%2Fjuejin.cn%2Fpost%2F7141965586099077157 "https://juejin.cn/post/7141965586099077157")

三十三、`vuex`
----------

`vuex`是状态管理仓库，一般使用的场景为：多个视图依赖于同一状态，来自不同视图的行为需要变更同一状态。其管理的状态是响应式的，修改也只能显式提交`mutation`的方式修改。`vuex`有`state`、`getter`、`mutation`、`action`和`module`五个核心，并且通过`module`实现了`vuex`树的管理。

了解`vuex`的底层实现请参考[vue2状态管理：vuex](https://juejin.cn/post/7141965586099077157 "https://juejin.cn/post/7141965586099077157")

三十四、`eventBus`
--------------

**使用场景**：兄弟组件传参

```
const eventBus = new Vue();

const A = {
  template: `<div @click="send">component-a</div>`,
  methods: {
    send() {
      eventBus.$emit('sendData', 'data from A')
    }
  },
}

const B = {
  template: `<div>component-b</div>`,
  created() {
    eventBus.$on('sendData', (args) => {
      console.log(args)
    })
  },
}

new Vue({
  el: '#app',
  components: {
    A,
    B,
  },
  template: `<div><A></A><B></B></div>`,
})
复制代码
```

在当前例子中，`A`组件和`B`组件称为兄弟组件，`A`组件通过事件总线`eventBus`中的`$emit`分发事件，`B`组件则通过`$on`来监听事件。

**实现原理：**`eventsMixin`

```
export function eventsMixin (Vue: Class<Component>) {
  const hookRE = /^hook:/
  Vue.prototype.$on = function (event: string | Array<string>, fn: Function): Component {
    const vm: Component = this
    if (Array.isArray(event)) {
      for (let i = 0, l = event.length; i < l; i++) {
        vm.$on(event[i], fn)
      }
    } else {
      (vm._events[event] || (vm._events[event] = [])).push(fn)
      // optimize hook:event cost by using a boolean flag marked at registration
      // instead of a hash lookup
      if (hookRE.test(event)) {
        vm._hasHookEvent = true
      }
    }
    return vm
  }

  Vue.prototype.$once = function (event: string, fn: Function): Component {
    const vm: Component = this
    function on () {
      vm.$off(event, on)
      fn.apply(vm, arguments)
    }
    on.fn = fn
    vm.$on(event, on)
    return vm
  }

  Vue.prototype.$off = function (event?: string | Array<string>, fn?: Function): Component {
    const vm: Component = this
    // all
    if (!arguments.length) {
      vm._events = Object.create(null)
      return vm
    }
    // array of events
    if (Array.isArray(event)) {
      for (let i = 0, l = event.length; i < l; i++) {
        vm.$off(event[i], fn)
      }
      return vm
    }
    // specific event
    const cbs = vm._events[event]
    if (!cbs) {
      return vm
    }
    if (!fn) {
      vm._events[event] = null
      return vm
    }
    // specific handler
    let cb
    let i = cbs.length
    while (i--) {
      cb = cbs[i]
      if (cb === fn || cb.fn === fn) {
        cbs.splice(i, 1)
        break
      }
    }
    return vm
  }

  Vue.prototype.$emit = function (event: string): Component {
    const vm: Component = this
    if (process.env.NODE_ENV !== 'production') {
      const lowerCaseEvent = event.toLowerCase()
      if (lowerCaseEvent !== event && vm._events[lowerCaseEvent]) {
        tip(
          `Event "${lowerCaseEvent}" is emitted in component ` +
          `${formatComponentName(vm)} but the handler is registered for "${event}". ` +
          `Note that HTML attributes are case-insensitive and you cannot use ` +
          `v-on to listen to camelCase events when using in-DOM templates. ` +
          `You should probably use "${hyphenate(event)}" instead of "${event}".`
        )
      }
    }
    let cbs = vm._events[event]
    if (cbs) {
      cbs = cbs.length > 1 ? toArray(cbs) : cbs
      const args = toArray(arguments, 1)
      const info = `event handler for "${event}"`
      for (let i = 0, l = cbs.length; i < l; i++) {
        invokeWithErrorHandling(cbs[i], vm, args, vm, info)
      }
    }
    return vm
  }
}
复制代码
```

在`Vue`构造函数定义完执行的`eventsMixin`函数中，在`Vue.prototype`上分别定义了`$on`、`$emit`、`$off`和`$once`的方法易实现对事件的绑定、分发、取消和只执行一次的方法。`eventBus`就是利用了当`new Vue`实例化后实例上的`$on`、`$emit`、`$off`和`$once`进行数据传递。

三十五、`ref`
---------

**使用场景：** 父组件获取子组件数据或者执行子组件方法

```
const A = {
  template: `<div>{{childData.age}}</div>`,
  data() {
    return {
      childData: {
        name: 'qb',
        age: 30
      },
    }
  },
  methods: {
    increaseAge() {
      this.childData.age++;
    }
  }
}

new Vue({
  el: '#app',
  components: {
    A,
  },
  template: `<A ref='childRef' @click.native='changeChildData'></A>`,
  methods: {
    changeChildData() {
      // 执行子组件的方法
      this.$refs.childRef.increaseAge()
      // 获取子组件的数据
      console.log(this.$refs.childRef.childData);
    },
  }
})
复制代码
```

在当前例子中，通过`ref='childRef'`的方式在当前组件中定义一个`ref`，可以通过`this.$refs.childRef`的方式获取到子组件`A`。可以通过`this.$refs.childRef.increaseAge()`的方式执行子组件中`age`增加的方法，也可以通过`this.$refs.childRef.childData`的方式获取到子组件中的数据。

三十六、`props`
-----------

**使用场景：** 父子传参

```
const A = {
  template: `<div @click='emitData'>{{childData}}</div>`,
  props: ['childData'],
  methods: {
    emitData() {
      this.$emit('emitChildData', 'data from child')
    }
  },
}

new Vue({
  el: '#app',
  components: {
    A
  },
  template: `<A :childData='parentData' @emitChildData='getChildData'></A>`,
  data() {
    return {
      parentData: 'data from parent'
    }
  },
  methods: {
    getChildData(v) {
      console.log(v);
    }
  }
})
复制代码
```

从当前例子中可以看出，数据父传子是通过`:childData='parentData'`的方式，数据子传父是通过`this.$emit('emitChildData', 'data from child')`的方式，然后，父组件通过`@emitChildData='getChildData'`的方式进行获取。

### 1、父组件`render`函数

`new Vue`中传入的模板`template`经过遍历生成的`render`函数如下：

```
with(this) {
    return _c('A', {
        attrs: {
            "childData": parentData
        },
        on: {
            "emitChildData": getChildData
        }
    })
}
复制代码
```

其中`data`部分有`attrs`和`on`来描述属性和方法。

在通过`createComponent`创建组件`vnode`的过程中，会通过`const propsData = extractPropsFromVNodeData(data, Ctor, tag)`的方式获取`props`，通过`const listeners = data.on`的方式获取`listeners`，最后将其作为参数通过`new VNode(options)`的方式实例化组件`vnode`。

### 2、子组件渲染

在通过`const child = vnode.componentInstance = createComponentInstanceForVnode( vnode, activeInstance )`创建组件实例的过程中，会执行到组件继承自`Vue`的`._init`方法，通过`initEvents`将事件处理后存储到`vm._events`中，通过`initProps`将`childData`赋值到子组件`A`的`vm`实例上，并进行响应式处理，让其可以通过`vm.childData`的方式访问，并且数据发生变化时视图也可以发生改变。

组件模板编译后对应的`render`函数是:

```
with(this) {
    return _c('div', {
        on: {
            "click": emitData
        }
    }, [_v(_s(childData))])
}
复制代码
```

在`createElm`完成节点的创建后，在`invokeCreateHooks(vnode, insertedVnodeQueue)`阶段，给`DOM`原生节点节点绑定`emitData`。

### 3、`this.$emit`

在点击执行`this.$emit`时，会通过`var cbs = vm._events[event]`取出`_events`中的事件进行执行。

至此，父组件中的传递的数据就在子组件中可以通过`this.xxx`的方式获得，也可以通过`this.$emit`的方式将子组件中的数据传递给父组件。

`prop`数据发生改变引起视图变化的底层逻辑请参考[vue2从数据变化到视图变化：props引起视图变化详解](https://link.juejin.cn?target=https%3A%2F%2Fjuejin.cn%2Fpost%2F7132758292681457700 "https://juejin.cn/post/7132758292681457700")

三十七、`$attrs`和`$listeners`
-------------------------

**使用场景：** 父子组件非`props`属性和非`native`方法传递

```
// main.js文件
import Vue from "vue";

const B = {
  template: `<div @click="emitData">{{ formParentData }}</div>`,
  data() {
    return {
      formParentData: ''
    }
  },
  inheritAttrs: false,

  created() {
    this.formParentData = this.$attrs;
    console.log(this.$attrs, '--------------a-component-$attrs')
    console.log(this.$listeners, '--------------b-component-$listeners')
  },
  methods: {
    emitData() {
      this.$emit('onFun', 'form B component')
    }
  },
}

const A = {
  template: `<B v-bind='$attrs' v-on='$listeners'></B>`,
  components: {
    B,
  },
  props: ['propData'],
  inheritAttrs: false,
  created() {
    console.log(this.$attrs, '--------------b-component-$attrs')
    console.log(this.$listeners, '--------------b-component-$listeners')
  }
}

new Vue({
  el: '#app',
  components: {
    A,
  },
  template: `<A :attrData='parentData' :propData='parentData' @click.native="nativeFun" @onFun="onFun"></A>`,
  data() {
    return {
      parentData: 'msg'
    }
  },
  methods: {
    nativeFun() {
      console.log('方法A');
    },
    onFun(v) {
      console.log('方法B', v);
    },
  }
})
复制代码
```

当前例子中，`new Vue`的`template`模板中有`attrData`、`propData`、`click.native`和`onFun`在进行传递。实际运行后，在`A`组件中`this.$attrs`为`{attrData: 'msg'}`，`this.$listeners`为`{onFun:f(...)}`。在`A`组件中通过`v-bind='$attrs'`和`v-on='$listeners'`的方式继续进行属性和方法的传递，在`B`组件中就可以获取到`A`组件中传入的`$attrs`和`$listeners`。

当前例子中完成了非`props`属性和非`native`方法的传递，并且通过`v-bind='$attrs'`和`v-on='$listeners'`的方式实现了属性和方法的跨层级传递。

同时通过`this.$emit`的方法触发了根节点中`onFun`事件。

关于例子中的`inheritAttrs: false`，默认情况下父作用域的不被认作`props`的`attribute`绑定将会“回退”且作为普通的`HTML`属性应用在子组件的根元素上。当撰写包裹一个目标元素或另一个组件的组件时，这可能不会总是符合预期行为。通过设置`inheritAttrs`到`false`，这些默认行为将会被去掉。

三十八、`$parent`和`$children`
-------------------------

**使用场景：** 利用父子关系进行数据的获取或者方法的调用

```
const A = {
  template: `<div @click="changeParentData">{{childRandom}}</div>`,
  data() {
    return {
      childRandom: Math.random()
    }
  },
  mounted() {
    console.log(this.$parent.parentCount, '--child-created--'); // 获取父组件中的parentCount
  },
  methods: {
    changeParentData() {
      console.log(this.$parent); // 打印当前实例的$parent
      this.$parent.changeParentData(); // 调用当前父级中的方法`changeParentData`
    },
    changeChildData() {
      this.childRandom = Math.random();
    }
  }
}
const B = {
  template: `<div>b-component</div>`,
}

new Vue({
  el: '#app',
  components: {
    A,
    B,
  },
  template: `<div><A></A><B></B><p>{{parentCount}}</p><button  @click="changeChildrenData">修改子组件数据</button></div>`,
  data() {
    return {
      parentCount: 1
    }
  },
  mounted() {
    console.log(this.$children[0].childRandom, '--parent-created--'); // 获取第一个子组件中的childRandom
  },
  methods: {
    changeParentData() {
      this.parentCount++;
    },
    changeChildrenData() {
      console.log(this.$children); // 此时有两个子组件
      this.$children[0].changeChildData(); // 调起第一个子组件中的'changeChildData'方法
    }
  }
})
复制代码
```

在当前例子中，父组件可以通过`this.$children`获取所有的子组件，这里有`A`组件和`B`组件，可以通过`this.$children[0].childRandom`的方式获取子组件`A`中的数据，也可以通过`this.$children[0].changeChildData()`的方式调起子组件`A`中的方法。

子组件可以通过`this.$parent`的方式获取父组件，可以通过`this.$parent.parentCount`获取父组件中的数据，也可以通过`this.$parent.changeParentData()`的方式修改父组件中的数据。

`Vue`中`$parent`和`$children`父子关系的底层构建请参考[杂谈：parent/children的底层逻辑](https://juejin.cn/post/7162097166725414942/ "https://juejin.cn/post/7162097166725414942/")

三十九、`inject`和`provide`
----------------------

使用场景：嵌套组件多层级传参

```
const B = {
  template: `<div>{{parentData1}}{{parentData2}}</div>`,
  inject: ['parentData1', 'parentData2'],
}

const A = {
  template: `<B></B>`,
  components: {
    B,
  },
}

new Vue({
  el: '#app',
  components: {
    A,
  },
  template: `<A></A>`,
  provide: {
    parentData1: {
      name: 'name-2',
      age: 30
    },
    parentData2: {
      name: 'name-2',
      age: 29
    },
  }
})
复制代码
```

例子中在`new Vue`的时候通过`provide`提供了两个数据来源`parentData1`和`parentData2`，然后跨了一个`A`组件在`B`组件中通过`inject`注入了这两个数据。

### 1、`initProvide`

在执行组件内部的`this._init`初始化方法时，会执行到`initProvide`逻辑：

```
export function initProvide (vm: Component) {
  const provide = vm.$options.provide
  if (provide) {
    vm._provided = typeof provide === 'function'
      ? provide.call(vm)
      : provide
  }
}
复制代码
```

如果在当前`vm.$options`中存在`provide`，会将其执行结果赋值给`vm._provided`。

### 2、`initInjections`

```
function initInjections (vm: Component) {
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
function resolveInject (inject: any, vm: Component): ?Object {
  if (inject) {
    // inject is :any because flow is not smart enough to figure out cached
    const result = Object.create(null)
    const keys = hasSymbol
      ? Reflect.ownKeys(inject)
      : Object.keys(inject)

    for (let i = 0; i < keys.length; i++) {
      const key = keys[i]
      // #6574 in case the inject object is observed...
      if (key === '__ob__') continue
      const provideKey = inject[key].from
      let source = vm
      while (source) {
        if (source._provided && hasOwn(source._provided, provideKey)) {
          result[key] = source._provided[provideKey]
          break
        }
        source = source.$parent
      }
      if (!source) {
        if ('default' in inject[key]) {
          const provideDefault = inject[key].default
          result[key] = typeof provideDefault === 'function'
            ? provideDefault.call(vm)
            : provideDefault
        } else if (process.env.NODE_ENV !== 'production') {
          warn(`Injection "${key}" not found`, vm)
        }
      }
    }
    return result
  }
}
复制代码
```

如果当前组件中有选项`inject`，会以`while`循环的方式不断在`source = source.$parent`中寻找`_provided`，然后获取到祖先组件中提供的数据源，这是实现祖先组件向所有子孙后代注入依赖的核心。

四十、`Vue`项目能做的性能优化
-----------------

### 1、`v-if`和`v-show`

*   频繁切换时使用`v-show`，利用其缓存特性
*   首屏渲染时使用`v-if`，如果为`false`则不进行渲染

### 2、`v-for`的`key`

*   列表变化时，循环时使用唯一不变的`key`，借助其本地复用策略
*   列表只进行一次渲染时，`key`可以采用循环的`index`

### 3、侦听器和计算属性

*   侦听器`watch`用于数据变化时引起其他行为
*   多使用`compouter`计算属性顾名思义就是新计算而来的属性，如果依赖的数据未发生变化，不会触发重新计算

### 4、合理使用生命周期

*   在`destroyed`阶段进行绑定事件或者定时器的销毁
*   使用动态组件的时候通过`keep-alive`包裹进行缓存处理，相关的操作可以在`actived`阶段激活

### 5、数据响应式处理

*   不需要响应式处理的数据可以通过`Object.freeze`处理，或者直接通过`this.xxx = xxx`的方式进行定义
*   需要响应式处理的属性可以通过`this.$set`的方式处理，而不是`JSON.parse(JSON.stringify(XXX))`的方式

### 6、路由加载方式

*   页面组件可以采用异步加载的方式

### 7、插件引入

*   第三方插件可以采用按需加载的方式，比如`element-ui`。

### 8、减少代码量

*   采用`mixin`的方式抽离公共方法
*   抽离公共组件
*   定义公共方法至公共`js`中
*   抽离公共`css`

### 9、编译方式

*   如果线上需要`template`的编译，可以采用完成版`vue.esm.js`
*   如果线上无需`template`的编译，可采用运行时版本`vue.runtime.esm.js`，相比完整版体积要小大约`30%`

### 10、渲染方式

*   服务端渲染，如果是需要`SEO`的网站可以采用服务端渲染的方式
*   前端渲染，一些企业内部使用的后端管理系统可以采用前端渲染的方式

### 11、字体图标的使用

*   有些图片图标尽可能使用字体图标

四十一、`Vue`项目白屏问题
---------------

*   1、开启`gzip`压缩减小文件体积。
*   2、`webpack`设置`productionSourceMap:false`，不在线上环境打包`.map`文件。
*   3、路由懒加载
*   4、异步组件的使用
*   5、静态资源使用`cdn`链接引入
*   6、采用`ssr`服务端渲染方案
*   7、骨架屏或者`loading`效果填充空白间隙
*   8、首次不渲染的隐藏采用`v-if`
*   9、注重代码规范：抽取公共组件，公共js，公共css样式，减小代码体积。删除无用代码，减少非必要注释。防止写出死循环等等
*   10、删除辅助开发的`console.log`
*   11、非`Vue`角度思考：非重要文件采用异步加载方式、css样式采用媒体查询、采用域名分片技术、http1升级成http2、如果是SSR项目考虑服务端渲染有没有可优化的点、请求头是否带了多余信息等思路

内容有些多，大体可以归类为从服务端拿到资源的速度、资源的体积和渲染是否阻塞的角度去作答。

四十二、从`0`到`1`构建一个`Vue`项目需要注意什么
-----------------------------

*   架子：选用合适的初始化脚手架(`vue-cli2.0`或者`vue-cli3.0`)
*   请求：数据`axios`请求的配置
*   登录：登录注册系统
*   路由：路由管理页面
*   数据：`vuex`全局数据管理
*   权限：权限管理系统
*   埋点：埋点系统
*   插件：第三方插件的选取以及引入方式
*   错误：错误页面
*   入口：前端资源直接当静态资源，或者服务端模板拉取
*   `SEO`：如果考虑`SEO`建议采用`SSR`方案
*   组件：基础组件/业务组件
*   样式：样式预处理起，公共样式抽取
*   方法：公共方法抽离

四十三、`SSR`
---------

### 1、什么是服务端渲染（SSR）？

Vue.js 是构建客户端应用程序的框架。默认情况下，可以在浏览器中输出 Vue 组件，进行生成 DOM 和操作 DOM。然而，也可以将同一个组件渲染为服务器端的 HTML 字符串，将它们直接发送到浏览器，最后将这些静态标记"激活"为客户端上完全可交互的应用程序。

### 2、为什么使用服务端渲染（SSR）？

与传统 SPA (单页应用程序 (Single-Page Application)) 相比，服务器端渲染 (SSR) 的优势主要在于：

*   更好的 SEO，由于搜索引擎爬虫抓取工具可以直接查看完全渲染的页面。
*   更快的内容到达时间 (time-to-content)，特别是对于缓慢的网络情况或运行缓慢的设备。

### 3、使用服务器端渲染 (SSR) 时需要考虑的问题？

使用服务器端渲染 (SSR) 时还需要有一些权衡之处

*   开发条件所限。浏览器特定的代码，只能在某些生命周期钩子函数 (lifecycle hook) 中使用；一些外部扩展库 (external library) 可能需要特殊处理，才能在服务器渲染应用程序中运行。
*   涉及构建设置和部署的更多要求。与可以部署在任何静态文件服务器上的完全静态单页面应用程序 (SPA) 不同，服务器渲染应用程序，需要处于 Node.js server 运行环境。
*   更多的服务器端负载。在 Node.js 中渲染完整的应用程序，显然会比仅仅提供静态文件的 server 更加大量占用 CPU 资源 (CPU-intensive - CPU 密集)，因此如果你预料在高流量环境 (high traffic) 下使用，请准备相应的服务器负载，并明智地采用缓存策略。

四十四、`scoped`
------------

在`Vue`项目开发的项目中如果样式中未使用`scoped`，组件间的样式会出现覆盖的问题。

**反例：**

```
// app.vue文件
<template>
  <div>
    <h3 class="title">app-txt</h3>
    <child></child>
  </div>
</template>

<script>
import child from "@/components/child";
export default {
  components: { child },
};
</script>

<style>
.title {
  color: red;
}
</style>
复制代码
```

```
// child.vue文件
<template>
  <h3 class="title">child-txt</h3>
</template>

<style>
  .title {
    color: green;
  }
</style>
复制代码
```

父组件和子组件的样式颜色都为`green`，子组件中的样式覆盖了父组件的样式。

**正例：**

```
<template>
  <h3 class="title">child-txt</h3>
</template>

<style scoped>
  .title {
    color: green;
  }
</style>
复制代码
```

此时，父组件中颜色为`red`，子组件中颜色为`green`。

**主要原因：**

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e4b3818e9f4e40a9acc00e957c114b71~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8343809daa3f49a98914c238b13e1ae7~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?) 例子中的DOM节点和CSS层叠样式中都被添加了`data-v-xxx`来表示唯一，所以`scoped`是给当前组件的节点和样式唯一标识为`data-v-xxx`，避免了样式覆盖。
