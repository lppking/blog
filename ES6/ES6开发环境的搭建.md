## ES6开发环境的搭建
首先，为什么ES6需要一个开发环境，而不是像ES5那样直接在浏览器打开就可以？原因就是到目前为止，ES6的某些新特性还不能被主流浏览器普遍支持。需要把ES6编译为ES5的代码才能保证代码的可用性。接下来是搭建编译环境的基本步骤：
#### 初始化项目
新建一个文件夹，在文件夹中运行`npm init -y`生成package.json文件。
#### 2. 安装编译工具
Babel官方不建议全局安装babel-cli，因为考虑到以后有可能会涉及到不同项目不同babel版本的问题。
通过运行如下命令安装编译环境: 
```
npm install --save-dev babel-cli
npm install --save-dev babel-preset-es2015
```
#### 3. 配置.babelrc
使用babelrc有两种方式，一种是在package.json中进行配置，另一种是建立一个.babelrc文件。
babel会优先寻找.babelrc，如果找不到，才会去package.json中寻找babel配置项。
.babelrc的基本配置如下：
```
{
  "presets": [
    "es2015"
  ],
  "plugins": []
}
```
更多的配置项可以查阅官方文档。
#### 4. 编译文件
完成上述步骤，我们就可以编译ES6语法的文件了。这里有一个需要注意的点，如果你是全局安装的babel-cli，那么你可以直接运行：
```
babel src/index.js -o dist/index.js
``` 
如果你只是在项目中安装的babel-cli，那么需要在命令前面加一个npx，如下所示：
```
npx babel src/index.js -o dist/index.js
```
#### 5. 设置编译脚本
在package.json中的scripts项中可以设置快捷脚本。scripts如下所示：
```
"scripts": {
    "babel": "npx babel src/index.js -o dist/index.js"
}
```
babel是脚本名称，设置如上命令之后，我们就可以直接在命令行输入`npm run babel`，就相当于运行了`npx babel src/index.js -o dist/index.js`命令。