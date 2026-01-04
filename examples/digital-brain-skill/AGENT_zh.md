# 数字大脑 - Claude 指令 (Digital Brain - Claude Instructions)

这是一个“数字大脑”个人操作系统。在此项目中工作时，请遵循以下规则：

## 核心规则

1. **在撰写任何内容之前，务必先阅读 `identity/voice.md`** —— 匹配用户的真实声音。
2. **对 JSONL 文件进行追加，绝不覆盖** —— 保留历史记录。
3. **修改所跟踪的数据时更新时间戳**。
4. **进行跨模块引用** —— 知识库指导内容创作，人脉网络指导业务运营。

## 快速参考

- **撰写内容**：先阅读 `identity/voice.md`，然后使用 `content/templates/` 中的模板。
- **查找联系人**：搜索 `network/contacts.jsonl`，在 `interactions.jsonl` 中查看历史记录。
- **内容灵感**：检查 `content/ideas.jsonl`，运行 `agents/scripts/content_ideas.py`。
- **任务管理**：使用 `operations/todos.md`，并与 `operations/goals.yaml` 保持一致。
- **每周复盘**：运行 `agents/scripts/weekly_review.py`。

## 文件规范

- `.jsonl` 文件：每行一个 JSON 对象，仅限追加。
- `.md` 文件：人类可读，可自由编辑。
- `.yaml` 文件：配置和结构化数据。
- `_template.md` 或 `_schema` 条目：参考格式，请勿修改。

## 当用户要求……

| 请求 | 动作 |
|---------|--------|
| “写一篇关于 X 的帖子” | 阅读 voice.md → 起草 → 匹配声音模式 |
| “为与 Y 的会议做准备” | 查找联系人 → 获取互动记录 → 总结 |
| “我该创作什么？” | 运行 content_ideas.py → 检查日历 |
| “添加联系人 Z” | 按照完整 Schema 追加到 contacts.jsonl |
| “周复盘” | 运行 weekly_review.py → 展示见解 |
