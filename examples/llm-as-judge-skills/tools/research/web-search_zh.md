# 网页搜索工具 (Web Search Tool)

## 用途

在网络上针对给定话题搜索相关信息。返回包含摘要、URL 及元数据的结构化结果。

## 工具定义

```typescript
import { tool } from "ai";
import { z } from "zod";

export const webSearch = tool({
  description: `在网络上搜索话题相关信息。
返回包含摘要和 URL 的相关结果。
用于收集最新信息、核实事实或进行研究。`,

  parameters: z.object({
    query: z.string()
      .describe("搜索查询词 —— 越具体效果越好"),
    
    maxResults: z.number().min(1).max(20).default(10)
      .describe("返回结果的最大数量"),
    
    filters: z.object({
      dateRange: z.enum(["day (天)", "week (周)", "month (月)", "year (年)", "any (任何)"]).default("any")
        .describe("将结果限制在特定时间段内"),
      
      sourceType: z.enum(["all (全部)", "news (新闻)", "academic (学术)", "documentation (文档)"]).default("all")
        .describe("优先考虑的资源类型"),
      
      excludeDomains: z.array(z.string()).optional()
        .describe("要从结果中排除的域名列表")
    }).optional()
  }),

  execute: async (input) => {
    return performWebSearch(input);
  }
});
```

## 输入 Schema

| 字段 | 类型 | 必需 | 描述 |
|-------|------|----------|-------------|
| query | string | 是 | 搜索查询词 |
| maxResults | number | 否 | 最大结果数 (默认: 10) |
| filters.dateRange | enum | 否 | 时间段过滤器 |
| filters.sourceType | enum | 否 | 来源类型优先级 |
| filters.excludeDomains | string[] | 否 | 排除的域名列表 |

## 输出 Schema

```typescript
interface WebSearchResult {
  success: boolean;
  
  results: {
    title: string;       // 标题
    url: string;         // URL
    snippet: string;     // 摘要
    source: string;      // 来源域名
    publishedDate?: string;
    relevanceScore: number; // 相关性评分
  }[];
  
  totalResults: number;
  
  metadata: {
    query: string;
    searchTimeMs: number;
    filtersApplied: string[]; // 已应用的过滤器
  };
}
```

## 使用示例

```typescript
const results = await webSearch.execute({
  query: "LLM-as-a-Judge 评估方法 2024",
  maxResults: 10,
  filters: {
    dateRange: "year",
    sourceType: "academic"
  }
});

// 结果示例:
// {
//   success: true,
//   results: [
//     {
//       title: "Judging LLM-as-a-Judge with MT-Bench",
//       url: "https://arxiv.org/abs/...",
//       snippet: "我们研究了 LLM-as-a-Judge 的有效性……",
//       source: "arxiv.org",
//       publishedDate: "2024-01-15",
//       relevanceScore: 0.95
//     },
//     ...
//   ],
//   totalResults: 10,
//   metadata: {
//     query: "LLM-as-a-Judge 评估方法 2024",
//     searchTimeMs: 342,
//     filtersApplied: ["dateRange:year", "sourceType:academic"]
//   }
// }
```

## 查询优化技巧

1. **术语准确**：使用精确的专业术语。
2. **引号匹配**：对于精确短语，请使用引号。
3. **搜索操作符**：支持 site:、-关键词 (排除项) 或 OR。
4. **上下文信息**：在查询词中包含相关的背景词汇。
5. **时效性**：针对最近的信息，添加年份关键词。

## 实现说明

1. **速率限制**：实施适当的 API 请求速率限制。
2. **缓存机制**：针对重复查询缓存搜索结果。
3. **结果质量**：过滤掉低质量的信息来源。
4. **错误处理**：得体处理 API 调用失败的情况。
5. **隐保护隐私**：妥善记录查询日志。
