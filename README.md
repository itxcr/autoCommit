1. Eslint 使用

    1. 安装

       ```shell
       # 下载，安装为开发时依赖
       
       npm install eslint --save-dev
       
       # 初始化
       
       npx eslint --init
       ```

       init 完毕之后，在项目的根目录会生成一个 eslint 的配置文件`.eslintrc.{js,yml,json}`

    2. 借助 [eslint-config-standard](https://www.npmjs.com/package/eslint-config-standard)的插件，它是标准的`ESlint`规则， 我们在项目中继承这个标准就可以了

       ```shell
       npm install --save-dev eslint-config-standard eslint-plugin-promise eslint-plugin-import eslint-plugin-node
       ```

       ```json
       // .eslintrc.json
       {
         "extends": "standard"
       }
       ```

       配置文件一旦被扩展，将继承另一份配置文件的所有属性，包括规则、插件、语言解析选项**[Extending Configuration Files](https://eslint.org/docs/user-guide/configuring/configuration-files#extending-configuration-files)**。

2. Prettier 使用

   当使用 Prettier + ESLint 的时候，其实格式问题两个都有参与，disable ESLint 之后，其实格式的问题已经全部由 prettier 接手了，期望报错的来源依旧是 ESLint ，相当于**把 Prettier 推荐的格式配置以 ESLint rules 的方式写入**，这样相当于可以统一代码问题的来源，**[官方的推荐配置](https://github.com/prettier/eslint-plugin-prettier)**

   ```shell
   npm install --save-dev eslint-plugin-prettier
   npm install --save-dev prettier
   # 用 eslint-config-prettier 来关掉 (disable) 所有和 Prettier 冲突的 ESLint 的配置
   npm install --save-dev eslint-config-prettier
   ```

   ```json
   // .eslintrc.json
   {
     "extends": ["plugin:prettier/recommended"]
   }
   ```

   ```json
   // .prettier.json
   {
     "trailingComma": "all",
     "tabWidth": 2,
     "semi": false,
     "singleQuote": false,
     "printWidth": 280,
     "endOfLine": "auto",
     "arrowParens": "always"
   }
   
   ```

3. husky 使用

   通过eslint代码检测的代码依然能被提交到仓库中去。此时需要借助**[husky]( https://github.com/typicode/husky#readme)**来拦截 git 操作，在 git 操作之前再进行一次代码检测

   ```shell
   npm install -D husky
   
   # husky 初始化，创建.husky/目录并指定该目录为git hooks所在的目录
   yarn husky install 
   
   # .husky/目录下会新增pre-commit的shell脚本
   # 在进行 git commit 之前运行 npx eslint src/** 检查
   npx husky add .husky/pre-commit "npx eslint src/**"
   ```

   关于`husky install`官网推荐的是在packgae.json中添加prepare脚本，prepare脚本会在`npm install`（不带参数）之后自动执行

   ```js
   {
     "scripts": {
       "prepare": "husky install"
     }
   }
   ```

   生成的 .husky/pre-commit 文件如下

   ```shell
   #!/bin/sh
   
   . "$(dirname "$0")/_/husky.sh"
   
   npx eslint src/** --fix
   ```

4. lint-staged 使用

   对于单次提交而言，如果每次都检查 src 下的所有文件，可能不是必要的，特别是对于有历史包袱的老项目而言，可能无法一次性修复不符合规则的写法。所以我们需要使用**[lint-staged](https://github.com/okonet/lint-staged)**工具只针对当前修改的部分进行检测。

   ```shell
   npm install --save-dev lint-staged
   ```

   ```json
   // package.json
   {
     "lint-staged": {
       "*.{js,ts,vue}": [
         "npx eslint --fix"
       ]
     }
   }
   ```

   🌰中配置表示的是，对当前改动的 .js 和 .ts文件在提交时进行检测和自动修复，自动修复完成后 lint-staged默认会把改动的文件再次 add 到暂存区，如果有无法修复的错误会报错提示。

   同时还需要改动一下之前的 husky 配置，修改 .husky/pre-commit，在 commit 之前运行`npx lint-staged`来校验提交到暂存区中的文件：

   ```shell
   # pre-commit
   #!/bin/sh
   
   . "$(dirname "$0")/_/husky.sh"
   
   npx lint-staged
   ```

5. 生成符合规范的 commit message

    1. 安装对应包

       ```shell
       yarn add commitizen cz-conventional-changelog -D
       ```

    2. 安装提交检验工具

       ```shell
       yarn add @commitlint/config-conventional @commitlint/cli -D
       
       # 添加 commit-msg 钩子
       yarn husky add .husky/commit-msg 'npx --no -- commitlint --edit "$1"'
       ```

    3. 自定义提交规范

       ```shell
       yarn add commitlint-config-cz  cz-customizable -D
       ```

    4. 创建 `commitlint.config.js` 文件

       ```js
       module.exports = {
         extends: ["@commitlint/config-conventional", "cz"],
       }
       ```

    5. 设置 `commitizen` 配置路径

       ```json
         "config": {
           "commitizen": {
             "path": "node_modules/cz-customizable"
           },
           "cz-customizable": {
             "config": "config/.cz-config.js"
           }
         },
       ```

    6. 创建配置文件夹

       ```js
       // config/.cz-config.js
       module.exports = {
         types: [
           { value: "feat",          name: "✨feat:          增加新功能" },
           { value: "fix",           name: "🐛fix:           修复bug" },
           { value: "docs",          name: "📝docs:          修改文档" },
           { value: "perf",          name: "⚡perf:          性能优化" },
           { value: "init",          name: "🎉init:          初始提交" },
           { value: "add",           name: "➕add:           添加依赖" },
           { value: "build",         name: "🔨build:         打包" },
           { value: "chore",         name: "🔧chore:         更改配置文件" },
           { value: "ci",            name: "👷ci:            CI部署" },
           { value: "del",           name: "🔥del:           删除代码/文件" },
           { value: "refactor",      name: "♻refactor:       代码重构" },
           { value: "revert",        name: "⏪revert:        版本回退" },
           { value: "style",         name: "🍱style:         样式修改不影响逻辑" },
           { value: "test",          name: "✅test:          增删测试" },
         ],
         scopes: [],
         messages: {
           type: "选择一种你的提交类型:\n",
           scope: "更改的范围:\n",
           // 如果allowcustomscopes为true，则使用
           // customScope: 'Denote the SCOPE of this change:',
           subject: "简短描述:\n",
           body: '详细描述. 使用"|"换行:\n',
           breaking: "Breaking Changes列表:\n",
           footer: "关闭的issues列表. E.g.: #31, #34:\n",
           confirmCommit: "确认提交?",
         },
         allowCustomScopes: true,
         allowBreakingChanges: ["feat", "fix"],
       };
       ```

    7. 设置提交脚本

       ```json
       // package.json
       "scripts": {
           "commit": "git add . && git-cz",
         },
       ```

6.  生成版本日志打tag

```shell
yarn add standard-version -D
```

```json
// package.json
"scripts": {
    "release": "standard-version"
  },
```

   

