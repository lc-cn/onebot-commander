# OneBot Commander

[![npm version](https://img.shields.io/npm/v/onebot-commander.svg)](https://www.npmjs.com/package/onebot-commander)
[![npm downloads](https://img.shields.io/npm/dm/onebot-commander.svg)](https://www.npmjs.com/package/onebot-commander)
[![License](https://img.shields.io/npm/l/onebot-commander.svg)](https://github.com/your-username/onebot-commander/blob/main/LICENSE)
[![Test Coverage](https://img.shields.io/badge/coverage-90%25-brightgreen.svg)](https://github.com/your-username/onebot-commander)
[![Node.js Version](https://img.shields.io/node/v/onebot-commander.svg)](https://nodejs.org/)

OneBot12 消息段命令解析器 - TypeScript 版本，支持 ESM/CJS 双格式

## 📖 文档

- [📚 完整文档](https://onebot-commander.pages.dev/) - 详细的 API 文档和使用指南
- [🚀 快速开始](/docs/guide/) - 5分钟快速上手
- [💡 使用示例](/docs/examples/) - 丰富的代码示例
- [🔄 迁移指南](/docs/migration/) - 从其他库迁移
- [🤝 贡献指南](/docs/contributing/) - 参与项目开发

### 🎨 新特性文档

- [🎯 特殊类型规则](/docs/guide/special-type-rules.md) - 自动类型转换详解
- [📝 可选参数](/docs/guide/optional-parameters.md) - 可选参数和默认值使用
- [🔄 动态字段映射](/docs/guide/dynamic-field-mapping.md) - 自定义字段映射配置
- [🏗️ TypeMatcher API](/docs/api/type-matchers.md) - TypeMatcher 系统 API 参考

## ✨ 特性

- 🎯 **精确匹配**: 支持复杂的消息段模式匹配
- ⚡ **高性能**: 基于优化的匹配算法，性能优异
- 🔧 **灵活配置**: 支持自定义类型化字面量字段映射
- 🛡️ **类型安全**: 完整的 TypeScript 支持
- 🔗 **链式调用**: 优雅的 API 设计
- 📦 **双格式**: 同时支持 ESM 和 CommonJS
- 🧪 **测试完善**: 90%+ 测试覆盖率
- 🎨 **特殊类型规则**: 自动类型转换（number, integer, float, boolean）
- 📝 **可选参数**: 支持带默认值的可选参数 `[param:type=default]`
- 🔄 **动态字段映射**: 智能字段映射，支持多平台适配

## 🚀 快速开始

### 安装

```bash
npm install onebot-commander
```

### 基础用法

```typescript
import { Commander } from 'onebot-commander';

// 创建命令解析器（注意空格敏感）
const commander = new Commander('hello <name:text>'); // "hello " 后面的空格

// 添加处理逻辑
commander
  .action((params) => {
    console.log(`Hello, ${params.name}!`);
    return params.name.toUpperCase();
  })
  .action((upperName) => {
    console.log(`Uppercase: ${upperName}`);
  });

// 匹配消息段
const segments = [
  { type: 'text', data: { text: 'hello Alice' } } // 注意 "hello " 后面的空格
];

const result = commander.match(segments);
// 输出: Hello, Alice!
// 输出: Uppercase: ALICE
```

### 🎨 新特性速览

#### 特殊类型规则

支持自动类型转换，无需手动解析：

```typescript
import { Commander } from 'onebot-commander';

// 数字类型自动转换
const ageCmd = new Commander('设置年龄 <age:number>');
ageCmd.action((params) => {
  console.log(`年龄: ${params.age} (类型: ${typeof params.age})`);
});

// 整数类型（只接受整数）
const countCmd = new Commander('重复 <times:integer> 次');

// 浮点数类型（必须包含小数点）
const rateCmd = new Commander('设置比例 <rate:float>');

// 布尔类型自动转换
const enableCmd = new Commander('启用功能 <enabled:boolean>');
enableCmd.action((params) => {
  console.log(`功能状态: ${params.enabled} (类型: ${typeof params.enabled})`);
});

// 示例匹配
ageCmd.match([{ type: 'text', data: { text: '设置年龄 25' } }]);
// 输出: 年龄: 25 (类型: number)

enableCmd.match([{ type: 'text', data: { text: '启用功能 true' } }]);
// 输出: 功能状态: true (类型: boolean)
```

#### 可选参数和默认值

```typescript
// 可选参数带默认值
const greetCmd = new Commander('你好 [name:text=世界]');
greetCmd.action((params) => {
  console.log(`Hello, ${params.name}!`);
});

// 数字类型的可选参数
const configCmd = new Commander('配置 [timeout:number=30] [retries:integer=3]');

// 示例匹配
greetCmd.match([{ type: 'text', data: { text: '你好 ' } }]);
// 输出: Hello, 世界!

greetCmd.match([{ type: 'text', data: { text: '你好 张三' } }]);
// 输出: Hello, 张三!
```

#### 动态字段映射

支持自定义消息段字段映射，适配不同平台：

```typescript
// 自定义字段映射
const customCmd = new Commander('发送图片 <img:image>', {
  image: 'src'  // 使用 'src' 字段而不是默认的 'file' 或 'url'
});

// 多字段优先级映射
const multiCmd = new Commander('头像 <avatar:image>', {
  image: ['primary', 'secondary', 'file']  // 按优先级尝试
});

// 示例匹配
customCmd.match([
  { type: 'text', data: { text: '发送图片 ' } },
  { type: 'image', data: { src: 'photo.jpg' } }  // 使用自定义字段
]);
```

### ⚠️ 空格敏感特性

OneBot Commander 对空格非常敏感，这是确保命令精确匹配的重要特性：

```typescript
// 模式: "ping [count:number={value:1}]"
const commander = new Commander('ping [count:number={value:1}]'); // "ping " 后面的空格

commander.action((params) => {
  const count = params.count || { value: 1 };
  return `Pong! (${count.value} times)`;
});

// ✅ 用户输入 "ping " - 匹配成功
const segments1 = [{ type: 'text', data: { text: 'ping ' } }];
const result1 = commander.match(segments1); // ['Pong! (1 times)']

// ❌ 用户输入 "ping" - 匹配失败
const segments2 = [{ type: 'text', data: { text: 'ping' } }];
const result2 = commander.match(segments2); // []
```

## 📚 文档站

本项目包含完整的文档站，使用 VitePress 构建：

### 本地开发

```bash
# 启动开发服务器
npm run docs:dev

# 构建文档
npm run docs:build

# 预览构建结果
npm run docs:preview
```

### 部署

```bash
# 部署到 GitHub Pages
npm run docs:deploy:github

# 部署到 Netlify
npm run docs:deploy:netlify

# 部署到 Vercel
npm run docs:deploy:vercel

# 生成部署配置
npm run docs:config
```

访问 http://localhost:5173 查看本地文档。

## 🧪 测试

```bash
# 运行测试
npm test

# 运行测试并生成覆盖率报告
npm run test:coverage

# 运行基准测试
npm run benchmark
```

## 📦 构建

```bash
# 构建项目
npm run build

# 清理构建产物
npm run clean
```

## 🤝 贡献

欢迎贡献！请查看 [贡献指南](/docs/contributing/) 了解详情。

## 📄 许可证

MIT License - 查看 [LICENSE](LICENSE) 文件了解详情。

## 🔗 相关链接

- [OneBot 官网](https://onebot.dev/)
- [OneBot12 规范](https://12.onebot.dev/)
- [GitHub 仓库](https://github.com/lc-cn/onebot-commander)
- [npm 包](https://www.npmjs.com/package/onebot-commander)
- [在线文档](https://onebot-commander.pages.dev/) 