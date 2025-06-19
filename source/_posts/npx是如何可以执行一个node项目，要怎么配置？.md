---
title: npx是如何可以执行一个node项目，要怎么配置？
date: 2025-06-19 9:30:17
tags:
  - Node.js
  - npx
  - npm
  - CLI
  - 基础
categories:
  - 基建
---

## 如何执行？

npx 查找命令的优先级：**项目本地的 node_modules > 全局缓存 > npm 仓库**
注：从 npm 仓库拉下来的包，执行完后会直接删除

## 如何配置？

### 创建一个可执行文件

通常`build/index.js`是 build 后生成的文件，因此，这里我们可以直接创建这个`src/index.ts`，在文件的顶部添加上`#!/usr/bin/env node`。这行代码告诉操作系统应该用哪个解释器来运行这个脚本
`src/index.ts`

```
#!/usr/bin/env node

console.log('你好，这是一个可以通过 npx 执行的项目！👋');
// 在这里编写你的项目核心逻辑
const args = process.argv.slice(2);
console.log('你传入的参数是:', args);
```

### 配置 package.json 的 bin 字段

    - 作用：将一个或多个命令名称映射到你项目中的可执行文件
    - 示例：

```
	"bin": {
    "nei-mcp-server": "build/index.js"
  },
```

### 调试

- 本地调试：`npm link` 可以指向当前项目。然后就可以在终端执行 `npx nei-mcp-server`
- 发布 npm：发布上去后，项目依赖这个 nei-mcp-server 后，也可以在终端执行 `npx nei-mcp-server`

## 小结

1. 需要 npx 执行的脚本，要记得加上标识
2. link 调试结束后，记得 unlink 掉
