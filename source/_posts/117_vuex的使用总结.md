---
title: vuex的使用总结
date: 2021-01-21 10:04:29
tags: vue
---


## 1.Vuex概述
Vuex是实现组件全局状态（数据）管理的一种机制，可以方便的实现组件之间的数据共享

使用Vuex管理数据的好处：
- 能够在vuex中集中管理共享的数据，便于开发和后期进行维护
- 能够高效的实现组件之间的数据共享，提高开发效率
- 存储在vuex中的数据是响应式的，当数据发生改变时，页面中的数据也会同步更新

## 2.Vuex的基本使用
创建带有vuex的vue项目，打开终端，输入命令：`vue ui`
当项目仪表盘打开之后，我们点击页面左上角的项目管理下拉列表，再点击Vue项目管理器
点击创建项目，如下图所示
**第一步，设置项目名称和包管理器**
![image.png](https://halo-1257208482.image.myqcloud.com/image_1611194723904.png!webp)
**第二步，设置手动配置项目**
![image.png](https://halo-1257208482.image.myqcloud.com/image_1611194748009.png!webp)
**第三步，设置功能项**
![image.png](https://halo-1257208482.image.myqcloud.com/image_1611194763383.png!webp)
![image.png](https://halo-1257208482.image.myqcloud.com/image_1611194774191.png!webp)
**第四步，创建项目**
![image.png](https://halo-1257208482.image.myqcloud.com/image_1611194785805.png!webp)

## 3.使用Vuex完成计数器案例
打开刚刚创建的vuex项目，找到src目录中的App.vue组件，将代码重新编写如下：

```js
<template>
  <div>
    <my-addition></my-addition>

    <p>----------------------------------------</p>

    <my-subtraction></my-subtraction>
  </div>
</template>

<script>
import Addition from './components/Addition.vue'
import Subtraction from './components/Subtraction.vue'

export default {
  data() {
    return {}
  },
  components: {
    'my-subtraction': Subtraction,
    'my-addition': Addition
  }
}
</script>

<style>
</style>
```

在components文件夹中创建Addition.vue组件，代码如下：

```js
<template>
    <div>
        <h3>当前最新的count值为：</h3>
        <button>+1</button>
    </div>
</template>

<script>
export default {
  data() {
    return {}
  }
}
</script>

<style>
</style>
```

在components文件夹中创建Subtraction.vue组件，代码如下：

```js
<template>
    <div>
        <h3>当前最新的count值为：</h3>
        <button>-1</button>
    </div>
</template>

<script>
export default {
  data() {
    return {}
  }
}
</script>

<style>
</style>
```

最后在项目根目录(与src平级)中创建 .prettierrc 文件，编写代码如下：

```javascript
{
    "semi":false,
    "singleQuote":true
}
```

## 4.Vuex中的核心特性
### A.State
State提供唯一的公共数据源，所有共享的数据都要统一放到Store中的State中存储
 例如，打开项目中的store.js文件，在State对象中可以添加我们要共享的数据，如：count:0
在组件中访问State的方式：
1. this.$store.state.全局数据名称  如：`this.$store.state.count`
2. 先按需导入mapState函数：` import { mapState } from 'vuex'`
然后数据映射为计算属性： `computed:{ ...mapState(['全局数据名称']) }`

### B.Mutation
Mutation用于修改变更$store中的数据
使用方式：
打开store.js文件，在mutations中添加代码如下

```javascript
mutations: {
    add(state,step){
      //第一个形参永远都是state也就是$state对象
      //第二个形参是调用add时传递的参数
      state.count+=step;
    }
  }
```
然后在Addition.vue中给按钮添加事件代码如下：
```
<button @click="Add">+1</button>

methods:{
  Add(){
    //使用commit函数调用mutations中的对应函数，
    //第一个参数就是我们要调用的mutations中的函数名
    //第二个参数就是传递给add函数的参数
    this.$store.commit('add',10)
  }
}
```

使用mutations的第二种方式：
```vue
import { mapMutations } from 'vuex'

methods:{
  ...mapMutations(['add'])
}
```
如下：

```js
import { mapState,mapMutations } from 'vuex'

export default {
  data() {
    return {}
  },
  methods:{
      //获得mapMutations映射的sub函数
      ...mapMutations(['sub']),
      //当点击按钮时触发Sub函数
      Sub(){
          //调用sub函数完成对数据的操作
          this.sub(10);
      }
  },
  computed:{
      ...mapState(['count'])
      
  }
}
```




### C.Action
在mutations中不能编写异步的代码，会导致vue调试器的显示出错。
在vuex中我们可以使用Action来执行异步操作。
操作步骤如下：
打开store.js文件，修改Action，如下：
```js
actions: {
  addAsync(context,step){
    setTimeout(()=>{
      context.commit('add',step);
    },2000)
  }
}
```
然后在Addition.vue中给按钮添加事件代码如下：
```vue
<button @click="AddAsync">...+1</button>

methods:{
  AddAsync(){
    this.$store.dispatch('addAsync',5)
  }
}
```

第二种方式：
```js
import { mapActions } from 'vuex'

methods:{
  ...mapMutations(['subAsync'])
}
```
如下：
```js
import { mapState,mapMutations,mapActions } from 'vuex'

export default {
  data() {
    return {}
  },
  methods:{
      //获得mapMutations映射的sub函数
      ...mapMutations(['sub']),
      //当点击按钮时触发Sub函数
      Sub(){
          //调用sub函数完成对数据的操作
          this.sub(10);
      },
      //获得mapActions映射的addAsync函数
      ...mapActions(['subAsync']),
      asyncSub(){
          this.subAsync(5);
      }
  },
  computed:{
      ...mapState(['count'])
      
  }
}
```


### D.Getter
Getter用于对Store中的数据进行加工处理形成新的数据
它只会包装Store中保存的数据，并不会修改Store中保存的数据，当Store中的数据发生变化时，Getter生成的内容也会随之变化
打开store.js文件，添加getters，如下：
```js
export default new Vuex.Store({
  .......
  getters:{
    //添加了一个showNum的属性
    showNum : state =>{
      return '最新的count值为：'+state.count;
    }
  }
})
```
然后打开Addition.vue中，添加插值表达式使用getters
`<h3>{{$store.getters.showNum}}</h3>`

或者也可以在Addition.vue中，导入mapGetters，并将之映射为计算属性
```js
import { mapGetters } from 'vuex'
computed:{
  ...mapGetters(['showNum'])
}
```
## 5. vuex数据持久化
默认情况下vuex中的数据是存储再内存中的，只要浏览器一刷新它就没了，会造成一些不好的体验，所以使用一些插件把vuex中的数据存储到localStorage或者sessionStorage。
1. 安装 `npm install vuex-persistedstate --save`
2. 在store下的index.js中配置，存储在sessionStorage
```js
import createPersistedState from "vuex-persistedstate"
 
conststore = new Vuex.Store({
 
    // ...
 
    plugins: [createPersistedState({
 
        storage:window.sessionStorage
 
    })]
 
})
```




