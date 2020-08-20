## 了解NPM

对于需要发布的包，有几个前置知识是需要知道的。

对于dependencies，只要有用户下载你这个包，dependencies里面的包都会进行安装。

对于devDependencies，用户下载你这个包不会安装这些包。

对于peerDependencies，用户下载你这个包后如果用户没安装会进行报警，如果用户已有这个包但版本与你的不匹配会报警。用户下载你这个包不会自动安装peerDependencies。

## 修改依赖

有了上面知识，我们需要对package.json进行修改。

所有的声明文件时不需要被用户安装的。

react，以及styled-components是需要放入peerDependencies的。

dependecies只保留polished

```json
	"dependencies": {
		"polished": "^3.6.5"
	},
	"peerDependencies": {
		"react": ">=16.8.0",
		"react-dom": ">=16.8.0",
		"styled-components": "^5.1.1"
	  },
```

## 编译输出

目前，这个包还不能用，因为没有编译输出，我们肯定不能用react-scripts来输出。需要自行制作编译。

由于使用ts，所以我们需要做个tsconfig.build.json作为输出的配置文件。

```json
{
	"compilerOptions": {
		"outDir": "dist",
		"target": "es5",
		"module": "esnext",
		"declaration": true,
		"jsx": "react",
		"moduleResolution": "node",
		"allowSyntheticDefaultImports": true,
		"skipLibCheck": true,
		"esModuleInterop": true,
		"strict": true,
		"forceConsistentCasingInFileNames": true,
		"resolveJsonModule": true
	},
	"include": ["src"],
	"exclude": ["src/**/*.stories.tsx", "src/**/*.test.tsx"]
}
```

修改script配置：

```
"build": "tsc -p tsconfig.build.json",
```

之后可以生成dist目录

将dist 添加进gitignore不写入。

package.json对发包目录进行配置：

```
	"files": [
		"dist"
	],
```

对私有进行修改：

```
	"private": false,
```



## 发包

使用npm publish进行发包。

## 使用验证

我们可以新建个cra项目，安装刚才发的包进行测试。

由于styledComponent是peerDependency所以记得还要安装下styledComponent。

```javascript
import React from "react";
import Button from "atom-design-explorer/dist/components/button";
import { GlobalStyle } from "atom-design-explorer/dist/components/shared/global";

function App() {
	return (
		<div className="App">
			<GlobalStyle></GlobalStyle>
			<Button appearance="primary">2222</Button>
		</div>
	);
}

export default App;

```

npm run start  可以看见，全局样式和按钮已经正确的显示出来了。

## TreeShaking&SideEffects

这种导入方式略微有点麻烦，可以制作个index.tsx将导出全部统一起来管理。这样的话就不得不说一下tree-shaking了。

使用es语法可以静态分析模块从而进行拆包，比如我们要做的index.tsx将所有导出集中到一个文件上，如果没有拆包，那么我们引入这个文件上的任意一个文件，都会把整个库里所有文件打包进来。而如果要拆包，需要配置sideEffects，因为sideEffects默认为true，代表我这个包的文件全部有副作用，别给我乱tree-shaking。对于我们这种无css文件组成的组件库，只要没有写什么乱七八糟的骚操作，直接配置为false即可，代表我的包你随便拆吧，都是单独的模块。

制作index.tsx

```typescript
export * from "./components/button";
export * from "./components/shared/global";
export * from "./components/shared/styles";
export * from "./components/shared/animation";
```

对package.json进行设置：

```
	"module": "dist/index.js",
	"types": "dist/index.d.ts",
```

让ts可以识别入口。

以及sideeffects:

```
"sideEffects": false,
```

调整版本号再发布一次。

进行验证：

```javascript
import React from "react";
import { Button } from "atom-design-explorer";
import { GlobalStyle } from "atom-design-explorer";

function App() {
	return (
		<div className="App">
			<GlobalStyle></GlobalStyle>
			<Button appearance="primary">2222</Button>
		</div>
	);
}

export default App;

```

npm run start 看见按钮就成功了。

## 今日作业

使用npm发包成功验证自己库效果。

### 可选作业

#### GitHub Package 发包
- 您可以将新包发布到[ GitHub 包注册表](https://docs.github.com/cn/packages/publishing-and-managing-packages)，查看和安装现有包，并且在特殊情况下删除现有包

 目前只支持作用域包，所以我们需要先进行改名，改成作用域+包名。

githubPackage的包不会显示在npm页面上，虽然都是使用npm进行安装。所以很多开源项目一般不发github package。

以我的包为例，我的包本来叫atom-design-explorer，现在改名叫"@yehuozhili/atom-design-explorer"

同时，package.json新增字段：

```
 "publishConfig": {
    "registry": "https://npm.pkg.github.com/@yehuozhili"
  },
```

上传代码至github。

试着发布看看

如果你以前已经登录过，可能你不需要登录作用域名就已经发布成功了，打开github检查仓库中的package栏目是否有对应的包。

如果没有，你可能还需要进行下面的步骤：

使用作用域名进行登录：

```
npm login --registry=https://npm.pkg.github.com --scope=@yehuozhili
```

会让你输入密码，这个密码是**githubtoken**,注意，是**githubtoken** ,是**githubtoken**。重要的事说3遍，因为这个设计有点反人类，它是粘贴并且看不见粘贴上没有。

githubtoken在github的[setting](https://github.com/settings/tokens)里面生成。

88d3b3913ba206f0756529421c398c0a03acbfb7

#### GitHub Actions 持续集成

storybook是可以生成静态页面的，需要改一下命令：

```
"build-storybook": "build-storybook --no-dll --quiet"
```

为了让action安装完毕后正常工作，还需要将peerDependencies的依赖复制进devDependencies。

项目的settings中的secrets里面可以填入githubToken。 b285ca4574ed83df39d184bfc69fcbde63ade079

这里我name为ACCESS_TOKEN，值为生成的githubToken。

[建立github actions](https://github.com/zhufengnodejs/zhufengreactui/actions)
，使用向导创建main.yml，复制以下内容粘贴进去：

```
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
          GITHUB_TOKEN: ${{ secrets.ACCESS_TOKEN }}
          BRANCH: gh-pages # The branch the action should deploy to.
          FOLDER: storybook-static # The folder the action should deploy.
```

之后只要push就可以自动部署githubpage页面。

[github-actions参考文档](http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)

#### COVERALLS代码覆盖率

前面我们已经可以通过npm run coverage生成代码覆盖率，这样还要敲个命令麻烦，可以通过coveralls来告诉我们。

官网https://coveralls.io/

注册并登录，同步后使用add repo添加已上传项目。并在github仓库的security里设置环境变量。

npm官网https://www.npmjs.com/package/coveralls

我们还需要自己安装coveralls

在scripts中加入：

```
"coverall": "npm run coverage  && cat ./coverage/lcov.info | ./node_modules/coveralls/bin/coveralls.js && rm -rf ./coverage",
```

同时githubActions中加入：

```
  - name: Coveralls
        env:
          COVERALLS_SERVICE_NAME: 'GitHub CI'
          COVERALLS_GIT_BRANCH: master
          COVERALLS_REPO_TOKEN : ${{ secrets.COVERALLS_REPO_TOKEN }}
        run: |
          npm run coverall
```

push 触发action。

从而在coverall网站上获得详细的覆盖率信息，并且第几行没测到可以直接翻出来看源代码。

#### CODE FACTOR代码因子

对于多人协作项目，代码因子可以对线上代码进行检测并提供pr。

登录并注册账号 https://www.codefactor.io/

import 需要检测仓库。

之后需要等待一段时间，对你仓库的代码进行检测。可能会提出一些问题之类，最终会给你仓库代码质量打个评分。

#### 添加sponsor

github设置里添加sponsor选项，点击对勾后你的项目会出现sponsor按钮，同时并要求配置yml文件，由于自带支付不支持中国地区，所以使用自定义外链，链接到你的收款码即可。

#### 完成Highlight组件

这个组件本来不在计划内，后续组件也不需要用这个组件，但是可能有人需要用csf来写代码，所以有必要写一下。

一般比较流行的是highlightjs和prismjs，这次使用prismjs来制作。

https://www.npmjs.com/package/prismjs

直接放代码 ，唯一要说的就是这里的require，为什么不用import，因为如果用import 的话，里面没使用，就会被treeShaking掉，所以这里用require导入，防止被treeShaking。

```tsx
import Prism from "prismjs";

require("prismjs/components/prism-typescript");
require("prismjs/components/prism-javascript");
require("prismjs/components/prism-json");
require("prismjs/components/prism-css");

export type languagesType = "javascript" | "typescript" | "json" | "css";

const HighlightBlock = styled.div`
	.token.cdata,
	.token.comment,
	.token.doctype,
	.token.prolog {
		color: #708090;
	}
	.token.punctuation {
		color: #999;
	}
	.namespace {
		opacity: 0.7;
	}
	.token.boolean,
	.token.constant,
	.token.deleted,
	.token.number,
	.token.property,
	.token.symbol,
	.token.tag {
		color: #905;
	}
	.token.attr-name,
	.token.builtin,
	.token.char,
	.token.inserted,
	.token.selector,
	.token.string {
		color: #690;
	}
	.language-css .token.string,
	.style .token.string,
	.token.entity,
	.token.operator,
	.token.url {
		color: #a67f59;
		background: hsla(0, 0%, 100%, 0.5);
	}
	.token.atrule,
	.token.attr-value,
	.token.keyword {
		color: #07a;
	}
	.token.class-name,
	.token.function {
		color: #dd4a68;
	}
	.token.important,
	.token.regex,
	.token.variable {
		color: #e90;
	}
	.token.bold,
	.token.important {
		font-weight: 700;
	}
	.token.italic {
		font-style: italic;
	}
	.token.entity {
		cursor: help;
	}

	code,
	pre {
		color: ${color.darkest};
	}

	code {
		white-space: pre;
	}

	pre {
		padding: 11px 1rem;
		margin: 1rem 0;
		background: ${color.lighter};
		overflow: auto;
	}

	.language-bash .token.operator,
	.language-bash .token.function,
	.language-bash .token.builtin {
		color: ${color.darkest};
		background: none;
	}
`;

export type HighlightProps = {
	language?: languagesType;
};

const htmlEscapes = {
	"&": "&amp;",
	"<": "&lt;",
	">": "&gt;",
	'"': "&quot;",
	"'": "&#39;",
};
const reUnescapedHtml = /[&<>"']/g;
const reHasUnescapedHtml = RegExp(reUnescapedHtml.source);

function escape(string: string) {
	return string && reHasUnescapedHtml.test(string)
		? string.replace(
				reUnescapedHtml,
				(chr: string) => htmlEscapes[chr as keyof typeof htmlEscapes]
		  )
		: string || "";
}

export function Highlight(props: PropsWithChildren<HighlightProps>) {
	const nodeRef = useRef<HTMLDivElement>(null);
	const { children, language } = props;
	const codeBlock = (
		<div dangerouslySetInnerHTML={{ __html: escape(children + "") }} />
	);
	useEffect(() => {
		if (nodeRef.current) {
			Prism.highlightAllUnder(nodeRef.current);
		}
	}, [nodeRef]);
	return (
		<HighlightBlock ref={nodeRef}>
			<pre className={`language-${language}`}>
				<code className={`language-${language}`}>{codeBlock}</code>
			</pre>
		</HighlightBlock>
	);
}

```

story随便整点代码写上去：

```tsx
export default {
	title: "Highlight",
	component: Highlight,
	decorators: [withKnobs],
};


const javascriptCode = `import React from 'react'

const MyComponent = () => (
  <div>My component renders all the things</div>
)

export default MyComponent
`;

const typescriptCode = `import React from 'react'

export function Highlight(props: PropsWithChildren<HighlightProps>) {
	const nodeRef = useRef<HTMLDivElement>(null);
	const { children, language } = props;
	const codeBlock = (
		<div dangerouslySetInnerHTML={{ __html: escape(children + "") }} />
	);
	useEffect(() => {
		if (nodeRef.current) {
			Prism.highlightAllUnder(nodeRef.current);
		}
	}, [nodeRef]);
	return (
		<HighlightBlock ref={nodeRef}>
			<pre className={\`language-$\{language}\`}>
				<code className={\`language-$\{language}\`}>{codeBlock}</code>
			</pre>
		</HighlightBlock>
	);
}
`;

const cssCode = `.someClass {
  position: relative;
}
`;

const jsonCode = `{
  "name": "@yourorg/package",
  "version": "0.0.1",
  "description": "A sweet package",
  "repository": {
    "type": "git",
    "url": "https://github.com/yourorg/package.git"
  }
}`;
export const javascript = () => (
	<>
		<Highlight language="javascript">{javascriptCode}</Highlight>
	</>
);
export const typescript = () => (
	<>
		<Highlight language="typescript">{typescriptCode}</Highlight>
	</>
);
export const css = () => (
	<>
		<Highlight language="css">{cssCode}</Highlight>
	</>
);
export const json = () => (
	<>
		<Highlight language="css">{jsonCode}</Highlight>
	</>
);
```

