## 1. Situation（情景）什么背景要搭建自己的组件库

公司有自己的业务组件，每次去一个新的项目都需要手动 copy 业务组件过去，这样可能会导致代码缺失，组件依赖的包版本也不一样
所以我们想能不能像 ant-design 组件库一样，通过 npm i 安装方式直接使用

## 2. Target（目标） 搭建组件库的目的

通过 npm i 的形式安装方式来 `快速的引入自己需要的组件库`。

## 3. Action（行动） 搭建自己的组件库

### 3.1 react 组件的设计原则

#### 有意义

-   命名准确，充分表意
-   参数准确，必要的类型检查
-   适当的注释

#### 通用性

-   不要耦合特殊的业务功能
-   不要包含特定的代码处理逻辑

`错误代码演示`

```jsx
import React from 'react';
interface CardProps {
	title: string;
	children: React.ReactNode;
	install: boolean; // 某业务参数
}
function Card(props: CardProps) {
	return (
		<div className="card">
			   <div className="card-title">{props.title}</div>
			   <div className="card-content">{props.children}</div>
			{/* 特定的代码处理逻辑 */}
			{props.install ? (
				<span className="card-tag">已安装</span>
			) : (
				<span className="card-tag">未安装</span>
			)}
			  
		</div>
	);
}
```

`正确代码演示`

```jsx
import React from 'react';
interface CardProps {
	title: string;
	children: React.ReactNode;
	tag: string | null;
}
function Card(props: CardProps) {
	return (
		<div className="card">
			   <div className="card-title">{props.title}</div>
			   <div className="card-content">{props.children}</div>
			{/* 特定的代码处理逻辑通过传参的形式解决 */}
			  {props.tag && <span className="card-tag">{props.tag}</span>}
			  
		</div>
	);
}
```

#### 无状态，无副作用

-   状态向上层提取 (props)，尽量少⽤内部状态
-   解耦 IO 操作

#### 避免过度封装

-   合理冗余
-   避免过度抽象

#### 单一职责

-   ⼀个组件只完成⼀个功能
-   尽量避免不同组件⻅相互依赖、循环依赖

#### 易于测试

-   更容易的单元测试覆盖

### 3.2 创建项目

#### 1. 创建 React-Component-lyk 结构

react-components-lyk
├─ src 源代码目录
├─ test 测试目录
├─ rollup.config.js rollup 打包配置文件
├─ babel.config babel 配置文件
├─ tsconfig ts 配置文件  
├─ README.md  
└─ package.json

### 2. 在 `package.json` 文件中指定入口文件为 cli.js

```js
"bin": {
    "lyk-cli": "./bin/cli.js" // 配置启动文件路径，lyk-cli 为别名
},
```

#### 3. 打开 `cli.js` 进行编辑

```js
#! /usr/bin/env node

// #! 符号的名称叫 Shebang，用于指定脚本的解释程序
// Node CLI 应用入口文件必须要有这样的文件头
// 如果是Linux 或者 macOS 系统下还需要修改此文件的读写权限为 755
// 具体就是通过 chmod 755 cli.js 实现修改

// 检查入口文件是否正常运行
console.log('lyk-react-cli working ~');
```

#### 4. 使用 `npm link` 链接到全局，方便本地测试

```js
npm link
```

#### 5. 在控制台测试一下我们配置的 `lyk-cli`

在命令行执行 lyk-cli 命令输出结果如下

```js

~/Desktop/cli/lyk-react-cli -> lyk-cli
lyk-react-cli working   // 打印内容
```

OK，得到了我们想要的打印内容

### 2. 创建脚手架启动命令 [commander](https://www.npmjs.com/package/commander)

1. 通过 commander 插件去实现自定义命令行指令
2. 如果创建的存在，需要提示是否覆盖

#### 1. 安装依赖

```js
npm install commander --save
```

#### 2. 创建命令

打开 `cli.js` 进行编辑

```js
#! /usr/bin/env node

const program = require('commander');
const packageConfig = require('../package.json');

// 1. 创建自定义命令         lyk-cli create app-name -f/--force
program
	// 定义命令和参数           lyk-cli create app-name
	.command('create <app-name>')
	// 描述
	.description('create a new project')
	// -f or --force 为强制创建，如果创建的目录存在则直接覆盖         配置 options 参数
	.option('-f, --force', 'overwrite target directory if it exist')
	.action((name, options) => {
		// 打印执行结果  TODO: 在这里做你的事情
		console.log('name:', name, 'options:', options);
	});

// 2. 配置版本命令信息  lyk-cli -V
program
	// 配置版本号信息
	.version(`v${packageConfig.version}`)
	.usage('<command> [option]');

// 解析用户执行命令传入参数
program.parse(process.argv);
```

完成上面的配置后，重新执行一下 lyk-cli 命令，出现如下图所示，就代表自定义命令创建完成

![commander创建命令1](https://cdn.nlark.com/yuque/0/2022/png/566044/1663229835853-949320c4-c184-4473-9fc5-6cee32f9da38.png)

接着我们可以使用以下几个命令做操作

-   lyk-cli create project-name
    -   创建目录名称为 project-name 的项目
-   lyk-cli create project-name -f/--force
    -   创建目录名称为 project-name 的项目，并且覆盖之前创建的目录
-   lyk-cli -V / --version
    -   查看脚手架的版本号
-   lyk-cli -h / --helop
    -   查看脚手架的帮助文档

#### 3. 执行命令

创建 lib 文件夹并在文件夹下创建 `create.js`，用来做创建项目目录

在创建目录的时候，需要思考一个问题：目录是否已经存在？

1. 如果不存在，直接创建
2. 如果存在
    - 当 { force: true } 时，直接移除原来的目录，直接创建
    - 当 { force: false } 时 询问用户是否需要覆盖

上面要对文件进行操作，用到了 fs 的扩展工具 fs-extra，内部对 fs 模块进行了封装

```js
# fs-extra 是对 fs 模块的扩展，支持 promise
$ npm install fs-extra --save
```

实现 `create.js` 的逻辑

```js
// lib/create.js

const path = require('path');
const fs = require('fs-extra');

/**
 * 创建一个目录
 * @param {*} projectName 项目名称
 * @param {*} options 参数
 */
module.exports = async function (projectName, options) {
	console.log(
		`create 项目名称为: ${projectName} options: ${JSON.stringify(options)}`
	);
	// 执行创建命令

	// 1. 当前命令行选择的目录
	const cwd = process.cwd();

	// 2. 需要创建的目录地址
	const targetAir = path.join(cwd, projectName);

	// 3. 判断目录是否存在
	// 不存在，直接创建目录
	if (!fs.existsSync(targetAir)) {
		// TODO: 创建目录
	} else {
		// 存在目录，做如下判断

		// 是否为强制创建？
		if (options.force) {
			// 删除已有的的目录
			await fs.remove(targetAir);

			// TODO: 创建目录
		} else {
			// TODO：询问用户是否确定要覆盖
		}
	}
};
```

部分逻辑后面处理

#### 4. 创建更多命令

如果想添加其他命令也是同样的处理方式，这里就不扩展说明了，示例如下

```js
// bin/cli.js

// 配置 config 命令
program
	.command('config [value]')
	.description('inspect and modify the config')
	.option('-g, --get <path>', 'get value from option')
	.option('-s, --set <path> <value>')
	.option('-d, --delete <path>', 'delete option from config')
	.action((value, options) => {
		console.log(value, options);
	});

// 配置 ui 命令
program
	.command('ui')
	.description('start add open roc-cli ui')
	.option('-p, --port <port>', 'Port used for the UI Server')
	.action((option) => {
		console.log(option);
	});
```

#### 5. 完善帮助信息

结尾处少了一条说明信息，这里我们做补充，重点需要注意说明信息是带有颜色的，这里就需要用到我们工具库里面的 [chalk](https://www.npmjs.com/package/chalk/v/4.1.2) 来处理

安装 chalk

```js
npm install chalk --save
```

基本使用

```js
// bin/cli.js
const chalk = require('chalk');

program
	// 监听 --help 执行
	.on('--help', () => {
		// 新增说明信息
		console.log(
			`\r\nRun ${chalk.cyan(
				`zr <command> --help`
			)} for detailed usage of given command\r\n`
		);
	});
```

#### 6. 打印 Logo

安装工具库 [figlet](https://www.npmjs.com/package/figlet) ，给脚手架整个 Logo

```js
npm i figlet --save
```

基本使用

```js
// bin/cli.js

program.on('--help', () => {
	// 使用 figlet 绘制 Logo
	console.log(
		'\r\n' +
			figlet.textSync('lyk', {
				font: 'Ghost',
				horizontalLayout: 'default',
				verticalLayout: 'default',
				width: 80,
				whitespaceBreak: true,
			})
	);
	// 新增说明信息
	console.log(
		`\r\nRun ${chalk.cyan(
			`lyk-cli <command> --help`
		)} for detailed usage of given command\r\n`
	);
});
```

### 3. 询问用户问题获取创建所需信息

#### 1. 询问是否覆盖已存在的目录

安装工具库 [inquirer](https://www.npmjs.com/package/inquirer) ，询问用户是否需要覆盖已存在的目录

```js
npm i inquirer --save
```

基本使用

```js
// lib/create.js

const path = require('path'); // 路劲操作
const fs = require('fs-extra'); // 文件操作
const inquirer = require('inquirer'); // 询问用户信息

/**
 * 创建一个目录
 * @param {*} projectName 项目名称
 * @param {*} options 参数
 */
module.exports = async function (projectName, options) {
	console.log(
		`create 项目名称为: ${projectName} options: ${JSON.stringify(options)}`
	);
	// 执行创建命令

	// 1. 当前命令行选择的目录
	const cwd = process.cwd();

	// 2. 需要创建的目录地址
	const targetAir = path.join(cwd, projectName);

	// 3. 判断目录是否存在
	// 不存在，直接创建目录
	if (!fs.existsSync(targetAir)) {
		// TODO: 创建目录
		console.log('create directory');
	} else {
		// 存在目录，做如下判断

		// 是否为强制创建？
		if (options.force) {
			// 删除已有的的目录
			await fs.remove(targetAir);
			console.log('delete directory');
		} else {
			// 询问用户是否确定要覆盖
			const { action } = await inquirer.prompt([
				{
					name: 'action',
					type: 'list',
					message: 'Target directory already exists Pick an action:',
					choices: [
						{
							name: 'Overwrite',
							value: 'overwrite',
						},
						{
							name: 'Cancel',
							value: false,
						},
					],
				},
			]);
			// 不需要覆盖直接跳出
			if (!action) {
				return;
			} else if (action === 'overwrite') {
				// 移除已存在的目录
				console.log(`\r\nRemoving...`);
				console.log('overwirte directory');
				await fs.remove(targetAir);
			}
		}
	}
};
```

简单测试一下

-   创建项目 my-project-test1，my-project-test2
-   执行 lyk-cli create my-project-test1，效果如下，my-project-test1 被移除

    ![覆盖已存在的目录](https://cdn.nlark.com/yuque/0/2023/png/566044/1675072273358-aac9ef14-beed-437c-af75-f563227830c4.png)

-   执行 lyk-cli create my-project-test2 -f，可以直接看到 my-project-test2 被移除

#### 2. 如何获取模板信息

##### 1. 上传模板到一个远程仓库

这里我上传到了 github 上面 [github.com/lyk-cli](https://github.com/lyk-cli)

##### 2. vue-template 模板为例

-   给远程仓库模板建立了以下版本信息供选择

##### 3. 我们通过 github 提供的 api 去拉远程仓库代码

-   api.github.com/orgs/lyk-cli 接口获取模板信息
-   api.github.com/repos/lyk-cli 接口获取版本信息

我们在 lib 目录下创建一个 axios.js 专门处理模板和版本信息的获取

1. 安装 axios

```js
npm i axios --save
```

2. 使用 axios

```js
// 通过 axios 处理请求
const axios = require('axios'); // 引入 axios 库

// 对响应体进行拦截处理
axios.interceptors.response.use((res) => {
	return res.data;
});

/**
 * 获取模板列表
 * @returns Promise
 */
async function getTemplateList() {
	return axios.get('https://api.github.com/orgs/lyk-cli/repos');
}

/**
 * 获取模板版本信息
 * @param {string} template 模板名称
 * @returns Promise
 */
async function getVersionList(template) {
	return axios.get(
		`https://api.github.com/repos/zhurong-cli/${template}/tags`
	);
}

module.exports = {
	getTemplateList,
	getVersionList,
};
```

#### 3. 用户选择模板

##### 1. ora 命令行 loading 动效

项目加载中时间等待比较长，我们需要一个 [ora](https://www.npmjs.com/package/ora) 库执行 加载动画特效

安装 ora

```js
npm i ora --save
```

使用 ora

```js
// lib/utils.js
const ora = require('ora'); // ora 动画特效库

// 添加加载动画
async function wrapLoading(fn, message, ...args) {
	// 使用 ora 初始化，传入提示信息 message
	const spinner = ora(message);
	// 开始加载动画
	spinner.start();

	try {
		// 执行传入方法 fn
		const result = await fn(...args);
		// 结束加载动画
		spinner.succeed();
		// 返回结果
		return result;
	} catch (error) {
		// 加载动画提示失败
		spinner.fail('Request failed, refetch ...');
	}
}

module.exports = {
	wrapLoading,
};
```

##### 2. GeneratorProject.js 来处理项目创建核心逻辑

```js
// lib/GeneratorProject.js
const { getTemplateList, getVersionList } = require('./axios'); // 引入接口拉取模板
const inquirer = require('inquirer'); // 询问用户信息
const { wrapLoading } = require('./utils'); // 引入公共方法

// 生成项目的生成器构造函数
class GeneratorProject {
	constructor(projectName, targetDir) {
		this.projectName = projectName; // 项目名称
		this.targetDir = targetDir; // 项目创建位置
	}

	// 获取用户选择的模板
	// 1）从远程拉取模板数据
	// 2）用户选择自己要下载的模板名称
	// 3) 返回用户选择的模板名称
	async getTemplateName() {
		// 1）从远程拉取模板数据
		const templateList = await wrapLoading(
			getTemplateList,
			'waiting fetch template'
		);

		// 模板不存在直接返回
		if (!templateList || templateList.length === 0)
			return console.log('Template does not exist');

		// 过滤我们需要的模板名称
		const templates = templateList.map((item) => item.name);

		// 2）用户选择自己新下载的模板名称
		const { template } = await inquirer.prompt({
			name: 'template',
			type: 'list',
			choices: templates,
			message: 'Please choose a template to create project',
		});

		// 3）返回用户选择的名称
		return template;
	}

	// 核心创建逻辑
	async create() {
		// 1）获取模板名称
		const templateName = await this.getTemplateName();
		console.log('Template selected:' + templateName);
	}
}

module.exports = GeneratorProject;
```

##### 3. 在 create.js 中引入 Generator 类

```js
// lib/create.js

const path = require('path'); // 路劲操作
const fs = require('fs-extra'); // 文件操作
const inquirer = require('inquirer'); // 询问用户信息
const Generator = require('./GeneratorProject'); // 引入模板生成器

/**
 * 创建一个目录
 * @param {*} projectName 项目名称
 * @param {*} options 参数
 */
module.exports = async function (projectName, options) {
	console.log(
		`create 项目名称为: ${projectName} options: ${JSON.stringify(options)}`
	);
	// 执行创建命令

	// 1. 当前命令行选择的目录
	const cwd = process.cwd();

	// 2. 需要创建的目录地址
	const targetAir = path.join(cwd, projectName);

	// 3. 项目目录存在
	if (fs.existsSync(targetAir)) {
		// ...
	}
	// 4. 创建项目
	console.log('create directory loading...');
	// 初始化创建项目
	const generator = new Generator(projectName, targetAir);
	// 开始创建项目
	generator.create();
};
```

##### 4. 测试一下 demo

我们再运行一下 lyk-cli create my-vue-project1，如图所示

![模板列表](https://cdn.nlark.com/yuque/0/2023/png/566044/1675158432324-6ed6ebe4-e8d3-4489-a087-8368b3ac056a.png)

这里我选择了 vue2.0-template，如图所示
![选择模板](https://cdn.nlark.com/yuque/0/2023/png/566044/1675158680807-a4387c0f-be75-44e2-8859-c7fabf2b0023.png)

#### 4. 用户选择版本

步骤和上面用户选择模板一样，先通过获取模板名称，再通过模板名称获取模板的版本信息

`代码如下`

```js
// lib/GeneratorProject.js
const { getTemplateList, getVersionList } = require('./axios'); // 引入接口拉取模板
const inquirer = require('inquirer'); // 询问用户信息
const { wrapLoading } = require('./utils'); // 引入公共方法

// 生成项目的生成器构造函数
class GeneratorProject {
	constructor(projectName, targetDir) {
		this.projectName = projectName; // 项目名称
		this.targetDir = targetDir; // 项目创建位置
	}

	// 获取用户选择的模板
	// 1）从远程拉取模板数据
	// 2）用户选择自己要下载的模板名称
	// 3) 返回用户选择的模板名称
	async getTemplateName() {
		// 1）从远程拉取模板数据
		const templateList = await wrapLoading(
			getTemplateList,
			'waiting fetch template'
		);

		// 模板不存在直接返回
		if (!templateList) return console.log('Template does not exist');

		// 过滤我们需要的模板名称
		const templates = templateList.map((item) => item.name);

		// 2）用户选择自己新下载的模板名称
		const { template } = await inquirer.prompt({
			name: 'template',
			type: 'list',
			choices: templates,
			message: 'Please choose a template to create project',
		});

		// 3）返回用户选择的名称
		return template;
	}

	// 获取用户选择的版本
	// 1）基于 template 结果，远程拉取对应的 tag 列表
	// 2）用户选择自己需要下载的 tag
	// 3) 返回用户选择的 tag

	async getTagVersion(template) {
		// 1）基于 template 结果，远程拉取对应的 tag 列表
		const versions = await wrapLoading(
			getVersionList,
			'waiting fetch tag',
			template
		);
		// 版本号不存在
		if (!versions || versions.length === 0) return;

		// 过滤我们需要的 tag 名称
		const versionList = versions.map((tag) => tag.name);

		// 2）用户选择自己需要下载的 tag
		const { tag } = await inquirer.prompt({
			name: 'tag',
			type: 'list',
			choices: versionList,
			message: 'Place choose a tag to create project',
		});

		// 3）返回用户选择的 tag
		return tag;
	}

	// 核心创建逻辑
	async create() {
		// 1）获取模板名称
		const templateName = await this.getTemplateName();
		console.log('Template selected:' + templateName);

		// 2) 根据模板名称获取tag版本信息
		const versionNo = await this.getTagVersion(templateName);
		console.log('version selected:' + versionNo);
	}
}

module.exports = GeneratorProject;
```

执行 `lyk-cli create my-project`，测试结果如下图所示
![模板版本列表](https://cdn.nlark.com/yuque/0/2023/png/566044/1675230938835-8d096400-7519-4e59-bb9a-06714955edeb.png)

这里我选择了 v2.0.0，如图所示
![选择模板版本](https://cdn.nlark.com/yuque/0/2023/png/566044/1675230944101-5f808099-ca53-4d09-96c5-366d87086c3d.png)

### 4. 下载远程模板

下载远程模版需要使用 [download-git-repo](https://www.npmjs.com/package/download-git-repo) 库，它本身不支持 promise 的，需要使用 util 模块中的 [promisify](http://nodejs.cn/api/util.html#util_util_promisify_original) 方法对其进行 promise 化

#### 1. 安装 download-git-repo

```js
npm i download-git-repo --save
```

#### 2. download-git-repo 进行 promise 化处理

```js
// lib/GeneratorProject.js
...
const util = require('util'); // 引入util工具模块
const downloadGitRepo = require('download-git-repo'); // 引入下载模板的库

// 生成项目的生成器构造函数
class GeneratorProject {
	constructor(projectName, targetDir) {
		this.projectName = projectName; // 项目名称
		this.targetDir = targetDir; // 项目创建位置

		// 对 download-git-repo 进行 promise 化
		this.downloadGitRepo = util.promisify(downloadGitRepo);
	}
    ...
}
module.exports = GeneratorProject;
```

#### 3. 核心下载功能

```js
// lib/GeneratorProject.js
const { getTemplateList, getVersionList } = require('./axios'); // 引入接口拉取模板
const inquirer = require('inquirer'); // 询问用户信息
const { wrapLoading } = require('./utils'); // 引入公共方法
const util = require('util'); // 引入util工具模块
const downloadGitRepo = require('download-git-repo'); // 引入下载模板的库
const path = require('path'); // 引入path模块
const chalk = require('./chalk'); // 引入颜色模块

// 生成项目的生成器构造函数
class GeneratorProject {
	constructor(projectName, targetDir) {
		this.projectName = projectName; // 项目名称
		this.targetDir = targetDir; // 项目创建位置

		// 对 download-git-repo 进行 promise 化
		this.downloadGitRepo = util.promisify(downloadGitRepo);
	}

	// 获取用户选择的模板
	// 1）从远程拉取模板数据
	// 2）用户选择自己要下载的模板名称
	// 3) 返回用户选择的模板名称
	async getTemplateName() {
		// 1）从远程拉取模板数据
		const templateList = await wrapLoading(
			getTemplateList,
			'waiting fetch template'
		);

		// 模板不存在直接返回
		if (!templateList || templateList.length === 0)
			return console.log('Template does not exist');

		// 过滤我们需要的模板名称
		const templates = templateList.map((item) => item.name);

		// 2）用户选择自己新下载的模板名称
		const { template } = await inquirer.prompt({
			name: 'template',
			type: 'list',
			choices: templates,
			message: 'Please choose a template to create project',
		});

		// 3）返回用户选择的名称
		return template;
	}

	// 获取用户选择的版本
	// 1）基于 template 结果，远程拉取对应的 tag 列表
	// 2）用户选择自己需要下载的 tag
	// 3) 返回用户选择的 tag

	async getTagVersion(template) {
		// 1）基于 template 结果，远程拉取对应的 tag 列表
		const versions = await wrapLoading(
			getVersionList,
			'waiting fetch tag',
			template
		);
		// 版本号不存在
		if (!versions || versions.length === 0) return;

		// 过滤我们需要的 tag 名称
		const versionList = versions.map((tag) => tag.name);

		// 2）用户选择自己需要下载的 tag
		const { tag } = await inquirer.prompt({
			name: 'tag',
			type: 'list',
			choices: versionList,
			message: 'Place choose a tag to create project',
		});

		// 3）返回用户选择的 tag
		return tag;
	}

	// 下载远程模板
	// 1）拼接远程仓库下载地址
	// 2）调用下载方法拉取远程仓库代码
	async downloadTemplate(templateName, versionNo) {
		// 1）拼接远程仓库下载地址
		// 1.1 拼接远程模板仓库的地址
		let requestUrl = `lyk-cli/${templateName}`;
		// 1.2 拼接远程模板的版本地址
		if (versionNo) {
			requestUrl += `${versionNo ? '#' + versionNo : ''}`;
		}

		// 2）调用下载方法拉取远程仓库代码
		await wrapLoading(
			this.downloadGitRepo, // 远程下载方法
			'waiting download template', // 加载提示信息
			requestUrl, // 参数1: 下载地址
			path.resolve(process.cwd(), this.targetDir) // 参数2: 创建位置
		);
	}

	// 核心创建逻辑
	async create() {
		// 1）获取模板名称
		const templateName = await this.getTemplateName();
		console.log('Template selected:' + templateName);

		// 2) 根据模板名称获取tag版本信息
		const versionNo = await this.getTagVersion(templateName);
		console.log('version selected:' + versionNo);

		// 3) 下载远程模板到本地目录
		await this.downloadTemplate(templateName, versionNo);

		// 4）模板使用提示
		console.log();
		console.log(
			`Successfully created project ${chalk.cyan(this.projectName)}`
		);
		console.log();
		console.log(`  ${chalk.yellow(`cd ${this.projectName}`)}`);
		console.log(`  ${chalk.yellow(`npm run dev`)}`);
		console.log();
	}
}

module.exports = GeneratorProject;
```

到这里一个简单的脚手架就完成了。

执行下 `lyk-cli create my-project-test4`，如图所示就代表下载模板成功了

![下载模板](https://cdn.nlark.com/yuque/0/2023/png/566044/1675302167699-9aaeaead-b074-4418-8f7d-ed7e551a178c.png)

### 5. 发布项目

上面都是在本地测试，实际在使用的时候，可能就需要发布到 npm 仓库，通过 npm 全局安装之后，直接到目标目录下面去创建项目，如何发布呢？

#### 1. 第一步，在 git 上建好仓库

#### 2. 第二步，完善 package.json 中的配置

```js
{
	"name": "lyk-cli",
	"version": "1.0.0",
	"description": "lyk simple cli",
	"main": "index.js",
	"bin": {
		"lyk-cli": "./bin/cli.js"
	},
	"scripts": {
		"test": "echo \"Error: no test specified\" && exit 1"
	},
	"files": [
		"bin",
		"lib"
	],
	"keywords": [
		"lyk-cli",
		"lyk",
		"脚手架"
	],
	"author": {
		"name": "lyk",
		"email": "1210167160@qq.com"
	},
	"license": "MIT",
	"dependencies": {
		"axios": "^1.2.6",
		"chalk": "^4.1.2",
		"commander": "^9.4.0",
		"download-git-repo": "^3.0.2",
		"figlet": "^1.5.2",
		"fs-extra": "^10.1.0",
		"inquirer": "^8.0.0",
		"ora": "^5.4.1"
	}
}

```

#### 3. 第三步，使用 npm publish 进行发布，更新到时候，注意修改版本号

![npm publish](https://cdn.nlark.com/yuque/0/2023/png/566044/1675303722028-1e1d2578-e03f-4672-90ea-249e4141c8c4.png)

出现以上图示，就代表我们发布成功了，我们打开 npm 网站搜下我们发布的包

![npm package](https://cdn.nlark.com/yuque/0/2023/png/566044/1675304691557-521eb1a4-97ca-4f44-8ba6-610090c1829c.png)

已经可以找到它了，这样我们就可以通过 npm 或者 yarn 全局安装使用了

## 4. Result（结果）得到的效果

每次我们去了一个新的项目组，只需要从脚手架中拉取我们自己需要的项目模板就行了，不用手动去复制粘贴之前的项目了，并且删除一些不需要的代码，减少了人力和时间成本，提高了开发效率。

## 5. 方案对比

### 1. Yeoman 一个通用的脚手架系统

### 2. plop 一款小而美的脚手架工具
