# 第 16 章 -- HTML5 脚本编程


## Catalog
- 16.1 跨稳定消息传递
- 16.2 原生拖放
    + 16.2.1 拖放事件 
    + 16.2.2 自定义放置目标
    + 16.2.3 dataTransfer 对象
    + 16.2.4 dropEffect 与 effectAllowed
    + 16.2.5 可拖动
    + 16.2.6 其他成员
- 16.3 媒体元素
    + 16.3.1 属性
    + 16.3.2 事件
    + 16.3.3 自定义媒体播放器
    + 16.3.4 检测编解码器的支持情况
    + 16.3.5 Audio 类型
- 16.4 历史状态管理
- 16.5 小结 




## New Words





## Content
### 16.1 跨稳定消息传递


### 16.2 原生拖放
#### 16.2.1 拖放事件 
#### 16.2.2 自定义放置目标
#### 16.2.3 dataTransfer 对象
#### 16.2.4 dropEffect 与 effectAllowed
#### 16.2.5 可拖动
#### 16.2.6 其他成员


### 16.3 媒体元素
#### 16.3.1 属性
#### 16.3.2 事件
#### 16.3.3 自定义媒体播放器
#### 16.3.4 检测编解码器的支持情况
#### 16.3.5 Audio 类型


### 16.4 历史状态管理
- 历史状态(history state)管理是现在 Web 应用开发中的一个难点. 在现在 Web 应用中,
  用户的每次操作不一定会打开一个全新的页面. 因此 "后退" 和 "前进" 按钮也就失去了作用,
  导致用户很难在不同状态间切换. 要解决这个问题, 首先使用 `hashchange` 事件(第 13
  章曾讨论过). HTML5 通过更新 `history` 对象为管理历史状态提供了方便.
    + tip: `history` 对象为 window 对象的属性. -- 第 8 章. <br/>
      在浏览器的 console 控制台中输入 `window` 在输出的 window 对象内便可看到.
  
  通过 `hashchange` 事件, 可以知道 URL 的参数什么时候发生了变化,
  即什么时候该有所反应.
  
  而通过状态管理 API, 能在不加载新页面的情况下改变浏览器的 URL.
  为此, 需要使用 `history.pushState()` 方法, 该方法可以接收 3 个参数: 
    + (1) 状态对象; (tip: 下面会讲解这个参数) 
    + (2) 新状态的标题
    + (3) 可选的相对 URL.
  ```js
    history.pushState({name: "Nicholas"}, "Nicholas's page", "nicholas.html");
  ```
  执行 `pushState` 方法后, 新的状态信息就会被加入历史状态栈,
  而浏览器地址栏也会变成新的相对 URL. 但是, 浏览器并不会真的向服务器发送请求,
  当状态改变之后查询 `location.href` 也会返回与地址栏中相同的地址. 另外,
  第二个参数为新页面的标题. 第一个参数应该尽可能提供初始化页面状态所需要的各种信息.

  因为 `pushState()` **会创建新的历史状态**, 所以你会发现 "后退" 按钮也能使用了.
  按下 "后退" 按钮, 会触发 window 对象的 `popstate` 事件`(1)`. `popstate`
  事件的 `event` 对象有一个 `state` 属性, 这个属性就包含着上面以第一个参数传递给
  `pushState()` 的状态对象.
    + `(1)`: `popstate` 事件发生后, 事件对象中的状态对象(`event.state`)
      是当前状态.
  ```js
    window.addEventListener('popstate', function(event) {
        var state = event.state;
        // - 当加载的为第一个页面时 state 为空
        if (state) {
            // - 其他操作
            processState(state);
        }
    }, false)
  ```
  得到这个状态对象后, 必须把页面重置为状态对象中的数据表示的状态
  (因为浏览器不会自动为你做这些). 记住, 浏览器加载的第一个页面没有状态, 因此单击
  "后退" 按钮返回浏览器加载的第一个页面时, `event.state` 值为 `null`.
- 要更新当前状态, 可以调用 `replaceState()`, 传入的参数与 `pushState()`
  的前 2 个参数相同. 调用这个方法不会再历史状态中创建新状态, 只会重写当前状态.
  ```js
    history.replaceState({name: 'Greg'}, "Greg's page")
  ```
  在支持 HTML5 历史状态管理的浏览器中, 传递给 `pushState()` 或 `replaceState()`
  的状态对象中不能包含 DOM 元素. 

- **Hint:** 在使用 HTML5 的状态管理机制时, 请确保使用 `pushState()` 创造的每一个
  "假" URL, 在 Web 服务器上都有一个真的, 实际存在的 URL 与之对应. 否则, 单击
  "刷新" 按钮会导致 404 错误.

- **Additional Info**: vue-router 中提供的 history 模式,
  就是使用的此节说讲的历史状态管理; 我们在 "第 13 章 -- 事件" 的
  "`hashchange` 事件" 一节中讲了使用 `#` 号来改变页面的 URL 不会向浏览器发送请求,
  当前的 `history` 模式是和 `#` 号的实现原理是类似的, 但是当前的 history
  模式因为没有 `#` 号, 当用户执行刷新页面之类的操作时, 浏览器还是会给服务器发送请求的.
  所以, 为了避免出现这种情况, 就需要服务器的支持, 需要把所有路由都重定向到跟页面.
  也就是说, 比如当用户在浏览器中直接访问 `http://oursite.com/user/id`
  如果没有后台配置, 那么就会返回 404, 所以如果用户输入的 URL 匹配不到任何静态资源,
  则应该返回同一个 `index.html` 页面, 这个页面就是你 app 依赖的页面.

### 16.5 小结 
