# 参与贡献 LLM-as-a-Judge 技能

感谢你对参与本项目贡献的兴趣！本项目是 [Agent Skills for Context Engineering](https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering) 生态系统的一部分。

## 如何参与贡献

### 报告问题

- 首先检查是否已存在类似的问题。
- 提供清晰的复现步骤。
- 如果适用，请包含测试输出。

### 添加新工具

1. **编写实现代码**：在 `src/tools/<类别>/<工具名>.ts` 中开始：
   - 定义输入/输出的 Zod Schema。
   - 实现带有错误处理的 `execute` 函数。
   - 包含妥善的 TypeScript 类型定义。

2. **从索引中导出**：在 `src/tools/<类别>/index.ts` 中导出。

3. **添加文档**：在 `tools/<类别>/<工具名>.md` 中编写说明：
   - 用途及适用场景。
   - 输入/输出的具体规范。
   - 使用示例。

4. **编写测试**：在 `tests/` 目录下编写测试：
   - 针对 Schema 验证的单元测试。
   - 包含真实 API 调用的集成测试。

### 代码规范

- 在提交代码前运行 `npm run lint` 进行校验。
- 运行 `npm run format` 以确保格式一致。
- 使用 TypeScript 严格模式。
- 为公共 API 添加 JSDoc 注释。

### Pull Request 流程

1. Fork 本仓库。
2. 创建特性分支：`git checkout -b feature/my-feature`。
3. 提交你的更改。
4. 运行测试：`npm test`。
5. 提交代码：`git commit -m 'Add my feature'`。
6. 推送分支：`git push origin feature/my-feature`。
7. 发起 Pull Request。

### 测试指南

- 测试会连接真实的 OpenAI API（需要 API 密钥）。
- 单个 API 调用请设置 `60000ms` 的超时。
- 多个 API 调用请设置 `120000ms` 的超时。
- 尽管 LLM 存在波动，测试仍应保持确定性。

## 开发环境搭建

```bash
# 克隆仓库
git clone https://github.com/muratcankoylan/llm-as-judge-skills.git
cd llm-as-judge-skills

# 安装依赖
npm install

# 配置环境变量
cp env.example .env
# 在 .env 文件中添加你的 OPENAI_API_KEY

# 构建项目
npm run build

# 运行测试
npm test
```

## 有疑问？

请提交 Issue 或通过主仓库联系我们。
 luxury
