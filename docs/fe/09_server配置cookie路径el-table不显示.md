# 前端 server配置,cookie路径,el-table不显示



### 1. el-table空白不显示问题

在项目中使用了element-UI,使用vite构建时发现无法显示el-table组件,但是使用webpack构建可以显示

[解决办法参考GitHub](https://github.com/ElemeFE/element/issues/21968#issuecomment-1537071209)

```js
// vite.config.js
resolve: {
        extensions: ['.js', '.vue', '.jsx', 'tsx', '.json'],
        alias: {
            vue: 'vue/dist/vue.esm.js',
        },
    },
```

在resolve的alias配置中,把vue指定`vue/dist/vue.esm.js`即可



> cookie 中的 domain和path
>
> 在HTTP Cookie中，"domain"和"path"是两个用来限制Cookie可见性的属性。 
>
> 1. Domain（域）：该属性规定哪些域名可以访问Cookie。
>
>     - 只有包含在Cookie的domain属性中的域名或其子域名才能访问该Cookie。
>     - 例如，如果domain属性设置为".example.com"，则所有以"example.com"结尾的域名以及其子域名都可以访问该Cookie。 
>
> 2. Path（路径）：该属性指定了路径限制，定义了哪些路径下的页面可以访问Cookie。
>
>     - 只有与Cookie的path属性匹配的页面才能访问该Cookie。
>     - 例如，如果path属性设置为"/admin"，则只有以"/admin"开头的页面才能访问该Cookie。 
>
>     
>
>     这两个属性的组合可用于更精确地控制Cookie的可见性和访问权限。

### 2. 本地起后端服务,无法连接和登录后提示未登录问题

有2种方式,webpack和vite

#### 第1种:vite构建,vite.config.js

```JS
server: {
	proxy: {
		'/gateway': {
			target: 'http://localhost:8089', // 本地服务的ip和端口 可以是127.0.0.1 | localhost | 内网ip地址
			changeOrigin: true, // 自动修改http header里面的host
            ws: false, // // 如果是https接口，需要配置这个参数
            secure: false, //是否使用HTTPS协议。默认，false。
            cookiePathRewrite: '/', // 本地服务统一cookie的path为'/' (解决因path不一致导致登录成功后提示用户未登录问题!)
            rewrite: (path) => path.replace(/^\/gateway/, '') // 本地服务没有gateway,替换为空
		}
	}
}
```

因为本地起服务没有 `/gateway`所以,路径需要重写`rewrite: (path) => path.replace(/^\/gateway/, '')`

另外,cookie设置的path不统一,登录后提示未登录.

因为登录接口响应的cookie的path是/admin,到首页会请求另外一个接口,直接提示"用户未登录"

解决办法是把cookie的path直接指向根路径 `cookiePathRewrite: '/'`

参考链接:

[https://cn.vitejs.dev/config/server-options.html#server-proxy](https://cn.vitejs.dev/config/server-options.html#server-proxy)

[https://github.com/http-party/node-http-proxy#options](https://github.com/http-party/node-http-proxy#options)

- **cookiePathRewrite**: rewrites path of `set-cookie` headers. Possible values:

    - `false` (default): disable cookie rewriting

    - String: new path, for example `cookiePathRewrite: "/newPath/"`. To remove the path, use `cookiePathRewrite: ""`. To set path to root use `cookiePathRewrite: "/"`.

    - Object: mapping of paths to new paths, use "*"to match all paths. For example, to keep one path unchanged, rewrite one path and remove other paths:

        ```JS
        cookiePathRewrite: {
          "/unchanged.path/": "/unchanged.path/",
          "/old.path/": "/new.path/",
          "*": ""
        }
        ```

    

### 第2种:webpack构建,vue.config.js

```JS
	proxy: {
		'/gateway': {
			target: 'http://10.20.100.65:8089',
			pathRewrite: (path) => path.replace(/^\/gateway/, ''),
			cookiePathRewrite: '/'
		}
	}
```
参考链接

[https://cli.vuejs.org/zh/config/#devserver-proxy](https://cli.vuejs.org/zh/config/#devserver-proxy)

[https://github.com/chimurai/http-proxy-middleware/tree/v2.0.4#readme](https://github.com/chimurai/http-proxy-middleware/tree/v2.0.4#readme)

找到`pathRewrite`和 `cookiePathRewrite`关键字



