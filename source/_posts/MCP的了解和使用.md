---
title: MCP的了解和使用
date: 2025-06-18 17:26:49
comments: true
tags:
  - ai
  - mcp
categories:
  - AI
---

# 学习路径

![image.png](https://pic-go-1304657655.cos.ap-guangzhou.myqcloud.com/20250617102616.png)

# 前言

MCP（**Model Context Protocol**，即**模型上下文协议**），主要帮助打通 AI 和不同工具、数据之间的的桥梁。一般的架构就是客户端-服务器这样

**架构：**

- **MCP Hosts**: 如 Claude Desktop、IDE 或 AI 工具，希望通过 MCP 访问数据的程序
- **MCP Clients**: 维护与服务器一对一连接的协议客户端，客户端负责查找资源和工具
- **MCP Servers**: 轻量级程序，通过标准的 Model Context Protocol 提供特定能力，服务端则负责返回客户端需要的信息
- **本地数据源**: MCP 服务器可安全访问的计算机文件、数据库和服务
- **远程服务**: MCP 服务器可连接的互联网上的外部系统（如通过 APIs）

后面要实现的例子就是，基于 cursor 这个 Client 来调用我们的 server

![mcp运作方式](https://linux.do/uploads/default/original/4X/a/b/5/ab5a727d90a12c5331063092a26e1c4a9e88073c.gif)

# MCP 支持的接入协议

Stdio/SSE/Streamable HTTP/同进程调用

#### 标准输入/输出（Stdio）

> 主要是进程通信

Stdio 协议通过标准输入和输出流实现通信，非常适合本地集成和命令行工具开发。
常见的使用场景：

- 构建命令行工具
- 实现本地系统集成
- 简单的进程间通信
- 使用 shell 脚本自动化

### Streamable HTTP

可以替代 SSE

- 可以双向通信

### SSE

> 通过 http 来通信

注：被弃用，改为 Streamable HTTP 了，之前的 SSE 会逐步计划迁移
SSE 传输通过 HTTP POST 请求实现服务器到客户端的流式通信。
常见的使用场景：

- 仅需要 server-to-client 的流式通信
- 在受限网络中工作
- 实现简单更新
- 单向通信

# 使用 MCP-用 cursor 生成旅游规划

### 部分效果图

![image.png](https://pic-go-1304657655.cos.ap-guangzhou.myqcloud.com/20250605191741.png)

### MCP 清单

- [小红书](https://github.com/jobsonlook/xhs-mcp)
- [高德](https://www.modelscope.cn/mcp/servers/@amap/amap-maps)
- [12306](https://www.modelscope.cn/mcp/servers/@Joooook/12306-mcp)

### 如何配置 MCP

#### 小红书

**拿到 web 端的 cookie**
![image.png](https://pic-go-1304657655.cos.ap-guangzhou.myqcloud.com/20250605190525.png)

**配置**

```json
{
  "mcpServers": {
    "xhs-mcp": {
      "command": "uv",
      "args": ["--directory", "/Users/xxx/xhs-mcp", "run", "main.py"],
      "env": {
        "XHS_COOKIE": "xxxx"
      }
    }
  }
}
```

#### 高德

需要拿到高德平台上的 key，具体可以自己查一下文章
**配置**

```json
"amap-amap-sse": {
	"url": "https://mcp.amap.com/sse?key=xxx"
}
```

#### 12306

在 https://www.modelscope.cn/mcp/servers/@Joooook/12306-mcp 拿一个专属的 sse 链接
**配置**

```json
"12306-mcp": {
	"type": "sse",
	"url": "xxx"
}
```

### Prompt

> 来源于 文档 3 的博主分享
> 用 Claude 和 Gemini 做攻略都还可以
> 转 html 的话，Claude 效果比较好

**攻略**

```
你好，我想请你帮我策划一份长沙的旅游攻略。

我的具体需求如下：

预算范围： 人均人民币2000元左右
行程时长： 3天左右
期望出行时间： 5月31日
出发城市： 广州市
主要交通方式： 已购票, 去程xxx动车，返程xxx动车
必去景点/活动： 岳麓山、夜市美食街
兴趣偏好： 希望品尝地道的长沙菜小吃，小众景点
行程风格： 不要安排那么紧凑，希望能轻松游玩，每天景点不超过2-3个
住宿：预算400一间房，离景点不要太远
餐饮偏好： 以当地特色菜为主
时间规划要符合车次，也要考虑刚到长沙时的饮食问题

信息搜集与规划辅助：
请主要参考小红书上的资料，包括攻略内容和评论区。
如果引用了攻略内容，请尽量保留攻略内容的引用来源链接。
请（模拟或建议如何）使用高德地图规划景点之间的浏览路线顺序。
请帮忙查询或预估车票，机票和主要城市间交通的费用。
```

**将攻略转 html**

```
# 旅行规划表设计提示词

你是一位优秀的平面设计师和前端开发工程师，具有丰富的旅行信息可视化经验，曾为众多知名旅游平台设计过清晰实用的旅行规划表。现在需要为我创建一个A4纸张大小的旅行规划表，适合打印出来随身携带使用。请使用HTML、CSS和JavaScript代码实现以下要求：

## 基本要求

**尺寸与基础结构**
- 严格符合A4纸尺寸（210mm×297mm），比例为1:1.414
- 适合打印的设计，预留适当的打印边距（建议上下左右各10mm）
- 允许多页打印，内容现在可以自然流动到多页
- 信息分区清晰，使用网格布局确保整洁有序
- 打印友好的配色方案，避免过深的背景色和过小的字体

**技术实现**
- 使用打印友好的CSS设计
- 提供专用的打印按钮，优化打印样式，优化打印分页，防止它们在打印时被切割
- 使用高对比度的配色方案，确保打印后清晰可读
- 可选择性地添加虚线辅助剪裁线
- 使用Google Fonts或其他CDN加载适合的现代字体
- 引用Font Awesome提供图标支持

**专业设计技巧**

**图形元素与图表：**
1.  **图标 (Font Awesome)：**
    * **来源：** 通过 CDN 引入 Font Awesome (v5/v6)。
    * **风格：** 偏好简洁、现代的**线框风格 (outline-style)** 图标。
    * **使用：** 放置于主标题附近，可选择性地（且需微妙地）用于迷你卡片内部（靠近标题处）、列表前缀等。**严格禁止使用 Emoji 作为功能性图标**。颜色应协调；关键图标可使用高亮色。
2.  **数据可视化 (推荐 Chart.js)：**
    * **应用场景：** 用于展示趋势、增长率、构成（饼图/环形图）、比较（柱状图）等适合的数据 [引用：数据可视化最佳实践]。
    * **技术：** 通过 CDN 嵌入 Chart.js。
    * **位置：** 放置在讨论财务或业务分析的相关主卡片内部。
    * **样式：** 确保图表清晰、易读且响应式。

**技术与动画：**
1.  **技术栈：**
    * HTML5, TailwindCSS 3+ (CDN), 原生 JavaScript (用于 Intersection Observer/图表初始化), Font Awesome (CDN), Chart.js (CDN)。
2.  **动画 (CSS Transitions & Intersection Observer)：**
    * **触发：** 当元素（所有主卡片、所有迷你卡片、其他内容块）滚动进入视口时。
    * **效果：** 平滑、微妙的**淡入/向上滑动**效果（模仿 Apple 风格）。通过 JavaScript 的 `Intersection Observer API` 添加/移除 CSS 类来触发 `CSS Transitions` 实现。确保动画性能流畅。为网格项应用轻微延迟以产生交错效果。
3.  **响应式设计：**
    * **强制要求**。使用 Tailwind 的响应式修饰符（特别是针对网格布局），确保在手机、平板和桌面设备上均具有出色的显示效果和可用性。
    - 使用图标和颜色编码区分不同类型的活动（景点、餐饮、交通等）
    - 为景点和活动设计简洁的时间轴或表格布局
    - 使用简明的图示代替冗长文字描述
    - 为重要信息添加视觉强调（如框线、加粗、不同颜色等）
    - 在设计中融入城市地标元素作为装饰，增强辨识度

## 设计风格

- **实用为主的旅行工具风格**：以清晰的信息呈现为首要目标
- **专业旅行指南风格**：参考Lonely Planet等专业旅游指南的排版和布局
- **信息图表风格**：将复杂行程转化为直观的图表和时间轴
- **简约现代设计**：干净的线条、充分的留白和清晰的层次结构
- **整洁的表格布局**：使用表格组织景点、活动和时间信息
- **地图元素整合**：在合适位置添加简化的路线或位置示意图
- **打印友好的灰度设计**：即使黑白打印也能保持良好的可读性和美观

## 内容区块

1.  **行程标题区**：
    - 目的地名称（主标题，醒目位置）
    - 旅行日期和总天数
    - 旅行者姓名/团队名称（可选）
    - 天气信息摘要
2.  **行程概览区**：
    - 按日期分区的行程简表
    - 每天主要活动/景点的概览
    - 使用图标标识不同类型的活动
3.  **详细时间表区**：
    - 以表格或时间轴形式呈现详细行程
    - 包含时间、地点、活动描述
    - 每个景点的停留时间
    - 标注门票价格和必要预订信息
4.  **交通信息区**：
    - 主要交通换乘点及方式
    - 地铁/公交线路和站点信息
    - 预计交通时间
    - 使用箭头或连线表示行程路线
5.  **住宿与餐饮区**：
    - 酒店/住宿地址和联系方式
    - 入住和退房时间
    - 推荐餐厅列表（标注特色菜和价格区间）
    - 附近便利设施（如超市、药店等）
6.  **实用信息区**：
    - 紧急联系电话
    - 重要提示和注意事项
    - 预算摘要
    - 行李清单提醒

## 示例内容（基于上海一日游）

**目的地**：上海一日游
**日期**：2025年3月30日（星期日）
**天气**：阴，13°C/7°C，东风1-3级

**时间表**：
| 时间        | 活动           | 地点         | 详情        |
|-------------|----------------|--------------|-------------|
| 09:00-11:00 | 游览豫园       | 福佑路168号  | 门票：40元  |
| 11:00-12:30 | 城隍庙午餐     | 城隍庙商圈   | 推荐：南翔小笼包 |
| 13:30-15:00 | 参观东方明珠   | 世纪大道1号  | 门票：80元起 |
| 15:30-17:30 | 漫步陆家嘴     | 陆家嘴金融区 | 免费活动    |
| 18:30-21:00 | 迪士尼小镇或黄浦江夜游 | 详见备注     | 夜游票：120元 |

**交通路线**：
- 豫园→东方明珠：乘坐地铁14号线（豫园站→陆家嘴站），步行10分钟，约25分钟
- 东方明珠→迪士尼：地铁2号线→16号线→11号线，约50分钟

**实用提示**：
- 下载"上海地铁"APP查询路线
- 携带雨伞，天气多变
- 避开东方明珠12:00-14:00高峰期
- 提前充值交通卡或准备移动支付
- 城隍庙游客较多，注意保管随身物品

**重要电话**：
- 旅游咨询：021-12301
- 紧急求助：110（警察）/120（急救）

请创建一个既美观又实用的旅行规划表，适合打印在A4纸上随身携带，帮助用户清晰掌握行程安排。
```

# 实践 MCP：生成接口对接 MCP

> cursor 下执行需要用 Agent 模式

## Server 的核心 MCP 概念

> Cursor 目前只支持 Tools

1. **Resources**：提供给 Clients 读取文件之类的数据
2. **Tools**：LLM 可以调起的功能
3. **Prompts**：预写好的 prompt 模板？

## 前期工作

### 需求

1. 支持分组、url、接口名的查找
2. 缓存全部接口数据，减少 http 请求
3. 用 zod，需要生成让 AI 更好理解的数据格式

### 拆解 Tools

1. 拉取接口平台的数据--这里会返回全部的 json
   1. 洗一遍数据，保留有效字段
   2. 处理参数嵌套情况
2. 查询分组数据
   1. 分组数据返回包含接口列表，便于接口对接
3. 查询接口数据
   1. 支持接口名、uri 查询

## 实践

### 搭建项目

基于 fastmcp 来构建，本地运行的 node 最低版本需要>=20

```shell
mkdir nei-mcp-server
cd nei-mcp-server
# 初始化项目
npm init -y
# 下载fastmcp
npm install fastmcp

# 构建项目
npm build

# 运行项目
npx fastmcp dev src/index.ts
# 浏览器调试tools
npx fastmcp inspect src/index.ts
```

### coding

已完成的项目地址：https://github.com/leila-huang/nei-mcp-server，具体可以看仓库代码

### 调试

##### 在浏览器上用 inspect 调试 tools

使用 `fastmcp` 的 `inspect` 命令可以方便地在本地交互式地测试所有工具。需要在网页上配置上 `SERVER_URL` 和 `PROJECT_ID`。

```bash
npx fastmcp inspect src/index.ts
```

该命令会提供一个交互式界面，你可以在其中选择要执行的工具并输入参数，非常适合用于调试和探索。

#### **mcp 客户端调试**

> 用 cursor 的 mcp 调试，记得切换到 agent 模式

```bash
    {
      "mcpServers": {
          "nei-mcp-server": {
              "command": "npx",
              "args": ["tsx", "/PATH/TO/YOUR_PROJECT/src/index.ts"],
              "env": {
                  "SERVER_URL": "xxx",
                  "PROJECT_ID": "xxx"
              }
          }
      }
    }
```

### 发布

```shell
#登录npm
npm login
#检查登录状态
npm whoami
#发布项目 注：@xx/xx，带有用户名的项目需要跟当前用户名一致，不然会提示403，没有权限。同时需要设置成public
npm publish --access public
```

### 使用

```json
{
  "mcpServers": {
    "nei-mcp-server": {
      "command": "npx",
      "args": ["-y", "@leila329/nei-mcp-server"],
      "env": {
        "SERVER_URL": "xxx",
        "PROJECT_ID": "xxx"
      }
    }
  }
}
```

## 疑问 ❓

### MCP 是怎么对接大模型？

LLM 通过用户给到的信息来推测当前可以利用的 mcp，(可以理解为 tools 的描述越到位越容易命中？)

### cursor 中怎么看 mcp 的日志？

底部的输出栏-选择 mcp-logs
![image.png](https://pic-go-1304657655.cos.ap-guangzhou.myqcloud.com/20250618164152.png)

### 遇到的错误

#### Cannot find module '/Users/leila/.npm/\_npx/16469c90bdfa7901/node_modules/uri-js/dist/es5/uri.all.js'. Please verify that the package.json has a valid "main" entry

> 还挺容易出现类似缓存的问题=-=

清理下 npx 缓存!  
`rm -rf /Users/leila/.npm/_npx`

# 相关文档

1. [从零玩转系列之 MCP Server 理论+项目实战开发你的 MCP Server](https://linux.do/t/topic/648929)
2. [mcp 文档](https://modelcontextprotocol.io/introduction)
3. [使用 MCP 完美规划旅行线路！从零开始的图文教程](https://linux.do/t/topic/656539)
4. [MCP 是怎么对接大模型的？抓取 AI 提示词，拆解 MCP 的底层原理](https://www.youtube.com/watch?v=wiLQgCDzp44&ab_channel=%E6%8A%80%E6%9C%AF%E7%88%AC%E7%88%AC%E8%99%BETechShrimp)
5. [MCP 教程-吴恩达团队](https://www.deeplearning.ai/short-courses/mcp-build-rich-context-ai-apps-with-anthropic/)
