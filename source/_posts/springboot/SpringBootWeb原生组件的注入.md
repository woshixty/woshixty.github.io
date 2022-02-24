Web原生组件的注入（Servlet、Filter、Listener）

### 一、使用`Servlet API`

- `@ServletComponentScan(basePackages = "com.example.xty")`

指定原生`Servlet`组件的放置位置

- `@WebServlet(urlPatterns = "/my")`

直接响应没有Spring的拦截器



### 二、使用`RegistrationBean`