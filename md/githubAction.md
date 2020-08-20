## github-actions参考文档
- [github-actions参考文档](http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)

## 1.生成token
- 1.登录github并访问 [tokens](https://github.com/settings/tokens)页面
- 2. 生成token并拷贝到剪贴板上
### 访问token页面
![](http://img.zhufengpeixun.cn/1.token.png)
### 创建token,给权限
![](http://img.zhufengpeixun.cn/2.token.png)
### 把token拷贝出来
![](http://img.zhufengpeixun.cn/3.token.png)

## 2.修改命令
在package.json中增加修改`build-storybook`命令

```diff
  "scripts": {
    "start": "react-scripts start",
    "build": "tsc -p tsconfig.build.json",
    "test": "react-scripts test",
    "eject": "react-scripts eject",
    "storybook": "start-storybook -p 6006 -s public",
+    "build-storybook": "build-storybook --no-dll --quiet",
    "coverage": "react-scripts test --coverage --watchAll=false",
    "coverall": "npm run coverage  && cat ./coverage/lcov.info | ./node_modules/coveralls/bin/coveralls.js && rm -rf ./coverage"
  },
```

## 3.添加ACCESS_TOKEN
- 在[secrets](https://github.com/zhufengnodejs/zhufengreactui/settings/secrets)中添加一个Secret
- 名称为`ACCESS_TOKEN`, 值为上面第一步生成的`token`

![writesccesstoken](http://img.zhufengpeixun.cn/writesccesstoken.png)

## 4.添加Actions
- 打开项目的[actions(https://github.com/zhufengnodejs/zhufengreactui/actions)页面
- 根据向导添加脚本,会自动写到`.github\workflows\blank.yml`目录中

![setupworkflow](http://img.zhufengpeixun.cn/setupworkflow.png)

```js
name: Build and Deploy
on: [push]
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout 🛎️
        uses: actions/checkout@v2 # If you're using actions/checkout@v2 you must set persist-credentials to false in most cases for the deployment to work correctly.
        with:
          persist-credentials: false

      - name: Install and Build 🔧 # This example project is built using npm and outputs the result to the 'build' folder. Replace with the commands required to build your project, or remove this step entirely if your site is pre-built.
        run: |
          npm install
          npm run build-storybook

      - name: Deploy 🚀
        uses: JamesIves/github-pages-deploy-action@releases/v3
        with:
          GITHUB_TOKEN: ${{secrets.ACCESS_TOKEN}}
          BRANCH: gh-pages # The branch the action should deploy to.
          FOLDER: storybook-static # The folder the action should deploy. 
```

## 5.拉取Action代码
- 在本地用`git pull` 把服务器端生成的`blank.yml`代码拉取本地

## 6.合并后推送
- 在[这里](https://github.com/zhufengnodejs/zhufengreactui/actions)就可以看到构建任务了

![buildsuccess](http://img.zhufengpeixun.cn/buildsuccess.png)