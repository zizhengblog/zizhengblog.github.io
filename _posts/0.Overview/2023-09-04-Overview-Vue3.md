---
layout: post
title: "Vue3概要"
date: 2023-09-04
tag: Overview
---   




## 目录
* [组合式api](#content1)
* [父子通讯](#content2)
* [双向绑定](#content3)
* [路由](#content4)
* [pinia](#content5)
* [pinia的持久化](#content6)




<!-- ************************************************ -->
## <a id="content1">组合式api</a>

#### **一、文件样式**   
```js
// 行为
<script setup>
// setup语法糖，比beforeCreate执行时机更早     

// 用来声明响应数据      
import { ref } from 'vue'

// 声明普通数据     
const message = 'this is a message'
const logMessage = () => {
  console.log(message)
}

// 声明响应数据
const count = ref(0)
const setCount = () => {
  //使用相应数据
  count.value++
}
</script>

// 结构
<template>
  // 普通数据的使用
  <div>\{\{ message \}\}</div>
  <button @click="logMessage">按钮</button>

  // 响应数据的使用   
  <div>\{\{ count \}\}</div>
  <button @click="setCount">点击加1</button>
</template>


// 样式
<style lang="scss" scoped>
</style>
```


#### **二、几个模块**    
```js
<script setup>
import { ref } from 'vue'

// 响应式数据使用ref
const person = ref({
  name:'xiaoming',
  age:18
})
const list = ref([1, 2, 3, 4, 5, 6])

// 计算属性
const computedList = computed(() => {
  return list.value.filter(item => item > 2)
})

// 方法
const addAge = function () {
  person.value.age++
}

// 监听某个属性
watch(function () {

  // return person.value

  // 可以监听某个具体的属性
  return person.age.value
},function (newValue, oldValue){
  console.log(newValue, oldValue)
}, {
  deep:true,
  immediate:true
})
</script>
```

#### **三、声明周期钩子函数**   
```js
<script setup>
  console.log('setUp')
  console.log(document.querySelector('.test')) // null 此时模板还没有渲染

  // 模板渲染前
  onBeforeMount(function(){
    console.log('onBeforeMount')
    console.log(document.querySelector('.test')) // null 此时模板还没有渲染
  })

  // 模板渲染后
  onMount(function(){
    console.log('onMount')
    console.log(document.querySelector('.test')) // 有值
  })

  // 数据修改了，视图还没更新
  onBeforeUpdate(function(){
    console.log('onBeforeUpdate')
  })

  // 数据修改了，视图已经更新
  onUpdated(function(){
    console.log('onUpdated')
  })

  // 卸载前
  onBeforeUnmount(function(){
    console.log('onBeforeUnmount')
  })

  // 卸载后
  onUnmount(function(){
    console.log('onUnmount')
  })
</script>
```


<!-- ************************************************ -->
## <a id="content2">父子通讯</a>

#### **一、与vue2一样**  
父向子传值与vue2一样    
子组件引入后，不需要注册    
```js
<script setup>
import { ref } from 'vue'
import SonCom from '@/components/SonCom.vue'

const person = ref({
  name:'xiaoming',
  age:18,
  car:'奥迪',
  money:'200万'
})
</script>

<template>
  <div>
    <SonCom
      :car="person.car"
      :money="person.money"
      @changeMoney="person.money = $event">
    </SonCom>
  </div>
</template>

<style scoped></style>
```

SonCom 子组件   
props需要定义    
emit事件名需要定义    
```js
<script setup>
// 注意：由于写了 setup，所以无法直接配置 props 选项
// 所以：此处需要借助于 “编译器宏” 函数接收子组件传递的数据
const props = defineProps({
  car: String,
  money: Number
})
const emit = defineEmits(['changeMoney'])
console.log(props.car)
console.log(props.money)

const buy = () => {
  // 需要 emit 触发事件
  emit('changeMoney', 5)
}
</script>

<template>
  <!-- 对于props传递过来的数据，模板中可以直接使用 -->
  <div class="son">
    我是子组件 - {{ car }} - {{ money }}
    <button @click="buy">花钱</button>
  </div>
</template>

<style scoped>
.son {
  border: 1px solid #000;
  padding: 30px;
}
</style>
```

#### **二、provide 和 inject**     

```js
<script setup>
import CenterCom from '@/components/center-com.vue'
import { provide, ref } from 'vue'

// 1. 跨层传递普通数据
provide('theme-color', 'pink')

// 2. 跨层传递响应式数据
const count = ref(100)
provide('count', count)

setTimeout(() => {
  count.value = 500
}, 2000)

// 3. 跨层传递函数 => 给子孙后代传递可以修改数据的方法
provide('changeCount', (newCount) => {
  count.value = newCount
})

</script>

<template>
<div>
  <h1>我是顶层组件</h1>
  <CenterCom></CenterCom>
</div>
</template>
```

```js
<script setup>
import BottomCom from './bottom-com.vue'
</script>

<template>
<div>
  <h2>我是中间组件</h2>
  <BottomCom></BottomCom>
</div>
</template>
```

```js
<script setup>
import { inject } from 'vue'
const themeColor = inject('theme-color')
const count = inject('count')
const changeCount = inject('changeCount')
const clickFn = () => {
  changeCount(1000)
}
</script>

<template>
<div>
  <h3>我是底层组件-{{ themeColor }} - {{ count }}</h3>
  <button @click="clickFn">更新count</button>
</div>
</template>
```


<!-- ************************************************ -->
## <a id="content3">双向绑定</a>

#### **一、v-model的原理跟vue2中相似**         

传值       
:modelValue = ‘变量’ @update:modelValue=’变量=$event’    
简写成：v-model = ‘变量’      

内部：    
emit(‘update:modelValue’,新值)     

```js
<script setup>
import { ref } from 'vue'
import SonCom from '@/components/SonCom.vue'

const person = ref({
  name:'xiaoming',
  age:18,
  car:'奥迪',
  money:'200万'
})
</script>

<template>
  <div>
    <SonCom v-model="person.car"></SonCom>
  </div>
</template>

<style scoped></style>
```

SonCom 子组件   
```js
<script setup>
defineProps({
  modelValue: String,
})
const emit = defineEmits(['update:modelValue'])
const fn = function (e) {
  // emit('update:modelValue', e.target.value)
  emit('update:modelValue', '问界')
}
</script>

<template>
   <input type="text" :value="modelValue" @input="fn">
</template>
```

#### **二、实验属性**    

**1、使用defineModel(实验属性)**         
外部不用变，内部使用defineModel    

```js
<script setup>

const modelValue = defineModel()
const fn = function (e) {
  modelValue.value = e.target.value
}
</script>

<template>
  <input type="text" :value="modelValue" @input="fn">
</template>
```

**2、在vite.config.js中增加配置**   

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    vue({
      script: {
        defineModel: true
      }
    }),
  ],
})
```



<!-- ************************************************ -->
## <a id="content4">路由</a>

#### **一、引入vue-router包**       
vue3使用的vue-router是4.x版本    

```js
"dependencies": {
  "@element-plus/icons-vue": "^2.1.0",
  "@vueup/vue-quill": "^1.2.0",
  "axios": "^1.4.0",
  "element-plus": "^2.3.7",
  "pinia": "^2.1.3",
  "vue": "^3.3.4",
  "vue-router": "^4.2.2"
},
```


#### **二、main.js挂载**      
```js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

const app = createApp(App)
app.use(router)
app.mount('#app')
```

#### **三、设置路由出口**    
```js
<template>
  <div>
    <router-view></router-view>
  </div>
</template>

<style scoped></style>
```

#### **四、编辑路由**     
在单独的文件中router/index.js编辑路由     

```js
import { createRouter, createWebHistory } from 'vue-router'

// createRouter 创建路由实例
// 创建路由实例由 createRouter 实现:是对 new VueRouter()的封装

// 配置 history 模式
// 1. history模式：createWebHistory     地址栏不带 #
// 2. hash模式：   createWebHashHistory 地址栏带 #
// console.log(import.meta.env.DEV)

// 参数是基础路径
// vite 中的环境变量 import.meta.env.BASE_URL  就是 vite.config.js 中的 base 配置项
// import.meta.env.BASE_URL  默认/
// 修改的话在 vite.config.js 中的 path: '/',


const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    { path: '/login', component: () => import('@/views/login/LoginPage.vue') }, // 登录页
    {
      path: '/',
      component: () => import('@/views/layout/LayoutContainer.vue'),
      redirect: '/article/manage',
      children: [
        {
          path: '/article/manage',
          component: () => import('@/views/article/ArticleManage.vue')
        },
        {
          path: '/article/channel',
          component: () => import('@/views/article/ArticleChannel.vue')
        },
        {
          path: '/user/profile',
          component: () => import('@/views/user/UserProfile.vue')
        },
        {
          path: '/user/avatar',
          component: () => import('@/views/user/UserAvatar.vue')
        },
        {
          path: '/user/password',
          component: () => import('@/views/user/UserPassword.vue')
        }
      ]
    }
  ]
})

export default router
```


#### **五、路由的跳转传参**   

编程式导航

**1、在结构中跟vue2的使用方式相同**      
```js
<button @click="$router.push('/home')">跳转首页</button>
```

**2、在行为中**   
由于vue3的setup中没有this,所以跟vue2中的使用有所不同  

```js
// vue2 
// 跳转
this.$router.push('path')  
// 获取参数
this.$route.query.key

// vue3
// 跳转：这里是router
const router = useRouter();
router.push('path');

// 获取参数：这里是route
const route = useRoute();
route.query.key;
```


<!-- ************************************************ -->
## <a id="content5">pinia</a>

#### **一、安装pinia**   
在 package 文件中引用    
```js
"dependencies": {
  "pinia": "^2.1.3",
  "vue": "^3.3.4",
  "vue-router": "^4.2.2"
},
```

或者  
```js
npm install pinia
```

#### **二、在main.js中配置pinia**        
```js
import { createApp } from 'vue'
import App from './App.vue'

import { createPinia } from 'pinia'

const pinia = createPinia()

const app = createApp(App)

app.use(pinia)

app.mount('#app')
```

#### **三、创建一个仓库**    

在src/store/counter.js 文件内   
```js
import { defineStore } from 'pinia'
import { computed, ref } from 'vue'
import axios from 'axios'

// useCounterStore是一个函数
export const useCounterStore = defineStore('counter', () => {
    // 相当于state
    const count = ref(100)
    const channelList = ref([])

    // function 就是 actions
    // action 合并了vuex中的actions 和 mutations
    function addfn() {
        count.value++
    }
    // 支持异步
    const getList = async () => {
        const url = 'http://geek.itheima.net/v1_0/channels'
        const { data: { data } } = await axios.get(url)
        channelList.value = data.channels
    }

    // computed 就是getters
    const double = computed(() => {
        return count.value * 2
    })
    return { count, channelList, addfn, getList, double }
})
```


#### **四、使用共享数据**    

```js

<script setup>
import { useCounterStore } from '@/store/counter';
import { storeToRefs } from 'pinia'

// 调用后会返回一个包含响应式信息的对象(还有很多其它的内容)
const counterStore = useCounterStore()
// 不能直接解构会丢失响应式
const { count, double } = storeToRefs(counterStore)
// 方法可以直接解构
const { getList } = counterStore
</script>

<template>
    <div>
        <slot></slot>
        <div>SonCom使用数据: \{\{ counterStore.count \}\} - \{\{ counterStore.double \}\} - \{\{ count }} - \{\{ double \}\}</div>
        <div @click="counterStore.addfn">加数据</div>
        <div @click="counterStore.getList">获取列表</div>
        <div @click="getList">获取列表</div>
        <div v-for="item in counterStore.channelList" :key="item.id"> \{\{ item.name \}\} </div>
    </div>
</template>

<style scoped></style>
```


<!-- ************************************************ -->
## <a id="content6">pinia的持久化</a>

#### **一、安装插件**   
```shell
npm install pinia-plugin-persistedstate
```
或者在文件中配置，然后执行`npm install`          
```js
"dependencies": {
  "axios": "^1.4.0",
  "element-plus": "^2.3.7",
  "pinia": "^2.1.3",
  "vue": "^3.3.4",
  "vue-router": "^4.2.2"
},
"devDependencies": {
  "pinia-plugin-persistedstate": "^3.1.0",
},
```

```js
"dependencies": {
  "axios": "^1.4.0",
  "pinia": "^2.1.4",
  "pinia-plugin-persistedstate": "^3.1.0",
  "vue": "^3.3.4"
},
```
疑问：不确定是放在 dependencies 还是 devDependencies 中    

#### **二、在main.js中配置**      
```js
import { createApp } from 'vue'
import { createPinia } from 'pinia'
// 导入持久化的插件
import persist from 'pinia-plugin-persistedstate'

import App from './App.vue'
const pinia = createPinia() // 创建Pinia实例
const app = createApp(App) // 创建根实例
app.use(pinia.use(persist)) // pinia插件的安装配置
app.mount('#app') // 视图的挂载
```


#### **三、在仓库中配置**    

```js
import { defineStore } from 'pinia'
import { computed, ref } from 'vue'

export const useCounterStore = defineStore('counter', () => {
  const count = ref(100)
  const double = computed(() => count.value * 2)

  return {
    count,
    double,
  }
}, {
  // persist: true // 开启当前模块的持久化
  persist: {
    key: 'hm-counter', // 修改本地存储的唯一标识
    paths: ['count'] // 存储的是哪些数据
  }
})
```


----------
>  行者常至，为者常成！





