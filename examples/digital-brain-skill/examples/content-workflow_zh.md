# 示例：内容创作工作流 (Content Creation Workflow)

使用“数字大脑”进行内容创作的完整演示。

---

## 场景设定

您想要写一个关于“公开建设 (building in public) 的教训”的 Twitter 线程 (Thread)。

---

## 第一步：智能体激活技能

**用户输入**：“帮我写一个关于公开建设教训的推特线程。”

**智能体识别**：
- 触发词：“写一个线程” → 内容创作。
- 话题：“公开建设” → 匹配品牌支柱。
- 动作：加载“数字大脑”技能。

---

## 第二步：加载身份信息（必选）

智能体阅读 `identity/voice.md`：

```yaml
# 提取的声音模式：
正式/随意程度：7/10 (相当随意)
标志性短语：
  - "Here's the thing" (事实是)
  - "Let me break it down" (让我拆解一下)
核心动力词：
  - "ship" (交付), "iterate" (迭代), "leverage" (杠杆)
禁词/禁忌：
  - “synergy” (协同效应), “circle back” (后续跟进)
  - 过多的表情符号
```

智能体阅读 `identity/brand.md`：

```yaml
# 确认话题契合度：
内容支柱：
  - building_in_public (公开建设)  ✓ 匹配
  - ai_agents (AI 智能体)
  - founder_lessons (创始人教训)
目标受众：“早期阶段的技术创始人”
```

---

## 第三步：参考历史成功案例

智能体扫描 `content/posts.jsonl`：

```json
// 找到一个类似的高质量线程：
{
  "type": "thread",
  "pillar": "building_in_public",
  "metrics": {"likes": 234, "reposts": 45},
  "notes": "故事格式效果很好"
}
```

**洞察**：对于该支柱，故事驱动型的线程表现最佳。

---

## 第四步：检查灵感库

智能体扫描 `content/ideas.jsonl`：

```json
// 找到一个相关的、尚未开发的灵感：
{
  "idea": "关于公开建设过程中心理波动的线程",
  "status": "raw",
  "priority": "high"
}
```

**决策**：可以将这个角度融入线程中。

---

## 第五步：生成草稿

智能体使用 `content/templates/thread.md` 作为脚手架：

```markdown
## 1/ 钩子
反直觉观点：公开建设并不全是为了展示胜利。

事实是 (Here's the thing) —— 它更多关乎那些混乱的中段。

让我来拆解一下 (Let me break it down) 🧵

## 2/ 背景
我已经进行公开建设 18 个月了。

这是我希望在第一天就有人告诉我的事：

## 3-7/ 主要观点
[包含 5 个具体教训及其背景故事]

## 8/ 总结要点
透明度的复利效应胜过任何营销策略。

## 9/ 行动号召 (CTA)
关注我 @账户名，我将交付 (ship) 更多创始人教训。
```

**声音模式对齐检查**：
- ✓ 使用了 "Here's the thing" (标志性短语)
- ✓ 使用了 "Let me break it down" (标志性短语)
- ✓ 语气随意但具实质内容 (7/10)
- ✓ 无公司化陈辞滥调
- ⚠️ 少量表情符号 (一个 🧵 —— 在可接受范围内)

---

## 第六步：用户反馈与迭代

用户：“能把第 3 点改为关于‘害怕被评判’的内容吗？”

智能体：
1. 保持相同的声音/结构。
2. 以“恐惧”角度重写第 3 点。
3. 维护线程的整体连贯性。

---

## 第七步：记录灵感（若不立即发布）

如果用户想要存为稍后发布，智能体会将其追加到 `content/ideas.jsonl`：

```json
{
  "id": "idea_20241229_160000",
  "created": "2024-12-29T16:00:00Z",
  "idea": "线程：公开建设的 5 个教训（恐惧、胜利、社区……）",
  "source": "developed_draft (已开发草稿)",
  "pillar": "building_in_public",
  "status": "ready (就绪)",
  "priority": "high",
  "notes": "草稿已完成，已审核声音对齐情况",
  "tags": ["thread", "building_in_public", "founder_lessons"]
}
```

---

## 第八步：发布后记录

在用户发布后，智能体将其追加到 `content/posts.jsonl`：

```json
{
  "id": "post_20241229_180000",
  "published": "2024-12-29T18:00:00Z",
  "platform": "twitter",
  "type": "thread",
  "content": "反直觉观点：公开建设并不全是为了展示胜利……",
  "url": "https://twitter.com/user/status/123456789",
  "pillar": "building_in_public",
  "metrics": {
    "impressions": 0,
    "likes": 0,
    "comments": 0,
    "reposts": 0
  },
  "metrics_updated": "2024-12-29T18:00:00Z",
  "notes": "故事驱动格式，起草时的‘恐惧’角度引起了共鸣",
  "tags": ["thread", "building_in_public"]
}
```

---

## 涉及文件

| 文件 | 用途 | Token 消耗 (约) |
|------|---------|------------|
| `SKILL.md` | 路由 | 50 |
| `identity/voice.md` | 声音模式 | 200 |
| `identity/brand.md` | 话题验证 | 150 |
| `content/posts.jsonl` | 历史表现 | 100 |
| `content/ideas.jsonl` | 现有灵感 | 50 |
| `content/templates/thread.md` | 结构 | 100 |

**总计**：约 650 tokens，对比加载整个“大脑”数据（约 5000 tokens）具有显著优势。

---

## 核心收获

1. **声音先行**：在起草前始终预先加载。
2. **渐进式加载**：仅访问相关的模块。
3. **模式匹配**：过去的成功案例指导新内容的创作。
4. **全链路流水线**：灵感 → 草稿 → 发布 → 记录。
5. **仅限追加**：灵感和帖子会被记录，绝不删除。
