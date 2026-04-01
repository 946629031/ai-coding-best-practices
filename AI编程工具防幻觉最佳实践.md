# AI 编程工具防幻觉 & 提升代码质量：完整指南

> 适用工具：Claude Code、GitHub Copilot、Cursor、Codex 等 AI 辅助编程工具
> 
> 核心理念：**减少 AI 的猜测空间，提供足够且精确的上下文，并建立多层级的反馈与验证机制。**

---

## 一、提示词工程（Prompt Engineering）

### 1. 显式授权 "不知道"

在每次任务开始时，明确告诉 AI：

> "如果你对某个 API、类库或具体实现细节不确定，或缺乏足够的项目上下文，请直接说明，**不要尝试猜测或编造**。"

这能从源头掐断大部分强行补全导致的幻觉。

### 2. 强制思维链（Chain of Thought）

要求 AI 在输出代码前先"理清思路"：

> "请先分析现有项目代码的相关逻辑，列出你的实现步骤，最后再输出代码。"

这种方式迫使 AI 必须基于你提供的实际情况进行推理，而不是直接跳到结果。

### 3. 事实锚定

要求 AI 在修改代码前，先从已有的 codebase 中引用它依赖的特定方法或接口定义，确保其所基于的前提条件是真实存在的，没有凭空捏造 API。

### 4. 使用 XML 标签结构化输入

向 AI 提供长文本或多个文件内容时，用标签划清边界，防止指令与参考代码混杂：

```xml
<context>
  <!-- 粘贴相关的已有代码 -->
</context>

<instructions>
  <!-- 你的具体需求 -->
</instructions>

<rules>
  <!-- 你的项目规范 -->
</rules>
```

### 5. Few-Shot 提示（提供示例）

与其长篇大论地解释你想要什么风格的代码，不如直接说：

> "请参考 `src/components/MyExistingForm.tsx` 的结构和风格来实现新的 `UserForm`。"

---

## 二、固定"项目上下文"（核心高级技巧）

这是最有效、最持久的防幻觉手段 —— 给 AI 一个长期上下文文件。

### 上下文文件模板

```markdown
# project-context.md

## 技术栈
- Vue3 + TypeScript + Vite

## 编码规范
- 禁止使用 `any` 类型
- 必须使用 Composition API（禁止 Options API）
- HTTP 请求统一通过 `src/utils/request.ts` 封装的 axios 发起
- 组件文件名使用 PascalCase

## 禁止行为
- 不得引入项目 `package.json` 中不存在的 npm 包
- 不得使用已废弃的 Vue2 API
- 禁止在 `<template>` 中编写复杂业务逻辑

## 目录结构
- `src/api/` - 接口定义
- `src/components/` - 公共组件
- `src/stores/` - Pinia 状态管理
- `src/utils/` - 工具函数

## 常用命令
- 启动开发服务器：`npm run dev`
- 运行测试：`npm run test`
```

### 如何"喂"给各工具

| 工具 | 方法 | 说明 |
|------|------|------|
| **Claude Code** | 项目根目录创建 `CLAUDE.md` | 每次启动时**自动静默读取**，无需手动提醒；运行 `/init` 可自动生成草稿 |
| **Cursor** | 项目根目录创建 `.cursorrules` | Cursor 原生支持，会强制注入为系统提示词 |
| **GitHub Copilot** | 创建 `.github/copilot-instructions.md` | Chat 模式和补全模式均生效 |
| **Antigravity (本 AI)** | 口头说"请先读取 project-context.md" | Agent 会自动调用工具读取文件，或可存入长期知识库 (KI) |
| **直接调用 API** | 将内容放入 `system` 字段 | 每次请求必须手动传入 |

---

## 三、工程化与工作流

### 1. 职责转变：从"作者"变为"审核者"

不要闭着眼睛接受 AI 生成的大段代码。把 AI 当作**初级程序员**，你自己是 **Tech Lead**：

- 重点审查：AI 调用的第三方库方法是否真实存在？
- 重点审查：业务逻辑是否有漏洞或边界条件缺失？
- 而不仅仅是：语法是否正确？

### 2. 任务切片（细化颗粒度）

每次给 AI 分配的任务应该**足够小**，能在一次对话中完整验证：

- ❌ **不好**："帮我实现用户管理模块"
- ✅ **好**："帮我在 `src/api/user.ts` 中新增一个 `getUserById(id: number)` 函数，返回类型参考同文件中的 `getUserList` 函数"

### 3. 多智能体互相审核

如果有条件，可以：
- 一个 AI 负责**编写**代码
- 另一个 AI 扮演"**资深安全工程师**"或"**QA 专家**"角色，专门找前者的问题

---

## 四、技术防线（自动化护栏）

### 1. 强类型 + Linter（最强物理防线）

配合 TypeScript 严格模式 + ESLint，AI 写出的代码如果调用了不存在的属性，IDE 层面立即报错，直接将错误信息抛回给 AI 让其自行纠正（Self-correction）。

```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true
  }
}
```

### 2. 测试驱动（TDD 试错反馈）

每当 AI 完成一个独立的功能块，立即运行单元测试。通过**真实的报错堆栈（Stack Trace）**让 AI "面对现实"，这比口头纠正要有效得多：

> "你刚才生成的代码测试失败，错误如下：`[粘贴报错]`。请根据报错修正代码。"

### 3. 确保 Codebase 索引最新

如果你用的工具提供基于语义的 Codebase Indexing（如 Cursor），确保 Index 是最新的，以便 AI 能搜索到真实的类型定义，而不是凭空想象。

---

## 五、总结

```
防幻觉 = 锁死规范（上下文文件）
       + 强制推理（思维链提示）
       + 任务切片（降低猜测需求）
       + 自动校验（TypeScript + 测试）
```

当以上四层防线同时生效时，AI 就像一位**守规矩、有经验约束**的开发者，而不是一个在猜你想要什么的胡乱发挥者。
