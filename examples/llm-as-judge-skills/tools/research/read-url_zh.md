# 读取 URL 工具 (Read URL Tool)

## 用途

从指定的 URL 中提取并解析内容。返回包含源信息的结构化文本内容及元数据。

## 工具定义

```typescript
import { tool } from "ai";
import { z } from "zod";

export const readUrl = tool({
  description: `从 URL 中读取并提取内容。
返回去除了导航栏和广告的核心文本内容。
在 webSearch 之后使用，以便从相关的搜索结果中获取完整内容。`,

  parameters: z.object({
    url: z.string().url()
      .describe("要读取的 URL"),
    
    contentType: z.enum(["auto (自动)", "article (文章)", "documentation (文档)", "paper (论文)", "code (代码)"]).default("auto")
      .describe("内容类型提示，用于优化提取效果"),
    
    maxLength: z.number().min(1000).max(50000).default(10000)
      .describe("返回的最大字符数"),
    
    extractSections: z.boolean().default(true)
      .describe("是否识别并标记章节 (Sections)"),
    
    includeMetadata: z.boolean().default(true)
      .describe("是否包含作者、日期等元数据")
  }),

  execute: async (input) => {
    return extractUrlContent(input);
  }
});
```

## 输入 Schema

| 字段 | 类型 | 必需 | 描述 |
|-------|------|----------|-------------|
| url | string | 是 | 要读取的 URL |
| contentType | enum | 否 | 内容类型提示 |
| maxLength | number | 否 | 最大字符数 (默认: 10000) |
| extractSections | boolean | 否 | 标记章节 |
| includeMetadata | boolean | 否 | 包含元数据 |

## 输出 Schema

```typescript
interface ReadUrlResult {
  success: boolean;
  
  url: string;
  title: string;
  
  content: {
    full: string;         // 全文
    sections?: {          // 章节
      heading: string;    // 标题
      level: number;      // 层级：h1=1, h2=2, 等
      content: string;    // 内容
    }[];
  };
  
  metadata?: {
    author?: string;      // 作者
    publishedDate?: string; // 发布日期
    lastModified?: string;  // 最近修改
    description?: string;   // 描述
    keywords?: string[];    // 关键词
    source: string;         // 来源
  };
  
  stats: {
    totalCharacters: number; // 总字符数
    truncated: boolean;      // 是否被截断
    sectionsFound: number;   // 找到的章节数
  };
  
  error?: {
    code: string;
    message: string;
  };
}
```

## 使用示例

```typescript
const content = await readUrl.execute({
  url: "https://eugeneyan.com/writing/llm-evaluators/",
  contentType: "article",
  maxLength: 15000,
  extractSections: true,
  includeMetadata: true
});

// 结果示例:
// {
//   success: true,
//   url: "https://eugeneyan.com/writing/llm-evaluators/",
//   title: "Evaluating the Effectiveness of LLM-Evaluators",
//   content: {
//     full: "LLM-evaluators, also known as LLM-as-a-Judge...",
//     sections: [
//       {
//         heading: "Key considerations before adopting an LLM-evaluator",
//         level: 2,
//         content: "Before reviewing the literature..."
//       },
//       ...
//     ]
//   },
//   metadata: {
//     author: "Eugene Yan",
//     publishedDate: "2024-06-15",
//     source: "eugeneyan.com"
//   },
//   stats: {
//     totalCharacters: 15000,
//     truncated: true,
//     sectionsFound: 8
//   }
// }
```

## 内容类型处理

| 类型 | 优化方案 |
|------|-------------|
| 文章 (article) | 优先提取正文，跳过侧边栏 |
| 文档 (documentation) | 保留代码块，维护结构 |
| 论文 (paper) | 提取摘要、章节、参考文献 |
| 代码 (code) | 保留格式，语法高亮 |
| 自动 (auto) | 从内容中自动检测类型 |

## 错误处理

```typescript
const errorCodes = {
  "URL_NOT_FOUND": "页面不存在 (404)",
  "ACCESS_DENIED": "页面需要身份验证 (401/403)",
  "TIMEOUT": "请求超时",
  "BLOCKED": "访问被 robots.txt 或频率限制所阻止",
  "INVALID_CONTENT": "内容无法被解析",
  "UNSUPPORTED_TYPE": "内容类型不受支持 (例如二进制文件)"
};
```

## 实现说明

1. **遵守 robots.txt**：检查并遵守 robots.txt 协议中的指令。
2. **速率限制**：避免对同一域名进行极高频率的连续请求。
3. **User Agent**：使用适当的 User Agent 标识。
4. **超时设置**：设置合理的超时时间 (10-30s)。
5. **JavaScript 渲染**：对于由 JS 驱动渲染的站点，考虑使用 Headless 浏览器。
6. **缓存**：针对重复读取请求缓存网页内容。
