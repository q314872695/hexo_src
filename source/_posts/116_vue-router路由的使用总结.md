---
title: vue-router路由的使用总结
date: 2021-01-19 09:49:46 
tags: vue
---


# 1. 路由的概念
路由的本质就是一种对应关系，比如说我们在url地址中输入我们要访问的url地址之后，浏览器要去请求这个url地址对应的资源。
那么url地址和真实的资源之间就有一种对应的关系，就是路由。

**路由分为前端路由和后端路由**
1. 后端路由是由服务器端进行实现，并完成资源的分发
2. 前端路由是依靠hash值(锚链接)的变化进行实现 

后端路由性能相对前端路由来说较低，所以，我们接下来主要学习的是前端路由
**前端路由的基本概念：** 根据不同的事件来显示不同的页面内容，即事件与事件处理函数之间的对应关系
前端路由主要做的事情就是监听事件并分发执行事件处理函数

# 2. 前端路由的初体验
前端路由是基于hash值的变化进行实现的（比如点击页面中的菜单或者按钮改变URL的hash值，根据hash值的变化来控制组件的切换）
核心实现依靠一个事件，即监听hash值变化的事件
```
window.onhashchange = function(){
    //location.hash可以获取到最新的hash值
    location.hash
}
```

前端路由实现tab栏切换：
```
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <meta http-equiv="X-UA-Compatible" content="ie=edge" />
        <title>Document</title>
        <!-- 导入 vue 文件 -->
        <script src="./lib/vue_2.5.22.js"></script>
    </head>
    <body>
        <!-- 被 vue 实例控制的 div 区域 -->
        <div id="app">
        <!-- 切换组件的超链接 -->
        <a href="#/zhuye">主页</a> 
        <a href="#/keji">科技</a> 
        <a href="#/caijing">财经</a>
        <a href="#/yule">娱乐</a>

        <!-- 根据 :is 属性指定的组件名称，把对应的组件渲染到 component 标签所在的位置 -->
        <!-- 可以把 component 标签当做是【组件的占位符】 -->
        <component :is="comName"></component>
        </div>

        <script>
        // #region 定义需要被切换的 4 个组件
        // 主页组件
        const zhuye = {
            template: '<h1>主页信息</h1>'
        }

        // 科技组件
        const keji = {
            template: '<h1>科技信息</h1>'
        }

        // 财经组件
        const caijing = {
            template: '<h1>财经信息</h1>'
        }

        // 娱乐组件
        const yule = {
            template: '<h1>娱乐信息</h1>'
        }
        // #endregion

        // #region vue 实例对象
        const vm = new Vue({
            el: '#app',
            data: {
            comName: 'zhuye'
            },
            // 注册私有组件
            components: {
            zhuye,
            keji,
            caijing,
            yule
            }
        })
        // #endregion

        // 监听 window 的 onhashchange 事件，根据获取到的最新的 hash 值，切换要显示的组件的名称
        window.onhashchange = function() {
            // 通过 location.hash 获取到最新的 hash 值
            console.log(location.hash);
            switch(location.hash.slice(1)){
            case '/zhuye':
                vm.comName = 'zhuye'
            break
            case '/keji':
                vm.comName = 'keji'
            break
            case '/caijing':
                vm.comName = 'caijing'
            break
            case '/yule':
                vm.comName = 'yule'
            break
            }
        }
        </script>
    </body>
    </html>
```


核心思路：
在页面中有一个vue实例对象，vue实例对象中有四个组件，分别是tab栏切换需要显示的组件内容
在页面中有四个超链接，如下：

```
<a href="#/zhuye">主页</a> 
<a href="#/keji">科技</a> 
<a href="#/caijing">财经</a>
<a href="#/yule">娱乐</a>
```
当我们点击这些超链接的时候，就会改变url地址中的hash值，当hash值被改变时，就会触发onhashchange事件
在触发onhashchange事件的时候，我们根据hash值来让不同的组件进行显示：
```
window.onhashchange = function() {
    // 通过 location.hash 获取到最新的 hash 值
    console.log(location.hash);
    switch(location.hash.slice(1)){
        case '/zhuye':
        //通过更改数据comName来指定显示的组件
        //因为 <component :is="comName"></component> ，组件已经绑定了comName
        vm.comName = 'zhuye'
        break
        case '/keji':
        vm.comName = 'keji'
        break
        case '/caijing':
        vm.comName = 'caijing'
        break
        case '/yule':
        vm.comName = 'yule'
        break
    }
}
```

 # 3. Vue Router简介
它是一个Vue.js官方提供的路由管理器。是一个功能更加强大的前端路由器，推荐使用。
Vue Router和Vue.js非常契合，可以一起方便的实现SPA(single page web application,单页应用程序)应用程序的开发。
Vue Router依赖于Vue，所以需要先引入Vue，再引入Vue Router

Vue Router的特性：
- 支持H5历史模式或者hash模式
- 支持嵌套路由
- 支持路由参数
- 支持编程式路由
- 支持命名路由
- 支持路由导航守卫
- 支持路由过渡动画特效
- 支持路由懒加载
- 支持路由滚动行为

# 4.Vue Router的使用步骤(★★★)
1. 导入js文件
`<script src="lib/vue_2.5.22.js"></script>`
`<script src="lib/vue-router_3.0.2.js"></script>`
2. 添加路由链接:<router-link>是路由中提供的标签，默认会被渲染为a标签，to属性默认被渲染为href属性，to属性的值会被渲染为#开头的hash地址
`<router-link to="/user">User</router-link>`
`<router-link to="/login">Login</router-link>`
3. 添加路由填充位（路由占位符）
`<router-view></router-view>`
4. 定义路由组件
```javascript
var User = { template:"<div>This is User</div>" }
var Login = { template:"<div>This is Login</div>" }
```
5. 配置路由规则并创建路由实例
```javascript
var myRouter = new VueRouter({
    //routes是路由规则数组
    routes:[
        //每一个路由规则都是一个对象，对象中至少包含path和component两个属性
        //path表示  路由匹配的hash地址，component表示路由规则对应要展示的组件对象
        {path:"/user",component:User},
        {path:"/login",component:Login}
    ]
})
```
6. 将路由挂载到Vue实例中
```javascript
new Vue({
    el:"#app",
    //通过router属性挂载路由对象
    router:myRouter
})
```

小结：
Vue Router的使用步骤还是比较清晰的，按照步骤一步一步就能完成路由操作
A.导入js文件
B.添加路由链接
C.添加路由占位符(最后路由展示的组件就会在占位符的位置显示)
D.定义路由组件
E.配置路由规则并创建路由实例
F.将路由挂载到Vue实例中

补充：
路由重定向：可以通过路由重定向为页面设置默认展示的组件
在路由规则中添加一条路由规则即可，如下：
```javascript
var myRouter = new VueRouter({
    //routes是路由规则数组
    routes: [
        //path设置为/表示页面最初始的地址 / ,redirect表示要被重定向的新地址，设置为一个路由即可
        { path:"/",redirect:"/user"},
        { path: "/user", component: User },
        { path: "/login", component: Login }
    ]
})
```

# 5.嵌套路由，动态路由的实现方式
## A.嵌套路由的概念(★★★)
当我们进行路由的时候显示的组件中还有新的子级路由链接以及内容。

嵌套路由最关键的代码在于理解子级路由的概念：
比如我们有一个/login的路由
那么/login下面还可以添加子级路由，如:
/login/account
/login/phone

参考代码如下：
```
var User = { template: "<div>This is User</div>" }
//Login组件中的模板代码里面包含了子级路由链接以及子级路由的占位符
    var Login = { template: `<div>
        <h1>This is Login</h1>
        <hr>
        <router-link to="/login/account">账号密码登录</router-link>
        <router-link to="/login/phone">扫码登录</router-link>
        <!-- 子路由组件将会在router-view中显示 -->
        <router-view></router-view>
        </div>` }

    //定义两个子级路由组件
    var account = { template:"<div>账号：<input><br>密码：<input></div>"};
    var phone = { template:"<h1>扫我二维码</h1>"};
    var myRouter = new VueRouter({
        //routes是路由规则数组
        routes: [
            { path:"/",redirect:"/user"},
            { path: "/user", component: User },
            { 
                path: "/login", 
                component: Login,
                //通过children属性为/login添加子路由规则
                children:[
                    { path: "/login/account", component: account },
                    { path: "/login/phone", component: phone },
                ]
            }
        ]
    })

    var vm = new Vue({
        el: '#app',
        data: {},
        methods: {},
        router:myRouter
    });
```


## B.动态路由匹配(★★★)

```javascript
var User = { template:"<div>用户：{{$route.params.id}}</div>"}

var myRouter = new VueRouter({
    //routes是路由规则数组
    routes: [
        //通过/:参数名  的形式传递参数 
        { path: "/user/:id", component: User },
        
    ]
})
```

补充：
如果使用`$route.params.id`来获取路径传参的数据不够灵活。
1.我们可以通过props来接收参数
```javascript
var User = { 
    props:["id"],
    template:"<div>用户：{{id}}</div>"
    }

var myRouter = new VueRouter({
    //routes是路由规则数组
    routes: [
        //通过/:参数名  的形式传递参数 
        //如果props设置为true，route.params将会被设置为组件属性
        { path: "/user/:id", component: User,props:true },
        
    ]
})

```
2.还有一种情况，我们可以将props设置为对象，那么就直接将对象的数据传递给
组件进行使用
```javascript
var User = { 
    props:["username","pwd"],
    template:"<div>用户：{{username}}---{{pwd}}</div>"
    }

var myRouter = new VueRouter({
    //routes是路由规则数组
    routes: [
        //通过/:参数名  的形式传递参数 
        //如果props设置为对象，则传递的是对象中的数据给组件
        { path: "/user/:id", component: User,props:{username:"jack",pwd:123} },
        
    ]
})
```

3.如果想要获取传递的参数值还想要获取传递的对象数据，那么props应该设置为
函数形式。
```javascript
var User = { 
    props:["username","pwd","id"],
    template:"<div>用户：{{id}} -> {{username}}---{{pwd}}</div>"
    }

var myRouter = new VueRouter({
    //routes是路由规则数组
    routes: [
        //通过/:参数名  的形式传递参数 
        //如果props设置为函数，则通过函数的第一个参数获取路由对象
        //并可以通过路由对象的params属性获取传递的参数
        //
        { path: "/user/:id", component: User,props:(route)=>{
            return {username:"jack",pwd:123,id:route.params.id}
            } 
        },
        
    ]
})
```


# 6.命名路由以及编程式导航
## A.命名路由：给路由取别名
案例：

```javascript
var myRouter = new VueRouter({
    //routes是路由规则数组
    routes: [
        //通过name属性为路由添加一个别名
        { path: "/user/:id", component: User, name:"user"},
        
    ]
})
```

添加了别名之后，可以使用别名进行跳转
```html
<router-link to="/user">User</router-link>
<router-link :to="{ name:'user' , params: {id:123} }">User</router-link>

```
还可以编程式导航
`myRouter.push( { name:'user' , params: {id:123} } )`

## B.编程式导航(★★★)
页面导航的两种方式：
1. 声明式导航：通过点击链接的方式实现的导航
2. 编程式导航：调用js的api方法实现导航

Vue-Router中常见的导航方式：
```javascript
this.$router.push("hash地址");
this.$router.push("/login");
this.$router.push({ name:'user' , params: {id:123} });
this.$router.push({ path:"/login" });
this.$router.push({ path:"/login",query:{username:"jack"} });

this.$router.go( n );//n为数字，参考history.go
this.$router.go( -1 );
```