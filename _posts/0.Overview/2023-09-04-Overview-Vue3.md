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



----------
>  行者常至，为者常成！



