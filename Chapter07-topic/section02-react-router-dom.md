# 客户端路由

> 在SPA（单页面应用）兴起之前，「路由（Route）」概念多指的是后端服务器路由（也叫，静态路由），用于控制请求到具体处理的endpoint，比如返回一个html文件。而客户端路由（对应也称呼，动态路由），简而言之，是控制单页面应用内的页面跳转（在不刷新页面情况下）。本文以React库最佳客户端路由搭配[react-router](https://reactrouter.com/)为例作一些介绍，vue框架参考：https://router.vuejs.org

[toc]

## 1. 概念

react-router 由Ryan Florence等人开发，是第三方库，并非React团队开发。 使用前，先明确3个不同的包

- `react-router`（core），包含了react-router-dom、react-router-native共通的组件。**基本上不需要直接安装**

- `react-router-dom`，用于 web开发（以下不明确说明，均指[react-router-dom](https://reactrouter.com/web/guides/quick-start) v5版本）
- `react-router-native`，用于 react-native 开发



先不忙安装，了解下官网这篇文章：[The Future of React Router and @reach/router](https://reacttraining.com/blog/reach-react-router-future/)。文章写于**19年**，指出了未来react-router方向。（到21年，react-router v5、 @reach/router 1.0均已发布）。 细节不多说，总结：

- React Router 以及新包Reach Router（@reach/router）将一如既往完善基于hook的接口
- React Router 将只是一个存续的项目。
- React Router v5 100%向后兼容；@reach/router 1.0 bux修复（当时还没发布）
- 如何做出选择？
  - 如果现有项目路由不多、想尝鲜，可以使用 @reach/router
  - 如果现有项目路由很多，特别频繁使用`Route` 和`Switch`，可以使用Reach Router v5基于hook的接口，逐步升级。

## 2. 基础组件

基础组件3大类

- 路由入口（routers）类，如
  - `<BrowserRouter>`
  - `<HashRouter>`
- 路由匹配（route matchers）类，如
  - `<Route>`
  - `<Switch>`
- 路由导航（navigation）类，如
  - `<Link>`
  - `<NavLink>`
  - `<Redirect>`



### 2.1 [Routers](https://reactrouter.com/web/guides/primary-components/routers)

任何需要使用react-router的web项目，都需要用Router包裹 `<App />`。可以这么理解，Router组件相当于React概念中「HOC」组件。`<BrowserRouter>` 与 `<HashRouter>`区别：

- `<BrowserRouter>` 使用通常的url路径。但需要应用服务器配合，即客户端访问任何路径下页面，服务端均需要返回同一个静态文件。具体说明，可参考 create-react-app官网说明：[Serving Apps with Client-Side Routing](https://create-react-app.dev/docs/deployment/#serving-apps-with-client-side-routing)。举个栗子：

  - 当你访问home页，浏览器地址栏显示：http://myapp.com，发起http请求：`GET /`

  - 当你跳转到登录页，浏览器地址栏显示：http://myapp.com/login，发起http请求：`GET /login`

  - 当你跳转到看板页，浏览器地址栏显示：http://myapp.com/dashboard，发起http请求：`GET /dashboard`

  - 好处在于浏览器显示的地址栏显而易见；短处，需要配置服务器 `GET /`、`GET /login`、`GET /dashboard`均返回同一个文件，以 express服务为例，

    ```js
    var express = require("express");
    var path = require("path");
    var app = express();
    
    app.use(express.static(path.join(__dirname, "build")));
    app.get("/*", (req, res) => {
      res.sendFile(path.join(__dirname, "build", "index.html"));
    });
    app.listen(25798, () => {
      console.log("Launch app on port 25798");
    });
    ```

- `<HashRouter>` 将当前路径作为hash部分拼在url中，以"#"分割。服务器无需特殊配置，但利用了http请求协议对带“#”之后路径（[fragment identifier](https://blogs.msdn.microsoft.com/ieinternals/2011/05/16/url-fragments-and-redirects/)）特殊处理。举个例子：

  - 当你访问home页，浏览器地址栏将显示：http://myapp.com，发起http请求：`GET /`
  - 当你跳转到登录页，浏览器地址栏显示：http://myapp.com/#/login，且忽略#及之后的路径，发起http请求：`GET /`，拿到静态文件后，react-router配合浏览器跳转到登录页
  - 当你跳转到看板页，浏览器地址栏显示：http://myapp.com/#/dashboard，且忽略#及之后的路径，发起http请求：`GET /`，拿到静态文件后，react-router配合浏览器跳转到看板页



### 2.2 [Route Matchers](https://reactrouter.com/web/guides/primary-components/route-matchers)

- `<Switch>` 当该组件被渲染时，会从children（一系列`<Route>`）中遍历，匹配最适合当前url的组件，渲染该`<Route>`并忽略剩余其他子组件。如果没有匹配任何一个，则渲染 null。建议在`<Switch>`最后一个`<Route>`设置为重定向404页面。

  ```jsx
  const App = () => {
    return (
      <Switch>
        <Route path="/about" component={About} />
        <Route path="/contact/:id" component={Contact} />
        <Route path="/contact" component={AllContacts} />
        <Route path="/" component={Home} />
        <Route path="/404" component={Page404} />
        <Redirect to="/404" />
      </Switch>
    );
  }
  ```
  - ⚠️：`<Switch>` 子组件遍历是按次序的；路径的匹配过程是左到右匹配模糊匹配。如上例子，如果把`<Route path="/" component={Home} />`上移到第一位，将会导致访问任何页面均渲染`<Home>`所在路由。可通过设置`<Route exact path="/" component={Home} />`，限制为精确匹配。
  
- 虽然React Router不支持`<Switch>` 以外的`<Route>`渲染，但v5.1以后，[推荐使用`useRouteMatch`](https://reacttraining.com/blog/react-router-v5-1/#useroutematch)

### 2.3 [Navigation (or Route Changers)](https://reactrouter.com/web/guides/primary-components/navigation-or-route-changers)

- `<Link>` 组件将创建应用内的跳转链接，本质上渲染后会形成一个`<a>`标签

  ```jsx
  <Link to="/">Home</Link>
  // <a href="/">Home</a>
  ```

- `<NavLink>` 是`<Link>`的特例，支持激活状态显示

  ```jsx
  <NavLink to="/react" activeClassName="hurray">
    React
  </NavLink>
  // When the URL is /react, this renders:
  // <a href="/react" className="hurray">React</a>
  
  // When it's something else:
  // <a href="/react">React</a>
  ```

- `<Redirect>` 重定向页面到。``<Redirect to="/404">`` 跳转到404页面



## 3. Hook组件

`useHistory`、`useLocation`、`useParams`、`useRouteMatch` 在hook之前都有等价的实现方式，但比较复杂、冗余。

- `useHistory`  返回`history`实例，来源于[history包（点击查看接口介绍）](https://github.com/ReactTraining/history/blob/master/docs/api-reference.md)。

  ```jsx
  import { useHistory } from "react-router-dom";
  
  const HomeButton() => {
    // history 对象
    let history = useHistory();
    const handleClick() => {
      history.push("/home");
    }
  
    return (
      <button type="button" onClick={handleClick}>
        Go home
      </button>
    );
  }
  ```

  - `history.location`
    - `history.location.pathname`  
    - `history.location.search` 
    - `history.location.hash` 
    - `history.location.state` 获取state对象
  - `history.action`
  - `history.push(path, [state])`
  - `history.replace(path, [state])`
  - `history.go(n)` 往回n步
  - `history.goBack() `，相当于`history.go(-1)`
  - `history.goForward()`，相当于`history.go(1)`
  - `history.length` 历史访问路径个数
  
  - ⚠️ history对象是可变的（[history is mutable](https://reactrouter.com/web/api/history/history-is-mutable)）

- `useLocation` 返回[location](https://reactrouter.com/web/api/location)实例，代表当前所在位置。具体接口，类似 `history.location`
  - ⚠️ 浏览器自带支持`location.*`一些操作，参考：https://developer.mozilla.org/en-US/docs/Web/API/Location。猜测React-router location实例只是前者的一个封装？

- [`useParams`](https://reactrouter.com/web/api/Hooks/useparams) 返回当前路径参数（`/contact/:id`），⚠️ 不是query 参数（`?username=zhangsan`）。如果想取query参数，参考后文4.4。

- [`useRouteMatch`](https://reactrouter.com/web/api/Hooks/useroutematch) 匹配（match）当前url，类似`<Route>`功能。相比于后者，`useRouteMatch`可以在渲染前接触到数据。

  ```jsx
  // 不用 useRouteMatch
  import { Route } from "react-router-dom";
  
  const BlogPost() => {
    return (
      <Route
        path="/blog/:slug"
        render={({ match }) => {
          // Do whatever you want with the match...
          return <div />;
        }}
      />
    );
  }
  
  // 不用 useRouteMatch
  import { useRouteMatch } from "react-router-dom";
  
  const BlogPost() => {
    let match = useRouteMatch("/blog/:slug");
    // Do whatever you want with the match...
    return match && <div />;
  }
  ```

  - [match](https://reactrouter.com/web/api/match) 对象包含了如何将一个url匹配到`<Route path='...'>`，如下属性：
    - params
    - isExact 是否精确匹配
    - path 
    - url
  - `useRouteMatch()` 不带参数，则返回当前 `<Route>`的match对象。或者接受`path` string对象，或者一个可以传递给`<Route>` 的object对象。

## 4. 功能示例

### 4.1 基础使用

- index.js 入口文件引入`<BrowserRouter>`
- App.jsx 通过`<Switch>`和`<Route>`集中设置路由
- Home.jsx 使用 `<Link>`组件跳转
- Contact.jsx 使用 `useLocation`、`useParams`、`useHistory`等hook

```jsx
// index.js
import React from "react";
import ReactDOM from "react-dom";
import { BrowserRouter } from "react-router-dom";
import App from './App';

ReactDOM.render(
  <BrowserRouter>
    <App />
  </BrowserRouter>,
  document.getElementById("root")
);
```

```jsx
// App.jsx
import React from "react";
import Home from './Home';
import Contact from './Contact';
import Page404 from './Page404';

const App = () => {
  return (
    <Switch>
      <Route exact path="/" component={Home} />
      <Route path="/contact/:id" component={Contact} />
      <Route path="/404" component={Page404} />
      <Redirect to="/404" />
    </Switch>
  );
}
```

```jsx
// Home.jsx
import React from "react";
import { Link } from "react-router-dom";

const Home = () => {
  return (
    <ul>
      <li><Link to='/login'>登陆</Link></li>
      <li><Link to='/register'>注册</Link></li>
    </ul>
  );
}
```

```jsx
// Contact.jsx
import React from "react";
import { useParams, useLocation, useHistory, useRouteMatch } from "react-router-dom";

const Contact = () => {
  const location = useLocation();
  const pathParams = useParams();
  const history = useHistory();
  const { path, url } = useRouteMatch();
  
  const btnHandle = () => {
    history.push("/login")
  }
  
  return (
    <ul>
      <li>`Path Name from useLocation(): ${location.pathname}`</li>
      <li>`Path Params from useParams(): ${pathParams.id}`</li>
      <li>
        <button onClick={btnHandle}>
  				跳转至login页面
				</button>
      </li>
      <li>`Path from useRouteMatch: ${path}`</li>
      <li>`Url from useRouteMatch: ${url}`</li>
    </ul>
  )
}
```

### 4.2 权限路由

路由跳转前，判断用户是否登陆。可开发通用`<PrivateRoute>`来做

```jsx
// App.jsx
import React from "react";
import Home from './Home';
import Contact from './Contact';
import Page404 from './Page404';

const PrivateRoute = ({ component: Component, ...restProps }) => {
  const { authState } = useContext(AuthContext);
  return (
    <Route {...restProps}
      render={props =>
        authState.isAuthenticated ? (
          <Component {...props} />
        ) : (
          <Redirect to="/sso" />
        )}
    />
  );
};

const App = () => {
  return (
    <Switch>
      <Route exact path="/" component={Home} />
      <PrivateRoute path="/contact/:id" component={Contact} />
      <Route path="/404" component={Page404} />
      <Redirect to="/404" />
    </Switch>
  );
}
```

### 4.3 从配置中生成路由

随着网站页面数越来越多，同时考虑到 路由处、菜单处都要配置一些与页面相关的信息。考虑集中配置下页面，然后程序化地生成路由。

```jsx
import React from "react";
import {
  BrowserRouter,
  Switch,
  Route,
  Link
} from "react-router-dom";

// 4个页面
const Sandwiches() => <h2>Sandwiches</h2>;
const Tacos() => <h2>Tacos</h2>;
const Bus() => <h2>Bus</h2>;
const Cart() => <h2>Cart</h2>;

// 路由配置
const routes = [
  {
    path: "/sandwiches",
    component: Sandwiches
  },
  {
    path: "/tacos",
    component: Tacos,
    routes: [
      {
        path: "/tacos/bus",
        component: Bus
      },
      {
        path: "/tacos/cart",
        component: Cart
      }
    ]
  }
];

// 将 routes转化为路由的组件
const RouteWithSubRoutes = (route) => {
  return (
    <Route
      path={route.path}
      render={props => (
        // 向 routes 属性传递值，可以保持嵌套
        <route.component {...props} routes={route.routes} />
      )}
    />
  );
}

export default const Example() {
  return (
    <BrowserRouter>
      // 内容部分
      <div>
        <ul>
          <li>
            <Link to="/tacos">Tacos</Link>
          </li>
          <li>
            <Link to="/sandwiches">Sandwiches</Link>
          </li>
        </ul>
				
        // 路由部分
        <Switch>
          {routes.map((route, i) => (
            <RouteWithSubRoutes key={i} {...route} />
          ))}
        </Switch>
      </div>
    </BrowserRouter>
  );
}
```

### 4.4 获取路径中query参数

⚠️ 再次强调下 `useParams`获取的是路径中的参数（`/contact/:id`），不是query中的参数`?username=zhangsan`）。需要自定义下

```jsx
import React from "react";
import { useLocation } from "react-router-dom";

// 假设路由地址为：/dashboard?from=2021-01-01&to=2021-12-31&metric=uv
const Dashboard = () => {
  const search = useLocation().search;
  // 直接用下面也行
  // const search = window.location.search;
  const qParams = new URLSearchParams(search);
  
  return (
    <ul>
      <li>`From: ${qParams.from}`</li>
      <li>`To: ${qParams.to}`</li>
      <li>`Metric: ${qParams.metric}`</li>
    </ul>
  )
}
```

