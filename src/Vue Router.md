# 1 web发展阶段

- 后端路由阶段；
- 前后端分离阶段；
- 单页面富应用（SPA）；

## 1.1 后端路由阶段

在早期的网站开发，整个html页面由**服务器来渲染**，服务器直接渲染好对应的html页面，返回给客户端进行展示；

一个**页面有对应的网址**，也就是**URL**，统一资源定位符；

URL会发送到服务器，服务器通过**正则表达式**对该URL进行**匹配**，最后**交给一个Controller进行处理**；

Controller最终进行各种处理，最终生成**html或者数据**，返回给客户端；

这就是**后端路由**，后端路由**有利于SEO**。

但是，后端路由**缺点**也非常明显：

- 整个页面的模块由**后端人员**来编写；
- 前端开发人员如果要开发页面，需要通过**PHP和Java**等语言来编写页面代码；
- html代码和数据的处理逻辑会混在一起，编写和维护的**体验都很糟糕**；

## 1.2 前后端分离阶段

随着**Ajax技术的**出现，有了**前后端分离**的开发模式；

**后端**只提供**API**来返回数据，**前端通过Ajax获取数据**，并且可以**通过js将数据渲染到页面**中；

这样前后端责任清晰，**后端**专注于**数据**上，**前端**专注于**交互和可视化**上；



## 1.3 单页面富应用

SPA（single page application）

- SPA单页面应用程序：整个网站只有一个页面，内容的变化通过Ajax局部更新实现、同时支持浏览器地址栏的后退操作
- SPA实现原理之一：**基于url地址的hash**（hash的变化会导致浏览器记录访问历史的变化，但是hash的变化不会触发新的url请求）
- 在实现SPA过程中，最核心的技术点就是**前端路由**

# 2 前端路由

**什么叫前端渲染？**

每次请求涉及到的静态资源都会从**静态服务器获取**，这些资源**包括HTML+CSS+JS**，然后在**前端对这些请求回来的资源进行渲染**；

前端路由其实就是由**前端负责**维护**路径和组件**之间的**映射关系**



## 2.1 改变路径而不刷新

### url的hash

也叫锚点（#），本质的改变**window.location的href属性**

可以通过直接赋值**location.hash**来改变href，但**页面不发生刷新**

**案例**

```html
<div id="app">
    <a href="#/home">home</a>
    <a href="#/about">about</a>
    <div class="content"></div>
</div>
```

```js
const contentEl = document.querySelector('.content')
window.addEventListener('hashchange', () => {
  switch (location.hash) {
    case '#/home':
      contentEl.innerHTML = 'Home'
      break
    case '#/about':
      contentEl.innerHTML = 'about'
      break
    default:
    contentEl.innerHTML = 'Default'
      break;
  }
})
```

监听hash值的改变，当点击home或about的链接，**location.hash**会发生改变，根据不同hash显示不同内容~

### html5的history

history是html5新增，有6种方式改变URL而不刷新页面

#### pushState(栈结构)

使用**新的路径**，**旧的路径**压入历史记录栈中，所以**可以回退**

`history.pushState({},'','home')`

#### replaceState

新路径**替换**旧的路径，所以**没有后退**

`replaceState({},'','home')`

#### popState

路径的**回退**，在**历史记录栈**中找路径

#### go、forward、back

`history.go(-1) = history.back()`

`history.go(1) = history.forward()`

# 3 Vue Router

vue router（官网 https://router.vuejs.org/zh）是vue.js 官方的**路由管理器**

vue router包含的功能：

支持**html5的history**模式或**hash**模式

- 支持嵌套路由
- 支持路由参数
- 支持编程式路由
- 支持命名路由

## 3.1 基本使用

### 方式一：引入相关的库文件

```html
<!-- 导入vue文件，为全局window对象挂载vue构造函数 -->
<script src="lib/vue_2.5.22.js"></script>

<!-- 导入vue-router文件，为全局window对象挂载vueRouter构造函数 -->
<script src="lib/vue-router_3.0.2.js"></script>
```

**或者**

### 方式二：npm安装

`npm install vue-router -s`

### 模块工程中使用它

1. **导入**路由对象，并且调用**Vue.use(VueRouter)**
2. 创建**router实例**，传入路由**映射配置**routes
3. 导出router实例
4. 在**Vue实例**中挂载**router实例**
5. 通过<router-link>和<router-view>使用

文件夹router下**index.js**

```js
import VueRouter from 'vue-router'

const routes = [
    {
        path: '/',
        components: ...
    }
]
const router = new VueRouter({
    routes
})
export default router
```

vueRouter**3.x**使用**vueRouter()**创建可以被vue应用程序使用的**路由实例**

而vueRouter**4.x**使用**createRouter()**创建**路由实例**

```js
import { createRouter } from 'vue-router'

const routes = [
    {
        path: '/',
        components: ...
    }
]
const router = new createRouter({
    routes
})
export default router
```

**main.js**

**vue2.x写法**

```js
import Vue from 'vue'
import App from './App.vue'
import router from './router'

Vue.use(router)
new Vue({
    router,
    render: h => h(App)
}).$mount('#app')
```

**vue3.x写法**

```js
import { creatApp } from 'vue'
import { router } from './router'
import App from './App.vue'

const app = creatApp(App)
app.use(router)
app.mount('#app')
```

**为什么url上有#？**

默认情况下，路径的改变使用的是url的hash

**怎么去掉#？**

路径的改变换成HTML5的history模式

创建router实例的时候再添加一个属性mode

**VueRouter3.x写法**

```js
const router = new VueRouter({
    routes,
    mode: 'history'
})
```

**VueRouter4.x写法**

```js
import { createRouter, createWebHistory } from 'vue-router'

const routes = [
    {
        path: '/',
        components: ...
    }
]
const router = new createRouter({
    routes,
    history: createWebHistory()
})
export default router
```

**路由和组件的对应关系**就搞定了；

但这还没完，还得告诉页面**哪个地方显示**不同路由对应的组件内容，只有这样，当你**切换路由**时，页面才知道**哪个地方**显示不同组件内容，这是就要用到**router-view**了；

## 3.2 router-view

是vue提供的**内置组件**，**路由占位符**，当路由切换时，会动态的显示不同内容

```html
<router-link to="/home">首页</router-link>
<router-link to="/about">关于</router-link>
<router-view></router-view>
```

当点击**首页**，router-view中的内容变成**/home**对应的组件；

当点击**关于**，router-view中的内容变成**/about**对应的组件；

### 增强

router-view也有**作用域插槽**，将**props.Componet**传给子组件；

搭配**transition**这个内置组件，可以实现路由**跳转时动画效果**；

```html
<router-view v-slot="props">
    <transition name="zsf">
        <component :is="props.Component"></component>
    </transition>
</router-view>
```

```css
.zsf-enter-from,
.zsf-leave-to {
    opacity: 0;
}

.zsf-enter-to,
.zsf-leave-from {
    opacity: 1;
}

.zsf-enter-active,
.zsf-leave-active {
    transition: opacity 2s ease;
}
```



### 与keep-alive搭配

路由跳转的时候，组件内部的状态是**没有被保存下来**的，**每次切换回来的时候都是重新渲染**(重复执行组件的那些生命周期函数)，**如果不希望重新渲染呢？**

```html
<keep-alive>
    <router-view></router-view>
</keep-alive>
```

只有组件保持了状态使用了**keep-alive**时，生命周期函数**activated()**和**deactivated()**才是有效的

## 3.3 router-link

router-link默认会渲染成a标签

**但要是想渲染成其它标签呢？**

比如button

### tag

添加一个tag="button"属性，会渲染成button元素

```vue
<router-link to="/home" tag="button">首页</router-link>
```

**Vue Router4.x**已经删除这个属性了

### replace

添加一个replace属性

```vue
<router-link to="/home" replace>首页</router-link>
```

设置replace属性的话，点击时，会调用**router.replace()**,新路径**替换**旧路径，且**没有后退**；

当然这个用的比较少，因为允许用户返回体验更好~

**如果点击到某个按钮时，希望它是红色（激活），怎么做？**

### active-class

添加一个**active-class="active"**属性

```vue
<router-link to="/home" active-class="active">首页</router-link>
```

```css
.active {
    color: red;
}
```

**运用场景**：

**导航栏菜单**或者**底部tabbar**需要**高亮**显示时会用到

### v-slot

**Vue Router4.x**已经删除了**tag**属性，要是渲染成其它**元素或组件**，直接写，内部用的是**插槽**的原理

**元素**

```html
<router-link to="/home">
    <button>首页</button>
</router-link>
```

要想给router-link包裹的**子组件或元素**传递某些数据，可以使用**作用域插槽**

```html
<router-link to="/home" v-slot="props">
    <button>首页</button>
</router-link>
```

props中属性很多，详情可查看官网~

## 3.4 路由重定向

用户在访问地址a的时候，强制用户跳转到地址c，从而展示特定的组件页面

通过路由规则的redirect属性，指定一个新的路由地址，可以很方便地设置路由的重定向**（路由的默认路径）**

```js
const routes = [
    { path:"/",redirect:"/user"},
    { path: "/user", component: User },
]
```

## 3.5 路由懒加载

打包构建应用时，如果所有所有的打包结果都放**app.[hash].js**这个文件，会变得非常大，影响**首屏加载时间**

所以我们需要对某些打包结果进行**分包**，放到别的文件~，这就需要使用**webpack**提供的**import()**

路由懒加载的作用：**将路由对应的组件打包成一个个的js文件，只有在这个路由被访问到时才加载对应组件**

简而言之，**用到时再加载**

### 方式

方式一：结合vue的异步组件和webpack的代码分割**（老、长）**

```js
const Home = resolve => { require.ensure(['../components/Home.vue'], () =>{ resolve(require('../components/Home.vue'))})};
```

方式二：AMD写法

```js
const About = resolve => require(['../components/About.vue'], resolve)
```

方式三：es6**（推荐）**

```js
const Home = () => import('../components/Home.vue')
```

### 使用

```js
const routes = [
    {
    	path: '/',
    	componenet: () => import('../components/Home.vue')
	}  
]
```

**componenet**属性可以是个**函数**，但这函数必须**返回一个Promise**，而**import()**返回值恰好是Promise；

不过打包之后，由于使用了**hash值命名**，不知道是哪个组件对应的打包结果；

如果想对**打包结果命名**，可以使用**魔法注释**（magic comment），给**import()**传入注释；

```js
const routes = [
    {
    	path: '/',
    	componenet: () => import(/* webpackChunkName: '名字' */'../components/Home.vue')
	}  
]
```

`/* webpackChunkName: '名字' */` 是固定格式，只有名字是自定义的，不过一般会在自定义名字的基础上，加上 `-chunk`，比如

`/* webpackChunkName: 'home-chunk' */` 个人习惯~

## 3.6 动态路由匹配

某些情况下，一个页面的path路径可能是不确定的，比如进入用户界面时，希望是如下路径：

- user/aaa或user/bbb
- user/用户id

**如果希望组件获取到那个用户id并展示，怎么做？**

文件夹router下的**index.js**

```js
routes: [
    // 动态路径参数，以冒号开头
    { 
        path: "/user/:username", 
        component: User 
    },
]
```

### vue2写法

**App.vue**

```html
<router-link :to="'/user/'+ userId">用户</router-link>
{{ userId }}
```

```js
created() {
    userId () {
        return this.$route.params.id
    }
}
```

**this.$route**获取到的处于**活跃状态**的路由信息；

上面例子中，处于活跃的路由是`{ path: "/user/:username", component: User }`；

### vue3写法

由于**setup()**中this获取不到当前组件实例，所以**this.$route不可行**;

**Vue Router4.x**提供了一个hook函数，**useRoute()**，它返回**当前组件**对应的**路由对象**;

**App.vue**

```html
<router-link :to="'/user/'+ username">用户</router-link>
{{ route.params.username }}
```

```js
import { useRoute } from 'vue-router'

setup() {
    const route = useRoute()
    return {
        route
    }
}
```

**注意**

是**$route**，不是**$router**!

### 多条件匹配

当然，不止支持一个条件匹配，也支持**多条件**；

```js
routes: [
    // 动态路径参数，以冒号开头
    { 
        path: "/user/:username/id/:id", 
        component: User 
    },
]
```



### NotFound

当某个路由**没有对应组件**时，页面会显示**空白**；

这对**用户体验非常不友好**，应该给出**提示**；

应该有个**NotFound信息提示**的组件；

```js
routes: [
    // 动态路径参数，以冒号开头
    { 
        path: '/:pathMatch(.*)', 
        component: () => import('./NotFound.vue') 
    },
]
```

NotFound页面也可以获取到**对应的路由信息**，通过**$route.params.pathMatch**；

如果路径匹配时在使用 `:pathMatch(.*)` 的基础上再加个*****，获取到的路由信息将**以/为分隔符**，放入一个数组中；

```js
routes: [
    // 动态路径参数，以冒号开头
    { 
        path: '/:pathMatch(.*)*', 
        component: () => import('./NotFound.vue') 
    },
]
```

**Vue Router3.x**使用通配符*

## 3.7 路由嵌套

在home页面中，我们希望通过/home/news和/home/messages访问一些内容。

**怎么做？**

### 使用

router文件夹下的**index.js**

```js
const routes = [
    {
    	path: '/home',
    	componenet: () => import('../components/Home.vue'),
    	children: [
    		{
    			path: 'news',
    			component: () => import('../components/HomeNews.vue')
			}
    	]
	}  
]
```

**Home.vue**

```html
<div>
    <h2>Home组件</h2>
    <router-link to="/home/news"></router-link>
    <router-view>给HomeNews组件的占位符</router-view>
</div>
```



## 3.8 编程式导航

### 页面导航方式

- **声明式导航**：通过**点击链接**实现导航的方式，叫做声明式导航

  例如：普通网页中的<a></a>链接或vue中的<router-link></router-link>

- **编程式导航**：通过**调用javascript形式的api**实现导航的方式，叫做编程式导航

  例如：普通网页中的location.href

### 编程式导航

使用**$router对象**

**vue2写法**

```html
<button @click="goRegister">跳转到注册页面</button>
```

```js
methods: {
    goRegister() {
        this.$router.push('/register');
    }
}
```

**为什么this.$router没有定义却能使用？**

因为vue-router给**所有组件**都添加了**$router**属性,所有组件可以通过**this.$router**拿到

**通过源码发现，所有vue组件都继承了vue的原型（prototype），当执行 `Vue.prototype.name = 'zsf'` 时，所有vue组件都有了name这个属性，方法同理，$router和$route就是这样给所有组件加上去的**

**router.push()**的参数规则

```js
// 字符串（路径名称）
router.push('/home')
// 对象
router.push( { path:'/home' } )
// 命名的路由
router.push({ name: '/user', params: { userid: 123 } })
// 带查询参数，变成/register?uname=lisi
router.push({ path: '/register', query: { uname: 'lisi' } })
```

**vue3写法**

```html
<button @click="goRegister">跳转到注册页面</button>
```

```js
import { useRouter } from 'vue-router'

setup() {
    const router = useRouter()
    const goRegister = () => {
        router.push('/register')
    }
    return {
        goRegister
    }
}
```



## 3.9 动态添加路由

### 一级

一般情况，路由对象的routes属性是**内容固定**的，路由规则已经写好；

但某些情况下，希望routes的**路由规则**是**动态**的，这时就需要**动态添加路由**了；

使用路由对象的**addRoute()**

```js
// 假设路由对象router已创建
const routes = [...]
const homeRoute = {
	path: '/home',
    component: () => import('./Home.vue')
}
router.addRoute(homeRoute)
```

### 二级

**二级路由呢？**

**addRoute()**如果有两个参数，第一个是一级路由

```js
// 假设路由对象router已创建
const routes = [...]
const homeRoute = {
	path: '/home',
    component: () => import('./Home.vue')
}
router.addRoute('app', homeRoute)
```

效果与下面同理，不过这是**动态添加**的

```js
const routes = [
    {
    	path: '/app',
        name: 'app'
    	componenet: () => import('./App.vue'),
    	children: [
    		{
    			path: '/home',
    			component: () => import('./Home.vue')
			}
    	]
	}  
]
```



## 3.10 导航守卫

通过**跳转或取消**的方式守卫某一次导航；

比如一个**登陆页面**，当你填写完信息，点击登陆；

导航守卫会**拦截**你这次跳转，判断你的信息是否正确；

信息正确，导航到主页；

信息错误，导航到登陆页；

### 前置路由守卫

**router.beforEach()**在**导航时**会触发回调

**Vue Router3.x**时，该回调函数传入三个参数：

- to，即将跳转的**route对象**
- from，当前**route对象**
- next，next()

```js
router.beforEach((to, from, next) => {
    
})
```

**Vue Router4.x**时,第三个参数**不推荐使用**了，因为会**执行next()多次**

**router.beforEach()返回值有四种类型**

- **false**，不进行导航；
- **undefined**，进行默认导航；
- **字符串**，跳转到对应路由；
- **对象**，类似router.push({...});

### **简单实现登陆逻辑**

- 登陆成功在**localStorage**设置**token**；
- 每次导航**非登录页**时，进行**导航守卫**；
- 如果**localStorage**的**token**有值，返回**undefined**，允许导航（默认）；
- 如果**localStorage**的**token**没有值，返回**字符串**（'/login'）,导航到登录页；

登陆成功时

```js
const token = window.localStorage.setItem('token', 'zsf')
```

登陆导航守卫

```js
router.beforEach((to, from) => {
    if(to.path !== '/login') {
        const token = window.localStorage.getItem('token')
        if(!token) {
            return '/login'
        }
    }
})
```



### 案例-改变网页标题

**在一个SPA应用中，如何改变网页的标题？**

网页标题是通过title标签来显示的，但是SPA只有一个固定的HTML，切换不同页面时，标题不会改变

但是可以通过javaScript来修改title的内容： `window.document.title = '新标题'`

**那在vue项目中，在哪里修改？什么时候修改比较合适呢？**

在生命周期函数created()中

```js
created () {
    document.title = '首页'
} 
```

**但是页面多了或者需求更改之后这个做法不好维护**

既然页面通过路由跳转，那能不能监听一下路由跳转的过程？当每次监听发生跳转的时候，改成对应的标题就可以了

```js
const routes = [
    {
    	path: '/',
        meta: {
    		title: '首页'
		}
    	componenets: Home,
    	childern: [
    		{
    			path: 'news',
    			components: HomeNews
			}
    	]
	}  
]
router.beforEach((to, from, next) => {
    document.title = to.matched[0].meta.title
    next()
})
```

- 一定要调用**next()**,不然页面不发生跳转
- 要给每个route对象加个**元数据meta**

当然，还有其它守卫，详情看官方文档；

## 3.11 historyAPIFallback

主要作用是解决SPA页面在路由跳转之后，进行页面刷新时，返回404的错误；

**默认值**是false，如果设置为true，那么刷新时，返回404错误时，会自动返回**index.html**的内容；

事实上devServer中实现**historyApiFallback**功能时通过**connect-history-api-fallback**库的；

**如何修改这个配置？**

- 修改**cli-service**源码
- 在**vue.config.js**中配置

这里只说vue.config.js中配置

```js
module.exports = {
    configureWebpack: {
        devServer: {
            historyAPIFallback: true
        }
    }
}
```

