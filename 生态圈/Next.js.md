说起 服务端渲染(ssr) 和 SPA，相信你总能说个一二：

- **与 服务端渲染(ssr) 相比**，SPA 这样的架构设计可以让前后端开发并行进行，职责清晰。
- **与 SPA 相比**，服务端渲染(ssr) 首屏等待更短
- **与 SPA 相比**，服务端渲染(ssr) 是服务端直接返回完整的 html 内容，有利于网站的 SEO

这两者的优缺点都很明显，所以随着时代的进步和技术的演变，诞生了 **SSR + SPA** 的新模式。并且在 `react/vue` 等前端框架相结合 `node (ssr)` 下，可以最大限度的重用代码（同构），减少开发维护成本。

## 什么是同构应用

![未命名文件 (5).jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5c1159863c52442189223b8647892940~tplv-k3u1fbpfcp-watermark.image)

web 应用可以分为 **浏览器端** 和 **服务器端** 两部分，对于同构应用来说，分为 **初次发送请求（ SSR ）** 和 **后续操作（ SPA ）**。

- **初次发送请求** 时，服务器端负责渲染页面的全部内容，然后返回给浏览器端，浏览器只需要直接显示返回的内容，所以首屏渲染的速度会非常快；
- 浏览器端的 **后续操作** 则不再需要服务器端的介入，也不会引起页面的刷新，所有的 UI 操作在前端完成。

基于 **React** 可以在服务端渲染的能力，**Next** 提供了一套完整且方便的同构应用解决方案。

## 开始 Next.js

**Next.js**，这是一个 React 的同构应用开发框架。

- 直观的、 [基于页面](https://www.nextjs.cn/docs/basic-features/pages) 的路由系统（并支持 [动态路由](https://www.nextjs.cn/docs/routing/dynamic-routes)）
- [预渲染](https://www.nextjs.cn/docs/basic-features/pages#pre-rendering)。支持在页面级的 [静态生成 (SSG)](https://www.nextjs.cn/docs/basic-features/pages#static-generation-recommended) 和 [服务器端渲染 (SSR)](https://www.nextjs.cn/docs/basic-features/pages#server-side-rendering)
- 自动代码拆分，提升页面加载速度
- 具有经过优化的预取功能的 [客户端路由](https://www.nextjs.cn/docs/routing/introduction#linking-between-pages)
- [内置 CSS](https://www.nextjs.cn/docs/basic-features/built-in-css-support) 和 [Sass 的支持](https://www.nextjs.cn/docs/basic-features/built-in-css-support#sass-support)，并支持任何 [CSS-in-JS](https://www.nextjs.cn/docs/basic-features/built-in-css-support#css-in-js) 库
- 开发环境支持 [快速刷新](https://www.nextjs.cn/docs/basic-features/fast-refresh)
- 利用 Serverless Functions 及 [API 路由](https://www.nextjs.cn/docs/api-routes/introduction) 构建 API 功能
- 完全可扩展

## 创建一个 Next.js 应用

我们用 CLI 工具 -- [Create Next App](https://www.npmjs.com/package/create-next-app) 来快速创建 Next.js 应用，通过配置参数 `-e, --example [name]|[github-url]` 可以直接以 [Next.js Repo](https://github.com/vercel/next.js/tree/master/examples)中的模版创建应用：

```
npx create-next-app nextjs-auth0 --example "https://github.com/vercel/next.js/tree/master/examples/auth0"
```

我们今天使用 Next.js 的[入门模版](https://github.com/vercel/next-learn-starter/tree/master/learn-starter)（默认）：

```
npx create-next-app my-nextjs
```

运行成功后，进入 `my-nextjs` 目录，运行启动命令，应用默认运行在 3000 端口：

```
cd my-nextjs
npm run dev
```

http://localhost:3000/， 查看页面源代码，可以发现所有的内容都是服务器端返回的：
![next-source.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f1590039b2244a4a79f5ce6fed53b17~tplv-k3u1fbpfcp-watermark.image)

## Pages 页面

**Next.js** 在开发时遵循了一定的约定：

- 所有的页面必须放在 `pages` 目录下，页面作为一个组件。
- 页面与基于其文件名的路由相关联。
  - `pages/index.js` 关联的路由是 `/`
  - `pages/posts/first-post.js` 关联的路由是 `/posts/first-post`
- `page` 具有静态方法 `getInitialProps`，可以获取接口数据作为组件的 `props`。

### 创建页面

`page/index.js` 已经存在，我们新建一个 `pages/posts/first-post.js` :

```js
export default function FirstPost() {
  return <h1>First Post</h1>;
}
```

这时候，你访问 `http://localhost:3000/posts/first-post`， 应该可以看到：
![](https://www.nextjs.cn/static/images/learn/navigate-between-pages/first-post.png)

### 页面间的导航

`<Link>` 组件可以在客户端实现页面间的导航，和 React Router 一样实现的是前端路由，不会引起页面的刷新：

```js
import Link from "next/link";
```

**使用时，将 Link 包裹在 a 标签外**，比如在 `posts/first-post` 添加一个回首页的链接：

```js
import Link from "next/link";

export default function FirstPost() {
  return (
    <>
      <h1>First Post</h1>
      <h2>
        <Link href="/">
          <a>Back to home</a>
        </Link>
      </h2>
    </>
  );
}
```

**`Link` 组件默认在后台预取目标页面。** 通过设置 `prefetch={false}` （默认为 `true` ） 可以禁止页面预取。当 `prefetch` 被设置为 `false` 时，鼠标悬停在 <Link /> 上时仍然会触发预取。预取功能只在生产环境中开启。

```jsx
<Link href="/posts/first-post" prefetch={false}>
  <FirstPost />
</Link>
```

`Link` 组件的默认行为是将新 `URL` 推送到 `history` 堆栈中，**可以使用 `replace` 来替换 URL**，而不是新增，这样发生跳转后，将无法后退到上一个页面。

```jsx
<Link href="/about" replace>
  <a>About us</a>
</Link>
```

**Next.js 支持具有动态路由的 pages（页面）**。例如，如果你创建了一个命名为 `pages/posts/[id].js` 的文件，那么就可以通过 `posts/1`、`posts/2` 等类似的路径进行访问。

## 静态文件服务

如果你添加了一张图片到 `public/me.png` 路径，通过 `/me.png` 可以直接访问到它：

```jsx
import Image from "next/image";

function Avatar() {
  return <Image src="/me.png" alt="me" width="64" height="64" />;
}

export default Avatar;
```

**`public` 目录映射静态文件**，包括 `robots.txt`、`favicon.ico`、Google 站点验证文件以及任何其它静态文件（包括 `.html` 文件）。

> **注意:**
>
> - 请勿为 public 改名。此名称是写死的，不能修改，并且只有此目录能过够存放静态资源并对外提供访问。
> - 请确保静态文件中没有与 `pages/` 目录下的文件重名的，否则这将导致错误。

## 动态加载页面（Lazy loading）

动态加载（Lazy loading）是一种仅在资源需要时加载它们的策略。Next.js 也提供了 `import dynamic from "next/dynamic"` 帮助我们实现动态加载策略。

普通 import 一个组件：

```jsx
import Link from "next/link";
import Hello from "../../component/Hello";

export default function FirstPost() {
  return (
    <>
      <h1>First Post</h1>
      <Hello />
      <h2>
        <Link href="/">
          <a>Back to home</a>
        </Link>
      </h2>
    </>
  );
}
```

动态加载一个组件：

```jsx
import Link from "next/link";
import dynamic from "next/dynamic";
// import Hello from "../../component/Hello";

const DynamicComponent = dynamic(() => import("../../component/Hello"));

export default function FirstPost() {
  return (
    <>
      <h1>First Post</h1>
      <DynamicComponent />
      <h2>
        <Link href="/">
          <a>Back to home</a>
        </Link>
      </h2>
    </>
  );
}
```

从 network 可以看出，动态加载的情况下，Hello 组件被作为了单独资源：
![未命名文件 (6).jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9275607d53aa480493a7372f9b33bf0b~tplv-k3u1fbpfcp-watermark.image)

## 参考资料

- Next.js：https://nextjs.org/
- Next.js 中文：https://www.nextjs.cn/
