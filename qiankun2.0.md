# qiankun 乾坤2.0 vue项目配置

**主应用配置**

- 目录和文件
- core
- microApp-register.ts
- microApp-config.ts

```
// =======================microApp-config.ts

/**
 * @author hevin
 * @time  
 * @name 无需服务端获取的微应用
 */

const noAuthApps = [
  {
    module: "subapp-login",
    defaultRegister: true,
    devEntry: "//localhost:2753",
    depEntry: "http://login.mfe.wlui.com.cn/",
    routerBase: "/login",
    data: [
      {
        id: "1",
        title: "login",
        icon: "el-icon-monitor",
        children: [
          {
            id: "1-1",
            title: "home",
            url: "/login"
          }
        ]
      }
    ]
  },
]

export default noAuthApps;



  //  ===================microApp-register.ts
  import { registerMicroApps, runAfterFirstMounted, setDefaultMountApp, start, initGlobalState } from "qiankun";
  import store from "../store";

  /**
   * @name 导入想传递给子应用的方法，其他类型的数据皆可按此方式传递
   * @description emit建议主要为提供子应用调用主应用方法的途径
   */
  import emits from "../utils/emit"

  /**
   * @name 导入qiankun应用间通信机制appStore
   */
  import appStore from '../utils/app-store'

/**
 * @name 导入全局常量给子应用
 */
// import GLOBAL from '../global'

/**
 * @name 声明子应用挂载dom，如果不需要做keep-alive，则只需要一个dom即可；
 */
const appContainer = "#subapp-viewport";

/**
 * @name 声明要传递给子应用的信息
 * @param data 主应要传递给子应用的数据类信息
 * @param emits 主应要传递给子应用的方法类信息
 * @param GLOBAL 主应要传递给子应用的全局常量
 * @param utils 主应要传递给子应用的工具类信息（只是一种方案）
 * @param components 主应要传递给子应用的组件类信息（只是一种方案）
 */
let props = {
  data: store.getters,
  emits,
  GLOBAL
}

/**
 * @name 启用qiankun微前端应用
 * @param {Array} list 应用注册表信息
 */
const qianKunStart = (list) => {
  /**
   * @name 处理子应用注册表数据
   */
  let apps = []; // 子应用数组盒子
  let defaultApp = null; // 默认注册应用路由前缀
  let isDev = process.env.NODE_ENV === 'development'; // 根据开发环境|线上环境加载不同entry
  list.forEach(i => {
    apps.push({
      name: i.module,
      entry: isDev ? i.devEntry : i.depEntry,
      container: appContainer,
      activeRule: i.routerBase,
      props: { ...props, routes: i.data, routerBase: i.routerBase }
    })
    if (i.defaultRegister) defaultApp = i.routerBase;
  });

  /**
  * @name 注册子应用
  * @param {Array} list subApps
  */
  registerMicroApps(
    apps,
    {
      beforeLoad: [
        app => {
          console.log('[LifeCycle] before load %c%s', 'color: green;', app.name);
        },
      ],
      beforeMount: [
        app => {
          console.log('[LifeCycle] before mount %c%s', 'color: green;', app.name);
        },
      ],
      afterUnmount: [
        app => {
          console.log('[LifeCycle] after unmount %c%s', 'color: green;', app.name);
        },
      ],
    },
  )

  /**
   * @name 设置默认进入的子应用
   * @param {String} 需要进入的子应用路由前缀
   */
  setDefaultMountApp(defaultApp + '/');

  /**
   * @name 启动微前端
   */
  start();

  /**
   * @name 微前端启动进入第一个子应用后回调函数
   */
  runAfterFirstMounted(() => { });

  /**
 * @name 启动qiankun应用间通信机制
 */
  appStore(initGlobalState);
}

export default qianKunStart;









  // ===================utils/emit.ts

  /**
   * @name 退出登录
   */
  const logout = () => {
    console.log('退出登录')
  }

  export default [
    logout
  ]


  //   ===================utils/app-store

  import store from "@/store";
  /**
   * @name 导入注册并启动微应用函数
   */

  /**
   * @name 启动qiankun应用间通信机制
   * @param {Function} initGlobalState 官方通信函数
   * @description 注意：主应用是从qiankun中导出的initGlobalState方法，
   * @description 注意：子应用是附加在props上的onGlobalStateChange, setGlobalState方法（只用主应用注册了通信才会有）
   */
  const appStore = (initGlobalState) => {
    /**
     * @name 初始化数据内容
     */
    const { onGlobalStateChange, setGlobalState } = initGlobalState({
      msg: '',
      token: '',
      appsRefresh: false,
    });

    /**
     * @name 监听数据变动
     * @param {Function} 监听到数据发生改变后的回调函数
     * @des 将监听到的数据存入vuex
     */
    onGlobalStateChange((value, prev) => {
      console.log('[onGlobalStateChange - master]:', value, prev);
      'msg' in value && store.dispatch('appstore/setMsg', value.msg);
      value.token && store.dispatch('app/setToken', value.token);
      value.appsRefresh && window?.location?.reload?.();
    });

    /**
     * @name 改变数据并向所有应用广播
     */
    setGlobalState({
      ignore: 'master',
      msg: '来自master动态设定的消息',
    });
  }

  export default appStore;



```
