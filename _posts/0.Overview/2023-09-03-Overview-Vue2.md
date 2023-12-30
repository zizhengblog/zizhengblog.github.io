---
layout: post
title: "Vue2概要"
date: 2023-09-03
tag: Overview
---   




## 目录
* [开发 Vue 的两种方式](#content1)
* [Vue文件](#content2)
* [Vue中的指令](#content3)





<!-- ************************************************ -->
## <a id="content1">开发 Vue 的两种方式</a>

#### **一、传统模式**          
核心包传统开发模式:基于 html / css / js 文件，直接引入核心包，开发 Vue。     

```js
<body>
    <div id="app">
        <h1 class="title">{{title}}</h1>
        <div @click="title = '修改后的标题'">点击修改标题</div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
    <script>
        const app = new Vue({
            el: '#app',
            data: {
                title: '标题'
            }
        })
    </script>
</body>
```



#### **二、工程化开发模式**    

基于构建工具(例如:webpack ) 的环境中开发 Vue。     

**1、什么是脚手架？**     
工程化开发面临的问题：webpack配置不简单、雷同的基础配置、缺乏统一标准     
Vue CLI 是 Vue 官方提供的一个全局命令工具（脚手架工具）。     
可以帮助我们快速创建一个开发 Vue 项目的标准化基础架子。【集成了 webpack 配置:babel less es】     
这个搭建好的标准化的架子就是脚手架。     
babel, less, es6(浏览器不支持)  ————>  webpack配置(打包)  ————>  js(es3, es5), css(浏览器支持)     

xy:脚手架帮助我们创建一个集成了打包工具的标准化项目    

**2、脚手架的安装**  
安装脚手架工具：`npm i @vue/cli -g`          
查看 Vue 版本:`vue --version`       
创建项目架子:`vue create projectName`(项目名-不能用中文)       
启动项目: `npm run serve`(找package.json)       
       
运行npm run serve时报错。错误如下Error: The project seems to require yarn but it's not installed.        
清除yarn.lock 文件 重新 `npm install` 和 `npm run serve`       

**3、项目的目录结构**  

<div style="display:flex;flexDirection:row;justifyContent:flexStar">
    <img src="/images/vue/1.png" style="height:300px;margin:0 10px;">
    <div>
        目录：src、public、dist、node_module<br>
        文件：<br>
        src/main.js： 默认的入口文件<br>
        src/App.vue：根组件<br>
        public/index.html：默认的模板文件<br>
    </div>
</div>



<!-- ************************************************ -->
## <a id="content2">Vue文件</a>

#### **一、三个部分：结构、行为、样式**    

```js
// 结构
<template>
  <div class="search-box">
     <button @click="goSearch">搜索一下</button>
  </div>
</template>


// 行为
<script>
export default {
  name: 'FindMusic',
  data() {
    return {
      inpValue: '搜索一下'
    }
  },
  methods: {
    goSearch() { }
  }
}
</script>


// 样式
<style>
.search-box {
  display: flex;
  justify-content: center;
}

.search-box button {
  width: 100px;
  height: 36px;
  border: none;
  background-color: #ad2a26;
  color: #fff;
  position: relative;
  left: -2px;
  border-radius: 0 4px 4px 0;
}
</style>
```

#### **二、组件：根组件、全局组件、普通组件**    

**1、根组件**     
src/App.vue 就是根组件    

**2、全局组件**     
在main.js中注册           
```js
import Vue from 'vue'
import BaseGoodsItem from './components/BaseGoodsItem'
Vue.component('BaseGoodsItem', BaseGoodsItem)
```

**3、普通组件**  
在其它组件内注册       
```vue
<template>
  <div class="App">
    <XtxShortCut></XtxShortCut>
  </div>
</template>

<script>
import XtxShortCut from './components/XtxShortCut.vue'
export default {
  data () {
    return {
      count: 0
    }
  },
  
  // 注册局部组件
  components: {
    XtxShortCut,
  }
}
</script>
```

#### **三、生命周期**   
```js
export default {
    data () {
        return {
            count: 0
        }
    },

    // 创建阶段：创建响应式数据
    beforeCreate() {
        // beforeCreate 响应式数据准备好之前 undefined
        console.log('beforeCreate 响应式数据准备好之前', this.title)
    },
    created() {
        // created 响应式数据准备好之后 标题
        console.log('created 响应式数据准备好之后', this.title)
    },

    // 挂载阶段：渲染模版
    beforeMount() {
        // beforeMount 模板渲染之前 {{title}}
        console.log('beforeMount 模板渲染之前', document.querySelector('.title').innerHTML)
    },
    mounted() {
        // mounted 模板渲染之后 标题
        console.log('mounted 模板渲染之后', document.querySelector('.title').innerHTML)
        document.querySelector('.focus-test').focus()
    },

    // 更新阶段：修改数据，更新视图
    beforeUpdate() {
        console.log('beforeUpdate 数据修改了，视图还没更新', document.querySelector('.title').innerHTML)
    },
    updated() {
        console.log('updated 数据修改了，视图已经更新', document.querySelector('.title').innerHTML)
    },

    // 销毁阶段：销毁实例，在控制台输入：app.$destroy(); 模拟销毁流程
    beforeDestroy() {
        console.log('beforeDestroy, 卸载前')
        console.log('清除掉一些Vue以外的资源占用，定时器，延时器...')
    },
    destroyed() {
        console.log('destroyed，卸载后')
    }
}
```


#### **四、其它**  
```js
export default {
    // 数据
    data () {
        return {
            title:'大标题',
            message: 'Hello, Vue.js',
            obj:{
                words:''
            }
        }
    },
    // 属性
    props:['userName'],
    // 计算属性：本质就是方法
    computed: {
        upMessage:function () {
            // 在这里编写计算逻辑
            return this.message.toUpperCase();
        }
    },
    methods:{
        editFn () {}
    },
    watch: {
        title(newValue) {
            console.log('变化了', newValue)
        },
        'obj.words'(newValue) {
            console.log('变化了', newValue)
        },
        obj: {
            deep:true,//深度监视
            immediate:true,//立即执行，一进入页面handler就立刻执行一次
            handler(newValue) {}
        },
        components:{
            XtxBanner
        }
    }
}
```



<!-- ************************************************ -->
## <a id="content3">Vue中的指令</a>

#### **一、常用指令**         
v-bind 动态的设置html的标签属性(src, title, url等)v-bind:src="url" ，可以简写为 :src="url"  
```js
<img :src='url'>
```

v-for  基于数据循环， 多次渲染整个元素  v-for="(item, index) in booksList" :key="item.id" 不要把key漏掉   
```js
<ul>
    <li v-for="(item, index) in list" :key="item.id">{{item.name}}</li>
</ul>
```   
v-model 可以让数据和视图，形成双向数据绑定 
```js
<input type="text" v-model="inputText">
```     
v-on:click="count++" 内联语句、 v-on:click="fn" 配置函数、 @click="fn" 简写、 @click="fn(param)" 参数传递  
```js
<button v-on:click="count++">+</button>
<button v-on:click="fn(param)">+</button>
<button @click="fn(param)">+</button>
```

v-if  底层原理：根据 判断条件 控制元素的 创建 和 移除（条件渲染）v-else-if   v-else   
```js
<p v-if="score >= 90">成绩评定A：奖励电脑一台</p>
<p v-else-if="score >= 70">成绩评定B：奖励周末郊游</p>
<p v-else-if="score >= 60">成绩评定C：奖励零食礼包</p>
<p v-else>成绩评定D：惩罚一周不能玩手机</p>
```
v-show 底层原理：切换 css 的 display: none 来控制显示隐藏      
```js
 <div v-show="flag" class="box">我是v-show控制的盒子</div>
 ```

#### **二、自定义指令**    

**1、局部注册**   
```js
<template>
    <div class="box2" v-loading="isLoading"></div>
</template>

export default {
    directives: {
        loading: {
            // 标签插入文档时调用：el是标签，binding是传递的数据
            inserted (el, binding) {
                binding.value ? el.classList.add('loading') : el.classList.remove('loading')
            },
            // 更新时调用：比如isLoading的值变化了
            update (el, binding) {
                binding.value ? el.classList.add('loading') : el.classList.remove('loading')
            }
        }
    }
}

<style>
.loading:before {
  content: '';
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background: #fff url('./loading.gif') no-repeat center;
}
</style>
```

**2、全局注册**   
在main.js中     
```js
Vue.directive('focus', {
  // 指令所在的dom元素，被插入到页面中时触发
  inserted (el) {

    //el 就是指令所绑定的元素  
    el.focus()
  }
})


// 在其他模板中使用
<input v-focus ref="inp" type="text">
```


----------
>  行者常至，为者常成！



