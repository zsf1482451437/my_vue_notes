# 1 组件中的data

**为什么组件中的data是函数？**

- 当data是函数时，每次使用组件都会返回一个**新的data对象**，使用**独立的地址空间**
- 如果不是函数，那复用组件时 将共用数据源，不符合组件化思想

# 2 props

用于父组件向子组件传数据

**本质就是给子组件添加自定义属性**

## 2.1 对象形式

父组件

```html
<son :msg="msg"></son>
```

```js
data() {
    return {
        msg: ['a', 'b', 'c']
    }
}
```

子组件son.vue

```html
<span v-for="item in msg" :key="item">{{ item }}</span>
```

```js
props: {
    msg: {
        type: Array,
            default() {
                return []
            }
    }
}
```

这样，子组件就可以使用父组件传过来的数据啦

# 3 emits（vue3）

**vue3新增**，用来定义一个组件**可以向其父组件发射的事件**。用法与props类似

**数组 | 对象**

## 2.1 vue2的做法

1. 在**子组件**中，通过**$emit()**来发射事件
2. 在**父组件**中，通过**v-on**来监听子组件事件

子组件`son.vue`

```html
<buttom @click="increment">+1</buttom>
```

```js
emits: ['add'],
methods: {
    increment() {
        this.$emit('add')
    }
}
```

父组件

```html
<h2>
    {{ counter }}
</h2>
<son @add="addOne"></son>
```

```js
data() {
    return {
        counter: 0
    }
},
methods: {
    addOne() {
        this.counter ++
    }
}
```



## 2.2 vue3的做法

### 数组形式

子组件`son.vue`

```html
<buttom @click="increment">+1</buttom>
```

```js
emits: ['add'],
methods: {
    increment() {
        this.$emit('add')
    }
}
```

父组件

```html
<h2>
    {{ counter }}
</h2>
<son @add="addOne"></son>
```

```js
data() {
    return {
        counter: 0
    }
},
methods: {
    addOne() {
        this.counter ++
    }
}
```

### 对象形式

对象写法的目的是进行**参数验证**

子组件`son.vue`

```html
<buttom @click="increment">+1</buttom>
```

```js
emits: {
    add: null
},
methods: {
    increment() {
        this.$emit('add')
    }
}
```

父组件

```html
<h2>
    {{ counter }}
</h2>
<son @add="addOne"></son>
```

```js
data() {
    return {
        counter: 0
    }
},
methods: {
    addOne() {
        this.counter ++
    }
}
```

null表示无参

当有参数时

```js
emits: {
    add: (num1, num2) => {
        return true
    }
}
```

对参数有限制时（比如大于第一个参数得大于10）

```js
emits: {
    add: (num1, num2) => {
        if(num1 > 10) {
            return true
        }
        return false
    }
}
```

虽然还是能传过去，但是会有警告；这样会清楚地知道**传递的参数是有问题的**

**提示**

**官网强烈建议使用 `emits` 记录每个组件所触发的所有事件。**

这尤为重要，因为**移除了 `.native` 修饰符**。任何未在 `emits` 中声明的**事件监听器**都会被算入组件的 `$attrs` 中，并将默认绑定到**组件的根节点**上。

# 4 provide和inject

用于**非父子**组件之间共享数据

如果通过props逐级往下传，将会非常麻烦。

无论**层级结构**有多深，父组件都可以作为其所有子组件的**依赖提供者**

父组件有一个**provide**选项来提供数据

子组件有一个**inject**选项来使用这些数据

**这个和props有什么区别呢？**

- 父组件不需要知道**哪些子组件使用**了provide的property
- 子组件不需要知道inject的property**来自哪里**

## 4.1 基本用法

父组件

```js
provide: {
    name: 'zsf',
    age: 18
}
```

子孙组件

```html
<h3>
    {{ name }} {{ age }}
</h3>
```

```js
inject: ['name', 'age']
```

## 4.2 使用data里面的数据

要想provide使用data里面的数据，并且通过this拿到

```js
data() {
  return {
    names: ['zsf','aaa']
  }
},
provide() {
    return {
      length: this.name.length,
    }
}
```

如果不写成函数，那**this**指向的就不是**组件实例**

## 4.3 处理响应式

如果改变names的长度，你会发现provide里面的length没有更新。

那要想它能做到更新，需要用到vue的computed()

```js
import { computed } from 'vue'
provide() {
    return {
      length: computed(() => this.name.length),
    }
}
```

# 5 事件总线

## 5.1 vue2的事件总线

### 初始化

**第一种方式**

将一个空的vue对象挂载到Vue原型上，这样每个组件对象都可以使用~

```js
Vue.prototype.$EventBus = new Vue()
```

**第二种方式**

创建一个模块Bus.js，导出一个空的vue对象，需要就导入

```js
// Bus.js
import Vue from 'vue'
export const EventBus = new Vue();
```

实质上，它是一个不具备 DOM 的组件，它具有的仅仅只是组件的**实例方法**而已，因此它非常的**轻便**。

### 发送和接收事件

- `EventBus.$emit('emit事件名'，数据)`发送
- `EventBus.$on("emit事件名", callback(payload1,…))` 接收

举例导入Bus.js模块的方式通过事件总线传递信息

A.vue

```html
<p>{{msgB}}</p>
<button @click="sendMsgA()">-</button>
```

```js
import { EventBus } from "../Bus.js"

data(){
    return {
    msg: ''
    }
},
mounted() {
    EventBus.$on("bMsg", (msg) => {
        // a组件接受 b发送来的消息
        this.msg = msg;
    });
},
methods: {
    sendMsgA() {
        EventBus.$emit("aMsg", '来自A页面的消息'); // a 发送数据
    }
}
```

B.vue

```html
<p>{{msgA}}</p>
<button @click="sendMsgB()">-</button>
```

```js
import { EventBus } from "../event-bus.js"

data(){
    return {
    msg: ''
    }
},
mounted() {
    EventBus.$on("aMsg", (msg) => {
        // b组件接受 a发送来的消息
        this.msg = msg;
    });
},
methods: {
    sendMsgB() {
        EventBus.$emit("bMsg", '来自b页面的消息'); // b发送数据
    }
}
```

如果只想接收一次，可以使用`EventBus.$once('事件名', callback(payload1,…)`

### 优缺点

**优点**

- 解决了多层组件之间繁琐的事件传播。
- 使用原理十分简单，代码量少。

**缺点**

- vue是单页面应用，如果在某一个页面刷新了之后，与之相关的EventBus会被移除，这样可能出现一下意外bug
- 如果有反复操作的页面，EventBus在监听的时候就会**触发很多次**，也是一个非常大的隐患。通常会用到，在vue页面**销毁时**，同时**移除EventBus**事件监听。
- 由于是都使用一个Vue实例，所以容易出现**重复触发**的情景，两个页面都定义了**同一个事件名**，并且没有用$off销毁（常出现在路由切换时）。

## 5.2 vue3的事件总线

vue3从实例中**移除了$on、$off、$once**方法，如果想使用**全局事件总线**，要通过**第三方库**

官方推荐**mitt**或**tiny-emitter**

### 使用mitt（vue3）

**安装**

`npm instal mitt`

**封装一个工具eventBus.js**

```js
import mitt from 'mitt'
const emitter = mitt()
export default emitter
```

发送组件sent.vue

```html
<buttom @click="btnClick"></buttom>
```

```js
import emitter from './eventBus.js'
methods: {
    btnClick() {
        emitter.emit('zsf',参数)
    }
}
```

接收组件accept.vue

```js
import emitter from './eventBus.js'
created() {
    emitter.on('zsf', (参数) => {
        拿到参数
    })
}
```

写法与Vue2类似，不过是使用了第三方库

### 多个事件的发射与监听

发送组件sent.vue

```js
import emitter from './eventBus.js'
methods: {
    btnClick() {
        emitter.emit('zsf',参数)
        emitter.emit('aaa',参数)
    }
}
```

接收组件accept.vue

```js
import emitter from './eventBus.js'
created() {
    emitter.on('zsf', (参数) => {
        拿到参数
    })
    emitter.on('aaa', (参数) => {
        拿到参数
    })
}
```

# 6 $refs

这种方式只需要在**需要访问的子组件或元素上加个ref="xxx"**的属性

可以通过**this.$refs**访问到**子组件或元素**的信息

## 6.1 ref在元素上

父组件

```html
<h2 ref='h'>
    
</h2>
```

这样，父组件就可以通过**this.$refs.h**获取到**h2的元素对象**

## 6.2 ref在组件上

父组件

```html
<son ref="item"></son>
```

这样，父组件就可以通过**this.$refs.item**获取到**组件son的实例对象**啦，子组件的信息都可以拿到（data、methods等等）

# 7 插槽

## 7.1 动态插槽名

后面补充~

## 7.2 作用域插槽

先来看看什么是**渲染作用域**

- 父级模板里的所有内容都是**在父级作用域中编译**的
- 子级模板里的所有内容都是**在子级作用域中编译**的

比如有个**父组件包了一个子组件**，**子组件**有个**title**数据，想**直接**在父组件里面**显示title**，这是不可以的。

这就是**渲染作用域**

但是，有时候我们希望**插槽可以访问到子组件中的内容**

**常见应用：**

当一个组件用来渲染一个**数组元素**时，**又想使用插槽**，并且**希望插槽中没有显示每项内容**

父组件

```html
<son :names="names"></son>
```

```js
data() {
    return {
        names: ['zsf', 'abc', 'sss']
    }
}
```

展示组件son.vue

```html
<template v-for="item in names" :key="item">
    <span>{{ item }}</span>
</template>
```

```js
props: {
    names: {
        type: Array,
        default: () => []
    }
}
```

一般情况是这样的。但是，要是父元素不想使用span展示，想用其它元素展示（换句话说，父元素使用son组件时，可以决定使用什么元素展示）

展示组件son.vue

```html
<template v-for="item in names" :key="item">
    <slot>{{ item }}</slot>
</template>
```

**父组件这样写对吗？**

```html
<son :names="names">
    <button>{{ item }}</button>
</son>
```

**不对**。由于存在渲染作用域，**button访问不到slot内部的item**

这时你可能会问：**为什么不直接在父组件遍历并展示？**

上面有说到：我们希望通过**复用其它组件**展示，又想使用**插槽**。。。

这就用到**作用域插槽**了

### 用法

展示组件son.vue，在定义插槽时声明

```html
<template v-for="(item, index) in names" :key="item">
    <slot :item="item" :index="index">{{ item }}</slot>
</template>
```

父组件这样写

```html
<son :names="names">
    <template v-slot="slotPros">
        <button>{{ slotPros.item }}</button>
    </template>
</son>
```

这样，**slotPros**可以拿到slot定义的那些属性（item、index）

# 8 动态组件

## 8.1 基本用法

使用**component**这个内置组件的**is**属性

is的值可以是**局部注册**过的组件，或者**全局注册**过的。

标签栏切换案例

```html
<div id="dynamic-component-demo">
  <button
     v-for="tab in tabs"
     :key="tab"
     :class="{ active: currentTab === tab }"
     @click="currentTab = tab"
   >
    {{ tab }}
  </button>

  <component :is="currentTab"></component>
</div>
```

```js
components: {
    Home,
    Posts,
    Archive
},
data() {
    return {
      currentTab: 'Home',
      tabs: ['Home', 'Posts', 'Archive']
    }
  }
```

## 8.2 给动态子组件传值

直接在component组件**加上属性**即可，就是把要传的值当component的属性

```html
<component :is="currentTab" name="zsf" :age="18"></component>
```

这样，切换到的组件都会拿到name和age，通过**props**拿到；

当然，动态子组件也可以通过**emits**给父组件传**事件**

## 8.3 状态缓存

你有没有想过这样一个问题：**切换子组件时，要想再切回去，以前的状态会保留吗？**

不会。一旦切换，上一个子组件就会被**销毁**，状态没了；切换回去时，是**重新创建**。

每一次的切换来切换去都是**销毁-重建**的过程，这是**耗性能**的一件事

**能不能将组件的状态缓存起来呢？**

可以。使用内置组件**keep-alive**包裹起来

```html
<keep-alive>
    <component :is="currentTab" name="zsf" :age="18"></component>
</keep-alive>
```

**keep-alive的三个属性**

### include

string | RegExp | Array

只有**名称匹配**的组件才会被**缓存状态**

### exclude

string | RegExp | Array

**匹配名称**的组件**不会缓存状态**

### max

number | string

最多可以缓存组件数量，一旦到达这数字，缓存组件**最近没有被访问**的实例会被销毁

**提示**

由于include和exclude都是根据**名称匹配**，所以要给对应组件加上**name选项**

# 9 异步组件

某些组件在一开始用不上，打包他们时，可以进行**分包**，优化**首屏渲染**时间

## 9.1 defineAsyncComponent（vue3）

Vue3提供了一个api：**defineAsyncComponent**

接收两种类型参数：

- 工厂函数，该工厂函数需要返回一个promise对象
- 对象，可以对异步组件进行更多配置

利用webpack的特性在Vue中使用异步组件

**接收工厂函数写法**

```js
import { defineAsyncComponent } from 'vue'
const AsyncDetail = defineAsyncComponent(() => import('./AsyncDetail.vue'))
```

import()返回的就是promise，并且会在打包时进行分包操作

**接收对象的写法**

```js
import { defineAsyncComponent } from 'vue'
const AsyncDetail = defineAsyncComponent({
    loader: () => import('./AsyncDetail.vue'),
    ...
})
```

更多配置可以查看官网

## 9.2 和suspense一起使用

Suspense是一个内置的**全局组件**，该组件有**两个插槽**

- default 如果default可以显示，就显示default插槽的内容
- fallback 如果default无法显示，就显示fallback插槽的内容

```html
<suspense>
    <template #default>
        <async-home></async-home>
    </template>
    <template #fallback>
        <loading></loading>
    </template>
</suspense>
```

# 10 组件使用v-model

能不能封装一个组件，使用的时候可以使用**v-model**实现**双向数据绑定**呢？

## 10.1 input使用

input元素可以直接使用v-model

```html
<input v-model="message">
<h2>
    {{ message }}
</h2>
```

```js
data() {
    return {
        message: 'hhh'
    }
}
```

v-model的本质是

```html
<input :value="message" @input="message = $event.taget.value">
```

**v-bind**绑定input的**value**属性，然后**监听**input的**input事件**，当input事件触发时，就将输入框中的value**赋值**给message。这样，就是实现了双向数据绑定。

## 10.2 自定义组件上使用

**那我要是想在自定义组件上使用呢？**

父组件

```html
<Sf v-model="message"></Sf>
```

```js
data() {
    return {
        message: 'hhh'
    }
}
```

`<Sf v-model="message"></Sf>`也可以写成

```html
<Sf :modelValue="message" @update:model-value="message = $event"></Sf>
```

由于是自定义组件，通过**$event**就可以拿到（不是$event.target.value）

Sf.vue

```html
<input v-model="value">
<h2>
    {{ modelValue }}
</h2>
```

```js
props: {
    modelValue: String
},
emits: ['update:model-value'],
computed() {
    value: {
        get() {
            return this.modelValue
        },
        set(value) {
            this.emit('update:modelValue', value)
        }
    }
}
```

## 10.3 自定义v-model绑定多个

父组件

```html
<Sf v-model="message" v-model:title='title'></Sf>
<h3>
    {{ message }}-{{ title }}
</h3>
```

Sf.vue

```html
<input v-model="value">
<input v-model="zsf">
```

```js
props: {
    modelValue: String,
    title: String
},
emits: ['update:model-value', 'update:title'],
computed() {
    value: {
        get() {
            return this.modelValue
        },
        set(value) {
            this.emit('update:modelValue', value)
        }
    },
	zsf: {
		set(zsf) {
			this.emit('update:title', zsf)
		},
		get() {
			return this.title
		}
	}
}
```

# 11 render函数

vue推荐在**绝大多数**情况下使用**模板**来创建**html**，只有一些**特殊的场景**，才需要js的**完全编程能力**。

这时，可以使用**render函数**，它**比模板更接近编译器**

## 11.1 VNode

vue在生成真实的DOM之前，会**将节点转成VNode**，而VNode组合在一起形成一颗**树结构**，也就是虚拟DOM（VDOM）

## 11.2 template变真实DOM

**template的里的html是怎么变成真实DOM的呢？**

看这么一段代码

```html
<template>
    <div>哈哈哈</div>
</template>
```

经过compiler，将template转化成**render函数**

然后执行render函数，生成**VNode**

```js
const vnode = {
    tag: 'div',
    children: '哈哈哈'
}
```

VNode最终变成**真实DOM**

```html
<div>哈哈哈</div>
```

然后浏览器经过渲染真实DOM，显示**哈哈哈**

## 11.3 h函数

如果想充分利用js的编程能力，可以自己来编写**createdVNode**函数，生成**对应的VNode**

**怎么做呢？**

使用**h函数**

- h函数用于创建VNode
- 更准确应该叫**createdVNode()**,vue将它简化为**h()**

### 参数

**参数1**  html标签 | 组件, 'div'

**参数2**  属性, {}

**参数3**  子节点(内容), 'hello'

使用**render()**创建VNode就不需要template啦

```js
import { h } from 'vue'
render() {
    return h('h2', {class: 'title'}, 'hello world')
}
```

(vue2**把h()当参数**传给render()函数)

## 11.4 实现计数器

**怎么使用render函数实现计数器呢？**

```js
import { ref, h } from 'vue'

setup() {
    const counter = ref(0)
    return {
        counter
    }
},
render() {
    return h('div', {class: 'app'}, [
        h('h2', null, `当前计数：${this.counter}`),
        h('button', {
            onClick: () => this.counter++
        }, '+1')
        h('button', {
            onClick: () => this.counter--
        }, '-1')
    ])
}
```

**为什么render()里面可以使用this.counter获取counter？**

render()内部是有**绑定this**的，且**this指向当前组件实例**

**setup还可以替换掉render()这个选项**

所以还可以这样写

```js
import { ref, h } from 'vue'

setup() {
    const counter = ref(0)
    return () => {
        return h('div', {class: 'app'}, [
            h('h2', null, `当前计数：${counter.value}`),
            h('button', {
                onClick: () => counter.value++
            }, '+1')
            h('button', {
                onClick: () => counter.value--
            }, '-1')
        ])
    }
}
```

在setup内部，所以就可以省掉this啦

注意了，在setup里面是不会自动解包的，所以要使用**counter.value**才能拿到counter

## 11.5 jsx

如果希望在项目中使用jsx，那么需要**添加对jsx的支持**

通常使用**Babel**来进行转换

来看个案例

```html
<h2 class="title">hello</h2>
```

使用**render()**是这样写的

```js
import { h } from 'vue'
render() {
    return h('h2', {class: 'title'}, 'hello world')
}
```

使用jsx之后，render()是这样写的

```js
import { h } from 'vue'
render() {
    return <h2 class="title">hello</h2>
}
```

如果你的脚手架不支持jsx，**安装相关插件**并在**Babel.config.js**填写相关配置即可

`npm install @vue/babel-plugin-jsx -D`

Babel.config.js

```js
module.exports = {
    presets: [
        '@vue/cli-plugin-babel/preset'
    ],
    plugins: [
        '@vue/babel-plugin-jsx'
    ]
}
```

# 12 vue插件

通常向vue**全局添加一些功能**时，会采用**插件**的模式，它有**两种**编写方式：

- 对象，必须包含一个**install函数**，该函数会在**安装插件时执行**
- 函数，在**安装插件时自动执行**

## 12.1 插件的强大

完成的功能没有限制：

- 添加**全局方法或属性**，通过把方法或属性添加到**config.globalProperties**上实现；
- 添加**全局资源**：**指令/过滤器/过渡**等
- 通过**全局mixin**来添加一些**组件选项**
- 一个**库**，提供自己的**API**

## 12.2 对象形式

```js
export default {
    install(app) {
        app.config.globalProperties.$name = 'zsf'
    }
}
```

为了防止冲突，一般给添加的属性命名加上$

**怎么使用呢？**

main.js

```js
import myPlugin from './plugins'

import { createApp } from 'vue'

const app = createApp(根组件)
app.use(myPlugin)
```

**怎么获取到刚刚添加的name呢？**

vue2是这样获取的

```js
mounted() {
    console.log(this.$name)
}
```

换成其它生命周期或选项也是通过**this.$name**获取

**vue3怎么获取呢？**

由于setup中this没有指向当前组件实例，获取到刚刚添加的name有点麻烦,要借助一个vue得API

```js
import { getCurrentInstance } from 'vue'
setup() {
    const instance = getCurrentInstance()
    console.log(instance.appContext.config.globalProperties.$name)
}
```

那么长一段才能获取~

## 12.3 函数形式

```js
export default function(app){
    app.config.globalProperties.$name = 'zsf'
}
```

**main.js**

```js
import myPlugin from './plugins'

import { createApp } from 'vue'

const app = createApp(根组件)
app.use(myPlugin)
```

你会发现这种形式和对象形式差别不大，无非就是通过传入app，然后进行一系列操作~

# 13 nextTick

## 13.1 基本使用

有这么一个**需求**：

点击一个**按钮**，会修改在**h2**中显示的message；

message修改后，获取h2的**高度**；

实现有三种方式：

- 方式一，在点击按钮后立即获取h2的高度；
- 方式二，在updated生命周期中获取h2高度；
- 方式三，使用nextTick();

方式一是**错误**的。此时**DOM并没有更新**，这样获取的h2高度是不对的；

方式二确实**能获取**准确的h2高度，但其它节点更新，也会获取h2高度，这样做**不妥**；

方式三就可以，等下一次DOM更新完再获取h2高度。

## 13.2 原理

**nextTick是如何做到的呢？**

Vue 在更新 DOM 时是**异步**执行的；

每个数据，都会有对应的watch，当**数据更新**，就会执行watch()中的**回调函数**；

比如某个**数据连续更新了100次（同步代码），界面是不会刷新100次的**；

原因是Vue内部将**watch()**中的回调函数放入**微任务队列**中；

等**主线程**的同步代码都执行完，再去执行**watch()**中的回调函数；

这样的调度大大**提高了性能**；

`nextTick()` 内部是使用`Promise` 的，把DOM更新完要做的操作放到**微任务队列**队尾；

等DOM更新完（主线程同步代码），在执行nextTick()中的回调函数；

这样就能确保获取最新DOM的信息是**准确**的；

通俗一点来讲就是：

DOM的更新，获取DOM的信息，这**两个操作**需要排队，**更新在前，获取在后**。

