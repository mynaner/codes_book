# umijs

https://umijs.org/zh-CN/docs/getting-started



### umi-requset 封装

```js
/**
 * request 网络请求工具
 * 更详细的 api 文档: https://github.com/umijs/umi-request
 */
import { extend } from 'umi-request';

import { notification, message } from 'antd';

const codeMessage = {
  200: '服务器成功返回请求的数据。',
  201: '新建或修改数据成功。',
  202: '一个请求已经进入后台排队（异步任务）。',
  204: '删除数据成功。',
  400: '发出的请求有错误，服务器没有进行新建或修改数据的操作。',
  401: '用户没有权限（令牌、用户名、密码错误）。',
  403: '用户得到授权，但是访问是被禁止的。',
  404: '发出的请求针对的是不存在的记录，服务器没有进行操作。',
  406: '请求的格式不可得。',
  410: '请求的资源被永久删除，且不会再得到的。',
  422: '当创建一个对象时，发生一个验证错误。',
  500: '服务器发生错误，请检查服务器。',
  502: '网关错误。',
  503: '服务不可用，服务器暂时过载或维护。',
  504: '网关超时。',
};

/**
 * 异常处理程序
 */
const errorHandler = error => {
  const { response } = error;
  if (response && response.status) {
    const errorText = codeMessage[response.status] || response.statusText;
    const { status, url } = response;
    message.error(errorText);
  }

  return { success: false };
};
const request = extend({
  errorHandler,
  // 默认错误处理
  // credentials: 'omit', // 默认请求是否带上cookie
});

// request拦截器, 改变url 或 options.
request.interceptors.request.use(async (url, options) => {
  let Authorization = localStorage.getItem('token');
  const headers = {
    'Content-Type': 'application/json',
    Accept: 'application/json',
    Authorization: Authorization,
  };
  return {
    url: 'https://testapi.yizhiting.net/api-admin' + url,
    options: { ...options, headers: headers },
  };
});

// response拦截器, 处理response
request.interceptors.response.use(async (response, options) => { 
  if (response && response.status == 200) {
    const data = await response.clone().json();
    !data.success?message.error(data.msg):null; 
  }
  return response;
});

export default request;
```



### dva model

```js

import * as services from "../services" // umi-request 
export default {
  namespace: 'login', // 默认是文件名
  state: {
    // 存 登陆后 返回的用户信息
    userInfo:null,
    // 暂存  token 需要放入 locastorage 中
    menus:[],
    // 城市地图坐标
    map:null
  }, 
  effects: {
    /**
     *  
     * call 在内部调用其他方法,一般用于数据请求services的方法
     * put 出发reducers的方法，改变内部状态数据
     * select 拿去内部状态数据
     */
    *login({ payload:{username,password}}, { call, put, select }) {
      const res = yield call(services.login,{username,password})
      if(res.success){
        const {token,avatar,city,createdAt,latitude,longitude,menus,name,phone,roles,username} = res.result; 
        
        localStorage.setItem("token", token); 
        
        yield put({type:"saveUserInfo",payload:{userInfo: {avatar,name,phone,roles,username},menus,map:{latitude,longitude}}})
        // yield select(state=>{
        //   console.log('====================================');
        //   console.log(state);
        //   console.log('====================================');
        // })
        return res.result;
      }
    },
  },
  reducers: {
    saveUserInfo(state, {payload:{userInfo,menus,map}}) {
      return {
        ...state,
        userInfo,
        menus,
        map
      };
    },
    // 启用 immer 之后
    // save(state, action) {
    //   state.name = action.payload;
    // },
  },
  subscriptions: {
    // setup({ dispatch, history }) {
      /**
       * 一般用于 监听到指定页面。然后立即执行 effects 里到方法
       */
      // return history.listen(({ pathname }) => {
      //   if (pathname === '/login') {
      //     dispatch({
      //       type: 'login',
      //       payload:{
      //         username:"admin",
      //         password:"000000"
      //       }
      //     })
      //   }
      // });
    // }
  }
};
```





