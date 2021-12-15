github地址：https://github.com/typicode/husky

# 软件简介

husky 是一个 Git Hook 工具。

husky 可以防止使用 Git hooks 的一些不好的 commit 或者 push。

安装：

```
npm install husky --save-dev
```

代码示例：

```
// package.json
{
  "scripts": {
    "precommit": "npm test",
    "prepush": "npm test",
    "...": "..."
  }
}
```

# 实现提交前 eslint 校验和 commit 信息的规范校验

原文地址：https://blog.csdn.net/huangpb123/article/details/102690412

## 一、配置步骤

1. 安装 husky，lint-staged，@commitlint/cli，@commitlint/config-conventional 依赖

- lint-staged: 用于实现每次提交只检查本次提交所修改的文件。

```js
npm i -D husky lint-staged @commitlint/cli @commitlint/config-conventional
```

    注意：一定要使用 npm 安装 eslint 和 husky，因为在 windows 操作系统下, 用 yarn 安装依赖，不会触发 husky pre-commit 钩子命令。

2. 创建 .huskyrc


```yaml
{
  "hooks": {
    "pre-commit": "lint-staged",
    "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
  }
}
```

2. 创建 .lintstagedrc

```yaml
{
  "src/**/*.js": "eslint"
}
```

设置 fix 可以自动修复错误：

```yaml
{
   "src/**/*.js": ["eslint --fix", "git add"]
}
```

或者使用下面的配置，自动格式化代码（谨慎使用）：

```yaml
{
   "src/**/*.js": ["prettier --write", "git add"]
}
```

4. 创建 commitlint.config.js

```js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
        2,
        'always',
        [
        'feat', // 新功能（feature）
        'fix', // 修补bug
        'docs', // 文档（documentation）
        'style', // 格式（不影响代码运行的变动）
        'refactor', // 重构（即不是新增功能，也不是修改bug的代码变动）
        'test', // 增加测试
        'revert', // 回滚
        'config', // 构建过程或辅助工具的变动
        'chore', // 其他改动
        ],
    ],
    'type-empty': [2, 'never'], // 提交不符合规范时,也可以提交,但是会有警告
    'subject-empty': [2, 'never'], // 提交不符合规范时,也可以提交,但是会有警告
    'subject-full-stop': [0, 'never'],
    'subject-case': [0, 'never'],
  }
}
```

## 二、配置不拆分成多个文件的话也可以全部写在 package.json 里面

```yaml
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "src/**/*.js": "prettier --write --ignore-unknown"
  }
}
```
