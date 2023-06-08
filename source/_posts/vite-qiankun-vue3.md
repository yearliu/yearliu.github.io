---
title: Vite + Qiankun + vue3 搭建微前端平台 
---
该篇为个人进行微前端框架搭建的一些心得笔记，希望有参考价值
个人使用的平台为vite + qiankun + vue3 平台完成搭建，搭建过程中发现网络上对于vite进行搭建的案例较少，所以发出来作为一个学习参考

>vite ^4.3.5
qiankun ^2.10.8

## 为什么是微前端

简要来说，微前端是前端借照微程序的设计思维，设计出来的一种微框架设计
他设计的目标为降应用拆分为多个子应用，可以将子应用分别进行部署，参与过运维与发布的同学应该对于前端应用的发布流程非常熟悉
前端发布通常通过脚本跑编译指令，将开发代码打包为可发布代码，然后通过整包替换将整个前端程序发布到服务器上，完成前端项目部署
但是在此过程中，是需要将整个程序的代码全部进行替换一次的，那么在复杂系统中，进行一次全部替换的成本是比较高的
首先是代码管理成本：复杂系统经过长时间的运营后，内部早已分化为不同的功能模块，以及不同的设计思路代码，发布时往往是要经过代码分支迁出，比对，合并，打包部署然后再进行发布，可能改动一个小点会导致非常多的代码产生变更
系统维护成本：运行多年的程序会导致开发人员不敢轻易的变更框架依赖的版本，但是复杂系统往往会采用很多不同的依赖，依赖之间也会有对应的使用关系，在此情况下新增或者删除某个依赖可能会导致整体系统的崩盘，往往是要小心再小心的操作，更大的可能性是系统再也不进行升级；保持原有依赖版本直到整体系统已经落后于时代再进行整体的重构工作；并且如果框架搭建初期就没有使用一些工具去固定依赖版本，发布过程中往往会导致依赖版本无意识变更，产生不可预料的错误；
发布成本：传统项目发布需要将全系统整体打包发布，发布脚本往往设计是重新拉取所有依赖，重新打包发布，在此过程中需要将所有的依赖都重新拉取一次，效率较低
测试成本：原因同上，因为一些功能上的变更或者依赖版本的变更，小功能改动有可能会导致全系统重测，这个过程投入的测试成本是相当客观的；

那么在以上痛点下，前端提出了一套微前端的概念，最初期的微前端是采用iframe嵌套实现的，后期慢慢出现了更多的框架层面的微前端解决方案，实现了业务之间的隔离，可以将代码进行分开发布；实现了功能的隔离，可以将公司内的各个项目集成在一套微前端框架内进行访问，并且微前端子系统可以使用不同的框架进行开发，意味着公司各个年代的系统功能可以更加容易的进行业务整合，而不需要做太多的变更

以上为微前端的产生契机以及设计的优势

## 微前端组成 
1. 微前端主进程： 用于挂载子应用，在主进程中应该进行子应用挂载配置，应用外壳搭建，以及相应路由配置
2. 微前端子应用： 用于实现具体的业务模块，模块之间应该相互隔离，保持黑箱状态

## 搭建 
这里我使用vue的脚手架进行初始化 npm create vue@3 
后续步骤按照具体需求自行选配，但是这边还是建议最好是使用TypeScript进行开发，因为TypeScript的各方面都优于JavaScript，也一定是未来的趋势；并且微前端的设计初衷就是用于复杂框架搭建，TypeScript的特性来说更适合于这种大型程序的构建
这边初始化的主进程我使用qiankun-main 命名 子应用使用 qiankun-helloWord 命名
> 主进程 ： qiankun-main
子应用 ： qiankun-helloWorld

### 主进程
需要引入的依赖：
> npm install -s qiankun 

修改配置：
/src/main.ts

``` typescript 
import './assets/main.css'

import { createApp } from 'vue'
import App from './App.vue'
import appConfig from './app.config.json'
import router from './router'
import { type RegistrableApp, type ObjectType } from 'qiankun'
import { registerMicroApps, start } from 'qiankun'
const app = createApp(App)

app.use(router)

app.mount('#app')

const appList: RegistrableApp<ObjectType>[] = appConfig;
registerMicroApps(appList, {
  beforeLoad: [
    //@ts-ignore
    (event: any) => {
      console.log('before load ---- ', event)
      // debugger
    }
  ],
  beforeMount: [
    //@ts-ignore
    (event: any) => {
      console.log('before beforeMount ---- ', event)
    }
  ],
  beforeUnmount: [
    //@ts-ignore
    (event: any) => {
      console.log('before beforeUnmount ---- ', event)
    }
  ],
})

// 如果需要进行拦截，可以在此处导航守卫设置，如果不需要，可以省略
const childrenPath = ['/helloworld']

router.beforeEach((to, from, next) => {
  // 如果有 name 属性，表示是主应用
  if (to.name) {
    next()
  }
  // 如果是子应用
  if (childrenPath.some((item) => to.path.includes(item))) {
    console.log(to, from, 'childpath')
    next();
  }
  // 如果没有当路由
  else { }
})

// 启动 qiankin
start()

```
/vite.config.ts
``` typescript 
import { fileURLToPath, URL } from 'node:url'

import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({

  plugins: [vue()],
  server: {
    headers: {
      'Access-Control-Allow-Origin': '*',	// 设置允许跨域请求，否则会因为在其他端口号获取资源报错 
    },
    port: 8080,	// 设置每次打开本地的端口号
    open: true,	// 每次 serve 完成自动打开
  },
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url))
    }
  },
})

```
/src/app.config.json
``` javascript
[
    {
        "name": "qiankun-helloworld", //子应用名称
        "entry": "http://localhost:5173", //访问端口路径
        "container": "#helloworld-box", // 挂载容器名称 需要在主进程中创建对应元素
        "activeRule": "/helloworld" //命中路由，如果路由命中则会加载子应用
    }
]

```




### 子应用 
需要引入的依赖：
> npm install -s qiankun 
npm install -s vite-plugin-qiankun

/src/main.ts
``` typescript 
import './public-path'
import './assets/main.css'

import { createApp } from 'vue'
import AppVue from './App.vue'
import routes from './router'
import { renderWithQiankun, qiankunWindow, type QiankunProps } from 'vite-plugin-qiankun/dist/helper';
import { createRouter, createWebHistory } from 'vue-router'
import type { App } from '@vue/runtime-core'
// const app = createApp(AppVue)
let router = null;
let instance: App<Element> | null = null;
interface RenderParams {
    container?: any | undefined
}

const render = ({ container }: RenderParams): void => {

    router = createRouter({
        routes,
        //@ts-ignore
        history: createWebHistory(qiankunWindow.__POWERED_BY_QIANKUN__ ? '/helloworld' : '/'),
        //@ts-ignore
        base: qiankunWindow.__POWERED_BY_QIANKUN__ ? '/helloworld' : '/'
    })
    console.log(router,'rputere')
    instance = createApp(AppVue)
    instance.use(router).mount(container ? container.querySelector("#app") : "#app") // 根元素名称 要跟index.html内id相同
}

// some code
renderWithQiankun({
    mount(props) {
        console.log('mount')
        render(props)
    },
    bootstrap() {
        console.log('bootstrap')
    },
    unmount(props: any) {
        console.log('unmount', props)
        const { container } = props
        instance?.unmount()
        if (instance?._container) {
            instance._container.innerHTML = ''
        }
        instance = null
        router = null
    },
    update: function (props: QiankunProps): void | Promise<void> {
        throw new Error('Function not implemented.')
    }
});

if (!qiankunWindow.__POWERED_BY_QIANKUN__) {
    render({});
}
```

/src/public-path.ts
``` typescript 
//@ts-ignore
if (window.__POWERED_BY_QIANKUN__) {
    //@ts-ignore
    __webpack_public_path__ = window.__INJECTED_PUBLIC_PATH_BY_QIANKUN__;
}
```

/src/router/index.ts

``` typescript 
    return RouteRecordRaw[] 
```

/vite.config.ts
``` typescript 
import { fileURLToPath, URL } from 'node:url'

import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import path from 'node:path'
const { name } = require('./package.json')
import qiankun from 'vite-plugin-qiankun'
const useDevMode = true;
// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    vue(),
    qiankun(`${name}`, {
      useDevMode
    })
  ],
  base: './',
  server: {
    headers: {
      'Access-Control-Allow-Origin': '*',	// 设置允许跨域请求，否则会因为在其他端口号获取资源报错 
    },
    port: 5173,	// 设置每次打开本地的端口号
  },
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url))
    }
  },

})

```

## 开发环境
以上配置可以直接在开发环境运行

## 发布应用
注意子引用的挂载地址要更改为实际地址 /* app.config.json */ entry 参数，发布要改为实际生产配置