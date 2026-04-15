# 部署手册 (Deployment Playbook)

Matrix Labs 项目标准化部署 SOP。每次生产部署必须遵循相同的 4 产物流程 — 无例外。

**[English README](README.md)**

---

## 为什么需要这个

部署失败的原因是可预测的：未测试的代码、没有回滚方案、全量流量切换没有灰度、事故时权责不清。本手册通过要求 4 个标准产物来消除这些失败模式。

## 4 个必须产物

| # | 产物 | 模板 | 目的 |
|---|------|------|------|
| 1 | **测试报告** | [`templates/test-report.md`](templates/test-report.md) | 证明构建可以安全发布 |
| 2 | **部署计划** | [`templates/deployment-plan.md`](templates/deployment-plan.md) | 逐步操作 + 检查清单 |
| 3 | **灰度/金丝雀计划** | [`templates/canary-plan.md`](templates/canary-plan.md) | 分阶段发布 + 通过/回退决策门 |
| 4 | **回滚计划** | [`templates/rollback-plan.md`](templates/rollback-plan.md) | 出问题时如何恢复 |

---

## 如何使用

### 方式一：GitHub Issue 模板（推荐）

最快的落地方式。

**第 1 步：复制 issue 模板到你的项目**

```bash
# 在你的项目根目录
mkdir -p .github/ISSUE_TEMPLATE
curl -o .github/ISSUE_TEMPLATE/deployment-request.yml \
  https://raw.githubusercontent.com/WhiteMatrixTech/deployment-playbook/main/.github/ISSUE_TEMPLATE/deployment-request.yml
git add .github/ISSUE_TEMPLATE/deployment-request.yml
git commit -m "chore: 添加部署请求 issue 模板"
```

**第 2 步：创建部署请求**

进入你的项目 → Issues → New Issue → "部署请求 / Deployment Request"

Issue 表单**默认中文**（字段标签、区段标题、预填骨架全部为中文）。填写全部 4 个产物部分。

**第 3 步：获得审批**

| 风险等级 | 审批要求 |
|---------|---------|
| Low | 自行审批 |
| Medium | 一名 reviewer 审批 issue |
| High / Critical | 技术负责人 + 值班工程师 审批 |

**第 4 步：执行**

按部署计划中的 checkbox 逐项执行。每完成一步，勾选对应 checkbox。Issue 就成了本次部署的完整记录。

---

### 方式二：模板复制到项目中

适合习惯在代码库中管理文档的团队。

```bash
# 克隆本手册
git clone https://github.com/WhiteMatrixTech/deployment-playbook.git /tmp/dp

# 复制模板到你的项目
mkdir -p docs/deploy-templates
cp /tmp/dp/templates/*.md docs/deploy-templates/
cp /tmp/dp/checklists/*.md docs/deploy-templates/

# 每次部署时创建带日期的目录
mkdir -p docs/deploys/2026-04-15-backend-v1.5.3/
cp docs/deploy-templates/*.md docs/deploys/2026-04-15-backend-v1.5.3/
# 填写模板、提交、code review
```

---

### 方式三：AI Agent 生成

适合使用 Claude Code、Cursor、Copilot 等 AI 编码助手的团队。

**第 1 步：添加到项目的 agent 指令**

在 `CLAUDE.md`（或等效文件）中添加：

```markdown
## 部署 SOP

任何生产部署前，必须参考以下模板生成部署计划：
https://github.com/WhiteMatrixTech/deployment-playbook/templates/

必须产物：
1. 测试报告 (templates/test-report.md)
2. 部署计划 (templates/deployment-plan.md)
3. 灰度计划 (templates/canary-plan.md)
4. 回滚计划 (templates/rollback-plan.md)

使用项目实际的部署命令，不要用通用示例。
从真实的基线指标填充阈值。
输出为 GitHub Issue 格式。
```

**第 2 步：让 agent 生成**

```
帮我生成 backend v1.5.3 到生产环境的部署计划。
服务：openclaw-backend
变更：PR #592（修复微信绑定二维码）
部署命令：deploy.sh prod backend --build --infra --yes
回滚：gcloud run services update-traffic
```

Agent 会读取模板并生成包含全部 4 个产物的完整部署请求。

**第 3 步：Review 并执行**

Review 生成的计划。调整阈值和命令。创建 GitHub Issue。执行。

---

### 方式四：CI/CD 集成

在 CI 管线中强制执行本手册。

```yaml
# .github/workflows/deploy-gate.yml
name: Deploy Gate
on:
  issues:
    types: [labeled]

jobs:
  check-deployment-artifacts:
    if: contains(github.event.label.name, 'deployment')
    runs-on: ubuntu-latest
    steps:
      - name: 验证 4 个产物是否齐全
        run: |
          BODY="${{ github.event.issue.body }}"
          for section in "Test Report" "Deployment Plan" "Canary" "Rollback Plan"; do
            if ! echo "$BODY" | grep -qi "$section"; then
              echo "::error::缺少必须产物: $section"
              exit 1
            fi
          done
          echo "全部 4 个产物齐全"
```

---

## 仓库结构

```
deployment-playbook/
├── README.md                           # 英文版
├── README.zh.md                        # 中文版（本文件）
├── CLAUDE.md                           # AI agent 指令
├── templates/                          # 4 个核心模板
│   ├── test-report.md                  # 测试报告
│   ├── deployment-plan.md              # 部署计划（含 checkbox）
│   ├── canary-plan.md                  # 灰度/金丝雀发布计划
│   └── rollback-plan.md               # 回滚计划
├── .github/ISSUE_TEMPLATE/
│   └── deployment-request.yml          # GitHub Issue 表单（一键开部署 issue）
├── examples/
│   └── cloudrun-backend-deploy.md      # 真实 Cloud Run 部署示例
├── checklists/                         # 检查清单
│   ├── pre-deploy.md                   # 部署前门禁检查
│   ├── post-deploy.md                  # 部署后验证
│   └── incident-response.md            # 部署事故响应
└── CLAUDE.md                           # Agent 使用指南
```

## 支持的平台

模板与平台无关，但为以下平台提供了具体指引：

| 平台 | 灰度方式 | 回滚方式 | 示例 |
|------|---------|---------|------|
| **Google Cloud Run** | 流量分割（修订版 %） | 修订版回退（~30秒） | 已含 |
| **GCE / VM 池** | 滚动重启（1台→全量） | 镜像族群回退 | 已含 |
| **Kubernetes** | Istio/Linkerd 流量转移 | 回滚 deployment | 有指引 |
| **Serverless** | 别名灰度 | 别名回退 | 有指引 |
| **静态托管** (Vercel/CF) | Preview 部署 | `vercel rollback` | 有指引 |

## 工作流程图

```
              ┌─────────────┐
              │  代码就绪     │
              │  (PR 已合并)  │
              └──────┬───────┘
                     │
              ┌──────▼───────┐
              │  测试报告      │◄── 必须 PASS
              │  (产物 1)     │
              └──────┬───────┘
                     │
              ┌──────▼───────┐
              │  部署计划      │◄── 所有步骤带 checkbox
              │  (产物 2)     │
              └──────┬───────┘
                     │
              ┌──────▼───────┐
              │  灰度计划      │◄── 分阶段 + 决策门
              │  (产物 3)     │
              └──────┬───────┘
                     │
              ┌──────▼───────┐
              │  回滚计划      │◄── 预先规划、预先测试
              │  (产物 4)     │
              └──────┬───────┘
                     │
              ┌──────▼───────┐     ┌──────────────┐
              │  部署前检查    │────►│   已审批？    │
              └───────────────┘     └──────┬───────┘
                                           │
                                    ┌──────▼───────┐
                                    │    灰度部署    │
                                    └──────┬───────┘
                                           │
                                ┌──────────▼──────────┐
                           通过  │   指标正常？         │  未通过
                          ┌─────┤   (通过/回退 决策门)  ├─────┐
                          │     └─────────────────────┘     │
                   ┌──────▼───────┐              ┌──────────▼─────────┐
                   │  全量发布     │              │  执行回滚           │
                   └──────┬───────┘              └──────────┬─────────┘
                          │                                 │
                   ┌──────▼───────┐              ┌──────────▼─────────┐
                   │  部署后验证    │              │  事故响应           │
                   │              │              │  + 事后复盘         │
                   └──────────────┘              └────────────────────┘
```

## 核心原则

1. **没有计划不部署**。即使是"小修复"也要有轻量级计划。
2. **回滚永远预先规划**。不要在事故中现场想回滚方案。
3. **全量前必须灰度**。不灰度直接全量 = 事故等着发生。
4. **能自动则自动，不能自动则文档化**。手动步骤必须在 checkbox 里。
5. **事后复盘反哺模板**。每次事故后更新相关检查清单。
6. **无责文化**。事故是系统问题，不是个人问题。

## 常见问题

**Q: dev/staging 也要全部 4 个产物吗？**
A: 不需要。Dev 只需部署计划。Staging 需要部署计划 + 测试报告。生产环境需全部 4 个。

**Q: 一行代码的改动也要这么多吗？**
A: 用轻量版：测试报告（只写结论行） + 部署计划（3-4 个 checkbox） + 回滚计划（只写命令）。灰度可以写"N/A — hotfix，直接发布并监控"。

**Q: 紧急修复可以跳过灰度吗？**
A: 仅限 SEV-1（服务宕机）。在部署计划中记录跳过原因。灭火后补做灰度分析。

**Q: 如何适配我自己的项目？**
A: Fork 这个仓库，自定义模板（特别是部署命令和指标阈值），更新 Issue 模板。结构保持不变。

**Q: AI Agent 能用这个吗？**
A: 可以。把仓库 URL 加入 agent 上下文（CLAUDE.md、.cursorrules 等）。参见上文"方式三"。

## 贡献

1. Fork 本仓库
2. 编辑或添加 `templates/` 或 `checklists/` 下的模板
3. 提 PR，说明改了什么、为什么改
4. 至少一名 reviewer 审批

## 许可证

MIT
