# 實戰範例：用 Everything Claude Code 從零建立 SaaS 站台（Next.js）

> 以「任務管理 SaaS（Task Manager）」為具體案例，示範每個開發階段如何運用 everything-claude-code。

---

## 整體流程

```
第一階段：專案初始化（雙代理並行）
第二階段：架構設計（architect 代理）
第三階段：功能開發（技能 + 規則驅動）
第四階段：程式碼審查（reviewer 代理）
第五階段：測試與驗證（e2e-runner 代理）
第六階段：部署與記憶持久化（hooks）
```

---

## 第一階段：專案初始化（雙代理並行）

**終端 A — 鷹架代理**
```bash
claude "建立 Next.js 14 SaaS 專案結構，包含 CLAUDE.md、.claude/rules/、.claude/skills/"
```

**終端 B — 研究代理**
```bash
claude "研究 Clerk（認證）、Supabase（資料庫）、Stripe（付款）整合方式，輸出 integration-plan.md"
```

---

## 第二階段：架構設計

```bash
claude --agent architect "根據 integration-plan.md，設計 Task Manager 架構：Schema、API 路由、元件樹"
```

**Schema 範例：**
```
Users → id, email, name, plan
Teams → id, name, ownerId, memberIds[]
Tasks → id, title, status, assigneeId, teamId, dueDate
Subscriptions → id, userId, stripeId, plan, status
```

---

## 第三階段：功能開發

**.claude/skills/new-feature.md 範本：**
```markdown
# Skill：新功能開發流程
1. 閱讀 architecture.md
2. 新增型別定義（src/types/）
3. 實作 API Route Handler（含 zod 驗證）
4. 實作 React 元件（Server Component 優先）
5. 撰寫單元測試
```

**呼叫技能：**
```bash
/new-feature "實作任務建立功能：POST /api/tasks + TaskCreateForm 元件"
```

---

## 第四階段：程式碼審查

```bash
claude --agent typescript-reviewer "審查 src/app/api/tasks/"
claude --agent security-reviewer "審查整個 src/app/api/"
```

---

## 第五階段：測試

```bash
claude --agent e2e-runner "建立 Playwright 測試：註冊→建立任務→登出、升級付費方案、邀請團隊成員"
node tests/run-all.js
```

---

## 第六階段：Hooks 設定

**PostToolUse Hook（自動格式化）：**
```json
{"hooks": {"PostToolUse": [{"matcher": "Write|Edit", "hooks": [
  {"type": "command", "command": "npx prettier --write $FILE"},
  {"type": "command", "command": "npx tsc --noEmit"}
]}]}}
```

---

## MCP 策略

| 服務 | 策略 | 原因 |
|------|------|------|
| Supabase | 封裝成 Skill | CLI 夠用 |
| GitHub | 封裝成 Command | /pr-review 即可 |
| Vercel | 啟用 MCP | 需即時查詢部署狀態 |

---

## 跨會話記憶

```bash
claude --system-prompt "$(cat .claude/memory.md)" "繼續開發 Task Manager"
```

**memory.md 範本：**
```markdown
## 已完成
- [x] 認證系統（Clerk）
- [x] 任務 CRUD API

## 進行中
- [ ] 團隊協作功能

## 下次繼續
從 src/app/api/teams/invite/route.ts 開始
```

---

## 各工具角色對照

| 工具 | 使用時機 |
|------|----------|
| 雙代理並行 | 專案啟動 |
| architect | 架構設計 |
| Skills | 重複任務標準化 |
| Rules | 品質底線 |
| typescript-reviewer | PR 前型別安全審查 |
| security-reviewer | 上線前漏洞掃描 |
| PostToolUse Hook | 每次存檔自動格式化 |
| memory.md | 跨會話保留上下文 |

*本範例為 everything-claude-code 學習指南實戰補充（繁體中文）*
