## 1.生成token
- 1.登录github并访问 [tokens](https://github.com/settings/tokens)页面
- 2. 生成token并拷贝到剪贴板上
### 访问token页面
![](http://img.zhufengpeixun.cn/1.token.png)
### 创建token,给权限
![](http://img.zhufengpeixun.cn/2.token.png)
### 把token拷贝出来
![](http://img.zhufengpeixun.cn/3.token.png)

## 2.修改 package.json
- name为 @用户名/原包名
- 把仓库上传到github,并把地址贴到`repository`里,仓库的名字和包名是一样的
- registry为 https://npm.pkg.github.com/用户名

```diff
{
+  "name": "@zhufengnodejs/zhufengreactui",
+  "repository":"https://github.com/zhufengnodejs/zhufengreactui",
  "publishConfig": {
+    "registry": "https://npm.pkg.github.com/zhufengnodejs"
  }
```

## 3.登录npm

```js
npm login --registry=https://npm.pkg.github.com --scope=@zhufengnodejs
username 为你的github用户名
password 为刚才第一步生成的token
```

## 4. 发包
```js
npm publish
```

## 5.在github上搜索包名
- [github仓库](https://github.com/zhufengnodejs/zhufengreactui)
- [packages](https://github.com/zhufengnodejs/zhufengreactui/packages/365183)