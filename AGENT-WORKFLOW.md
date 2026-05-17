# EpicArena 多智能体工作流 · 职责边界

> 版本：V0.6 | 日期：2026-05-17
> 更新：Epic 升级为主管（替代原 ArkClaw 主管角色）；新增 GitHub Issue 任务池机制
> 架构说明：ArkClaw（火山引擎云Agent）降级为副主管/技术顾问，无文件系统权限

---

## 整体架构拓扑

```
老袁（战略/拍板/最终审核）
    │
    ├──→ Epic（主管）                              ← Hermes-Agent 云主机，常驻
    │         ↓ 创建 GitHub Issue（任务池）
    │         ↓ 飞书通知老袁（进度/结果）
    │    Forge（全栈执行）                         ← Hermes-Agent 云主机，常驻
    │    ├── 前后端开发：总站框架、HVAC 平台、UI
    │    └── 架构运维：云部署、配置、部署文档
    │         ↓ 轮询 GitHub Issues（每5分钟）
    │         ↓ 执行 → PR → 飞书汇报 Epic
    │
    ├──→ EpicOps（运营）                          ← Hermes-Agent，按需启动
    │         ↓ 轮询 GitHub Issues
    │    客服/文书/记账/下载分身（按需）
    │
    ├──→ Arena（阿丽娜，宣发）                     ← Hermes-Agent，按需启动
    │         ↓ 轮询 GitHub Issues
    │    直播协助/内容运营分身
    │
    └──→ ArkClaw（副主管/技术顾问）               ← 火山引擎云Agent，无文件系统
              直接向老袁汇报
              职责：对外服务路由/技术顾问/审核协助
```

---

## 任务池机制（核心设计）

### 为什么不用飞书 @

Agent 间飞书 @ 响应不稳定（Epic @Forge 不触发），改用 **GitHub Issues 作为任务池**，执行 Agent 主动轮询拉取。

### 完整流程

```
1. 老袁 → WorkBuddy → 小腾（转发飞书通知 Epic）
2. Epic → GitHub 创建 Issue（assignee=forge/epicops/arena，label=priority:P0/P1/P2）
3. Epic → 飞书群通知老袁：📋 新任务已派发 + Issue URL
4. Forge/EpicOps/Arena → 每5分钟轮询 GitHub：gh issue list --assignee @me --state open
5. 执行 Agent 抢到任务 → 开始执行
6. 执行完成 → 更新 Issue comment + 关闭 Issue → 飞书汇报老袁
7. Epic → 检查 PR/Issue 状态 → 验收通过/打回
```

### GitHub Issue 规范

| 字段 | 要求 |
|------|------|
| 标题格式 | `[Forge] <任务>` / `[EpicOps] <任务>` / `[Arena] <任务>` |
| assignee | 设置为对应 Agent 的 GitHub 用户名 |
| Label | `priority:P0`/`priority:P1`/`priority:P2` + `type:feat/fix/config/docs` |
| body | 任务描述 + 验收标准（含可验证凭证要求）|

仓库：epic-arena-dev/core（主仓库）

---

## Epic 职责（主管）

### 做什么
- 接收老袁指令 → 创建 GitHub Issue（任务池）
- 分配 Issue 给 Forge/EpicOps/Arena（通过 assignee）
- 审核 Forge 提交的 GitHub PR
- 飞书通知老袁任务派发/完成/风险
- 不合格 → 打回，说明问题，不关闭 Issue

### 不做什么
- 不写代码（Forge 职责）
- 不直接 SSH 操作服务器
- 不依赖飞书 @ 让 Agent 执行任务

---

## Forge 职责（执行）

### 分身类型
| 分身 | 职责 |
|------|------|
| 前后端开发分身 | 总站框架、HVAC 平台、UI/功能迭代 |
| 架构运维分身 | 云部署、配置管理、部署文档 |

### 工作流程
1. 每5分钟轮询：`gh issue list --assignee @me --state open --repo epic-arena-dev/core`
2. 按优先级 P0 > P1 > P2 处理
3. 抢到任务 → 立即执行（不等待）
4. 执行 → GitHub PR → 更新 Issue comment → 飞书汇报
5. Epic 审核 PR → 合并/打回

---

## EpicOps 职责（运营）

### 分身类型
| 分身 | 职责 |
|------|------|
| 客服分身 | 客户咨询、购买引导、开通、售后 |
| 内容运营分身 | 语料处理、清洗切片、向量入库 |
| 记账分身 | 订单记录、积分对账、自动开通 |
| 下载分身 | 抢占主机池（<10台），1人1IP1任务，下载即销毁 |

### 工作流程
1. 每5分钟轮询 GitHub Issues（assignee=epicops）
2. 收到任务 → 启动对应分身
3. 分身执行 → 更新 Issue → 飞书汇报

---

## ArkClaw 职责（副主管/技术顾问）

- 无文件系统权限，无法读写代码或配置
- 直接向老袁汇报，不参与任务池管理
- 职责：对外服务路由策略、多业务子域名调度、技术顾问

---

## 决策权限边界

| 决策项 | 负责方 |
|--------|--------|
| 需求拍板 / 架构变更 | 老袁 |
| 任务派发 / 审核 / 上线 | Epic |
| 任务执行 / 代码开发 | Forge |
| 运营操作 / 客服 / 开通 | EpicOps |
| 宣发 / 直播协助 | Arena |
| 技术顾问 / 对外路由 | ArkClaw |

---

## 当前状态（2026-05-17）

| 组件 | 状态 |
|------|------|
| Epic | 常驻，主管，任务池机制 V1 |
| Forge | 常驻，轮询任务池 |
| EpicOps | 按需启动，轮询任务池 |
| Arena | 按需启动，轮询任务池 |
| ArkClaw | 火山引擎，副主管 |
| GitHub Issue 任务池 | 已建立（仓库 epic-arena-dev/core）|
| 飞书群 | 通知老袁进度，Agent 间不走飞书 @ |
