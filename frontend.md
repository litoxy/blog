### 响应式数据的理解
- initData 用户传入数据
- observe 数据观测
    - 观测对象
    **this.walks**遍历对象的所有属性**defineReactive**，通过**Object.defineProperty**进行属性观测。拦截属性的获取，进行依赖收集(dep 数组收集当前的watcher).如果属性变化会通知相关依赖进行更新操作。
    - 观测数组
    使用函数劫持发的方式,重写了数组的方法（push pop shift unsift splice sort reverse).因为只有这七个方法才会改变数组.劫持这几个方法,在用户用这方法的时。还是会调用原有方法，但是会同时用**dep.notify**通知视图更新。会给数组的每一项进行观测，监听他们的变化


### 为何Vue采用异步渲染。
- vue是组件级更新,如果不采用异步更新。每次跟新数据都会对当前数组进行重新渲染。所以为了性能考虑,vue会在本轮数据跟新后，再去异步跟新视图(nextTick).每次渲染会把watcher放在一个队列里面。放入队列的时候会通过watcher id判断你是不是已经存在,如果存在不会在插入队列。完了会使用nextTick 全部更新

### nextTick实现原理
- 经常使用nextTick保证视图跟新完成,是因为我们把这个放在队列的最后
netxTick是一个异步的方法,主要使用了红任务和微任务，定义了一个异步方法。多次调用这个会将方法存入队列中》通过这个异步方法清空当前队列。
    -  是不是支持promise 如果支持 用 promise 的then 
    - 如果不是ie 但是支持Mutationobserver 
    - setImmediate
    - setTimeout

### Vue中computed(计算属性)的特点

- computed也是一个watcher是具备缓存的,只要依赖属性发生变化才会跟新视图，method只要变化就会更新。性能更低
- computed 和 watch 的区别
    -  计算属性的watcher 优先于 渲染watcher
      

### watch中deep:true是如何实现的
   - 如果用户指定了watch中的deep属性为true时候，如果当前监听的是数组。会对对象中的每一项进行求知，此时会将当前的watcher存入到对应的属性依赖中。这样数组中对象发生变化也会通知数据跟新。
   - 如果lazy 为true 会调用 get 方法,取值,进行依赖收集。computer 没有deep true 是用的在模板的，使用JSON.stringify.对对象的所有属性求职.将watcher放在全局上
   - watcher 计算属性 渲染 用户定义

### vue组件的生命周期
#### 理解
#### 每个声明周期什么时候调用
- beforeCreate 实力初始化之后,数据观测 data observer 之前别调用
- created 实例已经创建完成之后调用，这一步实例已经完成以下的配置：
    - 数据观测 dataobserver
    - 属性和计算方法
    - watch/event 时间回调.这里没有$el dom
- beforeMount 在挂载之前被调用，相关的render函数首次被调用
- mounted el被新创建的**vm.$el**替换,并挂载到实例上去之后调用改钩子
- beforeupdate 数据更新的时候调用，发生在虚拟DOM重新渲染之前
- updated 由于数据更改导致的虚拟DOM重新渲染和打不动，在这后会调用
- beforeDestory 实例销毁之前调用。这一步，实例仍然完全可用
- destoryed 实例校徽之后。调用后，vue实例指示的所用东西都会被绑定，所有的时间监听器会被移除。所有的子实例也会被销毁。在服务端渲染期间不被调用

#### 掌握每个生命周期内部可以作甚
- created 实例已经创建完成,因为最早出发的原因可以进行一些数据资源请求（ajax）.
- mounted 实例已经完成挂载,可以进行一些dom操作
- beforeUpdated 可以进行进一步的更改状态,不会出发附
- updated 可以执行以来Dom操作,尽量避免在这个阶段进行状态更新,可能会导致死循环
- destoryed 清空定时器，解除绑定事件
#### AJAX 请求在那个生命周期
- created 中可以，但是这个阶段拿不到真实的dom $el.无法找到相关的元素
- mounted 有dom已经都渲染了，可以直接操作dom节点.但是服务端渲染(ssr)不支持mounted，因为服务端渲染都是通过请求返回了html字符串
#### 啥时候用beforeDestroy
- 如果页面中使用了$on 方法,需要在这个时候解绑 $off
- 清除定义的定时器
- 解绑时间 scroll mousemove
### Vue模板编译原理
- 将模板转换成ast树(用一个对象描述这个语法)

> 用Html成对标签的方式
```
{
    tag:"div",
    type:1,
    children:[Attrs],
    attrs:[{"name":id,value:"container"}],
    parent:null
}
```

- 将ast数（对象）生成js源码
- 包装成函数
```
let code = generate(root) 
//with 解决作用域的问题
let render = `with(this){return ${code}}`

//---

with(obj){
    console.log(a)
}

等价于

console.log(obj.a)

```
#### v-if v-show 区别
- v-if如果条件不成立不会渲染dom
- v-show 会渲染只是会隐藏

##### 原理
- 在模板渲染的时候，判断有没有v-if = true如果没有 返回 createEmptyVmNode

- v-show 编译出来的时候是一个指令在运行的时候，如果v-show 会给style 加上display none

##### 为什么 v-if 和 v-for 不能连用
v-if 的优先级高于v-for，在渲染的时候。会按照循环次数多次渲染，性能会很低

#### 用vmode 来描述一个dom结构
可以用一个对象来描述dom结构是虚拟节点
```
<div id="container"><p></p></div>

let obj = {
        tag:"div",
        data:{
            id:"container"
        },
        children:[
        {
            tag:"p",
            data:{},
            children:[]
        }
        ],
        text:"111"
    
}
render(){
    return _c("div",{ id:"container",_c("p",{})})
}

```
> tempate ast树=》codegen(生成代码)=》转换成render函数=》内部调用_c方法=》虚拟DOM
#### vue diff算法原理 o(n3) 
- 同级比较
#### v-for中为什么有key.
- dom diff 的时候，可以通过key 这个值来判断如果是同级别只是会替换，而不会销毁。增加效率。
因为有了key之后




### wbpack 打包原理
#### AST 抽象语法树
- babel  
- sass less 
- eslink 
- typescript
都转换成抽象语法树后再去允许。
- 拿到入口文件的内容(es6代码)

##### 使用fs文件处理插件，读取入口文件的内容。
- bable 转换成ast
##### 然后把读取的内容解析成ast使用bable的包重的*bable* 中的*parse*方法解析上面中的内容 ~~bable.parse~~
- 然后做两件事 第一就是变量语法书查找依赖。第二把es6转换成es5
##### 使*babale traverse*操作ast,

```
traverse(ast,{
    
    
})
```

- 生成文件依赖图 graph 
- 实现 cmd api 整合 模块化代码
- 打包到bundle.js中
### 浏览器渲染原理与过程
首先用户看到的页面实际上有两个过程:
- 页面内容加载完成 DOMContentLoad事件触发的时候,说明Dom完成。不包括资源
- 页面资源加载完成 Load事件触发的时候，页面上所有的DOM、样式表、图片都加载完成了
#### 浏览器页面渲染有以下几个个步骤
- 浏览器将获取的html文档解析成DOM树

> 在DOM树渲染的过程中,当html解析到script标签，会立即阻塞dom树的构建。让JS引擎处理js。结束后再从中断的地方恢复DOM树的构建，所以为了页面的渲染速度。一般将JS放在最后

- 处理CSS标记，构成叠层样式模表型CSSOM(CSSOBJECT MODEL)
- 将DOM和CSSOM合并为构建渲染树，请输出到绘制流程，将像素输出到屏幕

#### 页面的重绘 repaint 和重排（回流） reflow
- 重绘 屏幕的一部分要重绘,渲染树的节点发生改变.但不影响节点在页面当中的空间位置大小。譬如某个div标签节点的背景颜色、字体颜色繁盛改变但是节点的宽高和内外边距不发生编号。此时出发浏览器的重绘
- 重排 某个渲染数的几点发生了改变,影响了节点的集合属性。如 宽高 内网边距 或者是 float position display:none等等

##### 如何减少重排/回流
reflow 成本高于 repaint .reflow 会导致耗电和渲染慢
1. 直接改变className，如果动态改变样式，则使用cssText（考虑没有优化的浏览器）；

2. 让要操作的元素进行”离线处理”，处理完后一起更新；

    a. 使用DocumentFragment进行缓存操作,引发一次回流和重绘；
    b. 使用display:none技术，只引发两次回流和重绘；
    c. 使用cloneNode(true or false) 和 replaceChild 技术，引发一次回流和重绘；

3.不要经常访问会引起浏览器flush队列的属性，如果你确实要访问，利用缓存；

4. 让元素脱离动画流，减少回流的Render Tree的规模；

#### JS闭包
```
var a = '111';
function b (){
    console.log(a)
}
```
>「函数」和「函数内部能访问到的变量」（也叫环境）的总和，就是一个闭包。

##### 闭包的作用

包常常用来「间接访问一个变量」。换句话说，「隐藏一个变量」。



假设我们在做一个游戏，在写其中关于「还剩几条命」的代码。

如果不用闭包，你可以直接用一个全局变量：
```
window.lives = 30 // 还有三十条命
```
这样看起来很不妥。万一不小心把这个值改成 -1 了怎么办。所以我们不能让别人「直接访问」这个变量。怎么办呢？
用局部变量。
但是用局部变量别人又访问不到，怎么办呢？
暴露一个访问器（函数），让别人可以「间接访问」。
```
!function(){

  var lives = 50

  window.奖励一条命 = function(){
    lives += 1
  }

  window.死一条命 = function(){
    lives -= 1
  }

}()
```
#### VUE父子组件兄弟组件通信
- props & $emit
- $refs
- .sync语法糖
- $attrs 和 $listeners
- provide / inject
```
<div id="app">
  <son></son>
</div>
```
```
let Son = Vue.extend({
  template: '<h2>son</h2>',
  inject: {
    house: {
      default: '没房'
    },
    car: {
      default: '没车'
    },
    money: {
      // 长大工作了虽然有点钱
      // 仅供生活费，需要向父母要
      default: '￥4500'
    }
  },
  created () {
    console.log(this.house, this.car, this.money)
    // -> '房子', '车子', '￥10000'
  }
})

new Vue({
  el: '#app',
  provide: {
    house: '房子',
    car: '车子',
    money: '￥10000'
  },
  components: {
    Son
  }
})
```
- Vuex 兄弟
- $emit $on

#### VUE 热更新原理
Vue cli 热重载是请依赖于VUE框架的，利用vue的watcher监听，通过vue的生命周期函数进行名称模块的变更的替换。webpack是利用sock.js进行浏览器端和本地的服务器端通信，本地watch监听则是webpack 和 webpack-dev-server对模块名称的监听。替换过程使用的是jsonp/ajax

#### 防抖和节流
- 防抖
1.1 什么是防抖
在事件被触发n秒后再执行回调函数，如果在这n秒内又被触发，则重新计时。

1.2 应用场景
(1) 用户在输入框中连续输入一串字符后，只会在输入完后去执行最后一次的查询ajax请求，这样可以有效减少请求次数，节约请求资源；

(2) window的resize、scroll事件，不断地调整浏览器的窗口大小、或者滚动时会触发对应事件，防抖让其只触发一次；

1.3 实现
