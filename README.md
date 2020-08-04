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
- 首先initState()初始化实例状态， 在initState()中调用了initData()将data属性注入到vue实例，然后调用observe()将data转换为响应式对象。
- 在observe中对值进行判断，不是对象直接返回，是对象看其是否有__ob__属性，有该属性直接返回，无该属性创建确保一个observer对象并返回
- 创建observer对象时，为value定义不可枚举的__ob__属性，记录当前的observer对象，之后对数组和队形进行响应式处理。重新设置数组的的方法，pop、push、splice等，遍历每个成员，对其调用observe方法，如果成员是对象，会被转换成响应式对象。对象的响应式处理就是调用walk方法，遍历对象的所有属性，对每个属性调用defineReactive方法。
- 在defineReactive方法中，为每一个属性创建dep对象去收集依赖，若当前的属性值是对象，调用observe，将其转换成响应式。defineReactive中的getter，负责收集依赖，包括子对象，然后返回属性值。setter负责保存新值，若新值是对象，同样调用observe将其转换成响应式对象，然后调用dep.notify()发送通知。
- 收集依赖的时候，在watcher对象的get方法中调用pushTarget来记录Dep.target属性，访问data中的成员时收集依赖，defineReactive中的getter收集依赖，然后把属性对象的watcher对象添加到dep的subs数组中。childOb也收集依赖，这样可以在子对象添加和删除成员时发送通知。
- 数据发生变化，dep.notify() 开发发送通知，调用watcher对象的update()方法。在update()中的queueWatcher判定该watcher是否已经被处理，没有处理过的watcher被添加到queue队列，调用flushSchedulerQueue()。
- flushSchedulerQueue中触发 beforeUpdate钩子函数，调用watcher.run()，一次执行run()、get()、getter()、updateComponent()。
- 最后清空上一次的依赖，触发actived钩子函数和updated钩子函数


## 3、请简述虚拟 DOM 中 Key 的作用和好处
- 主要作用在Vue的虚拟算法中，在新旧nodes对比时，来辨识VNodes。
- 使用key来跟踪节点，可以使节点最大限度的得到重用。
- 在diff算法时，减少操作，优化性能。

## 4、请简述 Vue 中模板编译的过程
- 首先调用compileToFunctions，加载render函数，如果缓存中有编译好的render函数，从中读取，如果缓存中无render，调用compile来生成。
- 在compile方法中，先进行options合并，之后调用baseCompile来编译模板，传入trim处理的template以及合并后的options。
- baseCompile中主要有三个步骤，解析->优化->生成。解析时，将template转换成ast树，然后进行优化，标记ast树中的静态子树，这样patch阶段可直接跳过这些静态树，最后将ast树生成js的创建代码。
- 最后再次调用compileToFunctions，将baseCompile生成的js代码转换为函数，调用createFunctions，初始化render和staticRenderFns，并将其挂载到Vue实例options对应的属性。
