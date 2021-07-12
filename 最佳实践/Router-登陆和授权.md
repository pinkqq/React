- store 管理登陆状态（loggedIn），通过 loggedIn 显示 login/logout 内容。
- 按角色权限配置路由，比如 publicRoutes（公共路由）、privateRoutes（普通用户路由）、adminRoute（管理员路由）。
  路由模版大概如下：
  ```js
  const privateRoute = [
    {
      path: "/backend",
      component: Admin,
      exact: true,
      role: "user",
      backUrl: "/login",
    },
  ];
  ```
- 为权限路由添加一个高阶组件，根据角色权限动态渲染路由。

  ```jsx
  /*
   ** App.js
   */
  import { useState } from "react";

  function App() {
    const [user, setUser] = useState([]);

    function loginAsUser() {
      setUser({ role: ["user"] });
    }

    function loginAsAdmin() {
      serUser({ role: ["user", "admin"] });
    }

    return (
      <Router>
        <AuthRoute key={route.path} {...route} user={user} />
      </Router>
    );
  }
  ```

  ```js
  /*
   ** component/AuthRoute.js
   */

  function authRoute(props) {
    const {
      user: { role: userRole },
      role: routeRole,
      backUrl,
      ...otherProps
    } = props;

    if (userRole && userRole.indexOf(routeRole) > -1) {
      return <Route {...otherProps}>
    } else {
      return <Redirct to={backUrl} />;
    }
  }
  ```
