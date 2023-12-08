# 一文搞懂前端组件发布 npm 库
发布 npm
## 1. 创建项目
`npm version patch` 递增更新 npm 版本 **git控制下，需要先commit再更新版本**

`npm login` 登录 npm 账号 ，该命令会弹出一个带chunk的链接，点击跳转浏览器登录

`npm publish` 发布 npm 包，不报错即成功，访问 https://www.npmjs.com/ 就能看到了
 
`npm run build` 构建产物-> dist/library.js