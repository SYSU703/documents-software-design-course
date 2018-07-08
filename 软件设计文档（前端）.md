# fastchat-fe 设计文档

## 数据获取抽象化
由于各种及时通讯工具在功能上大同小异，因此一个前端是有可能兼容多套即时通讯服务的。fastchat-fe就以“高可移植性”为其中一个目标。为此，fastchat-fe将数据获取的逻辑解耦出来，fastchat-fe不依赖于具体的数据获取逻辑，而是依赖于一个抽象类`ServiceAgent`。移植者只要实现了这个抽象类的所有接口，fastchat-fe就能够正常地运行于这套即时通讯服务之上。
目前fastchat-fe已经兼容了[fastchat-se](https://github.com/csr632/fastchat-se)和[703-se](https://github.com/SYSU703/fastchat-se)两套完全不同的服务端。

## 数据更新
fastchat-fe相比于其他前端项目，其中一个难点在于：其他前端项目的数据更新很大程度上来自于用户交互（比如用户创建了某个资源以后，前端就知道需要刷新资源列表）；而在及时通讯的场景中，有很多数据的更新需要通过服务端才能知道（比如新消息、好友昵称改变、群聊成员改变等等，前端不知道什么时候数据会更新）。
获取数据更新的方式有以下几种：轮询、长连接、server push、web socket。其中，轮询可以配合HTTP缓存`Last-Modified`使用，减小开销。
fastchat-fe不对获取数据更新的方式作出任何假设，移植者可以在实现`ServiceAgent`的时候选择任何方式。不管通过何种方式，只要发现数据需要更新，就调用对应的回调函数（以新的数据为参数），从而fastchat-fe得知新的数据。

## 状态管理和视图更新
以vuex store作为“single source of truth”，component中仅仅保存一些局部的状态（比如按钮的状态、菜单的状态等），其他所有应用程序需要的数据都由vuex来管理。大部分数据传递的路径是`vuex state -> vuex getter -> component props/computed -> component view`。数据驱动视图更新：当需要更新视图时，通过vuex actions改变vuex state，最终导致视图的更新。
