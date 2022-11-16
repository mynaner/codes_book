## 使用 @ 作为路径根目录

```json
/// tsconfig.json
"compilerOptions": {
  //...
  "baseUrl": ".",
  "paths": {
    "@/*":["src/*"]
  }
},


// vite.config.ts
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";
import { resolve } from "path";
// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      "@": resolve(__dirname, "src"), // 路径别名
    },
    extensions: [".js", ".json", ".ts"], // 使用路径别名时想要省略的后缀名，可以自己 增减
  },
});

```





## vue-devtools安装成功，但是控制台不显示的问题

```ts
/// main.js
const win: Window = window;
if (process.env.NODE_ENV === "development") {
  if ("__VUE_DEVTOOLS_GLOBAL_HOOK__" in win) {
    //@ts-ignore
    win.__VUE_DEVTOOLS_GLOBAL_HOOK__.Vue = app;
  }
}
```



![img](./image/vue3proxy.webp)







# Ant Design of Vue 

### Vite 按需

```ts
// vite.config.js
import Components from 'unplugin-vue-components/vite'
import { AntDesignVueResolver } from 'unplugin-vue-components/resolvers'

export default {
  plugins: [
    /* ... */
    Components({
      resolvers: [
        AntDesignVueResolver(),
      ],
    }),
  ],
};
```



