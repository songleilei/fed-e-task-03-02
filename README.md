# fed-e-task-03-02

Vue.js 源码剖析-响应式原理、虚拟 DOM、模板编译和组件化

一、简答题

#### 1、请简述 Vue 首次渲染的过程。

1. Vue 初始化，实例成员，静态成员
2. new Vue() 调用构造函数
3. 构造函数中调用 this.\_init() 相当于整个 Vue 的入口
4. \_init 方法中调用 vm.\$mount() 主要作用把模板编译成 render 函数
5. vm.\$mount() 运行时版本。重新获取 el，然后调用 mountComponent
6. mountComponent(this,el)

   - 判断是否有 render 选项，如果没有但是传入了模板，并且当前是开发环境的话会发送警告
   - 触发 beforeMount
   - 定义 updateComponent
     1. vm.\_update(vm.\_render(),...)
     2. vm.\_render() 渲染，渲染虚拟 dom
     3. vm.\_update()更新，将虚拟 dom 转换成真实 dom 并挂载到页面上
   - 创建 Watcher 实例 传递 updateComponent 函数；调用 get() 方法
   - 触发 mounted
   - return vm

7. watcher.get()
   - 创建完 watcher 会调用一次 get
   - 调用 updateComponent()
   - 调用 vm.\_render() 创建 vnode
   - 调用 vm.\_update(vnode,...)
     1. 调用 vm.\_patch\_(vm.\$el, vnode) 挂载真实 dom
     2. 记录 vm.\$el

#### 2、请简述 Vue 响应式原理。

1. 从 Vue 实例中的 init 方法开始，initState() 初始化状态 -> initData() 将 data 属性注入到实例上 -> observe() 把 data 对象转换为响应式对象

2. observe(value)

   - 判断 value 是否是对象，如果不是对象直接返回
   - 判断 value 对象是否有 \_ob\_, 如果有直接返回
   - 如果没有，创建 observe 对象
   - 返回 observe 对象

3. Observe

   - 给 value 对象定义不可枚举的\_ob\_属性，记录当前的 observer 对象
   - 数组的响应式处理 主要是设置数组的几个特殊的方法 push/pop 等
   - 对象的响应式处理，调用 walk 方法

4. defineReactive

   - 为每一个属性创建 dep 对象
   - 如果当前属性的值是对象，调用 observe
   - 定义 getter
     1. 收集依赖
     2. 返回属性的值
   - 定义 setter
     1. 保存新值
     2. 如果新值是对象，调用 observe
     3. 派发更新（发送通知），调用 dep.notify()

5. 依赖收集

   - 在 watcher 对象的 get 方法中调用 pushTarget 记录 Dep.target 属性
   - 访问 data 中的成员的时候收集依赖，defineReactive 的 getter 中收集
   - 把属性对应的 watcher 对象添加到 dep 的 subs 数组中
   - 给 childOb 收集依赖，目的是子对象添加和删除成员时发送通知

6. Watcher
   - dep.notify 在调用 watcher 对象的 update()方法
   - queueWatcher() 判断 watcher 是否被处理，如果没有的话，添加到 queue 队列中，并调用 flushSchedulerQueue()
   - flushSchedulerQueue()
     1. 触发 beforeUpdate 钩子函数
     2. 调用 watcher.run()
        1. run()->get()->getter()->updateComponent()
     3. 清空上一次的依赖
     4. 触发 actived 钩子函数
     5. 触发 updated 钩子函数

#### 3、请简述虚拟 DOM 中 Key 的作用和好处。

1. key 的作用：在新旧 nodes 对比时辨识 VNodes。如果不使用 key，Vue 会使用一种最大限度减少动态元素并且尽可能的尝试就地修改/复用相同类型元素的算法。而使用 key 时，它会基于 key 的变化重新排列元素顺序，并且会移除 key 不存在的元素。 有相同父元素的子元素必须有独特的 key。重复的 key 会造成渲染错误。

2. ​ key 的好处：高效的更新虚拟 DOM。

#### 4、请简述 Vue 中模板编译的过程。

1. 模板编译入口函数:compileToFunctions(template, {}, this)

   1. 先从缓存中加载编译好的 render 函数

   2. 缓存中没有则调用 compile(template, options)

2. compile(template, options) 函数

   1. 合并 options

   2. baseCompile(template.trim(), finalOptions) 编译模板

3. baseCompile

   - 解析 parse

     - 把 template 转换成 AST tree

   - 优化 optimize

     - 标记 AST tree 中的静态 sub trees
     - 检测到静态子树时，设置为静态，不需要在每次重新渲染的时候重新生成节点
     - patch 阶段跳过静态子树

   - 生成 generate

     - AST tree 生成 js 的创建代码

4. compile 函数执行完之后，回到 入口函数 compileToFunctions

   - 通过 createFunction() 把上一步中生成的字符串形式 js 代码 转换为函数
   - render 和 staticRenderFns 初始化完毕，挂载到 vue 实例的 options 对应的属性中
