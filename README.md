# 简答题

## 1、请简述 Vue 首次渲染的过程。

- Vue 初始化，实例成员，调用initGlobal()初始化静态成员
- new Vue()构造函数中调用_init()方法，进行标记设置，判断当前vue是实例还是组件，进行options合并
- _init()中设置Proxy、执行一系列init后，调用入口文件的$mount()挂载
- vm.$mount()中判断是否传入render选项，获取template，存在template是字符串或nodeType，返回innerHTML为模板，不存在template获取el的outerHTML作为模板返回
- 获取到模板后调用compileToFunctions()，将其编译为render函数，存储在options.render中
- 执行mount方法，重新获取el，运行时版本不走entry-runtime-with-compiler.js在runtime/index.js的$mount()中重新获取el
- 调用mountComponent()，判断options中是否有render函数，触发beforeMount，定义updateComponent()方法，用来将虚拟dom转换成真实dom。
- 创建 watcher实例，将vm、updateComponent()传入，先调用一次get()，然后调用updateComponent()，updateComponent中调用_render创建VNode并返回，
- callHook(vm, 'mounted')，进行页面挂载。
- 最后返回vue实例。

## 2、请简述 Vue 响应式原理
- 



## 3、请简述虚拟 DOM 中 Key 的作用和好处
- 主要作用在Vue的虚拟算法中，在新旧nodes对比时，来辨识VNodes。
- 使用key来跟踪节点，可以使节点最大限度的得到重用。

## 4、请简述 Vue 中模板编译的过程
