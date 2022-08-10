

# 概述



## MVVM框架



**优势：**MVVM框架实现了页面和数据的分离，代码结构更加清晰，责任更加明确，同时实现自动化，数据变化，页面随之变化，无需写代码——这是javascript、jquery、bootstrap等无法做到的，也是前端为何开始推崇Vue这些框架的根本原因，也标示着Jquery的终结。 



![image](E:\doc_repo\013-前端\images\image.png)









# 启动工程

> 安装npm后，在工程根目录下cmd进入DOS窗口；
>
> 输入 npm run serve 即可启动项目；
>
> 登录 localhost:8080 登录首页



**代码编辑器**：HBuilder



# 工程架构

## src目录

| 文件夹或文件 |                             作用                             |
| :----------: | :----------------------------------------------------------: |
|   main.js    | 一、入口文件，import工程目录 和 组件；二、全局配置，比如Axios的url前缀等 |
|   app.vue    |               app.vue是主组件，是页面入口文件                |
|    router    | 内部的index.js定义了路由规则 及 路由导航守卫（通过`let token=window.sessionStorage.getItem("token")`获取token，若为null则返回login页面）—— 在components中新完成的网页需要先在最上import |
|   plugins    | element UI 网上找到的组件，必须要在element.js里引入import或者use才能使用 |
|  components  |                           网页文件                           |
|    assets    |                  图标、logo图片、CSS样式等                   |



## 网页文件

|    标签     |                             功能                             |
| :---------: | :----------------------------------------------------------: |
| \<template> |      模板、打样，element UI网页上的 组件代码块 复制于此      |
|  \<script>  | export default { } 内包含 data()、methods()、mounted() 等对象内容 |
|  \<style>   | 利用 元素选择器 选定对应的内容，比如 .class名 / #id名 来设定其样式 |



## Data格式

```javascript
# 方式一 ：将data看作一个参数
data : {
	msg1 : "Hi Siri",
	msg2 : "Hi Vue"
}

# 方式二：将data作为一个方法的返回值 ——> 优点：内部可附带运算，最终把需要的数据return即可
data : function() { 		
	return { msg1 : "Hi Siri" , msg2 : "Hi Vue" } 
}

# 方式三：同方式二
data() {
	return { msg1 : "Hi Siri", msg2 : "Hi Vue" }
}


N.B. 方式三等价于方式二，三对应函数创建方式A*，二对应方式B
方式A：
	function data() {...}
方式B：
	let data = function() {}

```





## V指令

|       指令        |                             功能                             |
| :---------------: | :----------------------------------------------------------: |
|    **v-model**    |           标签内容 与 model数据绑定，MV绑定的基础            |
|    **v-html**     | html语法 解析标签内的内容，比如 model 内容里有 \<h1> 标签，Vue组件默认是不解析的，使用v-html = "变量名"，则可解析 |
|    **v-cloak**    | 针对遇到插值表达式加载，浏览器页面闪烁对用户不好的现象 —— 在所挂载的标签中增加指令：v-cloak |
| **v-if / v-show** | 可判断data是否满足展示条件，比如\<p v-if = "dad.age>=50"> 选择性展示内容 \<p> ；差别在于 v-if 根据条件判断该组件是否加载；v-show则是 先加载再决定display与否 |
|     **v-for**     | 用于绑定数组，如 \<li  v-for = "**index, data** in hobby"><\li>, 若index，data仅有一个参数，则为数据 |
|     **v-on***     | 监听事件，将 事件 与 \<script>里的 methods 绑定，可简写如：v-on:click = login(); 为 @click = login() |
|    **v-bind**     | 常规的html标签属性，比如p标签中的href属性，在\<p href=“url”>中的href属性，url是无法与data里的变量值绑定的，此时需要 \<p v-bind:href="url">, 也可简写为 :href = "url" |



***v-on :** 

- click/dblclick = "XXX()"
- mouseover = "XXX()"
- mouseout = "XXX()"
- focus = "XXX()"
- blur = "XXX()"
- mousemove = "XXX()"
- mouseup/down = "XXX()"





## Ajax格式

**全称**：Asynchronized javascript and XML

**功能**：局部刷新、异步访问

**前提**：利用\<script>标签引入 axios.js 插件 `<script src="../js/axios.js"></script>`

**基本格式：**axios .请求方法 ( url, 请求对象 ) .then( function(promise) {} )

**promise参数**：请求后获取的结果，参数data为结果数据，function(promise) {} 方法体中为拿到结果数据后的回调函数内容，比如 console.log ( promise.data ) ;

|     请求方式     |                           参数传入                           |                        备注                         |
| :--------------: | :----------------------------------------------------------: | :-------------------------------------------------: |
| **get / delete** | 若为常规get / delete方式请求参数在url中，则参数只需要传入url |                                                     |
|                  | 若请求参数以对象的形式封装在前端对象中，则请求对象参数为 **{params : 对象名}** |                                                     |
|  **post / put**  | 请求参数必然以对象的形式封装在前端对象中，请求对象直接传入变量即可 | 2次请求：第一次请求判断是否跨域；第二次正式发起请求 |

**常规请求格式缺陷**：回调地狱问题，每次请求后获取的结果，如果想作为参数再次发起请求，必须嵌套then语句



### async - await

**优势** : 解决回调地狱问题

```javascript
async selectAllUsers() {
    const { data : result } = await axios.get(url, id);
}
```

**案例：**

![Snipaste_2022-08-10_17-30-04](E:\doc_repo\013-前端\images\Snipaste_2022-08-10_17-30-04.png)





## 作用域插槽

**功能：** slot-scope属性可以获取整个单元格的信息，比如商品列表中某件商品的状态更改，需要附带上商品的id，在表格某一列上的操作需要附带上行的信息，此时就需要用上作用域插槽；

再比如，在使用elementUI组件库的表格组件时，表格的编辑 和 删除操作 要用到作用域插槽。因为一个表格组件，就是当前组件的子组件。通过作用域插槽很容易拿到当前表格行的索引id和完整内容，这样就可以很方便地进行编辑展示、删除的操作。



![20220810image](E:\doc_repo\013-前端\images\20220810image.png)













# 常见问题



## Vue组件部分属性前有冒号

- 加冒号的，说明后面的是一个变量或者表达式，比如 :model 与 data 中的loginForm绑定 ；
- 没加冒号的后面就是对应的字符串字面量！ 

![无标题](E:\doc_repo\013-前端\images\无标题.png)







## vue.use()和import的区别





```javascript
import axios from 'axios'
Vue.prototype.$http = axios //全局注册，在工程文件中都可以通过$axios直接来使用axios


/* 导入富文本编辑器 */
import VueQuillEditor from 'vue-quill-editor'

/* 将富文本编辑器注册为全局可用的组件 */
Vue.use(VueQuillEditor)
```








## VUE钩子函数

钩子函数 = 生命周期函数 = 实例在某个时间点会自动执行的函数

Vue钩子函数可以分为3个阶段，一共8个钩子：

- 初始化阶段（创建前 / 后, 载入前 / 后）
- 运行中（更新前 / 后）
- 销毁（销毁前 / 销毁后）

使用最多地是初始化阶段，beforeCreate、Create、beforeMount、Mounted，比如我们 一般**打开网页就会自动呈现加载好的 好友列表、商品列表等都是写在钩子函数中**





## 关于前端知识了解多少？



- Vue工程本地启动，前后端联调；
- 依靠elementUI组件库添加组件；
- **组件 与 数据Model的挂载、前端通过ajax发送异步请求也可以完成；**
- 可以完成基本的 路由守卫规则；



**如何添加一个新的网页： **

- 在components包中新建一个vue文件；
- 页面格式定义、数据挂载；
- 路由文件index.js定义进入规则；









