# SOP-POS-WORKFLOW.md — POS 工作流标准操作流程

> 版本：V1.0 | 日期：2026-05-04
> 适用范围：Forge（执行）、ArkClaw（审核）、Epic（执法）
> 依据文档：需求总纲 V0.3 十九.5

---

## 一、流程总览

```
需求确认 → Issue 创建 → 分支开发 → 自审 → PR 提交 → 自动审核 → 人工 Review → 合并上线
```

| 阶段 | 执行者 | 产出物 | 状态流转 |
|------|--------|--------|----------|
| 需求确认 | 老袁/ArkClaw | 飞书群任务单 | Draft |
| Issue 创建 | ArkClaw | GitHub Issue | Open |
| 分支开发 | Forge | feat/XXX 或 config/XXX 分支 | In Progress |
| 自审 | Forge | report.md（自审报告） | Ready for Review |
| PR 提交 | Forge | Pull Request（含验收凭证） | Review |
| 自动审核 | GitHub Actions | 审核报告 | Pass / Blocked |
| 人工 Review | ArkClaw | Review 评论 | Approved / Changes Requested |
| 合并上线 | Forge | 合并 commit + 部署 | Closed / Done |

---

## 二、阶段详情

### 2.1 需求确认

**触发条件**：老袁在飞书群提出需求，或 ArkClaw 根据需求文档拆解任务。

**执行者**：ArkClaw

**输出规范**：
- 任务单必须包含：`task_id`、`action`（任务描述）、`priority`（P0/P1/P2）、`deadline`、`acceptance_criteria`（验收标准）
- 任务单发到飞书群，@Forge 接收
- 关联需求文档版本号（如：依据 V0.3 第X章）

**示例**：
```json
{
  "task_id": "TASK-20260504-001",
  "agent": "forge",
  "priority": "P0",
  "action": "实现国标查询接口 /api/hvac/standards",
  "deadline": "2026-05-06",
  "acceptance_criteria": [
    "接口返回 200，支持按关键词模糊搜索",
    "结果按相关度排序，每页 20 条",
    "请求耗时 < 2s"
  ],
  "source_doc": "需求总纲 V0.3 二.2 AI工程主投"
}
```

---

### 2.2 Issue 创建

**执行者**：ArkClaw

**步骤**：
1. 在对应仓库创建 Issue（标题格式：`[P0] 国标查询接口实现`）
2. Issue 内容包含：任务描述、验收标准、关联需求文档
3. Assignee 设为 Forge
4. Label 设为优先级标签
5. 飞书群通知：`已创建 Issue #XX，@Forge 请认领`

**Issue 模板**：
```markdown
## 任务描述
[任务描述内容]

## 验收标准
- [ ] [验收标准1]
- [ ] [验收标准2]

## 关联文档
- 需求总纲 V0.X 第X章

## 备注
[其他说明]
```

---

### 2.3 分支开发

**执行者**：Forge

**分支命名规范**：

| 类型 | 格式 | 示例 |
|------|------|------|
| 新功能 | `feat/任务简述` | `feat/standard-query-api` |
| 配置变更 | `config/任务简述` | `config/forge-env-update` |
| Bug 修复 | `fix/任务简述` | `fix/auth-token-expire` |
| 文档 | `docs/任务简述` | `docs/api-reference` |

**开发规范**：
1. 从 main 拉取最新代码：`git checkout main && git pull`
2. 创建分支：`git checkout -b feat/xxx`
3. 开发过程中 commit 频率自定，但 commit message 必须有意义
4. commit message 格式：`[类型] 简述（关联 Issue #XX）`
5. 开发完成后自测所有验收标准

**commit message 示例**：
```
feat: 实现国标查询接口（关联 Issue #12）

- 新增 /api/hvac/standards 端点
- 支持关键词模糊搜索 + 分页
- 添加单元测试
```

---

### 2.4 自审

**执行者**：Forge

**自审清单**：
- [ ] 所有验收标准通过
- [ ] 代码无硬编码密钥/API Key
- [ ] 无 console.log / print 调试残留
- [ ] 新增文件已添加到 .gitignore（如需忽略）
- [ ] 数据库操作按 enterprise_id 隔离
- [ ] 大文件（>5MB）已存 OSS，未提交到仓库
- [ ] 国标编号等数据未编造

**自审报告**（report.md）：
```markdown
## 自审报告 — TASK-20260504-001

### 验收结果
| 验收标准 | 结果 | 说明 |
|----------|------|------|
| 接口返回 200 | 通过 | curl 测试通过 |
| 模糊搜索 | 通过 | 测试用例覆盖 |
| 请求耗时 < 2s | 通过 | 平均 0.8s |

### 修改文件清单
- backend/api/standards.py（新增）
- backend/tests/test_standards.py（新增）

### 风险点
- 无
```

---

### 2.5 PR 提交

**执行者**：Forge

**步骤**：
1. 推送分支：`git push origin feat/xxx`
2. 创建 PR，标题格式：`[P0] 任务简述（关联 Issue #XX）`
3. PR 描述必须包含：
   - 关联 Issue 编号
   - 修改文件清单
   - 验收凭证（截图/URL/测试结果）
   - 自审结论

**PR 模板**：
```markdown
## 关联 Issue
Closes #XX

## 修改内容
- [修改1]
- [修改2]

## 验收凭证
| 验收标准 | 凭证 | 结果 |
|----------|------|------|
| 标准1 | [截图/URL] | 通过 |
| 标准2 | [截图/URL] | 通过 |

## 自审结论
[自审结论]

## 部署说明
[如需特殊部署步骤，在此说明]
```

**关键规则**：
- **无验收凭证的 PR，ArkClaw 有权直接打回**
- Epic 秘书监听 PR 提交，如缺验收凭证立即纠正

---

### 2.6 自动审核（GitHub Actions）

**触发**：PR 创建/更新时自动运行。

**审核项**：

| 审核项 | 工具 | 严重级别 | 处理 |
|--------|------|----------|------|
| 敏感文件检测 | GitLeaks | **阻断** | PR 不可合并 |
| 硬编码密钥扫描 | 正则匹配 | **警告** | 人工复核 |
| 核心代码路径校验 | 路径规则 | **阻断** | PR 不可合并 |
| Skill 安全扫描 | 自定义脚本 | **阻断** | PR 不可合并 |

**阻断项处理**：Forge 必须修复后重新推送，自动审核通过后才可进入人工 Review。

---

### 2.7 人工 Review

**执行者**：ArkClaw

**审核维度**：
1. 代码质量：逻辑正确性、可维护性、命名规范
2. 安全性：无敏感信息泄露、无越权操作
3. 完整性：所有验收标准通过
4. 一致性：与需求文档描述一致
5. 验收凭证：真实有效（非伪造截图）

**审核结论**：

| 结论 | 含义 | 后续 |
|------|------|------|
| Approved | 审核通过 | 可以合并 |
| Changes Requested | 需要修改 | Forge 修改后重新提交 |
| Blocked | 严重问题 | Forge 必须回退到开发阶段 |

**审核时效**：ArkClaw 收到 PR 后 24h 内完成审核（P0 任务优先）。

---

### 2.8 合并上线

**执行者**：Forge（ArkClaw 审核通过后执行）

**步骤**：
1. 合并 PR 到 main（Squash Merge，保留关联信息）
2. 关联 Issue 自动关闭
3. 飞书群通知：`Issue #XX 已合并，@ArkClaw 已审核通过`
4. 部署（如需）：
   - 配置变更：SCP 上传 + service restart
   - 代码变更：按部署流程执行
5. 验证部署成功：检查服务状态 + 关键接口

**上线后可观测性验证**：
- Langfuse 监控 30 分钟，确认无新增异常 trace（错误率、延迟突增）
- 如发现异常：立即走 SOP-DEV-ROLLBACK.md

**合并规则**：
- **禁止 force push main**
- 使用 Squash Merge 保持 main 历史整洁
- 合并后删除开发分支

---

## 三、异常处理

| 异常 | 处理方式 | 负责人 |
|------|----------|--------|
| 验收标准无法全部满足 | PR 标记 partial，说明原因，ArkClaw 决定是否接受 | Forge → ArkClaw |
| 自动审核阻断 | Forge 修复后重新推送 | Forge |
| ArkClaw 24h 未审核 | Forge 飞书提醒 ArkClaw | Forge |
| 线上 bug | 走 SOP-DEV-ROLLBACK.md 回滚流程 | Forge |
| 需求变更 | ArkClaw 更新 Issue，通知 Forge | ArkClaw |

---

## 四、Epic 执法点

Epic 秘书在以下节点强制执行检查：

| 节点 | 检查内容 | 违规处理 |
|------|----------|----------|
| PR 提交 | 是否包含验收凭证 | 缺凭证 → 直接指出，要求补充 |
| Issue 关联 | PR 是否关联 Issue | 未关联 → 提醒 Forge |
| 分支命名 | 是否符合命名规范 | 不符合 → 提醒 Forge |
| 自动审核结果 | 阻断项是否已修复 | 未修复 → 阻止合并 |

---

*本 SOP 由小腾根据需求总纲 V0.3 编写，2026-05-04*
