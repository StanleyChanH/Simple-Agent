<div align="center">

<img src="banner.png" alt="Simple-Agent" width="100%"/>

**用 ~700 行代码实现现代 AI Agent 核心功能的教学项目**

</div>

一个极简但功能完整的 AI 智能体框架，复现了 Claude Code、Cursor 等现代 Agent 的核心机制。

## 核心特性

| 功能模块 | 说明 |
|---------|------|
| **工具调用** | Bash命令、文件读写编辑、路径安全检查 |
| **任务系统** | 持久化任务管理、依赖追踪、状态机 |
| **子智能体** | 委托式探索/执行，上下文隔离 |
| **上下文压缩** | 自动/手动压缩，保留关键信息 |
| **多智能体协作** | 消息总线、广播、队友生成 |
| **技能系统** | 动态加载 Markdown 格式的专业知识 |
| **后台任务** | 异步执行、通知队列 |
| **Todo管理** | 进度跟踪、状态可视化 |

## 架构图

```
┌─────────────────────────────────────────────────────────┐
│                    Lead Agent (主控)                      │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐       │
│  │  Tools  │ │  Tasks  │ │  Skills │ │   Todo  │       │
│  └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘       │
└───────┼──────────┼──────────┼──────────┼───────────────┘
        │          │          │          │
        ▼          ▼          ▼          ▼
┌─────────────────────────────────────────────────────────┐
│                    Message Bus                           │
└───────────────────────┬─────────────────────────────────┘
                        │
        ┌───────────────┼───────────────┐
        ▼               ▼               ▼
   ┌─────────┐    ┌─────────┐    ┌─────────┐
   │ Agent A │    │ Agent B │    │ Agent C │
   │ (Worker)│    │ (Explore)│   │ (Write) │
   └─────────┘    └─────────┘    └─────────┘
```

## 快速开始

### 1. 安装依赖

```bash
uv sync
```

### 2. 配置环境变量

```bash
cp .env.example .env
# 编辑 .env 填入你的 API Key
```

支持任何兼容 OpenAI API 的服务：
- OpenAI: `OPENAI_BASE_URL=https://api.openai.com/v1`
- 阿里云 Qwen: `OPENAI_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1`
- DeepSeek: `OPENAI_BASE_URL=https://api.deepseek.com/v1`
- 本地模型: `OPENAI_BASE_URL=http://localhost:11434/v1`

### 3. 运行

```bash
uv run main.py
```

## 交互命令

| 命令 | 说明 |
|-----|------|
| `/compact` | 手动压缩对话上下文 |
| `/tasks` | 查看所有任务 |
| `/team` | 查看队友状态 |
| `/inbox` | 读取收件箱消息 |
| `q` / `exit` | 退出程序 |

## 可用工具

智能体可以使用以下工具：

```python
# 文件操作
bash(command)           # 执行 shell 命令
read_file(path)         # 读取文件
write_file(path, content)  # 写入文件
edit_file(path, old, new)  # 编辑文件

# 任务管理
task_create(subject, description)  # 创建任务
task_get(task_id)                  # 获取任务
task_update(task_id, status, ...)  # 更新任务
task_list()                        # 列出任务

# 子智能体
task(prompt, agent_type="Explore")  # 委托子任务

# 多智能体
spawn_teammate(name, role, prompt)  # 创建队友
send_message(to, content)           # 发送消息
broadcast(content)                  # 广播消息
shutdown_request(teammate)          # 请求关闭

# 其他
TodoWrite(items)        # 更新待办
load_skill(name)        # 加载技能
background_run(command) # 后台任务
```

## 代码结构

```
main.py (~700 行)
├── 基础工具 (safe_path, run_bash, run_read, run_write, run_edit)
├── TodoManager          # 待办事项管理
├── SkillLoader          # 技能加载器
├── TaskManager          # 持久化任务管理
├── BackgroundManager    # 后台任务管理
├── MessageBus           # 消息总线
├── TeammateManager      # 多智能体协作
├── 上下文压缩 (microcompact, auto_compact)
├── 子智能体 (run_subagent)
├── 工具定义 (TOOLS, TOOL_HANDLERS)
└── 主循环 (agent_loop)
```

## 核心设计

### 1. 上下文压缩策略

```python
# 微压缩: 清除旧的 tool 结果
def microcompact(messages):
    for msg in tool_messages[:-3]:
        if len(msg["content"]) > 100:
            msg["content"] = "[已清除]"

# 自动压缩: 超过阈值时用 LLM 摘要
def auto_compact(messages):
    summary = llm.summarize(messages)
    return [summary_message]
```

### 2. 多智能体通信

```python
# 消息格式
{
    "type": "message|broadcast|shutdown_request",
    "from": "sender_name",
    "content": "...",
    "timestamp": 1234567890
}

# 基于文件的收件箱
INBOX_DIR / "{agent_name}.jsonl"
```

### 3. 工具分发

```python
TOOL_HANDLERS = {
    "bash": run_bash,
    "read_file": run_read,
    # ...
}

# 动态调用
handler = TOOL_HANDLERS.get(tool_name)
output = handler(**args)
```

## 扩展技能

在 `skills/` 目录下创建 `SKILL.md`:

```markdown
---
name: my-skill
description: 我的自定义技能
---

## 技能内容
这里是具体的指令和知识...
```

## 教学目的

这个项目展示了现代 AI Agent 的核心概念：

1. **ReAct 模式** - 推理 + 行动循环
2. **工具调用** - Function Calling 机制
3. **上下文管理** - 长对话的压缩策略
4. **多智能体** - 协作与通信模式
5. **持久化** - 任务和状态的保存

## 对比参考

| 特性 | Simple-Agent | Claude Code |
|-----|-------------|-------------|
| 代码行数 | ~700 | ~10,000+ |
| 工具数量 | 20+ | 60+ |
| 多智能体 | ✅ | ✅ |
| 上下文压缩 | ✅ | ✅ |
| 任务持久化 | ✅ | ✅ |

## License

MIT
