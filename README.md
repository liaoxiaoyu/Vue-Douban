# 微豆 Vdo

一个使用 Vue.js重构 [豆瓣](http://www.douban.com) 的项目。  


![GIF 图片](/static/md/example.gif)


# 快速使用  

# 安装依赖
npm install

# 在 localhost:8080 启动项目
npm run dev
```

# 教程  


## 安装 vue-cli 脚手架  


运行如下命令，即可创建一个名为 my-project 的 vue 项目，并且通过本地 8080 端口启动服务   
    
``` bash
npm install -g vue-cli
vue init webpack my-project
cd my-project
npm install
npm run dev
```


安装完成后，即可看到以下文件结构：

```
.
|-- build                            // 项目构建相关代码
|   |-- build.js                     // 生产环境构建代码
|   |-- check-version.js             // 检查 node、npm 等版本
|   |-- dev-client.js                // 热重载相关
|   |-- dev-server.js                // 构建本地服务器
|   |-- utils.js                     // 构建工具相关
|   |-- webpack.base.conf.js         // webpack 基础配置（出入口和 loader）
|   |-- webpack.dev.conf.js          // webpack 开发环境配置
|   |-- webpack.prod.conf.js         // webpack 生产环境配置
|-- config                           // 项目开发环境配置
|   |-- dev.env.js                   // 开发环境变量
|   |-- index.js                     // 项目一些配置变量（开发环境接口转发将在此配置）
|   |-- prod.env.js                  // 生产环境变量
|   |-- test.env.js                  // 测试环境变量
|-- src                              // 源码目录
|   |-- components                   // vue 公共组件
|   |-- store                        // vuex 的状态管理
|   |-- App.vue                      // 页面入口文件
|   |-- main.js                      // 程序入口文件，加载各种公共组件
|-- static                           // 静态文件，比如一些图片，json数据等
|-- test                             // 自动化测试相关文件
|-- .babelrc                         // ES6语法编译配置
|-- .editorconfig                    // 定义代码格式
|-- .eslintignore                    // ESLint 检查忽略的文件
|-- .eslistrc.js                     // ESLint 文件，如需更改规则则在此文件添加
|-- .gitignore                       // git 上传需要忽略的文件
|-- README.md                        // 项目说明
|-- index.html                       // 入口页面
|-- package.json                     // 项目基本信息
.
```


## 静态页面开发  

本项目使用了基于 Vue 2.0 和 Material Desigin 的 UI 组件库 [Muse-UI](https://museui.github.io/)。


## vue-router 2 使用

当一个个静态组件完成后，需要按照路由组织这些组件文件。  

请前往 [vue-router 2 介绍](https://router.vuejs.org/zh-cn/) 阅读 __基础__ 部分教程，并可以边阅读边配置路由。  

路由文件是 `./src/router.index.js` 。  

本项目中使用了 [HTML5 History 模式](https://router.vuejs.org/zh-cn/essentials/history-mode.html)，路由配置比较简单，可以参考。  


## API 请求转发配置

至此，你应该已经完成了所有的静态页面的工作，接下来我们准备搭建请求，为后面的 xhr 请求做好准备。  

1. 打开 `http://api.douban.com/v2/movie/in_theaters` 查看接口数据，留意此地址。  

2. 在 `./config/index.js` 中的 `proxyTable` 配置代理：  

    ```js 
    proxyTable: {
        '/api': {
            target: 'http://api.douban.com/v2',
            changeOrigin: true,
            pathRewrite: {
                '^/api': ''
            }
        }
    }
    ```  

3. 重新启动 `npm run dev`，打开 `localhost:8080/api/movie/in_theaters` 查看结果是否与直接请求豆瓣 API 相同。  

4. 本应该使用了以下 API：  
    - `/v2/movie/search?q={text}` 电影搜索api；  
    - `/v2/movie/in_theaters` 正在上映的电影；  
    - `/v2/movie/coming_soon` 即将上映的电影；  
    - `/v2/movie/subject/:id` 单个电影条目信息。  

> 更多请参考 [豆瓣电影 API](https://developers.douban.com/wiki) 文档。    

这样我们就可以在应用中调用 `/api/movie/in_theaters` 来访问 `http://api.douban.com/v2/movie/in_theaters`，从而解决跨域的问题。

## 使用 axios 

axios 库使用起来相当简单。  

你可以在单个组件中尝试引入并调用：  
```javascript
import axios from 'axios';
axios.get('/v2/movie/in_theaters', { 'city': '广州' })
    .then((result) => {
        console.log(result);
    });
```
这里，可以用返回的 `result` 去更新 `data(){ }` 中 `return` 的数据。  
> 更多 axios 用法请参考 [文档](https://github.com/mzabriskie/axios#example)

## 使用 Vuex 并分离代码  

为了试代码更加结构化，我们应当将数据请求和视图分离。  

这一节中，我们有两个任务要做：  

1. 分离数据请求层逻辑。  
2. 使用 Vuex 管理状态。   

将二者放到同一节中主要是因为二者再同一目录下，我们来查看 `./store` 文件夹的结构：
```
.
|-- store                          // 数据处理根目录
|   |-- movies                     // 单个电影模块文件夹
|   |   |-- api.js                 // 电影模块对外开放的接口
|   |   |-- module.js              // Vuex 模块
|   |   |-- type.js                // Vuex 操作 key
|   |-- base.js                    // 基础方法
|   |-- store.js                   // Vuex 入口
.
```
针对第一个任务：
- `base.js` 存放封装的基础请求函数  
- `**/api.js` 存放该模块下公开的请求函数  

针对第二个任务，我们需要先了解 Vuex。  

请查看 [Vuex 文档](https://vuex.vuejs.org/zh-cn/)，了解其 **核心概念**。  

>Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。Vuex 也集成到 Vue 的官方调试工具 devtools extension，提供了诸如零配置的 time-travel 调试、状态快照导入导出等高级调试功能。  

其实在我看来，Vuex 相当于某种意义上设置了读写权限的全局变量，将数据保存保存到该“全局变量”下，并通过一定的方法去读写数据。（希望这能帮助你理解 Vuex）

为了方便模块化管理：  
-  `store.js` 作为入口文件，去挂载各个模块；  
- `/movies/`文件夹下为电影相关的模块；  
- `/movies/moudule.js` 为电影模块的主要 Vuex 文件；  
- `/movies/type.js` 为[使用常量替代 Mutation 事件类型](https://vuex.vuejs.org/zh-cn/mutations.html)的实现。  

到此便完成了所有开发上的基础问题。

## 发布

1. 运行 `npm run build`，即可在生成的 `/dist` 文件夹下看到所有文件。  
2. 将文件复制到你的服务器上某个目录（我的是`/var/www/Vdo/dist`），按照下一节配置 Nginx 即可

提示：可以使用 `scp` 命令将本地文件拷贝至服务器，例如 `scp -P 20 -r dist user@host:/target/location`



欢迎 Star 本项目。


# 感谢&参考
- https://github.com/superman66/vue2.x-douban
- http://blog.guowenfh.com/2016/03/24/vue-webpack-01-base/
- https://github.com/mzabriskie/axios
- https://museui.github.io/
- https://vuejs-templates.github.io/webpack/
